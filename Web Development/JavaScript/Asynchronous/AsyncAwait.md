# Async/Await — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** `async/await` is syntactic sugar built on top of Promises that allows asynchronous, promise-based code to be written in a synchronous style. The `async` keyword marks a function as asynchronous, and the `await` operator pauses the function's execution until a Promise settles, returning its fulfilled value or throwing its rejection reason.

**Technical Definition:** Per the ECMAScript specification, an `async function` declaration creates an `AsyncFunction` object. When called, an async function always returns a Promise that is resolved with the function's return value or rejected with an uncaught exception. The `await` operator, which can only be used inside async functions or at the top level of modules, pauses the execution of its surrounding async function until the passed Promise settles, then resumes it and returns the fulfillment value. This "staged execution" allows the function to behave as if it were synchronous between `await` points while still managing asynchronous operations.

**Beginner-Friendly Explanation:** `async/await` lets you write asynchronous code that looks like regular synchronous code. You put `async` before a function, and then use `await` before any Promise. The `await` makes the function wait for the Promise to resolve, then continues. It's much easier to read than a chain of `.then()` calls. When an async function is called, it runs synchronously until the first `await`, then pauses, yields control back to the caller, and resumes later when the awaited Promise settles.

### Key Characteristics

- **Syntactic sugar:** `async/await` is built on Promises; it does not replace them.
- **Always returns a Promise:** An async function always returns a Promise, regardless of whether the `return` statement explicitly returns one.
- **`await` pauses execution:** `await` suspends the async function until the Promise settles; it does not block the main thread.
- **`try...catch` works:** Errors from awaited Promises can be caught with synchronous-style `try...catch` blocks.
- **Sequential by default:** Multiple `await` expressions execute sequentially; use `Promise.all()` for concurrency.
- **Top-level await:** In ES modules, `await` can be used at the top level outside any function.
- **Async iteration:** `for await...of` iterates over async iterables, and async generators (`async function*`) combine `yield` with `await`.

### Prerequisites

- Understanding of Promises (states, `then`, `catch`, `finally`, chaining).
- Familiarity with the event loop and microtask queue.
- Knowledge of `try...catch` error handling.
- Awareness of ES modules (for top-level await).

### Related Programming Areas

- **Asynchronous JavaScript:** The primary use case for `async/await`.
- **Node.js:** `fs.promises`, `stream/promises`, and modern APIs.
- **Web development:** Fetch API, asynchronous data loading.
- **Async iteration:** Streams, paginated APIs, and event-driven data sources.

### Core Concepts / Features

---

## `async` Functions

### Definitions

**Core Definition:** An `async function` is a function declared with the `async` keyword that always returns a Promise and allows the use of `await` inside its body.

**Technical Definition:** Per MDN, "The `async function` declaration creates a binding of a new async function to a given name. The `await` keyword is permitted within the function body, enabling asynchronous, promise-based behavior to be written in a cleaner style and avoiding the need to explicitly configure promise chains". The `async` and `function` keywords must be on the same line; a line terminator between them triggers Automatic Semicolon Insertion, making `async` an identifier.

**Beginner-Friendly Explanation:** An async function is a regular function with the `async` keyword added. It always returns a Promise, and you can use `await` inside it to wait for other Promises.

### Purposes

- To write asynchronous code that reads like synchronous code.
- To eliminate the need for explicit Promise chains (`.then()` / `.catch()`).
- To enable the use of `await` for pausing execution until a Promise settles.
- To automatically wrap return values in a Promise.

### Syntax Rules and Structure

```js
async function name(param0, param1, ...) {
  // statements
  const result = await somePromise();
  return result;
}
```

**Component Breakdown:**
- `async`: Keyword that marks the function as asynchronous.
- `name`: The function name.
- `param0, param1, ...`: The parameter list.
- `return result`: Automatically wrapped in a resolved Promise.

**Syntax Rules:**
- `async` and `function` must not be separated by a line terminator.
- An async function always returns a Promise.
- `await` is only valid inside async functions or at the top level of modules.

**Constraints and Limitations:**
- Async functions cannot be used as constructors (no `new`).
- Async functions cannot be generators (use `async function*` instead).
- Throwing inside an async function rejects the returned Promise.

### Annotated Code Examples

#### Example 1: Basic Async Function

```js
// Step 1: Define an async function.
async function greet() {
  return "Hello, async!";
}

// Step 2: Call it — it returns a Promise.
greet().then((message) => {
  console.log(message); // "Hello, async!"
});

console.log("This runs before the async result");
```

**Expected Output:**
```
This runs before the async result
Hello, async!
```

**Why This Output:** The async function returns a Promise that resolves with `"Hello, async!"`. The `.then()` handler runs asynchronously, after the synchronous `console.log`.

#### Example 2: Async Function Execution Flow

```js
// Step 1: Async function with await.
async function fetchData() {
  console.log("1. Start fetching");
  const result = await Promise.resolve("data");
  console.log("3. Data received:", result);
  return result;
}

// Step 2: Call the function.
console.log("2. Before calling fetchData");
fetchData().then(() => console.log("4. Fetch complete"));
console.log("2b. After calling fetchData");
```

**Expected Output:**
```
2. Before calling fetchData
1. Start fetching
2b. After calling fetchData
3. Data received: data
4. Fetch complete
```

**Why This Output:** The async function runs synchronously until the first `await`. The `await` pauses the function and yields control back to the caller, so `"2b"` logs before `"3"`. After the Promise resolves, the function resumes.

### Real-World Cases

- **API calls:** `async function getUser(id) { const res = await fetch('/api/users/' + id); return res.json(); }`.
- **File I/O:** `async function readFile(path) { return await fs.promises.readFile(path, 'utf8'); }`.
- **Database queries:** `async function findUser(id) { return await db.collection('users').findOne({ _id: id }); }`.

---

## `await` Operator

### Definitions

**Core Definition:** The `await` operator pauses the execution of its surrounding async function until a Promise settles, returning the fulfillment value or throwing the rejection reason.

**Technical Definition:** Per MDN, "The `await` operator is used to wait for a Promise and get its fulfillment value. It can only be used inside an async function or at the top level of a module". The expression is resolved the same way as `Promise.resolve()`: native Promises are awaited directly, thenables are converted to native Promises, and non-thenable values are wrapped in an already-fulfilled Promise. Even if the Promise is already fulfilled, the async function's execution is still paused until the next tick.

**Beginner-Friendly Explanation:** `await` means "wait for this Promise to finish, then give me the result." It can only be used inside an async function (or at the top level of a module). It doesn't block the whole program — it only pauses the async function it's inside.

### Purposes

- To pause an async function until a Promise settles.
- To obtain the fulfillment value of a Promise without using `.then()`.
- To enable sequential asynchronous operations in a readable way.
- To convert Promise rejections into catchable exceptions.

### Syntax Rules and Structure

```js
const value = await expression;
```

**Component Breakdown:**
- `expression`: A Promise, thenable, or any value.
- If the expression is a Promise, `await` returns its fulfillment value.
- If the expression is not a thenable, `await` returns the value itself.

**Syntax Rules:**
- `await` can only be used inside async functions or at the top level of modules.
- If the awaited Promise rejects, `await` throws the rejection reason.
- `await` always yields to the event loop, even for already-resolved Promises.

**Constraints and Limitations:**
- Using `await` outside an async function (or module top level) is a `SyntaxError`.
- `await` in a loop causes sequential execution; use `Promise.all()` for concurrency.
- Awaiting a non-Promise value still yields to the event loop (unnecessary overhead).

### Annotated Code Examples

#### Example 1: Basic `await` Usage

```js
// Step 1: Define an async function.
async function example() {
  // Step 2: Await a Promise.
  const value = await Promise.resolve(42);
  console.log(value); // 42
}

example();
```

**Expected Output:**
```
42
```

**Why This Output:** `await Promise.resolve(42)` pauses the function until the Promise resolves, then assigns `42` to `value`.

#### Example 2: `await` with Rejection

```js
async function example() {
  try {
    const value = await Promise.reject(new Error("Failed"));
  } catch (err) {
    console.log("Caught:", err.message); // "Caught: Failed"
  }
}

example();
```

**Expected Output:**
```
Caught: Failed
```

**Why This Output:** When the awaited Promise rejects, `await` throws the rejection reason as an exception, which is caught by the `try...catch` block.

### Real-World Cases

- **Fetch API:** `const response = await fetch(url); const data = await response.json();`.
- **Delays:** `await new Promise(resolve => setTimeout(resolve, 1000));`.
- **Reading streams:** `const chunk = await reader.read();`.

---

## Async Function Return Values

### Definitions

**Core Definition:** An async function always returns a Promise. If the function returns a value, the Promise is fulfilled with that value. If the function throws an error, the Promise is rejected with that error.

**Technical Definition:** Per MDN, "Each time an async function is called, it returns a new Promise which will be resolved with the value returned by the async function, or rejected with an exception uncaught within the async function". If the return value is already a Promise, the async function's returned Promise adopts its state.

**Beginner-Friendly Explanation:** When you return a value from an async function, JavaScript automatically wraps it in a resolved Promise. If you throw an error, it becomes a rejected Promise. You can also return a Promise, and the async function will wait for it.

### Purposes

- To automatically wrap synchronous return values in a Promise.
- To propagate errors as Promise rejections.
- To allow async functions to be used interchangeably with Promise-returning functions.

### Syntax Rules and Structure

```js
async function example() {
  return value;          // Returns Promise.resolve(value)
  // or
  throw new Error(...);  // Returns Promise.reject(error)
  // or
  return somePromise;    // Adopts the state of somePromise
}
```

**Component Breakdown:**
- `return value`: The value is wrapped in a resolved Promise.
- `throw error`: The error is wrapped in a rejected Promise.
- `return somePromise`: The returned Promise adopts the state of `somePromise`.

**Syntax Rules:**
- The return value is always a Promise, even if the function explicitly returns a non-Promise.
- If no value is returned, the Promise resolves with `undefined`.

**Constraints and Limitations:**
- Returning a Promise from an async function adds an extra microtask tick compared to returning a value directly.
- The async function's Promise identity is distinct from the returned Promise (when returning a Promise).

### Annotated Code Examples

#### Example 1: Return Value Wrapping

```js
async function getValue() {
  return 42;
}

getValue().then((value) => {
  console.log(value); // 42
  console.log(value instanceof Promise); // false
});
```

**Expected Output:**
```
42
false
```

**Why This Output:** `return 42` is automatically wrapped in `Promise.resolve(42)`. The `.then()` handler receives the unwrapped value `42`.

#### Example 2: Returning a Promise

```js
async function getData() {
  return fetch("/api/data"); // Returns a Promise
}

getData().then((response) => {
  console.log(response.status); // 200 (or whatever)
});
```

**Expected Output:** The response status is logged.

**Why This Output:** The async function returns a Promise. The async function's returned Promise adopts the state of the fetch Promise.

### Real-World Cases

- **API wrappers:** Async functions that return the result of `fetch().then(r => r.json())`.
- **Database queries:** Async functions that return query results.
- **Utility functions:** Async functions that return computed values.

---

## Error Handling with `try/catch`

### Definitions

**Core Definition:** `try...catch` blocks in async functions catch errors from rejected Promises and synchronous throws, providing a unified error-handling mechanism.

**Technical Definition:** Per MDN, "Using `async` and `await` allows you to use ordinary `try / catch` blocks around asynchronous code". When an awaited Promise rejects, the `await` expression throws the rejection reason, which propagates up the async function and can be caught by a surrounding `try...catch` block. Synchronous errors thrown inside the async function are also caught.

**Beginner-Friendly Explanation:** With `async/await`, you can use the same `try...catch` syntax you already know for synchronous code. If an awaited Promise fails, the error is thrown and caught by the nearest `catch` block.

### Purposes

- To centralise error handling for multiple awaited operations.
- To unify synchronous and asynchronous error handling.
- To provide a familiar syntax for developers coming from synchronous programming.

### Syntax Rules and Structure

```js
async function example() {
  try {
    const result = await riskyOperation();
    return result;
  } catch (error) {
    // Handle error
    console.error(error.message);
  } finally {
    // Cleanup (optional)
  }
}
```

**Component Breakdown:**
- `try`: Contains the code that may throw (including awaited Promises that reject).
- `catch (error)`: Receives the rejection reason or thrown error.
- `finally`: Executes regardless of outcome.

**Syntax Rules:**
- `try...catch` works for both synchronous throws and Promise rejections.
- A single `try...catch` can wrap multiple `await` expressions.
- Errors not caught inside the async function reject the returned Promise.

**Constraints and Limitations:**
- Forgetting `try...catch` around `await` leads to unhandled rejections.
- `try...catch` can make code verbose if used around every `await`.

### Annotated Code Examples

#### Example 1: Basic Error Handling

```js
async function fetchUser(id) {
  try {
    const response = await fetch(`/api/users/${id}`);
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }
    return await response.json();
  } catch (error) {
    console.error("Failed to fetch user:", error.message);
    throw error; // Re-throw to let the caller handle it
  }
}

fetchUser(123).catch((err) => {
  console.log("Caller caught:", err.message);
});
```

**Expected Output:**
```
Failed to fetch user: HTTP 404
Caller caught: HTTP 404
```

**Why This Output:** The `try...catch` inside `fetchUser` catches the error, logs it, and re-throws it. The caller's `.catch()` handles the re-thrown error.

#### Example 2: Multiple Awaits with One `catch`

```js
async function processData() {
  try {
    const user = await fetchUser();
    const posts = await fetchPosts(user.id);
    const comments = await fetchComments(posts[0].id);
    return comments;
  } catch (error) {
    console.error("Processing failed:", error.message);
    return []; // Fallback
  }
}
```

**Expected Output:** If any of the three awaits rejects, the `catch` block logs the error and returns an empty array.

**Why This Output:** A single `try...catch` wraps all three awaits, so any rejection is caught in one place.

### Real-World Cases

- **API error handling:** Catching network errors and showing user-friendly messages.
- **Fallback data:** Returning cached data when a request fails.
- **Validation:** Throwing and catching validation errors.

---

## Sequential Execution

### Definitions

**Core Definition:** Sequential execution means multiple asynchronous operations run one after another, each waiting for the previous one to complete before starting.

**Technical Definition:** Per the async patterns reference, "One at a time | Sequential | `for...of` + await". When multiple `await` expressions appear in sequence, each one pauses the function until the awaited Promise settles before proceeding to the next. This is the default behaviour of `await` and is appropriate when operations depend on each other's results.

**Beginner-Friendly Explanation:** Sequential execution is like standing in line at a bank: you wait for the person in front of you to finish before you step up. Each operation waits for the previous one to complete.

### Purposes

- To execute operations that depend on the result of previous operations.
- To control the order of asynchronous operations.
- To avoid overwhelming a resource with concurrent requests.

### Syntax Rules and Structure

```js
async function sequential() {
  const result1 = await operation1();
  const result2 = await operation2(result1); // Uses result1
  const result3 = await operation3(result2); // Uses result2
  return result3;
}
```

**Component Breakdown:**
- Each `await` pauses until the previous operation completes.
- Later operations can use the results of earlier ones.

**Syntax Rules:**
- `await` in sequence executes sequentially.
- `for...of` with `await` iterates sequentially.
- `forEach` with `async/await` does **not** work as expected (callbacks are not awaited).

**Constraints and Limitations:**
- Sequential execution is slower than concurrent execution for independent operations.
- Using `await` in a loop for independent operations is an anti-pattern.

### Annotated Code Examples

#### Example 1: Sequential Async Operations

```js
async function loadUserData(userId) {
  console.log("Loading user...");
  const user = await fetchUser(userId);
  console.log("Loading posts...");
  const posts = await fetchPosts(user.id);
  console.log("Loading comments...");
  const comments = await fetchComments(posts[0].id);
  return { user, posts, comments };
}
```

**Expected Output:**
```
Loading user...
Loading posts...
Loading comments...
```

**Why This Output:** Each `await` pauses until the previous operation completes. The logs appear in order.

#### Example 2: Sequential Loop with `for...of`

```js
async function processItems(items) {
  const results = [];
  for (const item of items) {
    const result = await processItem(item);
    results.push(result);
  }
  return results;
}
```

**Expected Output:** Items are processed one at a time, in order.

**Why This Output:** `for...of` with `await` processes each item sequentially.

### Real-World Cases

- **User onboarding:** Fetch user → create profile → send welcome email.
- **E-commerce checkout:** Validate cart → process payment → confirm order.
- **Pagination:** Fetch page 1 → fetch page 2 → fetch page 3.

---

## Concurrent Execution

### Definitions

**Core Definition:** Concurrent execution means multiple asynchronous operations run in parallel, with the total time approximately equal to the longest operation rather than the sum of all operations.

**Technical Definition:** Per the async patterns reference, "All at once | Parallel | `Promise.all`". To run independent operations concurrently, start all Promises first, then await them together using `Promise.all()`, `Promise.allSettled()`, etc. Per the reference, "Promise.all() on the other hand, runs all promises in parallel and waits for all of them to finish".

**Beginner-Friendly Explanation:** Concurrent execution is like ordering several dishes at a restaurant: they all start cooking at the same time, and you get them when they're ready. You don't wait for one dish to finish before ordering the next.

### Purposes

- To reduce total execution time for independent operations.
- To efficiently use network bandwidth and server resources.
- To improve application performance.

### Syntax Rules and Structure

```js
async function concurrent() {
  const [result1, result2, result3] = await Promise.all([
    operation1(),
    operation2(),
    operation3(),
  ]);
  return { result1, result2, result3 };
}
```

**Component Breakdown:**
- `Promise.all([...])`: Starts all operations and waits for all to fulfill.
- Destructuring: Assigns each result to a named variable.

**Syntax Rules:**
- Start all Promises before awaiting them.
- Use `Promise.all()` for fail-fast concurrency.
- Use `Promise.allSettled()` for fault-tolerant concurrency.

**Constraints and Limitations:**
- `Promise.all()` rejects immediately if any input rejects.
- Unbounded concurrency can exhaust resources (use `p-limit` for controlled concurrency).

### Annotated Code Examples

#### Example 1: Concurrent Fetches with `Promise.all()`

```js
async function loadDashboard() {
  console.log("Starting all requests...");
  const [user, posts, stats] = await Promise.all([
    fetch("/api/user").then((r) => r.json()),
    fetch("/api/posts").then((r) => r.json()),
    fetch("/api/stats").then((r) => r.json()),
  ]);
  console.log("All requests complete");
  return { user, posts, stats };
}
```

**Expected Output:**
```
Starting all requests...
All requests complete
```

**Why This Output:** All three fetches start simultaneously. `Promise.all()` waits for all to complete, then logs the message.

#### Example 2: Controlled Concurrency with `p-limit`

```js
import pLimit from "p-limit";

const limit = pLimit(5); // Max 5 concurrent operations

async function processAll(items) {
  return Promise.all(
    items.map((item) => limit(() => processItem(item)))
  );
}
```

**Expected Output:** Items are processed with at most 5 concurrent operations.

**Why This Output:** `p-limit` wraps each operation, ensuring no more than 5 run concurrently.

### Real-World Cases

- **Dashboard loading:** Fetching multiple widgets concurrently.
- **Batch operations:** Processing multiple items in parallel.
- **Mirror racing:** Using the fastest response from multiple servers.

---

## `Promise.all()` with Await

### Definitions

**Core Definition:** `Promise.all()` with `await` starts multiple Promises concurrently and waits for all of them to fulfill, returning an array of their values.

**Technical Definition:** Per MDN, `Promise.all()` takes an iterable of Promises and returns a single Promise that fulfills when all inputs fulfill, with an array of fulfillment values. It rejects immediately if any input rejects.

**Beginner-Friendly Explanation:** `await Promise.all([...])` is like saying "start all these tasks at once, and wait until every single one is done."

### Purposes

- To run independent asynchronous operations concurrently.
- To wait for all operations to complete before proceeding.
- To collect results from multiple sources into a single array.

### Syntax Rules and Structure

```js
const [result1, result2] = await Promise.all([promise1, promise2]);
```

**Component Breakdown:**
- `Promise.all([...])`: The combinator.
- `await`: Waits for the combined Promise.

**Syntax Rules:**
- The array order is preserved in the result.
- Rejects immediately if any input rejects.

**Constraints and Limitations:**
- A single rejection causes the entire `Promise.all()` to reject.
- Cannot handle partial results.

### Annotated Code Examples

#### Example 1: Concurrent Data Loading

```js
async function loadData() {
  const [users, posts] = await Promise.all([
    fetch("/api/users").then((r) => r.json()),
    fetch("/api/posts").then((r) => r.json()),
  ]);
  console.log("Users:", users.length);
  console.log("Posts:", posts.length);
}
```

**Expected Output:**
```
Users: 100
Posts: 50
```

**Why This Output:** Both fetches run concurrently. `Promise.all()` waits for both and returns their results in order.

### Real-World Cases

- **Dashboard loading:** Fetching multiple widgets concurrently.
- **Batch API calls:** Sending multiple requests at once.

---

## `Promise.allSettled()` with Await

### Definitions

**Core Definition:** `Promise.allSettled()` with `await` waits for all Promises to settle (fulfill or reject), returning an array of status objects.

**Technical Definition:** Per MDN, `Promise.allSettled()` returns a Promise that fulfills when all inputs settle, with an array of objects containing `status` and either `value` or `reason`.

**Beginner-Friendly Explanation:** `await Promise.allSettled([...])` is like saying "start all these tasks, and wait until every single one finishes — regardless of whether it succeeded or failed."

### Purposes

- To run multiple independent operations and get the result of each.
- To handle partial failures without losing successful results.
- To implement batch operations where each item's outcome matters.

### Syntax Rules and Structure

```js
const results = await Promise.allSettled([promise1, promise2]);
// results[0] = { status: 'fulfilled', value: ... } or { status: 'rejected', reason: ... }
```

**Component Breakdown:**
- Each result has a `status` property (`"fulfilled"` or `"rejected"`).
- Fulfilled results have a `value`; rejected have a `reason`.

**Syntax Rules:**
- Never rejects (unless the iterable itself is invalid).
- Always waits for all inputs to settle.

**Constraints and Limitations:**
- Cannot be short-circuited.
- Does not provide a single aggregated error.

### Annotated Code Examples

#### Example 1: Fault-Tolerant Batch Processing

```js
async function uploadFiles(files) {
  const results = await Promise.allSettled(
    files.map((file) => uploadFile(file))
  );

  const succeeded = results.filter((r) => r.status === "fulfilled");
  const failed = results.filter((r) => r.status === "rejected");

  console.log(`Uploaded: ${succeeded.length}, Failed: ${failed.length}`);
  return { succeeded, failed };
}
```

**Expected Output:**
```
Uploaded: 3, Failed: 1
```

**Why This Output:** `Promise.allSettled()` waits for all uploads to settle, then the code counts successes and failures.

### Real-World Cases

- **Batch uploads:** Uploading multiple files and reporting per-file status.
- **Notifications:** Sending notifications to multiple users and logging successes/failures.
- **Health checks:** Pinging multiple services and reporting their availability.

---

## `Promise.race()` with Await

### Definitions

**Core Definition:** `Promise.race()` with `await` settles with the first Promise to settle (fulfill or reject).

**Technical Definition:** Per MDN, `Promise.race()` returns a Promise that settles with the eventual state of the first input Promise to settle.

**Beginner-Friendly Explanation:** `await Promise.race([...])` is like a race — whichever Promise finishes first (whether it succeeds or fails) determines the result.

### Purposes

- To implement timeouts for asynchronous operations.
- To get the result of the fastest operation.
- To cancel or abandon slow operations when a faster one completes.

### Syntax Rules and Structure

```js
const result = await Promise.race([promise1, promise2]);
```

**Component Breakdown:**
- Returns the value or rejection reason of the first settled Promise.

**Syntax Rules:**
- Non-Promise values are treated as already fulfilled.
- Empty iterable returns a forever-pending Promise.

**Constraints and Limitations:**
- Does not cancel the other Promises.
- The first rejection causes the returned Promise to reject.

### Annotated Code Examples

#### Example 1: Timeout Pattern

```js
function timeout(ms) {
  return new Promise((_, reject) =>
    setTimeout(() => reject(new Error("Timeout")), ms)
  );
}

async function fetchWithTimeout(url, ms) {
  try {
    const response = await Promise.race([
      fetch(url),
      timeout(ms),
    ]);
    return response;
  } catch (error) {
    console.error(error.message); // "Timeout" if fetch takes too long
  }
}
```

**Expected Output:** The fetch response, or "Timeout" if it exceeds `ms` milliseconds.

**Why This Output:** `Promise.race()` settles with whichever completes first — the fetch or the timeout.

### Real-World Cases

- **Timeouts:** Abandoning operations that take too long.
- **Mirror racing:** Using the fastest response from multiple servers.

---

## `Promise.any()` with Await

### Definitions

**Core Definition:** `Promise.any()` with `await` fulfills with the first fulfillment value. If all Promises reject, it rejects with an `AggregateError`.

**Technical Definition:** Per MDN, `Promise.any()` returns a Promise that fulfills when any input fulfills, with the first fulfillment value. It rejects only if all inputs reject, with an `AggregateError` containing an array of rejection reasons.

**Beginner-Friendly Explanation:** `await Promise.any([...])` is like asking several friends for a favour. You only need one to say yes. You ignore the "no" responses and wait for the first "yes."

### Purposes

- To get the first successful result from multiple sources.
- To implement fallback mechanisms.
- To ignore failures and only care about success.

### Syntax Rules and Structure

```js
try {
  const value = await Promise.any([promise1, promise2]);
} catch (error) {
  // error is AggregateError with error.errors array
}
```

**Component Breakdown:**
- Fulfills with the first fulfillment value.
- Rejects with `AggregateError` if all reject.

**Syntax Rules:**
- Does not cancel the other Promises.
- The `AggregateError` contains all rejection reasons.

**Constraints and Limitations:**
- Does not cancel the other Promises when one fulfills.
- The `AggregateError` contains all rejection reasons.

### Annotated Code Examples

#### Example 1: First Successful Fetch

```js
async function fetchFromMirrors(urls) {
  try {
    const response = await Promise.any(
      urls.map((url) => fetch(url))
    );
    return await response.json();
  } catch (error) {
    console.error("All mirrors failed:");
    for (const err of error.errors) {
      console.error("-", err.message);
    }
  }
}
```

**Expected Output:** The JSON data from the first successful mirror, or a list of errors if all fail.

**Why This Output:** `Promise.any()` ignores rejections and fulfills with the first successful response.

### Real-World Cases

- **CDN fallback:** Trying multiple CDNs and using the first successful response.
- **Redundant APIs:** Fetching from multiple endpoints and using the fastest success.

---

## Top-Level Await

### Definitions

**Core Definition:** Top-level await allows the `await` keyword to be used at the top level of an ECMAScript module, outside any async function.

**Technical Definition:** Per the TC39 proposal, "Top-level await enables modules to act as big async functions: With top-level await, ECMAScript Modules (ESM) can await resources, causing other modules who import them to wait before they start evaluating their body". It is Stage 4 (standardised) and works in ES modules.

**Beginner-Friendly Explanation:** Normally, `await` can only be used inside an async function. Top-level await lets you use `await` directly in a module's top-level code, making the module wait for asynchronous operations before its exports are available.

### Purposes

- To await resources at module initialisation time.
- To eliminate the need for IIAFEs (Immediately Invoked Async Function Expressions).
- To enable dynamic imports and conditional module loading.

### Syntax Rules and Structure

```js
// In an ES module (.mjs or type: "module"):
const data = await fetch("/api/data");
export const result = await data.json();
```

**Component Breakdown:**
- `await` at the top level of a module.
- Imports of this module will wait until the await resolves.

**Syntax Rules:**
- Only works in ES modules (`.mjs` or `"type": "module"` in `package.json`).
- Works in `<script type="module">` in browsers.
- Not allowed in CommonJS or traditional scripts.

**Constraints and Limitations:**
- Not supported in CommonJS.
- A module that never resolves will cause the importing module to hang.
- Top-level await can introduce deadlocks if modules have circular dependencies.

### Annotated Code Examples

#### Example 1: Basic Top-Level Await

```js
// data.mjs
const response = await fetch("/api/data");
const data = await response.json();
export default data;

// main.mjs
import data from "./data.mjs";
console.log(data); // The fetched data
```

**Expected Output:** The fetched data is logged after the module loads.

**Why This Output:** The `data.mjs` module awaits the fetch before exporting. `main.mjs` waits for `data.mjs` to finish loading.

#### Example 2: Dynamic Import with Top-Level Await

```js
// module.mjs
const moduleName = "utils";
const { default: utils } = await import(`./${moduleName}.mjs`);
export default utils;
```

**Expected Output:** The dynamically imported module is available.

**Why This Output:** Top-level await allows dynamic imports to be resolved before the module's exports are used.

### Real-World Cases

- **Configuration loading:** Fetching config before exporting it.
- **Database connections:** Establishing a connection at module load time.
- **Conditional imports:** Loading different modules based on environment.

---

## Async Iteration

### Definitions

**Core Definition:** Async iteration is the process of iterating over asynchronous data sources using the async iterator protocol and the `for await...of` loop.

**Technical Definition:** Per MDN, "The `for await...of` statement creates a loop iterating over async iterable objects as well as sync iterables". An async iterable implements the `Symbol.asyncIterator` method, which returns an async iterator whose `next()` method returns a Promise for an iterator result object.

**Beginner-Friendly Explanation:** Async iteration lets you loop over data that arrives asynchronously — like reading chunks from a stream or fetching pages from an API. `for await...of` waits for each value to be ready before processing it.

### Purposes

- To process data from asynchronous sources in a loop.
- To handle streams, paginated APIs, and event-driven data.
- To combine `await` with iteration in a readable way.

### Syntax Rules and Structure

```js
for await (const value of asyncIterable) {
  // process value
}
```

**Component Breakdown:**
- `asyncIterable`: An object implementing `Symbol.asyncIterator` or a sync iterable.
- `value`: The current value, awaited before assignment.

**Syntax Rules:**
- Only valid inside async functions or at the top level of modules.
- Works on both sync and async iterables.
- Each iteration awaits the next value.

**Constraints and Limitations:**
- Sequential: each iteration waits for the previous one.
- Cannot be used with `break`/`continue` in some older engines (now supported).

### Annotated Code Examples

#### Example 1: Iterating an Async Iterable

```js
async function processStream(stream) {
  for await (const chunk of stream) {
    console.log("Chunk:", chunk);
  }
  console.log("Stream complete");
}
```

**Expected Output:** Each chunk is logged as it arrives.

**Why This Output:** `for await...of` waits for each chunk to be available before processing it.

#### Example 2: Paginated API

```js
async function* fetchPages(baseUrl) {
  let page = 1;
  while (true) {
    const response = await fetch(`${baseUrl}?page=${page}`);
    const data = await response.json();
    if (data.length === 0) break;
    yield data;
    page++;
  }
}

async function getAllData() {
  const allItems = [];
  for await (const page of fetchPages("/api/items")) {
    allItems.push(...page);
  }
  return allItems;
}
```

**Expected Output:** All items are collected across pages.

**Why This Output:** The async generator yields each page, and `for await...of` processes them sequentially.

### Real-World Cases

- **Streams:** Reading from Node.js readable streams.
- **Paginated APIs:** Iterating through API pages.
- **Event streams:** Processing Server-Sent Events (SSE).

---

## `for await...of`

### Definitions

**Core Definition:** `for await...of` is a loop statement that iterates over async iterable objects as well as sync iterables, awaiting each value.

**Technical Definition:** Per MDN, "The `for await...of` statement creates a loop iterating over async iterable objects as well as sync iterables". It can only be used in contexts where `await` is valid (async function bodies and module top level).

**Beginner-Friendly Explanation:** `for await...of` is like a regular `for...of` loop, but it waits for each value to be available before processing it. It's designed for async data sources.

### Purposes

- To iterate over async data sources.
- To process streams and paginated data.
- To combine iteration with `await` in a readable way.

### Syntax Rules and Structure

```js
for await (const variable of iterable) {
  // statement
}
```

**Component Breakdown:**
- `variable`: Receives each value from the sequence.
- `iterable`: An async iterable or sync iterable.

**Syntax Rules:**
- Only valid inside async functions or at the top level of modules.
- Works on both sync and async iterables.
- Variables declared with `var` are not local to the loop.

**Constraints and Limitations:**
- Sequential: each iteration waits for the previous one.
- Not a replacement for `Promise.all()` when concurrency is desired.

### Annotated Code Examples

#### Example 1: Basic `for await...of`

```js
async function example() {
  const asyncIterable = {
    [Symbol.asyncIterator]() {
      let i = 0;
      return {
        async next() {
          if (i < 3) {
            return { value: i++, done: false };
          }
          return { value: undefined, done: true };
        },
      };
    },
  };

  for await (const value of asyncIterable) {
    console.log(value); // 0, 1, 2
  }
}

example();
```

**Expected Output:**
```
0
1
2
```

**Why This Output:** `for await...of` calls `next()` on the async iterator, awaiting each result.

### Real-World Cases

- **Node.js streams:** `for await (const chunk of readableStream) { ... }`.
- **Database cursors:** Iterating over query results.
- **WebSocket messages:** Processing incoming messages.

---

## Async Generators

### Definitions

**Core Definition:** An async generator is a function declared with `async function*` that returns an async generator object, combining the features of async functions and generator functions.

**Technical Definition:** Per MDN, "The `async function*` declaration creates a binding of a new async generator function to a given name". Each call returns a new `AsyncGenerator` object that conforms to both the async iterable and async iterator protocols. Each call to `next()` returns a Promise that resolves to an iterator result object. Async generator functions combine the features of async functions and generator functions.

**Beginner-Friendly Explanation:** An async generator is a function that can both `await` asynchronous operations and `yield` values one at a time. It's like a generator that can do async work between yields.

### Purposes

- To produce a stream of values asynchronously.
- To combine `await` and `yield` in the same function.
- To implement lazy, asynchronous data sources.
- To create paginated or stream-based data pipelines.

### Syntax Rules and Structure

```js
async function* generatorName() {
  yield await someAsyncOperation();
  yield await anotherAsyncOperation();
}
```

**Component Breakdown:**
- `async function*`: The declaration.
- `yield await promise`: Yields the awaited value.

**Syntax Rules:**
- `await` and `yield` can both be used in the function body.
- Each `next()` call returns a Promise.
- The generator is consumed with `for await...of` or manually with `next()`.

**Constraints and Limitations:**
- No arrow function equivalent for async generators.
- Must be consumed with `for await...of` or manual iteration.

### Annotated Code Examples

#### Example 1: Basic Async Generator

```js
async function* generateSequence() {
  yield await Promise.resolve(1);
  yield await Promise.resolve(2);
  yield await Promise.resolve(3);
}

async function consume() {
  for await (const value of generateSequence()) {
    console.log(value); // 1, 2, 3
  }
}

consume();
```

**Expected Output:**
```
1
2
3
```

**Why This Output:** The async generator yields values one at a time, awaiting each Promise. `for await...of` consumes them in order.

#### Example 2: Paginated Data Generator

```js
async function* fetchAllPages(baseUrl) {
  let page = 1;
  while (true) {
    const response = await fetch(`${baseUrl}?page=${page}`);
    const data = await response.json();
    if (data.length === 0) break;
    yield data;
    page++;
  }
}

async function getAllUsers() {
  const users = [];
  for await (const page of fetchAllPages("/api/users")) {
    users.push(...page);
  }
  return users;
}
```

**Expected Output:** All users are collected across pages.

**Why This Output:** The async generator yields each page of data, and the consumer collects them.

### Real-World Cases

- **Paginated APIs:** Fetching pages lazily as needed.
- **Streams:** Reading chunks from a stream.
- **Event sourcing:** Producing events as they occur.

---

## Async/Await Pitfalls

### Definitions

**Core Definition:** Async/await pitfalls are common mistakes and anti-patterns that lead to bugs, performance issues, or unmaintainable code.

**Technical Definition:** Common pitfalls include: (1) forgetting to `await` an async call (floating promises); (2) using `await` in a loop for independent operations (sequential instead of concurrent); (3) using `forEach` with `async/await` (callbacks are not awaited); (4) not handling errors with `try...catch`; (5) using `async` in constructors; (6) blocking with synchronous code inside async functions.

**Beginner-Friendly Explanation:** Async/await makes asynchronous code look synchronous, but it's easy to forget that `await` is still asynchronous. Common mistakes include forgetting to `await`, using `await` in loops when you could run things in parallel, and assuming `forEach` waits for async callbacks (it doesn't).

### Purposes

- To recognise and avoid common async/await mistakes.
- To write more reliable and performant asynchronous code.
- To improve code readability and maintainability.

### Syntax Rules and Structure

Common pitfalls and their solutions:

```js
// PITFALL 1: Floating promise (missing await)
async function bad() {
  doSomethingAsync(); // Not awaited!
}

// FIX:
async function good() {
  await doSomethingAsync();
}

// PITFALL 2: forEach with async
items.forEach(async (item) => {
  await processItem(item); // Does NOT wait
});

// FIX:
for (const item of items) {
  await processItem(item); // Sequential
}
// or
await Promise.all(items.map((item) => processItem(item))); // Concurrent

// PITFALL 3: Sequential awaits for independent operations
const a = await fetchA(); // Waits
const b = await fetchB(); // Waits

// FIX:
const [a, b] = await Promise.all([fetchA(), fetchB()]);
```

**Component Breakdown:**
- **Floating promise:** Calling an async function without `await` or `.catch()`.
- **forEach trap:** `forEach` does not await async callbacks.
- **Sequential independent ops:** Using `await` sequentially when operations are independent.

**Syntax Rules:**
- Always `await` async calls unless you intentionally want a floating promise.
- Use `for...of` for sequential loops, `Promise.all()` for concurrent.
- Never use `async` in constructors; use factory functions.

**Constraints and Limitations:**
- Some pitfalls are subtle and only manifest under certain conditions.
- Linters (ESLint) can catch many of these issues.

### Annotated Code Examples

#### Example 1: The `forEach` Trap

```js
// BAD: forEach does not await
async function bad() {
  [1, 2, 3].forEach(async (n) => {
    await delay(n * 100);
    console.log(n);
  });
  console.log("Done"); // Logs BEFORE the async callbacks complete
}

// GOOD: for...of awaits sequentially
async function good() {
  for (const n of [1, 2, 3]) {
    await delay(n * 100);
    console.log(n);
  }
  console.log("Done"); // Logs AFTER all iterations
}
```

**Expected Output (bad):**
```
Done
1
2
3
```

**Expected Output (good):**
```
1
2
3
Done
```

**Why This Output:** `forEach` does not await async callbacks, so `"Done"` logs first. `for...of` with `await` waits for each iteration.

#### Example 2: Sequential vs Concurrent

```js
// BAD: Sequential (slower)
async function slow() {
  const a = await fetchA(); // Wait
  const b = await fetchB(); // Wait
  return { a, b };
}

// GOOD: Concurrent (faster)
async function fast() {
  const [a, b] = await Promise.all([fetchA(), fetchB()]);
  return { a, b };
}
```

**Expected Output:** The concurrent version is significantly faster.

**Why This Output:** `Promise.all()` starts both fetches simultaneously, while sequential `await` waits for each one.

### Real-World Cases

- **API clients:** Forgetting to await fetch calls.
- **Data processing:** Using `forEach` with async and getting unexpected results.
- **Performance:** Sequential awaits causing slow page loads.

---

## Async/Await Best Practices

### Definitions

**Core Definition:** Async/await best practices are guidelines for writing clean, maintainable, and performant asynchronous code.

**Technical Definition:** Per the best practices guide, "By adopting these best practices, developers can write more reliable, efficient, and maintainable JavaScript code that effectively manages asynchronous tasks". Key practices include: always handling errors with `try...catch`, using `Promise.all()` for concurrent operations, avoiding sequential awaits for independent tasks, not using `async` in constructors, and preferring `async/await` over raw Promise chains for readability.

**Beginner-Friendly Explanation:** Best practices are the rules of thumb for using async/await correctly. Always handle errors. Use `Promise.all()` for parallel operations. Don't use `await` in a loop if you don't need to. Use `for...of` for sequential iteration. And prefer `async/await` over `.then()` chains for readability.

### Purposes

- To write maintainable and readable asynchronous code.
- To ensure errors are always handled.
- To optimise performance by using concurrency where appropriate.
- To avoid common pitfalls and anti-patterns.

### Syntax Rules and Structure

Best practices in practice:

```js
// GOOD: Error handling + concurrency
async function loadData() {
  try {
    const [user, posts] = await Promise.all([
      fetchUser(),
      fetchPosts(),
    ]);
    return { user, posts };
  } catch (error) {
    console.error("Load failed:", error.message);
    throw error;
  } finally {
    hideLoadingSpinner();
  }
}

// GOOD: Sequential iteration with for...of
async function processSequentially(items) {
  for (const item of items) {
    await processItem(item);
  }
}

// GOOD: Factory function instead of async constructor
class Database {
  static async create(config) {
    const connection = await connect(config);
    return new Database(connection);
  }
}
```

**Component Breakdown:**
- **Error handling:** Always wrap `await` in `try...catch`.
- **Concurrency:** Use `Promise.all()` for independent operations.
- **Sequential iteration:** Use `for...of` for dependent iterations.
- **No async constructors:** Use static factory methods.

**Syntax Rules:**
- Always `await` async calls.
- Use `try...catch` for error handling.
- Use `Promise.all()` for concurrent operations.
- Use `for...of` for sequential loops.
- Never use `async` in constructors.

**Constraints and Limitations:**
- Best practices are guidelines, not strict rules.
- Context matters; sometimes sequential is necessary.

### Annotated Code Examples

#### Example 1: Comprehensive Best Practices

```js
async function fetchUserData(userId) {
  try {
    // Concurrent data loading
    const [user, posts, stats] = await Promise.all([
      fetchUser(userId),
      fetchPosts(userId),
      fetchStats(userId),
    ]);

    // Process sequentially where needed
    const enrichedPosts = [];
    for (const post of posts) {
      const comments = await fetchComments(post.id);
      enrichedPosts.push({ ...post, comments });
    }

    return { user, posts: enrichedPosts, stats };
  } catch (error) {
    console.error("Failed to load user data:", error.message);
    throw error;
  } finally {
    hideLoadingSpinner();
  }
}
```

**Expected Output:** The enriched user data, or an error if any operation fails.

**Why This Output:** The function uses `Promise.all()` for concurrent operations, `for...of` for sequential post enrichment, `try...catch` for error handling, and `finally` for cleanup.

#### Example 2: Avoiding Sequential Awaits

```js
// BAD: Sequential (slow)
async function slow() {
  const user = await fetchUser();
  const posts = await fetchPosts();
  const stats = await fetchStats();
  return { user, posts, stats };
}

// GOOD: Concurrent (fast)
async function fast() {
  const [user, posts, stats] = await Promise.all([
    fetchUser(),
    fetchPosts(),
    fetchStats(),
  ]);
  return { user, posts, stats };
}
```

**Expected Output:** The concurrent version is significantly faster.

**Why This Output:** `Promise.all()` starts all three fetches simultaneously.

### Real-World Cases

- **Node.js applications:** Using `async/await` with error handling and concurrency.
- **React:** Using `async/await` in effects and event handlers.
- **API clients:** Using `Promise.all()` for parallel data fetching.
- **Database operations:** Using factory functions for async initialisation.

---

## References

- MDN Web Docs – async function — https://mdn.org.cn/en-US/docs/Web/JavaScript/Reference/Statements/async_function
- MDN Web Docs – await — https://mdn.org.cn/en-US/docs/Web/JavaScript/Reference/Operators/await
- MDN Web Docs – for await...of — https://ligm.univ-eiffel.fr//~forax/MDN/developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for-await...html
- MDN Web Docs – async function* — https://mdn.org.cn/en-US/docs/Web/JavaScript/Reference/Statements/async_function*
- MDN Web Docs – Promise.allSettled() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/allSettled
- MDN Web Docs – Promise.any() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/any
- MDN Web Docs – Promise.race() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/race
- TC39 – Top-level await proposal — https://www.proposals.es/proposals/Top-level%20await
- Node.js Documentation – ESM Top-level await — https://nodejs.org/api/esm.html
- Async Patterns in Node.js (mcollina/skills) — https://github.com/mcollina/skills/blob/main/skills/node/rules/async-patterns.md
- Understanding JavaScript Async/Await Best Practices — https://github.com/serpuniversity/learn-docs/blob/main/javascript/JavaScript%20Bad%20await.md
- CoreUI – How to handle async/await errors in JavaScript — https://coreui.io/answers/how-to-handle-async-await-errors-in-javascript/
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) — https://ecma-international.org/wp-content/uploads/ECMA-262_16th_edition_june_2025.pdf
- V8 – Top-level await — https://v8.node.org.cn/features/top-level-await/