# React Error Boundaries — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** An error boundary is a React component that catches JavaScript errors anywhere in its child component tree, logs those errors, and displays a fallback UI instead of the component tree that crashed.

**Technical Definition:** A class component becomes an error boundary if it defines either (or both) of the lifecycle methods `static getDerivedStateFromError()` or `componentDidCatch()`. `getDerivedStateFromError` is used to render a fallback UI after an error has been thrown, while `componentDidCatch` is used to log error information. Error boundaries catch errors during rendering, in lifecycle methods, and in constructors of the whole tree below them. They work like a JavaScript `catch {}` block, but for components, and only class components can serve as error boundaries in vanilla React.

**Beginner-Friendly Explanation:** An error boundary is a safety net for part of your UI. If a component inside it crashes during rendering, the error boundary catches the crash and shows a friendly message (or a retry button) instead of letting the whole page go blank. It is like wrapping a fragile part of your interface in bubble wrap: if something breaks inside, only that section is affected, and the rest of your app keeps working.

### Key Characteristics

- **Class-only in vanilla React:** Error boundaries must be class components; there is no built-in function-component primitive.
- **Child-tree scope:** An error boundary only catches errors in the components below it; it cannot catch errors within itself.
- **Rendering-error focus:** It catches errors during rendering, in lifecycle methods, and in constructors — not in event handlers, async code, or SSR.
- **Fallback UI:** It renders a developer-controlled fallback instead of the crashed subtree, preventing a blank screen.
- **Uncaught errors unmount the tree:** As of React 16, errors not caught by any boundary unmount the whole React component tree.
- **Community wrapper:** The `react-error-boundary` library provides a convenient function-component wrapper with reset keys, fallback components, and hooks.

### Prerequisites

- React function components and JSX.
- Understanding of the render → commit lifecycle.
- Familiarity with class component lifecycle methods (for vanilla implementation).
- Knowledge of `try`/`catch` and asynchronous JavaScript error handling.

### Related Programming Areas

- **Suspense** — error boundaries catch errors thrown by suspended components when a Promise rejects.
- **Data Fetching** — errors from Suspense-enabled sources propagate to the nearest boundary.
- **Routing** — route-level boundaries isolate page crashes.
- **Observability** — `componentDidCatch` and `onError` are the integration points for error reporting services.

---

## 1. Error Boundaries

### Definitions

**Core Definition:** An error boundary is a React component that catches rendering errors in its child tree and displays a fallback UI.

**Technical Definition:** An error boundary is a class component that implements `static getDerivedStateFromError()` to update its state and render a fallback, and optionally `componentDidCatch()` to log the error. It establishes a boundary: errors thrown by descendants are caught, state is updated to reflect the error, and the fallback is rendered instead of the crashed subtree. Errors in the boundary's own render are not caught; they propagate to the closest boundary above it.

**Beginner-Friendly Explanation:** An error boundary is a component that says: “If anything below me crashes, I will catch it and show this fallback instead.”

### Purposes

- To prevent a single component failure from crashing the entire application.
- To display a meaningful fallback UI instead of a blank screen.
- To log rendering errors to monitoring services.
- To isolate failures to the smallest possible UI section.
- To provide recovery mechanisms (retry, reset) for failed sections.

### Syntax Rules and Structure

**General Syntax (Class Component):**

```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, info) {
    logErrorToMyService(error, info);
  }

  render() {
    if (this.state.hasError) {
      return <h1>Something went wrong.</h1>;
    }
    return this.props.children;
  }
}
```

**Component Breakdown:**

| Method | Role |
|--------|------|
| `static getDerivedStateFromError(error)` | Updates state to trigger the fallback render. |
| `componentDidCatch(error, info)` | Logs the error to a reporting service. |
| `render()` | Renders the fallback when `hasError` is true; otherwise renders `children`. |

**Syntax Rules:**

1. A class component becomes an error boundary if it defines `getDerivedStateFromError`, `componentDidCatch`, or both.
2. Use `getDerivedStateFromError` to render a fallback; use `componentDidCatch` only for logging.
3. `getDerivedStateFromError` must be pure and synchronous; it cannot contain side effects.
4. The fallback can be any custom UI, not just an error message.

**Constraints and Limitations:**

- Only class components can be error boundaries in vanilla React.
- Error boundaries do not catch errors in event handlers, async code (`setTimeout`, Promises), SSR, or the boundary itself.
- Uncaught errors unmount the entire React tree.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Basic Class Error Boundary

```jsx
import React from 'react';

class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    // Update state so the next render shows the fallback UI
    return { hasError: true };
  }

  componentDidCatch(error, info) {
    // Log the error to an error reporting service
    logErrorToMyService(error, info.componentStack);
  }

  render() {
    if (this.state.hasError) {
      return <h1>Something went wrong.</h1>;
    }
    return this.props.children;
  }
}

// Usage
<ErrorBoundary>
  <MyWidget />
</ErrorBoundary>
```

**Expected Output:** If `MyWidget` throws during render, the boundary shows “Something went wrong.” instead of the widget. The rest of the application remains functional.

**Why this output:** `getDerivedStateFromError` sets `hasError` to `true`, triggering a re-render with the fallback. `componentDidCatch` logs the error without affecting the UI.

#### Example 2: Using the `react-error-boundary` Library

```jsx
import { ErrorBoundary } from 'react-error-boundary';

function Fallback({ error, resetErrorBoundary }) {
  return (
    <div role="alert">
      <p>Something went wrong: {error.message}</p>
      <button onClick={resetErrorBoundary}>Try again</button>
    </div>
  );
}

function App() {
  return (
    <ErrorBoundary
      FallbackComponent={Fallback}
      onError={(error, info) => logErrorToMyService(error, info)}
      resetKeys={[userId]}
    >
      <UserProfile userId={userId} />
    </ErrorBoundary>
  );
}
```

**Expected Output:** If `UserProfile` throws, the `Fallback` renders with the error message and a retry button. Changing `userId` automatically resets the boundary.

**Why this output:** `react-error-boundary` wraps the class implementation and provides `FallbackComponent`, `onError`, and `resetKeys` as declarative props.

### Real-World Cases

- **Route-level isolation:** Wrapping each route so a page crash does not take down the app shell.
- **Widget isolation:** Wrapping independent dashboard widgets so one failure does not hide the others.
- **Third-party components:** Wrapping components from external libraries that may throw unexpectedly.

### References

- React — Error Boundaries — https://ru.react.js.org/docs/error-boundaries.html
- React — Catching rendering errors with an error boundary — https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary
- react-error-boundary — npm — https://www.npmjs.com/package/react-error-boundary

---

## 2. `componentDidCatch`

### Definitions

**Core Definition:** `componentDidCatch` is a lifecycle method called after an error has been thrown by a descendant component, used for logging error information.

**Technical Definition:** `componentDidCatch(error, info)` is invoked during the commit phase after an error has been caught by the boundary. It receives the thrown `error` and an `info` object containing a `componentStack` trace. It is the correct place for side effects such as logging to an error reporting service. It should not be used to update state for rendering the fallback; that is the role of `getDerivedStateFromError`.

**Beginner-Friendly Explanation:** `componentDidCatch` is where you write down what went wrong. You use it to send the error to a logging service so you can fix it later.

### Purposes

- To log errors to monitoring or analytics services.
- To capture the component stack trace for debugging.
- To perform side effects (e.g., incrementing error counters) after a crash.
- To integrate with error reporting platforms (Sentry, Datadog, etc.).

### Syntax Rules and Structure

```jsx
componentDidCatch(error, info) {
  logErrorToMyService(error, info.componentStack);
}
```

**Syntax Rules:**

1. `componentDidCatch` is optional; `getDerivedStateFromError` is sufficient for rendering a fallback.
2. It runs during the commit phase, after the fallback has been rendered.
3. It receives two arguments: `error` and `info` (`info.componentStack`).
4. It should contain side effects only, not state updates for rendering.

**Constraints and Limitations:**

- It does not catch errors thrown in the boundary itself.
- It is not called for errors in event handlers or async code.

### Annotated Code Examples

```jsx
class ErrorBoundary extends React.Component {
  state = { hasError: false };

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, info) {
    // ✅ Correct: logging in componentDidCatch
    logErrorToMyService(error, info.componentStack);
  }

  render() {
    if (this.state.hasError) {
      return <FallbackUI />;
    }
    return this.props.children;
  }
}
```

**Expected Output:** When a child throws, the boundary renders the fallback and logs the error with its component stack.

**Why this output:** `getDerivedStateFromError` handles the state update for the fallback; `componentDidCatch` handles the logging side effect.

### Real-World Cases

- **Production monitoring:** Sending caught errors to Sentry or a custom logging endpoint.
- **Debugging:** Capturing `componentStack` to locate the failing component.
- **Metrics:** Counting error occurrences per boundary.

### References

- React — componentDidCatch — https://react.dev/reference/react/Component#componentdidcatch
- Stack Overflow — Use getDerivedStateFromError for fallback, componentDidCatch for logging — https://stackoverflow.com/revisions/b5a16513-fc20-4b84-ab19-eb9ffc70176c/view-source

---

## 3. `getDerivedStateFromError`

### Definitions

**Core Definition:** `getDerivedStateFromError` is a static lifecycle method that updates the boundary's state in response to a caught error, causing the fallback UI to render.

**Technical Definition:** `static getDerivedStateFromError(error)` is called during the render phase after a descendant throws. It must return an object to update state or `null` to update nothing. It is the mechanism that transitions the boundary from rendering `children` to rendering the fallback. It must be pure — no side effects — because it runs during rendering.

**Beginner-Friendly Explanation:** `getDerivedStateFromError` is the switch that flips the boundary into “error mode.” It tells React: “An error happened; show the fallback instead.”

### Purposes

- To update state so the fallback UI renders.
- To capture error details in state for the fallback to display.
- To transition the boundary from children to fallback cleanly.

### Syntax Rules and Structure

```jsx
static getDerivedStateFromError(error) {
  return { hasError: true };
}
```

**Syntax Rules:**

1. It must be `static` (it has no access to `this`).
2. It must return an object to update state or `null`.
3. It must be pure and synchronous; no side effects.
4. It runs during the render phase, before commit.

**Constraints and Limitations:**

- Cannot perform side effects (logging belongs in `componentDidCatch`).
- Cannot be async.
- Does not catch errors in the boundary itself.

### Annotated Code Examples

```jsx
class ErrorBoundary extends React.Component {
  state = { hasError: false, error: null };

  static getDerivedStateFromError(error) {
    // ✅ Pure state update for the fallback
    return { hasError: true, error };
  }

  render() {
    if (this.state.hasError) {
      return <FallbackUI error={this.state.error} />;
    }
    return this.props.children;
  }
}
```

**Expected Output:** The fallback renders with the caught error object available for display.

**Why this output:** `getDerivedStateFromError` returns a new state object containing the error, triggering a re-render with the fallback.

### Real-World Cases

- **User-facing error messages:** Storing the error message to display in the fallback.
- **Error type discrimination:** Choosing different fallback UIs based on error type.
- **Recovery flows:** Resetting state when the user retries.

### References

- React — static getDerivedStateFromError — https://react.dev/reference/react/Component#static-getderivedstatefromerror
- Stack Overflow — Use getDerivedStateFromError for fallback, componentDidCatch for logging — https://stackoverflow.com/revisions/b5a16513-fc20-4b84-ab19-eb9ffc70176c/view-source

---

## 4. Error Boundary Placement

### Definitions

**Core Definition:** Error boundary placement is the decision of where in the component tree to introduce boundaries so that failures are isolated at the right level.

**Technical Definition:** Error boundaries can be placed at any level of the tree. Placement determines the blast radius of a rendering error: a boundary at the root isolates nothing (any error takes down the whole page), while a boundary around each independent section isolates failures to that section. The React documentation states that the granularity is up to the developer: top-level route boundaries for page-level errors, and individual widget boundaries for component-level isolation.

**Beginner-Friendly Explanation:** Where you put the safety net determines what gets saved. Put it around the whole app, and a single bug takes everything down. Put it around each section, and only the broken section shows an error.

### Purposes

- To isolate failures to the smallest meaningful UI section.
- To keep the rest of the application functional when one part crashes.
- To provide context-appropriate fallback UIs (page-level vs section-level).
- To enable recovery without a full page reload.

### Syntax Rules and Structure

**Placement Strategy:**

| Level | Scope | Use for |
|-------|-------|---------|
| Application root | Entire app | Catastrophic failures, last-resort fallback |
| Route level | Each route | Page-level error isolation |
| Layout section | Sidebar, header, content | Independent layout regions |
| Widget level | Individual widgets | Independent data-fetching components |

**Syntax Rules:**

1. Place boundaries at meaningful UI sections where an error state makes sense to the user.
2. Avoid a single root boundary for apps with independent sections; it causes full-page crashes.
3. Wrap route components to show route-specific error pages.
4. Wrap independent data-fetching sections to prevent one API failure from hiding the rest of the page.

### Annotated Code Examples

**❌ Single root boundary (one failure kills everything):**

```jsx
<ErrorBoundary fallback={<FullPageError />}>
  <Dashboard />
</ErrorBoundary>
```

**✅ Per-section boundaries (failures stay contained):**

```jsx
function DashboardPage() {
  return (
    <>
      <DashboardHeader />
      <ErrorBoundary FallbackComponent={SectionError}>
        <RevenueChart />
      </ErrorBoundary>
      <ErrorBoundary FallbackComponent={SectionError}>
        <RecentOrders />
      </ErrorBoundary>
      <ErrorBoundary FallbackComponent={SectionError}>
        <TeamActivity />
      </ErrorBoundary>
    </>
  );
}
```

**Expected Output:** If `RevenueChart` throws, only its section shows the error fallback; `RecentOrders` and `TeamActivity` remain fully functional.

**Why this output:** Each boundary catches errors from its own child tree. The failure is isolated to the section that actually crashed.

### Real-World Cases

- **Dashboards:** Independent boundaries per widget so one failing chart does not hide the entire dashboard.
- **E-commerce:** Product info, reviews, and recommendations as separate boundaries.
- **Admin panels:** Route-level boundaries with route-specific error pages.

### References

- React — Where to Place Error Boundaries — https://ru.react.js.org/docs/error-boundaries.html
- Error Boundary Placement Strategies — https://github.com/oakoss/agent-skills/blob/85e3a3919d9e0ec7f7302a5143ec4b3e66f5f6ad/skills/react-error-handling/references/boundary-placement.md
- Place Error Boundaries at Data Fetch Granularity — https://github.com/pproenca/dot-skills/blob/HEAD/skills/.experimental/react-refactor/references/data-granular-error-boundaries.md

---

## 5. Error Boundary Granularity

### Definitions

**Core Definition:** Error boundary granularity is the size and number of UI sections each boundary protects — how coarse or fine the isolation is.

**Technical Definition:** Granularity is a trade-off between isolation and complexity. Coarse granularity (one boundary for the whole app) is simple but causes total failure. Fine granularity (one boundary per tiny element) provides maximum isolation but adds excessive boilerplate and confusing error states. The right granularity groups components that logically fail together — a `UserCard` containing an avatar, name, and bio should share one boundary, not three.

**Beginner-Friendly Explanation:** Granularity is about how big each safety net is. Too big, and one bug crashes everything. Too small, and you have a hundred nets to maintain.

### Purposes

- To balance failure isolation against code complexity.
- To group components that logically fail as a unit.
- To avoid both full-page crashes and excessive boundary boilerplate.
- To provide error states that make sense to the user.

### Syntax Rules and Structure

**Granularity Decision Guide:**

| Granularity | Example | Trade-off |
|-------------|---------|-----------|
| Too coarse | `<ErrorBoundary><App /></ErrorBoundary>` | One error crashes everything |
| Too fine | One boundary per `<Avatar>`, `<UserName>`, `<UserBio>` | Excessive boilerplate, confusing states |
| Right balance | One boundary around `<UserCard>` containing all three | Logical grouping that fails as a unit |

**Syntax Rules:**

1. Group components that share a data source or feature into one boundary.
2. Separate boundaries for components that can fail independently.
3. Do not wrap every element; wrap meaningful UI sections.
4. Use route-level boundaries for page isolation and widget-level boundaries for data-fetching sections.

### Annotated Code Examples

**✅ Right balance (logical grouping):**

```jsx
<ErrorBoundary FallbackComponent={UserCardError}>
  <UserCard>
    <Avatar />
    <UserName />
    <UserBio />
  </UserCard>
</ErrorBoundary>
```

**Expected Output:** If any part of `UserCard` throws, the `UserCardError` fallback renders for the entire card, not for individual pieces.

**Why this output:** The boundary wraps a logical unit (`UserCard`). If the avatar, name, or bio fails, the whole card is replaced by a coherent error state, which is easier for the user to understand than three partial errors.

### Real-World Cases

- **Design systems:** One boundary per card or panel, not per element.
- **Data tables:** One boundary per table, not per row.
- **Navigation:** Separate boundaries for sidebar and main content.

### References

- Error Boundary Placement Strategies — https://github.com/oakoss/agent-skills/blob/85e3a3919d9e0ec7f7302a5143ec4b3e66f5f6ad/skills/react-error-handling/references/boundary-placement.md
- React — Where to Place Error Boundaries — https://ru.react.js.org/docs/error-boundaries.html

---

## 6. Error Boundary Limitations

### Definitions

**Core Definition:** Error boundary limitations are the categories of errors that error boundaries cannot catch, requiring alternative handling strategies.

**Technical Definition:** Error boundaries catch errors only during rendering, in lifecycle methods, and in constructors of child components. They explicitly do not catch: (1) errors in event handlers, (2) errors in asynchronous code (`setTimeout`, `requestAnimationFrame`, Promises), (3) errors during server-side rendering, and (4) errors thrown in the boundary itself rather than its children.

**Beginner-Friendly Explanation:** Error boundaries are not magic. They catch crashes that happen while React is drawing the screen. They do not catch crashes that happen when you click a button, when a timer fires, or when a Promise rejects.

### Purposes

- To understand the precise scope of error boundary coverage.
- To choose the right error-handling strategy for each error category.
- To avoid relying on error boundaries for errors they cannot catch.

### Limitations Table

| Error category | Caught by boundary? | Correct handling |
|----------------|---------------------|------------------|
| Rendering errors | Yes | Error boundary |
| Lifecycle method errors | Yes | Error boundary |
| Constructor errors | Yes | Error boundary |
| Event handler errors | No | `try`/`catch` inside the handler |
| Async code (`setTimeout`, Promises) | No | `try`/`catch` + `useErrorBoundary` or `startTransition`（React 19） |
| Server-side rendering errors | No | Server-level error handling |
| Errors in the boundary itself | No | Parent boundary |

**Syntax Rules:**

1. Use `try`/`catch` in event handlers; error boundaries will not catch them.
2. Use `useErrorBoundary` (from `react-error-boundary`) or `startTransition` (React 19) to propagate async errors to a boundary.
3. Handle SSR errors on the server; boundaries do not catch them.
4. Place a parent boundary above any boundary that might fail itself.

### Annotated Code Examples

**❌ Relying on a boundary for an event handler:**

```jsx
<ErrorBoundary>
  <button onClick={() => { throw new Error('Click failed'); }}>
    Click
  </button>
</ErrorBoundary>
```

**✅ Correct: `try`/`catch` in the event handler:**

```jsx
function SafeButton() {
  function handleClick() {
    try {
      riskyOperation();
    } catch (error) {
      logError(error);
      showToast('Something went wrong. Please try again.');
    }
  }
  return <button onClick={handleClick}>Click</button>;
}
```

**✅ Correct: `useErrorBoundary` for async errors:**

```jsx
import { useErrorBoundary } from 'react-error-boundary';

function AsyncComponent() {
  const { showBoundary } = useErrorBoundary();

  useEffect(() => {
    fetchData().catch(error => showBoundary(error));
  }, [showBoundary]);

  return <div>Loading…</div>;
}
```

**Expected Output:** The event handler catches its own error; the async error is forwarded to the nearest boundary.

**Why this output:** Event handler errors never reach the boundary. Async errors need to be explicitly forwarded to a boundary via `showBoundary` or a React 19 transition action.

### Real-World Cases

- **Form submissions:** `try`/`catch` in submit handlers; boundaries do not catch them.
- **Data fetching:** `useErrorBoundary` or query library `onError` for async failures.
- **SSR:** Server-level error middleware; boundaries do not run on the server.

### References

- React — Error Boundaries: Limitations — https://ru.react.js.org/docs/error-boundaries.html
- Frontend Error Management — https://github.com/packmindhub/packmind/blob/8047be6447b1e9116d9e2fc2cfec701c0a415551/apps/frontend/.packmind/standards/frontend-error-management.md
- react-error-boundary PR #242 — https://github.com/bvaughn/react-error-boundary/pull/242/files

---

## 7. Error Boundary Alternatives

### Definitions

**Core Definition:** Error boundary alternatives are libraries and patterns that provide error-handling capabilities beyond vanilla class-based error boundaries.

**Technical Definition:** Vanilla React requires class components for error boundaries. Alternatives include: `react-error-boundary` (a function-component wrapper with `FallbackComponent`, `resetKeys`, `onError`, and `useErrorBoundary` hooks), `@suspensive/react` (a declarative boundary with `shouldCatch` filtering and `ErrorBoundaryGroup`), and `@sentry/react` (Sentry integration with `withErrorBoundary`). React 19 introduces `onCaughtError` and `onUncaughtError` root options for global error handling.

**Beginner-Friendly Explanation:** If you do not want to write class components, you can use a library that wraps the class for you. These libraries give you extra features like retry buttons, error filtering, and automatic resets.

### Purposes

- To avoid writing class components for every boundary.
- To gain features like reset keys, error filtering, and fallback prop drilling.
- To integrate with error reporting services.
- To leverage React 19's global error handling options.

### Comparison Table

| Solution | Type | Key features |
|----------|------|--------------|
| Vanilla class boundary | Class component | Full control, no dependency |
| `react-error-boundary` | Library | Function wrapper, `resetKeys`, `onError`, `useErrorBoundary` |
| `@suspensive/react` | Library | `shouldCatch` filtering, `ErrorBoundaryGroup`, safe fallback propagation |
| `@sentry/react` | Library | Sentry integration, `withErrorBoundary` |
| React 19 root options | Built-in | `onCaughtError`, `onUncaughtError` for global handling |

**Syntax Rules:**

1. Use `react-error-boundary` for a lightweight, declarative wrapper with reset support.
2. Use `@suspensive/react` when you need error-type filtering or grouped resets.
3. Use `@sentry/react` when error reporting is the primary concern.
4. Use React 19 root options for global error logging without a boundary component.

### Annotated Code Examples

**`react-error-boundary`:**

```jsx
import { ErrorBoundary } from 'react-error-boundary';

<ErrorBoundary
  FallbackComponent={MyFallback}
  onError={(error, info) => logError(error, info)}
  resetKeys={[userId]}
>
  <UserProfile userId={userId} />
</ErrorBoundary>
```

**`@suspensive/react` with `shouldCatch`:**

```jsx
import { ErrorBoundary } from '@suspensive/react';

<ErrorBoundary
  shouldCatch={ZodError}
  fallback={<ValidationErrorUI />}
>
  <Form />
</ErrorBoundary>
```

**Expected Output:** `react-error-boundary` renders `MyFallback` and resets when `userId` changes. `@suspensive/react` catches only `ZodError` and renders `ValidationErrorUI` for validation failures.

**Why this output:** `react-error-boundary` provides a declarative wrapper around the class API. `@suspensive/react` adds error-type filtering via `shouldCatch`.

### Real-World Cases

- **Design systems:** `react-error-boundary` for consistent fallback components.
- **Form-heavy apps:** `@suspensive/react` for catching specific validation errors.
- **Production monitoring:** `@sentry/react` for automatic error reporting.
- **React 19 migration:** Root `onCaughtError` / `onUncaughtError` for global logging.

### References

- react-error-boundary — npm — https://www.npmjs.com/package/react-error-boundary
- Suspensive vs Other Error Boundary Libraries — https://suspensive.org/en/docs/react/comparison
- React 19 — Root options — https://react.dev/reference/react-dom/client/createRoot#parameters

---

## 8. Error Boundary Best Practices

### Definitions

**Core Definition:** Error boundary best practices are the guidelines for placing, implementing, and operating error boundaries to maximise resilience and user experience.

**Technical Definition:** Best practices encompass: placing boundaries at meaningful UI sections (not one root boundary, not one per element), using `getDerivedStateFromError` for fallback state and `componentDidCatch` for logging, providing reset mechanisms (`resetKeys` or retry buttons), handling non-rendering errors with `try`/`catch` and `useErrorBoundary`, testing boundaries with error-throwing components, and avoiding error boundaries for expected validation or API errors.

**Beginner-Friendly Explanation:** Put boundaries around sections that make sense to fail together. Use the right method for the right job. Always give users a way to recover. And remember: boundaries are for unexpected crashes, not for form validation.

### Best Practices Table

| Practice | Description |
|----------|-------------|
| **Place at logical sections** | Wrap meaningful UI units, not the whole app or every element. |
| **Use `getDerivedStateFromError` for fallback** | Keep it pure; do not perform side effects. |
| **Use `componentDidCatch` for logging** | Side effects belong here, not in the static method. |
| **Provide a reset mechanism** | Use `resetKeys` or a retry button to recover without reload. |
| **Handle non-rendering errors separately** | Use `try`/`catch` for event handlers; `useErrorBoundary` for async. |
| **Do not use boundaries for expected errors** | Form validation and expected API errors should be handled inline. |
| **Test boundaries** | Render error-throwing components to verify fallback and logging. |
| **Log to a monitoring service** | Always wire `onError` or `componentDidCatch` to Sentry or similar. |
| **Avoid showing stack traces in production** | Display user-friendly messages; keep stack traces for development. |

### Annotated Code Examples

**✅ Good Practices:**

```jsx
import { ErrorBoundary } from 'react-error-boundary';

function SectionError({ error, resetErrorBoundary }) {
  return (
    <div role="alert">
      <p>Could not load this section.</p>
      <button onClick={resetErrorBoundary}>Retry</button>
    </div>
  );
}

function Dashboard() {
  return (
    <>
      <Header />
      <ErrorBoundary
        FallbackComponent={SectionError}
        onError={(error, info) => logErrorToService(error, info)}
        resetKeys={[filters]}
      >
        <RevenueChart filters={filters} />
      </ErrorBoundary>
      <ErrorBoundary FallbackComponent={SectionError}>
        <RecentOrders />
      </ErrorBoundary>
    </>
  );
}
```

**❌ Bad Practices:**

```jsx
// ❌ Single root boundary: one failure crashes everything
<ErrorBoundary fallback={<FullPageError />}>
  <App />
</ErrorBoundary>

// ❌ Logging in getDerivedStateFromError (must be pure)
static getDerivedStateFromError(error) {
  logErrorToService(error); // Side effect!
  return { hasError: true };
}

// ❌ No reset mechanism: user is stuck
<ErrorBoundary fallback={<p>Error</p>}>
  <Component />
</ErrorBoundary>
```

**Expected Output:** The good practices isolate failures per section, log errors correctly, and provide recovery. The bad practices cause full-page crashes, impure lifecycle methods, and stuck error states.

**Why this output:** Per-section boundaries contain failures. `getDerivedStateFromError` must remain pure. Reset mechanisms allow recovery without a reload.

### Real-World Cases

- **All React applications:** These practices apply universally.
- **Design systems:** Consistent fallback components with reset support.
- **Production monitoring:** `onError` wired to Sentry or Datadog.

### References

- React — Error Boundaries — https://ru.react.js.org/docs/error-boundaries.html
- Frontend Error Management — https://github.com/packmindhub/packmind/blob/8047be6447b1e9116d9e2fc2cfec701c0a415551/apps/frontend/.packmind/standards/frontend-error-management.md
- react-error-handling — https://raw.githubusercontent.com/oakoss/agent-skills/refs/heads/main/skills/react-error-handling/SKILL.md
- Place Error Boundaries at Data Fetch Granularity — https://github.com/pproenca/dot-skills/blob/HEAD/skills/.experimental/react-refactor/references/data-granular-error-boundaries.md

---

## References

- React — Error Boundaries — https://ru.react.js.org/docs/error-boundaries.html
- React — Catching rendering errors with an error boundary — https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary
- React — componentDidCatch — https://react.dev/reference/react/Component#componentdidcatch
- React — static getDerivedStateFromError — https://react.dev/reference/react/Component#static-getderivedstatefromerror
- Stack Overflow — Use getDerivedStateFromError for fallback, componentDidCatch for logging — https://stackoverflow.com/revisions/b5a16513-fc20-4b84-ab19-eb9ffc70176c/view-source
- Frontend Error Management — https://github.com/packmindhub/packmind/blob/8047be6447b1e9116d9e2fc2cfec701c0a415551/apps/frontend/.packmind/standards/frontend-error-management.md
- Error Boundary Placement Strategies — https://github.com/oakoss/agent-skills/blob/85e3a3919d9e0ec7f7302a5143ec4b3e66f5f6ad/skills/react-error-handling/references/boundary-placement.md
- Place Error Boundaries at Data Fetch Granularity — https://github.com/pproenca/dot-skills/blob/HEAD/skills/.experimental/react-refactor/references/data-granular-error-boundaries.md
- react-error-boundary PR #242 — https://github.com/bvaughn/react-error-boundary/pull/242/files
- Suspensive vs Other Error Boundary Libraries — https://suspensive.org/en/docs/react/comparison
- react-error-handling — https://raw.githubusercontent.com/oakoss/agent-skills/refs/heads/main/skills/react-error-handling/SKILL.md
- react-error-boundary — npm — https://www.npmjs.com/package/react-error-boundary