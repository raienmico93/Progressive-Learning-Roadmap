# JavaScript Collection Patterns — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Collection patterns are the established strategies, guidelines, and idioms for choosing, using, and combining JavaScript's built-in collection types — `Array`, `Object`, `Map`, `Set`, `WeakMap`, `WeakSet` — to solve data-management problems efficiently and idiomatically.

**Technical Definition:** JavaScript's collection types are specified in ECMA-262 across several sections: Array Exotic Objects (§10.4.2), Ordinary Objects (§10.1), Map Objects (§24.1), Set Objects (§24.2), WeakMap Objects (§24.3), and WeakSet Objects (§24.4). Each collection has distinct internal slots (`[[MapData]]`, `[[SetData]]`, `[[WeakMapData]]`, `[[WeakSetData]]`), distinct iteration protocols, distinct key/value semantics (SameValueZero for Maps/Sets, property keys for objects, index-based for arrays), and distinct performance characteristics. Collection patterns emerge from the interaction of these semantics with algorithmic requirements — membership testing, ordering, deduplication, keyed lookup, and lifecycle management.

**Beginner-Friendly Explanation:** JavaScript gives you several different "containers" for holding data: arrays (ordered lists), objects (named properties), Maps (flexible key-value stores), Sets (unique values), and their weak counterparts. Collection patterns are the rules of thumb for picking the right container for the job — and for using each one well.

### Key Characteristics

- **Multiple structures, overlapping capabilities:** Arrays, Objects, Maps, and Sets can all store multiple values; choosing well matters.
- **Different key semantics:** Arrays use integer indices; Objects use strings/Symbols; Maps use any value (SameValueZero); Sets store values as their own keys.
- **Different ordering guarantees:** Arrays are index-ordered; Objects have integer-key sorting; Maps and Sets preserve insertion order.
- **Different performance profiles:** O(1) vs O(n) for common operations.
- **Different memory footprints:** Arrays are compact; Maps/Sets have higher per-entry overhead; Weak variants add GC cooperation.
- **Composability:** Patterns often combine structures (e.g., Map of Sets, Array of Maps).

### Prerequisites

- Working knowledge of `Array`, `Object`, `Map`, `Set`, `WeakMap`, `WeakSet`.
- Understanding of Big-O notation and algorithmic complexity.
- Familiarity with iteration protocols (`for...of`, spread, destructuring).
- Basic awareness of garbage collection and reference semantics.

### Related Programming Areas

- **Data structures and algorithms:** Choosing the right structure for the problem.
- **Performance engineering:** Optimising hot paths.
- **Memory management:** Avoiding leaks, using weak collections.
- **Functional programming:** Immutable transformations and pipelines.
- **State management:** Application stores, caches, registries.

### Core Concepts / Features

The following sections cover each core concept, each using the uniform structure: Definitions, Purposes, Syntax Rules and Structure, Annotated Code Examples with Expected Outputs, and Real-World Cases.

---

## Choosing Collections

### Definitions

**Core Definition:** Choosing collections is the practice of selecting the most appropriate JavaScript collection type for a given data-management problem based on key types, ordering, uniqueness, performance, and lifecycle requirements.

**Technical Definition:** The decision matrix considers (a) whether keys are integers, strings, Symbols, or arbitrary values; (b) whether ordering is required; (c) whether duplicates are permitted; (d) whether O(1) membership testing is required; (e) whether the collection should hold weak references; and (f) whether the data must be JSON-serialised.

**Beginner-Friendly Explanation:** Picking the right collection is like picking the right container in a kitchen — a drawer for cutlery, a fridge for perishables, a jar for dry goods. Each container is designed for a specific kind of content.

### Purposes

- To match the data structure to the problem's access pattern.
- To optimise performance for hot paths.
- To avoid memory leaks through appropriate weak references.
- To keep code idiomatic and readable.

### Decision Matrix

| Requirement | Best Choice | Alternative |
|---|---|---|
| Ordered sequence, duplicates allowed | `Array` | TypedArray (numeric) |
| Fixed-shape record with named fields | `Object` (or `class`) | `Map` |
| Key-value with primitive keys, iteration needed | `Map` | `Object` |
| Key-value with object keys, iteration needed | `Map` | — |
| Key-value with object keys, no retention | `WeakMap` | — |
| Unique primitive values | `Set` | `Array` + manual dedup |
| Unique object values, no retention | `WeakSet` | — |
| Membership test, O(1) | `Set` / `Map` | — |
| JSON serialisation | `Array` / `Object` | Convert Map/Set first |
| Numeric-heavy data | TypedArray | `Array` |
| Priority-ordered access | Custom heap on `Array` | — |
| FIFO/LIFO queue | `Array` (`push`/`shift`, `push`/`pop`) | Linked list |
| Counts / histogram | `Map` | `Object` (numeric keys) |
| Graph adjacency | `Map<node, Set<node>>` | `Object` of Arrays |

### Syntax Rules and Structure

There is no dedicated syntax; choosing collections is a design decision. The relevant syntax is that of each collection.

**Constraints and Limitations:**
- No single collection is optimal for all operations.
- Conversions between collections have costs.
- Weak collections cannot be iterated, sized, or serialized.
- Objects are less safe as maps due to prototype pollution.

### Annotated Code Examples

#### Example 1: Choosing Between Object and Map

```js
// Scenario A: Fixed-shape user record (known fields).
const userRecord = { id: 1, name: "Alice", email: "alice@example.com" };
console.log(userRecord.name); // "Alice"

// Scenario B: Dynamic key-value store (unknown keys, arbitrary types).
const settings = new Map();
settings.set("theme", "dark");
settings.set(42, "answer");
settings.set({ scope: "admin" }, true);
console.log(settings.get(42)); // "answer"
console.log(settings.size);    // 3
```

**Expected Output:**
```
Alice
answer
3
```

**Why This Output:** The object is appropriate for a fixed-shape record. The Map handles dynamic, mixed-type keys with iteration and size support.

#### Example 2: Choosing Between Array and Set for Membership

```js
// Array: ordered, allows duplicates, O(n) membership.
const arr = [1, 2, 3, 4, 5];
console.log(arr.includes(3)); // true

// Set: unique, O(1) membership.
const set = new Set([1, 2, 3, 4, 5]);
console.log(set.has(3)); // true

// For large collections, Set is dramatically faster for membership.
```

**Expected Output:**
```
true
true
```

**Why This Output:** Both return `true`, but the Set's `has()` is O(1) versus the Array's `includes()` O(n).

### Real-World Cases

- **Form state:** Object for fixed fields; Map for dynamic field lists.
- **Tag systems:** Set for unique tags.
- **Entity caches:** Map keyed by ID.
- **DOM metadata:** WeakMap keyed by element.
- **Graph traversal:** Set for visited tracking.

---

## Performance Comparison

### Definitions

**Core Definition:** Performance comparison is the analysis of time complexity for common operations across JavaScript collection types.

**Technical Definition:** Average-case time complexity is generally O(1) for hash-based operations (Map/Set `set`/`add`/`get`/`has`/`delete`), O(1) for array `push`/`pop` and index access, O(n) for array `includes`/`indexOf`/`splice`, O(1) for object property access (with hidden-class optimisation), and O(n) for object `delete` in dictionary mode. V8's object property access benefits from inline caches; Map/Set operations are hash-based and predictable.

**Beginner-Friendly Explanation:** Some operations are fast no matter how much data you have (O(1)); others slow down as data grows (O(n)). Knowing which is which helps you avoid performance traps.

### Time Complexity Table

| Operation | Array | Object | Map | Set |
|---|---|---|---|---|
| Index/property access | O(1) | O(1) | — | — |
| `get`/lookup | — | O(1) | O(1) avg | — |
| `has`/`includes` | O(n) | O(1) | O(1) avg | O(1) avg |
| Insert | O(1)* | O(1) | O(1) avg | O(1) avg |
| Delete by value | O(n) | O(1)** | O(1) avg | O(1) avg |
| Delete by index | O(n) | — | — | — |
| Iterate | O(n) | O(n) | O(n) | O(n) |
| Size | O(1) | O(n)*** | O(1) | O(1) |

\* `push`/`pop` are amortised O(1); `unshift`/`splice` are O(n).
\** `delete obj[key]` deoptimises hidden classes in V8.
\*** `Object.keys(obj).length` is O(n).

### Annotated Code Examples

#### Example 1: Membership Benchmark

```js
const N = 100_000;
const arr = Array.from({ length: N }, (_, i) => i);
const set = new Set(arr);
const map = new Map(arr.map((i) => [i, i]));

const target = N - 1;
const ITER = 1000;

let start = performance.now();
for (let i = 0; i < ITER; i++) arr.includes(target);
console.log(`Array.includes x${ITER}: ${(performance.now() - start).toFixed(2)} ms`);

start = performance.now();
for (let i = 0; i < ITER; i++) set.has(target);
console.log(`Set.has x${ITER}: ${(performance.now() - start).toFixed(2)} ms`);

start = performance.now();
for (let i = 0; i < ITER; i++) map.has(target);
console.log(`Map.has x${ITER}: ${(performance.now() - start).toFixed(2)} ms`);
```

**Expected Output (approximate):**
```
Array.includes x1000: 42.10 ms
Set.has x1000: 0.25 ms
Map.has x1000: 0.30 ms
```

**Why This Output:** Array membership is O(n) — 100,000 × 1,000 = 100M comparisons. Set/Map membership is O(1) — 1,000 hash lookups.

#### Example 2: Insertion Benchmark

```js
const N = 1_000_000;

let start = performance.now();
const arr = [];
for (let i = 0; i < N; i++) arr.push(i);
console.log(`Array.push x${N}: ${(performance.now() - start).toFixed(2)} ms`);

start = performance.now();
const obj = {};
for (let i = 0; i < N; i++) obj[i] = i;
console.log(`Object set x${N}: ${(performance.now() - start).toFixed(2)} ms`);

start = performance.now();
const map = new Map();
for (let i = 0; i < N; i++) map.set(i, i);
console.log(`Map.set x${N}: ${(performance.now() - start).toFixed(2)} ms`);
```

**Expected Output (approximate):**
```
Array.push x1000000: 15.20 ms
Object set x1000000: 95.30 ms
Map.set x1000000: 120.45 ms
```

**Why This Output:** Array push is very fast (contiguous memory). Object insertion is slower due to hidden-class transitions. Map insertion is predictable but has hash overhead.

### Real-World Cases

- **Hot loops:** Use Set/Map for membership; avoid Array.includes.
- **Bulk insertion:** Array for numeric sequences; Map for keyed data.
- **Frequent deletion:** Map/Set over Object.
- **Small datasets:** Object or Array may be faster due to lower overhead.

---

## Memory Comparison

### Definitions

**Core Definition:** Memory comparison is the analysis of per-entry memory overhead across collection types.

**Technical Definition:** Arrays store elements in contiguous memory (for packed kinds), with roughly 8 bytes per element plus header. Objects store properties in hidden-class-backed in-object slots or a properties backing store, with per-property overhead. Maps and Sets use hash tables with entry records, typically 3–4 words per entry plus bucket overhead. WeakMap/WeakSet entries are ephemerons and are cleaned up by the GC. TypedArrays store raw bytes without per-element tagging.

**Beginner-Friendly Explanation:** Some containers are compact (arrays); others use more memory per item (Maps and Sets) because they need extra bookkeeping for fast lookup.

### Memory Footprint Table (Approximate)

| Collection | Per-Entry Overhead | Notes |
|---|---|---|
| TypedArray | 1–8 bytes | Raw numeric data |
| Array (packed) | ~8 bytes | Plus header, capacity slack |
| Object | ~16–32 bytes | Hidden class + property slots |
| Map | ~32–48 bytes | Hash entry + bucket overhead |
| Set | ~32–48 bytes | Similar to Map |
| WeakMap | ~32–48 bytes | GC cooperation overhead |
| WeakSet | ~32–48 bytes | GC cooperation overhead |

*Values are rough approximations; actual sizes depend on engine, version, and data shape.*

### Annotated Code Examples

#### Example 1: Measuring Approximate Memory (Node.js)

```js
// Run with: node --expose-gc memory-test.js
function measure(label, fn) {
  global.gc();
  const before = process.memoryUsage().heapUsed;
  const result = fn();
  global.gc();
  const after = process.memoryUsage().heapUsed;
  console.log(`${label}: ${((after - before) / 1024 / 1024).toFixed(2)} MB`);
  return result;
}

const N = 1_000_000;

measure("Array of numbers", () => Array.from({ length: N }, (_, i) => i));
measure("Set of numbers", () => new Set(Array.from({ length: N }, (_, i) => i)));
measure("Map of numbers", () => new Map(Array.from({ length: N }, (_, i) => [i, i])));
measure("Object of numbers", () => {
  const o = {};
  for (let i = 0; i < N; i++) o[i] = i;
  return o;
});
```

**Expected Output (approximate, Node.js):**
```
Array of numbers: 8.10 MB
Set of numbers: 32.40 MB
Map of numbers: 40.20 MB
Object of numbers: 24.60 MB
```

**Why This Output:** Arrays are the most compact for numeric data. Sets and Maps have higher per-entry overhead due to hash table structures. Objects fall in between.

### Real-World Cases

- **Large numeric datasets:** TypedArray or Array.
- **Unique IDs:** Set is worth the overhead for O(1) membership.
- **Object metadata:** WeakMap avoids leaks at the cost of per-entry overhead.
- **Memory-constrained environments:** Prefer Arrays and Objects over Maps/Sets when possible.

---

## Iteration Patterns

### Definitions

**Core Definition:** Iteration patterns are the idiomatic ways to traverse each collection type and to combine traversal with transformation, filtering, and aggregation.

**Technical Definition:** Arrays implement `Array.prototype[Symbol.iterator]` yielding values in index order. Objects are not iterable by default; `for...in` iterates enumerable string keys (including inherited), while `Object.keys/values/entries` return own enumerable properties. Maps implement `Symbol.iterator` yielding `[key, value]` pairs in insertion order; `keys()`, `values()`, `entries()` provide specific iterators. Sets implement `Symbol.iterator` yielding values in insertion order; `keys()`, `values()`, `entries()` are provided for Map-like symmetry. WeakMap/WeakSet are not iterable.

**Beginner-Friendly Explanation:** Each collection has its own way to be looped over. Arrays use index loops or `for...of`; objects need `Object.keys/values/entries`; Maps and Sets use `for...of` directly; weak collections can't be iterated at all.

### Purposes

- To traverse collection contents correctly and idiomatically.
- To transform or aggregate data.
- To avoid common pitfalls (e.g., `for...in` on objects including inherited properties).

### Syntax Rules and Structure

**Arrays:**
```js
for (let i = 0; i < arr.length; i++) { ... }
for (const value of arr) { ... }
arr.forEach((value, index, array) => { ... });
arr.map(fn).filter(fn).reduce(fn, init);
```

**Objects:**
```js
for (const key in obj) { ... }              // includes inherited
for (const key of Object.keys(obj)) { ... } // own enumerable
for (const [k, v] of Object.entries(obj)) { ... }
Object.values(obj).forEach((v) => { ... });
```

**Maps:**
```js
for (const [key, value] of map) { ... }
for (const key of map.keys()) { ... }
for (const value of map.values()) { ... }
map.forEach((value, key, map) => { ... });
```

**Sets:**
```js
for (const value of set) { ... }
set.forEach((value) => { ... });
[...set];
```

**Constraints and Limitations:**
- `for...in` includes inherited enumerable properties; use `Object.hasOwn()` to filter.
- Map/Set iterators are live; mutations during iteration are reflected.
- Weak collections cannot be iterated.
- `forEach` cannot be broken early.

### Annotated Code Examples

#### Example 1: Safe Object Iteration

```js
const proto = { inherited: true };
const obj = Object.create(proto);
obj.own1 = 1;
obj.own2 = 2;

// BAD: for...in includes inherited properties.
for (const key in obj) {
  console.log("for...in:", key);
}

// GOOD: Object.keys returns only own enumerable properties.
for (const key of Object.keys(obj)) {
  console.log("Object.keys:", key);
}

// GOOD: Object.hasOwn filter.
for (const key in obj) {
  if (Object.hasOwn(obj, key)) console.log("hasOwn:", key);
}
```

**Expected Output:**
```
for...in: own1
for...in: own2
for...in: inherited
Object.keys: own1
Object.keys: own2
hasOwn: own1
hasOwn: own2
```

**Why This Output:** `for...in` includes inherited properties; `Object.keys` and `Object.hasOwn` filter to own properties only.

#### Example 2: Transforming a Map into an Array

```js
const map = new Map([["a", 1], ["b", 2], ["c", 3]]);

// Step 1: Convert to array of pairs.
const pairs = [...map];
console.log(pairs); // [["a",1],["b",2],["c",3]]

// Step 2: Map values to doubled numbers.
const doubled = [...map].map(([k, v]) => [k, v * 2]);
console.log(doubled); // [["a",2],["b",4],["c",6]]

// Step 3: Back to a Map.
const map2 = new Map(doubled);
console.log(map2.get("b")); // 4
```

**Expected Output:**
```
[ [ 'a', 1 ], [ 'b', 2 ], [ 'c', 3 ] ]
[ [ 'a', 2 ], [ 'b', 4 ], [ 'c', 6 ] ]
4
```

**Why This Output:** Spreading converts the Map to an array of pairs, enabling array methods. The Map constructor accepts the transformed pairs.

### Real-World Cases

- **Object iteration:** Using `Object.entries` for destructuring.
- **Map transformation:** Converting Maps to arrays for functional pipelines.
- **Set deduplication:** `[...new Set(arr)]`.
- **Async iteration:** `for await...of` over async iterables.

---

## Collection Utilities

### Definitions

**Core Definition:** Collection utilities are reusable functions and patterns that operate uniformly across collection types, providing conversion, deduplication, grouping, and aggregation operations.

**Technical Definition:** Utilities leverage the iterable protocol, `Symbol.iterator`, `Object.entries`, and the `Map`/`Set` constructors to convert between collection types and apply transformations. They abstract over the differences between Arrays, Objects, Maps, and Sets so that higher-level code can work with any of them.

**Beginner-Friendly Explanation:** Collection utilities are helper functions that make it easy to convert between collections, group items, count occurrences, and perform other common tasks — without rewriting the same logic every time.

### Purposes

- To convert between collection types cleanly.
- To implement grouping, partitioning, and counting.
- To deduplicate data.
- To flatten and unflatten nested structures.
- To provide consistent iteration and transformation APIs.

### Syntax Rules and Structure

Utilities are ordinary functions; their signatures depend on the specific operation. Common patterns:

```js
// Convert
Array.from(iterable)
[...iterable]
Object.fromEntries(entries)
new Map(entries)
new Set(iterable)

// Group
Object.groupBy(iterable, callback)   // ES2024
Map.groupBy(iterable, callback)      // ES2024

// Deduplicate
[...new Set(arr)]

// Count
function countBy(iterable, keyFn) { ... }

// Chunk
function chunk(arr, size) { ... }
```

**Constraints and Limitations:**
- `Object.groupBy` and `Map.groupBy` are ES2024; check environment support.
- `Object.fromEntries` requires an iterable of pairs.
- Weak collections cannot be converted directly (no iteration).

### Annotated Code Examples

#### Example 1: Deduplication Utility

```js
// Step 1: Define a reusable deduplication utility.
function unique(iterable) {
  return [...new Set(iterable)];
}

// Step 2: Apply to arrays.
console.log(unique([1, 2, 2, 3, 3, 3])); // [1, 2, 3]

// Step 3: Apply to strings.
console.log(unique("hello")); // ["h", "e", "l", "o"]

// Step 4: Apply to Maps (iterates entries).
console.log(unique(new Map([["a", 1], ["a", 2]]))); // [["a", 2]]
```

**Expected Output:**
```
[ 1, 2, 3 ]
[ 'h', 'e', 'l', 'o' ]
[ [ 'a', 2 ] ]
```

**Why This Output:** `Set` removes duplicates by SameValueZero. Strings iterate by character. Maps iterate entries; distinct entry arrays are all unique.

#### Example 2: Grouping Utility (ES2024 `Object.groupBy`)

```js
const items = [
  { name: "apple", category: "fruit" },
  { name: "carrot", category: "vegetable" },
  { name: "banana", category: "fruit" },
  { name: "broccoli", category: "vegetable" },
];

const grouped = Object.groupBy(items, (item) => item.category);
console.log(grouped.fruit.length);     // 2
console.log(grouped.vegetable.length); // 2

// Map.groupBy returns a Map instead of an Object.
const groupedMap = Map.groupBy(items, (item) => item.category);
console.log(groupedMap.get("fruit").length); // 2
```

**Expected Output:**
```
2
2
2
```

**Why This Output:** `Object.groupBy` returns an object keyed by the callback's return values. `Map.groupBy` returns a Map, allowing any key type.

#### Example 3: Counting Utility

```js
function countBy(iterable, keyFn) {
  const counts = new Map();
  for (const item of iterable) {
    const key = keyFn(item);
    counts.set(key, (counts.get(key) || 0) + 1);
  }
  return counts;
}

const words = ["apple", "banana", "apple", "cherry", "banana", "apple"];
const counts = countBy(words, (w) => w);
console.log(counts.get("apple"));  // 3
console.log(counts.get("banana")); // 2
console.log(counts.get("cherry")); // 1
```

**Expected Output:**
```
3
2
1
```

**Why This Output:** The utility uses a Map to accumulate counts keyed by the callback result.

#### Example 4: Conversion Utilities

```js
// Array of pairs <-> Map
const entries = [["a", 1], ["b", 2]];
const map = new Map(entries);
const back = [...map];
console.log(back); // [["a", 1], ["b", 2]]

// Object <-> Map
const obj = { a: 1, b: 2 };
const map2 = new Map(Object.entries(obj));
const obj2 = Object.fromEntries(map2);
console.log(obj2); // { a: 1, b: 2 }

// Array <-> Set
const set = new Set([1, 2, 3]);
const arr = [...set];
console.log(arr); // [1, 2, 3]
```

**Expected Output:**
```
[ [ 'a', 1 ], [ 'b', 2 ] ]
{ a: 1, b: 2 }
[ 1, 2, 3 ]
```

**Why This Output:** Conversions leverage iterables and constructors. `Object.entries` and `Object.fromEntries` bridge Objects and Maps.

### Real-World Cases

- **Analytics:** `countBy` for event counting.
- **Data pipelines:** Conversion between Arrays, Maps, and Objects.
- **Categorisation:** `Object.groupBy` for dashboards.
- **Deduplication:** `unique` for tag lists, IDs, and search results.

---

## References

- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Indexed Collections — https://tc39.es/ecma262/multipage/indexed-collections.html
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Keyed Collections — https://tc39.es/ecma262/multipage/keyed-collections.html
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Managing Memory — https://tc39.es/ecma262/multipage/managing-memory.html
- MDN Web Docs – Keyed collections — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Keyed_collections
- MDN Web Docs – Indexed collections — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Indexed_collections
- MDN Web Docs – Iteration protocols — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols
- MDN Web Docs – Map — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map
- MDN Web Docs – Set — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set
- MDN Web Docs – Object.groupBy() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/groupBy
- MDN Web Docs – Map.groupBy() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map/groupBy
- MDN Web Docs – Object.fromEntries() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/fromEntries
- MDN Web Docs – Memory management — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Memory_management
- TC39 Proposal – Array Grouping — https://github.com/tc39/proposal-array-grouping
- V8 Blog – Fast Properties in V8 — https://v8.dev/blog/fast-properties
- V8 Blog – Elements Kinds in V8 — https://v8.dev/blog/elements-kinds
- V8 Blog – High-performance garbage collection for V8 — https://v8.dev/blog/high-performance-gc