# React Parent-to-Child Communication: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** React parent-to-child communication is the mechanism by which a parent component passes data, configuration, and behaviour to its child components through the `props` object, enabling components to remain reusable, composable, and decoupled.

**Technical Definition:** In React, `props` (short for "properties") is a plain JavaScript object passed as the sole argument to a function component or accessed via `this.props` in a class component. Props flow in a single direction—from parent to child—and are immutable within the receiving component. React components use props to communicate with each other, and every parent component can pass some information to its child components by giving them props. Props may resemble HTML attributes, but any JavaScript value can be passed, including objects, arrays, functions, and JSX elements. The `children` prop is a special prop that receives the JSX content nested between a component's opening and closing tags. For sharing stateful logic, render props (a prop whose value is a function returning a React element) provide a technique for cross-cutting concern reuse. Callback functions passed as props allow child components to communicate back to the parent, enabling bidirectional data flow within React's unidirectional architecture.

**Beginner-Friendly Explanation:** Imagine a parent component as a manager and child components as team members. The manager hands each team member a folder of instructions (props)—some containing data like "here's the employee name," some containing tools like "here's a function you can call if you need to report something." The team member reads the folder but cannot change what's written in it. If the manager needs to change the instructions, they create a new folder and hand it down. This one-way flow of information keeps everything predictable and easy to debug.

### Key Characteristics

- **Unidirectional Data Flow:** Props flow strictly from parent to child; children cannot modify the props they receive.
- **Immutable Within Child:** A component must never modify its own props directly.
- **Any JavaScript Value:** Props can carry primitives, objects, arrays, functions, JSX elements, and even other components.
- **Re-render on Change:** When a parent re-renders with new prop values, the child re-renders with the updated props.
- **Composition via `children`:** The `children` prop enables component composition by nesting JSX between opening and closing tags.
- **Callback Props Enable Uplift:** Functions passed as props let children trigger parent-defined actions, effectively communicating upward.
- **Render Props for Logic Sharing:** A function-as-prop pattern that shares stateful logic while letting the consumer control rendering.

### Prerequisites

- Solid understanding of JavaScript functions, objects, destructuring, and closures.
- Familiarity with JSX syntax and React component creation (function and class components).
- Working knowledge of `useState` and basic component state management.
- Understanding of the component tree hierarchy and render cycle.

### Related Programming Areas

- **Component Composition:** Building complex UIs from smaller, reusable pieces.
- **State Lifting:** Moving state up to a common ancestor so multiple children can share it.
- **Dependency Injection:** Passing behaviour (functions) into components to invert control.
- **Design Patterns:** Higher-order components (HOCs), render props, and compound components.
- **Prop Drilling:** The problem of passing props through many intermediate components, and its solutions (Context API, composition).

### Core Concepts / Features

1. Props (Read-Only Configuration and Data)
2. Callback Functions (Child-to-Parent Communication)
3. Configuration Objects (Bundled Related Properties)
4. Render Props (Function-as-Prop for Logic Sharing)

---

## Core Concept 1: Props (Read-Only Configuration and Data)

### Definitions

**Core Definition:** Props are the read-only inputs passed from a parent component to a child component, carrying data, configuration, and behaviour that the child uses to render its output.

**Technical Definition:** `props` is a JavaScript object that React passes as the first argument to a function component or assigns to `this.props` in a class component. Every React element has a `props` object containing the attributes defined on the JSX tag. Props are immutable within the receiving component—attempting to reassign a prop value will either fail silently or throw an error in strict mode. React components must act like pure functions with respect to their props: given the same props, a component should always render the same output. Props can be destructured directly in the function signature for convenience, and default values can be assigned using JavaScript default parameters (recommended over the deprecated `defaultProps` for function components). The special `children` prop receives any JSX nested between a component's tags, enabling composition.

**Beginner-Friendly Explanation:** Think of a component as a vending machine. You insert coins (props) and select an item (render output). The machine doesn't keep your coins—it uses them to decide what to dispense. You can't reach inside and change the coin's value after inserting it. Different coins produce different results. That's exactly how props work in React.

### Purposes

- To pass data (strings, numbers, booleans, objects, arrays) from parent to child.
- To configure a child component's appearance or behaviour without modifying its code.
- To enable component reuse by allowing the same component to render differently based on props.
- To compose complex UIs by nesting components and passing content via the `children` prop.
- To establish a clear, one-way data flow that makes applications easier to reason about and debug.
- To provide default values that ensure components render gracefully even when certain props are omitted.

### Syntax Rules and Structure

**General Syntax (Passing Props):**
```jsx
<ChildComponent
  propName={value}
  anotherProp="string value"
  booleanProp={true}
  objectProp={{ key: 'value' }}
  arrayProp={[1, 2, 3]}
  functionProp={handleFunction}
>
  {childrenContent}
</ChildComponent>
```

**Component Breakdown:**
- `propName={value}`: Passes any JavaScript expression as a prop.
- `anotherProp="string value"`: String literals can be passed without curly braces.
- `objectProp={{ key: 'value' }}`: Double curly braces: outer for JSX expression, inner for object literal.
- `childrenContent`: JSX nested between tags is received as the `children` prop.

**General Syntax (Reading Props — Function Component):**
```jsx
// Full props object
function ChildComponent(props) {
  return <p>{props.propName}</p>;
}

// Destructured props (preferred)
function ChildComponent({ propName, anotherProp = 'default' }) {
  return <p>{propName}</p>;
}
```

**Component Breakdown:**
- `function ChildComponent(props)`: Receives the full props object as the first argument.
- `({ propName, anotherProp })`: Destructuring in the parameter list for cleaner code.
- `anotherProp = 'default'`: JavaScript default parameter value (future-proof approach).

**General Syntax (Spreading Props):**
```jsx
<ChildComponent {...allProps} />
```

**Component Breakdown:**
- `{...allProps}`: Spreads all enumerable properties of `allProps` as individual props.
- Useful for forwarding props from a parent to a child without listing each one.

**Syntax Rules:**
- Props are passed as attributes on JSX tags; they are read-only within the child.
- Never mutate props directly (`props.x = 5` is forbidden); create a local variable instead.
- Use destructuring in the function signature for readability.
- Use JavaScript default parameters (`function Child({ x = 10 })`) instead of `Child.defaultProps` for function components (the latter is deprecated in React 18.3+ and removed in React 19).
- The `children` prop is automatically populated with any JSX nested between the opening and closing tags of a component.
- Props with boolean `true` values can be written shorthand: `<Button disabled />` is equivalent to `<Button disabled={true} />`.

**Constraints and Limitations:**
- Props are immutable within the receiving component; they can only be changed by the parent re-rendering with new values.
- Passing new object or array literals as props creates a new reference on every render, which can cause unnecessary re-renders in memoised children.
- Prop drilling—passing props through many intermediate components that don't use them—is a common maintainability challenge.
- The `children` prop can only be used once per component instance; multiple JSX elements passed as children are wrapped in an array.
- `defaultProps` on function components is deprecated; use default parameters instead.

### Annotated Code Examples

**Example 1: Passing and Reading Primitive, Object, and Array Props**

```jsx
// Child component: Avatar receives person (object) and size (number) props
function Avatar({ person, size = 100 }) {
  // person and size are available here as local variables
  const imageUrl = `https://i.imgur.com/${person.imageId}.jpg`;

  return (
    <img
      className="avatar"
      src={imageUrl}
      alt={person.name}
      width={size}
      height={size}
    />
  );
}

// Parent component: Profile passes props to Avatar
export default function Profile() {
  return (
    <div>
      {/* Passing an object literal as the person prop */}
      <Avatar
        size={100}
        person={{ name: 'Katsuko Saruhashi', imageId: 'YfeOqp2' }}
      />
      {/* Passing a different object; size uses the default value of 100 */}
      <Avatar
        person={{ name: 'Aklilu Lemma', imageId: 'OKS67lh' }}
      />
      {/* Passing a smaller size */}
      <Avatar
        size={50}
        person={{ name: 'Lin Lanying', imageId: '1bX5QH6' }}
      />
    </div>
  );
}
```

**Expected Output:** Three avatar images are rendered. The first is 100×100 pixels and shows Katsuko Saruhashi. The second is also 100×100 (using the default `size` value) and shows Aklilu Lemma. The third is 50×50 pixels and shows Lin Lanying.

**Why This Output Occurs:** The `Profile` component passes different `person` objects and `size` values to each `Avatar` instance. The `Avatar` component destructures these props and uses them to construct the image URL and dimensions. The second avatar omits the `size` prop, so the default parameter value of `100` is used. Props let you think about parent and child components independently: you can change the `person` or `size` props inside `Profile` without having to think about how `Avatar` uses them.

**Example 2: The `children` Prop for Composition**

```jsx
// Card component: a generic container that accepts children
function Card({ children, title }) {
  return (
    <div className="card">
      <h2>{title}</h2>
      <div className="card-content">
        {children} {/* Renders whatever JSX was nested inside */}
      </div>
    </div>
  );
}

// Parent component: composes Card with different content
export default function App() {
  return (
    <div>
      {/* Card with text children */}
      <Card title="Welcome">
        <p>This is the card content.</p>
      </Card>

      {/* Card with an image and a button as children */}
      <Card title="Media">
        <img src="/photo.jpg" alt="Photo" />
        <button>View</button>
      </Card>
    </div>
  );
}
```

**Expected Output:** Two card containers are rendered. The first has the title "Welcome" and contains a paragraph. The second has the title "Media" and contains an image and a button. Both cards share the same wrapper structure (a div with class `card`, an `h2` title, and a content div), but their inner content differs entirely.

**Why This Output Occurs:** The `Card` component doesn't know or care what content it will contain—it simply renders whatever JSX is passed as `children`. The parent component decides what goes inside each card by nesting JSX between the `<Card>` tags. This is the composition pattern: the parent controls the structure and content, while the child provides the reusable wrapper and styling.

### Real-World Cases

- **Design systems:** A `Button` component receives `variant`, `size`, `disabled`, and `children` props to render different button styles while sharing the same underlying logic.
- **Layout components:** A `Grid` component receives `columns`, `gap`, and `children` to lay out its content without knowing what the content is.
- **Form libraries:** An `Input` component receives `label`, `value`, `onChange`, `error`, and `placeholder` props to render a fully configured form field.
- **Data display:** A `Table` component receives `data`, `columns`, and `onRowClick` props to render a data grid with configurable behaviour.

### References

- React Official Documentation – Passing Props to a Component: https://react.dev/learn/passing-props-to-a-component
- React Official Documentation – Props and State (Mintlify): https://mintlify.wiki/facebook/react/concepts/props-and-state
- React Official Documentation – Sharing State Between Components: https://react.dev/learn/sharing-state-between-components

---

## Core Concept 2: Callback Functions (Child-to-Parent Communication)

### Definitions

**Core Definition:** A callback function prop is a function defined in a parent component and passed down to a child component, which the child invokes to report events, send data, or trigger parent-defined actions.

**Technical Definition:** Callback props are the mechanism by which data flows upward in React's otherwise unidirectional architecture. A parent component defines a function (e.g., `handleSubmit`, `onItemSelect`) and passes it as a prop to a child. The child component calls this function—typically inside an event handler—to communicate back to the parent. This is how children communicate with parents via callbacks. The callback receives arguments from the child (e.g., form data, an item ID), allowing the parent to update its own state or perform side effects. This pattern is essential for lifting state up: moving state to the closest common ancestor and passing callbacks down so children can trigger state updates in the parent. Common conventions include naming props with an `on` prefix (e.g., `onSubmit`, `onChange`, `onSelect`) and `handle` prefix for the function definition (e.g., `handleSubmit`).

**Beginner-Friendly Explanation:** Imagine a child component is a TV remote and the parent is the TV. The parent gives the child a set of buttons (callback functions) and says: "Press this one to tell me to change the channel, press that one to tell me to adjust the volume." The child can't change the channel itself—it can only press the button, which triggers the parent to do the work. This keeps the TV (parent) in control of its own state, while the remote (child) provides the interface.

### Purposes

- To allow a child component to report events (clicks, form submissions, selections) back to the parent.
- To enable the parent to update its state in response to child-triggered actions.
- To lift state up to the closest common ancestor so multiple children can share and update it.
- To keep the parent in control of business logic and data mutations while the child focuses on presentation.
- To create reusable child components that don't need to know *how* the parent will respond to an action—only *that* it should be notified.
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
- When passing a callback that takes no arguments, you can pass the function reference directly: `onClick={handleClick}` (not `onClick={handleClick()}`).
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
        Name:
        <input
          type="text"
          value={name}
          onChange={(e) => setName(e.target.value)}
        />
      </label>
      <label>
        Email:
        <input
          type="email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
        />
      </label>
      <br />
      <button onClick={handleFormSubmit}>Submit</button>
    </div>
  );
}
```

**Expected Output:** The form displays name and email inputs and a "Submit" button. When both fields are filled and the button is clicked, an alert shows "Form submitted with: {"name":"...","email":"..."}". If either field is empty, an alert says "Please fill in all fields!".

**Why This Output Occurs:** The `FormInput` child component manages its own local input state (`name`, `email`) but does not know what happens after submission. When the user clicks "Submit", the child calls `onSubmit({ name, email })`—the callback prop passed from the parent. The parent's `handleFormSubmit` function receives this data and performs the actual submission logic (logging and alerting). The `ParentComponent` holds the form submission logic in the `handleSubmit` function, and the `FormInput` component captures user input and triggers the parent's `onSubmit` function when the user clicks the "Submit" button.

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

**Why This Output Occurs:** The `temperature` state lives in the parent (`TemperatureApp`), making it the "single source of truth." The parent passes both the current value (`temperature`) and a callback (`onTemperatureChange`, which is `setTemperature`) to the child. When the user types in the input, the child calls `onTemperatureChange(newValue)`, which updates the parent's state. The parent re-renders with the new temperature, passing the updated value back down to the child. This is the "lifting state up" pattern: the state is moved to the closest common ancestor so multiple components can share it.

### Real-World Cases

- **Modal components:** A parent passes an `onClose` callback to a modal child; the child calls it when the user clicks the close button or presses Escape.
- **Dropdown/Select components:** A parent passes `onSelect` to a dropdown child; the child calls it with the selected option when the user makes a choice.
- **Todo lists:** A parent passes `onToggle` and `onDelete` callbacks to each todo item child; the child calls the appropriate callback when the user interacts with the item.
- **Pagination:** A parent passes `onPageChange` to a pagination child; the child calls it with the new page number when the user clicks a page button.
- **Search bars:** A parent passes `onSearch` to a search input child; the child calls it with the query when the user submits or types.

### References

- React Official Documentation – Sharing State Between Components: https://react.dev/learn/sharing-state-between-components
- React Official Documentation – Responding to Events: https://react.dev/learn/responding-to-events
- React Legacy Documentation – Passing Functions to Components: https://id.legacy.reactjs.org/docs/faq-functions.html
- Tsecurity.de – React Component Communication: Parent-Child and Child-Parent Interactions: https://tsecurity.de/de/2458759/IT+Programmierung/React+Component+Communication:+Parent-Child+and+Child-Parent+Interactions/

---

## Core Concept 3: Configuration Objects (Bundled Related Properties)

### Definitions

**Core Definition:** A configuration object prop is a single object prop that bundles multiple related properties together, reducing the number of individual props a component receives and keeping its interface clean and manageable.

**Technical Definition:** Rather than passing many loose, related props individually (e.g., `size`, `color`, `borderRadius`, `shadow`), a parent passes a single object (e.g., `theme={{ size: 'large', color: 'blue', borderRadius: 8, shadow: true }}`). The child destructures the specific properties it needs from this object. This pattern is particularly useful when a component has many configuration options, when the options naturally group into categories, or when defaults need to be applied to the entire group. It also makes the component's interface more extensible—new configuration options can be added to the object without changing the component's prop signature. The pattern is closely related to the "options object" pattern in JavaScript and is used extensively in design systems and third-party libraries.

**Beginner-Friendly Explanation:** Instead of handing someone a dozen separate sticky notes, you put them all in one folder and hand over the folder. The folder is the configuration object, and each sticky note inside is a related property. This keeps things organised and makes it easy to add more notes later without changing how you hand things over.

### Purposes

- To reduce the number of individual props a component receives, keeping its interface simple.
- To group logically related configuration options into a single, cohesive unit.
- To make component interfaces more extensible—new options can be added to the object without changing the prop signature.
- To provide sensible defaults for an entire group of related options at once.
- To improve readability by making the relationship between configuration values explicit.
- To facilitate passing configuration through multiple component layers without listing every individual prop.

### Syntax Rules and Structure

**General Syntax (Passing a Configuration Object):**
```jsx
<ChildComponent
  config={{
    size: 'large',
    color: 'blue',
    borderRadius: 8,
    shadow: true,
  }}
/>
```

**Component Breakdown:**
- `config={{ ... }}`: A single prop named `config` receives an object literal.
- `size: 'large'`, `color: 'blue'`, etc.: Individual configuration properties within the object.

**General Syntax (Reading a Configuration Object):**
```jsx
function ChildComponent({ config }) {
  // Destructure with defaults
  const {
    size = 'medium',
    color = 'gray',
    borderRadius = 4,
    shadow = false,
  } = config;

  return (
    <div style={{ fontSize: size === 'large' ? 24 : 16, color, borderRadius, boxShadow: shadow ? '0 2px 4px rgba(0,0,0,0.2)' : 'none' }}>
      Configured Content
    </div>
  );
}
```

**Component Breakdown:**
- `({ config })`: Destructures the `config` prop from the props object.
- `const { size = 'medium', ... } = config`: Destructures individual properties from the config object with default values.
- Each property is used independently within the component.

**Syntax Rules:**
- Use a descriptive name for the configuration prop (e.g., `config`, `theme`, `settings`, `options`, `style`).
- Define default values for individual properties within the object using destructuring defaults, or provide a default object itself: `function Child({ config = {} })`.
- Avoid deeply nested configuration objects (more than two levels); they become difficult to destructure and maintain.
- When the configuration object is large, consider splitting it into multiple named objects (e.g., `styleConfig`, `behaviorConfig`).
- Use TypeScript interfaces or PropTypes to document the shape of the configuration object.
- If the configuration object is passed through many layers, consider using the Context API instead of prop drilling.

**Constraints and Limitations:**
- A new object literal is created on every render, causing a new reference each time; this can trigger unnecessary re-renders in memoised children.
- Destructuring deeply nested properties can become verbose.
- The component's API is less discoverable when configuration is hidden inside an object; developers must read the child's code to know what properties are available.
- Overusing configuration objects can lead to a "god object" anti-pattern where a single prop contains unrelated options.

### Annotated Code Examples

**Example 1: Theme Configuration Object for a Button**

```jsx
// Child component: Button receives a theme configuration object
function Button({ children, theme = {} }) {
  // Destructure theme properties with defaults
  const {
    backgroundColor = '#007bff',
    textColor = '#ffffff',
    padding = '10px 20px',
    borderRadius = '4px',
    fontSize = '16px',
  } = theme;

  return (
    <button
      style={{
        backgroundColor,
        color: textColor,
        padding,
        borderRadius,
        fontSize,
        border: 'none',
        cursor: 'pointer',
      }}
    >
      {children}
    </button>
  );
}

// Parent component: passes different theme configurations
export default function App() {
  return (
    <div>
      {/* Default theme (no theme prop) */}
      <Button>Default Button</Button>

      {/* Custom theme configuration object */}
      <Button
        theme={{
          backgroundColor: '#28a745',
          textColor: '#ffffff',
          borderRadius: '20px',
          fontSize: '18px',
        }}
      >
        Success Button
      </Button>

      {/* Another custom theme */}
      <Button
        theme={{
          backgroundColor: '#dc3545',
          padding: '12px 30px',
        }}
      >
        Danger Button
      </Button>
    </div>
  );
}
```

**Expected Output:** Three buttons are rendered. The first is blue with default styling. The second is green with rounded corners and larger text. The third is red with extra padding. All share the same underlying `Button` component but appear differently based on their `theme` configuration objects.

**Why This Output Occurs:** The `Button` component accepts a single `theme` object prop and destructures individual style properties from it, applying defaults for any properties not provided. This keeps the `Button` component's interface clean—adding a new theme option (e.g., `borderWidth`) only requires updating the `Button` component's destructuring, not the parent's prop list.

**Example 2: Chart Configuration Object**

```jsx
// Child component: a simplified chart that accepts a config object
function SimpleChart({ data, config = {} }) {
  const {
    width = 400,
    height = 300,
    barColor = 'steelblue',
    showGrid = true,
    animate = false,
  } = config;

  // Simplified rendering logic
  return (
    <div
      style={{
        width,
        height,
        border: '1px solid #ccc',
        display: 'flex',
        alignItems: 'flex-end',
        gap: '4px',
        padding: '10px',
      }}
    >
      {data.map((value, index) => (
        <div
          key={index}
          style={{
            height: `${(value / Math.max(...data)) * 100}%`,
            width: '20px',
            backgroundColor: barColor,
            transition: animate ? 'height 0.3s ease' : 'none',
          }}
        />
      ))}
    </div>
  );
}

// Parent component
export default function Dashboard() {
  const salesData = [120, 200, 150, 80, 170, 220];

  return (
    <div>
      <h2>Sales Overview</h2>
      <SimpleChart
        data={salesData}
        config={{
          width: 500,
          height: 350,
          barColor: '#17a2b8',
          showGrid: false,
          animate: true,
        }}
      />
    </div>
  );
}
```

**Expected Output:** A bar chart is rendered with a width of 500px and height of 350px. The bars are teal-colored and animate their height on initial render. The grid is not shown (though in this simplified example, the grid isn't actually rendered—it's just a configuration option).

**Why This Output Occurs:** The `SimpleChart` component accepts a `config` object containing all its visual configuration. The parent (`Dashboard`) passes a single object that customises the chart's appearance. If the parent wanted a different chart style, it would simply pass a different `config` object—no changes to the `SimpleChart` component needed.

### Real-World Cases

- **Design system components:** A `Modal` component receives a `config` object with `size`, `closeOnOverlay`, `showCloseButton`, and `animation` properties.
- **Charting libraries:** Chart.js and Recharts accept a `options` object that bundles axes configuration, tooltip settings, and legend display.
- **Form libraries:** Formik and React Hook Form accept a `validationSchema` object that bundles validation rules for all form fields.
- **Data tables:** A `DataTable` component receives a `columns` array and a `sortConfig` object that specifies the default sort field and direction.
- **Map components:** Leaflet and Mapbox React wrappers accept a `mapOptions` object with `center`, `zoom`, `minZoom`, `maxZoom`, and `attribution` properties.

### References

- React Official Documentation – Passing Props to a Component: https://react.dev/learn/passing-props-to-a-component
- React TypeScript Cheatsheet – Types for Props: https://react-typescript-cheatsheet.netlify.app/docs/basic/getting-started/basic_type_example
- Patterns.dev – Props Patterns: https://www.patterns.dev/react/render-props-pattern

---

## Core Concept 4: Render Props (Function-as-Prop for Logic Sharing)

### Definitions

**Core Definition:** A render prop is a technique for sharing code between React components using a prop whose value is a function that returns a React element.

**Technical Definition:** The term "render prop" refers to a technique for sharing code between React components using a prop whose value is a function. A component with a render prop takes a function that returns a React element and calls it instead of implementing its own render logic. The component encapsulates stateful behaviour (e.g., mouse tracking, data fetching, form validation) and passes the relevant state or API down to the render prop function. The consumer of the component decides what UI to render based on the data provided. The render prop doesn't have to be called `render`—any prop whose value is a function returning JSX qualifies (e.g., `children`, `renderItem`, `renderEmpty`). In modern React, custom Hooks have largely replaced render props for sharing *stateful logic*, but render props remain valuable when the consumer needs to control *what is rendered* based on the shared logic.

**Beginner-Friendly Explanation:** Imagine a component that tracks the mouse position. Instead of hard-coding what to display (like "the mouse is at X, Y"), it lets you decide. You pass it a function that says: "Here's the mouse position—you figure out what to draw." The component does the hard work of tracking the mouse, and you do the fun part of deciding what the screen looks like. That function you pass is the "render prop."

### Purposes

- To share stateful logic between components without duplicating the logic.
- To allow the consumer of a component to control what is rendered while the component controls the behaviour.
- To create reusable behaviour components (mouse trackers, data fetchers, form validators) that are decoupled from their presentation.
- To avoid "wrapper hell" caused by higher-order components (HOCs) while still sharing cross-cutting concerns.
- To provide a flexible alternative to HOCs when the consumer needs dynamic control over rendering.
- To enable composition of behaviour without inheritance.

### Syntax Rules and Structure

**General Syntax (Component with Render Prop):**
```jsx
function DataProvider({ render }) {
  // Component manages state and behaviour
  const [data, setData] = React.useState(null);

  React.useEffect(() => {
    fetchData().then(setData);
  }, []);

  // Call the render prop function with the data
  return <>{render(data)}</>;
}
```

**Component Breakdown:**
- `{ render }`: Destructures the render prop function from the component's props.
- `render(data)`: Calls the function, passing the component's internal state as arguments.
- The return value of `render(data)` is JSX that the component renders.

**General Syntax (Using the Render Prop):**
```jsx
<DataProvider
  render={(data) => (
    // Consumer decides what to render
    data ? <p>Data: {data}</p> : <p>Loading...</p>
  )}
/>
```

**Component Breakdown:**
- `render={(data) => (...)}`: An arrow function that receives the component's state.
- The function returns JSX based on the received data.

**General Syntax (Children as a Function — "Function as a Child"):**
```jsx
<DataProvider>
  {(data) => (
    data ? <p>Data: {data}</p> : <p>Loading...</p>
  )}
</DataProvider>
```

**Component Breakdown:**
- `<DataProvider>...</DataProvider>`: The function is passed as `children` instead of a named render prop.
- Inside `DataProvider`, `props.children(data)` is called instead of `props.render(data)`.
- This is the "function as a child" variant of the render prop pattern.

**Syntax Rules:**
- The render prop function must return a React element (or `null`).
- Name the prop descriptively (`render`, `renderItem`, `renderEmpty`, `children`) based on its purpose.
- The component calls the render prop function with whatever arguments it wants to expose (state, handlers, loading flags).
- The render prop function is re-created on every render of the consumer; this is generally fine, but can cause performance issues in memoised children.
- When using `children` as a function, the component must call `props.children(args)` instead of rendering `{props.children}` directly.
- Render props are a technique, not a React API—any prop whose value is a function returning JSX qualifies.
- In modern React, prefer custom Hooks for sharing stateful logic; use render props when the consumer needs to control rendering based on shared behaviour.

**Constraints and Limitations:**
- Render props can cause "callback hell" when nested multiple levels deep, making the JSX difficult to read.
- The render prop function creates a new function on every render, which can break memoisation in child components.
- Custom Hooks have largely replaced render props for sharing stateful logic; render props are now primarily used for "inversion of control" rendering.
- The pattern is less common in modern React codebases, so new developers may be unfamiliar with it.
- React Router and Downshift are libraries that still use render props (or their modern equivalents).

### Annotated Code Examples

**Example 1: Mouse Tracker with Render Prop**

```jsx
// Component that encapsulates mouse tracking behaviour
class MouseTracker extends React.Component {
  state = { x: 0, y: 0 };

  handleMouseMove = (event) => {
    this.setState({ x: event.clientX, y: event.clientY });
  };

  render() {
    return (
      <div style={{ height: '100vh' }} onMouseMove={this.handleMouseMove}>
        {/* Call the render prop with the current mouse position */}
        {this.props.render(this.state)}
      </div>
    );
  }
}

// Consumer 1: Display coordinates as text
function CoordinateDisplay() {
  return (
    <MouseTracker
      render={({ x, y }) => (
        <p>The mouse position is ({x}, {y})</p>
      )}
    />
  );
}

// Consumer 2: Display a cat image following the mouse
function CatImage() {
  return (
    <MouseTracker
      render={({ x, y }) => (
        <img
          src="/cat.jpg"
          alt="Cat"
          style={{ position: 'absolute', left: x, top: y, width: 50 }}
        />
      )}
    />
  );
}

// Consumer 3: Use children as a function
function MouseFollower() {
  return (
    <MouseTracker>
      {({ x, y }) => (
        <div style={{ position: 'absolute', left: x, top: y }}>
          🐭
        </div>
      )}
    </MouseTracker>
  );
}
```

**Expected Output:** Three different UI elements follow the mouse: a text display showing coordinates, a cat image, and an emoji. All three share the same `MouseTracker` behaviour component but render completely different UI.

**Why This Output Occurs:** The `MouseTracker` component encapsulates all the logic for listening to `mousemove` events and storing the cursor position. It doesn't decide what to render—instead, it calls `this.props.render(this.state)` (or `props.children(this.state)` for the function-as-child variant) and lets each consumer decide. This is the core value of render props: separating behaviour from presentation.

**Example 2: Form Validator with Render Prop (Modern Functional Component)**

```jsx
import { useState } from 'react';

// Behaviour component: manages form state and validation
function FormValidator({ initialValues, validate, onSubmit, children }) {
  const [values, setValues] = useState(initialValues);
  const errors = validate(values);
  const isValid = Object.keys(errors).length === 0;

  const setField = (key, value) => {
    setValues((prev) => ({ ...prev, [key]: value }));
  };

  const submit = () => {
    if (isValid) onSubmit(values);
  };

  // Call the children function with the form API
  return <>{children({ values, errors, isValid, setField, submit })}</>;
}

// Consumer: decides how the form looks
function SignInForm() {
  return (
    <FormValidator
      initialValues={{ email: '', password: '' }}
      validate={(v) => ({
        email: v.email.includes('@') ? undefined : 'Not an email',
        password: v.password.length >= 8 ? undefined : 'Too short',
      })}
      onSubmit={(v) => console.log('Signing in:', v)}
    >
      {({ values, errors, isValid, setField, submit }) => (
        <form
          onSubmit={(e) => {
            e.preventDefault();
            submit();
          }}
        >
          <div>
            <label>Email:</label>
            <input
              type="email"
              value={values.email}
              onChange={(e) => setField('email', e.target.value)}
            />
            {errors.email && <span style={{ color: 'red' }}>{errors.email}</span>}
          </div>

          <div>
            <label>Password:</label>
            <input
              type="password"
              value={values.password}
              onChange={(e) => setField('password', e.target.value)}
            />
            {errors.password && <span style={{ color: 'red' }}>{errors.password}</span>}
          </div>

          <button type="submit" disabled={!isValid}>
            Sign In
          </button>
        </form>
      )}
    </FormValidator>
  );
}
```

**Expected Output:** A sign-in form with email and password fields. As the user types, validation errors appear below the fields (e.g., "Not an email" if the email lacks an `@`, "Too short" if the password is fewer than 8 characters). The "Sign In" button is disabled until both fields are valid. Clicking it logs the form values.

**Why This Output Occurs:** The `FormValidator` component owns all the form logic—state management, validation, and submission handling. But it doesn't render any UI itself. Instead, it calls `children({ values, errors, isValid, setField, submit })`, passing the entire form API to the consumer's function. The `SignInForm` consumer decides exactly how the form looks, using whatever markup and styling it wants. This is the render prop pattern in its modern, functional form.

### Real-World Cases

- **React Router:** The `<Route>` component accepts a `render` prop or `children` function to render UI based on route matching.
- **Downshift:** A library for building accessible dropdowns, comboboxes, and autocomplete inputs, uses render props to expose its internal state and handlers.
- **Formik:** The `<Formik>` component accepts a `children` render prop that receives the form state and helpers.
- **Data fetching:** A `<Fetch>` component that accepts a `render` prop receiving `{ loading, error, data }` and lets the consumer decide what to render in each state.
- **Animation libraries:** Components that expose animation progress or state via render props, letting consumers animate different elements.

### References

- React Legacy Documentation – Render Props: https://legacy.reactjs.org/docs/render-props.html
- Patterns.dev – Render Props Pattern: https://www.patterns.dev/react/render-props-pattern
- GitHub – Render Props vs Hooks Comparison: https://github.com/dev48v/render-props-vs-hooks
- Kent C. Dodds – Use a Render Prop!: https://kentcdodds.com/blog/use-a-render-prop

---

## References

- React Official Documentation – Passing Props to a Component: https://react.dev/learn/passing-props-to-a-component
- React Official Documentation – Sharing State Between Components: https://react.dev/learn/sharing-state-between-components
- React Official Documentation – Responding to Events: https://react.dev/learn/responding-to-events
- React Legacy Documentation – Render Props: https://legacy.reactjs.org/docs/render-props.html
- React Legacy Documentation – Passing Functions to Components: https://legacy.reactjs.org/docs/faq-functions.html
- React Documentation – Props and State (Mintlify): https://mintlify.wiki/facebook/react/concepts/props-and-state
- Patterns.dev – Render Props Pattern: https://www.patterns.dev/react/render-props-pattern
- Tsecurity.de – React Component Communication: Parent-Child and Child-Parent Interactions: https://tsecurity.de/de/2458759/IT+Programmierung/React+Component+Communication:+Parent-Child+and+Child-Parent+Interactions/
- Kent C. Dodds – Use a Render Prop!: https://kentcdodds.com/blog/use-a-render-prop
- GitHub – Render Props vs Hooks Comparison: https://github.com/dev48v/render-props-vs-hooks
- React TypeScript Cheatsheet – Types for Props: https://react-typescript-cheatsheet.netlify.app/docs/basic/getting-started/basic_type_example