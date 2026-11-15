# React Routing Fundamentals — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Routing is the mechanism that maps a URL to a specific view or component tree, and keeps that mapping in sync as the user navigates.

**Technical Definition:** Routing in a React application is the process of resolving the current URL into a matched route, rendering the corresponding component tree, and updating the URL when navigation occurs without necessarily reloading the document. Two architectural models exist: **server-side routing**, where each URL request is handled by the server and a fresh HTML document is returned, and **client-side routing**, where a JavaScript router intercepts navigation, updates the URL via the History API, and swaps components in place. React itself has no built-in router; the ecosystem provides routers such as React Router (declarative, data, and framework modes), TanStack Router (type-safe, search-param-first), and framework-integrated routers such as the Next.js App Router (file-based, Server Component aware).

**Beginner-Friendly Explanation:** Routing is how your app knows which page to show for which address. When you type `/dashboard` or click a link, routing decides what appears on screen. There are two big approaches. In a traditional website, every click asks the server for a brand-new page — that is server-side routing. In a modern single-page app, the browser loads one HTML document and a JavaScript router takes over: it changes the URL in the address bar and swaps the content without a full reload. That feels instant, but it means the router has to handle things the browser used to handle for free — like the back button, deep links, and scroll restoration.

### Key Characteristics

- **URL is the source of truth:** The router derives UI from the URL, so deep links, refresh, and back/forward navigation all work.
- **Two rendering models:** Server-side routing returns full documents per request; client-side routing swaps components in a single document.
- **History API is the substrate:** Client-side routing depends on `pushState`, `replaceState`, and the `popstate` event.
- **Route matching drives rendering:** Routers parse the URL into params and search params, match it against route definitions, and render the matched tree.
- **Nested routes compose layouts:** Parent routes render shared layout and an `<Outlet />` where child routes appear.
- **Guards and loaders gate navigation:** Authentication checks and data loading can run before a route renders.
- **Performance is a routing concern:** Code splitting per route, prefetching on intent, and avoiding waterfalls are router-level optimisations.

### Prerequisites

- React function components, JSX, and props.
- Basic understanding of URLs, paths, and query strings.
- Familiarity with `useState` and `useEffect`.
- Knowledge of the browser's `window.history` and `location` objects.
- Understanding of single-page vs multi-page application architecture.

### Related Programming Areas

- **Code Splitting** — route-level splitting is the highest-impact loading optimisation.
- **Server-Side Rendering (SSR)** — server routing and streaming SSR.
- **React Server Components** — file-based routers integrate RSC.
- **State Management** — URL state (`nuqs`, search params) vs. client state.
- **Accessibility** — focus management and route announcements.

---

## 1. Client-Side Routing

### Definitions

**Core Definition:** Client-side routing is navigation handled entirely in the browser by JavaScript, where the URL changes and components swap without a full page reload.

**Technical Definition:** Client-side routing intercepts link clicks and programmatic navigation, prevents the browser's default document request, updates the URL using the History API (`pushState`/`replaceState`), and renders a new component tree based on the matched route. The initial HTML document is loaded once; subsequent navigations fetch data and code as needed. React Router, TanStack Router, and framework routers (Next.js, Remix) implement this model. Modern routers additionally support data loading (`loader`), route-level code splitting (`lazy`), and pending UI states.

**Beginner-Friendly Explanation:** Client-side routing means your app loads once and then changes what is on screen without asking the server for a whole new page. The URL still updates so bookmarks and the back button work, but the page never fully reloads.

### Purposes

- To provide instant-feeling navigation without full document reloads.
- To preserve application state (e.g., player state, open modals) across navigations.
- To enable route-level code splitting and prefetching.
- To support rich, app-like interactions and transitions.
- To keep the URL as the source of truth for deep linking and back/forward navigation.

### Syntax Rules and Structure

**General Syntax (React Router v7 Declarative Mode):**

```jsx
import { BrowserRouter, Routes, Route } from 'react-router';

<BrowserRouter>
  <Routes>
    <Route path="/" element={<Home />} />
    <Route path="/about" element={<About />} />
    <Route path="/users/:id" element={<User />} />
  </Routes>
</BrowserRouter>
```

**Component Breakdown:**

| Element | Description |
|---------|-------------|
| `BrowserRouter` | Provides the history-based router context. |
| `Routes` | Matches the current URL against child routes. |
| `Route` | Defines a path and the element to render. |
| `path` | The URL pattern, including `:param` segments. |
| `element` | The React element rendered when the route matches. |

**Syntax Rules:**

1. Wrap the app in a router (`BrowserRouter`, `createBrowserRouter`, or a framework router).
2. Define routes with `path` and `element` (or `component`).
3. Use `:param` for dynamic segments and `*` for splats.
4. Use `Link` or `NavLink` for internal navigation; plain `<a href>` triggers a full reload.
5. Use `useNavigate` for programmatic navigation.

**Constraints and Limitations:**

- Deep links require the server to return the app shell for all routes (SPA fallback).
- SEO requires SSR, SSG, or prerendering for content pages.
- State is lost on full reload unless persisted.
- Memory and bundle size grow with the app; route-level splitting is essential.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Basic Client-Side Router

```jsx
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

**Why this output:** `BrowserRouter` listens for navigation events. `Link` calls `preventDefault` and uses the History API. `Routes` matches `/about` and renders the corresponding element.

#### Example 2: Programmatic Navigation

```jsx
import { useNavigate } from 'react-router';

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

**Expected Output:** After a successful login, the app navigates to `/dashboard` and the login page is replaced in history.

**Why this output:** `useNavigate()` returns a function that pushes (or replaces) a history entry. `replace: true` prevents the login page from appearing in the back stack.

### Real-World Cases

- **SaaS dashboards:** Navigating between views without losing application state.
- **Content apps:** Instant navigation between articles with prefetching.
- **E-commerce:** Product listing → product detail → cart, all client-side.

### References

- React Router — Official Documentation — https://reactrouter.com
- React Router — Picking a Mode — https://reactrouter.com/start/modes
- MDN — History API — https://developer.mozilla.org/en-US/docs/Web/API/History_API

---

## 2. Server-Side Routing

### Definitions

**Core Definition:** Server-side routing is navigation in which each URL request is handled by the server, which returns a complete HTML document for that route.

**Technical Definition:** In server-side routing, the browser requests a URL, the server maps that URL to a handler, renders or fetches the corresponding HTML, and returns a full document. Each navigation is a document request. Modern React frameworks combine server-side routing with **SSR** (rendering React on the server) and **hydration** (making the HTML interactive). Frameworks such as Next.js, Remix/React Router framework mode, and TanStack Start use file-based or configuration-based server routing with Server Components and streaming.

**Beginner-Friendly Explanation:** Server-side routing is the classic model: every click asks the server for a new page, and the server sends back a complete HTML document.

### Purposes

- To deliver fully rendered HTML for SEO and fast first paint.
- To support users without JavaScript (progressive enhancement).
- To keep sensitive logic and data fetching on the server.
- To simplify deep linking and browser behaviour (back/forward, refresh).
- To enable streaming SSR and React Server Components.

### Syntax Rules and Structure

**General Syntax (Next.js App Router, file-based):**

```text
app/
├── layout.tsx        → shared layout
├── page.tsx          → "/" route
├── about/
│   └── page.tsx      → "/about" route
└── users/
    └── [id]/
        └── page.tsx  → "/users/:id" route
```

**Component Breakdown:**

| File / Folder | Description |
|---------------|-------------|
| `layout.tsx` | Shared UI that wraps child routes. |
| `page.tsx` | The route's page component. |
| `[id]` | Dynamic segment; exposed as a param. |
| `loading.tsx` | Suspense fallback for the route segment. |
| `error.tsx` | Error boundary for the route segment. |

**Syntax Rules:**

1. File-system folders define URL segments; `page.tsx` makes a segment routable.
2. Dynamic segments use square brackets: `[id]`.
3. Layouts wrap children and persist across navigation.
4. `loading.tsx` and `error.tsx` provide route-level pending and error states.
5. Server Components run on the server and ship no JavaScript.

**Constraints and Limitations:**

- Each navigation is a document request unless enhanced with client-side navigation.
- Server capacity and latency affect every navigation.
- Client-side interactivity requires hydration or Client Components.

### Annotated Code Examples

```tsx
// app/users/[id]/page.tsx — Server Component
export default async function UserPage({ params }: { params: { id: string } }) {
  const user = await fetchUser(params.id);
  return <h1>{user.name}</h1>;
}
```

**Expected Output:** The server renders the user's name into the HTML for `/users/123`. The browser receives a complete document.

**Why this output:** The file path `app/users/[id]/page.tsx` maps to `/users/:id`. The component is async and runs on the server, fetching data before rendering HTML.

### Real-World Cases

- **Content sites:** Blogs and documentation that need SEO and fast first paint.
- **E-commerce:** Product pages that must be indexable and render fast.
- **Enterprise apps:** Server-rendered shells with client-side interactivity.

### References

- Next.js — Routing Fundamentals — https://nextjs.org/docs/app/building-your-application/routing
- Next.js — Linking and Navigating — https://nextjs.org/docs/app/building-your-application/routing/linking-and-navigating
- React Router — Framework Mode — https://reactrouter.com/start/modes

---

## 3. Single-Page Applications (SPAs)

### Definitions

**Core Definition:** A single-page application is a web application that loads one HTML document and updates the view dynamically as the user navigates, without full page reloads.

**Technical Definition:** An SPA serves a minimal HTML shell and a JavaScript bundle. A client-side router manages navigation, rendering route components into the document. Data is fetched via APIs. Modern SPAs may be enhanced with SSR/SSG (hybrid architectures) and route-level code splitting. The SPA model trades the simplicity of server routing for stateful, app-like interactivity and reduced navigation latency.

**Beginner-Friendly Explanation:** An SPA is an app that loads once and then changes content on the fly. It feels like a desktop app rather than a website that reloads on every click.

### Purposes

- To provide fast, stateful navigation after the initial load.
- To preserve application state across views.
- To enable rich client-side interactions and transitions.
- To decouple the frontend from the backend via APIs.
- To support offline and progressive web app capabilities.

### Syntax Rules and Structure

**General Syntax (SPA Fallback on the Server):**

```js
// Express example: serve index.html for all non-API routes
app.get(/^(?!\/api).*/, (req, res) => {
  res.sendFile(path.join(__dirname, 'dist/index.html'));
});
```

**Syntax Rules:**

1. Configure the server to return the SPA shell for all client routes.
2. Use a client-side router (`BrowserRouter`, `createBrowserRouter`).
3. Split code per route to control the initial bundle.
4. Provide a `<noscript>` fallback or SSR for accessibility and SEO.
5. Use a 404 route for unmatched paths.

**Constraints and Limitations:**

- The initial load includes the router and shell before any content.
- SEO requires SSR, SSG, or prerendering.
- Deep links fail if the server does not serve the shell for all routes.

### Annotated Code Examples

```jsx
// SPA root with route-level code splitting
import { lazy, Suspense } from 'react';
import { BrowserRouter, Routes, Route } from 'react-router';

const Home = lazy(() => import('./pages/Home'));
const Dashboard = lazy(() => import('./pages/Dashboard'));

function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<PageSkeleton />}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/dashboard" element={<Dashboard />} />
          <Route path="*" element={<NotFound />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
}
```

**Expected Output:** The SPA loads once; navigating to `/dashboard` fetches only the dashboard chunk.

**Why this output:** `lazy` creates a split point per route. `Suspense` shows a fallback while the chunk loads. The wildcard route handles unknown URLs.

### Real-World Cases

- **SaaS products:** Dashboards and tools with heavy client interactivity.
- **Internal tools:** Admin panels behind authentication.
- **Progressive web apps:** Offline-capable applications.

### References

- MDN — SPA (Single-page application) — https://developer.mozilla.org/en-US/docs/Glossary/SPA
- React Router — Picking a Mode — https://reactrouter.com/start/modes

---

## 4. Multi-Page Applications (MPAs)

### Definitions

**Core Definition:** A multi-page application is a web application composed of multiple HTML documents, each served by the server for a distinct URL.

**Technical Definition:** In an MPA, every navigation is a document request. The server routes the URL, renders or fetches HTML, and returns a full page. React can still be used per page (islands or per-page roots), but state does not persist across navigations. Modern frameworks blur the line: Next.js and Remix use server routing with client-side transitions after hydration, producing a **hybrid** model.

**Beginner-Friendly Explanation:** An MPA is a traditional website where every link loads a new page from the server. React can still power each page, but the browser reloads between pages.

### Purposes

- To simplify SEO and deep linking through server-rendered documents.
- To keep the initial payload small per page.
- To isolate failures and state between pages.
- To support users without JavaScript.
- To leverage server-side caching and CDNs per page.

### Syntax Rules and Structure

**General Syntax (per-page React root):**

```jsx
// Each page has its own entry point
import { createRoot } from 'react-dom/client';

const root = createRoot(document.getElementById('react-root'));
root.render(<ProductPage product={window.__PRODUCT__} />);
```

**Syntax Rules:**

1. Each page has its own HTML document and entry script.
2. Server-side templating injects initial data into the page.
3. Navigation uses standard `<a href>` links.
4. Shared components are bundled per page or shared across pages.

**Constraints and Limitations:**

- Full reloads lose client state and add latency.
- Shared bundles can be duplicated across pages unless configured.
- Transitions and app-like interactions are limited.

### Annotated Code Examples

```html
<!-- page.html -->
<div id="react-root"></div>
<script>
  window.__PRODUCT__ = { id: 1, name: 'Widget' };
</script>
<script type="module" src="/product.js"></script>
```

**Expected Output:** The product page renders with server-injected data and no client-side router.

**Why this output:** The server provides the HTML and initial data; React hydrates or renders the page root.

### Real-World Cases

- **Marketing sites:** Static or server-rendered pages with occasional React islands.
- **Documentation sites:** Each page is a separate document.
- **Legacy migrations:** Adding React to an existing multi-page site.

### References

- MDN — SPA (contrast with MPA) — https://developer.mozilla.org/en-US/docs/Glossary/SPA
- Next.js — Rendering — https://nextjs.org/docs/app/building-your-application/rendering

---

## 5. History API

### Definitions

**Core Definition:** The History API is the browser interface (`window.history`) that lets JavaScript add, replace, and respond to browser history entries without reloading the page.

**Technical Definition:** The History API exposes `history.pushState(state, unused, url)`, `history.replaceState(state, unused, url)`, `history.back()`, `history.forward()`, `history.go(n)`, and the `popstate` event. Client-side routers use `pushState` for new navigations and `replaceState` for redirects or state replacement. When the user presses back/forward, the browser fires `popstate`, and the router re-renders based on the new `location`. The newer Navigation API (`window.navigation`) offers a more capable successor in supporting browsers.

**Beginner-Friendly Explanation:** The History API lets your app change the URL without reloading. It is how client-side routers make the address bar update while the page stays put.

### Purposes

- To update the URL without a full page reload.
- To add or replace history entries for back/forward navigation.
- To respond to back/forward navigation with `popstate`.
- To store small amounts of state associated with a history entry.
- To enable shareable, bookmarkable client-side URLs.

### Syntax Rules and Structure

**General Syntax:**

```js
history.pushState({ page: 1 }, '', '/page-1');
history.replaceState({ page: 2 }, '', '/page-2');

window.addEventListener('popstate', (event) => {
  console.log('location:', location.pathname, 'state:', event.state);
});
```

**Component Breakdown:**

| API | Description |
|-----|-------------|
| `pushState(state, unused, url)` | Adds a new history entry. |
| `replaceState(state, unused, url)` | Replaces the current entry. |
| `back()` / `forward()` / `go(n)` | Navigate through history. |
| `popstate` event | Fired on back/forward navigation. |
| `history.state` | The state associated with the current entry. |

**Syntax Rules:**

1. Use `pushState` for new navigations; `replaceState` for redirects and corrections.
2. The `state` object must be structured-cloneable.
3. `popstate` fires only on back/forward, not on `pushState`/`replaceState`.
4. Always handle `popstate` to keep the UI in sync with the URL.
5. The server must serve the SPA shell for client-side paths.

**Constraints and Limitations:**

- `pushState` does not trigger a page load; the app must render the new view itself.
- `popstate` does not fire when `pushState` is called.
- Cross-origin URLs are not allowed in `pushState`.
- The Navigation API is not yet universally supported.

### Annotated Code Examples

```jsx
import { useEffect, useState } from 'react';

function usePathname() {
  const [pathname, setPathname] = useState(window.location.pathname);

  useEffect(() => {
    const onPop = () => setPathname(window.location.pathname);
    window.addEventListener('popstate', onPop);
    return () => window.removeEventListener('popstate', onPop);
  }, []);

  function navigate(to) {
    window.history.pushState({}, '', to);
    setPathname(to); // pushState does not fire popstate
  }

  return { pathname, navigate };
}
```

**Expected Output:** Calling `navigate('/about')` updates the URL and the rendered path. Pressing Back fires `popstate` and restores the previous path.

**Why this output:** `pushState` updates the URL but does not notify listeners, so the hook updates state manually. `popstate` handles browser-initiated navigation.

### Real-World Cases

- **Custom routers:** Building a minimal router without a library.
- **URL-driven filters:** Updating search params without reloading.
- **Modal routes:** Opening a modal at a URL that can be shared and backed out of.

### References

- MDN — History API — https://developer.mozilla.org/en-US/docs/Web/API/History_API
- MDN — Working with the History API — https://developer.mozilla.org/en-US/docs/Web/API/History_API/Working_with_the_History_API
- MDN — Window: popstate event — https://developer.mozilla.org/en-US/docs/Web/API/Window/popstate_event
- MDN — Navigation API — https://developer.mozilla.org/en-US/docs/Web/API/Navigation_API

---

## 6. Hash Routing

### Definitions

**Core Definition:** Hash routing is client-side routing that uses the URL fragment (`#/path`) instead of the pathname, so the browser never sends the route to the server.

**Technical Definition:** Hash routing stores the route in `location.hash` (e.g., `https://example.com/#/users/123`). Changing the hash fires `hashchange` and does not trigger a document request, which means it works without server-side SPA fallback configuration. React Router provides `createHashRouter`; TanStack Router supports hash history. The trade-off is that fragments are not sent to the server, so they cannot be used for SSR route resolution, and they are less idiomatic for modern SEO-aware applications.

**Beginner-Friendly Explanation:** Hash routing puts the page path after a `#` in the URL. The browser ignores everything after `#` when talking to the server, so it works even on static file hosts without special configuration.

### Purposes

- To enable client-side routing without server configuration.
- To support routing on static hosts (GitHub Pages, S3) that cannot serve SPA fallbacks.
- To avoid server-side route resolution entirely.
- To support embedding scenarios where pathname control is unavailable.

### Syntax Rules and Structure

**General Syntax (React Router):**

```jsx
import { createHashRouter, RouterProvider } from 'react-router';

const router = createHashRouter([
  { path: '/', element: <Home /> },
  { path: '/about', element: <About /> },
]);

<RouterProvider router={router} />
```

**Native Hash Handling:**

```js
window.addEventListener('hashchange', () => {
  const route = window.location.hash.slice(1); // "/about"
  renderRoute(route);
});
```

**Syntax Rules:**

1. Use `createHashRouter` (React Router) or hash history (TanStack Router).
2. The route lives after `#`; the pathname stays `/`.
3. Listen to `hashchange` for native implementations.
4. Hash routes are not sent to the server; SSR cannot resolve them.

**Constraints and Limitations:**

- Fragments are invisible to the server; SSR and server-side redirects cannot use them.
- URLs look unusual (`/#/about`) and are less SEO-friendly.
- Some analytics tools handle hash routes poorly.
- Anchor links and hash routing can conflict.

### Annotated Code Examples

```jsx
import { createHashRouter, RouterProvider } from 'react-router';

const router = createHashRouter([
  { path: '/', element: <Home /> },
  { path: '/about', element: <About /> },
]);

function App() {
  return <RouterProvider router={router} />;
}
```

**Expected Output:** Navigating to `/about` produces `https://example.com/#/about` and renders `About` without a server request.

**Why this output:** `createHashRouter` uses `location.hash` for routing. The browser does not send the fragment to the server, so no SPA fallback is required.

### Real-World Cases

- **Static hosting:** GitHub Pages and S3 deployments without rewrite rules.
- **Electron apps:** File-based apps where pathname routing is awkward.
- **Embedded widgets:** Routing inside an iframe or widget where pathname is controlled by the host.

### References

- React Router — createHashRouter — https://reactrouter.com/api/data-routers/createHashRouter
- MDN — Window: hashchange event — https://developer.mozilla.org/en-US/docs/Web/API/Window/hashchange_event
- TanStack Router — History Types — https://tanstack.com/router/latest/docs/framework/react/guide/history-types

---

## 7. Route Matching

### Definitions

**Core Definition:** Route matching is the process of comparing the current URL against route definitions to determine which route (and component tree) should render.

**Technical Definition:** Routers parse the URL into pathname segments, match them against route patterns (static segments, dynamic `:params`, optional segments, and splats `*`), and select the best match by specificity. React Router ranks routes by specificity and supports nested matching; TanStack Router uses a route tree with typed paths and search schemas. Matching also extracts params and search params for the matched route.

**Beginner-Friendly Explanation:** Route matching is how the router decides “this URL belongs to this page.” It compares the address to your list of routes and picks the one that fits.

### Purposes

- To resolve the current URL to a component tree.
- To extract dynamic segments (`:id`) and search params.
- To support nested layouts through parent-child matching.
- To provide deterministic behaviour when multiple patterns could match.
- To enable type-safe routing with typed params and search.

### Syntax Rules and Structure

**General Syntax (React Router route objects):**

```jsx
const router = createBrowserRouter([
  { path: '/', element: <Home /> },
  { path: '/users', element: <UsersLayout />, children: [
    { index: true, element: <UserList /> },
    { path: ':id', element: <UserDetail /> },
    { path: '*', element: <NotFound /> },
  ]},
]);
```

**Pattern Types:**

| Pattern | Example | Matches |
|---------|---------|---------|
| Static | `/about` | Exactly `/about` |
| Dynamic param | `/users/:id` | `/users/123` |
| Optional param | `/users/:id?` | `/users` and `/users/123` |
| Splat | `/files/*` | `/files/a/b/c` |
| Index | `index: true` | The parent path itself |

**Syntax Rules:**

1. Define the most specific routes first; routers rank by specificity, but explicit ordering aids readability.
2. Use `:param` for dynamic segments and `*` for catch-all.
3. Use `index: true` for the default child of a layout route.
4. Provide a catch-all route (`*`) for 404s.
5. Prefer route objects (`createBrowserRouter`) for data APIs; use JSX `<Routes>` for simple cases.

**Constraints and Limitations:**

- Ambiguous patterns can produce unexpected matches; keep patterns distinct.
- Route order can matter in some routers; consult the router's ranking rules.
- Dynamic segments are strings; parse and validate them in the component or loader.

### Annotated Code Examples

```jsx
import { createBrowserRouter, RouterProvider } from 'react-router';

const router = createBrowserRouter([
  { path: '/', element: <Home /> },
  { path: '/products/:category/:id', element: <ProductDetail /> },
  { path: '/files/*', element: <FileBrowser /> },
  { path: '*', element: <NotFound /> },
]);

function App() {
  return <RouterProvider router={router} />;
}
```

**Expected Output:** `/products/shoes/42` matches `ProductDetail` with `category="shoes"` and `id="42"`. `/files/a/b` matches `FileBrowser` with a splat. Any other path matches `NotFound`.

**Why this output:** The router parses pathname segments, matches `:category` and `:id` as params, and uses the splat for the remainder of the path.

### Real-World Cases

- **E-commerce:** `/products/:category/:slug` patterns.
- **Documentation:** `/docs/:section/:page` with nested layouts.
- **Dashboards:** `/orgs/:orgId/projects/:projectId`.

### References

- React Router — Route Objects — https://reactrouter.com/start/data/route-object
- React Router — Route — https://reactrouter.com/api/components/Route
- TanStack Router — Route Trees & Nesting — https://tanstack.com/router/latest/docs/framework/react/guide/route-trees

---

## 8. Route Parameters

### Definitions

**Core Definition:** Route parameters are dynamic segments in a URL pattern (e.g., `:id` in `/users/:id`) whose values are extracted and made available to the matched component.

**Technical Definition:** Route params are parsed from the pathname by the router and exposed via a hook (`useParams` in React Router and TanStack Router) or as a prop/loader argument. Params are strings; they must be validated and parsed (e.g., to numbers or IDs) before use. They are part of the URL's identity and should be treated as the source of truth for resource selection.

**Beginner-Friendly Explanation:** Route params are the variable parts of a URL. In `/users/123`, the `123` is a parameter that tells your component which user to show.

### Purposes

- To select a resource by its identifier from the URL.
- To make URLs shareable and bookmarkable for specific resources.
- To avoid storing selection state separately from the URL.
- To drive data loading in loaders based on the param.
- To enable nested resource routes (e.g., `/orgs/:orgId/projects/:projectId`).

### Syntax Rules and Structure

**General Syntax:**

```jsx
// Route definition
<Route path="/users/:id" element={<UserDetail />} />

// Accessing the param
import { useParams } from 'react-router';

function UserDetail() {
  const { id } = useParams();
  return <h1>User {id}</h1>;
}
```

**Syntax Rules:**

1. Define params in the path with `:name`.
2. Read them with `useParams()` (React Router, TanStack Router).
3. Params are always strings; parse and validate them.
4. Use the param in loaders to fetch data before rendering.
5. Do not put sensitive data in params; URLs are logged and shared.

**Constraints and Limitations:**

- Params cannot contain `/` unless encoded.
- Invalid or missing params must be handled (404 or redirect).
- Params do not update component state automatically; use them as derived inputs.

### Annotated Code Examples

```tsx
import { useParams } from 'react-router';

function UserDetail() {
  const { id } = useParams();
  const userId = Number(id);

  if (!Number.isInteger(userId)) {
    return <NotFound />;
  }

  return <UserProfile userId={userId} />;
}
```

**Expected Output:** `/users/123` renders the profile for user 123. `/users/abc` renders `NotFound`.

**Why this output:** `useParams` extracts `id` as a string. The component parses and validates it before rendering. Invalid values are handled explicitly.

### Real-World Cases

- **User profiles:** `/users/:id`.
- **Product pages:** `/products/:slug`.
- **Multi-tenant apps:** `/orgs/:orgId/settings`.
- **Nested resources:** `/projects/:projectId/tasks/:taskId`.

### References

- React Router — useParams — https://reactrouter.com/api/hooks/useParams
- TanStack Router — Path Params — https://tanstack.com/router/latest/docs/framework/react/guide/path-params

---

## 9. Query Parameters

### Definitions

**Core Definition:** Query parameters are key-value pairs appended to a URL after `?` (e.g., `?page=2&sort=asc`) that configure the view without changing the path.

**Technical Definition:** Query parameters (search params) are parsed from `location.search` and exposed via `useSearchParams` (React Router) or a typed search schema (TanStack Router). They are ideal for **view configuration** — filters, sorting, pagination, tabs — because they are shareable, bookmarkable, and survive reloads. TanStack Router treats search params as first-class, type-safe state with schema validation. React Router's `useSearchParams` returns a `URLSearchParams` pair with a setter.

**Beginner-Friendly Explanation:** Query parameters are the extra settings in a URL after the `?`. They are perfect for things like “page 2, sorted by price” because you can share that exact view with someone.

### Purposes

- To store view configuration (filters, sort, pagination) in the URL.
- To make filtered and paginated views shareable and bookmarkable.
- To survive reloads and back/forward navigation.
- To separate resource identity (path params) from view options (search params).
- To enable server-side and client-side filtering from the same URL.

### Syntax Rules and Structure

**General Syntax (React Router):**

```jsx
import { useSearchParams } from 'react-router';

const [searchParams, setSearchParams] = useSearchParams();
const page = Number(searchParams.get('page') ?? '1');

setSearchParams({ page: '2', sort: 'asc' });
```

**General Syntax (TanStack Router, typed):**

```tsx
const search = useSearch({ from: '/products' });
// search is typed from the route's validateSearch schema
```

**Syntax Rules:**

1. Use `useSearchParams` or the router's typed search API.
2. Parse and validate search params (numbers, enums, booleans).
3. Use `setSearchParams` with `replace: true` for filter changes that should not add history entries.
4. Keep resource identity in the path and view options in the query string.
5. Provide defaults for missing params.

**Constraints and Limitations:**

- Search params are strings; parsing and validation are required.
- Long or complex state is awkward in the URL; use a schema and compact keys.
- Sensitive data should not be placed in search params.

### Annotated Code Examples

```tsx
import { useSearchParams } from 'react-router';

function ProductList() {
  const [searchParams, setSearchParams] = useSearchParams();
  const page = Number(searchParams.get('page') ?? '1');
  const sort = searchParams.get('sort') ?? 'relevance';

  function updateSort(nextSort) {
    setSearchParams({ page: '1', sort: nextSort }, { replace: true });
  }

  return (
    <div>
      <p>Page {page}, sorted by {sort}</p>
      <button onClick={() => updateSort('price')}>Sort by price</button>
    </div>
  );
}
```

**Expected Output:** Clicking “Sort by price” updates the URL to `?page=1&sort=price` and re-renders the list. Reloading preserves the sort.

**Why this output:** `useSearchParams` reads from and writes to `location.search`. `replace: true` avoids polluting history with every filter change.

### Real-World Cases

- **E-commerce filters:** `?category=shoes&size=42&sort=price`.
- **Search results:** `?q=react&page=3`.
- **Dashboards:** `?range=30d&tab=revenue`.
- **Documentation:** `?version=latest`.

### References

- React Router — useSearchParams — https://reactrouter.com/api/hooks/useSearchParams
- TanStack Router — Search Params — https://tanstack.com/router/latest/docs/framework/react/guide/search-params

---

## 10. Nested Routes

### Definitions

**Core Definition:** Nested routes are child routes rendered inside a parent route's layout, allowing shared UI to persist while only the child content changes.

**Technical Definition:** Nested routes define a parent-child hierarchy: the parent route renders shared layout (navigation, sidebar, header) and an `<Outlet />` (React Router) or `<Outlet />` equivalent where the matched child renders. The router matches the full URL against the route tree and renders the chain of matched routes from root to leaf. Nested routes reduce duplication and make layouts persistent across navigations.

**Beginner-Friendly Explanation:** Nested routes let you share a layout across pages. The sidebar and header stay put while the main content changes as you navigate between child pages.

### Purposes

- To share layout UI across related routes.
- To avoid duplicating headers, sidebars, and navigation.
- To preserve layout state (e.g., scroll position in a sidebar) across child navigation.
- To mirror the application's information architecture.
- To enable route-level code splitting per nested segment.

### Syntax Rules and Structure

**General Syntax (React Router):**

```jsx
import { Outlet } from 'react-router';

function DashboardLayout() {
  return (
    <div className="dashboard">
      <Sidebar />
      <main>
        <Outlet /> {/* Child route renders here */}
      </main>
    </div>
  );
}

const router = createBrowserRouter([
  {
    path: '/dashboard',
    element: <DashboardLayout />,
    children: [
      { index: true, element: <Overview /> },
      { path: 'reports', element: <Reports /> },
      { path: 'settings', element: <Settings /> },
    ],
  },
]);
```

**Syntax Rules:**

1. Define child routes in the parent's `children` array (React Router) or nested route files (TanStack Router, Next.js).
2. Render `<Outlet />` in the parent to display the matched child.
3. Use `index: true` for the default child at the parent path.
4. Paths in children are relative to the parent (`reports`, not `/dashboard/reports`).
5. Layouts persist across child navigation; only the outlet content changes.

**Constraints and Limitations:**

- Deeply nested layouts can complicate data loading and error boundaries.
- Layout state is preserved, which may be undesirable for some transitions.
- Relative path resolution can be confusing; use leading `/` for absolute paths.

### Annotated Code Examples

```tsx
import { Outlet } from 'react-router';

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

**Expected Output:** Navigating from `/settings/profile` to `/settings/billing` keeps the settings navigation visible and swaps only the content area.

**Why this output:** `SettingsLayout` is the parent route. `Outlet` renders the matched child (`Profile` or `Billing`). The parent layout persists across child navigation.

### Real-World Cases

- **Dashboards:** Persistent sidebar with changing main content.
- **Settings pages:** Shared settings navigation with per-section content.
- **Documentation:** Persistent sidebar and header with nested doc pages.
- **E-commerce:** Shared product layout with tabs (description, reviews, shipping).

### References

- React Router — Outlet — https://reactrouter.com/api/components/Outlet
- React Router — Nested Routes — https://reactrouter.com/start/data/route-object
- TanStack Router — Route Trees & Nesting — https://tanstack.com/router/latest/docs/framework/react/guide/route-trees

---

## 11. Route Guards

### Definitions

**Core Definition:** A route guard is a check that runs before a route renders or before navigation completes, deciding whether to allow, redirect, or block access.

**Technical Definition:** Route guards enforce authorisation and preconditions. In React Router data mode, guards are implemented in `loader` functions by throwing a `redirect()` when the condition fails. In TanStack Router, `beforeLoad` runs before loaders and can throw redirects or return context. In Next.js, middleware and layout-level checks enforce access. Guards must always be backed by server-side authorisation; client-side guards are a UX layer, not a security boundary.

**Beginner-Friendly Explanation:** A route guard is a bouncer for pages. Before letting you into `/admin`, it checks whether you are allowed in. If not, it sends you to the login page or a “forbidden” page.

### Purposes

- To prevent unauthenticated users from accessing protected routes.
- To redirect users based on roles or permissions.
- To enforce preconditions (e.g., onboarding completed, terms accepted).
- To avoid rendering pages that would immediately fail.
- To centralise access logic instead of duplicating checks in components.

### Syntax Rules and Structure

**General Syntax (React Router loader guard):**

```jsx
import { redirect } from 'react-router';

async function requireAuth() {
  const user = await getUser();
  if (!user) throw redirect('/login');
  return user;
}

const router = createBrowserRouter([
  {
    path: '/admin',
    loader: async () => {
      const user = await requireAuth();
      if (user.role !== 'admin') throw redirect('/forbidden');
      return null;
    },
    element: <AdminLayout />,
  },
]);
```

**General Syntax (TanStack Router `beforeLoad`):**

```tsx
const adminRoute = createRoute({
  getParentRoute: () => rootRoute,
  path: '/admin',
  beforeLoad: async ({ context }) => {
    if (!context.auth.user) throw redirect({ to: '/login' });
  },
});
```

**Syntax Rules:**

1. Run guards in `loader` (React Router) or `beforeLoad` (TanStack Router) so they execute before rendering.
2. Throw `redirect()` to navigate away; do not render the protected component.
3. Always enforce authorisation on the server; client guards are UX only.
4. Preserve the intended destination (`?redirect=/admin`) for post-login return.
5. Handle loading and error states for async guards.

**Constraints and Limitations:**

- Client-side guards can be bypassed; they are not a security control.
- Guards that run after rendering cause flicker and wasted work.
- Redirect loops can occur if the guard and login routes are misconfigured.

### Annotated Code Examples

```tsx
import { redirect } from 'react-router';

export async function loader({ request }) {
  const user = await getCurrentUser();
  if (!user) {
    const url = new URL(request.url);
    throw redirect(`/login?redirect=${encodeURIComponent(url.pathname)}`);
  }
  return { user };
}
```

**Expected Output:** Unauthenticated users are redirected to `/login` with a `redirect` parameter so they return to the original page after logging in.

**Why this output:** The loader runs before the route renders. Throwing `redirect` aborts rendering and navigates. The `redirect` param preserves the intended destination.

### Real-World Cases

- **Admin panels:** Role-based access to `/admin/*`.
- **Account pages:** Requiring authentication for `/account/*`.
- **Onboarding:** Redirecting to onboarding until the profile is complete.
- **Feature flags:** Gating routes behind enabled features.

### References

- React Router — redirect — https://reactrouter.com/api/data-routers/redirect
- React Router — Loaders — https://reactrouter.com/start/data/route-object
- TanStack Router — Authenticated Routes — https://tanstack.com/router/latest/docs/framework/react/guide/authenticated-routes
- Next.js — Middleware — https://nextjs.org/docs/app/building-your-application/routing/middleware

---

## 12. Route Transitions

### Definitions

**Core Definition:** Route transitions are the visual and focus changes that occur when navigating from one route to another.

**Technical Definition:** Route transitions include pending UI (spinners, progress bars, optimistic navigation), view transitions (`document.startViewTransition`), scroll restoration, and focus management for accessibility. React Router exposes `useNavigation()` to detect pending navigation states. TanStack Router provides `useRouterState` and pending-match APIs. The browser View Transitions API can animate between routes in supporting browsers.

**Beginner-Friendly Explanation:** A route transition is what the user sees while the next page is loading and after it appears — a spinner, a smooth fade, the scroll position resetting, and focus moving to the new content.

### Purposes

- To provide feedback during navigation (pending UI).
- To make navigation feel smooth and intentional.
- To reset scroll position appropriately on navigation.
- To move focus to the new content for screen-reader users.
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

**Syntax Rules:**

1. Use the router's pending-navigation API for progress indicators.
2. Scroll to the top on pathname change (unless the router handles it).
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

## 13. Route Rendering

### Definitions

**Core Definition:** Route rendering is the process by which a router turns the matched route into rendered React elements.

**Technical Definition:** Route rendering has three common modes: **element-based** (the route declares an `element` that renders immediately), **component-based** (the route names a component), and **data-aware** (the router runs `loader` functions before rendering and provides the data via `useLoaderData`). React Router v7 offers declarative, data, and framework modes; TanStack Router uses a route tree with `component` and `loader`. Rendering may also be lazy (`lazy` route property or `React.lazy`) to defer route code.

**Beginner-Friendly Explanation:** Route rendering is how the router actually puts the page on screen. It either renders the component right away, or fetches the data first and then renders with that data.

### Purposes

- To render the component tree for the matched route.
- To load route data before rendering (data-aware rendering).
- To lazily load route code to reduce the initial bundle.
- To provide route-level loading and error states.
- To coordinate rendering with Suspense and streaming.

### Syntax Rules and Structure

**Data-aware rendering (React Router):**

```jsx
// Route definition
{ path: '/users/:id', loader: userLoader, element: <UserDetail /> }

// Component
import { useLoaderData } from 'react-router';

function UserDetail() {
  const user = useLoaderData();
  return <h1>{user.name}</h1>;
}
```

**Lazy route rendering:**

```jsx
const router = createBrowserRouter([
  {
    path: '/reports',
    lazy: async () => {
      const { Reports, reportsLoader } = await import('./reports');
      return { Component: Reports, loader: reportsLoader };
    },
  },
]);
```

**Syntax Rules:**

1. Choose `element` for simple routes and `loader` + `useLoaderData` for data-aware routes.
2. Use `lazy` to defer route code until navigation.
3. Use `HydrateFallback` (React Router) or `loading.tsx` (Next.js) for pending UI during data loading.
4. Handle loader errors with `errorElement` or `error.tsx`.
5. Combine with Suspense for streaming and progressive rendering.

**Constraints and Limitations:**

- Loaders run before rendering; slow loaders delay the route.
- `lazy` adds a network round trip on first navigation unless prefetched.
- Data-aware rendering couples routes to the router's data APIs.

### Annotated Code Examples

```tsx
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

**Expected Output:** The router runs `loader` before rendering, so `UserDetail` receives data immediately and does not render a loading state.

**Why this output:** `loader` fetches data for the matched route. `useLoaderData` provides the result to the component after the loader resolves.

### Real-World Cases

- **Data-heavy routes:** Dashboards and detail pages that need data before render.
- **Code-split routes:** Large admin sections loaded on demand.
- **Streaming pages:** Suspense boundaries with server-rendered content.

### References

- React Router — useLoaderData — https://reactrouter.com/api/hooks/useLoaderData
- React Router — Lazy — https://reactrouter.com/api/data-routers/lazy
- TanStack Router — Data Loading — https://tanstack.com/router/latest/docs/framework/react/guide/data-loading

---

## 14. Route Performance

### Definitions

**Core Definition:** Route performance is the speed and smoothness with which routes load, render, and become interactive.

**Technical Definition:** Route performance is shaped by **code splitting** (route-level chunks), **prefetching** (loading route code or data on link hover or viewport visibility), **data loading waterfalls** (sequential loaders that could be parallel), **render cost** (route-level re-renders), and **navigation latency** (network + parse + execute). Route-level code splitting is the highest-impact optimisation for SPAs; prefetching on intent recovers the perceived latency of lazy loading. In data routers, parallel loaders and `defer`/streaming reduce time-to-content.

**Beginner-Friendly Explanation:** Route performance is about making page changes feel instant. The biggest wins are loading only the code for the page you are on, and starting to load the next page before the user clicks.

### Purposes

- To reduce the initial bundle by splitting per route.
- To make navigations feel instant through prefetching.
- To avoid sequential data waterfalls in loaders.
- To keep route transitions smooth and responsive.
- To measure and budget route-level performance.

### Syntax Rules and Structure

**Route-level code splitting:**

```jsx
const Dashboard = lazy(() => import('./pages/Dashboard'));
<Suspense fallback={<PageSkeleton />}>
  <Dashboard />
</Suspense>
```

**Router-native lazy:**

```jsx
{ path: '/reports', lazy: () => import('./pages/Reports') }
```

**Prefetch on intent:**

```jsx
<Link to="/dashboard" onMouseEnter={() => import('./pages/Dashboard')}>
  Dashboard
</Link>
```

**Syntax Rules:**

1. Split code at route boundaries; each route becomes its own chunk.
2. Prefetch route chunks on hover, focus, or viewport visibility.
3. Run independent loaders in parallel (`Promise.all`) instead of sequentially.
4. Use `defer`/streaming for non-critical data.
5. Measure navigation timing and route bundle sizes with the Performance panel and bundle analyser.

**Constraints and Limitations:**

- Too many tiny route chunks hurt compression and increase request overhead.
- Prefetching too aggressively wastes bandwidth.
- Sequential loaders create waterfalls; parallelise where possible.
- Lazy loading adds a round trip on first navigation.

### Annotated Code Examples

```jsx
import { lazy, Suspense } from 'react';
import { Link, Routes, Route } from 'react-router';

const Dashboard = lazy(() => import('./pages/Dashboard'));
const Reports = lazy(() => import('./pages/Reports'));

function App() {
  return (
    <>
      <nav>
        <Link to="/dashboard" onMouseEnter={() => import('./pages/Dashboard')}>
          Dashboard
        </Link>
        <Link to="/reports" onMouseEnter={() => import('./pages/Reports')}>
          Reports
        </Link>
      </nav>
      <Suspense fallback={<PageSkeleton />}>
        <Routes>
          <Route path="/dashboard" element={<Dashboard />} />
          <Route path="/reports" element={<Reports />} />
        </Routes>
      </Suspense>
    </>
  );
}
```

**Expected Output:** Hovering a link prefetches its chunk; clicking navigates without a visible loading delay. The initial bundle excludes both page chunks.

**Why this output:** `lazy` creates a split point per route. `onMouseEnter` triggers the dynamic import before the click. `Suspense` provides a fallback for the first navigation if prefetching has not completed.

### Real-World Cases

- **Large SPAs:** Route-level splitting keeps the initial bundle small.
- **Dashboards:** Prefetching the most-used routes on hover.
- **E-commerce:** Parallel loaders for product and recommendation data.
- **Content sites:** Prefetching article chunks on link visibility.

### References

- React Router — Lazy — https://reactrouter.com/api/data-routers/lazy
- TanStack Router — Preloading — https://tanstack.com/router/latest/docs/framework/react/guide/preloading
- Next.js — Linking and Navigating (prefetching) — https://nextjs.org/docs/app/building-your-application/routing/linking-and-navigating
- web.dev — Code splitting — https://web.dev/articles/reduce-javascript-payloads-with-code-splitting

---

## 15. Route Best Practices

### Definitions

**Core Definition:** Route best practices are the guidelines for designing URL structures, route hierarchies, data loading, and navigation behaviour that remain maintainable and performant as an application grows.

**Technical Definition:** Route best practices combine **URL design** (semantic, stable, shareable paths), **hierarchy design** (nested layouts that mirror the information architecture), **data strategy** (loaders, parallel fetching, caching), **performance strategy** (route-level splitting, prefetching, budgets), **accessibility** (focus management, route announcements, skip links), and **security** (server-enforced guards, no sensitive data in URLs). The choice of router (React Router, TanStack Router, Next.js App Router) should follow from the rendering model (SPA, SSR, RSC) rather than from familiarity alone.

**Beginner-Friendly Explanation:** Good routing makes an app easy to use, easy to share, and easy to maintain. URLs should make sense, pages should load fast, protected pages should stay protected, and screen-reader users should know when the page changes.

### Best Practices Table

| Practice | Description |
|----------|-------------|
| **Design semantic URLs** | `/products/:slug` rather than `/p?id=123`. |
| **Keep resource identity in the path** | Path params for resources; search params for view options. |
| **Use nested routes for shared layouts** | Parent layout + `<Outlet />` for child content. |
| **Split code per route** | Route-level `lazy` or `React.lazy` to keep the initial bundle small. |
| **Prefetch on intent** | Load route code or data on hover, focus, or visibility. |
| **Parallelise loaders** | Use `Promise.all` for independent data; avoid waterfalls. |
| **Enforce auth on the server** | Client guards are UX; server checks are security. |
| **Manage focus and scroll on navigation** | Move focus to main content and reset scroll. |
| **Provide a 404 route** | Catch-all route with a helpful not-found page. |
| **Handle loading and error states per route** | `HydrateFallback`/`loading.tsx` and `errorElement`/`error.tsx`. |
| **Avoid sensitive data in URLs** | URLs are logged, shared, and stored in history. |
| **Choose the router for the rendering model** | SPA → React Router/TanStack Router; SSR/RSC → framework router. |

### Annotated Code Examples

**✅ Good Practices:**

```tsx
import { lazy, Suspense } from 'react';
import { createBrowserRouter, RouterProvider, Outlet, redirect } from 'react-router';

const Dashboard = lazy(() => import('./pages/Dashboard'));

async function requireAuth() {
  const user = await getCurrentUser();
  if (!user) throw redirect('/login');
  return user;
}

const router = createBrowserRouter([
  {
    path: '/',
    element: <RootLayout />,
    errorElement: <RootError />,
    children: [
      { index: true, element: <Home /> },
      {
        path: 'dashboard',
        loader: requireAuth,
        element: (
          <Suspense fallback={<PageSkeleton />}>
            <Dashboard />
          </Suspense>
        ),
      },
      { path: '*', element: <NotFound /> },
    ],
  },
]);

function RootLayout() {
  return (
    <>
      <SkipLink />
      <Header />
      <main id="main" tabIndex={-1}>
        <Outlet />
      </main>
      <Footer />
    </>
  );
}
```

**❌ Bad Practices:**

```tsx
// ❌ Sensitive data in the URL
<Link to={`/account?token=${secretToken}`}>Account</Link>

// ❌ Client-only auth guard rendered after the page
function AdminPage() {
  const user = useUser();
  if (!user) return <p>Please log in</p>; // Page already rendered
  return <AdminPanel />;
}

// ❌ Sequential loaders causing a waterfall
const user = await fetchUser(id);
const posts = await fetchPosts(user.id);

// ❌ No 404 route
<Routes>{/* no catch-all */}</Routes>
```

**Expected Output:** The good practices produce secure, accessible, fast routes. The bad practices leak secrets, render protected content before checking access, create waterfalls, and leave users stranded on unknown URLs.

**Why this output:** URLs are not secret. Client-only guards render before the check and can be bypassed. Sequential awaits create waterfalls. A catch-all route prevents blank screens.

### Real-World Cases

- **All React applications:** These practices apply universally.
- **Multi-tenant SaaS:** Org-scoped paths with server-enforced access.
- **E-commerce:** Semantic product URLs and parallel loaders.
- **Content platforms:** Nested layouts, route-level splitting, and prefetching.

### References

- React Router — Picking a Mode — https://reactrouter.com/start/modes
- React Router — Route Objects — https://reactrouter.com/start/data/route-object
- TanStack Router — Official Documentation — https://tanstack.com/router
- Next.js — Routing Fundamentals — https://nextjs.org/docs/app/building-your-application/routing
- web.dev — Code splitting — https://web.dev/articles/reduce-javascript-payloads-with-code-splitting

---

## References

- React Router — Official Documentation — https://reactrouter.com
- React Router — Picking a Mode — https://reactrouter.com/start/modes
- React Router — Route Objects — https://reactrouter.com/start/data/route-object
- React Router — useParams — https://reactrouter.com/api/hooks/useParams
- React Router — useSearchParams — https://reactrouter.com/api/hooks/useSearchParams
- React Router — Outlet — https://reactrouter.com/api/components/Outlet
- React Router — useLoaderData — https://reactrouter.com/api/hooks/useLoaderData
- React Router — useNavigation — https://reactrouter.com/api/hooks/useNavigation
- React Router — redirect — https://reactrouter.com/api/data-routers/redirect
- React Router — Lazy — https://reactrouter.com/api/data-routers/lazy
- React Router — createHashRouter — https://reactrouter.com/api/data-routers/createHashRouter
- TanStack Router — Official Documentation — https://tanstack.com/router
- TanStack Router — Route Trees & Nesting — https://tanstack.com/router/latest/docs/framework/react/guide/route-trees
- TanStack Router — Path Params — https://tanstack.com/router/latest/docs/framework/react/guide/path-params
- TanStack Router — Search Params — https://tanstack.com/router/latest/docs/framework/react/guide/search-params
- TanStack Router — Authenticated Routes — https://tanstack.com/router/latest/docs/framework/react/guide/authenticated-routes
- TanStack Router — Data Loading — https://tanstack.com/router/latest/docs/framework/react/guide/data-loading
- TanStack Router — Preloading — https://tanstack.com/router/latest/docs/framework/react/guide/preloading
- TanStack Router — Pending Matches — https://tanstack.com/router/latest/docs/framework/react/guide/pending-matches
- TanStack Router — History Types — https://tanstack.com/router/latest/docs/framework/react/guide/history-types
- Next.js — Routing Fundamentals — https://nextjs.org/docs/app/building-your-application/routing
- Next.js — Linking and Navigating — https://nextjs.org/docs/app/building-your-application/routing/linking-and-navigating
- Next.js — Rendering — https://nextjs.org/docs/app/building-your-application/rendering
- Next.js — Middleware — https://nextjs.org/docs/app/building-your-application/routing/middleware
- MDN — History API — https://developer.mozilla.org/en-US/docs/Web/API/History_API
- MDN — Working with the History API — https://developer.mozilla.org/en-US/docs/Web/API/History_API/Working_with_the_History_API
- MDN — Window: popstate event — https://developer.mozilla.org/en-US/docs/Web/API/Window/popstate_event
- MDN — Window: hashchange event — https://developer.mozilla.org/en-US/docs/Web/API/Window/hashchange_event
- MDN — Navigation API — https://developer.mozilla.org/en-US/docs/Web/API/Navigation_API
- MDN — View Transition API — https://developer.mozilla.org/en-US/docs/Web/API/View_Transition_API
- MDN — SPA (Single-page application) — https://developer.mozilla.org/en-US/docs/Glossary/SPA
- web.dev — Code splitting — https://web.dev/articles/reduce-javascript-payloads-with-code-splitting