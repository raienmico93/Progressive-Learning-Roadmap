# React Router: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** React Router is the standard routing library for React applications, providing declarative, component-based routing that maps URL paths to React components while supporting nested layouts, dynamic segments, data loading, and navigation APIs.

**Technical Definition:** React Router v7 is a multi-strategy routing library that supports three modes: **declarative** (JSX-based `<BrowserRouter>`, `<Routes>`, `<Route>`), **data** (object-based `createBrowserRouter` with route objects supporting `loader`, `action`, and `middleware`), and **framework** (a Vite plugin with file-based or config-based routes, server rendering, and typed route modules). It requires Node 20+, React 18+, and React DOM 18+. The library provides hooks (`useParams`, `useSearchParams`, `useNavigate`, `useLocation`), components (`Link`, `NavLink`, `Outlet`, `Navigate`), and utilities (`redirect`, `generatePath`) for building complete client-side routing experiences. React Router v7 recommends the data router (`createBrowserRouter` + `RouterProvider`) as the default for its data-loading features, while `<BrowserRouter>` remains available for simpler declarative setups.

**Beginner-Friendly Explanation:** React Router is the traffic controller of your React app. When a user clicks a link or types a URL, React Router decides which component to show, keeps the browser's back/forward buttons working, and lets you build complex layouts with nested sections—all without reloading the page. It is like a GPS for your app: it knows every route, how to get there, and what to display when you arrive.

### Key Characteristics

- **Three Modes:** Declarative (JSX routes), Data (object routes with loaders/actions), and Framework (full-stack Vite plugin).
- **Nested Routing:** Routes nest inside parent routes, with child content rendered through `<Outlet />`.
- **Dynamic Segments:** URL parameters via `:paramName` syntax, accessed with `useParams`.
- **Query Parameter Synchronization:** URL search parameters managed with `useSearchParams`; setting them causes navigation.
- **Accessible Navigation:** `<Link>` and `<NavLink>` render real anchor tags, preserving keyboard and screen-reader support.
- **Data Loading:** Route objects support `loader` functions that provide data before rendering.
- **Programmatic Navigation:** `useNavigate` for imperative navigation; `<Navigate>` for declarative redirects.

### Prerequisites

- Solid understanding of React function components, JSX, and props.
- Familiarity with the `useState` and `useEffect` Hooks.
- Working knowledge of the component tree and conditional rendering.
- Basic understanding of URL structure (paths, query strings).

### Related Programming Areas

- **SPA Architecture:** Single-page applications and client-side navigation.
- **State Management:** URL state, navigation state, and application state synchronization.
- **Data Loading:** Server-state management with route loaders.
- **Accessibility:** Focus management, semantic markup, and screen-reader support.

### Core Concepts / Features

1. Installation, Setup, and Router Types
2. Route Components and Object-Based Route Definitions
3. Route Parameters and `useParams`
4. Nested Routes and the `Outlet` Component
5. Dynamic Routes and Catch-All Paths
6. Query Parameters and `useSearchParams`
7. Navigation APIs (`Link`, `NavLink`, `useNavigate`, `useLocation`)
8. Redirects and Programmatic Navigation (`Navigate`)
9. Route Layouts and Presentation Wrappers

---

## Core Concept 1: Installation, Setup, and Router Types

### Definitions

**Core Definition:** React Router installation involves adding the `react-router` package to a project and choosing between two primary router types: the declarative `<BrowserRouter>` and the data-oriented `createBrowserRouter`.

**Technical Definition:** React Router v7 requires Node 20+, React 18+, and React DOM 18+. Install with `npm i react-router`. Two router types are available. `<BrowserRouter>` is a component wrapper that provides routing context to the entire app and works with `<Routes>` and `<Route>` JSX components. `createBrowserRouter` is a function that accepts an array of route objects and returns a router instance rendered with `<RouterProvider>`. The data router (`createBrowserRouter`) supports loaders, actions, middleware, and error boundaries; `<BrowserRouter>` does not. In v7, the data router is recommended as the default for applications needing data features.

**Beginner-Friendly Explanation:** There are two ways to set up React Router. The simple way is to wrap your app in `<BrowserRouter>` and define routes with `<Route>` tags—like hanging signs on a wall. The more powerful way is to use `createBrowserRouter`, which lets you define routes as configuration objects and attach data-loading logic to each route—like a smart sign that also fetches the information before showing it.

### Purposes

- To install and configure React Router in a React project.
- To choose the appropriate router type based on data-loading needs.
- To set up the routing context that all navigation hooks and components depend on.
- To enable route-level data loading and mutations with the data router.

### Syntax Rules and Structure

**Installation:**
```bash
npm i react-router
```

**Declarative Setup (`<BrowserRouter>`):**
```jsx
import { BrowserRouter, Routes, Route } from "react-router";
import App from "./app";

const root = document.getElementById("root");
ReactDOM.createRoot(root).render(
  <BrowserRouter>
    <App />
  </BrowserRouter>
);
```

**Data Router Setup (`createBrowserRouter`):**
```jsx
import { createBrowserRouter, RouterProvider } from "react-router";

const router = createBrowserRouter([
  {
    path: "/",
    Component: App,
    children: [
      { index: true, Component: Home },
      { path: "about", Component: About },
    ],
  },
]);

ReactDOM.createRoot(document.getElementById("root")).render(
  <RouterProvider router={router} />
);
```

**Syntax Rules:**
- Install `react-router` (not `react-router-dom` in v7).
- Wrap the app in `<BrowserRouter>` for declarative routing, or use `<RouterProvider router={router} />` for data routing.
- Route objects in `createBrowserRouter` use `path`, `Component` (or `element`), `loader`, `action`, and `children`.
- The data router requires `createBrowserRouter` + `RouterProvider`; hooks like `useBlocker` only work with data routers.

**Constraints and Limitations:**
- `<BrowserRouter>` does not support data features (loaders, actions, middleware).
- `createBrowserRouter` is recommended for most new applications.
- Node 20+ and React 18+ are required for v7.

### Annotated Code Example: Data Router with Loader

```jsx
import { createBrowserRouter, RouterProvider, useLoaderData } from "react-router";

// Loader: runs before the component renders
async function todosLoader() {
  const res = await fetch("https://jsonplaceholder.typicode.com/todos?_limit=5");
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.json();
}

// Component: reads loader data
function Todos() {
  const todos = useLoaderData();
  return (
    <ul>
      {todos.map(t => <li key={t.id}>{t.title}</li>)}
    </ul>
  );
}

// Route configuration
const router = createBrowserRouter([
  { path: "/", element: <Todos />, loader: todosLoader },
]);

export default function App() {
  return <RouterProvider router={router} />;
}
```

**Expected Output:** The app renders a list of five todos fetched from the API before the component renders. No loading spinner is needed—the loader resolves before the component mounts.

**Why This Output Occurs:** The `loader` function runs before the route component renders. `useLoaderData` reads the resolved data. The `RouterProvider` manages the routing context and data lifecycle.

### References

- React Router v7 – Installation: https://reactrouter.com/7.1.0/start/library/installation
- React Router – Route Object: https://reactrouter.com/start/data/route-object
- React Router v7 – Migration Guide: https://www.uniflow.kr

---

## Core Concept 2: Route Components and Object-Based Route Definitions

### Definitions

**Core Definition:** Route components are the JSX elements (`<Route>`, `<Routes>`) or configuration objects that define the mapping between URL paths and React components.

**Technical Definition:** In declarative mode, routes are defined by rendering `<Routes>` and `<Route>` components that couple URL segments to UI elements. In data mode, routes are objects passed to `createBrowserRouter`, with properties including `path`, `Component` (or `element`), `loader`, `action`, `children`, `errorElement`, and `middleware`. `useRoutes` is the hook version of `<Routes>` that accepts an array of route objects and returns a React element. `createRoutesFromElements` converts JSX routes into route objects for use with data routers.

**Beginner-Friendly Explanation:** A route is a rule that says: "When the URL looks like this, show that component." You can write these rules as JSX tags (`<Route path="/about" element={<About />} />`) or as configuration objects (`{ path: "/about", element: <About /> }`). The object-based approach is more powerful because you can attach data loaders and error handling to each route.

### Purposes

- To declare the mapping between URL paths and React components.
- To use object-based route definitions for type safety and data features.
- To support nested route hierarchies with `children`.
- To attach loaders, actions, and error boundaries at the route level.

### Syntax Rules and Structure

**Declarative Routes (JSX):**
```jsx
import { Routes, Route } from "react-router";

<Routes>
  <Route path="/" element={<Home />} />
  <Route path="about" element={<About />} />
  <Route path="users">
    <Route path=":userId" element={<UserProfile />} />
  </Route>
</Routes>
```

**Object-Based Routes:**
```jsx
const routes = [
  { path: "/", element: <Home /> },
  { path: "about", element: <About /> },
  {
    path: "users",
    children: [
      { path: ":userId", element: <UserProfile /> },
    ],
  },
];
```

**Component vs. Element:**
```jsx
// Using element prop
{ path: "/", element: <Home /> }

// Using Component prop (React Router calls createElement internally)
{ path: "/", Component: Home }
```

**Syntax Rules:**
- In object routes, use `Component` instead of `element` for lazy loading and data router optimizations.
- Nested routes use the `children` array.
- Index routes use `{ index: true, element: <Home /> }`.
- `useRoutes` accepts an array of route objects and returns the route tree.

**Constraints and Limitations:**
- `element` and `Component` cannot be used together on the same route.
- Object routes must be passed to `createBrowserRouter` or `useRoutes`.

### Annotated Code Example: Object Routes with Nested Children

```jsx
import { createBrowserRouter, RouterProvider, Outlet } from "react-router";

function Dashboard() {
  return (
    <div>
      <h1>Dashboard</h1>
      <Outlet />
    </div>
  );
}

const router = createBrowserRouter([
  {
    path: "/",
    element: <Dashboard />,
    children: [
      { index: true, element: <p>Overview</p> },
      { path: "settings", element: <p>Settings</p> },
      { path: "profile", element: <p>Profile</p> },
    ],
  },
]);

export default function App() {
  return <RouterProvider router={router} />;
}
```

**Expected Output:** Navigating to `/` shows "Dashboard" with "Overview" below. Navigating to `/settings` shows "Dashboard" with "Settings".

**Why This Output Occurs:** The parent route renders `<Outlet />`, which renders the matching child route. The `index` route renders at the parent's path.

### References

- React Router – Route Object: https://reactrouter.com/start/data/route-object
- React Router – useRoutes: https://reactrouter.com/api/hooks/useRoutes
- React Router – createRoutesFromElements: https://reactrouter.com/api/utils/createRoutesFromElements

---

## Core Concept 3: Route Parameters and the `useParams` Hook

### Definitions

**Core Definition:** Route parameters are dynamic segments in a URL path (defined with `:paramName`) that capture values from the URL and expose them to the component via the `useParams` Hook.

**Technical Definition:** The `useParams` Hook returns an object of key/value pairs of the dynamic params from the current URL that were matched by the route path. Child routes inherit all params from their parent routes. For example, if the route pattern is `/posts/:postId` and the URL is `/posts/123`, then `params.postId` is `"123"`. Params are always strings; convert them to numbers if needed.

**Beginner-Friendly Explanation:** Route parameters are placeholders in your URL. If your route is `/users/:userId`, then `/users/42` captures `42` as the `userId` parameter. The `useParams` Hook is how your component reads that value. It's like a mail slot: the URL puts the value in, and `useParams` takes it out.

### Purposes

- To capture dynamic values from the URL (IDs, slugs, usernames).
- To render components that depend on URL data (profile pages, product details).
- To inherit parent route params in nested child routes.

### Syntax Rules and Structure

**Route Definition:**
```jsx
<Route path="users/:userId" element={<UserProfile />} />
```

**Reading Params:**
```jsx
import { useParams } from "react-router";

function UserProfile() {
  const { userId } = useParams();
  return <h1>User ID: {userId}</h1>;
}
```

**Multiple Params:**
```jsx
<Route path="/posts/:postId/comments/:commentId" element={<Comment />} />

function Comment() {
  const { postId, commentId } = useParams();
  return <p>Post {postId}, Comment {commentId}</p>;
}
```

**Syntax Rules:**
- Define params with `:paramName` in the route path.
- Access params with `useParams()`—it returns an object.
- Params are always strings; convert to numbers with `Number()` or `parseInt()`.
- Child routes inherit parent params.

**Constraints and Limitations:**
- Params cannot be optional in v7 (use query params for optional values).
- `useParams` returns an empty object if no params are matched.
- Changing params does not remount the component; use `useEffect` to react to param changes.

### Annotated Code Example: Product Detail with Param

```jsx
import { Routes, Route, useParams } from "react-router";

const products = {
  1: { name: "Laptop", price: 999 },
  2: { name: "Phone", price: 699 },
};

function ProductDetail() {
  const { productId } = useParams();
  const product = products[productId];

  if (!product) return <p>Product not found</p>;

  return (
    <div>
      <h1>{product.name}</h1>
      <p>${product.price}</p>
    </div>
  );
}

export default function App() {
  return (
    <Routes>
      <Route path="/products/:productId" element={<ProductDetail />} />
    </Routes>
  );
}
```

**Expected Output:** Navigating to `/products/1` shows "Laptop — $999". Navigating to `/products/2` shows "Phone — $699".

**Why This Output Occurs:** The `:productId` param captures the URL segment. `useParams` extracts it, and the component looks up the product in the `products` object.

### References

- React Router – useParams: https://reactrouter.com/6.30.3/hooks/use-params
- React Router – useParams (v7): https://beta.reactrouter.com

---

## Core Concept 4: Nested Routes and the `Outlet` Component

### Definitions

**Core Definition:** Nested routes allow routes to be defined inside other routes, with child route content rendered through the parent route's `<Outlet />` component, enabling hierarchical layouts.

**Technical Definition:** `<Outlet>` renders the matching child route of a parent route, or nothing if no child route matches. It can be used multiple times in nested route hierarchies. Index routes render into their parent's `<Outlet />` at the parent's URL (like a default child route). `useOutletContext` provides a context value to the child routes from the parent route.

**Beginner-Friendly Explanation:** Nested routes are like Russian dolls. The outer doll (parent route) has a space inside where the next doll (child route) fits. The `<Outlet />` is that space. When you navigate to a child route, React Router puts the child's component inside the parent's `<Outlet />`, so both the parent layout and child content are visible.

### Purposes

- To build hierarchical layouts where parent UI persists across child routes.
- To share layout chrome (headers, sidebars, footers) across multiple pages.
- To compose complex UIs from smaller route segments.
- To support index routes as default children.

### Syntax Rules and Structure

**Parent Route with Outlet:**
```jsx
import { Outlet } from "react-router";

function Dashboard() {
  return (
    <div>
      <h1>Dashboard</h1>
      <nav>
        <a href="/dashboard/messages">Messages</a>
        <a href="/dashboard/tasks">Tasks</a>
      </nav>
      <Outlet />
    </div>
  );
}
```

**Route Configuration:**
```jsx
<Routes>
  <Route path="/" element={<Dashboard />}>
    <Route path="messages" element={<DashboardMessages />} />
    <Route path="tasks" element={<DashboardTasks />} />
  </Route>
</Routes>
```

**Index Route:**
```jsx
<Route index element={<Overview />} />
```

**Syntax Rules:**
- The parent route must render `<Outlet />` where child routes should appear.
- Child routes use relative paths (no leading slash).
- Index routes use `index` instead of `path` and render at the parent's URL.
- Nested routes can be arbitrarily deep.

**Constraints and Limitations:**
- A parent route without an `<Outlet />` will not render child routes.
- Index routes cannot have children.
- `<Outlet />` can be used multiple times, but only the matching child renders.

### Annotated Code Example: Dashboard with Nested Routes

```jsx
import { Routes, Route, Outlet, Link } from "react-router";

function Dashboard() {
  return (
    <div>
      <h1>Dashboard</h1>
      <nav>
        <Link to="messages">Messages</Link>
        <Link to="tasks">Tasks</Link>
      </nav>
      <Outlet />
    </div>
  );
}

function Messages() { return <p>Your messages</p>; }
function Tasks() { return <p>Your tasks</p>; }

export default function App() {
  return (
    <Routes>
      <Route path="/" element={<Dashboard />}>
        <Route index element={<p>Overview</p>} />
        <Route path="messages" element={<Messages />} />
        <Route path="tasks" element={<Tasks />} />
      </Route>
    </Routes>
  );
}
```

**Expected Output:** Navigating to `/` shows "Dashboard" with "Overview". Navigating to `/messages` shows "Dashboard" with "Your messages". Navigating to `/tasks` shows "Dashboard" with "Your tasks".

**Why This Output Occurs:** The `Dashboard` component renders `<Outlet />`, which renders the matching child route. The `index` route renders at `/`, and the `messages` and `tasks` routes render at their respective paths.

### References

- React Router – Outlet: https://reactrouter.com/6.30.3/components/outlet
- React Router – Nested Routes: https://mintlify.wiki/remix-run/react-router/routing/nested-routes
- React Router – useOutletContext: https://reactrouter.de

---

## Core Concept 5: Dynamic Routes and Catch-All Paths

### Definitions

**Core Definition:** Dynamic routes use URL parameters to match variable path segments, while catch-all paths use wildcard characters (`*`) to match any remaining segments for 404 handling.

**Technical Definition:** Dynamic segments are defined with `:paramName` syntax. Catch-all params are defined with `*` and can be accessed via `useParams()["*"]`. The wildcard `*` allows the route to match any descendant path, making it ideal for 404 pages. React Router also supports splat routes with `path="*"` for catching undefined routes.

**Beginner-Friendly Explanation:** A dynamic route is a flexible path that accepts any value in a specific position—like a form with a blank you fill in. A catch-all route is a safety net that catches any URL that doesn't match your defined routes, showing a 404 page instead of a blank screen.

### Purposes

- To handle variable URL segments (IDs, slugs, usernames).
- To create 404 pages that catch undefined routes.
- To support splat routes for matching any remaining path.
- To enable dynamic content rendering based on URL segments.

### Syntax Rules and Structure

**Dynamic Segment:**
```jsx
<Route path="users/:userId" element={<User />} />
// /users/42 → userId = "42"
```

**Catch-All Route:**
```jsx
<Route path="*" element={<NotFound />} />
// Any unmatched URL → NotFound
```

**Splat Route with Params:**
```jsx
<Route path="files/*" element={<FileViewer />} />
// /files/a/b/c → params["*"] = "a/b/c"

function FileViewer() {
  const params = useParams();
  const splat = params["*"]; // "a/b/c"
}
```

**Syntax Rules:**
- Define catch-all routes with `path="*"`.
- Access splat params with `useParams()["*"]`.
- Use `*` as the last route to catch all unmatched paths.
- Dynamic segments use `:paramName` and are accessed with `useParams()`.

**Constraints and Limitations:**
- Catch-all routes should be the last route defined.
- Splat params are always strings and may be empty.
- v7 does not support regex constraints in path patterns.

### Annotated Code Example: 404 Page with Catch-All

```jsx
import { Routes, Route, useParams } from "react-router";

function NotFound() {
  return <h1>404 — Page not found</h1>;
}

function CatchAll() {
  const params = useParams();
  return <p>Caught path: {params["*"]}</p>;
}

export default function App() {
  return (
    <Routes>
      <Route path="/" element={<h1>Home</h1>} />
      <Route path="/about" element={<h1>About</h1>} />
      <Route path="/files/*" element={<CatchAll />} />
      <Route path="*" element={<NotFound />} />
    </Routes>
  );
}
```

**Expected Output:** Navigating to `/` shows "Home". Navigating to `/about` shows "About". Navigating to `/files/a/b/c` shows "Caught path: a/b/c". Navigating to `/unknown` shows "404 — Page not found".

**Why This Output Occurs:** The `/files/*` route catches any path starting with `/files/` and captures the rest as a splat param. The `*` route catches all other unmatched paths and renders the 404 page.

### References

- React Router – useParams (Catchall Params): https://reactrouter.com/6.30.3/hooks/use-params
- React Router – Catch-All Routes: https://www.educative.io
- Stack Overflow – Dynamic Routes with Catch-All

---

## Core Concept 6: Query Parameters and the `useSearchParams` Hook

### Definitions

**Core Definition:** Query parameters are key-value pairs in the URL's search string (after `?`), managed in React Router via the `useSearchParams` Hook, which returns the current params and a function to update them.

**Technical Definition:** `useSearchParams` returns a tuple of the current URL's `URLSearchParams` object and a `setSearchParams` function. Setting the search params causes a navigation. The setter accepts a query string (`"?tab=1"`), a shorthand object (`{ tab: "1" }`), an array of tuples (`[["tab", "1"]]`), or a `URLSearchParams` object. It also supports a function callback similar to React's `setState`, though without queueing logic. The `searchParams` object is a stable reference, making it safe for `useEffect` dependencies.

**Beginner-Friendly Explanation:** Query parameters are the extra bits of information in a URL after the `?`. For example, in `/products?category=electronics&sort=price`, `category` and `sort` are query parameters. They are great for filters, search queries, and pagination because they are shareable and bookmarkable. `useSearchParams` lets you read and update them.

### Purposes

- To read and update URL query parameters.
- To synchronize filter, sort, and pagination state with the URL.
- To make UI state shareable and bookmarkable.
- To trigger navigation when query params change.

### Syntax Rules and Structure

**Reading Params:**
```jsx
const [searchParams, setSearchParams] = useSearchParams();
const category = searchParams.get("category") ?? "all";
const page = Number(searchParams.get("page") ?? "1");
```

**Setting Params:**
```jsx
// Query string
setSearchParams("?tab=1");

// Shorthand object
setSearchParams({ tab: "1" });

// Array of tuples
setSearchParams([["tab", "1"]]);

// Multiple values for one key
setSearchParams({ brand: ["nike", "reebok"] });

// Function callback
setSearchParams(prev => {
  prev.set("tab", "2");
  return prev;
});
```

**Syntax Rules:**
- Use `searchParams.get(key)` to read a single value.
- Use `searchParams.getAll(key)` for multiple values.
- Use default values with `??` to handle missing params.
- `searchParams` is a stable reference for `useEffect` dependencies.
- Do not mutate `searchParams` without calling `setSearchParams`.

**Constraints and Limitations:**
- URL state is public—do not store sensitive data.
- URL length is limited (2,000–8,000 characters).
- `setSearchParams` causes a navigation and re-render.
- The function callback does not support React's queueing logic.

### Annotated Code Example: Filter with Query Params

```jsx
import { useSearchParams } from "react-router";

function ProductFilters() {
  const [searchParams, setSearchParams] = useSearchParams();
  const category = searchParams.get("category") ?? "all";
  const sort = searchParams.get("sort") ?? "name";

  function updateFilter(key, value) {
    setSearchParams({ ...Object.fromEntries(searchParams), [key]: value });
  }

  return (
    <div>
      <select value={category} onChange={e => updateFilter("category", e.target.value)}>
        <option value="all">All</option>
        <option value="electronics">Electronics</option>
        <option value="clothing">Clothing</option>
      </select>
      <select value={sort} onChange={e => updateFilter("sort", e.target.value)}>
        <option value="name">Name</option>
        <option value="price">Price</option>
      </select>
      <p>Category: {category}, Sort: {sort}</p>
    </div>
  );
}
```

**Expected Output:** Changing the category or sort dropdown updates the URL query params (e.g., `?category=electronics&sort=price`) and the displayed text.

**Why This Output Occurs:** `useSearchParams` reads the current params. `setSearchParams` updates them, causing a navigation. The component re-renders with the new values.

### References

- React Router – useSearchParams: https://api.reactrouter.com/v7/functions/react-router.useSearchParams.html
- React Router – useSearchParams: https://reactrouter.com/api/hooks/useSearchParams

---

## Core Concept 7: Navigation APIs (`Link`, `NavLink`, `useNavigate`, `useLocation`)

### Definitions

**Core Definition:** React Router provides declarative navigation components (`Link`, `NavLink`) and imperative hooks (`useNavigate`, `useLocation`) for navigating between routes and reading the current location.

**Technical Definition:** `<Link>` renders an anchor tag that navigates client-side. `<NavLink>` behaves like `<Link>` but applies `aria-current="page"` when the route matches. `useNavigate` returns a function for imperative navigation. `useLocation` returns the current location object (`pathname`, `search`, `hash`, `state`). The `Link` component supports `to`, `replace`, `state`, and `preventScrollReset` props.

**Beginner-Friendly Explanation:** `<Link>` and `<NavLink>` are the "proper" ways to create navigation links—they look like real links and behave like real links, but they don't cause a full page reload. `useNavigate` is for when you need to navigate programmatically, like after a form submission. `useLocation` tells you where the user currently is, so you can read the URL or state.

### Purposes

- To create accessible navigation links with `<Link>` and `<NavLink>`.
- To navigate programmatically with `useNavigate`.
- To read the current location with `useLocation`.
- To apply active styling with `<NavLink>`.

### Syntax Rules and Structure

**Link:**
```jsx
import { Link } from "react-router";

<Link to="/about">About</Link>
<Link to="/users/1" state={{ from: "home" }}>User 1</Link>
```

**NavLink:**
```jsx
import { NavLink } from "react-router";

<NavLink to="/about" className={({ isActive }) => isActive ? "active" : ""}>
  About
</NavLink>
```

**useNavigate:**
```jsx
import { useNavigate } from "react-router";

const navigate = useNavigate();
navigate("/dashboard");
navigate(-1); // Go back
navigate("/login", { replace: true });
```

**useLocation:**
```jsx
import { useLocation } from "react-router";

const location = useLocation();
console.log(location.pathname); // "/about"
console.log(location.search);   // "?tab=1"
console.log(location.state);    // { from: "home" }
```

**Syntax Rules:**
- Use `<Link>` for user-initiated navigation that should be an anchor tag.
- Use `<NavLink>` for navigation menus with active state.
- Use `useNavigate` in event handlers and effects.
- Use `useLocation` to read the current URL or state.
- Use `replace: true` to replace the history entry instead of adding a new one.

**Constraints and Limitations:**
- `useNavigate` causes extra renders compared to `<Link>`.
- `navigate(-1)` may navigate outside the app if the history stack is shallow.
- `<NavLink>` active state matches by prefix by default; use `end` for exact matching.

### Annotated Code Example: Navigation Menu with Active States

```jsx
import { NavLink, useNavigate, useLocation } from "react-router";

function Navbar() {
  const navigate = useNavigate();
  const location = useLocation();

  return (
    <nav>
      <NavLink to="/" end className={({ isActive }) => isActive ? "active" : ""}>
        Home
      </NavLink>
      <NavLink to="/products" className={({ isActive }) => isActive ? "active" : ""}>
        Products
      </NavLink>
      <button onClick={() => navigate("/cart")}>Cart</button>
      <p>Current: {location.pathname}</p>
    </nav>
  );
}
```

**Expected Output:** The navbar shows "Home" and "Products" links with the current page highlighted. Clicking "Cart" navigates to `/cart`. The current path is displayed.

**Why This Output Occurs:** `<NavLink>` adds `aria-current` and applies the active class when the route matches. `useNavigate` navigates programmatically. `useLocation` reads the current pathname.

### References

- React Router – Link: https://reactrouter.com/api/components/Link
- React Router – NavLink: https://reactrouter.com/api/components/NavLink
- React Router – useNavigate: https://reactrouter.com/api/hooks/useNavigate
- React Router – useLocation: https://reactrouter.com/api/hooks/useLocation

---

## Core Concept 8: Redirects and Programmatic Navigation (`Navigate`)

### Definitions

**Core Definition:** Redirects in React Router can be performed declaratively with the `<Navigate>` component (which navigates immediately when rendered) or programmatically with the `useNavigate` Hook.

**Technical Definition:** `<Navigate>` is a component wrapper around `useNavigate` that changes the current location when rendered. It accepts `to`, `replace`, and `state` props. It is used for conditional redirects like route protection. `useNavigate` returns a function for imperative navigation, ideal for redirects after form submissions or button clicks. In data routers, `redirect()` can be returned from loaders and actions.

**Beginner-Friendly Explanation:** A redirect is like a detour sign. When the user reaches a point where they shouldn't be (e.g., a protected page when not logged in), you send them somewhere else. `<Navigate>` is a sign you place in the route render. `useNavigate` is a sign you hold in your hand and show only when you need to.

### Purposes

- To redirect users conditionally (e.g., auth guards).
- To navigate programmatically after user actions.
- To replace the current history entry instead of adding a new one.
- To redirect from loaders and actions in data routers.

### Syntax Rules and Structure

**Declarative Redirect:**
```jsx
import { Navigate } from "react-router";

function ProtectedRoute({ isLoggedIn, children }) {
  if (!isLoggedIn) {
    return <Navigate to="/login" replace />;
  }
  return children;
}
```

**Programmatic Redirect:**
```jsx
import { useNavigate } from "react-router";

function LoginForm() {
  const navigate = useNavigate();

  async function handleSubmit() {
    await login();
    navigate("/dashboard", { replace: true });
  }

  return <form onSubmit={handleSubmit}>...</form>;
}
```

**Redirect from Loader:**
```jsx
import { redirect } from "react-router";

async function loader({ request }) {
  const user = await getUser();
  if (!user) return redirect("/login");
  return { user };
}
```

**Syntax Rules:**
- Use `<Navigate>` for conditional redirects during render.
- Use `useNavigate` for redirects after user actions.
- Use `replace: true` to avoid adding to the history stack.
- Use `redirect()` in loaders and actions for data routers.

**Constraints and Limitations:**
- `<Navigate>` must be rendered during render; calling it in a callback has no effect.
- `useNavigate` causes an extra render.
- External URLs should use `window.location.href` instead of React Router navigation.

### Annotated Code Example: Auth Guard with Navigate

```jsx
import { Routes, Route, Navigate } from "react-router";

function ProtectedRoute({ isLoggedIn, children }) {
  if (!isLoggedIn) {
    return <Navigate to="/login" replace />;
  }
  return children;
}

export default function App() {
  const [isLoggedIn, setIsLoggedIn] = React.useState(false);

  return (
    <Routes>
      <Route path="/login" element={<button onClick={() => setIsLoggedIn(true)}>Login</button>} />
      <Route
        path="/dashboard"
        element={
          <ProtectedRoute isLoggedIn={isLoggedIn}>
            <h1>Dashboard</h1>
          </ProtectedRoute>
        }
      />
    </Routes>
  );
}
```

**Expected Output:** Navigating to `/dashboard` while not logged in redirects to `/login`. Clicking "Login" sets `isLoggedIn` to true, and navigating to `/dashboard` now shows the dashboard.

**Why This Output Occurs:** The `ProtectedRoute` component checks `isLoggedIn` and returns `<Navigate to="/login" replace />` if false. The `replace` prop replaces the dashboard entry in history, so the Back button doesn't return to the protected page.

### References

- React Router – Navigate: https://reactrouter.com/6.30.3/components/navigate
- React Router – useNavigate: https://reactrouter.com/6.30.3/hooks/use-navigate
- CoreUI – How to redirect in React Router: https://coreui.io/answers/how-to-redirect-in-react-router/

---

## Core Concept 9: Route Layouts and Presentation Wrappers

### Definitions

**Core Definition:** Layout routes (also called "pathless routes") are routes that provide shared UI structure for child routes without adding segments to the URL, using the `layout()` helper or a pathless route configuration.

**Technical Definition:** Layout routes are defined with `layout("routes/app-layout.tsx", [ ...children ])` or as a route without a `path` property. They render shared UI (headers, sidebars, footers) and an `<Outlet />` where child routes appear. Multiple layout routes can be used for different sections of an app (marketing layout, app layout, auth layout). Layout routes can be nested for hierarchical UI structures.

**Beginner-Friendly Explanation:** A layout route is like a picture frame. The frame (header, sidebar, footer) stays the same, but the picture inside (the child route content) changes. You can have different frames for different sections of your app—a marketing frame, a dashboard frame, a login frame—and each frame wraps its own set of pages.

### Purposes

- To share UI structure (headers, sidebars, footers) across multiple child routes.
- To avoid duplicating layout code across pages.
- To create different layouts for different app sections.
- To nest layouts for hierarchical UI structures.

### Syntax Rules and Structure

**Layout Route with `layout()` Helper:**
```tsx
// app/routes.ts
import { layout, route } from "@react-router/dev/routes";

export default [
  layout("routes/app-layout.tsx", [
    route("dashboard", "routes/dashboard.tsx"),
    route("profile", "routes/profile.tsx"),
    route("settings", "routes/settings.tsx"),
  ]),
];
```

**Layout Component:**
```tsx
// routes/app-layout.tsx
import { Outlet, Link } from "react-router";

export default function AppLayout() {
  return (
    <div className="app">
      <header>
        <nav>
          <Link to="/dashboard">Dashboard</Link>
          <Link to="/profile">Profile</Link>
          <Link to="/settings">Settings</Link>
        </nav>
      </header>
      <main>
        <Outlet />
      </main>
      <footer>
        <p>© 2024 My App</p>
      </footer>
    </div>
  );
}
```

**Pathless Route (Object Config):**
```jsx
{
  element: <AppLayout />,
  children: [
    { path: "dashboard", element: <Dashboard /> },
    { path: "profile", element: <Profile /> },
  ],
}
```

**Syntax Rules:**
- Use `layout()` helper or a route without `path` for layout routes.
- The layout component must render `<Outlet />` for child routes.
- Layout routes do not add segments to the URL.
- Multiple layout routes can be defined for different app sections.
- Nested layouts create hierarchical UI structures.

**Constraints and Limitations:**
- A layout route without `<Outlet />` will not render child routes.
- Layout routes add a level of nesting; deeply nested layouts can be hard to trace.
- The `layout()` helper is specific to the framework mode; object config uses a pathless route.

### Annotated Code Example: Marketing and App Layouts

```jsx
import { Routes, Route, Outlet, Link } from "react-router";

function MarketingLayout() {
  return (
    <div>
      <header><h1>My Company</h1></header>
      <nav>
        <Link to="/">Home</Link>
        <Link to="/about">About</Link>
      </nav>
      <Outlet />
    </div>
  );
}

function AppLayout() {
  return (
    <div>
      <header><h1>App Dashboard</h1></header>
      <nav>
        <Link to="/dashboard">Dashboard</Link>
        <Link to="/settings">Settings</Link>
      </nav>
      <Outlet />
    </div>
  );
}

export default function App() {
  return (
    <Routes>
      <Route element={<MarketingLayout />}>
        <Route path="/" element={<h2>Welcome</h2>} />
        <Route path="/about" element={<h2>About Us</h2>} />
      </Route>
      <Route element={<AppLayout />}>
        <Route path="/dashboard" element={<h2>Dashboard Content</h2>} />
        <Route path="/settings" element={<h2>Settings</h2>} />
      </Route>
    </Routes>
  );
}
```

**Expected Output:** Navigating to `/` shows the marketing header with "Welcome". Navigating to `/dashboard` shows the app header with "Dashboard Content". The layouts are different for different sections.

**Why This Output Occurs:** The pathless `<Route element={<MarketingLayout />}>` wraps the marketing routes, and `<Route element={<AppLayout />}>` wraps the app routes. Each layout renders its own `<Outlet />`, and the matching child route renders inside.

### References

- React Router – Layout Routes: https://mintlify.wiki/remix-run/react-router/routing/layout-routes
- Stack Overflow – Layout Routes in createBrowserRouter: https://stackoverflow.com/questions/75773395
- React Router – Pathless Routes: https://raw.githubusercontent.com/remix-run/react-router/main/docs

---

## Comparison and Decision Guidance

| Feature | `<BrowserRouter>` (Declarative) | `createBrowserRouter` (Data) |
|---|---|---|
| **Setup** | Wrap app in `<BrowserRouter>` | `createBrowserRouter` + `RouterProvider` |
| **Route Definition** | JSX `<Routes>` / `<Route>` | Route objects |
| **Data Loading** | No | Yes (`loader`, `action`, `middleware`) |
| **Error Boundaries** | Manual | Route-level `errorElement` |
| **Blocking** | No | Yes (`useBlocker`) |
| **Recommended For** | Simple apps, quick prototypes | Data-driven apps, production |

**Decision Guidance:**
- **Use `createBrowserRouter`** for most new applications that need data loading, error handling, or blocking.
- **Use `<BrowserRouter>`** for simple apps or when migrating from older React Router versions.
- **Use layout routes** to share UI across multiple pages without duplicating code.
- **Use `useSearchParams`** for filter, sort, and pagination state that should be shareable.
- **Use `<Navigate>`** for auth guards and conditional redirects.
- **Use `useNavigate`** for redirects after form submissions or user actions.

---

## References

- React Router v7 – Installation: https://reactrouter.com/7.1.0/start/library/installation
- React Router – Route Object: https://reactrouter.com/start/data/route-object
- React Router – useParams: https://reactrouter.com/6.30.3/hooks/use-params
- React Router – Outlet: https://reactrouter.com/6.30.3/components/outlet
- React Router – useSearchParams: https://api.reactrouter.com/v7/functions/react-router.useSearchParams.html
- React Router – useNavigate: https://reactrouter.com/api/hooks/useNavigate
- React Router – Navigate: https://reactrouter.com/6.30.3/components/navigate
- React Router – Layout Routes: https://mintlify.wiki/remix-run/react-router/routing/layout-routes
- React Router – Nested Routes: https://mintlify.wiki/remix-run/react-router/routing/nested-routes
- React Router – useRoutes: https://reactrouter.com/api/hooks/useRoutes
- React Router – createRoutesFromElements: https://reactrouter.com/api/utils/createRoutesFromElements
- React Router – Link: https://reactrouter.com/api/components/Link
- React Router – NavLink: https://reactrouter.com/api/components/NavLink
- React Router – useLocation: https://reactrouter.com/api/hooks/useLocation
- React Router – Accessibility: https://reactrouter.com/8.4.0/how-to/accessibility
- CoreUI – How to redirect in React Router: https://coreui.io/answers/how-to-redirect-in-react-router/
- Stack Overflow – Layout Routes in createBrowserRouter: https://stackoverflow.com/questions/75773395
- Stack Overflow – Dynamic Routes with Catch-All: https://stackoverflow.com/questions/79671894
- Uniflow – React Router v6 to v7 Migration Guide: https://www.uniflow.kr