# `useContext` Hook — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** `useContext` is a React Hook that lets a component read and subscribe to a context value from the nearest provider above it in the component tree.

**Technical Definition:** `useContext(SomeContext)` is a React Hook that accepts a context object (created by `createContext`) and returns the current context value for the calling component. React determines the value by searching the component tree upward from the calling component and finding the closest matching context provider. If no provider exists above the component, the hook returns the `defaultValue` passed to `createContext`. React automatically re-renders any component that calls `useContext` when the context value changes, comparing the previous and next values with `Object.is`.

**Beginner-Friendly Explanation:** `useContext` is a way for a component to "tune in" to a shared value that lives somewhere above it in the component tree. Instead of passing data down through every intermediate component (called prop drilling), you place the data in a context provider at a higher level, and any component below can read it directly with `useContext`. Think of it like a radio broadcast: the provider is the station, and `useContext` is the radio that picks up the signal.

### Key Characteristics

- **Prop drilling alternative:** Eliminates the need to pass props through every intermediate component.
- **Subscription-based:** Components automatically re-render when the context value they read changes.
- **Nearest provider wins:** `useContext` always reads from the closest matching provider above the calling component.
- **Default value fallback:** Returns the `defaultValue` from `createContext` when no provider exists above.
- **Top-level only:** Must be called at the top level of a component or custom Hook.
- **Reference-based comparison:** React uses `Object.is` to compare context values, so new object references always trigger re-renders.

### Prerequisites

- JavaScript functions, objects, and destructuring.
- React function components and JSX.
- Understanding of props and one-way data flow.
- Familiarity with `useState` and component re-rendering.

### Related Programming Areas

- **Props** — the primary data-passing mechanism; Context is an alternative for deeply nested data.
- **State Management** — Redux, Zustand, Jotai for complex global state.
- **`useReducer`** — often combined with Context for shared state logic.
- **Provider Pattern** — the architectural pattern that Context implements.

---

## 1. `useContext`

### Definitions

**Core Definition:** `useContext` is a React Hook that reads and subscribes to a context value from the nearest provider above the component.

**Technical Definition:** `const value = useContext(SomeContext)` is called at the top level of a component to read the current context value. The `SomeContext` parameter is a context object previously created with `createContext`. React returns the value passed to the closest `<SomeContext>` provider above the calling component. If no such provider exists, the hook returns the `defaultValue` passed to `createContext`. The returned value is always up-to-date, and React automatically re-renders the component when the context value changes.

**Beginner-Friendly Explanation:** `useContext` is how you read a shared value from a context. You pass it the context object, and it gives you the current value from the nearest provider.

### Purposes

- To read a context value without passing props through intermediate components.
- To subscribe to context changes so the component re-renders when the value updates.
- To access data provided by an ancestor component at any depth.
- To implement the Provider pattern for dependency injection.

### Syntax Rules and Structure

**General Syntax:**

```jsx
import { useContext } from 'react';

const value = useContext(SomeContext);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `SomeContext` | A context object created with `createContext`. |
| `value` | The current context value from the nearest provider. |

**Syntax Rules:**

1. Call `useContext` at the top level of a component or custom Hook.
2. The context object must be created with `createContext` before use.
3. `useContext` searches upward from the calling component and does not consider providers in the same component.
4. React re-renders the component when the context value changes.

**Constraints and Limitations:**

- Cannot be called inside loops or conditions.
- The `SomeContext` object used to provide context and the one used to read it must be exactly the same object (compared with `===`).
- Context does not optimise re-renders by itself; it is a broadcast mechanism.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Basic Context Consumption

```jsx
import { createContext, useContext } from 'react';

// Step 1: Create the context with a default value
const ThemeContext = createContext('light');

// Step 2: Create a provider component
function App() {
  return (
    <ThemeContext value="dark">
      <Toolbar />
    </ThemeContext>
  );
}

// Step 3: Consume the context in a deeply nested component
function Toolbar() {
  return (
    <div>
      <ThemedButton />
    </div>
  );
}

function ThemedButton() {
  // Step 4: Read the context value
  const theme = useContext(ThemeContext);
  return <button className={theme}>Themed Button</button>;
}
```

**Expected Output:** A button with `className="dark"`.

**Why this output:** `ThemedButton` calls `useContext(ThemeContext)`. React searches upward, finds the `<ThemeContext value="dark">` provider in `App`, and returns `"dark"` as the value. The button uses this value as its class name.

#### Example 2: Context with Changing Values

```jsx
import { createContext, useContext, useState } from 'react';

const CountContext = createContext(0);

function App() {
  const [count, setCount] = useState(0);

  return (
    <CountContext value={count}>
      <button onClick={() => setCount(c => c + 1)}>Increment</button>
      <Display />
    </CountContext>
  );
}

function Display() {
  const count = useContext(CountContext);
  return <p>Count: {count}</p>;
}
```

**Expected Output:** The count updates in `Display` when the button is clicked.

**Why this output:** When `count` changes, the provider's value changes. React re-renders all components that read `CountContext`, including `Display`.

### Real-World Cases

- **Theming:** Light/dark mode values shared across all components.
- **Authentication:** Current user data accessible anywhere in the app.
- **Localisation:** Language and locale settings shared globally.
- **Routing:** React Router uses Context to track the current route.

### References

- React — useContext — https://de.react.dev/reference/react/useContext
- React — Passing Data Deeply with Context — https://18.react.dev/learn/passing-data-deeply-with-context

---

## 2. Context

### Definitions

**Core Definition:** Context is React's mechanism for passing data through the component tree without manually passing props at every level.

**Technical Definition:** Context provides a way to share values like the current theme, user, or locale between components without explicitly passing a prop through every level of the tree. A context is created with `createContext`, provided at a higher level with a provider, and consumed at any depth with `useContext`. Context is designed for data that can be considered "global" for a subtree of React components.

**Beginner-Friendly Explanation:** Context is like a global variable for a part of your component tree. You put a value into a context at the top, and any component below can read it without you having to pass it down through every level.

### Purposes

- To avoid prop drilling when data is needed by deeply nested components.
- To share data that is genuinely global for a subtree (theme, auth, locale).
- To enable dependency injection in React applications.
- To decouple data provision from data consumption.

### Syntax Rules and Structure

**General Syntax (Creation):**

```jsx
import { createContext } from 'react';

const MyContext = createContext(defaultValue);
```

**General Syntax (Providing):**

```jsx
<MyContext value={someValue}>
  {children}
</MyContext>
```

**General Syntax (Consuming):**

```jsx
const value = useContext(MyContext);
```

**Syntax Rules:**

1. `createContext` must be called outside any component.
2. The provider's `value` prop can be of any type.
3. The context object itself does not hold information; it represents which context to read or provide.

**Constraints and Limitations:**

- Context is not a state management solution on its own; it is a transport mechanism.
- Overusing Context creates hidden coupling between components.

### Annotated Code Examples

```jsx
import { createContext, useContext } from 'react';

// Create the context
const ThemeContext = createContext('light');

function App() {
  return (
    <ThemeContext value="dark">
      <Page />
    </ThemeContext>
  );
}

function Page() {
  return <Panel title="Welcome" />;
}

function Panel({ title }) {
  const theme = useContext(ThemeContext);
  return <div className={`panel-${theme}`}>{title}</div>;
}
```

**Expected Output:** A panel with `className="panel-dark"`.

**Why this output:** The `ThemeContext` is provided with value `"dark"` at the top. `Panel` reads it with `useContext`, regardless of how many intermediate components exist between `App` and `Panel`.

### Real-World Cases

- **Design systems:** Theme context for consistent styling.
- **Multi-tenant apps:** Tenant context for brand-specific data.
- **Feature flags:** Context for enabling/disabling features.

### References

- React — Context — https://mintlify.wiki/facebook/react/guides/context
- React — Passing Data Deeply with Context — https://18.react.dev/learn/passing-data-deeply-with-context

---

## 3. `React.createContext`

### Definitions

**Core Definition:** `createContext` is a React function that creates a context object which components can provide or read.

**Technical Definition:** `createContext(defaultValue)` is called outside any component to create a context. It returns a context object that does not hold any information itself; it represents which context other components read or provide. The context object has a Provider component (or can be rendered directly as `<SomeContext>` in React 19+) and is consumed with `useContext`.

**Beginner-Friendly Explanation:** `createContext` is how you make a new context. You give it a default value, and it gives you back a context object that you can use to provide and read data.

### Purposes

- To create a context object for sharing data.
- To define a default value as a fallback.
- To enable the Provider pattern.

### Syntax Rules and Structure

```jsx
import { createContext } from 'react';

const MyContext = createContext(defaultValue);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `defaultValue` | Fallback value when no provider exists above. |
| `MyContext` | The context object. |

**Syntax Rules:**

1. Call `createContext` outside any component.
2. Pass `null` if there is no meaningful default value.
3. The default value is static and never changes over time.

**Constraints and Limitations:**

- The default value is only used when no provider exists above the component.
- Passing an empty object as default is an anti-pattern; use `undefined` or a meaningful value.

### Annotated Code Examples

```jsx
import { createContext } from 'react';

// Create context with a default value
const ThemeContext = createContext('light');

// Create context with no meaningful default
const AuthContext = createContext(null);
```

**Expected Output:** Two context objects are created.

**Why this output:** `createContext` returns a context object. The `defaultValue` is used only if no provider is found above the consuming component.

### Real-World Cases

- **Theming:** `createContext('light')`.
- **Authentication:** `createContext(null)`.
- **Localisation:** `createContext('en')`.

### References

- React — createContext — https://uk.react.dev/reference/react/createContext

---

## 4. Context Provider

### Definitions

**Core Definition:** A context provider is a component that supplies a context value to all components below it in the tree.

**Technical Definition:** The provider is rendered as `<SomeContext value={...}>` (React 19+) or `<SomeContext.Provider value={...}>` (legacy). It accepts a `value` prop and makes that value available to all components inside it, no matter how deep. A component calling `useContext` inside the provider receives the value of the innermost matching provider above it.

**Beginner-Friendly Explanation:** The provider is where you put the data into the context. Everything wrapped inside the provider can read that data.

### Purposes

- To supply a context value to a subtree of components.
- To make data available without prop drilling.
- To scope context values to a specific part of the component tree.

### Syntax Rules and Structure

**React 19+:**

```jsx
<MyContext value={someValue}>
  {children}
</MyContext>
```

**Legacy (React 18 and earlier):**

```jsx
<MyContext.Provider value={someValue}>
  {children}
</MyContext.Provider>
```

**Syntax Rules:**

1. The `value` prop can be of any type.
2. A component reads from the innermost matching provider above it.
3. In React 19+, you can render `<SomeContext>` directly as a provider.

**Constraints and Limitations:**

- Nesting multiple providers of the same context creates "provider hell".
- The provider's value should be stable to avoid unnecessary re-renders.

### Annotated Code Examples

```jsx
import { createContext, useContext, useState } from 'react';

const ThemeContext = createContext('light');

function App() {
  const [theme, setTheme] = useState('light');

  return (
    <ThemeContext value={theme}>
      <Toolbar />
      <button onClick={() => setTheme(t => t === 'light' ? 'dark' : 'light')}>
        Toggle Theme
      </button>
    </ThemeContext>
  );
}

function Toolbar() {
  return <ThemedButton />;
}

function ThemedButton() {
  const theme = useContext(ThemeContext);
  return <button className={theme}>Themed Button</button>;
}
```

**Expected Output:** A button that changes its class when the theme toggles.

**Why this output:** The `ThemeContext` provider supplies the current theme value. `ThemedButton` reads it with `useContext` and re-renders when the value changes.

### Real-World Cases

- **Theme providers:** Supplying light/dark mode to the entire app.
- **Auth providers:** Supplying the current user to all components.
- **Locale providers:** Supplying language settings globally.

### References

- React — createContext: Provider — https://uk.react.dev/reference/react/createContext#provider
- React — Passing Data Deeply with Context — https://18.react.dev/learn/passing-data-deeply-with-context

---

## 5. Context Consumer

### Definitions

**Core Definition:** A context consumer is a component or Hook that reads a context value from the nearest provider above it.

**Technical Definition:** The modern way to consume context is with the `useContext` Hook. Before Hooks, the legacy way was `<SomeContext.Consumer>`, which takes a function as a child and calls it with the current context value. The Consumer approach is still supported but not recommended for new code. `useContext` is the recommended approach.

**Beginner-Friendly Explanation:** The consumer is how you read the data from a context. The modern way is `useContext`; the old way was `<Context.Consumer>`.

### Purposes

- To read a context value from the nearest provider.
- To subscribe to context changes.
- To access data provided by an ancestor component.

### Syntax Rules and Structure

**Modern (useContext):**

```jsx
const value = useContext(MyContext);
```

**Legacy (Consumer):**

```jsx
<MyContext.Consumer>
  {value => <div>{value}</div>}
</MyContext.Consumer>
```

**Syntax Rules:**

1. `useContext` is the recommended way to consume context.
2. The Consumer component takes a function as its child.
3. React re-runs the Consumer's function when the context value changes.

**Constraints and Limitations:**

- The Consumer pattern is legacy and not recommended for new code.
- `useContext` is simpler and more readable.

### Annotated Code Examples

```jsx
import { createContext, useContext } from 'react';

const ThemeContext = createContext('light');

// ✅ Recommended: useContext
function ThemedButton() {
  const theme = useContext(ThemeContext);
  return <button className={theme}>Themed</button>;
}

// ❌ Legacy: Consumer (not recommended)
function LegacyButton() {
  return (
    <ThemeContext.Consumer>
      {theme => <button className={theme}>Legacy</button>}
    </ThemeContext.Consumer>
  );
}
```

**Expected Output:** Both buttons render with the theme class.

**Why this output:** `useContext` and the Consumer both read the same context value. `useContext` is preferred because it is simpler and aligns with the Hooks pattern.

### Real-World Cases

- **Legacy codebases:** Maintaining older React code that uses Consumer.
- **Migration:** Converting Consumer components to `useContext`.

### References

- React — createContext: Consumer — https://uk.react.dev/reference/react/createContext#consumer

---

## 6. Context Value

### Definitions

**Core Definition:** The context value is the data supplied by the provider and read by consumers.

**Technical Definition:** The `value` prop on a provider can be of any type: a primitive, object, array, function, or JSX element. React compares the previous and next values using `Object.is`. If the value is a new object or array reference on every render, all consumers re-render even if the contents are unchanged. Therefore, the value should be stabilised with `useMemo` or `useCallback` when it is an object or function.

**Beginner-Friendly Explanation:** The context value is the data you put into the context. It can be anything: a string, a number, an object, or a function.

### Purposes

- To supply data to consumers.
- To enable state sharing across the component tree.
- To provide callbacks and actions alongside state.

### Syntax Rules and Structure

```jsx
<MyContext value={{ user, setUser }}>
  {children}
</MyContext>
```

**Syntax Rules:**

1. The value can be of any type.
2. React compares values with `Object.is`.
3. Stabilise object and function values with `useMemo` and `useCallback` to avoid unnecessary re-renders.

**Constraints and Limitations:**

- A new object reference on every render triggers re-renders in all consumers.
- Context values should be kept as small and stable as possible.

### Annotated Code Examples

```jsx
import { createContext, useState, useMemo } from 'react';

const UserContext = createContext(null);

function UserProvider({ children }) {
  const [user, setUser] = useState(null);

  // ✅ Stable value with useMemo
  const value = useMemo(() => ({ user, setUser }), [user]);

  return (
    <UserContext value={value}>
      {children}
    </UserContext>
  );
}
```

**Expected Output:** The context value only changes when `user` changes.

**Why this output:** Without `useMemo`, the object `{ user, setUser }` is recreated on every render, causing all consumers to re-render even when `user` is unchanged. `useMemo` keeps the reference stable until `user` changes.

### Real-World Cases

- **Auth state:** Providing `{ user, login, logout }` as a stable value.
- **Theme state:** Providing `{ theme, toggleTheme }` as a stable value.
- **Cart state:** Providing `{ items, addItem, removeItem }` as a stable value.

### References

- React — useContext: Caveats — https://de.react.dev/reference/react/useContext#caveats
- Steve Kinney — Separating Actions from State with Two Contexts — https://stevekinney.com

---

## 7. Context Default Value

### Definitions

**Core Definition:** The context default value is the fallback value returned by `useContext` when no matching provider exists above the calling component.

**Technical Definition:** The `defaultValue` argument to `createContext` is used only when a component reads context but there is no matching provider above it in the tree. The default value is meant as a "last resort" fallback and is static — it never changes over time. If there is no meaningful default value, `null` should be specified.

**Beginner-Friendly Explanation:** The default value is what you get if there is no provider above the component. It is a fallback, not the primary way to supply data.

### Purposes

- To provide a fallback when no provider exists.
- To avoid crashes when context is missing.
- To define a sensible default for standalone component usage.

### Syntax Rules and Structure

```jsx
const MyContext = createContext(defaultValue);
```

**Syntax Rules:**

1. The default value is used only when no provider exists above.
2. Pass `null` if there is no meaningful default.
3. The default value is static and never changes.

**Constraints and Limitations:**

- Do not pass an empty object as default; use `undefined` or a meaningful value.
- Relying on the default value can silently hide missing providers.

### Annotated Code Examples

```jsx
import { createContext, useContext } from 'react';

// Default value is 'light'
const ThemeContext = createContext('light');

function Button() {
  const theme = useContext(ThemeContext);
  // If no provider exists, theme is 'light'
  return <button className={theme}>Click</button>;
}

// Without a provider, the default value is used
function App() {
  return <Button />; // theme = 'light'
}
```

**Expected Output:** A button with `className="light"`.

**Why this output:** No provider exists above `Button`, so `useContext` returns the default value `'light'`.

### Real-World Cases

- **Standalone components:** Components that work with or without a provider.
- **Testing:** Providing defaults for unit tests.
- **Fallback themes:** A default theme when no theme provider is present.

### References

- React — createContext: Parameters — https://uk.react.dev/reference/react/createContext#parameters

---

## 8. Context Nesting

### Definitions

**Core Definition:** Context nesting is the practice of placing multiple context providers inside one another to supply different contexts to different parts of the tree.

**Technical Definition:** React allows multiple context providers to be nested. Each provider supplies its own context value to the components below it. When providers of the same context are nested, the innermost provider's value overrides the outer one for its subtree. Nesting different contexts allows independent data to be shared at different levels. However, deep nesting creates "provider hell" or "wrapper hell," which makes the component tree harder to read and maintain.

**Beginner-Friendly Explanation:** You can put providers inside other providers. Each one provides its own data. Just be careful not to nest too many, or your code becomes hard to read.

### Purposes

- To supply different contexts to different subtrees.
- To override a context value for a specific part of the tree.
- To compose multiple providers for an application.

### Syntax Rules and Structure

```jsx
<ThemeContext value="dark">
  <AuthContext value={user}>
    <CartContext value={cart}>
      {children}
    </CartContext>
  </AuthContext>
</ThemeContext>
```

**Syntax Rules:**

1. Providers can be nested to any depth.
2. The innermost provider of the same context wins.
3. Different contexts can be nested independently.

**Constraints and Limitations:**

- Deep nesting creates "provider hell".
- Consider composing providers into a single component to reduce nesting.

### Annotated Code Examples

```jsx
function AppProviders({ children }) {
  return (
    <ThemeProvider>
      <AuthProvider>
        <CartProvider>
          {children}
        </CartProvider>
      </AuthProvider>
    </ThemeProvider>
  );
}

function App() {
  return (
    <AppProviders>
      <Main />
    </AppProviders>
  );
}
```

**Expected Output:** All contexts are available to `Main` and its descendants.

**Why this output:** The nested providers supply their respective contexts to the entire subtree. Composing them into `AppProviders` keeps `App` clean.

### Real-World Cases

- **Application root:** Composing theme, auth, and locale providers.
- **Feature modules:** Scoping a context to a specific feature.
- **Testing:** Overriding a context value for a specific test case.

### References

- GitHub — React Context Composition — https://github.com/Compile-N-Run/Compile-N-Run

---

## 9. Context Performance

### Definitions

**Core Definition:** Context performance refers to the re-rendering behaviour caused by context value changes and strategies to optimise it.

**Technical Definition:** Context is a broadcast mechanism, not an optimisation tool. When a provider's `value` changes, React re-renders all components that read that context, regardless of whether they use the specific part of the value that changed. React compares the previous and next values with `Object.is`. If the value is a new object reference on every render, all consumers re-render. Performance optimisations include: splitting contexts (separating state from actions), memoising the value with `useMemo`, and using component composition to avoid unnecessary consumer re-renders.

**Beginner-Friendly Explanation:** Context can cause performance problems if not used carefully. When the value changes, every component that reads it re-renders. To avoid this, split your contexts and keep values stable.

### Purposes

- To understand why context changes cause re-renders.
- To optimise context usage for performance.
- To avoid re-render cascades.

### Syntax Rules and Structure

**❌ Performance Pitfall: Single Mega Context**

```jsx
const AppContext = createContext(null);

function AppProvider({ children }) {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');

  // New object every render — all consumers re-render!
  return (
    <AppContext value={{ user, theme, setUser, setTheme }}>
      {children}
    </AppContext>
  );
}
```

**✅ Optimised: Split Contexts with Stable Values**

```jsx
const UserContext = createContext(null);
const ThemeContext = createContext('light');

function AppProvider({ children }) {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');

  const userValue = useMemo(() => ({ user, setUser }), [user]);
  const themeValue = useMemo(() => ({ theme, setTheme }), [theme]);

  return (
    <UserContext value={userValue}>
      <ThemeContext value={themeValue}>
        {children}
      </ThemeContext>
    </UserContext>
  );
}
```

**Syntax Rules:**

1. Split contexts by domain (state vs actions, user vs theme).
2. Stabilise object values with `useMemo`.
3. Use component composition to avoid wrapping entire trees unnecessarily.

**Constraints and Limitations:**

- `React.memo` does not prevent re-renders from context changes.
- Context is not a replacement for a dedicated state management library in large apps.

### Annotated Code Examples

```jsx
// ❌ Bad: every consumer re-renders when any value changes
<AppContext value={{ user, theme, cart }}>

// ✅ Good: consumers only re-render when their context changes
<UserContext value={userValue}>
  <ThemeContext value={themeValue}>
    <CartContext value={cartValue}>
      {children}
    </CartContext>
  </ThemeContext>
</UserContext>
```

**Expected Output:** In the good version, changing the theme does not re-render components that only read the user context.

**Why this output:** Splitting contexts means each provider broadcasts independently. Components only subscribe to the context they read, so they only re-render when that specific context changes.

### Real-World Cases

- **Large applications:** Splitting contexts by domain.
- **Performance-critical UIs:** Avoiding re-render cascades.
- **Form state:** Separating form state from form actions.

### References

- Steve Kinney — Context API Performance Pitfalls — https://stevekinney.com/courses/react-performance/context-api-performance-pitfalls
- Steve Kinney — Separating Actions from State with Two Contexts — https://stevekinney.com

---

## 10. Context vs Props

### Definitions

**Core Definition:** Context and props are two mechanisms for passing data in React; props are explicit and local, while context is implicit and global to a subtree.

**Technical Definition:** Props are passed explicitly from parent to child and are the default way to share data in React. Context allows a parent component to make data available to any component in the tree below it, no matter how deep, without passing it explicitly through props. Props are preferred for direct parent-child communication; context is preferred when many components at different levels need the same data, or when prop drilling becomes cumbersome.

**Beginner-Friendly Explanation:** Props are like handing something directly to someone. Context is like making an announcement that everyone in the room can hear.

### Purposes

- **Props:** To pass data explicitly and predictably between adjacent components.
- **Context:** To avoid prop drilling when data is needed deep in the tree.
- **Props:** To make data flow easy to trace.
- **Context:** To share global data without repetitive prop passing.

### Syntax Rules and Structure

| Aspect | Props | Context |
|--------|-------|---------|
| Explicitness | Explicit — visible at each level | Implicit — data appears without props |
| Scope | Parent to direct child | Any descendant in the subtree |
| Use case | Adjacent components | Deeply nested, many consumers |
| Tracing | Easy to trace | Harder to trace |

**Syntax Rules:**

1. Use props by default.
2. Use context when props become verbose or inconvenient.
3. Do not use context for every piece of data; it creates hidden coupling.

### Annotated Code Examples

```jsx
// ✅ Props: direct parent-child
function Parent() {
  return <Child name="Alice" />;
}

// ✅ Context: deeply nested data
const UserContext = createContext(null);

function App() {
  return (
    <UserContext value={{ name: 'Alice' }}>
      <Layout />
    </UserContext>
  );
}

function Layout() {
  return <Sidebar />;
}

function Sidebar() {
  const user = useContext(UserContext);
  return <p>{user.name}</p>;
}
```

**Expected Output:** Both approaches render "Alice". Props are used for direct passing; context is used to avoid drilling through `Layout`.

**Why this output:** Props are explicit and local. Context avoids passing `user` through `Layout`, which does not need it.

### Real-World Cases

- **Props:** Component configuration, event handlers, simple data.
- **Context:** Theme, auth, locale, routing.

### References

- React — Passing Data Deeply with Context — https://18.react.dev/learn/passing-data-deeply-with-context

---

## 11. Context vs State Management

### Definitions

**Core Definition:** Context is a data-passing mechanism, while state management libraries (Redux, Zustand) are complete state management solutions with additional features.

**Technical Definition:** React's Context API provides a built-in way to manage global state without prop drilling. Unlike third-party libraries like Redux or MobX, Context API requires no additional setup, making it suitable for small to moderately complex apps. Redux and MobX offer powerful features for large-scale applications, but introduce complexity and boilerplate. Context is best for sharing data that changes infrequently (theme, auth, locale). For complex state with frequent updates, middleware, and advanced debugging, a dedicated library is preferred.

**Beginner-Friendly Explanation:** Context is great for simple shared data like themes and user info. For complex apps with lots of state changes, Redux or Zustand is better.

### Purposes

- **Context:** To share simple, infrequently changing data.
- **State libraries:** To manage complex state with many actions and frequent updates.
- **Context:** To avoid adding dependencies for simple use cases.

### Comparison Table

| Aspect | Context | Redux/Zustand |
|--------|---------|---------------|
| Setup | Built-in | Requires installation |
| Best for | Small–medium apps | Large, complex apps |
| Boilerplate | Low | Higher (Redux) / Low (Zustand) |
| Performance | Can cause re-renders | Optimised with selectors |
| Debugging | Basic | DevTools, time-travel |

**Syntax Rules:**

1. Use Context for simple, low-frequency shared data.
2. Use a state library for complex, high-frequency state.
3. Context and state libraries can be combined.

### Annotated Code Examples

```jsx
// ✅ Context: simple theme sharing
const ThemeContext = createContext('light');

// ✅ Zustand: complex state
import { create } from 'zustand';

const useStore = create((set) => ({
  items: [],
  addItem: (item) => set((state) => ({ items: [...state.items, item] })),
}));
```

**Expected Output:** Context handles simple data; Zustand handles complex state.

**Why this output:** Context is sufficient for theme data that changes rarely. Zustand is better for a shopping cart with frequent add/remove operations.

### Real-World Cases

- **Context:** Theme, auth, locale, feature flags.
- **Redux:** Large enterprise apps with complex state.
- **Zustand:** Medium apps needing simple global state.

### References

- IEEE — Web Application State Management: A Review of Leading React Frameworks — https://ieeexplore.ieee.org

---

## 12. Context Best Practices

### Definitions

**Core Definition:** Context best practices are guidelines for using Context effectively, safely, and performantly.

**Technical Definition:** Best practices include: creating custom hooks for context consumption (with runtime checks), splitting contexts by domain, memoising context values with `useMemo` and `useCallback`, using `undefined` as the default value, keeping context values small and stable, and scoping context providers as narrowly as possible. These practices prevent common pitfalls like hidden coupling, re-render cascades, and silent failures from missing providers.

**Beginner-Friendly Explanation:** These are the "do's and don'ts" that experienced React developers follow to keep their context usage clean and performant.

### Best Practices Table

| Practice | Description |
|----------|-------------|
| **Custom hook for consumption** | Wrap `useContext` in a custom hook with a runtime check. |
| **Split contexts** | Separate state and actions into different contexts. |
| **Memoise the value** | Use `useMemo` for objects and `useCallback` for functions. |
| **Use `undefined` as default** | Forces the custom hook to throw if no provider. |
| **Keep values small** | Avoid mega contexts; only include what is needed. |
| **Scope providers narrowly** | Wrap only the subtree that needs the context. |
| **Don't overuse Context** | Use props by default; context for genuinely global data. |

### Annotated Code Examples

```jsx
// ✅ Good: custom hook with runtime check
const UserContext = createContext(undefined);

export function useUser() {
  const context = useContext(UserContext);
  if (context === undefined) {
    throw new Error('useUser must be used within a UserProvider');
  }
  return context;
}

// ✅ Good: split contexts + memoised values
const UserStateContext = createContext(null);
const UserActionsContext = createContext(null);

function UserProvider({ children }) {
  const [user, setUser] = useState(null);
  const stateValue = useMemo(() => ({ user }), [user]);
  const actionsValue = useMemo(() => ({ setUser }), []);

  return (
    <UserStateContext value={stateValue}>
      <UserActionsContext value={actionsValue}>
        {children}
      </UserActionsContext>
    </UserStateContext>
  );
}
```

**Expected Output:** Clean, performant context with runtime safety.

**Why this output:** The custom hook throws a clear error if the provider is missing. Split contexts prevent unnecessary re-renders. Memoised values keep references stable.

### Real-World Cases

- **All applications using Context:** These practices apply universally.
- **Team projects:** Consistency reduces integration bugs.
- **Large codebases:** Best practices keep code maintainable.

### References

- Steve Kinney — Typing Context and Selector Patterns — https://stevekinney.com
- Feature-Sliced Design — React's Context API: Friend or Architectural Foe? — https://feature-sliced.design

---

## 13. Context Pitfalls

### Definitions

**Core Definition:** Context pitfalls are common mistakes developers make when using Context, leading to performance issues, hidden coupling, or bugs.

**Technical Definition:** Common pitfalls include: the mega context (everything in one context), not splitting state from actions, ignoring memo boundaries, reading context in the same component as the provider, missing provider guards, and overusing context for data that should be passed as props. These pitfalls cause re-render cascades, hidden dependencies, and silent failures.

**Beginner-Friendly Explanation:** These are the traps that catch developers when working with Context.

### Common Pitfalls

| Pitfall | Problem | Solution |
|---------|---------|----------|
| **Mega context** | Changing any value re-renders all consumers. | Split contexts by domain. |
| **Not splitting state/actions** | Actions change less often but trigger re-renders. | Use two contexts: one for state, one for actions. |
| **Unstable value** | New object every render causes all consumers to re-render. | Wrap in `useMemo`. |
| **Reading context in provider component** | `useContext` searches upward and won't find the same component's provider. | Move the consumer to a child component. |
| **Missing provider guard** | Silent failure returns `defaultValue`. | Custom hook that throws if context is `undefined`. |
| **Overusing context** | Hidden coupling; any component can depend on "the app". | Use props by default; context only for global data. |

### Annotated Code Examples

**Pitfall 1: Mega Context**

```jsx
// ❌ Bad: everything in one context
<AppContext value={{ user, theme, cart, notifications, ... }}>
```

**Pitfall 2: Unstable Value**

```jsx
// ❌ Bad: new object every render
<AppContext value={{ user, setUser }}>
```

**Pitfall 3: Missing Provider Guard**

```jsx
// ❌ Bad: silently returns default
const user = useContext(UserContext); // undefined if no provider

// ✅ Good: custom hook with check
function useUser() {
  const context = useContext(UserContext);
  if (context === undefined) throw new Error('Missing UserProvider');
  return context;
}
```

**Expected Output:** The good practices prevent re-render cascades and silent failures.

**Why this output:** Mega contexts cause all consumers to re-render on any change. Unstable values cause re-renders even when data is unchanged. Missing guards hide bugs.

### Real-World Cases

- **Debugging:** Identifying why components re-render unexpectedly.
- **Code reviews:** Catching context anti-patterns.
- **Architecture:** Deciding when to use Context vs props vs state libraries.

### References

- Steve Kinney — Context API Performance Pitfalls — https://stevekinney.com/courses/react-performance/context-api-performance-pitfalls
- Feature-Sliced Design — React's Context API: Friend or Architectural Foe? — https://feature-sliced.design

---

## References

- React — useContext — https://de.react.dev/reference/react/useContext
- React — createContext — https://uk.react.dev/reference/react/createContext
- React — Passing Data Deeply with Context — https://18.react.dev/learn/passing-data-deeply-with-context
- React — Context (Legacy) — https://mintlify.wiki/facebook/react/guides/context
- LogRocket — React Context tutorial: Complete guide with practical examples — https://blog.logrocket.com/react-context-tutorial
- CoreUI — How to use Context API in React — https://coreui.io
- Steve Kinney — Context API Performance Pitfalls — https://stevekinney.com/courses/react-performance/context-api-performance-pitfalls
- Steve Kinney — Separating Actions from State with Two Contexts — https://stevekinney.com
- Steve Kinney — Typing Context and Selector Patterns — https://stevekinney.com
- Feature-Sliced Design — React's Context API: Friend or Architectural Foe? — https://feature-sliced.design
- IEEE — Web Application State Management: A Review of Leading React Frameworks — https://ieeexplore.ieee.org
- GitHub — React Context Composition (Compile-N-Run) — https://github.com/Compile-N-Run/Compile-N-Run
- GitHub — React Context Anti-Patterns (Impertio-Studio) — https://github.com/Impertio-Studio/React-Claude-Skill-Package
- GitHub — Context API Rules (aios-labs) — https://github.com/aios-labs/projectrules
- Vercel — React Stable Context Provider Value — https://vercel-docs.vercel.sh