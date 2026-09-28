# NumPy Transposing: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Transposing in NumPy is the operation of reversing or permuting the axes of an array, producing a new array whose dimensions are rearranged relative to the original.

**Technical Definition:** Transposing is a metadata operation that modifies the `strides` of an array without altering the data buffer. The `.T` attribute and `transpose()` method return views with reversed or permuted axes, while `swapaxes()` interchanges exactly two specified axes. Because no data is moved, transposition is O(1) in time and memory.

**Beginner-Friendly Explanation:** Transposing is like rotating a table of data. If you have a table with 3 rows and 4 columns, transposing it gives you a table with 4 rows and 3 columns—the same numbers, just arranged differently. NumPy does this without copying any data; it simply changes how it "looks at" the existing memory.

### Key Characteristics

- **View Semantics:** Transposition always returns a view, never a copy. Modifying the transposed array modifies the original.
- **Stride Manipulation:** Transposing reverses or permutes the `strides` tuple, changing how memory is traversed without moving data.
- **O(1) Operation:** Transposing is instantaneous regardless of array size because no data is copied.
- **1-D Arrays:** For 1-D arrays, transposing has no effect—the array is unchanged.
- **Axis Permutation:** `transpose()` accepts an explicit tuple of axis indices to define a custom permutation.

### Prerequisites

- NumPy installed (`pip install numpy`).
- Understanding of array shapes, axes, and strides.
- Familiarity with the `import numpy as np` convention.
- Conceptual understanding of memory layout (row-major vs. column-major).

### Related Programming Areas

- **Machine Learning:** Converting image tensors between HWC (Height, Width, Channels) and CHW (Channels, Height, Width) layouts for different frameworks.
- **Linear Algebra:** Computing matrix transposes for operations like `A.T @ B`.
- **Signal Processing:** Rearranging time-frequency representations.
- **Data Analysis:** Pivoting data tables to change the orientation of rows and columns.

### Core Concepts / Features

The following core concepts are explored in detail: (1) `.T` attribute, (2) `transpose()` method, (3) Axis permutation, (4) Transposing higher-dimensional arrays, and (5) `swapaxes()`.

---

## Core Concept 1: .T Attribute

### Definitions

**Core Definition:** The `.T` attribute returns a view of the array with its axes reversed.

**Technical Definition:** `ndarray.T` is a property that returns a view of the transposed array, equivalent to calling `self.transpose()`. For a 2-D array, this is the standard matrix transpose (rows become columns and vice versa). For an n-D array, it reverses the order of the axes, so `a.T.shape == a.shape[::-1]`. For a 1-D array, `.T` returns an unchanged view of the original array.

**Beginner-Friendly Explanation:** `.T` is the quickest way to transpose an array. Just add `.T` after the array name, and you get the transposed version. For a table, it swaps rows and columns. It's as simple as that—no function call needed.

### Purposes

- To quickly transpose a 2-D matrix without calling a function.
- To reverse the axes of a multidimensional array in one operation.
- To provide a concise, readable syntax for the most common transpose operation.
- To enable matrix multiplication conventions where the second operand must be transposed.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
transposed = array.T
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `array` | Any NumPy `ndarray`. |
| `.T` | Property returning a view with reversed axes. |
| Equivalent | `array.transpose()` |

**Syntax Rules:**

- `.T` takes no arguments.
- For a 1-D array, `.T` returns the same array (no change).
- For a 2-D array, `.T` swaps rows and columns.
- For an n-D array, `.T` reverses the axis order.

**Constraints and Limitations:**

- `.T` cannot specify a custom axis order; use `transpose()` for that.
- The returned array is a view; modifications affect the original.
- For very high-dimensional arrays, `.T` may not produce the desired permutation; use `transpose()` with explicit axes.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Basic .T on 1-D and 2-D Arrays**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create a 1-D array.
a = np.array([1, 2, 3, 4])
print("1-D array:", a)
print("a.T:", a.T)
print("  (1-D transpose has no effect)")
print()

# Step 3: Create a 2-D array.
b = np.array([[1, 2, 3],
              [4, 5, 6]])
print("2-D array (2x3):\n", b)
print("b.T (3x2):\n", b.T)
print("Shape of b.T:", b.T.shape)
print()

# Step 4: Verify that .T returns a view.
print("b.T.base is b:", b.T.base is b)
print("Shares memory:", np.shares_memory(b, b.T))
```

**Expected Output:**

```
1-D array: [1 2 3 4]
a.T: [1 2 3 4]
  (1-D transpose has no effect)

2-D array (2x3):
 [[1 2 3]
 [4 5 6]]
b.T (3x2):
 [[1 4]
 [2 5]
 [3 6]]
Shape of b.T: (3, 2)

b.T.base is b: True
Shares memory: True
```

**Why This Result Occurs:** For the 1-D array, transposing has no effect because a vector's transpose is the same vector. For the 2-D array, `.T` swaps the rows and columns: the first row `[1, 2, 3]` becomes the first column `[1, 4]`, and so on. The `.base` attribute confirms that `.T` returns a view, not a copy.

### Real-World Cases

- **Linear Algebra:** Computing `A.T @ B` for matrix multiplication.
- **Data Tables:** Quickly swapping rows and columns of a DataFrame-like array.
- **Image Processing:** Transposing a 2-D grayscale image to rotate it 90 degrees.

### References

- NumPy ndarray.T – https://numpy.org/doc/2.2/reference/generated/numpy.ndarray.T.html
- NumPy transpose – https://numpy.org/doc/devdocs/reference/generated/numpy.transpose.html

---

## Core Concept 2: transpose() Method

### Definitions

**Core Definition:** The `transpose()` method returns a view of the array with its axes permuted according to a specified order or, by default, reversed.

**Technical Definition:** `ndarray.transpose(*axes)` or `numpy.transpose(a, axes=None)` returns a view of the array with axes transposed. For an n-D array, if `axes` is given, it must be a permutation of `[0, 1, ..., N-1]`; the i-th axis of the returned array corresponds to axis `axes[i]` of the input. If `axes` is not provided, it defaults to reversing the order of the axes (`range(a.ndim)[::-1]`). For a 1-D array, transpose has no effect.

**Beginner-Friendly Explanation:** `transpose()` does the same thing as `.T`, but it lets you specify exactly how you want the axes rearranged. If you have a 3-D array and you want to move the last axis to the front, you can tell `transpose()` the order you want.

### Purposes

- To reverse the axes of an array when `.T` is not sufficient.
- To specify a custom axis permutation for multidimensional arrays.
- To provide a function-based interface for transposing arrays.
- To enable axis reordering in data pipelines where the desired order is known.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# As a method
transposed = array.transpose(*axes)

# As a function
transposed = np.transpose(array, axes=None)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `array` | Input array. |
| `axes` | Optional tuple or list of integers specifying the permutation. |
| Default | Reverses the axes: `range(a.ndim)[::-1]`. |
| Negative indices | Allowed; count from the end. |

**Syntax Rules:**

- `axes` must contain a permutation of `[0, 1, ..., N-1]`.
- The i-th axis of the result corresponds to axis `axes[i]` of the input.
- For a 1-D array, transpose returns an unchanged view.

**Constraints and Limitations:**

- If `axes` is provided, it must be a valid permutation; otherwise, a `ValueError` is raised.
- The returned array is always a view; no copy is made.
- Transposing does not affect the memory layout; only the strides change.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: transpose() with Default and Custom Axes**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create a 3-D array.
a = np.ones((1, 2, 3))
print("Original shape:", a.shape)
print()

# Step 3: Default transpose (reverses axes).
default_t = np.transpose(a)
print("Default transpose shape:", default_t.shape)
print("  (reverses to (3, 2, 1))")
print()

# Step 4: Custom transpose with axes (1, 0, 2).
custom_t = np.transpose(a, (1, 0, 2))
print("Custom transpose (1, 0, 2) shape:", custom_t.shape)
print("  (axis 0 and 1 are swapped, axis 2 stays)")
print()

# Step 5: Verify view semantics.
b = np.arange(6).reshape(2, 3)
print("Original b:\n", b)
print("b.transpose():\n", b.transpose())
print("b.T.base is b:", b.T.base is b)
```

**Expected Output:**

```
Original shape: (1, 2, 3)

Default transpose shape: (3, 2, 1)
  (reverses to (3, 2, 1))

Custom transpose (1, 0, 2) shape: (2, 1, 3)
  (axis 0 and 1 are swapped, axis 2 stays)

Original b:
 [[0 1 2]
 [3 4 5]]
b.transpose():
 [[0 3]
 [1 4]
 [2 5]]
b.T.base is b: True
```

**Why This Result Occurs:** The default transpose reverses the axes, so a shape of `(1, 2, 3)` becomes `(3, 2, 1)`. The custom transpose `(1, 0, 2)` swaps axes 0 and 1 (resulting in `(2, 1, 3)`) while keeping axis 2 in place. The `.base` attribute confirms that the transposed array is a view of the original.

### Real-World Cases

- **Framework Interoperability:** Converting between tensor layouts for different deep learning frameworks.
- **Data Pipelines:** Reordering axes of a multidimensional dataset for analysis.
- **Scientific Computing:** Permuting axes of simulation data for visualization.

### References

- NumPy transpose – https://numpy.org/doc/devdocs/reference/generated/numpy.transpose.html
- NumPy ndarray.transpose – https://numpy.org/doc/stable/reference/generated/numpy.ndarray.transpose.html

---

## Core Concept 3: Axis Permutation

### Definitions

**Core Definition:** Axis permutation is the specification of a custom order for an array's axes, provided as a tuple of integers to `transpose()`.

**Technical Definition:** When `axes` is passed to `transpose()`, it must be a permutation of `[0, 1, ..., N-1]`. The returned array's i-th axis corresponds to the input array's axis `axes[i]`. Negative indices can be used to count axes from the end.

**Beginner-Friendly Explanation:** Axis permutation lets you say exactly how you want the axes rearranged. If you have a 3-D array with axes labeled 0, 1, and 2, you can tell NumPy to make the new array's first axis come from the old axis 2, the second from old axis 0, and the third from old axis 1—just by passing `(2, 0, 1)`.

### Purposes

- To rearrange axes in an arbitrary order for custom data layouts.
- To convert between different tensor format conventions (e.g., HWC to CHW).
- To enable axis-specific operations in data processing pipelines.
- To support broadcasting and alignment of arrays with different axis orders.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
permuted = array.transpose(axes_tuple)
permuted = np.transpose(array, axes_tuple)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `axes_tuple` | A permutation of `[0, 1, ..., N-1]`. |
| Negative indices | Allowed; e.g., `-1` refers to the last axis. |
| Result | `result.shape[i] == array.shape[axes[i]]`. |

**Syntax Rules:**

- `axes_tuple` must contain exactly `N` integers for an N-D array.
- Each integer in `axes_tuple` must be unique and in the range `[-N, N-1]`.
- The order of integers in `axes_tuple` defines the new axis order.

**Constraints and Limitations:**

- Duplicate or out-of-range axis indices raise `ValueError`.
- The number of axes in `axes_tuple` must match the array's `ndim`.
- Axis permutation always returns a view.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Custom Axis Permutations**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create a 3-D array with shape (2, 3, 4).
a = np.arange(24).reshape(2, 3, 4)
print("Original shape:", a.shape)
print()

# Step 3: Permute axes to (2, 0, 1).
perm_1 = a.transpose(2, 0, 1)
print("transpose(2, 0, 1) shape:", perm_1.shape)
print("  (axis 2 -> 0, axis 0 -> 1, axis 1 -> 2)")
print()

# Step 4: Permute axes to (1, 2, 0).
perm_2 = a.transpose(1, 2, 0)
print("transpose(1, 2, 0) shape:", perm_2.shape)
print("  (axis 1 -> 0, axis 2 -> 1, axis 0 -> 2)")
print()

# Step 5: Use negative indices.
perm_3 = a.transpose(-1, -3, -2)
print("transpose(-1, -3, -2) shape:", perm_3.shape)
print("  (same as (2, 0, 1))")
```

**Expected Output:**

```
Original shape: (2, 3, 4)

transpose(2, 0, 1) shape: (4, 2, 3)
  (axis 2 -> 0, axis 0 -> 1, axis 1 -> 2)

transpose(1, 2, 0) shape: (3, 4, 2)
  (axis 1 -> 0, axis 2 -> 1, axis 0 -> 2)

transpose(-1, -3, -2) shape: (4, 2, 3)
  (same as (2, 0, 1))
```

**Why This Result Occurs:** The permutation `(2, 0, 1)` means the new first axis is the old third axis (size 4), the new second axis is the old first axis (size 2), and the new third axis is the old second axis (size 3), producing shape `(4, 2, 3)`. The permutation `(1, 2, 0)` produces `(3, 4, 2)`. Negative indices `(-1, -3, -2)` map to `(2, 0, 1)` in positive terms, producing the same shape.

### Real-World Cases

- **Deep Learning:** Converting NHWC (batch, height, width, channels) to NCHW (batch, channels, height, width) for GPU frameworks.
- **Image Processing:** Rearranging (height, width, channels) to (channels, height, width).
- **Climate Data:** Reordering (time, latitude, longitude) to (latitude, longitude, time).

### References

- NumPy transpose – https://numpy.org/doc/devdocs/reference/generated/numpy.transpose.html
- NumPy ndarray.transpose – https://numpy.org/doc/stable/reference/generated/numpy.ndarray.transpose.html

---

## Core Concept 4: Transposing Higher-Dimensional Arrays

### Definitions

**Core Definition:** Transposing higher-dimensional arrays involves rearranging the axes of arrays with three or more dimensions, such as converting between HWC and CHW tensor layouts.

**Technical Definition:** For a 3-D array with shape `(H, W, C)`, transposing to `(C, H, W)` is achieved by `transpose(2, 0, 1)`. The data buffer is unchanged; only the strides are reordered. This is a view operation, but the resulting array may be non-contiguous, which can affect performance and interoperability with libraries that expect contiguous memory.

**Beginner-Friendly Explanation:** In image processing and deep learning, tensors often have different axis orders. For example, some libraries expect images in "height, width, channels" order, while others expect "channels, height, width." Transposing lets you switch between these layouts without copying the data.

### Purposes

- To convert between tensor layout conventions for different libraries.
- To rearrange spatial dimensions of image or video data.
- To enable batch operations where the batch dimension must be moved.
- To prepare data for operations that expect specific axis orders.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# HWC -> CHW
chw = hwc.transpose(2, 0, 1)

# CHW -> HWC
hwc = chw.transpose(1, 2, 0)
```

**Component Breakdown:**

| Layout | Shape | Transpose to CHW |
|--------|-------|------------------|
| HWC | (H, W, C) | `(2, 0, 1)` |
| CHW | (C, H, W) | — |
| NHWC | (N, H, W, C) | `(0, 3, 1, 2)` for NCHW |

**Syntax Rules:**

- The permutation tuple must have the same number of elements as the array's `ndim`.
- For HWC to CHW, the permutation is `(2, 0, 1)`.
- For CHW to HWC, the permutation is `(1, 2, 0)`.

**Constraints and Limitations:**

- The transposed array may be non-contiguous; use `np.ascontiguousarray()` if a library requires contiguous memory.
- Transposing does not change the data values, only the axis order.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: HWC to CHW Conversion**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create a 3-D array representing an image in HWC layout.
# Shape: (height=2, width=3, channels=3)
hwc = np.arange(18).reshape(2, 3, 3)
print("HWC array shape:", hwc.shape)
print("HWC array:\n", hwc)
print()

# Step 3: Transpose to CHW layout.
chw = hwc.transpose(2, 0, 1)
print("CHW array shape:", chw.shape)
print("CHW array:\n", chw)
print()

# Step 4: Verify view semantics.
print("chw.base is hwc:", chw.base is hwc)
print("Shares memory:", np.shares_memory(hwc, chw))
print()

# Step 5: Convert back to HWC.
back_to_hwc = chw.transpose(1, 2, 0)
print("Back to HWC shape:", back_to_hwc.shape)
print("Same as original:", np.array_equal(hwc, back_to_hwc))
```

**Expected Output:**

```
HWC array shape: (2, 3, 3)
HWC array:
 [[[ 0  1  2]
  [ 3  4  5]
  [ 6  7  8]]

 [[ 9 10 11]
  [12 13 14]
  [15 16 17]]]

CHW array shape: (3, 2, 3)
CHW array:
 [[[ 0  1  2]
  [ 9 10 11]]

 [[ 3  4  5]
  [12 13 14]]

 [[ 6  7  8]
  [15 16 17]]]

chw.base is hwc: True
Shares memory: True

Back to HWC shape: (2, 3, 3)
Same as original: True
```

**Why This Result Occurs:** The `transpose(2, 0, 1)` operation rearranges the axes so that the channel axis (originally axis 2) becomes the first axis, the height axis (originally axis 0) becomes the second, and the width axis (originally axis 1) becomes the third. The data values are unchanged; only the axis order is different. The `.base` attribute confirms that the transposed array is a view of the original.

### Real-World Cases

- **Deep Learning:** Converting image batches from NHWC (TensorFlow) to NCHW (PyTorch).
- **Image Processing:** Preparing images for OpenCV, which expects HWC layout.
- **Video Processing:** Rearranging (frames, height, width, channels) to (channels, frames, height, width).

### References

- Stack Overflow: OpenCV expects HWC but you have CHW – https://stackoverflow.com/revisions/73612159/2
- NumPy transpose – https://numpy.org/doc/devdocs/reference/generated/numpy.transpose.html

---

## Core Concept 5: swapaxes() Method

### Definitions

**Core Definition:** `swapaxes()` interchanges exactly two specified axes of an array while keeping all other axes in their original positions.

**Technical Definition:** `numpy.swapaxes(a, axis1, axis2)` returns a view of the array with `axis1` and `axis2` interchanged. For NumPy >= 1.10.0, if `a` is an `ndarray`, a view is returned; otherwise, a new array is created. The shape of the result is the same as the input except that the sizes of `axis1` and `axis2` are swapped.

**Beginner-Friendly Explanation:** `swapaxes()` is like `transpose()`, but instead of rearranging all axes at once, it only swaps two axes and leaves everything else exactly where it was. It's simpler when you just need to exchange two dimensions.

### Purposes

- To interchange exactly two axes without affecting the others.
- To simplify axis swapping when only two dimensions need to change.
- To provide a more readable alternative to `transpose()` for two-axis swaps.
- To enable specific data rearrangements in multidimensional pipelines.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
swapped = np.swapaxes(array, axis1, axis2)
swapped = array.swapaxes(axis1, axis2)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `array` | Input array. |
| `axis1` | First axis to swap. |
| `axis2` | Second axis to swap. |
| Negative indices | Allowed; count from the end. |

**Syntax Rules:**

- `axis1` and `axis2` must be valid axis indices (0 to `ndim-1`).
- The result is a view (for NumPy >= 1.10.0).
- All other axes remain in their original order.

**Constraints and Limitations:**

- Only two axes can be swapped at a time; use `transpose()` for multi-axis permutations.
- The returned view may be non-contiguous.
- If `axis1 == axis2`, the array is returned unchanged.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Swapping Axes in 2-D and 3-D Arrays**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create a 2-D array.
x = np.array([[1, 2, 3]])
print("Original 2-D array (1x3):\n", x)
print("Shape:", x.shape)
print()

# Step 3: Swap axes 0 and 1.
swapped_2d = np.swapaxes(x, 0, 1)
print("swapaxes(x, 0, 1) (3x1):\n", swapped_2d)
print("Shape:", swapped_2d.shape)
print()

# Step 4: Create a 3-D array.
y = np.array([[[0, 1], [2, 3]],
              [[4, 5], [6, 7]]])
print("Original 3-D array shape:", y.shape)
print("3-D array:\n", y)
print()

# Step 5: Swap axes 0 and 2.
swapped_3d = np.swapaxes(y, 0, 2)
print("swapaxes(y, 0, 2) shape:", swapped_3d.shape)
print("Swapped array:\n", swapped_3d)
```

**Expected Output:**

```
Original 2-D array (1x3):
 [[1 2 3]]
Shape: (1, 3)

swapaxes(x, 0, 1) (3x1):
 [[1]
 [2]
 [3]]
Shape: (3, 1)

Original 3-D array shape: (2, 2, 2)
3-D array:
 [[[0 1]
  [2 3]]

 [[4 5]
  [6 7]]]

swapaxes(y, 0, 2) shape: (2, 2, 2)
Swapped array:
 [[[0 4]
  [2 6]]

 [[1 5]
  [3 7]]]
```

**Why This Result Occurs:** For the 2-D array, swapping axes 0 and 1 turns a 1×3 row vector into a 3×1 column vector. For the 3-D array, swapping axes 0 and 2 interchanges the outermost and innermost dimensions, effectively transposing the first and third axes while leaving the middle axis unchanged.

### Real-World Cases

- **Batch Processing:** Swapping batch and channel axes in a 4-D tensor.
- **Image Processing:** Swapping height and width axes to rotate an image.
- **Scientific Computing:** Interchanging spatial and temporal axes in simulation data.

### References

- NumPy swapaxes – https://numpy.org/doc/2.0/reference/generated/numpy.swapaxes.html
- NumPy ndarray.swapaxes – https://numpy.org/doc/stable/reference/generated/numpy.ndarray.swapaxes.html

---

## Summary of Deprecated and Version-Specific Features

| Feature | Status | Recommendation |
|---------|--------|----------------|
| `np.matrix.T` | Should no longer be used | Use regular 2-D `ndarray` and `.T` |
| `np.rollaxis` | Deprecated in favor of `moveaxis` | Use `np.moveaxis()` or `transpose()` |
| `swapaxes` returning view for all inputs | View only for `ndarray` since NumPy 1.10.0 | Ensure input is `ndarray` for view semantics |

---

## Consolidated Reference List

- NumPy ndarray.T – https://numpy.org/doc/2.2/reference/generated/numpy.ndarray.T.html
- NumPy transpose – https://numpy.org/doc/devdocs/reference/generated/numpy.transpose.html
- NumPy ndarray.transpose – https://numpy.org/doc/stable/reference/generated/numpy.ndarray.transpose.html
- NumPy swapaxes – https://numpy.org/doc/2.0/reference/generated/numpy.swapaxes.html
- NumPy ndarray.swapaxes – https://numpy.org/doc/stable/reference/generated/numpy.ndarray.swapaxes.html
- NumPy Copies and Views – https://numpy.org/doc/1.22/user/basics.copies.html
- NumPy Internal Organization – https://numpy.org/doc/stable/dev/internals.html
- Stack Overflow: OpenCV HWC vs CHW – https://stackoverflow.com/revisions/73612159/2
- NumPy Discussion: Will transpose ever need to copy data? – https://mail.python.org/pipermail/numpy-discussion/2013-June/