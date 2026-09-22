# React Event Handling — Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**

React event handling is the mechanism by which React components respond to user interactions—such as clicks, typing, and form submissions—by attaching handler functions to JSX elements.

**Technical Definition**

React provides a cross-browser event system that wraps native browser events in `SyntheticEvent` objects, normalizing event behavior across browsers. Event handlers are JavaScript functions attached to JSX elements via `on*` props (e.g., `onClick`, `onChange`, `onSubmit`). When an event fires, React calls the corresponding handler with a `SyntheticEvent` instance, which provides the same interface as the native event (including `stopPropagation()` and `preventDefault()`) but works consistently across all browsers .

**Beginner-Friendly Explanation**

React event handling is how you make your app respond to what users do. When someone clicks a button, types in an input, or submits a form, you want something to happen. You write a function (the "handler") and tell React "when this event happens, run this function." It's like setting up a doorbell—when someone presses it, the bell rings.

### Key Characteristics

- **Synthetic Events**: React wraps native browser events in `SyntheticEvent` objects, providing cross-browser consistency .
- **CamelCase Naming**: Event props use camelCase (`onClick`, `onChange`, `onSubmit`) rather than lowercase HTML names (`onclick`, `onchange`) .
- **Function References**: You pass the function itself, not a call to it: `onClick={handleClick}`, not `onClick={handleClick()}` .
- **Event Propagation**: Events bubble up through the component tree unless stopped with `e.stopPropagation()` .
- **No Pooling in React 17+**: `e.persist()` is no longer needed because `SyntheticEvent` objects are no longer pooled .

### Prerequisites

Before studying React Event Handling, learners should be comfortable with:

- **JavaScript Functions**: Declaring functions, arrow functions, and function references.
- **JSX**: How to add attributes and expressions to JSX elements.
- **React Components**: Functional components and props.
- **State**: Using `useState` to store and update values.

### Related Programming Areas

- **State Management**: Event handlers often update state, which triggers re-renders.
- **Forms**: Controlled components rely on `onChange` and `onSubmit` events.
- **Accessibility**: Proper use of semantic HTML elements (e.g., `<button>` vs. `<div>`) enables keyboard navigation .
- **Effect Events**: React's `useEffectEvent` separates event logic from reactive Effect logic .

### Core Concepts / Features

- Click Events
- Change Events
- Submit Events
- Keyboard Events
- Mouse Events
- Event Handler Functions
- Event Objects
- Passing Arguments to Handlers

---

## 1. Click Events

### Definitions

**Core Definition**

Click events are user interactions triggered when a pointing device (mouse, touchpad, touchscreen) is pressed and released on an element.

**Technical Definition**

React's `onClick` prop attaches a handler function to an element that fires when the element is clicked. The handler receives a `SyntheticEvent` object containing information about the click, including the target element and cursor coordinates. React uses event delegation, attaching a single listener at the root and dispatching to the appropriate handler .

**Beginner-Friendly Explanation**

A click event is what happens when you click something. You attach a function to a button's `onClick` prop, and React runs that function whenever the button is clicked.

### Purposes

- To respond to button presses and interactive element clicks.
- To trigger state updates based on user actions.
- To navigate between views or open dialogs.
- To submit data or perform actions in response to clicks.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
<button onClick={handleClick}>Click Me</button>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `onClick` | The event prop (camelCase). |
| `handleClick` | A reference to the handler function (no parentheses). |
| `{handleClick}` | JSX expression passing the function. |

**Syntax Rules**

1. **Pass the function, don't call it**: `onClick={handleClick}` not `onClick={handleClick()}` .
2. **Use semantic elements**: Use `<button onClick={...}>` rather than `<div onClick={...}>` for built-in keyboard accessibility .
3. **Inline handlers**: For short logic, use an arrow function: `onClick={() => alert('Hi')}`.

**Constraints and Limitations**

- Calling the handler directly in JSX (`onClick={handleClick()}`) runs it during render, not on click .
- `onClick` on non-interactive elements (like `<div>`) lacks keyboard accessibility.

### Annotated Complete Code Examples

**Example 1: Basic Click Handler**

```jsx
function Button() {
  function handleClick() {
    alert('You clicked me!');
  }

  return (
    <button onClick={handleClick}>
      Click me
    </button>
  );
}
```

**Expected Output**

```
[Button with text "Click me"]
When clicked: Alert displays "You clicked me!"
```

**Why This Output Occurs**

`handleClick` is defined inside the component. `onClick={handleClick}` passes the function reference to React. React stores it and calls it only when the button is clicked .

**Example 2: Inline Click Handler**

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  );
}
```

**Expected Output (after two clicks)**

```
Count: 2
```

**Why This Output Occurs**

The arrow function `() => setCount(count + 1)` is passed to `onClick`. React calls it on each click, updating state and triggering a re-render.

### Real-World Cases

**Case: Navigation Menu**

A `Navbar` component uses `onClick` handlers on navigation links to change the active route without a full page reload.

**Case: Modal Trigger**

A "Delete" button uses `onClick` to open a confirmation modal, which itself has buttons with `onClick` handlers for confirming or canceling.

### References

- React — Responding to Events - https://react.dev/learn/responding-to-events

---

## 2. Change Events

### Definitions

**Core Definition**

Change events fire when the value of an input, textarea, or select element changes, typically on every keystroke in React.

**Technical Definition**

React's `onChange` prop attaches a handler that fires immediately when an input's value changes by user interaction. For historical reasons, React's `onChange` behaves like the browser's `input` event, firing on every keystroke rather than on blur . This is essential for controlled components, where the input's value is driven by React state.

**Beginner-Friendly Explanation**

A change event is what happens when you type into a text box. Every time you press a key, React fires the `onChange` handler, letting you update your state with the new value.

### Purposes

- To update state with the current input value.
- To implement controlled form components.
- To validate input as the user types.
- To enable real-time search or filtering.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
<input value={value} onChange={e => setValue(e.target.value)} />
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `value={value}` | Controlled value from state. |
| `onChange={...}` | Handler fires on every keystroke. |
| `e.target.value` | The current value from the input element. |

**Syntax Rules**

1. **Controlled inputs require `onChange`**: If you pass `value` without `onChange`, the input becomes read-only .
2. **Use `e.target.value`**: Access the new value from the event object.
3. **React's `onChange` ≈ native `onInput`**: In React, `onChange` fires on every keystroke; native `onChange` fires on blur .

**Constraints and Limitations**

- Missing `onChange` with `value` causes a read-only field warning .
- Updating state asynchronously in `onChange` can cause the caret to jump .

### Annotated Complete Code Examples

**Example 1: Controlled Input**

```jsx
function NameInput() {
  const [name, setName] = useState('');

  return (
    <div>
      <input
        value={name}
        onChange={e => setName(e.target.value)}
        placeholder="Enter your name"
      />
      <p>Hello, {name || 'stranger'}!</p>
    </div>
  );
}
```

**Expected Output (after typing "Alice")**

```
Hello, Alice!
```

**Why This Output Occurs**

`value={name}` binds the input to state. `onChange` fires on each keystroke, calling `setName` with the new value. React re-renders with the updated `name`, which is displayed in the paragraph .

**Example 2: Textarea (Controlled)**

```jsx
function PostEditor() {
  const [content, setContent] = useState('');

  return (
    <textarea
      value={content}
      onChange={e => setContent(e.target.value)}
      placeholder="Write your post..."
    />
  );
}
```

**Explanation**

Textareas work the same way as inputs. The `value` prop controls the content, and `onChange` updates state on each keystroke. Without `onChange`, typing would be impossible .

### Real-World Cases

**Case: Search-as-You-Type**

A search box uses `onChange` to filter a list of results in real time. Each keystroke updates the search term state, and the filtered list is derived during render.

**Case: Form Validation**

A form validates email format on every `onChange`, displaying an error message immediately when the input is invalid.

### References

- React — `<textarea>` Reference (onChange) - https://18.react.dev/reference/react-dom/components/textarea
- Stack Overflow — React onChange vs onInput - https://stackoverflow.com/revisions/befe1f8d-849f-4bce-b80c-3e9b25b483e4/view-source

---

## 3. Submit Events

### Definitions

**Core Definition**

Submit events fire when a form is submitted, either by clicking a submit button or pressing Enter in a form field.

**Technical Definition**

React's `onSubmit` prop attaches a handler to a `<form>` element that fires when the form is submitted. The handler receives a `SyntheticEvent` and must typically call `e.preventDefault()` to stop the browser's default behavior of reloading the page . React 19 also supports passing a function to the `action` prop for form submission with automatic state handling .

**Beginner-Friendly Explanation**

A submit event is what happens when you hit the "Submit" button on a form. You write a handler that reads the form data, and you usually tell the browser "don't reload the page—I'll handle this myself."

### Purposes

- To process form data when the user submits.
- To validate all fields before submission.
- To send data to a server via fetch or API call.
- To reset the form after successful submission.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
<form onSubmit={handleSubmit}>
  <input name="email" />
  <button type="submit">Submit</button>
</form>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `onSubmit` | Form event prop. |
| `handleSubmit` | Handler receiving the event object. |
| `e.preventDefault()` | Stops page reload. |

**Syntax Rules**

1. **Always call `e.preventDefault()`**: Otherwise, the browser reloads the page .
2. **Use `type="submit"`**: Buttons inside forms submit by default; use `type="button"` for non-submit buttons.
3. **React 19 `action` prop**: Pass a function to `<form action={fn}>` for automatic form handling .

**Constraints and Limitations**

- Without `preventDefault()`, the page reloads and state is lost.
- Form data can be accessed via `new FormData(e.target)` or controlled state.

### Annotated Complete Code Examples

**Example 1: Basic Submit Handler**

```jsx
function LoginForm() {
  const [email, setEmail] = useState('');

  function handleSubmit(e) {
    e.preventDefault(); // Stop page reload
    alert(`Logging in with: ${email}`);
  }

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={email}
        onChange={e => setEmail(e.target.value)}
        placeholder="Email"
      />
      <button type="submit">Log In</button>
    </form>
  );
}
```

**Expected Output**

```
[Input field] [Log In button]
When submitted: Alert displays "Logging in with: [email]"
```

**Why This Output Occurs**

`onSubmit` fires when the form is submitted. `e.preventDefault()` stops the browser's default page reload. The handler reads the current `email` state and displays it .

**Example 2: React 19 `action` Prop**

```jsx
function Search() {
  function search(formData) {
    const query = formData.get('query');
    alert(`You searched for '${query}'`);
  }

  return (
    <form action={search}>
      <input name="query" />
      <button type="submit">Search</button>
    </form>
  );
}
```

**Explanation**

In React 19, passing a function to the `action` prop handles form submission automatically. The function receives a `FormData` object with the form's field values. No `preventDefault()` is needed .

### Real-World Cases

**Case: Login Form**

A login form uses `onSubmit` to validate credentials, call an authentication API, and redirect on success—all without a page reload.

**Case: Multi-Step Wizard**

A wizard form uses `onSubmit` on each step to validate and save data before advancing to the next step.

### References

- React — `<form>` Reference - https://zh-hant.react.dev/reference/react-dom/components/form
- React — Responding to Events (Preventing default behavior) - https://react.dev/learn/responding-to-events

---

## 4. Keyboard Events

### Definitions

**Core Definition**

Keyboard events fire when a user presses, holds, or releases a key on the keyboard while an element has focus.

**Technical Definition**

React provides `onKeyDown`, `onKeyPress` (deprecated), and `onKeyUp` props that fire during keyboard interaction. The `SyntheticEvent` includes properties like `key`, `code`, `altKey`, `ctrlKey`, `shiftKey`, and `metaKey` . Keyboard events are commonly used for shortcuts, accessibility features, and form navigation.

**Beginner-Friendly Explanation**

Keyboard events are what happen when you press a key. You can make your app respond to specific keys—like pressing Enter to submit a form or Escape to close a modal.

### Purposes

- To implement keyboard shortcuts (e.g., Ctrl+S to save).
- To handle Enter key for form submission or navigation.
- To support accessibility for keyboard-only users.
- To detect modifier keys (Shift, Ctrl, Alt, Meta).

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
<input onKeyDown={e => {
  if (e.key === 'Enter') { /* ... */ }
}} />
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `onKeyDown` | Fires when key is pressed down. |
| `e.key` | The value of the key pressed (e.g., 'Enter', 'Escape'). |
| `e.ctrlKey` | Boolean indicating Ctrl was held. |

**Syntax Rules**

1. **`e.key` gives the key value**: 'Enter', 'Escape', 'a', etc. .
2. **Modifier keys**: Check `e.ctrlKey`, `e.shiftKey`, `e.altKey`, `e.metaKey` for combinations.
3. **`onKeyPress` is deprecated**: Use `onKeyDown` or `onKeyUp` instead.

**Constraints and Limitations**

- Keyboard events only fire when the element has focus.
- `onKeyPress` is deprecated and should be avoided.
- Browser default behavior may interfere (e.g., Enter submits forms).

### Annotated Complete Code Examples

**Example 1: Enter Key Handler**

```jsx
function SearchBox() {
  const [query, setQuery] = useState('');

  function handleKeyDown(e) {
    if (e.key === 'Enter') {
      alert(`Searching for: ${query}`);
    }
  }

  return (
    <input
      value={query}
      onChange={e => setQuery(e.target.value)}
      onKeyDown={handleKeyDown}
      placeholder="Search..."
    />
  );
}
```

**Expected Output**

```
When typing and pressing Enter: Alert displays "Searching for: [query]"
```

**Why This Output Occurs**

`onKeyDown` fires on every key press. The handler checks `e.key === 'Enter'` and runs the search logic only when Enter is pressed .

**Example 2: Keyboard Shortcut**

```jsx
function Editor() {
  function handleKeyDown(e) {
    if (e.ctrlKey && e.key === 's') {
      e.preventDefault(); // Stop browser save dialog
      alert('Saved!');
    }
  }

  return (
    <textarea onKeyDown={handleKeyDown} placeholder="Type here..." />
  );
}
```

**Explanation**

`e.ctrlKey` is `true` when Ctrl is held. `e.key === 's'` checks for the S key. `e.preventDefault()` stops the browser's default save dialog .

### Real-World Cases

**Case: Modal Escape Key**

A modal component listens for `Escape` key presses and calls `onClose` to dismiss the modal.

**Case: Keyboard Navigation**

A dropdown menu uses arrow keys (`ArrowDown`, `ArrowUp`) to navigate options and Enter to select.

### References

- React — SyntheticEvent (Keyboard Events) - https://ru.react.js.org/docs/events.html
- MDN — Event handling (overview) - http://developer.typescripts.org/en-US/docs/Web/Events/Event_handlers

---

## 5. Mouse Events

### Definitions

**Core Definition**

Mouse events fire when the user interacts with an element using a pointing device (mouse, trackpad, touchscreen), including clicks, hovers, and drags.

**Technical Definition**

React provides mouse event props including `onClick`, `onDoubleClick`, `onMouseDown`, `onMouseUp`, `onMouseEnter`, `onMouseLeave`, `onMouseMove`, and `onContextMenu` . The `SyntheticEvent` includes properties like `clientX`, `clientY`, `pageX`, `pageY`, `button`, `ctrlKey`, and `shiftKey`.

**Beginner-Friendly Explanation**

Mouse events are everything that happens with your mouse: clicking, double-clicking, hovering, moving, and dragging. You can make elements respond when the mouse enters, leaves, or moves over them.

### Purposes

- To detect hover states for tooltips or visual effects.
- To handle double-clicks for special actions.
- To track mouse position for drag-and-drop or drawing.
- To detect right-clicks for context menus.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
<div 
  onMouseEnter={() => setHover(true)}
  onMouseLeave={() => setHover(false)}
>
  Content
</div>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `onMouseEnter` | Fires when mouse enters the element. |
| `onMouseLeave` | Fires when mouse leaves the element. |
| `onMouseMove` | Fires as mouse moves over the element. |
| `clientX/clientY` | Mouse coordinates relative to viewport. |

**Syntax Rules**

1. **`onMouseEnter`/`onMouseLeave` do not bubble**: Unlike `onMouseOver`/`onMouseOut` .
2. **`clientX/clientY`**: Coordinates relative to the browser viewport.
3. **`pageX/pageY`**: Coordinates relative to the document.

**Constraints and Limitations**

- `onMouseEnter`/`onMouseLeave` are not propagated to parent elements .
- Touch devices may not fire mouse events consistently.

### Annotated Complete Code Examples

**Example 1: Hover Effect**

```jsx
function HoverBox() {
  const [isHovered, setIsHovered] = useState(false);

  return (
    <div
      onMouseEnter={() => setIsHovered(true)}
      onMouseLeave={() => setIsHovered(false)}
      style={{
        backgroundColor: isHovered ? 'lightblue' : 'lightgray',
        padding: '20px'
      }}
    >
      {isHovered ? 'Hovering!' : 'Hover over me'}
    </div>
  );
}
```

**Expected Output**

```
[Gray box with "Hover over me"]
When hovered: Blue box with "Hovering!"
```

**Why This Output Occurs**

`onMouseEnter` sets `isHovered` to `true`, changing the background color and text. `onMouseLeave` resets it .

**Example 2: Mouse Position Tracking**

```jsx
function MouseTracker() {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  function handleMouseMove(e) {
    setPosition({ x: e.clientX, y: e.clientY });
  }

  return (
    <div onMouseMove={handleMouseMove} style={{ height: '200px', border: '1px solid black' }}>
      Mouse position: ({position.x}, {position.y})
    </div>
  );
}
```

**Explanation**

`onMouseMove` fires continuously as the mouse moves. `e.clientX` and `e.clientY` provide coordinates relative to the viewport.

### Real-World Cases

**Case: Tooltip on Hover**

A button with `onMouseEnter` shows a tooltip; `onMouseLeave` hides it.

**Case: Drag and Drop**

A draggable element uses `onMouseDown` to start dragging, `onMouseMove` to track position, and `onMouseUp` to drop.

### References

- React — SyntheticEvent (Mouse Events) - https://ru.react.js.org/docs/events.html
- Steve Kinney — DOM and React Event Types - https://stevekinney.com/courses/react-typescript/typing-dom-and-react-events

---

## 6. Event Handler Functions

### Definitions

**Core Definition**

Event handler functions are JavaScript functions that React calls when a specific event occurs on an element.

**Technical Definition**

An event handler is a function passed as a prop to a JSX element. React stores the function reference and invokes it with a `SyntheticEvent` when the event fires. Handlers can be defined inside the component (closure over props/state), passed as props from parents, or defined inline in JSX .

**Beginner-Friendly Explanation**

An event handler is the function that says "when this happens, do that." You write the function, give it to React, and React runs it at the right time.

### Purposes

- To define the behavior that occurs in response to an event.
- To update state based on user interaction.
- To communicate with parent components via callback props.
- To encapsulate event logic within the component.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
// Named handler
function handleClick() { /* ... */ }
<button onClick={handleClick}>Click</button>

// Inline arrow function
<button onClick={() => { /* ... */ }}>Click</button>

// Handler passed as prop
function Child({ onAction }) {
  return <button onClick={onAction}>Click</button>;
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `handleClick` | Named function (convention: `handle` + event). |
| `() => {}` | Inline arrow function. |
| `onAction` | Handler received as prop. |

**Syntax Rules**

1. **Pass, don't call**: `onClick={handleClick}` not `onClick={handleClick()}` .
2. **Naming convention**: `handle` + event name (e.g., `handleClick`, `handleSubmit`).
3. **Prop naming**: `on` + event/action (e.g., `onClick`, `onPlayMovie`) .

**Constraints and Limitations**

- Inline arrow functions create new references on every render, potentially causing performance issues in memoized components.
- Handlers defined in class components may need binding.

### Annotated Complete Code Examples

**Example 1: Named vs. Inline Handlers**

```jsx
function AlertButton({ message, children }) {
  // Named handler
  function handleClick() {
    alert(message);
  }

  return (
    <button onClick={handleClick}>
      {children}
    </button>
  );
}

function InlineButton({ message, children }) {
  // Inline handler
  return (
    <button onClick={() => alert(message)}>
      {children}
    </button>
  );
}
```

**Explanation**

Both approaches work. Named handlers are better for complex logic and reusability. Inline handlers are convenient for short, one-off logic .

**Example 2: Passing Handlers as Props**

```jsx
function Button({ onClick, children }) {
  return <button onClick={onClick}>{children}</button>;
}

function Toolbar() {
  function handlePlay() {
    alert('Playing!');
  }

  return (
    <Button onClick={handlePlay}>
      Play
    </Button>
  );
}
```

**Explanation**

The parent `Toolbar` defines the handler and passes it to `Button` as an `onClick` prop. The `Button` component forwards it to the native `<button>`. This pattern allows parents to customize child behavior .

### Real-World Cases

**Case: Design System Buttons**

A design system's `Button` component accepts an `onClick` prop but doesn't define the behavior. Each usage provides its own handler for its specific action.

**Case: Form Components**

A `Form` component defines `handleSubmit` internally, which validates and calls an `onSubmit` prop passed by the parent.

### References

- React — Responding to Events - https://react.dev/learn/responding-to-events
- React — Passing handlers as props - https://react.dev/learn/responding-to-events

---

## 7. Event Objects

### Definitions

**Core Definition**

Event objects are `SyntheticEvent` instances passed to event handlers, containing information about the event and methods for controlling it.

**Technical Definition**

React's `SyntheticEvent` is a cross-browser wrapper around the native browser event. It provides the same interface as the native event, including `target`, `currentTarget`, `type`, `preventDefault()`, and `stopPropagation()`. The native event is accessible via `e.nativeEvent` .

**Beginner-Friendly Explanation**

The event object is a package of information about what just happened. It tells you what element was clicked, what key was pressed, and lets you control how the event propagates.

### Purposes

- To access information about the event (target, key, coordinates).
- To prevent default browser behavior.
- To stop event propagation.
- To access the native browser event when needed.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
function handleClick(e) {
  console.log(e.target);       // The element that triggered the event
  console.log(e.currentTarget); // The element with the handler
  e.preventDefault();          // Stop default behavior
  e.stopPropagation();         // Stop bubbling
}
```

**Component Breakdown**

| Property/Method | Description |
|-----------------|-------------|
| `e.target` | The element where the event originated. |
| `e.currentTarget` | The element with the event handler attached. |
| `e.preventDefault()` | Stops the browser's default action. |
| `e.stopPropagation()` | Stops the event from bubbling. |
| `e.nativeEvent` | The underlying browser event. |

**Syntax Rules**

1. **`e.target` vs `e.currentTarget`**: `target` is where the event happened; `currentTarget` is where the handler is attached .
2. **`e.preventDefault()` vs `e.stopPropagation()`**: They are unrelated—one stops default behavior, the other stops bubbling .
3. **No `e.persist()` needed**: Since React 17, `SyntheticEvent` is no longer pooled .

**Constraints and Limitations**

- `e.target` can be a child element if the event originates from a nested element.
- `e.preventDefault()` only works on events with default behavior (e.g., links, form submission).

### Annotated Complete Code Examples

**Example 1: Target vs. CurrentTarget**

```jsx
function NestedButton() {
  function handleClick(e) {
    console.log('Target:', e.target.tagName);       // SPAN (where clicked)
    console.log('CurrentTarget:', e.currentTarget.tagName); // BUTTON (handler)
  }

  return (
    <button onClick={handleClick}>
      <span>Click the text</span>
    </button>
  );
}
```

**Expected Output**

```
Target: SPAN
CurrentTarget: BUTTON
```

**Why This Output Occurs**

`e.target` is the deepest element clicked (the `<span>`). `e.currentTarget` is the element with the handler (the `<button>`) .

**Example 2: Preventing Default and Stopping Propagation**

```jsx
function LinkWrapper() {
  function handleWrapperClick() {
    alert('Wrapper clicked!');
  }

  function handleLinkClick(e) {
    e.preventDefault();      // Don't navigate
    e.stopPropagation();     // Don't bubble to wrapper
    alert('Link clicked!');
  }

  return (
    <div onClick={handleWrapperClick}>
      <a href="/" onClick={handleLinkClick}>Click me</a>
    </div>
  );
}
```

**Expected Output**

```
When clicking the link: Only "Link clicked!" alert appears.
```

**Why This Output Occurs**

`e.preventDefault()` stops the link navigation. `e.stopPropagation()` stops the event from reaching the wrapper's `onClick` handler .

### Real-World Cases

**Case: Analytics Tracking**

A top-level `onClickCapture` handler logs every click for analytics, using `e.target` to identify what was clicked, even if child handlers stop propagation .

**Case: Form Validation**

A submit handler uses `e.preventDefault()` to stop page reload and then validates form data before processing.

### References

- React — SyntheticEvent - https://ru.react.js.org/docs/events.html
- React — Responding to Events (Event propagation) - https://react.dev/learn/responding-to-events

---

## 8. Passing Arguments to Handlers

### Definitions

**Core Definition**

Passing arguments to handlers is the technique of providing extra data to an event handler beyond the event object itself.

**Technical Definition**

Because event handlers receive only the event object as an argument, passing additional data requires wrapping the handler in an arrow function that closes over the desired values. This allows handlers to receive both the event and custom parameters .

**Beginner-Friendly Explanation**

Sometimes you need to tell your handler more than just "a click happened"—you need to say "a click happened on item 5." You do this by wrapping the handler in an arrow function that passes the extra information.

### Purposes

- To identify which item in a list was clicked.
- To pass data to handlers in loops.
- To customize handler behavior per usage.
- To pass bound arguments to handlers.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
// Without arguments (receives event)
<button onClick={handleClick}>Click</button>

// With arguments (wrapped in arrow function)
<button onClick={() => handleClick(id)}>Click</button>

// With both event and arguments
<button onClick={(e) => handleClick(id, e)}>Click</button>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `() => handleClick(id)` | Arrow function that calls handler with argument. |
| `(e) => handleClick(id, e)` | Receives event and passes it along. |

**Syntax Rules**

1. **Wrap in arrow function**: `onClick={() => handleClick(arg)}` .
2. **Do not call directly**: `onClick={handleClick(arg)}` runs during render .
3. **Access event if needed**: Include `e` as a parameter in the arrow function.

**Constraints and Limitations**

- Inline arrow functions create new references on every render.
- For lists, use `key` and pass the item's identifier, not the entire item, when possible.

### Annotated Complete Code Examples

**Example 1: Passing an ID to a Handler**

```jsx
function ItemList({ items }) {
  function handleDelete(id) {
    alert(`Deleting item ${id}`);
  }

  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>
          {item.name}
          <button onClick={() => handleDelete(item.id)}>
            Delete
          </button>
        </li>
      ))}
    </ul>
  );
}
```

**Expected Output**

```
[Item A] [Delete] [Item B] [Delete]
Clicking Delete on Item A: Alert displays "Deleting item 1"
```

**Why This Output Occurs**

The arrow function `() => handleDelete(item.id)` captures the current `item.id` in its closure. Each button's handler calls `handleDelete` with the correct ID .

**Example 2: Passing Event and Argument**

```jsx
function Form() {
  function handleChange(fieldName, e) {
    console.log(`Field ${fieldName} changed to: ${e.target.value}`);
  }

  return (
    <div>
      <input onChange={e => handleChange('email', e)} />
      <input onChange={e => handleChange('password', e)} />
    </div>
  );
}
```

**Explanation**

The arrow function receives the event `e` and passes it along with the field name. This allows a single handler to handle multiple fields.

### Real-World Cases

**Case: Dynamic Lists**

A todo list passes each todo's ID to the toggle handler: `onClick={() => toggleTodo(todo.id)}`. This lets one handler manage all items.

**Case: Form Field Handler**

A form uses a single `handleChange(field, e)` function for all fields, passing the field name to identify which state to update.

### References

- React — Responding to Events (Passing arguments) - https://react.dev/learn/responding-to-events
- MDN — addEventListener (Closures for data passing) - https://mdn.org.cn/en-US/docs/Web/API/EventTarget/addEventListener

---

## References

- React Official Documentation — Responding to Events - https://react.dev/learn/responding-to-events
- React Official Documentation — SyntheticEvent - https://ru.react.js.org/docs/events.html
- React Official Documentation — `<form>` Reference - https://zh-hant.react.dev/reference/react-dom/components/form
- React Official Documentation — `<textarea>` Reference - https://18.react.dev/reference/react-dom/components/textarea
- React Official Documentation — Separating Events from Effects - https://pt-br.react.dev/learn/separating-events-from-effects
- React Official Documentation — Adding Interactivity - https://18.react.dev/learn/adding-interactivity
- React Legacy Documentation — SyntheticEvent (v17 pooling note) - https://fr.legacy.reactjs.org/docs/events.html
- Steve Kinney — DOM and React Event Types - https://stevekinney.com/courses/react-typescript/typing-dom-and-react-events
- Stack Overflow — React onChange vs onInput - https://stackoverflow.com/revisions/befe1f8d-849f-4bce-b80c-3e9b25b483e4/view-source
- MDN — Event handling (overview) - http://developer.typescripts.org/en-US/docs/Web/Events/Event_handlers
- MDN — EventTarget.addEventListener() - https://mdn.org.cn/en-US/docs/Web/API/EventTarget/addEventListener
- GitHub — Store Event Handlers in Refs (Best Practices) - https://github.com/sickn33/agentic-awesome-skills/blob/main/skills/react-best-practices/rules/advanced-event-handler-refs.md