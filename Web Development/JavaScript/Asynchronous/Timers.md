# Timers — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Timers are browser and Node.js APIs that allow JavaScript code to schedule functions to run at a future point in time, either once (`setTimeout`) or repeatedly (`setInterval`).

**Technical Definition:** Per the HTML Living Standard, "The `setTimeout()` and `setInterval()` methods allow authors to schedule timer-based callbacks". These methods are part of the `WindowOrWorkerGlobalScope` mixin, available in both browser and worker contexts. In Node.js, timer functions are globals provided by the `timers` module. All timer functions return an opaque identifier that can be used to cancel the scheduled callback. Timers are asynchronous and are integrated into the event loop: callbacks are queued as macrotasks (tasks) and executed when the call stack is empty.

**Beginner-Friendly Explanation:** Timers let you say "run this function after X milliseconds" (`setTimeout`) or "run this function every X milliseconds" (`setInterval`). They don't block your code — they schedule work to happen later, in the background, while your program continues running.

### Key Characteristics

- **Asynchronous scheduling:** Timer callbacks are queued as macrotasks and executed by the event loop.
- **Minimum delay:** Modern browsers enforce a minimum delay of 4ms for nested timers (after 5 levels of nesting).
- **Not precise:** Timer delays are minimums, not guarantees; actual execution time depends on CPU load and other tasks.
- **Throttling:** Browsers throttle timers in background tabs to save battery and reduce resource consumption.
- **Cancellable:** Timers return IDs that can be passed to `clearTimeout()` or `clearInterval()` to cancel execution.
- **String evaluation risk:** Passing a string as the first argument to `setTimeout`/`setInterval` evaluates it as code, posing an XSS risk.

### Prerequisites

- Understanding of JavaScript functions, callbacks, and closures.
- Familiarity with the event loop, task queue, and microtask queue.
- Basic knowledge of asynchronous programming.

### Related Programming Areas

- **Event loop:** Timers are integrated into the event loop as macrotasks.
- **Animation:** `requestAnimationFrame` is preferred over timers for visual updates.
- **Debouncing and throttling:** Timers are the foundation of these techniques.
- **Node.js:** Timers module provides `setImmediate` and `process.nextTick` in addition to `setTimeout`/`setInterval`.
- **Performance:** Timer throttling and drift affect application responsiveness.

### Core Concepts / Features

---

## `setTimeout()`

### Definitions

**Core Definition:** `setTimeout()` is a method that sets a timer which executes a function or specified piece of code once the timer expires.

**Technical Definition:** Per MDN, "The `setTimeout()` method of the `Window` interface sets a timer which executes a function or specified piece of code once the timer expires". It returns a positive integer (typically within the range of 1 to 2,147,483,647) that uniquely identifies the timeout. The delay parameter is a non-negative integer indicating how long the timer should wait before executing the callback, in milliseconds; it defaults to 0 if not specified. The delay has a maximum value of 2147483647 ms (approximately 24.8 days); specifying larger values may result in overflow or a 0 value being used. The actual delay may be longer than set — setting `delay` to 0 will execute in the next event cycle rather than "immediately".

**Beginner-Friendly Explanation:** `setTimeout()` is a one-shot timer. You tell it "run this function after X milliseconds," and it does that once. It doesn't block your code — it schedules the function to run later.

### Purposes

- To delay the execution of a function by a specified amount of time.
- To schedule one-time asynchronous work.
- To break up long-running tasks into smaller chunks that yield to the event loop.
- To implement debouncing and other delay-based patterns.

### Syntax Rules and Structure

```js
setTimeout(func, delay, param1, param2, ...);
setTimeout(code, delay);
```

**Component Breakdown:**
- `func`: A function to be executed after the timer expires.
- `code`: A `TrustedScript` or string of arbitrary code (strongly discouraged).
- `delay` (optional): A non-negative integer in milliseconds. Defaults to 0.
- `param1, param2, ...` (optional): Additional arguments passed to `func`.

**Syntax Rules:**
- Returns a positive integer `timeoutID` for use with `clearTimeout()`.
- The delay is a minimum, not a guarantee.
- `setTimeout()` is asynchronous; it does not pause execution.

**Constraints and Limitations:**
- Nested timers (5+ levels deep) are forced to a minimum delay of 4ms per the HTML spec.
- Maximum delay is 2147483647 ms; larger values may overflow.
- Passing a string as `code` is a security risk (XSS vector) and is strongly discouraged.

### Annotated Code Examples

#### Example 1: Basic `setTimeout`

```js
// Step 1: Schedule a callback after 1000ms.
const timeoutId = setTimeout(() => {
  console.log("Executed after 1 second");
}, 1000);

// Step 2: This runs immediately.
console.log("Scheduled");

// Step 3: Cancel the timeout (optional).
// clearTimeout(timeoutId);
```

**Expected Output:**
```
Scheduled
Executed after 1 second
```

**Why This Output:** The synchronous `console.log("Scheduled")` runs first. After 1000ms, the event loop executes the timer callback.

#### Example 2: Passing Arguments to `setTimeout`

```js
// Step 1: Define a function that takes arguments.
function greet(name, greeting) {
  console.log(`${greeting}, ${name}!`);
}

// Step 2: Pass arguments after the delay.
setTimeout(greet, 500, "Alice", "Hello");
```

**Expected Output (after 500ms):**
```
Hello, Alice!
```

**Why This Output:** Additional arguments after `delay` are passed directly to the callback function.

### Real-World Cases

- **Debouncing search input:** Delaying an API call until the user stops typing.
- **UI animations:** Sequencing transitions after a delay.
- **Deferred initialisation:** Scheduling non-critical setup work after page load.

---

## `setInterval()`

### Definitions

**Core Definition:** `setInterval()` is a method that repeatedly calls a function or executes a code snippet, with a fixed time delay between each call.

**Technical Definition:** Per MDN, "The `setInterval()` method of the `Window` interface repeatedly calls a function or executes a code snippet, with a fixed time delay between each call". It returns a positive integer (typically within the range of 1 to 2,147,483,647) that uniquely identifies the interval timer created by the call. The first execution happens after `delay` milliseconds. The delay defaults to 0 if not specified.

**Beginner-Friendly Explanation:** `setInterval()` is like `setTimeout()`, but it repeats. It keeps calling your function every X milliseconds until you stop it with `clearInterval()`.

### Purposes

- To execute a function repeatedly at fixed intervals.
- To implement polling and periodic updates.
- To create repeating animations or timers.

### Syntax Rules and Structure

```js
setInterval(func, delay, param1, param2, ...);
setInterval(code, delay);
```

**Component Breakdown:**
- `func`: A function to be executed every `delay` milliseconds.
- `delay` (optional): The delay time between executions, in milliseconds. Defaults to 0.
- `param1, param2, ...` (optional): Additional arguments passed to `func`.

**Syntax Rules:**
- Returns a positive integer `intervalID` for use with `clearInterval()`.
- The first execution happens after `delay` milliseconds.
- `setInterval()` and `setTimeout()` share the same ID pool.

**Constraints and Limitations:**
- If the callback takes longer than the delay, calls may overlap or be skipped.
- The delay is a minimum, not a guarantee.
- Throttling applies in background tabs (see Timer Throttling).

### Annotated Code Examples

#### Example 1: Basic `setInterval`

```js
// Step 1: Start a repeating timer.
let count = 0;
const intervalId = setInterval(() => {
  count++;
  console.log(`Interval fired ${count} time(s)`);

  // Step 2: Stop after 3 executions.
  if (count >= 3) {
    clearInterval(intervalId);
  }
}, 1000);
```

**Expected Output:**
```
Interval fired 1 time(s)
Interval fired 2 time(s)
Interval fired 3 time(s)
```

**Why This Output:** The interval fires every 1000ms. After 3 fires, `clearInterval` stops it.

#### Example 2: Polling with `setInterval`

```js
// Step 1: Poll a server every 5 seconds.
const pollId = setInterval(async () => {
  try {
    const response = await fetch("/api/status");
    const data = await response.json();
    console.log("Status:", data.status);

    // Step 2: Stop polling when complete.
    if (data.status === "complete") {
      clearInterval(pollId);
    }
  } catch (error) {
    console.error("Poll failed:", error.message);
  }
}, 5000);
```

**Expected Output:** The status is logged every 5 seconds until it becomes `"complete"`.

**Why This Output:** The interval fires every 5 seconds, fetching the status. When the status is `"complete"`, the interval is cleared.

### Real-World Cases

- **Clocks:** Updating a digital clock every second.
- **Polling:** Checking a server for updates every N seconds.
- **Game loops:** Updating game state at fixed intervals.
- **Auto-save:** Periodically saving form data.

---

## Timer Precision

### Definitions

**Core Definition:** Timer precision refers to the accuracy with which timer callbacks execute relative to their specified delay. JavaScript timers are not precise; the actual delay may be longer than requested.

**Technical Definition:** Per the HTML spec, "This API does not guarantee that timers will run exactly on schedule. Delays due to CPU load, other tasks, etc, are to be expected". Timer precision is affected by: (1) the minimum delay (4ms for nested timers), (2) the event loop's current workload, (3) browser throttling, and (4) the resolution of the system clock.

**Beginner-Friendly Explanation:** `setTimeout(fn, 100)` does not mean "run `fn` exactly 100ms from now." It means "run `fn` no sooner than 100ms from now." If the browser is busy with other tasks, it might run at 105ms or 150ms.

### Purposes

- To understand the limitations of timer-based scheduling.
- To design applications that tolerate timing variance.
- To choose the right timing mechanism for the task.

### Syntax Rules and Structure

Timer precision is not a syntax construct; it is a runtime characteristic. The minimum delay is enforced as:

```js
// After 5 levels of nesting, delay is forced to at least 4ms:
setTimeout(() => {
  setTimeout(() => {
    setTimeout(() => {
      setTimeout(() => {
        setTimeout(() => {
          // Nested 5 levels — minimum delay is now 4ms
        }, 0);
      }, 0);
    }, 0);
  }, 0);
}, 0);
```

**Component Breakdown:**
- **Minimum delay:** 4ms for nested timers (5+ levels).
- **Actual delay:** Depends on event loop workload.

**Syntax Rules:**
- The delay parameter is a minimum, not a guarantee.
- Nested timers are clamped to a 4ms minimum.

**Constraints and Limitations:**
- Timer precision is affected by CPU load, rendering, and other tasks.
- High-precision timing requires `performance.now()` or `requestAnimationFrame`.

### Annotated Code Examples

#### Example 1: Measuring Timer Precision

```js
const start = performance.now();

setTimeout(() => {
  const elapsed = performance.now() - start;
  console.log(`Requested: 100ms, Actual: ${elapsed.toFixed(2)}ms`);
}, 100);
```

**Expected Output:**
```
Requested: 100ms, Actual: 102.34ms (approximate)
```

**Why This Output:** The actual delay is slightly longer than 100ms due to event loop overhead and system scheduling.

### Real-World Cases

- **Animations:** Using `requestAnimationFrame` instead of timers for smooth visuals.
- **Audio:** Web Audio API provides sample-accurate timing.
- **Games:** Using `performance.now()` for game loops.

---

## Timer Throttling

### Definitions

**Core Definition:** Timer throttling is the browser's deliberate reduction of timer execution frequency in background tabs to save battery and reduce resource consumption.

**Technical Definition:** Per the reference, "Modern browsers throttle JavaScript timers (`setTimeout`, `setInterval`) when tabs are in the background to save battery and improve performance". Chrome's "Intensive Wake Up Throttling" feature throttles JavaScript timers in background tabs (hidden for >5 minutes) to run only once per minute (60 seconds). Safari and other browsers have similar throttling mechanisms.

**Beginner-Friendly Explanation:** When you switch to another tab, browsers slow down timers in the background tab to save power. A timer that fires every second in the foreground might only fire once per minute in the background.

### Purposes

- To conserve battery on mobile devices.
- To reduce CPU usage from background tabs.
- To improve performance of the foreground tab.

### Syntax Rules and Structure

Throttling is not controlled by JavaScript; it is a browser policy. Workarounds include:

```js
// Web Worker keeps timing accurate in background tabs.
const worker = new Worker("timer-worker.js");
worker.postMessage({ interval: 1000 });
worker.onmessage = (event) => {
  console.log("Timer tick from worker");
};
```

**Component Breakdown:**
- **Web Worker:** Runs in a separate thread, unaffected by background tab throttling.
- **Visibility API:** `document.hidden` can detect when the tab is hidden.

**Syntax Rules:**
- Throttling applies to `setTimeout` and `setInterval` in background tabs.
- Web Workers are not throttled in the same way.
- `requestAnimationFrame` is paused entirely in background tabs.

**Constraints and Limitations:**
- Throttling policies vary by browser and device.
- Web Workers have communication overhead.

### Annotated Code Examples

#### Example 1: Detecting Tab Visibility

```js
document.addEventListener("visibilitychange", () => {
  if (document.hidden) {
    console.log("Tab is hidden — timers will be throttled");
  } else {
    console.log("Tab is visible — timers run normally");
  }
});
```

**Expected Output:**
```
Tab is hidden — timers will be throttled (when switching away)
Tab is visible — timers run normally (when returning)
```

**Why This Output:** The `visibilitychange` event fires when the tab's visibility changes, allowing the application to adjust behaviour.

### Real-World Cases

- **Real-time applications:** Using Web Workers to maintain connections in background tabs.
- **Games:** Pausing game logic when the tab is hidden.
- **Analytics:** Batching analytics events when the tab is hidden.

---

## Timer Drift

### Definitions

**Core Definition:** Timer drift is the gradual accumulation of timing errors in `setInterval` due to the difference between the scheduled time and the actual execution time.

**Technical Definition:** Per the reference, "`setInterval` — Still drifts. The WHATWG issue #3151 asking whether `setInterval` should account for execution delays to prevent drift is still open and unresolved". Chrome self-corrects drift in `setInterval`, but Node.js and other browsers do not. The drift occurs because `setInterval` schedules the next execution based on the **scheduled** time, not the **actual** execution time. If the callback takes longer than the delay, executions pile up.

**Beginner-Friendly Explanation:** If you set an interval of 1 second, but each callback takes 1.1 seconds to run, the interval will gradually drift behind. After 10 executions, it will be 1 second behind.

### Purposes

- To understand why `setInterval` is inaccurate over long periods.
- To implement drift-corrected timing using `setTimeout`.
- To choose the right timer for time-sensitive applications.

### Syntax Rules and Structure

Drift-corrected timing uses `setTimeout` and tracks the actual elapsed time:

```js
function driftCorrectedInterval(callback, delay) {
  let expected = Date.now() + delay;

  function step() {
    const drift = Date.now() - expected;
    callback();
    expected += delay;
    setTimeout(step, Math.max(0, delay - drift));
  }

  setTimeout(step, delay);
}
```

**Component Breakdown:**
- `expected`: The expected time of the next execution.
- `drift`: The difference between actual and expected time.
- `setTimeout(step, delay - drift)`: Adjusts the next delay to compensate for drift.

**Syntax Rules:**
- `setInterval` does not correct for drift.
- Use `setTimeout` recursively with drift correction for accurate timing.

**Constraints and Limitations:**
- Drift correction adds complexity.
- Even with correction, timers are not perfectly precise.

### Annotated Code Examples

#### Example 1: Demonstrating Drift

```js
let count = 0;
const start = Date.now();

const intervalId = setInterval(() => {
  count++;
  const elapsed = Date.now() - start;
  const expected = count * 1000;
  const drift = elapsed - expected;
  console.log(`Tick ${count}: expected ${expected}ms, actual ${elapsed}ms, drift ${drift}ms`);

  if (count >= 5) clearInterval(intervalId);
}, 1000);
```

**Expected Output (approximate):**
```
Tick 1: expected 1000ms, actual 1002ms, drift 2ms
Tick 2: expected 2000ms, actual 2005ms, drift 5ms
Tick 3: expected 3000ms, actual 3010ms, drift 10ms
...
```

**Why This Output:** The drift accumulates because `setInterval` does not compensate for execution delays.

### Real-World Cases

- **Clocks:** Drift causes clocks to fall behind.
- **Games:** Drift causes game logic to desynchronise.
- **Audio sequencing:** Drift causes audio to go out of sync.

---

## Clearing Timers

### Definitions

**Core Definition:** Clearing timers is the process of cancelling a scheduled `setTimeout` or `setInterval` using `clearTimeout()` or `clearInterval()`.

**Technical Definition:** Per the HTML spec, `clearTimeout(id)` "Cancels the timeout set with `setTimeout()` or `setInterval()` identified by `id`", and `clearInterval(id)` "Cancels the timeout set with `setInterval()` or `setTimeout()` identified by `id`". Both methods use the same list of entries, so `clearTimeout` and `clearInterval` can technically clear each other's timers. For consistency, MDN recommends using `clearTimeout()` for `setTimeout()` entries and `clearInterval()` for `setInterval()` entries.

**Beginner-Friendly Explanation:** Clearing a timer means telling JavaScript "cancel that timer I set earlier." You pass the ID that `setTimeout` or `setInterval` returned.

### Purposes

- To cancel a scheduled timeout before it fires.
- To stop a repeating interval.
- To prevent memory leaks from forgotten timers.
- To clean up timers when components unmount or pages unload.

### Syntax Rules and Structure

```js
const timeoutId = setTimeout(callback, delay);
clearTimeout(timeoutId);

const intervalId = setInterval(callback, delay);
clearInterval(intervalId);
```

**Component Breakdown:**
- `timeoutId` / `intervalId`: The ID returned by `setTimeout`/`setInterval`.
- `clearTimeout` / `clearInterval`: The cancellation methods.

**Syntax Rules:**
- The ID must be the value returned by the timer function.
- If the timer has already fired, clearing has no effect.
- Both methods share the same ID pool.

**Constraints and Limitations:**
- Forgetting to clear timers causes memory leaks.
- Timers should be cleared when components unmount or pages unload.

### Annotated Code Examples

#### Example 1: Clearing a Timeout

```js
// Step 1: Schedule a timeout.
const timeoutId = setTimeout(() => {
  console.log("This will not be printed");
}, 1000);

// Step 2: Cancel it before it fires.
clearTimeout(timeoutId);
console.log("Timeout cancelled");
```

**Expected Output:**
```
Timeout cancelled
```

**Why This Output:** `clearTimeout` cancels the scheduled callback before it executes.

#### Example 2: Clearing an Interval in a Component Lifecycle

```js
// Step 1: Start an interval.
let intervalId = setInterval(() => {
  console.log("Polling...");
}, 1000);

// Step 2: Clear it when no longer needed.
function cleanup() {
  clearInterval(intervalId);
  console.log("Interval cleared");
}

// Call cleanup when component unmounts or page unloads.
// cleanup();
```

**Expected Output:**
```
Polling...
Polling...
Interval cleared
```

**Why This Output:** The interval is cleared, stopping further polling.

### Real-World Cases

- **React:** Clearing intervals in `useEffect` cleanup functions.
- **SPAs:** Clearing timers on route changes to prevent memory leaks.
- **Node.js:** Clearing intervals when a server shuts down.

---

## Nested Timers

### Definitions

**Core Definition:** Nested timers are timers that schedule other timers from within their callbacks, creating a chain or recursion of timer-based execution.

**Technical Definition:** Per the HTML spec, "Timers can be nested; after five such nested timers, however, the interval is forced to be at least four milliseconds". This clamping was introduced to prevent CPU hogging from `setTimeout(fn, 0)` recursion. Prior to HTML5, the minimum was 10ms in some browsers.

**Beginner-Friendly Explanation:** Nested timers are when a timer's callback starts another timer, which starts another timer, and so on. After 5 levels of nesting, the minimum delay becomes 4ms.

### Purposes

- To create recursive scheduling without using `setInterval`.
- To avoid overlapping executions (unlike `setInterval`).
- To implement drift-corrected intervals.

### Syntax Rules and Structure

```js
function recursiveTimer() {
  // Do work
  setTimeout(recursiveTimer, 0);
}
setTimeout(recursiveTimer, 0);
```

**Component Breakdown:**
- The callback schedules the next iteration.
- After 5 levels, the delay is forced to at least 4ms.

**Syntax Rules:**
- The nesting level is tracked by the browser.
- The 4ms minimum applies after 5 nested levels.

**Constraints and Limitations:**
- The 4ms minimum can slow down rapid recursion.
- Use `setImmediate` in Node.js for true zero-delay recursion.

### Annotated Code Examples

#### Example 1: Recursive `setTimeout`

```js
let count = 0;

function recursive() {
  count++;
  console.log(`Iteration ${count}`);

  if (count < 5) {
    setTimeout(recursive, 0);
  }
}

setTimeout(recursive, 0);
```

**Expected Output:**
```
Iteration 1
Iteration 2
Iteration 3
Iteration 4
Iteration 5
```

**Why This Output:** Each iteration schedules the next one. After 5 iterations, the recursion stops.

#### Example 2: Avoiding Overlap with `setInterval`

```js
// BAD: setInterval can overlap if callback is slow.
setInterval(async () => {
  await slowOperation(); // If this takes longer than the interval, calls overlap
}, 1000);

// GOOD: Recursive setTimeout avoids overlap.
async function poll() {
  await slowOperation();
  setTimeout(poll, 1000);
}
setTimeout(poll, 1000);
```

**Expected Output:** The recursive version never overlaps; the interval version may.

**Why This Output:** The recursive version waits for the operation to complete before scheduling the next one.

### Real-World Cases

- **Polling:** Avoiding overlapping requests.
- **Game loops:** Using recursive `setTimeout` for frame-based logic.
- **Data processing:** Processing items one at a time without overlap.

---

## Timer Best Practices

### Definitions

**Core Definition:** Timer best practices are guidelines for using timers effectively, including always clearing timers, preferring `requestAnimationFrame` for animations, and using recursive `setTimeout` over `setInterval` for critical timing.

**Technical Definition:** Per the best practices reference, "It's always recommended to track your timeouts and intervals and clean them up properly when no longer needed". Failing to clear timers causes callbacks to execute indefinitely and prevents garbage collection of referenced objects. For animations, `requestAnimationFrame` should be used instead of `setTimeout` or `setInterval`.

**Beginner-Friendly Explanation:** Always clean up your timers. Don't use `setInterval` for animations — use `requestAnimationFrame`. Use recursive `setTimeout` for polling to avoid overlap. And be aware that timers are not precise.

### Purposes

- To prevent memory leaks from forgotten timers.
- To improve animation performance.
- To avoid overlapping executions.
- To write maintainable, predictable timer-based code.

### Syntax Rules and Structure

Best practices in practice:

```js
// GOOD: Clear timers on cleanup.
function setup() {
  const id = setInterval(doWork, 1000);
  return () => clearInterval(id);
}

// GOOD: requestAnimationFrame for animations.
function animate() {
  // Update animation
  requestAnimationFrame(animate);
}
requestAnimationFrame(animate);

// GOOD: Recursive setTimeout for polling.
function poll() {
  fetchData().then(() => setTimeout(poll, 1000));
}
setTimeout(poll, 1000);
```

**Component Breakdown:**
- **Cleanup:** Always clear timers when they are no longer needed.
- **`requestAnimationFrame`:** Use for visual updates instead of `setTimeout`/`setInterval`.
- **Recursive `setTimeout`:** Use for polling to avoid overlap.

**Syntax Rules:**
- Track timer IDs and clear them in cleanup functions.
- Use `requestAnimationFrame` for animations.
- Use recursive `setTimeout` instead of `setInterval` for sequential async operations.

**Constraints and Limitations:**
- `requestAnimationFrame` is paused in background tabs.
- Recursive `setTimeout` has a 4ms minimum after 5 levels.

### Annotated Code Examples

#### Example 1: Cleanup in React

```js
import { useEffect, useState } from "react";

function TimerComponent() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const id = setInterval(() => {
      setCount((c) => c + 1);
    }, 1000);

    // Cleanup: clear interval on unmount.
    return () => clearInterval(id);
  }, []);

  return <div>Count: {count}</div>;
}
```

**Expected Output:** The count increments every second and stops when the component unmounts.

**Why This Output:** The `useEffect` cleanup function clears the interval, preventing memory leaks.

#### Example 2: `requestAnimationFrame` vs `setTimeout`

```js
// BAD: Using setTimeout for animation.
setTimeout(() => {
  element.style.transform = "translateX(100px)";
}, 16); // ~60fps, but may miss frames

// GOOD: Using requestAnimationFrame.
function animate() {
  element.style.transform = "translateX(100px)";
  // Next frame will be scheduled only if needed.
}
requestAnimationFrame(animate);
```

**Expected Output:** The `requestAnimationFrame` version produces smoother animations.

**Why This Output:** `requestAnimationFrame` aligns with the browser's rendering cycle, reducing jank.

### Real-World Cases

- **React/Vue/Angular:** Clearing timers in component lifecycle hooks.
- **Games:** Using `requestAnimationFrame` for game loops.
- **Node.js:** Using `setImmediate` for recursive operations instead of `setTimeout`.

---

## References

- MDN Web Docs – Window: setTimeout() method — https://developer.mozilla.org/en-US/docs/Web/API/Window/setTimeout
- MDN Web Docs – Window: setInterval() method — https://developer.mozilla.org/en-US/docs/Web/API/Window/setInterval
- HTML: The Living Standard – Timers — https://html.spec.whatwg.org/dev/timers-and-user-prompts.html
- MDN Web Docs – Window: clearTimeout() — https://developer.mozilla.org/en-US/docs/Web/API/Window/clearTimeout
- MDN Web Docs – Window: clearInterval() — https://developer.mozilla.org/en-US/docs/Web/API/Window/clearInterval
- NativeScript Docs – Best Practices with Timers and Intervals — https://beta.docs.nativescript.org
- JavaScript.info – Scheduling: setTimeout and setInterval — https://javascript.info/settimeout-setinterval
- web.dev – Optimize JavaScript Execution (requestAnimationFrame) — https://web.developers.google.cn
- Chrome for Developers – Intensive Wake Up Throttling — https://developer.chrome.com
- WHATWG Issue #3151 – setInterval drift — https://github.com/whatwg/html/issues/3151
- Node.js Documentation – Timers — https://nodejs.org/api/timers.html
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) — https://ecma-international.org/wp-content/uploads/ECMA-262_16th_edition_june_2025.pdf