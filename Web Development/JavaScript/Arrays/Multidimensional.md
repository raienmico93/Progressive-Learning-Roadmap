# JavaScript Multidimensional Arrays — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A multidimensional array is an array whose elements are themselves arrays (or other collections), forming a nested structure that can represent grids, matrices, tables, and higher-dimensional data.

**Technical Definition:** Per ECMAScript 2025 (ECMA-262), JavaScript does **not** have a native multidimensional array type. Instead, multidimensional arrays are simulated using **arrays of arrays** (jagged arrays). An array of arrays is an Array exotic object whose elements are also Array exotic objects. Each inner array can have a different length (making the structure "jagged"), and accessing an element requires multiple index operations: `matrix[row][col]`. ES2019 introduced `Array.prototype.flat()` and `flatMap()` for flattening nested arrays.

**Beginner-Friendly Explanation:** A multidimensional array is like a grid or a table. A regular array is a single row of items; a 2D array is a list of rows, where each row is itself an array. You can think of it as a spreadsheet with rows and columns, or a chess board with rows and columns. Accessing an item requires two indices: the row first, then the column.

### Key Characteristics

- **No native type:** Multidimensional arrays are arrays of arrays, not a distinct type.
- **Jagged structure:** Inner arrays can have different lengths.
- **Reference semantics:** Inner arrays are shared references when copied shallowly.
- **Row-major access:** `matrix[row][col]` accesses the element at row, column.
- **Iteration is nested:** Iterating requires nested loops or methods.
- **Flattening available:** `flat()` and `flatMap()` convert nested arrays to flat ones.
- **Memory implications:** Each inner array is a separate object.

### Prerequisites

- Basic JavaScript syntax (arrays, indices, loops).
- Understanding of arrays and array methods.
- Familiarity with nested loops and iteration.
- Awareness of reference semantics.
- Basic knowledge of `map`, `reduce`, `flat`.

### Related Programming Areas

- **Mathematics:** Matrices, vectors, tensors.
- **Graphics:** Pixel grids, transformation matrices.
- **Games:** Game boards, tile maps.
- **Data analysis:** Tables, spreadsheets.
- **Machine learning:** Tensors, weight matrices.
- **Image processing:** RGB pixel arrays.

---

## Nested Arrays

### Definitions

**Core Definition:** Nested arrays are arrays that contain other arrays as elements, forming a hierarchy of depth greater than one.

**Technical Definition:** Per ECMA-262, a nested array is an Array exotic object whose elements are themselves Array exotic objects. Each level is a separate array with its own `length` and elements. Access requires successive index operations.

**Beginner-Friendly Explanation:** A nested array is an "array inside an array." You can think of it as a list of lists, or a folder containing sub-folders.

### Purposes

- To represent hierarchical data.
- To create grids and tables.
- To group related items.
- To model matrices and tensors.

### Syntax Rules and Structure

```js
const nested = [[1, 2], [3, 4], [5, 6]];
const deep = [[[1, 2], [3, 4]], [[5, 6], [7, 8]]];
```

**Component Breakdown:**
- Outer `[ ]`: The top-level array.
- Inner `[ ]`: Sub-arrays.
- Each level can have different lengths (jagged).

**Syntax Rules:**
- Created with array literals or `Array.from`.
- Access requires `arr[i][j][k]...`.
- Inner arrays are independent objects.
- `length` applies to each level separately.

**Constraints and Limitations:**
- Deeply nested arrays are hard to read and debug.
- Shallow copies share inner arrays.
- No built-in method for deep operations.

### Annotated Code Examples

#### Example 1: Basic Nested Array

```js
// Step 1: Create a nested array.
const matrix = [
  [1, 2, 3],
  [4, 5, 6],
  [7, 8, 9],
];

// Step 2: Access elements.
console.log(matrix[0]);     // [1, 2, 3]
console.log(matrix[1][1]);  // 5 (row 1, col 1)
console.log(matrix[2][0]);  // 7 (row 2, col 0)

// Step 3: Modify an element.
matrix[1][2] = 99;
console.log(matrix[1]);     // [4, 5, 99]
```

**Expected Output:**
```
[ 1, 2, 3 ]
5
7
[ 4, 5, 99 ]
```

**Why This Output:** `matrix[1]` returns the second inner array; `matrix[1][1]` accesses the element at index 1 within that array.

#### Example 2: Jagged Arrays

```js
const jagged = [
  [1],
  [2, 3],
  [4, 5, 6],
];

console.log(jagged[0].length); // 1
console.log(jagged[1].length); // 2
console.log(jagged[2].length); // 3

// Iterate over all elements.
for (const row of jagged) {
  for (const value of row) {
    console.log(value);
  }
}
```

**Expected Output:**
```
1
2
3
1
2
3
4
5
6
```

**Why This Output:** Inner arrays can have different lengths; nested loops handle the jagged structure.

#### Example 3: Iterating with `forEach`

```js
const grid = [
  [1, 2],
  [3, 4],
];

grid.forEach((row, i) => {
  row.forEach((value, j) => {
    console.log(`[${i}][${j}] = ${value}`);
  });
});
```

**Expected Output:**
```
[0][0] = 1
[0][1] = 2
[1][0] = 3
[1][1] = 4
```

**Why This Output:** Nested `forEach` iterates rows then columns.

### Real-World Cases

- **Chess board:** 8×8 grid.
- **Pixel data:** Image rows and columns.
- **Spreadsheet:** Rows of cells.
- **Tile maps:** Game world layouts.

---

## Matrix Representation

### Definitions

**Core Definition:** A matrix is a rectangular 2D array where all rows have the same length, used to represent mathematical matrices.

**Technical Definition:** In JavaScript, a matrix is represented as an array of rows, each row being an array of numbers (or other values). Matrices support operations like addition, multiplication, and transposition, which require nested loops.

**Beginner-Friendly Explanation:** A matrix is a grid of numbers arranged in rows and columns. In JavaScript, you represent it as an array of arrays, where each inner array is a row.

### Purposes

- To represent mathematical matrices.
- To perform linear algebra operations.
- To implement transformations in graphics.
- To store tabular numeric data.

### Syntax Rules and Structure

```js
const matrix = [
  [1, 2, 3],
  [4, 5, 6],
];
// 2 rows × 3 columns
```

**Component Breakdown:**
- Rows: `matrix.length`.
- Columns: `matrix[0].length` (assuming rectangular).
- Element: `matrix[row][col]`.

**Syntax Rules:**
- Rectangular matrices have equal row lengths.
- Row-major order: `matrix[i][j]` is row `i`, column `j`.
- Column-major order is not native but can be simulated.

**Constraints and Limitations:**
- JavaScript has no built-in matrix type.
- Performance: Nested loops are slower than typed arrays.
- For heavy numeric work, consider `TypedArray` or libraries (math.js).

### Annotated Code Examples

#### Example 1: Matrix Addition

```js
function addMatrices(a, b) {
  const rows = a.length;
  const cols = a[0].length;
  const result = [];
  for (let i = 0; i < rows; i++) {
    const row = [];
    for (let j = 0; j < cols; j++) {
      row.push(a[i][j] + b[i][j]);
    }
    result.push(row);
  }
  return result;
}

const A = [[1, 2], [3, 4]];
const B = [[5, 6], [7, 8]];
console.log(addMatrices(A, B));
// [[6, 8], [10, 12]]
```

**Expected Output:**
```
[ [ 6, 8 ], [ 10, 12 ] ]
```

**Why This Output:** Element-wise addition of corresponding matrix elements.

#### Example 2: Matrix Multiplication

```js
function multiplyMatrices(a, b) {
  const rowsA = a.length;
  const colsA = a[0].length;
  const colsB = b[0].length;
  const result = Array.from({ length: rowsA }, () => Array(colsB).fill(0));
  for (let i = 0; i < rowsA; i++) {
    for (let j = 0; j < colsB; j++) {
      for (let k = 0; k < colsA; k++) {
        result[i][j] += a[i][k] * b[k][j];
      }
    }
  }
  return result;
}

const A = [[1, 2], [3, 4]];
const B = [[5, 6], [7, 8]];
console.log(multiplyMatrices(A, B));
// [[19, 22], [43, 50]]
```

**Expected Output:**
```
[ [ 19, 22 ], [ 43, 50 ] ]
```

**Why This Output:** Standard matrix multiplication: `result[i][j] = Σ a[i][k] * b[k][j]`.

#### Example 3: Matrix Transposition

```js
function transpose(matrix) {
  const rows = matrix.length;
  const cols = matrix[0].length;
  const result = Array.from({ length: cols }, () => []);
  for (let i = 0; i < rows; i++) {
    for (let j = 0; j < cols; j++) {
      result[j][i] = matrix[i][j];
    }
  }
  return result;
}

console.log(transpose([[1, 2, 3], [4, 5, 6]]));
// [[1, 4], [2, 5], [3, 6]]
```

**Expected Output:**
```
[ [ 1, 4 ], [ 2, 5 ], [ 3, 6 ] ]
```

**Why This Output:** Rows become columns and vice versa.

### Real-World Cases

- **Graphics:** Transformation matrices.
- **Machine learning:** Weight matrices.
- **Physics:** Rotation, scaling.
- **Data analysis:** Correlation matrices.

---

## Matrix Operations

### Definitions

**Core Definition:** Matrix operations are algorithms that manipulate matrices, including addition, multiplication, transposition, rotation, and search.

**Technical Definition:** Matrix operations typically involve nested loops with time complexity O(n²) for element-wise operations and O(n³) for multiplication. They can be implemented with plain nested loops, higher-order methods, or typed arrays for performance.

**Beginner-Friendly Explanation:** Matrix operations are the things you do with matrices — adding them, multiplying them, rotating them, finding elements.

### Common Operations

#### 1. Element-wise Addition

```js
function add(a, b) {
  return a.map((row, i) => row.map((val, j) => val + b[i][j]));
}
```

#### 2. Element-wise Multiplication

```js
function hadamard(a, b) {
  return a.map((row, i) => row.map((val, j) => val * b[i][j]));
}
```

#### 3. Scalar Multiplication

```js
function scale(matrix, k) {
  return matrix.map((row) => row.map((val) => val * k));
}
```

#### 4. Transposition

```js
function transpose(matrix) {
  return matrix[0].map((_, i) => matrix.map((row) => row[i]));
}
```

#### 5. Rotation (90° clockwise)

```js
function rotate90(matrix) {
  return matrix[0].map((_, i) => matrix.map((row) => row[i]).reverse());
}
```

#### 6. Sum of All Elements

```js
function sumAll(matrix) {
  return matrix.reduce((sum, row) => sum + row.reduce((s, v) => s + v, 0), 0);
}
```

#### 7. Searching for a Value

```js
function findValue(matrix, target) {
  for (let i = 0; i < matrix.length; i++) {
    for (let j = 0; j < matrix[i].length; j++) {
      if (matrix[i][j] === target) return [i, j];
    }
  }
  return null;
}
```

### Annotated Code Examples

#### Example 1: Element-wise Addition with `map`

```js
function add(a, b) {
  return a.map((row, i) => row.map((val, j) => val + b[i][j]));
}

console.log(add([[1, 2], [3, 4]], [[5, 6], [7, 8]]));
// [[6, 8], [10, 12]]
```

**Expected Output:**
```
[ [ 6, 8 ], [ 10, 12 ] ]
```

**Why This Output:** `map` applies the addition element-wise across rows.

#### Example 2: 90° Rotation

```js
function rotate90(matrix) {
  return matrix[0].map((_, i) =>
    matrix.map((row) => row[i]).reverse()
  );
}

console.log(rotate90([[1, 2], [3, 4]]));
// [[3, 1], [4, 2]]
```

**Expected Output:**
```
[ [ 3, 1 ], [ 4, 2 ] ]
```

**Why This Output:** Each new row is a column of the original, reversed.

#### Example 3: Sum of All Elements

```js
function sumAll(matrix) {
  return matrix.reduce(
    (sum, row) => sum + row.reduce((s, v) => s + v, 0),
    0
  );
}

console.log(sumAll([[1, 2, 3], [4, 5, 6]])); // 21
```

**Expected Output:**
```
21
```

**Why This Output:** The outer `reduce` sums row totals; the inner `reduce` sums each row.

### Real-World Cases

- **Image processing:** Convolution, blurring, edge detection.
- **Game development:** Rotating tile maps.
- **Data analysis:** Aggregating tabular data.
- **Cryptography:** Matrix-based ciphers.

---

## Flattening Arrays

### Definitions

**Core Definition:** Flattening an array is the process of converting a nested array into a single-level array.

**Technical Definition:** Per ES2019, `Array.prototype.flat(depth)` creates a new array with sub-array elements concatenated up to the specified `depth` (default 1). `Array.prototype.flatMap(callback)` maps each element then flattens the result by one level. For deeper flattening, `flat(Infinity)` or recursive functions are used.

**Beginner-Friendly Explanation:** Flattening means "un-nesting" an array. If you have `[[1, 2], [3, 4]]`, flattening gives you `[1, 2, 3, 4]`.

### Purposes

- To convert nested arrays to flat ones.
- To simplify data processing.
- To merge arrays of arrays.
- To prepare data for functions that expect flat arrays.

### Syntax Rules and Structure

```js
arr.flat()        // Flatten one level
arr.flat(depth)   // Flatten `depth` levels
arr.flat(Infinity) // Flatten all levels
arr.flatMap(fn)   // Map then flatten one level
```

**Component Breakdown:**
- `depth` (optional): Number of levels to flatten (default 1).
- `Infinity`: Flattens all levels.
- `flatMap(fn)`: Equivalent to `arr.map(fn).flat(1)`.

**Syntax Rules:**
- `flat()` removes empty slots.
- `flat()` does not flatten non-array elements.
- `flatMap()` is more efficient than `map().flat()`.
- `depth` must be a non-negative integer.

**Constraints and Limitations:**
- `flat(Infinity)` may cause stack overflow on extremely deep arrays.
- `flat` does not flatten nested objects.
- `flatMap` only flattens one level.

### Annotated Code Examples

#### Example 1: Basic `flat()`

```js
const nested = [1, [2, 3], [4, [5, 6]]];

console.log(nested.flat());        // [1, 2, 3, 4, [5, 6]]
console.log(nested.flat(2));       // [1, 2, 3, 4, 5, 6]
console.log(nested.flat(Infinity)); // [1, 2, 3, 4, 5, 6]
```

**Expected Output:**
```
[ 1, 2, 3, 4, [ 5, 6 ] ]
[ 1, 2, 3, 4, 5, 6 ]
[ 1, 2, 3, 4, 5, 6 ]
```

**Why This Output:** `flat()` flattens one level by default; `flat(2)` flattens two levels; `flat(Infinity)` flattens all.

#### Example 2: `flatMap()`

```js
const sentences = ["Hello world", "Goodbye moon"];

const words = sentences.flatMap((s) => s.split(" "));
console.log(words); // ["Hello", "world", "Goodbye", "moon"]
```

**Expected Output:**
```
[ 'Hello', 'world', 'Goodbye', 'moon' ]
```

**Why This Output:** `flatMap` maps each sentence to an array of words, then flattens one level.

#### Example 3: Flattening a Matrix

```js
const matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]];

const flat = matrix.flat();
console.log(flat); // [1, 2, 3, 4, 5, 6, 7, 8, 9]

// Sum all elements.
const sum = matrix.flat().reduce((a, b) => a + b, 0);
console.log(sum); // 45
```

**Expected Output:**
```
[ 1, 2, 3, 4, 5, 6, 7, 8, 9 ]
45
```

**Why This Output:** `flat()` converts the matrix to a single array; `reduce` sums the elements.

### Real-World Cases

- **Data processing:** Flattening API responses.
- **Rendering:** Combining nested lists.
- **Search:** Flattening for `includes` or `indexOf`.
- **Math:** Summing matrix elements.

---

## Deep Flattening

### Definitions

**Core Definition:** Deep flattening is the process of flattening an array to a single level regardless of nesting depth.

**Technical Definition:** Deep flattening can be achieved with `arr.flat(Infinity)`, a recursive function, or a stack-based iterative approach. The recursive approach uses `Array.isArray` to detect nested arrays and `concat` or `push` to accumulate. The iterative approach uses an explicit stack to avoid recursion limits.

**Beginner-Friendly Explanation:** Deep flattening means "un-nesting everything." No matter how deeply nested the arrays are, the result is a single flat array.

### Purposes

- To handle arbitrarily nested data.
- To normalise irregular structures.
- To avoid recursion limits with iterative approaches.
- To process deeply nested JSON.

### Syntax Rules and Structure

**Using `flat(Infinity)`:**
```js
const flat = deeplyNested.flat(Infinity);
```

**Recursive:**
```js
function deepFlatten(arr) {
  return arr.reduce(
    (acc, val) => acc.concat(Array.isArray(val) ? deepFlatten(val) : val),
    []
  );
}
```

**Iterative (stack-based):**
```js
function deepFlattenIterative(arr) {
  const stack = [...arr];
  const result = [];
  while (stack.length) {
    const next = stack.pop();
    if (Array.isArray(next)) {
      stack.push(...next);
    } else {
      result.push(next);
    }
  }
  return result.reverse();
}
```

**Constraints and Limitations:**
- `flat(Infinity)` may overflow the stack on extremely deep arrays.
- Recursive functions have stack limits.
- Iterative approaches avoid stack overflow but are more verbose.
- Non-array iterables are not flattened.

### Annotated Code Examples

#### Example 1: `flat(Infinity)`

```js
const deep = [1, [2, [3, [4, [5]]]]];

console.log(deep.flat(Infinity)); // [1, 2, 3, 4, 5]
```

**Expected Output:**
```
[ 1, 2, 3, 4, 5 ]
```

**Why This Output:** `flat(Infinity)` flattens all levels regardless of depth.

#### Example 2: Recursive Deep Flatten

```js
function deepFlatten(arr) {
  return arr.reduce(
    (acc, val) => acc.concat(Array.isArray(val) ? deepFlatten(val) : val),
    []
  );
}

console.log(deepFlatten([1, [2, [3, [4]]]])); // [1, 2, 3, 4]
```

**Expected Output:**
```
[ 1, 2, 3, 4 ]
```

**Why This Output:** The recursive function processes nested arrays by calling itself.

#### Example 3: Iterative Deep Flatten

```js
function deepFlattenIterative(arr) {
  const stack = [...arr];
  const result = [];
  while (stack.length) {
    const next = stack.pop();
    if (Array.isArray(next)) {
      stack.push(...next);
    } else {
      result.push(next);
    }
  }
  return result.reverse();
}

console.log(deepFlattenIterative([1, [2, [3, [4, [5]]]]]));
// [1, 2, 3, 4, 5]
```

**Expected Output:**
```
[ 1, 2, 3, 4, 5 ]
```

**Why This Output:** The stack-based approach processes nested arrays iteratively, avoiding recursion limits.

#### Example 4: Flattening Object Values

```js
const data = {
  a: [1, 2],
  b: [[3, 4], [5, 6]],
  c: [7, [8, [9]]],
};

const allValues = Object.values(data).flat(Infinity);
console.log(allValues); // [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

**Expected Output:**
```
[ 1, 2, 3, 4, 5, 6, 7, 8, 9 ]
```

**Why This Output:** `Object.values` extracts arrays; `flat(Infinity)` flattens them all.

### Real-World Cases

- **JSON processing:** Flattening nested API responses.
- **Tree structures:** Collecting all nodes.
- **File systems:** Flattening directory trees.
- **Data analysis:** Normalising irregular data.

---

## References

- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Array Exotic Objects — https://tc39.es/ecma262/multipage/ordinary-and-exotic-objects-behaviours.html#sec-array-exotic-objects
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Array.prototype.flat — https://tc39.es/ecma262/multipage/indexed-collections.html#sec-array.prototype.flat
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Array.prototype.flatMap — https://tc39.es/ecma262/multipage/indexed-collections.html#sec-array.prototype.flatmap
- MDN Web Docs – Array — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array
- MDN Web Docs – Array.prototype.flat() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/flat
- MDN Web Docs – Array.prototype.flatMap() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/flatMap
- MDN Web Docs – Indexed collections — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Indexed_collections
- MDN Web Docs – Spread syntax — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax
- MDN Web Docs – Array.from() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/from
- MDN Web Docs – TypedArray — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypedArray
- Exploring JS – Arrays — https://exploringjs.com/impatient-js/ch_arrays.html
- JavaScript.info – Arrays — https://javascript.info/array
- TC39 Proposal – Array.prototype.flat/flatMap — https://github.com/tc39/proposal-flatMap
- math.js Documentation – Matrices — https://mathjs.org/docs/datatypes/matrices.html
- V8 Blog – Elements Kinds in V8 — https://v8.dev/blog/elements-kinds
- Wikipedia – Matrix (mathematics) — https://en.wikipedia.org/wiki/Matrix_(mathematics)