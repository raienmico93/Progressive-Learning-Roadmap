# React Loading Performance — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** React loading performance is the practice of minimizing the time and resources required for a React application to fetch its assets, render meaningful content, and become usable for the user.

**Technical Definition:** Loading performance in React concerns the critical path from the initial HTTP request to the moment the largest visible content has rendered (Largest Contentful Paint) and the application is ready to accept input. It encompasses network transfer (TTFB), resource discovery and prioritization (preload, preconnect, fetchpriority), render-blocking resource elimination (critical CSS), deferred loading (lazy loading, code splitting), asset optimization (images, fonts), and caching strategies (service workers). React's architecture introduces specific considerations: client-side rendering can delay content until JavaScript executes, Server Components and streaming change the loading model, and resource preloading APIs from `react-dom` allow components to declare their resource dependencies declaratively.

**Beginner-Friendly Explanation:** Loading performance is about how fast your app shows something useful after a user opens it. Every file the browser downloads — HTML, CSS, JavaScript, images, fonts — adds time before the user sees content. Good loading performance means the important stuff appears quickly, the page does not jump around, and the app does not download code for pages the user may never visit.

### Key Characteristics

- **Critical path focus:** Only the resources needed for the first meaningful paint should block rendering.
- **Resource prioritization:** Browsers guess at importance; `fetchpriority`, `preload`, and `preconnect` remove the guesswork.
- **Progressive delivery:** Critical CSS is inlined, non-critical CSS is deferred, and JavaScript is code-split.
- **Asset discipline:** Images use modern formats, explicit dimensions, and appropriate lazy loading; fonts use `font-display` and metric-matched fallbacks.
- **Caching as acceleration:** Service workers turn repeat visits into near-instant loads by serving cached assets.
- **Metrics-driven:** LCP ≤ 2.5s, FCP ≤ 1.8s, CLS ≤ 0.1 are the targets.

### Prerequisites

- React function components and JSX.
- Understanding of the browser's critical rendering path (HTML → CSS → render-blocking resources).
- Familiarity with HTTP caching, resource hints, and the Network panel.
- Basic knowledge of Core Web Vitals (LCP, FCP, CLS).

### Related Programming Areas

- **Code Splitting** — reducing the initial JavaScript payload.
- **Bundle Optimization** — minification, compression, tree shaking.
- **Rendering Performance** — main-thread work after load.
- **Core Web Vitals** — the metrics that quantify loading success.

---

## 1. Initial Load

### Definitions

**Core Definition:** Initial load is the sequence of events from the first HTTP request to the moment the application becomes usable for the first time.

**Technical Definition:** Initial load encompasses DNS resolution, connection establishment (TCP/TLS), server response (TTFB), HTML parsing, CSS and JavaScript fetching and execution, and the first paint. In a React application, the initial load may involve server-rendered HTML (SSR/SSG) or a minimal HTML shell that requires JavaScript to render content (CSR). The initial load is measured by First Contentful Paint (FCP), Largest Contentful Paint (LCP), and Time to First Byte (TTFB).

**Beginner-Friendly Explanation:** Initial load is the very first experience a user has with your app — everything that happens from tapping a link to seeing something on screen.

### Purposes

- To deliver the first meaningful content as quickly as possible.
- To minimize the time the user stares at a blank screen.
- To establish the foundation for interaction readiness.
- To meet Core Web Vitals thresholds for LCP and FCP.

### Syntax Rules and Structure

**Initial Load Stages:**

| Stage | What happens | Optimization lever |
|-------|--------------|---------------------|
| DNS + connection | Resolve domain, establish TCP/TLS | `preconnect`, `dns-prefetch` |
| Server response | HTML generation and transfer | TTFB < 200ms, SSR/streaming |
| Resource discovery | Parse HTML, discover CSS/JS | Critical CSS inline, preload |
| Render-blocking | CSS and synchronous JS block paint | Defer non-critical JS, inline critical CSS |
| First paint | FCP, then LCP | Image optimization, font loading |

**Syntax Rules:**

1. Minimize render-blocking resources in `<head>`.
2. Inline critical CSS and defer the rest.
3. Defer non-critical JavaScript.
4. Preload the LCP image and critical fonts.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Preloading the LCP Image in React

```jsx
import { preload } from 'react-dom';

function Hero() {
  // Tell the browser to fetch the LCP image at high priority
  preload('/hero.avif', { as: 'image', fetchPriority: 'high' });

  return (
    <img
      src="/hero.avif"
      alt="Hero"
      width={1200}
      height={600}
      fetchpriority="high"
    />
  );
}
```

**Expected Output:** The hero image is fetched at high priority, improving LCP.

**Why this output:** `preload` from `react-dom` declares the resource before the browser discovers it naturally, and `fetchpriority="high"` overrides the browser's default priority.

#### Example 2: Deferring Non-Critical JavaScript

```jsx
import { lazy, Suspense } from 'react';

const Analytics = lazy(() => import('./Analytics'));

function App() {
  return (
    <>
      <MainContent />
      <Suspense fallback={null}>
        <Analytics />
      </Suspense>
    </>
  );
}
```

**Expected Output:** The analytics bundle does not block the initial load; it loads after the main content.

**Why this output:** `React.lazy` creates a separate chunk that is fetched only when the `Analytics` component renders. The `Suspense` fallback is `null`, so it does not affect the initial paint.

### Real-World Cases

- **Marketing landing pages:** LCP is typically the hero image or headline.
- **E-commerce product pages:** The product image and buy button must appear quickly.
- **SaaS dashboards:** The application shell and first data view determine perceived speed.

### References

- Steve Kinney — Core Web Vitals for React Applications — https://stevekinney.com/courses/react-performance/core-web-vitals-for-react
- Steve Kinney — Priority Hints & Resource Loading — https://stevekinney.com/courses/react-performance/priority-hints-resource-loading
- React — Resource Preloading APIs — https://18.react.dev/reference/react-dom

---

## 2. Above-the-Fold Content

### Definitions

**Core Definition:** Above-the-fold content is the portion of a page visible in the viewport without scrolling, and it should render as quickly as possible.

**Technical Definition:** Above-the-fold content is the content within the initial viewport. It is the content users see first, and it determines FCP and often LCP. Optimizing above-the-fold content means prioritizing its CSS, images, and fonts while deferring everything below the fold. In React, this often means identifying the hero section, inlining its critical styles, preloading its image, and ensuring its component does not depend on data that arrives too late.

**Beginner-Friendly Explanation:** Above-the-fold content is what you see when the page first opens, without scrolling. It needs to load first because it is the user's first impression.

### Purposes

- To deliver the first visual content as fast as possible.
- To improve FCP and LCP.
- To avoid blank screens and late-rendered hero sections.
- To guide the browser's resource prioritization.

### Syntax Rules and Structure

**Above-the-Fold Optimization Checklist:**

| Resource | Action |
|----------|--------|
| Hero image | `fetchpriority="high"`, `loading="eager"`, preload |
| Critical CSS | Inline in `<head>` |
| Critical fonts | Preload, `font-display: swap` |
| Below-fold images | `loading="lazy"` |
| Below-fold components | Lazy load or defer |

**Syntax Rules:**

1. Never lazy-load above-the-fold images.
2. Inline the CSS required for above-the-fold content.
3. Preload the hero image and primary font.
4. Defer below-the-fold JavaScript and images.

### Annotated Code Examples

```jsx
function AboveTheFoldHero() {
  return (
    <section className="hero">
      <img
        src="/hero.avif"
        alt="Hero"
        width={1200}
        height={600}
        fetchpriority="high"
        loading="eager"  {/* Never lazy-load above-the-fold images */}
      />
      <h1>Welcome</h1>
    </section>
  );
}
```

**Expected Output:** The hero image is fetched eagerly at high priority, improving LCP.

**Why this output:** `fetchpriority="high"` and `loading="eager"` ensure the browser does not defer the image. Explicit dimensions prevent layout shift.

### Real-World Cases

- **Landing pages:** The hero section must paint immediately.
- **News articles:** The headline and lead image determine first impression.
- **Product pages:** The product image and price must appear before the fold.

### References

- Critical CSS by Sergey Labuts (GitNation) — https://gitnation.com
- web.dev — Extract critical CSS — https://web.dev/articles/extract-critical-css
- Critical: Painting the Fold Before the Stylesheet Lands — https://reactlibs.dev

---

## 3. Critical CSS

### Definitions

**Core Definition:** Critical CSS is the subset of CSS required to render above-the-fold content, inlined into the HTML to eliminate render-blocking stylesheet requests.

**Technical Definition:** Critical CSS extraction identifies the styles used by above-the-fold content, inlines them in a `<style>` block in the HTML `<head>`, and loads the remaining CSS asynchronously. This eliminates the render-blocking round trip for the full stylesheet, allowing the browser to paint the above-the-fold content sooner. For large applications, extracting critical CSS for above-the-fold content improves FCP, often by 100–300ms on slow connections.

**Beginner-Friendly Explanation:** Critical CSS is the minimum CSS needed to make the first screen look right. By putting it directly in the HTML, the browser does not have to wait for a separate CSS file before drawing anything.

### Purposes

- To eliminate render-blocking CSS requests for the initial view.
- To improve First Contentful Paint (FCP) by 100–300ms.
- To reduce the time to first meaningful paint on slow networks.
- To complement code splitting by delivering only the CSS needed for the initial route.

### Syntax Rules and Structure

**Critical CSS Workflow:**

| Step | Action |
|------|--------|
| 1. Identify | Determine which styles affect above-the-fold content |
| 2. Extract | Run a critical CSS tool at build time (Critical, Penthouse, Beasties) |
| 3. Inline | Place the extracted CSS in a `<style>` block in `<head>` |
| 4. Defer | Load the full stylesheet asynchronously with `preload` + `onload` |

**Syntax Rules:**

1. Extract critical CSS at build time, not at runtime; runtime extraction increases server load.
2. Inline only the styles needed for above-the-fold content.
3. Do not inline hover, focus, or transition rules; they belong in the deferred stylesheet.
4. Most modern frameworks (Next.js App Router, Nuxt, SvelteKit) handle this automatically.

### Annotated Code Examples

**Build-Time Extraction (Conceptual):**

```bash
# Using the `critical` tool
npx critical index.html --inline --minify > index-critical.html
```

**Resulting HTML:**

```html
<head>
  <style>
    /* Inlined critical CSS for above-the-fold content */
    .hero { display: flex; padding: 2rem; }
    .hero img { width: 100%; height: auto; }
  </style>
  <link
    rel="preload"
    href="/styles/main.css"
    as="style"
    onload="this.onload=null;this.rel='stylesheet'"
  />
  <noscript><link rel="stylesheet" href="/styles/main.css" /></noscript>
</head>
```

**Expected Output:** The above-the-fold content renders immediately with the inlined CSS; the full stylesheet loads asynchronously.

**Why this output:** The inline styles eliminate the render-blocking CSS request for the first paint. The `preload` + `onload` pattern loads the remaining CSS without blocking.

### Real-World Cases

- **Server-rendered React (Next.js):** Framework-provided critical CSS extraction.
- **Client-rendered SPAs:** Using tools like Beasties or Critical against the prerendered HTML.
- **Large Tailwind applications:** Extracting only the utilities used above the fold.

### References

- Critical CSS by Sergey Labuts — https://gitnation.com
- Extract Critical CSS for Initial Render — https://github.com/pproenca/dot-skills/blob/master/skills/.curated/tailwind/references/bundle-split-critical-css.md
- Critical: Painting the Fold Before the Stylesheet Lands — https://reactlibs.dev
- Critical CSS extraction (Linaria docs) — https://raw.githubusercontent.com

---

## 4. Lazy Loading

### Definitions

**Core Definition:** Lazy loading is the practice of deferring the loading of non-critical resources — components, images, or code — until they are actually needed.

**Technical Definition:** Lazy loading in React is implemented through `React.lazy` and `Suspense` for components, and through the native `loading="lazy"` attribute for images and iframes. It defers non-critical resources until they are required, dramatically reducing the initial bundle size and improving LCP. Lazy loading is most effective for below-the-fold content, modals, heavy libraries, and routes the user has not visited.

**Beginner-Friendly Explanation:** Lazy loading means “do not load it until you need it.” If a user is not going to see a component or image right away, there is no reason to download it during the initial load.

### Purposes

- To reduce the initial JavaScript bundle size.
- To defer below-the-fold images and components.
- To improve LCP and FCP by shortening the critical path.
- To avoid downloading code for routes the user may never visit.

### Syntax Rules and Structure

**Component Lazy Loading:**

```jsx
import { lazy, Suspense } from 'react';

const HeavyChart = lazy(() => import('./HeavyChart'));

<Suspense fallback={<ChartSkeleton />}>
  <HeavyChart />
</Suspense>
```

**Image Lazy Loading:**

```jsx
<img src="/below-fold.avif" loading="lazy" width={800} height={600} />
```

**Syntax Rules:**

1. Wrap lazy components in `Suspense` with a meaningful fallback.
2. Use `loading="lazy"` for below-the-fold images and iframes.
3. Never lazy-load above-the-fold images or critical components.
4. Lazy load based on user intent (hover, visibility) where appropriate.

### Annotated Code Examples

```jsx
import { lazy, Suspense, useState } from 'react';

const HeavyEditor = lazy(() => import('./HeavyEditor'));

function App() {
  const [showEditor, setShowEditor] = useState(false);

  return (
    <div>
      <button onClick={() => setShowEditor(true)}>Open Editor</button>
      {showEditor && (
        <Suspense fallback={<EditorSkeleton />}>
          <HeavyEditor />
        </Suspense>
      )}
    </div>
  );
}
```

**Expected Output:** The editor's chunk is downloaded only when the user clicks “Open Editor.”

**Why this output:** `React.lazy` creates a split point. The `HeavyEditor` chunk is not in the initial bundle; it is fetched when the component renders.

### Real-World Cases

- **Modals and dialogs:** Loading modal content only when opened.
- **Rich text editors:** Deferring Monaco or CodeMirror.
- **Below-the-fold images:** Deferring off-screen images.
- **Routes:** Loading each route component on navigation.

### References

- Steve Kinney — Code-Splitting and Lazy Loading — https://stevekinney.com/courses/react-performance/code-splitting-and-lazy-loading
- React — lazy — https://react.dev/reference/react/lazy
- How to Performance Optimization Techniques in React and Next.js — https://tsecurity.de
- certlab ADR-016 Performance Optimization — https://github.com/certlab/certlab

---

## 5. Image Optimization

### Definitions

**Core Definition:** Image optimization is the practice of serving images in the smallest appropriate format and size, with explicit dimensions and appropriate loading behaviour, to minimize transfer cost and layout shift.

**Technical Definition:** Image optimization encompasses modern formats (AVIF, WebP), responsive `srcset` and `sizes`, explicit `width` and `height` attributes to prevent Cumulative Layout Shift (CLS), `fetchpriority` for LCP images, and `loading="lazy"` for below-the-fold images. Images without dimensions cause CLS; explicit dimensions allow the browser to reserve space before the image downloads.

**Beginner-Friendly Explanation:** Images are often the biggest files on a page. Optimizing them means using smaller modern formats, telling the browser exactly how big they are so the layout does not jump, and only loading the ones the user will see.

### Purposes

- To reduce image transfer size by 50–80% with modern formats.
- To prevent CLS by reserving space with explicit dimensions.
- To prioritize the LCP image and defer below-the-fold images.
- To serve the right image size for each device with responsive images.

### Syntax Rules and Structure

**Image Optimization Checklist:**

| Technique | Implementation |
|-----------|----------------|
| Modern format | `<img src="image.avif" />` with WebP/JPEG fallback |
| Responsive sizes | `srcset` and `sizes` attributes |
| CLS prevention | Explicit `width` and `height` |
| LCP priority | `fetchpriority="high"`, `loading="eager"` |
| Below-fold deferral | `loading="lazy"` |

**Syntax Rules:**

1. Always specify `width` and `height` (or an aspect ratio) to prevent CLS.
2. Use AVIF or WebP with a fallback for older browsers.
3. Use `srcset` and `sizes` for responsive images.
4. Set `fetchpriority="high"` and `loading="eager"` on the LCP image.
5. Use `loading="lazy"` on below-the-fold images.

### Annotated Code Examples

```jsx
function OptimizedImage({ src, alt, width, height, priority = false }) {
  return (
    <img
      src={src}
      alt={alt}
      width={width}
      height={height}
      loading={priority ? 'eager' : 'lazy'}
      fetchpriority={priority ? 'high' : 'auto'}
    />
  );
}
```

**Expected Output:** The image reserves space (no CLS), loads eagerly at high priority if it is the LCP image, and loads lazily otherwise.

**Why this output:** Explicit dimensions prevent layout shift. `fetchpriority` and `loading` control network priority and deferral.

### Real-World Cases

- **E-commerce:** Product images are often the LCP element.
- **Media sites:** Hero images and thumbnails dominate the payload.
- **Dashboards:** Avatars and chart images benefit from lazy loading.

### References

- Steve Kinney — Image & Asset Optimization for React — https://stevekinney.com/courses/react-performance/image-and-asset-optimization
- @page-speed/img — https://www.npmjs.com/package/@page-speed/img
- How to Performance Optimization Techniques in React and Next.js — https://tsecurity.de

---

## 6. Font Optimization

### Definitions

**Core Definition:** Font optimization is the practice of loading web fonts in a way that minimizes render delay and layout shift, using `font-display`, preloading, and metric-matched fallbacks.

**Technical Definition:** Font optimization addresses two problems: render-blocking font downloads and layout shift when the web font replaces the fallback font. `font-display: swap` renders text immediately with a fallback and swaps in the web font when it arrives. However, the swap can cause CLS if the fallback and web font have different metrics. The fix is `size-adjust`, `ascent-override`, `descent-override`, and `line-gap-override` in `@font-face` to match the fallback's metrics to the web font's metrics, eliminating the shift.

**Beginner-Friendly Explanation:** Web fonts can delay text rendering or cause the page to jump when the font changes. Font optimization means showing text immediately with a system font, then swapping to the web font without the layout moving.

### Purposes

- To eliminate invisible text during font loading.
- To prevent CLS caused by font swap.
- To reduce the time to first text paint.
- To match fallback font metrics to the web font.

### Syntax Rules and Structure

**Font Loading Strategy:**

| Technique | Effect |
|-----------|--------|
| `font-display: swap` | Text renders immediately with fallback |
| `preload` the font | Fetches the font early |
| `size-adjust` fallback | Matches fallback metrics to web font |
| `ascent-override` etc. | Fine-tunes fallback metrics |

**Syntax Rules:**

1. Use `font-display: swap` or `optional` to avoid invisible text.
2. Preload the primary font file.
3. Define a metric-matched fallback `@font-face` with `size-adjust`, `ascent-override`, `descent-override`, and `line-gap-override`.
4. Self-host fonts to avoid third-party connection overhead.

### Annotated Code Examples

```css
@font-face {
  font-family: 'Inter';
  src: url('/fonts/inter.woff2') format('woff2');
  font-display: swap;
}

@font-face {
  font-family: 'Inter-Fallback';
  src: local('Arial');
  size-adjust: 107%;
  ascent-override: 90%;
  descent-override: 22%;
  line-gap-override: 0%;
}

body {
  font-family: 'Inter', 'Inter-Fallback', sans-serif;
}
```

**Expected Output:** Text renders immediately with the metric-matched fallback, and the swap to Inter does not cause layout shift.

**Why this output:** `font-display: swap` ensures text is visible immediately. The fallback's metrics are adjusted to match Inter, so the swap does not change line height or character width.

### Real-World Cases

- **Content sites:** Fonts often block text rendering and cause CLS.
- **Design systems:** Metric-matched fallbacks keep layouts stable.
- **Multi-language sites:** Different scripts may require different font strategies.

### References

- Chrome for Developers — Framework tools for font fallbacks — https://developer.chrome.com
- react_on_rails Issue #3875 — First-party font optimization — https://github.com/shakacode/react_on_rails/issues/3875
- How to Performance Optimization Techniques in React and Next.js — https://tsecurity.de

---

## 7. Preloading

### Definitions

**Core Definition:** Preloading is declaring a resource that the current page needs very soon, scheduling it for high-priority download and caching, without executing it.

**Technical Definition:** `preload` tells the browser to fetch a resource (stylesheet, font, image, or script) at high priority because it is needed for the current page. It is declared via `<link rel="preload">` or, in React 19+, via the `preload` function from `react-dom`. Preloading is appropriate for critical CSS, the LCP image, and the primary font. It should not be used for resources that are not needed on the current page.

**Beginner-Friendly Explanation:** Preloading means telling the browser, “I need this file right now for this page, so fetch it first.” It is for critical resources, not future pages.

### Purposes

- To fetch critical resources before the browser discovers them naturally.
- To reduce the time to LCP and FCP.
- To avoid render-blocking waterfalls for CSS and fonts.
- To give the browser explicit priority information.

### Syntax Rules and Structure

**React 19+ (`react-dom`):**

```jsx
import { preload } from 'react-dom';

preload('/hero.avif', { as: 'image', fetchPriority: 'high' });
preload('/fonts/inter.woff2', { as: 'font', type: 'font/woff2', crossOrigin: 'anonymous' });
```

**HTML:**

```html
<link rel="preload" href="/hero.avif" as="image" fetchpriority="high" />
<link rel="preload" href="/fonts/inter.woff2" as="font" type="font/woff2" crossorigin />
```

**Syntax Rules:**

1. Preload only resources needed for the current page.
2. Specify the correct `as` value (`image`, `font`, `style`, `script`, `fetch`).
3. Use `fetchpriority="high"` for the LCP image.
4. Do not preload route chunks for future navigations; use prefetch instead.

### Annotated Code Examples

```jsx
import { preload } from 'react-dom';

function ProductPage({ product }) {
  preload(product.imageUrl, { as: 'image', fetchPriority: 'high' });
  preload('/fonts/inter.woff2', { as: 'font', type: 'font/woff2', crossOrigin: 'anonymous' });

  return (
    <img src={product.imageUrl} alt={product.name} width={800} height={600} fetchpriority="high" />
  );
}
```

**Expected Output:** The product image and font are fetched at high priority, reducing LCP.

**Why this output:** `preload` declares the resources before the browser would otherwise discover them, eliminating the discovery waterfall.

### Real-World Cases

- **LCP image:** Preloading the hero image improves LCP.
- **Critical fonts:** Preloading the primary font avoids invisible text.
- **Critical CSS:** Preloading a stylesheet that cannot be inlined.

### References

- React — Resource Preloading APIs — https://18.react.dev/reference/react-dom
- Steve Kinney — Priority Hints & Resource Loading — https://stevekinney.com/courses/react-performance/priority-hints-resource-loading
- MDN — Resource Hints — https://developer.mozilla.org
- web.dev — Resource hints — https://web.dev

---

## 8. Prefetching

### Definitions

**Core Definition:** Prefetching is downloading a resource during idle time because it is likely to be needed for a future navigation, without blocking the current page.

**Technical Definition:** `prefetch` tells the browser to fetch a resource at low priority during idle time and store it in the HTTP cache for a likely future navigation. Unlike preload, which is for the current page, prefetch is for the next page. In React, prefetching is often combined with code splitting: route chunks and data for predicted routes are prefetched on hover or viewport visibility.

**Beginner-Friendly Explanation:** Prefetching means, “I will probably need this soon, so download it in the background while I am not busy.”

### Purposes

- To make future navigations feel instant.
- To use idle network time productively.
- To reduce the latency of route transitions.
- To prefetch data and code for predicted next steps.

### Syntax Rules and Structure

**HTML:**

```html
<link rel="prefetch" href="/next-page.js" as="script" />
```

**React Router:**

```jsx
<Link to="/dashboard" prefetch="intent">Dashboard</Link>
```

**Syntax Rules:**

1. Prefetch resources for future navigations, not the current page.
2. Use low priority; do not compete with critical resources.
3. Prefetch on user intent (hover, focus) where possible.
4. Do not prefetch chunks for routes the user is unlikely to visit.

### Annotated Code Examples

```jsx
import { Link } from 'react-router-dom';

function Navigation() {
  return (
    <Link
      to="/dashboard"
      prefetch="intent"  // Prefetch when the user hovers or focuses
    >
      Dashboard
    </Link>
  );
}
```

**Expected Output:** The dashboard route chunk is prefetched when the user hovers over the link, making navigation instant.

**Why this output:** `prefetch="intent"` triggers a low-priority prefetch on hover or focus, so the chunk is cached before the user clicks.

### Real-World Cases

- **Navigation menus:** Prefetching routes on hover.
- **Multi-step wizards:** Prefetching the next step.
- **E-commerce:** Prefetching product pages from a listing.

### References

- MDN — Resource Hints — https://developer.mozilla.org
- web.dev — Resource hints — https://web.dev
- React Router — Prefetching — https://mintlify.wiki
- Steve Kinney — Priority Hints & Resource Loading — https://stevekinney.com/courses/react-performance/priority-hints-resource-loading

---

## 9. Preconnecting

### Definitions

**Core Definition:** Preconnecting is establishing an early connection (DNS + TCP + TLS) to an origin that the page will need, reducing the latency of future requests to that origin.

**Technical Definition:** `preconnect` tells the browser to perform DNS resolution, TCP handshake, and TLS negotiation with an origin before the page actually requests a resource from it. This is particularly valuable for third-party origins (CDNs, font providers, API servers) where the connection setup cost would otherwise be added to the first request. `dns-prefetch` is a cheaper alternative that only resolves DNS.

**Beginner-Friendly Explanation:** Preconnecting is like opening a phone line to a server before you need to make a call. When you finally need something from that server, the connection is already ready.

### Purposes

- To eliminate connection setup latency for critical third-party origins.
- To reduce TTFB for API and CDN requests.
- To complement preload and prefetch for cross-origin resources.
- To improve LCP when the LCP resource is on a third-party origin.

### Syntax Rules and Structure

**React 19+ (`react-dom`):**

```jsx
import { preconnect, prefetchDNS } from 'react-dom';

preconnect('https://fonts.googleapis.com');
prefetchDNS('https://analytics.example.com');
```

**HTML:**

```html
<link rel="preconnect" href="https://fonts.googleapis.com" />
<link rel="dns-prefetch" href="https://analytics.example.com" />
```

**Syntax Rules:**

1. Use `preconnect` for origins the page will definitely use.
2. Use `dns-prefetch` when only DNS resolution is needed.
3. Do not overuse preconnect; each connection consumes resources.
4. Combine with `preload` for cross-origin critical resources.

### Annotated Code Examples

```jsx
import { preconnect } from 'react-dom';

function App() {
  preconnect('https://cdn.example.com');
  preconnect('https://fonts.gstatic.com', { crossOrigin: 'anonymous' });

  return <MainContent />;
}
```

**Expected Output:** Connections to the CDN and font origin are established early, reducing the latency of subsequent requests.

**Why this output:** `preconnect` initiates DNS, TCP, and TLS before the browser discovers the actual resource URLs, removing connection setup from the critical path.

### Real-World Cases

- **CDN-hosted assets:** Preconnecting to the asset CDN.
- **Google Fonts:** Preconnecting to `fonts.gstatic.com`.
- **API servers:** Preconnecting to the backend API origin.

### References

- React — Resource Preloading APIs — https://18.react.dev/reference/react-dom
- MDN — Resource Hints — https://developer.mozilla.org
- web.dev — Preconnect and dns-prefetch — https://web.dev

---

## 10. Service Workers

### Definitions

**Core Definition:** A service worker is a script that runs in a separate browser thread, intercepts network requests, and can serve cached responses to enable instant loading and offline support.

**Technical Definition:** Service workers act as a programmable proxy between the React application and the network. They intercept `fetch` events, allowing the application to implement caching strategies: **CacheFirst** for versioned static assets (scripts, styles, images, fonts), **NetworkFirst** for API and HTML responses where freshness matters, **StaleWhileRevalidate** for content that can be slightly stale, and **NetworkOnly** for authenticated or dynamic routes. Workbox is the industry-standard library for implementing service worker caching strategies.

**Beginner-Friendly Explanation:** A service worker is like a smart cache that sits between your app and the internet. On repeat visits, it can serve files from the cache instantly, making the app feel like it loads immediately.

### Purposes

- To enable instant loading on repeat visits with cache-first strategies.
- To provide offline support.
- To reduce network dependency for static assets.
- To implement background synchronization when connectivity returns.

### Syntax Rules and Structure

**Registration:**

```jsx
useEffect(() => {
  if ('serviceWorker' in navigator && process.env.NODE_ENV === 'production') {
    navigator.serviceWorker.register('/sw.js');
  }
}, []);
```

**Workbox Caching Strategies:**

| Request type | Strategy | Rationale |
|--------------|----------|-----------|
| Static assets (`/_next/static/*`, scripts, styles, images, fonts) | CacheFirst | Versioned assets; cache forever with expiry |
| API (`/api/*`) | NetworkFirst | Freshness priority; cache fallback for offline |
| HTML navigation | NetworkFirst | Fresh HTML with offline fallback |
| Authenticated routes | NetworkOnly | Never cache sensitive data |

**Syntax Rules:**

1. Register the service worker only in production.
2. Use CacheFirst for versioned static assets with a 30-day expiry.
3. Use NetworkFirst for API and HTML responses.
4. Use NetworkOnly for authenticated or dynamic routes.
5. Handle updates with a user-facing “update available” notification.

### Annotated Code Examples

```js
// sw.js — Workbox configuration
import { precacheAndRoute } from 'workbox-precaching';
import { registerRoute } from 'workbox-routing';
import { CacheFirst, NetworkFirst } from 'workbox-strategies';
import { ExpirationPlugin } from 'workbox-expiration';

precacheAndRoute(self.__WB_MANIFEST);

// Static assets: CacheFirst with 30-day expiry
registerRoute(
  ({ request }) => ['script', 'style', 'image', 'font'].includes(request.destination),
  new CacheFirst({
    cacheName: 'static-v1',
    plugins: [new ExpirationPlugin({ maxAgeSeconds: 30 * 24 * 60 * 60 })],
  })
);

// API: NetworkFirst with 5-minute cache
registerRoute(
  ({ url }) => url.pathname.startsWith('/api/'),
  new NetworkFirst({
    cacheName: 'api-v1',
    plugins: [new ExpirationPlugin({ maxAgeSeconds: 5 * 60 })],
  })
);
```

**Expected Output:** Static assets are served from cache instantly on repeat visits; API responses are fetched from the network with a cache fallback.

**Why this output:** `CacheFirst` prioritizes speed for immutable assets. `NetworkFirst` prioritizes freshness for dynamic data while maintaining an offline fallback.

### Real-World Cases

- **SPAs with repeat visits:** Instant load from cache.
- **Offline-first apps:** Serving a fallback page when offline.
- **Micro-frontends:** Service workers dynamically cache remote modules.

### References

- Steve Kinney — Service Worker Strategies for React Apps — https://stevekinney.com/courses/react-performance/service-worker-strategies
- Workbox cache strategies — https://raw.githubusercontent.com
- Sourdough ADR-019 Progressive Web App — https://github.com/Sourdough-start/sourdough
- How to add offline support in React · CoreUI — https://coreui.io

---

## 11. Loading Performance Best Practices

### Definitions

**Core Definition:** Loading performance best practices are the guidelines that systematically reduce the time to first meaningful content and ensure the application loads efficiently across devices and networks.

**Technical Definition:** Loading performance best practices combine critical path optimization (critical CSS, resource prioritization), payload reduction (code splitting, lazy loading, tree shaking, compression), asset optimization (modern image formats, font metric matching), delivery optimization (preconnect, preload, prefetch, service worker caching), and measurement-driven iteration (Core Web Vitals, RUM). The React-specific dimension is the use of `React.lazy`, `Suspense`, and `react-dom` resource preloading APIs to align component rendering with resource fetching.

**Beginner-Friendly Explanation:** Loading performance is a stack of habits. Inline the CSS needed for the first screen, lazy-load everything else, compress your images, load fonts without blocking text, tell the browser what to fetch first, cache aggressively, and measure the result.

### Best Practices Table

| Practice | Description | Impact |
|----------|-------------|--------|
| **Inline critical CSS** | Eliminate render-blocking CSS for above-the-fold content | 100–300ms faster FCP |
| **Preload the LCP image** | `fetchpriority="high"` + `preload` | Direct LCP improvement |
| **Preconnect to critical origins** | CDN, font, API origins | Removes connection setup latency |
| **Lazy-load below-the-fold content** | `React.lazy` + `Suspense`, `loading="lazy"` | Reduces initial bundle |
| **Code-split by route** | Each route loads on demand | 40–70% initial bundle reduction |
| **Optimize images** | AVIF/WebP, explicit dimensions, responsive `srcset` | 50–80% image size reduction |
| **Optimize fonts** | `font-display: swap`, metric-matched fallback | Eliminates font CLS |
| **Prefetch predicted routes** | Prefetch on hover/intent | Instant navigation |
| **Use service workers** | CacheFirst for static, NetworkFirst for API | Instant repeat visits |
| **Defer third-party scripts** | Load analytics after hydration | Removes blocking scripts |
| **Set performance budgets** | Enforce LCP ≤ 2.5s, FCP ≤ 1.8s, CLS ≤ 0.1 | Prevents regressions |
| **Measure with RUM** | `web-vitals` library, real user data | Data-driven optimization |

### Annotated Code Examples

**✅ Good Practices (Combined):**

```jsx
import { lazy, Suspense } from 'react';
import { preload, preconnect } from 'react-dom';

// Preconnect to critical origins
preconnect('https://cdn.example.com');

const HeavyChart = lazy(() => import('./HeavyChart'));

function ProductPage({ product }) {
  // Preload the LCP image
  preload(product.imageUrl, { as: 'image', fetchPriority: 'high' });

  return (
    <>
      {/* Above-the-fold: eager, high priority */}
      <img
        src={product.imageUrl}
        alt={product.name}
        width={800}
        height={600}
        fetchpriority="high"
        loading="eager"
      />
      {/* Below-the-fold: lazy */}
      <Suspense fallback={<ChartSkeleton />}>
        <HeavyChart data={product.chartData} />
      </Suspense>
    </>
  );
}
```

**❌ Bad Practices:**

```jsx
// ❌ No priority hints, no dimensions, lazy-loading the LCP image
<img src="/hero.jpg" loading="lazy" />

// ❌ Eagerly importing all routes
import Dashboard from './pages/Dashboard';
import Settings from './pages/Settings';

// ❌ Render-blocking font without fallback metrics
@font-face {
  font-family: 'Inter';
  src: url('/inter.woff2');
  /* No font-display, no fallback metrics */
}
```

**Expected Output:** The good practices produce a fast first paint, a high-priority LCP image, deferred below-the-fold code, and no layout shift. The bad practices produce a slow first paint, a delayed LCP, a large initial bundle, and font-induced CLS.

**Why this output:** Preloading and `fetchpriority` put the LCP image on the critical path. `React.lazy` removes the chart from the initial bundle. Explicit dimensions prevent CLS. Font metric overrides prevent swap-induced shift.

### Real-World Cases

- **All React applications:** These practices apply universally, with emphasis on RSC and SSR for content-heavy sites.
- **E-commerce:** LCP and image optimization are critical for conversion.
- **SaaS dashboards:** Code splitting and service worker caching improve repeat visits.

### References

- Steve Kinney — Core Web Vitals for React Applications — https://stevekinney.com/courses/react-performance/core-web-vitals-for-react
- Steve Kinney — Priority Hints & Resource Loading — https://stevekinney.com/courses/react-performance/priority-hints-resource-loading
- Steve Kinney — Code-Splitting and Lazy Loading — https://stevekinney.com/courses/react-performance/code-splitting-and-lazy-loading
- Steve Kinney — Service Worker Strategies for React Apps — https://stevekinney.com/courses/react-performance/service-worker-strategies
- React — Resource Preloading APIs — https://18.react.dev/reference/react-dom
- Bundle Size Optimization (CRITICAL) — https://raw.githubusercontent.com
- MDN — Resource Hints — https://developer.mozilla.org
- web.dev — Critical CSS — https://web.dev/articles/extract-critical-css
- Critical CSS by Sergey Labuts — https://gitnation.com
- Chrome for Developers — Framework tools for font fallbacks — https://developer.chrome.com
- Workbox cache strategies — https://raw.githubusercontent.com
- Sourdough ADR-019 Progressive Web App — https://github.com/Sourdough-start/sourdough

---

## References

- Steve Kinney — Core Web Vitals for React Applications — https://stevekinney.com/courses/react-performance/core-web-vitals-for-react
- Steve Kinney — Priority Hints & Resource Loading — https://stevekinney.com/courses/react-performance/priority-hints-resource-loading
- Steve Kinney — Code-Splitting and Lazy Loading — https://stevekinney.com/courses/react-performance/code-splitting-and-lazy-loading
- Steve Kinney — Service Worker Strategies for React Apps — https://stevekinney.com/courses/react-performance/service-worker-strategies
- Steve Kinney — Image & Asset Optimization for React — https://stevekinney.com/courses/react-performance/image-and-asset-optimization
- React — Resource Preloading APIs — https://18.react.dev/reference/react-dom
- React — lazy — https://react.dev/reference/react/lazy
- MDN — Resource Hints — https://developer.mozilla.org
- web.dev — Extract critical CSS — https://web.dev/articles/extract-critical-css
- web.dev — Preconnect and dns-prefetch — https://web.dev
- Critical CSS by Sergey Labuts (GitNation) — https://gitnation.com
- Critical: Painting the Fold Before the Stylesheet Lands — https://reactlibs.dev
- Chrome for Developers — Framework tools for font fallbacks — https://developer.chrome.com
- react_on_rails Issue #3875 — First-party font optimization — https://github.com/shakacode/react_on_rails/issues/3875
- @page-speed/img — https://www.npmjs.com/package/@page-speed/img
- Bundle Size Optimization — https://raw.githubusercontent.com
- Workbox cache strategies — https://raw.githubusercontent.com
- Sourdough ADR-019 Progressive Web App — https://github.com/Sourdough-start/sourdough
- How to add offline support in React · CoreUI — https://coreui.io
- How to Performance Optimization Techniques in React and Next.js — https://tsecurity.de
- certlab ADR-016 Performance Optimization — https://github.com/certlab/certlab
- Extract Critical CSS for Initial Render — https://github.com/pproenca/dot-skills/blob/master/skills/.curated/tailwind/references/bundle-split-critical-css.md
- Critical CSS extraction (Linaria docs) — https://raw.githubusercontent.com