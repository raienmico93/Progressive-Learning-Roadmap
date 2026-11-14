# React Router — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** React Router is the standard routing library for React, providing declarative, component-based routing that keeps the URL and UI in sync.

**Technical Definition:** React Router is a multi-strategy router for React that maps URL paths to component trees. It ships in three modes: **Declarative Mode** (classic `<BrowserRouter>` with JSX routes), **Data Mode** (`createBrowserRouter` with route objects, loaders, and actions), and **Framework Mode** (full-stack SSR with file-based routing, the successor to Remix). The library is built on the History API and provides a comprehensive set of components (`Link`, `NavLink`, `Navigate`, `Outlet`) and hooks (`useNavigate`, `useParams`, `useSearchParams`, `useLocation`, `useMatch`, `useResolvedPath`, `useHref`, `useNavigationType`) for navigation and routing state. React Router v7 is the current major version (v8 shipped in 2026), and all modes remain supported.

**Beginner-Friendly Explanation:** React Router is the tool that lets your React app have multiple “pages” without reloading the browser. You define which URL shows which component, and React Router handles the rest — clicking links, updating the address bar, handling the back button, and passing data to your components based on the URL. It has three levels of complexity: the simple version for small apps, the data version for apps that need to load data before rendering, and the full-stack version for server-rendered apps.

### Key Characteristics

- **Three modes:** Declarative (SPA routing), Data (loaders/actions), Framework (full-stack SSR).
- **URL as source of truth:** Navigation state, params, and search params are derived from the URL.
- **Nested routing:** Parent layouts render `<Outlet />` for child routes.
- **Data APIs:** Loaders fetch data before render; actions handle mutations with automatic revalidation.
- **Lazy loading:** Route-level code splitting via `lazy` for reduced bundle size.
- **Error boundaries:** Route-level `errorElement` catches rendering and loading errors.
- **Rich hook API:** `useParams`, `useSearchParams`, `useLocation`, `useNavigate`, `useMatch`, and more.

### Prerequisites

- React function components, JSX, and props.
- Basic understanding of URLs and the History API.
- Familiarity with `useState` and `useEffect`.
- Knowledge of Promises and async JavaScript (for loaders/actions).

### Related Programming Areas

- **Client-Side Routing** — the broader architectural model.
- **Server-Side Rendering** — Framework Mode provides SSR.
- **Code Splitting** — route-level `lazy` is the primary splitting strategy.
- **Form Handling** — `<Form>` and actions integrate with data mutations.

---

## 1. React Router

### Definitions

**Core Definition:** React Router is a complete routing library for React that synchronizes the UI with the URL and provides navigation, data loading, and mutation primitives.

**Technical Definition:** React Router is a collection of navigational components and hooks that map URLs to React component trees. It provides three distinct modes: Declarative (JSX `<Routes>`/`<Route>`), Data (`createBrowserRouter` with loaders, actions, and error boundaries), and Framework (file-based SSR with the Vite plugin). The library is published as `react-router` (v7+) and `react-router-dom` (legacy v6 and below).

**Beginner-Friendly Explanation:** React Router is the library that gives your React app multiple pages. It watches the URL and renders the right component for each path.

### Purposes

- To map URLs to React component trees.
- To provide client-side navigation without full page reloads.
- To handle dynamic route parameters and query strings.
- To load data before rendering routes (Data Mode).
- To handle form submissions and mutations with automatic revalidation.
- To support server-side rendering and full-stack applications (Framework Mode).

### Syntax Rules and Structure

**General Syntax (Declarative Mode):**

```jsx
import { BrowserRouter, Routes, Route } from 'react-router';

<BrowserRouter>
  <Routes>
    <Route path="/" element={<Home />} />
    <Route path="/about" element={<About />} />
  </Routes>
</BrowserRouter>
```

**Component Breakdown:**

| Mode | Router Setup | Route Definition |
|------|--------------|------------------|
| Declarative | `<BrowserRouter>` | JSX `<Routes>` / `<Route>` |
| Data | `createBrowserRouter([...])` | Route objects with `loader`/`action` |
| Framework | Vite plugin + `routes.ts` | File-based route modules |

**Syntax Rules:**

1. Choose the mode based on your app's complexity: Declarative for simple SPAs, Data for SPAs needing loaders/actions, Framework for SSR.
2. Wrap the app in a router component or provide a router instance.
3. Define routes with `path` and `element` (Declarative) or `path` and `Component` (Data/Framework).
4. Use nested routes with `<Outlet />` for shared layouts.

### Annotated Code Examples

```jsx
// Declarative Mode
import { BrowserRouter, Routes, Route, Link } from 'react-router';

function App() {
  return (
    <BrowserRouter>
      <nav>
        <Link to="/">Home</Link>
        <Link to="/about">About</Link>
      </nav>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
      </Routes>
    </BrowserRouter>
  );
}
```

**Expected Output:** Clicking “About” updates the URL to `/about` and renders the `About` component without a page reload.

**Why this output:** `BrowserRouter` uses the History API. `Link` prevents default navigation and uses `pushState`. `Routes` matches the current URL and renders the corresponding element.

### Real-World Cases

- **SPAs:** Dashboards, SaaS products, and internal tools.
- **Data-driven apps:** E-commerce, content sites with loaders.
- **Full-stack apps:** Applications needing SSR and server mutations.

### References

- React Router — Official Documentation — https://reactrouter.com/home
- React Router — Picking a Mode — https://reactrouter.com/start/modes
- Ashukr321/Route-Architect — https://github.com/Ashukr321/Route-Architect

---

## 2. `BrowserRouter`

### Definitions

**Core Definition:** `BrowserRouter` is the router component that uses the HTML5 History API to keep the UI in sync with the URL in a client-side application.

**Technical Definition:** `<BrowserRouter>` creates a router that stores the current location in `window.history` and uses `pushState` and `replaceState` for navigation. It is the standard router for browser-based SPAs. It does not support the data APIs (`loader`, `action`); those require `createBrowserRouter` (Data Mode) or Framework Mode.

**Beginner-Friendly Explanation:** `BrowserRouter` is the default router for web apps. It uses clean URLs (no `#`) and the browser's back button works normally.

### Purposes

- To provide client-side routing with clean URLs.
- To integrate with the browser's History API.
- To enable back/forward navigation in SPAs.
- To serve as the foundation for Declarative Mode routing.

### Syntax Rules and Structure

```jsx
import { BrowserRouter } from 'react-router';

<BrowserRouter>
  <App />
</BrowserRouter>
```

**Syntax Rules:**

1. Wrap the entire app (or the routed subtree) in `<BrowserRouter>`.
2. It accepts no required props; optional `basename` sets a base path.
3. It requires the server to serve the SPA shell for all client-side routes.
4. It does not support data APIs; use `createBrowserRouter` for loaders/actions.

**Constraints and Limitations:**

- Deep links fail if the server does not serve the SPA shell for all paths.
- No data loading APIs (loaders/actions).

### Annotated Code Examples

```jsx
import { BrowserRouter, Routes, Route } from 'react-router';

function App() {
  return (
    <BrowserRouter basename="/app">
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/dashboard" element={<Dashboard />} />
      </Routes>
    </BrowserRouter>
  );
}
```

**Expected Output:** The app routes under `/app`, so the dashboard is at `/app/dashboard`.

**Why this output:** `basename` prefixes all routes and links, allowing the app to be deployed under a subpath.

### Real-World Cases

- **Standard SPAs:** Any client-side app with clean URLs.
- **Subpath deployments:** Apps hosted under `/app` or `/admin`.

### References

- React Router — BrowserRouter (v7.5.3) — https://reactrouter.com/7.5.3/api/declarative-routers/BrowserRouter
- React Router — Picking a Router — https://reactrouter.com/start/declarative/installation

---

## 3. `HashRouter`

### Definitions

**Core Definition:** `HashRouter` is a router component that stores the current location in the URL hash (`#/path`) instead of the pathname, so the browser never sends the route to the server.

**Technical Definition:** `<HashRouter>` uses `window.location.hash` for routing. Changing the hash fires `hashchange` and does not trigger a document request. This makes it suitable for static hosting without SPA fallback configuration. It does not support data APIs.

**Beginner-Friendly Explanation:** `HashRouter` puts the route after a `#` in the URL. It works on any static host because the server never sees the route.

### Purposes

- To enable client-side routing on static hosts without server configuration.
- To avoid SPA fallback requirements.
- To support routing in embedded or file-based contexts.

### Syntax Rules and Structure

```jsx
import { HashRouter } from 'react-router';

<HashRouter>
  <App />
</HashRouter>
```

**Syntax Rules:**

1. Wrap the app in `<HashRouter>`.
2. Routes appear after `#` (e.g., `/#/about`).
3. The server always receives a request for `/`; no fallback is needed.
4. Hash routes are invisible to the server, so SSR cannot resolve them.

**Constraints and Limitations:**

- URLs look unusual and are less SEO-friendly.
- Fragments are not sent to the server; no server-side route resolution.
- Some analytics tools handle hash routes poorly.

### Annotated Code Examples

```jsx
import { HashRouter, Routes, Route } from 'react-router';

function App() {
  return (
    <HashRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
      </Routes>
    </HashRouter>
  );
}
```

**Expected Output:** Navigating to About produces `https://example.com/#/about` and renders `About` without a server request.

**Why this output:** `HashRouter` reads and writes `window.location.hash`. The browser does not send the fragment to the server.

### Real-World Cases

- **Static hosting:** GitHub Pages, S3, and other hosts without rewrite rules.
- **Electron apps:** File-based apps where pathname routing is awkward.
- **Embedded widgets:** Routing inside iframes.

### References

- React Router — Picking a Router (HashRouter) — https://reactrouter.com/6.5.0/routers/picking-a-router
- React Router — createHashRouter — https://reactrouter.com/api/data-routers/createHashRouter

---

## 4. `MemoryRouter`

### Definitions

**Core Definition:** `MemoryRouter` is a router component that stores the current location in memory, without interacting with the browser's URL or history.

**Technical Definition:** `<MemoryRouter>` keeps the location in an internal array and does not read or write `window.history` or `window.location`. It is primarily used for testing and for non-browser environments (React Native, embedded apps). It supports `initialEntries` and `initialIndex` props to control the starting location.

**Beginner-Friendly Explanation:** `MemoryRouter` is a router that does not touch the browser's URL. It is used for tests and non-browser environments.

### Purposes

- To test components that use routing without a real browser URL.
- To support routing in React Native and other non-DOM environments.
- To control the initial route in tests with `initialEntries`.
- To isolate routing state from the browser history.

### Syntax Rules and Structure

```jsx
import { MemoryRouter } from 'react-router';

<MemoryRouter initialEntries={['/']} initialIndex={0}>
  <App />
</MemoryRouter>
```

**Syntax Rules:**

1. Wrap the app or test in `<MemoryRouter>`.
2. Use `initialEntries` to set the starting location(s).
3. Use `initialIndex` to select which entry is active.
4. Navigation works in memory; the browser URL does not change.

**Constraints and Limitations:**

- The browser URL is not updated; the address bar stays the same.
- Not suitable for production browser apps where URLs must be shareable.

### Annotated Code Examples

```jsx
import { MemoryRouter, Routes, Route } from 'react-router';
import { render, screen } from '@testing-library/react';

test('renders about page', () => {
  render(
    <MemoryRouter initialEntries={['/about']}>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
      </Routes>
    </MemoryRouter>
  );
  expect(screen.getByText('About')).toBeInTheDocument();
});
```

**Expected Output:** The test renders the `About` page because the initial entry is `/about`.

**Why this output:** `MemoryRouter` starts at `/about`, and `Routes` matches it without needing a real browser URL.

### Real-World Cases

- **Unit tests:** Testing routed components in isolation.
- **React Native:** Routing without a URL bar.
- **Storybook:** Simulating routing in component stories.

### References

- React Router — Picking a Router (MemoryRouter) — https://reactrouter.com/6.5.0/routers/picking-a-router
- React Router — createMemoryRouter — https://reactrouter.com/7.5.3/api/data-routers/createMemoryRouter

---

## 5. `Routes`

### Definitions

**Core Definition:** `Routes` is the component that matches the current URL against its child `<Route>` elements and renders the best match.

**Technical Definition:** `<Routes>` is the declarative route-matching container. It reads the current location from the router context, iterates over its child `<Route>` elements, and renders the branch that best matches the URL. It replaces the v5 `<Switch>` component and supports nested routes.

**Beginner-Friendly Explanation:** `Routes` is the container that holds all your route definitions. It looks at the current URL and decides which route to show.

### Purposes

- To match the current URL against route definitions.
- To render the matched route branch.
- To support nested route hierarchies.
- To replace the legacy `<Switch>` component.

### Syntax Rules and Structure

```jsx
import { Routes, Route } from 'react-router';

<Routes>
  <Route path="/" element={<Home />} />
  <Route path="/about" element={<About />} />
  <Route path="*" element={<NotFound />} />
</Routes>
```

**Syntax Rules:**

1. `Routes` must be inside a router (`BrowserRouter`, `HashRouter`, `MemoryRouter`).
2. Child `<Route>` elements define the routes.
3. `Routes` renders the first matching route (by specificity).
4. Use a `*` route for 404s.
5. Nested routes use `<Outlet />` in the parent to render children.

### Annotated Code Examples

```jsx
<Routes>
  <Route path="/" element={<Home />} />
  <Route path="/users" element={<UsersLayout />}>
    <Route index element={<UserList />} />
    <Route path=":id" element={<UserDetail />} />
  </Route>
  <Route path="*" element={<NotFound />} />
</Routes>
```

**Expected Output:** `/users` renders the user list; `/users/123` renders the user detail. Unknown paths render `NotFound`.

**Why this output:** `Routes` matches the pathname against the route patterns. The nested `:id` route matches dynamic segments.

### Real-World Cases

- **All Declarative Mode apps:** `Routes` is the route container.
- **Nested layouts:** Parent routes with child routes.

### References

- React Router — Routes — https://beta.reactrouter.com/6.26.2/components/Routes
- React Router — Routing (v7.1.1) — https://beta.reactrouter.com/7.1.1/start/framework/routing

---

## 6. `Route`

### Definitions

**Core Definition:** `Route` is the component that defines a mapping between a URL path and a React element (or component).

**Technical Definition:** `<Route>` declares a path pattern and the element to render when that pattern matches. It can be nested inside `<Routes>` or inside another `<Route>` to create nested UI. In Data Mode, the equivalent is a route object with `path`, `Component`, `loader`, `action`, and other properties.

**Beginner-Friendly Explanation:** `Route` is a single rule: “When the URL looks like this, show this component.”

### Purposes

- To define a URL pattern and its rendered element.
- To support dynamic segments (`:id`), index routes (`index`), and splats (`*`).
- To enable nested routing through parent-child relationships.
- To attach data loading and error handling in Data Mode.

### Syntax Rules and Structure

```jsx
<Route path="/users/:id" element={<UserDetail />} />
<Route index element={<Home />} />
<Route path="*" element={<NotFound />} />
```

**Syntax Rules:**

1. `path` defines the URL pattern; `element` defines what renders.
2. Use `:param` for dynamic segments.
3. Use `index` for the default child route at the parent path.
4. Use `*` as a splat or catch-all.
5. Nested routes are defined by nesting `<Route>` inside another `<Route>`.

**Constraints and Limitations:**

- Route paths must be unique among siblings.
- Dynamic segments are strings; parse and validate them.

### Annotated Code Examples

```jsx
<Route path="/products/:category/:id" element={<ProductDetail />} />
```

**Expected Output:** `/products/shoes/42` renders `ProductDetail` with `category="shoes"` and `id="42"`.

**Why this output:** The route pattern matches the path segments and extracts them as params accessible via `useParams`.

### Real-World Cases

- **E-commerce:** `/products/:category/:id`.
- **User profiles:** `/users/:id`.
- **Catch-all:** `*` for 404 pages.

### References

- React Router — Route Object — https://beta.reactrouter.com/7.6.0/start/data/route-object
- React Router — Route — https://beta.reactrouter.com/7.18.5/api/components/Route

---

## 7. `Link`

### Definitions

**Core Definition:** `Link` is the component for declarative, client-side navigation that renders an accessible `<a>` element and prevents full page reloads.

**Technical Definition:** `<Link to="...">` renders an `<a href="...">` and intercepts clicks to perform client-side navigation via the router's history. It supports `replace`, `state`, and `preventScrollReset` props. In Data Mode, it also supports `prefetch` for preloading route data.

**Beginner-Friendly Explanation:** `Link` is the React Router way to make a link. It looks like a normal link but navigates without reloading the page.

### Purposes

- To provide client-side navigation without full reloads.
- To render accessible anchor elements with correct `href` values.
- To support relative and absolute paths.
- To enable state passing and history replacement.

### Syntax Rules and Structure

```jsx
import { Link } from 'react-router';

<Link to="/about">About</Link>
<Link to="/users/123" replace>User</Link>
<Link to="/dashboard" state={{ from: 'home' }}>Dashboard</Link>
```

**Syntax Rules:**

1. Use `to` for the destination path.
2. Use `replace` to replace the current history entry.
3. Use `state` to pass data to the next location.
4. Relative paths are resolved relative to the current route.
5. `Link` renders an `<a>` tag for accessibility.

### Annotated Code Examples

```jsx
<Link to="/products/42" replace>View Product</Link>
```

**Expected Output:** Clicking navigates to `/products/42` and replaces the current history entry, so the back button skips the previous page.

**Why this output:** `replace` uses `history.replaceState` instead of `pushState`, avoiding a new history entry.

### Real-World Cases

- **Navigation menus:** Internal links between pages.
- **Redirects:** Replacing the current entry after login.
- **State passing:** Sending context to the next route.

### References

- React Router — Link — https://reactrouter.com/api/components/Link
- React Router — Navigating — https://reactrouter.com/start/declarative/navigating

---

## 8. `NavLink`

### Definitions

**Core Definition:** `NavLink` is a special `Link` that automatically applies an “active” style when its destination matches the current URL.

**Technical Definition:** `<NavLink>` extends `<Link>` with active-state awareness. It applies the `active` class (or a custom class via `className` function) when the link's `to` path matches the current location. It also sets `aria-current="page"` for accessibility. The `end` prop restricts matching to the exact path.

**Beginner-Friendly Explanation:** `NavLink` is a link that knows when it is the active page, so you can style it differently.

### Purposes

- To highlight the current navigation item.
- To provide accessible current-page indication with `aria-current`.
- To simplify active-state styling in navigation menus.
- To support exact matching with `end`.

### Syntax Rules and Structure

```jsx
import { NavLink } from 'react-router';

<NavLink to="/about" className={({ isActive }) => isActive ? 'active' : ''}>
  About
</NavLink>

<NavLink to="/" end>Home</NavLink>
```

**Syntax Rules:**

1. `className` can be a string or a function receiving `{ isActive }`.
2. `style` can also be a function receiving `{ isActive }`.
3. Use `end` to match only the exact path (not child paths).
4. `NavLink` automatically adds `aria-current="page"` when active.

### Annotated Code Examples

```jsx
<nav>
  <NavLink to="/" end className={({ isActive }) => isActive ? 'nav-active' : ''}>
    Home
  </NavLink>
  <NavLink to="/reports" className={({ isActive }) => isActive ? 'nav-active' : ''}>
    Reports
  </NavLink>
</nav>
```

**Expected Output:** The active link receives the `nav-active` class and `aria-current="page"`.

**Why this output:** `NavLink` compares the current location to `to` and applies the active styling and ARIA attribute.

### Real-World Cases

- **Sidebar navigation:** Highlighting the active section.
- **Tabs:** Active tab styling.
- **Breadcrumbs:** Current page indication.

### References

- React Router — NavLink — https://reactrouter.com/api/components/NavLink

---

## 9. `Navigate`

### Definitions

**Core Definition:** `Navigate` is a component that performs an immediate navigation when rendered, useful for declarative redirects.

**Technical Definition:** `<Navigate to="..." replace />` triggers a navigation as soon as it renders. It is used for redirects (e.g., after login, for protected routes) and replaces the current history entry by default. It is the component equivalent of calling `useNavigate()` in an effect.

**Beginner-Friendly Explanation:** `Navigate` is a redirect. When it renders, it sends the user to another page.

### Purposes

- To perform declarative redirects.
- To redirect after authentication.
- To replace the current history entry during redirects.
- To redirect from old routes to new ones.

### Syntax Rules and Structure

```jsx
import { Navigate } from 'react-router';

<Navigate to="/login" replace />
<Navigate to="/dashboard" state={{ from: 'login' }} />
```

**Syntax Rules:**

1. `to` specifies the destination.
2. `replace` replaces the current history entry (recommended for redirects).
3. `state` passes data to the next location.
4. `Navigate` renders nothing; it performs navigation during render.

### Annotated Code Examples

```jsx
function ProtectedRoute({ user, children }) {
  if (!user) {
    return <Navigate to="/login" replace />;
  }
  return children;
}
```

**Expected Output:** Unauthenticated users are immediately redirected to `/login`, replacing the protected route in history.

**Why this output:** `Navigate` triggers navigation when rendered. `replace` prevents the protected route from appearing in the back stack.

### Real-World Cases

- **Auth redirects:** Sending unauthenticated users to login.
- **Route aliases:** Redirecting old paths to new ones.
- **Post-submission redirects:** Redirecting after a successful action.

### References

- React Router — Navigate — https://reactrouter.com/api/components/Navigate

---

## 10. `Outlet`

### Definitions

**Core Definition:** `Outlet` is the placeholder component in a parent route that renders the matched child route's element.

**Technical Definition:** `<Outlet>` renders the next matched route in the hierarchy. Parent layouts use it to display child content while keeping shared UI (navigation, sidebar) persistent. It is the mechanism behind nested routing.

**Beginner-Friendly Explanation:** `Outlet` is the hole in a layout where child pages appear. The sidebar and header stay, and the child page fills the outlet.

### Purposes

- To render child routes inside a parent layout.
- To keep shared UI persistent across navigations.
- To enable nested routing without duplicating layout code.
- To compose multi-level layouts.

### Syntax Rules and Structure

```jsx
import { Outlet } from 'react-router';

function DashboardLayout() {
  return (
    <div>
      <Sidebar />
      <main><Outlet /></main>
    </div>
  );
}
```

**Syntax Rules:**

1. Render `<Outlet />` where child routes should appear.
2. The parent route must have child routes defined.
3. `Outlet` renders the child route's element.
4. Index routes render into the parent's outlet at the parent URL.

### Annotated Code Examples

```jsx
<Route path="/dashboard" element={<DashboardLayout />}>
  <Route index element={<Overview />} />
  <Route path="reports" element={<Reports />} />
</Route>
```

**Expected Output:** `/dashboard` renders `Overview` inside the dashboard layout. `/dashboard/reports` renders `Reports` inside the same layout.

**Why this output:** The parent route renders `<Outlet />`, and the matched child route fills it.

### Real-World Cases

- **Dashboards:** Persistent sidebar with changing content.
- **Settings pages:** Shared navigation with per-section content.
- **Documentation:** Persistent sidebar and header.

### References

- React Router — Outlet — https://reactrouter.com/api/components/Outlet
- React Router — Routing (Index Route) — https://reactrouter.com/start/framework/routing

---

## 11. `useNavigate`

### Definitions

**Core Definition:** `useNavigate` is a hook that returns a function for programmatic navigation.

**Technical Definition:** `const navigate = useNavigate()` returns a function that navigates to a path, with options for `replace`, `state`, and relative resolution. It is used for navigation in event handlers, effects, and async callbacks where a `<Link>` cannot be used.

**Beginner-Friendly Explanation:** `useNavigate` gives you a function to change the page from your code — after a form submit, a timeout, or any other event.

### Purposes

- To navigate programmatically after an action.
- To redirect after form submission.
- To navigate in effects or async callbacks.
- To replace history entries.

### Syntax Rules and Structure

```jsx
import { useNavigate } from 'react-router';

const navigate = useNavigate();

navigate('/dashboard');
navigate('/login', { replace: true });
navigate(-1); // go back
navigate(1); // go forward
```

**Syntax Rules:**

1. Call `useNavigate()` inside a component under a router.
2. Pass a path string or a number (history delta).
3. Use `{ replace: true }` to replace the current entry.
4. Use `{ state: {...} }` to pass state.

### Annotated Code Examples

```jsx
function LoginForm() {
  const navigate = useNavigate();

  async function handleSubmit(e) {
    e.preventDefault();
    await login();
    navigate('/dashboard', { replace: true });
  }

  return <form onSubmit={handleSubmit}>{/* fields */}</form>;
}
```

**Expected Output:** After login, the app navigates to `/dashboard` and replaces the login page in history.

**Why this output:** `navigate('/dashboard', { replace: true })` pushes a replacement history entry.

### Real-World Cases

- **Post-login redirects:** Navigating to the dashboard.
- **Post-submission navigation:** Redirecting after creating a resource.
- **Go back:** `navigate(-1)` for cancel buttons.

### References

- React Router — useNavigate — https://reactrouter.com/api/hooks/useNavigate

---

## 12. `useParams`

### Definitions

**Core Definition:** `useParams` is a hook that returns an object of key-value pairs of the dynamic parameters from the current URL.

**Technical Definition:** `const { id } = useParams()` extracts the values of `:id` (and other dynamic segments) from the matched route path. Params are always strings; they must be parsed and validated before use.

**Beginner-Friendly Explanation:** `useParams` gives you the values from the dynamic parts of the URL, like the `123` in `/users/123`.

### Purposes

- To read dynamic URL segments in a component.
- To select a resource by its identifier.
- To drive data fetching and rendering based on the URL.
- To pass route parameters to loaders or components.

### Syntax Rules and Structure

```jsx
import { useParams } from 'react-router';

function UserDetail() {
  const { id } = useParams();
  const userId = Number(id);
  // ...
}
```

**Syntax Rules:**

1. Call `useParams()` inside a component under the matched route.
2. Destructure the params you need.
3. Params are strings; parse and validate them.
4. The hook returns an empty object if no params match.

### Annotated Code Examples

```tsx
function ProductDetail() {
  const { category, id } = useParams();
  return <h1>{category} / {id}</h1>;
}
```

**Expected Output:** `/products/shoes/42` renders “shoes / 42”.

**Why this output:** `useParams` extracts `category` and `id` from the route pattern `/products/:category/:id`.

### Real-World Cases

- **User profiles:** `/users/:id`.
- **Product pages:** `/products/:category/:id`.
- **Multi-tenant apps:** `/orgs/:orgId/settings`.

### References

- React Router — useParams — https://reactrouter.com/api/hooks/useParams
- React Router — URL Values — https://reactrouter.com/start/declarative/url-values

---

## 13. `useSearchParams`

### Definitions

**Core Definition:** `useSearchParams` is a hook that reads and writes the URL's query string (search params).

**Technical Definition:** `const [searchParams, setSearchParams] = useSearchParams()` returns a `URLSearchParams` object and a setter. Reading uses `.get()`, `.getAll()`, and `.entries()`. Writing updates the URL's query string, triggering a navigation. An options object supports `{ replace: true }` to replace the history entry.

**Beginner-Friendly Explanation:** `useSearchParams` lets you read and change the query string — the part of the URL after `?` — so you can store filters, sorting, and pagination in the URL.

### Purposes

- To read query parameters from the URL.
- To update query parameters without reloading.
- To store view state (filters, pagination) in the URL.
- To make filtered views shareable and bookmarkable.

### Syntax Rules and Structure

```jsx
import { useSearchParams } from 'react-router';

const [searchParams, setSearchParams] = useSearchParams();
const page = Number(searchParams.get('page') ?? '1');

setSearchParams({ page: '2', sort: 'asc' });
setSearchParams(prev => { prev.set('page', '3'); return prev; });
```

**Syntax Rules:**

1. `searchParams` is a `URLSearchParams` instance.
2. Use `setSearchParams` to update the query string.
3. Pass `{ replace: true }` to replace the history entry.
4. Use the functional updater to modify the previous params.
5. Search params are strings; parse and validate them.

### Annotated Code Examples

```jsx
function ProductList() {
  const [searchParams, setSearchParams] = useSearchParams();
  const sort = searchParams.get('sort') ?? 'relevance';

  function updateSort(nextSort) {
    setSearchParams({ sort: nextSort, page: '1' }, { replace: true });
  }

  return (
    <div>
      <p>Sorted by {sort}</p>
      <button onClick={() => updateSort('price')}>Sort by price</button>
    </div>
  );
}
```

**Expected Output:** Clicking “Sort by price” updates the URL to `?sort=price&page=1` and re-renders the list.

**Why this output:** `setSearchParams` writes to `location.search` and triggers a navigation. `replace: true` avoids a history entry.

### Real-World Cases

- **E-commerce filters:** `?category=shoes&sort=price`.
- **Search results:** `?q=react&page=2`.
- **Dashboards:** `?range=30d&tab=revenue`.

### References

- React Router — useSearchParams — https://reactrouter.com/api/hooks/useSearchParams
- React Router — URL Values — https://reactrouter.com/start/declarative/url-values

---

## 14. `useLocation`

### Definitions

**Core Definition:** `useLocation` is a hook that returns the current location object, including the pathname, search string, hash, state, and key.

**Technical Definition:** `const location = useLocation()` returns an object with `pathname`, `search`, `hash`, `state`, and `key`. The `state` is the data passed via `<Link state={...}>` or `navigate(path, { state })`. The `key` is a unique identifier for the history entry, useful for scroll restoration and analytics.

**Beginner-Friendly Explanation:** `useLocation` tells you where the user is right now — the path, the query string, and any state passed during navigation.

### Purposes

- To read the current URL pathname and search string.
- To access navigation state passed via `Link` or `navigate`.
- To trigger side effects on location changes (analytics, scroll).
- To access the history entry key for restoration.

### Syntax Rules and Structure

```jsx
import { useLocation } from 'react-router';

const location = useLocation();
location.pathname; // "/users/123"
location.search;   // "?tab=profile"
location.hash;     // "#section"
location.state;    // { from: 'home' }
location.key;      // "abc123"
```

**Syntax Rules:**

1. Call `useLocation()` inside a component under a router.
2. `pathname` is the URL path.
3. `search` includes the leading `?`.
4. `state` is the data passed during navigation.
5. `key` is a unique string for the history entry.

### Annotated Code Examples

```jsx
function Analytics() {
  const location = useLocation();

  useEffect(() => {
    trackPageView(location.pathname);
  }, [location.pathname]);

  return null;
}
```

**Expected Output:** A page view is tracked every time the pathname changes.

**Why this output:** The effect depends on `location.pathname`, so it re-runs on navigation.

### Real-World Cases

- **Analytics:** Tracking page views.
- **Scroll restoration:** Using `location.key` to restore scroll.
- **Breadcrumbs:** Building breadcrumbs from the pathname.

### References

- React Router — useLocation — https://reactrouter.com/api/hooks/useLocation
- React Router — URL Values — https://reactrouter.com/start/declarative/url-values

---

## 15. `useMatch`

### Definitions

**Core Definition:** `useMatch` is a hook that returns match information when the current URL matches a given path pattern, or `null` otherwise.

**Technical Definition:** `const match = useMatch('/users/:id')` returns an object with `params`, `pathname`, `pathnameBase`, and `pattern` when the current location matches the pattern. It is useful for conditional rendering based on the current route and for extracting params in components that are not direct route children.

**Beginner-Friendly Explanation:** `useMatch` checks whether the current URL matches a pattern. It is useful when you need to know “am I on this page?” inside a component.

### Purposes

- To conditionally render based on the current route.
- To extract params in components outside the route's direct tree.
- To implement active-state logic for custom navigation components.
- To match against patterns that are not defined as routes.

### Syntax Rules and Structure

```jsx
import { useMatch } from 'react-router';

const match = useMatch('/users/:id');
if (match) {
  const { id } = match.params;
}
```

**Syntax Rules:**

1. Pass a path pattern to `useMatch`.
2. Returns `null` if the current location does not match.
3. Returns an object with `params`, `pathname`, `pathnameBase`, and `pattern` if it matches.
4. Params are strings.

### Annotated Code Examples

```jsx
function UserBreadcrumb() {
  const match = useMatch('/users/:id');
  if (!match) return null;
  return <span>User {match.params.id}</span>;
}
```

**Expected Output:** The breadcrumb renders only when the URL matches `/users/:id`.

**Why this output:** `useMatch` returns the match object when the pattern matches, and `null` otherwise.

### Real-World Cases

- **Conditional UI:** Showing a sidebar only on specific routes.
- **Breadcrumbs:** Extracting params for breadcrumb labels.
- **Custom nav components:** Implementing active-state logic.

### References

- React Router — useMatch — https://reactrouter.com/api/hooks/useMatch

---

## 16. `useResolvedPath`

### Definitions

**Core Definition:** `useResolvedPath` is a hook that resolves a path string to an absolute path based on the current location.

**Technical Definition:** `const resolved = useResolvedPath('..')` returns a `Path` object with the resolved `pathname`, `search`, and `hash`. It is used to resolve relative paths into absolute ones, especially for building links or matching logic.

**Beginner-Friendly Explanation:** `useResolvedPath` takes a relative path and tells you the absolute path it points to.

### Purposes

- To resolve relative paths to absolute paths.
- To build correct `href` values for custom link components.
- To compute the destination of a relative link for prefetching or analytics.
- To support relative path resolution in custom navigation logic.

### Syntax Rules and Structure

```jsx
import { useResolvedPath } from 'react-router';

const resolved = useResolvedPath('../about');
// resolved.pathname is the absolute path
```

**Syntax Rules:**

1. Pass a path string (relative or absolute) to `useResolvedPath`.
2. Returns a `Path` object with `pathname`, `search`, and `hash`.
3. Relative paths are resolved against the current location.

### Annotated Code Examples

```jsx
function CustomLink({ to, children }) {
  const resolved = useResolvedPath(to);
  const href = useHref(resolved);
  return <a href={href}>{children}</a>;
}
```

**Expected Output:** The `href` is the absolute URL for the relative `to` path.

**Why this output:** `useResolvedPath` converts the relative path to an absolute path, and `useHref` converts it to a full URL string.

### Real-World Cases

- **Custom link components:** Building `href` values.
- **Prefetching:** Resolving the destination before prefetching.
- **Analytics:** Logging the resolved destination.

### References

- React Router — useResolvedPath — https://reactrouter.com/api/hooks/useResolvedPath

---

## 17. `useHref`

### Definitions

**Core Definition:** `useHref` is a hook that returns the full URL string for a given path, including the router's `basename`.

**Technical Definition:** `const href = useHref('/about')` returns the `href` string that would navigate to the given path. It combines the router's `basename` with the path, making it useful for building custom link components or `<a>` elements that integrate with the router.

**Beginner-Friendly Explanation:** `useHref` gives you the actual URL for a path, including any base path the router is configured with.

### Purposes

- To build `href` values for custom link components.
- To include the router's `basename` in URLs.
- To generate correct URLs for `<a>` elements or sharing.
- To integrate third-party link components with the router.

### Syntax Rules and Structure

```jsx
import { useHref } from 'react-router';

const href = useHref('/about');
```

**Syntax Rules:**

1. Pass a path string to `useHref`.
2. Returns a string with the router's `basename` prepended.
3. Use it for custom link components or `href` attributes.

### Annotated Code Examples

```jsx
function ExternalLink({ to, children }) {
  const href = useHref(to);
  return <a href={href}>{children}</a>;
}
```

**Expected Output:** The `href` includes the router's `basename` and the given path.

**Why this output:** `useHref` combines the base path with the provided path.

### Real-World Cases

- **Custom link components:** Building accessible links.
- **Third-party integration:** Wiring non-React-Router links to the router.
- **URL generation:** Producing correct URLs for sharing.

### References

- React Router — useHref — https://reactrouter.com/api/hooks/useHref

---

## 18. `useNavigationType`

### Definitions

**Core Definition:** `useNavigationType` is a hook that returns the type of navigation that led to the current location: `POP`, `PUSH`, or `REPLACE`.

**Technical Definition:** `const navigationType = useNavigationType()` returns how the user arrived at the current location. `POP` means the user pressed back/forward or loaded the initial page; `PUSH` means a new history entry was pushed; `REPLACE` means the current entry was replaced. It is useful for distinguishing between navigations and initial loads.

**Beginner-Friendly Explanation:** `useNavigationType` tells you how the user got to the current page — did they click a link, press back, or was the page replaced?

### Purposes

- To distinguish between forward navigation and back/forward navigation.
- To implement scroll restoration only on POP or only on PUSH.
- To trigger analytics differently for back/forward vs. new navigations.
- To conditionally run effects based on how the location was reached.

### Syntax Rules and Structure

```jsx
import { useNavigationType } from 'react-router';

const navigationType = useNavigationType(); // 'POP' | 'PUSH' | 'REPLACE'
```

**Syntax Rules:**

1. Returns `'POP'`, `'PUSH'`, or `'REPLACE'`.
2. `POP` includes back/forward and initial load.
3. `PUSH` is a new history entry (typical link click).
4. `REPLACE` is a history entry replacement (redirects).

### Annotated Code Examples

```jsx
function ScrollRestoration() {
  const navigationType = useNavigationType();
  const { key } = useLocation();

  useEffect(() => {
    if (navigationType === 'POP') {
      restoreScroll(key);
    } else {
      window.scrollTo(0, 0);
    }
  }, [key, navigationType]);

  return null;
}
```

**Expected Output:** Back/forward navigations restore scroll position; new navigations scroll to the top.

**Why this output:** `navigationType === 'POP'` indicates back/forward navigation, while `PUSH`/`REPLACE` indicate new navigation.

### Real-World Cases

- **Scroll restoration:** Restoring on POP, resetting on PUSH.
- **Analytics:** Differentiating back/forward from new page views.
- **Transitions:** Applying different animations for POP vs. PUSH.

### References

- React Router — useNavigationType — https://reactrouter.com/api/hooks/useNavigationType

---

## 19. Route Configuration

### Definitions

**Core Definition:** Route configuration is the way routes are defined and organized in an application, either as JSX `<Routes>`/`<Route>` elements or as route objects in Data Mode.

**Technical Definition:** In Declarative Mode, routes are configured as JSX inside `<Routes>`. In Data Mode, routes are configured as plain JavaScript objects passed to `createBrowserRouter`. In Framework Mode, routes are defined in `routes.ts` and/or file-based route modules. Route configuration defines paths, components, loaders, actions, error boundaries, and lazy loading.

**Beginner-Friendly Explanation:** Route configuration is how you tell React Router what routes exist and what they do.

### Purposes

- To define the mapping from URLs to components.
- To attach data loading and mutation logic (Data Mode).
- To configure error boundaries and lazy loading.
- To organize nested layouts and index routes.
- To enable type-safe routing in Framework Mode.

### Syntax Rules and Structure

**Declarative Mode:**

```jsx
<Routes>
  <Route path="/" element={<Home />} />
  <Route path="/users/:id" element={<UserDetail />} />
</Routes>
```

**Data Mode:**

```js
const router = createBrowserRouter([
  { path: '/', Component: Home },
  { path: '/users/:id', Component: UserDetail, loader: userLoader },
]);
```

**Framework Mode:**

```ts
// app/routes.ts
import { type RouteConfig, index, route } from '@react-router/dev/routes';

export default [
  index('routes/home.tsx'),
  route('users/:id', 'routes/user.tsx'),
] satisfies RouteConfig;
```

**Syntax Rules:**

1. Choose the configuration style based on the mode.
2. Define `path`, `element`/`Component`, and optional `loader`/`action`/`errorElement`.
3. Use nested route objects/JSX for shared layouts.
4. Use `index` for the default child route.
5. Use `lazy` for route-level code splitting.

### Annotated Code Examples

```js
const router = createBrowserRouter([
  {
    path: '/',
    Component: RootLayout,
    errorElement: <RootError />,
    children: [
      { index: true, Component: Home },
      { path: 'dashboard', Component: Dashboard, loader: dashboardLoader },
    ],
  },
]);
```

**Expected Output:** `/` renders `Home` inside `RootLayout`; `/dashboard` renders `Dashboard` with data from `dashboardLoader`.

**Why this output:** The route object tree defines the hierarchy. `index: true` makes `Home` the default child. `loader` provides data before rendering.

### Real-World Cases

- **All React Router apps:** Route configuration is the foundation.
- **Data-driven apps:** Route objects with loaders and actions.
- **Full-stack apps:** File-based route modules.

### References

- React Router — Route Object — https://beta.reactrouter.com/7.6.0/start/data/route-object
- React Router — Data Mode — https://www.mintlify.com/remix-run/react-router/getting-started/data-mode
- React Router — Routing (Framework Mode) — https://reactrouter.com/start/framework/routing

---

## 20. Route Nesting

### Definitions

**Core Definition:** Route nesting is the practice of defining child routes inside parent routes, so parent layouts persist while child content changes.

**Technical Definition:** Nested routes create a route hierarchy. The parent route renders shared layout and `<Outlet />`; the matched child route renders into the outlet. URL paths also nest: a child path `reports` under parent `/dashboard` matches `/dashboard/reports`.

**Beginner-Friendly Explanation:** Nested routes let you share a layout across pages. The parent stays, and the child swaps in and out.

### Purposes

- To share layouts across related routes.
- To avoid duplicating navigation and headers.
- To persist layout state across child navigation.
- To mirror the application's information architecture.
- To enable route-level code splitting per nested segment.

### Syntax Rules and Structure

**Declarative Mode:**

```jsx
<Route path="/dashboard" element={<DashboardLayout />}>
  <Route index element={<Overview />} />
  <Route path="reports" element={<Reports />} />
</Route>
```

**Data Mode:**

```js
{
  path: '/dashboard',
  Component: DashboardLayout,
  children: [
    { index: true, Component: Overview },
    { path: 'reports', Component: Reports },
  ],
}
```

**Syntax Rules:**

1. Child paths are relative to the parent (no leading `/`).
2. Parent renders `<Outlet />` where children appear.
3. Use `index: true` or `<Route index>` for the default child.
4. Nested routes can be arbitrarily deep.

### Annotated Code Examples

```jsx
function DashboardLayout() {
  return (
    <div>
      <Sidebar />
      <main><Outlet /></main>
    </div>
  );
}

<Route path="/dashboard" element={<DashboardLayout />}>
  <Route index element={<Overview />} />
  <Route path="reports" element={<Reports />} />
</Route>
```

**Expected Output:** Navigating from `/dashboard` to `/dashboard/reports` keeps the sidebar and swaps only the main content.

**Why this output:** The parent route renders `<Outlet />`, which displays the matched child. The layout persists across child navigation.

### Real-World Cases

- **Dashboards:** Persistent sidebar with changing content.
- **Settings:** Shared navigation with per-section content.
- **E-commerce:** Product layout with tabs.

### References

- React Router — Routing (Nested Routes) — https://reactrouter.com/start/framework/routing
- React Router — Outlet — https://reactrouter.com/api/components/Outlet

---

## 21. Route Layouts

### Definitions

**Core Definition:** Route layouts are parent route components that render shared UI around child routes through an `<Outlet />`.

**Technical Definition:** A layout route defines a component that wraps child routes. The layout component renders persistent UI (header, sidebar, footer) and an `<Outlet />` where the child route's element appears. Layouts are the implementation of nested routing.

**Beginner-Friendly Explanation:** A layout is the frame around your pages. The layout stays the same while the page inside changes.

### Purposes

- To share persistent UI across routes.
- To avoid duplicating headers, sidebars, and navigation.
- To preserve layout state (e.g., sidebar scroll) across navigation.
- To compose multi-level layouts.

### Syntax Rules and Structure

```jsx
function RootLayout() {
  return (
    <>
      <Header />
      <main><Outlet /></main>
      <Footer />
    </>
  );
}
```

**Syntax Rules:**

1. Create a component that renders shared UI and `<Outlet />`.
2. Attach it to a parent route with child routes.
3. The layout persists across child navigation.
4. Layouts can be nested for multi-level composition.

### Annotated Code Examples

```jsx
function SettingsLayout() {
  return (
    <section>
      <nav aria-label="Settings">
        <NavLink to="profile">Profile</NavLink>
        <NavLink to="billing">Billing</NavLink>
      </nav>
      <Outlet />
    </section>
  );
}
```

**Expected Output:** The settings navigation stays visible while the profile or billing content changes.

**Why this output:** `SettingsLayout` is the parent route; `<Outlet />` renders the matched child. The layout persists across child navigation.

### Real-World Cases

- **Dashboards:** Persistent sidebar and header.
- **Settings pages:** Shared settings navigation.
- **Documentation:** Persistent sidebar and table of contents.

### References

- React Router — Routing (Layouts) — https://reactrouter.com/start/framework/routing
- React Router — Outlet — https://reactrouter.com/api/components/Outlet

---

## 22. Route Loaders

### Definitions

**Core Definition:** Route loaders are functions that fetch data for a route before the route's component renders.

**Technical Definition:** In Data Mode, a `loader` function runs before the route renders. It receives `{ request, params }` and returns data that is available via `useLoaderData()`. Loaders run in parallel for nested routes, avoiding waterfalls. They can throw `Response` objects to trigger error boundaries and redirects.

**Beginner-Friendly Explanation:** A loader is a function that gets the data ready before the page appears. The component can then render with data already available.

### Purposes

- To fetch data before rendering a route.
- To avoid loading spinners inside components.
- To run loaders in parallel for nested routes.
- To handle errors and redirects declaratively.
- To keep data fetching colocated with route definitions.

### Syntax Rules and Structure

```js
const router = createBrowserRouter([
  {
    path: '/users/:id',
    loader: async ({ params }) => {
      const user = await fetchUser(params.id);
      if (!user) throw new Response('Not Found', { status: 404 });
      return { user };
    },
    Component: UserDetail,
  },
]);
```

**Syntax Rules:**

1. Define `loader` on the route object.
2. The loader receives `{ request, params }`.
3. Return data to be accessed via `useLoaderData()`.
4. Throw `Response` for errors (`404`, `500`) and `redirect()` for redirects.
5. Loaders run in parallel for nested routes.

**Constraints and Limitations:**

- Loaders run on navigation; slow loaders delay rendering.
- Loader data is revalidated after actions by default.

### Annotated Code Examples

```jsx
import { useLoaderData } from 'react-router';

export async function loader({ params }) {
  const user = await fetchUser(params.id);
  return { user };
}

export function UserDetail() {
  const { user } = useLoaderData();
  return <h1>{user.name}</h1>;
}
```

**Expected Output:** The loader runs before render, so `UserDetail` receives `user` immediately and does not render a loading state.

**Why this output:** `useLoaderData()` provides the loader's return value. The router runs the loader before rendering the route.

### Real-World Cases

- **Data-heavy routes:** Dashboards and detail pages.
- **SEO-critical pages:** Server-rendered content with loaders.
- **Parallel data fetching:** Nested routes with independent loaders.

### References

- React Router — Route Object: loader — https://beta.reactrouter.com/7.6.0/start/data/route-object
- React Router — Data Loading — https://www.mintlify.com/remix-run/react-router/getting-started/data-mode

---

## 23. Route Actions

### Definitions

**Core Definition:** Route actions are functions that handle data mutations (form submissions) for a route.

**Technical Definition:** An `action` function runs when a form is submitted to a route. It receives `{ request, params }`, reads `request.formData()`, performs the mutation, and returns data or a redirect. After an action completes, React Router automatically revalidates all loader data on the page, keeping the UI in sync.

**Beginner-Friendly Explanation:** An action is what happens when a form is submitted. It handles the data change and then refreshes the data on the page.

### Purposes

- To handle form submissions and data mutations.
- To automatically revalidate loader data after mutations.
- To keep mutations colocated with route definitions.
- To support progressive enhancement with `<Form>`.
- To return validation errors or redirects.

### Syntax Rules and Structure

```js
{
  path: '/projects/new',
  action: async ({ request }) => {
    const formData = await request.formData();
    const project = await createProject({
      name: formData.get('name'),
    });
    return redirect(`/projects/${project.id}`);
  },
  Component: NewProject,
}
```

**Syntax Rules:**

1. Define `action` on the route object.
2. Read form data with `request.formData()`.
3. Return data (available via `useActionData()`) or `redirect()`.
4. Actions trigger automatic revalidation of loaders.
5. Use `<Form method="post">` to submit to the route's action.

### Annotated Code Examples

```jsx
import { Form, redirect } from 'react-router';

export async function action({ request }) {
  const formData = await request.formData();
  const project = await createProject({
    name: formData.get('name'),
  });
  return redirect(`/projects/${project.id}`);
}

export function NewProject() {
  return (
    <Form method="post">
      <input name="name" />
      <button type="submit">Create</button>
    </Form>
  );
}
```

**Expected Output:** Submitting the form calls the action, creates the project, and redirects to the new project's page. Loader data is revalidated.

**Why this output:** `<Form method="post">` submits to the route's `action`. The action returns a redirect, and React Router revalidates loaders.

### Real-World Cases

- **Create/update/delete operations:** Forms that mutate data.
- **Login/logout:** Authentication actions.
- **Settings updates:** Profile and preference changes.

### References

- React Router — Route Object: action — https://beta.reactrouter.com/7.6.0/start/data/route-object
- React Router — Data Mutations — https://www.mintlify.com/remix-run/react-router/getting-started/data-mode

---

## 24. Route Errors

### Definitions

**Core Definition:** Route errors are error boundaries attached to routes that catch errors from loaders, actions, and rendering.

**Technical Definition:** In Data Mode, `errorElement` on a route object defines the UI rendered when that route (or its children) throws. Errors bubble to the nearest ancestor with an `errorElement`. The `useRouteError()` hook provides the error for display. `isRouteErrorResponse()` distinguishes thrown `Response` objects from other errors.

**Beginner-Friendly Explanation:** Route errors are error boundaries for routes. If a loader fails or a component crashes, the nearest error route shows a fallback instead of a blank page.

### Purposes

- To catch and display errors from loaders, actions, and rendering.
- To prevent a single route failure from crashing the app.
- To provide route-specific error UIs.
- To bubble errors to the nearest error boundary.
- To distinguish between HTTP errors and unexpected errors.

### Syntax Rules and Structure

```js
{
  path: '/users/:id',
  loader: userLoader,
  Component: UserDetail,
  errorElement: <UserError />,
}
```

```jsx
import { useRouteError, isRouteErrorResponse } from 'react-router';

function UserError() {
  const error = useRouteError();
  if (isRouteErrorResponse(error)) {
    return <h1>{error.status} {error.statusText}</h1>;
  }
  return <h1>Something went wrong</h1>;
}
```

**Syntax Rules:**

1. Add `errorElement` to the route object.
2. Use `useRouteError()` to access the error.
3. Use `isRouteErrorResponse()` to check for HTTP errors.
4. Errors bubble to the nearest ancestor with an `errorElement`.
5. Provide a root `errorElement` as a catch-all.

### Annotated Code Examples

```jsx
const router = createBrowserRouter([
  {
    path: '/',
    Component: Root,
    errorElement: <RootError />,
    children: [
      { path: 'users/:id', Component: UserDetail, loader: userLoader },
    ],
  },
]);

function RootError() {
  const error = useRouteError();
  return isRouteErrorResponse(error)
    ? <p>{error.status} {error.statusText}</p>
    : <p>Unexpected error</p>;
}
```

**Expected Output:** A loader error in `/users/:id` renders `RootError` because the child route has no `errorElement`.

**Why this output:** Errors bubble to the nearest ancestor with an `errorElement`. The root route provides the catch-all.

### Real-World Cases

- **Loader failures:** API errors and 404s.
- **Action failures:** Validation errors and server errors.
- **Component crashes:** Rendering errors in route components.

### References

- React Router — Error Boundaries — https://reactrouter.com/start/data/error-boundaries
- React Router — useRouteError — https://reactrouter.com/api/hooks/useRouteError
- React Router — isRouteErrorResponse — https://reactrouter.com/api/utils/isRouteErrorResponse

---

## 25. Route Lazy Loading

### Definitions

**Core Definition:** Route lazy loading is the practice of deferring the loading of a route's code (component, loader, action) until the route is navigated to.

**Technical Definition:** In Data Mode, the `lazy` property on a route object is a function that returns a Promise resolving to the route's lazily loaded properties (`Component`, `loader`, `action`, etc.). In Framework Mode, routes are automatically code-split. The `lazy` function can load multiple properties in parallel with `Promise.all()`.

**Beginner-Friendly Explanation:** Lazy loading means the code for a route is only downloaded when you navigate to it, not upfront.

### Purposes

- To reduce the initial bundle size.
- To load route code on demand.
- To improve initial load performance.
- To split code by route boundaries.
- To load loader, action, and component code in parallel.

### Syntax Rules and Structure

```js
{
  path: '/reports',
  lazy: async () => {
    const [Component, loader] = await Promise.all([
      import('./pages/Reports'),
      import('./loaders/reportsLoader'),
    ]);
    return { Component, loader };
  },
}
```

**Syntax Rules:**

1. Use the `lazy` property on a route object.
2. Return a Promise resolving to route properties.
3. Use `Promise.all()` to load multiple properties in parallel.
4. The `lazy` function runs on first navigation to the route.
5. In Framework Mode, routes are automatically split.

### Annotated Code Examples

```jsx
const router = createBrowserRouter([
  {
    path: '/dashboard',
    lazy: async () => {
      const { Dashboard, dashboardLoader } = await import('./dashboard');
      return { Component: Dashboard, loader: dashboardLoader };
    },
  },
]);
```

**Expected Output:** The dashboard code is not in the initial bundle; it is downloaded when the user navigates to `/dashboard`.

**Why this output:** The `lazy` function defers the import until navigation, and the router waits for the Promise before rendering.

### Real-World Cases

- **Large admin sections:** Loading admin code only when accessed.
- **Heavy reports:** Deferring report components and loaders.
- **Settings pages:** Loading settings code on demand.

### References

- React Router — Route Object: lazy — https://beta.reactrouter.com/7.6.0/start/data/route-object
- React Router — Automatic Code Splitting — https://beta.reactrouter.com/7.3.0/start/framework/automatic-code-splitting
- React Router — Bring Your Own Bundler: Lazy Loading Routes — https://beta.reactrouter.com/7.0.1/start/framework/bring-your-own-bundler

---

## 26. Route Data APIs

### Definitions

**Core Definition:** Route data APIs are the set of hooks, components, and route properties that enable data loading, mutations, and pending states in Data Mode.

**Technical Definition:** React Router's data APIs include `loader`, `action`, `useLoaderData`, `useActionData`, `useFetcher`, `useNavigation`, `useRouteError`, `shouldRevalidate`, and `<Form>`. These APIs decouple data fetching from rendering, run loaders before components render, and automatically revalidate after actions.

**Beginner-Friendly Explanation:** Data APIs are the tools React Router gives you to load data and handle form submissions before rendering, instead of doing it inside components with `useEffect`.

### Purposes

- To load data before rendering routes.
- To handle mutations and revalidate automatically.
- To provide pending UI states during navigation and submission.
- To handle errors at the route level.
- To decouple data fetching from component rendering.

### Syntax Rules and Structure

**Data API Summary:**

| API | Purpose |
|-----|---------|
| `loader` | Fetch data before render |
| `action` | Handle form mutations |
| `useLoaderData()` | Access loader data in components |
| `useActionData()` | Access action return data |
| `useFetcher()` | Programmatic data loading/mutation |
| `useNavigation()` | Pending navigation state |
| `useRouteError()` | Route error handling |
| `<Form>` | Declarative form submission to actions |

**Syntax Rules:**

1. Define loaders and actions on route objects.
2. Access data with `useLoaderData()` and `useActionData()`.
3. Use `useNavigation()` for pending states.
4. Use `<Form>` for progressive-enhancement-friendly submissions.
5. Use `useFetcher()` for non-navigation data operations.

### Annotated Code Examples

```jsx
function Root() {
  const navigation = useNavigation();
  const isLoading = navigation.state === 'loading';

  return (
    <>
      {isLoading && <TopProgressBar />}
      <Outlet />
    </>
  );
}
```

**Expected Output:** A progress bar appears at the top of the page during navigation.

**Why this output:** `useNavigation()` returns the current navigation state; `state === 'loading'` indicates a pending navigation.

### Real-World Cases

- **Data-driven SPAs:** Loaders and actions for data loading.
- **Progressive enhancement:** `<Form>` works without JavaScript.
- **Optimistic UI:** `useFetcher()` for non-navigation mutations.

### References

- React Router — Data Mode — https://www.mintlify.com/remix-run/react-router/getting-started/data-mode
- React Router — Route Object — https://beta.reactrouter.com/7.6.0/start/data/route-object
- React Router — useNavigation — https://reactrouter.com/api/hooks/useNavigation

---

## 27. React Router Versions

### Definitions

**Core Definition:** React Router versions denote the major releases of the library, each introducing new architectures, APIs, and modes.

**Technical Definition:** React Router v5 and earlier used a component-based API with `<Switch>` and `useHistory`. v6 introduced `<Routes>` and `<Route element>`, `useNavigate`, and nested routing. v6.4 added the data APIs (`createBrowserRouter`, `loader`, `action`). v7 merged Remix into React Router, introducing three modes (Declarative, Data, Framework) and file-based routing. v8 (2026) continues the v7 architecture with further refinements. The package name is `react-router` for v7+; `react-router-dom` is the v6 and earlier package.

**Beginner-Friendly Explanation:** React Router has evolved over time. Older versions used different APIs. If you are using v6 or v7, you are on a modern version. v7 is the current major version with three modes.

### Purposes

- To understand the API differences between versions.
- To plan upgrades from v5/v6 to v7.
- To choose the right package (`react-router` vs `react-router-dom`).
- To understand the three modes introduced in v7.

### Version History Table

| Version | Key Changes | Package |
|---------|-------------|---------|
| v5 | `<Switch>`, `useHistory`, `component`/`render` props | `react-router-dom` |
| v6 | `<Routes>`, `element`, `useNavigate`, nested routes | `react-router-dom` |
| v6.4 | Data APIs (`createBrowserRouter`, `loader`, `action`) | `react-router-dom` |
| v7 | Merged Remix, three modes (Declarative, Data, Framework) | `react-router` |
| v8 | Continued v7 architecture (2026) | `react-router` |

**Syntax Rules:**

1. For v7+, import from `react-router` (not `react-router-dom`).
2. Use the migration guide when upgrading from v6.
3. Choose the mode based on app complexity.
4. Declarative Mode is the classic `<BrowserRouter>` experience.
5. Data Mode uses `createBrowserRouter` with loaders/actions.
6. Framework Mode is the full-stack SSR mode.

### Annotated Code Examples

```jsx
// v7 Declarative Mode
import { BrowserRouter, Routes, Route } from 'react-router';

// v7 Data Mode
import { createBrowserRouter } from 'react-router';
```

**Expected Output:** The import path is `react-router` for v7+, not `react-router-dom`.

**Why this output:** v7 unified the package name. `react-router-dom` is the v6 package.

### Real-World Cases

- **Legacy migrations:** Upgrading from v5 or v6 to v7.
- **New projects:** Starting with v7 Data or Framework Mode.
- **Mode selection:** Choosing between Declarative, Data, and Framework.

### References

- React Router — Picking a Mode — https://reactrouter.com/start/modes
- React Router — Upgrading from v6 — https://reactrouter.com/upgrading/v6
- React Router — Changelog — https://reactrouter.com/changelog

---

## 28. React Router Best Practices

### Definitions

**Core Definition:** React Router best practices are the guidelines for structuring routes, data loading, navigation, and error handling to build maintainable, performant, and accessible applications.

**Technical Definition:** Best practices encompass route design (semantic URLs, nested layouts), data strategy (loaders, parallel fetching, revalidation), performance (lazy loading, prefetching, code splitting), error handling (error boundaries, 404 routes), accessibility (focus management, `aria-current`), and security (server-enforced auth). The choice of mode (Declarative, Data, Framework) should follow from the rendering model (SPA, SSR, full-stack).

**Beginner-Friendly Explanation:** Good routing makes an app easy to use, easy to share, and easy to maintain. URLs should make sense, pages should load fast, errors should be handled gracefully, and navigation should be accessible.

### Best Practices Table

| Practice | Description |
|----------|-------------|
| **Design semantic URLs** | `/products/:slug` rather than `/p?id=123`. |
| **Use nested routes for layouts** | Parent layout + `<Outlet />` for shared UI. |
| **Lazy load routes** | Use `lazy` to keep the initial bundle small. |
| **Run loaders in parallel** | Nested loaders run in parallel; avoid waterfalls. |
| **Provide error boundaries** | `errorElement` on routes; root catch-all. |
| **Use `NavLink` for active state** | Automatic `aria-current` and active styling. |
| **Manage focus on navigation** | Move focus to main content after route change. |
| **Use `useSearchParams` for view state** | Filters, pagination, sorting in the URL. |
| **Enforce auth on the server** | Client guards are UX; server checks are security. |
| **Prefetch on intent** | Load route code/data on hover or visibility. |
| **Use `useNavigationType` for scroll restoration** | POP restores, PUSH resets. |
| **Choose the right mode** | Declarative for simple SPAs, Data for data-driven, Framework for SSR. |

### Annotated Code Examples

**✅ Good Practices:**

```tsx
import { createBrowserRouter, Outlet, redirect } from 'react-router';

async function requireAuth() {
  const user = await getUser();
  if (!user) throw redirect('/login');
  return user;
}

const router = createBrowserRouter([
  {
    path: '/',
    Component: RootLayout,
    errorElement: <RootError />,
    children: [
      { index: true, Component: Home },
      {
        path: 'dashboard',
        loader: requireAuth,
        lazy: () => import('./pages/Dashboard'),
      },
      { path: '*', Component: NotFound },
    ],
  },
]);
```

**❌ Bad Practices:**

```tsx
// ❌ Client-only auth after rendering
function Admin() {
  const user = useUser();
  if (!user) return <p>Log in</p>; // Page already rendered
  return <AdminPanel />;
}

// ❌ No error boundary
<Route path="/dashboard" element={<Dashboard />} />

// ❌ No lazy loading
import Dashboard from './pages/Dashboard';
```

**Expected Output:** The good practices produce secure, accessible, fast routes. The bad practices render protected content before checking access, crash on errors, and bloat the initial bundle.

**Why this output:** Client-only guards render before the check. Missing error boundaries crash the app. Eager imports bloat the bundle.

### Real-World Cases

- **All React Router applications:** These practices apply universally.
- **Multi-tenant SaaS:** Server-enforced auth and nested layouts.
- **E-commerce:** Semantic URLs and lazy-loaded routes.

### References

- React Router — Picking a Mode — https://reactrouter.com/start/modes
- React Router — Route Object — https://beta.reactrouter.com/7.6.0/start/data/route-object
- React Router — Error Boundaries — https://reactrouter.com/start/data/error-boundaries
- React Router — Data Mode — https://www.mintlify.com/remix-run/react-router/getting-started/data-mode

---

## References

- React Router — Official Documentation — https://reactrouter.com/home
- React Router — Picking a Mode — https://reactrouter.com/start/modes
- React Router — BrowserRouter (v7.5.3) — https://reactrouter.com/7.5.3/api/declarative-routers/BrowserRouter
- React Router — HashRouter (Picking a Router) — https://reactrouter.com/6.5.0/routers/picking-a-router
- React Router — MemoryRouter (Picking a Router) — https://reactrouter.com/6.5.0/routers/picking-a-router
- React Router — Routes — https://beta.reactrouter.com/6.26.2/components/Routes
- React Router — Route — https://beta.reactrouter.com/7.18.5/api/components/Route
- React Router — Link — https://reactrouter.com/api/components/Link
- React Router — NavLink — https://reactrouter.com/api/components/NavLink
- React Router — Navigate — https://reactrouter.com/api/components/Navigate
- React Router — Outlet — https://reactrouter.com/api/components/Outlet
- React Router — useNavigate — https://reactrouter.com/api/hooks/useNavigate
- React Router — useParams — https://reactrouter.com/api/hooks/useParams
- React Router — useSearchParams — https://reactrouter.com/api/hooks/useSearchParams
- React Router — useLocation — https://reactrouter.com/api/hooks/useLocation
- React Router — useMatch — https://reactrouter.com/api/hooks/useMatch
- React Router — useResolvedPath — https://reactrouter.com/api/hooks/useResolvedPath
- React Router — useHref — https://reactrouter.com/api/hooks/useHref
- React Router — useNavigationType — https://reactrouter.com/api/hooks/useNavigationType
- React Router — Route Object — https://beta.reactrouter.com/7.6.0/start/data/route-object
- React Router — Data Mode — https://www.mintlify.com/remix-run/react-router/getting-started/data-mode
- React Router — Error Boundaries — https://reactrouter.com/start/data/error-boundaries
- React Router — useRouteError — https://reactrouter.com/api/hooks/useRouteError
- React Router — isRouteErrorResponse — https://reactrouter.com/api/utils/isRouteErrorResponse
- React Router — Automatic Code Splitting — https://beta.reactrouter.com/7.3.0/start/framework/automatic-code-splitting
- React Router — Bring Your Own Bundler — https://beta.reactrouter.com/7.0.1/start/framework/bring-your-own-bundler
- React Router — Upgrading from v6 — https://reactrouter.com/upgrading/v6
- React Router — Changelog — https://reactrouter.com/changelog
- Ashukr321/Route-Architect — https://github.com/Ashukr321/Route-Architect
- DeepWiki — Hooks API — https://deepwiki.com/remix-run/react-router
- agents-inc/skills — web-routing-react-router — https://github.com/agents-inc/skills
- vasilyu1983/AI-Agents-public — Remix / React Router Patterns — https://github.com/vasilyu1983/AI-Agents-public