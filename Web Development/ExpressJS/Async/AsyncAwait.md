# Async/Await & Promise Integration — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Async/await is a JavaScript syntax that allows asynchronous, Promise-based code to be written in a sequential, synchronous-looking style, making it easier to read, reason about, and handle errors. Promise integration in Express refers to the patterns and practices for using Promises and async functions within route handlers and middleware, and the differences between how Express 4 and Express 5 handle rejected Promises.

**Technical Definition:** A Promise is an object representing the eventual completion or failure of an asynchronous operation. It exists in one of three mutually exclusive states: **pending** (initial state, neither fulfilled nor rejected), **fulfilled** (the operation completed successfully), or **rejected** (the operation failed). When a Promise is created, its executor function runs immediately, but the callbacks registered via `.then()`, `.catch()`, or `.finally()` are queued as **microtasks** — they run after the current synchronous script completes but before any macrotasks (like `setTimeout` or I/O). The `async` keyword guarantees a function returns a Promise; `await` non-blockingly pauses execution within an async function until the Promise settles, resuming with the fulfilled value or throwing the rejection reason.

**Beginner-Friendly Explanation:** A Promise is like a restaurant order ticket. When you place an order, the kitchen gives you a ticket (the Promise). The ticket is "pending" — your food isn't ready yet. When your food is ready, the ticket is "fulfilled" — and you get your meal. If the kitchen runs out of ingredients, the ticket is "rejected" — and you get an explanation of what went wrong. `async/await` is like having a waiter who stands at the kitchen counter and waits for your ticket to resolve before bringing you your food — except the waiter doesn't block other tables while waiting; they just pause your table's service and handle other tables in the meantime.

### Key Characteristics

- **Three states:** Pending, fulfilled, rejected. Once settled, a Promise's state is permanent.
- **Microtask priority:** Promise callbacks run in the microtask queue, ahead of macrotasks like `setTimeout` and I/O.
- **Async functions return Promises:** The `async` keyword guarantees a Promise return value, converting returned values and thrown errors automatically.
- **Express 5 native support:** Route handlers and middleware returning a Promise call `next(value)` automatically on rejection.
- **Express 4 requires manual handling:** Rejected Promises in async middleware are **not** caught automatically — they crash the process or leave the request hanging.
- **Unhandled rejections crash Node.js:** In Node.js 15+, unhandled Promise rejections terminate the process by default.

### Prerequisites

- **Node.js runtime** (v18 or higher for Express 5.x; v16.4+ for stable `AsyncLocalStorage`).
- **Express.js installed:** `npm install express`.
- **Basic JavaScript knowledge:** Functions, callbacks, and the event loop.
- **Understanding of Promises:** `.then()`, `.catch()`, and the three states.

### Related Programming Areas

- **Error Handling Architecture:** Async errors feed into the centralised error middleware.
- **Middleware Fundamentals:** Async middleware follows the same `(req, res, next)` signature but returns a Promise.
- **Production Error Handling:** Unhandled rejections are a primary cause of production crashes.
- **Database Access:** Repository methods return Promises that are awaited in services.
- **External API Integration:** `fetch()` and SDK calls return Promises.

### Core Concepts

1. **Promise Fundamentals** — states, microtask queue priority, and avoiding unhandled rejections.
2. **Async Route Handlers** — writing clean, sequential controller code using async/await.
3. **Try/Catch Block Mechanics** — isolating execution blocks and letting errors bubble up.
4. **Error Propagation & Express Differences** — Express 4 vs. Express 5 async error handling.

---

## Core Concept 1: Promise Fundamentals

### Definitions

**Core Definition:** A Promise is a JavaScript object that represents the eventual completion (or failure) of an asynchronous operation, providing a standardised way to handle asynchronous results and errors.

**Technical Definition:** Any Promise object is in one of three mutually exclusive states: **pending**, **fulfilled**, or **rejected**. A Promise starts in the pending state. If the operation succeeds, it transitions to fulfilled with a value. If it fails, it transitions to rejected with a reason (typically an Error object). Once settled (fulfilled or rejected), the state is permanent. Promise callbacks registered via `.then()`, `.catch()`, and `.finally()` are queued as microtasks. The event loop processes all microtasks before moving to the next macrotask (e.g., `setTimeout`, I/O callbacks). An **unhandled rejection** occurs when a Promise rejects and no `.catch()` handler or `try/catch` wrapper exists to handle the error. In Node.js 15+, unhandled rejections terminate the process by default.

**Beginner-Friendly Explanation:** A Promise is like a "claim ticket" at a coat check. When you hand over your coat, you get a ticket (the Promise). The ticket is "pending." When you come back, either your coat is ready (fulfilled) or the coat check lost it (rejected). You can't change the ticket's state once it's settled. The microtask queue is like a VIP line — Promise callbacks always jump ahead of regular callbacks (like `setTimeout`).

### Purposes

- To represent the eventual result of an asynchronous operation in a standardised way.
- To enable chaining of asynchronous operations via `.then()` and `.catch()`.
- To provide a foundation for the `async/await` syntax.
- To ensure that asynchronous errors are captured and handled predictably.
- To prioritise Promise callbacks over macrotasks via the microtask queue.

### Syntax Rules and Structure

#### Promise States

Pending: neither fulfilled nor rejected
```js
const pending = new Promise(() => {});
console.log(pending); // Promise { <pending> }
```
\
Fulfilled: operation succeeded
```js
const fulfilled = Promise.resolve('success');
fulfilled.then(value => console.log(value)); // "success"
```
\
Rejected: operation failed
```js
const rejected = Promise.reject(new Error('failure'));
rejected.catch(err => console.log(err.message)); // "failure"
```

| State | Description | Transition |
|-------|-------------|-----------|
| Pending | Initial state; operation in progress. | Resolves to fulfilled or rejected. |
| Fulfilled | Operation completed successfully. | Permanent; `.then()` receives the value. |
| Rejected | Operation failed. | Permanent; `.catch()` receives the reason. |

#### Microtask Queue Priority

```javascript
console.log('1: synchronous');

Promise.resolve().then(() => console.log('2: microtask'));

setTimeout(() => console.log('3: macrotask'), 0);

console.log('4: synchronous');
```

**Expected Output:**
```
1: synchronous
4: synchronous
2: microtask
3: macrotask
```

**Why this output:** The synchronous `console.log` calls run first (`1` and `4`). Then the microtask queue processes the Promise callback (`2`). Finally, the macrotask queue processes the `setTimeout` callback (`3`). Promise callbacks always run before macrotasks.

#### Avoiding Unhandled Rejections

```javascript
// ❌ BAD: Unhandled rejection — crashes the process
async function fetchData() {
  const data = await fetch('https://api.example.com/data');
  return data.json();
}
fetchData(); // No .catch() or try/catch

// ✅ GOOD: Handle rejection
fetchData()
  .then(data => console.log(data))
  .catch(err => console.error('Fetch failed:', err.message));

// ✅ GOOD: try/catch in async function
async function safeFetch() {
  try {
    const data = await fetchData();
    return data;
  } catch (err) {
    console.error('Fetch failed:', err.message);
  }
}
```

**Rules:**
- Every Promise chain must end with a `.catch()` handler.
- Async functions that await Promises must use `try/catch` or be called within a `.catch()` chain.
- In Node.js 15+, unhandled rejections terminate the process by default.
- Attach `.catch()` handlers synchronously to avoid memory and file descriptor leaks.

**Constraints:**
- `unhandledRejection` event is deprecated in Node.js; use `make-promises-safe` to ensure unhandled rejections always throw.
- Promise callbacks cannot be cancelled once queued.

### Annotated Code Example

```javascript
// promise-states.js
// 1. Creating a Promise
const orderFood = new Promise((resolve, reject) => {
  const isAvailable = true;

  setTimeout(() => {
    if (isAvailable) {
      resolve({ dish: 'Pasta', price: 15 });  // Fulfilled
    } else {
      reject(new Error('Dish not available')); // Rejected
    }
  }, 1000);
});

// 2. Consuming the Promise
console.log('Order placed'); // Synchronous — runs first

orderFood
  .then(order => {
    console.log(`Got ${order.dish} for $${order.price}`); // Fulfilled handler
  })
  .catch(err => {
    console.error(`Order failed: ${err.message}`);        // Rejected handler
  })
  .finally(() => {
    console.log('Order process complete');                // Always runs
  });

console.log('Waiting for food...'); // Synchronous — runs second
```

**Expected Output:**
```
Order placed
Waiting for food...
Got Pasta for $15
Order process complete
```

**Why this output:** The synchronous `console.log` calls run first. After 1 second, the Promise resolves, and the `.then()` handler runs in the microtask queue. The `.finally()` handler runs after `.then()` regardless of outcome.

### Real-World Cases

- **HTTP requests:** `fetch()` returns a Promise that resolves with the response.
- **Database queries:** ORM methods like `User.findAll()` return Promises.
- **File I/O:** `fs.promises.readFile()` returns a Promise.
- **Timers:** Wrapping `setTimeout` in a Promise for `await`-able delays.

---

## Core Concept 2: Async Route Handlers

### Definitions

**Core Definition:** Async route handlers are Express route handler functions declared with the `async` keyword, allowing the use of `await` to write sequential, readable asynchronous code instead of nested callbacks or `.then()` chains.

**Technical Definition:** An async route handler is a function with the signature `async (req, res, next)` that returns a Promise. Inside the handler, `await` pauses execution until the awaited Promise settles, then resumes with the fulfilled value or throws the rejection reason. This enables writing database queries, API calls, and file operations in a linear, top-to-bottom style. In Express 5, if the returned Promise rejects, Express automatically calls `next(err)` with the rejection reason. In Express 4, the rejection is unhandled and crashes the process or leaves the request hanging.

**Beginner-Friendly Explanation:** An async route handler is like a chef who follows a recipe step by step. Instead of shouting "start boiling water, and when it's done, come back and tell me" (callback style), the chef simply writes "boil water, then add pasta, then drain" (async/await style). The chef doesn't do anything else while waiting for the water to boil — they just pause that step and resume when it's done.

### Purposes

- To write clean, sequential controller code using `async/await` syntax.
- To eliminate callback nesting and `.then()` chaining in route handlers.
- To make asynchronous error handling natural via `try/catch`.
- To enable non-blocking I/O while maintaining readable code structure.

### Syntax Rules and Structure

```javascript
// Async route handler
app.get('/users/:id', async (req, res) => {
  const user = await User.findById(req.params.id);  // Pause until resolved
  if (!user) {
    return res.status(404).json({ error: 'User not found' });
  }
  res.json({ data: user });
});
```

| Component | Breakdown |
|-----------|-----------|
| `async (req, res)` | Declares an async handler that returns a Promise. |
| `await User.findById()` | Pauses execution until the Promise settles. |
| `return res.status()` | Early return to exit the handler. |
| `res.json()` | Sends the response. |

**Rules:**
- Async handlers return a Promise — Express 5 automatically forwards rejections to `next(err)`.
- Use `await` for every asynchronous operation (database, API, file I/O).
- Early returns (`return res.status().json()`) prevent further execution.
- Avoid mixing callbacks and Promises in the same handler.

**Constraints:**
- In Express 4, async errors must be caught manually or wrapped — they are not caught automatically.
- Async handlers add a small overhead due to Promise allocation.
- Sequential `await` calls are slower than parallel `Promise.all()` when operations are independent.

### Annotated Code Example

```javascript
// async-route-handler.js
const express = require('express');
const app = express();

// Simulated async database
const db = {
  findUser: async (id) => {
    await new Promise(r => setTimeout(r, 100)); // Simulate latency
    return { id, name: 'Alice', email: 'alice@example.com' };
  },
  findPosts: async (userId) => {
    await new Promise(r => setTimeout(r, 100));
    return [{ id: 1, title: 'Post 1' }, { id: 2, title: 'Post 2' }];
  }
};

// Sequential async handler
app.get('/users/:id', async (req, res) => {
  console.log('1. Fetching user...');
  const user = await db.findUser(req.params.id);  // Pause here
  console.log('2. User fetched:', user.name);

  console.log('3. Fetching posts...');
  const posts = await db.findPosts(user.id);       // Pause here
  console.log('4. Posts fetched:', posts.length);

  res.json({ data: { user, posts } });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /users/1`):**
```
1. Fetching user...
2. User fetched: Alice
3. Fetching posts...
4. Posts fetched: 2

HTTP Response:
{
  "data": {
    "user": { "id": "1", "name": "Alice", "email": "alice@example.com" },
    "posts": [{ "id": 1, "title": "Post 1" }, { "id": 2, "title": "Post 2" }]
  }
}
```

**Why this output:** The handler executes sequentially. Each `await` pauses execution until the Promise resolves, then resumes with the value. The `console.log` statements appear in order because the awaits are sequential. The final `res.json()` sends both the user and posts in a single response.

### Real-World Cases

- **User profiles:** Awaiting user data, then their posts, then their comments.
- **E-commerce checkout:** Awaiting payment processing, then order creation, then inventory update.
- **Authentication:** Awaiting password verification, then token generation, then email sending.

---

## Core Concept 3: Try/Catch Block Mechanics

### Definitions

**Core Definition:** `try/catch` is a JavaScript control-flow construct that isolates a block of code (the `try` block) and catches any errors thrown within it (in the `catch` block), allowing the program to handle errors gracefully without crashing.

**Technical Definition:** In async functions, `await` throws the rejection reason as an exception when the awaited Promise rejects. This means `try/catch` can be used to handle async errors just like synchronous errors. Without `try/catch`, the rejection propagates up the async function call stack. In Express 4, this rejection is unhandled and crashes the process. In Express 5, the rejection is automatically forwarded to `next(err)`. The key principle is to let errors bubble up naturally: wrap only the code that can throw in `try/catch`, and use `next(err)` to pass the error to the centralised error handler rather than formatting responses inside the catch block.

**Beginner-Friendly Explanation:** `try/catch` is like a safety net under a trapeze artist. The trapeze artist (the `try` block) performs their routine. If they fall (an error is thrown), the safety net (the `catch` block) catches them. Without the net, the fall (error) crashes the whole show (process).

### Purposes

- To isolate execution blocks and catch errors locally when specific handling is needed.
- To avoid redundant nesting by letting errors bubble up naturally.
- To pass errors to the centralised error handler via `next(err)` instead of formatting responses inline.
- To distinguish between recoverable errors (handled in `catch`) and unrecoverable errors (passed to `next`).

### Syntax Rules and Structure

```javascript
// try/catch in async route handler
app.get('/users/:id', async (req, res, next) => {
  try {
    const user = await User.findById(req.params.id);
    if (!user) throw new NotFoundError('User');
    res.json({ data: user });
  } catch (err) {
    next(err); // Delegate to global error handler
  }
});
```

| Component | Breakdown |
|-----------|-----------|
| `try { ... }` | Contains code that may throw. |
| `catch (err) { ... }` | Receives the thrown error. |
| `next(err)` | Passes the error to the error-handling middleware. |

**Rules:**
- Wrap only the code that can throw — not the entire handler.
- Use `next(err)` in the catch block instead of formatting the response inline.
- Let errors bubble up naturally — avoid deeply nested `try/catch` blocks.
- The catch block should handle the error or delegate it, not both.

**Constraints:**
- Nested `try/catch` blocks can swallow errors if the inner catch does not re-throw.
- `try/catch` adds a small performance overhead but is negligible for I/O-bound operations.

### Annotated Code Example

```javascript
// try-catch-mechanics.js
const express = require('express');
const app = express();

// ❌ BAD: Nested try/catch that swallows errors
app.get('/bad/:id', async (req, res) => {
  try {
    try {
      const user = await findUser(req.params.id);
      if (!user) throw new Error('Not found');
    } catch (innerErr) {
      // Inner catch swallows the error — outer catch never sees it
      console.log('Inner caught:', innerErr.message);
    }
    // Execution continues even though user wasn't found
    res.json({ data: null }); // Wrong response!
  } catch (outerErr) {
    res.status(500).json({ error: outerErr.message });
  }
});

// ✅ GOOD: Single try/catch with next(err)
app.get('/good/:id', async (req, res, next) => {
  try {
    const user = await findUser(req.params.id);
    if (!user) throw new Error('User not found');
    res.json({ data: user });
  } catch (err) {
    next(err); // Bubble up to global handler
  }
});

// Global error handler
app.use((err, req, res, next) => {
  res.status(500).json({ error: err.message });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /bad/999`):**
```json
{"data": null}
```

**Expected Output (for `GET /good/999`):**
```json
{"error": "User not found"}
```

**Why this output:** In the bad example, the inner `catch` swallows the error, so the outer `catch` never runs, and the handler incorrectly returns `{ data: null }`. In the good example, the error is thrown and caught by the single `catch`, which delegates to `next(err)`. The global error handler sends the correct response.

### Real-World Cases

- **Validation errors:** Throwing `ValidationError` and letting the global handler format it.
- **Database errors:** Catching database connection failures and passing to `next(err)`.
- **Authentication:** Throwing `UnauthorizedError` when a token is invalid.

---

## Core Concept 4: Error Propagation & Express Differences

### Definitions

**Core Definition:** Error propagation in async Express handlers refers to how rejected Promises are detected and forwarded to the error-handling middleware. Express 5 handles this natively, while Express 4 requires manual handling via `try/catch`, wrapper functions, or the `express-async-errors` package.

**Technical Definition:** For errors returned from asynchronous functions invoked by route handlers and middleware, you must pass them to the `next()` function in Express 4, where Express will catch and process them. Starting with Express 5, route handlers and middleware that return a Promise will call `next(value)` automatically when they reject or throw an error. If no rejected value is provided, `next` will be called with a default Error object provided by the Express router. In Express 4, an unhandled rejection in async middleware crashes the process or leaves the request hanging.

**Beginner-Friendly Explanation:** Imagine a package delivery system. In Express 5, if a package gets lost (a Promise rejects), the system automatically notifies the claims department (error handler). In Express 4, the system doesn't notice — the package just disappears, and the customer waits forever. To fix this, you have to manually tell the system "hey, this package is lost" by calling `next(err)`.

### Purposes

- To manage the shift between Express 4 and Express 5 async error handling.
- To prevent unhandled Promise rejections from crashing the process.
- To provide a consistent error-handling pattern across Express versions.
- To understand when to use `try/catch`, `next(err)`, wrapper functions, or monkey-patching.

### Sub-Feature 4.1: Express 5 — Native Async Error Catching

#### Syntax Rules and Structure

```javascript
// Express 5 — This just works
app.get('/users/:id', async (req, res) => {
  const user = await User.findById(req.params.id); // If this rejects...
  if (!user) throw new NotFoundError('User');       // Or this throws...
  res.json({ data: user });
});
// Express 5 automatically calls next(err)
```

**Rules:**
- Async handlers returning a Promise automatically forward rejections to `next(err)`.
- No wrapper function or `try/catch` is required (though `try/catch` is still useful for custom handling).
- If no rejected value is provided, Express uses a default Error object.

---

### Sub-Feature 4.2: Express 4 — Manual Handling Required

#### Option 1: try/catch + next(err)

```javascript
app.get('/users/:id', async (req, res, next) => {
  try {
    const user = await User.findById(req.params.id);
    res.json({ data: user });
  } catch (err) {
    next(err); // Manual propagation
  }
});
```

#### Option 2: Async Wrapper

```javascript
// Define once
const asyncHandler = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};

// Use with async routes
app.get('/users/:id', asyncHandler(async (req, res) => {
  const user = await User.findById(req.params.id);
  res.json({ data: user });
}));
```

#### Option 3: express-async-errors

```javascript
// Import once at the top of the entry file
require('express-async-errors');

// Now async errors are caught automatically
app.get('/users/:id', async (req, res) => {
  const user = await User.findById(req.params.id);
  res.json({ data: user });
});
```

| Approach | Pros | Cons |
|----------|------|------|
| `try/catch` | Explicit, no dependencies. | Verbose; easy to forget. |
| Wrapper | Clean controllers, one-time setup. | Requires wrapping every handler. |
| `express-async-errors` | Zero code changes. | Monkey-patches Express internals. |

**Rules:**
- Express 4 does **not** catch async errors automatically.
- `express-async-errors` must be imported **before** any routes are defined.
- The wrapper pattern uses `Promise.resolve(fn(...)).catch(next)` to forward rejections.
- `try/catch` must call `next(err)` — merely logging the error is insufficient.

**Constraints:**
- `express-async-errors` patches `expressLayer.prototype.handle_request` to add async support.
- The wrapper adds a small overhead per request but is negligible.

### Annotated Code Example

```javascript
// express4-async-handler.js
const express = require('express');
const app = express();

// Async wrapper for Express 4
const asyncHandler = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};

// Simulated async operation that fails
const riskyOperation = async () => {
  throw new Error('Database connection failed');
};

// Express 4 — WRONG: Unhandled rejection crashes the process
app.get('/wrong', async (req, res) => {
  const result = await riskyOperation(); // Rejection unhandled!
  res.json({ result });
});

// Express 4 — CORRECT: Wrapper catches rejection
app.get('/correct', asyncHandler(async (req, res) => {
  const result = await riskyOperation();
  res.json({ result });
}));

// Global error handler
app.use((err, req, res, next) => {
  console.error('Error caught:', err.message);
  res.status(500).json({ error: err.message });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /wrong` — Express 4):**
```
(node:12345) UnhandledPromiseRejectionWarning: Error: Database connection failed
(node:12345) UnhandledPromiseRejectionWarning: Unhandled promise rejection...
(process crashes or request hangs)
```

**Expected Output (for `GET /correct` — Express 4):**
```
Error caught: Database connection failed

HTTP Response:
{"error":"Database connection failed"}
```

**Why this output:** In the `/wrong` route, the async handler rejects, but Express 4 doesn't catch it — the process logs an unhandled rejection warning and crashes (or the request hangs). In the `/correct` route, the `asyncHandler` wrapper catches the rejection via `.catch(next)` and forwards it to the global error handler.

### Real-World Cases

- **Express 4 legacy APIs:** Using `express-async-errors` to avoid wrapping every handler.
- **Express 5 migration:** Removing wrappers and `try/catch` blocks as Express 5 handles them natively.
- **Cross-version compatibility:** Using a wrapper that works on both Express 4 and 5.

---

## References

- Express.js 5.x — Error Handling Guide — https://expressjs.com/en/5x/guide/error-handling/
- Express.js 4.x — Error Handling Guide — https://expressjs.com/en/4x/guide/error-handling/
- Express 5 Migration Guide — https://expressjs.com/en/guide/migrating-5.html
- MDN — Promise — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise
- MDN — async function — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function
- MDN — await — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await
- Node.js — Errors (Promises) — https://nodejs.org/api/errors.html
- Node.js — Process Events (`unhandledRejection`) — https://nodejs.org/api/process.html
- Secrets of the JavaScript Ninja, Third Edition — Manning — https://www.manning.com/books/secrets-of-the-javascript-ninja-third-edition
- Steve Kinney — Type-Safe Async Error Handling in Express Routes — https://stevekinney.com/courses/full-stack-typescript/type-safe-asynchronous-route-handlers-with-express
- express-async-errors — npm — https://www.npmjs.com/package/express-async-errors
- js-async-handler — npm — https://www.npmjs.com/package/js-async-handler
- express-async-error-patch — npm — https://www.npmjs.com/package/express-async-error-patch
- make-promises-safe — npm — https://www.npmjs.com/package/make-promises-safe
- Stop writing boilerplate code! Async Handlers in Express Backend — Medium — https://medium.com/@shouryaupadhyaya79/stop-writing-boiler-plate-code-async-handlers-in-express-backend-3eac2974f186
- Node.js Best Practices — Error Handling — https://github.com/goldbergyoni/nodebestpractices
- Stack Overflow — Bubbling up error response with nested async/await try/catch — https://stackoverflow.com/questions/49956674/
- Stack Overflow — How to handle errors in Express async middleware — https://stackoverflow.com/questions/51391080/
- OWASP — Error Handling Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Error_Handling_Cheat_Sheet.html