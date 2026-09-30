# Node.js Utilities & Diagnostics — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** The `node:util` and `node:diagnostics_channel` modules provide a collection of utility functions for debugging, Promise/callback interoperability, object inspection, deprecation management, and low-overhead diagnostic messaging.

**Technical Definition:** The `node:util` module supports the needs of Node.js internal APIs, and many of the utilities are useful for application and module developers as well. The `node:diagnostics_channel` module provides an API to create named channels to report arbitrary message data for diagnostics purposes. Together these modules form Node.js's toolkit for development-time diagnostics and runtime observability.

**Beginner-Friendly Explanation:** Think of these modules as a developer's Swiss Army knife. `util` gives you tools to convert between old-style callbacks and modern Promises, pretty-print objects for debugging, and mark functions as "outdated." `diagnostics_channel` is like a private radio frequency that libraries can broadcast on — only interested listeners tune in, and if nobody is listening, the broadcast costs almost nothing.

### Key Characteristics

- **Interoperability bridge:** `util.promisify` and `util.callbackify` convert between Node.js's two dominant asynchronous programming styles.
- **Debugging without side effects:** `util.inspect` returns a string representation of any JavaScript value, designed for debugging and logging.
- **Deprecation lifecycle management:** `util.deprecate` wraps functions to emit `DeprecationWarning` messages, giving library authors a standard way to phase out APIs.
- **Zero-cost observability:** `diagnostics_channel` uses a publish-subscribe pattern where publishing to a channel with no subscribers has negligible overhead.
- **Pure and synchronous:** Most `util` methods are synchronous and do not perform I/O.

### Prerequisites

- **Node.js runtime:** Both modules are built into Node.js. `util` has been stable since v0.10.0; `diagnostics_channel` was introduced in Node.js v15.1.0.
- **Basic JavaScript knowledge:** Understanding of functions, Promises, and callbacks.
- **Familiarity with `require`/`import`:** Knowing how to import Node.js built-in modules.
- **Asynchronous programming concepts:** The event loop, error-first callbacks, and `async/await`.

### Related Programming Areas

- **File System (`fs`):** `util.promisify` is commonly applied to `fs` callback methods.
- **Streams:** `util.promisify` works with `stream.pipeline` and similar callback-based APIs.
- **Process management:** `util.deprecate` integrates with `process.on('warning')` and deprecation flags.
- **Performance monitoring:** `diagnostics_channel` is used by APM tools, OpenTelemetry, and Node.js core itself.
- **Testing:** `util.inspect` output is frequently used in assertion messages.

### Core Concepts

The following core concepts are covered in this cheat sheet:

1. **util Module Utilities** — the module import and general-purpose helpers.
2. **Modern Promisification and Callback-ification** — `util.promisify` and `util.callbackify`.
3. **Deep Object Inspection** — `util.inspect` with custom formatting.
4. **Advanced Debugging Helpers and Deprecation Wrappers** — `util.deprecate` and related diagnostic helpers.
5. **Diagnostic Tracking** — `diagnostics_channel` publish-subscribe.

---

## Core Concept 1: util Module Utilities

### Definitions

**Core Definition:** The `node:util` module is a collection of utility functions for formatting strings, inspecting objects, converting between asynchronous styles, and providing type-checking helpers.

**Technical Definition:** The `node:util` module supports the needs of Node.js internal APIs. Many of the utilities are useful for application and module developers as well. It can be accessed using `import util from 'node:util'` or `const util = require('node:util')`. The module includes functions such as `util.format`, `util.inspect`, `util.promisify`, `util.callbackify`, `util.deprecate`, and `util.parseArgs`。

**Beginner-Friendly Explanation:** The `util` module is Node.js's toolbox of handy helpers. When you need to format a string with placeholders, turn a callback into a Promise, or check if a value is a certain type, `util` has a function for it.

### Purposes

- To provide string formatting utilities that go beyond template literals.
- To convert between callback-based and Promise-based asynchronous APIs.
- To produce human-readable representations of objects for debugging.
- To implement deprecation warnings in a standardised way.
- To parse command-line arguments in a structured manner.

### Syntax Rules and Structure

**CommonJS import:**
```js
const util = require('node:util');
```
| Component | Breakdown |
|-----------|-----------|
| `require` | CommonJS import function. |
| `'node:util'` | Built-in module specifier. |
| Returns | The `util` module object. |

**ESM import:**
```js
import util from 'node:util';
```
| Component | Breakdown |
|-----------|-----------|
| `import util` | Default import. |
| `from 'node:util'` | Module specifier. |

**Constraints and Limitations:**
- Many type-checking methods (`util.isArray`, `util.isString`, `util.isObject`, etc.) are deprecated or removed. Use native JavaScript alternatives: `Array.isArray()`, `typeof`, `value === null`。
- `util.format` is a synchronous method intended mainly as a debugging tool; some input values can have significant performance overhead that can block the event loop。

### Annotated Code Example

```js
// util-basics.js
const util = require('node:util');

// util.format: printf-like string formatting
console.log(util.format('%s has %d apples', 'Alice', 5));
// → 'Alice has 5 apples'

// util.format with object inspection
console.log(util.format('Object: %o', { a: 1, b: [2, 3] }));
// → 'Object: { a: 1, b: [ 2, 3 ] }'

// util.parseArgs: structured command-line argument parsing
const { parseArgs } = require('node:util');
const args = ['-f', '--bar', 'b'];
const options = {
  foo: { type: 'boolean', short: 'f' },
  bar: { type: 'string' },
};
const { values, positionals } = parseArgs({ args, options });
console.log(values, positionals);
// → [Object: null prototype] { foo: true, bar: 'b' } []
```

**Expected Output:**
```
Alice has 5 apples
Object: { a: 1, b: [ 2, 3 ] }
[Object: null prototype] { foo: true, bar: 'b' } []
```

**Why this output:** `util.format` replaces `%s` with a string and `%d` with a number. The `%o` specifier uses `util.inspect` to format the object. `util.parseArgs` interprets `-f` as a boolean flag and `--bar` as a string option, returning structured `values` and `positionals`。

### Real-World Cases

- **CLI tools:** Using `util.parseArgs` for structured argument parsing without external dependencies.
- **Logging:** Using `util.format` to build log messages with mixed string and object content.
- **Migration:** Replacing deprecated `util.is*` methods with native JavaScript equivalents.

---

## Core Concept 2: Modern Promisification and Callback-ification

### Sub-Feature 2.1: `util.promisify`

#### Definitions

**Core Definition:** `util.promisify()` converts an error-first callback function into a version that returns a Promise.

**Technical Definition:** The `util.promisify(original)` method takes a function following the common error-first callback style — i.e., taking an `(err, value) => ...` callback as the last argument — and returns a version that returns Promises. `promisify()` assumes that `original` is a function taking a callback as its final argument in all cases. If `original` is not a function, `promisify()` will throw an error。

**Beginner-Friendly Explanation:** Many older Node.js functions use callbacks: you pass a function that gets called with `(error, result)` when the work is done. Promises and `async/await` are more modern and easier to read. `util.promisify` is a translator: it takes an old callback-style function and gives you back a new version that you can `await`。

#### Purposes

- To modernise legacy callback-based APIs for use with `async/await`.
- To avoid manually wrapping functions in `new Promise()` constructors.
- To enable uniform error handling through `try/catch` and Promise rejection.
- To convert `fs`, `dns`, `crypto`, and other core callback APIs for modern codebases.

#### Syntax Rules and Structure

```js
const promisified = util.promisify(originalFunction);
```
| Component | Breakdown |
|-----------|-----------|
| `originalFunction` | A function taking an error-first callback as its last argument. |
| Returns | A new function that returns a Promise. |
| `promisify.custom` | A symbol that allows overriding the promisified version. |

**Constraints and Limitations:**
- `promisify()` assumes the last argument is an error-first callback. Non-standard callback signatures will not work correctly。
- Using `promisify()` on class methods that use `this` may not work as expected unless the method is bound to the instance。
- Custom promisified functions can be defined using the `util.promisify.custom` symbol。

#### Annotated Code Example

```js
// promisify-example.js
const util = require('node:util');
const fs = require('node:fs');

// Promisify the callback-based fs.stat
const stat = util.promisify(fs.stat);

(async () => {
  try {
    const stats = await stat('.');
    console.log('Directory stats retrieved successfully.');
    console.log('Is directory:', stats.isDirectory());
  } catch (err) {
    console.error('Error:', err.message);
  }
})();

// Without promisify, the callback style would be:
// fs.stat('.', (err, stats) => {
//   if (err) throw err;
//   console.log(stats.isDirectory());
// });
```

**Expected Output:**
```
Directory stats retrieved successfully.
Is directory: true
```

**Why this output:** `util.promisify(fs.stat)` returns a new function that, when called, returns a Promise. The Promise resolves with the `stats` object (the same value the callback would have received as its second argument) or rejects with the error. The `await` keyword pauses execution until the Promise settles.

#### Using `util.promisify.custom`

```js
// promisify-custom.js
const util = require('node:util');

function doSomething(foo, callback) {
  // Non-standard callback: only returns a value, no error
  callback(foo * 2);
}

// Define a custom promisified version
doSomething[util.promisify.custom] = (foo) => {
  return new Promise((resolve) => {
    resolve(foo * 2);
  });
};

const promisified = util.promisify(doSomething);
promisified(21).then((result) => console.log(result));
// → 42
```

**Expected Output:**
```
42
```

**Why this output:** Because `doSomething` does not follow the error-first callback convention (its callback receives only a value, no error), the default `util.promisify` would not work correctly. By defining `doSomething[util.promisify.custom]`, we provide a proper Promise-returning implementation that `promisify` will use instead。

### Sub-Feature 2.2: `util.callbackify`

#### Definitions

**Core Definition:** `util.callbackify()` converts an `async` function (or a function returning a Promise) into a function following the error-first callback style.

**Technical Definition:** The `util.callbackify(original)` method takes an `async` function (or a function that returns a Promise) and returns a function following the error-first callback style, i.e., taking an `(err, value) => ...` callback as the last argument. In the callback, the first argument will be the rejection reason (or `null` if the Promise resolved), and the second argument will be the resolved value。

**Beginner-Friendly Explanation:** `callbackify` is the opposite of `promisify`. If you have a modern `async` function and need to pass it to an older API that expects a callback, `callbackify` wraps it for you. It's useful when integrating with legacy code that hasn't been updated to Promises.

#### Purposes

- To integrate Promise-based functions with callback-based APIs.
- To provide backwards-compatible interfaces for library consumers using callbacks.
- To bridge modern and legacy code during incremental migration.

#### Syntax Rules and Structure

```js
const callbackFn = util.callbackify(asyncFunction);
callbackFn((err, result) => { /* ... */ });
```
| Component | Breakdown |
|-----------|-----------|
| `asyncFunction` | An `async` function or a function returning a Promise. |
| `callbackFn` | A function taking an error-first callback. |
| Callback args | `(err, result)` — `err` is `null` on success. |

**Constraints and Limitations:**
- The callback is executed asynchronously and will have a limited stack trace。
- If the Promise rejects with a falsy value, it is wrapped in an `Error` object with the original value stored in the `reason` property.

#### Annotated Code Example

```js
// callbackify-example.js
const util = require('node:util');

async function fetchData() {
  // Simulate an async operation
  return { id: 1, name: 'Alice' };
}

// Convert to callback style
const fetchDataCallback = util.callbackify(fetchData);

fetchDataCallback((err, data) => {
  if (err) {
    console.error('Error:', err);
    return;
  }
  console.log('Data received:', data);
});
```

**Expected Output:**
```
Data received: { id: 1, name: 'Alice' }
```

**Why this output:** `util.callbackify` wraps the async function. When the Promise resolves, the callback is invoked with `null` as the error and the resolved value as the second argument. If the Promise had rejected, the callback would receive the rejection reason as the error.

### Real-World Cases

- **Modernising `fs` calls:** `const readFile = util.promisify(fs.readFile);` — the most common use case.
- **Library compatibility:** Providing both Promise and callback interfaces for library consumers.
- **Testing:** Wrapping async test helpers for use with callback-based test runners.
- **Gradual migration:** Using `callbackify` to integrate new async code into a legacy callback codebase.

---

## Core Concept 3: Deep Object Inspection (`util.inspect`)

### Definitions

**Core Definition:** `util.inspect()` returns a string representation of a JavaScript object, designed for debugging and logging.

**Technical Definition:** The `util.inspect(object, options)` method returns a string representation of `object` that is intended for debugging. The output may change at any time and should not be relied upon programmatically. Additional options can be passed to alter the resulting string, including `showHidden`, `depth`, `colors`, `customInspect`, `maxArrayLength`, `maxStringLength`, `breakLength`, and `compact`。

**Beginner-Friendly Explanation:** When you `console.log` an object, Node.js uses `util.inspect` under the hood to decide how to display it. But `util.inspect` gives you fine-grained control: how deep to go into nested objects, whether to show hidden properties, whether to use colours, and how to format custom objects.

### Purposes

- To produce human-readable representations of complex objects for debugging.
- To control the depth and detail of object output.
- To enable ANSI colour output for terminal-based debugging.
- To allow objects to define their own inspection format via `util.inspect.custom`。

### Syntax Rules and Structure

```js
util.inspect(object, { depth, colors, showHidden, customInspect, maxArrayLength });
```
| Option | Breakdown |
|--------|-----------|
| `depth` | Number of times to recurse; `Infinity` or `null` for full depth. Default: 2. |
| `colors` | `true` to style output with ANSI colour codes. Default: `false`. |
| `showHidden` | `true` to include non-enumerable symbols and properties. Default: `false`. |
| `customInspect` | `false` to disable custom inspect functions. Default: `true`. |
| `maxArrayLength` | Maximum array elements to display; `null`/`Infinity` for all. Default: 100. |
| `breakLength` | Line length at which values split across multiple lines. Default: 80. |

**Custom inspection via `util.inspect.custom`:**
```js
class MyClass {
  [util.inspect.custom](depth, opts, inspect) {
    return `MyClass(${this.value})`;
  }
}
```
| Component | Breakdown |
|-----------|-----------|
| `util.inspect.custom` | A globally registered symbol (`Symbol.for('nodejs.util.inspect.custom')`). |
| `depth` | Current recursion depth. |
| `opts` | The options object passed to `util.inspect`. |
| `inspect` | A reference to `util.inspect` for recursive calls. |

**Constraints and Limitations:**
- The output format is not guaranteed stable across Node.js versions; do not parse it programmatically.
- `util.inspect` is synchronous; inspecting very large or deeply nested objects can block the event loop.
- Custom inspect functions are not invoked if `customInspect` is set to `false`。

### Multiple Annotated Code Examples

#### Example 1: Controlling Depth and Output

```js
// inspect-depth.js
const util = require('node:util');

const nested = {
  level1: {
    level2: {
      level3: {
        value: 'deep',
      },
    },
  },
};

// Default depth is 2
console.log('Default depth:');
console.log(util.inspect(nested));
// → { level1: { level2: [Object] } }

// Unlimited depth
console.log('Infinite depth:');
console.log(util.inspect(nested, { depth: null }));
// → { level1: { level2: { level3: { value: 'deep' } } } }
```

**Expected Output:**
```
Default depth:
{ level1: { level2: [Object] } }
Infinite depth:
{ level1: { level2: { level3: { value: 'deep' } } } }
```

**Why this output:** The default `depth: 2` means `util.inspect` recurses two levels deep, replacing deeper objects with `[Object]`. Setting `depth: null` (or `Infinity`) removes the limit, showing the full structure.

#### Example 2: Custom Inspection with `util.inspect.custom`

```js
// custom-inspect.js
const util = require('node:util');

class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }

  // Define custom inspection format
  [util.inspect.custom](depth, opts, inspect) {
    return `Point(${this.x}, ${this.y})`;
  }
}

const p = new Point(3, 4);
console.log(util.inspect(p));
// → Point(3, 4)

// Without custom inspect, it would show:
// → Point { x: 3, y: 4 }
```

**Expected Output:**
```
Point(3, 4)
```

**Why this output:** When `util.inspect` encounters an object with a `[util.inspect.custom]` method, it calls that method instead of the default formatting. The custom method returns a string that replaces the default representation.

#### Example 3: Colour Output and Array Truncation

```js
// inspect-colors.js
const util = require('node:util');

const data = {
  name: 'Alice',
  scores: [95, 87, 92, 78, 88, 91, 85, 90, 93, 86, 89, 94, 82, 87, 91],
};

// Show with colours (visible in terminal)
console.log(util.inspect(data, { colors: true, compact: false }));

// Limit array display
console.log(util.inspect(data, { maxArrayLength: 3 }));
// → { name: 'Alice', scores: [ 95, 87, 92, ... 12 more items ] }
```

**Expected Output (without colour rendering):**
```
{
  name: 'Alice',
  scores: [
    95, 87, 92, 78, 88, 91, 85,
    90, 93, 86, 89, 94, 82, 87,
    91
  ]
}
{ name: 'Alice', scores: [ 95, 87, 92, ... 12 more items ] }
```

**Why this output:** `compact: false` forces each array element onto its own line when `breakLength` is exceeded. `maxArrayLength: 3` truncates the array display after three elements, showing `... 12 more items` to indicate the remainder.

### Real-World Cases

- **Debugging complex state:** Inspecting deeply nested configuration or state objects.
- **Custom class representation:** Giving domain objects meaningful `toString`-like output.
- **Test assertions:** Using `util.inspect` in assertion error messages for readable diffs.
- **Log formatting:** Producing structured, human-readable log output.

---

## Core Concept 4: Advanced Debugging Helpers and Deprecation Wrappers (`util.deprecate`)

### Definitions

**Core Definition:** `util.deprecate()` wraps a function or class so that a `DeprecationWarning` is emitted the first time it is called.

**Technical Definition:** The `util.deprecate(fn, msg, code?, options?)` method wraps `fn` (which may be a function or class) in such a way that it is marked as deprecated. When called, `util.deprecate()` will return a function that will emit a `DeprecationWarning` using the `'warning'` event. The warning will be emitted and printed to stderr the first time the returned function is called. After the warning is emitted, the wrapped function is called without emitting a warning。

**Beginner-Friendly Explanation:** When a library author wants to tell users "this function is old, please stop using it," they wrap it with `util.deprecate`. The first time someone calls the old function, Node.js prints a warning message. After that, it stays quiet — so users aren't spammed with warnings on every call.

### Purposes

- To signal to consumers that a function or class is outdated and will be removed.
- To provide migration guidance in the warning message.
- To suppress repeated warnings using a shared deprecation code.
- To integrate with command-line flags (`--no-deprecation`, `--throw-deprecation`) for testing.

### Syntax Rules and Structure

```js
const deprecatedFn = util.deprecate(fn, message, code, options);
```
| Component | Breakdown |
|-----------|-----------|
| `fn` | The function or class to deprecate. |
| `message` | Warning message shown when the deprecated function is called. |
| `code` | Optional deprecation code (e.g., `'DEP0001'`). |
| `options.modifyPrototype` | When `false`, do not change the prototype. Default: `true`. |

**Command-line flags:**
| Flag | Effect |
|------|--------|
| `--no-deprecation` | Suppresses all deprecation warnings. |
| `--trace-deprecation` | Prints a stack trace with the warning. |
| `--throw-deprecation` | Throws an exception on deprecated function calls. |

**Constraints and Limitations:**
- The warning is emitted only once per function by default. If the same `code` is supplied in multiple calls to `util.deprecate()`, the warning is emitted only once for that code。
- If `--no-deprecation` or `--no-warnings` flags are used, `util.deprecate()` does nothing。
- `--throw-deprecation` takes precedence over `--trace-deprecation`。

### Annotated Code Example

```js
// deprecate-example.js
const util = require('node:util');

// Old function that we want to deprecate
function oldFetch(url, callback) {
  callback(null, { url, data: 'legacy' });
}

// Wrap it with deprecation warning
const deprecatedFetch = util.deprecate(
  oldFetch,
  'oldFetch() is deprecated. Use fetchData() instead.',
  'DEP0001'
);

// First call — emits warning and calls the function
deprecatedFetch('https://api.example.com', (err, result) => {
  console.log('Result:', result);
});

// Second call — function runs but no warning
deprecatedFetch('https://api.example.com', (err, result) => {
  console.log('Second result:', result);
});
```

**Expected Output (stderr + stdout):**
```
(node:12345) DeprecationWarning: oldFetch() is deprecated. Use fetchData() instead.
(Use `node --trace-deprecation ...` to show where the warning was created)
Result: { url: 'https://api.example.com', data: 'legacy' }
Second result: { url: 'https://api.example.com', data: 'legacy' }
```

**Why this output:** The first call to `deprecatedFetch` emits the `DeprecationWarning` to stderr and then invokes the original function. The second call invokes the function directly without emitting a warning, because the warning is only emitted once per function.

### Real-World Cases

- **Library API evolution:** Deprecating old method names in favour of new ones.
- **Node.js core:** Node.js itself uses this mechanism for its own deprecated APIs.
- **Framework migrations:** Guiding users from v1 APIs to v2 APIs with clear messages.
- **Testing deprecations:** Using `--throw-deprecation` in CI to catch usage of deprecated functions during test runs.

---

## Core Concept 5: Diagnostic Tracking via `diagnostics_channel`

### Definitions

**Core Definition:** The `node:diagnostics_channel` module provides an API to create named channels for reporting arbitrary message data for diagnostics purposes.

**Technical Definition:** The `node:diagnostics_channel` module provides an API to create named channels to report arbitrary message data for diagnostics purposes. It is intended that a module writer wanting to report diagnostics messages will create one or many top-level channels to report messages through. Channels may also be acquired at runtime but it is not encouraged due to the additional overhead of doing so. Channels are designed as a pub/sub system where publishers broadcast data and subscribers receive it, with near-zero overhead when there are no subscribers.

**Beginner-Friendly Explanation:** Imagine your application has different parts — a database layer, an HTTP layer, a caching layer — and each part wants to announce what it's doing. `diagnostics_channel` lets each part create a named "radio channel" and broadcast messages. Monitoring tools can tune in to whichever channels they care about. If nobody is listening, the broadcast costs nothing, so it's safe to leave the broadcasts in your code permanently.

### Purposes

- To enable low-overhead observability in libraries and applications.
- To allow monitoring tools to subscribe to specific diagnostic events without modifying the library.
- To provide a standard mechanism for publishing structured diagnostic data.
- To avoid the performance cost of building diagnostic data when no one is listening.

### Syntax Rules and Structure

**Acquiring a channel:**
```js
const diagnostics_channel = require('node:diagnostics_channel');
const channel = diagnostics_channel.channel('my-channel');
```
| Component | Breakdown |
|-----------|-----------|
| `channel(name)` | Returns a reusable `Channel` object for the given name. |
| `name` | String or symbol identifying the channel. |

**Subscribing to a channel:**
```js
diagnostics_channel.subscribe('my-channel', onMessage);
```
| Component | Breakdown |
|-----------|-----------|
| `onMessage(message, name)` | Callback invoked when a message is published. |
| `message` | The data published to the channel. |
| `name` | The channel name. |

**Publishing to a channel:**
```js
if (channel.hasSubscribers) {
  channel.publish({ some: 'data' });
}
```
| Component | Breakdown |
|-----------|-----------|
| `hasSubscribers` | Boolean indicating whether anyone is subscribed. |
| `publish(data)` | Sends data to all subscribers synchronously. |

**Constraints and Limitations:**
- Message handlers are executed **synchronously** in the same context; a slow handler blocks the publisher。
- Channel names should include the module name to avoid collisions with other modules。
- Publishing without checking `hasSubscribers` is wasteful if the message data is expensive to prepare。
- The `TracingChannel` class provides a higher-level abstraction for tracing operations.

### Multiple Annotated Code Examples

#### Example 1: Basic Publish-Subscribe

```js
// diagnostics-basic.js
const diagnostics_channel = require('node:diagnostics_channel');

// Get a channel object
const channel = diagnostics_channel.channel('my-app:database');

// Define a message handler
function onMessage(message, name) {
  console.log(`[${name}] Query executed:`, message.query);
}

// Subscribe to the channel
diagnostics_channel.subscribe('my-app:database', onMessage);

// Publish a message (only if there are subscribers)
if (channel.hasSubscribers) {
  channel.publish({ query: 'SELECT * FROM users', duration: 42 });
}

// Unsubscribe
diagnostics_channel.unsubscribe('my-app:database', onMessage);
```

**Expected Output:**
```
[my-app:database] Query executed: SELECT * FROM users
```

**Why this output:** The subscriber registers a handler for the `'my-app:database'` channel. When `publish()` is called, the handler is invoked synchronously with the message object. The `hasSubscribers` check ensures the message is only prepared when someone is listening.

#### Example 2: Guarding Expensive Message Preparation

```js
// diagnostics-guard.js
const diagnostics_channel = require('node:diagnostics_channel');

const channel = diagnostics_channel.channel('my-app:slow-operation');

function performOperation(data) {
  // Only prepare the expensive diagnostic message if someone is listening
  if (channel.hasSubscribers) {
    const expensiveDiagnostic = {
      timestamp: new Date().toISOString(),
      memoryUsage: process.memoryUsage(),
      inputHash: require('node:crypto')
        .createHash('sha256')
        .update(data)
        .digest('hex'),
    };
    channel.publish(expensiveDiagnostic);
  }

  // Actual operation logic
  return data.toUpperCase();
}

// No subscribers — the expensive diagnostic is never prepared
console.log(performOperation('hello'));
// → 'HELLO'

// With a subscriber
diagnostics_channel.subscribe('my-app:slow-operation', (msg) => {
  console.log('Diagnostic received, hash length:', msg.inputHash.length);
});

console.log(performOperation('world'));
// → Diagnostic received, hash length: 64
// → 'WORLD'
```

**Expected Output:**
```
HELLO
Diagnostic received, hash length: 64
WORLD
```

**Why this output:** When no subscribers are present, the `if (channel.hasSubscribers)` block is skipped entirely, so the expensive hash computation and memory usage snapshot are never created. Once a subscriber is added, the diagnostic is prepared and published. This demonstrates the zero-cost-when-unused design。

### Real-World Cases

- **APM tools:** Application Performance Monitoring tools subscribe to channels published by database drivers, HTTP clients, and caches.
- **OpenTelemetry:** Uses `diagnostics_channel` to instrument Node.js core modules.
- **Debugging production issues:** Temporarily subscribing to diagnostic channels to trace specific operations without redeploying.
- **Library instrumentation:** Libraries publish diagnostic events that consumers can subscribe to for custom monitoring.

---

## References

- Node.js Documentation — Util — https://nodejs.org/api/util.html
- Node.js Documentation — `util.promisify` — https://nodejs.org/api/util.html#utilpromisifyoriginal
- Node.js Documentation — `util.callbackify` — https://nodejs.org/api/util.html#utilcallbackifyoriginal
- Node.js Documentation — `util.inspect` — https://nodejs.org/api/util.html#utilinspectobject-options
- Node.js Documentation — `util.inspect.custom` — https://nodejs.org/api/util.html#utilinspectcustom
- Node.js Documentation — `util.deprecate` — https://nodejs.org/api/util.html#utildeprecatefn-msg-code
- Node.js Documentation — `util.format` — https://nodejs.org/api/util.html#utilformatformat-args
- Node.js Documentation — `util.parseArgs` — https://nodejs.org/api/util.html#utilparseargsconfig
- Node.js Documentation — Diagnostics Channel — https://nodejs.org/api/diagnostics_channel.html
- Node.js Documentation — `diagnostics_channel.channel` — https://nodejs.org/api/diagnostics_channel.html#diagnostics_channelchannelname
- Node.js Documentation — `diagnostics_channel.subscribe` — https://nodejs.org/api/diagnostics_channel.html#diagnostics_channelsubscribename-onmessage
- Node.js Documentation — `diagnostics_channel.Channel.hasSubscribers` — https://nodejs.org/api/diagnostics_channel.html#channelhassubscribers
- Node.js Documentation — `diagnostics_channel.Channel.publish` — https://nodejs.org/api/diagnostics_channel.html#channelpublishmessage
- Node.js Documentation — Deprecated APIs — https://nodejs.org/api/deprecations.html
- Node.js Documentation — `process` warning event — https://nodejs.org/api/process.html#event-warning
- Node.js Documentation — `TracingChannel` — https://nodejs.org/api/diagnostics_channel.html#class-tracingchannel