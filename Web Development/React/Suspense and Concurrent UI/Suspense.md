# React Suspense Concepts: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** React Suspense is a built-in React component that declaratively manages asynchronous loading states by displaying a fallback UI until its children have finished loading.

**Technical Definition:** `<Suspense>` is a React component that accepts `children` and a `fallback` prop. When a component in the `children` tree suspends during rendering—either because it is lazily loaded via `React.lazy`, reads a pending Promise via `use()`, or uses a Suspense-enabled data source—the closest parent `<Suspense>` boundary renders its `fallback` instead of the children. Once the asynchronous operation completes, React retries rendering the suspended subtree from scratch and replaces the fallback with the actual content. Suspense integrates with React's concurrent rendering features, including streaming server rendering and selective hydration.

**Beginner-Friendly Explanation:** Imagine you're building a webpage that needs to load data from a server. Instead of showing a blank screen while you wait, Suspense lets you show a nice loading spinner or skeleton screen. You wrap the part of your app that might need to wait inside a `<Suspense>` component and tell it what to show while loading. When the data arrives, React automatically swaps the loading placeholder with the real content.

### Key Characteristics

- **Declarative Loading:** You describe *what* to show while loading, not *when* to show it.
- **Boundary-Based:** Each `<Suspense>` boundary isolates a portion of the UI tree. When a component suspends, only the closest boundary above it shows a fallback.
- **Nestable:** Multiple Suspense boundaries can be nested to create granular loading sequences, allowing fast sections to appear while slow sections continue loading.
- **Automatic Retry:** React automatically retries rendering the suspended subtree when the underlying Promise resolves.
- **Integration with Concurrent Features:** Suspense works with `startTransition`, `useDeferredValue`, streaming server rendering, and selective hydration.
- **Promise-Throwing Mechanism:** Suspense works by catching thrown Promises. When a component needs data that isn't ready, it throws a Promise; React catches it and shows the fallback.

### Prerequisites

- Solid understanding of React components, props, and state.
- Familiarity with React Hooks (`useState`, `useEffect`).
- Basic understanding of JavaScript Promises and async/await.
- Knowledge of React.lazy for code-splitting (for basic usage).
- Familiarity with Error Boundaries for handling rejected Promises.

### Related Programming Areas

- **Concurrent Rendering:** React's ability to prepare multiple versions of the UI simultaneously.
- **Code Splitting:** Loading JavaScript bundles on demand.
- **Server-Side Rendering (SSR):** Streaming HTML from the server with progressive hydration.
- **React Server Components (RSC):** Server-rendered components that integrate with Suspense for streaming.
- **Data Fetching Libraries:** TanStack Query, Relay, SWR, and framework-specific data layers.
- **Error Handling:** Error Boundaries for catching rendering errors and rejected Promises.

### Core Concepts / Features

1. Suspense Boundaries
2. Loading Fallbacks
3. Suspense for Data Fetching
4. Cascading & Parallel Loading
5. Error Boundaries Collaboration

---

## Core Concept 1: Suspense Boundaries

### Definitions

**Core Definition:** A Suspense boundary is a `<Suspense>` component that wraps a portion of the UI tree and controls what is displayed while its children are loading.

**Technical Definition:** A Suspense boundary is created by rendering a `<Suspense>` component with a `fallback` prop. When any component within the boundary's `children` tree suspends during rendering, React unwinds to the nearest Suspense boundary above the suspended component and renders its `fallback` instead of the children. When the suspended operation completes, React retries rendering the entire subtree from scratch. Suspense boundaries can be nested, with each boundary handling the loading state for its own subtree independently. If a `fallback` itself suspends, it activates the closest parent Suspense boundary.

**Beginner-Friendly Explanation:** A Suspense boundary is like a "loading zone" you draw around part of your app. If anything inside that zone needs to wait for data, React shows the loading placeholder you defined for that zone. If you have multiple zones, each one can load independently—one part of the page can show a spinner while another part is already fully loaded.

### Purposes

- To isolate asynchronous loading states to specific regions of the UI tree.
- To prevent a slow-loading component from blocking the entire page.
- To enable granular control over which parts of the UI show loading indicators.
- To create loading sequences where fast content appears before slow content.
- To integrate with streaming server rendering for progressive HTML delivery.
- To coordinate with Error Boundaries for complete async lifecycle handling.

### Syntax Rules and Structure

**General Syntax:**
```jsx
import { Suspense } from 'react';

<Suspense fallback={<Loading />}>
  <SomeComponent />
</Suspense>
```

**Component Breakdown:**
- `<Suspense>`: The React component that creates the boundary.
- `fallback`: A prop that accepts any valid React node. This is rendered when the children suspend.
- `children`: The actual UI you intend to render. If any child suspends, the boundary switches to the fallback.
- `import { Suspense } from 'react'`: Suspense is a named export from the React package.

**Nested Suspense Syntax:**
```jsx
<Suspense fallback={<PageSkeleton />}>
  <Header />
  <Suspense fallback={<SidebarSkeleton />}>
    <Sidebar />
  </Suspense>
  <Suspense fallback={<ContentSkeleton />}>
    <MainContent />
  </Suspense>
</Suspense>
```

**Component Breakdown:**
- Outer `<Suspense>`: Handles the loading state for the entire page shell.
- Inner `<Suspense>` boundaries: Handle loading states for individual sections independently.
- Each boundary's fallback is filled in as the next level of content becomes available.

**Syntax Rules:**
- `<Suspense>` must wrap the component that suspends, not the other way around.
- Only Suspense-enabled data sources activate Suspense: `React.lazy`, `use()` with a Promise, and Suspense-enabled frameworks like Relay and Next.js.
- Suspense does not detect data fetched inside `useEffect` or event handlers.
- The `fallback` prop accepts any React node, but lightweight placeholders (spinners, skeletons) are recommended.
- Nested Suspense boundaries create a loading sequence: the outer boundary shows its fallback until the inner boundaries are ready to render their own fallbacks.

**Constraints and Limitations:**
- React does not preserve state for renders that suspended before mounting for the first time. When the component loads, React retries rendering from scratch.
- If Suspense was already displaying content and then the content suspends again, the fallback will be shown again unless the update was caused by `startTransition` or `useDeferredValue`.
- When React hides already-visible content because it suspended again, it cleans up layout Effects in the content tree. When the content is ready, React fires the layout Effects again.
- Suspense does not catch errors; rejected Promises propagate to the nearest Error Boundary, not the Suspense fallback.

### Annotated Code Examples

**Example 1: Basic Suspense Boundary with React.lazy**

```jsx
import React, { Suspense, lazy } from 'react';

// Lazy-load the component — this creates a Promise that resolves
// when the component's code chunk is downloaded.
const HeavyChart = lazy(() => import('./HeavyChart'));

function Dashboard() {
  return (
    <div>
      <h1>Sales Dashboard</h1>

      {/* Suspense boundary wraps the lazy component */}
      <Suspense fallback={<div>Loading chart...</div>}>
        <HeavyChart />
      </Suspense>

      <p>Other dashboard content loads immediately.</p>
    </div>
  );
}

export default Dashboard;
```

**Expected Output:** The page displays "Sales Dashboard", a "Loading chart..." message, and "Other dashboard content loads immediately." Once the `HeavyChart` component's JavaScript chunk is downloaded and parsed, the loading message is replaced by the chart.

**Why This Output Occurs:** `React.lazy` returns a component that suspends while its code is being loaded. When React attempts to render `<HeavyChart />` inside the Suspense boundary, it throws a Promise. React catches this Promise, renders the `fallback` ("Loading chart...") instead, and waits for the Promise to resolve. When the chunk finishes loading, React retries rendering, and the chart appears. The `<h1>` and `<p>` outside the Suspense boundary render immediately because they do not suspend.

**Example 2: Nested Suspense Boundaries for Granular Loading**

```jsx
import React, { Suspense, lazy } from 'react';

const Header = lazy(() => import('./Header'));
const Sidebar = lazy(() => import('./Sidebar'));
const MainContent = lazy(() => import('./MainContent'));

function App() {
  return (
    <Suspense fallback={<PageSkeleton />}>
      {/* Outer boundary: waits for Header to load */}
      <Header />

      <div style={{ display: 'flex' }}>
        {/* Inner boundary 1: Sidebar loads independently */}
        <Suspense fallback={<SidebarSkeleton />}>
          <Sidebar />
        </Suspense>

        {/* Inner boundary 2: MainContent loads independently */}
        <Suspense fallback={<ContentSkeleton />}>
          <MainContent />
        </Suspense>
      </div>
    </Suspense>
  );
}

// Placeholder components (simplified)
function PageSkeleton() {
  return <div>Loading page layout...</div>;
}
function SidebarSkeleton() {
  return <div>Loading sidebar...</div>;
}
function ContentSkeleton() {
  return <div>Loading content...</div>;
}

export default App;
```

**Expected Output:** The page first shows "Loading page layout..." until the `Header` component loads. Once the header is ready, the outer Suspense boundary renders its children, which include the two inner Suspense boundaries. The sidebar and main content areas then show their own loading placeholders ("Loading sidebar..." and "Loading content...") independently. The sidebar may appear before the main content, or vice versa, depending on which chunk loads first.

**Why This Output Occurs:** The outer Suspense boundary waits for `Header` because it is a direct child. Once `Header` resolves, React renders the full tree including the inner boundaries. Each inner boundary independently handles the loading state for its own child. This creates a loading sequence where the page shell appears first, followed by individual sections as they become available.

### Real-World Cases

- **Dashboard applications:** Wrapping individual widgets (charts, tables, feeds) in separate Suspense boundaries so a slow chart doesn't block the entire dashboard.
- **E-commerce product pages:** Showing the product title and price immediately while reviews and recommendations load in their own boundaries.
- **Social media feeds:** Loading the feed shell with a skeleton while individual posts stream in.
- **Multi-step forms:** Isolating each step's data requirements in separate boundaries.
- **Streaming article pages:** Wrapping each article section in its own Suspense boundary for progressive rendering.

---

## Core Concept 2: Loading Fallbacks

### Definitions

**Core Definition:** A loading fallback is the UI rendered by a Suspense boundary while its children are suspended, typically implemented as a skeleton screen, spinner, or layout placeholder.

**Technical Definition:** The `fallback` prop of `<Suspense>` accepts any valid React node. When the Suspense boundary activates—either because a child suspended during initial render or because a child suspended during an update—React renders the `fallback` in place of the `children`. The fallback should be a lightweight placeholder that approximates the layout and dimensions of the eventual content, minimising layout shift when the real content arrives. A fallback can itself contain other Suspense boundaries, but if a fallback suspends, it activates the closest parent Suspense boundary.

**Beginner-Friendly Explanation:** A loading fallback is what your users see while they wait. Instead of a blank screen or a generic spinner, you can show a "skeleton" that looks like the content that's about to appear—grey boxes where text will be, rectangles where images will go. This makes the waiting experience feel faster and more polished because the page structure is already visible.

### Purposes

- To provide visual feedback that content is loading.
- To reduce perceived loading time by showing an approximate layout.
- To minimise cumulative layout shift (CLS) when content appears.
- To communicate the structure of the incoming content.
- To maintain visual consistency with the application's design language.
- To prevent users from seeing jarring transitions between empty and filled states.

### Syntax Rules and Structure

**General Syntax:**
```jsx
<Suspense fallback={<LoadingSkeleton />}>
  <ContentComponent />
</Suspense>
```

**Component Breakdown:**
- `fallback={<LoadingSkeleton />}`: The `fallback` prop accepts a React element.
- `<LoadingSkeleton />`: A component that renders the placeholder UI.
- The fallback is rendered when `ContentComponent` suspends and replaced when it resolves.

**Skeleton Screen Syntax:**
```jsx
function ArticleSkeleton() {
  return (
    <div className="article-skeleton">
      {/* Title placeholder */}
      <div style={{
        height: '32px',
        width: '60%',
        backgroundColor: '#e0e0e0',
        borderRadius: '4px',
        marginBottom: '16px'
      }} />

      {/* Paragraph placeholders */}
      <div style={{
        height: '16px',
        width: '100%',
        backgroundColor: '#e0e0e0',
        borderRadius: '4px',
        marginBottom: '8px'
      }} />
      <div style={{
        height: '16px',
        width: '90%',
        backgroundColor: '#e0e0e0',
        borderRadius: '4px',
        marginBottom: '8px'
      }} />
      <div style={{
        height: '16px',
        width: '75%',
        backgroundColor: '#e0e0e0',
        borderRadius: '4px'
      }} />
    </div>
  );
}

<Suspense fallback={<ArticleSkeleton />}>
  <Article />
</Suspense>
```

**Component Breakdown:**
- `ArticleSkeleton`: A component that renders grey placeholder blocks matching the expected layout of the article.
- The skeleton uses the same dimensions and spacing as the real content to minimise layout shift.
- When the `Article` component suspends, React renders `ArticleSkeleton` instead.

**Syntax Rules:**
- The fallback should be a React element or component, not a function.
- Fallbacks should be lightweight; avoid heavy computations or data fetching inside fallbacks.
- If the fallback itself suspends, it activates the closest parent Suspense boundary.
- The fallback is rendered in place of the entire `children` tree, not alongside it.
- For the best user experience, the fallback's dimensions should closely match the eventual content's dimensions.

**Constraints and Limitations:**
- Fallbacks cannot be async functions.
- A fallback that suspends will propagate to the parent Suspense boundary, which may show a different fallback.
- Overly complex fallbacks can delay the initial render.
- In StrictMode, fallbacks may render twice in development.

### Annotated Code Examples

**Example 1: Skeleton Screen for a User Profile Card**

```jsx
import React, { Suspense, lazy } from 'react';

const UserProfile = lazy(() => import('./UserProfile'));

function ProfileSkeleton() {
  return (
    <div style={{
      border: '1px solid #ddd',
      borderRadius: '8px',
      padding: '16px',
      maxWidth: '400px'
    }}>
      {/* Avatar placeholder */}
      <div style={{
        width: '80px',
        height: '80px',
        borderRadius: '50%',
        backgroundColor: '#e0e0e0',
        marginBottom: '16px'
      }} />

      {/* Name placeholder */}
      <div style={{
        height: '24px',
        width: '50%',
        backgroundColor: '#e0e0e0',
        borderRadius: '4px',
        marginBottom: '12px'
      }} />

      {/* Bio placeholders */}
      <div style={{
        height: '16px',
        width: '100%',
        backgroundColor: '#e0e0e0',
        borderRadius: '4px',
        marginBottom: '8px'
      }} />
      <div style={{
        height: '16px',
        width: '80%',
        backgroundColor: '#e0e0e0',
        borderRadius: '4px'
      }} />
    </div>
  );
}

function ProfilePage() {
  return (
    <Suspense fallback={<ProfileSkeleton />}>
      <UserProfile userId={123} />
    </Suspense>
  );
}

export default ProfilePage;
```

**Expected Output:** The page displays a grey placeholder card with a circular avatar placeholder, a name bar, and two bio lines. Once the `UserProfile` component loads, the skeleton is replaced by the actual user profile with the avatar, name, and bio.

**Why This Output Occurs:** The `ProfileSkeleton` component renders placeholder divs with grey backgrounds that approximate the layout of the real profile. When `UserProfile` suspends, React renders the skeleton instead. The skeleton uses the same dimensions (80px avatar, similar spacing) as the real content, so when the real content appears, there is minimal layout shift.

**Example 2: Spinner Fallback with Accessibility Considerations**

```jsx
import React, { Suspense, lazy } from 'react';

const DataTable = lazy(() => import('./DataTable'));

function TableSpinner() {
  return (
    <div
      role="status"
      aria-live="polite"
      style={{
        display: 'flex',
        flexDirection: 'column',
        alignItems: 'center',
        justifyContent: 'center',
        padding: '40px'
      }}
    >
      <div style={{
        width: '40px',
        height: '40px',
        border: '4px solid #f3f3f3',
        borderTop: '4px solid #3498db',
        borderRadius: '50%',
        animation: 'spin 1s linear infinite'
      }} />
      <style>{`
        @keyframes spin {
          0% { transform: rotate(0deg); }
          100% { transform: rotate(360deg); }
        }
      `}</style>
      <p style={{ marginTop: '12px', color: '#666' }}>
        Loading data table...
      </p>
    </div>
  );
}

function ReportsPage() {
  return (
    <div>
      <h1>Reports</h1>
      <Suspense fallback={<TableSpinner />}>
        <DataTable reportId="monthly-sales" />
      </Suspense>
    </div>
  );
}

export default ReportsPage;
```

**Expected Output:** The page displays "Reports", followed by a spinning animation and the text "Loading data table...". Once the `DataTable` component loads, the spinner is replaced by the data table.

**Why This Output Occurs:** The `TableSpinner` component includes `role="status"` and `aria-live="polite"` to announce the loading state to screen readers. The CSS animation creates a spinning border effect. When `DataTable` suspends, React renders the spinner. The `aria-live` attribute ensures assistive technologies are notified of the loading state.

### Real-World Cases

- **Content-heavy pages:** Using skeleton screens that mirror the article or blog post layout.
- **Data tables:** Showing placeholder rows while data is fetched.
- **Image galleries:** Displaying grey boxes with the same aspect ratio as the images.
- **Dashboard widgets:** Using widget-specific skeletons that match the chart or metric layout.
- **E-commerce product grids:** Showing product card skeletons while the product list loads.
- **Social media feeds:** Displaying post skeletons with avatar and text placeholders.

---

## Core Concept 3: Suspense for Data Fetching

### Definitions

**Core Definition:** Suspense for data fetching is the integration of Suspense boundaries with data-fetching mechanisms that throw Promises, allowing components to suspend while data is being loaded.

**Technical Definition:** Suspense for data fetching relies on the promise-throwing mechanism: when a component needs data that is not yet available, it throws a Promise. React catches this Promise, pauses rendering of that component subtree, and shows the nearest Suspense fallback. When the Promise resolves, React retries rendering the component with the resolved data. This pattern is supported by React 19's `use()` API, which reads the value of a Promise and integrates with Suspense and Error Boundaries. Suspense-enabled data sources include Relay, Next.js, TanStack Query's `useSuspenseQuery`, and any library that implements the Suspense contract.

**Beginner-Friendly Explanation:** Normally, when you fetch data in React, you use `useEffect` and manage loading states manually with `useState`. Suspense for data fetching flips this model: instead of manually tracking whether data is loading, you just write your component as if the data is already there. If the data isn't ready, React automatically pauses rendering and shows the loading fallback you defined. When the data arrives, React tries again. This makes components cleaner because they don't need to handle loading states themselves.

### Purposes

- To eliminate manual loading state management (`isLoading`, `isError`).
- To make components render declaratively without conditional loading logic.
- To enable streaming server rendering with progressive data delivery.
- To integrate with modern data-fetching libraries that implement the Suspense contract.
- To simplify component code by removing `useEffect` and `useState` for data fetching.
- To enable parallel data loading when multiple components suspend simultaneously.

### Syntax Rules and Structure

**General Syntax with `use()` (React 19):**
```jsx
import { use, Suspense } from 'react';

function Comments({ commentsPromise }) {
  // use() reads the Promise's value.
  // If the Promise is pending, the component suspends.
  // If the Promise rejects, the nearest Error Boundary activates.
  const comments = use(commentsPromise);

  return (
    <ul>
      {comments.map(comment => (
        <li key={comment.id}>{comment.text}</li>
      ))}
    </ul>
  );
}

function ArticlePage() {
  // Create the Promise (ideally on the server or in a Suspense-enabled source)
  const commentsPromise = fetchComments();

  return (
    <Suspense fallback={<p>Loading comments...</p>}>
      <Comments commentsPromise={commentsPromise} />
    </Suspense>
  );
}
```

**Component Breakdown:**
- `use(commentsPromise)`: The `use()` API reads the value of a Promise. If the Promise is pending, `use()` throws the Promise, causing the component to suspend.
- `<Suspense fallback={...}>`: The Suspense boundary catches the thrown Promise and renders the fallback.
- `commentsPromise`: The Promise must be stable across re-renders. Creating it inside the component body causes it to be recreated on every render, which is a common pitfall.
- When the Promise resolves, React re-renders `Comments` with the resolved data.

**General Syntax with TanStack Query `useSuspenseQuery`:**
```jsx
import { useSuspenseQuery } from '@tanstack/react-query';

function UserList() {
  // useSuspenseQuery always returns defined data (never undefined)
  // It suspends the component while the query is loading
  const { data } = useSuspenseQuery({
    queryKey: ['users'],
    queryFn: fetchUsers,
  });

  // data is guaranteed to be defined — no loading checks needed
  return (
    <ul>
      {data.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}

// Usage
function App() {
  return (
    <Suspense fallback={<p>Loading users...</p>}>
      <UserList />
    </Suspense>
  );
}
```

**Component Breakdown:**
- `useSuspenseQuery`: A TanStack Query hook designed for Suspense. It suspends the component while the query is loading and returns guaranteed-defined data.
- `queryKey` and `queryFn`: Standard TanStack Query options.
- The component does not need to handle `isLoading` or `isError` states; Suspense and Error Boundaries handle them.

**Syntax Rules:**
- The Promise passed to `use()` must be stable across re-renders. Prefer creating Promises in Server Components and passing them to Client Components.
- Do not wrap `use()` in try/catch blocks. `use()` suspends; it does not throw errors in the traditional sense. Error Boundaries handle rejected Promises.
- `use()` can be called conditionally (unlike Hooks), but it must be called inside a Component or Hook.
- With TanStack Query, multiple `useSuspenseQuery` calls in the same component suspend serially, causing a request waterfall. Use `useSuspenseQueries` for parallel fetching.
- Suspense for data fetching does not work with data fetched inside `useEffect` or event handlers.

**Constraints and Limitations:**
- The Promise must be cached or stable; creating a new Promise on every render causes infinite suspension loops.
- `use()` cannot be used in Server Components for data fetching; prefer `async`/`await` in Server Components.
- TanStack Query's `useSuspenseQuery` does not support `enabled` or `placeholderData` options.
- Errors from rejected Promises propagate to the nearest Error Boundary, not the Suspense fallback.
- React 19's `use()` is a stable API, but Suspense for data fetching in client components without a framework requires careful Promise management.

### Annotated Code Examples

**Example 1: Using `use()` with a Cached Promise**

```jsx
import React, { use, Suspense, useState } from 'react';

// A simple cache to ensure the Promise is stable across re-renders
const promiseCache = new Map();

function fetchUserData(userId) {
  const cacheKey = `user-${userId}`;

  if (!promiseCache.has(cacheKey)) {
    // Create the Promise only once per userId
    promiseCache.set(
      cacheKey,
      fetch(`https://jsonplaceholder.typicode.com/users/${userId}`)
        .then(res => {
          if (!res.ok) throw new Error('Failed to fetch user');
          return res.json();
        })
    );
  }

  return promiseCache.get(cacheKey);
}

function UserProfile({ userId }) {
  // use() reads the Promise's value
  // If pending, the component suspends
  const user = use(fetchUserData(userId));

  return (
    <div>
      <h2>{user.name}</h2>
      <p>Email: {user.email}</p>
      <p>Phone: {user.phone}</p>
    </div>
  );
}

function UserPage() {
  const [userId, setUserId] = useState(1);

  return (
    <div>
      <button onClick={() => setUserId(prev => prev + 1)}>
        Next User
      </button>

      <Suspense fallback={<p>Loading user profile...</p>}>
        <UserProfile userId={userId} />
      </Suspense>
    </div>
  );
}

export default UserPage;
```

**Expected Output:** The page displays a "Next User" button and "Loading user profile..." initially. Once the user data is fetched, the loading message is replaced by the user's name, email, and phone. Clicking "Next User" changes the `userId`, causing the component to suspend again and show the fallback until the new user's data arrives.

**Why This Output Occurs:** The `promiseCache` ensures that the same Promise is returned for the same `userId` across re-renders. Without this cache, calling `fetch()` inside the component body would create a new Promise on every render, causing an infinite suspension loop. When `use()` reads a pending Promise, it throws it to the Suspense boundary. When the Promise resolves, React re-renders `UserProfile` with the resolved data.

**Example 2: TanStack Query with `useSuspenseQuery`**

```jsx
import React, { Suspense } from 'react';
import { QueryClient, QueryClientProvider, useSuspenseQuery } from '@tanstack/react-query';
import { ErrorBoundary } from 'react-error-boundary';

// Create a QueryClient instance
const queryClient = new QueryClient();

async function fetchPosts() {
  const response = await fetch('https://jsonplaceholder.typicode.com/posts');
  if (!response.ok) throw new Error('Failed to fetch posts');
  return response.json();
}

function PostList() {
  // useSuspenseQuery suspends the component while loading
  // data is guaranteed to be defined — no loading checks needed
  const { data: posts } = useSuspenseQuery({
    queryKey: ['posts'],
    queryFn: fetchPosts,
  });

  return (
    <ul>
      {posts.slice(0, 5).map(post => (
        <li key={post.id}>
          <strong>{post.title}</strong>
        </li>
      ))}
    </ul>
  );
}

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <ErrorBoundary fallback={<p>Something went wrong loading posts.</p>}>
        <Suspense fallback={<p>Loading posts...</p>}>
          <PostList />
        </Suspense>
      </ErrorBoundary>
    </QueryClientProvider>
  );
}

export default App;
```

**Expected Output:** The page displays "Loading posts..." initially. Once the posts are fetched, the loading message is replaced by a list of the first five post titles. If the fetch fails, the Error Boundary displays "Something went wrong loading posts."

**Why This Output Occurs:** `useSuspenseQuery` throws a Promise to the Suspense boundary while the query is loading. TanStack Query caches the result, so subsequent renders return the cached data immediately. The `data` is guaranteed to be defined, eliminating the need for `if (isLoading)` checks. The Error Boundary catches any rejected Promises (fetch failures) and renders its fallback.

### Real-World Cases

- **Next.js App Router:** Using `async` Server Components with `<Suspense>` to stream data progressively.
- **Relay applications:** Using Relay's Suspense integration for GraphQL queries.
- **Dashboard widgets:** Each widget uses `useSuspenseQuery` with its own Suspense boundary.
- **E-commerce product pages:** Fetching product details, reviews, and recommendations in parallel with separate Suspense boundaries.
- **Social media feeds:** Using `use()` with cached Promises to render posts as they arrive.
- **Search results:** Suspending while search results load, with the query term as a dependency.

---

## Core Concept 4: Cascading & Parallel Loading

### Definitions

**Core Definition:** Cascading (sequential) loading is a data-fetching pattern where requests are made one after another, each waiting for the previous to complete; parallel loading is a pattern where independent requests are initiated simultaneously.

**Technical Definition:** In React Suspense, cascading loading occurs when components nested inside a suspended component cannot begin their own data fetching until the parent component has resolved and rendered. This creates a "waterfall" effect where the total loading time is the sum of all request durations. Parallel loading is achieved by restructuring components so that independent data fetches are initiated simultaneously, either by placing them in sibling Suspense boundaries, using `Promise.all`, or using hooks like `useSuspenseQueries` that fetch multiple queries in parallel. React 18+ allows siblings of a suspended component to pre-render in parallel, but nested components still create waterfalls.

**Beginner-Friendly Explanation:** Imagine you need to make three API calls to load a page: one for the user, one for their posts, and one for their comments. If you make them one at a time, you wait for the user call to finish before starting the posts call, and so on. That's a waterfall—slow and wasteful. If you make all three calls at the same time, the page loads much faster. Suspense lets you structure your components so that independent data loads in parallel.

### Purposes

- To minimise total page load time by fetching independent data simultaneously.
- To identify and eliminate request waterfalls in component hierarchies.
- To choreograph loading sequences where some content depends on other content.
- To use `Promise.all` for parallel fetching of related data.
- To use `useSuspenseQueries` for parallel queries in TanStack Query.
- To structure component trees so that independent sections have their own Suspense boundaries.

### Syntax Rules and Structure

**Cascading (Waterfall) Pattern:**
```jsx
// ❌ This creates a waterfall — child fetches only start after parent resolves
function ParentComponent() {
  const parentData = use(fetchParentData());

  return (
    <div>
      <p>{parentData.title}</p>
      {/* ChildComponent only renders after parentData resolves */}
      <ChildComponent parentId={parentData.id} />
    </div>
  );
}

function ChildComponent({ parentId }) {
  const childData = use(fetchChildData(parentId));
  return <p>{childData.detail}</p>;
}
```

**Component Breakdown:**
- `ParentComponent` suspends while `fetchParentData()` resolves.
- Only after `ParentComponent` renders does `ChildComponent` mount and begin fetching its own data.
- Total time = parent fetch time + child fetch time.

**Parallel Pattern with Sibling Suspense Boundaries:**
```jsx
// ✅ Parallel loading — both fetches start simultaneously
function Dashboard() {
  return (
    <div>
      <Suspense fallback={<UserSkeleton />}>
        <UserWidget />  {/* Fetches user data independently */}
      </Suspense>

      <Suspense fallback={<PostsSkeleton />}>
        <PostsWidget />  {/* Fetches posts data independently */}
      </Suspense>
    </div>
  );
}
```

**Component Breakdown:**
- `UserWidget` and `PostsWidget` are siblings, each wrapped in its own Suspense boundary.
- When `Dashboard` renders, React attempts to render both `UserWidget` and `PostsWidget` simultaneously.
- If both suspend, each boundary shows its own fallback.
- Both data fetches are initiated in parallel.

**Parallel Pattern with `Promise.all`:**
```jsx
function ProfilePage({ userId }) {
  // Start both fetches immediately, in parallel
  const userPromise = fetchUser(userId);
  const postsPromise = fetchPosts(userId);

  return (
    <Suspense fallback={<ProfileSkeleton />}>
      <UserDetails userPromise={userPromise} />
      <UserPosts postsPromise={postsPromise} />
    </Suspense>
  );
}

function UserDetails({ userPromise }) {
  const user = use(userPromise);
  return <h1>{user.name}</h1>;
}

function UserPosts({ postsPromise }) {
  const posts = use(postsPromise);
  return <ul>{posts.map(p => <li key={p.id}>{p.title}</li>)}</ul>;
}
```

**Component Breakdown:**
- Both Promises are created before rendering the Suspense boundary.
- `UserDetails` and `UserPosts` each read their own Promise via `use()`.
- Both fetches run in parallel because the Promises were created simultaneously.
- The Suspense boundary shows a single fallback until both components are ready.

**Syntax Rules:**
- To avoid waterfalls, create Promises as early as possible, ideally before rendering the component that reads them.
- Use sibling Suspense boundaries for independent sections that should load in parallel.
- Use `Promise.all` when you need to combine multiple Promises into one.
- With TanStack Query, use `useSuspenseQueries` instead of multiple `useSuspenseQuery` calls in the same component.
- In React Server Components, initiate all independent fetches before awaiting any of them.

**Constraints and Limitations:**
- React 18+ allows siblings of a suspended component to pre-render in parallel, but nested components still create waterfalls.
- `Promise.all` rejects immediately if any Promise rejects, potentially losing the results of successful Promises.
- TanStack Query's `useSuspenseQuery` suspends serially when called multiple times in the same component.
- Parallel loading increases the number of simultaneous requests, which may strain server or network resources.
- Cascading loading is sometimes unavoidable when a child component genuinely depends on the parent's data (e.g., `parentId` is needed to fetch child data).

### Annotated Code Examples

**Example 1: Waterfall vs Parallel Comparison**

```jsx
import React, { use, Suspense } from 'react';

// Simulated API calls with 500ms delay each
function fetchUser(id) {
  return new Promise(resolve =>
    setTimeout(() => resolve({ id, name: `User ${id}` }), 500)
  );
}

function fetchPosts(userId) {
  return new Promise(resolve =>
    setTimeout(() => resolve([
      { id: 1, title: `Post 1 by User ${userId}` },
      { id: 2, title: `Post 2 by User ${userId}` },
    ]), 500)
  );
}

// ❌ Waterfall: posts fetch waits for user fetch
function WaterfallProfile({ userId }) {
  const user = use(fetchUser(userId));
  const posts = use(fetchPosts(user.id)); // Starts only after user resolves

  return (
    <div>
      <h2>{user.name}</h2>
      <ul>{posts.map(p => <li key={p.id}>{p.title}</li>)}</ul>
    </div>
  );
}

// ✅ Parallel: both fetches start simultaneously
function ParallelProfile({ userId }) {
  const userPromise = fetchUser(userId);
  const postsPromise = fetchPosts(userId); // Starts immediately

  return (
    <Suspense fallback={<p>Loading profile...</p>}>
      <UserInfo userPromise={userPromise} />
      <PostList postsPromise={postsPromise} />
    </Suspense>
  );
}

function UserInfo({ userPromise }) {
  const user = use(userPromise);
  return <h2>{user.name}</h2>;
}

function PostList({ postsPromise }) {
  const posts = use(postsPromise);
  return <ul>{posts.map(p => <li key={p.id}>{p.title}</li>)}</ul>;
}

export { WaterfallProfile, ParallelProfile };
```

**Expected Output:** The `WaterfallProfile` takes approximately 1000ms to load (500ms for user + 500ms for posts). The `ParallelProfile` takes approximately 500ms (both fetches run simultaneously).

**Why This Output Occurs:** In `WaterfallProfile`, `use(fetchUser(userId))` suspends the component until the user data arrives. Only after the user data resolves does the component re-render, at which point `use(fetchPosts(user.id))` is called, starting the second fetch. The total time is the sum of both fetches. In `ParallelProfile`, both Promises are created before the Suspense boundary renders. `UserInfo` and `PostList` are siblings, so React attempts to render both simultaneously. Both fetches run in parallel, and the total time is the maximum of the two fetches.

**Example 2: TanStack Query Parallel Queries with `useSuspenseQueries`**

```jsx
import React, { Suspense } from 'react';
import { QueryClient, QueryClientProvider, useSuspenseQueries } from '@tanstack/react-query';

const queryClient = new QueryClient();

async function fetchUser(id) {
  const res = await fetch(`https://jsonplaceholder.typicode.com/users/${id}`);
  return res.json();
}

async function fetchPosts(userId) {
  const res = await fetch(`https://jsonplaceholder.typicode.com/posts?userId=${userId}`);
  return res.json();
}

async function fetchAlbums(userId) {
  const res = await fetch(`https://jsonplaceholder.typicode.com/albums?userId=${userId}`);
  return res.json();
}

function UserDashboard({ userId }) {
  // useSuspenseQueries fetches all queries in parallel
  const results = useSuspenseQueries({
    queries: [
      { queryKey: ['user', userId], queryFn: () => fetchUser(userId) },
      { queryKey: ['posts', userId], queryFn: () => fetchPosts(userId) },
      { queryKey: ['albums', userId], queryFn: () => fetchAlbums(userId) },
    ],
  });

  const [userData, postsData, albumsData] = results;
  const user = userData.data;
  const posts = postsData.data;
  const albums = albumsData.data;

  return (
    <div>
      <h1>{user.name}</h1>
      <h2>Posts ({posts.length})</h2>
      <h2>Albums ({albums.length})</h2>
    </div>
  );
}

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <Suspense fallback={<p>Loading dashboard...</p>}>
        <UserDashboard userId={1} />
      </Suspense>
    </QueryClientProvider>
  );
}

export default App;
```

**Expected Output:** The page displays "Loading dashboard..." until all three queries complete. Because `useSuspenseQueries` fetches them in parallel, the loading time is approximately the duration of the slowest query, not the sum of all three.

**Why This Output Occurs:** `useSuspenseQueries` accepts an array of query configurations and executes them in parallel. Unlike calling `useSuspenseQuery` three times sequentially (which would cause a waterfall), `useSuspenseQueries` initiates all queries simultaneously. The component suspends until all queries resolve, then renders with all data available.

### Real-World Cases

- **Dashboard with multiple widgets:** Each widget fetches its own data in a separate Suspense boundary, loading in parallel.
- **E-commerce product page:** Fetching product details, reviews, and recommendations in parallel.
- **Social media profile:** Fetching user info, posts, and followers simultaneously.
- **Analytics dashboard:** Fetching metrics from multiple endpoints in parallel with `useSuspenseQueries`.
- **Server-rendered pages:** Initiating all independent data fetches before awaiting any of them in Server Components.

---

## Core Concept 5: Error Boundaries Collaboration

### Definitions

**Core Definition:** Error Boundary collaboration with Suspense refers to pairing Suspense boundaries with Error Boundaries to handle both loading states (Suspense) and error states (Error Boundaries) in asynchronous rendering.

**Technical Definition:** An Error Boundary is a class component that implements `static getDerivedStateFromError()` and/or `componentDidCatch()` to catch JavaScript errors anywhere in its child component tree. In the context of Suspense, when a Promise passed to `use()` is rejected, React looks for the nearest Error Boundary above the suspended component and renders its fallback. This collaboration creates a complete async lifecycle: Suspense handles the pending state (loading), Error Boundaries handle the rejected state (errors), and the component renders normally when the Promise resolves successfully. The `react-error-boundary` library provides a functional wrapper component (`<ErrorBoundary>`) that simplifies usage and adds features like reset functionality.

**Beginner-Friendly Explanation:** Suspense handles the "I'm waiting for data" state, but what happens if the data fetch fails? That's where Error Boundaries come in. You wrap your Suspense boundary with an Error Boundary, and if anything goes wrong—like the server returns an error or the network fails—the Error Boundary catches it and shows a friendly error message instead of a broken page.

### Purposes

- To handle rejected Promises from Suspense-enabled data fetching.
- To prevent a single failed request from crashing the entire application.
- To provide user-friendly error messages with recovery options.
- To isolate errors to specific parts of the UI tree.
- To enable retry mechanisms when errors occur.
- To integrate with TanStack Query's `QueryErrorResetBoundary` for error recovery.

### Syntax Rules and Structure

**General Syntax with React Class-Based Error Boundary:**
```jsx
import React, { Component, Suspense } from 'react';

class ErrorBoundary extends Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error) {
    // Update state so the next render shows the fallback UI
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    // Log the error to an error reporting service
    console.error('Error caught by boundary:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      // Render fallback UI
      return this.props.fallback;
    }
    return this.props.children;
  }
}

// Usage with Suspense
function App() {
  return (
    <ErrorBoundary fallback={<p>Something went wrong.</p>}>
      <Suspense fallback={<p>Loading...</p>}>
        <DataComponent />
      </Suspense>
    </ErrorBoundary>
  );
}
```

**Component Breakdown:**
- `ErrorBoundary`: A class component with `getDerivedStateFromError` (for rendering the fallback) and `componentDidCatch` (for logging).
- `hasError` and `error`: State properties that track whether an error occurred and the error itself.
- `fallback`: A prop that accepts the UI to render when an error occurs.
- The Error Boundary wraps the Suspense boundary, so rejected Promises from `use()` are caught by the Error Boundary.

**General Syntax with `react-error-boundary`:**
```jsx
import { ErrorBoundary } from 'react-error-boundary';
import { Suspense } from 'react';

function ErrorFallback({ error, resetErrorBoundary }) {
  return (
    <div role="alert">
      <p>Something went wrong:</p>
      <pre>{error.message}</pre>
      <button onClick={resetErrorBoundary}>Try again</button>
    </div>
  );
}

function App() {
  return (
    <ErrorBoundary
      FallbackComponent={ErrorFallback}
      onReset={() => {
        // Reset any state that caused the error
      }}
    >
      <Suspense fallback={<p>Loading...</p>}>
        <DataComponent />
      </Suspense>
    </ErrorBoundary>
  );
}
```

**Component Breakdown:**
- `<ErrorBoundary>`: A functional wrapper from `react-error-boundary`.
- `FallbackComponent`: A component that receives `error` and `resetErrorBoundary` props.
- `resetErrorBoundary`: A function that resets the Error Boundary's state, causing it to retry rendering its children.
- `onReset`: A callback invoked when the boundary resets.

**Syntax Rules:**
- Error Boundaries must be class components (or use the `react-error-boundary` library for a functional equivalent).
- Error Boundaries catch errors during rendering, in lifecycle methods, and in constructors of the whole tree below them.
- Error Boundaries do not catch errors in event handlers, asynchronous code (e.g., `setTimeout`), server-side rendering, or errors thrown in the Error Boundary itself.
- For Suspense, the Error Boundary must wrap the Suspense boundary (or be an ancestor of it) to catch rejected Promises from `use()`.
- `use()` does not throw errors in the traditional sense; it suspends. Rejected Promises propagate to the nearest Error Boundary.
- With TanStack Query, use `QueryErrorResetBoundary` or `useQueryErrorResetBoundary` to reset query errors.

**Constraints and Limitations:**
- Error Boundaries do not catch errors in event handlers; use try/catch for those.
- Error Boundaries do not catch errors in asynchronous code like `setTimeout` or `requestAnimationFrame`.
- Only class components can be Error Boundaries (without a library).
- Error Boundaries catch errors in the entire subtree, so placing them too high can cause too much UI to be replaced by the fallback.
- In React 19, errors in Server Components are handled differently; Error Boundaries may not catch them in all cases.

### Annotated Code Examples

**Example 1: Error Boundary with Suspense and `use()`**

```jsx
import React, { use, Suspense } from 'react';
import { ErrorBoundary } from 'react-error-boundary';

// A Promise that rejects after 1 second
const failingPromise = new Promise((_, reject) =>
  setTimeout(() => reject(new Error('Failed to load data')), 1000)
);

// A Promise that resolves successfully
const successPromise = new Promise(resolve =>
  setTimeout(() => resolve({ name: 'John Doe', role: 'Developer' }), 500)
);

function UserProfile({ userPromise }) {
  const user = use(userPromise);

  return (
    <div>
      <h2>{user.name}</h2>
      <p>Role: {user.role}</p>
    </div>
  );
}

function ErrorFallback({ error, resetErrorBoundary }) {
  return (
    <div role="alert" style={{ color: 'red' }}>
      <p>Error: {error.message}</p>
      <button onClick={resetErrorBoundary}>Retry</button>
    </div>
  );
}

function App() {
  return (
    <div>
      <h1>User Profile</h1>

      {/* Error Boundary wraps the Suspense boundary */}
      <ErrorBoundary FallbackComponent={ErrorFallback}>
        <Suspense fallback={<p>Loading profile...</p>}>
          <UserProfile userPromise={failingPromise} />
        </Suspense>
      </ErrorBoundary>

      {/* A successful profile for comparison */}
      <ErrorBoundary FallbackComponent={ErrorFallback}>
        <Suspense fallback={<p>Loading profile...</p>}>
          <UserProfile userPromise={successPromise} />
        </Suspense>
      </ErrorBoundary>
    </div>
  );
}

export default App;
```

**Expected Output:** The page displays "User Profile" and two "Loading profile..." messages initially. After 500ms, the second profile resolves and shows "John Doe" and "Role: Developer". After 1000ms, the first Promise rejects, and the Error Boundary replaces the first loading message with "Error: Failed to load data" and a "Retry" button.

**Why This Output Occurs:** The `failingPromise` rejects after 1 second, causing `use()` to propagate the rejection to the nearest Error Boundary. The `successPromise` resolves after 500ms, so `UserProfile` renders normally. The Error Boundary catches the rejection and renders `ErrorFallback` with the error message and a reset button. The `resetErrorBoundary` function resets the boundary's state, causing it to re-render its children and retry the Promise.

**Example 2: TanStack Query Error Boundary with Reset**

```jsx
import React, { Suspense } from 'react';
import { QueryClient, QueryClientProvider, useSuspenseQuery, QueryErrorResetBoundary } from '@tanstack/react-query';
import { ErrorBoundary } from 'react-error-boundary';

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      retry: 0, // Disable retry for demonstration
    },
  },
});

async function fetchUser(id) {
  const res = await fetch(`https://jsonplaceholder.typicode.com/users/${id}`);
  if (!res.ok) throw new Error('Failed to fetch user');
  return res.json();
}

function UserCard({ userId }) {
  const { data: user } = useSuspenseQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
  });

  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}

function ErrorFallback({ error, resetErrorBoundary }) {
  return (
    <div role="alert">
      <p>Could not load user: {error.message}</p>
      <button onClick={resetErrorBoundary}>Try again</button>
    </div>
  );
}

function App() {
  const [userId, setUserId] = React.useState(1);

  return (
    <QueryClientProvider client={queryClient}>
      {/* QueryErrorResetBoundary resets query errors on retry */}
      <QueryErrorResetBoundary>
        {({ reset }) => (
          <ErrorBoundary
            FallbackComponent={ErrorFallback}
            onReset={reset}
          >
            <Suspense fallback={<p>Loading user...</p>}>
              <UserCard userId={userId} />
            </Suspense>
          </ErrorBoundary>
        )}
      </QueryErrorResetBoundary>

      <button onClick={() => setUserId(prev => prev + 1)}>
        Next User
      </button>
    </QueryClientProvider>
  );
}

export default App;
```

**Expected Output:** The page displays "Loading user..." initially, then the user's name and email. If the fetch fails, the Error Boundary shows "Could not load user: Failed to fetch user" with a "Try again" button. Clicking "Try again" resets the query error and retries the fetch.

**Why This Output Occurs:** `QueryErrorResetBoundary` provides a `reset` function that clears the query's error state. When the Error Boundary's `onReset` calls this function, TanStack Query resets the error and re-fetches the data. The `ErrorBoundary` catches the rejection from `useSuspenseQuery` and renders `ErrorFallback`. The `resetErrorBoundary` function triggers `onReset`, which calls the query's `reset` function, allowing the query to retry.

### Real-World Cases

- **Data dashboards:** Each widget has its own Error Boundary and Suspense boundary, so a failed widget shows an error message while others continue to work.
- **E-commerce product pages:** If the recommendation service fails, the Error Boundary shows a fallback while the main product content remains visible.
- **Social media feeds:** A failed post fetch shows a retry button without breaking the entire feed.
- **Multi-step forms:** Each step's data fetch is wrapped in an Error Boundary, allowing users to retry a failed step.
- **Real-time dashboards:** WebSocket connection failures are caught by Error Boundaries with reconnect options.

---

## References

- React Official Documentation – `<Suspense>`: https://react.dev/reference/react/Suspense
- React Official Documentation – `use()` API: https://react.dev/reference/react/use
- React Official Documentation – Error Boundaries: https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary
- React Official Documentation – `React.lazy`: https://react.dev/reference/react/lazy
- React Official Documentation – `startTransition`: https://react.dev/reference/react/startTransition
- React Official Documentation – `useDeferredValue`: https://react.dev/reference/react/useDeferredValue
- TanStack Query – Suspense Guide: https://tanstack.com/query/v5/docs/framework/react/guides/suspense
- TanStack Query – `useSuspenseQuery` Reference: https://tanstack.com/query/v5/docs/framework/react/reference/useSuspenseQuery
- TanStack Query – `useSuspenseQueries` Reference: https://tanstack.com/query/v5/docs/framework/react/reference/useSuspenseQueries
- Next.js – Fetching Data: https://nextjs.org/docs/app/getting-started/fetching-data
- Next.js – Streaming with Suspense: https://nextjs.org/docs/app/building-your-application/routing/loading-ui-and-streaming
- React Error Boundary Library: https://github.com/bvaughn/react-error-boundary
- React Working Group – Suspense Architecture Overview: https://github.com/reactwg/react-18/discussions/37
- React 19 Release Notes: https://react.dev/blog/2024/12/05/react-19
- MDN Web Docs – Promise: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise
- MDN Web Docs – AbortController: https://developer.mozilla.org/en-US/docs/Web/API/AbortController
- Epic React – How React Suspense Works Under the Hood: https://www.epicreact.dev/how-react-suspense-works-under-the-hood-throwing-promises-and-declarative-loading-states
- Epic React – The Big Server Waterfall Problem with RSCs: https://www.epicreact.dev/the-big-server-waterfall-problem-with-rscs
- Steve Kinney – Suspense for Data Fetching: https://stevekinney.com/courses/react-performance/suspense-for-data-fetching
- Steve Kinney – The `use()` Hook: https://stevekinney.com/courses/react-performance/the-use-hook
- Syncfusion – React 19 Suspense for Data Fetching: https://www.syncfusion.com/blogs/post/react-19-suspense-data-fetching
- SitePoint – React 19 `use()` Hook Data Fetching Patterns: https://www.sitepoint.com/react-19-use-hook-data-fetching-patterns
- FreeCodeCamp – The Modern React Data Fetching Handbook: https://www.freecodecamp.org/news/the-modern-react-data-fetching-handbook-suspense-use-and-errorboundary-explained/