# TanStack Router — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** TanStack Router is a type-safe, data-first routing library for React and Solid applications that treats the URL as a first-class state manager, inferring path params, search params, loader data, and navigation contracts directly from the route tree.

**Technical Definition:** TanStack Router is a fully inferred TypeScript routing framework that generates a route tree from either the filesystem (file-based routing) or code (code-based routing). Every route is defined with `createFileRoute`, which binds the route's path string to its configuration — component, loader, `beforeLoad`, `validateSearch`, `loaderDeps`, `errorComponent`, `pendingComponent`, and more. The router uses TypeScript's declaration merging to register the route tree globally, so hooks like `Route.useParams()`, `Route.useSearch()`, and `Route.useLoaderData()` return precisely typed values without manual generics. Nested route loaders run in parallel by default, and the router ships with built-in Stale-While-Revalidate caching keyed on the parsed pathname plus `loaderDeps`. Preloading is configurable per router or per `<Link>` using `intent`, `viewport`, or `render` strategies. TanStack Router is client-first; TanStack Start extends it with SSR, server functions, and server routes.

**Beginner-Friendly Explanation:** TanStack Router is a router that makes TypeScript do the heavy lifting. When you define a route like `/posts/$postId`, TypeScript knows that navigating there requires a `postId` string, that the loader receives that `postId`, and that the component gets back exactly the type the loader returns. You cannot accidentally link to a route that does not exist, forget a required parameter, or read a search param as the wrong type — the compiler catches it. It is also built for data: loaders run before the component renders, nested loaders run in parallel instead of one after another, and the router caches loader results with configurable staleness. If React Router is a UI-mapping layer you enhance with data features, TanStack Router is a typed URL state system that happens to render UI.

### Key Characteristics

- **100% inferred TypeScript:** Route paths, params, search params, loader data, and route context are all typed from the route tree; no manual generics for consumers.
- **Route tree as contract:** Define routes once; TypeScript carries the contract through links, loaders, hooks, and boundaries.
- **File-based or code-based routing:** Both produce the same route tree; file-based requires less code and is recommended.
- **Parallel loaders:** Nested route loaders run in parallel by default, avoiding artificial waterfalls.
- **Built-in SWR caching:** `staleTime`, `gcTime`, and `preloadStaleTime` control freshness and garbage collection for loader data.
- **First-class search params:** Search params are JSON-first, schema-validated, and typed end-to-end.
- **Configurable preloading:** `intent`, `viewport`, and `render` strategies preload route code and data before navigation.
- **Client-first with optional server:** TanStack Router is the client-first application model; TanStack Start adds SSR and server functions.

### Prerequisites

- React function components, JSX, and props.
- TypeScript fundamentals (generics, type inference, declaration merging).
- Basic understanding of URLs, path params, and search params.
- Familiarity with Promises and async JavaScript.
- Knowledge of a schema validator (Zod) for search-param validation.
- Understanding of Stale-While-Revalidate caching concepts.

### Related Programming Areas

- **Type-Safe Routing** — the core value proposition.
- **Data Loading** — loaders, `beforeLoad`, and TanStack Query integration.
- **URL State Management** — search params as application state.
- **SSR and TanStack Start** — server rendering, server functions, and streaming.
- **React Router** — the primary alternative and comparison point.

---

## 1. TanStack Router

### Definitions

**Core Definition:** TanStack Router is a type-safe router for React and Solid that provides file-based or code-based route trees, typed navigation, parallel data loaders, first-class search params, built-in SWR caching, and configurable preloading.

**Technical Definition:** TanStack Router is a routing framework built with TypeScript from the ground up. It supports React and Solid, uses a route tree generated from the filesystem or defined in code, and provides lossless type inference across navigation, params, search state, loaders, links, and boundaries. The router is client-first; TanStack Start adds full-document rendering, server functions, server routes, middleware, and deployable server output while preserving the same route tree. Key APIs include `createFileRoute`, `createRootRoute`, `createRouter`, `RouterProvider`, `Link`, `useNavigate`, `Route.useParams()`, `Route.useSearch()`, `Route.useLoaderData()`, `loaderDeps`, `validateSearch`, `beforeLoad`, `errorComponent`, and `pendingComponent`.

**Beginner-Friendly Explanation:** TanStack Router is a complete routing solution that prioritises type safety and URL state. You define your routes once, and everything else — links, loaders, hooks, errors — is typed and connected.

### Purposes

- To provide fully inferred TypeScript types across the entire routing layer.
- To treat the URL as validated, typed application state (path params and search params).
- To load data before routes render, with parallel nested loaders.
- To cache loader data with configurable Stale-While-Revalidate semantics.
- To preload route code and data before the user navigates.
- To support file-based routing with automatic route tree generation.
- To offer a client-first model that scales to SSR via TanStack Start.

### Syntax Rules and Structure

**General Syntax (File-Based Route):**

```tsx
import { createFileRoute } from '@tanstack/react-router';

export const Route = createFileRoute('/posts/$postId')({
  loader: async ({ params }) => fetchPost(params.postId),
  component: PostComponent,
});
```

**General Syntax (Router Creation):**

```tsx
import { createRouter } from '@tanstack/react-router';
import { routeTree } from './routeTree.gen';

const router = createRouter({ routeTree });
```

**Component Breakdown:**

| Element | Description |
|---------|-------------|
| `createFileRoute('/path')` | Binds the route's path string to its configuration; the path is generated by the bundler plugin or CLI. |
| `Route` | The exported route object used by `Route.useParams()`, `Route.useSearch()`, and `Route.useLoaderData()`. |
| `loader` | Async function that fetches data before the component renders. |
| `component` | The component rendered when the route matches. |
| `routeTree` | The generated tree passed to `createRouter`. |

**Syntax Rules:**

1. Every non-root route uses `createFileRoute` with its exact path string; the path is managed by the TanStack Router bundler plugin or Router CLI.
2. The root route uses `createRootRoute()` or `createRootRouteWithContext<T>()` and has no path; it is always matched.
3. Export the route as `Route` so the typed hooks are available.
4. Pass the generated `routeTree` to `createRouter`.
5. Register the router type via declaration merging for project-wide type safety.

**Constraints and Limitations:**

- Deep TypeScript inference can increase compile times as the route tree and schemas grow.
- Loaders run on the client by default; do not use Node-only APIs (`fs`, database clients) directly in loaders unless using TanStack Start server functions.
- File-based routing requires the bundler plugin or CLI to generate `routeTree.gen.ts`.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: File-Based Route with Loader and Typed Params

```tsx
// src/routes/posts.$postId.tsx
import { createFileRoute } from '@tanstack/react-router';

export const Route = createFileRoute('/posts/$postId')({
  loader: async ({ params }) => {
    // params.postId is inferred as string
    const post = await fetchPost(params.postId);
    return { post };
  },
  component: PostComponent,
});

function PostComponent() {
  // post is inferred from the loader's return type
  const { post } = Route.useLoaderData();
  return <h1>{post.title}</h1>;
}
```

**Expected Output:** Navigating to `/posts/123` renders the post title. TypeScript errors if `postId` is missing from a `<Link>` or `navigate` call.

**Why this output:** `createFileRoute('/posts/$postId')` binds the path pattern. The loader receives typed `params` and returns `{ post }`. `Route.useLoaderData()` infers that return type, so `post.title` is type-safe.

#### Example 2: Code-Based Route Tree

```tsx
import { createRootRoute, createRoute, createRouter } from '@tanstack/react-router';

const rootRoute = createRootRoute();

const postsRoute = createRoute({
  getParentRoute: () => rootRoute,
  path: '/posts',
});

const postRoute = createRoute({
  getParentRoute: () => postsRoute,
  path: '$postId',
  loader: ({ params }) => fetchPost(params.postId),
  component: PostComponent,
});

const routeTree = rootRoute.addChildren([
  postsRoute.addChildren([postRoute]),
]);

const router = createRouter({ routeTree });
```

**Expected Output:** The same typed route tree is built in code; navigation to `/posts/123` is type-checked.

**Why this output:** `createRoute` defines each route with `getParentRoute`, `path`, and configuration. `addChildren` builds the tree. The router receives the tree and provides the same typed APIs.

### Real-World Cases

- **Type-safe SPAs:** Dashboards and SaaS products where URL state drives the UI.
- **Search-heavy applications:** E-commerce and data explorers with validated, shareable filters.
- **Multi-tenant apps:** Typed route context carrying auth and tenant information.
- **Full-stack applications:** TanStack Start for SSR, server functions, and streaming.

### References

- TanStack Router — Overview — https://tanstack.com/router/latest/docs/overview
- TanStack Router — Routing Concepts — https://tanstack.com/router/latest/docs/routing/routing-concepts
- TanStack Router — Official Site — https://tanstack.com/router/latest
- Vercel — TanStack Router vs React Router — https://vercel.com/i/tanstack-router-vs-react-router

---

## 2. Type-Safe Routing

### Definitions

**Core Definition:** Type-safe routing is the property that every navigation, route path, path parameter, search parameter, and loader data shape is checked by TypeScript at compile time, so invalid routes or missing parameters fail in the editor rather than at runtime.

**Technical Definition:** TanStack Router achieves type safety by generating a route tree from file-based or code-based definitions, then using TypeScript declaration merging to register that tree globally. `createFileRoute` binds the path string to the route configuration, allowing TypeScript to infer valid `to` paths for `<Link>` and `navigate`, required `params` for dynamic segments, the shape of `validateSearch` output for search params, and the return type of `loader` for `Route.useLoaderData()`. Parsed params preserve their types through `params.parse`, so a `postId` parsed to a number remains a number in the component.

**Beginner-Friendly Explanation:** Type-safe routing means the compiler catches routing mistakes. If you try to link to `/postz` instead of `/posts`, or forget the `postId`, or treat a search param as a number when it is a string, TypeScript tells you immediately.

### Purposes

- To eliminate broken links and invalid navigation at compile time.
- To enforce required path parameters before navigation.
- To validate and type search params end-to-end.
- To infer loader data types without manual generics.
- To make refactors safe: renaming a route or changing a param surfaces every affected call site.

### Syntax Rules and Structure

**Type-Safe Navigation:**

```tsx
const navigate = useNavigate();

// ✅ Valid
navigate({ to: '/posts/$postId', params: { postId: '123' } });

// ❌ TypeScript error: missing postId
navigate({ to: '/posts/$postId' });

// ❌ TypeScript error: route does not exist
navigate({ to: '/invalid-route' });
```

**Typed Route Hooks:**

```tsx
const { postId } = Route.useParams(); // string
const { page } = Route.useSearch();   // number (from schema)
const { post } = Route.useLoaderData(); // inferred from loader
```

**Syntax Rules:**

1. Register the router type via declaration merging in a `declare module` block so `Link`, `useNavigate`, and hooks are typed project-wide.
2. Use `createFileRoute` or `createRoute` to define routes; the path is the type source.
3. Dynamic segments (`$postId`) are typed as `string` by default; use `params.parse` to narrow to other types.
4. Search params are typed from `validateSearch`; the schema output becomes the search state type.
5. Loader data is typed from the loader's return value.

**Constraints and Limitations:**

- Deep inference can increase compile times.
- Type safety depends on the generated `routeTree.gen.ts` being up to date; use the bundler plugin or CLI watch mode.
- `as` casts or `any` in loader returns can weaken inference.

### Annotated Code Examples

```tsx
// src/routes/posts.$postId.tsx
import { createFileRoute } from '@tanstack/react-router';

export const Route = createFileRoute('/posts/$postId')({
  params: {
    parse: (params) => ({ postId: Number(params.postId) }),
    stringify: (params) => ({ postId: String(params.postId) }),
  },
  loader: ({ params }) => fetchPost(params.postId), // postId is number
  component: PostComponent,
});

function PostComponent() {
  const { postId } = Route.useParams(); // number
  const post = Route.useLoaderData();   // inferred from loader
  return <h1>{post.title} (ID: {postId})</h1>;
}
```

**Expected Output:** `postId` is a `number` in both the loader and the component. Navigating with a non-numeric string is still allowed at the URL level but the parser converts it; invalid values can be caught with validation.

**Why this output:** `params.parse` transforms `postId` from string to number and preserves that type in the loader and component. The loader's return type flows to `useLoaderData`.

### Real-World Cases

- **Large codebases:** Refactoring route names without breaking links.
- **Search-driven UIs:** Typed filters, sorting, and pagination.
- **Multi-tenant apps:** Typed route context and params.

### References

- TanStack Router — Type Safety — https://mintlify.wiki/TanStack/router/router/concepts/type-safety
- TanStack Router — Routing Concepts — https://tanstack.com/router/latest/docs/routing/routing-concepts
- TanStack Router — Overview — https://tanstack.com/router/latest/docs/overview
- Vercel — TanStack Router vs React Router — https://vercel.com/i/tanstack-router-vs-react-router

---

## 3. Route Trees

### Definitions

**Core Definition:** A route tree is the hierarchical structure of all routes in an application, defining parent-child relationships, paths, layouts, and configuration for each route.

**Technical Definition:** TanStack Router builds a route tree from either the filesystem (file-based routing) or code (`createRoute` and `addChildren`). The tree includes the root route, layout routes (with or without paths), index routes, dynamic segments (`$param`), optional parameters (`{-$param}`), splat routes (`$`), and non-nested routes (`_` suffix). Pathless layout routes (`_` prefix) wrap children without adding a URL segment. Pathless route group directories (`()` ) organise files without affecting the tree. The generated `routeTree.gen.ts` is the single source of truth for type inference.

**Beginner-Friendly Explanation:** The route tree is the map of your application's pages and how they nest. File-based routing draws the map from your folder structure; code-based routing draws it explicitly.

### Purposes

- To define the complete navigation structure of the application.
- To enable nested layouts and shared UI through parent-child relationships.
- To generate types for paths, params, and search state.
- To organise routes into logical groups.
- To control matching precedence and route ranking.

### Syntax Rules and Structure

**File-Based Tree Examples:**

| File | Path | Type |
|------|------|------|
| `index.tsx` | `/` | Index |
| `about.tsx` | `/about` | Basic |
| `posts.$postId.tsx` | `/posts/$postId` | Dynamic |
| `posts.{-$category}.tsx` | `/posts/{-$category}` | Optional |
| `files.$.tsx` | `/files/$` | Splat |
| `app.tsx` + `app.dashboard.tsx` | `/app`, `/app/dashboard` | Layout |
| `_pathlessLayout.tsx` | (no URL) | Pathless layout |
| `(app)/dashboard.tsx` | `/dashboard` | Route group |

**Syntax Rules:**

1. The root route is created with `createRootRoute()` and has no path; it always renders.
2. Layout routes wrap children and render `<Outlet />`; they can have a path or be pathless (`_` prefix).
3. Index routes target the parent path exactly (`/posts/`).
4. Dynamic segments use `$name`; optional segments use `{-$name}`; splats use `$`.
5. Non-nested routes use a `_` suffix on the parent segment to un-nest from the parent tree.
6. Files and folders prefixed with `-` are excluded from route generation.
7. Route group directories `()` organise files without affecting the tree.

**Constraints and Limitations:**

- File-based routing requires the bundler plugin or CLI to generate the tree.
- Pathless route IDs must be unique for TypeScript inference.
- Very deep trees can increase compile time.

### Annotated Code Examples

**File-Based Layout and Nested Routes:**

```tsx
// src/routes/app.tsx
import { Outlet, createFileRoute } from '@tanstack/react-router';

export const Route = createFileRoute('/app')({
  component: AppLayout,
});

function AppLayout() {
  return (
    <div>
      <h1>App Layout</h1>
      <Outlet />
    </div>
  );
}
```

```tsx
// src/routes/app.dashboard.tsx
import { createFileRoute } from '@tanstack/react-router';

export const Route = createFileRoute('/app/dashboard')({
  component: Dashboard,
});

function Dashboard() {
  return <h2>Dashboard</h2>;
}
```

**Expected Output:** Navigating to `/app/dashboard` renders the app layout with the dashboard inside the outlet. The layout persists across child navigation.

**Why this output:** `app.tsx` is a layout route; `app.dashboard.tsx` is its child. The router composes them, rendering `<AppLayout><Dashboard /></AppLayout>`.

### Real-World Cases

- **Dashboards:** Layout routes for persistent sidebars and headers.
- **Settings:** Nested routes for profile, billing, and notifications.
- **Content sites:** Pathless layouts for shared article chrome.
- **Multi-section apps:** Route groups for auth and app sections.

### References

- TanStack Router — Routing Concepts — https://tanstack.com/router/latest/docs/routing/routing-concepts
- TanStack Router — Code-Based Routing — https://tanstack.com/router/latest/docs/routing/code-based-routing
- TanStack Router — File-Based Routing — https://tanstack.com/router/latest/docs/routing/file-based-routing

---

## 4. Route Loaders

### Definitions

**Core Definition:** Route loaders are async functions that fetch a route's data before its component renders, ensuring data is ready at first render.

**Technical Definition:** A `loader` receives `{ params, deps, context, abortController, cause, preload, location, parentMatchPromise, route }` and returns data consumed via `Route.useLoaderData()`. Nested route loaders run in parallel by default, not sequentially. `loaderDeps` declares which search params affect the loader's cache key, so the loader re-runs only when those deps change. `pendingComponent` provides the loading UI, with `pendingMs` and `pendingMinMs` controlling when it appears and how long it stays to avoid flashing.

**Beginner-Friendly Explanation:** A loader fetches the data a page needs before the page appears. Because nested loaders run at the same time, you avoid waiting for parent data before starting child data.

### Purposes

- To fetch data before rendering, eliminating loading spinners inside components.
- To run nested loaders in parallel and avoid waterfalls.
- To key loader caching on `loaderDeps` for search-param-driven data.
- To provide pending UI with `pendingComponent` and timing controls.
- To integrate with TanStack Query via `ensureQueryData` for shared caching.

### Syntax Rules and Structure

**Basic Loader:**

```tsx
export const Route = createFileRoute('/posts')({
  loader: () => fetchPosts(),
  component: PostsComponent,
});

function PostsComponent() {
  const posts = Route.useLoaderData();
  return <PostList posts={posts} />;
}
```

**Loader with Params and Deps:**

```tsx
export const Route = createFileRoute('/posts/$postId')({
  validateSearch: z.object({ tab: z.enum(['comments', 'related']).default('comments') }),
  loaderDeps: ({ search: { tab } }) => ({ tab }),
  loader: async ({ params, deps }) => {
    const post = await fetchPost(params.postId);
    const tabData = await fetchTabData(params.postId, deps.tab);
    return { post, tabData };
  },
});
```

**Syntax Rules:**

1. Define `loader` on the route; return the data the component needs.
2. Use `Route.useLoaderData()` to access the loader's return value with full type inference.
3. Use `loaderDeps` to declare which search params affect the loader; the loader only re-runs when deps change.
4. Use `pendingComponent` with `pendingMs` and `pendingMinMs` for loading states.
5. Use `Promise.all` for independent fetches within a loader; nested loaders already run in parallel.
6. Use `queryClient.ensureQueryData()` (not `prefetchQuery`) in loaders when integrating TanStack Query, so errors propagate to boundaries.
7. Use `abortController.signal` in the loader for cancellable fetches.

**Constraints and Limitations:**

- Loaders run on the client by default; do not use Node-only APIs directly.
- `loaderDeps` must include every search param the loader reads; otherwise the cache key is incomplete.
- Pending UI flashes if `pendingMs` and `pendingMinMs` are not tuned.

### Annotated Code Examples

**Parallel Nested Loaders:**

```tsx
// routes/posts.tsx
export const Route = createFileRoute('/posts')({
  loader: async () => {
    const categories = await fetchCategories();
    return { categories };
  },
});

// routes/posts.$postId.tsx
export const Route = createFileRoute('/posts/$postId')({
  loader: async ({ params }) => {
    const post = await fetchPost(params.postId);
    const comments = await fetchComments(params.postId);
    return { post, comments };
  },
});
```

**Expected Output:** Navigating to `/posts/123` starts the parent and child loaders at the same time. Total time is the maximum of the two, not the sum.

**Why this output:** TanStack Router runs nested loaders in parallel by default. The parent loader fetches categories while the child loader fetches post and comments simultaneously.

**TanStack Query Integration:**

```tsx
const postQueryOptions = (postId: string) =>
  queryOptions({
    queryKey: ['posts', postId],
    queryFn: () => fetchPost(postId),
    staleTime: 5 * 60 * 1000,
  });

export const Route = createFileRoute('/posts/$postId')({
  loader: async ({ params, context: { queryClient } }) => {
    await queryClient.ensureQueryData(postQueryOptions(params.postId));
  },
  component: PostPage,
});

function PostPage() {
  const { postId } = Route.useParams();
  const { data: post } = useSuspenseQuery(postQueryOptions(postId));
  return <PostContent post={post} />;
}
```

**Expected Output:** The loader ensures the post is cached before render. The component reads the same cache entry via `useSuspenseQuery`, so there is no duplicate fetch and errors propagate to the route's error boundary.

**Why this output:** `ensureQueryData` awaits missing data and throws on error, unlike `prefetchQuery`, which swallows errors.

### Real-World Cases

- **Data-heavy dashboards:** Parallel loaders for widgets and stats.
- **E-commerce:** Product and recommendation data loaded before render.
- **Content sites:** Article and comment data with deferred comments for slow sections.
- **Search pages:** `loaderDeps` tied to pagination and filters.

### References

- TanStack Router — Router Data Loading — https://github.com/compozy/compozy/blob/5f48b457f7e4eaabaccfe08d83952be3dd743d18/.agents/skills/tanstack/references/router-data-loading.md
- TanStack Router — Data Loading — https://tanstack.com/intent/registry/%40tanstack%2Frouter-core/data-loading
- TanStack Router — load-parallel — https://github.com/supasheet/supasheet/blob/cdfcda6545ddbe3f536602ce0ef06842d460867c/.agents/skills/tanstack-router-best-practices/rules/load-parallel.md

---

## 5. Route Actions

### Definitions

**Core Definition:** Route actions are the mechanisms for handling data mutations — creating, updating, or deleting data — in response to user interactions, with the router coordinating cache invalidation.

**Technical Definition:** TanStack Router does not prescribe a single mutation API. In practice, mutations are handled with TanStack Query's `useMutation` or TanStack Actions (a lightweight action library). After a mutation succeeds, the router's loader data must be invalidated with `router.invalidate({ sync: true })` so that loaders re-run and the UI reflects the new state. Treat the mutation and invalidation as a single workflow: persist first, then invalidate, and ensure the loader reads from the same authoritative store.

**Beginner-Friendly Explanation:** Actions handle changes to data — submitting a form, deleting a row, updating a profile. After the change succeeds, you tell the router to refresh its data so the page shows the new state.

### Purposes

- To handle form submissions and data mutations.
- To keep loader data in sync after mutations via invalidation.
- To coordinate mutations with TanStack Query or TanStack Actions.
- To verify that mutations persist before invalidating loaders.
- To avoid stale loader data after create, update, or delete operations.

### Syntax Rules and Structure

**Mutation with Invalidation:**

```tsx
function AddPostButton() {
  const router = useRouter();
  const mutation = useMutation({
    mutationFn: (title: string) => createPost({ title }),
    onSuccess: async () => {
      await router.invalidate({ sync: true });
    },
  });

  return (
    <button onClick={() => mutation.mutate('New post')}>
      Add Post
    </button>
  );
}
```

**Syntax Rules:**

1. Use `useMutation` (TanStack Query) or TanStack Actions for mutations.
2. After a successful mutation, call `router.invalidate({ sync: true })` to re-run loaders and refresh the UI.
3. Treat mutation and invalidation as one workflow; persist before invalidating.
4. Verify create, update, and delete through the rendered route, including a fresh reload.
5. Use the router's error boundaries to catch mutation failures.

**Constraints and Limitations:**

- `reset()` in an error component only resets the UI; it does not re-run loaders. Use `router.invalidate()` for loader errors.
- If the loader reads from a different store than the mutation writes to, invalidation will not reflect the change.

### Annotated Code Examples

```tsx
const createPostMutation = useMutation({
  mutationFn: (input: { title: string }) => createPost(input),
  onSuccess: async () => {
    await router.invalidate({ sync: true });
  },
});

function NewPostForm() {
  const [title, setTitle] = useState('');
  const router = useRouter();

  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    createPostMutation.mutate(
      { title },
      {
        onSuccess: async () => {
          await router.invalidate({ sync: true });
          setTitle('');
        },
      }
    );
  }

  return (
    <form onSubmit={handleSubmit}>
      <input value={title} onChange={(e) => setTitle(e.target.value)} />
      <button disabled={createPostMutation.isPending}>
        {createPostMutation.isPending ? 'Creating…' : 'Create'}
      </button>
    </form>
  );
}
```

**Expected Output:** Submitting the form creates the post, invalidates the router, and re-runs the relevant loaders so the new post appears.

**Why this output:** The mutation persists the new post; `router.invalidate({ sync: true })` forces loaders to re-run and fetch the updated data from the authoritative store.

### Real-World Cases

- **CRUD interfaces:** Creating, editing, and deleting records.
- **Form submissions:** Profile updates, settings changes, checkout.
- **Optimistic UI:** Updating the UI before the server confirms, then invalidating on success.
- **Admin panels:** Actions that affect lists and counts.

### References

- TanStack Router — Data Mutations — https://github.com/TanStack/router/blob/92655370a14761aaf59cacb86b1f7675b035d354/docs/guide/data-mutations.md
- TanStack Router — Data Loading (invalidation) — https://tanstack.com/intent/registry/%40tanstack%2Frouter-core/data-loading
- TanStack Query — useMutation — https://tanstack.com/query

---

## 6. Route Search Params

### Definitions

**Core Definition:** Route search params are the key-value pairs in the URL query string (after `?`) that TanStack Router treats as typed, validated, JSON-first application state.

**Technical Definition:** TanStack Router treats search params as first-class state. They are declared with `validateSearch`, which can use a Zod schema (via `zodValidator` and the `@tanstack/zod-adapter` package) or a plain function. The validated output is typed and accessed with `Route.useSearch()`. Search params support JSON values (objects, arrays, numbers, booleans), not just strings. `loaderDeps` selects which search params affect the loader's cache key. Search params are inherited by child routes and are bookmarkable, shareable, and safe to consume in loaders and components.

**Beginner-Friendly Explanation:** Search params are the filters, page numbers, and tabs in the URL. TanStack Router treats them as real data — typed, validated, and able to hold objects and arrays — instead of raw strings you parse yourself.

### Purposes

- To model view state (filters, pagination, sorting, tabs) in the URL.
- To validate search params at the route boundary and reject invalid values.
- To type search state so components and loaders read the correct types.
- To make filtered and paginated views bookmarkable and shareable.
- To connect search params to loader caching via `loaderDeps`.

### Syntax Rules and Structure

**Zod-Validated Search Params:**

```tsx
import { createFileRoute } from '@tanstack/react-router';
import { zodValidator, fallback } from '@tanstack/zod-adapter';
import { z } from 'zod';

const searchSchema = z.object({
  page: fallback(z.number().int().min(1), 1),
  filter: z.string().optional(),
  sortBy: z.enum(['date', 'title', 'author']).default('date'),
});

export const Route = createFileRoute('/posts')({
  validateSearch: zodValidator(searchSchema),
  loaderDeps: ({ search: { page, filter, sortBy } }) => ({ page, filter, sortBy }),
  loader: ({ deps }) => fetchPosts(deps),
});
```

**Syntax Rules:**

1. Define `validateSearch` on the route with a schema or validator function.
2. Use `zodValidator(schema)` from `@tanstack/zod-adapter` for Zod schemas; Zod v3 requires the adapter wrapper.
3. Use `fallback()` from the adapter instead of Zod's `.catch()` when using `zodValidator` and Zod v3.
4. Access typed search state with `Route.useSearch()`.
5. Declare search params that affect loader data in `loaderDeps`; the loader only re-runs when those deps change.
6. Search params are inherited by child routes automatically.

**Constraints and Limitations:**

- Without `loaderDeps`, the loader will not re-run when search params change.
- Using `.catch()` with `zodValidator` and Zod v3 can destroy type safety; use `fallback()` instead.
- Search params must be JSON-serialisable for the JSON-first state model.

### Annotated Code Examples

```tsx
// src/routes/products.tsx
import { createFileRoute } from '@tanstack/react-router';
import { zodValidator, fallback } from '@tanstack/zod-adapter';
import { z } from 'zod';

const productSearchSchema = z.object({
  page: fallback(z.number().int().min(1), 1),
  sort: z.enum(['price', 'name', 'newest']).default('newest'),
  category: z.string().optional(),
});

export const Route = createFileRoute('/products')({
  validateSearch: zodValidator(productSearchSchema),
  loaderDeps: ({ search: { page, sort, category } }) => ({ page, sort, category }),
  loader: ({ deps }) => fetchProducts(deps),
  component: ProductList,
});

function ProductList() {
  const { page, sort, category } = Route.useSearch();
  const products = Route.useLoaderData();
  return <ProductGrid products={products} page={page} sort={sort} />;
}
```

**Expected Output:** `/products?page=2&sort=price` parses `page` as a number and `sort` as an enum. The loader receives `{ page: 2, sort: 'price', category: undefined }` and fetches the matching page.

**Why this output:** `validateSearch` parses and types the search params. `loaderDeps` declares which params affect the loader, so changing `page` re-runs the loader with the new deps.

### Real-World Cases

- **E-commerce filters:** Category, price range, sort, and pagination.
- **Search results:** Query, page, and sort state.
- **Dashboards:** Date range, tab, and view mode.
- **Documentation:** Version and section selection.

### References

- TanStack Router — Search Params — https://tanstack.com/router/latest/docs/guide/search-params
- TanStack Router — Search Param Validation — https://tanstack.com/intent/registry/%40tanstack%2Frouter-core/search-params
- TanStack Router — How to Set Up Basic Search Parameters — https://tanstack.com/router/latest/docs/guide/search-params
- Vercel — TanStack Router vs React Router (search params example) — https://vercel.com/i/tanstack-router-vs-react-router

---

## 7. Route Validation

### Definitions

**Core Definition:** Route validation is the set of checks that run when a route matches, verifying path params, search params, and route context before the loader and component execute.

**Technical Definition:** TanStack Router validates routes at multiple stages. `params.parse` transforms and validates dynamic path segments. `validateSearch` validates the query string and produces the typed search state. `beforeLoad` runs before loaders and can validate context, enforce authentication, or throw redirects. The loading lifecycle is: route matching → `params.parse` → `validateSearch` → `beforeLoad` → loaders (parallel) → components. Throwing in `beforeLoad` prevents all child routes from loading.

**Beginner-Friendly Explanation:** Validation makes sure the URL and context make sense before the page tries to load. If the `postId` is not a number, or the search param is invalid, or the user is not logged in, the router can stop before running the loader.

### Purposes

- To validate and transform path params before use.
- To validate search params and reject invalid values.
- To enforce authentication and authorisation in `beforeLoad`.
- To provide shared context to child routes after validation.
- To prevent loaders from running with invalid inputs.

### Syntax Rules and Structure

**Path Param Validation:**

```tsx
export const Route = createFileRoute('/posts/$postId')({
  params: {
    parse: (params) => {
      const postId = Number(params.postId);
      if (!Number.isInteger(postId)) throw new Error('Invalid post ID');
      return { postId };
    },
    stringify: (params) => ({ postId: String(params.postId) }),
  },
  loader: ({ params }) => fetchPost(params.postId), // postId is number
});
```

**`beforeLoad` Guard:**

```tsx
export const Route = createFileRoute('/admin')({
  beforeLoad: async ({ context }) => {
    if (!context.auth.user) throw redirect({ to: '/login' });
  },
  component: AdminLayout,
});
```

**Syntax Rules:**

1. Use `params.parse` and `params.stringify` to transform path params and preserve types.
2. Use `validateSearch` for search params; it runs before `beforeLoad` and loaders.
3. Use `beforeLoad` for authentication, context setup, and redirects; it runs before loaders.
4. Throwing in `beforeLoad` prevents children from loading.
5. Return context from `beforeLoad` to make it available to child loaders and components.

**Constraints and Limitations:**

- Validation errors should throw or redirect; silent failures hide problems.
- `beforeLoad` runs serially (parent before child); avoid long-running work there.
- Server-side authorisation must still be enforced; `beforeLoad` is a client-side guard unless running under TanStack Start.

### Annotated Code Examples

```tsx
export const Route = createFileRoute('/dashboard')({
  beforeLoad: async ({ context }) => {
    if (!context.auth.user) {
      throw redirect({ to: '/login', search: { redirect: '/dashboard' } });
    }
    return { user: context.auth.user };
  },
  loader: ({ context: { user } }) => fetchDashboard(user.id),
  component: Dashboard,
});
```

**Expected Output:** Unauthenticated users are redirected to `/login` with a redirect back to `/dashboard`. Authenticated users proceed to the loader with the validated `user` context.

**Why this output:** `beforeLoad` validates the session and throws a redirect when missing. It returns the `user` context that the loader consumes.

### Real-World Cases

- **Authentication:** Requiring login for protected routes.
- **Role-based access:** Restricting admin routes to admins.
- **Param validation:** Ensuring `postId` is numeric.
- **Search validation:** Rejecting invalid sort or filter values.

### References

- TanStack Router — Data Loading Lifecycle — https://tanstack.com/intent/registry/%40tanstack%2Frouter-core/data-loading
- TanStack Router — Type Safety (params.parse) — https://mintlify.wiki/TanStack/router/router/concepts/type-safety
- TanStack Router — load-parallel (beforeLoad) — https://github.com/supasheet/supasheet/blob/cdfcda6545ddbe3f536602ce0ef06842d460867c/.agents/skills/tanstack-router-best-practices/rules/load-parallel.md

---

## 8. Route Caching

### Definitions

**Core Definition:** Route caching is the built-in Stale-While-Revalidate mechanism that stores loader results and reuses them across navigations according to configurable freshness and garbage-collection settings.

**Technical Definition:** TanStack Router caches loader data keyed on the route's parsed pathname plus `loaderDeps`. `staleTime` controls how long data is considered fresh; within this window, cached data is used without refetching. After `staleTime`, data is still served immediately but refetched in the background (SWR). `gcTime` controls how long an unused cache entry remains before garbage collection. `preloadStaleTime` (default 30 seconds) prevents preloaded data from being refetched immediately. Defaults: `staleTime: 0`, `preloadStaleTime: 30_000`, `gcTime: 30 * 60 * 1000`. The router uses structural sharing and identity checks to avoid redundant reloads.

**Beginner-Friendly Explanation:** Route caching remembers the data from previous visits. If you go back to a page you just visited, it shows the cached data instantly and checks for updates in the background. You can configure how long data is considered fresh and how long unused data stays in memory.

### Purposes

- To provide instant navigation to previously visited routes.
- To avoid unnecessary refetches for data that has not changed.
- To control data freshness with `staleTime` and `gcTime`.
- To prevent preloaded data from being immediately refetched.
- To coordinate router caching with TanStack Query when both are used.

### Syntax Rules and Structure

**Cache Configuration:**

```tsx
export const Route = createFileRoute('/posts')({
  loader: () => fetchPosts(),
  staleTime: 10_000,          // Fresh for 10 seconds
  gcTime: 5 * 60 * 1000,      // Garbage collect after 5 minutes
  preloadStaleTime: 30_000,   // Preloaded data stays fresh for 30 seconds
});
```

**Global Defaults:**

```tsx
const router = createRouter({
  routeTree,
  defaultStaleTime: Infinity, // Disable background refetch globally
});
```

**Syntax Rules:**

1. Set `staleTime` per route to control how long data is fresh; default is `0` (always stale, refetch in background).
2. Set `gcTime` to control how long unused data stays in memory; default is 30 minutes.
3. Set `preloadStaleTime` to avoid refetching preloaded data immediately; default is 30 seconds.
4. Set `staleTime: Infinity` to disable SWR refetching entirely.
5. When using TanStack Query, let Query's `staleTime`/`gcTime` control caching and keep the router's `staleTime` at `0` to defer to Query.

**Constraints and Limitations:**

- Default `staleTime: 0` means data refetches in the background on every re-match; set a higher value for static or expensive data.
- The cache key is pathname + `loaderDeps`; if a loader reads a search param not declared in `loaderDeps`, the cache key is incomplete.
- Structural sharing prevents redundant reloads but does not replace schema validation.

### Annotated Code Examples

```tsx
export const Route = createFileRoute('/dashboard')({
  loader: () => fetchDashboardStats(),
  staleTime: 30_000,           // Fresh for 30 seconds
  gcTime: 10 * 60 * 1000,      // Keep unused data for 10 minutes
  pendingMs: 500,              // Show pending UI after 500ms
  pendingMinMs: 300,           // Keep pending UI for at least 300ms
  pendingComponent: () => <DashboardSkeleton />,
});
```

**Expected Output:** Returning to `/dashboard` within 30 seconds shows cached stats without a request. After 30 seconds, cached stats appear immediately and a background refetch updates them.

**Why this output:** `staleTime: 30_000` keeps data fresh for 30 seconds. After that, SWR serves stale data and refetches in the background. `gcTime` keeps the cache entry for 10 minutes.

### Real-World Cases

- **Dashboards:** Cached stats with short staleness for near-real-time data.
- **E-commerce:** Product listings cached for longer, with background refresh.
- **Content sites:** Articles cached aggressively, with `staleTime` tuned to publishing frequency.
- **TanStack Query integration:** Query owns the cache; router defers with `staleTime: 0`.

### References

- TanStack Router — Caching — https://mintlify.wiki/TanStack/router/router/guide/caching
- TanStack Router — Data Loading (SWR caching) — https://tanstack.com/intent/registry/%40tanstack%2Frouter-core/data-loading
- TanStack Router — Data Loading (caching defaults) — https://github.com/compozy/compozy/blob/5f48b457f7e4eaabaccfe08d83952be3dd743d18/.agents/skills/tanstack/references/router-data-loading.md

---

## 9. Route Prefetching

### Definitions

**Core Definition:** Route prefetching is the practice of loading a route's code and data before the user navigates to it, so the navigation feels instant.

**Technical Definition:** TanStack Router supports automatic preloading at the router level (`defaultPreload`) and per-link (`preload` prop). Strategies are `'intent'` (preload on hover or `touchstart`), `'viewport'` (preload when the link enters the viewport via Intersection Observer), and `'render'` (preload when the link renders). `defaultPreloadDelay` controls the delay before preloading. Preloaded data is cached with `preloadStaleTime` to avoid immediate refetching. By the time navigation commits, the next screen's code and data can already be waiting.

**Beginner-Friendly Explanation:** Prefetching starts loading the next page before the user clicks. When they hover over a link or when the link scrolls into view, the router fetches the code and data in the background.

### Purposes

- To make navigations feel instant by preloading code and data.
- To use idle time and user intent as signals for preloading.
- To avoid loading waterfalls on navigation.
- To improve perceived performance without blocking the current page.
- To let preloaded data stay fresh via `preloadStaleTime`.

### Syntax Rules and Structure

**Global Preloading:**

```tsx
const router = createRouter({
  routeTree,
  defaultPreload: 'intent',      // Preload on hover/touchstart
  defaultPreloadDelay: 50,       // Optional delay in ms
});
```

**Per-Link Preloading:**

```tsx
<Link to="/dashboard" preload="viewport">
  Dashboard
</Link>
```

**Syntax Rules:**

1. Set `defaultPreload` to `'intent'`, `'viewport'`, or `'render'` on the router.
2. Use the `preload` prop on individual `<Link>` components to override the global strategy.
3. Use `defaultPreloadDelay` to avoid preloading on accidental hovers.
4. Preloaded data respects `preloadStaleTime` (default 30 seconds).
5. `'viewport'` uses the Intersection Observer API to detect when links enter the viewport.

**Constraints and Limitations:**

- Preloading too aggressively can waste bandwidth and server resources.
- `'viewport'` preloading may fetch routes the user never visits if many links are visible.
- Preloaded data is cached but can become stale; `preloadStaleTime` controls refetch behaviour.

### Annotated Code Examples

```tsx
const router = createRouter({
  routeTree,
  defaultPreload: 'intent',
});

function Navigation() {
  return (
    <nav>
      <Link to="/dashboard" preload="viewport">
        Dashboard
      </Link>
      <Link to="/reports">Reports</Link> {/* Uses default 'intent' */}
    </nav>
  );
}
```

**Expected Output:** Hovering over “Reports” preloads its code and data. Scrolling “Dashboard” into view preloads it via the viewport strategy.

**Why this output:** The router's `defaultPreload: 'intent'` applies to all links; the per-link `preload="viewport"` overrides it for the dashboard link.

### Real-World Cases

- **Dashboards:** Preloading the most-used routes on hover.
- **E-commerce:** Preloading product pages from listing cards.
- **Content sites:** Preloading articles when links enter the viewport.
- **Multi-step flows:** Preloading the next step based on intent.

### References

- TanStack Router — Preloading — https://tanstack.com/router/latest/docs/guide/preloading
- TanStack Router — RouterOptions (preload) — https://tanstack.com/router/latest/docs/api/router/RouterOptions
- TanStack Router — Router API (preloading strategies) — https://mintlify.wiki/TanStack/router/router/api/router

---

## 10. TanStack Router vs React Router

### Definitions

**Core Definition:** TanStack Router and React Router are both production-grade React routers, but they differ in philosophy: TanStack Router is a typed URL state system, while React Router is a UI-mapping layer that can be progressively enhanced with data and framework modes.

**Technical Definition:** React Router v7 offers three modes: Declarative (thin UI routing), Data (loaders, actions, `createBrowserRouter`), and Framework (SSR, file-based routes, codegen types). TanStack Router offers a client-first typed route tree with built-in search-param validation, SWR caching, parallel loaders, and automatic preloading; TanStack Start adds SSR and server functions. The type-safety mechanisms differ: TanStack Router puts the cost in TypeScript inference over the generated route tree, while React Router v7 framework mode uses codegen to generate per-route types. React Router has a longer history (since 2014) and a larger ecosystem; TanStack Router (stable v1.0.0 December 2023) is newer and more opinionated about URL state.

**Beginner-Friendly Explanation:** React Router is the long-standing, flexible choice that starts simple and adds features as you need them. TanStack Router is the newer, opinionated choice that makes TypeScript and URL state the centre of the architecture from day one.

### Purposes

- To choose the router that matches the team's priorities and application shape.
- To understand the trade-offs in type safety, data loading, and ecosystem.
- To evaluate migration or adoption decisions with clear criteria.
- To align router choice with the rendering model (SPA, SSR, full-stack).

### Comparison Table

| Dimension | TanStack Router | React Router v7 |
|-----------|-----------------|-----------------|
| **Philosophy** | State-first: URL → validated state → loader data → UI | UI-first: URL → component; data layered on |
| **Type safety** | 100% inferred from route tree; no codegen step for consumers | Declarative/Data: limited; Framework mode: codegen types |
| **Search params** | First-class, JSON-first, schema-validated, typed | Stringly typed in Declarative/Data; improved in Framework |
| **Loaders** | Parallel by default; `loaderDeps` for cache keys | Data mode loaders; framework mode SSR |
| **Caching** | Built-in SWR (`staleTime`, `gcTime`) | Data mode loaders; Query integration |
| **Preloading** | `intent`, `viewport`, `render` | `prefetch` on links (framework mode) |
| **SSR** | Via TanStack Start | Framework mode (Vite plugin) |
| **Ecosystem** | Smaller, growing | Larger, longer history |
| **Bundle size** | Lightweight (~12 KB core) | Comparable; framework mode adds more |
| **Best for** | Type-safe, URL-state-heavy apps | Teams wanting gradual adoption and ecosystem depth |

**Syntax Rules:**

1. Choose TanStack Router when the URL is the primary state manager and type safety is a hard requirement.
2. Choose React Router when the team values ecosystem depth, gradual adoption, or existing v6 investment.
3. Both are production-grade; neither is universally "better."
4. TanStack Start is the closest comparison to React Router's framework mode.

### Annotated Code Examples

**TanStack Router (typed search + loader):**

```tsx
const schema = z.object({ page: fallback(z.number(), 1) });

export const Route = createFileRoute('/dashboard')({
  validateSearch: zodValidator(schema),
  loaderDeps: ({ search: { page } }) => ({ page }),
  loader: ({ deps }) => fetchIssues(deps),
});
```

**React Router v7 Framework Mode (codegen types):**

```tsx
export async function loader({ request }: Route.LoaderArgs) {
  const url = new URL(request.url);
  const page = Number(url.searchParams.get('page') ?? '1');
  const issues = await fetchIssues({ page });
  return { page, issues };
}
```

**Expected Output:** TanStack Router infers `page` as a number from the schema without manual parsing in the component. React Router Framework mode gets types from generated route modules but requires manual parsing inside the loader.

**Why this output:** TanStack Router's `validateSearch` output is the source of truth for search types. React Router's framework mode provides loader argument types via codegen, but search params remain stringly typed unless manually parsed.

### Real-World Cases

- **Type-safe SaaS:** TanStack Router for validated URL state.
- **Legacy React Router apps:** Incremental adoption of v7 data/framework mode.
- **E-commerce filters:** TanStack Router's typed search params.
- **Full-stack apps:** TanStack Start vs React Router framework mode.

### References

- Vercel — TanStack Router vs React Router — https://vercel.com/i/tanstack-router-vs-react-router
- TanStack Router — Comparison — https://tanstack.com/router/latest/docs/comparison
- TanStackShip — TanStack Router vs React Router v7 (2026) — https://tanstackship.com
- GitHub — Routing comparison issue — https://github.com/Pili-pinas/pili-pinas-app/issues/4

---

## 11. TanStack Router Best Practices

### Definitions

**Core Definition:** TanStack Router best practices are the guidelines for structuring routes, loaders, search params, caching, and preloading to build maintainable, performant, and type-safe applications.

**Technical Definition:** Best practices combine parallel data loading (nested loaders run in parallel; use `Promise.all` within loaders), factory-based router creation for SSR (to avoid cross-request state pollution), typed search-param validation with `fallback()` from the Zod adapter, `loaderDeps` discipline for cache keys, `ensureQueryData` for TanStack Query integration, `router.invalidate()` for post-mutation refresh, and intentional preloading strategies. They also include avoiding Node-only APIs in client loaders, tuning `staleTime` and `pendingMs`/`pendingMinMs`, and using pathless layouts and route groups to organise the tree without affecting URLs.

**Beginner-Friendly Explanation:** Good TanStack Router usage means loading data in parallel, validating the URL, keeping caches honest, refreshing after mutations, and preloading intentionally. It also means keeping server-only code out of client loaders and tuning loading states so they do not flash.

### Best Practices Table

| Practice | Description |
|----------|-------------|
| **Factory pattern for SSR** | Create a new router per request; never use a singleton in SSR. |
| **Parallel loaders** | Nested loaders run in parallel; use `Promise.all` for independent fetches within a loader. |
| **Use `ensureQueryData`** | In loaders with TanStack Query, use `ensureQueryData` (not `prefetchQuery`) so errors propagate. |
| **Declare `loaderDeps`** | Include every search param the loader reads in `loaderDeps` for correct cache keys. |
| **Validate search with schemas** | Use Zod + `zodValidator`; use `fallback()` instead of `.catch()` for Zod v3. |
| **Tune `staleTime`** | Default is `0`; set higher for static or expensive data. |
| **Tune pending UI** | `pendingMs` and `pendingMinMs` prevent flashing and ensure minimum display time. |
| **Invalidate after mutations** | Use `router.invalidate({ sync: true })` to re-run loaders. |
| **Use `router.invalidate()` for loader errors** | `reset()` only resets UI; `invalidate()` re-runs the loader. |
| **Avoid Node-only APIs in client loaders** | Use TanStack Start server functions for server-only work. |
| **Preload intentionally** | Choose `intent`, `viewport`, or `render` based on user behaviour; use delays. |
| **Use pathless layouts and groups** | Organise code without adding URL segments. |

### Annotated Code Examples

**✅ Good Practices:**

```tsx
// Factory pattern for SSR
export function createAppRouter() {
  return createRouter({
    routeTree,
    defaultPreload: 'intent',
    context: { queryClient },
  });
}

// Parallel loaders + ensureQueryData
export const Route = createFileRoute('/dashboard')({
  loader: async ({ context: { queryClient } }) => {
    await Promise.all([
      queryClient.ensureQueryData(statsQueryOptions()),
      queryClient.ensureQueryData(activityQueryOptions()),
    ]);
  },
  staleTime: 30_000,
  pendingMs: 500,
  pendingComponent: () => <DashboardSkeleton />,
});
```

**❌ Bad Practices:**

```tsx
// ❌ Waterfall: sequential awaits
const user = await fetchUser();
const posts = await fetchPosts(user.id);

// ❌ Missing loaderDeps: loader won't re-run on search change
loader: ({ search }) => fetchPosts(search.page),

// ❌ Node-only API in client loader
loader: async () => {
  const fs = await import('fs');
  return JSON.parse(fs.readFileSync('data.json', 'utf8'));
};

// ❌ Singleton router in SSR
export const router = createRouter({ routeTree });
```

**Expected Output:** The good practices produce parallel data loading, correct cache keys, safe SSR, and tuned loading states. The bad practices create waterfalls, stale data, browser crashes, and cross-request state pollution.

**Why this output:** Sequential awaits waste time. Missing `loaderDeps` breaks cache keys. Node APIs fail in the browser. Singleton routers leak state between SSR requests.

### Real-World Cases

- **SSR/SSG deployments:** Factory router pattern and entry separation.
- **Data-heavy dashboards:** Parallel loaders and tuned caching.
- **E-commerce:** Validated search params and intentional preloading.
- **All TanStack Router apps:** Schema validation, `ensureQueryData`, and invalidation discipline.

### References

- TanStack Router — SSR/SSG Best Practices (Cloudflare Pages) — https://cloud.tencent.cn/developer/article/2629209
- TanStack Router — load-parallel — https://github.com/supasheet/supasheet/blob/cdfcda6545ddbe3f536602ce0ef06842d460867c/.agents/skills/tanstack-router-best-practices/rules/load-parallel.md
- TanStack Router — Data Loading — https://tanstack.com/intent/registry/%40tanstack%2Frouter-core/data-loading
- TanStack Router — Router Data Loading — https://github.com/compozy/compozy/blob/5f48b457f7e4eaabaccfe08d83952be3dd743d18/.agents/skills/tanstack/references/router-data-loading.md

---

## References

- TanStack Router — Official Site — https://tanstack.com/router/latest
- TanStack Router — Overview — https://tanstack.com/router/latest/docs/overview
- TanStack Router — Routing Concepts — https://tanstack.com/router/latest/docs/routing/routing-concepts
- TanStack Router — Code-Based Routing — https://tanstack.com/router/latest/docs/routing/code-based-routing
- TanStack Router — File-Based Routing — https://tanstack.com/router/latest/docs/routing/file-based-routing
- TanStack Router — Type Safety — https://mintlify.wiki/TanStack/router/router/concepts/type-safety
- TanStack Router — Search Params — https://tanstack.com/router/latest/docs/guide/search-params
- TanStack Router — Preloading — https://tanstack.com/router/latest/docs/guide/preloading
- TanStack Router — RouterOptions (preload) — https://tanstack.com/router/latest/docs/api/router/RouterOptions
- TanStack Router — Comparison — https://tanstack.com/router/latest/docs/comparison
- TanStack Router — Data Loading — https://tanstack.com/intent/registry/%40tanstack%2Frouter-core/data-loading
- TanStack Router — Search Param Validation — https://tanstack.com/intent/registry/%40tanstack%2Frouter-core/search-params
- TanStack Router — Router Data Loading (compozy) — https://github.com/compozy/compozy/blob/5f48b457f7e4eaabaccfe08d83952be3dd743d18/.agents/skills/tanstack/references/router-data-loading.md
- TanStack Router — load-parallel (best practices) — https://github.com/supasheet/supasheet/blob/cdfcda6545ddbe3f536602ce0ef06842d460867c/.agents/skills/tanstack-router-best-practices/rules/load-parallel.md
- TanStack Router — Data Mutations — https://github.com/TanStack/router/blob/92655370a14761aaf59cacb86b1f7675b035d354/docs/guide/data-mutations.md
- TanStack Router — Caching — https://mintlify.wiki/TanStack/router/router/guide/caching
- TanStack Router — Router API (preloading) — https://mintlify.wiki/TanStack/router/router/api/router
- Vercel — TanStack Router vs React Router — https://vercel.com/i/tanstack-router-vs-react-router
- TanStackShip — TanStack Router vs React Router v7 (2026) — https://tanstackship.com
- TanStack Router — SSR/SSG Best Practices (Cloudflare Pages) — https://cloud.tencent.cn/developer/article/2629209
- GitHub — Routing comparison issue — https://github.com/Pili-pinas/pili-pinas-app/issues/4
- TanStack Router — createRouter type safety — https://github.com/TanStack/router
- TanStack Router — Router Type Safety — https://github.com/TanStack/router