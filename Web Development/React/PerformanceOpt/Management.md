# React Performance Measurement: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** React performance measurement is the systematic practice of capturing, analysing, and acting on data about how a React application renders, commits, and interacts—using the React Profiler API, browser DevTools, Core Web Vitals, production tracing, and CI performance budgets.

**Technical Definition:** React performance measurement encompasses the instrumentation and tooling used to quantify rendering cost and user-perceived performance. It operates at four layers: **component-level profiling** (React Profiler API and React DevTools Profiler, which measure `actualDuration`, `baseDuration`, commit phases, and re-render reasons), **browser-level profiling** (Chrome DevTools Performance panel, which captures CPU profiles, flame charts, and long tasks with CPU/network throttling), **user-centric metrics** (Core Web Vitals—LCP, INP, CLS—measured with the `web-vitals` library in production), and **regression prevention** (performance budgets enforced in CI/CD pipelines that fail builds when bundle sizes or other thresholds are exceeded). Together, these layers provide the data needed to identify bottlenecks, validate optimisations, and prevent performance regressions.

**Beginner-Friendly Explanation:** Optimisation without measurement is guessing. React performance measurement is the set of tools that tell you exactly where your app is slow—which components take the longest to render, which commit phases block the browser, how long users wait for content to appear, and whether your latest change made things better or worse. The React Profiler shows you the component tree with timing data; Chrome DevTools shows you the browser-level flame chart; Core Web Vitals tell you what real users experience; and performance budgets stop regressions before they ship.

### Key Characteristics

- **Component-Level Granularity:** The React Profiler API measures per-component render time, commit phase, and memoization effectiveness.
- **Browser-Level Context:** Chrome DevTools reveals what the browser is doing during those renders—scripting, style recalculation, layout, and long tasks.
- **User-Centric Metrics:** Core Web Vitals (LCP, INP, CLS) measure what users actually experience, not just what the lab reports.
- **Production Tracing:** Real-user monitoring (RUM) captures performance data from actual devices and networks, not synthetic environments.
- **Budgets Prevent Regression:** Hard limits on bundle sizes and metrics, enforced in CI, catch performance degradation at the pull request level.
- **Layered Approach:** No single tool tells the whole story; component profiling, browser profiling, user metrics, and budgets are complementary.

### Prerequisites

- Solid understanding of React function components, JSX, and Hooks.
- Working knowledge of the render/commit lifecycle and re-render triggers.
- Familiarity with Chrome DevTools (Performance panel, Network panel).
- Basic understanding of Core Web Vitals and their thresholds.
- Awareness of CI/CD pipelines and build tooling (Vite, Webpack, Next.js).

### Related Programming Areas

- **Rendering Performance:** Re-render triggers, memoisation, and component boundaries.
- **Browser Performance:** Long tasks, main thread blocking, layout thrashing.
- **User Experience Metrics:** Core Web Vitals and real-user monitoring.
- **Build Engineering:** Bundle analysis, tree-shaking, and performance budgets.
- **Observability:** Production tracing, RUM, and error monitoring.

### Core Concepts / Features

1. The React Profiler API
2. Browser Performance Tools
3. User-Centric Metrics (Core Web Vitals)
4. Production Tracing
5. Performance Budgets & CI/CD

---

## Core Concept 1: The React Profiler API

### Definitions

**Core Definition:** The React Profiler API (`<Profiler>`) is a built-in React component that measures the rendering performance of a React tree programmatically, calling an `onRender` callback every time a component within the profiled tree commits an update.

**Technical Definition:** `<Profiler id="App" onRender={onRender}>` wraps a React tree and invokes the `onRender` callback with six parameters: `id` (the profiler's string identifier), `phase` (`"mount"`, `"update"`, or `"nested-update"`), `actualDuration` (milliseconds spent rendering the profiled subtree for the current update), `baseDuration` (estimated milliseconds to re-render the entire subtree without memoization), `startTime` (timestamp when React began rendering), and `commitTime` (timestamp when React committed the update). Profiling adds overhead and is disabled in production builds by default; a special profiling build is required for production measurement. The React DevTools Profiler provides an interactive UI for the same data, including flame graphs, ranked views, and re-render reasons.

**Beginner-Friendly Explanation:** The Profiler is like a stopwatch for your components. It tells you how long each component took to render, whether it was a first mount or an update, and how much time you saved by using `React.memo` or `useMemo`. The `actualDuration` is what actually happened; the `baseDuration` is what would have happened without any optimisation. If `actualDuration` is much smaller than `baseDuration`, your memoization is working. The React DevTools Profiler shows this as a colour-coded flame graph.

### Purposes

- To measure per-component render time programmatically.
- To identify components that take the longest to render.
- To verify that memoization is working (compare `actualDuration` to `baseDuration`).
- To detect unnecessary re-renders by examining the `phase` and render reasons.
- To measure the cost of layout effects (`useLayoutEffect`) and passive effects (`useEffect`) in the commit phase.
- To instrument production builds for real-user performance monitoring.

### Syntax Rules and Structure

**General Syntax:**
```jsx
import { Profiler } from 'react';

function onRender(id, phase, actualDuration, baseDuration, startTime, commitTime) {
  // Aggregate or log render timings
}

<Profiler id="App" onRender={onRender}>
  <App />
</Profiler>
```

**Component Breakdown:**
- `id`: A string identifying the profiled tree.
- `onRender`: Called every time a component within the tree commits an update.
- `phase`: `"mount"`, `"update"`, or `"nested-update"`.
- `actualDuration`: Milliseconds spent rendering the subtree for the current update.
- `baseDuration`: Estimated milliseconds without memoization.
- `startTime` / `commitTime`: Timestamps for the render and commit.

**Logging Render Timings:**
```jsx
function onRenderCallback(
  id,               // "App"
  phase,            // "mount" | "update" | "nested-update"
  actualDuration,   // ms spent rendering
  baseDuration,     // ms without memoization
  startTime,        // timestamp
  commitTime        // timestamp
) {
  console.log(`${id} (${phase}): ${actualDuration.toFixed(2)}ms (base: ${baseDuration.toFixed(2)}ms)`);
}

<Profiler id="App" onRender={onRenderCallback}>
  <App />
</Profiler>
```

**Measuring Multiple Subtrees:**
```jsx
<App>
  <Profiler id="Sidebar" onRender={onRenderCallback}>
    <Sidebar />
  </Profiler>
  <Profiler id="Content" onRender={onRenderCallback}>
    <Content />
  </Profiler>
</App>
```

**Component Breakdown:**
- Multiple `<Profiler>` components can measure different parts of the tree.
- Each call to `onRender` includes the profiler's `id`, allowing you to attribute timings.

**Production Profiling Build (React 18):**
```bash
# React 18 production profiling build
npm install react@profiling react-dom@profiling
```

**Component Breakdown:**
- The `profiling` build includes the Profiler overhead in production.
- Use it only for performance monitoring, not for end-user delivery.

**Syntax Rules:**
- Wrap a component tree in `<Profiler>` with a unique `id` and an `onRender` callback.
- Use multiple `<Profiler>` components for different subtrees.
- Compare `actualDuration` to `baseDuration` to verify memoization.
- Use the `phase` to distinguish mount from update renders.
- Profiling adds overhead; it is disabled in production builds by default.
- For production profiling, use the special `react@profiling` build.

**Constraints and Limitations:**
- Profiling adds memory and CPU overhead; do not leave it enabled in production without the profiling build.
- The Profiler does not tell you *why* a component re-rendered (use React DevTools for that).
- `baseDuration` is an estimate, not an exact measurement.
- The Profiler measures render time, not commit time (layout effects, DOM mutations).
- In React 19, the Profiler API is stable, but the React Compiler may reduce the need for manual profiling.

### Annotated Code Examples

**Example 1: Comparing Memoised and Non-Memoised Subtree**

```jsx
import { Profiler, useState, memo, useMemo } from 'react';

const MemoisedList = memo(function MemoisedList({ items, filter }) {
  const filtered = useMemo(
    () => items.filter(i => i.name.includes(filter)),
    [items, filter]
  );
  return (
    <ul>
      {filtered.map(item => <li key={item.id}>{item.name}</li>)}
    </ul>
  );
});

function NonMemoisedList({ items, filter }) {
  const filtered = items.filter(i => i.name.includes(filter));
  return (
    <ul>
      {filtered.map(item => <li key={item.id}>{item.name}</li>)}
    </ul>
  );
}

function onRender(id, phase, actualDuration, baseDuration) {
  console.log(`${id} (${phase}): ${actualDuration.toFixed(2)}ms / base: ${baseDuration.toFixed(2)}ms`);
}

const ITEMS = Array.from({ length: 5000 }, (_, i) => ({ id: i, name: `Item ${i}` }));

export default function App() {
  const [filter, setFilter] = useState('');

  return (
    <div>
      <input value={filter} onChange={e => setFilter(e.target.value)} />
      <Profiler id="Memoised" onRender={onRender}>
        <MemoisedList items={ITEMS} filter={filter} />
      </Profiler>
      <Profiler id="NonMemoised" onRender={onRender}>
        <NonMemoisedList items={ITEMS} filter={filter} />
      </Profiler>
    </div>
  );
}
```

**Expected Output (console):**
```
Memoised (mount): 12.40ms / base: 45.20ms
NonMemoised (mount): 44.80ms / base: 44.80ms
--- typing "a" ---
Memoised (update): 3.20ms / base: 45.20ms
NonMemoised (update): 44.50ms / base: 44.50ms
```

**Why This Output Occurs:** The memoised list uses `useMemo` to cache the filtered result, so `actualDuration` is much lower than `baseDuration` on updates. The non-memoised list recomputes the filter on every render, so `actualDuration` stays close to `baseDuration`.

### Real-World Cases

- **Dashboard widgets:** Profiling each widget to ensure one slow chart does not block the entire dashboard.
- **Data tables:** Measuring sort/filter operations to verify memoization.
- **E-commerce product grids:** Profiling filter and sort interactions.
- **Real-time feeds:** Measuring the cost of appending new items.
- **Production monitoring:** Using the profiling build to capture render timings from real users.

### References

- React Official Documentation – `<Profiler>`: https://react.dev/reference/react/Profiler
- React Official Documentation – Profiler `onRender` Callback: https://react.dev/reference/react/Profiler#onrender-callback
- React Official Documentation – React Developer Tools: https://react.dev/learn/react-developer-tools
- React 18 Profiling Build: https://fb.me/react-profiling
- RFC – Profiler Measure Commit Durations: https://raw.githubusercontent.com/reactjs/rfcs/ab1b62deb91273427c2058239ddf92500b059cb6/text/0000-profiler-measure-commit-durations.md

---

## Core Concept 2: Browser Performance Tools

### Definitions

**Core Definition:** Browser performance tools are the built-in DevTools features—primarily Chrome DevTools' Performance panel—that capture and visualise the browser's main-thread activity, including CPU profiles, flame charts, long tasks, and rendering events.

**Technical Definition:** Chrome DevTools' Performance panel records a timeline of browser activity, including JavaScript execution, style recalculation, layout, paint, and compositing. The **flame chart** visualises the call stack over time, with the x-axis representing time and the y-axis representing the call hierarchy. **Long tasks** (any task exceeding 50ms) are highlighted with a red triangle and red shading. **CPU throttling** (e.g., 4× or 20× slowdown) simulates slower devices, revealing performance issues that are imperceptible on high-end hardware. **Network throttling** simulates slow connections. The panel also provides a **Bottom-Up** view (aggregated by function) and **Call Tree** view (hierarchical), making it possible to identify the exact function responsible for a long task.

**Beginner-Friendly Explanation:** Chrome DevTools' Performance panel is like a security camera for your browser. It records everything the browser does—running JavaScript, calculating styles, laying out the page, painting—and shows it as a timeline. The flame chart shows which functions called which, and how long each took. Red triangles mark "long tasks"—anything over 50ms that blocks the browser and makes the page feel laggy. CPU throttling lets you test on a simulated slow device without buying one.

### Purposes

- To capture a complete timeline of browser activity during a user interaction.
- To identify long tasks that block the main thread and delay interactivity.
- To simulate slow devices and networks with CPU and network throttling.
- To trace the call stack to the exact function causing a bottleneck.
- To measure the cost of style recalculation, layout, paint, and compositing.
- To correlate JavaScript execution with rendering work.

### Syntax Rules and Structure

**Recording a Performance Profile:**
```
1. Open Chrome DevTools (F12 or Cmd+Option+I).
2. Switch to the Performance panel.
3. Click the record button (circle).
4. Interact with the app (scroll, type, click).
5. Click stop.
6. Analyse the flame chart, Bottom-Up, and Call Tree views.
```

**Reading the Flame Chart:**
```
- X-axis: time
- Y-axis: call stack (top = events, bottom = deeper calls)
- Width of a bar: duration
- Colour: script (yellow), rendering (purple), painting (green), system (grey)
- Red triangle: long task (> 50ms)
- Red shading: the portion exceeding 50ms
```

**CPU Throttling:**
```
1. In the Performance panel, open Capture settings.
2. Set CPU throttling to 4× or 20× slowdown.
3. Record the interaction.
4. Compare with unthrottled results.
```

**Component Breakdown:**
- `4× slowdown`: Simulates a mid-range mobile device.
- `20× slowdown`: Simulates a low-end device.
- Always test with throttling to catch issues that high-end hardware hides.

**Identifying Long Tasks:**
```javascript
// Long tasks are highlighted in red in the flame chart.
// Look for the red triangle and the red-shaded portion.
// Click on the long task to see its call stack.
// The Bottom-Up view shows which functions contributed the most time.
```

**Component Breakdown:**
- A long task is any task > 50ms.
- Tasks > 50ms delay the next paint and make the UI feel unresponsive.
- Use the Bottom-Up view to find the function responsible.

**Syntax Rules:**
- Always record with CPU throttling (4× or 20×) to simulate real devices.
- Record in incognito mode to avoid extension interference.
- Enable "Screenshots" and "Web Vitals" in Capture settings for context.
- Use the flame chart to trace the call stack; use Bottom-Up to aggregate by function.
- Look for red triangles (long tasks) and purple bars (rendering work).
- Correlate long tasks with user interactions to find the trigger.

**Constraints and Limitations:**
- DevTools recording adds overhead; timings are approximate.
- Recording too long produces a large, unwieldy profile.
- DevTools performance is not identical to production; use RUM for real-user data.
- CPU throttling approximates but does not perfectly replicate a real device.
- The Performance panel shows browser-level activity, not React-specific component data.

### Annotated Code Examples

**Example: Recording and Analysing a Long Task**

```javascript
// A component that causes a long task due to synchronous heavy computation
function SlowComponent({ items }) {
  // ❌ This blocks the main thread for hundreds of milliseconds
  const result = items.map(item => {
    let sum = 0;
    for (let i = 0; i < 1000000; i++) sum += item.value;
    return sum;
  });

  return <div>{result.length} items processed</div>;
}
```

**DevTools Procedure:**
```
1. Open the Performance panel.
2. Set CPU throttling to 4× slowdown.
3. Click record.
4. Trigger the slow component (e.g., navigate to it).
5. Click stop.
6. In the flame chart, find the red triangle (long task).
7. Click the long task to see the call stack.
8. Use Bottom-Up to find the function consuming the most time.
```

**Expected Output:** The flame chart shows a long task (> 400ms) with the `map` callback dominating the time. The red triangle marks the task as exceeding 50ms. The Bottom-Up view shows the anonymous function inside `SlowComponent` as the top consumer.

**Why This Output Occurs:** The synchronous loop runs on the main thread and cannot be interrupted. The browser cannot paint or respond to input until the loop completes. CPU throttling amplifies the effect, making it easier to see.

### Real-World Cases

- **Scroll jank:** Recording a scroll interaction to find the function causing frame drops.
- **Typing lag:** Recording keystrokes to find long tasks in input handlers.
- **Slow navigation:** Recording route transitions to find blocking renders.
- **Animation stutter:** Recording animations to find layout thrashing.
- **Third-party scripts:** Identifying which third-party script is blocking the main thread.

### References

- Chrome DevTools – Performance Panel Reference: https://developer.chrome.com/docs/devtools/performance/reference
- Chrome DevTools – Analyze Runtime Performance: https://developer.chrome.com/docs/devtools/performance
- Chrome DevTools – CPU Throttling: https://developer.chrome.com/docs/devtools/performance/reference#cpu-throttling
- web.dev – Long Tasks: https://web.dev/articles/long-tasks
- web.dev – Optimize Long Tasks: https://web.dev/articles/optimize-long-tasks
- MDN Web Docs – Performance API: https://developer.mozilla.org/en-US/docs/Web/API/Performance_API

---

## Core Concept 3: User-Centric Metrics (Core Web Vitals)

### Definitions

**Core Definition:** Core Web Vitals are three standardised, user-centric metrics—Largest Contentful Paint (LCP), Interaction to Next Paint (INP), and Cumulative Layout Shift (CLS)—that measure loading performance, interactivity, and visual stability.

**Technical Definition:** Core Web Vitals are defined by Google and measured in the field using the `web-vitals` library. **LCP** measures the render time of the largest image, text block, or video visible in the viewport; a good score is ≤ 2.5 seconds. **INP** measures the latency of all qualifying interactions (clicks, taps, keyboard input) throughout the page's lifespan; a good score is ≤ 200 milliseconds. **CLS** measures the unexpected shifting of visible content during loading; a good score is ≤ 0.1. These metrics are collected from real users (field data) and reported at the 75th percentile. Lab tools (Lighthouse, Chrome DevTools) provide synthetic measurements, but field data is the authoritative source.

**Beginner-Friendly Explanation:** Core Web Vitals are the three numbers that tell you how users actually experience your site. LCP is "how fast does the main content appear?" (≤ 2.5s is good). INP is "how quickly does the page respond when I click or type?" (≤ 200ms is good). CLS is "does the page jump around while loading?" (≤ 0.1 is good). These are measured from real users, not just from your laptop on a fast connection.

### Purposes

- To measure what real users experience on real devices and networks.
- To identify loading, interactivity, and visual stability problems.
- To track performance over time and detect regressions.
- To improve SEO (Core Web Vitals are a Google ranking signal).
- To prioritise optimisation work based on actual user impact.

### Syntax Rules and Structure

**Installing the `web-vitals` Library:**
```bash
npm install web-vitals
```

**Measuring Core Web Vitals (React):**
```jsx
import { onLCP, onINP, onCLS, onFCP, onTTFB } from 'web-vitals';

function sendToAnalytics(metric) {
  const body = JSON.stringify({
    name: metric.name,
    value: metric.value,
    rating: metric.rating, // "good" | "needs-improvement" | "poor"
    delta: metric.delta,
    id: metric.id,
  });
  navigator.sendBeacon('/analytics', body);
}

export function reportWebVitals() {
  onLCP(sendToAnalytics);
  onINP(sendToAnalytics);
  onCLS(sendToAnalytics);
  onFCP(sendToAnalytics);
  onTTFB(sendToAnalytics);
}
```

**Component Breakdown:**
- `onLCP`, `onINP`, `onCLS`: Register callbacks for each metric.
- `metric.value`: The numeric value.
- `metric.rating`: `"good"`, `"needs-improvement"`, or `"poor"`.
- `navigator.sendBeacon`: Sends data without blocking navigation.

**Core Web Vitals Thresholds:**

| Metric | Good | Needs Improvement | Poor |
|---|---|---|---|
| **LCP** | ≤ 2.5s | 2.5s – 4.0s | > 4.0s |
| **INP** | ≤ 200ms | 200ms – 500ms | > 500ms |
| **CLS** | ≤ 0.1 | 0.1 – 0.25 | > 0.25 |

**Detecting the LCP Element:**
```jsx
function useLCPDetection() {
  useEffect(() => {
    if (process.env.NODE_ENV !== 'development') return;
    let lcpElement = null;
    new PerformanceObserver((list) => {
      const entries = list.getEntries();
      const lastEntry = entries[entries.length - 1];
      if (lastEntry?.element) {
        if (lcpElement) lcpElement.style.outline = '';
        lcpElement = lastEntry.element;
        lcpElement.style.outline = '3px solid red';
        console.log('LCP element:', lcpElement);
      }
    }).observe({ type: 'largest-contentful-paint', buffered: true });
  }, []);
}
```

**Component Breakdown:**
- `PerformanceObserver`: Observes LCP entries.
- `lastEntry.element`: The DOM element that is the LCP candidate.
- The element is outlined in red for visual identification.

**Syntax Rules:**
- Install `web-vitals` and call `onLCP`, `onINP`, `onCLS` in the app's entry point.
- Send metrics to your analytics service using `navigator.sendBeacon`.
- Measure in production, not just in the lab; field data is authoritative.
- Use the `rating` property to classify metrics as good, needs-improvement, or poor.
- For React apps, report Web Vitals after hydration and route changes.
- INP and CLS report after interaction or layout shift; LCP reports after the largest element renders.

**Constraints and Limitations:**
- Field data requires traffic; low-traffic sites may not have enough data.
- INP is a relatively new metric (replaced FID in March 2024); tooling is still evolving.
- CLS can be caused by late-loading content, fonts, images, and ads.
- LCP can be caused by slow server response, render-blocking resources, or slow resource load.
- Core Web Vitals measure the 75th percentile; the median user may have a better experience.

### Annotated Code Example: Reporting Core Web Vitals in React

```jsx
import { useEffect } from 'react';
import { onLCP, onINP, onCLS } from 'web-vitals';

function reportMetric(metric) {
  const { name, value, rating, delta, id } = metric;
  console.log(`${name}: ${value.toFixed(2)} (${rating})`);

  // Send to your analytics endpoint
  const body = JSON.stringify({ name, value, rating, delta, id });
  navigator.sendBeacon('/analytics/vitals', body);
}

export default function App() {
  useEffect(() => {
    onLCP(reportMetric);
    onINP(reportMetric);
    onCLS(reportMetric);
  }, []);

  return <div>{/* app content */}</div>;
}
```

**Expected Output (console):**
```
LCP: 1850.00 (good)
INP: 120.00 (good)
CLS: 0.05 (good)
```

**Why This Output Occurs:** `onLCP`, `onINP`, and `onCLS` register performance observers that fire when the respective metrics are available. `reportMetric` logs the value and rating, then sends the data to `/analytics/vitals` using `sendBeacon`, which does not block navigation.

### Real-World Cases

- **E-commerce:** Tracking LCP for product images, INP for add-to-cart interactions, CLS for layout stability.
- **SaaS dashboards:** Tracking INP for filter and sort interactions.
- **Marketing sites:** Tracking LCP for hero images and CLS for font loading.
- **News sites:** Tracking LCP for article content and CLS for ad insertion.
- **Mobile web:** Tracking all three metrics on real mobile devices and networks.

### References

- web-vitals – GitHub: https://github.com/GoogleChrome/web-vitals
- web.dev – Core Web Vitals: https://web.dev/articles/vitals
- web.dev – Interaction to Next Paint (INP): https://web.dev/articles/inp
- web.dev – Largest Contentful Paint (LCP): https://web.dev/articles/lcp
- web.dev – Cumulative Layout Shift (CLS): https://web.dev/articles/cls
- web.dev – Optimize Long Tasks: https://web.dev/articles/optimize-long-tasks
- Google Search Central – Core Web Vitals: https://developers.google.com/search/docs/appearance/core-web-vitals

---

## Core Concept 4: Production Tracing

### Definitions

**Core Definition:** Production tracing is the practice of capturing real-world performance profiles from production builds—using the React Profiler API's production profiling build, `useLayoutEffect` timing, or integrated performance monitors—to understand how the application performs for real users on real devices.

**Technical Definition:** Production tracing involves instrumenting the application to emit performance data from production builds. React provides a special `react@profiling` build that enables the Profiler API in production with minimal overhead. React's internal Profiler measures time spent in `useLayoutEffect` and `useEffect` by calling `performance.now()` before and after executing user code, and these durations are reported through the Profiler API (in the `onCommit` and `onPostCommit` callbacks, per RFC). Production tracing also includes Real User Monitoring (RUM) using `PerformanceObserver` to capture Long Tasks, `PerformanceLongTaskTiming`, and custom marks/measures. The data is aggregated and sent to an analytics backend for analysis.

**Beginner-Friendly Explanation:** Development profiling is useful, but production is where the real users are—on slower devices, slower networks, and with real data. Production tracing captures performance data from actual users. The React profiling build gives you component render timings from production. `useLayoutEffect` timing tells you how long your layout effects take. RUM captures long tasks and other browser-level metrics. This data tells you what is actually slow for real people, not just on your fast development machine.

### Purposes

- To measure component render times from real users in production.
- To measure the cost of `useLayoutEffect` and `useEffect` in the commit phase.
- To capture long tasks and other browser-level performance data from real users.
- To correlate performance data with user interactions and routes.
- To detect performance regressions in production before users complain.
- To provide data for continuous performance improvement.

### Syntax Rules and Structure

**Production Profiling Build (React 18):**
```bash
npm install react@profiling react-dom@profiling
```

**Component Breakdown:**
- The `profiling` build includes the Profiler API overhead in production.
- Use it for performance monitoring only; it should not be the primary production build for all users.

**Measuring Effect Durations (RFC Proposal):**
```jsx
<Profiler
  id="Navigation"
  onRender={recordRenderDurations}
  onCommit={recordLayoutEffectDurations}
  onPostCommit={recordEffectDurations}
>
  <Navigation />
</Profiler>
```

**Component Breakdown:**
- `onRender`: Render phase durations.
- `onCommit`: Layout effect (`useLayoutEffect`) durations.
- `onPostCommit`: Passive effect (`useEffect`) durations.
- React measures these by calling `performance.now()` before and after executing user code.

**Long Task Observer (RUM):**
```javascript
function observeLongTasks() {
  if (!('PerformanceObserver' in window)) return;

  const observer = new PerformanceObserver((list) => {
    for (const entry of list.getEntries()) {
      console.log('Long task:', entry.duration.toFixed(2), 'ms', entry);
      // Send to analytics
      navigator.sendBeacon('/analytics/long-tasks', JSON.stringify({
        duration: entry.duration,
        startTime: entry.startTime,
        name: entry.name,
      }));
    }
  });

  observer.observe({ type: 'longtask', buffered: true });
}
```

**Component Breakdown:**
- `PerformanceObserver`: Observes long task entries.
- `entry.duration`: The duration in milliseconds.
- `navigator.sendBeacon`: Sends data without blocking.

**Custom Performance Marks:**
```javascript
function measureInteraction(name, fn) {
  performance.mark(`${name}-start`);
  fn();
  performance.mark(`${name}-end`);
  performance.measure(name, `${name}-start`, `${name}-end`);
  const measure = performance.getEntriesByName(name)[0];
  console.log(`${name}: ${measure.duration.toFixed(2)}ms`);
}
```

**Component Breakdown:**
- `performance.mark`: Creates a timestamp.
- `performance.measure`: Measures the duration between two marks.
- `performance.getEntriesByName`: Retrieves the measurement.

**Syntax Rules:**
- Use the `react@profiling` build for production render profiling.
- Use `onCommit` and `onPostCommit` (when available) to measure effect durations.
- Use `PerformanceObserver` with `type: 'longtask'` for RUM.
- Use `performance.mark` and `performance.measure` for custom timings.
- Send data to analytics using `navigator.sendBeacon` to avoid blocking.
- Sample production tracing data to reduce overhead (e.g., 10% of users).
- Correlate performance data with user interactions and routes.

**Constraints and Limitations:**
- Production profiling adds overhead; sample or enable for a subset of users.
- The `onCommit` and `onPostCommit` callbacks are RFC proposals; check React version support.
- Long task observation is not supported in all browsers.
- Custom marks must be removed after measurement to avoid memory leaks.
- RUM data is noisy; aggregate and filter before analysis.

### Annotated Code Example: Production RUM with Long Task Observer

```javascript
// rum.js — production tracing utility
export function initPerformanceMonitoring() {
  // 1. Observe long tasks
  if ('PerformanceObserver' in window) {
    try {
      new PerformanceObserver((list) => {
        for (const entry of list.getEntries()) {
          if (entry.duration > 100) {
            navigator.sendBeacon('/analytics/long-tasks', JSON.stringify({
              duration: entry.duration,
              startTime: entry.startTime,
              name: entry.name,
              url: window.location.href,
            }));
          }
        }
      }).observe({ type: 'longtask', buffered: true });
    } catch (e) {
      // Long task observer not supported
    }
  }

  // 2. Observe resource timing
  new PerformanceObserver((list) => {
    for (const entry of list.getEntries()) {
      if (entry.duration > 1000) {
        navigator.sendBeacon('/analytics/slow-resources', JSON.stringify({
          name: entry.name,
          duration: entry.duration,
          type: entry.initiatorType,
        }));
      }
    }
  }).observe({ type: 'resource', buffered: true });

  // 3. Custom interaction mark
  window.addEventListener('click', (e) => {
    performance.mark('click-start');
    requestAnimationFrame(() => {
      performance.mark('click-end');
      performance.measure('click-response', 'click-start', 'click-end');
      const measure = performance.getEntriesByName('click-response')[0];
      if (measure.duration > 200) {
        navigator.sendBeacon('/analytics/slow-clicks', JSON.stringify({
          duration: measure.duration,
          target: e.target.tagName,
        }));
      }
    });
  });
}
```

**Expected Output:** Long tasks over 100ms, slow resources over 1s, and slow click responses over 200ms are sent to the analytics endpoint. The data can be aggregated to identify patterns (e.g., a specific page or interaction that consistently produces long tasks).

**Why This Output Occurs:** `PerformanceObserver` captures long tasks and resource timings as they occur. The custom click measurement uses `performance.mark` and `performance.measure` to time the response to a click. `navigator.sendBeacon` sends the data without blocking the main thread.

### Real-World Cases

- **E-commerce:** Capturing long tasks during add-to-cart and checkout.
- **SaaS dashboards:** Measuring render times for dashboard widgets.
- **Media sites:** Capturing slow resource loads for images and videos.
- **Mobile web:** Capturing long tasks on low-end devices.
- **Single-page apps:** Capturing route transition performance.

### References

- React Official Documentation – `<Profiler>`: https://react.dev/reference/react/Profiler
- React Profiling Build: https://fb.me/react-profiling
- RFC – Profiler Measure Commit Durations: https://raw.githubusercontent.com/reactjs/rfcs/ab1b62deb91273427c2058239ddf92500b059cb6/text/0000-profiler-measure-commit-durations.md
- MDN Web Docs – PerformanceObserver: https://developer.mozilla.org/en-US/docs/Web/API/PerformanceObserver
- MDN Web Docs – Long Tasks API: https://developer.mozilla.org/en-US/docs/Web/API/PerformanceLongTaskTiming
- MDN Web Docs – `performance.mark()`: https://developer.mozilla.org/en-US/docs/Web/API/Performance/mark
- MDN Web Docs – `performance.measure()`: https://developer.mozilla.org/en-US/docs/Web/API/Performance/measure
- MDN Web Docs – `navigator.sendBeacon()`: https://developer.mozilla.org/en-US/docs/Web/API/Navigator/sendBeacon

---

## Core Concept 5: Performance Budgets & CI/CD

### Definitions

**Core Definition:** Performance budgets are hard limits on metrics such as bundle size, render time, or Core Web Vitals, enforced in CI/CD pipelines so that builds fail when the limits are exceeded, preventing performance regressions from reaching production.

**Technical Definition:** A performance budget defines maximum acceptable values for performance metrics (e.g., "main bundle ≤ 200KB gzipped," "LCP ≤ 2.5s," "component render ≤ 16ms"). In CI/CD, these budgets are enforced by tools that scan the built assets (`import-cost-enforcer`, `performance-budget-enforcer`, `bundlesize`), run Lighthouse CI, or analyse bundle stats (`webpack-bundle-analyzer`). When a budget is exceeded, the CI step exits with a non-zero code, failing the build and preventing the merge. Budgets can be absolute (maximum size) or relative (maximum percentage increase per PR). The goal is to catch regressions at the pull request level, before they compound.

**Beginner-Friendly Explanation:** Performance budgets are like a speed limit for your app. Without them, performance degrades slowly—every PR adds a few kilobytes, every feature adds a few milliseconds, and one day the app is slow. With budgets, CI fails when a PR exceeds the limit, forcing the developer to fix it before merging. It is like having a scale that stops you from boarding a plane if your luggage is too heavy.

### Purposes

- To prevent performance regressions from reaching production.
- To make performance a first-class concern in code review.
- To catch accidental bloat (new dependencies, large assets) at the PR level.
- To provide objective, measurable targets for performance work.
- To create a culture where performance is continuously monitored.

### Syntax Rules and Structure

**`import-cost-enforcer` Configuration:**
```json
{
  "distDir": "dist",
  "threshold": { "type": "percent", "limit": 30 },
  "metrics": ["brotli", "gzip"],
  "budgets": [
    { "target": "total", "maxKB": 800, "metric": "brotli" },
    { "target": "file:assets/*.js", "maxKB": 200, "metric": "gzip" }
  ],
  "ignore": ["**/*.map"],
  "report": { "jsonOut": "reports/import-cost.json" },
  "baselineIn": ".cache/baseline.json"
}
```

**Component Breakdown:**
- `threshold`: Maximum percentage increase per PR (30%).
- `budgets`: Absolute limits (800KB brotli total, 200KB gzip per JS file).
- `baselineIn`: Baseline file for comparison.
- `ignore`: Files to exclude from analysis.

**CI Workflow (GitHub Actions):**
```yaml
name: Performance Budget
on: [pull_request]

jobs:
  budget:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci
      - run: npm run build
      - name: Enforce import costs
        run: |
          npx import-cost-enforcer \
            --dist dist \
            --baseline .cache/baseline.json \
            --metrics gzip,brotli \
            --json-out reports/import-cost.json
```

**Component Breakdown:**
- Runs on every pull request.
- Builds the app and runs `import-cost-enforcer`.
- Exits with code 1 if budgets are exceeded, failing the CI check.

**Lighthouse CI Configuration:**
```json
{
  "ci": {
    "collect": {
      "url": ["https://preview.example.com/"],
      "numberOfRuns": 3
    },
    "assert": {
      "assertions": {
        "categories:performance": ["error", { "minScore": 0.9 }],
        "largest-contentful-paint": ["error", { "maxNumericValue": 2500 }],
        "interaction-to-next-paint": ["error", { "maxNumericValue": 200 }],
        "cumulative-layout-shift": ["error", { "maxNumericValue": 0.1 }]
      }
    }
  }
}
```

**Component Breakdown:**
- `assertions`: Fails CI if LCP > 2.5s, INP > 200ms, CLS > 0.1, or performance score < 90%.

**Syntax Rules:**
- Define budgets for bundle size (raw, gzip, brotli) and Core Web Vitals.
- Use a baseline file for relative comparisons (percentage increase per PR).
- Use absolute budgets for total or per-file limits.
- Run budget enforcement on every pull request.
- Fail the CI step (exit code 1) when budgets are exceeded.
- Report results in PR comments for visibility.
- Refresh the baseline intentionally after deliberate changes.

**Constraints and Limitations:**
- Budgets must be realistic; too strict causes developer friction.
- Bundle size is a proxy for performance, not a direct measure.
- Lighthouse CI requires a deployed preview URL; local builds are not sufficient.
- CI enforcement adds build time; balance thoroughness with speed.
- Budgets should be reviewed periodically as the app evolves.

### Annotated Code Example: Complete Performance Budget CI

```yaml
# .github/workflows/performance.yml
name: Performance Budget
on:
  pull_request:
    branches: [main]

jobs:
  bundle-budget:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'

      - run: npm ci
      - run: npm run build

      - name: Check bundle size
        run: |
          npx import-cost-enforcer \
            --dist dist \
            --baseline .cache/baseline.json \
            --metrics gzip,brotli \
            --json-out reports/bundle-budget.json

      - name: Upload bundle report
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: bundle-budget-report
          path: reports/bundle-budget.json
```

```json
// .cache/baseline.json (committed to the repo)
{
  "total": { "gzip": 245, "brotli": 198 },
  "files": {
    "assets/main.js": { "gzip": 120, "brotli": 95 },
    "assets/vendor.js": { "gzip": 125, "brotli": 103 }
  }
}
```

**Expected Output:** The CI step builds the app and runs `import-cost-enforcer`. If the main bundle exceeds 120KB gzip or the total exceeds 245KB gzip, the step fails with a clear report. The report is uploaded as an artifact for review.

**Why This Output Occurs:** `import-cost-enforcer` compares the current build against the baseline and the configured budgets. If any budget is exceeded, it exits with code 1, failing the CI check. The baseline is committed to the repo and updated intentionally (using `--write-baseline`) when the size increase is justified.

### Real-World Cases

- **E-commerce:** Enforcing a 200KB gzip budget for the main bundle.
- **SaaS dashboards:** Enforcing LCP ≤ 2.5s and INP ≤ 200ms in Lighthouse CI.
- **Marketing sites:** Enforcing CLS ≤ 0.1 and total transfer ≤ 1MB.
- **Design systems:** Enforcing per-component bundle size limits.
- **Monorepos:** Enforcing budgets per package.

### References

- import-cost-enforcer – npm: https://www.npmjs.com/package/import-cost-enforcer
- performance-budget-enforcer – Snyk: https://security.snyk.io/package/npm/performance-budget-enforcer/1.0.0
- Lighthouse CI: https://github.com/GoogleChrome/lighthouse-ci
- webpack-bundle-analyzer: https://github.com/webpack-contrib/webpack-bundle-analyzer
- bundlesize: https://github.com/siddharthkp/bundlesize
- Steve Kinney – Performance Budgets & Automation: https://stevekinney.com/courses/react-performance/performance-budgets-and-automation
- web.dev – Performance Budgets: https://web.dev/articles/performance-budgets-101

---

## Comparison and Decision Guidance

| Tool/Metric | What It Measures | When to Use | Key Output |
|---|---|---|---|
| **React Profiler API** | Per-component render time | Development + production profiling | `actualDuration`, `baseDuration` |
| **React DevTools Profiler** | Component render time (interactive) | Development | Flame graph, ranked view, render reasons |
| **Chrome DevTools Performance** | Browser-level activity | Development + testing | Flame chart, long tasks, CPU profiles |
| **Core Web Vitals** | User-perceived performance | Production | LCP, INP, CLS (field data) |
| **Production Tracing** | Real-user render + commit times | Production | Effect durations, long tasks, custom marks |
| **Performance Budgets** | Regression prevention | CI/CD | Build failures on budget exceed |

**Decision Guidance:**
- **Start with React DevTools Profiler** for component-level investigation.
- **Use the React Profiler API** for programmatic measurement and production profiling.
- **Use Chrome DevTools Performance** for browser-level bottlenecks (long tasks, layout thrashing).
- **Measure Core Web Vitals in production** with `web-vitals` to understand real-user experience.
- **Use production tracing** (profiling build, Long Task Observer, custom marks) for real-user data.
- **Enforce performance budgets in CI** to prevent regressions.
- **Combine tools:** Profiler for components, DevTools for browser, Core Web Vitals for users, budgets for prevention.
- **Measure before and after every optimisation**; remove changes that do not help.

---

## References

- React Official Documentation – `<Profiler>`: https://react.dev/reference/react/Profiler
- React Official Documentation – Profiler `onRender` Callback: https://react.dev/reference/react/Profiler#onrender-callback
- React Official Documentation – React Developer Tools: https://react.dev/learn/react-developer-tools
- React Profiling Build: https://fb.me/react-profiling
- RFC – Profiler Measure Commit Durations: https://raw.githubusercontent.com/reactjs/rfcs/ab1b62deb91273427c2058239ddf92500b059cb6/text/0000-profiler-measure-commit-durations.md
- Chrome DevTools – Performance Panel Reference: https://developer.chrome.com/docs/devtools/performance/reference
- Chrome DevTools – Analyze Runtime Performance: https://developer.chrome.com/docs/devtools/performance
- web.dev – Long Tasks: https://web.dev/articles/long-tasks
- web.dev – Optimize Long Tasks: https://web.dev/articles/optimize-long-tasks
- web.dev – Core Web Vitals: https://web.dev/articles/vitals
- web.dev – Interaction to Next Paint (INP): https://web.dev/articles/inp
- web.dev – Largest Contentful Paint (LCP): https://web.dev/articles/lcp
- web.dev – Cumulative Layout Shift (CLS): https://web.dev/articles/cls
- web.dev – Performance Budgets: https://web.dev/articles/performance-budgets-101
- web-vitals – GitHub: https://github.com/GoogleChrome/web-vitals
- Lighthouse CI: https://github.com/GoogleChrome/lighthouse-ci
- import-cost-enforcer – npm: https://www.npmjs.com/package/import-cost-enforcer
- performance-budget-enforcer – Snyk: https://security.snyk.io/package/npm/performance-budget-enforcer/1.0.0
- webpack-bundle-analyzer: https://github.com/webpack-contrib/webpack-bundle-analyzer
- bundlesize: https://github.com/siddharthkp/bundlesize
- MDN Web Docs – PerformanceObserver: https://developer.mozilla.org/en-US/docs/Web/API/PerformanceObserver
- MDN Web Docs – Long Tasks API: https://developer.mozilla.org/en-US/docs/Web/API/PerformanceLongTaskTiming
- MDN Web Docs – `performance.mark()`: https://developer.mozilla.org/en-US/docs/Web/API/Performance/mark
- MDN Web Docs – `performance.measure()`: https://developer.mozilla.org/en-US/docs/Web/API/Performance/measure
- MDN Web Docs – `navigator.sendBeacon()`: https://developer.mozilla.org/en-US/docs/Web/API/Navigator/sendBeacon
- Steve Kinney – Measuring Performance with Real Tools: https://stevekinney.com/courses/react-performance/measuring-performance-with-real-tools
- Steve Kinney – Core Web Vitals for React Applications: https://stevekinney.com/courses/react-performance/core-web-vitals-for-react
- Steve Kinney – Performance Budgets & Automation: https://stevekinney.com/courses/react-performance/performance-budgets-and-automation
- Steve Kinney – Performance Characteristics of useLayoutEffect: https://stevekinney.com/courses/react-performance/uselayouteffect-performance