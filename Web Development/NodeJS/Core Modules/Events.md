# Node.js Events — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** The `node:events` module provides the `EventEmitter` class, which is the foundation of Node.js's event-driven architecture, enabling objects to emit named events that trigger associated listener functions.

**Technical Definition:** Much of the Node.js core API is built around an idiomatic asynchronous event-driven architecture in which certain kinds of objects (called "emitters") emit named events that cause `Function` objects ("listeners") to be called. All objects that emit events are instances of the `EventEmitter` class, which exposes an `eventEmitter.on()` function that allows one or more functions to be attached to named events emitted by the object. When the `EventEmitter` object emits an event, all of the functions attached to that specific event are called synchronously, and any values returned by the called listeners are ignored and discarded.

**Beginner-Friendly Explanation:** Imagine a radio station and its listeners. The radio station (the EventEmitter) broadcasts different shows (events) like "morning news" or "evening music." Listeners (callback functions) tune in to specific shows — when the station broadcasts "morning news," only the people listening to that show react. The `EventEmitter` class lets your JavaScript objects do the same thing: broadcast named signals that other parts of your program can react to.

### Key Characteristics

- **Synchronous dispatch:** Listeners are called synchronously in the order they were registered when an event is emitted.
- **Named events:** Event names are typically camel-cased strings, but any valid JavaScript property key can be used.
- **`this` binding:** When an ordinary listener function is called, the standard `this` keyword is intentionally set to reference the `EventEmitter` instance to which the listener is attached.
- **Error event is special:** If an `EventEmitter` does not have at least one listener registered for the `'error'` event and an `'error'` event is emitted, the error is thrown, a stack trace is printed, and the Node.js process exits.
- **Max listeners warning:** By default, a maximum of 10 listeners can be registered for any single event; exceeding this outputs a warning to stderr indicating a "possible EventEmitter memory leak" has been detected.
- **Introspection events:** All EventEmitters emit the `'newListener'` event when new listeners are added and `'removeListener'` when existing listeners are removed.

### Prerequisites

- **Node.js runtime:** The `events` module is built into Node.js; no external installation is required. It has been stable since v0.10.0.
- **Basic JavaScript knowledge:** Understanding of functions, callbacks, and the `this` keyword.
- **Familiarity with `require`/`import`:** Knowing how to import Node.js built-in modules.
- **Asynchronous programming concepts:** The event loop and callback-based execution.

### Related Programming Areas

- **Streams:** All streams (`fs`, `net`, `http`) are EventEmitters.
- **HTTP servers:** `http.Server` emits `'request'`, `'connection'`, and `'close'` events.
- **Process management:** `process` is an EventEmitter emitting `'exit'`, `'uncaughtException'`, and signal events.
- **Child processes:** `child_process` instances emit `'exit'`, `'error'`, and data events.
- **Design patterns:** Observer pattern, publish-subscribe, and reactive programming.

### Core Concepts

The following core concepts are covered in this cheat sheet:

1. **EventEmitter Class Fundamentals** — the module import and basic usage.
2. **Registering Listeners (`on`, `once`) and Emitting Custom Events** — `on()`, `once()`, `emit()`.
3. **Removing Listeners and Managing Memory Leaks** — `removeListener()`, `off()`, `removeAllListeners()`, `setMaxListeners()`.
4. **Error Event Handling Conventions** — the special `'error'` event.
5. **Architecture Patterns: Designing Custom Event-Driven Classes** — extending `EventEmitter`.

---

## Core Concept 1: EventEmitter Class Fundamentals

### Definitions

**Core Definition:** The `EventEmitter` class is Node.js's implementation of the observer pattern, allowing objects to emit named events and register listener functions that respond to those events.

**Technical Definition:** The `EventEmitter` class is defined and exposed by the `events` module. All objects that emit events are instances of the `EventEmitter` class. These objects expose an `eventEmitter.on()` function that allows one or more functions to be attached to named events emitted by the object. The `EventEmitter` class is used throughout both Node's built-in modules and third-party modules.

**Beginner-Friendly Explanation:** The `EventEmitter` is like a walkie-talkie system. You create a walkie-talkie (an emitter), and people can tune into specific channels (events) by pressing the listen button (`on`). When someone broadcasts on a channel (`emit`), everyone tuned into that channel hears the message and reacts.

### Purposes

- To enable loose coupling between components through event-based communication.
- To provide a standardised mechanism for asynchronous notifications within Node.js applications.
- To serve as the foundation for streams, servers, and other core Node.js APIs.
- To allow objects to broadcast state changes without knowing who is listening.

### Syntax Rules and Structure

**CommonJS import:**
```js
const { EventEmitter } = require('node:events');
```
| Component | Breakdown |
|-----------|-----------|
| `require` | CommonJS import function. |
| `'node:events'` | Built-in module specifier. |
| `{ EventEmitter }` | Destructured class from the module. |

**ESM import:**
```js
import { EventEmitter } from 'node:events';
```
| Component | Breakdown |
|-----------|-----------|
| `import` | ESM import keyword. |
| `{ EventEmitter }` | Named import. |

**Creating an emitter:**
```js
const emitter = new EventEmitter();
```
| Component | Breakdown |
|-----------|-----------|
| `new EventEmitter()` | Creates a new emitter instance. |

**Constraints and Limitations:**
- Event names must be strings or symbols; other types will be coerced or throw errors.
- Listeners are called synchronously in registration order; a slow listener blocks subsequent listeners.
- The `EventEmitter` does not provide built-in backpressure or flow control.

### Annotated Code Example

```js
// basic-emitter.js
const { EventEmitter } = require('node:events');

// Create an emitter instance
const myEmitter = new EventEmitter();

// Register a listener for the 'greet' event
myEmitter.on('greet', (name) => {
  console.log(`Hello, ${name}!`);
});

// Emit the 'greet' event with an argument
myEmitter.emit('greet', 'Alice');
// → 'Hello, Alice!'

// Emit the same event again
myEmitter.emit('greet', 'Bob');
// → 'Hello, Bob!'
```

**Expected Output:**
```
Hello, Alice!
Hello, Bob!
```

**Why this output:** The `on()` method registers a listener for the `'greet'` event. Each `emit()` call synchronously invokes the listener with the provided argument. The listener runs once per emit call.

### Real-World Cases

- **HTTP servers:** `http.Server` extends `EventEmitter` and emits `'request'` on each incoming request.
- **Streams:** `fs.ReadStream` emits `'data'` when chunks are available and `'end'` when the file is fully read.
- **Process monitoring:** `process` emits `'exit'`, `'uncaughtException'`, and signal events.

---

## Core Concept 2: Registering Listeners (`on`, `once`) and Emitting Custom Events

### Definitions

**Core Definition:** `on()` registers a persistent listener for an event, `once()` registers a one-time listener that is removed after its first invocation, and `emit()` triggers all listeners registered for a given event.

**Technical Definition:** The `eventEmitter.on(eventName, listener)` method adds the `listener` function to the end of the listeners array for the event named `eventName`. No checks are made to see if the `listener` has already been added; multiple calls passing the same combination of `eventName` and `listener` will result in the `listener` being added and called multiple times. The `eventEmitter.once(eventName, listener)` method adds a one-time `listener` function for the event named `eventName`. The next time `eventName` is triggered, this listener is removed and then invoked. The `eventEmitter.emit(eventName[, ...args])` method synchronously calls each of the listeners registered for the event named `eventName`, in the order they were registered, passing the supplied arguments to each.

**Beginner-Friendly Explanation:** `on()` is like subscribing to a magazine — you get every issue. `once()` is like ordering a single copy — you get it once and then you're automatically unsubscribed. `emit()` is the act of publishing the magazine — everyone subscribed gets their copy.

### Purposes

- To register listener functions that respond to specific events.
- To create one-time handlers for events that should only trigger once (e.g., initialisation completion).
- To trigger events with arbitrary data passed to listeners.
- To implement publish-subscribe communication between decoupled components.

### Syntax Rules and Structure

**`emitter.on(eventName, listener)`:**
```js
emitter.on('event', (arg1, arg2) => { /* ... */ });
```
| Component | Breakdown |
|-----------|-----------|
| `eventName` | String or symbol identifying the event. |
| `listener` | Function to call when the event is emitted. |
| Returns | A reference to the `EventEmitter`, so calls can be chained. |

**`emitter.once(eventName, listener)`:**
```js
emitter.once('event', () => { /* ... */ });
```
| Component | Breakdown |
|-----------|-----------|
| `eventName` | String or symbol. |
| `listener` | Function called at most once. |
| Behaviour | Listener is removed after first invocation. |

**`emitter.emit(eventName[, ...args])`:**
```js
emitter.emit('event', arg1, arg2);
```
| Component | Breakdown |
|-----------|-----------|
| `eventName` | String or symbol. |
| `...args` | Arbitrary arguments passed to each listener. |
| Returns | `true` if the event had listeners, `false` otherwise. |

**Constraints and Limitations:**
- `on()` does not check for duplicate listeners; the same function can be registered multiple times and will be called multiple times.
- `once()` listeners are removed before being invoked, so they cannot be removed by `removeListener()` during execution.
- `emit()` is synchronous; if a listener throws, subsequent listeners are not called.

### Multiple Annotated Code Examples

#### Example 1: `on()` vs. `once()`

```js
// on-vs-once.js
const { EventEmitter } = require('node:events');

const emitter = new EventEmitter();

// Persistent listener
emitter.on('tick', () => console.log('on: tick'));

// One-time listener
emitter.once('tick', () => console.log('once: tick'));

emitter.emit('tick'); // Both run
emitter.emit('tick'); // Only 'on' runs
emitter.emit('tick'); // Only 'on' runs
```

**Expected Output:**
```
on: tick
once: tick
on: tick
on: tick
```

**Why this output:** The `on()` listener runs on every emit. The `once()` listener runs only on the first emit and is automatically removed afterward. This demonstrates the fundamental difference between persistent and one-time listeners.

#### Example 2: Passing Arguments to Listeners

```js
// emit-args.js
const { EventEmitter } = require('node:events');

const emitter = new EventEmitter();

emitter.on('sum', (a, b) => {
  console.log(`${a} + ${b} = ${a + b}`);
});

emitter.emit('sum', 3, 7);
emitter.emit('sum', 10, 20);
```

**Expected Output:**
```
3 + 7 = 10
10 + 20 = 30
```

**Why this output:** `emit()` accepts an arbitrary set of arguments after the event name. These are passed directly to each listener function. The listener receives them as its parameters and processes them accordingly.

#### Example 3: Using `this` Inside Listeners

```js
// this-binding.js
const { EventEmitter } = require('node:events');

class MyEmitter extends EventEmitter {}

const myEmitter = new MyEmitter();

myEmitter.on('event', function (a, b) {
  console.log(a, b, this === myEmitter);
});

myEmitter.emit('event', 'a', 'b');
```

**Expected Output:**
```
a b true
```

**Why this output:** When an ordinary function (not an arrow function) is used as a listener, `this` is intentionally set to reference the `EventEmitter` instance. The comparison `this === myEmitter` evaluates to `true`, confirming the binding.

### Real-World Cases

- **Database connections:** Emitting `'connected'` once when the connection is established, and `'data'` on every incoming row.
- **File watchers:** `fs.watch` emits `'change'` on every file modification; `once('close')` handles cleanup.
- **Event-driven APIs:** WebSocket servers emitting `'message'` on every received message.

---

## Core Concept 3: Removing Listeners and Managing Memory Leaks

### Definitions

**Core Definition:** Listeners can be removed individually with `removeListener()` (or its alias `off()`), all at once with `removeAllListeners()`, and the maximum listener count per event can be adjusted with `setMaxListeners()` to suppress or trigger memory leak warnings.

**Technical Definition:** The `emitter.removeListener(eventName, listener)` method removes the specified `listener` from the listener array for the event named `eventName`. If any single listener has been added multiple times to the listener array for the specified `eventName`, then `removeListener()` must be called multiple times to remove each instance. The `emitter.removeAllListeners([eventName])` method removes all listeners, or those of the specified `eventName`. The `emitter.setMaxListeners(n)` method modifies the limit for a specific `EventEmitter` instance, where a value of `Infinity` (or `0`) indicates an unlimited number of listeners.

**Beginner-Friendly Explanation:** Every time you add a listener, it takes up a little bit of memory. If you keep adding listeners without removing them, your program can slowly consume more and more memory — a memory leak. Node.js warns you when you exceed 10 listeners for a single event because that's often a sign of a leak. You can remove listeners individually with `removeListener` or `off`, or clear them all with `removeAllListeners`. If you genuinely need more than 10 listeners, you can raise the limit with `setMaxListeners`.

### Purposes

- To prevent memory leaks by cleaning up listeners that are no longer needed.
- To remove event handlers when a component is destroyed or disconnected.
- To avoid duplicate listener registrations.
- To suppress or investigate the "possible EventEmitter memory leak" warning.
- To ensure that listeners are not called after the object that registered them is no longer in use.

### Syntax Rules and Structure

**`emitter.removeListener(eventName, listener)`:**
```js
emitter.removeListener('event', myListener);
```
| Component | Breakdown |
|-----------|-----------|
| `eventName` | String or symbol. |
| `listener` | The exact function reference used in `on()`. |
| Alias | `off()` is an alias for `removeListener()`. |
| Returns | A reference to the emitter for chaining. |

**`emitter.removeAllListeners([eventName])`:**
```js
emitter.removeAllListeners('event'); // or no argument for all
```
| Component | Breakdown |
|-----------|-----------|
| `eventName` | Optional; if omitted, removes all listeners for all events. |
| Returns | A reference to the emitter. |

**`emitter.setMaxListeners(n)`:**
```js
emitter.setMaxListeners(20);
```
| Component | Breakdown |
|-----------|-----------|
| `n` | Number of listeners before warning; `0` or `Infinity` for unlimited. |
| Returns | A reference to the emitter. |

**Constraints and Limitations:**
- `removeListener()` requires the exact same function reference that was passed to `on()`; anonymous functions cannot be removed.
- `removeAllListeners()` is discouraged when the emitter is shared or created by another module (e.g., sockets, file streams).
- `setMaxListeners()` only changes the warning threshold; it does not prevent memory leaks.
- Once an event is emitted, all listeners attached at the time of emit are called in order; calling `removeListener()` during emit does not affect the currently executing emit.

### Multiple Annotated Code Examples

#### Example 1: Removing Specific Listeners

```js
// remove-listener.js
const { EventEmitter } = require('node:events');

const emitter = new EventEmitter();

// Named function — can be removed later
function onData(data) {
  console.log('Received:', data);
}

// Anonymous function — cannot be removed
emitter.on('data', onData);
emitter.on('data', (d) => console.log('Anonymous:', d));

emitter.emit('data', 'first');
// → 'Received: first' and 'Anonymous: first'

// Remove only the named listener
emitter.removeListener('data', onData);

emitter.emit('data', 'second');
// → only 'Anonymous: second'
```

**Expected Output:**
```
Received: first
Anonymous: first
Anonymous: second
```

**Why this output:** The named function `onData` is removed successfully because it was defined as a named function. The anonymous arrow function remains registered because it has no reference that can be passed to `removeListener()`.

#### Example 2: Managing Max Listeners

```js
// max-listeners.js
const { EventEmitter } = require('node:events');

const emitter = new EventEmitter();

// Raise the limit to 15
emitter.setMaxListeners(15);

// Add 15 listeners — no warning
for (let i = 0; i < 15; i++) {
  emitter.on('event', () => {});
}

console.log('Max listeners:', emitter.getMaxListeners());
console.log('Listener count:', emitter.listenerCount('event'));
// → Max listeners: 15
// → Listener count: 15
```

**Expected Output:**
```
Max listeners: 15
Listener count: 15
```

**Why this output:** By default, adding more than 10 listeners would trigger a `MaxListenersExceededWarning`. Setting `setMaxListeners(15)` raises the threshold, allowing 15 listeners without warning. The `getMaxListeners()` method confirms the new limit.

#### Example 3: Using `once()` as a Memory Leak Prevention Pattern

```js
// once-cleanup.js
const { EventEmitter } = require('node:events');

const emitter = new EventEmitter();

// A listener that removes itself after the first call
emitter.once('ready', () => {
  console.log('Ready — this listener will not run again.');
});

emitter.emit('ready'); // Runs
emitter.emit('ready'); // Does not run
console.log('Listeners remaining:', emitter.listenerCount('ready'));
// → 0
```

**Expected Output:**
```
Ready — this listener will not run again.
Listeners remaining: 0
```

**Why this output:** The `once()` method automatically removes the listener after its first invocation. This is the recommended approach for event handlers that should fire exactly once per object lifecycle, such as cleanup or initialisation handlers.

### Real-World Cases

- **WebSocket cleanup:** Removing `'message'` listeners when a client disconnects to prevent memory leaks.
- **Event listener lifecycle:** Using `once('close')` for cleanup handlers that should run only when a resource is released.
- **Long-running servers:** Periodically auditing `listenerCount()` to detect listener accumulation.
- **Framework development:** Raising `setMaxListeners` for emitters that legitimately have many subscribers (e.g., a global event bus).

---

## Core Concept 4: Error Event Handling Conventions

### Definitions

**Core Definition:** The `'error'` event is a special event type in Node.js — if an `EventEmitter` emits `'error'` without at least one registered listener, the error is thrown, a stack trace is printed, and the Node.js process exits.

**Technical Definition:** When an `EventEmitter` instance experiences an error, the typical action is to emit an `'error'` event. Error events are treated as a special case in Node.js. If there is no listener for it, then the default action is to print a stack trace and exit the program. For all `EventEmitter` objects, if an `'error'` event handler is not provided, the error will be thrown, causing the Node.js process to report an uncaught exception and crash unless either an `'error'` listener is registered or the error is handled by some other mechanism.

**Beginner-Friendly Explanation:** The `'error'` event is like a fire alarm that must always have someone to hear it. If a fire alarm goes off and nobody is listening, the building automatically evacuates (the process crashes). By registering an `'error'` listener, you're saying "I'm here to handle this fire" — and you can decide what to do instead of letting the process crash.

### Purposes

- To provide a standardised channel for communicating errors from an emitter to its consumers.
- To prevent unhandled errors from crashing the entire Node.js process.
- To allow callers to decide how to handle errors (retry, log, fallback, etc.).
- To integrate with the `events.once()` utility for Promise-based error handling.

### Syntax Rules and Structure

**Registering an error listener:**
```js
emitter.on('error', (err) => {
  console.error('An error occurred:', err.message);
});
```
| Component | Breakdown |
|-----------|-----------|
| `'error'` | The special event name. |
| `err` | The `Error` object (or any value) passed to `emit()`. |

**Emitting an error:**
```js
emitter.emit('error', new Error('Something went wrong'));
```
| Component | Breakdown |
|-----------|-----------|
| `'error'` | Event name. |
| `Error` | Should be an `Error` instance for proper stack traces. |

**Using `events.once()` for error handling:**
```js
const { once } = require('node:events');
try {
  await once(emitter, 'ready');
} catch (err) {
  // Rejected if 'error' is emitted while waiting
}
```
| Component | Breakdown |
|-----------|-----------|
| `once(emitter, event)` | Returns a Promise that rejects if `'error'` is emitted. |
| Special handling | Only applied when `once()` waits for a non-`'error'` event. |

**Constraints and Limitations:**
- The `'error'` event is the only event with special handling; other events without listeners are silently ignored.
- If you register an `'error'` listener, you **must** handle the error; otherwise the process may still crash in unexpected ways.
- `events.once()` special error handling only applies when waiting for a non-`'error'` event; if waiting for `'error'` itself, it resolves normally.

### Annotated Code Example

```js
// error-handling.js
const { EventEmitter } = require('node:events');

const emitter = new EventEmitter();

// Register an error listener to prevent crash
emitter.on('error', (err) => {
  console.error('Caught error:', err.message);
});

// Emit an error — handled gracefully
emitter.emit('error', new Error('Database connection failed'));
// → 'Caught error: Database connection failed'

// Emit another error — still handled
emitter.emit('error', new Error('Timeout'));
// → 'Caught error: Timeout'
```

**Expected Output:**
```
Caught error: Database connection failed
Caught error: Timeout
```

**Why this output:** By registering an `'error'` listener, the emitter no longer crashes the process when an error is emitted. The listener receives the `Error` object and can log it, retry, or take other corrective action.

### Real-World Cases

- **Network servers:** Handling socket errors without crashing the server.
- **Database connections:** Emitting `'error'` on connection failure; the application decides whether to retry or exit.
- **Stream processing:** Handling `'error'` events on `fs.ReadStream` and `fs.WriteStream`.
- **Child processes:** Handling `'error'` when a child process fails to spawn.

---

## Core Concept 5: Architecture Patterns — Designing Custom Event-Driven Classes

### Definitions

**Core Definition:** Custom event-driven classes are created by extending the `EventEmitter` class, giving any class the ability to emit and respond to events.

**Technical Definition:** It is common practice to extend the `EventEmitter` class to create specialised event emitters that encapsulate specific business logic. This enables any class to inherit the capabilities of the `EventEmitter` class, therefore becoming an observable object. The most effective approach is extending the `EventEmitter` class to create specialised event emitters that encapsulate specific business logic — this pattern enables loose coupling and makes code more testable and maintainable.

**Beginner-Friendly Explanation:** If you want your own class — say, a `UserManager` or a `FileProcessor` — to be able to broadcast events, you can make it a child of `EventEmitter`. This is like teaching your class to speak the same language as Node.js's built-in modules. Other parts of your program can then "listen" to your class just like they listen to HTTP servers or file streams.

### Purposes

- To create domain-specific classes that emit events relevant to their behaviour.
- To decouple producers of events from consumers, improving modularity and testability.
- To build reusable components that can be plugged into different applications.
- To implement the Observer pattern in a Node.js-idiomatic way.

### Syntax Rules and Structure

**Extending EventEmitter (ES6 class syntax):**
```js
const { EventEmitter } = require('node:events');

class MyClass extends EventEmitter {
  constructor() {
    super(); // Must call before using `this`
    // ...
  }
}
```
| Component | Breakdown |
|-----------|-----------|
| `extends EventEmitter` | Inherits all emitter methods. |
| `super()` | Initialises the EventEmitter parent. |

**Composition alternative:**
```js
class MyClass {
  constructor() {
    this.emitter = new EventEmitter();
  }
  on(event, listener) { return this.emitter.on(event, listener); }
}
```
| Component | Breakdown |
|-----------|-----------|
| `this.emitter` | An internal EventEmitter instance. |
| Delegation | Forward `on`, `emit`, etc. |

**Constraints and Limitations:**
- `super()` must be called before accessing `this` in the constructor.
- Inheriting from `EventEmitter` exposes all emitter methods publicly; some developers prefer composition for encapsulation.
- If `emit()` is called before the parent constructor completes, the emitter may not be fully initialised.

### Multiple Annotated Code Examples

#### Example 1: Extending EventEmitter for a Custom Class

```js
// custom-emitter.js
const { EventEmitter } = require('node:events');

class TaskRunner extends EventEmitter {
  constructor() {
    super();
    this.tasks = [];
  }

  addTask(name) {
    this.tasks.push(name);
    this.emit('taskAdded', name);       // Notify listeners
  }

  runAll() {
    for (const task of this.tasks) {
      this.emit('taskStarted', task);
      // Simulate work
      this.emit('taskCompleted', task);
    }
    this.emit('allCompleted');
  }
}

// Usage
const runner = new TaskRunner();

runner.on('taskAdded', (name) => {
  console.log(`Task added: ${name}`);
});

runner.on('taskStarted', (name) => {
  console.log(`  Started: ${name}`);
});

runner.on('taskCompleted', (name) => {
  console.log(`  Completed: ${name}`);
});

runner.on('allCompleted', () => {
  console.log('All tasks completed.');
});

runner.addTask('Download');
runner.addTask('Process');
runner.runAll();
```

**Expected Output:**
```
Task added: Download
Task added: Process
  Started: Download
  Completed: Download
  Started: Process
  Completed: Process
All tasks completed.
```

**Why this output:** The `TaskRunner` class extends `EventEmitter`, inheriting `on()` and `emit()`. When `addTask()` is called, it emits `'taskAdded'` with the task name. When `runAll()` iterates over the tasks, it emits `'taskStarted'` and `'taskCompleted'` for each task, then `'allCompleted'` at the end. Listeners registered with `on()` respond to each event in order.

#### Example 2: Error Handling in a Custom Emitter

```js
// custom-emitter-error.js
const { EventEmitter } = require('node:events');

class DataFetcher extends EventEmitter {
  fetch(url) {
    // Simulate a failed fetch
    setTimeout(() => {
      if (!url.startsWith('https')) {
        this.emit('error', new Error('URL must use HTTPS'));
      } else {
        this.emit('data', { url, body: '...' });
      }
    }, 10);
  }
}

const fetcher = new DataFetcher();

fetcher.on('data', (result) => {
  console.log('Fetched:', result.url);
});

fetcher.on('error', (err) => {
  console.error('Fetch error:', err.message);
});

fetcher.fetch('http://example.com');  // Triggers error
fetcher.fetch('https://example.com'); // Triggers data
```

**Expected Output:**
```
Fetch error: URL must use HTTPS
Fetched: https://example.com
```

**Why this output:** The `DataFetcher` class emits `'error'` for invalid URLs and `'data'` for valid ones. The error listener handles the invalid case gracefully without crashing the process. Both fetches run asynchronously via `setTimeout`, but the error is handled because a listener is registered.

### Real-World Cases

- **Job queues:** A `JobQueue` class emits `'jobStarted'`, `'jobCompleted'`, and `'jobFailed'`.
- **Chat applications:** A `ChatRoom` class emits `'message'`, `'userJoined'`, and `'userLeft'`.
- **Build tools:** A `Bundler` class emits `'buildStart'`, `'buildEnd'`, and `'error'`.
- **Game engines:** A `GameWorld` class emits `'entitySpawned'`, `'entityDestroyed'`, and `'collision'`.

---

## References

- Node.js Documentation — Events — https://nodejs.org/api/events.html
- Node.js Documentation — `events.once` — https://nodejs.org/api/events.html#eventsonceemitter-name-options
- Node.js Documentation — `events.defaultMaxListeners` — https://nodejs.org/api/events.html#eventsdefaultmaxlisteners
- Node.js Documentation — `emitter.setMaxListeners(n)` — https://nodejs.org/api/events.html#emittersetmaxlistenersn
- Node.js Documentation — `emitter.removeAllListeners([eventName])` — https://nodejs.org/api/events.html#emitterremovealllistenerseventname
- Node.js Documentation — `emitter.removeListener(eventName, listener)` — https://nodejs.org/api/events.html#emitterremovelistenereventname-listener
- Node.js Documentation — `emitter.on(eventName, listener)` — https://nodejs.org/api/events.html#emitteroneventname-listener
- Node.js Documentation — `emitter.once(eventName, listener)` — https://nodejs.org/api/events.html#emitteronceeventname-listener
- Node.js Documentation — `emitter.emit(eventName[, ...args])` — https://nodejs.org/api/events.html#emitteremiteventname-args
- Node.js Learn — The Node.js Event emitter — https://nodejs.org/learn/asynchronous-work/the-nodejs-event-emitter
- Node.js Documentation — `events.errorMonitor` — https://nodejs.org/api/events.html#eventserrormonitor
- Node.js Documentation — `events.EventEmitterAsyncResource` — https://nodejs.org/api/events.html#class-eventemitterasyncresource
- Node.js Documentation — `events.getEventListeners` — https://nodejs.org/api/events.html#eventsgeteventlistenersemitter-eventname
- Node.js Documentation — `events.listenerCount` — https://nodejs.org/api/events.html#eventslistenercountemitter-eventname
- Stack Overflow — Inheriting from EventEmitter — https://stackoverflow.com/questions/17157295/inheriting-from-eventemitter
- CoreUI — How to Remove Event Listeners in Node.js — https://coreui.io/blog/how-to-remove-event-listeners-in-node-js/
- Netguru — Node.js Memory Leaks: Find, Diagnose, and Fix Them — https://www.netguru.com/blog/node-js-memory-leaks