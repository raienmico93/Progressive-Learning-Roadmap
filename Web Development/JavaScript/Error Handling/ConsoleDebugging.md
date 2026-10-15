# JavaScript Console Debugging — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Console debugging is the practice of using the browser or runtime `console` API to inspect program state, track execution flow, measure performance, and diagnose issues during development.

**Technical Definition:** Per the WHATWG Console Standard, the `console` object is a namespace providing methods for logging, grouping, timing, counting, and inspecting values. The standard defines `log`, `info`, `warn`, `error`, `debug`, `assert`, `clear`, `count`, `countReset`, `dir`, `dirxml`, `group`, `groupCollapsed`, `groupEnd`, `table`, `time`, `timeEnd`, `timeLog`, and `trace`. The `console` object is exposed on `globalThis` (window in browsers, global in Node.js). Output destination and formatting are implementation-defined; DevTools provides rich rendering for objects, tables, and styles. The `%c` directive enables CSS styling in browsers.

**Beginner-Friendly Explanation:** `console.log()` and friends are the simplest debugging tools. You print values to the console and read them. The console API goes far beyond `log`: you can print tables, group related messages, measure timing, count occurrences, and style output with CSS. It's fast, non-invasive, and works everywhere JavaScript runs.

### Key Characteristics

- **Standard API:** Defined by the WHATWG Console Standard.
- **Non-blocking:** Logging doesn't pause execution.
- **Rich rendering:** DevTools renders objects, arrays, DOM nodes interactively.
- **Multiple levels:** `log`, `info`, `warn`, `error`, `debug` (severity levels).
- **Grouping and timing:** Organise and measure output.
- **Styling:** `%c` directive for CSS in browsers.
- **Works everywhere:** Browsers, Node.js, Deno, Bun, workers.
- **Production caution:** Logs can leak data and slow performance.

### Prerequisites

- Basic JavaScript syntax (functions, objects, arrays).
- Familiarity with the browser DevTools Console.
- Understanding of values, types, and references.
- Awareness of production vs development environments.
- Basic knowledge of string formatting.

### Related Programming Areas

- **Debugging:** Breakpoints, stepping, inspection.
- **Logging frameworks:** Winston, Pino, Bunyan (Node.js).
- **Observability:** Structured logs, monitoring.
- **Performance profiling:** `console.time`, DevTools Performance.
- **Error handling:** `console.error` for diagnostics.
- **Testing:** Assertions with `console.assert`.

---

## `console.log()`

### Definitions

**Core Definition:** `console.log()` outputs one or more values to the console with automatic formatting.

**Technical Definition:** Per the Console Standard, `console.log(...data)` writes a message to the console using the `log` log level. Values are converted to strings for display, but DevTools may render them as interactive objects. In browsers, `%c` applies CSS styles, `%s`, `%d`, `%i`, `%f`, `%o`, `%O` are format specifiers.

**Beginner-Friendly Explanation:** `console.log()` is the most basic way to print something to the console. You can pass multiple values, and they'll all be shown.

### Purposes

- To inspect variable values.
- To trace code execution.
- To verify assumptions.
- To output formatted messages.

### Syntax Rules and Structure

```js
console.log(value);
console.log(value1, value2, ...);
console.log("format %s and %d", "string", 42);
```

**Format Specifiers (browsers):**
- `%s`: String
- `%d` / `%i`: Integer
- `%f`: Float
- `%o` / `%O`: Object
- `%c`: CSS style

### Annotated Code Examples

#### Example 1: Basic Logging

```js
console.log("Hello");
console.log("Name:", "Alice", "Age:", 30);
console.log({ name: "Alice", age: 30 });
console.log([1, 2, 3]);
```

**Expected Output (in Console):**
```
Hello
Name: Alice Age: 30
{name: "Alice", age: 30}
(3) [1, 2, 3]
```

#### Example 2: Format Specifiers

```js
console.log("User %s is %d years old", "Alice", 30);
// "User Alice is 30 years old"

console.log("Object: %o", { a: 1, b: 2 });
// "Object: {a: 1, b: 2}"
```

#### Example 3: Object Reference Trap

```js
const obj = { value: 1 };
console.log(obj); // Shows { value: 1 } initially
obj.value = 2;    // Console may show { value: 2 } (live reference)
console.log(JSON.stringify(obj)); // Snapshot: '{"value":2}'
```

**Note:** DevTools may store live references; use `JSON.parse(JSON.stringify(obj))` or `structuredClone(obj)` for snapshots.

### Real-World Cases

- **Quick inspection:** `console.log(user)`.
- **Flow tracing:** `console.log("step 1")`.
- **Conditional logging:** `if (debug) console.log(data)`.
- **Formatting:** `console.log("%cImportant", "color:red")`.

---

## `console.error()`

### Definitions

**Core Definition:** `console.error()` outputs a message at the error level, typically styled red and including a stack trace in DevTools.

**Technical Definition:** Per the Console Standard, `console.error(...data)` writes at the `error` log level. DevTools renders error-level messages in red and includes a stack trace showing where the call originated.

**Beginner-Friendly Explanation:** `console.error()` is for logging errors. It shows up in red and can include a stack trace.

### Purposes

- To log errors and exceptions.
- To make problems visually prominent.
- To capture stack traces for diagnostics.
- To integrate with error monitoring.

### Annotated Code Examples

```js
try {
  JSON.parse("invalid");
} catch (err) {
  console.error("Parse failed:", err);
}
```

**Expected Output (in Console):**
```
Parse failed: SyntaxError: Unexpected token 'i', "invalid" is not valid JSON
    at <anonymous>:3:8
```

### Real-World Cases

- **Exception logging:** `console.error(err)`.
- **Validation failures:** `console.error("Invalid input")`.
- **Monitoring integration:** Sentry hooks into `console.error`.

---

## `console.warn()`

### Definitions

**Core Definition:** `console.warn()` outputs a warning-level message, typically styled yellow.

**Technical Definition:** Per the Console Standard, `console.warn(...data)` writes at the `warn` log level. DevTools renders it in yellow with a warning icon.

**Beginner-Friendly Explanation:** `console.warn()` is for warnings — things that aren't errors but deserve attention.

### Annotated Code Examples

```js
if (!process.env.API_KEY) {
  console.warn("API_KEY is not set; using default");
}
```

**Expected Output (in Console):**
```
⚠️ API_KEY is not set; using default
```

### Real-World Cases

- **Deprecation warnings:** `console.warn("deprecated")`.
- **Missing configuration:** `console.warn("config missing")`.
- **Fallback notices:** `console.warn("using fallback")`.

---

## `console.info()`

### Definitions

**Core Definition:** `console.info()` outputs an informational message, typically at the same level as `log`.

**Technical Definition:** Per the Console Standard, `console.info(...data)` writes at the `info` log level. In most browsers, it behaves identically to `console.log`.

**Beginner-Friendly Explanation:** `console.info()` is like `console.log()` but semantically for informational messages.

### Annotated Code Examples

```js
console.info("Application started");
console.info("Version:", "1.2.3");
```

### Real-World Cases

- **Informational messages:** App lifecycle events.
- **Semantic logging:** Differentiate info from debug.

---

## `console.debug()`

### Definitions

**Core Definition:** `console.debug()` outputs a message at the debug level, often hidden by default in DevTools.

**Technical Definition:** Per the Console Standard, `console.debug(...data)` writes at the `debug` log level. In Chrome DevTools, debug messages are hidden unless the "Verbose" level is enabled.

**Beginner-Friendly Explanation:** `console.debug()` is for detailed debugging messages that you usually don't want to see.

### Annotated Code Examples

```js
console.debug("Loop iteration:", i, "value:", arr[i]);
```

**Note:** Enable "Verbose" in DevTools console levels to see debug output.

### Real-World Cases

- **Detailed traces:** Verbose logging.
- **Development-only:** Hidden in production.
- **Library internals:** Debug-level diagnostics.

---

## `console.table()`

### Definitions

**Core Definition:** `console.table()` displays tabular data as a table.

**Technical Definition:** Per the Console Standard, `console.table(data, columns)` renders arrays or objects as tables. If `data` is an array of objects, each row is an object and columns are properties. If `data` is an object, keys become rows. The optional `columns` argument restricts displayed columns.

**Beginner-Friendly Explanation:** `console.table()` shows arrays and objects as nice tables instead of nested text.

### Annotated Code Examples

```js
const users = [
  { id: 1, name: "Alice", age: 30 },
  { id: 2, name: "Bob", age: 25 },
  { id: 3, name: "Carol", age: 35 },
];

console.table(users);

console.table(users, ["name", "age"]); // Only name and age columns
```

**Expected Output (in Console):**
```
┌─────────┬────┬─────────┬─────┐
│ (index) │ id │ name    │ age │
├─────────┼────┼─────────┼─────┤
│ 0       │ 1  │ 'Alice' │ 30  │
│ 1       │ 2  │ 'Bob'   │ 25  │
│ 2       │ 3  │ 'Carol' │ 35  │
└─────────┴────┴─────────┴─────┘
```

### Real-World Cases

- **API responses:** Display arrays of records.
- **Data comparison:** Compare objects side by side.
- **Testing:** Verify table data.

---

## `console.group()`

### Definitions

**Core Definition:** `console.group()` starts an indented group of console messages.

**Technical Definition:** Per the Console Standard, `console.group(...data)` creates a new group. Subsequent messages are indented until `console.groupEnd()`. Groups can be nested.

**Beginner-Friendly Explanation:** `console.group()` groups related messages together, with indentation.

### Annotated Code Examples

```js
console.group("User Processing");
console.log("Name:", "Alice");
console.log("Age:", 30);
console.group("Address");
console.log("City:", "Paris");
console.log("Country:", "France");
console.groupEnd();
console.groupEnd();
```

**Expected Output (in Console):**
```
▼ User Processing
    Name: Alice
    Age: 30
  ▼ Address
      City: Paris
      Country: France
```

### Real-World Cases

- **Structured logs:** Group related operations.
- **Nested contexts:** Show hierarchy.
- **Request tracing:** Group by request ID.

---

## `console.groupCollapsed()`

### Definitions

**Core Definition:** `console.groupCollapsed()` is like `console.group()` but starts with the group collapsed.

**Technical Definition:** Per the Console Standard, `console.groupCollapsed(...data)` creates a group that is initially collapsed in the UI. DevTools may fall back to `group` if collapsed mode is unsupported.

**Beginner-Friendly Explanation:** `console.groupCollapsed()` creates a group that's collapsed by default, so it doesn't clutter the console.

### Annotated Code Examples

```js
console.groupCollapsed("Details");
console.log("Some detailed info");
console.log("More details");
console.groupEnd();
```

**Expected Output (in Console):**
```
▶ Details (click to expand)
```

### Real-World Cases

- **Verbose sections:** Hide details by default.
- **Large data:** Keep console readable.
- **Optional info:** Expand when needed.

---

## `console.groupEnd()`

### Definitions

**Core Definition:** `console.groupEnd()` closes the current console group.

**Technical Definition:** Per the Console Standard, `console.groupEnd()` decrements the group indentation level. Unmatched `groupEnd` calls are no-ops.

**Beginner-Friendly Explanation:** `console.groupEnd()` ends the current group started by `group()` or `groupCollapsed()`.

### Real-World Cases

- **Balanced groups:** Always pair with `group`/`groupCollapsed`.
- **Nested groups:** Match nesting depth.

---

## `console.time()`

### Definitions

**Core Definition:** `console.time()` starts a named timer for measuring elapsed time.

**Technical Definition:** Per the Console Standard, `console.time(label)` starts a timer with the given label. Labels are unique; starting an existing label is a no-op with a warning. Requires `console.timeEnd(label)` to stop and log the duration.

**Beginner-Friendly Explanation:** `console.time()` starts a stopwatch with a name. `console.timeEnd()` stops it and logs how long it took.

### Annotated Code Examples

```js
console.time("loop");
for (let i = 0; i < 1e6; i++) {}
console.timeEnd("loop"); // "loop: 5.23ms"
```

### Real-World Cases

- **Performance measurement:** Time functions.
- **Benchmarking:** Compare implementations.
- **Hot path analysis:** Identify slow sections.

---

## `console.timeEnd()`

### Definitions

**Core Definition:** `console.timeEnd()` stops a named timer and logs the elapsed time.

**Technical Definition:** Per the Console Standard, `console.timeEnd(label)` stops the timer and writes the label and duration at the `info` log level. If no timer with the label exists, a warning is logged.

**Beginner-Friendly Explanation:** `console.timeEnd()` stops the timer started by `console.time()` and prints the duration.

### Real-World Cases

- **Duration logging:** `console.timeEnd("fetch")`.
- **Conditional timing:** Start/stop around suspected slow code.

---

## `console.timeLog()`

### Definitions

**Core Definition:** `console.timeLog()` logs the current elapsed time of a named timer without stopping it.

**Technical Definition:** Per the Console Standard, `console.timeLog(label, ...data)` writes the current duration of the timer along with additional data.

**Beginner-Friendly Explanation:** `console.timeLog()` shows the elapsed time without stopping the timer.

### Annotated Code Examples

```js
console.time("process");
// ... step 1 ...
console.timeLog("process", "after step 1");
// ... step 2 ...
console.timeLog("process", "after step 2");
console.timeEnd("process");
```

**Expected Output (in Console):**
```
process: 120.5ms after step 1
process: 250.3ms after step 2
process: 380.1ms
```

### Real-World Cases

- **Checkpoints:** Log intermediate timings.
- **Multi-step profiling:** Track progress.

---

## `console.trace()`

### Definitions

**Core Definition:** `console.trace()` logs a stack trace to the console.

**Technical Definition:** Per the Console Standard, `console.trace(...data)` writes the data plus a stack trace at the `error` log level.

**Beginner-Friendly Explanation:** `console.trace()` shows you the call stack at the point where it was called.

### Annotated Code Examples

```js
function a() { b(); }
function b() { c(); }
function c() { console.trace("Trace from c"); }
a();
```

**Expected Output (in Console):**
```
Trace from c
    at c (<anonymous>:3:20)
    at b (<anonymous>:2:13)
    at a (<anonymous>:1:13)
    at <anonymous>:4:1
```

### Real-World Cases

- **Debugging flow:** See how code reached a point.
- **Unexpected calls:** Trace who called a function.
- **Async debugging:** Trace async callbacks.

---

## `console.assert()`

### Definitions

**Core Definition:** `console.assert()` logs an error if the assertion is falsy.

**Technical Definition:** Per the Console Standard, `console.assert(condition, ...data)` writes the data at the `error` log level if `condition` is falsy. If truthy, it does nothing.

**Beginner-Friendly Explanation:** `console.assert()` checks a condition and logs an error if it's false.

### Annotated Code Examples

```js
const user = { name: "Alice", age: 30 };

console.assert(user.age > 0, "Age must be positive");     // No output
console.assert(user.age > 100, "Age unusually high", user); // Error logged
```

**Expected Output (in Console):**
```
Assertion failed: Age unusually high {name: "Alice", age: 30}
```

### Real-World Cases

- **Invariant checks:** Ensure assumptions hold.
- **Development assertions:** Catch bugs early.
- **Data validation:** Verify input shape.

---

## `console.count()`

### Definitions

**Core Definition:** `console.count()` logs the number of times it has been called with a given label.

**Technical Definition:** Per the Console Standard, `console.count(label)` increments and logs a counter associated with the label (default `"default"`).

**Beginner-Friendly Explanation:** `console.count()` counts how many times a piece of code runs.

### Annotated Code Examples

```js
for (let i = 0; i < 3; i++) {
  console.count("loop");
}
console.count("other");
```

**Expected Output (in Console):**
```
loop: 1
loop: 2
loop: 3
other: 1
```

### Real-World Cases

- **Loop iterations:** Verify iteration counts.
- **Function calls:** Count invocations.
- **Conditional checks:** Count how often a branch runs.

---

## `console.countReset()`

### Definitions

**Core Definition:** `console.countReset()` resets the counter for a given label.

**Technical Definition:** Per the Console Standard, `console.countReset(label)` resets the counter to 0. If no counter exists, a warning is logged.

**Beginner-Friendly Explanation:** `console.countReset()` resets a counter to zero.

### Annotated Code Examples

```js
console.count("test"); // test: 1
console.count("test"); // test: 2
console.countReset("test");
console.count("test"); // test: 1
```

### Real-World Cases

- **Test isolation:** Reset counters between tests.
- **Batch processing:** Reset per batch.

---

## `console.dir()`

### Definitions

**Core Definition:** `console.dir()` displays an interactive list of an object's properties.

**Technical Definition:** Per the Console Standard, `console.dir(object, options)` displays an object's properties as an expandable tree. The `options` argument can specify `depth` and `showHidden`.

**Beginner-Friendly Explanation:** `console.dir()` shows an object's properties in an expandable tree view.

### Annotated Code Examples

```js
const obj = { a: 1, b: { c: 2, d: { e: 3 } } };

console.log(obj);      // Concise representation
console.dir(obj, { depth: null }); // Full expansion
```

### Real-World Cases

- **Deep objects:** Inspect nested properties.
- **DOM nodes:** See element properties.
- **Functions:** Inspect function properties.

---

## `console.dirxml()`

### Definitions

**Core Definition:** `console.dirxml()` displays an XML/HTML representation of an object.

**Technical Definition:** Per the Console Standard, `console.dirxml(...data)` writes an XML representation of each argument. For DOM elements, this shows the HTML tree.

**Beginner-Friendly Explanation:** `console.dirxml()` shows the HTML/XML representation of a DOM node.

### Annotated Code Examples

```js
const div = document.createElement("div");
div.innerHTML = "<p>Hello</p>";
console.dirxml(div);
// <div><p>Hello</p></div>
```

### Real-World Cases

- **DOM inspection:** View element HTML.
- **XML debugging:** Inspect XML documents.

---

## `console.clear()`

### Definitions

**Core Definition:** `console.clear()` clears the console output.

**Technical Definition:** Per the Console Standard, `console.clear()` clears the console if the environment supports it. In some environments, it may print a message instead.

**Beginner-Friendly Explanation:** `console.clear()` empties the console.

### Annotated Code Examples

```js
console.log("Before clear");
console.clear();
console.log("After clear");
```

### Real-World Cases

- **Fresh start:** Clear noisy output.
- **Development:** Reset between tests.

---

## Console Styling

### Definitions

**Core Definition:** Console styling uses the `%c` format directive to apply CSS styles to console output in browsers.

**Technical Definition:** In browsers, `console.log("%cMessage", "css")` applies the given CSS to the portion of the message following the `%c` directive. Multiple `%c` directives can apply different styles to different parts.

**Beginner-Friendly Explanation:** You can style console messages with CSS using `%c`.

### Annotated Code Examples

#### Example 1: Basic Styling

```js
console.log("%cHello", "color: red; font-size: 20px; font-weight: bold;");
```

#### Example 2: Multiple Styles

```js
console.log(
  "%cERROR%c Something went wrong",
  "color: white; background: red; padding: 2px 6px; border-radius: 3px;",
  "color: red;"
);
```

#### Example 3: Branded Log

```js
console.log(
  "%cMyApp%c v1.2.3",
  "color: #fff; background: #4CAF50; padding: 4px 8px; border-radius: 4px; font-weight: bold;",
  "color: #4CAF50; font-weight: bold;"
);
```

**Expected Output (in Console):** Styled "MyApp" badge followed by version.

### Real-World Cases

- **Branding:** Styled logos in console.
- **Warnings:** Emphasise critical messages.
- **Debugging:** Colour-code log levels.

---

## Console Best Practices

### Definitions

**Core Definition:** Console best practices are the conventions that keep console logging useful, performant, and safe.

**Technical Definition:** Best practices include removing logs before production, using appropriate log levels, avoiding logging sensitive data, using structured logging for machine parsing, leveraging DevTools features (logpoints, groups), and avoiding logging in hot loops.

**Beginner-Friendly Explanation:** Console logging is powerful, but you should use it wisely. Don't leave `console.log` everywhere, don't log passwords, and clean up before shipping.

### Best Practices

1. **Remove `console.log` before production.**
   - Use linters (`no-console`) or build tools (Terser `drop_console`).

2. **Use appropriate log levels.**
   - `log` for general, `warn` for warnings, `error` for errors, `debug` for verbose.

3. **Avoid logging sensitive data.**
   - Never log passwords, tokens, or PII.

4. **Use structured logging for machine parsing.**
   - Log objects, not concatenated strings.

5. **Prefer logpoints over `console.log`.**
   - Log without editing code (in DevTools).

6. **Group related messages.**
   - Use `console.group` and `console.groupEnd`.

7. **Time critical operations.**
   - Use `console.time`/`console.timeEnd`.

8. **Use `console.table` for arrays of objects.**
   - Easier to read than nested logs.

9. **Avoid logging in hot loops.**
   - Logging is slow; batch or sample.

10. **Use `console.trace` when you need the stack.**
    - Better than guessing where code runs.

11. **Snapshot objects when needed.**
    - `JSON.parse(JSON.stringify(obj))` for a copy.

12. **Use conditional logging.**
    - `if (DEBUG) console.log(...)`.

13. **Style important messages with `%c`.**
    - Highlight warnings and branding.

14. **Remember DevTools filters.**
    - Filter by level (Verbose, Info, Warnings, Errors).

### Annotated Code Examples

#### Example 1: Development-Only Logging

```js
const DEBUG = process.env.NODE_ENV !== "production";

function debugLog(...args) {
  if (DEBUG) console.log("[DEBUG]", ...args);
}

debugLog("User loaded", user); // Only in development
```

#### Example 2: Structured Logging

```js
function logEvent(event, data) {
  console.log(JSON.stringify({
    timestamp: new Date().toISOString(),
    event,
    ...data,
  }));
}

logEvent("user_login", { userId: 123, method: "password" });
```

#### Example 3: Removing Logs with Terser

```js
// terser.config.js
module.exports = {
  compress: {
    drop_console: true, // Remove all console.* in production
    pure_funcs: ["console.debug"], // Remove specific methods
  },
};
```

### Real-World Cases

- **Production builds:** Strip `console.*`.
- **Sentry:** Uses `console.error` for integration.
- **Structured logging:** JSON for log aggregators.
- **Linting:** ESLint `no-console` rule.

---

## References

- WHATWG Console Standard — https://console.spec.whatwg.org/
- MDN Web Docs – console — https://developer.mozilla.org/en-US/docs/Web/API/console
- MDN Web Docs – console.log() — https://developer.mozilla.org/en-US/docs/Web/API/console/log_static
- MDN Web Docs – console.error() — https://developer.mozilla.org/en-US/docs/Web/API/console/error_static
- MDN Web Docs – console.warn() — https://developer.mozilla.org/en-US/docs/Web/API/console/warn_static
- MDN Web Docs – console.info() — https://developer.mozilla.org/en-US/docs/Web/API/console/info_static
- MDN Web Docs – console.debug() — https://developer.mozilla.org/en-US/docs/Web/API/console/debug_static
- MDN Web Docs – console.table() — https://developer.mozilla.org/en-US/docs/Web/API/console/table_static
- MDN Web Docs – console.group() — https://developer.mozilla.org/en-US/docs/Web/API/console/group_static
- MDN Web Docs – console.groupCollapsed() — https://developer.mozilla.org/en-US/docs/Web/API/console/groupCollapsed_static
- MDN Web Docs – console.groupEnd() — https://developer.mozilla.org/en-US/docs/Web/API/console/groupEnd_static
- MDN Web Docs – console.time() — https://developer.mozilla.org/en-US/docs/Web/API/console/time_static
- MDN Web Docs – console.timeEnd() — https://developer.mozilla.org/en-US/docs/Web/API/console/timeEnd_static
- MDN Web Docs – console.timeLog() — https://developer.mozilla.org/en-US/docs/Web/API/console/timeLog_static
- MDN Web Docs – console.trace() — https://developer.mozilla.org/en-US/docs/Web/API/console/trace_static
- MDN Web Docs – console.assert() — https://developer.mozilla.org/en-US/docs/Web/API/console/assert_static
- MDN Web Docs – console.count() — https://developer.mozilla.org/en-US/docs/Web/API/console/count_static
- MDN Web Docs – console.countReset() — https://developer.mozilla.org/en-US/docs/Web/API/console/countReset_static
- MDN Web Docs – console.dir() — https://developer.mozilla.org/en-US/docs/Web/API/console/dir_static
- MDN Web Docs – console.dirxml() — https://developer.mozilla.org/en-US/docs/Web/API/console/dirxml_static
- MDN Web Docs – console.clear() — https://developer.mozilla.org/en-US/docs/Web/API/console/clear_static
- Chrome DevTools – Console API reference — https://developer.chrome.com/docs/devtools/console/api/
- Chrome DevTools – Console overview — https://developer.chrome.com/docs/devtools/console/
- Chrome DevTools – Log messages in the Console — https://developer.chrome.com/docs/devtools/console/log/
- Chrome DevTools – Console utilities API reference — https://developer.chrome.com/docs/devtools/console/utilities/
- ESLint – no-console — https://eslint.org/docs/latest/rules/no-console
- Terser – drop_console — https://terser.org/docs/options/#compress-options
- Node.js Documentation – Console — https://nodejs.org/api/console.html
- JavaScript.info – Debugging in the browser — https://javascript.info/debugging-chrome