# JavaScript Array Fundamentals — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** An array is an ordered, indexed collection of values in JavaScript, capable of holding elements of any type and dynamically resizing as needed.

**Technical Definition:** Per ECMAScript 2025 (ECMA-262, §10.4.2), an Array is an exotic object that gives special treatment to array-index property keys. Array indices are integer-valued keys in the range 0 to 2³² − 2. Every Array object has a non-configurable `length` property whose value is always a non-negative integer less than 2³². The `length` property is automatically updated when array elements are added or removed. Arrays are created via array literals, the `Array` constructor, `Array.of()`, `Array.from()`, or methods that return new arrays.

**Beginner-Friendly Explanation:** An array is like a numbered list. Each item in the list has a position (called an index), starting at 0. You can add, remove, and access items by their position. Arrays can hold anything — numbers, strings, objects, even other arrays.

### Key Characteristics

- **Ordered:** Elements are stored in a specific order and accessed by index.
- **Zero-indexed:** The first element is at index `0`.
- **Dynamic length:** Arrays grow and shrink automatically.
- **Heterogeneous:** Can hold mixed types.
- **Exotic object:** Special treatment of integer-indexed keys.
- **Reference type:** Assigned and passed by reference.
- **Iterable:** Supports `for...of`, spread, and destructuring.

### Prerequisites

- Basic JavaScript syntax (variables, values, types).
- Understanding of objects and property access.
- Familiarity with `typeof` and value types.
- Awareness of reference semantics.
- Basic knowledge of loops and iteration.

### Related Programming Areas

- **Data structures:** Lists, stacks, queues, matrices.
- **Functional programming:** `map`, `filter`, `reduce`.
- **DOM manipulation:** `querySelectorAll` returns NodeList (array-like).
- **JSON:** Arrays serialise to JSON arrays.
- **TypedArrays:** Fixed-type numeric arrays.
- **Iteration protocols:** Arrays are iterable.

---

## Arrays

### Definitions

**Core Definition:** An array is an ordered collection of values, each identified by a numeric index.

**Technical Definition:** Per ECMA-262 §10.4.2, an Array exotic object has a `length` property and treats integer-indexed keys specially. Setting `length` truncates or extends the array. Array elements are own properties with integer-like string keys. Arrays inherit from `Array.prototype`, which provides methods like `push`, `pop`, `map`, `filter`, and `reduce`.

**Beginner-Friendly Explanation:** An array is a container that holds a list of items in order. You can access items by their position (index) and use built-in methods to add, remove, or transform them.

### Purposes

- To store ordered lists of values.
- To represent sequences, queues, stacks, and matrices.
- To enable iteration and transformation.
- To hold collections of related data.

### Syntax Rules and Structure

```js
const arr = [1, 2, 3];
const empty = [];
const mixed = [1, "two", true, null, { a: 1 }, [2, 3]];
```

**Constraints and Limitations:**
- Arrays are reference types.
- `typeof []` returns `"object"` (use `Array.isArray()`).
- Indices are coerced to strings internally.
- Maximum length is 2³² − 1.

### Annotated Code Examples

```js
// Step 1: Create an array.
const fruits = ["apple", "banana", "cherry"];

// Step 2: Access elements.
console.log(fruits[0]); // "apple"
console.log(fruits[2]); // "cherry"
console.log(fruits.length); // 3

// Step 3: Modify elements.
fruits[1] = "blueberry";
console.log(fruits); // ["apple", "blueberry", "cherry"]
```

**Expected Output:**
```
apple
cherry
3
[ 'apple', 'blueberry', 'cherry' ]
```

**Why This Output:** Arrays are ordered, zero-indexed collections; elements can be read and written by index.

### Real-World Cases

- **Todo lists:** Ordered tasks.
- **Shopping carts:** Items and quantities.
- **API responses:** Arrays of records.
- **Game state:** Entities, positions.

---

## Array Literals

### Definitions

**Core Definition:** Array literals are the `[ ]` syntax used to create arrays with initial elements.

**Technical Definition:** Per ECMA-262 §13.2.4, an `ArrayLiteral` consists of zero or more `AssignmentExpression` elements separated by commas, enclosed in square brackets. Trailing commas are allowed and do not create extra elements. Elisions (consecutive commas) create holes in sparse arrays.

**Beginner-Friendly Explanation:** Array literals are the simplest way to create an array — just list the items inside `[ ]`.

### Purposes

- To create arrays concisely.
- To initialise arrays with known values.
- To create nested or multi-dimensional arrays.

### Syntax Rules and Structure

```js
const arr = [1, 2, 3];
const trailing = [1, 2, 3,]; // Same as [1, 2, 3]
const nested = [[1, 2], [3, 4]];
const sparse = [1, , 3]; // Hole at index 1
```

**Constraints and Limitations:**
- Elisions create sparse arrays (holes).
- A trailing comma is ignored.
- The last element cannot be a hole: `[1, 2, ,]` has length 3 (not 4).

### Annotated Code Examples

```js
// Step 1: Basic literal.
const nums = [1, 2, 3];
console.log(nums.length); // 3

// Step 2: Trailing comma.
const withTrailing = [1, 2, 3,];
console.log(withTrailing.length); // 3

// Step 3: Nested.
const matrix = [[1, 2], [3, 4]];
console.log(matrix[1][0]); // 3

// Step 4: Sparse (hole).
const sparse = [1, , 3];
console.log(sparse.length); // 3
console.log(1 in sparse);   // false
```

**Expected Output:**
```
3
3
3
3
false
```

**Why This Output:** Trailing commas are ignored; elisions create holes with `length` counting them.

### Real-World Cases

- **Configuration:** `const hosts = ["a.example.com", "b.example.com"];`.
- **Static data:** `const days = ["Sun", "Mon", ...];`.
- **Matrices:** `[[1, 0], [0, 1]]`.

---

## `Array` Constructor

### Definitions

**Core Definition:** The `Array` constructor creates arrays via `new Array(...)` or `Array(...)`.

**Technical Definition:** Per ECMA-262 §20.4.1, `Array(len)` with a single number argument creates an array of that length with empty slots. `Array(...items)` with multiple arguments creates an array containing those items. `Array` can be called with or without `new`; both produce array instances.

**Beginner-Friendly Explanation:** The `Array` constructor is an older way to create arrays. It has a quirk: `Array(3)` creates an empty array of length 3, but `Array(3, 4)` creates `[3, 4]`.

### Purposes

- To create arrays with a specified length.
- To create arrays programmatically.
- To understand legacy array creation.

### Syntax Rules and Structure

```js
new Array()
new Array(length)
new Array(element0, element1, ...)
Array() // Same as new Array()
```

**Syntax Rules:**
- Single numeric argument: creates an empty array of that length.
- Multiple arguments: creates an array with those elements.
- Single non-numeric argument: creates an array with one element.
- `Array()` with no arguments: creates an empty array.

**Constraints and Limitations:**
- `Array(3)` does NOT create `[3]`; it creates `[ <3 empty items> ]`.
- `Array(-1)` throws `RangeError`.
- `Array(3.5)` throws `RangeError` (not a valid length).

### Annotated Code Examples

```js
// Step 1: Empty array.
console.log(new Array()); // []

// Step 2: Single number = length.
const empty3 = new Array(3);
console.log(empty3.length); // 3
console.log(empty3);        // [ <3 empty items> ]

// Step 3: Multiple arguments = elements.
const arr = new Array(3, 4);
console.log(arr); // [3, 4]

// Step 4: Single non-number = element.
const single = new Array("3");
console.log(single); // ["3"]

// Step 5: RangeError.
try {
  new Array(-1);
} catch (err) {
  console.log(err.name); // "RangeError"
}
```

**Expected Output:**
```
[]
3
[ <3 empty items> ]
[ 3, 4 ]
[ '3' ]
RangeError
```

**Why This Output:** The single-number form creates a sparse array of the given length; multiple arguments create a normal array.

### Real-World Cases

- **Preallocation:** `new Array(n)` for fixed-size loops.
- **Legacy code:** Older codebases.
- **Avoidance:** Prefer `[]` or `Array.of()`.

---

## `Array.of()`

### Definitions

**Core Definition:** `Array.of()` creates an array from its arguments, regardless of their number or type.

**Technical Definition:** Per ECMA-262 §20.4.2.3, `Array.of(...items)` returns a new array whose elements are the arguments in order. Unlike the `Array` constructor, `Array.of(3)` creates `[3]`, not an empty array of length 3.

**Beginner-Friendly Explanation:** `Array.of()` is the "fixed" version of the `Array` constructor. It does what you'd expect: `Array.of(3)` creates `[3]`.

### Purposes

- To create arrays from arguments without the `Array` constructor's quirk.
- To convert array-like arguments into real arrays.
- To provide consistent behaviour.

### Syntax Rules and Structure

```js
Array.of(element0, element1, ...)
```

**Returns:** A new array.

### Annotated Code Examples

```js
console.log(Array.of());       // []
console.log(Array.of(3));      // [3]
console.log(Array.of(3, 4));   // [3, 4]
console.log(Array.of("3"));    // ["3"]
console.log(Array.of(null));   // [null]

// Comparison with Array constructor.
console.log(new Array(3));     // [ <3 empty items> ]
console.log(Array.of(3));      // [3]
```

**Expected Output:**
```
[]
[ 3 ]
[ 3, 4 ]
[ '3' ]
[ null ]
[ <3 empty items> ]
[ 3 ]
```

**Why This Output:** `Array.of` always treats its arguments as elements, unlike the `Array` constructor.

### Real-World Cases

- **Safe array creation:** Avoiding the single-number quirk.
- **Functional programming:** `Array.of(...args)`.
- **Polyfills:** Available since ES2015.

---

## `Array.from()`

### Definitions

**Core Definition:** `Array.from()` creates a new array from an iterable or array-like object, with an optional mapping function.

**Technical Definition:** Per ECMA-262 §20.4.2.1, `Array.from(items, mapFn, thisArg)` iterates the source (using the iterator protocol if available, or array-like indexing otherwise), applies `mapFn` to each element, and returns a new array.

**Beginner-Friendly Explanation:** `Array.from()` converts things that look like arrays (but aren't) into real arrays. It works on strings, Sets, Maps, NodeLists, and any iterable.

### Purposes

- To convert iterables to arrays.
- To convert array-like objects to arrays.
- To map values during conversion.
- To create arrays from strings, Sets, Maps, and generators.

### Syntax Rules and Structure

```js
Array.from(items)
Array.from(items, mapFn)
Array.from(items, mapFn, thisArg)
```

**Component Breakdown:**
- `items`: An iterable or array-like object.
- `mapFn` (optional): Mapping function `(element, index) => newValue`.
- `thisArg` (optional): `this` for `mapFn`.

### Annotated Code Examples

```js
// Step 1: From a string.
console.log(Array.from("hello")); // ["h", "e", "l", "l", "o"]

// Step 2: From a Set.
const set = new Set([1, 2, 2, 3]);
console.log(Array.from(set)); // [1, 2, 3]

// Step 3: From a Map.
const map = new Map([["a", 1], ["b", 2]]);
console.log(Array.from(map)); // [["a", 1], ["b", 2]]

// Step 4: With a mapping function.
console.log(Array.from([1, 2, 3], (x) => x * 2)); // [2, 4, 6]

// Step 5: From array-like (length + indices).
const arrayLike = { 0: "a", 1: "b", length: 2 };
console.log(Array.from(arrayLike)); // ["a", "b"]

// Step 6: Generating a range.
console.log(Array.from({ length: 5 }, (_, i) => i)); // [0, 1, 2, 3, 4]
```

**Expected Output:**
```
[ 'h', 'e', 'l', 'l', 'o' ]
[ 1, 2, 3 ]
[ [ 'a', 1 ], [ 'b', 2 ] ]
[ 2, 4, 6 ]
[ 'a', 'b' ]
[ 0, 1, 2, 3, 4 ]
```

**Why This Output:** `Array.from` handles iterables and array-likes, and the mapping function transforms elements during creation.

### Real-World Cases

- **NodeList conversion:** `Array.from(document.querySelectorAll("div"))`.
- **Set/Map to array:** `Array.from(mySet)`.
- **Range generation:** `Array.from({ length: n }, (_, i) => i)`.
- **String manipulation:** `Array.from(str)`.

---

## Array Length

### Definitions

**Core Definition:** The `length` property of an array indicates the number of elements and is automatically updated as elements are added or removed.

**Technical Definition:** Per ECMA-262 §10.4.2.4, `length` is a non-configurable, writable data property whose value is always a non-negative integer less than 2³². Setting `length` to a smaller value truncates the array; setting it to a larger value creates empty slots.

**Beginner-Friendly Explanation:** `length` tells you how many items are in the array. Assigning to `length` can add or remove elements.

### Purposes

- To determine the number of elements.
- To truncate or extend an array.
- To loop over elements.

### Syntax Rules and Structure

```js
arr.length           // Read
arr.length = n       // Write (truncate or extend)
```

**Constraints and Limitations:**
- Setting `length` to a non-integer or negative throws `RangeError`.
- Truncating removes elements permanently.
- Extending creates holes (sparse).

### Annotated Code Examples

```js
const arr = [1, 2, 3, 4, 5];

// Step 1: Read length.
console.log(arr.length); // 5

// Step 2: Truncate.
arr.length = 3;
console.log(arr); // [1, 2, 3]

// Step 3: Extend (creates holes).
arr.length = 5;
console.log(arr); // [1, 2, 3, <2 empty items>]

// Step 4: Invalid assignment.
try {
  arr.length = -1;
} catch (err) {
  console.log(err.name); // "RangeError"
}
```

**Expected Output:**
```
5
[ 1, 2, 3 ]
[ 1, 2, 3, <2 empty items> ]
RangeError
```

**Why This Output:** `length` is writable and reflects element count; invalid values throw.

### Real-World Cases

- **Clearing an array:** `arr.length = 0`.
- **Loop bounds:** `for (let i = 0; i < arr.length; i++)`.
- **Preallocation:** `arr.length = n`.

---

## Array Indexing

### Definitions

**Core Definition:** Array indexing is the use of numeric indices (starting at 0) to access or modify elements.

**Technical Definition:** Per ECMA-262, array indices are integer-valued property keys in the range 0 to 2³² − 2. Accessing `arr[i]` performs a property lookup with the key `String(i)`. Out-of-bounds access returns `undefined`.

**Beginner-Friendly Explanation:** Arrays are zero-indexed. `arr[0]` is the first element, `arr[1]` is the second, and so on.

### Purposes

- To read or write individual elements.
- To iterate over elements by index.
- To access nested arrays.

### Syntax Rules and Structure

```js
arr[0]         // Read
arr[0] = 42    // Write
arr[arr.length - 1] // Last element
```

**Constraints and Limitations:**
- Out-of-bounds access returns `undefined`.
- Negative indices are not supported (use `arr.at(-1)`).
- Non-integer indices create properties, not elements.

### Annotated Code Examples

```js
const arr = ["a", "b", "c"];

// Step 1: Read by index.
console.log(arr[0]); // "a"
console.log(arr[2]); // "c"
console.log(arr[3]); // undefined

// Step 2: Write by index.
arr[1] = "B";
console.log(arr); // ["a", "B", "c"]

// Step 3: Negative index (not supported).
console.log(arr[-1]); // undefined

// Step 4: Use .at() for negative indexing.
console.log(arr.at(-1)); // "c"
console.log(arr.at(-2)); // "B"
```

**Expected Output:**
```
a
c
undefined
[ 'a', 'B', 'c' ]
undefined
c
B
```

**Why This Output:** Indices start at 0; negative indices are property keys, not array positions; `.at()` supports negatives.

### Real-World Cases

- **First/last element:** `arr[0]`, `arr.at(-1)`.
- **Iteration:** `for (let i = 0; i < arr.length; i++)`.
- **Nested access:** `matrix[row][col]`.

---

## Sparse Arrays

### Definitions

**Core Definition:** A sparse array is an array with "holes" — indices that have no element.

**Technical Definition:** Per ECMA-262, a hole is an array index that is not an own property of the array. Sparse arrays arise from elisions in literals, `Array(n)`, `delete arr[i]`, or setting `length` larger than the current count. Most array methods skip holes.

**Beginner-Friendly Explanation:** A sparse array is like a list with some missing slots. The length still counts them, but there's nothing there.

### Purposes (and Cautions)

- **Anti-purpose:** Sparse arrays are usually unintentional and can cause bugs.
- To understand why `forEach` skips some elements.
- To recognise when `Array(n)` creates sparse arrays.

### Syntax Rules and Structure

```js
const sparse = [1, , 3];       // Hole at index 1
const sparse2 = new Array(3);  // 3 holes
const arr = [1, 2, 3];
delete arr[1];                 // Hole at index 1
```

**Constraints and Limitations:**
- `forEach`, `map`, `filter`, `some`, `every` skip holes.
- `for...in` skips holes.
- `for...of` treats holes as `undefined`.
- `JSON.stringify` converts holes to `null`.

### Annotated Code Examples

```js
const sparse = [1, , 3];

// Step 1: Length includes holes.
console.log(sparse.length); // 3

// Step 2: Check for holes.
console.log(1 in sparse); // false

// Step 3: Methods skip holes.
sparse.forEach((v, i) => console.log(i, v)); // "0 1" and "2 3"

// Step 4: for...of treats holes as undefined.
for (const v of sparse) console.log(v); // 1, undefined, 3

// Step 5: JSON converts holes to null.
console.log(JSON.stringify(sparse)); // "[1,null,3]"
```

**Expected Output:**
```
3
false
0 1
2 3
1
undefined
3
[1,null,3]
```

**Why This Output:** Holes are not own properties; different iteration methods handle them differently.

### Real-World Cases

- **Bug source:** Unexpected `undefined` in arrays.
- **JSON:** Holes become `null`.
- **Serialisation:** Prefer `Array.from({ length: n })` for dense arrays.

---

## Array-Like Objects

### Definitions

**Core Definition:** Array-like objects are objects that have a `length` property and indexed elements but do not inherit from `Array.prototype`.

**Technical Definition:** Per ECMA-262, an array-like object has a non-negative integer `length` property and indexed access. Examples include `arguments`, `NodeList`, `HTMLCollection`, and strings. Array-likes are not iterable by default (unless they implement `Symbol.iterator`), but `Array.from()` and spread can convert them.

**Beginner-Friendly Explanation:** Array-like objects look like arrays (they have a length and indices) but don't have array methods like `push` or `map`.

### Purposes (and Conversion)

- To understand DOM collections and `arguments`.
- To convert to real arrays for method access.
- To recognise when array methods are unavailable.

### Syntax Rules and Structure

```js
// Example array-like
const arrayLike = { 0: "a", 1: "b", length: 2 };

// Convert to array
const arr1 = Array.from(arrayLike);
const arr2 = [...arrayLike]; // Only if iterable
const arr3 = Array.prototype.slice.call(arrayLike);
```

**Constraints and Limitations:**
- No array methods.
- Not iterable unless they implement `Symbol.iterator`.
- `arguments` is array-like but not an array.

### Annotated Code Examples

```js
// Step 1: Array-like from arguments.
function sum() {
  console.log(arguments.length); // 3
  // arguments.map is undefined
  const args = Array.from(arguments);
  return args.reduce((a, b) => a + b, 0);
}
console.log(sum(1, 2, 3)); // 6

// Step 2: DOM NodeList.
// const nodes = document.querySelectorAll("div"); // NodeList
// const arr = Array.from(nodes); // Real array

// Step 3: Check array-likeness.
function isArrayLike(obj) {
  return obj != null && typeof obj.length === "number" && obj.length >= 0;
}
console.log(isArrayLike({ 0: "a", length: 1 })); // true
console.log(isArrayLike([1, 2]));                // true
console.log(isArrayLike("hello"));               // true
```

**Expected Output:**
```
3
6
true
true
true
```

**Why This Output:** `arguments` is array-like; `Array.from` converts it; strings are array-like.

### Real-World Cases

- **`arguments`:** Legacy function arguments object.
- **DOM:** `NodeList`, `HTMLCollection`.
- **Strings:** Array-like with `length` and indices.
- **jQuery objects:** Array-like wrappers.

---

## Array Detection

### Definitions

**Core Definition:** Array detection is the process of determining whether a value is an array.

**Technical Definition:** Per ECMA-262 §7.2.2, `Array.isArray(value)` returns `true` if `value` is an Array exotic object. It is the recommended way to detect arrays, as it works across realms (iframes) and is not fooled by `typeof` or `instanceof`.

**Beginner-Friendly Explanation:** To check if something is an array, use `Array.isArray()`. Don't use `typeof` (it returns `"object"`) or `instanceof` (it fails across iframes).

### Purposes

- To verify a value is an array before using array methods.
- To handle polymorphic inputs.
- To safely process external data.

### Syntax Rules and Structure

```js
Array.isArray(value)
```

**Returns:** Boolean.

**Constraints and Limitations:**
- `Array.isArray` cannot be fooled by `Symbol.toStringTag` spoofing.
- `instanceof Array` fails across realms.
- `typeof []` returns `"object"`.

### Annotated Code Examples

```js
// Step 1: Basic detection.
console.log(Array.isArray([]));        // true
console.log(Array.isArray([1, 2, 3])); // true
console.log(Array.isArray("hello"));   // false
console.log(Array.isArray({ length: 0 })); // false

// Step 2: typeof is misleading.
console.log(typeof []); // "object"

// Step 3: instanceof works within a realm.
console.log([] instanceof Array); // true

// Step 4: Array.isArray across realms (conceptually).
// In a browser, an array from an iframe:
// iframeArray instanceof Array === false
// Array.isArray(iframeArray) === true

// Step 5: Proxy-wrapped arrays.
const proxy = new Proxy([], {});
console.log(Array.isArray(proxy)); // true
```

**Expected Output:**
```
true
true
false
false
object
true
true
```

**Why This Output:** `Array.isArray` reliably detects arrays across realms and through proxies.

### Real-World Cases

- **API validation:** Checking if a response field is an array.
- **Utility functions:** Handling both single values and arrays.
- **Cross-realm code:** Iframes and workers.
- **Framework internals:** React, Vue check for arrays.

---

## References

- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Array Exotic Objects — https://tc39.es/ecma262/multipage/ordinary-and-exotic-objects-behaviours.html#sec-array-exotic-objects
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Array Objects — https://tc39.es/ecma262/multipage/indexed-collections.html#sec-array-objects
- MDN Web Docs – Array — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array
- MDN Web Docs – Array() constructor — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Array
- MDN Web Docs – Array.of() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/of
- MDN Web Docs – Array.from() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/from
- MDN Web Docs – Array.isArray() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/isArray
- MDN Web Docs – Array: length — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/length
- MDN Web Docs – Indexed collections — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Indexed_collections
- MDN Web Docs – Array.prototype.at() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/at
- MDN Web Docs – Spread syntax — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax
- MDN Web Docs – Array-like objects — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array#array-like_objects
- Exploring JS – Arrays — https://exploringjs.com/impatient-js/ch_arrays.html
- You Don't Know JS – Types & Grammar (Arrays) — https://github.com/getify/You-Dont-Know-JS/blob/2nd-ed/types-grammar/ch3.md
- V8 Blog – Elements Kinds in V8 — https://v8.dev/blog/elements-kinds
- Web.dev – Array methods — https://web.dev/learn/javascript/arrays