# Event Loop — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** The event loop is the runtime mechanism that allows JavaScript to perform non-blocking asynchronous operations by coordinating the call stack, the task queue, and the microtask queue, despite being single-threaded.

**Technical Definition:** Per MDN, "JavaScript has a runtime model based on an **event loop**, which is responsible for executing the code, collecting and processing events, and executing queued sub-tasks". The event loop continuously checks if the call stack is empty; if so, it takes the first message from the queue and processes it, creating a new call stack frame for the associated function. This process repeats indefinitely. In Node.js, the event loop is implemented by the libuv library and operates in multiple phases.

**Beginner-Friendly Explanation:** The event loop is like a manager who keeps checking: "Is the current task done? Yes? Then take the next task from the queue and run it." It ensures that JavaScript can handle many tasks without getting stuck on any one of them. Even though JavaScript can only do one thing at a time, the event loop makes it *appear* to do many things simultaneously by switching between tasks quickly.

### Key Characteristics

- **Single-threaded:** JavaScript executes one piece of code at a time; there is no true multi-threading in the main execution context.
- **Non-blocking:** Asynchronous operations return immediately, allowing the program to continue.
- **Run-to-completion:** Each message is processed completely before the next message.
- **Two queue classes:** Task queue (macrotasks) and microtask queue.
- **Microtasks have priority:** All microtasks are drained after each task and before the next macrotask.
- **Node.js phases:** The Node.js event loop has six main phases: timers, pending callbacks, idle/prepare, poll, check, and close callbacks.

### Prerequisites

- Understanding of JavaScript functions, callbacks, and Promises.
- Familiarity with the call stack and execution contexts.
- Basic knowledge of asynchronous programming concepts.

### Related Programming Areas

- **Asynchronous programming:** Callbacks, Promises, `async/await`.
- **Browser APIs:** `setTimeout`, `setInterval`, `requestAnimationFrame`, `requestIdleCallback`.
- **Node.js APIs:** `setImmediate`, `process.nextTick`, I/O operations.
- **Performance optimisation:** Non-blocking I/O, worker threads, task scheduling.

### Core Concepts / Features

---

## Event Loop

### Definitions

**Core Definition:** The event loop is the mechanism that orchestrates the execution of JavaScript code, events, and queued sub-tasks in a single-threaded runtime.

**Technical Definition:** Per MDN, "JavaScript has a runtime model based on an **event loop**, which is responsible for executing the code, collecting and processing events, and executing queued sub-tasks". The event loop operates on a simple principle: it waits for messages, processes them one at a time, and repeats. The pseudo-code is:

```js
while (queue.waitForMessage()) {
  queue.processNextMessage();
}
```

Per MDN, "`queue.waitForMessage()` synchronously waits for a message to arrive if one is not currently present".

**Beginner-Friendly Explanation:** The event loop is the "heart" of JavaScript's asynchronous behaviour. It keeps checking if there's any work to do (like a callback from a timer or a network request) and runs it when the call stack is empty.

### Purposes

- To enable asynchronous, non-blocking execution in a single-threaded language.
- To coordinate the execution of callbacks, promises, and timers.
- To ensure that the call stack is never blocked by long-running operations.
- To provide a predictable execution model for JavaScript programs.

### Syntax Rules and Structure

The event loop is not a syntax construct; it is a runtime mechanism. The conceptual model is:

```
Call Stack (empty?) → Task Queue (dequeue) → Execute Task → Drain Microtask Queue → Render → Repeat
```

**Component Breakdown:**
- **Call stack:** Tracks the currently executing function.
- **Task queue (macrotask queue):** Holds messages (callbacks) waiting to be processed. Includes: initial script execution, `setTimeout`/`setInterval` callbacks, event dispatch, I/O.
- **Microtask queue:** Holds promise callbacks, `async/await` continuations, and `queueMicrotask()` callbacks.

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

#### Example 2: Microtask Interleaving with `await`

```js
let i = 0;
queueMicrotask(function test() {
  i++;
  console.log("microtask", i);
  if (i < 3) queueMicrotask(test);
});

(async () => {
  console.log("async function start");
  for (let i = 1; i < 3; i++) {
    await null;
    console.log("async function resume", i);
  }
  await null;
  console.log("async function end");
})();

queueMicrotask(() => console.log("queueMicrotask() after calling async function"));
console.log("script sync part end");
```

**Expected Output:**
```
async function start
script sync part end
microtask 1
async function resume 1
queueMicrotask() after calling async function
microtask 2
async function resume 2
microtask 3
async function end
```

**Why This Output:** Each `await null` re-enters the microtask queue, interleaving with other pending microtasks.

### Real-World Cases

- **UI responsiveness:** Event loop blocking causes unresponsive interfaces.
- **Server throughput:** Non-blocking I/O allows servers to handle many requests.
- **Animation:** `requestAnimationFrame` schedules work before the next repaint.

---

## Call Stack

### Definitions

**Core Definition:** The call stack is a mechanism for an interpreter to keep track of its place in a script that calls multiple functions — what function is currently being run and what functions are called from within that function.

**Technical Definition:** Per MDN, "A **call stack** is a mechanism for an interpreter (like the JavaScript interpreter in a web browser) to keep track of its place in a script that calls multiple functions". When a script calls a function, the interpreter adds it to the call stack and then starts carrying out the function. Any functions called by that function are added to the call stack further up, and run where their calls are reached. When a function is finished, the interpreter takes it off the stack and resumes execution where it left off.

**Beginner-Friendly Explanation:** The call stack is like a stack of sticky notes. Each time you call a function, you put a new note on top. When the function finishes, you remove the note and go back to what you were doing.

### Purposes

- To track the execution order of function calls.
- To manage the return addresses for function calls.
- To detect stack overflow errors from infinite recursion.
- To provide the execution context for each function.

### Syntax Rules and Structure

The call stack is a runtime mechanism. Example:

```js
function foo(b) {
  const a = 10;
  return a + b + 11;
}

function bar(x) {
  const y = 3;
  return foo(x * y);
}

const baz = bar(7); // assigns 42 to baz
```

**Component Breakdown:**
- When `bar(7)` is called, a frame is created for `bar`.
- When `bar` calls `foo`, a second frame is pushed on top.
- When `foo` returns, its frame is popped.
- When `bar` returns, the stack is empty.

**Syntax Rules:**
- Each function call creates a new stack frame.
- The stack grows and shrinks as functions are called and return.
- If the stack exceeds its allocated space, a "stack overflow" error occurs.

**Constraints and Limitations:**
- Deep recursion can overflow the call stack.
- Blocking the call stack blocks the event loop.

### Annotated Code Examples

#### Example 1: Call Stack Trace

```js
function first() {
  console.log("First function");
  second();
}

function second() {
  console.log("Second function");
  third();
}

function third() {
  console.log("Third function");
}

first();
```

**Expected Output:**
```
First function
Second function
Third function
```

**Why This Output:** `first()` is called, pushing its frame. It calls `second()`, pushing another frame. `second()` calls `third()`, pushing a third frame. Each logs its message, then returns, popping frames in reverse order.

### Real-World Cases

- **Debugging:** Stack traces show the call stack at the point of an error.
- **Recursion:** Understanding the call stack is essential for recursive algorithms.
- **Stack overflow:** Infinite recursion causes a `RangeError: Maximum call stack size exceeded`.

---

## Task Queue

### Definitions

**Core Definition:** The task queue (also called the macrotask queue) is a queue of tasks (callbacks) waiting to be processed by the event loop.

**Technical Definition:** Per the WHATWG specification, the task queue is a queue of tasks. Each task is a unit of work that the event loop processes completely before moving to the next task. Tasks are added to the task queue by mechanisms such as `setTimeout`, `setInterval`, `setImmediate` (Node.js), event dispatch, and I/O operations.

**Beginner-Friendly Explanation:** The task queue is like a to-do list. The event loop picks the oldest item, does it completely, then picks the next one. It never starts a new task until the current one is completely finished.

### Purposes

- To hold callbacks that are ready to be executed by the event loop.
- To ensure that tasks are processed in the order they were added (FIFO).
- To provide a mechanism for scheduling asynchronous work.

### Syntax Rules and Structure

Tasks are added to the task queue via APIs like `setTimeout`. Example:

```js
setTimeout(() => {
  // This callback is a task
  console.log("Task executed");
}, 0);
```

**Component Breakdown:**
- `setTimeout`: Schedules a task after a minimum delay.
- The callback is added to the task queue when the delay expires.
- The event loop dequeues and executes the task.

**Syntax Rules:**
- Only one task is processed per event loop iteration.
- Tasks queued during a task wait for the next iteration.
- The task queue is a FIFO queue.

**Constraints and Limitations:**
- Tasks cannot be prioritised (except via `scheduler.postTask`).
- Long tasks block the event loop.

### Annotated Code Examples

#### Example 1: Task Queue Order

```js
setTimeout(() => console.log("Task 1"), 0);
setTimeout(() => console.log("Task 2"), 0);
setTimeout(() => console.log("Task 3"), 0);

console.log("Synchronous");
```

**Expected Output:**
```
Synchronous
Task 1
Task 2
Task 3
```

**Why This Output:** The synchronous `console.log` runs first. Then the event loop processes the three tasks in the order they were queued.

### Real-World Cases

- **Timers:** `setTimeout` and `setInterval` schedule tasks.
- **Event handling:** User events (clicks, keypresses) dispatch tasks.
- **I/O:** Network responses and file reads schedule tasks.

---

## Microtask Queue

### Definitions

**Core Definition:** The microtask queue is a queue of microtasks — short functions that are executed after the current task completes and before the event loop processes the next task.

**Technical Definition:** Per MDN, "A **microtask** is a short function which is executed after the function or program which created it exits _and_ only if the JavaScript execution stack is empty, but before returning control to the event loop". The microtask queue includes Promise `.then`/`.catch`/`.finally` callbacks, `async/await` continuations, and `queueMicrotask()` callbacks.

**Beginner-Friendly Explanation:** The microtask queue is like a "priority lane" for small tasks. After each regular task, JavaScript processes **all** microtasks before moving on to the next regular task. This is why Promises run before `setTimeout`.

### Purposes

- To provide a high-priority queue for short, critical tasks.
- To ensure that Promise callbacks run before the next macrotask.
- To allow libraries and frameworks to schedule microtasks safely.

### Syntax Rules and Structure

Microtasks are added via `queueMicrotask()`, Promise callbacks, or `await`. Example:

```js
Promise.resolve().then(() => {
  console.log("Microtask 1");
});

queueMicrotask(() => {
  console.log("Microtask 2");
});

console.log("Synchronous");
```

**Component Breakdown:**
- `Promise.resolve().then()`: Adds a microtask.
- `queueMicrotask()`: Adds a microtask.
- All microtasks are drained after the current task.

**Syntax Rules:**
- All microtasks are processed before the next macrotask.
- Microtasks added during microtask processing are also processed.
- The microtask queue is fully drained before rendering or the next task.

**Constraints and Limitations:**
- Recursive microtask scheduling can starve the event loop (starvation).
- Microtasks cannot be cancelled.

### Annotated Code Examples

#### Example 1: Microtask vs Macrotask

```js
setTimeout(() => console.log("Macrotask"), 0);
Promise.resolve().then(() => console.log("Microtask 1"));
Promise.resolve().then(() => console.log("Microtask 2"));

console.log("Synchronous");
```

**Expected Output:**
```
Synchronous
Microtask 1
Microtask 2
Macrotask
```

**Why This Output:** All microtasks run before the macrotask, even though the `setTimeout` was queued first.

### Real-World Cases

- **Promise resolution:** `.then()` callbacks are microtasks.
- **`async/await`:** Each `await` continuation is a microtask.
- **MutationObserver:** Uses microtasks to batch DOM changes.

---

## Macrotask Queue

### Definitions

**Core Definition:** The macrotask queue is another name for the task queue — it holds macrotasks (larger units of work) that are processed one per event loop iteration.

**Technical Definition:** Per the reference, "**Task queue (macrotasks):** initial script execution, `setTimeout`/`setInterval` callbacks, event dispatch, I/O". Macrotasks are distinct from microtasks in that only one macrotask is processed per event loop iteration, whereas all microtasks are processed after each macrotask.

**Beginner-Friendly Explanation:** Macrotasks are the "regular" tasks. They include timers, events, and I/O. The event loop processes one macrotask, then all microtasks, then the next macrotask.

### Purposes

- To schedule regular asynchronous work.
- To differentiate between high-priority (microtasks) and normal-priority (macrotasks) work.
- To control the order of execution for timers, events, and I/O.

### Syntax Rules and Structure

Macrotasks are scheduled via `setTimeout`, `setInterval`, `setImmediate` (Node.js), and event handlers. Example:

```js
setTimeout(() => console.log("Macrotask 1"), 0);
setTimeout(() => console.log("Macrotask 2"), 0);
```

**Component Breakdown:**
- Each `setTimeout` callback is a macrotask.
- Macrotasks are processed one per event loop iteration.

**Syntax Rules:**
- One macrotask per iteration.
- All microtasks are drained between macrotasks.
- Rendering occurs between macrotasks (in browsers).

**Constraints and Limitations:**
- Macrotasks have lower priority than microtasks.
- Long macrotasks block the event loop.

### Annotated Code Examples

#### Example 1: Macrotask Order with Microtasks

```js
setTimeout(() => console.log("Macrotask 1"), 0);
Promise.resolve().then(() => console.log("Microtask"));
setTimeout(() => console.log("Macrotask 2"), 0);
```

**Expected Output:**
```
Microtask
Macrotask 1
Macrotask 2
```

**Why This Output:** The microtask runs after the current task completes but before the first macrotask.

### Real-World Cases

- **Timers:** `setTimeout` and `setInterval`.
- **I/O callbacks:** File reads, network responses.
- **User events:** Clicks, keypresses.

---

## Rendering

### Definitions

**Core Definition:** Rendering is the browser process of updating the visual display based on the current state of the DOM and CSSOM.

**Technical Definition:** Per the MDN event loop model, rendering occurs between macrotasks in browsers. The browser may choose to update rendering after processing microtasks. The rendering pipeline includes style calculation, layout, paint, and composite. `requestAnimationFrame` callbacks run before the next repaint.

**Beginner-Friendly Explanation:** Rendering is when the browser actually draws the page on your screen. It happens between tasks in the event loop, usually about 60 times per second (60fps).

### Purposes

- To update the visual display after DOM changes.
- To provide smooth animations and transitions.
- To ensure the UI reflects the current application state.

### Syntax Rules and Structure

Rendering is not directly controlled by JavaScript; it is managed by the browser. However, `requestAnimationFrame` allows scheduling work before rendering:

```js
function animate() {
  // Update animation state
  requestAnimationFrame(animate);
}
requestAnimationFrame(animate);
```

**Component Breakdown:**
- `requestAnimationFrame`: Schedules a callback before the next repaint.
- The browser renders after processing microtasks and before the next macrotask.

**Syntax Rules:**
- Rendering happens between macrotasks.
- Microtasks can prevent rendering if they run indefinitely.
- `requestAnimationFrame` is the preferred way to schedule animations.

**Constraints and Limitations:**
- Rendering is skipped if the tab is hidden.
- Long-running tasks delay rendering.

### Annotated Code Examples

#### Example 1: Simple Animation

```js
let position = 0;
function animate() {
  position += 1;
  document.getElementById("box").style.transform = `translateX(${position}px)`;
  if (position < 100) {
    requestAnimationFrame(animate);
  }
}
requestAnimationFrame(animate);
```

**Expected Output:** The box moves 100 pixels to the right over several frames.

**Why This Output:** Each `requestAnimationFrame` callback updates the position and requests the next frame. The browser renders the updated position before the next repaint.

### Real-World Cases

- **Animations:** Smooth movement, fades, and transitions.
- **Games:** Updating game state and rendering frames.
- **Data visualisation:** Charts and graphs that update in real time.

---

## `setTimeout()`

### Definitions

**Core Definition:** `setTimeout()` is a global function that schedules a callback to be executed after a specified delay (in milliseconds).

**Technical Definition:** Per MDN, "The global `setTimeout()` method sets a timer which executes a function or specified piece of code once the timer expires". The returned `timeoutID` is a positive integer that identifies the timer and can be passed to `clearTimeout()` to cancel it.

**Beginner-Friendly Explanation:** `setTimeout()` lets you say "run this function after X milliseconds." It doesn't guarantee exact timing — it guarantees that the function will run no sooner than the specified delay.

### Purposes

- To delay the execution of a function.
- To schedule asynchronous work after a minimum delay.
- To break up long-running tasks into smaller chunks.

### Syntax Rules and Structure

```js
setTimeout(functionRef, delay, param1, param2, ...);
```

**Component Breakdown:**
- `functionRef`: The function to execute after the delay.
- `delay` (optional): Time in milliseconds. Defaults to `0`.
- `param1, param2, ...` (optional): Additional parameters passed to `functionRef`.

**Syntax Rules:**
- The delay is a minimum, not a guarantee.
- `setTimeout` returns a `timeoutID` that can be used with `clearTimeout()`.
- Nested `setTimeout` calls are clamped to a minimum of 4ms after 5 levels of nesting.

**Constraints and Limitations:**
- `setTimeout(fn, 0)` does not execute immediately; it waits for the next event loop iteration.
- The delay is coerced to a signed 32-bit integer.

### Annotated Code Examples

#### Example 1: Basic `setTimeout`

```js
setTimeout(() => {
  console.log("Executed after 1 second");
}, 1000);
```

**Expected Output (after 1 second):**
```
Executed after 1 second
```

**Why This Output:** The callback is scheduled and executed after 1000ms.

#### Example 2: Passing Arguments

```js
function greet(name, greeting) {
  console.log(`${greeting}, ${name}!`);
}

setTimeout(greet, 500, "Alice", "Hello");
```

**Expected Output (after 500ms):**
```
Hello, Alice!
```

**Why This Output:** The additional arguments are passed to the callback.

### Real-World Cases

- **Debouncing:** Delaying a function until the user stops typing.
- **Polling:** Periodically checking for updates.
- **Animation delays:** Sequencing animations.

---

## `setInterval()`

### Definitions

**Core Definition:** `setInterval()` is a global function that repeatedly calls a function with a fixed time delay between each call.

**Technical Definition:** Per MDN, "The `setInterval()` method... repeatedly calls a function or executes a code snippet, with a fixed time delay between each call". It returns an `intervalID` that can be passed to `clearInterval()` to cancel the interval.

**Beginner-Friendly Explanation:** `setInterval()` is like `setTimeout()`, but it repeats. It keeps calling your function every X milliseconds until you stop it with `clearInterval()`.

### Purposes

- To execute a function repeatedly at fixed intervals.
- To implement polling and periodic updates.
- To create repeating animations or timers.

### Syntax Rules and Structure

```js
setInterval(func, delay, arg1, arg2, ...);
```

**Component Breakdown:**
- `func`: The function to execute repeatedly.
- `delay`: Time in milliseconds between executions.
- `arg1, arg2, ...`: Additional arguments passed to `func`.

**Syntax Rules:**
- The delay is a minimum, not a guarantee.
- `setInterval` returns an `intervalID` for `clearInterval()`.
- `setInterval` and `setTimeout` share the same ID pool.

**Constraints and Limitations:**
- If the callback takes longer than the delay, calls may overlap or be skipped.
- The delay is clamped to 2147483647ms (about 24.8 days).

### Annotated Code Examples

#### Example 1: Basic `setInterval`

```js
let count = 0;
const intervalId = setInterval(() => {
  count++;
  console.log(`Interval fired ${count} time(s)`);
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

**Why This Output:** The interval fires every second. After 3 fires, `clearInterval` stops it.

### Real-World Cases

- **Clocks:** Updating a digital clock every second.
- **Polling:** Checking a server for updates every 5 seconds.
- **Animations:** Simple repeating animations.

---

## `clearTimeout()`

### Definitions

**Core Definition:** `clearTimeout()` cancels a timeout previously established by calling `setTimeout()`.

**Technical Definition:** Per MDN, `clearTimeout()` uses the same list of entries as `clearInterval()` and can technically clear either, but for consistency you should use `clearTimeout()` for `setTimeout()` entries.

**Beginner-Friendly Explanation:** `clearTimeout()` stops a timer you previously set with `setTimeout()`. You pass it the ID that `setTimeout()` returned.

### Purposes

- To cancel a scheduled timeout before it fires.
- To prevent a callback from executing.
- To manage and clean up timers.

### Syntax Rules and Structure

```js
clearTimeout(timeoutID);
```

**Component Breakdown:**
- `timeoutID`: The ID returned by `setTimeout()`.

**Syntax Rules:**
- The ID must be the value returned by `setTimeout()`.
- If the timeout has already fired, `clearTimeout()` has no effect.

### Annotated Code Examples

#### Example 1: Cancelling a Timeout

```js
const timeoutId = setTimeout(() => {
  console.log("This will not be printed");
}, 1000);

clearTimeout(timeoutId);
console.log("Timeout cancelled");
```

**Expected Output:**
```
Timeout cancelled
```

**Why This Output:** `clearTimeout` cancels the timeout before it fires.

### Real-World Cases

- **Debouncing:** Cancelling a previous timeout when a new event occurs.
- **User interaction:** Cancelling a timeout when the user performs an action.

---

## `clearInterval()`

### Definitions

**Core Definition:** `clearInterval()` cancels a repeating interval previously established by calling `setInterval()`.

**Technical Definition:** Per MDN, "The `clearInterval()` method cancels a timed, repeating action which was previously established by a call to `setInterval()`". It shares the same ID pool as `clearTimeout()`.

**Beginner-Friendly Explanation:** `clearInterval()` stops a repeating timer you set with `setInterval()`. You pass it the ID that `setInterval()` returned.

### Purposes

- To stop a repeating interval.
- To prevent further executions of an interval callback.
- To clean up timers when they are no longer needed.

### Syntax Rules and Structure

```js
clearInterval(intervalID);
```

**Component Breakdown:**
- `intervalID`: The ID returned by `setInterval()`.

**Syntax Rules:**
- The ID must be the value returned by `setInterval()`.
- If the interval has already been cleared, `clearInterval()` has no effect.

### Annotated Code Examples

#### Example 1: Stopping an Interval

```js
let count = 0;
const intervalId = setInterval(() => {
  count++;
  console.log(`Count: ${count}`);
  if (count === 3) {
    clearInterval(intervalId);
  }
}, 1000);
```

**Expected Output:**
```
Count: 1
Count: 2
Count: 3
```

**Why This Output:** The interval is cleared after the third execution.

### Real-World Cases

- **Polling:** Stopping a poll when the desired data is received.
- **Animations:** Stopping a repeating animation.
- **Timers:** Stopping a countdown timer.

---

## `setImmediate()`

### Definitions

**Core Definition:** `setImmediate()` is a Node.js function that schedules a callback to be executed in the check phase of the event loop, after I/O events.

**Technical Definition:** Per Node.js, "Any function passed as the `setImmediate()` argument is a callback that's executed in the next iteration of the event loop". It is executed in the **check** phase, which runs after the **poll** phase.

**Beginner-Friendly Explanation:** `setImmediate()` is a Node.js-specific function that runs your callback after I/O events have been processed. It's like saying "do this as soon as possible, but after I/O."

### Purposes

- To execute a callback after I/O events in the current event loop iteration.
- To schedule work without blocking I/O.
- To avoid starvation of I/O (unlike `process.nextTick`).

### Syntax Rules and Structure

```js
setImmediate(callback, ...args);
```

**Component Breakdown:**
- `callback`: The function to execute.
- `...args`: Additional arguments passed to the callback.

**Syntax Rules:**
- `setImmediate` is only available in Node.js.
- It runs in the check phase of the event loop.
- It returns an `Immediate` object that can be used with `clearImmediate()`.

**Constraints and Limitations:**
- Not available in browsers.
- The execution order relative to `setTimeout(fn, 0)` can vary.

### Annotated Code Examples

#### Example 1: `setImmediate` vs `setTimeout`

```js
setTimeout(() => console.log("setTimeout"), 0);
setImmediate(() => console.log("setImmediate"));
```

**Expected Output (varies):**
```
setTimeout
setImmediate
```
or
```
setImmediate
setTimeout
```

**Why This Output:** The order depends on the context. Inside an I/O callback, `setImmediate` always runs first.

### Real-World Cases

- **I/O-heavy applications:** Scheduling work after I/O events.
- **Recursive operations:** Using `setImmediate` instead of `process.nextTick` to avoid starvation.

---

## `queueMicrotask()`

### Definitions

**Core Definition:** `queueMicrotask()` is a global function that adds a microtask to the microtask queue for execution after the current task completes.

**Technical Definition:** Per MDN, "The `queueMicrotask()` method... queues a microtask to be executed at a safe time before control is returned to the browser's event loop". A microtask is a short function that runs after the current task's code finishes and before the event loop continues.

**Beginner-Friendly Explanation:** `queueMicrotask()` lets you schedule a small task to run right after the current code finishes, but before anything else. It's like saying "run this immediately after the current function, before any timers or rendering."

### Purposes

- To schedule a microtask explicitly.
- To allow libraries and frameworks to use microtasks.
- To defer work without using Promises.

### Syntax Rules and Structure

```js
queueMicrotask(callback);
```

**Component Breakdown:**
- `callback`: The function to execute as a microtask.

**Syntax Rules:**
- The callback runs after the current task, before the next macrotask.
- Microtasks are processed before rendering.
- Recursive `queueMicrotask` calls can starve the event loop.

### Annotated Code Examples

#### Example 1: Basic `queueMicrotask`

```js
queueMicrotask(() => {
  console.log("Microtask executed");
});
console.log("Synchronous");
```

**Expected Output:**
```
Synchronous
Microtask executed
```

**Why This Output:** The microtask runs after the synchronous code completes.

### Real-World Cases

- **Promise polyfills:** Implementing microtask semantics without native Promises.
- **Batching:** Deferring DOM updates to avoid multiple reflows.

---

## `process.nextTick()`

### Definitions

**Core Definition:** `process.nextTick()` is a Node.js function that adds a callback to the "next tick queue", which is processed after the current operation completes but before the event loop continues.

**Technical Definition:** Per Node.js, "`process.nextTick()` adds `callback` to the 'next tick queue'. This queue is fully drained after the current operation on the JavaScript stack runs to completion and before the event loop is allowed to continue". It is **not** technically part of the event loop; it runs before the event loop continues.

**Beginner-Friendly Explanation:** `process.nextTick()` runs your callback immediately after the current function finishes, before anything else. It's even faster than microtasks (Promises).

### Purposes

- To execute a callback immediately after the current operation.
- To ensure consistency in API behaviour (always sync or always async).
- To allow users to assign event handlers after object construction.

### Syntax Rules and Structure

```js
process.nextTick(callback, ...args);
```

**Component Breakdown:**
- `callback`: The function to execute.
- `...args`: Additional arguments.

**Syntax Rules:**
- `process.nextTick` is only available in Node.js.
- The next tick queue is processed before the microtask queue.
- Recursive `process.nextTick` calls can starve I/O.

**Constraints and Limitations:**
- Recursive use can starve the event loop.
- Node.js has a built-in protection (`process.maxTickDepth`) but it is deprecated.

### Annotated Code Examples

#### Example 1: `process.nextTick` vs Promise

```js
process.nextTick(() => console.log("nextTick"));
Promise.resolve().then(() => console.log("Promise"));
console.log("Synchronous");
```

**Expected Output:**
```
Synchronous
nextTick
Promise
```

**Why This Output:** The next tick queue is processed before the microtask queue.

### Real-World Cases

- **API consistency:** Ensuring callbacks are always asynchronous.
- **Event emitter:** Emitting events after the current operation.

---

## `requestAnimationFrame()`

### Definitions

**Core Definition:** `requestAnimationFrame()` is a browser function that schedules a callback to run before the next repaint, typically matching the display's refresh rate.

**Technical Definition:** Per MDN, "The `window.requestAnimationFrame()` method tells the browser you wish to perform an animation. It requests the browser to call a user-supplied callback function before the next repaint". The callback receives a timestamp indicating the end of the previous frame's rendering.

**Beginner-Friendly Explanation:** `requestAnimationFrame()` is the best way to do animations in the browser. It tells the browser "call my function right before you draw the next frame." This ensures smooth animations at the display's refresh rate (usually 60fps).

### Purposes

- To create smooth, performant animations.
- To synchronise updates with the browser's rendering cycle.
- To avoid unnecessary work when the tab is hidden.

### Syntax Rules and Structure

```js
requestAnimationFrame(callback);
```

**Component Breakdown:**
- `callback`: The function to call before the next repaint. Receives a `DOMHighResTimeStamp`.

**Syntax Rules:**
- The callback must call `requestAnimationFrame` again to continue the animation.
- It runs before the next repaint, after microtasks.
- It is paused when the tab is hidden.

**Constraints and Limitations:**
- Only one callback per frame (you must request the next frame).
- The timestamp is the same for all callbacks in the same frame.

### Annotated Code Examples

#### Example 1: Basic Animation

```js
function animate(timestamp) {
  // Use timestamp to calculate progress
  console.log(`Frame at ${timestamp}ms`);
  requestAnimationFrame(animate);
}
requestAnimationFrame(animate);
```

**Expected Output:** Frame timestamps are logged at approximately 60fps.

**Why This Output:** The callback runs before each repaint, receiving the timestamp of the previous frame.

### Real-World Cases

- **Games:** Updating game state and rendering.
- **Animations:** Smooth transitions and movements.
- **Data visualisation:** Real-time charts.

---

## `requestIdleCallback()`

### Definitions

**Core Definition:** `requestIdleCallback()` is a browser function that schedules a callback to run during the browser's idle periods.

**Technical Definition:** Per MDN, "The `window.requestIdleCallback()` method queues a function to be called during a browser's idle periods. This enables developers to perform background and low priority work on the main thread". The callback receives an `IdleDeadline` object that describes how much time is available.

**Beginner-Friendly Explanation:** `requestIdleCallback()` lets you run non-urgent tasks when the browser isn't busy. It's like saying "do this when you have nothing else to do."

### Purposes

- To perform low-priority work without affecting responsiveness.
- To run background tasks during idle periods.
- To avoid blocking animations and user input.

### Syntax Rules and Structure

```js
requestIdleCallback(callback, options);
```

**Component Breakdown:**
- `callback`: The function to run during idle time. Receives an `IdleDeadline`.
- `options` (optional): An object with a `timeout` property.

**Syntax Rules:**
- The callback runs during idle periods.
- A `timeout` can force execution after a certain time.
- It is not Baseline; support varies across browsers.

**Constraints and Limitations:**
- Not available in all browsers.
- Callbacks may be delayed significantly if the browser is busy.

### Annotated Code Examples

#### Example 1: Idle Callback

```js
requestIdleCallback((deadline) => {
  while (deadline.timeRemaining() > 0) {
    // Do some work
  }
});
```

**Expected Output:** Work is performed during idle time.

**Why This Output:** The callback runs when the browser is idle and uses `deadline.timeRemaining()` to check how much time is available.

### Real-World Cases

- **Analytics:** Sending non-critical data to a server.
- **Preloading:** Fetching resources for the next page.
- **Cleanup:** Removing unused DOM elements.

---

## Event Loop Phases (Node.js)

### Definitions

**Core Definition:** The Node.js event loop operates in six main phases: timers, pending callbacks, idle/prepare, poll, check, and close callbacks.

**Technical Definition:** Per Node.js, "Each box will be referred to as a 'phase' of the event loop. Each phase has a FIFO queue of callbacks to execute". The phases are executed in order: timers → pending callbacks → idle, prepare → poll → check → close callbacks.

**Beginner-Friendly Explanation:** The Node.js event loop goes through a series of stages, like a washing machine cycle. Each stage handles a specific type of callback.

### Purposes

- To provide a structured, deterministic order for processing asynchronous operations.
- To differentiate between timers, I/O, and immediate callbacks.
- To allow fine-grained control over callback scheduling.

### Syntax Rules and Structure

The phases are not directly controlled by JavaScript code; they are managed by libuv. The order is:

1. **Timers:** Executes callbacks scheduled by `setTimeout()` and `setInterval()`.
2. **Pending callbacks:** Executes I/O callbacks deferred to the next loop iteration.
3. **Idle, prepare:** Only used internally.
4. **Poll:** Retrieves new I/O events; executes I/O related callbacks.
5. **Check:** Executes `setImmediate()` callbacks.
6. **Close callbacks:** Executes close callbacks like `socket.on('close')`.

**Component Breakdown:**
- Each phase has its own FIFO queue.
- After each phase, `process.nextTick` and microtasks are drained.

**Syntax Rules:**
- The event loop iterates through the phases in order.
- Each phase processes its queue until exhausted or the callback limit is reached.
- `process.nextTick` and microtasks run between phases.

**Constraints and Limitations:**
- Long-running callbacks in any phase delay the entire loop.
- The order of `setTimeout(fn, 0)` vs `setImmediate` depends on the context.

### Annotated Code Examples

#### Example 1: Phase Order

```js
const fs = require("node:fs");

fs.readFile(__filename, () => {
  setTimeout(() => console.log("setTimeout"), 0);
  setImmediate(() => console.log("setImmediate"));
});
```

**Expected Output:**
```
setImmediate
setTimeout
```

**Why This Output:** Inside an I/O callback, `setImmediate` always runs before `setTimeout` because the check phase runs before the timers phase in the next iteration.

### Real-World Cases

- **I/O-heavy servers:** Understanding phases helps optimise I/O handling.
- **Timers:** Knowing when timers fire relative to I/O.
- **Debugging:** Tracing callback execution order.

---

## Starvation

### Definitions

**Core Definition:** Starvation is a condition where a type of callback is indefinitely prevented from running because the event loop is continuously processing other types of callbacks.

**Technical Definition:** Per the reference, "`process.nextTick`... runs the risk of 'starving the event loop', by preventing evaluation of jobs on the Job Queue". Similarly, recursively scheduling microtasks (e.g., via `Promise.then` or `queueMicrotask`) can starve macrotasks and rendering.

**Beginner-Friendly Explanation:** Starvation happens when one type of task keeps getting delayed because the event loop is always busy with another type. For example, if you keep scheduling microtasks, the event loop never gets to timers or rendering.

### Purposes

- To understand why certain callbacks may never run.
- To avoid writing code that starves I/O or rendering.
- To design fairer scheduling mechanisms.

### Syntax Rules and Structure

Starvation occurs when callbacks recursively schedule more of the same type:

```js
function starve() {
  queueMicrotask(starve);
}
starve(); // This will starve the event loop
```

**Component Breakdown:**
- Each microtask schedules another microtask.
- The microtask queue never empties.
- Macrotasks and rendering never run.

**Syntax Rules:**
- Recursive microtask scheduling causes starvation.
- Recursive `process.nextTick` calls cause starvation.
- Recursive `setTimeout` with 0 delay can also cause starvation (but less aggressively).

**Constraints and Limitations:**
- Node.js has a built-in `process.maxTickDepth` (deprecated) to limit starvation.
- The browser has no built-in protection.

### Annotated Code Examples

#### Example 1: Microtask Starvation

```js
function starve() {
  Promise.resolve().then(starve);
}
starve();

setTimeout(() => console.log("This will never run"), 0);
```

**Expected Output:** The `setTimeout` callback never runs because the microtask queue never empties.

**Why This Output:** Each microtask schedules another microtask, preventing the event loop from moving to the next macrotask.

### Real-World Cases

- **Recursive promise chains:** Accidentally starving the event loop.
- **`process.nextTick` loops:** Blocking I/O in Node.js.
- **Infinite microtask loops:** Freezing the browser.

---

## Blocking the Event Loop

### Definitions

**Core Definition:** Blocking the event loop means executing a long-running synchronous operation that prevents the event loop from processing other tasks.

**Technical Definition:** Per the reference, "Any JavaScript code that takes too long to return back control to the event loop will block the execution of any JavaScript code in the page, even block the UI thread". Blocking can be caused by CPU-intensive loops, synchronous I/O, or long-running callbacks.

**Beginner-Friendly Explanation:** Blocking the event loop is like standing in a doorway and not letting anyone through. While you're blocking, no other tasks can run — not even rendering or user input.

### Purposes

- To understand why the UI freezes during long operations.
- To recognise blocking code and avoid it.
- To learn techniques for breaking up long tasks.

### Syntax Rules and Structure

Blocking occurs with synchronous, long-running code:

```js
// This blocks the event loop
const start = Date.now();
while (Date.now() - start < 5000) {
  // Busy wait for 5 seconds
}
console.log("Done blocking");
```

**Component Breakdown:**
- The `while` loop runs synchronously for 5 seconds.
- No other JavaScript can run during this time.
- Rendering and user input are frozen.

**Syntax Rules:**
- Any synchronous code that takes too long blocks the event loop.
- Synchronous I/O (`fs.readFileSync`) blocks the event loop.
- CPU-intensive algorithms block the event loop.

**Constraints and Limitations:**
- There is no way to "pause" blocking code.
- The only solution is to break the code into smaller chunks.

### Annotated Code Examples

#### Example 1: Blocking with a Loop

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

**Why This Output:** The `while` loop blocks the event loop for 2 seconds. The `setTimeout` callback cannot run until the blocking code finishes.

### Real-World Cases

- **UI freezes:** Long computations block rendering.
- **Server unresponsiveness:** Synchronous I/O blocks other requests.
- **Animation jank:** Long tasks cause dropped frames.

---

## Event Loop Best Practices

### Definitions

**Core Definition:** Event loop best practices are guidelines for writing code that keeps the event loop responsive and efficient.

**Technical Definition:** Per the Node.js best practices, "Critical rule: NEVER block the event loop. Blocking operations (CPU-heavy, synchronous I/O) stall ALL concurrent requests". Best practices include: avoiding synchronous I/O, breaking up long tasks, using `setImmediate` instead of `process.nextTick` for recursion, and using `scheduler.yield()` to yield to the main thread.

**Beginner-Friendly Explanation:** The golden rule is: don't block the event loop. Keep your tasks small and let the event loop breathe. Use asynchronous APIs instead of synchronous ones, and break big tasks into smaller pieces.

### Purposes

- To keep the UI responsive.
- To maximise server throughput.
- To avoid starvation and blocking.
- To write efficient, scalable asynchronous code.

### Syntax Rules and Structure

Best practices in practice:

```js
// GOOD: Non-blocking I/O
await fs.promises.readFile("file.txt");

// BAD: Blocking I/O
fs.readFileSync("file.txt");

// GOOD: Break up long tasks
async function processItems(items) {
  for (const item of items) {
    await processItem(item);
    await scheduler.yield(); // Yield to main thread
  }
}

// GOOD: Use setImmediate for recursive ops (not nextTick)
function recursiveOperation() {
  // ... work ...
  setImmediate(recursiveOperation);
}
```

**Component Breakdown:**
- **Avoid blocking:** Use async APIs.
- **Break up tasks:** Use `scheduler.yield()`, `setTimeout`, or `setImmediate`.
- **Use `setImmediate` for recursion:** Avoid `process.nextTick` for recursive operations.
- **Monitor performance:** Use DevTools to find long tasks.

**Syntax Rules:**
- Never use synchronous I/O in performance-critical code.
- Break long tasks into chunks smaller than 50ms.
- Use `setImmediate` for recursive operations in Node.js.

**Constraints and Limitations:**
- Some operations are inherently blocking (e.g., heavy computation).
- Worker threads are the solution for CPU-intensive work.

### Annotated Code Examples

#### Example 1: Breaking Up a Long Task

```js
async function processLargeArray(items) {
  for (let i = 0; i < items.length; i++) {
    // Process one item
    processItem(items[i]);

    // Yield to the main thread every 100 items
    if (i % 100 === 0) {
      await scheduler.yield();
    }
  }
}
```

**Expected Output:** The UI remains responsive while processing the array.

**Why This Output:** The function yields to the main thread periodically, allowing the browser to handle rendering and user input.

#### Example 2: Avoiding Synchronous I/O

```js
// BAD: Blocks the event loop
const data = fs.readFileSync("large-file.txt");

// GOOD: Non-blocking
const data = await fs.promises.readFile("large-file.txt");
```

**Expected Output:** The asynchronous version allows other operations to proceed.

**Why This Output:** `readFileSync` blocks until the file is read. `fs.promises.readFile` returns a Promise and does not block the event loop.

### Real-World Cases

- **Web servers:** Non-blocking I/O allows handling many concurrent requests.
- **Single-page applications:** Breaking up tasks keeps the UI responsive.
- **Games:** Using `requestAnimationFrame` for smooth animations.

---

## References

- MDN Web Docs – Event Loop — https://mdn.org.cn/en-US/docs/Web/JavaScript/Event_loop
- Node.js Documentation – The Node.js Event Loop — https://nodejs.org/learn/asynchronous-work/event-loop-timers-and-nexttick
- MDN Web Docs – Call stack — https://developer.mozilla.org/fr/docs/Glossary/Call_stack
- MDN Web Docs – setTimeout() — https://mdn.org.cn/en-US/docs/Web/API/setTimeout
- MDN Web Docs – setInterval() — https://mdn.org.cn/en-US/docs/Web/API/setInterval
- MDN Web Docs – queueMicrotask() — https://mdn.org.cn/en-US/docs/Web/API/HTML_DOM_API/Microtask_guide
- MDN Web Docs – requestAnimationFrame() — https://mdn.org.cn/en-US/docs/Web/API/Window/requestAnimationFrame
- MDN Web Docs – requestIdleCallback() — https://mdn.org.cn/en-US/docs/Web/API/Window/requestIdleCallback
- Node.js Documentation – setImmediate() — https://raw.githubusercontent.com/nodejs/nodejs.org/2e7f65b67a8b43e4b62afd07da2f6a3360c4809b/pages/en/learn/asynchronous-work/understanding-setimmediate.md
- Node.js Documentation – process.nextTick() — https://url.nodejs.cn/api/v26/process/process_nexttick_callback_args.html
- Node.js Documentation – Event Loop Phases — https://nodejs.org/learn/asynchronous-work/event-loop-timers-and-nexttick
- Event-Loop Ordering Trace Patterns — https://raw.githubusercontent.com/VincentChuWaiChow/vanguard-frontier-agentic/refs/heads/master/skills/frontend/javascript-runtime-async-review/references/event-loop-tracing.md
- Chrome for Developers – Using scheduler.yield() — https://developer.chrome.com/blog/use-scheduler-yield
- Node.js Best Practices – Event Loop Gotchas — https://raw.githubusercontent.com/9tykeshav/mern-ninja-cc/main/skills/code-reviewer/reference/nodejs.md
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) — https://ecma-international.org/wp-content/uploads/ECMA-262_16th_edition_june_2025.pdf