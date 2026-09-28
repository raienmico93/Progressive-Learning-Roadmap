# NumPy Splitting Arrays: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Splitting arrays in NumPy is the set of operations that divide a single array into multiple sub-arrays along a specified axis, either into equal-sized partitions or at designated index positions.

**Technical Definition:** NumPy provides a family of array-splitting routines: `split` divides an array into equal-sized sub-arrays and raises an error if equal division is not possible; `array_split` performs the same operation but allows for unequal divisions; and the convenience wrappers `hsplit`, `vsplit`, and `dsplit` split along specific axes (columns, rows, and depth, respectively). All splitting operations return lists of views into the original array.

**Beginner-Friendly Explanation:** Splitting is the opposite of combining. Instead of joining pieces together, you're cutting a large array into smaller pieces. You can cut it into equal parts, or you can cut it at specific positions. NumPy offers different functions depending on which direction you want to cut—horizontally, vertically, or depth-wise.

### Key Characteristics

- **View Semantics:** Splitting returns views of the original array, not copies. Modifying a split sub-array modifies the original.
- **Axis-Based:** Every splitting operation works along a specified axis.
- **Equal vs. Near-Equal:** `split` requires equal division; `array_split` allows for near-equal divisions when the axis length is not perfectly divisible.
- **Index-Based:** Splitting can be specified either by number of sections or by explicit index positions.
- **Convenience Wrappers:** `hsplit`, `vsplit`, and `dsplit` are shortcuts for `split` along axes 1, 0, and 2, respectively.

### Prerequisites

- NumPy installed (`pip install numpy`).
- Understanding of array shapes, axes, and dimensions.
- Familiarity with the `import numpy as np` convention.

### Related Programming Areas

- **Data Partitioning:** Splitting datasets into training, validation, and test sets.
- **Image Processing:** Dividing images into tiles or blocks for parallel processing.
- **Signal Processing:** Segmenting time series into windows for spectral analysis.
- **Parallel Computing:** Distributing array chunks across multiple processors.

### Core Concepts / Features

The following core concepts are explored in detail: (1) `split()`, (2) `array_split()`, (3) `hsplit()`, (4) `vsplit()`, and (5) `dsplit()`.

---

## Core Concept 1: split()

### Definitions

**Core Definition:** `split()` divides an array into multiple equal-sized sub-arrays along a specified axis, raising an error if the array cannot be divided equally.

**Technical Definition:** `numpy.split(ary, indices_or_sections, axis=0)` splits an array into multiple sub-arrays as views into `ary`. If `indices_or_sections` is an integer `N`, the array is divided into `N` equal arrays along `axis`. If such a split is not possible, an error is raised. If `indices_or_sections` is a 1-D array of sorted integers, the entries indicate where along `axis` the array is split. For example, `[2, 3]` would, for `axis=0`, result in `ary[:2]`, `ary[2:3]`, and `ary[3:]`. If an index exceeds the dimension of the array along `axis`, an empty sub-array is returned correspondingly.

**Beginner-Friendly Explanation:** `split()` is like cutting a cake into equal slices. You tell it how many pieces you want, and it cuts evenly. If the cake can't be cut evenly, it complains—that's when you'd use `array_split()` instead.

### Purposes

- To divide an array into equal-sized partitions along a specified axis.
- To split arrays at exact index positions using sorted integer arrays.
- To reassemble arrays that were previously combined with `concatenate`.
- To provide the foundational operation that other splitting functions are built upon.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
numpy.split(ary, indices_or_sections, axis=0)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `ary` | Input array to be divided into sub-arrays. |
| `indices_or_sections` | Int or 1-D array. If an integer `N`, divide into `N` equal arrays. If a 1-D array of sorted integers, the entries indicate split points. |
| `axis` | Int, optional. The axis along which to split. Default is 0. |

**Syntax Rules:**

- If `indices_or_sections` is an integer, the array must be evenly divisible by that integer along the specified axis.
- If `indices_or_sections` is a 1-D array, the entries are split points: `[2, 3]` for `axis=0` produces `ary[:2]`, `ary[2:3]`, `ary[3:]`.
- If an index exceeds the dimension of the array along `axis`, an empty sub-array is returned.
- The result is a list of sub-arrays as views into `ary`.

**Constraints and Limitations:**

- If `indices_or_sections` is an integer and the split does not result in equal division, a `ValueError` is raised.
- The number of sections must not exceed the size of the array along the specified axis.
- Splitting returns views, not copies; modifications to sub-arrays affect the original.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Splitting into Equal Parts and at Index Positions**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create a 1-D array of 9 elements.
x = np.arange(9.0)
print("Original array:", x)
print()

# Step 3: Split into 3 equal parts.
equal_split = np.split(x, 3)
print("np.split(x, 3):")
for i, sub in enumerate(equal_split):
    print(f"  Sub-array {i}: {sub}")
print()

# Step 4: Create another array of 8 elements.
y = np.arange(8.0)
print("Array y:", y)
print()

# Step 5: Split at specific indices [3, 5, 6, 10].
index_split = np.split(y, [3, 5, 6, 10])
print("np.split(y, [3, 5, 6, 10]):")
for i, sub in enumerate(index_split):
    print(f"  Sub-array {i}: {sub}")
print("  (last sub-array is empty because index 10 exceeds length 8)")
print()

# Step 6: Attempt an unequal split (raises error).
try:
    np.split(y, 3)  # 8 is not divisible by 3
except ValueError as e:
    print("np.split(y, 3) raises:", e)
```

**Expected Output:**

```
Original array: [0. 1. 2. 3. 4. 5. 6. 7. 8.]

np.split(x, 3):
  Sub-array 0: [0. 1. 2.]
  Sub-array 1: [3. 4. 5.]
  Sub-array 2: [6. 7. 8.]

Array y: [0. 1. 2. 3. 4. 5. 6. 7.]

np.split(y, [3, 5, 6, 10]):
  Sub-array 0: [0. 1. 2.]
  Sub-array 1: [3. 4.]
  Sub-array 2: [5.]
  Sub-array 3: [6. 7.]
  Sub-array 4: []
  (last sub-array is empty because index 10 exceeds length 8)

np.split(y, 3) raises: array split does not result in an equal division
```

**Why This Result Occurs:** For the 9-element array, `np.split(x, 3)` divides it into three equal parts of 3 elements each. For the 8-element array, `np.split(y, [3, 5, 6, 10])` creates five sub-arrays: the first three elements, elements at indices 3–4, the element at index 5, elements at indices 6–7, and an empty array because index 10 exceeds the array length. Attempting `np.split(y, 3)` raises a `ValueError` because 8 is not evenly divisible by 3.

### Real-World Cases

- **Cross-Validation:** Splitting a dataset into K equal folds for K-fold cross-validation.
- **Data Parallelism:** Dividing an array into equal chunks for distribution across multiple processors.
- **Batch Processing:** Splitting a large batch into smaller equal-sized mini-batches.

### References

- numpy.split – https://numpy.org/doc/1.20/reference/generated/numpy.split.html

---

## Core Concept 2: array_split()

### Definitions

**Core Definition:** `array_split()` divides an array into multiple sub-arrays of equal or near-equal size, safely adapting when the array's axis length is not perfectly divisible by the split factor.

**Technical Definition:** `numpy.array_split(ary, indices_or_sections, axis=0)` splits an array into multiple sub-arrays. The only difference between this function and `split` is that `array_split` allows `indices_or_sections` to be an integer that does not equally divide the axis. For an array of length `l` that should be split into `n` sections, it returns `l % n` sub-arrays of size `l//n + 1` and the rest of size `l//n`.

**Beginner-Friendly Explanation:** `array_split()` is like `split()`, but it's more forgiving. If you have 10 items and want to split them into 3 groups, `split()` would fail, but `array_split()` will give you groups of 4, 3, and 3 (or similar) without complaining. It's the practical choice when you need near-equal splits.

### Purposes

- To divide an array into near-equal sub-arrays when equal division is not possible.
- To safely split arrays of any length without pre-checking divisibility.
- To distribute data as evenly as possible across a fixed number of partitions.
- To provide a more robust alternative to `split()` for real-world data.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
numpy.array_split(ary, indices_or_sections, axis=0)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `ary` | Input array to be divided into sub-arrays. |
| `indices_or_sections` | Int or 1-D array. If an integer `N`, divide into `N` near-equal arrays. |
| `axis` | Int, optional. The axis along which to split. Default is 0. |

**Syntax Rules:**

- If `indices_or_sections` is an integer `N`, the array is divided into `N` sub-arrays. The first `l % n` sub-arrays have size `l//n + 1`, and the rest have size `l//n`.
- If `indices_or_sections` is a 1-D array, it behaves identically to `split`.
- The result is a list of sub-arrays as views into `ary`.

**Constraints and Limitations:**

- The number of sections must not exceed the size of the array along the specified axis.
- Splitting returns views, not copies; modifications to sub-arrays affect the original.
- If `indices_or_sections` is a floating-point number, it is truncated towards zero before splitting.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Near-Equal Splitting of Arrays**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create a 1-D array of 8 elements.
x = np.arange(8.0)
print("Original array x:", x)
print()

# Step 3: Split into 3 near-equal parts.
split_3 = np.array_split(x, 3)
print("np.array_split(x, 3):")
for i, sub in enumerate(split_3):
    print(f"  Sub-array {i}: {sub}")
print("  (sizes: 3, 3, 2)")
print()

# Step 4: Create a 1-D array of 9 elements.
y = np.arange(9)
print("Original array y:", y)
print()

# Step 5: Split into 4 near-equal parts.
split_4 = np.array_split(y, 4)
print("np.array_split(y, 4):")
for i, sub in enumerate(split_4):
    print(f"  Sub-array {i}: {sub}")
print("  (sizes: 3, 2, 2, 2)")
print()

# Step 6: Compare with split() on the same array.
try:
    np.split(y, 4)
except ValueError as e:
    print("np.split(y, 4) raises:", e)
```

**Expected Output:**

```
Original array x: [0. 1. 2. 3. 4. 5. 6. 7.]

np.array_split(x, 3):
  Sub-array 0: [0. 1. 2.]
  Sub-array 1: [3. 4. 5.]
  Sub-array 2: [6. 7.]
  (sizes: 3, 3, 2)

Original array y: [0 1 2 3 4 5 6 7 8]

np.array_split(y, 4):
  Sub-array 0: [0 1 2]
  Sub-array 1: [3 4]
  Sub-array 2: [5 6]
  Sub-array 3: [7 8]
  (sizes: 3, 2, 2, 2)

np.split(y, 4) raises: array split does not result in an equal division
```

**Why This Result Occurs:** For the 8-element array split into 3 parts, `8 % 3 = 2`, so the first 2 sub-arrays have size `8//3 + 1 = 3`, and the remaining 1 sub-array has size `8//3 = 2`. For the 9-element array split into 4 parts, `9 % 4 = 1`, so the first 1 sub-array has size `9//4 + 1 = 3`, and the remaining 3 sub-arrays have size `9//4 = 2`. `split()` raises an error because 9 is not evenly divisible by 4.

### Real-World Cases

- **Train/Validation Split:** Dividing a dataset of arbitrary size into training and validation sets with near-equal proportions.
- **Mini-Batch Creation:** Splitting a dataset into a fixed number of mini-batches when the dataset size is not divisible by the batch size.
- **Parallel Processing:** Distributing work across a fixed number of workers as evenly as possible.

### References

- numpy.array_split – https://numpy.org/doc/2.2/numpy-ref.pdf#277#153

---

## Core Concept 3: hsplit()

### Definitions

**Core Definition:** `hsplit()` splits an array into multiple sub-arrays horizontally (column-wise) along axis 1.

**Technical Definition:** `numpy.hsplit(ary, indices_or_sections)` splits an array into multiple sub-arrays horizontally (column-wise). `hsplit` is equivalent to `split` with `axis=1`, the array is always split along the second axis except for 1-D arrays, where it is split at `axis=0`.

**Beginner-Friendly Explanation:** `hsplit()` cuts an array into columns. If you have a table, `hsplit()` divides it vertically into separate column groups. For 1-D arrays, it simply splits the array into segments.

### Purposes

- To split arrays horizontally (column-wise) into sub-arrays.
- To separate feature columns from a data matrix.
- To divide 1-D arrays into segments.
- To provide a more intuitive alternative to `split(axis=1)`.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
numpy.hsplit(ary, indices_or_sections)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `ary` | Input array to be divided into sub-arrays. |
| `indices_or_sections` | Int or 1-D array. If an integer `N`, divide into `N` equal arrays. If a 1-D array of sorted integers, the entries indicate split points. |

**Syntax Rules:**

- For arrays with 2+ dimensions, `hsplit` splits along axis 1 (columns).
- For 1-D arrays, `hsplit` splits along axis 0 (the only axis).
- The result is a list of sub-arrays as views into `ary`.

**Constraints and Limitations:**

- If `indices_or_sections` is an integer and the split does not result in equal division, a `ValueError` is raised.
- The number of sections must not exceed the size of the array along the split axis.
- Splitting returns views, not copies.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Horizontal Splitting of 2-D and 1-D Arrays**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create a 4x4 matrix.
x = np.arange(16.0).reshape(4, 4)
print("Original 4x4 matrix:\n", x)
print()

# Step 3: Split into 2 equal halves horizontally.
hsplit_2 = np.hsplit(x, 2)
print("np.hsplit(x, 2):")
for i, sub in enumerate(hsplit_2):
    print(f"  Sub-array {i} (shape {sub.shape}):\n{sub}")
print()

# Step 4: Split at specific column indices [3, 6].
hsplit_idx = np.hsplit(x, np.array([3, 6]))
print("np.hsplit(x, np.array([3, 6])):")
for i, sub in enumerate(hsplit_idx):
    print(f"  Sub-array {i} (shape {sub.shape}):\n{sub}")
print()

# Step 5: Split a 1-D array horizontally.
y = np.arange(9.0)
print("1-D array y:", y)
hsplit_1d = np.hsplit(y, 3)
print("np.hsplit(y, 3):")
for i, sub in enumerate(hsplit_1d):
    print(f"  Sub-array {i}: {sub}")
```

**Expected Output:**

```
Original 4x4 matrix:
 [[ 0.  1.  2.  3.]
 [ 4.  5.  6.  7.]
 [ 8.  9. 10. 11.]
 [12. 13. 14. 15.]]

np.hsplit(x, 2):
  Sub-array 0 (shape (4, 2)):
[[ 0.  1.]
 [ 4.  5.]
 [ 8.  9.]
 [12. 13.]]
  Sub-array 1 (shape (4, 2)):
[[ 2.  3.]
 [ 6.  7.]
 [10. 11.]
 [14. 15.]]

np.hsplit(x, np.array([3, 6])):
  Sub-array 0 (shape (4, 3)):
[[ 0.  1.  2.]
 [ 4.  5.  6.]
 [ 8.  9. 10.]
 [12. 13. 14.]]
  Sub-array 1 (shape (4, 1)):
[[ 3.]
 [ 7.]
 [11.]
 [15.]]
  Sub-array 2 (shape (4, 0)):
[]

1-D array y: [0. 1. 2. 3. 4. 5. 6. 7. 8.]
np.hsplit(y, 3):
  Sub-array 0: [0. 1. 2.]
  Sub-array 1: [3. 4. 5.]
  Sub-array 2: [6. 7. 8.]
```

**Why This Result Occurs:** For the 4×4 matrix, `np.hsplit(x, 2)` splits the four columns into two groups of two, producing two 4×2 sub-arrays. `np.hsplit(x, [3, 6])` splits at column indices 3 and 6, producing a 4×3 sub-array (columns 0–2), a 4×1 sub-array (column 3), and an empty 4×0 sub-array (since index 6 exceeds the 4 columns). For the 1-D array, `hsplit` behaves like `split` along axis 0.

### Real-World Cases

- **Feature Separation:** Separating a feature matrix into groups of columns for different processing pipelines.
- **Image Tiling:** Splitting an image into vertical strips for parallel processing.
- **Data Partitioning:** Dividing a table into column subsets for distributed storage.

### References

- numpy.hsplit – https://numpy.org/doc/2.2/numpy-ref.pdf#277#153

---

## Core Concept 4: vsplit()

### Definitions

**Core Definition:** `vsplit()` splits an array into multiple sub-arrays vertically (row-wise) along axis 0.

**Technical Definition:** `numpy.vsplit(ary, indices_or_sections)` splits an array into multiple sub-arrays vertically (row-wise). `vsplit` is equivalent to `split` with `axis=0` (default), the array is always split along the first axis regardless of the array dimension.

**Beginner-Friendly Explanation:** `vsplit()` cuts an array into rows. If you have a table, `vsplit()` divides it horizontally into separate row groups. It's the row-wise counterpart to `hsplit()`.

### Purposes

- To split arrays vertically (row-wise) into sub-arrays.
- To divide a data matrix into row groups (e.g., train/test splits).
- To separate samples from a dataset.
- To provide a more intuitive alternative to `split(axis=0)`.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
numpy.vsplit(ary, indices_or_sections)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `ary` | Input array to be divided into sub-arrays. |
| `indices_or_sections` | Int or 1-D array. If an integer `N`, divide into `N` equal arrays. If a 1-D array of sorted integers, the entries indicate split points. |

**Syntax Rules:**

- `vsplit` always splits along axis 0 (rows), regardless of the array's dimensionality.
- The result is a list of sub-arrays as views into `ary`.

**Constraints and Limitations:**

- If `indices_or_sections` is an integer and the split does not result in equal division, a `ValueError` is raised.
- The number of sections must not exceed the size of the array along axis 0.
- Splitting returns views, not copies.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Vertical Splitting of a Matrix**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create a 6x4 matrix.
x = np.arange(24.0).reshape(6, 4)
print("Original 6x4 matrix:\n", x)
print()

# Step 3: Split into 3 equal row groups.
vsplit_3 = np.vsplit(x, 3)
print("np.vsplit(x, 3):")
for i, sub in enumerate(vsplit_3):
    print(f"  Sub-array {i} (shape {sub.shape}):\n{sub}")
print()

# Step 4: Split at specific row indices [1, 4].
vsplit_idx = np.vsplit(x, np.array([1, 4]))
print("np.vsplit(x, np.array([1, 4])):")
for i, sub in enumerate(vsplit_idx):
    print(f"  Sub-array {i} (shape {sub.shape}):\n{sub}")
```

**Expected Output:**

```
Original 6x4 matrix:
 [[ 0.  1.  2.  3.]
 [ 4.  5.  6.  7.]
 [ 8.  9. 10. 11.]
 [12. 13. 14. 15.]
 [16. 17. 18. 19.]
 [20. 21. 22. 23.]]

np.vsplit(x, 3):
  Sub-array 0 (shape (2, 4)):
[[ 0.  1.  2.  3.]
 [ 4.  5.  6.  7.]]
  Sub-array 1 (shape (2, 4)):
[[ 8.  9. 10. 11.]
 [12. 13. 14. 15.]]
  Sub-array 2 (shape (2, 4)):
[[16. 17. 18. 19.]
 [20. 21. 22. 23.]]

np.vsplit(x, np.array([1, 4])):
  Sub-array 0 (shape (1, 4)):
[[0. 1. 2. 3.]]
  Sub-array 1 (shape (3, 4)):
[[ 4.  5.  6.  7.]
 [ 8.  9. 10. 11.]
 [12. 13. 14. 15.]]
  Sub-array 2 (shape (2, 4)):
[[16. 17. 18. 19.]
 [20. 21. 22. 23.]]
```

**Why This Result Occurs:** `np.vsplit(x, 3)` divides the 6 rows into three groups of 2 rows each. `np.vsplit(x, [1, 4])` splits at row indices 1 and 4, producing a 1×4 sub-array (row 0), a 3×4 sub-array (rows 1–3), and a 2×4 sub-array (rows 4–5).

### Real-World Cases

- **Train/Test Split:** Splitting a dataset into training and testing sets row-wise.
- **Batch Processing:** Dividing a large batch of samples into smaller mini-batches.
- **Time Series Segmentation:** Splitting a time series into segments along the time axis.

### References

- numpy.vsplit – https://numpy.org/doc/2.2/numpy-ref.pdf#277#153

---

## Core Concept 5: dsplit()

### Definitions

**Core Definition:** `dsplit()` splits an array into multiple sub-arrays along the third axis (depth, axis 2).

**Technical Definition:** `numpy.dsplit(ary, indices_or_sections)` splits an array into multiple sub-arrays along the 3rd axis (depth). `dsplit` is equivalent to `split` with `axis=2`, the array is always split along the third axis provided the array dimension is greater than or equal to 3.

**Beginner-Friendly Explanation:** `dsplit()` cuts an array into layers along the depth dimension. If you have a stack of images or a volume of data, `dsplit()` separates it into depth slices. It only works on arrays with at least 3 dimensions.

### Purposes

- To split arrays along the depth (third) axis into sub-arrays.
- To separate channels or slices from a 3-D volume.
- To divide multi-channel data for parallel processing.
- To provide a more intuitive alternative to `split(axis=2)`.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
numpy.dsplit(ary, indices_or_sections)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `ary` | Input array to be divided into sub-arrays. Must have at least 3 dimensions. |
| `indices_or_sections` | Int or 1-D array. If an integer `N`, divide into `N` equal arrays. If a 1-D array of sorted integers, the entries indicate split points. |

**Syntax Rules:**

- `dsplit` always splits along axis 2 (depth).
- The input array must have `ndim >= 3`.
- The result is a list of sub-arrays as views into `ary`.

**Constraints and Limitations:**

- If the array has fewer than 3 dimensions, a `ValueError` is raised.
- If `indices_or_sections` is an integer and the split does not result in equal division, a `ValueError` is raised.
- Splitting returns views, not copies.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Depth Splitting of a 3-D Array**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create a 3-D array of shape (2, 2, 4).
x = np.arange(16.0).reshape(2, 2, 4)
print("Original 3-D array (shape 2x2x4):\n", x)
print()

# Step 3: Split into 2 equal depth slices.
dsplit_2 = np.dsplit(x, 2)
print("np.dsplit(x, 2):")
for i, sub in enumerate(dsplit_2):
    print(f"  Sub-array {i} (shape {sub.shape}):\n{sub}")
print()

# Step 4: Split at specific depth indices [3, 6].
dsplit_idx = np.dsplit(x, np.array([3, 6]))
print("np.dsplit(x, np.array([3, 6])):")
for i, sub in enumerate(dsplit_idx):
    print(f"  Sub-array {i} (shape {sub.shape}):\n{sub}")
print()

# Step 5: Attempt dsplit on a 2-D array (raises error).
try:
    np.dsplit(np.arange(6).reshape(2, 3), 2)
except ValueError as e:
    print("np.dsplit on 2-D array raises:", e)
```

**Expected Output:**

```
Original 3-D array (shape 2x2x4):
 [[[ 0.  1.  2.  3.]
  [ 4.  5.  6.  7.]]

 [[ 8.  9. 10. 11.]
  [12. 13. 14. 15.]]]

np.dsplit(x, 2):
  Sub-array 0 (shape (2, 2, 2)):
[[[ 0.  1.]
  [ 4.  5.]]

 [[ 8.  9.]
  [12. 13.]]]
  Sub-array 1 (shape (2, 2, 2)):
[[[ 2.  3.]
  [ 6.  7.]]

 [[10. 11.]
  [14. 15.]]]

np.dsplit(x, np.array([3, 6])):
  Sub-array 0 (shape (2, 2, 3)):
[[[ 0.  1.  2.]
  [ 4.  5.  6.]]

 [[ 8.  9. 10.]
  [12. 13. 14.]]]
  Sub-array 1 (shape (2, 2, 1)):
[[[ 3.]
  [ 7.]]

 [[11.]
  [15.]]]
  Sub-array 2 (shape (2, 2, 0)):
[]

np.dsplit on 2-D array raises: dsplit only works on arrays of 3 or more dimensions
```

**Why This Result Occurs:** For the 3-D array of shape (2, 2, 4), `np.dsplit(x, 2)` splits the depth dimension (size 4) into two equal parts of size 2, producing two sub-arrays of shape (2, 2, 2). `np.dsplit(x, [3, 6])` splits at depth indices 3 and 6, producing a sub-array of shape (2, 2, 3), a sub-array of shape (2, 2, 1), and an empty sub-array of shape (2, 2, 0). Attempting to use `dsplit` on a 2-D array raises a `ValueError` because the function requires at least 3 dimensions.

### Real-World Cases

- **Multi-Channel Image Processing:** Separating color channels (RGB) from a 3-D image array.
- **Medical Imaging:** Splitting a 3-D volume into individual slices for analysis.
- **Video Processing:** Separating frames from a depth-stacked video array.

### References

- numpy.dsplit – https://numpy.org/doc/2.2/numpy-ref.pdf#277#153

---

## Comparison Summary

| Function | Splits Along | Dimensionality Requirement | Equal Division Required |
|----------|-------------|---------------------------|------------------------|
| `split` | Any axis | Any | Yes (integer sections) |
| `array_split` | Any axis | Any | No |
| `hsplit` | Axis 1 (or 0 for 1-D) | Any | Yes (integer sections) |
| `vsplit` | Axis 0 | Any | Yes (integer sections) |
| `dsplit` | Axis 2 | ≥3-D | Yes (integer sections) |

---

## Summary of Deprecated and Version-Specific Features

| Feature | Status | Recommendation |
|---------|--------|----------------|
| `array_split` with float `indices_or_sections` | Truncated to int since NumPy 1.23 | Use integer values explicitly |
| `np.ma.split` for masked arrays | Use `np.ma.concatenate` for masked array operations | Not applicable for standard ndarrays |

---

## Consolidated Reference List

- numpy.split – https://numpy.org/doc/1.20/reference/generated/numpy.split.html
- numpy.array_split – https://numpy.org/doc/2.2/numpy-ref.pdf#277#153
- numpy.hsplit – https://numpy.org/doc/2.2/numpy-ref.pdf#277#153
- numpy.vsplit – https://numpy.org/doc/2.2/numpy-ref.pdf#277#153
- numpy.dsplit – https://numpy.org/doc/2.2/numpy-ref.pdf#277#153
- NumPy Array manipulation routines – https://numpy.org/doc/stable/reference/routines.array-manipulation.html
- W3Schools: NumPy Splitting Array – https://www.w3schools.com/python/numpy/numpy_array_split.asp
- NumPy Quickstart (Splitting) – https://numpy.org/doc/stable/user/quickstart.html