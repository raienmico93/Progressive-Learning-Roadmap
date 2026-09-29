# React Advanced Hooks: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** React advanced hooks are the built-in Hooks beyond `useState`, `useEffect`, `useMemo`, and `useCallback`—`useLayoutEffect`, `useImperativeHandle`, `useId`, `useTransition`, `useDeferredValue`, `useSyncExternalStore`, and `useInsertionEffect`—that address specialised concerns such as layout measurement, imperative APIs, accessibility IDs, concurrent rendering, external store integration, and CSS-in-JS style insertion.

**Technical Definition:** Advanced Hooks are React's escape hatches for concerns that the primary Hooks cannot express cleanly. `useLayoutEffect` runs synchronously after DOM mutations but before the browser paints, enabling layout measurement and synchronous re-renders. `useImperativeHandle` customises the ref value exposed to parent components. `useId` generates unique, stable IDs that are SSR-safe. `useTransition` marks state updates as non-urgent, allowing React to keep the UI responsive. `useDeferredValue` defers a value's update, letting slow renders lag behind fast ones. `useSyncExternalStore` subscribes to external stores in a way that is safe with concurrent rendering. `useInsertionEffect` runs before layout effects and is designed exclusively for CSS-in-JS libraries to insert `<style>` tags before layout is read. Each Hook is a targeted tool for a specific problem, and using them incorrectly can cause subtle bugs or performance regressions.

**Beginner-Friendly Explanation:** Most of the time, `useState` and `useEffect` are all you need. But sometimes React's standard tools are not enough. You need to measure an element before the screen paints (useLayoutEffect). You need to expose a custom API to a parent component (useImperativeHandle). You need IDs that match between server and client (useId). You need to keep the UI responsive while a heavy update runs (useTransition, useDeferredValue). You need to connect to a store outside React (useSyncExternalStore). Or you are building a CSS-in-JS library and need to inject styles at exactly the right moment (useInsertionEffect). These advanced Hooks are the specialised tools for those jobs.

### Key Characteristics

- **Escape Hatches, Not Defaults:** Advanced Hooks exist for problems the primary Hooks cannot solve; they should not be used by default.
- **Synchronous vs. Asynchronous Timing:** `useLayoutEffect` and `useInsertionEffect` run synchronously during the commit phase; `useEffect` runs asynchronously after paint.
- **Concurrent-Rendering Safety:** `useTransition`, `useDeferredValue`, and `useSyncExternalStore` are designed for React 18+ concurrent rendering; they do not work correctly with older patterns.
- **SSR-Safe Identity:** `useId` generates IDs that are consistent between server and client, avoiding hydration mismatches.
- **Imperative Boundaries:** `useImperativeHandle` intentionally breaks the declarative model to expose imperative APIs (e.g., `focus()`, `scrollTo()`).
- **Store Integration:** `useSyncExternalStore` is the recommended way to subscribe to external stores without tearing during concurrent rendering.
- **Library-Only Hooks:** `useInsertionEffect` is designed for CSS-in-JS library authors, not application code.

### Prerequisites

- Solid understanding of React function components, JSX, and the primary Hooks.
- Working knowledge of the render/commit lifecycle and React's reconciliation.
- Familiarity with `useRef`, refs, and forwardRef.
- Basic understanding of React 18 concurrent rendering (`startTransition`, Suspense).
- Awareness of external state management (Redux, Zustand) and CSS-in-JS architectures.

### Related Programming Areas

- **Layout and Measurement:** Synchronous DOM reads, scroll position, element dimensions.
- **Imperative APIs:** Focus management, media playback, canvas control.
- **Accessibility:** Unique ID generation for `aria-labelledby`, `aria-describedby`, form labels.
- **Concurrent Rendering:** Transitions, deferred values, Suspense integration.
- **External Stores:** Subscription-based state (Redux, Zustand, browser APIs).
- **CSS-in-JS:** Style insertion order, `useInsertionEffect` for library authors.

### Core Concepts / Features

1. `useLayoutEffect`
2. `useImperativeHandle`
3. `useId`
4. `useTransition`
5. `useDeferredValue`
6. `useSyncExternalStore`
7. `useInsertionEffect`
8. Understanding When Advanced Hooks Are Appropriate

---

## Core Concept 1: `useLayoutEffect`

### Definitions

**Core Definition:** `useLayoutEffect` is a version of `useEffect` that fires synchronously after all DOM mutations but before the browser paints, allowing you to read layout from the DOM and synchronously re-render.

**Technical Definition:** `useLayoutEffect(setup, dependencies)` schedules a function to run after React has committed DOM changes but *before* the browser has painted those changes to the screen. This means any state updates triggered inside `useLayoutEffect` are flushed synchronously before paint, so the user never sees the intermediate state. This is essential for measuring DOM elements (e.g., `getBoundingClientRect()`) and adjusting the layout based on those measurements (e.g., tooltip positioning). The trade-off is performance: because `useLayoutEffect` blocks the browser's paint, heavy work inside it can cause visible jank. `useEffect` is the default; `useLayoutEffect` is used only when synchronous DOM measurement and re-rendering are required.

**Beginner-Friendly Explanation:** Imagine you are hanging a picture on the wall. `useEffect` is like hanging it, stepping back, looking at it, and then adjusting—the user sees the first position briefly. `useLayoutEffect` is like measuring the wall, adjusting the hook, and *then* showing the picture—the user never sees the wrong position. It is more work up front, but it prevents visible flicker.

### Purposes

- To measure DOM elements before the browser paints.
- To synchronously adjust layout based on measurements (tooltips, popovers, animations).
- To prevent visible flicker caused by a render that changes layout.
- To read scroll position and adjust before paint.
- To integrate with third-party libraries that require synchronous DOM reads.

### Syntax Rules and Structure

**General Syntax:**
```jsx
useLayoutEffect(() => {
  // Read layout from the DOM and synchronously re-render
  return () => {
    // Cleanup
  };
}, [dependencies]);
```

**Component Breakdown:**
- The setup function runs after React commits DOM changes but before paint.
- Any `setState` calls inside are flushed synchronously before paint.
- The optional cleanup function runs before the next setup or on unmount.
- Dependencies work exactly like `useEffect` (`Object.is` comparison).

**Measuring an Element:**
```jsx
import { useLayoutEffect, useRef, useState } from 'react';

function Tooltip({ targetRef, children }) {
  const tooltipRef = useRef(null);
  const [position, setPosition] = useState({ top: 0, left: 0 });

  useLayoutEffect(() => {
    if (!targetRef.current || !tooltipRef.current) return;
    const targetRect = targetRef.current.getBoundingClientRect();
    const tooltipRect = tooltipRef.current.getBoundingClientRect();
    setPosition({
      top: targetRect.top - tooltipRect.height - 8,
      left: targetRect.left + (targetRect.width - tooltipRect.width) / 2,
    });
  }, [targetRef]);

  return (
    <div
      ref={tooltipRef}
      style={{ position: 'absolute', top: position.top, left: position.left }}
    >
      {children}
    </div>
  );
}
```

**Syntax Rules:**
- Use `useLayoutEffect` only when you need to read layout and synchronously update state before paint.
- Use `useEffect` for everything else; it does not block paint.
- Always provide a dependency array; without it, the effect runs on every render.
- Avoid heavy computations inside `useLayoutEffect`; they block paint.
- In SSR, `useLayoutEffect` does not run on the server; use `useEffect` or guard against `window`.
- The cleanup function should mirror the setup (e.g., remove listeners, cancel animations).

**Constraints and Limitations:**
- `useLayoutEffect` blocks the browser's paint; heavy work causes visible jank.
- On the server, `useLayoutEffect` logs a warning because there is no DOM; use `useIsomorphicLayoutEffect` or `useEffect` for SSR.
- `useLayoutEffect` fires twice in StrictMode (development), like `useEffect`.
- Overusing `useLayoutEffect` for non-layout work degrades performance.
- It cannot be called conditionally; it must follow the Rules of Hooks.

### Annotated Code Example: Tooltip Positioning

```jsx
import { useLayoutEffect, useRef, useState } from 'react';

function Tooltip({ children, content }) {
  const [isVisible, setIsVisible] = useState(false);
  const [position, setPosition] = useState({ top: 0, left: 0 });
  const triggerRef = useRef(null);
  const tooltipRef = useRef(null);

  useLayoutEffect(() => {
    if (!isVisible || !triggerRef.current || !tooltipRef.current) return;

    const triggerRect = triggerRef.current.getBoundingClientRect();
    const tooltipRect = tooltipRef.current.getBoundingClientRect();

    setPosition({
      top: triggerRect.top - tooltipRect.height - 8,
      left: triggerRect.left + (triggerRect.width - tooltipRect.width) / 2,
    });
  }, [isVisible]);

  return (
    <span style={{ position: 'relative', display: 'inline-block' }}>
      <span
        ref={triggerRef}
        onMouseEnter={() => setIsVisible(true)}
        onMouseLeave={() => setIsVisible(false)}
      >
        {children}
      </span>
      {isVisible && (
        <div
          ref={tooltipRef}
          role="tooltip"
          style={{
            position: 'fixed',
            top: position.top,
            left: position.left,
            background: '#333',
            color: '#fff',
            padding: '4px 8px',
            borderRadius: 4,
            fontSize: 14,
            whiteSpace: 'nowrap',
            pointerEvents: 'none',
          }}
        >
          {content}
        </div>
      )}
    </span>
  );
}

export default function App() {
  return (
    <div style={{ padding: 100 }}>
      <Tooltip content="This is a tooltip">Hover me</Tooltip>
    </div>
  );
}
```

**Expected Output:** A "Hover me" text with a tooltip that appears above it when hovered. The tooltip is positioned correctly (centred above the trigger) without flickering, because the position is calculated in `useLayoutEffect` before paint.

**Why This Output Occurs:** `useLayoutEffect` runs after the tooltip is rendered into the DOM but before the browser paints. It measures the trigger and tooltip rects, calculates the position, and calls `setPosition`. React then synchronously re-renders with the correct position before paint, so the user never sees the tooltip at the wrong place.

### Real-World Cases

- **Tooltips and popovers:** Positioning based on trigger and content dimensions.
- **Dropdowns and menus:** Flipping direction when near viewport edges.
- **Animations:** Measuring element dimensions before animating.
- **Scroll restoration:** Restoring scroll position before paint.
- **Third-party integrations:** Charts, maps, and editors that require synchronous DOM reads.

### References

- React Official Documentation – `useLayoutEffect`: https://react.dev/reference/react/useLayoutEffect
- React Official Documentation – `useEffect` vs `useLayoutEffect`: https://react.dev/reference/react/useLayoutEffect#my-effect-is-rerunning-too-many-times
- MDN Web Docs – `getBoundingClientRect()`: https://developer.mozilla.org/en-US/docs/Web/API/Element/getBoundingClientRect
- React – Rules of Hooks: https://react.dev/reference/rules/rules-of-hooks

---

## Core Concept 2: `useImperativeHandle`

### Definitions

**Core Definition:** `useImperativeHandle` is a React Hook that customises the value exposed to a parent component when the parent accesses the child's ref, allowing the child to provide a custom imperative API.

**Technical Definition:** `useImperativeHandle(ref, createHandle, dependencies)` is used inside a component wrapped in `forwardRef`. It allows the component to control what value is assigned to the ref. Instead of exposing the raw DOM element (the default for a DOM ref), the component exposes an object with specific methods (e.g., `{ focus(), scrollTo(), play() }`). This is an escape hatch from React's declarative model, and its use should be limited to imperative APIs that cannot be expressed declaratively (focus, media playback, canvas control, scroll position). The `createHandle` function receives no arguments and returns the handle object; it is re-run when dependencies change.

**Beginner-Friendly Explanation:** Imagine you have a TV (the child component) and a remote control (the parent). You do not want to expose all the TV's internal buttons to the remote—only the useful ones like "power" and "volume". `useImperativeHandle` is how the TV says to the remote: "Here are the buttons you can use." The parent does not get full access to the child's internals; it gets only the specific methods the child chooses to expose.

### Purposes

- To expose a custom imperative API from a child component to its parent.
- To limit what a parent can do with the child (e.g., only `focus()` and `scrollTo()`).
- To integrate with third-party libraries that expect imperative handles (e.g., media players).
- To abstract DOM manipulation behind a clean API.
- To avoid exposing the raw DOM node when it is not necessary.

### Syntax Rules and Structure

**General Syntax:**
```jsx
const MyInput = forwardRef(function MyInput(props, ref) {
  const inputRef = useRef(null);

  useImperativeHandle(ref, () => ({
    focus() { inputRef.current.focus(); },
    scrollIntoView() { inputRef.current.scrollIntoView(); },
  }), []);

  return <input ref={inputRef} {...props} />;
});
```

**Component Breakdown:**
- `forwardRef`: Allows the component to receive a `ref` from its parent.
- `useImperativeHandle(ref, createHandle, deps)`: Defines what the ref exposes.
- `createHandle`: Returns the object with the imperative methods.
- `inputRef`: The internal ref to the actual DOM element.

**Using the Imperative Handle:**
```jsx
function Parent() {
  const inputRef = useRef(null);

  return (
    <div>
      <MyInput ref={inputRef} />
      <button onClick={() => inputRef.current.focus()}>Focus</button>
      <button onClick={() => inputRef.current.scrollIntoView()}>Scroll</button>
    </div>
  );
}
```

**Syntax Rules:**
- Use `forwardRef` to receive the ref from the parent.
- `useImperativeHandle` must be called inside a `forwardRef` component.
- The `createHandle` function returns the object that the parent sees.
- Include all reactive values used in the handle in the dependency array.
- Expose only the methods the parent genuinely needs.
- Prefer declarative props over imperative handles when possible.

**Constraints and Limitations:**
- `useImperativeHandle` breaks React's declarative model; it should be used sparingly.
- It requires `forwardRef`, adding boilerplate.
- In React 19, `ref` is passed as a regular prop to function components, simplifying the API.
- The handle is only available after the component mounts; calling methods before mount throws.
- Exposing too many methods couples the parent to the child's implementation.

### Annotated Code Example: Custom Video Player Handle

```jsx
import { forwardRef, useImperativeHandle, useRef, useState } from 'react';

const VideoPlayer = forwardRef(function VideoPlayer({ src }, ref) {
  const videoRef = useRef(null);
  const [isPlaying, setIsPlaying] = useState(false);

  useImperativeHandle(ref, () => ({
    play() {
      videoRef.current.play();
      setIsPlaying(true);
    },
    pause() {
      videoRef.current.pause();
      setIsPlaying(false);
    },
    seek(time) {
      videoRef.current.currentTime = time;
    },
    getDuration() {
      return videoRef.current.duration;
    },
  }), []);

  return (
    <div>
      <video ref={videoRef} src={src} width="320" />
      <p>{isPlaying ? 'Playing' : 'Paused'}</p>
    </div>
  );
});

export default function App() {
  const playerRef = useRef(null);

  return (
    <div>
      <VideoPlayer ref={playerRef} src="/video.mp4" />
      <div>
        <button onClick={() => playerRef.current.play()}>Play</button>
        <button onClick={() => playerRef.current.pause()}>Pause</button>
        <button onClick={() => playerRef.current.seek(30)}>Seek to 0:30</button>
        <button onClick={() => alert(`Duration: ${playerRef.current.getDuration()}s`)}>
          Show Duration
        </button>
      </div>
    </div>
  );
}
```

**Expected Output:** A video player with Play, Pause, Seek, and Show Duration buttons. Clicking Play starts the video, Pause stops it, Seek jumps to 30 seconds, and Show Duration alerts the total duration.

**Why This Output Occurs:** `useImperativeHandle` exposes only the `play`, `pause`, `seek`, and `getDuration` methods to the parent. The parent cannot access the underlying `<video>` element or any other internal state. The `videoRef` is private to the child.

### Real-World Cases

- **Video/audio players:** Play, pause, seek, volume.
- **Custom inputs:** Focus, blur, select, setSelectionRange.
- **Canvas components:** Draw, clear, export.
- **Chart components:** Resize, export image, scroll to data point.
- **Third-party integrations:** Leaflet maps, CodeMirror editors, Stripe Elements.

### References

- React Official Documentation – `useImperativeHandle`: https://react.dev/reference/react/useImperativeHandle
- React Official Documentation – `forwardRef`: https://react.dev/reference/react/forwardRef
- React Official Documentation – Manipulating the DOM with Refs: https://react.dev/learn/manipulating-the-dom-with-refs
- React 19 – Ref as a Prop: https://react.dev/blog/2024/12/05/react-19#ref-as-a-prop

---

## Core Concept 3: `useId`

### Definitions

**Core Definition:** `useId` is a React Hook that generates a unique, stable ID that is consistent between server and client, making it safe for accessibility attributes and SSR.

**Technical Definition:** `useId()` returns a unique string ID that is stable across renders and consistent between server-rendered HTML and client-side hydration. It is designed for accessibility attributes such as `id`, `htmlFor`, `aria-labelledby`, and `aria-describedby`, where the ID must match between the server-rendered HTML and the client. Unlike random ID generation (which causes hydration mismatches) or incrementing counters (which are not SSR-safe), `useId` generates IDs based on the component's position in the tree, ensuring they match on both sides. The ID format includes colons (e.g., `:r0:`) which are valid in HTML IDs but must be escaped in CSS selectors.

**Beginner-Friendly Explanation:** Imagine you have a form with labels and inputs. The label needs to know which input it belongs to (`htmlFor="email"`), and the input needs an `id="email"`. When you render on the server and then hydrate on the client, these IDs must match exactly, or React will warn about hydration mismatches. `useId` generates IDs that always match, no matter where the component renders. It is like a serial number that is assigned the same way on the server and the client.

### Purposes

- To generate unique IDs for accessibility attributes (`htmlFor`, `aria-labelledby`, `aria-describedby`).
- To avoid hydration mismatches between server-rendered and client-rendered HTML.
- To avoid collisions when the same component is rendered multiple times.
- To generate IDs without relying on external libraries or global counters.
- To support SSR, SSG, and concurrent rendering.

### Syntax Rules and Structure

**General Syntax:**
```jsx
const id = useId();
```

**Component Breakdown:**
- `useId()`: Returns a unique string ID.
- The ID is stable across re-renders of the same component instance.
- Different instances of the same component get different IDs.

**Basic Usage:**
```jsx
function EmailField() {
  const id = useId();
  return (
    <div>
      <label htmlFor={id}>Email</label>
      <input id={id} type="email" />
    </div>
  );
}
```

**Multiple IDs from One Call:**
```jsx
function FormField({ label, description }) {
  const id = useId();
  const inputId = `${id}-input`;
  const descId = `${id}-desc`;

  return (
    <div>
      <label htmlFor={inputId}>{label}</label>
      <input id={inputId} aria-describedby={descId} />
      <p id={descId}>{description}</p>
    </div>
  );
}
```

**Syntax Rules:**
- Call `useId` at the top level of a component.
- Use the returned ID for `id`, `htmlFor`, `aria-labelledby`, `aria-describedby`, and other ID-referencing attributes.
- Derive multiple IDs by suffixing (`${id}-input`, `${id}-desc`).
- Do not use `useId` to generate keys for list items; use stable data IDs instead.
- In CSS, escape the colons in the ID (e.g., `#\:r0\:` or use attribute selectors).

**Constraints and Limitations:**
- `useId` generates IDs that contain colons (`:r0:`), which are valid in HTML but must be escaped in CSS.
- `useId` should not be used to generate keys for lists; keys should come from data.
- The IDs are not guaranteed to be stable across different versions of React; do not persist them.
- `useId` is not a replacement for a UUID library when you need globally unique IDs (e.g., for database records).
- Calling `useId` conditionally violates the Rules of Hooks.

### Annotated Code Example: Accessible Form with useId

```jsx
import { useId } from 'react';

function TextField({ label, description, error, ...props }) {
  const id = useId();
  const inputId = `${id}-input`;
  const descId = `${id}-desc`;
  const errorId = `${id}-error`;

  const describedBy = [description && descId, error && errorId]
    .filter(Boolean)
    .join(' ') || undefined;

  return (
    <div style={{ marginBottom: 16 }}>
      <label htmlFor={inputId} style={{ display: 'block', fontWeight: 600 }}>
        {label}
      </label>
      <input
        id={inputId}
        aria-describedby={describedBy}
        aria-invalid={error ? 'true' : 'false'}
        {...props}
      />
      {description && (
        <p id={descId} style={{ color: '#666', fontSize: 14 }}>
          {description}
        </p>
      )}
      {error && (
        <p id={errorId} role="alert" style={{ color: 'red', fontSize: 14 }}>
          {error}
        </p>
      )}
    </div>
  );
}

export default function RegistrationForm() {
  return (
    <form>
      <TextField
        label="Email"
        description="We'll never share your email."
        type="email"
        placeholder="you@example.com"
      />
      <TextField
        label="Password"
        description="At least 8 characters."
        type="password"
        error="Password is required"
      />
    </form>
  );
}
```

**Expected Output:** A registration form with two fields. Each field has a label associated with its input via `htmlFor`/`id`, a description linked via `aria-describedby`, and an error message linked via `aria-describedby` and `aria-invalid`. Screen readers announce the label, description, and error when the input is focused.

**Why This Output Occurs:** `useId` generates a unique ID for each `TextField` instance. The label, description, and error are all linked to the input via `htmlFor`, `id`, and `aria-describedby`. Because `useId` is SSR-safe, the IDs match between server and client, avoiding hydration warnings.

### Real-World Cases

- **Form libraries:** React Aria and React Hook Form use `useId` internally for accessible IDs.
- **Design systems:** Every labelled input, dialog, and tab panel uses `useId` for ARIA relationships.
- **SSR/SSG applications:** Next.js, Remix, and Gatsby use `useId` to avoid hydration mismatches.
- **Component libraries:** Reusable components that render multiple times on the same page need unique IDs.

### References

- React Official Documentation – `useId`: https://react.dev/reference/react/useId
- React Official Documentation – Generating Unique IDs: https://react.dev/reference/react/useId#generating-unique-ids-for-accessibility-attributes
- React Official Documentation – Hydration Mismatches: https://react.dev/reference/react-dom/client/hydrateRoot#hydrating-server-rendered-html
- W3C WAI-ARIA – `aria-describedby`: https://www.w3.org/WAI/ARIA/apg/

---

## Core Concept 4: `useTransition`

### Definitions

**Core Definition:** `useTransition` is a React Hook that marks a state update as non-urgent (a "transition"), allowing React to keep the UI responsive by prioritising urgent updates and interrupting the transition if needed.

**Technical Definition:** `useTransition()` returns a tuple `[isPending, startTransition]`. The `startTransition` function accepts a callback containing state updates that should be treated as non-urgent. React prioritises urgent updates (typing, clicking) over transitions (rendering a large list, navigating to a new view). If the user types while a transition is running, React interrupts the transition, processes the urgent update, and restarts the transition with the new state. The `isPending` flag indicates whether a transition is in progress, allowing the UI to show a loading state without blocking. In React 19, `startTransition` supports async functions (Actions), enabling pending states for async work.

**Beginner-Friendly Explanation:** Imagine you are typing in a search box while a huge list of results is rendering. Without `useTransition`, the list update blocks the typing, and the input feels laggy. With `useTransition`, React says: "Typing is urgent; rendering the list can wait." If you keep typing, React interrupts the list rendering and restarts it with the new query. The input stays responsive, and the list catches up when you pause.

### Purposes

- To keep the UI responsive during expensive state updates.
- To mark non-urgent updates (search results, tab changes, navigation) as transitions.
- To show a pending indicator while a transition is in progress.
- To interrupt long-running renders when the user interacts.
- To integrate with Suspense for data fetching transitions.

### Syntax Rules and Structure

**General Syntax:**
```jsx
const [isPending, startTransition] = useTransition();

function handleClick() {
  startTransition(() => {
    setState(newValue); // Non-urgent update
  });
}
```

**Component Breakdown:**
- `isPending`: `true` while the transition is running.
- `startTransition`: Marks the enclosed state updates as non-urgent.
- The callback must be synchronous in React 18; React 19 supports async callbacks.

**Search with Transition:**
```jsx
import { useState, useTransition } from 'react';

function SearchPage() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  const [isPending, startTransition] = useTransition();

  function handleChange(e) {
    const value = e.target.value;
    setQuery(value); // Urgent: update the input immediately

    startTransition(() => {
      setResults(filterLargeList(value)); // Non-urgent: update results
    });
  }

  return (
    <div>
      <input value={query} onChange={handleChange} placeholder="Search..." />
      {isPending && <span>Updating results...</span>}
      <ul>
        {results.map((r) => <li key={r.id}>{r.name}</li>)}
      </ul>
    </div>
  );
}
```

**Syntax Rules:**
- Use `startTransition` for state updates that are not urgent (list filtering, tab switching).
- Keep urgent updates (input value) outside the transition.
- Use `isPending` to show a subtle loading indicator.
- In React 18, the callback must be synchronous; for async, use React 19's Actions.
- Transitions can be interrupted by urgent updates; do not rely on them for critical side effects.
- Do not use transitions for controlled input values (the input must update urgently).

**Constraints and Limitations:**
- Transitions do not work with state updates outside the `startTransition` callback.
- In React 18, `startTransition` cannot wrap async functions; use React 19's `startTransition(async () => {})`.
- Transitions are not a substitute for Suspense; they complement it.
- The `isPending` flag does not tell you which transition is running when there are multiple.
- Transitions do not prevent the state update from happening; they only make it interruptible.

### Annotated Code Example: Tab Switching with Transition

```jsx
import { useState, useTransition } from 'react';

function SlowList({ query }) {
  // Simulate a slow render
  const items = [];
  for (let i = 0; i < 10000; i++) {
    if (String(i).includes(query)) items.push(i);
  }
  return <ul>{items.map((i) => <li key={i}>{i}</li>)}</ul>;
}

export default function TabSwitcher() {
  const [tab, setTab] = useState('home');
  const [query, setQuery] = useState('');
  const [isPending, startTransition] = useTransition();

  function switchTab(newTab) {
    startTransition(() => {
      setTab(newTab);
    });
  }

  return (
    <div>
      <div>
        <button onClick={() => switchTab('home')} disabled={tab === 'home'}>
          Home
        </button>
        <button onClick={() => switchTab('list')} disabled={tab === 'list'}>
          List
        </button>
        {isPending && <span>Loading...</span>}
      </div>

      {tab === 'home' && <p>Welcome to the home tab.</p>}
      {tab === 'list' && (
        <div>
          <input
            value={query}
            onChange={(e) => setQuery(e.target.value)}
            placeholder="Filter numbers..."
          />
          <SlowList query={query} />
        </div>
      )}
    </div>
  );
}
```

**Expected Output:** Clicking "List" switches to a tab with 10,000 items. The tab switch is marked as a transition, so the UI shows "Loading..." while the slow list renders. If the user clicks "Home" while the list is rendering, React interrupts the list render and switches back immediately.

**Why This Output Occurs:** `startTransition(() => setTab(newTab))` marks the tab change as non-urgent. React begins rendering the slow list but keeps the UI responsive. `isPending` is `true` while the transition is running, showing "Loading...". If the user clicks another tab, React interrupts the current transition and processes the new one.

### Real-World Cases

- **Search boxes:** Filtering large lists while keeping typing responsive.
- **Tab switching:** Switching between heavy tabs without blocking.
- **Navigation:** Navigating to a new route while keeping the current page interactive.
- **Theme switching:** Re-rendering the whole app with a new theme without blocking.
- **Data table filtering:** Filtering and sorting large tables without input lag.

### References

- React Official Documentation – `useTransition`: https://react.dev/reference/react/useTransition
- React Official Documentation – `startTransition`: https://react.dev/reference/react/startTransition
- React 19 – Actions: https://react.dev/reference/react/useTransition#performing-actions-in-transitions
- React Official Documentation – Concurrent Rendering: https://react.dev/blog/2022/03/29/react-v18#what-is-concurrent-react

---

## Core Concept 5: `useDeferredValue`

### Definitions

**Core Definition:** `useDeferredValue` is a React Hook that returns a deferred version of a value, allowing the UI to lag behind fast updates (like typing) while a slow render catches up.

**Technical Definition:** `useDeferredValue(value)` accepts a value and returns a copy that "lags behind" the original. During urgent updates (typing), the deferred value remains at its previous value, so slow components that depend on it do not re-render immediately. React then schedules a background render with the new value. If the user makes another urgent update before the background render completes, React interrupts and restarts with the newest value. Unlike `useTransition`, `useDeferredValue` does not require you to wrap state updates in `startTransition`; it works on any value. It is especially useful when the slow component receives the value as a prop and you do not control the update.

**Beginner-Friendly Explanation:** Imagine you are typing in a search box, and a huge list of results is below it. Without `useDeferredValue`, every keystroke triggers a full re-render of the list, making typing laggy. With `useDeferredValue`, the input updates immediately (urgent), but the list uses a "deferred" version of the query that only updates when React has time. The input stays fast, and the list catches up.

### Purposes

- To keep fast updates (typing) responsive while slow renders (lists) catch up.
- To defer expensive renders without wrapping state updates in `startTransition`.
- To work with values passed as props when you do not control the state update.
- To show stale content while new content is being prepared.
- To integrate with Suspense for deferred data fetching.

### Syntax Rules and Structure

**General Syntax:**
```jsx
const deferredValue = useDeferredValue(value);
```

**Component Breakdown:**
- `value`: The value to defer (usually a fast-changing value like a query).
- `deferredValue`: A copy that lags behind the original until React has time to update it.

**Search with Deferred Value:**
```jsx
import { useState, useDeferredValue } from 'react';

function SearchPage() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);

  return (
    <div>
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Search..."
      />
      {/* SlowList uses the deferred query, so typing stays fast */}
      <SlowList query={deferredQuery} />
      {/* Show a stale indicator when deferred differs from current */}
      {query !== deferredQuery && <span>Updating...</span>}
    </div>
  );
}

function SlowList({ query }) {
  const items = [];
  for (let i = 0; i < 10000; i++) {
    if (String(i).includes(query)) items.push(i);
  }
  return <ul>{items.map((i) => <li key={i}>{i}</li>)}</ul>;
}
```

**Syntax Rules:**
- Use `useDeferredValue` for values that drive expensive renders (search queries, filter values).
- Do not use `useDeferredValue` for values that must be synchronised (form inputs' current value).
- Compare `value` and `deferredValue` to show a stale indicator.
- `useDeferredValue` accepts an optional second argument (`initialValue`) in React 19.
- Do not wrap the state update in `startTransition` when using `useDeferredValue`; they are alternatives.

**Constraints and Limitations:**
- `useDeferredValue` does not prevent the slow render; it only delays it.
- The deferred value eventually catches up; it is not a permanent cache.
- `useDeferredValue` only helps if the slow component's render is actually expensive; otherwise, it adds overhead.
- It cannot defer side effects (only render output).
- In React 18, the second argument (`initialValue`) is not available.

### Annotated Code Example: Deferred Filtering

```jsx
import { useState, useDeferredValue, memo } from 'react';

const ItemList = memo(function ItemList({ query }) {
  const items = [];
  for (let i = 0; i < 50000; i++) {
    if (String(i).includes(query)) items.push(i);
  }
  return (
    <ul>
      {items.slice(0, 100).map((i) => <li key={i}>{i}</li>)}
    </ul>
  );
});

export default function FilterApp() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);
  const isStale = query !== deferredQuery;

  return (
    <div>
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Filter numbers..."
      />
      <p>{isStale ? 'Updating...' : `${deferredQuery ? 'Filtered' : 'All'} results`}</p>
      <div style={{ opacity: isStale ? 0.5 : 1 }}>
        <ItemList query={deferredQuery} />
      </div>
    </div>
  );
}
```

**Expected Output:** Typing in the input is immediately responsive. The list updates with a slight delay (using the deferred query). While the list is catching up, the UI shows "Updating..." and the list is dimmed (opacity 0.5). Once the deferred value catches up, the list shows the correct results at full opacity.

**Why This Output Occurs:** The input uses `query` (urgent) so it updates immediately. `ItemList` uses `deferredQuery`, so it re-renders only when React has time. The `isStale` flag compares the two and shows the loading indicator and dimming. `React.memo` on `ItemList` prevents re-renders when `query` changes but `deferredQuery` has not yet.

### Real-World Cases

- **Search boxes:** Deferring search results while typing.
- **Filterable lists:** Deferring filter application on large datasets.
- **Data tables:** Deferring sort/filter operations.
- **Code editors:** Deferring syntax highlighting while typing.
- **Charts:** Deferring chart re-renders while dragging a slider.

### References

- React Official Documentation – `useDeferredValue`: https://react.dev/reference/react/useDeferredValue
- React Official Documentation – Deferring a Value: https://react.dev/reference/react/useDeferredValue#deferring-a-value
- React Official Documentation – `useTransition` vs `useDeferredValue`: https://react.dev/reference/react/useDeferredValue#whats-the-difference-between-usedeferredvalue-and-usetransition
- React Official Documentation – Concurrent Rendering: https://react.dev/blog/2022/03/29/react-v18#what-is-concurrent-react

---

## Core Concept 6: `useSyncExternalStore`

### Definitions

**Core Definition:** `useSyncExternalStore` is a React Hook that subscribes to an external store and returns its current value, designed to prevent tearing during concurrent rendering.

**Technical Definition:** `useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot?)` is the recommended way to read from external stores (Redux, Zustand, browser APIs, custom stores) in React. It accepts a `subscribe` function (which registers a callback and returns an unsubscribe function), a `getSnapshot` function (which returns the current store value), and an optional `getServerSnapshot` function (for SSR). React calls `getSnapshot` during render and compares the result with `Object.is` to decide whether to re-render. When the store changes, React schedules a re-render. The Hook ensures that all components see a consistent snapshot of the store, even during concurrent rendering—this is called "tearing prevention."

**Beginner-Friendly Explanation:** Imagine a big scoreboard at a sports game. Multiple cameras (components) are showing the score at slightly different times. During concurrent rendering, React might pause one camera to update another, and if the score changes in the meantime, one camera might show an old score while another shows a new one. That is "tearing." `useSyncExternalStore` is the official scoreboard feed that ensures every camera sees the same score at the same time. It is the recommended way to connect React to external state.

### Purposes

- To subscribe to external stores (Redux, Zustand, custom stores) safely in React 18+.
- To prevent tearing during concurrent rendering.
- To read browser APIs (online status, media queries, window dimensions) reactively.
- To integrate with third-party state libraries.
- To provide SSR-safe snapshots via `getServerSnapshot`.

### Syntax Rules and Structure

**General Syntax:**
```jsx
const snapshot = useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot);
```

**Component Breakdown:**
- `subscribe(onStoreChange)`: Registers a callback that React calls when the store changes; returns an unsubscribe function.
- `getSnapshot()`: Returns the current store value; must be stable (same value for the same store state).
- `getServerSnapshot()`: Returns the initial value on the server; must match the client's initial value.

**Custom Store Example:**
```jsx
import { useSyncExternalStore } from 'react';

// A minimal external store
function createStore(initialState) {
  let state = initialState;
  const listeners = new Set();

  return {
    getState: () => state,
    setState: (newState) => {
      state = newState;
      listeners.forEach((listener) => listener());
    },
    subscribe: (listener) => {
      listeners.add(listener);
      return () => listeners.delete(listener);
    },
  };
}

const counterStore = createStore({ count: 0 });

function useCounter() {
  const snapshot = useSyncExternalStore(
    counterStore.subscribe,
    counterStore.getState,
    counterStore.getState
  );
  return snapshot;
}

function Counter() {
  const { count } = useCounter();
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => counterStore.setState({ count: count + 1 })}>
        Increment
      </button>
    </div>
  );
}
```

**Browser API Example (Online Status):**
```jsx
import { useSyncExternalStore } from 'react';

function subscribe(callback) {
  window.addEventListener('online', callback);
  window.addEventListener('offline', callback);
  return () => {
    window.removeEventListener('online', callback);
    window.removeEventListener('offline', callback);
  };
}

function useOnlineStatus() {
  return useSyncExternalStore(
    subscribe,
    () => navigator.onLine,
    () => true // Assume online on the server
  );
}

function StatusIndicator() {
  const isOnline = useOnlineStatus();
  return <p>{isOnline ? 'Online' : 'Offline'}</p>;
}
```

**Syntax Rules:**
- `subscribe` must return an unsubscribe function.
- `getSnapshot` must return the same value for the same store state (no new objects on every call).
- `getServerSnapshot` is required for SSR; it must match the client's initial snapshot.
- The subscribe function must be stable (do not re-create it on every render).
- If `getSnapshot` returns a new object on every call, React will re-render infinitely.

**Constraints and Limitations:**
- `getSnapshot` must return a cached value; returning a new object/array on every call causes an infinite loop.
- The `subscribe` function must handle multiple subscribers correctly.
- `useSyncExternalStore` does not provide selectors; you must implement selective subscription yourself.
- Mutating the store during render is forbidden; updates must happen in event handlers or effects.
- Most state libraries (Redux, Zustand) already use `useSyncExternalStore` internally; you rarely call it directly.

### Annotated Code Example: Window Size Store

```jsx
import { useSyncExternalStore } from 'react';

function subscribe(callback) {
  window.addEventListener('resize', callback);
  return () => window.removeEventListener('resize', callback);
}

function getSnapshot() {
  return window.innerWidth;
}

function getServerSnapshot() {
  return 1024; // Default for SSR
}

function useWindowWidth() {
  return useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot);
}

export default function ResponsiveComponent() {
  const width = useWindowWidth();

  return (
    <div>
      <p>Window width: {width}px</p>
      <p>
        {width < 768
          ? 'Mobile layout'
          : width < 1024
          ? 'Tablet layout'
          : 'Desktop layout'}
      </p>
    </div>
  );
}
```

**Expected Output:** The component displays the current window width and the layout category (Mobile, Tablet, Desktop). Resizing the window updates the values in real time. On the server, the width defaults to 1024 (Desktop), avoiding hydration mismatches.

**Why This Output Occurs:** `useSyncExternalStore` subscribes to the `resize` event and reads `window.innerWidth` as the snapshot. When the window resizes, the subscriber callback fires, and React re-renders with the new width. `getServerSnapshot` returns a default value (1024) during SSR, so the server-rendered HTML matches the client's initial render.

### Real-World Cases

- **State libraries:** Redux, Zustand, Jotai, and Valtio use `useSyncExternalStore` internally.
- **Browser APIs:** Online/offline status, media queries, window dimensions, `prefers-color-scheme`.
- **Custom stores:** Application-specific stores that need to be shared across components.
- **WebSocket state:** Connection status and last message timestamp.
- **Third-party libraries:** Any external library with a subscription API.

### References

- React Official Documentation – `useSyncExternalStore`: https://react.dev/reference/react/useSyncExternalStore
- React Official Documentation – Adding a Store: https://react.dev/reference/react/useSyncExternalStore#adding-support-for-a-store
- React Official Documentation – Tearing: https://react.dev/reference/react/useSyncExternalStore#my-subscribe-function-gets-called-after-every-re-render
- Redux – `useSyncExternalStore` Integration: https://redux.js.org/usage/implementing-undo-history

---

## Core Concept 7: `useInsertionEffect`

### Definitions

**Core Definition:** `useInsertionEffect` is a React Hook that fires before any layout effects, designed specifically for CSS-in-JS libraries to insert `<style>` tags before layout is read.

**Technical Definition:** `useInsertionEffect(setup, dependencies)` fires synchronously after React commits DOM mutations but *before* `useLayoutEffect` runs. Its purpose is to allow CSS-in-JS libraries to inject `<style>` tags into the document before any component reads layout (which happens in `useLayoutEffect`). This prevents a flash of unstyled content and ensures that styles are present when layout effects measure the DOM. Application code should never use `useInsertionEffect`; it is exclusively for library authors (styled-components, Emotion, etc.). The Hook cannot update state, and its cleanup runs before the next setup.

**Beginner-Friendly Explanation:** Imagine you are getting dressed in the morning. `useInsertionEffect` is like putting on your underwear before your outer clothes—it has to happen first. `useLayoutEffect` is like checking yourself in the mirror (reading layout). If you check the mirror before putting on your clothes, you will see the wrong thing. `useInsertionEffect` ensures the styles (clothes) are in place before the layout (mirror check) happens. It is a library-only tool—app developers rarely need it.

### Purposes

- To insert `<style>` tags before layout effects run.
- To prevent a flash of unstyled content (FOUC).
- To ensure that styles are present when `useLayoutEffect` measures the DOM.
- To give CSS-in-JS libraries a dedicated insertion phase.
- To avoid the performance cost of inserting styles during render.

### Syntax Rules and Structure

**General Syntax:**
```jsx
useInsertionEffect(() => {
  // Insert styles here
  return () => {
    // Remove styles
  };
}, [dependencies]);
```

**Component Breakdown:**
- Runs synchronously after DOM mutations but before `useLayoutEffect`.
- Cannot call `setState` (it is not allowed).
- The cleanup removes the inserted styles.

**CSS-in-JS Library Example (Simplified):**
```jsx
function useCSS(rule) {
  useInsertionEffect(() => {
    const style = document.createElement('style');
    style.textContent = rule;
    document.head.appendChild(style);
    return () => {
      document.head.removeChild(style);
    };
  }, [rule]);
}
```

**Component Breakdown:**
- Creates a `<style>` element with the CSS rule.
- Appends it to `<head>` before layout effects run.
- Removes it on cleanup.

**Syntax Rules:**
- Use `useInsertionEffect` only in CSS-in-JS library code.
- Never call `setState` inside `useInsertionEffect`.
- The setup and cleanup must be synchronous.
- Dependencies work like `useEffect`.
- `useInsertionEffect` does not run on the server; SSR style extraction is handled separately.

**Constraints and Limitations:**
- Application code should not use `useInsertionEffect`; use `useEffect` or `useLayoutEffect` instead.
- `useInsertionEffect` cannot update state.
- It only runs on the client; server rendering requires a separate style extraction mechanism.
- The Hook is not intended for general side effects; it is exclusively for style insertion.
- Using it incorrectly can cause hydration mismatches or missing styles.

### Annotated Code Example: Minimal CSS-in-JS Runtime

```jsx
import { useInsertionEffect, useId } from 'react';

function useStyles(css) {
  const id = useId();
  useInsertionEffect(() => {
    const style = document.createElement('style');
    style.setAttribute('data-css-id', id);
    style.textContent = css;
    document.head.appendChild(style);
    return () => {
      document.head.removeChild(style);
    };
  }, [css, id]);
}

function Button({ children }) {
  useStyles(`
    .btn-${'x'} {
      background: #007bff;
      color: white;
      padding: 8px 16px;
      border-radius: 4px;
    }
  `);

  return <button className="btn-x">{children}</button>;
}

export default function App() {
  return (
    <div>
      <Button>Click me</Button>
    </div>
  );
}
```

**Expected Output:** A styled button with a blue background and white text. The styles are inserted into the `<head>` before layout effects run, so there is no flash of unstyled content.

**Why This Output Occurs:** `useInsertionEffect` fires before `useLayoutEffect` and before paint, ensuring the `<style>` tag is in the DOM when the browser is ready to paint. The cleanup removes the style tag when the component unmounts.

### Real-World Cases

- **styled-components:** Uses `useInsertionEffect` internally to insert styles.
- **Emotion:** Uses `useInsertionEffect` for style insertion.
- **Panda CSS:** Uses `useInsertionEffect` in its runtime components.
- **Any CSS-in-JS library:** The Hook exists specifically for this purpose.
- **Application code:** Never; use `useEffect` or `useLayoutEffect` instead.

### References

- React Official Documentation – `useInsertionEffect`: https://react.dev/reference/react/useInsertionEffect
- React Official Documentation – `useInsertionEffect` for Library Authors: https://react.dev/reference/react/useInsertionEffect#useinsertioneffect-for-library-authors
- React Official Documentation – `useLayoutEffect` vs `useInsertionEffect`: https://react.dev/reference/react/useInsertionEffect#whats-the-difference-between-useinsertioneffect-and-uselayouteffect
- styled-components – Internal Implementation: https://github.com/styled-components/styled-components

---

## Core Concept 8: Understanding When Advanced Hooks Are Appropriate

### Definitions

**Core Definition:** Understanding when advanced hooks are appropriate is the discipline of recognising the specific problems each Hook solves and using them only when those problems genuinely occur.

**Technical Definition:** Advanced Hooks are targeted solutions to specific problems. `useLayoutEffect` is for synchronous DOM measurement; `useImperativeHandle` is for exposing imperative APIs; `useId` is for SSR-safe accessibility IDs; `useTransition` and `useDeferredValue` are for concurrent rendering responsiveness; `useSyncExternalStore` is for external store integration; and `useInsertionEffect` is for CSS-in-JS library authors. Using an advanced Hook when a primary Hook would suffice is over-engineering; using a primary Hook when an advanced Hook is needed causes subtle bugs (flicker, hydration mismatches, torn UI). The decision framework is: identify the *symptom* (flicker, jank, hydration warning, tearing), find the *matching Hook*, and verify the fix with profiling or testing.

**Beginner-Friendly Explanation:** Advanced Hooks are like specialised tools in a workshop. You do not use a torque wrench to hammer a nail—you use it when you need precise torque. Similarly, you do not use `useTransition` for a small list, or `useImperativeHandle` for a simple ref. You use them when the specific problem they solve actually appears. The rule of thumb: start with the primary Hooks, and reach for advanced Hooks only when you can name the exact problem you are solving.

### Purposes

- To match the right Hook to the right problem.
- To avoid over-engineering simple components.
- To recognise the symptoms that indicate an advanced Hook is needed.
- To understand the trade-offs of each advanced Hook.
- To make informed decisions about when to use escape hatches.

### Syntax Rules and Structure

**Decision Matrix:**

| Symptom | Hook | Alternative to Consider First |
|---|---|---|
| Visible flicker when measuring DOM | `useLayoutEffect` | `useEffect` with CSS-based solutions |
| Need to expose `focus()`, `play()`, `scrollTo()` | `useImperativeHandle` | Declarative props (value, autoFocus) |
| Hydration mismatch from random IDs | `useId` | External UUID library (for non-ARIA IDs) |
| Laggy input while slow render runs | `useTransition` or `useDeferredValue` | Debouncing, virtualisation |
| UI tears when reading external store | `useSyncExternalStore` | State library (Redux, Zustand) |
| FOUC in a CSS-in-JS library | `useInsertionEffect` | `useLayoutEffect` (application code) |

**When to Use Each Hook:**

**`useLayoutEffect` — Use when:**
- You need to measure the DOM and synchronously re-render before paint.
- The user would see a flicker if you used `useEffect`.
- You are integrating with a library that requires synchronous DOM reads.

**`useImperativeHandle` — Use when:**
- A parent genuinely needs to call imperative methods on a child (play, focus, scroll).
- Declarative props cannot express the behaviour (e.g., "play from this timestamp").
- The imperative API is small and well-defined.

**`useId` — Use when:**
- You need unique IDs for `htmlFor`, `aria-labelledby`, `aria-describedby`.
- The component renders on both server and client (SSR).
- The same component appears multiple times on the page.

**`useTransition` — Use when:**
- A state update causes an expensive render that blocks the UI.
- The update is not urgent (tab switch, filter change).
- You can show a pending indicator.

**`useDeferredValue` — Use when:**
- A prop value drives an expensive render.
- You do not control the state update (the value comes from props).
- You want to show stale content while new content is prepared.

**`useSyncExternalStore` — Use when:**
- You are integrating an external store with React.
- You need to prevent tearing during concurrent rendering.
- You are reading browser APIs reactively (online status, media queries).

**`useInsertionEffect` — Use when:**
- You are a CSS-in-JS library author inserting `<style>` tags.
- Application developers should not use this Hook.

**Syntax Rules:**
- Start with primary Hooks (`useState`, `useEffect`, `useMemo`, `useCallback`, `useRef`).
- Reach for advanced Hooks only when a primary Hook cannot solve the problem.
- Name the specific symptom before choosing a Hook.
- Profile or test after applying an advanced Hook to confirm the fix.
- Remove the Hook if it does not measurably improve the situation.

**Constraints and Limitations:**
- Advanced Hooks add complexity; use them only when justified.
- Some advanced Hooks (e.g., `useSyncExternalStore`) are rarely needed in application code because libraries handle them.
- Overusing `useTransition` can make the UI feel unresponsive (deferred updates feel slow).
- `useLayoutEffect` in SSR causes warnings; guard against it.
- `useImperativeHandle` breaks the declarative model; use sparingly.

### Annotated Code Example: Choosing the Right Hook

```jsx
// Scenario 1: Tooltip positioning (needs synchronous measurement)
function Tooltip() {
  useLayoutEffect(() => { /* measure and position */ }, []);
  // ✅ useLayoutEffect is correct here
}

// Scenario 2: Video player controls (imperative API)
const VideoPlayer = forwardRef(function (props, ref) {
  useImperativeHandle(ref, () => ({ play, pause }), []);
  // ✅ useImperativeHandle is correct here
});

// Scenario 3: Accessible form field (SSR-safe IDs)
function TextField() {
  const id = useId();
  // ✅ useId is correct here
}

// Scenario 4: Large list filtering (concurrent rendering)
function SearchPage() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);
  // ✅ useDeferredValue is correct here
}

// Scenario 5: Online status (external store)
function useOnlineStatus() {
  return useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot);
  // ✅ useSyncExternalStore is correct here
}
```

**Expected Output:** Each scenario uses the correct Hook for its specific problem. None of them use an advanced Hook unnecessarily.

**Why This Output Occurs:** Each Hook solves a distinct problem: layout measurement, imperative API exposure, SSR-safe IDs, concurrent rendering responsiveness, and external store integration. Using the wrong Hook would cause flicker, boilerplate, hydration warnings, jank, or tearing.

### Real-World Cases

- **Design systems:** Use `useId` for all accessible IDs; use `useImperativeHandle` for inputs and media players.
- **Dashboards:** Use `useDeferredValue` for large data tables; use `useTransition` for tab switches.
- **SSR applications:** Use `useId` everywhere IDs are needed; avoid `useLayoutEffect` on the server.
- **State libraries:** Use `useSyncExternalStore` internally.
- **CSS-in-JS:** Use `useInsertionEffect` in the library runtime.

### References

- React Official Documentation – Hooks Reference: https://react.dev/reference/react/hooks
- React Official Documentation – Rules of Hooks: https://react.dev/reference/rules/rules-of-hooks
- React Official Documentation – Reusing Logic with Custom Hooks: https://react.dev/learn/reusing-logic-with-custom-hooks
- React Official Documentation – Synchronizing with Effects: https://react.dev/learn/synchronizing-with-effects

---

## Comparison and Decision Guidance

| Hook | Phase | Use Case | Application vs Library |
|---|---|---|---|
| **`useLayoutEffect`** | After DOM mutation, before paint | DOM measurement, synchronous re-render | Application (sparingly) |
| **`useImperativeHandle`** | During render (defines handle) | Expose imperative API to parent | Application (escape hatch) |
| **`useId`** | During render | SSR-safe unique IDs for ARIA | Application (common) |
| **`useTransition`** | During state update | Mark non-urgent updates | Application (concurrent UI) |
| **`useDeferredValue`** | During render | Defer slow value updates | Application (concurrent UI) |
| **`useSyncExternalStore`** | During render + subscription | External store integration | Library (state libraries) |
| **`useInsertionEffect`** | Before layout effects | Insert `<style>` tags | Library (CSS-in-JS only) |

**Decision Guidance:**
- **Start with the primary Hooks** (`useState`, `useEffect`, `useMemo`, `useCallback`, `useRef`).
- **Use `useLayoutEffect`** only when you need synchronous DOM measurement before paint.
- **Use `useImperativeHandle`** only when a parent genuinely needs imperative control.
- **Use `useId`** whenever you need IDs for accessibility attributes, especially in SSR.
- **Use `useTransition`** when an expensive update should not block urgent interactions.
- **Use `useDeferredValue`** when a prop-driven render is slow and you cannot wrap the update in `startTransition`.
- **Use `useSyncExternalStore`** when integrating an external store or browser API.
- **Use `useInsertionEffect`** only if you are building a CSS-in-JS library.
- **Always name the specific symptom** before reaching for an advanced Hook.
- **Profile or test** after applying an advanced Hook to confirm it helps.

---

## References

- React Official Documentation – `useLayoutEffect`: https://react.dev/reference/react/useLayoutEffect
- React Official Documentation – `useImperativeHandle`: https://react.dev/reference/react/useImperativeHandle
- React Official Documentation – `useId`: https://react.dev/reference/react/useId
- React Official Documentation – `useTransition`: https://react.dev/reference/react/useTransition
- React Official Documentation – `useDeferredValue`: https://react.dev/reference/react/useDeferredValue
- React Official Documentation – `useSyncExternalStore`: https://react.dev/reference/react/useSyncExternalStore
- React Official Documentation – `useInsertionEffect`: https://react.dev/reference/react/useInsertionEffect
- React Official Documentation – `forwardRef`: https://react.dev/reference/react/forwardRef
- React Official Documentation – `startTransition`: https://react.dev/reference/react/startTransition
- React Official Documentation – Rules of Hooks: https://react.dev/reference/rules/rules-of-hooks
- React Official Documentation – Reusing Logic with Custom Hooks: https://react.dev/learn/reusing-logic-with-custom-hooks
- React 19 – Ref as a Prop: https://react.dev/blog/2024/12/05/react-19#ref-as-a-prop
- React 19 – Actions: https://react.dev/reference/react/useTransition#performing-actions-in-transitions
- React 18 – Concurrent Rendering: https://react.dev/blog/2022/03/29/react-v18#what-is-concurrent-react
- MDN Web Docs – `getBoundingClientRect()`: https://developer.mozilla.org/en-US/docs/Web/API/Element/getBoundingClientRect
- MDN Web Docs – `navigator.onLine`: https://developer.mozilla.org/en-US/docs/Web/API/Navigator/onLine
- styled-components – Internal Implementation: https://github.com/styled-components/styled-components
- Redux – `useSyncExternalStore` Integration: https://redux.js.org/usage/implementing-undo-history