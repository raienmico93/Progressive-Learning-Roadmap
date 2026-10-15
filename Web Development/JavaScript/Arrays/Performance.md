# JavaScript Array Performance — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Array performance refers to the runtime speed and memory efficiency of array operations in JavaScript, determined by the engine's internal representation of arrays, the choice of iteration mechanism, and the patterns used to manipulate array data.

**Technical Definition:** Per ECMAScript 2025 (ECMA-262, §10.4.2), arrays are exotic objects with special treatment for integer-indexed property keys and a `length` property. V8 (Chrome, Node.js) optimises arrays internally using **elements kinds** — a taxonomy of array representations (PACKED_SMI_ELEMENTS, PACKED_DOUBLE_ELEMENTS, PACKED_ELEMENTS, HOLEY_* variants, DICTIONARY_ELEMENTS) that determines access speed and memory layout. Transitions between kinds (e.g., adding a double to an SMI array, creating holes) cause deoptimisation. Iteration mechanisms differ in overhead: classic `for` loops avoid iterator and callback overhead, while `forEach`, `map`, `filter`, and `reduce` add function call costs. Array resizing (via `push`, `pop`, `shift`, `unshift`, `splice`) has varying complexity depending on which end is affected.

**Beginner-Friendly Explanation:** Arrays are usually fast, but not all array operations are equally fast. Some patterns (like storing mixed types, creating sparse arrays, or using `for...in`) are much slower than others. Understanding these performance characteristics helps you write code that runs quickly, especially when processing large amounts of data.

### Key Characteristics

- **Elements kinds:** V8 optimises arrays based on their contents and structure.
- **Packed vs holey:** Dense arrays are faster than sparse (holey) arrays.
- **Monotype vs mixed:** Uniform element types are faster than mixed types.
- **Iteration overhead:** Classic `for` is often fastest; methods add callback overhead.
- **Resizing costs:** `push`/`pop` are O(1) amortised; `shift`/`unshift` are O(n).
- **Memory locality:** Contiguous storage improves cache performance.
- **Deoptimisation triggers:** Holes, mixed types, and large sparse arrays.

### Prerequisites

- Basic JavaScript syntax (arrays, loops, methods).
- Understanding of arrays and array methods.
- Familiarity with iteration mechanisms.
- Awareness of Big-O notation.
- Basic knowledge of memory and references.

### Related Programming Areas

- **V8 internals:** Elements kinds, hidden classes.
- **Algorithm complexity:** Big-O analysis.
- **Memory management:** Allocation and garbage collection.
- **Functional programming:** `map`, `filter`, `reduce` trade-offs.
- **Data processing:** Large datasets, ETL pipelines.
- **Game development:** Frame-rate-critical loops.

---

## Array Performance Characteristics

### Definitions

**Core Definition:** Array performance characteristics are the runtime properties of arrays that determine the speed and memory cost of operations.

**Technical Definition:** Per V8's implementation, arrays have an internal "elements kind" that determines their representation. The fastest kinds are PACKED_SMI_ELEMENTS (dense small integers) and PACKED_DOUBLE_ELEMENTS (dense doubles). PACKED_ELEMENTS holds arbitrary values. HOLEY_* variants are slower due to holes. DICTIONARY_ELEMENTS is used for very sparse arrays and is much slower. Transitions from faster to slower kinds happen when you add incompatible values or create holes.

**Beginner-Friendly Explanation:** JavaScript arrays are fastest when they hold only one type of data, don't have missing slots, and are accessed sequentially. Mixed types, holes, and unusual access patterns slow them down.

### Elements Kinds (V8)

| Kind | Description | Speed |
|---|---|---|
| PACKED_SMI_ELEMENTS | Dense small integers | Fastest |
| PACKED_DOUBLE_ELEMENTS | Dense doubles | Fast |
| PACKED_ELEMENTS | Dense mixed/objects | Fast |
| HOLEY_SMI_ELEMENTS | Sparse small integers | Slower |
| HOLEY_DOUBLE_ELEMENTS | Sparse doubles | Slower |
| HOLEY_ELEMENTS | Sparse mixed | Slow |
| DICTIONARY_ELEMENTS | Very sparse | Slowest |

### Annotated Code Examples

#### Example 1: Fast vs Slow Arrays

```js
// FAST: Packed SMIs.
const fast = [1, 2, 3, 4, 5];

// SLOWER: Packed doubles (transition).
const doubles = [1, 2, 3];
doubles.push(4.5); // Transitions to PACKED_DOUBLE_ELEMENTS

// SLOWER: Packed elements (mixed types).
const mixed = [1, "two", { three: 3 }];

// SLOWER: Holey (sparse).
const holey = [1, , 3]; // Hole at index 1

// SLOWEST: Dictionary (very sparse).
const sparse = [];
sparse[1000000] = "value"; // Dictionary mode
```

**Expected Output:** No output; performance differences are internal.

**Why This Output:** Each pattern triggers a different elements kind, affecting access speed.

#### Example 2: Benchmarking Access

```js
const N = 1_000_000;

// Packed SMIs.
const packed = Array.from({ length: N }, (_, i) => i);
let start = performance.now();
let sum = 0;
for (let i = 0; i < N; i++) sum += packed[i];
console.log(`Packed: ${(performance.now() - start).toFixed(2)} ms`);

// Holey.
const holey = new Array(N);
for (let i = 0; i < N; i += 2) holey[i] = i;
start = performance.now();
sum = 0;
for (let i = 0; i < N; i++) sum += holey[i] || 0;
console.log(`Holey: ${(performance.now() - start).toFixed(2)} ms`);
```

**Expected Output (approximate):**
```
Packed: 4.20 ms
Holey: 12.50 ms
```

**Why This Output:** Holey arrays are slower due to extra bounds/prototype checks.

### Real-World Cases

- **Data processing:** Use packed arrays for large datasets.
- **Numerical computation:** Prefer typed arrays for numeric data.
- **Game loops:** Avoid holes and mixed types.

---

## Sparse Arrays

### Definitions

**Core Definition:** A sparse array is an array with holes — indices that have no element.

**Technical Definition:** Per ECMA-262, a hole is an array index that is not an own property. Sparse arrays trigger `HOLEY_*` elements kinds in V8, which require additional checks during access. Very sparse arrays may transition to `DICTIONARY_ELEMENTS`, which uses a hash table instead of a contiguous array.

**Beginner-Friendly Explanation:** Sparse arrays have "gaps." They're slower to access because the engine can't assume every index exists.

### Performance Impact

- **HOLEY kinds:** Slower than PACKED due to prototype chain checks.
- **DICTIONARY mode:** Much slower for access; used for very sparse arrays.
- **Method behaviour:** `forEach`, `map`, `filter` skip holes.
- **Memory:** Sparse arrays may use more memory per element in dictionary mode.

### Annotated Code Examples

```js
// Step 1: Create sparse via literal.
const sparse1 = [1, , 3];

// Step 2: Create sparse via new Array.
const sparse2 = new Array(5); // 5 holes

// Step 3: Create sparse via delete.
const sparse3 = [1, 2, 3];
delete sparse3[1]; // Hole at index 1

// Step 4: Create sparse via high index.
const sparse4 = [];
sparse4[1000] = "value"; // Dictionary mode

// All of these are slower than packed arrays.
```

### Annotated Code Examples: Avoiding Sparse Arrays

```js
// BAD: Sparse via new Array.
const bad = new Array(5);
bad[0] = "a"; // Still holey

// GOOD: Dense via Array.from.
const good = Array.from({ length: 5 }, () => 0);
good[0] = "a"; // Packed

// BAD: Deleting elements.
const arr = [1, 2, 3];
delete arr[1]; // Creates hole

// GOOD: Filtering into a new array.
const filtered = [1, 2, 3].filter((_, i) => i !== 1);
```

### Real-World Cases

- **Avoid `delete` on arrays:** Use `filter` instead.
- **Avoid `new Array(n)`:** Use `Array.from({ length: n })`.
- **Sparse data:** Use `Map` for truly sparse data.

---

## Array Resizing

### Definitions

**Core Definition:** Array resizing is the process of changing an array's length by adding or removing elements.

**Technical Definition:** In V8, arrays have an internal capacity that may exceed `length`. `push` and `pop` are amortised O(1) because they operate at the end and may trigger occasional reallocation. `shift` and `unshift` are O(n) because they move all elements. `splice` is O(n) for insertion/removal at arbitrary positions. Setting `length` smaller truncates in O(1) for the length change but may free elements.

**Beginner-Friendly Explanation:** Adding or removing items at the end of an array is fast. Adding or removing at the beginning is slow because everything else has to shift.

### Complexity Table

| Operation | Complexity | Notes |
|---|---|---|
| `push` | O(1) amortised | May reallocate |
| `pop` | O(1) | |
| `shift` | O(n) | Moves all elements |
| `unshift` | O(n) | Moves all elements |
| `splice` (end) | O(1) | |
| `splice` (middle) | O(n) | |
| `arr.length = n` | O(1) | Truncation frees memory |
| `concat` | O(n) | Creates new array |
| `slice` | O(n) | Creates new array |

### Annotated Code Examples

```js
// FAST: push/pop.
const arr = [];
for (let i = 0; i < 100000; i++) arr.push(i); // Fast

// SLOW: unshift.
const arr2 = [];
for (let i = 0; i < 10000; i++) arr2.unshift(i); // Slow (O(n²) total)

// FAST: pop.
while (arr.length) arr.pop(); // Fast
```

### Real-World Cases

- **Queue:** Use `push`/`shift` (or a ring buffer) for FIFO.
- **Stack:** Use `push`/`pop` for LIFO.
- **Bulk operations:** Prefer building with `push` then reversing if needed.

---

## Array Methods Performance

### Definitions

**Core Definition:** Array methods performance is the relative speed of built-in methods like `map`, `filter`, `reduce`, `forEach`, and their loop equivalents.

**Technical Definition:** Methods like `map`, `filter`, `reduce`, and `forEach` iterate the array and invoke a callback per element. The callback invocation adds overhead compared to inline loop bodies. `map` and `filter` also allocate new arrays, adding memory pressure. `reduce` threads an accumulator. `some` and `every` short-circuit, which can be faster than full iteration. `find` and `findIndex` also short-circuit.

**Beginner-Friendly Explanation:** Built-in methods are convenient but slightly slower than hand-written loops because they call a function for each element. For most code, the difference is negligible; for hot paths, classic loops win.

### Performance Comparison

| Method | Speed | Notes |
|---|---|---|
| Classic `for` | Fastest | No callback overhead |
| `for...of` | Slower | Iterator protocol |
| `forEach` | Medium | Callback overhead |
| `map` | Medium | Callback + new array |
| `filter` | Medium | Callback + new array |
| `reduce` | Medium | Callback + accumulator |
| `some`/`every` | Fast (short-circuit) | Stops early |
| `find`/`findIndex` | Fast (short-circuit) | Stops early |

### Annotated Code Examples

```js
const N = 1_000_000;
const arr = Array.from({ length: N }, (_, i) => i);

// Classic for.
let start = performance.now();
let sum = 0;
for (let i = 0; i < arr.length; i++) sum += arr[i];
console.log(`for: ${(performance.now() - start).toFixed(2)} ms`);

// forEach.
start = performance.now();
sum = 0;
arr.forEach((x) => { sum += x; });
console.log(`forEach: ${(performance.now() - start).toFixed(2)} ms`);

// reduce.
start = performance.now();
sum = arr.reduce((a, b) => a + b, 0);
console.log(`reduce: ${(performance.now() - start).toFixed(2)} ms`);

// map (creates new array).
start = performance.now();
const doubled = arr.map((x) => x * 2);
console.log(`map: ${(performance.now() - start).toFixed(2)} ms`);
```

**Expected Output (approximate):**
```
for: 4.50 ms
forEach: 8.20 ms
reduce: 9.10 ms
map: 12.30 ms
```

**Why This Output:** Classic `for` has no callback overhead; `map` allocates a new array.

### Real-World Cases

- **Hot paths:** Use classic `for`.
- **Readability:** Use methods when performance is not critical.
- **Short-circuiting:** Prefer `some`/`every`/`find` for early exit.
- **Large datasets:** Benchmark before optimising.

---

## `for` vs `forEach`

### Definitions

**Core Definition:** `for` and `forEach` are two common iteration mechanisms with different performance and semantic characteristics.

**Technical Definition:** The classic `for` loop is a language statement; `forEach` is an `Array.prototype` method that invokes a callback per element. `for` allows `break`, `continue`, and `return` (in functions); `forEach` cannot be interrupted. `for` gives direct index access; `forEach` passes `(element, index, array)`.

**Beginner-Friendly Explanation:** `for` is a loop you control fully; `forEach` is a method that runs a function on each element but can't be stopped early.

### Comparison Table

| Feature | `for` | `forEach` |
|---|---|---|
| Break early | Yes | No |
| Continue | Yes | `return` in callback (skips rest of callback, not iteration) |
| Index access | Direct | Via callback arg |
| Return value | N/A | `undefined` |
| Chainable | No | No |
| Callback overhead | No | Yes |
| Speed | Faster | Slower |
| Readability | Verbose | Concise |

### Annotated Code Examples

```js
// Classic for: can break.
const arr = [1, 2, 3, 4, 5];
for (let i = 0; i < arr.length; i++) {
  if (arr[i] === 3) break;
  console.log(arr[i]); // 1, 2
}

// forEach: cannot break.
arr.forEach((x) => {
  if (x === 3) return; // Skips rest of this callback, not the loop
  console.log(x); // 1, 2, 4, 5
});
```

**Expected Output:**
```
1
2
1
2
4
5
```

**Why This Output:** `break` exits the loop; `return` in `forEach` only exits the current callback.

### Real-World Cases

- **Early exit:** Use `for` or `some`/`every`.
- **Side effects:** `forEach` is fine when no early exit is needed.
- **Performance:** `for` for hot paths.

---

## `map` vs `for`

### Definitions

**Core Definition:** `map` and `for` are two ways to transform array elements, differing in allocation, return value, and readability.

**Technical Definition:** `map` creates a new array of the same length, invoking a callback per element. A `for` loop can transform in place or build a new array manually. `map` is optimised in V8 but incurs callback and allocation overhead. `for` can reuse existing arrays and avoid allocation.

**Beginner-Friendly Explanation:** `map` is concise and creates a new array; `for` is more flexible and can avoid allocation.

### Comparison Table

| Feature | `map` | `for` |
|---|---|---|
| Returns new array | Yes | No (unless you build one) |
| Modifies original | No | Optional |
| Callback overhead | Yes | No |
| Allocation | Always | Optional |
| Readability | High | Medium |
| Speed | Slower | Faster |
| Chainable | Yes | No |

### Annotated Code Examples

#### Example 1: `map` Creates a New Array

```js
const nums = [1, 2, 3];
const doubled = nums.map((x) => x * 2);
console.log(doubled); // [2, 4, 6]
console.log(nums);    // [1, 2, 3] (unchanged)
```

#### Example 2: `for` Can Transform In Place

```js
const nums = [1, 2, 3];
for (let i = 0; i < nums.length; i++) {
  nums[i] *= 2;
}
console.log(nums); // [2, 4, 6]
```

#### Example 3: `for` Can Build a New Array Without Callback Overhead

```js
const nums = [1, 2, 3];
const doubled = new Array(nums.length);
for (let i = 0; i < nums.length; i++) {
  doubled[i] = nums[i] * 2;
}
console.log(doubled); // [2, 4, 6]
```

### Real-World Cases

- **Immutability:** Use `map` for non-mutating transformations.
- **Performance:** Use `for` for hot paths.
- **Chaining:** Use `map` for functional pipelines.
- **In-place:** Use `for` when you own the array.

---

## Memory Considerations

### Definitions

**Core Definition:** Memory considerations for arrays include the memory footprint of element storage, the cost of allocation, and the impact on garbage collection.

**Technical Definition:** In V8, packed SMI arrays use 4 bytes per element (or less for small integers), packed double arrays use 8 bytes, and packed element arrays use pointer-sized slots (8 bytes on 64-bit). Dictionary-mode arrays use hash tables with higher overhead. Creating new arrays (via `map`, `filter`, `slice`, `concat`) allocates memory, increasing GC pressure. Reusing arrays via `length = 0` avoids reallocation.

**Beginner-Friendly Explanation:** Arrays take up memory. Creating new arrays uses more memory and triggers garbage collection. Reusing arrays can reduce memory pressure.

### Memory Tips

- **Preallocate when size is known:** `new Array(n)` (but beware holes) or `Array.from({ length: n })`.
- **Reuse arrays:** `arr.length = 0` clears without reallocating.
- **Avoid unnecessary copies:** `map`/`filter` allocate; use `for` when possible.
- **Use typed arrays for numeric data:** Much smaller footprint.
- **Avoid sparse arrays:** They may use more memory.

### Annotated Code Examples

```js
// BAD: Creating many temporary arrays.
let result = [];
for (let i = 0; i < 1000; i++) {
  const temp = data.map((x) => x * i); // Allocates 1000 arrays
  result = result.concat(temp); // Allocates more
}

// GOOD: Reuse and preallocate.
const result2 = new Array(data.length * 1000);
let idx = 0;
for (let i = 0; i < 1000; i++) {
  for (let j = 0; j < data.length; j++) {
    result2[idx++] = data[j] * i;
  }
}
```

### Real-World Cases

- **Data pipelines:** Avoid intermediate arrays.
- **Games:** Reuse buffers.
- **Node.js servers:** Reduce GC pressure.
- **Mobile:** Memory-constrained devices.

---

## Optimization Tips

### Definitions

**Core Definition:** Array optimisation tips are practical guidelines for writing faster and more memory-efficient array code.

**Technical Definition:** Optimisations include keeping arrays packed and monotype, avoiding holes and mixed types, using classic `for` loops in hot paths, avoiding unnecessary allocations, preallocating when size is known, using typed arrays for numeric data, and preferring short-circuiting methods (`some`, `every`, `find`) when applicable.

**Beginner-Friendly Explanation:** These are the rules of thumb for making array code fast.

### Optimization Tips

1. **Keep arrays packed and monotype.**
   ```js
   // BAD: Mixed types.
   const bad = [1, "two", { three: 3 }];
   // GOOD: Uniform.
   const good = [1, 2, 3];
   ```

2. **Avoid holes.**
   ```js
   // BAD
   const bad = [1, , 3];
   // GOOD
   const good = [1, 0, 3];
   ```

3. **Use classic `for` in hot paths.**
   ```js
   for (let i = 0, len = arr.length; i < len; i++) { ... }
   ```

4. **Cache `length` for large arrays.**
   ```js
   for (let i = 0, len = arr.length; i < len; i++) { ... }
   ```

5. **Avoid `delete`; use `filter` or `splice`.**
   ```js
   // BAD
   delete arr[i];
   // GOOD
   arr.splice(i, 1);
   ```

6. **Reuse arrays instead of allocating.**
   ```js
   arr.length = 0; // Clear without reallocating
   ```

7. **Preallocate when size is known.**
   ```js
   const arr = new Array(n).fill(0);
   ```

8. **Use typed arrays for numeric data.**
   ```js
   const ta = new Float64Array(1000);
   ```

9. **Prefer `some`/`every`/`find` for short-circuit.**
   ```js
   const found = arr.find((x) => x > 10);
   ```

10. **Benchmark before optimising.**
    ```js
    console.time("loop");
    // ...
    console.timeEnd("loop");
    ```

### Annotated Code Examples

#### Example 1: Optimised Sum

```js
// SLOW: forEach with callback overhead.
function sumSlow(arr) {
  let sum = 0;
  arr.forEach((x) => { sum += x; });
  return sum;
}

// FAST: classic for.
function sumFast(arr) {
  let sum = 0;
  for (let i = 0, len = arr.length; i < len; i++) {
    sum += arr[i];
  }
  return sum;
}

const arr = Array.from({ length: 1_000_000 }, (_, i) => i);
console.time("slow");
sumSlow(arr);
console.timeEnd("slow");
console.time("fast");
sumFast(arr);
console.timeEnd("fast");
```

**Expected Output (approximate):**
```
slow: 8.20ms
fast: 4.50ms
```

#### Example 2: Avoiding Unnecessary Allocations

```js
// BAD: Chained methods allocate multiple arrays.
const result = data
  .filter((x) => x > 0)
  .map((x) => x * 2)
  .reduce((a, b) => a + b, 0);

// GOOD: Single pass.
let sum = 0;
for (let i = 0, len = data.length; i < len; i++) {
  if (data[i] > 0) sum += data[i] * 2;
}
```

### Real-World Cases

- **Hot loops:** Game loops, data processing.
- **Large datasets:** ETL, analytics.
- **Mobile:** Battery and memory constraints.
- **Node.js servers:** Throughput and latency.

---

## References

- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Array Exotic Objects — https://tc39.es/ecma262/multipage/ordinary-and-exotic-objects-behaviours.html#sec-array-exotic-objects
- MDN Web Docs – Array — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array
- MDN Web Docs – Array.prototype.push() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/push
- MDN Web Docs – Array.prototype.pop() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/pop
- MDN Web Docs – Array.prototype.shift() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/shift
- MDN Web Docs – Array.prototype.unshift() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/unshift
- MDN Web Docs – Array.prototype.splice() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/splice
- MDN Web Docs – Array.prototype.map() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map
- MDN Web Docs – Array.prototype.filter() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter
- MDN Web Docs – Array.prototype.reduce() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce
- MDN Web Docs – Array.prototype.forEach() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach
- MDN Web Docs – Array.prototype.some() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/some
- MDN Web Docs – Array.prototype.every() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/every
- MDN Web Docs – Array.prototype.find() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/find
- V8 Blog – Elements Kinds in V8 — https://v8.dev/blog/elements-kinds
- V8 Blog – Fast Properties in V8 — https://v8.dev/blog/fast-properties
- V8 Blog – High-performance garbage collection for V8 — https://v8.dev/blog/high-performance-gc
- JavaScript.info – Arrays — https://javascript.info/array
- Exploring JS – Arrays — https://exploringjs.com/impatient-js/ch_arrays.html
- You Don't Know JS – Types & Grammar (Arrays) — https://github.com/getify/You-Dont-Know-JS/blob/2nd-ed/types-grammar/ch3.md
- Chrome DevTools – Memory panel overview — https://developer.chrome.com/docs/devtools/memory
- Mathias Bynens – V8 Internals: Elements Kinds — https://www.youtube.com/watch?v=m9cTaYI95Zc