# React Router — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** React Router is the standard routing library for React applications. It enables client-side routing, allowing a single-page application (SPA) to display different views based on the URL without full-page reloads.

**Technical Definition:** React Router is a collection of React components, hooks, and utilities that synchronize the UI with the browser's URL. It provides declarative routing through `<Routes>` and `<Route>`, programmatic navigation via `useNavigate`, and data APIs (loaders, actions) that couple URL segments to data loading and mutations. The library is published across three packages: `react-router` (core), `react-router-dom` (web-specific additions), and `react-router-native` (React Native additions).

**Beginner-Friendly Explanation:** React Router lets you build a multi-page-feeling app using React. It watches the URL and shows the right component for each page — like `/home`, `/about`, or `/users/123` — without ever refreshing the browser.

### Key Characteristics

- **Client-side routing:** Navigation happens in JavaScript without requesting new HTML from the server.
- **Declarative configuration:** Routes are defined using JSX `<Routes>`/`<Route>` or JavaScript route objects.
- **Nested layouts:** Routes can be nested to create shared layouts and data boundaries.
- **Data APIs (v6.4+):** Loaders fetch data before rendering; actions handle form submissions and mutations.
- **Multiple router types:** `BrowserRouter` (clean URLs), `HashRouter` (hash-based URLs), `MemoryRouter` (testing), `StaticRouter` (SSR), `NativeRouter` (React Native).
- **Hooks-based access:** `useParams`, `useSearchParams`, `useLocation`, `useNavigate`, and others provide routing state to components.

### Prerequisites

- A React application (v16.8+ for hooks; v18+ for v7).
- A build tool configured for JSX (Vite, Next.js, Create React App, etc.).
- Basic understanding of React components, props, and state.
- Familiarity with URL structure (pathname, query string, hash).

### Related Programming Areas

- **Single-Page Application (SPA) architecture** — React Router is the de facto routing solution for React SPAs.
- **Server-Side Rendering (SSR)** — `StaticRouter` and framework integrations (Remix, Next.js) enable SSR with routing.
- **Data fetching and mutations** — Loaders and actions replace `useEffect`-based fetching with route-level data APIs.
- **Code splitting** — `route.lazy` enables route-level lazy loading for smaller initial bundles.
- **Accessibility** — `Link` and `NavLink` render accessible anchor tags with keyboard support.

### Core Concepts / Features

---

## 1. React Router

### Definitions

**Core Definition:** React Router is the library that provides routing capabilities to React applications.

**Technical Definition:** React Router is an npm package ecosystem (`react-router`, `react-router-dom`, `react-router-native`) that implements client-side routing. It maintains a history stack, matches URL paths to route configurations, and renders the appropriate component tree. The core package contains the route matching algorithm and most components/hooks; the DOM package adds browser-specific APIs.

**Beginner-Friendly Explanation:** React Router is the tool you install to give your React app multiple "pages." It listens to the URL and swaps components in and out as the URL changes.

### Purposes

- To enable navigation between different views in a React SPA without full page reloads.
- To keep the UI in sync with the browser URL (back/forward buttons, bookmarks, deep linking).
- To provide nested layouts, data loading, and error handling at the route level.
- To support multiple environments (web, native, server, testing) through interchangeable routers.

### Syntax Rules and Structure

**Installation:**

```bash
# For web projects
npm install react-router-dom

# For React Native projects
npm install react-router-native
```

**Basic Setup:**

```jsx
import { BrowserRouter, Routes, Route } from 'react-router-dom';

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
      </Routes>
    </BrowserRouter>
  );
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `BrowserRouter` | Root router component for web browsers using the HTML5 history API. |
| `Routes` | Wraps `<Route>` elements and selects the best match for the current URL. |
| `Route` | Couples a URL path to a React element. |

**Syntax Rules:**

1. The router (`BrowserRouter`, `HashRouter`, etc.) must wrap the entire application or the portion that uses routing.
2. `Routes` must be a descendant of a router.
3. `Route` elements must be children of `Routes` (or nested inside other `Route` elements).
4. Import from `react-router-dom` (web) or `react-router-native` (native), never directly from `react-router`.

**Constraints and Limitations:**

- React Router does not provide a "redirect" component that works outside the router context; `Navigate` must be rendered inside a router.
- Data APIs (loaders, actions) are only available in data routers (`createBrowserRouter`, `createHashRouter`, `createMemoryRouter`).

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Minimal React Router Setup

```jsx
// Step 1: Install react-router-dom
// npm install react-router-dom

// Step 2: Import required components
import React from 'react';
import ReactDOM from 'react-dom/client';
import { BrowserRouter, Routes, Route } from 'react-router-dom';

// Step 3: Define page components
function Home() {
  return <h1>Home Page</h1>;
}

function About() {
  return <h1>About Page</h1>;
}

// Step 4: Configure routes
function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
      </Routes>
    </BrowserRouter>
  );
}

// Step 5: Render the app
const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<App />);
```

**Expected Output (at `/`):**

```html
<h1>Home Page</h1>
```

**Expected Output (at `/about`):**

```html
<h1>About Page</h1>
```

**Why this output:** `BrowserRouter` provides routing context. `Routes` inspects the current URL and renders the `<Route>` whose `path` matches. At `/`, the `Home` component renders; at `/about`, the `About` component renders.

### Real-World Cases

- **Marketing websites:** Multi-page marketing sites built as SPAs with React Router.
- **SaaS dashboards:** Authenticated dashboards with nested routes for settings, analytics, and user management.
- **E-commerce storefronts:** Product listing pages, product detail pages, cart, and checkout flows.

### References

- React Router — Official Documentation — https://reactrouter.com/
- React Router — API Reference (GitHub) — https://github.com/remix-run/react-router/blob/main/docs/api.md

---

## 2. BrowserRouter

### Definitions

**Core Definition:** `BrowserRouter` is a router component that uses the HTML5 History API (`pushState`, `replaceState`) to keep the UI in sync with the browser URL.

**Technical Definition:** `BrowserRouter` stores the current location in `window.history.state` and listens to `popstate` events. It enables clean URLs (e.g., `/about`) without hash fragments. It requires server configuration to serve `index.html` for all routes to avoid 404 errors on refresh.

**Beginner-Friendly Explanation:** `BrowserRouter` makes your URLs look like normal website URLs (`/about`, `/users/123`) instead of having `#` in them. It is the most common router for web apps.

### Purposes

- To provide clean, SEO-friendly URLs without hash fragments.
- To integrate with the HTML5 History API for back/forward navigation.
- To enable server-side rendering compatibility (when paired with `StaticRouter` on the server).

### Syntax Rules and Structure

```jsx
import { BrowserRouter } from 'react-router-dom';

<BrowserRouter>
  <App />
</BrowserRouter>
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `BrowserRouter` | Wraps the app and provides routing context using the history API. |

**Syntax Rules:**

1. Must wrap the component tree that uses routing.
2. Requires server configuration: all requests should return `index.html` to avoid 404s on direct URL access.
3. In React Router v6.4+, prefer `createBrowserRouter` for data APIs.

**Constraints and Limitations:**

- Does not work with static file servers without server-side rewrite rules.
- Not suitable for testing; use `MemoryRouter` instead.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: BrowserRouter with Server Configuration

```jsx
import { BrowserRouter, Routes, Route, Link } from 'react-router-dom';

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

**Server configuration (Express example):**

```js
app.get('*', (req, res) => {
  res.sendFile(path.join(__dirname, 'build', 'index.html'));
});
```

**Expected Output:** Navigating to `/about` displays the `About` component with the URL `http://localhost:3000/about`.

**Why this output:** `BrowserRouter` uses `history.pushState` to change the URL without a page reload. The server must return `index.html` for any path so the app can boot and React Router can render the correct route.

### Real-World Cases

- **Public-facing websites:** Blogs, portfolios, and marketing sites where clean URLs matter for SEO.
- **SaaS applications:** Dashboards with shareable, bookmarkable URLs.
- **Progressive Web Apps (PWAs):** Apps that need to work offline and use clean URLs.

### References

- React Router — BrowserRouter — https://reactrouter.com/6.30.3/router-components/browser-router
- React Router — Picking a Router — https://reactrouter.com/6.30.3/routers/picking-a-router

---

## 3. HashRouter

### Definitions

**Core Definition:** `HashRouter` is a router component that uses the hash portion of the URL (`window.location.hash`) to manage routing state.

**Technical Definition:** `HashRouter` stores the current location in the URL fragment (e.g., `/#/about`). It listens to `hashchange` events. Because the hash is not sent to the server, `HashRouter` works without any server-side configuration.

**Beginner-Friendly Explanation:** `HashRouter` puts a `#` in your URL (like `/#/about`). It works even on simple static servers because the server only sees the part before the `#`.

### Purposes

- To enable routing on static file servers without server rewrite rules.
- To support legacy browsers that do not fully support the History API.
- To embed a React Router app in an environment where full URLs cannot be controlled.

### Syntax Rules and Structure

```jsx
import { HashRouter } from 'react-router-dom';

<HashRouter>
  <App />
</HashRouter>
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `HashRouter` | Wraps the app and provides routing context using the URL hash. |

**Syntax Rules:**

1. Must wrap the component tree that uses routing.
2. URLs appear as `example.com/#/about`.
3. No server configuration is required.

**Constraints and Limitations:**

- Hash URLs are less SEO-friendly than clean URLs.
- Hash URLs are considered less modern and may look unusual to users.
- Server-side rendering with hash routing is limited.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: HashRouter on a Static Server

```jsx
import { HashRouter, Routes, Route, Link } from 'react-router-dom';

function App() {
  return (
    <HashRouter>
      <nav>
        <Link to="/">Home</Link>
        <Link to="/about">About</Link>
      </nav>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
      </Routes>
    </HashRouter>
  );
}
```

**Expected Output:** Navigating to "About" changes the URL to `example.com/#/about` and renders the `About` component.

**Why this output:** `HashRouter` changes `window.location.hash`. The browser does not send the hash to the server, so no server configuration is needed.

### Real-World Cases

- **GitHub Pages deployments:** Static hosting environments where server rewrites are not possible.
- **Electron apps:** Desktop applications built with React that load local HTML files.
- **Quick prototypes:** Demos hosted on simple static servers.

### References

- React Router — HashRouter — https://reactrouter.com/6.30.3/router-components/hash-router
- React Router — Picking a Router — https://reactrouter.com/6.30.3/routers/picking-a-router

---

## 4. MemoryRouter

### Definitions

**Core Definition:** `MemoryRouter` is a router component that stores location history in memory (an array) rather than in the browser URL.

**Technical Definition:** `MemoryRouter` does not interact with `window.history` or `window.location`. It is primarily used in testing environments and non-browser environments where a DOM history API is unavailable. It can also serve as a reference implementation for other routers.

**Beginner-Friendly Explanation:** `MemoryRouter` keeps track of navigation in memory, not in the browser's address bar. It is mainly used for testing components that use routing.

### Purposes

- To test components that use React Router APIs without requiring a browser environment.
- To provide routing in non-browser environments (e.g., React Native tests, server-side rendering tests).
- To serve as a reference implementation for understanding router internals.

### Syntax Rules and Structure

```jsx
import { MemoryRouter } from 'react-router-dom';

<MemoryRouter initialEntries={['/']} initialIndex={0}>
  <App />
</MemoryRouter>
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `MemoryRouter` | Provides routing context backed by an in-memory history array. |
| `initialEntries` | Optional array of initial locations (default: `['/']`). |
| `initialIndex` | Optional index of the initial location in the entries array. |

**Syntax Rules:**

1. Does not change the browser URL.
2. Accepts `initialEntries` to simulate a navigation history.
3. Must wrap the component tree that uses routing.

**Constraints and Limitations:**

- Not suitable for production web applications (URL does not change).
- Cannot be used for SEO or deep linking.
- Primarily a testing and reference tool.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Testing a Component with MemoryRouter

```jsx
import { render, screen } from '@testing-library/react';
import { MemoryRouter, Routes, Route } from 'react-router-dom';
import UserProfile from './UserProfile';

test('renders user profile for route param', () => {
  render(
    <MemoryRouter initialEntries={['/users/42']}>
      <Routes>
        <Route path="/users/:id" element={<UserProfile />} />
      </Routes>
    </MemoryRouter>
  );

  expect(screen.getByText('User ID: 42')).toBeInTheDocument();
});
```

**Expected Output:** The test passes, and `UserProfile` renders with `id` equal to `42`.

**Why this output:** `MemoryRouter` initializes its in-memory history with `/users/42`. `Routes` matches the `:id` param and passes `42` to `UserProfile` via `useParams`.

### Real-World Cases

- **Unit and integration tests:** Testing components that use `useNavigate`, `useParams`, or `<Link>`.
- **Storybook stories:** Rendering routed components in isolation.
- **Server-side rendering tests:** Simulating navigation on the server.

### References

- React Router — MemoryRouter — https://reactrouter.com/6.30.3/router-components/memory-router
- React Router — Picking a Router (Testing) — https://reactrouter.com/6.30.3/routers/picking-a-router

---

## 5. Routes

### Definitions

**Core Definition:** `Routes` is a component that wraps `<Route>` elements and renders the one that best matches the current URL.

**Technical Definition:** `Routes` replaces the `Switch` component from React Router v5. It uses a ranking algorithm to select the best matching route from its children. Only the matched route's element is rendered. `Routes` must be a descendant of a router component.

**Beginner-Friendly Explanation:** `Routes` is like a switchboard. You give it a list of possible routes, and it picks the one that matches the current URL and shows that component.

### Purposes

- To group and organize `<Route>` definitions.
- To ensure only one route (the best match) renders at a time.
- To enable nested routing when `<Route>` elements are nested inside each other.

### Syntax Rules and Structure

```jsx
import { Routes, Route } from 'react-router-dom';

<Routes>
  <Route path="/" element={<Home />} />
  <Route path="/about" element={<About />} />
</Routes>
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Routes` | The container that selects the best matching route. |
| `Route` | A single route definition with a `path` and `element`. |

**Syntax Rules:**

1. `Routes` must be inside a router component.
2. `Route` elements must be direct children of `Routes` (or nested inside other `Route` elements).
3. Only the best-matching route renders.
4. `Routes` can be nested to create nested layouts.

**Constraints and Limitations:**

- `Routes` does not render anything if no route matches (unless a `*` catch-all route is defined).
- Path matching is case-insensitive by default but can be made case-sensitive with `caseSensitive` on `Route`.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Basic Routes Configuration

```jsx
import { BrowserRouter, Routes, Route } from 'react-router-dom';

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/products" element={<Products />} />
        <Route path="/products/:id" element={<ProductDetail />} />
        <Route path="*" element={<NotFound />} />
      </Routes>
    </BrowserRouter>
  );
}
```

**Expected Output:**

- `/` → `Home`
- `/products` → `Products`
- `/products/123` → `ProductDetail` (with `id` param `123`)
- `/unknown` → `NotFound`

**Why this output:** `Routes` ranks the routes and selects the best match. The `*` route acts as a catch-all for unmatched URLs.

### Real-World Cases

- **Application shell routing:** Defining top-level routes for major sections of an app.
- **Catch-all 404 pages:** Using `path="*"` to render a "Not Found" page.
- **Dynamic route parameters:** Routes like `/users/:id` that render a component based on the URL parameter.

### References

- React Router — Routes — https://reactrouter.com/6.30.3/components/routes
- React Router — Route — https://reactrouter.com/6.30.3/route/route

---

## 6. Route

### Definitions

**Core Definition:** `Route` is the fundamental building block of React Router that couples a URL path to a React element (and optionally data APIs).

**Technical Definition:** A `Route` object (or JSX element) specifies a `path` pattern, an `element` to render, and optional data APIs (`loader`, `action`, `errorElement`, `lazy`). Routes can be nested to create hierarchical layouts and data dependencies.

**Beginner-Friendly Explanation:** A `Route` tells React Router: "When the URL matches this path, show this component." You can also tell it how to load data before showing the component.

### Purposes

- To map URL paths to React components.
- To define dynamic segments (`:id`) for parameterized routes.
- To attach data loading (`loader`) and mutation (`action`) logic to a route.
- To define error boundaries (`errorElement`) for route-level error handling.
- To enable lazy loading of route components.

### Syntax Rules and Structure

**JSX Syntax:**

```jsx
<Route
  path="/teams/:teamId"
  element={<Team />}
  loader={async ({ params }) => fetch(`/api/teams/${params.teamId}`)}
  action={async ({ request }) => updateTeam(await request.formData())}
  errorElement={<ErrorBoundary />}
/>
```

**Route Object Syntax:**

```js
{
  path: "/teams/:teamId",
  element: <Team />,
  loader: async ({ params }) => fetch(`/api/teams/${params.teamId}`),
  action: async ({ request }) => updateTeam(await request.formData()),
  errorElement: <ErrorBoundary />,
}
```

**Component Breakdown:**

| Property | Description |
|----------|-------------|
| `path` | The URL pattern to match (supports dynamic segments `:param`). |
| `element` | The React element to render when the route matches. |
| `Component` | Alternative to `element` for data routers; React Router calls `createElement` internally. |
| `loader` | A function that fetches data before the route renders. |
| `action` | A function that handles form submissions and mutations. |
| `errorElement` | A React element rendered when the loader/action throws an error. |
| `lazy` | An async function that resolves non-route-matching portions of the route (for code splitting). |

**Syntax Rules:**

1. Dynamic segments start with `:` and are provided as `params` to loaders, actions, and `useParams`.
2. Multiple dynamic segments are allowed: `/c/:categoryId/p/:productId`.
3. Dynamic segments cannot be partial: `/teams-:teamId` is invalid; use `/teams/:teamId`.
4. Routes can be nested via `children`.
5. `index` routes act as default child routes without adding a URL segment.

**Constraints and Limitations:**

- Loaders and actions only work in data routers (`createBrowserRouter`, etc.).
- `element` and `Component` should not be used simultaneously on the same route.
- Route paths are relative to their parent route when nested.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Route with Dynamic Params and Loader

```jsx
import { createBrowserRouter, RouterProvider, useParams } from 'react-router-dom';

async function teamLoader({ params }) {
  const res = await fetch(`/api/teams/${params.teamId}`);
  return res.json();
}

function Team() {
  const { teamId } = useParams();
  return <h1>Team: {teamId}</h1>;
}

const router = createBrowserRouter([
  {
    path: '/teams/:teamId',
    element: <Team />,
    loader: teamLoader,
  },
]);

function App() {
  return <RouterProvider router={router} />;
}
```

**Expected Output (at `/teams/hotspur`):**

```html
<h1>Team: hotspur</h1>
```

**Why this output:** The route matches `/teams/hotspur`, the `loader` fetches data (though not used in this simplified example), and `useParams` extracts `teamId` as `"hotspur"`.

### Real-World Cases

- **User profiles:** `/users/:userId` routes that load user data via a loader.
- **Product pages:** `/products/:productId` routes with product details and reviews.
- **Edit forms:** Routes with an `action` that handles form submission for updating a resource.

### References

- React Router — Route — https://reactrouter.com/6.30.3/route/route
- React Router — Route Object — https://reactrouter.com/6.30.3/route/route#type-declaration

---

## 7. Link

### Definitions

**Core Definition:** `Link` is a component that renders an accessible `<a>` element for declarative navigation.

**Technical Definition:** `Link` renders an `<a>` tag with an `href` that points to the target route. Clicking it triggers client-side navigation via React Router's history API instead of a full page reload. It provides accessibility features like keyboard navigation and right-click context menu support.

**Beginner-Friendly Explanation:** `Link` is like an HTML `<a>` tag, but it navigates within your React app without reloading the page.

### Purposes

- To provide declarative navigation between routes.
- To ensure accessibility (keyboard focus, screen reader announcements).
- To support "open in new tab" and right-click context menus (unlike `useNavigate`).
- To enable relative and absolute path navigation.

### Syntax Rules and Structure

```jsx
import { Link } from 'react-router-dom';

<Link to="/about">About</Link>
<Link to={`/users/${userId}`}>User Profile</Link>
```

**Component Breakdown:**

| Prop | Description |
|------|-------------|
| `to` | The target path (string or object). |
| `replace` | If `true`, replaces the current history entry instead of pushing. |
| `state` | Optional state to pass to the new location. |
| `relative` | Controls relative path resolution (`"route"` or `"path"`). |

**Syntax Rules:**

1. `to` can be a string path or a location object (`{ pathname, search, hash }`).
2. Relative paths resolve relative to the current route's path.
3. `Link` renders an `<a>` element with the correct `href`.

**Constraints and Limitations:**

- `Link` is for declarative navigation; use `useNavigate` for programmatic navigation in event handlers or effects.
- External links should use a regular `<a>` tag, not `Link`.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Basic and Dynamic Links

```jsx
import { BrowserRouter, Routes, Route, Link } from 'react-router-dom';

function UserList({ users }) {
  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>
          <Link to={`/users/${user.id}`}>{user.name}</Link>
        </li>
      ))}
    </ul>
  );
}

function App() {
  return (
    <BrowserRouter>
      <nav>
        <Link to="/">Home</Link>
        <Link to="/users">Users</Link>
      </nav>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/users" element={<UserList users={users} />} />
        <Route path="/users/:id" element={<UserDetail />} />
      </Routes>
    </BrowserRouter>
  );
}
```

**Expected Output:** Clicking a user's name navigates to `/users/123` without a page reload and renders `UserDetail`.

**Why this output:** `Link` intercepts the click and uses `history.pushState` to change the URL. React Router re-renders the matching route.

### Real-World Cases

- **Navigation menus:** Header and sidebar navigation with links to different sections.
- **List item links:** Rendering a list of articles or products where each item links to its detail page.
- **Breadcrumb navigation:** Links to parent pages in a hierarchical structure.

### References

- React Router — Link — https://reactrouter.com/6.30.3/components/link
- React Router — Navigating — https://reactrouter.com/6.30.3/navigating

---

## 8. NavLink

### Definitions

**Core Definition:** `NavLink` is a special version of `Link` that automatically applies an "active" class or style when the current URL matches its `to` prop.

**Technical Definition:** `NavLink` extends `Link` and uses `useMatch` internally to determine if it is active. It accepts `className`, `style`, and `children` as functions that receive an `isActive` boolean, enabling dynamic styling of active navigation items.

**Beginner-Friendly Explanation:** `NavLink` is a `Link` that knows when it is "active" (i.e., when the user is on that link's page). You can style it differently to show the user where they are.

### Purposes

- To visually indicate the active navigation item.
- To provide accessible active state to screen readers (via `aria-current`).
- To simplify the implementation of navigation menus with active highlighting.

### Syntax Rules and Structure

```jsx
import { NavLink } from 'react-router-dom';

<NavLink
  to="/about"
  className={({ isActive }) => isActive ? 'active' : ''}
>
  About
</NavLink>
```

**Component Breakdown:**

| Prop | Description |
|------|-------------|
| `to` | Target path (same as `Link`). |
| `className` | Can be a string or a function `({ isActive }) => string`. |
| `style` | Can be an object or a function `({ isActive }) => object`. |
| `children` | Can be a function `({ isActive }) => ReactNode`. |
| `end` | If `true`, only matches when the URL ends exactly at `to`. |

**Syntax Rules:**

1. `className` can be a function that receives `{ isActive, isPending, isTransitioning }`.
2. By default, `NavLink` is active for descendant routes; use `end` to match only the exact path.
3. Active links automatically receive `aria-current="page"`.

**Constraints and Limitations:**

- `NavLink` is slightly heavier than `Link` because it computes active state.
- `end` prop is required for root routes (`to="/"`) to avoid always-active behavior.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Navigation with Active Styling

```jsx
import { NavLink, Routes, Route } from 'react-router-dom';

function NavBar() {
  return (
    <nav>
      <NavLink
        to="/"
        end
        className={({ isActive }) => isActive ? 'nav-active' : 'nav-item'}
      >
        Home
      </NavLink>
      <NavLink
        to="/about"
        className={({ isActive }) => isActive ? 'nav-active' : 'nav-item'}
      >
        About
      </NavLink>
      <NavLink
        to="/contact"
        className={({ isActive }) => isActive ? 'nav-active' : 'nav-item'}
      >
        Contact
      </NavLink>
    </nav>
  );
}
```

**Expected Output (when on `/about`):**

```html
<nav>
  <a href="/" class="nav-item">Home</a>
  <a href="/about" class="nav-active" aria-current="page">About</a>
  <a href="/contact" class="nav-item">Contact</a>
</nav>
```

**Why this output:** `NavLink` detects that `/about` is active and applies the `nav-active` class and `aria-current="page"`. The `end` prop on the home link prevents it from being active on every route.

### Real-World Cases

- **Main navigation menus:** Highlighting the current section in a header navigation.
- **Tab interfaces:** Tabs that use routing and need active styling.
- **Sidebar navigation:** Highlighting the selected item in a settings or dashboard sidebar.

### References

- React Router — NavLink — https://reactrouter.com/6.30.3/components/nav-link
- React Router — Navigating (NavLink) — https://reactrouter.com/6.30.3/navigating

---

## 9. Navigate

### Definitions

**Core Definition:** `Navigate` is a component that, when rendered, triggers a navigation to a new location.

**Technical Definition:** `Navigate` is a declarative wrapper around `useNavigate`. Rendering `<Navigate to="/login" />` causes the router to navigate to `/login` immediately. It is commonly used for redirects (e.g., after login, or when authentication is required). It renders nothing to the DOM.

**Beginner-Friendly Explanation:** `Navigate` is a component that automatically redirects the user to another page when it renders. It is useful for things like sending users to the login page if they are not authenticated.

### Purposes

- To perform redirects based on conditions (e.g., authentication status).
- To navigate after a form submission or action completes.
- To replace the current history entry (`replace` prop) instead of adding a new one.

### Syntax Rules and Structure

```jsx
import { Navigate } from 'react-router-dom';

<Navigate to="/login" replace />
<Navigate to="/dashboard" state={{ from: location }} />
```

**Component Breakdown:**

| Prop | Description |
|------|-------------|
| `to` | The target path or location object. |
| `replace` | If `true`, replaces the current history entry. |
| `state` | Optional state to pass to the new location. |

**Syntax Rules:**

1. `Navigate` must be rendered inside a router context.
2. It triggers navigation as a side effect of rendering.
3. Use `replace` to avoid adding the redirecting page to the history stack.

**Constraints and Limitations:**

- `Navigate` should not be used in the render path of a component that also renders other content, as it will immediately redirect.
- For programmatic navigation in event handlers, prefer `useNavigate`.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Authentication Redirect

```jsx
import { Navigate, useLocation } from 'react-router-dom';

function ProtectedRoute({ isAuthenticated, children }) {
  const location = useLocation();

  if (!isAuthenticated) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }

  return children;
}

// Usage
<Route
  path="/dashboard"
  element={
    <ProtectedRoute isAuthenticated={user !== null}>
      <Dashboard />
    </ProtectedRoute>
  }
/>
```

**Expected Output:** If the user is not authenticated and tries to visit `/dashboard`, they are redirected to `/login`. The `from` state preserves the intended destination.

**Why this output:** `Navigate` renders and immediately changes the location. The `replace` prop prevents the `/dashboard` URL from being saved in the history stack.

### Real-World Cases

- **Authentication guards:** Redirecting unauthenticated users to a login page.
- **Post-login redirects:** Sending users to their intended destination after login.
- **Onboarding flows:** Redirecting users who have not completed onboarding to the onboarding page.

### References

- React Router — Navigate — https://reactrouter.com/6.30.3/components/navigate
- React Router — Navigating (Navigate) — https://reactrouter.com/6.30.3/navigating

---

## 10. Outlet

### Definitions

**Core Definition:** `Outlet` is a component that renders the child route element of a parent route.

**Technical Definition:** `Outlet` acts as a placeholder inside a parent route's component. When routes are nested, the parent route renders its own layout and includes `<Outlet />` where the child route's element should appear. It is essential for nested routing and shared layouts.

**Beginner-Friendly Explanation:** `Outlet` is a "hole" in a parent component where child route components are rendered. It lets you create a shared layout (like a header and sidebar) while the main content changes based on the child route.

### Purposes

- To render child routes inside a parent layout.
- To create shared layouts (navigation bars, sidebars, footers) that persist across child routes.
- To enable nested routing with persistent UI elements.

### Syntax Rules and Structure

```jsx
import { Outlet } from 'react-router-dom';

function Layout() {
  return (
    <div>
      <Header />
      <Sidebar />
      <main>
        <Outlet />  {/* Child routes render here */}
      </main>
      <Footer />
    </div>
  );
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Outlet` | Renders the matched child route's element. |

**Syntax Rules:**

1. `Outlet` must be used inside a parent route's component that has child routes.
2. It renders the next match in the route hierarchy.
3. `Outlet` can receive a `context` prop to pass data to child routes.

**Constraints and Limitations:**

- If a parent route has no child routes, `Outlet` renders nothing.
- `Outlet` does not render index routes unless the parent route has an `index` child.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Shared Dashboard Layout

```jsx
import { Routes, Route, Outlet, Link } from 'react-router-dom';

function DashboardLayout() {
  return (
    <div>
      <nav>
        <Link to="/dashboard">Overview</Link>
        <Link to="/dashboard/settings">Settings</Link>
      </nav>
      <main>
        <Outlet />  {/* Overview or Settings renders here */}
      </main>
    </div>
  );
}

function App() {
  return (
    <Routes>
      <Route path="/dashboard" element={<DashboardLayout />}>
        <Route index element={<Overview />} />
        <Route path="settings" element={<Settings />} />
      </Route>
    </Routes>
  );
}
```

**Expected Output (at `/dashboard/settings`):**

```html
<div>
  <nav>
    <a href="/dashboard">Overview</a>
    <a href="/dashboard/settings">Settings</a>
  </nav>
  <main>
    <h1>Settings Page</h1>
  </main>
</div>
```

**Why this output:** The parent route renders `DashboardLayout` (with nav and `<Outlet />`). The child route `settings` matches and its element (`Settings`) renders inside the outlet.

### Real-World Cases

- **Application shells:** Headers, sidebars, and footers that persist across all pages.
- **Settings pages:** A settings layout with a sidebar of settings categories, where the right panel changes based on the selected child route.
- **Multi-step forms:** A layout with a progress indicator that stays visible while each step's form renders in the outlet.

### References

- React Router — Outlet — https://reactrouter.com/6.30.3/components/outlet
- React Router — Nested Routes — https://reactrouter.com/6.30.3/start/concepts#nested-routes

---

## 11. useNavigate

### Definitions

**Core Definition:** `useNavigate` is a hook that returns a function for programmatic navigation.

**Technical Definition:** `useNavigate` returns a `navigate` function that accepts a path string, a `To` object, or a numeric delta (for history traversal). It supports options like `replace`, `state`, `preventScrollReset`, and `viewTransition`. In data mode, it returns a `Promise<void>` that resolves when navigation completes.

**Beginner-Friendly Explanation:** `useNavigate` gives you a function you can call to change the page programmatically — for example, after a form submission or a button click.

### Purposes

- To navigate programmatically in event handlers and effects.
- To navigate after asynchronous operations (e.g., API calls) complete.
- To implement "go back" functionality using numeric deltas.
- To navigate with `replace` to avoid polluting the history stack.

### Syntax Rules and Structure

```jsx
import { useNavigate } from 'react-router-dom';

function MyComponent() {
  const navigate = useNavigate();

  function handleClick() {
    navigate('/dashboard');
    // or navigate(-1) to go back
    // or navigate('/login', { replace: true })
  }

  return <button onClick={handleClick}>Go to Dashboard</button>;
}
```

**Component Breakdown:**

| Parameter | Description |
|-----------|-------------|
| `to` | Path string or `To` object. |
| `options.replace` | Replace the current history entry. |
| `options.state` | State to pass to the new location. |
| `options.preventScrollReset` | Prevent scroll restoration. |
| `options.viewTransition` | Enable view transitions. |

**Syntax Rules:**

1. Must be called inside a component rendered within a router.
2. The returned `navigate` function is stable across renders.
3. Numeric arguments navigate through history: `navigate(-1)` goes back, `navigate(1)` goes forward.

**Constraints and Limitations:**

- `useNavigate` cannot be used outside a router context.
- For normal user-initiated navigation, prefer `Link` or `NavLink` for better accessibility and UX.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Post-Login Navigation

```jsx
import { useNavigate } from 'react-router-dom';
import { useState } from 'react';

function LoginForm() {
  const navigate = useNavigate();
  const [username, setUsername] = useState('');

  async function handleSubmit(e) {
    e.preventDefault();
    const success = await login(username);
    if (success) {
      navigate('/dashboard', { replace: true });
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={username}
        onChange={(e) => setUsername(e.target.value)}
        placeholder="Username"
      />
      <button type="submit">Log In</button>
    </form>
  );
}
```

**Expected Output:** After a successful login, the user is redirected to `/dashboard`. The `/login` page is replaced in the history stack.

**Why this output:** `navigate('/dashboard', { replace: true })` changes the URL and replaces the current history entry, so pressing the back button does not return to the login page.

### Real-World Cases

- **Form submissions:** Navigating to a success page after a form is submitted.
- **Logout:** Navigating to the home page after logging out.
- **Timed redirects:** Navigating after a countdown or timeout.

### References

- React Router — useNavigate — https://reactrouter.com/6.30.3/hooks/use-navigate
- React Router — Navigating (useNavigate) — https://reactrouter.com/6.30.3/navigating

---

## 12. useParams

### Definitions

**Core Definition:** `useParams` is a hook that returns an object of key/value pairs of the dynamic segments from the current URL.

**Technical Definition:** `useParams` reads the matched route's dynamic segments (e.g., `:id`, `:categoryId`) and returns them as a string-keyed object. The keys correspond to the segment names defined in the route path.

**Beginner-Friendly Explanation:** `useParams` lets you read the values from the URL. If your route is `/users/:id` and the URL is `/users/42`, `useParams()` returns `{ id: '42' }`.

### Purposes

- To access URL parameters in a component.
- To fetch data based on the current URL (e.g., user ID, product ID).
- To render content dynamically based on the URL.

### Syntax Rules and Structure

```jsx
import { useParams } from 'react-router-dom';

function UserProfile() {
  const { userId } = useParams();
  return <h1>User ID: {userId}</h1>;
}
```

**Component Breakdown:**

| Return Value | Description |
|--------------|-------------|
| Object | Key/value pairs of dynamic segments from the URL. |

**Syntax Rules:**

1. Must be used inside a component rendered by a route with dynamic segments.
2. The returned object is empty if the route has no dynamic segments.
3. Keys are the segment names without the `:` prefix.

**Constraints and Limitations:**

- `useParams` returns strings, even if the segment looks like a number.
- If the route has no params, it returns an empty object.
- Params are not available in parent routes unless they also define the same dynamic segments.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Accessing Multiple Params

```jsx
import { useParams } from 'react-router-dom';

function ProductDetail() {
  const { categoryId, productId } = useParams();

  return (
    <div>
      <p>Category: {categoryId}</p>
      <p>Product: {productId}</p>
    </div>
  );
}

// Route: <Route path="/c/:categoryId/p/:productId" element={<ProductDetail />} />
// URL: /c/electronics/p/123
// Output: Category: electronics, Product: 123
```

**Expected Output:**

```html
<div>
  <p>Category: electronics</p>
  <p>Product: 123</p>
</div>
```

**Why this output:** The route path defines two dynamic segments, `:categoryId` and `:productId`. `useParams` returns both as strings.

### Real-World Cases

- **User profiles:** Extracting `userId` from `/users/:userId`.
- **E-commerce:** Extracting `productId` and `categoryId` from product URLs.
- **Blog posts:** Extracting `slug` from `/blog/:slug`.

### References

- React Router — useParams — https://reactrouter.com/6.30.3/hooks/use-params

---

## 13. useSearchParams

### Definitions

**Core Definition:** `useSearchParams` is a hook that provides read and write access to the query string of the current URL.

**Technical Definition:** `useSearchParams` returns a tuple of `[searchParams, setSearchParams]`, where `searchParams` is a `URLSearchParams` object and `setSearchParams` is a function that updates the query string. It is the hook equivalent of `URLSearchParams` for React Router.

**Beginner-Friendly Explanation:** `useSearchParams` lets you read and change the `?key=value` part of the URL. You can use it to implement filters, search, and pagination.

### Purposes

- To read query parameters (e.g., `?page=2&sort=name`).
- To update the query string without a full navigation.
- To implement search, filtering, and pagination.

### Syntax Rules and Structure

```jsx
import { useSearchParams } from 'react-router-dom';

function SearchPage() {
  const [searchParams, setSearchParams] = useSearchParams();
  const query = searchParams.get('q') || '';

  function handleSearch(e) {
    setSearchParams({ q: e.target.value });
  }

  return <input value={query} onChange={handleSearch} />;
}
```

**Component Breakdown:**

| Return Value | Description |
|--------------|-------------|
| `searchParams` | A `URLSearchParams` object. |
| `setSearchParams` | A function to update the query string. |

**Syntax Rules:**

1. `searchParams.get('key')` returns the value or `null`.
2. `searchParams.has('key')` checks if a key exists.
3. `setSearchParams(object)` replaces the entire query string; `setSearchParams(prev => ...)` can update based on previous params.
4. `setSearchParams` accepts an options object with `replace` and `state`.

**Constraints and Limitations:**

- When updating, you must include all existing params you want to keep; otherwise, they are removed.
- Query params are always strings.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Pagination with Search Params

```jsx
import { useSearchParams } from 'react-router-dom';

function PaginatedList() {
  const [searchParams, setSearchParams] = useSearchParams();
  const page = parseInt(searchParams.get('page') || '1', 10);

  function goToNextPage() {
    setSearchParams({ page: String(page + 1) });
  }

  function goToPrevPage() {
    setSearchParams({ page: String(Math.max(1, page - 1)) });
  }

  return (
    <div>
      <p>Current Page: {page}</p>
      <button onClick={goToPrevPage} disabled={page === 1}>
        Previous
      </button>
      <button onClick={goToNextPage}>Next</button>
    </div>
  );
}
```

**Expected Output (at `?page=2`):**

```html
<div>
  <p>Current Page: 2</p>
  <button>Previous</button>
  <button>Next</button>
</div>
```

**Why this output:** `searchParams.get('page')` reads `"2"`, which is parsed to the number `2`. Clicking "Next" calls `setSearchParams({ page: "3" })`, updating the URL to `?page=3`.

### Real-World Cases

- **Search results:** Reading `?q=react` and rendering search results.
- **Product filters:** Reading `?category=electronics&sort=price` to filter products.
- **Pagination:** Reading and updating `?page=N`.

### References

- React Router — useSearchParams — https://reactrouter.com/6.30.3/hooks/use-search-params

---

## 14. useLocation

### Definitions

**Core Definition:** `useLocation` is a hook that returns the current `location` object, representing the current URL.

**Technical Definition:** `useLocation` returns a `Location` object with `pathname`, `search`, `hash`, `state`, and `key` properties. It is useful for performing side effects whenever the URL changes, such as page view tracking or scroll restoration.

**Beginner-Friendly Explanation:** `useLocation` tells you the current URL and any state passed during navigation. It is useful for tracking page views or displaying breadcrumbs.

### Purposes

- To read the current URL's `pathname`, `search`, `hash`, and `state`.
- To perform side effects on URL changes (e.g., analytics page views).
- To display breadcrumbs or navigation context.

### Syntax Rules and Structure

```jsx
import { useLocation } from 'react-router-dom';

function PageTracker() {
  const location = useLocation();

  useEffect(() => {
    trackPageView(location.pathname);
  }, [location.pathname]);

  return null;
}
```

**Component Breakdown:**

| Property | Description |
|----------|-------------|
| `pathname` | The URL path (e.g., `/users/123`). |
| `search` | The query string (e.g., `?page=2`). |
| `hash` | The hash fragment (e.g., `#section`). |
| `state` | The state object passed during navigation. |
| `key` | A unique key for the location. |

**Syntax Rules:**

1. The `location` object is immutable and updates on every navigation.
2. `state` is `null` if no state was passed.
3. Must be used inside a router context.

**Constraints and Limitations:**

- The `location` object is read-only; use `useNavigate` or `setSearchParams` to change the URL.
- State is lost on page refresh (unless persisted).

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Breadcrumbs from Location

```jsx
import { useLocation, Link } from 'react-router-dom';

function Breadcrumbs() {
  const { pathname } = useLocation();
  const segments = pathname.split('/').filter(Boolean);

  return (
    <nav>
      <Link to="/">Home</Link>
      {segments.map((segment, index) => {
        const path = '/' + segments.slice(0, index + 1).join('/');
        return (
          <span key={path}>
            {' / '}
            <Link to={path}>{segment}</Link>
          </span>
        );
      })}
    </nav>
  );
}
```

**Expected Output (at `/users/123/posts`):**

```html
<nav>
  <a href="/">Home</a>
  <span> / <a href="/users">users</a></span>
  <span> / <a href="/users/123">123</a></span>
  <span> / <a href="/users/123/posts">posts</a></span>
</nav>
```

**Why this output:** `useLocation` provides the current pathname. Splitting it into segments and building cumulative paths creates a hierarchical breadcrumb trail.

### Real-World Cases

- **Analytics:** Sending page view events whenever the location changes.
- **Breadcrumbs:** Building navigation trails from the current path.
- **Scroll restoration:** Reading the location to decide where to scroll.

### References

- React Router — useLocation — https://reactrouter.com/6.30.3/hooks/use-location

---

## 15. useMatch

### Definitions

**Core Definition:** `useMatch` is a hook that returns match information if a given path pattern matches the current URL.

**Technical Definition:** `useMatch(pattern)` returns a `PathMatch` object (containing `params`, `pathname`, `pathnameBase`, and `pattern`) if the current location matches the given path pattern; otherwise, it returns `null`.

**Beginner-Friendly Explanation:** `useMatch` checks if the current URL matches a specific pattern. It is useful for components that need to know their own active state or match details outside of `<Route>`.

### Purposes

- To determine if a specific path pattern matches the current URL.
- To access match params outside of a `<Route>` element.
- To implement custom active-state logic in components.

### Syntax Rules and Structure

```jsx
import { useMatch } from 'react-router-dom';

const match = useMatch('/users/:id');
// match is null or { params: { id: '123' }, pathname: '/users/123', ... }
```

**Component Breakdown:**

| Return Value | Description |
|--------------|-------------|
| `PathMatch` or `null` | Match details if the pattern matches; otherwise `null`. |

**Syntax Rules:**

1. The pattern can be a string or a `PathPattern` object.
2. Returns `null` if no match.
3. The match object contains `params`, `pathname`, `pathnameBase`, and `pattern`.

**Constraints and Limitations:**

- `useMatch` is a low-level hook; most components should use `<Route>` or `NavLink` instead.
- Patterns are matched against the entire URL path.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Custom Active State

```jsx
import { useMatch, Link } from 'react-router-dom';

function CustomNavItem({ to, children }) {
  const match = useMatch(to);
  const isActive = match !== null;

  return (
    <Link
      to={to}
      style={{ fontWeight: isActive ? 'bold' : 'normal' }}
    >
      {children}
    </Link>
  );
}
```

**Expected Output:** When the current URL matches `to`, the link text is bold.

**Why this output:** `useMatch(to)` returns a match object when the URL matches, and `null` otherwise. The component uses this to apply conditional styling.

### Real-World Cases

- **Custom navigation components:** Building navigation items that are not `<NavLink>` but need active-state logic.
- **Conditional rendering based on URL:** Rendering a sidebar only on routes that match a certain pattern.

### References

- React Router — useMatch — https://reactrouter.com/6.30.3/hooks/use-match

---

## 16. useResolvedPath

### Definitions

**Core Definition:** `useResolvedPath` is a hook that resolves a given `to` path against the current location's pathname.

**Technical Definition:** `useResolvedPath(to)` returns a `Path` object (`{ pathname, search, hash }`) with the path resolved relative to the current route. It is useful for building custom links or determining the absolute path of a relative target.

**Beginner-Friendly Explanation:** `useResolvedPath` takes a relative path (like `"edit"`) and tells you the absolute path it points to based on where you are now.

### Purposes

- To resolve relative paths to absolute paths.
- To build custom navigation components that need the resolved URL.
- To compute the full path for comparison or display.

### Syntax Rules and Structure

```jsx
import { useResolvedPath } from 'react-router-dom';

function MyComponent() {
  const resolved = useResolvedPath('edit');
  // If current location is /users/123, resolved.pathname is /users/123/edit
  return <a href={resolved.pathname}>Edit</a>;
}
```

**Component Breakdown:**

| Return Value | Description |
|--------------|-------------|
| `Path` object | `{ pathname, search, hash }` with the resolved path. |

**Syntax Rules:**

1. Resolves relative to the current route's path.
2. Handles `..` and `.` segments.
3. Prepends the router's `basename` if configured.

**Constraints and Limitations:**

- `useResolvedPath` does not perform navigation; it only resolves the path.
- It is a utility hook and is less commonly used in application code.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Building a Custom Link

```jsx
import { useResolvedPath, Link } from 'react-router-dom';

function RelativeEditLink() {
  const resolved = useResolvedPath('edit');

  return (
    <Link to={resolved}>
      Edit this item
    </Link>
  );
}
```

**Expected Output:** If the current route is `/posts/5`, the link points to `/posts/5/edit`.

**Why this output:** `useResolvedPath('edit')` resolves `edit` relative to the current pathname `/posts/5`, producing `/posts/5/edit`.

### Real-World Cases

- **Custom link components:** Building wrapper components around `Link` that need to compute the resolved path.
- **Active state logic:** Comparing the resolved path with the current pathname for custom active detection.

### References

- React Router — useResolvedPath — https://reactrouter.com/6.30.3/hooks/use-resolved-path

---

## 17. useHref

### Definitions

**Core Definition:** `useHref` is a hook that returns the `href` string for a given `to` path, resolving relative paths and prepending the router's `basename`.

**Technical Definition:** `useHref(to)` returns a URL string suitable for use in an `<a href>`. It correctly handles parent route segments (`..`), sibling routes, trailing slashes, and the router's `basename` configuration.

**Beginner-Friendly Explanation:** `useHref` tells you the actual URL that a link will point to. It is useful when you need the `href` for an anchor tag but are not using `<Link>`.

### Purposes

- To get the resolved `href` for a target path.
- To build custom anchor tags that integrate with React Router navigation.
- To display the target URL in the UI.

### Syntax Rules and Structure

```jsx
import { useHref } from 'react-router-dom';

const href = useHref('/about');
// href is '/about' (or '/base/about' if basename is set)
```

**Component Breakdown:**

| Return Value | Description |
|--------------|-------------|
| `string` | The resolved URL string. |

**Syntax Rules:**

1. Resolves relative paths against the current location.
2. Prepends the router's `basename` if configured.
3. Handles `..` and `.` segments correctly.

**Constraints and Limitations:**

- `useHref` returns a string; it does not trigger navigation.
- For most navigation needs, `<Link>` or `useNavigate` is more appropriate.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Custom Anchor with useHref

```jsx
import { useHref, useLinkClickHandler } from 'react-router-dom';

function CustomLink({ to, children }) {
  const href = useHref(to);
  const handleClick = useLinkClickHandler(to);

  return (
    <a href={href} onClick={handleClick}>
      {children}
    </a>
  );
}
```

**Expected Output:** Renders an `<a>` tag with the correct `href` and client-side navigation on click.

**Why this output:** `useHref` provides the correct URL string, and `useLinkClickHandler` (another React Router hook) intercepts clicks for client-side navigation.

### Real-World Cases

- **Design system link components:** Building a custom `Link` component that uses `useHref` for the `href` and `useLinkClickHandler` for navigation.
- **Displaying URLs:** Showing the resolved URL in a preview or settings panel.

### References

- React Router — useHref — https://reactrouter.com/6.30.3/hooks/use-href

---

## 18. useNavigationType

### Definitions

**Core Definition:** `useNavigationType` is a hook that returns the type of navigation that led to the current location: `"POP"`, `"PUSH"`, or `"REPLACE"`.

**Technical Definition:** `useNavigationType` reads the `navigationType` from the router's history state. `"POP"` indicates a back/forward navigation; `"PUSH"` indicates a new entry was added; `"REPLACE"` indicates the current entry was replaced.

**Beginner-Friendly Explanation:** `useNavigationType` tells you how the user arrived at the current page — whether they clicked a link (PUSH), went back (POP), or were redirected (REPLACE).

### Purposes

- To distinguish between back/forward navigation and link clicks.
- To conditionally perform side effects based on navigation type (e.g., scroll restoration only on `POP`).
- To implement analytics that differentiate between navigation types.

### Syntax Rules and Structure

```jsx
import { useNavigationType } from 'react-router-dom';

function ScrollRestoration() {
  const navigationType = useNavigationType();

  useEffect(() => {
    if (navigationType === 'POP') {
      // Restore scroll position on back/forward
    }
  }, [navigationType]);
}
```

**Component Breakdown:**

| Return Value | Description |
|--------------|-------------|
| `"POP"` \| `"PUSH"` \| `"REPLACE"` | The navigation type. |

**Syntax Rules:**

1. Returns `"POP"` when the user clicks the browser's back or forward button.
2. Returns `"PUSH"` for normal link clicks and `navigate()` calls.
3. Returns `"REPLACE"` when `replace: true` is used.

**Constraints and Limitations:**

- The navigation type is read-only.
- It is primarily useful for side effects, not for rendering decisions.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Conditional Scroll Restoration

```jsx
import { useNavigationType } from 'react-router-dom';
import { useEffect } from 'react';

function ScrollToTop() {
  const navigationType = useNavigationType();

  useEffect(() => {
    if (navigationType === 'PUSH') {
      window.scrollTo(0, 0);
    }
    // Do not scroll to top on POP (back/forward)
  }, [navigationType]);

  return null;
}
```

**Expected Output:** On link clicks (`PUSH`), the page scrolls to the top. On back/forward (`POP`), the scroll position is preserved.

**Why this output:** The effect checks `navigationType` and only scrolls to top for new navigations, preserving the user's scroll position when they navigate back.

### Real-World Cases

- **Scroll restoration:** Deciding whether to restore scroll position based on navigation type.
- **Analytics:** Differentiating page views from link clicks versus back/forward navigation.
- **Animation control:** Triggering enter animations only on forward navigation.

### References

- React Router — useNavigationType — https://reactrouter.com/6.30.3/hooks/use-navigation-type

---

## 19. Route Configuration

### Definitions

**Core Definition:** Route configuration is the process of defining the mapping between URL paths and React components, including nested routes and data APIs.

**Technical Definition:** Route configuration can be expressed in two ways: JSX (`<Routes>` and `<Route>`) or JavaScript objects (passed to `createBrowserRouter`, `createHashRouter`, or `createMemoryRouter`). Route objects support properties like `path`, `element`, `loader`, `action`, `errorElement`, `lazy`, `children`, and `index`.

**Beginner-Friendly Explanation:** Route configuration is how you tell React Router which component to show for each URL. You can write it with JSX tags or with JavaScript objects.

### Purposes

- To define the complete routing tree of the application.
- To couple URL segments with components, data loading, and error handling.
- To enable nested layouts and data boundaries.
- To support lazy loading and code splitting.

### Syntax Rules and Structure

**JSX Configuration:**

```jsx
<Routes>
  <Route path="/" element={<Layout />}>
    <Route index element={<Home />} />
    <Route path="about" element={<About />} />
    <Route path="users/:id" element={<UserProfile />} loader={userLoader} />
  </Route>
</Routes>
```

**Object Configuration:**

```js
const router = createBrowserRouter([
  {
    path: '/',
    element: <Layout />,
    children: [
      { index: true, element: <Home /> },
      { path: 'about', element: <About /> },
      { path: 'users/:id', element: <UserProfile />, loader: userLoader },
    ],
  },
]);
```

**Component Breakdown:**

| Property | Description |
|----------|-------------|
| `path` | URL segment pattern. |
| `index` | Marks a default child route (no URL segment added). |
| `children` | Nested route objects. |
| `element` / `Component` | The React element/component to render. |
| `loader` | Data loading function. |
| `action` | Data mutation function. |
| `errorElement` | Error boundary element. |
| `lazy` | Async function for lazy route loading. |

**Syntax Rules:**

1. JSX and object styles are functionally identical; choose based on preference.
2. Index routes have no `path` and render when the parent path matches.
3. Nested routes inherit the parent's path unless they start with `/`.
4. Dynamic segments (`:param`) are defined in the `path`.

**Constraints and Limitations:**

- Data APIs (loaders, actions) only work with data routers (created with `createBrowserRouter`, etc.), not with `<BrowserRouter>` alone.
- Route objects should be defined outside of render to avoid re-creation.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Object-Based Configuration with Data APIs

```jsx
import { createBrowserRouter, RouterProvider } from 'react-router-dom';

async function userLoader({ params }) {
  const res = await fetch(`/api/users/${params.id}`);
  return res.json();
}

const router = createBrowserRouter([
  {
    path: '/',
    element: <Layout />,
    children: [
      { index: true, element: <Home /> },
      {
        path: 'users/:id',
        element: <UserProfile />,
        loader: userLoader,
        errorElement: <UserError />,
      },
    ],
  },
]);

function App() {
  return <RouterProvider router={router} />;
}
```

**Expected Output:** Navigating to `/users/42` loads user data via `userLoader` before rendering `UserProfile`. If the loader throws, `UserError` renders.

**Why this output:** The route object defines a `loader` that runs before the component renders. `RouterProvider` supplies the router context and handles data loading and error bubbling.

### Real-World Cases

- **Large applications:** Object-based configuration is easier to split across files and generate dynamically.
- **Data-driven apps:** Routes with loaders and actions for server-state management.
- **Framework integrations:** Remix and React Router framework mode use route object configurations internally.

### References

- React Router — Route — https://reactrouter.com/6.30.3/route/route
- React Router — createBrowserRouter — https://reactrouter.com/6.30.3/routers/create-browser-router

---

## 20. Route Nesting

### Definitions

**Core Definition:** Route nesting is the practice of placing `<Route>` elements inside other `<Route>` elements to create a hierarchy of routes and layouts.

**Technical Definition:** Nested routes allow parent routes to render a layout component with an `<Outlet />`, while child routes render their content inside that outlet. URL paths are also nested: a child route's path is appended to its parent's path.

**Beginner-Friendly Explanation:** Route nesting lets you build layouts that wrap child pages. The parent route shows a header and sidebar, and the child route's content appears in the middle.

### Purposes

- To create shared layouts that persist across multiple pages.
- To organize routes hierarchically based on the URL structure.
- To enable data boundaries where parent routes load data for all children.
- To keep route configurations modular and readable.

### Syntax Rules and Structure

```jsx
<Routes>
  <Route path="/dashboard" element={<DashboardLayout />}>
    <Route index element={<Overview />} />
    <Route path="settings" element={<Settings />} />
    <Route path="analytics" element={<Analytics />} />
  </Route>
</Routes>
```

**Component Breakdown:**

| Route | Path | Renders |
|-------|------|---------|
| Parent | `/dashboard` | `DashboardLayout` (with `<Outlet />`) |
| Index child | `/dashboard` | `Overview` (inside outlet) |
| Child | `/dashboard/settings` | `Settings` (inside outlet) |
| Child | `/dashboard/analytics` | `Analytics` (inside outlet) |

**Syntax Rules:**

1. Child route paths are relative to the parent path.
2. `index` routes render at the parent's path exactly.
3. The parent component must include `<Outlet />` for children to render.
4. Nested routes can be nested multiple levels deep.

**Constraints and Limitations:**

- A parent route without an `<Outlet />` will not render its children.
- Pathless routes (layout routes) can be used to group routes without adding a URL segment.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Three-Level Nesting

```jsx
function App() {
  return (
    <Routes>
      <Route path="/" element={<RootLayout />}>
        <Route index element={<Home />} />
        <Route path="dashboard" element={<DashboardLayout />}>
          <Route index element={<Overview />} />
          <Route path="settings" element={<Settings />} />
        </Route>
      </Route>
    </Routes>
  );
}

function RootLayout() {
  return (
    <div>
      <Header />
      <Outlet />  {/* DashboardLayout or Home renders here */}
    </div>
  );
}

function DashboardLayout() {
  return (
    <div>
      <Sidebar />
      <Outlet />  {/* Overview or Settings renders here */}
    </div>
  );
}
```

**Expected Output (at `/dashboard/settings`):**

```html
<div>
  <Header />
  <div>
    <Sidebar />
    <h1>Settings</h1>
  </div>
</div>
```

**Why this output:** `RootLayout` renders `Header` and an outlet. The `dashboard` child matches, rendering `DashboardLayout` (with `Sidebar` and its own outlet). The `settings` grandchild renders inside the innermost outlet.

### Real-World Cases

- **Admin dashboards:** A dashboard layout with sidebar navigation, where each section (users, settings, analytics) is a child route.
- **Settings pages:** A settings layout with tabs, where each tab is a nested route.
- **Multi-step wizards:** A wizard layout with a progress bar, where each step is a child route.

### References

- React Router — Nested Routes — https://reactrouter.com/6.30.3/start/concepts#nested-routes
- React Router — Outlet — https://reactrouter.com/6.30.3/components/outlet

---

## 21. Route Layouts

### Definitions

**Core Definition:** Route layouts are React components that provide shared UI (headers, sidebars, footers) across multiple child routes.

**Technical Definition:** A layout route is a route that renders a component containing an `<Outlet />`. The layout component persists across child route changes, preserving its state and avoiding re-mounting. Layout routes can be pathless (no URL segment) or have a path.

**Beginner-Friendly Explanation:** A layout is a component that wraps around page content — like a website header and footer that stay the same while the main content changes.

### Purposes

- To avoid re-rendering shared UI (header, sidebar) on every navigation.
- To provide a consistent structure across related pages.
- To group routes that share data or authentication requirements.
- To enable nested data loading boundaries.

### Syntax Rules and Structure

**Pathless Layout Route:**

```jsx
<Route element={<Layout />}>
  <Route path="/" element={<Home />} />
  <Route path="/about" element={<About />} />
</Route>
```

**Pathed Layout Route:**

```jsx
<Route path="/dashboard" element={<DashboardLayout />}>
  <Route index element={<Overview />} />
  <Route path="settings" element={<Settings />} />
</Route>
```

**Component Breakdown:**

| Type | Description |
|------|-------------|
| Pathless layout | Groups routes without adding a URL segment. |
| Pathed layout | Adds a URL segment and provides a layout for child routes. |

**Syntax Rules:**

1. The layout component must render `<Outlet />` to display child routes.
2. Layouts can be nested.
3. Pathless layouts are useful for grouping routes under a shared provider or error boundary.

**Constraints and Limitations:**

- Layout components re-render when the route changes, but they are not unmounted, so state is preserved.
- Layouts cannot conditionally render `<Outlet />` based on child route without additional logic.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Authentication Layout

```jsx
function AuthLayout() {
  return (
    <div className="auth-container">
      <Logo />
      <Outlet />  {/* Login or Register form renders here */}
    </div>
  );
}

function App() {
  return (
    <Routes>
      <Route element={<AuthLayout />}>
        <Route path="/login" element={<Login />} />
        <Route path="/register" element={<Register />} />
      </Route>
    </Routes>
  );
}
```

**Expected Output (at `/login`):**

```html
<div class="auth-container">
  <header>Logo</header>
  <form>Login form</form>
</div>
```

**Why this output:** The pathless `AuthLayout` route wraps both `/login` and `/register`. The layout renders the logo and an outlet, and the matched child route renders inside the outlet.

### Real-World Cases

- **Authentication layouts:** Centered card layouts for login and registration pages.
- **Dashboard layouts:** Sidebar navigation and top bar shared across all dashboard pages.
- **Marketing layouts:** Header and footer shared across marketing pages.

### References

- React Router — Layout Routes — https://reactrouter.com/6.30.3/route/route#layout-routes
- React Router — Outlet — https://reactrouter.com/6.30.3/components/outlet

---

## 22. Route Loaders

### Definitions

**Core Definition:** A route loader is a function that fetches data before a route's component renders.

**Technical Definition:** A `loader` is an async function defined on a route object. It receives `{ request, params, context }` and returns data (or a `Response`). React Router calls the loader before rendering the route and provides the data to the component via `useLoaderData()`. Loaders run in parallel for nested routes.

**Beginner-Friendly Explanation:** A loader loads data before showing a page. Instead of showing a loading spinner and then fetching data inside the component, the loader fetches everything first and then renders the page with data already available.

### Purposes

- To fetch data before rendering a route, eliminating loading spinners.
- To centralize data fetching logic at the route level.
- To enable parallel data loading for nested routes.
- To provide data to components via `useLoaderData()`.

### Syntax Rules and Structure

```jsx
async function userLoader({ params, request }) {
  const res = await fetch(`/api/users/${params.id}`, {
    signal: request.signal,
  });
  if (!res.ok) throw new Response('Not Found', { status: 404 });
  return res.json();
}

// Route definition
<Route path="/users/:id" element={<UserProfile />} loader={userLoader} />
```

**Component Breakdown:**

| Parameter | Description |
|-----------|-------------|
| `params` | Dynamic segments from the URL. |
| `request` | The Fetch API `Request` object (includes `signal` for cancellation). |
| `context` | Optional router context. |
| Return value | Any serializable data or a `Response`. |

**Syntax Rules:**

1. Loaders run before the route component renders.
2. Throwing a `Response` triggers the nearest `errorElement`.
3. Loaders run in parallel for nested routes.
4. Access data in the component via `useLoaderData()`.

**Constraints and Limitations:**

- Loaders only work in data routers (`createBrowserRouter`, etc.).
- Loaders should be defined outside the component render function to avoid re-creation.
- The `request.signal` should be used to cancel pending requests when the user navigates away.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Loader with Error Handling

```jsx
import { useLoaderData } from 'react-router-dom';

async function postLoader({ params }) {
  const res = await fetch(`https://jsonplaceholder.typicode.com/posts/${params.id}`);
  if (!res.ok) {
    throw new Response('Post Not Found', { status: 404 });
  }
  return res.json();
}

function PostDetail() {
  const post = useLoaderData();
  return (
    <article>
      <h1>{post.title}</h1>
      <p>{post.body}</p>
    </article>
  );
}

// Route: <Route path="/posts/:id" element={<PostDetail />} loader={postLoader} errorElement={<NotFound />} />
```

**Expected Output (at `/posts/1`):**

```html
<article>
  <h1>sunt aut facere repellat provident occaecati excepturi optio reprehenderit</h1>
  <p>quia et suscipit...</p>
</article>
```

**Why this output:** The loader fetches the post before rendering. If the fetch returns a 404, the loader throws a `Response`, and the `errorElement` renders instead.

### Real-World Cases

- **User profiles:** Loading user data before rendering the profile page.
- **Product pages:** Fetching product details and reviews in parallel.
- **Dashboard overviews:** Loading summary statistics and recent activity.

### References

- React Router — loader — https://reactrouter.com/6.30.3/route/loader
- React Router — useLoaderData — https://reactrouter.com/6.30.3/hooks/use-loader-data

---

## 23. Route Actions

### Definitions

**Core Definition:** A route action is a function that handles data mutations (form submissions) for a route.

**Technical Definition:** An `action` is an async function defined on a route object. It is called when a form is submitted via `<Form method="post">` or `useSubmit`. It receives `{ request, params }` and returns data or a `Response`. After the action completes, React Router revalidates loaders on the page.

**Beginner-Friendly Explanation:** An action handles form submissions. When a user submits a form, the action runs, updates the data, and the page reloads with fresh data — all without manual `onSubmit` handlers.

### Purposes

- To handle form submissions and data mutations declaratively.
- To automatically revalidate loaders after mutations.
- To centralize mutation logic at the route level.
- To provide progressive enhancement (forms work without JavaScript).

### Syntax Rules and Structure

```jsx
async function updateUserAction({ request, params }) {
  const formData = await request.formData();
  const updates = Object.fromEntries(formData);
  await updateUser(params.id, updates);
  return redirect(`/users/${params.id}`);
}

// Route: <Route path="/users/:id/edit" element={<EditUser />} action={updateUserAction} />
```

**Component Breakdown:**

| Parameter | Description |
|-----------|-------------|
| `request` | The `Request` object containing the submitted form data. |
| `params` | Dynamic segments from the URL. |
| Return value | Data, a `Response`, or `redirect()`. |

**Syntax Rules:**

1. Actions are triggered by `<Form method="post">` or `useSubmit`.
2. Returning `redirect()` navigates after the action.
3. Actions automatically revalidate all loaders on the page after completion.
4. Actions can throw `Response` objects for error handling.

**Constraints and Limitations:**

- Actions only work in data routers.
- Actions must be defined outside the component render function.
- Multiple actions on the same route are not supported; use a single action and branch on form intent.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Form Submission with Action

```jsx
import { Form, redirect, useActionData } from 'react-router-dom';

async function createTodoAction({ request }) {
  const formData = await request.formData();
  const title = formData.get('title');

  if (!title) {
    return { error: 'Title is required' };
  }

  await createTodo({ title });
  return redirect('/todos');
}

function NewTodo() {
  const actionData = useActionData();

  return (
    <Form method="post">
      <input name="title" placeholder="Todo title" />
      {actionData?.error && <p className="error">{actionData.error}</p>}
      <button type="submit">Create</button>
    </Form>
  );
}

// Route: <Route path="/todos/new" element={<NewTodo />} action={createTodoAction} />
```

**Expected Output:** Submitting the form creates a todo and redirects to `/todos`. If the title is empty, the error message renders.

**Why this output:** The `<Form>` submits to the route's action. The action validates the data, creates the todo, and returns a redirect. `useActionData` provides the returned error data to the component.

### Real-World Cases

- **Create/update forms:** Forms for creating and editing resources (users, posts, products).
- **Delete confirmations:** Actions that delete a resource and redirect to a list page.
- **Settings updates:** Forms that update user preferences or application settings.

### References

- React Router — action — https://reactrouter.com/6.30.3/route/action
- React Router — Form — https://reactrouter.com/6.30.3/components/form

---

## 24. Route Errors

### Definitions

**Core Definition:** Route errors are errors thrown by loaders or actions that are caught and rendered by an `errorElement`.

**Technical Definition:** When a `loader` or `action` throws an error (including `Response` objects), React Router renders the nearest `errorElement` in the route hierarchy. This provides route-level error boundaries without manual try/catch in components.

**Beginner-Friendly Explanation:** If something goes wrong while loading data for a page, React Router can show a friendly error page instead of crashing the whole app. You define this error page with `errorElement`.

### Purposes

- To handle data loading errors gracefully at the route level.
- To provide contextual error messages (e.g., "User not found").
- To avoid crashing the entire application when one route fails.
- To enable error bubbling and recovery.

### Syntax Rules and Structure

```jsx
<Route
  path="/users/:id"
  element={<UserProfile />}
  loader={userLoader}
  errorElement={<UserError />}
/>
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `errorElement` | A React element rendered when a loader/action in this route or its children throws. |

**Syntax Rules:**

1. `errorElement` catches errors from the route's `loader`, `action`, and child routes.
2. Throwing a `Response` with a status code makes `useRouteError()` return that response.
3. Errors bubble up to the nearest `errorElement`.
4. `useRouteError()` provides the thrown error or response.

**Constraints and Limitations:**

- `errorElement` does not catch errors thrown in event handlers or `useEffect`; those require traditional error boundaries.
- If no `errorElement` is defined, the error bubbles to the router's default error handling.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Route-Level Error Boundary

```jsx
import { useRouteError } from 'react-router-dom';

function UserError() {
  const error = useRouteError();

  if (error instanceof Response) {
    return (
      <div>
        <h1>Error {error.status}</h1>
        <p>{error.statusText}</p>
      </div>
    );
  }

  return <h1>Something went wrong</h1>;
}

// Route
<Route
  path="/users/:id"
  element={<UserProfile />}
  loader={userLoader}
  errorElement={<UserError />}
/>
```

**Expected Output (when the user is not found):**

```html
<div>
  <h1>Error 404</h1>
  <p>Not Found</p>
</div>
```

**Why this output:** The loader throws a `Response` with status 404. React Router catches it and renders the nearest `errorElement`. `useRouteError()` returns the `Response` object, which the component inspects.

### Real-World Cases

- **404 pages:** Showing a "Not Found" page when a resource does not exist.
- **Permission errors:** Displaying an "Access Denied" message when the user lacks permission.
- **Network errors:** Showing a retry UI when a fetch fails.

### References

- React Router — errorElement — https://reactrouter.com/6.30.3/route/error-element
- React Router — useRouteError — https://reactrouter.com/6.30.3/hooks/use-route-error

---

## 25. Route Lazy Loading

### Definitions

**Core Definition:** Route lazy loading is the technique of loading a route's component and data modules on demand, rather than including them in the initial bundle.

**Technical Definition:** The `lazy` property on a route object is an async function that resolves non-route-matching portions of the route definition (`loader`, `action`, `Component`/`element`, `ErrorBoundary`/`errorElement`). React Router calls this function only when the route is first matched, enabling code splitting.

**Beginner-Friendly Explanation:** Lazy loading means React Router downloads the code for a page only when the user visits that page. This makes the initial app load faster because not all pages' code is loaded upfront.

### Purposes

- To reduce the initial JavaScript bundle size.
- To improve initial page load performance.
- To load route components, loaders, and error boundaries on demand.
- To enable code splitting at the route level.

### Syntax Rules and Structure

```jsx
const router = createBrowserRouter([
  {
    path: '/dashboard',
    lazy: () => import('./Dashboard'),
    // The imported module should export Component, loader, action, etc.
  },
]);
```

**Lazy module exports:**

```jsx
// Dashboard.jsx
export function Component() {
  return <h1>Dashboard</h1>;
}

export async function loader() {
  return fetchDashboardData();
}

export function ErrorBoundary() {
  return <h1>Dashboard Error</h1>;
}
```

**Component Breakdown:**

| Export | Description |
|--------|-------------|
| `Component` | The route component. |
| `loader` | The route loader. |
| `action` | The route action. |
| `ErrorBoundary` | The route error boundary. |

**Syntax Rules:**

1. The `lazy` function must return a Promise that resolves to an object with the route's non-matching properties.
2. React Router automatically maps exported names (`Component`, `loader`, `action`, `ErrorBoundary`) to route properties.
3. Static `loader`/`action` definitions are called in parallel with the `lazy` function for optimization.

**Constraints and Limitations:**

- The `lazy` function must be defined on the route object; it cannot be used with declarative `<Routes>`.
- Lazy loading introduces a network round trip on first navigation to the route.
- Proper bundler configuration (Vite, webpack) is required for code splitting.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Lazy Loading a Route Module

```jsx
// router.js
import { createBrowserRouter } from 'react-router-dom';

const router = createBrowserRouter([
  {
    path: '/',
    element: <RootLayout />,
    children: [
      {
        path: 'dashboard',
        lazy: () => import('./routes/dashboard'),
      },
    ],
  },
]);
```

```jsx
// routes/dashboard.jsx
export function Component() {
  return (
    <div>
      <h1>Dashboard</h1>
      <p>Loaded lazily!</p>
    </div>
  );
}

export async function loader() {
  return { stats: await fetchStats() };
}

export function ErrorBoundary() {
  return <h1>Failed to load dashboard</h1>;
}
```

**Expected Output:** When the user navigates to `/dashboard`, React Router dynamically imports `routes/dashboard.jsx`, then renders the `Component` with data from `loader`. If the import fails, `ErrorBoundary` renders.

**Why this output:** The `lazy` function is called only when the route matches. The bundler creates a separate chunk for `dashboard.jsx`, which is downloaded on demand.

### Real-World Cases

- **Large applications:** Splitting routes into separate chunks to keep the initial bundle small.
- **Admin panels:** Lazy loading admin-only routes that most users never visit.
- **Feature modules:** Loading feature-specific code only when the feature is accessed.

### References

- React Router — lazy — https://reactrouter.com/6.30.3/route/lazy
- React Router — Lazy Loading Routes — https://reactrouter.com/6.30.3/start/tutorial#lazy-loading

---

## 26. Route Data APIs

### Definitions

**Core Definition:** Route data APIs are the set of features (loaders, actions, error boundaries, lazy loading, revalidation) that enable data-driven routing in React Router v6.4+.

**Technical Definition:** Data APIs are available only when using data routers (`createBrowserRouter`, `createHashRouter`, `createMemoryRouter`). They provide route-level data fetching (`loader`), mutations (`action`), error handling (`errorElement`), code splitting (`lazy`), and automatic revalidation after mutations.

**Beginner-Friendly Explanation:** Data APIs let routes handle their own data — fetching data before rendering, handling form submissions, and showing error pages — all without writing manual `useEffect` or `onSubmit` code.

### Purposes

- To eliminate the "fetch in useEffect" waterfall pattern.
- To enable parallel data loading for nested routes.
- To provide automatic revalidation after mutations.
- To centralize data logic at the route level.
- To improve performance by starting data fetching before rendering.

### Syntax Rules and Structure

**Data Router Setup:**

```jsx
import { createBrowserRouter, RouterProvider } from 'react-router-dom';

const router = createBrowserRouter([
  {
    path: '/users/:id',
    element: <UserProfile />,
    loader: userLoader,
    action: userAction,
    errorElement: <UserError />,
    lazy: () => import('./UserProfile'),
  },
]);

function App() {
  return <RouterProvider router={router} />;
}
```

**Component Breakdown:**

| API | Description |
|-----|-------------|
| `createBrowserRouter` | Creates a data router that supports loaders, actions, etc. |
| `loader` | Fetches data before rendering. |
| `action` | Handles form mutations. |
| `errorElement` | Renders when loader/action throws. |
| `lazy` | Lazy loads route modules. |
| `useLoaderData` | Accesses loader data in components. |
| `useActionData` | Accesses action data in components. |
| `useNavigation` | Provides pending navigation state (for loading indicators). |

**Syntax Rules:**

1. Data APIs require a data router (`createBrowserRouter`, `createHashRouter`, `createMemoryRouter`).
2. `RouterProvider` must be used instead of `<BrowserRouter>`.
3. Loaders run in parallel for nested routes.
4. Actions automatically revalidate loaders on the page.

**Constraints and Limitations:**

- Data APIs are not supported in `react-router-native` (as of v6.30).
- Declarative `<BrowserRouter>` does not support loaders/actions.
- Data APIs require React 18+ and Node 18+ (for `fetch`).

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Complete Data API Setup

```jsx
import {
  createBrowserRouter,
  RouterProvider,
  useLoaderData,
  useNavigation,
} from 'react-router-dom';

async function todosLoader() {
  const res = await fetch('/api/todos');
  return res.json();
}

async function addTodoAction({ request }) {
  const formData = await request.formData();
  await fetch('/api/todos', {
    method: 'POST',
    body: formData,
  });
  return null; // Triggers revalidation of todosLoader
}

function Todos() {
  const todos = useLoaderData();
  const navigation = useNavigation();
  const isSubmitting = navigation.state === 'submitting';

  return (
    <div>
      {isSubmitting && <p>Adding todo...</p>}
      <ul>
        {todos.map((todo) => (
          <li key={todo.id}>{todo.title}</li>
        ))}
      </ul>
      <Form method="post">
        <input name="title" />
        <button type="submit">Add</button>
      </Form>
    </div>
  );
}

const router = createBrowserRouter([
  {
    path: '/',
    element: <Todos />,
    loader: todosLoader,
    action: addTodoAction,
  },
]);

function App() {
  return <RouterProvider router={router} />;
}
```

**Expected Output:** The todo list loads before rendering. Submitting the form triggers the action, which adds the todo and revalidates the loader, updating the list. `useNavigation` shows a pending indicator during submission.

**Why this output:** The data router calls `todosLoader` before rendering. The `<Form>` submits to `addTodoAction`, which mutates data and returns `null`, triggering automatic revalidation of the loader. The component receives fresh data via `useLoaderData`.

### Real-World Cases

- **CRUD applications:** List, create, edit, and delete resources with loaders and actions.
- **Dashboards:** Loading multiple data sources in parallel for a dashboard overview.
- **Form-heavy apps:** Using actions for form submissions and automatic revalidation.

### References

- React Router — Data APIs — https://reactrouter.com/6.30.3/routers/picking-a-router#data-apis
- React Router — Data Mode — https://reactrouter.com/6.30.3/start/data-mode

---

## 27. React Router Versions

### Definitions

**Core Definition:** React Router versions represent major releases of the library, each introducing new APIs, deprecations, and migration paths.

**Technical Definition:** React Router follows semantic versioning. Major versions (v5, v6, v7, v8) may include breaking changes. React Router v6 introduced `Routes`/`Route` (replacing `Switch`), `useNavigate` (replacing `useHistory`), and nested routing. v6.4 added data APIs (loaders, actions). v7 unified packages and requires React 18+. Future flags and opt-in APIs are used to ease migrations.

**Beginner-Friendly Explanation:** React Router has had several major versions. If you search for tutorials online, make sure they match your version — v6 is very different from v5, and v7 continues the data API approach.

### Purposes

- To understand the evolution of React Router APIs.
- To choose the correct version for a project.
- To plan migrations between major versions.
- To use future flags for gradual adoption of new features.

### Syntax Rules and Structure

**Version History Overview:**

| Version | Key Changes |
|---------|-------------|
| v5 | `Switch`, `useHistory`, `component`/`render`/`children` props. |
| v6 | `Routes`, `useNavigate`, `element` prop, nested routing, relative paths. |
| v6.4 | Data APIs: `createBrowserRouter`, `loader`, `action`, `errorElement`, `useLoaderData`. |
| v7 | Unified package (`react-router`), requires React 18+, framework mode, future flags. |
| v8 | Future release; adoption of v7 future flags eases migration. |

**Migration Approach:**

1. Update to the latest minor version of the current major (e.g., latest v7.x).
2. Adopt all future flags (opt-in APIs) for the next major version.
3. Address warnings and deprecations.
4. Upgrade to the next major version.

**Constraints and Limitations:**

- v6 does not support class component `lifecycle` methods for data fetching; use loaders.
- v7 requires Node 20+ and React 18+.
- Future flags are version-specific and must be enabled via router configuration.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Using Future Flags (v7 Example)

```jsx
// Enabling future flags in v7 to prepare for v8
const router = createBrowserRouter(
  [
    // routes
  ],
  {
    future: {
      // Enable v8 future flags (example names)
      v8_middleware: true,
      v8_relativeSplatPath: true,
    },
  }
);
```

**Expected behavior:** Enabling future flags opts into v8 behaviors while still on v7, reducing migration friction when v8 is released.

### Real-World Cases

- **Legacy applications:** Migrating from v5 to v6 (or v6 to v7) requires understanding the API differences.
- **New projects:** Starting with v7 and using data APIs from the beginning.
- **Framework integration:** Remix and React Router framework mode follow the v7+ API direction.

### References

- React Router — Upgrading from v5 — https://reactrouter.com/6.30.3/upgrading/v5
- React Router — Upgrading from v6 — https://reactrouter.com/upgrading/v6
- React Router — Future Changes — https://reactrouter.com/upgrading/future

---

## 28. React Router Best Practices

### Definitions

**Core Definition:** React Router best practices are guidelines for structuring routes, managing navigation, handling data, and avoiding common pitfalls.

**Technical Definition:** Best practices encompass choosing the right router for the environment, using data routers for data-driven apps, preferring `Link`/`NavLink` over `useNavigate` for user-initiated navigation, keeping route objects outside render, using nested layouts with `<Outlet>`, leveraging loaders/actions over `useEffect` fetching, and sanitizing user input.

**Beginner-Friendly Explanation:** Best practices are the recommended ways to use React Router based on what works well in real projects. Following them helps you avoid bugs and performance issues.

### Purposes

- To choose the correct router for each environment (web, test, native).
- To optimize data loading and avoid waterfalls.
- To ensure accessibility and good user experience.
- To maintain a scalable and readable route configuration.
- To avoid common security and performance pitfalls.

### Syntax Rules and Structure

**Key Best Practices Checklist:**

| Area | Best Practice |
|------|---------------|
| Router choice | Use `createBrowserRouter` for web apps with data APIs. |
| Testing | Use `MemoryRouter` or `createMemoryRouter`. |
| Navigation | Prefer `Link`/`NavLink` for user-initiated navigation; reserve `useNavigate` for programmatic cases. |
| Route objects | Define route objects outside of render functions. |
| Layouts | Use nested routes with `<Outlet>` for shared layouts. |
| Data fetching | Prefer loaders/actions over `useEffect` fetching. |
| Index routes | Use index routes for default child content. |
| Lazy loading | Use `route.lazy` for code splitting. |
| Error handling | Define `errorElement` at appropriate route levels. |
| Dynamic segments | Use `:param` for dynamic segments; avoid partial segments. |
| Security | Sanitize HTML if using `dangerouslySetInnerHTML`; validate spread props. |

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Recommended Data Router Setup

```jsx
import { createBrowserRouter, RouterProvider } from 'react-router-dom';

// Route objects defined outside render
const router = createBrowserRouter([
  {
    path: '/',
    element: <RootLayout />,
    errorElement: <GlobalError />,
    children: [
      { index: true, element: <Home /> },
      {
        path: 'dashboard',
        lazy: () => import('./routes/dashboard'),
        children: [
          { index: true, element: <Overview /> },
          { path: 'settings', element: <Settings /> },
        ],
      },
    ],
  },
]);

function App() {
  return <RouterProvider router={router} />;
}
```

**Why this follows best practices:**

- Uses `createBrowserRouter` (recommended for web apps).
- Defines the router outside the component (avoids re-creation on every render).
- Uses nested routes with a layout route (`RootLayout` + `<Outlet>`).
- Uses `index` routes for default child content.
- Uses `lazy` for code splitting.
- Defines `errorElement` at the root level for global error handling.
- Uses `lazy` for the dashboard route to keep the initial bundle small.

### Real-World Cases

- **Production SPA:** Following these practices ensures fast initial loads, graceful error handling, and maintainable code.
- **Team projects:** Consistency in routing patterns reduces onboarding time and bugs.
- **Framework migration:** React Router best practices align with Remix and Next.js App Router patterns.

### References

- React Router — Picking a Router — https://reactrouter.com/6.30.3/routers/picking-a-router
- React Router — Data Mode — https://reactrouter.com/6.30.3/start/data-mode
- React Router — Navigating — https://reactrouter.com/6.30.3/navigating
- React Router — Route — https://reactrouter.com/6.30.3/route/route

---

## References

- React Router — Official Documentation — https://reactrouter.com/
- React Router — API Reference (GitHub) — https://github.com/remix-run/react-router/blob/main/docs/api.md
- React Router — Picking a Router — https://reactrouter.com/6.30.3/routers/picking-a-router
- React Router — BrowserRouter — https://reactrouter.com/6.30.3/router-components/browser-router
- React Router — HashRouter — https://reactrouter.com/6.30.3/router-components/hash-router
- React Router — MemoryRouter — https://reactrouter.com/6.30.3/router-components/memory-router
- React Router — Routes — https://reactrouter.com/6.30.3/components/routes
- React Router — Route — https://reactrouter.com/6.30.3/route/route
- React Router — Link — https://reactrouter.com/6.30.3/components/link
- React Router — NavLink — https://reactrouter.com/6.30.3/components/nav-link
- React Router — Navigate — https://reactrouter.com/6.30.3/components/navigate
- React Router — Outlet — https://reactrouter.com/6.30.3/components/outlet
- React Router — useNavigate — https://reactrouter.com/6.30.3/hooks/use-navigate
- React Router — useParams — https://reactrouter.com/6.30.3/hooks/use-params
- React Router — useSearchParams — https://reactrouter.com/6.30.3/hooks/use-search-params
- React Router — useLocation — https://reactrouter.com/6.30.3/hooks/use-location
- React Router — useMatch — https://reactrouter.com/6.30.3/hooks/use-match
- React Router — useResolvedPath — https://reactrouter.com/6.30.3/hooks/use-resolved-path
- React Router — useHref — https://reactrouter.com/6.30.3/hooks/use-href
- React Router — useNavigationType — https://reactrouter.com/6.30.3/hooks/use-navigation-type
- React Router — loader — https://reactrouter.com/6.30.3/route/loader
- React Router — action — https://reactrouter.com/6.30.3/route/action
- React Router — errorElement — https://reactrouter.com/6.30.3/route/error-element
- React Router — lazy — https://reactrouter.com/6.30.3/route/lazy
- React Router — Data Mode — https://reactrouter.com/6.30.3/start/data-mode
- React Router — Upgrading from v5 — https://reactrouter.com/6.30.3/upgrading/v5
- React Router — Upgrading from v6 — https://reactrouter.com/upgrading/v6
- React Router — Future Changes — https://reactrouter.com/upgrading/future