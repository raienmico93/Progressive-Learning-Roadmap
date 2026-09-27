# React Side-Effect Management: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** React side-effect management is the practice of controlling operations that occur outside of React's pure rendering cycle—such as network requests, browser API interactions, and external subscriptions—using the `useEffect` Hook.

**Technical Definition:** In React, a "side effect" (often capitalized as "Effect" in React-specific contexts) is any operation that reaches outside the React component tree to synchronise with an external system. The `useEffect` Hook allows functional components to perform these operations after render, with optional cleanup logic to prevent resource leaks. Effects run at the end of a commit after the screen updates, making this an appropriate time to synchronise React components with external systems such as networks, browser APIs, or third-party libraries.

**Beginner-Friendly Explanation:** When you build a React app, most of your code calculates what should appear on screen. But sometimes you need to do things that aren't just calculations—like fetching data from a server, setting a timer, or listening for window resizing. These "side effects" can't happen during rendering because they would break React's rules. The `useEffect` Hook is React's way of letting you say: "After you've finished drawing the screen, please also do this other thing for me."

### Key Characteristics

- **Post-Render Execution:** Effects run after React has committed changes to the DOM, not during rendering.
- **Declarative Synchronisation:** Effects describe *what* should be synchronised, not *when* to imperatively execute code.
- **Cleanup Lifecycle:** Every Effect can return a cleanup function that runs before the next Effect execution or when the component unmounts.
- **Dependency-Driven:** The dependency array controls precisely when an Effect re-runs.
- **StrictMode Double-Invocation:** In development, React intentionally runs Effects twice to help detect missing cleanup logic.
- **Separation from Events:** Effects respond to rendering, not to specific user interactions. Event handlers handle user actions; Effects handle synchronisation caused by the component appearing or changing.

### Prerequisites

- Solid understanding of JavaScript fundamentals (closures, promises, asynchronous programming).
- Familiarity with React components, props, and state.
- Working knowledge of `useState` and the render/commit lifecycle.
- Basic understanding of the browser's event model and Web APIs.

### Related Programming Areas

- **Reactive Programming:** Managing data flow and change propagation.
- **Lifecycle Management:** Component mounting, updating, and unmounting.
- **Resource Management:** Preventing memory leaks and orphaned connections.
- **Concurrency Control:** Handling race conditions in asynchronous operations.
- **Declarative UI:** Describing outcomes rather than imperative steps.

### Core Concepts / Features

1. API Communication (Fetching Data and Mutations)
2. Browser APIs (localStorage, document.title, window size)
3. Timers (setTimeout, setInterval)
4. Event Subscriptions (window/DOM events)
5. WebSocket Connections
6. Third-Party Library Integration
7. Cleanup Responsibilities

---

## Core Concept 1: API Communication

### Definitions

**Core Definition:** API communication in React side-effect management refers to fetching data from and sending mutations to remote servers within the `useEffect` Hook lifecycle.

**Technical Definition:** API communication involves initiating HTTP requests (via `fetch` or libraries like Axios) inside a `useEffect` callback, managing loading/error states, and optionally using `AbortController` to cancel in-flight requests during cleanup. The Effect should be structured to handle asynchronous operations without directly passing async functions to `useEffect`.

**Beginner-Friendly Explanation:** When your React component needs to get information from a server (like a list of users) or send information to a server (like submitting a form), you use `useEffect` to start that conversation after the component appears on screen. You also need to handle what happens if the user leaves the page before the server responds—otherwise, your app might try to update a component that no longer exists.

### Purposes

- To retrieve remote data and display it in the component.
- To send user-generated data (mutations) to a server.
- To synchronise local component state with server-side state.
- To handle loading, success, and error states gracefully.
- To cancel pending requests when the component unmounts or dependencies change.

### Syntax Rules and Structure

**General Syntax with fetch:**
```javascript
useEffect(() => {
  let ignore = false;

  async function fetchData() {
    try {
      const response = await fetch(url);
      const data = await response.json();
      if (!ignore) {
        setData(data);
      }
    } catch (error) {
      if (!ignore) {
        setError(error);
      }
    }
  }

  fetchData();

  return () => {
    ignore = true;
  };
}, [url]);
```

**Component Breakdown:**
- `useEffect(() => { ... }, [url])`: Registers the Effect. The dependency array `[url]` means the Effect re-runs whenever `url` changes.
- `let ignore = false`: A flag to prevent state updates after the Effect has been cleaned up.
- `async function fetchData()`: An inner async function is defined because `useEffect` cannot directly accept an async function.
- `await fetch(url)`: Initiates the HTTP request.
- `if (!ignore) { setData(data) }`: Guards against updating state if the Effect has been cleaned up.
- `return () => { ignore = true }`: The cleanup function sets the flag, indicating that any pending request's result should be ignored.

**General Syntax with Axios:**
```javascript
useEffect(() => {
  const controller = new AbortController();

  axios.get('/api/users', { signal: controller.signal })
    .then(response => setData(response.data))
    .catch(error => {
      if (!axios.isCancel(error)) {
        setError(error);
      }
    })
    .finally(() => setLoading(false));

  return () => controller.abort();
}, []);
```

**Component Breakdown:**
- `new AbortController()`: Creates a controller for cancelling the request.
- `axios.get('/api/users', { signal: controller.signal })`: Passes the abort signal to Axios.
- `.then()` / `.catch()` / `.finally()`: Handles the promise chain.
- `axios.isCancel(error)`: Checks whether the error was caused by cancellation.
- `controller.abort()`: Cancels the request during cleanup.

**Syntax Rules:**
- Never pass an async function directly to `useEffect`. Define an async function inside and call it immediately.
- Always handle loading and error states explicitly.
- Use `AbortController` or a boolean flag to prevent state updates after unmounting.
- Include all variables used inside the Effect in the dependency array (e.g., `url`, `id`).
- When using `fetch`, check `response.ok` before parsing JSON.

**Constraints and Limitations:**
- `useEffect` does not support `async` callbacks directly.
- Race conditions can occur if the dependency changes before a previous request completes.
- StrictMode double-invocation in development will trigger two requests; the cleanup must handle this.
- Cancellation via `AbortController` is not supported by all environments (older browsers or certain polyfills).

### Annotated Code Examples

**Example 1: Fetching User Data with Loading and Error States**

```javascript
import React, { useState, useEffect } from 'react';

function UserProfile({ userId }) {
  // State for storing the fetched user data
  const [user, setUser] = useState(null);

  // State for tracking loading status
  const [loading, setLoading] = useState(true);
  
  // State for storing any error that occurs
  const [error, setError] = useState(null);

  useEffect(() => {
    // Create an AbortController to cancel the request if needed
    const controller = new AbortController();

    // Define the async function inside the Effect
    async function fetchUser() {
      try {
        // Reset states before starting the request
        setLoading(true);
        setError(null);

        // Make the API request with the abort signal
        const response = await fetch(
          `https://jsonplaceholder.typicode.com/users/${userId}`,
          { signal: controller.signal }
        );

        // Check if the response was successful
        if (!response.ok) {
          throw new Error(`HTTP error! Status: ${response.status}`);
        }

        // Parse the JSON response
        const data = await response.json();

        // Update state only if the request was not aborted
        setUser(data);
      } catch (err) {
        // Ignore abort errors (they are expected during cleanup)
        if (err.name !== 'AbortError') {
          setError(err.message);
        }
      } finally {
        // Stop loading regardless of success or failure
        setLoading(false);
      }
    }

    // Execute the fetch function
    fetchUser();

    // Cleanup: abort the request if the Effect re-runs or unmounts
    return () => {
      controller.abort();
    };
  }, [userId]); // Re-run when userId changes

  // Render loading state
  if (loading) return <p>Loading user profile...</p>;

  // Render error state
  if (error) return <p>Error: {error}</p>;

  // Render the user data
  return (
    <div>
      <h1>{user.name}</h1>
      <p>Email: {user.email}</p>
      <p>Phone: {user.phone}</p>
    </div>
  );
}

export default UserProfile;
```

**Expected Output:** When the component mounts with a valid `userId`, the browser first displays "Loading user profile...", then after the API responds, displays the user's name, email, and phone. If the request fails, an error message is shown. If the user navigates away or `userId` changes before the request completes, the request is aborted and no state update occurs.

**Why This Output Occurs:** The `loading` state starts as `true`, so the loading message renders first. The `useEffect` fires after the initial render, initiating the fetch. When the response arrives, `setUser` and `setLoading(false)` trigger a re-render showing the user data. The `AbortController` ensures that if `userId` changes or the component unmounts, the pending request is cancelled, preventing a state update on an unmounted component.

**Example 2: Sending a POST Request (Mutation)**

```javascript
import React, { useState } from 'react';

function CreatePost() {
  // State for form input
  const [title, setTitle] = useState('');

  // State for submission status
  const [submitting, setSubmitting] = useState(false);
  
  // State for success feedback
  const [success, setSuccess] = useState(false);

  // Event handler for form submission (NOT an Effect)
  async function handleSubmit(e) {
    e.preventDefault();
    setSubmitting(true);
    setSuccess(false);

    try {
      const response = await fetch('https://jsonplaceholder.typicode.com/posts', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          title: title,
          body: 'This is the post body.',
          userId: 1,
        }),
      });

      if (!response.ok) {
        throw new Error('Failed to create post');
      }

      setSuccess(true);
      setTitle('');
    } catch (error) {
      alert('Error: ' + error.message);
    } finally {
      setSubmitting(false);
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={title}
        onChange={(e) => setTitle(e.target.value)}
        placeholder="Post title"
        disabled={submitting}
      />
      <button type="submit" disabled={submitting || !title}>
        {submitting ? 'Submitting...' : 'Create Post'}
      </button>
      {success && <p>Post created successfully!</p>}
    </form>
  );
}

export default CreatePost;
```

**Expected Output:** The form displays an input and a submit button. When the user types a title and clicks "Create Post", the button text changes to "Submitting..." and becomes disabled. After the server responds successfully, the input clears and "Post created successfully!" appears.

**Why This Output Occurs:** This example deliberately uses an event handler rather than `useEffect` for the mutation because the POST request is directly caused by a user action (clicking the submit button), not by the component rendering. This is a critical distinction: **Effects are for synchronisation caused by rendering, while event handlers are for user-initiated actions**.

### Real-World Cases

- **E-commerce product listing:** Fetching products from an API when the category page loads, with pagination via dependency array changes.
- **User dashboard:** Loading user statistics and activity feed on mount, with real-time updates via WebSocket.
- **Search autocomplete:** Fetching search suggestions as the user types, debounced with timers and cancelled when the query changes.
- **Form submission:** Sending data to a REST API on button click (event handler) while displaying optimistic UI updates.
- **Infinite scroll:** Loading more items when the scroll position reaches a threshold, using Intersection Observer in an Effect.

---

## Core Concept 2: Browser APIs

### Definitions

**Core Definition:** Browser API interaction in React side-effect management refers to using Web Platform APIs—such as `localStorage`, `document.title`, `window.innerWidth`, and `navigator`—within the `useEffect` Hook to synchronise component state with the browser environment.

**Technical Definition:** Browser APIs are JavaScript interfaces provided by the browser runtime (not by React) that allow access to browser features. In React, these must be accessed inside `useEffect` because they are side effects that cannot run during rendering. In server-rendering environments (Next.js, Remix), browser APIs are unavailable on the server and must be guarded by environment checks such as `typeof window !== 'undefined'` or React 19.3's `browser()` API.

**Beginner-Friendly Explanation:** Your React app doesn't just run in a vacuum—it runs inside a web browser that has its own set of tools: storing data locally, changing the page title, checking how wide the window is, and so on. But these tools don't exist when React runs on a server (for server-side rendering). So you need to use `useEffect` to access them, because Effects only run in the browser.

### Purposes

- To persist data across browser sessions using `localStorage` or `sessionStorage`.
- To update the document title dynamically based on component state.
- To read and respond to browser window dimensions.
- To access browser language settings and user preferences.
- To interact with the clipboard, geolocation, or notification APIs.
- To safely separate browser-only code from server rendering.

### Syntax Rules and Structure

**General Syntax for localStorage:**
```javascript
useEffect(() => {
  // Read from localStorage on mount
  const savedTheme = localStorage.getItem('theme');
  if (savedTheme) {
    setTheme(savedTheme);
  }
}, []);

useEffect(() => {
  // Write to localStorage whenever theme changes
  localStorage.setItem('theme', theme);
}, [theme]);
```

**Component Breakdown:**
- First Effect with empty dependency array `[]`: Runs once on mount to read the initial value.
- `localStorage.getItem('theme')`: Retrieves the stored value (returns `null` if not found).
- Second Effect with `[theme]`: Runs whenever `theme` changes to persist the new value.
- `localStorage.setItem('theme', theme)`: Stores the value as a string.

**General Syntax for document.title:**
```javascript
useEffect(() => {
  // Save the previous title
  const previousTitle = document.title;

  // Set the new title
  document.title = `Page: ${pageName}`;

  // Restore the previous title on cleanup
  return () => {
    document.title = previousTitle;
  };
}, [pageName]);
```

**Component Breakdown:**
- `document.title = ...`: Directly sets the browser tab title.
- `return () => { document.title = previousTitle }`: Cleanup restores the title when the component unmounts or `pageName` changes.

**General Syntax for window size:**
```javascript
useEffect(() => {
  function handleResize() {
    setWindowSize({
      width: window.innerWidth,
      height: window.innerHeight,
    });
  }

  // Set initial size
  handleResize();

  // Add event listener
  window.addEventListener('resize', handleResize);

  // Cleanup: remove listener
  return () => {
    window.removeEventListener('resize', handleResize);
  };
}, []);
```

**Component Breakdown:**
- `handleResize`: Updates state with current window dimensions.
- `handleResize()`: Called immediately to set the initial size (the event won't fire until the user resizes).
- `window.addEventListener('resize', handleResize)`: Subscribes to resize events.
- `return () => { window.removeEventListener('resize', handleResize) }`: Unsubscribes on cleanup to prevent memory leaks.

**Syntax Rules:**
- Always guard browser API access with environment checks when using server-side rendering: `if (typeof window !== 'undefined')` or React 19.3's `browser()`.
- Never access browser APIs during rendering; always use `useEffect`.
- Provide cleanup for any event listeners added.
- When reading from `localStorage`, handle the case where the value is `null`.
- The `resize` event can fire very frequently; consider throttling or debouncing in production.

**Constraints and Limitations:**
- `localStorage` is synchronous and can block the main thread for large data.
- `localStorage` has a storage limit (typically 5–10 MB).
- `document.title` changes may not be reflected immediately in all browsers.
- `window.innerWidth` includes the scrollbar width, which may differ from `document.documentElement.clientWidth`.
- React 19.3's `browser()` API is version-specific and not available in earlier React versions.

### Annotated Code Examples

**Example 1: Persisting Theme Preference with localStorage**

```javascript
import React, { useState, useEffect } from 'react';

function ThemeToggle() {
  // Initialise theme from localStorage or default to 'light'
  const [theme, setTheme] = useState(() => {
    // Lazy initialiser runs only once during the first render
    if (typeof window !== 'undefined') {
      return localStorage.getItem('theme') || 'light';
    }
    return 'light';
  });

  // Effect: update localStorage whenever theme changes
  useEffect(() => {
    // Guard against server-side rendering where localStorage is unavailable
    if (typeof window !== 'undefined') {
      localStorage.setItem('theme', theme);
    }
  }, [theme]);

  // Effect: apply the theme to the document body
  useEffect(() => {
    document.body.className = theme;
  }, [theme]);

  return (
    <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
      Switch to {theme === 'light' ? 'Dark' : 'Light'} Mode
    </button>
  );
}

export default ThemeToggle;
```

**Expected Output:** A button labelled "Switch to Dark Mode" (or "Switch to Light Mode" depending on the saved preference). Clicking it toggles the theme, updates the body's CSS class, and persists the choice in `localStorage`. Reloading the page restores the saved theme.

**Why This Output Occurs:** The lazy initialiser in `useState` reads `localStorage` during the first render, but it is guarded by `typeof window !== 'undefined'` to prevent errors during server rendering. The first `useEffect` writes the theme to `localStorage` whenever it changes. The second `useEffect` applies the theme class to the document body. Because `localStorage` is a browser-only API, it must be accessed inside an Effect or a lazy initialiser with a guard.

**Example 2: Dynamic Document Title**

```javascript
import React, { useState, useEffect } from 'react';

function MessageCounter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    // Save the original title
    const originalTitle = document.title;

    // Update the title with the current count
    document.title = `(${count}) New Messages`;

    // Cleanup: restore the original title when the component unmounts
    return () => {
      document.title = originalTitle;
    };
  }, [count]); // Re-run whenever count changes

  return (
    <div>
      <p>You have {count} new messages.</p>
      <button onClick={() => setCount(c => c + 1)}>
        Add Message
      </button>
      <button onClick={() => setCount(0)}>
        Clear Messages
      </button>
    </div>
  );
}

export default MessageCounter;
```

**Expected Output:** The browser tab title displays "(0) New Messages" initially. Clicking "Add Message" increments the counter and updates the title to "(1) New Messages", "(2) New Messages", and so on. Clicking "Clear Messages" resets the count and title to "(0) New Messages".

**Why This Output Occurs:** The `useEffect` runs after every render where `count` has changed. It saves the original title, sets the new title based on `count`, and returns a cleanup function that restores the original title. When `count` changes, React first runs the cleanup from the previous Effect (restoring the old title) and then runs the new Effect (setting the new title).

### Real-World Cases

- **Dark mode toggle:** Persisting user theme preference in `localStorage`.
- **Document title for notifications:** Showing the unread message count in the browser tab.
- **Responsive layout:** Using `window.innerWidth` to conditionally render mobile or desktop layouts.
- **User language detection:** Reading `navigator.language` to set the application locale.
- **Form autosave:** Storing draft form data in `sessionStorage` so it survives page refreshes.

---

## Core Concept 3: Timers

### Definitions

**Core Definition:** Timer management in React side-effect management refers to using `setTimeout` and `setInterval` within the `useEffect` Hook to schedule delayed or repeated code execution, with mandatory cleanup to prevent memory leaks and unexpected behaviour.

**Technical Definition:** `setTimeout` schedules a callback to execute once after a specified delay, returning a timer ID. `setInterval` schedules a callback to execute repeatedly at a specified interval, also returning a timer ID. In React, both must be created inside `useEffect` and cleared in the cleanup function using `clearTimeout` or `clearInterval` respectively. Failure to clean up timers results in callbacks executing after the component has unmounted, leading to memory leaks and "Can't perform a React state update on an unmounted component" warnings.

**Beginner-Friendly Explanation:** Sometimes you want your app to do something after a delay (like hiding a notification after 3 seconds) or repeatedly at fixed intervals (like updating a clock every second). But if the user navigates away from the page, those timers keep running in the background unless you stop them. React's `useEffect` lets you start the timer and then stop it when the component is no longer needed.

### Purposes

- To delay the execution of code (e.g., showing a temporary notification).
- To execute code repeatedly at fixed intervals (e.g., polling for updates).
- To debounce or throttle user input (e.g., search suggestions).
- To implement animations or countdowns.
- To prevent memory leaks by clearing timers on unmount.

### Syntax Rules and Structure

**General Syntax for setTimeout:**
```javascript
useEffect(() => {
  // Start the timer
  const timerId = setTimeout(() => {
    // Code to execute after the delay
  }, delay);

  // Cleanup: cancel the timer if the Effect re-runs or unmounts
  return () => {
    clearTimeout(timerId);
  };
}, [delay]);
```

**Component Breakdown:**
- `setTimeout(() => { ... }, delay)`: Schedules the callback to run after `delay` milliseconds.
- `timerId`: The returned identifier used to cancel the timer.
- `return () => { clearTimeout(timerId) }`: Cancels the timer during cleanup.

**General Syntax for setInterval:**
```javascript
useEffect(() => {
  // Start the interval
  const intervalId = setInterval(() => {
    // Code to execute on each interval tick
  }, interval);

  // Cleanup: clear the interval
  return () => {
    clearInterval(intervalId);
  };
}, [interval]);
```

**Component Breakdown:**
- `setInterval(() => { ... }, interval)`: Schedules the callback to run every `interval` milliseconds.
- `intervalId`: The returned identifier.
- `return () => { clearInterval(intervalId) }`: Clears the interval during cleanup.

**Syntax Rules:**
- Always assign the timer ID to a constant or variable so it can be cleared.
- Always return a cleanup function that clears the timer.
- Include the delay/interval value in the dependency array if it is dynamic.
- Use a ref to store the timer ID if the callback needs access to the latest state without re-creating the timer.
- For timers that depend on changing state, consider using a ref for the callback to avoid stale closures.

**Constraints and Limitations:**
- Timer delays are not guaranteed to be precise; the browser may delay execution.
- `setInterval` can drift over time; for precise timing, use a self-rescheduling `setTimeout`.
- In React StrictMode, Effects run twice in development, which can cause timers to be created and destroyed twice.
- Accessing state inside a timer callback can capture a stale value if the callback is not updated when state changes.

### Annotated Code Examples

**Example 1: Auto-Hiding Notification with setTimeout**

```javascript
import React, { useState, useEffect } from 'react';

function Notification({ message, duration = 3000 }) {
  const [visible, setVisible] = useState(true);

  useEffect(() => {
    // If the notification is not visible, do nothing
    if (!visible) return;

    // Set a timer to hide the notification after `duration` ms
    const timerId = setTimeout(() => {
      setVisible(false);
    }, duration);

    // Cleanup: clear the timer if the component unmounts or visible changes
    return () => {
      clearTimeout(timerId);
    };
  }, [visible, duration]);

  // Do not render anything if not visible
  if (!visible) return null;

  return (
    <div className="notification">
      {message}
    </div>
  );
}

export default function App() {
  const [showNotification, setShowNotification] = useState(false);

  return (
    <div>
      <button onClick={() => setShowNotification(true)}>
        Show Notification
      </button>
      {showNotification && (
        <Notification message="Operation completed!" duration={2000} />
      )}
    </div>
  );
}
```

**Expected Output:** Clicking "Show Notification" displays a notification box with the message "Operation completed!". After 2 seconds, the notification disappears automatically.

**Why This Output Occurs:** When the `Notification` component mounts, `visible` is `true`, so the Effect runs and sets a `setTimeout` for 2000 ms. After 2000 ms, the callback calls `setVisible(false)`, which triggers a re-render. Since `visible` is now `false`, the component returns `null` and the notification disappears. The cleanup function clears the timer if the component unmounts before the delay elapses.

**Example 2: Real-Time Clock with setInterval**

```javascript
import React, { useState, useEffect } from 'react';

function Clock() {
  const [time, setTime] = useState(new Date());

  useEffect(() => {
    // Set up an interval that updates the time every second
    const intervalId = setInterval(() => {
      setTime(new Date());
    }, 1000);

    // Cleanup: clear the interval when the component unmounts
    return () => {
      clearInterval(intervalId);
    };
  }, []); // Empty array: set up the interval once on mount

  return (
    <div>
      <h1>Current Time</h1>
      <p>{time.toLocaleTimeString()}</p>
    </div>
  );
}

export default Clock;
```

**Expected Output:** The component displays the current time and updates it every second. The time continues to update as long as the component is mounted. When the component unmounts, the interval is cleared and no further updates occur.

**Why This Output Occurs:** The `useEffect` with an empty dependency array runs once on mount, creating a `setInterval` that calls `setTime(new Date())` every 1000 ms. Each call triggers a re-render with the new time. The cleanup function clears the interval when the component unmounts, preventing a memory leak.

### Real-World Cases

- **Toast notifications:** Auto-dismissing after a set duration.
- **Polling for updates:** Checking a server for new data every 30 seconds.
- **Debounced search:** Waiting for the user to stop typing before making an API request.
- **Countdown timers:** Displaying time remaining for an offer or quiz.
- **Animation frames:** Using `requestAnimationFrame` for smooth animations (with cleanup via `cancelAnimationFrame`).

---

## Core Concept 4: Event Subscriptions

### Definitions

**Core Definition:** Event subscription management in React side-effect management refers to using `useEffect` to add and remove global or DOM event listeners, ensuring that components respond to browser events without leaking listeners or causing memory leaks.

**Technical Definition:** Event subscriptions involve calling `addEventListener` on the `window` object, the `document`, or specific DOM elements inside a `useEffect` callback, and returning a cleanup function that calls `removeEventListener` with the same event type and handler reference. This pattern is essential for responding to events that occur outside the component's own DOM subtree, such as `resize`, `scroll`, `keydown`, `online`/`offline`, and `visibilitychange`. The handler function must be stable (defined inside the Effect or memoised with `useCallback`) to ensure `removeEventListener` removes the correct listener.

**Beginner-Friendly Explanation:** Sometimes your component needs to know when something happens outside of itself—like when the user resizes the window, presses a key, or scrolls the page. You can "listen" for these events by attaching an event listener. But if you don't remove the listener when your component disappears, it keeps listening forever, which wastes memory and can cause bugs. `useEffect` lets you attach the listener when the component appears and detach it when the component goes away.

### Purposes

- To respond to window resize events for responsive layouts.
- To handle keyboard shortcuts (e.g., Escape to close a modal).
- To detect scroll position for sticky headers or infinite scrolling.
- To listen for online/offline status changes.
- To handle clicks outside a component (e.g., closing a dropdown).
- To respond to browser visibility changes (e.g., pausing video when the tab is hidden).

### Syntax Rules and Structure

**General Syntax for window event subscription:**
```javascript
useEffect(() => {
  // Define the event handler inside the Effect
  function handleEvent(event) {
    // Handle the event
  }

  // Subscribe to the event
  window.addEventListener('eventName', handleEvent);

  // Cleanup: unsubscribe from the event
  return () => {
    window.removeEventListener('eventName', handleEvent);
  };
}, []); // Empty array: subscribe once on mount
```

**Component Breakdown:**
- `function handleEvent(event) { ... }`: Defines the handler inside the Effect so it is recreated with each Effect run (and the reference matches for removal).
- `window.addEventListener('eventName', handleEvent)`: Attaches the listener.
- `return () => { window.removeEventListener('eventName', handleEvent) }`: Removes the listener during cleanup.

**General Syntax for element-specific event subscription with useRef:**
```javascript
import { useRef, useEffect } from 'react';

function ClickOutside({ onClose }) {
  const ref = useRef(null);

  useEffect(() => {
    function handleClickOutside(event) {
      if (ref.current && !ref.current.contains(event.target)) {
        onClose();
      }
    }

    document.addEventListener('mousedown', handleClickOutside);

    return () => {
      document.removeEventListener('mousedown', handleClickOutside);
    };
  }, [onClose]);

  return <div ref={ref}>Click outside me to close</div>;
}
```

**Component Breakdown:**
- `useRef(null)`: Creates a ref to attach to the DOM element.
- `ref.current.contains(event.target)`: Checks whether the click occurred inside the referenced element.
- `document.addEventListener('mousedown', handleClickOutside)`: Subscribes to mouse events on the document.
- `onClose` in the dependency array: The Effect re-runs if the `onClose` callback changes.

**Syntax Rules:**
- The handler function must be defined inside the Effect (or memoised with `useCallback`) so that the same reference is used for both `addEventListener` and `removeEventListener`.
- If the handler depends on props or state, include them in the dependency array. Alternatively, use a ref to store the latest handler to avoid re-subscribing on every change.
- Always provide a cleanup function that removes the listener.
- For events on specific elements (not `window` or `document`), use a `ref` to access the element.
- Consider using `useEffectEvent` (React 19+) for a cleaner API when the handler needs access to the latest props/state without re-subscribing.

**Constraints and Limitations:**
- Re-adding listeners on every render can cause performance issues; use stable references or refs.
- Some events (e.g., `scroll`) fire very frequently and should be throttled or debounced.
- Passive event listeners (`{ passive: true }`) can improve scroll performance but cannot call `preventDefault()`.
- The `resize` event on `window` does not fire when the window is resized programmatically in some browsers.

### Annotated Code Examples

**Example 1: Window Resize Listener**

```javascript
import React, { useState, useEffect } from 'react';

function WindowSizeDisplay() {
  const [windowSize, setWindowSize] = useState({
    width: typeof window !== 'undefined' ? window.innerWidth : 0,
    height: typeof window !== 'undefined' ? window.innerHeight : 0,
  });

  useEffect(() => {
    // Handler that updates state with current window dimensions
    function handleResize() {
      setWindowSize({
        width: window.innerWidth,
        height: window.innerHeight,
      });
    }

    // Call once immediately to set the initial size
    handleResize();

    // Subscribe to the resize event
    window.addEventListener('resize', handleResize);

    // Cleanup: remove the event listener
    return () => {
      window.removeEventListener('resize', handleResize);
    };
  }, []); // Empty array: subscribe once on mount

  return (
    <div>
      <h1>Window Size</h1>
      <p>Width: {windowSize.width}px</p>
      <p>Height: {windowSize.height}px</p>
    </div>
  );
}

export default WindowSizeDisplay;
```

**Expected Output:** The component displays the current window width and height. As the user resizes the browser window, the displayed values update in real time.

**Why This Output Occurs:** On mount, the `useEffect` runs, calling `handleResize()` immediately to set the initial dimensions. It then adds a `resize` event listener to the window. Whenever the user resizes the window, `handleResize` fires and updates the state, causing a re-render with the new dimensions. The cleanup function removes the listener when the component unmounts.

**Example 2: Keyboard Shortcut Handler**

```javascript
import React, { useEffect } from 'react';

function KeyboardShortcut({ onSave }) {
  useEffect(() => {
    function handleKeyDown(event) {
      // Check for Ctrl+S or Cmd+S
      if ((event.ctrlKey || event.metaKey) && event.key === 's') {
        event.preventDefault(); // Prevent the browser's save dialog
        onSave();
      }
    }

    // Subscribe to keydown events on the window
    window.addEventListener('keydown', handleKeyDown);

    // Cleanup: remove the listener
    return () => {
      window.removeEventListener('keydown', handleKeyDown);
    };
  }, [onSave]); // Re-subscribe if onSave changes

  return <p>Press Ctrl+S (or Cmd+S) to save</p>;
}

export default KeyboardShortcut;
```

**Expected Output:** The component displays "Press Ctrl+S (or Cmd+S) to save". Pressing the shortcut triggers the `onSave` callback and prevents the browser's default save dialog.

**Why This Output Occurs:** The Effect attaches a `keydown` listener to the window. When the user presses Ctrl+S or Cmd+S, the handler checks the modifier keys and the key value, calls `preventDefault()` to stop the browser's default behaviour, and invokes the `onSave` callback. The cleanup function removes the listener when the component unmounts or when `onSave` changes.

### Real-World Cases

- **Modal Escape key:** Closing a modal when the user presses Escape.
- **Responsive navigation:** Showing/hiding mobile navigation based on window width.
- **Infinite scroll:** Detecting when the user scrolls near the bottom of the page.
- **Online/offline detection:** Displaying a banner when the user loses internet connectivity.
- **Click-outside detection:** Closing dropdowns, popovers, or modals when the user clicks outside them.

---

## Core Concept 5: WebSocket Connections

### Definitions

**Core Definition:** WebSocket connection management in React side-effect management refers to establishing, maintaining, and closing WebSocket connections within `useEffect`, ensuring that connections are properly closed on component unmount or dependency change.

**Technical Definition:** A WebSocket is a persistent, full-duplex communication channel over a single TCP connection. In React, the `WebSocket` constructor is called inside a `useEffect` callback, and the returned socket object is stored in a ref or state. Event handlers (`onopen`, `onmessage`, `onerror`, `onclose`) are assigned to update component state. The cleanup function must call `socket.close()` to terminate the connection, preventing orphaned connections and memory leaks. Common pitfalls include duplicate connections when the Effect re-runs without closing the previous connection, and state updates on unmounted components.

**Beginner-Friendly Explanation:** A WebSocket is like a phone call between your app and a server—once the connection is open, both sides can send messages back and forth instantly. But if you don't hang up when your component is no longer on screen, the connection stays open forever, wasting resources and potentially causing duplicate messages. `useEffect` lets you dial the server when the component appears and hang up when it disappears.

### Purposes

- To receive real-time updates from a server (e.g., chat messages, stock prices, notifications).
- To send messages to a server without HTTP request overhead.
- To maintain a persistent connection for collaborative features.
- To ensure that connections are closed when the component unmounts, preventing resource leaks.
- To handle reconnection logic when the connection drops.

### Syntax Rules and Structure

**General Syntax:**
```javascript
useEffect(() => {
  // Create the WebSocket connection
  const socket = new WebSocket('wss://example.com/socket');

  // Handle incoming messages
  socket.onmessage = (event) => {
    const data = JSON.parse(event.data);
    setMessages(prev => [...prev, data]);
  };

  // Handle connection open
  socket.onopen = () => {
    console.log('Connected');
  };

  // Handle errors
  socket.onerror = (error) => {
    console.error('WebSocket error:', error);
  };

  // Handle connection close
  socket.onclose = () => {
    console.log('Disconnected');
  };

  // Cleanup: close the connection
  return () => {
    socket.close();
  };
}, [url]); // Re-connect when url changes
```

**Component Breakdown:**
- `new WebSocket('wss://example.com/socket')`: Creates the connection.
- `socket.onmessage = (event) => { ... }`: Defines the message handler.
- `socket.onopen` / `socket.onerror` / `socket.onclose`: Lifecycle event handlers.
- `return () => { socket.close() }`: Closes the connection during cleanup.

**Syntax Rules:**
- Always close the WebSocket in the cleanup function.
- If the Effect can re-run (e.g., when `url` changes), close the existing connection before creating a new one.
- Store the socket in a `useRef` to access it from other handlers or to close it before creating a new one.
- Handle reconnection logic with exponential backoff if the connection drops unexpectedly.
- Parse incoming message data safely; wrap `JSON.parse` in a try-catch.
- Use `wss://` (WebSocket Secure) for production applications.

**Constraints and Limitations:**
- WebSocket connections are not automatically re-established after network interruptions; reconnection logic must be implemented manually or via a library.
- WebSockets are not supported in all environments (though support is now widespread).
- Binary data requires `ArrayBuffer` or `Blob` handling.
- In React StrictMode, the Effect runs twice in development, which can create and immediately close a connection; the cleanup should handle this gracefully.
- State updates from `onmessage` must be guarded against unmounted components.

### Annotated Code Examples

**Example 1: Basic WebSocket Message Receiver**

```javascript
import React, { useState, useEffect, useRef } from 'react';

function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  const [connected, setConnected] = useState(false);
  const socketRef = useRef(null);

  useEffect(() => {
    // Close any existing connection before creating a new one
    if (socketRef.current) {
      socketRef.current.close();
    }

    // Create a new WebSocket connection
    const socket = new WebSocket(`wss://echo.websocket.org/?room=${roomId}`);
    socketRef.current = socket;

    // Connection opened
    socket.onopen = () => {
      setConnected(true);
    };

    // Message received
    socket.onmessage = (event) => {
      setMessages(prev => [...prev, event.data]);
    };

    // Connection error
    socket.onerror = (error) => {
      console.error('WebSocket error:', error);
    };

    // Connection closed
    socket.onclose = () => {
      setConnected(false);
    };

    // Cleanup: close the connection and clear the ref
    return () => {
      socket.close();
      socketRef.current = null;
    };
  }, [roomId]); // Re-connect when roomId changes

  return (
    <div>
      <h2>Room: {roomId}</h2>
      <p>Status: {connected ? 'Connected' : 'Disconnected'}</p>
      <ul>
        {messages.map((msg, index) => (
          <li key={index}>{msg}</li>
        ))}
      </ul>
    </div>
  );
}

export default ChatRoom;
```

**Expected Output:** The component displays the room ID, connection status ("Connected" or "Disconnected"), and a list of messages. When the component mounts, it connects to the WebSocket server. Received messages appear in the list. When `roomId` changes, the old connection is closed and a new one is opened. When the component unmounts, the connection is closed.

**Why This Output Occurs:** The `useRef` stores the socket reference. Before creating a new connection, the Effect checks if `socketRef.current` exists and closes it. The `onopen`, `onmessage`, and `onclose` handlers update state accordingly. The cleanup function closes the socket and resets the ref. Using a ref ensures that the socket can be accessed and closed even if the Effect re-runs.

**Example 2: WebSocket with Reconnection Logic**

```javascript
import React, { useState, useEffect, useRef, useCallback } from 'react';

function RealTimeFeed({ feedUrl }) {
  const [data, setData] = useState([]);
  const [status, setStatus] = useState('disconnected');
  const socketRef = useRef(null);
  const reconnectTimeoutRef = useRef(null);

  const connect = useCallback(() => {
    // Close existing connection if any
    if (socketRef.current) {
      socketRef.current.close();
    }

    setStatus('connecting');
    const socket = new WebSocket(feedUrl);
    socketRef.current = socket;

    socket.onopen = () => {
      setStatus('connected');
    };

    socket.onmessage = (event) => {
      try {
        const parsed = JSON.parse(event.data);
        setData(prev => [...prev, parsed]);
      } catch (err) {
        console.error('Invalid message format:', err);
      }
    };

    socket.onclose = () => {
      setStatus('disconnected');
      // Attempt to reconnect after 3 seconds
      reconnectTimeoutRef.current = setTimeout(() => {
        connect();
      }, 3000);
    };

    socket.onerror = () => {
      setStatus('error');
    };
  }, [feedUrl]);

  useEffect(() => {
    connect();

    // Cleanup: close the connection and clear any pending reconnection
    return () => {
      if (socketRef.current) {
        socketRef.current.close();
      }
      if (reconnectTimeoutRef.current) {
        clearTimeout(reconnectTimeoutRef.current);
      }
    };
  }, [connect]);

  return (
    <div>
      <h2>Real-Time Feed</h2>
      <p>Status: {status}</p>
      <ul>
        {data.map((item, index) => (
          <li key={index}>{JSON.stringify(item)}</li>
        ))}
      </ul>
    </div>
  );
}

export default RealTimeFeed;
```

**Expected Output:** The component displays the connection status and incoming data. If the connection drops, the status changes to "disconnected" and after 3 seconds the component attempts to reconnect automatically. When the component unmounts, the connection is closed and any pending reconnection timer is cleared.

**Why This Output Occurs:** The `connect` function is memoised with `useCallback` so it can be safely used in the dependency array. The `onclose` handler sets a `setTimeout` for reconnection. The cleanup function closes the socket and clears the reconnection timer. The `useRef` for the reconnection timeout ensures it can be cleared even if the Effect re-runs.

### Real-World Cases

- **Chat applications:** Receiving and sending messages in real time.
- **Stock trading platforms:** Streaming live price updates.
- **Collaborative editing:** Syncing document changes across users.
- **IoT dashboards:** Receiving sensor data from connected devices.
- **Live sports scores:** Updating scores as events happen.

---

## Core Concept 6: Third-Party Library Integration

### Definitions

**Core Definition:** Third-party library integration in React side-effect management refers to initialising, configuring, and cleaning up non-React libraries (such as charting libraries, map providers, animation engines, and rich text editors) within the `useEffect` Hook.

**Technical Definition:** Many JavaScript libraries are designed to manipulate the DOM directly, manage their own internal state, and require explicit lifecycle management. In React, these libraries are integrated by using `useRef` to provide a stable DOM container, calling the library's initialisation code inside `useEffect`, and returning a cleanup function that destroys or tears down the library instance. This prevents duplicate instances, memory leaks, and conflicts with React's virtual DOM.

**Beginner-Friendly Explanation:** Sometimes you want to use a cool JavaScript library that wasn't built for React—like a charting library or a map. These libraries need to be started up when your component appears and shut down when it disappears. `useEffect` is the perfect place to do that: you start the library when the component mounts and stop it when the component unmounts.

### Purposes

- To render complex visualisations (charts, graphs, maps) that React cannot efficiently manage.
- To integrate animation libraries (GSAP, Anime.js) that manipulate the DOM directly.
- To embed third-party widgets (video players, social media embeds, payment forms).
- To use rich text editors that manage their own contenteditable DOM.
- To ensure that third-party instances are destroyed when the component unmounts, preventing memory leaks.

### Syntax Rules and Structure

**General Syntax:**
```javascript
import { useRef, useEffect } from 'react';
import ThirdPartyLibrary from 'third-party-library';

function MyComponent() {
  const containerRef = useRef(null);
  const libraryInstanceRef = useRef(null);

  useEffect(() => {
    // Initialise the library with the DOM container
    libraryInstanceRef.current = new ThirdPartyLibrary(containerRef.current, {
      // configuration options
    });

    // Cleanup: destroy the library instance
    return () => {
      if (libraryInstanceRef.current) {
        libraryInstanceRef.current.destroy();
        libraryInstanceRef.current = null;
      }
    };
  }, []); // Initialise once on mount

  return <div ref={containerRef}></div>;
}
```

**Component Breakdown:**
- `useRef(null)`: Creates a ref for the DOM container and the library instance.
- `new ThirdPartyLibrary(containerRef.current, { ... })`: Initialises the library, passing the DOM element.
- `return () => { libraryInstanceRef.current.destroy() }`: Destroys the library instance during cleanup.
- `return <div ref={containerRef}></div>`: The container element that the library will render into.

**Syntax Rules:**
- Always use `useRef` to provide a stable DOM container for the library.
- Initialise the library inside `useEffect`, never during rendering.
- Always return a cleanup function that destroys or tears down the library.
- Include any configuration values that affect initialisation in the dependency array if they can change.
- For libraries that need to update when props change, either re-initialise (with cleanup) or call an update method inside a separate Effect.

**Constraints and Limitations:**
- Some libraries do not provide a `destroy()` method; you may need to manually remove the DOM element or reset the container.
- Re-initialising on every render is expensive; use an empty dependency array for one-time setup.
- Libraries that manage their own event listeners must have those listeners cleaned up; check the library documentation.
- Server-side rendering requires special handling (e.g., dynamic imports) because third-party libraries often access `window` or `document`.

### Annotated Code Examples

**Example 1: Integrating a Charting Library (Chart.js)**

```javascript
import React, { useRef, useEffect } from 'react';
import Chart from 'chart.js/auto';

function SalesChart({ data }) {
  const canvasRef = useRef(null);
  const chartRef = useRef(null);

  useEffect(() => {
    // Initialise the chart on the canvas element
    const ctx = canvasRef.current.getContext('2d');
    chartRef.current = new Chart(ctx, {
      type: 'bar',
      data: {
        labels: data.labels,
        datasets: [{
          label: 'Sales',
          data: data.values,
          backgroundColor: 'rgba(54, 162, 235, 0.5)',
        }],
      },
      options: {
        responsive: true,
        plugins: {
          legend: { position: 'top' },
        },
      },
    });

    // Cleanup: destroy the chart instance
    return () => {
      if (chartRef.current) {
        chartRef.current.destroy();
        chartRef.current = null;
      }
    };
  }, []); // Initialise once; chart updates handled separately

  // Effect to update the chart when data changes
  useEffect(() => {
    if (!chartRef.current) return;

    // Update the chart's data and re-render
    chartRef.current.data.labels = data.labels;
    chartRef.current.data.datasets[0].data = data.values;
    chartRef.current.update();
  }, [data]);

  return (
    <div style={{ width: '600px', height: '400px' }}>
      <canvas ref={canvasRef}></canvas>
    </div>
  );
}

export default SalesChart;
```

**Expected Output:** The component renders a bar chart displaying sales data. When the `data` prop changes, the chart updates smoothly without being recreated.

**Why This Output Occurs:** The first `useEffect` initialises the Chart.js instance on the canvas element and stores the instance in a ref. The cleanup function destroys the chart when the component unmounts. The second `useEffect` watches for changes to the `data` prop and updates the chart's internal data, then calls `update()` to re-render. This two-Effect pattern separates one-time initialisation from ongoing updates.

**Example 2: Integrating GSAP for Animation**

```javascript
import React, { useRef } from 'react';
import { useGSAP } from '@gsap/react';
import gsap from 'gsap';

// Register the useGSAP hook as a plugin
gsap.registerPlugin(useGSAP);

function AnimatedBox({ isVisible }) {
  const containerRef = useRef(null);

  useGSAP(() => {
    // Animate the box whenever isVisible changes
    gsap.to('.box', {
      x: isVisible ? 300 : 0,
      rotation: isVisible ? 360 : 0,
      duration: 1,
      ease: 'power2.inOut',
    });
  }, { dependencies: [isVisible], scope: containerRef });

  return (
    <div ref={containerRef}>
      <div className="box" style={{
        width: 100,
        height: 100,
        backgroundColor: 'steelblue',
      }}></div>
    </div>
  );
}

export default AnimatedBox;
```

**Expected Output:** A blue square box appears on the page. When the `isVisible` prop becomes `true`, the box animates 300 pixels to the right while rotating 360 degrees. When `isVisible` becomes `false`, it animates back to its original position and rotation.

**Why This Output Occurs:** The `useGSAP` hook from `@gsap/react` is a drop-in replacement for `useEffect` that automatically handles cleanup using `gsap.context()`. The `scope` option ensures that selector text like `.box` only matches elements inside the container ref. The `dependencies` array ensures the animation re-runs when `isVisible` changes. The hook automatically reverts animations on cleanup, preventing memory leaks.

### Real-World Cases

- **Data dashboards:** Integrating Chart.js, D3.js, or Recharts for data visualisation.
- **Map applications:** Embedding Leaflet, Mapbox, or Google Maps.
- **Rich text editors:** Integrating Quill, TipTap, or ProseMirror.
- **Video players:** Embedding Video.js or Plyr.
- **Animation-heavy landing pages:** Using GSAP or Anime.js for scroll-triggered animations.
- **Payment forms:** Embedding Stripe Elements or PayPal buttons.

---

## Core Concept 7: Cleanup Responsibilities

### Definitions

**Core Definition:** Cleanup responsibilities in React side-effect management refer to the systematic disposal of resources—timers, event listeners, network connections, subscriptions, and third-party instances—that were created during the Effect's execution, executed when the component unmounts or before the Effect re-runs.

**Technical Definition:** The `useEffect` Hook accepts an optional return value: a cleanup function. React calls this function before the component is removed from the DOM (unmount) and before the next execution of the Effect (on dependency change). The cleanup function is essential for preventing memory leaks, avoiding state updates on unmounted components, and ensuring that external resources are properly released. Cleanup responsibilities include clearing timers (`clearTimeout`, `clearInterval`), removing event listeners (`removeEventListener`), cancelling network requests (`AbortController`), closing connections (`WebSocket.close()`), unsubscribing from observables, and destroying third-party instances.

**Beginner-Friendly Explanation:** When you start something in a `useEffect`—like a timer, an event listener, or a connection to a server—you need to stop it when your component is no longer needed. If you don't, those things keep running in the background, wasting memory and potentially causing bugs. React gives you a way to "clean up" after yourself by returning a function from `useEffect`. React calls this function automatically when the component disappears or when the Effect needs to run again.

### Purposes

- To prevent memory leaks caused by lingering timers, listeners, and connections.
- To avoid state updates on unmounted components.
- To ensure that external resources (network connections, subscriptions) are released.
- To maintain application performance by preventing unnecessary background work.
- To comply with React's requirement that Effects must be idempotent in StrictMode.

### Syntax Rules and Structure

**General Syntax:**
```javascript
useEffect(() => {
  // Setup: create resources
  const resource = createResource();

  // Return cleanup function
  return () => {
    // Cleanup: release resources
    resource.destroy();
  };
}, [dependencies]);
```

**Component Breakdown:**
- `useEffect(() => { ... }, [dependencies])`: The Effect Hook with a dependency array.
- Setup code: Runs after render when dependencies change or on mount.
- `return () => { ... }`: The cleanup function, called before the next Effect run and on unmount.
- `resource.destroy()`: Releases the resource (e.g., `clearTimeout`, `removeEventListener`, `socket.close()`).

**Syntax Rules:**
- The cleanup function must be returned from the Effect callback, not called directly.
- The cleanup function should mirror the setup: if you add a listener, remove it; if you create a timer, clear it; if you open a connection, close it.
- Do not include async operations in the cleanup function without careful handling.
- The cleanup function runs on every dependency change, not just on unmount.
- Multiple Effects should be used for unrelated concerns rather than combining them into one Effect with a complex cleanup.

**Constraints and Limitations:**
- The cleanup function cannot be async; if you need to await something during cleanup, you must handle it separately.
- In StrictMode, the cleanup function runs immediately after the Effect in development, which can cause issues if the cleanup is not idempotent.
- Some resources (e.g., `AbortController`) are not supported in older environments.
- Cleanup does not run when the browser tab is closed; use `beforeunload` for those scenarios.

### Annotated Code Examples

**Example 1: Comprehensive Cleanup for Multiple Resource Types**

```javascript
import React, { useState, useEffect, useRef } from 'react';

function Dashboard({ userId }) {
  const [data, setData] = useState(null);
  const [onlineStatus, setOnlineStatus] = useState(navigator.onLine);
  const socketRef = useRef(null);

  // Effect 1: Fetch data with abort control
  useEffect(() => {
    const controller = new AbortController();

    async function fetchData() {
      try {
        const response = await fetch(
          `https://api.example.com/users/${userId}`,
          { signal: controller.signal }
        );
        const result = await response.json();
        setData(result);
      } catch (error) {
        if (error.name !== 'AbortError') {
          console.error('Fetch error:', error);
        }
      }
    }

    fetchData();

    return () => {
      controller.abort(); // Cancel the fetch request
    };
  }, [userId]);

  // Effect 2: Online/offline event listeners
  useEffect(() => {
    function handleOnline() {
      setOnlineStatus(true);
    }
    function handleOffline() {
      setOnlineStatus(false);
    }

    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);

    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);

  // Effect 3: WebSocket connection
  useEffect(() => {
    const socket = new WebSocket(`wss://api.example.com/feed/${userId}`);
    socketRef.current = socket;

    socket.onmessage = (event) => {
      setData(prev => ({ ...prev, liveUpdate: event.data }));
    };

    return () => {
      socket.close(); // Close the WebSocket connection
      socketRef.current = null;
    };
  }, [userId]);

  // Effect 4: Timer for polling
  useEffect(() => {
    const intervalId = setInterval(() => {
      console.log('Polling for updates...');
    }, 30000);

    return () => {
      clearInterval(intervalId); // Clear the interval
    };
  }, []);

  return (
    <div>
      <h1>Dashboard</h1>
      <p>Status: {onlineStatus ? 'Online' : 'Offline'}</p>
      <p>User: {userId}</p>
      {data && <pre>{JSON.stringify(data, null, 2)}</pre>}
    </div>
  );
}

export default Dashboard;
```

**Expected Output:** The component displays user data, online/offline status, and live updates. When `userId` changes, the old fetch request is aborted, the old WebSocket is closed, and new resources are created. When the component unmounts, all four Effects run their cleanup functions, releasing all resources.

**Why This Output Occurs:** Each Effect is responsible for one concern and has its own cleanup function. The fetch Effect uses `AbortController` to cancel the request. The online/offline Effect removes its listeners. The WebSocket Effect closes the connection. The timer Effect clears the interval. This separation of concerns ensures that each resource is cleaned up correctly and independently.

**Example 2: Visualising Cleanup Order**

```javascript
import React, { useEffect } from 'react';

function CleanupDemo() {
  useEffect(() => {
    console.log('Effect 1: Setup');
    return () => {
      console.log('Effect 1: Cleanup');
    };
  }, []);

  useEffect(() => {
    console.log('Effect 2: Setup');
    return () => {
      console.log('Effect 2: Cleanup');
    };
  }, []);

  return <p>Check the console</p>;
}

export default CleanupDemo;
```

**Expected Output (in development with StrictMode):**
```
Effect 1: Setup
Effect 2: Setup
Effect 1: Cleanup
Effect 2: Cleanup
Effect 1: Setup
Effect 2: Setup
```

When the component unmounts:
```
Effect 1: Cleanup
Effect 2: Cleanup
```

**Why This Output Occurs:** In StrictMode, React runs the setup, then immediately runs the cleanup, then runs the setup again. This is intentional: it helps developers detect missing cleanup logic. The cleanup functions run in the order the Effects were defined. On unmount, both cleanup functions run in the same order.

### Real-World Cases

- **SPA navigation:** Cleaning up listeners and connections when the user navigates between routes.
- **Modal components:** Removing keydown listeners and clearing timers when a modal closes.
- **Data fetching components:** Aborting in-flight requests when the component unmounts.
- **Real-time dashboards:** Closing WebSocket connections when the user leaves the dashboard.
- **Animation components:** Reverting GSAP contexts or cancelling `requestAnimationFrame` loops.

---

## References

- React Official Documentation – Synchronizing with Effects: https://react.dev/learn/synchronizing-with-effects
- React Official Documentation – useEffect Reference: https://react.dev/reference/react/useEffect
- React Official Documentation – Removing Effect Dependencies: https://react.dev/learn/removing-effect-dependencies
- React Official Documentation – You Might Not Need an Effect: https://react.dev/learn/you-might-not-need-an-effect
- React Official Documentation – exhaustive-deps ESLint Rule: https://react.dev/reference/eslint-plugin-react-hooks/lints/exhaustive-deps
- MDN Web Docs – WebSocket API: https://developer.mozilla.org/en-US/docs/Web/API/WebSocket
- MDN Web Docs – AbortController: https://developer.mozilla.org/en-US/docs/Web/API/AbortController
- MDN Web Docs – setTimeout: https://developer.mozilla.org/en-US/docs/Web/API/setTimeout
- MDN Web Docs – setInterval: https://developer.mozilla.org/en-US/docs/Web/API/setInterval
- MDN Web Docs – Window.localStorage: https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage
- MDN Web Docs – EventTarget.addEventListener: https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener
- GSAP React Documentation: https://gsap.com/resources/React/
- Axios Documentation: https://axios-http.com/docs/intro
- Chart.js Documentation: https://www.chartjs.org/docs/latest/
- React 19.3 browser() API (C# Corner): https://www.c-sharpcorner.com/article/react-19-3-browser-keeping-browser-only-code-out-of-server-rendering
- LogRocket – How to use the useEffect hook in React: A complete guide: https://blog.logrocket.com/useeffect-react-hook-complete-guide/
- CoreUI – How to fetch data in React with Axios: https://coreui.io/answers/how-to-fetch-data-in-react-with-axios/
- Stack Overflow – How to properly handle WebSocket connections in a React app with useEffect: https://stackoverflow.com/questions/79540619/