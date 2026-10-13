# JavaScript `Map` — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A `Map` is a built-in JavaScript collection that stores key-value pairs, where keys may be of any type (including objects, functions, and primitives), and maintains insertion order.

**Technical Definition:** Per ECMAScript 2025 (ECMA-262, §24.1), `Map` is a constructor whose instances are exotic objects with a `[[MapData]]` internal slot containing a list of key-value records. Key equality is determined by the SameValueZero algorithm (which treats `NaN` as equal to itself and `+0` as equal to `-0`), not by string coercion or property lookup. `Map.prototype` provides the methods `set`, `get`, `has`, `delete`, `clear`, `forEach`, `keys`, `values`, and `entries`, plus the `size` accessor. `Map` is iterable via `Symbol.iterator`, which is aliased to `entries()`.

**Beginner-Friendly Explanation:** A `Map` is like a dictionary or a phone book. You store information as pairs: a *key* (the word you look up) and a *value* (the definition). Unlike a plain object, a `Map` lets you use anything as a key — numbers, strings, objects, even functions — and it remembers the order in which you added items.

### Key Characteristics

- **Any key type:** Keys can be primitives or object references.
- **Insertion-order iteration:** Entries iterate in the order they were inserted.
- **SameValueZero key equality:** `NaN` equals `NaN`; `+0` equals `-0`.
- **No prototype pollution risk:** `Map` keys are not converted to strings and do not interact with `Object.prototype`.
- **Direct size access:** The `size` property is a number, not a method.
- **Iterable:** `Map` is directly usable in `for...of`, spread, and destructuring.
- **Weak references available:** For garbage-collectable keys, use `WeakMap`.

### Prerequisites

- Basic JavaScript syntax (variables, functions, objects).
- Understanding of primitive types vs. reference types.
- Familiarity with arrays and iteration (`for...of`).
- Basic knowledge of `Object` as a key-value store.

### Related Programming Areas

- **Data structures and algorithms:** Hash maps, dictionaries, caches.
- **Functional programming:** Using `Map` with `map`, `filter`, `reduce`.
- **State management:** Storing application state keyed by entity IDs.
- **Caching and memoisation:** Using `Map` for LRU-style caches.
- **WeakMap and WeakSet:** For object-keyed, garbage-collectable collections.

### Core Concepts / Features

The following sections cover each core concept and feature of `Map`, each using the uniform structure: Definitions, Purposes, Syntax Rules and Structure, Annotated Code Examples with Expected Outputs, and Real-World Cases.

---

## `Map`

### Definitions

**Core Definition:** `Map` is the built-in constructor function used to create map instances.

**Technical Definition:** `Map` is a global intrinsic object whose `[[Prototype]]` is `Function.prototype`. Calling `Map()` without `new` throws a `TypeError`. When invoked with `new`, it creates a Map exotic object, optionally initialised from an iterable of key-value pairs.

**Beginner-Friendly Explanation:** `Map` is the "factory" you use to make new map objects. You write `new Map()` to create an empty map, or `new Map([[key, value], ...])` to create one with initial entries.

### Purposes

- To create empty or pre-populated map instances.
- To provide a namespace for `Map.prototype` methods.
- To enable `instanceof Map` checks.
- To serve as the base for subclassing (e.g., custom caches).

### Syntax Rules and Structure

**General Syntax:**
```js
new Map()
new Map(iterable)
```

**Component Breakdown:**
- `new`: Required operator; `Map()` without `new` throws `TypeError`.
- `iterable` (optional): An iterable object (usually an array) whose elements are two-element arrays `[key, value]`. Each element is destructured into a key and a value.

**Syntax Rules:**
- The constructor accepts at most one argument.
- The argument must be iterable (arrays, other maps, generators, strings of pairs).
- Each entry must be a two-element structure; otherwise, `TypeError`.
- `null` and `undefined` are treated as "no argument" (empty map).

**Constraints and Limitations:**
- Keys are compared by SameValueZero, not by value equality for objects. Two distinct objects with the same content are different keys.
- Iteration order is guaranteed to be insertion order (including for numeric keys, unlike objects).
- `Map` is not JSON-serializable by default (`JSON.stringify` returns `{}`).

### Annotated Code Examples

#### Example 1: Creating an Empty Map

```js
// Step 1: Create an empty Map.
const map = new Map();

// Step 2: Verify it is empty.
console.log(map.size);           // 0
console.log(map instanceof Map); // true
```

**Expected Output:**
```
0
true
```

**Why This Output:** `new Map()` creates an empty Map with `size` 0. The instance inherits from `Map.prototype`, so `instanceof Map` is true.

#### Example 2: Creating a Map from an Array of Pairs

```js
// Step 1: Provide an array of [key, value] pairs.
const entries = [
  ["name", "Alice"],
  ["age", 30],
  ["city", "Paris"],
];

// Step 2: Pass the array to the Map constructor.
const map = new Map(entries);

// Step 3: Inspect the resulting map.
console.log(map.size);           // 3
console.log(map.get("name"));    // "Alice"
console.log(map.get("age"));     // 30
```

**Expected Output:**
```
3
Alice
30
```

**Why This Output:** The constructor iterates the array, destructures each pair into key and value, and inserts them in order. `size` reflects the number of entries.

#### Example 3: Creating a Map from Another Map

```js
const original = new Map([["a", 1], ["b", 2]]);
const copy = new Map(original);

console.log(copy.size);       // 2
console.log(copy.get("a"));   // 1
console.log(copy === original); // false
```

**Expected Output:**
```
2
1
false
```

**Why This Output:** Passing an existing Map to the constructor iterates its entries and creates a new Map. The two Map objects are distinct references.

### Real-World Cases

- **Configuration stores:** Initialising a Map from a config array.
- **Copying caches:** Cloning an existing Map before mutation.
- **Data transformation:** Converting an array of records into a keyed lookup.

---

## Map Creation

### Definitions

**Core Definition:** Map creation is the process of instantiating a `Map` object, either empty or pre-populated.

**Technical Definition:** Instantiation is performed via the `Map` constructor with `new`. The internal `[[MapData]]` slot is initialised to an empty List; if an iterable argument is provided, the `addEntriesFromIterable` abstract operation inserts each entry, throwing `TypeError` if any entry is not a two-element object.

**Beginner-Friendly Explanation:** You create a Map by calling `new Map()`. You can start empty or pass in a list of pairs to fill it immediately.

### Purposes

- To obtain a mutable key-value store.
- To seed a map with known data.
- To convert arrays or objects into Map form.

### Syntax Rules and Structure

**Syntax Variants:**
```js
new Map()                          // Empty
new Map([[k1, v1], [k2, v2]])      // From array of pairs
new Map(otherMap)                  // Copy from another Map
new Map(Object.entries(obj))       // From plain object
new Map(generatorFunction())       // From any iterable of pairs
```

**Constraints and Limitations:**
- Non-iterable arguments throw `TypeError`.
- Iterables yielding non-pair entries throw `TypeError`.
- Duplicate keys: later entries overwrite earlier ones.

### Annotated Code Examples

#### Example 1: Converting an Object to a Map

```js
// Step 1: Define a plain object.
const obj = { a: 1, b: 2, c: 3 };

// Step 2: Convert to a Map using Object.entries().
const map = new Map(Object.entries(obj));

// Step 3: Inspect.
console.log(map.size);        // 3
console.log(map.get("b"));    // 2
```

**Expected Output:**
```
3
2
```

**Why This Output:** `Object.entries(obj)` returns `[["a",1],["b",2],["c",3]]`, which the Map constructor accepts.

#### Example 2: Handling Duplicate Keys

```js
// Step 1: Provide entries with a duplicate key.
const map = new Map([
  ["x", 1],
  ["y", 2],
  ["x", 99], // Overwrites the first "x"
]);

console.log(map.size);      // 2
console.log(map.get("x"));  // 99
```

**Expected Output:**
```
2
99
```

**Why This Output:** When a duplicate key is inserted, the value is updated but the insertion position is preserved (the key keeps its original position in iteration order).

### Real-World Cases

- **API response indexing:** Converting an array of records into a Map keyed by ID.
- **Environment configuration:** Merging default and override settings into a Map.
- **Memoisation tables:** Seeding a Map with precomputed values.

---

## Map Methods

### Definitions

**Core Definition:** Map methods are the functions on `Map.prototype` that allow reading, writing, and querying map entries.

**Technical Definition:** The `Map.prototype` object defines `set`, `get`, `has`, `delete`, `clear`, `forEach`, `keys`, `values`, and `entries`, plus the `size` accessor and `Symbol.iterator`. Each method operates on the `[[MapData]]` internal slot of the receiver.

**Beginner-Friendly Explanation:** Map methods are the operations you perform on a Map — adding items, looking them up, checking if they exist, removing them, and iterating over them.

---

### `set()`

#### Definitions

- **Core Definition:** `set()` adds or updates a key-value pair in the Map.
- **Technical Definition:** `Map.prototype.set(key, value)` normalises `-0` to `+0`, searches `[[MapData]]` for an entry whose key is SameValueZero-equal to `key`, updates the value if found, or appends a new record. Returns the Map for chaining.
- **Beginner-Friendly Explanation:** `set()` is how you put something into a Map.

#### Purposes

- To insert a new entry.
- To update the value of an existing key.
- To enable method chaining.

#### Syntax Rules and Structure

```js
map.set(key, value)
```

**Component Breakdown:**
- `key`: Any JavaScript value.
- `value`: Any JavaScript value.
- Returns: The same Map object (for chaining).

**Constraints and Limitations:**
- `-0` is normalised to `+0`.
- Object keys are compared by reference, not structure.
- Inserting a duplicate key updates the value but preserves insertion position.

#### Annotated Code Examples

```js
const map = new Map();

// Step 1: Insert entries with chaining.
map.set("a", 1).set("b", 2).set("c", 3);

// Step 2: Update an existing key.
map.set("a", 100);

// Step 3: Use an object as a key.
const objKey = { id: 1 };
map.set(objKey, "object value");

console.log(map.get("a"));       // 100
console.log(map.get(objKey));    // "object value"
console.log(map.size);           // 4
```

**Expected Output:**
```
100
object value
4
```

**Why This Output:** Chaining works because `set()` returns the Map. Updating `"a"` overwrites the value. The object key is stored by reference.

#### Real-World Cases

- **Caches:** `cache.set(url, response)`.
- **Counters:** `counts.set(word, (counts.get(word) || 0) + 1)`.
- **Graph adjacency:** `graph.set(node, [neighbours])`.

---

### `get()`

#### Definitions

- **Core Definition:** `get()` retrieves the value associated with a key.
- **Technical Definition:** `Map.prototype.get(key)` searches `[[MapData]]` for an entry whose key is SameValueZero-equal to `key` and returns its value, or `undefined` if not found.
- **Beginner-Friendly Explanation:** `get()` is how you look up a value in a Map.

#### Purposes

- To retrieve stored values.
- To check for presence indirectly (though `has()` is preferred).
- To read counters, config values, cached data.

#### Syntax Rules and Structure

```js
map.get(key)
```

**Returns:** The associated value, or `undefined`.

**Constraints and Limitations:**
- Cannot distinguish between "key absent" and "key present with value `undefined`". Use `has()` for that.
- `NaN` as a key works because SameValueZero treats `NaN` as equal to itself.

#### Annotated Code Examples

```js
const map = new Map([["x", 1], ["y", undefined]]);

console.log(map.get("x"));     // 1
console.log(map.get("y"));     // undefined (present, but undefined)
console.log(map.get("z"));     // undefined (absent)
console.log(map.has("y"));     // true
console.log(map.has("z"));     // false

// NaN as key
map.set(NaN, "not a number");
console.log(map.get(NaN));     // "not a number"
```

**Expected Output:**
```
1
undefined
undefined
true
false
not a number
```

**Why This Output:** `get("y")` returns `undefined` because that's the stored value; `get("z")` returns `undefined` because the key is absent. `has()` disambiguates.

#### Real-World Cases

- **Lookup tables:** `translations.get(locale)`.
- **Caches:** `cache.get(key)` before fetching.
- **Metadata:** `userRoles.get(userId)`.

---

### `has()`

#### Definitions

- **Core Definition:** `has()` tests whether a key exists in the Map.
- **Technical Definition:** `Map.prototype.has(key)` returns `true` if an entry with a SameValueZero-equal key exists in `[[MapData]]`, else `false`.
- **Beginner-Friendly Explanation:** `has()` answers "Is this key in the Map?"

#### Purposes

- To check for key existence without retrieving the value.
- To distinguish "absent" from "present with `undefined` value".
- To guard against overwriting existing entries.

#### Syntax Rules and Structure

```js
map.has(key)
```

**Returns:** Boolean.

**Constraints and Limitations:**
- Uses SameValueZero, so `NaN` matches `NaN`, and `+0` matches `-0`.
- Object keys must be the same reference.

#### Annotated Code Examples

```js
const map = new Map([["name", "Alice"], ["age", undefined]]);

console.log(map.has("name"));   // true
console.log(map.has("age"));    // true (present, value undefined)
console.log(map.has("email"));  // false

// Object keys
const k1 = { id: 1 };
const k2 = { id: 1 };
map.set(k1, "one");
console.log(map.has(k1));       // true
console.log(map.has(k2));       // false (different reference)
```

**Expected Output:**
```
true
true
false
true
false
```

**Why This Output:** `has()` checks key presence using SameValueZero. Object keys are compared by reference, so structurally equal objects are distinct keys.

#### Real-World Cases

- **Deduplication:** `if (!seen.has(item)) { ... }`.
- **Cache validation:** Checking if a key is cached before fetching.
- **Permission checks:** `permissions.has("admin")`.

---

### `delete()`

#### Definitions

- **Core Definition:** `delete()` removes a key-value pair from the Map.
- **Technical Definition:** `Map.prototype.delete(key)` finds an entry with a SameValueZero-equal key, removes it from `[[MapData]]`, and returns `true`. Returns `false` if no such entry exists.
- **Beginner-Friendly Explanation:** `delete()` removes an item from the Map.

#### Purposes

- To remove entries that are no longer needed.
- To implement cache eviction.
- To manage lifecycle of keyed resources.

#### Syntax Rules and Structure

```js
map.delete(key)
```

**Returns:** `true` if an entry was removed, `false` otherwise.

**Constraints and Limitations:**
- Removing a non-existent key is a no-op returning `false`.
- Object keys must match by reference.

#### Annotated Code Examples

```js
const map = new Map([["a", 1], ["b", 2], ["c", 3]]);

console.log(map.delete("b"));  // true
console.log(map.size);         // 2
console.log(map.delete("z"));  // false
console.log(map.has("b"));     // false
```

**Expected Output:**
```
true
2
false
false
```

**Why This Output:** `delete("b")` removes the entry and returns `true`. `delete("z")` finds nothing and returns `false`.

#### Real-World Cases

- **Cache eviction:** `cache.delete(oldestKey)`.
- **Session cleanup:** `sessions.delete(sessionId)`.
- **Dynamic configuration:** Removing overrides.

---

### `clear()`

#### Definitions

- **Core Definition:** `clear()` removes all entries from the Map.
- **Technical Definition:** `Map.prototype.clear()` sets the `[[MapData]]` list to empty. Returns `undefined`.
- **Beginner-Friendly Explanation:** `clear()` empties the Map completely.

#### Purposes

- To reset a Map without creating a new instance.
- To release references to all keys and values.
- To reinitialise a cache or store.

#### Syntax Rules and Structure

```js
map.clear()
```

**Returns:** `undefined`.

**Constraints and Limitations:**
- Irreversible.
- Does not affect other references to the same Map.

#### Annotated Code Examples

```js
const map = new Map([["a", 1], ["b", 2]]);
console.log(map.size); // 2

map.clear();
console.log(map.size); // 0
console.log(map.has("a")); // false
```

**Expected Output:**
```
2
0
false
```

**Why This Output:** `clear()` empties `[[MapData]]`, so `size` becomes 0 and all keys are gone.

#### Real-World Cases

- **Logout:** Clearing user session data.
- **Cache reset:** Emptying a memoisation table.
- **Test teardown:** Resetting shared state.

---

### `size`

#### Definitions

- **Core Definition:** `size` is a read-only accessor returning the number of entries in the Map.
- **Technical Definition:** `get Map.prototype.size` returns the length of the `[[MapData]]` list.
- **Beginner-Friendly Explanation:** `size` tells you how many items are in the Map.

#### Purposes

- To check emptiness (`size === 0`).
- To iterate by count.
- To report collection statistics.

#### Syntax Rules and Structure

```js
map.size
```

**Returns:** A non-negative integer.

**Constraints and Limitations:**
- Read-only; assignment throws in strict mode.
- Unlike arrays, `size` is a property, not a method (`map.size`, not `map.size()`).

#### Annotated Code Examples

```js
const map = new Map();
console.log(map.size);      // 0

map.set("a", 1).set("b", 2);
console.log(map.size);      // 2

map.delete("a");
console.log(map.size);      // 1
```

**Expected Output:**
```
0
2
1
```

**Why This Output:** `size` reflects the current number of entries after each operation.

#### Real-World Cases

- **Cache metrics:** Reporting cache size to monitoring.
- **Guard clauses:** `if (map.size === 0) return;`.
- **Progress tracking:** `completed.size / total`.

---

### `keys()`

#### Definitions

- **Core Definition:** `keys()` returns an iterator over the Map's keys in insertion order.
- **Technical Definition:** `Map.prototype.keys()` returns a new Map Iterator object whose `next()` yields each key in `[[MapData]]` order.
- **Beginner-Friendly Explanation:** `keys()` gives you a list-like iterator of all the keys in the Map.

#### Purposes

- To iterate over keys only.
- To convert keys to an array (`[...map.keys()]`).
- To feed keys into other operations.

#### Syntax Rules and Structure

```js
map.keys()
```

**Returns:** A `MapIterator` object.

**Constraints and Limitations:**
- The iterator is live: mutations during iteration are reflected.
- Single-use: once exhausted, it does not reset.

#### Annotated Code Examples

```js
const map = new Map([["a", 1], ["b", 2], ["c", 3]]);

for (const key of map.keys()) {
  console.log(key);
}

const keyArray = [...map.keys()];
console.log(keyArray); // ["a", "b", "c"]
```

**Expected Output:**
```
a
b
c
["a", "b", "c"]
```

**Why This Output:** `keys()` yields each key in insertion order; spreading collects them into an array.

#### Real-World Cases

- **Enumerating IDs:** Iterating over entity IDs.
- **Schema introspection:** Listing all configured keys.
- **Bulk operations:** Applying a function to all keys.

---

### `values()`

#### Definitions

- **Core Definition:** `values()` returns an iterator over the Map's values in insertion order.
- **Technical Definition:** `Map.prototype.values()` returns a Map Iterator yielding each value in `[[MapData]]` order.
- **Beginner-Friendly Explanation:** `values()` gives you an iterator of all the values in the Map.

#### Purposes

- To iterate over values only.
- To convert values to an array.
- To aggregate or reduce values.

#### Syntax Rules and Structure

```js
map.values()
```

**Returns:** A `MapIterator` object.

#### Annotated Code Examples

```js
const map = new Map([["a", 10], ["b", 20], ["c", 30]]);

let sum = 0;
for (const value of map.values()) {
  sum += value;
}
console.log(sum); // 60

console.log([...map.values()]); // [10, 20, 30]
```

**Expected Output:**
```
60
[10, 20, 30]
```

**Why This Output:** `values()` yields each value; the loop sums them; spreading collects them.

#### Real-World Cases

- **Aggregation:** Summing scores, averaging metrics.
- **Serialisation:** Converting values to JSON arrays.
- **Reporting:** Listing all cached responses.

---

### `entries()`

#### Definitions

- **Core Definition:** `entries()` returns an iterator over `[key, value]` pairs in insertion order.
- **Technical Definition:** `Map.prototype.entries()` returns a Map Iterator yielding two-element arrays. It is also the default iterator via `Map.prototype[Symbol.iterator]`.
- **Beginner-Friendly Explanation:** `entries()` gives you both keys and values together as pairs.

#### Purposes

- To iterate over key-value pairs.
- To destructure in `for...of`.
- To convert a Map to an array of pairs.

#### Syntax Rules and Structure

```js
map.entries()
map[Symbol.iterator]() // same as entries()
```

**Returns:** A `MapIterator` yielding `[key, value]` arrays.

#### Annotated Code Examples

```js
const map = new Map([["a", 1], ["b", 2]]);

// Step 1: Destructure in for...of.
for (const [key, value] of map.entries()) {
  console.log(`${key} = ${value}`);
}

// Step 2: Same as iterating the Map directly.
for (const [key, value] of map) {
  console.log(`${key}:${value}`);
}
```

**Expected Output:**
```
a = 1
b = 2
a:1
b:2
```

**Why This Output:** `entries()` and `Symbol.iterator` yield the same pairs. Destructuring extracts key and value.

#### Real-World Cases

- **Serialisation:** `JSON.stringify([...map.entries()])`.
- **Transformation:** Mapping pairs to new structures.
- **Debugging:** Printing all entries.

---

### `forEach()`

#### Definitions

- **Core Definition:** `forEach()` executes a callback for each key-value pair in insertion order.
- **Technical Definition:** `Map.prototype.forEach(callback, thisArg)` invokes `callback(value, key, map)` for each entry. `thisArg` sets `this` inside the callback.
- **Beginner-Friendly Explanation:** `forEach()` runs a function on every item in the Map.

#### Purposes

- To perform side effects for each entry.
- To accumulate results into external variables.
- To integrate with code that uses callback style.

#### Syntax Rules and Structure

```js
map.forEach((value, key, map) => { ... }, thisArg)
```

**Component Breakdown:**
- `value`: The entry's value.
- `key`: The entry's key.
- `map`: The Map being iterated.
- `thisArg` (optional): Value of `this` inside the callback.

**Constraints and Limitations:**
- Cannot be broken early (no `break`); use `for...of` for early exit.
- Not chainable (returns `undefined`).
- Order is insertion order.

#### Annotated Code Examples

```js
const map = new Map([["a", 1], ["b", 2], ["c", 3]]);

// Step 1: Log each pair.
map.forEach((value, key) => {
  console.log(`${key}: ${value}`);
});

// Step 2: Accumulate.
let total = 0;
map.forEach((value) => { total += value; });
console.log("Total:", total);
```

**Expected Output:**
```
a: 1
b: 2
c: 3
Total: 6
```

**Why This Output:** `forEach` visits each entry in order; the callback receives value first, key second (unlike arrays).

#### Real-World Cases

- **Logging:** Iterating over config entries.
- **Aggregation:** Summing values.
- **Event dispatch:** Calling handlers registered per key.

---

## Map Iteration

### Definitions

**Core Definition:** Map iteration is the process of traversing a Map's entries, keys, or values in insertion order.

**Technical Definition:** Maps implement the iterable protocol via `Map.prototype[Symbol.iterator]`, which is the same function object as `Map.prototype.entries`. Iteration is insertion-ordered and live (mutations during iteration are observable).

**Beginner-Friendly Explanation:** You can loop over a Map using `for...of`, spread it into an array, or use `forEach`. The order is always the order you added items.

### Purposes

- To process all entries, keys, or values.
- To convert Maps to arrays or other structures.
- To implement algorithms that depend on iteration order.

### Syntax Rules and Structure

```js
// for...of (entries)
for (const [key, value] of map) { ... }

// for...of (keys)
for (const key of map.keys()) { ... }

// for...of (values)
for (const value of map.values()) { ... }

// Spread
[...map]              // array of [key, value] pairs
[...map.keys()]       // array of keys
[...map.values()]     // array of values

// Destructuring
const [[k1, v1], [k2, v2]] = map;
```

**Constraints and Limitations:**
- The iterator is live; adding entries during iteration may lead to infinite loops if not careful.
- Deleting entries during iteration is allowed but the deleted entry is skipped.

### Annotated Code Examples

#### Example 1: Insertion Order Guarantee

```js
const map = new Map();
map.set(3, "three");
map.set(1, "one");
map.set(2, "two");

for (const [k, v] of map) {
  console.log(k, v);
}
```

**Expected Output:**
```
3 three
1 one
2 two
```

**Why This Output:** Maps preserve insertion order even for numeric keys, unlike objects (which order integer-like keys ascending).

#### Example 2: Converting to Array and Back

```js
const map = new Map([["a", 1], ["b", 2]]);

// To array of pairs
const arr = [...map];
console.log(arr); // [["a",1],["b",2]]

// Back to Map
const map2 = new Map(arr);
console.log(map2.get("b")); // 2
```

**Expected Output:**
```
[ [ 'a', 1 ], [ 'b', 2 ] ]
2
```

**Why This Output:** Spread collects entries as arrays; the Map constructor accepts them back.

### Real-World Cases

- **Ordered configuration:** Preserving user-defined order.
- **Serialisation round-trips:** Map → array → JSON → array → Map.
- **Algorithm implementation:** BFS/DFS using Map for visited sets.

---

## Map vs Object

### Definitions

**Core Definition:** `Map` and plain `Object` are both key-value stores, but they differ in key types, iteration, size access, and prototype behaviour.

**Technical Definition:** Objects use property keys (strings and symbols only); Maps use arbitrary values as keys via the `[[MapData]]` slot. Objects inherit from `Object.prototype` (unless created with `Object.create(null)`); Maps do not expose keys as properties. Objects have no direct `size` property (must use `Object.keys(obj).length`); Maps have `size`.

**Beginner-Friendly Explanation:** Both objects and Maps store key-value pairs, but Maps are more flexible (any key type), safer (no prototype pollution), and easier to iterate.

### Comparison Table

| Feature | Map | Object |
|---|---|---|
| Key types | Any value | String or Symbol |
| Key equality | SameValueZero | String coercion / reference for symbols |
| Insertion order | Guaranteed | Guaranteed for strings; integer keys sorted ascending |
| Size | `map.size` | `Object.keys(obj).length` |
| Iteration | Directly iterable | `for...in` (includes inherited); `Object.entries()` |
| Prototype pollution | No | Yes (unless `Object.create(null)`) |
| JSON serialisation | Not direct | Native |
| Performance | Better for frequent add/delete | Better for static, small data |
| Methods | `set`, `get`, `has`, `delete`, `clear` | Property access, `delete`, `Object.*` |

### Purposes

- To choose the right structure for the data at hand.
- To avoid prototype pollution and key coercion bugs.
- To optimise performance for frequent additions and deletions.

### Syntax Rules and Structure

```js
// Map
const map = new Map();
map.set("key", "value");
map.get("key");

// Object
const obj = {};
obj.key = "value";
obj["key"];
```

### Annotated Code Examples

#### Example 1: Object Key Coercion vs Map

```js
// Object: all keys become strings.
const obj = {};
obj[1] = "one";
obj["1"] = "ONE";
console.log(obj[1]); // "ONE" (same key)

// Map: 1 and "1" are distinct.
const map = new Map();
map.set(1, "one");
map.set("1", "ONE");
console.log(map.get(1));   // "one"
console.log(map.get("1")); // "ONE"
```

**Expected Output:**
```
ONE
one
ONE
```

**Why This Output:** Object property keys are coerced to strings, so `1` and `"1"` collide. Maps use SameValueZero, so number `1` and string `"1"` are distinct keys.

#### Example 2: Prototype Pollution

```js
// Object: inherited properties can be accidentally accessed.
const obj = {};
console.log(obj.toString); // [Function: toString]

// Map: no inherited key collisions.
const map = new Map();
console.log(map.get("toString")); // undefined
```

**Expected Output:**
```
[Function: toString]
undefined
```

**Why This Output:** Objects inherit enumerable and non-enumerable properties from `Object.prototype`; Maps store only explicitly set entries.

### Real-World Cases

- **User data:** Use Map for user IDs (numbers) to avoid string coercion.
- **Configuration:** Use Object for static, JSON-serialisable config.
- **Caches:** Use Map for high-frequency add/delete.

---

## Map Use Cases

### Definitions

**Core Definition:** Map use cases are scenarios where Map's properties make it the superior choice over objects or arrays.

**Technical Definition:** Maps excel in scenarios requiring non-string keys, frequent insertion/deletion, guaranteed iteration order, and safe key namespaces. They are less suitable for JSON-serialised config or cases needing prototype methods.

**Beginner-Friendly Explanation:** Use a Map when you need a flexible, ordered, and safe key-value store — especially when your keys aren't strings.

### Purposes

- To provide efficient keyed access with arbitrary keys.
- To maintain insertion order for ordered data.
- To avoid prototype pollution and key coercion.
- To implement caches, graphs, and frequency counters.

### Common Use Cases

1. **Caching / Memoisation**
   ```js
   const cache = new Map();
   function expensive(n) {
     if (cache.has(n)) return cache.get(n);
     const result = n * n;
     cache.set(n, result);
     return result;
   }
   ```

2. **Frequency Counting**
   ```js
   const counts = new Map();
   for (const word of words) {
     counts.set(word, (counts.get(word) || 0) + 1);
   }
   ```

3. **Graph Adjacency Lists**
   ```js
   const graph = new Map();
   graph.set("A", ["B", "C"]);
   graph.set("B", ["A"]);
   ```

4. **Object Metadata**
   ```js
   const metadata = new Map();
   metadata.set(domNode, { clicks: 0 });
   ```

5. **Ordered Deduplication**
   ```js
   const unique = [...new Map(items.map((i) => [i.id, i])).values()];
   ```

6. **Two-Way Lookups**
   ```js
   const idToName = new Map([[1, "Alice"], [2, "Bob"]]);
   const nameToId = new Map([...idToName].map(([id, name]) => [name, id]));
   ```

### Annotated Code Examples

#### Example 1: Memoisation with Map

```js
// Step 1: Create a cache.
const cache = new Map();

// Step 2: Define an expensive function with memoisation.
function fib(n) {
  if (n < 2) return n;
  if (cache.has(n)) return cache.get(n);
  const result = fib(n - 1) + fib(n - 2);
  cache.set(n, result);
  return result;
}

console.log(fib(40)); // 102334155
console.log(cache.size); // Number of cached results
```

**Expected Output:**
```
102334155
39
```

**Why This Output:** Memoisation avoids recomputation, dramatically speeding up recursive Fibonacci. The cache stores 39 intermediate values.

#### Example 2: Object Metadata Store

```js
const elementData = new Map();

function trackClick(element) {
  if (!elementData.has(element)) {
    elementData.set(element, { clicks: 0 });
  }
  elementData.get(element).clicks++;
}

const btn = document.createElement("button");
trackClick(btn);
trackClick(btn);
console.log(elementData.get(btn).clicks); // 2
```

**Expected Output:**
```
2
```

**Why This Output:** The DOM element is used as an object key, associating metadata with the element without modifying it. (`WeakMap` would be preferable for garbage collection.)

### Real-World Cases

- **React internals:** Uses Maps for fiber node metadata.
- **Vue reactivity:** Tracks dependencies with Maps and WeakMaps.
- **GraphQL clients:** Normalised caches keyed by entity ID.
- **Node.js:** Uses Maps for module caches and event listener registries.

---

## Map Performance

### Definitions

**Core Definition:** Map performance refers to the time and space complexity characteristics of Map operations relative to alternative structures.

**Technical Definition:** In V8 and other engines, `Map` is implemented as a hash table with open addressing (or a variant). Average-case time complexity for `set`, `get`, `has`, and `delete` is O(1). Iteration is O(n). Objects in V8 use hidden classes and inline caches, which can outperform Maps for small, static, string-keyed data but degrade with frequent additions/deletions.

**Beginner-Friendly Explanation:** Maps are generally fast for adding, looking up, and removing items — even with lots of data. Objects can be faster for small, fixed sets of string keys, but Maps win when keys are non-strings or when items are added and removed frequently.

### Purposes

- To choose the right data structure for performance-critical code.
- To understand when Maps outperform objects and vice versa.
- To avoid performance pitfalls (e.g., deleting many object properties).

### Performance Characteristics

| Operation | Map | Object |
|---|---|---|
| Insert (new key) | O(1) amortised | O(1) amortised (with hidden class transition) |
| Lookup | O(1) average | O(1) average (with inline cache) |
| Delete | O(1) average | O(1) but deoptimises hidden class |
| Iterate | O(n) | O(n) (with `Object.entries`) |
| Size | O(1) | O(n) (with `Object.keys`) |
| Memory | Higher per-entry overhead | Lower for small objects |

### Syntax Rules and Structure

No special syntax; performance is about choosing the right operations and avoiding anti-patterns.

**Constraints and Limitations:**
- Deleting object properties causes hidden-class transitions and slows subsequent access.
- Map entries are not properties, so they don't benefit from V8 inline caches.
- Very large Maps may cause rehashing pauses (amortised O(1)).

### Annotated Code Examples

#### Example 1: Benchmarking Map vs Object Insert

```js
// Step 1: Benchmark Map insertion.
const N = 1_000_000;
let start = performance.now();
const map = new Map();
for (let i = 0; i < N; i++) map.set(i, i);
let mapTime = performance.now() - start;

// Step 2: Benchmark Object insertion.
start = performance.now();
const obj = {};
for (let i = 0; i < N; i++) obj[i] = i;
let objTime = performance.now() - start;

console.log(`Map: ${mapTime.toFixed(2)} ms`);
console.log(`Object: ${objTime.toFixed(2)} ms`);
```

**Expected Output (approximate, engine-dependent):**
```
Map: 120.45 ms
Object: 95.32 ms
```

**Why This Output:** Objects can be faster for pure numeric-keyed insertion due to hidden classes, but Maps are competitive and more predictable. Actual results vary by engine and workload.

#### Example 2: Frequent Deletion

```js
const N = 100_000;

// Map: delete entries in a loop.
const map = new Map();
for (let i = 0; i < N; i++) map.set(i, i);
let start = performance.now();
for (let i = 0; i < N; i++) map.delete(i);
console.log(`Map delete: ${(performance.now() - start).toFixed(2)} ms`);

// Object: delete properties in a loop.
const obj = {};
for (let i = 0; i < N; i++) obj[i] = i;
start = performance.now();
for (let i = 0; i < N; i++) delete obj[i];
console.log(`Object delete: ${(performance.now() - start).toFixed(2)} ms`);
```

**Expected Output (approximate):**
```
Map delete: 15.20 ms
Object delete: 85.40 ms
```

**Why This Output:** Map deletion is O(1) and does not deoptimise hidden classes. Object deletion forces V8 to transition to dictionary mode, which is slower.

### Real-World Cases

- **High-frequency caches:** Use Map to avoid hidden-class deoptimisation.
- **Small config objects:** Use Object for readability and JSON serialisation.
- **Large datasets:** Use Map for predictable performance.
- **Non-string keys:** Map is the only choice (or WeakMap for objects).

---

## References

- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Map Objects — https://tc39.es/ecma262/multipage/keyed-collections.html#sec-map-objects
- MDN Web Docs – Map — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map
- MDN Web Docs – Map.prototype.set() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map/set
- MDN Web Docs – Map.prototype.get() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map/get
- MDN Web Docs – Map.prototype.has() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map/has
- MDN Web Docs – Map.prototype.delete() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map/delete
- MDN Web Docs – Map.prototype.clear() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map/clear
- MDN Web Docs – Map.prototype.size — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map/size
- MDN Web Docs – Map.prototype.keys() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map/keys
- MDN Web Docs – Map.prototype.values() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map/values
- MDN Web Docs – Map.prototype.entries() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map/entries
- MDN Web Docs – Map.prototype.forEach() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map/forEach
- MDN Web Docs – Keyed collections — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Keyed_collections
- V8 Blog – Fast Properties in V8 — https://v8.dev/blog/fast-properties
- V8 Blog – Elements Kinds in V8 — https://v8.dev/blog/elements-kinds