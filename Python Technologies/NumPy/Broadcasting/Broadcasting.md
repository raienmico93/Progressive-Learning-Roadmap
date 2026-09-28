# NumPy Broadcasting Fundamentals: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Broadcasting is NumPy's mechanism for performing arithmetic operations between arrays of different shapes by virtually expanding the smaller array's dimensions to match the larger array's shape, without physically copying data.

**Technical Definition:** Broadcasting describes how NumPy treats arrays with different shapes during arithmetic operations. Subject to certain constraints, the smaller array is "broadcast" across the larger array so that they have compatible shapes. This is accomplished by manipulating the `strides` metadata: broadcasted axes are assigned a stride of 0, so that advancing along that axis repeatedly reads the same memory location. The result is a view-like expansion that requires no additional memory.

**Beginner-Friendly Explanation:** Broadcasting is NumPy's way of doing math between arrays of different sizes. If you have a table of data and a single row of values, NumPy can automatically "stretch" the row to match the table's size—not by copying it, but by pretending it's larger. This lets you write `data - row_mean` without manually repeating the row.

### Key Characteristics

- **No Data Copying:** Broadcasting is implemented through stride manipulation, not physical duplication of data.
- **Right-to-Left Alignment:** Shapes are compared from the trailing (rightmost) dimension to the leading dimension.
- **Singleton Expansion:** Dimensions of size 1 can be expanded to match larger dimensions.
- **Automatic Padding:** When arrays have different numbers of dimensions, the smaller shape is padded with 1s on the left.
- **Memory Efficient:** The broadcasted array occupies the same memory as the original.
- **Read-Only Views:** Explicit broadcasting via `np.broadcast_to()` returns a read-only view.

### Prerequisites

- NumPy installed (`pip install numpy`).
- Understanding of array shapes, dimensions, and axes.
- Familiarity with the `import numpy as np` convention.
- Conceptual understanding of strides and memory layout.

### Related Programming Areas

- **Data Normalization:** Subtracting column means from a matrix.
- **Machine Learning:** Adding bias vectors to batches of activations.
- **Image Processing:** Applying per-channel color adjustments to images.
- **Scientific Computing:** Combining spatial grids with time series.
- **Linear Algebra:** Outer products and rank-1 updates.

### Core Concepts / Features

The following core concepts are explored in detail: (1) Broadcasting concept, (2) Compatible dimensions, (3) Singleton dimensions, (4) Dimension alignment, and (5) Stride manipulation.

---

## Core Concept 1: Broadcasting Concept

### Definitions

**Core Definition:** Broadcasting is the set of rules that allows NumPy to perform element-wise operations on arrays of different shapes by virtually expanding the smaller array.

**Technical Definition:** The term broadcasting describes how NumPy treats arrays with different shapes during arithmetic operations. Subject to certain constraints, the smaller array is "broadcast" across the larger array so that they have compatible shapes. Broadcasting provides a means of vectorizing array operations so that looping occurs in C instead of Python, and it does this without making needless copies of data, leading to efficient algorithm implementations.

**Beginner-Friendly Explanation:** Imagine you have a table of numbers and a single number. Broadcasting lets you add the single number to every cell in the table without writing a loop. Extend that idea to a row vector and a matrix: NumPy automatically treats the row as if it were repeated for every row of the matrix.

### Purposes

- To enable arithmetic operations between arrays of different shapes without explicit loops.
- To avoid unnecessary memory allocation from manually tiling or repeating arrays.
- To provide a concise, readable syntax for common numerical patterns (e.g., normalization, bias addition).
- To move looping into compiled C code for performance.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Implicit broadcasting during operations
result = array_large + array_small

# Explicit broadcasting to a target shape
broadcasted = np.broadcast_to(array, shape)

# Broadcasting multiple arrays together
b1, b2 = np.broadcast_arrays(a1, a2)

# Compute the broadcast shape
shape = np.broadcast_shapes(shape1, shape2)
```

**Component Breakdown:**

| Function | Description |
|----------|-------------|
| `array_large + array_small` | Implicit broadcasting during arithmetic. |
| `np.broadcast_to(array, shape)` | Returns a read-only view of `array` broadcast to `shape`. |
| `np.broadcast_arrays(*arrays)` | Returns views of the inputs broadcast to a common shape. |
| `np.broadcast_shapes(*shapes)` | Returns the broadcast-compatible shape without creating arrays. |

**Syntax Rules:**

- Broadcasting applies automatically to all element-wise ufunc operations.
- The smaller array's dimensions must be compatible with the larger array's (see Core Concept 2).
- `np.broadcast_to()` returns a read-only view (writing raises `ValueError`).

**Constraints and Limitations:**

- Broadcasting does not consume extra memory but may produce surprising shapes if rules are misunderstood.
- Broadcasting a large array to a very large shape can still cause memory issues when the result is materialized (e.g., through assignment).
- `np.broadcast_to()` returns a read-only view; use `.copy()` to obtain a writable array.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Scalar, Vector, and Matrix Broadcasting**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Broadcasting with a scalar.
arr = np.array([1, 2, 3])
result_scalar = arr + 10
print("arr + 10:", result_scalar)
print("  (scalar broadcast to all elements)")
print()

# Step 3: Broadcasting a vector against a matrix.
matrix = np.array([[1, 2, 3],
                   [4, 5, 6],
                   [7, 8, 9]])
row = np.array([10, 20, 30])
result_row = matrix + row
print("matrix + row:\n", result_row)
print("  (row broadcast across each row of the matrix)")
print()

# Step 4: Broadcasting a column vector against a matrix.
col = np.array([[100], [200], [300]])
result_col = matrix + col
print("matrix + col:\n", result_col)
print("  (column broadcast across each column of the matrix)")
```

**Expected Output:**

```
arr + 10: [11 12 13]
  (scalar broadcast to all elements)

matrix + row:
 [[11 22 33]
 [14 25 36]
 [17 28 39]]
  (row broadcast across each row of the matrix)

matrix + col:
 [[101 102 103]
 [204 205 206]
 [307 308 309]]
  (column broadcast across each column of the matrix)
```

**Why This Result Occurs:** A scalar is broadcast to every element. A row vector of shape `(3,)` is treated as shape `(1, 3)` and broadcast across the three rows of the `(3, 3)` matrix. A column vector of shape `(3, 1)` is broadcast across the three columns.

### Real-World Cases

- **Data Normalization:** `X - X.mean(axis=0)` subtracts the per-feature mean (shape `(n_features,)`) from a data matrix (shape `(n_samples, n_features)`).
- **Neural Networks:** `activations + bias` adds a bias vector (shape `(n_neurons,)`) to a batch of activations (shape `(batch_size, n_neurons)`).
- **Image Processing:** `image * channel_scale` applies per-channel scaling to an HWC image.

### References

- Broadcasting – https://numpy.org/doc/stable/user/basics.broadcasting.html
- numpy.broadcast_arrays – https://numpy.org/doc/stable/reference/generated/numpy.broadcast_arrays.html
- numpy.broadcast_to – https://numpy.org/doc/stable/reference/generated/numpy.broadcast_to.html
- numpy.broadcast_shapes – https://numpy.org/doc/stable/reference/generated/numpy.broadcast_shapes.html

---

## Core Concept 2: Compatible Dimensions

### Definitions

**Core Definition:** Two dimensions are compatible for broadcasting when they are equal or one of them is 1.

**Technical Definition:** When operating on two arrays, NumPy compares their shapes element-wise. It starts with the trailing (i.e., rightmost) dimension and works its way left. Two dimensions are compatible when: (1) they are equal, or (2) one of them is 1. If these conditions are not met, a `ValueError` is raised, indicating that the arrays could not be broadcast together.

**Beginner-Friendly Explanation:** Imagine stacking two grids on top of each other. The grids can be combined if, for every direction, they either have the same size or one of them is a single row/column that can be stretched. NumPy checks from the back (rightmost) dimension forward.

### Purposes

- To determine whether two arrays can be combined in an element-wise operation.
- To predict the shape of the result before performing the operation.
- To diagnose `ValueError: operands could not be broadcast together` errors.
- To design array shapes that align correctly.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Check compatibility by attempting an operation
result = a + b   # Raises ValueError if incompatible

# Compute broadcast shape without performing operation
shape = np.broadcast_shapes(a.shape, b.shape)
```

**Component Breakdown:**

| Rule | Description |
|------|-------------|
| Align from the right | Compare trailing dimensions first. |
| Equal dimensions | Compatible. |
| One dimension is 1 | Compatible (the 1 is stretched). |
| Otherwise | Incompatible → `ValueError`. |

**Syntax Rules:**

- The shapes are aligned from the right; missing leading dimensions are treated as 1.
- The result shape has, for each aligned position, the maximum of the two sizes.
- Incompatibility raises a `ValueError` at runtime.

**Constraints and Limitations:**

- Broadcasting is only supported for element-wise operations; it does not change the semantics of matrix multiplication.
- Some operations (e.g., `np.dot`, `@`) have their own dimension rules and do not broadcast in the same way.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Compatible and Incompatible Shapes**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Compatible shapes (trailing dims equal).
a = np.ones((3, 4))
b = np.ones((3, 4))
print("(3,4) + (3,4) -> shape:", (a + b).shape)
print()

# Step 3: Compatible with trailing 1.
c = np.ones((3, 1))
print("(3,4) + (3,1) -> shape:", (a + c).shape)
print()

# Step 4: Compatible with fewer dimensions.
d = np.ones((4,))
print("(3,4) + (4,) -> shape:", (a + d).shape)
print("  (d treated as shape (1, 4))")
print()

# Step 5: Incompatible shapes (trailing dims 4 and 3).
e = np.ones((3, 3))
try:
    _ = a + e
except ValueError as err:
    print("(3,4) + (3,3) raises:", err)
print()

# Step 6: Compute broadcast shape without operation.
print("np.broadcast_shapes((3,4), (3,1)):", np.broadcast_shapes((3, 4), (3, 1)))
```

**Expected Output:**

```
(3,4) + (3,4) -> shape: (3, 4)

(3,4) + (3,1) -> shape: (3, 4)

(3,4) + (4,) -> shape: (3, 4)
  (d treated as shape (1, 4))

(3,4) + (3,3) raises: operands could not be broadcast together with shapes (3,4) (3,3) 

np.broadcast_shapes((3,4), (3,1)): (3, 4)
```

**Why This Result Occurs:** `(3, 4)` and `(3, 4)` are identical. `(3, 1)` aligns with `(3, 4)` because the trailing dimension 1 is compatible with 4. The shape `(4,)` is padded to `(1, 4)`, which is compatible. The shapes `(3, 4)` and `(3, 3)` are incompatible because 4 ≠ 3 and neither is 1.

### Real-World Cases

- **Feature Scaling:** Multiplying `(n_samples, n_features)` by `(n_features,)` scales each feature by its own factor.
- **Batch Bias Addition:** Adding `(n_neurons,)` to `(batch_size, n_neurons)` works because the bias broadcasts across the batch.
- **Time-Weighted Averages:** Multiplying `(time, features)` by `(time, 1)` applies per-time-step weights.

### References

- Broadcasting – https://numpy.org/doc/stable/user/basics.broadcasting.html
- numpy.broadcast_shapes – https://numpy.org/doc/stable/reference/generated/numpy.broadcast_shapes.html

---

## Core Concept 3: Singleton Dimensions

### Definitions

**Core Definition:** A singleton dimension is an axis of length 1 that can be expanded dynamically to match a larger dimension during broadcasting.

**Technical Definition:** Singleton dimensions are axes with a shape entry of 1. Under broadcasting rules, a dimension of size 1 is treated as if it had the size of the corresponding dimension in the other array. This is a "virtual" expansion: the data is not duplicated; the stride for that axis is set to 0, so the same element is reused along that axis.

**Beginner-Friendly Explanation:** A singleton dimension is like a single row or column that can be repeated as many times as needed without actually being copied. You create singleton dimensions explicitly with `np.newaxis` (or `None`) or `np.expand_dims()`.

### Purposes

- To create a dimension that will be stretched during broadcasting.
- To control alignment between arrays of different dimensionalities.
- To compute outer products and pairwise operations.
- To prepare arrays for operations that expect specific dimensional structures.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Create a singleton dimension with np.newaxis
column = arr[:, np.newaxis]    # Shape (n, 1)
row = arr[np.newaxis, :]       # Shape (1, n)

# Create with np.expand_dims
column = np.expand_dims(arr, axis=1)
```

**Component Breakdown:**

| Technique | Result Shape | Description |
|-----------|-------------|-------------|
| `arr[:, np.newaxis]` | `(n, 1)` | Column vector. |
| `arr[np.newaxis, :]` | `(1, n)` | Row vector. |
| `np.expand_dims(arr, axis)` | Insert at `axis`. | Programmatic insertion. |
| `np.squeeze(arr)` | Remove all size-1 axes. | Reverse operation. |

**Syntax Rules:**

- A dimension of size 1 can be expanded to any size during broadcasting.
- `np.newaxis` is an alias for `None`.
- Adding a singleton dimension does not copy data.

**Constraints and Limitations:**

- Once expanded via broadcasting, the result of the operation is a new array (the broadcast view itself is read-only).
- Too many singleton dimensions can make code harder to read.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Creating and Using Singleton Dimensions**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create 1-D arrays.
a = np.array([1, 2, 3])
b = np.array([10, 20, 30, 40])
print("a:", a, "| shape:", a.shape)
print("b:", b, "| shape:", b.shape)
print()

# Step 3: Attempt direct operation (fails).
try:
    _ = a * b
except ValueError as err:
    print("a * b raises:", err)
print()

# Step 4: Insert singleton dimensions to enable broadcasting.
a_col = a[:, np.newaxis]   # Shape (3, 1)
b_row = b[np.newaxis, :]   # Shape (1, 4)
print("a_col shape:", a_col.shape)
print("b_row shape:", b_row.shape)
print()

# Step 5: Perform outer product via broadcasting.
outer = a_col * b_row
print("a_col * b_row:\n", outer)
print("Shape:", outer.shape)
print()

# Step 6: Verify the singleton dimensions are not copies.
print("a_col.base is a:", a_col.base is a)
print("Shares memory with a:", np.shares_memory(a, a_col))
```

**Expected Output:**

```
a: [1 2 3] | shape: (3,)
b: [10 20 30 40] | shape: (4,)

a * b raises: operands could not be broadcast together with shapes (3,) (4,) 

a_col shape: (3, 1)
b_row shape: (1, 4)

a_col * b_row:
 [[ 10  20  30  40]
 [ 20  40  60  80]
 [ 30  60  90 120]]
Shape: (3, 4)

a_col.base is a: True
Shares memory with a: True
```

**Why This Result Occurs:** Adding singleton dimensions transforms `a` into a column vector `(3, 1)` and `b` into a row vector `(1, 4)`. These shapes are broadcast-compatible, producing a `(3, 4)` result. The `.base` attribute confirms that `a_col` is a view of `a`, not a copy.

### Real-World Cases

- **Outer Products:** `np.outer(a, b)` is equivalent to `a[:, np.newaxis] * b[np.newaxis, :]`.
- **Pairwise Distances:** `np.sqrt(((X[:, np.newaxis] - X[np.newaxis, :]) ** 2).sum(-1))`.
- **Batch Normalization:** `(X - X.mean(axis=0, keepdims=True)) / X.std(axis=0, keepdims=True)`.

### References

- Broadcasting – https://numpy.org/doc/stable/user/basics.broadcasting.html
- numpy.expand_dims – https://numpy.org/doc/stable/reference/generated/numpy.expand_dims.html

---

## Core Concept 4: Dimension Alignment

### Definitions

**Core Definition:** Dimension alignment is the practice of structuring array shapes so their trailing axes match or evaluate to 1, ensuring error-free broadcasting.

**Technical Definition:** When two arrays have different numbers of dimensions, NumPy pads the shape of the smaller array with 1s on the left (leading) side until both shapes have the same length. Then, each pair of dimensions is checked for compatibility. Dimension alignment is the deliberate design of array shapes—typically using `np.newaxis`, `reshape`, or `keepdims=True`—to satisfy these rules.

**Beginner-Friendly Explanation:** If you have a matrix of shape `(100, 5)` and a vector of shape `(5,)`, NumPy automatically treats the vector as `(1, 5)` so it can be subtracted from each row. But if you have a vector of shape `(100,)`, you need to explicitly reshape it to `(100, 1)` so it aligns with the rows.

### Purposes

- To ensure that operations between arrays of different shapes succeed.
- To control which axis a smaller array broadcasts along.
- To avoid cryptic broadcast errors.
- To write flexible code that adapts to different input shapes.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Automatic padding (no explicit action needed)
result = matrix + vector   # vector shape (n,) padded to (1, n)

# Explicit alignment with newaxis
aligned = vector[:, np.newaxis]   # shape (n, 1)

# Alignment with keepdims
means = X.mean(axis=0, keepdims=True)   # shape (1, n_features)
```

**Component Breakdown:**

| Technique | Effect |
|-----------|--------|
| Automatic padding | Leading 1s added to the smaller shape. |
| `np.newaxis` | Explicit insertion of a size-1 axis. |
| `keepdims=True` | Preserves reduced axes as size-1 dimensions. |
| `reshape` | Restructures shape entirely. |

**Syntax Rules:**

- Trailing (rightmost) dimensions must be equal or one must be 1.
- Leading dimensions can always be padded with 1s.
- Use `keepdims=True` in reductions to preserve alignment for subsequent broadcasts.

**Constraints and Limitations:**

- Alignment does not change the data; it only changes the shape metadata.
- Misaligned shapes that are not compatible raise `ValueError`.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Common Alignment Patterns**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create a data matrix.
X = np.arange(12).reshape(4, 3)
print("X (4, 3):\n", X)
print()

# Step 3: Subtract column means (automatic padding).
col_means = X.mean(axis=0)   # Shape (3,)
print("col_means:", col_means, "| shape:", col_means.shape)
centered_auto = X - col_means
print("X - col_means (automatic):\n", centered_auto)
print()

# Step 4: Subtract column means with keepdims.
col_means_kd = X.mean(axis=0, keepdims=True)   # Shape (1, 3)
centered_kd = X - col_means_kd
print("X - col_means_keepdims:\n", centered_kd)
print("  (same result, but shape is explicit)")
print()

# Step 5: Subtract row means (requires alignment).
row_means = X.mean(axis=1)   # Shape (4,)
print("row_means:", row_means, "| shape:", row_means.shape)
row_means_aligned = row_means[:, np.newaxis]   # Shape (4, 1)
centered_rows = X - row_means_aligned
print("X - row_means[:, np.newaxis]:\n", centered_rows)
```

**Expected Output:**

```
X (4, 3):
 [[ 0  1  2]
 [ 3  4  5]
 [ 6  7  8]
 [ 9 10 11]]

col_means: [4.5 5.5 6.5] | shape: (3,)
X - col_means (automatic):
 [[-4.5 -4.5 -4.5]
 [-1.5 -1.5 -1.5]
 [ 1.5  1.5  1.5]
 [ 4.5  4.5  4.5]]

X - col_means_keepdims:
 [[-4.5 -4.5 -4.5]
 [-1.5 -1.5 -1.5]
 [ 1.5  1.5  1.5]
 [ 4.5  4.5  4.5]]
  (same result, but shape is explicit)

row_means: [ 1.  4.  7. 10.] | shape: (4,)
X - row_means[:, np.newaxis]:
 [[-1.  0.  1.]
 [-1.  0.  1.]
 [-1.  0.  1.]
 [-1.  0.  1.]]
```

**Why This Result Occurs:** `col_means` has shape `(3,)`, which is automatically padded to `(1, 3)` and broadcast across the four rows. `row_means` has shape `(4,)`, which would be padded to `(1, 4)` and fail to align with `(4, 3)`. Adding `[:, np.newaxis]` gives shape `(4, 1)`, which broadcasts across the three columns.

### Real-World Cases

- **Feature Standardization:** `(X - X.mean(axis=0)) / X.std(axis=0)`.
- **Batch Processing:** `X[:, np.newaxis, :] - Y[np.newaxis, :, :]` for pairwise differences.
- **Attention Mechanisms:** `scores = queries @ keys.T / sqrt(d_k)` followed by softmax along the last axis.

### References

- Broadcasting – https://numpy.org/doc/stable/user/basics.broadcasting.html
- numpy.mean – https://numpy.org/doc/stable/reference/generated/numpy.mean.html

---

## Core Concept 5: Stride Manipulation

### Definitions

**Core Definition:** Stride manipulation is the internal C-level mechanism by which NumPy achieves broadcasting efficiency: broadcasted axes are assigned a stride of 0, causing the same memory location to be read repeatedly.

**Technical Definition:** Each ndarray has a `strides` tuple that specifies, for each axis, how many bytes to skip in memory to advance to the next element along that axis. When an axis is broadcast (expanded from size 1 to a larger size), NumPy sets the stride for that axis to 0. This means advancing along the broadcast axis does not move in memory—the same element is reused—so no data is copied. The broadcasted array is therefore a view with modified strides.

**Beginner-Friendly Explanation:** A stride is like a step size. Normally, moving to the next column means stepping forward by 8 bytes. But if a row is broadcast (repeated), NumPy sets the step size to 0 for that direction, so you keep reading the same row over and over. This is why broadcasting costs no extra memory—it's just a clever way of telling the CPU "read the same location again."

### Purposes

- To achieve broadcasting without duplicating data in memory.
- To reduce memory bandwidth and cache pressure in large computations.
- To enable zero-copy views of expanded arrays.
- To explain why broadcasting is so much more memory-efficient than `np.tile` or `np.repeat`.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Inspect strides
arr.strides   # Tuple of byte steps per dimension

# Explicit broadcasting (read-only view)
broadcasted = np.broadcast_to(arr, shape)
print(broadcasted.strides)   # Contains 0 for broadcast axes

# Use writable copy if modification is needed
writable = np.broadcast_to(arr, shape).copy()
```

**Component Breakdown:**

| Attribute | Description |
|-----------|-------------|
| `strides` | Tuple of byte offsets per dimension. |
| `0` stride | Indicates a broadcast axis (same element reused). |
| `np.broadcast_to` | Returns a read-only view with 0 strides. |
| `.copy()` | Materializes the broadcast into a writable array. |

**Syntax Rules:**

- A stride of 0 means advancing along that axis does not change the memory address.
- `np.broadcast_to()` returns a read-only view; writing raises `ValueError`.
- Explicit broadcasting returns a view, not a copy, so memory usage is unchanged.

**Constraints and Limitations:**

- Broadcasted views are read-only since NumPy 2.0 (writing was deprecated earlier).
- Some operations may copy the broadcasted data if a writable result is required.
- Strides can be negative (for reversed views) in addition to zero.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Inspecting Broadcast Strides**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create a 1-D array.
a = np.array([1, 2, 3])
print("a:", a)
print("a.shape:", a.shape)
print("a.strides:", a.strides)
print("  (8 bytes per element, the dtype itemsize)")
print()

# Step 3: Broadcast to a 2-D shape.
b = np.broadcast_to(a, (4, 3))
print("b = np.broadcast_to(a, (4, 3)):")
print(b)
print("b.shape:", b.shape)
print("b.strides:", b.strides)
print("  (stride 0 on axis 0 means the same row is reused)")
print()

# Step 4: Verify memory efficiency.
print("a.nbytes:", a.nbytes, "bytes")
print("b.nbytes:", b.nbytes, "bytes")
print("  (nbytes reflects the broadcast shape, but the data is shared)")
print("Shares memory with a:", np.shares_memory(a, b))
print()

# Step 5: Confirm read-only nature.
print("b.flags['WRITEABLE']:", b.flags['WRITEABLE'])
try:
    b[0, 0] = 99
except ValueError as err:
    print("Writing to broadcasted view raises:", err)
```

**Expected Output:**

```
a: [1 2 3]
a.shape: (3,)
a.strides: (8,)
  (8 bytes per element, the dtype itemsize)

b = np.broadcast_to(a, (4, 3)):
 [[1 2 3]
 [1 2 3]
 [1 2 3]
 [1 2 3]]
b.shape: (4, 3)
b.strides: (0, 8)
  (stride 0 on axis 0 means the same row is reused)

a.nbytes: 24 bytes
b.nbytes: 96 bytes
  (nbytes reflects the broadcast shape, but the data is shared)
Shares memory with a: True

b.flags['WRITEABLE']: False
Writing to broadcasted view raises: assignment destination is read-only
```

**Why This Result Occurs:** The original array `a` has strides `(8,)`, meaning moving one step forward skips 8 bytes (one `int64` element). When broadcast to `(4, 3)`, the new view has strides `(0, 8)`: moving along axis 0 (to the next row) skips 0 bytes, so the same row is reused; moving along axis 1 (to the next column) skips 8 bytes. The broadcasted view is read-only because writing to it would silently overwrite the original data through the shared buffer.

**Example 2: Broadcasting vs. Tiling Memory Usage**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create a row vector.
row = np.array([1.0, 2.0, 3.0])

# Step 3: Broadcast (view, no data copy).
broadcasted = np.broadcast_to(row, (1_000_000, 3))
print("Broadcasted view:")
print("  shape:", broadcasted.shape)
print("  base is row:", broadcasted.base is row)
print("  shares memory:", np.shares_memory(row, broadcasted))
print()

# Step 4: Tile (copy, duplicates data).
tiled = np.tile(row, (1_000_000, 1))
print("Tiled copy:")
print("  shape:", tiled.shape)
print("  base is row:", tiled.base is row)
print("  shares memory:", np.shares_memory(row, tiled))
print()

# Step 5: Compare memory consumption.
print(f"Broadcasted nbytes: {broadcasted.nbytes / 1e6:.1f} MB (virtual)")
print(f"Tiled nbytes:       {tiled.nbytes / 1e6:.1f} MB (physical)")
```

**Expected Output:**

```
Broadcasted view:
  shape: (1000000, 3)
  base is row: True
  shares memory: True

Tiled copy:
  shape: (1000000, 3)
  base is row: False
  shares memory: False

Broadcasted nbytes: 24.0 MB (virtual)
Tiled nbytes:       24.0 MB (physical)
```

**Why This Result Occurs:** `np.broadcast_to` returns a view with stride 0, so the underlying data remains just 24 bytes (three floats). `np.tile` physically duplicates the row, allocating 24 MB. Both report the same `nbytes` because `nbytes` is computed from the shape and itemsize, but the broadcasted array's memory is shared with the original 24-byte row. The key lesson: `np.broadcast_to` is a view; `np.tile` is a copy.

### Real-World Cases

- **Memory-Efficient Normalization:** Subtracting a row of means from a large matrix uses broadcasting with 0 strides, avoiding a tiled copy.
- **Neural Network Bias Addition:** Adding a bias vector to a batch of activations uses broadcast strides, not repeated memory.
- **Outer Products:** `a[:, np.newaxis] * b[np.newaxis, :]` uses broadcast strides on both sides.

### References

- Broadcasting – https://numpy.org/doc/stable/user/basics.broadcasting.html
- numpy.broadcast_to – https://numpy.org/doc/stable/reference/generated/numpy.broadcast_to.html
- NumPy internals – https://numpy.org/doc/stable/dev/internals.html
- numpy.ndarray.strides – https://numpy.org/doc/stable/reference/generated/numpy.ndarray.strides.html

---

## Summary of Broadcasting Rules

| Rule | Description |
|------|-------------|
| Align from the right | Compare trailing dimensions first. |
| Pad with 1s | Missing leading dimensions are treated as size 1. |
| Equal or 1 | Two dimensions are compatible if equal or one is 1. |
| Result shape | Element-wise maximum of aligned dimensions. |
| Zero strides | Broadcast axes have stride 0; no data is copied. |
| Read-only views | Explicit broadcasts are read-only; use `.copy()` to write. |

---

## Summary of Deprecated and Version-Specific Features

| Feature | Status | Recommendation |
|---------|--------|----------------|
| Writing to `np.broadcast_to` result | Read-only since NumPy 2.0; writing previously emitted `FutureWarning` | Use `.copy()` to get a writable array |
| `np.broadcast` object | Still supported | Use for explicit broadcasting iteration |
| Legacy broadcasting of object arrays | Behavior may differ | Test with current NumPy version |

---

## Consolidated Reference List

- Broadcasting – https://numpy.org/doc/stable/user/basics.broadcasting.html
- numpy.broadcast_arrays – https://numpy.org/doc/stable/reference/generated/numpy.broadcast_arrays.html
- numpy.broadcast_to – https://numpy.org/doc/stable/reference/generated/numpy.broadcast_to.html
- numpy.broadcast_shapes – https://numpy.org/doc/stable/reference/generated/numpy.broadcast_shapes.html
- numpy.broadcast – https://numpy.org/doc/stable/reference/generated/numpy.broadcast.html
- numpy.ndarray.strides – https://numpy.org/doc/stable/reference/generated/numpy.ndarray.strides.html
- NumPy internals – https://numpy.org/doc/stable/dev/internals.html
- numpy.expand_dims – https://numpy.org/doc/stable/reference/generated/numpy.expand_dims.html
- numpy.mean – https://numpy.org/doc/stable/reference/generated/numpy.mean.html
- Copies and views – https://numpy.org/doc/stable/user/basics.copies.html