# Static & Hybrid Rendering Models: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Static and hybrid rendering models are strategies for generating HTML for React applications at build time, on a schedule, or at request time, with the goal of optimising Time to First Byte (TTFB), First Contentful Paint (FCP), and server cost by serving pre-rendered content from a CDN wherever possible.

**Technical Definition:** Static and hybrid rendering models sit on a spectrum between fully dynamic server rendering (SSR) and fully client-side rendering (CSR). **Static Site Generation (SSG)** produces HTML files at build time, which are deployed to a CDN and served without server compute. **Incremental Static Regeneration (ISR)** extends SSG by allowing individual pages to be regenerated in the background after a time interval or on-demand event, without rebuilding the entire site. **Partial Prerendering (PPR)** combines a static shell with dynamic, streamed holes within the same route, letting a single page serve cached static content instantly while streaming personalised or real-time content into place. **Caching primitives** — request memoization, `use cache`, `cacheLife`, `cacheTag`, `revalidateTag`, and `revalidatePath` — provide the framework-level mechanisms for controlling how long data is cached and when it should be invalidated. These models are implemented primarily in Next.js (App Router and Pages Router) and React Router v7 (Framework Mode pre-rendering).

**Beginner-Friendly Explanation:** When you build a React app, you need to decide how the HTML gets created. You could build it every time someone visits (SSR), or you could build it once and serve the same file to everyone (SSG). Static and hybrid models are about finding the sweet spot: build once where you can, regenerate when content changes, and stream dynamic bits into an otherwise static page. This makes your app fast (CDN-served HTML loads in milliseconds) and cheap (no server compute for most requests), while still supporting fresh and personalised content where needed.

### Key Characteristics

- **Build-Time by Default:** SSG generates HTML at build time, producing static files that can be served from a CDN at edge latency. This gives the fastest possible TTFB because no server compute is required per request.
- **Stale-While-Revalidate Semantics:** ISR follows the stale-while-revalidate pattern: visitors get a fast cached response, and the page is regenerated in the background based on a time interval or an API call.
- **Static Shell with Dynamic Holes:** PPR prerenders a static shell at build time and leaves dynamic holes wrapped in `<Suspense>` that stream in per request. The shell is served from the CDN, and dynamic content is rendered at the origin and streamed into the same response.
- **Layered Caching Primitives:** Next.js provides four caching layers: request memoization (per-request deduplication), Data Cache (persistent across requests), Full Route Cache (HTML and RSC payload), and Router Cache (client-side navigation). Each layer can be configured and revalidated independently.
- **Tag-Based Invalidation:** `cacheTag` and `revalidateTag` allow precise invalidation of cached data by tag, avoiding the over-invalidation caused by path-based revalidation. This is the recommended approach for on-demand revalidation.

### Prerequisites

- Solid understanding of React components, Hooks, and JSX.
- Familiarity with HTTP fundamentals (requests, responses, status codes, Cache-Control headers).
- Basic understanding of Server-Side Rendering (SSR) and client-side hydration.
- Experience with a modern React framework (Next.js App Router or React Router v7 Framework Mode).
- Awareness of CDN caching behaviour and cache invalidation concepts.

### Related Programming Areas

- **Server-Side Rendering (SSR):** Rendering on every request for fresh, personalised content.
- **Client-Side Rendering (CSR):** Rendering entirely in the browser; the baseline that SSG and SSR improve upon.
- **Streaming SSR:** Sending HTML in progressive chunks over a single HTTP connection.
- **React Server Components (RSC):** Components that run only on the server and ship no JavaScript.
- **CDN Caching:** Edge caching of static HTML, assets, and API responses.
- **Cache Invalidation:** Time-based and event-driven mechanisms for keeping cached content fresh.

### Core Concepts / Features

1. Static Site Generation (SSG)
2. Incremental Static Regeneration (ISR)
3. Partial Prerendering (PPR)
4. Caching & Core Primitives

---

## Core Concept 1: Static Site Generation (SSG)

### Definitions

**Core Definition:** Static Site Generation (SSG) is the process of generating the HTML for every route at build time, producing static files that are deployed to a CDN and served to users without any server-side computation per request.

**Technical Definition:** Static rendering produces the HTML for each accessible route at build time. Each URL maps to a file on disk that a CDN can serve from cache in tens of milliseconds, regardless of the user's location or concurrent request volume. Static rendering resolves the TTFB penalty of SSR (server compute time) and the FCP penalty of CSR (JavaScript download and execution). In Next.js, static generation with data is achieved via `getStaticProps` and `getStaticPaths` in the Pages Router, or via async Server Components and `generateStaticParams` in the App Router. In React Router v7, pre-rendering is configured with the `prerender` option in the Vite plugin config, which generates static HTML and `.data` files at build time. The output is a set of static files (`about.html`, `blog/my-post.html`, etc.) that can be served by any static file host or CDN.

**Beginner-Friendly Explanation:** SSG is like printing a book. Once the book is printed, every copy is identical and reading it is instant. When you build your site, the framework generates an HTML file for every page. Those files sit on a CDN, and when someone visits your site, the CDN hands them the pre-built file. There's no server thinking, no database querying, no delay. The trade-off is that if you change content, you have to "reprint the book" — rebuild the site.

### Purposes

- To achieve the fastest possible Time to First Byte (TTFB) by serving pre-built HTML from a CDN edge location.
- To eliminate server compute cost for routes that do not require per-request customisation.
- To improve First Contentful Paint (FCP) by delivering complete HTML without waiting for JavaScript execution.
- To provide high resiliency and availability: static files can be cached at multiple layers and survive origin outages.
- To reduce infrastructure complexity by removing the need for a running server for static content.
- To enable deployment to any static hosting provider (Vercel, Netlify, Cloudflare Pages, GitHub Pages, S3 + CloudFront).

### Syntax Rules and Structure

**Next.js App Router — Static Rendering (Default):**

```tsx
// app/about/page.tsx — Static by default (no dynamic APIs)
export default function AboutPage() {
  return (
    <div>
      <h1>About Us</h1>
      <p>This page is statically generated at build time.</p>
    </div>
  );
}
```

**Component Breakdown:**
- The page component is a Server Component by default.
- No dynamic APIs (`cookies`, `headers`, `searchParams`) are used, so Next.js prerenders the route at build time.
- The output is a static HTML file served from the CDN.

**Next.js App Router — Static Generation with Dynamic Routes:**

```tsx
// app/blog/[slug]/page.tsx
export async function generateStaticParams() {
  const posts = await fetch('https://api.example.com/posts').then((res) => res.json());
  return posts.map((post) => ({ slug: post.slug }));
}

export default async function BlogPost({ params }: { params: { slug: string } }) {
  const post = await fetch(`https://api.example.com/posts/${params.slug}`).then((res) => res.json());
  return (
    <article>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </article>
  );
}
```

**Component Breakdown:**
- `generateStaticParams()`: Returns an array of parameter objects for dynamic routes. Each object generates a static page at build time.
- The page component is an `async` Server Component that fetches data at build time.
- The output is a static HTML file for each slug (e.g., `blog/my-first-post.html`).

**React Router v7 — Pre-rendering Configuration:**

```typescript
// react-router.config.ts
import type { Config } from "@react-router/dev/config";

export default {
  // Pre-render all static route paths (no dynamic segments)
  prerender: true,
  // Or specify explicit URLs
  // prerender: ["/", "/blog", "/blog/popular-post"],
  // Or use an async function for CMS-driven paths
  // async prerender({ getStaticPaths }) {
  //   const posts = await fakeGetPostsFromCMS();
  //   return ["/", "/blog"].concat(posts.map((post) => post.href));
  // },
} satisfies Config;
```

**Component Breakdown:**
- `prerender: true`: Pre-renders all routes that have no dynamic segments.
- `prerender: [...]`: Pre-renders the specified URLs.
- `async prerender({ getStaticPaths })`: Uses a function to determine which paths to pre-render, useful for CMS-driven content.
- Pre-rendering uses the same route `loader` functions as server rendering. The build creates a `new Request()` and runs it through the app, generating static HTML and `.data` files in the `build/client` directory.

**Syntax Rules:**
- In Next.js App Router, routes are statically rendered by default unless they use dynamic APIs (`cookies`, `headers`, `searchParams`, `connection`, `draftMode`, `unstable_noStore`, or `fetch` with `cache: 'no-store'`).
- Use `generateStaticParams` to pre-render dynamic routes at build time.
- In React Router v7, add the `prerender` option to `react-router.config.ts` and run `react-router build` to generate static files.
- Pre-rendered output is written to the `build/client` directory.
- Static files can be served by any CDN or static file host.

**Constraints and Limitations:**
- SSG is not suitable for personalised content or data that changes in real time.
- Build times grow with the number of pages; large sites (millions of pages) require ISR or on-demand generation.
- `generateStaticParams` must return all possible parameter values at build time; dynamic routes not included are 404 (unless `dynamicParams` is enabled).
- React Router v7 pre-rendering does not support SPA fallback in all configurations; check the `prerender` documentation for edge cases.
- A known vulnerability (GHSA-f22v-gfqf-p8f3) in React Router v7 pre-rendering involves improper neutralisation of the `Location` header value when the redirect location comes from an untrusted source.

### Annotated Code Examples

**Example 1: Complete SSG Blog with Next.js App Router**

```tsx
// app/blog/page.tsx — Static blog listing
import Link from 'next/link';

export default async function BlogIndex() {
  const posts = await fetch('https://api.example.com/posts', {
    cache: 'force-cache', // Static by default at build time
  }).then((res) => res.json());

  return (
    <div>
      <h1>Blog</h1>
      <ul>
        {posts.map((post) => (
          <li key={post.slug}>
            <Link href={`/blog/${post.slug}`}>{post.title}</Link>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

```tsx
// app/blog/[slug]/page.tsx — Static blog post
export async function generateStaticParams() {
  const posts = await fetch('https://api.example.com/posts').then((res) => res.json());
  return posts.map((post) => ({ slug: post.slug }));
}

export default async function BlogPost({ params }: { params: { slug: string } }) {
  const post = await fetch(`https://api.example.com/posts/${params.slug}`).then((res) => res.json());

  return (
    <article>
      <h1>{post.title}</h1>
      <time>{post.publishedAt}</time>
      <div dangerouslySetInnerHTML={{ __html: post.content }} />
    </article>
  );
}
```

**Expected Output:** During `next build`, Next.js fetches all posts, generates a static HTML file for the blog index and for each post slug. The output is a set of static files in the `.next` directory, deployable to a CDN. When a user visits `/blog/my-first-post`, the CDN serves the pre-built HTML in milliseconds.

**Why This Output Occurs:** The `generateStaticParams` function returns all post slugs, so Next.js knows which dynamic routes to pre-render at build time. The `fetch` calls use the default `force-cache`, so data is fetched once at build time and embedded in the static HTML. No server compute is required at request time.

### Real-World Cases

- **Marketing sites:** Homepages, landing pages, and about pages that change infrequently.
- **Blogs and documentation:** Article pages, docs pages, and changelogs.
- **E-commerce product listings:** Category pages and product pages where data changes on a known schedule.
- **Portfolio sites:** Personal websites with project showcases.
- **Static content behind authentication:** Public documentation that does not require per-user customisation.

---

## Core Concept 2: Incremental Static Regeneration (ISR)

### Definitions

**Core Definition:** Incremental Static Regeneration (ISR) is a caching strategy that combines the speed of static content with the flexibility of server-side rendering, allowing individual pages to be regenerated in the background after a time interval or on-demand event without rebuilding the entire site.

**Technical Definition:** ISR follows the stale-while-revalidate pattern: visitors receive a fast cached response, and the page is regenerated in the background based on a time interval (`revalidate`) or an API call (`revalidateTag`, `revalidatePath`, `res.revalidate`). In Next.js Pages Router, ISR is enabled by returning `revalidate` from `getStaticProps`. In the App Router, ISR is enabled by exporting a `revalidate` route segment config value or by using `cacheLife` with Cache Components. On-demand revalidation allows precise invalidation of cached content without waiting for a time interval. Tag-based revalidation (`revalidateTag`) invalidates all cached entries associated with a tag, while path-based revalidation (`revalidatePath`) invalidates a specific route. Vercel's CDN adds automatic request collapsing (multiple requests for the same uncached path collapse into one function invocation), durable storage (31-day cache retention), and globally consistent purging (all regions update within 300ms).

**Beginner-Friendly Explanation:** ISR is like a newspaper that updates itself. You print the first edition, but instead of throwing it away when news changes, you keep serving the old edition to readers while you quietly print a new one in the background. After a set time (say, 60 seconds), the next reader gets the new edition. If something urgent happens, you can trigger a reprint immediately by calling an API or revalidating a tag. Your site stays fast because readers always get a cached page, but the content doesn't go stale.

### Purposes

- To update static content without rebuilding the entire site, saving build time and resources.
- To serve pre-rendered, static pages for most requests while keeping content reasonably fresh.
- To reduce server load by serving cached content and regenerating only when necessary.
- To scale to millions of pages without long build times by pre-rendering popular pages at build time and generating the rest on demand.
- To enable on-demand revalidation triggered by CMS webhooks, database mutations, or API calls.
- To provide globally consistent content updates with minimal latency.

### Syntax Rules and Structure

**Next.js Pages Router — Time-Based ISR:**

```typescript
// pages/blog/[id].tsx
import type { GetStaticPaths, GetStaticProps } from 'next';

export const getStaticPaths: GetStaticPaths = async () => {
  const posts = await fetch('https://api.vercel.app/blog').then((res) => res.json());
  return {
    paths: posts.map((post) => ({ params: { id: String(post.id) } })),
    fallback: 'blocking',
  };
};

export const getStaticProps: GetStaticProps = async ({ params }) => {
  const post = await fetch(`https://api.vercel.app/blog/${params.id}`).then((res) => res.json());
  return {
    props: { post },
    revalidate: 60, // Regenerate at most once every 60 seconds
  };
};

export default function Page({ post }: { post: { title: string; content: string } }) {
  return (
    <main>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </main>
  );
}
```

**Component Breakdown:**
- `fallback: 'blocking'`: Requests for paths not generated at build time are server-rendered on demand and then cached.
- `revalidate: 60`: After 60 seconds, the next request returns the stale page and triggers regeneration in the background.
- The updated page is served to subsequent requests once generation completes.

**Next.js App Router — Time-Based ISR with Cache Components:**

```tsx
// app/lib/data.ts
import { cacheLife } from 'next/cache';

export async function getProducts() {
  'use cache';
  cacheLife('hours'); // Revalidate every hour
  return db.query('SELECT * FROM products');
}
```

**Component Breakdown:**
- `'use cache'`: Marks the function's return value as cacheable.
- `cacheLife('hours')`: Uses the built-in `hours` profile (stale: 5m, revalidate: 1h, expire: 1d).
- Custom profiles can be passed as an object: `cacheLife({ stale: 3600, revalidate: 7200, expire: 86400 })`.

**Next.js — On-Demand Revalidation:**

```typescript
// app/api/revalidate/route.ts
import { revalidatePath, revalidateTag } from 'next/cache';

export async function POST(request: Request) {
  const { path, tag } = await request.json();

  if (tag) {
    revalidateTag(tag); // Invalidate all cache entries with this tag
  }
  if (path) {
    revalidatePath(path); // Invalidate a specific route
  }

  return Response.json({ revalidated: true });
}
```

**Component Breakdown:**
- `revalidateTag(tag)`: Invalidates all cached entries associated with the tag using stale-while-revalidate semantics.
- `revalidatePath(path)`: Invalidates a specific route's HTML and RSC payload cache.
- This route handler can be called from a CMS webhook or a backend mutation.

**React Router v7 — ISR-Style Caching with Headers:**

React Router v7 does not have built-in ISR; caching is manual. You can export a `headers` function from a route module to set `Cache-Control` headers:

```typescript
// app/routes/blog.tsx
export function headers() {
  return {
    'Cache-Control': 'public, s-maxage=300, stale-while-revalidate=60',
  };
}
```

**Component Breakdown:**
- `s-maxage=300`: The CDN caches the response for 300 seconds.
- `stale-while-revalidate=60`: After 300 seconds, the CDN serves the stale response for up to 60 seconds while revalidating in the background.
- This achieves ISR-like behaviour at the CDN level.

**Syntax Rules:**
- Use `revalidate: N` in `getStaticProps` (Pages Router) or `cacheLife('hours')` (App Router) for time-based ISR.
- Use `revalidateTag(tag)` for precise, tag-based invalidation across all routes using that tag.
- Use `revalidatePath(path)` for route-specific invalidation.
- Prefer tag-based revalidation over path-based when possible — it is more precise and avoids over-invalidating.
- In React Router v7, use `Cache-Control` headers with `stale-while-revalidate` for CDN-level ISR-like behaviour.
- Vercel automatically adds `Cache-Control` headers and manages ISR cache storage when deploying Next.js.

**Constraints and Limitations:**
- ISR is not suitable for real-time data (use SSR or streaming for that).
- On Vercel, the ISR cache is scoped to a specific deployment; each new deployment starts with a fresh cache and does not reuse the previous deployment's cache.
- `revalidatePath` resets the route HTML/RSC cache only; the fetch Data Cache is separate and must be revalidated with `revalidateTag`.
- Time-based revalidation means some users may see stale content until the background regeneration completes.
- ISR is not available in React Router v7 out of the box; you must implement it manually with CDN cache headers.

### Annotated Code Examples

**Example 1: E-Commerce Product Catalog with ISR**

```tsx
// app/products/[slug]/page.tsx
import { cacheTag, cacheLife } from 'next/cache';

export async function generateStaticParams() {
  const products = await db.product.findMany({ select: { slug: true } });
  return products.map((p) => ({ slug: p.slug }));
}

export default async function ProductPage({ params }: { params: { slug: string } }) {
  const product = await getProduct(params.slug);
  return (
    <div>
      <h1>{product.name}</h1>
      <p>${product.price}</p>
    </div>
  );
}

async function getProduct(slug: string) {
  'use cache';
  cacheLife('hours');
  cacheTag(`product-${slug}`);
  return db.product.findUnique({ where: { slug } });
}
```

```typescript
// app/api/revalidate-product/route.ts
import { revalidateTag } from 'next/cache';

export async function POST(request: Request) {
  const { slug } = await request.json();
  revalidateTag(`product-${slug}`); // Invalidate only this product's cache
  return Response.json({ revalidated: true });
}
```

**Expected Output:** The product page is statically generated at build time for all product slugs. It is cached for up to 1 hour. When a product is updated in the CMS, the backend calls `/api/revalidate-product` with the slug, which calls `revalidateTag('product-xyz')`. The next visitor to that product page gets the fresh content, while other product pages remain cached.

**Why This Output Occurs:** The `cacheTag` function associates the cached product data with a unique tag. When `revalidateTag` is called, Next.js invalidates only the cache entries matching that tag, leaving other product pages untouched. This is the recommended precision-invalidation pattern.

### Real-World Cases

- **E-commerce:** Large product catalogs that need current pricing and availability without rebuilding the entire site.
- **Media and publishing:** Content pages that update when authors publish in a headless CMS.
- **Generative AI platforms:** Pages generated from discrete events like git syncs or API updates.
- **Documentation sites:** API reference pages that update when the underlying API changes.
- **Real estate listings:** Property listings that are added and updated regularly but do not need real-time freshness.

---

## Core Concept 3: Partial Prerendering (PPR)

### Definitions

**Core Definition:** Partial Prerendering (PPR) is a rendering strategy that combines a static shell prerendered at build time with dynamic, streamed holes within the same route, allowing a single page to serve cached static content instantly while streaming personalised or real-time content into place.

**Technical Definition:** PPR combines static and dynamic rendering in a single route. At build time, Next.js generates a **static HTML shell** containing all content that can be prerendered, with Suspense fallbacks where dynamic content will appear. It also generates a **postponedState blob** — a serialized string that the server uses to resume rendering the dynamic portions at request time. At request time, the static shell is served immediately (from the CDN edge in optimised deployments), and dynamic content is rendered at the origin and streamed to the client into the dynamic holes. The dynamic holes are marked by `<Suspense>` boundaries; wrapping a component in Suspense does not make it dynamic, but using dynamic APIs (cookies, headers, searchParams, etc.) inside a Suspense boundary causes that boundary to be postponed. PPR is built into the Cache Components model in Next.js 16 (enabled via `cacheComponents: true`) and replaces the experimental `ppr` flag from Next.js 15.

**Beginner-Friendly Explanation:** PPR is like a picture frame with removable panels. The frame itself (the static shell) is built once and hung on the wall. Some panels are filled in with permanent content (header, footer, product info). Other panels are empty holes where you can slide in different pictures (the user's cart, personalised recommendations, live pricing). When someone visits the page, they see the frame and the permanent content instantly, and the personalised panels slide in a moment later. This means the page loads fast for everyone, but still shows personal stuff where it matters.

### Purposes

- To improve initial page performance by serving a static shell instantly from the CDN.
- To support personalised, dynamic data within the same route without sacrificing static performance.
- To eliminate client-to-server waterfalls by streaming dynamic components in parallel while serving the initial prerender.
- To reduce TTFB by serving the static shell at edge latency rather than origin latency.
- To combine the benefits of SSG (fast, cacheable shell) and SSR (personalised, fresh dynamic content) in a single response.
- To enable a smooth migration path from fully static to fully dynamic rendering on a per-component basis.

### Syntax Rules and Structure

**Next.js 16 — Enabling PPR via Cache Components:**

```typescript
// next.config.ts
import type { NextConfig } from 'next';

const nextConfig: NextConfig = {
  cacheComponents: true, // Enables PPR as the default behaviour
};

export default nextConfig;
```

**Component Breakdown:**
- `cacheComponents: true`: Enables the Cache Components model, which implements PPR as the default behaviour in the App Router.
- Without `cacheComponents`, PPR was experimental and enabled via `experimental.ppr` in Next.js 15.

**General Syntax for a PPR Route:**

```tsx
// app/product/[id]/page.tsx
import { Suspense } from 'react';

export default async function ProductPage({ params }: { params: { id: string } }) {
  const product = await getProduct(params.id); // Static at build time

  return (
    <div>
      {/* Static shell: product info, images, description */}
      <ProductInfo product={product} />
      <ProductImages images={product.images} />

      {/* Dynamic hole: personalised recommendations */}
      <Suspense fallback={<RecommendationsSkeleton />}>
        <Recommendations productId={params.id} />
      </Suspense>

      {/* Dynamic hole: user's cart summary */}
      <Suspense fallback={<CartSkeleton />}>
        <CartSummary />
      </Suspense>

      {/* Dynamic hole: live pricing */}
      <Suspense fallback={<PriceSkeleton />}>
        <LivePrice productId={params.id} />
      </Suspense>
    </div>
  );
}
```

**Component Breakdown:**
- `ProductInfo` and `ProductImages` are part of the static shell — they use only build-time data and no dynamic APIs.
- `<Suspense>` boundaries mark dynamic holes. Components inside them may use `cookies`, `headers`, `searchParams`, or `fetch` with `cache: 'no-store'`.
- At build time, Next.js prerenders the shell and the Suspense fallbacks. The dynamic content is postponed.
- At request time, the shell is served immediately, and the dynamic holes are streamed in parallel.

**Static Shell Prerendering Behaviour:**

| Component Type | Build Time | Request Time |
|----------------|------------|--------------|
| Static (no dynamic APIs) | Prerendered into shell | Served from CDN |
| Dynamic (uses `cookies`, `headers`, etc.) inside Suspense | Fallback prerendered | Rendered and streamed |
| Dynamic outside Suspense | Build error | — |

**Syntax Rules:**
- Enable PPR by setting `cacheComponents: true` in `next.config.ts` (Next.js 16+).
- Wrap dynamic components in `<Suspense>` boundaries to mark them as dynamic holes.
- Do not use dynamic APIs (`cookies`, `headers`, `searchParams`, `connection`, `draftMode`, `unstable_noStore`) outside a Suspense boundary — this causes a build error.
- The static shell is served immediately; dynamic content is streamed in parallel.
- The `postponedState` blob must be stored and updated atomically with the static shell. Serving a new shell with an old postponed state produces incorrect output.

**Constraints and Limitations:**
- PPR was experimental in Next.js 15 and is stable in Next.js 16 as part of Cache Components.
- PPR requires platform support for storing the static shell and postponed state together, or streaming the shell and dynamic content in a single response.
- Using dynamic APIs outside a Suspense boundary causes a build error.
- `postponedState` is an opaque value; altering it produces incorrect dynamic rendering output.
- CDN shell caching requires the CDN to support combining cached and dynamic content in a single streaming response.
- Short-lived caches (seconds profile, `revalidate: 0`, or `expire` under 5 minutes) are automatically excluded from prerenders and become dynamic holes instead.

### Annotated Code Examples

**Example 1: E-Commerce Product Page with PPR**

```tsx
// app/product/[id]/page.tsx
import { Suspense } from 'react';
import { cookies } from 'next/headers';

async function getProduct(id: string) {
  'use cache';
  return db.product.findUnique({ where: { id } });
}

async function getRecommendations(productId: string) {
  const session = cookies().get('session')?.value;
  return fetch(`/api/recommendations?product=${productId}&session=${session}`).then((r) => r.json());
}

export default async function ProductPage({ params }: { params: { id: string } }) {
  const product = await getProduct(params.id);

  return (
    <div className="product-page">
      {/* Static shell */}
      <header>
        <h1>{product.name}</h1>
        <p>{product.description}</p>
        <img src={product.image} alt={product.name} />
      </header>

      {/* Dynamic hole: personalised recommendations */}
      <Suspense fallback={<div className="h-48 animate-pulse bg-gray-100" />}>
        <Recommendations productId={params.id} />
      </Suspense>

      {/* Dynamic hole: user cart */}
      <Suspense fallback={<div className="h-24 animate-pulse bg-gray-50" />}>
        <CartSummary />
      </Suspense>
    </div>
  );
}

async function Recommendations({ productId }: { productId: string }) {
  const recs = await getRecommendations(productId);
  return <ul>{recs.map((r) => <li key={r.id}>{r.name}</li>)}</ul>;
}
```

**Expected Output:** The product header (name, description, image) is part of the static shell and is served immediately from the CDN. The recommendations and cart sections are dynamic holes that stream in per request. The user sees the product info instantly and the personalised sections a moment later.

**Why This Output Occurs:** `getProduct` uses `'use cache'`, so it is prerendered into the static shell. `getRecommendations` uses `cookies()`, which is a dynamic API, so it is wrapped in `<Suspense>` and becomes a dynamic hole. At request time, the shell is served immediately, and the recommendations are rendered at the origin and streamed into the hole.

### Real-World Cases

- **E-commerce:** Product pages with static product info and dynamic cart, recommendations, and live pricing.
- **SaaS dashboards:** Static dashboard shell with dynamic user-specific widgets (notifications, recent activity, usage metrics).
- **Media sites:** Static article content with dynamic comments, related articles, and personalised ads.
- **Travel platforms:** Static destination pages with dynamic availability, pricing, and user reviews.
- **Financial dashboards:** Static portfolio summary with dynamic live prices and transaction feeds.

---

## Core Concept 4: Caching & Core Primitives

### Definitions

**Core Definition:** Caching and core primitives are the framework-level mechanisms — request memoization, `use cache`, `cacheLife`, `cacheTag`, `revalidateTag`, and `revalidatePath` — that control how long data is cached, how it is invalidated, and how duplicate requests are deduplicated.

**Technical Definition:** Next.js provides four caching layers: **Request Memoization** (React feature that deduplicates `fetch` calls with the same URL and options during a single server render pass); **Data Cache** (persistent cache for `fetch` responses and `'use cache'` function outputs across requests and deployments); **Full Route Cache** (cached HTML and RSC payload for statically rendered routes); and **Router Cache** (client-side cache of RSC payloads for navigation). The `use cache` directive marks a function, component, or file as cacheable and requires the function to be `async`. `cacheLife` controls cache lifetime using profiles (`seconds`, `minutes`, `hours`, `days`, `weeks`, `max`) or a custom object. `cacheTag` associates cached data with a tag for precise on-demand invalidation. `revalidateTag` invalidates all cache entries with a matching tag using stale-while-revalidate semantics. `revalidatePath` invalidates a specific route's HTML and RSC payload cache. Request memoization is a React feature (not Next.js) and applies only within the React component tree, not in Route Handlers.

**Beginner-Friendly Explanation:** Caching primitives are the knobs and dials that control how your app remembers things. Request memoization is automatic: if you ask for the same data twice in one render, React only fetches it once. The Data Cache is like a filing cabinet: `'use cache'` puts a function's result in the cabinet, `cacheLife` says how long it stays fresh, and `cacheTag` puts a label on it. When something changes, `revalidateTag` pulls all the files with that label, and `revalidatePath` pulls a specific page. This lets you keep pages fast without serving stale content.

### Purposes

- To deduplicate identical `fetch` requests during a single server render pass, reducing network overhead.
- To persist cached data across requests and deployments so expensive computations and API calls are not repeated.
- To control cache lifetime with configurable profiles that balance freshness against performance.
- To enable precise on-demand invalidation by tagging cached data and revalidating by tag.
- To invalidate specific routes when a single page's content changes, without affecting other routes.
- To provide a clear, layered caching model that developers can reason about and configure.

### Syntax Rules and Structure

**Request Memoization (Automatic):**

```tsx
// app/example.tsx
async function getItem() {
  // fetch is automatically memoized: only executed once per render pass
  const res = await fetch('https://api.example.com/item/1');
  return res.json();
}

// Both calls return the same cached result
const item1 = await getItem(); // cache MISS
const item2 = await getItem(); // cache HIT
```

**Component Breakdown:**
- `fetch` calls with the same URL and options are automatically memoized during a single React render pass.
- This is a React feature, not a Next.js feature.
- Memoization applies only within the React component tree, not in Route Handlers.

**`use cache` Directive:**

```tsx
// Function level
export async function getProducts() {
  'use cache';
  return db.query('SELECT * FROM products');
}

// Component level
export async function ProductList() {
  'use cache';
  const products = await getProducts();
  return <ul>{products.map((p) => <li key={p.id}>{p.name}</li>)}</ul>;
}

// File level (all exports cached)
'use cache';
export default async function Page() {
  // ...
}
```

**Component Breakdown:**
- `'use cache'`: Marks the function's return value as cacheable.
- Functions and components using `'use cache'` must be `async`.
- At file level, every exported function becomes a cached function.
- Cache keys include build ID, function ID, serializable arguments, and HMR refresh hash (development).

**`cacheLife` — Time-Based Revalidation:**

```tsx
import { cacheLife } from 'next/cache';

export async function getProducts() {
  'use cache';
  cacheLife('hours'); // Built-in profile
  return db.query('SELECT * FROM products');
}

// Custom configuration
export async function getAnalytics() {
  'use cache';
  cacheLife({
    stale: 3600,     // 1 hour until considered stale
    revalidate: 7200, // 2 hours until revalidated
    expire: 86400,    // 1 day until expired
  });
  return db.query('SELECT * FROM analytics');
}
```

**Component Breakdown:**
- `cacheLife('hours')`: Uses the `hours` profile (stale: 5m, revalidate: 1h, expire: 1d).
- Custom object: Fine-grained control over stale, revalidate, and expire durations.
- Short-lived caches (`seconds` profile, `revalidate: 0`, or `expire` under 5 minutes) are excluded from prerenders.

**`cacheTag` and `revalidateTag` — Tag-Based Invalidation:**

```tsx
import { cacheTag, revalidateTag } from 'next/cache';

export async function getProduct(id: string) {
  'use cache';
  cacheTag(`product-${id}`);
  return db.product.findUnique({ where: { id } });
}

// Invalidate all cache entries with this tag
revalidateTag('product-123');
```

**Component Breakdown:**
- `cacheTag(tag)`: Associates cached data with a tag.
- `revalidateTag(tag)`: Invalidates all cache entries with the matching tag using stale-while-revalidate semantics.
- Tags are strings; max length is 256 characters, max items per fetch is 128.

**`revalidatePath` — Path-Based Invalidation:**

```tsx
import { revalidatePath } from 'next/cache';

// Invalidate a specific route
revalidatePath('/blog/my-post');

// Invalidate all routes under a path
revalidatePath('/blog', 'layout');

// Invalidate all routes
revalidatePath('/', 'layout');
```

**Component Breakdown:**
- `revalidatePath(path)`: Invalidates a specific route's HTML and RSC payload cache.
- Second argument (`'page'` or `'layout'`) controls the scope of invalidation.
- `revalidatePath` does not invalidate the fetch Data Cache; use `revalidateTag` for that.

**Syntax Rules:**
- Request memoization is automatic for `fetch` with the same URL and options.
- Use `'use cache'` to mark a function, component, or file as cacheable.
- Use `cacheLife` for time-based revalidation and `cacheTag` for on-demand invalidation.
- Prefer tag-based revalidation over path-based when possible — it is more precise.
- `revalidateTag` uses stale-while-revalidate semantics: stale content is served immediately while fresh content loads in the background.
- `revalidatePath` invalidates route HTML and RSC payload; the fetch Data Cache is separate.

**Constraints and Limitations:**
- Request memoization does not apply to Route Handlers (they are not part of the React component tree).
- `use cache` requires `cacheComponents: true` in Next.js 16+; in Next.js 15, it is experimental.
- `revalidatePath` invalidates only the route cache, not the fetch Data Cache.
- `revalidateTag` and `revalidatePath` do not automatically purge CDN caches; platforms like Vercel handle this automatically, but custom CDNs may require manual purge API calls.
- Cache entries from previous deployments are not reused by new deployments on Vercel; each deployment has its own ISR cache.

### Annotated Code Examples

**Example 1: Complete Caching Workflow with Tag-Based Invalidation**

```tsx
// app/lib/data.ts
import { cacheTag, cacheLife } from 'next/cache';

export async function getPost(slug: string) {
  'use cache';
  cacheLife('days');
  cacheTag(`post-${slug}`);
  return db.post.findUnique({ where: { slug } });
}

export async function getPosts() {
  'use cache';
  cacheLife('hours');
  cacheTag('posts-list');
  return db.post.findMany();
}
```

```tsx
// app/blog/[slug]/page.tsx
export default async function BlogPost({ params }: { params: { slug: string } }) {
  const post = await getPost(params.slug);
  return (
    <article>
      <h1>{post.title}</h1>
      <div dangerouslySetInnerHTML={{ __html: post.content }} />
    </article>
  );
}
```

```typescript
// app/api/revalidate-post/route.ts
import { revalidateTag } from 'next/cache';

export async function POST(request: Request) {
  const { slug } = await request.json();
  revalidateTag(`post-${slug}`); // Invalidate this post's cache
  revalidateTag('posts-list');   // Invalidate the post list cache
  return Response.json({ revalidated: true });
}
```

**Expected Output:** The blog post page is statically generated and cached for days. When the post is updated in the CMS, the backend calls `/api/revalidate-post` with the slug. `revalidateTag('post-my-slug')` invalidates the specific post, and `revalidateTag('posts-list')` invalidates the post list. The next visitor to the post gets the fresh content, while other posts remain cached.

**Why This Output Occurs:** The `cacheTag` calls associate each cached function with tags. When `revalidateTag` is called with a tag, Next.js marks all cache entries matching that tag as stale and regenerates them in the background on the next request. This is the recommended precision-invalidation pattern for content-driven sites.

### Real-World Cases

- **CMS-driven sites:** Tagging posts, products, and pages by their IDs and revalidating on publish.
- **E-commerce:** Tagging products by category and revalidating category pages when products change.
- **Analytics dashboards:** Using `cacheLife` with short profiles for frequently updated metrics.
- **Documentation:** Tagging API references by version and revalidating when the API changes.
- **Multi-tenant platforms:** Tagging data by tenant ID and revalidating per tenant.

---

## References

- Static Rendering – Patterns.dev: https://github.com/PatternsDev/skills/blob/153ef33dc25f49015e1f3967d015a368bc8c3947/react/static-rendering/SKILL.md
- Incremental Static Regeneration (ISR) – Vercel: https://vercel.com/docs/incremental-static-regeneration
- Partial Prerendering – Next.js: https://nextjs.org/docs/15/app/getting-started/partial-prerendering.md
- Implementing Partial Prerendering on your platform – Next.js: https://nextjs.org/docs/app/guides/ppr-platform-guide
- Revalidating – Next.js: https://nextjs.org/docs/app/getting-started/revalidating
- Caching in Next.js – Next.js: https://nextjs.org/docs/15/app/guides/caching
- `use cache` Directive – Next.js: https://nextjs.org/docs/app/api-reference/directives/use-cache
- Pre-Rendering – React Router v7: https://reactrouter.com/7.0.1/how-to/pre-rendering
- How to implement Incremental Static Regeneration (ISR) – Next.js: https://nextjs.org/docs/pages/guides/incremental-static-regeneration
- `cacheComponents` – Next.js: https://nextjs.org/docs/app/api-reference/config/next-config-js/cacheComponents
- ISR with Cache Components – Next.js: https://nextjs.org/docs/app/guides/isr-with-cache-components
- How Revalidation Works – Next.js: https://nextjs.org/docs/app/guides/how-revalidation-works
- All Rendering and Caching Strategies on the Web – Medium: https://medium.com/@amirakbarpour86/all-rendering-and-caching-strategies-on-the-web-ssr-ssg-isr-swr-09f66ffc1693
- Next.js 16 Cache Components – Vercel Plugin Skills: https://github.com/vercel/vercel-plugin/blob/main/skills/next-cache-components/upstream/SKILL.md
- Next.js PPR Patterns – pproenca/dot-skills: https://github.com/pproenca/dot-skills/blob/main/skills/.experimental/nextjs-ppr-patterns/SKILL.md