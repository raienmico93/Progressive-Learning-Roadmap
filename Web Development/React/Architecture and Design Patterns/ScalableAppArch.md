# Scalable & High-Performance Enterprise Architecture: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Scalable & High-Performance Enterprise Architecture is the set of structural principles, layering strategies, performance optimisation techniques, and resilience patterns that enable a React application to remain maintainable, fast, and reliable as it grows in codebase size, team size, and user scale.

**Technical Definition:** Enterprise-scale React architecture operates across five interdependent dimensions: **modular boundaries** that enforce dependency directions between feature modules and shared infrastructure; **shared infrastructure layers** that standardise design systems, HTTP clients, and cross-cutting utilities; **code-splitting and lazy loading** that reduce initial bundle size through route-based and component-level dynamic imports with prefetching; **selective memoisation** that applies `React.memo`, `useMemo`, and `useCallback` only where profiling proves benefit, avoiding the overhead of defensive memoisation; and **granular error boundaries** that contain failures at page, section, and component levels while reporting crashes to telemetry systems. The unifying principle is that architecture is about **managing dependency direction and runtime cost**: every import creates a coupling, and every render consumes a budget.

**Beginner-Friendly Explanation:** As your React app grows from a small project to an enterprise application, you need structure. Without it, every file depends on every other file, the bundle becomes enormous, the app gets slow, and one broken component takes down the whole page. Enterprise architecture solves this with clear rules: features are isolated and don't import from each other; shared code lives in a common layer; heavy code loads only when needed; performance optimisations are applied surgically, not everywhere; and errors are contained so a single widget crash doesn't blank the screen.

### Key Characteristics

- **Enforced Dependency Direction:** Feature modules may import from the shared layer but never from other features. The shared layer never imports from features. This is enforced by ESLint boundary rules and TypeScript path aliases.
- **Stable Public APIs:** Each feature exposes only what other parts of the app need through a barrel file (`index.ts`). Internal services, hooks, and utilities are private.
- **Performance by Measurement, Not Guesswork:** Memoisation is applied only after profiling reveals a bottleneck. Over-memoising with `useMemo`, `useCallback`, and `React.memo` adds overhead that can exceed the cost of re-rendering.
- **Surgical Code-Splitting:** Code is split at route boundaries first (highest impact), then at heavy component boundaries (charts, editors, modals). Prefetching on hover or focus eliminates the navigation delay caused by the network round-trip.
- **Granular Failure Containment:** A three-level error boundary architecture — page, section, component — ensures that a single widget crash does not blank the page.

### Prerequisites

- Solid understanding of React components, Hooks, and the render/commit lifecycle.
- Familiarity with module systems, dynamic imports, and bundler configuration (Vite, Webpack).
- Knowledge of React Context and state management for understanding shared layers.
- Experience with React Router or Next.js routing for route-based splitting.
- Awareness of React DevTools Profiler for performance measurement.

### Related Programming Areas

- **Feature-Sliced Design (FSD):** A methodology for feature-oriented frontend organisation with strict layering.
- **Clean Architecture:** Dependency inversion, layer separation, and testable business logic.
- **Performance Engineering:** Bundle analysis, rendering optimisation, and Core Web Vitals.
- **Resilience Engineering:** Error boundaries, graceful degradation, and crash reporting.
- **Monorepo Management:** Turborepo, Nx, and workspace-based package organisation.

### Core Concepts / Features

1. Feature Modules & Clean Code Boundaries
2. Shared Infrastructure Layers
3. Code-Splitting & Lazy Loading Patterns
4. Performance Optimization Patterns
5. Error Boundaries & Resilient UI

---

## Core Concept 1: Feature Modules & Clean Code Boundaries

### Definitions

**Core Definition:** Feature modules are self-contained directories that encapsulate all code related to a business domain — components, hooks, types, constants, and services — with strict rules preventing imports between features.

**Technical Definition:** Feature-based architecture (also called feature-first or feature-sliced) organises a React application by business capability rather than by technical type. Each feature lives in `src/features/<domain>/` and contains its own `components/`, `hooks/`, `types/`, `constants/`, and `index.ts` barrel file. The **dependency rule** is one-directional: `features` may import from `shared`, but `shared` must never import from `features`, and features must never import from each other. The barrel file (`index.ts`) is the feature's only public API — external code imports from `@features/resume`, never from `@features/resume/components/ResumeForm`. This boundary is enforced by ESLint's `boundaries` plugin or `import/no-restricted-paths`, which fails the build if a violation is introduced. 

**Beginner-Friendly Explanation:** Instead of putting all your buttons in one folder and all your hooks in another, you group everything related to a specific feature — like "resume" or "auth" — into its own folder. That folder is like a private apartment: everything inside it belongs together, and the only way to interact with it is through the front door (the `index.ts` file). Other features can't peek inside. This means you can change, test, or even delete a feature without worrying about breaking anything else.

### Purposes

- To enforce strict encapsulation of domain features, preventing unintended coupling between unrelated parts of the application.
- To establish a one-directional dependency flow: `features → shared`, never the reverse and never feature-to-feature.
- To provide a single, stable public API for each feature through a barrel file.
- To enable independent development, testing, and deployment of features by different teams.
- To make the codebase navigable: finding all code related to "checkout" means opening one folder.
- To support incremental migration: legacy code can be moved into feature folders one domain at a time.

### Syntax Rules and Structure

**Folder Structure:**

```
src/
├── app/                          # Next.js App Router (Server-centric)
│   ├── (routes)/
│   └── layout.tsx
├── features/
│   ├── resume/
│   │   ├── components/           # UI components for this feature
│   │   │   ├── ResumeForm.tsx
│   │   │   └── ResumePreview.tsx
│   │   ├── hooks/                # Feature-specific hooks
│   │   │   └── use-resume.ts
│   │   ├── types/                # Feature-specific types
│   │   │   └── resume.types.ts
│   │   ├── constants/            # Feature-specific constants
│   │   │   └── resume.constants.ts
│   │   └── index.ts              # Public API (barrel export)
│   ├── auth/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── types/
│   │   └── index.ts
│   └── billing/
│       └── ...
├── shared/                       # Cross-feature shared layer
│   ├── components/
│   │   └── ui/                   # Design system primitives (Button, Card)
│   ├── lib/
│   │   └── server/               # DB/API calls (server-side only)
│   ├── hooks/                    # Shared hooks (useMediaQuery, useDebounce)
│   ├── types/                    # Shared global types
│   └── constants/                # Shared constants
└── tsconfig.json                 # Path aliases: @features/*, @shared/*
```

**Component Breakdown:**
- `features/<domain>/`: Each feature is entirely self-contained. All its UI, hooks, types, and constants live within this folder.
- `features/<domain>/index.ts`: The barrel file that defines the feature's public API. Only what is exported here can be imported by other parts of the app.
- `shared/`: A cross-cutting layer containing code used by multiple features. It must never import from `features/`.
- `app/`: The Next.js App Router entry point. It orchestrates features and providers but contains no business logic.

**General Syntax for a Feature's Public API:**

```typescript
// features/resume/index.ts
// Only export what other features or the app layer need
export { ResumeForm } from './components/ResumeForm';
export { ResumePreview } from './components/ResumePreview';
export { useResume } from './hooks/use-resume';
export type { Resume, ResumeSection } from './types/resume.types';
// Do NOT export internal services, constants, or private hooks
```

**General Syntax for ESLint Boundary Enforcement:**

```javascript
// .eslintrc.js — enforcing dependency direction
module.exports = {
  plugins: ['boundaries'],
  settings: {
    'boundaries/elements': [
      { type: 'app', pattern: 'src/app/*' },
      { type: 'features', pattern: 'src/features/*' },
      { type: 'shared', pattern: 'src/shared/*' },
    ],
  },
  rules: {
    'boundaries/element-types': [
      'error',
      {
        default: 'disallow',
        rules: [
          { from: 'app', allow: ['features', 'shared'] },
          { from: 'features', allow: ['shared'] },
          { from: 'shared', allow: [] }, // shared cannot import from features or app
        ],
      },
    ],
  },
};
```

**Component Breakdown:**
- `boundaries/element-types`: Defines which element types may import from which others.
- `from: 'features', allow: ['shared']`: Features may import from shared but not from other features.
- `from: 'shared', allow: []`: Shared code cannot import from features or app.
- `default: 'disallow'`: Any import not explicitly allowed is an error.

**Syntax Rules:**
- Each feature must have a `index.ts` barrel file that defines its public API.
- External code imports from `@features/<domain>`, never from deep paths.
- Features must not import from other features, directly or transitively.
- The `shared/` layer must not import from `features/` or `app/`.
- Path aliases (`@features/*`, `@shared/*`) should be configured in `tsconfig.json` and the bundler.
- ESLint boundary rules should be enforced in CI to prevent violations from being merged.

**Constraints and Limitations:**
- Boundary enforcement requires tooling configuration (ESLint plugin, path aliases, CI checks).
- The barrel file adds a small maintenance overhead and can cause circular imports if not managed carefully.
- For very small applications, feature-based structure may be over-engineered.
- Cross-feature communication (e.g., auth affecting checkout) must go through the shared layer or a global store, requiring careful design.

### Annotated Code Examples

**Example 1: Feature Module with Public API**

```typescript
// features/auth/types/auth.types.ts
export type User = {
  id: string;
  name: string;
  email: string;
  role: 'admin' | 'user';
};

export type AuthState = {
  user: User | null;
  isAuthenticated: boolean;
};
```

```typescript
// features/auth/services/auth.service.ts (private — not exported)
import type { User } from '../types/auth.types';

export async function login(email: string, password: string): Promise<User> {
  const res = await fetch('/api/auth/login', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ email, password }),
  });
  if (!res.ok) throw new Error('Invalid credentials');
  return res.json();
}
```

```typescript
// features/auth/hooks/use-auth.ts
import { useState, useCallback } from 'react';
import { login as loginService } from '../services/auth.service';
import type { User } from '../types/auth.types';

export function useAuth() {
  const [user, setUser] = useState<User | null>(null);

  const login = useCallback(async (email: string, password: string) => {
    const userData = await loginService(email, password);
    setUser(userData);
  }, []);

  const logout = useCallback(() => setUser(null), []);

  return { user, login, logout, isAuthenticated: !!user };
}
```

```typescript
// features/auth/index.ts — PUBLIC API
export { LoginForm } from './components/LoginForm';
export { useAuth } from './hooks/use-auth';
export type { User, AuthState } from './types/auth.types';
// auth.service.ts is NOT exported — it is private to the feature
```

```tsx
// app/dashboard/page.tsx — imports from feature's public API
import { useAuth } from '@features/auth';
import { Button } from '@shared/components/ui';

export default function DashboardPage() {
  const { user, logout, isAuthenticated } = useAuth();

  if (!isAuthenticated) return <p>Please log in.</p>;

  return (
    <div>
      <h1>Welcome, {user.name}</h1>
      <Button onClick={logout}>Log Out</Button>
    </div>
  );
}
```

**Expected Output:** The `DashboardPage` imports `useAuth` from `@features/auth` (the public API) and `Button` from `@shared/components/ui`. It cannot import `auth.service.ts` directly because it is not exported from the barrel file. The ESLint boundary rule would flag any attempt to import from `@features/auth/services/auth.service`.

**Why This Output Occurs:** The barrel file (`index.ts`) acts as the feature's front door. Everything inside the feature folder is private by default; only what is explicitly exported is accessible from outside. This enforces encapsulation and allows the feature's internal structure to change without breaking consumers.

### Real-World Cases

- **Enterprise SaaS:** Features like `billing`, `team-management`, `analytics`, and `settings`, each owned by a different team and encapsulated in its own folder.
- **E-commerce:** Features like `cart`, `checkout`, `product-search`, and `user-reviews`, with strict boundaries preventing the cart from importing checkout internals.
- **Healthcare applications:** Features like `patient-records`, `appointments`, and `prescriptions`, with domain isolation for compliance and auditability.
- **Multi-tenant platforms:** Features per business capability, with tenant-specific customisation layered on top.

---

## Core Concept 2: Shared Infrastructure Layers

### Definitions

**Core Definition:** Shared infrastructure layers are the standardised, cross-cutting modules — design system components, HTTP clients, logging utilities, and shared domain primitives — that provide consistent foundations for all feature modules without creating coupling between features.

**Technical Definition:** The shared layer (also called the shared kernel) is a cross-context module that provides reusable infrastructure to all features. It is organised into four sub-layers: **shared/domain** (pure TypeScript business primitives with zero external dependencies — value objects, shared repository interfaces), **shared/application** (shared use cases, DTOs, and service interfaces), **shared/infrastructure** (HTTP clients, IndexedDB helpers, logging utilities that implement domain interfaces and may use external libraries), and **shared/ui** (design system components, reusable layouts, and generic UI hooks).  The critical architectural rule is that the shared layer must **never import from features or app** — it is a dependency of features, not a dependent on them. This ensures that shared infrastructure remains stable and reusable.

**Beginner-Friendly Explanation:** The shared layer is your app's toolbox. It contains all the common things every feature needs: buttons, inputs, API clients, date formatters, and shared types. But here's the key rule: the toolbox can't depend on any specific feature. If the "billing" feature needs something, it takes it from the toolbox. The toolbox doesn't know "billing" exists. This means you can change the toolbox without breaking features, and you can add new features without modifying the toolbox.

### Purposes

- To standardise UI components through a design system, ensuring visual consistency across all features.
- To provide a single HTTP client with consistent error handling, token refresh, and request configuration.
- To share domain primitives (value objects, types) across features without duplicating logic.
- To centralise logging, analytics, and telemetry infrastructure.
- To prevent features from reimplementing common utilities, reducing code duplication.
- To provide a stable foundation that features depend on, enabling independent feature development.

### Syntax Rules and Structure

**General Syntax for Shared Layer Structure:**

```
shared/
├── domain/                       # Pure business primitives (zero external deps)
│   ├── value-objects/
│   │   ├── Email.ts
│   │   ├── Money.ts
│   │   └── DateRange.ts
│   └── types/
│       └── global.types.ts
├── application/                  # Shared application logic
│   ├── use-cases/
│   └── dtos/
├── infrastructure/               # Shared infrastructure (may use external libs)
│   ├── fetchApi/
│   │   └── api-client.ts         # Generic HTTP client
│   ├── logging/
│   │   └── logger.ts
│   └── storage/
│       └── indexeddb-helper.ts
└── ui/                           # Design system components
    ├── Button/
    ├── Input/
    ├── Card/
    └── hooks/
        ├── use-media-query.ts
        └── use-debounce.ts
```

**Component Breakdown:**
- `shared/domain/`: Pure TypeScript. No React, no libraries, no side effects. Contains value objects (`Email`, `Money`) and shared repository interfaces.
- `shared/application/`: Shared use cases and DTOs used by multiple features.
- `shared/infrastructure/`: Implements interfaces from `shared/domain`. May use external libraries (Axios, IndexedDB wrappers).
- `shared/ui/`: Design system components and generic UI hooks. The only place where shared React components live.

**General Syntax for a Shared HTTP Client:**

```typescript
// shared/infrastructure/fetchApi/api-client.ts
import axios from 'axios';

export const apiClient = axios.create({
  baseURL: '/api',
  timeout: 10000,
  headers: { 'Content-Type': 'application/json' },
});

// Request interceptor: attach auth token
apiClient.interceptors.request.use((config) => {
  const token = localStorage.getItem('accessToken');
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});

// Response interceptor: handle 401 with token refresh
apiClient.interceptors.response.use(
  (response) => response,
  async (error) => {
    const originalRequest = error.config;
    if (error.response?.status === 401 && !originalRequest._retry) {
      originalRequest._retry = true;
      try {
        const { data } = await axios.post('/api/auth/refresh', {}, { withCredentials: true });
        localStorage.setItem('accessToken', data.accessToken);
        originalRequest.headers.Authorization = `Bearer ${data.accessToken}`;
        return apiClient(originalRequest);
      } catch (refreshError) {
        localStorage.clear();
        window.location.href = '/login';
        return Promise.reject(refreshError);
      }
    }
    return Promise.reject(error);
  }
);
```

**Component Breakdown:**
- `apiClient`: A configured Axios instance shared across all features.
- Request interceptor: Attaches the auth token to every request.
- Response interceptor: Handles 401 errors by attempting a token refresh before retrying.
- Features import `apiClient` from `@shared/infrastructure/fetchApi` rather than creating their own.

**General Syntax for a Shared Value Object:**

```typescript
// shared/domain/value-objects/Email.ts
export class Email {
  private constructor(public readonly value: string) {}

  static create(value: string): Email {
    if (!value.includes('@')) {
      throw new Error('Invalid email address');
    }
    return new Email(value.trim().toLowerCase());
  }

  equals(other: Email): boolean {
    return this.value === other.value;
  }

  toString(): string {
    return this.value;
  }
}
```

**Component Breakdown:**
- `Email`: A value object with validation logic. No React, no libraries — pure TypeScript.
- `create(value)`: Validates and normalises the email.
- Features import `Email` from `@shared/domain/value-objects` for consistent email handling.

**Syntax Rules:**
- `shared/domain/` must have zero external dependencies — only pure TypeScript.
- `shared/infrastructure/` implements interfaces from `shared/domain` and may use external libraries.
- `shared/ui/` contains design system components and generic UI hooks.
- The shared layer must never import from `features/` or `app/`.
- Features import from `@shared/*`, never from deep paths within shared modules.
- The shared layer should be versioned and treated as a stable API.

**Constraints and Limitations:**
- The shared layer can become a dumping ground for code that isn't truly shared; discipline is required.
- Changes to shared infrastructure affect all features simultaneously; this must be managed carefully.
- The shared layer adds a dependency that every feature relies on; it must be well-tested.
- Over-abstraction in the shared layer can make it harder to understand and maintain.

### Annotated Code Examples

**Example 1: Shared Design System Component**

```tsx
// shared/ui/Button/Button.tsx
import type { FC, ButtonHTMLAttributes } from 'react';

export type ButtonProps = ButtonHTMLAttributes<HTMLButtonElement> & {
  variant?: 'primary' | 'secondary' | 'danger';
  size?: 'sm' | 'md' | 'lg';
};

export const Button: FC<ButtonProps> = ({
  variant = 'primary',
  size = 'md',
  children,
  ...props
}) => (
  <button className={`btn btn--${variant} btn--${size}`} {...props}>
    {children}
  </button>
);
```

```tsx
// shared/ui/index.ts — public API for shared UI
export { Button } from './Button';
export { Input } from './Input';
export { Card } from './Card';
export { useMediaQuery } from './hooks/use-media-query';
export { useDebounce } from './hooks/use-debounce';
```

```tsx
// features/checkout/components/CheckoutButton.tsx
import { Button } from '@shared/ui';

export function CheckoutButton({ onCheckout }: { onCheckout: () => void }) {
  return <Button variant="primary" size="lg" onClick={onCheckout}>Place Order</Button>;
}
```

**Expected Output:** The `CheckoutButton` uses the shared `Button` component, ensuring visual consistency. If the design system changes the button's style, all features using the shared `Button` are updated automatically.

**Why This Output Occurs:** The shared UI layer provides a single source of truth for design system components. Features import from `@shared/ui` rather than creating their own buttons. This ensures consistency and reduces duplication across the application.

### Real-World Cases

- **Design systems:** Shared `Button`, `Input`, `Card`, `Modal` components used by all features, ensuring visual consistency.
- **API clients:** A single shared `apiClient` with token refresh, error normalisation, and request/response interceptors.
- **Logging:** A shared `logger` module that sends structured events to a telemetry service.
- **Feature flags:** A shared `useFeatureFlag` hook that reads from a remote config service.
- **Domain primitives:** Shared `Money`, `Email`, and `DateRange` value objects used across billing, auth, and reporting features.

---

## Core Concept 3: Code-Splitting & Lazy Loading Patterns

### Definitions

**Core Definition:** Code-splitting is the practice of breaking a JavaScript bundle into smaller chunks that are loaded on demand, and lazy loading is the technique of loading those chunks only when the component is actually rendered.

**Technical Definition:** Code-splitting divides a React application's bundle into multiple chunks at build time, so the initial page load downloads only the code needed for the current route. The primary mechanism in React is `React.lazy()`, which accepts a function returning a dynamic `import()` and returns a lazy component that suspends while the chunk loads. The lazy component must be wrapped in a `<Suspense>` boundary with a fallback. **Route-based splitting** splits code at route boundaries (`/dashboard`, `/settings`), which is the highest-impact strategy because it aligns chunks with user navigation. **Component-level splitting** lazy-loads heavy components (charts, rich text editors, modals) that are not needed on initial render. **Prefetching** loads chunks before the user navigates — typically on hover or focus of a link — eliminating the network round-trip delay.  Bundlers like Vite and Webpack support manual chunk configuration to group related modules into vendor and feature chunks.

**Beginner-Friendly Explanation:** When a user first opens your app, they don't need all the code. They only need the code for the page they're viewing. Code-splitting breaks your app into separate files ("chunks") so the browser only downloads what's needed for the current page. When the user navigates to a new page, the browser downloads that page's chunk. Lazy loading means the chunk is downloaded only when it's actually needed. Prefetching loads the chunk before the user clicks, so the page feels instant.

### Purposes

- To reduce the initial bundle size, improving Time to Interactive (TTI) and First Contentful Paint (FCP).
- To align code chunks with user navigation, so each route loads only its own dependencies.
- To defer loading of heavy components (charts, editors, modals) until they are visible or needed.
- To prefetch chunks on user intent (hover, focus) so navigation feels instant.
- To group vendor dependencies (React, router, query client) into stable chunks that cache well across deployments.
- To monitor bundle size and catch regressions before they reach production.

### Syntax Rules and Structure

**General Syntax for Route-Based Splitting (React Router 7.x):**

```tsx
import { lazy, Suspense } from 'react';
import { createBrowserRouter } from 'react-router';

// Lazy routes: each route is a separate chunk
const routes = [
  {
    path: '/',
    lazy: () => import('./pages/Home'),
  },
  {
    path: '/dashboard',
    lazy: () => import('./pages/Dashboard'),
    children: [
      { path: 'analytics', lazy: () => import('./pages/Analytics') },
      { path: 'settings', lazy: () => import('./pages/Settings') },
    ],
  },
];

const router = createBrowserRouter(routes);
```

**Component Breakdown:**
- `lazy: () => import('./pages/Dashboard')`: Dynamic import that creates a separate chunk.
- React Router handles the Suspense boundary internally in v7.
- Each route's code is loaded only when the user navigates to that route.

**General Syntax for Component-Level Lazy Loading:**

```tsx
import { lazy, Suspense } from 'react';

// Lazy-load a heavy component
const HeavyChart = lazy(() => import('./HeavyChart'));

function Dashboard() {
  return (
    <Suspense fallback={<div>Loading chart...</div>}>
      <HeavyChart data={data} />
    </Suspense>
  );
}
```

**Component Breakdown:**
- `lazy(() => import('./HeavyChart'))`: Creates a lazy component that loads the chart chunk on demand.
- `<Suspense fallback={...}>`: Shows a fallback while the chunk loads.
- The chart is only downloaded when the `Dashboard` component renders.

**General Syntax for Prefetching on Hover:**

```tsx
import { useQueryClient } from '@tanstack/react-query';
import { Link } from 'react-router';

function NavLink({ to, children }: { to: string; children: React.ReactNode }) {
  const queryClient = useQueryClient();

  const prefetch = () => {
    queryClient.prefetchQuery({
      queryKey: ['route', to],
      queryFn: () => fetchRouteData(to),
    });
  };

  return (
    <Link to={to} onMouseEnter={prefetch} onFocus={prefetch} preload="intent">
      {children}
    </Link>
  );
}
```

**Component Breakdown:**
- `onMouseEnter={prefetch}` and `onFocus={prefetch}`: Trigger prefetching when the user hovers or focuses the link.
- `preload="intent"`: A hint to the browser to prefetch the chunk on user intent.
- The chunk is loaded before the user clicks, eliminating the navigation delay.

**General Syntax for Vite Manual Chunks:**

```typescript
// vite.config.ts
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          'react-vendor': ['react', 'react-dom', 'react-router'],
          'query-vendor': ['@tanstack/react-query'],
          'dashboard': ['./src/pages/Dashboard', './src/pages/Analytics'],
          'settings': ['./src/pages/Settings', './src/pages/Profile'],
        },
      },
    },
  },
});
```

**Component Breakdown:**
- `react-vendor`: Groups React and router into a stable vendor chunk.
- `dashboard`: Groups dashboard-related pages into a feature chunk.
- Vendor chunks cache well across deployments because they rarely change.

**Syntax Rules:**
- Split at route boundaries first — this is the highest-impact strategy.
- Use `React.lazy()` with `<Suspense>` for component-level splitting.
- Prefetch chunks on hover or focus to eliminate navigation delay.
- Group vendor dependencies into stable chunks that cache across deployments.
- Avoid barrel-file imports in hot paths — they can pull in entire feature chunks unintentionally.
- Monitor bundle size with `vite-bundle-visualizer` or `webpack-bundle-analyzer`.

**Constraints and Limitations:**
- Code-splitting adds a network round-trip on navigation; prefetching mitigates this.
- Too many small chunks increase HTTP overhead; balance granularity with request count.
- `React.lazy()` requires a default export from the imported module.
- Suspense fallbacks must be designed carefully to avoid layout shift.
- Dynamic imports do not work with server-side rendering (SSR) without framework support.

### Annotated Code Examples

**Example 1: Route-Based Splitting with Prefetching**

```tsx
import { lazy, Suspense } from 'react';
import { createBrowserRouter, Link, Outlet } from 'react-router';
import { useQueryClient } from '@tanstack/react-query';

const Dashboard = lazy(() => import('./pages/Dashboard'));
const Settings = lazy(() => import('./pages/Settings'));
const Analytics = lazy(() => import('./pages/Analytics'));

function Layout() {
  const queryClient = useQueryClient();

  function prefetchRoute(path: string) {
    if (path === '/dashboard') import('./pages/Dashboard');
    if (path === '/settings') import('./pages/Settings');
    if (path === '/analytics') import('./pages/Analytics');
  }

  return (
    <div>
      <nav>
        <Link to="/dashboard" onMouseEnter={() => prefetchRoute('/dashboard')}>Dashboard</Link>
        <Link to="/settings" onMouseEnter={() => prefetchRoute('/settings')}>Settings</Link>
        <Link to="/analytics" onMouseEnter={() => prefetchRoute('/analytics')}>Analytics</Link>
      </nav>
      <Suspense fallback={<div>Loading page...</div>}>
        <Outlet />
      </Suspense>
    </div>
  );
}

const router = createBrowserRouter([
  {
    path: '/',
    element: <Layout />,
    children: [
      { path: 'dashboard', element: <Dashboard /> },
      { path: 'settings', element: <Settings /> },
      { path: 'analytics', element: <Analytics /> },
    ],
  },
]);
```

**Expected Output:** Each page is a separate chunk. When the user hovers over "Dashboard", the `Dashboard` chunk is prefetched. When they click, the page appears instantly because the chunk is already loaded.

**Why This Output Occurs:** The `lazy()` calls create separate chunks for each page. The `prefetchRoute` function triggers a dynamic import on hover, loading the chunk before the user clicks. The `<Suspense>` boundary shows a fallback while any chunk is loading.

### Real-World Cases

- **SaaS dashboards:** Splitting at route boundaries for dashboard, analytics, settings, and billing.
- **E-commerce:** Lazy-loading the product image gallery, reviews section, and recommendation carousel.
- **Content platforms:** Lazy-loading the rich text editor and media uploader.
- **Admin panels:** Splitting the user management, role management, and audit log pages.
- **Multi-step forms:** Lazy-loading each step's component as the user progresses.

---

## Core Concept 4: Performance Optimization Patterns

### Definitions

**Core Definition:** Performance optimisation patterns in React are the selective application of memoisation techniques — `React.memo`, `useMemo`, and `useCallback` — to prevent unnecessary re-renders and expensive computations, applied only where profiling proves benefit.

**Technical Definition:** React is fast by default. Over-memoising — wrapping every component in `React.memo` and every function in `useCallback` — adds overhead from dependency comparison and memory allocation that can exceed the cost of re-rendering.  The three memoisation tools have distinct purposes: `React.memo` wraps a **component** and skips re-rendering if its props are shallowly equal; `useMemo` caches a **computed value** inside a component and recomputes only when dependencies change; `useCallback` caches a **function reference** and returns the same function instance unless dependencies change.  The critical insight is that `useCallback` is only valuable when passing callbacks to memoised children — if the child is not wrapped in `React.memo`, the stable reference provides no benefit and adds overhead.  The decision framework is: profile first, identify the bottleneck, apply the minimum memoisation required, and verify the improvement.

**Beginner-Friendly Explanation:** React re-renders components when their state or props change. Usually this is fast enough. But sometimes a component re-renders unnecessarily — for example, when a parent re-renders and passes new function references to a child that doesn't actually need to update. Memoisation fixes this, but it costs something: React has to compare the old and new props, and storing the cached values uses memory. So the rule is: don't memoise everything. Profile your app, find the actual slow spots, and memoise only those. Over-memoising makes your app slower, not faster.

### Purposes

- To prevent unnecessary re-renders of expensive or frequently rendered components.
- To cache expensive computations (filtering, sorting, aggregations) so they run only when inputs change.
- To stabilise function references passed to memoised children.
- To stabilise object and array references passed as props.
- To apply memoisation surgically based on profiling data, not defensively.
- To avoid the overhead of comparison and memory allocation that comes with over-memoisation.

### Syntax Rules and Structure

**The Decision Framework:**

```
Is the component re-rendering unnecessarily?     → Profile first
Is the re-render expensive?                       → React.memo
Is the computation expensive (O(n) or higher)?    → useMemo
Is a callback passed to a memoised child?         → useCallback
None of the above?                                → Do nothing
```

**General Syntax for `React.memo`:**

```tsx
import { memo } from 'react';

const ExpensiveList = memo(function ExpensiveList({ items }) {
  return (
    <ul>
      {items.map((item) => <li key={item.id}>{item.name}</li>)}
    </ul>
  );
});
```

**Component Breakdown:**
- `memo(...)`: Wraps the component so it skips re-rendering if props are shallowly equal.
- Only useful for components that render the same output for the same props and re-render frequently.
- Do not wrap every component; overuse adds overhead.

**General Syntax for `useMemo`:**

```tsx
import { useMemo } from 'react';

function FilteredList({ items, query }) {
  // ✅ Expensive computation: filter 10,000 items
  const filteredItems = useMemo(
    () => items.filter((item) => item.name.includes(query)),
    [items, query]
  );

  return <ul>{filteredItems.map((item) => <li key={item.id}>{item.name}</li>)}</ul>;
}
```

**Component Breakdown:**
- `useMemo(() => ..., [items, query])`: Caches the filtered result and recomputes only when `items` or `query` changes.
- Use only for expensive computations — not for simple object literals or arrays.
- For cheap computations, compute inline without `useMemo`.

**General Syntax for `useCallback`:**

```tsx
import { useCallback, memo } from 'react';

const MemoizedChild = memo(function Child({ onClick }) {
  return <button onClick={onClick}>Click</button>;
});

function Parent({ items }) {
  // ✅ useCallback is valuable here because Child is memoised
  const handleClick = useCallback(() => {
    console.log('clicked');
  }, []);

  return <MemoizedChild onClick={handleClick} />;
}
```

**Component Breakdown:**
- `useCallback(() => ..., [])`: Returns the same function reference unless dependencies change.
- Only valuable when the consumer is memoised (`React.memo`).
- Without `React.memo` on the child, `useCallback` provides no benefit.

**Syntax Rules:**
- Profile first with React DevTools Profiler; never optimise based on guesswork.
- Use `React.memo` only for components that re-render frequently with the same props.
- Use `useMemo` only for computations with significant CPU cost (filtering, sorting, aggregations).
- Use `useCallback` only when passing callbacks to memoised children.
- Do not memoise trivial computations — the comparison overhead exceeds the savings.
- Use `why-did-you-render` in development to detect unnecessary re-renders in memoised components.
- React Compiler (when available) automatically applies memoisation equivalent to `useMemo`, `useCallback`, and `React.memo` at build time.

**Constraints and Limitations:**
- Over-memoising can slow the app down due to comparison and memory overhead.
- `React.memo` performs a shallow comparison; deeply nested objects always cause re-renders.
- `useMemo` is not a guarantee — React may discard cached values under memory pressure.
- `useCallback` with an inline arrow function recreates the function every render if dependencies change.
- Memoisation does not help if the component's props change on every render anyway.

### Annotated Code Examples

**Example 1: When to Use `useMemo` (Expensive Computation)**

```tsx
// ✅ CORRECT: Expensive filter operation
function ProductList({ products, query }) {
  const filteredProducts = useMemo(
    () => products.filter((p) => p.name.toLowerCase().includes(query.toLowerCase())),
    [products, query]
  );

  return <ul>{filteredProducts.map((p) => <li key={p.id}>{p.name}</li>)}</ul>;
}

// ❌ WRONG: Trivial computation wrapped in useMemo
function BadExample({ items }) {
  const count = useMemo(() => items.length, [items]); // Overhead exceeds benefit

  return <p>{count} items</p>;
}
```

**Expected Output:** The first component avoids re-filtering a large list on every render. The second component adds unnecessary overhead for a trivial `.length` access.

**Why This Output Occurs:** Filtering a large array is O(n) and runs on every render without `useMemo`. Accessing `.length` is O(1) and does not benefit from caching. The rule is: memoise only when the computation is expensive.

**Example 2: When to Use `useCallback` (Memoised Child)**

```tsx
const UserRow = memo(function UserRow({ user, onSelect }) {
  console.log('UserRow rendered:', user.name);
  return <div onClick={() => onSelect(user.id)}>{user.name}</div>;
});

function UserList({ users }) {
  // ✅ useCallback is valuable because UserRow is memoised
  const handleSelect = useCallback((id) => {
    console.log('Selected:', id);
  }, []);

  return users.map((user) => (
    <UserRow key={user.id} user={user} onSelect={handleSelect} />
  ));
}
```

**Expected Output:** `UserRow` re-renders only when its `user` prop changes, not when `handleSelect` is recreated (because `useCallback` keeps it stable).

**Why This Output Occurs:** `UserRow` is wrapped in `React.memo`, so it skips re-rendering if props are shallowly equal. `useCallback` ensures `handleSelect` has a stable reference. Without `useCallback`, a new function would be created on every render, breaking the memoisation and causing every `UserRow` to re-render.

### Real-World Cases

- **Data tables:** Memoising row components and expensive sort/filter computations.
- **Dashboard charts:** Memoising chart data transformations and callback handlers.
- **Large lists:** Wrapping list items in `React.memo` and stabilising callbacks with `useCallback`.
- **Search interfaces:** Memoising filtered results with `useMemo`.
- **Form libraries:** Memoising validation functions and change handlers passed to fields.

---

## Core Concept 5: Error Boundaries & Resilient UI

### Definitions

**Core Definition:** Error boundaries are React components that catch JavaScript errors in their child component tree, render a fallback UI instead of a blank screen, and report the error to a telemetry service, with granular placement at page, section, and component levels.

**Technical Definition:** A React Error Boundary is a class component implementing `static getDerivedStateFromError()` (to update state and render the fallback) and/or `componentDidCatch()` (to log error information). Error boundaries catch errors during rendering, in lifecycle methods, and in constructors of the entire tree below them. They do **not** catch errors in event handlers, asynchronous code (`setTimeout`, `requestAnimationFrame`), or server-side rendering.  A **three-level architecture** is recommended for enterprise applications: **page-level** boundaries catch catastrophic failures and render a full-page fallback; **section-level** boundaries isolate feature areas (nutrition, ingredients, compare columns) and render an inline fallback; and **component-level** boundaries protect individual widgets (badges, charts, cards) and render a minimal placeholder or nothing.  The fallback UI should always offer a recovery path: "Try Again" (re-render), "Go Home" (navigate away), or "Report Issue" (open a feedback form). All caught errors must be logged with component name, error message, component stack, and product context.

**Beginner-Friendly Explanation:** When a React component crashes, React unmounts the entire component tree by default, leaving a blank white screen. That's terrible for users. An error boundary is a safety net: you wrap it around a part of your UI, and if anything inside crashes, the boundary catches the error and shows a friendly message instead of a blank screen. In a big app, you use three levels of safety nets: one for the whole page, one for each section, and one for each small widget. This way, a broken chart doesn't take down the entire dashboard.

### Purposes

- To prevent a single component crash from unmounting the entire application and showing a blank screen.
- To contain failures at the appropriate level: page, section, or component.
- To provide users with a clear recovery path: retry, navigate away, or report the issue.
- To log errors with full context (component stack, user session, product data) for debugging.
- To preserve user input and state outside the failed boundary.
- To integrate with telemetry services (Sentry, CrashGuard, Grafana Faro) for production crash reporting.

### Syntax Rules and Structure

**General Syntax for a Class-Based Error Boundary:**

```tsx
import React, { Component, ErrorInfo, ReactNode } from 'react';

interface Props {
  children: ReactNode;
  fallback: ReactNode;
  onError?: (error: Error, errorInfo: ErrorInfo) => void;
}

interface State {
  hasError: boolean;
  error: Error | null;
}

export class ErrorBoundary extends Component<Props, State> {
  state: State = { hasError: false, error: null };

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    // Report to telemetry service
    this.props.onError?.(error, errorInfo);
  }

  reset = () => this.setState({ hasError: false, error: null });

  render() {
    if (this.state.hasError) {
      return this.props.fallback;
    }
    return this.props.children;
  }
}
```

**Component Breakdown:**
- `getDerivedStateFromError`: Static method called during render when a child throws. Returns state to trigger the fallback.
- `componentDidCatch`: Lifecycle method called during commit. Used for logging and telemetry.
- `reset`: Clears the error state, allowing children to re-render.
- `fallback`: The UI rendered when an error is caught.

**General Syntax for the Three-Level Architecture:**

```tsx
function App() {
  return (
    <PageErrorBoundary fallback={<FullPageError />}>
      <Header />
      <SectionErrorBoundary fallback={<SectionError name="Dashboard" />}>
        <DashboardWidgets />
      </SectionErrorBoundary>
      <Footer />
    </PageErrorBoundary>
  );
}

function DashboardWidgets() {
  return (
    <div>
      <ComponentErrorBoundary fallback={<WidgetError name="Chart" />}>
        <RevenueChart />
      </ComponentErrorBoundary>
      <ComponentErrorBoundary fallback={<WidgetError name="Stats" />}>
        <StatsCards />
      </ComponentErrorBoundary>
    </div>
  );
}
```

**Component Breakdown:**
- `PageErrorBoundary`: Outermost boundary. Catches anything that escapes lower boundaries. Renders a full-page fallback.
- `SectionErrorBoundary`: Isolates feature areas. Renders an inline fallback for that section.
- `ComponentErrorBoundary`: Protects individual widgets. Renders a minimal placeholder.
- Each boundary can have its own `onError` callback for telemetry.

**General Syntax with `react-error-boundary` (Functional Alternative):**

```tsx
import { ErrorBoundary } from 'react-error-boundary';

function ErrorFallback({ error, resetErrorBoundary }) {
  return (
    <div role="alert">
      <h2>Something went wrong</h2>
      <pre>{error.message}</pre>
      <button onClick={resetErrorBoundary}>Try again</button>
    </div>
  );
}

function App() {
  return (
    <ErrorBoundary
      FallbackComponent={ErrorFallback}
      onError={(error, info) => reportToSentry(error, info)}
      onReset={() => { /* reset state */ }}
    >
      <MyComponent />
    </ErrorBoundary>
  );
}
```

**Component Breakdown:**
- `ErrorBoundary`: A functional wrapper from the `react-error-boundary` library.
- `FallbackComponent`: Receives `error` and `resetErrorBoundary` props.
- `onError`: Callback for telemetry reporting.
- `onReset`: Callback for resetting parent state when the boundary resets.

**Syntax Rules:**
- Place boundaries at data-fetch granularity: one per independent section or widget.
- Use the three-level architecture: page, section, component.
- Always provide a recovery path in the fallback: "Try Again", "Go Home", or "Report Issue".
- Report all caught errors to a telemetry service with component stack and context.
- Do not use a single global boundary; it is too coarse and causes full-app reloads for minor crashes.
- Error boundaries do not catch errors in event handlers — use `try`/`catch` or `useErrorHandler` for those.
- Wrap the application root with a global boundary as a last resort.

**Constraints and Limitations:**
- Error boundaries must be class components (or use `react-error-boundary`).
- They do not catch errors in event handlers, asynchronous code, or server-side rendering.
- A fallback that itself throws will propagate to the next boundary above it.
- Error boundaries add a small amount of overhead; use them strategically, not everywhere.
- Telemetry integration requires configuration (Sentry DSN, CrashGuard endpoint).

### Annotated Code Examples

**Example 1: Three-Level Error Boundary Architecture**

```tsx
// ErrorBoundary.tsx — reusable class-based boundary
import React, { Component, ErrorInfo, ReactNode } from 'react';

interface Props {
  children: ReactNode;
  fallback: ReactNode;
  onError?: (error: Error, errorInfo: ErrorInfo) => void;
}

interface State {
  hasError: boolean;
}

export class ErrorBoundary extends Component<Props, State> {
  state: State = { hasError: false };

  static getDerivedStateFromError(): State {
    return { hasError: true };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    console.error('Error caught:', error, errorInfo);
    this.props.onError?.(error, errorInfo);
  }

  reset = () => this.setState({ hasError: false });

  render() {
    if (this.state.hasError) {
      return (
        <div role="alert" style={{ padding: '16px', border: '1px solid red' }}>
          {this.props.fallback}
          <button onClick={this.reset}>Try Again</button>
        </div>
      );
    }
    return this.props.children;
  }
}
```

```tsx
// App.tsx — three-level architecture
import { ErrorBoundary } from './ErrorBoundary';

function BrokenWidget() {
  throw new Error('Widget data unavailable');
}

function WorkingWidget() {
  return <p>Widget loaded successfully</p>;
}

function Dashboard() {
  return (
    <div>
      <h2>Dashboard</h2>
      <ErrorBoundary fallback={<p>Chart failed to load</p>}>
        <BrokenWidget />
      </ErrorBoundary>
      <ErrorBoundary fallback={<p>Stats failed to load</p>}>
        <WorkingWidget />
      </ErrorBoundary>
    </div>
  );
}

export default function App() {
  return (
    <ErrorBoundary fallback={<h1>Application Error</h1>}>
      <header>App Header</header>
      <ErrorBoundary fallback={<h2>Page Error</h2>}>
        <Dashboard />
      </ErrorBoundary>
      <footer>App Footer</footer>
    </ErrorBoundary>
  );
}
```

**Expected Output:** The header and footer render normally. The dashboard shows "Chart failed to load" with a "Try Again" button for the broken widget, and "Widget loaded successfully" for the working widget. The page-level and app-level boundaries are not triggered.

**Why This Output Occurs:** The component-level boundary around `BrokenWidget` catches its error first and renders the fallback. The other component boundary and the page/app boundaries are unaffected. This demonstrates granular containment: a single widget crash does not take down the entire dashboard.

### Real-World Cases

- **Enterprise dashboards:** Component-level boundaries around each widget (charts, stats, feeds) so one failed widget doesn't blank the dashboard.
- **E-commerce:** Section-level boundaries around recommendations, reviews, and product details.
- **SaaS applications:** Page-level boundaries for route-level failures with a "Go Home" recovery path.
- **Financial applications:** Error boundaries with telemetry integration (Sentry, CrashGuard) for production crash reporting.
- **Healthcare applications:** Granular boundaries with detailed error logging for compliance and auditability.

---

## References

- 機能別アーキテクチャ導入（Next.js / React / TypeScript）· Issue #16 · RTP-RagToPent/Tier-Map-Frontend – GitHub: https://github.com/RTP-RagToPent/Tier-Map-Frontend/issues/16
- lab-clean-architecture-react/docs/architecture/folder-structure.md – GitHub: https://github.com/pplancq/lab-clean-architecture-react/blob/e6866c45ba88ce7a831c540475b3ddbad7512c54/docs/architecture/folder-structure.md
- Feature-Sliced Design – Layered Architecture: https://feature-sliced.design/docs/reference/layers
- Route-Based Code Splitting – OrchestKit: https://github.com/yonatangross/orchestkit/blob/8cca80c80f8568ca1008d0f4ebb8aedceaa2588c/plugins/ork/skills/performance/references/route-splitting.md
- Performance Optimization for React-based projects – rtCamp: https://rtcamp.com/handbook/react-best-practices/performance-optimization/
- React Best Practices Guide 2025 – GitHub: https://raw.githubusercontent.com/whereq/whereq.github.io-docusaurus/refs/heads/main/blog/React-Best-Practices-Guide-2025.md
- Error Boundary & Fallback UI Standard – GitHub: https://github.com/ericsocrat/tryvit/issues/68
- crashguard-react – npm: https://www.npmjs.com/package/crashguard-react
- react-error-boundary – GitHub: https://github.com/bvaughn/react-error-boundary
- React Documentation – Error Boundaries: https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary
- React Documentation – `React.lazy`: https://react.dev/reference/react/lazy
- React Documentation – `useMemo`: https://react.dev/reference/react/useMemo
- React Documentation – `useCallback`: https://react.dev/reference/react/useCallback
- React Documentation – `memo`: https://react.dev/reference/react/memo
- Vite – Build Options (Manual Chunks): https://vite.dev/config/build-options
- React Router 7.x – Lazy Loading: https://reactrouter.com/start/framework/route-module
- TanStack Query – Prefetching: https://tanstack.com/query/latest/docs/framework/react/guides/prefetching
- ESLint Plugin Boundaries: https://github.com/javierbrea/eslint-plugin-boundaries
- Sentry for React: https://docs.sentry.io/platforms/javascript/guides/react/