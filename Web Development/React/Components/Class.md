# React Class Components — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A class component is an ES6 JavaScript class that extends `React.Component` (or `React.PureComponent`) and implements a `render()` method to define its UI.

**Technical Definition:** Class components are the original way of defining stateful React components. They extend the base `Component` class, which provides lifecycle methods, `this.state`, `this.props`, and `this.setState()`. Class components are still fully supported by React, but the React team recommends using function components with Hooks for all new code. Class components are defined using ES6 class syntax: `class MyComponent extends React.Component { render() { return <JSX />; } }`.

**Beginner-Friendly Explanation:** A class component is like a blueprint written as a JavaScript class. It has special methods that React calls at different times in the component's life (when it appears, when it updates, and when it disappears). It was the standard way to write React components before Hooks were introduced in 2018.

### Key Characteristics

- **ES6 class-based:** Uses JavaScript class syntax with `extends React.Component`.
- **Lifecycle methods:** Provides hooks into mounting, updating, and unmounting phases.
- **`this` context:** Uses `this.props`, `this.state`, and `this.setState()`.
- **Stateful by default:** Can manage local state without additional APIs.
- **Still supported but not recommended:** React still supports class components, but function components are preferred for new code.
- **Required for error boundaries:** Only class components can be React error boundaries.

### Prerequisites

- JavaScript ES6 classes (`class`, `extends`, `constructor`, `this`).
- Understanding of JSX syntax.
- Familiarity with React props and state concepts.
- Understanding of `this` binding in JavaScript.

### Related Programming Areas

- **React Hooks** — the modern alternative for state and lifecycle management.
- **Error Boundaries** — a class-only feature for catching rendering errors.
- **Legacy codebases** — maintaining older React applications.
- **Migration strategies** — converting class components to function components.

---

## 1. Class Components

### Definitions

**Core Definition:** A class component is a React component defined as an ES6 class that extends `React.Component`.

**Technical Definition:** A class component is a JavaScript class that extends `React.Component` and implements a `render()` method returning React elements. It can access props via `this.props`, manage state via `this.state` and `this.setState()`, and override lifecycle methods. Only the `render()` method is required; all other methods are optional.

**Beginner-Friendly Explanation:** A class component is a component written as a class. It has a `render()` method that tells React what to show on screen. It can also have other methods that run at specific times.

### Purposes

- To define React components using ES6 class syntax.
- To access lifecycle methods for side effects.
- To manage local state with `this.setState()`.
- To create error boundaries (which require class components).
- To maintain legacy React codebases.

### Syntax Rules and Structure

**General Syntax:**

```jsx
import { Component } from 'react';

class ComponentName extends Component {
  constructor(props) {
    super(props);
    this.state = { ... };
  }

  render() {
    return <JSX />;
  }
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `class ComponentName extends Component` | Class declaration extending React's Component. |
| `constructor(props)` | Initializes state and binds methods. |
| `super(props)` | Required call to parent constructor. |
| `this.state` | The component's internal state object. |
| `render()` | Required method that returns JSX. |

**Syntax Rules:**

1. Must extend `React.Component` or `React.PureComponent`.
2. Must define a `render()` method that returns a single root element, fragment, `null`, or array.
3. `super(props)` must be called first in the constructor.
4. State is initialized as an object in the constructor.
5. Event handlers must be bound to `this` (or use class fields/arrow functions).

**Constraints and Limitations:**

- Cannot use React Hooks.
- More verbose than function components.
- `this` binding can cause bugs if not handled correctly.
- Not recommended for new code by the React team.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Basic Class Component

```jsx
import React from 'react';

// Step 1: Define the class component
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}!</h1>;
  }
}

// Step 2: Use the component
function App() {
  return (
    <div>
      <Welcome name="Alice" />
      <Welcome name="Bob" />
    </div>
  );
}
```

**Expected Output:**

```html
<div>
  <h1>Hello, Alice!</h1>
  <h1>Hello, Bob!</h1>
</div>
```

**Why this output:** The `Welcome` class component is instantiated twice with different `name` props. The `render()` method accesses `this.props.name` to display the name.

#### Example 2: Class Component with State and Lifecycle

```jsx
import React from 'react';

class Counter extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
  }

  componentDidMount() {
    console.log('Component mounted');
  }

  componentDidUpdate(prevProps, prevState) {
    console.log(`Count changed from ${prevState.count} to ${this.state.count}`);
  }

  componentWillUnmount() {
    console.log('Component will unmount');
  }

  handleClick = () => {
    this.setState({ count: this.state.count + 1 });
  };

  render() {
    return (
      <div>
        <p>Count: {this.state.count}</p>
        <button onClick={this.handleClick}>Increment</button>
      </div>
    );
  }
}
```

**Expected Output:** A counter that logs lifecycle events and increments on click.

**Why this output:** `componentDidMount` runs after the first render. `componentDidUpdate` runs after each state update. `componentWillUnmount` runs before removal. The `handleClick` class field automatically binds `this`.

### Real-World Cases

- **Legacy applications:** Maintaining older React codebases that use class components.
- **Error boundaries:** Class components are required for error boundaries.
- **Lifecycle-heavy components:** Components needing fine-grained lifecycle control (e.g., `getSnapshotBeforeUpdate`).

### References

- React — Component — https://react.dev/reference/react/Component
- React — Components and Props — https://legacy.reactjs.org/docs/components-and-props.html

---

## 2. `React.Component`

### Definitions

**Core Definition:** `React.Component` is the base class for defining React components as ES6 classes.

**Technical Definition:** `React.Component` is an abstract base class provided by the React library. When a class extends `Component`, it inherits the `setState()` method, the `this.props` and `this.state` properties, and the ability to override lifecycle methods. The only method that must be defined in a subclass is `render()`.

**Beginner-Friendly Explanation:** `React.Component` is the parent class you extend to create a class component. It gives your component all the React features it needs.

### Purposes

- To provide the base functionality for class components.
- To inherit `setState()`, `this.props`, and `this.state`.
- To enable lifecycle method overrides.
- To serve as the foundation for error boundaries.

### Syntax Rules and Structure

**General Syntax:**

```jsx
import { Component } from 'react';

class MyComponent extends Component {
  render() {
    return <div>Hello</div>;
  }
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Component` | The base class imported from React. |
| `extends Component` | Inherits all Component APIs. |
| `render()` | Required method. |

**Syntax Rules:**

1. Import `Component` from `react`.
2. Extend `Component` in the class declaration.
3. Define `render()`.
4. Other methods (lifecycle, handlers) are optional.

**Constraints and Limitations:**

- `React.Component` is an abstract class; it is rarely referenced directly.
- Cannot be used with Hooks.

### Annotated Code Examples

```jsx
import React, { Component } from 'react';

class Greeting extends Component {
  render() {
    return <h1>Hello, {this.props.name}!</h1>;
  }
}

// Usage
<Greeting name="Taylor" />
```

**Expected Output:**

```html
<h1>Hello, Taylor!</h1>
```

**Why this output:** `Greeting` extends `Component`, so it inherits `this.props` and the `render()` method is called by React.

### Real-World Cases

- **All class components:** Every class component extends `React.Component` or `React.PureComponent`.
- **Error boundaries:** Error boundaries must extend `Component` and implement `componentDidCatch()`.

### References

- React — Component — https://react.dev/reference/react/Component

---

## 3. `React.PureComponent`

### Definitions

**Core Definition:** `React.PureComponent` is a subclass of `Component` that skips re-renders when props and state have not changed.

**Technical Definition:** `PureComponent` implements a shallow `shouldComponentUpdate()` comparison of `props` and `state`. If the new props and state are shallowly equal to the previous ones, React skips the re-render. It is equivalent to defining a custom `shouldComponentUpdate()` with shallow comparison.

**Beginner-Friendly Explanation:** `PureComponent` is like `Component`, but it is smart about re-rendering. If nothing has changed, it does not re-render, which can make your app faster.

### Purposes

- To skip unnecessary re-renders for performance optimization.
- To avoid manual `shouldComponentUpdate()` implementations.
- To optimize components with simple, shallow props and state.

### Syntax Rules and Structure

**General Syntax:**

```jsx
import { PureComponent } from 'react';

class Greeting extends PureComponent {
  render() {
    return <h1>Hello, {this.props.name}!</h1>;
  }
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `PureComponent` | Base class imported from React. |
| `extends PureComponent` | Inherits all Component APIs plus shallow comparison. |

**Syntax Rules:**

1. Extend `PureComponent` instead of `Component`.
2. Component must have pure rendering logic (same output for same props/state).
3. Shallow comparison means nested object changes are not detected.

**Constraints and Limitations:**

- Only shallow comparison: nested object changes may be missed.
- Not a guarantee of performance improvement; measure first.
- Context changes still trigger re-renders.

### Annotated Code Examples

```jsx
import { PureComponent, useState } from 'react';

class Greeting extends PureComponent {
  render() {
    console.log("Greeting rendered");
    return <h3>Hello{this.props.name && ', '}{this.props.name}!</h3>;
  }
}

function App() {
  const [name, setName] = useState('');
  const [address, setAddress] = useState('');

  return (
    <div>
      <input value={name} onChange={(e) => setName(e.target.value)} placeholder="Name" />
      <input value={address} onChange={(e) => setAddress(e.target.value)} placeholder="Address" />
      <Greeting name={name} />
    </div>
  );
}
```

**Expected Output:** "Greeting rendered" logs only when `name` changes, not when `address` changes.

**Why this output:** `PureComponent` shallowly compares `props`. Since `address` is not passed to `Greeting`, changing it does not cause a re-render of `Greeting`.

### Real-World Cases

- **Large lists:** PureComponent list items prevent unnecessary re-renders.
- **Performance-sensitive components:** Components with expensive rendering logic.
- **Design systems:** PureComponent primitives used in many places.

### References

- React — PureComponent — https://react.dev/reference/react/PureComponent

---

## 4. `render()` Method

### Definitions

**Core Definition:** The `render()` method is the only required method in a class component; it returns the JSX that describes the UI.

**Technical Definition:** `render()` is called by React during the render phase. It must be pure — it should return the same output for the same props and state, and must not modify state or interact with the browser. It returns React elements, fragments, `null`, or arrays.

**Beginner-Friendly Explanation:** `render()` is where you write what the component should look like. React calls it whenever it needs to update the screen.

### Purposes

- To define the component's UI output.
- To compute the JSX based on props and state.
- To be called by React during rendering and re-rendering.

### Syntax Rules and Structure

**General Syntax:**

```jsx
render() {
  return <JSX />;
}
```

**Syntax Rules:**

1. Must return a single value: JSX element, fragment, `null`, array, string, or number.
2. Must be pure (no side effects, no state updates).
3. Must not call `setState()`.
4. Must not directly interact with the DOM.

**Constraints and Limitations:**

- Cannot contain side effects (use `componentDidMount` or `componentDidUpdate` for that).
- Must not modify state.
- Called frequently; keep it fast.

### Annotated Code Examples

```jsx
class UserProfile extends React.Component {
  render() {
    const { name, age, isAdmin } = this.props;

    return (
      <div className="profile">
        <h2>{name}</h2>
        <p>Age: {age}</p>
        {isAdmin && <span className="badge">Admin</span>}
      </div>
    );
  }
}
```

**Expected Output (for `name="Alice" age={30} isAdmin={true}`):**

```html
<div class="profile">
  <h2>Alice</h2>
  <p>Age: 30</p>
  <span class="badge">Admin</span>
</div>
```

**Why this output:** `render()` destructures props and returns JSX. The conditional `isAdmin && ...` renders the badge only when `isAdmin` is true.

### Real-World Cases

- **Every class component:** `render()` is the only required method.
- **Conditional rendering:** The `render()` method often contains ternary or logical operators for conditional UI.

### References

- React — Component: render — https://react.dev/reference/react/Component#render

---

## 5. `constructor()`

### Definitions

**Core Definition:** The `constructor()` is a special method that runs before the component mounts, used to initialize state and bind methods.

**Technical Definition:** The constructor runs before the component is added to the screen. It is typically used for two purposes: declaring state (`this.state = { ... }`) and binding event handler methods (`this.handleClick = this.handleClick.bind(this)`). If you do not need to initialize state or bind methods, you can omit the constructor. Modern class field syntax often eliminates the need for a constructor entirely.

**Beginner-Friendly Explanation:** The constructor is the setup method. It runs once when the component is created. You use it to set up the initial state and make sure your event handlers work correctly.

### Purposes

- To initialize the component's state.
- To bind event handler methods to the component instance.
- To perform any setup that must happen before the first render.

### Syntax Rules and Structure

**General Syntax:**

```jsx
constructor(props) {
  super(props);
  this.state = { ... };
  this.handleClick = this.handleClick.bind(this);
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `constructor(props)` | The constructor method. |
| `super(props)` | Required call to parent constructor. |
| `this.state = { ... }` | Initializes state. |
| `this.method = this.method.bind(this)` | Binds methods to `this`. |

**Syntax Rules:**

1. `super(props)` must be called before `this`.
2. State should be assigned directly in the constructor.
3. Methods passed as callbacks must be bound to `this`.

**Constraints and Limitations:**

- Do not call `setState()` in the constructor; use `this.state = ...` instead.
- Do not perform side effects in the constructor.
- Constructors are often unnecessary with class field syntax.

### Annotated Code Examples

```jsx
class Counter extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    this.setState({ count: this.state.count + 1 });
  }

  render() {
    return (
      <div>
        <p>Count: {this.state.count}</p>
        <button onClick={this.handleClick}>Increment</button>
      </div>
    );
  }
}
```

**Expected Output:** A counter that increments on click.

**Why this output:** The constructor initializes `state` and binds `handleClick` to `this`. The bound method can access `this.setState()`.

### Real-World Cases

- **State initialization:** Setting initial values for component state.
- **Method binding:** Ensuring event handlers have access to `this`.
- **Legacy code:** Understanding older React code patterns.

### References

- React — Component: constructor — https://react.dev/reference/react/Component#constructor

---

## 6. Component Lifecycle Methods

### Definitions

**Core Definition:** Lifecycle methods are special methods that React calls at specific points in a component's life: mounting, updating, and unmounting.

**Technical Definition:** Class components provide lifecycle methods that allow developers to run code at specific phases of a component's existence. These include `componentDidMount()`, `componentDidUpdate()`, `componentWillUnmount()`, and others. The React team provides a lifecycle diagram as a reference.

**Beginner-Friendly Explanation:** Lifecycle methods are like hooks in a component's life. They run when the component appears, when it changes, and when it disappears.

### Purposes

- To run side effects at specific lifecycle phases.
- To fetch data when a component mounts.
- To clean up resources when a component unmounts.
- To optimize rendering with `shouldComponentUpdate()`.
- To handle errors with `componentDidCatch()`.

---

### 6.1 `componentDidMount()`

#### Definitions

**Core Definition:** `componentDidMount()` is called after the component is inserted into the DOM.

**Technical Definition:** `componentDidMount()` is invoked immediately after a component is mounted (inserted into the DOM tree). It is the ideal place for data fetching, subscriptions, or DOM manipulations that require the component to be rendered first.

**Beginner-Friendly Explanation:** `componentDidMount()` runs after the component appears on screen. It is the best place to fetch data or set up things that need the DOM.

#### Purposes

- To fetch data from an API after the component mounts.
- To set up subscriptions (WebSocket, event listeners).
- To perform DOM measurements that require the rendered element.
- To initialize third-party libraries that need a DOM node.

#### Syntax Rules and Structure

```jsx
componentDidMount() {
  // Side effects here
}
```

**Syntax Rules:**

1. Called once after the first render.
2. Safe to call `setState()` here (it triggers a re-render).
3. Cannot be called on the server (SSR).

#### Annotated Code Example

```jsx
class DataFetcher extends React.Component {
  state = { data: null };

  componentDidMount() {
    fetch('/api/data')
      .then((res) => res.json())
      .then((data) => this.setState({ data }));
  }

  render() {
    return <div>{this.state.data ? this.state.data.title : 'Loading...'}</div>;
  }
}
```

**Expected Output:** "Loading..." initially, then the fetched data title.

**Why this output:** `componentDidMount` runs after the first render, fetches data, and updates state, triggering a second render.

#### References

- React — componentDidMount — https://react.dev/reference/react/Component#componentdidmount

---

### 6.2 `componentDidUpdate()`

#### Definitions

**Core Definition:** `componentDidUpdate()` is called after the component re-renders due to props or state changes.

**Technical Definition:** `componentDidUpdate(prevProps, prevState, snapshot)` is invoked immediately after updating occurs. It is not called for the initial render. It receives the previous props and state, allowing comparisons. It is a good place for DOM updates or network requests based on prop changes.

**Beginner-Friendly Explanation:** `componentDidUpdate()` runs after the component updates. You can compare old and new props or state here.

#### Purposes

- To perform DOM operations after updates.
- To fetch new data when props change.
- To compare previous and current state/props.

#### Syntax Rules and Structure

```jsx
componentDidUpdate(prevProps, prevState, snapshot) {
  if (prevProps.id !== this.props.id) {
    // Fetch new data
  }
}
```

**Syntax Rules:**

1. Not called on initial render.
2. Wrap `setState()` in a condition to avoid infinite loops.
3. The third argument is the snapshot from `getSnapshotBeforeUpdate()`.

#### Annotated Code Example

```jsx
class UserProfile extends React.Component {
  state = { user: null };

  componentDidUpdate(prevProps) {
    if (prevProps.userId !== this.props.userId) {
      fetch(`/api/users/${this.props.userId}`)
        .then((res) => res.json())
        .then((user) => this.setState({ user }));
    }
  }

  render() {
    return <div>{this.state.user ? this.state.user.name : 'Loading...'}</div>;
  }
}
```

**Expected Output:** The user profile updates when `userId` changes.

**Why this output:** `componentDidUpdate` compares `prevProps.userId` with the current prop and fetches new data only when it changes.

#### References

- React — componentDidUpdate — https://react.dev/reference/react/Component#componentdidupdate

---

### 6.3 `componentWillUnmount()`

#### Definitions

**Core Definition:** `componentWillUnmount()` is called immediately before a component is removed from the DOM.

**Technical Definition:** `componentWillUnmount()` is invoked right before the component is unmounted and destroyed. It is the place to perform cleanup: cancelling timers, aborting network requests, removing event listeners, and unsubscribing from external data sources.

**Beginner-Friendly Explanation:** `componentWillUnmount()` runs when the component is about to disappear. Use it to clean up anything the component was doing.

#### Purposes

- To cancel timers and intervals.
- To abort in-flight network requests.
- To remove event listeners.
- To unsubscribe from WebSocket or external subscriptions.

#### Syntax Rules and Structure

```jsx
componentWillUnmount() {
  // Cleanup code
}
```

**Syntax Rules:**

1. Called once before unmounting.
2. Do not call `setState()` here (the component will not re-render).

#### Annotated Code Example

```jsx
class Timer extends React.Component {
  state = { seconds: 0 };

  componentDidMount() {
    this.interval = setInterval(() => {
      this.setState((s) => ({ seconds: s.seconds + 1 }));
    }, 1000);
  }

  componentWillUnmount() {
    clearInterval(this.interval);
  }

  render() {
    return <p>Seconds: {this.state.seconds}</p>;
  }
}
```

**Expected Output:** A timer that increments every second and stops when unmounted.

**Why this output:** `componentDidMount` starts the interval. `componentWillUnmount` clears it, preventing memory leaks.

#### References

- React — componentWillUnmount — https://react.dev/reference/react/Component#componentwillunmount

---

### 6.4 `shouldComponentUpdate()`

#### Definitions

**Core Definition:** `shouldComponentUpdate()` determines whether a component should re-render when props or state change.

**Technical Definition:** `shouldComponentUpdate(nextProps, nextState)` is called before rendering when new props or state are received. Returning `false` prevents the re-render. It is used for performance optimization. `PureComponent` implements this with shallow comparison.

**Beginner-Friendly Explanation:** `shouldComponentUpdate()` lets you tell React whether to re-render or not. Returning `false` skips the update.

#### Purposes

- To optimize performance by preventing unnecessary re-renders.
- To implement custom comparison logic for props and state.

#### Syntax Rules and Structure

```jsx
shouldComponentUpdate(nextProps, nextState) {
  return nextProps.value !== this.props.value;
}
```

**Syntax Rules:**

1. Returning `false` skips the render and `componentDidUpdate`.
2. Not called on initial render or when `forceUpdate()` is used.
3. Must be pure.

#### Annotated Code Example

```jsx
class Item extends React.Component {
  shouldComponentUpdate(nextProps) {
    return nextProps.value !== this.props.value;
  }

  render() {
    console.log('Item rendered');
    return <div>{this.props.value}</div>;
  }
}
```

**Expected Output:** "Item rendered" logs only when `value` changes.

**Why this output:** `shouldComponentUpdate` returns `false` when the value is unchanged, skipping the re-render.

#### References

- React — shouldComponentUpdate — https://react.dev/reference/react/Component#shouldcomponentupdate

---

### 6.5 `getDerivedStateFromProps()`

#### Definitions

**Core Definition:** `getDerivedStateFromProps()` is a static method called before rendering, used to update state based on props.

**Technical Definition:** `static getDerivedStateFromProps(props, state)` is invoked before the `render()` method, both on initial mount and on subsequent updates. It returns an object to update state, or `null` to indicate no state update. It is a rare-use lifecycle method for state that depends on props over time.

**Beginner-Friendly Explanation:** `getDerivedStateFromProps()` lets you update state when props change. It runs before rendering.

#### Purposes

- To update state in response to prop changes.
- To implement derived state from props.

#### Syntax Rules and Structure

```jsx
static getDerivedStateFromProps(props, state) {
  return { key: props.value };
}
```

**Syntax Rules:**

1. Must be `static`.
2. Must return an object or `null`.
3. Cannot access `this`.

#### Annotated Code Example

```jsx
class Counter extends React.Component {
  state = { count: 0 };

  static getDerivedStateFromProps(props, state) {
    if (props.initialCount !== state.count) {
      return { count: props.initialCount };
    }
    return null;
  }

  render() {
    return <p>Count: {this.state.count}</p>;
  }
}
```

**Expected Output:** The counter initializes with `initialCount` and updates when the prop changes.

**Why this output:** `getDerivedStateFromProps` syncs state with the `initialCount` prop before each render.

#### References

- React — getDerivedStateFromProps — https://react.dev/reference/react/Component#static-getderivedstatefromprops

---

### 6.6 `getSnapshotBeforeUpdate()`

#### Definitions

**Core Definition:** `getSnapshotBeforeUpdate()` captures some information from the DOM before it is potentially changed.

**Technical Definition:** `getSnapshotBeforeUpdate(prevProps, prevState)` is invoked right before the most recently rendered output is committed to the DOM. Its return value is passed as the third parameter to `componentDidUpdate()`. It is used to capture DOM values like scroll position before an update.

**Beginner-Friendly Explanation:** `getSnapshotBeforeUpdate()` lets you grab information from the DOM before React updates it, like saving the scroll position.

#### Purposes

- To capture scroll position before a DOM update.
- To measure DOM elements before they change.
- To preserve visual state during updates.

#### Syntax Rules and Structure

```jsx
getSnapshotBeforeUpdate(prevProps, prevState) {
  return this.listRef.current.scrollHeight;
}

componentDidUpdate(prevProps, prevState, snapshot) {
  // Use snapshot
}
```

**Syntax Rules:**

1. Return value is passed to `componentDidUpdate`.
2. Must be used with `componentDidUpdate`.
3. Rarely used; only for specific DOM measurement cases.

#### Annotated Code Example

```jsx
class ChatList extends React.Component {
  listRef = React.createRef();

  getSnapshotBeforeUpdate(prevProps) {
    if (prevProps.messages.length < this.props.messages.length) {
      const list = this.listRef.current;
      return list.scrollHeight - list.scrollTop;
    }
    return null;
  }

  componentDidUpdate(prevProps, prevState, snapshot) {
    if (snapshot !== null) {
      const list = this.listRef.current;
      list.scrollTop = list.scrollHeight - snapshot;
    }
  }

  render() {
    return (
      <div ref={this.listRef}>
        {this.props.messages.map((msg) => (
          <div key={msg.id}>{msg.text}</div>
        ))}
      </div>
    );
  }
}
```

**Expected Output:** The chat list maintains scroll position when new messages are added.

**Why this output:** `getSnapshotBeforeUpdate` captures the scroll offset before the update. `componentDidUpdate` restores it after the DOM changes.

#### References

- React — getSnapshotBeforeUpdate — https://react.dev/reference/react/Component#getsnapshotbeforeupdate

---

### 6.7 `componentDidCatch()`

#### Definitions

**Core Definition:** `componentDidCatch()` is a lifecycle method used in error boundaries to catch errors in child components.

**Technical Definition:** `componentDidCatch(error, info)` is invoked after an error has been thrown by a descendant component. It is used to log error information and display a fallback UI. Class components that implement `componentDidCatch()` (and optionally `static getDerivedStateFromError()`) become error boundaries.

**Beginner-Friendly Explanation:** `componentDidCatch()` catches errors that happen in child components. It lets you show a friendly error message instead of a broken app.

#### Purposes

- To catch rendering errors in child components.
- To log errors to an error reporting service.
- To display a fallback UI when errors occur.

#### Syntax Rules and Structure

```jsx
class ErrorBoundary extends React.Component {
  state = { hasError: false };

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, info) {
    logErrorToMyService(error, info);
  }

  render() {
    if (this.state.hasError) {
      return <h1>Something went wrong.</h1>;
    }
    return this.props.children;
  }
}
```

**Syntax Rules:**

1. Only class components can be error boundaries.
2. Must implement `getDerivedStateFromError` or `componentDidCatch`.
3. Does not catch errors in event handlers or async code.

#### Annotated Code Example

```jsx
class ErrorBoundary extends React.Component {
  state = { hasError: false };

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    console.error('Error caught:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return <h1>Something went wrong.</h1>;
    }
    return this.props.children;
  }
}

function BuggyComponent() {
  throw new Error('I crashed!');
}

<ErrorBoundary>
  <BuggyComponent />
</ErrorBoundary>
```

**Expected Output:** "Something went wrong." is displayed instead of a crashed app.

**Why this output:** `BuggyComponent` throws during render. `ErrorBoundary` catches the error and renders the fallback UI.

#### References

- React — Error Boundaries — https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary

---

## 7. `this.state`

### Definitions

**Core Definition:** `this.state` is the object that holds a class component's mutable data.

**Technical Definition:** `this.state` is a plain JavaScript object that stores data specific to the component instance. It is initialized in the constructor or as a class field. It should be treated as immutable — never mutate it directly; always use `this.setState()` to update it.

**Beginner-Friendly Explanation:** `this.state` is the component's personal data storage. It holds values that can change over time.

### Purposes

- To store data that changes over time.
- To trigger re-renders when updated via `setState()`.
- To manage component-specific UI state.

### Syntax Rules and Structure

```jsx
// Constructor initialization
constructor(props) {
  super(props);
  this.state = { count: 0 };
}

// Class field initialization
state = { count: 0 };
```

**Syntax Rules:**

1. State must be an object.
2. Never mutate state directly.
3. Use `this.setState()` to update state.

### Annotated Code Example

```jsx
class Toggle extends React.Component {
  state = { isOn: false };

  handleClick = () => {
    this.setState({ isOn: !this.state.isOn });
  };

  render() {
    return (
      <button onClick={this.handleClick}>
        {this.state.isOn ? 'ON' : 'OFF'}
      </button>
    );
  }
}
```

**Expected Output:** A button that toggles between "ON" and "OFF".

**Why this output:** `this.state.isOn` determines the button text. Clicking calls `setState`, triggering a re-render.

### References

- React — Component: state — https://react.dev/reference/react/Component#state

---

## 8. `this.setState()`

### Definitions

**Core Definition:** `this.setState()` schedules an update to a class component's state object.

**Technical Definition:** `setState(updater, callback)` enqueues changes to the component's state. When state changes, the component re-renders. `setState` is asynchronous; do not rely on `this.state` reflecting the new value immediately after calling it. For state that depends on previous state, pass an updater function.

**Beginner-Friendly Explanation:** `this.setState()` is how you change a class component's state. It tells React to update the component.

### Purposes

- To update component state and trigger re-renders.
- To batch state updates for performance.
- To compute new state based on previous state with updater functions.

### Syntax Rules and Structure

**Object Syntax:**

```jsx
this.setState({ count: this.state.count + 1 });
```

**Updater Function Syntax:**

```jsx
this.setState((state) => ({ count: state.count + 1 }));
```

**Syntax Rules:**

1. State updates are asynchronous and batched.
2. Use updater functions when new state depends on previous state.
3. Do not mutate `this.state` directly.
4. An optional callback can run after the update.

### Annotated Code Example

```jsx
class Counter extends React.Component {
  state = { count: 0 };

  increment = () => {
    // Updater function ensures correct value
    this.setState((state) => ({ count: state.count + 1 }));
  };

  incrementThree = () => {
    this.increment();
    this.increment();
    this.increment();
  };

  render() {
    return (
      <div>
        <p>Count: {this.state.count}</p>
        <button onClick={this.incrementThree}>+3</button>
      </div>
    );
  }
}
```

**Expected Output:** The counter increments by 3 when the button is clicked.

**Why this output:** The updater function form of `setState` reads the latest state value, so chained updates correctly build on each other.

### References

- React — setState — https://react.dev/reference/react/Component#setstate

---

## 9. `this.props`

### Definitions

**Core Definition:** `this.props` is the object containing the read-only inputs passed to a class component from its parent.

**Technical Definition:** `this.props` contains the props passed to the component when it was created. Props are immutable from the component's perspective and flow one-way from parent to child. In class components, props are accessed via `this.props`.

**Beginner-Friendly Explanation:** `this.props` is how a class component receives data from its parent. It cannot change the props it receives.

### Purposes

- To receive data from parent components.
- To customize component behavior and appearance.
- To pass event handlers and callbacks.

### Syntax Rules and Structure

```jsx
class Greeting extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}!</h1>;
  }
}

<Greeting name="Taylor" />
```

**Syntax Rules:**

1. Props are read-only; never modify `this.props`.
2. Props flow one-way from parent to child.
3. Props can be any JavaScript value.

### Annotated Code Example

```jsx
class UserCard extends React.Component {
  render() {
    const { user, onEdit } = this.props;

    return (
      <div className="card">
        <h2>{user.name}</h2>
        <p>{user.email}</p>
        <button onClick={() => onEdit(user.id)}>Edit</button>
      </div>
    );
  }
}
```

**Expected Output:** A card showing user details with an edit button.

**Why this output:** `this.props.user` and `this.props.onEdit` are accessed and used in the render method.

### References

- React — Component: props — https://react.dev/reference/react/Component#props

---

## 10. Binding Methods

### Definitions

**Core Definition:** Binding methods ensures that event handler functions have the correct `this` context when called.

**Technical Definition:** In JavaScript, class methods are not bound by default. When a method is passed as a callback (e.g., `onClick={this.handleClick}`), `this` becomes `undefined` inside the method. Binding (`this.handleClick = this.handleClick.bind(this)`) or using arrow functions (class fields) preserves the correct `this` context.

**Beginner-Friendly Explanation:** When you pass a method to an event handler, JavaScript loses track of `this`. Binding fixes that.

### Purposes

- To ensure event handlers can access `this.props` and `this.state`.
- To prevent `this is undefined` errors in callbacks.

### Syntax Rules and Structure

**Bind in Constructor:**

```jsx
constructor(props) {
  super(props);
  this.handleClick = this.handleClick.bind(this);
}
```

**Class Field (Arrow Function):**

```jsx
handleClick = () => {
  // 'this' is automatically bound
};
```

**Arrow Function in Render:**

```jsx
<button onClick={() => this.handleClick()}>Click</button>
```

**Syntax Rules:**

1. Binding in constructor creates one bound function per instance.
2. Class fields with arrow functions automatically bind `this`.
3. Arrow functions in render create a new function on every render.

### Annotated Code Example

```jsx
class Button extends React.Component {
  constructor(props) {
    super(props);
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    console.log('Clicked', this.props.label);
  }

  render() {
    return <button onClick={this.handleClick}>{this.props.label}</button>;
  }
}
```

**Expected Output:** Clicking the button logs the label.

**Why this output:** The constructor binds `handleClick` to the instance, so `this.props.label` is accessible when called.

### References

- React — Passing Functions to Components — https://legacy.reactjs.org/docs/faq-functions.html

---

## 11. Class Fields

### Definitions

**Core Definition:** Class fields are a JavaScript syntax that allows properties to be defined directly on a class without a constructor.

**Technical Definition:** Class fields (also called class properties) let you declare instance properties and arrow function methods directly in the class body. This is a Stage 3 proposal supported by Babel and TypeScript. It eliminates the need for a constructor when initializing state or binding methods.

**Beginner-Friendly Explanation:** Class fields let you write state and methods outside the constructor, making your code cleaner.

### Purposes

- To initialize state without a constructor.
- To bind event handlers using arrow function class fields.
- To reduce boilerplate in class components.

### Syntax Rules and Structure

```jsx
class Counter extends React.Component {
  state = { count: 0 };

  handleClick = () => {
    this.setState({ count: this.state.count + 1 });
  };

  render() {
    return <button onClick={this.handleClick}>{this.state.count}</button>;
  }
}
```

**Syntax Rules:**

1. Class fields are defined with `=` directly in the class body.
2. Arrow function class fields automatically bind `this`.
3. Supported by Babel's class properties transform and TypeScript.

### Annotated Code Example

```jsx
class Toggle extends React.Component {
  state = { isOn: false };

  handleClick = () => {
    this.setState({ isOn: !this.state.isOn });
  };

  render() {
    return (
      <button onClick={this.handleClick}>
        {this.state.isOn ? 'ON' : 'OFF'}
      </button>
    );
  }
}
```

**Expected Output:** A button that toggles between "ON" and "OFF".

**Why this output:** The class field `state` initializes state without a constructor. The arrow function class field `handleClick` is automatically bound to the instance.

### References

- Babel — Class Properties — https://babeljs.io/docs/en/babel-plugin-transform-class-properties/
- TC39 — Class Fields Proposal — https://github.com/tc39/proposal-class-fields

---

## 12. Class Components vs Function Components

### Definitions

**Core Definition:** Class components and function components are the two ways to define React components, differing in syntax, features, and recommended usage.

**Technical Definition:** Class components use ES6 class syntax, provide lifecycle methods, and access state via `this.state`/`this.setState()`. Function components are plain functions that use Hooks (`useState`, `useEffect`) for state and lifecycle. The React team recommends function components for all new code.

**Beginner-Friendly Explanation:** Class components are the old way; function components are the new, recommended way. Both can do the same things, but functions are simpler.

### Purposes

- To understand the trade-offs between the two approaches.
- To guide decisions on which to use in a project.
- To facilitate migration from class to function components.

### Comparison Table

| Aspect | Class Components | Function Components |
|--------|-----------------|-------------------|
| Syntax | ES6 classes | Functions |
| State | `this.state`, `this.setState()` | `useState` |
| Lifecycle | Lifecycle methods | `useEffect` |
| `this` binding | Required for handlers | Not needed |
| Hooks | Not supported | Supported |
| Error boundaries | Supported | Not supported |
| Code size | Larger | Smaller |
| Recommended | No (legacy) | Yes |

### Annotated Code Examples

**Class Component:**

```jsx
class Counter extends React.Component {
  state = { count: 0 };
  handleClick = () => this.setState({ count: this.state.count + 1 });
  render() {
    return <button onClick={this.handleClick}>{this.state.count}</button>;
  }
}
```

**Function Component:**

```jsx
function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

**Expected Output (both):** A button that increments a counter.

**Why this output:** Both achieve the same behavior. The function component is more concise and uses Hooks.

### Real-World Cases

- **New projects:** Always use function components.
- **Legacy projects:** Class components may still exist; migrate incrementally.
- **Error boundaries:** Must use class components.

### References

- React — Migrating from Class Components to Functions — https://react.dev/reference/react/Component#alternatives
- React — Your First Component — https://react.dev/learn/your-first-component

---

## 13. When to Use Class Components

### Definitions

**Core Definition:** Class components should be used only in specific legacy or edge-case scenarios.

**Technical Definition:** The React team recommends function components for all new code. Class components are still needed for error boundaries (which cannot be function components) and for maintaining existing class-based codebases. Other use cases (state, lifecycle, refs) are now fully supported in function components via Hooks.

**Beginner-Friendly Explanation:** Use class components only if you are maintaining old code or need an error boundary. For everything else, use function components.

### Purposes

- To identify the remaining legitimate use cases for class components.
- To avoid unnecessary class components in new code.
- To plan migration strategies.

### Use Cases

| Scenario | Recommendation |
|----------|---------------|
| New components | Use function components |
| Error boundaries | Must use class components |
| Legacy code maintenance | Keep class components until migration |
| Third-party class components | Use as-is; wrap in function components if needed |
| Components needing `getSnapshotBeforeUpdate` | Class components (rare) |

### Annotated Code Example

```jsx
// Error boundary must be a class component
class ErrorBoundary extends React.Component {
  state = { hasError: false };
  static getDerivedStateFromError() {
    return { hasError: true };
  }
  render() {
    if (this.state.hasError) return <h1>Error</h1>;
    return this.props.children;
  }
}
```

**Expected Output:** A fallback UI when a child throws.

**Why this output:** Error boundaries require class components; there is no function component equivalent.

### References

- React — Error Boundaries — https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary
- React — Migrating from Class Components to Functions — https://react.dev/reference/react/Component#alternatives

---

## 14. Migrating to Function Components

### Definitions

**Core Definition:** Migration is the process of converting existing class components to function components with Hooks.

**Technical Definition:** Migration involves replacing `this.state` with `useState`, lifecycle methods with `useEffect`, `this.props` with function parameters, and class methods with functions or `useCallback`. Automated codemods (e.g., `class-to-function-generator`) can handle approximately 80% of conversions, but manual review is required for complex state and lifecycle logic.

**Beginner-Friendly Explanation:** Migration means rewriting class components as function components. Tools can help, but some parts need manual work.

### Purposes

- To modernize a codebase to use React's recommended patterns.
- To reduce boilerplate and improve readability.
- To enable the use of Hooks and future React features.

### Migration Mapping Table

| Class Component | Function Component |
|----------------|-------------------|
| `this.state` | `useState` |
| `this.setState()` | `setState` from `useState` |
| `componentDidMount` | `useEffect(() => {}, [])` |
| `componentDidUpdate` | `useEffect(() => {}, [deps])` |
| `componentWillUnmount` | `useEffect` cleanup function |
| `this.props` | Function parameters |
| Lifecycle methods | `useEffect` |
| `this.method.bind(this)` | Arrow functions or `useCallback` |
| `PureComponent` | `React.memo` |
| `componentDidCatch` | Error boundaries (still class) |

### Annotated Code Example

**Before (Class Component):**

```jsx
class UserProfile extends React.Component {
  state = { user: null };

  componentDidMount() {
    fetch(`/api/users/${this.props.userId}`)
      .then((res) => res.json())
      .then((user) => this.setState({ user }));
  }

  render() {
    return <div>{this.state.user?.name || 'Loading...'}</div>;
  }
}
```

**After (Function Component):**

```jsx
import { useState, useEffect } from 'react';

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then((res) => res.json())
      .then(setUser);
  }, [userId]);

  return <div>{user?.name || 'Loading...'}</div>;
}
```

**Expected Output (both):** The user name is fetched and displayed.

**Why this output:** The function component uses `useState` for state and `useEffect` for the fetch, replacing `componentDidMount` and `this.setState`.

### Real-World Cases

- **Modernization projects:** Upgrading legacy React codebases.
- **Codebase consistency:** Ensuring all components use the same patterns.
- **Performance:** Function components with Hooks are often more efficient.

### References

- React — Migrating from Class Components to Functions — https://react.dev/reference/react/Component#alternatives
- class-to-function-generator — https://www.npmjs.com/package/class-to-function-generator

---

## References

- React — Component — https://react.dev/reference/react/Component
- React — PureComponent — https://react.dev/reference/react/PureComponent
- React — Component: render — https://react.dev/reference/react/Component#render
- React — Component: constructor — https://react.dev/reference/react/Component#constructor
- React — componentDidMount — https://react.dev/reference/react/Component#componentdidmount
- React — componentDidUpdate — https://react.dev/reference/react/Component#componentdidupdate
- React — componentWillUnmount — https://react.dev/reference/react/Component#componentwillunmount
- React — shouldComponentUpdate — https://react.dev/reference/react/Component#shouldcomponentupdate
- React — getDerivedStateFromProps — https://react.dev/reference/react/Component#static-getderivedstatefromprops
- React — getSnapshotBeforeUpdate — https://react.dev/reference/react/Component#getsnapshotbeforeupdate
- React — Error Boundaries — https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary
- React — setState — https://react.dev/reference/react/Component#setstate
- React — Passing Functions to Components — https://legacy.reactjs.org/docs/faq-functions.html
- React — Migrating from Class Components to Functions — https://react.dev/reference/react/Component#alternatives
- Babel — Class Properties — https://babeljs.io/docs/en/babel-plugin-transform-class-properties/
- TC39 — Class Fields Proposal — https://github.com/tc39/proposal-class-fields
- class-to-function-generator — https://www.npmjs.com/package/class-to-function-generator