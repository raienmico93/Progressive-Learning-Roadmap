# React Context API — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Context API is React's built-in mechanism for passing data through the component tree without manually threading props through every intermediate component.

**Technical Definition:** Context provides a way to share values — theme, authenticated user, locale, cart — that are considered “global” for a subtree of React components. A context object is created with `createContext`, supplied to a subtree via a provider, and read by any descendant with `useContext` (or `use` in React 19). When the provider's `value` changes, React re-renders every component that reads that context, comparing previous and next values by reference (`Object.is`). Context is a **broadcast mechanism**, not a state manager: it carries values but does not hold or optimize them.

**Beginner-Friendly Explanation:** Context is React's way of avoiding “prop drilling” — passing the same prop through five components that do not use it just so the sixth can. You put the value in a provider at the top, and any component below can read it directly. The catch is that when the value changes, everyone reading it re-renders, so it works best for data that changes rarely, like a theme or a logged-in user.

### Key Characteristics

- **Prop-drilling alternative:** Eliminates intermediate prop passing for genuinely shared data.
- **Broadcast, not state:** Context does not hold state; it carries a value supplied by a provider.
- **Nearest-provider semantics:** A consumer reads from the closest matching provider above it.
- **Reference-based comparison:** React compares context values by identity; a new object every render re-renders all consumers.
- **No built-in selector:** Every consumer re-renders when the value changes, even if it only uses a small slice.
- **Custom-hook encapsulation:** The recommended pattern is to hide the raw context and expose a hook that throws without a provider.
- **React 19 simplification:** Context can be rendered directly as `<Context value={...}>`, and `use(Context)` can be called conditionally.

### Prerequisites

- React function components, props, and `useState`.
- JavaScript closures and reference equality.
- Basic understanding of component trees.
- Familiarity with `useReducer` (for the reducer + context pattern).

### Related Programming Areas

- **State Management** — Context is one tool among local state, URL state, and server state.
- **Performance** — splitting, memoization, and selector patterns.
- **React 19** — `use()` and the simplified provider syntax.
- **Server Components** — Context works differently across the server/client boundary.

---

## 1. Context API

### Definitions

**Core Definition:** The Context API is React's built-in system for broadcasting a value to a subtree without passing props through every level.

**Technical Definition:** Context solves the problem of sharing data that a wide part of the application needs and that changes rarely — theme, signed-in user, language, cart. One component **provides** the value, and any component below it **reads** it. The component reading gets the value of the **nearest** provider above it; with no provider, it sees the default value given to `createContext`. React automatically re-renders all children that use a particular context starting from the provider that receives a different value, comparing previous and next values with a reference check.

**Beginner-Friendly Explanation:** Context is the built-in way to share a value with many components without passing it through each one. You create a context, wrap part of the tree in a provider, and read it anywhere below.

### Purposes

- To avoid prop drilling for data needed by deeply nested components.
- To share genuinely app-wide values (theme, auth, locale) without a state library.
- To compose providers for different concerns and keep component APIs clean.
- To provide a simple, dependency-free alternative for low-frequency shared state.
- To enable custom hooks that encapsulate context access and enforce provider presence.

### Syntax Rules and Structure

**General Syntax (React 19+):**

```tsx
import { createContext, useContext, useState } from 'react';

const ThemeContext = createContext('light');

function App() {
  const [theme, setTheme] = useState('light');
  return (
    <ThemeContext value={theme}>
      <Page />
    </ThemeContext>
  );
}

function Page() {
  const theme = useContext(ThemeContext);
  return <div className={theme}>…</div>;
}
```

**Component Breakdown:**

| Element | Description |
|---------|-------------|
| `createContext(defaultValue)` | Creates a context object; default is a static “last resort” fallback. |
| `<ThemeContext value={...}>` | React 19 provider syntax; `<ThemeContext.Provider>` is legacy but still works. |
| `useContext(ThemeContext)` | Reads the nearest provider's value; cannot be called conditionally. |
| `use(ThemeContext)` | React 19 alternative; can be called conditionally. |

**Syntax Rules:**

1. Call `createContext` outside any component.
2. Wrap the consuming subtree in a provider; the nearest provider wins.
3. Read with `useContext` (or `use` in React 19).
4. The value can be of any type.
5. React compares values by reference (`Object.is`).

**Constraints and Limitations:**

- Every consumer re-renders when the value changes; there is no built-in selector.
- A new object/function created on every provider render re-renders all consumers.
- Context is not a replacement for server-state caching or frequently changing state.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Basic Theme Context

```tsx
import { createContext, useContext, useState } from 'react';

const ThemeContext = createContext<'light' | 'dark'>('light');

function App() {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');

  return (
    <ThemeContext value={theme}>
      <Toolbar />
      <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
        Toggle theme
      </button>
    </ThemeContext>
  );
}

function Toolbar() {
  return <ThemedButton />;
}

function ThemedButton() {
  const theme = useContext(ThemeContext);
  return <button className={theme}>Themed</button>;
}
```

**Expected Output:** Clicking “Toggle theme” switches the button's class between `light` and `dark`. `Toolbar` does not receive a `theme` prop.

**Why this output:** `ThemedButton` reads the nearest provider's value. Changing `theme` updates the context value, and React re-renders `ThemedButton`. `Toolbar` is not a consumer, so it does not re-render from the context change.

#### Example 2: Custom Hook with Provider Guard

```tsx
import { createContext, useContext, type ReactNode } from 'react';

type CartContextValue = { items: string[]; addItem: (item: string) => void };

const CartContext = createContext<CartContextValue | null>(null);

export function useCart() {
  const cart = useContext(CartContext);
  if (cart === null) {
    throw new Error('useCart must be used within CartProvider.');
  }
  return cart;
}

export function CartProvider({ children }: { children: ReactNode }) {
  const [items, setItems] = useState<string[]>([]);
  const value = useMemo(() => ({
    items,
    addItem: (item: string) => setItems((prev) => [...prev, item]),
  }), [items]);

  return <CartContext value={value}>{children}</CartContext>;
}
```

**Expected Output:** Calling `useCart()` outside a provider throws a clear error. Inside the provider, it returns the cart value.

**Why this output:** The context default is `null`, and the custom hook checks for `null` and throws. The raw context is not exported, forcing consumers through the guarded hook.

### Real-World Cases

- **Theme management:** Light/dark mode read by many components.
- **Authentication:** Current user and permissions.
- **Localisation:** Language and locale settings.
- **Shopping cart:** Shared cart state for a subtree.

### References

- React — createContext — https://react.dev/reference/react/createContext
- React — Passing Data Deeply with Context — https://react.dev/learn/passing-data-deeply-with-context
- Context (virastack academy) — https://github.com/virastack/academy/blob/6a28815094da2825f46e17325d6dcc85902dbe6c/content/en/react/context.md
- Get to Know React's New Context API — https://www.w3reference.com/blog/get-to-know-react-s-new-context-api/

---

## 2. `React.createContext`

### Definitions

**Core Definition:** `createContext` is the React function that creates a context object components can provide or read.

**Technical Definition:** `createContext(defaultValue)` is called outside any component. It returns a context object that represents **which** context other components read or provide. The object itself holds no information. The `defaultValue` is used only when no matching provider exists above the reader; it is a static “last resort” fallback and never changes over time. If there is no meaningful default, pass `null`.

**Beginner-Friendly Explanation:** `createContext` creates the context itself. You give it a fallback value, and it gives you back an object you will later use to provide and read values.

### Purposes

- To create a context object for a specific concern.
- To define a static fallback for components used without a provider.
- To enable type-safe context with TypeScript generics.
- To separate concerns by creating one context per domain.

### Syntax Rules and Structure

```tsx
import { createContext } from 'react';

const ThemeContext = createContext('light');
const AuthContext = createContext<User | null>(null);
```

**Syntax Rules:**

1. Call `createContext` at module scope, never inside a component.
2. Pass `null` when there is no meaningful default value.
3. The default value is static and never changes.
4. Use a TypeScript generic to type the context value.

**Constraints and Limitations:**

- The default value is not a substitute for a provider; it is a fallback.
- A context object does not hold state.

### Annotated Code Examples

```tsx
import { createContext } from 'react';

type Theme = 'light' | 'dark';
const ThemeContext = createContext<Theme>('light');

// No meaningful default
type AuthContextValue = { user: { name: string } | null; login: () => void };
const AuthContext = createContext<AuthContextValue | null>(null);
```

**Expected Output:** `ThemeContext` has a meaningful fallback; `AuthContext` defaults to `null` so a custom hook can throw without a provider.

**Why this output:** `createContext` stores the default for the no-provider case only. The `null` default forces explicit provider presence when paired with a guard hook.

### Real-World Cases

- **Theme context:** `createContext<'light' | 'dark'>('light')`.
- **Auth context:** `createContext<Auth | null>(null)`.
- **Locale context:** `createContext('en')`.

### References

- React — createContext: Parameters — https://react.dev/reference/react/createContext#parameters
- React — createContext (React 18) — https://18.react.dev/reference/react/createContext

---

## 3. Context Provider

### Definitions

**Core Definition:** A context provider is the component that supplies a context value to every component inside it, no matter how deep.

**Technical Definition:** The provider is written as `<SomeContext value={...}>` in React 19 (and `<SomeContext.Provider value={...}>` in React 18 and earlier). It accepts a `value` prop of any type. A component calling `useContext(SomeContext)` inside the provider receives the value of the **innermost** corresponding provider above it. When the provider's `value` changes, React re-renders every consumer below that provider.

**Beginner-Friendly Explanation:** The provider is where you put the value in. Everything wrapped inside it can read that value.

### Purposes

- To supply a context value to a subtree.
- To scope a value to a part of the application.
- To override an outer provider's value for a nested subtree.
- To compose multiple providers for different concerns.

### Syntax Rules and Structure

**React 19+:**

```tsx
<ThemeContext value={theme}>
  <Page />
</ThemeContext>
```

**React 18 and earlier:**

```tsx
<ThemeContext.Provider value={theme}>
  <Page />
</ThemeContext.Provider>
```

**Syntax Rules:**

1. React 19 renders `<Context>` directly as the provider.
2. `<Context.Provider>` still works but is legacy.
3. The `value` can be any type.
4. The innermost matching provider wins.
5. Changing the `value` reference re-renders all consumers.

**Constraints and Limitations:**

- A new object or function created on every provider render causes all consumers to re-render.
- Deeply nested providers of the same context create “provider hell” if not composed.

### Annotated Code Examples

```tsx
function App() {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');

  return (
    <ThemeContext value={theme}>
      <Toolbar />
      <button onClick={() => setTheme(t => t === 'light' ? 'dark' : 'light')}>
        Toggle
      </button>
    </ThemeContext>
  );
}
```

**Expected Output:** All consumers inside the provider read the current theme; clicking Toggle updates them.

**Why this output:** The provider supplies `theme`. When `theme` changes, React re-renders every component reading `ThemeContext`.

### Real-World Cases

- **App-wide theme:** Wrapping the root layout.
- **Authenticated area:** Wrapping only the authenticated subtree.
- **Multi-tenant apps:** Providing a tenant context at a route boundary.

### References

- React — createContext: Provider — https://react.dev/reference/react/createContext#provider
- Context (virastack) — https://github.com/virastack/academy/blob/6a28815094da2825f46e17325d6dcc85902dbe6c/content/en/react/context.md

---

## 4. Context Consumer

### Definitions

**Core Definition:** A context consumer is the component or hook that reads a context value from the nearest provider above it.

**Technical Definition:** The modern way to consume context is `useContext(SomeContext)` (or `use(SomeContext)` in React 19). The legacy `<SomeContext.Consumer>` render-prop component still works but is rarely used and not recommended for new code. The consumer receives the value of the nearest provider above it, or the default value if no provider exists.

**Beginner-Friendly Explanation:** The consumer is how you read the value. Today you use `useContext`; the old `<Consumer>` render prop is legacy.

### Purposes

- To read a context value in a component.
- To subscribe the component to context changes.
- To access the nearest provider's value.
- To replace legacy render-prop consumers with a hook.

### Syntax Rules and Structure

**Modern:**

```tsx
const theme = useContext(ThemeContext);
```

**Legacy (not recommended):**

```tsx
<ThemeContext.Consumer>
  {theme => <button className={theme} />}
</ThemeContext.Consumer>
```

**Syntax Rules:**

1. `useContext` is the recommended consumer.
2. `use` can also read context in React 19 and can be called conditionally.
3. The legacy `Consumer` uses a render prop and re-runs when the value changes.
4. Only components that call `useContext` re-render on context change.

**Constraints and Limitations:**

- `useContext` cannot be called inside conditions or loops.
- The legacy `Consumer` cannot be used with hooks.

### Annotated Code Examples

```tsx
function ThemedButton() {
  const theme = useContext(ThemeContext);
  return <button className={theme}>Click</button>;
}
```

**Expected Output:** The button's class reflects the nearest provider's theme; it re-renders only when the theme changes.

**Why this output:** `useContext` subscribes the component to the context. React re-renders it when the provider's value changes.

### Real-World Cases

- **Theme consumers:** Buttons and badges reading the theme.
- **Auth consumers:** Menus and avatars reading the current user.
- **Locale consumers:** Formatters reading the current locale.

### References

- React — createContext: Consumer — https://react.dev/reference/react/createContext#consumer
- React — useContext — https://react.dev/reference/react/useContext

---

## 5. `useContext`

### Definitions

**Core Definition:** `useContext` is the React Hook that reads and subscribes to a context value from the nearest provider above the calling component.

**Technical Definition:** `const value = useContext(SomeContext)` returns the current context value. React searches upward from the calling component and returns the value of the innermost matching provider; if none exists, it returns the default value from `createContext`. React automatically re-renders the component when the context value changes. `useContext` must be called at the top level of a component or custom Hook and cannot be called conditionally.

**Beginner-Friendly Explanation:** `useContext` is how a component reads a context. You pass it the context object, and it gives you the current value.

### Purposes

- To read a context value without prop drilling.
- To subscribe the component to context changes.
- To access the nearest provider's value.
- To encapsulate context access in a custom hook.

### Syntax Rules and Structure

```tsx
const value = useContext(SomeContext);
```

**Syntax Rules:**

1. Call at the top level of a component or custom Hook.
2. Cannot be called inside conditions or loops.
3. React compares context values by reference.
4. The nearest provider wins.
5. In React 19, `use(Context)` is an alternative that **can** be called conditionally.

**Constraints and Limitations:**

- No selector support: every consumer re-renders when the value changes.
- Cannot read a provider in the same component that renders it.

### Annotated Code Examples

```tsx
function UserName() {
  const { user } = useContext(AuthContext);
  return <span>{user?.name ?? 'Guest'}</span>;
}
```

**Expected Output:** The user's name renders from the nearest auth provider, or “Guest” if there is no user.

**Why this output:** `useContext` reads the auth value. If the user changes, the component re-renders.

### Real-World Cases

- **Auth-aware components:** Reading the current user.
- **Theme-aware components:** Reading the theme.
- **Locale-aware components:** Reading the locale.

### References

- React — useContext — https://react.dev/reference/react/useContext
- React 19 Context improvements — https://github.com/fusengine/agents/blob/main/plugins/react-expert/skills/react-19/references/context-improvements.md

---

## 6. Context Value

### Definitions

**Core Definition:** The context value is the data supplied by the provider and read by consumers.

**Technical Definition:** The `value` prop can be of any type: a primitive, object, array, function, or JSX. React compares the previous and next values with `Object.is`. If the value is a new object or function reference on every render, all consumers re-render even if the contents are unchanged. Values should therefore be stabilised with `useMemo` (objects) or `useCallback` (functions) when they are not primitives.

**Beginner-Friendly Explanation:** The context value is what you put into the provider and what consumers read. It can be anything, but if you create a new object every time, consumers will re-render unnecessarily.

### Purposes

- To supply data to consumers.
- To share state and actions together.
- To enable theming and configuration.
- To support functions as context values (e.g., `dispatch`, `login`).

### Syntax Rules and Structure

```tsx
<SomeContext value={{ user, theme }}>
  {children}
</SomeContext>
```

**Syntax Rules:**

1. The value can be any type.
2. React compares by reference (`Object.is`).
3. Wrap object values in `useMemo` and function values in `useCallback`.
4. Split state and actions into separate contexts when performance matters.

**Constraints and Limitations:**

- A new object/function reference on every render re-renders all consumers.
- Large context values make selective consumption harder.

### Annotated Code Examples

```tsx
const value = useMemo(() => ({ user, setUser }), [user]);
return <UserContext value={value}>{children}</UserContext>;
```

**Expected Output:** The context value only changes when `user` changes; consumers do not re-render on unrelated parent renders.

**Why this output:** `useMemo` keeps the object reference stable until `user` changes.

### Real-World Cases

- **Auth value:** `{ user, login, logout }`.
- **Theme value:** `{ theme, setTheme }`.
- **Dispatch value:** `dispatch` from `useReducer` (stable reference).

### References

- React — Passing Data Deeply with Context — https://react.dev/learn/passing-data-deeply-with-context
- Vercel — REACT_STABLE_CONTEXT_PROVIDER_VALUE — https://vercel.com/docs/conformance/rules/react_stable_context_provider_value
- rslint — jsx-no-constructed-context-values — https://github.com/web-infra-dev/rslint/blob/bca18898f54c441205d6384ab02a7d3cf7982631/internal/plugins/react/rules/jsx_no_constructed_context_values/jsx_no_constructed_context_values.md

---

## 7. Context Default Value

### Definitions

**Core Definition:** The context default value is the fallback returned by `useContext` when there is no matching provider above the reading component.

**Technical Definition:** The `defaultValue` passed to `createContext` is used only when a component reads the context but no matching provider exists above it. It is a static “last resort” fallback and never changes. If there is no meaningful default, pass `null`. A common pattern is to default to `null`, keep the context unexported, and expose a custom hook that throws when the context is `null`.

**Beginner-Friendly Explanation:** The default is what you get if you forgot to wrap the component in a provider. It is a safety net, not the normal way to supply data.

### Purposes

- To provide a fallback when no provider exists.
- To avoid crashes when a component is used standalone.
- To force explicit provider presence by defaulting to `null` and throwing.
- To document the expected shape of the value.

### Syntax Rules and Structure

```tsx
const ThemeContext = createContext('light');
const AuthContext = createContext<Auth | null>(null);
```

**Syntax Rules:**

1. Use a meaningful default only when standalone use makes sense.
2. Use `null` when no meaningful default exists.
3. The default is static and never changes.
4. Pair a `null` default with a guard hook that throws.

**Constraints and Limitations:**

- The default is not a substitute for a provider.
- A non-null default can hide a missing provider and cause subtle bugs.

### Annotated Code Examples

```tsx
const CartContext = createContext<CartContextValue | null>(null);

export function useCart() {
  const cart = useContext(CartContext);
  if (cart === null) {
    throw new Error('useCart must be used within CartProvider.');
  }
  return cart;
}
```

**Expected Output:** Using `useCart` outside the provider throws a clear error instead of silently receiving a default value.

**Why this output:** The default is `null`; the guard hook converts a missing provider into an explicit error.

### Real-World Cases

- **Auth context:** `null` default with a guard hook.
- **Theme context:** `'light'` default for standalone components.
- **Locale context:** `'en'` default.

### References

- React — createContext: defaultValue — https://react.dev/reference/react/createContext#parameters
- Context (virastack): custom hook pattern — https://github.com/virastack/academy/blob/6a28815094da2825f46e17325d6dcc85902dbe6c/content/en/react/context.md#1

---

## 8. Context Nesting

### Definitions

**Core Definition:** Context nesting is placing multiple providers inside one another to supply different contexts to different subtrees.

**Technical Definition:** React allows providers to be nested. Each provider supplies its own context value to the components below it. When providers of the **same** context are nested, the innermost provider's value overrides the outer one for its subtree. Nesting different contexts allows independent data to be shared at different levels. Deep nesting of the same context creates “provider hell” and should be avoided.

**Beginner-Friendly Explanation:** You can put providers inside other providers. Each one supplies its own data, and the closest provider wins.

### Purposes

- To scope multiple contexts to a subtree.
- To override a context value for a nested region.
- To compose independent providers for different concerns.
- To avoid one giant provider.

### Syntax Rules and Structure

```tsx
<ThemeContext value="dark">
  <AuthContext value={user}>
    <CartContext value={cart}>
      {children}
    </CartContext>
  </AuthContext>
</ThemeContext>
```

**Syntax Rules:**

1. Different contexts can be nested freely.
2. The innermost provider of the same context wins.
3. Compose providers into a single `AppProviders` component to reduce nesting in the app tree.
4. Avoid deeply nested same-context providers unless intentionally overriding.

**Constraints and Limitations:**

- Deep nesting creates “provider hell”.
- The innermost same-context provider completely shadows the outer one.

### Annotated Code Examples

```tsx
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
```

**Expected Output:** All three contexts are available to `children`; the app tree stays clean.

**Why this output:** The providers are composed into one wrapper. Nesting different contexts is fine; the wrapper hides the boilerplate.

### Real-World Cases

- **Application root:** Theme, auth, and locale providers.
- **Route-level scoping:** A tenant provider around a route.
- **Testing:** Overriding a context value for a specific test.

### References

- React — Passing Data Deeply with Context — https://react.dev/learn/passing-data-deeply-with-context
- Context (virastack) — https://github.com/virastack/academy/blob/6a28815094da2825f46e17325d6dcc85902dbe6c/content/en/react/context.md

---

## 9. Context Composition

### Definitions

**Core Definition:** Context composition is the practice of combining multiple focused contexts and providers to model different concerns, rather than using one monolithic context.

**Technical Definition:** Composition means creating one context per concern (theme, auth, cart), nesting their providers, and letting each consumer read only the context it needs. It is the structural counterpart to context splitting: splitting is about **not** bundling unrelated values into one context; composition is about **how** multiple focused contexts are arranged and consumed together. A component that needs several contexts simply calls several hooks, or uses a `use` call for each.

**Beginner-Friendly Explanation:** Composition means using several small contexts together instead of one giant one. Each component reads only the contexts it actually needs.

### Purposes

- To keep contexts narrow and independent.
- To prevent one concern's updates from re-rendering consumers of another concern.
- To compose providers at the application root cleanly.
- To allow components to opt into only the contexts they need.

### Syntax Rules and Structure

```tsx
function AppProviders({ children }) {
  return (
    <ThemeProvider>
      <AuthProvider>
        <CartProvider>{children}</CartProvider>
      </AuthProvider>
    </ThemeProvider>
  );
}

function Header() {
  const theme = useTheme();
  const { user } = useAuth();
  return <header className={theme}>{user?.name}</header>;
}
```

**Syntax Rules:**

1. One context per independently changing concern.
2. Compose providers in a single wrapper.
3. Consumers call only the hooks they need.
4. Do not bundle unrelated values into one context “for convenience”.

**Constraints and Limitations:**

- More contexts mean more provider components; compose them to keep the tree readable.
- A component that needs many contexts becomes a heavy consumer; consider whether it should be split.

### Annotated Code Examples

```tsx
function AppProviders({ children }: { children: React.ReactNode }) {
  return (
    <ThemeProvider>
      <AuthProvider>
        <NotificationProvider>{children}</NotificationProvider>
      </AuthProvider>
    </ThemeProvider>
  );
}
```

**Expected Output:** A theme consumer re-renders only when the theme changes, not when notifications change.

**Why this output:** Each provider has its own value and its own consumer set. Composition keeps the concerns independent.

### Real-World Cases

- **SaaS layout:** Theme, auth, and notification providers composed at the root.
- **E-commerce:** Cart, currency, and auth providers.
- **Multi-tenant apps:** Tenant, theme, and auth providers.

### References

- React — Passing Data Deeply with Context — https://react.dev/learn/passing-data-deeply-with-context
- pproenca — Context optimization: split by concern — https://github.com/pproenca/dot-skills/blob/9c015c68df33adf461ea9fa7071c119ca0061a0a/skills/.curated/react/references/rstate-context-optimization.md#1

---

## 10. Context Performance

### Definitions

**Core Definition:** Context performance is the re-render cost incurred when a provider's value changes and every consumer re-renders.

**Technical Definition:** Context is a **broadcast mechanism**, not an optimization. When a provider's `value` changes, React re-renders every component that reads that context, even if the component only uses a small, unchanged slice. React compares the previous and next values by reference (`Object.is`); a new object or function reference on every render causes all consumers to re-render unconditionally. Memoizing a consumer with `React.memo` does **not** prevent it from re-rendering when its context value changes. Context is therefore best for values that change rarely; frequently changing state should use a store with selectors.

**Beginner-Friendly Explanation:** Context does not make rendering faster — it makes prop drilling easier. When the value changes, everyone reading it re-renders. If the value changes often, the whole subtree becomes slow.

### Purposes

- To understand why context changes cause re-renders.
- To choose the right data for context (low-frequency values).
- To identify re-render cascades before they become production problems.
- To justify splitting and memoization when context is the right tool.

### Syntax Rules and Structure

**Performance characteristics:**

| Characteristic | Behaviour |
|----------------|-----------|
| Value change | All consumers re-render |
| Reference comparison | `Object.is` on the value |
| Selector support | None built-in |
| `React.memo` on consumer | Does **not** block context updates |
| Best data | Low-frequency values (theme, auth, locale) |
| Poor data | High-frequency values (form input, mouse position) |

**Syntax Rules:**

1. Put low-frequency values in context.
2. Do not put per-keystroke state in context; use local state or a store.
3. Split contexts by update frequency.
4. Memoize the value and actions to avoid unnecessary changes.

**Constraints and Limitations:**

- Context has no built-in selector; every consumer receives the whole value.
- `React.memo` cannot prevent a consumer from re-rendering when its context changes.

### Annotated Code Examples

```tsx
// ❌ High-frequency context: every mouse move re-renders every consumer
const MouseContext = createContext({ x: 0, y: 0 });

function MouseProvider({ children }) {
  const [pos, setPos] = useState({ x: 0, y: 0 });
  useEffect(() => {
    const onMove = (e: MouseEvent) => setPos({ x: e.clientX, y: e.clientY });
    window.addEventListener('mousemove', onMove);
    return () => window.removeEventListener('mousemove', onMove);
  }, []);
  return <MouseContext value={pos}>{children}</MouseContext>;
}
```

**Expected Output:** Every mouse movement re-renders every consumer of `MouseContext`.

**Why this output:** The value changes on every mousemove; every consumer re-renders. This is a poor fit for context.

### Real-World Cases

- **Theme:** Low-frequency; good fit.
- **Auth:** Low-frequency; good fit.
- **Mouse position:** High-frequency; use a ref, local state, or a store.
- **Form input:** High-frequency; use local/form state.

### References

- Steve Kinney — Context API Performance Pitfalls — https://stevekinney.com/courses/react-performance/context-api-performance-pitfalls
- Sharp Edges — Context render cascade — https://github.com/omer-metin/skills-for-antigravity/blob/e8dcf4e8737921a10088bd5c9eb65e81f74c051f/skills/state-management/references/sharp_edges.md
- React — Passing Data Deeply with Context — https://react.dev/learn/passing-data-deeply-with-context

---

## 11. Context Splitting

### Definitions

**Core Definition:** Context splitting is the practice of dividing one large context into multiple focused contexts, each holding one independently changing piece of state.

**Technical Definition:** A context whose value bundles many independent fields (`{ user, theme, notifications }`) re-renders **all** consumers whenever **any** field changes. Splitting means creating one context per concern (`UserContext`, `ThemeContext`, `NotificationContext`) and nesting their providers. Each consumer then reads only the context it needs and re-renders only when that specific context changes. Splitting is the primary structural fix for context re-render cascades when the data genuinely belongs in context.

**Beginner-Friendly Explanation:** Splitting means using several small contexts instead of one big one. A theme consumer then re-renders only when the theme changes, not when the user or notifications change.

### Purposes

- To prevent unrelated context updates from re-rendering consumers.
- To let consumers subscribe to only the slice they need.
- To make context values narrow and independently changeable.
- To improve performance without abandoning context.

### Syntax Rules and Structure

**❌ Fat context:**

```tsx
const AppContext = createContext({ user: null, theme: 'light', notifications: [] });
```

**✅ Split contexts:**

```tsx
const UserContext = createContext<User | null>(null);
const ThemeContext = createContext<'light' | 'dark'>('light');
const NotificationContext = createContext<Notification[]>([]);
```

**Syntax Rules:**

1. One context per independently changing concern.
2. Nest the providers; consumers read the context they need.
3. Split **runtime** providers, not just TypeScript interfaces; a type-only split provides no re-render benefit.
4. Split by update frequency: rarely changing (theme, auth) vs frequently changing (cart, notifications).

**Constraints and Limitations:**

- More contexts mean more providers; compose them.
- Splitting is not a substitute for selectors when a single context genuinely bundles a large, frequently changing object.

### Annotated Code Examples

```tsx
function AppProviders({ children }: { children: React.ReactNode }) {
  return (
    <UserContext value={user}>
      <ThemeContext value={theme}>
        <NotificationContext value={notifications}>
          {children}
        </NotificationContext>
      </ThemeContext>
    </UserContext>
  );
}

function ThemeButton() {
  const theme = useContext(ThemeContext); // Only re-renders on theme change
  return <button className={theme}>Toggle</button>;
}
```

**Expected Output:** `ThemeButton` does not re-render when the user or notifications change.

**Why this output:** `ThemeButton` reads only `ThemeContext`. The other providers' changes do not affect it.

### Real-World Cases

- **App shell:** User, theme, and notifications split into separate contexts.
- **E-commerce:** Cart and auth split.
- **Dashboards:** Filters and user preferences split.

### References

- pproenca — Context optimization: split fat contexts — https://github.com/pproenca/dot-skills/blob/9c015c68df33adf461ea9fa7071c119ca0061a0a/skills/.curated/react/references/rstate-context-optimization.md#1
- Steve Kinney — Separating Actions from State with Two Contexts — https://stevekinney.com/courses/react-performance/separating-actions-from-state-with-two-contexts
- FreeCodeCamp — Shared State Complexity in React — https://www.freecodecamp.org/news/shared-state-complexity-in-react-handbook/

---

## 12. Context Memoization

### Definitions

**Core Definition:** Context memoization is stabilising the context value with `useMemo` (objects) or `useCallback` (functions) so consumers do not re-render on every provider render.

**Technical Definition:** React compares context values by reference. A provider that creates `value={{ user, theme }}` inline creates a new object on every render, causing all consumers to re-render even when the data is unchanged. Memoising the value with `useMemo` keyed on the actual changing parts keeps the reference stable. Similarly, functions passed in the value should be wrapped in `useCallback`. Memoising the **consumer** with `React.memo` is not sufficient: a context update still reaches it.

**Beginner-Friendly Explanation:** Memoization means keeping the context value the same object between renders when its contents have not changed, so consumers do not re-render for no reason.

### Purposes

- To prevent consumers from re-rendering when the value's contents are unchanged.
- To keep object and function references stable.
- To complement splitting when a context must carry a composite value.
- To avoid unnecessary re-renders from parent renders.

### Syntax Rules and Structure

```tsx
const value = useMemo(() => ({ user, setUser }), [user]);
const login = useCallback(async (credentials) => { /* ... */ }, []);
```

**Syntax Rules:**

1. Wrap object values in `useMemo`.
2. Wrap function values in `useCallback`.
3. Include the actual changing parts in the dependency array.
4. Do not rely on `React.memo` alone to block context updates.
5. Prefer splitting over memoising a very large, frequently changing object.

**Constraints and Limitations:**

- `useMemo` is a performance hint, not a semantic guarantee.
- Memoising a value that changes every render provides no benefit.

### Annotated Code Examples

```tsx
function CartProvider({ children }: { children: React.ReactNode }) {
  const [items, setItems] = useState<CartItem[]>([]);

  const addItem = useCallback((item: CartItem) => {
    setItems((prev) => [...prev, item]);
  }, []);

  const value = useMemo(() => ({ items, addItem }), [items, addItem]);

  return <CartContext value={value}>{children}</CartContext>;
}
```

**Expected Output:** Consumers re-render only when `items` changes, not on every `CartProvider` render.

**Why this output:** `useCallback` stabilises `addItem`; `useMemo` stabilises the value object until `items` changes.

### Real-World Cases

- **Auth providers:** Stable `login` and `logout` callbacks.
- **Cart providers:** Stable `addItem` and `removeItem`.
- **Theme providers:** Stable `toggleTheme`.

### References

- Vercel — REACT_STABLE_CONTEXT_PROVIDER_VALUE — https://vercel.com/docs/conformance/rules/react_stable_context_provider_value
- rslint — jsx-no-constructed-context-values — https://github.com/web-infra-dev/rslint/blob/bca18898f54c441205d6384ab02a7d3cf7982631/internal/plugins/react/rules/jsx_no_constructed_context_values/jsx_no_constructed_context_values.md
- FreeCodeCamp — Shared State Complexity in React — https://www.freecodecamp.org/news/shared-state-complexity-in-react-handbook/

---

## 13. Context Best Practices

### Definitions

**Core Definition:** Context best practices are the guidelines for using context safely, performantly, and maintainably in React applications.

**Technical Definition:** Best practices combine **provider guards** (custom hooks that throw when context is missing), **splitting by concern**, **memoising values**, **avoiding high-frequency state**, **composing providers**, **keeping the raw context private**, and **migrating to React 19 syntax** where appropriate. The overarching rule is: context is for low-frequency, genuinely shared data; local state, URL state, and server-state libraries handle their categories better.

### Best Practices Table

| Practice | Description |
|----------|-------------|
| **Custom hook with guard** | Default to `null`; throw if no provider; export the hook, not the raw context |
| **Split by concern** | One context per independently changing value |
| **Memoise the value** | `useMemo` for objects, `useCallback` for functions |
| **Avoid high-frequency state** | Do not put form input or mouse position in context |
| **Compose providers** | Nest providers in a single `AppProviders` wrapper |
| **Keep context narrow** | Do not bundle unrelated values |
| **Use React 19 syntax** | `<Context value={...}>` and `use(Context)` where appropriate |
| **Lift before context** | Try props and `children` first; use context only when needed |
| **Do not duplicate server state** | Use a query cache, not context, for server data |
| **Test consumers** | Verify guard errors and re-render boundaries |

### Annotated Code Examples

**✅ Good Practices:**

```tsx
const AuthContext = createContext<AuthContextValue | null>(null);

export function useAuth() {
  const auth = useContext(AuthContext);
  if (auth === null) throw new Error('useAuth must be used within AuthProvider');
  return auth;
}

export function AuthProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<User | null>(null);
  const login = useCallback(async (credentials: Credentials) => {
    setUser(await api.login(credentials));
  }, []);
  const value = useMemo(() => ({ user, login }), [user, login]);
  return <AuthContext value={value}>{children}</AuthContext>;
}
```

**❌ Bad Practices:**

```tsx
// ❌ Fat context with high-frequency state
const AppContext = createContext({ user, theme, mouse, form, notifications });

// ❌ Unmemoized value
<AppContext value={{ user, theme }}>

// ❌ Raw context exported without a guard
export const AuthContext = createContext(null);

// ❌ Provider without memoising callbacks
<CartContext value={{ items, addItem: (i) => setItems([...items, i]) }}>
```

**Expected Output:** The good practices produce guarded, narrow, stable contexts. The bad practices cause re-render cascades, silent null values, and unnecessary renders.

**Why this output:** Guards catch missing providers; splitting isolates updates; memoisation stabilises references; high-frequency state belongs elsewhere.

### Real-World Cases

- **All context-based applications:** These practices apply universally.
- **Design systems:** Theme and auth providers with guards.
- **SaaS apps:** Composed providers at the root.

### References

- Context (virastack): custom hook pattern — https://github.com/virastack/academy/blob/6a28815094da2825f46e17325d6dcc85902dbe6c/content/en/react/context.md#1
- Steve Kinney — Context API Performance Pitfalls — https://stevekinney.com/courses/react-performance/context-api-performance-pitfalls
- React 19 Context improvements — https://github.com/fusengine/agents/blob/main/plugins/react-expert/skills/react-19/references/context-improvements.md

---

## 14. Context Pitfalls

### Definitions

**Core Definition:** Context pitfalls are the common mistakes that cause unnecessary re-renders, silent failures, and maintenance problems when using context.

**Technical Definition:** The canonical pitfalls are: **the mega context** (everything in one context), **not splitting state from actions**, **ignoring memo boundaries**, **misunderstanding when context triggers re-renders**, **unmemoised value objects**, **missing provider guards**, **using context for high-frequency state**, and **exporting the raw context**. Each pitfall has a specific remedy.

### Common Pitfalls Table

| Pitfall | Problem | Solution |
|---------|---------|----------|
| **Mega context** | Any change re-renders every consumer | Split by concern |
| **State and actions together** | Action-only consumers re-render on state change | Two contexts: state and dispatch |
| **Unmemoised value** | New object every render → all consumers re-render | `useMemo` / `useCallback` |
| **`React.memo` on consumer** | Does not block context updates | Split or memoise the value |
| **High-frequency state** | Re-render cascades on every keystroke/move | Local state or a store |
| **No provider guard** | Silent `null` or default value | `null` default + throwing hook |
| **Raw context exported** | Consumers bypass the guard | Export the hook, not the context |
| **Provider inside the consumer** | `useContext` cannot see a provider in the same component | Move the provider above the consumer |
| **Duplicated server state** | Context copied from a query cache goes stale | Use a query cache, not context |

### Annotated Code Examples

**Pitfall 1: Mega context**

```tsx
// ❌ Theme consumer re-renders when user or notifications change
const AppContext = createContext({ user, theme, notifications });
```

**Pitfall 2: Unmemoised value**

```tsx
// ❌ New object every render → all consumers re-render
<AppContext value={{ user, theme }} />
```

**Pitfall 3: No guard**

```tsx
// ❌ Silent null or default value
const AuthContext = createContext(null);
```

**Expected Output:** The correct versions split, memoise, and guard; the incorrect versions re-render unnecessarily or fail silently.

**Why this output:** Mega contexts and unmemoised values cause cascading re-renders; missing guards hide provider mistakes.

### Real-World Cases

- **Large apps:** Mega contexts are the most common source of “the app feels slow”.
- **Design systems:** Unmemoised values cause every themed component to re-render.
- **Auth:** Missing guards cause silent null dereferences.

### References

- Steve Kinney — Context API Performance Pitfalls — https://stevekinney.com/courses/react-performance/context-api-performance-pitfalls
- Sharp Edges — Context render cascade — https://github.com/omer-metin/skills-for-antigravity/blob/e8dcf4e8737921a10088bd5c9eb65e81f74c051f/skills/state-management/references/sharp_edges.md
- FreeCodeCamp — Shared State Complexity in React — https://www.freecodecamp.org/news/shared-state-complexity-in-react-handbook/

---

## References

- React — createContext — https://react.dev/reference/react/createContext
- React — useContext — https://react.dev/reference/react/useContext
- React — Passing Data Deeply with Context — https://react.dev/learn/passing-data-deeply-with-context
- Context (virastack academy) — https://github.com/virastack/academy/blob/6a28815094da2825f46e17325d6dcc85902dbe6c/content/en/react/context.md
- Context (virastack academy, alternate) — https://github.com/virastack/academy/blob/4e06ca19c061fe7ceb7fd1e93ae7e872f2a2098d/content/en/react/context.md
- Steve Kinney — Context API Performance Pitfalls — https://stevekinney.com/courses/react-performance/context-api-performance-pitfalls
- Steve Kinney — Separating Actions from State with Two Contexts — https://stevekinney.com/courses/react-performance/separating-actions-from-state-with-two-contexts
- pproenca/dot-skills — Context optimization: split fat contexts — https://github.com/pproenca/dot-skills/blob/9c015c68df33adf461ea9fa7071c119ca0061a0a/skills/.curated/react/references/rstate-context-optimization.md
- Sharp Edges — State Management — https://github.com/omer-metin/skills-for-antigravity/blob/e8dcf4e8737921a10088bd5c9eb65e81f74c051f/skills/state-management/references/sharp_edges.md
- Vercel — REACT_STABLE_CONTEXT_PROVIDER_VALUE — https://vercel.com/docs/conformance/rules/react_stable_context_provider_value
- rslint — jsx-no-constructed-context-values — https://github.com/web-infra-dev/rslint/blob/bca18898f54c441205d6384ab02a7d3cf7982631/internal/plugins/react/rules/jsx_no_constructed_context_values/jsx_no_constructed_context_values.md
- FreeCodeCamp — Shared State Complexity in React — https://www.freecodecamp.org/news/shared-state-complexity-in-react-handbook/
- Get to Know React's New Context API — https://www.w3reference.com/blog/get-to-know-react-s-new-context-api/
- React 19 Context improvements (fusengine) — https://github.com/fusengine/agents/blob/main/plugins/react-expert/skills/react-19/references/context-improvements.md
- React 19 context changes (botharness) — https://github.com/botharness/deepseekbot/blob/fce475b7ba199d0444a540c5014d6143fe9d62d5/.agents/skills/expo-upgrade/references/react-19.md
- Context Selector pattern — https://raw.githubusercontent.com/mjmirza/patterns/refs/heads/main/patterns/13-frontend-ui/context-selector.md