# JavaScript `WeakSet` — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A `WeakSet` is a built-in JavaScript collection that stores unique object values weakly, meaning it does not prevent garbage collection of the objects it contains.

**Technical Definition:** Per ECMAScript 2025 (ECMA-262, §24.4), `WeakSet` is a constructor whose instances are exotic objects with a `[[WeakSetData]]` internal slot containing a List of unique values. Values must be objects or non-registered symbols; primitive values throw a `TypeError` when passed to `add()`. The `[[WeakSetData]]` entries are held weakly: if a value becomes unreachable elsewhere, the entry is removed and the value becomes eligible for garbage collection. `WeakSet.prototype` provides only `add`, `has`, and `delete` — no iteration methods, no `clear`, no `size`. `WeakSet` is not iterable.

**Beginner-Friendly Explanation:** A `WeakSet` is like a `Set`, but with two big differences: (1) it can only hold objects (not strings, numbers, or other primitives), and (2) if nothing else in your program is using an object, the browser can automatically remove it from the WeakSet to free memory. This makes `WeakSet` perfect for tracking "have I seen this object?" without worrying about memory leaks.

### Key Characteristics

- **Object values only:** Values must be objects or non-registered symbols; primitives throw `TypeError`.
- **Weak references:** Values are held weakly; unreachable values are garbage-collected along with their entries.
- **Not enumerable:** No `size`, no `keys()`, `values()`, `entries()`, or `forEach()`.
- **Not iterable:** Cannot use `for...of` or spread.
- **Non-deterministic cleanup:** You cannot know when an entry will be removed; it depends on the garbage collector.
- **Three methods only:** `add`, `has`, `delete`.
- **Uniqueness:** Duplicate values are silently ignored.

### Prerequisites

- Basic JavaScript syntax and object references.
- Understanding of `Set` (for contrast).
- Familiarity with reference semantics and identity.
- Basic understanding of garbage collection and memory management.
- Awareness of `WeakMap` (parallel structure).

### Related Programming Areas

- **Memory management:** Avoiding leaks in long-lived data structures.
- **Object tracking:** Marking objects as "seen" or "processed."
- **Graph algorithms:** Tracking visited nodes without retaining them.
- **Event handling:** Tracking elements that have had listeners attached.
- **Security:** Marking trusted objects (branding).

### Core Concepts / Features

The following sections cover each core concept and feature of `WeakSet`, each using the uniform structure: Definitions, Purposes, Syntax Rules and Structure, Annotated Code Examples with Expected Outputs, and Real-World Cases.

---

## `WeakSet`

### Definitions

**Core Definition:** `WeakSet` is the built-in constructor function used to create weak set instances.

**Technical Definition:** `WeakSet` is a global intrinsic object whose `[[Prototype]]` is `Function.prototype`. Calling `WeakSet()` without `new` throws a `TypeError`. When invoked with `new`, it creates a WeakSet exotic object, optionally initialised from an iterable of values. The `[[WeakSetData]]` slot holds unique values weakly. `WeakSet.prototype` exposes only `add`, `has`, and `delete`, plus `Symbol.toStringTag`.

**Beginner-Friendly Explanation:** `WeakSet` is the "factory" you use to make new weak set objects. Write `new WeakSet()` to create an empty weak set, or `new WeakSet([obj1, obj2])` to create one with initial values.

### Purposes

- To create empty or pre-populated weak set instances.
- To provide a namespace for `WeakSet.prototype` methods.
- To enable `instanceof WeakSet` checks.
- To track object membership without preventing collection.

### Syntax Rules and Structure

**General Syntax:**
```js
new WeakSet()
new WeakSet(iterable)
```

**Component Breakdown:**
- `new`: Required operator; `WeakSet()` without `new` throws `TypeError`.
- `iterable` (optional): Any iterable of values where each value is an object or non-registered symbol.

**Syntax Rules:**
- The constructor accepts at most one argument.
- Each value must be an object or non-registered symbol; otherwise, `TypeError`.
- `null` and `undefined` are treated as "no argument" (empty weak set).

**Constraints and Limitations:**
- Values must be objects or non-registered symbols.
- No iteration, no `size`, no `clear`.
- Garbage collection timing is implementation-dependent.
- Cannot be JSON-serialized.

### Annotated Code Examples

#### Example 1: Creating an Empty WeakSet

```js
// Step 1: Create an empty WeakSet.
const ws = new WeakSet();

// Step 2: Verify it is a WeakSet.
console.log(ws instanceof WeakSet); // true

// Step 3: Check for size (undefined — no size property).
console.log(ws.size); // undefined
```

**Expected Output:**
```
true
undefined
```

**Why This Output:** `new WeakSet()` creates an empty WeakSet. The `size` property does not exist on `WeakSet.prototype`, so `ws.size` is `undefined`.

#### Example 2: Creating a WeakSet from an Array of Objects

```js
// Step 1: Create value objects.
const a = { id: 1 };
const b = { id: 2 };
const c = { id: 3 };

// Step 2: Provide an array of values.
const values = [a, b, c, a]; // Note: duplicate `a`

// Step 3: Pass the array to the WeakSet constructor.
const ws = new WeakSet(values);

// Step 4: Inspect membership.
console.log(ws.has(a)); // true
console.log(ws.has(b)); // true
console.log(ws.has(c)); // true
console.log(ws.has({ id: 1 })); // false (different reference)
```

**Expected Output:**
```
true
true
true
false
```

**Why This Output:** The constructor iterates the array, validates each value as an object, and adds it. Duplicate `a` is silently ignored. A new object literal is a distinct reference.

#### Example 3: Primitive Values Throw

```js
try {
  const ws = new WeakSet([1, 2, 3]);
} catch (err) {
  console.log(err.name);    // "TypeError"
  console.log(err.message); // "Invalid value used in weak set"
}
```

**Expected Output:**
```
TypeError
Invalid value used in weak set
```

**Why This Output:** WeakSet values must be objects or non-registered symbols. The number `1` is a primitive and is rejected.

### Real-World Cases

- **Visited node tracking:** Marking graph nodes as visited without retaining them.
- **Event listener deduplication:** Ensuring a listener is attached only once per element.
- **Object branding:** Marking objects as trusted or validated.
- **Processed item tracking:** Recording that an object has been handled.

---

## WeakSet Methods

### Definitions

**Core Definition:** WeakSet methods are the three functions on `WeakSet.prototype` — `add`, `has`, and `delete` — that manipulate weak set membership.

**Technical Definition:** `WeakSet.prototype` defines exactly three methods and one accessor (`Symbol.toStringTag`). There are no iteration methods, no `size`, and no `clear`. Each method operates on the `[[WeakSetData]]` internal slot.

**Beginner-Friendly Explanation:** WeakSets have only three operations: add an object, check if an object is present, and remove an object. That's it — no looping, no counting.

---

### `add()`

#### Definitions

- **Core Definition:** `add()` inserts a value into the WeakSet if it is not already present.
- **Technical Definition:** `WeakSet.prototype.add(value)` validates that `value` can be held weakly (throws `TypeError` otherwise), then searches `[[WeakSetData]]` for an identical value. If absent, appends it. Returns the WeakSet for chaining.
- **Beginner-Friendly Explanation:** `add()` puts an object into the WeakSet. If it's already there, nothing changes.

#### Purposes

- To insert a new unique value.
- To enable method chaining.
- To mark an object as "seen" or "processed."

#### Syntax Rules and Structure

```js
weakSet.add(value)
```

**Component Breakdown:**
- `value`: An object or non-registered symbol.
- Returns: The same WeakSet (for chaining).

**Constraints and Limitations:**
- Primitive values throw `TypeError`.
- Adding a duplicate is a no-op.
- Object values are compared by reference.

#### Annotated Code Examples

```js
const ws = new WeakSet();
const a = { id: 1 };
const b = { id: 2 };

// Step 1: Add values with chaining.
ws.add(a).add(b);

// Step 2: Attempt to add a duplicate.
ws.add(a);

console.log(ws.has(a)); // true
console.log(ws.has(b)); // true

// Step 3: Attempt to add a primitive (throws).
try {
  ws.add(42);
} catch (err) {
  console.log(err.name); // "TypeError"
}
```

**Expected Output:**
```
true
true
TypeError
```

**Why This Output:** `add()` returns the WeakSet, allowing chaining. Duplicate `a` is ignored. The primitive `42` throws `TypeError`.

#### Real-World Cases

- **Visited sets:** `visited.add(node)` in graph traversal.
- **Listener tracking:** `attached.add(element)`.
- **Branding:** `trusted.add(obj)` to mark validated objects.

---

### `has()`

#### Definitions

- **Core Definition:** `has()` tests whether a value exists in the WeakSet.
- **Technical Definition:** `WeakSet.prototype.has(value)` returns `true` if an identical value exists in `[[WeakSetData]]`, else `false`. Returns `false` for primitive values (does not throw).
- **Beginner-Friendly Explanation:** `has()` answers "Is this object in the WeakSet?"

#### Purposes

- To check membership efficiently (O(1) average).
- To guard against duplicate processing.
- To implement conditional logic based on presence.

#### Syntax Rules and Structure

```js
weakSet.has(value)
```

**Returns:** Boolean.

**Constraints and Limitations:**
- Returns `false` for primitive values (does not throw).
- Object values must match by reference.
- Non-registered symbols are valid values.

#### Annotated Code Examples

```js
const ws = new WeakSet();
const a = { id: 1 };
ws.add(a);

console.log(ws.has(a));          // true
console.log(ws.has({ id: 1 }));  // false (different reference)
console.log(ws.has("string"));   // false (primitive, no throw)
console.log(ws.has(42));         // false (primitive, no throw)

// Non-registered symbol as value
const sym = Symbol("unique");
ws.add(sym);
console.log(ws.has(sym));        // true
console.log(ws.has(Symbol.for("x"))); // false (registered symbol, not added)
```

**Expected Output:**
```
true
false
false
false
true
false
```

**Why This Output:** Only the original `a` reference matches. Primitives return `false` without throwing. Non-registered symbols are valid values.

#### Real-World Cases

- **Cycle detection:** `if (visited.has(obj)) return;`.
- **Idempotency:** `if (processed.has(item)) return;`.
- **Trust checks:** `if (trusted.has(obj))`.

---

### `delete()`

#### Definitions

- **Core Definition:** `delete()` removes a value from the WeakSet.
- **Technical Definition:** `WeakSet.prototype.delete(value)` finds an identical value, removes it from `[[WeakSetData]]`, and returns `true`. Returns `false` if no such value exists or if `value` is invalid.
- **Beginner-Friendly Explanation:** `delete()` removes an object from the WeakSet.

#### Purposes

- To remove values that are no longer needed.
- To implement dynamic membership.
- To allow re-processing of objects.

#### Syntax Rules and Structure

```js
weakSet.delete(value)
```

**Returns:** `true` if a value was removed, `false` otherwise.

**Constraints and Limitations:**
- Returns `false` for primitive values (does not throw).
- Object values must match by reference.
- Removing a value does not affect the value object itself.

#### Annotated Code Examples

```js
const ws = new WeakSet();
const a = { id: 1 };
ws.add(a);

console.log(ws.delete(a));       // true
console.log(ws.has(a));          // false
console.log(ws.delete(a));       // false (already removed)
console.log(ws.delete("string")); // false (invalid value, no throw)
```

**Expected Output:**
```
true
false
false
false
```

**Why This Output:** The first `delete` removes the value and returns `true`. Subsequent deletes return `false`. Invalid values return `false`.

#### Real-World Cases

- **Cache eviction:** `cache.delete(obj)`.
- **Cleanup:** Removing an element from a "processed" set.
- **Re-processing:** Allowing an object to be handled again.

---

## WeakSet Use Cases

### Definitions

**Core Definition:** WeakSet use cases are scenarios where WeakSet's object-only and garbage-collectable properties make it the right choice.

**Technical Definition:** WeakSets are appropriate when (a) values are objects, (b) entries should not prevent garbage collection of values, and (c) iteration or size is not required. They are used for visited tracking, deduplication, branding, and idempotency.

**Beginner-Friendly Explanation:** Use a WeakSet when you want to remember "I've seen this object" without preventing it from being garbage-collected.

### Common Use Cases

1. **Visited Node Tracking (Graph Traversal)**
   ```js
   const visited = new WeakSet();
   function dfs(node) {
     if (visited.has(node)) return;
     visited.add(node);
     node.neighbours.forEach(dfs);
   }
   ```

2. **Preventing Duplicate Event Listeners**
   ```js
   const attached = new WeakSet();
   function attachOnce(element, handler) {
     if (attached.has(element)) return;
     element.addEventListener("click", handler);
     attached.add(element);
   }
   ```

3. **Object Branding / Trust Marking**
   ```js
   const trusted = new WeakSet();
   function markTrusted(obj) { trusted.add(obj); }
   function isTrusted(obj) { return trusted.has(obj); }
   ```

4. **Cycle Detection in Recursive Structures**
   ```js
   const seen = new WeakSet();
   function traverse(obj) {
     if (typeof obj !== "object" || obj === null) return;
     if (seen.has(obj)) return;
     seen.add(obj);
     for (const key in obj) traverse(obj[key]);
   }
   ```

5. **Idempotency Guards**
   ```js
   const processed = new WeakSet();
   function processOnce(item) {
     if (processed.has(item)) return;
     processed.add(item);
     // ... process ...
   }
   ```

6. **Weak Caching of Object-Level Flags**
   ```js
   const initialised = new WeakSet();
   function ensureInitialised(component) {
     if (initialised.has(component)) return;
     component.init();
     initialised.add(component);
   }
   ```

### Annotated Code Examples

#### Example 1: Graph Traversal with Visited WeakSet

```js
// Step 1: Define a graph using objects as nodes.
const nodeA = { name: "A", neighbours: [] };
const nodeB = { name: "B", neighbours: [] };
const nodeC = { name: "C", neighbours: [] };
nodeA.neighbours.push(nodeB, nodeC);
nodeB.neighbours.push(nodeA, nodeC);
nodeC.neighbours.push(nodeA);

// Step 2: Traverse with a visited WeakSet.
const visited = new WeakSet();
function dfs(node) {
  if (visited.has(node)) return;
  visited.add(node);
  console.log("Visited:", node.name);
  node.neighbours.forEach(dfs);
}

dfs(nodeA);
```

**Expected Output:**
```
Visited: A
Visited: B
Visited: C
```

**Why This Output:** `visited` prevents revisiting nodes. The WeakSet does not retain nodes after traversal completes; once `nodeA`, `nodeB`, and `nodeC` are unreachable from the program, they can be garbage-collected.

#### Example 2: Attaching Event Listeners Only Once

```js
const attached = new WeakSet();

function attachOnce(element, handler) {
  if (attached.has(element)) {
    console.log("Listener already attached.");
    return;
  }
  element.addEventListener("click", handler);
  attached.add(element);
  console.log("Listener attached.");
}

const btn = document.createElement("button");
const handler = () => console.log("Clicked!");

attachOnce(btn, handler); // "Listener attached."
attachOnce(btn, handler); // "Listener already attached."
```

**Expected Output:**
```
Listener attached.
Listener already attached.
```

**Why This Output:** The WeakSet tracks which elements already have the listener. Duplicate calls are prevented. When `btn` is removed from the DOM and no longer referenced, the WeakSet entry is garbage-collected.

### Real-World Cases

- **React internals:** Uses WeakSets to track processed objects during reconciliation.
- **Vue reactivity:** Tracks raw vs. reactive objects with WeakSets.
- **GraphQL clients:** Uses WeakSets for circular reference detection.
- **Libraries:** Use WeakSets for "branding" objects as belonging to a specific library.

---

## WeakSet vs Set

### Definitions

**Core Definition:** `WeakSet` and `Set` are both collections of unique values, but they differ in value types, iterability, size access, and garbage-collection semantics.

**Technical Definition:** `Set` allows any value type and provides full iteration and size access, but holds values strongly. `WeakSet` restricts values to objects/non-registered symbols, does not allow iteration or size access, and holds values weakly, allowing garbage collection.

**Beginner-Friendly Explanation:** Use `Set` when you need to iterate, count, or store primitives. Use `WeakSet` when you want to track objects without preventing their cleanup.

### Comparison Table

| Feature | WeakSet | Set |
|---|---|---|
| Value types | Objects + non-registered symbols | Any value |
| Value reference | Weak | Strong |
| Garbage collection | Entries removed when value unreachable | Entries persist until deleted |
| Iteration | Not iterable | Iterable |
| `size` | Not available | Available |
| `clear()` | Not available | Available |
| `forEach()` | Not available | Available |
| `keys()`, `values()`, `entries()` | Not available | Available |
| JSON serialisation | No | No (but can convert) |
| Use case | Object tracking, visited sets | General unique-value collection |

### Purposes

- To choose the right structure based on value types and memory needs.
- To avoid memory leaks in long-lived data structures.
- To enable iteration when required.

### Annotated Code Examples

#### Example 1: Set Retains Values, WeakSet Does Not

```js
let obj = { id: 1 };

const set = new Set();
const ws = new WeakSet();

set.add(obj);
ws.add(obj);

console.log(set.size);   // 1
console.log(ws.has(obj)); // true

// Remove the only external reference.
obj = null;

// The Set still holds the object; the WeakSet entry will be collected
// (timing is non-deterministic, so we cannot observe it directly).
console.log(set.size);   // 1 (still holds the object)
```

**Expected Output:**
```
1
true
1
```

**Why This Output:** The `Set` holds a strong reference to the original object even after `obj` is set to `null`, so `set.size` remains 1. The `WeakSet` entry will eventually be removed, but this is not observable from code.

#### Example 2: Iterating a Set (Impossible with WeakSet)

```js
const set = new Set([1, 2, 3]);
for (const value of set) {
  console.log(value);
}

// WeakSet equivalent would throw: ws is not iterable.
```

**Expected Output:**
```
1
2
3
```

**Why This Output:** Sets are iterable; WeakSets are not.

#### Example 3: Primitive Values

```js
// Set accepts primitives.
const set = new Set([1, "two", true]);
console.log(set.size); // 3

// WeakSet rejects primitives.
try {
  const ws = new WeakSet([1, "two", true]);
} catch (err) {
  console.log(err.name); // "TypeError"
}
```

**Expected Output:**
```
3
TypeError
```

**Why This Output:** Sets accept any value; WeakSets accept only objects and non-registered symbols.

### Real-World Cases

- **Set:** Deduplication of primitives, tag systems, membership testing.
- **WeakSet:** Visited tracking, listener deduplication, object branding.

---

## References

- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – WeakSet Objects — https://tc39.es/ecma262/multipage/keyed-collections.html#sec-weakset-objects
- MDN Web Docs – WeakSet — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakSet
- MDN Web Docs – WeakSet.prototype.add() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakSet/add
- MDN Web Docs – WeakSet.prototype.has() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakSet/has
- MDN Web Docs – WeakSet.prototype.delete() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakSet/delete
- MDN Web Docs – Memory management — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Memory_management
- MDN Web Docs – Keyed collections — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Keyed_collections
- MDN Web Docs – WeakMap — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakMap
- V8 Blog – Ephemeron Garbage Collection — https://v8.dev/blog/high-performance-gc
- V8 Blog – WeakRefs and FinalizationRegistry — https://v8.dev/features/weak-references
- TC39 Proposal – WeakRefs — https://github.com/tc39/proposal-weakrefs