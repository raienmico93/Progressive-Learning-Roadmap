# useContext — Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**

`useContext` is a React Hook that lets a component read and subscribe to a context value provided by the nearest matching provider above it in the component tree.

**Technical Definition**

`useContext(SomeContext)` accepts a context object created by `createContext` and returns the current context value. React determines the value by searching up the component tree for the closest matching provider; if no provider is found, the default value passed to `createContext` is returned. The returned value is always the latest value. When the context value changes, React automatically re-renders all components that read that context .

**Beginner-Friendly Explanation**

`useContext` is like a walkie-talkie that lets a component listen to a broadcast from a parent component far up the tree. Instead of passing information down through every intermediate component, you can just "tune in" to the context and receive the value directly.

### Key Characteristics

- **Avoids Prop Drilling**: Context lets a parent make information available to any descendant without passing props through every level .
- **Nearest Provider Wins**: `useContext` always finds the **closest** provider above the component calling it; it does not consider providers in the same component .
- **Default Value Fallback**: If no provider exists above, the default value from `createContext` is returned. This value never changes .
- **Automatic Re-renders**: When the context value changes, all components reading that context re-render .

### Prerequisites

- React Components and JSX
- `useState` and state management concepts
- Understanding of props and component trees

### Related Programming Areas

- **`createContext`**: The function that creates a context object.
- **Context Provider**: The component that supplies the context value.
- **`useReducer`**: Often combined with context for complex global state .
- **Prop Drilling**: The problem context solves.

### Core Concepts / Features

- Context Creation
- Context Provider
- Context Consumer
- Reading Context
- Global Configuration
- Theme State
- Authentication State

---

## 1. Context Creation

### Definitions

**Core Definition**

Context creation is the process of calling `createContext` to create a context object that components can later provide or read.

**Technical Definition**

`createContext(defaultValue)` returns a context object. This object itself does not hold information; it represents *which* context components read or provide. The `defaultValue` is used only when a component reads the context but no matching provider exists above it. The default value is static and never changes .

**Beginner-Friendly Explanation**

Creating a context is like creating a radio channel. You give it a name and a default broadcast, and later components can tune in or broadcast on that channel.

### Purposes

- To create a named context that components can share.
- To define a fallback default value for when no provider is present.
- To enable communication between distant components without prop drilling.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
import { createContext } from 'react';

const SomeContext = createContext(defaultValue);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `createContext` | Function imported from `react`. |
| `defaultValue` | Fallback value when no provider exists. Can be any type; use `null` if no meaningful default . |
| `SomeContext` | The returned context object. |

**Syntax Rules**

1. **Call outside any component**: `createContext` must be called at the module level, not inside a component .
2. **Default value is static**: It never changes. To update context, combine with state and pass a dynamic value to the provider .
3. **Export from a separate file**: Common practice is to declare contexts in a `Contexts.js` file for import elsewhere .

**Constraints and Limitations**

- The context object itself contains no information.
- Default values are only used when no provider exists above the consumer.

### Annotated Complete Code Example

**Example: Creating a Theme Context**

```jsx
// Contexts.js
import { createContext } from 'react';

// Create a context with 'light' as the default theme
export const ThemeContext = createContext('light');
```

```jsx
// AuthContext.js
import { createContext } from 'react';

// Create an auth context with null as default (no user logged in)
export const AuthContext = createContext(null);
```

**Explanation**

The `ThemeContext` will return `'light'` if a component reads it without a provider. The `AuthContext` returns `null` when no user is logged in. Both are created at the module level and exported for use in other files .

### Real-World Cases

**Case: Theme and Authentication Contexts**

An app creates `ThemeContext` and `AuthContext` in separate files. Components across the application import these to read or provide theme and user data.

### References

- React — createContext API Reference - https://react.dev/reference/react/createContext
- React — createContext (Chinese) - https://zh-hans.react.dev/reference/react/createContext

---

## 2. Context Provider

### Definitions

**Core Definition**

A context provider is a component that supplies a value to all components below it in the tree that read the matching context.

**Technical Definition**

In React 19+, you can render `<SomeContext value={...}>` directly as a provider. In older versions, use `<SomeContext.Provider value={...}>`. The `value` prop is passed to all components reading that context within the provider's subtree, regardless of depth. When the `value` changes, React re-renders all consuming components .

**Beginner-Friendly Explanation**

A provider is like a radio tower that broadcasts a signal. Any radio (component) within range (the subtree) can tune in and receive the broadcast.

### Purposes

- To supply a context value to a subtree of components.
- To make dynamic data (state) available throughout the tree.
- To override context values for specific parts of the tree.

### Syntax Rules and Structure

**Complete General Syntax (React 19+)**

```jsx
<SomeContext value={value}>
  {children}
</SomeContext>
```

**Complete General Syntax (Pre-React 19)**

```jsx
<SomeContext.Provider value={value}>
  {children}
</SomeContext.Provider>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `<SomeContext>` | The context object used as a provider (React 19+). |
| `<SomeContext.Provider>` | Legacy provider syntax (pre-React 19). |
| `value` | The value to pass to all consumers below. Can be any type. |

**Syntax Rules**

1. **Any depth**: The provider can be anywhere above the consumer; intermediate components do not need to know about the context .
2. **Nesting overrides**: You can nest providers with different values to override context for part of the tree .
3. **Dynamic values**: Pass state variables as `value` to update context over time .

**Constraints and Limitations**

- The provider must be **above** the consumer in the tree; a provider in the same component as `useContext` will not be seen .
- Creating new objects/functions for `value` on every render causes all consumers to re-render; use `useMemo` and `useCallback` for optimization .

### Annotated Complete Code Example

**Example 1: Basic Provider with State**

```jsx
import { createContext, useContext, useState } from 'react';

const ThemeContext = createContext('light');

export default function MyApp() {
  // State drives the context value
  const [theme, setTheme] = useState('light');

  return (
    // Provide the current theme to the entire tree
    <ThemeContext value={theme}>
      <Form />
      <label>
        <input
          type="checkbox"
          checked={theme === 'dark'}
          onChange={(e) => setTheme(e.target.checked ? 'dark' : 'light')}
        />
        Use dark mode
      </label>
    </ThemeContext>
  );
}

function Form() {
  return (
    <Panel title="Welcome">
      <Button>Sign up</Button>
      <Button>Log in</Button>
    </Panel>
  );
}

function Panel({ title, children }) {
  // Reads the nearest theme from context
  const theme = useContext(ThemeContext);
  const className = 'panel-' + theme;
  return (
    <section className={className}>
      <h1>{title}</h1>
      {children}
    </section>
  );
}

function Button({ children }) {
  // Also reads theme from context
  const theme = useContext(ThemeContext);
  const className = 'button-' + theme;
  return <button className={className}>{children}</button>;
}
```

**Expected Behavior**

Checking "Use dark mode" updates the theme state, which re-renders `Panel` and `Button` with the new theme.

**Why This Occurs**

`MyApp` holds the `theme` state and passes it as the `value` to `ThemeContext`. `Panel` and `Button` call `useContext(ThemeContext)` to read the nearest provided value. When `setTheme` changes the state, the provider receives a new value, and React re-renders all consumers .

**Example 2: Overriding Context for Part of the Tree**

```jsx
function Footer() {
  return (
    <ThemeContext value="light">
      <Button>Settings</Button>
    </ThemeContext>
  );
}
```

**Explanation**

Even if the rest of the app is in dark mode, the `Button` inside `Footer` receives `"light"` because the closest provider above it overrides the outer one .

### Real-World Cases

**Case: Theme Provider**

A `ThemeProvider` component holds theme state and provides it to the entire app, allowing any component to read the current theme without prop drilling.

### References

- React — createContext (Provider) - https://react.dev/reference/react/createContext
- React — useContext (Updating data via context) - https://18.react.dev/reference/react/useContext

---

## 3. Context Consumer

### Definitions

**Core Definition**

A context consumer is a component or code that reads the current value from a context.

**Technical Definition**

The modern way to consume context is the `useContext` Hook, which returns the current context value. The legacy way is `<SomeContext.Consumer>`, which uses a render prop function receiving the context value .

**Beginner-Friendly Explanation**

A consumer is a radio that tunes into the context channel. The `useContext` Hook is the modern, simpler way to tune in.

### Purposes

- To read the current value from a context.
- To subscribe to context changes and re-render when they occur.
- To use context values in component logic and rendering.

### Syntax Rules and Structure

**Complete General Syntax (Modern — Recommended)**

```jsx
import { useContext } from 'react';

const value = useContext(SomeContext);
```

**Complete General Syntax (Legacy — Not Recommended)**

```jsx
<SomeContext.Consumer>
  {value => <div>{value}</div>}
</SomeContext.Consumer>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `useContext(SomeContext)` | Returns the current context value. |
| `<SomeContext.Consumer>` | Legacy render-prop consumer. |

**Syntax Rules**

1. **Call at top level**: `useContext` must be called at the top level of a component, not inside loops or conditions .
2. **Pass the context object**: Pass `SomeContext` itself, not `SomeContext.Consumer` .
3. **Automatic subscription**: React automatically re-renders the component when the context value changes .

**Constraints and Limitations**

- `useContext` only finds providers **above** the calling component, not in the same component .
- The legacy `Consumer` API still works but is not recommended for new code .

### Annotated Complete Code Example

**Example: Reading Context in a Component**

```jsx
import { useContext } from 'react';
import { ThemeContext } from './Contexts.js';

function ThemedButton() {
  // Read the current theme value
  const theme = useContext(ThemeContext);

  return (
    <button className={`btn-${theme}`}>
      I am styled by the {theme} theme!
    </button>
  );
}
```

**Explanation**

`ThemedButton` calls `useContext(ThemeContext)` to read the nearest provided theme value. It uses that value to determine the CSS class. When the theme changes, React re-renders this component .

### Real-World Cases

**Case: Displaying the Current User**

A `Profile` component uses `useContext(AuthContext)` to read the current user object and display the user's name.

### References

- React — useContext API Reference - https://react.dev/reference/react/useContext
- React — useContext (Chinese) - https://zh-hans.react.dev/reference/react/useContext

---

## 4. Reading Context

### Definitions

**Core Definition**

Reading context is the act of calling `useContext` with a context object to retrieve its current value.

**Technical Definition**

`useContext(SomeContext)` searches up the component tree for the nearest matching provider and returns its `value`. If no provider is found, the default value from `createContext` is returned. The returned value is always the latest; React re-renders the component when the context changes .

**Beginner-Friendly Explanation**

Reading context is like looking up at the closest radio tower to see what it's broadcasting. You get the current signal, and if the tower changes its broadcast, you get the new one automatically.

### Purposes

- To access shared data without prop drilling.
- To subscribe to context updates.
- To use global configuration in component logic.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
const value = useContext(SomeContext);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `SomeContext` | The context object created by `createContext`. |
| `value` | The current context value. |

**Syntax Rules**

1. **Top-level call**: Must be called at the top level of a component .
2. **Nearest provider**: React searches upward for the closest provider .
3. **Default fallback**: If no provider, the default value from `createContext` is returned .

**Constraints and Limitations**

- Cannot read providers in the same component .
- Build issues (duplicate modules, symlinks) can cause two "different" context objects, breaking the lookup .

### Annotated Complete Code Example

**Example: Reading Multiple Contexts**

```jsx
import { useContext } from 'react';
import { ThemeContext, AuthContext } from './Contexts.js';

function Dashboard() {
  const theme = useContext(ThemeContext);
  const user = useContext(AuthContext);

  return (
    <div className={`dashboard-${theme}`}>
      <h1>Welcome, {user?.name ?? 'Guest'}</h1>
    </div>
  );
}
```

**Explanation**

`Dashboard` reads two independent contexts. React searches for the nearest provider for each context separately. If either context value changes, the component re-renders .

### Real-World Cases

**Case: Internationalization**

A component reads a `LocaleContext` to display text in the user's chosen language.

### References

- React — useContext (Reading context) - https://18.react.dev/reference/react/useContext
- React — Passing Data Deeply with Context - https://18.react.dev/learn/passing-data-deeply-with-context

---

## 5. Global Configuration

### Definitions

**Core Definition**

Global configuration is application-wide settings (API endpoints, feature flags, locale) shared through context.

**Technical Definition**

A configuration context holds an object with global settings and is provided near the root of the application. Components read individual settings via `useContext`, avoiding prop drilling and ensuring consistency .

**Beginner-Friendly Explanation**

Global configuration is like a settings panel that every part of the app can read. Instead of telling each component the API URL separately, you put it in one place and let them all read it.

### Purposes

- To share application-wide settings without prop drilling.
- To ensure all components use the same configuration.
- To allow runtime configuration changes.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
// ConfigContext.js
import { createContext } from 'react';
export const ConfigContext = createContext({});

// App.jsx
<ConfigContext value={{ apiUrl: '/api', featureFlags: {...} }}>
  <App />
</ConfigContext>
```

**Syntax Rules**

1. **Provide near root**: Configuration should be provided high in the tree.
2. **Use `useMemo`**: Wrap the config object to avoid unnecessary re-renders .

### Annotated Complete Code Example

**Example: API Configuration Context**

```jsx
import { createContext, useContext, useMemo } from 'react';

const ConfigContext = createContext({});

function ConfigProvider({ children }) {
  const config = useMemo(() => ({
    apiUrl: 'https://api.example.com',
    timeout: 5000,
    features: { darkMode: true, beta: false }
  }), []);

  return (
    <ConfigContext value={config}>
      {children}
    </ConfigContext>
  );
}

function useConfig() {
  return useContext(ConfigContext);
}

function DataFetcher() {
  const { apiUrl } = useConfig();
  // Use apiUrl for fetch requests
}
```

**Explanation**

The config object is memoized so it isn't recreated on every render. Components read the config via `useContext` or a custom hook .

### Real-World Cases

**Case: Feature Flags**

A `FeatureFlagContext` provides flags to the entire app, allowing components to conditionally render features.

### References

- CoreUI — How to use Context API in React - https://coreui.io/answers/how-to-use-context-api-in-react/
- React — Passing Data Deeply with Context - https://18.react.dev/learn/passing-data-deeply-with-context

---

## 6. Theme State

### Definitions

**Core Definition**

Theme state is the current visual theme (light/dark) stored in state and shared via context.

**Technical Definition**

Theme state is held in a `useState` variable in a provider component and passed as the context value. Toggling the theme calls the setter, which updates the context value and re-renders all theme consumers .

**Beginner-Friendly Explanation**

Theme state is like a light switch that controls the whole app. When you flip it, all components that read the theme update their appearance.

### Purposes

- To allow users to switch between light and dark modes.
- To apply consistent styling across all components.
- To persist theme preference.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
const ThemeContext = createContext('light');

function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  return (
    <ThemeContext value={{ theme, setTheme }}>
      {children}
    </ThemeContext>
  );
}
```

### Annotated Complete Code Example

**Example: Theme Toggle**

```jsx
import { createContext, useContext, useState, useMemo, useCallback } from 'react';

const ThemeContext = createContext();

function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');

  const toggleTheme = useCallback(() => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  }, []);

  const value = useMemo(() => ({ theme, toggleTheme }), [theme, toggleTheme]);

  return (
    <ThemeContext value={value}>
      {children}
    </ThemeContext>
  );
}

function useTheme() {
  return useContext(ThemeContext);
}

function ThemedButton() {
  const { theme, toggleTheme } = useTheme();
  return (
    <button onClick={toggleTheme} className={`btn-${theme}`}>
      Switch to {theme === 'light' ? 'dark' : 'light'} mode
    </button>
  );
}
```

**Explanation**

The `ThemeProvider` holds `theme` state and provides it with `toggleTheme`. The value is memoized to prevent unnecessary re-renders. `ThemedButton` reads and toggles the theme .

### Real-World Cases

**Case: Dashboard Theme**

A dashboard app provides a theme context, and all widgets read the theme to style themselves consistently.

### References

- CoreUI — How to use Context API in React - https://coreui.io/answers/how-to-use-context-api-in-react/
- React — useContext (Updating a value via context) - https://18.react.dev/reference/react/useContext

---

## 7. Authentication State

### Definitions

**Core Definition**

Authentication state is the current user's login status and profile information, shared through context.

**Technical Definition**

An `AuthContext` holds the current user object and functions for login/logout. The provider manages this state, and components read it to conditionally render UI or access user data .

**Beginner-Friendly Explanation**

Authentication state is like a membership card that the whole app can check. If you're logged in, the app knows who you are and can show your information everywhere.

### Purposes

- To track whether a user is logged in.
- To provide the current user's data throughout the app.
- To enable login/logout from any component.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
const AuthContext = createContext(null);

function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const login = async (username, password) => { /* ... */ };
  const logout = () => setUser(null);

  return (
    <AuthContext value={{ user, login, logout }}>
      {children}
    </AuthContext>
  );
}
```

### Annotated Complete Code Example

**Example: Authentication Provider and Hook**

```jsx
import { createContext, useContext, useState, useMemo, useCallback } from 'react';

const AuthContext = createContext(null);

function AuthProvider({ children }) {
  const [user, setUser] = useState(null);

  const login = useCallback(async (username, password) => {
    const response = await fetch('/api/login', {
      method: 'POST',
      body: JSON.stringify({ username, password })
    });
    const userData = await response.json();
    setUser(userData);
  }, []);

  const logout = useCallback(() => setUser(null), []);

  const value = useMemo(() => ({ user, login, logout }), [user, login, logout]);

  return (
    <AuthContext value={value}>
      {children}
    </AuthContext>
  );
}

function useAuth() {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
}

function LoginForm() {
  const { login } = useAuth();
  const handleSubmit = (e) => {
    e.preventDefault();
    const formData = new FormData(e.target);
    login(formData.get('username'), formData.get('password'));
  };
  return (
    <form onSubmit={handleSubmit}>
      <input name="username" placeholder="Username" />
      <input name="password" type="password" placeholder="Password" />
      <button type="submit">Login</button>
    </form>
  );
}

function UserProfile() {
  const { user, logout } = useAuth();
  if (!user) return <p>Not logged in</p>;
  return (
    <div>
      <h2>{user.name}</h2>
      <button onClick={logout}>Log out</button>
    </div>
  );
}
```

**Explanation**

The `AuthProvider` manages `user` state and provides `login` and `logout` functions. The `useAuth` custom hook reads the context and throws if used outside the provider. `LoginForm` and `UserProfile` consume the auth context .

### Real-World Cases

**Case: Protected Routes**

A `ProtectedRoute` component uses `useAuth` to check if the user is logged in, redirecting to login if not.

### References

- CoreUI — How to use Context API in React - https://coreui.io/answers/how-to-use-context-api-in-react/
- React — Scaling Up with Reducer and Context - https://react.dev/learn/scaling-up-with-reducer-and-context

---

## References

- React Official Documentation — useContext - https://react.dev/reference/react/useContext
- React Official Documentation — useContext (React 18) - https://18.react.dev/reference/react/useContext
- React Official Documentation — createContext - https://react.dev/reference/react/createContext
- React Official Documentation — createContext (React 18) - https://18.react.dev/reference/react/createContext
- React Official Documentation — Passing Data Deeply with Context - https://18.react.dev/learn/passing-data-deeply-with-context
- React Official Documentation — Scaling Up with Reducer and Context - https://react.dev/learn/scaling-up-with-reducer-and-context
- React Official Documentation (Chinese) — useContext - https://zh-hans.react.dev/reference/react/useContext
- React Official Documentation (Chinese) — createContext - https://zh-hans.react.dev/reference/react/createContext
- React Official Documentation (Chinese) — 使用 Context 深层传递参数 - https://zh-hans.react.dev/learn/passing-data-deeply-with-context
- CoreUI — How to use Context API in React - https://coreui.io/answers/how-to-use-context-api-in-react/
- React Official Documentation (German) — useContext - https://de.react.dev/reference/react/useContext
- React Official Documentation (Portuguese) — createContext - https://pt-br.react.dev/reference/react/createContext
- React Official Documentation (Japanese) — リデューサとコンテクストでスケールアップ - https://ja.react.dev/learn/scaling-up-with-reducer-and-context
- React Official Documentation (French) — Mise à l'échelle en combinant réducteur et contexte - https://fr.react.dev/learn/scaling-up-with-reducer-and-context
- GitHub — Passing Data Deeply with Context Summary - https://github.com/saschb2b/okf-bundles/blob/main/bundles/react/managing-state/passing-data-deeply-with-context.md