# JavaScript Performance Debugging — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Performance debugging is the systematic process of measuring, analysing, and optimising the runtime performance of JavaScript applications — including CPU usage, memory consumption, rendering, and network activity — using browser DevTools and profiling tools.

**Technical Definition:** Per the W3C Performance Timeline specification (and the WHATWG/Chrome DevTools Protocol), performance debugging relies on several observable interfaces: the Performance Timeline (`performance.now()`, `performance.mark()`, `performance.measure()`), the Navigation Timing API, the User Timing API, the Resource Timing API, and the Long Tasks API. Browser DevTools implement performance profiling via the Chrome DevTools Protocol (CDP) — specifically the `Profiler`, `HeapProfiler`, `Tracing`, and `Performance` domains. The Performance panel records a trace of events (function calls, layout, paint, GC, network) and visualises them as a flame chart, call tree, and bottom-up summary. The Memory panel captures heap snapshots, allocation timelines, and allocation sampling data for diagnosing memory issues.

**Beginner-Friendly Explanation:** Performance debugging is finding out why your app feels slow. Maybe a function takes too long, the page stutters during animation, or memory keeps growing until the tab crashes. DevTools gives you tools to record what your app is doing, see where time is spent, and identify the culprits — like a slow function in a loop, excessive DOM updates, or a memory leak. Once you've found the problem, you can fix it and measure again to confirm the improvement.

### Key Characteristics

- **Measurement-driven:** You can't optimise what you don't measure.
- **Multi-dimensional:** CPU, memory, rendering, network, and I/O all matter.
- **Trace-based:** Performance panel records events over time.
- **Flame charts:** Visualise call stacks over time.
- **Frames and FPS:** Target 60 FPS (16.67ms per frame) for smooth UI.
- **Throttling:** Simulate slow CPUs and networks.
- **Memory-focused:** Heap snapshots and allocation timelines for leaks.
- **Iterative:** Measure, optimise, measure again.

### Prerequisites

- Solid JavaScript fundamentals (functions, closures, arrays).
- Understanding of the event loop and rendering pipeline.
- Familiarity with DevTools panels (Sources, Console, Network).
- Awareness of Big-O notation and algorithmic complexity.
- Basic knowledge of memory management and garbage collection.

### Related Programming Areas

- **Rendering performance:** Layout, paint, composite.
- **JavaScript performance:** V8 optimisation, JIT.
- **Memory management:** Heap, GC, leaks.
- **Network performance:** Latency, bandwidth, caching.
- **Core Web Vitals:** LCP, INP, CLS.
- **Observability:** RUM, synthetic monitoring.

---

## Performance Panel

### Definitions

**Core Definition:** The Performance panel is DevTools' primary tool for recording and analysing runtime performance traces.

**Technical Definition:** The Performance panel (Chrome DevTools) records a trace using the CDP `Tracing` domain. It captures events from the browser's main thread, compositor threads, GPU, and network. The recording is visualised as a flame chart (main thread activity over time), a call tree, a bottom-up summary, and an event log. Key metrics include FPS, CPU usage, network activity, and long tasks.

**Beginner-Friendly Explanation:** The Performance panel records everything your app does while you interact with it, then shows you a timeline where you can see slow spots.

### Opening the Panel

- **Chrome/Edge:** DevTools → Performance tab.
- **Firefox:** DevTools → Performance tab.
- **Safari:** Develop menu → Show Web Inspector → Timelines.

### Key Sections

- **Controls:** Record, reload, clear.
- **Overview:** FPS, CPU, NET charts.
- **Flame chart:** Main thread, compositor, GPU, network.
- **Bottom-up / Call tree / Event log:** Analysis views.
- **Details:** Selected event info.

### Real-World Cases

- **Slow page load:** Record reload and inspect.
- **Janky scrolling:** Record scroll and find long tasks.
- **Slow interactions:** Record click and find bottlenecks.

---

## Performance Profiling

### Definitions

**Core Definition:** Performance profiling is the practice of recording a trace of your application's runtime behaviour and analysing it to identify bottlenecks.

**Technical Definition:** Profiling in the Performance panel records a trace of events: function calls (`FunctionCall`), script evaluation (`EvaluateScript`), layout (`Layout`), paint (`Paint`), style recalculation (`RecalculateStyles`), garbage collection (`GC`), timers, and network requests. Each event has a timestamp, duration, and stack. The resulting trace is visualised as a flame chart.

**Beginner-Friendly Explanation:** Profiling means recording what your app does while it runs, so you can see where time is spent.

### Profiling Workflow

1. **Open Performance panel.**
2. **Click Record.**
3. **Interact with the page** (reload, click, scroll).
4. **Stop recording.**
5. **Analyse** the flame chart, call tree, bottom-up.
6. **Identify** long tasks, repeated work, layout thrashing.
7. **Fix** the issue.
8. **Re-record** to confirm improvement.

### Annotated Code Examples

```js
// Code to profile: an inefficient function.
function slowFunction() {
  const result = [];
  for (let i = 0; i < 100000; i++) {
    result.push(Math.sqrt(i));
  }
  return result;
}

// Add a mark for easy identification in the trace.
performance.mark("slowFunction:start");
slowFunction();
performance.mark("slowFunction:end");
performance.measure("slowFunction", "slowFunction:start", "slowFunction:end");

// View the measurement:
console.log(performance.getEntriesByName("slowFunction")[0].duration);
```

**Expected Output (approximate):**
```
50.25 (ms)
```

**Why This Helps:** `performance.mark` and `performance.measure` create custom events visible in the Performance panel, making it easy to locate specific functions.

### Real-World Cases

- **Initial load:** Profile the load sequence.
- **User interactions:** Profile click/input handlers.
- **Animations:** Profile `requestAnimationFrame` callbacks.

---

## Flame Charts

### Definitions

**Core Definition:** A flame chart is a visualisation of call stacks over time, with time on the horizontal axis and stack depth on the vertical axis.

**Technical Definition:** In the Performance panel, the flame chart (also called the "Main" track) shows the main thread's activity. Each bar represents a function call, with width proportional to duration and vertical position representing stack depth. Colours indicate categories (scripting, rendering, painting, system). Aggregated repeated calls may be collapsed. The chart is interactive: hover for details, click to select.

**Beginner-Friendly Explanation:** A flame chart is a picture of what your code was doing over time. Wider bars mean more time; taller stacks mean deeper calls.

### How to Read

- **Horizontal axis:** Time.
- **Vertical axis:** Call stack depth (top = outermost, bottom = innermost).
- **Bar width:** Duration.
- **Colour:** Category (yellow = scripting, purple = rendering, green = painting, grey = system).
- **Zoom:** Click and drag to zoom into a time range.

### Annotated Code Examples

```js
function outer() {
  for (let i = 0; i < 3; i++) {
    inner(i);
  }
}

function inner(n) {
  const arr = new Array(10000).fill(0);
  return arr.map((_, i) => i * n);
}

// In the flame chart, you'll see:
// outer (wide bar)
//   inner (three bars, one per iteration)
//     Array.fill, map (nested bars)
outer();
```

### Real-World Cases

- **Long tasks:** Find wide bars.
- **Repeated work:** Find repeated similar bars.
- **Deep recursion:** Find tall stacks.

---

## Call Tree

### Definitions

**Core Definition:** The Call Tree view aggregates trace events by function call hierarchy, showing total and self time.

**Technical Definition:** The Call Tree (in the Performance panel's Bottom-Up/Call Tree/Event Log tabs) shows a hierarchical view of function calls. Each node has a "Total Time" (including children) and "Self Time" (excluding children). Sorting by self time reveals hotspots.

**Beginner-Friendly Explanation:** The Call Tree shows you which functions took the most time, including how much time was spent in the function itself vs. its children.

### Key Metrics

- **Total Time:** Time in the function plus all descendants.
- **Self Time:** Time in the function excluding descendants.
- **Aggregated:** Repeated calls are grouped.

### Annotated Code Examples

```js
function a() {
  b();
  c();
}
function b() {
  for (let i = 0; i < 1e6; i++) {} // Self time
}
function c() {
  for (let i = 0; i < 5e5; i++) {} // Self time
}

a();
```

**Call Tree (approximate):**
```
a               Total: 10ms   Self: 0ms
├── b           Total: 6ms    Self: 6ms
└── c           Total: 4ms    Self: 4ms
```

### Real-World Cases

- **Hotspot identification:** Sort by self time.
- **Library overhead:** Identify library functions with high total time.
- **Refactoring targets:** Focus on high-self-time functions.

---

## Bottom-Up View

### Definitions

**Core Definition:** The Bottom-Up view shows which functions contributed most to a selected activity, aggregated across the trace.

**Technical Definition:** Bottom-Up aggregates trace events by function, regardless of stack depth. Selecting a function shows its callers (who called it) and callees (who it called). This is useful for identifying which top-level functions dominate time.

**Beginner-Friendly Explanation:** Bottom-Up tells you "this function is responsible for X% of the time" and lets you drill into who called it.

### Annotated Code Examples

```
// Bottom-Up view (aggregated):
Function        Self Time   Total Time
map             45ms        50ms
sqrt            30ms        30ms
render          20ms        80ms
```

### Real-World Cases

- **Aggregated hotspots:** Find functions that appear many times.
- **Caller analysis:** See who calls a slow function.
- **Library hotspots:** Identify expensive library calls.

---

## Event Log

### Definitions

**Core Definition:** The Event Log view lists trace events in chronological order.

**Technical Definition:** The Event Log shows every recorded event (function calls, layout, paint, GC, timers) with timestamps and durations. It is useful for understanding the exact sequence of events.

**Beginner-Friendly Explanation:** The Event Log is a chronological list of everything that happened during the recording.

### Annotated Code Examples

```
Timestamp   Duration   Event
0.0ms       0.5ms      EvaluateScript
0.5ms       2.0ms      FunctionCall (slowFunction)
2.5ms       5.0ms      Layout
7.5ms       3.0ms      Paint
10.5ms      0.2ms      GC
```

### Real-World Cases

- **Sequence analysis:** Understand ordering.
- **GC identification:** Find garbage collection events.
- **Layout/paint timing:** See rendering costs.

---

## Frames

### Definitions

**Core Definition:** Frames are the individual images rendered by the browser, typically at 60 FPS (16.67 ms per frame).

**Technical Definition:** In the Performance panel, the "Frames" track shows each rendered frame. Green frames meet the frame budget; red frames exceed it (jank). Frames are produced by the rendering pipeline (JavaScript → style → layout → paint → composite).

**Beginner-Friendly Explanation:** A frame is one image on screen. To feel smooth, your app should render 60 frames per second.

### Frame Budget

- **60 FPS:** 16.67 ms per frame.
- **30 FPS:** 33.33 ms per frame.
- **120 FPS:** 8.33 ms per frame (high-refresh displays).

### Real-World Cases

- **Animation jank:** Red frames indicate dropped frames.
- **Scroll performance:** Smooth scrolling requires consistent frames.
- **Input latency:** Long frames delay interaction response.

---

## FPS

### Definitions

**Core Definition:** FPS (frames per second) measures how many frames the browser renders per second.

**Technical Definition:** In the Performance panel, the FPS chart shows the frame rate over time. Green bars indicate frames at or above 60 FPS; red bars indicate dropped frames. FPS is affected by JavaScript execution, layout, paint, and compositing.

**Beginner-Friendly Explanation:** FPS is a measure of smoothness. 60 FPS is smooth; anything below feels janky.

### Real-World Cases

- **Games:** Target 60 FPS.
- **Scrolling:** Smooth scroll requires consistent FPS.
- **Animations:** CSS animations often outperform JS animations.

---

## CPU Throttling

### Definitions

**Core Definition:** CPU throttling simulates a slower CPU to test performance on lower-end devices.

**Technical Definition:** In the Performance panel, CPU throttling (via CDP `Emulation.setCPUThrottlingRate`) slows JavaScript execution by a factor (e.g., 4× or 6×). This simulates mid-tier or low-end mobile devices.

**Beginner-Friendly Explanation:** CPU throttling makes your computer act slower so you can see how your app performs on a slow phone.

### How to Use

1. Open Performance panel.
2. Click the "CPU" dropdown.
3. Select a throttling rate (4× or 6×).
4. Record and analyse.

### Real-World Cases

- **Mobile testing:** Simulate low-end devices.
- **Battery mode:** Test under CPU constraints.
- **Cross-device parity:** Ensure performance on all devices.

---

## Network Throttling

### Definitions

**Core Definition:** Network throttling simulates slow network conditions to test loading performance.

**Technical Definition:** In the Network panel (and Performance panel), network throttling (via CDP `Network.emulateNetworkConditions`) limits bandwidth and latency. Presets include "Slow 3G", "Fast 3G", and custom profiles.

**Beginner-Friendly Explanation:** Network throttling makes your internet connection act slower so you can test how your app loads on slow networks.

### Real-World Cases

- **Mobile networks:** Test on 3G/4G.
- **Rural areas:** Test on slow connections.
- **International users:** Simulate high latency.

---

## Memory Profiling

### Definitions

**Core Definition:** Memory profiling is the practice of measuring and analysing memory usage to find leaks, bloat, and inefficiencies.

**Technical Definition:** The Memory panel in DevTools provides three profiling tools: Heap Snapshot (memory distribution across objects), Allocation Instrumentation on Timeline (record allocations over time), and Allocation Sampling (profile allocations by function). It uses the CDP `HeapProfiler` domain.

**Beginner-Friendly Explanation:** Memory profiling shows you how much memory your app uses and what's using it.

### Tools

- **Heap Snapshot:** Point-in-time view of the heap.
- **Allocation Timeline:** Record allocations over time.
- **Allocation Sampling:** Profile allocations by function.

### Real-World Cases

- **Memory leaks:** Growing heap over time.
- **Bloat:** Large retained objects.
- **Optimisation:** Reduce allocation frequency.

---

## Heap Snapshots

### Definitions

**Core Definition:** A heap snapshot is a point-in-time capture of all objects in the JavaScript heap.

**Technical Definition:** A heap snapshot (via CDP `HeapProfiler.takeHeapSnapshot`) records every reachable object, its type, size, and retainers. DevTools renders snapshots as a tree or summary view. Comparing two snapshots reveals objects allocated between them.

**Beginner-Friendly Explanation:** A heap snapshot is a "photo" of all the objects in memory at a moment in time.

### How to Use

1. Open Memory panel.
2. Select "Heap snapshot".
3. Click "Take snapshot".
4. Interact with the app.
5. Take another snapshot.
6. Compare to find growing objects.

### Real-World Cases

- **Leak detection:** Compare snapshots.
- **Retainer analysis:** Find what's holding objects.
- **Object counts:** Identify unexpectedly large collections.

---

## Allocation Timelines

### Definitions

**Core Definition:** An allocation timeline records memory allocations over time, showing when and where memory is allocated.

**Technical Definition:** The Allocation Timeline (via CDP `HeapProfiler.startTrackingHeapObjects`) records every allocation during a session and displays it as a timeline of blue bars (new allocations) and grey bars (deallocations). This helps identify allocation hotspots.

**Beginner-Friendly Explanation:** The allocation timeline shows when memory is allocated, so you can see what's creating lots of objects.

### Real-World Cases

- **Hot allocation paths:** Find functions allocating frequently.
- **GC pressure:** Reduce allocation to reduce GC.
- **Event handler leaks:** Find listeners retaining memory.

---

## Memory Leaks

### Definitions

**Core Definition:** A memory leak is memory that is no longer needed but is not released because something still references it.

**Technical Definition:** In JavaScript, memory leaks occur when objects remain reachable from roots (global object, DOM, closures, timers) long after they are needed. Common causes: detached DOM nodes, uncleared event listeners, unintentional globals, closures capturing too much, unbounded caches.

**Beginner-Friendly Explanation:** A memory leak is when your app keeps holding onto things it doesn't need anymore, so memory grows over time.

### Common Causes

- **Detached DOM nodes:** Removed from DOM but still referenced.
- **Uncleared listeners:** Listener holds target.
- **Unintentional globals:** Missing `var`/`let`/`const`.
- **Unbounded caches:** Arrays/Maps that grow forever.
- **Timers:** `setInterval` never cleared.
- **Closures:** Capturing large objects.

### Detection

- **Memory panel:** Compare heap snapshots.
- **Performance panel:** Watch memory over time.
- **Chrome Task Manager:** Shift+Esc to see memory.

### Real-World Cases

- **SPAs:** Leaks from unmounted components.
- **Node.js servers:** Accumulating state.
- **Browser extensions:** Leaking across pages.

---

## Performance Best Practices

### Definitions

**Core Definition:** Performance best practices are the guidelines for writing fast, efficient JavaScript and web applications.

**Technical Definition:** Best practices span JavaScript execution (avoid long tasks, batch DOM reads/writes, use `requestAnimationFrame`), rendering (avoid layout thrashing, minimise reflows, use CSS transforms/opacity), memory (avoid leaks, reuse objects), network (cache, compress, prefetch), and measurement (profile, benchmark, monitor).

**Beginner-Friendly Explanation:** These are the rules of thumb for making your app fast.

### Best Practices

#### JavaScript Execution
1. **Avoid long tasks (>50ms).** Break work into chunks.
2. **Batch DOM reads and writes.** Avoid layout thrashing.
3. **Use `requestAnimationFrame` for animations.**
4. **Debounce/throttle expensive event handlers.**
5. **Avoid blocking the main thread.** Use Web Workers.
6. **Prefer `for` loops in hot paths.**
7. **Cache `length` and DOM lookups.**

#### Rendering
8. **Use CSS transforms and opacity for animations.**
9. **Avoid forced synchronous layout.**
10. **Minimise reflows and repaints.**
11. **Use `will-change` sparingly.**
12. **Lazy-load images and content.**

#### Memory
13. **Avoid leaks (remove listeners, clear timers).**
14. **Reuse objects and arrays.**
15. **Use `WeakMap`/`WeakSet` for object metadata.**
16. **Null out references when done.**

#### Network
17. **Cache aggressively.**
18. **Compress assets (gzip, brotli).**
19. **Use HTTP/2 or HTTP/3.**
20. **Prefetch and preload critical resources.**
21. **Code-split and lazy-load.**

#### Measurement
22. **Profile before optimising.**
23. **Measure with realistic conditions (throttling).**
24. **Monitor Core Web Vitals (LCP, INP, CLS).**
25. **Set performance budgets.**
26. **Re-measure after fixes.**

### Annotated Code Examples

#### Example 1: Avoiding Layout Thrashing

```js
// BAD: Read-write-read-write causes layout thrashing.
for (let i = 0; i < elements.length; i++) {
  elements[i].style.width = elements[i].offsetWidth + 10 + "px";
}

// GOOD: Batch reads, then batch writes.
const widths = elements.map((el) => el.offsetWidth);
elements.forEach((el, i) => {
  el.style.width = widths[i] + 10 + "px";
});
```

#### Example 2: Debouncing Expensive Handlers

```js
function debounce(fn, delay) {
  let timer;
  return (...args) => {
    clearTimeout(timer);
    timer = setTimeout(() => fn(...args), delay);
  };
}

window.addEventListener("resize", debounce(() => {
  recalculateLayout();
}, 200));
```

#### Example 3: Using `requestAnimationFrame`

```js
function animate() {
  element.style.transform = `translateX(${x}px)`;
  x += 1;
  if (x < 100) requestAnimationFrame(animate);
}
requestAnimationFrame(animate);
```

#### Example 4: Web Worker for Heavy Computation

```js
// main.js
const worker = new Worker("worker.js");
worker.postMessage({ data: largeArray });
worker.onmessage = (e) => console.log("Result:", e.data);

// worker.js
self.onmessage = (e) => {
  const result = e.data.data.map(expensiveTransform);
  self.postMessage(result);
};
```

### Real-World Cases

- **E-commerce:** Fast product pages.
- **Social media:** Smooth infinite scroll.
- **Dashboards:** Responsive charts.
- **Games:** 60 FPS rendering.

---

## References

- W3C – Performance Timeline — https://www.w3.org/TR/performance-timeline/
- W3C – User Timing Level 3 — https://www.w3.org/TR/user-timing/
- W3C – Navigation Timing Level 2 — https://www.w3.org/TR/navigation-timing-2/
- W3C – Resource Timing Level 2 — https://www.w3.org/TR/resource-timing-2/
- W3C – Long Tasks API — https://www.w3.org/TR/longtasks/
- WHATWG – Console Standard — https://console.spec.whatwg.org/
- Chrome DevTools – Performance panel overview — https://developer.chrome.com/docs/devtools/performance/
- Chrome DevTools – Record performance — https://developer.chrome.com/docs/devtools/performance/reference/
- Chrome DevTools – Analyze runtime performance — https://developer.chrome.com/docs/devtools/performance/reference/
- Chrome DevTools – Memory panel overview — https://developer.chrome.com/docs/devtools/memory/
- Chrome DevTools – Fix memory problems — https://developer.chrome.com/docs/devtools/memory-problems/
- Chrome DevTools – Heap snapshots — https://developer.chrome.com/docs/devtools/memory-problems/heap-snapshots/
- Chrome DevTools – Allocation timelines — https://developer.chrome.com/docs/devtools/memory-problems/allocation-profiler/
- Chrome DevTools Protocol – Profiler Domain — https://chromedevtools.github.io/devtools-protocol/tot/Profiler/
- Chrome DevTools Protocol – HeapProfiler Domain — https://chromedevtools.github.io/devtools-protocol/tot/HeapProfiler/
- Chrome DevTools Protocol – Tracing Domain — https://chromedevtools.github.io/devtools-protocol/tot/Tracing/
- MDN Web Docs – Performance API — https://developer.mozilla.org/en-US/docs/Web/API/Performance_API
- MDN Web Docs – performance.now() — https://developer.mozilla.org/en-US/docs/Web/API/Performance/now
- web.dev – Core Web Vitals — https://web.dev/vitals/
- web.dev – Optimize long tasks — https://web.dev/articles/optimize-long-tasks
- web.dev – Avoid large, complex layouts and layout thrashing — https://web.dev/articles/avoid-large-complex-layouts-and-layout-thrashing
- web.dev – Rendering performance — https://web.dev/articles/rendering-performance
- V8 Blog – Elements Kinds in V8 — https://v8.dev/blog/elements-kinds
- V8 Blog – High-performance garbage collection for V8 — https://v8.dev/blog/high-performance-gc
- V8 Blog – JavaScript performance — https://v8.dev/blog
- Firefox Source Docs – Performance — https://firefox-source-docs.mozilla.org/devtools-user/performance/
- Safari Web Inspector – Timelines — https://developer.apple.com/safari/tools/
- Google Web Fundamentals – Performance — https://web.dev/learn/performance
- Steve Souders – High Performance Web Sites — https://stevesouders.com/hpws/