# React Child-to-Parent Communication: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** React child-to-parent communication is the set of patterns and techniques by which a child component sends data, signals, or user intent upward to its parent component, enabling bidirectional interaction within React's fundamentally unidirectional data flow architecture.

**Technical Definition:** React enforces a unidirectional data flow: props pass data downward from parent to child, and there is no direct API for a child to mutate a parent's state. Child-to-parent communication is therefore achieved indirectly through callback functions passed as props. When a child invokes a callback prop, it executes a function defined in the parent's scope, allowing the parent to update its own state, perform side effects, or coordinate sibling components. This pattern is foundational to the "lifting state up" technique, where state is moved to the closest common ancestor of components that need to share it, and children are given controlled access to that state via props and callbacks. Two closely related patterns extend this foundation: event lifting (capturing low-level DOM events in the child and mapping them to domain-meaningful events for the parent) and state ownership (defining state in the parent to maintain a single source of truth while giving the child controlled access).

**Beginner-Friendly Explanation:** In React, data flows downhill—parents pass information to children, but children cannot directly change their parents' data. So how does a child tell its parent something? It uses a callback function. The parent says to the child: "Here's a function. Call it when you need to tell me something." The child calls the function with the relevant data, and the parent's function runs, updating the parent's state. This is how children "talk back" to parents in React.

### Key Characteristics

- **Unidirectional Data Flow:** Props flow down; callbacks flow up. The parent always remains the owner of its state.
- **Callback Props as the Primary Mechanism:** Functions passed from parent to child are the standard way for children to communicate upward.
- **Lifting State Up:** When multiple components need to share state, it is moved to their closest common ancestor, which then passes it down via props and provides callbacks for updates.
- **Single Source of Truth:** Each piece of state has exactly one owner—the component that holds the `useState` (or `useReducer`) call.
- **Controlled Components:** A child that receives its value from a prop and reports changes via a callback is a "controlled component"; its state is owned by the parent.
- **Event Lifting:** DOM events (clicks, input changes) are captured in the child and mapped to domain-meaningful callbacks that the parent can interpret.
- **Explicit Communication:** All child-to-parent communication is explicit—there is no implicit event bubbling or two-way binding.

### Prerequisites

- Solid understanding of JavaScript functions, closures, and callbacks.
- Familiarity with React function components, JSX, and the `useState` Hook.
- Working knowledge of the props system and component tree hierarchy.
- Understanding of React's render cycle and how state updates trigger re-renders.

### Related Programming Areas

- **State Management:** Coordinating state across multiple components.
- **Component Composition:** Building complex UIs from smaller, reusable pieces.
- **Controlled vs. Uncontrolled Components:** Patterns for form inputs and interactive widgets.
- **Prop Drilling:** The challenge of passing callbacks through many intermediate components.
- **Context API:** A solution for avoiding prop drilling when callbacks must travel deep into the tree.

### Core Concepts / Features

1. Callback Props (Invoking Parent-Supplied Functions from the Child)
2. Event Lifting (Mapping DOM Events to Domain Events for the Parent)
3. State Ownership (Defining State in the Parent for a Single Source of Truth)

---

## Core Concept 1: Callback Props (Invoking Parent-Supplied Functions from the Child)

### Definitions

**Core Definition:** A callback prop is a function defined in a parent component and passed down to a child component as a prop, which the child invokes to send data upward, report events, or trigger parent-defined actions.

**Technical Definition:** Callback props are the mechanism by which data flows upward in React's otherwise unidirectional architecture. A parent component defines a function (e.g., `handleSubmit`, `onItemSelect`) and passes it as a prop to a child. The child component calls this function—typically inside an event handler—to communicate back to the parent. React's official documentation states that when a child needs to communicate with its parent, it can use a callback function passed down as a prop. The child calls this function to notify the parent of an event or to send data back up. By convention, callback props are named with an `on` prefix (e.g., `onClick`, `onSubmit`, `onSelect`), and the parent's handler function is named with a `handle` prefix (e.g., `handleClick`, `handleSubmit`). The callback receives whatever arguments the child passes to it, allowing the parent to process the data in its own scope.

**Beginner-Friendly Explanation:** Imagine a child component is a TV remote and the parent is the TV. The parent gives the child a set of buttons (callback functions) and says: "Press this one to tell me to change the channel, press that one to tell me to adjust the volume." The child can't change the channel itself—it can only press the button, which triggers the parent to do the work. This keeps the TV (parent) in control of its own state, while the remote (child) provides the interface.

### Purposes

- To allow a child component to report events (clicks, form submissions, selections) back to the parent.
- To enable the parent to update its state in response to child-triggered actions.
- To lift state up to the closest common ancestor so multiple children can share and update it.
- To keep the parent in control of business logic and data mutations while the child focuses on presentation.
- To create reusable child components that do not need to know how the parent will respond to an action—only that it should be notified.
- To provide a clean separation of concerns: the child handles UI interaction, the parent handles data and side effects.

### Syntax Rules and Structure

**General Syntax (Parent Defines and Passes Callback):**
```jsx
function ParentComponent() {
  // Parent defines the callback handler
  function handleChildAction(dataFromChild) {
    // Parent logic: update state, make API calls, etc.
    console.log('Child reported:', dataFromChild);
  }

  // Pass the function as a prop (conventionally prefixed with "on")
  return <ChildComponent onAction={handleChildAction} />;
}
```

**Component Breakdown:**
- `handleChildAction(dataFromChild)`: The callback function, defined inside the parent.
- `onAction={handleChildAction}`: The callback is passed as a prop named `onAction`.

**General Syntax (Child Invokes the Callback):**
```jsx
function ChildComponent({ onAction }) {
  // Child defines an event handler that calls the parent's callback
  function handleClick() {
    const data = 'Some data from the child';
    onAction(data); // Invoke the callback with the data
  }

  return <button onClick={handleClick}>Trigger Parent Action</button>;
}
```

**Component Breakdown:**
- `{ onAction }`: Destructures the callback prop from the props object.
- `onAction(data)`: Calls the parent's function, passing the data upward.
- `onClick={handleClick}`: The child's own event handler that invokes the callback.

**Syntax Rules:**
- Name callback props with an `on` prefix (`onClick`, `onSubmit`, `onSelect`) and the handler function with a `handle` prefix (`handleClick`, `handleSubmit`) for clarity.
- The child should call the callback with the necessary data as arguments: `onAction(payload)`.
- The parent's callback should be defined as a function inside the component body (or memoised with `useCallback` if passed to a memoised child).
- If the callback needs to access the parent's current state, define it inside the parent component so it closes over the latest state.
- When passing a callback that takes no arguments, pass the function reference directly: `onClick={handleClick}` (not `onClick={handleClick()}`).
- For callbacks that need to pass additional context (e.g., an item ID), use an arrow function: `onClick={() => onSelect(item.id)}`.

**Constraints and Limitations:**
- Callbacks create a new function reference on every render of the parent; this can cause unnecessary re-renders in memoised children unless `useCallback` is used.
- Deeply nested callback chains (callback passed through many intermediate components) become difficult to maintain—this is the "prop drilling" problem.
- The child cannot know what the parent will do with the callback, so error handling and side effects belong entirely to the parent.
- If the parent unmounts before the callback is invoked, invoking the callback can cause a state update on an unmounted component (though React 18+ no longer warns about this).

### Annotated Code Examples

**Example 1: Form Submission with Callback Prop**

```jsx
// Parent component: holds the submission logic
function ParentComponent() {
  // Parent's callback: receives form data from the child
  function handleFormSubmit(formData) {
    console.log('Form submitted with:', formData);
    alert(`Form submitted with: ${JSON.stringify(formData)}`);
  }

  return (
    <div>
      <h1>Submit Form</h1>
      {/* Pass the callback as the onSubmit prop */}
      <FormInput onSubmit={handleFormSubmit} />
    </div>
  );
}

// Child component: captures input and triggers the parent's callback
function FormInput({ onSubmit }) {
  const [name, setName] = React.useState('');
  const [email, setEmail] = React.useState('');

  function handleFormSubmit() {
    if (name && email) {
      // Invoke the parent's callback with the form data
      onSubmit({ name, email });
    } else {
      alert('Please fill in all fields!');
    }
  }

  return (
    <div>
      <label>
        Name: <input type="text" value={name} onChange={(e) => setName(e.target.value)}/>
      </label>
      <label>
        Email: <input type="email" value={email} onChange={(e) => setEmail(e.target.value)}/>
      </label>
      <br />
      <button onClick={handleFormSubmit}>Submit</button>
    </div>
  );
}
```

**Expected Output:** The form displays name and email inputs and a "Submit" button. When both fields are filled and the button is clicked, an alert shows "Form submitted with: {"name":"...","email":"..."}". If either field is empty, an alert says "Please fill in all fields!".

**Why This Output Occurs:** The `FormInput` child component manages its own local input state (`name`, `email`) but does not know what happens after submission. When the user clicks "Submit", the child calls `onSubmit({ name, email })`—the callback prop passed from the parent. The parent's `handleFormSubmit` function receives this data and performs the actual submission logic (logging and alerting).

**Example 2: Lifting State Up with Callback Props**

```jsx
// Parent component: holds the shared state
function TemperatureApp() {
  const [temperature, setTemperature] = React.useState(20);

  return (
    <div>
      <h1>Temperature: {temperature}°C</h1>
      {/* Pass the state value and a callback to update it */}
      <TemperatureInput
        temperature={temperature}
        onTemperatureChange={setTemperature}
      />
    </div>
  );
}

// Child component: controlled input that reports changes to the parent
function TemperatureInput({ temperature, onTemperatureChange }) {
  function handleChange(e) {
    // Call the parent's callback with the new value
    onTemperatureChange(Number(e.target.value));
  }

  return (
    <div>
      <label>
        Set temperature:
        <input
          type="number"
          value={temperature}
          onChange={handleChange}
        />
      </label>
    </div>
  );
}
```

**Expected Output:** The parent displays the current temperature (initially 20°C). The child renders a number input pre-filled with 20. When the user types a new number, the parent's state updates and the displayed temperature changes in real time.

**Why This Output Occurs:** The `temperature` state lives in the parent (`TemperatureApp`), making it the "single source of truth." The parent passes both the current value (`temperature`) and a callback (`onTemperatureChange`, which is `setTemperature`) to the child. When the user types in the input, the child calls `onTemperatureChange(newValue)`, which updates the parent's state. The parent re-renders with the new temperature, passing the updated value back down to the child. This is the "lifting state up" pattern.

### Real-World Cases

- **Modal components:** A parent passes an `onClose` callback to a modal child; the child calls it when the user clicks the close button or presses Escape.
- **Dropdown/Select components:** A parent passes `onSelect` to a dropdown child; the child calls it with the selected option when the user makes a choice.
- **Todo lists:** A parent passes `onToggle` and `onDelete` callbacks to each todo item child; the child calls the appropriate callback when the user interacts with the item.
- **Pagination:** A parent passes `onPageChange` to a pagination child; the child calls it with the new page number when the user clicks a page button.
- **Search bars:** A parent passes `onSearch` to a search input child; the child calls it with the query when the user submits or types.

### References

- React Official Documentation – Sharing State Between Components: https://react.dev/learn/sharing-state-between-components
- React Official Documentation – Responding to Events: https://react.dev/learn/responding-to-events
- React Legacy Documentation – Passing Functions to Components: https://legacy.reactjs.org/docs/faq-functions.html
- DEV Community – Callback Props in React: https://dev.to/ali007depug/callback-props-in-react-37m0
- JavaScript Plain English – Communication Patterns in React: https://javascript.plainenglish.io/communication-patterns-in-react-30df2de702eb

---

## Core Concept 2: Event Lifting (Mapping DOM Events to Domain Events for the Parent)

### Definitions

**Core Definition:** Event lifting is the pattern of capturing low-level DOM events (clicks, input changes, key presses) inside a child component and mapping them to higher-level, domain-meaningful events that the parent component can handle.

**Technical Definition:** Event lifting is a design pattern in React where a child component abstracts the raw DOM event handling and exposes a semantic callback to the parent. Instead of the parent attaching event listeners directly to DOM elements (which it often cannot do, since the elements live inside the child's JSX), the child listens for DOM events and, when appropriate, invokes a callback prop with domain-relevant data. This separates the concerns of DOM interaction (the child's responsibility) from business logic (the parent's responsibility). The pattern is closely related to lifting state up—when a child needs to communicate a meaningful event upward, the state associated with that event is often lifted to the parent, and the child becomes a controlled component that reports changes via callbacks. The child maps something like `onKeyUp` or `onChange` to a more meaningful `onSearch`, `onSubmit`, or `onItemSelect` callback.

**Beginner-Friendly Explanation:** A child component might be a search input. The DOM fires a `keyup` event every time the user types a letter, but the parent doesn't care about individual keystrokes—it cares about "the user searched for something." So the child listens to the raw `keyup` events, and when the user presses Enter (or after a debounce), it calls the parent's `onSearch` callback with the final query. The child translates "low-level DOM events" into "high-level domain events" that the parent understands.

### Purposes

- To abstract away DOM-specific event handling from the parent component.
- To expose meaningful, domain-level events (e.g., `onSearch`, `onSubmit`) to the parent instead of raw DOM events.
- To allow the child component to control when and how a domain event is triggered (e.g., debouncing, validation, or Enter-key detection).
- To keep the parent's code focused on business logic rather than DOM event details.
- To enable the reuse of the child component with different parent-specific event handling logic.
- To simplify the parent's API by providing a clean, semantic callback interface.

### Syntax Rules and Structure

**General Syntax (Child Maps DOM Event to Domain Callback):**
```jsx
function SearchInput({ onSearch }) {
  const [query, setQuery] = useState('');

  // Child handles the low-level DOM event
  function handleKeyDown(event) {
    if (event.key === 'Enter') {
      // Map to a high-level domain event
      onSearch(query);
    }
  }

  return (
    <input
      value={query}
      onChange={(e) => setQuery(e.target.value)}
      onKeyDown={handleKeyDown}
      placeholder="Type and press Enter to search"
    />
  );
}
```

**Component Breakdown:**
- `onSearch`: The domain-level callback prop exposed to the parent.
- `handleKeyDown(event)`: The child's internal DOM event handler.
- `if (event.key === 'Enter')`: The condition that maps the DOM event to the domain event.
- `onSearch(query)`: Invokes the parent's callback with the domain-relevant data.

**General Syntax (Parent Handles the Domain Event):**
```jsx
function ParentComponent() {
  function handleSearch(query) {
    // Parent logic: fetch results, update URL, etc.
    console.log('Searching for:', query);
  }

  return <SearchInput onSearch={handleSearch} />;
}
```

**Component Breakdown:**
- `handleSearch(query)`: The parent's domain-level handler.
- `onSearch={handleSearch}`: The parent passes its handler to the child.

**Syntax Rules:**
- Name the domain callback after the *meaning* of the event, not the DOM event that triggered it (e.g., `onSearch` rather than `onKeyDown` or `onInputChange`).
- The child should handle the raw DOM event and decide when to invoke the domain callback (e.g., on Enter, on blur, after debounce).
- The child should pass domain-relevant data to the callback (e.g., the final query string, not the DOM event object).
- If the parent needs the raw DOM event (rare), the child can pass it as a second argument: `onSearch(query, event)`.
- When the child is a controlled component, it receives its value from the parent and reports changes via a domain callback (e.g., `onTemperatureChange`).
- Use `event.preventDefault()` or `event.stopPropagation()` in the child only when the child's domain logic requires it; avoid leaking DOM concerns upward.

**Constraints and Limitations:**
- The child takes on the responsibility of interpreting DOM events, which may require additional logic (debouncing, validation, keyboard handling).
- If the child exposes too many domain callbacks, its interface becomes complex; consider consolidating related events.
- The parent loses direct access to the raw DOM event, which can be limiting in rare cases (e.g., accessing `event.target` properties not included in the domain payload).
- The pattern can be overkill for simple cases where the parent could pass a simple `onClick` callback directly.

### Annotated Code Examples

**Example 1: Search Input That Maps Key Events to `onSearch`**

```jsx
import { useState } from 'react';

// Child component: captures raw keyboard events and maps them to a domain event
function SearchInput({ onSearch, placeholder }) {
  const [query, setQuery] = useState('');

  function handleKeyDown(event) {
    // Map the DOM event (keydown) to the domain event (search)
    if (event.key === 'Enter' && query.trim()) {
      onSearch(query.trim()); // Pass the domain-relevant data upward
      setQuery(''); // Clear the input after search
    }
  }

  function handleChange(event) {
    setQuery(event.target.value); // Update local state on each keystroke
  }

  return (
    <input
      type="text"
      value={query}
      onChange={handleChange}
      onKeyDown={handleKeyDown}
      placeholder={placeholder ?? 'Search...'}
    />
  );
}

// Parent component: handles the domain-level search event
function SearchPage() {
  const [results, setResults] = useState([]);

  function handleSearch(query) {
    // Parent performs the actual search logic
    console.log('Searching for:', query);
    // Simulate fetching results
    setResults([`Result for "${query}" 1`, `Result for "${query}" 2`]);
  }

  return (
    <div>
      <h1>Search</h1>
      <SearchInput onSearch={handleSearch} placeholder="Search products..." />
      <ul>
        {results.map((r, i) => (
          <li key={i}>{r}</li>
        ))}
      </ul>
    </div>
  );
}
```

**Expected Output:** A search input is displayed with the placeholder "Search products...". When the user types a query and presses Enter, the input clears and a list of simulated results appears below (e.g., "Result for 'shoes' 1", "Result for 'shoes' 2").

**Why This Output Occurs:** The `SearchInput` child listens to the raw `keydown` and `change` DOM events. When the user presses Enter with a non-empty query, the child invokes `onSearch(query.trim())`, passing the domain-relevant data (the search query) to the parent. The parent's `handleSearch` function receives this data and performs the actual search logic (simulated here with `setResults`).

**Example 2: Rating Widget That Maps Click Events to `onRatingChange`**

```jsx
// Child component: maps clicks on star elements to a domain-level rating change
function StarRating({ rating, onRatingChange, maxStars = 5 }) {
  function handleStarClick(value) {
    // Map the DOM click event to the domain event
    onRatingChange(value);
  }

  return (
    <div className="star-rating">
      {Array.from({ length: maxStars }, (_, i) => {
        const starValue = i + 1;
        return (
          <span
            key={starValue}
            className={starValue <= rating ? 'star filled' : 'star'}
            onClick={() => handleStarClick(starValue)}
            style={{ cursor: 'pointer', fontSize: '24px' }}
          >
            ★
          </span>
        );
      })}
    </div>
  );
}

// Parent component: handles the domain-level rating change
function ReviewForm() {
  const [rating, setRating] = useState(0);

  function handleRatingChange(newRating) {
    console.log('Rating changed to:', newRating);
    setRating(newRating);
  }

  return (
    <div>
      <h2>Rate this product</h2>
      <StarRating rating={rating} onRatingChange={handleRatingChange} />
      <p>You rated: {rating} star{rating !== 1 ? 's' : ''}</p>
    </div>
  );
}
```

**Expected Output:** Five star characters are displayed. Clicking the third star fills the first three stars and displays "You rated: 3 stars". Clicking the fifth star fills all five and displays "You rated: 5 stars".

**Why This Output Occurs:** The `StarRating` child maps each star's `onClick` DOM event to a domain-level `onRatingChange(value)` callback. The parent receives the numeric rating value, not the raw click event, and updates its state accordingly. This keeps the parent's code focused on the domain logic ("what does a rating change mean?") rather than DOM details ("which star was clicked?").

### Real-World Cases

- **Search components:** A child search input maps `keydown`/`change` events to `onSearch(query)` for the parent.
- **Form libraries:** A form child component maps `input` and `blur` events to `onFieldChange(name, value)` and `onFieldBlur(name)` domain events.
- **File uploaders:** A child drop zone maps `dragover`/`drop` DOM events to `onFileDrop(files)` for the parent.
- **Date pickers:** A child calendar maps `click` events on date cells to `onDateSelect(date)` domain events.
- **Rating widgets:** A child star-rating component maps star clicks to `onRatingChange(value)`.
- **Tabs:** A child tab list maps clicks to `onTabChange(tabId)` for the parent to switch the active panel.

### References

- React Official Documentation – Responding to Events: https://react.dev/learn/responding-to-events
- React Official Documentation – Sharing State Between Components: https://react.dev/learn/sharing-state-between-components
- React Legacy Documentation – Lifting State Up: https://legacy.reactjs.org/docs/lifting-state-up.html
- Stack Overflow – Lifting State Up (Event Handler Binding): https://stackoverflow.com/revisions/87e1cf40-e2f3-4a3f-bd5d-d987c2a77da3/view-source

---

## Core Concept 3: State Ownership (Defining State in the Parent for a Single Source of Truth)

### Definitions

**Core Definition:** State ownership is the design principle that each piece of state in a React application should have exactly one owning component—typically the closest common ancestor of all components that need to read or modify that state—which serves as the single source of truth.

**Technical Definition:** In React, every piece of state is owned by a specific component—the component that calls `useState` or `useReducer` for that value. This state can only be directly modified by the owning component; child components receive it as props and can request changes only through callback functions passed down by the owner. The "single source of truth" principle ensures that for each unique piece of state, there is one canonical location where the current value resides. When multiple components need to share state, the state is "lifted" to their closest common ancestor, which becomes the owner. The ownership decision follows a hierarchy: local ownership (state lives in the component that uses and updates it), shared ownership (multiple children rely on the same state, but one parent owns it and passes it down), lifted ownership (ownership moves to the closest common ancestor for sibling or nested components), and centralized ownership (app-wide state managed via Context or external stores). Misplacing ownership—storing state too high, too low, or in multiple places—leads to unnecessary re-renders, duplicated state, and synchronization bugs.

**Beginner-Friendly Explanation:** Imagine a classroom where students (child components) need to know the current time. If each student had their own clock, the clocks might disagree, and someone would have to run around updating them all. Instead, the teacher (parent component) has the only official clock (the state), and all students look at it (via props). If a student wants to change the time, they can't just change their own clock—they have to ask the teacher, who updates the official clock and tells everyone the new time. That's state ownership: one component owns each piece of state, and everyone else reads it or requests changes.

### Purposes

- To establish a single, authoritative source for each piece of state, preventing synchronization bugs.
- To make data flow predictable and explicit: state flows down via props, and changes flow up via callbacks.
- To avoid duplicated state that can drift out of sync.
- To give the parent control over when and how state updates occur.
- To enable coordination between multiple child components that need to agree on a shared value.
- To provide controlled components with a clear, consistent value and update mechanism.
- To make the application's state architecture easier to reason about, test, and debug.

### Syntax Rules and Structure

**General Syntax (Parent Owns State, Passes Value and Setter):**
```jsx
function ParentComponent() {
  // Parent owns the state — it is the single source of truth
  const [selectedItem, setSelectedItem] = useState(null);

  return (
    <div>
      {/* Pass the value down as a prop */}
      <ItemList
        selectedItem={selectedItem}
        onItemSelect={setSelectedItem} // Pass the setter as a callback
      />
      {/* Another child reads the same state */}
      <ItemDetail item={selectedItem} />
    </div>
  );
}
```

**Component Breakdown:**
- `const [selectedItem, setSelectedItem] = useState(null)`: The parent owns the state.
- `selectedItem={selectedItem}`: The value is passed down to children as a prop.
- `onItemSelect={setSelectedItem}`: The setter function is passed down as a callback for children to request changes.

**General Syntax (Child Reads Value and Requests Updates):**
```jsx
function ItemList({ selectedItem, onItemSelect }) {
  return (
    <ul>
      {items.map(item => (
        <li
          key={item.id}
          className={selectedItem?.id === item.id ? 'selected' : ''}
          onClick={() => onItemSelect(item)} // Request a state change
        >
          {item.name}
        </li>
      ))}
    </ul>
  );
}
```

**Component Breakdown:**
- `selectedItem`: The child reads the current state value.
- `onItemSelect(item)`: The child requests a change by calling the parent's callback.
- The child does not modify `selectedItem` directly—it only calls the callback.

**Syntax Rules:**
- For each piece of state, identify the closest common ancestor of all components that need to read or modify it. That ancestor should own the state.
- Pass the state value down as a prop and the state setter (or a wrapper function) down as a callback.
- The child should never mutate the state value it receives; it should only read it and request changes via callbacks.
- Use controlled components for form inputs: the value comes from props, and changes are reported via callbacks.
- When state is shared between siblings, lift it to their common parent. The parent owns it and passes it to both children.
- Use the "state ownership decision tree" to decide where state belongs: start with local state; climb to lifted state, reducer, context, or external store only when the lower rung cannot hold the state.
- Avoid storing the same data in multiple components' state; derive it instead.

**Constraints and Limitations:**
- Lifting state up can lead to prop drilling if the state must travel through many intermediate components that do not use it.
- The parent becomes responsible for managing state that may conceptually belong to a child (e.g., a form's input values), which can feel unnatural but is necessary for controlled components.
- Over-lifting state (storing everything in a top-level component) causes unnecessary re-renders and tight coupling.
- Under-lifting state (duplicating state across siblings) causes synchronization bugs.
- The single source of truth principle does not mean all state lives in one place—it means each piece of state has one owner.

### Annotated Code Examples

**Example 1: Lifting State Up for Coordinated Panels**

This is the canonical example from React's official documentation. Two `Panel` components each have their own `isActive` state, but the parent wants only one panel expanded at a time.

```jsx
import { useState } from 'react';

// Child component: now a controlled component — isActive comes from the parent
function Panel({ title, children, isActive, onShow }) {
  return (
    <section className="panel">
      <h3>{title}</h3>
      {isActive ? (
        <p>{children}</p>
      ) : (
        <button onClick={onShow}>Show</button>
      )}
    </section>
  );
}

// Parent component: owns the activeIndex state — the single source of truth
export default function Accordion() {
  const [activeIndex, setActiveIndex] = useState(0);

  return (
    <>
      <h2>Almaty, Kazakhstan</h2>
      <Panel
        title="About"
        isActive={activeIndex === 0}
        onShow={() => setActiveIndex(0)}
      >
        With a population of about 2 million, Almaty is Kazakhstan's largest city.
      </Panel>
      <Panel
        title="Etymology"
        isActive={activeIndex === 1}
        onShow={() => setActiveIndex(1)}
      >
        The name comes from the Kazakh word for "apple".
      </Panel>
    </>
  );
}
```

**Expected Output:** Two panels are displayed: "About" and "Etymology". Initially, the "About" panel is expanded (showing its content), and the "Etymology" panel shows a "Show" button. Clicking "Show" on the "Etymology" panel collapses "About" and expands "Etymology". Only one panel can be open at a time.

**Why This Output Occurs:** The `activeIndex` state lives in the parent (`Accordion`), making it the single source of truth. Each `Panel` receives `isActive` (a boolean derived from `activeIndex`) and an `onShow` callback that updates `activeIndex`. When the user clicks "Show" on the second panel, `onShow` calls `setActiveIndex(1)`, which re-renders the parent. The parent then passes `isActive={true}` to the second panel and `isActive={false}` to the first, collapsing the first and expanding the second. The two panels are now coordinated because they share the same source of truth.

**Example 2: Shared State Between Siblings**

```jsx
import { useState } from 'react';

// Parent owns the shared state
function App() {
  const [selectedUser, setSelectedUser] = useState(null);
  const [users, setUsers] = useState([
    { id: 1, name: 'John Doe', email: 'john@example.com' },
    { id: 2, name: 'Jane Smith', email: 'jane@example.com' },
  ]);

  const handleUserSelect = (user) => {
    setSelectedUser(user);
  };

  const handleUserUpdate = (updatedUser) => {
    setUsers(prevUsers =>
      prevUsers.map(user =>
        user.id === updatedUser.id ? updatedUser : user
      )
    );
    setSelectedUser(updatedUser);
  };

  return (
    <div>
      <h1>User Manager</h1>
      <UserList
        users={users}
        selectedUser={selectedUser}
        onUserSelect={handleUserSelect}
      />
      <UserDetails
        user={selectedUser}
        onUserUpdate={handleUserUpdate}
      />
    </div>
  );
}

// Child 1: displays the list and reports selections
function UserList({ users, selectedUser, onUserSelect }) {
  return (
    <div>
      {users.map(user => (
        <div
          key={user.id}
          onClick={() => onUserSelect(user)}
          style={{
            background: selectedUser?.id === user.id ? '#f0f0f0' : 'white',
            cursor: 'pointer',
          }}
        >
          {user.name}
        </div>
      ))}
    </div>
  );
}

// Child 2: displays details and reports updates
function UserDetails({ user, onUserUpdate }) {
  if (!user) return <div>Select a user</div>;

  return (
    <div>
      <h3>{user.name}</h3>
      <p>{user.email}</p>
      <button
        onClick={() =>
          onUserUpdate({ ...user, name: user.name + ' (Updated)' })
        }
      >
        Update User
      </button>
    </div>
  );
}
```

**Expected Output:** The left side shows a list of two users (John Doe, Jane Smith). Initially, no user is selected, and the right side displays "Select a user". Clicking "John Doe" highlights him in the list and displays his details (name and email) on the right. Clicking "Update User" changes his name to "John Doe (Updated)" in both the list and the details panel.

**Why This Output Occurs:** The `selectedUser` state lives in the parent (`App`), making it the single source of truth for which user is selected. Both `UserList` and `UserDetails` read this state via props. When `UserList` calls `onUserSelect(user)`, the parent updates `selectedUser`, and both children re-render with the new selection. When `UserDetails` calls `onUserUpdate(updatedUser)`, the parent updates both `users` and `selectedUser`, and both children reflect the change. The parent owns the state; the children read it and request changes.

### Real-World Cases

- **Tabbed interfaces:** The active tab index is owned by the parent; each tab button reports clicks via `onTabChange`, and the tab panel reads the active index.
- **Shopping carts:** The cart items state is owned by a top-level component; product list items and the cart summary both read it, and add/remove actions are reported via callbacks.
- **Form wizards:** The current step and form data are owned by the parent wizard component; each step child reads the data and reports changes via callbacks.
- **Filterable lists:** The filter criteria state is owned by the parent; the filter controls report changes, and the list component reads the filter to display filtered results.
- **Dashboard widgets:** Shared date-range state is owned by the dashboard parent; each widget reads the range and reports changes via callbacks.

### References

- React Official Documentation – Sharing State Between Components: https://react.dev/learn/sharing-state-between-components
- React Official Documentation – Choosing the State Structure: https://react.dev/learn/choosing-the-state-structure
- React Official Documentation – Managing State: https://react.dev/learn/managing-state
- Educative – State Ownership Philosophy: https://www.educative.io/courses/learn-react/state-ownership-philosophy
- Code Crunch Worldwide – The State-Ownership Decision Tree: https://codecrunchglobal.vercel.app/course?course=c29&path=curriculum%2Fweek-04-state-management-local-context-and-external-stores%2Flecture-notes%2F01-the-state-ownership-decision-tree.md
- CoreUI – How to Lift State Up in React: https://coreui.io/answers/how-to-lift-state-up-in-react/
- Stack Overflow – Best Way to Allow a Component to Offer a Callback to Any Parent: https://stackoverflow.com/questions/79726632/best-way-to-allow-a-component-to-offer-a-callback-to-any-parent-calling-it
- Educative – Understanding and Managing State in React Components: https://www.educative.io/courses/building-teslas-battery-range-calculator-with-react-and-redux/1-9-state-of-application

---

## References

- React Official Documentation – Sharing State Between Components: https://react.dev/learn/sharing-state-between-components
- React Official Documentation – Responding to Events: https://react.dev/learn/responding-to-events
- React Official Documentation – Choosing the State Structure: https://react.dev/learn/choosing-the-state-structure
- React Official Documentation – Managing State: https://react.dev/learn/managing-state
- React Legacy Documentation – Lifting State Up: https://legacy.reactjs.org/docs/lifting-state-up.html
- React Legacy Documentation – Passing Functions to Components: https://legacy.reactjs.org/docs/faq-functions.html
- Educative – State Ownership Philosophy: https://www.educative.io/courses/learn-react/state-ownership-philosophy
- Educative – Understanding and Managing State in React Components: https://www.educative.io/courses/building-teslas-battery-range-calculator-with-react-and-redux/1-9-state-of-application
- Code Crunch Worldwide – The State-Ownership Decision Tree: https://codecrunchglobal.vercel.app/course?course=c29&path=curriculum%2Fweek-04-state-management-local-context-and-external-stores%2Flecture-notes%2F01-the-state-ownership-decision-tree.md
- CoreUI – How to Lift State Up in React: https://coreui.io/answers/how-to-lift-state-up-in-react/
- DEV Community – Callback Props in React: https://dev.to/ali007depug/callback-props-in-react-37m0
- JavaScript Plain English – Communication Patterns in React: https://javascript.plainenglish.io/communication-patterns-in-react-30df2de702eb
- Stack Overflow – Best Way to Allow a Component to Offer a Callback to Any Parent: https://stackoverflow.com/questions/79726632/best-way-to-allow-a-component-to-offer-a-callback-to-any-parent-calling-it
- Stack Overflow – Lifting State Up (Event Handler Binding): https://stackoverflow.com/revisions/87e1cf40-e2f3-4a3f-bd5d-d987c2a77da3/view-source