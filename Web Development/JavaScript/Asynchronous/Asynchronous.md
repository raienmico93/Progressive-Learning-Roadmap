# Asynchronous Programming Fundamentals — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Asynchronous programming is a programming paradigm that allows a program to start a potentially long-running operation and continue executing other code while waiting for that operation to complete, rather than blocking execution until the operation finishes.

**Technical Definition:** In JavaScript, asynchronous programming is enabled by the runtime's **event loop** — a concurrency model responsible for executing code, collecting and processing events, and executing queued sub-tasks. JavaScript is single-threaded, meaning only one piece of JavaScript code can execute at a time. Asynchronous operations (I/O, timers, network requests) are delegated to the host environment (browser or Node.js), and their callbacks or promise continuations are queued for execution once the current call stack is empty. Per the Node.js documentation, "JavaScript execution in Node.js is single threaded, so concurrency refers to the event loop's capacity to execute JavaScript callback functions after completing other work".

**Beginner-Friendly Explanation:** Normally, JavaScript runs code one line at a time, from top to bottom. If one line takes a long time (like downloading a file), everything else has to wait. Asynchronous programming lets you say "start this download, and when it's done, run this other code" — meanwhile, the rest of your program keeps running. This is essential for responsive web pages and efficient servers.

### Key Characteristics

- **Single-threaded:** JavaScript executes one piece of code at a time; there is no true multi-threading in the main execution context.
- **Non-blocking:** Asynchronous operations return immediately, allowing the program to continue.
- **Event-loop driven:** The event loop orchestrates the execution of queued callbacks and microtasks.
- **Callback-based or promise-based:** Asynchronous results are delivered via callbacks, promises, or `async/await`.
- **Concurrency without parallelism:** JavaScript achieves concurrency (interleaved execution) but not parallelism (simultaneous execution) in the main thread.
- **Run-to-completion:** Each message in the event loop queue is processed completely before the next message.

### Prerequisites

- Understanding of JavaScript functions (declarations, expressions, arrow functions).
- Familiarity with the call stack and execution contexts.
- Basic knowledge of callbacks and higher-order functions.
- Awareness of the difference between synchronous and asynchronous code.

### Related Programming Areas

- **Web development:** Fetching data from APIs, handling user events, and updating the DOM.
- **Node.js:** File I/O, database queries, and network servers.
- **Real-time applications:** WebSockets, chat applications, and live updates.
- **Performance optimisation:** Non-blocking I/O, worker threads, and parallel processing.

### Core Concepts / Features

---

## Synchronous vs Asynchronous

### Definitions

**Core Definition:** Synchronous code executes one statement at a time, in order, waiting for each operation to complete before moving to the next. Asynchronous code starts an operation, moves on to other work, and handles the result when it becomes available.

**Technical Definition:** Synchronous execution refers to code executing in sequence — each statement must complete before the next begins. Asynchronous execution does not run in the sequence it appears in the code; operations are started and their results are handled later via callbacks, promises, or async/await. JavaScript has a mix of synchronous and asynchronous APIs.

**Beginner-Friendly Explanation:** Synchronous is like standing in a queue: you wait for the person in front of you to finish before you can move. Asynchronous is like ordering food at a restaurant: you place your order, sit down, and the waiter brings your food when it's ready — you don't have to stand at the counter waiting.

### Purposes

- To understand the fundamental execution model of JavaScript.
- To choose the appropriate approach for different types of operations.
- To avoid blocking the main thread with long-running synchronous operations.
- To write responsive user interfaces and scalable servers.

### Syntax Rules and Structure

```js
// Synchronous:
console.log("First");
console.log("Second");
console.log("Third");
// Output: First, Second, Third

// Asynchronous:
console.log("First");
setTimeout(() => console.log("Second"), 0);
console.log("Third");
// Output: First, Third, Second
```

**Component Breakdown:**
- Synchronous statements execute in order.
- Asynchronous operations (e.g., `setTimeout`) schedule a callback and return immediately.

**Syntax Rules:**
- Synchronous code blocks the call stack until it completes.
- Asynchronous code returns immediately and delivers its result via a callback, promise, or async/await.
- The event loop ensures that callbacks are executed only after the current call stack is empty.

**Constraints and Limitations:**
- Blocking synchronous code freezes the UI in browsers and prevents the server from handling other requests.
- Asynchronous code is more complex to reason about because execution order is not linear.

### Annotated Code Examples

#### Example 1: Synchronous vs Asynchronous Execution Order

```js
// Step 1: Synchronous logging.
console.log("A");

// Step 2: Asynchronous operation (scheduled for later).
setTimeout(() => {
  console.log("B (async)");
}, 0);

// Step 3: More synchronous logging.
console.log("C");
```

**Expected Output:**
```
A
C
B (async)
```

**Why This Output:** `setTimeout` schedules its callback for the next event loop iteration. The synchronous code (`console.log("A")` and `console.log("C")`) runs first because the call stack is not yet empty. Only after the synchronous code finishes does the event loop execute the callback, logging `"B (async)"`.

#### Example 2: Blocking vs Non-Blocking File Read (Node.js)

```js
const fs = require("node:fs");

// Synchronous (blocking):
const dataSync = fs.readFileSync("/file.md");
console.log("Sync read complete");
// Execution blocks here until the file is read.

// Asynchronous (non-blocking):
fs.readFile("/file.md", (err, data) => {
  if (err) throw err;
  console.log("Async read complete");
});
console.log("This runs before the async read completes");
```

**Expected Output:**
```
Sync read complete
This runs before the async read completes
Async read complete
```

**Why This Output:** `readFileSync` blocks until the file is fully read, so `"Sync read complete"` is logged first. The asynchronous `readFile` returns immediately; `"This runs before the async read completes"` executes before the callback fires with the file data.

### Real-World Cases

- **Web APIs:** `fetch()` is asynchronous; it returns a promise immediately and resolves when the response arrives.
- **Node.js servers:** Asynchronous I/O allows a single server to handle thousands of concurrent connections.
- **User interfaces:** Asynchronous event handlers keep the UI responsive during data loading.

---

## Blocking vs Non-Blocking

### Definitions

**Core Definition:** Blocking operations prevent the execution of additional JavaScript until they complete. Non-blocking operations return immediately, allowing the program to continue.

**Technical Definition:** Per Node.js documentation, "Blocking is when the execution of additional JavaScript in the Node.js process must wait until a non-JavaScript operation completes". All I/O methods in the Node.js standard library provide asynchronous (non-blocking) versions that accept callbacks, as well as blocking counterparts whose names end with `Sync`. Blocking methods execute synchronously; non-blocking methods execute asynchronously.

**Beginner-Friendly Explanation:** A blocking operation is like a traffic jam — nothing else can move until it clears. A non-blocking operation is like merging onto a highway — you keep moving while other cars do their thing. In JavaScript, blocking the main thread means the browser can't respond to clicks, and a server can't handle other requests.

### Purposes

- To identify operations that may freeze the UI or server.
- To choose non-blocking alternatives for I/O and network operations.
- To understand why asynchronous APIs are preferred in performance-critical code.
- To avoid blocking the event loop with CPU-intensive tasks.

### Syntax Rules and Structure

```js
// Blocking (synchronous):
const data = fs.readFileSync("/file.md"); // Blocks here until file is read
console.log(data);
moreWork(); // Runs after console.log

// Non-blocking (asynchronous):
fs.readFile("/file.md", (err, data) => {
  if (err) throw err;
  console.log(data);
});
moreWork(); // Runs before console.log
```

**Component Breakdown:**
- `readFileSync`: Blocking; returns the data directly.
- `readFile`: Non-blocking; returns immediately and calls the callback with the data.

**Syntax Rules:**
- Blocking methods typically have names ending in `Sync`.
- Non-blocking methods accept a callback (or return a promise) and return immediately.
- Blocking methods should be avoided in performance-critical sections of the event loop.

**Constraints and Limitations:**
- Blocking the event loop prevents all other operations from proceeding.
- CPU-bound operations are not considered "blocking" in the I/O sense but can still block the event loop if they take too long.

### Annotated Code Examples

#### Example 1: Blocking vs Non-Blocking Execution Order

```js
const fs = require("node:fs");

// Blocking:
const dataSync = fs.readFileSync("/file.md");
console.log(dataSync);
moreWork(); // Runs after console.log

// Non-blocking:
fs.readFile("/file.md", (err, data) => {
  if (err) throw err;
  console.log(data);
});
moreWork(); // Runs before console.log
```

**Expected Output (asynchronous version):**
```
moreWork output
(file data logged after read completes)
```

**Why This Output:** In the asynchronous version, `fs.readFile()` is non-blocking, so JavaScript execution continues and `moreWork()` is called first. The file data is logged later, when the read operation completes and its callback is placed in the event loop queue.

#### Example 2: Impact on Throughput

```js
// Suppose each request takes 50ms, with 45ms of database I/O.
// Blocking: each request ties up the server for the full 50ms.
// Non-blocking: the 45ms of I/O frees up the event loop to handle other requests.
```

**Expected Output:** Non-blocking code can handle significantly more concurrent requests because the event loop is not blocked during I/O.

### Real-World Cases

- **Web servers:** Non-blocking I/O allows a single Node.js process to handle thousands of concurrent connections.
- **Browsers:** Blocking the main thread freezes the UI; non-blocking event handlers keep it responsive.
- **File processing:** Reading large files synchronously can cause noticeable delays.

---

## Callbacks

### Definitions

**Core Definition:** A callback is a function passed as an argument to another function, which is then invoked inside the outer function to complete some kind of routine or action.

**Technical Definition:** In asynchronous programming, callbacks are functions registered with an asynchronous API. The API stores the callback and invokes it from the event loop once the operation is complete. Per Node.js documentation, the callback pattern is the original mechanism for handling asynchronous results.

**Beginner-Friendly Explanation:** A callback is a function you give to another function and say, "Call this when you're done." For example, when you read a file, you pass a callback that receives the file data. The callback runs later, when the data is ready.

### Purposes

- To handle asynchronous results without blocking execution.
- To customise the behaviour of asynchronous APIs.
- To implement event-driven programming.
- To provide a mechanism for the event loop to resume execution after an operation completes.

### Syntax Rules and Structure

```js
asyncOperation((err, data) => {
  if (err) {
    // Handle error
  } else {
    // Use data
  }
});
```

**Component Breakdown:**
- `asyncOperation`: The asynchronous function.
- `(err, data) => {}`: The callback, using the error-first convention (Node.js style).
- `err`: The error object (or `null` on success).
- `data`: The result of the operation.

**Syntax Rules:**
- Callbacks are passed as arguments to asynchronous functions.
- Node.js uses the error-first callback convention: the first argument is the error.
- Callbacks are invoked by the event loop when the operation completes.

**Constraints and Limitations:**
- Callbacks can lead to deeply nested code (callback hell).
- Error handling requires explicit checks in each callback.
- Callbacks invert control: the API decides when to call your function.

### Annotated Code Examples

#### Example 1: Basic Callback

```js
// Step 1: Define an asynchronous function that accepts a callback.
function fetchData(callback) {
  setTimeout(() => {
    callback(null, { id: 1, name: "Item" });
  }, 1000);
}

// Step 2: Call with an error-first callback.
fetchData((err, data) => {
  if (err) {
    console.error("Error:", err);
  } else {
    console.log("Data:", data);
  }
});
```

**Expected Output (after 1 second):**
```
Data: { id: 1, name: 'Item' }
```

**Why This Output:** The callback is invoked after 1 second with `null` for the error and the data object. The callback checks `err` and logs the data.

#### Example 2: Nested Callbacks (Callback Hell)

```js
// Step 1: Nested asynchronous operations.
getUser(userId, (err, user) => {
  if (err) return handleError(err);
  getPosts(user.id, (err, posts) => {
    if (err) return handleError(err);
    getComments(posts[0].id, (err, comments) => {
      if (err) return handleError(err);
      displayComments(comments);
    });
  });
});
```

**Expected Output:** The comments are displayed after all nested operations complete.

**Why This Output:** Each callback depends on the result of the previous one, leading to deeply nested code — the "pyramid of doom."

### Real-World Cases

- **Node.js file system:** `fs.readFile(path, (err, data) => { ... })`.
- **Event listeners:** `button.addEventListener("click", handler)`.
- **Timers:** `setTimeout(callback, delay)`.

---

## Callback Hell

### Definitions

**Core Definition:** Callback hell (also called the "pyramid of doom") is an anti-pattern in asynchronous programming where deeply nested callbacks make code difficult to read, understand, and maintain.

**Technical Definition:** Callback hell occurs when multiple asynchronous operations depend on each other, leading to deeply nested callback functions. Per the OWASP cheat sheet, "In such code, the errors and results get lost within the callback". It is characterised by a pyramid-shaped indentation of nested functions.

**Beginner-Friendly Explanation:** Callback hell is what happens when you need to do several asynchronous things in a row, and each one has to wait for the previous one to finish. You end up with a bunch of nested functions that look like a pyramid, making the code hard to read and fix.

### Purposes

- To recognise and avoid the anti-pattern in asynchronous code.
- To motivate the use of Promises and async/await.
- To understand the limitations of the callback pattern.
- To refactor existing callback-based code into more maintainable forms.

### Syntax Rules and Structure

```js
firstOperation((err, result1) => {
  if (err) return handleError(err);
  secondOperation(result1, (err, result2) => {
    if (err) return handleError(err);
    thirdOperation(result2, (err, result3) => {
      if (err) return handleError(err);
      // ... and so on
    });
  });
});
```

**Component Breakdown:**
- Each nested callback depends on the result of the previous one.
- Error handling is repeated at each level.

**Syntax Rules:**
- Callback hell is not a syntax error; it is a code smell.
- It arises when callbacks are used for sequential asynchronous operations.
- Promises and `async/await` are the primary solutions.

**Constraints and Limitations:**
- Deeply nested code is hard to read and maintain.
- Error handling becomes repetitive and error-prone.
- The structure does not compose well.

### Annotated Code Examples

#### Example 1: Callback Hell Example

```js
getUser(userId, (err, user) => {
  if (err) return console.error(err);
  getPosts(user.id, (err, posts) => {
    if (err) return console.error(err);
    getComments(posts[0].id, (err, comments) => {
      if (err) return console.error(err);
      console.log(comments);
    });
  });
});
```

**Expected Output:** The comments are logged after all three operations complete.

**Why This Output:** Each operation is nested inside the previous one's callback, creating a pyramid structure.

#### Example 2: Refactored with Promises

```js
getUser(userId)
  .then((user) => getPosts(user.id))
  .then((posts) => getComments(posts[0].id))
  .then((comments) => console.log(comments))
  .catch((err) => console.error(err));
```

**Expected Output:** The same result, but with flat, readable code.

**Why This Output:** Promises allow chaining, eliminating the nesting and centralising error handling.

### Real-World Cases

- **Legacy codebases:** Older Node.js code often contains callback hell.
- **Third-party APIs:** Some libraries still use callbacks and require careful wrapping.
- **Refactoring projects:** Converting callback-based code to promises is a common task.

---

## Inversion of Control

### Definitions

**Core Definition:** Inversion of control (IoC) is a design principle where the control of program flow is inverted — instead of your code calling library functions, a framework or API calls your code. In the context of callbacks, IoC means you hand over control to an asynchronous API, which decides when to invoke your callback.

**Technical Definition:** Per the reference, "Inversion of control is one of the problems of using callbacks for asynchronous programming in JavaScript. Callbacks force you to trust another party". The callback approach is called an "inversion of control" because the function that accepts a callback says "Don't call me, I'll call you". Promises "un-invert" the inversion by separating input arguments from control-flow arguments.

**Beginner-Friendly Explanation:** Normally, you control when your functions run. But when you pass a callback to an API, you're giving up that control — the API decides when to call your function. This is "inversion of control." It can be risky because you have to trust the API to call your callback correctly and at the right time. Promises give you back some of that control.

### Purposes

- To understand the trust relationship between your code and asynchronous APIs.
- To recognise the risks of handing control to third-party libraries.
- To appreciate why Promises were introduced as a solution.
- To design APIs that minimise the risks of inversion of control.

### Syntax Rules and Structure

Inversion of control is a conceptual issue, not a syntax construct. The pattern is:

```js
// Your code hands control to the API:
thirdPartyAPI(data, (err, result) => {
  // You trust the API to call this callback correctly.
});
```

**Component Breakdown:**
- `thirdPartyAPI`: The external API that receives your callback.
- The API decides when (and how many times) to call your callback.

**Syntax Rules:**
- IoC is inherent in the callback pattern.
- Promises reduce IoC by returning a result object you can attach handlers to.

**Constraints and Limitations:**
- The API may call your callback too early, too late, multiple times, or never.
- You cannot easily enforce timeout or cancellation with callbacks.

### Annotated Code Examples

#### Example 1: IoC with Callbacks

```js
// Step 1: A third-party API that takes a callback.
function unreliableAPI(callback) {
  // The API might call the callback multiple times or not at all.
  setTimeout(() => callback(null, "Result"), 100);
}

// Step 2: Your code hands over control.
unreliableAPI((err, result) => {
  console.log(result); // You trust the API to call this once.
});
```

**Expected Output:**
```
Result
```

**Why This Output:** The API controls when the callback is invoked. Your code has no control over the timing or number of invocations.

#### Example 2: Un-inverting with Promises

```js
function reliableAPI() {
  return new Promise((resolve, reject) => {
    setTimeout(() => resolve("Result"), 100);
  });
}

reliableAPI()
  .then((result) => console.log(result))
  .catch((err) => console.error(err));
```

**Expected Output:**
```
Result
```

**Why This Output:** The promise returns control to the caller. You decide when to attach handlers and how to handle errors. The promise can only resolve or reject once.

### Real-World Cases

- **Third-party libraries:** Using APIs that accept callbacks requires trusting their implementation.
- **Promise adoption:** Promises were introduced to mitigate IoC risks.
- **Cancellation:** Promises (and AbortController) allow cancellation, which callbacks do not.

---

## Promises

### Definitions

**Core Definition:** A Promise is an object that represents the eventual completion (or failure) of an asynchronous operation and its resulting value.

**Technical Definition:** Per MDN, "A promise is an object returned by an asynchronous function, which represents the current state of the operation". A promise can be in one of three states: **pending** (operation not yet complete), **fulfilled** (operation succeeded), or **rejected** (operation failed). Promises provide methods (`then`, `catch`, `finally`) to attach handlers for these outcomes.

**Beginner-Friendly Explanation:** A promise is like a receipt you get when you order something online. It's not the item itself — it's a guarantee that you'll either get the item or a notification that it couldn't be delivered. You can attach a handler to say what to do when it arrives, and another to say what to do if it fails.

### Purposes

- To provide a cleaner, more manageable way to handle asynchronous operations.
- To avoid callback hell through chaining.
- To centralise error handling with `.catch()`.
- To enable composition of multiple asynchronous operations (`Promise.all`, `Promise.race`).
- To un-invert control by returning a result object.

### Syntax Rules and Structure

```js
// Creating a promise:
const promise = new Promise((resolve, reject) => {
  // Async operation
  if (success) {
    resolve(value);
  } else {
    reject(error);
  }
});

// Using a promise:
promise
  .then((value) => { /* handle success */ })
  .catch((error) => { /* handle failure */ })
  .finally(() => { /* cleanup */ });
```

**Component Breakdown:**
- `resolve`: Function to call when the operation succeeds.
- `reject`: Function to call when the operation fails.
- `then`: Attaches a success handler.
- `catch`: Attaches a failure handler.
- `finally`: Attaches a handler that runs regardless of outcome.

**Syntax Rules:**
- A promise can only settle once (either fulfilled or rejected).
- `then` returns a new promise, enabling chaining.
- Errors propagate down the chain until a `catch` handler is found.

**Constraints and Limitations:**
- Promises cannot be cancelled (though `AbortController` can be used with some APIs).
- Unhandled rejections trigger global events.
- Promises are eager: the executor function runs immediately.

### Annotated Code Examples

#### Example 1: Basic Promise Usage

```js
// Step 1: Create a promise.
const fetchData = new Promise((resolve, reject) => {
  setTimeout(() => {
    const success = true;
    if (success) {
      resolve({ id: 1, name: "Item" });
    } else {
      reject(new Error("Failed to fetch"));
    }
  }, 1000);
});

// Step 2: Attach handlers.
fetchData
  .then((data) => console.log("Data:", data))
  .catch((err) => console.error("Error:", err.message));
```

**Expected Output (after 1 second):**
```
Data: { id: 1, name: 'Item' }
```

**Why This Output:** The promise resolves with the data object after 1 second. The `then` handler receives the data and logs it.

#### Example 2: Promise Chaining

```js
// Step 1: Define promise-returning functions.
function getUser(id) {
  return Promise.resolve({ id, name: "Alice" });
}
function getPosts(userId) {
  return Promise.resolve([{ id: 1, title: "Post 1" }]);
}

// Step 2: Chain them.
getUser(1)
  .then((user) => getPosts(user.id))
  .then((posts) => console.log(posts))
  .catch((err) => console.error(err));
```

**Expected Output:**
```
[ { id: 1, title: 'Post 1' } ]
```

**Why This Output:** Each `then` returns a new promise, allowing the chain to proceed. Errors are caught by a single `catch` at the end.

### Real-World Cases

- **Fetch API:** `fetch()` returns a promise that resolves with the HTTP response.
- **Database queries:** Modern database drivers return promises.
- **Node.js utilities:** `fs.promises` provides promise-based file system methods.

---

## Async/Await

### Definitions

**Core Definition:** `async/await` is syntactic sugar built on top of Promises that allows asynchronous code to be written in a synchronous style, making it easier to read and reason about.

**Technical Definition:** Per MDN, "The `async` keyword before a function declaration makes it an asynchronous function. The `await` keyword pauses the function execution until the Promise resolves". `async/await` is syntactic sugar using generators and `yield` statements to "pause" execution. An `async` function always returns a promise, and `await` can only be used inside `async` functions (or top-level modules).

**Beginner-Friendly Explanation:** `async/await` lets you write asynchronous code that looks like regular synchronous code. You put `async` before a function, and then use `await` before any promise. The `await` makes the function wait for the promise to resolve, then continues. It's much easier to read than a chain of `.then()` calls.

### Purposes

- To write asynchronous code that reads like synchronous code.
- To simplify error handling with `try...catch`.
- To improve readability and maintainability of asynchronous logic.
- To avoid the complexity of promise chains for sequential operations.

### Syntax Rules and Structure

```js
async function functionName() {
  try {
    const result = await somePromise();
    // Use result
  } catch (error) {
    // Handle error
  }
}
```

**Component Breakdown:**
- `async`: Marks a function as asynchronous; it always returns a promise.
- `await`: Pauses the function until the promise settles, returning the resolved value or throwing the rejection reason.

**Syntax Rules:**
- `await` can only be used inside `async` functions (or top-level modules).
- `async` functions always return a promise.
- Error handling uses `try...catch`.

**Constraints and Limitations:**
- `await` blocks the execution of the async function (but not the entire program).
- Using `await` in a loop can serialise operations that could run in parallel.
- `async/await` is built on promises; it does not replace them.

### Annotated Code Examples

#### Example 1: Basic Async/Await

```js
// Step 1: Define an async function.
async function fetchUser() {
  try {
    const response = await fetch("/api/user");
    const user = await response.json();
    console.log(user);
  } catch (error) {
    console.error("Error:", error.message);
  }
}

fetchUser();
```

**Expected Output:** The user data is logged after the fetch completes.

**Why This Output:** `await` pauses the function until the fetch promise resolves, then continues. Errors are caught by the `try...catch` block.

#### Example 2: Sequential vs Parallel with Async/Await

```js
// Sequential (slower):
async function sequential() {
  const user = await getUser();
  const posts = await getPosts(user.id);
  return posts;
}

// Parallel (faster):
async function parallel() {
  const [user, posts] = await Promise.all([getUser(), getPosts()]);
  return posts;
}
```

**Expected Output:** Parallel execution is faster when operations are independent.

**Why This Output:** `Promise.all` starts both operations simultaneously, while sequential `await` waits for each one to finish before starting the next.

### Real-World Cases

- **API calls:** Fetching data from multiple endpoints.
- **Database operations:** Running queries and processing results.
- **File system:** Reading and writing files in Node.js.

---

## Event Loop

### Definitions

**Core Definition:** The event loop is the runtime mechanism that allows JavaScript to perform non-blocking asynchronous operations by coordinating the call stack, the message queue, and the microtask queue.

**Technical Definition:** Per MDN, "JavaScript has a runtime model based on an event loop, which is responsible for executing the code, collecting and processing events, and executing queued sub-tasks". The event loop continuously checks if the call stack is empty; if so, it takes the first message from the queue and processes it, creating a new call stack frame for the associated function. This process repeats indefinitely.

**Beginner-Friendly Explanation:** The event loop is like a manager who keeps checking: "Is the current task done? Yes? Then take the next task from the queue and run it." It ensures that JavaScript can handle many tasks without getting stuck on any one of them.

### Purposes

- To enable asynchronous, non-blocking execution in a single-threaded language.
- To coordinate the execution of callbacks, promises, and timers.
- To ensure that the call stack is never blocked by long-running operations.
- To provide a predictable execution model for JavaScript programs.

### Syntax Rules and Structure

The event loop is not a syntax construct; it is a runtime mechanism. The conceptual algorithm is:

```js
while (queue.waitForMessage()) {
  queue.processNextMessage();
}
```

**Component Breakdown:**
- **Call stack:** Tracks the currently executing function.
- **Message queue (task queue):** Holds messages (callbacks) waiting to be processed.
- **Microtask queue:** Holds promise callbacks and other microtasks, processed after each message.

**Syntax Rules:**
- Each message is processed completely before the next one (run-to-completion).
- Microtasks are processed immediately after the current task, before the next message.
- The event loop is blocked if a message takes too long to process.

**Constraints and Limitations:**
- Long-running synchronous code blocks the event loop, freezing the UI.
- The event loop does not guarantee exact timing for `setTimeout` and `setInterval`.

### Annotated Code Examples

#### Example 1: Event Loop Order

```js
console.log("Script start");

setTimeout(() => {
  console.log("setTimeout");
}, 0);

Promise.resolve().then(() => {
  console.log("Promise");
});

console.log("Script end");
```

**Expected Output:**
```
Script start
Script end
Promise
setTimeout
```

**Why This Output:** Synchronous code runs first (`Script start`, `Script end`). Then microtasks (promise callbacks) are processed, logging `Promise`. Finally, the macrotask (`setTimeout`) is processed, logging `setTimeout`.

#### Example 2: Blocking the Event Loop

```js
setTimeout(() => {
  console.log("Timeout");
}, 0);

// Block the event loop for 2 seconds.
const start = Date.now();
while (Date.now() - start < 2000) {}
console.log("Blocking done");
```

**Expected Output:**
```
Blocking done
Timeout
```

**Why This Output:** The `while` loop blocks the event loop for 2 seconds. The `setTimeout` callback cannot run until the blocking code finishes, even though the timeout is set to 0.

### Real-World Cases

- **UI responsiveness:** Event loop blocking causes unresponsive interfaces.
- **Server throughput:** Non-blocking I/O allows servers to handle many requests.
- **Animation:** `requestAnimationFrame` schedules work before the next repaint.

---

## Concurrency

### Definitions

**Core Definition:** Concurrency is the ability of a program to handle multiple tasks by interleaving their execution, making progress on each without necessarily running them simultaneously.

**Technical Definition:** In JavaScript, concurrency is achieved through the event loop, which allows multiple asynchronous operations to be in progress at the same time by interleaving their callbacks. Per Stack Overflow, "Due to JavaScript being single-threaded, it is technically only concurrent, not parallel". The event loop allows us to coordinate the order in which instructions are executed such that certain instructions are non-blocking.

**Beginner-Friendly Explanation:** Concurrency is like a chef who starts boiling water, then chops vegetables while the water heats, then adds pasta while the vegetables cook. The chef is doing multiple things, but not literally at the same instant — they're switching between tasks. JavaScript does this with the event loop.

### Purposes

- To handle multiple asynchronous operations efficiently.
- To improve throughput by not waiting for one operation to finish before starting another.
- To enable responsive user interfaces during long-running operations.
- To model real-world scenarios where multiple things happen at once.

### Syntax Rules and Structure

Concurrency in JavaScript is achieved through:

```js
// Starting multiple asynchronous operations:
const promise1 = fetch("/api/data1");
const promise2 = fetch("/api/data2");

// Both are in progress concurrently.
Promise.all([promise1, promise2]).then(([data1, data2]) => {
  // Both completed.
});
```

**Component Breakdown:**
- Multiple promises are started simultaneously.
- The event loop interleaves their callbacks as each completes.

**Syntax Rules:**
- Concurrency is achieved through non-blocking operations.
- `Promise.all` waits for multiple promises concurrently.
- The event loop ensures that callbacks are processed as operations complete.

**Constraints and Limitations:**
- Concurrency does not mean simultaneous execution; only one piece of JavaScript runs at a time.
- CPU-bound operations cannot be made concurrent without worker threads.

### Annotated Code Examples

#### Example 1: Concurrent Fetch Requests

```js
// Start multiple fetches concurrently.
const userPromise = fetch("/api/user");
const postsPromise = fetch("/api/posts");
const commentsPromise = fetch("/api/comments");

Promise.all([userPromise, postsPromise, commentsPromise])
  .then(([user, posts, comments]) => {
    console.log("All data loaded");
  });
```

**Expected Output:** All data is loaded concurrently, and the total time is approximately the time of the slowest request, not the sum of all requests.

**Why This Output:** All three fetches start at the same time. The event loop processes their callbacks as each completes.

#### Example 2: Sequential vs Concurrent

```js
// Sequential (slower):
const user = await fetch("/api/user");
const posts = await fetch("/api/posts");
// Total time: time(user) + time(posts)

// Concurrent (faster):
const [user, posts] = await Promise.all([
  fetch("/api/user"),
  fetch("/api/posts"),
]);
// Total time: max(time(user), time(posts))
```

**Expected Output:** Concurrent execution is faster for independent operations.

**Why This Output:** `Promise.all` starts both fetches simultaneously, while sequential `await` waits for the first to finish before starting the second.

### Real-World Cases

- **Web scraping:** Fetching multiple pages concurrently.
- **Dashboard loading:** Loading multiple widgets in parallel.
- **Batch processing:** Processing multiple items concurrently.

---

## Parallelism

### Definitions

**Core Definition:** Parallelism is the simultaneous execution of multiple tasks on multiple threads or CPU cores. Unlike concurrency, parallelism requires multiple execution units.

**Technical Definition:** JavaScript is traditionally single-threaded, so true parallelism is not possible in the main execution context. However, **Web Workers** (in browsers) and **worker_threads** (in Node.js) allow JavaScript code to run in separate threads, enabling true parallelism. Per the reference, "True parallelism therefore does not exist [in the main thread], but a clever programming model still allows 'asynchronous' code".

**Beginner-Friendly Explanation:** Parallelism is like having multiple chefs in a kitchen, each doing a different task at the same time. JavaScript normally has one chef (one thread), but you can hire more chefs using Web Workers or worker_threads.

### Purposes

- To execute CPU-intensive tasks without blocking the main thread.
- To leverage multi-core processors for faster computation.
- To improve performance for tasks that can be parallelised.
- To keep the UI responsive during heavy computation.

### Syntax Rules and Structure

```js
// Browser: Web Worker
const worker = new Worker("worker.js");
worker.postMessage({ data: largeArray });
worker.onmessage = (event) => {
  console.log("Result:", event.data);
};

// Node.js: worker_threads
const { Worker } = require("node:worker_threads");
const worker = new Worker("./worker.js");
worker.postMessage({ data: largeArray });
worker.on("message", (result) => {
  console.log("Result:", result);
});
```

**Component Breakdown:**
- `Worker`: Creates a new thread.
- `postMessage`: Sends data to the worker.
- `onmessage` / `on("message")`: Receives results from the worker.

**Syntax Rules:**
- Workers run in a separate thread with their own event loop.
- Communication is via message passing (structured clone).
- Workers cannot access the DOM or shared memory (unless using `SharedArrayBuffer`).

**Constraints and Limitations:**
- Workers have overhead; they are best for CPU-intensive tasks.
- Data is copied between threads (unless using `SharedArrayBuffer`).
- Workers cannot share the same memory by default.

### Annotated Code Examples

#### Example 1: Web Worker for CPU-Intensive Task

```js
// Main thread:
const worker = new Worker("worker.js");

worker.postMessage({ numbers: [1, 2, 3, 4, 5] });

worker.onmessage = (event) => {
  console.log("Sum:", event.data.sum);
};

// worker.js:
self.onmessage = (event) => {
  const sum = event.data.numbers.reduce((a, b) => a + b, 0);
  self.postMessage({ sum });
};
```

**Expected Output:**
```
Sum: 15
```

**Why This Output:** The worker computes the sum in a separate thread, keeping the main thread responsive. The result is sent back via `postMessage`.

#### Example 2: Node.js worker_threads

```js
const { Worker } = require("node:worker_threads");

const worker = new Worker(`
  const { parentPort } = require("node:worker_threads");
  parentPort.on("message", (data) => {
    const result = data.map((n) => n * 2);
    parentPort.postMessage(result);
  });
`, { eval: true });

worker.postMessage([1, 2, 3]);
worker.on("message", (result) => {
  console.log(result); // [2, 4, 6]
});
```

**Expected Output:**
```
[ 2, 4, 6 ]
```

**Why This Output:** The worker thread processes the array in parallel with the main thread, sending the result back when done.

### Real-World Cases

- **Image processing:** Applying filters to large images.
- **Data analysis:** Processing large datasets in the background.
- **Game development:** Physics simulations and AI calculations.
- **Video encoding:** Transcoding video without freezing the UI.

---

## References

- Node.js Documentation – Overview of Blocking vs Non-Blocking — https://nodejs.org/learn/asynchronous-work/overview-of-blocking-vs-non-blocking
- MDN Web Docs – Event Loop — https://mdn.org.cn/en-US/docs/Web/JavaScript/Event_loop
- MDN Web Docs – How to Use Promises — https://mdn.org.cn/en-US/docs/Learn_web_development/Extensions/Async_JS/Promises
- MDN Web Docs – How to Use Promises (English) — http://developer.typescripts.org/en-US/docs/Learn/JavaScript/Asynchronous/Promises
- OWASP – NodeJS Security Cheat Sheet (Callback Hell) — https://cheatsheetseries.owasp.org
- Stack Overflow – Is async in JavaScript concurrent or parallel? — https://stackoverflow.com
- JavaScript.info – Event Loop: Microtasks and Macrotasks — https://javascript.info/event-loop
- Toptal – Asynchronous JavaScript: From Callback Hell to Async and Await — https://www.toptal.com
- DigitalOcean – Managing Asynchronous Code in JavaScript — https://www.digitalocean.com
- FreeCodeCamp – When to Use Async/Await vs Promises in JavaScript — https://www.freecodecamp.org