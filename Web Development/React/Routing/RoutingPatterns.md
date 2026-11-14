# React Routing Patterns — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** React routing patterns are the reusable architectural solutions for controlling who can access which routes, how navigation is guarded, how transitions are presented, and how route code and data are loaded efficiently.

**Technical Definition:** Routing patterns in React address five distinct concerns. **Access control** determines whether a route is public, private (authenticated), or role-gated, enforced through guards, loaders, middleware, or wrapper components. **Navigation control** covers redirects (declarative and imperative), route guards that run before rendering, and the distinction between client-side UX guards and server-side security checks. **Transition and animation** patterns govern pending states, view transitions, scroll restoration, and focus management for accessibility. **Loading strategy** includes route-level code splitting, lazy loading, and prefetching on intent. **Error handling** uses route-level error boundaries, 404 routes, and loaders that throw HTTP errors. The overarching principle is that **routing is a security boundary only on the server** — client-side guards are a UX convenience and must always be backed by server-side authorisation.

**Beginner-Friendly Explanation:** Routing patterns are the rules for how pages behave. Some pages are open to everyone (public). Some require you to be logged in (private). Some require a specific role like admin (role-based). Some pages should redirect you somewhere else if you are not allowed in. And some pages should load smoothly with a nice transition, or not load at all until you click. This cheat sheet covers all of those patterns — and the most important lesson: the client can always be bypassed, so the server must always check.

### Key Characteristics

- **Access control is layered:** Public → Private (authenticated) → Role-based (authorised), with each layer enforced at the appropriate boundary.
- **Client guards are UX; server checks are security:** Never rely on a client-side guard to protect sensitive data or actions.
- **Redirects are declarative or imperative:** `<Navigate>` for render-time redirects, `useNavigate`/`redirect()` for event or loader-driven redirects.
- **Route guards run before rendering:** Loaders, `beforeLoad`, and middleware prevent rendering of protected content rather than rendering-then-redirecting.
- **Transitions improve perceived performance:** Pending UI, view transitions, scroll restoration, and focus management make navigation feel intentional.
- **Loading strategy is a routing concern:** Route-level splitting, `lazy`, and prefetching on intent are all router-level optimisations.
- **Error handling is segmented:** 404 routes, route-level error boundaries, and loader-thrown errors isolate failures.

### Prerequisites

- React function components, `useState`, and `useEffect`.
- React Router or TanStack Router basics (routes, links, hooks).
- Understanding of authentication and authorisation concepts.
- Familiarity with `localStorage`, cookies, and server sessions.
- Knowledge of React Context for auth state.
- Understanding of route-level code splitting (`React.lazy` or router-native `lazy`).

### Related Programming Areas

- **Authentication & Authorization** — sessions, tokens, roles, permissions.
- **React Router & TanStack Router** — the routers implementing these patterns.
- **Code Splitting & Performance** — lazy loading, prefetching, bundles.
- **Accessibility** — focus management, route announcements, skip links.
- **Error Handling** — error boundaries, HTTP errors, fallbacks.

---

## 1. Protected Routes

### Definitions

**Core Definition:** A protected route is a route that requires the user to be authenticated before it can be accessed.

**Technical Definition:** Protected routes are routes that check for a valid session or token before rendering. If the check fails, the user is redirected to a login page (often with the intended destination preserved for post-login return). Protected routes are the broad category that includes private routes and role-based routes. In React Router Data Mode, protection is implemented in a `loader` that throws `redirect()`; in Declarative Mode, a wrapper component or route guard renders `<Navigate>` when unauthenticated. In Next.js, `middleware.ts` and layout-level checks enforce protection. Critically, client-side protection is a UX layer; the server must independently verify authorization for every request.

**Beginner-Friendly Explanation:** A protected route is a page that says “you must be logged in to see this.” If you are not logged in, it sends you to the login page.

### Purposes

- To prevent unauthenticated users from viewing account-specific content.
- To redirect users to login with the intended destination preserved.
- To centralise authentication checks rather than duplicating them in every component.
- To avoid rendering content that would immediately fail API calls.
- To provide a consistent authentication experience across protected areas.

### Syntax Rules and Structure

**React Router Data Mode (loader guard):**

```jsx
import { redirect } from 'react-router';

async function requireAuth() {
  const user = await getUser();
  if (!user) throw redirect('/login?redirect=' + encodeURIComponent(location.pathname));
  return user;
}

const router = createBrowserRouter([
  {
    path: '/dashboard',
    loader: requireAuth,
    element: <DashboardLayout />,
    children: [{ index: true, element: <Dashboard /> }],
  },
]);
```

**React Router Declarative Mode (wrapper):**

```jsx
function ProtectedRoute({ children }) {
  const user = useUser();
  if (!user) return <Navigate to="/login" replace />;
  return children;
}

<Route path="/dashboard" element={<ProtectedRoute><Dashboard /></ProtectedRoute>} />
```

**TanStack Router (`beforeLoad`):**

```tsx
export const Route = createFileRoute('/dashboard')({
  beforeLoad: async ({ context }) => {
    if (!context.auth.user) throw redirect({ to: '/login' });
  },
  component: Dashboard,
});
```

**Syntax Rules:**

1. Run the auth check before rendering, not after: use a loader (React Router Data), `beforeLoad` (TanStack Router), or middleware (Next.js).
2. Throw/return a redirect rather than rendering a “please log in” message inside the protected page.
3. Preserve the intended destination in the redirect query string for post-login return.
4. Never rely on client-side protection alone; the server must enforce authorisation for every request.
5. Group protected routes under a shared layout so the guard applies once.

**Constraints and Limitations:**

- Client-side guards can be bypassed by disabling JavaScript or calling APIs directly.
- Guards that run after rendering cause flicker and wasted API calls.
- Redirect loops occur when the login route itself is protected.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: React Router Loader Guard with Redirect Preservation

```jsx
import { redirect, createBrowserRouter, Outlet } from 'react-router';

async function requireAuth({ request }) {
  const user = await getCurrentUser();
  if (!user) {
    const url = new URL(request.url);
    const redirectTo = encodeURIComponent(url.pathname + url.search);
    throw redirect(`/login?redirect=${redirectTo}`);
  }
  return { user };
}

const router = createBrowserRouter([
  {
    path: '/',
    element: <RootLayout />,
    children: [
      { index: true, element: <Home /> },
      {
        path: 'dashboard',
        loader: requireAuth,
        element: <DashboardLayout />,
        children: [{ index: true, element: <Dashboard /> }],
      },
      { path: 'login', element: <Login /> },
    ],
  },
]);
```

**Expected Output:** An unauthenticated user visiting `/dashboard` is redirected to `/login?redirect=%2Fdashboard`. After logging in, the login page reads the `redirect` param and navigates back to `/dashboard`.

**Why this output:** The loader runs before rendering and throws `redirect()` when the user is missing. Encoding the original path preserves the destination, and the login page uses it to return the user.

#### Example 2: TanStack Router `beforeLoad` Guard

```tsx
export const Route = createFileRoute('/dashboard')({
  beforeLoad: async ({ context, location }) => {
    if (!context.auth.user) {
      throw redirect({
        to: '/login',
        search: { redirect: location.href },
      });
    }
    return { user: context.auth.user };
  },
  loader: ({ context: { user } }) => fetchDashboard(user.id),
  component: Dashboard,
});
```

**Expected Output:** Unauthenticated users are redirected to `/login?redirect=/dashboard`. Authenticated users proceed to the loader with the validated user context.

**Why this output:** `beforeLoad` runs before loaders and rendering. It throws a typed redirect with the current location and returns the user context for downstream use.

### Real-World Cases

- **SaaS dashboards:** Requiring login for all `/app/*` routes.
- **Account pages:** Profile, billing, and settings behind authentication.
- **E-commerce:** Order history and saved addresses behind login.
- **Admin panels:** Protected by authentication and role checks.

### References

- React Router — redirect — https://reactrouter.com/api/data-routers/redirect
- TanStack Router — Authenticated Routes — https://tanstack.com/router/latest/docs/framework/react/guide/authenticated-routes
- Next.js — Middleware — https://nextjs.org/docs/app/building-your-application/routing/middleware
- Clerk — Protected Routes — https://clerk.com/docs

---

## 2. Public Routes

### Definitions

**Core Definition:** A public route is a route that is accessible to all users, regardless of authentication status.

**Technical Definition:** Public routes do not require authentication or authorisation checks. They include marketing pages, landing pages, login and signup pages, password reset, public documentation, and legal pages. In applications with a mix of public and protected areas, public routes are defined outside the protected layout, or explicitly excluded from middleware matchers. The login and signup routes must be public to avoid redirect loops.

**Beginner-Friendly Explanation:** Public routes are open to everyone — the home page, the about page, the login page. No login required.

### Purposes

- To make marketing and content pages accessible to all visitors.
- To allow unauthenticated users to log in or sign up.
- To support password reset and email verification flows.
- To avoid redirect loops by keeping auth routes public.
- To allow search engines to index public content.

### Syntax Rules and Structure

**React Router:**

```jsx
const router = createBrowserRouter([
  { path: '/', element: <Landing /> },          // public
  { path: '/pricing', element: <Pricing /> },    // public
  { path: '/login', element: <Login /> },        // public
  { path: '/signup', element: <Signup /> },      // public
  { path: '/dashboard', loader: requireAuth, element: <Dashboard /> }, // protected
]);
```

**Next.js Middleware (exclude public paths):**

```ts
export const config = {
  matcher: ['/dashboard/:path*', '/account/:path*'], // Only protect these
};
```

**Syntax Rules:**

1. Define public routes outside the protected layout or loader guard.
2. Keep `/login`, `/signup`, `/reset-password`, and verification routes public.
3. In middleware, use a narrow matcher for protected paths rather than a broad matcher with exclusions.
4. Ensure public routes do not render user-specific data that requires authentication.

**Constraints and Limitations:**

- Public routes must not leak sensitive data or trigger authenticated API calls.
- Broad middleware matchers with exclusion logic are error-prone; prefer narrow inclusion matchers.
- Public routes that redirect authenticated users away (e.g., login) need their own guard.

### Annotated Code Examples

```tsx
// Public landing page
export default function Landing() {
  return (
    <main>
      <Hero />
      <Features />
      <Link to="/login">Log in</Link>
      <Link to="/signup">Get started</Link>
    </main>
  );
}
```

**Expected Output:** The landing page renders for everyone, with links to login and signup.

**Why this output:** The route has no loader guard or middleware protection. It is accessible without authentication.

### Real-World Cases

- **Marketing sites:** Home, pricing, features, blog.
- **Authentication pages:** Login, signup, password reset, email verification.
- **Legal pages:** Terms, privacy, cookies.
- **Public documentation:** API docs and guides.

### References

- React Router — Route Objects — https://reactrouter.com/start/data/route-object
- Next.js — Middleware matchers — https://nextjs.org/docs/app/building-your-application/routing/middleware
- Clerk — Public Routes — https://clerk.com/docs

---

## 3. Private Routes

### Definitions

**Core Definition:** A private route is a route that is accessible only to authenticated users and is not visible or reachable by unauthenticated visitors.

**Technical Definition:** Private routes are the authenticated subset of protected routes. They check for a valid session and redirect unauthenticated users to login. Private routes often share a protected layout that renders the authenticated shell (sidebar, header) and an `<Outlet />`. In React Router Data Mode, a parent route loader performs the auth check, and all child routes inherit the protection. The distinction from “protected routes” is mostly terminological: private routes emphasise that the route is not part of the public surface at all.

**Beginner-Friendly Explanation:** Private routes are the pages behind the login wall. You cannot see them unless you are logged in.

### Purposes

- To hide the authenticated application shell from unauthenticated users.
- To apply authentication checks once to an entire route subtree.
- To provide a consistent authenticated layout (sidebar, header).
- To redirect unauthenticated users to login with return-to-destination.
- To keep public and private route trees cleanly separated.

### Syntax Rules and Structure

**Parent loader protects the subtree:**

```jsx
const router = createBrowserRouter([
  {
    path: '/app',
    loader: requireAuth,          // Runs for all /app/* routes
    element: <AppLayout />,       // Authenticated shell
    children: [
      { index: true, element: <Dashboard /> },
      { path: 'projects', element: <Projects /> },
      { path: 'settings', element: <Settings /> },
    ],
  },
]);
```

**Syntax Rules:**

1. Attach the auth guard to a parent layout route so all children inherit it.
2. Render the authenticated shell (sidebar, header) in the parent layout.
3. Keep private routes under a distinct path prefix (e.g., `/app`) for clarity.
4. Preserve the intended destination when redirecting to login.

**Constraints and Limitations:**

- The parent loader runs on every navigation into the subtree; keep it fast.
- Child routes cannot opt out of the parent loader's guard.
- Client-side guards must be backed by server-side authorisation.

### Annotated Code Examples

```tsx
function AppLayout() {
  return (
    <div className="app-shell">
      <Sidebar />
      <main><Outlet /></main>
    </div>
  );
}

const router = createBrowserRouter([
  {
    path: '/app',
    loader: requireAuth,
    element: <AppLayout />,
    children: [
      { index: true, element: <Dashboard /> },
      { path: 'settings', element: <Settings /> },
    ],
  },
]);
```

**Expected Output:** `/app` and `/app/settings` render inside the authenticated shell. Unauthenticated users are redirected to login before the shell renders.

**Why this output:** The parent loader runs before rendering the layout. The shell only appears after the guard passes.

### Real-World Cases

- **SaaS applications:** `/app/*` routes behind login.
- **Banking apps:** Account dashboards and transactions.
- **Healthcare portals:** Patient records and appointments.
- **Enterprise tools:** Internal dashboards and admin panels.

### References

- React Router — Route Objects (nested loaders) — https://reactrouter.com/start/data/route-object
- TanStack Router — Authenticated Routes — https://tanstack.com/router/latest/docs/framework/react/guide/authenticated-routes
- Next.js — Middleware — https://nextjs.org/docs/app/building-your-application/routing/middleware

---

## 4. Role-Based Routes

### Definitions

**Core Definition:** A role-based route is a route that is accessible only to users with specific roles or permissions.

**Technical Definition:** Role-based routes extend protected routes with an authorisation check beyond authentication. After confirming the user is logged in, the guard checks whether the user's role (e.g., `admin`, `editor`, `viewer`) or permission set allows access. If not, the user is redirected to a forbidden page or the dashboard. Role checks must be enforced on the server; client-side checks are UX only. In React Router, role checks run in loaders after the auth check; in TanStack Router, they run in `beforeLoad`; in Next.js, they run in middleware or server-side layout checks.

**Beginner-Friendly Explanation:** Role-based routes are pages only certain people can see — like admin pages that only admins can access.

### Purposes

- To restrict sensitive functionality to authorised roles.
- To prevent privilege escalation through direct URL access.
- To provide role-appropriate navigation and UI.
- To centralise authorisation logic in guards rather than scattering checks.
- To redirect unauthorised users to a safe page.

### Syntax Rules and Structure

**React Router loader with role check:**

```jsx
async function requireRole(role) {
  const user = await getCurrentUser();
  if (!user) throw redirect('/login');
  if (user.role !== role) throw redirect('/forbidden');
  return user;
}

const router = createBrowserRouter([
  {
    path: '/admin',
    loader: () => requireRole('admin'),
    element: <AdminLayout />,
    children: [{ index: true, element: <AdminDashboard /> }],
  },
]);
```

**TanStack Router `beforeLoad` with role check:**

```tsx
export const Route = createFileRoute('/admin')({
  beforeLoad: async ({ context }) => {
    if (!context.auth.user) throw redirect({ to: '/login' });
    if (context.auth.user.role !== 'admin') throw redirect({ to: '/forbidden' });
  },
});
```

**Syntax Rules:**

1. Perform the authentication check first, then the role check.
2. Redirect unauthorised users to a dedicated forbidden page, not to login.
3. Enforce role checks on the server for every API call that the route depends on.
4. Use a single source of truth for roles and permissions (e.g., a `usePermissions` hook or server-provided context).
5. Group role-protected routes under a parent guard where possible.

**Constraints and Limitations:**

- Client-side role checks can be bypassed; server-side checks are mandatory.
- Roles can be stale if the user's role changes mid-session; revalidate on sensitive actions.
- Complex permission models (per-resource permissions) may need a dedicated policy engine.

### Annotated Code Examples

```tsx
async function requireAdmin({ request }) {
  const user = await getCurrentUser();
  if (!user) throw redirect('/login');
  if (user.role !== 'admin') throw redirect('/forbidden');
  return { user };
}

const router = createBrowserRouter([
  {
    path: '/admin',
    loader: requireAdmin,
    element: <AdminLayout />,
    children: [
      { index: true, element: <AdminDashboard /> },
      { path: 'users', element: <UserManagement /> },
    ],
  },
]);
```

**Expected Output:** Admins access `/admin` and `/admin/users`. Non-admin authenticated users are redirected to `/forbidden`. Unauthenticated users are redirected to `/login`.

**Why this output:** The loader checks authentication first, then role. The redirect target differs based on whether the user is missing or unauthorised.

### Real-World Cases

- **Admin panels:** User management and system settings.
- **Billing:** Plan changes restricted to account owners.
- **Content management:** Editor-only publishing controls.
- **Support tools:** Agent-only customer data access.

### References

- React Router — redirect — https://reactrouter.com/api/data-routers/redirect
- TanStack Router — Authenticated Routes (role checks) — https://tanstack.com/router/latest/docs/framework/react/guide/authenticated-routes
- Next.js — Middleware (role checks) — https://nextjs.org/docs/app/building-your-application/routing/middleware

---

## 5. Authentication Routes

### Definitions

**Core Definition:** Authentication routes are the routes dedicated to the sign-in, sign-up, sign-out, and account-recovery flows.

**Technical Definition:** Authentication routes include `/login`, `/signup`, `/logout`, `/forgot-password`, `/reset-password`, and verification routes. They are public (must be reachable without authentication) but often have inverse guards: authenticated users visiting `/login` may be redirected to the dashboard. Authentication routes handle credential submission, session creation, and redirect to the intended destination. They should never be protected by the auth guard itself, or they cause redirect loops.

**Beginner-Friendly Explanation:** Authentication routes are the login and signup pages. They must be open to everyone, or you could never log in.

### Purposes

- To allow users to authenticate and create sessions.
- To provide password recovery and account verification.
- To redirect authenticated users away from auth pages.
- To preserve the intended destination through the login flow.
- To keep auth routes public to avoid redirect loops.

### Syntax Rules and Structure

**Public auth routes:**

```jsx
const router = createBrowserRouter([
  { path: '/login', element: <Login /> },
  { path: '/signup', element: <Signup /> },
  { path: '/forgot-password', element: <ForgotPassword /> },
  { path: '/reset-password', element: <ResetPassword /> },
]);
```

**Inverse guard (redirect authenticated users away from login):**

```jsx
function LoginRoute() {
  const user = useUser();
  if (user) return <Navigate to="/dashboard" replace />;
  return <Login />;
}
```

**Syntax Rules:**

1. Keep auth routes outside the protected layout and guard.
2. Add an inverse guard so authenticated users do not see login/signup.
3. Read the `redirect` query param after successful login and navigate there.
4. Use `replace` when redirecting to avoid polluting history.
5. Invalidate the session and redirect on logout.

**Constraints and Limitations:**

- Protecting the login route causes redirect loops.
- Forgetting the `redirect` param returns users to the dashboard instead of their intended page.
- Auth state can be stale after login; ensure the auth context updates before navigating.

### Annotated Code Examples

```tsx
function Login() {
  const navigate = useNavigate();
  const [searchParams] = useSearchParams();
  const { login } = useAuth();

  async function handleSubmit(e) {
    e.preventDefault();
    const formData = new FormData(e.currentTarget);
    await login({
      email: formData.get('email'),
      password: formData.get('password'),
    });
    const redirectTo = searchParams.get('redirect') ?? '/dashboard';
    navigate(redirectTo, { replace: true });
  }

  return <form onSubmit={handleSubmit}>{/* fields */}</form>;
}
```

**Expected Output:** After login, the user is redirected to the `redirect` param (e.g., `/dashboard`) rather than always to the dashboard.

**Why this output:** The login handler reads the preserved destination and navigates there with `replace` to avoid a history entry for the login page.

### Real-World Cases

- **SaaS login:** Email/password and OAuth login.
- **E-commerce:** Account login and password reset.
- **Enterprise SSO:** SAML/OIDC authentication routes.
- **Multi-tenant apps:** Tenant-specific login.

### References

- React Router — Navigate — https://reactrouter.com/api/components/Navigate
- React Router — useSearchParams — https://reactrouter.com/api/hooks/useSearchParams
- TanStack Router — Authenticated Routes — https://tanstack.com/router/latest/docs/framework/react/guide/authenticated-routes

---

## 6. Authorization Routes

### Definitions

**Core Definition:** Authorization routes are the routes and guards that determine what an authenticated user is allowed to do, not just whether they are logged in.

**Technical Definition:** Authorization is distinct from authentication: authentication answers “who are you?”, authorization answers “what are you allowed to do?”. Authorization routes enforce permission checks — role-based access, resource-level ownership, feature entitlements, or policy-based rules (e.g., ABAC). They are implemented in the same guard layer as role-based routes but may require server-side policy evaluation. In React Router, authorisation can be enforced in loaders; in TanStack Router, in `beforeLoad`; in Next.js, in middleware or server-side layout checks. Server-side authorization is mandatory for every data access.

**Beginner-Friendly Explanation:** Authorization is about permissions, not identity. Being logged in is not enough — you also need permission to see or do this thing.

### Purposes

- To enforce fine-grained permissions beyond simple roles.
- To prevent privilege escalation through direct URL access.
- To gate features by subscription tier or entitlement.
- To protect resource-level operations (e.g., only the owner can edit).
- To provide a consistent authorisation model across routes and APIs.

### Syntax Rules and Structure

**React Router loader with permission check:**

```jsx
async function requirePermission(permission) {
  const user = await getCurrentUser();
  if (!user) throw redirect('/login');
  const allowed = await checkPermission(user.id, permission);
  if (!allowed) throw redirect('/forbidden');
  return user;
}

const router = createBrowserRouter([
  {
    path: '/billing',
    loader: () => requirePermission('billing:manage'),
    element: <BillingPage />,
  },
]);
```

**Syntax Rules:**

1. Perform authentication first, then authorisation.
2. Use a central permission-check function (`hasPermission`, `can`) rather than inline role comparisons.
3. Distinguish between “not logged in” (redirect to login) and “not allowed” (redirect to forbidden or show a 403).
4. Enforce the same permission on the server for every API endpoint the route calls.
5. Revalidate permissions when they may have changed (e.g., after a role change or subscription update).

**Constraints and Limitations:**

- Client-side authorisation is UX; server-side checks are mandatory.
- Permission caches can become stale; invalidate on role changes.
- Complex policies (ABAC, per-resource ownership) may require a dedicated policy engine.

### Annotated Code Examples

```tsx
async function requireBillingAccess() {
  const user = await getCurrentUser();
  if (!user) throw redirect('/login');
  if (!user.permissions.includes('billing:manage')) throw redirect('/forbidden');
  return user;
}

const router = createBrowserRouter([
  {
    path: '/billing',
    loader: requireBillingAccess,
    element: <BillingPage />,
  },
]);
```

**Expected Output:** Users with `billing:manage` access the billing page. Users without it are redirected to `/forbidden`. Unauthenticated users are redirected to `/login`.

**Why this output:** The guard checks authentication first, then the specific permission. The redirect target distinguishes the two failure modes.

### Real-World Cases

- **Subscription tiers:** Feature-gated routes by plan.
- **Team roles:** Owner, admin, member, viewer permissions.
- **Resource ownership:** Only the author can edit a post.
- **Compliance:** Audit-log access restricted to compliance officers.

### References

- React Router — redirect — https://reactrouter.com/api/data-routers/redirect
- TanStack Router — Authenticated Routes — https://tanstack.com/router/latest/docs/framework/react/guide/authenticated-routes
- Next.js — Middleware — https://nextjs.org/docs/app/building-your-application/routing/middleware

---

## 7. Redirects

### Definitions

**Core Definition:** A redirect is a navigation instruction that sends the user from one route to another, either declaratively during render, imperatively from code, or at the server/middleware level.

**Technical Definition:** Redirects in React routing have several forms. **Declarative redirects** use `<Navigate to="..." replace />` in Declarative Mode or `throw redirect()` in Data Mode loaders/actions. **Imperative redirects** use `useNavigate()` or `router.navigate()` in event handlers and effects. **Server-level redirects** use Next.js `redirect()` in Server Actions or `NextResponse.redirect()` in middleware. **Route aliases** redirect old paths to new ones. Redirects should use `replace` for authentication and alias cases to avoid polluting the back stack.

**Beginner-Friendly Explanation:** A redirect is when the app says “not here, go there instead.” After login, you are redirected to the dashboard. If you visit an old URL, you are redirected to the new one.

### Purposes

- To send users to the intended destination after login.
- To redirect old routes to new ones after a rename.
- To send unauthorised users to login or forbidden pages.
- To enforce canonical URLs (e.g., trailing slash, lowercase).
- To redirect authenticated users away from login/signup.

### Syntax Rules and Structure

**Declarative (React Router):**

```jsx
<Navigate to="/login" replace />
```

**Loader (React Router Data Mode):**

```js
throw redirect('/login');
```

**Imperative (React Router):**

```jsx
const navigate = useNavigate();
navigate('/dashboard', { replace: true });
```

**Next.js Middleware:**

```ts
return NextResponse.redirect(new URL('/login', request.url));
```

**Syntax Rules:**

1. Use `replace` for auth redirects and aliases to avoid polluting history.
2. Use `push` (default) when the user should be able to go back (e.g., normal navigation).
3. Preserve the intended destination when redirecting to login.
4. Avoid redirect loops: do not redirect a route to itself, and keep auth routes public.
5. Use server-side redirects for canonical URLs and SEO-sensitive redirects.

**Constraints and Limitations:**

- Redirect loops crash the app; test all auth redirect paths.
- `replace` means the back button skips the redirected-from page.
- Redirects during render must not depend on side effects that run after render.

### Annotated Code Examples

**Declarative redirect:**

```jsx
function LoginRoute() {
  const user = useUser();
  if (user) return <Navigate to="/dashboard" replace />;
  return <Login />;
}
```

**Loader redirect:**

```jsx
async function loader() {
  const user = await getUser();
  if (!user) throw redirect('/login');
  return { user };
}
```

**Expected Output:** Authenticated users visiting `/login` are redirected to `/dashboard`. Unauthenticated users visiting a protected route are redirected to `/login`.

**Why this output:** `<Navigate>` handles the inverse guard for auth routes. `throw redirect()` handles the forward guard for protected routes.

### Real-World Cases

- **Auth flows:** Login → dashboard, logout → login.
- **Route renames:** `/old-path` → `/new-path`.
- **Canonical URLs:** `/About` → `/about`.
- **Multi-tenant routing:** `/?tenant=x` → `/x/dashboard`.

### References

- React Router — Navigate — https://reactrouter.com/api/components/Navigate
- React Router — redirect — https://reactrouter.com/api/data-routers/redirect
- Next.js — redirect — https://nextjs.org/docs/app/api-reference/functions/redirect
- Next.js — Middleware (redirects) — https://nextjs.org/docs/app/building-your-application/routing/middleware

---

## 8. Route Guards

### Definitions

**Core Definition:** A route guard is a check that runs before a route renders or before navigation completes, deciding whether to allow, redirect, or block access.

**Technical Definition:** Route guards enforce access control at the routing layer. In React Router Data Mode, guards are `loader` functions that throw `redirect()`. In Declarative Mode, guards are wrapper components that conditionally render `<Navigate>` or `children`. In TanStack Router, guards are `beforeLoad` functions. In Next.js, guards are `middleware.ts` and server-side layout checks. Guards differ from in-component checks because they run **before** rendering, avoiding flicker and wasted work. Guards must be backed by server-side authorisation; they are a UX layer, not a security boundary.

**Beginner-Friendly Explanation:** A route guard is a bouncer at the door. Before you enter a page, the guard checks whether you are allowed in. If not, it sends you somewhere else.

### Purposes

- To prevent unauthorised users from rendering protected content.
- To centralise access logic instead of duplicating checks.
- To run authentication and authorisation checks before rendering.
- To redirect users to login or forbidden pages.
- To avoid flicker and wasted API calls from post-render checks.

### Syntax Rules and Structure

**React Router Data Mode (loader guard):**

```jsx
async function requireAuth({ request }) {
  const user = await getUser();
  if (!user) {
    const url = new URL(request.url);
    throw redirect(`/login?redirect=${encodeURIComponent(url.pathname)}`);
  }
  return { user };
}
```

**TanStack Router (`beforeLoad`):**

```tsx
export const Route = createFileRoute('/admin')({
  beforeLoad: async ({ context }) => {
    if (!context.auth.user) throw redirect({ to: '/login' });
    if (context.auth.user.role !== 'admin') throw redirect({ to: '/forbidden' });
  },
});
```

**React Router Declarative Mode (wrapper):**

```jsx
function Guard({ children }) {
  const user = useUser();
  if (!user) return <Navigate to="/login" replace />;
  return children;
}
```

**Syntax Rules:**

1. Run guards before rendering: loaders, `beforeLoad`, or middleware.
2. Throw/return redirects instead of rendering “please log in” inside protected content.
3. Check authentication first, then authorisation.
4. Preserve the intended destination for post-login return.
5. Back every client guard with server-side authorisation.

**Constraints and Limitations:**

- Client guards can be bypassed; never use them as the only security layer.
- Guards that run after render cause flicker and wasted API calls.
- Guards that perform slow async work delay navigation; keep them fast or move heavy checks to server middleware.

### Annotated Code Examples

```tsx
async function guard({ request, context }) {
  const user = await getUser();
  if (!user) throw redirect(`/login?redirect=${new URL(request.url).pathname}`);
  if (!user.permissions.includes('admin:access')) throw redirect('/forbidden');
  return { user };
}

const router = createBrowserRouter([
  {
    path: '/admin',
    loader: guard,
    element: <AdminLayout />,
  },
]);
```

**Expected Output:** Unauthenticated users go to `/login`; authenticated users without the permission go to `/forbidden`; authorised admins see the admin layout.

**Why this output:** The guard checks authentication and permission before rendering. The redirect target distinguishes the two failure modes.

### Real-World Cases

- **Auth gates:** Protecting all `/app/*` routes.
- **Role gates:** Admin-only sections.
- **Permission gates:** Subscription-tier features.
- **Onboarding gates:** Redirecting to onboarding until complete.

### References

- React Router — redirect — https://reactrouter.com/api/data-routers/redirect
- TanStack Router — Authenticated Routes — https://tanstack.com/router/latest/docs/framework/react/guide/authenticated-routes
- Next.js — Middleware — https://nextjs.org/docs/app/building-your-application/routing/middleware

---

## 9. Route Transitions

### Definitions

**Core Definition:** Route transitions are the visual and focus changes that occur when navigating from one route to another.

**Technical Definition:** Route transitions include pending UI (progress bars, spinners, skeleton screens), view transitions (the View Transitions API), scroll restoration, and focus management for accessibility. React Router exposes `useNavigation()` to detect pending navigation states. TanStack Router provides `useRouterState` and pending-match APIs. The browser View Transitions API can animate between routes in supporting browsers. Focus management moves focus to the main heading or a `tabIndex={-1}` container after navigation so screen-reader users start at the new content.

**Beginner-Friendly Explanation:** A route transition is what the user sees while the next page loads and after it appears — a spinner, a smooth fade, the scroll position resetting, and focus moving to the new content.

### Purposes

- To provide feedback during navigation (pending UI).
- To make navigation feel smooth and intentional.
- To reset scroll position appropriately on navigation.
- To move focus to new content for screen-reader users.
- To animate between views without jarring flashes.

### Syntax Rules and Structure

**Pending UI (React Router):**

```jsx
import { useNavigation } from 'react-router';

function Layout() {
  const navigation = useNavigation();
  const isNavigating = navigation.state === 'loading';

  return (
    <>
      {isNavigating && <TopProgressBar />}
      <Outlet />
    </>
  );
}
```

**Scroll restoration and focus:**

```jsx
useEffect(() => {
  window.scrollTo(0, 0);
  mainRef.current?.focus();
}, [location.pathname]);
```

**View Transition (browser API):**

```js
document.startViewTransition(() => navigate('/next'));
```

**Syntax Rules:**

1. Use the router's pending-navigation API for progress indicators.
2. Scroll to the top on pathname change unless using the router's scroll restoration.
3. Move focus to the main heading or a `tabIndex={-1}` container after navigation.
4. Use `startViewTransition` where supported for animated transitions.
5. Announce route changes with an `aria-live` region for screen-reader users.

**Constraints and Limitations:**

- View Transitions API support varies by browser.
- Forced scroll restoration can fight the browser's native restoration; choose one strategy.
- Focus management must not steal focus from inputs the user is interacting with.

### Annotated Code Examples

```tsx
function ScrollAndFocusReset() {
  const { pathname } = useLocation();
  const mainRef = useRef(null);

  useEffect(() => {
    window.scrollTo(0, 0);
    mainRef.current?.focus();
  }, [pathname]);

  return <main ref={mainRef} tabIndex={-1}>{/* routed content */}</main>;
}
```

**Expected Output:** After each navigation, the page scrolls to the top and focus moves to the main content, so keyboard and screen-reader users start at the new page.

**Why this output:** The effect runs on `pathname` change. `scrollTo(0, 0)` resets scroll. `focus()` moves focus to the `main` element, which has `tabIndex={-1}` to be programmatically focusable.

### Real-World Cases

- **Content sites:** Scroll-to-top on article navigation.
- **SPAs:** Top progress bars during data loading.
- **Accessibility:** Focus reset on route change for screen-reader users.
- **Marketing sites:** Animated page transitions.

### References

- React Router — useNavigation — https://reactrouter.com/api/hooks/useNavigation
- MDN — View Transition API — https://developer.mozilla.org/en-US/docs/Web/API/View_Transition_API
- TanStack Router — Pending Matches — https://tanstack.com/router/latest/docs/framework/react/guide/pending-matches

---

## 10. Route Animations

### Definitions

**Core Definition:** Route animations are the visual effects applied when navigating between routes, such as fades, slides, or shared-element transitions.

**Technical Definition:** Route animations are implemented with CSS transitions, the View Transitions API, or animation libraries (Framer Motion, React Transition Group). They require the router to keep the outgoing and incoming routes in the DOM simultaneously during the transition, or to capture a snapshot of the old view. The View Transitions API provides a browser-native mechanism (`document.startViewTransition`) that snapshots the old and new views and animates between them. Animation libraries offer more control but require the router's location to be stable during the transition.

**Beginner-Friendly Explanation:** Route animations make page changes feel smooth. Instead of the content instantly swapping, it fades or slides to the new page.

### Purposes

- To make navigation feel polished and intentional.
- To provide visual continuity between pages.
- To reduce the jarring effect of instant content swaps.
- To draw attention to the new content.
- To support shared-element transitions (e.g., image expanding into a detail view).

### Syntax Rules and Structure

**View Transitions API:**

```js
function navigateWithTransition(to) {
  if (document.startViewTransition) {
    document.startViewTransition(() => navigate(to));
  } else {
    navigate(to);
  }
}
```

**CSS for view transitions:**

```css
::view-transition-old(root) {
  animation: fade-out 200ms ease-in;
}
::view-transition-new(root) {
  animation: fade-in 200ms ease-out;
}
```

**Animation library (Framer Motion):**

```jsx
import { AnimatePresence, motion } from 'framer-motion';

<AnimatePresence mode="wait">
  <motion.div
    key={location.pathname}
    initial={{ opacity: 0 }}
    animate={{ opacity: 1 }}
    exit={{ opacity: 0 }}
  >
    <Routes location={location} />
  </motion.div>
</AnimatePresence>
```

**Syntax Rules:**

1. Use the View Transitions API for simple, browser-native transitions.
2. Use an animation library for orchestrated transitions and shared elements.
3. Respect `prefers-reduced-motion`; disable animations for users who prefer them.
4. Keep transitions short (200–300ms) to avoid feeling sluggish.
5. Ensure the router's location is stable during the animation (or pass a stable `location` to `<Routes>`).

**Constraints and Limitations:**

- View Transitions API support varies by browser.
- Animation libraries add bundle size.
- Animating between routes can conflict with scroll restoration and focus management.
- Heavy animations can degrade performance on low-end devices.

### Annotated Code Examples

```tsx
import { AnimatePresence, motion } from 'framer-motion';
import { useLocation, Routes, Route } from 'react-router';

function AnimatedRoutes() {
  const location = useLocation();

  return (
    <AnimatePresence mode="wait">
      <motion.div
        key={location.pathname}
        initial={{ opacity: 0, y: 8 }}
        animate={{ opacity: 1, y: 0 }}
        exit={{ opacity: 0, y: -8 }}
        transition={{ duration: 0.2 }}
      >
        <Routes location={location}>
          <Route path="/" element={<Home />} />
          <Route path="/about" element={<About />} />
        </Routes>
      </motion.div>
    </AnimatePresence>
  );
}
```

**Expected Output:** Navigating between `/` and `/about` fades and slides the new page in while the old page fades out.

**Why this output:** `AnimatePresence` keeps the exiting route mounted during the exit animation. `key={location.pathname}` tells it which route changed. Passing `location` to `<Routes>` ensures the correct route renders during the animation.

### Real-World Cases

- **Marketing sites:** Polished page transitions.
- **Portfolios:** Shared-element image transitions.
- **E-commerce:** Smooth product detail transitions.
- **Dashboards:** Subtle tab transitions.

### References

- MDN — View Transition API — https://developer.mozilla.org/en-US/docs/Web/API/View_Transition_API
- Framer Motion — AnimatePresence — https://motion.dev
- React Router — useLocation — https://reactrouter.com/api/hooks/useLocation

---

## 11. Route Prefetching

### Definitions

**Core Definition:** Route prefetching is loading a route's code or data before the user navigates to it, so the navigation feels instant.

**Technical Definition:** Prefetching can target **code** (the route's lazy-loaded chunk) and **data** (loader results or query data). Triggers include hover (`mouseenter`), focus, viewport visibility (Intersection Observer), and idle time (`requestIdleCallback`). React Router v7 supports `prefetch` on links in framework mode. TanStack Router supports `defaultPreload: 'intent' | 'viewport' | 'render'` and per-link `preload`. The browser deduplicates repeated `import()` calls, so prefetching the same chunk twice is safe.

**Beginner-Friendly Explanation:** Prefetching starts loading the next page before the user clicks. When they hover over a link or when the link scrolls into view, the router fetches the code and data in the background.

### Purposes

- To make navigations feel instant.
- To use idle time and user intent as preloading signals.
- To avoid loading waterfalls on navigation.
- To improve perceived performance without blocking the current page.
- To reduce the impact of lazy loading on the first navigation.

### Syntax Rules and Structure

**Hover-based prefetch:**

```jsx
<Link
  to="/dashboard"
  onMouseEnter={() => import('./pages/Dashboard')}
>
  Dashboard
</Link>
```

**React Router framework mode:**

```jsx
<Link to="/dashboard" prefetch="intent">Dashboard</Link>
```

**TanStack Router global preload:**

```tsx
const router = createRouter({
  routeTree,
  defaultPreload: 'intent',
  defaultPreloadDelay: 50,
});
```

**TanStack Router per-link preload:**

```tsx
<Link to="/dashboard" preload="viewport">Dashboard</Link>
```

**Syntax Rules:**

1. Prefetch on intent (hover, focus) rather than on mount for all links.
2. Use viewport-based prefetching for links likely to be clicked (e.g., visible cards).
3. Use `defaultPreloadDelay` to avoid prefetching on accidental hovers.
4. Prefetch data with the same query/loader cache used by the destination.
5. Do not prefetch routes the user is unlikely to visit.

**Constraints and Limitations:**

- Aggressive prefetching wastes bandwidth and server resources.
- Prefetched data can become stale; use the router/query's staleness settings.
- Prefetching is an optimisation, not a guarantee; the route must still handle loading.

### Annotated Code Examples

```tsx
function Navigation() {
  return (
    <nav>
      <Link
        to="/dashboard"
        prefetch="intent"
        onMouseEnter={() => import('./pages/Dashboard')}
      >
        Dashboard
      </Link>
      <Link to="/reports" prefetch="viewport">Reports</Link>
    </nav>
  );
}
```

**Expected Output:** Hovering over “Dashboard” prefetches its chunk. Scrolling “Reports” into view prefetches its chunk via the viewport strategy.

**Why this output:** `prefetch="intent"` and `onMouseEnter` trigger the prefetch on hover. `prefetch="viewport"` uses Intersection Observer to prefetch when the link becomes visible.

### Real-World Cases

- **Dashboards:** Prefetching the most-used routes on hover.
- **E-commerce:** Prefetching product pages from listing cards.
- **Content sites:** Prefetching articles when links enter the viewport.
- **Multi-step flows:** Prefetching the next step based on intent.

### References

- React Router — Prefetching — https://reactrouter.com/start/framework/links
- TanStack Router — Preloading — https://tanstack.com/router/latest/docs/framework/react/guide/preloading
- Next.js — Linking and Navigating (prefetching) — https://nextjs.org/docs/app/building-your-application/routing/linking-and-navigating

---

## 12. Route Lazy Loading

### Definitions

**Core Definition:** Route lazy loading is deferring the loading of a route's component code until the route is navigated to.

**Technical Definition:** Lazy loading uses `React.lazy` with `Suspense` in Declarative Mode, the `lazy` route property in React Router Data Mode (a function returning a Promise of route properties), or the `lazy` property in TanStack Router. The route's component (and optionally its loader) is fetched on first navigation. This reduces the initial bundle size and improves initial load performance. The trade-off is a network round trip on first navigation, which can be mitigated with prefetching.

**Beginner-Friendly Explanation:** Lazy loading means the code for a route is only downloaded when you navigate to it, not upfront.

### Purposes

- To reduce the initial bundle size.
- To load route code on demand.
- To improve initial load performance.
- To split code by route boundaries.
- To load loader, action, and component code in parallel.

### Syntax Rules and Structure

**React Router Declarative Mode:**

```jsx
const Dashboard = lazy(() => import('./pages/Dashboard'));

<Suspense fallback={<PageSkeleton />}>
  <Routes>
    <Route path="/dashboard" element={<Dashboard />} />
  </Routes>
</Suspense>
```

**React Router Data Mode:**

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

**TanStack Router:**

```tsx
// The bundler plugin can lazy-load route components automatically
export const Route = createFileRoute('/reports')({
  component: lazyRouteComponent(() => import('./pages/Reports')),
});
```

**Syntax Rules:**

1. Use `React.lazy` + `Suspense` in Declarative Mode.
2. Use the `lazy` route property in Data Mode; load component and loader in parallel with `Promise.all`.
3. Use the router's native lazy route mechanism where available (TanStack Router, Next.js).
4. Provide meaningful fallbacks; do not lazy-load above-the-fold content.
5. Combine with prefetching to avoid the first-navigation delay.

**Constraints and Limitations:**

- Adds a network round trip on first navigation.
- Too many tiny route chunks hurt compression.
- Lazy components cannot be server-rendered directly without framework support.

### Annotated Code Examples

```jsx
const Reports = lazy(() => import('./pages/Reports'));

function App() {
  return (
    <Suspense fallback={<PageSkeleton />}>
      <Routes>
        <Route path="/reports" element={<Reports />} />
      </Routes>
    </Suspense>
  );
}
```

**Expected Output:** The reports chunk is downloaded only when the user navigates to `/reports`. The skeleton is shown while it loads.

**Why this output:** `lazy` creates a split point. The chunk is fetched on first render of the route. `Suspense` provides the fallback.

### Real-World Cases

- **Large admin sections:** Loading admin code only when accessed.
- **Heavy reports:** Deferring report components and loaders.
- **Settings pages:** Loading settings code on demand.
- **Public marketing pages:** Keeping the landing page bundle small.

### References

- React — lazy — https://react.dev/reference/react/lazy
- React Router — Route Object: lazy — https://reactrouter.com/start/data/route-object
- TanStack Router — Lazy Loading — https://tanstack.com/router/latest/docs/framework/react/guide/code-splitting
- Next.js — Lazy Loading — https://nextjs.org/docs/app/building-your-application/optimizing/lazy-loading

---

## 13. Route Code Splitting

### Definitions

**Core Definition:** Route code splitting is the build-time partitioning of the application into separate chunks per route, so each route's code is downloaded only when needed.

**Technical Definition:** Route code splitting is the bundler-level implementation of lazy loading. The bundler analyses dynamic imports and router lazy configurations, emits a separate chunk per route, and generates a manifest. Route-level splitting is the highest-impact loading optimisation for SPAs, reducing the initial bundle by 40–70%. Vendor splitting isolates stable dependencies into separately cacheable chunks. React Router Data Mode's `lazy`, TanStack Router's file-based route splitting, and Next.js's automatic route-based splitting all implement this pattern.

**Beginner-Friendly Explanation:** Route code splitting means each page gets its own file. Users download only the page they are visiting, not the whole app.

### Purposes

- To reduce the initial JavaScript payload.
- To improve load performance and Largest Contentful Paint.
- To raise cache hit rates by isolating stable vendor code.
- To keep route chunks small and focused.
- To enable on-demand loading of rarely used routes.

### Syntax Rules and Structure

**Route-level lazy splitting:**

```jsx
const Home = lazy(() => import('./pages/Home'));
const Dashboard = lazy(() => import('./pages/Dashboard'));
```

**Vendor splitting (Vite):**

```js
manualChunks: {
  'vendor-react': ['react', 'react-dom'],
  'vendor-router': ['react-router'],
}
```

**Next.js automatic splitting:**

```text
app/
├── page.tsx            → own chunk
├── dashboard/page.tsx  → own chunk
└── reports/page.tsx    → own chunk
```

**Syntax Rules:**

1. Split at route boundaries; each route should be its own chunk.
2. Isolate `node_modules` dependencies into stable vendor chunks.
3. Avoid creating too many tiny chunks (HTTP/2 sweet spot ~10–30 chunks).
4. Name chunks for readable build output.
5. Use framework/router-native splitting where available.

**Constraints and Limitations:**

- Too many small chunks hurt compression and increase request overhead.
- Chunk loading failures must be handled (e.g., stale deploy → reload prompt).
- Code splitting does not reduce the total code; it changes when it loads.

### Annotated Code Examples

```jsx
// Vite vendor splitting + route splitting
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          'vendor-react': ['react', 'react-dom'],
          'vendor-router': ['react-router'],
        },
      },
    },
  },
});
```

**Expected Output:** React and React Router are bundled into cacheable vendor chunks. Each route is a separate chunk loaded on navigation.

**Why this output:** `manualChunks` separates stable dependencies from application code. Route-level `lazy` creates the route chunks.

### Real-World Cases

- **SPAs:** Route-based splitting plus vendor splitting.
- **E-commerce:** Product, cart, and checkout chunks.
- **Dashboards:** Each section loaded on demand.
- **Content sites:** Article chunks loaded per route.

### References

- web.dev — Code splitting — https://web.dev/articles/reduce-javascript-payloads-with-code-splitting
- React Router — Lazy — https://reactrouter.com/api/data-routers/lazy
- TanStack Router — Code Splitting — https://tanstack.com/router/latest/docs/framework/react/guide/code-splitting
- Next.js — Code Splitting — https://nextjs.org/docs/app/building-your-application/optimizing/lazy-loading

---

## 14. Route Error Handling

### Definitions

**Core Definition:** Route error handling is the practice of catching and displaying errors that occur during route loading, rendering, or data fetching, at the appropriate route segment.

**Technical Definition:** Route error handling has three layers. **404 handling** uses catch-all routes (`*` in React Router, `not-found.tsx` in Next.js, `notFound()` in loaders) for unmatched or missing resources. **Error boundaries** catch rendering and loader errors: `errorElement` in React Router Data Mode, `errorComponent` in TanStack Router, `error.tsx` in Next.js. **Loader-thrown errors** use `throw new Response()` or `throw redirect()` to signal HTTP errors and redirects. Errors bubble to the nearest boundary, so placing boundaries at meaningful segments isolates failures.

**Beginner-Friendly Explanation:** Route error handling means each part of the app has a safety net. If a page fails to load, shows an error, or the URL does not exist, the user sees a helpful message instead of a broken page.

### Purposes

- To catch errors from loaders, actions, and rendering.
- To display user-friendly error UI instead of crashes.
- To isolate failures to the smallest meaningful segment.
- To provide a 404 page for unmatched routes.
- To allow retry and recovery without a full reload.

### Syntax Rules and Structure

**React Router Data Mode:**

```jsx
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

**TanStack Router:**

```tsx
export const Route = createFileRoute('/users/$id')({
  loader: ({ params }) => fetchUser(params.id),
  errorComponent: ({ error }) => <UserError error={error} />,
});
```

**Next.js:**

```tsx
// app/users/[id]/error.tsx
'use client';
export default function Error({ error, reset }) {
  return <button onClick={reset}>Try again</button>;
}
```

**Syntax Rules:**

1. Add an error boundary to meaningful route segments.
2. Use a catch-all route (`*`) or `not-found.tsx` for unmatched paths.
3. Use `isRouteErrorResponse` (React Router) to distinguish HTTP errors from unexpected errors.
4. Throw `Response` or `notFound()` for missing resources in loaders.
5. Provide retry (`reset` / `router.invalidate()`) where appropriate.

**Constraints and Limitations:**

- Error boundaries do not catch errors in event handlers or async code outside loaders.
- A root error boundary is a catch-all; segment-level boundaries isolate failures better.
- `reset()` in Next.js does not refetch data; use `router.refresh()` for that.

### Annotated Code Examples

```tsx
const router = createBrowserRouter([
  {
    path: '/',
    Component: Root,
    errorElement: <RootError />,
    children: [
      {
        path: 'users/:id',
        Component: UserDetail,
        loader: async ({ params }) => {
          const user = await fetchUser(params.id);
          if (!user) throw new Response('Not Found', { status: 404 });
          return { user };
        },
        errorElement: <UserError />,
      },
      { path: '*', element: <NotFound /> },
    ],
  },
]);
```

**Expected Output:** A missing user renders `UserError` with a 404 message. Unmatched URLs render `NotFound`. Other errors bubble to `RootError`.

**Why this output:** The route-level `errorElement` catches loader errors for `/users/:id`. The wildcard route catches unmatched paths. The root `errorElement` is the final catch-all.

### Real-World Cases

- **Content sites:** 404 pages for missing articles.
- **E-commerce:** Product-not-found handling.
- **Dashboards:** Isolated widget error boundaries.
- **API-driven apps:** Loader error boundaries with retry.

### References

- React Router — Error Boundaries — https://reactrouter.com/start/data/error-boundaries
- React Router — useRouteError — https://reactrouter.com/api/hooks/useRouteError
- TanStack Router — Error Handling — https://tanstack.com/router/latest/docs/framework/react/guide/error-handling
- Next.js — error.js — https://nextjs.org/docs/app/api-reference/file-conventions/error

---

## 15. Route Best Practices

### Definitions

**Core Definition:** Route best practices are the guidelines for designing, securing, and operating routes to build maintainable, performant, accessible, and secure applications.

**Technical Definition:** Best practices span access control (server-enforced authorisation, guards before render), URL design (semantic paths, resource identity in the path, view state in search params), data strategy (parallel loaders, caching, prefetching), error handling (segmented boundaries, 404 routes), performance (route-level splitting, vendor chunks, prefetching on intent), and accessibility (focus management, route announcements, skip links). The choice of router and rendering model should follow from the application's requirements, not from familiarity.

**Beginner-Friendly Explanation:** Good routing makes an app safe, fast, and easy to use. Protected pages are actually protected by the server, URLs make sense, pages load quickly, errors are handled, and screen-reader users know when the page changes.

### Best Practices Table

| Practice | Description |
|----------|-------------|
| **Enforce auth on the server** | Client guards are UX; server checks are security. |
| **Run guards before rendering** | Loaders, `beforeLoad`, or middleware; avoid flicker. |
| **Preserve intended destination** | Redirect to login with a `redirect` param. |
| **Keep auth routes public** | Avoid redirect loops on login/signup. |
| **Distinguish auth from authorisation** | Login check first, permission check second. |
| **Use semantic URLs** | `/products/:slug` rather than `/p?id=123`. |
| **Use nested layouts** | Parent layout + outlet for shared UI. |
| **Split code per route** | Route-level lazy loading for smaller bundles. |
| **Prefetch on intent** | Hover/focus/viewport prefetching for instant navigation. |
| **Parallelise loaders** | Nested loaders run in parallel; avoid waterfalls. |
| **Segment error boundaries** | `errorElement`/`error.tsx` at meaningful segments. |
| **Provide a 404 route** | Catch-all route with a helpful not-found page. |
| **Manage focus and scroll** | Move focus to main content; reset or restore scroll. |
| **Respect reduced motion** | Disable animations for users who prefer them. |
| **Handle chunk-load failures** | Stale deploys can cause import errors; show a reload prompt. |

### Annotated Code Examples

**✅ Good Practices:**

```tsx
// Server-enforced auth + client guard
async function requireAuth({ request }) {
  const user = await getServerSession(request); // Server-side check
  if (!user) throw redirect(`/login?redirect=${new URL(request.url).pathname}`);
  return { user };
}

const router = createBrowserRouter([
  {
    path: '/app',
    loader: requireAuth,
    Component: AppLayout,
    errorElement: <AppError />,
    children: [
      { index: true, Component: Dashboard },
      { path: 'settings', Component: Settings, lazy: () => import('./pages/Settings') },
    ],
  },
  { path: '/login', Component: Login },  // Public
  { path: '*', Component: NotFound },    // 404
]);
```

**❌ Bad Practices:**

```tsx
// ❌ Client-only auth after rendering
function Admin() {
  const user = useUser();
  if (!user) return <p>Please log in</p>; // Page already rendered
  return <AdminPanel />;
}

// ❌ Protecting the login route
<Route path="/login" loader={requireAuth} element={<Login />} />

// ❌ No error boundary
<Route path="/dashboard" element={<Dashboard />} />

// ❌ No 404 route
<Routes>{/* no catch-all */}</Routes>
```

**Expected Output:** The good practices produce secure, accessible, fast routes. The bad practices render protected content before checking access, cause redirect loops, crash on errors, and leave users stranded on unknown URLs.

**Why this output:** Client-only guards render before the check and can be bypassed. Protecting login creates a loop. Missing error boundaries crash the app. Missing 404 routes leave users stuck.

### Real-World Cases

- **All React applications:** These practices apply universally.
- **Multi-tenant SaaS:** Server-enforced auth and role checks.
- **E-commerce:** Semantic URLs, route splitting, and prefetching.
- **Content platforms:** 404 handling, error boundaries, and focus management.

### References

- React Router — Error Boundaries — https://reactrouter.com/start/data/error-boundaries
- React Router — Route Object — https://reactrouter.com/start/data/route-object
- TanStack Router — Authenticated Routes — https://tanstack.com/router/latest/docs/framework/react/guide/authenticated-routes
- Next.js — Middleware — https://nextjs.org/docs/app/building-your-application/routing/middleware
- web.dev — Code splitting — https://web.dev/articles/reduce-javascript-payloads-with-code-splitting
- W3C WAI — Page Titled (WCAG 2.4.2) — https://www.w3.org/WAI/WCAG21/Understanding/page-titled.html

---

## References

- React Router — Official Documentation — https://reactrouter.com
- React Router — Route Object — https://reactrouter.com/start/data/route-object
- React Router — Error Boundaries — https://reactrouter.com/start/data/error-boundaries
- React Router — redirect — https://reactrouter.com/api/data-routers/redirect
- React Router — Navigate — https://reactrouter.com/api/components/Navigate
- React Router — useNavigation — https://reactrouter.com/api/hooks/useNavigation
- React Router — useLocation — https://reactrouter.com/api/hooks/useLocation
- React Router — useSearchParams — https://reactrouter.com/api/hooks/useSearchParams
- React Router — useRouteError — https://reactrouter.com/api/hooks/useRouteError
- React Router — Lazy — https://reactrouter.com/api/data-routers/lazy
- React Router — Prefetching — https://reactrouter.com/start/framework/links
- TanStack Router — Authenticated Routes — https://tanstack.com/router/latest/docs/framework/react/guide/authenticated-routes
- TanStack Router — Preloading — https://tanstack.com/router/latest/docs/framework/react/guide/preloading
- TanStack Router — Code Splitting — https://tanstack.com/router/latest/docs/framework/react/guide/code-splitting
- TanStack Router — Error Handling — https://tanstack.com/router/latest/docs/framework/react/guide/error-handling
- TanStack Router — Pending Matches — https://tanstack.com/router/latest/docs/framework/react/guide/pending-matches
- Next.js — Middleware — https://nextjs.org/docs/app/building-your-application/routing/middleware
- Next.js — redirect — https://nextjs.org/docs/app/api-reference/functions/redirect
- Next.js — error.js — https://nextjs.org/docs/app/api-reference/file-conventions/error
- Next.js — Lazy Loading — https://nextjs.org/docs/app/building-your-application/optimizing/lazy-loading
- Next.js — Linking and Navigating — https://nextjs.org/docs/app/building-your-application/routing/linking-and-navigating
- React — lazy — https://react.dev/reference/react/lazy
- MDN — View Transition API — https://developer.mozilla.org/en-US/docs/Web/API/View_Transition_API
- web.dev — Code splitting — https://web.dev/articles/reduce-javascript-payloads-with-code-splitting
- W3C WAI — Page Titled (WCAG 2.4.2) — https://www.w3.org/WAI/WCAG21/Understanding/page-titled.html
- Clerk — Protected Routes — https://clerk.com/docs
- Clerk — Public Routes — https://clerk.com/docs
- Framer Motion — AnimatePresence — https://motion.dev