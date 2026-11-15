# React TS Typing Components — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Typing React components with TypeScript means annotating a component's **props, state, event handlers, refs, and return values** so the compiler can verify that the component is used correctly at every call site.

**Technical Definition:** React component typing applies TypeScript's structural type system to React's component model. Every function component is a function `(props: P) => ReactNode`, and TypeScript checks that the JSX passed to it matches `P`. Typing extends across the full component surface: **props interfaces**, **children** (`ReactNode` vs. `ReactElement`), **default props** (destructuring defaults vs. `defaultProps`), **event handlers** (`React.ChangeEvent<HTMLInputElement>` etc.), **refs** (`useRef<T>` and `forwardRef`), **context** (null-checked `createContext`), **hooks** (`useState<T>`, `useReducer` with discriminated unions), **actions and selectors** (typed Redux-style flows), **generics** (`<T>` parameterized components), and **polymorphic components** (`as` prop with `ElementType` and `ComponentPropsWithoutRef<T>`). The consensus in 2026 is to **avoid `React.FC`** and use plain function declarations with explicit props types.

**Beginner-Friendly Explanation:** When you build a React component in TypeScript, you tell the compiler exactly what "inputs" (props) it accepts and what it does with them. This cheat sheet covers every part of a component you need to type — from simple props to complex generic and polymorphic components — so you catch mistakes before users do.

### Key Characteristics

- **Props-first:** The props interface is the contract; everything else (children, events, refs) flows from it.
- **Avoid `React.FC`:** Plain function declarations with explicit props interfaces are the modern standard.
- **Discriminated unions for state:** Model loading/error/success states and reducer actions as tagged unions for exhaustive checking.
- **Generics preserve caller types:** Generic components keep the type `T` through the component tree.
- **Polymorphism with `as`:** `ElementType` + `ComponentPropsWithoutRef<T>` enables type-safe rendering of different HTML elements.

### Prerequisites

- A React + TypeScript project with `strict: true` in `tsconfig.json`.
- Familiarity with TypeScript interfaces, unions, generics, and utility types.
- Understanding of React component props, children, events, and hooks.

### Related Programming Areas

- **React TypeScript Fundamentals** — the language features used here.
- **React Hooks** — `useState`, `useReducer`, `useRef`, `useContext`.
- **State management** — Redux, Zustand, and custom reducer patterns.
- **Component design systems** — polymorphic and generic components.

### Core Concepts / Features

---

## 1. Typing Function Components

### Definitions

**Core Definition:** A typed function component is a plain function that takes a typed props object and returns JSX, with the props type declared explicitly.

**Technical Definition:** The recommended modern pattern is a **plain function declaration** with an explicit props interface — not `React.FC`. The function signature `function Component(props: Props): ReactNode` lets TypeScript infer the return type and validate every prop at the call site. `React.FC` is discouraged because it adds implicit `children` (pre-React 18), does not support generic components, and has awkward `defaultProps` typing.

**Beginner-Friendly Explanation:** Write a normal function, give it a typed props object, and return JSX. Do not wrap it in `React.FC` — the plain function is simpler and more powerful.

### Purposes

- To define a component whose props are validated at every call site.
- To enable autocomplete and refactoring for component consumers.
- To avoid the pitfalls of `React.FC` (implicit children, no generics).
- To keep the component's contract explicit and readable.

### Syntax Rules and Structure

```tsx
interface GreetingProps {
  name: string;
  age?: number;
}

function Greeting({ name, age }: GreetingProps) {
  return <h1>Hello, {name}{age ? ` (${age})` : ''}!</h1>;
}
```

**Constraints and Limitations:**

- Do not use `React.FC` — it is the wrong default in 2026.
- The return type is usually inferred; explicit `: ReactNode` is optional but can be useful for clarity.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Plain Function Component (Recommended)

```tsx
interface ButtonProps {
  label: string;
  variant?: 'primary' | 'secondary';
  onClick: () => void;
}

function Button({ label, variant = 'primary', onClick }: ButtonProps) {
  return (
    <button className={`btn-${variant}`} onClick={onClick}>
      {label}
    </button>
  );
}
```

**Expected Output:** TypeScript validates `label` and `onClick` as required, `variant` as an optional literal union, and infers the return type as `JSX.Element`.

### Real-World Cases

- **Design systems:** Every component uses a plain function with an explicit props interface.
- **Application code:** shadcn/ui, Next.js, and Vercel templates all use plain function components.

### References

- React KB — Don't Use `React.FC` — https://github.com/Nguyen-Mau-Anh/react-kb/blob/main/05.routing-and-styling/03-typescript-with-react.md
- Backstage ADR — Avoid `React.FC` — https://github.com/backstage/backstage/blob/master/docs/architecture-decisions/adr006-avoid-react-fc.md

---

## 2. Typing Props

### Definitions

**Core Definition:** Typing props means declaring an interface (or type alias) that describes every prop a component accepts, including which are required and which are optional.

**Technical Definition:** Props are typed with an **interface** (preferred for extendability and better error messages) or a **type alias**. Required props are declared without `?`; optional props use `?`. Literal unions (`'primary' | 'secondary'`) constrain allowed values. `ComponentPropsWithoutRef<'button'>` inherits native HTML attributes when wrapping a native element.

**Beginner-Friendly Explanation:** A props interface is the list of everything a component can receive. Required props must always be passed; optional props use `?`.

### Purposes

- To define the component's public API clearly.
- To catch missing or mistyped props at compile time.
- To enable editor autocomplete for consumers.
- To constrain props to valid values (literal unions).

### Syntax Rules and Structure

```tsx
interface ButtonProps {
  label: string;                    // Required
  variant?: 'primary' | 'secondary'; // Optional literal union
  onClick: () => void;              // Required callback
}
```

**Constraints and Limitations:**

- Prefer `interface` for props — type aliases cannot `extend` for composition.
- Inline prop types are not reusable and should be avoided.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Interface with Optional and Literal Props

```tsx
interface BadgeProps {
  label: string;
  variant?: 'success' | 'warning' | 'error';
  size?: 'sm' | 'md' | 'lg';
}

function Badge({ label, variant = 'success', size = 'md' }: BadgeProps) {
  return <span className={`badge badge-${variant} badge-${size}`}>{label}</span>;
}
```

**Expected Output:** `label` is required; `variant` and `size` are optional with defaults. Passing `variant="info"` produces a compile error.

### Real-World Cases

- **Component libraries:** Every component has a named props interface.
- **Form components:** Props define value, onChange, error, and label contracts.

### References

- TypeScript React Patterns — Component Props Interface — https://raw.githubusercontent.com/voku/agent-skills/refs/heads/copilot/update-php-best-practices/skills/typescript-react-patterns/AGENTS.md

---

## 3. Typing Children

### Definitions

**Core Definition:** Typing children means declaring the `children` prop with the correct React type — `ReactNode` for anything renderable, or `ReactElement` when only a single JSX element is allowed.

**Technical Definition:** `ReactNode` represents anything React can render: `string`, `number`, `JSX.Element`, fragments, arrays, `null`, `undefined`, and `boolean`. `ReactElement` is a single JSX element (e.g., `<div>` or `<MyComponent />`) and is used when the component needs to `cloneElement` or inspect `.type`. `JSX.Element` is the return type of JSX expressions.

**Beginner-Friendly Explanation:** Use `ReactNode` for children when they can be anything — text, numbers, elements, or nothing. Use `ReactElement` when you need exactly one JSX element.

### Purposes

- To correctly type wrapper components that accept arbitrary children.
- To constrain children when only a single element is valid.
- To avoid the implicit-children surprise of `React.FC`.

### Syntax Rules and Structure

```tsx
import type { ReactNode, ReactElement } from 'react';

interface CardProps {
  title: string;
  children: ReactNode; // Anything renderable
}
```

**Constraints and Limitations:**

- `ReactNode` includes `null`, `undefined`, and `boolean` — good for optional content.
- `ReactElement` does not include strings or numbers — use it only for single-element children.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Wrapper Component with `ReactNode` Children

```tsx
import type { ReactNode } from 'react';

interface CardProps {
  title: string;
  children: ReactNode;
}

function Card({ title, children }: CardProps) {
  return (
    <div className="card">
      <h2>{title}</h2>
      <div className="card-body">{children}</div>
    </div>
  );
}
```

**Expected Output:** `<Card title="Hi">Hello <strong>world</strong></Card>` and `<Card title="Hi">{null}</Card>` both compile.

### Real-World Cases

- **Layout components:** Cards, modals, and panels that wrap arbitrary content.
- **Providers:** Context providers that accept children.

### References

- React KB — Typing `children` — https://github.com/Nguyen-Mau-Anh/react-kb/blob/main/05.routing-and-styling/03-typescript-with-react.md

---

## 4. Typing Default Props

### Definitions

**Core Definition:** Typing default props means declaring props as optional and providing their default values through destructuring in the function parameter list.

**Technical Definition:** The recommended approach is **destructuring defaults** directly in the parameter list — TypeScript infers the prop as optional automatically. Alternatively, type the props with the defaulted keys as required and let `LibraryManagedAttributes` (applied by JSX) make them optional at the call site. `React.FC` and `defaultProps` interact poorly and should be avoided.

**Beginner-Friendly Explanation:** Put default values in the destructuring: `function Button({ variant = 'primary' })`. TypeScript knows `variant` is optional and defaults to `'primary'`.

### Purposes

- To provide sensible defaults without requiring callers to pass every prop.
- To keep the props type accurate — optional where a default exists.
- To avoid the known issues with `defaultProps` and function components.

### Syntax Rules and Structure

```tsx
interface ButtonProps {
  label: string;
  variant?: 'primary' | 'secondary'; // Optional because it has a default
  disabled?: boolean;
}

function Button({ label, variant = 'primary', disabled = false }: ButtonProps) {
  // ...
}
```

**Constraints and Limitations:**

- Do not use `defaultProps` on function components — it is deprecated for function components and has known typing issues.
- Defaults must be compatible with the declared type.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Destructuring Defaults

```tsx
interface AlertProps {
  message: string;
  variant?: 'info' | 'warning' | 'error';
  dismissible?: boolean;
}

function Alert({ message, variant = 'info', dismissible = true }: AlertProps) {
  return <div className={`alert-${variant}`}>{message}</div>;
}
```

**Expected Output:** `<Alert message="Saved" />` compiles and uses `variant = 'info'` and `dismissible = true`.

### Real-World Cases

- **UI components:** Buttons, badges, and alerts with default variants.
- **Form fields:** Inputs with default `type="text"` and `size="md"`.

### References

- Typing Default Props — https://github.com/dot-skills/skills/blob/main/skills/.experimental/typescript-refactor/references/default-props.md
- CoreUI — How to set default props in React — https://coreui.io/blog/how-to-set-default-props-in-react/

---

## 5. Typing Event Handlers

### Definitions

**Core Definition:** Typing event handlers means annotating React event callbacks with the correct `React.SyntheticEvent` generic that specifies the target HTML element.

**Technical Definition:** React event types are generic over the target element: `React.ChangeEvent<HTMLInputElement>`, `React.MouseEvent<HTMLButtonElement>`, `React.FormEvent<HTMLFormElement>`, `React.KeyboardEvent<HTMLInputElement>`. Using the specific event type gives accurate `event.target.value` typing and prevents accessing properties that do not exist on the element.

**Beginner-Friendly Explanation:** When you type an input change handler, use `React.ChangeEvent<HTMLInputElement>` so TypeScript knows `event.target.value` is a string. Do not use generic `Event` or `any`.

### Purposes

- To get accurate typing for `event.target`, `event.currentTarget`, and element-specific properties.
- To catch incorrect event usage at compile time.
- To make handlers reusable and self-documenting.

### Syntax Rules and Structure

```tsx
function handleChange(event: React.ChangeEvent<HTMLInputElement>) {
  console.log(event.target.value); // string
}

function handleClick(event: React.MouseEvent<HTMLButtonElement>) {
  console.log(event.currentTarget.disabled); // boolean
}
```

**Constraints and Limitations:**

- Use `React.ChangeEvent` for inputs, `React.FormEvent` for forms, `React.MouseEvent` for clicks.
- `event.target` is the element that fired the event; `event.currentTarget` is the element the handler is attached to.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Typed Input and Button Handlers

```tsx
function SearchForm() {
  const [query, setQuery] = useState('');

  function handleChange(event: React.ChangeEvent<HTMLInputElement>) {
    setQuery(event.target.value);
  }

  function handleSubmit(event: React.FormEvent<HTMLFormElement>) {
    event.preventDefault();
    console.log(query);
  }

  return (
    <form onSubmit={handleSubmit}>
      <input value={query} onChange={handleChange} />
      <button type="submit">Search</button>
    </form>
  );
}
```

**Expected Output:** `event.target.value` is typed as `string` in `handleChange`; `event.preventDefault()` is available in `handleSubmit`.

### Real-World Cases

- **Forms:** Input change and form submit handlers.
- **Buttons:** Click handlers with `currentTarget` typing.
- **Keyboard shortcuts:** `React.KeyboardEvent` for Enter/Escape handling.

### References

- TypeScript React Patterns — Event Handling — https://raw.githubusercontent.com/voku/agent-skills/refs/heads/copilot/update-php-best-practices/skills/typescript-react-patterns/AGENTS.md

---

## 6. Typing Refs

### Definitions

**Core Definition:** Typing refs means declaring `useRef` with the correct element type for DOM refs, or the correct value type for mutable instance refs.

**Technical Definition:** `useRef<HTMLInputElement>(null)` creates a ref to an input element. `useRef<number>(0)` creates a mutable ref holding a number. For custom components, `forwardRef<HTMLButtonElement, ButtonProps>` forwards a ref to the underlying DOM node. `useImperativeHandle` exposes imperative methods to parent refs.

**Beginner-Friendly Explanation:** A DOM ref holds an HTML element — type it with the element type, like `HTMLInputElement`. A value ref holds any mutable value — type it with that value's type.

### Purposes

- To access DOM methods (`focus()`, `select()`, `scrollIntoView()`) with full type safety.
- To forward refs through custom components.
- To hold mutable values across renders without causing re-renders.

### Syntax Rules and Structure

```tsx
// DOM ref
const inputRef = useRef<HTMLInputElement>(null);
// inputRef.current?.focus();

// Value ref
const countRef = useRef<number>(0);
countRef.current += 1;

// Forwarded ref
interface ButtonProps { label: string; }
const Button = forwardRef<HTMLButtonElement, ButtonProps>(({ label }, ref) => (
  <button ref={ref}>{label}</button>
));
```

**Constraints and Limitations:**

- In React 19, `useRef()` requires an initial value — use `useRef<T>(null)`.
- Do not read or write `ref.current` during render — only in event handlers and effects.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Forwarding a Ref to a DOM Element

```tsx
import { forwardRef, useRef } from 'react';

interface InputProps {
  label: string;
}

const TextInput = forwardRef<HTMLInputElement, InputProps>(({ label }, ref) => (
  <label>
    {label}
    <input ref={ref} type="text" />
  </label>
));

function Form() {
  const inputRef = useRef<HTMLInputElement>(null);

  return (
    <>
      <TextInput label="Name" ref={inputRef} />
      <button onClick={() => inputRef.current?.focus()}>Focus</button>
    </>
  );
}
```

**Expected Output:** `inputRef.current` is typed as `HTMLInputElement | null`; the Focus button calls `.focus()` with full type safety.

### Real-World Cases

- **Focus management:** Focusing inputs and modals on open.
- **Form libraries:** React Hook Form uses refs extensively.
- **Third-party integrations:** Passing refs to chart or map libraries.

### References

- TypeScript React Patterns — Ref Typing — https://raw.githubusercontent.com/voku/agent-skills/refs/heads/copilot/update-php-best-practices/skills/typescript-react-patterns/AGENTS.md

---

## 7. Typing Context

### Definitions

**Core Definition:** Typing context means declaring the context value type and creating a custom hook that throws when used outside a provider, eliminating `null` checks.

**Technical Definition:** `createContext<ValueType | null>(null)` allows a `null` default so that using the context outside a provider is detectable. A custom `useMyContext` hook reads the context and throws if it is `null`, returning a non-null `ValueType`. This pattern is used throughout React libraries.

**Beginner-Friendly Explanation:** Create a context with a `null` default, then write a custom hook that throws a helpful error if the value is `null`. Consumers get a non-null value without checking every time.

### Purposes

- To provide type-safe shared state across the component tree.
- To catch missing providers at runtime with a clear error.
- To avoid repetitive `null` checks in every consumer.

### Syntax Rules and Structure

```tsx
interface ThemeContextValue {
  theme: 'light' | 'dark';
  toggle: () => void;
}

const ThemeContext = createContext<ThemeContextValue | null>(null);

function useTheme(): ThemeContextValue {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
}
```

**Constraints and Limitations:**

- Always provide a `null` default so missing providers are detectable.
- The custom hook must be used within the provider subtree.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Typed Context with Custom Hook

```tsx
interface AuthContextValue {
  user: User | null;
  login: (email: string) => Promise<void>;
}

const AuthContext = createContext<AuthContextValue | null>(null);

export function useAuth(): AuthContextValue {
  const ctx = useContext(AuthContext);
  if (!ctx) throw new Error('useAuth must be used within AuthProvider');
  return ctx;
}

export function AuthProvider({ children }: { children: ReactNode }) {
  const [user, setUser] = useState<User | null>(null);

  const login = useCallback(async (email: string) => {
    const user = await api.login(email);
    setUser(user);
  }, []);

  return (
    <AuthContext.Provider value={{ user, login }}>
      {children}
    </AuthContext.Provider>
  );
}
```

**Expected Output:** `useAuth()` returns `AuthContextValue` with non-null `login` and `user` typed as `User | null`.

### Real-World Cases

- **Authentication:** Providing user and login/logout functions.
- **Theming:** Providing theme and toggle functions.
- **Feature flags:** Providing flag values and setters.

### References

- TypeScript React Patterns — Context & State — https://raw.githubusercontent.com/voku/agent-skills/refs/heads/copilot/update-php-best-practices/skills/typescript-react-patterns/AGENTS.md

---

## 8. Typing Hooks

### Definitions

**Core Definition:** Typing hooks means providing explicit generic arguments to React's built-in hooks (`useState<T>`, `useRef<T>`, `useReducer<S, A>`) and typing custom hooks' parameters and return values.

**Technical Definition:** `useState<T>` requires an explicit generic when the initial value is `null` or `undefined`. `useRef<T>(null)` types a DOM ref. `useReducer<Reducer<State, Action>>` types the reducer. Custom hooks declare their parameter types and return type, often as a tuple `as const` for stable ordering.

**Beginner-Friendly Explanation:** Give `useState` a type when it starts as `null`; give `useRef` the element type; give `useReducer` the state and action types.

### Purposes

- To make state, refs, and reducers type-safe.
- To ensure custom hooks have clear, documented contracts.
- To avoid `any` in hook-heavy code.

### Syntax Rules and Structure

```tsx
// useState with null initial
const [user, setUser] = useState<User | null>(null);

// useRef for DOM
const inputRef = useRef<HTMLInputElement>(null);

// Custom hook with typed return
function useToggle(initial = false): [boolean, () => void] {
  const [value, setValue] = useState(initial);
  const toggle = useCallback(() => setValue((v) => !v), []);
  return [value, toggle];
}
```

**Constraints and Limitations:**

- `useState(null)` infers `null` — always provide the generic.
- Custom hook return types should be explicit for public hooks.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Typed `useState` and Custom Hook

```tsx
type Status = 'idle' | 'loading' | 'success' | 'error';

function useAsyncStatus() {
  const [status, setStatus] = useState<Status>('idle');
  return { status, setStatus };
}

function Component() {
  const { status, setStatus } = useAsyncStatus();
  return <button onClick={() => setStatus('loading')}>{status}</button>;
}
```

**Expected Output:** `status` is narrowed to the `Status` union; `setStatus('invalid')` is a compile error.

### Real-World Cases

- **Data fetching hooks:** `useFetch<T>` returning `{ data: T | null; loading: boolean; error: Error | null }`.
- **Form hooks:** `useForm<T>` with typed values and errors.
- **Storage hooks:** `useLocalStorage<T>` with typed values.

### References

- TypeScript React Patterns — Hook Typing — https://raw.githubusercontent.com/voku/agent-skills/refs/heads/copilot/update-php-best-practices/skills/typescript-react-patterns/AGENTS.md

---

## 9. Typing Reducers

### Definitions

**Core Definition:** Typing a reducer means declaring the `State` and `Action` types for `useReducer` so that every action is validated and every state transition is type-safe.

**Technical Definition:** `useReducer` is typed as `useReducer<Reducer<State, Action>>(reducer, initialState)` or inferred from a typed reducer function. The reducer signature is `(state: State, action: Action) => State`. Discriminated union actions enable exhaustive `switch` statements with a `never` check in the `default` branch.

**Beginner-Friendly Explanation:** Give your reducer a state type and an action type. The action type is a union of every possible action, each with a unique `type` string. TypeScript then checks that you handle every action.

### Purposes

- To make reducer state and actions fully type-safe.
- To enforce exhaustive action handling at compile time.
- To prevent invalid state transitions.

### Syntax Rules and Structure

```tsx
interface State { count: number; }

type Action =
  | { type: 'increment' }
  | { type: 'decrement' }
  | { type: 'reset' };

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'increment': return { count: state.count + 1 };
    case 'decrement': return { count: state.count - 1 };
    case 'reset': return { count: 0 };
    default: {
      const _exhaustive: never = action;
      return state;
    }
  }
}
```

**Constraints and Limitations:**

- Always include the `never` exhaustiveness check — it catches missing cases.
- Do not use `payload?: any` — give each action its own payload type.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Typed Reducer with Exhaustiveness

```tsx
interface TodoState { todos: Todo[]; filter: 'all' | 'active' | 'done'; }

type TodoAction =
  | { type: 'ADD'; payload: { text: string } }
  | { type: 'TOGGLE'; payload: { id: string } }
  | { type: 'SET_FILTER'; payload: 'all' | 'active' | 'done' };

function todoReducer(state: TodoState, action: TodoAction): TodoState {
  switch (action.type) {
    case 'ADD':
      return { ...state, todos: [...state.todos, { id: crypto.randomUUID(), text: action.payload.text, done: false }] };
    case 'TOGGLE':
      return { ...state, todos: state.todos.map((t) => t.id === action.payload.id ? { ...t, done: !t.done } : t) };
    case 'SET_FILTER':
      return { ...state, filter: action.payload };
    default: {
      const _exhaustive: never = action;
      return state;
    }
  }
}
```

**Expected Output:** `action.payload.text` is typed as `string` inside the `'ADD'` case; adding a new action without a case produces a compile error at the `never` check.

### Real-World Cases

- **Todo apps:** Add, toggle, delete, and filter actions.
- **Shopping carts:** Add item, remove item, update quantity.
- **Form wizards:** Next step, previous step, reset.

### References

- useReducer Typing — https://raw.githubusercontent.com/AsyrafHussin/agent-skills/1aa0ff717c10309226c9e678f00873976450fd76/skills/typescript-react-patterns/rules/hook-use-reducer.md
- Educative — `useReducer` Hook with Discriminated Unions — https://www.educative.io/

---

## 10. Typing Actions

### Definitions

**Core Definition:** Typing actions means defining a discriminated union where each action has a literal `type` discriminant and its own payload shape.

**Technical Definition:** An action type is a union of object types, each with a `type` field whose value is a unique string literal. This allows TypeScript to narrow the action to the correct member inside a `switch` on `action.type`, giving type-safe access to the payload.

**Beginner-Friendly Explanation:** Each action gets its own shape — `{ type: 'ADD', payload: { text: string } }` or `{ type: 'DELETE', payload: { id: string } }`. The `type` field is a literal string that tells TypeScript which action it is.

### Purposes

- To give every action a precise payload type.
- To enable exhaustive switch statements.
- To prevent dispatching invalid actions.

### Syntax Rules and Structure

```tsx
type Action =
  | { type: 'ADD_TODO'; payload: { text: string } }
  | { type: 'TOGGLE_TODO'; payload: { id: string } }
  | { type: 'DELETE_TODO'; payload: { id: string } };
```

**Constraints and Limitations:**

- Every action must have a unique literal `type`.
- Do not use a generic `{ type: string; payload?: any }` — it defeats the purpose.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Discriminated Action Union

```tsx
type CounterAction =
  | { type: 'increment' }
  | { type: 'decrement' }
  | { type: 'setStep'; payload: number };

function dispatchAction(action: CounterAction) {
  switch (action.type) {
    case 'increment':
      // no payload
      break;
    case 'setStep':
      console.log(action.payload); // number
      break;
  }
}
```

**Expected Output:** `action.payload` is only accessible in the `'setStep'` case, where it is typed as `number`.

### Real-World Cases

- **Redux reducers:** Action unions for every dispatched action.
- **`useReducer`:** The action union passed to the reducer.
- **Event buses:** Typed events across modules.

### References

- Discriminated Unions in ReactJS — https://tsecurity.de/
- Records and Discriminated Union Access — https://stackoverflow.com/

---

## 11. Typing Selectors

### Definitions

**Core Definition:** Typing selectors means annotating selector functions with the root state type and their return type, and deriving memoized selectors with `createSelector`.

**Technical Definition:** A selector is a function `(state: RootState) => Result`. Typing selectors with `RootState` gives autocomplete and type safety when accessing nested state. Memoized selectors created with `createSelector` infer their return type from the input selectors and the result function. Selectors should not be typed with `any` — the root state type is the single source of truth.

**Beginner-Friendly Explanation:** A selector takes the whole state and returns a piece of it. Type it with the root state type so TypeScript knows what properties exist.

### Purposes

- To get type-safe access to nested state in selectors.
- To infer memoized selector return types automatically.
- To keep state shape and selector contracts in sync.

### Syntax Rules and Structure

```tsx
interface RootState {
  todos: { items: Todo[]; filter: string };
}

const selectTodos = (state: RootState) => state.todos.items;

const selectVisibleTodos = createSelector(
  [selectTodos, (state: RootState) => state.todos.filter],
  (todos, filter) => todos.filter((t) => filter === 'all' || t.status === filter)
);
```

**Constraints and Limitations:**

- The root state type must be defined once and reused.
- Do not use `any` for state in selectors — it defeats type safety.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Typed Memoized Selector

```tsx
import { createSelector } from '@reduxjs/toolkit';

interface RootState {
  cart: { items: CartItem[]; taxRate: number };
}

const selectItems = (state: RootState) => state.cart.items;
const selectTaxRate = (state: RootState) => state.cart.taxRate;

const selectTotalWithTax = createSelector(
  [selectItems, selectTaxRate],
  (items, taxRate) => items.reduce((sum, i) => sum + i.price * i.qty, 0) * (1 + taxRate)
);
```

**Expected Output:** `selectTotalWithTax(state)` returns `number`; accessing `state.cart.unknown` is a compile error.

### Real-World Cases

- **E-commerce carts:** Computing totals, tax, and shipping.
- **Filtered lists:** Deriving visible items from filters.
- **Dashboard metrics:** Aggregating counts and averages.

### References

- Redux — Writing Tests: Selectors — https://redux.js.org/usage/writing-tests

---

## 12. Typing Generics

### Definitions

**Core Definition:** Generic components are components parameterized by one or more type variables, allowing them to work with different data types while preserving type safety at each call site.

**Technical Definition:** A generic component declares `<T>` in its function signature: `function List<T>({ items, renderItem }: ListProps<T>)`. TypeScript infers `T` from the props, so `items` and `renderItem` are linked. Constraints (`T extends { id: string }`) restrict what `T` can be. Generics are especially valuable for list, select, and table components.

**Beginner-Friendly Explanation:** A generic component is a component that works with any type. You write `<List<T> ...>` and TypeScript figures out `T` from what you pass in.

### Purposes

- To build reusable components that preserve the caller's type.
- To avoid duplicating components for each data type.
- To constrain types with `extends` and `keyof`.

### Syntax Rules and Structure

```tsx
interface ListProps<T> {
  items: T[];
  renderItem: (item: T) => ReactNode;
}

function List<T>({ items, renderItem }: ListProps<T>) {
  return <ul>{items.map((item, i) => <li key={i}>{renderItem(item)}</li>)}</ul>;
}

// Usage: T is inferred as User
<List items={users} renderItem={(user) => user.name} />
```

**Constraints and Limitations:**

- `React.FC` does not support generic components — use plain function declarations.
- Keep constraints simple and well-named.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Generic List with Constraint

```tsx
interface Identifiable { id: string; }

function List<T extends Identifiable>({ items }: { items: T[] }) {
  return <ul>{items.map((item) => <li key={item.id}>{String(item)}</li>)}</ul>;
}

// T must have an id
List<{ id: string; name: string }>({ items: [{ id: '1', name: 'Alice' }] });
```

**Expected Output:** `items` must be an array of objects with an `id` string; passing an array without `id` is a compile error.

### Real-World Cases

- **List components:** Generic lists with typed item renderers.
- **Select components:** Generic selects with typed option values.
- **Table components:** Generic tables with typed columns and rows.

### References

- TypeScript React Patterns — Generic Components — https://raw.githubusercontent.com/voku/agent-skills/refs/heads/copilot/update-php-best-practices/skills/typescript-react-patterns/AGENTS.md

---

## 13. Typing Polymorphic Components

### Definitions

**Core Definition:** A polymorphic component is a component that can render as different HTML elements or custom components based on an `as` prop, while preserving the correct prop types for whatever element it renders.

**Technical Definition:** The polymorphic pattern uses a generic `T extends ElementType`, an `as?: T` prop, and `ComponentPropsWithoutRef<T>` to merge the component's own props with the props of the rendered element. The component resolves `const Component = as || 'button'` and spreads the remaining props onto it. This provides full autocomplete and type checking for the rendered element without `any`.

**Beginner-Friendly Explanation:** A polymorphic component can be a `<button>`, an `<a>`, or a custom `<Link>` depending on an `as` prop. TypeScript knows which props are valid for whichever element you choose.

### Purposes

- To build one component that renders as different elements (button, link, etc.) with correct prop typing.
- To avoid `any` or unsafe casts in flexible component APIs.
- To enable design systems with polymorphic primitives.

### Syntax Rules and Structure

```tsx
import type { ElementType, ComponentPropsWithoutRef, ReactNode } from 'react';

interface BaseProps {
  variant?: 'primary' | 'secondary';
  children: ReactNode;
}

type PolymorphicProps<T extends ElementType> = BaseProps & {
  as?: T;
} & ComponentPropsWithoutRef<T>;

function Button<T extends ElementType = 'button'>({
  as,
  variant = 'primary',
  children,
  ...props
}: PolymorphicProps<T>) {
  const Component = as || 'button';
  return <Component className={`btn-${variant}`} {...props}>{children}</Component>;
}
```

**Constraints and Limitations:**

- Requires TypeScript 4.1+ for `ComponentPropsWithoutRef`.
- The complexity is justified only for truly reusable primitives — do not make every component polymorphic.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Polymorphic Button

```tsx
// Renders as a <button>
<Button onClick={() => {}}>Click me</Button>

// Renders as an <a> — href is typed correctly
<Button as="a" href="/about">About</Button>

// Renders as a custom Link — Link's props are typed
<Button as={Link} to="/home">Home</Button>
```

**Expected Output:** `onClick` is valid on the button; `href` is valid on the anchor; `to` is valid on the custom Link. Using `href` on the button produces a compile error.

### Real-World Cases

- **Design system primitives:** Buttons, headings, and text components that can render as different elements.
- **Routing-aware components:** Buttons that render as `<Link>` for client-side navigation.
- **Layout components:** Box and Flex components that render as any element.

### References

- Steve Kinney — Polymorphic Components and the `as` Prop — https://stevekinney.com/courses/react-typescript/polymorphic-components-and-as-prop

---

## 14. Typing Best Practices

### Definitions

**Core Definition:** React TypeScript typing best practices are the guidelines that keep component types safe, readable, and maintainable across a codebase.

**Technical Definition:** Best practices include: **avoid `React.FC`** and use plain function declarations with explicit props interfaces; **prefer interfaces for props** (extendable, better errors) and type aliases for unions and intersections; **use `ReactNode` for children** and `ReactElement` only when a single element is required; **use destructuring defaults** instead of `defaultProps`; **type event handlers with specific SyntheticEvent generics**; **type refs with the correct DOM element**; **type context with a null default and a throwing custom hook**; **model state and actions with discriminated unions** and exhaustive `never` checks; and **enable `strict: true` and `noUncheckedIndexedAccess`** in `tsconfig.json`.

**Beginner-Friendly Explanation:** Use interfaces for props, plain functions instead of `React.FC`, `ReactNode` for children, destructuring for defaults, and discriminated unions for state. Turn on strict mode. These habits keep your types correct and your code clean.

### Purposes

- To maximize type safety across the component tree.
- To keep component APIs consistent and predictable.
- To avoid the known pitfalls of `React.FC`, `defaultProps`, and `any`.
- To make refactoring safe and code reviews fast.

### Syntax Rules and Structure

**Best practices checklist:**

| Practice | Description |
|----------|-------------|
| **No `React.FC`** | Plain function declarations with explicit props. |
| **Interface for props** | Extendable and better error messages. |
| **`ReactNode` for children** | Anything renderable. |
| **Destructuring defaults** | Not `defaultProps`. |
| **Specific event types** | `ChangeEvent<HTMLInputElement>`, not `any`. |
| **Typed refs** | `useRef<HTMLInputElement>(null)`. |
| **Null-checked context** | `createContext<T \| null>(null)` + throwing hook. |
| **Discriminated unions** | For reducer state and actions. |
| **Strict tsconfig** | `strict: true`, `noUncheckedIndexedAccess`. |

**Constraints and Limitations:**

- Strict flags can be noisy initially — the long-term benefit outweighs the friction.
- Polymorphic and generic components add complexity — use them only when reuse justifies it.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Best-Practice Component

```tsx
import type { ReactNode, ChangeEvent } from 'react';

interface SearchBoxProps {
  label: string;
  value: string;
  onChange: (event: ChangeEvent<HTMLInputElement>) => void;
  children?: ReactNode;
}

function SearchBox({ label, value, onChange, children }: SearchBoxProps) {
  return (
    <label>
      {label}
      <input type="search" value={value} onChange={onChange} />
      {children}
    </label>
  );
}
```

**Why this follows best practices:** Plain function (no `React.FC`), interface for props, specific `ChangeEvent` type, `ReactNode` for optional children, and no `any`.

### Real-World Cases

- **Design systems:** Every component follows the same typing patterns.
- **Code reviews:** These rules provide a checklist for reviewers.
- **Strict CI:** `tsc --noEmit` enforces the rules on every build.

### References

- React KB — TypeScript with React — https://github.com/Nguyen-Mau-Anh/react-kb/blob/main/05.routing-and-styling/03-typescript-with-react.md
- Design System AGENTS.md — Avoid `React.FC` — https://code.overheid.nl/
- TypeScript React Patterns — https://raw.githubusercontent.com/voku/agent-skills/refs/heads/copilot/update-php-best-practices/skills/typescript-react-patterns/AGENTS.md

---

## References

- React TypeScript Cheatsheet — https://react-typescript-cheatsheet.netlify.app
- React + TypeScript Guide — https://react.dev/learn/typescript
- TypeScript Handbook — https://www.typescriptlang.org/docs/handbook/
- TypeScript React Patterns — Complete Reference — https://raw.githubusercontent.com/voku/agent-skills/refs/heads/copilot/update-php-best-practices/skills/typescript-react-patterns/AGENTS.md
- React KB — TypeScript with React — https://github.com/Nguyen-Mau-Anh/react-kb/blob/main/05.routing-and-styling/03-typescript-with-react.md
- Backstage ADR — Avoid `React.FC` — https://github.com/backstage/backstage/blob/master/docs/architecture-decisions/adr006-avoid-react-fc.md
- Steve Kinney — Polymorphic Components and the `as` Prop — https://stevekinney.com/courses/react-typescript/polymorphic-components-and-as-prop
- useReducer Typing — https://raw.githubusercontent.com/AsyrafHussin/agent-skills/1aa0ff717c10309226c9e678f00873976450fd76/skills/typescript-react-patterns/rules/hook-use-reducer.md
- Typing Default Props — https://github.com/dot-skills/skills/blob/main/skills/.experimental/typescript-refactor/references/default-props.md
- CoreUI — How to set default props in React — https://coreui.io/blog/how-to-set-default-props-in-react/
- Redux — Writing Tests: Selectors — https://redux.js.org/usage/writing-tests
- Educative — `useReducer` Hook with Discriminated Unions — https://www.educative.io/
- Discriminated Unions in ReactJS — https://tsecurity.de/
- Records and Discriminated Union Access — https://stackoverflow.com/