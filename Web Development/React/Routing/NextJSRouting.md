# Next.js Routing — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Next.js routing is a file-system-based routing system where folders and files define URL segments, layouts, loading states, errors, and API endpoints.

**Technical Definition:** Next.js provides two routers. The **App Router** (`app/` directory, default since Next.js 13.4) is built on React Server Components, nested layouts, streaming, and Suspense. It uses special file conventions (`page.tsx`, `layout.tsx`, `loading.tsx`, `error.tsx`, `template.tsx`, `route.ts`) to define routes and their behaviour. The **Pages Router** (`pages/` directory, legacy) uses `pages/about.tsx` for pages and `pages/api/*.ts` for API routes. Both use file-based routing: folders map to URL segments, dynamic segments use `[param]`, catch-all segments use `[...param]`, optional catch-all uses `[[...param]]`, and route groups `(folder)` organise files without affecting the URL. The App Router adds **parallel routes** (`@slot`) for simultaneous rendering of multiple pages in one layout, **intercepting routes** (`(.)`, `(..)`, `(...)`) for modal patterns, **route handlers** (`route.ts`) for API endpoints, and **middleware** (`middleware.ts`) for request-time logic.

**Beginner-Friendly Explanation:** Next.js routing is the most literal kind of routing: your folder structure *is* your URL structure. A folder called `about` with a `page.tsx` inside becomes `/about`. A folder called `[id]` becomes a dynamic segment that matches `/123`, `/abc`, and so on. The App Router adds special files that give each route its own layout, loading skeleton, and error boundary without you writing a single route configuration. It also adds advanced patterns — parallel routes for dashboards with multiple panels, intercepting routes for modals that have their own URL, and route handlers for building APIs alongside your pages.

### Key Characteristics

- **File-system routing:** Folders and files map directly to URL segments and route behaviour.
- **App Router is the default:** React Server Components, nested layouts, streaming, and Suspense are built in.
- **Special file conventions:** `page`, `layout`, `loading`, `error`, `not-found`, `template`, `route`, `default` — each has a specific role.
- **Dynamic and catch-all segments:** `[id]`, `[...slug]`, `[[...slug]]` for flexible URL patterns.
- **Route groups:** `(folder)` organises files without adding URL segments.
- **Parallel routes:** `@slot` renders multiple pages simultaneously in one layout.
- **Intercepting routes:** `(.)`, `(..)`, `(...)` render a route in the context of another route (modal pattern).
- **Route handlers:** `route.ts` files export HTTP method handlers for APIs.
- **Middleware:** `middleware.ts` runs before a request is completed for auth, redirects, and rewrites.
- **Colocation:** Folders can contain components, tests, and styles without affecting routing.

### Prerequisites

- React function components, JSX, and props.
- Basic understanding of URLs, path segments, and query strings.
- Familiarity with React Server Components (for App Router).
- Knowledge of TypeScript (for typed route params).
- Understanding of HTTP methods and APIs (for route handlers).

### Related Programming Areas

- **React Server Components** — App Router is built on RSC.
- **Server Actions** — mutations in App Router.
- **Streaming and Suspense** — `loading.tsx` and Suspense boundaries.
- **Middleware** — request-time logic before routing completes.
- **API Routes / Route Handlers** — building backend endpoints alongside the frontend.

---

## 1. Next.js App Router

### Definitions

**Core Definition:** The Next.js App Router is the file-system router in the `app/` directory that uses React Server Components, nested layouts, and special file conventions to define routes and their behaviour.

**Technical Definition:** The App Router (`app/` directory) was introduced in Next.js 13 and became stable in 13.4. It uses a **route segment** model: each folder in `app/` represents a URL segment, and special files (`page.tsx`, `layout.tsx`, `loading.tsx`, `error.tsx`, `not-found.tsx`, `template.tsx`, `route.ts`, `default.tsx`) define how that segment behaves. It supports nested layouts that persist across navigations, streaming with Suspense, Server Components by default, parallel routes, intercepting routes, and route handlers. Dynamic segments use `[param]`; catch-all uses `[...param]`; optional catch-all uses `[[...param]]`; route groups use `(folder)`.

**Beginner-Friendly Explanation:** The App Router is the modern Next.js router. You create folders inside `app/`, and each folder becomes a URL path. A `page.tsx` file makes that folder a page. A `layout.tsx` file wraps the page and its children with shared UI. `loading.tsx` and `error.tsx` give the route automatic loading and error states.

### Purposes

- To define routes using the file system with minimal configuration.
- To share layouts across nested routes without re-rendering them on navigation.
- To stream content progressively with Suspense and `loading.tsx`.
- To use React Server Components by default for better performance.
- To colocate route-specific components, tests, and styles.
- To support advanced patterns (parallel routes, intercepting routes, route handlers, middleware).

### Syntax Rules and Structure

**General Syntax (Route Segments):**

```text
app/
├── layout.tsx          → root layout (required)
├── page.tsx            → "/" route
├── about/
│   └── page.tsx        → "/about" route
├── blog/
│   ├── page.tsx        → "/blog" route
│   └── [slug]/
│       └── page.tsx    → "/blog/:slug" route
└── dashboard/
    ├── layout.tsx      → dashboard layout
    ├── page.tsx        → "/dashboard" route
    └── settings/
        └── page.tsx    → "/dashboard/settings" route
```

**Special Files:**

| File | Purpose |
|------|---------|
| `layout.tsx` | Shared UI that wraps the page and its children; persists across navigation. |
| `page.tsx` | The route's page component; makes the folder publicly routable. |
| `loading.tsx` | Suspense fallback shown while the page loads. |
| `error.tsx` | Error boundary for the route segment. |
| `not-found.tsx` | 404 UI for the segment. |
| `template.tsx` | Like layout, but re-mounts on navigation. |
| `default.tsx` | Fallback for parallel route slots. |
| `route.ts` | HTTP method handlers (API endpoint). |

**Syntax Rules:**

1. The root `app/layout.tsx` is required and must contain `<html>` and `<body>` tags.
2. A folder becomes a route only when it contains a `page.tsx` or `route.ts`.
3. Folders without a `page.tsx` can still contain layouts, components, and styles.
4. Special files are colocated with the route they affect.
5. Nested layouts wrap child routes and persist across navigation.

**Constraints and Limitations:**

- The App Router requires React 18+ and Next.js 13.4+.
- Server Components cannot use client-side Hooks (`useState`, `useEffect`) without `'use client'`.
- Layouts do not re-render on navigation; use `template.tsx` if you need re-mounting.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Basic Page and Layout

```tsx
// app/layout.tsx
export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>
        <header>My App</header>
        <main>{children}</main>
      </body>
    </html>
  );
}
```

```tsx
// app/page.tsx
export default function HomePage() {
  return <h1>Welcome</h1>;
}
```

**Expected Output:** `/` renders “Welcome” inside the root layout with the header.

**Why this output:** `app/layout.tsx` defines the shared shell. `app/page.tsx` defines the `/` route. The layout wraps the page and persists across navigations.

#### Example 2: Nested Layout with Loading and Error

```tsx
// app/dashboard/layout.tsx
export default function DashboardLayout({ children }: { children: React.ReactNode }) {
  return (
    <div className="dashboard">
      <Sidebar />
      <section>{children}</section>
    </div>
  );
}
```

```tsx
// app/dashboard/page.tsx
export default async function DashboardPage() {
  const stats = await fetchStats();
  return <StatsGrid stats={stats} />;
}
```

```tsx
// app/dashboard/loading.tsx
export default function Loading() {
  return <DashboardSkeleton />;
}
```

```tsx
// app/dashboard/error.tsx
'use client';
export default function Error({ error, reset }: { error: Error; reset: () => void }) {
  return (
    <div>
      <p>Something went wrong.</p>
      <button onClick={reset}>Try again</button>
    </div>
  );
}
```

**Expected Output:** Navigating to `/dashboard` renders the sidebar layout. While the page loads, `DashboardSkeleton` appears. If `fetchStats` throws, the error boundary shows the fallback with a retry button.

**Why this output:** `loading.tsx` provides the Suspense fallback for the segment. `error.tsx` catches errors from the page and its children. The layout persists across dashboard sub-routes.

### Real-World Cases

- **Dashboards:** Nested layouts for sidebar and header, with per-widget loading states.
- **E-commerce:** Product pages with shared layouts and dynamic segments.
- **Content sites:** Blog routes with dynamic slugs and streaming.
- **SaaS:** Authenticated routes with middleware and route handlers.

### References

- Next.js — App Router — https://nextjs.org/docs/app
- Next.js — Routing Fundamentals — https://nextjs.org/docs/app/building-your-application/routing
- Next.js — Project Structure — https://nextjs.org/docs/app/getting-started/project-structure
- Next.js — Layouts and Pages — https://nextjs.org/docs/app/getting-started/layouts-and-pages

---

## 2. Next.js Pages Router

### Definitions

**Core Definition:** The Next.js Pages Router is the legacy file-system router in the `pages/` directory that uses `pages/about.tsx` for pages and `pages/api/*.ts` for API routes.

**Technical Definition:** The Pages Router (`pages/` directory) is Next.js's original router. Each file in `pages/` becomes a route: `pages/index.tsx` → `/`, `pages/about.tsx` → `/about`, `pages/blog/[slug].tsx` → `/blog/:slug`. API routes live in `pages/api/` and export a default handler receiving `req` and `res`. It supports `getServerSideProps`, `getStaticProps`, and `getStaticPaths` for data fetching. The Pages Router remains supported but new projects should use the App Router.

**Beginner-Friendly Explanation:** The Pages Router is the older Next.js routing system. Each file in the `pages` folder becomes a page. API endpoints go in `pages/api`.

### Purposes

- To maintain existing Next.js applications built with the Pages Router.
- To use `getServerSideProps` / `getStaticProps` data fetching.
- To build API routes with `pages/api/`.
- To support legacy patterns while migrating to the App Router.

### Syntax Rules and Structure

**General Syntax:**

```text
pages/
├── index.tsx          → "/"
├── about.tsx          → "/about"
├── blog/
│   ├── index.tsx      → "/blog"
│   └── [slug].tsx     → "/blog/:slug"
└── api/
    └── hello.ts       → "/api/hello"
```

**Syntax Rules:**

1. Each file in `pages/` becomes a route; the file path maps to the URL.
2. `index.tsx` maps to the folder's root path.
3. Dynamic segments use `[param].tsx`; catch-all uses `[...param].tsx`.
4. API routes live in `pages/api/` and export a default handler.
5. Data fetching uses `getServerSideProps`, `getStaticProps`, and `getStaticPaths`.

**Constraints and Limitations:**

- No React Server Components.
- No nested layouts (use `_app.tsx` for shared UI).
- No streaming or Suspense-based loading UI.
- New projects should use the App Router.

### Annotated Code Examples

```tsx
// pages/blog/[slug].tsx
import { GetServerSideProps } from 'next';

export default function BlogPost({ post }: { post: Post }) {
  return <h1>{post.title}</h1>;
}

export const getServerSideProps: GetServerSideProps = async ({ params }) => {
  const post = await fetchPost(params?.slug as string);
  return { props: { post } };
};
```

**Expected Output:** `/blog/hello` renders the post fetched by `getServerSideProps`.

**Why this output:** The file path `pages/blog/[slug].tsx` maps to `/blog/:slug`. `getServerSideProps` fetches data on each request and passes it as props.

### Real-World Cases

- **Legacy applications:** Maintaining existing Pages Router codebases.
- **Incremental migration:** Moving pages to the App Router one route at a time.
- **API endpoints:** Using `pages/api/` for simple APIs.

### References

- Next.js — Pages Router — https://nextjs.org/docs/pages
- Next.js — Routing (Pages Router) — https://nextjs.org/docs/pages/building-your-application/routing
- Next.js — App Router vs Pages Router — https://nextjs.org/docs/app/building-your-application/routing

---

## 3. File-Based Routing

### Definitions

**Core Definition:** File-based routing is the Next.js model where the file system structure directly defines the application's URL structure.

**Technical Definition:** In both App and Pages Routers, folders and files map to URL segments. A folder becomes a URL segment; a `page.tsx` (App Router) or a file (Pages Router) makes it routable. Nested folders create nested URLs. The file system is the single source of truth for routes, eliminating separate route configuration files.

**Beginner-Friendly Explanation:** File-based routing means your folders are your URLs. Create a folder called `about`, add a `page.tsx`, and you have an `/about` route.

### Purposes

- To eliminate manual route configuration.
- To make the URL structure visible in the file system.
- To colocate route-specific files (components, tests, styles).
- To enable intuitive navigation and refactoring.

### Syntax Rules and Structure

```text
app/
├── page.tsx              → "/"
├── about/page.tsx        → "/about"
├── blog/page.tsx         → "/blog"
├── blog/[slug]/page.tsx  → "/blog/:slug"
└── (marketing)/pricing/page.tsx → "/pricing" (route group)
```

**Syntax Rules:**

1. Folders define URL segments; `page.tsx` makes them routable.
2. Nested folders create nested URL paths.
3. Special files (`layout`, `loading`, `error`) are colocated with their route.
4. Folders without a `page.tsx` or `route.ts` are not routable.

### Annotated Code Examples

```text
app/
├── page.tsx                    → "/"
├── products/
│   ├── page.tsx                → "/products"
│   └── [id]/
│       └── page.tsx            → "/products/:id"
```

**Expected Output:** `/products` renders the products list; `/products/123` renders the product detail for ID 123.

**Why this output:** The folder `products` maps to `/products`. The dynamic folder `[id]` maps to any value after `/products/`.

### Real-World Cases

- **All Next.js applications:** File-based routing is the foundation.
- **Large codebases:** Intuitive URL structure reduces onboarding time.
- **Colocation:** Route-specific files live next to the route.

### References

- Next.js — Project Structure — https://nextjs.org/docs/app/getting-started/project-structure
- Next.js — Routing Fundamentals — https://nextjs.org/docs/app/building-your-application/routing

---

## 4. Dynamic Routes

### Definitions

**Core Definition:** Dynamic routes are routes with variable segments, defined by wrapping a folder name in square brackets (`[param]`), allowing one route pattern to match many URLs.

**Technical Definition:** A dynamic segment folder (e.g., `[id]`) matches any value at that position. The matched value is passed to the page as a `params` prop (`{ id: '123' }`). Dynamic segments can be nested (`[category]/[id]`) and combined with static segments. In the App Router, `params` is a Promise in Next.js 15+ and must be awaited in async Server Components; in the Pages Router, params are available on the context object.

**Beginner-Friendly Explanation:** Dynamic routes are flexible URL slots. A folder called `[id]` matches any value, and your page receives that value as a parameter.

### Purposes

- To create routes that match variable values (IDs, slugs).
- To avoid defining a separate route for every possible value.
- To support detail pages (products, users, posts).
- To pass dynamic values to data fetching and rendering.

### Syntax Rules and Structure

```text
app/
├── products/
│   └── [id]/
│       └── page.tsx    → "/products/:id"
├── blog/
│   └── [category]/
│       └── [slug]/
│           └── page.tsx → "/blog/:category/:slug"
```

```tsx
// app/products/[id]/page.tsx
export default async function ProductPage({ params }: { params: Promise<{ id: string }> }) {
  const { id } = await params;
  const product = await fetchProduct(id);
  return <h1>{product.name}</h1>;
}
```

**Syntax Rules:**

1. Wrap the folder name in brackets to create a dynamic segment.
2. The page receives `params` with the matched values.
3. In Next.js 15+, `params` is a Promise and must be awaited.
4. Dynamic segments can be nested and combined with static segments.
5. Use `generateStaticParams` to pre-render dynamic routes at build time.

**Constraints and Limitations:**

- Params are strings; parse and validate them.
- In Next.js 15+, forgetting to await `params` causes errors.
- Dynamic routes cannot have static siblings with the same pattern.

### Annotated Code Examples

```tsx
// app/blog/[slug]/page.tsx
export default async function BlogPost({ params }: { params: Promise<{ slug: string }> }) {
  const { slug } = await params;
  const post = await fetchPost(slug);
  if (!post) notFound();
  return <article><h1>{post.title}</h1></article>;
}
```

**Expected Output:** `/blog/hello-world` renders the post with slug `hello-world`. Missing posts call `notFound()` and render the 404 page.

**Why this output:** `[slug]` matches the URL segment. `params` contains the slug. `notFound()` triggers the nearest `not-found.tsx`.

### Real-World Cases

- **Product pages:** `/products/[id]`.
- **Blog posts:** `/blog/[slug]`.
- **User profiles:** `/users/[id]`.
- **Nested resources:** `/blog/[category]/[slug]`.

### References

- Next.js — Dynamic Segments — https://nextjs.org/docs/app/api-reference/file-conventions/dynamic-routes
- Next.js — Layouts and Pages (dynamic segments) — https://nextjs.org/docs/app/getting-started/layouts-and-pages

---

## 5. Catch-All Routes

### Definitions

**Core Definition:** Catch-all routes are dynamic routes that match one or more URL segments, defined with `[...param]`.

**Technical Definition:** A catch-all folder (`[...slug]`) matches any number of segments after its position. The `params.slug` value is an array of strings (e.g., `['a', 'b', 'c']` for `/a/b/c`). Catch-all routes are useful for documentation, file browsers, and nested content where the depth is not known in advance. They do not match the parent path without a segment; use optional catch-all for that.

**Beginner-Friendly Explanation:** Catch-all routes catch everything after a point. `/docs/[...slug]` matches `/docs/intro`, `/docs/api/router`, and `/docs/a/b/c/d`.

### Purposes

- To match routes with variable depth.
- To build documentation and file-browser routes.
- To handle nested content without defining every level.
- To provide a single route for a hierarchy of pages.

### Syntax Rules and Structure

```text
app/docs/[...slug]/page.tsx → "/docs/:slug*"
```

```tsx
export default async function DocsPage({ params }: { params: Promise<{ slug: string[] }> }) {
  const { slug } = await params; // string[]
  return <DocContent path={slug.join('/')} />;
}
```

**Syntax Rules:**

1. Wrap the folder name in `[...param]` for catch-all.
2. The param is an array of strings.
3. Catch-all requires at least one segment; it does not match the parent path.
4. Use `generateStaticParams` to pre-render known paths.

**Constraints and Limitations:**

- Does not match the parent path without a segment; use optional catch-all.
- The array must be parsed and validated.

### Annotated Code Examples

```tsx
// app/docs/[...slug]/page.tsx
export default async function DocsPage({ params }: { params: Promise<{ slug: string[] }> }) {
  const { slug } = await params;
  const doc = await fetchDoc(slug);
  if (!doc) notFound();
  return <article>{doc.content}</article>;
}
```

**Expected Output:** `/docs/getting-started/install` renders the doc for `['getting-started', 'install']`.

**Why this output:** `[...slug]` captures all segments after `/docs/` as an array.

### Real-World Cases

- **Documentation:** `/docs/[...slug]`.
- **File browsers:** `/files/[...path]`.
- **CMS content:** `/pages/[...slug]`.

### References

- Next.js — Dynamic Segments (catch-all) — https://nextjs.org/docs/app/api-reference/file-conventions/dynamic-routes
- Next.js — Route Segments — https://nextjs.org/docs/app/api-reference/file-conventions/route-segment-config

---

## 6. Optional Catch-All Routes

### Definitions

**Core Definition:** Optional catch-all routes are dynamic routes that match zero or more URL segments, defined with `[[...param]]`.

**Technical Definition:** An optional catch-all folder (`[[...slug]]`) matches the parent path with no segments **and** any number of segments. The `params.slug` is `undefined` when no segments are present, and an array of strings otherwise. This is useful when a hierarchy has an index page and nested pages handled by the same route.

**Beginner-Friendly Explanation:** Optional catch-all routes match both the parent path and everything after it. `/shop` and `/shop/a/b/c` both match `[[...slug]]`.

### Purposes

- To handle both the index and nested paths with one route.
- To simplify hierarchies where the root and children share behaviour.
- To support optional path segments.
- To reduce duplicate route definitions.

### Syntax Rules and Structure

```text
app/shop/[[...slug]]/page.tsx → "/shop" and "/shop/:slug*"
```

```tsx
export default async function ShopPage({ params }: { params: Promise<{ slug?: string[] }> }) {
  const { slug } = await params;
  if (!slug) return <ShopIndex />;
  return <CategoryPage path={slug} />;
}
```

**Syntax Rules:**

1. Wrap the folder name in `[[...param]]` for optional catch-all.
2. The param is `undefined` or an array of strings.
3. It matches both the parent path and nested paths.
4. Use conditional rendering based on whether `slug` is defined.

**Constraints and Limitations:**

- Must handle the `undefined` case explicitly.
- Cannot coexist with a sibling catch-all at the same level.

### Annotated Code Examples

```tsx
// app/shop/[[...slug]]/page.tsx
export default async function ShopPage({ params }: { params: Promise<{ slug?: string[] }> }) {
  const { slug } = await params;
  if (!slug) return <ShopIndex />;
  return <CategoryPage path={slug} />;
}
```

**Expected Output:** `/shop` renders the shop index. `/shop/shoes/running` renders the category page.

**Why this output:** `[[...slug]]` matches both the parent and nested paths. The `undefined` check distinguishes the index from nested paths.

### Real-World Cases

- **Shop hierarchies:** `/shop` and `/shop/category/subcategory`.
- **CMS pages:** `/pages` and `/pages/about/team`.
- **Documentation roots:** `/docs` and `/docs/guide/intro`.

### References

- Next.js — Dynamic Segments (optional catch-all) — https://nextjs.org/docs/app/api-reference/file-conventions/dynamic-routes
- Next.js — Route Segments — https://nextjs.org/docs/app/api-reference/file-conventions/route-segment-config

---

## 7. Route Groups

### Definitions

**Core Definition:** Route groups are folders wrapped in parentheses `(folder)` that organise route files without adding a URL segment.

**Technical Definition:** A route group `(marketing)` groups routes for organisational purposes (e.g., different layouts, team ownership) without appearing in the URL. `/app/(marketing)/about/page.tsx` maps to `/about`, not `/marketing/about`. Route groups enable multiple root layouts, separate layouts per group, and clearer project organisation.

**Beginner-Friendly Explanation:** Route groups are folders that do not show up in the URL. They let you organise files and apply different layouts without changing the address.

### Purposes

- To organise routes by feature, team, or layout.
- To apply different layouts to different sections without URL changes.
- To create multiple root layouts (e.g., marketing vs. app).
- To keep the file system tidy without affecting URLs.

### Syntax Rules and Structure

```text
app/
├── (marketing)/
│   ├── layout.tsx       → marketing layout
│   ├── about/page.tsx   → "/about"
│   └── pricing/page.tsx → "/pricing"
└── (app)/
    ├── layout.tsx       → app layout
    └── dashboard/page.tsx → "/dashboard"
```

**Syntax Rules:**

1. Wrap the folder name in parentheses to create a route group.
2. The group name does not appear in the URL.
3. Each group can have its own `layout.tsx`.
4. Multiple root layouts are possible by placing `layout.tsx` inside each group.

**Constraints and Limitations:**

- Routes across different root layouts require a full page load when navigating between them.
- Group names must be unique.

### Annotated Code Examples

```text
app/
├── (marketing)/
│   ├── layout.tsx
│   └── about/page.tsx    → "/about"
└── (app)/
    ├── layout.tsx
    └── dashboard/page.tsx → "/dashboard"
```

**Expected Output:** `/about` uses the marketing layout; `/dashboard` uses the app layout. Neither URL contains the group name.

**Why this output:** Parentheses exclude the folder from the URL. Each group's `layout.tsx` applies to its routes.

### Real-World Cases

- **Marketing vs. app:** Separate layouts for public and authenticated areas.
- **Team ownership:** Grouping routes by team without affecting URLs.
- **Multiple root layouts:** Different `<html>` shells for different sections.

### References

- Next.js — Route Groups — https://nextjs.org/docs/app/api-reference/file-conventions/route-groups
- Next.js — Project Structure (route groups) — https://nextjs.org/docs/app/getting-started/project-structure

---

## 8. Parallel Routes

### Definitions

**Core Definition:** Parallel routes are named slots defined with `@folder` that allow a layout to render multiple pages simultaneously in the same view.

**Technical Definition:** A parallel route slot (`@analytics`, `@team`) is a folder prefixed with `@`. The parent layout receives each slot as a prop and renders them independently. Slots can have their own `loading.tsx`, `error.tsx`, and `default.tsx`. This is useful for dashboards with independent sections, split views, and conditional rendering of slots. Slots do not affect the URL.

**Beginner-Friendly Explanation:** Parallel routes let a single page show multiple sections at once, each with its own loading and error states. For example, a dashboard can have a `@team` panel and an `@analytics` panel side by side.

### Purposes

- To render multiple pages simultaneously in one layout.
- To give each section independent loading and error states.
- To support dashboards with independent widgets.
- To enable conditional rendering of slots.
- To support split views and multi-pane interfaces.

### Syntax Rules and Structure

```text
app/dashboard/
├── layout.tsx
├── page.tsx
├── @team/
│   ├── page.tsx
│   └── loading.tsx
└── @analytics/
    ├── page.tsx
    └── loading.tsx
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
      <main>{children}</main>
      <aside>{team}</aside>
      <section>{analytics}</section>
    </div>
  );
}
```

**Syntax Rules:**

1. Prefix the folder with `@` to create a parallel route slot.
2. The layout receives the slot as a prop named after the folder (without `@`).
3. Each slot can have its own `loading.tsx`, `error.tsx`, and `default.tsx`.
4. Slots do not affect the URL.
5. Use `default.tsx` to provide fallback content when a slot does not match.

**Constraints and Limitations:**

- Slot names must be unique.
- `default.tsx` is required for slots that may not match on hard navigation.
- Complex slot combinations can be difficult to reason about.

### Annotated Code Examples

```tsx
// app/dashboard/@team/page.tsx
export default async function TeamPage() {
  const team = await fetchTeam();
  return <TeamList team={team} />;
}

// app/dashboard/@team/loading.tsx
export default function TeamLoading() {
  return <TeamSkeleton />;
}
```

**Expected Output:** The dashboard renders the main page, the team panel, and the analytics panel simultaneously. The team panel shows a skeleton while its data loads.

**Why this output:** `@team` is a parallel slot. Its `loading.tsx` provides a Suspense fallback independent of the main page and the analytics slot.

### Real-World Cases

- **Dashboards:** Main content, team panel, and analytics panel.
- **Split views:** List and detail side by side.
- **Conditional slots:** Showing a login modal in a `@auth` slot.
- **Multi-pane interfaces:** Email clients with folders, list, and reading pane.

### References

- Next.js — Parallel Routes — https://nextjs.org/docs/app/api-reference/file-conventions/parallel-routes
- Next.js — Parallel Routes (guide) — https://nextjs.org/docs/app/building-your-application/routing/parallel-routes

---

## 9. Intercepting Routes

### Definitions

**Core Definition:** Intercepting routes are routes that render in the context of another route, typically to show a modal over the current page while preserving the URL of the intercepted route.

**Technical Definition:** Intercepting routes use `(.)` for same-level, `(..)` for one level up, and `(...)` for root-level interception. When a user navigates to the intercepted route from within the app, the intercepted version renders (e.g., a modal); on hard navigation or refresh, the full page renders. This is the pattern for photo modals, login dialogs, and quick-view overlays that have shareable URLs.

**Beginner-Friendly Explanation:** Intercepting routes let a page open as a modal when you click a link, but as a full page if you refresh or share the URL.

### Purposes

- To render modals with their own URLs.
- To preserve the underlying page when showing a modal.
- To support shareable links to modal content.
- To provide a full-page fallback on hard navigation.
- To implement login dialogs and quick views.

### Syntax Rules and Structure

```text
app/
├── feed/
│   └── page.tsx
├── photo/
│   └── [id]/page.tsx          → full page
└── feed/
    └── (.)photo/
        └── [id]/page.tsx      → intercepted (modal)
```

**Syntax Rules:**

1. Use `(.)` to intercept a route at the same level.
2. Use `(..)` to intercept a route one level up.
3. Use `(...)` to intercept a route from the root.
4. The intercepted route renders in the current layout; the full route renders on hard navigation.
5. Use a modal component in the intercepted page.

**Constraints and Limitations:**

- Requires careful folder placement.
- Hard navigation always renders the full page, not the intercepted version.
- State and scroll position must be managed explicitly.

### Annotated Code Examples

```tsx
// app/feed/(.)photo/[id]/page.tsx — intercepted modal
export default async function PhotoModal({ params }: { params: Promise<{ id: string }> }) {
  const { id } = await params;
  return (
    <Modal>
      <Photo id={id} />
    </Modal>
  );
}
```

**Expected Output:** Clicking a photo in the feed opens a modal at `/photo/123`. Refreshing the page renders the full photo page instead.

**Why this output:** The `(.)photo` intercepts the navigation from the feed, rendering the modal version. Hard navigation bypasses the interception and renders `app/photo/[id]/page.tsx`.

### Real-World Cases

- **Photo feeds:** Modal photo views with shareable URLs.
- **Login dialogs:** A `/login` route that opens as a modal from the app.
- **Quick views:** Product quick-view modals with full-page fallbacks.

### References

- Next.js — Intercepting Routes — https://nextjs.org/docs/app/api-reference/file-conventions/intercepting-routes
- Next.js — Intercepting Routes (guide) — https://nextjs.org/docs/app/building-your-application/routing/intercepting-routes

---

## 10. Route Handlers

### Definitions

**Core Definition:** Route handlers are `route.ts` files that export HTTP method functions (`GET`, `POST`, etc.) to create API endpoints in the App Router.

**Technical Definition:** A `route.ts` file in the App Router exports functions named after HTTP methods. The handler receives a `Request` object and returns a `Response` (or `NextResponse`). Route handlers can be colocated with pages, support dynamic segments, and are server-only. They replace the Pages Router's `pages/api/` routes.

**Beginner-Friendly Explanation:** Route handlers are how you build APIs in the App Router. Create a `route.ts` file, export a `GET` or `POST` function, and you have an endpoint.

### Purposes

- To build API endpoints in the App Router.
- To handle `GET`, `POST`, `PUT`, `DELETE`, and other HTTP methods.
- To colocate APIs with the routes that use them.
- To integrate with authentication and middleware.
- To replace Pages Router API routes.

### Syntax Rules and Structure

```ts
// app/api/hello/route.ts
export async function GET(request: Request) {
  return Response.json({ message: 'Hello' });
}

export async function POST(request: Request) {
  const body = await request.json();
  return Response.json({ received: body }, { status: 201 });
}
```

**Syntax Rules:**

1. Create a `route.ts` file at the desired path.
2. Export functions named after HTTP methods (`GET`, `POST`, `PUT`, `DELETE`, `PATCH`).
3. The handler receives a `Request`; return a `Response`.
4. Dynamic segments are available via the `params` argument.
5. Route handlers are server-only and cannot be used in client components.

**Constraints and Limitations:**

- Cannot coexist with a `page.tsx` at the same route segment.
- Route handlers do not support client-side navigation.
- Use `NextResponse` for advanced response features (cookies, redirects).

### Annotated Code Examples

```ts
// app/api/posts/[id]/route.ts
export async function GET(request: Request, { params }: { params: Promise<{ id: string }> }) {
  const { id } = await params;
  const post = await fetchPost(id);
  if (!post) return new Response('Not Found', { status: 404 });
  return Response.json(post);
}

export async function DELETE(request: Request, { params }: { params: Promise<{ id: string }> }) {
  const { id } = await params;
  await deletePost(id);
  return new Response(null, { status: 204 });
}
```

**Expected Output:** `GET /api/posts/123` returns the post JSON. `DELETE /api/posts/123` deletes it and returns 204.

**Why this output:** The file path `app/api/posts/[id]/route.ts` maps to `/api/posts/:id`. The exported `GET` and `DELETE` functions handle the respective methods.

### Real-World Cases

- **REST APIs:** CRUD endpoints for resources.
- **Webhooks:** Receiving third-party callbacks.
- **File uploads:** Handling multipart form data.
- **Authentication endpoints:** Login, logout, token refresh.

### References

- Next.js — Route Handlers — https://nextjs.org/docs/app/api-reference/file-conventions/route
- Next.js — Route Handlers (guide) — https://nextjs.org/docs/app/building-your-application/routing/route-handlers

---

## 11. Route Middleware

### Definitions

**Core Definition:** Route middleware is a `middleware.ts` file that runs before a request is completed, allowing you to modify the response, redirect, rewrite, or set headers.

**Technical Definition:** Middleware runs in the Edge Runtime before routing. It receives a `NextRequest` and returns a `NextResponse`. It is used for authentication checks, internationalisation, redirects, rewrites, and setting security headers. Middleware is configured with a `matcher` to control which paths it runs on. It does not have access to the full Node.js runtime by default.

**Beginner-Friendly Explanation:** Middleware is code that runs before every request. It can check if the user is logged in, redirect them, or rewrite the URL.

### Purposes

- To enforce authentication before rendering protected routes.
- To redirect users based on conditions (e.g., locale).
- To rewrite URLs without changing them in the browser.
- To set security headers.
- To implement A/B testing and feature flags at the edge.

### Syntax Rules and Structure

```ts
// middleware.ts
import { NextResponse, NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  const token = request.cookies.get('token');

  if (!token && request.nextUrl.pathname.startsWith('/dashboard')) {
    return NextResponse.redirect(new URL('/login', request.url));
  }

  return NextResponse.next();
}

export const config = {
  matcher: ['/dashboard/:path*', '/settings/:path*'],
};
```

**Syntax Rules:**

1. Create `middleware.ts` at the root of the project (or `src/`).
2. Export a `middleware` function and a `config` with `matcher`.
3. Use `NextResponse.redirect`, `NextResponse.rewrite`, or `NextResponse.next`.
4. Middleware runs on the Edge Runtime.
5. Match only the paths you need; broad matchers can slow down the app.

**Constraints and Limitations:**

- No full Node.js runtime access by default (no `fs`, native modules).
- Middleware runs on every matched request; keep it fast.
- Cannot be used for database queries directly.
- Client-side guards are still needed for UX; middleware is not a complete security boundary without server checks.

### Annotated Code Examples

```ts
import { NextResponse, NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  const isLoggedIn = Boolean(request.cookies.get('session'));
  const isProtected = request.nextUrl.pathname.startsWith('/dashboard');

  if (isProtected && !isLoggedIn) {
    const loginUrl = new URL('/login', request.url);
    loginUrl.searchParams.set('redirect', request.nextUrl.pathname);
    return NextResponse.redirect(loginUrl);
  }

  return NextResponse.next();
}

export const config = {
  matcher: ['/dashboard/:path*'],
};
```

**Expected Output:** Unauthenticated users accessing `/dashboard/*` are redirected to `/login?redirect=/dashboard`.

**Why this output:** Middleware checks the session cookie before the request reaches the dashboard. It redirects with the original path for post-login return.

### Real-World Cases

- **Auth gating:** Redirecting unauthenticated users.
- **Internationalisation:** Redirecting based on `Accept-Language`.
- **A/B testing:** Rewriting to different variants.
- **Security headers:** Adding CSP, HSTS, and other headers.

### References

- Next.js — Middleware — https://nextjs.org/docs/app/api-reference/file-conventions/middleware
- Next.js — Middleware (guide) — https://nextjs.org/docs/app/building-your-application/routing/middleware

---

## 12. Route Loading UI

### Definitions

**Core Definition:** Route loading UI is the `loading.tsx` file that provides an instant loading state for a route segment while its content loads.

**Technical Definition:** `loading.tsx` creates a Suspense boundary around the route's page and its children. It is shown immediately on navigation while the server renders the page. It works with streaming and `useSuspenseQuery`. For nested routes, `loading.tsx` applies to the segment and its children. For finer control, use explicit `<Suspense>` boundaries inside the page.

**Beginner-Friendly Explanation:** `loading.tsx` is the skeleton or spinner that appears while a page loads. Create the file, and Next.js shows it automatically.

### Purposes

- To provide instant feedback during navigation.
- To prevent blank screens while pages stream.
- To create route-level skeletons.
- To leverage streaming SSR automatically.

### Syntax Rules and Structure

```tsx
// app/dashboard/loading.tsx
export default function Loading() {
  return <DashboardSkeleton />;
}
```

**Syntax Rules:**

1. Create `loading.tsx` in the route segment.
2. It wraps the segment's page and children in a Suspense boundary.
3. Use it for route-level loading states.
4. For component-level loading, use `<Suspense>` inside the page.

**Constraints and Limitations:**

- `loading.tsx` applies to the whole segment; for granular loading, use `<Suspense>`.
- It does not catch errors; use `error.tsx` for that.

### Annotated Code Examples

```tsx
// app/products/loading.tsx
export default function ProductsLoading() {
  return (
    <div className="grid">
      {Array.from({ length: 6 }).map((_, i) => (
        <ProductSkeleton key={i} />
      ))}
    </div>
  );
}
```

**Expected Output:** Navigating to `/products` shows a grid of skeletons while the product data loads.

**Why this output:** `loading.tsx` is the Suspense fallback for the `/products` segment. It appears immediately during streaming.

### Real-World Cases

- **Product listings:** Skeleton grids during data fetch.
- **Dashboards:** Skeleton widgets for each panel.
- **Content pages:** Skeleton article layouts.

### References

- Next.js — loading.js — https://nextjs.org/docs/app/api-reference/file-conventions/loading
- Next.js — Loading UI and Streaming — https://nextjs.org/docs/app/building-your-application/routing/loading-ui-and-streaming

---

## 13. Route Error UI

### Definitions

**Core Definition:** Route error UI is the `error.tsx` file that creates an error boundary for a route segment, catching errors and displaying a fallback.

**Technical Definition:** `error.tsx` must be a Client Component (`'use client'`). It receives `error` and `reset` props. It catches errors from the segment's page and children, but not from the layout at the same level. Errors bubble to the nearest `error.tsx` ancestor. `global-error.tsx` handles errors in the root layout.

**Beginner-Friendly Explanation:** `error.tsx` is the error screen for a route. If the page crashes, this file shows a friendly message instead of a broken page.

### Purposes

- To catch errors in route segments.
- To display a user-friendly error UI.
- To provide a retry mechanism with `reset`.
- To isolate failures to the smallest route segment.
- To handle root layout errors with `global-error.tsx`.

### Syntax Rules and Structure

```tsx
// app/dashboard/error.tsx
'use client';

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  return (
    <div>
      <h2>Something went wrong</h2>
      <button onClick={reset}>Try again</button>
    </div>
  );
}
```

**Syntax Rules:**

1. `error.tsx` must be a Client Component (`'use client'`).
2. It receives `error` and `reset` props.
3. `reset()` re-renders the segment; it does not refetch data.
4. Errors bubble to the nearest `error.tsx` ancestor.
5. Use `global-error.tsx` for errors in the root layout.

**Constraints and Limitations:**

- `error.tsx` does not catch errors in the layout at the same level.
- `reset()` does not refetch data; use `router.refresh()` or `revalidatePath` for that.
- Error boundaries are client-side; server errors must be handled server-side.

### Annotated Code Examples

```tsx
// app/products/error.tsx
'use client';

import { useEffect } from 'react';

export default function ProductsError({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  useEffect(() => {
    console.error(error);
  }, [error]);

  return (
    <div role="alert">
      <h2>Could not load products</h2>
      <button onClick={() => reset()}>Try again</button>
    </div>
  );
}
```

**Expected Output:** If the products page throws, the error UI appears with a retry button. The error is logged to the console.

**Why this output:** `error.tsx` is a Client Component error boundary. `reset()` attempts to re-render the segment. `useEffect` logs the error.

### Real-World Cases

- **Data-fetching routes:** Catching API failures.
- **Dashboard widgets:** Isolating widget errors.
- **Root layout errors:** `global-error.tsx` for catastrophic failures.

### References

- Next.js — error.js — https://nextjs.org/docs/app/api-reference/file-conventions/error
- Next.js — Error Handling — https://nextjs.org/docs/app/building-your-application/routing/error-handling

---

## 14. Route Layout

### Definitions

**Core Definition:** A route layout is a `layout.tsx` file that wraps a route segment and its children with shared UI, persisting across navigations.

**Technical Definition:** `layout.tsx` defines a UI shell that is shared across a segment and its children. It receives `children` and renders them. Layouts preserve state and do not re-render on navigation within the segment. The root layout is required and must contain `<html>` and `<body>`. Layouts can be nested; each segment can have its own layout.

**Beginner-Friendly Explanation:** A layout is the frame around a page. The layout stays the same while you navigate between its child pages.

### Purposes

- To share UI (headers, sidebars, footers) across routes.
- To preserve layout state across navigation.
- To avoid re-rendering shared UI.
- To compose nested layouts for complex sections.

### Syntax Rules and Structure

```tsx
// app/dashboard/layout.tsx
export default function DashboardLayout({ children }: { children: React.ReactNode }) {
  return (
    <div className="dashboard">
      <Sidebar />
      <main>{children}</main>
    </div>
  );
}
```

**Syntax Rules:**

1. Create `layout.tsx` in the segment.
2. It receives `children` and renders them.
3. Layouts persist across navigation and do not re-render.
4. The root `app/layout.tsx` must contain `<html>` and `<body>`.
5. Layouts can be nested; each segment can define its own.

**Constraints and Limitations:**

- Layouts do not re-render on navigation; use `template.tsx` if you need re-mounting.
- Layouts cannot access `params` from child segments.
- Fetching data in layouts can block rendering; prefer fetching in pages or with Suspense.

### Annotated Code Examples

```tsx
// app/dashboard/layout.tsx
export default function DashboardLayout({ children }: { children: React.ReactNode }) {
  return (
    <div className="dashboard">
      <nav aria-label="Dashboard">...</nav>
      <main>{children}</main>
    </div>
  );
}
```

**Expected Output:** Navigating between `/dashboard` and `/dashboard/settings` keeps the dashboard navigation visible; only `children` changes.

**Why this output:** The layout wraps the segment and its children. It persists across child navigation, so only the page content changes.

### Real-World Cases

- **Dashboards:** Persistent sidebar and header.
- **Settings:** Shared settings navigation.
- **Marketing sites:** Shared header and footer.
- **Multi-tenant apps:** Tenant-specific layout.

### References

- Next.js — layout.js — https://nextjs.org/docs/app/api-reference/file-conventions/layout
- Next.js — Layouts and Pages — https://nextjs.org/docs/app/getting-started/layouts-and-pages

---

## 15. Route Templates

### Definitions

**Core Definition:** A route template is a `template.tsx` file that wraps a route segment like a layout, but re-mounts on navigation, resetting state and effects.

**Technical Definition:** `template.tsx` is similar to `layout.tsx` but creates a new instance for each child route on navigation. It is useful for enter/exit animations, logging page views, and resetting state when navigating. Templates receive `children` and render them, but do not persist across navigations the way layouts do.

**Beginner-Friendly Explanation:** A template is like a layout that resets every time you navigate. It is useful when you want animations or state to restart on each page change.

### Purposes

- To reset component state on navigation.
- To implement enter/exit animations.
- To log page views on navigation.
- To run effects that should re-run on each route change.

### Syntax Rules and Structure

```tsx
// app/dashboard/template.tsx
export default function Template({ children }: { children: React.ReactNode }) {
  return <div className="page-transition">{children}</div>;
}
```

**Syntax Rules:**

1. Create `template.tsx` in the segment.
2. It receives `children` and renders them.
3. It re-mounts on each navigation within the segment.
4. Use it for animations or state resets; use `layout.tsx` for persistent UI.

**Constraints and Limitations:**

- Re-mounting resets state; do not use for shared state.
- Cannot replace the root layout.
- Enter/exit animations require CSS or a library.

### Annotated Code Examples

```tsx
'use client';

import { motion } from 'framer-motion';

export default function Template({ children }: { children: React.ReactNode }) {
  return (
    <motion.div
      initial={{ opacity: 0 }}
      animate={{ opacity: 1 }}
      exit={{ opacity: 0 }}
    >
      {children}
    </motion.div>
  );
}
```

**Expected Output:** Navigating between child routes fades in the new content. The template re-mounts on each navigation.

**Why this output:** `template.tsx` re-mounts on navigation, so the `motion.div` runs its enter animation each time.

### Real-World Cases

- **Page transitions:** Fade or slide animations between routes.
- **Analytics:** Logging page views on each navigation.
- **Form state resets:** Resetting forms when navigating away.

### References

- Next.js — template.js — https://nextjs.org/docs/app/api-reference/file-conventions/template
- Next.js — Layouts and Templates — https://nextjs.org/docs/app/getting-started/layouts-and-pages

---

## 16. Next.js Routing Best Practices

### Definitions

**Core Definition:** Next.js routing best practices are the guidelines for structuring routes, layouts, loading states, errors, and middleware to build maintainable, performant, and accessible Next.js applications.

**Technical Definition:** Best practices combine colocation (route-specific files live with the route), layout discipline (shared UI in layouts, not pages), loading strategy (`loading.tsx` for route-level, `<Suspense>` for granular), error isolation (`error.tsx` at meaningful segments), dynamic segment validation (`params` awaited and validated), middleware hygiene (narrow matchers, no heavy work), route group organisation (marketing vs. app), and performance (streaming, parallel routes, prefetching).

**Beginner-Friendly Explanation:** Good Next.js routing means organising files logically, using layouts for shared UI, providing loading and error states, keeping middleware fast, and validating dynamic params.

### Best Practices Table

| Practice | Description |
|----------|-------------|
| **Colocate route files** | Keep components, tests, and styles next to the route. |
| **Use layouts for shared UI** | Persist headers, sidebars, and footers across navigation. |
| **Use `loading.tsx` for route-level loading** | Instant feedback during streaming. |
| **Use `<Suspense>` for granular loading** | Component-level loading states. |
| **Use `error.tsx` at meaningful segments** | Isolate failures to the smallest segment. |
| **Await `params`** | Next.js 15+ `params` is a Promise; await it in Server Components. |
| **Validate dynamic params** | Parse and validate IDs before fetching. |
| **Use route groups for organisation** | Separate marketing and app layouts without URL changes. |
| **Use parallel routes for dashboards** | Independent loading and error states per slot. |
| **Use intercepting routes for modals** | Shareable URLs with modal fallbacks. |
| **Keep middleware fast** | Narrow matchers; no heavy work on every request. |
| **Use `notFound()` for missing resources** | Render the nearest `not-found.tsx`. |
| **Use `generateStaticParams` for static dynamic routes** | Pre-render known paths at build time. |
| **Handle route handlers with proper HTTP methods** | Export `GET`, `POST`, etc., and return proper status codes. |

### Annotated Code Examples

**✅ Good Practices:**

```tsx
// app/dashboard/layout.tsx — shared UI
export default function DashboardLayout({ children }: { children: React.ReactNode }) {
  return (
    <div className="dashboard">
      <Sidebar />
      <main>{children}</main>
    </div>
  );
}

// app/dashboard/loading.tsx — route-level loading
export default function Loading() {
  return <DashboardSkeleton />;
}

// app/dashboard/error.tsx — route-level error
'use client';
export default function Error({ error, reset }) {
  return <ErrorUI error={error} onRetry={reset} />;
}

// app/dashboard/page.tsx — validate params
export default async function DashboardPage() {
  const stats = await fetchStats();
  return <StatsGrid stats={stats} />;
}
```

**❌ Bad Practices:**

```tsx
// ❌ Shared UI duplicated in every page
export default function Page() {
  return (
    <div>
      <Sidebar />
      <Main />
    </div>
  );
}

// ❌ No loading state
export default async function Page() {
  const data = await fetchData(); // Blank screen while loading
  return <Content data={data} />;
}

// ❌ No error boundary
export default async function Page() {
  throw new Error('Crash');
}

// ❌ Broad middleware matcher
export const config = { matcher: '/:path*' };
```

**Expected Output:** The good practices produce shared layouts, instant loading states, isolated errors, and fast middleware. The bad practices duplicate layout code, cause blank screens, crash the app, and slow every request.

**Why this output:** Layouts prevent duplication. `loading.tsx` and `error.tsx` provide states. Narrow middleware avoids unnecessary work.

### Real-World Cases

- **All Next.js applications:** These practices apply universally.
- **E-commerce:** Shared layouts, product dynamic routes, streaming reviews.
- **SaaS dashboards:** Parallel routes, intercepting modals, middleware auth.
- **Content sites:** Static params, streaming, and error boundaries.

### References

- Next.js — Routing Fundamentals — https://nextjs.org/docs/app/building-your-application/routing
- Next.js — Layouts and Pages — https://nextjs.org/docs/app/getting-started/layouts-and-pages
- Next.js — Loading UI and Streaming — https://nextjs.org/docs/app/building-your-application/routing/loading-ui-and-streaming
- Next.js — Error Handling — https://nextjs.org/docs/app/building-your-application/routing/error-handling
- Next.js — Middleware — https://nextjs.org/docs/app/building-your-application/routing/middleware

---

## References

- Next.js — App Router — https://nextjs.org/docs/app
- Next.js — Routing Fundamentals — https://nextjs.org/docs/app/building-your-application/routing
- Next.js — Project Structure — https://nextjs.org/docs/app/getting-started/project-structure
- Next.js — Layouts and Pages — https://nextjs.org/docs/app/getting-started/layouts-and-pages
- Next.js — Pages Router — https://nextjs.org/docs/pages
- Next.js — Routing (Pages Router) — https://nextjs.org/docs/pages/building-your-application/routing
- Next.js — Dynamic Segments — https://nextjs.org/docs/app/api-reference/file-conventions/dynamic-routes
- Next.js — Route Segments — https://nextjs.org/docs/app/api-reference/file-conventions/route-segment-config
- Next.js — Route Groups — https://nextjs.org/docs/app/api-reference/file-conventions/route-groups
- Next.js — Parallel Routes — https://nextjs.org/docs/app/api-reference/file-conventions/parallel-routes
- Next.js — Parallel Routes (guide) — https://nextjs.org/docs/app/building-your-application/routing/parallel-routes
- Next.js — Intercepting Routes — https://nextjs.org/docs/app/api-reference/file-conventions/intercepting-routes
- Next.js — Intercepting Routes (guide) — https://nextjs.org/docs/app/building-your-application/routing/intercepting-routes
- Next.js — Route Handlers — https://nextjs.org/docs/app/api-reference/file-conventions/route
- Next.js — Route Handlers (guide) — https://nextjs.org/docs/app/building-your-application/routing/route-handlers
- Next.js — Middleware — https://nextjs.org/docs/app/api-reference/file-conventions/middleware
- Next.js — Middleware (guide) — https://nextjs.org/docs/app/building-your-application/routing/middleware
- Next.js — loading.js — https://nextjs.org/docs/app/api-reference/file-conventions/loading
- Next.js — Loading UI and Streaming — https://nextjs.org/docs/app/building-your-application/routing/loading-ui-and-streaming
- Next.js — error.js — https://nextjs.org/docs/app/api-reference/file-conventions/error
- Next.js — Error Handling — https://nextjs.org/docs/app/building-your-application/routing/error-handling
- Next.js — layout.js — https://nextjs.org/docs/app/api-reference/file-conventions/layout
- Next.js — template.js — https://nextjs.org/docs/app/api-reference/file-conventions/template