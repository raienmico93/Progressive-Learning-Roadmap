# JavaScript `Set` — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A `Set` is a built-in JavaScript collection that stores unique values of any type, maintaining insertion order and providing efficient membership testing.

**Technical Definition:** Per ECMAScript 2025 (ECMA-262, §24.2), `Set` is a constructor whose instances are exotic objects with a `[[SetData]]` internal slot containing a List of unique values. Uniqueness is determined by the SameValueZero algorithm (so `NaN` equals `NaN`, and `+0` equals `-0`). `Set.prototype` provides `add`, `has`, `delete`, `clear`, `forEach`, `keys`, `values`, and `entries`, plus the `size` accessor. `Set` is iterable via `Symbol.iterator`, which is aliased to `values()`. ES2025 introduces native set operations: `union`, `intersection`, `difference`, `symmetricDifference`, `isSubsetOf`, `isSupersetOf`, and `isDisjointFrom`.

**Beginner-Friendly Explanation:** A `Set` is like a bag that only holds each item once. If you try to add a duplicate, it's simply ignored. Sets are great for removing duplicates, checking membership quickly, and performing mathematical set operations like unions and intersections.

### Key Characteristics

- **Uniqueness:** Duplicate values are automatically ignored.
- **Any value type:** Primitives and object references are accepted.
- **Insertion-order iteration:** Values iterate in the order they were added.
- **SameValueZero equality:** `NaN` is considered equal to `NaN`; `+0` equals `-0`.
- **Direct size access:** The `size` property is a number, not a method.
- **Iterable:** Usable in `for...of`, spread, and destructuring.
- **Native set operations (ES2025):** Union, intersection, difference, symmetric difference, subset/superset/disjoint checks.
- **Weak variant:** `WeakSet` for garbage-collectable object membership.

### Prerequisites

- Basic JavaScript syntax and data types.
- Understanding of arrays and iteration.
- Familiarity with equality (`===`, `Object.is`) and reference semantics.
- Basic knowledge of `Map` (for parallel structure).

### Related Programming Areas

- **Data structures and algorithms:** Hash sets, membership testing.
- **Functional programming:** Deduplication, set algebra.
- **Graph algorithms:** Visited sets, adjacency tracking.
- **Permissions and roles:** Fast membership checks.
- **Text processing:** Unique word extraction, tag systems.

### Core Concepts / Features

The following sections cover each core concept and feature of `Set`, each using the uniform structure: Definitions, Purposes, Syntax Rules and Structure, Annotated Code Examples with Expected Outputs, and Real-World Cases.

---

## `Set`

### Definitions

**Core Definition:** `Set` is the built-in constructor function used to create set instances.

**Technical Definition:** `Set` is a global intrinsic object whose `[[Prototype]]` is `Function.prototype`. Calling `Set()` without `new` throws a `TypeError`. When invoked with `new`, it creates a Set exotic object, optionally initialised from an iterable of values. Internally, values are stored in a List within the `[[SetData]]` slot, with a hash-based index for O(1) average membership testing.

**Beginner-Friendly Explanation:** `Set` is the "factory" you use to make new set objects. Write `new Set()` to create an empty set, or `new Set([1, 2, 2, 3])` to create one from an array (duplicates removed automatically).

### Purposes

- To create empty or pre-populated set instances.
- To provide a namespace for `Set.prototype` methods.
- To enable `instanceof Set` checks.
- To serve as the base for subclassing.

### Syntax Rules and Structure

**General Syntax:**
```js
new Set()
new Set(iterable)
```

**Component Breakdown:**
- `new`: Required operator; `Set()` without `new` throws `TypeError`.
- `iterable` (optional): Any iterable object (array, string, another Set, generator). Each yielded value is added to the set. Duplicates are ignored.

**Syntax Rules:**
- The constructor accepts at most one argument.
- The argument must be iterable if provided.
- `null` and `undefined` are treated as "no argument" (empty set).
- Strings are iterated character by character.

**Constraints and Limitations:**
- Object values are compared by reference; two structurally identical objects are distinct.
- `Set` is not JSON-serializable by default (`JSON.stringify` returns `{}`).
- `size` is a property, not a method (`set.size`, not `set.size()`).

### Annotated Code Examples

#### Example 1: Creating an Empty Set

```js
// Step 1: Create an empty Set.
const set = new Set();

// Step 2: Verify it is empty.
console.log(set.size);           // 0
console.log(set instanceof Set); // true
```

**Expected Output:**
```
0
true
```

**Why This Output:** `new Set()` creates an empty Set with `size` 0. The instance inherits from `Set.prototype`, so `instanceof Set` is true.

#### Example 2: Creating a Set from an Array with Duplicates

```js
// Step 1: Provide an array with duplicates.
const numbers = [1, 2, 2, 3, 3, 3, 4];

// Step 2: Pass it to the Set constructor.
const unique = new Set(numbers);

// Step 3: Inspect.
console.log(unique.size);         // 4
console.log([...unique]);         // [1, 2, 3, 4]
```

**Expected Output:**
```
4
[ 1, 2, 3, 4 ]
```

**Why This Output:** The constructor iterates the array and adds each value; duplicates are ignored because Sets enforce uniqueness.

#### Example 3: Creating a Set from a String

```js
// Step 1: Pass a string to the Set constructor.
const letters = new Set("hello");

// Step 2: Inspect the unique characters.
console.log([...letters]); // ["h", "e", "l", "o"]
console.log(letters.size); // 4
```

**Expected Output:**
```
[ 'h', 'e', 'l', 'o' ]
4
```

**Why This Output:** Strings are iterable by character; the duplicate `"l"` is stored only once.

#### Example 4: Creating a Set from Another Set

```js
const original = new Set([1, 2, 3]);
const copy = new Set(original);

console.log(copy.size);         // 3
console.log(copy === original); // false
```

**Expected Output:**
```
3
false
```

**Why This Output:** Passing an existing Set iterates its values into a new Set. The two objects are distinct references.

### Real-World Cases

- **Deduplication:** `[...new Set(array)]` removes duplicates.
- **Tag systems:** Storing unique tags for an article.
- **Visited tracking:** Recording visited nodes in graph traversal.
- **Permission sets:** Storing unique roles or scopes.

---

## Set Creation

### Definitions

**Core Definition:** Set creation is the process of instantiating a `Set` object, either empty or pre-populated.

**Technical Definition:** Instantiation is performed via the `Set` constructor with `new`. The internal `[[SetData]]` slot is initialised to an empty List; if an iterable argument is provided, the `addEntriesFromIterable` abstract operation inserts each value, ignoring duplicates per SameValueZero.

**Beginner-Friendly Explanation:** You create a Set by calling `new Set()`. You can start empty or pass in any iterable to fill it immediately — duplicates are dropped.

### Purposes

- To obtain a mutable unique-value collection.
- To seed a set with known data.
- To convert arrays, strings, or other iterables into Set form.
- To remove duplicates in one step.

### Syntax Rules and Structure

**Syntax Variants:**
```js
new Set()                       // Empty
new Set([1, 2, 3])              // From array
new Set("abc")                  // From string
new Set(otherSet)               // Copy from another Set
new Set(map.keys())             // From a Map's keys
new Set(generatorFunction())    // From any iterable
```

**Constraints and Limitations:**
- Non-iterable arguments throw `TypeError`.
- Duplicate values are silently ignored.
- Object values are compared by reference.

### Annotated Code Examples

#### Example 1: Deduplicating an Array

```js
// Step 1: Define an array with duplicates.
const arr = [1, 2, 3, 2, 1, 4];

// Step 2: Convert to a Set and back to an array.
const unique = [...new Set(arr)];

console.log(unique); // [1, 2, 3, 4]
```

**Expected Output:**
```
[ 1, 2, 3, 4 ]
```

**Why This Output:** The Set constructor drops duplicates during iteration; spreading converts it back to an array.

#### Example 2: Object Reference Equality

```js
// Step 1: Create two structurally identical objects.
const a = { id: 1 };
const b = { id: 1 };

// Step 2: Add both to a Set.
const set = new Set([a, b]);

// Step 3: Observe the size.
console.log(set.size);        // 2
console.log(set.has(a));      // true
console.log(set.has(b));      // true
console.log(set.has({ id: 1 })); // false (different reference)
```

**Expected Output:**
```
2
true
true
false
```

**Why This Output:** Sets compare objects by reference, not structure. `a` and `b` are distinct objects, so both are stored. A new literal `{ id: 1 }` is a third reference, not present.

### Real-World Cases

- **Unique user IDs:** `new Set(userIds)`.
- **Character sets:** Validating input against allowed characters.
- **Event listeners:** Deduplicating handler functions.

---

## Set Methods

### Definitions

**Core Definition:** Set methods are the functions on `Set.prototype` that allow adding, testing, removing, and iterating values.

**Technical Definition:** The `Set.prototype` object defines `add`, `has`, `delete`, `clear`, `forEach`, `keys`, `values`, and `entries`, plus the `size` accessor and `Symbol.iterator`. Each method operates on the `[[SetData]]` internal slot of the receiver.

**Beginner-Friendly Explanation:** Set methods are the operations you perform — adding items, checking membership, removing them, and iterating over them.

---

### `add()`

#### Definitions

- **Core Definition:** `add()` inserts a value into the Set if it is not already present.
- **Technical Definition:** `Set.prototype.add(value)` normalises `-0` to `+0`, checks for SameValueZero equality against existing entries, and appends the value if absent. Returns the Set for chaining.
- **Beginner-Friendly Explanation:** `add()` puts a value into the Set. If it's already there, nothing changes.

#### Purposes

- To insert a new unique value.
- To enable method chaining.
- To build a Set incrementally.

#### Syntax Rules and Structure

```js
set.add(value)
```

**Component Breakdown:**
- `value`: Any JavaScript value.
- Returns: The same Set object (for chaining).

**Constraints and Limitations:**
- Adding a duplicate is a no-op (does not change `size` or iteration order).
- `-0` is normalised to `+0`.
- Object values are stored by reference.

#### Annotated Code Examples

```js
const set = new Set();

// Step 1: Add values with chaining.
set.add(1).add(2).add(3);

// Step 2: Attempt to add a duplicate.
set.add(2);

console.log(set.size);        // 3
console.log([...set]);        // [1, 2, 3]

// Step 3: Add an object.
const obj = { id: 1 };
set.add(obj);
console.log(set.has(obj));    // true
```

**Expected Output:**
```
3
[ 1, 2, 3 ]
true
```

**Why This Output:** Adding `2` again does not change the Set. Chaining works because `add()` returns the Set.

#### Real-World Cases

- **Visited sets:** `visited.add(node)` in graph traversal.
- **Tag collection:** `tags.add(tag)` as users apply tags.
- **Unique ID tracking:** `seen.add(id)` in streaming data.

---

### `has()`

#### Definitions

- **Core Definition:** `has()` tests whether a value exists in the Set.
- **Technical Definition:** `Set.prototype.has(value)` returns `true` if a SameValueZero-equal value exists in `[[SetData]]`, else `false`.
- **Beginner-Friendly Explanation:** `has()` answers "Is this value in the Set?"

#### Purposes

- To check membership efficiently (O(1) average).
- To guard against duplicate processing.
- To implement conditional logic based on presence.

#### Syntax Rules and Structure

```js
set.has(value)
```

**Returns:** Boolean.

**Constraints and Limitations:**
- Uses SameValueZero, so `NaN` matches `NaN`.
- Object values must be the same reference.

#### Annotated Code Examples

```js
const set = new Set([1, 2, 3, NaN]);

console.log(set.has(2));         // true
console.log(set.has(4));         // false
console.log(set.has(NaN));       // true (SameValueZero)
console.log(set.has("2"));       // false (different type)

// Object reference
const obj = { id: 1 };
set.add(obj);
console.log(set.has(obj));       // true
console.log(set.has({ id: 1 })); // false
```

**Expected Output:**
```
true
false
true
false
true
false
```

**Why This Output:** `has()` uses SameValueZero, so `NaN` is found. Numbers and strings are distinct. Object identity matters.

#### Real-World Cases

- **Duplicate detection:** `if (seen.has(id)) return;`.
- **Permission checks:** `roles.has("admin")`.
- **Whitelist validation:** `allowedExtensions.has(ext)`.

---

### `delete()`

#### Definitions

- **Core Definition:** `delete()` removes a value from the Set.
- **Technical Definition:** `Set.prototype.delete(value)` finds a SameValueZero-equal value, removes it from `[[SetData]]`, and returns `true`. Returns `false` if no such value exists.
- **Beginner-Friendly Explanation:** `delete()` removes a value from the Set.

#### Purposes

- To remove values that are no longer needed.
- To implement dynamic membership.
- To free resources associated with values.

#### Syntax Rules and Structure

```js
set.delete(value)
```

**Returns:** `true` if a value was removed, `false` otherwise.

**Constraints and Limitations:**
- Removing a non-existent value is a no-op returning `false`.
- Object values must match by reference.

#### Annotated Code Examples

```js
const set = new Set(["a", "b", "c"]);

console.log(set.delete("b"));  // true
console.log(set.size);         // 2
console.log(set.delete("z"));  // false
console.log(set.has("b"));     // false
```

**Expected Output:**
```
true
2
false
false
```

**Why This Output:** `delete("b")` finds and removes the value; `delete("z")` finds nothing.

#### Real-World Cases

- **Session expiry:** Removing expired session IDs.
- **Dynamic permissions:** Revoking roles.
- **Cache invalidation:** Removing stale entries.

---

### `clear()`

#### Definitions

- **Core Definition:** `clear()` removes all values from the Set.
- **Technical Definition:** `Set.prototype.clear()` sets the `[[SetData]]` list to empty. Returns `undefined`.
- **Beginner-Friendly Explanation:** `clear()` empties the Set completely.

#### Purposes

- To reset a Set without creating a new instance.
- To release references to all values.
- To reinitialise state.

#### Syntax Rules and Structure

```js
set.clear()
```

**Returns:** `undefined`.

**Constraints and Limitations:**
- Irreversible.
- Does not affect other references to the same Set.

#### Annotated Code Examples

```js
const set = new Set([1, 2, 3]);
console.log(set.size); // 3

set.clear();
console.log(set.size); // 0
console.log(set.has(1)); // false
```

**Expected Output:**
```
3
0
false
```

**Why This Output:** `clear()` empties `[[SetData]]`.

#### Real-World Cases

- **Logout:** Clearing session scopes.
- **Test teardown:** Resetting shared state.
- **Cache reset:** Emptying a memoisation set.

---

### `size`

#### Definitions

- **Core Definition:** `size` is a read-only accessor returning the number of values in the Set.
- **Technical Definition:** `get Set.prototype.size` returns the length of the `[[SetData]]` list.
- **Beginner-Friendly Explanation:** `size` tells you how many unique values are in the Set.

#### Purposes

- To check emptiness (`size === 0`).
- To report collection statistics.
- To iterate by count.

#### Syntax Rules and Structure

```js
set.size
```

**Returns:** A non-negative integer.

**Constraints and Limitations:**
- Read-only; assignment throws in strict mode.
- Property, not method (`set.size`, not `set.size()`).

#### Annotated Code Examples

```js
const set = new Set();
console.log(set.size);      // 0

set.add(1).add(2).add(2);
console.log(set.size);      // 2

set.delete(1);
console.log(set.size);      // 1
```

**Expected Output:**
```
0
2
1
```

**Why This Output:** `size` reflects the current number of unique values.

#### Real-World Cases

- **Cardinality checks:** Reporting unique visitor counts.
- **Guard clauses:** `if (set.size === 0) return;`.
- **Progress metrics:** `completed.size / total`.

---

### `keys()`

#### Definitions

- **Core Definition:** `keys()` returns an iterator over the Set's values in insertion order.
- **Technical Definition:** `Set.prototype.keys()` is the same function object as `Set.prototype.values` (provided for Map-like API symmetry). It returns a Set Iterator yielding each value.
- **Beginner-Friendly Explanation:** `keys()` and `values()` do the same thing for Sets — they give you an iterator of all values.

#### Purposes

- To iterate over values using a Map-like API.
- To maintain API symmetry with `Map`.

#### Syntax Rules and Structure

```js
set.keys()
```

**Returns:** A `SetIterator`.

**Constraints and Limitations:**
- Identical to `values()`.
- Single-use iterator.

#### Annotated Code Examples

```js
const set = new Set(["x", "y", "z"]);

console.log([...set.keys()]); // ["x", "y", "z"]
```

**Expected Output:**
```
[ 'x', 'y', 'z' ]
```

**Why This Output:** `keys()` yields each value in insertion order.

#### Real-World Cases

- **API compatibility:** Code that treats Sets and Maps uniformly.
- **Iteration:** General value traversal.

---

### `values()`

#### Definitions

- **Core Definition:** `values()` returns an iterator over the Set's values in insertion order.
- **Technical Definition:** `Set.prototype.values()` returns a Set Iterator yielding each value in `[[SetData]]` order. It is the same as `Set.prototype[Symbol.iterator]`.
- **Beginner-Friendly Explanation:** `values()` gives you an iterator of all the values in the Set.

#### Purposes

- To iterate over values.
- To convert values to an array.
- To feed values into other operations.

#### Syntax Rules and Structure

```js
set.values()
```

**Returns:** A `SetIterator`.

#### Annotated Code Examples

```js
const set = new Set([10, 20, 30]);

let sum = 0;
for (const value of set.values()) {
  sum += value;
}
console.log(sum); // 60

console.log([...set.values()]); // [10, 20, 30]
```

**Expected Output:**
```
60
[ 10, 20, 30 ]
```

**Why This Output:** `values()` yields each value; the loop sums them; spreading collects them.

#### Real-World Cases

- **Aggregation:** Summing scores.
- **Serialisation:** `JSON.stringify([...set.values()])`.
- **Reporting:** Listing all unique items.

---

### `entries()`

#### Definitions

- **Core Definition:** `entries()` returns an iterator over `[value, value]` pairs in insertion order.
- **Technical Definition:** `Set.prototype.entries()` returns a Set Iterator yielding two-element arrays where both elements are the same value. This provides API symmetry with `Map.prototype.entries`.
- **Beginner-Friendly Explanation:** `entries()` gives you pairs where both the "key" and "value" are the same — useful for compatibility with Map-like code.

#### Purposes

- To maintain API symmetry with `Map`.
- To enable code that treats Sets and Maps uniformly.
- To destructure in `for...of`.

#### Syntax Rules and Structure

```js
set.entries()
```

**Returns:** A `SetIterator` yielding `[value, value]` arrays.

#### Annotated Code Examples

```js
const set = new Set(["a", "b"]);

for (const [k, v] of set.entries()) {
  console.log(k, v);
}
```

**Expected Output:**
```
a a
b b
```

**Why This Output:** `entries()` yields `[value, value]` pairs; destructuring assigns the same value to both variables.

#### Real-World Cases

- **Generic iteration:** Code that handles both Maps and Sets.
- **Debugging:** Printing values in pair form.

---

### `forEach()`

#### Definitions

- **Core Definition:** `forEach()` executes a callback for each value in insertion order.
- **Technical Definition:** `Set.prototype.forEach(callback, thisArg)` invokes `callback(value, value, set)` for each entry. `thisArg` sets `this` inside the callback.
- **Beginner-Friendly Explanation:** `forEach()` runs a function on every value in the Set.

#### Purposes

- To perform side effects for each value.
- To accumulate results into external variables.
- To integrate with callback-style code.

#### Syntax Rules and Structure

```js
set.forEach((value, value2, set) => { ... }, thisArg)
```

**Component Breakdown:**
- `value`: The current value.
- `value2`: The same value (Map-like symmetry).
- `set`: The Set being iterated.
- `thisArg` (optional): Value of `this` inside the callback.

**Constraints and Limitations:**
- Cannot be broken early (no `break`); use `for...of`.
- Not chainable (returns `undefined`).
- Order is insertion order.

#### Annotated Code Examples

```js
const set = new Set([1, 2, 3]);

// Step 1: Log each value.
set.forEach((value) => {
  console.log(value);
});

// Step 2: Accumulate.
let total = 0;
set.forEach((value) => { total += value; });
console.log("Total:", total);
```

**Expected Output:**
```
1
2
3
Total: 6
```

**Why This Output:** `forEach` visits each value in order.

#### Real-World Cases

- **Logging:** Iterating over unique IDs.
- **Aggregation:** Summing values.
- **Event dispatch:** Calling handlers registered per value.

---

## Set Iteration

### Definitions

**Core Definition:** Set iteration is the process of traversing a Set's values in insertion order.

**Technical Definition:** Sets implement the iterable protocol via `Set.prototype[Symbol.iterator]`, which is the same function object as `Set.prototype.values`. Iteration is insertion-ordered and live (mutations during iteration are observable).

**Beginner-Friendly Explanation:** You can loop over a Set using `for...of`, spread it into an array, or use `forEach`. The order is always the order you added values.

### Purposes

- To process all values.
- To convert Sets to arrays or other structures.
- To implement algorithms that depend on iteration order.

### Syntax Rules and Structure

```js
// for...of
for (const value of set) { ... }

// Spread
[...set]                // array of values

// Destructuring
const [first, second] = set;

// Array.from
Array.from(set);
```

**Constraints and Limitations:**
- The iterator is live; adding values during iteration may lead to infinite loops.
- Deleting values during iteration is allowed but the deleted value is skipped.

### Annotated Code Examples

#### Example 1: Insertion Order Guarantee

```js
const set = new Set();
set.add(3);
set.add(1);
set.add(2);

for (const v of set) {
  console.log(v);
}
```

**Expected Output:**
```
3
1
2
```

**Why This Output:** Sets preserve insertion order.

#### Example 2: Converting to Array and Back

```js
const set = new Set(["a", "b", "c"]);

const arr = [...set];
console.log(arr); // ["a", "b", "c"]

const set2 = new Set(arr);
console.log(set2.size); // 3
```

**Expected Output:**
```
[ 'a', 'b', 'c' ]
3
```

**Why This Output:** Spread collects values; the Set constructor accepts them back.

### Real-World Cases

- **Ordered deduplication:** Preserving first-occurrence order.
- **Serialisation round-trips:** Set → array → JSON → array → Set.
- **Algorithm implementation:** BFS visited sets.

---

## Set Operations

### Definitions

**Core Definition:** Set operations are mathematical operations (union, intersection, difference, symmetric difference, subset/superset checks) that combine or compare Sets.

**Technical Definition:** ES2025 introduces native methods on `Set.prototype`: `union(other)`, `intersection(other)`, `difference(other)`, `symmetricDifference(other)`, `isSubsetOf(other)`, `isSupersetOf(other)`, and `isDisjointFrom(other)`. These accept any set-like object (with `size`, `has`, and `keys`). Prior to ES2025, these operations were implemented manually.

**Beginner-Friendly Explanation:** Set operations let you combine Sets mathematically: union (all items from both), intersection (items in both), difference (items in the first but not the second), and symmetric difference (items in exactly one).

### Purposes

- To combine sets of items.
- To find common or unique elements.
- To compare sets for containment or overlap.
- To implement permission logic, tag filtering, and data analysis.

---

### Union

#### Definitions

- **Core Definition:** Union returns a new Set containing all values from both Sets.
- **Technical Definition:** `Set.prototype.union(other)` returns a new Set with elements from `this` and `other`, deduplicated.
- **Beginner-Friendly Explanation:** Union combines two Sets into one with no duplicates.

#### Syntax

```js
const result = setA.union(setB);
```

#### Annotated Code Example

```js
const a = new Set([1, 2, 3]);
const b = new Set([3, 4, 5]);

const union = a.union(b);
console.log([...union]); // [1, 2, 3, 4, 5]
```

**Expected Output:**
```
[ 1, 2, 3, 4, 5 ]
```

**Why This Output:** Union collects all unique values from both Sets.

#### Real-World Cases

- **Merging tag lists:** Combining tags from multiple sources.
- **Permission aggregation:** All permissions across roles.
- **Contact merging:** Combining contact lists.

---

### Intersection

#### Definitions

- **Core Definition:** Intersection returns a new Set containing only values present in both Sets.
- **Technical Definition:** `Set.prototype.intersection(other)` returns a new Set with elements in both `this` and `other`.
- **Beginner-Friendly Explanation:** Intersection finds what two Sets have in common.

#### Syntax

```js
const result = setA.intersection(setB);
```

#### Annotated Code Example

```js
const a = new Set([1, 2, 3, 4]);
const b = new Set([3, 4, 5, 6]);

const intersection = a.intersection(b);
console.log([...intersection]); // [3, 4]
```

**Expected Output:**
```
[ 3, 4 ]
```

**Why This Output:** Only values present in both Sets are included.

#### Real-World Cases

- **Common followers:** Intersection of follower lists.
- **Shared permissions:** What two roles have in common.
- **Matching tags:** Articles with overlapping tags.

---

### Difference

#### Definitions

- **Core Definition:** Difference returns a new Set with values in the first Set but not the second.
- **Technical Definition:** `Set.prototype.difference(other)` returns a new Set with elements in `this` but not in `other`.
- **Beginner-Friendly Explanation:** Difference finds what's in the first Set but not the second.

#### Syntax

```js
const result = setA.difference(setB);
```

#### Annotated Code Example

```js
const a = new Set([1, 2, 3, 4]);
const b = new Set([3, 4, 5]);

const diff = a.difference(b);
console.log([...diff]); // [1, 2]
```

**Expected Output:**
```
[ 1, 2 ]
```

**Why This Output:** Only values in `a` but not in `b` are included.

#### Real-World Cases

- **Removed permissions:** Permissions a user had but lost.
- **New items:** Items added since last sync.
- **Excluded tags:** Tags to filter out.

---

### Symmetric Difference

#### Definitions

- **Core Definition:** Symmetric difference returns a new Set with values in exactly one of the two Sets (not both).
- **Technical Definition:** `Set.prototype.symmetricDifference(other)` returns a new Set with elements in `this` or `other` but not both.
- **Beginner-Friendly Explanation:** Symmetric difference finds what's unique to each Set — items that aren't shared.

#### Syntax

```js
const result = setA.symmetricDifference(setB);
```

#### Annotated Code Example

```js
const a = new Set([1, 2, 3]);
const b = new Set([3, 4, 5]);

const symDiff = a.symmetricDifference(b);
console.log([...symDiff]); // [1, 2, 4, 5]
```

**Expected Output:**
```
[ 1, 2, 4, 5 ]
```

**Why This Output:** Values in both Sets (3) are excluded; everything else is included.

#### Real-World Cases

- **Divergent data:** Records that differ between two systems.
- **Change detection:** Items added or removed between snapshots.
- **Tag divergence:** Tags unique to one of two articles.

---

### Additional ES2025 Operations

#### `isSubsetOf(other)`

Returns `true` if every element of `this` is in `other`.

```js
new Set([1, 2]).isSubsetOf(new Set([1, 2, 3])); // true
```

#### `isSupersetOf(other)`

Returns `true` if every element of `other` is in `this`.

```js
new Set([1, 2, 3]).isSupersetOf(new Set([1, 2])); // true
```

#### `isDisjointFrom(other)`

Returns `true` if `this` and `other` have no elements in common.

```js
new Set([1, 2]).isDisjointFrom(new Set([3, 4])); // true
```

**Note:** These operations are ES2025 features; check browser/runtime support or use a polyfill for older environments.

### Real-World Cases (Set Operations)

- **Access control:** Checking if user roles are a subset of required roles.
- **Data reconciliation:** Finding differences between two datasets.
- **Recommendation systems:** Finding common interests (intersection).

---

## Set vs Array

### Definitions

**Core Definition:** `Set` and `Array` are both ordered collections, but they differ in uniqueness, access patterns, and performance characteristics.

**Technical Definition:** Arrays are indexed, ordered lists allowing duplicates, with O(1) index access but O(n) search. Sets are unordered-by-index collections of unique values with O(1) average membership testing but no index access.

**Beginner-Friendly Explanation:** Arrays are lists where order and duplicates matter, and you access items by position. Sets are collections where each item appears once, and you check membership rather than position.

### Comparison Table

| Feature | Set | Array |
|---|---|---|
| Duplicates | Not allowed | Allowed |
| Index access | No | Yes (`arr[0]`) |
| Membership test | O(1) `has()` | O(n) `includes()` |
| Insert | O(1) `add()` | O(1) `push()` |
| Delete by value | O(1) `delete()` | O(n) `splice()` |
| Order | Insertion order | Index order |
| Size | `set.size` | `arr.length` |
| Iteration | `for...of`, `forEach` | `for`, `for...of`, `forEach` |
| Map/filter/reduce | No (convert to array) | Yes |
| JSON serialisation | No | Yes |

### Purposes

- To choose the right structure for the task.
- To optimise membership testing and deduplication.
- To preserve order and allow duplicates when needed.

### Annotated Code Examples

#### Example 1: Membership Test Performance

```js
const N = 1_000_000;
const arr = Array.from({ length: N }, (_, i) => i);
const set = new Set(arr);

// Array includes: O(n)
let start = performance.now();
arr.includes(N - 1);
console.log(`Array includes: ${(performance.now() - start).toFixed(4)} ms`);

// Set has: O(1)
start = performance.now();
set.has(N - 1);
console.log(`Set has: ${(performance.now() - start).toFixed(4)} ms`);
```

**Expected Output (approximate):**
```
Array includes: 0.5000 ms
Set has: 0.0050 ms
```

**Why This Output:** Set membership is O(1); array membership is O(n). The difference is dramatic for large collections.

#### Example 2: Deduplication

```js
// Array: manual deduplication
const arr = [1, 2, 2, 3, 3, 3];
const uniqueArr = [...new Set(arr)];
console.log(uniqueArr); // [1, 2, 3]
```

**Expected Output:**
```
[ 1, 2, 3 ]
```

**Why This Output:** Converting to a Set removes duplicates; spreading converts back to an array.

### Real-World Cases

- **Membership checks:** Use Set for "is this ID in the list?".
- **Ordered lists:** Use Array for sequences where duplicates matter.
- **Deduplication:** Use Set to remove duplicates.

---

## Set Use Cases

### Definitions

**Core Definition:** Set use cases are scenarios where Set's uniqueness and O(1) membership properties make it the superior choice.

**Technical Definition:** Sets excel in deduplication, membership testing, set algebra, and tracking visited/seen states. They are less suitable for indexed access, ordered sequences with duplicates, or JSON-serialised data.

**Beginner-Friendly Explanation:** Use a Set when you need unique values and fast "is this in here?" checks.

### Common Use Cases

1. **Deduplication**
   ```js
   const unique = [...new Set([1, 1, 2, 3, 3])]; // [1, 2, 3]
   ```

2. **Membership Testing**
   ```js
   const allowed = new Set(["jpg", "png", "gif"]);
   if (allowed.has(extension)) { ... }
   ```

3. **Visited Tracking (Graph Traversal)**
   ```js
   const visited = new Set();
   function dfs(node) {
     if (visited.has(node)) return;
     visited.add(node);
     node.neighbours.forEach(dfs);
   }
   ```

4. **Tag/Keyword Systems**
   ```js
   const tags = new Set();
   tags.add("javascript").add("web");
   ```

5. **Set Algebra**
   ```js
   const common = setA.intersection(setB);
   ```

6. **Removing Duplicates from Streams**
   ```js
   const seen = new Set();
   stream.filter((item) => !seen.has(item.id) && seen.add(item.id));
   ```

### Annotated Code Examples

#### Example 1: Graph Traversal with Visited Set

```js
// Step 1: Define a graph.
const graph = {
  A: ["B", "C"],
  B: ["A", "D"],
  C: ["A", "D"],
  D: ["B", "C"],
};

// Step 2: Traverse with a visited Set.
const visited = new Set();
function dfs(node) {
  if (visited.has(node)) return;
  visited.add(node);
  console.log("Visited:", node);
  graph[node].forEach(dfs);
}

dfs("A");
console.log("Total visited:", visited.size);
```

**Expected Output:**
```
Visited: A
Visited: B
Visited: D
Visited: C
Total visited: 4
```

**Why This Output:** `visited` prevents revisiting nodes. The Set's O(1) `has()` makes the check efficient.

#### Example 2: Unique Word Count

```js
const text = "the quick brown fox jumps over the lazy dog the fox";
const words = text.split(" ");
const uniqueWords = new Set(words);

console.log("Total words:", words.length);       // 12
console.log("Unique words:", uniqueWords.size);  // 9
```

**Expected Output:**
```
Total words: 12
Unique words: 9
```

**Why This Output:** Splitting gives all words (with duplicates); the Set keeps only unique ones.

### Real-World Cases

- **React internals:** Uses Sets for effect dependencies.
- **Vue reactivity:** Tracks dependencies with Sets.
- **Lodash:** `_.uniq()` uses Set internally.
- **Databases:** Query planners use Sets for unique column values.

---

## Set Performance

### Definitions

**Core Definition:** Set performance refers to the time and space complexity characteristics of Set operations relative to alternative structures.

**Technical Definition:** In V8 and other engines, `Set` is implemented as a hash table (ordered hash set) with O(1) average time for `add`, `has`, and `delete`. Iteration is O(n). Memory overhead per entry is higher than arrays due to hash table structures. `WeakSet` allows garbage collection of object keys.

**Beginner-Friendly Explanation:** Sets are fast for adding, checking, and removing items — even with lots of data. They use more memory than arrays but save time on membership checks.

### Performance Characteristics

| Operation | Set | Array |
|---|---|---|
| Add | O(1) amortised | O(1) `push` |
| Has/Includes | O(1) average | O(n) |
| Delete by value | O(1) average | O(n) `splice` |
| Iterate | O(n) | O(n) |
| Size | O(1) | O(1) `length` |
| Memory per entry | Higher | Lower |

### Annotated Code Examples

#### Example 1: Benchmarking Membership

```js
const N = 100_000;
const arr = Array.from({ length: N }, (_, i) => i);
const set = new Set(arr);

let start = performance.now();
for (let i = 0; i < 1000; i++) arr.includes(N - 1);
console.log(`Array includes x1000: ${(performance.now() - start).toFixed(2)} ms`);

start = performance.now();
for (let i = 0; i < 1000; i++) set.has(N - 1);
console.log(`Set has x1000: ${(performance.now() - start).toFixed(2)} ms`);
```

**Expected Output (approximate):**
```
Array includes x1000: 45.30 ms
Set has x1000: 0.25 ms
```

**Why This Output:** Set membership is O(1); array membership is O(n). Repeating the check amplifies the difference.

#### Example 2: Deduplication Performance

```js
const N = 1_000_000;
const arr = Array.from({ length: N }, () => Math.floor(Math.random() * 1000));

const start = performance.now();
const unique = [...new Set(arr)];
console.log(`Dedup time: ${(performance.now() - start).toFixed(2)} ms`);
console.log(`Unique count: ${unique.length}`);
```

**Expected Output (approximate):**
```
Dedup time: 85.40 ms
Unique count: 1000
```

**Why This Output:** The Set constructor iterates once, dropping duplicates in O(n) total time.

### Constraints and Limitations

- Set operations are O(1) average, but worst-case can degrade (rare with good hash functions).
- Memory overhead per entry is higher than arrays.
- `WeakSet` is not iterable and only accepts objects.
- Very large Sets may cause rehashing pauses.

### Real-World Cases

- **High-frequency membership:** Use Set for fast checks.
- **Large datasets:** Use Set for deduplication.
- **Memory-constrained:** Use arrays or typed arrays when possible.
- **Object tracking:** Use WeakSet for garbage-collectable membership.

---

## References

- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Set Objects — https://tc39.es/ecma262/multipage/keyed-collections.html#sec-set-objects
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Set Methods (ES2025) — https://tc39.es/ecma262/multipage/keyed-collections.html#sec-set-objects
- MDN Web Docs – Set — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set
- MDN Web Docs – Set.prototype.add() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set/add
- MDN Web Docs – Set.prototype.has() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set/has
- MDN Web Docs – Set.prototype.delete() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set/delete
- MDN Web Docs – Set.prototype.clear() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set/clear
- MDN Web Docs – Set.prototype.size — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set/size
- MDN Web Docs – Set.prototype.keys() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set/keys
- MDN Web Docs – Set.prototype.values() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set/values
- MDN Web Docs – Set.prototype.entries() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set/entries
- MDN Web Docs – Set.prototype.forEach() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set/forEach
- MDN Web Docs – Set.prototype.union() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set/union
- MDN Web Docs – Set.prototype.intersection() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set/intersection
- MDN Web Docs – Set.prototype.difference() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set/difference
- MDN Web Docs – Set.prototype.symmetricDifference() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set/symmetricDifference
- MDN Web Docs – Set.prototype.isSubsetOf() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set/isSubsetOf
- MDN Web Docs – Set.prototype.isSupersetOf() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set/isSupersetOf
- MDN Web Docs – Set.prototype.isDisjointFrom() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set/isDisjointFrom
- MDN Web Docs – Keyed collections — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Keyed_collections
- TC39 Proposal – Set Methods — https://github.com/tc39/proposal-set-methods
- V8 Blog – Fast Properties in V8 — https://v8.dev/blog/fast-properties