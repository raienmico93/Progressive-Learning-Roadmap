# JavaScript Memory Management — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Memory management is the process by which a program allocates memory for its data, uses that memory during execution, and releases it when the data is no longer needed. In JavaScript, this process is largely automatic through a mechanism called garbage collection.

**Technical Definition:** Per MDN, low-level languages like C have manual memory management primitives such as `malloc()` and `free()`. In contrast, JavaScript automatically allocates memory when objects are created and frees it when they are not used anymore (garbage collection). The memory life cycle is consistent across programming languages: (1) allocate the memory you need, (2) use the allocated memory (read, write), (3) release the allocated memory when it is not needed anymore. The second part is explicit in all languages; the first and last parts are explicit in low-level languages but mostly implicit in high-level languages like JavaScript.

**Beginner-Friendly Explanation:** JavaScript manages memory for you. When you create an object, string, or function, JavaScript sets aside some memory to store it. When you're done using it, JavaScript automatically cleans it up to free space for other things. This is different from languages like C, where you have to manually ask for memory and manually give it back. The good news is you don't have to think about it most of the time. The tricky part is knowing when JavaScript decides something is "no longer needed" — and that's where memory leaks can happen.

### Key Characteristics

- **Automatic:** JavaScript handles allocation and release without explicit developer intervention.
- **Garbage-collected:** A garbage collector monitors memory allocation and reclaims memory when it is no longer reachable.
- **Non-deterministic:** The exact timing of garbage collection is not predictable and varies by engine.
- **Reachability-based:** Modern engines use reachability (not reference counting) to determine what can be collected.
- **Weak references available:** `WeakMap`, `WeakSet`, `WeakRef`, and `FinalizationRegistry` allow developers to hold references that do not prevent collection.
- **Profiling tools available:** Chrome DevTools provides heap snapshots, allocation timelines, and other tools for diagnosing memory issues.

### Prerequisites

- Basic JavaScript syntax (variables, objects, functions).
- Understanding of references and reference semantics.
- Familiarity with scope and closures.
- Awareness of the call stack and execution contexts.
- Basic knowledge of data structures (arrays, objects, maps, sets).

### Related Programming Areas

- **Garbage collection algorithms:** Mark-and-sweep, generational collection.
- **Memory profiling:** Heap snapshots, allocation timelines, memory timelines.
- **Weak collections:** `WeakMap`, `WeakSet`, `WeakRef`, `FinalizationRegistry`.
- **Performance optimisation:** Reducing memory pressure, avoiding leaks.
- **Closures:** Capturing variables and their impact on memory lifetime.
- **DOM programming:** Detached DOM nodes and event listener leaks.

### Core Concepts / Features

The following sections cover each core concept and feature, each using the uniform structure.

---

## Memory Lifecycle

### Definitions

**Core Definition:** The memory lifecycle is the three-phase process that all programming languages follow: allocation, usage, and release.

**Technical Definition:** Per MDN, the memory life cycle is: (1) Allocate the memory you need, (2) Use the allocated memory (read, write), (3) Release the allocated memory when it is not needed anymore. The second part is explicit in all languages. The first and last parts are explicit in low-level languages but mostly implicit in high-level languages like JavaScript.

**Beginner-Friendly Explanation:** Every piece of data in your program goes through three stages: it's created (allocated), it's used (read or written), and it's eventually cleaned up (released). In JavaScript, the first and last stages happen automatically.

### Purposes

- To provide a structured framework for understanding how memory is used in a program.
- To identify where memory management issues (leaks) are most likely to occur.
- To guide developers in writing memory-efficient code.

### Syntax Rules and Structure

The lifecycle is conceptual, not syntactic:

```js
// Phase 1: Allocation
const arr = [1, 2, 3];       // Memory allocated for array
const obj = { name: "Alice" }; // Memory allocated for object
function greet() {}            // Memory allocated for function

// Phase 2: Usage
arr.push(4);                  // Read/write memory
console.log(obj.name);        // Read memory
greet();                      // Use function

// Phase 3: Release
// No explicit action needed; garbage collector handles it
// when arr, obj, and greet are no longer reachable.
```

**Constraints and Limitations:**
- The majority of memory management issues occur at the release phase.
- The most difficult aspect is determining when allocated memory is no longer needed.
- JavaScript's automatic release is an approximation because determining whether a specific block of memory is still needed is undecidable in the general case.

### Real-World Cases

- **Long-running SPAs:** Understanding the lifecycle helps diagnose memory growth over time.
- **Node.js servers:** Memory leaks accumulate over days/weeks if not managed.
- **Mobile browsers:** Memory-constrained devices are more sensitive to leaks.

---

## Allocation

### Definitions

**Core Definition:** Allocation is the process of reserving memory for a value, object, or function when it is created.

**Technical Definition:** Per MDN, JavaScript automatically allocates memory when values are initially declared. This includes numbers, strings, objects, arrays, and functions. Some function calls also result in object allocation (e.g., `new Date()`, `document.createElement()`).

**Beginner-Friendly Explanation:** Every time you create something in JavaScript — a variable, an object, a function — the engine sets aside memory for it. You don't have to ask; it just happens.

### Purposes

- To reserve space for values before they are used.
- To provide storage for objects, arrays, and functions.
- To enable dynamic creation of data structures at runtime.

### Syntax Rules and Structure

```js
// Value initialization — automatic allocation
const n = 123;                    // Number
const s = "string";               // String
const o = { a: 1, b: null };     // Object
const a = [1, null, "str2"];     // Array

// Function allocation
function f(a) { return a + 2; }  // Function (callable object)

// Allocation via function calls
const d = new Date();             // Date object
const e = document.createElement("div"); // DOM element

// Allocation via methods
const s2 = s.substring(0, 3);     // New string (or slice reference)
const a3 = a.concat([4, 5]);      // New array
```

**Syntax Rules:**
- Allocation happens automatically on declaration, creation, or method invocation.
- Strings are immutable; operations like `substring()` may create new strings or reuse memory.
- Function expressions and arrow functions allocate objects.
- Event listeners allocate closures.

**Constraints and Limitations:**
- Allocation itself has a cost; excessive allocation can trigger frequent GC.
- Large objects or many small objects increase memory pressure.
- Some operations (e.g., string concatenation in loops) allocate more than expected.

### Annotated Code Examples

#### Example 1: Common Allocation Patterns

```js
// Step 1: Primitive allocation.
const num = 42;           // Allocates memory for a number
const str = "hello";      // Allocates memory for a string

// Step 2: Object allocation.
const user = { name: "Alice", age: 30 }; // Allocates object + properties

// Step 3: Array allocation.
const items = [1, 2, 3];  // Allocates array + elements

// Step 4: Function allocation.
function greet() { return "Hello"; } // Allocates function object

// Step 5: Function call allocation.
const now = new Date();   // Allocates Date object
```

**Expected Output:** No output; allocation is implicit.

**Why This Output:** Each declaration reserves memory for the value. The engine manages the details.

#### Example 2: Allocation via Methods

```js
const base = "hello world";
const sub = base.substring(0, 5); // Allocates new string (or slice)
console.log(sub); // "hello"

const arr1 = [1, 2];
const arr2 = [3, 4];
const combined = arr1.concat(arr2); // Allocates new array
console.log(combined); // [1, 2, 3, 4]
```

**Expected Output:**
```
hello
[ 1, 2, 3, 4 ]
```

**Why This Output:** `substring()` and `concat()` return new values, which require allocation.

### Real-World Cases

- **Data processing:** Allocating large arrays for batch operations.
- **DOM manipulation:** Creating and appending elements allocates memory.
- **Event handling:** Each listener is a closure (allocated object).

---

## Usage

### Definitions

**Core Definition:** Usage is the phase of the memory lifecycle where allocated memory is read from or written to during program execution.

**Technical Definition:** Per MDN, using values basically means reading and writing in allocated memory. This can be done by reading or writing the value of a variable or an object property or even passing an argument to a function.

**Beginner-Friendly Explanation:** Usage is just "using" your data — reading it, changing it, passing it to functions, storing it in objects.

### Purposes

- To perform computations and transformations on data.
- To store and retrieve information.
- To pass data between functions and modules.
- To enable all program logic.

### Syntax Rules and Structure

```js
// Reading
const x = obj.name;
console.log(arr[0]);

// Writing
obj.name = "Bob";
arr[0] = 99;

// Passing to functions
processData(obj);
```

**Constraints and Limitations:**
- Usage itself doesn't cause leaks; it's the holding of references that matters.
- Closures can capture variables during usage, extending their lifetime.

### Annotated Code Examples

```js
const user = { name: "Alice" };

// Step 1: Read.
console.log(user.name); // "Alice"

// Step 2: Write.
user.name = "Bob";
console.log(user.name); // "Bob"

// Step 3: Pass to function.
function greet(u) { return `Hello, ${u.name}`; }
console.log(greet(user)); // "Hello, Bob"
```

**Expected Output:**
```
Alice
Bob
Hello, Bob
```

**Why This Output:** Reading, writing, and passing are all usage operations on allocated memory.

### Real-World Cases

- **Form handling:** Reading input values, writing to state.
- **API responses:** Reading JSON data, writing to caches.
- **Game loops:** Continuously reading/writing position, velocity, state.

---

## Release

### Definitions

**Core Definition:** Release is the phase where memory that is no longer needed is freed for reuse.

**Technical Definition:** Per MDN, the majority of memory management issues occur at this phase. The most difficult aspect is determining when the allocated memory is no longer needed. Low-level languages require manual determination; JavaScript uses automatic memory management known as garbage collection.

**Beginner-Friendly Explanation:** When you're done with something, JavaScript cleans it up. But "done" is tricky — it means the data is no longer reachable from anywhere in your program.

### Purposes

- To free memory for reuse by other parts of the program.
- To prevent memory exhaustion in long-running applications.
- To maintain performance by keeping memory usage bounded.

### Syntax Rules and Structure

```js
// Release is implicit — you don't call any function.
// You influence it by removing references.

let obj = { data: "large" };
obj = null; // Removes the reference; object becomes eligible for GC

// Or by using block scope.
{
  let temp = { data: "temporary" };
  // temp is released when the block exits (if not captured)
}
```

**Constraints and Limitations:**
- You cannot force garbage collection from JavaScript.
- The exact timing of release is non-deterministic.
- Closures, event listeners, timers, and global variables can prevent release.

### Annotated Code Examples

```js
function createAndRelease() {
  let largeData = new Array(1000000).fill("x");
  // ... use largeData ...
  largeData = null; // Remove the reference
  // largeData is now eligible for GC (when the engine runs)
}
createAndRelease();
```

**Expected Output:** No output; release is implicit.

**Why This Output:** Setting `largeData = null` removes the only reference, making the array eligible for collection.

### Real-World Cases

- **Single-page applications:** Releasing component state on unmount.
- **Node.js streams:** Releasing buffers after processing.
- **Image processing:** Releasing canvas data after export.

---

## Garbage Collection

### Definitions

**Core Definition:** Garbage collection (GC) is the automatic process by which a JavaScript engine reclaims memory occupied by objects that are no longer reachable from the program.

**Technical Definition:** Per MDN, some high-level languages, such as JavaScript, utilize a form of automatic memory management known as garbage collection. The purpose of a garbage collector is to monitor memory allocation and determine when a block of allocated memory is no longer needed and reclaim it. This automatic process is an approximation because the general problem of determining whether a specific piece of memory is still needed is undecidable.

**Beginner-Friendly Explanation:** Garbage collection is JavaScript's automatic memory cleaner. It finds data you're no longer using and frees it. You can't control when it runs, but you can help by removing references to things you don't need.

### Purposes

- To automatically reclaim memory without developer intervention.
- To prevent memory leaks from forgotten allocations.
- To maintain a consistent memory footprint in long-running programs.
- To enable developers to focus on logic rather than memory management.

### Syntax Rules and Structure

Garbage collection is not directly controllable from JavaScript. You influence it by:

```js
// Removing references.
let obj = { data: "x" };
obj = null; // Eligible for GC

// Letting variables go out of scope.
function doWork() {
  let temp = { data: "y" };
  // temp eligible for GC when function returns
}

// Using weak references for non-essential data.
const weak = new WeakMap();
```

**Constraints and Limitations:**
- GC runs at unpredictable times; you cannot force it.
- GC pauses can cause performance hiccups.
- Modern engines use generational collection to minimise pauses.
- Circular references are handled by mark-and-sweep, not reference counting.

### Annotated Code Examples

```js
// Objects are reachable and not collected.
let globalRef = { name: "global" };

function localScope() {
  let localRef = { name: "local" };
  // localRef is reachable during function execution.
}
localScope();
// After return, localRef is no longer reachable → eligible for GC.

// Global remains reachable → never collected (until reference removed).
```

**Expected Output:** No output.

**Why This Output:** Reachability determines collection. `globalRef` persists; `localRef` becomes eligible after the function returns.

### Real-World Cases

- **Long-running SPAs:** GC keeps memory usage stable.
- **Node.js servers:** GC prevents memory exhaustion over time.
- **Mobile browsers:** Efficient GC is critical for battery and performance.

---

## Mark and Sweep

### Definitions

**Core Definition:** Mark-and-sweep is the garbage collection algorithm used by modern JavaScript engines, which determines reachability by marking all reachable objects from roots and sweeping unmarked objects.

**Technical Definition:** Mark-and-sweep works by marking roots, such as temporary variables (in the current method) and global (static) variables, and then sweeping the heap removing any unmarked references. Most JavaScript engines use some variant of a compacting mark-and-sweep garbage collector, not a simple reference counting GC, so reference cycles do not cause problems.

**Beginner-Friendly Explanation:** Mark-and-sweep is like a search party. The GC starts from known places (global variables, the call stack) and "marks" everything it can reach. Anything not marked is considered garbage and is swept away.

### Purposes

- To accurately identify unreachable memory, including circular references.
- To reclaim memory without requiring developer intervention.
- To handle complex object graphs with cycles.

### Syntax Rules and Structure

No explicit syntax; the algorithm runs internally.

**Algorithm steps:**
1. **Mark:** Starting from roots (global object, call stack, etc.), traverse all reachable objects and mark them.
2. **Sweep:** Scan the heap and collect all unmarked objects.
3. **Compact (optional):** Move surviving objects to reduce fragmentation.

### Annotated Code Examples

```js
// Circular reference — handled by mark-and-sweep.
function createCircular() {
  const a = {};
  const b = {};
  a.ref = b;
  b.ref = a;
  // When createCircular() returns, a and b are unreachable
  // even though they reference each other.
  // Mark-and-sweep correctly collects them.
}
createCircular();
```

**Expected Output:** No output.

**Why This Output:** Mark-and-sweep starts from roots and marks reachable objects. Since `a` and `b` are not reachable from any root after the function returns, they are collected despite their circular reference.

### Real-World Cases

- **DOM trees:** Mark-and-sweep handles complex parent-child references.
- **React component trees:** Fibre nodes are reachable during reconciliation.
- **Graph data structures:** Cyclic graphs are correctly collected.

---

## Reference Counting

### Definitions

**Core Definition:** Reference counting is a garbage collection strategy that tracks how many references point to each object, collecting an object when its reference count reaches zero.

**Technical Definition:** Reference counting is not used by modern JavaScript engines because it cannot handle circular references. However, it is used in some other languages (e.g., PHP). The problem that mark-and-sweep aims to solve is catching circular references.

**Beginner-Friendly Explanation:** Reference counting is like keeping a tally of how many people are pointing at an object. When nobody points at it, it's cleaned up. But if two objects point at each other, the tally never reaches zero — even if nobody else is using them.

### Purposes (Conceptual)

- To understand the historical evolution of garbage collection.
- To recognise why circular references don't cause leaks in modern JS.
- To contrast with mark-and-sweep.

### Syntax Rules and Structure

No syntax; reference counting is an engine-internal strategy.

### Annotated Code Examples

```js
// Circular reference would leak with reference counting.
function circularLeak() {
  const a = {};
  const b = {};
  a.ref = b; // a references b
  b.ref = a; // b references a
  // With reference counting: a and b have refcounts > 0
  // even after function returns, so they leak.
  // With mark-and-sweep: they are unreachable → collected.
}
circularLeak();
```

**Expected Output:** No output.

**Why This Output:** Modern engines use mark-and-sweep, which handles this correctly.

### Real-World Cases

- **Legacy browsers:** Older engines (IE6/7) used reference counting, causing leaks.
- **PHP:** Still uses reference counting with cycle detection.
- **Understanding history:** Why modern JS doesn't leak on circular references.

---

## Memory Leaks

### Definitions

**Core Definition:** A memory leak occurs when memory that is no longer needed by the program is not released, causing memory usage to grow over time.

**Technical Definition:** Per Chrome DevTools, a memory leak occurs when a bug in the page causes the page to progressively use more memory over time. Common causes include uncleared event listeners, detached DOM nodes, unintentional global variables, closures, and unbounded data structures.

**Beginner-Friendly Explanation:** A memory leak is when your program forgets to clean up things it no longer needs. Over time, memory usage grows, and the page or app slows down or crashes.

### Common Causes

1. **Uncleared event listeners:** An event listener maintains a reference to its target element, potentially leading to memory issues.
2. **Detached DOM nodes:** Nodes removed from the document tree but still referenced by a JavaScript variable.
3. **Unintentional global variables:** Variables declared without proper scoping that remain in memory forever.
4. **Closures:** Closures that capture too much data and keep it alive.
5. **Unbounded data structures:** Arrays, Maps, or Sets that grow without limit.
6. **Forgotten timers and intervals:** A `setInterval` that is never cleared keeps its callback and everything it closes over alive indefinitely.

### Annotated Code Examples

#### Example 1: Detached DOM Leak

```js
// BAD: Detached DOM node leak.
let detachedNode;

function createLeak() {
  const div = document.createElement("div");
  div.textContent = "I leak!";
  document.body.appendChild(div);
  detachedNode = div; // Global reference keeps node alive
  document.body.removeChild(div); // Removed from DOM, but still referenced
}

createLeak();
// detachedNode still holds the removed div → memory leak.
```

**Expected Output:** No output; leak is silent.

**Why This Output:** The node is removed from the DOM but `detachedNode` still references it, preventing garbage collection.

#### Example 2: Event Listener Leak

```js
// BAD: Listener holds reference to element.
function addLeakyListener() {
  const button = document.createElement("button");
  document.body.appendChild(button);

  const handler = () => console.log("clicked");
  button.addEventListener("click", handler);

  // Later, the button is removed, but the listener still references it.
  document.body.removeChild(button);
  // The handler (and button) remain in memory.
}
addLeakyListener();
```

**Expected Output:** No output.

**Why This Output:** The event listener creates a reference cycle between the button and the handler, preventing collection.

#### Example 3: Global Leak

```js
// BAD: Accidental global.
function leaky() {
  leakedData = new Array(1000000).fill("x"); // No var/let/const
}
leaky();
// leakedData is a global property → never collected.
```

**Expected Output:** No output.

**Why This Output:** The undeclared assignment creates a global variable in non-strict mode.

### Real-World Cases

- **React components:** Unmounted components with active timers or subscriptions.
- **Node.js servers:** Accumulating request data in caches.
- **Browser extensions:** Leaking DOM references across page navigations.

---

## Memory Profiling

### Definitions

**Core Definition:** Memory profiling is the practice of measuring and analysing memory usage to identify leaks, bloat, and inefficiencies.

**Technical Definition:** Chrome DevTools provides several profiling tools: Heap Snapshots (memory distribution among JavaScript objects and DOM nodes), Allocation Instrumentation on Timeline (tracks allocations over time), Allocation Sampling (profiles allocations by function), and Memory Timeline (visualises memory usage over time).

**Beginner-Friendly Explanation:** Memory profiling is like a health check for your app's memory. It shows you how much memory is being used, what's using it, and whether it's growing over time.

### Purposes

- To identify memory leaks.
- To find detached DOM nodes.
- To understand allocation patterns.
- To optimise memory usage.
- To debug memory growth in long-running applications.

### Using Chrome DevTools

1. **Chrome Task Manager:** Press `Shift+Esc` to see real-time memory usage.
2. **Heap Snapshot:** Open DevTools → Memory panel → Heap Snapshot → Take Snapshot.
3. **Allocation Timeline:** Record allocations over time to see when memory is allocated.
4. **Memory Timeline:** Visualise memory usage over time to spot leaks.

### Annotated Code Examples

```js
// Code to profile: intentionally leaking.
const leaks = [];

function createLeak() {
  const data = new Array(10000).fill("leak");
  leaks.push(data); // Unbounded growth
}

// In DevTools:
// 1. Take a heap snapshot.
// 2. Run createLeak() several times.
// 3. Take another snapshot.
// 4. Compare: Array objects growing in count.
```

**Expected Output:** Heap snapshot shows increasing Array instances.

**Why This Output:** The `leaks` array holds references, preventing collection.

### Real-World Cases

- **Production monitoring:** Heap snapshots in staging to catch leaks before release.
- **Performance regression:** Allocation timelines to identify hot allocation sites.
- **Memory bloat:** Comparing snapshots to find retained objects.

---

## Weak References

### Definitions

**Core Definition:** A weak reference is a reference to an object that does not prevent the object from being garbage-collected.

**Technical Definition:** Per MDN, the `WeakRef` and `FinalizationRegistry` objects hold references to garbage-collectable objects without creating strong references that prevent their garbage collection. `WeakMap` and `WeakSet` also hold weak references, allowing entries to be removed when keys/values are no longer needed.

**Beginner-Friendly Explanation:** Normally, if you reference an object, it stays alive. A weak reference says "I'd like to use this object if it's still around, but don't keep it alive just for me."

### Purposes

- To hold references to objects without preventing collection.
- To implement caches that release memory under pressure.
- To observe object lifetimes.
- To attach metadata to objects without extending their lifetime.

### Available Weak Collections

- **`WeakMap`:** Key-value pairs where keys are objects and held weakly.
- **`WeakSet`:** Set of objects held weakly.
- **`WeakRef`:** A single weak reference to an object.
- **`FinalizationRegistry`:** Callbacks when objects are collected.

### Annotated Code Examples

```js
// WeakMap: metadata without extending lifetime.
const metadata = new WeakMap();

function attach(el, data) {
  metadata.set(el, data); // el held weakly
}

const el = document.createElement("div");
attach(el, { clicks: 0 });
// When el is removed from DOM and unreferenced,
// the metadata entry is automatically removed.
```

### Real-World Cases

- **DOM metadata:** Attaching data to elements without leaks.
- **Caching:** Weak caches that release memory under pressure.
- **Private data:** Storing private state keyed by instance.

---

## `WeakMap`

### Definitions

**Core Definition:** `WeakMap` is a collection of key-value pairs where keys must be objects and are held weakly, meaning entries are automatically removed when keys are garbage-collected.

**Technical Definition:** Per ECMA-262, `WeakMap` instances have a `[[WeakMapData]]` internal slot. Keys must be objects or non-registered symbols. Entries are removed when the key is no longer reachable. `WeakMap` is not iterable and has no `size` property.

**Beginner-Friendly Explanation:** `WeakMap` is like a `Map` that doesn't keep its keys alive. If the only thing referencing a key is the `WeakMap` itself, the entry is automatically removed.

### Purposes

- To associate data with objects without preventing their collection.
- To implement private data patterns.
- To attach metadata to DOM nodes.
- To create memory-efficient caches.

### Syntax Rules and Structure

```js
new WeakMap()
new WeakMap(iterable)
```

**Methods:** `set(key, value)`, `get(key)`, `has(key)`, `delete(key)`.

**Constraints:** No `size`, no iteration, no `clear`. Keys must be objects.

### Annotated Code Examples

```js
const wm = new WeakMap();
const key = { id: 1 };

wm.set(key, "value");
console.log(wm.get(key)); // "value"
console.log(wm.has(key)); // true

wm.delete(key);
console.log(wm.has(key)); // false
```

**Expected Output:**
```
value
true
false
```

**Why This Output:** `WeakMap` stores entries by key identity and removes them on `delete`.

### Real-World Cases

- **Private class fields (pre-`#`):** Storing instance-private data.
- **DOM metadata:** `elementData.set(el, { clicks: 0 })`.
- **Caching:** `cache.set(objKey, computedValue)`.

---

## `WeakSet`

### Definitions

**Core Definition:** `WeakSet` is a collection of unique objects held weakly, meaning entries are automatically removed when objects are garbage-collected.

**Technical Definition:** Per ECMA-262, `WeakSet` instances have a `[[WeakSetData]]` internal slot. Values must be objects or non-registered symbols. `WeakSet` is not iterable and has no `size` property.

**Beginner-Friendly Explanation:** `WeakSet` is like a `Set` that doesn't keep its values alive. It's useful for tracking "have I seen this object?" without preventing collection.

### Purposes

- To track object membership without retaining objects.
- To mark objects as "visited" or "processed."
- To deduplicate event listeners.

### Syntax Rules and Structure

```js
new WeakSet()
new WeakSet(iterable)
```

**Methods:** `add(value)`, `has(value)`, `delete(value)`.

**Constraints:** No `size`, no iteration, no `clear`. Values must be objects.

### Annotated Code Examples

```js
const ws = new WeakSet();
const obj = { id: 1 };

ws.add(obj);
console.log(ws.has(obj)); // true

ws.delete(obj);
console.log(ws.has(obj)); // false
```

**Expected Output:**
```
true
false
```

**Why This Output:** `WeakSet` tracks membership by object identity.

### Real-World Cases

- **Visited tracking:** `visited.add(node)` in graph traversal.
- **Listener deduplication:** `attached.add(element)`.
- **Branding:** `trusted.add(obj)` to mark validated objects.

---

## `WeakRef`

### Definitions

**Core Definition:** `WeakRef` creates a weak reference to an object, allowing access to the object without preventing its garbage collection.

**Technical Definition:** Per ES2021, `WeakRef` holds a reference to a target object that does not prevent collection. The `deref()` method returns the target if still alive, or `undefined` if collected.

**Beginner-Friendly Explanation:** `WeakRef` is like a sticky note that says "here's where that object was." If the object is still around, `deref()` gives it to you; if it's been cleaned up, you get `undefined`.

### Purposes

- To hold a reference without preventing collection.
- To implement weak caches.
- To observe object lifetime.

### Syntax Rules and Structure

```js
const ref = new WeakRef(target);
const value = ref.deref(); // target or undefined
```

**Constraints:** Target must be an object. Collection timing is non-deterministic.

### Annotated Code Examples

```js
let obj = { data: "value" };
const ref = new WeakRef(obj);

console.log(ref.deref()); // { data: "value" }

obj = null;
// Later: ref.deref() may return undefined (non-deterministic)
```

**Expected Output:**
```
{ data: 'value' }
```

**Why This Output:** The object is still reachable through `obj` at the time of `deref()`.

### Real-World Cases

- **Weak caches:** Storing computed values with weak references to source objects.
- **Observers:** Tracking objects without extending their lifetime.

---

## `FinalizationRegistry`

### Definitions

**Core Definition:** `FinalizationRegistry` allows you to register a callback that runs after an object is garbage-collected.

**Technical Definition:** Per ES2021, `FinalizationRegistry` manages cleanup callbacks. `register(target, heldValue, unregisterToken)` registers a target; `unregister(token)` removes registrations. Callbacks run as tasks after collection.

**Beginner-Friendly Explanation:** `FinalizationRegistry` is like a "when this object is cleaned up, do this" service. You register an object and a callback, and the callback runs after the object is collected.

### Purposes

- To run cleanup logic after object collection.
- To release external resources.
- To implement advanced cache eviction.

### Syntax Rules and Structure

```js
const registry = new FinalizationRegistry((heldValue) => {
  // cleanup logic
});
registry.register(target, heldValue, unregisterToken);
registry.unregister(unregisterToken);
```

**Constraints:** Timing is non-deterministic. Callbacks run as tasks. Errors are uncaught.

### Annotated Code Examples

```js
const registry = new FinalizationRegistry((id) => {
  console.log("Cleaned:", id);
});

let obj = { data: "x" };
registry.register(obj, "object-1");
obj = null;
// Eventually: "Cleaned: object-1"
```

**Expected Output (eventually):**
```
Cleaned: object-1
```

**Why This Output:** When `obj` is collected, the callback runs with the held value.

### Real-World Cases

- **Resource cleanup:** Releasing WebGL contexts, file handles.
- **Cache invalidation:** Removing secondary cache entries.
- **Metrics:** Counting object lifetimes.

---

## References

- MDN Web Docs – Memory management — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Memory_management
- Chrome DevTools – Fix memory problems — https://developer.chrome.com/docs/devtools/memory-problems
- Chrome DevTools – Memory panel overview — https://developer.chrome.com/docs/devtools/memory
- MDN Web Docs – WeakMap — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakMap
- MDN Web Docs – WeakSet — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakSet
- MDN Web Docs – WeakRef — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakRef
- MDN Web Docs – FinalizationRegistry — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/FinalizationRegistry
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Managing Memory — https://tc39.es/ecma262/multipage/managing-memory.html
- TC39 Proposal – Symbol as WeakMap Keys — https://tc39.es/proposal-symbols-as-weakmap-keys/
- AppSignal Blog – JavaScript Internals: Garbage Collection — https://blog.appsignal.com/2020/10/21/javascript-internals-garbage-collection.html
- V8 Blog – High-performance garbage collection for V8 — https://v8.dev/blog/high-performance-gc
- V8 Blog – WeakRefs and FinalizationRegistry — https://v8.dev/features/weak-references
- Web Directions – Strength in weakness: JavaScript memory management and weak references — https://webdirections.org/blog/strength-in-weakness-javascript-memory-management-and-weak-references/