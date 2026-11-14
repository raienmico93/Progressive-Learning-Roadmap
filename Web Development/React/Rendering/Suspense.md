# React Suspense — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** `Suspense` is a React component that lets you declaratively display a fallback UI (such as a loading spinner or skeleton) until its children have finished loading.

**Technical Definition:** `<Suspense>` is a React boundary component that catches suspended rendering in its subtree and displays a fallback instead. A component "suspends" when it throws a Promise during render — React catches the Promise, renders the nearest `Suspense` boundary's `fallback`, and retries rendering the suspended tree once the Promise resolves. Suspense integrates with `React.lazy` (code splitting), `use()` (reading Promises in React 19), Suspense-enabled data frameworks (Relay, Next.js), and streaming server rendering with selective hydration. Each `Suspense` boundary defines a unit of content that appears together when its data is ready.

**Beginner-Friendly Explanation:** Suspense is React's built-in loading boundary. You wrap a part of your UI in `<Suspense>` and give it a fallback — a spinner, a skeleton, or any placeholder. If that part of the UI is not ready yet (because its code is still downloading, or its data is still loading), React shows the fallback instead. When the content is ready, React swaps the fallback for the real content. You do not have to write `isLoading` state or `useEffect` fetching logic — Suspense handles the loading coordination declaratively.

### Key Characteristics

- **Declarative loading states:** You declare what to show while loading, not how to manage loading state manually.
- **Boundary-based:** Each `Suspense` boundary catches suspensions from its subtree; the nearest boundary wins.
- **Fallback-driven:** The `fallback` prop accepts any React node — spinners, skeletons, or even `null`.
- **State-loss on initial suspension:** If a tree suspends before it ever mounts, React discards the partial render and retries from scratch when ready.
- **Re-suspension shows fallback again:** If already-visible content suspends again, the fallback reappears unless the update was caused by `startTransition` or `useDeferredValue`.
- **Batched reveals:** React reveals suspended content at most once every 300ms, batching boundaries that become ready together.
- **SSR and streaming integration:** Suspense powers streaming server rendering and selective hydration under the hood.

### Prerequisites

- React function components, JSX, and props.
- Understanding of Promises and asynchronous JavaScript.
- Familiarity with `React.lazy` for code splitting.
- Basic knowledge of React 19's `use()` Hook (for data fetching).
- Understanding of error boundaries (for handling rejected Promises).

### Related Programming Areas

- **Code Splitting** — `React.lazy` with Suspense for lazy-loaded components.
- **Data Fetching** — `use()` Hook and Suspense-enabled frameworks (Next.js, Relay).
- **Streaming SSR and Selective Hydration** — Suspense boundaries define streamable and hydratable chunks.
- **Concurrent Rendering** — Suspense works with `startTransition` and `useDeferredValue` to avoid fallback flashes.
- **Error Boundaries** — Suspense handles loading; error boundaries handle failures.

---

## 1. Suspense

### Definitions

**Core Definition:** Suspense is React's mechanism for declaratively specifying a loading state while waiting for asynchronous operations to complete.

**Technical Definition:** Suspense is a React component and rendering primitive that enables components to "suspend" rendering while waiting for asynchronous data to load. When a component throws a Promise during render, React catches it, activates the nearest `Suspense` boundary, and renders the boundary's `fallback`. React retries rendering the suspended tree once the Promise resolves. Suspense is not a data-fetching library — it is a coordination mechanism that works with Suspense-enabled data sources.

**Beginner-Friendly Explanation:** Suspense is React's way of saying "show this while that is loading." You wrap content in a Suspense boundary, give it a fallback, and React handles the rest.

### Purposes

- To declaratively specify loading states for asynchronous operations.
- To coordinate code loading (`React.lazy`) and data loading (`use()`, frameworks) with UI rendering.
- To enable streaming server rendering and selective hydration.
- To keep the UI responsive by showing meaningful placeholders instead of blank screens.
- To avoid manual `isLoading` state management for Suspense-enabled sources.

### Syntax Rules and Structure

**General Syntax:**

```jsx
import { Suspense } from 'react';

<Suspense fallback={<Loading />}>
  <SomeComponent />
</Suspense>
```

**Component Breakdown:**

| Prop | Description |
|------|-------------|
| `children` | The actual UI you intend to render. If it suspends, the fallback is shown. |
| `fallback` | An alternate UI to render if children have not finished loading. Any valid React node. |
| `defer` | (Experimental) Boolean; when `true`, React may show fallback first and stream children later. |

**Syntax Rules:**

1. `Suspense` is imported from `react`.
2. The `fallback` prop is required when using Suspense for loading states.
3. Suspense only activates for Suspense-enabled sources: `lazy()`, `use()`, Suspense-enabled frameworks, and Suspense-enabled stylesheets.
4. If the fallback itself suspends, React activates the closest parent Suspense boundary.
5. Suspense does not detect data fetching inside `useEffect` or event handlers.

**Constraints and Limitations:**

- Suspense does not preserve state for renders that suspend before first mount; React retries from scratch.
- If already-visible content suspends again, the fallback reappears unless the update was a transition or deferred value.
- React reveals suspended content at most once every 300ms, batching ready boundaries.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Basic Suspense Boundary

```jsx
import { Suspense } from 'react';
import Albums from './Albums';

function ArtistPage({ artist }) {
  return (
    <>
      <h1>{artist.name}</h1>
      <Suspense fallback={<Loading />}>
        <Albums artistId={artist.id} />
      </Suspense>
    </>
  );
}

function Loading() {
  return <h2>Loading albums…</h2>;
}
```

**Expected Output:** The artist name renders immediately. The albums section shows "Loading albums…" until the data or code is ready, then renders the albums.

**Why this output:** `Albums` suspends while its data loads. The nearest `Suspense` boundary renders `Loading` as the fallback. When the Promise resolves, React retries rendering `Albums` with data and replaces the fallback.

#### Example 2: Suspense with Re-suspension Behaviour

```jsx
function App() {
  const [query, setQuery] = useState('');
  return (
    <Suspense fallback={<ResultsSkeleton />}>
      <SearchResults query={query} />
    </Suspense>
  );
}
```

**Expected Output:** Changing `query` causes `SearchResults` to suspend again. The skeleton reappears until the new results are ready — unless the update is wrapped in `startTransition`.

**Why this output:** If already-visible content suspends again, React shows the fallback again. To avoid the fallback flash, wrap the state update in `startTransition`.

### Real-World Cases

- **Route-level loading:** Showing a page skeleton while route data loads.
- **Widget boundaries:** Independent loading states for dashboard widgets.
- **Modal content:** Deferring modal content until opened.

### References

- React — `<Suspense>` — https://react.dev/reference/react/Suspense
- React — `<Suspense>` (React 18) — https://18.react.dev/reference/react/Suspense
- React — Suspense (Mintlify) — https://mintlify.wiki/facebook/react/api/suspense

---

## 2. `Suspense` Component

### Definitions

**Core Definition:** The `Suspense` component is the React element that wraps a subtree and provides a fallback UI while that subtree loads.

**Technical Definition:** `<Suspense>` is a React component (internally represented by `Symbol.for('react.suspense')`) that establishes a boundary for suspended rendering. It accepts `children` and a `fallback`. When any component in `children` throws a Promise during render, React catches the Promise, renders the `fallback`, and retries the suspended tree when the Promise resolves. The component does not fetch data itself; it coordinates with Suspense-enabled sources.

**Beginner-Friendly Explanation:** The `Suspense` component is the actual wrapper you put around content. It is the "loading box" that holds a fallback and the real content.

### Purposes

- To wrap a subtree and provide a fallback during loading.
- To establish a boundary for Suspense-enabled sources.
- To coordinate loading states across a group of components.
- To enable streaming and selective hydration when used with SSR.

### Syntax Rules and Structure

```jsx
<Suspense fallback={<Loading />}>
  <Children />
</Suspense>
```

**Syntax Rules:**

1. The `fallback` prop accepts any valid React node, including `null`.
2. The `children` can be any React node.
3. If `fallback` suspends, the closest parent Suspense boundary activates.
4. The `defer` prop is experimental and only for expensive-to-render content.

### Annotated Code Examples

```jsx
import { Suspense } from 'react';

function Page() {
  return (
    <Suspense fallback={<PageSkeleton />}>
      <Header />
      <MainContent />
      <Footer />
    </Suspense>
  );
}
```

**Expected Output:** The entire page shows `PageSkeleton` until all three children are ready, then reveals them together.

**Why this output:** A single Suspense boundary groups all children into one unit. The boundary waits for all suspended children before revealing content.

### Real-World Cases

- **Page-level boundaries:** Wrapping an entire route's content.
- **Section boundaries:** Wrapping independent sections of a layout.
- **Component boundaries:** Wrapping a single lazy-loaded component.

### References

- React — `<Suspense>` — https://react.dev/reference/react/Suspense

---

## 3. Fallback UI

### Definitions

**Core Definition:** Fallback UI is the alternate content that React renders in place of suspended children until they are ready.

**Technical Definition:** The `fallback` prop is any valid React node that React renders when the Suspense boundary's children suspend. It is automatically swapped back to the real children when loading completes. A well-designed fallback matches the dimensions of the eventual content to prevent Cumulative Layout Shift (CLS). Fallback UI should be a lightweight placeholder — a spinner, skeleton, or loading message — not a blank screen or a jarring placeholder.

**Beginner-Friendly Explanation:** Fallback UI is what the user sees while waiting. It should look like a placeholder for the real content, so the page does not jump when the content arrives.

### Purposes

- To provide meaningful visual feedback while content loads.
- To prevent blank screens and layout shifts.
- To communicate loading state to users.
- To match the dimensions of eventual content to prevent CLS.

### Syntax Rules and Structure

```jsx
<Suspense fallback={<ContentSkeleton />}>
  <Content />
</Suspense>
```

**Syntax Rules:**

1. The fallback can be any React node: spinner, skeleton, text, or `null`.
2. The fallback should visually approximate the final content's size and shape.
3. The fallback should be lightweight; heavy fallbacks delay the initial render.
4. If the fallback suspends, React activates the parent Suspense boundary.

**Constraints and Limitations:**

- A mismatch between fallback and content dimensions causes layout shift.
- A fallback that is too generic (e.g., full-page spinner) hides fast content unnecessarily.
- An empty fallback (`null`) provides no feedback and can feel broken.

### Annotated Code Examples

```jsx
// ✅ Skeleton matching content dimensions
function RevenueChart() {
  return (
    <div className="chart-container">
      {/* Chart renders here */}
    </div>
  );
}

function RevenueChartSkeleton() {
  return <div className="chart-container skeleton" />;
}

<Suspense fallback={<RevenueChartSkeleton />}>
  <RevenueChart />
</Suspense>
```

**Expected Output:** The skeleton occupies the same space as the chart, preventing layout shift when the chart loads.

**Why this output:** The skeleton and the chart share the same CSS class (`chart-container`) and dimensions, so the layout does not jump when content replaces the fallback.

### Real-World Cases

- **Data tables:** Skeleton rows matching table structure.
- **Charts:** Placeholder charts with matching dimensions.
- **Content cards:** Skeleton cards matching card layout.

### References

- Vercel — Strategic Suspense Boundaries — https://github.com/vercel-labs/agent-skills
- pproenca/dot-skills — Place Suspense Boundaries Around Independently-Paced Subtrees — https://github.com/pproenca/dot-skills
- vanguard-frontier-agentic — Suspense Streaming Structure — https://github.com/VincentChuWaiChow/vanguard-frontier-agentic

---

## 4. Suspense Boundaries

### Definitions

**Core Definition:** A Suspense boundary is the scope of a `Suspense` component — the subtree it wraps and the fallback it displays when that subtree suspends.

**Technical Definition:** A Suspense boundary defines a unit of content that appears together when its data is ready. Each boundary catches suspensions from its subtree; the nearest boundary above the suspending component activates. Boundary placement determines granularity: one page-level boundary means everything waits for the slowest part; many granular boundaries mean fast content appears immediately while slow content streams in independently. Best practice is to place boundaries around **independently-paced subtrees** — not one per page (too coarse) and not around every JSX element (too fine).

**Beginner-Friendly Explanation:** A Suspense boundary is the "loading zone" you define. It decides what content appears together and what fallback shows while waiting. If you put one boundary around the whole page, everything waits for the slowest part. If you put boundaries around each section, each section appears as soon as it is ready.

### Purposes

- To define a unit of content that appears together.
- To control loading granularity and perceived performance.
- To isolate slow sections from fast content.
- To prevent one slow component from blocking the entire page.

### Syntax Rules and Structure

**Boundary Granularity:**

| Placement | Effect |
|-----------|--------|
| Page-level | Entire page waits for slowest part |
| Section-level | Each section streams independently |
| Component-level | Individual components load independently |

**Syntax Rules:**

1. Place boundaries around independently-paced async leaves.
2. Group related components that should appear together in a single boundary.
3. Static content should stay outside Suspense boundaries.
4. Fallbacks should match the dimensions of the eventual content.

**Constraints and Limitations:**

- A page-level boundary causes the entire page to wait for the slowest data.
- Too many boundaries add noise without benefit.
- A boundary inside an async component where the parent already awaited never shows its fallback.

### Annotated Code Examples

**❌ Page-level boundary (everything waits):**

```jsx
<Suspense fallback={<FullPageLoader />}>
  <Dashboard />
</Suspense>
```

**✅ Granular boundaries (each section streams independently):**

```jsx
<div className="dashboard-grid">
  <Header /> {/* Static, outside Suspense */}
  <Suspense fallback={<StatsSkeleton />}>
    <StatsWidget />
  </Suspense>
  <Suspense fallback={<ChartSkeleton />}>
    <RevenueChart />
  </Suspense>
  <Suspense fallback={<TableSkeleton />}>
    <RecentOrders />
  </Suspense>
</div>
```

**Expected Output:** The header renders immediately. Each widget streams in as soon as its data is ready, independently of the others.

**Why this output:** Each Suspense boundary wraps one independently-paced async leaf. Fast widgets appear immediately; slow widgets do not block the others.

### Real-World Cases

- **Dashboards:** Independent boundaries per widget.
- **E-commerce product pages:** Separate boundaries for product info, reviews, and recommendations.
- **Content sites:** Separate boundaries for article body and comments.

### References

- pproenca/dot-skills — Place Suspense Boundaries Around Independently-Paced Subtrees — https://github.com/pproenca/dot-skills
- Vercel — Strategic Suspense Boundaries — https://github.com/vercel-labs/agent-skills
- Educative — Nested Suspense Boundaries — https://www.educative.io

---

## 5. Nested Suspense

### Definitions

**Core Definition:** Nested Suspense is the practice of placing Suspense boundaries inside other Suspense boundaries to create progressive, cascading loading states.

**Technical Definition:** Suspense boundaries can be nested to any depth. Inner boundaries catch suspensions first; if an inner boundary's fallback also suspends, the parent boundary activates. Nested boundaries enable progressive rendering: a stable parent frame (title, layout, controls) can be committed while a sub-region continues loading. This avoids making the whole feature wait for the slowest dependency.

**Beginner-Friendly Explanation:** Nested Suspense is like having loading zones inside loading zones. The outer zone shows a page skeleton, and inner zones show their own section skeletons. As each inner section loads, it replaces its skeleton while the rest of the page stays visible.

### Purposes

- To commit a stable parent frame while a sub-region continues loading.
- To create cascading, progressive loading states.
- To avoid blocking the entire page on a slow sub-region.
- To match the hierarchical structure of the UI.

### Syntax Rules and Structure

```jsx
<Suspense fallback={<PageSkeleton />}>
  <Header />
  <Suspense fallback={<ContentSkeleton />}>
    <MainContent />
  </Suspense>
  <Suspense fallback={<SidebarSkeleton />}>
    <Sidebar />
  </Suspense>
</Suspense>
```

**Syntax Rules:**

1. Inner boundaries catch suspensions first.
2. If an inner fallback suspends, the parent boundary activates.
3. The parent boundary can provide a broader fallback if inner content is not ready.
4. Nested boundaries should align with independently-paced UI regions.

**Constraints and Limitations:**

- Deep nesting can cause sequential loading (a "waterfall") if inner components depend on outer data.
- Nested boundaries that are too granular add complexity without benefit.
- A parent boundary with a full-page spinner hides inner loading states.

### Annotated Code Examples

```jsx
function App() {
  return (
    <Suspense fallback={<FullPageSpinner />}>
      <Layout>
        <Suspense fallback={<ArticleSkeleton />}>
          <ArticleContent />
        </Suspense>
        <Suspense fallback={<SidebarSkeleton />}>
          <RelatedArticles />
        </Suspense>
      </Layout>
    </Suspense>
  );
}
```

**Expected Output:** The layout renders immediately. `ArticleContent` and `RelatedArticles` show their own skeletons and load independently. If the layout itself suspends, `FullPageSpinner` appears.

**Why this output:** Inner boundaries catch their own suspensions, allowing the layout to remain stable while content loads. The outer boundary only activates if the layout itself is not ready.

### Real-World Cases

- **Article pages:** Article body and sidebar loading independently.
- **Dashboards:** KPI widgets and charts loading at different speeds.
- **Social feeds:** Post content and comments loading separately.

### References

- React — Suspense (Mintlify) — https://mintlify.wiki/facebook/react/api/suspense
- Educative — Nested Suspense Boundaries — https://www.educative.io
- Stack Overflow — Preventing Suspense Waterfall — https://stackoverflow.com

---

## 6. Suspense with Lazy

### Definitions

**Core Definition:** Suspense with `lazy` is the pattern of wrapping lazily loaded components (`React.lazy`) in a Suspense boundary to show a fallback while the component's code downloads.

**Technical Definition:** `React.lazy(() => import('./Component'))` creates a component that suspends while its code chunk is being fetched. Wrapping it in `<Suspense fallback={...}>` displays a fallback during the download. When the chunk arrives, React retries rendering and replaces the fallback with the component. This is the most common use of Suspense and the foundation of route-based code splitting.

**Beginner-Friendly Explanation:** Lazy loading means downloading a component's code only when it is needed. Suspense makes that safe: it shows a fallback while the code is downloading, then displays the component when it arrives.

### Purposes

- To lazy-load component code with a declarative loading state.
- To enable route-based and component-based code splitting.
- To reduce the initial bundle size without sacrificing user feedback.
- To defer heavy components until they are actually rendered.

### Syntax Rules and Structure

```jsx
import { lazy, Suspense } from 'react';

const LazyComponent = lazy(() => import('./Component'));

<Suspense fallback={<Loading />}>
  <LazyComponent />
</Suspense>
```

**Syntax Rules:**

1. `lazy` requires a function returning a dynamic `import()`.
2. The imported module must have a `default` export.
3. The lazy component must be inside a Suspense boundary.
4. Provide a meaningful fallback (skeleton, not blank).

**Constraints and Limitations:**

- If the lazy component's chunk fails to load, the error propagates to the nearest error boundary.
- Lazy components cannot be server-rendered directly without framework support.
- Too many lazy boundaries can cause a waterfall of sequential loads.

### Annotated Code Examples

```jsx
import { lazy, Suspense } from 'react';

const HeavyChart = lazy(() => import('./HeavyChart'));

function Dashboard() {
  return (
    <Suspense fallback={<ChartSkeleton />}>
      <HeavyChart />
    </Suspense>
  );
}
```

**Expected Output:** The chart's chunk downloads when `Dashboard` renders. While downloading, `ChartSkeleton` is shown. Once loaded, the chart appears.

**Why this output:** `lazy` suspends until the dynamic import resolves. Suspense shows the fallback. When the chunk arrives, React renders the chart and removes the fallback.

### Real-World Cases

- **Route components:** Each route loads its own chunk.
- **Modals and editors:** Loading heavy modal content only when opened.
- **Charts and data grids:** Deferring large third-party libraries.

### References

- React — `<Suspense>` — https://react.dev/reference/react/Suspense
- React — lazy — https://18.react.dev/reference/react/lazy
- CoreUI — How to use Suspense in React — https://coreui.io/answers/how-to-use-suspense-in-react/

---

## 7. Suspense with Data Fetching

### Definitions

**Core Definition:** Suspense with data fetching is the pattern of using Suspense boundaries to coordinate loading states for asynchronous data, typically via React 19's `use()` Hook or a Suspense-enabled framework.

**Technical Definition:** Suspense-enabled data fetching allows a component to suspend while waiting for a Promise to resolve. In React 19, the `use()` Hook accepts a Promise and suspends the component until it resolves. The parent creates the Promise (memoized to avoid re-fetching on re-render) and passes it to the child. The child calls `use(promise)` and renders with the resolved data as if it were synchronous. If the Promise rejects, the error propagates to the nearest error boundary. Suspense does not detect data fetching inside `useEffect` or event handlers.

**Beginner-Friendly Explanation:** Instead of manually managing `isLoading`, `error`, and `data` state, you let your component "pause" while data loads. Suspense shows the fallback. When data arrives, the component renders. React 19's `use()` Hook makes this possible without a framework.

### Purposes

- To declaratively coordinate data loading with UI rendering.
- To eliminate manual `isLoading` state and `useEffect` boilerplate.
- To delegate loading states to Suspense and error states to error boundaries.
- To enable parallel data fetching across independent boundaries.
- To integrate with Suspense-enabled frameworks (Next.js, Relay).

### Syntax Rules and Structure

```jsx
import { use, Suspense } from 'react';

function UserProfile({ userPromise }) {
  const user = use(userPromise); // Suspends until resolved
  return <h1>{user.name}</h1>;
}

function App() {
  const userPromise = useMemo(() => fetchUser(1), []);
  return (
    <Suspense fallback={<ProfileSkeleton />}>
      <UserProfile userPromise={userPromise} />
    </Suspense>
  );
}
```

**Syntax Rules:**

1. Create Promises outside render or memoize them with `useMemo` to avoid re-fetching on every render.
2. `use(promise)` suspends the component; wrap it in Suspense.
3. Rejected Promises propagate to the nearest error boundary.
4. `use()` is exempt from the top-level-only rule and can be called conditionally.
5. Suspense does not detect fetching inside `useEffect` or event handlers.

**Constraints and Limitations:**

- Without a Suspense-enabled framework, `use()` requires manually creating and passing Promises.
- Creating a new Promise on every render causes infinite suspension loops.
- Suspense does not manage caching; you must cache Promises (e.g., via a framework or a cache layer).

### Annotated Code Examples

```jsx
import { use, useMemo, Suspense } from 'react';

function App() {
  const userPromise = useMemo(() => fetchUser(1), []);
  return (
    <Suspense fallback={<p>Loading profile…</p>}>
      <UserProfile userPromise={userPromise} />
    </Suspense>
  );
}

function UserProfile({ userPromise }) {
  const user = use(userPromise);
  return <h1>{user.name}</h1>;
}
```

**Expected Output:** The profile section shows "Loading profile…" until the Promise resolves, then displays the user's name.

**Why this output:** `use()` suspends `UserProfile` until `userPromise` resolves. Suspense shows the fallback. When the Promise resolves, React retries and renders the name.

### Real-World Cases

- **User profiles:** Loading user data with Suspense.
- **Product pages:** Loading product details and reviews in parallel boundaries.
- **Dashboards:** Loading independent widgets from different APIs.

### References

- Cybrosys — How to Use the New use() API in React 19 — https://www.cybrosys.com/blog/how-to-use-the-new-use-api-in-react-19-for-smarter-data-fetching
- SitePoint — React 19 use() Hook: Data Fetching Patterns That Actually Work — https://www.sitepoint.com/react-19-use-hook-data-fetching-patterns-that-actually-work/
- Steve Kinney — The use() Hook — https://stevekinney.com/courses/react-performance/the-use-hook
- Syncfusion — React 19 Suspense for Data Fetching — https://www.syncfusion.com/blogs/post/react-19-suspense-data-fetching

---

## 8. Suspense with Server Components

### Definitions

**Core Definition:** Suspense with Server Components is the integration of Suspense boundaries with React Server Components (RSC), allowing server-rendered content to stream progressively to the client.

**Technical Definition:** React Server Components are components that render exclusively on the server and ship zero JavaScript to the client. They can fetch data directly on the server and integrate with Suspense for progressive streaming. When a Server Component suspends (e.g., while awaiting data), React streams the shell and other ready content first, then streams the suspended Server Component's HTML when its data resolves. RSC is stable in React 19.

**Beginner-Friendly Explanation:** Server Components render on the server and send HTML to the browser. Suspense lets the server send the page in pieces: the shell arrives immediately, and slow Server Components stream in later as their data becomes ready.

### Purposes

- To stream server-rendered content progressively.
- To keep the shell interactive while slow Server Components load.
- To reduce client-side JavaScript by shipping zero-JS Server Components.
- To coordinate server data fetching with Suspense boundaries.

### Syntax Rules and Structure

```jsx
// Server Component
async function UserProfile({ userId }) {
  const user = await fetchUser(userId); // Fetches on the server
  return <h1>{user.name}</h1>;
}

// Page with Suspense
import { Suspense } from 'react';

function Page() {
  return (
    <>
      <Header />
      <Suspense fallback={<ProfileSkeleton />}>
        <UserProfile userId={1} />
      </Suspense>
    </>
  );
}
```

**Syntax Rules:**

1. Server Components can be `async` and await data directly.
2. Wrap Server Components in Suspense boundaries to stream them.
3. The shell (static content) renders immediately; suspended Server Components stream in.
4. Server Components ship zero JavaScript to the client.

**Constraints and Limitations:**

- Server Components cannot use client-side Hooks like `useState` or `useEffect`.
- Suspense boundaries around Server Components define streamable chunks.
- Framework support (Next.js, etc.) is required for RSC routing and bundling.

### Annotated Code Examples

```jsx
// app/page.tsx (Server Component)
import { Suspense } from 'react';
import UserProfile from './UserProfile';

export default function Page() {
  return (
    <div>
      <h1>Dashboard</h1>
      <Suspense fallback={<ProfileSkeleton />}>
        <UserProfile userId={1} />
      </Suspense>
    </div>
  );
}

// UserProfile.tsx (Server Component)
async function UserProfile({ userId }) {
  const user = await fetchUser(userId);
  return <p>{user.name}</p>;
}
```

**Expected Output:** The dashboard title renders immediately. The profile section streams in once the server-side fetch completes.

**Why this output:** `UserProfile` suspends on the server while fetching. React streams the shell and streams the profile HTML when ready. The client hydrates the shell first.

### Real World Cases

- **Content-heavy pages:** Article body and comments as separate Server Component boundaries.
- **Dashboards:** Server-rendered widgets streaming independently.
- **E-commerce:** Product info and recommendations as separate RSC boundaries.

### References

- React Rendering Architecture: Fiber, Lanes, Streaming SSR, and RSC — https://raw.githubusercontent.com
- Steve Kinney — Server Components and Streaming SSR — https://stevekinney.com/courses/react-typescript/server-components-streaming-ssr
- Steve Kinney — Streaming SSR with TypeScript — https://stevekinney.com/courses/react-typescript/streaming-ssr-typescript

---

## 9. Suspense with Streaming

### Definitions

**Core Definition:** Suspense with streaming is the use of Suspense boundaries to define chunks of HTML that the server streams to the browser as they become ready, rather than waiting for the entire page.

**Technical Definition:** Streaming SSR sends HTML in pieces as they are ready. Each Suspense boundary defines a unit that can stream independently. The server uses `renderToPipeableStream` (Node) or `renderToReadableStream` (Web Streams) to send the shell first, then streams each Suspense boundary's HTML when its content resolves. The client receives the shell immediately and hydrates it while later chunks arrive. This improves Time to First Byte (TTFB) and allows users to see and interact with fast content before slow content arrives.

**Beginner-Friendly Explanation:** Streaming Suspense means the server does not wait for everything before sending the page. It sends the shell right away, then sends each section as it becomes ready. The browser can start showing and hydrating content while the rest is still being prepared.

### Purposes

- To improve TTFB by sending the shell immediately.
- To let users see and interact with fast content while slow content streams in.
- To prevent slow data dependencies from blocking the entire page.
- To enable selective hydration of interactive components before the full page loads.

### Syntax Rules and Structure

**Server:**

```js
import { renderToPipeableStream } from 'react-dom/server';

const { pipe } = renderToPipeableStream(<App />, {
  bootstrapScripts: ['/client.js'],
  onShellReady() {
    res.setHeader('Content-Type', 'text/html');
    pipe(res);
  },
});
```

**Client:**

```js
import { hydrateRoot } from 'react-dom/client';

hydrateRoot(document, <App />);
```

**Syntax Rules:**

1. Use `renderToPipeableStream` (Node) or `renderToReadableStream` (edge).
2. Start piping in `onShellReady` so the shell reaches the browser immediately.
3. Wrap slow sections in Suspense boundaries to stream them separately.
4. Use `hydrateRoot` on the client to enable selective hydration.

**Constraints and Limitations:**

- Requires a server runtime that supports streaming.
- Framework support (Next.js, etc.) is often required for Suspense-enabled data fetching.
- Errors during streaming must be handled with `onError`.

### Annotated Code Examples

```jsx
function App() {
  return (
    <div>
      <Header />
      <Suspense fallback={<UserSkeleton />}>
        <UserProfile /> {/* Streams when ready */}
      </Suspense>
      <Suspense fallback={<PostsSkeleton />}>
        <PostList /> {/* Streams when ready */}
      </Suspense>
      <Footer />
    </div>
  );
}
```

**Expected Output:** The shell (header, footer) is sent immediately. `UserProfile` and `PostList` stream in independently as their data resolves.

**Why this output:** Each Suspense boundary is a streamable chunk. React sends the shell first, then streams each boundary's HTML when ready.

### Real-World Cases

- **E-commerce:** Streaming recommendations and reviews after product info.
- **Content sites:** Streaming comments after the article body.
- **Dashboards:** Streaming widgets as their data loads.

### References

- Steve Kinney — Streaming SSR with TypeScript — https://stevekinney.com/courses/react-typescript/streaming-ssr-typescript
- Steve Kinney — Server Components and Streaming SSR — https://stevekinney.com/courses/react-typescript/server-components-streaming-ssr
- React — `<Suspense>` — https://react.dev/reference/react/Suspense

---

## 10. Suspense Best Practices

### Definitions

**Core Definition:** Suspense best practices are the guidelines for placing boundaries, designing fallbacks, and integrating Suspense with code splitting and data fetching to maximise perceived performance.

**Technical Definition:** Best practices centre on boundary granularity (wrap independently-paced subtrees, not whole pages), fallback design (match dimensions to prevent CLS), error boundary pairing (handle rejected Promises), transition integration (avoid fallback flashes for non-urgent updates), and parallel data fetching (avoid waterfalls by starting independent Promises before rendering). Suspense is a coordination tool, not a data-fetching library; it works best when paired with Suspense-enabled sources.

**Beginner-Friendly Explanation:** Place Suspense boundaries around sections that load independently. Make fallbacks look like the content that will replace them. Use error boundaries for failures. Use transitions to avoid fallback flashes. And fetch data in parallel, not one after another.

### Best Practices Table

| Practice | Description |
|----------|-------------|
| **Granular boundaries** | Wrap independently-paced subtrees, not whole pages. |
| **Matching fallbacks** | Fallbacks should match content dimensions to prevent CLS. |
| **Static content outside Suspense** | Headers, footers, and layout should render immediately. |
| **Pair with error boundaries** | Handle rejected Promises and failed loads gracefully. |
| **Use transitions for non-urgent updates** | Avoid fallback flashes when updating visible content. |
| **Fetch data in parallel** | Start independent Promises before rendering to avoid waterfalls. |
| **Prioritise above-the-fold content** | Place critical boundaries earlier in the tree. |
| **Use `useMemo` for Promises** | Avoid creating new Promises on every render. |

### Annotated Code Examples

**✅ Good Practices:**

```jsx
// Granular boundaries, matching skeletons, parallel fetching
function Dashboard() {
  const statsPromise = useMemo(() => fetchStats(), []);
  const chartPromise = useMemo(() => fetchChart(), []);

  return (
    <div className="grid">
      <Suspense fallback={<StatsSkeleton />}>
        <StatsWidget promise={statsPromise} />
      </Suspense>
      <Suspense fallback={<ChartSkeleton />}>
        <RevenueChart promise={chartPromise} />
      </Suspense>
    </div>
  );
}
```

**❌ Bad Practices:**

```jsx
// ❌ One page-level boundary: everything waits
<Suspense fallback={<FullPageSpinner />}>
  <Dashboard />
</Suspense>

// ❌ Fallback mismatch: layout shift
<Suspense fallback={<div />}> {/* Empty fallback */}
  <HeavyChart />
</Suspense>

// ❌ Sequential fetching: waterfall
const user = use(fetchUser());
const posts = use(fetchPosts(user.id)); // Waits for user
```

**Expected Output:** The good practices produce independent loading, stable layout, and fast perceived performance. The bad practices cause full-page blocking, layout shift, and waterfalls.

**Why this output:** Granular boundaries let fast content appear immediately. Matching fallbacks prevent CLS. Parallel fetching avoids sequential delays.

### Real-World Cases

- **Dashboards:** Granular widget boundaries with matching skeletons.
- **E-commerce:** Parallel product and recommendation fetching.
- **Content sites:** Separate article and comment boundaries.

### References

- Vercel — Strategic Suspense Boundaries — https://github.com/vercel-labs/agent-skills
- pproenca/dot-skills — Place Suspense Boundaries Around Independently-Paced Subtrees — https://github.com/pproenca/dot-skills
- SitePoint — React 19 use() Hook: Data Fetching Patterns — https://www.sitepoint.com/react-19-use-hook-data-fetching-patterns-that-actually-work/
- Syncfusion — React 19 Suspense for Data Fetching — https://www.syncfusion.com/blogs/post/react-19-suspense-data-fetching

---

## 11. Suspense Pitfalls

### Definitions

**Core Definition:** Suspense pitfalls are common mistakes that cause unnecessary loading states, layout shifts, waterfalls, or broken error handling when using Suspense.

**Technical Definition:** Common pitfalls include: placing one page-level boundary that blocks everything on the slowest part; using fallbacks that do not match content dimensions (causing CLS); nesting boundaries that create sequential waterfalls; creating new Promises on every render (causing infinite suspension loops); using Suspense with data fetching in `useEffect` (which Suspense does not detect); forgetting error boundaries for rejected Promises; and wrapping non-Suspense-enabled sources in Suspense without effect.

**Beginner-Friendly Explanation:** These are the traps that make Suspense ineffective or harmful. The biggest one is putting one boundary around the whole page, so nothing appears until the slowest data is ready. Another is forgetting that Suspense only works with Suspense-enabled sources — wrapping a `useEffect` fetch in Suspense does nothing.

### Common Pitfalls Table

| Pitfall | Problem | Solution |
|---------|---------|----------|
| **Page-level boundary** | Everything waits for the slowest part | Use granular boundaries per section. |
| **Fallback/content mismatch** | Layout shift (CLS) when content loads | Match fallback dimensions to content. |
| **Suspense waterfall** | Nested boundaries load sequentially | Fetch data in parallel; avoid dependent boundaries. |
| **New Promise every render** | Infinite suspension loop | Memoize Promises with `useMemo` or cache them. |
| **Suspense with `useEffect` fetching** | Suspense does not detect the fetch | Use `use()` or a Suspense-enabled framework. |
| **No error boundary** | Rejected Promises crash the app | Pair Suspense with error boundaries. |
| **Empty fallback** | No loading feedback; feels broken | Use a meaningful skeleton or spinner. |

### Annotated Code Examples

**Pitfall 1: Page-level boundary**

```jsx
// ❌ Everything waits for the slowest widget
<Suspense fallback={<FullPageSpinner />}>
  <Dashboard />
</Suspense>

// ✅ Granular boundaries
<Suspense fallback={<StatsSkeleton />}><StatsWidget /></Suspense>
<Suspense fallback={<ChartSkeleton />}><RevenueChart /></Suspense>
```

**Pitfall 2: New Promise every render**

```jsx
// ❌ Infinite suspension loop
function UserProfile() {
  const user = use(fetchUser()); // New Promise on every render
  return <h1>{user.name}</h1>;
}

// ✅ Memoized Promise
function UserProfile({ userPromise }) {
  const user = use(userPromise);
  return <h1>{user.name}</h1>;
}
```

**Pitfall 3: Suspense with `useEffect`**

```jsx
// ❌ Suspense does not detect this fetch
function BadComponent() {
  const [data, setData] = useState(null);
  useEffect(() => { fetchData().then(setData); }, []);
  if (!data) return <Skeleton />;
  return <div>{data}</div>;
}

// ✅ Use Suspense-enabled source
function GoodComponent({ dataPromise }) {
  const data = use(dataPromise);
  return <div>{data}</div>;
}
```

**Expected Output:** The correct versions produce independent loading, stable layout, and proper error handling. The incorrect versions cause full-page blocking, infinite loops, or undetected loading.

**Why this output:** Page-level boundaries block everything. New Promises cause re-suspension on every render. `useEffect` fetching is invisible to Suspense.

### Real-World Cases

- **Dashboards:** Page-level boundaries are the most common Suspense mistake.
- **Data fetching:** Creating Promises inline is a frequent source of infinite loops.
- **Error handling:** Missing error boundaries cause crashes on rejected Promises.

### References

- Vercel — Strategic Suspense Boundaries — https://github.com/vercel-labs/agent-skills
- SitePoint — React 19 use() Hook: Data Fetching Patterns (Common Pitfalls) — https://www.sitepoint.com/react-19-use-hook-data-fetching-patterns-that-actually-work/
- React — `<Suspense>` Caveats — https://react.dev/reference/react/Suspense

---

## References

- React — `<Suspense>` — https://react.dev/reference/react/Suspense
- React — `<Suspense>` (React 18) — https://18.react.dev/reference/react/Suspense
- React — Suspense (Mintlify) — https://mintlify.wiki/facebook/react/api/suspense
- React — lazy — https://18.react.dev/reference/react/lazy
- Vercel — Strategic Suspense Boundaries — https://github.com/vercel-labs/agent-skills/blob/react-best-practices/skills/react-best-practices/references/rules/async-suspense-boundaries.md
- pproenca/dot-skills — Place Suspense Boundaries Around Independently-Paced Subtrees — https://github.com/pproenca/dot-skills/blob/HEAD/skills/.curated/nextjs/references/stream-suspense-boundaries.md
- vanguard-frontier-agentic — Suspense Streaming Structure — https://github.com/VincentChuWaiChow/vanguard-frontier-agentic/blob/master/frontend/ssr-hydration-streaming-diagnosis/references/suspense-streaming-structure.md
- Cybrosys — How to Use the New use() API in React 19 for Smarter Data Fetching — https://www.cybrosys.com/blog/how-to-use-the-new-use-api-in-react-19-for-smarter-data-fetching
- SitePoint — React 19 use() Hook: Data Fetching Patterns That Actually Work — https://www.sitepoint.com/react-19-use-hook-data-fetching-patterns-that-actually-work/
- Steve Kinney — The use() Hook — https://stevekinney.com/courses/react-performance/the-use-hook
- Steve Kinney — Streaming SSR with TypeScript — https://stevekinney.com/courses/react-typescript/streaming-ssr-typescript
- Steve Kinney — Server Components and Streaming SSR — https://stevekinney.com/courses/react-typescript/server-components-streaming-ssr
- Syncfusion — React 19 Suspense for Data Fetching: A New Model for Async UI — https://www.syncfusion.com/blogs/post/react-19-suspense-data-fetching
- CoreUI — How to use Suspense in React — https://coreui.io/answers/how-to-use-suspense-in-react/
- Educative — Nested Suspense Boundaries — https://www.educative.io
- React Rendering Architecture: Fiber, Lanes, Streaming SSR, and RSC — https://raw.githubusercontent.com