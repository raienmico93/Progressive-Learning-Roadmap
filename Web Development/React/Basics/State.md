# React State Fundamentals — Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**

State is a component's built-in mechanism for remembering information that changes over time, enabling components to update their rendered output in response to user interactions and other events.

**Technical Definition**

State is component-specific memory managed by React, declared using the `useState` Hook (or `useReducer` for complex logic). Unlike regular JavaScript variables that disappear when a function returns, state persists across renders because React stores it externally and provides a snapshot of the current value to each render. Calling a state setter function queues a re-render and updates the stored value for subsequent renders, but does not mutate the value within the currently executing code.

**Beginner-Friendly Explanation**

Think of state like a component's memory. When you type into an input field, the component needs to "remember" what you typed so it can display it. When you click a button to increment a counter, the component needs to "remember" the new number. State is how React components remember things between renders—without state, your UI would reset to its initial values every time it updated.

### Key Characteristics

- **Component-Specific Memory**: State "lives" in React itself, not in the component function. It persists across renders and is restored when the component re-renders.
- **Snapshot Behavior**: State variables behave like snapshots. Setting state does not change the variable in the currently executing code—it only affects the next render .
- **Isolated and Private**: State is local to a component instance. Rendering the same component twice creates two independent states that do not share values .
- **Immutable Updates**: State should never be mutated directly. Objects and arrays in state must be replaced with new copies when updated .
- **Re-render Trigger**: Calling a state setter requests a re-render from React, which then provides the updated state to the next render .

### Prerequisites

Before studying React State, learners should be comfortable with:

- **JavaScript Fundamentals**: Variables, functions, arrays, objects, and the spread operator.
- **React Components**: Functional components and JSX syntax.
- **Props**: How data flows from parent to child components.
- **Event Handling**: Writing event handlers and passing them as props.

### Related Programming Areas

- **React Hooks**: `useState`, `useReducer`, and other built-in hooks that manage state.
- **State Management Libraries**: Redux, Zustand, Jotai, and Context API for sharing state across components.
- **Data Fetching**: Loading and caching server data in component state.
- **Form Management**: Controlled components where input values are stored in state.

### Core Concepts / Features

- State Definition
- Local Component State
- State Updates
- State Persistence Across Renders
- Derived State
- State Ownership
- State Lifting

---

## 1. State Definition

### Definitions

**Core Definition**

State is a component's private, persistent memory that holds values which, when changed, trigger the component to re-render with updated output.

**Technical Definition**

State is data managed by React's internal memory system on behalf of a component. It is declared via `useState` (which returns a `[value, setter]` pair) or `useReducer` (which returns a `[state, dispatch]` pair). React stores state outside the component function and provides the current value to each render as a snapshot. Updates are queued through setter functions and applied on the next render.

**Beginner-Friendly Explanation**

State is like a whiteboard that belongs to a specific component. You can write information on it, and every time the component redraws itself, it looks at the whiteboard to know what to display. If you change what's on the whiteboard, the component knows to redraw.

### Purposes

- To enable components to remember information between renders.
- To trigger UI updates when data changes.
- To manage interactive data like form inputs, toggles, and counters.
- To serve as the source of truth for a component's dynamic content.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
import { useState } from 'react';

function Component() {
  const [stateVariable, setStateVariable] = useState(initialValue);
  
  return <div>{stateVariable}</div>;
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `useState(initialValue)` | Hook call that declares a state variable. Returns a pair: current value and setter function. |
| `stateVariable` | The current state value for this render. |
| `setStateVariable` | Function to update the state and trigger a re-render. |
| `initialValue` | The value used on the first render only. Ignored on subsequent renders. |

**Syntax Rules**

1. **Hooks at the top level**: `useState` must be called unconditionally at the top level of a component or custom hook .
2. **Initial value**: The initial value is only used on the first render. React saves it and ignores it on subsequent renders .
3. **Multiple state variables**: You can call `useState` multiple times. React matches them by their order of declaration .
4. **Lazy initialization**: For expensive initial values, pass a function instead of a value: `useState(() => computeExpensiveValue())` .

**Constraints and Limitations**

- Hooks cannot be called inside conditions, loops, or nested functions.
- State must be updated using the setter function, never by direct mutation.
- State variables are constants within a single render; their value does not change mid-render.

### Annotated Complete Code Examples

**Example 1: Basic State Declaration**

```jsx
import { useState } from 'react';

function Counter() {
  // Declare state: count starts at 0
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
    </div>
  );
}
```

**Expected Output (after two clicks)**

```
Count: 2
[Increment button]
```

**Why This Output Occurs**

`useState(0)` initializes `count` to `0`. Clicking the button calls `setCount(count + 1)`, which queues a re-render with the new value. React calls the component again, providing `count` as `1` (then `2`). The UI reflects the latest state.

**Example 2: Multiple State Variables**

```jsx
import { useState } from 'react';

function UserForm() {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');
  const [age, setAge] = useState(0);
  
  return (
    <div>
      <input 
        value={name} 
        onChange={e => setName(e.target.value)} 
        placeholder="Name"
      />
      <input 
        value={email} 
        onChange={e => setEmail(e.target.value)} 
        placeholder="Email"
      />
      <input 
        type="number"
        value={age} 
        onChange={e => setAge(Number(e.target.value))} 
        placeholder="Age"
      />
      <p>{name} ({email}), age {age}</p>
    </div>
  );
}
```

**Explanation**

Each `useState` call creates an independent state variable. React tracks them by call order: the first `useState` is always `name`, the second is always `email`, and the third is always `age`. This is why Hooks must be called unconditionally—skipping a call would shift the order and break the mapping .

### Real-World Cases

**Case: Controlled Form Inputs**

A login form uses state for `email` and `password`, with `value` and `onChange` props binding inputs to state. The form submission handler reads the current state values .

**Case: Shopping Cart Quantity**

A product card uses state for the selected quantity. Clicking "+" or "-" updates the state, which re-renders the quantity display and the total price.

### References

- React — State: A Component's Memory - https://react.dev/learn/state-a-components-memory
- React — useState API Reference - https://react.dev/reference/react/useState

---

## 2. Local Component State

### Definitions

**Core Definition**

Local component state is state that is owned and managed by a single component, not shared with or accessible to other components except through props passed down.

**Technical Definition**

Local state is declared within a component using `useState` and is private to that component. It is not accessible from parent components (unlike props, which flow down). The parent cannot read or modify a child's local state; the child cannot read a parent's state except through props. This encapsulation allows components to manage their own concerns independently.

**Beginner-Friendly Explanation**

Local state is like a personal diary that belongs to one component. Other components can't read it or write in it. If a parent needs information from a child's state, the child must send it up (via a callback prop), or the parent must own the state instead.

### Purposes

- To encapsulate data that only one component needs.
- To keep component logic self-contained and independent.
- To avoid unnecessary complexity from sharing state that doesn't need sharing.
- To enable components to manage their own UI interactions.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
function Component() {
  const [localState, setLocalState] = useState(initialValue);
  // localState is accessible only within this component
  return <div>{localState}</div>;
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `localState` | Private to the component instance. |
| `setLocalState` | Only callable within this component. |

**Syntax Rules**

1. Local state is declared with `useState` inside the component function.
2. It is not accessible from outside the component, including parent components.
3. Rendering the same component multiple times creates independent local states .
4. To share state with a parent, the state must be lifted up or a callback prop must be passed.

**Constraints and Limitations**

- Local state cannot be read or modified by other components.
- If two components need the same data, local state is insufficient; state must be lifted.
- Local state resets when the component unmounts.

### Annotated Complete Code Examples

**Example 1: Independent Local State**

```jsx
function Toggle() {
  const [isOn, setIsOn] = useState(false);
  
  return (
    <button onClick={() => setIsOn(!isOn)}>
      {isOn ? 'ON' : 'OFF'}
    </button>
  );
}

function App() {
  return (
    <div>
      <Toggle />
      <Toggle />
      <Toggle />
    </div>
  );
}
```

**Expected Output**

```
[OFF button] [OFF button] [OFF button]
```

**Why This Output Occurs**

Each `<Toggle />` is a separate component instance with its own `isOn` state. Clicking one button only changes that instance's state. The three toggles are completely independent, demonstrating that state is isolated and private to each component instance .

### Real-World Cases

**Case: A Modal Component**

A `Modal` component uses local state (`isOpen`) to track whether it is visible. The parent controls when to render the modal, but the modal manages its own open/closed animation state.

**Case: An Accordion Panel**

Each `Panel` component uses local state (`isExpanded`) to track whether its content is shown. This works well when panels are independent, but not when only one panel should be open at a time (requiring lifted state) .

### References

- React — State is Isolated and Private - https://react.dev/learn/state-a-components-memory

---

## 3. State Updates

### Definitions

**Core Definition**

State updates are the actions of changing a state variable's value by calling its setter function, which queues a re-render and updates the stored state for the next render.

**Technical Definition**

Calling a state setter (e.g., `setCount(5)`) does not mutate the state variable in the currently executing code. Instead, it schedules an update: React will re-render the component, and during that render, the state variable will have the new value. Multiple updates can be batched, and updates can be queued using updater functions to ensure each update operates on the latest pending state.

**Beginner-Friendly Explanation**

Updating state is like placing an order at a restaurant. You tell the waiter (setter function) what you want. The waiter doesn't immediately change your food—they take the order to the kitchen. The kitchen (React) prepares the new dish and brings it to you on the next round (re-render). If you place multiple orders before the food arrives, they all get handled together.

### Purposes

- To change the data a component displays.
- To trigger re-renders when data changes.
- To respond to user interactions and events.
- To schedule state changes that apply to the next render.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
// Direct value
setState(newValue);

// Updater function (for updates based on previous state)
setState(prevState => newValue);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `setState(newValue)` | Sets state to the given value. |
| `setState(prev => ...)` | Receives the pending state and returns the new state. |

**Syntax Rules**

1. **State is a snapshot**: Within a single render, a state variable's value never changes, even after calling its setter .
2. **Batching**: React batches multiple state updates within the same event handler for performance.
3. **Updater functions**: When updating based on previous state, use `setState(prev => prev + 1)` to ensure correct queueing .
4. **Objects and arrays**: Never mutate state directly. Create new objects/arrays with the spread operator .

**Constraints and Limitations**

- Calling the setter does not change the variable in the current render.
- Multiple `setState(value + 1)` calls in one handler all use the same `value`, resulting in one increment.
- Mutating state objects/arrays directly does not trigger re-renders and causes bugs.

### Annotated Complete Code Examples

**Example 1: The Snapshot Problem**

```jsx
function Score() {
  const [score, setScore] = useState(0);
  
  function handleClick() {
    // All three calls use score = 0 from this render
    setScore(score + 1); // setScore(0 + 1)
    setScore(score + 1); // setScore(0 + 1)
    setScore(score + 1); // setScore(0 + 1)
    // Result: score becomes 1, not 3
  }
  
  return <button onClick={handleClick}>Score: {score}</button>;
}
```

**Expected Output (clicking once)**

```
Score: 1
```

**Why This Output Occurs**

State is a snapshot. In the current render, `score` is `0`. All three `setScore(score + 1)` calls compute `0 + 1 = 1`. React processes these as three requests to set the same value (1), resulting in a single update to 1 .

**Example 2: Updater Functions Fix the Problem**

```jsx
function Score() {
  const [score, setScore] = useState(0);
  
  function handleClick() {
    // Each updater receives the pending state
    setScore(prev => prev + 1); // 0 + 1 = 1
    setScore(prev => prev + 1); // 1 + 1 = 2
    setScore(prev => prev + 1); // 2 + 1 = 3
    // Result: score becomes 3
  }
  
  return <button onClick={handleClick}>Score: {score}</button>;
}
```

**Expected Output (clicking once)**

```
Score: 3
```

**Why This Output Occurs**

When passing an updater function, React queues the functions and calls them in order with the latest pending state. The first receives `0`, returns `1`. The second receives `1`, returns `2`. The third receives `2`, returns `3`. The final state is `3` .

**Example 3: Updating Objects in State**

```jsx
function Profile() {
  const [user, setUser] = useState({ name: 'Alice', age: 30 });
  
  function birthday() {
    // Create a new object with the updated age
    setUser({
      ...user,       // Copy all properties
      age: user.age + 1  // Override age
    });
  }
  
  return (
    <div>
      <p>{user.name} is {user.age}</p>
      <button onClick={birthday}>Birthday!</button>
    </div>
  );
}
```

**Expected Output (after two clicks)**

```
Alice is 32
```

**Why This Output Occurs**

The spread operator creates a new object with all properties from `user`, then overrides `age`. React sees a new object reference and triggers a re-render. Mutating `user.age` directly would not work because React compares references .

### Real-World Cases

**Case: Multi-Step Form**

A wizard form uses updater functions to ensure each step's data is correctly accumulated in a single state object, avoiding lost updates when multiple fields change rapidly.

**Case: Shopping Cart**

Adding items to a cart uses `setCart(prev => [...prev, newItem])` to ensure the new item is added to the latest cart state, not a stale snapshot .

### References

- React — State as a Snapshot - https://react.dev/learn/state-as-a-snapshot
- React — Queueing a Series of State Updates - https://react.dev/learn/queueing-a-series-of-state-updates
- React — Updating Objects in State - https://react.dev/learn/updating-objects-in-state

---

## 4. State Persistence Across Renders

### Definitions

**Core Definition**

State persistence across renders means that state values survive component re-renders, remaining available and unchanged unless explicitly updated.

**Technical Definition**

React stores state externally from the component function in its internal Fiber tree. When a component re-renders (due to state or parent changes), React retrieves the stored state values and provides them to the component function as the current state. This is why state variables "remember" their values between renders, unlike regular variables that are reinitialized on every function call.

**Beginner-Friendly Explanation**

State persistence is why your form input doesn't lose what you typed when the component re-renders for another reason. The state is stored safely outside the component, and React brings it back each time the component runs.

### Purposes

- To retain user input and interaction state during re-renders.
- To maintain component state across parent-triggered updates.
- To allow state to accumulate and change over the component's lifetime.
- To distinguish state from regular variables that reset on every render.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
function Component() {
  // This variable resets every render
  const regularVar = 0;
  
  // This state persists across renders
  const [persistentState, setPersistentState] = useState(0);
  
  return <div>{regularVar} {persistentState}</div>;
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `regularVar` | Reinitialized to 0 on every render. Changes do not persist. |
| `persistentState` | Retained by React across renders. Only changes when setter is called. |

**Syntax Rules**

1. **State lives in React**: State is stored in React's internal data structures, not in the component function's local scope .
2. **Re-render preserves state**: Calling a setter triggers a re-render, but the new value is provided to that render.
3. **Unmount destroys state**: When a component is removed from the tree, its state is destroyed and will be reinitialized if the component remounts .
4. **Position matters**: React preserves state based on the component's position in the tree. Reordering can cause state loss .

**Constraints and Limitations**

- State is destroyed when the component unmounts.
- Changing a component's position in the tree (or key) resets its state.
- Regular variables do not persist across renders; only state and refs do.

### Annotated Complete Code Examples

**Example 1: State Persists, Variables Do Not**

```jsx
function Demo() {
  const [count, setCount] = useState(0);
  let regularVar = 0;
  
  // This runs on every render
  console.log('Render:', { count, regularVar });
  
  function handleClick() {
    regularVar += 1; // Changes local variable, lost on next render
    setCount(count + 1); // Changes state, persists
    console.log('After click:', { count, regularVar });
  }
  
  return <button onClick={handleClick}>Count: {count}</button>;
}
```

**Expected Output (clicking once)**

```
Render: { count: 0, regularVar: 0 }
After click: { count: 0, regularVar: 1 }
Render: { count: 1, regularVar: 0 }  ← regularVar reset!
```

**Why This Output Occurs**

`regularVar` is recreated on every render, always starting at 0. The increment inside `handleClick` only affects the local variable for that render. `count` is stored by React and persists: the next render provides `count` as 1 .

**Example 2: State Reset on Unmount**

```jsx
function Child() {
  const [input, setInput] = useState('');
  return (
    <input 
      value={input} 
      onChange={e => setInput(e.target.value)} 
      placeholder="Type here..."
    />
  );
}

function App() {
  const [show, setShow] = useState(true);
  
  return (
    <div>
      <button onClick={() => setShow(!show)}>
        {show ? 'Hide' : 'Show'}
      </button>
      {show && <Child />}
    </div>
  );
}
```

**Expected Behavior**

Typing in the input and then hiding/showing the child resets the input.

**Why This Occurs**

When `show` becomes `false`, `<Child />` is removed from the tree. React destroys the `Child` component and its state. When `show` becomes `true` again, a new `Child` instance is created with fresh state .

### Real-World Cases

**Case: Preserving Form Drafts**

State persistence ensures that if a parent re-renders (e.g., due to a theme change), the user's form input is not lost. The input state survives the re-render.

**Case: Resetting State with Key**

A chat application uses `key={recipientId}` on the `Chat` component so that switching recipients resets the input field, preventing messages from being sent to the wrong person .

### References

- React — Preserving and Resetting State - https://react.dev/learn/preserving-and-resetting-state
- React — State as a Snapshot - https://react.dev/learn/state-as-a-snapshot

---

## 5. Derived State

### Definitions

**Core Definition**

Derived state is data that can be computed from existing state or props during render, rather than being stored as separate state.

**Technical Definition**

Derived state is a value calculated from current props or state, not held in its own `useState` variable. Storing derived values in state creates a "source of truth" problem: two state variables can become out of sync. The recommended approach is to compute derived values directly during render, or memoize them with `useMemo` for expensive calculations.

**Beginner-Friendly Explanation**

Derived state is like calculating a total from a list of prices. You don't need to store the total separately—you can calculate it whenever you need it. Storing it separately risks the total becoming wrong if you forget to update it.

### Purposes

- To avoid redundant state variables that can become out of sync.
- To keep a single source of truth for data.
- To simplify component logic by computing values on demand.
- To improve performance by memoizing expensive calculations.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
// ❌ Avoid: redundant state
const [firstName, setFirstName] = useState('');
const [lastName, setLastName] = useState('');
const [fullName, setFullName] = useState(''); // Redundant!

// ✅ Prefer: compute during render
const [firstName, setFirstName] = useState('');
const [lastName, setLastName] = useState('');
const fullName = firstName + ' ' + lastName; // Derived
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `firstName`, `lastName` | Source state variables. |
| `fullName` | Derived value computed during render. |

**Syntax Rules**

1. **Single source of truth**: Store only the minimal state needed. Compute everything else .
2. **Compute during render**: Simple derivations should be computed directly in the component body.
3. **Memoize expensive calculations**: Use `useMemo` for computationally expensive derivations .
4. **Avoid effect-based derivation**: Do not use `useEffect` to sync derived state—this causes cascading updates and extra renders .

**Constraints and Limitations**

- Derived values are recomputed on every render unless memoized.
- `useMemo` is a performance optimization, not a semantic guarantee; it may be discarded by React.

### Annotated Complete Code Examples

**Example 1: Avoid Redundant State**

```jsx
// ❌ Problematic: two sources of truth
function BadForm() {
  const [firstName, setFirstName] = useState('Barbara');
  const [lastName, setLastName] = useState('Hepworth');
  const [fullName, setFullName] = useState('Barbara Hepworth');
  
  function handleFirstNameChange(e) {
    setFirstName(e.target.value);
    setFullName(e.target.value + ' ' + lastName); // Must remember to update!
  }
  
  // ...
}

// ✅ Better: single source of truth
function GoodForm() {
  const [firstName, setFirstName] = useState('Barbara');
  const [lastName, setLastName] = useState('Hepworth');
  
  // Computed during render — always correct
  const fullName = firstName + ' ' + lastName;
  
  // ...
}
```

**Explanation**

The bad version stores `fullName` separately, requiring manual synchronization every time `firstName` or `lastName` changes. The good version computes `fullName` during render, eliminating the possibility of it being out of sync .

**Example 2: Derived State from Props**

```jsx
// ❌ Problematic: state derived from prop
function EmailInput({ user }) {
  const [email, setEmail] = useState(user.email);
  
  // If user.email changes, state is not updated
  // ...
}

// ✅ Better: reset with key
function App() {
  return <EmailInput key={user.id} user={user} />;
}

function EmailInput({ user }) {
  const [email, setEmail] = useState(user.email);
  // When user.id changes, key changes, component resets
  // ...
}
```

**Explanation**

When state is derived from a prop that can change, using state to "remember" the initial value causes synchronization problems. The recommended solution is to use a `key` that changes when the source data changes, forcing a component reset .

### Real-World Cases

**Case: Filtered Lists**

A component holds a list of items in state and a filter term in state. The filtered list is computed during render: `const visibleItems = items.filter(item => item.name.includes(filter))`. No separate state is needed for `visibleItems`.

**Case: Total Price Calculation**

A shopping cart stores items in state. The total price is derived: `const total = items.reduce((sum, item) => sum + item.price, 0)`. Storing the total separately would require updates on every item change.

### References

- React — Managing State (Avoiding Redundant State) - https://react.dev/learn/managing-state
- React — You Probably Don't Need Derived State - https://reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html

---

## 6. State Ownership

### Definitions

**Core Definition**

State ownership is the principle that each piece of state should have a single, clearly defined component responsible for storing and updating it.

**Technical Definition**

State ownership means determining which component "owns" a given piece of state—the component where the state is declared and whose setter function controls updates. Ownership is determined by the closest common ancestor of all components that need to read or modify that state. The owner is the "source of truth" for that data.

**Beginner-Friendly Explanation**

State ownership is like deciding which family member keeps the car keys. If two people need to use the car, one person should be responsible for the keys, and they coordinate with the other. In React, if two components need the same data, one component (the owner) should hold the state and share it.

### Purposes

- To establish a single source of truth for each piece of data.
- To avoid synchronization bugs from duplicate state.
- To clarify which component is responsible for updates.
- To determine when state lifting is necessary.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
// State owner: the component that declares the state
function Owner() {
  const [sharedState, setSharedState] = useState(initialValue);
  
  // Pass state down to children
  return (
    <div>
      <ChildA value={sharedState} />
      <ChildB onChange={setSharedState} />
    </div>
  );
}

// Children: receive state via props
function ChildA({ value }) {
  return <p>{value}</p>;
}

function ChildB({ onChange }) {
  return <button onClick={() => onChange(newValue)}>Update</button>;
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `Owner` | Declares and owns the state. |
| `ChildA` | Reads state via props. |
| `ChildB` | Updates state via callback prop. |

**Syntax Rules**

1. **Closest common ancestor**: The owner should be the closest component that is an ancestor of all components needing the state .
2. **Single owner**: Each piece of state should have exactly one owner.
3. **Props flow down**: The owner passes state down as props.
4. **Callbacks flow up**: The owner passes setters or callbacks down for children to request changes.

**Constraints and Limitations**

- Identifying the correct owner requires understanding which components need the state.
- If the owner is too high, unnecessary prop drilling may occur.
- If the owner is too low, state cannot be shared.

### Annotated Complete Code Examples

**Example 1: Determining Ownership**

```jsx
// ❌ Problem: Two components need the same data, but state is in one child
function Accordion() {
  return (
    <div>
      <Panel title="First" />
      <Panel title="Second" />
    </div>
  );
}

function Panel({ title }) {
  const [isActive, setIsActive] = useState(false); // Only this panel knows
  // Panels cannot coordinate — each has independent state
}

// ✅ Solution: Lift state to the common parent
function Accordion() {
  const [activeIndex, setActiveIndex] = useState(0); // Owner is Accordion
  
  return (
    <div>
      <Panel
        title="First"
        isActive={activeIndex === 0}
        onShow={() => setActiveIndex(0)}
      />
      <Panel
        title="Second"
        isActive={activeIndex === 1}
        onShow={() => setActiveIndex(1)}
      />
    </div>
  );
}
```

**Explanation**

In the problematic version, each `Panel` owns its own `isActive` state, so they cannot coordinate. The solution moves ownership to `Accordion`, which becomes the source of truth for which panel is active .

### Real-World Cases

**Case: Shopping Cart**

An e-commerce app has a cart icon in the header and an "Add to Cart" button in product cards. The cart state must be owned by a common ancestor (e.g., `App` or a context provider) so both can access and update it.

**Case: Theme Toggle**

A theme toggle in the settings page affects the entire app. The theme state must be owned by a top-level component or context provider.

### References

- React — Sharing State Between Components - https://react.dev/learn/sharing-state-between-components
- React — Managing State (Choosing the State Structure) - https://react.dev/learn/managing-state

---

## 7. State Lifting

### Definitions

**Core Definition**

State lifting is the process of moving state from a child component up to its closest common parent so that multiple components can share and coordinate that state.

**Technical Definition**

Lifting state involves three steps: (1) remove state from child components, (2) pass data from the parent down as props, and (3) add state to the parent and pass it down along with event handlers. This makes the parent the "source of truth" for the shared state and allows sibling components to coordinate.

**Beginner-Friendly Explanation**

State lifting is like moving a toy from one child's room to the living room so both children can play with it together. The child who owned the toy gives it up, and the parent manages it, letting both children use it.

### Purposes

- To enable multiple components to share and coordinate state.
- To establish a single source of truth for shared data.
- To allow sibling components to communicate through a common parent.
- To fix synchronization bugs caused by duplicate state.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
// Before lifting: each child owns its own state
function Child() {
  const [value, setValue] = useState(false);
  return <button onClick={() => setValue(!value)}>{value ? 'ON' : 'OFF'}</button>;
}

// After lifting: parent owns state, children receive props
function Parent() {
  const [value, setValue] = useState(false);
  
  return (
    <div>
      <Child value={value} onChange={setValue} />
      <OtherChild value={value} />
    </div>
  );
}

function Child({ value, onChange }) {
  return <button onClick={() => onChange(!value)}>{value ? 'ON' : 'OFF'}</button>;
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `Parent` | New owner of the lifted state. |
| `Child` | Receives `value` and `onChange` as props; no longer owns state. |

**Syntax Rules**

1. **Remove state from children**: Delete `useState` from the child components .
2. **Pass data down**: The parent passes the state value to children as props.
3. **Pass callbacks down**: The parent passes event handlers that update its state.
4. **Closest common parent**: Lift to the closest ancestor of all components that need the state .

**Constraints and Limitations**

- Lifting state too high can cause prop drilling through components that don't need the state.
- For deeply nested trees, Context or external state management may be preferable.
- Lifting state changes the "nature" of what is stored (e.g., from boolean per panel to active index).

### Annotated Complete Code Examples

**Example 1: Lifting State for Coordinated Panels**

```jsx
// Before: independent panels
function Panel({ title, children }) {
  const [isActive, setIsActive] = useState(false);
  return (
    <section>
      <h3>{title}</h3>
      {isActive ? (
        <p>{children}</p>
      ) : (
        <button onClick={() => setIsActive(true)}>Show</button>
      )}
    </section>
  );
}

// After: lifted state to parent
function Accordion() {
  const [activeIndex, setActiveIndex] = useState(0);
  
  return (
    <>
      <Panel
        title="About"
        isActive={activeIndex === 0}
        onShow={() => setActiveIndex(0)}
      >
        Content for About
      </Panel>
      <Panel
        title="Etymology"
        isActive={activeIndex === 1}
        onShow={() => setActiveIndex(1)}
      >
        Content for Etymology
      </Panel>
    </>
  );
}

function Panel({ title, children, isActive, onShow }) {
  return (
    <section>
      <h3>{title}</h3>
      {isActive ? (
        <p>{children}</p>
      ) : (
        <button onClick={onShow}>Show</button>
      )}
    </section>
  );
}
```

**Expected Behavior**

Clicking "Show" on one panel collapses the other. Only one panel is active at a time.

**Why This Occurs**

State was lifted from `Panel` to `Accordion`. `Accordion` now owns `activeIndex` (0 or 1). Each `Panel` receives `isActive` (whether it's the active one) and `onShow` (a callback to make itself active). The parent coordinates the state, so only one panel can be active .

### Real-World Cases

**Case: Tabs Component**

A `Tabs` component owns the `activeTab` state. Each `Tab` receives `isActive` and an `onSelect` callback. Clicking a tab updates the parent's state, switching the visible content.

**Case: Form with Multiple Sections**

A form owns the `formData` state. Each form section receives its slice of data and an `onChange` callback. This centralizes validation and submission logic.

### References

- React — Sharing State Between Components (Lifting State Up) - https://react.dev/learn/sharing-state-between-components
- React — Managing State (Sharing State Between Components) - https://react.dev/learn/managing-state

---

## References

- React Official Documentation — State: A Component's Memory - https://react.dev/learn/state-a-components-memory
- React Official Documentation — State as a Snapshot - https://react.dev/learn/state-as-a-snapshot
- React Official Documentation — Queueing a Series of State Updates - https://react.dev/learn/queueing-a-series-of-state-updates
- React Official Documentation — Updating Objects in State - https://react.dev/learn/updating-objects-in-state
- React Official Documentation — Updating Arrays in State - https://react.dev/learn/updating-arrays-in-state
- React Official Documentation — Preserving and Resetting State - https://react.dev/learn/preserving-and-resetting-state
- React Official Documentation — Sharing State Between Components - https://react.dev/learn/sharing-state-between-components
- React Official Documentation — Managing State - https://react.dev/learn/managing-state
- React Official Documentation — useState API Reference - https://react.dev/reference/react/useState
- React Blog — You Probably Don't Need Derived State - https://reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html
- The Road to React — Local Component State (GitHub) - https://github.com/the-road-to-learn-react/the-road-to-learn-react
- Mimo — React State Management Glossary - https://mimo.org/glossary/react/state-management