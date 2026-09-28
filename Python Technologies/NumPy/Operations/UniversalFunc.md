# NumPy Universal Functions (ufuncs): A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A universal function (ufunc) is a function that operates on ndarrays in an element-by-element fashion, supporting array broadcasting, type casting, and several other standard features.

**Technical Definition:** A ufunc is an instance of the `numpy.ufunc` class—a "vectorized" wrapper for a function that takes a fixed number of specific inputs and produces a fixed number of specific outputs. It is implemented in compiled C code and operates on whole arrays without Python-level loops.

**Beginner-Friendly Explanation:** A ufunc is a super-fast math function that works on entire arrays at once. Instead of writing a loop to add 1 to every number in a list, you just call `np.add(array, 1)` and it does it all in one go, at C speed. NumPy has dozens of built-in ufuncs for arithmetic, trigonometry, logarithms, comparisons, and more.

### Key Characteristics

- **Element-Wise:** Operates on each element of an array independently.
- **Compiled C Loops:** Uses highly optimized compiled code, avoiding Python loop overhead.
- **Broadcasting Support:** Automatically aligns arrays of different shapes.
- **Type Casting:** Handles automatic promotion and casting between dtypes.
- **Five Core Methods:** All ufuncs have `reduce`, `accumulate`, `reduceat`, `outer`, and `at` methods.
- **`out` Parameter:** Supports writing results directly to pre-allocated arrays.

### Prerequisites

- NumPy installed (`pip install numpy`).
- Basic Python syntax and understanding of variables.
- Familiarity with NumPy array creation and the `import numpy as np` convention.
- Conceptual understanding of broadcasting.

### Related Programming Areas

- **Scientific Computing:** Applying mathematical functions to large datasets.
- **Signal Processing:** Computing trigonometric and exponential transformations.
- **Machine Learning:** Implementing activation functions and gradient computations.
- **Data Analysis:** Performing element-wise statistical operations.

### Core Concepts / Features

The following core concepts are explored in detail: (1) Concept of ufuncs, (2) Element-wise execution, (3) Unary ufuncs, (4) Binary ufuncs, (5) Advanced ufunc methods (`reduce`, `accumulate`, `outer`, `at`), and (6) The `out` parameter.

---

## Core Concept 1: Concept of Ufuncs

### Definitions

**Core Definition:** A ufunc is a function that operates element-by-element on whole arrays, wrapping a compiled C loop that processes the data without Python-level iteration.

**Technical Definition:** Ufuncs are instances of `numpy.ufunc`. They perform element-by-element operations on ndarrays, supporting broadcasting, type casting, and several other standard features. More than 150 ufuncs exist, covering a wide range of operations.

**Beginner-Friendly Explanation:** Think of a ufunc as a "vectorized" version of a mathematical function. When you call `np.sin(arr)`, NumPy doesn't loop through the array in Python—it uses a pre-compiled C loop that processes all elements at once. This makes ufuncs orders of magnitude faster than Python loops for large arrays.

### Purposes

- To provide highly optimized, C-level element-wise operations on arrays.
- To avoid the severe performance overhead of Python loops for numerical computations.
- To enable vectorized code that is both concise and fast.
- To support broadcasting, type casting, and other array features automatically.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
result = ufunc(*inputs, out=None, where=True, **kwargs)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `ufunc` | The universal function (e.g., `np.add`, `np.sin`). |
| `*inputs` | One or more input arrays or scalars. |
| `out` | Optional output array(s). |
| `where` | Boolean condition for selective computation. |
| `**kwargs` | Additional keyword arguments (e.g., `dtype`, `casting`). |

**Syntax Rules:**

- Ufuncs accept a fixed number of inputs and produce a fixed number of outputs.
- All inputs must be broadcastable to a common shape.
- Ufuncs are called with positional inputs and optional keyword arguments.

**Constraints and Limitations:**

- Ufuncs do not accept arbitrary keyword arguments as inputs; only `out`, `where`, `dtype`, `casting`, `order`, `subok`, and `signature` are recognized.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Basic Ufunc Usage**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create an array.
arr = np.array([0, np.pi/2, np.pi])
print("Input array:", arr)
print()

# Step 3: Apply np.sin (a unary ufunc).
result = np.sin(arr)
print("np.sin(arr):", result)
print()

# Step 4: Apply np.add (a binary ufunc).
a = np.array([1, 2, 3])
b = np.array([10, 20, 30])
sum_result = np.add(a, b)
print("np.add(a, b):", sum_result)
print()

# Step 5: Inspect ufunc properties.
print("np.add.ntypes:", np.add.ntypes)
print("np.sin.nin:", np.sin.nin)
print("np.add.nout:", np.add.nout)
```

**Expected Output:**

```
Input array: [0.         1.57079633 3.14159265]

np.sin(arr): [0.0000000e+00 1.0000000e+00 1.2246468e-16]

np.add(a, b): [11 22 33]

np.add.ntypes: 18
np.sin.nin: 1
np.add.nout: 1
```

**Why This Result Occurs:** `np.sin` computes the sine of each element. `np.add` adds corresponding elements. The `ntypes` attribute shows that `np.add` supports 18 different numeric types. `nin` and `nout` indicate the number of inputs and outputs.

### Real-World Cases

- **Signal Processing:** Applying trigonometric functions to waveform data.
- **Data Normalization:** Using `np.exp` or `np.log` for transformations.
- **Physics Simulations:** Computing element-wise physical quantities.

### References

- Universal functions (ufunc) basics – https://numpy.org/doc/stable/user/basics.ufuncs.html
- numpy.ufunc – https://numpy.org/doc/stable/reference/generated/numpy.ufunc.html

---

## Core Concept 2: Element-Wise Execution

### Definitions

**Core Definition:** Element-wise execution means that a ufunc applies its operation independently to each corresponding element of the input arrays.

**Technical Definition:** Ufuncs evaluate type signatures (`ufunc.types`) to determine the appropriate compiled loop for the input dtypes. During execution, type coercion and promotion are applied automatically to find a common dtype. The result is a new array (or a view if `out` is provided) with the broadcast shape.

**Beginner-Friendly Explanation:** When you apply a ufunc to an array, it performs the operation on each element separately. For example, `np.sqrt([1, 4, 9])` computes the square root of 1, 4, and 9 individually, returning `[1, 2, 3]`. The ufunc decides which internal C loop to use based on the input types.

### Purposes

- To apply mathematical operations uniformly across all elements of an array.
- To leverage type-specific optimized loops for maximum performance.
- To handle mixed-type inputs through automatic type promotion.
- To enable parallel processing of array elements at the hardware level.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
result = ufunc(x1, x2, ..., dtype=None, casting='same_kind')
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `ufunc.types` | List of supported type signatures (e.g., `'ff->f'`, `'dd->d'`). |
| `dtype` | Optional target dtype for the computation. |
| `casting` | Casting safety mode. |

**Syntax Rules:**

- The ufunc selects the appropriate type signature based on input dtypes.
- If no exact signature matches, type promotion finds the best fit.
- The result dtype follows the selected signature.

**Constraints and Limitations:**

- Some ufuncs have limited type support (e.g., `np.exp` supports only floating-point types).
- Type promotion may upcast to a larger dtype than expected.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Type Signatures and Promotion**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Inspect type signatures for np.add.
print("np.add.types (first 10):", np.add.types[:10])
print()

# Step 3: Apply to different dtypes and observe promotion.
a_int32 = np.array([1, 2, 3], dtype=np.int32)
b_float64 = np.array([1.5, 2.5, 3.5], dtype=np.float64)
result = np.add(a_int32, b_float64)
print("int32 + float64 ->", result.dtype)
print("Result:", result)
print()

# Step 4: Force a specific dtype.
forced = np.add(a_int32, b_float64, dtype=np.float32)
print("Forced dtype float32:", forced.dtype)
```

**Expected Output:**

```
np.add.types (first 10): ['??->?', 'bb->b', 'BB->B', 'hh->h', 'HH->H', 'ii->i', 'II->I', 'll->l', 'LL->L', 'qq->q']

int32 + float64 -> float64
Result: [2.5 4.5 6.5]

Forced dtype float32: float32
```

**Why This Result Occurs:** The type signatures show that `np.add` supports many combinations. When adding `int32` and `float64`, NumPy promotes to `float64`, the smallest type that can represent both. Forcing `dtype=np.float32` overrides the promotion.

### Real-World Cases

- **Mixed-Type Data:** Adding integer counts to float measurements.
- **Precision Control:** Forcing `float32` for memory efficiency in neural networks.
- **Hardware Optimization:** Using type signatures that match native CPU widths.

### References

- numpy.ufunc.types – https://numpy.org/doc/stable/reference/generated/numpy.ufunc.types.html

---

## Core Concept 3: Unary Ufuncs

### Definitions

**Core Definition:** A unary ufunc takes a single input array and returns a single output array, applying the operation element-wise.

**Technical Definition:** Unary ufuncs have `nin=1` and `nout=1`. They include trigonometric functions (`np.sin`, `np.cos`, `np.tan`), exponential and logarithmic functions (`np.exp`, `np.log`, `np.sqrt`), absolute value (`np.abs`), and many others.

**Beginner-Friendly Explanation:** A unary ufunc is a function that takes one array and gives you back one array. For example, `np.sqrt(arr)` takes the square root of every element. `np.abs(arr)` gives the absolute value of every element.

### Purposes

- To apply mathematical transformations to each element of an array.
- To compute trigonometric, exponential, and logarithmic values.
- To compute absolute values, signs, and rounding operations.
- To implement activation functions in neural networks.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
result = unary_ufunc(input_array, out=None, where=True)
```

**Component Breakdown:**

| Ufunc | Description |
|-------|-------------|
| `np.sin` | Trigonometric sine. |
| `np.cos` | Trigonometric cosine. |
| `np.exp` | Exponential (e^x). |
| `np.log` | Natural logarithm. |
| `np.sqrt` | Square root. |
| `np.abs` | Absolute value. |
| `np.negative` | Negation. |
| `np.reciprocal` | Reciprocal (1/x). |

**Syntax Rules:**

- Unary ufuncs accept a single input array.
- The output has the same shape as the input.
- Domain errors (e.g., `np.sqrt(-1)`) produce `nan` with a warning.

**Constraints and Limitations:**

- Some unary ufuncs only support floating-point types.
- Domain errors produce `nan` and a `RuntimeWarning`.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Common Unary Ufuncs**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create an array.
x = np.array([1.0, 4.0, 9.0, 16.0])
print("Input:", x)
print()

# Step 3: Apply np.sqrt.
print("np.sqrt(x):", np.sqrt(x))
print()

# Step 4: Apply np.exp.
print("np.exp(x):", np.exp(x))
print()

# Step 5: Apply np.log.
print("np.log(x):", np.log(x))
print()

# Step 6: Apply np.abs to negative values.
y = np.array([-3, -1, 0, 1, 3])
print("np.abs(y):", np.abs(y))
```

**Expected Output:**

```
Input: [ 1.  4.  9. 16.]

np.sqrt(x): [1. 2. 3. 4.]

np.exp(x): [2.71828183e+00 5.45981500e+01 8.10308393e+03 8.88611052e+06]

np.log(x): [0.         1.38629436 2.19722458 2.77258872]

np.abs(y): [3 1 0 1 3]
```

**Why This Result Occurs:** `np.sqrt` computes the square root of each element. `np.exp` computes e raised to each power. `np.log` computes the natural logarithm. `np.abs` returns the absolute value.

### Real-World Cases

- **Neural Networks:** Using `np.exp` for softmax and `np.log` for cross-entropy loss.
- **Signal Processing:** Using `np.sin` and `np.cos` for waveform generation.
- **Data Preprocessing:** Using `np.sqrt` for variance stabilization.

### References

- Universal functions (ufunc) basics – https://numpy.org/doc/stable/user/basics.ufuncs.html
- NumPy Mathematical functions – https://numpy.org/doc/stable/reference/routines.math.html

---

## Core Concept 4: Binary Ufuncs

### Definitions

**Core Definition:** A binary ufunc takes two input arrays and returns a single output array, applying the operation element-wise to corresponding pairs.

**Technical Definition:** Binary ufuncs have `nin=2` and `nout=1`. They include arithmetic operations (`np.add`, `np.subtract`, `np.multiply`, `np.divide`), comparison operations (`np.maximum`, `np.minimum`), and special functions (`np.hypot`, `np.arctan2`, `np.power`).

**Beginner-Friendly Explanation:** A binary ufunc takes two arrays and combines them element by element. For example, `np.maximum(a, b)` returns an array where each element is the larger of the corresponding elements from `a` and `b`.

### Purposes

- To combine two arrays element-wise (addition, subtraction, etc.).
- To compute element-wise maximum or minimum.
- To perform element-wise comparisons and logical operations.
- To implement distance and angle calculations.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
result = binary_ufunc(array1, array2, out=None, where=True)
```

**Component Breakdown:**

| Ufunc | Description |
|-------|-------------|
| `np.add` | Element-wise addition. |
| `np.subtract` | Element-wise subtraction. |
| `np.multiply` | Element-wise multiplication. |
| `np.divide` | Element-wise division. |
| `np.maximum` | Element-wise maximum. |
| `np.minimum` | Element-wise minimum. |
| `np.hypot` | Hypotenuse (sqrt(x² + y²)). |
| `np.arctan2` | Element-wise arctangent of y/x. |

**Syntax Rules:**

- Both inputs must be broadcastable to a common shape.
- The output has the broadcast shape of the inputs.
- Binary ufuncs can be called with the corresponding operator (e.g., `a + b` for `np.add`).

**Constraints and Limitations:**

- Broadcasting rules apply; incompatible shapes raise a `ValueError`.
- Some binary ufuncs (e.g., `np.maximum`) do not have operator equivalents.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Binary Ufuncs in Action**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create two arrays.
a = np.array([1, 4, 9])
b = np.array([2, 3, 5])
print("a:", a)
print("b:", b)
print()

# Step 3: Element-wise maximum.
print("np.maximum(a, b):", np.maximum(a, b))
print()

# Step 4: Element-wise minimum.
print("np.minimum(a, b):", np.minimum(a, b))
print()

# Step 5: Hypotenuse.
print("np.hypot(3, 4):", np.hypot(3, 4))
print()

# Step 6: Broadcasting with a scalar.
print("np.maximum(a, 5):", np.maximum(a, 5))
```

**Expected Output:**

```
a: [1 4 9]
b: [2 3 5]

np.maximum(a, b): [2 4 9]

np.minimum(a, b): [1 3 5]

np.hypot(3, 4): 5.0

np.maximum(a, 5): [5 5 9]
```

**Why This Result Occurs:** `np.maximum` selects the larger of each pair. `np.minimum` selects the smaller. `np.hypot(3, 4)` computes `sqrt(3² + 4²) = 5`. Broadcasting the scalar 5 applies the maximum operation against every element of `a`.

### Real-World Cases

- **Image Processing:** Computing pixel-wise maximum for image blending.
- **Physics:** Computing resultant vectors with `np.hypot`.
- **Neural Networks:** Using `np.maximum` for ReLU activation.

### References

- Universal functions (ufunc) basics – https://numpy.org/doc/stable/user/basics.ufuncs.html
- NumPy Mathematical functions – https://numpy.org/doc/stable/reference/routines.math.html

---

## Core Concept 5: Advanced Ufunc Methods

### Definitions

**Core Definition:** Advanced ufunc methods are built-in operations (`reduce`, `accumulate`, `outer`, `at`) that extend ufuncs beyond simple element-wise application, enabling reductions, cumulative operations, outer products, and in-place indexed updates.

**Technical Definition:** All ufuncs have five methods: `reduce`, `accumulate`, `reduceat`, `outer`, and `at`. These methods leverage the same compiled C loops as the base ufunc but apply them in specialized patterns.

**Beginner-Friendly Explanation:** Ufunc methods let you do more than just apply a function element-wise. You can use them to reduce an array to a single value (like summing all elements), compute running totals, create outer products, or update specific elements in place.

### Purposes

- To perform reductions without needing separate reduction functions.
- To compute cumulative or running results.
- To generate outer products and pair-wise combinations.
- To perform in-place operations with advanced indexing.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Reduce
result = ufunc.reduce(array, axis=0, dtype=None, out=None)

# Accumulate
result = ufunc.accumulate(array, axis=0, dtype=None, out=None)

# Outer
result = ufunc.outer(A, B)

# At
ufunc.at(a, indices, b=None)
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `reduce` | Applies the ufunc repeatedly to reduce dimensions. |
| `accumulate` | Returns intermediate results of the reduction. |
| `reduceat` | Performs reduce over specified slices. |
| `outer` | Applies the ufunc to all pairs of elements. |
| `at` | Performs unbuffered in-place operation. |

**Syntax Rules:**

- `reduce` and `accumulate` accept an `axis` parameter.
- `outer` returns an array of dimension `M + N` for inputs of dimensions `M` and `N`.
- `at` performs unbuffered in-place operations and handles repeated indices correctly.

**Constraints and Limitations:**

- `reduce` requires a binary ufunc (two inputs).
- `outer` is only supported for binary ufuncs.
- `at` does not buffer, which is its key advantage for repeated indices.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Using reduce and accumulate**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create an array.
a = np.arange(1, 6)
print("Array:", a)
print()

# Step 3: np.add.reduce (sum).
print("np.add.reduce(a):", np.add.reduce(a))
print("  (equivalent to np.sum(a))")
print()

# Step 4: np.multiply.reduce (product).
print("np.multiply.reduce(a):", np.multiply.reduce(a))
print()

# Step 5: np.add.accumulate (cumulative sum).
print("np.add.accumulate(a):", np.add.accumulate(a))
print("  (equivalent to np.cumsum(a))")
print()

# Step 6: np.multiply.accumulate (cumulative product).
print("np.multiply.accumulate(a):", np.multiply.accumulate(a))
```

**Expected Output:**

```
Array: [1 2 3 4 5]

np.add.reduce(a): 15
  (equivalent to np.sum(a))

np.multiply.reduce(a): 120

np.add.accumulate(a): [ 1  3  6 10 15]
  (equivalent to np.cumsum(a))

np.multiply.accumulate(a): [  1   2   6  24 120]
```

**Why This Result Occurs:** `reduce` applies the operation repeatedly to collapse the array to a single value. `accumulate` returns the intermediate results at each step. `np.add.reduce(a)` computes `1+2+3+4+5 = 15`. `np.add.accumulate(a)` produces `[1, 1+2, 1+2+3, ...]`.

**Example 2: Using outer and at**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Compute outer product.
a = np.array([1, 2, 3])
b = np.array([10, 20])
outer_result = np.multiply.outer(a, b)
print("np.multiply.outer(a, b):\n", outer_result)
print("Shape:", outer_result.shape)
print()

# Step 3: Demonstrate at with repeated indices.
arr = np.zeros(5)
np.add.at(arr, [0, 2, 2, 4], 1)
print("After np.add.at(arr, [0, 2, 2, 4], 1):", arr)
print("  (index 2 is incremented twice)")
print()

# Step 4: Compare with regular indexing.
arr2 = np.zeros(5)
arr2[[0, 2, 2, 4]] += 1
print("After arr2[[0, 2, 2, 4]] += 1:", arr2)
print("  (index 2 is only incremented once due to buffering)")
```

**Expected Output:**

```
np.multiply.outer(a, b):
 [[10 20]
 [20 40]
 [30 60]]
Shape: (3, 2)

After np.add.at(arr, [0, 2, 2, 4], 1): [1. 0. 2. 0. 1.]
  (index 2 is incremented twice)

After arr2[[0, 2, 2, 4]] += 1: [1. 0. 1. 0. 1.]
  (index 2 is only incremented once due to buffering)
```

**Why This Result Occurs:** `np.multiply.outer(a, b)` computes all pairwise products, producing a 3×2 array. `np.add.at` performs unbuffered in-place addition, so when index 2 appears twice, it is incremented twice (resulting in 2). Regular indexing with `+=` buffers the operation and only applies it once to each unique index.

### Real-World Cases

- **Statistics:** Using `reduce` to compute sums, products, and other aggregates.
- **Finance:** Using `accumulate` for cumulative returns.
- **Linear Algebra:** Using `outer` for outer products.
- **Histogramming:** Using `at` to accumulate counts at specific indices.

### References

- numpy.ufunc.reduce – https://numpy.org/doc/stable/reference/generated/numpy.ufunc.reduce.html
- numpy.ufunc.accumulate – https://numpy.org/doc/stable/reference/generated/numpy.ufunc.accumulate.html
- numpy.ufunc.outer – https://numpy.org/doc/stable/reference/generated/numpy.ufunc.outer.html
- numpy.ufunc.at – https://numpy.org/doc/stable/reference/generated/numpy.ufunc.at.html

---

## Core Concept 6: The out Parameter

### Definitions

**Core Definition:** The `out` parameter allows you to specify a pre-allocated array where the ufunc's result should be stored, avoiding the creation of a new array.

**Technical Definition:** The `out` keyword argument is expected to be a tuple with one entry per output (which can be `None` for arrays to be allocated by the ufunc). For ufuncs with a single output, passing a single array (instead of a tuple) is deprecated. The `out` array must have the correct shape and dtype to receive the result.

**Beginner-Friendly Explanation:** Normally, when you call a ufunc, it creates a new array for the result. If you're doing many operations and want to save memory, you can pre-allocate an array and tell the ufunc to write its result there using `out=`. This avoids creating temporary arrays and can significantly reduce memory usage.

### Purposes

- To avoid temporary array allocations in memory-intensive computations.
- To reuse existing memory buffers for repeated operations.
- To improve performance by reducing memory allocation overhead.
- To write results directly into views or slices of larger arrays.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
ufunc(*inputs, out=output_array)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `out` | Pre-allocated output array (or tuple of arrays for multiple outputs). |
| `where` | Boolean condition; only elements where `where` is `True` are computed. |

**Syntax Rules:**

- The `out` array must have the correct shape and a dtype that can hold the result.
- If `out` is a tuple, it must have one entry per output.
- For single-output ufuncs, passing a single array is allowed but deprecated.
- Using `out=...` (Ellipsis) forces an array result even for 0-d results.

**Constraints and Limitations:**

- The `out` array's dtype must be compatible with the computation.
- Using `out` with an array that overlaps with the input can produce incorrect results.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Using out to Avoid Temporary Allocations**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create input arrays.
a = np.array([1.0, 2.0, 3.0])
b = np.array([4.0, 5.0, 6.0])
print("a:", a)
print("b:", b)
print()

# Step 3: Pre-allocate the output array.
result = np.empty_like(a)
print("Pre-allocated result:", result)
print()

# Step 4: Use out parameter.
np.add(a, b, out=result)
print("After np.add(a, b, out=result):", result)
print()

# Step 5: Demonstrate where parameter.
mask = np.array([True, False, True])
np.multiply(a, b, out=result, where=mask)
print("After np.multiply(a, b, out=result, where=mask):", result)
```

**Expected Output:**

```
a: [1. 2. 3.]
b: [4. 5. 6.]

Pre-allocated result: [0. 0. 0.]

After np.add(a, b, out=result): [5. 7. 9.]

After np.multiply(a, b, out=result, where=mask): [ 4.  7. 18.]
```

**Why This Result Occurs:** The `out` parameter directs the result into the pre-allocated `result` array. The `where` parameter computes the operation only where the mask is `True`; elsewhere, the existing value in `result` is preserved (7.0 at index 1, which was the result of the previous addition).

### Real-World Cases

- **Memory-Constrained Environments:** Reusing buffers in embedded systems or GPU memory.
- **Iterative Algorithms:** Updating arrays in place across iterations.
- **Large-Scale Data Processing:** Avoiding temporary arrays in pipelines.

### References

- Universal functions (ufunc) – https://numpy.org/doc/stable/reference/ufuncs.html
- numpy.ufunc – https://numpy.org/doc/stable/reference/generated/numpy.ufunc.html

---

## Summary of Ufunc Methods

| Method | Description | Equivalent To |
|--------|-------------|---------------|
| `reduce` | Collapse array by repeated application. | `np.sum`, `np.prod` |
| `accumulate` | Return intermediate results. | `np.cumsum`, `np.cumprod` |
| `reduceat` | Reduce over specified slices. | — |
| `outer` | Apply to all pairs. | `np.outer` |
| `at` | Unbuffered in-place operation. | `a[indices] += b` (buffered) |

---

## Consolidated Reference List

- Universal functions (ufunc) basics – https://numpy.org/doc/stable/user/basics.ufuncs.html
- numpy.ufunc – https://numpy.org/doc/stable/reference/generated/numpy.ufunc.html
- numpy.ufunc.types – https://numpy.org/doc/stable/reference/generated/numpy.ufunc.types.html
- numpy.ufunc.reduce – https://numpy.org/doc/stable/reference/generated/numpy.ufunc.reduce.html
- numpy.ufunc.accumulate – https://numpy.org/doc/stable/reference/generated/numpy.ufunc.accumulate.html
- numpy.ufunc.outer – https://numpy.org/doc/stable/reference/generated/numpy.ufunc.outer.html
- numpy.ufunc.at – https://numpy.org/doc/stable/reference/generated/numpy.ufunc.at.html
- NumPy Mathematical functions – https://numpy.org/doc/stable/reference/routines.math.html
- NumPy Logic functions – https://numpy.org/doc/stable/reference/routines.logic.html