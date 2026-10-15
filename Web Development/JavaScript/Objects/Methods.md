# JavaScript Object Methods — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Object methods are the static methods on the `Object` constructor and the instance methods on `Object.prototype` that provide utilities for inspecting, transforming, protecting, and interacting with JavaScript objects.

**Technical Definition:** Per ECMAScript 2025 (ECMA-262, §20.1), the `Object` constructor provides static methods for object manipulation, including property enumeration (`keys`, `values`, `entries`, `fromEntries`), copying (`assign`), integrity control (`freeze`, `seal`, `preventExtensions` and their `is*` counterparts), value comparison (`is`), ownership testing (`hasOwn`), and grouping (`groupBy`, ES2024). The `Object.prototype` object provides instance methods inherited by all objects: `hasOwnProperty`, `toString`, `valueOf`, `isPrototypeOf`, and `propertyIsEnumerable`. These methods are non-enumerable and non-writable by default.

**Beginner-Friendly Explanation:** Object methods are helper functions that come built into JavaScript for working with objects. Some are called on the `Object` constructor itself (like `Object.keys(obj)`), and some are called on any object instance (like `obj.hasOwnProperty("name")`). They let you list properties, copy objects, lock objects to prevent changes, check ownership, and more — all without writing custom code.

### Key Characteristics

- **Two categories:** Static methods on `Object` and instance methods on `Object.prototype`.
- **Static methods take the object as first argument:** `Object.keys(obj)`.
- **Instance methods are called on the object:** `obj.toString()`.
- **Non-enumerable by default:** `Object.prototype` methods do not appear in `for...in`.
- **Some are ES2024+ features:** `Object.groupBy` is relatively new.
- **Integrity methods:** `freeze`, `seal`, and `preventExtensions` progressively restrict modification.
- **Enumeration methods:** `keys`, `values`, `entries` respect enumerability.

### Prerequisites

- Basic JavaScript syntax (objects, properties, functions).
- Understanding of property attributes (enumerable, writable, configurable).
- Familiarity with the prototype chain.
- Awareness of strict mode behaviour.
- Basic knowledge of arrays and iteration.

### Related Programming Areas

- **Reflection and metaprogramming:** Inspecting and modifying objects at runtime.
- **Immutability:** Freezing and sealing objects.
- **Functional programming:** Transforming objects with `entries` and `fromEntries`.
- **Data processing:** Grouping, filtering, and mapping object data.
- **Serialisation:** Using `keys`, `values`, and `entries` for JSON conversion.
- **Prototype manipulation:** `isPrototypeOf`, `getPrototypeOf`, `setPrototypeOf`.

---

## `Object.keys()`

### Definitions

**Core Definition:** `Object.keys()` returns an array of an object's own enumerable string-keyed property names.

**Technical Definition:** Per ECMA-262 §20.1.2.17, `Object.keys(O)` returns a new array containing the keys of all own enumerable properties of `O` whose keys are strings. The order matches `[[OwnPropertyKeys]]`: integer-like keys ascending, then other string keys in insertion order.

**Beginner-Friendly Explanation:** `Object.keys()` gives you a list of all the "visible" property names on an object.

### Purposes

- To list an object's own enumerable property names.
- To iterate over an object's keys.
- To count properties (`Object.keys(obj).length`).
- To convert objects to arrays for further processing.

### Syntax Rules and Structure

```js
Object.keys(obj)
```

**Returns:** An array of strings.

**Constraints and Limitations:**
- Excludes non-enumerable properties.
- Excludes symbol-keyed properties.
- Excludes inherited properties.
- Throws `TypeError` for `null` or `undefined` in ES5; ES2015+ coerces to object (returns `[]` for `null`/`undefined` in some engines).

### Annotated Code Examples

#### Example 1: Basic Usage

```js
const user = { name: "Alice", age: 30, city: "Paris" };

// Step 1: Get all keys.
const keys = Object.keys(user);
console.log(keys); // ["name", "age", "city"]

// Step 2: Iterate over keys.
for (const key of Object.keys(user)) {
  console.log(`${key}: ${user[key]}`);
}
```

**Expected Output:**
```
[ 'name', 'age', 'city' ]
name: Alice
age: 30
city: Paris
```

**Why This Output:** `Object.keys` returns an array of own enumerable string keys, which can be iterated.

#### Example 2: Counting Properties

```js
const config = { a: 1, b: 2, c: 3 };
console.log(Object.keys(config).length); // 3
```

**Expected Output:**
```
3
```

**Why This Output:** `Object.keys` returns an array whose length equals the number of own enumerable properties.

### Real-World Cases

- **Serialisation:** Building custom JSON output.
- **Form validation:** Iterating form field names.
- **Configuration:** Enumerating settings keys.

---

## `Object.values()`

### Definitions

**Core Definition:** `Object.values()` returns an array of an object's own enumerable string-keyed property values.

**Technical Definition:** Per ECMA-262 §20.1.2.23, `Object.values(O)` returns a new array containing the values of all own enumerable string-keyed properties of `O`, in the same order as `Object.keys(O)`.

**Beginner-Friendly Explanation:** `Object.values()` gives you a list of all the values stored in an object's visible properties.

### Purposes

- To extract values for aggregation (sum, average, etc.).
- To iterate over values without needing keys.
- To convert object values to an array.

### Syntax Rules and Structure

```js
Object.values(obj)
```

**Returns:** An array of values.

**Constraints and Limitations:**
- Excludes non-enumerable and symbol-keyed properties.
- Excludes inherited properties.

### Annotated Code Examples

```js
const scores = { math: 90, science: 85, history: 78 };

// Step 1: Get all values.
const values = Object.values(scores);
console.log(values); // [90, 85, 78]

// Step 2: Sum the values.
const total = Object.values(scores).reduce((sum, v) => sum + v, 0);
console.log(total); // 253

// Step 3: Average.
console.log(total / Object.values(scores).length); // 84.333...
```

**Expected Output:**
```
[ 90, 85, 78 ]
253
84.33333333333333
```

**Why This Output:** `Object.values` extracts all values, enabling aggregation.

### Real-World Cases

- **Statistics:** Summing, averaging, or finding extremes.
- **Chart data:** Extracting values for visualisation.
- **Validation:** Checking all values meet criteria.

---

## `Object.entries()`

### Definitions

**Core Definition:** `Object.entries()` returns an array of an object's own enumerable string-keyed `[key, value]` pairs.

**Technical Definition:** Per ECMA-262 §20.1.2.5, `Object.entries(O)` returns a new array where each element is a two-element array `[key, value]` for each own enumerable string-keyed property of `O`, in the same order as `Object.keys(O)`.

**Beginner-Friendly Explanation:** `Object.entries()` gives you both keys and values together as pairs.

### Purposes

- To iterate over key-value pairs.
- To convert objects to Maps or arrays of pairs.
- To transform objects into other structures.
- To filter or map object entries.

### Syntax Rules and Structure

```js
Object.entries(obj)
```

**Returns:** An array of `[key, value]` pairs.

**Constraints and Limitations:**
- Excludes non-enumerable and symbol-keyed properties.
- Excludes inherited properties.

### Annotated Code Examples

#### Example 1: Destructuring Entries

```js
const user = { name: "Alice", age: 30 };

for (const [key, value] of Object.entries(user)) {
  console.log(`${key}: ${value}`);
}
```

**Expected Output:**
```
name: Alice
age: 30
```

**Why This Output:** `Object.entries` returns pairs that can be destructured in a loop.

#### Example 2: Converting to Map

```js
const obj = { a: 1, b: 2 };
const map = new Map(Object.entries(obj));
console.log(map.get("a")); // 1
console.log(map.size);     // 2
```

**Expected Output:**
```
1
2
```

**Why This Output:** `Object.entries` provides the iterable of pairs that the Map constructor expects.

#### Example 3: Transforming Objects

```js
const prices = { apple: 1.5, banana: 0.75, cherry: 3.0 };

// Double all prices.
const doubled = Object.fromEntries(
  Object.entries(prices).map(([k, v]) => [k, v * 2])
);
console.log(doubled); // { apple: 3, banana: 1.5, cherry: 6 }
```

**Expected Output:**
```
{ apple: 3, banana: 1.5, cherry: 6 }
```

**Why This Output:** `entries` → `map` → `fromEntries` is a common transformation pattern.

### Real-World Cases

- **Data transformation:** Converting objects between shapes.
- **Map/Object conversion:** Bridging the two structures.
- **Filtering:** Selecting a subset of properties.

---

## `Object.fromEntries()`

### Definitions

**Core Definition:** `Object.fromEntries()` transforms a list of key-value pairs into an object.

**Technical Definition:** Per ECMA-262 §20.1.2.7, `Object.fromEntries(iterable)` returns a new object whose properties are given by the entries of the iterable. Each entry must be a two-element object (array-like); the first element becomes the key (coerced to string or used as a symbol), and the second becomes the value.

**Beginner-Friendly Explanation:** `Object.fromEntries()` is the reverse of `Object.entries()` — it takes pairs and builds an object.

### Purposes

- To convert Maps to plain objects.
- To build objects from arrays of pairs.
- To reverse `Object.entries()` transformations.
- To create objects dynamically from computed data.

### Syntax Rules and Structure

```js
Object.fromEntries(iterable)
```

**Component Breakdown:**
- `iterable`: Any iterable of `[key, value]` pairs (array, Map, generator).
- Returns: A new object.

**Constraints and Limitations:**
- Entries must be iterable objects with exactly two elements.
- Duplicate keys: later entries overwrite earlier ones.
- Symbol keys are preserved.

### Annotated Code Examples

#### Example 1: From Map to Object

```js
const map = new Map([["a", 1], ["b", 2], ["c", 3]]);
const obj = Object.fromEntries(map);
console.log(obj); // { a: 1, b: 2, c: 3 }
```

**Expected Output:**
```
{ a: 1, b: 2, c: 3 }
```

**Why This Output:** `Object.fromEntries` accepts any iterable of pairs, including Maps.

#### Example 2: Transforming Entries

```js
const scores = { math: 90, science: 85 };

// Convert to entries, transform, back to object.
const grades = Object.fromEntries(
  Object.entries(scores).map(([subject, score]) => [
    subject,
    score >= 90 ? "A" : "B",
  ])
);
console.log(grades); // { math: "A", science: "B" }
```

**Expected Output:**
```
{ math: 'A', science: 'B' }
```

**Why This Output:** The pattern `entries → map → fromEntries` transforms object values.

### Real-World Cases

- **Map/Object conversion:** Bridging data structures.
- **Data transformation pipelines:** Functional processing of objects.
- **API normalisation:** Converting arrays of records to keyed objects.

---

## `Object.assign()`

### Definitions

**Core Definition:** `Object.assign()` copies all enumerable own properties from one or more source objects to a target object, returning the target.

**Technical Definition:** Per ECMA-262 §20.1.2.1, `Object.assign(target, ...sources)` invokes `[[Set]]` on the target for each own enumerable property (string or symbol keyed) of each source, in order. Returns the target.

**Beginner-Friendly Explanation:** `Object.assign()` is like copying everything from one or more objects into another object.

### Purposes

- To merge objects.
- To copy properties from one object to another.
- To clone objects shallowly.
- To apply default values.

### Syntax Rules and Structure

```js
Object.assign(target, ...sources)
```

**Component Breakdown:**
- `target`: The object to receive properties.
- `...sources`: One or more source objects.
- Returns: The target object (mutated).

**Constraints and Limitations:**
- Shallow copy (nested objects are shared references).
- Only copies enumerable own properties.
- Mutates the target.
- Later sources overwrite earlier ones.

### Annotated Code Examples

#### Example 1: Merging Objects

```js
const defaults = { theme: "light", fontSize: 14 };
const userPrefs = { fontSize: 16 };

const config = Object.assign({}, defaults, userPrefs);
console.log(config); // { theme: "light", fontSize: 16 }
```

**Expected Output:**
```
{ theme: 'light', fontSize: 16 }
```

**Why This Output:** Properties from `userPrefs` override those from `defaults` in the target.

#### Example 2: Shallow Copy Pitfall

```js
const original = { nested: { value: 1 } };
const copy = Object.assign({}, original);

copy.nested.value = 99;
console.log(original.nested.value); // 99 (shared reference!)
```

**Expected Output:**
```
99
```

**Why This Output:** `Object.assign` performs a shallow copy; nested objects are shared. Use `structuredClone()` for deep copies.

### Real-World Cases

- **Configuration merging:** Combining defaults with user options.
- **State updates:** Creating new state objects with overrides.
- **Shallow cloning:** Duplicating objects before mutation.

---

## `Object.freeze()`

### Definitions

**Core Definition:** `Object.freeze()` prevents new properties from being added, existing properties from being removed or changed, and the prototype from being changed.

**Technical Definition:** Per ECMA-262 §20.1.2.6, `Object.freeze(O)` sets `[[Extensible]]` to `false` and makes all own properties non-configurable and non-writable (data properties). Accessor properties' getters/setters are not affected. Returns `O`.

**Beginner-Friendly Explanation:** `Object.freeze()` makes an object completely immutable — you can't add, remove, or change anything.

### Purposes

- To create truly immutable objects.
- To enforce constants.
- To prevent accidental mutation.
- To share configuration safely.

### Syntax Rules and Structure

```js
Object.freeze(obj)
```

**Returns:** The frozen object.

**Constraints and Limitations:**
- Shallow: nested objects are not frozen.
- Non-strict mode: assignment fails silently.
- Strict mode: assignment throws `TypeError`.
- Cannot be undone.

### Annotated Code Examples

```js
const config = Object.freeze({ host: "localhost", port: 3000 });

config.port = 8080; // Silent failure (non-strict)
console.log(config.port); // 3000

config.newProp = "x";
console.log(config.newProp); // undefined

delete config.host;
console.log(config.host); // "localhost"

"use strict";
try {
  config.port = 8080;
} catch (err) {
  console.log(err.name); // "TypeError"
}
```

**Expected Output:**
```
3000
undefined
localhost
TypeError
```

**Why This Output:** Frozen objects reject all modifications; strict mode throws.

### Real-World Cases

- **Configuration constants:** Locking API endpoints.
- **Enum-like objects:** Defining constant sets.
- **Redux state:** Ensuring state immutability (with libraries).

---

## `Object.isFrozen()`

### Definitions

**Core Definition:** `Object.isFrozen()` determines whether an object is frozen.

**Technical Definition:** Per ECMA-262 §20.1.2.13, `Object.isFrozen(O)` returns `true` if `O` is not extensible, all own properties are non-configurable, and all data properties are non-writable.

**Beginner-Friendly Explanation:** `Object.isFrozen()` tells you if an object has been frozen.

### Purposes

- To check whether an object can be modified.
- To assert immutability in tests.
- To guard against mutation attempts.

### Syntax Rules and Structure

```js
Object.isFrozen(obj)
```

**Returns:** Boolean.

### Annotated Code Examples

```js
const frozen = Object.freeze({ a: 1 });
const notFrozen = { b: 2 };

console.log(Object.isFrozen(frozen));    // true
console.log(Object.isFrozen(notFrozen)); // false

// Empty objects are considered frozen if non-extensible.
const empty = Object.preventExtensions({});
console.log(Object.isFrozen(empty)); // true
```

**Expected Output:**
```
true
false
true
```

**Why This Output:** `isFrozen` returns `true` for non-extensible objects with no writable or configurable properties.

### Real-World Cases

- **Tests:** Asserting an object is immutable.
- **Library APIs:** Verifying configuration is locked.
- **Debugging:** Diagnosing why mutations fail.

---

## `Object.seal()`

### Definitions

**Core Definition:** `Object.seal()` prevents new properties from being added and existing properties from being removed, but allows existing properties to be changed.

**Technical Definition:** Per ECMA-262 §20.1.2.20, `Object.seal(O)` sets `[[Extensible]]` to `false` and marks all own properties as non-configurable. Writable data properties remain writable.

**Beginner-Friendly Explanation:** `Object.seal()` locks the shape of an object — you can't add or remove properties, but you can change existing values.

### Purposes

- To lock an object's structure while allowing value updates.
- To prevent accidental property addition.
- To enforce a fixed schema.

### Syntax Rules and Structure

```js
Object.seal(obj)
```

**Returns:** The sealed object.

**Constraints and Limitations:**
- Non-configurable but writable properties can still be changed.
- Cannot add or delete properties.
- Cannot be unsealed.

### Annotated Code Examples

```js
const user = Object.seal({ name: "Alice", age: 30 });

user.age = 31;          // Allowed (writable)
console.log(user.age);  // 31

user.email = "x";       // Silently fails
console.log(user.email); // undefined

delete user.name;       // Silently fails
console.log(user.name); // "Alice"
```

**Expected Output:**
```
31
undefined
Alice
```

**Why This Output:** Sealed objects allow value updates but prevent structural changes.

### Real-World Cases

- **Fixed-schema objects:** User profiles with known fields.
- **API contracts:** Objects that should not gain new properties.
- **State objects:** Allowing value updates but not shape changes.

---

## `Object.isSealed()`

### Definitions

**Core Definition:** `Object.isSealed()` determines whether an object is sealed.

**Technical Definition:** Per ECMA-262 §20.1.2.15, `Object.isSealed(O)` returns `true` if `O` is not extensible and all own properties are non-configurable.

**Beginner-Friendly Explanation:** `Object.isSealed()` tells you if an object has been sealed.

### Purposes

- To check whether an object's structure is locked.
- To assert schema integrity.

### Syntax Rules and Structure

```js
Object.isSealed(obj)
```

**Returns:** Boolean.

### Annotated Code Examples

```js
const sealed = Object.seal({ a: 1 });
const frozen = Object.freeze({ b: 2 });
const normal = { c: 3 };

console.log(Object.isSealed(sealed));  // true
console.log(Object.isSealed(frozen));  // true (frozen implies sealed)
console.log(Object.isSealed(normal));  // false
```

**Expected Output:**
```
true
true
false
```

**Why This Output:** Frozen objects are also sealed (they meet the sealing criteria).

### Real-World Cases

- **Integrity checks:** Verifying object structure.
- **Library assertions:** Confirming objects are locked.

---

## `Object.preventExtensions()`

### Definitions

**Core Definition:** `Object.preventExtensions()` prevents new properties from being added to an object, but allows existing properties to be changed or deleted.

**Technical Definition:** Per ECMA-262 §20.1.2.19, `Object.preventExtensions(O)` sets `[[Extensible]]` to `false`. Existing properties remain configurable and writable.

**Beginner-Friendly Explanation:** `Object.preventExtensions()` stops you from adding new properties but lets you modify or remove existing ones.

### Purposes

- To prevent accidental property addition.
- To lock an object's shape without freezing values.

### Syntax Rules and Structure

```js
Object.preventExtensions(obj)
```

**Returns:** The object.

### Annotated Code Examples

```js
const obj = Object.preventExtensions({ a: 1 });

obj.a = 100;        // Allowed
console.log(obj.a); // 100

obj.b = 2;          // Silently fails
console.log(obj.b); // undefined

delete obj.a;       // Allowed
console.log(obj.a); // undefined
```

**Expected Output:**
```
100
undefined
undefined
```

**Why This Output:** Prevented-extensions objects allow modification and deletion of existing properties but not addition.

### Real-World Cases

- **Schema enforcement:** Preventing new fields.
- **Security:** Blocking property injection.

---

## `Object.isExtensible()`

### Definitions

**Core Definition:** `Object.isExtensible()` determines whether new properties can be added to an object.

**Technical Definition:** Per ECMA-262 §20.1.2.12, `Object.isExtensible(O)` returns the value of `O.[[Extensible]]`.

**Beginner-Friendly Explanation:** `Object.isExtensible()` tells you if you can add new properties to an object.

### Purposes

- To check whether an object can gain new properties.
- To assert extensibility before adding properties.

### Syntax Rules and Structure

```js
Object.isExtensible(obj)
```

**Returns:** Boolean.

### Annotated Code Examples

```js
const normal = {};
const prevented = Object.preventExtensions({});
const sealed = Object.seal({});
const frozen = Object.freeze({});

console.log(Object.isExtensible(normal));    // true
console.log(Object.isExtensible(prevented)); // false
console.log(Object.isExtensible(sealed));    // false
console.log(Object.isExtensible(frozen));    // false
```

**Expected Output:**
```
true
false
false
false
```

**Why This Output:** `preventExtensions`, `seal`, and `freeze` all make objects non-extensible.

### Real-World Cases

- **Guards:** Checking before adding properties.
- **Debugging:** Diagnosing why property addition fails.

---

## `Object.is()`

### Definitions

**Core Definition:** `Object.is()` determines whether two values are the same value, using the SameValue algorithm.

**Technical Definition:** Per ECMA-262 §20.1.2.10, `Object.is(value1, value2)` returns `true` if the values are the same, with the differences from `===` being that `NaN` equals `NaN` and `+0` does not equal `-0`.

**Beginner-Friendly Explanation:** `Object.is()` is like `===` but with two fixes: `NaN` equals `NaN`, and `+0` is different from `-0`.

### Purposes

- To compare values including `NaN`.
- To distinguish `+0` from `-0`.
- To serve as the comparison used by testing frameworks.

### Syntax Rules and Structure

```js
Object.is(value1, value2)
```

**Returns:** Boolean.

### Annotated Code Examples

```js
console.log(Object.is(25, 25));    // true
console.log(Object.is(NaN, NaN));  // true (=== gives false)
console.log(Object.is(0, -0));     // false (=== gives true)
console.log(Object.is("a", "a"));  // true
console.log(Object.is({}, {}));    // false (different references)
```

**Expected Output:**
```
true
true
false
true
false
```

**Why This Output:** `Object.is` uses SameValue, which treats `NaN` as equal to itself and distinguishes signed zeros.

### Real-World Cases

- **Testing:** Vitest's `toBe` uses `Object.is`.
- **Precise comparisons:** Detecting `-0`.
- **React dependencies:** Uses `Object.is` for prop comparison.

---

## `Object.hasOwn()`

### Definitions

**Core Definition:** `Object.hasOwn()` determines whether an object has a given property as its own (not inherited) property.

**Technical Definition:** Per ECMA-262 §20.1.2.11, `Object.hasOwn(O, P)` returns `true` if `O` has an own property with key `P`, regardless of enumerability. It is the modern replacement for `Object.prototype.hasOwnProperty.call()`.

**Beginner-Friendly Explanation:** `Object.hasOwn()` tells you if a property belongs directly to an object (not inherited from its prototype).

### Purposes

- To safely check property ownership.
- To avoid the pitfalls of `hasOwnProperty` (which can be shadowed).
- To iterate only own properties.

### Syntax Rules and Structure

```js
Object.hasOwn(obj, propertyName)
```

**Returns:** Boolean.

### Annotated Code Examples

```js
const obj = { name: "Alice" };

console.log(Object.hasOwn(obj, "name"));      // true
console.log(Object.hasOwn(obj, "toString"));  // false (inherited)
console.log(Object.hasOwn(obj, "missing"));   // false

// Compared to hasOwnProperty:
const weird = { hasOwnProperty: () => true };
console.log(weird.hasOwnProperty("x"));       // true (shadowed!)
console.log(Object.hasOwn(weird, "x"));       // false (correct)
```

**Expected Output:**
```
true
false
false
true
false
```

**Why This Output:** `Object.hasOwn` is immune to property shadowing, unlike `hasOwnProperty`.

### Real-World Cases

- **Safe iteration:** Filtering own properties.
- **Serialisation:** Checking before copying.
- **Library code:** Reliable ownership testing.

---

## `Object.groupBy()`

### Definitions

**Core Definition:** `Object.groupBy()` groups the elements of an iterable into a new object, keyed by the result of a callback function (ES2024).

**Technical Definition:** Per ECMA-262 §20.1.2.9, `Object.groupBy(items, callbackFn)` returns a null-prototype object whose keys are the string-coerced results of `callbackFn(element, index)`, and whose values are arrays of elements that produced that key.

**Beginner-Friendly Explanation:** `Object.groupBy()` takes a list of items and groups them by whatever category you specify.

### Purposes

- To categorise data by a key.
- To build histograms or grouped views.
- To replace manual grouping logic.

### Syntax Rules and Structure

```js
Object.groupBy(items, callbackFn)
```

**Component Breakdown:**
- `items`: An iterable.
- `callbackFn(element, index)`: Returns the group key.
- Returns: A null-prototype object with group keys and arrays of elements.

**Constraints and Limitations:**
- ES2024 feature; check runtime support.
- Returns a null-prototype object (no inherited methods).
- Keys are coerced to strings.

### Annotated Code Examples

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
console.log(grouped.fruit[0].name);    // "apple"
```

**Expected Output:**
```
2
2
apple
```

**Why This Output:** `groupBy` collects items into arrays keyed by the callback's return value.

### Real-World Cases

- **Dashboard grouping:** Grouping sales by region.
- **Data analysis:** Building histograms.
- **UI categorisation:** Grouping items by type.

---

## `Object.prototype.hasOwnProperty()`

### Definitions

**Core Definition:** `Object.prototype.hasOwnProperty()` returns a boolean indicating whether the object has the specified property as its own (not inherited) property.

**Technical Definition:** Per ECMA-262 §20.1.3.2, `Object.prototype.hasOwnProperty(V)` converts `V` to a property key and returns `true` if the object has an own property with that key. It is inherited by all objects but can be shadowed by an own property of the same name.

**Beginner-Friendly Explanation:** This is the classic way to check if a property belongs to an object directly. `Object.hasOwn()` is now preferred because `hasOwnProperty` can be accidentally overridden.

### Purposes

- To check property ownership (legacy method).
- To filter inherited properties.

### Syntax Rules and Structure

```js
obj.hasOwnProperty(prop)
```

**Returns:** Boolean.

**Constraints and Limitations:**
- Can be shadowed by an own property named `hasOwnProperty`.
- Prefer `Object.hasOwn()` for safety.
- Works with symbols and strings.

### Annotated Code Examples

```js
const obj = { name: "Alice" };
console.log(obj.hasOwnProperty("name"));     // true
console.log(obj.hasOwnProperty("toString")); // false

// Safe usage via call.
const weird = { hasOwnProperty: "shadowed" };
console.log(Object.prototype.hasOwnProperty.call(weird, "hasOwnProperty")); // true
```

**Expected Output:**
```
true
false
true
```

**Why This Output:** `hasOwnProperty` works but can be shadowed; using `Object.hasOwn` or `call` avoids this.

### Real-World Cases

- **Legacy code:** Pre-ES2022 codebases.
- **Safe calls:** `Object.prototype.hasOwnProperty.call(obj, key)`.
- **Libraries:** Compatibility with older environments.

---

## `Object.prototype.toString()`

### Definitions

**Core Definition:** `Object.prototype.toString()` returns a string representing the object, used to determine an object's type.

**Technical Definition:** Per ECMA-262 §20.1.3.6, `Object.prototype.toString()` returns `"[object X]"` where `X` is determined by `Symbol.toStringTag` or the built-in type. This is the most reliable way to check the built-in type of a value.

**Beginner-Friendly Explanation:** `toString()` returns a string like `"[object Array]"` or `"[object Object]"` that tells you what kind of object it is.

### Purposes

- To reliably detect object types.
- To implement custom string representations via `Symbol.toStringTag`.
- To debug object types.

### Syntax Rules and Structure

```js
obj.toString()
Object.prototype.toString.call(value)
```

**Returns:** A string in the format `"[object Type]"`.

### Annotated Code Examples

```js
console.log(Object.prototype.toString.call([]));       // "[object Array]"
console.log(Object.prototype.toString.call({}));        // "[object Object]"
console.log(Object.prototype.toString.call(null));      // "[object Null]"
console.log(Object.prototype.toString.call(undefined)); // "[object Undefined]"
console.log(Object.prototype.toString.call(42));        // "[object Number]"

// Symbol.toStringTag
const custom = { [Symbol.toStringTag]: "Custom" };
console.log(Object.prototype.toString.call(custom));    // "[object Custom]"
```

**Expected Output:**
```
[object Array]
[object Object]
[object Null]
[object Undefined]
[object Number]
[object Custom]
```

**Why This Output:** `toString` returns the built-in type or the value of `Symbol.toStringTag`.

### Real-World Cases

- **Type detection:** Distinguishing arrays from objects.
- **Custom classes:** Setting `Symbol.toStringTag`.
- **Library code:** Reliable type checks.

---

## `Object.prototype.valueOf()`

### Definitions

**Core Definition:** `Object.prototype.valueOf()` returns the primitive value of the specified object.

**Technical Definition:** Per ECMA-262 §20.1.3.7, `Object.prototype.valueOf()` returns the `this` value converted to an object (i.e., the object itself for ordinary objects). Subclasses and wrapper objects override it to return their primitive value.

**Beginner-Friendly Explanation:** `valueOf()` returns the "primitive form" of an object, which JavaScript uses during type coercion.

### Purposes

- To define how an object converts to a primitive.
- To customise type coercion behaviour.
- To extract primitive values from wrapper objects.

### Syntax Rules and Structure

```js
obj.valueOf()
```

**Returns:** A primitive value.

### Annotated Code Examples

```js
// Built-in overrides
console.log((42).valueOf());           // 42
console.log("hello".valueOf());        // "hello"
console.log(new Date().valueOf());     // timestamp

// Custom valueOf
const money = {
  amount: 100,
  valueOf() { return this.amount; },
};
console.log(money + 50); // 150
```

**Expected Output:**
```
42
hello
<timestamp>
150
```

**Why This Output:** `valueOf` is used during numeric coercion. The custom `valueOf` makes `money` behave as a number.

### Real-World Cases

- **Custom numeric types:** Currency, measurements.
- **Date arithmetic:** `date1 - date2` uses `valueOf`.
- **Coercion control:** Defining how objects convert to primitives.

---

## `Object.prototype.isPrototypeOf()`

### Definitions

**Core Definition:** `Object.prototype.isPrototypeOf()` determines whether an object exists in the prototype chain of another object.

**Technical Definition:** Per ECMA-262 §20.1.3.3, `Object.prototype.isPrototypeOf(V)` returns `true` if `this` appears in the prototype chain of `V` (i.e., `V` inherits from `this`).

**Beginner-Friendly Explanation:** `isPrototypeOf()` tells you if one object is an ancestor of another in the prototype chain.

### Purposes

- To check inheritance relationships.
- To verify prototype chains.
- To implement `instanceof`-like checks.

### Syntax Rules and Structure

```js
proto.isPrototypeOf(obj)
```

**Returns:** Boolean.

### Annotated Code Examples

```js
const animal = { type: "animal" };
const dog = Object.create(animal);
const puppy = Object.create(dog);

console.log(animal.isPrototypeOf(dog));    // true
console.log(animal.isPrototypeOf(puppy));  // true (transitive)
console.log(dog.isPrototypeOf(puppy));     // true
console.log(puppy.isPrototypeOf(animal));  // false
```

**Expected Output:**
```
true
true
true
false
```

**Why This Output:** `isPrototypeOf` checks the entire prototype chain, not just the immediate prototype.

### Real-World Cases

- **Inheritance checks:** Verifying class hierarchies.
- **Polyfills:** Emulating `instanceof` behaviour.
- **Debugging:** Understanding prototype chains.

---

## `Object.prototype.propertyIsEnumerable()`

### Definitions

**Core Definition:** `Object.prototype.propertyIsEnumerable()` determines whether a specified property is enumerable and is an own property of the object.

**Technical Definition:** Per ECMA-262 §20.1.3.4, `Object.prototype.propertyIsEnumerable(V)` returns `true` if the object has an own property with key `V` and that property's `[[Enumerable]]` attribute is `true`.

**Beginner-Friendly Explanation:** `propertyIsEnumerable()` tells you if a property is both own and visible in loops.

### Purposes

- To check whether a property appears in `for...in`.
- To filter own enumerable properties.
- To debug serialisation issues.

### Syntax Rules and Structure

```js
obj.propertyIsEnumerable(prop)
```

**Returns:** Boolean.

### Annotated Code Examples

```js
const obj = { visible: 1 };
Object.defineProperty(obj, "hidden", { value: 2, enumerable: false });

console.log(obj.propertyIsEnumerable("visible")); // true
console.log(obj.propertyIsEnumerable("hidden"));  // false
console.log(obj.propertyIsEnumerable("toString")); // false (inherited)
```

**Expected Output:**
```
true
false
false
```

**Why This Output:** `propertyIsEnumerable` returns `true` only for own enumerable properties.

### Real-World Cases

- **Serialisation:** Checking whether a property will appear in JSON.
- **Filtering:** Selecting only enumerable properties.
- **Debugging:** Diagnosing missing properties in loops.

---

## References

- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Object Objects — https://tc39.es/ecma262/multipage/fundamental-objects.html#sec-object-objects
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Properties of the Object Constructor — https://tc39.es/ecma262/multipage/fundamental-objects.html#sec-properties-of-the-object-constructor
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Properties of the Object Prototype Object — https://tc39.es/ecma262/multipage/fundamental-objects.html#sec-properties-of-the-object-prototype-object
- MDN Web Docs – Object.keys() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/keys
- MDN Web Docs – Object.values() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/values
- MDN Web Docs – Object.entries() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/entries
- MDN Web Docs – Object.fromEntries() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/fromEntries
- MDN Web Docs – Object.assign() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign
- MDN Web Docs – Object.freeze() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze
- MDN Web Docs – Object.isFrozen() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/isFrozen
- MDN Web Docs – Object.seal() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/seal
- MDN Web Docs – Object.isSealed() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/isSealed
- MDN Web Docs – Object.preventExtensions() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/preventExtensions
- MDN Web Docs – Object.isExtensible() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/isExtensible
- MDN Web Docs – Object.is() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is
- MDN Web Docs – Object.hasOwn() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/hasOwn
- MDN Web Docs – Object.groupBy() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/groupBy
- MDN Web Docs – Object.prototype.hasOwnProperty() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/hasOwnProperty
- MDN Web Docs – Object.prototype.toString() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/toString
- MDN Web Docs – Object.prototype.valueOf() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/valueOf
- MDN Web Docs – Object.prototype.isPrototypeOf() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/isPrototypeOf
- MDN Web Docs – Object.prototype.propertyIsEnumerable() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/propertyIsEnumerable
- TC39 Proposal – Array Grouping — https://github.com/tc39/proposal-array-grouping
- TC39 Proposal – Object.hasOwn — https://github.com/tc39/proposal-accessible-object-hasownproperty