# Typing React Components: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Typing React components is the practice of annotating React function components, their props, state, events, refs, context, and hooks with TypeScript types so that the compiler enforces correct usage across the component tree.

**Technical Definition:** Typing React components involves applying TypeScript's structural type system to React's component model. It covers: (1) **props** — typing the shape of data passed from parent to child, including the choice between `React.FC` and direct destructured prop typing; (2) **children** — choosing between `React.ReactNode`, `React.ReactElement`, and `JSX.Element` based on what the component accepts; (3) **events** — using the correct `React.SyntheticEvent` subtype (`React.ChangeEvent<HTMLInputElement>`, `React.FormEvent<HTMLFormElement>`, `React.MouseEvent<HTMLButtonElement>`) for each handler; (4) **state** — relying on inference for simple values and explicit generics for unions, null, and complex objects; (5) **refs** — using `React.RefObject<T>` for DOM elements and `React.MutableRefObject<T>` for persistent mutable values; (6) **context** — handling `null` initial values, defining strict providers, and exporting custom consumption hooks; and (7) **advanced hooks** — typing `useReducer` with discriminated action unions, `useMemo`, `useCallback`, and concurrent features like `useTransition`. TypeScript's types are erased at runtime, so runtime validation remains the developer's responsibility.

**Beginner-Friendly Explanation:** When you write React components in TypeScript, you tell the compiler what each prop, state value, event, and ref should look like. This catches mistakes before you run the code—passing a number where a string is expected, forgetting a required prop, or using a ref that might be null. This cheat sheet covers the seven areas where React and TypeScript intersect and shows the idiomatic way to type each one.

### Key Characteristics

- **Structural Typing:** Types are compared by shape, not by name; any object with the right properties is compatible.
- **Inference Over Annotation:** TypeScript infers many types automatically; explicit annotations are needed only when inference is insufficient.
- **`React.FC` is Optional:** Modern React avoids `React.FC` because it implicitly types `children` and hides the return type.
- **Children Have Three Common Types:** `React.ReactNode` (anything renderable), `React.ReactElement` (a single element), `JSX.Element` (the return type of JSX).
- **Events Have Specific Subtypes:** Each event type (`ChangeEvent`, `FormEvent`, `MouseEvent`, `KeyboardEvent`, `FocusEvent`) corresponds to a specific DOM event and element.
- **Refs Come in Two Flavours:** `RefObject<T>` (read-only, for DOM elements) and `MutableRefObject<T>` (mutable, for persistent values).
- **Context Requires Null Handling:** `createContext<T | null>(null)` forces consumers to handle `null` or use a custom hook that throws.
- **Reducers Benefit from Discriminated Unions:** Action types as a discriminated union enable exhaustive `switch` handling and full type safety.

### Prerequisites

- Solid understanding of React function components, JSX, props, and Hooks.
- Working knowledge of TypeScript basics (types, interfaces, unions, generics).
- Familiarity with the DOM event model and HTML elements.
- Basic understanding of React Context and `useReducer`.
- Awareness of `tsconfig.json` strictness flags (`strict`, `strictNullChecks`).

### Related Programming Areas

- **TypeScript Fundamentals:** Primitives, unions, generics, narrowing, assertions.
- **React Component Architecture:** Composition, compound components, and slots.
- **State Management:** `useState`, `useReducer`, Context, and external stores.
- **Accessibility:** Typing ARIA attributes and event handlers.
- **Testing:** Typed test utilities and mocking.

### Core Concepts / Features

1. Props (`React.FC` vs. Direct Destructured Props)
2. Children (`React.ReactNode`, `React.ReactElement`, `JSX.Element`)
3. Events (`React.ChangeEvent`, `React.FormEvent`, `React.MouseEvent`)
4. Component State (Inference vs. Explicit Generics)
5. Refs (`RefObject` vs. `MutableRefObject`)
6. Context (Null Handling, Strict Providers, Custom Hooks)
7. Advanced Hooks (`useReducer`, `useMemo`, `useCallback`, `useTransition`)

---

## Core Concept 1: Props — `React.FC` vs. Direct Destructured Props

### Definitions

**Core Definition:** Typing props is the practice of declaring the shape of the data a component receives, either by annotating the props parameter directly or by using the `React.FC` type alias.

**Technical Definition:** In modern React with TypeScript, props are typed by defining a `Props` type (interface or type alias) and annotating the component's parameter: `function Button({ label, onClick }: ButtonProps)`. The `React.FC<Props>` (or `React.FunctionComponent<Props>`) type alias was historically used to type components but has fallen out of favour because it (1) implicitly includes `children?: React.ReactNode`, which is often undesired; (2) hides the component's return type, requiring explicit annotation for consistency; (3) does not support generic components cleanly; and (4) prevents TypeScript from inferring the component's return type. The React TypeScript Cheatsheet explicitly recommends against `React.FC` for these reasons. Direct typing with a `Props` type is more explicit, more flexible, and works with generics.

**Beginner-Friendly Explanation:** There are two ways to tell TypeScript what props a component takes. The old way is `React.FC<Props>`, which says "this is a function component with these props." The new way is to annotate the parameter directly: `function Button({ label }: Props)`. The direct way is better because it does not secretly add `children` to your props and works better with TypeScript's inference. Most modern codebases use the direct approach.

### Purposes

- To declare the shape of data a component accepts.
- To catch missing required props at compile time.
- To enable editor autocomplete for prop names and values.
- To document the component's public API.
- To enforce default values for optional props.
- To work with generics for reusable components.

### Syntax Rules and Structure

**Direct Destructured Props (Recommended):**
```tsx
type ButtonProps = {
  label: string;
  variant?: 'primary' | 'secondary';
  disabled?: boolean;
  onClick: () => void;
};

function Button({ label, variant = 'primary', disabled = false, onClick }: ButtonProps) {
  return (
    <button className={`btn-${variant}`} disabled={disabled} onClick={onClick}>
      {label}
    </button>
  );
}
```

**Component Breakdown:**
- `type ButtonProps`: Declares the shape of the props.
- `{ label, variant = 'primary', disabled = false, onClick }`: Destructures the props with defaults.
- `: ButtonProps`: Annotates the parameter with the props type.

**`React.FC` (Not Recommended):**
```tsx
const Button: React.FC<ButtonProps> = ({ label, variant = 'primary', onClick }) => {
  return <button className={`btn-${variant}`} onClick={onClick}>{label}</button>;
};
```

**Component Breakdown:**
- `React.FC<ButtonProps>`: Types the component as a function component.
- Implicitly includes `children?: React.ReactNode` (undesired in most cases).
- Hides the return type; TypeScript cannot infer it.
- Does not work well with generics.

**Props with `children` (Explicit):**
```tsx
type CardProps = {
  title: string;
  children: React.ReactNode;
};

function Card({ title, children }: CardProps) {
  return (
    <div className="card">
      <h2>{title}</h2>
      <div className="card-body">{children}</div>
    </div>
  );
}
```

**Component Breakdown:**
- `children: React.ReactNode`: Explicitly types the children prop (see Concept 2).
- Without `React.FC`, children are opt-in; the component only accepts children if declared.

**Extending HTML Element Props:**
```tsx
type ButtonProps = React.ComponentPropsWithoutRef<'button'> & {
  variant?: 'primary' | 'secondary';
};

function Button({ variant = 'primary', ...rest }: ButtonProps) {
  return <button className={`btn-${variant}`} {...rest} />;
}

// Usage: all native button props are accepted
<Button variant="primary" type="submit" disabled>Submit</Button>
```

**Component Breakdown:**
- `React.ComponentPropsWithoutRef<'button'>`: Extracts all props of `<button>` (excluding `ref`).
- `& { variant?: ... }`: Intersects with custom props.
- `...rest`: Forwards native props to the underlying `<button>`.

**Generic Component Props:**
```tsx
type ListProps<T> = {
  items: T[];
  renderItem: (item: T) => React.ReactNode;
};

function List<T>({ items, renderItem }: ListProps<T>) {
  return <ul>{items.map((item, i) => <li key={i}>{renderItem(item)}</li>)}</ul>;
}
```

**Component Breakdown:**
- `<T>`: Declares a generic type parameter.
- `ListProps<T>`: Uses the generic in the props type.
- TypeScript infers `T` from the `items` prop.

**Syntax Rules:**
- Prefer direct destructured props over `React.FC`.
- Define a `Props` type (interface or type alias) for every component.
- Mark optional props with `?`; provide defaults in the destructuring.
- Extend native element props with `React.ComponentPropsWithoutRef<'element'>` when wrapping DOM elements.
- Use `React.ComponentPropsWithRef<'element'>` when forwarding refs.
- Use generics for reusable components that work with any data type.
- Avoid `React.FC` unless maintaining legacy code.

**Constraints and Limitations:**
- `React.FC` implicitly adds `children`, which can hide missing children props.
- Direct typing requires explicit `children` when needed.
- `ComponentPropsWithoutRef` does not include `ref`; use `ComponentPropsWithRef` for ref-forwarding components.
- Excess property checks apply to object literals; passing a variable with extra properties is allowed.
- Generic components with `React.memo` require workarounds to preserve generics.

### Annotated Code Examples

**Example 1: Simple Button with Direct Props**

```tsx
type ButtonProps = {
  variant?: 'primary' | 'secondary' | 'danger';
  size?: 'sm' | 'md' | 'lg';
  disabled?: boolean;
  children: React.ReactNode;
  onClick?: () => void;
};

export function Button({
  variant = 'primary',
  size = 'md',
  disabled = false,
  children,
  onClick,
}: ButtonProps) {
  return (
    <button className={`btn btn-${variant} btn-${size}`} disabled={disabled} onClick={onClick}>
      {children}
    </button>
  );
}

// Usage
<Button variant="danger" size="lg" onClick={() => alert('Deleted')}>
  Delete
</Button>
```

**Expected Output:** A styled danger button with "Delete" text. TypeScript autocompletes `variant`, `size`, and `disabled`; passing an invalid variant (e.g., `"purple"`) errors at compile time.

**Why This Output Occurs:** The `ButtonProps` type declares the shape. Defaults are provided in destructuring. `children` is required. TypeScript checks every prop against the type.

**Example 2: Extending Native Button Props**

```tsx
type IconButtonProps = React.ComponentPropsWithoutRef<'button'> & {
  icon: React.ReactNode;
  'aria-label': string; // Required for icon-only buttons
};

export function IconButton({ icon, ...rest }: IconButtonProps) {
  return (
    <button {...rest} className="icon-button">
      {icon}
    </button>
  );
}

// Usage
<IconButton
  icon={<CloseIcon />}
  aria-label="Close dialog"
  onClick={close}
  type="button"
/>
```

**Expected Output:** A button with the icon and all native button props forwarded. TypeScript enforces that `aria-label` is provided (required) and allows `type`, `onClick`, `disabled`, etc.

**Why This Output Occurs:** `ComponentPropsWithoutRef<'button'>` includes all native button props. The intersection adds `icon` and makes `aria-label` required. The `...rest` forwards native props to the DOM element.

### Real-World Cases

- **Design systems:** Every component has a `Props` type exported for consumers.
- **Wrapper components:** Extending `ComponentPropsWithoutRef` to wrap native elements.
- **Generic lists:** `List<T>` for reusable data-driven components.
- **Compound components:** Each subcomponent has its own props type.
- **Polymorphic components:** `as` prop with generic typing.

### References

- React TypeScript Cheatsheet – Typing Component Props - https://react-typescript-cheatsheet.netlify.app/docs/basic/getting-started/basic_type_example
- React TypeScript Cheatsheet – `React.FC` - https://react-typescript-cheatsheet.netlify.app/docs/basic/getting-started/function_components
- React Official Documentation – TypeScript - https://react.dev/learn/typescript
- Matt Pocock – React Props - https://www.totaltypescript.com/react-props
- TypeScript Handbook – Intersection Types - https://www.typescriptlang.org/docs/handbook/2/objects.html#intersection-types

---

## Core Concept 2: Children — `React.ReactNode` vs. `React.ReactElement` vs. `JSX.Element`

### Definitions

**Core Definition:** `React.ReactNode`, `React.ReactElement`, and `JSX.Element` are three TypeScript types for React's `children` prop, each representing a different level of restrictiveness: `ReactNode` is the most permissive, `ReactElement` is a single element, and `JSX.Element` is the return type of a JSX expression.

**Technical Definition:** `React.ReactNode` is a union that includes `ReactElement`, `string`, `number`, `boolean`, `null`, `undefined`, and arrays of these—everything React can render. `React.ReactElement` is the type of a single React element created by `createElement` or JSX (an object with `type`, `props`, and `key`). `JSX.Element` is a subtype of `ReactElement` with `props: any` and is the return type of a JSX expression in the global JSX namespace. In practice, `ReactNode` is the correct type for the `children` prop in 99% of cases because it accepts text, numbers, elements, fragments, and arrays. `ReactElement` is used when the component expects exactly one element (e.g., a custom icon). `JSX.Element` is rarely used for props because it lacks the permissiveness of `ReactNode`.

**Beginner-Friendly Explanation:** `ReactNode` means "anything React can render"—text, numbers, elements, arrays, fragments, `null`. `ReactElement` means "exactly one React element." `JSX.Element` is the type TypeScript gives to a JSX expression (like `<div />`). For a `children` prop, use `ReactNode` almost always. Use `ReactElement` when you need exactly one element (like an icon). Avoid `JSX.Element` for props; it is too restrictive.

### Purposes

- To declare what a component accepts as children.
- To allow text, numbers, elements, and arrays where appropriate (`ReactNode`).
- To require exactly one element where appropriate (`ReactElement`).
- To document the component's flexibility or restriction.
- To avoid runtime errors from rendering invalid children.

### Syntax Rules and Structure

**`React.ReactNode` (Most Permissive):**
```tsx
type CardProps = {
  children: React.ReactNode;
};

function Card({ children }: CardProps) {
  return <div className="card">{children}</div>;
}

// Accepts anything
<Card>Hello</Card>
<Card>{42}</Card>
<Card>{null}</Card>
<Card><span>Element</span></Card>
<Card>{[<span key="1">A</span>, <span key="2">B</span>]}</Card>
<Card><><span>A</span><span>B</span></></Card>
```

**`React.ReactElement` (Single Element):**
```tsx
type IconButtonProps = {
  icon: React.ReactElement;
  children: React.ReactNode;
};

function IconButton({ icon, children }: IconButtonProps) {
  return <button>{icon}{children}</button>;
}

// Accepts exactly one element
<IconButton icon={<CloseIcon />}>Close</IconButton>
// <IconButton icon="x">Close</IconButton> // Error: string is not a ReactElement
```

**`JSX.Element` (Return Type):**
```tsx
function MyComponent(): JSX.Element {
  return <div>Hello</div>;
}

// JSX.Element is the type of the return value, not typically used for props
```

**`React.ReactNode` with Optional Children:**
```tsx
type PanelProps = {
  title: string;
  children?: React.ReactNode; // Optional
};

function Panel({ title, children }: PanelProps) {
  return (
    <div>
      <h2>{title}</h2>
      {children}
    </div>
  );
}
```

**`React.ReactNode` vs. `React.ReactChild` (Deprecated):**
```tsx
// ❌ Deprecated in React 18
type OldProps = { children: React.ReactChild };

// ✅ Use ReactNode
type NewProps = { children: React.ReactNode };
```

**Syntax Rules:**
- Use `React.ReactNode` for the `children` prop in almost all cases.
- Use `React.ReactElement` when the component requires exactly one element (icons, avatars).
- Avoid `JSX.Element` for props; use it only for return type annotations.
- Mark children optional with `?` when the component renders without them.
- `ReactNode` accepts arrays, fragments, strings, numbers, `null`, and `undefined`.
- `ReactElement` does not accept strings, numbers, arrays, or `null`.

**Constraints and Limitations:**
- `ReactNode` accepts `null` and `undefined`, so components must handle empty children.
- `ReactElement` is stricter but still allows any element type (including fragments).
- `JSX.Element` is global and may conflict with other JSX namespaces (e.g., Vue, Solid).
- `ReactNode` includes `boolean`, which renders as nothing but can be confusing.
- `ReactNode` arrays require keys when rendered as lists.

### Annotated Code Examples

**Example 1: Card with Flexible Children**

```tsx
type CardProps = {
  title: string;
  children: React.ReactNode;
  footer?: React.ReactNode;
};

function Card({ title, children, footer }: CardProps) {
  return (
    <div className="card">
      <h2>{title}</h2>
      <div className="card-body">{children}</div>
      {footer && <div className="card-footer">{footer}</div>}
    </div>
  );
}

// Usage
<Card title="Welcome" footer={<button>Close</button>}>
  <p>This is the card content.</p>
</Card>

<Card title="Numbers">
  {42} and {true} and {null}
</Card>
```

**Expected Output:** A card with a title, body, and optional footer. The body accepts paragraphs, numbers, booleans, and `null` without errors because `ReactNode` is permissive.

**Why This Output Occurs:** `ReactNode` accepts everything React can render. The `footer` prop is optional; when absent, the footer is not rendered.

**Example 2: Icon Component Requiring a Single Element**

```tsx
type IconButtonProps = {
  icon: React.ReactElement<{ className?: string }>;
  label: string;
  onClick: () => void;
};

function IconButton({ icon, label, onClick }: IconButtonProps) {
  return (
    <button aria-label={label} onClick={onClick}>
      {React.cloneElement(icon, { className: 'icon' })}
    </button>
  );
}

// Usage
<IconButton icon={<CloseIcon />} label="Close" onClick={close} />
// <IconButton icon="×" label="Close" onClick={close} /> // Error
```

**Expected Output:** An icon button that clones the icon and adds a class. TypeScript enforces that `icon` is a single React element, not a string or array.

**Why This Output Occurs:** `React.ReactElement<{ className?: string }>` types the element and its props, allowing `cloneElement` to pass `className` safely. Strings and arrays are rejected.

### Real-World Cases

- **Layout components:** `children: ReactNode` for arbitrary content.
- **Icon slots:** `icon: ReactElement` for a single icon.
- **Compound components:** Each subcomponent types its own children.
- **Portals:** `children: ReactNode` for portal content.
- **Render props:** Functions returning `ReactNode`.

### References

- React TypeScript Cheatsheet – Children - https://react-typescript-cheatsheet.netlify.app/docs/basic/getting-started/basic_type_example#children
- React TypeScript Cheatsheet – ReactNode vs ReactElement - https://react-typescript-cheatsheet.netlify.app/docs/basic/getting-started/basic_type_example
- React Official Documentation – `ReactNode` - https://react.dev/reference/react/ReactNode
- React Official Documentation – `cloneElement` - https://react.dev/reference/react/cloneElement
- TypeScript Handbook – JSX - https://www.typescriptlang.org/docs/handbook/jsx.html

---

## Core Concept 3: Events — `React.ChangeEvent`, `React.FormEvent`, `React.MouseEvent`

### Definitions

**Core Definition:** React event types are TypeScript types for the synthetic events React provides, each corresponding to a specific DOM event and element type—`React.ChangeEvent<HTMLInputElement>`, `React.FormEvent<HTMLFormElement>`, `React.MouseEvent<HTMLButtonElement>`, etc.

**Technical Definition:** React's synthetic event system wraps native DOM events in a `SyntheticEvent` object that normalises browser differences. Each event type is generic over the element type: `React.ChangeEvent<T>` where `T extends HTMLElement`. The most common event types are `ChangeEvent` (input, select, textarea changes), `FormEvent` (form submission), `MouseEvent` (click, mouseenter, etc.), `KeyboardEvent` (keydown, keyup), `FocusEvent` (focus, blur), and `SubmitEvent` (deprecated in favour of `FormEvent`). The element type narrows the `event.target` and `event.currentTarget` properties, enabling type-safe access to `.value`, `.checked`, `.files`, and other element-specific properties.

**Beginner-Friendly Explanation:** When you handle a click, a form submission, or a text input change, React gives you an event object. TypeScript needs to know what kind of event it is and what element it came from so it can type `.target`, `.value`, and other properties correctly. `React.ChangeEvent<HTMLInputElement>` tells TypeScript "this is a change event from an input element," so `.target.value` is a string. Using the wrong event type causes compile errors or unsafe access.

### Purposes

- To type event handler parameters correctly.
- To enable type-safe access to `event.target` and `event.currentTarget`.
- To distinguish between different event types (change, form, mouse, keyboard).
- To catch event handler signature mismatches at compile time.
- To support `preventDefault()`, `stopPropagation()`, and other event methods.

### Syntax Rules and Structure

**Change Events:**
```tsx
function TextInput() {
  const [value, setValue] = useState('');

  function handleChange(e: React.ChangeEvent<HTMLInputElement>) {
    setValue(e.target.value); // string
  }

  return <input value={value} onChange={handleChange} />;
}
```

**Component Breakdown:**
- `React.ChangeEvent<HTMLInputElement>`: Change event from an input element.
- `e.target.value`: `string` (typed correctly).
- `e.currentTarget`: `HTMLInputElement` (the element the handler is attached to).

**Form Events:**
```tsx
function LoginForm() {
  function handleSubmit(e: React.FormEvent<HTMLFormElement>) {
    e.preventDefault();
    const formData = new FormData(e.currentTarget);
    const email = formData.get('email');
  }

  return <form onSubmit={handleSubmit}>{/* ... */}</form>;
}
```

**Component Breakdown:**
- `React.FormEvent<HTMLFormElement>`: Form submission event.
- `e.currentTarget`: `HTMLFormElement` (typed correctly for `FormData`).
- `e.preventDefault()`: Prevents the default form submission.

**Mouse Events:**
```tsx
type ButtonProps = {
  onClick: (e: React.MouseEvent<HTMLButtonElement>) => void;
};

function Button({ onClick }: ButtonProps) {
  return <button onClick={onClick}>Click</button>;
}

// Usage
<Button onClick={(e) => console.log(e.currentTarget.disabled)} />
```

**Component Breakdown:**
- `React.MouseEvent<HTMLButtonElement>`: Mouse event from a button.
- `e.currentTarget`: `HTMLButtonElement` (typed correctly for `.disabled`, `.type`).

**Keyboard Events:**
```tsx
function SearchInput() {
  function handleKeyDown(e: React.KeyboardEvent<HTMLInputElement>) {
    if (e.key === 'Enter') {
      e.preventDefault();
      submit(e.currentTarget.value);
    }
  }

  return <input onKeyDown={handleKeyDown} />;
}
```

**Component Breakdown:**
- `React.KeyboardEvent<HTMLInputElement>`: Keyboard event from an input.
- `e.key`: The key that was pressed (e.g., `'Enter'`, `'Escape'`).
- `e.currentTarget.value`: The input's value.

**Focus Events:**
```tsx
function Field() {
  function handleFocus(e: React.FocusEvent<HTMLInputElement>) {
    console.log('Focused:', e.target.name);
  }

  return <input name="email" onFocus={handleFocus} />;
}
```

**Event Type Reference:**

| Event | Type | Common Elements |
|---|---|---|
| **Change** | `React.ChangeEvent<T>` | `<input>`, `<select>`, `<textarea>` |
| **Form** | `React.FormEvent<T>` | `<form>` |
| **Mouse** | `React.MouseEvent<T>` | Any element |
| **Keyboard** | `React.KeyboardEvent<T>` | Any element |
| **Focus** | `React.FocusEvent<T>` | Any focusable element |
| **Submit** | `React.FormEvent<T>` | `<form>` (use FormEvent, not SubmitEvent) |
| **Drag** | `React.DragEvent<T>` | Draggable elements |
| **Clipboard** | `React.ClipboardEvent<T>` | Any element |
| **Touch** | `React.TouchEvent<T>` | Touch-enabled elements |
| **Pointer** | `React.PointerEvent<T>` | Any element |
| **Wheel** | `React.WheelEvent<T>` | Scrollable elements |
| **Animation** | `React.AnimationEvent<T>` | Any element |
| **Transition** | `React.TransitionEvent<T>` | Any element |

**Syntax Rules:**
- Always specify the element type: `React.ChangeEvent<HTMLInputElement>`.
- Use `e.target` for the element that triggered the event.
- Use `e.currentTarget` for the element the handler is attached to (more reliable).
- Use `React.FormEvent<HTMLFormElement>` for form submissions (not `SubmitEvent`).
- Use `React.MouseEvent<HTMLButtonElement>` for button clicks.
- Use `React.KeyboardEvent<HTMLInputElement>` for input key handlers.
- For element-agnostic handlers, use the base type: `React.MouseEvent`.
- For handlers that accept any element, use `React.SyntheticEvent`.

**Constraints and Limitations:**
- `e.target` may be a child element; `e.currentTarget` is always the element the handler is attached to.
- `React.ChangeEvent<HTMLInputElement>` does not work for `<select>` or `<textarea>`; use the corresponding element type.
- TypeScript cannot narrow event types automatically; you must annotate the handler parameter.
- Inline arrow functions infer the event type from the JSX attribute.
- Custom events (e.g., from a library) require custom event types.

### Annotated Code Examples

**Example 1: Form with Multiple Field Types**

```tsx
function RegistrationForm() {
  const [email, setEmail] = useState('');
  const [age, setAge] = useState(0);
  const [subscribe, setSubscribe] = useState(false);

  function handleEmailChange(e: React.ChangeEvent<HTMLInputElement>) {
    setEmail(e.target.value);
  }

  function handleAgeChange(e: React.ChangeEvent<HTMLInputElement>) {
    setAge(Number(e.target.value));
  }

  function handleSubscribeChange(e: React.ChangeEvent<HTMLInputElement>) {
    setSubscribe(e.target.checked);
  }

  function handleSubmit(e: React.FormEvent<HTMLFormElement>) {
    e.preventDefault();
    console.log({ email, age, subscribe });
  }

  return (
    <form onSubmit={handleSubmit}>
      <label>Email <input type="email" value={email} onChange={handleEmailChange} /></label>
      <label>Age <input type="number" value={age} onChange={handleAgeChange} /></label>
      <label><input type="checkbox" checked={subscribe} onChange={handleSubscribeChange} /> Subscribe</label>
      <button type="submit">Register</button>
    </form>
  );
}
```

**Expected Output:** A form with email, age, and subscribe fields. TypeScript correctly types `e.target.value` as a string for email, `e.target.value` as a string (converted to number) for age, and `e.target.checked` as a boolean for the checkbox. `handleSubmit` prevents the default form submission.

**Why This Output Occurs:** Each handler annotates the correct `ChangeEvent<T>` type. TypeScript narrows the target properties based on the element type: `input[type="checkbox"]` has `.checked`, `input[type="email"]` has `.value` as a string, etc.

**Example 2: Keyboard Shortcut with KeyboardEvent**

```tsx
function SearchBox({ onSearch }: { onSearch: (q: string) => void }) {
  function handleKeyDown(e: React.KeyboardEvent<HTMLInputElement>) {
    if ((e.ctrlKey || e.metaKey) && e.key === 'k') {
      e.preventDefault();
      e.currentTarget.select();
    } else if (e.key === 'Enter') {
      onSearch(e.currentTarget.value);
    }
  }

  return <input type="search" onKeyDown={handleKeyDown} placeholder="Search (Ctrl+K)" />;
}
```

**Expected Output:** Pressing Ctrl+K (or Cmd+K) focuses and selects the input. Pressing Enter calls `onSearch` with the current value. TypeScript types `e.key`, `e.ctrlKey`, `e.metaKey`, and `e.currentTarget.value` correctly.

**Why This Output Occurs:** `React.KeyboardEvent<HTMLInputElement>` provides typed access to keyboard properties and the input element. `e.currentTarget` is the input, so `.value` and `.select()` are available.

### Real-World Cases

- **Forms:** `ChangeEvent` for inputs, `FormEvent` for submission.
- **Buttons:** `MouseEvent<HTMLButtonElement>` for click handlers.
- **Search inputs:** `KeyboardEvent<HTMLInputElement>` for Enter/Escape.
- **Drag-and-drop:** `DragEvent<HTMLDivElement>` for drag handlers.
- **Clipboard:** `ClipboardEvent<HTMLInputElement>` for paste handlers.
- **File uploads:** `ChangeEvent<HTMLInputElement>` with `e.target.files`.

### References

- React TypeScript Cheatsheet – Events - https://react-typescript-cheatsheet.netlify.app/docs/basic/getting-started/forms_and_events
- React Official Documentation – SyntheticEvent - https://react.dev/reference/react-dom/components/common#react-event-object
- React Official Documentation – TypeScript Events - https://react.dev/learn/typescript#typing-dom-events
- MDN Web Docs – Event reference - https://developer.mozilla.org/en-US/docs/Web/Events
- TypeScript Handbook – Generics - https://www.typescriptlang.org/docs/handbook/2/generics.html

---

## Core Concept 4: Component State — Inference vs. Explicit Generics

### Definitions

**Core Definition:** Typing component state is the practice of declaring the type of `useState` values, either by relying on TypeScript's inference from the initial value or by explicitly specifying a generic type parameter.

**Technical Definition:** `useState<T>(initialValue)` is generic over the state type `T`. When the initial value is a primitive (`0`, `''`, `false`), TypeScript infers `T` automatically. When the state can be a union (e.g., `User | null`), an empty array (`[]`), or a complex object, explicit typing is required: `useState<User | null>(null)`, `useState<User[]>([])`, `useState<{ count: number }>({ count: 0 })`. Lazy initialisers (`useState(() => computeInitial())`) are also typed. The setter type is `Dispatch<SetStateAction<T>>`, which accepts either a new value or an updater function `(prev: T) => T`. Explicit typing is required when inference would produce `any`, `unknown`, or a too-narrow type.

**Beginner-Friendly Explanation:** `useState` can usually figure out the type from the initial value. If you write `useState(0)`, it knows the state is a number. But if you write `useState(null)`, it only knows the state is `null`—it does not know you will later set it to a `User`. You have to tell it: `useState<User | null>(null)`. Similarly, `useState([])` infers `any[]`, so you should write `useState<User[]>([])`.

### Purposes

- To declare the type of a state value precisely.
- To allow `null` or `undefined` as valid state values.
- To type arrays and objects that start empty.
- To enable type-safe state updates (setter accepts the correct type).
- To support discriminated union state (e.g., request status).
- To enable editor autocomplete for state values.

### Syntax Rules and Structure

**Inference (Simple Values):**
```tsx
const [count, setCount] = useState(0);        // number
const [name, setName] = useState('');         // string
const [isOpen, setIsOpen] = useState(false);  // boolean
```

**Explicit Generic (Unions, Null, Empty Arrays):**
```tsx
type User = { id: number; name: string };

const [user, setUser] = useState<User | null>(null);
const [users, setUsers] = useState<User[]>([]);
const [status, setStatus] = useState<'idle' | 'loading' | 'success' | 'error'>('idle');
```

**Lazy Initialiser:**
```tsx
const [data, setData] = useState<User[]>(() => {
  const stored = localStorage.getItem('users');
  return stored ? JSON.parse(stored) : [];
});
```

**Component Breakdown:**
- `useState<User[]>(() => ...)`: Explicit generic with a lazy initialiser.
- The initialiser runs only on the first render.

**Discriminated Union State:**
```tsx
type RequestState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: Error };

function useRequest<T>() {
  const [state, setState] = useState<RequestState<T>>({ status: 'idle' });
  return { state, setState };
}

// Usage
function UserProfile() {
  const { state } = useRequest<User>();

  if (state.status === 'loading') return <Spinner />;
  if (state.status === 'error') return <Error message={state.error.message} />;
  if (state.status === 'success') return <h1>{state.data.name}</h1>;
  return <button>Load</button>;
}
```

**Component Breakdown:**
- `RequestState<T>`: A discriminated union of the four states.
- `useState<RequestState<T>>`: Explicit generic preserves the union.
- The `switch` on `state.status` narrows the type in each branch.

**Setter with Updater Function:**
```tsx
const [count, setCount] = useState(0);

setCount(1);                    // Direct value
setCount((prev) => prev + 1);   // Updater function
```

**Component Breakdown:**
- `setCount` accepts either a value or `(prev: number) => number`.
- The updater function avoids stale state in asynchronous updates.

**Syntax Rules:**
- Let TypeScript infer state types for primitives.
- Use explicit generics for unions, `null`, empty arrays, and complex objects.
- Use lazy initialisers for expensive initial computation.
- Use discriminated unions for state machines (idle, loading, success, error).
- Type the setter as `Dispatch<SetStateAction<T>>` only when passing it as a prop.
- For empty arrays, always specify the element type: `useState<User[]>([])`.
- For `null` initial values, use a union: `useState<User | null>(null)`.

**Constraints and Limitations:**
- `useState([])` infers `any[]`; the array can hold anything, and TypeScript does not catch element type errors.
- `useState(null)` infers `null`; you cannot set it to anything else without an explicit generic.
- Updater functions must return the same type as the state.
- Excessive state variables increase re-renders; consider `useReducer` for related state.
- `useState` does not support async initialisers; use a lazy initialiser that returns a value.

### Annotated Code Examples

**Example 1: User Profile with Nullable State**

```tsx
type User = {
  id: number;
  name: string;
  email: string;
};

function UserProfile({ userId }: { userId: number }) {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    setLoading(true);
    fetch(`/api/users/${userId}`)
      .then((res) => res.json())
      .then((data: User) => {
        setUser(data);
        setLoading(false);
      });
  }, [userId]);

  if (loading) return <p>Loading...</p>;
  if (!user) return <p>No user found</p>;
  return <h1>{user.name}</h1>;
}
```

**Expected Output:** The component shows "Loading..." initially, then the user's name. TypeScript enforces that `user` is checked for `null` before accessing `user.name`.

**Why This Output Occurs:** `useState<User | null>(null)` allows the state to be `User` or `null`. The `if (!user)` check narrows the type to `User`, so `user.name` is safe.

**Example 2: Discriminated Union State Machine**

```tsx
type AuthState =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'authenticated'; user: User }
  | { status: 'error'; message: string };

function LoginPage() {
  const [state, setState] = useState<AuthState>({ status: 'idle' });

  async function handleLogin(credentials: { email: string; password: string }) {
    setState({ status: 'loading' });
    try {
      const user = await login(credentials);
      setState({ status: 'authenticated', user });
    } catch (error) {
      setState({ status: 'error', message: (error as Error).message });
    }
  }

  switch (state.status) {
    case 'idle':
      return <LoginForm onSubmit={handleLogin} />;
    case 'loading':
      return <Spinner />;
    case 'authenticated':
      return <h1>Welcome, {state.user.name}</h1>;
    case 'error':
      return <p role="alert">{state.message}</p>;
  }
}
```

**Expected Output:** The page renders a login form, spinner, welcome message, or error based on the current state. TypeScript narrows the union in each `switch` branch.

**Why This Output Occurs:** The discriminated union uses `status` as the discriminant. TypeScript narrows the state type based on the `case`, so `state.user` is available only in the `authenticated` branch.

### Real-World Cases

- **Request state:** Idle, loading, success, error for data fetching.
- **Form state:** Values, errors, touched, dirty flags.
- **Modal state:** Closed, opening, open, closing.
- **Multi-step wizard:** Current step and accumulated data.
- **Authentication:** Idle, loading, authenticated, error.

### References

- React Official Documentation – `useState` - https://react.dev/reference/react/useState
- React Official Documentation – TypeScript - https://react.dev/learn/typescript
- React TypeScript Cheatsheet – useState - https://react-typescript-cheatsheet.netlify.app/docs/basic/getting-started/hooks#usestate
- TypeScript Handbook – Generics - https://www.typescriptlang.org/docs/handbook/2/generics.html
- TypeScript Handbook – Discriminated Unions - https://www.typescriptlang.org/docs/handbook/2/narrowing.html#discriminated-unions

---

## Core Concept 5: Refs — `RefObject` vs. `MutableRefObject`

### Definitions

**Core Definition:** `React.RefObject<T>` is a read-only ref whose `current` property cannot be reassigned, used for DOM elements; `React.MutableRefObject<T>` is a mutable ref whose `current` property can be reassigned, used for persistent values across renders.

**Technical Definition:** In React 19, `useRef<T>(initialValue)` returns `RefObject<T>` when the initial value is `T` and `MutableRefObject<T>` when the initial value is `T | null` and the ref is used for DOM elements. `RefObject<T>` has `readonly current: T | null`, reflecting that React manages the assignment of the DOM element. `MutableRefObject<T>` has `current: T`, allowing the developer to read and write it freely. The distinction is important: DOM refs are read-only because React assigns the element during commit; mutable refs are for persistent values (timers, previous values, instance variables) that the developer controls. The `useRef` overloads determine which type is returned based on the initial value and usage.

**Beginner-Friendly Explanation:** A ref is a box that persists across renders. `RefObject` is a box for a DOM element—React puts the element in the box, and you read it, but you do not put anything else in it. `MutableRefObject` is a box for anything you want to persist—you can put a timer ID, a previous value, or any other value in it, and you can change it whenever you want. The types reflect this difference: `RefObject.current` is `readonly`, `MutableRefObject.current` is not.

### Purposes

- To reference a DOM element (e.g., to focus an input).
- To store a mutable value across renders without triggering re-renders.
- To store the previous value of a prop or state.
- To store timer IDs, interval IDs, or subscription handles.
- To store an imperative handle (`useImperativeHandle`).
- To distinguish read-only DOM refs from mutable value refs.

### Syntax Rules and Structure

**DOM Ref (`RefObject`):**
```tsx
function AutoFocusInput() {
  const inputRef = useRef<HTMLInputElement>(null);
  // inputRef: React.RefObject<HTMLInputElement>

  useEffect(() => {
    inputRef.current?.focus(); // current is HTMLInputElement | null
  }, []);

  return <input ref={inputRef} />;
}
```

**Component Breakdown:**
- `useRef<HTMLInputElement>(null)`: Returns a `RefObject<HTMLInputElement>`.
- `inputRef.current`: `HTMLInputElement | null` (readonly).
- `inputRef.current?.focus()`: Safe access with optional chaining.

**Mutable Ref (`MutableRefObject`):**
```tsx
function Timer() {
  const timerRef = useRef<number | null>(null);
  // timerRef: React.MutableRefObject<number | null>

  function start() {
    timerRef.current = window.setInterval(() => {
      console.log('Tick');
    }, 1000);
  }

  function stop() {
    if (timerRef.current !== null) {
      clearInterval(timerRef.current);
      timerRef.current = null;
    }
  }

  return (
    <div>
      <button onClick={start}>Start</button>
      <button onClick={stop}>Stop</button>
    </div>
  );
}
```

**Component Breakdown:**
- `useRef<number | null>(null)`: Returns a `MutableRefObject<number | null>`.
- `timerRef.current`: `number | null` (mutable).
- The timer ID is stored and cleared across renders.

**Previous Value Ref:**
```tsx
function usePrevious<T>(value: T): T | undefined {
  const ref = useRef<T | undefined>(undefined);

  useEffect(() => {
    ref.current = value;
  }, [value]);

  return ref.current;
}

function Counter() {
  const [count, setCount] = useState(0);
  const previousCount = usePrevious(count);

  return (
    <p>
      Current: {count}, Previous: {previousCount ?? 'none'}
    </p>
  );
}
```

**Component Breakdown:**
- `usePrevious<T>`: A generic hook that stores the previous value.
- `ref.current = value`: Updates the ref in an effect.
- Returns the previous value (or `undefined` on the first render).

**Forwarding Refs:**
```tsx
type InputProps = React.ComponentPropsWithoutRef<'input'>;

const Input = React.forwardRef<HTMLInputElement, InputProps>(
  function Input(props, ref) {
    return <input ref={ref} {...props} />;
  }
);

// Usage
const inputRef = useRef<HTMLInputElement>(null);
<Input ref={inputRef} />;
```

**Component Breakdown:**
- `React.forwardRef<HTMLInputElement, InputProps>`: Types the ref and props.
- `props` and `ref`: Forwarded to the underlying `<input>`.

**Ref Type Reference:**

| Usage | Type | `current` Type | Mutable? |
|---|---|---|---|
| **DOM element** | `RefObject<T>` | `T \| null` | No (readonly) |
| **Mutable value** | `MutableRefObject<T>` | `T` | Yes |
| **DOM element (legacy)** | `MutableRefObject<T \| null>` | `T \| null` | Yes |

**Syntax Rules:**
- Use `useRef<HTMLInputElement>(null)` for DOM elements; it returns `RefObject<HTMLInputElement>`.
- Use `useRef<number | null>(null)` for mutable values; it returns `MutableRefObject<number | null>`.
- Always initialise `useRef` with a value; React 19 requires it.
- Use optional chaining (`ref.current?.focus()`) for DOM refs.
- Use `forwardRef<ElementType, PropsType>` to forward refs to DOM elements.
- In React 19, `ref` can be passed as a regular prop to function components.
- Type the ref parameter in `forwardRef` as `React.ForwardedRef<HTMLElement>`.

**Constraints and Limitations:**
- Refs do not trigger re-renders when their `current` changes.
- `ref.current` is `null` before the first commit and after unmount.
- `useRef` requires an initial value in React 19; `useRef()` without arguments is removed.
- `MutableRefObject<T>` with `T = null` requires null checks before use.
- Refs are not a substitute for state; use state for values that affect rendering.

### Annotated Code Examples

**Example 1: DOM Ref for Focus Management**

```tsx
function SearchBar() {
  const inputRef = useRef<HTMLInputElement>(null);

  useEffect(() => {
    inputRef.current?.focus();
  }, []);

  function handleClear() {
    if (inputRef.current) {
      inputRef.current.value = '';
      inputRef.current.focus();
    }
  }

  return (
    <div>
      <input ref={inputRef} type="search" placeholder="Search..." />
      <button onClick={handleClear}>Clear</button>
    </div>
  );
}
```

**Expected Output:** The input is focused on mount. Clicking "Clear" empties the input and refocuses it. TypeScript types `inputRef.current` as `HTMLInputElement | null`, requiring a null check before accessing `.value`.

**Why This Output Occurs:** `useRef<HTMLInputElement>(null)` returns a `RefObject<HTMLInputElement>` whose `current` is `HTMLInputElement | null`. The null check ensures safe access.

**Example 2: Mutable Ref for a Timer**

```tsx
function Stopwatch() {
  const [elapsed, setElapsed] = useState(0);
  const intervalRef = useRef<number | null>(null);

  function start() {
    if (intervalRef.current !== null) return;
    intervalRef.current = window.setInterval(() => {
      setElapsed((prev) => prev + 1);
    }, 1000);
  }

  function stop() {
    if (intervalRef.current !== null) {
      clearInterval(intervalRef.current);
      intervalRef.current = null;
    }
  }

  useEffect(() => {
    return () => {
      if (intervalRef.current !== null) {
        clearInterval(intervalRef.current);
      }
    };
  }, []);

  return (
    <div>
      <p>Elapsed: {elapsed}s</p>
      <button onClick={start}>Start</button>
      <button onClick={stop}>Stop</button>
    </div>
  );
}
```

**Expected Output:** The stopwatch counts seconds when started and stops when stopped. The interval ID is stored in a mutable ref, so it persists across renders without causing re-renders. Cleanup clears the interval on unmount.

**Why This Output Occurs:** `useRef<number | null>(null)` returns a `MutableRefObject<number | null>`. The ref holds the interval ID across renders. The cleanup effect ensures the interval is cleared on unmount.

### Real-World Cases

- **Focus management:** Ref to an input to focus it on mount or after an action.
- **Scroll position:** Ref to a container to scroll to a specific element.
- **Timers:** Ref to store interval IDs for cleanup.
- **Previous values:** Ref to store the previous value of a prop or state.
- **Third-party libraries:** Ref to a container for a chart, map, or editor.
- **Imperative handles:** `useImperativeHandle` with a `forwardRef`.

### References

- React Official Documentation – `useRef` - https://react.dev/reference/react/useRef
- React Official Documentation – `forwardRef` - https://react.dev/reference/react/forwardRef
- React Official Documentation – Manipulating the DOM with Refs - https://react.dev/learn/manipulating-the-dom-with-refs
- React TypeScript Cheatsheet – useRef - https://react-typescript-cheatsheet.netlify.app/docs/basic/getting-started/hooks#useref
- React 19 – Ref as a Prop - https://react.dev/blog/2024/12/05/react-19#ref-as-a-prop
- TypeScript Handbook – Generics - https://www.typescriptlang.org/docs/handbook/2/generics.html

---

## Core Concept 6: Context — Null Handling, Strict Providers, and Custom Hooks

### Definitions

**Core Definition:** Typing React Context is the practice of declaring the context value type, handling the `null` initial value, enforcing that providers supply a value, and exporting a custom hook that throws if used outside the provider.

**Technical Definition:** `React.createContext<T>(defaultValue)` is generic over the context value type `T`. When `T` includes `null` (e.g., `createContext<User | null>(null)`), consumers must handle `null`, but this weakens type safety because components may render without a provider. The recommended pattern is to use `createContext<T | null>(null)` and provide a custom hook that reads the context and throws if the value is `null`: this gives consumers a non-null `T` and fails loudly if the provider is missing. Strict providers wrap `Context.Provider` with a typed `value` prop and are often split into state and dispatch contexts to prevent re-renders of dispatch-only consumers. The custom hook is the public API; the context object itself is kept private.

**Beginner-Friendly Explanation:** Context is a way to share data across many components without prop drilling. When you create a context, you have to give it a default value. If the default is `null`, every consumer has to check for `null`—annoying and error-prone. The better pattern is to create the context with `null` as the default, then write a custom hook (`useUser`) that throws if the context is `null`. Now every consumer gets a non-null value, and if someone forgets to wrap the app in the provider, they get a clear error instead of a silent bug.

### Purposes

- To share data across a component tree without prop drilling.
- To type the context value precisely.
- To handle the `null` initial value safely.
- To enforce that a provider is present.
- To provide a clean, typed consumption API via a custom hook.
- To split state and dispatch contexts for performance.

### Syntax Rules and Structure

**Basic Context (Nullable):**
```tsx
type User = { id: number; name: string };

const UserContext = React.createContext<User | null>(null);

function useUser() {
  const user = React.useContext(UserContext);
  if (user === null) {
    throw new Error('useUser must be used within a UserProvider');
  }
  return user;
}

function UserProvider({ user, children }: { user: User; children: React.ReactNode }) {
  return <UserContext.Provider value={user}>{children}</UserContext.Provider>;
}

// Usage
function UserBadge() {
  const user = useUser(); // user: User (non-null)
  return <span>{user.name}</span>;
}
```

**Component Breakdown:**
- `createContext<User | null>(null)`: The context value is `User` or `null`.
- `useUser`: Custom hook that throws if the value is `null`.
- `UserProvider`: Wraps the tree and supplies the `user`.
- `UserBadge`: Uses `useUser` and gets a non-null `User`.

**Split State and Dispatch Contexts:**
```tsx
type State = { count: number };
type Action = { type: 'increment' } | { type: 'decrement' };

const StateContext = React.createContext<State | null>(null);
const DispatchContext = React.createContext<React.Dispatch<Action> | null>(null);

function useCounterState() {
  const state = React.useContext(StateContext);
  if (state === null) throw new Error('useCounterState must be used within CounterProvider');
  return state;
}

function useCounterDispatch() {
  const dispatch = React.useContext(DispatchContext);
  if (dispatch === null) throw new Error('useCounterDispatch must be used within CounterProvider');
  return dispatch;
}

function CounterProvider({ children }: { children: React.ReactNode }) {
  const [state, dispatch] = React.useReducer(reducer, { count: 0 });

  return (
    <StateContext.Provider value={state}>
      <DispatchContext.Provider value={dispatch}>
        {children}
      </DispatchContext.Provider>
    </StateContext.Provider>
  );
}
```

**Component Breakdown:**
- `StateContext`: Holds the state; consumers re-render when it changes.
- `DispatchContext`: Holds the dispatch function (stable reference); consumers never re-render from state changes.
- Separate hooks for state and dispatch.

**Generic Context:**
```tsx
function createGenericContext<T>() {
  const Context = React.createContext<T | null>(null);

  function useGenericContext() {
    const value = React.useContext(Context);
    if (value === null) throw new Error('useGenericContext must be used within a Provider');
    return value;
  }

  return [Context.Provider, useGenericContext] as const;
}

// Usage
const [ThemeProvider, useTheme] = createGenericContext<{ theme: string }>();
```

**Component Breakdown:**
- `createGenericContext<T>`: A factory that creates a typed context and hook.
- Returns a tuple of the provider and the custom hook.

**Syntax Rules:**
- Always use `createContext<T | null>(null)` and a custom hook that throws.
- Never expose the context object directly; export the custom hook instead.
- Split state and dispatch into separate contexts for performance.
- Type the provider's `value` prop precisely; TypeScript enforces it.
- Use `React.Dispatch<Action>` for the dispatch function's type.
- Wrap the provider's value in `useMemo` to prevent consumer re-renders.
- For generic contexts, use a factory function.

**Constraints and Limitations:**
- Context consumers re-render whenever the context value changes.
- A `null` default value requires either handling `null` or using a throwing hook.
- Splitting contexts adds boilerplate but improves performance.
- Context is not a state management solution; it is a dependency injection mechanism.
- Context values must be memoised to prevent unnecessary re-renders.

### Annotated Code Examples

**Example 1: Typed Theme Context**

```tsx
type Theme = 'light' | 'dark';

type ThemeContextValue = {
  theme: Theme;
  toggleTheme: () => void;
};

const ThemeContext = React.createContext<ThemeContextValue | null>(null);

export function useTheme(): ThemeContextValue {
  const context = React.useContext(ThemeContext);
  if (context === null) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
}

export function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = React.useState<Theme>('light');

  const value = React.useMemo(
    () => ({
      theme,
      toggleTheme: () => setTheme((t) => (t === 'light' ? 'dark' : 'light')),
    }),
    [theme]
  );

  return <ThemeContext.Provider value={value}>{children}</ThemeContext.Provider>;
}

// Usage
function ThemeToggle() {
  const { theme, toggleTheme } = useTheme();
  return <button onClick={toggleTheme}>Switch to {theme === 'light' ? 'dark' : 'light'}</button>;
}
```

**Expected Output:** A toggle button that switches between light and dark themes. `useTheme` returns a non-null `ThemeContextValue`; if used outside `ThemeProvider`, it throws a clear error. The value is memoised to prevent unnecessary re-renders.

**Why This Output Occurs:** `createContext<ThemeContextValue | null>(null)` allows the context to be `null` by default. `useTheme` throws if it is `null`, ensuring consumers get a non-null value. `useMemo` stabilises the value object.

**Example 2: Split State and Dispatch Contexts**

```tsx
type Todo = { id: number; text: string; done: boolean };
type Action =
  | { type: 'add'; text: string }
  | { type: 'toggle'; id: number }
  | { type: 'remove'; id: number };

const TodosStateContext = React.createContext<Todo[] | null>(null);
const TodosDispatchContext = React.createContext<React.Dispatch<Action> | null>(null);

function useTodosState(): Todo[] {
  const state = React.useContext(TodosStateContext);
  if (state === null) throw new Error('useTodosState must be used within TodosProvider');
  return state;
}

function useTodosDispatch(): React.Dispatch<Action> {
  const dispatch = React.useContext(TodosDispatchContext);
  if (dispatch === null) throw new Error('useTodosDispatch must be used within TodosProvider');
  return dispatch;
}

function todosReducer(state: Todo[], action: Action): Todo[] {
  switch (action.type) {
    case 'add':
      return [...state, { id: Date.now(), text: action.text, done: false }];
    case 'toggle':
      return state.map((t) => (t.id === action.id ? { ...t, done: !t.done } : t));
    case 'remove':
      return state.filter((t) => t.id !== action.id);
  }
}

export function TodosProvider({ children }: { children: React.ReactNode }) {
  const [todos, dispatch] = React.useReducer(todosReducer, []);

  return (
    <TodosStateContext.Provider value={todos}>
      <TodosDispatchContext.Provider value={dispatch}>
        {children}
      </TodosDispatchContext.Provider>
    </TodosStateContext.Provider>
  );
}

// Add form: only uses dispatch, never re-renders on state change
function AddTodo() {
  const dispatch = useTodosDispatch();
  const [text, setText] = React.useState('');

  return (
    <form onSubmit={(e) => { e.preventDefault(); dispatch({ type: 'add', text }); setText(''); }}>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <button>Add</button>
    </form>
  );
}

// List: uses state, re-renders on state change
function TodoList() {
  const todos = useTodosState();
  const dispatch = useTodosDispatch();
  return (
    <ul>
      {todos.map((todo) => (
        <li key={todo.id}>
          <input type="checkbox" checked={todo.done} onChange={() => dispatch({ type: 'toggle', id: todo.id })} />
          {todo.text}
        </li>
      ))}
    </ul>
  );
}
```

**Expected Output:** The `AddTodo` form never re-renders when the list changes (it only uses `useTodosDispatch`). The `TodoList` re-renders when todos change. The reducer handles all three action types exhaustively.

**Why This Output Occurs:** `TodosDispatchContext` holds the dispatch function, which is stable across renders. `AddTodo` subscribes only to dispatch, so it never re-renders when todos change. `TodoList` subscribes to state, so it re-renders on every todo change. The reducer's discriminated union ensures all action types are handled.

### Real-World Cases

- **Theme context:** Light/dark mode with a toggle function.
- **Auth context:** Current user, login, logout.
- **Cart context:** Cart items, add/remove functions.
- **Locale context:** Current language and translation function.
- **Feature flags:** Enabled/disabled features.
- **Form context:** Form state and validation (e.g., React Hook Form's `FormProvider`).

### References

- React Official Documentation – `createContext` - https://react.dev/reference/react/createContext
- React Official Documentation – `useContext` - https://react.dev/reference/react/useContext
- React Official Documentation – Scaling Up with Reducer and Context - https://react.dev/learn/scaling-up-with-reducer-and-context
- React TypeScript Cheatsheet – Context - https://react-typescript-cheatsheet.netlify.app/docs/basic/getting-started/context
- React TypeScript Cheatsheet – Excluding `null` from Context - https://react-typescript-cheatsheet.netlify.app/docs/basic/getting-started/context#excluding-null-from-context-with-a-custom-hook
- Steve Kinney – Separating Actions from State with Two Contexts - https://stevekinney.com/courses/react-performance/separating-actions-from-state-with-two-contexts

---

## Core Concept 7: Advanced Hooks — `useReducer`, `useMemo`, `useCallback`, `useTransition`

### Definitions

**Core Definition:** Typing advanced hooks is the practice of annotating the generics and parameters of `useReducer`, `useMemo`, `useCallback`, and `useTransition` so that reducers, memoised values, callbacks, and concurrent transitions are fully type-safe.

**Technical Definition:** `useReducer<Reducer<State, Action>>(reducer, initialState)` is generic over the state and action types; the reducer is typed as `(state: State, action: Action) => State`, and the action is best modelled as a discriminated union for exhaustive handling. `useMemo<T>(factory, deps)` infers `T` from the factory's return type; explicit typing is needed only when inference fails. `useCallback<T extends Function>(fn, deps)` infers the function type; the parameter and return types come from the function body. `useTransition()` returns `[isPending: boolean, startTransition: (callback: () => void) => void]`; in React 19, `startTransition` accepts an async callback, returning `Promise<void>`. TypeScript infers all of these; explicit annotations are for documentation and to catch errors at the call site.

**Beginner-Friendly Explanation:** These hooks are already type-safe by default—TypeScript infers their types from how you use them. The main work is (1) typing the reducer's state and action, (2) using discriminated unions for actions so every case is handled, and (3) understanding what `useTransition` returns. Everything else is inference.

### Purposes

- To type `useReducer`'s state and action types.
- To use discriminated unions for action types.
- To type memoised values and callbacks when inference is insufficient.
- To type `useTransition`'s `isPending` and `startTransition`.
- To catch missing action cases in reducers.
- To type async transitions in React 19.

### Syntax Rules and Structure

**`useReducer` with Discriminated Union:**
```tsx
type State = { count: number; history: number[] };

type Action =
  | { type: 'increment' }
  | { type: 'decrement' }
  | { type: 'reset'; value: number };

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1, history: [...state.history, state.count + 1] };
    case 'decrement':
      return { count: state.count - 1, history: [...state.history, state.count - 1] };
    case 'reset':
      return { count: action.value, history: [action.value] };
    default: {
      const _exhaustive: never = action;
      return state;
    }
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, { count: 0, history: [] });

  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <button onClick={() => dispatch({ type: 'reset', value: 0 })}>Reset</button>
    </div>
  );
}
```

**Component Breakdown:**
- `State`: The shape of the reducer's state.
- `Action`: A discriminated union of all possible actions.
- `reducer(state, action): State`: The reducer function.
- `default: never`: Ensures all action types are handled.
- `useReducer(reducer, initialState)`: Returns `[state, dispatch]`.

**`useReducer` with Lazy Initialiser:**
```tsx
function init(initialCount: number): State {
  return { count: initialCount, history: [] };
}

const [state, dispatch] = useReducer(reducer, 0, init);
```

**Component Breakdown:**
- `useReducer(reducer, initialArg, init)`: `init(initialArg)` produces the initial state.
- Useful when the initial state is expensive to compute.

**`useMemo` — Inference vs. Explicit:**
```tsx
// Inference works
const filtered = useMemo(() => items.filter((i) => i.active), [items]);

// Explicit generic when inference is insufficient
const config = useMemo<Config>(() => ({ theme, locale }), [theme, locale]);
```

**`useCallback` — Inference:**
```tsx
// Inference works; the parameter type comes from the function body
const handleClick = useCallback((id: number) => {
  setSelectedId(id);
}, []);

// Explicit typing
const handleClick = useCallback<(id: number) => void>((id) => {
  setSelectedId(id);
}, []);
```

**`useTransition` — Basic:**
```tsx
const [isPending, startTransition] = useTransition();
// isPending: boolean
// startTransition: (callback: () => void) => void

function handleClick() {
  startTransition(() => {
    setTab('list');
  });
}
```

**`useTransition` — React 19 Async:**
```tsx
const [isPending, startTransition] = useTransition();

function handleSave() {
  startTransition(async () => {
    await save();
    setStatus('saved');
  });
}
```

**Component Breakdown:**
- `startTransition(async () => { ... })`: React 19 supports async callbacks.
- `isPending`: `true` until the async work completes.

**Typing `useMemo` Return Type:**
```tsx
type User = { id: number; name: string };

const users = useMemo<User[]>(() => {
  return data.map((d) => ({ id: d.id, name: d.name }));
}, [data]);
```

**Syntax Rules:**
- Type the reducer's state and action explicitly.
- Use discriminated unions for action types.
- Add a `default: never` case to catch missing action handling.
- Use `useReducer(reducer, initialArg, init)` for lazy initial state.
- Let `useMemo` infer its type; specify the generic only when needed.
- Let `useCallback` infer the function type; specify the generic only for documentation.
- Type `useTransition` as `[boolean, (callback: () => void) => void]` (React 18) or with async support (React 19).
- Use `React.Dispatch<Action>` when passing `dispatch` as a prop.

**Constraints and Limitations:**
- The `default: never` case is required to catch missing action types; without it, TypeScript does not error on unhandled actions.
- `useReducer`'s state and action types must be explicit; TypeScript cannot infer them from the reducer alone.
- `useMemo` and `useCallback` are performance optimisations; their types are erased at runtime.
- `useTransition`'s `isPending` does not tell you which transition is running when there are multiple.
- In React 18, `startTransition` callbacks must be synchronous; async callbacks require React 19.

### Annotated Code Examples

**Example 1: Exhaustive Reducer with Discriminated Union**

```tsx
type FetchState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: Error };

type FetchAction<T> =
  | { type: 'fetch' }
  | { type: 'success'; data: T }
  | { type: 'error'; error: Error }
  | { type: 'reset' };

function fetchReducer<T>(state: FetchState<T>, action: FetchAction<T>): FetchState<T> {
  switch (action.type) {
    case 'fetch':
      return { status: 'loading' };
    case 'success':
      return { status: 'success', data: action.data };
    case 'error':
      return { status: 'error', error: action.error };
    case 'reset':
      return { status: 'idle' };
    default: {
      const _exhaustive: never = action;
      return state;
    }
  }
}

function useFetch<T>() {
  const [state, dispatch] = useReducer(fetchReducer<T>, { status: 'idle' });

  async function fetchData(url: string) {
    dispatch({ type: 'fetch' });
    try {
      const res = await fetch(url);
      const data: T = await res.json();
      dispatch({ type: 'success', data });
    } catch (error) {
      dispatch({ type: 'error', error: error as Error });
    }
  }

  return { state, fetchData, reset: () => dispatch({ type: 'reset' }) };
}

// Usage
function UserProfile({ userId }: { userId: number }) {
  const { state, fetchData } = useFetch<User>();

  useEffect(() => {
    fetchData(`/api/users/${userId}`);
  }, [userId]);

  switch (state.status) {
    case 'idle': return <button onClick={() => fetchData(`/api/users/${userId}`)}>Load</button>;
    case 'loading': return <Spinner />;
    case 'success': return <h1>{state.data.name}</h1>;
    case 'error': return <p role="alert">{state.error.message}</p>;
  }
}
```

**Expected Output:** The component renders a load button, spinner, user name, or error based on the state. The reducer handles all four action types, and TypeScript narrows the state in each `switch` branch.

**Why This Output Occurs:** `FetchState<T>` and `FetchAction<T>` are generic discriminated unions. The reducer handles every action type, and the `default: never` ensures exhaustiveness. The `useReducer` generic is inferred from the reducer and initial state.

**Example 2: Memoised Value and Callback**

```tsx
type Item = { id: number; name: string; active: boolean };

function ItemList({ items, query }: { items: Item[]; query: string }) {
  const filtered = useMemo<Item[]>(
    () => items.filter((i) => i.name.includes(query) && i.active),
    [items, query]
  );

  const handleSelect = useCallback<(id: number) => void>(
    (id) => console.log('Selected', id),
    []
  );

  return (
    <ul>
      {filtered.map((item) => (
        <li key={item.id} onClick={() => handleSelect(item.id)}>{item.name}</li>
      ))}
    </ul>
  );
}
```

**Expected Output:** The filtered list is memoised and recomputed only when `items` or `query` changes. `handleSelect` is stable across renders. TypeScript types `filtered` as `Item[]` and `handleSelect` as `(id: number) => void`.

**Why This Output Occurs:** `useMemo<Item[]>` and `useCallback<(id: number) => void>` explicitly type the memoised values. TypeScript would infer these types anyway; the explicit annotations serve as documentation and catch mistakes.

### Real-World Cases

- **Data fetching:** `useReducer` with a `FetchState` discriminated union.
- **Form state:** `useReducer` for values, errors, and touched flags.
- **Undo/redo:** `useReducer` with history in the state.
- **Memoised computations:** `useMemo` for filtered, sorted, and aggregated data.
- **Stable callbacks:** `useCallback` for event handlers passed to memoised children.
- **Concurrent UI:** `useTransition` for tab switching and route transitions.

### References

- React Official Documentation – `useReducer` - https://react.dev/reference/react/useReducer
- React Official Documentation – `useMemo` - https://react.dev/reference/react/useMemo
- React Official Documentation – `useCallback` - https://react.dev/reference/react/useCallback
- React Official Documentation – `useTransition` - https://react.dev/reference/react/useTransition
- React TypeScript Cheatsheet – useReducer - https://react-typescript-cheatsheet.netlify.app/docs/basic/getting-started/hooks#usereducer
- React TypeScript Cheatsheet – useMemo - https://react-typescript-cheatsheet.netlify.app/docs/basic/getting-started/hooks#usememo
- React TypeScript Cheatsheet – useCallback - https://react-typescript-cheatsheet.netlify.app/docs/basic/getting-started/hooks#usecallback
- TypeScript Handbook – Discriminated Unions - https://www.typescriptlang.org/docs/handbook/2/narrowing.html#discriminated-unions
- TypeScript Handbook – Exhaustiveness Checking - https://www.typescriptlang.org/docs/handbook/2/narrowing.html#exhaustiveness-checking

---

## Comparison and Decision Guidance

| Concept | Recommended Pattern | When to Avoid | Key Risk |
|---|---|---|---|
| **Props** | Direct destructured props with a `Props` type | `React.FC` (implicit children, hidden return type) | Missing `children` type |
| **Children** | `React.ReactNode` for flexibility | `JSX.Element` for props | Restricting children unnecessarily |
| **Children (single)** | `React.ReactElement` for icons | `ReactNode` for single-element slots | Accepting arrays where one is expected |
| **Events** | `React.ChangeEvent<HTMLInputElement>` etc. | `React.SyntheticEvent` for specific handlers | Wrong element type |
| **State** | Inference for primitives; explicit for unions/nulls | `useState(null)` without generic | `any[]` from empty arrays |
| **Refs** | `RefObject<T>` for DOM; `MutableRefObject<T>` for values | `MutableRefObject<T \| null>` for DOM | Unsafe `!` assertions |
| **Context** | `createContext<T \| null>(null)` + throwing hook | Exposing context directly | Null checks in every consumer |
| **Context (split)** | Separate state and dispatch contexts | Single context for both | Unnecessary re-renders |
| **useReducer** | Discriminated union actions + `default: never` | Loose action typing | Unhandled action types |
| **useMemo/useCallback** | Let TypeScript infer; specify generic only when needed | Over-annotating | Redundant types |
| **useTransition** | `[isPending, startTransition]`; async in React 19 | Manual `isPending` state | Sync callbacks in React 18 |

**Decision Guidance:**
- **Avoid `React.FC`** in new code; use direct destructured props.
- **Use `React.ReactNode`** for `children` unless you need exactly one element.
- **Always specify the element type** for events: `React.ChangeEvent<HTMLInputElement>`.
- **Use explicit generics** for `useState` when the initial value is `null` or `[]`.
- **Use `RefObject<T>`** for DOM refs and `MutableRefObject<T>` for persistent values.
- **Use `createContext<T | null>(null)`** with a throwing custom hook.
- **Split state and dispatch** into separate contexts for performance.
- **Use discriminated unions** for `useReducer` actions with a `default: never` case.
- **Let TypeScript infer** `useMemo` and `useCallback` types; specify generics only for clarity.
- **Use `useTransition`** for non-urgent updates; async callbacks require React 19.

---

## References

- React TypeScript Cheatsheet - https://react-typescript-cheatsheet.netlify.app/
- React TypeScript Cheatsheet – Basic Types - https://react-typescript-cheatsheet.netlify.app/docs/basic/getting-started/basic_type_example
- React TypeScript Cheatsheet – Function Components - https://react-typescript-cheatsheet.netlify.app/docs/basic/getting-started/function_components
- React TypeScript Cheatsheet – Forms and Events - https://react-typescript-cheatsheet.netlify.app/docs/basic/getting-started/forms_and_events
- React TypeScript Cheatsheet – Hooks - https://react-typescript-cheatsheet.netlify.app/docs/basic/getting-started/hooks
- React TypeScript Cheatsheet – Context - https://react-typescript-cheatsheet.netlify.app/docs/basic/getting-started/context
- React TypeScript Cheatsheet – Refs - https://react-typescript-cheatsheet.netlify.app/docs/basic/getting-started/hooks#useref
- React Official Documentation – TypeScript - https://react.dev/learn/typescript
- React Official Documentation – `useState` - https://react.dev/reference/react/useState
- React Official Documentation – `useReducer` - https://react.dev/reference/react/useReducer
- React Official Documentation – `useRef` - https://react.dev/reference/react/useRef
- React Official Documentation – `useMemo` - https://react.dev/reference/react/useMemo
- React Official Documentation – `useCallback` - https://react.dev/reference/react/useCallback
- React Official Documentation – `useTransition` - https://react.dev/reference/react/useTransition
- React Official Documentation – `createContext` - https://react.dev/reference/react/createContext
- React Official Documentation – `useContext` - https://react.dev/reference/react/useContext
- React Official Documentation – `forwardRef` - https://react.dev/reference/react/forwardRef
- React Official Documentation – Scaling Up with Reducer and Context - https://react.dev/learn/scaling-up-with-reducer-and-context
- React 19 – Ref as a Prop - https://react.dev/blog/2024/12/05/react-19#ref-as-a-prop
- TypeScript Handbook - https://www.typescriptlang.org/docs/handbook/intro.html
- TypeScript Handbook – Generics - https://www.typescriptlang.org/docs/handbook/2/generics.html
- TypeScript Handbook – Discriminated Unions - https://www.typescriptlang.org/docs/handbook/2/narrowing.html#discriminated-unions
- TypeScript Handbook – Exhaustiveness Checking - https://www.typescriptlang.org/docs/handbook/2/narrowing.html#exhaustiveness-checking
- TypeScript Handbook – Intersection Types - https://www.typescriptlang.org/docs/handbook/2/objects.html#intersection-types
- Matt Pocock – React Props - https://www.totaltypescript.com/react-props
- Matt Pocock – Total TypeScript - https://www.totaltypescript.com/