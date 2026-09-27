# React Component Composition: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** React component composition is the architectural practice of building complex user interfaces by combining smaller, self-contained components together, where each component encapsulates its own structure, logic, and styling, and communicates with other components through props.

**Technical Definition:** Component composition in React is a design paradigm in which components are treated as composable units that can be nested, combined, and configured to construct larger UI structures. It leverages React's component model — where a component is a function that returns React elements — to create hierarchies of parent and child components. Data flows unidirectionally from parent to child through props, including the special `children` prop, which allows a component to receive arbitrary JSX content. React's official documentation explicitly recommends composition over inheritance for code reuse between components, stating that "React has a powerful composition model, and we recommend using composition instead of inheritance to reuse code between components". The composition model encompasses containment (components receiving unknown children), specialization (generic components configured by more specific ones), and advanced patterns such as compound components and slots.

**Beginner-Friendly Explanation:** Think of building with Lego bricks. Each brick (component) is a small, self-contained piece. You can snap bricks together (compose them) to build anything from a simple wall to an entire castle. A brick doesn't need to know what it will be part of — it just does its job. And you never need to modify a brick's internal design to make it fit; you just use different bricks in different combinations. That's component composition: building big things from small, reusable pieces.

### Key Characteristics

- **Composition Over Inheritance:** React recommends composition instead of inheritance for code reuse; props and composition provide flexibility without the coupling of class hierarchies.
- **Parent-Child Hierarchy:** Components nest to form a tree, with parent components rendering child components and passing data downward.
- **Unidirectional Data Flow:** Data flows from parent to child via props; children communicate upward through callback props.
- **Containment via `children`:** The `children` prop allows components to receive arbitrary JSX content, enabling generic "box" components like dialogs and sidebars.
- **Specialization via Composition:** More specific components render more generic ones and configure them with props, achieving a form of "specialization" without inheritance.
- **Slot-Based Composition:** Named props (e.g., `header`, `footer`, `left`, `right`) provide multiple content areas, overcoming the single-slot limitation of `children`.
- **Compound Components:** A parent component manages shared state while child subcomponents control structure, sharing state implicitly through context.

### Prerequisites

- Solid understanding of React function components, JSX syntax, and the component tree.
- Familiarity with props, including passing objects, functions, and JSX as prop values.
- Working knowledge of the `children` prop and how JSX nesting translates to prop passing.
- Basic understanding of the `useState` Hook (for compound components that share state).
- Awareness of the `useContext` Hook (for compound components that share state via context).

### Related Programming Areas

- **Design Systems:** Building reusable component libraries with consistent APIs.
- **State Management:** Sharing state between components through composition and context.
- **Component Architecture:** Organising components into trees, layouts, and compound structures.
- **Code Reuse:** Reducing duplication through composition instead of inheritance.
- **Accessibility:** Ensuring composed components maintain semantic HTML and keyboard navigation.

### Core Concepts / Features

1. Parent-Child Relationships
2. Nested Components
3. Component Slots Using `children`
4. Reusable Layout Components
5. Composition versus Inheritance
6. Compound Components

---

## Core Concept 1: Parent-Child Relationships

### Definitions

**Core Definition:** A parent-child relationship in React is the hierarchical relationship formed when one component renders another component in its JSX output, where the rendering component is the "parent" and the rendered component is the "child."

**Technical Definition:** In React's component model, when a component's JSX includes another component as an element, that nesting creates a parent-child relationship. The parent component passes data to the child through props, and the child component receives that data as its first argument (in function components) or via `this.props` (in class components). This relationship forms the basis of React's render tree, a hierarchical representation of all components rendered in a single render pass. Communication between components depends on the direction: parents communicate to children via props, and when state changes, the props automatically change; children communicate to parents via callbacks. Each parent component may itself be a child of another component, creating arbitrarily deep hierarchies.

**Beginner-Friendly Explanation:** A parent-child relationship is like a parent telling a child what to do. The parent (component) says, "Here is some information (props)." The child receives the information and uses it to decide what to display. The child cannot change the information directly — it can only ask the parent (via a callback) to change it. This one-way flow keeps things predictable.

### Purposes

- To establish a clear hierarchy of responsibility, where parents own data and children render it.
- To enable data flow from parent to child through props.
- To allow parents to configure child components with different data and behaviour.
- To create a tree structure that React can efficiently reconcile and update.
- To support the "lifting state up" pattern, where shared state is moved to a common parent.

### Syntax Rules and Structure

**General Syntax (Parent Renders Child):**
```jsx
// Parent component
function Parent() {
  return <Child name="Alice" age={30} />;
}

// Child component
function Child({ name, age }) {
  return <p>{name} is {age} years old.</p>;
}
```

**Component Breakdown:**
- `<Child name="Alice" age={30} />`: The parent renders the child with props.
- `function Child({ name, age })`: The child destructures the props it receives.
- The child uses the props to render output.

**General Syntax (Parent with Multiple Children):**
```jsx
function Parent() {
    return (
        <div>
            <Child name="Alice" />
            <Child name="Bob" />
            <Child name="Charlie" />
        </div>
    );
}
```

**Component Breakdown:**
- The parent renders three instances of `Child`.
- Each instance receives different props.
- React reconciles them by position.

**Syntax Rules:**
- A parent can render multiple children; each child instance is independent.
- Props flow downward only; children cannot directly modify parent state.
- If a child needs to communicate upward, the parent passes a callback function as a prop.
- Never nest component *definitions* inside other components; define them at the top level and nest instances in JSX.

**Constraints and Limitations:**
- Props are read-only in the child; mutating them is forbidden.
- A child cannot access its parent's state directly; it must receive it via props.
- Deeply nested prop drilling can become cumbersome (solved by composition or context).
- When a parent re-renders, all its children re-render unless memoised.

### Annotated Code Example: Parent-Child with Props and Callback

```jsx
// Parent component: owns state and passes it down
function TemperatureApp() {
    const [temperature, setTemperature] = React.useState(20);
  
    return (
        <div>
            <h1>Temperature: {temperature}°C</h1>
            {/* Pass value and callback to child */}
            <TemperatureInput
                temperature={temperature}
                onTemperatureChange={setTemperature}
            />
        </div>
    );
}

// Child component: controlled input that reports changes to parent
function TemperatureInput({ temperature, onTemperatureChange }) {
  function handleChange(e) {
      // Call the parent's callback with the new value
      onTemperatureChange(Number(e.target.value));
  }

    return (
        <div>
            <label>
                Set temperature:
                <input type="number" value={temperature} onChange={handleChange} />
            </label>
        </div>
    );
}
```

**Expected Output:** The parent displays the current temperature (initially 20°C). The child renders a number input pre-filled with 20. When the user types a new number, the parent's state updates and the displayed temperature changes in real time.

**Why This Output Occurs:** The `temperature` state lives in the parent (`TemperatureApp`). The parent passes the current value (`temperature`) and a callback (`onTemperatureChange`, which is `setTemperature`) to the child. When the user types, the child calls `onTemperatureChange(newValue)`, which updates the parent's state. The parent re-renders with the new temperature, passing the updated value back down to the child.

### Real-World Cases

- **Form inputs:** A parent form component owns the form state and passes values and change handlers to child input components.
- **Lists:** A parent list component renders multiple child item components, each receiving item data as props.
- **Layouts:** A parent layout component passes configuration props to child header, sidebar, and footer components.
- **Themed components:** A parent theme provider passes theme values to child components via props or context.

### References

- React Official Documentation – Passing Props to a Component: https://react.dev/learn/passing-props-to-a-component
- React Official Documentation – Understanding Your UI as a Tree: https://react.dev/learn/understanding-your-ui-as-a-tree
- React Official Documentation – Sharing State Between Components: https://react.dev/learn/sharing-state-between-components

---

## Core Concept 2: Nested Components

### Definitions

**Core Definition:** Nested components are components that are defined inside other components (either in their render output or in their function body), creating a parent-child hierarchy where the outer component renders the inner component.

**Technical Definition:** In React, nesting occurs when a component's JSX includes other components as children. When you nest one component inside another, you are creating a parent → child relationship. The parent component receives the array of nested components as `props.children`; if there is only one child, it receives the component itself without the array wrapper for performance reasons. There are two distinct meanings of "nested components" in React: (1) **JSX nesting** — rendering component instances inside other components' JSX, which is the recommended and standard pattern; and (2) **Definition nesting** — defining a component's function inside another component's function body, which is an anti-pattern because the nested component is recreated on every render, losing state and causing performance issues.

**Beginner-Friendly Explanation:** Nesting components is like putting boxes inside boxes. The outer box (parent) holds the inner boxes (children). When you nest a component in JSX, React passes the inner components to the outer one as `children`. However, you should never define a component inside another component's function — that would be like creating a brand-new box every time you open the outer box, which is wasteful and confusing.

### Purposes

- To build complex UIs by combining simple components into hierarchies.
- To allow parent components to control the arrangement and structure of child components.
- To enable containment, where a parent component wraps arbitrary child content.
- To support recursive rendering patterns (e.g., trees, nested comments).
- To create reusable layout components that accept arbitrary children.

### Syntax Rules and Structure

**JSX Nesting (Recommended):**
```jsx
function Card({ children }) {
    return <div className="card">{children}</div>;
}

function App() {
    return (
        <Card>
            <h1>Title</h1>
            <p>Content inside the card</p>
        </Card>
    );
}
```

**Component Breakdown:**
- `<Card>` wraps two elements: an `<h1>` and a `<p>`.
- `Card` receives them as `children` and renders them inside a `<div>`.
- The nesting is in JSX, not in the component definition.

**Definition Nesting (Anti-Pattern):**
```jsx
// ❌ Anti-pattern: Child defined inside Parent
function Parent() {
    function Child() {
        return <p>Child</p>;
    }

    return <Child />;
}

// Child is recreated on every Parent render.
```

**Correct Pattern:**
```jsx
// ✅ Correct: Child defined outside Parent
function Child() {
    return <p>Child</p>;
}

function Parent() {
    return <Child />;
}
```

**Syntax Rules:**
- Nest component *instances* in JSX, not component *definitions* in function bodies.
- The parent receives nested JSX as `children` (or as named props for multiple slots).
- Never define a component inside another component's function body; it causes the component to be recreated on every render.
- Use keys when rendering arrays of nested components.

**Constraints and Limitations:**
- JSX nesting is limited to a single `children` slot unless you use named props for multiple slots.
- Definition nesting breaks React's reconciliation and causes state loss.
- Deeply nested JSX can become difficult to read; extract subcomponents for clarity.
- Nested components cannot directly pass props to each other; they communicate through the parent.

### Annotated Code Example: Nested Layout with Children

```jsx
// Generic card component that accepts arbitrary children
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

// App composes the Card with different content
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

**Expected Output:** Two card containers are rendered. The first has the title "Welcome" and contains a paragraph. The second has the title "Media" and contains an image and a button. Both cards share the same wrapper structure, but their inner content differs entirely.

**Why This Output Occurs:** The `Card` component doesn't know or care what content it will contain — it simply renders whatever JSX is passed as `children`. The parent component decides what goes inside each card by nesting JSX between the `<Card>` tags. This is the composition pattern: the parent controls the structure and content, while the child provides the reusable wrapper and styling.

### Real-World Cases

- **Layout components:** A `Layout` component wraps page content with header, sidebar, and footer, rendering children in the main area.
- **Modal dialogs:** A `Modal` component renders its `children` inside a backdrop and container.
- **Cards and panels:** A `Card` component wraps arbitrary content with consistent styling.
- **Recursive trees:** A `TreeNode` component renders its children recursively for file explorers or comment threads.
- **Accordions:** An `Accordion` component renders multiple `AccordionItem` children.

### References

- React Official Documentation – Understanding Your UI as a Tree: https://react.dev/learn/understanding-your-ui-as-a-tree
- React Official Documentation – Your First Component: https://react.dev/learn/your-first-component
- React Official Documentation – Passing JSX as Children: https://react.dev/learn/passing-props-to-a-component#passing-jsx-as-children

---

## Core Concept 3: Component Slots Using `children`

### Definitions

**Core Definition:** The `children` prop is a special React prop that receives whatever JSX content is nested between a component's opening and closing tags, enabling components to act as generic containers or "slots" for arbitrary content.

**Technical Definition:** Every React component receives a `children` prop automatically when JSX is nested inside its tags. Anything inside the `<FancyBorder>` JSX tag gets passed into the `FancyBorder` component as a `children` prop. Since `FancyBorder` renders `{props.children}` inside a `<div>`, the passed elements appear in the final output. The `children` prop can contain any valid React node: elements, strings, numbers, fragments, arrays, or even functions (render props). For components that need multiple content areas — often called "slots" or "holes" — React does not have a built-in slot syntax like Vue. Instead, developers use named props (e.g., `header`, `footer`, `left`, `right`) to pass multiple independent JSX pieces. React elements are just objects, so they can be passed as props like any other data. This pattern is commonly used in layout components to provide header, sidebar, and footer areas.

**Beginner-Friendly Explanation:** The `children` prop is like an empty box inside a component. When you use the component, you can put anything you want inside the box by nesting it between the component's opening and closing tags. For example, a `Card` component might have an empty box in the middle; you can put text, images, or buttons in that box. If you need multiple boxes (e.g., a header box and a footer box), you use named props — like giving each box a label.

### Purposes

- To allow components to wrap arbitrary content without knowing what that content is.
- To create generic container components (cards, modals, layouts) that accept any children.
- To reduce the number of props a component needs by accepting JSX directly.
- To enable the "containment" composition pattern described in React's official documentation.
- To provide multiple content areas ("slots") via named props for components with distinct sections.

### Syntax Rules and Structure

**Single Slot (`children`):**
```jsx
function FancyBorder(props) {
    return (
        <div className={'FancyBorder FancyBorder-' + props.color}>
            {props.children}
        </div>
    );
}

function WelcomeDialog() {
    return (
        <FancyBorder color="blue">
            <h1 className="Dialog-title">Welcome</h1>
            <p className="Dialog-message">Thank you for visiting!</p>
        </FancyBorder>
    );
}
```

**Component Breakdown:**
- `{props.children}`: Renders whatever JSX was nested inside `<FancyBorder>`.
- `<FancyBorder color="blue">...</FancyBorder>`: The nested content is passed as `children`.

**Multiple Slots (Named Props):**
```jsx
function SplitPane(props) {
    return (
        <div className="SplitPane">
            <div className="SplitPane-left">{props.left}</div>
            <div className="SplitPane-right">{props.right}</div>
        </div>
    );
}

function App() {
    return <SplitPane left={<Contacts />} right={<Chat />}/>;
}
```

**Component Breakdown:**
- `props.left` and `props.right`: Named slots for different content areas.
- `<Contacts />` and `<Chat />`: Passed as prop values (React elements are just objects).

**Syntax Rules:**
- Use `{props.children}` (or destructured `{ children }`) to render nested content.
- For multiple slots, use descriptive prop names (`header`, `footer`, `left`, `right`, `actions`).
- React elements can be passed as props: `left={<Contacts />}`.
- The `children` prop can contain any valid React node, including fragments and arrays.
- Use the `Children` utilities (`Children.map`, `Children.count`, etc.) only when you need to manipulate children programmatically; they are uncommon and can lead to fragile code.

**Constraints and Limitations:**
- A component has only one `children` prop; multiple slots require named props.
- `children` is opaque — the parent component does not know what the children are unless it inspects them.
- Manipulating `children` with `React.Children` utilities is uncommon and can break with fragments and arrays.
- Passing large JSX trees as props can make component APIs verbose.

### Annotated Code Example: Modal with Multiple Slots

```jsx
// Modal component with named slots for header, body, and footer
function Modal({ isOpen, onClose, header, body, footer }) {
    if (!isOpen) return null;
  
    return (
        <div className="modal-backdrop" onClick={onClose}>
            <div className="modal-container" onClick={e => e.stopPropagation()}>
                {/* Header slot */}
                <div className="modal-header">{header}</div>
        
                {/* Body slot */}
                <div className="modal-body">{body}</div>
        
                {/* Footer slot */}
                <div className="modal-footer">{footer}</div>
            </div>
        </div>
    );
}

// Usage: each slot receives independent JSX
export default function App() {
    const [isOpen, setIsOpen] = React.useState(false);
  
    return (
        <div>
            <button onClick={() => setIsOpen(true)}>Open Modal</button>
        
            <Modal
                isOpen={isOpen}
                onClose={() => setIsOpen(false)}
                header={<h2>Confirm Delete</h2>}
                body={<p>Are you sure you want to delete this item?</p>}
                footer={
                    <>
                        <button onClick={() => setIsOpen(false)}>Cancel</button>
                        <button onClick={() => setIsOpen(false)}>Delete</button>
                    </>
                }
            />
        </div>
    );
}
```

**Expected Output:** Clicking "Open Modal" displays a modal with a header ("Confirm Delete"), a body ("Are you sure you want to delete this item?"), and a footer with "Cancel" and "Delete" buttons. Clicking the backdrop or "Cancel" closes the modal.

**Why This Output Occurs:** The `Modal` component accepts three named slot props: `header`, `body`, and `footer`. Each slot receives independent JSX from the parent. The `Modal` renders each slot in its corresponding section. This approach keeps the `Modal` component generic while allowing the parent to control the content of each section.

### Real-World Cases

- **Modal dialogs:** Header, body, and footer slots for title, content, and actions.
- **Layout components:** Header, sidebar, main, and footer slots for page structure.
- **Split panes:** Left and right slots for side-by-side content.
- **Cards:** Header, media, body, and actions slots for structured content.
- **Data tables:** Header, body, and empty-state slots for table customisation.

### References

- React Official Documentation – Composition vs Inheritance: https://legacy.reactjs.org/docs/composition-vs-inheritance.html
- React Official Documentation – Passing JSX as Children: https://react.dev/learn/passing-props-to-a-component#passing-jsx-as-children
- React Official Documentation – Children API: https://react.dev/reference/react/Children
- GitHub – Slot Pattern for Flexible Content Areas: https://github.com/pproenca/dot-skills/blob/master/skills/.curated/shadcn/references/comp-use-slot-pattern-for-flexibility.md

---

## Core Concept 4: Reusable Layout Components

### Definitions

**Core Definition:** Reusable layout components are components that encapsulate common page or section structure (header, sidebar, main content, footer) and accept arbitrary content through slots or `children`, enabling consistent layouts across multiple pages without duplication.

**Technical Definition:** Reusable layout components leverage composition to provide structural consistency. They typically render a fixed skeleton (e.g., a `<header>`, `<aside>`, and `<main>`) and inject variable content through `children` or named slot props. Because React elements are plain objects that can be passed as props, layout components can accept multiple independent content regions and render them in designated positions. This pattern is closely related to the "slot pattern," which recommends using named slot patterns instead of render props or excessive boolean props for components with multiple content areas (header, footer, actions). Reusable layout components reduce duplication, enforce design consistency, and make it easy to change the overall page structure in one place.

**Beginner-Friendly Explanation:** A reusable layout component is like a picture frame. The frame (header, sidebar, footer) stays the same, but the picture inside (the page content) changes. You can use the same frame for many different pictures. If you want to change the frame's colour or size, you change it in one place, and all pages that use it are updated automatically.

### Purposes

- To provide consistent page structure across multiple routes or views.
- To reduce duplication of layout markup (headers, sidebars, footers).
- To make it easy to change the overall layout in one place.
- To allow pages to focus on their unique content while inheriting shared chrome.
- To support nested layouts where layouts themselves are composed.

### Syntax Rules and Structure

**Basic Layout with `children`:**
```jsx
function PageLayout({ children }) {
  return (
    <div className="page">
      <header>
        <h1>My App</h1>
        <nav>...</nav>
      </header>
      <main>{children}</main>
      <footer>© 2024</footer>
    </div>
  );
}

function HomePage() {
  return (
    <PageLayout>
      <h2>Welcome</h2>
      <p>This is the home page.</p>
    </PageLayout>
  );
}
```

**Layout with Named Slots:**
```jsx
function DashboardLayout({ sidebar, content, header }) {
  return (
    <div className="dashboard">
      <header>{header}</header>
      <div className="dashboard-body">
        <aside>{sidebar}</aside>
        <main>{content}</main>
      </div>
    </div>
  );
}

function DashboardPage() {
  return (
    <DashboardLayout
      header={<h1>Dashboard</h1>}
      sidebar={<nav>Dashboard Navigation</nav>}
      content={<p>Dashboard content goes here.</p>}
    />
  );
}
```

**Nested Layouts:**
```jsx
function AppLayout({ children }) {
  return (
    <div>
      <header>App Header</header>
      {children}
    </div>
  );
}

function DashboardLayout({ children }) {
  return (
    <div>
      <aside>Dashboard Sidebar</aside>
      <main>{children}</main>
    </div>
  );
}

// Usage
<AppLayout>
  <DashboardLayout>
    <p>Dashboard content</p>
  </DashboardLayout>
</AppLayout>
```

**Syntax Rules:**
- Use `children` for a single content area; use named props for multiple areas.
- Keep layout components presentational; avoid adding business logic.
- Layout components should render semantic HTML landmarks (`<header>`, `<nav>`, `<main>`, `<aside>`, `<footer>`).
- Nest layout components for hierarchical page structures.
- Use consistent prop names across layout components (`header`, `sidebar`, `content`, `footer`).

**Constraints and Limitations:**
- Layout components with too many slots become complex; extract sub-layouts if needed.
- Passing large JSX trees as slot props can be verbose.
- Layout components do not manage state; state management belongs to the pages or features.
- Deeply nested layouts can make the component tree hard to trace.

### Annotated Code Example: App Layout with Header, Sidebar, and Content

```jsx
// Reusable layout component with named slots
function AppLayout({ header, sidebar, children }) {
  return (
    <div className="app-layout">
      {/* Header slot */}
      <header className="app-header">{header}</header>

      <div className="app-body">
        {/* Sidebar slot */}
        <aside className="app-sidebar">{sidebar}</aside>

        {/* Main content via children */}
        <main className="app-content">{children}</main>
      </div>
    </div>
  );
}

// Page using the layout
export default function DashboardPage() {
  return (
    <AppLayout
      header={
        <div>
          <h1>Dashboard</h1>
          <button>Logout</button>
        </div>
      }
      sidebar={
        <nav>
          <a href="/overview">Overview</a>
          <a href="/settings">Settings</a>
        </nav>
      }
    >
      <h2>Welcome back!</h2>
      <p>Here is your dashboard content.</p>
    </AppLayout>
  );
}
```

**Expected Output:** A page with a header containing "Dashboard" and a "Logout" button, a sidebar with "Overview" and "Settings" links, and a main content area with "Welcome back!" and a paragraph.

**Why This Output Occurs:** The `AppLayout` component provides the structural skeleton (header, sidebar, main) and accepts content for each region via named props (`header`, `sidebar`) and `children` (main content). The `DashboardPage` supplies the specific content for each slot. The layout can be reused by other pages with different content.

### Real-World Cases

- **Admin dashboards:** Header, sidebar, and content layout reused across all admin pages.
- **Marketing sites:** Header, hero, content, and footer layout reused across landing pages.
- **Documentation sites:** Sidebar navigation, content area, and table-of-contents layout.
- **E-commerce:** Header, breadcrumb, product grid, and footer layout.
- **Multi-tenant apps:** Tenant-specific layout with shared structure.

### References

- React Official Documentation – Composition vs Inheritance: https://legacy.reactjs.org/docs/composition-vs-inheritance.html
- GitHub – Slot Pattern for Flexible Content Areas: https://github.com/pproenca/dot-skills/blob/master/skills/.curated/shadcn/references/comp-use-slot-pattern-for-flexibility.md
- React Official Documentation – Understanding Your UI as a Tree: https://react.dev/learn/understanding-your-ui-as-a-tree

---

## Core Concept 5: Composition versus Inheritance

### Definitions

**Core Definition:** Composition versus inheritance is the architectural choice between building components by combining smaller components (composition) and building components by extending base classes (inheritance); React strongly recommends composition.

**Technical Definition:** React's official documentation states that "React has a powerful composition model, and we recommend using composition instead of inheritance to reuse code between components". Inheritance follows the IS-A principle — a child class is a parent class — while composition follows the HAS-A principle — a class has certain attributes or functionality. In React, inheritance-based reuse would involve extending a base component class and overriding methods, whereas composition-based reuse involves rendering one component inside another and configuring it with props. The official documentation notes that at Facebook (now Meta), where React is used in thousands of components, they have not found any use cases where inheritance is recommended. Props and composition give you all the flexibility you need to customize a component's look and behavior in an explicit and safe way. The "specialization" pattern — where a more specific component renders a more generic one and configures it with props — is the composition-based alternative to inheritance.

**Beginner-Friendly Explanation:** Inheritance is like saying "a sports car is a car" — the sports car inherits all the car's features and adds its own. Composition is like saying "a car has an engine, wheels, and seats" — the car is built by combining these parts. React prefers composition because it's more flexible: you can swap out an engine without redesigning the whole car. Inheritance creates tight coupling (the sports car is stuck with the car's design), while composition keeps parts independent and swappable.

### Purposes

- To reuse code between components without the coupling of class hierarchies.
- To customize a component's look and behaviour explicitly through props.
- To avoid the fragility of inheritance (deep hierarchies, override confusion).
- To enable the "specialization" pattern (specific components configuring generic ones).
- To align with React's design philosophy, which favours props and composition over inheritance.

### Syntax Rules and Structure

**Inheritance (Anti-Pattern in React):**
```jsx
// ❌ Inheritance-based reuse (not recommended)
class BaseButton extends React.Component {
  render() {
    return <button className="base">{this.props.label}</button>;
  }
}

class PrimaryButton extends BaseButton {
  render() {
    // Overrides parent render — fragile and coupled
    return <button className="primary">{this.props.label}</button>;
  }
}
```

**Composition (Recommended):**
```jsx
// ✅ Composition-based reuse
function Button({ variant = "base", label, children }) {
  return (
    <button className={`btn btn-${variant}`}>
      {label || children}
    </button>
  );
}

function PrimaryButton({ label }) {
  return <Button variant="primary" label={label} />;
}
```

**Component Breakdown:**
- `Button`: A generic component that accepts a `variant` prop.
- `PrimaryButton`: A specialized component that renders `Button` with a specific variant.
- No inheritance; `PrimaryButton` "has a" `Button`, it "is not a" `Button`.

**Specialization Pattern:**
```jsx
// Generic Dialog component
function Dialog({ title, message, children }) {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title">{title}</h1>
      <p className="Dialog-message">{message}</p>
      {children}
    </FancyBorder>
  );
}

// Specialized WelcomeDialog
function WelcomeDialog() {
  return (
    <Dialog
      title="Welcome"
      message="Thank you for visiting our spacecraft!"
    />
  );
}
```

**Syntax Rules:**
- Prefer composition over inheritance for all component reuse.
- Use "specialization" — a specific component renders a generic one and configures it with props.
- Use props to customize component behaviour; do not override methods via inheritance.
- When you need to share logic, use custom Hooks or composition, not class inheritance.
- React components should be pure functions of props; inheritance breaks this model.

**Constraints and Limitations:**
- Composition may require more explicit prop passing than inheritance's implicit method sharing.
- Deeply nested composition can create prop drilling (solved by context or slots).
- Some developers with OOP backgrounds may instinctively reach for inheritance; React's model requires a shift in thinking.
- Composition does not provide the IS-A polymorphism of inheritance, but React's component model does not need it.

### Annotated Code Example: Specialization via Composition

```jsx
// Generic Dialog component
function Dialog({ title, message, children }) {
  return (
    <div className="dialog">
      <h1 className="dialog-title">{title}</h1>
      <p className="dialog-message">{message}</p>
      <div className="dialog-content">{children}</div>
    </div>
  );
}

// Specialized component: WelcomeDialog
function WelcomeDialog() {
  return (
    <Dialog
      title="Welcome"
      message="Thank you for visiting our spacecraft!"
    >
      <button>Get Started</button>
    </Dialog>
  );
}

// Specialized component: ErrorDialog
function ErrorDialog({ errorMessage }) {
  return (
    <Dialog
      title="Error"
      message={errorMessage}
    >
      <button>Retry</button>
    </Dialog>
  );
}

export default function App() {
  return (
    <div>
      <WelcomeDialog />
      <ErrorDialog errorMessage="Something went wrong." />
    </div>
  );
}
```

**Expected Output:** Two dialogs are rendered. The first has the title "Welcome" and a "Get Started" button. The second has the title "Error" and a "Retry" button. Both share the same `Dialog` structure but are configured differently.

**Why This Output Occurs:** `WelcomeDialog` and `ErrorDialog` are specialized components that render the generic `Dialog` component and configure it with different props. This is composition-based reuse: the specific components "have a" `Dialog`, they do not "inherit from" it. The `Dialog` component does not know or care which specialized component is using it.

### Real-World Cases

- **Design systems:** A generic `Button` component configured by `PrimaryButton`, `SecondaryButton`, and `DangerButton`.
- **Dialogs:** A generic `Dialog` configured by `ConfirmDialog`, `AlertDialog`, and `FormDialog`.
- **Cards:** A generic `Card` configured by `ProductCard`, `UserCard`, and `ArticleCard`.
- **Layouts:** A generic `Layout` configured by `DashboardLayout`, `AuthLayout`, and `MarketingLayout`.

### References

- React Official Documentation – Composition vs Inheritance: https://legacy.reactjs.org/docs/composition-vs-inheritance.html
- React Official Documentation – Composition vs Inheritance (GitHub Source): https://raw.githubusercontent.com/reactjs/react.dev/2739197161cd60b8c211039a3cadd3e2a8ac2422/content/docs/composition-vs-inheritance.md
- Pluralsight – React.js and Inheritance: https://www.pluralsight.com/guides/react-js-and-inheritance

---

## Core Concept 6: Compound Components

### Definitions

**Core Definition:** Compound components are a pattern where a parent component manages shared state and behaviour, while child subcomponents control structure and rendering, working together as a cohesive unit — similar to how `<select>` and `<option>` work in HTML.

**Technical Definition:** Compound components are a set of components that work together to form a complete UI, implicitly sharing state and behaviour. The parent component manages shared state and logic for its child components, while each child component handles its own rendering logic. State is typically shared through React Context, allowing child components to access shared state without prop drilling. You can think of a compound component as the HTML `<select>` and `<option>` components: by itself, `<select>` is useless; by itself, `<option>` is useless; but together, they are very useful, and they share implicit state between each other. The pattern turns rigid, prop-heavy widgets into composable, scalable building blocks. It solves problems like prop soup (dozens of props), mixed responsibilities (one component doing too much), and poor reusability.

**Beginner-Friendly Explanation:** Compound components are like a television and a remote control. The TV (parent) holds all the state (channel, volume) and does the hard work. The remote (child) provides the interface for changing the state. You can't use the remote without the TV, and the TV is hard to use without the remote. But together, they form a complete system. In React, a `Tabs` component might have `Tabs.List`, `Tabs.Tab`, and `Tabs.Panel` subcomponents — each handles its own rendering, but they all share the selected tab state from the parent.

### Purposes

- To avoid prop explosion — replacing dozens of props with semantic subcomponents.
- To separate responsibilities — the parent manages state and behaviour; children manage structure and rendering.
- To improve reusability — build one compound component that scales to many cases without forking.
- To improve readability and testing — smaller, focused pieces are easier to read and test.
- To share implicit state between subcomponents without prop drilling.
- To create flexible, composable APIs for complex UI widgets (tabs, accordions, modals, menus).

### Syntax Rules and Structure

**Basic Compound Component with Context:**
```jsx
import { createContext, useContext, useState } from "react";

// 1. Create context for shared state
const TabsContext = createContext(null);

// 2. Parent component: manages state
function Tabs({ children, defaultTab }) {
  const [activeTab, setActiveTab] = useState(defaultTab);

  return (
    <TabsContext.Provider value={{ activeTab, setActiveTab }}>
      <div className="tabs">{children}</div>
    </TabsContext.Provider>
  );
}

// 3. Child components: consume context
function TabList({ children }) {
  return <div className="tab-list">{children}</div>;
}

function Tab({ id, children }) {
  const { activeTab, setActiveTab } = useContext(TabsContext);
  return (
    <button
      className={activeTab === id ? "active" : ""}
      onClick={() => setActiveTab(id)}
    >
      {children}
    </button>
  );
}

function TabPanel({ id, children }) {
  const { activeTab } = useContext(TabsContext);
  if (activeTab !== id) return null;
  return <div className="tab-panel">{children}</div>;
}

// 4. Attach subcomponents as static properties
Tabs.List = TabList;
Tabs.Tab = Tab;
Tabs.Panel = TabPanel;
```

**Usage:**
```jsx
<Tabs defaultTab="overview">
  <Tabs.List>
    <Tabs.Tab id="overview">Overview</Tabs.Tab>
    <Tabs.Tab id="details">Details</Tabs.Tab>
  </Tabs.List>
  <Tabs.Panel id="overview">Overview content</Tabs.Panel>
  <Tabs.Panel id="details">Details content</Tabs.Panel>
</Tabs>
```

**Component Breakdown:**
- `TabsContext`: Shares `activeTab` and `setActiveTab` between all subcomponents.
- `Tabs`: Parent component that owns the state and provides it via context.
- `TabList`, `Tab`, `TabPanel`: Child components that consume the context.
- `Tabs.List = TabList`: Static properties allow `<Tabs.List>` syntax.

**Syntax Rules:**
- The parent component owns the shared state; children consume it via context.
- Attach subcomponents as static properties on the parent (`Tabs.List = TabList`).
- Use `useContext` in children to access shared state and setters.
- Throw a clear error if a child is used outside its parent context.
- The parent renders `children` inside a context provider.

**Constraints and Limitations:**
- Compound components require context, which adds a small amount of overhead.
- Children must be direct descendants of the parent provider; deep nesting still works with context, but the provider must be above all consumers.
- TypeScript typing of compound components can be complex; use `Object.assign` or static property declarations.
- The pattern is more complex than a single component with props; use it when prop explosion becomes a problem.

### Annotated Code Example: Accordion Compound Component

```jsx
import { createContext, useContext, useState } from "react";

// Context for shared accordion state
const AccordionContext = createContext(null);

// Parent: manages open/close state
function Accordion({ children, defaultOpen }) {
  const [openItem, setOpenItem] = useState(defaultOpen);

  return (
    <AccordionContext.Provider value={{ openItem, setOpenItem }}>
      <div className="accordion">{children}</div>
    </AccordionContext.Provider>
  );
}

// Child: Accordion.Item
function AccordionItem({ id, title, children }) {
  const { openItem, setOpenItem } = useContext(AccordionContext);
  const isOpen = openItem === id;

  return (
    <div className="accordion-item">
      <button
        className="accordion-header"
        onClick={() => setOpenItem(isOpen ? null : id)}
      >
        {title} {isOpen ? "▲" : "▼"}
      </button>
      {isOpen && <div className="accordion-body">{children}</div>}
    </div>
  );
}

// Attach subcomponent
Accordion.Item = AccordionItem;

// Usage
export default function App() {
  return (
    <Accordion defaultOpen="item1">
      <Accordion.Item id="item1" title="Section 1">
        <p>Content for section 1.</p>
      </Accordion.Item>
      <Accordion.Item id="item2" title="Section 2">
        <p>Content for section 2.</p>
      </Accordion.Item>
    </Accordion>
  );
}
```

**Expected Output:** An accordion with two sections. Section 1 is open by default (showing its content). Clicking "Section 2" collapses Section 1 and expands Section 2.

**Why This Output Occurs:** The `Accordion` parent owns the `openItem` state and provides it via context. Each `AccordionItem` consumes the context to check if it is the open item and to request a change. The parent controls which item is open, and the children render accordingly. This is the compound component pattern: shared state in the parent, independent rendering in the children.

### Real-World Cases

- **Tabs:** `Tabs`, `Tabs.List`, `Tabs.Tab`, `Tabs.Panel`.
- **Accordions:** `Accordion`, `Accordion.Item`.
- **Modals:** `Modal`, `Modal.Header`, `Modal.Body`, `Modal.Footer`.
- **Dropdown menus:** `Menu`, `Menu.Button`, `Menu.Items`, `Menu.Item`.
- **Select components:** `Select`, `Select.Trigger`, `Select.Options`, `Select.Option`.
- **Design systems:** Radix Primitives, Reach UI, Headless UI all use compound components.

### References

- Epic React – Compound Components Intro: https://www.epicreact.dev/modules/advanced-react-patterns-v1/compound-components-patterns-intro
- TechGig – How to Use the Compound Components Pattern in React: https://content.techgig.com/career-advice/master-the-compound-components-pattern-in-react-for-flexible-uis/amp_articleshow/124470802.cms
- GitHub – Compound Components Skill Reference: https://github.com/ahmad-ubaidillah/aizen-gate/blob/main/aizen-gate/skills-reference/skills/agent-frontend-patterns/sub-skills/compound-components.md
- Steve Kinney – Typing Compound Components and Slots: https://stevekinney.com
- FreeCodeCamp – How to Use the Compound Components Pattern in React: https://www.freecodecamp.org/news/how-to-use-the-compound-components-pattern-in-react/

---

## Comparison and Decision Guidance

| Pattern | Best For | State Sharing | API Complexity | Reusability |
|---|---|---|---|---|
| **Parent-Child** | Basic data flow | Props + callbacks | Low | High |
| **Nested Components** | Hierarchical UIs | Props + children | Low | High |
| **Children Prop** | Generic containers | None (content only) | Low | Very High |
| **Named Slots** | Multi-section layouts | None (content only) | Medium | Very High |
| **Composition vs Inheritance** | Code reuse | Props | Low | High |
| **Compound Components** | Complex widgets | Context | High | Very High |

**Decision Guidance:**
- **Use parent-child relationships** for all basic data flow; this is the default React pattern.
- **Use nested components** when building hierarchical UIs (layouts, trees, recursive structures).
- **Use the `children` prop** for generic containers (cards, modals, layouts) that accept arbitrary content.
- **Use named slots** when a component has multiple distinct content areas (header, body, footer).
- **Prefer composition over inheritance** for all code reuse; use specialization (specific components configuring generic ones).
- **Use compound components** when a widget has multiple coordinated parts that share implicit state (tabs, accordions, menus) and you want to avoid prop explosion.
- **Never define components inside other components** (definition nesting); always define at the top level and nest instances in JSX.

---

## References

- React Official Documentation – Composition vs Inheritance: https://legacy.reactjs.org/docs/composition-vs-inheritance.html
- React Official Documentation – Composition vs Inheritance (GitHub Source): https://raw.githubusercontent.com/reactjs/react.dev/2739197161cd60b8c211039a3cadd3e2a8ac2422/content/docs/composition-vs-inheritance.md
- React Official Documentation – Passing Props to a Component: https://react.dev/learn/passing-props-to-a-component
- React Official Documentation – Passing JSX as Children: https://react.dev/learn/passing-props-to-a-component#passing-jsx-as-children
- React Official Documentation – Understanding Your UI as a Tree: https://react.dev/learn/understanding-your-ui-as-a-tree
- React Official Documentation – Sharing State Between Components: https://react.dev/learn/sharing-state-between-components
- React Official Documentation – Your First Component: https://react.dev/learn/your-first-component
- React Official Documentation – Children API: https://react.dev/reference/react/Children
- Epic React – Compound Components Intro: https://www.epicreact.dev/modules/advanced-react-patterns-v1/compound-components-patterns-intro
- TechGig – How to Use the Compound Components Pattern in React: https://content.techgig.com/career-advice/master-the-compound-components-pattern-in-react-for-flexible-uis/amp_articleshow/124470802.cms
- GitHub – Slot Pattern for Flexible Content Areas: https://github.com/pproenca/dot-skills/blob/master/skills/.curated/shadcn/references/comp-use-slot-pattern-for-flexibility.md
- GitHub – Compound Components Skill Reference: https://github.com/ahmad-ubaidillah/aizen-gate/blob/main/aizen-gate/skills-reference/skills/agent-frontend-patterns/sub-skills/compound-components.md
- Steve Kinney – Typing Compound Components and Slots: https://stevekinney.com
- Pluralsight – React.js and Inheritance: https://www.pluralsight.com/guides/react-js-and-inheritance
- FreeCodeCamp – How to Use the Compound Components Pattern in React: https://www.freecodecamp.org/news/how-to-use-the-compound-components-pattern-in-react/