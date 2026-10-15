# JavaScript Debugging Fundamentals — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Debugging is the systematic process of identifying, isolating, and fixing defects (bugs) in software by inspecting program state, controlling execution, and analysing behaviour.

**Technical Definition:** Per ECMAScript 2025 (ECMA-262, §14.1), the `debugger` statement is a reserved keyword that, when evaluated, causes the implementation to pause execution if a debugger is attached or if the implementation defines a debugging facility. All JavaScript engines (V8, SpiderMonkey, JavaScriptCore) provide debugging facilities via the Chrome DevTools Protocol (CDP), the Firefox Debugger Protocol, or the WebKit Inspector Protocol. These protocols expose breakpoints, stepping, call stack inspection, scope inspection, and expression evaluation. The `debugger` statement is normative but implementation-defined: if no debugger is present, it has no observable effect.

**Beginner-Friendly Explanation:** Debugging is how you find and fix problems in your code. Instead of guessing why your program is misbehaving, you use special tools to pause your code mid-execution, look at the values of variables, step through line by line, and see exactly what's happening. Browser DevTools (opened with F12 or Ctrl+Shift+I) is the most common debugging environment for JavaScript.

### Key Characteristics

- **Tool-based:** Debugging relies on browser or runtime tooling.
- **Non-destructive:** You can inspect state without changing code (mostly).
- **Step-based:** Control execution one line at a time.
- **State-aware:** Inspect variables, scopes, and the call stack.
- **Universal:** All major browsers and Node.js support debugging.
- **Standardised protocol:** CDP allows cross-tool debugging.
- **Production-safe:** `debugger` statements are no-ops without a debugger.

### Prerequisites

- Basic JavaScript syntax (variables, functions, statements).
- Understanding of scope, closures, and the call stack.
- Familiarity with the browser environment (DOM, console).
- Awareness of execution contexts and hoisting.
- Basic knowledge of error handling.

### Related Programming Areas

- **Error handling:** Catching and reporting errors.
- **Logging:** `console.log`, `console.error`, structured logging.
- **Testing:** Unit tests, integration tests, assertions.
- **Performance profiling:** DevTools Performance panel.
- **Memory profiling:** DevTools Memory panel.
- **Network debugging:** DevTools Network panel.

---

## Debugging

### Definitions

**Core Definition:** Debugging is the process of finding and fixing errors in a program.

**Technical Definition:** Per MDN, debugging is the process of finding and fixing bugs in your code. It involves reproducing the bug, isolating the cause, inspecting the program state, and applying a fix. The term derives from "debugging" a computer — removing moths or other physical defects from early electromechanical computers. Modern debugging is primarily software-based.

**Beginner-Friendly Explanation:** Debugging is detective work. Something is wrong with your code, and you need to find out what and why.

### Purposes

- To identify the root cause of incorrect behaviour.
- To inspect program state at specific points.
- To verify assumptions about how code runs.
- To fix bugs efficiently.
- To understand unfamiliar code.

### Debugging Workflow

1. **Reproduce the bug:** Find a reliable way to trigger it.
2. **Isolate the cause:** Narrow down which code is responsible.
3. **Inspect state:** Use breakpoints and watches to examine variables.
4. **Form a hypothesis:** Predict what should happen.
5. **Test the hypothesis:** Step through or modify values.
6. **Fix the bug:** Apply the change.
7. **Verify the fix:** Re-run and confirm.
8. **Prevent regression:** Add tests.

### Real-World Cases

- **UI bugs:** Buttons not working, elements not rendering.
- **Logic errors:** Wrong calculations, incorrect conditions.
- **Async bugs:** Race conditions, unhandled promises.
- **Performance issues:** Slow loops, memory leaks.

---

## Debugging Tools

### Definitions

**Core Definition:** Debugging tools are software applications or features that help developers inspect and control program execution.

**Technical Definition:** JavaScript debugging tools include browser DevTools (Chrome, Firefox, Safari, Edge), Node.js Inspector, standalone debuggers (VS Code, WebStorm), and remote debugging tools (Chrome DevTools Protocol, React DevTools). Each provides breakpoints, stepping, call stack inspection, and console evaluation.

**Beginner-Friendly Explanation:** Debugging tools are the software you use to find bugs — like DevTools in your browser.

### Common Tools

- **Chrome DevTools:** Full-featured debugging for Chrome/Chromium.
- **Firefox Developer Tools:** Debugger, Inspector, Console.
- **Safari Web Inspector:** Apple's debugging tool.
- **Edge DevTools:** Chromium-based, similar to Chrome.
- **Node.js Inspector:** `node --inspect`.
- **VS Code:** Integrated debugger with launch configurations.
- **WebStorm:** Full IDE debugger.
- **React DevTools:** Component tree inspection.

### Real-World Cases

- **Browser bugs:** DevTools console and sources.
- **Node.js bugs:** `node --inspect` + Chrome DevTools.
- **Mobile bugs:** Remote debugging via USB.
- **Production bugs:** Source maps + error monitoring.

---

## Browser DevTools

### Definitions

**Core Definition:** Browser DevTools are built-in developer tools in modern browsers for inspecting and debugging web pages.

**Technical Definition:** Browser DevTools are implemented as a separate renderer that communicates with the page via a debugging protocol (CDP for Chromium-based browsers). Panels include Elements (DOM), Console (logs and evaluation), Sources (debugging), Network (requests), Performance (profiling), Memory (heap), Application (storage), and Security. The Sources panel provides the JavaScript debugger.

**Beginner-Friendly Explanation:** DevTools is the panel that opens when you press F12 or Ctrl+Shift+I. It lets you inspect the page, debug JavaScript, and see what's happening.

### Opening DevTools

- **Chrome/Edge:** F12 or Ctrl+Shift+I (Cmd+Option+I on Mac).
- **Firefox:** F12 or Ctrl+Shift+I.
- **Safari:** Enable Developer menu in Preferences, then Cmd+Option+I.

### Key Panels

- **Elements:** Inspect and modify the DOM.
- **Console:** Run JavaScript, view logs.
- **Sources:** Debug JavaScript with breakpoints.
- **Network:** Inspect HTTP requests.
- **Performance:** Profile runtime performance.
- **Memory:** Profile memory usage.
- **Application:** Inspect storage, service workers.
- **Security:** Check HTTPS, certificates.

### Real-World Cases

- **DOM debugging:** Inspect elements, styles.
- **JavaScript debugging:** Breakpoints in Sources.
- **Network issues:** Inspect request/response.
- **Performance:** Record profiles.

---

## Console

### Definitions

**Core Definition:** The Console is a DevTools panel for viewing logs, running JavaScript, and interacting with the page.

**Technical Definition:** The Console implements the `console` API (`log`, `warn`, `error`, `info`, `debug`, `table`, `group`, `time`, `timeEnd`, `trace`, `assert`, `count`, `dir`). It also provides a REPL for evaluating expressions in the page's global scope (or the selected frame's scope when paused).

**Beginner-Friendly Explanation:** The Console is where you see messages from your code and where you can type JavaScript to test things.

### Console Methods

```js
console.log("message");
console.warn("warning");
console.error("error");
console.info("info");
console.debug("debug");
console.table([{ a: 1 }, { a: 2 }]);
console.group("Group");
console.groupEnd();
console.time("timer");
console.timeEnd("timer");
console.trace("stack trace");
console.assert(condition, "message");
console.count("counter");
console.dir(object);
```

### Annotated Code Examples

```js
// Step 1: Basic logging.
console.log("Hello", { name: "Alice" }, [1, 2, 3]);

// Step 2: Table for arrays of objects.
console.table([
  { name: "Alice", age: 30 },
  { name: "Bob", age: 25 },
]);

// Step 3: Timing.
console.time("loop");
for (let i = 0; i < 1e6; i++) {}
console.timeEnd("loop"); // "loop: 5.23ms"

// Step 4: Assertions.
console.assert(1 === 2, "This will show an error");
```

**Expected Output (in Console):**
```
Hello {name: "Alice"} [1, 2, 3]
┌─────────┬─────────┬─────┐
│ (index) │ name    │ age │
├─────────┼─────────┼─────┤
│ 0       │ "Alice" │ 30  │
│ 1       │ "Bob"   │ 25  │
└─────────┴─────────┴─────┘
loop: 5.23ms
Assertion failed: This will show an error
```

### Real-World Cases

- **Quick inspection:** `console.log(obj)`.
- **Tabular data:** `console.table(arr)`.
- **Performance:** `console.time`.
- **Debugging stacks:** `console.trace()`.

---

## Sources Panel

### Definitions

**Core Definition:** The Sources panel is DevTools' JavaScript debugger, where you set breakpoints, step through code, and inspect state.

**Technical Definition:** The Sources panel in Chrome DevTools provides a file tree, code editor, debugger controls, call stack, scope inspector, breakpoints list, and watch expressions. It uses the Chrome DevTools Protocol to control the V8 debugger.

**Beginner-Friendly Explanation:** The Sources panel is where you debug JavaScript. You can pause your code, step through it line by line, and look at variables.

### Key Features

- **File tree:** Browse page resources.
- **Code editor:** View and edit source.
- **Breakpoints:** Pause execution at specific lines.
- **Call stack:** See the current execution path.
- **Scope:** Inspect local, closure, and global variables.
- **Watch:** Monitor specific expressions.
- **Controls:** Resume, step over, step into, step out.

### Real-World Cases

- **Debugging logic:** Pause and inspect.
- **Understanding flow:** Step through unfamiliar code.
- **Async debugging:** Use async stack traces.

---

## Breakpoints

### Definitions

**Core Definition:** A breakpoint is a marker that tells the debugger to pause execution when a specific line of code is reached.

**Technical Definition:** Breakpoints are set on specific source locations (file, line, column). When the V8 debugger reaches a breakpoint, it pauses execution and notifies the DevTools frontend. Breakpoints can be set on lines, conditional (pause only if a condition is true), logpoints (log without pausing), or DOM/event-based.

**Beginner-Friendly Explanation:** A breakpoint is like a "stop sign" in your code. When execution reaches that line, the debugger pauses so you can inspect what's happening.

### Types of Breakpoints

- **Line breakpoint:** Pause at a specific line.
- **Conditional breakpoint:** Pause only if a condition is true.
- **Logpoint:** Log a message without pausing.
- **DOM breakpoint:** Pause when the DOM changes.
- **Event listener breakpoint:** Pause on events (click, keydown, etc.).
- **XHR/fetch breakpoint:** Pause on network requests.
- **Exception breakpoint:** Pause on thrown exceptions.

### Annotated Code Examples

```js
function calculateTotal(items) {
  let total = 0;
  for (const item of items) {
    total += item.price; // Set breakpoint here
  }
  return total;
}

calculateTotal([
  { name: "Apple", price: 1.5 },
  { name: "Banana", price: 0.75 },
]);
```

**How to Debug:**
1. Open DevTools → Sources.
2. Find the file.
3. Click the line number to set a breakpoint.
4. Run the code.
5. Execution pauses; inspect `item`, `total`, `items`.

### Real-World Cases

- **Loop bugs:** Break inside the loop.
- **Conditional bugs:** Conditional breakpoint on `i === 5`.
- **Function entry:** Break at function start.
- **Network bugs:** XHR breakpoint on URL.

---

## Conditional Breakpoints

### Definitions

**Core Definition:** A conditional breakpoint pauses execution only when a specified condition evaluates to true.

**Technical Definition:** Conditional breakpoints are line breakpoints with an associated expression. When the debugger reaches the line, it evaluates the expression in the current scope; if truthy, execution pauses; otherwise, it continues.

**Beginner-Friendly Explanation:** A conditional breakpoint says "only pause if this condition is true." Useful when a bug only happens on a specific iteration or value.

### How to Set

1. Right-click a line number in Sources.
2. Select "Add conditional breakpoint".
3. Enter a condition (e.g., `i === 100`).
4. Press Enter.

### Annotated Code Examples

```js
function processItems(items) {
  for (let i = 0; i < items.length; i++) {
    // Conditional breakpoint: i === 100
    console.log(items[i]);
  }
}

processItems(Array.from({ length: 1000 }, (_, i) => i));
```

**Why This Helps:** Without a condition, the breakpoint would pause 1000 times. With `i === 100`, it pauses only once.

### Real-World Cases

- **Specific iteration:** `i === 5`.
- **Specific value:** `user.id === 42`.
- **Error condition:** `items[i] === undefined`.
- **Rare bug:** `Math.random() < 0.001`.

---

## Logpoints

### Definitions

**Core Definition:** A logpoint is a breakpoint that logs a message to the Console without pausing execution.

**Technical Definition:** Logpoints (also called "message breakpoints") are line breakpoints with an associated message expression. When the debugger reaches the line, it evaluates and logs the message but does not pause. Logpoints are a non-invasive alternative to `console.log`.

**Beginner-Friendly Explanation:** A logpoint is like a temporary `console.log` you can add without editing your code.

### How to Set

1. Right-click a line number.
2. Select "Add logpoint".
3. Enter a message with `{}` for expressions (e.g., `total is {total}`).
4. Press Enter.

### Annotated Code Examples

```js
function calculateTotal(items) {
  let total = 0;
  for (const item of items) {
    total += item.price; // Logpoint: "total = {total}, item = {item.name}"
  }
  return total;
}
```

**Console Output (when executed):**
```
total = 1.5, item = Apple
total = 2.25, item = Banana
```

### Real-World Cases

- **Avoid code edits:** Log without `console.log`.
- **Temporary logging:** Quick inspection.
- **Production-like debugging:** Non-blocking logs.

---

## Watch Expressions

### Definitions

**Core Definition:** A watch expression is a JavaScript expression that DevTools continuously evaluates and displays while debugging.

**Technical Definition:** Watch expressions are evaluated in the current scope each time execution pauses. They persist across sessions and can reference any in-scope variables. DevTools shows their current value and updates them as you step.

**Beginner-Friendly Explanation:** A watch expression lets you keep an eye on a specific value or expression as you step through code.

### How to Use

1. Open DevTools → Sources.
2. In the right panel, find "Watch".
3. Click "+" and enter an expression (e.g., `user.name`).
4. The value updates as you step.

### Annotated Code Examples

```js
function updateUser(user, changes) {
  // Watch: user, changes, Object.keys(changes)
  return { ...user, ...changes };
}

updateUser({ name: "Alice", age: 30 }, { age: 31 });
```

**Watch Panel:**
```
user: {name: "Alice", age: 30}
changes: {age: 31}
Object.keys(changes): ["age"]
```

### Real-World Cases

- **Complex expressions:** `items.filter(x => x.active).length`.
- **Object properties:** `user.profile.name`.
- **Loop invariants:** `total / count`.
- **Conditional state:** `isValid && !isLoading`.

---

## Call Stack

### Definitions

**Core Definition:** The call stack is the list of active function calls, shown in DevTools as frames from the current function back to the top-level.

**Technical Definition:** Per ECMA-262, execution contexts form a logical stack. DevTools displays this stack with frame names, file locations, and line numbers. Clicking a frame switches the scope inspector and watch evaluation to that frame's context.

**Beginner-Friendly Explanation:** The call stack shows how you got to where you are — which function called which function, all the way back to the top.

### How to Use

1. Pause at a breakpoint.
2. Look at the "Call Stack" section in the right panel.
3. Click a frame to inspect its scope.

### Annotated Code Examples

```js
function a() { b(); }
function b() { c(); }
function c() {
  debugger; // Call stack: c → b → a → (top level)
}
a();
```

**Call Stack Panel:**
```
c        (script.js:4)
b        (script.js:2)
a        (script.js:1)
(anonymous) (script.js:6)
```

### Real-World Cases

- **Trace execution:** Understand how code reached a point.
- **Async debugging:** See `async` frames.
- **Recursion:** See repeated frames.
- **Error diagnosis:** Follow the path to an error.

---

## Scope Inspection

### Definitions

**Core Definition:** Scope inspection is the examination of variables available at the current execution point, grouped by scope.

**Technical Definition:** DevTools groups variables into Local (current function), Closure (captured variables), Script (top-level script), and Global (window/globalThis). When paused, the Scope panel shows the value of each binding. You can expand objects, edit values in place, and check prototypes.

**Beginner-Friendly Explanation:** Scope inspection shows you all the variables you can access at the current point — local variables, closed-over variables, and globals.

### How to Use

1. Pause at a breakpoint.
2. In the right panel, find "Scope".
3. Expand "Local", "Closure", "Script", "Global".
4. Click values to inspect.

### Annotated Code Examples

```js
const globalVar = "global";

function outer() {
  const outerVar = "outer";
  function inner() {
    const innerVar = "inner";
    debugger; // Scope: innerVar (Local), outerVar (Closure), globalVar (Script/Global)
  }
  inner();
}
outer();
```

**Scope Panel:**
```
Local:
  innerVar: "inner"
Closure (outer):
  outerVar: "outer"
Script:
  globalVar: "global"
Global:
  window: Window
```

### Real-World Cases

- **Closure bugs:** Inspect captured variables.
- **Shadowing:** See which variable is in scope.
- **Async state:** Check what's captured in async callbacks.
- **Mutable state:** Edit values during debugging.

---

## Step Over

### Definitions

**Core Definition:** Step over executes the current line and pauses at the next line in the same function.

**Technical Definition:** In the CDP, "step over" (`Debugger.stepOver`) advances the debugger to the next statement in the current frame, treating function calls as a single step (not entering them).

**Beginner-Friendly Explanation:** Step over runs the current line and moves to the next line, without going inside any function calls.

### When to Use

- When you want to advance line by line.
- When a function call is not the source of the bug.

### Keyboard Shortcut

- **Chrome/Edge:** F10
- **Firefox:** F10
- **Safari:** F10

---

## Step Into

### Definitions

**Core Definition:** Step into enters the function called on the current line and pauses at its first statement.

**Technical Definition:** In the CDP, "step into" (`Debugger.stepInto`) moves the debugger into the function called at the current location, if any.

**Beginner-Friendly Explanation:** Step into goes inside a function call so you can debug it line by line.

### When to Use

- When you suspect the bug is inside a called function.
- To inspect library code or helper functions.

### Keyboard Shortcut

- **Chrome/Edge:** F11
- **Firefox:** F11
- **Safari:** F11

---

## Step Out

### Definitions

**Core Definition:** Step out continues execution until the current function returns to its caller.

**Technical Definition:** In the CDP, "step out" (`Debugger.stepOut`) runs the rest of the current function and pauses at the calling frame.

**Beginner-Friendly Explanation:** Step out finishes the current function and pauses in the function that called it.

### When to Use

- When you've finished debugging inside a function.
- To return to the caller's context.

### Keyboard Shortcut

- **Chrome/Edge:** Shift+F11
- **Firefox:** Shift+F11
- **Safari:** Shift+F11

---

## Resume

### Definitions

**Core Definition:** Resume continues execution until the next breakpoint or the program ends.

**Technical Definition:** In the CDP, "resume" (`Debugger.resume`) resumes execution from the current pause. The debugger will pause again at the next breakpoint, exception, or `debugger` statement.

**Beginner-Friendly Explanation:** Resume tells the debugger "continue running until the next stop."

### When to Use

- When you've finished inspecting the current state.
- To skip to the next breakpoint.

### Keyboard Shortcut

- **Chrome/Edge:** F8
- **Firefox:** F8
- **Safari:** F8

---

## `debugger` Statement

### Definitions

**Core Definition:** The `debugger` statement is a language keyword that pauses execution when a debugger is attached.

**Technical Definition:** Per ECMA-262 §14.1, the `debugger` statement, when evaluated, causes the implementation to pause if a debugger is attached. If no debugger is present, it is a no-op. It is defined as a `DebuggerStatement` in the grammar.

**Beginner-Friendly Explanation:** Writing `debugger;` in your code pauses execution when DevTools is open — like a breakpoint you can leave in your code.

### Purposes

- To pause execution at a specific line without setting a breakpoint.
- To debug code in environments where setting breakpoints is inconvenient.
- To conditionally pause with an `if` statement.

### Syntax Rules and Structure

```js
debugger;
```

**Syntax Rules:**
- Valid anywhere a statement is allowed.
- No effect if no debugger is attached.
- Often used inside conditionals.

**Constraints and Limitations:**
- Should be removed before production (or guarded).
- Linters may warn about `debugger` statements.

### Annotated Code Examples

#### Example 1: Basic `debugger`

```js
function processData(data) {
  debugger; // Pauses here if DevTools is open
  return data.map((x) => x * 2);
}
processData([1, 2, 3]);
```

**Expected Output:** Execution pauses at `debugger` when DevTools is open.

#### Example 2: Conditional `debugger`

```js
for (let i = 0; i < 100; i++) {
  if (i === 50) debugger; // Pauses only at i = 50
  console.log(i);
}
```

### Real-World Cases

- **Quick debugging:** Insert `debugger;` and refresh.
- **Conditional pause:** Wrap in an `if`.
- **Node.js:** `node inspect` pauses on `debugger`.

---

## Debugging Best Practices

### Definitions

**Core Definition:** Debugging best practices are the conventions and techniques that make debugging efficient and effective.

**Technical Definition:** Best practices include reproducing bugs reliably, isolating causes, using breakpoints over `console.log`, leveraging conditional breakpoints, watching expressions, inspecting the call stack, avoiding `debugger` in production, and using source maps for minified code.

**Beginner-Friendly Explanation:** These are the habits that make debugging faster and less frustrating.

### Best Practices

1. **Reproduce the bug reliably.**
   - Find a consistent sequence of steps.

2. **Isolate the cause.**
   - Comment out code, use binary search.

3. **Prefer breakpoints over `console.log`.**
   - Breakpoints inspect full state without editing code.

4. **Use conditional breakpoints for loops.**
   - `i === 100` instead of pausing 100 times.

5. **Use logpoints for non-blocking logging.**
   - Log without editing code.

6. **Watch key expressions.**
   - Add `user.name`, `items.length` to Watch.

7. **Inspect the call stack.**
   - Click frames to see how you got there.

8. **Inspect scope.**
   - Check Local, Closure, and Global scopes.

9. **Use `debugger` conditionally.**
   ```js
   if (process.env.NODE_ENV !== "production") debugger;
   ```

10. **Remove `debugger` and `console.log` before committing.**
    - Use linters (`no-debugger`, `no-console`).

11. **Use source maps for production.**
    - Debug minified code with original source.

12. **Use async stack traces.**
    - Enable "Async" in DevTools call stack.

13. **Take advantage of DevTools features.**
    - Blackboxing, event listener breakpoints, XHR breakpoints.

14. **Write tests to prevent regressions.**
    - Add a test that reproduces the bug.

15. **Document the fix.**
    - Add comments explaining the bug and solution.

### Annotated Code Examples

#### Example 1: Conditional `debugger` in Development

```js
function calculateTotal(items) {
  let total = 0;
  for (const item of items) {
    total += item.price;
    if (process.env.NODE_ENV !== "production" && total > 1000) {
      debugger; // Pause only in dev when total exceeds 1000
    }
  }
  return total;
}
```

#### Example 2: Using Logpoints Instead of `console.log`

```js
function processOrder(order) {
  // Logpoint: "Processing order {order.id}, total {order.total}"
  return order.items.reduce((sum, item) => sum + item.price, 0);
}
```

**Console Output (in DevTools):**
```
Processing order 123, total 0
Processing order 123, total 15.5
Processing order 123, total 22.75
```

### Real-World Cases

- **Production debugging:** Source maps + error monitoring.
- **Team collaboration:** Consistent debugging patterns.
- **Code review:** Removing debug statements.
- **Testing:** Regression tests from bugs.

---

## References

- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – The debugger Statement — https://tc39.es/ecma262/multipage/ecmascript-language-statements-and-declarations.html#sec-debugger-statement
- MDN Web Docs – Debugger — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/debugger
- MDN Web Docs – Debugging JavaScript — https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Scripting/Debugging_JavaScript
- Chrome DevTools – Debug JavaScript — https://developer.chrome.com/docs/devtools/javascript/
- Chrome DevTools – Sources panel overview — https://developer.chrome.com/docs/devtools/sources/
- Chrome DevTools – Console overview — https://developer.chrome.com/docs/devtools/console/
- Chrome DevTools – Pause your code with breakpoints — https://developer.chrome.com/docs/devtools/javascript/breakpoints/
- Chrome DevTools – Step over code — https://developer.chrome.com/docs/devtools/javascript/reference/
- Chrome DevTools – Watch expressions — https://developer.chrome.com/docs/devtools/javascript/watch/
- Chrome DevTools – View and edit local properties — https://developer.chrome.com/docs/devtools/javascript/reference/
- Firefox Developer Tools – Debugger — https://firefox-source-docs.mozilla.org/devtools-user/debugger/
- Safari Web Inspector – Debugging — https://developer.apple.com/safari/tools/
- Node.js Documentation – Debugger — https://nodejs.org/api/debugger.html
- VS Code Documentation – Debugging — https://code.visualstudio.com/docs/editor/debugging
- Chrome DevTools Protocol – Debugger Domain — https://chromedevtools.github.io/devtools-protocol/tot/Debugger/
- Chrome DevTools – Console API reference — https://developer.chrome.com/docs/devtools/console/api/
- JavaScript.info – Debugging in the browser — https://javascript.info/debugging-chrome
- Flaviocopes – The definitive guide to debugging JavaScript — https://flaviocopes.com/javascript-debugging/
- Google Web Fundamentals – Debugging — https://web.dev/learn/javascript
- MDN Web Docs – Console — https://developer.mozilla.org/en-US/docs/Web/API/console
- ESLint – no-debugger — https://eslint.org/docs/latest/rules/no-debugger
- ESLint – no-console — https://eslint.org/docs/latest/rules/no-console