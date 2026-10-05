# React Advanced Routing: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** React Advanced Routing encompasses the patterns, APIs, and techniques used to build production-grade routing in React applications, including route protection, authentication guards, error handling, code splitting, data loading, mutations, nested layouts, and URL state management.

**Technical Definition:** React Advanced Routing extends the foundational routing concepts of React Router v7 with data-aware, security-conscious, and performance-optimised patterns. It leverages the data router (`createBrowserRouter`) to support route-level `loader` functions (which fetch data before rendering), `action` functions (which handle mutations), `errorElement` properties (which provide route-scoped error boundaries), lazy route modules (which enable code splitting), and hooks like `useLoaderData`, `useActionData`, `useNavigation`, and `useRouteError`. It also covers authentication-aware navigation via protected route components and `redirect()` from loaders, `<Suspense>` integration for lazy-loaded routes, `<ScrollRestoration>` for scroll position management, and advanced URL state synchronisation through `useSearchParams`. These patterns collectively form the architecture of a modern React application that is secure, performant, resilient to errors, and pleasant to use.

**Beginner-Friendly Explanation:** Basic routing is about showing the right page when the user visits a URL. Advanced routing is about all the things that make a real app work: making sure only logged-in users can see certain pages, showing a nice error page instead of a blank screen when something breaks, loading only the code you need for the current page so the app starts fast, fetching data before the page appears so the user never sees a blank loading state, handling form submissions that change server data, keeping the same layout across pages, and remembering where the user was scrolling when they hit the Back button.

### Key Characteristics

- **Route-Level Data Loading:** Loaders fetch data before the component renders, eliminating loading spinners for initial renders.
- **Route-Level Mutations:** Actions handle form submissions and mutations, with `useNavigation` providing pending states for progressive enhancement.
- **Authentication-Aware Navigation:** Protected routes redirect unauthenticated users to login, preserving the intended destination via `state` or query params.
- **Error Resilience:** `errorElement` provides route-scoped Error Boundaries, with `useRouteError` accessing the thrown error.
- **Code Splitting:** `lazy` route properties load route modules on demand, reducing initial bundle size.
- **Progressive Enhancement:** `<Form>` works without JavaScript when paired with server-side actions.
- **Layout Inheritance:** Nested layouts share UI chrome across child routes without duplicating code.
- **URL as State:** Query params and path segments encode application state, enabling shareable and bookmarkable URLs.
- **Scroll Restoration:** `<ScrollRestoration>` restores scroll position on back/forward navigation.

### Prerequisites

- Solid understanding of React function components, JSX, and Hooks.
- Working knowledge of React Router v7 basics (routes, `<Link>`, `<Outlet>`, `useParams`).
- Familiarity with `useState`, `useEffect`, and asynchronous JavaScript (promises, `async`/`await`).
- Basic understanding of React Suspense and Error Boundaries.
- Awareness of HTTP methods, form handling, and authentication concepts.

### Related Programming Areas

- **Authentication and Authorization:** Protecting routes and managing user sessions.
- **Performance Optimisation:** Code splitting, lazy loading, and bundle size reduction.
- **Data Fetching:** Server-state management with route loaders.
- **Error Handling:** Route-level error boundaries and 404 pages.
- **User Experience:** Loading states, optimistic UI, and scroll restoration.

### Core Concepts / Features

1. Protected Routes and Conditional Rendering
2. Authentication-Aware Navigation and Route Guards
3. Error Boundaries, Error Elements, and 404 Handling
4. Lazy-Loaded Routes, Code Splitting, and Suspense Integration
5. Route-Level Data Loading and Mutations (Loaders, Actions, `<Form>`)
6. Data Hooks (`useLoaderData`, `useActionData`, `useNavigation`)
7. Nested Layouts and Layout-Level Inheritance
8. Advanced URL State Management and Scroll Restoration

---

## Core Concept 1: Protected Routes and Conditional Rendering

### Definitions

**Core Definition:** Protected routes are routes that render their content only when a specific condition is met (typically authentication), and redirect or render an alternative UI when the condition is not satisfied.

**Technical Definition:** Protected routes in React Router v7 are implemented through conditional rendering combined with `<Navigate>` (for declarative redirects) or `redirect()` in loaders (for data-router redirects). A wrapper component (often called `ProtectedRoute` or `RequireAuth`) checks the authentication condition and either renders its `children` (or `<Outlet />` for layout-based protection) or returns a `<Navigate to="/login" />`. With the data router, the check is performed inside a `loader` function, which is the recommended approach because it runs before rendering and avoids a flash of unauthenticated content. The loader can access the `request` object, cookies, and any authentication context, and returns `redirect('/login')` if the user is not authenticated.

**Beginner-Friendly Explanation:** A protected route is like a locked door with a guard. When you try to enter, the guard checks your ID. If you have the right ID (you're logged in), the door opens and you see the content. If not, the guard sends you to the login page instead. The guard doesn't ask you for your ID after you're inside—it checks before you enter, so you never see the protected content without permission.

### Purposes

- To restrict access to routes based on authentication status.
- To restrict access based on user roles or permissions.
- To redirect unauthenticated users to the login page while preserving their intended destination.
- To avoid flashes of protected content before redirects occur.
- To centralise access-control logic in a single wrapper or loader.

### Syntax Rules and Structure

**Declarative Protected Route (Wrapper Component):**
```jsx
import { Navigate, Outlet } from "react-router";

function ProtectedRoute({ isAuthenticated, children }) {
  if (!isAuthenticated) {
    return <Navigate to="/login" replace />;
  }
  return children ?? <Outlet />;
}

// Usage in route config
<Route element={<ProtectedRoute isAuthenticated={isLoggedIn} />}>
  <Route path="/dashboard" element={<Dashboard />} />
  <Route path="/settings" element={<Settings />} />
</Route>
```

**Component Breakdown:**
- `ProtectedRoute`: Wrapper component that checks authentication.
- `children ?? <Outlet />`: Renders either explicit children or nested routes.
- `<Navigate to="/login" replace />`: Redirects unauthenticated users.

**Data Router Loader Guard (Recommended):**
```jsx
import { redirect } from "react-router";

async function protectedLoader({ request }) {
  const user = await getUser();
  if (!user) {
    const url = new URL(request.url);
    return redirect(`/login?from=${url.pathname}`);
  }
  return { user };
}

const router = createBrowserRouter([
  {
    path: "/dashboard",
    loader: protectedLoader,
    element: <Dashboard />,
  },
]);
```

**Component Breakdown:**
- `protectedLoader`: Runs before the route renders.
- `redirect('/login?from=...')`: Redirects and preserves the intended destination.
- The component never renders if the user is not authenticated.

**Role-Based Protection:**
```jsx
function RoleProtectedRoute({ user, requiredRole, children }) {
  if (!user) return <Navigate to="/login" replace />;
  if (!user.roles.includes(requiredRole)) return <Navigate to="/forbidden" replace />;
  return children;
}
```

**Syntax Rules:**
- Prefer loader-based guards with the data router; they run before rendering and avoid content flashes.
- Use `<Navigate>` with `replace` for declarative redirects.
- Preserve the intended destination via `?from=` query param or `state` for post-login redirects.
- Use `<Outlet />` in layout-based protected routes to render nested protected routes.
- Check roles and permissions after authentication; never trust client-only checks for security-sensitive data.

**Constraints and Limitations:**
- Client-side route protection does not secure data; the server must also validate authentication and authorisation.
- Wrapper components re-render whenever authentication state changes; memoise if necessary.
- `Navigate` with `replace` avoids adding the protected route to history, but the user can still press Back to the login page.
- Loader guards cannot access React context; they receive the `request` object and any values from `context` passed to `RouterProvider`.

### Annotated Code Example: Auth Guard with Post-Login Redirect

```jsx
import { createBrowserRouter, RouterProvider, Navigate, Outlet, useLocation, useLoaderData } from "react-router";

// Simple auth state (in a real app, use Context or a store)
let currentUser = null;

function useAuth() {
  return { user: currentUser, login: (u) => { currentUser = u; } };
}

// Loader guard
async function requireAuth() {
  if (!currentUser) {
    return redirect("/login");
  }
  return { user: currentUser };
}

// Login page
function Login() {
  const { login } = useAuth();
  const location = useLocation();
  const params = new URLSearchParams(location.search);
  const from = params.get("from") || "/dashboard";

  function handleLogin() {
    login({ name: "Alice" });
    // Navigate to the intended destination
    window.history.pushState({}, "", from);
  }

  return (
    <div>
      <h1>Login</h1>
      <button onClick={handleLogin}>Log In as Alice</button>
    </div>
  );
}

// Protected dashboard
function Dashboard() {
  const { user } = useLoaderData();
  return <h1>Welcome, {user.name}!</h1>;
}

// Route config
const router = createBrowserRouter([
  {
    path: "/login",
    element: <Login />,
  },
  {
    path: "/",
    loader: requireAuth,
    element: <Outlet />,
    children: [
      { path: "dashboard", element: <Dashboard /> },
    ],
  },
]);

export default function App() {
  return <RouterProvider router={router} />;
}
```

**Expected Output:** Navigating to `/dashboard` without being logged in redirects to `/login?from=/dashboard`. Clicking "Log In as Alice" logs the user in and navigates to `/dashboard`, showing "Welcome, Alice!".

**Why This Output Occurs:** The `requireAuth` loader runs before the `/dashboard` route renders. If `currentUser` is `null`, it returns `redirect('/login')`. The `Login` component reads the `from` query param and navigates there after login. The `Dashboard` component reads the user from `useLoaderData`.

### Real-World Cases

- **SaaS dashboards:** Only authenticated users can access the dashboard; unauthenticated users are redirected to login.
- **Admin panels:** Only users with the `admin` role can access admin routes.
- **E-commerce checkout:** Only authenticated users can access checkout; guests are redirected to login or guest checkout.
- **Multi-tenant apps:** Users can only access routes for their own tenant.

### References

- React Router – redirect: https://reactrouter.com/api/data-routers/redirect
- React Router – Navigate: https://reactrouter.com/6.30.3/components/navigate
- TanStack Query – Authenticated Routes: https://tanstack.com/router/latest/docs/framework/react/guide/authenticated-routes

---

## Core Concept 2: Authentication-Aware Navigation and Route Guards

### Definitions

**Core Definition:** Authentication-aware navigation is the practice of adapting the navigation UI (links, menus, redirects) based on the user's authentication state and permissions.

**Technical Definition:** Authentication-aware navigation combines route guards (which protect routes) with conditional UI rendering (which hides or shows navigation elements based on auth state). The authentication state is typically stored in a Context or external store and consumed by navigation components (`<Navbar>`, `<Sidebar>`) to conditionally render login/logout links, user avatars, and role-gated menu items. When authentication state changes (login, logout, token expiry), the navigation must update immediately, and any protected routes the user is currently viewing must redirect. React Router's `useRevalidator` can be used to re-run loaders after authentication changes, and `useNavigate` can be used to redirect programmatically on logout. The pattern also includes preserving the user's intended destination across the login flow, typically via a `from` query parameter or `state` object.

**Beginner-Friendly Explanation:** Authentication-aware navigation is like a building where the signs and doors change based on whether you have a badge. When you're not logged in, you see "Login" signs. When you log in, the signs change to "Profile," "Settings," and "Logout." If you're an admin, you also see "Admin Panel." The navigation adapts to who you are and what you're allowed to do.

### Purposes

- To show or hide navigation elements based on authentication status.
- To display user-specific information (avatar, name) in the navigation.
- To redirect users on logout from protected routes.
- To re-run loaders after authentication changes so the UI reflects the new user.
- To preserve the user's intended destination across the login flow.

### Syntax Rules and Structure

**Auth Context:**
```jsx
import { createContext, useContext, useState, useMemo } from "react";

const AuthContext = createContext(null);

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const value = useMemo(() => ({ user, setUser }), [user]);
  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
}

export function useAuth() {
  return useContext(AuthContext);
}
```

**Auth-Aware Navigation:**
```jsx
import { Link, useNavigate } from "react-router";
import { useAuth } from "./AuthContext";

function Navbar() {
  const { user, setUser } = useAuth();
  const navigate = useNavigate();

  function handleLogout() {
    setUser(null);
    navigate("/login", { replace: true });
  }

  return (
    <nav>
      <Link to="/">Home</Link>
      {user ? (
        <>
          <Link to="/dashboard">Dashboard</Link>
          <span>Hello, {user.name}</span>
          <button onClick={handleLogout}>Logout</button>
        </>
      ) : (
        <Link to="/login">Login</Link>
      )}
    </nav>
  );
}
```

**Revalidating Loaders After Auth Change:**
```jsx
import { useRevalidator } from "react-router";

function useAuthSync() {
  const { user } = useAuth();
  const revalidator = useRevalidator();

  // Re-run all loaders when the user changes
  useEffect(() => {
    revalidator.revalidate();
  }, [user, revalidator]);
}
```

**Syntax Rules:**
- Store authentication state in a Context or external store; do not duplicate it in route components.
- Use `useAuth()` to conditionally render navigation elements.
- On logout, navigate with `replace: true` to prevent the Back button from returning to protected content.
- Use `useRevalidator()` to re-run loaders after login/logout so the UI reflects the new user.
- Preserve the intended destination via `?from=` query params or `state` for post-login redirects.
- Server-side validation is mandatory; client-side guards are for UX only.

**Constraints and Limitations:**
- Auth state in Context causes all consumers to re-render when the user changes; split auth state from auth actions if performance becomes an issue.
- `useRevalidator()` re-runs all active loaders, which can cause multiple simultaneous requests.
- Token expiry must be handled explicitly; a common pattern is to check token validity in a loader and redirect to login.
- Client-side auth state can be tampered with; never trust it for security decisions.

### Annotated Code Example: Login/Logout with Auth-Aware Navigation

Folder Structure
```
src/
├── context/
│   └── AuthContext.jsx
├── components/
│   └── Navbar.jsx
├── layouts/
│   └── RootLayout.jsx
├── pages/
│   ├── Home.jsx
│   ├── Login.jsx
│   └── Dashboard.jsx
├── router.jsx
└── App.jsx
```
\
1. `src/context/AuthContext.jsx`
```jsx
import { createContext, useContext, useState, useMemo } from "react";

const AuthContext = createContext(null);

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const value = useMemo(() => ({ user, setUser }), [user]);
  
  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
}

export function useAuth() {
  return useContext(AuthContext);
}
```
\
2. `src/components/Navbar.jsx`
```jsx
import { Link, useNavigate, useRevalidator } from "react-router";
import { useAuth } from "../context/AuthContext";

export function Navbar() {
  const { user, setUser } = useAuth();
  const navigate = useNavigate();
  const revalidator = useRevalidator();

  function handleLogout() {
    setUser(null);
    revalidator.revalidate();
    navigate("/login", { replace: true });
  }

  return (
    <nav>
      <Link to="/">Home</Link>
      {user ? (
        <>
          <Link to="/dashboard">Dashboard</Link>
          <span>Hello, {user.name}</span>
          <button onClick={handleLogout}>Logout</button>
        </>
      ) : (
        <Link to="/login">Login</Link>
      )}
    </nav>
  );
}
```
\
3. `src/pages/Home.jsx`
```jsx
export default function Home() {
  return <h1>Home</h1>;
}
```
\
4. `src/pages/Login.jsx`
```jsx
import { useNavigate, useRevalidator } from "react-router";
import { useAuth } from "../context/AuthContext";

export default function Login() {
  const { setUser } = useAuth();
  const navigate = useNavigate();
  const revalidator = useRevalidator();

  function handleLogin() {
    setUser({ name: "Alice" });
    revalidator.revalidate();
    navigate("/dashboard");
  }

  return (
    <div>
      <h1>Login</h1>
      <button onClick={handleLogin}>Log In</button>
    </div>
  );
}
```
\
5. `src/pages/Dashboard.jsx`
```jsx
import { useAuth } from "../context/AuthContext";

export default function Dashboard() {
  const { user } = useAuth();
  return <h1>Dashboard for {user?.name}</h1>;
}
```
\
6. `src/layouts/RootLayout.jsx`
```jsx
import { Outlet } from "react-router";
import { AuthProvider } from "../context/AuthContext";
import { Navbar } from "../components/Navbar";

export default function RootLayout() {
  return (
    <AuthProvider>
      <Navbar />
      <main>
        <Outlet />
      </main>
    </AuthProvider>
  );
}
```
\
7. `src/router.jsx`
```jsx
import { createBrowserRouter } from "react-router";
import RootLayout from "./layouts/RootLayout";
import Home from "./pages/Home";
import Login from "./pages/Login";
import Dashboard from "./pages/Dashboard";

export const router = createBrowserRouter([
  {
    path: "/",
    element: <RootLayout />,
    children: [
      { index: true, element: <Home /> },
      { path: "login", element: <Login /> },
      { path: "dashboard", element: <Dashboard /> },
    ],
  },
]);
```
\
8. `src/App.jsx`
```jsx
import { RouterProvider } from "react-router";
import { router } from "./router";

export default function App() {
  return <RouterProvider router={router} />;
}
```

**Expected Output:** The navbar shows "Home" and "Login" when logged out. Clicking "Log In" sets the user and navigates to `/dashboard`, where the navbar now shows "Dashboard," "Hello, Alice," and "Logout." Clicking "Logout" clears the user and redirects to `/login`.

**Why This Output Occurs:** The `AuthProvider` wraps the entire app. The `Navbar` reads the auth state from `useAuth()` and conditionally renders links. On login, `setUser` updates the context, `revalidator.revalidate()` re-runs loaders, and `navigate('/dashboard')` changes the route. On logout, the user is cleared, loaders are revalidated, and the user is redirected to `/login`.

### Real-World Cases

- **SaaS applications:** Navigation shows user-specific links when logged in, login/signup when logged out.
- **E-commerce:** Navigation shows the cart and account menu when logged in.
- **Admin panels:** Navigation shows admin-specific links only for users with admin roles.
- **Multi-tenant apps:** Navigation adapts to the tenant the user belongs to.

### References

- React Router – useRevalidator: https://reactrouter.com/api/hooks/useRevalidator
- React Router – useNavigate: https://reactrouter.com/api/hooks/useNavigate
- React Context – Passing Data Deeply: https://react.dev/learn/passing-data-deeply-with-context

---

## Core Concept 3: Error Boundaries, Error Elements, and 404 Handling

### Definitions

**Core Definition:** Error boundaries in React Router are route-scoped error handlers defined via the `errorElement` property, which catch errors thrown by loaders, actions, or route components and render a fallback UI instead of the entire app crashing.

**Technical Definition:** When a loader, action, or component throws an error, React Router catches it and renders the nearest `errorElement` in the route hierarchy. The error is accessible inside the `errorElement` component via the `useRouteError` Hook, which returns the thrown error object. React Router distinguishes between `ErrorResponse` instances (which include `status` and `statusText`) and other errors. The default error element renders a generic error page, but custom `errorElement` components can render tailored UIs for 404s, 401s, 500s, and thrown exceptions. A catch-all route (`path: "*"`) handles unmatched URLs with a 404 page. In the data router, errors in `loader` or `action` functions are caught by the nearest `errorElement`, and the error is not thrown to the browser's console (unless re-thrown).

**Beginner-Friendly Explanation:** An error element is like a safety net. If something goes wrong while loading a page—the server is down, the data is missing, or the code has a bug—the safety net catches the error and shows a friendly message instead of a blank screen. The user sees "Something went wrong" instead of a broken app. A 404 page is a special kind of safety net that catches URLs that don't match any route.

### Purposes

- To catch errors from loaders, actions, and components at the route level.
- To render user-friendly error pages instead of blank screens or crashes.
- To handle 404 (Not Found) errors with a dedicated catch-all route.
- To handle different error types (401, 403, 500) with tailored UIs.
- To log errors for monitoring without exposing technical details to users.
- To allow users to recover (retry, go home) from error states.

### Syntax Rules and Structure

**Route-Level errorElement:**
```jsx
const router = createBrowserRouter([
  {
    path: "/",
    element: <RootLayout />,
    errorElement: <RootErrorBoundary />, // Catches errors from all children
    children: [
      {
        path: "dashboard",
        element: <Dashboard />,
        loader: dashboardLoader,
        errorElement: <DashboardError />, // Nested error boundary
      },
    ],
  },
]);
```

**Using useRouteError:**
```jsx
import { useRouteError, isRouteErrorResponse, Link } from "react-router";

function RootErrorBoundary() {
  const error = useRouteError();

  if (isRouteErrorResponse(error)) {
    if (error.status === 404) {
      return <div><h1>404 — Not Found</h1><Link to="/">Go Home</Link></div>;
    }
    if (error.status === 401) {
      return <div><h1>Unauthorized</h1><Link to="/login">Log In</Link></div>;
    }
    return <div><h1>{error.status} — {error.statusText}</h1></div>;
  }

  return <div><h1>Something went wrong</h1><p>{error.message}</p></div>;
}
```

**404 Catch-All:**
```jsx
const router = createBrowserRouter([
  // ...other routes
  { path: "*", element: <NotFound /> },
]);
```

**Throwing Errors from Loaders:**
```jsx
async function dashboardLoader() {
  const res = await fetch("/api/dashboard");
  if (res.status === 404) {
    throw new Response("Not Found", { status: 404, statusText: "Not Found" });
  }
  if (!res.ok) {
    throw new Response("Server Error", { status: 500 });
  }
  return res.json();
}
```

**Syntax Rules:**
- Add `errorElement` to any route to catch errors from that route and its children.
- Use `useRouteError()` to access the thrown error.
- Use `isRouteErrorResponse(error)` to distinguish `Response` errors from thrown exceptions.
- Use `path: "*"` for a catch-all 404 route.
- Throw `Response` objects from loaders and actions to set HTTP status codes.
- Nest `errorElement` at different levels for granular error handling.

**Constraints and Limitations:**
- `errorElement` catches errors from loaders, actions, and rendering, but not from event handlers or async code outside those boundaries.
- A single top-level `errorElement` is an anti-pattern; granular error elements provide better UX.
- `useRouteError` returns `unknown`; you must narrow the type with `isRouteErrorResponse` or type guards.
- Errors thrown in `errorElement` itself are not caught by the same boundary; they propagate to the parent.

### Annotated Code Example: Root and Nested Error Boundaries

```jsx
import { createBrowserRouter, RouterProvider, useRouteError, isRouteErrorResponse, Link, Outlet } from "react-router";

// Root error boundary
function RootError() {
  const error = useRouteError();

  if (isRouteErrorResponse(error)) {
    return (
      <div>
        <h1>{error.status} — {error.statusText}</h1>
        {error.status === 404 && <Link to="/">Go Home</Link>}
      </div>
    );
  }

  return (
    <div>
      <h1>Something went wrong</h1>
      <p>{error?.message || "Unknown error"}</p>
    </div>
  );
}

// Dashboard error boundary (nested)
function DashboardError() {
  const error = useRouteError();
  return (
    <div>
      <h2>Dashboard failed to load</h2>
      <p>{error?.message}</p>
      <Link to="/">Back to safety</Link>
    </div>
  );
}

// Loader that may throw
async function dashboardLoader() {
  const res = await fetch("/api/dashboard");
  if (!res.ok) {
    throw new Response("Dashboard unavailable", { status: 503 });
  }
  return res.json();
}

const router = createBrowserRouter([
  {
    path: "/",
    element: <div><h1>App</h1><Outlet /></div>,
    errorElement: <RootError />,
    children: [
      { index: true, element: <h2>Home</h2> },
      {
        path: "dashboard",
        loader: dashboardLoader,
        element: <h2>Dashboard</h2>,
        errorElement: <DashboardError />,
      },
    ],
  },
]);

export default function App() {
  return <RouterProvider router={router} />;
}
```

**Expected Output:** Navigating to `/dashboard` when the API returns an error shows the `DashboardError` UI ("Dashboard failed to load") instead of crashing the app. Navigating to an unknown path shows the `RootError` UI with a 404 message.

**Why This Output Occurs:** The `dashboardLoader` throws a `Response` with status 503, which React Router catches and passes to the nearest `errorElement`—in this case, `DashboardError`. The rest of the app (root layout, home route) remains functional. A 404 from an unmatched URL is caught by the root `errorElement`.

### Real-World Cases

- **E-commerce:** Product detail page fails to load → nested error boundary shows "Product unavailable" with a link back to the catalogue.
- **Dashboard:** One widget fails → only that widget's error boundary triggers, not the entire dashboard.
- **API outages:** Global error boundary shows "Service temporarily unavailable" with a retry button.
- **Unauthorised access:** 401 error element redirects to login.

### References

- React Router – Error Boundaries: https://reactrouter.com/7.1.0/start/data/error-boundaries
- React Router – useRouteError: https://reactrouter.com/api/hooks/useRouteError
- React Router – isRouteErrorResponse: https://reactrouter.com/api/utils/isRouteErrorResponse
- React – Error Boundaries: https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary

---

## Core Concept 4: Lazy-Loaded Routes, Code Splitting, and Suspense Integration

### Definitions

**Core Definition:** Lazy-loaded routes are route modules that are loaded on demand (when the user navigates to them) rather than in the initial bundle, using React's `lazy()` and React Router's route-level `lazy` property, with `<Suspense>` providing fallback UI during loading.

**Technical Definition:** React Router v7 supports route-level lazy loading through the `lazy` property on route objects. The `lazy` property accepts a function that returns a promise resolving to a route module with `Component`, `loader`, `action`, and other exports. React Router handles the loading and integration of these modules automatically, eliminating the need for manual `React.lazy()` and `<Suspense>` wrappers in many cases. For component-level splitting (not route-level), React's `React.lazy()` combined with `<Suspense>` can be used. In the data router, lazy routes can also define their own `loader` and `action` functions, which are loaded alongside the component. The pattern reduces initial bundle size by deferring code for rarely-visited routes, improving Time to Interactive (TTI) and Largest Contentful Paint (LCP).

**Beginner-Friendly Explanation:** Lazy loading is like packing for a trip. Instead of carrying everything you own in one giant suitcase (the initial bundle), you pack only what you need for the first part of the trip. When you get to the next destination, you pick up another suitcase with the things you need there. React Router's lazy loading does the same for code: it loads only the code for the current page, and when you navigate to a new page, it fetches that page's code on demand.

### Purposes

- To reduce initial bundle size by splitting route modules into separate chunks.
- To improve Time to Interactive (TTI) and First Contentful Paint (FCP).
- To load route-specific loaders and actions alongside the component.
- To provide fallback UI during lazy loading with `<Suspense>`.
- To support per-route error boundaries that catch lazy-loading failures.

### Syntax Rules and Structure

**Route-Level Lazy (Data Router):**
```jsx
const router = createBrowserRouter([
  {
    path: "/",
    lazy: async () => {
      const module = await import("./routes/home");
      return { Component: module.default, loader: module.loader };
    },
  },
]);
```

**Lazy Module with Named Exports:**
```jsx
// routes/dashboard.tsx
export function loader() {
  return fetchDashboardData();
}

export function Component() {
  const data = useLoaderData();
  return <Dashboard data={data} />;
}
```

**Route Config with Lazy:**
```jsx
const router = createBrowserRouter([
  {
    path: "/dashboard",
    lazy: () => import("./routes/dashboard"),
  },
]);
```

**Component-Level Lazy with Suspense:**
```jsx
import { lazy, Suspense } from "react";

const HeavyChart = lazy(() => import("./HeavyChart"));

function Dashboard() {
  return (
    <Suspense fallback={<ChartSkeleton />}>
      <HeavyChart />
    </Suspense>
  );
}
```

**Syntax Rules:**
- Use the route object's `lazy` property for route-level code splitting.
- The `lazy` function must return a promise resolving to an object with route properties (`Component`, `loader`, `action`, `ErrorBoundary`, etc.).
- Lazy routes can define their own `loader` and `action` functions that are loaded with the component.
- Use `<Suspense>` for component-level lazy loading (not route-level).
- Provide meaningful fallback UIs (skeletons) rather than generic spinners.

**Constraints and Limitations:**
- Lazy loading adds a network round trip when navigating to a lazy route; preload critical routes if possible.
- Suspense fallbacks cause layout shift if they do not match the final content dimensions.
- Lazy routes' loaders are only available after the module loads; dependent queries must be gated.
- In development, lazy loading may behave differently than in production; test both.

### Annotated Code Example: Lazy-Loaded Dashboard with Loader

```jsx
// routes/dashboard.tsx
import { useLoaderData } from "react-router";

export async function loader() {
  const res = await fetch("/api/dashboard");
  if (!res.ok) throw new Response("Failed", { status: 500 });
  return res.json();
}

export function Component() {
  const data = useLoaderData();
  return (
    <div>
      <h1>Dashboard</h1>
      <pre>{JSON.stringify(data, null, 2)}</pre>
    </div>
  );
}
```

```jsx
// app.tsx
import { createBrowserRouter, RouterProvider, Suspense } from "react-router";
import { lazy } from "react";

const router = createBrowserRouter([
  {
    path: "/",
    element: <h1>Home</h1>,
  },
  {
    path: "/dashboard",
    lazy: () => import("./routes/dashboard"),
    HydrateFallback: () => <p>Loading dashboard...</p>,
  },
]);

export default function App() {
  return <RouterProvider router={router} />;
}
```

**Expected Output:** Navigating to `/dashboard` briefly shows "Loading dashboard..." while the module and its data are fetched, then renders the dashboard with its data. The initial bundle does not include the dashboard code.

**Why This Output Occurs:** The `lazy` property tells React Router to fetch the dashboard module when the user navigates to `/dashboard`. React Router loads the module, runs its `loader`, and renders the `Component`. The `HydrateFallback` shows during the initial load.

### Real-World Cases

- **Admin panels:** Heavy admin routes are lazy-loaded so regular users never download admin code.
- **Marketing sites:** The blog or documentation section is lazy-loaded from the main landing page.
- **E-commerce:** The checkout flow is lazy-loaded to keep the product listing fast.
- **Multi-tenant apps:** Tenant-specific routes are lazy-loaded based on the tenant.

### References

- React Router – Lazy Loading: https://reactrouter.com/7.1.0/start/data/lazy-loading
- React Router – lazy Route Property: https://reactrouter.com/api/data-routers/lazy
- React – lazy: https://react.dev/reference/react/lazy
- React – Suspense: https://react.dev/reference/react/Suspense

---

## Core Concept 5: Route-Level Data Loading and Mutations (Loaders, Actions, and Form Components)

### Definitions

**Core Definition:** Loaders are route-level functions that fetch data before a route renders, while actions are route-level functions that handle mutations (form submissions) and return data or redirects; `<Form>` is a React Router component that submits data to actions.

**Technical Definition:** A `loader` is a function that runs before a route's component renders, receiving `{ request, params, context }` and returning data (or throwing a `Response`/`redirect`). The data is accessible via `useLoaderData`. An `action` is a function that runs when a form is submitted to a route, receiving `{ request, params, context }` and returning data (accessible via `useActionData`), a `redirect()`, or throwing a `Response`. `<Form>` is a React Router wrapper around the native `<form>` that submits to the nearest route's `action`; without JavaScript, the form submits to the server and the server-side route handler runs. `useNavigation` provides the form's submission state (`idle`, `submitting`, `loading`). Loaders and actions run in parallel where possible and are revalidated automatically after actions complete. They can be defined in the route module alongside the component, enabling code splitting.

**Beginner-Friendly Explanation:** A loader is like a waiter who takes your order to the kitchen and brings your food before you sit down—you never see an empty table. An action is like a form you fill out and hand to the waiter; the kitchen processes it and either brings back a confirmation or an error. `<Form>` is the piece of paper you write on, and `useNavigation` tells you whether the waiter is still in the kitchen.

### Purposes

- To fetch data before rendering, eliminating loading spinners for initial loads.
- To handle form submissions with server-side actions and progressive enhancement.
- To revalidate data automatically after actions complete.
- To provide submission state (pending, submitting) for UI feedback.
- To enable code-split loaders and actions alongside components.

### Syntax Rules and Structure

**Loader:**
```jsx
export async function loader({ request, params, context }) {
  const url = new URL(request.url);
  const page = url.searchParams.get("page") || "1";
  const res = await fetch(`/api/items?page=${page}`);
  if (!res.ok) throw new Response("Failed", { status: res.status });
  return res.json();
}
```

**Action:**
```jsx
export async function action({ request }) {
  const formData = await request.formData();
  const title = formData.get("title");
  if (!title) {
    return { error: "Title is required" };
  }
  await createTodo({ title });
  return redirect("/todos");
}
```

**Component Using useLoaderData and useNavigation:**
```jsx
import { Form, useLoaderData, useNavigation } from "react-router";

export function Component() {
  const data = useLoaderData();
  const navigation = useNavigation();
  const isSubmitting = navigation.state === "submitting";

  return (
    <div>
      <ul>{data.map(item => <li key={item.id}>{item.title}</li>)}</ul>
      <Form method="post">
        <input name="title" placeholder="New todo" />
        <button type="submit" disabled={isSubmitting}>
          {isSubmitting ? "Adding..." : "Add"}
        </button>
      </Form>
    </div>
  );
}
```

**Syntax Rules:**
- Loaders run on GET requests; actions run on POST, PUT, PATCH, DELETE.
- Throw a `Response` for HTTP errors, not a generic `Error`.
- Return `redirect()` to navigate after an action completes.
- Use `<Form>` (not `<form>`) for progressive enhancement.
- Use `useNavigation().state` for pending UI; `navigation.state` is `"idle"`, `"submitting"`, or `"loading"`.
- Loaders and actions can be defined in the route module alongside the component for code splitting.
- After an action completes, all loaders on the page are revalidated automatically.

**Constraints and Limitations:**
- Loaders and actions cannot access React context; use `context` passed to `RouterProvider` or the `request` object.
- Loaders run in parallel for sibling routes; dependent data must be gated.
- Actions should be idempotent or use idempotency keys if retries are possible.
- `useNavigation` only tracks the current navigation; use `useFetcher` for multiple simultaneous submissions.

### Annotated Code Example: Todo List with Loader and Action

```jsx
// routes/todos.tsx
import { Form, useLoaderData, useActionData, useNavigation, redirect } from "react-router";

export async function loader() {
  const res = await fetch("https://jsonplaceholder.typicode.com/todos?_limit=5");
  if (!res.ok) throw new Response("Failed", { status: res.status });
  return res.json();
}

export async function action({ request }) {
  const formData = await request.formData();
  const title = formData.get("title");
  if (!title || title.trim().length === 0) {
    return { error: "Title is required" };
  }
  // In a real app, POST to the server here
  return { success: true, title: title.trim() };
}

export function Component() {
  const todos = useLoaderData();
  const actionData = useActionData();
  const navigation = useNavigation();
  const isSubmitting = navigation.state === "submitting";

  return (
    <div>
      <h1>Todos</h1>
      {actionData?.error && <p style={{ color: "red" }}>{actionData.error}</p>}
      {actionData?.success && <p>Added: {actionData.title}</p>}

      <ul>
        {todos.map(todo => <li key={todo.id}>{todo.title}</li>)}
      </ul>

      <Form method="post">
        <input name="title" placeholder="New todo title" />
        <button type="submit" disabled={isSubmitting}>
          {isSubmitting ? "Adding..." : "Add Todo"}
        </button>
      </Form>
    </div>
  );
}
```

**Expected Output:** The todo list is displayed with five todos. Submitting the form with an empty title shows "Title is required" in red. Submitting with a title shows "Added: [title]" and the button shows "Adding..." during submission.

**Why This Output Occurs:** The `loader` fetches the todos before the component renders. `useLoaderData` reads them. The `<Form method="post">` submits to the `action`, which validates the title and returns either an error or success object. `useActionData` reads the action's return value. `useNavigation().state` is `"submitting"` during the request, disabling the button.

### Real-World Cases

- **Todo apps:** Load todos via loader, add/toggle/delete via actions.
- **E-commerce:** Load product data via loader, add to cart via action.
- **Blog platforms:** Load posts via loader, create/edit posts via actions.
- **Admin panels:** Load records via loader, create/update/delete via actions.

### References

- React Router – Loaders: https://reactrouter.com/7.1.0/start/data/loaders
- React Router – Actions: https://reactrouter.com/7.1.0/start/data/actions
- React Router – Form: https://reactrouter.com/api/components/Form
- React Router – useNavigation: https://reactrouter.com/api/hooks/useNavigation

---

## Core Concept 6: Data Hooks (useLoaderData, useActionData, useNavigation State)

### Definitions

**Core Definition:** Data hooks are the React Router hooks that expose loader data, action results, and navigation state to route components.

**Technical Definition:** `useLoaderData` returns the data resolved by the nearest route's `loader`. `useActionData` returns the data returned by the nearest route's `action`. `useNavigation` returns a navigation object with `state` (`"idle"`, `"submitting"`, `"loading"`), `location`, `formData`, `json`, and `text`, enabling pending UI. `useFetcher` is a hook for interacting with loaders and actions without navigating. `useRouteLoaderData` reads loader data from a specific route in the hierarchy. `useMatches` returns the current route matches, useful for breadcrumbs and analytics.

**Beginner-Friendly Explanation:** These hooks are how your components get the data that loaders and actions fetched. `useLoaderData` says "give me the data from my loader." `useActionData` says "give me the result from my action." `useNavigation` says "tell me if a navigation or submission is in progress so I can show a loading state." They are the bridge between the route's data functions and the component's UI.

### Purposes

- To read loader data in route components.
- To read action results in route components.
- To show pending UI during navigation and form submission.
- To access loader data from parent routes (`useRouteLoaderData`).
- To interact with loaders and actions without navigating (`useFetcher`).

### Syntax Rules and Structure

**useLoaderData:**
```jsx
const data = useLoaderData(); // Returns the loader's return value
```

**useActionData:**
```jsx
const actionData = useActionData(); // Returns the action's return value (or undefined)
```

**useNavigation:**
```jsx
const navigation = useNavigation();
// navigation.state: "idle" | "submitting" | "loading"
// navigation.location: the location being navigated to
// navigation.formData: the submitted form data
// navigation.json: the JSON submitted (for fetchers)
```

**useRouteLoaderData:**
```jsx
const parentData = useRouteLoaderData("root"); // Route ID
```

**useMatches:**
```jsx
const matches = useMatches();
// Array of { id, pathname, params, data, handle }
```

**Syntax Rules:**
- `useLoaderData` can be called only in a route component or its children.
- `useActionData` returns `undefined` until an action has run.
- `useNavigation().state` is `"idle"` when no navigation is in progress.
- `useRouteLoaderData` requires a route ID (either auto-generated or set explicitly in the route config).
- `useMatches` is useful for breadcrumbs and analytics; the `handle` property is a place for route metadata.

**Constraints and Limitations:**
- `useLoaderData` returns the type inferred from the loader; use TypeScript generics for strict typing.
- `useNavigation` does not tell you which specific action is running when there are multiple fetchers.
- `useMatches` does not include loader data by default; use `useRouteLoaderData` for data.
- In v7, `useLoaderData` is route-scoped; child routes cannot access parent loader data directly without `useRouteLoaderData`.

### Annotated Code Example: Breadcrumbs with useMatches

```jsx
import { useMatches, Link } from "react-router";

function Breadcrumbs() {
  const matches = useMatches();

  const crumbs = matches
    .filter(match => match.handle?.crumb)
    .map(match => ({
      label: match.handle.crumb(match.data),
      path: match.pathname,
    }));

  return (
    <nav aria-label="Breadcrumb">
      <ol>
        {crumbs.map((crumb, i) => (
          <li key={crumb.path}>
            {i === crumbs.length - 1 ? (
              <span aria-current="page">{crumb.label}</span>
            ) : (
              <Link to={crumb.path}>{crumb.label}</Link>
            )}
          </li>
        ))}
      </ol>
    </nav>
  );
}
```

**Expected Output:** A breadcrumb navigation showing the current route hierarchy with links to parent routes.

**Why This Output Occurs:** `useMatches` returns all matches for the current URL. Routes define a `handle` object with a `crumb` function that derives the label from loader data. The component filters and maps the matches into breadcrumb items.

### Real-World Cases

- **Breadcrumbs:** `useMatches` with `handle.crumb` for hierarchical navigation.
- **Pending UI:** `useNavigation` for button states during form submission.
- **Parent data:** `useRouteLoaderData` to access data from a parent loader in a child route.
- **Non-navigating forms:** `useFetcher` for actions that should not change the route.

### References

- React Router – useLoaderData: https://reactrouter.com/api/hooks/useLoaderData
- React Router – useActionData: https://reactrouter.com/api/hooks/useActionData
- React Router – useNavigation: https://reactrouter.com/api/hooks/useNavigation
- React Router – useRouteLoaderData: https://reactrouter.com/api/hooks/useRouteLoaderData
- React Router – useMatches: https://reactrouter.com/api/hooks/useMatches

---

## Core Concept 7: Nested Layouts and Layout-Level Inheritance

### Definitions

**Core Definition:** Nested layouts allow routes to inherit UI chrome (headers, sidebars, footers) from parent routes, with child routes rendering inside the parent's `<Outlet />` and inheriting the parent's data and state.

**Technical Definition:** Layout routes in React Router v7 are pathless routes (or routes with a `layout()` helper) that render shared UI and an `<Outlet />` where child routes appear. Nested layouts create a hierarchy where multiple levels of shared UI wrap the leaf route. Data from parent loaders is available to child routes via `useRouteLoaderData`, and parent `<Outlet context={...}>` values are accessible via `useOutletContext`. Layouts can be deeply nested (e.g., app layout → dashboard layout → settings layout → settings page), each adding its own UI and data. Layout-level inheritance means child routes automatically benefit from the parent's error boundaries, loaders, and layout chrome without duplicating code.

**Beginner-Friendly Explanation:** Nested layouts are like Russian dolls. The outermost doll (app layout) has a header and footer. Inside it is the dashboard layout with a sidebar. Inside that is the settings layout with tabs. And inside that is the settings page. Each layer adds something, and the innermost page is completely wrapped by all the outer layers. When you navigate to a different settings page, only the innermost content changes—the outer layers stay put.

### Purposes

- To share UI chrome across multiple routes without duplication.
- To create hierarchical data scopes where parent loaders provide data to child routes.
- To enable nested error boundaries for granular error handling.
- To pass context from parent layouts to child routes via `<Outlet context>`.
- To structure complex applications into clear layout hierarchies.

### Syntax Rules and Structure

**Nested Layout Routes:**
```jsx
const router = createBrowserRouter([
  {
    path: "/",
    element: <AppLayout />,
    errorElement: <AppError />,
    children: [
      {
        path: "dashboard",
        element: <DashboardLayout />,
        loader: dashboardLoader,
        children: [
          { index: true, element: <DashboardOverview /> },
          { path: "settings", element: <SettingsPage /> },
        ],
      },
    ],
  },
]);
```

**Passing Context to Child Routes:**
```jsx
// Parent
import { Outlet } from "react-router";

function DashboardLayout() {
  const [theme, setTheme] = useState("light");
  return (
    <div>
      <h1>Dashboard</h1>
      <Outlet context={{ theme, setTheme }} />
    </div>
  );
}

// Child
import { useOutletContext } from "react-router";

function DashboardOverview() {
  const { theme } = useOutletContext();
  return <p>Theme: {theme}</p>;
}
```

**Accessing Parent Loader Data:**
```jsx
// Parent route config
{ id: "dashboard", path: "dashboard", loader: dashboardLoader, element: <DashboardLayout /> }

// Child component
import { useRouteLoaderData } from "react-router";

function SettingsPage() {
  const dashboardData = useRouteLoaderData("dashboard");
  return <p>Parent loaded: {dashboardData.title}</p>;
}
```

**Syntax Rules:**
- Use pathless routes or `layout()` for layout routes.
- Layout routes must render `<Outlet />` for child routes.
- Pass data to child routes via `<Outlet context={{ ... }} />` and read it with `useOutletContext()`.
- Access parent loader data via `useRouteLoaderData("parentRouteId")`.
- Nest layouts arbitrarily deep, but keep the hierarchy understandable.
- Each layout can have its own `errorElement` for granular error handling.

**Constraints and Limitations:**
- `useOutletContext` returns the context from the nearest parent `<Outlet>`; there is no deep merging of contexts.
- Parent loader data is only accessible via `useRouteLoaderData` with the route ID; the ID must be stable.
- Deeply nested layouts can cause prop forwarding and make debugging harder.
- Layout re-renders can cause child re-renders unless memoised.

### Annotated Code Example: Multi-Level Layout Hierarchy

```jsx
import { createBrowserRouter, RouterProvider, Outlet, useOutletContext, useRouteLoaderData } from "react-router";

// Level 1: App layout
function AppLayout() {
  return (
    <div>
      <header><h1>My App</h1></header>
      <Outlet />
      <footer>© 2024</footer>
    </div>
  );
}

// Level 2: Dashboard layout with data
async function dashboardLoader() {
  return { title: "Dashboard", widgets: 4 };
}

function DashboardLayout() {
  const data = useRouteLoaderData("dashboard");
  return (
    <div>
      <h2>{data.title}</h2>
      <nav>Dashboard Navigation</nav>
      <Outlet context={{ widgetCount: data.widgets }} />
    </div>
  );
}

// Level 3: Settings page
function SettingsPage() {
  const { widgetCount } = useOutletContext();
  return <p>You have {widgetCount} widgets configured.</p>;
}

// Route config
const router = createBrowserRouter([
  {
    path: "/",
    element: <AppLayout />,
    children: [
      {
        id: "dashboard",
        path: "dashboard",
        loader: dashboardLoader,
        element: <DashboardLayout />,
        children: [
          { path: "settings", element: <SettingsPage /> },
        ],
      },
    ],
  },
]);

export default function App() {
  return <RouterProvider router={router} />;
}
```

**Expected Output:** Navigating to `/dashboard/settings` renders the app layout (header + footer), the dashboard layout (title + nav), and the settings page ("You have 4 widgets configured."). All three levels of UI are visible simultaneously.

**Why This Output Occurs:** The app layout renders its `<Outlet />`, which renders the dashboard layout. The dashboard layout renders its own `<Outlet context={{ widgetCount }} />`, which renders the settings page. The settings page reads the context via `useOutletContext()`. The dashboard layout reads its loader data via `useRouteLoaderData("dashboard")`.

### Real-World Cases

- **SaaS dashboards:** App layout → dashboard layout → widget layouts → widget pages.
- **E-commerce:** Site layout → account layout → orders layout → order detail.
- **Documentation:** Site layout → docs layout → section layout → article.
- **Admin panels:** Admin layout → resource layout → list/detail/edit routes.

### References

- React Router – Nested Routes: https://mintlify.wiki/remix-run/react-router/routing/nested-routes
- React Router – useOutletContext: https://reactrouter.com/api/hooks/useOutletContext
- React Router – useRouteLoaderData: https://reactrouter.com/api/hooks/useRouteLoaderData
- React Router – Layout Routes: https://mintlify.wiki/remix-run/react-router/routing/layout-routes

---

## Core Concept 8: Advanced URL State Management and Scroll Restoration

### Definitions

**Core Definition:** Advanced URL state management is the practice of encoding complex application state (filters, pagination, sort order, tabs, search queries) in the URL's query parameters, while scroll restoration is the automatic restoration of scroll positions during back/forward navigation via the `<ScrollRestoration>` component.

**Technical Definition:** React Router provides `useSearchParams` for reading and writing query parameters (covered in the basic cheat sheet) and `useLocation` for accessing the full location object (`pathname`, `search`, `hash`, `state`). Advanced URL state management includes serialising complex objects into query params, validating and defaulting params, and synchronising them with other state (e.g., TanStack Query). `<ScrollRestoration>` is a component that renders nothing but restores the scroll position on navigation. It uses `sessionStorage` to save scroll positions per location key. On back/forward navigation, it restores the saved position; on new navigations, it scrolls to the top. It can be customised with `getKey` (for custom scroll restoration keys) and can be disabled with `<ScrollRestoration getKey={...} />` or by not rendering it. React Router v7 automatically restores scroll on back/forward navigation when `<ScrollRestoration />` is rendered in the root layout.

**Beginner-Friendly Explanation:** Advanced URL state management is about putting as much of your app's state as possible in the URL so it is shareable, bookmarkable, and compatible with the browser's back/forward buttons. Scroll restoration is about remembering where the user was scrolling on a page so that when they hit the Back button, they are returned to exactly where they left off—just like a native app would do.

### Purposes

- To encode complex application state (filters, pagination, sort) in the URL.
- To validate and default query parameters for robust URL handling.
- To synchronise URL state with server-state caches (TanStack Query).
- To restore scroll position on back/forward navigation.
- To scroll to the top on new navigations.
- To customise scroll restoration for special cases (e.g., tabs, modals).

### Syntax Rules and Structure

**Advanced useSearchParams:**
```jsx
import { useSearchParams } from "react-router";

function FilterPanel() {
  const [searchParams, setSearchParams] = useSearchParams();

  // Read with defaults
  const category = searchParams.get("category") ?? "all";
  const page = Number(searchParams.get("page") ?? "1");
  const sort = searchParams.get("sort") ?? "name";

  // Update multiple params at once
  function updateFilters({ category, page, sort }) {
    setSearchParams({
      ...(category && { category }),
      ...(page && { page: String(page) }),
      ...(sort && { sort }),
    });
  }

  // Reset all filters
  function resetFilters() {
    setSearchParams({});
  }

  return (
    <div>
      <p>Category: {category}, Page: {page}, Sort: {sort}</p>
      <button onClick={() => updateFilters({ category: "books", page: 1 })}>
        Filter Books
      </button>
      <button onClick={resetFilters}>Reset</button>
    </div>
  );
}
```

**ScrollRestoration:**
```jsx
import { ScrollRestoration } from "react-router";

function RootLayout() {
  return (
    <div>
      <header>Header</header>
      <main>
        <Outlet />
      </main>
      <footer>Footer</footer>
      <ScrollRestoration />
    </div>
  );
}
```

**Custom ScrollRestoration Key:**
```jsx
<ScrollRestoration
  getKey={(location, matches) => {
    // Customise the key used to save scroll positions
    return location.pathname;
  }}
/>
```

**Syntax Rules:**
- Use `useSearchParams` for query param state; provide defaults with `??`.
- Use `setSearchParams` with an object or `URLSearchParams` to update params.
- Use `<ScrollRestoration />` in the root layout to enable scroll restoration.
- The `getKey` prop customises the key used to save scroll positions (default: `location.key`).
- Scroll restoration saves positions in `sessionStorage`; clearing sessionStorage clears them.
- Scroll restoration works automatically on back/forward navigation; new navigations scroll to the top.

**Constraints and Limitations:**
- URL state is public; do not store sensitive data.
- URL length is limited; avoid storing large objects.
- Scroll restoration requires `<ScrollRestoration />` to be rendered; without it, no scroll restoration occurs.
- Custom `getKey` implementations must be stable and deterministic.
- Scroll restoration may conflict with custom scroll containers; use `getKey` and manual scroll handling if needed.

### Annotated Code Example: Filterable Product List with URL State and Scroll Restoration

```jsx
import { createBrowserRouter, RouterProvider, Outlet, ScrollRestoration, useSearchParams, useLoaderData } from "react-router";

// Loader reads URL params
async function productsLoader({ request }) {
  const url = new URL(request.url);
  const category = url.searchParams.get("category") ?? "all";
  const page = Number(url.searchParams.get("page") ?? "1");
  // Simulated data fetch
  return {
    products: Array.from({ length: 10 }, (_, i) => ({
      id: (page - 1) * 10 + i + 1,
      name: `Product ${(page - 1) * 10 + i + 1}`,
      category,
    })),
    category,
    page,
  };
}

function ProductsPage() {
  const { products, category, page } = useLoaderData();
  const [searchParams, setSearchParams] = useSearchParams();

  function goToPage(newPage) {
    setSearchParams({ ...Object.fromEntries(searchParams), page: String(newPage) });
  }

  return (
    <div>
      <h1>Products ({category})</h1>
      <ul>
        {products.map(p => <li key={p.id}>{p.name}</li>)}
      </ul>
      <button disabled={page <= 1} onClick={() => goToPage(page - 1)}>
        Previous
      </button>
      <span>Page {page}</span>
      <button onClick={() => goToPage(page + 1)}>Next</button>
    </div>
  );
}

function RootLayout() {
  return (
    <div>
      <header><h1>Store</h1></header>
      <main><Outlet /></main>
      <ScrollRestoration />
    </div>
  );
}

const router = createBrowserRouter([
  {
    path: "/",
    element: <RootLayout />,
    children: [
      {
        path: "products",
        loader: productsLoader,
        element: <ProductsPage />,
      },
    ],
  },
]);

export default function App() {
  return <RouterProvider router={router} />;
}
```

**Expected Output:** Navigating to `/products?category=books&page=2` shows products for page 2. Clicking "Next" updates the URL to `page=3` and shows page 3. Using the browser's Back button returns to page 2 and restores the scroll position.

**Why This Output Occurs:** The loader reads query params from the `request` URL. `useSearchParams` reads and updates them. `setSearchParams` causes a navigation and re-runs the loader. `<ScrollRestoration />` saves the scroll position for each history entry and restores it on back/forward navigation.

### Real-World Cases

- **E-commerce:** Product listing with category, sort, and pagination in the URL; scroll restoration on back.
- **Search results:** Search query and page in the URL; scroll restoration returns to the same position.
- **Data tables:** Sort column, direction, and page in the URL; scroll restoration preserves table position.
- **Documentation:** Section and article in the URL; scroll restoration returns to the reading position.

### References

- React Router – useSearchParams: https://api.reactrouter.com/v7/functions/react-router.useSearchParams.html
- React Router – ScrollRestoration: https://reactrouter.com/api/components/ScrollRestoration
- React Router – getKey: https://reactrouter.com/api/components/ScrollRestoration#getkey
- React Router – Session Storage: https://reactrouter.com/api/components/ScrollRestoration

---

## Comparison and Decision Guidance

| Concern | API/Pattern | When to Use |
|---|---|---|
| **Route Protection** | `redirect()` in loader, `<Navigate>` wrapper | Protect routes requiring auth/roles |
| **Auth-Aware Navigation** | Context + conditional rendering | Show/hide nav based on auth state |
| **Error Handling** | `errorElement`, `useRouteError` | Catch loader/action/component errors |
| **404 Handling** | `path: "*"`, `isRouteErrorResponse` | Catch unmatched URLs |
| **Code Splitting** | `lazy` route property, `React.lazy` | Reduce initial bundle size |
| **Data Loading** | `loader`, `useLoaderData` | Fetch data before rendering |
| **Mutations** | `action`, `<Form>`, `useActionData` | Handle form submissions |
| **Pending UI** | `useNavigation().state` | Show loading states during submission |
| **Nested Layouts** | Pathless routes, `<Outlet>`, `useOutletContext` | Share UI chrome across routes |
| **URL State** | `useSearchParams` | Filters, sort, pagination |
| **Scroll Restoration** | `<ScrollRestoration />` | Restore scroll on back/forward |

**Decision Guidance:**
- **Start with loaders and actions** for all data loading and mutations; they run before rendering and eliminate loading spinners.
- **Use `errorElement` at multiple levels** for granular error handling; avoid a single top-level error boundary.
- **Lazy-load non-critical routes** to reduce initial bundle size; keep critical routes in the main bundle.
- **Use `<Form>` over `<form>`** for progressive enhancement and automatic action integration.
- **Use `useNavigation` for pending UI** instead of local `isSubmitting` state.
- **Nest layouts deliberately** — each layout should add a distinct layer of UI chrome.
- **Put filter, sort, and pagination state in the URL** for shareability and browser navigation.
- **Always render `<ScrollRestoration />` in the root layout** for a native-like back/forward experience.

---

## References

- React Router – Loaders: https://reactrouter.com/7.1.0/start/data/loaders
- React Router – Actions: https://reactrouter.com/7.1.0/start/data/actions
- React Router – Error Boundaries: https://reactrouter.com/7.1.0/start/data/error-boundaries
- React Router – Lazy Loading: https://reactrouter.com/7.1.0/start/data/lazy-loading
- React Router – redirect: https://reactrouter.com/api/data-routers/redirect
- React Router – useLoaderData: https://reactrouter.com/api/hooks/useLoaderData
- React Router – useActionData: https://reactrouter.com/api/hooks/useActionData
- React Router – useNavigation: https://reactrouter.com/api/hooks/useNavigation
- React Router – useRouteError: https://reactrouter.com/api/hooks/useRouteError
- React Router – useRevalidator: https://reactrouter.com/api/hooks/useRevalidator
- React Router – useRouteLoaderData: https://reactrouter.com/api/hooks/useRouteLoaderData
- React Router – useMatches: https://reactrouter.com/api/hooks/useMatches
- React Router – useOutletContext: https://reactrouter.com/api/hooks/useOutletContext
- React Router – useSearchParams: https://api.reactrouter.com/v7/functions/react-router.useSearchParams.html
- React Router – ScrollRestoration: https://reactrouter.com/api/components/ScrollRestoration
- React Router – Form: https://reactrouter.com/api/components/Form
- React Router – isRouteErrorResponse: https://reactrouter.com/api/utils/isRouteErrorResponse
- React Router – Navigate: https://reactrouter.com/6.30.3/components/navigate
- React – Error Boundaries: https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary
- React – lazy: https://react.dev/reference/react/lazy
- React – Suspense: https://react.dev/reference/react/Suspense
- React Context – Passing Data Deeply: https://react.dev/learn/passing-data-deeply-with-context
- TanStack Query – Authenticated Routes: https://tanstack.com/router/latest/docs/framework/react/guide/authenticated-routes
- CoreUI – How to redirect in React Router: https://coreui.io/answers/how-to-redirect-in-react-router/