# JavaScript Call Stack — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** The call stack is a mechanism used by the JavaScript interpreter to keep track of its place in a script that calls multiple functions — specifically, what function is currently being run and what functions are called from within that function.

**Technical Definition:** Per ECMA-262, active execution contexts logically form a stack. The top execution context on this logical stack is the running execution context. A new execution context is created whenever control is transferred from the executable code associated with the currently running execution context to executable code that is not associated with that execution context. Transition of the running execution context status among execution contexts usually occurs in a stack-like last-in/first-out manner. However, some ECMAScript features (introduced in ES6) require non-LIFO transitions of the running execution context. The call stack is a LIFO (Last In, First Out) data structure that manages these execution contexts during synchronous code execution.

**Beginner-Friendly Explanation:** The call stack is like a stack of plates. When you call a function, JavaScript puts a "plate" (called a stack frame) on top of the stack. If that function calls another function, another plate goes on top. When a function finishes, its plate is removed. The function at the top of the stack is the one currently running. This is how JavaScript keeps track of where it is in your code and knows where to return after a function completes.

### Key Characteristics

- **LIFO structure:** The last function called is the first to finish.
- **Single-threaded:** JavaScript has one call stack, so only one function runs at a time.
- **Frame-based:** Each function call creates a stack frame containing arguments and local variables.
- **Automatic management:** Frames are pushed on call and popped on return, automatically.
- **Limited size:** The stack has a finite capacity; exceeding it throws a stack overflow error.
- **Synchronous only:** The call stack handles synchronous code; asynchronous callbacks are managed by the event loop.
- **Specification device:** The call stack is a specification mechanism; ECMAScript code cannot directly access or observe execution contexts.

### Prerequisites

- Basic JavaScript syntax (functions, variables, function calls).
- Understanding of execution contexts (global, function).
- Familiarity with the event loop and asynchronous JavaScript.
- Basic knowledge of recursion.
- Awareness of error handling and the `Error` object.

### Related Programming Areas

- **Execution contexts:** The call stack manages execution contexts.
- **Event loop:** Asynchronous callbacks are queued and executed when the stack is empty.
- **Recursion:** Each recursive call pushes a new frame.
- **Error handling:** Stack traces are generated from the call stack.
- **Debugging:** Debuggers use the call stack to show the current execution path.
- **Performance:** Deep call stacks consume memory and can cause overflows.

### Core Concepts / Features

---

## Call Stack

### Definitions

**Core Definition:** The call stack is a mechanism for an interpreter to keep track of its place in a script that calls multiple functions — what function is currently being run and what functions are called from within that function.

**Technical Definition:** Per ECMA-262, active execution contexts logically form a stack. The top execution context on this logical stack is the running execution context. A new execution context is created whenever control is transferred from the executable code associated with the currently running execution context to executable code that is not associated with that execution context. The call stack is a LIFO data structure that manages these contexts during synchronous execution.

**Beginner-Friendly Explanation:** The call stack is JavaScript's "to-do list" for function calls. When you call a function, it goes on top of the stack. When it finishes, it's removed, and JavaScript goes back to whatever was underneath. This is how JavaScript knows where to return after a function completes.

### Purposes

- To keep track of the current function being executed and the functions that called it.
- To manage the order of function calls and returns.
- To provide a mechanism for returning to the correct location after a function completes.
- To store local variables and arguments for each function call.
- To generate stack traces when errors occur.
- To enforce the single-threaded execution model of JavaScript.

### Syntax Rules and Structure

The call stack is managed automatically by the JavaScript engine; it is not directly manipulated by code.

```js
// When a function is called:
// 1. A new stack frame is created and pushed onto the stack.
// 2. The function's code executes.
// 3. When the function returns, its frame is popped off the stack.
// 4. Control returns to the calling function.

function greet(name) {
  return `Hello, ${name}!`;
}

function main() {
  const message = greet("Alice");
  console.log(message);
}

main();
```

**Stack operations:**
- **Push:** When a function is called, a new frame is pushed onto the top of the stack.
- **Pop:** When a function returns, the frame on top of the stack is popped off.
- **Peek:** The frame at the top of the stack is the currently running execution context.

**Constraints and Limitations:**
- The call stack has a finite size (engine-dependent).
- Only one function can execute at a time (single-threaded).
- Asynchronous callbacks are not placed on the call stack until their callback is invoked.
- The call stack is not directly accessible from JavaScript code.

### Annotated Code Examples

#### Example 1: Basic Call Stack Flow

```js
// Step 1: Global execution context is created and pushed onto the stack.
function greeting() {
  // Step 3: greeting() is called; a frame is pushed.
  sayHi();
  // Step 5: Control returns here after sayHi() finishes.
}
function sayHi() {
  // Step 4: sayHi() is called; a frame is pushed.
  return "Hi!";
  // Step 4b: sayHi() returns; its frame is popped.
}
// Step 2: greeting() is called; a frame is pushed.
greeting();
// Step 6: greeting() returns; its frame is popped.
// The stack is now empty (only global context remains).
```

**Expected Output:**
No output (the functions return values that are not logged).

**Why This Output:** The call stack pushes and pops frames as functions are called and return. `greeting()` is pushed first, then `sayHi()`. `sayHi()` returns first, then `greeting()` returns.

#### Example 2: Stack Trace on Error

```js
function level3() {
  throw new Error("Something went wrong");
}

function level2() {
  level3();
}

function level1() {
  level2();
}

try {
  level1();
} catch (err) {
  console.log(err.stack);
}
```

**Expected Output (approximate):**
```
Error: Something went wrong
    at level3 (<anonymous>:2:9)
    at level2 (<anonymous>:6:3)
    at level1 (<anonymous>:10:3)
    at <anonymous>:14:3
```

**Why This Output:** When the error is thrown, V8 captures a stack trace showing the call stack at the moment of the error. The trace lists frames from most recent (`level3`) to least recent (`level1`).

### Real-World Cases

- **Recursion:** Each recursive call pushes a new frame; the base case pops frames.
- **Event handlers:** When an event fires, its handler is pushed onto the stack.
- **Function composition:** Composed functions create nested stack frames.
- **Error debugging:** Stack traces reveal the call path to the error.

---

## Stack Frames

### Definitions

**Core Definition:** A stack frame is a data structure that represents a single function call on the call stack, containing the function's arguments, local variables, and return address.

**Technical Definition:** Function calls form a stack of frames. When calling a function, a frame is created containing references to its arguments and local variables. As always, calling a function creates a new stack frame for that function's use. In V8, stack frames are implemented as part of the execution context, with each frame storing the function's parameters, local variables, and a pointer to the caller's frame.

**Beginner-Friendly Explanation:** A stack frame is like a "work order" for a function call. It contains everything that function needs to do its job: the arguments you passed in, any variables it creates, and a note about where to return when it's done.

### Purposes

- To store the arguments passed to a function.
- To store local variables declared within the function.
- To record the return address (where to continue after the function returns).
- To isolate each function call's data from other calls.
- To provide context for debugging and stack traces.
- To enable recursion by giving each call its own frame.

### Syntax Rules and Structure

Stack frames are created automatically; they are not directly manipulated by code.

```js
function example(a, b) {
  // Stack frame for this call contains:
  // - a and b (arguments)
  // - localVar (local variable)
  // - return address (where to go after this function returns)
  var localVar = a + b;
  return localVar;
}

example(3, 4);
```

**Frame contents (conceptual):**
- **Arguments:** The values passed to the function.
- **Local variables:** Variables declared inside the function.
- **Return address:** The location in the calling code to return to.
- **Previous frame pointer:** A reference to the caller's frame.

**Constraints and Limitations:**
- Each frame consumes memory; deep stacks use more memory.
- Frames are engine-specific in their exact layout.
- Large local variables or many arguments increase frame size.

### Annotated Code Examples

#### Example 1: Multiple Stack Frames

```js
function multiply(a, b) {
  // Frame 3: a = 3, b = 4
  return a * b;
}

function add(x, y) {
  // Frame 2: x = 2, y = 12
  const product = multiply(3, 4); // Creates Frame 3
  return x + product;
}

function calculate() {
  // Frame 1: no arguments
  const result = add(2, 12); // Creates Frame 2
  return result;
}

console.log(calculate()); // 14
```

**Expected Output:**
```
14
```

**Why This Output:** `calculate()` is called first, creating Frame 1. It calls `add(2, 12)`, creating Frame 2. `add` calls `multiply(3, 4)`, creating Frame 3. `multiply` returns `12`, popping Frame 3. `add` returns `2 + 12 = 14`, popping Frame 2. `calculate` returns `14`, popping Frame 1.

#### Example 2: Stack Frame Isolation

```js
function outer() {
  let x = "outer";
  function inner() {
    let x = "inner"; // Different x, different frame
    console.log(x);  // "inner"
  }
  inner();
  console.log(x); // "outer" (unchanged)
}
outer();
```

**Expected Output:**
```
inner
outer
```

**Why This Output:** Each function call has its own stack frame. `inner`'s `x` is separate from `outer`'s `x`.

### Real-World Cases

- **Recursion:** Each recursive call has its own frame with its own parameters.
- **Callbacks:** Each callback invocation creates a new frame.
- **Closures:** Frames may be retained by closures after the function returns.
- **Debugging:** Debuggers display the stack of frames to show the current call path.

---

## Push and Pop

### Definitions

**Core Definition:** Push and pop are the two fundamental operations of the call stack: pushing adds a new frame when a function is called, and popping removes a frame when a function returns.

**Technical Definition:** When a function is called, a new frame is pushed onto the top of the stack. When the function returns, the frame on the top of the stack is popped off. The call stack operates on a Last-In-First-Out (LIFO) basis, meaning the most recently pushed frame is the first to be popped.

**Beginner-Friendly Explanation:** Think of the call stack as a stack of sticky notes. When you call a function, you write a note and stick it on top. When the function finishes, you take that note off. The note on top is always the one you're currently working on.

### Purposes

- To track the current position in the program's execution.
- To ensure functions return to the correct location.
- To maintain the order of function calls and returns.
- To manage memory for function calls efficiently.
- To support recursion by pushing/popping frames per call.

### Syntax Rules and Structure

Push and pop are automatic operations; they are not written in code.

```js
function a() {
  // Frame for a() pushed.
  b();
  // Frame for b() popped; control returns here.
}
function b() {
  // Frame for b() pushed.
  c();
  // Frame for c() popped.
}
function c() {
  // Frame for c() pushed.
  // c() returns; frame popped.
}
a(); // Frame for a() pushed; eventually popped.
```

**Operation Sequence:**
1. **Push:** Function call → new frame on top.
2. **Execute:** The top frame's function runs.
3. **Pop:** Function returns → top frame removed.
4. **Repeat:** The new top frame resumes execution.

**Constraints and Limitations:**
- Push operations are limited by the stack size.
- Pop operations must follow LIFO order (with rare exceptions for generators).
- Unbalanced push/pop (e.g., missing return) causes errors.

### Annotated Code Examples

#### Example 1: Push and Pop Sequence

```js
function first() {
  console.log("first: start");
  second(); // Push second
  console.log("first: end");
}

function second() {
  console.log("second: start");
  third(); // Push third
  console.log("second: end");
}

function third() {
  console.log("third");
  // third returns; pop third
}

first();
// first returns; pop first
```

**Expected Output:**
```
first: start
second: start
third
second: end
first: end
```

**Why This Output:** `first` is pushed, then `second`, then `third`. `third` finishes first (popped), then `second`, then `first`.

#### Example 2: Push and Pop with Return Values

```js
function square(n) {
  return n * n;
}

function sumOfSquares(a, b) {
  const sa = square(a); // Push square, pop after return
  const sb = square(b); // Push square, pop after return
  return sa + sb;
}

console.log(sumOfSquares(3, 4)); // 25
```

**Expected Output:**
```
25
```

**Why This Output:** Each call to `square` pushes a frame, computes the square, returns (popping the frame), and the result is stored in `sumOfSquares`'s frame.

### Real-World Cases

- **Recursion:** Push/pop happens for each recursive call.
- **Error handling:** Push/pop tracks the call path for stack traces.
- **Performance:** Push/pop operations are very fast but limited by stack size.
- **Debugging:** Debuggers show the current stack of pushed frames.

---

## Stack Overflow

### Definitions

**Core Definition:** A stack overflow occurs when the call stack exceeds its maximum size, typically due to infinite or excessively deep recursion.

**Technical Definition:** When the stack takes up more space than it was assigned, a "stack overflow" error is thrown. In V8 (Chrome, Node.js), this throws a `RangeError: Maximum call stack size exceeded`. In Firefox, it throws `InternalError: too much recursion`. The maximum stack size varies by engine and platform.

**Beginner-Friendly Explanation:** A stack overflow happens when you call too many functions without returning from them. It's like stacking too many plates — eventually the pile falls over. This usually happens with infinite recursion (a function that calls itself forever).

### Purposes (and Anti-Purposes)

- **Anti-purpose:** Stack overflow is an error condition, not a feature.
- To understand the limits of recursion.
- To recognise and fix infinite recursion bugs.
- To understand why iterative solutions are sometimes preferred.

### Syntax Rules and Structure

Stack overflow occurs at runtime when the stack exceeds its limit:

```js
function infinite() {
  infinite(); // Calls itself forever
}

try {
  infinite();
} catch (err) {
  console.log(err.name);    // "RangeError"
  console.log(err.message); // "Maximum call stack size exceeded"
}
```

**Common causes:**
- Missing base case in recursion.
- Base case never reached.
- Mutually recursive functions without termination.
- Setter/getter infinite loops.

**Browser stack limits (approximate):**
- Chrome/Chromium: ~11,000 function calls.
- Firefox: ~26,000 function calls.
- Safari: ~45,000 function calls.

**Constraints and Limitations:**
- The limit is engine- and platform-dependent.
- The limit can be affected by frame size (larger frames = fewer calls).
- Tail call optimisation (TCO) is not universally supported in JavaScript engines.

### Annotated Code Examples

#### Example 1: Infinite Recursion

```js
function recurse() {
  recurse(); // No base case
}

try {
  recurse();
} catch (err) {
  console.log(err.name);    // "RangeError"
  console.log(err.message); // "Maximum call stack size exceeded"
}
```

**Expected Output:**
```
RangeError
Maximum call stack size exceeded
```

**Why This Output:** The function calls itself indefinitely, pushing frames until the stack limit is reached, at which point a `RangeError` is thrown.

#### Example 2: Fixing Stack Overflow with a Base Case

```js
function factorial(n) {
  if (n <= 1) return 1; // Base case
  return n * factorial(n - 1);
}

console.log(factorial(5)); // 120
console.log(factorial(1000)); // Infinity (too large for Number)
```

**Expected Output:**
```
120
Infinity
```

**Why This Output:** The base case stops the recursion. `factorial(1000)` returns `Infinity` because the result exceeds `Number.MAX_VALUE`, not because of stack overflow.

#### Example 3: Mutual Recursion Overflow

```js
function isEven(n) {
  if (n === 0) return true;
  return isOdd(n - 1);
}

function isOdd(n) {
  if (n === 0) return false;
  return isEven(n - 1);
}

try {
  isEven(100000); // Too deep
} catch (err) {
  console.log(err.name); // "RangeError"
}
```

**Expected Output:**
```
RangeError
```

**Why This Output:** Even with a base case, the recursion depth exceeds the stack limit for large `n`. An iterative solution would avoid this.

### Real-World Cases

- **Recursive algorithms:** Factorial, Fibonacci, tree traversal.
- **Mutual recursion:** Parser combinators, state machines.
- **Getter/setter loops:** A setter that assigns to the same property.
- **Deeply nested data:** Recursively processing deeply nested JSON.

---

## Stack Traces

### Definitions

**Core Definition:** A stack trace is a report of the active stack frames at a certain point in time during the execution of a program, showing the sequence of function calls that led to the current point.

**Technical Definition:** All internal errors thrown in V8 capture a stack trace when they are created. This stack trace can be accessed from JavaScript through the non-standard `error.stack` property. By default, almost all errors thrown by V8 have a `stack` property that holds the topmost 10 stack frames, formatted as a string. The stack trace is collected when the error is created and is the same regardless of where or how many times the error is thrown.

**Beginner-Friendly Explanation:** A stack trace is like a "receipt" of function calls. When an error happens, JavaScript writes down the list of functions that were running, in order, from the most recent to the least recent. This list helps you find where the error came from.

### Purposes

- To identify the exact location where an error occurred.
- To trace the sequence of function calls leading to the error.
- To debug complex, nested, or asynchronous code.
- To provide diagnostic information in logs and error reports.
- To help developers understand the execution path at the time of failure.

### Syntax Rules and Structure

Stack traces are accessed via the `stack` property of `Error` objects:

```js
try {
  throw new Error("Something failed");
} catch (err) {
  console.log(err.stack);
}
```

**Typical format (V8):**
```
Error: Something failed
    at functionName (file.js:line:column)
    at callerName (file.js:line:column)
    ...
```

**Components:**
- **Error name and message:** The first line.
- **Frames:** Each line after the first represents a stack frame.
- **Frame format:** `at functionName (file:line:column)`.

**Constraints and Limitations:**
- The `stack` property is non-standard (though widely implemented).
- Format varies between engines (V8, SpiderMonkey, JavaScriptCore).
- `Error.stackTraceLimit` controls the number of frames collected (default 10).
- Async stack traces are limited to `await`, `Promise.all()`, and `Promise.any()`.

### Annotated Code Examples

#### Example 1: Reading a Stack Trace

```js
function a() {
  b();
}

function b() {
  c();
}

function c() {
  throw new Error("Deep error");
}

try {
  a();
} catch (err) {
  console.log(err.stack);
}
```

**Expected Output (approximate):**
```
Error: Deep error
    at c (<anonymous>:10:9)
    at b (<anonymous>:6:3)
    at a (<anonymous>:2:3)
    at <anonymous>:14:3
```

**Why This Output:** The stack trace shows the call chain from the error location (`c`) back to the top-level call (`a`). The topmost frame is the most recent call.

#### Example 2: Controlling Stack Trace Depth

```js
Error.stackTraceLimit = 3; // Only collect 3 frames

function a() { b(); }
function b() { c(); }
function c() { throw new Error("Limited trace"); }

try {
  a();
} catch (err) {
  console.log(err.stack);
}
```

**Expected Output (approximate):**
```
Error: Limited trace
    at c (<anonymous>:3:19)
    at b (<anonymous>:2:14)
    at a (<anonymous>:1:14)
```

**Why This Output:** `Error.stackTraceLimit` is set to 3, so only the top 3 frames are collected.

#### Example 3: Async Stack Trace

```js
async function foo() {
  await bar();
}

async function bar() {
  throw new Error("Async error");
}

foo().catch((err) => console.log(err.stack));
```

**Expected Output (approximate):**
```
Error: Async error
    at bar (<anonymous>:6:9)
    at async foo (<anonymous>:2:9)
```

**Why This Output:** V8's async stack traces include `async` frames for `await` locations, showing the async call chain.

### Real-World Cases

- **Production error monitoring:** Sentry, Bugsnag, and Rollbar parse stack traces.
- **Browser DevTools:** Console shows stack traces for errors.
- **Node.js debugging:** Stack traces in terminal output.
- **Source maps:** Mapping minified stack traces back to source code.

---

## Debugging with Stack Traces

### Definitions

**Core Definition:** Debugging with stack traces is the practice of using the call stack information in a stack trace to diagnose and fix errors in JavaScript code.

**Technical Definition:** The stack trace shows the active stack frames at the time an error was created. By analysing the sequence of frames, developers can determine the exact line of code that threw the error and the chain of function calls that led to it. Chrome DevTools shows the call stack beneath the text of each error in the Console. V8 captures stack traces when errors are created, making them available via the `error.stack` property.

**Beginner-Friendly Explanation:** When something breaks, the stack trace is your "map" of where the error happened. It shows you the list of functions that were running, so you can trace backwards to find the bug.

### Purposes

- To identify the exact location of an error (file, line, column).
- To trace the sequence of function calls leading to the error.
- To understand the execution path at the time of failure.
- To debug complex, nested, or asynchronous code.
- To reproduce and fix bugs faster.
- To map minified production errors back to source code.

### Syntax Rules and Structure

Debugging with stack traces involves reading the `error.stack` property and using browser DevTools:

```js
try {
  riskyOperation();
} catch (err) {
  // Log the stack trace for debugging.
  console.error(err.stack);
  // Or send to a monitoring service.
}
```

**Reading a stack trace:**
1. **First line:** Error type and message.
2. **Subsequent lines:** Stack frames, most recent first.
3. **Each frame:** Function name, file name, line number, column number.
4. **Top frame:** Where the error occurred.
5. **Bottom frame:** The outermost call (e.g., event handler, top-level script).

**Tools:**
- **Browser DevTools:** Console shows clickable stack traces.
- **Source maps:** Map minified code back to original source.
- **Error monitoring:** Sentry, Bugsnag, Rollbar aggregate and analyse traces.

**Constraints and Limitations:**
- Stack traces can be truncated (`Error.stackTraceLimit`).
- Async stack traces are limited to specific constructs.
- Minified code requires source maps to be readable.
- Some errors (e.g., syntax errors) don't produce useful stack traces.

### Annotated Code Examples

#### Example 1: Debugging with Stack Trace

```js
function validateInput(input) {
  if (!input) {
    throw new Error("Input cannot be empty");
  }
  return input.trim();
}

function processForm(data) {
  return validateInput(data.name);
}

try {
  processForm({ name: "" });
} catch (err) {
  // Read the stack trace to find the error origin.
  console.log("Error:", err.message);
  console.log("Stack:");
  console.log(err.stack);
}
```

**Expected Output:**
```
Error: Input cannot be empty
Stack:
Error: Input cannot be empty
    at validateInput (<anonymous>:3:11)
    at processForm (<anonymous>:9:10)
    at <anonymous>:12:3
```

**Why This Output:** The stack trace shows that the error originated in `validateInput`, was called by `processForm`, which was called from the top level. This tells the developer exactly where to look.

#### Example 2: Async Stack Trace Debugging

```js
async function fetchData() {
  await simulateNetworkCall();
}

async function simulateNetworkCall() {
  throw new Error("Network timeout");
}

fetchData().catch((err) => {
  console.log("Async error:", err.message);
  console.log(err.stack);
});
```

**Expected Output (approximate):**
```
Async error: Network timeout
Error: Network timeout
    at simulateNetworkCall (<anonymous>:6:9)
    at async fetchData (<anonymous>:2:9)
```

**Why This Output:** The async stack trace shows the `await` chain, helping debug asynchronous code.

#### Example 3: Custom Error with Stack Trace

```js
class ValidationError extends Error {
  constructor(message) {
    super(message);
    this.name = "ValidationError";
    // Capture stack trace, excluding the constructor.
    Error.captureStackTrace(this, ValidationError);
  }
}

try {
  throw new ValidationError("Invalid email");
} catch (err) {
  console.log(err.stack);
}
```

**Expected Output (approximate):**
```
ValidationError: Invalid email
    at <anonymous>:9:9
```

**Why This Output:** `Error.captureStackTrace` omits the constructor frame, producing a cleaner stack trace.

### Real-World Cases

- **Production debugging:** Mapping minified stack traces with source maps.
- **Error monitoring:** Sentry, Bugsnag, Rollbar use stack traces for grouping.
- **Development:** Chrome DevTools shows clickable stack traces.
- **Node.js:** Stack traces in server logs for API errors.
- **React:** Error boundaries log component stack traces.

---

## References

- MDN Web Docs – Call stack — https://developer.mozilla.org/en-US/docs/Glossary/Call_stack
- MDN Web Docs – The event loop — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Event_loop
- ECMA-262 (2015 Draft) – Execution Contexts — https://archives.ecma-international.org/2015/TC39/tc39-2015-016.pdf
- ECMA-262 (2008 Draft) – Execution Contexts — http://archives.ecma-international.org/2008/misc/tc39-es31-draft03nov08.pdf
- V8 Docs – Stack trace API — https://v8.dev/docs/stack-trace-api
- MDN Web Docs – InternalError: too much recursion — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Errors/Too_much_recursion
- Chrome DevTools – Debug JavaScript — https://developer.chrome.com/docs/devtools/javascript/
- MDN Web Docs – Error.prototype.stack — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error/stack
- MDN Web Docs – Error.stackTraceLimit — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error/stackTraceLimit
- V8 Blog – Async stack traces — https://v8.dev/blog/v8-release-73
- TrackJS – How to fix Maximum call stack size exceeded — https://trackjs.com/blog/how-to-fix-maximum-call-stack-size-exceeded/
- Wikipedia – Call stack — https://en.wikipedia.org/wiki/Call_stack
- MDN Web Docs – Execution context — https://developer.mozilla.org/en-US/docs/Glossary/Execution_context