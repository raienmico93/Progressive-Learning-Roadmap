# NumPy Adding and Removing Dimensions: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Adding and removing dimensions in NumPy refers to the manipulation of an array's shape by inserting new axes of length 1 (singleton dimensions) or removing existing length-1 axes, without altering the underlying data values.

**Technical Definition:** These operations modify the array's `shape` and `strides` metadata while leaving the data buffer untouched. `expand_dims()` and `np.newaxis` insert new axes into the shape tuple, while `squeeze()` removes axes whose size equals 1. All operations return views whenever possible.

**Beginner-Friendly Explanation:** Sometimes you need to change how many "directions" an array has without changing the numbers inside. Adding a dimension is like putting a single row of numbers into its own frame, turning a 1-D list into a 2-D table with one row. Removing a dimension is the reverse—stripping away redundant "frames" that contain only one row or column. NumPy makes these changes without copying data, so they're instant.

### Key Characteristics

- **View Semantics:** Both `expand_dims()` and `squeeze()` return views when possible, not copies.
- **Singleton Dimensions:** Dimensions of length 1 are called singleton dimensions; they affect broadcasting behavior.
- **O(1) Operations:** Adding or removing dimensions only changes metadata (shape and strides), so it's instant regardless of array size.
- **Broadcasting Control:** Inserting singleton dimensions is a primary technique for controlling NumPy's broadcasting rules.
- **Inverse Operations:** `expand_dims()` and `squeeze()` are inverses of each other.

### Prerequisites

- NumPy installed (`pip install numpy`).
- Understanding of array shapes, axes, and dimensions.
- Familiarity with the `import numpy as np` convention.
- Conceptual understanding of broadcasting.

### Related Programming Areas

- **Broadcasting:** Inserting dimensions to align arrays of different shapes for element-wise operations.
- **Machine Learning:** Adding batch dimensions to single samples for model inference.
- **Image Processing:** Converting between flat pixel vectors and 2-D/3-D image representations.
- **Data Alignment:** Reshaping data to match expected input formats for libraries and APIs.

### Core Concepts / Features

The following core concepts are explored in detail: (1) `expand_dims()`, (2) `squeeze()`, (3) `newaxis`, and (4) Singleton dimensions.

---

## Core Concept 1: expand_dims()

### Definitions

**Core Definition:** `expand_dims()` inserts a new axis of length 1 at a specified position in an array's shape.

**Technical Definition:** `numpy.expand_dims(a, axis)` returns a view of `a` with the number of dimensions increased by one. The new axis is placed at the position given by `axis` in the expanded array shape. The `axis` parameter can be an integer or, since NumPy 1.18.0, a tuple of integers to insert multiple axes simultaneously. Out-of-range axes raise an `AxisError`.

**Beginner-Friendly Explanation:** `expand_dims()` lets you add a "fake" dimension of size 1 to your array. For example, a 1-D array of shape `(5,)` can become a 2-D array of shape `(1, 5)` or `(5, 1)`, depending on where you insert the new axis. This is useful when a function expects a 2-D input but you only have a 1-D array.

### Purposes

- To add a singleton dimension for broadcasting with arrays of different shapes.
- To prepare a 1-D array for operations that require a 2-D input (e.g., matrix multiplication).
- To insert a batch dimension for processing single samples in a model that expects batches.
- To align array shapes for element-wise operations between arrays of different dimensionalities.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
numpy.expand_dims(a, axis)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `a` | Input array (array_like). |
| `axis` | Integer or tuple of integers specifying the position(s) of the new axis/axes. |
| `axis=0` | Insert at the beginning (becomes the first axis). |
| `axis=1` | Insert at position 1. |
| `axis=-1` | Insert at the end (becomes the last axis). |
| `axis=(0, 1)` | Insert axes at positions 0 and 1 (available since NumPy 1.18.0). |

**Syntax Rules:**

- The `axis` value must be in the range `[-(a.ndim+1), a.ndim]`.
- The `axis` parameter can be a tuple of integers (since NumPy 1.18.0), allowing multiple axes to be inserted at once.
- The result always has `a.ndim + 1` dimensions (or more if a tuple is used).
- The operation returns a view, not a copy.

**Constraints and Limitations:**

- Passing an `axis` where `axis > a.ndim` was deprecated since NumPy 1.13.0 and now raises an `AxisError`.
- Negative `axis` values count from the end of the new shape.
- The `expand_dims` function is the programmatic equivalent of using `np.newaxis` in indexing.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Adding Dimensions with expand_dims()**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create a 1-D array.
x = np.array([1, 2, 3, 4])
print("Original array:", x)
print("Original shape:", x.shape)
print()

# Step 3: Add a dimension at axis 0.
y0 = np.expand_dims(x, axis=0)
print("expand_dims(x, axis=0):", y0)
print("Shape:", y0.shape)
print("  (equivalent to x[np.newaxis, :])")
print()

# Step 4: Add a dimension at axis 1.
y1 = np.expand_dims(x, axis=1)
print("expand_dims(x, axis=1):\n", y1)
print("Shape:", y1.shape)
print("  (equivalent to x[:, np.newaxis])")
print()

# Step 5: Add a dimension at the end using negative axis.
y_neg = np.expand_dims(x, axis=-1)
print("expand_dims(x, axis=-1):\n", y_neg)
print("Shape:", y_neg.shape)
print()

# Step 6: Add multiple dimensions with a tuple (NumPy 1.18.0+).
y_multi = np.expand_dims(x, axis=(0, 1))
print("expand_dims(x, axis=(0, 1)):\n", y_multi)
print("Shape:", y_multi.shape)
print()

# Step 7: Verify view semantics.
print("y0.base is x:", y0.base is x)
print("Shares memory:", np.shares_memory(x, y0))
```

**Expected Output:**

```
Original array: [1 2 3 4]
Original shape: (4,)

expand_dims(x, axis=0): [[1 2 3 4]]
Shape: (1, 4)
  (equivalent to x[np.newaxis, :])

expand_dims(x, axis=1):
 [[1]
 [2]
 [3]
 [4]]
Shape: (4, 1)
  (equivalent to x[:, np.newaxis])

expand_dims(x, axis=-1):
 [[1]
 [2]
 [3]
 [4]]
Shape: (4, 1)

expand_dims(x, axis=(0, 1)):
 [[[1 2 3 4]]]
Shape: (1, 1, 4)

y0.base is x: True
Shares memory: True
```

**Why This Result Occurs:** `axis=0` inserts a new first dimension, turning the 1-D array into a 2-D row vector of shape `(1, 4)`. `axis=1` inserts a new second dimension, turning it into a 2-D column vector of shape `(4, 1)`. `axis=-1` inserts at the end, producing the same shape as `axis=1`. The tuple `(0, 1)` inserts two axes, producing a 3-D array of shape `(1, 1, 4)`. The `.base` attribute and `shares_memory` confirm the result is a view.

### Real-World Cases

- **Single-Sample Inference:** When a model expects a batch dimension, `np.expand_dims(sample, axis=0)` converts a single sample into a batch of size 1.
- **Broadcasting:** Computing the outer product `a[:, np.newaxis] * b[np.newaxis, :]` requires expanding both 1-D arrays into 2-D shapes.
- **Image Processing:** Converting a grayscale image from shape `(H, W)` to `(H, W, 1)` for a model that expects a channel dimension.

### References

- numpy.ma.expand_dims – https://numpy.org/doc/1.26/reference/generated/numpy.ma.expand_dims.html
- numpy.expand_dims – https://numpy.org/doc/stable/reference/generated/numpy.expand_dims.html
- Stack Overflow: What is np.newaxis? – https://stackoverflow.com/revisions/f3ac356d-aba3-4931-8b18-bd76f958ad25/view-source

---

## Core Concept 2: squeeze()

### Definitions

**Core Definition:** `squeeze()` removes axes of length 1 from an array's shape, reducing its dimensionality.

**Technical Definition:** `numpy.squeeze(a, axis=None)` removes single-dimensional entries from the shape of an array. If `axis` is `None`, all axes of length 1 are removed. If `axis` is specified (int or tuple of ints), only the selected axes are removed. If an axis selected for removal has a size greater than 1, a `ValueError` is raised. The result is always a view of the input array.

**Beginner-Friendly Explanation:** `squeeze()` is the opposite of `expand_dims()`. It strips away dimensions that contain only one element, simplifying the array's shape. For example, a `(1, 5, 1)` array becomes `(5,)`. This is useful when operations leave behind redundant singleton dimensions.

### Purposes

- To remove redundant singleton dimensions that result from slicing or reshaping operations.
- To simplify array shapes for downstream processing.
- To convert a single-element array back to a scalar (0-D array).
- To clean up shapes after broadcasting operations.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
numpy.squeeze(a, axis=None)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `a` | Input array (array_like). |
| `axis=None` | Remove all axes of length 1. |
| `axis=int` | Remove only the specified axis (must be of length 1). |
| `axis=tuple` | Remove only the specified axes (all must be of length 1). |

**Syntax Rules:**

- If `axis` is specified, the selected axis must have a shape entry of 1; otherwise, a `ValueError` is raised.
- The result is always a view, not a copy.
- If all axes are squeezed, the result is a 0-D array, not a Python scalar.
- Negative `axis` values are allowed and count from the end.

**Constraints and Limitations:**

- You cannot squeeze an axis whose length is greater than 1.
- The total number of elements must remain the same.
- `squeeze()` with `axis` specified selects a subset of singleton dimensions to remove, while `axis=None` removes all of them.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Removing Singleton Dimensions with squeeze()**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create an array with singleton dimensions.
x = np.array([[[0], [1], [2]]])
print("Original array:\n", x)
print("Original shape:", x.shape)
print()

# Step 3: Remove all singleton dimensions.
squeezed_all = np.squeeze(x)
print("squeeze(x):\n", squeezed_all)
print("Shape:", squeezed_all.shape)
print("  (all length-1 axes removed)")
print()

# Step 4: Remove only axis 0.
squeezed_axis0 = np.squeeze(x, axis=0)
print("squeeze(x, axis=0):\n", squeezed_axis0)
print("Shape:", squeezed_axis0.shape)
print()

# Step 5: Remove only axis 2.
squeezed_axis2 = np.squeeze(x, axis=2)
print("squeeze(x, axis=2):\n", squeezed_axis2)
print("Shape:", squeezed_axis2.shape)
print()

# Step 6: Attempt to squeeze a non-singleton axis (raises error).
try:
    np.squeeze(x, axis=1)  # axis 1 has size 3
except ValueError as e:
    print("squeeze(x, axis=1) raises:", e)
print()

# Step 7: Squeeze to a 0-D array.
y = np.array([[1234]])
print("Original y shape:", y.shape)
squeezed_y = np.squeeze(y)
print("Squeezed y:", squeezed_y)
print("Type:", type(squeezed_y))
print("Shape:", squeezed_y.shape)
```

**Expected Output:**

```
Original array:
 [[[0]
  [1]
  [2]]]
Original shape: (1, 3, 1)

squeeze(x):
 [0 1 2]
Shape: (3,)
  (all length-1 axes removed)

squeeze(x, axis=0):
 [[0]
 [1]
 [2]]
Shape: (3, 1)

squeeze(x, axis=2):
 [[0 1 2]]
Shape: (1, 3)

squeeze(x, axis=1) raises: cannot select an axis to squeeze out which has size not equal to one

Original y shape: (1, 1)
Squeezed y: 1234
Type: <class 'numpy.ndarray'>
Shape: ()
```

**Why This Result Occurs:** The original array has shape `(1, 3, 1)`. `np.squeeze(x)` removes both length-1 axes (axes 0 and 2), leaving shape `(3,)`. `axis=0` removes only the first axis, leaving `(3, 1)`. `axis=2` removes only the last axis, leaving `(1, 3)`. Attempting to squeeze axis 1 (size 3) raises a `ValueError`. Squeezing a `(1, 1)` array produces a 0-D array, which is an `ndarray` with shape `()`.

### Real-World Cases

- **Post-Processing:** After extracting a single channel from an image `(H, W, 1)`, squeeze to `(H, W)` for display.
- **Model Output:** Removing the batch dimension from a single-sample prediction: `prediction.squeeze(axis=0)`.
- **Data Loading:** Cleaning up shapes after loading data from files that include redundant singleton dimensions.

### References

- numpy.squeeze – https://numpy.org/doc/1.22/reference/generated/numpy.squeeze.html
- numpy.ndarray.squeeze – https://numpy.org/doc/stable/reference/generated/numpy.ndarray.squeeze.html

---

## Core Concept 3: newaxis

### Definitions

**Core Definition:** `np.newaxis` is a constant alias for `None` that, when used in array indexing, inserts a new axis of length 1 at the specified position.

**Technical Definition:** `np.newaxis is None` evaluates to `True`. It is used within index brackets to add dimensions to an array. For example, `a[np.newaxis, :]` adds a new first axis, and `a[:, np.newaxis]` adds a new second axis. The position of `np.newaxis` in the index tuple determines where the new axis is inserted.

**Beginner-Friendly Explanation:** `np.newaxis` is a convenient way to add a dimension to an array using the familiar indexing syntax. Instead of calling a function, you just write `np.newaxis` where you want the new dimension to go. It's a shorthand that makes code cleaner and more readable.

### Purposes

- To add singleton dimensions using indexing syntax instead of a function call.
- To provide a more readable alternative to `None` in indexing expressions.
- To insert dimensions on the fly for broadcasting operations.
- To align array shapes without calling `expand_dims()`.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Add a new first axis
a[np.newaxis, :]

# Add a new second axis
a[:, np.newaxis]

# Add multiple new axes
a[np.newaxis, :, np.newaxis]
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `np.newaxis` | Alias for `None`; inserts a new axis of length 1. |
| `a[np.newaxis, :]` | Shape becomes `(1,) + a.shape`. |
| `a[:, np.newaxis]` | Shape becomes `a.shape + (1,)`. |
| Multiple `np.newaxis` | Insert multiple singleton axes. |

**Syntax Rules:**

- `np.newaxis` is exactly `None`; they are interchangeable.
- The position in the index tuple determines where the new axis is inserted.
- Any number of `np.newaxis` entries can be used in a single indexing expression.

**Constraints and Limitations:**

- `np.newaxis` only works within indexing brackets, not as a standalone array operation.
- Using `None` directly works identically but is less readable.
- The result is always a view of the original array.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Using np.newaxis for Dimension Insertion**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create a 1-D array.
a = np.array([1, 2, 3, 4, 5, 6])
print("Original array:", a)
print("Original shape:", a.shape)
print()

# Step 3: Add a new first axis (row vector).
a_row = a[np.newaxis, :]
print("a[np.newaxis, :]:", a_row)
print("Shape:", a_row.shape)
print("  (1 row, 6 columns)")
print()

# Step 4: Add a new second axis (column vector).
a_col = a[:, np.newaxis]
print("a[:, np.newaxis]:\n", a_col)
print("Shape:", a_col.shape)
print("  (6 rows, 1 column)")
print()

# Step 5: Add multiple new axes.
a_3d = a[np.newaxis, :, np.newaxis]
print("a[np.newaxis, :, np.newaxis] shape:", a_3d.shape)
print("  (1 layer, 6 rows, 1 column)")
print()

# Step 6: Verify that np.newaxis is None.
print("np.newaxis is None:", np.newaxis is None)
print()

# Step 7: Use newaxis for broadcasting.
# Compute pairwise differences between elements.
diff = a[:, np.newaxis] - a[np.newaxis, :]
print("Pairwise differences (a[:, newaxis] - a[newaxis, :]):")
print(diff)
```

**Expected Output:**

```
Original array: [1 2 3 4 5 6]
Original shape: (6,)

a[np.newaxis, :]: [[1 2 3 4 5 6]]
Shape: (1, 6)
  (1 row, 6 columns)

a[:, np.newaxis]:
 [[1]
 [2]
 [3]
 [4]
 [5]
 [6]]
Shape: (6, 1)
  (6 rows, 1 column)

a[np.newaxis, :, np.newaxis] shape: (1, 6, 1)
  (1 layer, 6 rows, 1 column)

np.newaxis is None: True

Pairwise differences (a[:, newaxis] - a[newaxis, :]):
 [[ 0  1  2  3  4  5]
 [-1  0  1  2  3  4]
 [-2 -1  0  1  2  3]
 [-3 -2 -1  0  1  2]
 [-4 -3 -2 -1  0  1]
 [-5 -4 -3 -2 -1  0]]
```

**Why This Result Occurs:** `a[np.newaxis, :]` inserts a new axis at position 0, producing shape `(1, 6)`. `a[:, np.newaxis]` inserts a new axis at position 1, producing shape `(6, 1)`. Using both `np.newaxis` entries produces shape `(1, 6, 1)`. The pairwise difference computation uses broadcasting: `a[:, np.newaxis]` has shape `(6, 1)` and `a[np.newaxis, :]` has shape `(1, 6)`, and broadcasting expands them to `(6, 6)`.

### Real-World Cases

- **Broadcasting:** `a[:, np.newaxis] - b[np.newaxis, :]` computes all pairwise differences between two 1-D arrays.
- **Image Processing:** `image[np.newaxis, :, :]` adds a batch dimension to a single image for model inference.
- **Matrix Operations:** Converting a 1-D vector into a column vector for matrix multiplication.

### References

- Stack Overflow: What is np.newaxis? – https://stackoverflow.com/revisions/f3ac356d-aba3-4931-8b18-bd76f958ad25/view-source
- NumPy Broadcasting – https://numpy.org/doc/stable/user/basics.broadcasting.html

---

## Core Concept 4: Singleton Dimensions

### Definitions

**Core Definition:** A singleton dimension is a dimension of an array whose length is exactly 1.

**Technical Definition:** Singleton dimensions are axes with a shape entry of 1. They play a critical role in NumPy's broadcasting rules: when two arrays have different numbers of dimensions, the smaller array's shape is virtually padded with singleton dimensions on its left side. Singleton dimensions are also created by `expand_dims()`, `np.newaxis`, and certain slicing operations.

**Beginner-Friendly Explanation:** A singleton dimension is a "fake" dimension that contains only one element. It doesn't add any data, but it changes how NumPy aligns arrays during operations. For example, a `(3, 1)` array and a `(1, 4)` array can be broadcast together to produce a `(3, 4)` array because the singleton dimensions allow NumPy to expand them appropriately.

### Purposes

- To control broadcasting behavior between arrays of different shapes.
- To align array dimensions for element-wise operations.
- To represent batch, channel, or feature dimensions explicitly.
- To enable operations that require specific dimensional structures.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Creating a singleton dimension
a = np.array([1, 2, 3])
b = a[np.newaxis, :]       # Shape (1, 3)
c = a[:, np.newaxis]       # Shape (3, 1)

# Removing a singleton dimension
d = np.squeeze(b)           # Shape (3,)
```

**Component Breakdown:**

| Operation | Result Shape | Description |
|-----------|-------------|-------------|
| `a[np.newaxis, :]` | `(1,) + a.shape` | Adds a leading singleton dimension. |
| `a[:, np.newaxis]` | `a.shape + (1,)` | Adds a trailing singleton dimension. |
| `np.expand_dims(a, axis)` | Insert at `axis`. | Programmatic insertion. |
| `np.squeeze(a)` | Remove all size-1 axes. | Removes singleton dimensions. |

**Syntax Rules:**

- Singleton dimensions have a shape entry of exactly 1.
- Broadcasting aligns dimensions from the right; missing dimensions on the left are treated as singleton.
- Singleton dimensions can be explicitly created or removed as needed.

**Constraints and Limitations:**

- Too many singleton dimensions can make code harder to read; use them judiciously.
- Removing a singleton dimension that is needed for broadcasting may cause shape errors.
- `squeeze()` without `axis` removes all singleton dimensions, which may not always be desired.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Singleton Dimensions in Broadcasting**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create two 1-D arrays.
a = np.array([1, 2, 3])          # Shape (3,)
b = np.array([10, 20, 30, 40])   # Shape (4,)
print("a shape:", a.shape)
print("b shape:", b.shape)
print()

# Step 3: Attempt direct multiplication (raises error).
try:
    result = a * b
except ValueError as e:
    print("a * b raises:", e)
print()

# Step 4: Insert singleton dimensions to enable broadcasting.
a_col = a[:, np.newaxis]         # Shape (3, 1)
b_row = b[np.newaxis, :]         # Shape (1, 4)
print("a_col shape:", a_col.shape)
print("b_row shape:", b_row.shape)
print()

# Step 5: Perform broadcasting multiplication.
result = a_col * b_row
print("a_col * b_row:\n", result)
print("Result shape:", result.shape)
print("  (3x4 outer product)")
print()

# Step 6: Use keepdims to preserve singleton dimensions in reductions.
x = np.arange(24).reshape(2, 3, 4)
mean_keepdims = x.mean(axis=1, keepdims=True)
print("Original shape:", x.shape)
print("Mean over axis 1 (keepdims=True) shape:", mean_keepdims.shape)
print("  (singleton dimension preserved at axis 1)")
```

**Expected Output:**

```
a shape: (3,)
b shape: (4,)

a * b raises: operands could not be broadcast together with shapes (3,) (4,) 

a_col shape: (3, 1)
b_row shape: (1, 4)

a_col * b_row:
 [[ 10  20  30  40]
 [ 20  40  60  80]
 [ 30  60  90 120]]
Result shape: (3, 4)
  (3x4 outer product)

Original shape: (2, 3, 4)
Mean over axis 1 (keepdims=True) shape: (2, 1, 4)
  (singleton dimension preserved at axis 1)
```

**Why This Result Occurs:** Direct multiplication of `a` (shape `(3,)`) and `b` (shape `(4,)`) fails because their shapes are incompatible. Inserting singleton dimensions transforms `a` into a column vector `(3, 1)` and `b` into a row vector `(1, 4)`. Broadcasting then expands these to `(3, 4)`, producing the outer product. The `keepdims=True` parameter in the mean operation preserves the reduced axis as a singleton dimension, which is useful for subsequent broadcasting operations.

### Real-World Cases

- **Outer Products:** Computing all pairwise products between two vectors: `a[:, np.newaxis] * b[np.newaxis, :]`.
- **Normalization:** Subtracting the mean from each feature in a dataset: `X - X.mean(axis=0, keepdims=True)`.
- **Image Processing:** Adding a channel dimension to a grayscale image: `image[:, :, np.newaxis]`.
- **Batch Processing:** Adding a batch dimension to a single sample: `sample[np.newaxis, :]`.

### References

- NumPy Broadcasting – https://numpy.org/doc/stable/user/basics.broadcasting.html
- NumPy Discussion: Singleton Dimensions and Broadcasting – https://mail.python.org/archives/list/numpy-discussion@python.org/thread/53HKRAQYRROUXAIBJKPEL26N7HKZPSLL/?noscript
- Stack Overflow: Efficiently Defining a Multidimensional Array – https://mail.python.org/pipermail/numpy-discussion/2011-March/055666.html

---

## Summary of Deprecated and Version-Specific Features

| Feature | Status | Recommendation |
|---------|--------|----------------|
| `expand_dims` with `axis > a.ndim` | Deprecated since NumPy 1.13.0; raises `AxisError` | Use valid axis values within `[-(a.ndim+1), a.ndim]` |
| `expand_dims` with tuple `axis` | New in NumPy 1.18.0 | Available in NumPy 1.18+ only |
| `squeeze` with `axis` parameter | New in NumPy 1.7.0 | Available in NumPy 1.7+ only |
| `np.matrix` class | Should no longer be used | Use regular 2-D `ndarray` |

---

## Consolidated Reference List

- numpy.ma.expand_dims – https://numpy.org/doc/1.26/reference/generated/numpy.ma.expand_dims.html
- numpy.expand_dims – https://numpy.org/doc/stable/reference/generated/numpy.expand_dims.html
- numpy.squeeze – https://numpy.org/doc/1.22/reference/generated/numpy.squeeze.html
- numpy.ndarray.squeeze – https://numpy.org/doc/stable/reference/generated/numpy.ndarray.squeeze.html
- Stack Overflow: What is np.newaxis? – https://stackoverflow.com/revisions/f3ac356d-aba3-4931-8b18-bd76f958ad25/view-source
- NumPy Broadcasting – https://numpy.org/doc/stable/user/basics.broadcasting.html
- NumPy Discussion: Singleton Dimensions – https://mail.python.org/archives/list/numpy-discussion@python.org/thread/53HKRAQYRROUXAIBJKPEL26N7HKZPSLL/?noscript
- NumPy Discussion: Efficiently Defining a Multidimensional Array – https://mail.python.org/pipermail/numpy-discussion/2011-March/055666.html
- NumPy Absolute Beginners Guide – https://numpy.org/doc/stable/user/absolute_beginners.html