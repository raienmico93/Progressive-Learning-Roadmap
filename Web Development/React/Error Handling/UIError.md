# UI & Component Error Management: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** UI & Component Error Management in React refers to the practice of using Error Boundaries—special React components that catch JavaScript errors anywhere in their child component tree, log those errors, and display a fallback UI instead of letting the entire application crash.

**Technical Definition:** Error Boundaries are React class components that implement `static getDerivedStateFromError()` and/or `componentDidCatch()` lifecycle methods. When a descendant component throws an error during rendering, in a lifecycle method, or in a constructor, the nearest Error Boundary above it catches the error. `getDerivedStateFromError` is used to update state so the next render shows a fallback UI, while `componentDidCatch` is used for logging error information to an error reporting service. Error boundaries work like a JavaScript `catch {}` block, but for components.

**Beginner-Friendly Explanation:** Sometimes a component in your React app breaks—maybe a chart fails to render, or a widget crashes because of bad data. Without error management, one broken component can take down your entire app, leaving users with a blank white screen. Error Boundaries are like safety nets: you wrap them around parts of your app, and if anything inside breaks, the Error Boundary catches the problem and shows a friendly error message instead. The rest of your app keeps working.

### Key Characteristics

- **Component-Level Catching:** Error Boundaries catch errors in their child component tree, not in themselves. If an Error Boundary fails to render its fallback, the error propagates to the closest Error Boundary above it.
- **Specific Error Types:** Error Boundaries catch errors during rendering, in lifecycle methods, and in constructors of the whole tree below them. They do not catch errors in event handlers, asynchronous code (e.g., `setTimeout`, promise rejections), server-side rendering, or errors thrown in the Error Boundary itself.
- **Class-Component Requirement:** Only class components can be true Error Boundaries because they use the special `static getDerivedStateFromError` method. There is no direct functional-component equivalent in React.
- **Fallback UI:** When an error is caught, the Error Boundary renders a fallback UI instead of the crashed component tree. This fallback can be as simple as a text message or as detailed as a branded error screen with retry options.
- **Failure Isolation:** By placing Error Boundaries at strategic points in the component tree, you can isolate failures so that a single broken component doesn't crash the entire application.
- **Recovery Support:** Error Boundaries can provide "Try Again" mechanisms to clear the error state and attempt to re-render the failed component without a full page reload.

### Prerequisites

- Solid understanding of React components, props, and state.
- Familiarity with class components and React lifecycle methods.
- Basic understanding of JavaScript error handling (`try`/`catch`).
- Knowledge of React's component tree and rendering process.
- Familiarity with modern React patterns (functional components, Hooks) for understanding the distinction from Error Boundaries.

### Related Programming Areas

- **Error Handling:** JavaScript `try`/`catch`, global error handlers, and error logging.
- **Resilience Engineering:** Designing systems that degrade gracefully under failure.
- **User Experience (UX):** Designing contextual error states and recovery flows.
- **Monitoring & Observability:** Integrating with error reporting services (Sentry, LogRocket).
- **Suspense:** Complementary to Error Boundaries for handling async operations.
- **State Management:** Resetting application state as part of error recovery.

### Core Concepts / Features

1. Error Boundaries
2. Component Failure Isolation
3. Fallback Interfaces
4. Reset & Recovery Mechanisms

---

## Core Concept 1: Error Boundaries

### Definitions

**Core Definition:** An Error Boundary is a React class component that catches JavaScript errors anywhere in its child component tree, logs those errors, and displays a fallback UI instead of the crashed component tree.

**Technical Definition:** A class component becomes an Error Boundary if it defines either (or both) of the lifecycle methods `static getDerivedStateFromError()` or `componentDidCatch()`. `getDerivedStateFromError(error)` is invoked after an error has been thrown by a descendant component. It receives the error and should return a value to update state. It is called during the render phase, so side effects are not permitted. `componentDidCatch(error, errorInfo)` is invoked after an error has been thrown by a descendant component. It receives the error and an `errorInfo` object with a `componentStack` property containing stack trace information. It is called during the commit phase, so side effects are permitted. Error boundaries catch errors during rendering, in lifecycle methods, and in constructors of the whole tree below them.

**Beginner-Friendly Explanation:** An Error Boundary is a special component you build that acts like a safety net for a part of your app. You wrap it around other components, and if any of those components crash, the Error Boundary catches the crash and shows a friendly message instead. You write it as a class component with two special methods: one that updates the state to show the fallback, and another that logs the error for debugging.

### Purposes

- To catch JavaScript errors in child components and prevent the entire application from crashing.
- To display a fallback UI when a component fails, instead of a blank screen.
- To log error information to an error reporting service for debugging.
- To isolate failures to specific parts of the UI tree.
- To provide a consistent error-handling mechanism across the application.
- To enable recovery from errors without requiring a full page reload.

### Syntax Rules and Structure

**General Syntax (Class-Based Error Boundary):**
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

  componentDidCatch(error, errorInfo) {
    // Log the error to an error reporting service
    console.error('Error caught by boundary:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      // Render any custom fallback UI
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

**Component Breakdown:**
- `constructor(props)`: Initialises state with `hasError: false`.
- `static getDerivedStateFromError(error)`: A static method called during the render phase when a descendant throws. Returns `{ hasError: true }` to trigger the fallback UI.
- `componentDidCatch(error, errorInfo)`: A lifecycle method called during the commit phase. Used for logging errors to a service. Receives `error` and `errorInfo` with `componentStack`.
- `render()`: If `hasError` is `true`, renders the fallback UI; otherwise, renders `this.props.children`.
- `<ErrorBoundary>`: Wraps components that might throw errors.

**General Syntax with `react-error-boundary` (Functional Alternative):**
```jsx
import { ErrorBoundary } from 'react-error-boundary';

function ErrorFallback({ error, resetErrorBoundary }) {
  return (
    <div role="alert">
      <h2>Something went wrong</h2>
      <pre style={{ color: 'red' }}>{error.message}</pre>
      <button onClick={resetErrorBoundary}>Try again</button>
    </div>
  );
}

function App() {
  return (
    <ErrorBoundary FallbackComponent={ErrorFallback}>
      <MyComponent />
    </ErrorBoundary>
  );
}
```

**Component Breakdown:**
- `<ErrorBoundary>`: A functional wrapper from the `react-error-boundary` library.
- `FallbackComponent`: A component that receives `error` and `resetErrorBoundary` props.
- `resetErrorBoundary`: A function that resets the Error Boundary's state, causing it to retry rendering its children.
- `fallbackRender`: An alternative prop that accepts a render function instead of a component.

**Syntax Rules:**
- Error Boundaries must be class components (or use the `react-error-boundary` library for a functional equivalent).
- Error Boundaries catch errors during rendering, in lifecycle methods, and in constructors of the whole tree below them.
- Error Boundaries do not catch errors in event handlers, asynchronous code (e.g., `setTimeout`, `requestAnimationFrame` callbacks, unresolved promises), server-side rendering, or errors thrown in the Error Boundary itself.
- `getDerivedStateFromError` must be a static method and must return a state update object.
- `componentDidCatch` is optional; `getDerivedStateFromError` is sufficient for rendering a fallback UI.
- Error Boundaries only catch errors in the components below them in the tree. An Error Boundary cannot catch an error within itself.

**Constraints and Limitations:**
- Only class components can be true Error Boundaries. There is no direct functional-component equivalent in React as of React 19.2.0.
- Error Boundaries do not catch errors in event handlers; use `try`/`catch` for those.
- Error Boundaries do not catch errors in asynchronous code like `setTimeout` or `requestAnimationFrame` callbacks.
- Error Boundaries do not catch errors during server-side rendering.
- Errors thrown in the Error Boundary itself are not caught; they propagate to the nearest Error Boundary above it.
- As of React 16, errors not caught by any Error Boundary result in the unmounting of the whole React component tree.

### Annotated Code Examples

**Example 1: Basic Class-Based Error Boundary**

```jsx
import React from 'react';

// Error Boundary class component
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    // Initialise state: no error yet
    this.state = { hasError: false };
  }

  // Called during the render phase when a descendant throws
  static getDerivedStateFromError(error) {
    // Update state so the next render shows the fallback UI
    return { hasError: true };
  }

  // Called during the commit phase for logging
  componentDidCatch(error, errorInfo) {
    // Log the error to an error reporting service
    console.error('Error caught by boundary:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      // Render the fallback UI
      return (
        <div role="alert">
          <h2>Something went wrong.</h2>
          <p>Please try again later.</p>
        </div>
      );
    }

    // Render children normally
    return this.props.children;
  }
}

// A component that throws an error when a prop is invalid
function BuggyCounter({ count }) {
  if (count > 3) {
    throw new Error('Count is too high!');
  }
  return <p>Count: {count}</p>;
}

// App component using the Error Boundary
function App() {
  const [count, setCount] = React.useState(0);

  return (
    <div>
      <h1>Error Boundary Demo</h1>
      <button onClick={() => setCount(c => c + 1)}>
        Increment ({count})
      </button>

      <ErrorBoundary>
        <BuggyCounter count={count} />
      </ErrorBoundary>

      <p>This content outside the boundary still works.</p>
    </div>
  );
}

export default App;
```

**Expected Output:** The app displays a counter and an "Increment" button. Clicking the button increments the count up to 3, displaying "Count: 3". When the count reaches 4, `BuggyCounter` throws an error, and the Error Boundary replaces it with "Something went wrong. Please try again later." The content outside the boundary ("This content outside the boundary still works.") remains visible.

**Why This Output Occurs:** When `BuggyCounter` throws during rendering, React unwinds to the nearest Error Boundary above it. `getDerivedStateFromError` is called, setting `hasError` to `true`. The next render of `ErrorBoundary` sees `hasError === true` and renders the fallback UI instead of `this.props.children`. `componentDidCatch` logs the error. The rest of the component tree (outside the Error Boundary) is unaffected because the error was caught before propagating further.

**Example 2: Using `react-error-boundary` with FallbackComponent**

```jsx
import React, { useState } from 'react';
import { ErrorBoundary } from 'react-error-boundary';

// Fallback component that receives error and reset function
function ErrorFallback({ error, resetErrorBoundary }) {
  return (
    <div role="alert" style={{ padding: '20px', border: '1px solid red' }}>
      <h2>Something went wrong</h2>
      <pre style={{ color: 'red' }}>{error.message}</pre>
      <button onClick={resetErrorBoundary}>Try again</button>
    </div>
  );
}

// A component that throws based on a flag
function RiskyComponent({ shouldThrow }) {
  if (shouldThrow) {
    throw new Error('Risky component failed!');
  }
  return <p>Risky component rendered successfully.</p>;
}

function App() {
  const [shouldThrow, setShouldThrow] = useState(false);

  return (
    <div>
      <h1>react-error-boundary Demo</h1>
      <button onClick={() => setShouldThrow(prev => !prev)}>
        Toggle Error
      </button>

      <ErrorBoundary
        FallbackComponent={ErrorFallback}
        onReset={() => setShouldThrow(false)}
        resetKeys={[shouldThrow]}
      >
        <RiskyComponent shouldThrow={shouldThrow} />
      </ErrorBoundary>
    </div>
  );
}

export default App;
```

**Expected Output:** The app displays "Risky component rendered successfully." Initially. Clicking "Toggle Error" causes `RiskyComponent` to throw, and the Error Boundary displays "Something went wrong" with "Risky component failed!" and a "Try again" button. Clicking "Try again" resets the Error Boundary and the `shouldThrow` state, returning to the successful render.

**Why This Output Occurs:** The `react-error-boundary` library wraps the class-based Error Boundary in a functional component, exposing a `FallbackComponent` prop and a `resetErrorBoundary` function. When `RiskyComponent` throws, the `ErrorFallback` component is rendered with the `error` and `resetErrorBoundary` props. The `onReset` callback resets the `shouldThrow` state, and `resetKeys` automatically resets the boundary when `shouldThrow` changes. This demonstrates the library's simplified API compared to writing a class component manually.

### Real-World Cases

- **Dashboard applications:** Wrapping individual widgets (charts, tables, feeds) in separate Error Boundaries so a slow or broken widget doesn't crash the entire dashboard.
- **E-commerce product pages:** Isolating the recommendation carousel or review section so a failure in one doesn't break the main product display.
- **Social media feeds:** Wrapping individual posts in Error Boundaries so a single malformed post doesn't break the entire feed.
- **Multi-step forms:** Wrapping each step in an Error Boundary so a failure in one step doesn't lose data from previous steps.
- **Third-party integrations:** Wrapping embedded widgets (payment forms, maps, chat) in Error Boundaries to isolate third-party failures.

---

## Core Concept 2: Component Failure Isolation

### Definitions

**Core Definition:** Component Failure Isolation is the practice of placing Error Boundaries at strategic points in the component tree to ensure that a failure in one part of the UI does not bring down the entire application.

**Technical Definition:** Component Failure Isolation is achieved by strategically positioning Error Boundaries at different levels of granularity within the component tree. A single Error Boundary at the application root means any component failure replaces the entire page with a fallback UI. Placing Error Boundaries around each independent data-fetching section or feature isolates failures so the rest of the page stays functional. The granularity of Error Boundaries is up to the developer: you may wrap top-level route components to display a "Something went wrong" message, or wrap individual widgets to protect them from crashing the rest of the application. A common architecture uses a three-layer boundary system: a global boundary at the root (last line of defence), route-level boundaries (per-page isolation), and feature-level boundaries (finest-grained isolation for individual widgets).

**Beginner-Friendly Explanation:** Imagine your app is like a house with different rooms. If one room catches fire and you don't have fire doors, the whole house burns down. Component Failure Isolation is like installing fire doors between rooms: if one room has a problem, you can close the door and the rest of the house stays safe. In React, these "fire doors" are Error Boundaries placed at different levels of your component tree.

### Purposes

- To prevent a single component failure from unmounting the entire application tree.
- To isolate failures to specific sections of the UI.
- To keep the rest of the application functional when one part fails.
- To provide contextual error handling at different levels of granularity.
- To allow users to continue using unaffected parts of the application.
- To enable targeted recovery without requiring a full page reload.

### Syntax Rules and Structure

**General Syntax for Layered Error Boundaries:**
```jsx
function App() {
  return (
    // Layer 1: Global Error Boundary (last line of defence)
    <GlobalErrorBoundary>
      <Header />

      {/* Layer 2: Route-level Error Boundary */}
      <RouteErrorBoundary>
        <DashboardPage />
      </RouteErrorBoundary>

      <Footer />
    </GlobalErrorBoundary>
  );
}

function DashboardPage() {
  return (
    <div>
      <h1>Dashboard</h1>

      {/* Layer 3: Feature-level Error Boundaries */}
      <ErrorBoundary FallbackComponent={CardError}>
        <RevenueChart />
      </ErrorBoundary>

      <ErrorBoundary FallbackComponent={CardError}>
        <RecentOrders />
      </ErrorBoundary>

      <ErrorBoundary FallbackComponent={CardError}>
        <TeamActivity />
      </ErrorBoundary>
    </div>
  );
}
```

**Component Breakdown:**
- `GlobalErrorBoundary`: The outermost boundary. Catches anything that escapes lower-level boundaries. Shows a full-page fallback with recovery options.
- `RouteErrorBoundary`: Per-route isolation. A crash in one route doesn't affect other routes. The user can navigate away without a full reload.
- `Feature-level ErrorBoundary`: Finest-grained isolation. A broken widget shows an inline fallback while the rest of the page continues working.
- `CardError`: A fallback component designed for card-level errors (inline, minimal).

**Syntax Rules:**
- Place Error Boundaries at data-fetch granularity: around each independent data-fetching section.
- Do not place a single Error Boundary at the application root only; this kills the entire page when any component throws.
- Use separate Error Boundaries for independent sections that should fail independently.
- The granularity of boundaries is up to you: wrap top-level routes for page-level errors, and individual widgets for feature-level errors.
- Avoid adding redundant Error Boundaries where a parent boundary already provides appropriate fallback UX.
- For tiny utility components (e.g., a `<Tooltip>`), let the parent boundary handle errors rather than adding a per-component boundary.

**Constraints and Limitations:**
- Error Boundaries only catch errors in their child component tree; they cannot catch errors in themselves or in event handlers.
- Adding too many Error Boundaries can fragment the fallback UX and make it inconsistent.
- Each Error Boundary adds a small amount of overhead; use them strategically, not everywhere.
- Error Boundaries do not catch errors in asynchronous code; use a bridging mechanism (e.g., `useErrorHandler` from `react-error-boundary`) for async errors.
- The placement of Error Boundaries should align with the natural fault boundaries of the application (routes, features, widgets).

### Annotated Code Examples

**Example 1: Single Root Boundary vs Per-Section Boundaries**

```jsx
import React from 'react';
import { ErrorBoundary } from 'react-error-boundary';

// Fallback for full-page errors
function FullPageError({ error, resetErrorBoundary }) {
  return (
    <div style={{ padding: '40px', textAlign: 'center' }}>
      <h1>Something went wrong</h1>
      <p>{error.message}</p>
      <button onClick={resetErrorBoundary}>Reload Page</button>
    </div>
  );
}

// Fallback for section-level errors
function SectionError({ error, resetErrorBoundary }) {
  return (
    <div role="alert" style={{ padding: '16px', border: '1px solid orange' }}>
      <p>Failed to load: {error.message}</p>
      <button onClick={resetErrorBoundary}>Retry</button>
    </div>
  );
}

// Components that may throw
function RevenueChart() {
  throw new Error('Chart data unavailable');
}
function RecentOrders() {
  return <p>Recent orders loaded.</p>;
}
function TeamActivity() {
  return <p>Team activity loaded.</p>;
}

// ❌ BAD: Single root boundary — one failure kills everything
function BadDashboard() {
  return (
    <ErrorBoundary FallbackComponent={FullPageError}>
      <h1>Dashboard</h1>
      <RevenueChart />
      <RecentOrders />
      <TeamActivity />
    </ErrorBoundary>
  );
}

// ✅ GOOD: Per-section boundaries — failures stay contained
function GoodDashboard() {
  return (
    <div>
      <h1>Dashboard</h1>
      <ErrorBoundary FallbackComponent={SectionError}>
        <RevenueChart />
      </ErrorBoundary>
      <ErrorBoundary FallbackComponent={SectionError}>
        <RecentOrders />
      </ErrorBoundary>
      <ErrorBoundary FallbackComponent={SectionError}>
        <TeamActivity />
      </ErrorBoundary>
    </div>
  );
}

export { BadDashboard, GoodDashboard };
```

**Expected Output:** With `BadDashboard`, the entire dashboard (including the header) is replaced by the full-page error when `RevenueChart` throws. With `GoodDashboard`, only the `RevenueChart` section is replaced by the inline "Failed to load" message; `RecentOrders` and `TeamActivity` remain visible and functional.

**Why This Output Occurs:** In `BadDashboard`, the single Error Boundary wraps the entire dashboard. When `RevenueChart` throws, the boundary catches it and replaces all children with `FullPageError`. In `GoodDashboard`, each section has its own Error Boundary. When `RevenueChart` throws, only its boundary catches the error and renders `SectionError`; the other boundaries and their children are unaffected. This demonstrates the importance of placing Error Boundaries at data-fetch granularity.

**Example 2: Three-Layer Boundary Architecture**

```jsx
import React from 'react';
import { ErrorBoundary } from 'react-error-boundary';

// Layer 1: Global fallback
function GlobalError({ error, resetErrorBoundary }) {
  return (
    <div style={{ padding: '40px', textAlign: 'center' }}>
      <h1>Application Error</h1>
      <p>An unexpected error occurred. Please reload the page.</p>
      <button onClick={() => window.location.reload()}>Reload</button>
    </div>
  );
}

// Layer 2: Route fallback
function RouteError({ error, resetErrorBoundary }) {
  return (
    <div style={{ padding: '20px', border: '2px solid red' }}>
      <h2>Page Error</h2>
      <p>{error.message}</p>
      <button onClick={resetErrorBoundary}>Try Again</button>
    </div>
  );
}

// Layer 3: Feature fallback
function FeatureError({ error, resetErrorBoundary }) {
  return (
    <div role="alert" style={{ padding: '12px', border: '1px solid orange' }}>
      <p>Widget failed: {error.message}</p>
      <button onClick={resetErrorBoundary}>Retry Widget</button>
    </div>
  );
}

// Feature components
function BrokenChart() {
  throw new Error('Chart service unavailable');
}
function WorkingWidget() {
  return <p>Widget loaded successfully.</p>;
}

// Route component
function DashboardRoute() {
  return (
    <div>
      <h2>Dashboard</h2>
      <ErrorBoundary FallbackComponent={FeatureError}>
        <BrokenChart />
      </ErrorBoundary>
      <ErrorBoundary FallbackComponent={FeatureError}>
        <WorkingWidget />
      </ErrorBoundary>
    </div>
  );
}

// App with layered boundaries
function App() {
  return (
    <ErrorBoundary FallbackComponent={GlobalError}>
      <header>App Header</header>
      <ErrorBoundary FallbackComponent={RouteError}>
        <DashboardRoute />
      </ErrorBoundary>
      <footer>App Footer</footer>
    </ErrorBoundary>
  );
}

export default App;
```

**Expected Output:** The app displays the header, "Dashboard", a feature-level error for the broken chart ("Widget failed: Chart service unavailable" with a "Retry Widget" button), and "Widget loaded successfully." The footer is also visible. The route-level and global boundaries are not triggered because the feature-level boundary caught the chart error.

**Why This Output Occurs:** The three-layer architecture provides defence in depth. The feature-level Error Boundary around `BrokenChart` catches its error first, rendering `FeatureError`. The route-level and global boundaries are not activated because the error was already caught. If the feature-level boundary itself failed, the route-level boundary would catch it. If that also failed, the global boundary would catch it. This layered approach ensures that errors are handled at the most appropriate level.

### Real-World Cases

- **SaaS dashboards:** Placing Error Boundaries around each widget (revenue chart, recent orders, team activity) so one failed widget doesn't take down the entire dashboard.
- **E-commerce checkout:** Isolating the payment form, shipping calculator, and order summary in separate boundaries.
- **Social media platforms:** Wrapping each post or story in an Error Boundary to isolate malformed content.
- **Multi-tenant applications:** Isolating tenant-specific widgets so one tenant's failure doesn't affect others.
- **Micro-frontend architectures:** Each micro-frontend has its own Error Boundary for independent failure isolation.
- **Third-party integrations:** Wrapping embedded widgets (maps, chat, payment) in Error Boundaries to isolate third-party failures.

---

## Core Concept 3: Fallback Interfaces

### Definitions

**Core Definition:** A Fallback Interface is the UI rendered by an Error Boundary when it catches an error, designed to be contextual, user-friendly, and appropriate for the level at which the error occurred.

**Technical Definition:** The fallback UI is determined by the Error Boundary's `render()` method when `hasError` is `true`, or by the `FallbackComponent` / `fallbackRender` props when using `react-error-boundary`. Fallback interfaces should be contextual: a card-level error should show an inline error message within the card, while a page-level error should show a full-page error screen. Fallbacks typically include an error icon, a user-friendly message (not a raw stack trace), and recovery options such as "Try Again" or "Go Home". In development, fallbacks may include stack traces; in production, they should be polished and branded.

**Beginner-Friendly Explanation:** When something goes wrong, you don't want to show users a scary technical error message or a blank screen. Instead, you show them a nice, friendly message that fits where the error happened. If a small widget breaks, show a small inline error. If an entire page breaks, show a full-page error with a "Try Again" button. The fallback UI is whatever you want it to be—the important thing is that it helps the user understand what happened and what they can do next.

### Purposes

- To provide a user-friendly error message instead of a blank screen or technical stack trace.
- To match the visual context of the error location (card-level vs. page-level).
- To guide users toward recovery actions (retry, navigate away, reload).
- To maintain the application's branding and visual consistency.
- To communicate the nature of the error without exposing sensitive information.
- To differentiate between development and production error displays.

### Syntax Rules and Structure

**General Syntax for Fallback Components:**
```jsx
// Minimal fallback for tight spaces (sidebar, card)
function MinimalError({ error, resetErrorBoundary }) {
  return (
    <div style={{ padding: '8px', textAlign: 'center' }}>
      <p style={{ color: '#888' }}>Something went wrong</p>
      <button onClick={resetErrorBoundary} style={{ textDecoration: 'underline' }}>
        Try again
      </button>
    </div>
  );
}

// Detailed fallback with icon for full-page errors
function DetailedError({ error, resetErrorBoundary }) {
  return (
    <div style={{ display: 'flex', flexDirection: 'column', alignItems: 'center', padding: '40px' }}>
      <AlertCircle size={48} color="red" />
      <h2>Something went wrong</h2>
      <p style={{ color: '#666' }}>{error.message}</p>
      <button onClick={resetErrorBoundary}>Try again</button>
    </div>
  );
}
```

**Component Breakdown:**
- `MinimalError`: For limited spaces (cards, sidebars). Shows a short message and a text-link style retry button.
- `DetailedError`: For full-page errors. Includes an icon, heading, error message, and styled retry button.
- `error`: The error object, containing `message` and `stack`.
- `resetErrorBoundary`: A function to reset the boundary and retry rendering.

**Syntax for Card-Based Error:**
```jsx
function CardError({ error, resetErrorBoundary }) {
  return (
    <div style={{
      margin: '0 auto',
      maxWidth: '400px',
      border: '1px solid rgba(255,0,0,0.5)',
      borderRadius: '8px',
      padding: '24px',
    }}>
      <div style={{ display: 'flex', alignItems: 'center', gap: '8px', marginBottom: '16px' }}>
        <AlertTriangle size={20} color="red" />
        <h3>Error</h3>
      </div>
      <p style={{ marginBottom: '16px', color: '#666' }}>{error.message}</p>
      <div style={{ display: 'flex', gap: '8px' }}>
        <button onClick={resetErrorBoundary}>Retry</button>
        <button onClick={() => (window.location.href = '/')}>Go home</button>
      </div>
    </div>
  );
}
```

**Component Breakdown:**
- Card-based layout: Constrained width, border, rounded corners, padding.
- Error icon (e.g., `AlertTriangle`) for visual context.
- Error message displayed in a muted color.
- Two recovery actions: "Retry" (resets the boundary) and "Go home" (navigates away).

**Syntax for Development vs Production Fallback:**
```jsx
function EnvironmentAwareError({ error, resetErrorBoundary }) {
  const isDevelopment = process.env.NODE_ENV === 'development';

  return (
    <div style={{ padding: '24px' }}>
      <h2 style={{ color: 'red' }}>Application Error</h2>
      <p style={{ color: '#666' }}>{error.message}</p>

      {isDevelopment && (
        <details style={{ marginBottom: '16px' }}>
          <summary style={{ cursor: 'pointer' }}>Stack trace</summary>
          <pre style={{
            marginTop: '8px',
            overflow: 'auto',
            background: '#f5f5f5',
            padding: '16px',
            fontSize: '12px',
          }}>
            {error.stack}
          </pre>
        </details>
      )}

      <button onClick={resetErrorBoundary}>Try again</button>
    </div>
  );
}
```

**Component Breakdown:**
- `isDevelopment`: Checks `process.env.NODE_ENV` to determine the environment.
- In development, a `<details>` element shows the stack trace for debugging.
- In production, the stack trace is hidden, and only the user-friendly message is shown.

**Syntax Rules:**
- Fallbacks should be contextual: card-level fallbacks for widgets, page-level fallbacks for routes.
- Use `error.message` for the user-facing message; avoid exposing `error.stack` in production.
- Include a recovery action (e.g., "Try Again", "Retry", "Go Home").
- Use a consistent visual language (brand colors, fonts, icons) across all fallbacks.
- For development, include stack traces and detailed error information; for production, hide technical details.
- Fallback components receive `error` and `resetErrorBoundary` props (with `react-error-boundary`).

**Constraints and Limitations:**
- Fallbacks cannot be async functions.
- Fallbacks should not throw errors themselves; if a fallback throws, the error propagates to the nearest parent Error Boundary.
- Fallbacks should be lightweight; heavy computations in fallbacks can delay rendering.
- The `error.message` may contain sensitive information; sanitise or filter it before displaying to users.
- Fallbacks do not have access to the component that threw the error; they only receive the error object.

### Annotated Code Examples

**Example 1: Card-Level Fallback for a Dashboard Widget**

```jsx
import React from 'react';
import { ErrorBoundary } from 'react-error-boundary';

// Card-level fallback for individual widgets
function CardError({ error, resetErrorBoundary }) {
  return (
    <div style={{
      border: '1px solid #e0e0e0',
      borderRadius: '8px',
      padding: '16px',
      backgroundColor: '#fafafa',
    }}>
      <div style={{
        display: 'flex',
        alignItems: 'center',
        gap: '8px',
        marginBottom: '12px',
      }}>
        <span style={{ fontSize: '20px' }}>⚠️</span>
        <h4 style={{ margin: 0 }}>Widget Error</h4>
      </div>
      <p style={{ color: '#666', fontSize: '14px' }}>
        {error.message}
      </p>
      <button
        onClick={resetErrorBoundary}
        style={{
          padding: '6px 12px',
          backgroundColor: '#007bff',
          color: 'white',
          border: 'none',
          borderRadius: '4px',
          cursor: 'pointer',
        }}
      >
        Retry
      </button>
    </div>
  );
}

// A widget that throws an error
function BrokenWidget() {
  throw new Error('Unable to load widget data');
}

function WorkingWidget() {
  return <p>Widget loaded successfully.</p>;
}

function Dashboard() {
  return (
    <div style={{ display: 'grid', gap: '16px', padding: '20px' }}>
      <h1>Dashboard</h1>

      {/* Card-level error boundary for the broken widget */}
      <ErrorBoundary FallbackComponent={CardError}>
        <BrokenWidget />
      </ErrorBoundary>

      {/* Another widget that works fine */}
      <WorkingWidget />
    </div>
  );
}

export default Dashboard;
```

**Expected Output:** The dashboard displays "Dashboard", a card with a "⚠️ Widget Error" heading, the message "Unable to load widget data", and a "Retry" button. Below it, "Widget loaded successfully." is displayed. The broken widget's error is contained to its card.

**Why This Output Occurs:** The `CardError` fallback is designed for card-level contexts: it has a bordered card layout, a small warning icon, a concise message, and a retry button. When `BrokenWidget` throws, the Error Boundary catches it and renders `CardError` instead. The `WorkingWidget` is outside the boundary and renders normally. This demonstrates how fallback design should match the context: a small, inline card for a widget error.

**Example 2: Page-Level Fallback with Branding**

```jsx
import React from 'react';
import { ErrorBoundary } from 'react-error-boundary';

function PageError({ error, resetErrorBoundary }) {
  return (
    <div style={{
      minHeight: '100vh',
      display: 'flex',
      flexDirection: 'column',
      alignItems: 'center',
      justifyContent: 'center',
      padding: '40px',
      textAlign: 'center',
      fontFamily: 'system-ui, sans-serif',
    }}>
      {/* Brand logo placeholder */}
      <div style={{
        fontSize: '24px',
        fontWeight: 'bold',
        marginBottom: '24px',
        color: '#333',
      }}>
        MyApp
      </div>

      {/* Error icon */}
      <div style={{ fontSize: '64px', marginBottom: '16px' }}>😵</div>

      <h1 style={{ marginBottom: '8px' }}>Something went wrong</h1>
      <p style={{ color: '#666', marginBottom: '24px', maxWidth: '400px' }}>
        We encountered an unexpected error. Please try again or contact support if the problem persists.
      </p>

      <div style={{ display: 'flex', gap: '12px' }}>
        <button
          onClick={resetErrorBoundary}
          style={{
            padding: '10px 24px',
            backgroundColor: '#007bff',
            color: 'white',
            border: 'none',
            borderRadius: '6px',
            cursor: 'pointer',
            fontSize: '16px',
          }}
        >
          Try Again
        </button>
        <button
          onClick={() => window.location.href = '/'}
          style={{
            padding: '10px 24px',
            backgroundColor: 'transparent',
            color: '#007bff',
            border: '1px solid #007bff',
            borderRadius: '6px',
            cursor: 'pointer',
            fontSize: '16px',
          }}
        >
          Go Home
        </button>
      </div>
    </div>
  );
}

function BrokenPage() {
  throw new Error('Critical page failure');
}

function App() {
  return (
    <ErrorBoundary FallbackComponent={PageError}>
      <BrokenPage />
    </ErrorBoundary>
  );
}

export default App;
```

**Expected Output:** The entire page is replaced by a centered error screen with the app's logo ("MyApp"), a distressed face emoji (😵), "Something went wrong" heading, a friendly explanation, and two buttons: "Try Again" and "Go Home".

**Why This Output Occurs:** The `PageError` fallback is designed for full-page errors: it takes the full viewport height, centers its content, includes branding, and provides two recovery options. When `BrokenPage` throws, the top-level Error Boundary catches it and renders `PageError`. This demonstrates how a page-level fallback should be more prominent and include branding compared to a card-level fallback.

### Real-World Cases

- **Card-level fallbacks:** For dashboard widgets, sidebar panels, and individual feed items.
- **Page-level fallbacks:** For route-level errors, full-page error screens with branding and navigation.
- **Form-level fallbacks:** For form submission errors, showing an inline error message with a retry button.
- **Section-level fallbacks:** For independent content sections (e.g., "Related Articles" on a blog post).
- **Modal-level fallbacks:** For errors within modals or dialogs, showing a compact error message.
- **Development fallbacks:** Showing stack traces and detailed error information for debugging.
- **Production fallbacks:** Showing polished, branded error screens with recovery options.

---

## Core Concept 4: Reset & Recovery Mechanisms

### Definitions

**Core Definition:** Reset & Recovery Mechanisms are the features that allow users to clear an error state and attempt to re-render the failed component without requiring a full page reload.

**Technical Definition:** A Reset & Recovery Mechanism is implemented by providing a function that resets the Error Boundary's state (`hasError: false`) and triggers a re-render of its children. In class-based Error Boundaries, this is done by adding a method that calls `this.setState({ hasError: false })`. In the `react-error-boundary` library, the `resetErrorBoundary` function is passed to the fallback component, and the `onReset` prop allows the parent component to reset its own state when the boundary is reset. The `resetKeys` prop automatically resets the boundary when specified values change, which is ideal for route parameter changes, user switching, and filter/search state changes. For errors related to data fetching, libraries like TanStack Query provide `QueryErrorResetBoundary` to reset error boundaries and retry failed queries.

**Beginner-Friendly Explanation:** When something goes wrong, users shouldn't be stuck on an error screen forever. Reset & Recovery Mechanisms give them a "Try Again" button. When they click it, the Error Boundary clears the error state and tries to render the component again. If the underlying issue is fixed (or was temporary), the component renders successfully. This is much better than asking users to refresh the entire page, which loses their context and state.

### Purposes

- To allow users to recover from errors without a full page reload.
- To clear the error state and attempt re-rendering of the failed component.
- To reset parent component state that may have caused the error.
- To automatically reset the boundary when specific dependencies change (e.g., route parameters).
- To integrate with data-fetching libraries for retrying failed queries.
- To provide a smooth recovery experience that preserves user context.

### Syntax Rules and Structure

**General Syntax for Class-Based Reset:**
```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    console.error('Error caught:', error, errorInfo);
  }

  // Reset method to clear the error state
  resetError = () => {
    this.setState({ hasError: false });
  };

  render() {
    if (this.state.hasError) {
      return (
        <div>
          <h2>Something went wrong.</h2>
          <button onClick={this.resetError}>Try Again</button>
        </div>
      );
    }
    return this.props.children;
  }
}
```

**Component Breakdown:**
- `resetError`: A method that sets `hasError` back to `false`.
- `<button onClick={this.resetError}>`: The "Try Again" button calls `resetError`.
- When `hasError` becomes `false`, `render()` returns `this.props.children` again, retrying the render.

**General Syntax with `react-error-boundary`:**
```jsx
import { ErrorBoundary } from 'react-error-boundary';

function ErrorFallback({ error, resetErrorBoundary }) {
  return (
    <div role="alert">
      <h2>Something went wrong</h2>
      <pre>{error.message}</pre>
      <button onClick={resetErrorBoundary}>Try again</button>
    </div>
  );
}

function App() {
  return (
    <ErrorBoundary FallbackComponent={ErrorFallback}>
      <MyComponent />
    </ErrorBoundary>
  );
}
```

**Component Breakdown:**
- `resetErrorBoundary`: A function provided by `react-error-boundary` that resets the boundary.
- `<button onClick={resetErrorBoundary}>`: Clicking the button resets the boundary and retries rendering.
- The `FallbackComponent` receives `resetErrorBoundary` as a prop.

**General Syntax with `onReset` and `resetKeys`:**
```jsx
function App() {
  const [userId, setUserId] = useState('123');

  return (
    <ErrorBoundary
      FallbackComponent={ErrorFallback}
      onReset={() => {
        // Reset any state that caused the error
        setUserId('123');
      }}
      resetKeys={[userId]}
    >
      <UserProfile userId={userId} />
    </ErrorBoundary>
  );
}
```

**Component Breakdown:**
- `onReset`: A callback invoked when the boundary resets. Used to reset parent state.
- `resetKeys`: An array of values. When any of these values change, the boundary automatically resets.
- `resetKeys={[userId]}`: If `userId` changes (e.g., due to navigation), the boundary resets automatically.

**General Syntax with TanStack Query `QueryErrorResetBoundary`:**
```jsx
import { QueryErrorResetBoundary } from '@tanstack/react-query';
import { ErrorBoundary } from 'react-error-boundary';

function App() {
  return (
    <QueryErrorResetBoundary>
      {({ reset }) => (
        <ErrorBoundary
          onReset={reset}
          fallbackRender={({ error, resetErrorBoundary }) => (
            <div>
              <p>Error: {error.message}</p>
              <button onClick={resetErrorBoundary}>Try again</button>
            </div>
          )}
        >
          <Page />
        </ErrorBoundary>
      )}
    </QueryErrorResetBoundary>
  );
}
```

**Component Breakdown:**
- `QueryErrorResetBoundary`: Provides a `reset` function that clears query error state.
- `onReset={reset}`: When the Error Boundary resets, it also resets the query error state.
- `resetErrorBoundary`: Resets the Error Boundary itself.
- This pattern is ideal for Suspense queries and React Error Boundaries.

**Syntax Rules:**
- Always provide a recovery mechanism (e.g., "Try Again" button) in the fallback UI.
- Use `onReset` to reset parent state that may have caused the error.
- Use `resetKeys` for automatic reset when dependencies change (route params, user ID, filters).
- For data-fetching errors, use `QueryErrorResetBoundary` from TanStack Query or a similar library mechanism.
- The `resetErrorBoundary` function should be called in an event handler, not during render.
- After resetting, React will attempt to re-render the children. If the error persists, the boundary will catch it again.

**Constraints and Limitations:**
- Resetting the boundary does not fix the underlying error; it only retries the render.
- If the error is caused by a persistent issue (e.g., a broken API), the error will occur again.
- `resetKeys` uses shallow comparison; objects and arrays should be memoised to avoid unnecessary resets.
- Resetting the boundary does not reset state in child components; use `onReset` to reset parent state.
- For `React.lazy` components, a failed dynamic import is permanently cached; you must create a new lazy component instance after a failure.

### Annotated Code Examples

**Example 1: Class-Based Error Boundary with Reset**

```jsx
import React from 'react';

class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    console.error('Error caught:', error, errorInfo);
  }

  // Reset the error state
  resetError = () => {
    this.setState({ hasError: false, error: null });
  };

  render() {
    if (this.state.hasError) {
      return (
        <div style={{ padding: '20px', border: '1px solid red' }}>
          <h2>Something went wrong</h2>
          <p>{this.state.error.message}</p>
          <button onClick={this.resetError}>Try Again</button>
        </div>
      );
    }
    return this.props.children;
  }
}

// A component that throws based on a prop
function Counter({ count }) {
  if (count > 3) {
    throw new Error('Count too high!');
  }
  return <p>Count: {count}</p>;
}

function App() {
  const [count, setCount] = React.useState(0);
  const [resetKey, setResetKey] = React.useState(0);

  return (
    <div>
      <h1>Reset Demo</h1>
      <button onClick={() => setCount(c => c + 1)}>Increment</button>
      <button onClick={() => setCount(0)}>Reset Count</button>

      <ErrorBoundary key={resetKey}>
        <Counter count={count} />
      </ErrorBoundary>

      <button onClick={() => setResetKey(k => k + 1)}>
        Force Reset Boundary
      </button>
    </div>
  );
}

export default App;
```

**Expected Output:** The app displays a counter. Clicking "Increment" up to 3 works. At 4, the Error Boundary shows "Something went wrong" with "Count too high!" and a "Try Again" button. Clicking "Try Again" resets the boundary, but the error occurs again because `count` is still 4. Clicking "Reset Count" sets `count` to 0, and the component renders successfully. Clicking "Force Reset Boundary" changes the `key`, remounting the boundary and resetting its state.

**Why This Output Occurs:** The `resetError` method sets `hasError` to `false`, causing the boundary to retry rendering `Counter`. If `count` is still 4, `Counter` throws again, and the boundary catches it. The "Reset Count" button fixes the underlying issue by setting `count` to 0. The `key={resetKey}` prop on the Error Boundary forces React to remount it when `resetKey` changes, completely resetting its state.

**Example 2: Automatic Reset with `resetKeys`**

```jsx
import React, { useState } from 'react';
import { ErrorBoundary } from 'react-error-boundary';

function ErrorFallback({ error, resetErrorBoundary }) {
  return (
    <div role="alert" style={{ padding: '20px', border: '1px solid red' }}>
      <h2>Error loading profile</h2>
      <p>{error.message}</p>
      <button onClick={resetErrorBoundary}>Try again</button>
    </div>
  );
}

// Simulates a component that throws for certain userIds
function UserProfile({ userId }) {
  if (userId === 'error') {
    throw new Error(`Failed to load user: ${userId}`);
  }
  return <p>Profile for user: {userId}</p>;
}

function App() {
  const [userId, setUserId] = useState('123');

  return (
    <div>
      <h1>Reset Keys Demo</h1>

      <div>
        <button onClick={() => setUserId('123')}>User 123</button>
        <button onClick={() => setUserId('456')}>User 456</button>
        <button onClick={() => setUserId('error')}>Error User</button>
      </div>

      <ErrorBoundary
        FallbackComponent={ErrorFallback}
        resetKeys={[userId]}
        onReset={() => {
          // Optional: reset any parent state
          console.log('Boundary reset for userId:', userId);
        }}
      >
        <UserProfile userId={userId} />
      </ErrorBoundary>
    </div>
  );
}

export default App;
```

**Expected Output:** The app starts with "Profile for user: 123". Clicking "User 456" shows "Profile for user: 456". Clicking "Error User" shows the Error Boundary fallback with "Failed to load user: error" and a "Try again" button. Clicking "User 123" or "User 456" automatically resets the boundary (because `userId` changed via `resetKeys`) and displays the corresponding profile without needing to click "Try again".

**Why This Output Occurs:** The `resetKeys={[userId]}` prop tells the Error Boundary to automatically reset whenever `userId` changes. When the user clicks "Error User", the boundary catches the error and shows the fallback. When the user then clicks "User 123", `userId` changes, the boundary resets automatically, and `UserProfile` renders successfully. This pattern is ideal for route parameter changes, user switching, and filter/search state changes.

### Real-World Cases

- **Route parameter changes:** Automatically resetting the boundary when navigating to a different route or resource.
- **User switching:** Resetting error state when the logged-in user changes.
- **Search/filter changes:** Resetting the boundary when search terms or filters change.
- **Data refetching:** Integrating with TanStack Query's `QueryErrorResetBoundary` to retry failed queries.
- **Form submissions:** Providing a "Try Again" button that resets the form and resubmits.
- **Network recovery:** Offering a retry option when network errors occur.
- **Third-party widget failures:** Providing a reload button for embedded widgets that fail to initialise.

---

## References

- Error Boundaries – React: https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary
- Error Boundaries (Legacy) – React: https://legacy.reactjs.org/docs/error-boundaries.html
- Error Boundaries – React (Mintlify): https://mintlify.wiki/facebook/react/advanced/error-boundaries
- react-error-boundary Library: https://github.com/bvaughn/react-error-boundary
- react-error-boundary Documentation: https://react-error-boundary-lib.vercel.app/
- QueryErrorResetBoundary – TanStack Query: https://tanstack.com/query/v5/docs/framework/react/api-reference/QueryErrorResetBoundary
- react-crash-guard – npm: https://www.npmjs.com/package/react-crash-guard
- Data-Granular Error Boundaries – GitHub: https://github.com/pproenca/dot-skills/blob/HEAD/skills/.experimental/react-refactor/references/data-granular-error-boundaries.md
- Fallback UI Patterns – GitHub: https://github.com/oakoss/agent-skills/blob/main/skills/react-error-handling/references/fallback-patterns.md
- React Error Boundary Library Reference – GitHub: https://github.com/oakoss/agent-skills/blob/main/skills/react-error-handling/references/react-error-boundary.md
- How Can I Make Use of Error Boundaries in Functional React Components? – Stack Overflow: https://stackoverflow.com/questions/59821605/how-can-i-make-use-of-error-boundaries-in-functional-react-components
- Why React Error Boundaries Aren't Just Try/Catch for Components – Epic React: https://www.epicreact.dev/why-react-error-boundaries-arent-just-try-catch-for-components
- Reset Error Boundary – Epic React: https://www.epicreact.dev/reset
- Next.js – Error Handling: https://nextjs.org/docs/app/building-your-application/routing/error-handling