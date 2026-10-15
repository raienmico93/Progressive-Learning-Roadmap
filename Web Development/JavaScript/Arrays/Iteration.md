# JavaScript Array Iteration — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Array iteration is the process of traversing the elements of an array to read, transform, filter, or accumulate values, using loops or higher-order array methods.

**Technical Definition:** Per ECMAScript 2025 (ECMA-262), arrays support multiple iteration mechanisms: classic `for` loops using index access, `for...of` using the iterator protocol (`Array.prototype[Symbol.iterator]`), `for...in` enumerating enumerable string keys, and higher-order methods defined on `Array.prototype` (`forEach`, `map`, `filter`, `reduce`, and others). Each mechanism has distinct semantics for holes, early termination, return values, and performance characteristics. The iterator protocol (§7.4.1) defines a standard interface for producing sequences of values.

**Beginner-Friendly Explanation:** Array iteration is how you go through the items in an array one by one. JavaScript gives you several tools for this: classic `for` loops that count indices, `for...of` that gives you values directly, `for...in` that gives you keys (best avoided for arrays), and methods like `forEach`, `map`, `filter`, and `reduce` that process elements with functions.

### Key Characteristics

- **Multiple mechanisms:** Loops, iterators, and higher-order methods.
- **Different semantics:** Each handles holes, early exit, and return values differently.
- **Iterator protocol:** `for...of` uses `Symbol.iterator`.
- **Higher-order methods:** Accept callbacks that receive `(element, index, array)`.
- **Immutability options:** `map` and `filter` return new arrays; `forEach` does not.
- **Performance varies:** Classic `for` loops are often fastest; methods are more expressive.

### Prerequisites

- Basic JavaScript syntax (variables, functions, arrays).
- Understanding of array indexing and `length`.
- Familiarity with functions and callbacks.
- Awareness of arrow functions and `this`.
- Basic knowledge of the iterator protocol.

### Related Programming Areas

- **Functional programming:** `map`, `filter`, `reduce`.
- **Data processing:** Transformations and aggregations.
- **DOM manipulation:** Iterating NodeLists.
- **Async iteration:** `for await...of`.
- **Performance optimisation:** Choosing the right loop.
- **Array methods:** `some`, `every`, `find`, `flatMap`.

---

## `for` Loop

### Definitions

**Core Definition:** The classic `for` loop iterates over array indices, giving full control over initialisation, condition, and increment.

**Technical Definition:** Per ECMA-262 §14.7.3, the `for` statement evaluates an initialiser, then repeatedly evaluates a test expression and executes the loop body, followed by an update expression, until the test is falsy. When used with arrays, the test typically checks `i < arr.length`.

**Beginner-Friendly Explanation:** The classic `for` loop counts from a start index to an end index, running the body each time. It's the most explicit and often the fastest way to iterate.

### Purposes

- To iterate with full control over the index.
- To skip or reverse iteration.
- To break or continue early.
- To achieve maximum performance in hot paths.
- To iterate multiple arrays in parallel.

### Syntax Rules and Structure

```js
for (initialization; condition; update) {
  // body
}
```

**Component Breakdown:**
- `initialization`: Runs once before the loop.
- `condition`: Evaluated before each iteration; loop continues while truthy.
- `update`: Runs after each iteration.
- `body`: Executed each iteration.

**Syntax Rules:**
- All three parts are optional: `for (;;)` is an infinite loop.
- Use `let` for the counter to avoid leakage.
- `break` exits the loop; `continue` skips to the next iteration.
- Cache `arr.length` if it does not change and performance matters.

**Constraints and Limitations:**
- More verbose than higher-order methods.
- Manual index management can lead to off-by-one errors.
- Does not skip holes automatically (accesses `undefined`).

### Annotated Code Examples

#### Example 1: Basic Forward Iteration

```js
const fruits = ["apple", "banana", "cherry"];

// Step 1: Iterate from 0 to length - 1.
for (let i = 0; i < fruits.length; i++) {
  console.log(i, fruits[i]);
}
```

**Expected Output:**
```
0 apple
1 banana
2 cherry
```

**Why This Output:** The loop starts at `i = 0`, runs while `i < 3`, and increments `i` after each iteration.

#### Example 2: Reverse Iteration and Break

```js
const nums = [1, 2, 3, 4, 5];

// Step 1: Reverse iteration.
for (let i = nums.length - 1; i >= 0; i--) {
  if (nums[i] === 3) break; // Stop at 3
  console.log(nums[i]);
}
```

**Expected Output:**
```
5
4
```

**Why This Output:** Iteration starts at index 4 and decrements. When `nums[i] === 3`, `break` exits the loop.

#### Example 3: Iterating Two Arrays in Parallel

```js
const names = ["Alice", "Bob", "Carol"];
const ages = [30, 25, 35];

for (let i = 0; i < names.length; i++) {
  console.log(`${names[i]} is ${ages[i]}`);
}
```

**Expected Output:**
```
Alice is 30
Bob is 25
Carol is 35
```

**Why This Output:** A single index accesses both arrays, keeping them in sync.

### Real-World Cases

- **Performance-critical loops:** Game loops, numerical computation.
- **Reverse iteration:** Removing elements during iteration.
- **Parallel arrays:** Combining related data.
- **Manual skipping:** `i += 2` for every-other-element.

---

## `for...of`

### Definitions

**Core Definition:** The `for...of` loop iterates over iterable values, yielding each element directly.

**Technical Definition:** Per ECMA-262 §14.7.5, the `for...of` statement obtains the iterator from the iterable via `GetIterator`, then repeatedly calls `next()` and binds the `value` to the loop variable until `done` is `true`. Arrays implement `Symbol.iterator` via `Array.prototype[Symbol.iterator]` (aliased to `values`).

**Beginner-Friendly Explanation:** `for...of` gives you the values of an array directly, without needing an index. It's cleaner and works on any iterable (arrays, strings, Sets, Maps, generators).

### Purposes

- To iterate over values without managing indices.
- To work with any iterable, not just arrays.
- To use `break` and `continue`.
- To iterate over Sets, Maps, strings, and generators.

### Syntax Rules and Structure

```js
for (const element of iterable) {
  // body
}
```

**Component Breakdown:**
- `element`: The current value (can be destructured).
- `iterable`: Any object with `Symbol.iterator`.

**Syntax Rules:**
- Supports `break`, `continue`, and `return` in functions.
- `const` is preferred; `let` allows reassignment.
- Destructuring is supported: `for (const [k, v] of map)`.
- Works with any iterable, including `arguments`, NodeList, and generators.

**Constraints and Limitations:**
- Does not provide the index directly (use `entries()`).
- Holes are treated as `undefined`.
- Slightly slower than classic `for` in some engines.

### Annotated Code Examples

#### Example 1: Basic `for...of`

```js
const fruits = ["apple", "banana", "cherry"];

for (const fruit of fruits) {
  console.log(fruit);
}
```

**Expected Output:**
```
apple
banana
cherry
```

**Why This Output:** `for...of` yields each element in order.

#### Example 2: With Index via `entries()`

```js
const fruits = ["apple", "banana", "cherry"];

for (const [index, fruit] of fruits.entries()) {
  console.log(index, fruit);
}
```

**Expected Output:**
```
0 apple
1 banana
2 cherry
```

**Why This Output:** `entries()` returns an iterator of `[index, value]` pairs, destructured in the loop.

#### Example 3: Iterating a Set and a String

```js
const unique = new Set([1, 2, 2, 3]);
for (const n of unique) console.log(n); // 1, 2, 3

for (const char of "abc") console.log(char); // "a", "b", "c"
```

**Expected Output:**
```
1
2
3
a
b
c
```

**Why This Output:** `for...of` works on any iterable, not just arrays.

### Real-World Cases

- **Modern iteration:** Preferred over `for` when index isn't needed.
- **Sets and Maps:** Natural iteration.
- **Generators:** Consuming lazy sequences.
- **DOM NodeLists:** `for (const node of document.querySelectorAll("div"))`.

---

## `for...in`

### Definitions

**Core Definition:** The `for...in` loop iterates over the enumerable string-keyed properties of an object, including inherited ones.

**Technical Definition:** Per ECMA-262 §14.7.5.6, `for...in` enumerates the enumerable string keys of an object, walking the prototype chain. It is **not** recommended for arrays because it includes non-index properties and inherited properties.

**Beginner-Friendly Explanation:** `for...in` gives you keys, not values. It's designed for objects, not arrays. For arrays, use `for...of` or a classic `for` loop.

### Purposes

- To iterate over object keys (not arrays).
- To enumerate inherited enumerable properties (rarely desired).

### Syntax Rules and Structure

```js
for (const key in object) {
  // key is a string
}
```

**Constraints and Limitations:**
- Includes inherited enumerable properties.
- Order is not guaranteed for all keys.
- Includes non-index properties on arrays.
- Should be avoided for arrays; use `for...of` instead.

### Annotated Code Examples

#### Example 1: `for...in` on an Object

```js
const user = { name: "Alice", age: 30 };

for (const key in user) {
  if (Object.hasOwn(user, key)) {
    console.log(key, user[key]);
  }
}
```

**Expected Output:**
```
name Alice
age 30
```

**Why This Output:** `Object.hasOwn` filters out inherited properties.

#### Example 2: `for...in` on an Array (Anti-Pattern)

```js
const arr = ["a", "b", "c"];
arr.custom = "property";

for (const key in arr) {
  console.log(key);
}
```

**Expected Output:**
```
0
1
2
custom
```

**Why This Output:** `for...in` includes the custom property, which is usually not desired. This is why `for...in` is discouraged for arrays.

### Real-World Cases

- **Object iteration:** Enumerating keys of plain objects.
- **Debugging:** Inspecting all enumerable properties.
- **Avoid for arrays:** Use `for...of` or `Object.keys`.

---

## `forEach()`

### Definitions

**Core Definition:** `forEach()` executes a provided callback function once for each array element in ascending index order.

**Technical Definition:** Per ECMA-262 §23.1.3.12, `Array.prototype.forEach(callback, thisArg)` calls `callback(element, index, array)` for each element. It skips holes. It returns `undefined` and cannot be chained.

**Beginner-Friendly Explanation:** `forEach()` runs a function on each item in the array. It's for side effects (like logging or updating the DOM), not for producing a new array.

### Purposes

- To perform side effects for each element.
- To iterate without needing a return value.
- To avoid explicit index management.

### Syntax Rules and Structure

```js
arr.forEach((element, index, array) => {
  // body
}, thisArg);
```

**Component Breakdown:**
- `element`: Current value.
- `index`: Current index (optional).
- `array`: The array being iterated (optional).
- `thisArg` (optional): `this` inside the callback.

**Syntax Rules:**
- Skips holes in sparse arrays.
- Cannot be broken early (no `break`).
- Returns `undefined`.
- Not chainable.

**Constraints and Limitations:**
- No early termination.
- Cannot be chained.
- Skips holes.
- Slightly slower than classic `for` for large arrays.

### Annotated Code Examples

```js
const nums = [1, 2, 3];

// Step 1: Log each element.
nums.forEach((n, i) => console.log(i, n));

// Step 2: Accumulate (via external variable).
let sum = 0;
nums.forEach((n) => { sum += n; });
console.log("Sum:", sum);
```

**Expected Output:**
```
0 1
1 2
2 3
Sum: 6
```

**Why This Output:** `forEach` runs the callback for each element; side effects (logging, accumulating) happen inside.

### Real-World Cases

- **DOM updates:** Applying changes to each element.
- **Logging:** Simple iteration for side effects.
- **Testing:** Running assertions per element.

---

## `map()`

### Definitions

**Core Definition:** `map()` creates a new array populated with the results of calling a callback on every element.

**Technical Definition:** Per ECMA-262 §23.1.3.15, `Array.prototype.map(callback, thisArg)` creates a new array, calls `callback(element, index, array)` for each element, and stores the return value at the corresponding index. Skips holes.

**Beginner-Friendly Explanation:** `map()` transforms each item in an array and gives you a new array of the transformed values.

### Purposes

- To transform array elements.
- To extract properties from objects.
- To convert between data types.
- To prepare data for rendering or serialisation.

### Syntax Rules and Structure

```js
const newArr = arr.map((element, index, array) => {
  return transformedValue;
});
```

**Syntax Rules:**
- Returns a new array of the same length.
- Callback must return a value.
- Skips holes (result has holes at same indices).
- Does not modify the original array.

**Constraints and Limitations:**
- Cannot break early.
- Always returns a same-length array.
- Callback must return a value (otherwise `undefined`).

### Annotated Code Examples

```js
const nums = [1, 2, 3, 4];

// Step 1: Square each number.
const squares = nums.map((n) => n ** 2);
console.log(squares); // [1, 4, 9, 16]

// Step 2: Extract property.
const users = [{ name: "Alice" }, { name: "Bob" }];
const names = users.map((u) => u.name);
console.log(names); // ["Alice", "Bob"]

// Step 3: Original is unchanged.
console.log(nums); // [1, 2, 3, 4]
```

**Expected Output:**
```
[ 1, 4, 9, 16 ]
[ 'Alice', 'Bob' ]
[ 1, 2, 3, 4 ]
```

**Why This Output:** `map` creates a new array, leaving the original untouched.

### Real-World Cases

- **Data transformation:** Converting API responses.
- **React rendering:** `items.map((item) => <li key={item.id}>{item.name}</li>)`.
- **Formatting:** `prices.map((p) => \`$${p.toFixed(2)}\`)`.

---

## `filter()`

### Definitions

**Core Definition:** `filter()` creates a new array containing only elements that pass a test implemented by a callback.

**Technical Definition:** Per ECMA-262 §23.1.3.7, `Array.prototype.filter(callback, thisArg)` creates a new array with all elements for which `callback(element, index, array)` returns a truthy value. Skips holes.

**Beginner-Friendly Explanation:** `filter()` selects items that match a condition, producing a new array.

### Purposes

- To select a subset of elements.
- To remove unwanted items.
- To search for matching elements.
- To partition data.

### Syntax Rules and Structure

```js
const filtered = arr.filter((element, index, array) => {
  return condition; // truthy to keep
});
```

**Syntax Rules:**
- Returns a new array of variable length.
- Callback returns a truthy/falsy value.
- Skips holes.
- Does not modify the original.

**Constraints and Limitations:**
- Cannot break early.
- Only one array returned (no partition).
- Combines well with `map` for transform + filter.

### Annotated Code Examples

```js
const nums = [1, 2, 3, 4, 5, 6];

// Step 1: Keep evens.
const evens = nums.filter((n) => n % 2 === 0);
console.log(evens); // [2, 4, 6]

// Step 2: Filter objects.
const users = [
  { name: "Alice", active: true },
  { name: "Bob", active: false },
  { name: "Carol", active: true },
];
const activeUsers = users.filter((u) => u.active);
console.log(activeUsers.map((u) => u.name)); // ["Alice", "Carol"]

// Step 3: Chain with map.
const doubledEvens = nums.filter((n) => n % 2 === 0).map((n) => n * 2);
console.log(doubledEvens); // [4, 8, 12]
```

**Expected Output:**
```
[ 2, 4, 6 ]
[ 'Alice', 'Carol' ]
[ 4, 8, 12 ]
```

**Why This Output:** `filter` selects elements; `map` transforms the result.

### Real-World Cases

- **Search:** Filtering products by criteria.
- **Validation:** Removing invalid records.
- **Permissions:** Filtering allowed actions.

---

## `reduce()`

### Definitions

**Core Definition:** `reduce()` executes a reducer function on each element, accumulating a single result.

**Technical Definition:** Per ECMA-262 §23.1.3.22, `Array.prototype.reduce(callback, initialValue)` calls `callback(accumulator, element, index, array)` for each element, threading the accumulator. If no `initialValue` is provided, the first element is used as the accumulator and iteration starts at index 1. Throws `TypeError` on empty arrays without an initial value.

**Beginner-Friendly Explanation:** `reduce()` boils an array down to a single value — a sum, a maximum, a grouped object, anything.

### Purposes

- To sum, average, or aggregate values.
- To find maximum or minimum.
- To group elements into objects.
- To flatten nested arrays.
- To count occurrences.

### Syntax Rules and Structure

```js
const result = arr.reduce((accumulator, element, index, array) => {
  return newAccumulator;
}, initialValue);
```

**Component Breakdown:**
- `accumulator`: The running result.
- `element`: Current value.
- `index`: Current index.
- `array`: The source array.
- `initialValue` (recommended): Starting accumulator.

**Syntax Rules:**
- Without `initialValue`, the first element is used and iteration starts at index 1.
- With `initialValue`, iteration starts at index 0.
- Must return the new accumulator each time.
- Throws `TypeError` on empty array without `initialValue`.

**Constraints and Limitations:**
- Can be hard to read for complex logic.
- Skips holes.
- Must always return a value.

### Annotated Code Examples

#### Example 1: Sum and Maximum

```js
const nums = [1, 2, 3, 4, 5];

// Step 1: Sum.
const sum = nums.reduce((acc, n) => acc + n, 0);
console.log(sum); // 15

// Step 2: Maximum.
const max = nums.reduce((acc, n) => (n > acc ? n : acc), -Infinity);
console.log(max); // 5
```

**Expected Output:**
```
15
5
```

**Why This Output:** The accumulator carries the running result; the initial value sets the starting point.

#### Example 2: Grouping

```js
const items = [
  { name: "apple", category: "fruit" },
  { name: "carrot", category: "vegetable" },
  { name: "banana", category: "fruit" },
];

const grouped = items.reduce((acc, item) => {
  (acc[item.category] ||= []).push(item.name);
  return acc;
}, {});

console.log(grouped);
// { fruit: ["apple", "banana"], vegetable: ["carrot"] }
```

**Expected Output:**
```
{ fruit: [ 'apple', 'banana' ], vegetable: [ 'carrot' ] }
```

**Why This Output:** The accumulator is an object; each element is added to the appropriate group.

#### Example 3: Counting Occurrences

```js
const words = ["apple", "banana", "apple", "cherry", "banana", "apple"];

const counts = words.reduce((acc, word) => {
  acc[word] = (acc[word] || 0) + 1;
  return acc;
}, {});

console.log(counts);
// { apple: 3, banana: 2, cherry: 1 }
```

**Expected Output:**
```
{ apple: 3, banana: 2, cherry: 1 }
```

**Why This Output:** Each word increments its counter in the accumulator object.

### Real-World Cases

- **Statistics:** Sum, average, min, max.
- **Grouping:** Categorising data.
- **Flattening:** `arr.reduce((acc, x) => acc.concat(x), [])`.
- **Pipelines:** Composing functions.

---

## Iteration Performance

### Definitions

**Core Definition:** Iteration performance refers to the relative speed and memory cost of different array iteration mechanisms.

**Technical Definition:** Classic `for` loops with cached length are typically the fastest in V8. `for...of` uses the iterator protocol, adding overhead. Higher-order methods (`forEach`, `map`, `filter`, `reduce`) add callback invocation overhead but are optimised by engines. Performance depends on array size, engine version, and workload.

**Beginner-Friendly Explanation:** Some loops are faster than others. Classic `for` loops are usually fastest; methods are slower but more expressive.

### Performance Comparison (Approximate)

| Method | Relative Speed | Notes |
|---|---|---|
| Classic `for` (cached length) | Fastest | Minimal overhead |
| Classic `for` (uncached) | Fast | Slight overhead from `length` lookup |
| `forEach` | Medium | Callback overhead |
| `map` / `filter` | Medium | Callback + new array allocation |
| `reduce` | Medium | Callback + accumulator threading |
| `for...of` | Slower | Iterator protocol overhead |
| `for...in` | Slowest | Prototype chain walk |

### Annotated Code Examples

```js
const N = 1_000_000;
const arr = Array.from({ length: N }, (_, i) => i);

// Classic for
let start = performance.now();
let sum1 = 0;
for (let i = 0; i < arr.length; i++) sum1 += arr[i];
console.log(`for: ${(performance.now() - start).toFixed(2)} ms`);

// for...of
start = performance.now();
let sum2 = 0;
for (const n of arr) sum2 += n;
console.log(`for...of: ${(performance.now() - start).toFixed(2)} ms`);

// forEach
start = performance.now();
let sum3 = 0;
arr.forEach((n) => { sum3 += n; });
console.log(`forEach: ${(performance.now() - start).toFixed(2)} ms`);

// reduce
start = performance.now();
const sum4 = arr.reduce((a, n) => a + n, 0);
console.log(`reduce: ${(performance.now() - start).toFixed(2)} ms`);
```

**Expected Output (approximate):**
```
for: 4.50 ms
for...of: 12.30 ms
forEach: 8.20 ms
reduce: 9.10 ms
```

**Why This Output:** Classic `for` avoids iterator and callback overhead; `for...of` pays the iterator protocol cost.

### Real-World Cases

- **Hot paths:** Use classic `for` for maximum speed.
- **Readability:** Use methods when performance is not critical.
- **Large datasets:** Benchmark before optimising.
- **Avoid `for...in`:** Never use for arrays.

---

## Iteration Best Practices

### Definitions

**Core Definition:** Iteration best practices are the conventions and patterns that make array iteration code correct, readable, and performant.

**Technical Definition:** Best practices include choosing the right iteration mechanism for the task, avoiding `for...in` on arrays, preferring `for...of` when the index isn't needed, using `map`/`filter`/`reduce` for transformations, avoiding mutation during iteration, and caching `length` in hot loops.

**Beginner-Friendly Explanation:** These are the rules of thumb for iterating arrays well.

### Best Practices

1. **Use `for...of` when you need values, not indices.**
   ```js
   for (const item of items) { ... }
   ```

2. **Use `map` for transformations, `filter` for selection, `reduce` for aggregation.**
   ```js
   const names = users.map((u) => u.name);
   const active = users.filter((u) => u.active);
   const total = nums.reduce((a, b) => a + b, 0);
   ```

3. **Never use `for...in` on arrays.**
   ```js
   for (const key in arr) { ... } // BAD
   for (const value of arr) { ... } // GOOD
   ```

4. **Avoid mutating an array during iteration.**
   ```js
   // BAD: Skipping/duplicating elements
   for (let i = 0; i < arr.length; i++) { if (cond) arr.splice(i, 1); }
   // GOOD: Filter into a new array
   const filtered = arr.filter((x) => !cond(x));
   ```

5. **Use `entries()` when you need both index and value.**
   ```js
   for (const [i, v] of arr.entries()) { ... }
   ```

6. **Cache `length` in performance-critical loops.**
   ```js
   for (let i = 0, len = arr.length; i < len; i++) { ... }
   ```

7. **Use `for...of` for any iterable (Set, Map, string, generator).**
   ```js
   for (const [k, v] of map) { ... }
   ```

8. **Prefer `const` for loop variables unless reassignment is needed.**
   ```js
   for (const item of items) { ... }
   ```

9. **Use `Array.from` or spread to convert array-likes before iterating with methods.**
   ```js
   const arr = Array.from(document.querySelectorAll("div"));
   arr.forEach((el) => { ... });
   ```

10. **Chain methods for readable pipelines.**
    ```js
    const result = arr
      .filter((x) => x > 0)
      .map((x) => x * 2)
      .reduce((a, b) => a + b, 0);
    ```

### Annotated Code Examples

#### Example 1: Good vs Bad Iteration

```js
const arr = [1, 2, 3, 4, 5];

// GOOD: for...of for values.
let sum = 0;
for (const n of arr) sum += n;
console.log(sum); // 15

// BAD: for...in for arrays.
let sum2 = 0;
for (const key in arr) sum2 += arr[key];
console.log(sum2); // 15 (but risky)

// GOOD: map/filter/reduce chain.
const result = arr
  .filter((n) => n % 2 === 0)
  .map((n) => n * 10)
  .reduce((a, b) => a + b, 0);
console.log(result); // 60
```

**Expected Output:**
```
15
15
60
```

**Why This Output:** `for...of` is clean; `for...in` works but is fragile; the method chain is expressive.

#### Example 2: Avoiding Mutation During Iteration

```js
// BAD: Mutating during iteration.
const nums = [1, 2, 3, 4, 5];
for (let i = 0; i < nums.length; i++) {
  if (nums[i] % 2 === 0) nums.splice(i, 1); // Skips elements!
}
console.log(nums); // [1, 3, 5] — but only by luck here

// GOOD: Filter into a new array.
const odds = [1, 2, 3, 4, 5].filter((n) => n % 2 !== 0);
console.log(odds); // [1, 3, 5]
```

**Expected Output:**
```
[ 1, 3, 5 ]
[ 1, 3, 5 ]
```

**Why This Output:** `filter` creates a new array, avoiding index-shifting bugs.

### Real-World Cases

- **React:** `items.map()` for rendering lists.
- **Data pipelines:** Chaining `filter`/`map`/`reduce`.
- **Performance:** Classic `for` in game loops.
- **Modern code:** `for...of` as default iteration.

---

## References

- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Array.prototype.forEach — https://tc39.es/ecma262/multipage/indexed-collections.html#sec-array.prototype.foreach
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Array.prototype.map — https://tc39.es/ecma262/multipage/indexed-collections.html#sec-array.prototype.map
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Array.prototype.filter — https://tc39.es/ecma262/multipage/indexed-collections.html#sec-array.prototype.filter
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Array.prototype.reduce — https://tc39.es/ecma262/multipage/indexed-collections.html#sec-array.prototype.reduce
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – for...of Statement — https://tc39.es/ecma262/multipage/ecmascript-language-statements-and-declarations.html#sec-for-in-and-for-of-statements
- MDN Web Docs – Array — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array
- MDN Web Docs – for...of — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...of
- MDN Web Docs – for...in — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...in
- MDN Web Docs – Array.prototype.forEach() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach
- MDN Web Docs – Array.prototype.map() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map
- MDN Web Docs – Array.prototype.filter() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter
- MDN Web Docs – Array.prototype.reduce() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce
- MDN Web Docs – Iteration protocols — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols
- MDN Web Docs – Array.prototype.entries() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/entries
- Exploring JS – Arrays — https://exploringjs.com/impatient-js/ch_arrays.html
- V8 Blog – Elements Kinds in V8 — https://v8.dev/blog/elements-kinds
- You Don't Know JS – Types & Grammar (Arrays) — https://github.com/getify/You-Dont-Know-JS/blob/2nd-ed/types-grammar/ch3.md