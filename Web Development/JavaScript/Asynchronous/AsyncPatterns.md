# Asynchronous Patterns — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Asynchronous patterns are reusable architectural and algorithmic solutions for coordinating, controlling, and managing asynchronous operations in JavaScript. They address common challenges such as execution ordering, concurrency control, error recovery, and resource management.

**Technical Definition:** Asynchronous patterns in ECMAScript leverage Promises, `async/await`, `AbortController`, and the event loop to orchestrate non-blocking operations. They are built on the language's concurrency model: a single-threaded event loop with a call stack, task queue, and microtask queue. These patterns provide structured approaches to sequential and parallel execution, concurrency limiting, retry logic, timeout handling, cancellation, rate limiting, and backoff strategies. The ECMAScript specification defines the underlying primitives (Promise objects, async functions, `AbortSignal`), while the patterns themselves are architectural conventions built on these primitives.

**Beginner-Friendly Explanation:** When you write asynchronous code, you often face the same problems: "How do I run tasks one after another?", "How do I run many tasks at once without overwhelming the system?", "What if a task fails — should I retry?", "How do I cancel a task that's taking too long?" Asynchronous patterns are the tried-and-tested answers to these questions. They're the "recipes" experienced developers use to handle real-world asynchronous challenges.

### Key Characteristics

- **Built on Promises:** Most modern async patterns use Promises and `async/await` as their foundation.
- **Concurrency control:** Patterns like pools and semaphores limit how many operations run simultaneously.
- **Error resilience:** Retry and backoff patterns handle transient failures gracefully.
- **Cancellation support:** `AbortController` and `AbortSignal` provide a standard way to cancel in-flight operations.
- **Resource management:** Timeout patterns prevent operations from hanging indefinitely.
- **Rate awareness:** Debouncing, throttling, and rate limiting protect both the client and the server from excessive load.

### Prerequisites

- Understanding of Promises (states, `.then()`, `.catch()`, `async/await`).
- Familiarity with the event loop, task queue, and microtask queue.
- Basic knowledge of callbacks and callback hell.
- Awareness of `try...catch` error handling.

### Related Programming Areas

- **Distributed systems:** Retry, backoff, and circuit breaker patterns are essential in microservices.
- **API clients:** Rate limiting, throttling, and timeout patterns protect against overloading servers.
- **UI development:** Debouncing and throttling optimise event handling.
- **Data processing:** Sequential, parallel, and limited concurrency patterns handle large datasets.
- **Real-time applications:** Cancellation and timeout patterns manage long-lived connections.

### Core Concepts / Features

---

## Sequential Execution

### Definitions

**Core Definition:** Sequential execution is the pattern of running asynchronous operations one after another, where each operation waits for the previous one to complete before starting.

**Technical Definition:** Per the concurrency patterns reference, "One at a time | Sequential | `for...of` + `await` / `Array.fromAsync`". Each `await` expression pauses the async function until the awaited Promise settles, ensuring that operations execute in a deterministic order. This is the default behaviour of `await` when used in a loop or sequence.

**Beginner-Friendly Explanation:** Sequential execution is like standing in line at a bank: you wait for the person in front of you to finish before you step up. Each operation waits for the previous one to complete.

### Purposes

- To execute operations that depend on the result of previous operations.
- To control the order of asynchronous operations.
- To avoid overwhelming a resource with concurrent requests.
- To process items one at a time in a deterministic order.

### Syntax Rules and Structure

```js
async function sequential(items) {
  const results = [];
  for (const item of items) {
    results.push(await processItem(item));
  }
  return results;
}
```

**Component Breakdown:**
- `for...of`: Iterates over the items sequentially.
- `await processItem(item)`: Pauses until the current item is processed.
- `results.push(...)`: Collects the results in order.

**Syntax Rules:**
- `await` in a `for...of` loop causes sequential execution.
- `forEach` with `async/await` does **not** work as expected (callbacks are not awaited).
- `Array.fromAsync` (ES2025) provides a declarative alternative.

**Constraints and Limitations:**
- Sequential execution is slower than parallel execution for independent operations.
- Using `await` in a loop for independent operations is an anti-pattern.

### Annotated Code Examples

#### Example 1: Sequential Processing with `for...of`

```js
async function processSequentially(items) {
  const results = [];
  for (const item of items) {
    console.log(`Processing: ${item}`);
    const result = await processItem(item);
    results.push(result);
  }
  return results;
}

async function processItem(item) {
  return new Promise((resolve) =>
    setTimeout(() => resolve(`${item} processed`), 100)
  );
}

processSequentially(["A", "B", "C"]).then(console.log);
```

**Expected Output:**
```
Processing: A
Processing: B
Processing: C
[ 'A processed', 'B processed', 'C processed' ]
```

**Why This Output:** Each item is processed only after the previous one completes. The logs appear in order.

#### Example 2: `Array.fromAsync` with an Async Generator (ES2025)

```js
async function* processSequentially(items) {
  for (const item of items) {
    yield await processItem(item);
  }
}

const results = await Array.fromAsync(processSequentially(["A", "B", "C"]));
console.log(results); // ['A processed', 'B processed', 'C processed']
```

**Expected Output:**
```
[ 'A processed', 'B processed', 'C processed' ]
```

**Why This Output:** `Array.fromAsync` consumes the async generator sequentially, collecting the results into an array.

### Real-World Cases

- **User onboarding:** Fetch user → create profile → send welcome email.
- **E-commerce checkout:** Validate cart → process payment → confirm order.
- **Pagination:** Fetch page 1 → fetch page 2 → fetch page 3.

---

## Parallel Execution

### Definitions

**Core Definition:** Parallel execution is the pattern of starting multiple asynchronous operations simultaneously and waiting for all of them to complete, with the total time approximately equal to the longest operation.

**Technical Definition:** Per the concurrency patterns reference, "All at once | Parallel | `Promise.all`". `Promise.all()` takes an iterable of Promises and returns a single Promise that fulfills when all input Promises fulfill, with an array of their fulfillment values. It rejects immediately if any input rejects.

**Beginner-Friendly Explanation:** Parallel execution is like ordering several dishes at a restaurant: they all start cooking at the same time, and you get them when they're ready. You don't wait for one dish to finish before ordering the next.

### Purposes

- To reduce total execution time for independent operations.
- To efficiently use network bandwidth and server resources.
- To improve application performance.
- To collect results from multiple sources concurrently.

### Syntax Rules and Structure

```js
const results = await Promise.all(items.map(item => processItem(item)));
```

**Component Breakdown:**
- `items.map(item => processItem(item))`: Starts all operations simultaneously.
- `Promise.all([...])`: Waits for all to fulfill.
- `await`: Pauses until all are complete.

**Syntax Rules:**
- Start all Promises before awaiting them.
- The order of results matches the order of the input array.
- Rejects immediately if any input rejects.

**Constraints and Limitations:**
- Unbounded concurrency can exhaust resources.
- A single rejection causes the entire `Promise.all()` to reject.

### Annotated Code Examples

#### Example 1: Parallel Processing with `Promise.all()`

```js
async function processInParallel(items) {
  const results = await Promise.all(
    items.map((item) => processItem(item))
  );
  return results;
}

processInParallel(["A", "B", "C"]).then(console.log);
```

**Expected Output:**
```
[ 'A processed', 'B processed', 'C processed' ]
```

**Why This Output:** All three items are processed simultaneously. The total time is approximately the time of the slowest operation.

### Real-World Cases

- **Dashboard loading:** Fetching multiple widgets concurrently.
- **Batch API calls:** Sending multiple requests at once.
- **Data processing:** Transforming multiple items in parallel.

---

## Limited Concurrency

### Definitions

**Core Definition:** Limited concurrency is the pattern of running multiple asynchronous operations in parallel but limiting the number of operations that execute simultaneously.

**Technical Definition:** Per the concurrency patterns reference, "N simultaneous | Pool | `Promise.race` + `Set`" and "Limit concurrent access | Semaphore | Permit queue". A concurrency pool runs at most N operations at a time, starting a new one only when a running operation completes. A semaphore uses a permit-based approach to control access to a shared resource.

**Beginner-Friendly Explanation:** Limited concurrency is like having 5 checkout lanes at a supermarket. Even if 20 people are waiting, only 5 are being served at any moment. As each lane frees up, the next person is served.

### Purposes

- To prevent overwhelming a server or database with too many concurrent requests.
- To respect rate limits imposed by external APIs.
- To manage memory and CPU usage.
- To balance throughput and resource consumption.

### Syntax Rules and Structure

```js
async function pool(items, concurrency, fn) {
  const results = [];
  const executing = new Set();
  for (const item of items) {
    const p = fn(item).then(r => { executing.delete(p); return r; });
    results.push(p);
    executing.add(p);
    if (executing.size >= concurrency) {
      await Promise.race(executing);
    }
  }
  return Promise.all(results);
}
```

**Component Breakdown:**
- `executing`: A `Set` tracking currently running Promises.
- `Promise.race(executing)`: Waits for the fastest operation to complete, freeing up a slot.
- `concurrency`: The maximum number of simultaneous operations.

**Syntax Rules:**
- The pool starts new operations as slots become available.
- `p-limit` is a popular library implementing this pattern.
- A semaphore uses `acquire()` and `release()` methods.

**Constraints and Limitations:**
- More complex than simple `Promise.all()`.
- Requires careful handling of errors in individual operations.

### Annotated Code Examples

#### Example 1: Concurrency Pool

```js
async function pool(items, concurrency, fn) {
  const results = [];
  const executing = new Set();

  for (const item of items) {
    const p = fn(item).then((r) => {
      executing.delete(p);
      return r;
    });
    results.push(p);
    executing.add(p);

    if (executing.size >= concurrency) {
      await Promise.race(executing);
    }
  }

  return Promise.all(results);
}

// Process 10 items with a maximum of 3 concurrent operations.
const items = Array.from({ length: 10 }, (_, i) => i);
await pool(items, 3, processItem);
```

**Expected Output:** Items are processed with at most 3 concurrent operations.

**Why This Output:** The pool starts 3 operations. When one completes, the next one starts, maintaining the concurrency limit.

#### Example 2: Semaphore Pattern

```js
class Semaphore {
  #permits;
  #queue = [];

  constructor(permits) {
    this.#permits = permits;
  }

  async acquire() {
    if (this.#permits > 0) {
      this.#permits--;
      return;
    }
    const { promise, resolve } = Promise.withResolvers();
    this.#queue.push(resolve);
    return promise;
  }

  release() {
    if (this.#queue.length > 0) {
      this.#queue.shift()();
    } else {
      this.#permits++;
    }
  }

  async withPermit(fn) {
    await this.acquire();
    try {
      return await fn();
    } finally {
      this.release();
    }
  }
}

const semaphore = new Semaphore(3);
await Promise.all(
  items.map((item) => semaphore.withPermit(() => processItem(item)))
);
```

**Expected Output:** Items are processed with at most 3 concurrent operations.

**Why This Output:** The semaphore controls access, allowing only 3 operations to run simultaneously.

### Real-World Cases

- **API rate limiting:** Limiting concurrent requests to avoid 429 errors.
- **Database connection pools:** Reusing a fixed number of connections.
- **File uploads:** Limiting concurrent uploads to avoid bandwidth saturation.
- **Web scraping:** Limiting concurrent requests to avoid being blocked.

---

## Retry Patterns

### Definitions

**Core Definition:** Retry patterns automatically re-attempt failed operations a specified number of times, often with increasing delays between attempts, to recover from transient failures.

**Technical Definition:** Per the concurrency patterns reference, "Retry on failure | Retry | Exponential backoff". The retry pattern wraps an operation in a loop with a configurable number of retries and a delay strategy (fixed, linear, or exponential). Exponential backoff increases the delay exponentially between attempts.

**Beginner-Friendly Explanation:** Sometimes a network request fails for a temporary reason — like a brief Wi-Fi drop or a server that's momentarily busy. Instead of giving up immediately, you wait a short time and try again. If it fails again, you wait a bit longer and try again.

### Purposes

- To recover from transient network failures without user intervention.
- To improve reliability in distributed systems where temporary failures are common.
- To reduce the number of user-visible errors caused by momentary glitches.
- To handle rate-limiting responses (HTTP 429) by backing off and retrying later.

### Syntax Rules and Structure

```js
async function withRetry(fn, { retries = 3, delay = 1000, backoff = 2 } = {}) {
  let lastError;
  for (let attempt = 0; attempt < retries; attempt++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error;
      if (attempt < retries - 1) {
        await new Promise(r => setTimeout(r, delay * backoff ** attempt));
      }
    }
  }
  throw lastError;
}
```

**Component Breakdown:**
- `retries`: Maximum number of attempts.
- `delay`: Initial delay in milliseconds.
- `backoff`: Multiplier for exponential backoff.
- `lastError`: The most recent error, thrown if all retries fail.

**Syntax Rules:**
- Retry should only be applied to **idempotent** operations.
- Non-retryable errors (e.g., 400, 401) should not be retried.
- The total time budget should be bounded.

**Constraints and Limitations:**
- Retrying non-idempotent operations can cause duplicate side effects.
- Without jitter, synchronised retries from many clients can overwhelm a recovering service.

### Annotated Code Examples

#### Example 1: Basic Retry with Exponential Backoff

```js
async function withRetry(fn, { retries = 3, delay = 1000, backoff = 2 } = {}) {
  let lastError;
  for (let attempt = 0; attempt < retries; attempt++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error;
      if (attempt < retries - 1) {
        const waitTime = delay * backoff ** attempt;
        console.log(`Attempt ${attempt + 1} failed. Retrying in ${waitTime}ms...`);
        await new Promise((r) => setTimeout(r, waitTime));
      }
    }
  }
  throw lastError;
}

// Usage:
await withRetry(() => fetch("/api/unreliable"));
```

**Expected Output (on failure then success):**
```
Attempt 1 failed. Retrying in 1000ms...
Attempt 2 failed. Retrying in 2000ms...
(success)
```

**Why This Output:** The operation is retried up to 3 times with exponentially increasing delays.

### Real-World Cases

- **API clients:** Retrying failed HTTP requests to external services.
- **Database connections:** Retrying connection attempts when a database is temporarily unavailable.
- **Message queues:** Retrying failed message processing with backoff.

---

## Timeout Patterns

### Definitions

**Core Definition:** Timeout patterns prevent an asynchronous operation from running indefinitely by rejecting the operation if it does not complete within a specified time limit.

**Technical Definition:** Per the concurrency patterns reference, "Time limit | Timeout | `Promise.withResolvers` + `Promise.race`". The timeout pattern races the operation against a timeout Promise that rejects after a specified delay. If the operation completes first, its result is returned. If the timeout fires first, the operation is abandoned (though not cancelled unless `AbortController` is used).

**Beginner-Friendly Explanation:** A timeout pattern is like setting an alarm clock: if the operation doesn't finish before the alarm rings, you give up on it.

### Purposes

- To prevent operations from hanging indefinitely.
- To provide a better user experience by failing fast.
- To free up resources tied up by slow operations.
- To implement service-level agreements (SLAs) for response times.

### Syntax Rules and Structure

```js
function withTimeout(promise, ms, message = 'Timeout') {
  const { promise: timeout, reject } = Promise.withResolvers();
  const id = setTimeout(() => reject(new Error(message)), ms);
  return Promise.race([promise, timeout]).finally(() => clearTimeout(id));
}
```

**Component Breakdown:**
- `Promise.withResolvers()`: Creates a Promise with exposed `resolve` and `reject`.
- `setTimeout(() => reject(...), ms)`: Rejects after `ms` milliseconds.
- `Promise.race([...])`: Settles with whichever completes first.

**Syntax Rules:**
- The timeout Promise must be cancelled if the operation completes first.
- `Promise.race` does not cancel the losing operation; use `AbortController` for cancellation.

**Constraints and Limitations:**
- The operation continues running in the background unless explicitly cancelled.
- `AbortSignal.timeout()` provides a more modern alternative.

### Annotated Code Examples

#### Example 1: Timeout with `Promise.race`

```js
function withTimeout(promise, ms, message = 'Timeout') {
  const { promise: timeout, reject } = Promise.withResolvers();
  const id = setTimeout(() => reject(new Error(message)), ms);
  return Promise.race([promise, timeout]).finally(() => clearTimeout(id));
}

const data = await withTimeout(fetchData(), 5000);
```

**Expected Output:** The data if it arrives within 5 seconds; a "Timeout" error otherwise.

**Why This Output:** `Promise.race` settles with the first Promise to settle — either the data fetch or the timeout.

#### Example 2: Timeout with `AbortSignal.timeout()`

```js
async function fetchWithTimeout(url, ms) {
  const response = await fetch(url, {
    signal: AbortSignal.timeout(ms),
  });
  return response.json();
}

try {
  const data = await fetchWithTimeout("/api/data", 5000);
} catch (error) {
  if (error.name === "TimeoutError") {
    console.log("Request timed out");
  }
}
```

**Expected Output:** The data if it arrives within 5 seconds; a `TimeoutError` otherwise.

**Why This Output:** `AbortSignal.timeout(ms)` returns a signal that automatically aborts after `ms` milliseconds. The fetch is cancelled, and a `TimeoutError` is thrown.

### Real-World Cases

- **API calls:** Ensuring requests don't hang indefinitely.
- **Database queries:** Preventing slow queries from blocking the connection pool.
- **User interactions:** Showing a timeout message if an operation takes too long.

---

## Cancellation

### Definitions

**Core Definition:** Cancellation is the pattern of aborting an in-flight asynchronous operation before it completes, using `AbortController` and `AbortSignal`.

**Technical Definition:** Per the concurrency patterns reference, "Cancel in-flight | Cancellation | `AbortController`". The `AbortController` interface represents a controller object that allows you to abort one or more Web requests as and when desired. It communicates with asynchronous operations through an `AbortSignal` object. Calling `controller.abort()` aborts the operation, and the operation rejects with an `AbortError`.

**Beginner-Friendly Explanation:** Cancellation lets you say "stop that operation, I don't need it anymore." For example, if a user navigates away from a page while a fetch is in progress, you can cancel the fetch to free up resources.

### Purposes

- To abort in-flight operations when they are no longer needed.
- To free up network and memory resources.
- To prevent stale responses from updating the UI.
- To implement timeout and cancellation in a unified way.

### Syntax Rules and Structure

```js
const controller = new AbortController();
const signal = controller.signal;

fetch(url, { signal })
  .then((response) => response.json())
  .catch((error) => {
    if (error.name === "AbortError") {
      console.log("Request cancelled");
    }
  });

// Cancel the request:
controller.abort();
```

**Component Breakdown:**
- `new AbortController()`: Creates a controller.
- `controller.signal`: The `AbortSignal` to pass to the operation.
- `controller.abort()`: Aborts the operation.

**Syntax Rules:**
- The `AbortSignal` must be passed to the operation (e.g., `fetch(url, { signal })`).
- `AbortController` can abort multiple operations if they share the same signal.
- `AbortSignal.timeout(ms)` creates a signal that aborts automatically after `ms`.

**Constraints and Limitations:**
- Not all APIs support `AbortSignal`; check compatibility.
- Aborting an operation may leave resources in an inconsistent state if not handled properly.

### Annotated Code Examples

#### Example 1: Cancellable Fetch

```js
const controller = new AbortController();

// Start a fetch with the signal.
const promise = fetch("/api/data", { signal: controller.signal });

// Cancel after 2 seconds.
setTimeout(() => controller.abort(), 2000);

try {
  const response = await promise;
  const data = await response.json();
  console.log(data);
} catch (error) {
  if (error.name === "AbortError") {
    console.log("Fetch was cancelled");
  }
}
```

**Expected Output:** The data if it arrives within 2 seconds; "Fetch was cancelled" otherwise.

**Why This Output:** `controller.abort()` triggers the `AbortSignal`, which cancels the fetch and rejects the Promise with an `AbortError`.

#### Example 2: Cancellable Operation Factory

```js
function createCancellableOperation(fn) {
  const controller = new AbortController();
  const promise = (async () => {
    try {
      return await fn(controller.signal);
    } catch (e) {
      if (e.name === "AbortError") return { cancelled: true };
      throw e;
    }
  })();
  return { promise, cancel: () => controller.abort() };
}

const { promise, cancel } = createCancellableOperation((signal) =>
  fetch("/api/data", { signal })
);

// Cancel whenever needed:
cancel();
```

**Expected Output:** The operation result or `{ cancelled: true }`.

**Why This Output:** The factory returns both the Promise and a `cancel` function that aborts the operation.

### Real-World Cases

- **User navigation:** Cancelling API requests when the user leaves a page.
- **Search autocomplete:** Cancelling previous requests when a new one starts.
- **File uploads:** Cancelling uploads when the user clicks "Cancel".

---

## `AbortController`

### Definitions

**Core Definition:** `AbortController` is an interface that represents a controller object allowing you to abort one or more Web requests as and when desired.

**Technical Definition:** Per MDN, the `AbortController` interface represents a controller object that allows you to abort one or more Web requests as and when desired. The constructor `AbortController()` creates a new `AbortController` object, and the `signal` property returns an `AbortSignal` object instance that can be used to communicate with or abort an asynchronous operation.

**Beginner-Friendly Explanation:** `AbortController` is the "remote control" for cancellation. You create one, get its `signal`, and pass the signal to the operation you want to cancel. When you're ready to cancel, you call `controller.abort()`.

### Purposes

- To create an `AbortSignal` that can be passed to cancellable operations.
- To abort one or more operations programmatically.
- To implement cancellation in fetch, streams, and custom async functions.

### Syntax Rules and Structure

```js
const controller = new AbortController();
const signal = controller.signal;
controller.abort();
```

**Component Breakdown:**
- `controller.signal`: The signal to pass to operations.
- `controller.abort()`: Aborts the operation.

**Syntax Rules:**
- `AbortController` is available in browsers and Node.js (v15+).
- `AbortSignal` can be passed to `fetch`, `addEventListener`, and other APIs.

**Constraints and Limitations:**
- `AbortController` cannot be reused after calling `abort()`.
- Not all asynchronous APIs support `AbortSignal`.

### Annotated Code Examples

#### Example 1: Basic AbortController

```js
const controller = new AbortController();

fetch("/api/data", { signal: controller.signal })
  .then((response) => response.json())
  .catch((error) => {
    if (error.name === "AbortError") {
      console.log("Request aborted");
    }
  });

// Abort the request after 1 second.
setTimeout(() => controller.abort(), 1000);
```

**Expected Output:** The data if it arrives within 1 second; "Request aborted" otherwise.

### Real-World Cases

- **Fetch cancellation:** Aborting network requests.
- **Stream cancellation:** Aborting stream reads.
- **Event listener removal:** Using `AbortSignal` with `addEventListener`.

---

## `AbortSignal`

### Definitions

**Core Definition:** `AbortSignal` is an interface that represents a signal object allowing you to communicate with a DOM request and abort it if required via an `AbortController` object.

**Technical Definition:** Per MDN, the `AbortSignal` interface represents a signal object that allows you to communicate with a DOM request and abort it if required via an `AbortController` object. The `AbortSignal.abort()` static method returns an already-aborted signal, and `AbortSignal.timeout(ms)` returns a signal that aborts automatically after `ms` milliseconds.

**Beginner-Friendly Explanation:** `AbortSignal` is the "wire" that carries the abort instruction from the controller to the operation. You don't create signals directly — you get them from an `AbortController` or from `AbortSignal.timeout()`.

### Purposes

- To pass cancellation instructions to asynchronous operations.
- To create timeout-based cancellation with `AbortSignal.timeout()`.
- To listen for abort events.

### Syntax Rules and Structure

```js
// From a controller:
const signal = controller.signal;

// Timeout-based:
const signal = AbortSignal.timeout(5000);

// Already aborted:
const signal = AbortSignal.abort();
```

**Component Breakdown:**
- `controller.signal`: The signal from a controller.
- `AbortSignal.timeout(ms)`: A signal that aborts after `ms`.
- `AbortSignal.abort()`: An already-aborted signal.

**Syntax Rules:**
- `AbortSignal` is read-only; you cannot manually abort it except through the controller.
- `AbortSignal.timeout()` aborts with a `TimeoutError`; `controller.abort()` aborts with an `AbortError`.

**Constraints and Limitations:**
- `AbortSignal.timeout()` is a relatively new API (Baseline 2024).
- A signal can only be aborted once.

### Annotated Code Examples

#### Example 1: `AbortSignal.timeout()`

```js
async function fetchData(url) {
  try {
    const response = await fetch(url, {
      signal: AbortSignal.timeout(3000),
    });
    return await response.json();
  } catch (error) {
    if (error.name === "TimeoutError") {
      console.log("Request timed out after 3 seconds");
    }
    throw error;
  }
}
```

**Expected Output:** The data if it arrives within 3 seconds; a timeout error otherwise.

### Real-World Cases

- **Timeout handling:** Automatically aborting slow requests.
- **Cancellation propagation:** Sharing a signal across multiple operations.

---

## Debouncing

### Definitions

**Core Definition:** Debouncing is a technique that postpones the execution of a function until after a certain amount of time has passed since its last invocation. It ensures a function is only called once after a sequence of rapid calls.

**Technical Definition:** Per the concurrency patterns reference, "Delay rapid calls | Debounce | `clearTimeout` + `Promise.withResolvers`". Debouncing uses `setTimeout` and `clearTimeout` to reset a timer on each invocation. The function executes only after the timer expires without any new calls.

**Beginner-Friendly Explanation:** Debouncing is like waiting for someone to finish typing before you act. If you're typing in a search box, you don't want to search on every keystroke. Debouncing waits until you stop typing for, say, 500ms, and then runs the search.

### Purposes

- To delay function execution until after a burst of calls has stopped.
- To reduce the number of API calls triggered by user input.
- To prevent double-clicks on buttons.
- To optimise window resize and scroll handlers.

### Syntax Rules and Structure

```js
function debounce(fn, delay) {
  let timeoutId;
  return function(...args) {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => fn.apply(this, args), delay);
  };
}
```

**Component Breakdown:**
- `timeoutId`: The timer ID, cleared on each new call.
- `setTimeout(() => fn(...), delay)`: Schedules the function after the delay.

**Syntax Rules:**
- The timer is cleared and reset on each call.
- The function executes only after the delay has passed without any new calls.
- `immediate` option can trigger on the leading edge instead of the trailing edge.

**Constraints and Limitations:**
- Debouncing delays execution; it does not guarantee execution at regular intervals.
- If calls never stop, the function never executes.

### Annotated Code Examples

#### Example 1: Basic Debounce

```js
function debounce(fn, delay) {
  let timeoutId;
  return function (...args) {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => fn.apply(this, args), delay);
  };
}

const search = debounce((query) => {
  console.log(`Searching for: ${query}`);
}, 500);

search("a");
search("ab");
search("abc");
// After 500ms, only "abc" is logged.
```

**Expected Output (after 500ms):**
```
Searching for: abc
```

**Why This Output:** Each call clears the previous timer and sets a new one. Only the last call triggers the search.

#### Example 2: Async Debounce with Promise Rejection

```js
function debounceAsync(fn, ms) {
  let timeoutId, pending = null;
  return (...args) => {
    clearTimeout(timeoutId);
    pending?.reject?.(new Error("Debounced"));
    const { promise, resolve, reject } = Promise.withResolvers();
    pending = { reject };
    timeoutId = setTimeout(async () => {
      try {
        resolve(await fn(...args));
      } catch (e) {
        reject(e);
      }
    }, ms);
    return promise;
  };
}

const debouncedSearch = debounceAsync(searchAPI, 300);
```

**Expected Output:** Only the last call resolves; previous pending calls reject with "Debounced".

**Why This Output:** The previous pending Promise is rejected when a new call arrives, ensuring only the final call's result is used.

### Real-World Cases

- **Search autocomplete:** Debouncing input to avoid firing a request on every keystroke.
- **Window resize:** Running resize logic after the user stops resizing.
- **Form validation:** Validating input after the user stops typing.

---

## Throttling

### Definitions

**Core Definition:** Throttling is a technique that limits the number of times a function can be called within a given time frame. It ensures a function executes at a consistent rate during an ongoing action.

**Technical Definition:** Per the concurrency patterns reference, "Rate limit calls | Throttle | Elapsed time check". Throttling uses a timestamp check to ensure the function executes at most once per specified interval.

**Beginner-Friendly Explanation:** Throttling is like putting a speed limit on a function. If you throttle a scroll handler to once every 200ms, it will run at most five times per second, no matter how many scroll events fire.

### Purposes

- To limit the execution frequency of high-frequency events (scroll, resize, mousemove).
- To improve performance by reducing the number of function calls.
- To ensure consistent execution during ongoing actions.

### Syntax Rules and Structure

```js
function throttle(fn, limit) {
  let lastCall = 0;
  return function(...args) {
    const now = Date.now();
    if (now - lastCall >= limit) {
      lastCall = now;
      fn.apply(this, args);
    }
  };
}
```

**Component Breakdown:**
- `lastCall`: Timestamp of the last execution.
- `now - lastCall >= limit`: Checks if enough time has passed.

**Syntax Rules:**
- Throttling does not delay execution; it limits frequency.
- The first call may be delayed (unless a leading edge option is implemented).

**Constraints and Limitations:**
- Throttling does not guarantee execution at exact intervals.
- The first call may be delayed.

### Annotated Code Examples

#### Example 1: Basic Throttle

```js
function throttle(fn, limit) {
  let lastCall = 0;
  return function (...args) {
    const now = Date.now();
    if (now - lastCall >= limit) {
      lastCall = now;
      fn.apply(this, args);
    }
  };
}

const handleScroll = throttle(() => {
  console.log("Scroll handled at:", new Date().toISOString());
}, 1000);

window.addEventListener("scroll", handleScroll);
```

**Expected Output:** During rapid scrolling, the handler logs at most once per second.

#### Example 2: Async Throttle

```js
function throttleAsync(fn, ms) {
  let lastCall = 0, pending = null;
  return async (...args) => {
    const elapsed = Date.now() - lastCall;
    if (elapsed >= ms) {
      lastCall = Date.now();
      return fn(...args);
    }
    if (!pending) {
      pending = new Promise((resolve) => {
        setTimeout(async () => {
          lastCall = Date.now();
          pending = null;
          resolve(await fn(...args));
        }, ms - elapsed);
      });
    }
    return pending;
  };
}
```

**Expected Output:** The function executes at most once per `ms` milliseconds.

### Real-World Cases

- **Scroll events:** Infinite scroll, lazy loading images.
- **Mouse movement:** Tracking cursor position at limited intervals.
- **API rate limiting:** Limiting requests to a server.
- **Game loops:** Updating game state at a fixed rate.

---

## Rate Limiting

### Definitions

**Core Definition:** Rate limiting is the pattern of controlling the number of requests a client can make to a server within a specified time window, often using algorithms like token bucket or sliding window.

**Technical Definition:** Rate limiting algorithms include: (1) **Token Bucket** — a bucket of tokens is refilled at a fixed rate; each request consumes a token. (2) **Sliding Window** — counts requests within a rolling time window. (3) **Fixed Window** — counts requests within fixed time intervals. Token bucket allows bursts up to the bucket capacity, then refills over time.

**Beginner-Friendly Explanation:** Rate limiting is like a bouncer at a club who only lets in a certain number of people per minute. If too many requests come in, some are turned away or delayed.

### Purposes

- To protect servers from being overwhelmed by too many requests.
- To comply with API rate limits imposed by external services.
- To ensure fair usage of shared resources.
- To prevent abuse and denial-of-service attacks.

### Syntax Rules and Structure

```js
class TokenBucket {
  constructor(capacity, refillRate) {
    this.capacity = capacity;
    this.tokens = capacity;
    this.refillRate = refillRate; // tokens per ms
    this.lastRefill = Date.now();
  }

  refill() {
    const now = Date.now();
    const elapsed = now - this.lastRefill;
    this.tokens = Math.min(this.capacity, this.tokens + elapsed * this.refillRate);
    this.lastRefill = now;
  }

  async acquire() {
    this.refill();
    if (this.tokens >= 1) {
      this.tokens -= 1;
      return;
    }
    const waitTime = (1 - this.tokens) / this.refillRate;
    await new Promise((r) => setTimeout(r, waitTime));
    return this.acquire();
  }
}
```

**Component Breakdown:**
- `capacity`: Maximum number of tokens.
- `refillRate`: Tokens added per millisecond.
- `acquire()`: Consumes a token or waits until one is available.

**Syntax Rules:**
- Token bucket allows bursts up to capacity.
- Sliding window provides smoother rate limiting but is more complex.

**Constraints and Limitations:**
- Rate limiting is client-side only; server-side rate limiting is also required.
- Distributed rate limiting requires a shared store (e.g., Redis).

### Annotated Code Examples

#### Example 1: Token Bucket Rate Limiter

```js
class TokenBucket {
  #capacity;
  #tokens;
  #refillRate;
  #lastRefill;

  constructor(capacity, refillRatePerMs) {
    this.#capacity = capacity;
    this.#tokens = capacity;
    this.#refillRate = refillRatePerMs;
    this.#lastRefill = Date.now();
  }

  #refill() {
    const now = Date.now();
    const elapsed = now - this.#lastRefill;
    this.#tokens = Math.min(
      this.#capacity,
      this.#tokens + elapsed * this.#refillRate
    );
    this.#lastRefill = now;
  }

  async acquire() {
    this.#refill();
    if (this.#tokens >= 1) {
      this.#tokens -= 1;
      return;
    }
    const waitTime = (1 - this.#tokens) / this.#refillRate;
    await new Promise((r) => setTimeout(r, waitTime));
    return this.acquire();
  }
}

// 10 requests per second (capacity 10, refill 0.01 tokens/ms)
const limiter = new TokenBucket(10, 0.01);
await limiter.acquire();
```

**Expected Output:** Requests are processed at a rate of 10 per second.

### Real-World Cases

- **API clients:** Respecting rate limits imposed by external APIs.
- **Web scraping:** Limiting request frequency to avoid being blocked.
- **Microservices:** Protecting internal services from traffic spikes.

---

## Backoff Strategies

### Definitions

**Core Definition:** Backoff strategies define how long to wait between retry attempts when an operation fails, with exponential backoff being the most common approach.

**Technical Definition:** Per the AWS documentation, "The intervening delay between each call is configured with exponential backoff and random jitter to avoid retry storms". Common strategies include: (1) **Fixed** — constant delay. (2) **Linear** — delay increases linearly. (3) **Exponential** — delay increases exponentially (e.g., 1s, 2s, 4s, 8s). (4) **Exponential with Jitter** — adds random variation to prevent synchronisation.

**Beginner-Friendly Explanation:** Backoff is like waiting longer each time you fail at something. If you fail once, wait 1 second. If you fail again, wait 2 seconds. Then 4, then 8, and so on. Jitter adds randomness so that many clients don't all retry at the same moment.

### Purposes

- To avoid overwhelming a recovering service with retries.
- To reduce the "thundering herd" problem where many clients retry simultaneously.
- To balance persistence with resource conservation.
- To handle rate-limiting responses (429) gracefully.

### Syntax Rules and Structure

```js
async function withBackoff(fn, { retries = 5, baseDelay = 1000, maxDelay = 30000 } = {}) {
  for (let attempt = 0; attempt < retries; attempt++) {
    try {
      return await fn();
    } catch (error) {
      if (attempt === retries - 1) throw error;
      const delay = Math.min(maxDelay, baseDelay * 2 ** attempt);
      const jitter = delay * (0.5 + Math.random() * 0.5);
      await new Promise((r) => setTimeout(r, jitter));
    }
  }
}
```

**Component Breakdown:**
- `baseDelay`: Initial delay.
- `2 ** attempt`: Exponential growth.
- `maxDelay`: Cap on the delay.
- `jitter`: Random variation (50%–100% of the delay).

**Syntax Rules:**
- Always cap the maximum delay.
- Add jitter to prevent synchronisation.
- Only retry idempotent operations.

**Constraints and Limitations:**
- Exponential backoff can lead to long wait times.
- Jitter reduces but does not eliminate retry storms.

### Annotated Code Examples

#### Example 1: Exponential Backoff with Jitter

```js
async function withBackoff(fn, { retries = 5, baseDelay = 1000, maxDelay = 30000 } = {}) {
  for (let attempt = 0; attempt < retries; attempt++) {
    try {
      return await fn();
    } catch (error) {
      if (attempt === retries - 1) throw error;
      const delay = Math.min(maxDelay, baseDelay * 2 ** attempt);
      const jitter = delay * (0.5 + Math.random() * 0.5);
      console.log(`Attempt ${attempt + 1} failed. Retrying in ${Math.round(jitter)}ms...`);
      await new Promise((r) => setTimeout(r, jitter));
    }
  }
}

await withBackoff(() => fetch("/api/unreliable"));
```

**Expected Output (on repeated failures):**
```
Attempt 1 failed. Retrying in 872ms...
Attempt 2 failed. Retrying in 1543ms...
Attempt 3 failed. Retrying in 3210ms...
...
```

**Why This Output:** The delay increases exponentially, with jitter adding randomness to each delay.

#### Example 2: Full Jitter (AWS Recommended)

```js
function fullJitter(baseDelay, maxDelay, attempt) {
  const exponentialDelay = Math.min(maxDelay, baseDelay * 2 ** attempt);
  return Math.random() * exponentialDelay;
}
```

**Expected Output:** The delay is uniformly random between 0 and the exponential delay.

**Why This Output:** AWS recommends full jitter because it provides the best spread of retry attempts.

### Real-World Cases

- **AWS SDK:** Uses exponential backoff with full jitter for retries.
- **Database connections:** Retrying with backoff when a database is overloaded.
- **Message queues:** Retrying failed message processing with backoff.

---

## References

- Concurrency Patterns — Modern JavaScript Skill — https://raw.githubusercontent.com/marcusquinn/aidevops/dfe4d7ebf15f8aa3a9deb4142213b42555862bdc/.agents/tools/programming/modern-javascript-skill/concurrency.md
- MDN Web Docs – AbortController — https://developer.mozilla.org/en-US/docs/Web/API/AbortController
- MDN Web Docs – AbortSignal — https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal
- MDN Web Docs – AbortSignal.timeout() — https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal/timeout_static
- AWS – Retry behavior — https://docs.aws.amazon.com/sdkref/latest/guide/feature-retry-behavior.html
- AWS – Error Retries and Exponential Backoff — https://docs.aws.amazon.com/general/latest/gr/api-retries.html
- Node.js Documentation – AbortSignal — https://nodejs.org/api/globals.html
- p-limit – npm — https://www.npmjs.com/package/p-limit
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) — https://ecma-international.org/wp-content/uploads/ECMA-262_16th_edition_june_2025.pdf