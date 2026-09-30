# Authorization & Access Control in React: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Authorization & Access Control in React is the discipline of determining what an authenticated user is permitted to do within a React application, and enforcing those permissions consistently across the UI layer, the routing layer, and the server boundary.

**Technical Definition:** Authorization (AuthZ) is distinct from authentication (AuthN). Authentication answers "Who are you?" while authorization answers "What are you allowed to do?" In React applications, authorization is implemented through a combination of access control models (RBAC, PBAC, ABAC), declarative UI components that conditionally render based on permissions, route guards that block unauthorized navigation, and—critically—server-side enforcement that treats all client-side checks as UX affordances rather than security boundaries. The authoritative enforcement point must always be the server or a server-only data access layer. Client-side authorization controls only what the user interface shows; it does not protect data or operations. 

**Beginner-Friendly Explanation:** Once a user logs in, you need to decide what they're allowed to see and do. A regular user shouldn't see the "Delete All Users" button, and a guest shouldn't access the admin dashboard. Authorization is about setting those rules and making sure they're followed—both in what you show on screen and, more importantly, in what your server actually allows. Hiding a button is like putting a "Staff Only" sign on a door; it's helpful for users, but the real lock is on the server. 

### Key Characteristics

- **Separation of Concerns:** The UI decides what to show (UX); the server decides what to allow (security). These are two different jobs that must agree but serve different purposes. 
- **Defence in Depth:** Authorization is enforced at multiple layers—UI components, route guards, middleware, server components, and API handlers—with the server as the authoritative boundary. 
- **Model Flexibility:** Simple applications use Role-Based Access Control (RBAC); complex applications use Attribute-Based Access Control (ABAC) or Policy-Based Access Control (PBAC) for fine-grained, context-aware decisions. 
- **Declarative Abstraction:** Permissions are expressed declaratively through wrapper components (e.g., `<Can>`, `<Allow>`) and hooks (e.g., `useCan`, `useAbility`), keeping permission logic out of component internals. 
- **Single Source of Truth:** The same policy definition should drive both client-side UI rendering and server-side enforcement to prevent drift between what the UI offers and what the server permits. 
- **Fail-Safe Defaults:** When no policy matches, access is denied. Explicit deny rules override allow rules. 

### Prerequisites

- Solid understanding of React components, Hooks, and context.
- Familiarity with React Router (v6+) or Next.js routing for route protection.
- Knowledge of authentication flows (JWT, session cookies) to understand the authenticated user context.
- Basic understanding of HTTP status codes (401 Unauthorized, 403 Forbidden).
- Awareness of state management patterns (Context, Zustand) for sharing permission state.

### Related Programming Areas

- **Authentication:** Establishing user identity before authorization can occur.
- **State Management:** Sharing permission state across components.
- **Routing:** Protecting routes at the router or middleware level.
- **API Security:** Enforcing authorization at the API or data access layer.
- **Policy Engines:** External authorization services (Permit.io, Open Policy Agent).
- **Security Auditing:** Logging denied attempts and testing "should not access" scenarios.

### Core Concepts / Features

1. Access Control Models (RBAC & PBAC/ABAC)
2. Declarative Protected Components
3. Protected Routes & Navigation
4. UI Condensation vs. Server Enforcement

---

## Core Concept 1: Access Control Models

### Definitions

**Core Definition:** Access Control Models are the conceptual frameworks—Role-Based Access Control (RBAC) and Attribute/Permission-Based Access Control (PBAC/ABAC)—that define how permissions are assigned to users and evaluated when access decisions are made.

**Technical Definition:** **RBAC (Role-Based Access Control)** assigns permissions to roles rather than individual users. A user is assigned one or more roles (e.g., admin, editor, viewer), and each role is associated with a set of allowed actions. Access is granted if the user's role includes the required permission. **PBAC (Policy-Based Access Control)** and **ABAC (Attribute-Based Access Control)** make access decisions based on multiple attributes: user attributes (role, department, access level), resource attributes (owner, status, visibility), and environmental attributes (time, location). Policies are defined as rules that map subjects, objects, actions, and conditions. ABAC is more scalable than RBAC for complex applications because it enables fine-grained decisions such as "a marketing manager can delete a post only if it is archived and they own it." 

**Beginner-Friendly Explanation:** RBAC is like giving someone a key based on their job title—managers get a master key, employees get a regular key. ABAC is more like a smart lock that checks not just who you are, but also what time it is, which door you're trying to open, and whether you own the room. RBAC is simpler; ABAC is more flexible but requires more setup. 

### Purposes

- To define a structured, maintainable system for assigning and evaluating permissions.
- To support simple role hierarchies (RBAC) or complex, context-aware policies (ABAC/PBAC).
- To enable fine-grained access decisions based on resource ownership, status, or environmental conditions.
- To centralize permission definitions so they can be shared between client and server.
- To scale authorization logic as the application grows in complexity and user roles.
- To support explicit deny rules that override broad allow permissions.

### Syntax Rules and Structure

**General Syntax for RBAC Definition:**
```javascript
// RBAC policy: roles mapped to allowed actions
const roles = {
  admin: {
    actions: ['create', 'edit', 'delete', 'view'],
  },
  editor: {
    actions: ['create', 'edit', 'view'],
  },
  viewer: {
    actions: ['view'],
  },
};
```

**Component Breakdown:**
- `roles`: An object mapping role names to their allowed actions.
- `actions`: An array of permitted action strings for each role.
- Access is granted if the user's role includes the required action.

**General Syntax for ABAC Policy:**
```javascript
// ABAC policy: rules based on user, resource, and action attributes
const accessPolicies = [
  {
    subject: { role: ['admin'] },
    object: { resourceType: 'post', ownership: 'any' },
    action: ['create', 'read', 'update', 'delete'],
  },
  {
    subject: { role: ['editor'], department: ['Marketing'] },
    object: { resourceType: 'post', ownership: 'own', status: ['archived'] },
    action: ['delete'],
  },
  {
    subject: { role: ['viewer'] },
    object: { resourceType: 'post', visibility: 'public' },
    action: ['read'],
  },
];
```

**Component Breakdown:**
- `subject`: Conditions on the user (role, department, etc.).
- `object`: Conditions on the resource (type, ownership, status, visibility).
- `action`: The permitted actions when all conditions match.
- Access is granted if any policy matches all subject, object, and action conditions. 

**Syntax Rules:**
- Define policies as data (JSON or JavaScript objects), not as imperative code, so they can be shared between client and server.
- Use explicit deny rules to override broad allows: an explicit deny always takes precedence. 
- For RBAC, keep role definitions small and composable; avoid deep role hierarchies.
- For ABAC, evaluate policies in order and return on the first match (or use a deny-by-default strategy).
- The policy engine should be a pure function of `(user, resource, action) → decision` with no side effects. 

**Constraints and Limitations:**
- RBAC cannot express ownership-based rules ("edit your own posts") without additional logic.
- ABAC policies can become complex and difficult to audit if not structured carefully.
- ABAC evaluation requires access to user and resource attributes at decision time; missing attributes may cause incorrect denials.
- Client-side policy evaluation is for UX only; the server must independently evaluate the same policy.
- Role explosion is a common RBAC anti-pattern; prefer fine-grained permissions where possible.

### Annotated Code Examples

**Example 1: RBAC with a Shared Policy Engine**

```jsx
// policy.js — shared between client and server
const roles = {
  admin:  { actions: ['create', 'edit', 'delete', 'view'] },
  editor: { actions: ['create', 'edit', 'view'] },
  viewer: { actions: ['view'] },
};

// Pure function: (role, action) → boolean
function can(role, action) {
  const allowed = roles[role]?.actions || [];
  return allowed.includes(action);
}

export { roles, can };
```
```jsx
// React component using the policy
import React from 'react';
import { can } from './policy';

function PostActions({ userRole }) {
  return (
    <div>
      {can(userRole, 'view') && <button>View Post</button>}
      {can(userRole, 'edit') && <button>Edit Post</button>}
      {can(userRole, 'delete') && <button>Delete Post</button>}
    </div>
  );
}
```
```jsx
// Server-side enforcement (Express)
import { can } from './policy';

function requireAction(action) {
  return (req, res, next) => {
    const userRole = req.user?.role;
    if (!can(userRole, action)) {
      return res.status(403).json({ error: 'Forbidden' });
    }
    next();
  };
}

app.delete('/api/posts/:id', requireAction('delete'), deletePostHandler);
```

**Expected Output:** A user with the `admin` role sees all three buttons (View, Edit, Delete). An `editor` sees View and Edit but not Delete. A `viewer` sees only View. On the server, a request to delete a post from a viewer returns a 403 Forbidden response.

**Why This Output Occurs:** The `can()` function is a pure policy evaluation that checks whether the role's action list includes the requested action. The same function is used on both the client (to conditionally render buttons) and the server (to reject unauthorized requests). This ensures the UI and the server agree on permissions without duplicating logic. 

**Example 2: ABAC with Attribute-Based Policy Evaluation**

```javascript
// accessPolicies.js
const accessPolicies = [
  {
    subject: { role: ['admin'] },
    object: { resourceType: 'post', ownership: 'any' },
    action: ['create', 'read', 'update', 'delete'],
  },
  {
    subject: { role: ['editor'], department: ['Marketing'] },
    object: { resourceType: 'post', ownership: 'own', status: ['archived'] },
    action: ['delete'],
  },
  {
    subject: { role: ['viewer'] },
    object: { resourceType: 'post', visibility: 'public' },
    action: ['read'],
  },
];

// Centralized access control function
function checkAccess(user, resource, action) {
  if (!user) return false;

  return accessPolicies.some((policy) => {
    const { subject, object, action: allowedActions } = policy;

    // Check role
    if (!subject.role.includes(user.role)) return false;

    // Check department (if applicable)
    if (subject.department && !subject.department.includes(user.department)) {
      return false;
    }

    // Check resource type
    if (object.resourceType !== resource.type) return false;

    // Check ownership
    if (object.ownership === 'own' && resource.ownerId !== user.id) return false;

    // Check status
    if (object.status && !object.status.includes(resource.status)) return false;

    // Check visibility
    if (object.visibility && resource.visibility !== object.visibility) return false;

    // Check allowed actions
    return allowedActions.includes(action);
  });
}

// React component
function PostActions({ user, post }) {
  const actions = ['read', 'update', 'delete'];
  return (
    <div>
      {actions.map((action) => (
        checkAccess(user, post, action) && (
          <button key={action}>{action}</button>
        )
      ))}
    </div>
  );
}
```

**Expected Output:** An admin sees Read, Update, and Delete buttons for any post. A Marketing editor sees Delete only for their own archived posts. A viewer sees Read only for public posts.

**Why This Output Occurs:** The `checkAccess` function evaluates each policy rule against the user, resource, and action. All conditions in a rule must match for access to be granted. This enables fine-grained decisions that RBAC cannot express, such as ownership-based and status-based access. 

### Real-World Cases

- **Enterprise admin panels:** Using RBAC with roles like Admin, Manager, and Viewer to control access to user management, billing, and analytics features.
- **Content management systems:** Using ABAC to allow editors to delete only their own archived posts, while admins can manage all content.
- **Multi-tenant SaaS:** Using ABAC with tenant attributes to ensure users can only access resources within their organisation.
- **Healthcare applications:** Using ABAC with department, role, and patient-assignment attributes to enforce HIPAA-compliant access.
- **Financial platforms:** Using PBAC with policies that consider transaction amount, user role, and time of day for approvals.

---

## Core Concept 2: Declarative Protected Components

### Definitions

**Core Definition:** Declarative Protected Components are wrapper components (such as `<Can>`, `<Allow>`, or `<CanAccess>`) that conditionally render their children based on the current user's permissions, abstracting permission checks away from individual component logic.

**Technical Definition:** Declarative protected components integrate with an authorization library (such as CASL, react-rbac, or a custom permission provider) to expose a simple JSX API for conditional rendering. The component receives the permission requirements as props (e.g., `action="edit"`, `subject="post"`) and evaluates them against the current user's ability. If the permission check passes, the children are rendered; if not, the component renders nothing or a provided fallback. The `AbilityProvider` or `PermissionsProvider` at the application root supplies the user's ability instance through React context. This pattern keeps permission logic out of component internals and makes the UI declarative. 

**Beginner-Friendly Explanation:** Instead of writing `if (user.can('edit', 'post'))` inside every component, you wrap the parts of your UI that require permission in a `<Can>` component. The `<Can>` component handles the check for you: if the user has permission, it shows the content; if not, it hides it. It's like having a bouncer at the door of each UI element who checks the guest list automatically. 

### Purposes

- To abstract permission checks out of component logic and into a reusable wrapper.
- To make permission-gated UI declarative and readable in JSX.
- To centralize the permission evaluation logic in a provider component.
- To provide a consistent API for hiding or showing UI elements based on permissions.
- To support fallback rendering when permission is denied (e.g., a disabled button or a message).
- To integrate with authorization libraries like CASL for complex ABAC rules.

### Syntax Rules and Structure

**General Syntax with a Custom `<Can>` Component:**
```jsx
// PermissionsProvider.jsx
import { createContext, useContext, useMemo } from 'react';

const PermissionsContext = createContext(null);

export function PermissionsProvider({ user, children }) {
  const value = useMemo(() => ({ user }), [user]);
  return (
    <PermissionsContext.Provider value={value}>
      {children}
    </PermissionsContext.Provider>
  );
}

export function usePermissions() {
  return useContext(PermissionsContext);
}

// Can.jsx
import { usePermissions } from './PermissionsProvider';

export function Can({ action, subject, fallback = null, children }) {
  const { user } = usePermissions();
  const allowed = checkPermission(user, action, subject);

  return allowed ? children : fallback;
}
```

**Component Breakdown:**
- `PermissionsProvider`: Supplies the user (and their permissions) to the component tree via context.
- `Can`: A wrapper component that receives `action`, `subject`, `fallback`, and `children`.
- `checkPermission(user, action, subject)`: The permission evaluation function.
- `children`: Rendered if permission is granted.
- `fallback`: Rendered if permission is denied (defaults to `null`).

**General Syntax with CASL's `<Can>` Component:**
```jsx
import { Can } from '@casl/react';
import { AbilityProvider } from './AbilityProvider';

function App() {
  return (
    <AbilityProvider>
      <Can I="create" a="Post">
        <button>Create Post</button>
      </Can>

      <Can I="delete" a="Post" fallback={<span>No delete rights</span>}>
        <button>Delete Post</button>
      </Can>
    </AbilityProvider>
  );
}
```

**Component Breakdown:**
- `AbilityProvider`: Wraps the app and provides the CASL ability instance.
- `<Can I="create" a="Post">`: Renders children if the user can `create` a `Post`.
- `fallback`: Rendered when the permission check fails.
- CASL's `<Can>` also supports a render-function child for more complex logic. 

**Syntax Rules:**
- Wrap the application root with the provider (`PermissionsProvider`, `RBACProvider`, `AbilityProvider`) before using `<Can>` components.
- Use the `<Can>` component for straightforward conditional rendering in JSX.
- Use the corresponding hook (`useCan`, `useAbility`) when the permission check is part of more complex component logic. 
- Provide a `fallback` when the denied state needs to display something (e.g., a disabled button or an explanatory message).
- Keep the permission evaluation logic in a single shared module to avoid duplication.
- For CASL, use the `subject` helper when passing resource instances: `subject('Post', post)`. 

**Constraints and Limitations:**
- Declarative components only control what the UI shows; they do not enforce security. The server must independently enforce permissions. 
- Overusing `<Can>` components can clutter JSX; use hooks for complex logic.
- The provider must be rendered above all `<Can>` components; otherwise, the context will be `null`.
- CASL's `<Can>` component requires the `@casl/react` package and an `Ability` instance.
- Permission state must be available at render time; asynchronous permission loading requires a loading state or Suspense.

### Annotated Code Examples

**Example 1: Custom `<Can>` Component with RBAC**

```jsx
// PermissionsContext.jsx
import React, { createContext, useContext, useMemo } from 'react';

const PermissionsContext = createContext(null);

export function PermissionsProvider({ user, children }) {
  const value = useMemo(() => ({ user }), [user]);
  return (
    <PermissionsContext.Provider value={value}>
      {children}
    </PermissionsContext.Provider>
  );
}

export function usePermissions() {
  return useContext(PermissionsContext);
}

// Can.jsx
import React from 'react';
import { usePermissions } from './PermissionsContext';

function checkPermission(user, action, subject) {
  if (!user?.permissions) return false;
  return user.permissions.includes(`${subject}:${action}`);
}

export function Can({ action, subject, fallback = null, children }) {
  const { user } = usePermissions();
  const allowed = checkPermission(user, action, subject);

  return allowed ? children : fallback;
}

// App.jsx
import React from 'react';
import { PermissionsProvider, Can } from './PermissionsContext';

const user = {
  name: 'Alice',
  permissions: ['post:read', 'post:create', 'comment:read'],
};

function PostActions() {
  return (
    <div>
      <Can action="read" subject="post">
        <button>View Post</button>
      </Can>

      <Can action="create" subject="post">
        <button>Create Post</button>
      </Can>

      <Can action="delete" subject="post" fallback={<span>No delete rights</span>}>
        <button>Delete Post</button>
      </Can>
    </div>
  );
}

export default function App() {
  return (
    <PermissionsProvider user={user}>
      <h1>Post Actions</h1>
      <PostActions />
    </PermissionsProvider>
  );
}
```

**Expected Output:** The component renders "View Post" and "Create Post" buttons, and a "No delete rights" message (because the user does not have `post:delete` permission).

**Why This Output Occurs:** The `Can` component evaluates `checkPermission(user, action, subject)` by checking whether the user's permission list includes the string `subject:action`. The user has `post:read` and `post:create`, so those buttons are rendered. The user lacks `post:delete`, so the fallback is rendered instead. 

**Example 2: CASL `<Can>` Component with ABAC**

```jsx
import React from 'react';
import { createContext, useContext } from 'react';
import { Can, AbilityProvider } from '@casl/react';
import { AbilityBuilder, Ability } from '@casl/ability';

// Define ability for a specific user
function defineAbilityFor(user) {
  const { can, cannot, build } = new AbilityBuilder(Ability);

  if (user.role === 'admin') {
    can('manage', 'all');
  } else if (user.role === 'editor') {
    can('read', 'Post');
    can('update', 'Post', { authorId: user.id });
    cannot('delete', 'Post');
  } else {
    can('read', 'Post', { published: true });
  }

  return build();
}

const AbilityContext = createContext(null);

function App() {
  const user = { id: 1, role: 'editor' };
  const ability = defineAbilityFor(user);

  return (
    <AbilityProvider value={ability}>
      <h1>Post Actions</h1>
      <Can I="read" a="Post">
        <button>Read Post</button>
      </Can>
      <Can I="update" a="Post">
        <button>Edit Post</button>
      </Can>
      <Can I="delete" a="Post" fallback={<span>Cannot delete</span>}>
        <button>Delete Post</button>
      </Can>
    </AbilityProvider>
  );
}

export default App;
```

**Expected Output:** The component renders "Read Post" and "Edit Post" buttons, and a "Cannot delete" message.

**Why This Output Occurs:** The CASL ability is built from rules: editors can `read` any Post, `update` their own Posts, and cannot `delete` any Post. The `<Can>` component checks each rule against the ability. Since the user is an editor, `read` and `update` are allowed, but `delete` is explicitly denied (the `cannot` rule overrides the default deny). 

### Real-World Cases

- **Admin dashboards:** Wrapping delete and edit buttons in `<Can>` components to hide them from users without permission.
- **SaaS applications:** Using `<Can>` to control access to billing, user management, and feature toggles.
- **Content platforms:** Showing "Edit" buttons only for post authors and "Delete" buttons only for admins.
- **E-commerce admin:** Gating order refund buttons based on the user's role and the order's status.
- **Multi-tenant applications:** Using `<Can>` with tenant-scoped permissions to ensure users only see actions relevant to their organisation.

---

## Core Concept 3: Protected Routes & Navigation

### Definitions

**Core Definition:** Protected Routes & Navigation is the practice of blocking unauthorized access to routes at the router level, ensuring that users cannot navigate to pages they do not have permission to view.

**Technical Definition:** Protected routes are implemented by wrapping route elements in a guard component (e.g., `ProtectedRoute`, `RequireRole`) that checks the user's authentication status and permissions before rendering the route's content. In React Router v6+, this is achieved through a higher-order component that conditionally renders either the protected content or a redirect to a login page or 403 page. In Next.js, route protection is enforced through middleware that runs on the edge before the request reaches the page, reading authentication cookies and redirecting unauthorized users. However, middleware alone is not sufficient: Next.js explicitly states that Proxy (formerly middleware) should not be the only line of defence, and authorization must be re-enforced at the handler or data source level. 

**Beginner-Friendly Explanation:** Protected routes are like locked doors in your app. If a user tries to visit the admin dashboard without permission, the route guard stops them and sends them to the login page or a "you don't have access" page. In React Router, you create a `ProtectedRoute` component that checks the user's status before showing the page. In Next.js, you can use middleware to check cookies before the page even loads. But remember: locking the door in the UI doesn't protect the data behind it—the server must also enforce permissions. 

### Purposes

- To prevent unauthorized users from accessing protected pages.
- To redirect unauthenticated users to the login page with the intended destination preserved.
- To redirect authenticated but unauthorized users to a 403 page or restricted UI.
- To centralize route protection logic in a reusable guard component.
- To support role-based route access (e.g., admin-only routes).
- To integrate with Next.js middleware for edge-level route protection.

### Syntax Rules and Structure

**General Syntax for React Router v6 ProtectedRoute:**
```jsx
import { Navigate } from 'react-router-dom';
import { useAuth } from './auth';

function ProtectedRoute({ children, requiredRole }) {
  const { user, isAuthenticated } = useAuth();

  if (!isAuthenticated) {
    return <Navigate to="/login" replace />;
  }

  if (requiredRole && !user.roles.includes(requiredRole)) {
    return <Navigate to="/403" replace />;
  }

  return children;
}

// Usage
<Routes>
  <Route path="/login" element={<LoginPage />} />
  <Route
    path="/admin"
    element={
      <ProtectedRoute requiredRole="admin">
        <AdminDashboard />
      </ProtectedRoute>
    }
  />
</Routes>
```

**Component Breakdown:**
- `ProtectedRoute`: A guard component that receives `children` and an optional `requiredRole`.
- `useAuth()`: Provides `user` and `isAuthenticated` from the auth context.
- `!isAuthenticated`: Redirects to `/login` if the user is not logged in.
- `requiredRole`: If the user's roles do not include the required role, redirects to `/403`.
- `replace`: Ensures the protected route is replaced in the browser history. 

**General Syntax for Next.js Middleware:**
```javascript
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  const token = request.cookies.get('session_id')?.value;
  const { pathname } = request.nextUrl;

  if (pathname.startsWith('/dashboard')) {
    if (!token) {
      const loginUrl = new URL('/login', request.url);
      loginUrl.searchParams.set('from', pathname);
      return NextResponse.redirect(loginUrl);
    }
  }

  if (pathname.startsWith('/admin')) {
    if (!token) {
      return NextResponse.redirect(new URL('/login', request.url));
    }
    // Role check would require decoding the token or calling an API
  }

  return NextResponse.next();
}

export const config = {
  matcher: ['/dashboard/:path*', '/admin/:path*'],
};
```

**Component Breakdown:**
- `request.cookies.get('session_id')`: Reads the session cookie.
- `request.nextUrl.pathname`: The requested path.
- `NextResponse.redirect()`: Redirects unauthorized users.
- `config.matcher`: Specifies which routes the middleware applies to. 
- Middleware runs on the edge before the request reaches the page.

**Syntax Rules:**
- Wrap the application with an auth provider that supplies `user` and `isAuthenticated`.
- Use `Navigate` with `replace` to prevent users from navigating back to protected routes after logout.
- Preserve the intended destination in the login redirect (e.g., `?from=/dashboard`).
- In Next.js, use middleware for optimistic redirects, but always re-enforce authorization at the data source. 
- Use `requiredRole` or `requiredPermission` props to extend the guard for role-based access. 
- For React Router, consider using loaders for data-level authorization in addition to component-level guards. 

**Constraints and Limitations:**
- Client-side route guards only control what the UI shows; they do not protect API endpoints or data.
- Middleware can be bypassed if the matcher configuration omits a route (e.g., CVE-2025-29927). 
- Middleware runs on the edge and may not have access to full session data (e.g., a database).
- Redirecting to a 403 page without preserving the intended destination can confuse users.
- Route guards add a layer of complexity; keep them focused on authentication and role checks.

### Annotated Code Examples

**Example 1: React Router v6 Protected Route with Role Check**

```jsx
import React from 'react';
import { BrowserRouter, Routes, Route, Navigate } from 'react-router-dom';
import { useAuth } from './AuthContext';

function ProtectedRoute({ children, requiredRole }) {
  const { user, isAuthenticated, isLoading } = useAuth();

  if (isLoading) return <p>Loading...</p>;

  if (!isAuthenticated) {
    return <Navigate to="/login" replace />;
  }

  if (requiredRole && !user.roles.includes(requiredRole)) {
    return <Navigate to="/403" replace />;
  }

  return children;
}

function AdminDashboard() {
  return <h1>Admin Dashboard</h1>;
}

function UserDashboard() {
  return <h1>User Dashboard</h1>;
}

function LoginPage() {
  return <h1>Login Page</h1>;
}

function ForbiddenPage() {
  return <h1>403 — Access Denied</h1>;
}

function App() {
  const { user, login } = useAuth();

  return (
    <BrowserRouter>
      <Routes>
        <Route path="/login" element={<LoginPage />} />
        <Route path="/403" element={<ForbiddenPage />} />
        <Route
          path="/admin"
          element={
            <ProtectedRoute requiredRole="admin">
              <AdminDashboard />
            </ProtectedRoute>
          }
        />
        <Route
          path="/dashboard"
          element={
            <ProtectedRoute>
              <UserDashboard />
            </ProtectedRoute>
          }
        />
      </Routes>
    </BrowserRouter>
  );
}

export default App;
```

**Expected Output:** An unauthenticated user navigating to `/admin` is redirected to `/login`. An authenticated user without the `admin` role navigating to `/admin` is redirected to `/403`. An authenticated user with the `admin` role sees "Admin Dashboard". An authenticated user without any required role navigating to `/dashboard` sees "User Dashboard".

**Why This Output Occurs:** The `ProtectedRoute` component checks authentication first, then role. If either check fails, it renders a `Navigate` component that redirects the user. The `replace` prop ensures the protected route is replaced in the history, so the user cannot press "Back" to return. 

**Example 2: Next.js Middleware with Role-Based Redirects**

```javascript
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';
import { jwtVerify } from 'jose';

const SECRET = new TextEncoder().encode(process.env.JWT_SECRET);

export async function middleware(request: NextRequest) {
  const token = request.cookies.get('access_token')?.value;
  const { pathname } = request.nextUrl;

  // Public routes
  if (pathname === '/login' || pathname === '/register') {
    if (token) {
      return NextResponse.redirect(new URL('/dashboard', request.url));
    }
    return NextResponse.next();
  }

  // Protected routes require authentication
  if (pathname.startsWith('/dashboard') || pathname.startsWith('/admin')) {
    if (!token) {
      const loginUrl = new URL('/login', request.url);
      loginUrl.searchParams.set('from', pathname);
      return NextResponse.redirect(loginUrl);
    }

    // Admin routes require admin role
    if (pathname.startsWith('/admin')) {
      try {
        const { payload } = await jwtVerify(token, SECRET);
        if (payload.role !== 'admin') {
          return NextResponse.redirect(new URL('/403', request.url));
        }
      } catch {
        return NextResponse.redirect(new URL('/login', request.url));
      }
    }
  }

  return NextResponse.next();
}

export const config = {
  matcher: ['/dashboard/:path*', '/admin/:path*', '/login', '/register'],
};
```

**Expected Output:** An unauthenticated user visiting `/admin` is redirected to `/login?from=/admin`. An authenticated non-admin user visiting `/admin` is redirected to `/403`. An admin user sees the admin page. A logged-in user visiting `/login` is redirected to `/dashboard`.

**Why This Output Occurs:** The middleware reads the JWT from the cookie, verifies it, and checks the `role` claim. For admin routes, it enforces the admin role. The middleware runs on the edge before the page renders, providing fast redirects. However, the page itself must also enforce authorization at the data source (e.g., in a Server Component or data access layer) to prevent bypass. 

### Real-World Cases

- **Admin panels:** Restricting `/admin/*` routes to users with the admin role.
- **SaaS dashboards:** Requiring authentication for `/dashboard` and role-based access for `/settings/billing`.
- **E-commerce:** Protecting `/checkout` and `/account` routes for authenticated users.
- **Multi-tenant applications:** Ensuring users can only access routes for their organisation.
- **Content platforms:** Allowing authors to access `/author/*` routes while restricting `/admin/*` to administrators.

---

## Core Concept 4: UI Condensation vs. Server Enforcement

### Definitions

**Core Definition:** UI Condensation vs. Server Enforcement is the architectural principle that distinguishes between hiding UI elements for user experience (UI condensation) and enforcing authorization at the server or data access layer for actual security (server enforcement).

**Technical Definition:** UI condensation refers to the practice of hiding or disabling UI controls (buttons, links, form fields) that the user does not have permission to use. This is purely a UX mechanism: it prevents users from attempting actions that will fail, reduces confusion, and creates a cleaner interface. It is not a security measure. Server enforcement refers to the authoritative authorization checks performed on the server (or in a server-only data access layer) that reject unauthorized requests with a 403 Forbidden response. The server must never trust the client to enforce permissions. The UI is an affordance; the server is the boundary. When the two drift apart, you get either a phishable workflow (the UI shows a button the server 403s) or a hidden capability (the server allows an action the UI never surfaces). 

**Beginner-Friendly Explanation:** Hiding a button in your app is like putting a "Do Not Enter" sign on a door. It's helpful because it stops people from trying to open a door they can't use, but it doesn't actually lock the door. If someone really wants to get in, they can just walk around the sign. The real lock—the thing that actually stops unauthorized access—is on the server. You need both: the sign (UI condensation) for a good user experience, and the lock (server enforcement) for security. 

### Purposes

- To clearly distinguish between UX affordances (hiding UI) and security boundaries (server checks).
- To prevent phishable workflows where the UI offers actions the server rejects.
- To prevent hidden capabilities where the server allows actions the UI never surfaces.
- To ensure that the server is the authoritative enforcement point for all permissions.
- To use the same policy definition on both client and server to prevent drift.
- To provide a consistent user experience while maintaining robust security.

### Syntax Rules and Structure

**General Syntax for Shared Policy Enforcement:**
```javascript
// policy.js — shared between client and server
const policy = {
  roles: [
    { name: 'admin', permissions: [{ resource: '*', action: '*' }] },
    {
      name: 'sales',
      permissions: [
        { resource: 'order', action: 'read' },
        { resource: 'order', action: 'refund', effect: 'deny' },
      ],
    },
    {
      name: 'customer',
      permissions: [{ resource: 'self', action: 'update' }],
    },
  ],
};

function checkPolicy(role, action, resource) {
  const roleDef = policy.roles.find((r) => r.name === role);
  if (!roleDef) return 'deny';

  for (const perm of roleDef.permissions) {
    if (perm.resource === resource || perm.resource === '*') {
      if (perm.action === action || perm.action === '*') {
        if (perm.effect === 'deny') return 'deny';
        return 'allow';
      }
    }
  }
  return 'deny';
}

export { checkPolicy };
```

**Component Breakdown:**
- `policy`: The single source of truth for roles and permissions.
- `checkPolicy(role, action, resource)`: A pure function that returns `'allow'` or `'deny'`.
- Explicit deny rules override allows.
- The same function is used on the client (for UI) and the server (for enforcement). 

**General Syntax for Server-Side Enforcement (Express):**
```javascript
import { checkPolicy } from './policy';

function rbacGate(action, resource) {
  return (req, res, next) => {
    if (!req.auth) {
      return res.status(401).json({ error: 'Unauthenticated' });
    }

    const decision = checkPolicy(req.auth.role, action, resource);
    if (decision !== 'allow') {
      return res.status(403).json({ error: 'Forbidden' });
    }

    next();
  };
}

// Usage
app.patch(
  '/api/customer/me',
  requireAuth,
  rbacGate('update', 'self'),
  updateCustomerHandler
);
```

**Component Breakdown:**
- `rbacGate(action, resource)`: A middleware factory that enforces the policy.
- `req.auth`: The authenticated user context (set by `requireAuth`).
- `checkPolicy(...)`: The shared policy evaluation function.
- `res.status(403)`: The server rejects unauthorized requests. 

**General Syntax for UI Condensation with the Same Policy:**
```jsx
import { checkPolicy } from './policy';

function Can({ action, resource, children, fallback = null }) {
  const { user } = useAuth();
  const decision = checkPolicy(user.role, action, resource);

  return decision === 'allow' ? children : fallback;
}

// Usage
<Can action="refund" resource="order" fallback={<span>No refund rights</span>}>
  <RefundButton />
</Can>
```

**Component Breakdown:**
- `Can`: A wrapper component that uses the same `checkPolicy` function.
- The UI hides the refund button for sales users because the policy explicitly denies `order:refund`.
- If the user forges the request, the server's `rbacGate` returns 403. 

**Syntax Rules:**
- Use the same policy definition (roles, resources, actions) on both client and server.
- Treat the client-side check as a UX affordance; never rely on it for security.
- Always enforce authorization on the server before executing any protected operation.
- Log denied attempts on the server for security auditing.
- Test authorization by calling API endpoints directly, without using the UI, to verify that server enforcement works. 
- In Next.js, use a server-only Data Access Layer (DAL) that performs authorization and returns minimal DTOs. 

**Constraints and Limitations:**
- Client-side policy code is visible and modifiable by users; it cannot be trusted.
- Server enforcement requires access to the user's permissions, which may involve a database lookup or token decoding.
- Keeping client and server policies in sync requires discipline; shared policy modules help but are not foolproof.
- Server enforcement adds latency; cache permission decisions where possible.
- Object-level authorization (e.g., "can this user edit this specific post?") requires resource attributes to be available on the server.

### Annotated Code Examples

**Example 1: Shared Policy with UI Hiding and Server Enforcement**

```javascript
// policy.js — shared module
const roles = {
  admin: { permissions: ['post:read', 'post:create', 'post:edit', 'post:delete'] },
  editor: { permissions: ['post:read', 'post:create', 'post:edit'] },
  viewer: { permissions: ['post:read'] },
};

function can(role, permission) {
  return roles[role]?.permissions.includes(permission) || false;
}

export { can };

// React component (UI condensation)
import React from 'react';
import { can } from './policy';
import { useAuth } from './AuthContext';

function PostActions() {
  const { user } = useAuth();

  return (
    <div>
      {can(user.role, 'post:read') && <button>View</button>}
      {can(user.role, 'post:create') && <button>Create</button>}
      {can(user.role, 'post:edit') && <button>Edit</button>}
      {can(user.role, 'post:delete') && <button>Delete</button>}
    </div>
  );
}

// Server-side enforcement (Express)
import { can } from './policy';

function requirePermission(permission) {
  return (req, res, next) => {
    if (!req.user || !can(req.user.role, permission)) {
      return res.status(403).json({ error: 'Forbidden' });
    }
    next();
  };
}

app.delete(
  '/api/posts/:id',
  requireAuth,
  requirePermission('post:delete'),
  deletePostHandler
);
```

**Expected Output:** A viewer sees only the "View" button. An editor sees "View", "Create", and "Edit" buttons. An admin sees all four buttons. If a viewer attempts to send a DELETE request to `/api/posts/:id` directly (bypassing the UI), the server returns 403 Forbidden.

**Why This Output Occurs:** The `can()` function is shared between client and server. The UI uses it to conditionally render buttons (affordance), and the server uses it to reject unauthorized requests (security). Even if a malicious user modifies the client code to show the delete button, the server still enforces the policy and returns 403. 

**Example 2: Next.js Server Component with Data Access Layer Enforcement**

```tsx
// lib/dal.ts — server-only Data Access Layer
import 'server-only';
import { cache } from 'react';
import { cookies } from 'next/headers';
import { verifySession } from './session';

export const getPost = cache(async (postId: string) => {
  const session = await verifySession();
  if (!session) {
    throw new Error('Unauthenticated');
  }

  const post = await db.post.findUnique({ where: { id: postId } });

  if (!post) {
    throw new Error('Post not found');
  }

  // Authorization: only the author or an admin can read the post
  if (post.authorId !== session.userId && session.role !== 'admin') {
    throw new Error('Forbidden');
  }

  // Return a minimal DTO, not the full database record
  return {
    id: post.id,
    title: post.title,
    content: post.content,
    authorId: post.authorId,
  };
});

// app/posts/[id]/page.tsx — Server Component
import { getPost } from '@/lib/dal';
import { notFound, forbidden } from 'next/navigation';

export default async function PostPage({ params }) {
  try {
    const post = await getPost(params.id);
    return (
      <article>
        <h1>{post.title}</h1>
        <p>{post.content}</p>
      </article>
    );
  } catch (error) {
    if (error.message === 'Forbidden') {
      forbidden(); // Renders a 403 page
    }
    notFound(); // Renders a 404 page
  }
}
```

**Expected Output:** An authenticated user who is the author of the post sees the post content. An admin sees the post content. A different authenticated user sees a 403 Forbidden page. An unauthenticated user is redirected to login.

**Why This Output Occurs:** The Data Access Layer (`getPost`) performs authorization before returning data. The Server Component calls `getPost` and handles the error by calling `forbidden()` or `notFound()`. Because the authorization check happens on the server, it cannot be bypassed by modifying client-side code. The `server-only` import ensures the DAL cannot be imported into client components. 

### Real-World Cases

- **E-commerce:** Hiding the "Refund" button from sales staff (UI) while the server rejects refund requests with 403 (enforcement).
- **SaaS admin panels:** Hiding billing settings from non-admin users while the server enforces admin-only access to billing APIs.
- **Healthcare:** Hiding patient records from unauthorized staff while the server enforces department-level access to patient data.
- **Multi-tenant applications:** Hiding other tenants' data in the UI while the server enforces tenant isolation at the database level.
- **Financial platforms:** Hiding large-transaction approval buttons while the server enforces role-based approval limits.

---

## References

- OWASP Next.js Security Cheat Sheet – OWASP: https://cheatsheetseries.owasp.org/cheatsheets/Nextjs_Security_Cheat_Sheet.html
- How to Build Scalable Access Control for Your Web App [Full Handbook] – FreeCodeCamp: https://www.freecodecamp.org/news/how-to-build-scalable-access-control-for-your-web-app/
- RBAC in React and Node: enforce the same policy on both sides – DEV Community: https://dev.to/kensaadi/rbac-in-react-and-node-enforce-the-same-policy-on-both-sides-2ial
- Broken Access Control in React: Fixes & Code Examples – DEV Community: https://dev.to/pentest_testing_corp/broken-access-control-in-react-fixes-code-examples-5dl6
- Attribute-Based Access Control (ABAC) in React: A Scalable Approach – Medium: https://medium.com/@dev_aman/attribute-based-access-control-abac-in-react-a-scalable-approach-df4990c7cbf0
- Control frontend features with CASL and Permit – Permit.io Documentation: https://docs.permit.io/integrations/feature-flagging/casl/
- CASL React Documentation – unpkg: https://app.unpkg.com/@casl/react
- @react-rbac/rbac – npm: https://www.npmjs.com/package/@react-rbac/rbac
- How to protect routes in React Router – CoreUI: https://coreui.io/answers/how-to-protect-routes-in-react-router/
- Middleware – Stack Auth Documentation: https://mintlify.wiki/stack-auth/stack-auth/sdk/nextjs/middleware
- React Applications and License Enforcement Boundaries – Key Manager Docs: https://docs.getkeymanager.com/
- Client-Side vs Server-Side Authorization: Why You Need Both – OpenReplay: https://blog.openreplay.com/client-side-vs-server-side-authorization/
- CASL – Official Documentation: https://casl.js.org/
- React Router v6 Authentication Guide – LogRocket: https://blog.logrocket.com/authentication-react-router-v6/
- Next.js Authentication Guide – Next.js Documentation: https://nextjs.org/docs/app/guides/authentication
- Next.js Data Security Guide – Next.js Documentation: https://nextjs.org/docs/app/guides/data-security