# NumPy Combining Arrays: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Combining arrays in NumPy is the set of operations that join two or more arrays into a single array, either along an existing axis or along a newly created axis.

**Technical Definition:** NumPy provides a family of array-joining routines that differ in how they align input arrays: `concatenate` joins along an existing axis, `stack` joins along a new axis (increasing dimensionality), and the convenience wrappers `vstack`, `hstack`, `dstack`, and `column_stack` apply stacking along specific axes. `block` assembles arrays from nested lists of blocks, similar to MATLAB's bracket notation. All routines return new arrays; they do not modify the inputs.

**Beginner-Friendly Explanation:** Combining arrays is like joining pieces of a puzzle. You can attach pieces side by side (horizontally), stack them on top of each other (vertically), or pile them in layers (depth-wise). Different functions handle different joining directions, and some—like `block`—let you build complex structures from smaller pieces.

### Key Characteristics

- **Axis-Based:** Every combining operation works along a specific axis (existing or new).
- **Shape Requirements:** Input arrays must have compatible shapes along all axes except the joining axis.
- **New Arrays:** All combining routines return new arrays; inputs are not modified.
- **Dimensionality:** `concatenate` preserves dimensionality; `stack` increases it by one.
- **Convenience Wrappers:** `vstack`, `hstack`, `dstack`, and `column_stack` are shortcuts for common concatenation/stacking patterns.

### Prerequisites

- NumPy installed (`pip install numpy`).
- Understanding of array shapes, axes, and dimensions.
- Familiarity with the `import numpy as np` convention.

### Related Programming Areas

- **Data Preparation:** Combining feature matrices, batching samples, and assembling datasets.
- **Image Processing:** Stacking color channels or combining image tiles.
- **Machine Learning:** Stacking batches, concatenating features, and building block matrices.
- **Scientific Computing:** Assembling simulation results from multiple runs or grid blocks.

### Core Concepts / Features

The following core concepts are explored in detail: (1) `concatenate()`, (2) `stack()`, (3) `vstack()`, (4) `hstack()`, (5) `dstack()`, (6) `column_stack()`, and (7) `block()`.

---

## Core Concept 1: concatenate()

### Definitions

**Core Definition:** `concatenate()` joins a sequence of arrays along an existing axis.

**Technical Definition:** `numpy.concatenate(arrays, axis=0, out=None, *, dtype=None, casting='same_kind')` joins a sequence of arrays along an existing axis. The arrays must have the same shape, except in the dimension corresponding to `axis` (the first, by default). If `axis` is `None`, arrays are flattened before use. New in version 2.0: `numpy.concat` was added as a shorthand for `numpy.concatenate`.

**Beginner-Friendly Explanation:** `concatenate()` is the most general way to join arrays. You tell it which direction to join them (the axis), and it attaches them end-to-end along that direction. For example, joining two matrices row-wise (axis=0) stacks one on top of the other.

### Purposes

- To join arrays along an existing axis without creating a new dimension.
- To combine feature matrices, data batches, or signal segments.
- To reassemble arrays that were previously split.
- To provide the foundational operation that other stacking functions are built upon.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
numpy.concatenate(arrays, axis=0, out=None, *, dtype=None, casting='same_kind')
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `arrays` | Sequence of array_like (tuple or list of arrays). |
| `axis` | Int, optional. Axis along which arrays are joined. Default 0. If `None`, arrays are flattened. |
| `out` | Ndarray, optional. Destination array. |
| `dtype` | Str or dtype, optional. Target dtype. New in 1.20.0. |
| `casting` | Casting mode. Default `'same_kind'`. New in 1.20.0. |

**Syntax Rules:**

- All arrays must have the same shape except along the concatenation axis.
- `axis` must be a valid axis index for the input arrays.
- If `axis=None`, all arrays are flattened to 1-D before concatenation.

**Constraints and Limitations:**

- Shapes must match exactly along all non-concatenation axes.
- Cannot concatenate 0-D arrays along a given axis (they must be at least 1-D).

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Concatenating Arrays Along Different Axes**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create two 2-D arrays.
a = np.array([[1, 2],
              [3, 4]])
b = np.array([[5, 6],
              [7, 8]])
print("Array a:\n", a)
print("Array b:\n", b)
print()

# Step 3: Concatenate along axis 0 (rows).
concat_axis0 = np.concatenate((a, b), axis=0)
print("concatenate((a, b), axis=0):\n", concat_axis0)
print("Shape:", concat_axis0.shape)
print("  (rows stacked: 4x2)")
print()

# Step 4: Concatenate along axis 1 (columns).
concat_axis1 = np.concatenate((a, b), axis=1)
print("concatenate((a, b), axis=1):\n", concat_axis1)
print("Shape:", concat_axis1.shape)
print("  (columns stacked: 2x4)")
print()

# Step 5: Concatenate with axis=None (flatten first).
concat_flat = np.concatenate((a, b), axis=None)
print("concatenate((a, b), axis=None):", concat_flat)
print("Shape:", concat_flat.shape)
```

**Expected Output:**

```
Array a:
 [[1 2]
 [3 4]]
Array b:
 [[5 6]
 [7 8]]

concatenate((a, b), axis=0):
 [[1 2]
 [3 4]
 [5 6]
 [7 8]]
Shape: (4, 2)
  (rows stacked: 4x2)

concatenate((a, b), axis=1):
 [[1 2 5 6]
 [3 4 7 8]]
Shape: (2, 4)
  (columns stacked: 2x4)

concatenate((a, b), axis=None): [1 2 3 4 5 6 7 8]
Shape: (8,)
```

**Why This Result Occurs:** With `axis=0`, the arrays are stacked vertically (one on top of the other), producing a 4×2 array. With `axis=1`, they are stacked horizontally (side by side), producing a 2×4 array. With `axis=None`, both arrays are flattened to 1-D and then concatenated into a single 1-D array of 8 elements.

### Real-World Cases

- **Batch Combination:** Merging two batches of data before feeding them into a model.
- **Feature Concatenation:** Joining separate feature sets (e.g., demographic and behavioral features) into a single matrix.
- **Time Series Assembly:** Stitching together multiple time series segments into one continuous series.

### References

- numpy.concatenate – https://numpy.org/doc/stable/reference/generated/numpy.concatenate.html

---

## Core Concept 2: stack()

### Definitions

**Core Definition:** `stack()` joins a sequence of arrays along a new axis, increasing the dimensionality of the result.

**Technical Definition:** `numpy.stack(arrays, axis=0, out=None, *, dtype=None, casting='same_kind')` joins a sequence of arrays along a new axis. The `axis` parameter specifies the index of the new axis in the dimensions of the result. All input arrays must have the same shape. The stacked array has one more dimension than the input arrays.

**Beginner-Friendly Explanation:** `stack()` is like `concatenate()`, but instead of joining along an existing direction, it creates a brand-new direction. If you stack two 1-D arrays of length 3, you get a 2-D array of shape (2, 3)—a new "batch" dimension is created.

### Purposes

- To create a new axis for combining arrays (e.g., adding a batch dimension).
- To combine arrays of identical shape into a higher-dimensional structure.
- To prepare data for algorithms that expect batch or channel dimensions.
- To provide a more flexible alternative to `concatenate()` when a new dimension is needed.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
numpy.stack(arrays, axis=0, out=None, *, dtype=None, casting='same_kind')
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `arrays` | Sequence of ndarrays. All must have the same shape. |
| `axis` | Int, optional. Index of the new axis in the result. Default 0. |
| `out` | Ndarray, optional. Destination array. |
| `dtype` | Str or dtype, optional. New in 1.24. |
| `casting` | Casting mode. Default `'same_kind'`. New in 1.24. |

**Syntax Rules:**

- All input arrays must have exactly the same shape.
- `axis` specifies where the new axis is inserted: `axis=0` → first dimension, `axis=-1` → last dimension.
- The result has `input.ndim + 1` dimensions.

**Constraints and Limitations:**

- All arrays must have identical shapes; no broadcasting of shapes is performed.
- `stack` is the inverse of `unstack` (splitting along an axis).

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Stacking Along Different Axes**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create two 1-D arrays.
a = np.array([1, 2, 3])
b = np.array([4, 5, 6])
print("Array a:", a)
print("Array b:", b)
print()

# Step 3: Stack along axis 0 (default).
stack_axis0 = np.stack((a, b))
print("stack((a, b)) [axis=0]:\n", stack_axis0)
print("Shape:", stack_axis0.shape)
print("  (new first axis)")
print()

# Step 4: Stack along axis 1.
stack_axis1 = np.stack((a, b), axis=1)
print("stack((a, b), axis=1):\n", stack_axis1)
print("Shape:", stack_axis1.shape)
print("  (new second axis)")
print()

# Step 5: Stack along axis=-1 (last axis).
stack_axis_neg = np.stack((a, b), axis=-1)
print("stack((a, b), axis=-1):\n", stack_axis_neg)
print("Shape:", stack_axis_neg.shape)
```

**Expected Output:**

```
Array a: [1 2 3]
Array b: [4 5 6]

stack((a, b)) [axis=0]:
 [[1 2 3]
 [4 5 6]]
Shape: (2, 3)
  (new first axis)

stack((a, b), axis=1):
 [[1 4]
 [2 5]
 [3 6]]
Shape: (3, 2)
  (new second axis)

stack((a, b), axis=-1):
 [[1 4]
 [2 5]
 [3 6]]
Shape: (3, 2)
```

**Why This Result Occurs:** With `axis=0`, the new axis becomes the first dimension, so the result has shape (2, 3). With `axis=1`, the new axis becomes the second dimension, producing shape (3, 2). `axis=-1` is equivalent to `axis=1` for 1-D inputs, producing the same shape.

### Real-World Cases

- **Batch Creation:** Stacking individual samples into a batch for model training.
- **Channel Stacking:** Combining separate color channel arrays into a multi-channel image.
- **Ensemble Methods:** Stacking predictions from multiple models into a single array.

### References

- numpy.stack – https://numpy.org/doc/2.2/reference/generated/numpy.stack.html

---

## Core Concept 3: vstack()

### Definitions

**Core Definition:** `vstack()` stacks arrays vertically (row-wise) along the first axis.

**Technical Definition:** `numpy.vstack(tup, *, dtype=None, casting='same_kind')` stacks arrays in sequence vertically (row wise). This is equivalent to concatenation along the first axis after 1-D arrays of shape `(N,)` have been reshaped to `(1, N)`. The result is at least 2-D. It rebuilds arrays divided by `vsplit`.

**Beginner-Friendly Explanation:** `vstack()` stacks arrays on top of each other like layers in a cake. It always works along the first axis (rows), so the resulting array has more rows than the inputs.

### Purposes

- To stack arrays vertically (row-wise) in a simple, readable way.
- To combine data matrices by adding rows.
- To reassemble arrays that were split vertically with `vsplit`.
- To provide a more intuitive alternative to `concatenate(axis=0)`.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
numpy.vstack(tup, *, dtype=None, casting='same_kind')
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `tup` | Sequence of ndarrays. Arrays must have the same shape along all but the first axis. 1-D arrays must have the same length. |
| `dtype` | Optional target dtype. New in 1.24. |
| `casting` | Casting mode. Default `'same_kind'`. New in 1.24. |

**Syntax Rules:**

- 1-D arrays are reshaped to `(1, N)` before stacking.
- All arrays must have the same number of columns.
- The result is always at least 2-D.

**Constraints and Limitations:**

- Cannot stack arrays with different numbers of columns.
- For 0-D arrays, `vstack` raises an error.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Vertical Stacking of 1-D and 2-D Arrays**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Stack two 1-D arrays vertically.
a = np.array([1, 2, 3])
b = np.array([4, 5, 6])
vstack_1d = np.vstack((a, b))
print("vstack((a, b)) with 1-D arrays:\n", vstack_1d)
print("Shape:", vstack_1d.shape)
print()

# Step 3: Stack two 2-D arrays vertically.
c = np.array([[1], [2], [3]])
d = np.array([[4], [5], [6]])
vstack_2d = np.vstack((c, d))
print("vstack((c, d)) with 2-D arrays:\n", vstack_2d)
print("Shape:", vstack_2d.shape)
```

**Expected Output:**

```
vstack((a, b)) with 1-D arrays:
 [[1 2 3]
 [4 5 6]]
Shape: (2, 3)

vstack((c, d)) with 2-D arrays:
 [[1]
 [2]
 [3]
 [4]
 [5]
 [6]]
Shape: (6, 1)
```

**Why This Result Occurs:** The 1-D arrays are first reshaped to `(1, 3)` and then stacked along axis 0, producing a `(2, 3)` array. The 2-D arrays have shape `(3, 1)` and are stacked along axis 0 to produce a `(6, 1)` array.

### Real-World Cases

- **Adding Data Rows:** Appending new observations to an existing data matrix.
- **Combining Batches:** Stacking multiple batches of samples vertically.
- **Reassembling Splits:** Rebuilding an array after `vsplit`.

### References

- numpy.vstack – https://numpy.org/doc/stable/reference/generated/numpy.vstack.html

---

## Core Concept 4: hstack()

### Definitions

**Core Definition:** `hstack()` stacks arrays horizontally (column-wise) along the second axis (or first axis for 1-D arrays).

**Technical Definition:** `numpy.hstack(tup, *, dtype=None, casting='same_kind')` stacks arrays in sequence horizontally (column wise). This is equivalent to concatenation along the second axis, except for 1-D arrays where it concatenates along the first axis. It rebuilds arrays divided by `hsplit`.

**Beginner-Friendly Explanation:** `hstack()` joins arrays side by side, like attaching columns to a table. For 1-D arrays, it simply concatenates them into a longer 1-D array. For 2-D arrays, it adds columns.

### Purposes

- To stack arrays horizontally (column-wise) in a simple, readable way.
- To combine data matrices by adding columns.
- To reassemble arrays that were split horizontally with `hsplit`.
- To provide a more intuitive alternative to `concatenate(axis=1)`.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
numpy.hstack(tup, *, dtype=None, casting='same_kind')
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `tup` | Sequence of ndarrays. Arrays must have the same shape along all but the second axis, except 1-D arrays which can be any length. |
| `dtype` | Optional target dtype. New in 1.24. |
| `casting` | Casting mode. Default `'same_kind'`. New in 1.24. |

**Syntax Rules:**

- For 1-D arrays, `hstack` concatenates along the first axis.
- For arrays with 2+ dimensions, `hstack` concatenates along the second axis.
- All arrays must have the same number of rows (for 2-D inputs).

**Constraints and Limitations:**

- Cannot stack arrays with different numbers of rows (for 2-D inputs).
- For 0-D arrays, `hstack` raises an error.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Horizontal Stacking of 1-D and 2-D Arrays**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Stack two 1-D arrays horizontally.
a = np.array([1, 2, 3])
b = np.array([4, 5, 6])
hstack_1d = np.hstack((a, b))
print("hstack((a, b)) with 1-D arrays:", hstack_1d)
print("Shape:", hstack_1d.shape)
print()

# Step 3: Stack two 2-D arrays horizontally.
c = np.array([[1], [2], [3]])
d = np.array([[4], [5], [6]])
hstack_2d = np.hstack((c, d))
print("hstack((c, d)) with 2-D arrays:\n", hstack_2d)
print("Shape:", hstack_2d.shape)
```

**Expected Output:**

```
hstack((a, b)) with 1-D arrays: [1 2 3 4 5 6]
Shape: (6,)

hstack((c, d)) with 2-D arrays:
 [[1 4]
 [2 5]
 [3 6]]
Shape: (3, 2)
```

**Why This Result Occurs:** For 1-D arrays, `hstack` concatenates them along the first (and only) axis, producing a single 1-D array of length 6. For 2-D arrays, it concatenates along the second axis (columns), producing a 3×2 array.

### Real-World Cases

- **Feature Augmentation:** Adding new feature columns to an existing feature matrix.
- **Concatenating Time Series:** Joining two time series side by side for comparison.
- **Image Tiling:** Arranging image tiles horizontally.

### References

- numpy.hstack – https://numpy.org/doc/stable/reference/generated/numpy.hstack.html

---

## Core Concept 5: dstack()

### Definitions

**Core Definition:** `dstack()` stacks arrays depth-wise along the third axis (axis 2).

**Technical Definition:** `numpy.dstack(tup)` stacks arrays in sequence depth wise (along third axis). This is equivalent to concatenation along the third axis after 2-D arrays of shape `(M, N)` have been reshaped to `(M, N, 1)` and 1-D arrays of shape `(N,)` have been reshaped to `(1, N, 1)`. It rebuilds arrays divided by `dsplit`.

**Beginner-Friendly Explanation:** `dstack()` stacks arrays in layers, one behind the other. If you have several 2-D images, `dstack` combines them into a 3-D array where the third dimension represents the different images.

### Purposes

- To stack arrays along the depth (third) axis.
- To combine 2-D arrays into a 3-D volume.
- To reassemble arrays that were split with `dsplit`.
- To prepare multi-channel data for processing.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
numpy.dstack(tup)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `tup` | Sequence of arrays. Arrays must have the same shape along all but the third axis. 1-D or 2-D arrays must have the same shape. |

**Syntax Rules:**

- 1-D arrays of shape `(N,)` are reshaped to `(1, N, 1)`.
- 2-D arrays of shape `(M, N)` are reshaped to `(M, N, 1)`.
- The result is always at least 3-D.

**Constraints and Limitations:**

- All input arrays must have the same shape (for 1-D and 2-D inputs).
- For arrays with more than 3 dimensions, `dstack` may not behave as expected.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Depth Stacking of 1-D and 2-D Arrays**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Stack two 1-D arrays depth-wise.
a = np.array([1, 2, 3])
b = np.array([4, 5, 6])
dstack_1d = np.dstack((a, b))
print("dstack((a, b)) with 1-D arrays:\n", dstack_1d)
print("Shape:", dstack_1d.shape)
print()

# Step 3: Stack two 2-D arrays depth-wise.
c = np.array([[1], [2], [3]])
d = np.array([[4], [5], [6]])
dstack_2d = np.dstack((c, d))
print("dstack((c, d)) with 2-D arrays:\n", dstack_2d)
print("Shape:", dstack_2d.shape)
```

**Expected Output:**

```
dstack((a, b)) with 1-D arrays:
 [[[1 4]
  [2 5]
  [3 6]]]
Shape: (1, 3, 2)

dstack((c, d)) with 2-D arrays:
 [[[1 4]]

 [[2 5]]

 [[3 6]]]
Shape: (3, 1, 2)
```

**Why This Result Occurs:** For 1-D arrays, `dstack` reshapes each to `(1, 3, 1)` and then concatenates along the third axis, producing shape `(1, 3, 2)`. For 2-D arrays of shape `(3, 1)`, each is reshaped to `(3, 1, 1)` and concatenated along the third axis, producing shape `(3, 1, 2)`.

### Real-World Cases

- **Multi-Channel Images:** Combining separate color channels (red, green, blue) into a 3-D image array.
- **Volume Data:** Stacking 2-D slices into a 3-D volume for medical imaging.
- **Time Series Channels:** Combining multiple sensor readings into a depth-stacked array.

### References

- numpy.dstack – https://numpy.org/doc/stable/reference/generated/numpy.dstack.html

---

## Core Concept 6: column_stack()

### Definitions

**Core Definition:** `column_stack()` stacks 1-D arrays as columns into a 2-D array.

**Technical Definition:** `numpy.column_stack(tup)` takes a sequence of 1-D arrays and stacks them as columns to make a single 2-D array. 2-D arrays are stacked as-is, just like with `hstack`. 1-D arrays are turned into 2-D columns first. All arrays must have the same first dimension.

**Beginner-Friendly Explanation:** `column_stack()` is like creating a table from a list of columns. You give it several 1-D arrays (each representing a column of data), and it arranges them side by side into a 2-D matrix.

### Purposes

- To combine multiple 1-D arrays into a single 2-D matrix with each array as a column.
- To build feature matrices from separate feature vectors.
- To create tabular data from columnar sources.
- To provide a more intuitive alternative to `hstack` for 1-D arrays when a 2-D result is desired.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
numpy.column_stack(tup)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `tup` | Sequence of 1-D or 2-D arrays. All must have the same first dimension. |

**Syntax Rules:**

- 1-D arrays of shape `(N,)` are reshaped to `(N, 1)` before stacking.
- 2-D arrays are stacked as-is (equivalent to `hstack`).
- The result is always 2-D.

**Constraints and Limitations:**

- All input arrays must have the same first dimension (same number of rows).
- For 1-D arrays, the result has shape `(N, len(tup))`.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Column Stacking of 1-D Arrays**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create three 1-D arrays representing columns.
a = np.array([1, 2, 3])
b = np.array([4, 5, 6])
c = np.array([7, 8, 9])
print("Array a:", a)
print("Array b:", b)
print("Array c:", c)
print()

# Step 3: Column-stack them into a 2-D matrix.
result = np.column_stack((a, b, c))
print("column_stack((a, b, c)):\n", result)
print("Shape:", result.shape)
print("  (3 rows, 3 columns)")
print()

# Step 4: Compare with hstack on 1-D arrays.
hstack_result = np.hstack((a, b, c))
print("hstack((a, b, c)):", hstack_result)
print("Shape:", hstack_result.shape)
print("  (hstack produces 1-D, column_stack produces 2-D)")
```

**Expected Output:**

```
Array a: [1 2 3]
Array b: [4 5 6]
Array c: [7 8 9]

column_stack((a, b, c)):
 [[1 4 7]
 [2 5 8]
 [3 6 9]]
Shape: (3, 3)
  (3 rows, 3 columns)

hstack((a, b, c)): [1 2 3 4 5 6 7 8 9]
Shape: (9,)
  (hstack produces 1-D, column_stack produces 2-D)
```

**Why This Result Occurs:** `column_stack` reshapes each 1-D array to a column vector `(3, 1)` and then stacks them horizontally, producing a 3×3 matrix. `hstack` simply concatenates the 1-D arrays into a longer 1-D array. The key difference is that `column_stack` always produces a 2-D result, while `hstack` preserves the dimensionality of 1-D inputs.

### Real-World Cases

- **Feature Matrix Construction:** Combining separate feature vectors (age, income, score) into a single feature matrix.
- **Data Table Creation:** Building a table from columnar data sources.
- **Coordinate Arrays:** Stacking x, y, and z coordinate arrays into a single (N, 3) matrix.

### References

- numpy.column_stack – https://numpy.org/doc/2.4/reference/generated/numpy.column_stack.html

---

## Core Concept 7: block()

### Definitions

**Core Definition:** `block()` assembles an nd-array from nested lists of blocks.

**Technical Definition:** `numpy.block(arrays)` assembles an nd-array from nested lists of blocks. Blocks in the innermost lists are concatenated along the last dimension (-1), then these are concatenated along the second-last dimension (-2), and so on until the outermost list is reached. Blocks can be of any dimension, but will not be broadcasted using the normal rules. Instead, leading axes of size 1 are inserted, to make `block.ndim` the same for all blocks. This is primarily useful for working with scalars. New in version 1.13.0.

**Beginner-Friendly Explanation:** `block()` lets you build complex arrays from smaller pieces, like assembling a mosaic from tiles. You provide a nested list structure that describes the layout, and `block()` pieces everything together. For example, you can build a 2×2 block matrix from four smaller matrices.

### Purposes

- To assemble complex arrays from smaller sub-blocks using a MATLAB-like syntax.
- To build block matrices from their components.
- To combine arrays of different dimensions in a structured layout.
- To provide a more flexible alternative to repeated `concatenate` calls.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
numpy.block(arrays)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `arrays` | Nested list of array_like or scalars (but not tuples). |
| Innermost lists | Concatenated along the last dimension. |
| Outer lists | Concatenated along successively earlier dimensions. |

**Syntax Rules:**

- The depth of the nested list determines the number of dimensions.
- All blocks must have compatible shapes along the appropriate axes.
- Leading axes of size 1 are automatically inserted to match dimensions.
- When called with only scalars, `np.block` is equivalent to an `ndarray` call.

**Constraints and Limitations:**

- List depths must be consistent; mismatched depths raise `ValueError`.
- Empty lists raise `ValueError`.
- The function does not enforce that blocks lie on a fixed grid.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Building a Block Matrix**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create four smaller matrices (blocks).
A = np.array([[1, 1], [1, 1]])
B = np.array([[2, 2], [2, 2]])
C = np.array([[3, 3], [3, 3]])
D = np.array([[4, 4], [4, 4]])

# Step 3: Assemble a 2x2 block matrix.
block_matrix = np.block([[A, B], [C, D]])

print("Block matrix from [[A, B], [C, D]]:\n", block_matrix)
print("Shape:", block_matrix.shape)
print()

# Step 4: Build a block matrix from scalars and arrays.
v = np.array([1, 2, 3])
mixed_block = np.block([v, 1])

print("np.block([v, 1]) where v is 1-D:\n", mixed_block)
print("Shape:", mixed_block.shape)
print()

# Step 5: Use block to create a larger structure.
result = np.block([[np.eye(2), np.zeros((2, 2))],
                   [np.ones((2, 2)), np.eye(2) * 5]])

print("Block matrix with eye, zeros, and ones:\n", result)
```

**Expected Output:**

```
Block matrix from [[A, B], [C, D]]:
 [[1 1 2 2]
 [1 1 2 2]
 [3 3 4 4]
 [3 3 4 4]]
Shape: (4, 4)

np.block([v, 1]) where v is 1-D:
 [1 2 3 1]
Shape: (4,)

Block matrix with eye, zeros, and ones:
 [[1. 0. 0. 0.]
 [0. 1. 0. 0.]
 [1. 1. 5. 0.]
 [1. 1. 0. 5.]]
```

**Why This Result Occurs:** For the block matrix `[[A, B], [C, D]]`, `block` first concatenates A and B along the last axis (columns), and C and D along the last axis. Then it concatenates the two resulting rows along the second-to-last axis (rows), producing a 4×4 matrix. For `[v, 1]`, the scalar 1 is treated as a 0-D block and is broadcast to match the 1-D shape. In the final example, the 2×2 identity, zeros, ones, and scaled identity blocks are assembled into a 4×4 matrix.

### Real-World Cases

- **Block Matrix Construction:** Building large matrices from smaller sub-matrices in linear algebra.
- **Image Mosaics:** Assembling an image from smaller tile blocks.
- **Structured Data Layouts:** Creating complex data layouts from simpler components.

### References

- numpy.block – https://numpy.org/doc/2.0/reference/generated/numpy.block.html

---

## Comparison Summary

| Function | Joins Along | Dimensionality Change | Best For |
|----------|-------------|----------------------|----------|
| `concatenate` | Existing axis | Same | General-purpose joining |
| `stack` | New axis | +1 dimension | Adding batch/channel dimensions |
| `vstack` | Axis 0 (rows) | ≥2-D | Vertical stacking |
| `hstack` | Axis 1 (columns) | Same (or 1-D) | Horizontal stacking |
| `dstack` | Axis 2 (depth) | ≥3-D | Depth stacking |
| `column_stack` | Columns (1-D→2-D) | 1-D → 2-D | Building tables from vectors |
| `block` | Nested structure | Depends on nesting | Block matrices, mosaics |

---

## Consolidated Reference List

- numpy.concatenate – https://numpy.org/doc/stable/reference/generated/numpy.concatenate.html
- numpy.stack – https://numpy.org/doc/2.2/reference/generated/numpy.stack.html
- numpy.vstack – https://numpy.org/doc/stable/reference/generated/numpy.vstack.html
- numpy.hstack – https://numpy.org/doc/stable/reference/generated/numpy.hstack.html
- numpy.dstack – https://numpy.org/doc/stable/reference/generated/numpy.dstack.html
- numpy.column_stack – https://numpy.org/doc/2.4/reference/generated/numpy.column_stack.html
- numpy.block – https://numpy.org/doc/2.0/reference/generated/numpy.block.html
- NumPy Array manipulation routines – https://numpy.org/doc/stable/reference/routines.array-manipulation.html