# JavaScript Array Spread — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Array spread is the use of the `...` syntax to expand an iterable's elements into individual values, enabling concise array copying, merging, and function argument passing.

**Technical Definition:** Per ECMAScript 2025 (ECMA-262, §13.2.4.1), the `SpreadElement` production `... AssignmentExpression` in an `ArrayLiteral` and in an `ArgumentList` invokes `GetIterator` on the operand. When used in an array literal, each yielded value is added as an element of the new array via `CreateDataPropertyOrThrow`. When used in a function call, each yielded value is passed as a separate argument. Spread requires an iterable operand; non-iterable operands (including plain objects in array context) throw `TypeError`.

**Beginner-Friendly Explanation:** Spread is the `...` "three dots" syntax that "spreads out" the contents of an array (or any iterable) into individual values. Instead of writing `Math.max(arr[0], arr[1], arr[2])`, you can write `Math.max(...arr)`. Instead of concatenating arrays with `concat()`, you can write `[...a, ...b]`. It's like opening a box and laying out all the items individually.

### Key Characteristics

- **Unpacks iterables:** Works on arrays, strings, Sets, Maps, generators, and other iterables.
- **Creates new arrays:** Spread in an array literal produces a new array.
- **Shallow copy:** Only the top-level elements are copied; nested objects are shared by reference.
- **Position-independent:** Can appear anywhere in an array literal or argument list.
- **Requires iterable:** Non-iterable operands throw `TypeError`.
- **Different from rest:** Spread expands values; rest collects them.

### Prerequisites

- Basic JavaScript syntax (arrays, functions, arguments).
- Understanding of array literals and indexing.
- Familiarity with functions and function calls.
- Awareness of the iterator protocol.
- Basic knowledge of reference semantics.

### Related Programming Areas

- **Function calls:** Spreading arguments.
- **Rest parameters:** The complementary collecting syntax.
- **Object spread:** The object-literal counterpart.
- **Array methods:** `concat`, `push`, `apply`.
- **React:** Spreading props and children.
- **Iteration:** Spread relies on the iterator protocol.

---

## Spread in Arrays

### Definitions

**Core Definition:** Spread in arrays expands an iterable into individual elements within a new array literal.

**Technical Definition:** Per ECMA-262 §13.2.4.1, when an `ArrayLiteral` contains a `SpreadElement`, the array being constructed has elements from each spread source appended in order. The spread operand's iterator is consumed, and each yielded value is added via `CreateDataPropertyOrThrow`.

**Beginner-Friendly Explanation:** Spread inside `[ ]` "opens up" an array and inserts its elements into a new array.

### Purposes

- To copy arrays concisely.
- To merge multiple arrays.
- To insert elements at specific positions.
- To convert iterables to arrays.

### Syntax Rules and Structure

```js
const newArr = [...source];
const merged = [...arr1, ...arr2];
const withExtra = [0, ...arr, 99];
```

**Component Breakdown:**
- `...`: The spread operator.
- `source`: Any iterable.
- `[ ]`: The new array literal.
- Spread elements can appear anywhere in the literal.

**Syntax Rules:**
- Operand must be iterable; otherwise `TypeError`.
- Spread elements can appear multiple times in one literal.
- Order is preserved.
- Creates a new array; the source is unchanged.
- Nested objects are shared (shallow copy).

**Constraints and Limitations:**
- Shallow copy: nested objects are shared.
- Requires iterable; `{...null}` (object spread) is allowed, but `[...null]` throws.
- Large arrays have memory overhead for the new array.

### Annotated Code Examples

#### Example 1: Basic Spread

```js
// Step 1: Define a source array.
const original = [1, 2, 3];

// Step 2: Spread into a new array.
const copy = [...original];

console.log(copy);             // [1, 2, 3]
console.log(copy === original); // false (new array)
```

**Expected Output:**
```
[ 1, 2, 3 ]
false
```

**Why This Output:** Spread creates a new array with the same elements; the two arrays are distinct references.

#### Example 2: Spread at Multiple Positions

```js
const arr = [2, 3, 4];
const result = [1, ...arr, 5];
console.log(result); // [1, 2, 3, 4, 5]
```

**Expected Output:**
```
[ 1, 2, 3, 4, 5 ]
```

**Why This Output:** Spread elements are inserted in place, allowing values before and after.

#### Example 3: Spreading Strings

```js
const chars = [..."hello"];
console.log(chars); // ["h", "e", "l", "l", "o"]
```

**Expected Output:**
```
[ 'h', 'e', 'l', 'l', 'o' ]
```

**Why This Output:** Strings are iterable, so spread extracts individual characters.

### Real-World Cases

- **Cloning arrays:** `const copy = [...original];`.
- **Immutable updates:** `const newArr = [...state.items, newItem];`.
- **Building arrays from parts:** `[...header, ...body, ...footer]`.

---

## Copying Arrays

### Definitions

**Core Definition:** Copying an array with spread creates a new array with the same top-level elements as the source.

**Technical Definition:** Per ECMA-262, `[...source]` iterates the source and creates a new array with each yielded value. The copy is shallow: primitive elements are copied by value; object elements are copied by reference. The copy has the same length and elements as the source but is a distinct object.

**Beginner-Friendly Explanation:** Spread creates a "photocopy" of an array. The top-level items are copied, but if an item is itself an object, both the original and the copy point to the same object.

### Purposes

- To create an independent copy of an array.
- To avoid mutating shared arrays.
- To create immutable updates.
- To snapshot an array's current state.

### Syntax Rules and Structure

```js
const copy = [...original];
```

**Constraints and Limitations:**
- Shallow copy: nested objects are shared.
- For deep copies, use `structuredClone()`.
- Does not copy custom properties on the array.
- Sparse arrays become dense (holes become `undefined`).

### Annotated Code Examples

#### Example 1: Shallow Copy

```js
const original = [1, 2, 3];
const copy = [...original];

copy.push(4);
console.log(original); // [1, 2, 3] (unchanged)
console.log(copy);     // [1, 2, 3, 4]
```

**Expected Output:**
```
[ 1, 2, 3 ]
[ 1, 2, 3, 4 ]
```

**Why This Output:** The copy is a new array; modifying it does not affect the original.

#### Example 2: Shallow Copy Pitfall

```js
const original = [{ id: 1 }, { id: 2 }];
const copy = [...original];

copy[0].id = 99;
console.log(original[0].id); // 99 (shared reference!)
console.log(copy[0] === original[0]); // true
```

**Expected Output:**
```
99
true
```

**Why This Output:** Spread copies the references to nested objects, not the objects themselves.

#### Example 3: Deep Copy with `structuredClone`

```js
const original = [{ id: 1 }, { id: 2 }];
const deepCopy = structuredClone(original);

deepCopy[0].id = 99;
console.log(original[0].id); // 1 (unchanged)
console.log(deepCopy[0].id); // 99
```

**Expected Output:**
```
1
99
```

**Why This Output:** `structuredClone` performs a deep copy, so nested objects are independent.

### Real-World Cases

- **Immutable state:** React/Redux state updates.
- **Snapshots:** Saving a copy before mutation.
- **Function returns:** Returning a copy to prevent external mutation.

---

## Merging Arrays

### Definitions

**Core Definition:** Merging arrays with spread combines elements from multiple arrays into a single new array.

**Technical Definition:** Per ECMA-262, when multiple spread elements appear in an array literal, they are evaluated left to right, and their yielded values are appended in order. The result is a new array containing all elements from all sources.

**Beginner-Friendly Explanation:** Spread lets you "glue" arrays together: `[...a, ...b]` creates a new array with all of `a`'s elements followed by all of `b`'s.

### Purposes

- To combine arrays without mutating them.
- To concatenate with additional elements.
- To build arrays from multiple sources.
- To implement immutable append operations.

### Syntax Rules and Structure

```js
const merged = [...arr1, ...arr2];
const merged3 = [...arr1, ...arr2, ...arr3];
const withExtras = [0, ...arr1, ...arr2, 99];
```

**Syntax Rules:**
- Sources are evaluated left to right.
- Duplicates are preserved (spread does not deduplicate).
- Order is preserved.
- Creates a new array.

**Constraints and Limitations:**
- Duplicates are not removed (use `Set` for deduplication).
- Large merges allocate a new array.

### Annotated Code Examples

#### Example 1: Merging Two Arrays

```js
const a = [1, 2, 3];
const b = [4, 5, 6];

const merged = [...a, ...b];
console.log(merged); // [1, 2, 3, 4, 5, 6]
console.log(a);      // [1, 2, 3] (unchanged)
console.log(b);      // [4, 5, 6] (unchanged)
```

**Expected Output:**
```
[ 1, 2, 3, 4, 5, 6 ]
[ 1, 2, 3 ]
[ 4, 5, 6 ]
```

**Why This Output:** Spread creates a new array with all elements from both sources.

#### Example 2: Merging with Extras

```js
const head = [1, 2];
const tail = [4, 5];
const full = [0, ...head, 3, ...tail, 6];
console.log(full); // [0, 1, 2, 3, 4, 5, 6]
```

**Expected Output:**
```
[ 0, 1, 2, 3, 4, 5, 6 ]
```

**Why This Output:** Spread elements and explicit elements can be freely mixed.

#### Example 3: Deduplicating While Merging

```js
const a = [1, 2, 3];
const b = [2, 3, 4];

const unique = [...new Set([...a, ...b])];
console.log(unique); // [1, 2, 3, 4]
```

**Expected Output:**
```
[ 1, 2, 3, 4 ]
```

**Why This Output:** Spread merges; `Set` deduplicates; spread converts back to an array.

#### Example 4: Comparison with `concat`

```js
const a = [1, 2];
const b = [3, 4];

// Spread
const spreadMerge = [...a, ...b];

// concat
const concatMerge = a.concat(b);

console.log(spreadMerge); // [1, 2, 3, 4]
console.log(concatMerge); // [1, 2, 3, 4]
```

**Expected Output:**
```
[ 1, 2, 3, 4 ]
[ 1, 2, 3, 4 ]
```

**Why This Output:** Both produce equivalent results for simple merges. Spread is preferred in modern code for consistency with object spread.

### Real-World Cases

- **Immutable updates:** `const newList = [...oldList, newItem];`.
- **Combining data sources:** `[...apiData, ...cacheData]`.
- **Building HTML:** `[...headerRows, ...bodyRows]`.

---

## Spreading Iterables

### Definitions

**Core Definition:** Spreading iterables converts any iterable object into an array or expands it into individual values.

**Technical Definition:** Per ECMA-262, spread uses the iterator protocol (`GetIterator`). Any object implementing `Symbol.iterator` can be spread. This includes arrays, strings, Sets, Maps, generators, NodeLists, and custom iterables.

**Beginner-Friendly Explanation:** Spread works on anything that can be looped over — not just arrays. Sets, Maps, strings, and even custom objects that implement iteration.

### Purposes

- To convert Sets and Maps to arrays.
- To spread strings into characters.
- To consume generators.
- To convert DOM collections to arrays.
- To work with any iterable uniformly.

### Syntax Rules and Structure

```js
const fromSet = [...new Set([1, 2, 2, 3])];
const fromString = [..."abc"];
const fromMap = [...new Map([["a", 1]])];
const fromGenerator = [...gen()];
```

**Syntax Rules:**
- Operand must implement `Symbol.iterator`.
- Maps spread as `[key, value]` pairs.
- Generators are consumed (lazy sequences become arrays).
- DOM NodeLists are iterable in modern browsers.

**Constraints and Limitations:**
- Non-iterable objects throw `TypeError`.
- Generators are exhausted after spreading.
- Large iterables allocate large arrays.

### Annotated Code Examples

#### Example 1: Spreading a Set

```js
const set = new Set([1, 2, 2, 3, 3]);
const arr = [...set];
console.log(arr); // [1, 2, 3]
```

**Expected Output:**
```
[ 1, 2, 3 ]
```

**Why This Output:** Sets are iterable; spread extracts unique values.

#### Example 2: Spreading a Map

```js
const map = new Map([["a", 1], ["b", 2]]);
const entries = [...map];
console.log(entries); // [["a", 1], ["b", 2]]

const keys = [...map.keys()];
console.log(keys); // ["a", "b"]
```

**Expected Output:**
```
[ [ 'a', 1 ], [ 'b', 2 ] ]
[ 'a', 'b' ]
```

**Why This Output:** Maps iterate as `[key, value]` pairs; `.keys()` iterates keys.

#### Example 3: Spreading a Generator

```js
function* range(start, end) {
  for (let i = start; i < end; i++) yield i;
}

const nums = [...range(1, 5)];
console.log(nums); // [1, 2, 3, 4]
```

**Expected Output:**
```
[ 1, 2, 3, 4 ]
```

**Why This Output:** Generators are iterable; spread consumes them into an array.

#### Example 4: Custom Iterable

```js
const custom = {
  *[Symbol.iterator]() {
    yield "a";
    yield "b";
    yield "c";
  },
};

console.log([...custom]); // ["a", "b", "c"]
```

**Expected Output:**
```
[ 'a', 'b', 'c' ]
```

**Why This Output:** Any object with `Symbol.iterator` can be spread.

### Real-World Cases

- **Set deduplication:** `[...new Set(arr)]`.
- **Map to array:** `[...map.entries()]`.
- **Range generation:** `[...Array(5).keys()]`.
- **DOM:** `[...document.querySelectorAll("div")]`.

---

## Spread in Function Calls

### Definitions

**Core Definition:** Spread in function calls expands an iterable into individual arguments passed to the function.

**Technical Definition:** Per ECMA-262 §13.3.8, when a `SpreadElement` appears in an `ArgumentList`, the function is called with each yielded value as a separate argument. This replaces the older `Function.prototype.apply` pattern.

**Beginner-Friendly Explanation:** Spread lets you pass an array's elements as individual arguments: `fn(...arr)` instead of `fn(arr[0], arr[1], ...)`.

### Purposes

- To pass array elements as function arguments.
- To call variadic functions with arrays.
- To replace `apply()`.
- To spread multiple arrays into one call.

### Syntax Rules and Structure

```js
fn(...args);
fn(a, ...args, b);
Math.max(...numbers);
```

**Component Breakdown:**
- `fn`: The function to call.
- `...args`: Spread operand.
- Each element becomes a separate argument.

**Syntax Rules:**
- Operand must be iterable.
- Can be combined with regular arguments.
- Can spread multiple iterables in one call.
- Replaces `fn.apply(null, args)`.

**Constraints and Limitations:**
- Very large arrays may hit argument count limits (engine-dependent).
- Operand must be iterable; non-iterables throw.
- The order of arguments follows the order of spread elements.

### Annotated Code Examples

#### Example 1: `Math.max` with Spread

```js
const nums = [3, 1, 4, 1, 5, 9, 2, 6];

// Step 1: Without spread.
console.log(Math.max(nums[0], nums[1], /* ... */)); // Hard to write

// Step 2: With spread.
console.log(Math.max(...nums)); // 9
console.log(Math.min(...nums)); // 1
```

**Expected Output:**
```
9
1
```

**Why This Output:** Spread expands the array into individual arguments for `Math.max`.

#### Example 2: Combining Spread with Regular Arguments

```js
function log(level, ...messages) {
  console.log(`[${level}]`, ...messages);
}

const msgs = ["hello", "world"];
log("INFO", ...msgs);
```

**Expected Output:**
```
[INFO] hello world
```

**Why This Output:** Spread passes array elements as separate arguments, combined with the regular `level` argument.

#### Example 3: Replacing `apply()`

```js
function sum(a, b, c) {
  return a + b + c;
}

const args = [1, 2, 3];

// Old way
console.log(sum.apply(null, args)); // 6

// Modern way with spread
console.log(sum(...args)); // 6
```

**Expected Output:**
```
6
6
```

**Why This Output:** Spread achieves the same result as `apply` but is cleaner and more readable.

#### Example 4: Spreading Multiple Iterables

```js
const a = [1, 2];
const b = [3, 4];

function combine(...args) {
  return args;
}

console.log(combine(...a, ...b)); // [1, 2, 3, 4]
console.log(combine(0, ...a, ...b, 5)); // [0, 1, 2, 3, 4, 5]
```

**Expected Output:**
```
[ 1, 2, 3, 4 ]
[ 0, 1, 2, 3, 4, 5 ]
```

**Why This Output:** Multiple spread elements are expanded in order.

### Real-World Cases

- **Math functions:** `Math.max(...values)`.
- **Array methods:** `arr.push(...items)`.
- **Logging:** `console.log(...args)`.
- **React:** `<Component {...props} />` (object spread).
- **Constructors:** `new Date(...[2025, 0, 1])`.

---

## References

- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Spread Element — https://tc39.es/ecma262/multipage/ecmascript-language-expressions.html#sec-array-initializer
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Argument Lists — https://tc39.es/ecma262/multipage/ecmascript-language-expressions.html#sec-argument-lists
- MDN Web Docs – Spread syntax (...) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax
- MDN Web Docs – Rest parameters — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/rest_parameters
- MDN Web Docs – Array — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array
- MDN Web Docs – Function.prototype.apply() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/apply
- MDN Web Docs – Iteration protocols — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols
- MDN Web Docs – structuredClone() — https://developer.mozilla.org/en-US/docs/Web/API/Window/structuredClone
- MDN Web Docs – Set — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set
- MDN Web Docs – Map — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map
- MDN Web Docs – Math.max() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/max
- React Docs – JSX Spread Attributes — https://react.dev/learn/responding-to-events
- Redux Docs – Immutable Update Patterns — https://redux.js.org/usage/structuring-reducers/immutable-update-patterns
- Exploring JS – Spread syntax — https://exploringjs.com/impatient-js/ch_arrays.html#spread-elements-and-arrays
- TC39 Proposal – Object Rest/Spread Properties — https://github.com/tc39/proposal-object-rest-spread
- JavaScript.info – Spread syntax — https://javascript.info/rest-parameters-spread
- V8 Blog – Elements Kinds in V8 — https://v8.dev/blog/elements-kinds