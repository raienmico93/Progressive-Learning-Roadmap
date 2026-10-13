# JavaScript `WeakRef` and `FinalizationRegistry` — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** `WeakRef` and `FinalizationRegistry` are ES2021 features that provide controlled, observable interaction with JavaScript's garbage collector. `WeakRef` holds a weak reference to an object (allowing collection while still permitting access), and `FinalizationRegistry` runs cleanup callbacks after objects are garbage-collected.

**Technical Definition:** Per ECMAScript 2025 (ECMA-262, §26.1 and §26.2), `WeakRef` is a constructor whose instances have a `[[WeakRefTarget]]` internal slot holding a weak reference to an object or non-registered symbol. The `deref()` method returns the target if it is still reachable, else `undefined`. `FinalizationRegistry` is a constructor whose instances have a `[[Cells]]` list of records containing a target, a held value, and an unregister token. When a target is collected, the registry's cleanup callback is scheduled with the held value. Both features are designed for memory-sensitive caches and resource cleanup, and both are explicitly noted in the specification as having *non-deterministic* timing.

**Beginner-Friendly Explanation:** Normally, if you want to keep track of an object, you must hold a reference to it — which prevents it from being cleaned up. `WeakRef` lets you hold a reference that *doesn't* prevent cleanup, so you can still access the object if it's alive, but you won't keep it alive yourself. `FinalizationRegistry` lets you run a callback *after* an object has been cleaned up, so you can free related resources. Together, they enable advanced patterns like automatic caches and resource managers.

### Key Characteristics

- **Non-deterministic:** The garbage collector decides *if* and *when* to collect objects; cleanup callbacks may never run.
- **Not for critical logic:** Never rely on `WeakRef`/`FinalizationRegistry` for program correctness.
- **Microtask/task scheduling:** Cleanup callbacks run as tasks after collection, not synchronously.
- **Errors isolated:** If a cleanup callback throws, the error is reported as an uncaught exception but does not stop other callbacks.
- **No ordering guarantees:** Callbacks may run in any order; multiple callbacks for the same registry are not ordered.
- **Spec-noted caution:** The ECMAScript specification explicitly warns against relying on these features for resource management in most cases.

### Prerequisites

- Solid understanding of `WeakMap` and `WeakSet`.
- Familiarity with JavaScript garbage collection concepts.
- Understanding of references, reachability, and object identity.
- Knowledge of event loop and microtask/macrotask scheduling.

### Related Programming Areas

- **Memory management:** Caches, resource pools, leak prevention.
- **Cache eviction:** LRU-style caches that release memory under pressure.
- **Resource cleanup:** Releasing external resources (WebGL contexts, file handles).
- **Observability:** Tracking object lifetimes for debugging.
- **Advanced patterns:** Dependency injection, weak event listeners.

### Core Concepts / Features

The following sections cover each core concept and feature, each using the uniform structure: Definitions, Purposes, Syntax Rules and Structure, Annotated Code Examples with Expected Outputs, and Real-World Cases.

---

## `WeakRef`

### Definitions

**Core Definition:** `WeakRef` is a constructor that creates a weak reference to an object, allowing access to the object without preventing its garbage collection.

**Technical Definition:** `new WeakRef(target)` creates a WeakRef instance whose `[[WeakRefTarget]]` internal slot holds a weak reference to `target` (which must be an object or non-registered symbol). The WeakRef instance itself is strongly held; the target is weakly held. The `deref()` method returns the target if still reachable, or `undefined` if it has been collected.

**Beginner-Friendly Explanation:** A `WeakRef` is like a sticky note that says "here's where that object *was*." If the object is still around, `deref()` gives it to you. If the garbage collector has cleaned it up, `deref()` returns `undefined`.

### Purposes

- To hold a reference to an object without preventing its collection.
- To implement caches that release memory under pressure.
- To observe whether an object is still alive.
- To build weak event listeners or dependency tracking.

### Syntax Rules and Structure

**General Syntax:**
```js
new WeakRef(target)
```

**Component Breakdown:**
- `target`: An object or non-registered symbol. Primitives throw `TypeError`.
- Returns: A new WeakRef instance.

**Constraints and Limitations:**
- `target` must be an object or non-registered symbol.
- The WeakRef instance itself is a strong reference; only its `[[WeakRefTarget]]` is weak.
- Collection timing is non-deterministic.
- Not serializable; not transferable.

### Annotated Code Examples

#### Example 1: Basic WeakRef Usage

```js
// Step 1: Create an object and a WeakRef to it.
let obj = { id: 1, name: "target" };
const ref = new WeakRef(obj);

// Step 2: Dereference while the object is alive.
console.log(ref.deref()); // { id: 1, name: "target" }

// Step 3: Drop the strong reference.
obj = null;

// Step 4: At some later point, deref() may return undefined.
// (Timing is non-deterministic; may still return the object.)
console.log(ref.deref()); // { id: 1, name: "target" } or undefined
```

**Expected Output (non-deterministic):**
```
{ id: 1, name: 'target' }
{ id: 1, name: 'target' }  // or undefined, depending on GC timing
```

**Why This Output:** Immediately after setting `obj = null`, the object may still be reachable through the WeakRef's internal target until the GC runs. The first `deref()` always returns the object because it was alive at that moment. The second `deref()` is non-deterministic.

#### Example 2: Primitive Targets Throw

```js
try {
  const ref = new WeakRef(42);
} catch (err) {
  console.log(err.name);    // "TypeError"
  console.log(err.message); // "Invalid value used in weak ref"
}
```

**Expected Output:**
```
TypeError
Invalid value used in weak ref
```

**Why This Output:** WeakRef targets must be objects or non-registered symbols. The number `42` is a primitive and is rejected.

#### Example 3: Non-Registered Symbol Targets

```js
const sym = Symbol("unique");
const ref = new WeakRef(sym);
console.log(ref.deref() === sym); // true

// Registered symbols are invalid.
try {
  new WeakRef(Symbol.for("app.key"));
} catch (err) {
  console.log(err.name); // "TypeError"
}
```

**Expected Output:**
```
true
TypeError
```

**Why This Output:** Non-registered symbols can be held weakly; registered symbols are part of the global registry and are never collected, so they cannot be weak targets.

### Real-World Cases

- **Caches:** Storing computed values with weak references to source objects.
- **Observers:** Tracking objects without extending their lifetime.
- **Debugging:** Monitoring whether objects are collected.

---

## `deref()`

### Definitions

**Core Definition:** `deref()` returns the target object of a WeakRef if it is still reachable, or `undefined` if it has been garbage-collected.

**Technical Definition:** `WeakRef.prototype.deref()` returns `this.[[WeakRefTarget]]` if it is still in the set of reachable objects, else `undefined`. The specification permits the implementation to keep the target alive for a short period after the last strong reference is dropped to avoid thrashing.

**Beginner-Friendly Explanation:** `deref()` is how you "look up" the object behind a WeakRef. If it's still alive, you get it; if not, you get `undefined`.

### Purposes

- To retrieve the target of a WeakRef.
- To check whether the target has been collected.
- To implement cache lookups that gracefully handle collection.

### Syntax Rules and Structure

```js
ref.deref()
```

**Returns:** The target object, or `undefined`.

**Constraints and Limitations:**
- Cannot be called on non-WeakRef receivers.
- Returns `undefined` if collected.
- Even if `deref()` returns an object, a subsequent call may return `undefined` (collection may occur between calls).
- The returned object is a strong reference for the duration of its use.

### Annotated Code Examples

#### Example 1: Safe Dereferencing

```js
const obj = { value: 42 };
const ref = new WeakRef(obj);

// Step 1: Dereference and check.
const target = ref.deref();
if (target !== undefined) {
  console.log(target.value); // 42
} else {
  console.log("Object has been collected");
}
```

**Expected Output:**
```
42
```

**Why This Output:** The object is still strongly referenced by `obj`, so `deref()` returns it. The nullish check prevents errors when the object has been collected.

#### Example 2: Cache Pattern with Deref

```js
const cache = new Map();

function getOrCompute(key) {
  // Step 1: Check the cache.
  if (cache.has(key)) {
    const ref = cache.get(key);
    const value = ref.deref();
    if (value !== undefined) {
      console.log("Cache hit");
      return value;
    }
    // Step 2: Cached value was collected; remove stale entry.
    cache.delete(key);
  }
  // Step 3: Compute and cache.
  const value = { data: `computed for ${key}` };
  cache.set(key, new WeakRef(value));
  console.log("Cache miss");
  return value;
}

const objKey = {};
console.log(getOrCompute(objKey).data); // Cache miss, computed
console.log(getOrCompute(objKey).data); // Cache hit (if alive)
```

**Expected Output:**
```
Cache miss
computed for [object Object]
Cache hit
computed for [object Object]
```

**Why This Output:** The first call computes and caches a WeakRef to the value. The second call finds the ref alive and returns the cached value.

### Real-World Cases

- **Cache lookups:** Checking if a weak reference is still valid.
- **Observer patterns:** Accessing subjects without retaining them.
- **Debugging:** Probing whether an object is still alive.

---

## `FinalizationRegistry`

### Definitions

**Core Definition:** `FinalizationRegistry` is a constructor that lets you register cleanup callbacks to run after objects are garbage-collected.

**Technical Definition:** `new FinalizationRegistry(cleanupCallback)` creates a registry. The `register(target, heldValue, unregisterToken)` method adds a record to `[[Cells]]` with a weak reference to `target`, a strongly held `heldValue`, and an optional `unregisterToken`. When `target` is collected, the cleanup callback is scheduled with `heldValue`. The `unregister(unregisterToken)` method removes registered records.

**Beginner-Friendly Explanation:** A `FinalizationRegistry` is like a "when this object is cleaned up, do this" service. You tell it what object to watch, what value to pass to your callback, and optionally a token so you can cancel the registration later.

### Purposes

- To run cleanup logic after objects are collected.
- To release external resources associated with objects.
- To implement advanced cache eviction.
- To observe object lifetimes for debugging or metrics.

### Syntax Rules and Structure

**General Syntax:**
```js
new FinalizationRegistry(cleanupCallback)
registry.register(target, heldValue)
registry.register(target, heldValue, unregisterToken)
registry.unregister(unregisterToken)
```

**Component Breakdown:**
- `cleanupCallback`: A function called with `heldValue` after `target` is collected.
- `target`: The object to watch (must be an object or non-registered symbol).
- `heldValue`: Any value passed to the callback (must not be `target` itself).
- `unregisterToken`: Any value used to unregister the record later.

**Syntax Rules:**
- `cleanupCallback` must be callable; otherwise `TypeError`.
- `target` must be a valid weak target.
- `heldValue` must not be the same as `target` (throws `TypeError`).
- Multiple registrations for the same target are allowed.
- Same `unregisterToken` can unregister multiple records.

**Constraints and Limitations:**
- Cleanup callback timing is non-deterministic.
- Callbacks run as tasks; they never interrupt synchronous code.
- Errors in callbacks are reported as uncaught exceptions.
- Registries do not keep targets alive.
- The specification explicitly warns against relying on these for correctness.

### Annotated Code Examples

#### Example 1: Basic Finalization

```js
// Step 1: Create a FinalizationRegistry with a cleanup callback.
const registry = new FinalizationRegistry((heldValue) => {
  console.log(`Cleaned up: ${heldValue}`);
});

// Step 2: Create a target and register it.
let obj = { id: 1 };
registry.register(obj, "object-1");

// Step 3: Drop the strong reference.
obj = null;

// Step 4: At some later point, the callback runs:
// "Cleaned up: object-1"
// (Timing is non-deterministic.)
```

**Expected Output (eventually, non-deterministic):**
```
Cleaned up: object-1
```

**Why This Output:** When `obj` is collected, the registry schedules the callback with the held value `"object-1"`.

#### Example 2: Held Value Must Differ from Target

```js
const registry = new FinalizationRegistry(() => {});

const obj = {};
try {
  registry.register(obj, obj); // heldValue === target
} catch (err) {
  console.log(err.name);    // "TypeError"
  console.log(err.message); // "Cannot register the same object as the target and held value"
}
```

**Expected Output:**
```
TypeError
Cannot register the same object as the target and held value
```

**Why This Output:** Allowing the held value to be the target would create a self-reference that prevents collection, defeating the purpose of the registry.

### Real-World Cases

- **Resource cleanup:** Releasing WebGL contexts, file handles, or network sockets.
- **Cache invalidation:** Removing entries from secondary caches.
- **Metrics:** Counting object lifetimes.

---

## `register()`

### Definitions

**Core Definition:** `register()` associates a target object with a held value and an optional unregister token in a FinalizationRegistry.

**Technical Definition:** `FinalizationRegistry.prototype.register(target, heldValue, unregisterToken)` adds a record to `[[Cells]]`. The target is weakly held; the held value is strongly held; the unregister token is weakly held (if provided). Returns `undefined`.

**Beginner-Friendly Explanation:** `register()` is how you tell the registry "watch this object, and when it's cleaned up, call my callback with this value."

### Purposes

- To associate cleanup callbacks with objects.
- To pass context to cleanup callbacks via held values.
- To enable later unregistration via tokens.

### Syntax Rules and Structure

```js
registry.register(target, heldValue)
registry.register(target, heldValue, unregisterToken)
```

**Component Breakdown:**
- `target`: Object or non-registered symbol to watch.
- `heldValue`: Any value (must not be `target`) passed to the callback.
- `unregisterToken` (optional): Any value used to unregister.

**Constraints and Limitations:**
- `heldValue` cannot be `target`.
- `target` must be a valid weak target.
- Multiple registrations are allowed.
- No limit on the number of registrations (but memory is finite).

### Annotated Code Examples

#### Example 1: Registering with Held Value and Token

```js
const registry = new FinalizationRegistry((heldValue) => {
  console.log("Cleanup:", heldValue);
});

const obj = { id: 1 };
const token = { name: "token-1" };

// Step 1: Register with a held value and unregister token.
registry.register(obj, "held-data", token);
console.log("Registered.");
```

**Expected Output:**
```
Registered.
```

**Why This Output:** The registration succeeds; the callback will run later (non-deterministically).

#### Example 2: Multiple Registrations

```js
const registry = new FinalizationRegistry((held) => {
  console.log("Cleaned:", held);
});

const obj = { id: 1 };
registry.register(obj, "first");
registry.register(obj, "second");

// When obj is collected, BOTH callbacks are scheduled (order not guaranteed).
```

**Expected Output (eventually):**
```
Cleaned: first
Cleaned: second
```
(Order may vary.)

**Why This Output:** Each `register` call adds a separate record. Both callbacks fire when the target is collected.

### Real-World Cases

- **Resource pools:** Registering each resource for cleanup.
- **Cache tiers:** Registering the same object in multiple registries.
- **Debugging:** Registering objects with diagnostic labels.

---

## `unregister()`

### Definitions

**Core Definition:** `unregister()` removes registrations associated with a given unregister token.

**Technical Definition:** `FinalizationRegistry.prototype.unregister(unregisterToken)` removes all records in `[[Cells]]` whose unregister token is SameValue-equal to `unregisterToken`. Returns `true` if at least one record was removed, else `false`.

**Beginner-Friendly Explanation:** `unregister()` is how you cancel a registration. You pass the token you provided when registering, and the registry forgets about that target.

### Purposes

- To cancel cleanup callbacks when they are no longer needed.
- To prevent stale callbacks from running.
- To manage dynamic registrations.

### Syntax Rules and Structure

```js
registry.unregister(unregisterToken)
```

**Returns:** `true` if any records were removed, else `false`.

**Constraints and Limitations:**
- Unregister tokens are compared by SameValue (like `===` but with `NaN` matching `NaN`).
- Does not affect records with different tokens.
- Does not affect the target object itself.

### Annotated Code Examples

#### Example 1: Unregistering a Cleanup

```js
const registry = new FinalizationRegistry((held) => {
  console.log("Cleanup:", held);
});

const obj = { id: 1 };
const token = { name: "token-1" };

registry.register(obj, "data", token);
console.log(registry.unregister(token)); // true
console.log(registry.unregister(token)); // false (already removed)

// Now, when obj is collected, no callback runs.
```

**Expected Output:**
```
true
false
```

**Why This Output:** The first `unregister` removes the record and returns `true`; the second finds nothing and returns `false`.

#### Example 2: Unregistering Multiple Records with One Token

```js
const registry = new FinalizationRegistry(() => {});
const token = { name: "shared" };

const obj1 = {};
const obj2 = {};
registry.register(obj1, "one", token);
registry.register(obj2, "two", token);

console.log(registry.unregister(token)); // true (removes both)
```

**Expected Output:**
```
true
```

**Why This Output:** Both records share the same token, so both are removed in one call.

### Real-World Cases

- **Cache invalidation:** Removing cleanup when a cache entry is manually evicted.
- **Dynamic resources:** Cancelling cleanup when a resource is reused.
- **Testing:** Cleaning up registrations between tests.

---

## Cleanup Callbacks

### Definitions

**Core Definition:** Cleanup callbacks are the functions passed to a `FinalizationRegistry` constructor, invoked after registered targets are garbage-collected.

**Technical Definition:** The callback is invoked with the held value as its only argument. Invocation is scheduled as a task; the callback never runs synchronously during collection. Errors thrown in the callback are reported as uncaught exceptions but do not affect other callbacks or the registry.

**Beginner-Friendly Explanation:** A cleanup callback is the function that says "here's what to do when this object is gone." It receives the held value you provided during registration.

### Purposes

- To release external resources.
- To clean up related data structures.
- To log or report object collection.
- To trigger follow-up computations.

### Syntax Rules and Structure

```js
const registry = new FinalizationRegistry((heldValue) => {
  // cleanup logic
});
```

**Constraints and Limitations:**
- Timing is non-deterministic.
- Callbacks run as tasks, not microtasks.
- Errors are reported as uncaught exceptions.
- Callbacks may never run (e.g., if the program exits first).
- Callbacks should be fast and non-blocking.

### Annotated Code Examples

#### Example 1: Cleanup with Error Handling

```js
const registry = new FinalizationRegistry((held) => {
  try {
    // Step 1: Attempt cleanup.
    console.log("Cleaning:", held);
    if (held.shouldFail) throw new Error("Cleanup failed");
  } catch (err) {
    // Step 2: Handle errors locally to avoid uncaught exceptions.
    console.error("Cleanup error:", err.message);
  }
});

let obj = { name: "target" };
registry.register(obj, { id: 1, shouldFail: false });
obj = null;
// Eventually: "Cleaning: { id: 1, shouldFail: false }"
```

**Expected Output (eventually):**
```
Cleaning: { id: 1, shouldFail: false }
```

**Why This Output:** The cleanup callback runs after collection. Wrapping in try/catch prevents uncaught exceptions from propagating.

#### Example 2: Cleanup with External Resource Release

```js
const resourceRegistry = new FinalizationRegistry((resourceId) => {
  console.log(`Releasing external resource: ${resourceId}`);
  // In a real application: close file, release WebGL context, etc.
});

class ManagedResource {
  constructor(id) {
    this.id = id;
    resourceRegistry.register(this, id);
  }
}

let res = new ManagedResource("res-1");
res = null;
// Eventually: "Releasing external resource: res-1"
```

**Expected Output (eventually):**
```
Releasing external resource: res-1
```

**Why This Output:** When the `ManagedResource` instance is collected, the callback runs with the held `id`, allowing external cleanup.

### Real-World Cases

- **WebGL contexts:** Releasing GPU resources when objects are collected.
- **File handles (Node.js):** Closing files when wrapper objects are collected.
- **Cache eviction:** Removing stale entries from secondary structures.

---

## Use Cases

### Definitions

**Core Definition:** Use cases for `WeakRef` and `FinalizationRegistry` are scenarios where weak references or post-collection callbacks provide value.

**Technical Definition:** These features are designed for memory-sensitive caches, resource management, and observability. They are not appropriate for critical correctness logic because of non-deterministic timing.

**Beginner-Friendly Explanation:** Use these tools when you want to react to objects being cleaned up, or when you want to reference objects without preventing cleanup.

### Common Use Cases

1. **Cache with Automatic Eviction**
   ```js
   const cache = new Map();
   function get(key, compute) {
     const ref = cache.get(key);
     const cached = ref?.deref();
     if (cached !== undefined) return cached;
     const value = compute();
     cache.set(key, new WeakRef(value));
     return value;
   }
   ```

2. **External Resource Cleanup**
   ```js
   const registry = new FinalizationRegistry((id) => releaseResource(id));
   class Handle {
     constructor(id) {
       this.id = id;
       registry.register(this, id);
     }
   }
   ```

3. **Observer Without Retention**
   ```js
   const observers = new Set();
   function observe(obj) {
     const ref = new WeakRef(obj);
     observers.add(ref);
     return () => observers.delete(ref);
   }
   ```

4. **Deduplication with Weak Lifecycle**
   ```js
   const seen = new WeakSet();
   function processOnce(obj) {
     if (seen.has(obj)) return;
     seen.add(obj);
     // process...
   }
   ```

5. **Debugging Object Lifetimes**
   ```js
   const lifetimeRegistry = new FinalizationRegistry((label) => {
     console.log(`Collected: ${label}`);
   });
   ```

6. **Weak Event Listeners**
   ```js
   function addWeakListener(target, event, handler) {
     const ref = new WeakRef(handler);
     const wrapper = (e) => {
       const fn = ref.deref();
       if (fn) fn.call(target, e);
       else target.removeEventListener(event, wrapper);
     };
     target.addEventListener(event, wrapper);
   }
   ```

### Annotated Code Examples

#### Example 1: Weak-Reference Cache

```js
// Step 1: Define a cache keyed by string IDs.
const cache = new Map();

function getCached(key, compute) {
  // Step 2: Check for a live cached value.
  const ref = cache.get(key);
  const cached = ref?.deref();
  if (cached !== undefined) {
    console.log("Cache hit:", key);
    return cached;
  }
  // Step 3: Compute, cache, and return.
  const value = compute();
  cache.set(key, new WeakRef(value));
  console.log("Cache miss:", key);
  return value;
}

const obj = {};
console.log(getCached("a", () => ({ data: "value-a" })).data);
console.log(getCached("a", () => ({ data: "value-a" })).data);
```

**Expected Output:**
```
Cache miss: a
value-a
Cache hit: a
value-a
```

**Why This Output:** The first call computes and caches a WeakRef. The second finds the ref alive and returns the cached value.

#### Example 2: Resource Cleanup with FinalizationRegistry

```js
// Step 1: Simulate an external resource registry.
const resources = new Map();

function acquireResource(id) {
  const resource = { id, handle: `handle-${id}` };
  resources.set(id, resource);
  return resource;
}

function releaseResource(id) {
  resources.delete(id);
  console.log("Released:", id);
}

// Step 2: Create a FinalizationRegistry to release resources.
const registry = new FinalizationRegistry((id) => releaseResource(id));

// Step 3: Wrap resources in managed objects.
class ManagedResource {
  constructor(id) {
    this.inner = acquireResource(id);
    registry.register(this, id);
  }
}

let res = new ManagedResource("db-1");
console.log("Active resources:", resources.size); // 1
res = null;
// Eventually: "Released: db-1"
```

**Expected Output (eventually):**
```
Active resources: 1
Released: db-1
```

**Why This Output:** When the `ManagedResource` instance is collected, the callback releases the external resource.

### Real-World Cases

- **Browser caches:** Weak caches for computed styles or parsed data.
- **Node.js servers:** Weak references for per-request state.
- **Game engines:** Releasing GPU resources when scene objects are collected.
- **Observability:** Tracking object lifetimes in production.

---

## Cautions

### Definitions

**Core Definition:** Cautions are the warnings and limitations that developers must understand before using `WeakRef` and `FinalizationRegistry`.

**Technical Definition:** The ECMAScript specification (§26.1 and §26.2) explicitly notes that these features are designed for specific memory-sensitive use cases and that their behaviour is non-deterministic. The specification warns that relying on them for resource management is generally discouraged in favour of explicit lifetime management (e.g., `try...finally`, `DisposableStack`).

**Beginner-Friendly Explanation:** These tools are powerful but tricky. You can never be sure *when* (or *if*) cleanup will happen, so you shouldn't use them for anything critical.

### Key Cautions

1. **Non-deterministic timing:** You cannot know when the GC runs or when cleanup callbacks fire. Do not rely on them for correctness.

2. **Callbacks may never run:** If the program exits or the registry is itself collected before the target, callbacks may not fire at all.

3. **Errors are uncaught:** Errors thrown in cleanup callbacks become uncaught exceptions. Wrap cleanup logic in `try...catch`.

4. **Not a substitute for `try...finally`:** For deterministic cleanup, use `try...finally`, `using` declarations (ES2023 `Explicit Resource Management`), or `DisposableStack`.

5. **Reachability subtleties:** Objects held in closures, event listeners, or timers may remain reachable longer than expected. WeakRef does not change reachability semantics — it only adds a weak reference.

6. **Cross-realm issues:** WeakRefs and registries are per-realm; sharing them across realms (iframes, vm contexts) can be problematic.

7. **Performance overhead:** Creating many WeakRefs or registering many objects has overhead. Use sparingly.

8. **Debugging difficulty:** Because timing is non-deterministic, bugs related to these features are hard to reproduce.

9. **Engine-specific behaviour:** Some engines may retain objects longer (e.g., for optimization). Never assume a specific collection time.

10. **Spec guidance:** The specification explicitly recommends against using `FinalizationRegistry` for resource cleanup in most cases, favouring `DisposableStack` and `using`.

### Annotated Code Examples

#### Example 1: Anti-Pattern — Relying on Cleanup for Correctness

```js
// BAD: Assuming the callback will run and clean up before program exit.
const registry = new FinalizationRegistry((id) => {
  console.log("Cleaned:", id);
});

function process() {
  let obj = { id: 1 };
  registry.register(obj, obj.id);
  // Do NOT assume cleanup will run before process() returns.
  // The object is still reachable until the function exits,
  // and GC timing is not guaranteed.
}
process();
console.log("After process");

// Output order is non-deterministic:
// "After process" might appear before or after "Cleaned: 1".
```

**Expected Output (non-deterministic):**
```
After process
Cleaned: 1
```
or
```
Cleaned: 1
After process
```

**Why This Output:** Cleanup timing is non-deterministic. Never rely on the order of cleanup relative to your code.

#### Example 2: Best Practice — Deterministic Cleanup with `try...finally`

```js
// GOOD: Use try...finally for deterministic cleanup.
function process() {
  const resource = acquireResource();
  try {
    // Use resource...
  } finally {
    releaseResource(resource); // Always runs.
  }
}
```

**Expected Output:** No output; cleanup always runs synchronously.

**Why This Output:** `try...finally` guarantees cleanup regardless of errors or early returns. This is the correct pattern for most resource management.

#### Example 3: Using `DisposableStack` (ES2023) for Structured Cleanup

```js
// GOOD: Use DisposableStack for structured resource management.
function process() {
  using stack = new DisposableStack();
  const resource = stack.use(acquireResource());
  // Use resource...
  // Cleanup happens automatically at the end of the block.
}
```

**Note:** `using` and `DisposableStack` are ES2023 features; check environment support.

### Real-World Cases (Cautions)

- **Production code:** Prefer deterministic cleanup; use WeakRef/FinalizationRegistry only for optional optimizations.
- **Testing:** Avoid asserting on cleanup timing; test the observable effects instead.
- **Debugging:** Use `FinalizationRegistry` for diagnostics, not for critical logic.

---

## References

- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – WeakRef Objects — https://tc39.es/ecma262/multipage/managing-memory.html#sec-weak-ref-objects
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – FinalizationRegistry Objects — https://tc39.es/ecma262/multipage/managing-memory.html#sec-finalization-registry-objects
- MDN Web Docs – WeakRef — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakRef
- MDN Web Docs – WeakRef.prototype.deref() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakRef/deref
- MDN Web Docs – FinalizationRegistry — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/FinalizationRegistry
- MDN Web Docs – FinalizationRegistry.prototype.register() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/FinalizationRegistry/register
- MDN Web Docs – FinalizationRegistry.prototype.unregister() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/FinalizationRegistry/unregister
- MDN Web Docs – Memory management — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Memory_management
- TC39 Proposal – WeakRefs — https://github.com/tc39/proposal-weakrefs
- TC39 Proposal – Explicit Resource Management — https://github.com/tc39/proposal-explicit-resource-management
- V8 Blog – WeakRefs and FinalizationRegistry — https://v8.dev/features/weak-references
- V8 Blog – High-performance garbage collection for V8 — https://v8.dev/blog/high-performance-gc
- WebKit Blog – Weak References and Finalization — https://webkit.org/blog/10308/
- WHATWG – Memory Management — https://html.spec.whatwg.org/multipage/webappapis.html