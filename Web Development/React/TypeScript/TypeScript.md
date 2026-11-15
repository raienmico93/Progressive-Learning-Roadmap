# React TypeScript Fundamentals — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** React TypeScript fundamentals are the core language features and configuration practices of TypeScript that enable type-safe React application development — from basic type annotations and interfaces to generics, type narrowing, and compiler configuration.

**Technical Definition:** TypeScript is a statically typed superset of JavaScript that adds optional type annotations, interfaces, generics, and advanced type operators to the language. In a React context, TypeScript validates component props, state, event handlers, and hook return values at compile time, catching errors before runtime. The fundamentals include the structural type system (types, interfaces, unions, intersections), generic programming (parameterized types for reusable functions and components), type inference and narrowing (control-flow-based type refinement), utility types (built-in type transformations like `Partial`, `Pick`, `Omit`, `Record`), and strict compiler configuration (`strict: true`, `noUncheckedIndexedAccess`, `exactOptionalPropertyTypes`) that maximizes type safety.

**Beginner-Friendly Explanation:** TypeScript adds "types" to JavaScript — labels that describe what kind of data each variable, function parameter, and component prop should hold. React TypeScript fundamentals are the building blocks: how to describe objects (`interface`), create flexible types (`unions`, `generics`), let TypeScript figure out types automatically (`inference`), and configure the compiler to catch mistakes (`tsconfig`).

### Key Characteristics

- **Structural typing:** TypeScript compares types by their shape, not their name — if it looks like a `User`, it is a `User`.
- **Compile-time safety:** Errors are caught during development and build, not at runtime in production.
- **Inference-first:** TypeScript infers types wherever possible; explicit annotations are reserved for cases where inference is insufficient.
- **Strict mode by default:** Modern React TypeScript projects enable `strict: true` plus additional safety flags like `noUncheckedIndexedAccess` and `exactOptionalPropertyTypes`.
- **Ecosystem integration:** TypeScript integrates with React's JSX (`jsx: "react-jsx"`), Vite (`moduleResolution: "bundler"`), and modern tooling.

### Prerequisites

- A React project configured with TypeScript (Vite, Next.js, or Create React App with TypeScript template).
- A `tsconfig.json` with `strict: true` enabled.
- Basic familiarity with JavaScript types (string, number, boolean, arrays, objects, functions).
- An editor with TypeScript support (VS Code is the standard).

### Related Programming Areas

- **React component typing** — props, state, event handlers, children, refs.
- **React Hooks typing** — `useState`, `useRef`, `useReducer`, `useContext`, custom hooks.
- **API and data modeling** — discriminated unions for loading states, utility types for DTOs.
- **Build tooling** — Vite, SWC, esbuild, and `tsc` for type checking.
- **Linting and formatting** — Biome or ESLint with TypeScript rules.

### Core Concepts / Features

---

## 1. TypeScript

### Definitions

**Core Definition:** TypeScript is a statically typed programming language that builds on JavaScript by adding optional type syntax and a compile-time type checker.

**Technical Definition:** TypeScript is an open-source language developed by Microsoft that extends JavaScript with a **structural type system**. TypeScript code (`.ts`, `.tsx`) is compiled to plain JavaScript by `tsc` or a bundler (Vite, SWC, esbuild). The compiler performs type checking, catches errors, and emits JavaScript with types stripped. In React, TypeScript validates JSX props, state, hooks, and event handlers at compile time.

**Beginner-Friendly Explanation:** TypeScript is JavaScript with types. You write the same code, but you add labels that say "this is a string" or "this is a number." The TypeScript compiler checks those labels before your code runs, so you catch mistakes early.

### Purposes

- To catch type-related errors at compile time rather than at runtime.
- To provide rich editor support (autocomplete, inline documentation, refactoring).
- To serve as executable documentation for component props and function signatures.
- To enable safe refactoring across large codebases.
- To improve collaboration by making contracts between modules explicit.

### Syntax Rules and Structure

```typescript
// Basic type annotations
const name: string = 'Alice';
const age: number = 30;
const active: boolean = true;

// Function annotations
function greet(name: string): string {
  return `Hello, ${name}!`;
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `: string` | Type annotation for a variable or parameter. |
| `: number` | Numeric type. |
| `: boolean` | Boolean type. |
| `(param: type): returnType` | Function signature. |

**Constraints and Limitations:**

- TypeScript types are erased at runtime — they do not exist in the emitted JavaScript.
- TypeScript cannot catch all runtime errors — it validates types, not logic.
- `any` disables type checking and should be avoided; use `unknown` instead.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Basic Type Annotations

```typescript
// Explicit annotations
const username: string = 'alice';
const score: number = 100;
const isActive: boolean = true;

// Function with typed parameters and return
function formatUser(name: string, age: number): string {
  return `${name} is ${age} years old.`;
}

// Expected output: "alice is 30 years old."
console.log(formatUser(username, 30));
```

**Expected Output:** The function returns a formatted string. TypeScript validates that `name` is a string and `age` is a number at compile time.

### Real-World Cases

- **Every React project:** TypeScript validates props and state across the entire component tree.
- **API clients:** Typing request and response shapes catches contract mismatches early.
- **Large codebases:** Types make refactoring safe — the compiler finds every usage that needs updating.

### References

- TypeScript — Official Documentation — https://www.typescriptlang.org/docs/
- TypeScript Handbook — The Basics — https://www.typescriptlang.org/docs/handbook/2/basic-types.html

---

## 2. Types

### Definitions

**Core Definition:** A type is a description of what a value can be — its shape, its allowed values, or its behavior.

**Technical Definition:** In TypeScript, a **type** is a compile-time construct that describes a set of values. Types include primitives (`string`, `number`, `boolean`, `null`, `undefined`, `symbol`, `bigint`), object types (interfaces, type aliases), arrays (`T[]` or `Array<T>`), tuples (`[string, number]`), function types (`(a: T) => U`), literal types (`'north' | 'south'`), and special types (`any`, `unknown`, `never`, `void`). Types are structural — a value is assignable to a type if it has the required shape.

**Beginner-Friendly Explanation:** A type is a label that tells TypeScript what kind of value something is — like "this is a string" or "this is an object with a name and age." The compiler uses these labels to check your code.

### Purposes

- To describe the shape of data flowing through the application.
- To enable compile-time validation of values against expectations.
- To serve as documentation for functions, components, and APIs.
- To enable safe autocompletion and refactoring in the editor.

### Syntax Rules and Structure

**Primitive types:**

```typescript
const name: string = 'Alice';
const age: number = 30;
const active: boolean = true;
const nothing: null = null;
const missing: undefined = undefined;
```

**Array and tuple types:**

```typescript
const numbers: number[] = [1, 2, 3];
const pair: [string, number] = ['age', 30];
```

**Object types:**

```typescript
const user: { name: string; age: number } = { name: 'Alice', age: 30 };
```

**Function types:**

```typescript
type Comparator<T> = (a: T, b: T) => number;
```

**Constraints and Limitations:**

- `any` disables type checking — avoid it; use `unknown` for values that need narrowing.
- TypeScript does not perform runtime type checking — types are erased during compilation.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Primitive and Object Types

```typescript
// Primitives
const title: string = 'React TypeScript';
const version: number = 19;
const isStable: boolean = true;

// Object type
const config: { theme: 'light' | 'dark'; fontSize: number } = {
  theme: 'dark',
  fontSize: 14,
};

// Tuple
const coordinate: [number, number] = [40.7128, -74.0060];
```

**Expected Output:** TypeScript validates that each value matches its annotated type. Assigning `'blue'` to `theme` would produce a compile error because the type only allows `'light'` or `'dark'`.

### Real-World Cases

- **Component props:** Every prop has a type describing its expected value.
- **API responses:** Response shapes are typed to match the backend contract.
- **Configuration objects:** Settings and options are typed for safety.

### References

- TypeScript Handbook — Everyday Types — https://www.typescriptlang.org/docs/handbook/2/everyday-types.html
- TypeScript — Type System Reference — https://cdn.jsdelivr.net/npm/memory-journal-mcp@8.0.1/skills/typescript/references/type-system.md

---

## 3. Interfaces

### Definitions

**Core Definition:** An interface is a named declaration of the shape of an object — its property names, types, optionality, and methods.

**Technical Definition:** An `interface` in TypeScript declares the structure of an object type. It supports **optional properties** (`?`), **readonly properties** (`readonly`), **index signatures** (`[key: string]: T`), **extends** (inheritance from other interfaces), **implements** (class contracts), and **declaration merging** (multiple declarations with the same name are automatically merged). Interfaces are ideal for object shapes, class contracts, and public APIs because they produce clearer error messages and support extensibility.

**Beginner-Friendly Explanation:** An interface is a blueprint for an object. It says "any object of this type must have a name (string) and an age (number)." It is the most common way to type React component props.

### Purposes

- To define the shape of objects, especially component props and API models.
- To enable declaration merging for extensible type definitions.
- To serve as contracts for class implementations.
- To provide clear, readable type errors.

### Syntax Rules and Structure

```typescript
interface User {
  id: string;
  name: string;
  email?: string;       // Optional
  readonly createdAt: Date; // Cannot be reassigned
}
```

**Component Breakdown:**

| Feature | Syntax | Description |
|---------|--------|-------------|
| Property | `name: string` | Required property. |
| Optional property | `email?: string` | May be omitted. |
| Readonly property | `readonly createdAt: Date` | Cannot be modified after creation. |
| Extends | `interface Admin extends User { ... }` | Inherits all properties from `User`. |
| Implements | `class Manager implements User { ... }` | Class must conform to the interface. |
| Declaration merging | Two `interface Config` declarations merge. | Combines both sets of properties. |

**Constraints and Limitations:**

- Interfaces cannot express unions, tuples, mapped types, or conditional types — use type aliases for those.
- Interfaces are open — declaration merging can add properties unexpectedly in large codebases.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Interface for React Props

```typescript
interface ButtonProps {
  label: string;
  variant?: 'primary' | 'secondary';
  disabled?: boolean;
  onClick: () => void;
}

function Button({ label, variant = 'primary', disabled = false, onClick }: ButtonProps) {
  return (
    <button className={`btn-${variant}`} disabled={disabled} onClick={onClick}>
      {label}
    </button>
  );
}
```

**Expected Output:** TypeScript validates that `label` and `onClick` are always provided, while `variant` and `disabled` are optional. Passing an invalid `variant` value produces a compile error.

### Real-World Cases

- **Component props:** The standard way to type React component props.
- **API models:** User, Product, Order, and other domain entities.
- **Class contracts:** Services and repositories implement interfaces.

### References

- TypeScript Handbook — Interfaces — https://www.typescriptlang.org/docs/handbook/2/objects.html
- TypeScript Type System Reference — Interfaces vs Type Aliases — https://cdn.jsdelivr.net/npm/memory-journal-mcp@8.0.1/skills/typescript/references/type-system.md

---

## 4. Type Aliases

### Definitions

**Core Definition:** A type alias creates a new name for any type — primitives, unions, intersections, tuples, function types, mapped types, or conditional types.

**Technical Definition:** A `type` alias (`type Name = TypeExpression`) assigns a name to any type expression. Unlike interfaces, type aliases can represent **unions** (`'a' | 'b'`), **intersections** (`A & B`), **tuples** (`[string, number]`), **mapped types**, **conditional types**, and **primitive aliases** (`type ID = string | number`). They cannot be declaration-merged, but they are more expressive for complex type transformations.

**Beginner-Friendly Explanation:** A type alias is a nickname for a type. Instead of writing `string | number` everywhere, you write `type ID = string | number` and use `ID`. It is especially useful for unions and complex types.

### Purposes

- To give descriptive names to complex or repetitive types.
- To define unions and intersections that interfaces cannot express.
- To create aliases for primitives that carry semantic meaning (`type UserId = string`).
- To define function types and tuple types.

### Syntax Rules and Structure

```typescript
// Union alias
type Status = 'pending' | 'approved' | 'rejected';

// Primitive alias
type UserId = string;

// Intersection alias
type AdminUser = User & { role: 'admin'; permissions: string[] };

// Tuple alias
type Point = [x: number, y: number];

// Function alias
type Handler = (event: Event) => void;
```

**Constraints and Limitations:**

- Type aliases cannot be declaration-merged — two `type Config` declarations produce an error.
- Interfaces are generally preferred for object shapes because they produce better error messages and support `extends`/`implements`.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Union and Intersection Aliases

```typescript
interface User {
  id: string;
  name: string;
}

type AdminUser = User & { role: 'admin'; permissions: string[] };
type UserOrAdmin = User | AdminUser;
type Status = 'idle' | 'loading' | 'success' | 'error';

// Expected: TypeScript narrows based on discriminant
function getStatusLabel(status: Status): string {
  switch (status) {
    case 'idle': return 'Ready';
    case 'loading': return 'Loading...';
    case 'success': return 'Done';
    case 'error': return 'Failed';
  }
}
```

**Expected Output:** `Status` only allows the four literal values. Passing `'pending'` produces a compile error.

### Real-World Cases

- **API status types:** `'idle' | 'loading' | 'success' | 'error'`.
- **Branded types:** `type UserId = string & { __brand: 'UserId' }`.
- **Function contracts:** `type Comparator<T> = (a: T, b: T) => number`.

### References

- TypeScript Handbook — Type Aliases — https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#type-aliases
- TypeScript Type System Reference — When to Use Type Aliases — https://cdn.jsdelivr.net/npm/memory-journal-mcp@8.0.1/skills/typescript/references/type-system.md

---

## 5. Unions

### Definitions

**Core Definition:** A union type describes a value that can be one of several types — the value must match at least one member of the union.

**Technical Definition:** A union type (`A | B`) represents a value that can be either `A` or `B`. TypeScript requires narrowing before accessing properties specific to one member. Unions are the foundation of **discriminated unions** (tagged unions), where a shared literal property (the discriminator) enables automatic type narrowing in `switch` and `if` statements.

**Beginner-Friendly Explanation:** A union type says "this value can be one of these things." For example, a status can be `'idle'`, `'loading'`, `'success'`, or `'error'`. You use a check on a shared property to figure out which one it is.

### Purposes

- To model values that can take multiple forms (e.g., API responses, UI states).
- To enable discriminated unions for exhaustive state handling.
- To allow functions to accept multiple input types.
- To represent nullable values (`string | null`).

### Syntax Rules and Structure

```typescript
type Status = 'idle' | 'loading' | 'success' | 'error';
type StringOrNumber = string | number;
type NullableUser = User | null;
```

**Discriminated union:**

```typescript
type LoadingState =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: User[] }
  | { status: 'error'; error: Error; message: string };
```

**Constraints and Limitations:**

- You must narrow a union before accessing member-specific properties — TypeScript will not allow `state.data` on a union where only the `success` member has `data`.
- Unions can become complex — discriminated unions with a literal tag are the recommended pattern for state modeling.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Discriminated Union for Fetch State

```typescript
type FetchState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: Error };

function renderState<T>(state: FetchState<T>): string {
  switch (state.status) {
    case 'idle': return 'Ready to load';
    case 'loading': return 'Loading...';
    case 'success': return `Loaded ${state.data}`;
    case 'error': return `Error: ${state.error.message}`;
    default: {
      const _exhaustive: never = state; // Compile error if a case is missing
      return _exhaustive;
    }
  }
}
```

**Expected Output:** TypeScript narrows `state` to the correct member in each `case`. The `never` check ensures all cases are handled — adding a new status without handling it produces a compile error.

### Real-World Cases

- **UI state machines:** `'idle' | 'loading' | 'success' | 'error'`.
- **API responses:** Success and error shapes modeled as a discriminated union.
- **Redux actions:** Each action has a `type` discriminator with different payloads.

### References

- TypeScript Handbook — Narrowing / Discriminated Unions — https://www.typescriptlang.org/docs/handbook/2/narrowing.html#discriminated-unions
- Steve Kinney — Discriminated Unions — https://stevekinney.com/courses/react-typescript/typescript-discriminated-unions

---

## 6. Intersections

### Definitions

**Core Definition:** An intersection type combines multiple types into one — a value must satisfy **all** members of the intersection.

**Technical Definition:** An intersection type (`A & B`) creates a new type that has all properties of `A` and `B`. It is commonly used to **extend** object types (especially with type aliases) and to combine interfaces. If the intersected types have conflicting properties, the resulting type may be `never` for that property, which TypeScript catches at compile time.

**Beginner-Friendly Explanation:** An intersection means "this value must have everything from type A AND everything from type B." It is like combining two blueprints into one.

### Purposes

- To extend an existing type with additional properties (type-alias equivalent of `interface extends`).
- To combine multiple interfaces into a single type.
- To model objects that satisfy multiple contracts simultaneously.

### Syntax Rules and Structure

```typescript
interface User {
  id: string;
  name: string;
}

type AdminUser = User & { role: 'admin'; permissions: string[] };
// AdminUser requires id, name, role, and permissions
```

**Constraints and Limitations:**

- Conflicting property types produce `never` for that property — the intersection becomes uninhabitable for that key.
- For object extension, `interface extends` is often clearer than intersection aliases.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Extending with Intersection

```typescript
interface BaseProps {
  id: string;
  className?: string;
}

type ButtonProps = BaseProps & {
  label: string;
  onClick: () => void;
};

// ButtonProps requires: id, className (optional), label, onClick
```

**Expected Output:** `ButtonProps` includes all properties from both `BaseProps` and the inline object type.

### Real-World Cases

- **Extending component props:** Combining base props with component-specific props.
- **Mixins:** Combining multiple behavioral interfaces.
- **Domain modeling:** An `AdminUser` is a `User` **and** has admin properties.

### References

- TypeScript Handbook — Intersection Types — https://www.typescriptlang.org/docs/handbook/2/objects.html#intersection-types
- TypeScript Type System Reference — Union and Intersection Types — https://cdn.jsdelivr.net/npm/memory-journal-mcp@8.0.1/skills/typescript/references/type-system.md

---

## 7. Generics

### Definitions

**Core Definition:** Generics allow types to be **parameterized** — a function, interface, or component can work with a variety of types while preserving type safety.

**Technical Definition:** Generics use type parameters (`<T>`) to create reusable, type-safe abstractions. A generic function declares `<T>` and uses `T` as a placeholder for the actual type, which is supplied at the call site. Generics can be constrained (`<T extends object>`), defaulted (`<T = string>`), and used in functions, interfaces, type aliases, and classes. In React, generics enable reusable components and hooks that preserve the caller's type (`useState<T>`, `useFetch<T>`).

**Beginner-Friendly Explanation:** A generic is a type that is filled in later. Instead of writing a separate `Box` for strings and another for numbers, you write `Box<T>` and say "Box of string" or "Box of number" when you use it.

### Purposes

- To write reusable functions, components, and hooks that work with any type while preserving type safety.
- To avoid duplicating code for each type.
- To constrain type parameters to ensure they have the required shape.
- To enable type-safe collections, APIs, and utilities.

### Syntax Rules and Structure

```typescript
// Generic function
function identity<T>(value: T): T {
  return value;
}

// Generic interface
interface Box<T> {
  value: T;
}

// Constrained generic
function getLength<T extends { length: number }>(item: T): number {
  return item.length;
}

// Default type parameter
interface Paginated<T = unknown> {
  items: T[];
  total: number;
}
```

**Constraints and Limitations:**

- Generics add complexity — use them when type safety or reuse genuinely requires them.
- Constraint errors can be cryptic — provide clear constraints and defaults.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Generic Hook for Data Fetching

```typescript
type FetchState<T> =
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: Error };

function useFetch<T>(url: string): FetchState<T> {
  const [state, setState] = useState<FetchState<T>>({ status: 'loading' });

  useEffect(() => {
    fetch(url)
      .then((res) => res.json())
      .then((data: T) => setState({ status: 'success', data }))
      .catch((error: Error) => setState({ status: 'error', error }));
  }, [url]);

  return state;
}

// Usage — T is inferred as User[]
const state = useFetch<User[]>('/api/users');
```

**Expected Output:** `state.data` is typed as `User[]` when `status === 'success'`. The generic `T` preserves the caller's type through the hook.

### Real-World Cases

- **`useState<T>` and `useRef<T>`:** React's built-in hooks are generic.
- **API helpers:** `fetchJson<T>(url): Promise<T>`.
- **Reusable components:** `<List<T> items={items} renderItem={...} />`.
- **Utility functions:** `groupBy<T, K extends keyof T>(items: T[], key: K)`.

### References

- TypeScript Handbook — Generics — https://www.typescriptlang.org/docs/handbook/2/generics.html
- CoreUI — How to use TypeScript with React hooks — https://coreui.io/blog/how-to-use-typescript-with-react-hooks/

---

## 8. Type Inference

### Definitions

**Core Definition:** Type inference is TypeScript's ability to automatically determine the type of a value based on its usage and context, without explicit annotations.

**Technical Definition:** TypeScript infers types from initializers (`const x = 5` → `number`), function return values, contextual typing (the expected type from the surrounding context), and best common types. In React, `useState` infers the state type from the initial value, but **fails when the initial value is `null` or `undefined`** — explicit generics (`useState<User | null>(null)`) are required in those cases.

**Beginner-Friendly Explanation:** TypeScript is smart — it can often figure out the type without you telling it. `const name = 'Alice'` automatically gets the type `string`. But when you initialize with `null`, you have to tell TypeScript what the type will become later.

### Purposes

- To reduce boilerplate — you only annotate when inference is insufficient.
- To keep code readable and focused on intent.
- To automatically propagate types through functions and expressions.
- To catch errors without requiring explicit type declarations everywhere.

### Syntax Rules and Structure

```typescript
// Inference from initializer
const name = 'Alice';        // inferred: string
const age = 30;             // inferred: number
const active = true;        // inferred: boolean

// Inference from function return
function add(a: number, b: number) {
  return a + b;             // inferred return: number
}

// Contextual typing
const handler: (e: Event) => void = (e) => { /* e: Event */ };
```

**Constraints and Limitations:**

- `useState(null)` infers `null` — later assignments fail. Use `useState<User | null>(null)`.
- `useRef()` without an initial value is an error in React 19 — use `useRef<T>(null)`.
- Complex expressions may infer wider types than intended — explicit annotations or `as const` fix this.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Inference in `useState`

```typescript
// ✅ Inferred as number
const [count, setCount] = useState(0);

// ❌ Inferred as null — setUser('Alice') fails
// const [user, setUser] = useState(null);

// ✅ Explicit generic — User | null
const [user, setUser] = useState<User | null>(null);

// ✅ Inferred from initial object
const [form, setForm] = useState({ name: '', email: '' });
```

**Expected Output:** `count` is `number`, `user` is `User | null`, and `form` is `{ name: string; email: string }` — all inferred without explicit annotations where possible.

### Real-World Cases

- **`useState`:** Inference works for primitives and concrete objects; explicit generics are needed for null/undefined starts.
- **Event handlers:** Contextual typing infers the event type from the JSX attribute.
- **Generic functions:** TypeScript infers type arguments from the arguments passed.

### References

- TypeScript Handbook — Type Inference — https://www.typescriptlang.org/docs/handbook/type-inference.html
- Hook TypeScript Patterns — https://github.com/dot-skills/skills/blob/main/skills/.experimental/typescript-refactor/references/tsx-hook-typing.md

---

## 9. Type Guards

### Definitions

**Core Definition:** A type guard is an expression or function that performs a runtime check and tells TypeScript to narrow the type within a conditional block.

**Technical Definition:** Type guards include `typeof` checks (`typeof x === 'string'`), `instanceof` checks (`x instanceof Date`), `in` checks (`'name' in x`), equality checks, and **user-defined type predicates** (`function isUser(x: unknown): x is User`). TypeScript uses these checks to narrow the type of a value in the guarded block. Type predicates are the foundation of runtime validation functions that TypeScript can trust.

**Beginner-Friendly Explanation:** A type guard is a check you write to tell TypeScript "if this check passes, you can trust that the value is this type." It connects runtime logic to compile-time narrowing.

### Purposes

- To narrow `unknown` or union types to specific types safely.
- To validate API responses and external data at runtime.
- To enable type-safe handling of discriminated unions.
- To replace unsafe `as` assertions with checked operations.

### Syntax Rules and Structure

**Built-in guards:**

```typescript
if (typeof value === 'string') { /* value: string */ }
if (value instanceof Date) { /* value: Date */ }
if ('name' in obj) { /* obj has name */ }
```

**User-defined type predicate:**

```typescript
function isUser(value: unknown): value is User {
  return (
    typeof value === 'object' &&
    value !== null &&
    'name' in value &&
    typeof (value as User).name === 'string'
  );
}
```

**Constraints and Limitations:**

- Type predicates are assertions — if the runtime check is wrong, TypeScript will trust the narrow incorrectly.
- `typeof null` returns `'object'` — always check for `null` explicitly.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Type Predicate for API Response

```typescript
interface User {
  id: string;
  name: string;
}

function isUser(value: unknown): value is User {
  return (
    typeof value === 'object' &&
    value !== null &&
    'id' in value &&
    'name' in value &&
    typeof (value as User).id === 'string' &&
    typeof (value as User).name === 'string'
  );
}

function processResponse(data: unknown) {
  if (isUser(data)) {
    console.log(data.name); // data is User
  } else {
    console.log('Invalid user data');
  }
}
```

**Expected Output:** Inside the `if` block, `data` is narrowed to `User`, so `data.name` is accessible. Outside, it remains `unknown`.

### Real-World Cases

- **API validation:** Checking that response data matches the expected shape.
- **Discriminated unions:** `if (state.status === 'success')` narrows `state` to the success member.
- **Form validation:** Narrowing `unknown` form values before processing.

### References

- TypeScript Handbook — Narrowing — https://www.typescriptlang.org/docs/handbook/2/narrowing.html
- Type Guards and Narrowing Reference — https://github.com/dot-skills/skills/blob/main/skills/.experimental/typescript-refactor/references/tsx-hook-typing.md

---

## 10. Type Narrowing

### Definitions

**Core Definition:** Type narrowing is the process by which TypeScript refines a broad type (like a union or `unknown`) to a more specific type within a guarded code block.

**Technical Definition:** Narrowing occurs through **control flow analysis** — TypeScript tracks the conditions that must be true at each point in the code. Guards (`typeof`, `instanceof`, `in`, equality), discriminant checks on discriminated unions, truthiness checks, and user-defined type predicates all trigger narrowing. In `switch` statements on a discriminant property, each `case` narrows to the corresponding union member, and a `never` check at the end ensures exhaustiveness.

**Beginner-Friendly Explanation:** Type narrowing is TypeScript following your `if` and `switch` statements to figure out exactly which type you are working with at each line of code. If you check `status === 'success'`, TypeScript knows the data is there.

### Purposes

- To safely access properties that exist only on specific union members.
- To eliminate runtime errors from accessing properties on the wrong type.
- To enable exhaustive handling of all possible states.
- To replace unsafe casts with checked operations.

### Syntax Rules and Structure

**Narrowing techniques:**

| Technique | Example |
|-----------|---------|
| `typeof` | `if (typeof x === 'string')` |
| `instanceof` | `if (x instanceof Date)` |
| `in` | `if ('name' in obj)` |
| Equality | `if (status === 'success')` |
| Truthiness | `if (value)` |
| Discriminant | `switch (state.status)` |
| Type predicate | `if (isUser(data))` |

**Constraints and Limitations:**

- Narrowing is flow-sensitive — reassigning a narrowed variable may widen it again.
- Narrowing does not work across function boundaries unless the function is a type predicate.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Exhaustive Switch with `never`

```typescript
type Status = 'idle' | 'loading' | 'success' | 'error';

function getMessage(status: Status): string {
  switch (status) {
    case 'idle': return 'Ready';
    case 'loading': return 'Loading...';
    case 'success': return 'Done';
    case 'error': return 'Failed';
    default: {
      const _exhaustive: never = status;
      // If a new status is added and not handled,
      // _exhaustive will not be never → compile error
      return _exhaustive;
    }
  }
}
```

**Expected Output:** All four cases are handled. Adding `'cancelled'` to `Status` without adding a case produces a compile error at the `never` assignment.

### Real-World Cases

- **Reducer actions:** Narrowing on `action.type` to access the correct payload.
- **API responses:** Narrowing `unknown` response data with type predicates.
- **UI state:** Narrowing on `state.status` to render the correct view.

### References

- TypeScript Handbook — Narrowing — https://www.typescriptlang.org/docs/handbook/2/narrowing.html
- Steve Kinney — Discriminated Unions — https://stevekinney.com/courses/react-typescript/typescript-discriminated-unions

---

## 11. Utility Types

### Definitions

**Core Definition:** Utility types are built-in generic type transformations provided by TypeScript that create new types from existing ones.

**Technical Definition:** TypeScript includes a standard library of utility types that transform existing types: `Partial<T>` (all properties optional), `Required<T>` (all properties required), `Readonly<T>` (all properties readonly), `Pick<T, K>` (select properties), `Omit<T, K>` (exclude properties), `Record<K, V>` (object with keys `K` and values `V`), `Exclude<T, U>` (remove from union), `Extract<T, U>` (keep from union), `NonNullable<T>` (remove `null` and `undefined`), `ReturnType<F>` (function return type), and `Parameters<F>` (function parameter tuple).

**Beginner-Friendly Explanation:** Utility types are TypeScript's built-in "type transformers." `Partial<User>` makes all user fields optional. `Pick<User, 'id' | 'name'>` creates a type with only `id` and `name`. They save you from writing duplicate type definitions.

### Purposes

- To derive types from a single source of truth instead of duplicating definitions.
- To create flexible prop APIs (`Partial<Config>`, `Pick<User, 'id'>`).
- To model update payloads (`Partial<Omit<User, 'id'>>`).
- To extract function signatures and transform unions.

### Syntax Rules and Structure

**Essential utility types for React:**

| Utility | Purpose | Example |
|---------|---------|---------|
| `Partial<T>` | All properties optional | `Partial<User>` |
| `Pick<T, K>` | Select specific properties | `Pick<User, 'id' \| 'name'>` |
| `Omit<T, K>` | Exclude specific properties | `Omit<User, 'password'>` |
| `Record<K, V>` | Object with typed keys and values | `Record<string, User>` |
| `Required<T>` | All properties required | `Required<Config>` |
| `Readonly<T>` | All properties readonly | `Readonly<User>` |
| `NonNullable<T>` | Remove `null` and `undefined` | `NonNullable<string \| null>` |

**Constraints and Limitations:**

- Utility types create new types — they do not modify the original.
- Deep transformations require custom mapped types or libraries (e.g., `type-fest`).

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Deriving Update Types with `Partial` and `Omit`

```typescript
interface User {
  id: string;
  name: string;
  email: string;
  createdAt: Date;
}

// Update payload: all fields optional except id, createdAt excluded
type UserUpdate = Partial<Omit<User, 'id' | 'createdAt'>>;

function updateUser(id: string, updates: UserUpdate) {
  // updates: { name?: string; email?: string }
}

updateUser('u1', { name: 'Alice' });      // ✅
updateUser('u1', { email: 'a@b.com' });   // ✅
// updateUser('u1', { createdAt: new Date() }); // ❌ createdAt is not allowed
```

**Expected Output:** `UserUpdate` allows any subset of `name` and `email`, but rejects `id` and `createdAt`. The type stays in sync with `User` automatically.

#### Example 2: `Pick` for a Preview Component

```typescript
type UserPreview = Pick<User, 'id' | 'name'>;
// { id: string; name: string }
```

**Expected Output:** `UserPreview` contains only `id` and `name`, suitable for list-item components that do not need the full user object.

### Real-World Cases

- **API update payloads:** `Partial<Omit<User, 'id'>>` for PATCH requests.
- **Component prop subsets:** `Pick<User, 'id' | 'name'>` for lightweight list items.
- **Configuration merging:** `Partial<Config>` for optional overrides.
- **Record for maps:** `Record<string, User>` for indexed collections.

### References

- TypeScript Handbook — Utility Types — https://www.typescriptlang.org/docs/handbook/utility-types.html
- Steve Kinney — Utility Types That Unlock Clean React APIs — https://stevekinney.com/courses/react-typescript/utility-types-in-practice

---

## 12. TypeScript Configuration

### Definitions

**Core Definition:** TypeScript configuration is the `tsconfig.json` file that controls how the TypeScript compiler checks, transforms, and emits code.

**Technical Definition:** `tsconfig.json` specifies `compilerOptions` (target, module system, strictness flags, JSX mode, module resolution), `include`/`exclude` file patterns, and project references. For React projects, the critical settings are `jsx: "react-jsx"` (automatic JSX runtime), `moduleResolution: "bundler"` (Vite-optimized), and `strict: true` plus additional safety flags: `noUncheckedIndexedAccess` (array access returns `T | undefined`) and `exactOptionalPropertyTypes` (distinguishes absent from `undefined`).

**Beginner-Friendly Explanation:** `tsconfig.json` is the settings file for TypeScript. It tells the compiler how strict to be, what JavaScript version to target, and how to handle React's JSX syntax. Turning on `strict` makes TypeScript catch many more mistakes.

### Purposes

- To enable maximum type safety through strict compiler flags.
- To configure JSX handling for React (`react-jsx`).
- To optimize module resolution for the build tool (Vite/Next.js).
- To enforce consistent code quality across the project.

### Syntax Rules and Structure

**Recommended React `tsconfig.json`:**

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "jsx": "react-jsx",
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "noFallthroughCasesInSwitch": true,
    "noImplicitReturns": true,
    "verbatimModuleSyntax": true,
    "isolatedModules": true,
    "skipLibCheck": true
  },
  "include": ["src"]
}
```

**Key flags breakdown:**

| Flag | Purpose |
|------|---------|
| `strict: true` | Enables all core strict checks. |
| `noUncheckedIndexedAccess` | Array/object access returns `T \| undefined` — catches empty array bugs. |
| `exactOptionalPropertyTypes` | Distinguishes `{ x?: string }` from `{ x: string \| undefined }`. |
| `jsx: "react-jsx"` | Uses the automatic JSX runtime (no React import needed). |
| `moduleResolution: "bundler"` | Optimized for Vite/Next.js. |
| `noFallthroughCasesInSwitch` | Prevents accidental fallthrough in `switch` statements. |

**Constraints and Limitations:**

- `noUncheckedIndexedAccess` can be noisy in code that already validates array bounds — but it surfaces real bugs.
- `exactOptionalPropertyTypes` requires careful handling of optional props in React components.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Strict Configuration in Practice

```typescript
// With noUncheckedIndexedAccess: true
const users: User[] = [];
const first = users[0]; // type: User | undefined
if (first) {
  console.log(first.name); // ✅ safe — narrowed
}

// Without the flag: users[0] is User, and first.name crashes at runtime
```

**Expected Output:** TypeScript forces a null check on `users[0]`, preventing a runtime crash on empty arrays.

### Real-World Cases

- **New React projects:** Vite and Next.js templates ship with strict TypeScript configurations.
- **Monorepos:** Shared base `tsconfig.json` with project references.
- **CI type checking:** `tsc --noEmit` verifies types without emitting files.

### References

- TypeScript — tsconfig.json Reference — https://www.typescriptlang.org/tsconfig/
- Tooling Setup for React 19 — https://raw.githubusercontent.com/MadAppGang/claude-code/main/plugins/frontend/skills/tooling-setup/SKILL.md

---

## 13. TypeScript Best Practices

### Definitions

**Core Definition:** TypeScript best practices are the guidelines that maximize type safety, readability, and maintainability in a TypeScript codebase.

**Technical Definition:** Best practices include: preferring `interface` for object shapes and `type` for unions/intersections/tuples; using `unknown` over `any` and narrowing with type guards; modeling state with discriminated unions and exhaustive `never` checks; enabling `strict: true` with `noUncheckedIndexedAccess` and `exactOptionalPropertyTypes`; preferring type inference over redundant annotations; avoiding `React.FC` in favor of explicit props types; and typing event handlers with React's generic event types (`React.ChangeEvent<HTMLInputElement>`).

**Beginner-Friendly Explanation:** Best practices are the habits that keep TypeScript code safe and clean — use the right tool for each job, never use `any`, turn on strict mode, and let TypeScript infer types when it can.

### Purposes

- To maximize type safety and minimize runtime errors.
- To keep code readable and maintainable.
- To leverage TypeScript's strengths (inference, narrowing, discriminated unions).
- To avoid common anti-patterns (`any`, `React.FC`, unsafe casts).

### Best Practices Checklist

| Practice | Description |
|----------|-------------|
| **`interface` for objects, `type` for unions** | Interfaces extend and merge; type aliases handle unions, tuples, and mapped types. |
| **`unknown` over `any`** | `unknown` forces narrowing; `any` disables type checking entirely. |
| **Discriminated unions for state** | Model loading/success/error states with a literal discriminant and exhaustive `never` checks. |
| **Strict tsconfig** | Enable `strict`, `noUncheckedIndexedAccess`, and `exactOptionalPropertyTypes`. |
| **Prefer inference** | Annotate only when inference is insufficient (`useState(null)` needs a generic). |
| **Avoid `React.FC`** | Use explicit props interfaces — `React.FC` implicitly adds `children` and has other quirks. |
| **Type event handlers correctly** | Use `React.ChangeEvent<HTMLInputElement>`, `React.MouseEvent<HTMLButtonElement>`, etc.. |
| **Use `PropsWithChildren`** | For components that accept children, use `PropsWithChildren<T>` or `children: React.ReactNode`. |

**Constraints and Limitations:**

- Best practices are contextual — there are legitimate uses for `any` (migration, third-party types), but they should be rare and documented.
- Strict flags can be noisy initially — the long-term benefit outweighs the short-term friction.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Best-Practice Component

```typescript
interface ButtonProps {
  label: string;
  variant?: 'primary' | 'secondary';
  onClick: (event: React.MouseEvent<HTMLButtonElement>) => void;
  children?: React.ReactNode;
}

function Button({ label, variant = 'primary', onClick, children }: ButtonProps) {
  return (
    <button className={`btn-${variant}`} onClick={onClick}>
      {children ?? label}
    </button>
  );
}
```

**Why this follows best practices:**

- `interface` for object props (extensible, clear errors).
- `variant` is an optional literal union.
- `onClick` is typed with React's generic `MouseEvent`.
- `children` is typed as `React.ReactNode`.
- No `React.FC` — explicit props interface.

#### Example 2: Discriminated Union for API State

```typescript
type ApiState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: Error };

function render<T>(state: ApiState<T>) {
  switch (state.status) {
    case 'idle': return 'Ready';
    case 'loading': return 'Loading...';
    case 'success': return `Data: ${state.data}`;
    case 'error': return `Error: ${state.error.message}`;
    default: {
      const _exhaustive: never = state;
      return _exhaustive;
    }
  }
}
```

**Why this follows best practices:** The discriminated union makes invalid states impossible to represent, and the `never` check guarantees exhaustive handling.

### Real-World Cases

- **Production React applications:** Strict TypeScript catches bugs before deployment.
- **Design systems:** Typed props ensure components are used correctly across teams.
- **API layers:** Discriminated unions and type predicates validate external data safely.

### References

- TypeScript Patterns Skill — https://github.com/marvinrichter/clarc/blob/a8355b036a977300dfce1758af1c0d7e3371a26b/skills/typescript-patterns/SKILL.md
- React + TypeScript Cheatsheet — https://github.com/typescript-cheatsheets/react
- CoreUI — How to type events in React with TypeScript — https://coreui.io/blog/how-to-type-events-in-react-with-typescript/

---

## References

- TypeScript — Official Documentation — https://www.typescriptlang.org/docs/
- TypeScript Handbook — Everyday Types — https://www.typescriptlang.org/docs/handbook/2/everyday-types.html
- TypeScript Handbook — Narrowing — https://www.typescriptlang.org/docs/handbook/2/narrowing.html
- TypeScript Handbook — Generics — https://www.typescriptlang.org/docs/handbook/2/generics.html
- TypeScript Handbook — Utility Types — https://www.typescriptlang.org/docs/handbook/utility-types.html
- TypeScript Handbook — Type Inference — https://www.typescriptlang.org/docs/handbook/type-inference.html
- TypeScript — tsconfig.json Reference — https://www.typescriptlang.org/tsconfig/
- TypeScript Type System Reference — https://cdn.jsdelivr.net/npm/memory-journal-mcp@8.0.1/skills/typescript/references/type-system.md
- TypeScript Patterns Skill (clarc) — https://github.com/marvinrichter/clarc/blob/a8355b036a977300dfce1758af1c0d7e3371a26b/skills/typescript-patterns/SKILL.md
- React + TypeScript Cheatsheet — https://github.com/typescript-cheatsheets/react
- Steve Kinney — Discriminated Unions — https://stevekinney.com/courses/react-typescript/typescript-discriminated-unions
- Steve Kinney — Utility Types That Unlock Clean React APIs — https://stevekinney.com/courses/react-typescript/utility-types-in-practice
- CoreUI — How to type events in React with TypeScript — https://coreui.io/blog/how-to-type-events-in-react-with-typescript/
- CoreUI — How to use TypeScript with React hooks — https://coreui.io/blog/how-to-use-typescript-with-react-hooks/
- Tooling Setup for React 19 (strict tsconfig) — https://raw.githubusercontent.com/MadAppGang/claude-code/main/plugins/frontend/skills/tooling-setup/SKILL.md
- Hook TypeScript Patterns — https://github.com/dot-skills/skills/blob/main/skills/.experimental/typescript-refactor/references/tsx-hook-typing.md
- Type Guards and Narrowing Reference — https://github.com/dot-skills/skills/blob/main/skills/.experimental/typescript-refactor/references/tsx-hook-typing.md