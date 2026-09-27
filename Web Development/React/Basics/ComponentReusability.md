# React Component Reusability: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** React component reusability is the practice of designing components that can be used in multiple contexts—across different pages, features, or even applications—without modification, by accepting configuration through props, slots, and composition.

**Technical Definition:** Component reusability in React is achieved by designing components as pure, configurable units that receive all variable behaviour and content through props (including the `children` prop and named slot props), rather than hard-coding values, styles, or logic. A reusable component encapsulates a single responsibility, exposes a clear and minimal API, and delegates variation to its consumers. Key techniques include configurable props (with sensible defaults), generic typing (TypeScript generics), composition via `children` and slots, the `as` prop for polymorphic rendering, and compound component patterns for complex widgets. Design-system-oriented components extend reusability across teams and applications by enforcing consistent tokens, variants, sizes, and accessibility behaviours. Reusability is not free: it requires up-front API design, careful default values, and discipline to avoid over-abstraction (creating "god components" with dozens of props) or under-abstraction (duplicating nearly identical components).

**Beginner-Friendly Explanation:** A reusable component is like a cookie cutter. You can use the same cutter to make dozens of cookies—chocolate, vanilla, gingerbread—by changing the dough (props), but the shape stays the same. If you had to build a new cutter for every cookie, you would waste time and end up with inconsistent shapes. Reusable components are the same: you build a Button once, then use it everywhere with different labels, colours, and sizes.

### Key Characteristics

- **Configuration Over Duplication:** Variation is expressed through props and children, not by creating new components.
- **Single Responsibility:** Each reusable component does one thing well (a button renders a button, a card renders a card).
- **Sensible Defaults:** Every prop has a default value so the component works out of the box with minimal configuration.
- **Clear, Minimal API:** The component exposes only the props that consumers genuinely need; everything else is internal.
- **Composition-Friendly:** Reusable components accept `children` and slot props so consumers can inject arbitrary content.
- **Accessibility Built-In:** Reusable components handle keyboard navigation, ARIA attributes, and focus management so consumers do not have to.
- **Design-Token Driven:** Styling is driven by design tokens (colours, spacing, typography) rather than hard-coded values.
- **Polymorphic Capability:** Components can render as different HTML elements or React components via an `as` prop.

### Prerequisites

- Solid understanding of React function components, JSX, and props.
- Familiarity with the `children` prop and composition patterns.
- Working knowledge of the `useState` and `useContext` Hooks.
- Basic understanding of TypeScript generics (for typed reusable components).
- Awareness of accessibility principles (ARIA, keyboard navigation).

### Related Programming Areas

- **Design Systems:** Building reusable component libraries with consistent APIs.
- **Component Composition:** Combining components via children and slots.
- **State Management:** Managing state in reusable forms and compound components.
- **Accessibility:** Ensuring reusable components are accessible by default.
- **Theming:** Using design tokens and CSS variables for consistent styling.

### Core Concepts / Features

1. Configurable Components
2. Generic UI Components
3. Reusable Forms
4. Reusable Buttons
5. Reusable Modals
6. Reusable Cards
7. Reusable Navigation Components
8. Design-System-Oriented Components

---

## Core Concept 1: Configurable Components

### Definitions

**Core Definition:** A configurable component is a component whose behaviour, appearance, and content are controlled entirely through props, allowing the same component to serve many different use cases without modification.

**Technical Definition:** Configurable components accept props that determine their variant, size, state, content, and behaviour. Each prop has a sensible default, so the component works without any configuration but can be customised as needed. The component's internal logic remains fixed (e.g., a Button always renders a `<button>` and handles clicks); only the externally visible aspects are configurable. Common configuration patterns include variant props (`variant="primary"`), size props (`size="lg"`), boolean flags (`disabled`, `loading`), event handlers (`onClick`), and content props (`children`, `label`). The key to good configuration is restraint: expose props that consumers genuinely need, and keep the API small and predictable. Over-configuration (dozens of props) leads to "god components" that are hard to understand and maintain.

**Beginner-Friendly Explanation:** A configurable component is like a pizza. You choose the size (small, medium, large), the crust (thin, thick), and the toppings (pepperoni, mushrooms). The pizza oven (component) is the same; only the configuration changes. If the pizza shop had a separate oven for every possible pizza, it would be chaos—just like having a separate Button component for every possible button style.

### Purposes

- To avoid creating near-duplicate components for every variation.
- To provide a consistent API across similar components.
- To centralise styling and behaviour in one place, so changes propagate everywhere.
- To make components easier to test (fewer components, more configuration combinations).
- To reduce bundle size by avoiding duplicate code.

### Syntax Rules and Structure

**General Syntax:**
```jsx
function Button({
  variant = "primary",
  size = "md",
  disabled = false,
  loading = false,
  onClick,
  children,
  ...rest
}) {
  return (
    <button
      className={`btn btn-${variant} btn-${size}`}
      disabled={disabled || loading}
      onClick={onClick}
      {...rest}
    >
      {loading ? "Loading..." : children}
    </button>
  );
}
```

**Component Breakdown:**
- `variant = "primary"`: A prop with a default value; controls the visual style.
- `size = "md"`: A prop with a default value; controls the size.
- `disabled = false`: A boolean flag; controls the disabled state.
- `loading = false`: A boolean flag; controls the loading state.
- `onClick`: An event handler passed through to the DOM.
- `children`: The content inside the button.
- `...rest`: Forwards any additional props (e.g., `type`, `aria-label`) to the DOM element.

**Syntax Rules:**
- Provide default values for all props so the component works without configuration.
- Use descriptive prop names (`variant`, `size`, `disabled`) rather than generic ones (`type`, `style`).
- Use `...rest` to forward additional props to the underlying DOM element.
- Do not expose internal implementation details as props.
- Keep the number of props manageable; group related props into objects if they grow too numerous.

**Constraints and Limitations:**
- Over-configuration leads to "god components" with too many props.
- Every prop is part of the public API; changing or removing one is a breaking change.
- Boolean props with `false` defaults require explicit `true` to enable; consider the naming carefully.
- Forwarding `...rest` can accidentally pass invalid DOM attributes; use TypeScript to constrain.

### Annotated Code Example: Configurable Button

```jsx
function Button({
  variant = "primary",
  size = "md",
  disabled = false,
  loading = false,
  onClick,
  children,
  ...rest
}) {
  return (
    <button
      className={`btn btn-${variant} btn-${size}`}
      disabled={disabled || loading}
      onClick={onClick}
      {...rest}
    >
      {loading ? "Loading..." : children}
    </button>
  );
}

// Usage: one component, many configurations
export default function App() {
  return (
    <div>
      <Button>Default</Button>
      <Button variant="secondary" size="lg">Large Secondary</Button>
      <Button variant="danger" disabled>Disabled Danger</Button>
      <Button variant="primary" loading>Loading</Button>
      <Button variant="ghost" onClick={() => alert("Clicked!")}>
        Click Me
      </Button>
    </div>
  );
}
```

**Expected Output:** Five buttons with different appearances: default, large secondary, disabled danger, loading, and ghost. Clicking the last button shows an alert.

**Why This Output Occurs:** The `Button` component accepts `variant`, `size`, `disabled`, `loading`, `onClick`, and `children` props, each with a default. The same component renders different appearances based on the props it receives. No duplicate components are needed.

### Real-World Cases

- **Design systems:** A single `Button` component with variants (primary, secondary, danger, ghost) and sizes (sm, md, lg).
- **Form inputs:** A single `Input` component with variants (text, email, password) and states (error, disabled).
- **Alerts:** A single `Alert` component with variants (info, success, warning, error).
- **Badges:** A single `Badge` component with variants (default, success, warning, error) and sizes.

### References

- React Official Documentation – Passing Props to a Component: https://react.dev/learn/passing-props-to-a-component
- React Official Documentation – Component API Design: https://react.dev/learn/thinking-in-react
- Smashing Magazine – How to Build Reusable React Components: https://www.smashingmagazine.com/2023/08/building-reusable-react-components/

---

## Core Concept 2: Generic UI Components

### Definitions

**Core Definition:** Generic UI components are reusable components designed to work with any data type or content, using TypeScript generics, polymorphic `as` props, or untyped `children` slots to remain agnostic about their payload.

**Technical Definition:** Generic UI components decouple their structure from their content. In TypeScript, generics allow a component to accept and render any data type while preserving type safety: `function List<T>({ items, renderItem }: { items: T[]; renderItem: (item: T) => ReactNode })`. The polymorphic `as` prop allows a component to render as different HTML elements or React components while preserving the correct prop types: `function Box<T extends ElementType>({ as, ...rest }: { as: T } & ComponentPropsWithoutRef<T>)`. Untyped slots (`children`, `header`, `footer`) allow generic content injection without type constraints. Generic components are the foundation of design systems because they provide structure (layout, styling, accessibility) without assuming anything about content.

**Beginner-Friendly Explanation:** A generic UI component is like a picture frame. The frame provides structure (border, backing, hanging hardware) but doesn't care whether the picture is a photo, a painting, or a child's drawing. In React, a generic `List` component provides the structure (a `<ul>` with styling) but doesn't care whether the items are names, products, or tasks—the consumer provides a `renderItem` function that decides how each item looks.

### Purposes

- To provide structure and styling without coupling to specific data types.
- To enable type-safe reusable components with TypeScript generics.
- To allow polymorphic rendering (`as` prop) for semantic HTML flexibility.
- To reduce duplication across similar components that differ only in content.
- To separate layout/structure concerns from content concerns.

### Syntax Rules and Structure

**Generic List with TypeScript:**
```tsx
interface ListProps<T> {
  items: T[];
  renderItem: (item: T, index: number) => React.ReactNode;
  keyExtractor: (item: T) => string | number;
}

function List<T>({ items, renderItem, keyExtractor }: ListProps<T>) {
  return (
    <ul className="list">
      {items.map((item, index) => (
        <li key={keyExtractor(item)}>{renderItem(item, index)}</li>
      ))}
    </ul>
  );
}

// Usage with type inference
<List
  items={[{ id: 1, name: "Alice" }, { id: 2, name: "Bob" }]}
  renderItem={(user) => <span>{user.name}</span>}
  keyExtractor={(user) => user.id}
/>
```

**Component Breakdown:**
- `ListProps<T>`: A generic interface for the component's props.
- `items: T[]`: An array of any type `T`.
- `renderItem: (item: T) => ReactNode`: A function that renders each item.
- `keyExtractor`: A function that returns a stable key for each item.

**Polymorphic `as` Prop:**
```tsx
import { ElementType, ComponentPropsWithoutRef } from "react";

type BoxProps<T extends ElementType> = {
  as?: T;
} & ComponentPropsWithoutRef<T>;

function Box<T extends ElementType = "div">({ as, ...rest }: BoxProps<T>) {
  const Component = as || "div";
  return <Component {...rest} />;
}

// Usage
<Box as="section" className="section">Section content</Box>
<Box as="a" href="/about">About</Box>
<Box>Default div</Box>
```

**Component Breakdown:**
- `as?: T`: An optional prop that determines the rendered element.
- `ComponentPropsWithoutRef<T>`: Infers the correct props for the chosen element.
- `const Component = as || "div"`: Defaults to `div` if no `as` is provided.

**Syntax Rules:**
- Use TypeScript generics to preserve type safety in reusable components.
- Use the `as` prop for polymorphic components that render as different elements.
- Use `ComponentPropsWithoutRef<T>` to infer the correct props for the chosen element.
- Keep generic components focused on structure; delegate content rendering to `renderItem` or `children`.
- Provide sensible defaults (e.g., `as = "div"`).

**Constraints and Limitations:**
- TypeScript generics add complexity and can be intimidating to contributors.
- The `as` prop can cause TypeScript inference issues if not carefully typed.
- Generic components can become too abstract; if the API is hard to understand, split into specific components.
- Runtime type checking is not provided by TypeScript; validate data at boundaries if needed.

### Annotated Code Example: Generic List with Render Prop

```tsx
interface ListProps<T> {
  items: T[];
  renderItem: (item: T) => React.ReactNode;
  keyExtractor: (item: T) => string | number;
  emptyMessage?: string;
}

function List<T>({
  items,
  renderItem,
  keyExtractor,
  emptyMessage = "No items",
}: ListProps<T>) {
  if (items.length === 0) {
    return <p className="list-empty">{emptyMessage}</p>;
  }

  return (
    <ul className="list">
      {items.map((item) => (
        <li key={keyExtractor(item)}>{renderItem(item)}</li>
      ))}
    </ul>
  );
}

// Usage with different data types
export default function App() {
  const users = [
    { id: 1, name: "Alice" },
    { id: 2, name: "Bob" },
  ];

  const products = [
    { sku: "A1", title: "Laptop", price: 999 },
    { sku: "B2", title: "Phone", price: 699 },
  ];

  return (
    <div>
      <h2>Users</h2>
      <List
        items={users}
        renderItem={(user) => <span>{user.name}</span>}
        keyExtractor={(user) => user.id}
      />

      <h2>Products</h2>
      <List
        items={products}
        renderItem={(product) => (
          <span>{product.title} — ${product.price}</span>
        )}
        keyExtractor={(product) => product.sku}
        emptyMessage="No products available"
      />
    </div>
  );
}
```

**Expected Output:** Two lists: one of users (Alice, Bob) and one of products (Laptop — $999, Phone — $699). The same `List` component renders both, using different `renderItem` functions.

**Why This Output Occurs:** The `List` component is generic over `T`, so it accepts both `User[]` and `Product[]`. The `renderItem` function determines how each item is rendered, and `keyExtractor` provides a stable key. The component provides the structure (a `<ul>` with list items) but does not know or care what the items are.

### Real-World Cases

- **Lists:** Generic `List` component for users, products, tasks, and any other array.
- **Tables:** Generic `Table` component with column definitions and row renderers.
- **Selects:** Generic `Select` component with option types.
- **Grids:** Generic `Grid` component that lays out any children.
- **Polymorphic typography:** A `Text` component that renders as `<p>`, `<h1>`, `<span>`, or `<label>` via the `as` prop.

### References

- React TypeScript Cheatsheet – Generic Components: https://react-typescript-cheatsheet.netlify.app/docs/advanced/patterns_by_usecase/#generic-components
- React TypeScript Cheatsheet – Polymorphic Components: https://react-typescript-cheatsheet.netlify.app/docs/advanced/patterns_by_usecase/#polymorphic-components
- Smashing Magazine – How to Build Reusable React Components: https://www.smashingmagazine.com/2023/08/building-reusable-react-components/

---

## Core Concept 3: Reusable Forms

### Definitions

**Core Definition:** A reusable form component is a component (or set of components) that encapsulates form state management, validation, submission, and error display, and can be configured with field definitions and validation rules for different forms.

**Technical Definition:** Reusable forms are typically built using a form library (React Hook Form, Formik) combined with generic field components (`Input`, `Select`, `Checkbox`, `Textarea`) that accept `label`, `name`, `error`, and validation props. The form component manages state via the library, and the field components are controlled via `register` (React Hook Form) or `field` (Formik). A schema validation library (Zod, Yup) defines the validation rules, and the form component displays errors returned by the resolver. Reusable forms can be composed: a `Form` component provides the `<form>` element and submission handling, while `FormField` components wrap inputs with labels, error messages, and accessibility attributes. This separation allows the same field components to be reused across all forms in the application, enforcing consistent styling and validation UX.

**Beginner-Friendly Explanation:** A reusable form is like a set of pre-printed forms where you fill in the blanks. You don't have to design a new form every time you need to collect information—you use the same form template and just change the labels and validation rules. In React, reusable form components (like `Input` and `Select`) provide the consistent look and behaviour, and you configure them with different names and validation rules for each form.

### Purposes

- To avoid duplicating form field markup, validation, and error display across forms.
- To enforce consistent styling and accessibility across all forms.
- To centralise validation logic in schemas (Zod, Yup).
- To simplify form state management with a library (React Hook Form, Formik).
- To make forms easier to test by isolating field components.

### Syntax Rules and Structure

**Reusable Field Component (React Hook Form):**
```tsx
import { useFormContext } from "react-hook-form";

interface FormFieldProps {
  name: string;
  label: string;
  type?: string;
  placeholder?: string;
  required?: boolean;
}

function FormField({ name, label, type = "text", placeholder, required }: FormFieldProps) {
  const {
    register,
    formState: { errors },
  } = useFormContext();

  const error = errors[name];

  return (
    <div className="form-field">
      <label htmlFor={name}>
        {label} {required && <span aria-hidden="true">*</span>}
      </label>
      <input
        id={name}
        type={type}
        placeholder={placeholder}
        aria-invalid={!!error}
        aria-describedby={error ? `${name}-error` : undefined}
        {...register(name, { required: required ? `${label} is required` : false })}
      />
      {error && (
        <span id={`${name}-error`} role="alert" className="form-error">
          {error.message}
        </span>
      )}
    </div>
  );
}
```

**Component Breakdown:**
- `useFormContext()`: Accesses the form state from a parent `<FormProvider>`.
- `register(name, rules)`: Connects the input to React Hook Form.
- `errors[name]`: The validation error for this field, if any.
- `aria-invalid` and `aria-describedby`: Accessibility attributes for error state.

**Reusable Form Wrapper:**
```tsx
import { FormProvider, useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { z } from "zod";

function Form({ schema, onSubmit, children, defaultValues }) {
  const methods = useForm({
    resolver: zodResolver(schema),
    defaultValues,
  });

  return (
    <FormProvider {...methods}>
      <form onSubmit={methods.handleSubmit(onSubmit)}>
        {children}
        <button type="submit" disabled={methods.formState.isSubmitting}>
          {methods.formState.isSubmitting ? "Submitting..." : "Submit"}
        </button>
      </form>
    </FormProvider>
  );
}
```

**Usage:**
```tsx
const loginSchema = z.object({
  email: z.string().email("Invalid email"),
  password: z.string().min(8, "Password must be at least 8 characters"),
});

<Form schema={loginSchema} onSubmit={handleLogin} defaultValues={{ email: "", password: "" }}>
  <FormField name="email" label="Email" type="email" required />
  <FormField name="password" label="Password" type="password" required />
</Form>
```

**Syntax Rules:**
- Use `FormProvider` to make form methods available to nested field components.
- Field components use `useFormContext` instead of `useForm` to avoid creating separate form instances.
- Use `register` for native inputs; use `Controller` for custom components.
- Always provide `aria-invalid` and `aria-describedby` for error accessibility.
- Use schema validation (Zod, Yup) for type-safe, reusable validation rules.

**Constraints and Limitations:**
- `FormProvider` adds a context layer; ensure field components are descendants of the provider.
- Reusable field components may not cover every edge case (e.g., custom inputs, file uploads); provide escape hatches.
- Schema validation adds a dependency but provides type safety and reusability.
- Over-abstracting forms can make them harder to debug; keep the field components simple.

### Annotated Code Example: Reusable Login Form

```tsx
import { FormProvider, useForm, useFormContext } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { z } from "zod";

const loginSchema = z.object({
  email: z.string().email("Invalid email address"),
  password: z.string().min(8, "Password must be at least 8 characters"),
});

function FormField({ name, label, type = "text" }) {
  const { register, formState: { errors } } = useFormContext();
  const error = errors[name];

  return (
    <div style={{ marginBottom: 16 }}>
      <label htmlFor={name} style={{ display: "block" }}>{label}</label>
      <input
        id={name}
        type={type}
        aria-invalid={!!error}
        aria-describedby={error ? `${name}-error` : undefined}
        {...register(name)}
        style={{ borderColor: error ? "red" : "#ccc" }}
      />
      {error && (
        <span id={`${name}-error`} role="alert" style={{ color: "red" }}>
          {error.message}
        </span>
      )}
    </div>
  );
}

function LoginForm({ onSubmit }) {
  const methods = useForm({
    resolver: zodResolver(loginSchema),
    defaultValues: { email: "", password: "" },
  });

  return (
    <FormProvider {...methods}>
      <form onSubmit={methods.handleSubmit(onSubmit)}>
        <FormField name="email" label="Email" type="email" />
        <FormField name="password" label="Password" type="password" />
        <button type="submit" disabled={methods.formState.isSubmitting}>
          {methods.formState.isSubmitting ? "Logging in..." : "Log In"}
        </button>
      </form>
    </FormProvider>
  );
}

export default function App() {
  function handleLogin(data) {
    alert(`Logging in with ${data.email}`);
  }
  return <LoginForm onSubmit={handleLogin} />;
}
```

**Expected Output:** A login form with email and password fields. Submitting with an invalid email shows "Invalid email address" below the email field. Submitting with a short password shows "Password must be at least 8 characters". Submitting with valid data shows an alert.

**Why This Output Occurs:** The `LoginForm` component uses `useForm` with `zodResolver` to validate against the `loginSchema`. The `FormProvider` makes the form methods available to `FormField` components. Each `FormField` registers its input, reads errors, and displays them with accessibility attributes. The form is reusable—the same `FormField` component can be used in any form.

### Real-World Cases

- **Authentication:** Login, signup, password reset forms with shared field components.
- **E-commerce:** Checkout forms with shipping, billing, and payment fields.
- **Admin panels:** CRUD forms for creating and editing records.
- **Surveys:** Dynamic forms with conditional fields and validation.
- **Settings:** User profile and preference forms.

### References

- React Hook Form – Documentation: https://react-hook-form.com/
- React Hook Form – FormProvider: https://react-hook-form.com/docs/formprovider
- React Hook Form – useFormContext: https://react-hook-form.com/docs/useformcontext
- Refine – Essentials of Managing Form State with React Hook Form: https://refine.dev/blog/react-hook-form/

---

## Core Concept 4: Reusable Buttons

### Definitions

**Core Definition:** A reusable button component encapsulates all button styling, states (disabled, loading, active), accessibility, and behaviour in a single configurable component that can be used throughout an application.

**Technical Definition:** A reusable Button component accepts props for `variant` (visual style), `size` (sm, md, lg), `disabled`, `loading`, `onClick`, `type` (button, submit, reset), and `children` (content). It forwards additional props to the underlying `<button>` element via `...rest`. For polymorphic use, it can render as an `<a>` when an `href` prop is provided. The component handles accessibility by ensuring the button has an accessible name (from `children` or `aria-label`), provides a visible focus indicator, and uses `aria-busy` during loading. Variants are typically defined using a styling solution (CSS modules, Tailwind, styled-components) with design tokens for colours, spacing, and typography.

**Beginner-Friendly Explanation:** A reusable button is like a universal remote control. It has all the buttons you need (play, pause, volume) in one device, and you can use it for any TV. You don't need a separate remote for each TV—you just configure it. In React, a reusable `Button` component handles all the visual styles and states, and you just pass different props (variant, size, label) to get the button you need.

### Purposes

- To ensure consistent button styling across the entire application.
- To handle all button states (default, hover, focus, active, disabled, loading) in one place.
- To provide accessibility features (focus indicators, ARIA attributes) by default.
- To support polymorphic rendering (button or link) with the same API.
- To reduce the number of near-duplicate button components.

### Syntax Rules and Structure

**General Syntax:**
```tsx
interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: "primary" | "secondary" | "danger" | "ghost";
  size?: "sm" | "md" | "lg";
  loading?: boolean;
  as?: "button" | "a";
  href?: string;
}

function Button({
  variant = "primary",
  size = "md",
  loading = false,
  disabled = false,
  as = "button",
  href,
  children,
  ...rest
}: ButtonProps) {
  const className = `btn btn-${variant} btn-${size}${loading ? " btn-loading" : ""}`;
  const isDisabled = disabled || loading;

  if (as === "a") {
    return (
      <a className={className} href={href} aria-disabled={isDisabled} {...rest}>
        {loading ? <Spinner /> : children}
      </a>
    );
  }

  return (
    <button className={className} disabled={isDisabled} aria-busy={loading} {...rest}>
      {loading ? <Spinner /> : children}
    </button>
  );
}
```

**Component Breakdown:**
- `variant`, `size`: Control the visual appearance.
- `loading`, `disabled`: Control the interactive state.
- `as`: Determines whether to render a `<button>` or `<a>`.
- `...rest`: Forwards additional props (e.g., `type`, `onClick`, `aria-label`).
- `aria-busy`: Announces the loading state to screen readers.

**Syntax Rules:**
- Use `React.ButtonHTMLAttributes<HTMLButtonElement>` to type the component and allow prop forwarding.
- Provide a default `type="button"` to avoid accidental form submissions.
- Use `aria-busy` for loading states and `aria-disabled` for link-style buttons.
- Ensure the button has an accessible name via `children` or `aria-label`.
- Provide a visible focus indicator (do not remove the outline without a replacement).

**Constraints and Limitations:**
- Rendering as `<a>` requires careful handling of `disabled` (links cannot be disabled natively; use `aria-disabled` and prevent clicks).
- Icon-only buttons require `aria-label` for accessibility.
- The `loading` state should disable the button to prevent double submission.
- Over-variation (too many variants) leads to an inconsistent design; keep variants limited and purposeful.

### Annotated Code Example: Accessible Reusable Button

```tsx
import React from "react";

function Spinner() {
  return <span className="spinner" aria-hidden="true">⟳</span>;
}

function Button({
  variant = "primary",
  size = "md",
  loading = false,
  disabled = false,
  type = "button",
  children,
  ...rest
}) {
  const isDisabled = disabled || loading;

  return (
    <button
      type={type}
      className={`btn btn-${variant} btn-${size}`}
      disabled={isDisabled}
      aria-busy={loading}
      {...rest}
    >
      {loading && <Spinner />}
      {children}
    </button>
  );
}

// Usage
export default function App() {
  return (
    <div>
      <Button>Save</Button>
      <Button variant="secondary" size="lg">Cancel</Button>
      <Button variant="danger" loading>Saving...</Button>
      <Button variant="ghost" disabled>Disabled</Button>
      <Button type="submit" variant="primary">Submit</Button>
    </div>
  );
}
```

**Expected Output:** Five buttons with different variants and states. The loading button shows a spinner and is disabled. The submit button has `type="submit"` for form submission.

**Why This Output Occurs:** The `Button` component accepts `variant`, `size`, `loading`, `disabled`, `type`, and `children`. It applies the appropriate classes, disables the button when loading or disabled, and announces the loading state with `aria-busy`. The same component serves all button use cases.

### Real-World Cases

- **Forms:** Submit, cancel, and reset buttons with loading and disabled states.
- **Dialogs:** Primary and secondary action buttons.
- **Navigation:** Link-styled buttons using the polymorphic `as` prop.
- **Toolbars:** Icon-only buttons with `aria-label`.
- **Design systems:** A single `Button` component with variants and sizes.

### References

- React Official Documentation – Passing Props to a Component: https://react.dev/learn/passing-props-to-a-component
- Smashing Magazine – How to Build Reusable React Components: https://www.smashingmagazine.com/2023/08/building-reusable-react-components/
- Web Content Accessibility Guidelines (WCAG) – Buttons: https://www.w3.org/WAI/ARIA/apg/patterns/button/

---

## Core Concept 5: Reusable Modals

### Definitions

**Core Definition:** A reusable modal component is a dialog component that handles open/close state, focus management, backdrop, escape-key handling, and accessibility, while accepting arbitrary content through slots or children.

**Technical Definition:** A reusable Modal component typically renders a backdrop (an overlay that dims the background) and a container (the modal itself). It manages focus by trapping focus within the modal when open, restoring focus to the trigger element when closed, and closing on Escape key press. It uses `role="dialog"` and `aria-modal="true"` for accessibility, with `aria-labelledby` pointing to the modal's title. The Modal accepts `isOpen`, `onClose`, `title`, `children`, and optionally named slots for `header`, `footer`, and `actions`. It renders nothing when closed (or uses a portal to render outside the parent DOM hierarchy). The `<dialog>` HTML element is increasingly used as a native alternative, but React portals remain the most common approach for custom modals.

**Beginner-Friendly Explanation:** A reusable modal is like a pop-up book. The pop-up (modal) appears on top of the page, and you can close it by clicking outside or pressing Escape. The same pop-up frame can hold different content—a confirmation, a form, a photo. You don't build a new pop-up for every message; you use the same frame and change the contents.

### Purposes

- To provide a consistent dialog experience across the application.
- To handle focus management and keyboard navigation accessibly.
- To accept arbitrary content via slots (header, body, footer).
- To support different sizes and variants (small, medium, large, full-screen).
- To avoid duplicating modal logic across features.

### Syntax Rules and Structure

**General Syntax with Portal and Slots:**
```tsx
import { createPortal } from "react-dom";
import { useEffect, useRef } from "react";

function Modal({
  isOpen,
  onClose,
  title,
  children,
  footer,
  size = "md",
}) {
  const modalRef = useRef(null);

  // Close on Escape
  useEffect(() => {
    function handleKeyDown(e) {
      if (e.key === "Escape") onClose();
    }
    if (isOpen) {
      document.addEventListener("keydown", handleKeyDown);
      return () => document.removeEventListener("keydown", handleKeyDown);
    }
  }, [isOpen, onClose]);

  // Trap focus
  useEffect(() => {
    if (isOpen && modalRef.current) {
      modalRef.current.focus();
    }
  }, [isOpen]);

  if (!isOpen) return null;

  return createPortal(
    <div className="modal-backdrop" onClick={onClose}>
      <div
        ref={modalRef}
        className={`modal modal-${size}`}
        role="dialog"
        aria-modal="true"
        aria-labelledby="modal-title"
        tabIndex={-1}
        onClick={(e) => e.stopPropagation()}
      >
        <header className="modal-header">
          <h2 id="modal-title">{title}</h2>
          <button onClick={onClose} aria-label="Close">×</button>
        </header>
        <div className="modal-body">{children}</div>
        {footer && <footer className="modal-footer">{footer}</footer>}
      </div>
    </div>,
    document.body
  );
}
```

**Component Breakdown:**
- `createPortal`: Renders the modal outside the parent DOM hierarchy.
- `isOpen`, `onClose`: Control visibility and closing.
- `title`, `children`, `footer`: Slots for content.
- `role="dialog"`, `aria-modal="true"`, `aria-labelledby`: Accessibility attributes.
- `tabIndex={-1}`: Allows programmatic focus.

**Syntax Rules:**
- Use `createPortal` to render the modal at the document body level.
- Handle Escape key to close.
- Trap focus within the modal when open (use a focus trap library or implement manually).
- Restore focus to the trigger element when closed.
- Use `role="dialog"`, `aria-modal="true"`, and `aria-labelledby` for accessibility.
- Prevent clicks inside the modal from closing it (`stopPropagation`).
- Provide a close button with `aria-label="Close"`.

**Constraints and Limitations:**
- Focus trapping is complex; use a library (focus-trap-react) for production.
- Portals can complicate event bubbling and context propagation.
- Multiple modals can stack; manage z-index carefully.
- The `<dialog>` element provides native modal behaviour but has inconsistent browser support.

### Annotated Code Example: Confirmation Modal with Slots

```tsx
import { createPortal } from "react-dom";
import { useEffect, useRef, useState } from "react";

function Modal({ isOpen, onClose, title, children, footer }) {
  const ref = useRef(null);

  useEffect(() => {
    if (!isOpen) return;
    const handleKey = (e) => { if (e.key === "Escape") onClose(); };
    document.addEventListener("keydown", handleKey);
    ref.current?.focus();
    return () => document.removeEventListener("keydown", handleKey);
  }, [isOpen, onClose]);

  if (!isOpen) return null;

  return createPortal(
    <div className="backdrop" onClick={onClose}>
      <div
        ref={ref}
        role="dialog"
        aria-modal="true"
        aria-labelledby="modal-title"
        tabIndex={-1}
        className="modal"
        onClick={(e) => e.stopPropagation()}
      >
        <h2 id="modal-title">{title}</h2>
        <div>{children}</div>
        <div className="modal-footer">{footer}</div>
      </div>
    </div>,
    document.body
  );
}

export default function App() {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <div>
      <button onClick={() => setIsOpen(true)}>Delete Item</button>

      <Modal
        isOpen={isOpen}
        onClose={() => setIsOpen(false)}
        title="Confirm Delete"
        footer={
          <>
            <button onClick={() => setIsOpen(false)}>Cancel</button>
            <button onClick={() => { alert("Deleted!"); setIsOpen(false); }}>
              Delete
            </button>
          </>
        }
      >
        <p>Are you sure you want to delete this item? This action cannot be undone.</p>
      </Modal>
    </div>
  );
}
```

**Expected Output:** Clicking "Delete Item" opens a modal with the title "Confirm Delete", a warning message, and "Cancel" and "Delete" buttons. Pressing Escape or clicking the backdrop closes the modal.

**Why This Output Occurs:** The `Modal` component renders via a portal when `isOpen` is true. It handles Escape key and backdrop clicks to close. The `footer` slot receives the action buttons, and `children` receives the body content. The modal is reusable for any confirmation dialog.

### Real-World Cases

- **Confirmation dialogs:** Delete, discard, or irreversible action confirmations.
- **Forms in modals:** Login, signup, and settings forms presented as modals.
- **Image lightboxes:** Full-screen image viewers.
- **Notifications:** Toast and alert dialogs.
- **Multi-step wizards:** Step-by-step processes in a modal.

### References

- React Official Documentation – createPortal: https://react.dev/reference/react-dom/createPortal
- React Official Documentation – Component Composition: https://legacy.reactjs.org/docs/composition-vs-inheritance.html
- WAI-ARIA Authoring Practices – Dialog (Modal) Pattern: https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/

---

## Core Concept 6: Reusable Cards

### Definitions

**Core Definition:** A reusable card component is a flexible container that groups related content (media, title, body, actions) with consistent styling and accepts arbitrary content through slots or children.

**Technical Definition:** A reusable Card component provides a structural container with consistent padding, border, shadow, and border-radius, following design tokens. It accepts `children` for the body and optional named slots for `header`, `media`, `title`, `actions`, and `footer`. Cards are often composed with other reusable components (Button, Badge, Avatar) to build product cards, user cards, and article cards. The Card component is typically unopinionated about its content, allowing consumers to compose it with any subcomponents. Variants (elevated, outlined, filled) control the visual treatment, and sizes control padding and border-radius.

**Beginner-Friendly Explanation:** A reusable card is like a picture frame with a mat. The frame (card) provides a consistent border and background, and the mat (padding) provides spacing. Inside the frame, you can put anything—a photo, a certificate, a drawing. In React, a Card component provides the consistent styling, and you put whatever content you want inside it.

### Purposes

- To provide a consistent container for grouped content.
- To support different card types (product, user, article) with the same base component.
- To allow composition with other reusable components.
- To enforce consistent spacing, borders, and shadows across cards.
- To provide slots for header, media, body, and actions.

### Syntax Rules and Structure

**General Syntax:**
```tsx
interface CardProps {
  variant?: "elevated" | "outlined" | "filled";
  padding?: "sm" | "md" | "lg";
  children: React.ReactNode;
  header?: React.ReactNode;
  media?: React.ReactNode;
  actions?: React.ReactNode;
}

function Card({
  variant = "elevated",
  padding = "md",
  header,
  media,
  actions,
  children,
}: CardProps) {
  return (
    <div className={`card card-${variant} card-pad-${padding}`}>
      {media && <div className="card-media">{media}</div>}
      {header && <div className="card-header">{header}</div>}
      <div className="card-body">{children}</div>
      {actions && <div className="card-actions">{actions}</div>}
    </div>
  );
}
```

**Component Breakdown:**
- `variant`: Controls the visual style (elevated, outlined, filled).
- `padding`: Controls the internal spacing.
- `media`, `header`, `actions`: Optional named slots.
- `children`: The main body content.

**Usage:**
```tsx
<Card
  variant="elevated"
  header={<h3>Product Name</h3>}
  media={<img src="/product.jpg" alt="Product" />}
  actions={<Button>Add to Cart</Button>}
>
  <p>Product description goes here.</p>
</Card>
```

**Syntax Rules:**
- Use named slots (`header`, `media`, `actions`) for structured content.
- Use `children` for the main body.
- Provide variants for different visual treatments.
- Keep the Card presentational; do not add business logic.
- Compose Cards with other reusable components (Button, Badge, Avatar).

**Constraints and Limitations:**
- Too many slots make the Card API complex; keep slots minimal.
- Cards with fixed heights can break with long content; use flexible layouts.
- Nested cards can cause visual clutter; use sparingly.
- The Card component should not manage state; state belongs to the parent.

### Annotated Code Example: Product Card

```tsx
function Card({ variant = "elevated", media, header, actions, children }) {
  return (
    <div className={`card card-${variant}`}>
      {media && <div className="card-media">{media}</div>}
      {header && <div className="card-header">{header}</div>}
      <div className="card-body">{children}</div>
      {actions && <div className="card-actions">{actions}</div>}
    </div>
  );
}

function Button({ children, ...rest }) {
  return <button className="btn" {...rest}>{children}</button>;
}

export default function App() {
  const products = [
    { id: 1, name: "Laptop", price: 999, image: "/laptop.jpg" },
    { id: 2, name: "Phone", price: 699, image: "/phone.jpg" },
  ];

  return (
    <div className="product-grid">
      {products.map((product) => (
        <Card
          key={product.id}
          variant="elevated"
          header={<h3>{product.name}</h3>}
          media={<img src={product.image} alt={product.name} />}
          actions={<Button>Add to Cart — ${product.price}</Button>}
        >
          <p>High-quality {product.name.toLowerCase()} for everyday use.</p>
        </Card>
      ))}
    </div>
  );
}
```

**Expected Output:** A grid of product cards, each with a header (product name), a media area (image), a body (description), and an actions area (Add to Cart button with price).

**Why This Output Occurs:** The `Card` component provides the structural container with slots for header, media, body, and actions. The parent composes each card with product-specific content. The same Card component can be reused for user cards, article cards, or any other content type.

### Real-World Cases

- **E-commerce:** Product cards with image, name, price, and add-to-cart button.
- **Social media:** Post cards with avatar, content, and engagement actions.
- **Blogs:** Article cards with thumbnail, title, excerpt, and read-more link.
- **User profiles:** User cards with avatar, name, role, and contact actions.
- **Dashboards:** Metric cards with title, value, and trend indicator.

### References

- Smashing Magazine – How to Build Reusable React Components: https://www.smashingmagazine.com/2023/08/building-reusable-react-components/
- React Official Documentation – Composition vs Inheritance: https://legacy.reactjs.org/docs/composition-vs-inheritance.html

---

## Core Concept 7: Reusable Navigation Components

### Definitions

**Core Definition:** A reusable navigation component encapsulates navigation links, active state, responsive behaviour, and accessibility, providing a consistent navigation experience across the application.

**Technical Definition:** Reusable navigation components include `Navbar`, `Sidebar`, `Breadcrumbs`, `Tabs`, and `Pagination`. They typically accept an array of navigation items (each with `label`, `href`, `icon`, and optional `children` for nested menus) and render them with consistent styling. Active state is determined by comparing the current route (from `useLocation` in React Router or the router's equivalent) with each item's `href`. Accessibility features include `aria-current="page"` on the active link, keyboard navigation (arrow keys for menus), and proper semantic HTML (`<nav>`, `<ul>`, `<li>`). Responsive behaviour (collapsing into a hamburger menu on mobile) is often handled with CSS or a `useMediaQuery` hook.

**Beginner-Friendly Explanation:** A reusable navigation component is like a road sign system. The signs (links) are consistent in style and placement, and the current location is highlighted so you know where you are. You don't redesign the signs for every road—you use the same sign system everywhere, just with different destinations.

### Purposes

- To provide consistent navigation across all pages.
- To highlight the active page automatically.
- To support nested navigation (submenus, dropdowns).
- To handle responsive behaviour (mobile menu).
- To ensure accessibility (keyboard navigation, ARIA attributes).

### Syntax Rules and Structure

**General Syntax:**
```tsx
import { NavLink } from "react-router";

interface NavItem {
  label: string;
  href: string;
  icon?: React.ReactNode;
  children?: NavItem[];
}

function Navbar({ items }: { items: NavItem[] }) {
  return (
    <nav aria-label="Main navigation">
      <ul className="nav-list">
        {items.map((item) => (
          <li key={item.href}>
            <NavLink
              to={item.href}
              className={({ isActive }) => (isActive ? "nav-link active" : "nav-link")}
              end
            >
              {item.icon}
              {item.label}
            </NavLink>
          </li>
        ))}
      </ul>
    </nav>
  );
}
```

**Component Breakdown:**
- `items`: An array of navigation items.
- `<NavLink>`: Renders an anchor with `aria-current="page"` when active.
- `className={({ isActive }) => ...}`: Applies active styling.
- `aria-label="Main navigation"`: Distinguishes multiple navs for screen readers.

**Syntax Rules:**
- Use `<NavLink>` for automatic active-state detection and `aria-current`.
- Use `aria-label` on `<nav>` elements to distinguish multiple navigation regions.
- Use semantic HTML: `<nav>`, `<ul>`, `<li>`, `<a>`.
- Ensure keyboard navigability (links are focusable by default; custom menus need arrow-key handling).
- Use `end` on `<NavLink>` for exact matching (e.g., home link).

**Constraints and Limitations:**
- Active state matching by prefix can cause multiple links to appear active; use `end` for exact matches.
- Responsive navigation requires additional logic (hamburger menu, media queries).
- Nested menus require careful keyboard handling for accessibility.
- Navigation components should not manage authentication state; they receive items as props.

### Annotated Code Example: Reusable Navbar with Active State

```tsx
import { NavLink } from "react-router";

const navItems = [
  { label: "Home", href: "/" },
  { label: "Products", href: "/products" },
  { label: "About", href: "/about" },
  { label: "Contact", href: "/contact" },
];

function Navbar({ items }) {
  return (
    <nav aria-label="Main navigation">
      <ul className="nav-list">
        {items.map((item) => (
          <li key={item.href}>
            <NavLink
              to={item.href}
              end={item.href === "/"}
              className={({ isActive }) =>
                isActive ? "nav-link active" : "nav-link"
              }
            >
              {item.label}
            </NavLink>
          </li>
        ))}
      </ul>
    </nav>
  );
}

export default function App() {
  return (
    <div>
      <Navbar items={navItems} />
      <main>Page content</main>
    </div>
  );
}
```

**Expected Output:** A navigation bar with Home, Products, About, and Contact links. The current page's link is highlighted. Clicking a link navigates without a page reload.

**Why This Output Occurs:** The `Navbar` component receives an array of items and renders each as a `<NavLink>`. `<NavLink>` automatically applies the active class when the route matches and adds `aria-current="page"`. The `end` prop ensures the Home link is only active on the exact `/` path.

### Real-World Cases

- **Main navigation:** Top navbar with links to primary sections.
- **Sidebar navigation:** Vertical navigation for dashboards and admin panels.
- **Breadcrumbs:** Hierarchical navigation showing the current path.
- **Tabs:** In-page navigation between content sections.
- **Pagination:** Page navigation for lists and tables.
- **Mobile menus:** Hamburger menus with responsive behaviour.

### References

- React Router – NavLink: https://reactrouter.com/api/components/NavLink
- React Router – Link: https://reactrouter.com/api/components/Link
- WAI-ARIA Authoring Practices – Navigation: https://www.w3.org/WAI/ARIA/apg/

---

## Core Concept 8: Design-System-Oriented Components

### Definitions

**Core Definition:** Design-system-oriented components are reusable components built as part of a formal design system, enforcing consistent design tokens, variants, sizes, accessibility, and documentation across an organisation.

**Technical Definition:** Design-system-oriented components extend reusability beyond a single application to an entire organisation. They are built on design tokens (colours, spacing, typography, shadows, border-radius) defined as CSS variables or JavaScript constants. They expose a controlled set of variants and sizes, documented in a component library (Storybook, Styleguidist). They enforce accessibility by default (focus management, ARIA attributes, keyboard navigation). They are versioned and published as a package (npm, private registry) so multiple applications can consume them. Design-system components typically follow the compound component pattern for complex widgets, use TypeScript for type safety, and include visual regression tests, unit tests, and accessibility tests. The goal is consistency: a Button in one application looks and behaves the same as a Button in another application.

**Beginner-Friendly Explanation:** A design system is like a company's brand guidelines. Just as a company has rules about which fonts, colours, and logos to use, a design system has rules about how buttons, forms, and cards should look and behave. Design-system-oriented components are the building blocks that follow these rules. When every team uses these blocks, the entire company's products look and feel consistent—like they were made by one team.

### Purposes

- To enforce visual and behavioural consistency across multiple applications.
- To centralise design decisions (colours, spacing, typography) in design tokens.
- To provide a single source of truth for UI components.
- To reduce duplication of component code across teams.
- To ensure accessibility is built into every component.
- To enable faster development by providing ready-made, tested components.

### Syntax Rules and Structure

**Design Tokens:**
```css
:root {
  --color-primary: #007bff;
  --color-secondary: #6c757d;
  --color-danger: #dc3545;
  --spacing-sm: 4px;
  --spacing-md: 8px;
  --spacing-lg: 16px;
  --radius-sm: 4px;
  --radius-md: 8px;
  --font-size-sm: 14px;
  --font-size-md: 16px;
  --font-size-lg: 20px;
}
```

**Component Using Tokens:**
```tsx
function Button({ variant = "primary", size = "md", children, ...rest }) {
  const style = {
    backgroundColor: `var(--color-${variant})`,
    padding: `var(--spacing-${size === "sm" ? "sm" : size === "lg" ? "lg" : "md"})`,
    borderRadius: "var(--radius-md)",
    fontSize: `var(--font-size-${size})`,
    color: "white",
    border: "none",
    cursor: "pointer",
  };

  return <button style={style} {...rest}>{children}</button>;
}
```

**Component Breakdown:**
- `var(--color-primary)`: References a design token.
- `var(--spacing-md)`: References a spacing token.
- `var(--radius-md)`: References a border-radius token.
- The component is styled entirely with tokens, not hard-coded values.

**Documentation with Storybook:**
```tsx
// Button.stories.tsx
export default {
  title: "Components/Button",
  component: Button,
  argTypes: {
    variant: { control: "select", options: ["primary", "secondary", "danger"] },
    size: { control: "select", options: ["sm", "md", "lg"] },
  },
};

export const Primary = { args: { children: "Primary Button", variant: "primary" } };
export const Secondary = { args: { children: "Secondary Button", variant: "secondary" } };
export const Large = { args: { children: "Large Button", size: "lg" } };
```

**Syntax Rules:**
- Define design tokens as CSS variables or JavaScript constants.
- Use tokens for all colours, spacing, typography, and radii; never hard-code values.
- Expose a limited, purposeful set of variants and sizes.
- Document every component with Storybook or an equivalent tool.
- Include accessibility tests (axe-core) and visual regression tests.
- Publish the design system as a versioned package.
- Use semantic versioning; breaking changes require a major version bump.

**Constraints and Limitations:**
- Design systems require ongoing maintenance and governance.
- Adoption across teams requires communication and training.
- Overly rigid design systems can stifle creativity; provide escape hatches.
- Token naming and organisation require careful planning.
- Version upgrades can be disruptive for consuming applications.

### Annotated Code Example: Design-System Button with Tokens

```tsx
// tokens.css
:root {
  --color-primary: #007bff;
  --color-secondary: #6c757d;
  --color-danger: #dc3545;
  --color-ghost: transparent;
  --spacing-sm: 4px;
  --spacing-md: 8px;
  --spacing-lg: 16px;
  --radius-md: 8px;
  --font-size-sm: 14px;
  --font-size-md: 16px;
  --font-size-lg: 20px;
}

// Button.tsx
function Button({
  variant = "primary",
  size = "md",
  disabled = false,
  children,
  ...rest
}) {
  const style = {
    backgroundColor: `var(--color-${variant})`,
    color: variant === "ghost" ? "var(--color-primary)" : "white",
    padding: `var(--spacing-${size === "sm" ? "sm" : size === "lg" ? "lg" : "md"}) var(--spacing-lg)`,
    borderRadius: "var(--radius-md)",
    fontSize: `var(--font-size-${size})`,
    border: variant === "ghost" ? `1px solid var(--color-primary)` : "none",
    cursor: disabled ? "not-allowed" : "pointer",
    opacity: disabled ? 0.5 : 1,
  };

  return (
    <button style={style} disabled={disabled} {...rest}>
      {children}
    </button>
  );
}

// App.tsx
export default function App() {
  return (
    <div>
      <Button variant="primary">Primary</Button>
      <Button variant="secondary">Secondary</Button>
      <Button variant="danger" size="lg">Danger Large</Button>
      <Button variant="ghost" size="sm">Ghost Small</Button>
      <Button disabled>Disabled</Button>
    </div>
  );
}
```

**Expected Output:** A set of buttons styled with design tokens: primary blue, secondary grey, danger red, ghost transparent with a border, and a disabled button with reduced opacity. Sizes vary (small, medium, large).

**Why This Output Occurs:** The `Button` component references design tokens via CSS variables. Changing `--color-primary` in `tokens.css` updates every primary button across the entire application. The component is configurable (variant, size, disabled) but consistent (same spacing, radius, and typography across all buttons).

### Real-World Cases

- **Enterprise design systems:** Material UI, Ant Design, Chakra UI, and Shopify Polaris.
- **Multi-application organisations:** A shared component library published as an npm package.
- **Brand consistency:** Ensuring all products use the same colours, spacing, and typography.
- **Accessibility compliance:** Building accessibility into every component once, so all applications benefit.
- **Rapid prototyping:** Using a design system to build UIs quickly without designing from scratch.

### References

- Material UI – Documentation: https://mui.com/
- Chakra UI – Documentation: https://chakra-ui.com/
- Shopify Polaris – Documentation: https://polaris.shopify.com/
- Storybook – Documentation: https://storybook.js.org/
- Smashing Magazine – How to Build Reusable React Components: https://www.smashingmagazine.com/2023/08/building-reusable-react-components/

---

## Comparison and Decision Guidance

| Component Type | Key Props | Composition Pattern | Best For |
|---|---|---|---|
| **Configurable** | `variant`, `size`, `disabled`, `onClick` | Props + defaults | Buttons, inputs, badges |
| **Generic UI** | `items`, `renderItem`, `keyExtractor`, `as` | Render props / generics | Lists, tables, grids |
| **Forms** | `name`, `label`, `error`, `register` | FormProvider + context | Login, signup, checkout |
| **Buttons** | `variant`, `size`, `loading`, `as` | Props + `...rest` | All actions |
| **Modals** | `isOpen`, `onClose`, `title`, `footer` | Portal + slots | Dialogs, confirmations |
| **Cards** | `variant`, `media`, `header`, `actions` | Slots + children | Products, articles, users |
| **Navigation** | `items`, `activeClass`, `end` | NavLink + arrays | Navbars, sidebars, tabs |
| **Design System** | Tokens, variants, a11y | Tokens + Storybook | Cross-team consistency |

**Decision Guidance:**
- **Start with configurable components** for all UI primitives; they cover most use cases.
- **Use generic components** when the component's structure is independent of the data type.
- **Use reusable forms** with React Hook Form and schema validation for all forms.
- **Use a single Button component** with variants for all buttons; never create `PrimaryButton`, `SecondaryButton`, etc.
- **Use a reusable Modal** with slots for all dialogs; never duplicate modal logic.
- **Use a reusable Card** with slots for all card-like content.
- **Use reusable Navigation components** with `NavLink` for automatic active state.
- **Build a design system** when multiple applications or teams need consistent UI.
- **Avoid over-abstraction:** if a component is used only once, it may not need to be reusable.

---

## References

- React Official Documentation – Passing Props to a Component: https://react.dev/learn/passing-props-to-a-component
- React Official Documentation – Composition vs Inheritance: https://legacy.reactjs.org/docs/composition-vs-inheritance.html
- React Official Documentation – createPortal: https://react.dev/reference/react-dom/createPortal
- React Official Documentation – Thinking in React: https://react.dev/learn/thinking-in-react
- React TypeScript Cheatsheet – Generic Components: https://react-typescript-cheatsheet.netlify.app/docs/advanced/patterns_by_usecase/#generic-components
- React TypeScript Cheatsheet – Polymorphic Components: https://react-typescript-cheatsheet.netlify.app/docs/advanced/patterns_by_usecase/#polymorphic-components
- React Hook Form – Documentation: https://react-hook-form.com/
- React Hook Form – FormProvider: https://react-hook-form.com/docs/formprovider
- React Hook Form – useFormContext: https://react-hook-form.com/docs/useformcontext
- React Router – NavLink: https://reactrouter.com/api/components/NavLink
- React Router – Link: https://reactrouter.com/api/components/Link
- Smashing Magazine – How to Build Reusable React Components: https://www.smashingmagazine.com/2023/08/building-reusable-react-components/
- Refine – Essentials of Managing Form State with React Hook Form: https://refine.dev/blog/react-hook-form/
- WAI-ARIA Authoring Practices – Dialog (Modal) Pattern: https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/
- WAI-ARIA Authoring Practices – Button Pattern: https://www.w3.org/WAI/ARIA/apg/patterns/button/
- Material UI – Documentation: https://mui.com/
- Chakra UI – Documentation: https://chakra-ui.com/
- Shopify Polaris – Documentation: https://polaris.shopify.com/
- Storybook – Documentation: https://storybook.js.org/