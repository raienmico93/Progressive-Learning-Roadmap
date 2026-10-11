# `useRef` Hook — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** `useRef` is a React Hook that lets you reference a value that is not needed for rendering.

**Technical Definition:** `useRef(initialValue)` is a React Hook that returns a mutable ref object with a single property called `current`, initially set to the `initialValue` you provided. The ref object persists across renders, and unlike state, mutating `ref.current` does not trigger a re-render. Refs are React's "escape hatch" from the one-way data flow — they let you store information that React does not track, such as DOM nodes, timer IDs, or any mutable value that should not affect the visual output of the component. `useRef` must be called at the top level of a component or custom Hook, and you must not read or write `ref.current` during rendering (except for initialization).

**Beginner-Friendly Explanation:** `useRef` is like a secret pocket for your component. You can put anything in it — a DOM element, a number, an object — and it stays there between renders. The key difference from state is that changing what is in the pocket does not cause the component to re-render. Think of it as a box that React does not look inside. If you need to remember something but do not want the screen to update when it changes, `useRef` is the tool for the job.

### Key Characteristics

- **Persistent:** The ref object is the same object on every render; it survives re-renders.
- **Mutable:** You can freely read and write `ref.current` without triggering re-renders.
- **Non-reactive:** React does not track changes to `ref.current`; it will not re-render when the value changes.
- **Escape hatch:** Designed for stepping outside React's declarative model to interact with external systems.
- **Two main uses:** Storing mutable values (timer IDs, previous values) and accessing DOM elements.
- **Top-level only:** Must be called at the top level of a component or custom Hook.

### Prerequisites

- JavaScript fundamentals: objects, functions, closures, and the DOM.
- React function components, JSX, and event handling.
- Understanding of `useState` and re-rendering.
- Familiarity with browser APIs (`focus`, `scrollIntoView`, `setInterval`, etc.).

### Related Programming Areas

- **`useState`** — the reactive counterpart to `useRef`; triggers re-renders.
- **`forwardRef`** — forwarding refs through components.
- **`useImperativeHandle`** — customising the ref handle exposed to parents.
- **DOM APIs** — `focus()`, `scrollIntoView()`, `getBoundingClientRect()`, etc.

---

## 1. `useRef`

### Definitions

**Core Definition:** `useRef` is a React Hook that returns a mutable ref object whose `current` property persists across renders.

**Technical Definition:** `const ref = useRef(initialValue)` returns an object with a single property, `current`, initially set to `initialValue`. The same object is returned on every render. You can mutate `ref.current` at any time without triggering a re-render. React does not track changes to refs, so you must not read or write `ref.current` during rendering (except for initialization) — doing so makes component behaviour unpredictable. In Strict Mode, React calls the component function twice in development, creating two ref objects but discarding one.

**Beginner-Friendly Explanation:** `useRef` gives you a box that stays the same between renders. You can put anything in the box and change it whenever you want. React does not watch the box, so changing it does not update the screen.

### Purposes

- To store mutable values that should not trigger re-renders (timer IDs, previous values).
- To access DOM elements directly for imperative operations (focus, scroll, measure).
- To persist values between renders without causing unnecessary re-renders.
- To serve as an "escape hatch" for interacting with external systems.

### Syntax Rules and Structure

**General Syntax:**

```jsx
import { useRef } from 'react';

const ref = useRef(initialValue);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `initialValue` | The initial value for `ref.current`; ignored after the first render. |
| `ref` | The ref object with a mutable `current` property. |

**Syntax Rules:**

1. Call `useRef` at the top level of a component or custom Hook.
2. Do not read or write `ref.current` during rendering.
3. Mutating `ref.current` does not trigger a re-render.
4. The ref object is the same on every render.

**Constraints and Limitations:**

- Cannot be called inside loops or conditions.
- Refs are not a replacement for state; use state for values that affect rendering.
- In Strict Mode, ref objects are created twice in development.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Storing a Mutable Value (Click Counter)

```jsx
import { useRef } from 'react';

function ClickCounter() {
  // Step 1: Create a ref with initial value 0
  const countRef = useRef(0);

  // Step 2: Mutate ref.current in an event handler
  function handleClick() {
    countRef.current = countRef.current + 1;
    alert(`You clicked ${countRef.current} times!`);
  }

  return <button onClick={handleClick}>Click me!</button>;
}
```

**Expected Output:** An alert showing the click count. The component never re-renders.

**Why this output:** `countRef.current` is mutated in the event handler. React does not track this change, so the component does not re-render. The value persists between clicks because the ref object is the same on every render.

#### Example 2: Accessing a DOM Element (Focus Input)

```jsx
import { useRef } from 'react';

function Form() {
  // Step 1: Create a ref for the input element
  const inputRef = useRef(null);

  // Step 2: Access the DOM node in an event handler
  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <input ref={inputRef} />
      <button onClick={handleClick}>Focus the input</button>
    </>
  );
}
```

**Expected Output:** Clicking the button focuses the input.

**Why this output:** Passing `ref={inputRef}` to the `<input>` tells React to put the DOM node into `inputRef.current`. The event handler reads `inputRef.current` and calls `focus()` on it.

### Real-World Cases

- **Focus management:** Focusing an input after a button click or form submission.
- **Scroll position:** Scrolling to an element or restoring scroll position.
- **Timers:** Storing `setInterval` IDs for later `clearInterval` calls.
- **Previous values:** Tracking the previous value of a prop or state variable.
- **Third-party libraries:** Passing a DOM node to a charting or mapping library.

### References

- React — useRef — https://18.react.dev/reference/react/useRef
- React — Referencing Values with Refs — https://18.react.dev/learn/referencing-values-with-refs

---

## 2. Ref Objects

### Definitions

**Core Definition:** A ref object is the plain JavaScript object returned by `useRef`, containing a single mutable `current` property.

**Technical Definition:** A ref object is a plain JavaScript object with the shape `{ current: T }`. The `current` property holds the value you passed as `initialValue`. React guarantees that the same ref object is returned on every render, so the `current` property persists. Unlike state, the ref object is not frozen or protected — you can freely mutate `current`. When you pass a ref object to a JSX element as the `ref` attribute, React sets `current` to the DOM node (or `null` when unmounted).

**Beginner-Friendly Explanation:** A ref object is a simple box with one slot called `current`. You can put anything in that slot and change it whenever you like.

### Purposes

- To provide a stable container for mutable values across renders.
- To hold a reference to a DOM node.
- To enable imperative operations on DOM elements.

### Syntax Rules and Structure

```jsx
const ref = useRef(initialValue);
// ref = { current: initialValue }
```

**Syntax Rules:**

1. A ref object always has exactly one property: `current`.
2. The same object is returned on every render.
3. `ref.current` is initially `initialValue`.
4. When passed to a JSX element, React sets `current` to the DOM node.

### Annotated Code Examples

```jsx
import { useRef } from 'react';

function Example() {
  const ref = useRef(0);
  // ref = { current: 0 }

  console.log(ref.current); // 0

  ref.current = 42;
  console.log(ref.current); // 42
}
```

**Expected Output:** The ref object holds the value 42.

**Why this output:** `ref.current` is a mutable property. Assigning to it changes the value in the box.

### Real-World Cases

- **Storing values:** Timer IDs, previous values, instance variables.
- **DOM references:** Holding DOM nodes for imperative operations.
- **Cross-render persistence:** Remembering values without re-rendering.

### References

- React — useRef: Returns — https://18.react.dev/reference/react/useRef#returns
- React — Referencing Values with Refs — https://18.react.dev/learn/referencing-values-with-refs

---

## 3. `current` Property

### Definitions

**Core Definition:** The `current` property is the single mutable property of a ref object that holds the ref's value.

**Technical Definition:** `ref.current` is the property you read and write to access the ref's value. It is initially set to the `initialValue` passed to `useRef`. When the ref is attached to a JSX element, React sets `current` to the corresponding DOM node. When the element is unmounted, React sets `current` back to `null`. You must not read or write `current` during rendering (except for initialization) because React does not track changes and the behaviour becomes unpredictable.

**Beginner-Friendly Explanation:** `current` is the slot in the ref box where the value lives. You read it to get the value and assign to it to change the value.

### Purposes

- To read the current value of the ref.
- To write a new value to the ref.
- To access the DOM node attached to the ref.

### Syntax Rules and Structure

```jsx
ref.current = newValue;  // Write
const value = ref.current; // Read
```

**Syntax Rules:**

1. Read and write `current` only in event handlers and effects.
2. Do not read or write `current` during rendering (except initialization).
3. React sets `current` to the DOM node when the element mounts and `null` when it unmounts.

### Annotated Code Examples

```jsx
import { useRef, useEffect } from 'react';

function Timer() {
  const intervalRef = useRef(null);

  useEffect(() => {
    intervalRef.current = setInterval(() => {
      console.log('Tick');
    }, 1000);

    return () => clearInterval(intervalRef.current);
  }, []);

  return <p>Timer running</p>;
}
```

**Expected Output:** "Tick" logs every second while mounted.

**Why this output:** `intervalRef.current` is written in the effect (a valid place to write) and read in the cleanup. The value persists between renders because the ref object is stable.

### Real-World Cases

- **Timer IDs:** Storing and retrieving `setInterval` IDs.
- **DOM nodes:** Accessing the underlying element for `focus()` or `scrollIntoView()`.
- **Previous values:** Storing the previous value of a prop or state variable.

### References

- React — useRef: Caveats — https://18.react.dev/reference/react/useRef#caveats

---

## 4. DOM Refs

### Definitions

**Core Definition:** A DOM ref is a ref object that points to a DOM element managed by React.

**Technical Definition:** To access a DOM node managed by React, you declare a ref with `useRef(null)`, then pass it as the `ref` attribute to a JSX tag: `<div ref={myRef}>`. React puts the DOM node into `myRef.current` after the element is created and sets it back to `null` when the element is removed. You can then call browser APIs like `focus()`, `scrollIntoView()`, and `getBoundingClientRect()` on the node.

**Beginner-Friendly Explanation:** A DOM ref lets you grab hold of an actual HTML element so you can do things like focus it or scroll to it.

### Purposes

- To focus an input or button programmatically.
- To scroll to an element.
- To measure an element's size and position.
- To pass a DOM node to a third-party library.

### Syntax Rules and Structure

```jsx
const myRef = useRef(null);
<div ref={myRef}>...</div>
// Access: myRef.current
```

**Syntax Rules:**

1. Initialise the ref with `null`.
2. Pass the ref to the `ref` attribute of a JSX element.
3. Access the DOM node via `ref.current` in event handlers or effects.
4. The node is `null` before mount and after unmount.

### Annotated Code Examples

```jsx
import { useRef } from 'react';

function CatFriends() {
  const firstCatRef = useRef(null);
  const secondCatRef = useRef(null);
  const thirdCatRef = useRef(null);

  function scrollToFirstCat() {
    firstCatRef.current.scrollIntoView({ behavior: 'smooth', block: 'nearest' });
  }

  return (
    <>
      <nav>
        <button onClick={scrollToFirstCat}>Tom</button>
      </nav>
      <div>
        <ul>
          <li ref={firstCatRef}><img src="/tom.jpg" alt="Tom" /></li>
          <li ref={secondCatRef}><img src="/maru.jpg" alt="Maru" /></li>
          <li ref={thirdCatRef}><img src="/jellylorum.jpg" alt="Jellylorum" /></li>
        </ul>
      </div>
    </>
  );
}
```

**Expected Output:** Clicking "Tom" scrolls the first cat image into view.

**Why this output:** Each `<li>` has a ref attached. React puts the corresponding DOM node into `firstCatRef.current`, and the event handler calls `scrollIntoView()` on it.

### Real-World Cases

- **Form focus:** Focusing the first invalid field on submit.
- **Carousels:** Scrolling to a specific slide.
- **Charts:** Passing a canvas or container to a charting library.
- **Video players:** Calling `play()` or `pause()` on a `<video>` element.

### References

- React — Manipulating the DOM with Refs — https://18.react.dev/learn/manipulating-the-dom-with-refs
- React — useRef — https://18.react.dev/reference/react/useRef

---

## 5. Mutable Values

### Definitions

**Core Definition:** Mutable values stored in refs are values that persist across renders without triggering re-renders.

**Technical Definition:** A ref can hold any JavaScript value: numbers, strings, objects, arrays, functions, or even other refs. Because mutating `ref.current` does not cause a re-render, refs are ideal for storing values that are not needed for rendering — timer IDs, previous values, DOM nodes, or instance-like variables. This makes refs the functional component equivalent of instance variables in class components.

**Beginner-Friendly Explanation:** A mutable value is something you can change whenever you want, and the component will not re-render when you change it.

### Purposes

- To store timer IDs for later cleanup.
- To track previous values of props or state.
- To store instance-like variables that should not trigger re-renders.
- To hold values that change frequently without affecting the UI.

### Syntax Rules and Structure

```jsx
const timerRef = useRef(null);
timerRef.current = setInterval(...);
```

**Syntax Rules:**

1. The initial value can be of any type.
2. Mutate `current` in event handlers or effects.
3. Do not read `current` during rendering.

### Annotated Code Examples

```jsx
import { useRef, useEffect } from 'react';

function TimerComponent() {
  const [count, setCount] = useState(0);
  const [isRunning, setIsRunning] = useState(false);
  const timerRef = useRef(null);
  const prevCountRef = useRef(0);

  useEffect(() => {
    prevCountRef.current = count;
  });

  const startTimer = () => {
    if (!isRunning) {
      setIsRunning(true);
      timerRef.current = setInterval(() => {
        setCount(c => c + 1);
      }, 100);
    }
  };

  const stopTimer = () => {
    if (timerRef.current) {
      clearInterval(timerRef.current);
      timerRef.current = null;
      setIsRunning(false);
    }
  };

  return (
    <div>
      <p>Current: {count}</p>
      <p>Previous: {prevCountRef.current}</p>
      <button onClick={startTimer}>Start</button>
      <button onClick={stopTimer}>Stop</button>
    </div>
  );
}
```

**Expected Output:** A counter that shows the current and previous count, with start and stop buttons.

**Why this output:** `timerRef.current` stores the interval ID. `prevCountRef.current` stores the previous count. Neither triggers re-renders when mutated.

### Real-World Cases

- **Timers:** Storing `setInterval` and `setTimeout` IDs.
- **Previous values:** Tracking the previous value of a prop or state.
- **Abort controllers:** Storing an `AbortController` for fetch cancellation.
- **Instance variables:** Replacing class instance fields in function components.

### References

- React — Referencing Values with Refs — https://18.react.dev/learn/referencing-values-with-refs
- Steve Kinney — useRef, Callback Refs, and Imperative Handles — https://stevekinney.com/courses/react-typescript/useref-and-imperative-handles

---

## 6. Persisting Values

### Definitions

**Core Definition:** Persisting values with refs means storing data that survives re-renders without causing them.

**Technical Definition:** Unlike regular variables, which are recreated on every render, refs are retained by React between re-renders. When you mutate `ref.current`, the value persists for the lifetime of the component. This makes refs perfect for storing data that is not needed for rendering but must survive across renders — timer IDs, previous values, or any mutable state that should not trigger a UI update.

**Beginner-Friendly Explanation:** A ref remembers its value between renders, just like state does. The difference is that changing a ref does not make the component re-render.

### Purposes

- To remember values between renders without re-rendering.
- To store data that is not part of the visual output.
- To avoid unnecessary re-renders for frequently changing values.

### Syntax Rules and Structure

```jsx
const ref = useRef(initialValue);
// ref.current persists across renders
```

**Syntax Rules:**

1. Refs are retained between re-renders.
2. Mutating `ref.current` does not trigger a re-render.
3. Use refs for values that do not affect rendering.

### Annotated Code Examples

```jsx
import { useRef } from 'react';

function Counter() {
  const countRef = useRef(0);

  function handleClick() {
    countRef.current += 1;
    alert(`Clicked ${countRef.current} times`);
  }

  return <button onClick={handleClick}>Click me</button>;
}
```

**Expected Output:** The alert shows an incrementing count each time the button is clicked.

**Why this output:** The ref object is the same across renders, so `countRef.current` persists. Mutating it does not cause a re-render.

### Real-World Cases

- **Click counters:** Tracking how many times a button was clicked without re-rendering.
- **Scroll position:** Remembering scroll position across renders.
- **Form state:** Storing non-reactive form data.

### References

- React — Referencing Values with Refs — https://18.react.dev/learn/referencing-values-with-refs

---

## 7. Ref Forwarding

### Definitions

**Core Definition:** Ref forwarding is the technique of passing a ref through a component to one of its child elements.

**Technical Definition:** By default, components do not expose their DOM nodes to parent components. Ref forwarding allows a component to take a `ref` it receives and pass it down to a child. In React 19 and later, `ref` is a regular prop — you can access it as `props.ref` without `forwardRef`. In React 18 and earlier, `forwardRef` is required: `const MyInput = forwardRef(function MyInput(props, ref) { return <input ref={ref} />; })`.

**Beginner-Friendly Explanation:** Ref forwarding lets a parent component reach through a child component to get a ref to a DOM element inside it.

### Purposes

- To allow parent components to access DOM nodes inside child components.
- To build reusable components that expose their underlying DOM node.
- To enable imperative operations on child elements.

### Syntax Rules and Structure

**React 19+ (ref as prop):**

```jsx
function MyInput({ ref, ...props }) {
  return <input ref={ref} {...props} />;
}
```

**React 18 and earlier (forwardRef):**

```jsx
const MyInput = forwardRef(function MyInput(props, ref) {
  return <input ref={ref} {...props} />;
});
```

**Syntax Rules:**

1. React 19+: `ref` is a regular prop.
2. React 18 and earlier: use `forwardRef` to receive `ref` as the second argument.
3. Forward the ref to a child element to expose its DOM node.

### Annotated Code Examples

```jsx
import { forwardRef } from 'react';

// React 18 and earlier
const MyInput = forwardRef(function MyInput(props, ref) {
  return <input {...props} ref={ref} />;
});

// Parent component
function Form() {
  const inputRef = useRef(null);

  return (
    <>
      <MyInput ref={inputRef} placeholder="Type here" />
      <button onClick={() => inputRef.current.focus()}>Focus</button>
    </>
  );
}
```

**Expected Output:** Clicking the button focuses the input inside `MyInput`.

**Why this output:** `forwardRef` passes the parent's ref through `MyInput` to the `<input>` element. The parent can then access the DOM node via `inputRef.current`.

### Real-World Cases

- **Design systems:** Buttons, inputs, and other primitives that expose their DOM node.
- **Focus management:** Focusing a child input from a parent.
- **Third-party integration:** Passing DOM nodes to libraries.

### References

- React — forwardRef — https://react.dev/reference/react/forwardRef
- Steve Kinney — useRef, Callback Refs, and Imperative Handles — https://stevekinney.com/courses/react-typescript/useref-and-imperative-handles

---

## 8. `forwardRef`

### Definitions

**Core Definition:** `forwardRef` is a React function that lets a component receive a ref from its parent and forward it to a child.

**Technical Definition:** `forwardRef(render)` accepts a render function and returns a component that can receive a `ref` prop. The render function receives `(props, ref)` as arguments. In React 19 and later, `ref` is a regular prop, and `forwardRef` is no longer necessary — the API will be deprecated in a future release.

**Beginner-Friendly Explanation:** `forwardRef` is how you make a component pass a ref through to one of its children.

### Purposes

- To expose a child component's DOM node to a parent.
- To build reusable components that accept refs.
- To enable imperative operations through component boundaries.

### Syntax Rules and Structure

```jsx
const MyComponent = forwardRef(function MyComponent(props, ref) {
  return <div ref={ref} {...props} />;
});
```

**Syntax Rules:**

1. `forwardRef` takes a render function `(props, ref) => JSX`.
2. The component returned by `forwardRef` accepts a `ref` prop.
3. React 19+ does not require `forwardRef`.

**Constraints and Limitations:**

- `forwardRef` will be deprecated in a future React release.
- Use `ref` as a prop in React 19+ instead.

### Annotated Code Examples

```jsx
import { forwardRef, useRef, useImperativeHandle } from 'react';

const MyInput = forwardRef(function MyInput(props, ref) {
  const inputRef = useRef(null);

  useImperativeHandle(ref, () => ({
    focus() {
      inputRef.current.focus();
    },
    scrollIntoView() {
      inputRef.current.scrollIntoView();
    },
  }), []);

  return <input {...props} ref={inputRef} />;
});
```

**Expected Output:** The parent can call `focus()` and `scrollIntoView()` on the ref.

**Why this output:** `forwardRef` allows `MyInput` to receive the parent's ref. `useImperativeHandle` customises what the ref exposes — only `focus` and `scrollIntoView`, not the entire DOM node.

### Real-World Cases

- **Design systems:** Input, button, and modal components.
- **Form libraries:** Exposing input refs for validation.
- **Animation libraries:** Exposing DOM nodes for animation targets.

### References

- React — forwardRef — https://react.dev/reference/react/forwardRef

---

## 9. `useImperativeHandle`

### Definitions

**Core Definition:** `useImperativeHandle` is a React Hook that lets you customise the ref handle exposed to parent components.

**Technical Definition:** `useImperativeHandle(ref, createHandle, dependencies?)` is called inside a component that uses `forwardRef`. It customises the value exposed to the parent's ref. The `createHandle` function returns the object that will be assigned to `ref.current`. This lets you expose only specific methods instead of the entire DOM node. `useImperativeHandle` returns `undefined`.

**Beginner-Friendly Explanation:** `useImperativeHandle` lets you decide exactly what a parent can do with your component's ref. Instead of exposing the whole DOM node, you can expose just the methods you want.

### Purposes

- To restrict what a parent can access through a ref.
- To expose a custom imperative API (focus, scroll, play, etc.).
- To hide implementation details from parent components.

### Syntax Rules and Structure

```jsx
useImperativeHandle(ref, () => ({
  method1() { ... },
  method2() { ... },
}), [dependencies]);
```

**Syntax Rules:**

1. Call `useImperativeHandle` at the top level of a component using `forwardRef`.
2. `createHandle` returns the object exposed as `ref.current`.
3. Include all reactive values in the dependency array.

### Annotated Code Examples

```jsx
import { forwardRef, useRef, useImperativeHandle } from 'react';

const MyInput = forwardRef(function MyInput(props, ref) {
  const inputRef = useRef(null);

  useImperativeHandle(ref, () => ({
    focus() {
      inputRef.current.focus();
    },
    scrollIntoView() {
      inputRef.current.scrollIntoView();
    },
  }), []);

  return <input {...props} ref={inputRef} />;
});
```

**Expected Output:** The parent can call `focus()` and `scrollIntoView()` but cannot access the `<input>` DOM node directly.

**Why this output:** `useImperativeHandle` overrides the default ref handle. Instead of the DOM node, the ref exposes only the methods defined in `createHandle`.

### Real-World Cases

- **Design systems:** Exposing a limited API for components.
- **Form libraries:** Exposing validation methods.
- **Media players:** Exposing `play()`, `pause()`, and `seek()`.

### References

- React — useImperativeHandle — https://18.react.dev/reference/react/useImperativeHandle

---

## 10. Callback Refs

### Definitions

**Core Definition:** A callback ref is a function passed as the `ref` prop instead of a ref object.

**Technical Definition:** Instead of passing a ref object to the `ref` attribute, you can pass a function. React calls this function with the DOM node when the element is mounted and with `null` when it is unmounted. In React 19, callback refs can return a cleanup function, which React calls when the element is removed. Callback refs are useful for running code when a ref is attached or detached, or for dynamic refs.

**Beginner-Friendly Explanation:** A callback ref is a function that React calls with the DOM element when it appears and with `null` when it disappears. It is useful when you need to do something when the element mounts or unmounts.

### Purposes

- To run code when a ref is attached or detached.
- To handle dynamic refs (refs that change over time).
- To perform setup and cleanup directly in the ref callback.
- To avoid the need for `useEffect` for ref-related side effects.

### Syntax Rules and Structure

```jsx
<div ref={(node) => { /* called with DOM node or null */ }} />
```

**React 19+ cleanup:**

```jsx
<div ref={(node) => {
  // setup
  return () => {
    // cleanup
  };
}} />
```

**Syntax Rules:**

1. Pass a function instead of a ref object.
2. React calls the function with the DOM node on mount and `null` on unmount.
3. In React 19+, return a cleanup function.

### Annotated Code Examples

```jsx
function CallbackRefExample() {
  const [height, setHeight] = useState(0);

  const measureRef = (node) => {
    if (node !== null) {
      setHeight(node.getBoundingClientRect().height);
    }
  };

  return (
    <div>
      <div ref={measureRef}>Measure me!</div>
      <p>Height: {height}px</p>
    </div>
  );
}
```

**Expected Output:** The height of the div is measured and displayed.

**Why this output:** The callback ref is called with the DOM node when the div mounts. It measures the height and updates state, triggering a re-render.

### Real-World Cases

- **Measuring elements:** Getting dimensions when an element mounts.
- **Dynamic lists:** Attaching refs to list items that change.
- **Third-party libraries:** Passing DOM nodes to libraries that need them immediately.

### References

- Steve Kinney — useRef, Callback Refs, and Imperative Handles — https://stevekinney.com/courses/react-typescript/useref-and-imperative-handles
- React — Manipulating the DOM with Refs — https://18.react.dev/learn/manipulating-the-dom-with-refs

---

## 11. Ref Cleanup

### Definitions

**Core Definition:** Ref cleanup is the process of cleaning up resources when a ref is detached from a DOM element.

**Technical Definition:** In React 19, callback refs can return a cleanup function. React calls this cleanup function when the element is removed from the DOM or when the ref changes. Before React 19, cleanup was typically handled in `useEffect` return functions or by checking for `null` in the callback ref.

**Beginner-Friendly Explanation:** Ref cleanup lets you undo whatever you set up when the ref was attached, like removing an event listener or cancelling a subscription.

### Purposes

- To clean up resources when a ref is detached.
- To prevent memory leaks from event listeners or subscriptions.
- To replace `useEffect` cleanup in ref-related scenarios.

### Syntax Rules and Structure

**React 19+:**

```jsx
<div ref={(node) => {
  // Setup
  return () => {
    // Cleanup
  };
}} />
```

**React 18 and earlier:**

```jsx
<div ref={(node) => {
  if (node) {
    // Setup
  } else {
    // Cleanup
  }
}} />
```

### Annotated Code Examples

```jsx
function ResizeObserverExample() {
  const observerRef = useRef(null);

  const measureRef = (node) => {
    if (node !== null) {
      const observer = new ResizeObserver((entries) => {
        console.log('Size changed:', entries[0].contentRect);
      });
      observer.observe(node);
      observerRef.current = observer;

      // React 19+ cleanup
      return () => {
        observer.disconnect();
      };
    }
  };

  return <div ref={measureRef}>Resize me!</div>;
}
```

**Expected Output:** The size changes are logged to the console.

**Why this output:** The callback ref sets up a `ResizeObserver` when the node mounts and returns a cleanup that disconnects it when the node unmounts.

### Real-World Cases

- **ResizeObserver:** Observing element size changes.
- **IntersectionObserver:** Observing element visibility.
- **Event listeners:** Attaching and removing listeners on DOM nodes.

### References

- Steve Kinney — useRef, Callback Refs, and Imperative Handles — https://stevekinney.com/courses/react-typescript/useref-and-imperative-handles

---

## 12. Refs vs State

### Definitions

**Core Definition:** Refs and state are both ways to store data across renders, but refs do not trigger re-renders while state does.

**Technical Definition:** `useRef(initialValue)` returns a mutable object `{ current: initialValue }` that persists across renders. Mutating `ref.current` does not cause a re-render. `useState(initialValue)` returns `[value, setValue]`. Calling `setValue` triggers a re-render. Use state for values that affect rendering; use refs for values that do not.

**Beginner-Friendly Explanation:** State is for things that appear on screen. Refs are for things that do not.

### Comparison Table

| Aspect | `useRef` | `useState` |
|--------|----------|------------|
| Return value | `{ current: initialValue }` | `[value, setValue]` |
| Triggers re-render | No | Yes |
| Mutable | Yes — modify `current` directly | No — use setter |
| Read during render | Avoid (except initialization) | Yes — safe |
| Use case | DOM nodes, timer IDs, non-render values | Values that affect the UI |

**Syntax Rules:**

1. Use state when the value is used in JSX.
2. Use refs when the value is only needed in event handlers or effects.
3. Never read `ref.current` during rendering.

### Annotated Code Examples

```jsx
// ✅ State: value affects rendering
const [count, setCount] = useState(0);
return <p>Count: {count}</p>;

// ✅ Ref: value does not affect rendering
const countRef = useRef(0);
function handleClick() {
  countRef.current += 1;
  alert(`Clicked ${countRef.current} times`);
}
return <button onClick={handleClick}>Click</button>;
```

**Expected Output:** The state version re-renders; the ref version does not.

**Why this output:** `setCount` triggers a re-render so the paragraph updates. Mutating `countRef.current` does not trigger a re-render, so the button stays the same.

### Real-World Cases

- **State:** Form inputs, toggles, counters displayed on screen.
- **Refs:** Timer IDs, DOM nodes, previous values, scroll positions.

### References

- React — Referencing Values with Refs: Differences from state — https://18.react.dev/learn/referencing-values-with-refs#differences-between-refs-and-state
- saschb2b/okf-bundles — Refs vs State — https://github.com/saschb2b/okf-bundles

---

## 13. Ref Best Practices

### Definitions

**Core Definition:** Ref best practices are guidelines for using refs safely, correctly, and maintainably.

**Technical Definition:** Best practices include: treating refs as an escape hatch (use sparingly), never reading or writing `ref.current` during rendering, initializing DOM refs with `null`, using TypeScript generics for DOM element types, preferring callback refs for dynamic nodes, using `useImperativeHandle` to expose limited APIs, and ensuring cleanup for ref-based resources.

**Beginner-Friendly Explanation:** These are the "do's and don'ts" that experienced React developers follow to keep their ref usage clean and bug-free.

### Best Practices Table

| Practice | Description |
|----------|-------------|
| **Treat refs as an escape hatch** | Use refs only when you need to step outside React. |
| **Don't read/write during render** | Only access `ref.current` in event handlers and effects. |
| **Initialise DOM refs with `null`** | `useRef<HTMLInputElement>(null)` |
| **Use TypeScript generics** | `useRef<HTMLDivElement>(null)` for type safety. |
| **Use callback refs for dynamic nodes** | When the node may change or you need setup/cleanup. |
| **Use `useImperativeHandle`** | Expose only the methods the parent needs. |
| **Clean up ref resources** | Disconnect observers, remove listeners, clear timers. |
| **Don't overuse refs** | If your logic relies heavily on refs, rethink the design. |

### Annotated Code Examples

**✅ Good Practices:**

```jsx
// TypeScript generic for DOM refs
const inputRef = useRef<HTMLInputElement>(null);

// Read in event handler, not during render
function handleClick() {
  inputRef.current?.focus();
}

// Cleanup in callback ref (React 19+)
<div ref={(node) => {
  if (node) { /* setup */ }
  return () => { /* cleanup */ };
}} />
```

**❌ Bad Practices:**

```jsx
// ❌ Reading ref during render
const inputRef = useRef(null);
console.log(inputRef.current); // Unpredictable!

// ❌ Using refs for values that should be state
const countRef = useRef(0);
return <p>Count: {countRef.current}</p>; // Won't update!
```

**Expected Output:** The good practices produce predictable, performant code.

**Why this output:** Reading refs during render is unreliable because React does not track changes. Using refs for rendering values means the UI will not update when the value changes.

### Real-World Cases

- **All React applications:** These practices apply universally.
- **Code reviews:** Best practices are enforced through ESLint and review checklists.
- **TypeScript projects:** Generics ensure type safety for DOM refs.

### References

- React — Referencing Values with Refs: Best practices — https://18.react.dev/learn/referencing-values-with-refs#best-practices-for-refs
- GitHub — Ref and DOM Imperative Rules (voku/agent-skills) — https://github.com/voku/agent-skills
- Steve Kinney — useRef, Callback Refs, and Imperative Handles — https://stevekinney.com/courses/react-typescript/useref-and-imperative-handles

---

## References

- React — useRef — https://18.react.dev/reference/react/useRef
- React — Referencing Values with Refs — https://18.react.dev/learn/referencing-values-with-refs
- React — Manipulating the DOM with Refs — https://18.react.dev/learn/manipulating-the-dom-with-refs
- React — forwardRef — https://react.dev/reference/react/forwardRef
- React — useImperativeHandle — https://18.react.dev/reference/react/useImperativeHandle
- React — Escape Hatches — https://uk.react.dev/learn/escape-hatches
- Steve Kinney — useRef, Callback Refs, and Imperative Handles — https://stevekinney.com/courses/react-typescript/useref-and-imperative-handles
- CoreUI — How to type refs in React with TypeScript — https://coreui.io
- Refine — Understanding the React useRef Hook — https://refine.dev
- saschb2b/okf-bundles — Referencing Values with Refs — https://github.com/saschb2b/okf-bundles
- voku/agent-skills — TypeScript React Patterns: Ref and DOM Imperative — https://github.com/voku/agent-skills
- GitHub — React Syntax Refs — https://github.com/ReactTraining/react-syntax-refs
- Kinsta — Understanding the useRef Hook in React — https://kinsta.com
- React — Refs and the DOM (Legacy) — https://mintlify.wiki/facebook/react/guides/refs-and-the-dom
- Steve Kinney — React Performance: Context API Pitfalls — https://stevekinney.com/courses/react-performance/context-api-performance-pitfalls