# Advanced React TypeScript: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Advanced React TypeScript is the practice of applying TypeScript's most powerful type-system features—generics, discriminated unions, polymorphic types, utility types, and conditional types—to build reusable, type-safe React components, hooks, and API clients that scale across large applications.

**Technical Definition:** Advanced React TypeScript extends foundational component typing with type-level programming techniques. It covers **generic components** (`<List<T> />`) that preserve type information through render props, **discriminated unions** that model mutually exclusive prop combinations as type-safe state machines, **polymorphic components** with an `as` prop that dynamically change their rendered element while preserving correct prop types, **type-preserving wrappers** for `React.forwardRef` and `React.memo` that do not lose generics, **utility types** (`Pick`, `Omit`, `Partial`, `ComponentPropsWithoutRef`, `ComponentPropsWithRef`) that derive new types from existing ones, **custom hook typing** with strict parameter and return types (using `as const` for tuple returns), and **generic API response types** that integrate typed fetchers and Axios clients with backend data models. These patterns rely on TypeScript's structural type system, conditional types, mapped types, and `infer` keyword.

**Beginner-Friendly Explanation:** Basic TypeScript in React lets you type props and state. Advanced TypeScript lets you build components that adapt to any data type, enforce that exactly one of two props is provided, change which HTML element they render while keeping correct props, and wrap other components without losing their types. These are the patterns that make large component libraries and design systems work—components that are reusable across dozens of use cases without sacrificing type safety.

### Key Characteristics

- **Type-Level Programming:** Types themselves become parameters, conditions, and transformations—not just labels.
- **Inference Preservation:** The goal is to preserve TypeScript's inference through wrappers, generics, and polymorphic components.
- **Exhaustive Safety:** Discriminated unions and `never` checks ensure all cases are handled at compile time.
- **Composition Over Duplication:** Utility types derive new types from existing ones, avoiding duplication.
- **Single Source of Truth:** A single type (schema, API model, component props) can generate multiple derived types.
- **Erased at Runtime:** All advanced types disappear at compile time; they do not add runtime cost.
- **Library-Oriented:** These patterns are essential for design systems, component libraries, and shared packages.

### Prerequisites

- Solid understanding of React function components, JSX, props, and Hooks.
- Working knowledge of TypeScript fundamentals (types, interfaces, unions, generics, narrowing).
- Familiarity with the React TypeScript patterns (props, events, refs, context, state).
- Understanding of `React.forwardRef`, `React.memo`, and `React.ComponentProps`.
- Awareness of TypeScript's conditional types, mapped types, and `infer` keyword.

### Related Programming Areas

- **Design Systems:** Reusable components with strict prop APIs.
- **Component Libraries:** Public APIs that must be type-safe and backward-compatible.
- **API Clients:** Typed fetchers, Axios wrappers, and code generation.
- **State Management:** Typed reducers, stores, and context.
- **Testing:** Type-safe mocks and test utilities.

### Core Concepts / Features

1. Generic Components (`<List<T> />`)
2. Discriminated Unions (Exclusive Props)
3. Polymorphic Components (`as` Prop)
4. Component Forwarding & Memos (Typing `forwardRef` and `memo`)
5. Utility Types (`Pick`, `Omit`, `Partial`, `ComponentPropsWithoutRef`)
6. Custom Hook Typing (Parameters, Return Values, `as const`)
7. API Response Types (Generic Fetchers and Axios)

---

## Core Concept 1: Generic Components

### Definitions

**Core Definition:** A generic component is a React component that accepts a type parameter (`<T>`), allowing it to work with any data type while preserving full type safety for the consumer.

**Technical Definition:** A generic component is declared as `function Component<T>(props: Props<T>)` or `const Component = <T,>(props: Props<T>) => ...`. The type parameter `T` is inferred from the props (typically from an `items` array or `renderItem` function), then propagated through the rest of the props type. TypeScript infers `T` at the call site, so consumers rarely need to specify it explicitly. Generic components are essential for reusable lists, tables, selects, and data-driven widgets. Because `T` is erased at runtime, the component behaves identically to a non-generic component—the benefit is entirely at the type level. Generic components can be constrained (`<T extends object>`), defaulted (`<T = string>`), and combined with other generics (`<T, K extends keyof T>`).

**Beginner-Friendly Explanation:** A generic component is like a cookie cutter that works with any dough. A list component that renders users and a list component that renders products are the same component—only the type of items changes. Instead of writing `UserList`, `ProductList`, and `OrderList`, you write one `List<T>` and let TypeScript figure out what `T` is from the items you pass in. The result is less code, better type safety, and consistent behaviour.

### Purposes

- To write one component that works with many data types.
- To preserve type information from `items` through `renderItem`.
- To avoid duplicating list, table, and select components for each data type.
- To enable TypeScript autocomplete on `renderItem` parameters.
- To enforce type safety across reusable component APIs.
- To build design systems where data-driven components are truly generic.

### Syntax Rules and Structure

**General Syntax (Function Declaration):**
```tsx
type ListProps<T> = {
  items: T[];
  renderItem: (item: T) => React.ReactNode;
  keyExtractor: (item: T) => string | number;
};

function List<T>({ items, renderItem, keyExtractor }: ListProps<T>) {
  return (
    <ul>
      {items.map((item) => (
        <li key={keyExtractor(item)}>{renderItem(item)}</li>
      ))}
    </ul>
  );
}
```

**Component Breakdown:**
- `<T>`: Declares the generic type parameter.
- `ListProps<T>`: Uses `T` in the props type.
- `items: T[]`: The array determines `T` at the call site.
- `renderItem: (item: T) => ReactNode`: Preserves `T` in the callback.
- `keyExtractor: (item: T) => string | number`: Same.

**General Syntax (Arrow Function):**
```tsx
const List = <T,>({ items, renderItem }: ListProps<T>) => {
  return <ul>{items.map((item, i) => <li key={i}>{renderItem(item)}</li>)}</ul>;
};
```

**Component Breakdown:**
- `<T,>`: The trailing comma disambiguates from JSX in `.tsx` files.
- Arrow functions require this syntax; otherwise TypeScript parses `<T>` as a JSX element.

**Generic with Constraint:**
```tsx
type TableProps<T extends { id: number | string }> = {
  rows: T[];
  columns: Array<{ key: keyof T; header: string; render?: (row: T) => React.ReactNode }>;
};

function Table<T extends { id: number | string }>({ rows, columns }: TableProps<T>) {
  return (
    <table>
      <thead>
        <tr>{columns.map((col) => <th key={String(col.key)}>{col.header}</th>)}</tr>
      </thead>
      <tbody>
        {rows.map((row) => (
          <tr key={row.id}>
            {columns.map((col) => (
              <td key={String(col.key)}>
                {col.render ? col.render(row) : String(row[col.key])}
              </td>
            ))}
          </tr>
        ))}
      </tbody>
    </table>
  );
}
```

**Component Breakdown:**
- `<T extends { id: number | string }>`: Constrains `T` to have an `id` property.
- `keyof T`: Restricts column keys to the keys of `T`.
- `row[col.key]`: Indexed access with type safety.

**Generic Component with `renderItem` and Inference:**
```tsx
type SelectProps<T> = {
  options: T[];
  value: T | null;
  onChange: (value: T) => void;
  getLabel: (option: T) => string;
  getValue: (option: T) => string | number;
};

function Select<T>({ options, value, onChange, getLabel, getValue }: SelectProps<T>) {
  return (
    <select
      value={value ? String(getValue(value)) : ''}
      onChange={(e) => {
        const selected = options.find((o) => String(getValue(o)) === e.target.value);
        if (selected) onChange(selected);
      }}
    >
      <option value="">Select...</option>
      {options.map((option) => (
        <option key={getValue(option)} value={String(getValue(option))}>
          {getLabel(option)}
        </option>
      ))}
    </select>
  );
}

// Usage — T is inferred as User
<Select<User>
  options={users}
  value={selectedUser}
  onChange={setSelectedUser}
  getLabel={(u) => u.name}
  getValue={(u) => u.id}
/>
```

**Component Breakdown:**
- `T` is inferred from `options`.
- All other props are typed in terms of `T`.
- `onChange` receives a `User`; `getValue` returns the user's ID type.

**Syntax Rules:**
- Declare the type parameter immediately after the function name: `function Component<T>(props: Props<T>)`.
- For arrow functions, use `<T,>` (trailing comma) to avoid JSX ambiguity.
- Let TypeScript infer `T` from props; specify explicitly only when inference fails.
- Use `extends` to constrain `T` when the component requires certain properties.
- Use `keyof T` for keys, `T[K]` for indexed access, and utility types for derived props.
- Generic components cannot be passed directly to `React.memo` while preserving the generic; use the workaround in Concept 4.
- Export the generic type (`ListProps<T>`) so consumers can reference it.

**Constraints and Limitations:**
- TypeScript cannot infer `T` if it appears only in the return type or a callback without a corresponding value.
- Generic components with `React.memo` require a type-preserving wrapper.
- Generic arrow components in `.tsx` need the trailing comma.
- Overly constrained generics reduce flexibility; use the loosest constraint that works.
- Generic components do not exist at runtime; there is no way to introspect `T`.

### Annotated Code Examples

**Example 1: Generic List Component**

```tsx
type ListProps<T> = {
  items: T[];
  renderItem: (item: T, index: number) => React.ReactNode;
  emptyMessage?: string;
  keyExtractor: (item: T) => string | number;
};

export function List<T>({
  items,
  renderItem,
  emptyMessage = 'No items',
  keyExtractor,
}: ListProps<T>) {
  if (items.length === 0) return <p>{emptyMessage}</p>;
  return (
    <ul>
      {items.map((item, index) => (
        <li key={keyExtractor(item)}>{renderItem(item, index)}</li>
      ))}
    </ul>
  );
}

// Usage with strings
<List
  items={['Apple', 'Banana', 'Cherry']}
  renderItem={(fruit) => <strong>{fruit.toUpperCase()}</strong>}
  keyExtractor={(fruit) => fruit}
/>

// Usage with objects
type User = { id: number; name: string };
<List<User>
  items={[{ id: 1, name: 'Alice' }, { id: 2, name: 'Bob' }]}
  renderItem={(user) => <span>{user.name}</span>}
  keyExtractor={(user) => user.id}
/>
```

**Expected Output:** The `List` component renders a list of strings in the first case and a list of users in the second. TypeScript infers `T = string` for the first and `T = User` for the second, so `fruit.toUpperCase()` and `user.name` are type-safe. Passing an invalid property (e.g., `user.email`) errors at compile time.

**Why This Output Occurs:** `<T>` declares the generic. `items: T[]` makes the component generic. TypeScript infers `T` from `items` and applies it to `renderItem` and `keyExtractor`. The `keyExtractor` provides a stable key from the data.

**Example 2: Generic Table with Column Definitions**

```tsx
type Column<T> = {
  key: keyof T;
  header: string;
  render?: (row: T) => React.ReactNode;
};

type TableProps<T extends { id: number }> = {
  rows: T[];
  columns: Column<T>[];
  onRowClick?: (row: T) => void;
};

export function Table<T extends { id: number }>({ rows, columns, onRowClick }: TableProps<T>) {
  return (
    <table>
      <thead>
        <tr>
          {columns.map((col) => (
            <th key={String(col.key)}>{col.header}</th>
          ))}
        </tr>
      </thead>
      <tbody>
        {rows.map((row) => (
          <tr key={row.id} onClick={() => onRowClick?.(row)}>
            {columns.map((col) => (
              <td key={String(col.key)}>
                {col.render ? col.render(row) : String(row[col.key])}
              </td>
            ))}
          </tr>
        ))}
      </tbody>
    </table>
  );
}

// Usage
type Product = { id: number; name: string; price: number; inStock: boolean };

<Table<Product>
  rows={products}
  columns={[
    { key: 'name', header: 'Name' },
    { key: 'price', header: 'Price', render: (p) => `$${p.price.toFixed(2)}` },
    { key: 'inStock', header: 'Status', render: (p) => (p.inStock ? 'In Stock' : 'Out') },
  ]}
  onRowClick={(p) => console.log(p.name)}
/>
```

**Expected Output:** A table with columns defined by the `columns` prop. TypeScript enforces that `key` is a valid key of `Product`, that `render` receives a `Product`, and that `onRowClick` receives a `Product`. Passing `{ key: 'email' }` errors because `email` is not a key of `Product`.

**Why This Output Occurs:** `T extends { id: number }` constrains the row type. `keyof T` restricts column keys to the keys of `T`. The `render` function receives the correctly typed row.

### Real-World Cases

- **Data tables:** Generic table components with typed columns and rows.
- **Select/combobox:** Generic options with typed values and labels.
- **Lists and grids:** Generic rendering with typed item props.
- **Form field groups:** Generic field arrays with typed values.
- **Pagination:** Generic paginated lists with typed items.
- **API hooks:** `useFetch<T>` for typed API responses.

### References

- TypeScript Handbook – Generics - https://www.typescriptlang.org/docs/handbook/2/generics.html
- React TypeScript Cheatsheet – Generic Components - https://react-typescript-cheatsheet.netlify.app/docs/advanced/patterns_by_usecase/#generic-components
- Matt Pocock – Generic Components - https://www.totaltypescript.com/react-generic-components
- TypeScript Handbook – `keyof` Type Operator - https://www.typescriptlang.org/docs/handbook/2/keyof-types.html
- TypeScript Handbook – Indexed Access Types - https://www.typescriptlang.org/docs/handbook/2/indexed-access-types.html

---

## Core Concept 2: Discriminated Unions (Exclusive Props)

### Definitions

**Core Definition:** A discriminated union for props is a TypeScript pattern where a component's props are modelled as a union of variants, each with a literal discriminant property (e.g., `variant: 'primary' | 'secondary'`) that determines which other props are required.

**Technical Definition:** A discriminated union is a union of object types that share a common literal property (the discriminant). In React, this is used to model components whose props change shape based on a mode or variant. For example, a `Modal` component might have `{ mode: 'create' }` (no initial values) and `{ mode: 'edit', initialValues: FormValues }` (initial values required). TypeScript narrows the union based on the discriminant, so consumers cannot pass `initialValues` with `mode: 'create'`, and cannot omit it with `mode: 'edit'`. This pattern is sometimes called "exclusive props" or "conditional props." It replaces runtime checks (`if (mode === 'edit' && !initialValues) throw ...`) with compile-time guarantees.

**Beginner-Friendly Explanation:** Imagine a form component that can be in "create" mode or "edit" mode. In "create" mode, it does not need an initial value. In "edit" mode, it does. Without a discriminated union, both props would be optional, and you would have to check at runtime. With a discriminated union, TypeScript enforces the rule: if `mode` is `'edit'`, `initialValues` is required; if `mode` is `'create'`, it must not be passed. The type system catches mistakes before the code runs.

### Purposes

- To enforce that exactly one of several prop combinations is provided.
- To make invalid prop combinations impossible at compile time.
- To replace runtime validation with type-level guarantees.
- To model components with distinct modes (create/edit, controlled/uncontrolled).
- To enable exhaustive handling of component variants.
- To document the component's valid configurations clearly.

### Syntax Rules and Structure

**General Syntax:**
```tsx
type ComponentProps =
  | { variant: 'primary'; primaryColor: string }
  | { variant: 'secondary'; secondaryColor: string }
  | { variant: 'ghost' };
```

**Component Breakdown:**
- Each union member has a discriminant property (`variant`) with a literal value.
- Other props are unique to each variant.
- TypeScript narrows based on the discriminant.

**Create/Edit Modal:**
```tsx
type ModalProps =
  | { mode: 'create'; onSubmit: (values: FormValues) => void }
  | { mode: 'edit'; initialValues: FormValues; onSubmit: (values: FormValues) => void };

function Modal(props: ModalProps) {
  if (props.mode === 'create') {
    return <CreateForm onSubmit={props.onSubmit} />;
  }
  return <EditForm initialValues={props.initialValues} onSubmit={props.onSubmit} />;
}

// Usage
<Modal mode="create" onSubmit={handleCreate} />
<Modal mode="edit" initialValues={existing} onSubmit={handleUpdate} />
// <Modal mode="edit" onSubmit={handleUpdate} /> // Error: initialValues required
// <Modal mode="create" initialValues={existing} onSubmit={handleCreate} /> // Error: initialValues not allowed
```

**Component Breakdown:**
- `mode` is the discriminant.
- In `create` mode, `initialValues` is not allowed.
- In `edit` mode, `initialValues` is required.
- TypeScript narrows `props` in each branch.

**Controlled vs. Uncontrolled Input:**
```tsx
type InputProps =
  | { value: string; onChange: (v: string) => void; defaultValue?: never }
  | { defaultValue?: string; value?: never; onChange?: never };

function Input(props: InputProps) {
  if ('value' in props) {
    return <input value={props.value} onChange={(e) => props.onChange(e.target.value)} />;
  }
  return <input defaultValue={props.defaultValue} />;
}
```

**Component Breakdown:**
- `never` disables the other prop in each variant.
- Controlled: `value` and `onChange` required; `defaultValue` forbidden.
- Uncontrolled: `defaultValue` optional; `value` and `onChange` forbidden.

**Discriminated Union with Exhaustive Check:**
```tsx
type ButtonProps =
  | { variant: 'link'; href: string; onClick?: never }
  | { variant: 'button'; href?: never; onClick: () => void };

function Button(props: ButtonProps) {
  switch (props.variant) {
    case 'link':
      return <a href={props.href}>Link</a>;
    case 'button':
      return <button onClick={props.onClick}>Button</button>;
    default: {
      const _exhaustive: never = props;
      return null;
    }
  }
}
```

**Component Breakdown:**
- `link` variant requires `href` and forbids `onClick`.
- `button` variant requires `onClick` and forbids `href`.
- `default: never` ensures all variants are handled.

**Syntax Rules:**
- Use a single literal property as the discriminant (`variant`, `mode`, `type`, `kind`).
- Use `never` to forbid props in specific variants.
- Narrow with `switch` or `if` on the discriminant.
- Add a `default: never` case to catch unhandled variants.
- Do not use optional props (`?`) to model variants; use discriminated unions instead.
- Export the union type so consumers can reference each variant.
- Keep discriminants semantically meaningful (e.g., `mode` not `type` if `type` is used for HTML).

**Constraints and Limitations:**
- Discriminated unions do not work with optional discriminants; the discriminant must be a literal type.
- Excess property checks apply to object literals; passing a variable with extra properties is allowed.
- TypeScript cannot narrow if the discriminant is destructured too early; narrow on `props.mode` or keep the discriminant accessible.
- Adding a new variant requires updating all `switch` statements; the `default: never` catches this.
- Discriminated unions are not supported in React's `defaultProps` (function components use default parameters).

### Annotated Code Examples

**Example 1: Create/Edit Form Modal**

```tsx
type FormValues = { name: string; email: string };

type ModalProps =
  | {
      mode: 'create';
      onSubmit: (values: FormValues) => void;
    }
  | {
      mode: 'edit';
      initialValues: FormValues;
      onSubmit: (values: FormValues) => void;
    };

function Modal(props: ModalProps) {
  const [values, setValues] = React.useState<FormValues>(
    props.mode === 'edit' ? props.initialValues : { name: '', email: '' }
  );

  function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    props.onSubmit(values);
  }

  return (
    <form onSubmit={handleSubmit}>
      <h2>{props.mode === 'create' ? 'Create User' : 'Edit User'}</h2>
      <input
        value={values.name}
        onChange={(e) => setValues({ ...values, name: e.target.value })}
        placeholder="Name"
      />
      <input
        value={values.email}
        onChange={(e) => setValues({ ...values, email: e.target.value })}
        placeholder="Email"
      />
      <button type="submit">{props.mode === 'create' ? 'Create' : 'Save'}</button>
    </form>
  );
}

// Usage
<Modal mode="create" onSubmit={(values) => createUser(values)} />
<Modal mode="edit" initialValues={{ name: 'Alice', email: 'a@x.com' }} onSubmit={(values) => updateUser(values)} />
// <Modal mode="edit" onSubmit={...} /> // Error: initialValues required
// <Modal mode="create" initialValues={...} onSubmit={...} /> // Error: initialValues not allowed
```

**Expected Output:** The modal renders a create form (empty fields) or an edit form (pre-filled). TypeScript enforces that `initialValues` is provided only in edit mode. Consumers cannot accidentally omit it or pass it in create mode.

**Why This Output Occurs:** The discriminant `mode` is a literal type. TypeScript narrows `props` in each branch, so `props.initialValues` is available only when `mode === 'edit'`. The union prevents invalid combinations.

**Example 2: Controlled vs. Uncontrolled Input**

```tsx
type InputProps =
  | {
      value: string;
      onChange: (value: string) => void;
      defaultValue?: never;
    }
  | {
      defaultValue?: string;
      value?: never;
      onChange?: never;
    };

function Input(props: InputProps) {
  if ('value' in props) {
    return (
      <input
        value={props.value}
        onChange={(e) => props.onChange(e.target.value)}
      />
    );
  }
  return <input defaultValue={props.defaultValue} />;
}

// Controlled
<Input value={name} onChange={setName} />
// Uncontrolled
<Input defaultValue="initial" />
// <Input value={name} /> // Error: onChange required with value
// <Input value={name} defaultValue="x" /> // Error: defaultValue not allowed
```

**Expected Output:** The `Input` component is either controlled (value + onChange) or uncontrolled (defaultValue), never both. TypeScript enforces the exclusivity.

**Why This Output Occurs:** The `never` type forbids the other variant's props. `'value' in props` narrows the union, so the correct branch is chosen.

### Real-World Cases

- **Modal dialogs:** Create vs. edit vs. view modes.
- **Form inputs:** Controlled vs. uncontrolled.
- **Buttons:** Link vs. button vs. submit.
- **Alerts:** Dismissible vs. persistent.
- **Cards:** Clickable vs. static.
- **Select components:** Single vs. multiple selection.

### References

- TypeScript Handbook – Discriminated Unions - https://www.typescriptlang.org/docs/handbook/2/narrowing.html#discriminated-unions
- TypeScript Handbook – Exhaustiveness Checking - https://www.typescriptlang.org/docs/handbook/2/narrowing.html#exhaustiveness-checking
- React TypeScript Cheatsheet – Discriminated Unions - https://react-typescript-cheatsheet.netlify.app/docs/advanced/patterns_by_usecase/#discriminated-unions
- Matt Pocock – Discriminated Unions - https://www.totaltypescript.com/discriminated-unions
- Kent C. Dodds – Discriminated Unions for Component Props - https://kentcdodds.com/blog/discriminated-unions-for-component-props

---

## Core Concept 3: Polymorphic Components

### Definitions

**Core Definition:** A polymorphic component is a React component that accepts an `as` prop (or similar) that changes which HTML element or React component it renders, while preserving the correct prop types for the chosen element.

**Technical Definition:** A polymorphic component uses a generic type parameter constrained to `React.ElementType` (`<T extends React.ElementType>`), and its props are the intersection of the component's own props and `React.ComponentPropsWithoutRef<T>`. The component renders as `const Component = as || 'div'`, forwarding all remaining props to the chosen element. This preserves TypeScript's inference: if `as="a"`, the component accepts `href`; if `as="button"`, it accepts `type` and `disabled`; if `as={Link}`, it accepts the `Link` component's props. Polymorphic components are common in design systems (Box, Text, Heading) where a single component renders as different elements for semantic correctness. The `as` prop is a declarative way to change the underlying element without duplicating the component.

**Beginner-Friendly Explanation:** A polymorphic component is like a shape-shifter. You have a `Text` component that renders a paragraph by default, but you can tell it to render as an `<h1>`, a `<span>`, or a `<label>` by passing `as="h1"`. The best part: TypeScript adapts the props. If you say `as="a"`, TypeScript expects an `href`; if you say `as="button"`, it expects `onClick`. You get one component that adapts to any element, with full type safety.

### Purposes

- To build components that render as different HTML elements while preserving semantics.
- To avoid duplicating components for every element type.
- To preserve the correct prop types for the chosen element.
- To support custom React components (e.g., `as={Link}` from React Router).
- To enable accessibility-correct rendering (e.g., `<h1>` for headings, `<button>` for buttons).
- To build design system primitives (Box, Text, Heading, Stack).

### Syntax Rules and Structure

**Basic Polymorphic Component:**
```tsx
type BoxProps<T extends React.ElementType = 'div'> = {
  as?: T;
  children?: React.ReactNode;
} & Omit<React.ComponentPropsWithoutRef<T>, 'as' | 'children'>;

function Box<T extends React.ElementType = 'div'>({
  as,
  children,
  ...rest
}: BoxProps<T>) {
  const Component = as || 'div';
  return <Component {...rest}>{children}</Component>;
}

// Usage
<Box>Default div</Box>
<Box as="section" className="section">Section</Box>
<Box as="a" href="/about">Link</Box>
<Box as="button" onClick={handleClick}>Button</Box>
// <Box as="a" onClick={handleClick}>Link</Box> // OK: onClick is valid on <a>
// <Box as="button" href="/about">Button</Box> // Error: href is not valid on <button>
```

**Component Breakdown:**
- `T extends React.ElementType = 'div'`: The generic is constrained to valid element types, defaulting to `'div'`.
- `as?: T`: The optional `as` prop.
- `Omit<React.ComponentPropsWithoutRef<T>, 'as' | 'children'>`: The chosen element's props, minus `as` and `children`.
- `const Component = as || 'div'`: Resolves the element type.
- `<Component {...rest}>`: Spreads the remaining props.

**Polymorphic Component with `forwardRef`:**
```tsx
type TextProps<T extends React.ElementType = 'span'> = {
  as?: T;
  children?: React.ReactNode;
} & Omit<React.ComponentPropsWithoutRef<T>, 'as' | 'children'>;

type TextComponent = <T extends React.ElementType = 'span'>(
  props: TextProps<T> & { ref?: React.Ref<React.ElementRef<T>> }
) => React.ReactElement | null;

const Text = React.forwardRef(function Text<T extends React.ElementType = 'span'>(
  { as, children, ...rest }: TextProps<T>,
  ref: React.Ref<React.ElementRef<T>>
) {
  const Component = as || 'span';
  return <Component ref={ref} {...rest}>{children}</Component>;
}) as TextComponent;
```

**Component Breakdown:**
- `TextComponent`: A manually typed function component that preserves generics through `forwardRef`.
- `React.forwardRef` with the generic function.
- The `as TextComponent` cast restores the generic signature.

**Polymorphic Component with Custom Component:**
```tsx
import { Link } from 'react-router-dom';

type ButtonLikeProps<T extends React.ElementType = 'button'> = {
  as?: T;
  variant?: 'primary' | 'secondary';
  children: React.ReactNode;
} & Omit<React.ComponentPropsWithoutRef<T>, 'as' | 'children'>;

function ButtonLike<T extends React.ElementType = 'button'>({
  as,
  variant = 'primary',
  children,
  ...rest
}: ButtonLikeProps<T>) {
  const Component = as || 'button';
  return <Component className={`btn-${variant}`} {...rest}>{children}</Component>;
}

// Usage
<ButtonLike>Button</ButtonLike>
<ButtonLike as="a" href="/checkout">Link</ButtonLike>
<ButtonLike as={Link} to="/checkout">Router Link</ButtonLike>
// <ButtonLike as={Link} href="/checkout">Link</ButtonLike> // Error: Link expects `to`, not `href`
```

**Component Breakdown:**
- `as={Link}`: The component renders as the React Router `Link`, and TypeScript expects `to` instead of `href`.
- This works because `React.ComponentPropsWithoutRef<T>` extracts the `Link` component's props.

**Syntax Rules:**
- Constrain the generic to `React.ElementType`: `<T extends React.ElementType = 'div'>`.
- Use `Omit<ComponentPropsWithoutRef<T>, 'as' | 'children'>` to avoid conflicts.
- Resolve the element with `const Component = as || 'div'`.
- Spread `...rest` onto the resolved component.
- For `forwardRef`, manually type the component and cast to preserve generics.
- Export the `Props` type for consumers.
- Use semantic defaults (e.g., `'div'` for Box, `'span'` for Text, `'button'` for Button).
- Avoid `any` in polymorphic components; use proper generics.

**Constraints and Limitations:**
- TypeScript cannot infer `T` if `as` is not provided; the default is used.
- Polymorphic components with `forwardRef` require a manual type cast to preserve generics.
- The `Omit` is necessary to prevent conflicts between `as` and the element's props.
- Excess property checks do not apply when props are spread from a variable.
- Polymorphic components can be harder to read; document the API clearly.
- Ref forwarding requires `React.ElementRef<T>` and `React.Ref<T>` types.

### Annotated Code Examples

**Example 1: Polymorphic Text Component**

```tsx
type TextProps<T extends React.ElementType = 'span'> = {
  as?: T;
  size?: 'sm' | 'md' | 'lg';
  weight?: 'normal' | 'bold';
  children?: React.ReactNode;
} & Omit<React.ComponentPropsWithoutRef<T>, 'as' | 'children'>;

function Text<T extends React.ElementType = 'span'>({
  as,
  size = 'md',
  weight = 'normal',
  children,
  ...rest
}: TextProps<T>) {
  const Component = as || 'span';
  const className = `text-${size} text-${weight}`;
  return <Component className={className} {...rest}>{children}</Component>;
}

// Usage
<Text>Default span</Text>
<Text as="p">Paragraph</Text>
<Text as="h1" size="lg" weight="bold">Heading</Text>
<Text as="label" htmlFor="email">Email</Text>
<Text as="a" href="/about">About</Text>
// <Text as="label" href="/about">Email</Text> // Error: href is not valid on <label>
```

**Expected Output:** The `Text` component renders as `<span>`, `<p>`, `<h1>`, `<label>`, or `<a>` depending on the `as` prop. TypeScript enforces the correct props for each element: `htmlFor` for `<label>`, `href` for `<a>`. Passing `href` to a `<label>` errors.

**Why This Output Occurs:** `React.ComponentPropsWithoutRef<T>` extracts the props of the chosen element. The `Omit` removes `as` and `children` to avoid conflicts. TypeScript narrows `T` based on the `as` prop.

**Example 2: Polymorphic Box with `forwardRef`**

```tsx
type BoxProps<T extends React.ElementType = 'div'> = {
  as?: T;
  children?: React.ReactNode;
  padding?: 'sm' | 'md' | 'lg';
} & Omit<React.ComponentPropsWithoutRef<T>, 'as' | 'children'>;

type BoxComponent = <T extends React.ElementType = 'div'>(
  props: BoxProps<T> & { ref?: React.Ref<React.ElementRef<T>> }
) => React.ReactElement | null;

const Box = React.forwardRef(function Box<T extends React.ElementType = 'div'>(
  { as, padding = 'md', children, ...rest }: BoxProps<T>,
  ref: React.Ref<React.ElementRef<T>>
) {
  const Component = as || 'div';
  return (
    <Component ref={ref} className={`p-${padding}`} {...rest}>
      {children}
    </Component>
  );
}) as BoxComponent;

// Usage
const ref = React.useRef<HTMLButtonElement>(null);
<Box as="button" ref={ref} onClick={() => ref.current?.focus()}>
  Focus me
</Box>
```

**Expected Output:** A `Box` component that renders as any element, forwards refs, and preserves prop types. The ref is typed as `HTMLButtonElement` when `as="button"`.

**Why This Output Occurs:** The manual `BoxComponent` type preserves the generic signature through `forwardRef`. `React.ElementRef<T>` extracts the ref type of the chosen element.

### Real-World Cases

- **Design system primitives:** `Box`, `Text`, `Heading`, `Stack`, `Flex`.
- **Button components:** `as="a"` for links, `as="button"` for buttons.
- **Typography:** `Text as="h1"` through `Text as="h6"`.
- **Navigation:** `Link as={RouterLink}` for framework-agnostic links.
- **Layout:** `Grid as="section"` for semantic sections.
- **Accessibility:** `IconButton as="a"` for link-styled icons.

### References

- React TypeScript Cheatsheet – Polymorphic Components - https://react-typescript-cheatsheet.netlify.app/docs/advanced/patterns_by_usecase/#polymorphic-components
- TypeScript Handbook – Generic Constraints - https://www.typescriptlang.org/docs/handbook/2/generics.html#generic-constraints
- React Official Documentation – `ComponentPropsWithoutRef` - https://react.dev/reference/react/Component#ComponentPropsWithoutRef
- React Official Documentation – `ElementRef` - https://react.dev/reference/react/ElementRef
- Matt Pocock – Polymorphic Components - https://www.totaltypescript.com/polymorphic-components
- Radix UI – `asChild` Pattern - https://www.radix-ui.com/primitives/docs/guides/composition

---

## Core Concept 4: Component Forwarding & Memos

### Definitions

**Core Definition:** Typing `React.forwardRef` and `React.memo` correctly involves preserving the component's prop types, ref types, and—when the component is generic—its generic type parameters.

**Technical Definition:** `React.forwardRef<RefType, PropsType>(render)` is generic over the ref type and props type. The render function receives `(props: PropsType, ref: React.ForwardedRef<RefType>)`. `React.memo(Component, arePropsEqual?)` returns a memoised component with the same props type as the input component. For non-generic components, TypeScript infers the types automatically. For generic components, `forwardRef` and `memo` erase the generic signature, requiring a manual type cast to restore it. This pattern is essential for design systems that export generic, ref-forwarding components. The fix is to define a manual component type with the generic signature, cast the `forwardRef` result to that type, and use `React.ElementRef<T>` and `React.ComponentPropsWithoutRef<T>` to type the ref and props.

**Beginner-Friendly Explanation:** `forwardRef` and `memo` are wrappers that change how a component works—`forwardRef` passes a ref through, and `memo` skips re-renders when props have not changed. When you wrap a component with these, TypeScript sometimes loses the component's type information, especially if the component is generic. The fix is to manually type the wrapped component so TypeScript knows what props and refs it accepts. This is a common pattern in design systems where components are generic, ref-forwarding, and memoised all at once.

### Purposes

- To type `forwardRef` correctly with the right ref and props types.
- To type `memo` without losing the component's props type.
- To preserve generic type parameters through `forwardRef` and `memo`.
- To enable ref forwarding to DOM elements in wrapped components.
- To optimise generic components with `memo` without losing type safety.
- To build design system components that are ref-forwarding, memoised, and generic.

### Syntax Rules and Structure

**`forwardRef` (Non-Generic):**
```tsx
type InputProps = React.ComponentPropsWithoutRef<'input'> & {
  label: string;
};

const Input = React.forwardRef<HTMLInputElement, InputProps>(
  function Input({ label, ...rest }, ref) {
    return (
      <label>
        {label}
        <input ref={ref} {...rest} />
      </label>
    );
  }
);

// Usage
const ref = React.useRef<HTMLInputElement>(null);
<Input label="Email" ref={ref} />;
```

**Component Breakdown:**
- `React.forwardRef<HTMLInputElement, InputProps>`: The ref type is `HTMLInputElement`; the props type is `InputProps`.
- `ref: React.ForwardedRef<HTMLInputElement>`: Inferred from the generic.
- TypeScript infers everything automatically for non-generic components.

**`memo` (Non-Generic):**
```tsx
type CardProps = {
  title: string;
  children: React.ReactNode;
};

const Card = React.memo(function Card({ title, children }: CardProps) {
  return (
    <div className="card">
      <h2>{title}</h2>
      {children}
    </div>
  );
});

// Usage
<Card title="Welcome">Content</Card>;
```

**Component Breakdown:**
- `React.memo(Component)`: Returns a memoised component with the same props type.
- TypeScript preserves `CardProps`; no manual typing needed.

**Generic Component with `forwardRef` (Workaround):**
```tsx
type SelectProps<T> = {
  options: T[];
  value: T | null;
  onChange: (value: T) => void;
  getLabel: (option: T) => string;
  getValue: (option: T) => string | number;
};

type SelectComponent = <T>(
  props: SelectProps<T> & { ref?: React.Ref<HTMLSelectElement> }
) => React.ReactElement | null;

const Select = React.forwardRef(function Select<T>(
  { options, value, onChange, getLabel, getValue }: SelectProps<T>,
  ref: React.Ref<HTMLSelectElement>
) {
  return (
    <select
      ref={ref}
      value={value ? String(getValue(value)) : ''}
      onChange={(e) => {
        const selected = options.find((o) => String(getValue(o)) === e.target.value);
        if (selected) onChange(selected);
      }}
    >
      <option value="">Select...</option>
      {options.map((option) => (
        <option key={getValue(option)} value={String(getValue(option))}>
          {getLabel(option)}
        </option>
      ))}
    </select>
  );
}) as SelectComponent;
```

**Component Breakdown:**
- `SelectComponent`: A manual type with the generic signature.
- `React.forwardRef(...) as SelectComponent`: The cast restores the generic.
- `ref: React.Ref<HTMLSelectElement>`: The forwarded ref type.
- `T` is inferred from `options` at the call site.

**Generic Component with `memo` (Workaround):**
```tsx
type ListProps<T> = {
  items: T[];
  renderItem: (item: T) => React.ReactNode;
};

function List<T>({ items, renderItem }: ListProps<T>) {
  return <ul>{items.map((item, i) => <li key={i}>{renderItem(item)}</li>)}</ul>;
}

// ❌ Loses generics
// const MemoList = React.memo(List);

// ✅ Preserves generics
const MemoList = React.memo(List) as typeof List;

// Usage
<MemoList items={['a', 'b']} renderItem={(s) => <span>{s}</span>} />;
```

**Component Breakdown:**
- `React.memo(List)`: Returns a memoised component, but TypeScript loses the generic.
- `as typeof List`: Restores the generic signature.
- Consumers use `MemoList` exactly like `List`.

**Generic Component with Both `forwardRef` and `memo`:**
```tsx
type InputProps<T> = {
  value: T;
  onChange: (value: T) => void;
};

type InputComponent = <T>(
  props: InputProps<T> & { ref?: React.Ref<HTMLInputElement> }
) => React.ReactElement | null;

const Input = React.memo(
  React.forwardRef(function Input<T>(
    { value, onChange }: InputProps<T>,
    ref: React.Ref<HTMLInputElement>
  ) {
    return <input ref={ref} value={String(value)} onChange={(e) => onChange(e.target.value as T)} />;
  })
) as InputComponent;
```

**Component Breakdown:**
- `React.forwardRef` inside `React.memo`.
- The `as InputComponent` cast preserves the generic.
- The ref is `HTMLInputElement`; `T` is inferred from `value`.

**Syntax Rules:**
- For non-generic components, `forwardRef<RefType, PropsType>` and `memo(Component)` preserve types automatically.
- For generic components, define a manual component type and cast the wrapped result.
- Use `React.ElementRef<T>` to extract the ref type of an element.
- Use `React.ComponentPropsWithoutRef<T>` to extract the props of an element.
- Use `React.memo(Component) as typeof Component` to preserve generics in `memo`.
- Use `React.forwardRef(...) as ComponentType` to preserve generics in `forwardRef`.
- In React 19, `ref` can be passed as a regular prop; `forwardRef` is still needed for older React versions.
- Document the manual types clearly; they can be confusing to readers.

**Constraints and Limitations:**
- TypeScript cannot infer generics through `forwardRef` or `memo` automatically; manual casts are required.
- Manual casts lose TypeScript's inference; the component type must be written correctly.
- `React.memo` with a custom comparator can further complicate generic typing.
- React 19 changes the ref API; the `forwardRef` workaround may become unnecessary.
- The `as` cast is a type assertion; if the manual type is wrong, consumers get incorrect types.

### Annotated Code Examples

**Example 1: Non-Generic `forwardRef` Input**

```tsx
type InputProps = React.ComponentPropsWithoutRef<'input'> & {
  label: string;
  error?: string;
};

export const Input = React.forwardRef<HTMLInputElement, InputProps>(
  function Input({ label, error, id, ...rest }, ref) {
    const inputId = id ?? React.useId();
    return (
      <div>
        <label htmlFor={inputId}>{label}</label>
        <input
          id={inputId}
          ref={ref}
          aria-invalid={!!error}
          aria-describedby={error ? `${inputId}-error` : undefined}
          {...rest}
        />
        {error && <p id={`${inputId}-error`} role="alert">{error}</p>}
      </div>
    );
  }
);

// Usage
const emailRef = React.useRef<HTMLInputElement>(null);
<Input label="Email" ref={emailRef} error="Email is required" />;
```

**Expected Output:** A labelled input with error display and accessibility attributes. The ref is typed as `HTMLInputElement`, so `emailRef.current?.focus()` is type-safe.

**Why This Output Occurs:** `React.forwardRef<HTMLInputElement, InputProps>` types the ref and props. TypeScript infers `ref` as `React.ForwardedRef<HTMLInputElement>` and destructures `...rest` as the remaining input props.

**Example 2: Generic `memo` Component**

```tsx
type ListProps<T> = {
  items: T[];
  renderItem: (item: T) => React.ReactNode;
  keyExtractor: (item: T) => string | number;
};

function List<T>({ items, renderItem, keyExtractor }: ListProps<T>) {
  console.log('List rendered');
  return (
    <ul>
      {items.map((item) => (
        <li key={keyExtractor(item)}>{renderItem(item)}</li>
      ))}
    </ul>
  );
}

export const MemoList = React.memo(List) as typeof List;

// Usage — T is inferred from items
<MemoList
  items={[{ id: 1, name: 'Alice' }, { id: 2, name: 'Bob' }]}
  renderItem={(user) => <span>{user.name}</span>}
  keyExtractor={(user) => user.id}
/>
```

**Expected Output:** A memoised list that re-renders only when `items`, `renderItem`, or `keyExtractor` change. TypeScript infers `T = { id: number; name: string }` from `items`, so `user.name` is type-safe.

**Why This Output Occurs:** `React.memo(List)` memoises the component but erases the generic. `as typeof List` restores the generic signature. The consumer uses `MemoList` with the same type safety as `List`.

### Real-World Cases

- **Design systems:** Ref-forwarding inputs, buttons, and containers.
- **Generic lists:** Memoised list components that work with any data type.
- **Data tables:** Generic, memoised table components with typed columns.
- **Form fields:** Ref-forwarding inputs for focus management.
- **Third-party integrations:** Memoised wrappers around heavy components.

### References

- React Official Documentation – `forwardRef` - https://react.dev/reference/react/forwardRef
- React Official Documentation – `memo` - https://react.dev/reference/react/memo
- React TypeScript Cheatsheet – forwardRef - https://react-typescript-cheatsheet.netlify.app/docs/basic/getting-started/forward_and_create_ref
- React TypeScript Cheatsheet – memo - https://react-typescript-cheatsheet.netlify.app/docs/basic/getting-started/function_components
- React Official Documentation – `ElementRef` - https://react.dev/reference/react/ElementRef
- React 19 – Ref as a Prop - https://react.dev/blog/2024/12/05/react-19#ref-as-a-prop
- Matt Pocock – forwardRef - https://www.totaltypescript.com/react-forwardref

---

## Core Concept 5: Utility Types

### Definitions

**Core Definition:** Utility types are built-in TypeScript types (`Pick`, `Omit`, `Partial`, `ComponentPropsWithoutRef`, `ComponentPropsWithRef`, `ReturnType`, `Parameters`, `Record`) that derive new types from existing ones without duplication.

**Technical Definition:** Utility types transform existing types via mapped types, conditional types, and indexed access. In React, the most common are: `Pick<T, K>` (select properties from `T`), `Omit<T, K>` (exclude properties from `T`), `Partial<T>` (make all properties optional), `Required<T>` (make all properties required), `Readonly<T>` (make all properties readonly), `Record<K, V>` (create an object type with keys `K` and values `V`), `ComponentPropsWithoutRef<T>` (extract props from a React element type, excluding `ref`), `ComponentPropsWithRef<T>` (extract props including `ref`), `React.ElementRef<T>` (extract the ref type), `ReturnType<F>` (extract a function's return type), and `Parameters<F>` (extract a function's parameters as a tuple). These types reduce duplication and keep component APIs consistent with their underlying elements.

**Beginner-Friendly Explanation:** Utility types are transformations. `Pick` selects properties; `Omit` removes them; `Partial` makes everything optional. `ComponentPropsWithoutRef<'button'>` gives you all the props of a `<button>` element. Instead of copying the list of props by hand, you derive them from the element. This keeps your component's props in sync with the HTML element it wraps.

### Purposes

- To derive new types from existing ones without duplication.
- To extract the props of an HTML element or React component.
- To make all properties of a type optional or required.
- To select or exclude specific properties.
- To type `ref` extraction (`React.ElementRef<T>`).
- To extract a function's return type or parameters.
- To keep component APIs consistent with their underlying elements.

### Syntax Rules and Structure

**`Pick<T, K>`:**
```tsx
type User = { id: number; name: string; email: string; age: number };

type UserPreview = Pick<User, 'id' | 'name'>;
// { id: number; name: string }
```

**`Omit<T, K>`:**
```tsx
type UserWithoutEmail = Omit<User, 'email'>;
// { id: number; name: string; age: number }
```

**`Partial<T>`:**
```tsx
type PartialUser = Partial<User>;
// { id?: number; name?: string; email?: string; age?: number }
```

**`Required<T>`:**
```tsx
type RequiredUser = Required<Partial<User>>;
// { id: number; name: string; email: string; age: number }
```

**`Readonly<T>`:**
```tsx
type ReadonlyUser = Readonly<User>;
// { readonly id: number; readonly name: string; ... }
```

**`Record<K, V>`:**
```tsx
type UserMap = Record<string, User>;
// { [key: string]: User }
```

**`ComponentPropsWithoutRef<T>`:**
```tsx
type ButtonProps = React.ComponentPropsWithoutRef<'button'> & {
  variant?: 'primary' | 'secondary';
};

// ButtonProps includes all native button props (except ref) plus variant
```

**`ComponentPropsWithRef<T>`:**
```tsx
type ButtonPropsWithRef = React.ComponentPropsWithRef<'button'>;
// Includes `ref`
```

**`React.ElementRef<T>`:**
```tsx
type ButtonRef = React.ElementRef<'button'>;
// HTMLButtonElement
```

**`ReturnType<F>` and `Parameters<F>`:**
```tsx
function createUser(name: string, age: number) { /* ... */ }
type UserArgs = Parameters<typeof createUser>;  // [string, number]
type UserResult = ReturnType<typeof createUser>; // { id, name, age }
```

**Common React Patterns:**
```tsx
// Extend native props
type InputProps = React.ComponentPropsWithoutRef<'input'> & { label: string };

// Omit conflicting props
type CustomButtonProps = Omit<React.ComponentPropsWithoutRef<'button'>, 'type'> & {
  type: 'submit' | 'button';
};

// Pick only the props you need
type SearchInputProps = Pick<React.ComponentPropsWithoutRef<'input'>, 'value' | 'onChange' | 'placeholder'>;

// Derive a ref type
type DivRef = React.ElementRef<'div'>;
```

**Syntax Rules:**
- Use `Pick` to select a subset of properties.
- Use `Omit` to exclude specific properties (especially when extending native props).
- Use `Partial` for update payloads where all fields are optional.
- Use `Required` to enforce all fields are present.
- Use `Record` for maps and dictionaries.
- Use `ComponentPropsWithoutRef<'element'>` to extend native element props.
- Use `ComponentPropsWithRef<'element'>` when forwarding refs.
- Use `React.ElementRef<'element'>` to extract the ref type.
- Use `ReturnType<F>` and `Parameters<F>` for function-derived types.
- Combine utility types with intersections (`&`) to build composite props.

**Constraints and Limitations:**
- `Pick` and `Omit` operate on top-level properties only; nested properties require more complex types.
- `Omit` on a union distributes incorrectly; use a distributive conditional type instead.
- `Partial` makes all properties optional, including required ones; use `Pick` for more control.
- `ComponentPropsWithoutRef` does not include `ref`; use `ComponentPropsWithRef` if you need it.
- `React.ElementRef<T>` is being renamed to `React.ComponentRef<T>` in newer React versions.
- Utility types are erased at runtime; they have no performance cost.

### Annotated Code Examples

**Example 1: Extending Native Button Props**

```tsx
type ButtonProps = Omit<React.ComponentPropsWithoutRef<'button'>, 'type'> & {
  variant?: 'primary' | 'secondary' | 'danger';
  type?: 'submit' | 'button' | 'reset';
};

export function Button({
  variant = 'primary',
  type = 'button',
  className,
  ...rest
}: ButtonProps) {
  return (
    <button
      type={type}
      className={`btn btn-${variant} ${className ?? ''}`}
      {...rest}
    />
  );
}

// Usage — all native button props are accepted
<Button variant="danger" type="submit" disabled aria-label="Delete">
  Delete
</Button>
```

**Expected Output:** A button component that accepts all native button props (except the original `type`, which is re-typed) plus the custom `variant` prop. TypeScript enforces that `type` is one of the three allowed values.

**Why This Output Occurs:** `Omit<ComponentPropsWithoutRef<'button'>, 'type'>` removes the native `type` prop, then the intersection adds a custom `type` with literal options. The result is a button component with a stricter `type` prop.

**Example 2: Deriving Types from a Function**

```tsx
async function fetchUser(id: number) {
  const res = await fetch(`/api/users/${id}`);
  return res.json() as Promise<{ id: number; name: string; email: string }>;
}

type FetchUserParams = Parameters<typeof fetchUser>; // [id: number]
type FetchUserResult = Awaited<ReturnType<typeof fetchUser>>;
// { id: number; name: string; email: string }

function useUser(id: FetchUserParams[0]) {
  const [user, setUser] = React.useState<FetchUserResult | null>(null);
  // ...
  return user;
}
```

**Expected Output:** `FetchUserResult` is the awaited return type of `fetchUser`, so `user` is typed as the user object (not a promise). TypeScript infers the shape from the function's implementation.

**Why This Output Occurs:** `Parameters<typeof fetchUser>` extracts the parameter tuple; `ReturnType<typeof fetchUser>` extracts the promise; `Awaited<...>` unwraps it. This keeps the hook in sync with the fetcher's return type.

### Real-World Cases

- **Component libraries:** Extending native props with `ComponentPropsWithoutRef`.
- **Form inputs:** Deriving props from native `<input>`, `<select>`, `<textarea>`.
- **API clients:** Extracting return types from fetchers.
- **Redux/Zustand:** `Pick` and `Omit` for slice types.
- **Type narrowing:** `Partial` for update payloads, `Required` for final types.
- **Ref typing:** `React.ElementRef<'button'>` for imperative handles.

### References

- TypeScript Handbook – Utility Types - https://www.typescriptlang.org/docs/handbook/utility-types.html
- TypeScript Handbook – `Pick<T, K>` - https://www.typescriptlang.org/docs/handbook/utility-types.html#picktype-keys
- TypeScript Handbook – `Omit<T, K>` - https://www.typescriptlang.org/docs/handbook/utility-types.html#omittype-keys
- TypeScript Handbook – `Partial<T>` - https://www.typescriptlang.org/docs/handbook/utility-types.html#partialtype
- TypeScript Handbook – `Record<K, V>` - https://www.typescriptlang.org/docs/handbook/utility-types.html#recordkeys-type
- React Official Documentation – `ComponentPropsWithoutRef` - https://react.dev/reference/react/Component#ComponentPropsWithoutRef
- React Official Documentation – `ComponentPropsWithRef` - https://react.dev/reference/react/Component#ComponentPropsWithRef
- React Official Documentation – `ElementRef` - https://react.dev/reference/react/ElementRef
- React TypeScript Cheatsheet – Utility Types - https://react-typescript-cheatsheet.netlify.app/docs/basic/getting-started/basic_type_example

---

## Core Concept 6: Custom Hook Typing

### Definitions

**Core Definition:** Custom hook typing is the practice of declaring a hook's parameter types, return type, and tuple return structure so consumers get full type safety and TypeScript can infer correctly at the call site.

**Technical Definition:** A custom hook is a function whose name begins with `use` and that calls other Hooks. Its type is defined by its parameters (`function useHook(param: T): ReturnType`) and its return value. When the hook returns an array or tuple, TypeScript infers `(T | U)[]` unless the return is annotated with `as const` or an explicit tuple type (`[T, U]`). The `as const` assertion preserves the tuple structure, so destructuring gives the correct types to each element. Generic hooks (`function useLocalStorage<T>(key: string, initial: T): [T, (v: T) => void]`) preserve the type parameter through the hook's body. Hooks that return objects should have an explicit return type to keep the API stable. Hooks that accept callbacks should type the callback parameters precisely.

**Beginner-Friendly Explanation:** A custom hook is a function that reuses logic across components. Typing it means declaring what parameters it accepts and what it returns. The tricky part is the return value: if you return an array like `[value, setValue]`, TypeScript infers `(T | (v: T) => void)[]`—a mix of both types. You need `as const` or an explicit tuple type so that destructuring gives the correct type to each element. Otherwise, `value` and `setValue` have the same type, and you lose type safety.

### Purposes

- To declare the parameter types of a custom hook.
- To declare the return type precisely (object or tuple).
- To preserve generic type parameters through the hook.
- To enable correct destructuring of tuple returns.
- To type callbacks and event handlers passed to the hook.
- To provide a stable, documented API for consumers.

### Syntax Rules and Structure

**Basic Custom Hook:**
```tsx
function useToggle(initial: boolean = false): [boolean, () => void] {
  const [value, setValue] = React.useState(initial);
  const toggle = React.useCallback(() => setValue((v) => !v), []);
  return [value, toggle];
}

// Usage
const [isOpen, toggleOpen] = useToggle(false);
// isOpen: boolean, toggleOpen: () => void
```

**Component Breakdown:**
- `function useToggle(initial: boolean = false)`: The parameter type is `boolean`.
- `: [boolean, () => void]`: The explicit tuple return type.
- Destructuring gives `isOpen: boolean` and `toggleOpen: () => void`.

**Without `as const` or Tuple Annotation (Broken):**
```tsx
// ❌ Infers (boolean | (() => void))[]
function useToggleBroken(initial: boolean = false) {
  const [value, setValue] = React.useState(initial);
  return [value, () => setValue((v) => !v)];
}

const [isOpen, toggle] = useToggleBroken();
// isOpen: boolean | (() => void)
// toggle: boolean | (() => void)
```

**With `as const` (Fixed):**
```tsx
function useToggle(initial: boolean = false) {
  const [value, setValue] = React.useState(initial);
  return [value, () => setValue((v) => !v)] as const;
}

const [isOpen, toggle] = useToggle();
// isOpen: boolean
// toggle: () => void
```

**Generic Custom Hook:**
```tsx
function useLocalStorage<T>(key: string, initialValue: T): [T, (value: T) => void] {
  const [stored, setStored] = React.useState<T>(() => {
    try {
      const item = localStorage.getItem(key);
      return item ? (JSON.parse(item) as T) : initialValue;
    } catch {
      return initialValue;
    }
  });

  const setValue = React.useCallback(
    (value: T) => {
      setStored(value);
      localStorage.setItem(key, JSON.stringify(value));
    },
    [key]
  );

  return [stored, setValue];
}

// Usage
const [user, setUser] = useLocalStorage<User | null>('user', null);
// user: User | null, setUser: (value: User | null) => void
```

**Component Breakdown:**
- `<T>`: The generic type parameter.
- `: [T, (value: T) => void]`: The tuple return type preserves `T`.
- The hook infers `T` from `initialValue` or accepts it explicitly.

**Hook with Object Return:**
```tsx
type UseFetchResult<T> = {
  data: T | null;
  loading: boolean;
  error: Error | null;
  refetch: () => void;
};

function useFetch<T>(url: string): UseFetchResult<T> {
  const [data, setData] = React.useState<T | null>(null);
  const [loading, setLoading] = React.useState(true);
  const [error, setError] = React.useState<Error | null>(null);

  const refetch = React.useCallback(() => {
    setLoading(true);
    fetch(url)
      .then((res) => res.json())
      .then((json: T) => { setData(json); setError(null); })
      .catch(setError)
      .finally(() => setLoading(false));
  }, [url]);

  React.useEffect(() => { refetch(); }, [refetch]);

  return { data, loading, error, refetch };
}
```

**Component Breakdown:**
- `UseFetchResult<T>`: An explicit return type with a generic.
- The hook returns an object with typed properties.
- Consumers destructure by name, preserving types.

**Hook with Callback Parameters:**
```tsx
function useEventListener<K extends keyof WindowEventMap>(
  eventName: K,
  handler: (event: WindowEventMap[K]) => void,
  element: Window | HTMLElement = window
): void {
  const savedHandler = React.useRef(handler);

  React.useEffect(() => {
    savedHandler.current = handler;
  }, [handler]);

  React.useEffect(() => {
    const listener = (event: WindowEventMap[K]) => savedHandler.current(event);
    element.addEventListener(eventName, listener as EventListener);
    return () => element.removeEventListener(eventName, listener as EventListener);
  }, [eventName, element]);
}

// Usage — event is typed based on eventName
useEventListener('click', (e) => console.log(e.clientX));       // MouseEvent
useEventListener('keydown', (e) => console.log(e.key));         // KeyboardEvent
```

**Component Breakdown:**
- `<K extends keyof WindowEventMap>`: Constrains the event name.
- `handler: (event: WindowEventMap[K]) => void`: Types the event based on the name.
- The consumer gets the correct event type from the event name.

**Syntax Rules:**
- Annotate the return type explicitly, especially for tuples and objects.
- Use `as const` for tuple returns when not annotating the return type.
- Use generics for hooks that work with any type.
- Type callbacks with their parameter and return types.
- Use `React.Dispatch<React.SetStateAction<T>>` when returning a state setter.
- Use `React.RefObject<T>` for DOM refs returned by a hook.
- Document the hook's contract with an explicit return type.
- Prefer object returns for hooks with three or more values; tuples for two.

**Constraints and Limitations:**
- `as const` requires the array to be inferred as a tuple; it does not work with dynamic arrays.
- Tuple returns with more than two elements are hard to read; use objects.
- Generic hooks cannot be memoised easily; wrap with `useCallback` or `useMemo` inside.
- Custom hooks do not automatically preserve generics through `React.memo` or `forwardRef`.
- The hook's return type must be stable; changing it is a breaking change.

### Annotated Code Examples

**Example 1: `useLocalStorage` with Generic and Tuple Return**

```tsx
type UseLocalStorageReturn<T> = [T, (value: T) => void];

function useLocalStorage<T>(key: string, initialValue: T): UseLocalStorageReturn<T> {
  const [stored, setStored] = React.useState<T>(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? (JSON.parse(item) as T) : initialValue;
    } catch (error) {
      console.warn(`Error reading localStorage key "${key}":`, error);
      return initialValue;
    }
  });

  const setValue = React.useCallback(
    (value: T) => {
      try {
        setStored(value);
        window.localStorage.setItem(key, JSON.stringify(value));
      } catch (error) {
        console.warn(`Error setting localStorage key "${key}":`, error);
      }
    },
    [key]
  );

  return [stored, setValue];
}

// Usage
const [theme, setTheme] = useLocalStorage<'light' | 'dark'>('theme', 'light');
// theme: 'light' | 'dark'
// setTheme: (value: 'light' | 'dark') => void
```

**Expected Output:** A `useLocalStorage` hook that stores any type in `localStorage`. The tuple return preserves the type: `theme` is `'light' | 'dark'`, and `setTheme` accepts only those values.

**Why This Output Occurs:** The generic `<T>` and the explicit tuple return type `[T, (value: T) => void]` preserve `T` through the hook. Destructuring gives the correct type to each element.

**Example 2: `useDebouncedValue` with Generic**

```tsx
function useDebouncedValue<T>(value: T, delay: number = 300): T {
  const [debounced, setDebounced] = React.useState(value);

  React.useEffect(() => {
    const timer = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);

  return debounced;
}

// Usage
const [query, setQuery] = React.useState('');
const debouncedQuery = useDebouncedValue(query, 500);
// debouncedQuery: string
```

**Expected Output:** `debouncedQuery` is the debounced version of `query`, typed as `string` (matching `query`). The hook works with any type.

**Why This Output Occurs:** `<T>` preserves the value's type. The hook returns `T`, so `debouncedQuery` has the same type as `query`.

### Real-World Cases

- **`useLocalStorage<T>`:** Persisting any type to `localStorage`.
- **`useFetch<T>`:** Fetching typed API data.
- **`useDebouncedValue<T>`:** Debouncing input values.
- **`useToggle`:** Boolean toggles with a stable callback.
- **`useEventListener<K>`:** Typed event listeners.
- **`useMediaQuery`:** Returning booleans based on CSS media queries.
- **`useForm<T>`:** Typed form state and validation.

### References

- React Official Documentation – Reusing Logic with Custom Hooks - https://react.dev/learn/reusing-logic-with-custom-hooks
- React TypeScript Cheatsheet – Custom Hooks - https://react-typescript-cheatsheet.netlify.app/docs/basic/getting-started/hooks
- TypeScript Handbook – Generics - https://www.typescriptlang.org/docs/handbook/2/generics.html
- TypeScript Handbook – `as const` - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-4.html#const-assertions
- TypeScript Handbook – Tuple Types - https://www.typescriptlang.org/docs/handbook/2/objects.html#tuple-types
- Matt Pocock – Custom Hooks - https://www.totaltypescript.com/react-custom-hooks

---

## Core Concept 7: API Response Types

### Definitions

**Core Definition:** API response typing is the practice of declaring TypeScript types for the data returned by backend endpoints, and integrating those types with generic fetchers, Axios clients, and React hooks.

**Technical Definition:** API response typing involves defining types that mirror the backend's JSON payloads, then using those types in fetchers (`async function fetchUser(): Promise<User>`), Axios calls (`axios.get<User>(url)`), and React hooks (`useQuery<User>`). Because TypeScript types are erased at runtime, they do not validate the response—the developer must either trust the backend (and validate in tests) or use a runtime validation library (Zod, io-ts, Valibot) to parse and validate the response, inferring the type from the schema. The modern pattern is **schema-first**: define a Zod schema, infer the TypeScript type with `z.infer<typeof Schema>`, and parse the response with `Schema.parse(json)`. This gives both compile-time types and runtime guarantees. API response types should model the exact shape returned by the backend, including nested objects, optional fields, and discriminated unions.

**Beginner-Friendly Explanation:** When your React app fetches data from a server, TypeScript has no idea what shape that data has—it is just `any` or `unknown`. API response typing is about declaring what the data looks like so you can use it safely. The best approach is to define a schema (with Zod) that both validates the data at runtime and produces a TypeScript type. Then your fetcher and hooks are fully typed, and if the backend changes its shape, you get a clear error instead of a silent bug.

### Purposes

- To declare the shape of API responses.
- To type fetchers, Axios calls, and React Query hooks.
- To validate API responses at runtime with Zod or similar.
- To infer TypeScript types from schemas (single source of truth).
- To handle nested objects, optional fields, and discriminated unions.
- To integrate typed API models with components and hooks.
- To catch backend contract changes at compile time (when using generated types).

### Syntax Rules and Structure

**Manual Type Declaration:**
```typescript
type User = {
  id: number;
  name: string;
  email: string;
  avatar?: string;
  createdAt: string;
};

type ApiResponse<T> = {
  data: T;
  status: number;
  message?: string;
};

async function fetchUser(id: number): Promise<ApiResponse<User>> {
  const res = await fetch(`/api/users/${id}`);
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.json() as Promise<ApiResponse<User>>;
}
```

**Component Breakdown:**
- `User`: The shape of the user object.
- `ApiResponse<T>`: A generic envelope type.
- `fetchUser`: Returns `Promise<ApiResponse<User>>`.

**Zod Schema-First (Recommended):**
```typescript
import { z } from 'zod';

const UserSchema = z.object({
  id: z.number(),
  name: z.string(),
  email: z.string().email(),
  avatar: z.string().url().optional(),
  createdAt: z.string().datetime(),
});

type User = z.infer<typeof UserSchema>;

const ApiResponseSchema = <T extends z.ZodTypeAny>(dataSchema: T) =>
  z.object({
    data: dataSchema,
    status: z.number(),
    message: z.string().optional(),
  });

async function fetchUser(id: number): Promise<User> {
  const res = await fetch(`/api/users/${id}`);
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  const json = await res.json();
  const parsed = ApiResponseSchema(UserSchema).parse(json);
  return parsed.data;
}
```

**Component Breakdown:**
- `UserSchema`: The Zod schema defines the shape and validates at runtime.
- `z.infer<typeof UserSchema>`: Infers the TypeScript type from the schema.
- `ApiResponseSchema(dataSchema)`: A generic envelope schema.
- `parse(json)`: Validates and returns the typed data; throws on mismatch.

**Axios with Generic Types:**
```typescript
import axios from 'axios';

const api = axios.create({ baseURL: '/api' });

async function fetchUser(id: number): Promise<User> {
  const { data } = await api.get<ApiResponse<User>>(`/users/${id}`);
  return data.data;
}

async function createUser(input: { name: string; email: string }): Promise<User> {
  const { data } = await api.post<ApiResponse<User>>('/users', input);
  return data.data;
}
```

**Component Breakdown:**
- `api.get<ApiResponse<User>>(url)`: Types the response as `ApiResponse<User>`.
- `data.data`: Accesses the user payload.
- The generic is explicit because Axios cannot infer the response shape.

**TanStack Query with Generic Types:**
```tsx
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

function useUser(id: number) {
  return useQuery<User>({
    queryKey: ['user', id],
    queryFn: () => fetchUser(id),
  });
}

function useCreateUser() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (input: { name: string; email: string }) => createUser(input),
    onSuccess: () => queryClient.invalidateQueries({ queryKey: ['users'] }),
  });
}

function UserProfile({ id }: { id: number }) {
  const { data: user, isPending, isError } = useUser(id);

  if (isPending) return <p>Loading...</p>;
  if (isError) return <p>Error loading user</p>;
  return <h1>{user.name}</h1>; // user: User
}
```

**Component Breakdown:**
- `useQuery<User>`: Types the `data` as `User`.
- `useMutation`: Types the mutation's `mutate` arguments and result.
- `user.name`: Type-safe access to the user's properties.

**Discriminated Union for API Results:**
```typescript
type ApiResult<T> =
  | { status: 'success'; data: T }
  | { status: 'error'; error: { code: number; message: string } };

async function fetchUserResult(id: number): Promise<ApiResult<User>> {
  const res = await fetch(`/api/users/${id}`);
  if (!res.ok) {
    return { status: 'error', error: { code: res.status, message: res.statusText } };
  }
  const data = await res.json();
  return { status: 'success', data };
}

// Usage
const result = await fetchUserResult(1);
if (result.status === 'success') {
  console.log(result.data.name); // User
} else {
  console.error(result.error.message);
}
```

**Component Breakdown:**
- `ApiResult<T>`: A discriminated union with `success` and `error` variants.
- The consumer narrows based on `status`.

**Syntax Rules:**
- Define API response types that mirror the backend's JSON shape.
- Use Zod (or Valibot, io-ts) for runtime validation and type inference.
- Use `z.infer<typeof Schema>` to derive the TypeScript type from the schema.
- Use `ApiResponse<T>` or `ApiResult<T>` for envelope types.
- Type Axios calls with `axios.get<T>(url)`.
- Type TanStack Query with `useQuery<T>({ queryKey, queryFn })`.
- Use discriminated unions for results with success/error branches.
- Keep API types in a shared module (or generate them from OpenAPI/GraphQL).
- Validate at the boundary; never trust `as` assertions on API data.

**Constraints and Limitations:**
- Manual types are not validated at runtime; the backend may return a different shape.
- Zod adds a small runtime cost (parsing) but provides safety.
- Axios and fetch cannot infer response types automatically; you must specify them.
- Generated types (from OpenAPI, GraphQL codegen) are the most reliable for large APIs.
- Optional fields must be handled with optional chaining or defaults.
- Nested objects and arrays require recursive validation.

### Annotated Code Examples

**Example 1: Zod Schema-First API Client**

```typescript
import { z } from 'zod';

// 1. Define the schema
const UserSchema = z.object({
  id: z.number(),
  name: z.string(),
  email: z.string().email(),
  role: z.enum(['admin', 'user', 'guest']),
  createdAt: z.string().datetime(),
});

// 2. Infer the type
type User = z.infer<typeof UserSchema>;

// 3. Generic envelope schema
const ApiResponseSchema = <T extends z.ZodTypeAny>(dataSchema: T) =>
  z.object({
    data: dataSchema,
    status: z.number(),
    message: z.string().optional(),
  });

// 4. Typed fetcher with validation
async function fetchUser(id: number): Promise<User> {
  const res = await fetch(`/api/users/${id}`);
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  const json = await res.json();
  const parsed = ApiResponseSchema(UserSchema).parse(json);
  return parsed.data;
}

// 5. React hook
function useUser(id: number) {
  return useQuery<User>({
    queryKey: ['user', id],
    queryFn: () => fetchUser(id),
  });
}

// 6. Component
function UserProfile({ id }: { id: number }) {
  const { data: user, isPending, isError } = useUser(id);

  if (isPending) return <p>Loading...</p>;
  if (isError) return <p>Error</p>;
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
      <span>{user.role}</span>
    </div>
  );
}
```

**Expected Output:** The component fetches a user by ID, validates the response against the Zod schema, and renders the typed user. If the backend returns an invalid shape, `parse` throws and the query's `isError` becomes `true`.

**Why This Output Occurs:** `UserSchema` defines the shape and validates at runtime. `z.infer` produces the `User` type. `fetchUser` parses the response, throwing on invalid data. `useQuery<User>` types the `data` as `User`. The component accesses `user.name`, `user.email`, and `user.role` with full type safety.

**Example 2: Generic Axios Client**

```typescript
import axios from 'axios';
import { z } from 'zod';

const api = axios.create({ baseURL: '/api' });

type ApiResponse<T> = { data: T; status: number; message?: string };

// Generic fetcher
async function get<T>(url: string): Promise<T> {
  const { data } = await api.get<ApiResponse<T>>(url);
  return data.data;
}

async function post<T, B = unknown>(url: string, body: B): Promise<T> {
  const { data } = await api.post<ApiResponse<T>>(url, body);
  return data.data;
}

// Typed API methods
const UserSchema = z.object({ id: z.number(), name: z.string() });
type User = z.infer<typeof UserSchema>;

async function fetchUser(id: number): Promise<User> {
  const raw = await get<unknown>(`/users/${id}`);
  return UserSchema.parse(raw); // Validate and narrow
}

async function createUser(input: { name: string }): Promise<User> {
  return post<User, typeof input>('/users', input);
}
```

**Expected Output:** `get<T>` and `post<T, B>` are generic Axios wrappers that return typed data. `fetchUser` validates the raw response against `UserSchema` before returning. `createUser` types the body and the response.

**Why This Output Occurs:** `get<T>` types the Axios response as `ApiResponse<T>` and returns `data.data`. `fetchUser` validates with `UserSchema.parse` because Axios cannot guarantee the shape at runtime. The generic parameters make the wrappers reusable across all API endpoints.

### Real-World Cases

- **REST APIs:** Typed fetchers with Zod validation.
- **GraphQL:** Codegen-generated types for queries and mutations.
- **tRPC:** End-to-end type safety without codegen.
- **OpenAPI:** Generated types from OpenAPI specs.
- **Axios clients:** Generic `get`, `post`, `put`, `delete` methods with typed responses.
- **React Query:** Typed `useQuery` and `useMutation` hooks.
- **Form submissions:** Typed request and response models.

### References

- Zod – Documentation - https://zod.dev/
- Zod – `z.infer` - https://zod.dev/?id=infer-type
- TanStack Query – TypeScript - https://tanstack.com/query/latest/docs/framework/react/typescript
- Axios – TypeScript - https://axios-http.com/docs/typescript
- OpenAPI TypeScript - https://github.com/drwpow/openapi-typescript
- GraphQL Code Generator - https://the-guild.dev/graphql/codegen
- tRPC - https://trpc.io/
- TypeScript Handbook – Generics - https://www.typescriptlang.org/docs/handbook/2/generics.html
- React TypeScript Cheatsheet – API Response Types - https://react-typescript-cheatsheet.netlify.app/docs/basic/getting-started/basic_type_example

---

## Comparison and Decision Guidance

| Concept | When to Use | When to Avoid | Key Risk |
|---|---|---|---|
| **Generic components** | Reusable lists, tables, selects | Simple, single-type components | Over-constraining generics |
| **Discriminated unions** | Mutually exclusive prop combinations | Simple optional props | Forgetting `default: never` |
| **Polymorphic components** | Design system primitives, `as` prop | Components with fixed elements | Losing generics through `forwardRef` |
| **forwardRef + memo** | Ref-forwarding, memoised components | When React 19 ref-as-prop suffices | Manual type casts |
| **Utility types** | Extending native props, deriving types | Over-complex derivations | Deep `Omit`/`Pick` chains |
| **Custom hook typing** | All custom hooks | Returning `any` or untyped arrays | Missing tuple return types |
| **API response types** | All API calls | Trusting `as` assertions | Runtime validation gaps |

**Decision Guidance:**
- **Use generics** for any component that works with multiple data types.
- **Use discriminated unions** for components with mutually exclusive modes.
- **Use polymorphic components** in design systems where semantic elements matter.
- **Use `forwardRef` and `memo`** for ref-forwarding and performance-critical components.
- **Use utility types** to derive props from native elements instead of duplicating.
- **Annotate custom hook return types**, especially tuples; use `as const` for tuple inference.
- **Use Zod (or similar)** for API response validation; never trust `as` on API data.
- **Integrate API types with TanStack Query** via `useQuery<T>` and `useMutation`.
- **Keep API types in a shared module** or generate them from OpenAPI/GraphQL.
- **Prefer object returns** from hooks with three or more values; tuples for two.

---

## References

- TypeScript Handbook - https://www.typescriptlang.org/docs/handbook/intro.html
- TypeScript Handbook – Generics - https://www.typescriptlang.org/docs/handbook/2/generics.html
- TypeScript Handbook – Generic Constraints - https://www.typescriptlang.org/docs/handbook/2/generics.html#generic-constraints
- TypeScript Handbook – Discriminated Unions - https://www.typescriptlang.org/docs/handbook/2/narrowing.html#discriminated-unions
- TypeScript Handbook – Exhaustiveness Checking - https://www.typescriptlang.org/docs/handbook/2/narrowing.html#exhaustiveness-checking
- TypeScript Handbook – Utility Types - https://www.typescriptlang.org/docs/handbook/utility-types.html
- TypeScript Handbook – `keyof` Type Operator - https://www.typescriptlang.org/docs/handbook/2/keyof-types.html
- TypeScript Handbook – Indexed Access Types - https://www.typescriptlang.org/docs/handbook/2/indexed-access-types.html
- TypeScript Handbook – Tuple Types - https://www.typescriptlang.org/docs/handbook/2/objects.html#tuple-types
- TypeScript Handbook – `as const` - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-4.html#const-assertions
- React TypeScript Cheatsheet - https://react-typescript-cheatsheet.netlify.app/
- React TypeScript Cheatsheet – Generic Components - https://react-typescript-cheatsheet.netlify.app/docs/advanced/patterns_by_usecase/#generic-components
- React TypeScript Cheatsheet – Polymorphic Components - https://react-typescript-cheatsheet.netlify.app/docs/advanced/patterns_by_usecase/#polymorphic-components
- React TypeScript Cheatsheet – Discriminated Unions - https://react-typescript-cheatsheet.netlify.app/docs/advanced/patterns_by_usecase/#discriminated-unions
- React TypeScript Cheatsheet – Custom Hooks - https://react-typescript-cheatsheet.netlify.app/docs/basic/getting-started/hooks
- React TypeScript Cheatsheet – forwardRef - https://react-typescript-cheatsheet.netlify.app/docs/basic/getting-started/forward_and_create_ref
- React Official Documentation – TypeScript - https://react.dev/learn/typescript
- React Official Documentation – `forwardRef` - https://react.dev/reference/react/forwardRef
- React Official Documentation – `memo` - https://react.dev/reference/react/memo
- React Official Documentation – `ComponentPropsWithoutRef` - https://react.dev/reference/react/Component#ComponentPropsWithoutRef
- React Official Documentation – `ComponentPropsWithRef` - https://react.dev/reference/react/Component#ComponentPropsWithRef
- React Official Documentation – `ElementRef` - https://react.dev/reference/react/ElementRef
- React Official Documentation – Reusing Logic with Custom Hooks - https://react.dev/learn/reusing-logic-with-custom-hooks
- React 19 – Ref as a Prop - https://react.dev/blog/2024/12/05/react-19#ref-as-a-prop
- Zod – Documentation - https://zod.dev/
- TanStack Query – TypeScript - https://tanstack.com/query/latest/docs/framework/react/typescript
- Axios – TypeScript - https://axios-http.com/docs/typescript
- tRPC - https://trpc.io/
- GraphQL Code Generator - https://the-guild.dev/graphql/codegen
- Matt Pocock – Total TypeScript - https://www.totaltypescript.com/
- Matt Pocock – Generic Components - https://www.totaltypescript.com/react-generic-components
- Matt Pocock – Polymorphic Components - https://www.totaltypescript.com/polymorphic-components
- Matt Pocock – Custom Hooks - https://www.totaltypescript.com/react-custom-hooks
- Kent C. Dodds – Discriminated Unions for Component Props - https://kentcdodds.com/blog/discriminated-unions-for-component-props