# Promises — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A Promise is an object that represents the eventual completion (or failure) of an asynchronous operation and its resulting value.

**Technical Definition:** Per MDN, "The `Promise` object represents the eventual completion (or failure) of an asynchronous operation and its resulting value". A Promise acts as a proxy for a value not necessarily known when the promise is created. It allows you to associate handlers with an asynchronous action's eventual success value or failure reason. This lets asynchronous methods return values like synchronous methods: instead of immediately returning the final value, the asynchronous method returns a promise to supply the value at some point in the future. The ECMAScript specification defines Promise objects in §25.4 (ECMA-262), and the Promises/A+ specification provides the interoperability standard.

**Beginner-Friendly Explanation:** A Promise is like a receipt you get when you order something online. It's not the item itself — it's a guarantee that you'll either get the item (fulfilled) or a notification that it couldn't be delivered (rejected). You can attach handlers to say what to do when it arrives, and another to say what to do if it fails. Promises make asynchronous code easier to read and reason about than callbacks.

### Key Characteristics

- **Three states:** A Promise is always in one of three states — pending, fulfilled, or rejected.
- **Immutable settlement:** Once a Promise is settled (fulfilled or rejected), its state cannot change.
- **Chainable:** `.then()`, `.catch()`, and `.finally()` return new Promises, enabling chaining.
- **Asynchronous handlers:** Promise handlers are always called asynchronously (in a microtask).
- **Composable:** Static methods like `Promise.all()`, `Promise.allSettled()`, `Promise.race()`, and `Promise.any()` combine multiple Promises.
- **Error propagation:** Errors propagate down the chain until a `.catch()` handler is found.
- **No cancellation protocol:** Promises have no built-in cancellation, but `AbortController` can be used with some APIs.

### Prerequisites

- Understanding of JavaScript functions, callbacks, and higher-order functions.
- Familiarity with the event loop and asynchronous execution.
- Basic knowledge of callbacks and callback hell.

### Related Programming Areas

- **Asynchronous JavaScript:** `async/await` is built on Promises.
- **Network requests:** The Fetch API returns Promises.
- **Node.js:** Modern file system and stream APIs use Promises.
- **Concurrency:** Promise combinators enable concurrent asynchronous operations.

### Core Concepts / Features

---

## Promise States

### Definitions

**Core Definition:** A Promise is always in one of three mutually exclusive states: pending, fulfilled, or rejected. A Promise is "settled" if it is either fulfilled or rejected.

**Technical Definition:** Per MDN, "A `Promise` is in one of these states: **pending**: initial state, neither fulfilled nor rejected. **fulfilled**: meaning that the operation was completed successfully. **rejected**: meaning that the operation failed. The eventual state of a pending promise can either be fulfilled with a value or rejected with a reason (error)". A promise is said to be settled if it is either fulfilled or rejected, but not pending. The term "resolved" is also used: a promise is resolved when it is settled or "locked-in" to match the eventual state of another promise.

**Beginner-Friendly Explanation:** A Promise is like a package delivery. **Pending** means the package is on its way. **Fulfilled** means it arrived successfully. **Rejected** means it was lost or damaged. Once it arrives or is lost (settled), the state can't change.

### Purposes

- To provide a clear, standardised model for representing asynchronous operation outcomes.
- To enable predictable handling of success and failure paths.
- To support chaining and composition of asynchronous operations.

### Syntax Rules and Structure

The states are not set via syntax; they are the result of the executor function calling `resolve` or `reject`:

```js
const promise = new Promise((resolve, reject) => {
  // pending state
  if (success) {
    resolve(value);   // transitions to fulfilled
  } else {
    reject(error);    // transitions to rejected
  }
});
```

**Component Breakdown:**
- **Pending:** The initial state after `new Promise()` is called.
- **Fulfilled:** Reached when `resolve(value)` is called.
- **Rejected:** Reached when `reject(error)` is called.
- **Settled:** Either fulfilled or rejected.

**Syntax Rules:**
- A Promise can transition from pending to fulfilled or rejected exactly once.
- Once settled, calling `resolve` or `reject` again has no effect.
- Handlers attached after settlement are still called (asynchronously).

**Constraints and Limitations:**
- There is no way to inspect the state of a Promise synchronously (use `.then()` or `.catch()`).
- A Promise cannot be un-settled or reset.

### Annotated Code Examples

#### Example 1: Observing State Transitions

```js
// Step 1: Create a pending promise.
const promise = new Promise((resolve, reject) => {
  console.log("Executor runs immediately (pending state)");
  setTimeout(() => {
    resolve("Success!"); // Transition to fulfilled
  }, 1000);
});

// Step 2: Attach handlers.
promise
  .then((value) => {
    console.log("Fulfilled with:", value);
  })
  .catch((err) => {
    console.log("Rejected with:", err);
  });

console.log("Promise is pending...");
```

**Expected Output:**
```
Executor runs immediately (pending state)
Promise is pending...
Fulfilled with: Success!
```

**Why This Output:** The executor function runs immediately when the Promise is created. The Promise is initially pending. After 1 second, `resolve("Success!")` is called, transitioning the Promise to fulfilled. The `.then()` handler is then invoked with the resolved value.

#### Example 2: Settled Promise Handlers

```js
// Step 1: Create an already-fulfilled promise.
const resolved = Promise.resolve("Already done");

// Step 2: Attach handler after settlement.
resolved.then((value) => {
  console.log("Handler called even though promise was already fulfilled:", value);
});

console.log("This logs first");
```

**Expected Output:**
```
This logs first
Handler called even though promise was already fulfilled: Already done
```

**Why This Output:** Even though the Promise is already fulfilled when `.then()` is attached, the handler is still called — asynchronously, after the current synchronous code finishes.

### Real-World Cases

- **API requests:** `fetch()` returns a pending Promise that fulfills with the response or rejects with a network error.
- **File I/O:** `fs.promises.readFile()` returns a Promise that fulfills with file data.
- **Timers:** A Promise can wrap `setTimeout` to resolve after a delay.

---

## Promise Creation

### Definitions

**Core Definition:** A Promise is created using the `new Promise()` constructor, which takes an executor function with `resolve` and `reject` parameters.

**Technical Definition:** Per MDN, the `Promise()` constructor "creates `Promise` objects. It is primarily used to wrap callback-based APIs that don't already support promises". The executor function is called synchronously by the constructor before `new Promise()` returns. The `resolve` and `reject` callbacks are only available within the scope of the executor function.

**Beginner-Friendly Explanation:** You create a Promise by writing `new Promise((resolve, reject) => { ... })`. Inside the function, you do your asynchronous work. When it succeeds, call `resolve(value)`. When it fails, call `reject(error)`. The Promise is your "receipt" for that operation.

### Purposes

- To wrap callback-based APIs into Promise-based APIs.
- To create custom asynchronous operations that can be chained with `.then()`.
- To provide a standard interface for handling asynchronous results.

### Syntax Rules and Structure

```js
const promise = new Promise((resolve, reject) => {
  // Executor function — runs synchronously
  // Do async work...
  if (success) {
    resolve(value);
  } else {
    reject(error);
  }
});
```

**Component Breakdown:**
- `new Promise(executor)`: The constructor.
- `executor(resolve, reject)`: A function that receives two callbacks.
- `resolve(value)`: Call to transition the Promise to fulfilled.
- `reject(error)`: Call to transition the Promise to rejected.

**Syntax Rules:**
- The executor function runs synchronously and immediately.
- `resolve` and `reject` are functions provided by the Promise implementation.
- Calling `resolve` or `reject` more than once has no effect after the first call.

**Constraints and Limitations:**
- The `resolve` and `reject` callbacks are not accessible outside the executor scope. Use `Promise.withResolvers()` if you need them externally.
- Throwing an error inside the executor rejects the Promise.

### Annotated Code Examples

#### Example 1: Wrapping a Callback API

```js
// Step 1: Define a callback-based function.
function readFileCallback(path, callback) {
  setTimeout(() => {
    if (path) {
      callback(null, `Contents of ${path}`);
    } else {
      callback(new Error("Invalid path"));
    }
  }, 500);
}

// Step 2: Wrap it in a Promise.
function readFilePromise(path) {
  return new Promise((resolve, reject) => {
    readFileCallback(path, (err, data) => {
      if (err) {
        reject(err);
      } else {
        resolve(data);
      }
    });
  });
}

// Step 3: Use the Promise.
readFilePromise("/file.txt")
  .then((data) => console.log("Success:", data))
  .catch((err) => console.error("Error:", err.message));
```

**Expected Output (after 500ms):**
```
Success: Contents of /file.txt
```

**Why This Output:** The executor wraps the callback-based `readFileCallback`. When the callback is invoked with data, `resolve(data)` is called, fulfilling the Promise. The `.then()` handler receives the data.

#### Example 2: Promise Rejection from Executor

```js
const promise = new Promise((resolve, reject) => {
  // Throwing inside the executor rejects the promise.
  throw new Error("Something went wrong");
});

promise.catch((err) => {
  console.log("Caught:", err.message);
});
```

**Expected Output:**
```
Caught: Something went wrong
```

**Why This Output:** If the executor function throws synchronously, the Promise is rejected with that error. The `.catch()` handler captures it.

### Real-World Cases

- **Wrapping `setTimeout`:** `const delay = (ms) => new Promise((r) => setTimeout(r, ms))`.
- **XMLHttpRequest:** Wrapping `xhr` callbacks in a Promise for cleaner API usage.
- **Node.js `util.promisify`:** Converts error-first callbacks into Promise-returning functions.

---

## `new Promise()`

### Definitions

**Core Definition:** `new Promise()` is the constructor syntax used to create a new Promise object.

**Technical Definition:** The `Promise()` constructor is called with `new` and takes a single argument: an executor function. Per MDN, "The `resolve` and `reject` callbacks are only available within the scope of the executor function, which means you can't access them after the promise is constructed". The executor is called synchronously during construction.

**Beginner-Friendly Explanation:** `new Promise()` is how you create a Promise from scratch. You pass it a function that does the actual work and calls either `resolve` or `reject` when done.

### Purposes

- To create a new Promise instance.
- To wrap asynchronous operations that don't already return Promises.
- To provide a custom asynchronous workflow.

### Syntax Rules and Structure

```js
new Promise(executor);
```

**Component Breakdown:**
- `executor`: A function `(resolve, reject) => { ... }`.

**Syntax Rules:**
- `new` is required; calling `Promise()` without `new` throws a `TypeError`.
- The executor is called synchronously.
- The executor receives `resolve` and `reject` functions.

**Constraints and Limitations:**
- Cannot access `resolve`/`reject` outside the executor (unless using `withResolvers`).
- Nesting Promises is an anti-pattern; use `.then()` chaining instead.

### Annotated Code Examples

#### Example 1: Basic Promise Construction

```js
// Step 1: Create a promise that resolves after 1 second.
const wait = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("Done waiting");
  }, 1000);
});

// Step 2: Use the promise.
wait.then((message) => console.log(message));
```

**Expected Output (after 1 second):**
```
Done waiting
```

**Why This Output:** The executor calls `resolve("Done waiting")` after 1000ms. The `.then()` handler logs the resolved value.

### Real-World Cases

- **Custom async utilities:** `wait(ms)`, `timeout(promise, ms)`.
- **Wrapping event listeners:** A Promise that resolves on a specific event.
- **Database transactions:** A Promise that resolves when a transaction commits.

---

## `resolve`

### Definitions

**Core Definition:** `resolve` is a function provided to the Promise executor that transitions the Promise to the fulfilled state with a given value.

**Technical Definition:** Per MDN, "We call `resolve(...)` when what we were doing asynchronously was successful". If the value passed to `resolve` is a thenable (an object with a `.then()` method), the Promise will adopt the state of that thenable. This is called "resolution" — the Promise becomes "locked-in" to match the eventual state of the passed thenable.

**Beginner-Friendly Explanation:** `resolve(value)` is how you tell a Promise "it worked, here's the result." If the value is another Promise, the outer Promise will wait for the inner one and adopt its result.

### Purposes

- To fulfill a Promise with a success value.
- To chain Promises by resolving with another thenable.
- To signal successful completion of an asynchronous operation.

### Syntax Rules and Structure

```js
new Promise((resolve) => {
  resolve(value);
});
```

**Component Breakdown:**
- `resolve`: The fulfillment callback.
- `value`: Any JavaScript value, including another Promise.

**Syntax Rules:**
- Calling `resolve` after the Promise is already settled has no effect.
- If `value` is a thenable, the Promise adopts its state.

**Constraints and Limitations:**
- Resolving with a Promise that never settles leaves the outer Promise pending forever.
- Resolving with `undefined` fulfills the Promise with `undefined`.

### Annotated Code Examples

#### Example 1: Resolving with a Value

```js
const promise = new Promise((resolve) => {
  resolve(42);
});

promise.then((value) => console.log(value)); // 42
```

**Expected Output:**
```
42
```

**Why This Output:** The Promise is fulfilled with the value `42`. The `.then()` handler receives `42`.

#### Example 2: Resolving with Another Promise

```js
const inner = new Promise((resolve) => {
  setTimeout(() => resolve("Inner value"), 1000);
});

const outer = new Promise((resolve) => {
  resolve(inner); // Resolve with a thenable
});

outer.then((value) => console.log(value)); // "Inner value" after 1s
```

**Expected Output (after 1 second):**
```
Inner value
```

**Why This Output:** The outer Promise is resolved with the inner Promise. The outer Promise waits for the inner Promise to settle and adopts its value.

### Real-World Cases

- **Async data fetching:** `resolve(response.json())` — resolving with another Promise.
- **Caching:** Resolving with cached data if available.

---

## `reject`

### Definitions

**Core Definition:** `reject` is a function provided to the Promise executor that transitions the Promise to the rejected state with a given reason (usually an Error).

**Technical Definition:** Per MDN, "We call `reject(...)` when it failed". The rejection reason is typically an `Error` object, but can be any value. If the Promise is already settled, calling `reject` has no effect.

**Beginner-Friendly Explanation:** `reject(error)` is how you tell a Promise "it failed, here's why." The error is passed to the nearest `.catch()` handler.

### Purposes

- To signal failure of an asynchronous operation.
- To propagate errors to `.catch()` handlers.
- To provide a reason for the failure.

### Syntax Rules and Structure

```js
new Promise((resolve, reject) => {
  reject(error);
});
```

**Component Breakdown:**
- `reject`: The rejection callback.
- `error`: The reason for rejection (usually an `Error`).

**Syntax Rules:**
- Calling `reject` after settlement has no effect.
- Throwing inside the executor also rejects the Promise.

**Constraints and Limitations:**
- Rejecting with a non-Error value makes debugging harder.
- Unhandled rejections trigger global events (`unhandledrejection` in browsers, `unhandledRejection` in Node.js).

### Annotated Code Examples

#### Example 1: Rejecting with an Error

```js
const promise = new Promise((resolve, reject) => {
  reject(new Error("Something failed"));
});

promise.catch((err) => console.log(err.message)); // "Something failed"
```

**Expected Output:**
```
Something failed
```

**Why This Output:** The Promise is rejected with an `Error` object. The `.catch()` handler receives the error and logs its message.

#### Example 2: Unhandled Rejection

```js
// This creates an unhandled rejection.
Promise.reject(new Error("Unhandled"));

// Node.js:
process.on("unhandledRejection", (reason) => {
  console.error("Unhandled rejection:", reason.message);
});
```

**Expected Output:**
```
Unhandled rejection: Unhandled
```

**Why This Output:** When a Promise is rejected and no `.catch()` handler is attached, the `unhandledRejection` event is emitted. This is a critical debugging tool.

### Real-World Cases

- **API errors:** Rejecting with an `Error` when a network request fails.
- **Validation:** Rejecting with a `ValidationError` when input is invalid.
- **Timeouts:** Rejecting when an operation exceeds its time limit.

---

## `.then()`

### Definitions

**Core Definition:** `.then()` is a method that attaches fulfillment and/or rejection handlers to a Promise and returns a new Promise, enabling chaining.

**Technical Definition:** Per MDN, "The `.then()` method attaches fulfillment and rejection handlers to the promise and returns a new promise resolving to the return value of the called handler". `.then()` takes two optional arguments: `onFulfilled` and `onRejected`. If `onFulfilled` returns a value, the new Promise is fulfilled with that value. If it returns a Promise, the new Promise adopts that Promise's state.

**Beginner-Friendly Explanation:** `.then()` is how you say "when this Promise succeeds, do this." You can chain multiple `.then()` calls to process a value through a series of steps.

### Purposes

- To attach success handlers to a Promise.
- To transform the resolved value.
- To chain asynchronous operations sequentially.
- To handle errors (via the second argument, though `.catch()` is preferred).

### Syntax Rules and Structure

```js
promise.then(onFulfilled);
promise.then(onFulfilled, onRejected);
```

**Component Breakdown:**
- `onFulfilled(value)`: Called when the Promise fulfills.
- `onRejected(reason)`: Called when the Promise rejects (optional).

**Syntax Rules:**
- `.then()` always returns a new Promise, enabling chaining.
- If `onFulfilled` returns a value, the new Promise is fulfilled with it.
- If `onFulfilled` throws, the new Promise is rejected with the thrown error.
- If `onFulfilled` returns a Promise, the new Promise adopts its state.

**Constraints and Limitations:**
- Not returning a value from `.then()` causes the next `.then()` to receive `undefined`.
- Using the second argument to `.then()` for errors is discouraged; use `.catch()` instead.

### Annotated Code Examples

#### Example 1: Basic `.then()` Usage

```js
Promise.resolve(5)
  .then((value) => {
    console.log("Value:", value);
    return value * 2;
  })
  .then((doubled) => {
    console.log("Doubled:", doubled);
  });
```

**Expected Output:**
```
Value: 5
Doubled: 10
```

**Why This Output:** The first `.then()` receives `5`, logs it, and returns `10`. The second `.then()` receives `10` and logs it.

#### Example 2: Chaining Promises

```js
fetch("/api/user")
  .then((response) => response.json())
  .then((user) => fetch(`/api/posts/${user.id}`))
  .then((response) => response.json())
  .then((posts) => console.log(posts));
```

**Expected Output:** The posts are logged after all three asynchronous operations complete.

**Why This Output:** Each `.then()` returns a new Promise. The chain waits for each operation to complete before starting the next.

### Real-World Cases

- **Fetch API:** `fetch(url).then(res => res.json())`.
- **Sequential API calls:** Chaining user → posts → comments.
- **Data transformation:** Mapping, filtering, and formatting results.

---

## `.catch()`

### Definitions

**Core Definition:** `.catch()` is a method that attaches a rejection handler to a Promise, returning a new Promise.

**Technical Definition:** Per MDN, `.catch()` is equivalent to `.then(undefined, onRejected)`. It handles rejected Promises and errors thrown in preceding `.then()` handlers. Per OWASP, "If an error occurs in a Promise class, it skips over the `.then` functions and invokes the first `.catch` function it finds".

**Beginner-Friendly Explanation:** `.catch()` is how you handle errors in a Promise chain. If anything goes wrong at any step, the error "bubbles up" to the nearest `.catch()`.

### Purposes

- To centralise error handling in a Promise chain.
- To recover from errors by returning a fallback value.
- To log errors for debugging.

### Syntax Rules and Structure

```js
promise.catch(onRejected);
```

**Component Breakdown:**
- `onRejected(reason)`: Called when the Promise rejects.

**Syntax Rules:**
- `.catch()` returns a new Promise, so chaining can continue.
- If `onRejected` returns a value, the new Promise is fulfilled with that value.
- If `onRejected` throws, the new Promise is rejected.

**Constraints and Limitations:**
- `.catch()` only handles errors from preceding handlers, not subsequent ones.
- Unhandled rejections occur if no `.catch()` is attached.

### Annotated Code Examples

#### Example 1: Basic Error Handling

```js
Promise.reject(new Error("Network error"))
  .catch((err) => {
    console.error("Caught:", err.message);
    return "Fallback value"; // Recover from error
  })
  .then((value) => {
    console.log("After catch:", value);
  });
```

**Expected Output:**
```
Caught: Network error
After catch: Fallback value
```

**Why This Output:** The `.catch()` handler receives the error, logs it, and returns `"Fallback value"`. The next `.then()` receives that value, showing that the chain recovered from the error.

#### Example 2: Error Propagation

```js
Promise.resolve("Start")
  .then((value) => {
    throw new Error("Something went wrong");
  })
  .then((value) => {
    console.log("This is skipped");
  })
  .catch((err) => {
    console.log("Caught:", err.message);
  });
```

**Expected Output:**
```
Caught: Something went wrong
```

**Why This Output:** The error thrown in the first `.then()` skips the second `.then()` and is caught by the `.catch()`.

### Real-World Cases

- **API error handling:** Catching network errors and showing user-friendly messages.
- **Fallback data:** Returning cached data when a request fails.
- **Logging:** Logging errors to a monitoring service.

---

## `.finally()`

### Definitions

**Core Definition:** `.finally()` is a method that attaches a handler that runs regardless of whether the Promise is fulfilled or rejected.

**Technical Definition:** Per MDN, `.finally()` "allows you to register a callback function that will be called when the Promise is settled (i.e., either fulfilled or rejected)". It does not receive the fulfillment value or rejection reason, and it does not modify the Promise's settled state (unless it throws).

**Beginner-Friendly Explanation:** `.finally()` is like a cleanup crew — it runs no matter what happened. Use it to hide loading spinners, close connections, or perform any cleanup that should always happen.

### Purposes

- To perform cleanup operations regardless of outcome.
- To hide loading indicators after an operation completes.
- To release resources (connections, file handles).

### Syntax Rules and Structure

```js
promise.finally(onFinally);
```

**Component Breakdown:**
- `onFinally()`: A function that runs when the Promise settles. Receives no arguments.

**Syntax Rules:**
- `.finally()` passes through the fulfillment value or rejection reason unchanged.
- If `onFinally` throws, the returned Promise is rejected with that error.
- `.finally()` returns a new Promise.

**Constraints and Limitations:**
- `.finally()` does not receive the result or error, so it cannot distinguish between success and failure.
- It is not a replacement for `.then()` or `.catch()`.

### Annotated Code Examples

#### Example 1: Basic `.finally()` Usage

```js
Promise.resolve("Data loaded")
  .then((value) => console.log(value))
  .catch((err) => console.error(err))
  .finally(() => console.log("Loading complete"));
```

**Expected Output:**
```
Data loaded
Loading complete
```

**Why This Output:** The `.finally()` handler runs after the `.then()` handler, logging "Loading complete" regardless of the outcome.

#### Example 2: Cleanup with `.finally()`

```js
function fetchData() {
  showLoadingSpinner();
  return fetch("/api/data")
    .then((response) => response.json())
    .finally(() => hideLoadingSpinner());
}
```

**Expected Output:** The loading spinner is hidden whether the fetch succeeds or fails.

**Why This Output:** `.finally()` ensures the spinner is hidden after the operation completes, regardless of whether the fetch resolved or rejected.

### Real-World Cases

- **UI state:** Hiding loading spinners or progress bars.
- **Resource cleanup:** Closing database connections or file handles.
- **Timing:** Logging the duration of an operation.

---

## Promise Chaining

### Definitions

**Core Definition:** Promise chaining is the practice of linking multiple `.then()`, `.catch()`, and `.finally()` calls together, where each returns a new Promise, enabling sequential asynchronous operations.

**Technical Definition:** Per MDN, "A common need is to execute two or more asynchronous operations back to back, where each subsequent operation starts when the previous operation succeeds, with the result from the previous step". Promise chaining achieves this by attaching handlers to the Promise returned by each `.then()` call. The key is that `.then()` returns a new Promise.

**Beginner-Friendly Explanation:** Promise chaining is like an assembly line for asynchronous operations. Each `.then()` takes the result of the previous step, does something with it, and passes the result to the next step. This avoids the deep nesting of callback hell.

### Purposes

- To execute multiple asynchronous operations in sequence.
- To avoid callback hell (pyramid of doom).
- To enable clean, readable asynchronous workflows.
- To centralise error handling with a single `.catch()`.

### Syntax Rules and Structure

```js
promise
  .then(step1)
  .then(step2)
  .then(step3)
  .catch(handleError)
  .finally(cleanup);
```

**Component Breakdown:**
- Each `.then()` receives the result of the previous step.
- The chain is flat, not nested.
- A single `.catch()` at the end handles all errors.

**Syntax Rules:**
- Always return a value or Promise from `.then()` handlers to pass data forward.
- Errors propagate down the chain until a `.catch()` is found.
- Keep chains flat; avoid nesting Promises inside `.then()`.

**Constraints and Limitations:**
- Forgetting to return a value breaks the chain (the next `.then()` receives `undefined`).
- Chains can become long and difficult to debug if not well-structured.

### Annotated Code Examples

#### Example 1: Sequential API Calls

```js
fetch("/api/user")
  .then((response) => response.json())
  .then((user) => {
    console.log("User:", user.name);
    return fetch(`/api/posts/${user.id}`);
  })
  .then((response) => response.json())
  .then((posts) => {
    console.log("Posts:", posts.length);
  })
  .catch((err) => {
    console.error("Error:", err.message);
  });
```

**Expected Output:**
```
User: Alice
Posts: 5
```

**Why This Output:** Each `.then()` waits for the previous operation to complete. The user data is fetched, logged, and then used to fetch posts. Errors are caught by a single `.catch()`.

#### Example 2: Flat vs Nested Chains

```js
// GOOD: Flat chain
getUser()
  .then((user) => getPosts(user.id))
  .then((posts) => getComments(posts[0].id))
  .then((comments) => console.log(comments))
  .catch((err) => console.error(err));

// BAD: Nested promises
getUser().then((user) => {
  getPosts(user.id).then((posts) => {
    getComments(posts[0].id).then((comments) => {
      console.log(comments);
    });
  });
});
```

**Expected Output:** Both produce the same result, but the flat chain is more readable and maintainable.

**Why This Output:** The flat chain avoids nesting and centralises error handling, while the nested version reintroduces callback hell.

### Real-World Cases

- **User onboarding:** Fetch user → create profile → send welcome email.
- **E-commerce checkout:** Validate cart → process payment → confirm order.
- **Data pipelines:** Fetch → parse → transform → store.

---

## Promise Composition

### Definitions

**Core Definition:** Promise composition is the practice of combining multiple Promises into a single Promise using static methods like `Promise.all()`, `Promise.allSettled()`, `Promise.race()`, and `Promise.any()`.

**Technical Definition:** Promise composition methods are static methods on the `Promise` constructor that take an iterable of Promises and return a single Promise. Each method has different semantics for how it combines the results: `all()` waits for all to fulfill, `allSettled()` waits for all to settle, `race()` settles with the first to settle, and `any()` fulfills with the first to fulfill.

**Beginner-Friendly Explanation:** Promise composition is like coordinating a team of workers. `Promise.all()` waits for everyone to finish and gives you all their results. `Promise.race()` gives you the result of whoever finishes first. `Promise.any()` gives you the first successful result. `Promise.allSettled()` waits for everyone and tells you how each one did.

### Purposes

- To run multiple asynchronous operations concurrently.
- To coordinate results from multiple sources.
- To implement timeouts and fallbacks.
- To handle heterogeneous collections of asynchronous tasks.

### Syntax Rules and Structure

```js
Promise.all(iterable);
Promise.allSettled(iterable);
Promise.race(iterable);
Promise.any(iterable);
```

**Component Breakdown:**
- `iterable`: An array (or other iterable) of Promises or values.

**Syntax Rules:**
- All methods return a new Promise.
- Non-Promise values in the iterable are treated as already-fulfilled Promises.
- Empty iterables have different behaviours per method.

**Constraints and Limitations:**
- `Promise.all()` rejects immediately if any input rejects.
- `Promise.any()` rejects only if all inputs reject, with an `AggregateError`.
- `Promise.race()` settles with the first settled input, regardless of outcome.

### Annotated Code Examples

#### Example 1: Promise.all() — Wait for All

```js
const p1 = Promise.resolve("A");
const p2 = new Promise((r) => setTimeout(() => r("B"), 100));
const p3 = Promise.resolve("C");

Promise.all([p1, p2, p3]).then((values) => {
  console.log(values); // ["A", "B", "C"]
});
```

**Expected Output:**
```
[ 'A', 'B', 'C' ]
```

**Why This Output:** `Promise.all()` waits for all Promises to fulfill and returns an array of their values in the original order.

#### Example 2: Promise.allSettled() — Wait for All Settled

```js
const p1 = Promise.resolve("OK");
const p2 = Promise.reject(new Error("Fail"));

Promise.allSettled([p1, p2]).then((results) => {
  console.log(results);
  // [
  //   { status: 'fulfilled', value: 'OK' },
  //   { status: 'rejected', reason: Error: Fail }
  // ]
});
```

**Expected Output:**
```
[
  { status: 'fulfilled', value: 'OK' },
  { status: 'rejected', reason: Error: Fail }
]
```

**Why This Output:** `Promise.allSettled()` waits for all Promises to settle and returns an array of status objects.

#### Example 3: Promise.race() — First to Settle

```js
const fast = new Promise((r) => setTimeout(() => r("Fast"), 100));
const slow = new Promise((r) => setTimeout(() => r("Slow"), 500));

Promise.race([fast, slow]).then((value) => {
  console.log(value); // "Fast"
});
```

**Expected Output (after 100ms):**
```
Fast
```

**Why This Output:** `Promise.race()` settles with the first Promise to settle (fulfill or reject).

#### Example 4: Promise.any() — First to Fulfill

```js
const fail = new Promise((_, r) => setTimeout(() => r("Fail"), 100));
const success = new Promise((r) => setTimeout(() => r("Success"), 500));

Promise.any([fail, success]).then((value) => {
  console.log(value); // "Success"
});
```

**Expected Output (after 500ms):**
```
Success
```

**Why This Output:** `Promise.any()` ignores rejections and fulfills with the first Promise that fulfills.

### Real-World Cases

- **`Promise.all()`:** Loading multiple API endpoints that are all required for a page.
- **`Promise.allSettled()`:** Running batch operations where you want to know the result of each.
- **`Promise.race()`:** Implementing timeouts (`Promise.race([fetch(url), timeout(5000)])`).
- **`Promise.any()`:** Fetching from multiple mirrors and using the fastest successful response.

---

## `Promise.all()`

### Definitions

**Core Definition:** `Promise.all()` takes an iterable of Promises and returns a single Promise that fulfills when all input Promises fulfill, with an array of their fulfillment values.

**Technical Definition:** Per MDN, "The `Promise.all()` static method takes an iterable of promises as input and returns a single Promise. This returned promise fulfills when all of the input's promises fulfill (including when an empty iterable is passed), with an array of the fulfillment values. It rejects when any of the input's promises rejects, with this first rejection reason".

**Beginner-Friendly Explanation:** `Promise.all()` is like waiting for all your friends to arrive before starting the party. If everyone shows up, you get an array of all their gifts. If one person cancels, the party is called off immediately.

### Purposes

- To run multiple independent asynchronous operations concurrently.
- To wait for all operations to complete before proceeding.
- To collect results from multiple sources into a single array.

### Syntax Rules and Structure

```js
Promise.all(iterable);
```

**Component Breakdown:**
- `iterable`: An array of Promises (or values).

**Syntax Rules:**
- Returns a Promise that fulfills with an array of values.
- The order of results matches the order of the input iterable.
- Rejects immediately if any input rejects.

**Constraints and Limitations:**
- A single rejection causes the entire `Promise.all()` to reject.
- Cannot handle partial results; if one fails, all results are lost.

### Annotated Code Examples

#### Example 1: All Fulfilled

```js
Promise.all([
  fetch("/api/users").then((r) => r.json()),
  fetch("/api/posts").then((r) => r.json()),
]).then(([users, posts]) => {
  console.log("Users:", users.length);
  console.log("Posts:", posts.length);
});
```

**Expected Output:** The user and post counts are logged after both fetches complete.

**Why This Output:** Both fetches run concurrently. `Promise.all()` waits for both to resolve and provides the results in order.

#### Example 2: One Rejected

```js
Promise.all([
  Promise.resolve("OK"),
  Promise.reject(new Error("Failed")),
]).catch((err) => {
  console.log("Caught:", err.message); // "Failed"
});
```

**Expected Output:**
```
Caught: Failed
```

**Why This Output:** The rejection of any input causes `Promise.all()` to reject immediately with that error.

### Real-World Cases

- **Dashboard loading:** Fetching multiple widgets concurrently.
- **Batch operations:** Processing multiple items in parallel.
- **Initialisation:** Waiting for multiple resources to load before rendering.

---

## `Promise.allSettled()`

### Definitions

**Core Definition:** `Promise.allSettled()` takes an iterable of Promises and returns a single Promise that fulfills when all input Promises settle (fulfill or reject), with an array of status objects.

**Technical Definition:** Per MDN, "The `Promise.allSettled()` static method takes an iterable of promises as input and returns a single Promise. This returned promise fulfills when all of the input's promises settle (including when an empty iterable is passed), with an array of objects that describe the outcome of each promise". Each result object has a `status` property (`"fulfilled"` or `"rejected"`) and either a `value` or `reason` property.

**Beginner-Friendly Explanation:** `Promise.allSettled()` is like waiting for all your friends to either arrive or cancel. You get a report on each one — who showed up and who didn't — without the party being called off if someone cancels.

### Purposes

- To run multiple independent operations and get the result of each.
- To handle partial failures without losing successful results.
- To implement batch operations where you want to know which items succeeded and which failed.

### Syntax Rules and Structure

```js
Promise.allSettled(iterable);
```

**Component Breakdown:**
- `iterable`: An array of Promises (or values).

**Syntax Rules:**
- Returns a Promise that fulfills with an array of result objects.
- Never rejects (unless the iterable itself is invalid).
- Each result has `{ status, value }` or `{ status, reason }`.

**Constraints and Limitations:**
- Cannot be short-circuited; always waits for all Promises to settle.
- Does not provide a single aggregated error.

### Annotated Code Examples

#### Example 1: Mixed Results

```js
const promises = [
  Promise.resolve("Success 1"),
  Promise.reject(new Error("Failure 1")),
  Promise.resolve("Success 2"),
];

Promise.allSettled(promises).then((results) => {
  results.forEach((result) => {
    if (result.status === "fulfilled") {
      console.log("✓", result.value);
    } else {
      console.log("✗", result.reason.message);
    }
  });
});
```

**Expected Output:**
```
✓ Success 1
✗ Failure 1
✓ Success 2
```

**Why This Output:** `Promise.allSettled()` waits for all Promises and returns status objects for each. Successful results have a `value`; rejected ones have a `reason`.

#### Example 2: Empty Iterable

```js
Promise.allSettled([]).then((results) => {
  console.log(results); // []
});
```

**Expected Output:**
```
[]
```

**Why This Output:** An empty iterable returns a Promise that fulfills immediately with an empty array.

### Real-World Cases

- **Batch API calls:** Sending notifications to multiple users and logging which succeeded.
- **Data migration:** Migrating records and collecting per-record status.
- **Health checks:** Pinging multiple services and reporting their availability.

---

## `Promise.race()`

### Definitions

**Core Definition:** `Promise.race()` takes an iterable of Promises and returns a single Promise that settles with the eventual state of the first Promise to settle.

**Technical Definition:** Per MDN, "The `Promise.race()` static method takes an iterable of promises as input and returns a single Promise. This returned promise settles with the eventual state of the first promise that settles". If the first Promise to settle fulfills, the returned Promise fulfills with its value. If it rejects, the returned Promise rejects with its reason.

**Beginner-Friendly Explanation:** `Promise.race()` is like a race — whichever Promise finishes first (whether it succeeds or fails) determines the outcome. It's useful for timeouts and competitive operations.

### Purposes

- To implement timeouts for asynchronous operations.
- To get the result of the fastest operation.
- To cancel or abandon slow operations when a faster one completes.

### Syntax Rules and Structure

```js
Promise.race(iterable);
```

**Component Breakdown:**
- `iterable`: An array of Promises (or values).

**Syntax Rules:**
- Returns a Promise that settles with the first settled input.
- Non-Promise values are treated as already fulfilled.
- Empty iterable returns a forever-pending Promise.

**Constraints and Limitations:**
- The first rejection causes the returned Promise to reject.
- Does not cancel the other Promises; they continue executing.

### Annotated Code Examples

#### Example 1: Timeout Pattern

```js
function timeout(ms) {
  return new Promise((_, reject) => {
    setTimeout(() => reject(new Error("Timeout")), ms);
  });
}

Promise.race([
  fetch("/api/data"),
  timeout(5000),
]).then((response) => {
  console.log("Data received");
}).catch((err) => {
  console.log("Error:", err.message); // "Timeout" if fetch takes >5s
});
```

**Expected Output:** "Data received" if the fetch completes within 5 seconds; "Error: Timeout" otherwise.

**Why This Output:** `Promise.race()` settles with whichever completes first — the fetch or the timeout.

#### Example 2: First to Fulfill or Reject

```js
const p1 = new Promise((r) => setTimeout(() => r("Fast success"), 100));
const p2 = new Promise((_, r) => setTimeout(() => r(new Error("Fast failure")), 50));

Promise.race([p1, p2]).catch((err) => {
  console.log("Caught:", err.message); // "Fast failure" (p2 settles first)
});
```

**Expected Output:**
```
Caught: Fast failure
```

**Why This Output:** `p2` settles first (after 50ms) with a rejection. `Promise.race()` adopts that rejection.

### Real-World Cases

- **Timeouts:** Abandoning operations that take too long.
- **Mirror racing:** Using the fastest response from multiple servers.
- **Loading indicators:** Showing a spinner only if the operation takes longer than a threshold.

---

## `Promise.any()`

### Definitions

**Core Definition:** `Promise.any()` takes an iterable of Promises and returns a single Promise that fulfills with the first fulfillment value. If all Promises reject, it rejects with an `AggregateError`.

**Technical Definition:** Per MDN, "The `Promise.any()` static method takes an iterable of promises as input and returns a single Promise. This returned promise fulfills when any of the input's promises fulfills, with this first fulfillment value. It rejects when all of the input's promises reject (including when an empty iterable is passed), with an `AggregateError` containing an array of rejection reasons".

**Beginner-Friendly Explanation:** `Promise.any()` is like asking several friends for a favour. You only need one to say yes. You ignore the "no" responses and wait for the first "yes." If everyone says no, you get a list of all the reasons why.

### Purposes

- To get the first successful result from multiple sources.
- To implement fallback mechanisms (try multiple sources until one succeeds).
- To ignore failures and only care about success.

### Syntax Rules and Structure

```js
Promise.any(iterable);
```

**Component Breakdown:**
- `iterable`: An array of Promises (or values).

**Syntax Rules:**
- Fulfills with the first fulfillment value.
- Rejects only if all input Promises reject, with an `AggregateError`.
- Empty iterable rejects immediately.

**Constraints and Limitations:**
- Does not cancel the other Promises when one fulfills.
- The `AggregateError` contains all rejection reasons in order.

### Annotated Code Examples

#### Example 1: First Successful Result

```js
const p1 = Promise.reject(new Error("Server 1 down"));
const p2 = new Promise((r) => setTimeout(() => r("Server 2 OK"), 200));
const p3 = Promise.reject(new Error("Server 3 down"));

Promise.any([p1, p2, p3]).then((value) => {
  console.log("Success:", value); // "Server 2 OK"
});
```

**Expected Output:**
```
Success: Server 2 OK
```

**Why This Output:** `Promise.any()` ignores the rejections from `p1` and `p3` and fulfills with the first successful result from `p2`.

#### Example 2: All Rejected

```js
const p1 = Promise.reject(new Error("Fail 1"));
const p2 = Promise.reject(new Error("Fail 2"));

Promise.any([p1, p2]).catch((err) => {
  console.log(err.name);           // "AggregateError"
  console.log(err.errors.length);  // 2
  console.log(err.errors[0].message); // "Fail 1"
});
```

**Expected Output:**
```
AggregateError
2
Fail 1
```

**Why This Output:** When all Promises reject, `Promise.any()` rejects with an `AggregateError` containing all rejection reasons.

### Real-World Cases

- **CDN fallback:** Trying multiple CDNs and using the first successful response.
- **DNS resolution:** Querying multiple DNS servers.
- **Redundant APIs:** Fetching from multiple endpoints and using the fastest success.

---

## `Promise.resolve()`

### Definitions

**Core Definition:** `Promise.resolve()` returns a Promise that is resolved with a given value.

**Technical Definition:** Per MDN, `Promise.resolve(value)` returns a Promise that is resolved with the given value. If the value is a Promise, it returns that Promise. If it is a thenable, the returned Promise adopts its state. If it is a non-thenable value, the returned Promise is fulfilled with that value.

**Beginner-Friendly Explanation:** `Promise.resolve()` is a shortcut for creating a Promise that's already fulfilled with a value. It's useful for converting a plain value into a Promise so it can be chained with `.then()`.

### Purposes

- To create an already-fulfilled Promise.
- To convert a non-Promise value into a Promise.
- To normalise thenables into Promises.

### Syntax Rules and Structure

```js
Promise.resolve(value);
```

**Component Breakdown:**
- `value`: Any JavaScript value.

**Syntax Rules:**
- If `value` is a Promise, it is returned as-is.
- If `value` is a thenable, the returned Promise adopts its state.
- If `value` is any other value, the returned Promise is fulfilled with `value`.

**Constraints and Limitations:**
- Does not create a new Promise if `value` is already a Promise.
- The resolved Promise's state cannot be observed synchronously.

### Annotated Code Examples

#### Example 1: Creating a Fulfilled Promise

```js
const promise = Promise.resolve(42);
promise.then((value) => console.log(value)); // 42
```

**Expected Output:**
```
42
```

**Why This Output:** `Promise.resolve(42)` creates a Promise that is already fulfilled with `42`.

#### Example 2: Converting a Value

```js
// Synchronous value:
function getValue() {
  return 42;
}

// Promise-wrapped value:
function getValueAsync() {
  return Promise.resolve(42);
}

getValueAsync().then((value) => console.log(value)); // 42
```

**Expected Output:**
```
42
```

**Why This Output:** `Promise.resolve` converts the synchronous value into a Promise, allowing it to be used in Promise chains.

### Real-World Cases

- **Utility functions:** Converting values for Promise chains.
- **Testing:** Creating pre-resolved Promises for test fixtures.
- **Thenable normalisation:** Converting libraries' thenables into native Promises.

---

## `Promise.reject()`

### Definitions

**Core Definition:** `Promise.reject()` returns a Promise that is rejected with a given reason.

**Technical Definition:** Per MDN, `Promise.reject(reason)` returns a Promise that is rejected with the given reason. The reason is typically an `Error` object, but can be any value.

**Beginner-Friendly Explanation:** `Promise.reject()` is a shortcut for creating a Promise that's already rejected with an error. It's useful for testing error handling and for returning rejected Promises from functions.

### Purposes

- To create an already-rejected Promise.
- To test error handling in Promise chains.
- To normalise error conditions into Promises.

### Syntax Rules and Structure

```js
Promise.reject(reason);
```

**Component Breakdown:**
- `reason`: The rejection reason (usually an `Error`).

**Syntax Rules:**
- Always rejects with the given reason.
- The returned Promise is already settled.

**Constraints and Limitations:**
- Unhandled rejections trigger global events.
- The reason is not automatically wrapped in an `Error`.

### Annotated Code Examples

#### Example 1: Creating a Rejected Promise

```js
const promise = Promise.reject(new Error("Something failed"));
promise.catch((err) => console.log(err.message)); // "Something failed"
```

**Expected Output:**
```
Something failed
```

**Why This Output:** The Promise is rejected with the `Error` object. The `.catch()` handler receives it.

#### Example 2: Testing Error Handling

```js
function mayFail(shouldFail) {
  if (shouldFail) {
    return Promise.reject(new Error("Failed as requested"));
  }
  return Promise.resolve("Success");
}

mayFail(true).catch((err) => console.log("Caught:", err.message));
mayFail(false).then((value) => console.log("Got:", value));
```

**Expected Output:**
```
Caught: Failed as requested
Got: Success
```

**Why This Output:** `Promise.reject()` creates a rejected Promise that triggers the `.catch()` handler.

### Real-World Cases

- **Testing:** Creating rejected Promises for unit tests.
- **Validation:** Returning rejected Promises from validation functions.
- **Error propagation:** Propagating errors in Promise chains.

---

## `Promise.withResolvers()`

### Definitions

**Core Definition:** `Promise.withResolvers()` is a static method that returns an object containing a new Promise and the `resolve` and `reject` functions for that Promise.

**Technical Definition:** Per MDN, "The `Promise.withResolvers()` static method returns an object containing a new Promise object and two functions to resolve or reject it, corresponding to the two parameters passed to the executor of the `Promise()` constructor". It is equivalent to the following code but more concise and without needing `let`:

```js
let resolve, reject;
const promise = new Promise((res, rej) => {
  resolve = res;
  reject = rej;
});
```

**Beginner-Friendly Explanation:** `Promise.withResolvers()` gives you a Promise and the functions to resolve or reject it, all in one go. This is useful when you need to decide later how to settle the Promise, such as when handling events or streams.

### Purposes

- To create a Promise and gain access to its `resolve` and `reject` functions outside the executor.
- To handle cases where the resolution logic is external to the Promise creation.
- To simplify patterns involving streams, queues, and periodic events.

### Syntax Rules and Structure

```js
const { promise, resolve, reject } = Promise.withResolvers();
```

**Component Breakdown:**
- `promise`: The new Promise.
- `resolve`: The function to fulfill the Promise.
- `reject`: The function to reject the Promise.

**Syntax Rules:**
- Returns a plain object with `promise`, `resolve`, and `reject` properties.
- Generic and supports subclassing.

**Constraints and Limitations:**
- Baseline 2024 — not available in older environments.
- The Promise is pending until `resolve` or `reject` is called.

### Annotated Code Examples

#### Example 1: Basic Usage

```js
// Step 1: Create a promise with exposed resolvers.
const { promise, resolve, reject } = Promise.withResolvers();

// Step 2: Resolve it later.
setTimeout(() => resolve("Done!"), 1000);

// Step 3: Use the promise.
promise.then((value) => console.log(value)); // "Done!" after 1s
```

**Expected Output (after 1 second):**
```
Done!
```

**Why This Output:** `withResolvers()` creates a Promise and exposes `resolve` and `reject`. The `resolve` function is called later, fulfilling the Promise.

#### Example 2: Event-Driven Resolution

```js
const { promise, resolve } = Promise.withResolvers();

// Resolve the promise when a button is clicked.
document.getElementById("btn").addEventListener("click", () => {
  resolve("Button clicked!");
});

promise.then((value) => console.log(value));
```

**Expected Output (on click):**
```
Button clicked!
```

**Why This Output:** The Promise's resolution is tied to a DOM event. `withResolvers()` provides the `resolve` function that the event listener can call.

### Real-World Cases

- **Streams and queues:** Resolving Promises from external events.
- **Event listeners:** Converting events into Promises.
- **Testing:** Creating controllable Promises for async test scenarios.

---

## Promise Pitfalls

### Definitions

**Core Definition:** Promise pitfalls are common mistakes and anti-patterns that lead to bugs, performance issues, or unmaintainable code when working with Promises.

**Technical Definition:** Common pitfalls include: (1) forgetting to return from `.then()` handlers, breaking the chain; (2) nesting Promises instead of chaining them; (3) not handling rejections, causing unhandled rejection warnings; (4) using callbacks inside `.then()` handlers; (5) assuming Promise handlers run synchronously; (6) creating Promises unnecessarily (explicit construction anti-pattern).

**Beginner-Friendly Explanation:** Promises are powerful but easy to misuse. Common mistakes include forgetting to return a value from `.then()`, nesting `.then()` calls instead of chaining, and ignoring errors. Understanding these pitfalls helps you write cleaner, more reliable async code.

### Purposes

- To recognise and avoid common Promise mistakes.
- To write more robust asynchronous code.
- To improve code readability and maintainability.
- To prevent unhandled rejections and silent failures.

### Syntax Rules and Structure

Common pitfalls and their solutions:

```js
// PITFALL 1: Forgetting to return
fetch("/api").then((res) => {
  res.json(); // Missing return!
}).then((data) => {
  console.log(data); // undefined
});

// FIX:
fetch("/api").then((res) => {
  return res.json();
}).then((data) => {
  console.log(data); // Correct data
});

// PITFALL 2: Nesting instead of chaining
getUser().then((user) => {
  getPosts(user.id).then((posts) => { // Nested!
    console.log(posts);
  });
});

// FIX:
getUser()
  .then((user) => getPosts(user.id))
  .then((posts) => console.log(posts));
```

**Component Breakdown:**
- **Missing return:** Breaks the chain; next `.then()` receives `undefined`.
- **Nesting:** Reintroduces callback hell.
- **Unhandled rejections:** No `.catch()` attached.
- **Explicit construction:** Wrapping an existing Promise in `new Promise()` unnecessarily.

**Syntax Rules:**
- Always return from `.then()` handlers.
- Use flat chains, not nested Promises.
- Always attach a `.catch()` or use `async/await` with `try...catch`.
- Avoid the explicit Promise construction anti-pattern; use `Promise.resolve()` or return the existing Promise.

**Constraints and Limitations:**
- Some pitfalls are subtle and only manifest under certain conditions.
- Linters and code reviews help catch these issues.

### Annotated Code Examples

#### Example 1: Missing Return

```js
// BAD:
Promise.resolve("value")
  .then((value) => {
    Promise.resolve(value.toUpperCase()); // No return!
  })
  .then((result) => {
    console.log(result); // undefined
  });

// GOOD:
Promise.resolve("value")
  .then((value) => {
    return Promise.resolve(value.toUpperCase()); // Return!
  })
  .then((result) => {
    console.log(result); // "VALUE"
  });
```

**Expected Output:**
```
undefined
VALUE
```

**Why This Output:** Without `return`, the first `.then()` returns `undefined`, so the second `.then()` receives `undefined`. With `return`, the value is passed through.

#### Example 2: Unhandled Rejection

```js
// BAD: No catch handler.
Promise.reject(new Error("Oops"));

// GOOD:
Promise.reject(new Error("Oops")).catch((err) => {
  console.error("Handled:", err.message);
});
```

**Expected Output:**
```
Handled: Oops
```

**Why This Output:** Without a `.catch()`, the rejection is unhandled and triggers a global warning. With `.catch()`, the error is handled.

### Real-World Cases

- **Code reviews:** Checking for missing returns and nested Promises.
- **Linting:** Using ESLint rules like `promise/catch-or-return` and `promise/no-nesting`.
- **Debugging:** Identifying unhandled rejections with global handlers.

---

## Promise Best Practices

### Definitions

**Core Definition:** Promise best practices are guidelines for writing clean, maintainable, and reliable asynchronous code using Promises.

**Technical Definition:** Per MDN and the OWASP Node.js Security Cheat Sheet, best practices include: (1) always return Promises from `.then()` callbacks, even if the Promise always resolves to `undefined`; (2) use flat Promise chains instead of nested pyramids; (3) handle errors with `.catch()` or `try...catch`; (4) prefer `async/await` for complex control flow; (5) avoid the explicit Promise construction anti-pattern; (6) use `Promise.all()`, `Promise.allSettled()`, etc. for concurrency.

**Beginner-Friendly Explanation:** Best practices are the rules of thumb for using Promises correctly. Always return from `.then()`. Always handle errors. Use flat chains instead of nesting. Prefer `async/await` for complex flows. And use the built-in combinator methods for running things in parallel.

### Purposes

- To write maintainable and readable asynchronous code.
- To ensure errors are always handled.
- To avoid common pitfalls and anti-patterns.
- To leverage the full power of Promise composition.

### Syntax Rules and Structure

Best practices in practice:

```js
// GOOD: Flat chain with error handling
fetch("/api/user")
  .then((res) => res.json())
  .then((user) => fetch(`/api/posts/${user.id}`))
  .then((res) => res.json())
  .then((posts) => render(posts))
  .catch((err) => showError(err));

// GOOD: Async/await with try/catch
async function loadData() {
  try {
    const userRes = await fetch("/api/user");
    const user = await userRes.json();
    const postsRes = await fetch(`/api/posts/${user.id}`);
    const posts = await postsRes.json();
    render(posts);
  } catch (err) {
    showError(err);
  }
}
```

**Component Breakdown:**
- **Always return:** Ensures the chain is not broken.
- **Flat chains:** Avoid nesting and pyramid structures.
- **Error handling:** Every chain should end with `.catch()` or be wrapped in `try...catch`.
- **Async/await:** Preferred for complex control flow.

**Syntax Rules:**
- Return a value or Promise from every `.then()` handler.
- Keep chains flat and readable.
- Always handle rejections.
- Use `async/await` for sequential operations and `Promise.all()` for parallel operations.

**Constraints and Limitations:**
- `async/await` is built on Promises; understanding Promises is still essential.
- Some legacy APIs still use callbacks; `util.promisify` can convert them.

### Annotated Code Examples

#### Example 1: Flat Chain vs Nested Pyramid

```js
// GOOD: Flat chain
getUser()
  .then((user) => getPosts(user.id))
  .then((posts) => getComments(posts[0].id))
  .then((comments) => render(comments))
  .catch((err) => showError(err));

// BAD: Nested pyramid
getUser().then((user) => {
  getPosts(user.id).then((posts) => {
    getComments(posts[0].id).then((comments) => {
      render(comments);
    }).catch((err) => showError(err));
  }).catch((err) => showError(err));
}).catch((err) => showError(err));
```

**Expected Output:** Both work, but the flat chain is more readable and has a single error handler.

**Why This Output:** The flat chain avoids nesting, makes the flow clear, and centralises error handling.

#### Example 2: Parallel with Promise.all()

```js
// GOOD: Parallel loading
async function loadDashboard() {
  const [user, posts, stats] = await Promise.all([
    fetch("/api/user").then((r) => r.json()),
    fetch("/api/posts").then((r) => r.json()),
    fetch("/api/stats").then((r) => r.json()),
  ]);
  return { user, posts, stats };
}

// BAD: Sequential loading (slower)
async function loadDashboardSlow() {
  const user = await fetch("/api/user").then((r) => r.json());
  const posts = await fetch("/api/posts").then((r) => r.json());
  const stats = await fetch("/api/stats").then((r) => r.json());
  return { user, posts, stats };
}
```

**Expected Output:** The parallel version is faster because all requests run concurrently.

**Why This Output:** `Promise.all()` starts all three fetches simultaneously, while sequential `await` waits for each one to finish before starting the next.

### Real-World Cases

- **Node.js applications:** Using flat Promise chains and error handling.
- **React:** Using `async/await` in effects and event handlers.
- **API clients:** Using `Promise.all()` for parallel data fetching.
- **Testing:** Using `Promise.resolve()` and `Promise.reject()` for test fixtures.

---

## References

- MDN Web Docs – Promise — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise
- MDN Web Docs – Using promises — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises
- MDN Web Docs – Promise.withResolvers() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/withResolvers
- MDN Web Docs – Promise.allSettled() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/allSettled
- MDN Web Docs – Promise.any() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/any
- MDN Web Docs – Promise.all() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all
- MDN Web Docs – Promise.race() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/race
- MDN Web Docs – Promise.prototype.then() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/then
- MDN Web Docs – Promise.prototype.catch() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/catch
- MDN Web Docs – Promise.prototype.finally() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/finally
- MDN Web Docs – Promise() constructor — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/Promise
- OWASP – NodeJS Security Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Nodejs_Security_Cheat_Sheet.html
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – §25.4 Promise Objects — https://ecma-international.org/wp-content/uploads/ECMA-262_16th_edition_june_2025.pdf
- Promises/A+ Specification — https://promisesaplus.com/
- TC39 – Promise.allSettled Proposal — https://tc39.es/proposal-promise-allSettled/
- TC39 – Promise.any Proposal — https://github.com/tc39/proposal-promise-any
- TC39 – Promise.withResolvers Proposal — https://github.com/tc39/proposal-promise-with-resolvers