# React Performance Fundamentals — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** React performance is the practice of measuring and optimizing how quickly a React application loads, becomes interactive, and responds to user input, both objectively (measured timings) and subjectively (how fast it feels).

**Technical Definition:** Web performance is the objective measurements and the perceived user experience of load time and runtime. It encompasses how long a site takes to load, become interactive and responsive, and how smooth content is during interactions. In React, performance is shaped by the framework's rendering pipeline (render phase → commit phase), the JavaScript bundle size, the cost of component re-renders, and the efficiency of reconciliation. Performance is measured across three dimensions: **load performance** (time to first meaningful content), **runtime performance** (responsiveness during interaction), and **perceived performance** (how fast the experience *feels*, which can be improved through optimistic UI, skeleton screens, and progressive loading).

**Beginner-Friendly Explanation:** Performance is about making your app feel fast. That means it should load quickly, respond instantly when you click or type, and not jump around while loading. Some of this is objective — you can measure exactly how many milliseconds something takes. Some of it is subjective — an app can feel fast even if it technically takes a while, if it shows you something useful immediately. React performance work is about finding the slow parts, understanding why they are slow, and fixing them without guessing.

### Key Characteristics

- **Multi-dimensional:** Performance spans load (initial download and render), runtime (interaction responsiveness), and perceived speed.
- **User-centric:** Core Web Vitals measure real user experience, not synthetic benchmark scores.
- **Measurable:** Objective metrics (LCP, INP, CLS, TTFB, bundle size) and subjective experience (perceived latency) can both be tracked.
- **React-specific:** React's rendering model (reconciliation, re-renders, concurrent features) introduces unique performance considerations.
- **Budget-driven:** Performance budgets turn vague goals into enforceable CI/CD constraints.
- **Continuous:** Performance is monitored in production (RUM) and tested in CI (synthetic).

### Prerequisites

- React function components, state, props, and re-rendering concepts.
- Understanding of the browser rendering pipeline (parse → layout → paint → composite).
- Familiarity with the Network and Performance panels in browser DevTools.
- Basic knowledge of Core Web Vitals terminology.

### Related Programming Areas

- **React Rendering** — reconciliation, fiber, concurrent rendering.
- **Code Splitting and Lazy Loading** — reducing initial bundle size.
- **Memoization** — `React.memo`, `useMemo`, `useCallback` for avoiding unnecessary work.
- **Web Vitals** — LCP, INP, CLS as the standard user-experience metrics.
- **Observability** — RUM, synthetic monitoring, and alerting.

---

## 1. Performance

### Definitions

**Core Definition:** Performance is the efficiency with which a web application loads, renders, and responds to user interactions.

**Technical Definition:** Web performance is the objective measurements and the perceived user experience of load time and runtime. It measures how long a site takes to load, become interactive, and respond to input, and how smooth content is during interactions. Performance is not a single metric but a collection of measurements across the network, the main thread, and the rendering pipeline. It includes the time to fetch resources, parse and execute JavaScript, render pixels, and process user events.

**Beginner-Friendly Explanation:** Performance is how fast and smooth your app feels. It covers everything from the moment a user requests your page to the moment they click a button and see a response.

### Purposes

- To retain users and reduce abandonment caused by slow experiences.
- To improve search engine ranking through Core Web Vitals.
- To ensure the application remains usable on low-end devices and slow networks.
- To provide a smooth, responsive experience during interactions.
- To reduce infrastructure costs by shipping less code and making fewer requests.

### Syntax Rules and Structure

**Performance Dimensions:**

| Dimension | Description | Example Metrics |
|-----------|-------------|-----------------|
| Load performance | Time to fetch and render initial content | LCP, FCP, TTFB |
| Runtime performance | Responsiveness during interaction | INP, long tasks |
| Perceived performance | How fast the experience feels | Skeleton screens, optimistic UI |

**Syntax Rules:**

1. Performance is measured both in the lab (synthetic) and in the field (RUM).
2. Metrics must be tied to user experience, not just technical benchmarks.
3. Optimization should always be driven by measurement, not guesswork.

### Annotated Code Examples

```jsx
// Measuring a performance mark in React
import { useEffect } from 'react';

function ExpensiveComponent() {
  useEffect(() => {
    performance.mark('expensive-start');
    // ... expensive work
    performance.mark('expensive-end');
    performance.measure('expensive', 'expensive-start', 'expensive-end');
  }, []);
}
```

**Expected Output:** A performance measure named `"expensive"` appears in the browser's Performance panel, showing the duration of the work.

**Why this output:** The `performance.mark` and `performance.measure` APIs record timestamps and compute the duration, making it visible in DevTools.

### Real-World Cases

- **E-commerce:** Slow load times directly reduce conversion rates.
- **SaaS dashboards:** Unresponsive interactions frustrate users during data analysis.
- **Content platforms:** Poor LCP causes users to abandon before reading.

### References

- MDN — Web performance — https://developer.mozilla.org/en-US/docs/Web/Performance
- Steve Kinney — Core Web Vitals for React Applications — https://stevekinney.com/courses/react-performance/core-web-vitals-for-react
- Steve Kinney — Measuring Performance with Real Tools — https://stevekinney.com/courses/react-performance/measuring-performance-with-real-tools

---

## 2. Rendering Performance

### Definitions

**Core Definition:** Rendering performance is the efficiency with which React computes and commits UI updates to the DOM.

**Technical Definition:** Rendering performance in React concerns the **render phase** (calling component functions and reconciling the virtual tree) and the **commit phase** (applying DOM mutations). React's reconciliation algorithm compares the new element tree with the previous one; unnecessary re-renders, expensive component bodies, and large trees increase render time. Rendering performance is measured by component render duration, render counts, and the time from an interaction to the next paint.

**Beginner-Friendly Explanation:** Rendering performance is about how fast React can figure out what changed and update the screen. If your components do too much work or re-render too often, the UI feels sluggish.

### Purposes

- To minimize the time spent in React's render and commit phases.
- To avoid unnecessary re-renders of components whose output has not changed.
- To keep component render times under the frame budget (16ms for 60fps).
- To ensure interactions produce a next paint quickly (INP).

### Syntax Rules and Structure

**Rendering Performance Levers:**

| Lever | Mechanism |
|-------|-----------|
| `React.memo` | Skips re-render if props are shallowly equal |
| `useMemo` | Caches expensive computation results |
| `useCallback` | Caches function references for memoized children |
| Component splitting | Isolates state so re-renders do not cascade |
| Keys in lists | Preserves identity and avoids unnecessary DOM work |
| Concurrent features | `useTransition`, `useDeferredValue` for non-urgent updates |

**Syntax Rules:**

1. Measure before optimizing — use the React DevTools Profiler to identify slow components.
2. Avoid creating new objects or functions in render when passing them to memoized children.
3. Keep component bodies pure and free of expensive synchronous work.
4. Use `React.memo` only for components that render often with stable props.

### Annotated Code Examples

```jsx
import { memo, useMemo, useState } from 'react';

const ExpensiveChild = memo(function ExpensiveChild({ data }) {
  return <div>{data.join(', ')}</div>;
});

function Parent({ items }) {
  const [count, setCount] = useState(0);

  // ✅ Cached computation: only re-runs when items changes
  const sorted = useMemo(() => [...items].sort(), [items]);

  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>Count: {count}</button>
      {/* ✅ Stable prop reference: child skips re-render */}
      <ExpensiveChild data={sorted} />
    </div>
  );
}
```

**Expected Output:** Clicking the count button does not re-render `ExpensiveChild`.

**Why this output:** `useMemo` keeps the `sorted` array reference stable until `items` changes. `memo` compares the prop reference and skips re-rendering the child.

### Real-World Cases

- **Data grids:** Sorting and filtering large datasets on every render.
- **Dashboards:** Many independent widgets re-rendering on a single state change.
- **Lists:** Item components re-rendering when only the list length changes.

### References

- Steve Kinney — Measuring Performance with Real Tools — https://stevekinney.com/courses/react-performance/measuring-performance-with-real-tools
- Steve Kinney — Debugging React Performance Issues — https://stevekinney.com/courses/react-performance/debugging-performance-issues

---

## 3. Runtime Performance

### Definitions

**Core Definition:** Runtime performance is how efficiently an application responds to user interactions after it has loaded.

**Technical Definition:** Runtime performance encompasses the main-thread work required to process events, update state, re-render React components, and paint the result. It is measured primarily by **Interaction to Next Paint (INP)** , which captures the latency from a user interaction (click, tap, keypress) to the next frame that reflects the result. Long tasks (main-thread work exceeding 50ms) block the event loop and degrade responsiveness.

**Beginner-Friendly Explanation:** Runtime performance is how quickly your app reacts when you click, type, or scroll. If the page freezes for a moment before responding, runtime performance is the problem.

### Purposes

- To ensure interactions feel instant (INP ≤ 200ms).
- To avoid blocking the main thread with long tasks.
- To keep animations and scrolling smooth (60fps).
- To provide immediate feedback even when background work is occurring.

### Syntax Rules and Structure

**Runtime Performance Levers:**

| Lever | Mechanism |
|-------|-----------|
| Break up long tasks | Yield to the main thread between chunks |
| `startTransition` | Mark non-urgent state updates |
| `useDeferredValue` | Defer expensive re-renders |
| Web Workers | Move CPU-heavy work off the main thread |
| Event handler optimization | Avoid heavy synchronous work in handlers |
| Virtualization | Render only visible list items |

**Syntax Rules:**

1. INP thresholds: Good ≤ 200ms; Needs Improvement 200–500ms; Poor > 500ms.
2. Long tasks (>50ms) block the main thread and delay the next paint.
3. Use `scheduler.yield()` (where supported) or `setTimeout` to break up work.
4. Defer non-critical JavaScript and avoid heavy event handlers.

### Annotated Code Examples

```jsx
import { useState, useTransition } from 'react';

function Search() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  const [isPending, startTransition] = useTransition();

  function handleChange(e) {
    setQuery(e.target.value); // Urgent: update input immediately
    startTransition(() => {
      // Non-urgent: expensive search can be interrupted
      setResults(expensiveSearch(e.target.value));
    });
  }

  return (
    <div>
      <input value={query} onChange={handleChange} />
      {isPending && <p>Searching…</p>}
      <ResultsList results={results} />
    </div>
  );
}
```

**Expected Output:** The input stays responsive while the search runs in a transition. `isPending` indicates background work.

**Why this output:** `startTransition` marks the search update as non-urgent. React can interrupt it to process the urgent input update, keeping INP low.

### Real-World Cases

- **Search autocomplete:** Typing must remain responsive while results are computed.
- **Filters and sorting:** Applying filters should not freeze the UI.
- **Rich text editors:** Keystrokes must be processed immediately.

### References

- Vercel — How to improve Core Web Vitals — https://vercel.com/kb/guide/how-to-improve-core-web-vitals
- Steve Kinney — Core Web Vitals for React Applications — https://stevekinney.com/courses/react-performance/core-web-vitals-for-react
- GitHub — Web performance thresholds and INP guidance — https://github.com/agents-inc/skills

---

## 4. Load Performance

### Definitions

**Core Definition:** Load performance is the speed at which an application fetches its resources, renders initial content, and becomes usable.

**Technical Definition:** Load performance covers the network and parsing phases: DNS resolution, connection setup, server response (TTFB), downloading HTML, CSS, and JavaScript, parsing and executing scripts, and rendering the first meaningful content. **Largest Contentful Paint (LCP)** is the primary metric for load performance, measuring when the largest element in the viewport finishes rendering. LCP thresholds: Good ≤ 2.5s; Needs Improvement 2.5–4.0s; Poor > 4.0s.

**Beginner-Friendly Explanation:** Load performance is how fast your app appears on screen for the first time. If users stare at a blank screen for seconds, load performance is the issue.

### Purposes

- To minimize the time to first meaningful content (LCP).
- To reduce the initial JavaScript and CSS bundle size.
- To prioritize critical resources and defer non-critical ones.
- To improve perceived speed through progressive rendering.

### Syntax Rules and Structure

**Load Performance Levers:**

| Lever | Mechanism |
|-------|-----------|
| Code splitting | Load only the code needed for the current route |
| Lazy loading | Defer below-the-fold images and components |
| Resource hints | `preload`, `prefetch`, `preconnect` for critical assets |
| Image optimization | Modern formats (AVIF, WebP), responsive sizes |
| Server-side rendering (SSR) | Send HTML with content already rendered |
| Streaming | Send HTML in chunks as it becomes available |

**Syntax Rules:**

1. LCP is typically an image, heading, or hero block; identify it with `PerformanceObserver`.
2. Preload the LCP image and avoid lazy-loading it.
3. Minimize render-blocking CSS and JavaScript in the initial HTML.
4. Use modern image formats and explicitly set `width` and `height` to prevent CLS.

### Annotated Code Examples

```jsx
import { lazy, Suspense } from 'react';

// ✅ Code splitting: load the heavy chart only when needed
const HeavyChart = lazy(() => import('./HeavyChart'));

function Dashboard() {
  return (
    <Suspense fallback={<ChartSkeleton />}>
      <HeavyChart />
    </Suspense>
  );
}
```

**Expected Output:** The initial bundle excludes `HeavyChart`; it loads on demand, reducing the initial JavaScript payload and improving LCP.

**Why this output:** `React.lazy` creates a separate chunk for `HeavyChart`. The browser fetches it only when the component renders, so the initial load is smaller and faster.

### Real-World Cases

- **Marketing pages:** LCP directly affects conversion and SEO.
- **E-commerce product pages:** Hero images must render quickly.
- **News sites:** First meaningful content must appear immediately.

### References

- Steve Kinney — Core Web Vitals for React Applications — https://stevekinney.com/courses/react-performance/core-web-vitals-for-react
- Vercel — How to improve Core Web Vitals — https://vercel.com/kb/guide/how-to-improve-core-web-vitals
- Webkul — How to Improve Core Web Vitals in Next.js — https://webkul.com/blog/how-to-improve-core-web-vitals-in-next-js/

---

## 5. Perceived Performance

### Definitions

**Core Definition:** Perceived performance is how fast an application *feels* to the user, regardless of the actual measured timings.

**Technical Definition:** Perceived performance is the subjective user experience of load time and runtime — whether the content requested *feels* available and usable. It can be improved through techniques that mask latency: skeleton screens, optimistic UI updates, progressive loading, and immediate visual feedback. Perceived performance matters because users tolerate objective delays better when the interface acknowledges their actions immediately.

**Beginner-Friendly Explanation:** Perceived performance is about making your app *feel* fast, even if it technically is not. Showing a skeleton screen or updating the UI optimistically makes users feel like the app responded instantly.

### Purposes

- To reduce the perception of waiting during network requests.
- To provide immediate feedback for user actions.
- To make slow operations feel tolerable.
- To improve user satisfaction without necessarily reducing actual timings.

### Syntax Rules and Structure

**Perceived Performance Techniques:**

| Technique | Effect |
|-----------|--------|
| Skeleton screens | Show placeholder shapes while content loads |
| Optimistic UI | Update the UI immediately, roll back on error |
| Progressive loading | Render content in chunks as it arrives |
| Immediate feedback | Show a spinner or state change on click |
| Transition animations | Smooth state changes to mask latency |

**Syntax Rules:**

1. Perceived performance complements, but does not replace, real performance optimization.
2. Optimistic updates must include error handling to roll back on failure.
3. Skeleton screens should match the final layout to avoid layout shift.

### Annotated Code Examples

```jsx
function LikeButton({ postId, initialLikes }) {
  const [likes, setLikes] = useState(initialLikes);
  const [isPending, setIsPending] = useState(false);

  async function handleLike() {
    setLikes(l => l + 1); // Optimistic update
    setIsPending(true);

    try {
      await fetch(`/api/posts/${postId}/like`, { method: 'POST' });
    } catch {
      setLikes(l => l - 1); // Roll back on failure
    } finally {
      setIsPending(false);
    }
  }

  return (
    <button onClick={handleLike} disabled={isPending}>
      ♥ {likes}
    </button>
  );
}
```

**Expected Output:** The like count increments immediately when clicked, then rolls back if the request fails.

**Why this output:** The optimistic update makes the interaction feel instant. The rollback preserves correctness if the server rejects the request.

### Real-World Cases

- **Social media:** Likes, follows, and reactions update optimistically.
- **E-commerce:** “Add to cart” reflects immediately while the request completes in the background.
- **Dashboards:** Skeleton screens appear while widgets load.

### References

- MDN — Web performance (perceived user experience) — https://developer.mozilla.org/en-US/docs/Web/Performance
- Steve Kinney — Core Web Vitals for React Applications — https://stevekinney.com/courses/react-performance/core-web-vitals-for-react

---

## 6. Performance Metrics

### Definitions

**Core Definition:** Performance metrics are the standardized measurements used to quantify load, runtime, and perceived performance.

**Technical Definition:** Performance metrics are objective measurements collected from the browser's Performance APIs or from real user monitoring. They include navigation timing metrics (TTFB, FCP), rendering metrics (LCP), interactivity metrics (INP), and stability metrics (CLS). Metrics are classified as **field metrics** (collected from real users via RUM) or **lab metrics** (collected in controlled synthetic tests).

**Beginner-Friendly Explanation:** Performance metrics are the numbers you use to know whether your app is fast or slow. They include things like how long it takes for the biggest thing on screen to appear, how quickly buttons respond, and whether the layout jumps around.

### Purposes

- To provide objective targets for optimization.
- To compare performance across releases, devices, and geographies.
- To identify regressions before they reach users.
- To communicate performance status to stakeholders.

### Syntax Rules and Structure

**Key Metrics:**

| Metric | Measures | Source |
|--------|----------|--------|
| TTFB | Server response time | Navigation Timing API |
| FCP | First content paint | Paint Timing API |
| LCP | Largest content paint | PerformanceObserver |
| INP | Interaction responsiveness | Event Timing API |
| CLS | Visual stability | Layout Instability API |

**Syntax Rules:**

1. Field metrics reflect real users; lab metrics provide reproducibility.
2. Use the `web-vitals` library for accurate field measurement.
3. Do not rely on Lighthouse alone; real user data matters more.

### Annotated Code Examples

```jsx
import { onLCP, onINP, onCLS } from 'web-vitals';

function sendToAnalytics(metric) {
  navigator.sendBeacon('/analytics', JSON.stringify(metric));
}

onLCP(sendToAnalytics);
onINP(sendToAnalytics);
onCLS(sendToAnalytics);
```

**Expected Output:** LCP, INP, and CLS values are sent to an analytics endpoint for every user session.

**Why this output:** The `web-vitals` library registers observers for each metric and invokes the callback with the final value, enabling RUM collection.

### Real-World Cases

- **Performance dashboards:** Tracking LCP, INP, and CLS over time.
- **A/B testing:** Comparing performance between releases.
- **Alerting:** Triggering alerts when metrics exceed thresholds.

### References

- MDN — Web performance — https://developer.mozilla.org/en-US/docs/Web/Performance
- Steve Kinney — Core Web Vitals for React Applications — https://stevekinney.com/courses/react-performance/core-web-vitals-for-react
- OneUptime — How to Implement RUM in React Applications — https://github.com/OneUptime/blog

---

## 7. Core Web Vitals

### Definitions

**Core Definition:** Core Web Vitals are Google's standardized set of user-experience metrics that measure loading, interactivity, and visual stability.

**Technical Definition:** Core Web Vitals are three metrics — **Largest Contentful Paint (LCP)** , **Interaction to Next Paint (INP)** , and **Cumulative Layout Shift (CLS)** — that measure distinct aspects of user experience: loading performance, responsiveness, and visual stability. INP replaced FID as a Core Web Vital in March 2024. Thresholds: LCP ≤ 2.5s, INP ≤ 200ms, CLS ≤ 0.1.

**Beginner-Friendly Explanation:** Core Web Vitals are three scores that tell you how good your app's user experience is: how fast it loads (LCP), how quickly it responds (INP), and how stable it looks (CLS).

### Purposes

- To provide a unified, user-centric standard for performance.
- To serve as a search ranking signal.
- To guide optimization priorities across load, runtime, and stability.
- To enable comparison across sites and releases.

### Syntax Rules and Structure

**Core Web Vitals Thresholds:**

| Metric | Good | Needs Improvement | Poor |
|--------|------|-------------------|------|
| LCP | ≤ 2.5s | 2.5–4.0s | > 4.0s |
| INP | ≤ 200ms | 200–500ms | > 500ms |
| CLS | ≤ 0.1 | 0.1–0.25 | > 0.25 |

**Syntax Rules:**

1. All three metrics must be in the "Good" range to pass Core Web Vitals.
2. FID is deprecated; use INP for interactivity.
3. Measure in the field (RUM) for the most representative data.

### Annotated Code Examples

```jsx
import { onLCP, onINP, onCLS } from 'web-vitals';

onLCP(m => console.log('LCP', m.value));
onINP(m => console.log('INP', m.value));
onCLS(m => console.log('CLS', m.value));
```

**Expected Output:** The three Core Web Vitals values are logged when they are finalized.

**Why this output:** The `web-vitals` library reports each metric after the page has fully loaded and all interactions have been processed.

### Real-World Cases

- **SEO teams:** Core Web Vitals are a confirmed ranking signal.
- **Product teams:** The metrics map directly to user frustration points.
- **Engineering:** Each metric has distinct optimization strategies.

### References

- Steve Kinney — Core Web Vitals for React Applications — https://stevekinney.com/courses/react-performance/core-web-vitals-for-react
- GitHub — Core Web Vitals thresholds (confirmed unchanged) — https://github.com/vasilyu1983/AI-Agents-public
- Webkul — How to Improve Core Web Vitals in Next.js — https://webkul.com/blog/how-to-improve-core-web-vitals-in-next-js/

---

### 7.1 Largest Contentful Paint (LCP)

**Definitions:**

- **Core Definition:** LCP measures when the largest content element in the viewport finishes rendering.
- **Technical Definition:** LCP reports the render time of the largest image, text block, or video visible in the viewport, relative to when the page first started loading. It is the primary metric for loading performance.
- **Beginner-Friendly Explanation:** LCP is how long it takes for the biggest thing on the screen to appear.

**Purposes:**

- To measure perceived load speed.
- To identify slow-loading hero images or content blocks.
- To guide image optimization and critical resource prioritization.

**Syntax Rules and Structure:**

- LCP element is usually a hero image, `<h1>`, or large text block.
- Preload the LCP image and avoid lazy-loading it.
- Serve images in modern formats (AVIF, WebP) with explicit dimensions.

**Annotated Code Examples:**

```jsx
function Hero() {
  return (
    <img
      src="/hero.avif"
      alt="Hero"
      width={1200}
      height={600}
      fetchPriority="high" // ✅ Prioritize LCP image
    />
  );
}
```

**Expected Output:** The hero image loads with high priority, reducing LCP.

**Why this output:** `fetchPriority="high"` tells the browser to fetch the image sooner, so it renders earlier.

**Real-World Cases:**

- **E-commerce:** Product hero images must load in under 2.5s.
- **Marketing sites:** The main headline and hero visual determine LCP.

**References:**

- Steve Kinney — Core Web Vitals for React Applications — https://stevekinney.com/courses/react-performance/core-web-vitals-for-react
- GitHub — LCP optimization patterns — https://github.com/pproenca/dot-skills

---

### 7.2 First Input Delay (FID) — Deprecated

**Definitions:**

- **Core Definition:** FID measured the delay between a user's first interaction and the browser's ability to respond.
- **Technical Definition:** FID measured the time from when a user first interacted with the page (click, tap, keypress) to when the browser was able to begin processing event handlers. It was replaced by INP in March 2024.
- **Beginner-Friendly Explanation:** FID measured how long the page took to respond to the very first click or tap.

**Status:** **Deprecated.** Do not optimize for FID; use INP instead.

**References:**

- GitHub — INP replaced FID — https://github.com/agents-inc/skills

---

### 7.3 Cumulative Layout Shift (CLS)

**Definitions:**

- **Core Definition:** CLS measures the unexpected shifting of page content during loading.
- **Technical Definition:** CLS quantifies the visual instability of a page by measuring how much visible content shifts and how far it moves. It is calculated from layout shift entries reported by the browser. CLS ≤ 0.1 is "Good".
- **Beginner-Friendly Explanation:** CLS measures how much the page jumps around while loading.

**Purposes:**

- To prevent frustrating content shifts (e.g., text moving as images load).
- To improve readability and interaction accuracy.
- To signal visual stability to users.

**Syntax Rules and Structure:**

- Always set `width` and `height` on images and videos.
- Reserve space for dynamic content (ads, embeds, fonts).
- Avoid inserting content above existing content unless in response to interaction.

**Annotated Code Examples:**

```jsx
// ✅ Explicit dimensions prevent layout shift
<img src="/product.jpg" alt="Product" width={400} height={300} />

// ✅ Reserve space for an ad slot
<div style={{ minHeight: 250 }}>{/* ad loads here */}</div>
```

**Expected Output:** The layout does not shift when the image or ad loads.

**Why this output:** Explicit dimensions and reserved space tell the browser how much room to allocate before the content arrives.

**Real-World Cases:**

- **News sites:** Ads and images loading late cause severe CLS.
- **E-commerce:** Product images without dimensions shift the buy button.

**References:**

- OneUptime — RUM and CLS — https://github.com/OneUptime/blog
- Vercel — CLS prevention — https://vercel.com/kb/guide/how-to-improve-core-web-vitals

---

### 7.4 Interaction to Next Paint (INP)

**Definitions:**

- **Core Definition:** INP measures the responsiveness of a page to user interactions across the entire session.
- **Technical Definition:** INP observes the latency of all interactions (clicks, taps, keypresses) and reports a value that represents the worst-case (or near-worst-case) interaction latency. It replaced FID in March 2024 and covers every interaction, not just the first. INP ≤ 200ms is "Good".
- **Beginner-Friendly Explanation:** INP measures how quickly the page responds to every click, tap, and keypress throughout the session.

**Purposes:**

- To capture the full interaction lifecycle, not just the first input.
- To identify slow event handlers and long tasks.
- To ensure the app feels responsive during sustained use.

**Syntax Rules and Structure:**

- INP includes input delay, processing time, and presentation delay.
- Break up long tasks to allow the browser to paint.
- Use `startTransition` and `useDeferredValue` for non-urgent updates.
- Avoid heavy synchronous work in event handlers.

**Annotated Code Examples:**

```jsx
import { useDeferredValue, useMemo } from 'react';

function FilteredList({ items, query }) {
  const deferredQuery = useDeferredValue(query);
  const filtered = useMemo(
    () => items.filter(i => i.name.includes(deferredQuery)),
    [items, deferredQuery]
  );
  return <List items={filtered} />;
}
```

**Expected Output:** Typing in the input remains responsive; the list updates with a slight delay.

**Why this output:** `useDeferredValue` allows React to prioritize the urgent input update and defer the expensive list filtering.

**Real-World Cases:**

- **Search inputs:** Every keystroke must respond within 200ms.
- **Interactive dashboards:** Filters and toggles must feel instant.
- **Forms:** Validation and submission feedback must be immediate.

**References:**

- Vercel — INP optimization — https://vercel.com/kb/guide/how-to-improve-core-web-vitals
- Steve Kinney — Core Web Vitals for React Applications — https://stevekinney.com/courses/react-performance/core-web-vitals-for-react
- GitHub — INP yielding and long task guidance — https://github.com/mcorbett51090/RavenClaude

---

## 8. Performance Budgets

### Definitions

**Core Definition:** Performance budgets are explicit constraints on metrics that affect user experience, enforced to prevent regressions.

**Technical Definition:** Performance budgets are constraints set on various metrics — bundle size, loading milestones, runtime timings, and network characteristics — that turn vague goals into concrete, measurable targets. They are enforced in CI/CD pipelines to catch regressions before production.

**Beginner-Friendly Explanation:** Performance budgets are limits you set for your app, like “the main bundle must stay under 200KB” or “LCP must stay under 2.5 seconds.” If a change exceeds the budget, the build fails.

### Purposes

- To prevent performance regressions from reaching production.
- To make performance a first-class engineering constraint.
- To provide objective criteria for PR reviews.
- To create a culture of performance accountability.

### Syntax Rules and Structure

**Budget Categories:**

| Category | Example Budget |
|----------|----------------|
| Bundle size | Main JS ≤ 200KB; CSS ≤ 50KB |
| Loading | LCP ≤ 2.5s; FCP ≤ 1.8s |
| Runtime | Component render ≤ 16ms |
| Network | Max 50 requests; total transfer ≤ 1MB |

**Syntax Rules:**

1. Set budgets based on baseline measurements, not arbitrary numbers.
2. Enforce budgets in CI using tools like Lighthouse CI or bundle analyzers.
3. Allow per-route budgets for large applications.
4. Include a percentage increase limit (e.g., max 5% increase per PR).

### Annotated Code Examples

```json
// lighthouse-budget.json
[
  {
    "path": "/*",
    "timings": [
      { "metric": "largest-contentful-paint", "budget": 2500 },
      { "metric": "cumulative-layout-shift", "budget": 0.1 }
    ],
    "resourceSizes": [
      { "resourceType": "script", "budget": 200 },
      { "resourceType": "stylesheet", "budget": 50 }
    ]
  }
]
```

**Expected Output:** Lighthouse CI fails the build if any metric exceeds its budget.

**Why this output:** The budget file defines hard limits. The CI runner compares measured values against these limits and exits with a non-zero code on violation.

### Real-World Cases

- **Large teams:** Budgets prevent well-intentioned features from degrading performance.
- **Open-source libraries:** Bundle size budgets keep the library lightweight.
- **Enterprise apps:** Per-route budgets keep critical paths fast.

### References

- Steve Kinney — Performance Budgets & Automation — https://stevekinney.com/courses/react-performance/performance-budgets-and-automation
- GitHub — Performance budgets and CI enforcement — https://github.com/amritmalla/claude-full-stack-2.0
- npm — @samithahansaka/perflock — https://www.npmjs.com/package/@samithahansaka/perflock

---

## 9. Performance Profiling

### Definitions

**Core Definition:** Performance profiling is the process of measuring where an application spends its time in order to identify bottlenecks.

**Technical Definition:** Profiling uses tools to record and analyze the execution of an application, capturing component render durations, JavaScript execution, main-thread tasks, and paint timings. In React, the **React DevTools Profiler** records component renders and their durations; the **Chrome Performance tab** provides a deeper runtime analysis including JavaScript execution, layout, and paint.

**Beginner-Friendly Explanation:** Profiling is how you find out what is slow. You record your app while using it, and the profiler shows you which components took the longest and why.

### Purposes

- To identify slow components and unnecessary re-renders.
- To understand why a component re-rendered.
- To measure the impact of an optimization.
- To avoid guessing and optimize based on data.

### Syntax Rules and Structure

**Profiling Workflow:**

| Step | Action |
|------|--------|
| 1. Reproduce | Document exact steps, environment, and baseline metrics |
| 2. Profile | Record with React DevTools Profiler and Chrome Performance tab |
| 3. Hypothesize | Form a specific theory about the cause |
| 4. Fix | Apply the minimal fix |
| 5. Verify | Measure improvement under the same conditions |

**Syntax Rules:**

1. Always measure before and after an optimization.
2. Use the Profiler's “Ranked” view to find the worst offenders.
3. Check “Why did this render?” for each component.
4. Use the Chrome Performance tab when the Profiler shows *what* is slow but not *why*.

### Annotated Code Examples

```jsx
import { Profiler } from 'react';

function onRender(id, phase, actualDuration, baseDuration) {
  console.log({ id, phase, actualDuration, baseDuration });
}

function App() {
  return (
    <Profiler id="App" onRender={onRender}>
      <ExpensiveTree />
    </Profiler>
  );
}
```

**Expected Output:** Render durations are logged for the `App` tree, showing `actualDuration` and `baseDuration`.

**Why this output:** The `Profiler` component wraps a subtree and calls `onRender` with timing data after each commit, enabling programmatic profiling.

### Real-World Cases

- **Debugging jank:** Identifying long tasks during scrolling.
- **Optimizing lists:** Finding components that re-render unnecessarily.
- **Proving impact:** Demonstrating performance improvements to stakeholders.

### References

- Steve Kinney — Measuring Performance with Real Tools — https://stevekinney.com/courses/react-performance/measuring-performance-with-real-tools
- Steve Kinney — Debugging React Performance Issues — https://stevekinney.com/courses/react-performance/debugging-performance-issues
- CoreUI — How to measure performance in React — https://coreui.io

---

## 10. Performance Monitoring

### Definitions

**Core Definition:** Performance monitoring is the continuous collection and analysis of performance data from real users and synthetic tests in production.

**Technical Definition:** Performance monitoring combines **Real User Monitoring (RUM)** — collecting metrics from actual user sessions across devices, networks, and geographies — with **synthetic monitoring** — running scripted tests at fixed intervals to catch availability and performance issues before users encounter them. RUM reveals real-world performance; synthetic monitoring provides controlled, reproducible baselines.

**Beginner-Friendly Explanation:** Performance monitoring is how you keep an eye on your app's speed after it is deployed. You collect data from real users and run automated tests to catch problems early.

### Purposes

- To detect performance regressions in production.
- To understand performance across diverse devices and networks.
- To correlate performance with user engagement and conversion.
- To alert on-call engineers when metrics degrade.

### Syntax Rules and Structure

**Monitoring Approaches:**

| Approach | Data Source | Strength |
|----------|-------------|----------|
| RUM | Actual user sessions | Real-world conditions, all devices |
| Synthetic | Scripted tests | Controlled, reproducible, catches availability issues |

**Syntax Rules:**

1. Use the `web-vitals` library to collect LCP, INP, and CLS in production.
2. Send metrics via `navigator.sendBeacon` to avoid blocking unload.
3. Combine RUM with backend tracing for end-to-end visibility.
4. Sample RUM data to manage volume and cost.

### Annotated Code Examples

```jsx
import { onLCP, onINP, onCLS } from 'web-vitals';

function sendToAnalytics(metric) {
  navigator.sendBeacon('/analytics', JSON.stringify({
    name: metric.name,
    value: metric.value,
    id: metric.id,
  }));
}

onLCP(sendToAnalytics);
onINP(sendToAnalytics);
onCLS(sendToAnalytics);
```

**Expected Output:** Core Web Vitals are sent to an analytics endpoint for every user session.

**Why this output:** The `web-vitals` library observes each metric and calls the callback with the final value. `sendBeacon` ensures the data is sent even if the page is unloading.

### Real-World Cases

- **SaaS platforms:** RUM dashboards track performance across customer tenants.
- **E-commerce:** Synthetic tests run checkout flows from multiple regions.
- **Enterprise apps:** Alerting on INP spikes during business hours.

### References

- OneUptime — How to Implement RUM in React Applications — https://github.com/OneUptime/blog
- Carly Richmond — Observability for React Developers — https://gitnation.com/contents/observability-for-react-developers
- Dotcom-Monitor — Monitoring SPAs & Client-Side Routing — https://www.dotcom-monitor.com

---

## References

- MDN — Web performance — https://developer.mozilla.org/en-US/docs/Web/Performance
- Steve Kinney — Core Web Vitals for React Applications — https://stevekinney.com/courses/react-performance/core-web-vitals-for-react
- Steve Kinney — Measuring Performance with Real Tools — https://stevekinney.com/courses/react-performance/measuring-performance-with-real-tools
- Steve Kinney — Debugging React Performance Issues — https://stevekinney.com/courses/react-performance/debugging-performance-issues
- Steve Kinney — Performance Budgets & Automation — https://stevekinney.com/courses/react-performance/performance-budgets-and-automation
- Vercel — How to improve Core Web Vitals — https://vercel.com/kb/guide/how-to-improve-core-web-vitals
- Webkul — How to Improve Core Web Vitals in Next.js — https://webkul.com/blog/how-to-improve-core-web-vitals-in-next-js/
- OneUptime — How to Implement Real User Monitoring (RUM) in React Applications — https://github.com/OneUptime/blog
- Carly Richmond — Observability for React Developers — https://gitnation.com/contents/observability-for-react-developers
- CoreUI — How to measure performance in React — https://coreui.io
- GitHub — Core Web Vitals thresholds and INP guidance — https://github.com/agents-inc/skills
- GitHub — INP replaced FID (March 2024) — https://github.com/vasilyu1983/AI-Agents-public
- GitHub — LCP and CLS optimization patterns — https://github.com/pproenca/dot-skills
- GitHub — Performance budgets and CI enforcement — https://github.com/amritmalla/claude-full-stack-2.0
- GitHub — React performance profiling references — https://github.com/oakoss/agent-skills
- npm — @samithahansaka/perflock (component performance budgets) — https://www.npmjs.com/package/@samithahansaka/perflock
- Dotcom-Monitor — Monitoring SPAs & Client-Side Routing — https://www.dotcom-monitor.com
- GitHub — Web performance thresholds — https://github.com/mcorbett51090/RavenClaude