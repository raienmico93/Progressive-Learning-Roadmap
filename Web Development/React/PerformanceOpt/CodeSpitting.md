# React Code Splitting — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Code splitting is the practice of breaking a JavaScript bundle into smaller, independently loadable chunks so that users download only the code needed for the current view, rather than the entire application upfront.

**Technical Definition:** Code splitting is a build-time partitioning technique that transforms a monolithic bundle into multiple chunks that can be loaded on demand. React provides built-in support through `React.lazy` and dynamic `import()`, which create split points that bundlers such as webpack, Vite, and Rollup use to generate separate chunks. The core insight is that users access one route at a time, so they should only pay the download, parse, and compile cost for that route's code. Code splitting directly improves Core Web Vitals, particularly Largest Contentful Paint (LCP) and Interaction to Next Paint (INP).

**Beginner-Friendly Explanation:** Code splitting is like packing for a trip. Instead of stuffing everything into one giant suitcase, you pack separate bags for different activities. You only carry the bag you need right now, and the rest stay in the car until you need them. In a React app, code splitting means your users download only the JavaScript required for the page they are viewing, not the code for every page in the entire application.

### Key Characteristics

- **On-demand loading:** Chunks are fetched only when the user navigates to a route or triggers a component.
- **Build-time partitioning:** Bundlers determine chunk boundaries at build time; fully dynamic import paths cannot be split.
- **Chunk-count trade-off:** HTTP/2 sweet spot is roughly 10–30 chunks; too few reduces caching benefits, too many saturates streams.
- **Compression sensitivity:** Smaller chunks compress less efficiently than larger ones; a 10KB chunk may compress only 50%, while a 100KB chunk often reaches 70%+.
- **Loading strategy matters:** Splitting without intelligent preloading or prefetching creates waterfalls; preloading critical resources and prefetching predicted routes recovers most of the latency cost.
- **React-native support:** `React.lazy` and `Suspense` are built into React; `next/dynamic` and `@loadable/component` extend the pattern for SSR and advanced cases.

### Prerequisites

- React function components and JSX.
- Understanding of ES modules (`import`/`export`).
- Familiarity with a bundler (webpack, Vite, Rollup, or a framework like Next.js).
- Basic knowledge of `Suspense` and loading states.

### Related Programming Areas

- **React Rendering Performance** — code splitting reduces the initial JavaScript payload that must be parsed and executed.
- **Route-Based Architecture** — routes are the natural split boundaries.
- **Bundle Analysis** — tools like `webpack-bundle-analyzer` and `rollup-plugin-visualizer` verify that splitting works.
- **Server-Side Rendering** — `React.lazy` has SSR limitations; `@loadable/component` and `next/dynamic` provide alternatives.

---

## 1. Code Splitting

### Definitions

**Core Definition:** Code splitting is the technique of dividing a JavaScript application into multiple bundles that can be loaded independently and on demand.

**Technical Definition:** Code splitting uses dynamic `import()` to create split points. Bundlers analyze these calls at build time and emit separate chunks for each dynamically imported module. The initial bundle contains only the code needed for the first render; other chunks are fetched when their corresponding modules are rendered. React's `React.lazy` wraps dynamic imports for components, and `Suspense` provides a fallback UI while the chunk loads.

**Beginner-Friendly Explanation:** Code splitting means your app does not download everything at once. It downloads the first screen's code, and then fetches other parts of the app only when you navigate to them or interact with something that needs them.

### Purposes

- To reduce the initial JavaScript payload and improve load time.
- To improve Largest Contentful Paint (LCP) by shortening the critical path.
- To improve Interaction to Next Paint (INP) by reducing main-thread blocking from unused code.
- To raise cache hit rates by isolating rarely changing vendor code from frequently changing application code.
- To keep memory usage lower on constrained devices by loading only the modules in use.

### Syntax Rules and Structure

**General Syntax (Dynamic Import):**

```js
const module = await import('./heavy-module');
```

**General Syntax (React.lazy + Suspense):**

```jsx
import { lazy, Suspense } from 'react';

const LazyComponent = lazy(() => import('./LazyComponent'));

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <LazyComponent />
    </Suspense>
  );
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `import('./module')` | Dynamic import returning a Promise that resolves to the module. |
| `lazy(() => import(...))` | Wraps the dynamic import as a React component. |
| `<Suspense fallback={...}>` | Shows a fallback UI while the lazy component loads. |

**Syntax Rules:**

1. Dynamic `import()` returns a Promise; the bundler creates a separate chunk for the imported module.
2. `React.lazy` requires the dynamic import to return a module with a `default` export.
3. Lazy components must be rendered inside a `Suspense` boundary; otherwise, React throws an error.
4. `Suspense` fallbacks should be meaningful (skeletons, not blank screens).

**Constraints and Limitations:**

- Code splitting is a build-time transformation; fully dynamic import paths (`import(userInput)`) cannot be split.
- Too many small chunks can hurt compression and increase request overhead.
- `React.lazy` does not support server-side rendering directly; use `@loadable/component` or framework-provided alternatives.
- Nested `Suspense` boundaries can create a waterfall effect, where components load sequentially.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Basic Dynamic Import (Without React.lazy)

```js
// Step 1: Dynamic import creates a separate chunk
button.addEventListener('click', async () => {
  const { heavyFeature } = await import('./heavy-feature');
  heavyFeature();
});
```

**Expected Output:** The `heavy-feature` chunk is downloaded only when the button is clicked. The initial bundle does not include `heavy-feature`.

**Why this output:** The static import would bundle `heavy-feature` into the main chunk. The dynamic `import()` tells the bundler to split it into a separate chunk that is fetched on demand.

#### Example 2: Basic `React.lazy` + `Suspense`

```jsx
import { lazy, Suspense } from 'react';

// Step 1: Lazy-load a component
const LazyChart = lazy(() => import('./LazyChart'));

function Dashboard() {
  return (
    <Suspense fallback={<ChartSkeleton />}>
      <LazyChart />
    </Suspense>
  );
}
```

**Expected Output:** The chart chunk is downloaded when `Dashboard` renders. While loading, `ChartSkeleton` is shown. Once loaded, the chart appears.

**Why this output:** `React.lazy` creates a lazy component that suspends rendering until its chunk is available. `Suspense` catches the suspension and displays the fallback.

### Real-World Cases

- **Admin dashboards:** Loading charting and data-grid libraries only when the user visits the analytics page.
- **Rich text editors:** Loading Monaco or CodeMirror only when the user opens the editor.
- **Modals and dialogs:** Deferring modal content until the user clicks to open it.
- **Legal pages:** Splitting low-traffic pages out of the entry bundle.

### References

- Code Splitting - React — https://mintlify.wiki/facebook/react/advanced/code-splitting
- How to implement code splitting in JavaScript · CoreUI — https://coreui.io/answers/how-to-implement-code-splitting-in-javascript/
- Bundle Splitting Strategies — https://github.com/sujeet-pro/sujeet.pro/blob/main/content/articles/bundle-splitting-strategies/README.md
- Steve Kinney — Code-Splitting and Lazy Loading — https://stevekinney.com/courses/react-performance/code-splitting-and-lazy-loading

---

## 2. Dynamic Imports

### Definitions

**Core Definition:** A dynamic import is a JavaScript expression (`import()`) that loads a module asynchronously and returns a Promise, enabling bundlers to create a separate chunk.

**Technical Definition:** `import(specifier)` is the TC39 dynamic import proposal. Unlike static `import` declarations, which are hoisted and bundled into the main chunk, dynamic imports are evaluated at runtime and return a Promise that resolves to the module namespace object. Bundlers treat each dynamic import call as a split point and emit a separate chunk.

**Beginner-Friendly Explanation:** A dynamic import is a way to say “load this file later, when I actually need it.” Instead of putting everything in the main bundle, the bundler creates a separate file that is fetched only when the import runs.

### Purposes

- To create split points that bundlers can turn into separate chunks.
- To load modules conditionally based on user interaction or runtime conditions.
- To defer heavy libraries until they are needed.
- To enable lazy loading without React-specific APIs.

### Syntax Rules and Structure

**General Syntax:**

```js
const module = await import('./module-path');
```

**Syntax Rules:**

1. Dynamic `import()` returns a Promise.
2. The import path must be statically analyzable for bundlers to create a chunk; fully dynamic paths cannot be split.
3. The resolved module is the module namespace object; use `.default` for default exports.

**Constraints and Limitations:**

- A fully dynamic path (`import(userInput)`) cannot be statically analyzed and therefore cannot be split.
- Dynamic imports are asynchronous; the caller must handle the Promise.
- Server-side environments may not support dynamic imports in all configurations.

### Annotated Code Examples

```js
// ✅ Statically analyzable: bundler creates a chunk for './analytics'
async function loadAnalytics() {
  const { trackEvent } = await import('./analytics');
  trackEvent('page_view');
}

// ❌ Not statically analyzable: bundler cannot split
async function loadPage(page) {
  return import(`./pages/${page}`); // May fail to split
}
```

**Expected Output:** The first example creates a separate chunk for `./analytics`. The second example may not create a chunk because the path is dynamic.

**Why this output:** Bundlers need to know the module path at build time to generate a chunk. Template literals with dynamic values prevent static analysis.

### Real-World Cases

- **Third-party libraries:** Loading `monaco-editor` only when the editor is opened.
- **Polyfills:** Loading polyfills only for browsers that need them.
- **Feature modules:** Loading a feature module only when the user has permission to use it.

### References

- How to implement code splitting in JavaScript · CoreUI — https://coreui.io/answers/how-to-implement-code-splitting-in-javascript/
- Bundle Splitting Strategies — https://github.com/sujeet-pro/sujeet.pro/blob/main/content/articles/bundle-splitting-strategies/README.md

---

## 3. `React.lazy`

### Definitions

**Core Definition:** `React.lazy` is a React function that lets you render a dynamic import as a regular component, automatically loading the component's chunk when it is first rendered.

**Technical Definition:** `const LazyComponent = lazy(() => import('./Component'))` returns a lazy component. React renders a `Suspense` fallback while the dynamic import is pending, then renders the component once the Promise resolves. The dynamic import must return a module with a `default` export.

**Beginner-Friendly Explanation:** `React.lazy` is the React way to lazy-load a component. You give it a function that imports the component, and React handles showing a fallback while the component loads.

### Purposes

- To lazy-load React components with minimal code.
- To split component code into separate chunks.
- To defer heavy components until they are rendered.
- To integrate with `Suspense` for loading states.

### Syntax Rules and Structure

```jsx
import { lazy, Suspense } from 'react';

const LazyComponent = lazy(() => import('./LazyComponent'));

<Suspense fallback={<Loading />}>
  <LazyComponent />
</Suspense>
```

**Syntax Rules:**

1. `lazy` takes a function that returns a dynamic import Promise.
2. The imported module must have a `default` export.
3. The lazy component must be rendered inside a `Suspense` boundary.
4. Lazy components cannot be used with server-side rendering directly.

**Constraints and Limitations:**

- `React.lazy` does not support SSR without additional tooling.
- Lazy components are async; they suspend during the first render.
- Named exports are not supported by `React.lazy`; use default exports.

### Annotated Code Examples

```jsx
import { lazy, Suspense } from 'react';

// ✅ Correct: default export
const HeavyChart = lazy(() => import('./HeavyChart'));

function Dashboard() {
  return (
    <Suspense fallback={<ChartSkeleton />}>
      <HeavyChart />
    </Suspense>
  );
}
```

**Expected Output:** The `HeavyChart` chunk is fetched when `Dashboard` renders. While fetching, `ChartSkeleton` is shown.

**Why this output:** `lazy` creates a component that suspends until its chunk loads. `Suspense` provides the fallback.

### Real-World Cases

- **Route components:** Lazy-loading each page component.
- **Modals:** Loading modal content only when opened.
- **Heavy charts and editors:** Deferring large third-party libraries.

### References

- Code Splitting - React — https://mintlify.wiki/facebook/react/advanced/code-splitting
- Comparison with React.lazy — https://raw.githubusercontent.com/gregberge/loadable-components/refs/heads/main/website/src/pages/docs/loadable-vs-react-lazy.mdx
- React.lazy with streaming SSR — https://github.com/shakacode/react_on_rails/issues/3154

---

## 4. `Suspense`

### Definitions

**Core Definition:** `Suspense` is a React component that displays a fallback UI while its children are loading, including lazy-loaded components.

**Technical Definition:** `<Suspense fallback={...}>` catches suspended rendering in its subtree and renders the fallback until the suspension resolves. When used with `React.lazy`, the fallback is shown while the lazy chunk is fetched. `Suspense` boundaries can be nested, but nested boundaries may cause a waterfall effect if components load sequentially.

**Beginner-Friendly Explanation:** `Suspense` is what shows the loading spinner or skeleton while a lazy component is downloading. You wrap lazy components in `Suspense` and tell it what to display while waiting.

### Purposes

- To provide a loading state for lazy-loaded components.
- To prevent rendering errors when a component is not ready.
- To coordinate loading boundaries across a subtree.
- To enable streaming server rendering in React 18+.

### Syntax Rules and Structure

```jsx
<Suspense fallback={<Loading />}>
  <LazyComponent />
</Suspense>
```

**Syntax Rules:**

1. `Suspense` requires a `fallback` prop.
2. Lazy components must be inside a `Suspense` boundary.
3. Nested `Suspense` boundaries can create waterfalls; avoid deep nesting unless intentional.
4. `Suspense` is not available server-side for `React.lazy`; use framework alternatives.

**Constraints and Limitations:**

- Nested `Suspense` boundaries can cause sequential loading (waterfall).
- `Suspense` fallbacks should be meaningful, not blank screens.
- `React.lazy` + `Suspense` does not work with SSR without additional tooling.

### Annotated Code Examples

```jsx
function Page() {
  return (
    <>
      <Header />
      <Suspense fallback={<ChartSkeleton />}>
        <LazyChart />
      </Suspense>
      <Suspense fallback={<TableSkeleton />}>
        <LazyTable />
      </Suspense>
    </>
  );
}
```

**Expected Output:** The header renders immediately. The chart and table show their respective skeletons until their chunks load.

**Why this output:** Each `Suspense` boundary independently manages its fallback, allowing the header to render without waiting for the lazy components.

### Real-World Cases

- **Dashboard sections:** Independent loading states for charts, tables, and maps.
- **Route transitions:** Showing a spinner while the next route's chunk loads.
- **Modals:** Displaying a skeleton while modal content loads.

### References

- Code Splitting - React — https://mintlify.wiki/facebook/react/advanced/code-splitting
- How to Prevent Suspense Waterfall — https://stackoverflow.com/feeds/question/79707306

---

## 5. Route-Based Code Splitting

### Definitions

**Core Definition:** Route-based code splitting loads the code for each route only when the user navigates to that route, creating a separate chunk per page.

**Technical Definition:** Route-based splitting is the most impactful splitting strategy for single-page applications. Each route component is wrapped in `React.lazy`, so the bundler emits a separate chunk for each route. React Router 7.x supports `lazy()` route definitions directly, and framework modes (e.g., React Router framework mode) code-split each route file by default.

**Beginner-Friendly Explanation:** Route-based splitting means each page of your app is its own separate file. When a user goes to the dashboard, they download the dashboard code. When they go to settings, they download the settings code. They never download code for pages they do not visit.

### Purposes

- To ensure users download only the code for the page they visit.
- To reduce the initial bundle size and improve LCP.
- To raise cache hit rates because route chunks change less frequently than the entry bundle.
- To align code splitting with the application's information architecture.

### Syntax Rules and Structure

**React Router 7.x Lazy Routes:**

```jsx
import { createBrowserRouter } from 'react-router';

const router = createBrowserRouter([
  { path: '/', lazy: () => import('./pages/Home') },
  { path: '/dashboard', lazy: () => import('./pages/Dashboard') },
  { path: '/settings', lazy: () => import('./pages/Settings') },
]);
```

**React.lazy with React Router v6:**

```jsx
import { lazy, Suspense } from 'react';
import { BrowserRouter, Routes, Route } from 'react-router-dom';

const Home = lazy(() => import('./pages/Home'));
const Dashboard = lazy(() => import('./pages/Dashboard'));

function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<PageSkeleton />}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/dashboard" element={<Dashboard />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
}
```

**Syntax Rules:**

1. Use `lazy()` route definitions (React Router 7) or `React.lazy` + `Suspense` (v6).
2. Each route becomes a separate chunk; users download only the current route's code.
3. Name chunks for readable build output and caching.

**Constraints and Limitations:**

- Route-level splitting is the baseline; component-level splitting inside routes adds further granularity.
- Over-splitting into too many tiny routes can hurt compression and request overhead.

### Annotated Code Examples

```jsx
import { lazy, Suspense } from 'react';
import { BrowserRouter, Routes, Route } from 'react-router-dom';

const Home = lazy(() => import('./pages/Home'));
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Settings = lazy(() => import('./pages/Settings'));

function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<PageLoader />}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/dashboard" element={<Dashboard />} />
          <Route path="/settings" element={<Settings />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
}
```

**Expected Output:** Navigating to `/dashboard` downloads only the dashboard chunk. The home chunk is not downloaded unless the user visits `/`.

**Why this output:** Each `lazy()` call creates a split point. React Router renders the matching route, and `Suspense` shows the loader while the chunk fetches.

### Real-World Cases

- **Admin panels:** Dashboard, orders, and settings each load on demand.
- **E-commerce:** Product listing and checkout pages split into separate chunks.
- **Content sites:** Article pages split from the homepage.

### References

- Code Splitting - React Router — https://mintlify.wiki/remix-run/react-router/advanced/code-splitting
- Steve Kinney — Code-Splitting and Lazy Loading — https://stevekinney.com/courses/react-performance/code-splitting-and-lazy-loading
- Split code at route boundaries — https://github.com/yonatangross/orchestkit/blob/HEAD/plugins/ork/skills/performance/rules/loading-splitting.md

---

## 6. Component-Based Code Splitting

### Definitions

**Core Definition:** Component-based code splitting defers the loading of individual components until they are actually rendered, rather than splitting only at route boundaries.

**Technical Definition:** Component-based splitting uses `React.lazy` or dynamic imports to split heavy components (charts, editors, maps) into their own chunks. The component is rendered conditionally, so its chunk is fetched only when the condition is met (e.g., the user opens a tab or clicks a button).

**Beginner-Friendly Explanation:** Component-based splitting means you can lazy-load a single component, like a chart or a rich text editor, only when it is needed. You do not have to split the whole page.

### Purposes

- To defer heavy components that are not needed on initial render.
- To load components conditionally based on user interaction or state.
- To keep the initial route chunk small while still including the page's basic UI.
- To improve INP by avoiding the parse and execution cost of unused components.

### Syntax Rules and Structure

```jsx
const HeavyChart = lazy(() => import('./HeavyChart'));

function Dashboard({ activeTab }) {
  return (
    <Suspense fallback={<ChartSkeleton />}>
      {activeTab === 'chart' && <HeavyChart />}
    </Suspense>
  );
}
```

**Syntax Rules:**

1. Wrap the heavy component in `React.lazy`.
2. Render it conditionally inside a `Suspense` boundary.
3. Each lazy component gets its own chunk.
4. Provide a meaningful fallback for each component.

**Constraints and Limitations:**

- Component-level splitting adds more chunks than route-level splitting; balance granularity against HTTP/2 overhead.
- Conditional rendering must be inside `Suspense` to handle the loading state.

### Annotated Code Examples

```jsx
import { lazy, Suspense, useState } from 'react';

const HeavyChart = lazy(() => import('./components/HeavyChart'));
const HeavyDataTable = lazy(() => import('./components/HeavyDataTable'));

function Dashboard() {
  const [activeTab, setActiveTab] = useState('chart');

  return (
    <div>
      <nav>
        <button onClick={() => setActiveTab('chart')}>Chart</button>
        <button onClick={() => setActiveTab('table')}>Table</button>
      </nav>
      <Suspense fallback={<LoadingSpinner />}>
        {activeTab === 'chart' && <HeavyChart />}
        {activeTab === 'table' && <HeavyDataTable />}
      </Suspense>
    </div>
  );
}
```

**Expected Output:** The chart chunk loads when the chart tab is active. Switching to the table loads the table chunk. Neither chunk is in the initial bundle.

**Why this output:** Each heavy component is lazily imported. The conditional rendering ensures only the active tab's chunk is requested.

### Real-World Cases

- **Dashboard tabs:** Charts, tables, and maps loaded per tab.
- **Rich text editors:** Loading Monaco only when the editor is opened.
- **Settings panels:** Loading advanced settings only when expanded.

### References

- Code Splitting - React — https://mintlify.wiki/facebook/react/advanced/code-splitting
- How to implement code splitting in JavaScript · CoreUI — https://coreui.io/answers/how-to-implement-code-splitting-in-javascript/
- Code Splitting - React Router (third-party libraries) — https://mintlify.wiki/remix-run/react-router/advanced/code-splitting

---

## 7. Vendor Splitting

### Definitions

**Core Definition:** Vendor splitting isolates third-party dependencies (`node_modules`) into stable, separately cacheable chunks so that application code changes do not invalidate the vendor cache.

**Technical Definition:** Vendor splitting separates dependencies such as React, React DOM, charting libraries, and UI kits into dedicated chunks. Because dependencies change less frequently than application code, separate vendor chunks raise cache hit rates across deployments. Webpack uses `SplitChunksPlugin`; Vite/Rollup use `manualChunks` (or `advancedChunks` in Vite 8+).

**Beginner-Friendly Explanation:** Vendor splitting means putting all your third-party libraries into their own file. When you update your own code, users do not have to re-download React and other libraries because those files have not changed.

### Purposes

- To raise cache hit rates by isolating rarely changing dependencies.
- To reduce the size of the application chunk that changes on every deployment.
- To allow long-term caching of vendor code.
- To avoid duplicating shared dependencies across multiple entry chunks.

### Syntax Rules and Structure

**Webpack (`SplitChunksPlugin`):**

```js
optimization: {
  splitChunks: {
    chunks: 'all',
    cacheGroups: {
      vendor: {
        test: /[\\/]node_modules[\\/]/,
        name: 'vendors',
        priority: -10,
      },
    },
  },
}
```

**Vite/Rollup (`manualChunks`):**

```js
// vite.config.js
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
          charts: ['recharts', 'd3'],
        },
      },
    },
  },
});
```

**Syntax Rules:**

1. Isolate `node_modules` into one or more vendor chunks.
2. Name chunks for readable build output and caching.
3. Group related vendor libraries together (e.g., all charting libraries in one chunk).
4. Vite 8+ deprecates the object form of `manualChunks`; use `codeSplitting` or `advancedChunks`.

**Constraints and Limitations:**

- Over-splitting vendor code into too many chunks can hurt compression.
- Vendor chunks must be cache-friendly; hashed filenames should be used for long-term caching.
- Vite 8+ changes the `manualChunks` API; verify the current configuration syntax.

### Annotated Code Examples

```js
// vite.config.js — vendor splitting
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          'vendor-react': ['react', 'react-dom'],
          'vendor-charts': ['recharts', 'd3'],
        },
      },
    },
  },
});
```

**Expected Output:** React and React DOM are bundled into `vendor-react`, charting libraries into `vendor-charts`. When application code changes, these vendor chunks remain cached.

**Why this output:** The bundler creates separate chunks for the specified dependency groups. Their content hashes remain stable across application deployments.

### Real-World Cases

- **Design systems:** Vendor chunk for React and UI primitives.
- **Data visualisation:** Separate chunk for charting libraries.
- **Monorepos:** Shared vendor chunk across multiple applications.

### References

- Bundle Splitting Strategies — https://github.com/sujeet-pro/sujeet.pro/blob/main/content/articles/bundle-splitting-strategies/README.md
- Vite manualChunks / vendor splitting — https://github.com/pproenca/dot-skills/blob/master/skills/.experimental/vite/references/bundle-manual-chunks.md
- Split code at route boundaries — https://github.com/yonatangross/orchestkit/blob/HEAD/plugins/ork/skills/performance/rules/loading-splitting.md

---

## 8. Chunk Optimization

### Definitions

**Core Definition:** Chunk optimization is the practice of configuring the bundler to produce an optimal number and size of chunks, balancing caching benefits against request overhead and compression efficiency.

**Technical Definition:** Chunk optimization involves tuning chunk boundaries, naming, and grouping to hit the HTTP/2 sweet spot (roughly 10–30 chunks), preserving compression effectiveness, and avoiding waterfalls. It includes naming chunks for debugging, grouping shared dependencies, and using magic comments (`webpackChunkName`, `webpackPrefetch`, `webpackPreload`) to control chunk behaviour.

**Beginner-Friendly Explanation:** Chunk optimization is about finding the right balance: not too few chunks (poor caching) and not too many (too many requests). It also means giving your chunks clear names so you can debug them and controlling when they load.

### Purposes

- To balance caching benefits against HTTP/2 request overhead.
- To preserve compression efficiency by avoiding excessively small chunks.
- To name chunks for readable build output and debugging.
- To control preload and prefetch behaviour through bundler magic comments.

### Syntax Rules and Structure

**Webpack Magic Comments:**

```js
const Dashboard = lazy(() =>
  import(/* webpackChunkName: "dashboard" */ './pages/Dashboard')
);
```

**Prefetch and Preload:**

```js
import(/* webpackPrefetch: true */ './Settings');
import(/* webpackPreload: true */ './CriticalFeature');
```

**Syntax Rules:**

1. `webpackChunkName` sets a readable name for the chunk.
2. `webpackPrefetch: true` fetches the chunk during browser idle time.
3. `webpackPreload: true` fetches the chunk in parallel with the parent chunk.
4. HTTP/2 sweet spot is roughly 10–30 chunks.
5. Smaller chunks compress less efficiently; avoid splitting below ~10KB unless necessary.

**Constraints and Limitations:**

- Magic comments are webpack-specific; Vite/Rollup use different mechanisms.
- Over-splitting increases request count and runtime resolution overhead.
- Chunk names should be stable for long-term caching.

### Annotated Code Examples

```js
// ✅ Named chunk + prefetch
const Settings = lazy(() =>
  import(
    /* webpackChunkName: "settings" */
    /* webpackPrefetch: true */
    './pages/Settings'
  )
);
```

**Expected Output:** The settings chunk is named `settings` and is prefetched during idle time after the initial page loads.

**Why this output:** `webpackChunkName` gives the chunk a readable name. `webpackPrefetch` instructs the browser to download the chunk when idle, improving perceived navigation speed.

### Real-World Cases

- **Navigation menus:** Prefetching route chunks when the user hovers over a link.
- **Critical features:** Preloading chunks needed for the first interaction.
- **Bundle analysis:** Naming chunks to identify them in bundle analyzer output.

### References

- How to implement code splitting in JavaScript · CoreUI — https://coreui.io/answers/how-to-implement-code-splitting-in-javascript/
- Bundle Splitting Strategies — https://github.com/sujeet-pro/sujeet.pro/blob/main/content/articles/bundle-splitting-strategies/README.md
- Split code at route boundaries — https://github.com/yonatangross/orchestkit/blob/HEAD/plugins/ork/skills/performance/rules/loading-splitting.md

---

## 9. Preloading

### Definitions

**Core Definition:** Preloading is the practice of fetching a resource with high priority because it is needed for the current navigation or immediate interaction.

**Technical Definition:** Preloading can be implemented via bundler magic comments (`webpackPreload: true`) or via browser resource hints (`<link rel="preload">`). Preloading tells the browser to download the resource in parallel with the parent chunk, reducing the waterfall effect when the resource is needed quickly.

**Beginner-Friendly Explanation:** Preloading means “download this now because I am about to need it.” It is useful for chunks that are required immediately after the current page loads.

### Purposes

- To eliminate waterfalls for critical chunks.
- To fetch resources in parallel with the parent chunk.
- To ensure a smooth transition when navigating to a route that is known to be next.

### Syntax Rules and Structure

**Webpack Preload:**

```js
import(/* webpackPreload: true */ './CriticalFeature');
```

**Browser Resource Hint:**

```html
<link rel="preload" href="/chunks/critical.js" as="script">
```

**Syntax Rules:**

1. Use preload sparingly; overusing it competes for bandwidth with critical resources.
2. Preload only resources that are needed for the current navigation or first interaction.
3. Preload hints should be used for the initial route's chunks.

### Annotated Code Examples

```js
// ✅ Preload a chunk needed immediately after the current page
const CriticalFeature = lazy(() =>
  import(/* webpackPreload: true */ './CriticalFeature')
);
```

**Expected Output:** The `CriticalFeature` chunk is downloaded in parallel with the parent chunk, reducing the delay when it is rendered.

**Why this output:** `webpackPreload` instructs the browser to fetch the chunk with high priority alongside the parent, avoiding a sequential waterfall.

### Real-World Cases

- **Critical user flows:** Preloading the checkout chunk when the user is on the cart page.
- **Above-the-fold content:** Preloading the chunk for the most common next route.
- **First interaction:** Preloading the chunk needed for the primary CTA.

### References

- How to implement code splitting in JavaScript · CoreUI — https://coreui.io/answers/how-to-implement-code-splitting-in-javascript/
- Bundle Splitting Strategies (prefetch/preload timing) — https://github.com/sujeet-pro/sujeet.pro/blob/main/content/articles/bundle-splitting-strategies/README.md

---

## 10. Prefetching

### Definitions

**Core Definition:** Prefetching is the practice of downloading a resource during browser idle time because it is likely to be needed for a future navigation or interaction.

**Technical Definition:** Prefetching can be implemented via bundler magic comments (`webpackPrefetch: true`) or browser resource hints (`<link rel="prefetch">`). Prefetching uses low-priority background requests to download chunks for predicted routes, so they are already cached when the user navigates. Hover-based prefetching on navigation links is a common pattern.

**Beginner-Friendly Explanation:** Prefetching means “download this in the background because the user will probably need it soon.” It makes future navigations feel instant because the code is already downloaded.

### Purposes

- To improve perceived performance for predicted navigations.
- To use idle network time to pre-populate the cache.
- To reduce the latency of route transitions.

### Syntax Rules and Structure

**Webpack Prefetch:**

```js
import(/* webpackPrefetch: true */ './Settings');
```

**Hover-Based Prefetching:**

```jsx
<Link
  to="/dashboard"
  onMouseEnter={() => import('./pages/Dashboard')}
>
  Dashboard
</Link>
```

**Syntax Rules:**

1. Prefetch during idle time; do not compete with critical resources.
2. Prefetch predicted routes based on user behaviour (hover, viewport visibility).
3. Combine with route-based splitting for maximum benefit.

**Constraints and Limitations:**

- Prefetching too many chunks can waste bandwidth and battery.
- Prefetched chunks must be cacheable and versioned correctly.

### Annotated Code Examples

```jsx
// ✅ Hover-based prefetching
import { Link } from 'react-router-dom';

function Navigation() {
  return (
    <Link
      to="/dashboard"
      onMouseEnter={() => {
        import('./pages/Dashboard');
      }}
    >
      Dashboard
    </Link>
  );
}
```

**Expected Output:** When the user hovers over the “Dashboard” link, the dashboard chunk is prefetched. Clicking the link navigates instantly because the chunk is already cached.

**Why this output:** The `onMouseEnter` handler triggers the dynamic import, which downloads the chunk before the user commits to the navigation.

### Real-World Cases

- **Navigation menus:** Prefetching routes on hover.
- **Next-step wizards:** Prefetching the next step's chunk when the user completes the current step.
- **E-commerce:** Prefetching the product page chunk from a listing page.

### References

- How to implement code splitting in JavaScript · CoreUI — https://coreui.io/answers/how-to-implement-code-splitting-in-javascript/
- Code Splitting - React Router (preloading links) — https://mintlify.wiki/remix-run/react-router/advanced/code-splitting
- Bundle Splitting Strategies (prefetch/preload timing) — https://github.com/sujeet-pro/sujeet.pro/blob/main/content/articles/bundle-splitting-strategies/README.md

---

## 11. Code Splitting Best Practices

### Definitions

**Core Definition:** Code splitting best practices are the guidelines for choosing split boundaries, configuring bundlers, and loading chunks in a way that maximises performance without creating new problems.

**Technical Definition:** Best practices combine route-based splitting as the baseline, component-level splitting for heavy libraries, vendor splitting for cache stability, chunk-count balancing for HTTP/2, meaningful loading states, and SSR-compatible alternatives when `React.lazy` is insufficient. They also include avoiding over-splitting, monitoring bundle output with analyzers, and handling chunk-load failures gracefully.

**Beginner-Friendly Explanation:** These are the rules of thumb for code splitting: split by route first, then lazy-load heavy components. Keep vendor code separate. Do not create too many tiny chunks. Show a skeleton while loading. And test that your splitting actually works.

### Best Practices Table

| Practice | Description |
|----------|-------------|
| **Split at route boundaries first** | Every page should be its own chunk; this is the highest-impact split. |
| **Lazy-load heavy libraries (> ~50KB)** | Charts, editors, and maps should not be in the initial bundle. |
| **Use vendor splitting** | Isolate `node_modules` into stable chunks for long-term caching. |
| **Balance chunk count** | Aim for 10–30 chunks; too few hurts caching, too many hurts compression and HTTP/2. |
| **Name chunks** | Use `webpackChunkName` for readable build output and caching. |
| **Provide meaningful fallbacks** | Show skeletons, not blank screens. |
| **Preload critical chunks, prefetch predicted routes** | Recover the latency cost of splitting. |
| **Avoid Suspense waterfalls** | Do not nest `Suspense` boundaries unnecessarily. |
| **Handle chunk-load failures** | Stale deploys can cause dynamic imports to reject; show a reload prompt. |
| **Verify with bundle analyzers** | Use `webpack-bundle-analyzer` or `rollup-plugin-visualizer` to confirm splitting. |
| **Use `@loadable/component` for SSR** | `React.lazy` does not support SSR directly. |
| **Measure with Core Web Vitals** | Confirm LCP and INP improvements after splitting. |

### Annotated Code Examples

**✅ Good Practices:**

```jsx
// Route-based splitting + meaningful fallback + prefetch
const Dashboard = lazy(() =>
  import(/* webpackChunkName: "dashboard" */ './pages/Dashboard')
);

function App() {
  return (
    <Suspense fallback={<PageSkeleton />}>
      <Routes>
        <Route path="/dashboard" element={<Dashboard />} />
      </Routes>
    </Suspense>
  );
}
```

```js
// Vendor splitting in Vite
manualChunks: {
  vendor: ['react', 'react-dom'],
  charts: ['recharts', 'd3'],
}
```

**❌ Bad Practices:**

```jsx
// ❌ Eager imports bundle all routes together
import Home from './pages/Home';
import Dashboard from './pages/Dashboard';

// ❌ Lazy component without Suspense
<LazyComponent /> // Throws an error
```

**Expected Output:** The good practices produce a small initial bundle, cacheable vendor chunks, and smooth route transitions. The bad practices bundle everything together and cause runtime errors.

**Why this output:** Eager imports defeat code splitting. Lazy components require a `Suspense` boundary to handle the loading state.

### Real-World Cases

- **Large SPAs:** Route-based splitting plus vendor splitting plus bundle analysis.
- **SSR applications:** Using `@loadable/component` or framework-specific lazy loading instead of `React.lazy`.
- **E-commerce:** Splitting product pages, cart, and checkout into separate chunks with prefetching on navigation.

### References

- Bundle Splitting Strategies — https://github.com/sujeet-pro/sujeet.pro/blob/main/content/articles/bundle-splitting-strategies/README.md
- Code Splitting Best Practices — https://github.com/mcorbett51090/RavenClaude/blob/main/plugins/frontend-engineering/best-practices/code-split-by-route-and-lazy-load-heavy-components.md
- Bundle optimization and analyzer — https://github.com/curiositech/windags-skills
- Comparison with React.lazy (SSR limitations) — https://raw.githubusercontent.com/gregberge/loadable-components/refs/heads/main/website/src/pages/docs/loadable-vs-react-lazy.mdx
- React.lazy with streaming SSR — https://github.com/shakacode/react_on_rails/issues/3154
- How to Prevent Suspense Waterfall — https://stackoverflow.com/feeds/question/79707306

---

## References

- Code Splitting - React — https://mintlify.wiki/facebook/react/advanced/code-splitting
- How to implement code splitting in JavaScript · CoreUI — https://coreui.io/answers/how-to-implement-code-splitting-in-javascript/
- Code Splitting - React Router — https://mintlify.wiki/remix-run/react-router/advanced/code-splitting
- Steve Kinney — Code-Splitting and Lazy Loading — https://stevekinney.com/courses/react-performance/code-splitting-and-lazy-loading
- Bundle Splitting Strategies — https://github.com/sujeet-pro/sujeet.pro/blob/main/content/articles/bundle-splitting-strategies/README.md
- Comparison with React.lazy — https://raw.githubusercontent.com/gregberge/loadable-components/refs/heads/main/website/src/pages/docs/loadable-vs-react-lazy.mdx
- Next.js — Lazy Loading — https://nextjs.org/docs/15/app/guides/lazy-loading
- React.lazy with streaming SSR — https://github.com/shakacode/react_on_rails/issues/3154
- How to Prevent Suspense Waterfall — https://stackoverflow.com/feeds/question/79707306
- Split code at route boundaries — https://github.com/yonatangross/orchestkit/blob/HEAD/plugins/ork/skills/performance/rules/loading-splitting.md
- Vite manualChunks / vendor splitting — https://github.com/pproenca/dot-skills/blob/master/skills/.experimental/vite/references/bundle-manual-chunks.md
- Bundle optimization and analyzer — https://github.com/curiositech/windags-skills
- Code Splitting Best Practices — https://github.com/mcorbett51090/RavenClaude/blob/main/plugins/frontend-engineering/best-practices/code-split-by-route-and-lazy-load-heavy-components.md