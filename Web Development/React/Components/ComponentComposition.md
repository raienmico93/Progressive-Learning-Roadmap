# React Component Composition — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Component composition is the practice of building complex user interfaces by combining smaller, independent, and reusable components together.

**Technical Definition:** React's composition model enables code reuse through combining components rather than building inheritance hierarchies. Components can be composed by nesting them (containment), configuring generic components with props to create specialized versions, or using advanced patterns like render props and higher-order components. Props and composition provide all the flexibility needed to customize a component's look and behavior in an explicit and safe way.

**Beginner-Friendly Explanation:** Composition means building big things out of small things. Instead of making one giant component that does everything, you create many small components and combine them like LEGO blocks. Each block does one thing well, and you can rearrange them to build different things.

### Key Characteristics

- **HAS-A relationship:** Components "have" other components, rather than "being" other components (inheritance's IS-A).
- **Explicit and safe:** Props and composition customize behavior without hidden dependencies.
- **Flexible:** Components can be rearranged, nested, and configured freely.
- **Reusable:** Small components can be used in many different contexts.
- **Declarative:** The composition structure is visible in the JSX.

### Prerequisites

- JavaScript functions, objects, and ES6 syntax.
- React function components and props.
- JSX syntax and expressions.
- Basic understanding of state and event handlers.

### Related Programming Areas

- **Props** — the primary mechanism for passing data in composition.
- **Context** — sharing state without prop drilling.
- **Hooks** — reusable stateful logic.
- **Design patterns** — architectural patterns for scalable applications.

---

## 1. Composition

### Definitions

**Core Definition:** Composition is the technique of combining smaller components to build larger, more complex components.

**Technical Definition:** In React, composition is achieved by rendering components inside other components, either directly in JSX or by passing them as props. React recommends composition over inheritance for code reuse because it offers more flexibility and avoids the pitfalls of deep inheritance hierarchies. Props and composition give you all the flexibility you need to customize a component's look and behavior.

**Beginner-Friendly Explanation:** Composition means putting components inside other components to build bigger things. It is like building with LEGO blocks instead of creating one giant molded piece.

### Purposes

- To reuse code across components without inheritance.
- To create flexible, configurable component APIs.
- To separate concerns by keeping components small and focused.
- To build complex UIs from simple, testable pieces.

### Syntax Rules and Structure

**General Syntax (Nesting):**

```jsx
<ParentComponent>
  <ChildComponent />
  <AnotherChild />
</ParentComponent>
```

**General Syntax (Props):**

```jsx
<GenericComponent
  header={<Header />}
  content={<Content />}
/>
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `<ParentComponent>` | The container that composes children. |
| `children` | Nested JSX passed automatically. |
| Named props | JSX elements passed as custom props. |

**Syntax Rules:**

1. Components can render other components directly in JSX.
2. JSX elements can be passed as props (they are just objects).
3. Composition is preferred over inheritance for code reuse.
4. There are no limits on what can be passed as a prop.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Basic Composition with Children

```jsx
// Step 1: Define a generic container component
function FancyBorder(props) {
  return (
    <div className={'FancyBorder FancyBorder-' + props.color}>
      {props.children}
    </div>
  );
}

// Step 2: Compose the container with specific content
function WelcomeDialog() {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title">Welcome</h1>
      <p className="Dialog-message">
        Thank you for visiting our spacecraft!
      </p>
    </FancyBorder>
  );
}
```

**Expected Output:**

```html
<div class="FancyBorder FancyBorder-blue">
  <h1 class="Dialog-title">Welcome</h1>
  <p class="Dialog-message">Thank you for visiting our spacecraft!</p>
</div>
```

**Why this output:** Anything inside the `<FancyBorder>` JSX tags is passed to the component as `props.children`. Since `FancyBorder` renders `{props.children}` inside a `<div>`, the passed elements appear in the final output.

#### Example 2: Composition with Multiple Slots

```jsx
function SplitPane(props) {
  return (
    <div className="SplitPane">
      <div className="SplitPane-left">
        {props.left}
      </div>
      <div className="SplitPane-right">
        {props.right}
      </div>
    </div>
  );
}

function App() {
  return (
    <SplitPane
      left={<Contacts />}
      right={<Chat />}
    />
  );
}
```

**Expected Output:**

```html
<div class="SplitPane">
  <div class="SplitPane-left"><Contacts /></div>
  <div class="SplitPane-right"><Chat /></div>
</div>
```

**Why this output:** React elements like `<Contacts />` and `<Chat />` are just objects, so they can be passed as props like any other data. This approach is similar to "slots" in other libraries, but there are no restrictions on what can be passed as props in React.

### Real-World Cases

- **Layout components:** Headers, sidebars, and footers compose navigation and content.
- **Modal components:** Modal body content is composed as children.
- **Card components:** Card header, body, and footer are composed from smaller pieces.
- **Page layouts:** Each page composes reusable sections.

### References

- React — Composition vs Inheritance — https://legacy.reactjs.org/docs/composition-vs-inheritance.html
- Clay UI — Composition Philosophy — https://www.clayui.com/docs/introduction/composition

---

## 2. Composition vs Inheritance

### Definitions

**Core Definition:** Composition and inheritance are two approaches to code reuse; React strongly recommends composition.

**Technical Definition:** Inheritance builds components that extend other components (IS-A relationship), while composition builds components that contain other components (HAS-A relationship). React has a powerful composition model and recommends using composition instead of inheritance to reuse code between components. In the opinion of React, the right way to reuse code is to compose components of other components, rather than building components that inherit data and/or behaviors from another component.

**Beginner-Friendly Explanation:** Inheritance means "this component IS a special version of that component." Composition means "this component HAS these other components inside it." React prefers composition because it is more flexible and easier to understand.

### Purposes

- To reuse code without creating rigid class hierarchies.
- To avoid the pitfalls of deep inheritance chains.
- To keep components independent and testable.
- To customize behavior explicitly through props.

### Syntax Rules and Structure

**Inheritance (Not Recommended):**

```jsx
class SpecialButton extends Button {
  // Inherits everything from Button
  // Difficult to customize without side effects
}
```

**Composition (Recommended):**

```jsx
function SpecialButton(props) {
  return (
    <Button
      {...props}
      variant="special"
    />
  );
}
```

**Syntax Rules:**

1. React components should never inherit from other React components.
2. Use composition to customize behavior through props.
3. If you need to reuse non-UI logic, extract it into a JavaScript module.

### Annotated Code Examples

```jsx
// ❌ Inheritance: rigid, hard to customize
class FancyButton extends React.Component {
  render() {
    return <button className="fancy">{this.props.children}</button>;
  }
}

class DangerFancyButton extends FancyButton {
  render() {
    // Must call super.render() and modify output
    // Difficult to change structure
  }
}

// ✅ Composition: flexible, explicit
function Button({ variant, children, ...rest }) {
  return (
    <button className={`btn btn-${variant}`} {...rest}>
      {children}
    </button>
  );
}

function DangerButton(props) {
  return <Button variant="danger" {...props} />;
}
```

**Expected Output:** A danger-styled button.

**Why this output:** The composed version creates a new button by wrapping the generic `Button` and configuring it with props. No inheritance is involved.

### Real-World Cases

- **Design systems:** All components are built through composition.
- **React Native:** Same principles apply; composition is preferred.
- **Third-party libraries:** Components are designed for composition (e.g., Radix UI, Headless UI).

### References

- React — Composition vs Inheritance — https://legacy.reactjs.org/docs/composition-vs-inheritance.html
- Educative — React prefers composition over inheritance — https://www.educative.io

---

## 3. Containment

### Definitions

**Core Definition:** Containment is a composition pattern where a component uses the `children` prop to render arbitrary content passed by its parent.

**Technical Definition:** Some components do not know their children ahead of time, especially "box-like" components such as `Sidebar` or `Dialog`. React recommends such components use the special `children` prop to render nested elements directly in their output. This allows other components to pass arbitrary JSX as children, making the container infinitely reusable.

**Beginner-Friendly Explanation:** Containment means a component acts as a box that holds whatever you put inside it. The component does not need to know what it is holding — it just renders it.

### Purposes

- To create generic container components that accept arbitrary content.
- To build reusable layout boxes (dialogs, sidebars, cards).
- To decouple container structure from content.
- To enable flexible composition without pre-defined slots.

### Syntax Rules and Structure

**General Syntax:**

```jsx
function Container(props) {
  return <div className="container">{props.children}</div>;
}

<Container>
  <AnyContent />
</Container>
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `props.children` | The content passed between the container's tags. |
| `<Container>` | The generic container component. |

**Syntax Rules:**

1. The container renders `{props.children}` in its output.
2. Any valid JSX can be passed as children.
3. The container does not need to know what it is rendering.

### Annotated Code Examples

```jsx
function Card({ title, children }) {
  return (
    <div className="card">
      <div className="card-header">
        <h3>{title}</h3>
      </div>
      <div className="card-body">
        {children}  {/* Arbitrary content goes here */}
      </div>
    </div>
  );
}

function App() {
  return (
    <div>
      <Card title="User Info">
        <p>Name: Alice</p>
        <p>Email: alice@example.com</p>
      </Card>

      <Card title="Actions">
        <button>Edit</button>
        <button>Delete</button>
      </Card>
    </div>
  );
}
```

**Expected Output:**

```html
<div class="card">
  <div class="card-header"><h3>User Info</h3></div>
  <div class="card-body">
    <p>Name: Alice</p>
    <p>Email: alice@example.com</p>
  </div>
</div>
<div class="card">
  <div class="card-header"><h3>Actions</h3></div>
  <div class="card-body">
    <button>Edit</button>
    <button>Delete</button>
  </div>
</div>
```

**Why this output:** The `Card` component is a generic container. It renders whatever children it receives inside `card-body`. The same component is reused with completely different content.

### Real-World Cases

- **Dialog components:** The dialog box is a container; its content varies by use case.
- **Layout components:** Sidebars, panels, and drawers are containers.
- **Design system primitives:** Card, Modal, Popover use containment.

### References

- React — Containment — https://legacy.reactjs.org/docs/composition-vs-inheritance.html#containment
- Clay UI — Containment — https://www.clayui.com/docs/introduction/composition

---

## 4. Specialization

### Definitions

**Core Definition:** Specialization is a composition pattern where a more specific component renders a more generic one and configures it with props.

**Technical Definition:** Sometimes a component is a "special case" of another component. For example, a `WelcomeDialog` is a special case of a `Dialog`. In React, this is achieved by composition: the more specific component renders the more generic one and configures it with props. This is more flexible than inheritance because the relationship is explicit in the code.

**Beginner-Friendly Explanation:** Specialization means creating a specific component by using a general one and setting its options. For example, a `WelcomeDialog` is just a `Dialog` with a welcome message.

### Purposes

- To create specific components from generic ones.
- To avoid inheritance while still achieving code reuse.
- To make component relationships explicit and clear.
- To customize behavior through props rather than class extension.

### Syntax Rules and Structure

**General Syntax:**

```jsx
function GenericComponent(props) {
  return <div>{/* generic layout */}</div>;
}

function SpecificComponent() {
  return <GenericComponent prop="specificValue" />;
}
```

**Syntax Rules:**

1. The specific component renders the generic component.
2. Props configure the generic component's behavior.
3. The specific component may add its own wrapper or additional content.

### Annotated Code Examples

```jsx
// Generic dialog component
function Dialog({ title, message, children }) {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title">{title}</h1>
      <p className="Dialog-message">{message}</p>
      {children}
    </FancyBorder>
  );
}

// Specialized version
function WelcomeDialog() {
  return (
    <Dialog
      title="Welcome"
      message="Thank you for visiting our spacecraft!"
    />
  );
}

// Another specialized version
function ErrorDialog({ errorCode }) {
  return (
    <Dialog
      title="Error"
      message={`Something went wrong (${errorCode}).`}
    >
      <button>Retry</button>
    </Dialog>
  );
}
```

**Expected Output (WelcomeDialog):**

```html
<div class="FancyBorder FancyBorder-blue">
  <h1 class="Dialog-title">Welcome</h1>
  <p class="Dialog-message">Thank you for visiting our spacecraft!</p>
</div>
```

**Expected Output (ErrorDialog):**

```html
<div class="FancyBorder FancyBorder-blue">
  <h1 class="Dialog-title">Error</h1>
  <p class="Dialog-message">Something went wrong (500).</p>
  <button>Retry</button>
</div>
```

**Why this output:** The specific components configure the generic `Dialog` with different props. `ErrorDialog` also passes children (the retry button), showing that specialization and containment can be combined.

### Real-World Cases

- **Dialog variants:** Welcome, error, and confirmation dialogs.
- **Button variants:** Primary, secondary, danger buttons.
- **Alert variants:** Info, warning, error alerts.

### References

- React — Specialization — https://legacy.reactjs.org/docs/composition-vs-inheritance.html#specialization
- Clay UI — Specialization — https://www.clayui.com/docs/introduction/composition

---

## 5. Slots Pattern

### Definitions

**Core Definition:** The slots pattern uses named props to pass JSX elements into specific regions of a layout component, similar to slots in web components or Vue.

**Technical Definition:** React does not have a built-in slot mechanism, but the pattern can be emulated using props that accept JSX elements. For components with multiple content areas (header, footer, actions), named slot patterns are preferred over render props or excessive boolean props. Each named prop acts as a placeholder that the consumer fills with their own markup.

**Beginner-Friendly Explanation:** Slots are like labeled compartments in a component. You can put different content into each compartment without the component needing to know what you are putting there.

### Purposes

- To define multiple content regions in a layout component.
- To allow consumers to control content placement.
- To avoid boolean prop proliferation.
- To create flexible, configurable layouts.

### Syntax Rules and Structure

**General Syntax:**

```jsx
<Layout
  header={<Header />}
  sidebar={<Sidebar />}
  content={<Content />}
  footer={<Footer />}
/>
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `header={<Header />}` | Named slot prop. |
| `sidebar={<Sidebar />}` | Another named slot. |
| `content` | Main content slot. |

**Syntax Rules:**

1. Each slot is a prop whose value is a JSX element.
2. The layout component renders each slot in its designated location.
3. Slots are optional; the component should handle missing slots gracefully.

### Annotated Code Examples

```jsx
function PageLayout({ header, sidebar, content, footer }) {
  return (
    <div className="page">
      {header && <header className="page-header">{header}</header>}
      <div className="page-body">
        {sidebar && <aside className="page-sidebar">{sidebar}</aside>}
        <main className="page-content">{content}</main>
      </div>
      {footer && <footer className="page-footer">{footer}</footer>}
    </div>
  );
}

// Usage
function App() {
  return (
    <PageLayout
      header={<h1>My App</h1>}
      sidebar={<nav><a href="/">Home</a></nav>}
      content={<p>Welcome to the app!</p>}
      footer={<p>© 2026</p>}
    />
  );
}
```

**Expected Output:**

```html
<div class="page">
  <header class="page-header"><h1>My App</h1></header>
  <div class="page-body">
    <aside class="page-sidebar"><nav><a href="/">Home</a></nav></aside>
    <main class="page-content"><p>Welcome to the app!</p></main>
  </div>
  <footer class="page-footer"><p>© 2026</p></footer>
</div>
```

**Why this output:** Each named prop (`header`, `sidebar`, `content`, `footer`) is rendered in its specific location. The layout component controls structure; the consumer controls content.

### Real-World Cases

- **Page layouts:** Header, sidebar, main content, and footer slots.
- **Modal dialogs:** Title, body, and action button slots.
- **Card components:** Header, body, and footer slots.

### References

- React — Passing JSX as Children — https://react.dev/learn/passing-props-to-a-component
- shadcn — Use Slot Pattern for Flexible Content Areas — https://github.com/shipshitdev/skills

---

## 6. Children as Props

### Definitions

**Core Definition:** Children as props is the pattern of using the special `children` prop to pass JSX content into a component.

**Technical Definition:** In JSX, content placed between a component's opening and closing tags is automatically passed as the `children` prop. This is the simplest form of composition. The component can render `{props.children}` or `{children}` (after destructuring) to place the content in its output. Children can be elements, strings, numbers, arrays, fragments, functions, or `null`.

**Beginner-Friendly Explanation:** Anything you put between a component's opening and closing tags becomes its children. The component can render those children wherever it wants.

### Purposes

- To pass arbitrary content to a component.
- To create reusable wrapper components.
- To enable composition without knowledge of the content.
- To simplify APIs by using the built-in `children` prop.

### Syntax Rules and Structure

**General Syntax:**

```jsx
<Wrapper>
  <Content />
  <MoreContent />
</Wrapper>
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `<Wrapper>` | The component that receives children. |
| Content between tags | Becomes `props.children`. |

**Syntax Rules:**

1. `children` is automatically provided by React.
2. Components render children with `{props.children}`.
3. `children` can be explicitly passed as a prop: `<Comp children="text" />`.

### Annotated Code Examples

```jsx
function Card({ title, children }) {
  return (
    <div className="card">
      <h3>{title}</h3>
      <div className="card-body">{children}</div>
    </div>
  );
}

function App() {
  return (
    <Card title="Profile">
      <img src="/avatar.jpg" alt="Avatar" />
      <p>Name: Alice</p>
      <p>Role: Developer</p>
    </Card>
  );
}
```

**Expected Output:**

```html
<div class="card">
  <h3>Profile</h3>
  <div class="card-body">
    <img src="/avatar.jpg" alt="Avatar">
    <p>Name: Alice</p>
    <p>Role: Developer</p>
  </div>
</div>
```

**Why this output:** All content between `<Card>` tags is passed as `children` and rendered inside `card-body`.

### Real-World Cases

- **Layout components:** Any wrapper that needs to render arbitrary content.
- **UI primitives:** Card, Panel, Modal, Dialog.
- **Utility components:** Error boundaries, providers, contexts.

### References

- React — Passing JSX as Children — https://react.dev/learn/passing-props-to-a-component#passing-jsx-as-children

---

## 7. Named Slots

### Definitions

**Core Definition:** Named slots use custom props (instead of `children`) to pass JSX into specific regions of a component.

**Technical Definition:** When a component needs multiple content areas, `children` alone is insufficient. Named slots solve this by using custom props (e.g., `header`, `footer`, `actions`) that accept JSX elements. This is the React equivalent of named slots in web components or Vue. It is preferred over render props or boolean props for static content composition.

**Beginner-Friendly Explanation:** Named slots are like having multiple `children` props, each with a name. You can pass different content to each named slot.

### Purposes

- To define multiple content areas in a component.
- To avoid overloading the single `children` prop.
- To provide explicit, self-documenting composition APIs.
- To replace boolean props for layout variations.

### Syntax Rules and Structure

**General Syntax:**

```jsx
<Component
  slot1={<Content1 />}
  slot2={<Content2 />}
  slot3={<Content3 />}
/>
```

**Syntax Rules:**

1. Each slot is a prop with a descriptive name.
2. Slots accept JSX elements.
3. The component renders each slot in its designated location.

### Annotated Code Examples

```jsx
function Modal({ title, body, actions }) {
  return (
    <div className="modal">
      <div className="modal-header">
        <h2>{title}</h2>
      </div>
      <div className="modal-body">
        {body}
      </div>
      <div className="modal-footer">
        {actions}
      </div>
    </div>
  );
}

function App() {
  return (
    <Modal
      title="Confirm Delete"
      body={<p>Are you sure you want to delete this item?</p>}
      actions={
        <>
          <button>Cancel</button>
          <button className="danger">Delete</button>
        </>
      }
    />
  );
}
```

**Expected Output:**

```html
<div class="modal">
  <div class="modal-header"><h2>Confirm Delete</h2></div>
  <div class="modal-body"><p>Are you sure you want to delete this item?</p></div>
  <div class="modal-footer">
    <button>Cancel</button>
    <button class="danger">Delete</button>
  </div>
</div>
```

**Why this output:** Three named slots (`title`, `body`, `actions`) each receive JSX and are rendered in their specific locations.

### Real-World Cases

- **Modal dialogs:** Title, body, and action buttons.
- **Card components:** Header, body, and footer.
- **Page layouts:** Header, sidebar, content, footer.

### References

- shadcn — Use Slot Pattern for Flexible Content Areas — https://github.com/shipshitdev/skills
- @aiera-inc/react-slots — https://www.npmjs.com/package/@aiera-inc/react-slots

---

## 8. Higher-Order Components (HOC)

### Definitions

**Core Definition:** A higher-order component is a function that takes a component and returns a new, enhanced component.

**Technical Definition:** A higher-order component (HOC) is an advanced technique in React for reusing component logic. Concretely, a HOC is a function that takes a component and returns a new component. Whereas a component transforms props into UI, a HOC transforms a component into another component. HOCs are not part of the React API; they are a pattern that emerges from React's compositional nature. HOCs are common in third-party libraries like Redux's `connect` and Relay's `createFragmentContainer`.

**Beginner-Friendly Explanation:** A HOC is a function that wraps a component to give it extra abilities. It is like putting a component inside a special container that adds features.

### Purposes

- To reuse component logic across multiple components.
- To handle cross-cutting concerns (authentication, logging, data fetching).
- To add props or behavior to a component without modifying it.
- To abstract away repetitive code patterns.

### Syntax Rules and Structure

**General Syntax:**

```jsx
function withExtraProps(WrappedComponent) {
  return function EnhancedComponent(props) {
    const extraProps = { ... };
    return <WrappedComponent {...props} {...extraProps} />;
  };
}

const Enhanced = withExtraProps(MyComponent);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `withExtraProps` | The HOC function. |
| `WrappedComponent` | The component being enhanced. |
| `EnhancedComponent` | The new component returned by the HOC. |

**Syntax Rules:**

1. HOCs are pure functions with no side effects.
2. Pass unrelated props through to the wrapped component.
3. Do not mutate the original component.
4. Use `displayName` for easier debugging.

### Annotated Code Examples

```jsx
function withAuth(WrappedComponent) {
  return function WithAuth(props) {
    const { user, isLoading } = useAuth();  // Custom hook

    if (isLoading) return <Spinner />;
    if (!user) return <Redirect to="/login" />;

    return <WrappedComponent {...props} user={user} />;
  };
}

// Usage
function Dashboard({ user }) {
  return <h1>Welcome, {user.name}!</h1>;
}

const ProtectedDashboard = withAuth(Dashboard);

// Render
<ProtectedDashboard />
```

**Expected Output:** The dashboard renders only if the user is authenticated.

**Why this output:** `withAuth` wraps `Dashboard` and checks authentication before rendering. If authenticated, it passes the user as a prop.

### Real-World Cases

- **Authentication:** `withAuth`, `withPermission`.
- **Data fetching:** `withData`, `withSubscription`.
- **Logging:** `withLogger`.
- **Redux:** `connect`.

### References

- React — Higher-Order Components — https://ru.react.js.org/docs/higher-order-components.html

---

## 9. Render Props

### Definitions

**Core Definition:** A render prop is a prop whose value is a function that returns JSX, allowing a component to share logic while delegating rendering.

**Technical Definition:** The term "render prop" refers to a technique for sharing code between React components using a prop whose value is a function. A component with a render prop takes a function as a prop and calls it to determine what to render, receiving data or state from the component. This inverts control: the component provides the logic, and the consumer decides what to render.

**Beginner-Friendly Explanation:** A render prop is a function you pass to a component that tells it what to render. The component gives your function some data, and your function returns what to show.

### Purposes

- To share stateful logic between components.
- To invert control of rendering to the consumer.
- To provide flexible APIs where the consumer controls rendering.
- To avoid HOC nesting ("wrapper hell").

### Syntax Rules and Structure

**General Syntax:**

```jsx
<DataProvider render={(data) => <Child data={data} />} />
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `render` | The prop that accepts a function. |
| `(data) =>` | The function receives data from the provider. |
| `<Child data={data} />` | The function returns JSX. |

**Syntax Rules:**

1. The render prop is a function, not JSX.
2. The component calls the function with data or state.
3. The function returns React elements.

### Annotated Code Examples

```jsx
function MouseTracker({ render }) {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  useEffect(() => {
    const handleMove = (e) => setPosition({ x: e.clientX, y: e.clientY });
    window.addEventListener('mousemove', handleMove);
    return () => window.removeEventListener('mousemove', handleMove);
  }, []);

  return render(position);
}

// Usage
<MouseTracker
  render={({ x, y }) => (
    <p>Mouse position: {x}, {y}</p>
  )}
/>
```

**Expected Output:** The mouse position updates as the user moves the mouse.

**Why this output:** `MouseTracker` provides the mouse position to the render function, which decides what to render.

### Real-World Cases

- **Legacy libraries:** Many older React libraries use render props.
- **Custom Hooks migration:** Render props can be replaced by custom Hooks.
- **Flexible rendering:** When a component needs to delegate rendering decisions.

### References

- React — Render Props — https://legacy.reactjs.org/docs/render-props.html

---

## 10. Compound Components

### Definitions

**Core Definition:** Compound components are a set of related components that work together, sharing implicit state through context.

**Technical Definition:** Compound components structure complex components as a group of smaller, focused subcomponents that share state via React Context. The parent component owns the state, and children consume it through context. This pattern is used by libraries like Radix UI and Headless UI. It avoids boolean prop proliferation and provides a clean, expressive API. Instead of passing dozens of props down, related pieces are grouped under a shared context. Externally, the consumer controls layout and styling; internally, state coordination happens through context.

**Beginner-Friendly Explanation:** Compound components are like a set of related parts that work together. For example, a `<Tabs>` component comes with `<Tabs.List>`, `<Tabs.Tab>`, and `<Tabs.Panel>`. They share information automatically.

### Purposes

- To create expressive, self-documenting component APIs.
- To share state between related components without prop drilling.
- To avoid boolean prop proliferation.
- To enable flexible composition of complex UI patterns.

### Syntax Rules and Structure

**General Syntax:**

```jsx
<Tabs>
  <Tabs.List>
    <Tabs.Tab value="tab1">Tab 1</Tabs.Tab>
    <Tabs.Tab value="tab2">Tab 2</Tabs.Tab>
  </Tabs.List>
  <Tabs.Panel value="tab1">Content 1</Tabs.Panel>
  <Tabs.Panel value="tab2">Content 2</Tabs.Panel>
</Tabs>
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `<Tabs>` | Parent that owns state and provides context. |
| `<Tabs.List>` | Renders the tab list. |
| `<Tabs.Tab>` | A single tab button. |
| `<Tabs.Panel>` | Content panel for a tab. |

**Syntax Rules:**

1. The parent component provides state via context.
2. Subcomponents consume context to access and modify state.
3. Subcomponents are exported as static properties of the parent.
4. The consumer composes only the parts they need.

### Annotated Code Examples

```jsx
import { createContext, useContext, useState } from 'react';

const TabsContext = createContext();

function Tabs({ children, defaultTab }) {
  const [activeTab, setActiveTab] = useState(defaultTab);
  return (
    <TabsContext.Provider value={{ activeTab, setActiveTab }}>
      {children}
    </TabsContext.Provider>
  );
}

function TabList({ children }) {
  return <div role="tablist">{children}</div>;
}

function Tab({ value, children }) {
  const { activeTab, setActiveTab } = useContext(TabsContext);
  return (
    <button
      role="tab"
      aria-selected={activeTab === value}
      onClick={() => setActiveTab(value)}
    >
      {children}
    </button>
  );
}

function TabPanel({ value, children }) {
  const { activeTab } = useContext(TabsContext);
  return activeTab === value ? <div role="tabpanel">{children}</div> : null;
}

// Attach subcomponents
Tabs.List = TabList;
Tabs.Tab = Tab;
Tabs.Panel = TabPanel;

// Usage
<Tabs defaultTab="profile">
  <Tabs.List>
    <Tabs.Tab value="profile">Profile</Tabs.Tab>
    <Tabs.Tab value="settings">Settings</Tabs.Tab>
  </Tabs.List>
  <Tabs.Panel value="profile">Profile content</Tabs.Panel>
  <Tabs.Panel value="settings">Settings content</Tabs.Panel>
</Tabs>
```

**Expected Output:** A tabbed interface with Profile and Settings tabs.

**Why this output:** `Tabs` manages the active tab state and shares it via context. `Tab` and `TabPanel` consume the context to render correctly.

### Real-World Cases

- **Radix UI:** All primitives use compound components.
- **Headless UI:** Tabs, Dialog, Listbox, Menu.
- **Design systems:** Accordion, Dropdown, Select.

### References

- Headless Component APIs — Educative — https://www.educative.io/courses/learn-react/lta/headless-component-apis
- React Composition Patterns — Compound Components — https://github.com/sergiodxa/agent-skills

---

## 11. Headless Components

### Definitions

**Core Definition:** Headless components expose behavior and state without prescribing any UI markup or styling.

**Technical Definition:** A headless component does not render DOM directly. Instead, it provides an API — often through hooks or compound components — that consumers use to attach behavior to their own markup. The pattern separates the behavior layer (state, keyboard interactions, focus management, accessibility semantics) from the presentation layer (layout, styles, animation). This enables maximum flexibility: different visual structures can reuse the same behavior, styling systems remain independent, and interaction logic stays centralized and testable.

**Beginner-Friendly Explanation:** Headless components give you the logic but not the look. You build your own UI and attach the headless component's behavior to it.

### Purposes

- To separate behavior from presentation.
- To enable multiple UI implementations of the same logic.
- To maintain accessibility without prescribing markup.
- To avoid rigid, over-configured components.

### Syntax Rules and Structure

**General Syntax (Hook-based):**

```jsx
const { isOpen, toggle, getTriggerProps } = useDropdown();

<button {...getTriggerProps()}>Toggle</button>
{isOpen && <div>Content</div>}
```

**General Syntax (Compound Components):**

```jsx
<Dropdown>
  <Dropdown.Trigger>Toggle</Dropdown.Trigger>
  <Dropdown.Menu>
    <Dropdown.Item>Item 1</Dropdown.Item>
  </Dropdown.Menu>
</Dropdown>
```

**Syntax Rules:**

1. Headless components expose state and event bindings.
2. Consumers attach these to their own markup.
3. No styling is prescribed.
4. Accessibility is handled by the behavior layer.

### Annotated Code Examples

```jsx
function useDropdown() {
  const [isOpen, setIsOpen] = useState(false);

  const toggle = () => setIsOpen((o) => !o);
  const close = () => setIsOpen(false);

  return {
    isOpen,
    toggle,
    close,
    getTriggerProps: () => ({
      onClick: toggle,
      'aria-expanded': isOpen,
    }),
    getMenuProps: () => ({
      role: 'menu',
    }),
    getItemProps: () => ({
      role: 'menuitem',
      onClick: close,
    }),
  };
}

// Usage
function MyDropdown() {
  const { isOpen, getTriggerProps, getMenuProps, getItemProps } = useDropdown();

  return (
    <div>
      <button {...getTriggerProps()}>Menu</button>
      {isOpen && (
        <ul {...getMenuProps()}>
          <li {...getItemProps()}>Item 1</li>
          <li {...getItemProps()}>Item 2</li>
        </ul>
      )}
    </div>
  );
}
```

**Expected Output:** A dropdown menu with a toggle button and items.

**Why this output:** The `useDropdown` hook provides the behavior (open/close state, ARIA attributes, event handlers). The consumer provides the markup and styling.

### Real-World Cases

- **Downshift:** Headless autocomplete/select library.
- **React Aria:** Adobe's headless accessibility hooks.
- **TanStack Table:** Headless table logic.

### References

- Headless Component APIs — Educative — https://www.educative.io/courses/learn-react/lta/headless-component-apis
- MUI Base UI — Headless Component Guidelines — https://github.com/pproenca/dot-skills

---

## 12. Controlled Components

### Definitions

**Core Definition:** A controlled component is a form element whose value is controlled by React state.

**Technical Definition:** In a controlled component, form data is handled by a React component. The input's value is set via the `value` prop, and changes are handled via an `onChange` handler that updates state. React becomes the single source of truth for the input's value. Controlled components are recommended for most form implementations because they make the data flow explicit and predictable.

**Beginner-Friendly Explanation:** With a controlled component, React controls what the input shows. When the user types, React updates the state, and the input shows the new state value.

### Purposes

- To make form state explicit and predictable.
- To enable real-time validation and transformation.
- To allow conditional disabling or resetting of inputs.
- To provide a single source of truth for form data.

### Syntax Rules and Structure

**General Syntax:**

```jsx
const [value, setValue] = useState('');

<input
  value={value}
  onChange={(e) => setValue(e.target.value)}
/>
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `value={value}` | The input's value comes from state. |
| `onChange` | Updates state when the user types. |

**Syntax Rules:**

1. The `value` prop is set from state.
2. The `onChange` handler updates state.
3. The state is the single source of truth.

### Annotated Code Examples

```jsx
function NameForm() {
  const [name, setName] = useState('');

  function handleSubmit(e) {
    e.preventDefault();
    alert(`Submitted: ${name}`);
  }

  return (
    <form onSubmit={handleSubmit}>
      <label>
        Name:
        <input
          type="text"
          value={name}
          onChange={(e) => setName(e.target.value)}
        />
      </label>
      <button type="submit">Submit</button>
    </form>
  );
}
```

**Expected Output:** A form where the input value is controlled by React state. Submitting alerts the current name.

**Why this output:** The input's value is bound to `name` state. Typing triggers `onChange`, which updates state, which re-renders the input with the new value.

### Real-World Cases

- **Form validation:** Validating input as the user types.
- **Conditional fields:** Showing/hiding fields based on other values.
- **Form reset:** Resetting all fields by resetting state.

### References

- React — Controlled Components — https://ru.react.js.org/docs/forms.html#controlled-components
- Sharing State Between Components — https://ar.react.dev

---

## 13. Uncontrolled Components

### Definitions

**Core Definition:** An uncontrolled component is a form element whose value is managed by the DOM itself, not React state.

**Technical Definition:** In an uncontrolled component, form data is handled by the DOM. Instead of writing an event handler for every state update, you use a ref to get form values from the DOM. Uncontrolled components can be easier to integrate with non-React code, and require less code for simple forms. However, controlled components are generally recommended for most cases. Default values are specified with `defaultValue` (for inputs), `defaultChecked` (for checkboxes/radios), and `defaultValue` (for `<select>` and `<textarea>`).

**Beginner-Friendly Explanation:** With an uncontrolled component, the DOM itself keeps track of the input value. React only reads the value when needed (e.g., on submit) using a ref.

### Purposes

- To integrate with non-React code more easily.
- To write less code for simple forms.
- To handle file inputs (which are always uncontrolled in React).

### Syntax Rules and Structure

**General Syntax:**

```jsx
const inputRef = useRef();

<input ref={inputRef} defaultValue="Initial" />

// Access value: inputRef.current.value
```

**Syntax Rules:**

1. Use `useRef` to access the DOM node.
2. Use `defaultValue` / `defaultChecked` for initial values.
3. Read values from `ref.current.value` when needed.

### Annotated Code Examples

```jsx
function NameForm() {
  const inputRef = useRef();

  function handleSubmit(e) {
    e.preventDefault();
    alert(`Submitted: ${inputRef.current.value}`);
  }

  return (
    <form onSubmit={handleSubmit}>
      <label>
        Name:
        <input type="text" ref={inputRef} defaultValue="Bob" />
      </label>
      <button type="submit">Submit</button>
    </form>
  );
}
```

**Expected Output:** A form where the input starts with "Bob" and the value is read on submit.

**Why this output:** The input is uncontrolled; the DOM manages its value. On submit, the value is read from the ref.

### Real-World Cases

- **File inputs:** `<input type="file" />` is always uncontrolled.
- **Simple forms:** When validation is not needed.
- **Third-party integration:** When integrating with non-React libraries.

### References

- React — Uncontrolled Components — https://ru.react.js.org/docs/uncontrolled-components.html

---

## 14. Presentational Components

### Definitions

**Core Definition:** Presentational components are concerned with how things look; they receive data and callbacks exclusively via props.

**Technical Definition:** The Container/Presentational pattern, popularized by Dan Abramov in 2015, separates components into two categories. Presentational components (also called "dumb components") are concerned with how things look. They do not specify how data is loaded or mutated; they receive data and callbacks exclusively via props. They are usually stateless and focus purely on rendering. This separation allows UI changes without touching application logic.

**Beginner-Friendly Explanation:** Presentational components are the "look" components. They just render what they are given and do not worry about where the data comes from.

### Purposes

- To separate UI from business logic.
- To make UI components reusable across different data sources.
- To simplify testing of pure rendering logic.
- To allow designers and developers to work on UI independently.

### Syntax Rules and Structure

**General Syntax:**

```jsx
function UserCard({ name, email, onEdit }) {
  return (
    <div className="user-card">
      <h2>{name}</h2>
      <p>{email}</p>
      <button onClick={onEdit}>Edit</button>
    </div>
  );
}
```

**Syntax Rules:**

1. Receives data via props.
2. No state (or only UI state).
3. No data fetching or business logic.
4. Renders based purely on props.

### Annotated Code Examples

```jsx
// Presentational component
function DogImages({ dogs }) {
  return (
    <div className="dog-images">
      {dogs.map((dog, index) => (
        <img key={index} src={dog} alt="Dog" />
      ))}
    </div>
  );
}
```

**Expected Output:** A grid of dog images.

**Why this output:** The component receives `dogs` via props and renders them. It does not fetch data or manage state.

### Real-World Cases

- **Design systems:** All UI primitives are presentational.
- **Storybook:** Presentational components are perfect for Storybook.
- **Testing:** Easy to test with different prop combinations.

### References

- Patterns.dev — Container/Presentational Pattern — https://www.patterns.dev/vue/container-presentational
- Dan Abramov — Presentational and Container Components — https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0

---

## 15. Container Components

### Definitions

**Core Definition:** Container components are concerned with how things work; they provide data and behavior to presentational components.

**Technical Definition:** Container components (also called "smart components") are concerned with how things work. They provide data and behavior to presentational or other container components. They typically hold state, fetch data, and pass results to presentational components via props. With the emergence of Hooks, the clear boundary between presentational and container components has blurred, as logic can now be encapsulated in custom Hooks.

**Beginner-Friendly Explanation:** Container components are the "logic" components. They fetch data, manage state, and pass everything to presentational components for display.

### Purposes

- To separate data fetching and business logic from UI.
- To make logic reusable across different presentational components.
- To test business logic independently of UI.

### Syntax Rules and Structure

**General Syntax:**

```jsx
function UserContainer() {
  const [user, setUser] = useState(null);

  useEffect(() => {
    fetchUser().then(setUser);
  }, []);

  if (!user) return <Spinner />;
  return <UserCard name={user.name} email={user.email} />;
}
```

**Syntax Rules:**

1. Manages state and side effects.
2. Fetches data or performs computations.
3. Passes results to presentational components via props.
4. May render loading/error states.

### Annotated Code Examples

```jsx
function DogImagesContainer() {
  const [dogs, setDogs] = useState([]);

  useEffect(() => {
    fetch('https://dog.ceo/api/breeds/image/random/6')
      .then((res) => res.json())
      .then((data) => setDogs(data.message));
  }, []);

  return <DogImages dogs={dogs} />;
}
```

**Expected Output:** A loading state followed by a grid of dog images.

**Why this output:** The container fetches dog images and passes them to the presentational `DogImages` component.

### Real-World Cases

- **Data fetching:** Components that fetch and pass data to presentational children.
- **State management:** Components that manage complex state and pass it down.
- **Redux:** `connect` creates container components.

### References

- Patterns.dev — Container/Presentational Pattern — https://www.patterns.dev/vue/container-presentational

---

## 16. Smart vs Dumb Components

### Definitions

**Core Definition:** "Smart" and "dumb" are informal aliases for container and presentational components.

**Technical Definition:** The canonical name for this pattern is Container Presentational. The alias "Smart and Dumb Components" is the informal name popularized by Dan Abramov's article. Container components are the "smart" ones, holding state and logic; presentational components are the "dumb" ones, purely rendering what they are given. Since there cannot be state or extra logic, pure functional components are always presentational.

**Beginner-Friendly Explanation:** Smart components are the brains; dumb components are the looks. Smart components handle logic; dumb components just display what they are told.

### Purposes

- To provide an intuitive mental model for component roles.
- To simplify communication about component responsibilities.
- To encourage separation of concerns.

### Syntax Rules and Structure

**Smart (Container):**

```jsx
function UserContainer() {
  const [user, setUser] = useState(null);
  // Fetch data, manage state
  return <UserCard user={user} />;
}
```

**Dumb (Presentational):**

```jsx
function UserCard({ user }) {
  return <h2>{user.name}</h2>;
}
```

**Syntax Rules:**

1. Smart components manage state and logic.
2. Dumb components render UI from props.
3. Dumb components are more reusable.

### Annotated Code Examples

```jsx
// Smart component
function UserListContainer() {
  const [users, setUsers] = useState([]);

  useEffect(() => {
    fetchUsers().then(setUsers);
  }, []);

  return <UserList users={users} />;
}

// Dumb component
function UserList({ users }) {
  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

**Expected Output:** A list of user names.

**Why this output:** The smart component fetches users; the dumb component renders them.

### Real-World Cases

- **Team communication:** Clear roles for components.
- **Code organisation:** Separating logic from presentation.

### References

- Container Presentational Pattern — https://raw.githubusercontent.com
- Patterns.dev — Container/Presentational Pattern — https://www.patterns.dev/vue/container-presentational

---

## 17. Composition Patterns

### Definitions

**Core Definition:** Composition patterns are reusable architectural solutions for common React component design problems.

**Technical Definition:** Modern React composition patterns include replacing boolean props with composition, building compound components with context, creating explicit variant components, and preferring children over render props. These patterns help avoid boolean prop proliferation and make components more flexible and maintainable. Key rules include: avoid adding boolean props to customize behavior, structure complex components as compound components with shared context, and prefer children over render props for composition.

**Beginner-Friendly Explanation:** Composition patterns are recipes for building components in a clean, flexible way. They help you avoid common design problems.

### Purposes

- To avoid boolean prop proliferation.
- To create flexible, composable component APIs.
- To keep components maintainable as requirements grow.
- To align with modern React best practices.

### Syntax Rules and Structure

**Core Patterns:**

| Pattern | Description |
|---------|-------------|
| Children over render props | Use `children` for composition instead of `renderX` props. |
| Compound components | Structure as compound components with shared context. |
| Explicit variants | Create `ThreadComposer`, `EditComposer` instead of `Composer` with `isThread`. |
| Lift state | State in providers, not trapped in components. |
| Compose internals | Subcomponents access context, not props. |

**Syntax Rules:**

1. Avoid boolean props like `isThread`, `isEditing`, `hasIcon`.
2. Create explicit variant components instead.
3. Use compound components for complex UI patterns.
4. Prefer children over render props for static composition.

### Annotated Code Examples

```jsx
// ❌ Boolean prop proliferation
function Composer({ isThread, isEditing, hasAttachments, ... }) {
  // Complex conditional logic
}

// ✅ Explicit variants
function ThreadComposer() {
  return <Composer layout="thread" />;
}

function EditComposer() {
  return <Composer layout="edit" />;
}

// ✅ Compound components
<Tabs>
  <Tabs.List>
    <Tabs.Tab value="a">A</Tabs.Tab>
    <Tabs.Tab value="b">B</Tabs.Tab>
  </Tabs.List>
  <Tabs.Panel value="a">Panel A</Tabs.Panel>
  <Tabs.Panel value="b">Panel B</Tabs.Panel>
</Tabs>
```

**Expected Output:** Clean, explicit component APIs.

**Why this output:** Explicit variants and compound components make the API self-documenting and flexible.

### Real-World Cases

- **Design systems:** All modern component libraries follow these patterns.
- **Scalable applications:** Patterns prevent code rot as features grow.
- **Team collaboration:** Clear patterns reduce inconsistency.

### References

- React Composition Patterns — https://github.com/PracticalSwan/agent-skills
- Vercel Composition Patterns — https://github.com/vercel-labs/agent-skills

---

## References

- React — Composition vs Inheritance — https://legacy.reactjs.org/docs/composition-vs-inheritance.html
- React — Passing JSX as Children — https://react.dev/learn/passing-props-to-a-component
- React — Higher-Order Components — https://ru.react.js.org/docs/higher-order-components.html
- React — Render Props — https://legacy.reactjs.org/docs/render-props.html
- React — Uncontrolled Components — https://ru.react.js.org/docs/uncontrolled-components.html
- Patterns.dev — Container/Presentational Pattern — https://www.patterns.dev/vue/container-presentational
- Clay UI — Composition Philosophy — https://www.clayui.com/docs/introduction/composition
- Educative — Headless Component APIs — https://www.educative.io/courses/learn-react/lta/headless-component-apis
- React Composition Patterns (PracticalSwan) — https://github.com/PracticalSwan/agent-skills
- Vercel Composition Patterns — https://github.com/vercel-labs/agent-skills
- shadcn — Use Slot Pattern for Flexible Content Areas — https://github.com/shipshitdev/skills
- @aiera-inc/react-slots — https://www.npmjs.com/package/@aiera-inc/react-slots
- Dan Abramov — Presentational and Container Components — https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0