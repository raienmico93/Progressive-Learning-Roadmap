# TypeScript Fundamentals for React: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** TypeScript is a statically typed superset of JavaScript that compiles to plain JavaScript, adding optional type annotations, interfaces, generics, and compile-time type checking that catch errors before runtime—making React components, props, and state safer and more self-documenting.

**Technical Definition:** TypeScript extends ECMAScript with a structural type system that performs static analysis at compile time. It supports primitive types (`string`, `number`, `boolean`, `null`, `undefined`, `symbol`, `bigint`), composite types (arrays, tuples, objects), and advanced constructs (unions, intersections, generics, conditional types, mapped types). In React, TypeScript is used to type component props, state, event handlers, refs, context values, and custom hooks. The `tsconfig.json` file controls strictness (`strict`, `noImplicitAny`, `strictNullChecks`), module resolution, and JSX handling. TypeScript's type system is **structural** (based on shape, not name) and **erased** at runtime (types do not exist in the compiled output). This means type assertions (`as`) and non-null assertions (`!`) are compile-time-only constructs that do not perform runtime checks.

**Beginner-Friendly Explanation:** TypeScript is JavaScript with labels. You tell TypeScript what kind of data each variable, function parameter, and component prop should hold, and TypeScript warns you when you use them incorrectly—before you run the code. In React, this means if you pass a number to a prop that expects a string, TypeScript catches it in your editor. It does not change how your code runs; it just helps you catch mistakes earlier and makes your code easier to understand.

### Key Characteristics

- **Static Typing:** Types are checked at compile time, not runtime; type errors appear in the editor and CI, not in the browser.
- **Structural Typing:** Types are compared by shape, not by name—two types with the same structure are compatible.
- **Type Erasure:** All type annotations are removed during compilation; the emitted JavaScript contains no type information.
- **Gradual Adoption:** TypeScript is a superset of JavaScript; you can add types incrementally.
- **Strictness is Configurable:** `strict: true` enables all strict checks; individual flags (`noImplicitAny`, `strictNullChecks`) can be toggled.
- **React-Specific Patterns:** Props, state, refs, events, context, and generics each have idiomatic TypeScript patterns.
- **Editor Integration:** TypeScript powers autocomplete, inline documentation, and refactoring in VS Code and other editors.

### Prerequisites

- Solid understanding of JavaScript (variables, functions, objects, arrays, closures, modules).
- Familiarity with React function components, JSX, props, and Hooks.
- Working knowledge of ES modules (`import`/`export`).
- Basic understanding of the command line and npm.
- A code editor with TypeScript support (VS Code recommended).

### Related Programming Areas

- **React Development:** Component props, state, refs, context, and hooks.
- **Type Systems:** Structural typing, generics, and type inference.
- **Build Tooling:** `tsc`, Babel, Vite, Webpack, and `tsconfig.json`.
- **API Contracts:** Typed API clients, runtime validation (Zod), and code generation.
- **Testing:** Typed test utilities and mocking.

### Core Concepts / Features

1. Basic Types (Primitives, Arrays, Objects, `unknown` vs `any`)
2. Interfaces vs. Type Aliases
3. Union Types & Literal Types
4. Generics
5. Type Narrowing & Type Guards
6. Type Assertions & Type Casting

---

## Core Concept 1: Basic Types (Primitives, Arrays, Objects, `unknown` vs `any`)

### Definitions

**Core Definition:** Basic types are the primitive and composite type annotations TypeScript provides for strings, numbers, booleans, null, undefined, arrays, tuples, and objects, along with the special `unknown` and `any` types that represent values of uncertain type.

**Technical Definition:** TypeScript's primitive types are `string`, `number`, `boolean`, `null`, `undefined`, `symbol`, and `bigint`. Arrays are typed as `T[]` or `Array<T>`. Tuples are fixed-length arrays with per-position types (`[string, number]`). Object types are written inline (`{ name: string; age: number }`) or via interfaces/type aliases. The `any` type disables type checking entirely—values of type `any` can be assigned to and from anything. The `unknown` type is the type-safe counterpart of `any`: values of type `unknown` can hold any value, but you must narrow them before use. TypeScript's `strict` mode enables `noImplicitAny`, which errors on unannotated parameters and variables that would otherwise be `any`.

**Beginner-Friendly Explanation:** Basic types are labels you put on your data. `string` means text, `number` means a number, `boolean` means true/false. Arrays are lists of one type; objects are collections of named properties. The two special types are `any` (which says "I don't know and I don't care"—turn off checking) and `unknown` (which says "I don't know yet, so you must check before using"). Always prefer `unknown` over `any`; it is safer.

### Purposes

- To declare what kind of data a variable, parameter, or property holds.
- To catch type mismatches at compile time rather than runtime.
- To enable editor autocomplete and inline documentation.
- To document intent for other developers.
- To prevent `null`/`undefined` errors with `strictNullChecks`.
- To use `unknown` instead of `any` for type-safe handling of uncertain values.

### Syntax Rules and Structure

**Primitives:**
```typescript
let name: string = 'Alice';
let age: number = 30;
let isActive: boolean = true;
let nothing: null = null;
let notDefined: undefined = undefined;
let id: symbol = Symbol('id');
let big: bigint = 9007199254740991n;
```

**Arrays and Tuples:**
```typescript
let names: string[] = ['Alice', 'Bob'];
let scores: Array<number> = [95, 87, 78];
let pair: [string, number] = ['Alice', 30];       // Tuple
let coordinates: [number, number, number] = [1, 2, 3];
```

**Objects:**
```typescript
let user: { name: string; age: number; email?: string } = {
  name: 'Alice',
  age: 30,
};
// `email` is optional (email?: string)
```

**`any` vs. `unknown`:**
❌ any — disables type checking
```ts
let valueAny: any = 'hello';
valueAny.toFixed(2); // No error, but crashes at runtime
```
\
✅ unknown — requires narrowing before use
```ts
let valueUnknown: unknown = 'hello';
// valueUnknown.toFixed(2); // Error: Object is of type 'unknown'
if (typeof valueUnknown === 'string') {
  valueUnknown.toUpperCase(); // OK — narrowed to string
}
```

**React Component with Typed Props:**
```tsx
type ButtonProps = {
  label: string;
  count?: number;
  disabled?: boolean;
  onClick: () => void;
};

function Button({ label, count = 0, disabled = false, onClick }: ButtonProps) {
  return (
    <button onClick={onClick} disabled={disabled}>
      {label} ({count})
    </button>
  );
}
```

**Syntax Rules:**
- Use lowercase primitive type names (`string`, `number`, `boolean`), not `String`, `Number`, `Boolean`.
- Use `T[]` for arrays; use `Array<T>` for generic contexts.
- Use tuples (`[string, number]`) for fixed-length, position-typed arrays.
- Mark optional object properties with `?` (`email?: string`).
- Use `unknown` instead of `any` when the type is genuinely unknown.
- Never use `any` unless you have no alternative; prefer `unknown`, generics, or narrowing.
- Enable `strict: true` in `tsconfig.json` to enforce `noImplicitAny` and `strictNullChecks`.

**Constraints and Limitations:**
- TypeScript types are erased at runtime; they do not validate data from APIs or user input.
- `any` propagates: once a value is `any`, everything it touches becomes `any`.
- `strictNullChecks` makes `null` and `undefined` distinct types; code that assumes they are assignable to everything will error.
- Empty arrays are inferred as `any[]` unless annotated or initialised with values.
- Object types are structural; excess property checks apply only to object literals.

### Annotated Code Examples

**Example 1: Typed User Object with Optional Fields**

```tsx
type User = {
  id: number;
  name: string;
  email?: string;           // Optional
  readonly createdAt: Date; // Cannot be reassigned
};

function UserCard({ user }: { user: User }) {
  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email ?? 'No email provided'}</p>
      <small>Joined: {user.createdAt.toLocaleDateString()}</small>
    </div>
  );
}
```
```tsx
// Usage
const user: User = { id: 1, name: 'Alice', createdAt: new Date() };

<UserCard user={user} />;
```

**Expected Output:** A card with "Alice", "No email provided", and the join date. TypeScript enforces that `user` has `id`, `name`, and `createdAt`; `email` is optional. `readonly createdAt` prevents reassignment.

**Why This Output Occurs:** The `User` type defines the shape. `email?` makes `email` optional; `??` provides a fallback. `readonly` prevents mutation. TypeScript errors if a required field is missing or a field has the wrong type.

**Example 2: `unknown` vs `any` in a Safe API Response Handler**

```tsx
// ❌ any — no safety
function handleAny(data: any) {
  return data.user.name.toUpperCase(); // Crashes if data.user is undefined
}
```
```tsx
// ✅ unknown — requires narrowing
function handleUnknown(data: unknown) {
  if (
    typeof data === 'object' &&
    data !== null &&
    'user' in data &&
    typeof (data as { user: unknown }).user === 'object'
  ) {
    const user = (data as { user: { name: unknown } }).user;
    if (typeof user.name === 'string') {
      return user.name.toUpperCase();
    }
  }
  throw new Error('Invalid data shape');
}
```

**Expected Output:** `handleUnknown` safely narrows the `unknown` value through a series of `typeof`, `in`, and property checks before accessing `name`. `handleAny` accesses properties without checking, which will crash at runtime if the shape is unexpected.

**Why This Output Occurs:** `unknown` forces the developer to narrow before use, preventing runtime errors. `any` disables all checks, so the crash is not caught at compile time.

### Real-World Cases

- **Component props:** Every prop is typed to document its expected shape.
- **API responses:** Response types describe the shape returned by the server.
- **Event handlers:** Event types (`React.MouseEvent`, `React.ChangeEvent`) provide correct target types.
- **State:** `useState<User | null>(null)` ensures the state is either a `User` or `null`.
- **Refs:** `useRef<HTMLInputElement>(null)` types the ref's `current` property.

### References

- TypeScript Handbook – Everyday Types - https://www.typescriptlang.org/docs/handbook/2/everyday-types.html
- TypeScript Handbook – The Basics - https://www.typescriptlang.org/docs/handbook/2/basic-types.html
- React TypeScript Cheatsheet – Basic Types - https://react-typescript-cheatsheet.netlify.app/docs/basic/getting-started/basic_type_example
- TypeScript – `strict` - https://www.typescriptlang.org/tsconfig#strict
- TypeScript – `noImplicitAny` - https://www.typescriptlang.org/tsconfig#noImplicitAny
- TypeScript – `strictNullChecks` - https://www.typescriptlang.org/tsconfig#strictNullChecks
- MDN Web Docs – JavaScript Data Types - https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures

---

## Core Concept 2: Interfaces vs. Type Aliases

### Definitions

**Core Definition:** Interfaces and type aliases are two TypeScript constructs for naming object shapes; interfaces are open to declaration merging and are traditionally used for object-oriented APIs, while type aliases can name any type (unions, primitives, tuples, intersections) and are the modern preference for component props.

**Technical Definition:** An **interface** declares a named object type that can be extended (`extends`) and merged (multiple declarations with the same name are combined). A **type alias** (`type Name = ...`) creates a new name for any type expression, including unions, intersections, tuples, primitives, and functions. Both are structurally typed and erased at compile time. Interfaces support `implements` for classes; type aliases do not. Type aliases can express unions and conditional types, which interfaces cannot. In React, the community has largely converged on `type` for component props because it is more flexible and consistent, while `interface` remains common for public library APIs and objects that benefit from declaration merging.

**Beginner-Friendly Explanation:** An interface is like a contract for the shape of an object—other objects can "implement" it, and you can add to it later. A type alias is like a nickname for any type—it can describe objects, but also unions, tuples, and other types that interfaces cannot. For React props, most developers use `type` because it handles all the cases (including unions and intersections) with a single syntax.

### Purposes

- To name and document the shape of props, state, and API responses.
- To enable editor autocomplete and error checking for object properties.
- To share types across components, modules, and packages.
- To compose complex types from simpler ones (intersections, unions).
- To support `implements` for class-based APIs (interfaces only).
- To allow declaration merging for library augmentation (interfaces only).

### Syntax Rules and Structure

**Interface:**
```typescript
interface User {
  id: number;
  name: string;
  email?: string;
}

interface Admin extends User {
  role: 'admin' | 'superadmin';
}

// Declaration merging
interface Window {
  __APP_VERSION__: string;
}
```

**Type Alias:**
```typescript
type User = {
  id: number;
  name: string;
  email?: string;
};

type Admin = User & { role: 'admin' | 'superadmin' };

// Unions (interfaces cannot express this)
type Status = 'idle' | 'loading' | 'success' | 'error';

// Tuples
type Coordinate = [number, number];
```
\
**React Component Props — Both Work:**

Using interface
```tsx
interface ButtonProps {
  label: string;
  variant?: 'primary' | 'secondary';
  onClick: () => void;
}

function Button({ label, variant = 'primary', onClick }: ButtonProps) {
  return <button className={`btn-${variant}`} onClick={onClick}>{label}</button>;
}
```
\
Using type
```tsx
type ButtonProps = {
  label: string;
  variant?: 'primary' | 'secondary';
  onClick: () => void;
};
```
\
**Extending and Intersecting:**

Interface extends
```ts
interface BaseProps { 
    id: string; 
}

interface ButtonProps extends BaseProps { 
    label: string; 
}
```
\
Type intersection
```ts
type BaseProps = { 
    id: string 
};

type ButtonProps = BaseProps & { 
    label: string 
};
```
\
Type union (interface cannot do this)
```ts
type InputProps = 
  | { type: 'text'; value: string }
  | { type: 'number'; value: number };
```

**Syntax Rules:**
- Use `interface` for public APIs, object-oriented contracts, and types that benefit from declaration merging.
- Use `type` for component props, unions, intersections, tuples, and anything that is not a plain object shape.
- Prefer consistency: choose one style for component props within a codebase (the React community leans toward `type`).
- Interfaces can only describe object shapes (and functions); type aliases can describe any type.
- Interfaces cannot be used in unions directly (`type A = Interface1 | Interface2` is valid, but `interface A = ... | ...` is not).
- Type aliases cannot be re-opened for declaration merging.
- Both are erased at runtime; there is no performance difference.

**Constraints and Limitations:**
- Declaration merging can cause surprising type augmentation; use it deliberately.
- Interfaces cannot express unions, tuples, or conditional types.
- Type aliases cannot be implemented by classes (`class Foo implements Bar` requires `Bar` to be an interface or object type).
- Excess property checks apply to object literals assigned to either interfaces or type aliases.
- Some third-party libraries require `interface` for module augmentation.

### Annotated Code Examples

**Example 1: Composing Props with Intersection**

```tsx
type BaseButtonProps = {
  variant?: 'primary' | 'secondary' | 'danger';
  size?: 'sm' | 'md' | 'lg';
};

type ButtonProps = BaseButtonProps & {
  label: string;
  onClick: () => void;
  icon?: React.ReactNode;
};

function Button({ variant = 'primary', size = 'md', label, onClick, icon }: ButtonProps) {
  return (
    <button 
      className={`btn btn-${variant} btn-${size}`} 
      onClick={onClick}
    >
      {icon}{label}
    </button>
  );
}
```

**Expected Output:** A button component whose props are the intersection of `BaseButtonProps` and the additional button-specific props. TypeScript merges both sets; passing `label` without `variant` uses the default.

**Why This Output Occurs:** The `&` operator creates an intersection type that combines both shapes. `variant` and `size` are optional; `label` and `onClick` are required.

**Example 2: Discriminated Union of Props**

```tsx
type InputProps =
  | { type: 'text'; value: string; onChange: (v: string) => void }
  | { type: 'number'; value: number; onChange: (v: number) => void }
  | { type: 'checkbox'; checked: boolean; onChange: (v: boolean) => void };

function Input(props: InputProps) {
  if (props.type === 'text') {
    return <input type="text" value={props.value} onChange={(e) => props.onChange(e.target.value)} />;
  }
  if (props.type === 'number') {
    return <input type="number" value={props.value} onChange={(e) => props.onChange(Number(e.target.value))} />;
  }
  return <input type="checkbox" checked={props.checked} onChange={(e) => props.onChange(e.target.checked)} />;
}
```

**Expected Output:** The `Input` component accepts three different prop shapes discriminated by `type`. TypeScript narrows the prop type inside each branch, so `props.value` is `string` in the `text` branch and `number` in the `number` branch.

**Why This Output Occurs:** The discriminated union uses the `type` literal as the discriminant. TypeScript narrows the union based on the `type` check, ensuring that each branch accesses only the valid properties.

### Real-World Cases

- **Design systems:** `type` for props, `interface` for public component APIs.
- **API clients:** `interface` for module augmentation and `type` for response shapes.
- **State machines:** `type` for discriminated unions of state shapes.
- **Library authoring:** `interface` for extensible public APIs.
- **Component libraries:** `type` for props, `interface` for refs and imperative handles.

### References

- TypeScript Handbook – Interfaces - https://www.typescriptlang.org/docs/handbook/2/objects.html
- TypeScript Handbook – Type Aliases - https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#type-aliases
- TypeScript Handbook – Differences Between Type Aliases and Interfaces - https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#differences-between-type-aliases-and-interfaces
- React TypeScript Cheatsheet – Types vs Interfaces - https://react-typescript-cheatsheet.netlify.app/docs/basic/getting-started/basic_type_example#types-or-interfaces
- Matt Pocock – Type vs Interface: Which Should You Use? - https://www.totaltypescript.com/type-vs-interface-which-should-you-use

---

## Core Concept 3: Union Types & Literal Types

### Definitions

**Core Definition:** Union types describe a value that can be one of several types (`A | B`); literal types describe a value that must be exactly one specific literal (`'primary'`, `42`, `true`). Together, they enable precise, self-documenting component configuration options.

**Technical Definition:** A union type (`string | number`) means the value is either a `string` or a `number`. A literal type is a type whose only possible value is a specific literal (`'primary'` is a string literal type; `42` is a numeric literal type). When a union contains only literal types, it is a **literal union** (`'sm' | 'md' | 'lg'`), which is the idiomatic way to type component variants, sizes, and statuses. TypeScript infers literal types when you use `const` and when you annotate with `as const`. Discriminated unions combine literal types with object shapes to model variants, and the discriminant property enables exhaustive type narrowing in `switch` statements.

**Beginner-Friendly Explanation:** A union type says "this value can be one of these types"—like `string | number` for a value that could be either. A literal type says "this value must be exactly this"—like `'primary'`, `'secondary'`, or `'danger'` for a button variant. Together, they let you say "this prop must be one of these exact strings," which gives you autocomplete and catches typos at compile time.

### Purposes

- To restrict a prop to a specific set of values (variants, sizes, statuses).
- To model values that can be one of several types.
- To enable exhaustive type checking via discriminated unions.
- To provide autocomplete for valid values in the editor.
- To prevent invalid values from being passed to components.
- To document the valid options for a component's configuration.

### Syntax Rules and Structure

**Union Types:**
```typescript
let id: string | number;
id = 'abc';   // OK
id = 123;     // OK
// id = true; // Error: boolean is not assignable
```

**Literal Types:**
```typescript
let status: 'idle';
status = 'idle';  // OK
// status = 'loading'; // Error

// Literal union
let status: 'idle' | 'loading' | 'success' | 'error';
status = 'loading'; // OK
```

**Component Variants (Literal Union):**
```tsx
type ButtonProps = {
  variant: 'primary' | 'secondary' | 'danger' | 'ghost';
  size?: 'sm' | 'md' | 'lg';
  children: React.ReactNode;
  onClick?: () => void;
};

function Button({ variant, size = 'md', children, onClick }: ButtonProps) {
  return <button className={`btn btn-${variant} btn-${size}`} onClick={onClick}>{children}</button>;
}

// Usage
<Button variant="primary" size="lg">Click</Button>
// <Button variant="purple">Click</Button> // Error: not a valid variant
```

**Discriminated Union:**
```tsx
type Notification =
  | { type: 'success'; message: string }
  | { type: 'error'; message: string; code: number }
  | { type: 'info'; message: string; dismissible?: boolean };

function NotificationBanner({ notification }: { notification: Notification }) {
  switch (notification.type) {
    case 'success':
      return <div role="status">{notification.message}</div>;
    case 'error':
      return <div role="alert">{notification.message} (Code: {notification.code})</div>;
    case 'info':
      return (
        <div role="status">
          {notification.message}
          {notification.dismissible && <button>Dismiss</button>}
        </div>
      );
  }
}
```

**`as const` for Tuple and Object Literals:**
```typescript
const SIZES = ['sm', 'md', 'lg'] as const;
type Size = typeof SIZES[number]; // 'sm' | 'md' | 'lg'

const CONFIG = {
  variant: 'primary',
  size: 'md',
} as const;
// typeof CONFIG.variant = 'primary' (literal, not string)
```

**Syntax Rules:**
- Use `|` to combine types into a union.
- Use literal unions for variants, sizes, statuses, and other fixed sets.
- Use a discriminant property (`type`, `kind`, `status`) to model variants.
- Use `switch` on the discriminant for exhaustive checking; add a `default` that assigns to `never` to catch missing cases.
- Use `as const` to preserve literal types in arrays and objects.
- Derive union types from `as const` arrays with `typeof ARR[number]`.
- Prefer literal unions over enums for component props (more flexible, no runtime overhead).

**Constraints and Limitations:**
- Union types do not automatically narrow; you must use type guards (`typeof`, `in`, discriminant checks).
- `as const` makes the entire object deeply readonly.
- Enums produce runtime objects; literal unions do not.
- Union types can become unwieldy if they have many members; consider discriminated unions.
- Excess property checks apply to union object literals, which can be surprising.

### Annotated Code Examples

**Example 1: Button Variants with Literal Union**

```tsx
const BUTTON_VARIANTS = ['primary', 'secondary', 'danger', 'ghost'] as const;
type ButtonVariant = typeof BUTTON_VARIANTS[number];

type ButtonProps = {
  variant?: ButtonVariant;
  children: React.ReactNode;
  onClick?: () => void;
};

function Button({ variant = 'primary', children, onClick }: ButtonProps) {
  return <button className={`btn btn-${variant}`} onClick={onClick}>{children}</button>;
}

// Usage
<Button variant="primary">Save</Button>
<Button variant="danger" onClick={() => {}}>Delete</Button>
// <Button variant="purple">Save</Button> // Error: Type '"purple"' is not assignable
```

**Expected Output:** The button renders with the correct variant class. TypeScript autocompletes the four valid variants and errors on invalid ones. The `BUTTON_VARIANTS` array is the single source of truth.

**Why This Output Occurs:** `as const` makes the array readonly with literal types. `typeof BUTTON_VARIANTS[number]` extracts the union of its values. `ButtonVariant` is the union `'primary' | 'secondary' | 'danger' | 'ghost'`. TypeScript rejects any other string.

**Example 2: Exhaustive Discriminated Union**

```tsx
type Shape =
  | { kind: 'circle'; radius: number }
  | { kind: 'square'; side: number }
  | { kind: 'rectangle'; width: number; height: number };

function area(shape: Shape): number {
  switch (shape.kind) {
    case 'circle':
      return Math.PI * shape.radius ** 2;
    case 'square':
      return shape.side ** 2;
    case 'rectangle':
      return shape.width * shape.height;
    default: {
      const _exhaustive: never = shape;
      throw new Error(`Unhandled shape: ${_exhaustive}`);
    }
  }
}
```

**Expected Output:** `area` computes the area for each shape. If a new shape is added to the union and not handled in the `switch`, TypeScript errors on the `never` assignment in the `default` case, catching the missing case at compile time.

**Why This Output Occurs:** The `never` type is assignable only to `never`. If all cases are handled, `shape` is narrowed to `never` in the `default` branch, and the assignment is valid. If a case is missing, `shape` retains that type, and the assignment errors.

### Real-World Cases

- **Design system variants:** `variant`, `size`, `tone`, `state` props.
- **Status indicators:** `'idle' | 'loading' | 'success' | 'error'`.
- **API state machines:** Discriminated unions for request lifecycles.
- **Form field types:** `'text' | 'number' | 'email' | 'password'`.
- **Layout options:** `'row' | 'column'`, `'start' | 'center' | 'end'`.

### References

- TypeScript Handbook – Unions and Intersections - https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#union-types
- TypeScript Handbook – Literal Types - https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#literal-types
- TypeScript Handbook – Discriminated Unions - https://www.typescriptlang.org/docs/handbook/2/narrowing.html#discriminated-unions
- TypeScript Handbook – `as const` - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-4.html#const-assertions
- React TypeScript Cheatsheet – Union Types - https://react-typescript-cheatsheet.netlify.app/docs/basic/getting-started/basic_type_example
- Matt Pocock – Literal Types - https://www.totaltypescript.com/books/total-typescript-essentials/literal-types

---

## Core Concept 4: Generics

### Definitions

**Core Definition:** Generics are type parameters that let you write functions, components, and types that work with any type while preserving type safety—the type is specified or inferred at the call site.

**Technical Definition:** A generic is declared with angle brackets (`<T>`) and can be used anywhere a type is expected. `function identity<T>(value: T): T { return value; }` works for any `T`, and TypeScript infers `T` from the argument. Generics can be constrained (`<T extends object>`), defaulted (`<T = string>`), and combined (`<T, U>`). In React, generics are used for reusable components that work with any data type (lists, tables, selects), custom hooks that return typed values, and context providers. Generic components are written as `function Component<T>(props: Props<T>)` or with arrow functions using a trailing comma (`const Component = <T,>(props: Props<T>) => ...`). TypeScript infers `T` from the props, so consumers rarely need to specify it explicitly.

**Beginner-Friendly Explanation:** A generic is a placeholder for a type. Instead of writing one function for strings and another for numbers, you write `function identity<T>(value: T): T` and TypeScript figures out `T` from what you pass in. In React, a generic list component can render a list of users, products, or anything else, and TypeScript still knows the exact type of each item. Generics make components reusable without sacrificing type safety.

### Purposes

- To write reusable functions, components, and hooks that work with any type.
- To preserve type information through transformations (map, filter, etc.).
- To avoid duplicating code for each type.
- To provide type-safe APIs for data-driven components (lists, tables, selects).
- To constrain types while remaining flexible.
- To enable type inference at the call site.

### Syntax Rules and Structure

**Generic Function:**
```typescript
function identity<T>(value: T): T {
  return value;
}

const str = identity('hello');   // T = string
const num = identity(42);        // T = number
```

**Generic with Constraint:**
```typescript
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user = { name: 'Alice', age: 30 };
const name = getProperty(user, 'name'); // string
const age = getProperty(user, 'age');   // number
// getProperty(user, 'email'); // Error: 'email' is not a key of user
```

**Generic Component:**
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

// Usage — T inferred as { id: number; name: string }
<List
  items={[{ id: 1, name: 'Alice' }, { id: 2, name: 'Bob' }]}
  renderItem={(user) => <span>{user.name}</span>}
  keyExtractor={(user) => user.id}
/>
```

**Generic Arrow Component (with trailing comma):**
```tsx
const List = <T,>({ items, renderItem }: ListProps<T>) => {
  return <ul>{items.map(renderItem)}</ul>;
};
// The trailing comma disambiguates from JSX
```

**Generic Custom Hook:**
```typescript
function useLocalStorage<T>(key: string, initialValue: T) {
  const [value, setValue] = useState<T>(() => {
    const stored = localStorage.getItem(key);
    return stored ? (JSON.parse(stored) as T) : initialValue;
  });

  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value));
  }, [key, value]);

  return [value, setValue] as const;
}

// Usage
const [user, setUser] = useLocalStorage<User | null>('user', null);
```

**Generic with Default:**
```typescript
type ApiResponse<T = unknown> = {
  data: T;
  status: number;
  message?: string;
};

const response: ApiResponse<User> = { data: user, status: 200 };
const generic: ApiResponse = { data: anything, status: 200 }; // T = unknown
```

**Syntax Rules:**
- Declare generics with `<T>` (or `<T, U>`, `<T extends Constraint>`).
- Use `extends` to constrain what types are allowed.
- Use `keyof T` for keys of an object type.
- Use `T[K]` for indexed access types.
- In `.tsx` files, use `<T,>` for arrow function generics to avoid JSX ambiguity.
- Let TypeScript infer generic arguments from usage; only specify explicitly when inference fails.
- Use generics for components, hooks, and utility functions that operate on "any type."
- Use `as const` on the returned tuple to preserve the tuple type (`return [value, setValue] as const`).

**Constraints and Limitations:**
- Generic components cannot be memoised with `React.memo` directly while preserving generics; use a workaround (type cast or `memo` with a generic wrapper).
- Generics do not exist at runtime; they are erased like all TypeScript types.
- Overly complex generics reduce readability; use constraints and defaults to keep them manageable.
- TypeScript's inference may need explicit help in some cases (`<List<User> ... />`).
- Generic arrow functions in `.tsx` need a trailing comma (`<T,>`) to distinguish from JSX.

### Annotated Code Examples

**Example 1: Generic List Component**

```tsx
type ListProps<T> = {
  items: T[];
  renderItem: (item: T, index: number) => React.ReactNode;
  emptyMessage?: string;
};

function List<T>({ items, renderItem, emptyMessage = 'No items' }: ListProps<T>) {
  if (items.length === 0) return <p>{emptyMessage}</p>;
  return <ul>{items.map((item, index) => <li key={index}>{renderItem(item, index)}</li>)}</ul>;
}

// Usage with different types
<List items={['a', 'b', 'c']} renderItem={(s) => <strong>{s}</strong>} />
<List items={[{ id: 1, name: 'Alice' }]} renderItem={(u) => <span>{u.name}</span>} />
```

**Expected Output:** The same `List` component renders a list of strings and a list of objects. TypeScript infers `T = string` for the first and `T = { id: number; name: string }` for the second, so `renderItem`'s parameter is correctly typed in each case.

**Why This Output Occurs:** `<T>` declares a type parameter. `items: T[]` makes the component generic. TypeScript infers `T` from the `items` prop and applies it to `renderItem`, so `s` is a `string` and `u` is the user object.

**Example 2: Generic Select Component**

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

// Usage
type User = { id: number; name: string };
<Select<User>
  options={users}
  value={selectedUser}
  onChange={setSelectedUser}
  getLabel={(u) => u.name}
  getValue={(u) => u.id}
/>
```

**Expected Output:** A type-safe select component. TypeScript ensures that `value`, `onChange`, `getLabel`, and `getValue` all work with the same `T`. Passing a `User` option with a string `id` would error if `getValue` expected a number.

**Why This Output Occurs:** The generic `T` is inferred from `options`. All other props are typed in terms of `T`, so `onChange` receives a `User`, and `getValue` returns the user's ID type.

### Real-World Cases

- **Data tables:** Generic table components with typed column definitions.
- **Select and combobox components:** Generic options with typed values.
- **Lists and grids:** Generic rendering with type-safe item props.
- **Custom hooks:** `useLocalStorage<T>`, `useFetch<T>`, `useForm<T>`.
- **API clients:** Generic request/response typing.
- **Context providers:** Generic context values.

### References

- TypeScript Handbook – Generics - https://www.typescriptlang.org/docs/handbook/2/generics.html
- TypeScript Handbook – Generic Constraints - https://www.typescriptlang.org/docs/handbook/2/generics.html#generic-constraints
- TypeScript Handbook – `keyof` Type Operator - https://www.typescriptlang.org/docs/handbook/2/keyof-types.html
- TypeScript Handbook – Indexed Access Types - https://www.typescriptlang.org/docs/handbook/2/indexed-access-types.html
- React TypeScript Cheatsheet – Generic Components - https://react-typescript-cheatsheet.netlify.app/docs/advanced/patterns_by_usecase/#generic-components
- Matt Pocock – Generics - https://www.totaltypescript.com/books/total-typescript-essentials/generics

---

## Core Concept 5: Type Narrowing & Type Guards

### Definitions

**Core Definition:** Type narrowing is TypeScript's process of refining a broad type (e.g., `string | number`) to a more specific type within a conditional block; type guards are the expressions (`typeof`, `instanceof`, `in`, custom predicates) that trigger narrowing.

**Technical Definition:** TypeScript's control-flow analysis tracks the type of a value through branches. When a condition proves that a value is of a specific type, TypeScript narrows the type within that branch. Built-in type guards include `typeof` (for primitives), `instanceof` (for classes), `in` (for property existence), truthiness checks, equality checks, and discriminated union checks. Custom type guards use a type predicate (`value is Type`) as the return type of a function, telling TypeScript that if the function returns `true`, the value is of the specified type. The `asserts value is Type` form is an assertion function, which narrows the value for the rest of the enclosing scope.

**Beginner-Friendly Explanation:** Type narrowing is TypeScript figuring out "if this check passes, then the value must be this type." For example, if you check `typeof value === 'string'`, TypeScript knows that inside the `if` block, `value` is a `string`. Type guards are the tools you use to make those checks—`typeof` for primitives, `instanceof` for classes, `in` for objects, and custom functions that return `value is Type`.

### Purposes

- To safely access properties that exist only on specific types in a union.
- To handle API responses and user input that may be of uncertain type.
- To enable exhaustive handling of discriminated unions.
- To encapsulate complex narrowing logic in reusable functions.
- To satisfy TypeScript's strict null checks.
- To narrow `unknown` values to specific types.

### Syntax Rules and Structure

**`typeof` (Primitives):**
```typescript
function format(value: string | number): string {
  if (typeof value === 'string') {
    return value.toUpperCase(); // value: string
  }
  return value.toFixed(2);      // value: number
}
```

**`instanceof` (Classes):**
```typescript
function logError(error: Error | string) {
  if (error instanceof Error) {
    console.log(error.message); // error: Error
  } else {
    console.log(error);         // error: string
  }
}
```

**`in` (Property Existence):**
```typescript
type Admin = { role: 'admin'; permissions: string[] };
type User = { role: 'user'; email: string };

function describe(account: Admin | User) {
  if ('permissions' in account) {
    return `Admin with ${account.permissions.length} permissions`;
  }
  return `User with email ${account.email}`;
}
```

**Discriminated Union:**
```typescript
type Result =
  | { status: 'success'; data: string }
  | { status: 'error'; error: Error };

function handle(result: Result) {
  if (result.status === 'success') {
    console.log(result.data);  // result: success variant
  } else {
    console.log(result.error); // result: error variant
  }
}
```

**Custom Type Guard (Predicate):**
```typescript
type Fish = { swim: () => void };
type Bird = { fly: () => void };

function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined;
}

function move(pet: Fish | Bird) {
  if (isFish(pet)) {
    pet.swim(); // pet: Fish
  } else {
    pet.fly();  // pet: Bird
  }
}
```

**Assertion Function:**
```typescript
function assertIsString(value: unknown): asserts value is string {
  if (typeof value !== 'string') {
    throw new TypeError('Expected a string');
  }
}

function process(value: unknown) {
  assertIsString(value);
  console.log(value.toUpperCase()); // value: string
}
```

**React Example — Narrowing `unknown` API Data:**
```tsx
type User = { id: number; name: string };

function isUser(value: unknown): value is User {
  return (
    typeof value === 'object' &&
    value !== null &&
    'id' in value &&
    typeof (value as User).id === 'number' &&
    'name' in value &&
    typeof (value as User).name === 'string'
  );
}

function UserProfile({ data }: { data: unknown }) {
  if (!isUser(data)) {
    return <p>Invalid user data</p>;
  }
  return <h1>{data.name}</h1>; // data: User
}
```

**Syntax Rules:**
- Use `typeof` for `string`, `number`, `boolean`, `symbol`, `bigint`, `undefined`, `function`, and `object`.
- Use `instanceof` for class instances (including `Error`, `Date`, `Array`, etc.).
- Use `in` to narrow by property existence.
- Use discriminated unions with a literal discriminant property for the cleanest narrowing.
- Use custom type predicates (`value is Type`) for reusable narrowing logic.
- Use assertion functions (`asserts value is Type`) for narrowing that should throw.
- Add a `default` case with `never` to ensure exhaustive handling.
- Always narrow `unknown` before use; never cast it with `as` without validation.

**Constraints and Limitations:**
- `typeof null === 'object'`; always check `value !== null` explicitly.
- `instanceof` does not work across realms (iframes, workers).
- Custom type predicates are not validated by TypeScript; an incorrect predicate lies to the type system.
- Assertion functions require an explicit type annotation on the function signature.
- Narrowing does not persist across closures unless the value is captured in a `const`.
- Excess property checks apply to object literals assigned to union types.

### Annotated Code Examples

**Example 1: Safe API Response Handler**

```tsx
type ApiResponse =
  | { status: 'success'; data: { users: string[] } }
  | { status: 'error'; error: { message: string; code: number } };

function isSuccess(response: ApiResponse): response is Extract<ApiResponse, { status: 'success' }> {
  return response.status === 'success';
}

function UserList({ response }: { response: ApiResponse }) {
  if (isSuccess(response)) {
    return <ul>{response.data.users.map((u) => <li key={u}>{u}</li>)}</ul>;
  }
  return <p role="alert">{response.error.message} (Code: {response.error.code})</p>;
}
```

**Expected Output:** If `response.status === 'success'`, the component renders the user list and TypeScript narrows `response.data.users` to `string[]`. Otherwise, the error branch narrows to the error variant.

**Why This Output Occurs:** The `Extract<ApiResponse, { status: 'success' }>` utility extracts the success variant. The type predicate `response is ...` tells TypeScript that a `true` return narrows the value. The `status` discriminant drives the narrowing.

**Example 2: Narrowing `unknown` with Multiple Guards**

```tsx
function processInput(input: unknown): string {
  if (typeof input === 'string') return input.trim();
  if (typeof input === 'number') return input.toFixed(2);
  if (Array.isArray(input)) return input.join(', ');
  if (input instanceof Date) return input.toISOString();
  if (typeof input === 'object' && input !== null && 'toString' in input) {
    return String(input);
  }
  return 'Unknown input';
}
```

**Expected Output:** `processInput` handles strings, numbers, arrays, dates, and objects with `toString`. Each branch narrows the `unknown` to a specific type, and TypeScript allows the corresponding operations.

**Why This Output Occurs:** Each `typeof`, `Array.isArray`, `instanceof`, and `in` check narrows the `unknown` type. TypeScript's control-flow analysis tracks the narrowing through each branch.

### Real-World Cases

- **API response parsing:** Validating that a response matches an expected shape.
- **Event handlers:** Narrowing `Event` to `MouseEvent`, `KeyboardEvent`, or `ChangeEvent`.
- **Form validation:** Narrowing form values from `string | number | undefined` to specific types.
- **Feature flags:** Narrowing `unknown` config values.
- **Error handling:** Distinguishing between different error types.

### References

- TypeScript Handbook – Narrowing - https://www.typescriptlang.org/docs/handbook/2/narrowing.html
- TypeScript Handbook – Type Guards - https://www.typescriptlang.org/docs/handbook/2/narrowing.html#typeof-type-guards
- TypeScript Handbook – Custom Type Guards - https://www.typescriptlang.org/docs/handbook/2/narrowing.html#using-type-predicates
- TypeScript Handbook – Assertion Functions - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-7.html#assertion-functions
- TypeScript Handbook – Exhaustiveness Checking - https://www.typescriptlang.org/docs/handbook/2/narrowing.html#exhaustiveness-checking
- React TypeScript Cheatsheet – Type Guards - https://react-typescript-cheatsheet.netlify.app/docs/basic/getting-started/basic_type_example

---

## Core Concept 6: Type Assertions & Type Casting

### Definitions

**Core Definition:** Type assertions (`as Type`) and non-null assertions (`!`) tell TypeScript to treat a value as a different type than it inferred, overriding the compiler's analysis without performing any runtime check.

**Technical Definition:** The `as` syntax (`value as Type`) asserts that `value` is of type `Type`. TypeScript allows assertions between types that have some overlap (or via `as unknown as Type` for unrelated types, which is a code smell). The non-null assertion operator (`value!`) asserts that `value` is neither `null` nor `undefined`, removing those types from the union. Both are compile-time-only constructs: they do not validate anything at runtime, and if the assertion is wrong, the error will surface at runtime (often as a crash). Because they bypass type safety, they should be used sparingly—prefer type guards and narrowing. Common legitimate uses include DOM refs (`inputRef.current!`), event targets, and third-party library types with incomplete definitions.

**Beginner-Friendly Explanation:** A type assertion is you telling TypeScript "trust me, I know what this type is." TypeScript will not check—it just believes you. If you are right, everything is fine. If you are wrong, you get a runtime crash. The non-null assertion (`!`) is the same idea: "trust me, this is not null." Use assertions only when you have information TypeScript does not (e.g., a DOM ref that is guaranteed to exist), and prefer type guards when you can prove the type at runtime.

### Purposes

- To narrow a type when TypeScript cannot infer it from context.
- To access DOM elements via refs (`ref.current!`).
- To work with third-party libraries whose types are incomplete.
- To override an incorrect inferred type (rare; prefer fixing the inference).
- To convert between related types (e.g., `string` to a literal union).
- To handle `null`/`undefined` when you have external guarantees.

### Syntax Rules and Structure

**`as` Assertion:**
```typescript
const value: unknown = 'hello';
const str = value as string;
console.log(str.toUpperCase()); // OK at compile time
```

**`as const` (Different from `as Type`):**
```typescript
const config = { theme: 'dark' } as const;
// typeof config.theme = 'dark' (literal)
```

**Non-Null Assertion (`!`):**
```typescript
const inputRef = useRef<HTMLInputElement>(null);

function focusInput() {
  inputRef.current!.focus(); // Asserts current is not null
}
```

**DOM Refs in React:**
```tsx
function AutoFocusInput() {
  const inputRef = useRef<HTMLInputElement>(null);

  useEffect(() => {
    inputRef.current?.focus();      // Optional chaining (safe)
    // inputRef.current!.focus();   // Non-null assertion (crashes if null)
  }, []);

  return <input ref={inputRef} />;
}
```

**Event Target Narrowing:**
```tsx
function handleChange(e: React.ChangeEvent<HTMLInputElement>) {
  const value = e.target.value; // Already typed correctly
}

// ❌ Avoid: asserting the event target
function handleClick(e: React.MouseEvent) {
  const target = e.target as HTMLButtonElement; // Unsafe
}
```

**`as unknown as Type` (Double Assertion):**
```typescript
// ❌ Rarely justified
const value = 'hello' as unknown as number;

// ✅ Better: narrow with a type guard or fix the type
```

**Type Assertion vs. Type Annotation:**
```typescript
// Annotation: declares the type
const a: string = 'hello';

// Assertion: overrides the inferred type
const b = 'hello' as string;
```

**Syntax Rules:**
- Use `as Type` to assert a value's type; prefer narrowing via type guards when possible.
- Use `value!` only when you are certain the value is not `null` or `undefined`.
- Use `as const` to preserve literal types (different from `as Type`).
- Use optional chaining (`?.`) instead of `!` when the value might genuinely be null.
- Avoid `as unknown as Type`; it usually indicates a design problem.
- Assertions are erased at runtime; they do not perform validation.
- Prefer `unknown` + type guards over `as Type` for API data.
- In React, prefer conditional rendering (`{ref.current && ...}`) over `!` where possible.

**Constraints and Limitations:**
- Assertions do not perform runtime checks; wrong assertions cause runtime errors.
- `as` only works between types with some overlap; `as unknown as Type` bypasses this but is unsafe.
- Non-null assertions can hide bugs; if the value is actually `null`, the app crashes.
- TypeScript may warn about unnecessary assertions when the type is already correct.
- Assertions do not narrow; they replace the type entirely.
- In React 19, `ref` is passed as a regular prop, and `useRef` requires an initial value; the `!` operator is still needed for DOM refs in some cases.

### Annotated Code Examples

**Example 1: DOM Ref with Non-Null Assertion vs. Optional Chaining**

```tsx
import { useRef, useEffect } from 'react';

function SearchInput() {
  const inputRef = useRef<HTMLInputElement>(null);

  useEffect(() => {
    inputRef.current?.focus();     // ✅ Safe: optional chaining
  }, []);

  function handleClick() {
    inputRef.current!.focus();     // ⚠️ Unsafe if ref is not attached yet
  }

  return (
    <div>
      <input ref={inputRef} type="search" />
      <button onClick={handleClick}>Focus</button>
    </div>
  );
}
```

**Expected Output:** On mount, the input is focused via `inputRef.current?.focus()` (safe). Clicking the button calls `inputRef.current!.focus()`; since the input is always rendered, the ref is non-null and this works. If the input were conditionally rendered and not present, the `!` assertion would crash.

**Why This Output Occurs:** The `!` operator tells TypeScript to remove `null` from the type of `inputRef.current`. It does not check at runtime. The `?.` operator is a runtime check that short-circuits if `current` is `null`.

**Example 2: Asserting a Literal Union from a String**

```tsx
type Theme = 'light' | 'dark';

function isTheme(value: string): value is Theme {
  return value === 'light' || value === 'dark';
}

function applyTheme(value: string) {
  // ❌ Unsafe assertion
  const theme = value as Theme;
  document.body.dataset.theme = theme;

  // ✅ Safe: type guard
  if (isTheme(value)) {
    document.body.dataset.theme = value;
  }
}
```

**Expected Output:** The assertion version applies any string as the theme, which could set `data-theme="purple"` if the caller passes an invalid value. The type guard version only applies valid themes, silently ignoring invalid values.

**Why This Output Occurs:** `value as Theme` tells TypeScript to trust the caller. `isTheme(value)` performs a runtime check and narrows the type only if the check passes.

### Real-World Cases

- **DOM refs:** `inputRef.current!.focus()` in event handlers.
- **Event targets:** `(e.target as HTMLInputElement).value` in change handlers.
- **Third-party libraries:** Asserting types for libraries with incomplete definitions.
- **API responses:** Asserting types after runtime validation (with Zod, io-ts).
- **Feature flags:** Asserting `unknown` config values after checking.

### References

- TypeScript Handbook – Type Assertions - https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#type-assertions
- TypeScript Handbook – Non-null Assertion Operator - https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#non-null-assertion-operator-postfix-
- TypeScript Handbook – `as const` - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-4.html#const-assertions
- React TypeScript Cheatsheet – Type Assertions - https://react-typescript-cheatsheet.netlify.app/docs/basic/getting-started/basic_type_example
- React TypeScript Cheatsheet – Refs - https://react-typescript-cheatsheet.netlify.app/docs/basic/getting-started/hooks#useref
- Matt Pocock – Type Assertions - https://www.totaltypescript.com/books/total-typescript-essentials/type-assertions

---

## Comparison and Decision Guidance

| Concept | When to Use | When to Avoid | Key Risk |
|---|---|---|---|
| **Basic types** | Everywhere; annotate all public APIs | Over-annotating local variables (inference is fine) | Using `any` instead of `unknown` |
| **`interface`** | Public APIs, declaration merging, `implements` | Unions, tuples, complex type expressions | Overusing merging |
| **`type` alias** | Component props, unions, tuples, intersections | When `interface` merging is needed | Inconsistent style |
| **Union types** | Values that can be one of several types | Overly broad unions | Missing narrowing |
| **Literal types** | Variants, sizes, statuses, fixed sets | Large sets (use enums or constants) | Typos in strings |
| **Generics** | Reusable components, hooks, utilities | Simple components with fixed types | Over-abstraction |
| **Type guards** | Narrowing `unknown`, union types, API data | When assertion is justified by external knowledge | Incorrect predicates |
| **Type assertions** | DOM refs, third-party types, after validation | API data, user input, complex types | Runtime crashes if wrong |
| **Non-null assertion** | DOM refs in event handlers | Values that might genuinely be null | Hiding null bugs |

**Decision Guidance:**
- **Start with inference:** Let TypeScript infer types for local variables; annotate public APIs.
- **Prefer `unknown` over `any`:** Force narrowing; catch errors at compile time.
- **Use `type` for component props:** More flexible than `interface` for unions and intersections.
- **Use literal unions for variants:** Get autocomplete and catch typos.
- **Use generics for reusable components:** Preserve type safety across data types.
- **Use type guards for `unknown`:** Never cast `unknown` with `as` without validation.
- **Use assertions sparingly:** Prefer optional chaining and conditional rendering over `!`.
- **Enable `strict: true`:** Catch null/undefined bugs and implicit `any`.
- **Add a `default: never` case:** Ensure exhaustive handling of discriminated unions.

---

## References

- TypeScript Handbook - https://www.typescriptlang.org/docs/handbook/intro.html
- TypeScript Handbook – Everyday Types - https://www.typescriptlang.org/docs/handbook/2/everyday-types.html
- TypeScript Handbook – Narrowing - https://www.typescriptlang.org/docs/handbook/2/narrowing.html
- TypeScript Handbook – Generics - https://www.typescriptlang.org/docs/handbook/2/generics.html
- TypeScript Handbook – Interfaces - https://www.typescriptlang.org/docs/handbook/2/objects.html
- TypeScript Handbook – Type Aliases - https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#type-aliases
- TypeScript Handbook – Union Types - https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#union-types
- TypeScript Handbook – Literal Types - https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#literal-types
- TypeScript Handbook – Discriminated Unions - https://www.typescriptlang.org/docs/handbook/2/narrowing.html#discriminated-unions
- TypeScript Handbook – Type Assertions - https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#type-assertions
- TypeScript Handbook – Non-null Assertion Operator - https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#non-null-assertion-operator-postfix-
- TypeScript Handbook – `as const` - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-4.html#const-assertions
- TypeScript Handbook – Exhaustiveness Checking - https://www.typescriptlang.org/docs/handbook/2/narrowing.html#exhaustiveness-checking
- TypeScript Handbook – Assertion Functions - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-7.html#assertion-functions
- React TypeScript Cheatsheet - https://react-typescript-cheatsheet.netlify.app/
- React TypeScript Cheatsheet – Basic Types - https://react-typescript-cheatsheet.netlify.app/docs/basic/getting-started/basic_type_example
- React TypeScript Cheatsheet – Types vs Interfaces - https://react-typescript-cheatsheet.netlify.app/docs/basic/getting-started/basic_type_example#types-or-interfaces
- React TypeScript Cheatsheet – Generic Components - https://react-typescript-cheatsheet.netlify.app/docs/advanced/patterns_by_usecase/#generic-components
- React TypeScript Cheatsheet – Refs - https://react-typescript-cheatsheet.netlify.app/docs/basic/getting-started/hooks#useref
- React Official Documentation – TypeScript - https://react.dev/learn/typescript
- Matt Pocock – Total TypeScript - https://www.totaltypescript.com/
- Matt Pocock – Type vs Interface - https://www.totaltypescript.com/type-vs-interface-which-should-you-use
- Matt Pocock – Literal Types - https://www.totaltypescript.com/books/total-typescript-essentials/literal-types
- Matt Pocock – Generics - https://www.totaltypescript.com/books/total-typescript-essentials/generics
- Matt Pocock – Type Assertions - https://www.totaltypescript.com/books/total-typescript-essentials/type-assertions
- MDN Web Docs – JavaScript Data Types - https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures