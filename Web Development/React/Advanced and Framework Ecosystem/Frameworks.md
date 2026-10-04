# Modern React Frameworks: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Modern React frameworks are full-stack meta-frameworks—principally Next.js and React Router v7 (formerly Remix)—that provide file-system routing, server rendering, data loading, and build tooling out of the box, eliminating the need to assemble a custom toolchain from scratch.

**Technical Definition:** Modern React meta-frameworks are opinionated, full-stack frameworks that integrate routing, data fetching, server rendering, code-splitting, and build optimisation into a unified developer experience. Next.js (maintained by Vercel) uses the App Router paradigm with React Server Components as the default, file-system routing with dynamic segments, parallel routes, and intercepting routes, and Turbopack as its Rust-based bundler. React Router v7 (the successor to Remix v2) introduces a unified architecture with three modes—Declarative, Data, and Framework—where Framework Mode wraps Data Mode with a Vite plugin to add type-safe routing, SSR, and file-system route conventions. Both frameworks target multiple deployment primitives: Node.js servers, serverless functions, and edge runtimes (V8 isolates). React core's official guidance now recommends using a framework for any application that needs routing, as frameworks tightly integrate routing with data fetching, code-splitting, and rendering strategies.

**Beginner-Friendly Explanation:** A few years ago, if you wanted to build a React app, you started with Create React App and then had to add routing, data fetching, server rendering, and build tooling yourself. Modern React frameworks like Next.js and React Router v7 give you all of that out of the box. They handle the boring infrastructure so you can focus on building features. Next.js is the most popular choice, with its own bundler (Turbopack) and file-system routing. React Router v7 is the evolution of Remix—it's more flexible and lets you choose how much framework you want.

### Key Characteristics

- **Framework-First Development:** React core now recommends starting with a framework rather than assembling a custom stack. Create React App (CRA) was deprecated in early 2025, signalling the shift toward full-stack frameworks. 
- **File-System Routing as the Baseline:** Both Next.js and React Router v7 use file-system conventions for defining routes, with support for dynamic segments, nested layouts, and route-level code splitting. 
- **Server Components and Streaming:** Next.js App Router uses React Server Components by default, enabling direct database access from components and streaming HTML to the browser. React Router v7 has preview support for RSC in Framework Mode. 
- **Unified Build Ecosystems:** Next.js uses Turbopack (Rust-based, up to 10x faster than Webpack for large apps), while React Router v7 uses Vite (esbuild + Rollup, with Rolldown coming). Vite is the default for most non-Next.js frameworks. 
- **Deployment Flexibility:** Frameworks target multiple runtimes: Node.js servers (full API access), serverless functions (scalable, ~250ms cold boot), and edge runtimes (V8 isolates, instant cold boot, limited API). 

### Prerequisites

- Solid understanding of React components, Hooks, and JSX.
- Familiarity with HTTP fundamentals (requests, responses, status codes).
- Basic understanding of server-side rendering and client-side hydration.
- Experience with JavaScript/TypeScript module systems (import/export).
- Awareness of Node.js and browser runtime differences.

### Related Programming Areas

- **Server-Side Rendering (SSR):** Rendering React on the server for performance and SEO.
- **React Server Components (RSC):** Components that run only on the server.
- **File-System Routing:** Defining routes through folder and file conventions.
- **Build Tooling:** Bundlers, transpilers, and code-splitting.
- **Edge Computing:** Running code at the network edge for low latency.
- **Deployment Platforms:** Vercel, Netlify, Cloudflare Workers, AWS Lambda.

### Core Concepts / Features

1. Next.js & React Router v7: Unified Architectures
2. Meta-Framework Router Architectures
3. Unified Build Ecosystems
4. Deployment Primitives

---

## Core Concept 1: Next.js & React Router v7: Unified Architectures

### Definitions

**Core Definition:** Next.js and React Router v7 are the two dominant modern React meta-frameworks, each offering a unified architecture that integrates routing, data loading, server rendering, and build tooling into a single developer experience.

**Technical Definition:** Next.js (Vercel) is a full-stack React framework that uses the App Router paradigm, where React Server Components are the default, file-system routing defines routes through folders, and Turbopack handles bundling. React Router v7 (the successor to Remix v2) introduces a unified architecture with three modes: **Declarative Mode** (basic client-side routing with `BrowserRouter`), **Data Mode** (adds `loader` and `action` APIs with `createBrowserRouter`), and **Framework Mode** (wraps Data Mode with a Vite plugin to add type-safe routing, SSR, file-system routes, and intelligent code splitting). React Router v7 Framework Mode is the direct successor to Remix v2, and the two frameworks have converged in capabilities: both support server rendering, data loading with loaders/actions, nested layouts, and deployment to multiple runtimes.

**Beginner-Friendly Explanation:** Next.js and React Router v7 are the two main ways to build a full-stack React app today. Next.js is the more opinionated, "batteries-included" option—it has its own bundler (Turbopack), its own routing system (App Router), and Server Components by default. React Router v7 is more flexible—it lets you choose how much framework you want, from simple client-side routing all the way to full-stack SSR with file-system routes. If you're coming from Remix, React Router v7 Framework Mode is where you'll feel at home.

### Purposes

- To provide a complete, integrated developer experience for building full-stack React applications.
- To eliminate the need to assemble routing, data fetching, and build tooling from scratch.
- To enable server rendering and React Server Components with minimal configuration.
- To support multiple rendering strategies: SPA, SSR, static, and streaming.
- To offer a migration path from legacy frameworks (CRA, Remix v2) to modern architectures.
- To unify routing and data loading through framework-level APIs (`loader`, `action`).

### Syntax Rules and Structure

**Next.js App Router — File-System Routing:**

```
app/
├── layout.tsx              # Root layout (required)
├── page.tsx                # Home page (/)
├── dashboard/
│   ├── layout.tsx          # Dashboard layout
│   ├── page.tsx            # /dashboard
│   └── settings/
│       └── page.tsx        # /dashboard/settings
├── blog/
│   └── [slug]/
│       └── page.tsx        # /blog/:slug (dynamic segment)
└── @modal/                 # Parallel route slot
    └── login/
        └── page.tsx
```

**Component Breakdown:**
- `layout.tsx`: Shared UI that wraps child pages. Layouts preserve state and do not re-render on navigation.
- `page.tsx`: Unique UI for a route. Required to make a segment publicly accessible.
- `[slug]`: Dynamic segment that captures a value from the URL.
- `@modal`: Named slot for parallel routes.

**React Router v7 — Framework Mode Route Configuration:**

```typescript
// app/routes.ts
import { index, route } from "@react-router/dev/routes";

export default [
  index("./home.tsx"),
  route("about", "./about.tsx"),
  route("products/:id", "./product.tsx"),
  route("dashboard", "./dashboard.tsx", [
    route("settings", "./settings.tsx"),
  ]),
];
```

**Component Breakdown:**
- `index("./home.tsx")`: The index route (matches `/`).
- `route("products/:id", "./product.tsx")`: A route with a dynamic segment (`:id`).
- Nested routes are defined as an array in the third argument.
- Framework Mode uses a Vite plugin (`@react-router/dev`) to enable file-system routes, SSR, and type generation.

**Next.js Data Fetching with Server Components:**

```tsx
// app/users/[id]/page.tsx — Server Component
import { db } from '@/lib/db';

export default async function UserPage({ params }: { params: { id: string } }) {
  const user = await db.user.findUnique({ where: { id: params.id } });
  return <h1>{user.name}</h1>;
}
```

**Component Breakdown:**
- The component is `async` and runs only on the server.
- It accesses the database directly — no API layer needed.
- `params` contains the dynamic segment value (`id`).

**React Router v7 Loader and Action:**

```typescript
// app/routes/product.tsx
import type { Route } from "./+types/product";

// Loader: runs before render, fetches data
export async function loader({ params }: Route.LoaderArgs) {
  const product = await fetchProduct(params.id);
  return { product };
}

// Action: handles form submissions and mutations
export async function action({ request }: Route.ActionArgs) {
  const formData = await request.formData();
  await updateProduct(formData);
  return { success: true };
}

// Component receives loaderData
export default function Product({ loaderData }: Route.ComponentProps) {
  return <h1>{loaderData.product.name}</h1>;
}
```

**Component Breakdown:**
- `loader`: Runs on the server before rendering. Returns data that is passed to the component as `loaderData`.
- `action`: Handles form submissions and mutations. Automatically revalidates loader data after execution.
- `Route.LoaderArgs` / `Route.ComponentProps`: Type-safe route module API generated by React Router.

**Syntax Rules:**
- Next.js App Router uses file-system conventions; React Router v7 Framework Mode uses a `routes.ts` configuration file.
- In Next.js, Server Components are the default; use `'use client'` to opt into Client Components.
- In React Router v7, loaders run on the server; actions handle mutations and revalidate loader data.
- Both frameworks support nested layouts, dynamic segments, and route-level code splitting.
- React Router v7 requires Vite 7+ and React 19.2+ for Framework Mode.

**Constraints and Limitations:**
- Next.js App Router is more opinionated; migrating from Pages Router requires significant changes.
- React Router v7 Framework Mode is newer and less battle-tested than Next.js at enterprise scale.
- Server Components in React Router v7 are still in preview (as of v7.9.2) and require the `unstable_reactRouterRSC` Vite plugin.
- Both frameworks have their own conventions and APIs; switching between them requires relearning routing patterns.

### Annotated Code Examples

**Example 1: Next.js App Router with Parallel and Intercepting Routes**

```
app/
├── layout.tsx
├── page.tsx
├── feed/
│   ├── page.tsx              # /feed
│   └── (..)photo/[id]/
│       └── page.tsx          # Intercepts /photo/:id from within /feed
├── photo/[id]/
│   └── page.tsx              # /photo/:id (full page)
└── @modal/
    └── (..)photo/[id]/
        └── page.tsx          # Modal version of photo
```

```tsx
// app/layout.tsx
export default function Layout({
  children,
  modal,
}: {
  children: React.ReactNode;
  modal: React.ReactNode;
}) {
  return (
    <html>
      <body>
        {children}
        {modal}
      </body>
    </html>
  );
}
```

**Expected Output:** When a user clicks a photo in the feed, the photo opens in a modal (intercepting route) while the feed remains visible. The URL changes to `/photo/123`, making it shareable. If the user refreshes the page, the full photo page renders instead of the modal.

**Why This Output Occurs:** The intercepting route `(..)photo/[id]` within the `@modal` slot captures the navigation to `/photo/123` and renders it in the modal slot instead of the main content area. The `(..)` convention matches one route segment above the current level. When the page is loaded directly (refresh), the interception does not occur, and the full photo page renders.

**Example 2: React Router v7 Framework Mode with Nested Routes and Loaders**

```typescript
// app/routes.ts
import { index, route } from "@react-router/dev/routes";

export default [
  index("./routes/home.tsx"),
  route("dashboard", "./routes/dashboard.tsx", [
    index("./routes/dashboard-overview.tsx"),
    route("settings", "./routes/dashboard-settings.tsx"),
    route("analytics", "./routes/dashboard-analytics.tsx"),
  ]),
];
```

```typescript
// app/routes/dashboard.tsx
import { Outlet } from "react-router";
import type { Route } from "./+types/dashboard";

export async function loader({ request }: Route.LoaderArgs) {
  const user = await getUserFromSession(request);
  if (!user) throw redirect("/login");
  return { user };
}

export default function DashboardLayout({ loaderData }: Route.ComponentProps) {
  return (
    <div>
      <aside>Welcome, {loaderData.user.name}</aside>
      <main><Outlet /></main>
    </div>
  );
}
```

**Expected Output:** Navigating to `/dashboard` renders the dashboard layout with a sidebar showing the user's name. The child routes (`/dashboard/settings`, `/dashboard/analytics`) render inside the `<Outlet />`. If the user is not authenticated, they are redirected to `/login`.

**Why This Output Occurs:** The `loader` runs on the server and checks authentication before rendering. The layout component receives the loader data and renders the sidebar. The `<Outlet />` renders the matched child route. This pattern combines authentication, layout composition, and data loading in a single route module.

### Real-World Cases

- **Next.js App Router:** SaaS dashboards, e-commerce platforms, content sites, and any application requiring SSR, SEO, and React Server Components.
- **React Router v7 Framework Mode:** Full-stack applications migrating from Remix, projects that want more architectural control than Next.js offers, and teams that prefer explicit route configuration over file-system conventions.
- **Parallel and Intercepting Routes:** Modals that preserve context (photo galleries, login modals, shopping carts), split-view dashboards, and feeds with detail overlays.
- **Nested Layouts:** Dashboard layouts with persistent sidebars, multi-step forms with shared headers, and documentation sites with consistent navigation.

---

## Core Concept 2: Meta-Framework Router Architectures

### Definitions

**Core Definition:** Meta-framework router architectures are the routing systems provided by Next.js and React Router v7, encompassing file-system routing, dynamic segments, parallel routes, intercepting routes, and layout inheritance.

**Technical Definition:** Meta-framework routing architectures define how URLs map to components, how data is loaded, and how layouts compose. **File-system routing** maps folder structures to URL paths: `app/blog/[slug]/page.tsx` maps to `/blog/:slug`. **Dynamic segments** use square brackets (`[slug]`) to capture URL parameters. **Catch-all segments** (`[...slug]`) capture multiple segments. **Route groups** (`(marketing)`) organise routes without affecting the URL. **Parallel routes** (`@slot`) render multiple pages simultaneously within the same layout, useful for dashboards and split views. **Intercepting routes** (`(..)photo`) load a route from another part of the application within the current layout, commonly used for modals that preserve context. **Layout inheritance** nests layouts hierarchically: `app/dashboard/layout.tsx` wraps `app/dashboard/settings/page.tsx`.

**Beginner-Friendly Explanation:** Modern frameworks use your file system as the router. The folder structure *is* the URL structure. A folder named `[id]` means "this is a dynamic route that captures an ID." A folder named `(marketing)` is a route group—it organises files but doesn't add to the URL. Parallel routes let you show two pages at once (like a dashboard with a sidebar and a main area). Intercepting routes let you open a photo in a modal while keeping the feed behind it, and if you refresh the page, the photo opens as a full page instead.

### Purposes

- To define routes through file-system conventions, reducing configuration boilerplate.
- To support dynamic, catch-all, and optional catch-all segments for flexible URL matching.
- To enable parallel rendering of independent page sections within a shared layout.
- To intercept navigation for modal-like experiences that preserve context and support deep linking.
- To compose layouts hierarchically, sharing UI across routes without duplication.
- To organise routes into groups without affecting URL structure.

### Syntax Rules and Structure

**Next.js App Router — Route Conventions:**

| Convention | Syntax | Purpose |
|------------|--------|---------|
| Static route | `app/dashboard/page.tsx` | `/dashboard` |
| Dynamic segment | `app/blog/[slug]/page.tsx` | `/blog/:slug` |
| Catch-all | `app/docs/[...slug]/page.tsx` | `/docs/a/b/c` |
| Optional catch-all | `app/docs/[[...slug]]/page.tsx` | `/docs` and `/docs/a/b` |
| Route group | `app/(marketing)/about/page.tsx` | `/about` (group ignored in URL) |
| Parallel route | `app/@modal/login/page.tsx` | Renders in `@modal` slot |
| Intercepting route | `app/feed/(..)photo/[id]/page.tsx` | Intercepts `/photo/:id` from `/feed` |
| Private folder | `app/_components/Button.tsx` | Excluded from routing |

**Component Breakdown:**
- `[slug]`: Dynamic segment; matches a single URL segment.
- `[...slug]`: Catch-all segment; matches multiple segments.
- `(marketing)`: Route group; organises routes without affecting the URL.
- `@modal`: Named slot for parallel routes; passed as a prop to the parent layout.
- `(..)photo`: Intercepting route; matches a route one level above the current segment.

**React Router v7 — Route Configuration:**

```typescript
// app/routes.ts
import { index, route, layout, prefix } from "@react-router/dev/routes";

export default [
  index("./home.tsx"),
  route("about", "./about.tsx"),
  layout("./auth-layout.tsx", [
    route("login", "./login.tsx"),
    route("register", "./register.tsx"),
  ]),
  ...prefix("dashboard", [
    index("./dashboard-overview.tsx"),
    route("settings", "./dashboard-settings.tsx"),
  ]),
];
```

**Component Breakdown:**
- `index("./home.tsx")`: Index route for the parent path.
- `route("about", "./about.tsx")`: Static route.
- `layout("./auth-layout.tsx", [...])`: Layout route with child routes.
- `prefix("dashboard", [...])`: Adds a path prefix to all child routes.

**Syntax Rules:**
- Next.js uses file-system conventions; React Router v7 Framework Mode uses a `routes.ts` configuration file.
- Dynamic segments in Next.js use `[param]`; in React Router v7, use `:param` in the path string.
- Parallel routes in Next.js use `@slot` folders; in React Router v7, use layout routes with `<Outlet />`.
- Intercepting routes are a Next.js App Router feature; React Router v7 does not have a direct equivalent.
- Route groups in Next.js use `(group)` folders; React Router v7 uses `layout` routes for grouping.
- Layouts inherit hierarchically: each layout wraps its child routes.

**Constraints and Limitations:**
- Parallel routes cannot mix prerendered and dynamically rendered slots at the same route segment level.
- Intercepting routes use route segments, not the file system, for relative matching — `@slot` folders are ignored.
- `default.js` is required for unmatched parallel route slots on full-page reload; otherwise, a 404 is rendered.
- Route groups in Next.js can create multiple root layouts, but only one root layout can contain `<html>` and `<body>`.
- React Router v7 Framework Mode requires a Vite plugin and is less mature than Next.js for parallel/intercepting patterns.

### Annotated Code Examples

**Example 1: Next.js Parallel Routes for a Dashboard**

```
app/
├── layout.tsx
├── dashboard/
│   ├── layout.tsx
│   ├── @team/
│   │   └── page.tsx          # Team panel
│   └── @analytics/
│       └── page.tsx          # Analytics panel
```

```tsx
// app/dashboard/layout.tsx
export default function DashboardLayout({
  children,
  team,
  analytics,
}: {
  children: React.ReactNode;
  team: React.ReactNode;
  analytics: React.ReactNode;
}) {
  return (
    <div className="dashboard">
      {children}
      <div className="panels">
        {team}
        {analytics}
      </div>
    </div>
  );
}
```

**Expected Output:** The dashboard renders three sections simultaneously: the main content (`children`), the team panel (`@team`), and the analytics panel (`@analytics`). Each slot can have its own loading state and error boundary.

**Why This Output Occurs:** The `@team` and `@analytics` folders define named slots. The layout component receives them as props and renders them in parallel. Each slot can be navigated independently — navigating to `/dashboard/settings` updates the main content while keeping the team and analytics panels active.

**Example 2: React Router v7 Nested Layouts with Loaders**

```typescript
// app/routes.ts
import { route } from "@react-router/dev/routes";

export default [
  route("dashboard", "./dashboard-layout.tsx", [
    route("overview", "./overview.tsx", { loader: overviewLoader }),
    route("settings", "./settings.tsx", { loader: settingsLoader }),
  ]),
];
```

```typescript
// app/dashboard-layout.tsx
import { Outlet, useLoaderData } from "react-router";

export async function loader() {
  const user = await getCurrentUser();
  return { user };
}

export default function DashboardLayout() {
  const { user } = useLoaderData<typeof loader>();

  return (
    <div>
      <header>Dashboard — {user.name}</header>
      <nav>
        <Link to="overview">Overview</Link>
        <Link to="settings">Settings</Link>
      </nav>
      <main><Outlet /></main>
    </div>
  );
}
```

**Expected Output:** Navigating to `/dashboard/overview` renders the dashboard layout with the user's name in the header and the overview content in the main area. Navigating to `/dashboard/settings` updates only the main area, preserving the header and navigation.

**Why This Output Occurs:** The `dashboard-layout.tsx` route module defines a layout with a loader that fetches the current user. The `<Outlet />` renders the matched child route (`overview` or `settings`). The layout persists across navigation because it is not re-rendered — only the child route changes.

### Real-World Cases

- **E-commerce:** Parallel routes for product details and recommendations; intercepting routes for quick-view modals.
- **SaaS dashboards:** Parallel routes for team and analytics panels; nested layouts for settings pages.
- **Social media:** Intercepting routes for photo modals in feeds; parallel routes for sidebar and main content.
- **Documentation sites:** Nested layouts for sidebar navigation and content; route groups for versioned docs.
- **Multi-tenant platforms:** Route groups for tenant-specific layouts; parallel routes for tenant switchers.

---

## Core Concept 3: Unified Build Ecosystems

### Definitions

**Core Definition:** Unified build ecosystems are the bundlers, transpilers, and development servers—principally Vite and Turbopack—that power modern React frameworks, providing fast development, code-splitting, and optimised production builds.

**Technical Definition:** Modern React frameworks use one of two primary build ecosystems. **Vite** (esbuild + Rollup, with Rolldown coming) is a universal build tool that works with React, Vue, Svelte, Solid, and other frameworks. It provides instant dev server startup (300–500ms), hot module replacement (HMR) in under 50ms, and production builds via Rollup. **Turbopack** is a Rust-based bundler built by Vercel as the successor to Webpack, deeply integrated into Next.js. It offers cold starts under 1 second, HMR in under 10ms for leaf components, and production builds that are up to 10x faster than Webpack for large apps. Turbopack is not available as a standalone CLI — it is usable only through Next.js. Vite is the default for React Router v7, SvelteKit, Nuxt, and most non-Next.js frameworks.

**Beginner-Friendly Explanation:** Every React framework needs a build tool to turn your code into something the browser can understand. Vite is the universal option—it's fast, works with almost everything, and has thousands of plugins. Turbopack is Vercel's new Rust-based bundler that's incredibly fast, but it only works with Next.js. If you're using Next.js, you get Turbopack by default. If you're using React Router v7 or anything else, you use Vite.

### Purposes

- To provide fast development server startup and hot module replacement for rapid iteration.
- To transpile JSX, TypeScript, and modern JavaScript into browser-compatible code.
- To split code into chunks for optimal loading performance.
- To optimise production builds through minification, tree-shaking, and asset hashing.
- To integrate framework-specific transformations (SSR, RSC, file-system routing).
- To support plugin ecosystems for extending build capabilities.

### Syntax Rules and Structure

**Vite Configuration for React Router v7:**

```typescript
// vite.config.ts
import { defineConfig } from "vite";
import { reactRouter } from "@react-router/dev/vite";
import tsconfigPaths from "vite-tsconfig-paths";

export default defineConfig({
  plugins: [
    reactRouter(),        // React Router Framework Mode plugin
    tsconfigPaths(),      // Path alias support
  ],
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          'react-vendor': ['react', 'react-dom', 'react-router'],
          'query-vendor': ['@tanstack/react-query'],
        },
      },
    },
  },
});
```

**Component Breakdown:**
- `reactRouter()`: Enables Framework Mode with SSR, file-system routes, and type generation.
- `tsconfigPaths()`: Resolves path aliases from `tsconfig.json`.
- `manualChunks`: Groups vendor dependencies into stable chunks.

**Turbopack Configuration in Next.js:**

```typescript
// next.config.ts
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  turbopack: {
    // Turbopack is the default bundler in Next.js 16+
  },
};

export default nextConfig;
```

**Component Breakdown:**
- Turbopack is enabled by default in Next.js 16+. No additional configuration is required.
- For custom Webpack configuration, Turbopack compatibility must be considered.

**Build Tool Comparison:**

| Feature | Vite | Turbopack |
|---------|------|-----------|
| **Written in** | JavaScript (esbuild + Rollup) | Rust |
| **Cold start** | 300–500ms | Under 1 second |
| **HMR (leaf)** | Under 50ms | Under 10ms |
| **Framework support** | Universal (React, Vue, Svelte, etc.) | Next.js only |
| **Plugin ecosystem** | Thousands (Rollup + Vite plugins) | Minimal (Next.js config) |
| **Production bundler** | Rollup (Rolldown coming) | Turbopack |
| **Stable since** | Vite 2.0 (2021) | Next.js 15 (2025) |

**Syntax Rules:**
- Vite is the default for React Router v7, SvelteKit, Nuxt, and non-Next.js frameworks.
- Turbopack is the default in Next.js 16+ and cannot be used outside Next.js.
- Both tools support code-splitting, tree-shaking, and production optimisation.
- Vite's plugin ecosystem is significantly larger than Turbopack's.
- Manual chunk configuration is available in both tools for grouping vendor dependencies.

**Constraints and Limitations:**
- Turbopack is Next.js-only — there is no standalone CLI or adapter for other frameworks.
- Vite's HMR can drift to 300–400ms on very large codebases; Turbopack maintains constant HMR speed.
- Vite 8's Rolldown bundler outperforms Turbopack in production builds for some benchmarks (1.67s vs 7.38s on a 33-route app).
- Turbopack is still maturing — some Webpack plugins are not yet compatible.

### Annotated Code Examples

**Example 1: Vite Configuration with Code-Splitting**

```typescript
// vite.config.ts
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
  build: {
    rollupOptions: {
      output: {
        manualChunks(id) {
          if (id.includes('node_modules')) {
            if (id.includes('react')) return 'react-vendor';
            if (id.includes('@tanstack')) return 'query-vendor';
            return 'vendor';
          }
          if (id.includes('/pages/Dashboard')) return 'dashboard';
          if (id.includes('/pages/Settings')) return 'settings';
        },
      },
    },
  },
});
```

**Expected Output:** The production build creates separate chunks for React, TanStack Query, the dashboard page, and the settings page. Navigating to `/dashboard` loads only the dashboard chunk, not the settings chunk.

**Why This Output Occurs:** The `manualChunks` function inspects module IDs and groups them into named chunks. Vendor dependencies are grouped into stable chunks that cache well across deployments. Feature pages are split into their own chunks for lazy loading.

### Real-World Cases

- **Next.js applications:** Turbopack provides the fastest HMR and production builds for Next.js apps.
- **React Router v7 applications:** Vite provides fast dev server startup and a rich plugin ecosystem.
- **Monorepos:** Vite's workspace support and Turbopack's incremental compilation both benefit large codebases.
- **Design systems:** Vite's library mode and Turbopack's integration with Next.js support component library development.
- **Edge deployments:** Both tools produce optimised bundles for edge runtime deployment.

---

## Core Concept 4: Deployment Primitives

### Definitions

**Core Definition:** Deployment primitives are the runtime environments—Node.js servers, serverless functions, and edge runtimes—where React framework applications execute in production.

**Technical Definition:** Modern React frameworks target three deployment primitives. **Node.js Runtime** provides full access to Node.js APIs and all npm packages, but requires managing infrastructure (or using a platform like Vercel). **Serverless Node.js** runs on platforms like AWS Lambda, offering high scalability with cold boots around 250ms and a 50MB code size limit. **Edge Runtime** runs on V8 isolates (Cloudflare Workers, Vercel Edge), offering instant cold boots, global distribution, and the lowest latency, but with a limited API subset (no `fs`, limited `crypto`) and a 1–4MB code size limit. Next.js allows per-route runtime selection: `export const runtime = 'edge'` opts a route into the Edge Runtime; otherwise, the Node.js runtime is used. React Router v7 uses adapters (`@react-router/express`, `@react-router/cloudflare`, `@react-router/architect`) to deploy to different platforms.

**Beginner-Friendly Explanation:** Your React app needs to run somewhere. You have three main choices. Node.js servers are the most powerful — they can do anything, but you have to manage them. Serverless functions are easier — the platform manages them, they scale automatically, but they take ~250ms to "wake up." Edge runtimes are the fastest — they run all over the world and start instantly, but they can only do a limited set of things. Next.js lets you choose per route: use Edge for a fast auth check, use Node.js for a database-heavy API.

### Purposes

- To deploy React applications to the runtime that best matches their performance and compatibility needs.
- To achieve low latency through edge deployment for latency-sensitive, stateless operations.
- To access full Node.js APIs for database-heavy, dependency-heavy workloads.
- To scale automatically without managing infrastructure through serverless deployment.
- To select runtimes per route in Next.js for optimal performance.
- To deploy React Router v7 applications to any platform through adapters.

### Syntax Rules and Structure

**Next.js Runtime Selection:**

```typescript
// app/api/hello/route.ts — Edge Runtime
export const runtime = 'edge';

export async function GET() {
  return Response.json({ message: 'Hello from the edge' });
}
```

```typescript
// app/api/heavy/route.ts — Node.js Runtime (default)
import { db } from '@/lib/db';

export async function GET() {
  const data = await db.query('SELECT * FROM heavy_table');
  return Response.json(data);
}
```

**Component Breakdown:**
- `export const runtime = 'edge'`: Opts the route into the Edge Runtime.
- Omitting `runtime`: Uses the default Node.js runtime.
- Edge Runtime supports `fetch`, `Request`, `Response`, and a subset of Web APIs.
- Node.js Runtime supports all Node.js APIs and npm packages.

**React Router v7 Deployment Adapters:**

```typescript
// server/index.ts — Express adapter
import { createRequestHandler } from "@react-router/express";
import express from "express";

const app = express();

app.all(
  "*",
  createRequestHandler({
    build: require("./build"),
  })
);

app.listen(3000);
```

```typescript
// server/cloudflare.ts — Cloudflare adapter
import { createRequestHandler } from "@react-router/cloudflare";

export default {
  async fetch(request: Request, env: Env) {
    return createRequestHandler({ build: await import("./build/server") })(
      request,
      env
    );
  },
};
```

**Component Breakdown:**
- `@react-router/express`: Adapter for Express servers (Node.js runtime).
- `@react-router/cloudflare`: Adapter for Cloudflare Workers (Edge runtime).
- `@react-router/architect`: Adapter for AWS Lambda (serverless).
- Each adapter translates the platform's request/response primitives to the Web Fetch API.

**Runtime Comparison:**

| Feature | Node.js | Serverless Node.js | Edge Runtime |
|---------|---------|-------------------|--------------|
| **Cold boot** | N/A (always on) | ~250ms | Instant |
| **API access** | Full Node.js | Full Node.js | `fetch` only |
| **Code size limit** | N/A | 50MB | 1–4MB |
| **Scalability** | Manual | High | Highest |
| **Latency** | Normal | Low | Lowest |
| **npm packages** | All | All | Limited subset |
| **Best for** | Complex apps | Scalable APIs | Latency-sensitive, stateless |

**Syntax Rules:**
- Use `export const runtime = 'edge'` in Next.js route handlers or pages to opt into the Edge Runtime.
- Use the Node.js runtime (default) for database-heavy, dependency-heavy, or stateful workloads.
- Use Edge Runtime for authentication checks, personalisation, and latency-sensitive stateless operations.
- React Router v7 requires an adapter for deployment — choose the adapter that matches your platform.
- Edge Runtime code should use HTTP-friendly drivers (Neon Serverless, Turso/libSQL, Upstash) instead of raw TCP connection pools.

**Constraints and Limitations:**
- Edge Runtime does not support `fs`, raw TCP sockets, or many Node.js APIs.
- Edge Runtime code size is limited to 1–4MB on Vercel.
- Serverless functions have cold boot latency (~250ms) and a 50MB code size limit.
- Node.js runtime requires infrastructure management unless deployed to a platform like Vercel.
- Next.js has deprecated the Edge Runtime for new routes, recommending Node.js by default.

### Annotated Code Examples

**Example 1: Next.js Route with Edge Runtime for Auth**

```typescript
// app/api/auth/check/route.ts
export const runtime = 'edge';

export async function GET(request: Request) {
  const token = request.headers.get('Authorization')?.replace('Bearer ', '');

  if (!token) {
    return new Response(JSON.stringify({ error: 'Unauthorized' }), {
      status: 401,
      headers: { 'Content-Type': 'application/json' },
    });
  }

  try {
    const payload = await verifyJWT(token);
    return Response.json({ userId: payload.sub });
  } catch {
    return new Response(JSON.stringify({ error: 'Invalid token' }), {
      status: 401,
    });
  }
}
```

**Expected Output:** The auth check runs at the edge with instant cold boot and low latency. It verifies the JWT and returns the user ID or a 401 error.

**Why This Output Occurs:** The `export const runtime = 'edge'` directive opts the route into the Edge Runtime. The route uses only Web APIs (`Request`, `Response`, `fetch`), which are available in the Edge Runtime. This makes it ideal for latency-sensitive authentication checks.

**Example 2: React Router v7 Deployment to Cloudflare Workers**

```typescript
// server.ts — Cloudflare Workers entry
import { createRequestHandler } from "@react-router/cloudflare";
import * as build from "./build/server";

export default {
  async fetch(request: Request, env: Env, ctx: ExecutionContext) {
    const handler = createRequestHandler({
      build,
      getLoadContext: () => ({ env, ctx }),
    });

    return handler(request);
  },
};
```

```typescript
// app/routes/dashboard.tsx — Loader with Cloudflare env
export async function loader({ context }: Route.LoaderArgs) {
  const db = context.env.DB; // Cloudflare D1 database binding
  const users = await db.prepare('SELECT * FROM users').all();
  return { users: users.results };
}
```

**Expected Output:** The React Router v7 app runs on Cloudflare Workers, using the D1 database binding from the Cloudflare environment. Loaders access the database through the `context.env` object.

**Why This Output Occurs:** The Cloudflare adapter translates the Workers request into a format React Router can handle. The `getLoadContext` function passes the Cloudflare `env` object to loaders and actions. Loaders access the D1 database binding through `context.env.DB`.

### Real-World Cases

- **Edge authentication:** Auth checks at the edge with instant cold boots for low-latency login flows.
- **Personalisation:** Edge middleware that reads cookies and personalises content without a database round-trip.
- **Serverless APIs:** REST or GraphQL APIs deployed to AWS Lambda with automatic scaling.
- **Node.js servers:** Complex applications with database connections, background jobs, and heavy dependencies.
- **Hybrid deployments:** Next.js apps that use Edge for auth and Node.js for data-heavy routes.
- **Multi-cloud:** React Router v7 apps deployed to Cloudflare Workers, AWS Lambda, or Express servers using the same codebase with different adapters.

---

## References

- File-system conventions: Intercepting Routes – Next.js: https://nextjs.org/docs/app/api-reference/file-conventions/intercepting-routes
- File-system conventions: Parallel Routes – Next.js: https://nextjs.org/docs/app/api-reference/file-conventions/parallel-routes
- Picking a Mode – React Router v7: https://reactrouter.com/7.4.0/start/modes
- React Stack Patterns – PatternsDev: https://github.com/PatternsDev/skills/blob/9683dda38ff9b3d44ad7d80e67f245c6a9bb9c52/react/react-2026/SKILL.md
- Edge and Node.js Runtimes – Next.js: https://raw.githubusercontent.com/vercel/next.js/0ef13d5bd2de1912414e8a98332891e9f24f7363/docs/02-app/01-building-your-application/02-rendering/02-edge-and-nodejs-runtimes.mdx
- Middleware – React Router v7: https://reactrouter.com/7.7.1/how-to/middleware
- Pages and Layouts – Next.js: https://nextjs.org/docs/14/app/building-your-application/routing/pages-and-layouts.md
- Server Adapters – React Router v7: https://reactrouter.com/7.11.0/api/other-api/adapter
- Turbopack vs Vite: Next-Gen Bundler Battle 2026 – PkgPulse: https://www.pkgpulse.com/guides/turbopack-vs-vite-2026
- Dynamic Segments – Next.js: https://nextjs.org/docs/app/api-reference/file-conventions/dynamic-routes
- Framework Adoption from Component Routes – React Router v7: https://reactrouter.com/7.9.2/upgrading/framework-adoption
- Route Module – React Router v7: https://reactrouter.com/7.0.1/api/route-module
- Data Mode – React Router v7: https://www.mintlify.com/react-router/data-mode
- Edge Runtime – Next.js: https://nextjs.org/docs/app/api-reference/edge
- Runtime Selection – Next.js Best Practices: https://github.com/aiskillstore/marketplace/blob/36e07d5e13068e5be64447e8f20b427cf2cbd21a/marketplace/skills/vercel-labs/next-best-practices/runtime-selection.md