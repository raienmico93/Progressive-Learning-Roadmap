# React Code Splitting: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** React code splitting is the practice of breaking a JavaScript bundle into smaller chunks that are loaded on demand—rather than all at once—so that users download only the code they need for the current view, reducing initial load time and improving Time to Interactive (TTI).

**Technical Definition:** Code splitting is a bundler-level optimisation enabled by dynamic `import()` expressions, which JavaScript treats as split points. Bundlers (Webpack, Rollup, Vite, esbuild, Turbopack) detect `import()` calls and emit separate chunks that are fetched at runtime via the browser's dynamic module loading. React integrates with this through `React.lazy()` (which accepts a function returning a `Promise<{ default: Component }>`) and `<Suspense>` (which renders a fallback UI while the lazy component's chunk is loading). React 19 adds framework-level integration via Server Components, where `React.lazy` is not supported but Server Components and route-level lazy loading (e.g., Next.js `next/dynamic`, React Router `lazy`) provide the same effect. Code splitting is orthogonal to tree shaking: tree shaking removes unused exports from a bundle, while code splitting defers used code that is not needed immediately.

**Beginner-Friendly Explanation:** Imagine your app is a book. Without code splitting, users must download the entire book before reading page one. With code splitting, the book is divided into chapters, and users download only the chapter they are reading. When they navigate to the next chapter, that chapter downloads on demand. The first page appears much faster, and users never wait for content they may not visit. React's `React.lazy` and `<Suspense>` are the tools that make this possible at the component level.

### Key Characteristics

- **Dynamic `import()` is the Foundation:** Every code splitting strategy ultimately relies on JavaScript's dynamic `import()` syntax, which returns a `Promise`.
- **Bundler-Enforced Split Points:** Webpack, Rollup, Vite, and Turbopack treat `import()` expressions as split points and emit separate chunks.
- **`React.lazy` Wraps `import()`:** `React.lazy(() => import('./Component'))` returns a component that suspends while its chunk loads.
- **`<Suspense>` Provides the Fallback:** Every lazy component must be wrapped in a `<Suspense>` boundary that renders a fallback UI (skeleton, spinner) while loading.
- **Nesting Boundaries Improves UX:** Multiple `<Suspense>` boundaries let each section of the page load and stream independently, avoiding a single blocking spinner.
- **Route-Based is the Most Common Strategy:** Most applications split at route boundaries because routes are natural, infrequent navigation points.
- **Preloading Recovers the Cost:** Splitting adds a network round trip; preloading on hover, viewport entry, or idle time recovers perceived performance.
- **SSR Requires Care:** `React.lazy` does not work in Server Components; frameworks use `next/dynamic`, route `lazy` properties, or RSC streaming instead.

### Prerequisites

- Solid understanding of JavaScript modules (`import`/`export`), promises, and `async`/`await`.
- Familiarity with React function components, JSX, and the `useState` Hook.
- Working knowledge of `React.lazy`, `<Suspense>`, and Error Boundaries.
- Awareness of a bundler (Vite, Webpack, Next.js) and how it emits chunks.
- Basic understanding of HTTP caching, `Cache-Control`, and network waterfalls.

### Related Programming Areas

- **Bundling:** Webpack, Rollup, Vite, esbuild, Turbopack.
- **Performance:** Initial load time, Time to Interactive (TTI), Largest Contentful Paint (LCP).
- **Suspense:** Fallback UIs, nested boundaries, streaming SSR.
- **React Server Components:** Framework-level code splitting and streaming.
- **Prefetching:** Idle-time, hover, and viewport-based preloading.

### Core Concepts / Features

1. Dynamic Imports (`import()`)
2. Component Lazy Loading (`React.lazy`)
3. Suspense Architectures
4. Granular Splitting Topologies

---

## Core Concept 1: Dynamic Imports

### Definitions

**Core Definition:** A dynamic import is a JavaScript expression (`import('./module')`) that loads a module asynchronously at runtime and returns a `Promise` resolving to the module's namespace object.

**Technical Definition:** The dynamic `import()` syntax (ECMAScript 2020, previously stage 3) differs from static `import` in three ways: (1) it can appear anywhere in the code (not only at the top level); (2) it returns a `Promise<Module>` instead of being hoisted and evaluated at module load; and (3) bundlers treat it as a **split point**, emitting a separate chunk for the imported module and its dependencies. The browser fetches the chunk via a `<script type="module">` equivalent (`import()` is natively supported in all modern browsers), and the module is evaluated once and cached for subsequent `import()` calls. Bundlers can also use "magic comments" (Webpack) or naming conventions (Vite, Rollup) to control the chunk's file name and preload behaviour.

**Beginner-Friendly Explanation:** A static import is like ordering every dish on the menu at once—you get everything, whether you want it or not. A dynamic import is like ordering one dish at a time—you get exactly what you need, when you need it. The bundler sees `import()` and says: "This module is separate. Put it in its own file, and fetch it only when this line runs."

### Purposes

- To load a module only when it is needed, reducing the initial bundle size.
- To create a split point that bundlers use to emit a separate chunk.
- To defer the download of heavy dependencies (chart libraries, editors, maps).
- To conditionally load modules based on runtime values (feature flags, locale, platform).
- To enable React's `React.lazy` and route-based lazy loading.

### Syntax Rules and Structure

**General Syntax:**
```javascript
const module = await import('./module.js');
module.doSomething();
```

**Destructured Import:**
```javascript
const { doSomething, doSomethingElse } = await import('./module.js');
```

**Inside a Component (Returning a Promise):**
```javascript
function loadChart() {
  return import('./HeavyChart.js').then((module) => module.default);
}
```

**Component Breakdown:**
- `import('./module.js')`: Returns a `Promise<Module>`.
- `.then((module) => module.default)`: Accesses the default export.
- `await`: Pauses execution until the module is loaded.

**Webpack Magic Comments:**
```javascript
// Named chunk
import(/* webpackChunkName: "heavy-chart" */ './HeavyChart');

// Prefetch (load during idle time)
import(/* webpackPrefetch: true */ './FutureView');

// Preload (load in parallel with parent chunk)
import(/* webpackPreload: true */ './CriticalModule');

// Exclude from bundle (runtime-only, e.g., CDN module)
import(/* webpackIgnore: true */ 'https://cdn.example.com/lib.js');
```

**Component Breakdown:**
- `webpackChunkName`: Names the emitted chunk for debugging.
- `webpackPrefetch`: Adds a `<link rel="prefetch">` hint; loads when the browser is idle.
- `webpackPreload`: Adds a `<link rel="preload">` hint; loads in parallel with the parent.
- `webpackIgnore`: Leaves the import as a runtime `import()`, not a bundler split point.

**Vite/Rollup Chunk Naming (Rollup Output):**
```javascript
// vite.config.js
export default {
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          'vendor-chart': ['chart.js', 'react-chartjs-2'],
          'vendor-editor': ['@codemirror/state', '@codemirror/view'],
        },
      },
    },
  },
};
```

**Component Breakdown:**
- `manualChunks`: Groups modules into named chunks manually.
- Useful when multiple lazy components share heavy dependencies.

**Syntax Rules:**
- Use `import()` wherever you need to load code conditionally or on demand.
- Always handle the returned promise (`.then()` or `await`); unhandled rejections cause runtime errors.
- Use `webpackChunkName` (Webpack) or `manualChunks` (Vite/Rollup) to control chunk names.
- Use `webpackPrefetch` for future navigation; use `webpackPreload` for modules needed immediately after the parent.
- Dynamic imports work in all modern browsers (native ESM); older browsers need a polyfill.
- Bundlers treat each `import()` as a separate chunk by default; shared dependencies may be hoisted into a shared chunk.
- Do not use dynamic imports for modules that are always needed; that defeats the purpose.

**Constraints and Limitations:**
- Dynamic imports add a network round trip at runtime; preload critical chunks.
- Dynamic imports cannot be tree-shaken as aggressively as static imports.
- A dynamic import inside a function is evaluated every time the function runs (though the module itself is cached).
- Bundlers cannot statically analyse the imported path if it is fully dynamic (e.g., `import(\`./locales/${locale}.js\`)`); use a pattern that the bundler can enumerate.
- Server-side rendering requires bundler/framework support; `import()` does not work in RSC.

### Annotated Code Examples

**Example 1: Basic Dynamic Import with Error Handling**

```javascript
async function loadEditor() {
  try {
    const { Editor } = await import('./editor.js');
    return new Editor();
  } catch (error) {
    console.error('Failed to load editor:', error);
    throw error;
  }
}

document.getElementById('open-editor').addEventListener('click', async () => {
  const editor = await loadEditor();
  editor.mount('#editor-root');
});
```

**Expected Output:** Clicking the "Open Editor" button downloads `editor.js` (and its dependencies) as a separate chunk, then instantiates the editor. If the chunk fails to load (network error, 404), the error is logged.

**Why This Output Occurs:** `import('./editor.js')` instructs the bundler to emit `editor.js` as a separate chunk. At runtime, the browser fetches the chunk only when the button is clicked. The `try/catch` handles network failures gracefully.

**Example 2: Conditional Import Based on Feature Flag**

```javascript
async function loadAnalytics() {
  const { enabled } = await fetch('/api/features').then(r => r.json());
  if (!enabled) return null;

  const { track } = await import('./analytics.js');
  return track;
}

loadAnalytics().then((track) => {
  if (track) track('page_view');
});
```

**Expected Output:** The analytics module is downloaded only if the feature flag is enabled. If disabled, the module is never fetched.

**Why This Output Occurs:** The `import()` is inside a conditional block, so the bundler emits `analytics.js` as a separate chunk that is fetched only when the condition is true.

### Real-World Cases

- **Rich text editors:** Load CodeMirror or TipTap only when the user opens the editor.
- **Chart libraries:** Load Chart.js or D3 only when a chart is rendered.
- **Maps:** Load Leaflet or Mapbox only when the map section enters the viewport.
- **Localisation:** Load locale JSON files based on the user's language.
- **Feature flags:** Load experimental features only when enabled.

### References

- MDN Web Docs – `import()`: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/import
- TC39 – Dynamic Import Proposal: https://github.com/tc39/proposal-dynamic-import
- Webpack – Dynamic Imports: https://webpack.js.org/guides/code-splitting/#dynamic-imports
- Webpack – Magic Comments: https://webpack.js.org/api/module-methods/#magic-comments
- Vite – Dynamic Import: https://vitejs.dev/guide/features.html#dynamic-import
- Rollup – Code Splitting: https://rollupjs.org/tutorial/#code-splitting

---

## Core Concept 2: Component Lazy Loading

### Definitions

**Core Definition:** `React.lazy()` is a React API that accepts a function returning a promise resolving to a module with a `default` export, and returns a component that suspends while the module is loading.

**Technical Definition:** `React.lazy(load)` takes a function `load` that returns a `Promise<{ default: Component }>` (or, in React 19, a `Promise<{ default: Component, [key]: any }>` for named exports). It returns a `LazyExoticComponent` that suspends the first time it is rendered, triggering the nearest `<Suspense>` boundary's fallback. When the promise resolves, React retries rendering the component with the resolved module. The `load` function must be defined outside the component to avoid re-creating it on every render (which would cause an infinite suspend loop). React 19 removed the "lazy initialiser" restriction for `forwardRef` components and added support for `React.lazy` in Server Components via framework integration (Next.js `next/dynamic`, React Router `lazy`). `React.lazy` is compatible with `React.memo`, `forwardRef`, and `React.Suspense`.

**Beginner-Friendly Explanation:** `React.lazy` is like a bookshelf with a secret compartment. When you ask for the book inside, React says "wait here" and shows a placeholder (the `<Suspense>` fallback) while it fetches the book. When the book arrives, React swaps out the placeholder for the real book. You only fetch the book when someone asks for it, not when the shelf is built.

### Purposes

- To defer the download of a component's code until it is actually rendered.
- To reduce the initial bundle size by excluding rarely-used components.
- To integrate dynamic imports with React's rendering model.
- To combine with `<Suspense>` for declarative loading states.
- To enable route-based, component-based, and interaction-based splitting.

### Syntax Rules and Structure

**General Syntax:**
```jsx
const LazyComponent = React.lazy(() => import('./Component'));

function App() {
  return (
    <Suspense fallback={<Spinner />}>
      <LazyComponent />
    </Suspense>
  );
}
```

**Component Breakdown:**
- `React.lazy(() => import('./Component'))`: Creates a lazy component from a dynamic import.
- The `load` function returns a `Promise<{ default: Component }>`.
- `<Suspense fallback={...}>`: Renders the fallback while the lazy component loads.
- The lazy component's module must have a `default` export.

**Named Export Wrapper:**
```jsx
const LazyChart = React.lazy(() =>
  import('./Chart').then((module) => ({ default: module.Chart }))
);
```

**Component Breakdown:**
- Named exports require a `.then()` wrapper to map the named export to `default`.
- React 19 accepts `Promise<{ default: Component, [key]: any }>` directly for named exports in some frameworks.

**Lazy Component with Props:**
```jsx
const LazyModal = React.lazy(() => import('./Modal'));

function App() {
  const [isOpen, setIsOpen] = useState(false);
  return (
    <>
      <button onClick={() => setIsOpen(true)}>Open Modal</button>
      {isOpen && (
        <Suspense fallback={<ModalSkeleton />}>
          <LazyModal title="Confirm" onClose={() => setIsOpen(false)} />
        </Suspense>
      )}
    </>
  );
}
```

**Component Breakdown:**
- The lazy component accepts props like any normal component.
- Conditional rendering (`isOpen && ...`) delays the import until the modal is first opened.

**Error Boundary for Lazy Components:**
```jsx
class LazyErrorBoundary extends React.Component {
  state = { hasError: false };
  static getDerivedStateFromError() { return { hasError: true }; }
  render() {
    if (this.state.hasError) {
      return <button onClick={() => window.location.reload()}>Retry</button>;
    }
    return this.props.children;
  }
}

function App() {
  return (
    <LazyErrorBoundary>
      <Suspense fallback={<Spinner />}>
        <LazyComponent />
      </Suspense>
    </LazyErrorBoundary>
  );
}
```

**Component Breakdown:**
- Error Boundaries catch rejected promises from lazy components (network failures).
- The fallback includes a retry button that reloads the page.

**Syntax Rules:**
- Always wrap lazy components in `<Suspense>` (and optionally an Error Boundary).
- Define the `load` function outside the component so its reference is stable.
- For named exports, use `.then((module) => ({ default: module.Named }))`.
- Do not use `React.lazy` in Server Components; use framework-specific lazy loading.
- Use `React.memo(React.lazy(...))` only if the lazy component itself needs memoization (rare).
- Lazy components work with `forwardRef` and `memo` in React 19 without special handling.

**Constraints and Limitations:**
- `React.lazy` does not work in Server Components (RSC); use framework lazy loading.
- The `load` function must return a promise resolving to a module with a `default` export.
- Lazy components cannot be used for named exports without a wrapper (unless using React 19's named export support).
- Lazy components add a network waterfall: the parent chunk loads, then the lazy chunk loads.
- Error Boundaries are required to handle chunk load failures gracefully.

### Annotated Code Examples

**Example 1: Lazy Modal with Conditional Rendering**

```jsx
import { lazy, Suspense, useState, Component } from 'react';

const LazyHeavyModal = lazy(() => import('./HeavyModal'));

class ModalErrorBoundary extends Component {
  state = { hasError: false };
  static getDerivedStateFromError() { return { hasError: true }; }
  render() {
    if (this.state.hasError) {
      return <p role="alert">Failed to load modal. Please refresh.</p>;
    }
    return this.props.children;
  }
}

export default function App() {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <div>
      <button onClick={() => setIsOpen(true)}>Open Modal</button>
      {isOpen && (
        <ModalErrorBoundary>
          <Suspense fallback={<div>Loading modal...</div>}>
            <LazyHeavyModal onClose={() => setIsOpen(false)} />
          </Suspense>
        </ModalErrorBoundary>
      )}
    </div>
  );
}
```

**Expected Output:** Clicking "Open Modal" downloads the modal chunk (visible in the Network tab) and shows "Loading modal..." until the chunk is ready. Then the modal renders. If the chunk fails to load, the error boundary shows "Failed to load modal. Please refresh."

**Why This Output Occurs:** The `import('./HeavyModal')` is a split point; the modal chunk is fetched only when `isOpen` becomes `true`. `<Suspense>` shows the fallback while the chunk loads. The Error Boundary catches network failures.

**Example 2: Lazy Component with Named Export**

```jsx
import { lazy, Suspense } from 'react';

// Chart.js module exports `LineChart` as a named export
const LazyLineChart = lazy(() =>
  import('./charts').then((module) => ({ default: module.LineChart }))
);

export default function Dashboard() {
  return (
    <Suspense fallback={<div>Loading chart...</div>}>
      <LazyLineChart data={[1, 2, 3]} />
    </Suspense>
  );
}
```

**Expected Output:** The chart module is loaded on demand and rendered once ready. The fallback shows while loading.

**Why This Output Occurs:** The `.then()` maps the named export `LineChart` to `default`, satisfying `React.lazy`'s requirement.

### Real-World Cases

- **Route components:** Each route is a lazy component loaded when the user navigates to it.
- **Modals and drawers:** Loaded only when the user opens them.
- **Heavy editors:** Loaded only when the user clicks "Edit".
- **Charts and maps:** Loaded only when the section enters the viewport.
- **Admin panels:** Loaded only for users with admin permissions.

### References

- React Official Documentation – `lazy`: https://react.dev/reference/react/lazy
- React Official Documentation – `<Suspense>`: https://react.dev/reference/react/Suspense
- React Official Documentation – Code Splitting: https://legacy.reactjs.org/docs/code-splitting.html
- React 19 – `lazy` Named Exports: https://react.dev/blog/2024/12/05/react-19
- Webpack – Code Splitting with React: https://webpack.js.org/guides/code-splitting/#react-lazy

---

## Core Concept 3: Suspense Architectures

### Definitions

**Core Definition:** A Suspense architecture is the deliberate arrangement of `<Suspense>` boundaries—their placement, nesting, and fallback design—that determines how a React application streams content, handles loading states, and recovers from errors.

**Technical Definition:** `<Suspense>` is a React component that catches a "suspension" (a thrown promise) from any of its descendants and renders a fallback until the promise resolves. When the promise resolves, React retries rendering the suspended subtree. Multiple `<Suspense>` boundaries can be nested, each with its own fallback; a suspension is caught by the *nearest* boundary above it. In React 18+, Suspense supports streaming SSR: the server sends the HTML shell immediately and streams in the resolved content as it becomes available. React 19 extends this with `<Suspense>` around async Server Components and the `use` hook. A well-designed Suspense architecture places boundaries at meaningful UI seams (route, section, widget) rather than wrapping everything in one boundary, so fast content appears immediately while slow content streams in.

**Beginner-Friendly Explanation:** A `<Suspense>` boundary is like a curtain that says "come back later" while something is being fetched. If you have one curtain for the whole page, the user sees nothing until everything is ready. If you have several curtains—one for the header, one for the main content, one for the sidebar—the user sees each section as soon as it is ready. The art of Suspense architecture is deciding where to put the curtains so the page feels fast.

### Purposes

- To show a fallback UI while a lazy component or data is loading.
- To stream content progressively rather than blocking on the slowest resource.
- To isolate slow sections so fast sections render immediately.
- To provide error recovery via Error Boundaries nested with Suspense.
- To enable streaming SSR in React 18+ and Server Components in React 19.

### Syntax Rules and Structure

**Single Boundary (Simple):**
```jsx
<Suspense fallback={<PageSpinner />}>
  <LazyRoute />
</Suspense>
```

**Nested Boundaries (Streaming):**
```jsx
<Suspense fallback={<HeaderSkeleton />}>
  <Header />
</Suspense>

<Suspense fallback={<MainSkeleton />}>
  <main>
    <Suspense fallback={<ChartSkeleton />}>
      <LazyChart />
    </Suspense>
    <Suspense fallback={<TableSkeleton />}>
      <LazyTable />
    </Suspense>
  </main>
</Suspense>
```

**Component Breakdown:**
- The outer boundary wraps the whole main section; the inner boundaries wrap individual widgets.
- Each widget streams in independently; the chart and table do not block each other.
- The header loads and renders independently of the main content.

**Route-Level Suspense:**
```jsx
import { lazy, Suspense } from 'react';
import { Routes, Route } from 'react-router';

const Home = lazy(() => import('./routes/Home'));
const Products = lazy(() => import('./routes/Products'));
const Checkout = lazy(() => import('./routes/Checkout'));

function App() {
  return (
    <Suspense fallback={<AppSkeleton />}>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/products" element={<Products />} />
        <Route path="/checkout" element={<Checkout />} />
      </Routes>
    </Suspense>
  );
}
```

**Component Breakdown:**
- One Suspense boundary wraps all routes; navigating to any route shows the same fallback.
- For per-route fallbacks, wrap each `<Route element>` in its own `<Suspense>`.

**Suspense with Error Boundary:**
```jsx
<ErrorBoundary fallback={<ErrorPage />}>
  <Suspense fallback={<Skeleton />}>
    <LazyComponent />
  </Suspense>
</ErrorBoundary>
```

**Component Breakdown:**
- The Error Boundary catches rejections (network failures, component errors).
- The Suspense boundary catches suspensions (loading states).
- Both are required for a robust lazy-loading experience.

**Syntax Rules:**
- Place Suspense boundaries at meaningful UI seams (route, section, widget).
- Use skeletons that match the final content's dimensions to avoid layout shift.
- Nest boundaries to stream content progressively.
- Always pair Suspense with an Error Boundary for lazy components.
- Use `SuspenseList` (experimental) to coordinate multiple Suspense boundaries (reveal order, tail).
- Avoid a single top-level Suspense boundary for the entire app; it defeats streaming.
- In React 18+, Suspense supports streaming SSR; in React 19, it supports async Server Components.

**Constraints and Limitations:**
- Suspense only catches promises thrown by `React.lazy` or Suspense-enabled data sources (Relay, TanStack Query's `useSuspenseQuery`, React 19's `use`).
- It does not catch errors; an Error Boundary is required.
- A single top-level Suspense boundary causes the whole page to block on the slowest resource.
- Fallbacks that do not match the final layout cause Cumulative Layout Shift (CLS).
- `SuspenseList` is experimental and not recommended for production.

### Annotated Code Examples

**Example 1: Nested Suspense for a Dashboard**

```jsx
import { lazy, Suspense } from 'react';

const RevenueChart = lazy(() => import('./widgets/RevenueChart'));
const UserTable = lazy(() => import('./widgets/UserTable'));
const Notifications = lazy(() => import('./widgets/Notifications'));

function Skeleton({ height = 200 }) {
  return <div style={{ height, background: '#eee', borderRadius: 8 }} aria-hidden="true" />;
}

export default function Dashboard() {
  return (
    <div>
      <h1>Dashboard</h1>

      <section>
        <h2>Revenue</h2>
        <Suspense fallback={<Skeleton height={300} />}>
          <RevenueChart />
        </Suspense>
      </section>

      <section>
        <h2>Users</h2>
        <Suspense fallback={<Skeleton height={400} />}>
          <UserTable />
        </Suspense>
      </section>

      <section>
        <h2>Notifications</h2>
        <Suspense fallback={<Skeleton height={100} />}>
          <Notifications />
        </Suspense>
      </section>
    </div>
  );
}
```

**Expected Output:** The dashboard renders immediately with headings and skeletons. Each widget's chunk loads independently; as each resolves, its skeleton is replaced by the real content. A slow chart does not block the user table.

**Why This Output Occurs:** Each widget is wrapped in its own `<Suspense>` boundary. React does not wait for all widgets to load; it streams each one as its chunk resolves. The skeleton heights match the final content to minimise layout shift.

**Example 2: Route-Level Suspense with Per-Route Fallbacks**

```jsx
import { lazy, Suspense } from 'react';
import { Routes, Route } from 'react-router';

const Home = lazy(() => import('./routes/Home'));
const Products = lazy(() => import('./routes/Products'));

function RouteFallback({ label }) {
  return <div role="status">Loading {label}...</div>;
}

function App() {
  return (
    <Routes>
      <Route
        path="/"
        element={
          <Suspense fallback={<RouteFallback label="Home" />}>
            <Home />
          </Suspense>
        }
      />
      <Route
        path="/products"
        element={
          <Suspense fallback={<RouteFallback label="Products" />}>
            <Products />
          </Suspense>
        }
      />
    </Routes>
  );
}
```

**Expected Output:** Navigating to `/` shows "Loading Home..." while the Home chunk loads. Navigating to `/products` shows "Loading Products...". Each route has its own fallback.

**Why This Output Occurs:** Each `<Route>` element is wrapped in its own `<Suspense>` boundary, so the fallback is route-specific. React only suspends the component for the active route.

### Real-World Cases

- **Dashboards:** Each widget wrapped in its own Suspense boundary with a skeleton.
- **E-commerce:** Product grid, reviews, and recommendations load independently.
- **Social feeds:** Feed, sidebar, and notifications stream in separately.
- **Documentation:** Article content, table of contents, and code samples load independently.
- **Streaming SSR:** The server sends the shell immediately and streams widgets as they resolve.

### References

- React Official Documentation – `<Suspense>`: https://react.dev/reference/react/Suspense
- React Official Documentation – Suspense in React 18: https://react.dev/blog/2022/03/29/react-v18#suspense-in-data-frameworks
- React Official Documentation – Streaming SSR: https://react.dev/reference/react-dom/server/renderToPipeableStream
- React 19 – Suspense and Server Components: https://react.dev/blog/2024/12/05/react-19
- Next.js – Loading UI and Streaming: https://nextjs.org/docs/app/building-your-application/routing/loading-ui-and-streaming

---

## Core Concept 4: Granular Splitting Topologies

### Definitions

**Core Definition:** Granular splitting topologies are the strategies for deciding *where* to split a bundle—by route, by component, by user interaction, or by deferred preloading—each trading off initial bundle size against runtime network requests.

**Technical Definition:** Code splitting topologies can be categorised along two axes: **granularity** (route, feature, component, widget) and **timing** (immediate, on-interaction, on-idle, on-prefetch). The four canonical topologies are: (1) **Route-based splitting** — each route is a separate chunk, loaded on navigation; (2) **Component-based splitting** — individual heavy components (charts, editors, maps) are split regardless of route; (3) **Interaction-based splitting** — chunks are loaded in response to a user action (click, hover, focus); (4) **Deferred preloading** — chunks are prefetched during idle time, on viewport entry, or on hover, so they are ready when the user needs them. Modern frameworks (Next.js, React Router v7) provide route-level lazy loading and preloading out of the box. The goal is to keep the critical path small while ensuring that subsequent navigations and interactions feel instant.

**Beginner-Friendly Explanation:** Splitting is like deciding which books to keep on your desk and which to keep in the library. Route-based splitting is: "one chapter per shelf." Component-based splitting is: "the heavy reference book goes in the library; the dictionary stays on the desk." Interaction-based splitting is: "fetch the recipe book only when someone starts cooking." Preloading is: "while you are reading, quietly fetch the next chapter so it is ready when you turn the page."

### Purposes

- **Route-based:** To keep the initial bundle minimal and load each page on demand.
- **Component-based:** To defer heavy components (charts, editors) regardless of route.
- **Interaction-based:** To load code only when the user expresses intent (hover, click).
- **Deferred preloading:** To eliminate the perceived cost of splitting by fetching chunks ahead of time.
- **Combined:** To layer strategies for the best balance of initial size and perceived speed.

### Syntax Rules and Structure

**Route-Based Splitting (React Router v7):**
```jsx
// app/routes.ts
import { route } from '@react-router/dev/routes';

export default [
  route('dashboard', 'routes/dashboard.tsx'),
  route('settings', 'routes/settings.tsx'),
  route('reports', 'routes/reports.tsx'),
];
```

```tsx
// routes/dashboard.tsx
export async function loader() {
  return fetchDashboardData();
}

export function Component() {
  const data = useLoaderData();
  return <Dashboard data={data} />;
}
```

**Component Breakdown:**
- Each route is a separate module; React Router loads it on navigation.
- The loader and component are in the same module, so they load together.

**Component-Based Splitting (Lazy Heavy Component):**
```jsx
import { lazy, Suspense } from 'react';

const HeavyChart = lazy(() => import('./HeavyChart'));

function Analytics({ showChart }) {
  return (
    <div>
      <h2>Analytics</h2>
      {showChart && (
        <Suspense fallback={<ChartSkeleton />}>
          <HeavyChart />
        </Suspense>
      )}
    </div>
  );
}
```

**Component Breakdown:**
- `HeavyChart` is split regardless of route; it loads when `showChart` is true.
- Useful for charts, editors, maps, and other heavy widgets.

**Interaction-Based Splitting (Load on Hover):**
```jsx
function ProductLink({ productId }) {
  const [showPreview, setShowPreview] = useState(false);

  function handleMouseEnter() {
    // Trigger the dynamic import on hover
    import('./ProductPreview');
    setShowPreview(true);
  }

  return (
    <div onMouseEnter={handleMouseEnter}>
      <a href={`/products/${productId}`}>View Product</a>
      {showPreview && (
        <Suspense fallback={null}>
          <ProductPreview productId={productId} />
        </Suspense>
      )}
    </div>
  );
}
```

**Component Breakdown:**
- `import('./ProductPreview')` starts the download on hover.
- By the time the user clicks, the chunk is likely cached.
- `Suspense` with `fallback={null}` shows nothing while loading (the preview is optional).

**Deferred Preloading (Idle Time):**
```jsx
function PreloadOnIdle({ children, load }) {
  useEffect(() => {
    if ('requestIdleCallback' in window) {
      const id = requestIdleCallback(load);
      return () => cancelIdleCallback(id);
    }
    const id = setTimeout(load, 2000);
    return () => clearTimeout(id);
  }, [load]);

  return children;
}

function App() {
  return (
    <PreloadOnIdle load={() => import('./routes/Checkout')}>
      <RouterProvider router={router} />
    </PreloadOnIdle>
  );
}
```

**Component Breakdown:**
- `requestIdleCallback`: Runs the import when the browser is idle.
- `setTimeout` fallback for browsers without `requestIdleCallback`.
- The checkout chunk is ready before the user navigates to it.

**Viewport-Based Preloading (Intersection Observer):**
```jsx
function PreloadOnViewport({ children, load }) {
  const ref = useRef(null);

  useEffect(() => {
    const observer = new IntersectionObserver(([entry]) => {
      if (entry.isIntersecting) {
        load();
        observer.disconnect();
      }
    });
    if (ref.current) observer.observe(ref.current);
    return () => observer.disconnect();
  }, [load]);

  return <div ref={ref}>{children}</div>;
}
```

**Component Breakdown:**
- The import is triggered when the element enters the viewport.
- `observer.disconnect()` prevents repeated triggers.

**Syntax Rules:**
- **Route-based:** Split at route boundaries; use framework lazy loading (React Router `lazy`, Next.js `next/dynamic`).
- **Component-based:** Split heavy components regardless of route; use `React.lazy` + `<Suspense>`.
- **Interaction-based:** Trigger `import()` on hover, focus, or click; preload the chunk before the user commits.
- **Deferred preloading:** Use `requestIdleCallback`, Intersection Observer, or `webpackPrefetch` to fetch chunks ahead of time.
- **Combine strategies:** Route-based for navigation, component-based for heavy widgets, preloading for the next likely route.
- **Monitor the Network panel:** Verify that chunks are actually split and loaded as intended.
- **Avoid over-splitting:** Too many small chunks cause a network waterfall and HTTP overhead.

**Constraints and Limitations:**
- Over-splitting increases the number of network requests; HTTP/2 mitigates but does not eliminate the cost.
- Preloading can waste bandwidth if the user never navigates to the preloaded route.
- Interaction-based splitting does not help keyboard or screen-reader users unless hover is supplemented with focus.
- Deferred preloading competes with critical resources; use `requestIdleCallback` or low-priority hints.
- Framework-specific lazy loading (Next.js, React Router) may not support all custom patterns.

### Annotated Code Examples

**Example 1: Route-Based Splitting with React Router v7**

```tsx
// app/routes.ts
import { route, index } from '@react-router/dev/routes';

export default [
  index('routes/home.tsx'),
  route('products', 'routes/products.tsx'),
  route('products/:id', 'routes/product-detail.tsx'),
  route('checkout', 'routes/checkout.tsx'),
];
```

```tsx
// routes/products.tsx
export async function loader() {
  const products = await fetchProducts();
  return { products };
}

export function Component() {
  const { products } = useLoaderData();
  return <ProductGrid products={products} />;
}
```

**Expected Output:** Navigating to `/products` downloads only the products route chunk (plus shared chunks). The home route's chunk is not downloaded. The checkout chunk is downloaded only when the user navigates to `/checkout`.

**Why This Output Occurs:** React Router treats each route module as a split point. The bundler emits one chunk per route (plus shared chunks for common dependencies). Loaders and components load together because they are in the same module.

**Example 2: Interaction-Based Splitting for a Heavy Editor**

```jsx
import { lazy, Suspense, useState } from 'react';

const HeavyEditor = lazy(() => import('./HeavyEditor'));

export default function DocumentPage() {
  const [isEditing, setIsEditing] = useState(false);

  function handleEditClick() {
    // Preload the editor chunk on click
    import('./HeavyEditor');
    setIsEditing(true);
  }

  return (
    <div>
      <h1>Document</h1>
      {!isEditing && (
        <>
          <p>Document content...</p>
          <button onClick={handleEditClick}>Edit</button>
        </>
      )}
      {isEditing && (
        <Suspense fallback={<div>Loading editor...</div>}>
          <HeavyEditor onSave={() => setIsEditing(false)} />
        </Suspense>
      )}
    </div>
  );
}
```

**Expected Output:** The document renders without the editor. Clicking "Edit" downloads the editor chunk (visible in the Network tab) and shows "Loading editor..." briefly. The editor then mounts.

**Why This Output Occurs:** `import('./HeavyEditor')` is triggered on click, splitting the editor into its own chunk. The editor is not part of the initial bundle. `<Suspense>` shows the fallback while the chunk loads.

**Example 3: Deferred Preloading of the Next Likely Route**

```jsx
import { useEffect } from 'react';

export function PreloadCheckout() {
  useEffect(() => {
    // Preload the checkout chunk during idle time
    const id = requestIdleCallback(() => {
      import('./routes/Checkout');
    });
    return () => cancelIdleCallback(id);
  }, []);

  return null;
}

// Rendered on the cart page, preloading the checkout route
function CartPage() {
  return (
    <div>
      <PreloadCheckout />
      <h1>Your Cart</h1>
      {/* cart items */}
      <a href="/checkout">Proceed to Checkout</a>
    </div>
  );
}
```

**Expected Output:** When the cart page loads, the checkout chunk is fetched in the background during idle time. When the user clicks "Proceed to Checkout", the checkout page renders instantly because the chunk is already cached.

**Why This Output Occurs:** `requestIdleCallback` schedules the import during idle time, so it does not compete with critical resources. The chunk is stored in the browser cache and reused when the user navigates.

### Real-World Cases

- **E-commerce:** Route-based splitting for product listing, detail, and checkout; component-based splitting for reviews and recommendations; preloading the checkout chunk from the cart page.
- **SaaS dashboards:** Route-based splitting for dashboard sections; component-based splitting for charts and tables; preloading the next likely section.
- **Documentation:** Route-based splitting per article; component-based splitting for interactive examples; preloading the next article on hover.
- **Marketing sites:** Route-based splitting for landing pages; interaction-based splitting for chat widgets and forms.
- **Admin panels:** Route-based splitting for each admin section; component-based splitting for data grids and editors.

### References

- React Official Documentation – Code Splitting: https://legacy.reactjs.org/docs/code-splitting.html
- React Router v7 – Lazy Loading: https://reactrouter.com/7.1.0/start/data/lazy-loading
- Next.js – Lazy Loading: https://nextjs.org/docs/app/guides/lazy-loading
- Webpack – Code Splitting: https://webpack.js.org/guides/code-splitting/
- Webpack – Prefetching/Preloading: https://webpack.js.org/guides/code-splitting/#prefetchingpreloading-modules
- Vite – Code Splitting: https://vitejs.dev/guide/features.html#code-splitting
- MDN Web Docs – `requestIdleCallback`: https://developer.mozilla.org/en-US/docs/Web/API/Window/requestIdleCallback
- MDN Web Docs – Intersection Observer API: https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API
- web.dev – Reduce JavaScript Payloads with Code Splitting: https://web.dev/articles/reduce-javascript-payloads-with-code-splitting

---

## Comparison and Decision Guidance

| Topology | Split Point | Load Trigger | Best For | Cost |
|---|---|---|---|---|
| **Route-based** | Route module | Navigation | Most applications | Network round trip on navigation |
| **Component-based** | Heavy component | Render (conditional) | Charts, editors, maps | Network round trip when rendered |
| **Interaction-based** | Any module | Hover, click, focus | Modals, editors, previews | Network round trip on interaction |
| **Deferred preloading** | Any module | Idle, viewport, hover | Next likely route/widget | Wasted bandwidth if not visited |

**Decision Guidance:**
- **Start with route-based splitting** — it is the highest-impact, lowest-risk strategy.
- **Add component-based splitting** for heavy components that appear on multiple routes.
- **Use interaction-based splitting** for components that are only needed after a user action.
- **Add deferred preloading** for the next likely route or widget to eliminate perceived latency.
- **Combine strategies:** Route-based for navigation, component-based for widgets, preloading for the critical path.
- **Measure the impact:** Use Lighthouse, WebPageTest, or the Network panel to verify that chunks are split and loaded as intended.
- **Avoid over-splitting:** Too many small chunks cause a network waterfall; group related modules into shared chunks.
- **Match fallbacks to final layout:** Skeletons prevent layout shift; spinners do not.
- **Always pair lazy components with Error Boundaries** to handle chunk load failures.

---

## References

- MDN Web Docs – `import()`: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/import
- TC39 – Dynamic Import Proposal: https://github.com/tc39/proposal-dynamic-import
- React Official Documentation – `lazy`: https://react.dev/reference/react/lazy
- React Official Documentation – `<Suspense>`: https://react.dev/reference/react/Suspense
- React Official Documentation – Code Splitting: https://legacy.reactjs.org/docs/code-splitting.html
- React 19 – Release Notes: https://react.dev/blog/2024/12/05/react-19
- React 18 – Suspense in Data Frameworks: https://react.dev/blog/2022/03/29/react-v18#suspense-in-data-frameworks
- React Official Documentation – Streaming SSR: https://react.dev/reference/react-dom/server/renderToPipeableStream
- React Router v7 – Lazy Loading: https://reactrouter.com/7.1.0/start/data/lazy-loading
- Next.js – Lazy Loading: https://nextjs.org/docs/app/guides/lazy-loading
- Next.js – Loading UI and Streaming: https://nextjs.org/docs/app/building-your-application/routing/loading-ui-and-streaming
- Webpack – Code Splitting: https://webpack.js.org/guides/code-splitting/
- Webpack – Dynamic Imports: https://webpack.js.org/guides/code-splitting/#dynamic-imports
- Webpack – Magic Comments: https://webpack.js.org/api/module-methods/#magic-comments
- Webpack – Prefetching/Preloading: https://webpack.js.org/guides/code-splitting/#prefetchingpreloading-modules
- Vite – Dynamic Import: https://vitejs.dev/guide/features.html#dynamic-import
- Vite – Code Splitting: https://vitejs.dev/guide/features.html#code-splitting
- Rollup – Code Splitting: https://rollupjs.org/tutorial/#code-splitting
- MDN Web Docs – `requestIdleCallback`: https://developer.mozilla.org/en-US/docs/Web/API/Window/requestIdleCallback
- MDN Web Docs – Intersection Observer API: https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API
- web.dev – Reduce JavaScript Payloads with Code Splitting: https://web.dev/articles/reduce-javascript-payloads-with-code-splitting
- web.dev – Route-based Code Splitting: https://web.dev/articles/reduce-javascript-payloads-with-code-splitting#route-based_code-splitting
- Addy Osmani – The Cost of JavaScript: https://medium.com/@addyosmani/the-cost-of-javascript-in-2018-7d8950fbb5d4