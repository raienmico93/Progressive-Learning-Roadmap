# Callbacks — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A callback is a function passed as an argument to another function, which is then invoked inside that outer function to complete some kind of routine or action.

**Technical Definition:** Per MDN, "A callback function is a function passed into another function as an argument, which is then invoked inside the outer function to complete some kind of routine or action". Callbacks are JavaScript's foundational mechanism for asynchronous programming. They can be invoked either **synchronously** (immediately after the outer function is called) or **asynchronously** (later, after an asynchronous operation completes). In Node.js, asynchronous APIs follow the **error-first callback** convention, where the callback's first parameter is reserved for an error object (or `null` on success). Callbacks are the oldest and most fundamental pattern for handling asynchronous operations in JavaScript, and they underpin Promises, `async/await`, and event-driven programming.

**Beginner-Friendly Explanation:** A callback is a function you give to another function and say, "Call this when you're done." For example, when you click a button, the browser calls your event handler — that handler is a callback. When you read a file, you pass a callback that receives the file data. Callbacks let JavaScript do things asynchronously without blocking.

### Key Characteristics

- **First-class functions:** Callbacks are possible because JavaScript treats functions as values that can be passed around.
- **Synchronous or asynchronous:** Callbacks can be invoked immediately (e.g., `Array.prototype.map`) or later (e.g., `setTimeout`).
- **Error-first convention:** Node.js popularised the pattern where the first callback argument is an error object.
- **Inversion of control:** Passing a callback hands control to the API, which decides when to invoke it.
- **Prone to nesting:** Sequential asynchronous operations with callbacks lead to callback hell.
- **Foundation for Promises:** Promises and `async/await` are built on the callback mechanism.

### Prerequisites

- Understanding of JavaScript functions (declarations, expressions, arrow functions).
- Familiarity with scope, closures, and the call stack.
- Awareness of the event loop and asynchronous execution.

### Related Programming Areas

- **Asynchronous programming:** The primary use case for callbacks.
- **Event-driven programming:** Event listeners and handlers are callbacks.
- **Functional programming:** Higher-order functions accept callbacks.
- **Node.js:** Core APIs use error-first callbacks extensively.
- **Promises and async/await:** Built on top of the callback mechanism.

### Core Concepts / Features

---

## Callback Functions

### Definitions

**Core Definition:** A callback function is a function passed into another function as an argument, which is then invoked inside the outer function to complete some kind of routine or action.

**Technical Definition:** In ECMAScript, a callback is a function value passed as an argument to a `CallExpression`. The receiving function (the higher-order function) may invoke the callback zero or more times, either synchronously within its own execution or asynchronously after an operation completes. Per MDN, "the user of a callback-based API writes a function that is passed into the API. The provider of the API (called the caller) takes the function and calls back (or executes) the function at some point in the body of the caller".

**Beginner-Friendly Explanation:** A callback is a function you hand to another function and say, "Run this when you're ready." For example, `array.map(myFunction)` calls `myFunction` for each element. `setTimeout(myFunction, 1000)` calls `myFunction` after one second. In both cases, `myFunction` is the callback.

### Purposes

- To customise the behaviour of a function without modifying its source code.
- To handle asynchronous operations without blocking execution.
- To implement event-driven programming.
- To decouple the "what" (the callback) from the "when" (the higher-order function).

### Syntax Rules and Structure

```js
// Passing a named function:
function handleResult(result) {
  console.log(result);
}
asyncOperation(handleResult);

// Passing an anonymous function:
asyncOperation(function(result) {
  console.log(result);
});

// Passing an arrow function:
asyncOperation((result) => console.log(result));
```

**Component Breakdown:**
- `handleResult`: A function reference passed as a callback.
- `asyncOperation`: The higher-order function that receives the callback.
- `(result) => console.log(result)`: An inline arrow function callback.

**Syntax Rules:**
- Pass the function **reference** (e.g., `handleResult`), not the result of calling it (e.g., `handleResult()`).
- Callbacks can be named, anonymous, or arrow functions.
- The outer function decides when and how many times to call the callback.

**Constraints and Limitations:**
- Callbacks can lead to deeply nested code (callback hell).
- Error handling requires explicit checks in each callback.
- Callbacks invert control: the API decides when to call your function.

### Annotated Code Examples

#### Example 1: Synchronous Callback with Array.map

```js
// Step 1: Define an array.
const numbers = [1, 2, 3, 4];

// Step 2: Pass a callback to map().
const doubled = numbers.map(function(n) {
  return n * 2;
});

console.log(doubled); // [2, 4, 6, 8]
```

**Expected Output:**
```
[ 2, 4, 6, 8 ]
```

**Why This Output:** `map()` calls the callback for each element, passing the element as `n`. The callback returns `n * 2`, and `map` collects the results into a new array.

#### Example 2: Asynchronous Callback with setTimeout

```js
// Step 1: Define a callback.
function greet() {
  console.log("Hello after 1 second!");
}

// Step 2: Pass the callback to setTimeout.
setTimeout(greet, 1000);

// Step 3: This runs first.
console.log("This runs first");
```

**Expected Output:**
```
This runs first
Hello after 1 second!
```

**Why This Output:** `setTimeout` schedules `greet` to run after 1000ms and returns immediately. The synchronous code (`console.log("This runs first")`) executes first. After one second, the event loop invokes `greet`.

### Real-World Cases

- **Event listeners:** `button.addEventListener("click", handleClick)`.
- **Array methods:** `[1, 2, 3].map(n => n * 2)`.
- **Node.js file I/O:** `fs.readFile(path, (err, data) => { ... })`.
- **Timers:** `setTimeout(callback, delay)`.

---

## Callback Patterns

### Definitions

**Core Definition:** Callback patterns are recurring conventions and structures for using callbacks effectively, including error-first callbacks, named callbacks, and modularisation techniques.

**Technical Definition:** Callback patterns are design conventions that address common issues in callback-based asynchronous programming. The most important pattern is the **error-first callback** (Node.js style), where the callback's first argument is always an error object or `null`. Other patterns include using **named functions** (instead of anonymous functions) for better stack traces and reusability, and **modularisation** (extracting callbacks into separate functions) to avoid deep nesting.

**Beginner-Friendly Explanation:** Callback patterns are the "best practices" for using callbacks. The most important one is the error-first pattern: always put the error as the first argument of your callback. Another is to use named functions instead of anonymous ones, so your stack traces are readable. And if you find yourself nesting callbacks deeply, break them into separate functions.

### Purposes

- To ensure consistent error handling across asynchronous APIs.
- To improve code readability and maintainability.
- To make stack traces more informative for debugging.
- To reduce nesting and avoid callback hell.

### Syntax Rules and Structure

```js
// Error-first callback pattern:
fs.readFile(path, (err, data) => {
  if (err) {
    // Handle error
    return;
  }
  // Use data
});

// Named callback pattern:
function handleFileRead(err, data) {
  if (err) {
    console.error(err);
    return;
  }
  console.log(data);
}
fs.readFile(path, handleFileRead);
```

**Component Breakdown:**
- `(err, data) => {}`: Error-first callback — `err` is the first parameter.
- `handleFileRead`: A named function, reusable and identifiable in stack traces.

**Syntax Rules:**
- The callback is always the last parameter passed to an asynchronous API.
- The first parameter of the callback is reserved for errors.
- If no error occurred, the first parameter is `null`.

**Constraints and Limitations:**
- Error-first callbacks require explicit error checking in each callback.
- Named callbacks can clutter the namespace if not scoped properly.

### Annotated Code Examples

#### Example 1: Error-First Callback Pattern

```js
const fs = require("node:fs");

// Step 1: Read a file with an error-first callback.
fs.readFile("/path/to/file.csv", "utf8", function(err, data) {
  // Step 2: Always check for errors first.
  if (err) {
    console.log("There was an error", err);
    return; // Important: return to prevent success logic from running
  }
  // Step 3: If no error, use the data.
  console.log(data);
});
```

**Expected Output:** The file contents are logged, or an error message if the file does not exist.

**Why This Output:** The callback receives `err` as the first argument. If `err` is truthy (an error occurred), the error is handled and the function returns. Otherwise, `data` is valid and is logged.

#### Example 2: Named Callback for Reusability

```js
// Step 1: Define a named callback.
function handleRead(err, data) {
  if (err) {
    console.error("Error:", err.message);
    return;
  }
  console.log("Data:", data.length, "bytes");
}

// Step 2: Use it with multiple calls.
fs.readFile("/file1.txt", handleRead);
fs.readFile("/file2.txt", handleRead);
```

**Expected Output:** The byte length of each file is logged, or an error message.

**Why This Output:** The named callback `handleRead` is reused for both file reads. It is identifiable in stack traces and avoids duplicating the error-handling logic.

### Real-World Cases

- **Node.js core modules:** `fs`, `http`, `net`, and `child_process` all use error-first callbacks.
- **Express.js:** Middleware functions use a similar pattern with `(req, res, next)`.
- **Database drivers:** `mysql`, `pg`, and `mongodb` use callbacks for query results.

---

## Error-First Callbacks

### Definitions

**Core Definition:** An error-first callback (also called a Node.js-style callback) is a callback convention where the first parameter is reserved for an error object, and subsequent parameters contain the result data.

**Technical Definition:** Per the Node.js documentation, "Most asynchronous methods exposed by the Node.js core API follow an idiomatic pattern referred to as an error-first callback. With this pattern, a callback function is passed to the method as an argument. When the operation either completes or an error is raised, the callback function is called with the `Error` object (if any) passed as the first argument". The pattern requires: (1) the callback is the last parameter passed to the asynchronous API, and (2) the first parameter in the callback is reserved for errors.

**Beginner-Friendly Explanation:** Error-first callbacks are a simple rule: the first argument of any callback is always the error. If something went wrong, that argument contains an error object. If everything succeeded, it's `null`. The actual data is passed in the second argument. This convention makes error handling consistent and predictable.

### Purposes

- To ensure errors are always handled explicitly.
- To provide a consistent convention across all asynchronous APIs.
- To make error handling predictable and uniform.
- To separate error information from result data.

### Syntax Rules and Structure

```js
function callback(err, result) {
  if (err) {
    // Handle error
    return;
  }
  // Use result
}

asyncOperation(arg1, arg2, callback);
```

**Component Breakdown:**
- `err`: The first parameter — an `Error` object or `null`.
- `result`: The second parameter — the successful result.
- `callback`: Always the last argument to the asynchronous function.

**Syntax Rules:**
- The callback is always the **last** argument to the asynchronous function.
- The first callback parameter is always the error (or `null`).
- Always check `err` before using `result`.
- Use `return` after handling an error to prevent success logic from executing.

**Constraints and Limitations:**
- Requires explicit error checking in every callback.
- Does not work well with synchronous `try...catch` (errors occur asynchronously).
- Easily forgotten, leading to silent failures.

### Annotated Code Examples

#### Example 1: Standard Error-First Callback

```js
const fs = require("node:fs");

// Step 1: Call fs.readFile with an error-first callback.
fs.readFile("/file.json", (err, data) => {
  // Step 2: Check for error.
  if (err) {
    console.error("Failed to read file:", err.message);
    return;
  }
  // Step 3: Process the data.
  console.log("File contents:", data.toString());
});
```

**Expected Output:**
```
File contents: { ... } (or an error message)
```

**Why This Output:** If the file exists, `err` is `null` and `data` contains the file contents. If the file is missing, `err` contains a `Error` object with a `code` property (e.g., `ENOENT`).

#### Example 2: Error-First Callback with Multiple Results

```js
function divide(a, b, callback) {
  if (b === 0) {
    callback(new Error("Division by zero"), null);
  } else {
    callback(null, a / b);
  }
}

// Call with valid arguments:
divide(10, 2, (err, result) => {
  if (err) return console.error(err.message);
  console.log("Result:", result); // 5
});

// Call with invalid arguments:
divide(10, 0, (err, result) => {
  if (err) return console.error(err.message); // "Division by zero"
  console.log("Result:", result);
});
```

**Expected Output:**
```
Result: 5
Division by zero
```

**Why This Output:** On success, `err` is `null` and `result` contains the quotient. On failure, `err` contains an `Error` object and `result` is `null`.

### Real-World Cases

- **File system:** `fs.readFile`, `fs.writeFile`, `fs.mkdir`.
- **Network requests:** `http.get`, `https.request`.
- **Database queries:** `db.query`, `collection.find`.
- **Child processes:** `child_process.exec`, `child_process.spawn`.

---

## Callback Hell

### Definitions

**Core Definition:** Callback hell (also called the "pyramid of doom") is an anti-pattern in asynchronous programming where deeply nested callbacks make code difficult to read, understand, and maintain.

**Technical Definition:** Callback hell occurs when multiple asynchronous operations depend on each other, leading to deeply nested callback functions. Per the reference, "Callback Hell, also known as Pyramid of Doom, is an anti-pattern seen in code of asynchronous programming. It is a slang term used to describe and unwieldy number of nested 'if' statements or functions". The code takes on a triangular or pyramid-like shape due to increasing indentation, and error handling becomes repetitive and error-prone at each level.

**Beginner-Friendly Explanation:** Callback hell is what happens when you need to do several asynchronous things in a row, and each one has to wait for the previous one to finish. You end up with a bunch of nested functions that look like a pyramid, making the code hard to read and fix.

### Purposes

- To recognise and avoid the anti-pattern in asynchronous code.
- To motivate the use of Promises and `async/await`.
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
      fourthOperation(result3, (err, result4) => {
        if (err) return handleError(err);
        // ... and so on
      });
    });
  });
});
```

**Component Breakdown:**
- Each nested callback depends on the result of the previous one.
- Error handling is repeated at each level.
- The indentation grows with each nested operation.

**Syntax Rules:**
- Callback hell is not a syntax error; it is a code smell.
- It arises when callbacks are used for sequential asynchronous operations.
- Promises and `async/await` are the primary solutions.

**Constraints and Limitations:**
- Deeply nested code is hard to read and maintain.
- Error handling becomes repetitive and error-prone.
- The structure does not compose well.

### Annotated Code Examples

#### Example 1: Callback Hell (Pyramid of Doom)

```js
getUser(userId, (err, user) => {
  if (err) return console.error(err);
  getPosts(user.id, (err, posts) => {
    if (err) return console.error(err);
    getComments(posts[0].id, (err, comments) => {
      if (err) return console.error(err);
      getReplies(comments[0].id, (err, replies) => {
        if (err) return console.error(err);
        console.log(replies);
      });
    });
  });
});
```

**Expected Output:** The replies are logged after all four nested operations complete.

**Why This Output:** Each operation depends on the result of the previous one, creating a pyramid of nested callbacks.

#### Example 2: Refactored with Promises

```js
getUser(userId)
  .then((user) => getPosts(user.id))
  .then((posts) => getComments(posts[0].id))
  .then((comments) => getReplies(comments[0].id))
  .then((replies) => console.log(replies))
  .catch((err) => console.error(err));
```

**Expected Output:** The same result, but with flat, readable code and centralised error handling.

**Why This Output:** Promises allow chaining, eliminating nesting and consolidating error handling into a single `.catch()`.

### Real-World Cases

- **Legacy Node.js code:** Older codebases often contain callback hell.
- **Third-party APIs:** Some libraries still use callbacks and require careful wrapping.
- **Refactoring projects:** Converting callback-based code to promises is a common task.

---

## Callback Pyramid

### Definitions

**Core Definition:** The callback pyramid (also called the pyramid of doom) is the visual shape created by deeply nested callbacks, where each level of nesting adds another layer of indentation.

**Technical Definition:** The pyramid of doom is the structural manifestation of callback hell. Per the reference, "The 'Pyramid of Doom' refers to a code pattern where multiple levels of nested conditional statements or callbacks create a triangular-shaped indentation pattern that resembles a pyramid". The creeping lexical scope in the pyramid is due to inner functions being written inside outer callbacks, which can be mitigated by moving inner functions outside to minimise scope.

**Beginner-Friendly Explanation:** The pyramid of doom is what callback hell looks like on screen — a triangle of nested functions that keeps shifting further to the right. The deeper you go, the harder it is to read and understand.

### Purposes

- To visually identify callback hell in code.
- To motivate refactoring into flatter structures.
- To understand the relationship between nesting and readability.

### Syntax Rules and Structure

```js
// The pyramid shape:
step1((err, r1) => {
  if (err) return handle(err);
  step2(r1, (err, r2) => {
    if (err) return handle(err);
    step3(r2, (err, r3) => {
      if (err) return handle(err);
      // ...
    });
  });
});
```

**Component Breakdown:**
- Each nested function adds one level of indentation.
- The resulting shape is a triangle or pyramid.

**Syntax Rules:**
- The pyramid is not a syntax error; it is a readability problem.
- Refactoring techniques include named functions, modularisation, and Promises.

**Constraints and Limitations:**
- The pyramid can make code reviews and debugging difficult.
- The deeper the pyramid, the harder it is to trace error paths.

### Annotated Code Examples

#### Example 1: Pyramid of Doom

```js
step1((err, r1) => {
  if (err) return handle(err);
  step2(r1, (err, r2) => {
    if (err) return handle(err);
    step3(r2, (err, r3) => {
      if (err) return handle(err);
      step4(r3, (err, r4) => {
        if (err) return handle(err);
        console.log(r4);
      });
    });
  });
});
```

**Expected Output:** The final result is logged after all four steps complete.

**Why This Output:** Each step is nested inside the previous one's callback, creating the pyramid shape.

#### Example 2: Refactored with Named Functions

```js
function handleStep4(err, r4) {
  if (err) return handle(err);
  console.log(r4);
}

function handleStep3(err, r3) {
  if (err) return handle(err);
  step4(r3, handleStep4);
}

function handleStep2(err, r2) {
  if (err) return handle(err);
  step3(r2, handleStep3);
}

function handleStep1(err, r1) {
  if (err) return handle(err);
  step2(r1, handleStep2);
}

step1(handleStep1);
```

**Expected Output:** The same result, but with flat, modular code.

**Why This Output:** Extracting each callback into a named function eliminates nesting and makes the flow explicit.

### Real-World Cases

- **Legacy codebases:** Pyramid of doom is common in older Node.js code.
- **Code reviews:** Recognising the pyramid helps reviewers suggest refactoring.
- **Refactoring:** Breaking the pyramid into named functions or Promises.

---

## Callback Alternatives

### Definitions

**Core Definition:** Callback alternatives are modern JavaScript features — Promises and `async/await` — that address the limitations of callbacks, particularly callback hell and inversion of control.

**Technical Definition:** Per MDN, Promises were introduced as a solution to the problems of callbacks. "Essentially, a promise is an object returned to which callbacks are attached, instead of passing callbacks into a function". `async/await` is syntactic sugar built on top of Promises, allowing asynchronous code to be written in a synchronous style. Per Toptal, "Promises were the next logical step in escaping callback hell", and `async/await` represents the modern solution.

**Beginner-Friendly Explanation:** Callbacks are the old way of handling asynchronous code. Promises and `async/await` are the modern alternatives. A Promise is like a receipt — it represents a future value. `async/await` lets you write asynchronous code that looks like regular synchronous code, making it much easier to read. Most modern JavaScript code uses Promises and `async/await` instead of callbacks.

### Purposes

- To eliminate callback hell through flat, chainable structures.
- To centralise error handling with `.catch()` or `try...catch`.
- To improve readability and maintainability of asynchronous code.
- To enable composition of asynchronous operations (`Promise.all`, `Promise.race`).
- To un-invert control by returning a result object.

### Syntax Rules and Structure

```js
// Callback:
fs.readFile("/file", (err, data) => {
  if (err) return handleError(err);
  console.log(data);
});

// Promise:
fs.promises.readFile("/file")
  .then((data) => console.log(data))
  .catch((err) => handleError(err));

// Async/await:
async function readFile() {
  try {
    const data = await fs.promises.readFile("/file");
    console.log(data);
  } catch (err) {
    handleError(err);
  }
}
```

**Component Breakdown:**
- **Callback:** Function passed as an argument; error-first convention.
- **Promise:** Object returned; `.then()` and `.catch()` handlers attached.
- **Async/await:** `async` function with `await` expressions; `try...catch` for errors.

**Syntax Rules:**
- Promises are returned by modern APIs (`fetch`, `fs.promises`).
- `await` can only be used inside `async` functions (or top-level modules).
- Callback-based APIs can be converted using `util.promisify()`.

**Constraints and Limitations:**
- Promises cannot be cancelled (though `AbortController` helps).
- `async/await` is built on Promises; it does not replace them.
- Legacy callback APIs still exist and require wrapping.

### Annotated Code Examples

#### Example 1: Converting Callbacks to Promises

```js
const util = require("node:util");
const fs = require("node:fs");

// Step 1: Promisify the callback-based function.
const readFileAsync = util.promisify(fs.readFile);

// Step 2: Use the promise-based version.
readFileAsync("/file.txt", "utf8")
  .then((data) => console.log(data))
  .catch((err) => console.error(err));
```

**Expected Output:** The file contents are logged, or an error message if the file is missing.

**Why This Output:** `util.promisify` converts the error-first callback API into a promise-returning function, enabling `.then()` and `.catch()` usage.

#### Example 2: Async/Await for Sequential Operations

```js
async function processUser(userId) {
  try {
    const user = await getUser(userId);
    const posts = await getPosts(user.id);
    const comments = await getComments(posts[0].id);
    return comments;
  } catch (err) {
    console.error("Error:", err.message);
  }
}
```

**Expected Output:** The comments are returned after all three operations complete.

**Why This Output:** Each `await` pauses the async function until the promise settles. The code reads like synchronous code, and errors are handled by a single `try...catch`.

### Real-World Cases

- **Fetch API:** Built on Promises, not callbacks.
- **Modern Node.js:** `fs.promises`, `stream/promises`.
- **React:** Uses `async/await` in data fetching and effects.
- **TypeScript:** Modern async code uses Promises and `async/await`.

---

## Callback Best Practices

### Definitions

**Core Definition:** Callback best practices are guidelines for using callbacks effectively, including always checking for errors, using named functions, keeping callbacks small, and avoiding deep nesting.

**Technical Definition:** Per the best practices checklist, "Prefer named callbacks for readability and reuse; Keep callback functions small and single-purpose; In Node.js, follow the error-first pattern consistently; Ensure callbacks are called exactly once in all code paths; Avoid deep nesting: extract functions or switch to Promises/async". Additionally, "Don't use callbacks in new code. Callbacks are an old way of handling async operations. They still exist in older code and some APIs, but you should never write new code using callbacks. Use promises and async/await instead".

**Beginner-Friendly Explanation:** Best practices are the rules of thumb for using callbacks correctly. Always check for errors first. Use named functions instead of anonymous ones. Keep callbacks small and focused. Don't nest callbacks deeply. And for new code, use Promises and `async/await` instead of callbacks.

### Purposes

- To ensure errors are handled consistently.
- To improve code readability and maintainability.
- To make debugging easier with named functions and shallow nesting.
- To migrate away from callbacks in favour of Promises and `async/await`.

### Syntax Rules and Structure

```js
// BEST PRACTICE: Error-first with early return and named function
function handleFile(err, data) {
  if (err) {
    console.error("Error reading file:", err.message);
    return;
  }
  processData(data);
}

fs.readFile(path, handleFile);

// BAD PRACTICE: Anonymous, nested, error-checking after success logic
fs.readFile(path, (err, data) => {
  console.log(data); // Success logic before error check!
  if (err) console.error(err);
});
```

**Component Breakdown:**
- **Error-first:** Always check `err` before using `data`.
- **Early return:** Return immediately after handling an error.
- **Named functions:** Use named callbacks for better stack traces.
- **Small callbacks:** Each callback should do one thing.

**Syntax Rules:**
- Always check the `err` parameter first.
- Use `return` after error handling to avoid executing success logic.
- Prefer named functions over anonymous functions.
- Avoid nesting callbacks more than 2–3 levels deep.

**Constraints and Limitations:**
- Best practices are guidelines, not strict rules.
- Legacy APIs may require callbacks, so understanding them remains necessary.

### Annotated Code Examples

#### Example 1: Good vs Bad Callback

```js
// GOOD: Error-first, early return, named function.
function handleRead(err, data) {
  if (err) {
    console.error("Read failed:", err.message);
    return;
  }
  console.log("Read succeeded:", data.length, "bytes");
}
fs.readFile("/file.txt", handleRead);

// BAD: Anonymous, no error check, success logic first.
fs.readFile("/file.txt", (err, data) => {
  console.log(data); // Crashes if err is set and data is undefined!
  if (err) console.error(err);
});
```

**Expected Output:**
```
Read succeeded: 1024 bytes (good version)
(possibly a TypeError from the bad version)
```

**Why This Output:** The good version checks `err` first and returns early. The bad version uses `data` before checking `err`, which can cause a `TypeError` if the read failed.

#### Example 2: Avoiding Deep Nesting

```js
// BAD: Deeply nested callbacks.
getUser(userId, (err, user) => {
  if (err) return handle(err);
  getPosts(user.id, (err, posts) => {
    if (err) return handle(err);
    getComments(posts[0].id, (err, comments) => {
      if (err) return handle(err);
      console.log(comments);
    });
  });
});

// GOOD: Refactored with named functions.
function handleComments(err, comments) {
  if (err) return handle(err);
  console.log(comments);
}

function handlePosts(err, posts) {
  if (err) return handle(err);
  getComments(posts[0].id, handleComments);
}

function handleUser(err, user) {
  if (err) return handle(err);
  getPosts(user.id, handlePosts);
}

getUser(userId, handleUser);
```

**Expected Output:** The same result, but with flat, readable code.

**Why This Output:** Extracting each callback into a named function eliminates nesting and makes the flow explicit.

### Real-World Cases

- **Node.js core modules:** All follow error-first conventions.
- **Express.js middleware:** Uses `(req, res, next)` with error handling.
- **Legacy codebases:** Applying best practices during refactoring.
- **Code reviews:** Checking for error-first, early returns, and shallow nesting.

---

## References

- MDN Web Docs – Callback function — https://developer.mozilla.org/en-US/docs/Glossary/Callback_function
- MDN Web Docs – Introducing asynchronous JavaScript — https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Async_JS/Introducing
- Node.js Documentation – Error-first callbacks — https://nodejs.org/api/errors.html
- Oracle – Using the Callback Pattern and the Async Module — https://asktom.oracle.com/Misc/oramag/using-the-callback-pattern-and-the-async-module.html
- Syncfusion – Handling Concurrent Requests with JavaScript Callbacks — https://www.syncfusion.com/blogs/post/handling-concurrent-requests-with-javascript-callbacks
- Toptal – Asynchronous JavaScript: From Callback Hell to Async and Await — https://www.toptal.com
- DigitalOcean – Managing Asynchronous Code in JavaScript — https://www.digitalocean.com
- OWASP – NodeJS Security Cheat Sheet (Callback Hell) — https://cheatsheetseries.owasp.org
- Packt – Taming Asynchronous Control Flow in Node.js: From Callback Hell to Elegant Patterns — https://www.packtpub.com
- TSecurity – Understanding Callback Functions in JavaScript — https://tsecurity.de
- Scaler Topics – What is Callback Hell in JavaScript? — https://www.scaler.com
- Stack Overflow – Pyramid of Doom — https://stackoverflow.com
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) — https://ecma-international.org/wp-content/uploads/ECMA-262_16th_edition_june_2025.pdf