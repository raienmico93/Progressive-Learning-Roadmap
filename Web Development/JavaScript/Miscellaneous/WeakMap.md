# JavaScript `WeakMap` — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A `WeakMap` is a built-in JavaScript collection that stores key-value pairs where keys must be objects (or non-registered symbols), and the entries are held *weakly* — meaning they do not prevent garbage collection of the key object.

**Technical Definition:** Per ECMAScript 2025 (ECMA-262, §24.3), `WeakMap` is a constructor whose instances are exotic objects with a `[[WeakMapData]]` internal slot containing a list of key-value records. Keys must be objects or non-registered symbols; primitive keys (strings, numbers, booleans, `null`, `undefined`, registered symbols) throw a `TypeError`. The `[[WeakMapData]]` entries are *ephemerons* — the key is held weakly, so if the key object becomes unreachable elsewhere, the entire entry is removed and the value becomes eligible for garbage collection. `WeakMap.prototype` provides only `set`, `get`, `has`, and `delete` — no iteration methods, no `clear`, no `size`. `WeakMap` is not iterable.

**Beginner-Friendly Explanation:** A `WeakMap` is like a `Map`, but with two big differences: (1) keys must be objects, and (2) if nothing else in your program is using a key object, the browser can automatically delete that entry to free memory. This makes `WeakMap` perfect for attaching private data to objects without worrying about memory leaks — when the object goes away, so does its `WeakMap` data.

### Key Characteristics

- **Object keys only:** Keys must be objects or non-registered symbols; primitives throw `TypeError`.
- **Weak references:** Keys are held weakly; unreachable keys are garbage-collected along with their entries.
- **Not enumerable:** No `size`, no `keys()`, `values()`, `entries()`, `forEach()`, or `clear()`.
- **Not iterable:** Cannot use `for...of` or spread.
- **Non-deterministic cleanup:** You cannot know when an entry will be removed; it depends on the garbage collector.
- **Privacy-oriented:** Ideal for private data attached to objects.
- **Four methods only:** `set`, `get`, `has`, `delete`.

### Prerequisites

- Basic JavaScript syntax and object references.
- Understanding of `Map` (for contrast).
- Familiarity with reference semantics and identity.
- Basic understanding of garbage collection and memory management.

### Related Programming Areas

- **Memory management:** Avoiding leaks in long-lived data structures.
- **Encapsulation:** Private data for objects and classes.
- **DOM programming:** Attaching metadata to DOM nodes without modifying them.
- **Event handling:** Tracking listeners or state per element.
- **Caching:** Caches that should not prevent key objects from being collected.

### Core Concepts / Features

The following sections cover each core concept and feature of `WeakMap`, each using the uniform structure: Definitions, Purposes, Syntax Rules and Structure, Annotated Code Examples with Expected Outputs, and Real-World Cases.

---

## `WeakMap`

### Definitions

**Core Definition:** `WeakMap` is the built-in constructor function used to create weak map instances.

**Technical Definition:** `WeakMap` is a global intrinsic object whose `[[Prototype]]` is `Function.prototype`. Calling `WeakMap()` without `new` throws a `TypeError`. When invoked with `new`, it creates a WeakMap exotic object, optionally initialised from an iterable of key-value pairs. The `[[WeakMapData]]` slot holds ephemeron records; the keys are not strongly referenced.

**Beginner-Friendly Explanation:** `WeakMap` is the "factory" you use to make new weak map objects. Write `new WeakMap()` to create an empty weak map, or `new WeakMap([[obj1, "a"], [obj2, "b"]])` to create one with initial entries.

### Purposes

- To create empty or pre-populated weak map instances.
- To provide a namespace for `WeakMap.prototype` methods.
- To enable `instanceof WeakMap` checks.
- To attach data to objects without preventing their collection.

### Syntax Rules and Structure

**General Syntax:**
```js
new WeakMap()
new WeakMap(iterable)
```

**Component Breakdown:**
- `new`: Required operator; `WeakMap()` without `new` throws `TypeError`.
- `iterable` (optional): Any iterable of `[key, value]` pairs where each key is an object or non-registered symbol.

**Syntax Rules:**
- The constructor accepts at most one argument.
- Each key must be an object or non-registered symbol; otherwise, `TypeError`.
- `null` and `undefined` are treated as "no argument" (empty weak map).

**Constraints and Limitations:**
- Keys must be objects or non-registered symbols.
- No iteration, no `size`, no `clear`.
- Garbage collection timing is implementation-dependent.
- Cannot be JSON-serialized.

### Annotated Code Examples

#### Example 1: Creating an Empty WeakMap

```js
// Step 1: Create an empty WeakMap.
const wm = new WeakMap();

// Step 2: Verify it is a WeakMap.
console.log(wm instanceof WeakMap); // true

// Step 3: Check for size (undefined — no size property).
console.log(wm.size); // undefined
```

**Expected Output:**
```
true
undefined
```

**Why This Output:** `new WeakMap()` creates an empty WeakMap. The `size` property does not exist on `WeakMap.prototype`, so `wm.size` is `undefined`.

#### Example 2: Creating a WeakMap from an Array of Pairs

```js
// Step 1: Create key objects.
const k1 = { id: 1 };
const k2 = { id: 2 };

// Step 2: Provide an array of [key, value] pairs.
const entries = [
  [k1, "first"],
  [k2, "second"],
];

// Step 3: Pass the array to the WeakMap constructor.
const wm = new WeakMap(entries);

// Step 4: Inspect.
console.log(wm.get(k1)); // "first"
console.log(wm.get(k2)); // "second"
```

**Expected Output:**
```
first
second
```

**Why This Output:** The constructor iterates the array, validates each key as an object, and stores the entries.

#### Example 3: Primitive Keys Throw

```js
try {
  const wm = new WeakMap([[1, "one"]]);
} catch (err) {
  console.log(err.name);    // "TypeError"
  console.log(err.message); // "Invalid value used as weak map key"
}
```

**Expected Output:**
```
TypeError
Invalid value used as weak map key
```

**Why This Output:** WeakMap keys must be objects or non-registered symbols. The number `1` is a primitive and is rejected.

### Real-World Cases

- **Private class fields (pre-`#`):** Storing private state keyed by instance.
- **DOM metadata:** Attaching data to DOM nodes without modifying them.
- **Memoisation caches:** Caching computed results for object keys.

---

## WeakMap Keys

### Definitions

**Core Definition:** WeakMap keys are the objects (or non-registered symbols) used to look up values in a WeakMap.

**Technical Definition:** A valid WeakMap key is any value for which `CanBeHeldWeakly(value)` returns `true`. This includes all objects (including functions, arrays, DOM nodes, proxies) and non-registered symbols (symbols created via `Symbol()`, not `Symbol.for()`). Primitive values — strings, numbers, bigints, booleans, `null`, `undefined`, and registered symbols — are invalid.

**Beginner-Friendly Explanation:** WeakMap keys must be things JavaScript can garbage-collect — mainly objects. Strings and numbers can't be keys because they're not objects and can't be tracked for cleanup.

### Purposes

- To associate data with object identities.
- To enable garbage-collectable key-value associations.
- To support private data patterns.

### Syntax Rules and Structure

**Valid Keys:**
```js
const obj = {};
const arr = [];
const fn = function () {};
const sym = Symbol("key");
const dom = document.body;

const wm = new WeakMap();
wm.set(obj, "object key");
wm.set(arr, "array key");
wm.set(fn, "function key");
wm.set(sym, "symbol key");
wm.set(dom, "DOM node key");
```

**Invalid Keys (throw TypeError):**
```js
wm.set("string", 1);     // TypeError
wm.set(42, 1);           // TypeError
wm.set(true, 1);         // TypeError
wm.set(null, 1);         // TypeError
wm.set(undefined, 1);    // TypeError
wm.set(Symbol.for("x"), 1); // TypeError (registered symbol)
```

**Constraints and Limitations:**
- Registered symbols (`Symbol.for(...)`) are invalid.
- Non-registered symbols (`Symbol(...)`) are valid.
- Proxies are valid keys, but identity is preserved through the proxy.
- `document.all` (the legacy HTMLAllCollection) is technically an object but is treated as `undefined` in some contexts; use care.

### Annotated Code Examples

#### Example 1: Object, Function, and Symbol Keys

```js
const wm = new WeakMap();

const obj = { name: "object" };
const fn = () => "function";
const sym = Symbol("unique");

wm.set(obj, "stored for object");
wm.set(fn, "stored for function");
wm.set(sym, "stored for symbol");

console.log(wm.get(obj));  // "stored for object"
console.log(wm.get(fn));   // "stored for function"
console.log(wm.get(sym));  // "stored for symbol"
```

**Expected Output:**
```
stored for object
stored for function
stored for symbol
```

**Why This Output:** Objects, functions, and non-registered symbols are all valid weak keys.

#### Example 2: Registered Symbols Are Invalid

```js
const wm = new WeakMap();
const registered = Symbol.for("app.key");

try {
  wm.set(registered, "value");
} catch (err) {
  console.log(err.name);    // "TypeError"
  console.log(err.message); // "Invalid value used as weak map key"
}
```

**Expected Output:**
```
TypeError
Invalid value used as weak map key
```

**Why This Output:** Registered symbols are part of the global symbol registry and are never garbage-collected, so they cannot be weak keys.

### Real-World Cases

- **DOM nodes:** Attaching per-element metadata.
- **Class instances:** Storing private state.
- **Function caches:** Memoising results per function reference.

---

## WeakMap Methods

### Definitions

**Core Definition:** WeakMap methods are the four functions on `WeakMap.prototype` — `set`, `get`, `has`, and `delete` — that manipulate weak map entries.

**Technical Definition:** `WeakMap.prototype` defines exactly four methods and one accessor (`Symbol.toStringTag`). There are no iteration methods, no `size`, and no `clear`. Each method operates on the `[[WeakMapData]]` internal slot.

**Beginner-Friendly Explanation:** WeakMaps have only four operations: add an entry, look up a value, check if a key exists, and remove an entry. That's it — no looping, no counting.

---

### `set()`

#### Definitions

- **Core Definition:** `set()` adds or updates a key-value pair in the WeakMap.
- **Technical Definition:** `WeakMap.prototype.set(key, value)` validates that `key` can be held weakly (throws `TypeError` otherwise), then searches `[[WeakMapData]]` for a record with the same key identity. If found, updates the value; otherwise appends a new ephemeron record. Returns the WeakMap for chaining.
- **Beginner-Friendly Explanation:** `set()` is how you store a value under an object key.

#### Purposes

- To insert a new entry.
- To update the value of an existing key.
- To enable method chaining.

#### Syntax Rules and Structure

```js
weakMap.set(key, value)
```

**Component Breakdown:**
- `key`: An object or non-registered symbol.
- `value`: Any JavaScript value.
- Returns: The same WeakMap (for chaining).

**Constraints and Limitations:**
- Primitive keys throw `TypeError`.
- Updating an existing key preserves its identity but changes its value.
- No observable side effects on iteration (no iteration exists).

#### Annotated Code Examples

```js
const wm = new WeakMap();
const key = { id: 1 };

// Step 1: Insert with chaining.
wm.set(key, "first").set({ id: 2 }, "second");

// Step 2: Update the value for the first key.
wm.set(key, "updated");

console.log(wm.get(key)); // "updated"
```

**Expected Output:**
```
updated
```

**Why This Output:** `set()` returns the WeakMap, allowing chaining. Updating the value for `key` overwrites the previous value.

#### Real-World Cases

- **Private state:** `privateData.set(this, { secret: 42 })`.
- **DOM metadata:** `elementData.set(el, { clicks: 0 })`.
- **Caches:** `cache.set(objKey, computedValue)`.

---

### `get()`

#### Definitions

- **Core Definition:** `get()` retrieves the value associated with a key in the WeakMap.
- **Technical Definition:** `WeakMap.prototype.get(key)` searches `[[WeakMapData]]` for a record with the same key identity and returns its value, or `undefined` if not found or if `key` is not a valid weak key.
- **Beginner-Friendly Explanation:** `get()` is how you look up a value in a WeakMap.

#### Purposes

- To retrieve stored values.
- To check for presence indirectly (though `has()` is preferred).
- To read private state, cached values, or metadata.

#### Syntax Rules and Structure

```js
weakMap.get(key)
```

**Returns:** The associated value, or `undefined`.

**Constraints and Limitations:**
- Returns `undefined` for invalid (primitive) keys instead of throwing.
- Cannot distinguish "key absent" from "key present with value `undefined`". Use `has()` for that.
- Object keys must be the same reference.

#### Annotated Code Examples

```js
const wm = new WeakMap();
const key = { id: 1 };
wm.set(key, "value");

console.log(wm.get(key));         // "value"
console.log(wm.get({ id: 1 }));   // undefined (different reference)
console.log(wm.get(42));          // undefined (invalid key type)
```

**Expected Output:**
```
value
undefined
undefined
```

**Why This Output:** Only the original `key` reference matches. A new object literal is distinct. Primitives return `undefined` instead of throwing.

#### Real-World Cases

- **Reading private data:** `privateData.get(this)`.
- **Cache lookup:** `cache.get(objKey)`.
- **Metadata retrieval:** `elementData.get(el)?.clicks`.

---

### `has()`

#### Definitions

- **Core Definition:** `has()` tests whether a key exists in the WeakMap.
- **Technical Definition:** `WeakMap.prototype.has(key)` returns `true` if a record with the same key identity exists in `[[WeakMapData]]`, else `false`. Returns `false` for invalid (primitive) keys.
- **Beginner-Friendly Explanation:** `has()` answers "Is this object a key in the WeakMap?"

#### Purposes

- To check for key existence without retrieving the value.
- To distinguish "absent" from "present with `undefined` value".
- To guard against overwriting existing entries.

#### Syntax Rules and Structure

```js
weakMap.has(key)
```

**Returns:** Boolean.

**Constraints and Limitations:**
- Returns `false` for primitive keys (does not throw).
- Object keys must match by reference.

#### Annotated Code Examples

```js
const wm = new WeakMap();
const key = { id: 1 };
wm.set(key, undefined); // value is undefined, but key is present

console.log(wm.has(key));         // true
console.log(wm.has({ id: 1 }));   // false
console.log(wm.has("string"));    // false (primitive, no throw)
console.log(wm.has(Symbol.for("x"))); // false (registered symbol)
```

**Expected Output:**
```
true
false
false
false
```

**Why This Output:** `has()` distinguishes presence from absence. Invalid keys return `false` rather than throwing.

#### Real-World Cases

- **Lazy initialisation:** `if (!privateData.has(this)) privateData.set(this, init())`.
- **Cache validation:** `if (cache.has(objKey)) return cache.get(objKey)`.
- **Ownership checks:** `if (ownerMap.has(obj))`.

---

### `delete()`

#### Definitions

- **Core Definition:** `delete()` removes a key-value pair from the WeakMap.
- **Technical Definition:** `WeakMap.prototype.delete(key)` finds a record with the same key identity, removes it from `[[WeakMapData]]`, and returns `true`. Returns `false` if no such record exists or if `key` is invalid.
- **Beginner-Friendly Explanation:** `delete()` removes an entry from the WeakMap.

#### Purposes

- To remove entries that are no longer needed.
- To implement cache eviction.
- To manage lifecycle of keyed resources.

#### Syntax Rules and Structure

```js
weakMap.delete(key)
```

**Returns:** `true` if an entry was removed, `false` otherwise.

**Constraints and Limitations:**
- Returns `false` for primitive keys (does not throw).
- Object keys must match by reference.
- Removing an entry does not affect the key object itself.

#### Annotated Code Examples

```js
const wm = new WeakMap();
const key = { id: 1 };
wm.set(key, "value");

console.log(wm.delete(key));      // true
console.log(wm.has(key));         // false
console.log(wm.delete(key));      // false (already removed)
console.log(wm.delete("string")); // false (invalid key, no throw)
```

**Expected Output:**
```
true
false
false
false
```

**Why This Output:** The first `delete` removes the entry and returns `true`. Subsequent deletes return `false`. Invalid keys return `false`.

#### Real-World Cases

- **Cache eviction:** `cache.delete(objKey)`.
- **Cleanup:** Removing metadata when an object is disposed.
- **Session termination:** `sessions.delete(sessionObj)`.

---

## WeakMap Use Cases

### Definitions

**Core Definition:** WeakMap use cases are scenarios where WeakMap's object-key-only and garbage-collectable properties make it the right choice.

**Technical Definition:** WeakMaps are appropriate when (a) keys are objects, (b) entries should not prevent garbage collection of keys, and (c) iteration or size is not required. They are used for private data, DOM metadata, caches, and event listener tracking.

**Beginner-Friendly Explanation:** Use a WeakMap when you want to attach data to an object without modifying it and without preventing it from being garbage-collected.

### Common Use Cases

1. **Private Data for Classes (Pre-`#` Syntax)**
   ```js
   const _balance = new WeakMap();
   class Account {
     constructor(initial) {
       _balance.set(this, initial);
     }
     get balance() {
       return _balance.get(this);
     }
   }
   ```

2. **DOM Node Metadata**
   ```js
   const clicks = new WeakMap();
   document.querySelectorAll("button").forEach((btn) => {
     clicks.set(btn, 0);
     btn.addEventListener("click", () => {
       clicks.set(btn, clicks.get(btn) + 1);
     });
   });
   ```

3. **Memoisation Caches Keyed by Object**
   ```js
   const cache = new WeakMap();
   function processConfig(config) {
     if (cache.has(config)) return cache.get(config);
     const result = expensiveTransform(config);
     cache.set(config, result);
     return result;
   }
   ```

4. **Tracking Event Listeners**
   ```js
   const listeners = new WeakMap();
   function addListener(el, fn) {
     if (!listeners.has(el)) listeners.set(el, new Set());
     listeners.get(el).add(fn);
   }
   ```

5. **Avoiding Memory Leaks in Long-Lived Structures**
   ```js
   // Attaching data to transient objects without leaking.
   const transientData = new WeakMap();
   ```

### Annotated Code Examples

#### Example 1: Private Data with WeakMap

```js
// Step 1: Create a WeakMap to hold private state.
const _balance = new WeakMap();

class Account {
  constructor(initialBalance) {
    // Step 2: Store private data keyed by the instance.
    _balance.set(this, initialBalance);
  }

  deposit(amount) {
    _balance.set(this, _balance.get(this) + amount);
  }

  get balance() {
    return _balance.get(this);
  }
}

const acc = new Account(100);
acc.deposit(50);
console.log(acc.balance);    // 150
console.log(_balance.has(acc)); // true (but not accessible outside)
```

**Expected Output:**
```
150
true
```

**Why This Output:** `_balance` is module-scoped, so external code cannot access it directly. But it still works internally, and when `acc` is garbage-collected, the WeakMap entry is removed.

#### Example 2: DOM Metadata

```js
const metadata = new WeakMap();

function trackInteraction(element) {
  if (!metadata.has(element)) {
    metadata.set(element, { clicks: 0, hovers: 0 });
  }
  const data = metadata.get(element);
  data.clicks++;
  return data;
}

const btn = document.createElement("button");
trackInteraction(btn);
trackInteraction(btn);
console.log(metadata.get(btn).clicks); // 2
```

**Expected Output:**
```
2
```

**Why This Output:** The DOM element is used as a weak key. When the button is removed from the DOM and no longer referenced, the metadata is automatically garbage-collected.

### Real-World Cases

- **React internals:** Uses WeakMaps for fiber node metadata.
- **Vue reactivity:** Tracks dependencies with WeakMaps.
- **Lodash:** Uses WeakMap for memoisation.
- **jQuery:** Uses a WeakMap-like internal data store for element data.

---

## WeakMap vs Map

### Definitions

**Core Definition:** `WeakMap` and `Map` are both key-value collections, but they differ in key types, iterability, size access, and garbage-collection semantics.

**Technical Definition:** `Map` allows any key type and provides full iteration and size access, but holds keys strongly. `WeakMap` restricts keys to objects/non-registered symbols, does not allow iteration or size access, and holds keys weakly, allowing garbage collection.

**Beginner-Friendly Explanation:** Use `Map` when you need to iterate, count, or use primitive keys. Use `WeakMap` when you want to attach data to objects without preventing their cleanup.

### Comparison Table

| Feature | WeakMap | Map |
|---|---|---|
| Key types | Objects + non-registered symbols | Any value |
| Key reference | Weak | Strong |
| Garbage collection | Entries removed when key unreachable | Entries persist until deleted |
| Iteration | Not iterable | Iterable |
| `size` | Not available | Available |
| `clear()` | Not available | Available |
| `forEach()` | Not available | Available |
| `keys()`, `values()`, `entries()` | Not available | Available |
| JSON serialisation | No | No (but can convert) |
| Use case | Object metadata, private data | General key-value store |

### Purposes

- To choose the right structure based on key types and memory needs.
- To avoid memory leaks in long-lived data structures.
- To enable iteration when required.

### Annotated Code Examples

#### Example 1: Map Retains Keys, WeakMap Does Not

```js
let obj = { id: 1 };

const map = new Map();
const wm = new WeakMap();

map.set(obj, "in map");
wm.set(obj, "in weakmap");

console.log(map.size); // 1
console.log(wm.has(obj)); // true

// Remove the only external reference.
obj = null;

// The Map still holds the object; the WeakMap entry will be collected
// (timing is non-deterministic, so we cannot observe it directly).
console.log(map.size); // 1 (still holds the object)
```

**Expected Output:**
```
1
true
1
```

**Why This Output:** The `Map` holds a strong reference to the original object even after `obj` is set to `null`, so `map.size` remains 1. The `WeakMap` entry will eventually be removed, but this is not observable from code.

#### Example 2: Iterating a Map (Impossible with WeakMap)

```js
const map = new Map([[{ id: 1 }, "a"], [{ id: 2 }, "b"]]);
for (const [key, value] of map) {
  console.log(value);
}
// WeakMap equivalent would throw: wm is not iterable.
```

**Expected Output:**
```
a
b
```

**Why This Output:** Maps are iterable; WeakMaps are not.

### Real-World Cases

- **Map:** Configuration stores, caches with primitive keys, ordered dictionaries.
- **WeakMap:** DOM metadata, private class state, memoisation keyed by objects.

---

## Garbage Collection

### Definitions

**Core Definition:** Garbage collection (GC) is the automatic process by which a JavaScript engine reclaims memory occupied by objects that are no longer reachable from the program.

**Technical Definition:** Modern JavaScript engines (V8, SpiderMonkey, JavaScriptCore) use generational, mark-and-sweep collectors with incremental and concurrent phases. Reachability is determined from roots (global object, current stack, closures). WeakMap keys are held via ephemerons: the key is only considered reachable if the key is reachable through a path that does not go through the WeakMap itself. When the key becomes unreachable, the entry is removed and the value becomes eligible for collection.

**Beginner-Friendly Explanation:** Garbage collection is JavaScript's automatic memory cleaner. When an object is no longer used anywhere in your program, the engine deletes it to free memory. WeakMaps participate in this: if the key object is no longer used, the WeakMap entry disappears too.

### Purposes

- To free memory automatically without manual deallocation.
- To prevent memory leaks in long-running applications.
- To allow WeakMap entries to disappear when keys are no longer needed.

### Syntax Rules and Structure

There is no explicit syntax for garbage collection. You influence it by:

- Maintaining or dropping references to key objects.
- Using WeakMap instead of Map when keys are objects and should be collectable.

**Constraints and Limitations:**
- GC timing is non-deterministic; you cannot know when an entry is removed.
- `WeakMap` does not expose a way to observe GC (no `size`, no iteration).
- FinalizationRegistry can observe collection of objects, but is not part of WeakMap.

### Annotated Code Examples

#### Example 1: Observing Collection with FinalizationRegistry

```js
// Step 1: Create a FinalizationRegistry to observe collection.
const registry = new FinalizationRegistry((heldValue) => {
  console.log(`Collected: ${heldValue}`);
});

// Step 2: Create a WeakMap.
const wm = new WeakMap();

// Step 3: Create a key object and register it for observation.
let key = { id: 1 };
wm.set(key, "value");
registry.register(key, "key-1");

// Step 4: Drop the only strong reference.
key = null;

// Step 5: In a real environment, GC would eventually run.
// In a browser/Node session, you may see "Collected: key-1" later.
// Note: GC timing is non-deterministic.
```

**Expected Output (eventually, non-deterministic):**
```
Collected: key-1
```

**Why This Output:** When `key` becomes unreachable, the GC may collect it. The `FinalizationRegistry` callback fires after collection. The WeakMap entry is also removed. The exact timing depends on the engine.

### Real-World Cases

- **Long-lived SPAs:** WeakMaps prevent leaks from DOM metadata.
- **Node.js servers:** WeakMaps avoid retaining request-scoped objects.
- **Caches:** WeakMap caches release memory when keys are no longer used.

---

## Memory Management

### Definitions

**Core Definition:** Memory management is the practice of controlling how memory is allocated, used, and released in a program to avoid leaks and excessive consumption.

**Technical Definition:** In JavaScript, memory management is largely automatic via garbage collection, but developers influence it through reference management, data structure choices, and lifecycle patterns. WeakMaps are a key tool for object-keyed data that should not prevent collection. Other tools include `WeakRef`, `FinalizationRegistry`, and `WeakSet`.

**Beginner-Friendly Explanation:** Memory management means being careful not to hold onto things you no longer need. WeakMaps help by automatically releasing data when the associated object is no longer used.

### Purposes

- To avoid memory leaks in long-running applications.
- To reduce memory footprint.
- To improve performance by allowing GC to reclaim memory.
- To attach data to objects without extending their lifetime.

### Syntax Rules and Structure

There is no special syntax. Practice includes:

- Using `WeakMap` / `WeakSet` for object-keyed data.
- Using `WeakRef` for optional references.
- Avoiding accidental retention (closures, event listeners, global caches).
- Using `FinalizationRegistry` for cleanup notifications.

**Constraints and Limitations:**
- GC is non-deterministic; you cannot force it.
- WeakMap data is not observable; you cannot check if it's been collected.
- WeakRef and FinalizationRegistry should be used sparingly.

### Annotated Code Examples

#### Example 1: Avoiding a Memory Leak with WeakMap

```js
// BAD: Map retains all DOM nodes forever.
const badCache = new Map();
function badAttach(el) {
  badCache.set(el, { visited: true });
}
// Even after el is removed from the DOM, it stays in badCache.

// GOOD: WeakMap releases entries when nodes are removed.
const goodCache = new WeakMap();
function goodAttach(el) {
  goodCache.set(el, { visited: true });
}
// When el is removed from the DOM and no longer referenced,
// the entry is automatically removed.
```

**Expected Output:** No output; this is a structural comparison.

**Why This Output:** The Map-based cache holds strong references to DOM nodes, preventing GC. The WeakMap-based cache holds weak references, allowing GC.

#### Example 2: Combined WeakMap and FinalizationRegistry

```js
const registry = new FinalizationRegistry((id) => {
  console.log(`Cleanup for ID ${id}`);
});

const cache = new WeakMap();

function register(obj, id) {
  cache.set(obj, { id, data: "..." });
  registry.register(obj, id);
}

let obj = { name: "temporary" };
register(obj, 42);
obj = null;
// Eventually: "Cleanup for ID 42"
```

**Expected Output (eventually, non-deterministic):**
```
Cleanup for ID 42
```

**Why This Output:** When `obj` is collected, the registry callback fires, allowing cleanup logic. The WeakMap entry is also removed.

### Real-World Cases

- **SPAs:** WeakMap for component metadata; avoids leaks on unmount.
- **Node.js:** WeakMap for per-request state in middleware.
- **Libraries:** WeakMap for caching per-object computations.

---

## References

- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – WeakMap Objects — https://tc39.es/ecma262/multipage/keyed-collections.html#sec-weakmap-objects
- MDN Web Docs – WeakMap — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakMap
- MDN Web Docs – WeakMap.prototype.set() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakMap/set
- MDN Web Docs – WeakMap.prototype.get() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakMap/get
- MDN Web Docs – WeakMap.prototype.has() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakMap/has
- MDN Web Docs – WeakMap.prototype.delete() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakMap/delete
- MDN Web Docs – Memory management — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Memory_management
- MDN Web Docs – FinalizationRegistry — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/FinalizationRegistry
- MDN Web Docs – WeakRef — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakRef
- V8 Blog – Ephemeron Garbage Collection — https://v8.dev/blog/high-performance-gc
- V8 Blog – WeakRefs and FinalizationRegistry — https://v8.dev/features/weak-references
- TC39 Proposal – WeakRefs — https://github.com/tc39/proposal-weakrefs