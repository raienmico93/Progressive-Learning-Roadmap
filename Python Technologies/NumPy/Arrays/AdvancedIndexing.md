# NumPy Advanced Indexing: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Advanced indexing (also called "fancy" indexing) is a NumPy indexing mechanism that uses arrays of indices—either integer arrays or boolean arrays—to select arbitrary elements from an array. Unlike basic slicing, which selects contiguous subarrays using slices and integers, advanced indexing allows for non-contiguous, out-of-order selection and always returns a copy of the data.

**Technical Definition:** Advanced indexing is triggered when the selection object `obj` is a non-tuple sequence object, an `ndarray` (of data type integer or bool), or a tuple with at least one sequence object or `ndarray` (of data type integer or bool). There are two types: integer-array indexing and Boolean indexing. Advanced indexing always returns a copy of the data, unlike basic slicing which returns a view.

**Beginner-Friendly Explanation:** Basic slicing is like cutting a rectangular piece of cake—you always get a contiguous block. Advanced indexing is like reaching into the array and picking out exactly the pieces you want, in any order, even if they're scattered all over. You can say "give me rows 2, 5, and 7" or "give me all the numbers greater than 10." Because NumPy has to gather these scattered elements, it creates a new array (a copy) rather than a view of the original.

### Key Characteristics

- **Non-Contiguous Selection:** Advanced indexing can select elements that are not stored next to each other in memory.
- **Copy Guarantee:** Advanced indexing always returns a copy, never a view. This means modifications to the result do not affect the original array.
- **Broadcasting:** When multiple index arrays are used, they are broadcast against each other to determine the shape of the result.
- **Two Types:** Integer-array indexing (fancy indexing) and Boolean indexing.
- **Combinable:** Advanced indexing can be mixed with basic slicing in a single bracket operation.

### Prerequisites

- NumPy installed (`pip install numpy`).
- Understanding of basic indexing and slicing (integers, slices, negative indices).
- Familiarity with the `import numpy as np` convention.
- Basic knowledge of array shapes and dimensions.

### Related Programming Areas

- **Data Filtering:** Selecting subsets of data based on conditions.
- **Machine Learning:** Gathering specific samples or features from datasets.
- **Image Processing:** Applying masks to select or modify pixel regions.
- **Scientific Computing:** Extracting scattered measurement points from simulation grids.

### Core Concepts / Features

The following core concepts are explored in detail: (1) Integer-array indexing, (2) Boolean indexing, (3) Fancy indexing, (4) Combining indexing techniques, (5) Conditional selection, and (6) Copy guarantee.

---

## Core Concept 1: Integer-Array Indexing

### Definitions

**Core Definition:** Integer-array indexing uses lists or arrays of explicit integer index numbers to select arbitrary elements, rows, or columns from an array.

**Technical Definition:** When an integer `ndarray` or list is used as an index, NumPy performs vectorized indexing. For a 1-D array `x` and an index array `idx`, `x[idx]` returns an array where each element is `x[idx[i]]`. For multidimensional arrays, integer arrays can be used along one or more axes, and the result shape is determined by broadcasting the index arrays.

**Beginner-Friendly Explanation:** Imagine you have a list of items and you want to pick out items at positions 2, 5, and 7—but not in order, maybe 7, 2, and 5. Integer-array indexing lets you do exactly that. You give NumPy a list of the positions you want, and it hands you back those items in the order you specified.

### Purposes

- To select non-contiguous rows or columns from an array.
- To reorder or shuffle elements along an axis.
- To duplicate elements by including the same index multiple times.
- To select arbitrary elements from a multidimensional array using coordinate arrays.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# 1-D array
result = array[[i1, i2, i3, ...]]

# N-D array with one index array
result = array[[i1, i2, ...], :]

# N-D array with multiple index arrays
result = array[[i1, i2, ...], [j1, j2, ...]]
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `array` | Any NumPy `ndarray`. |
| `[i1, i2, ...]` | A list or integer array of indices. |
| `:` | Selects all elements along that axis. |
| Multiple index arrays | Broadcast against each other; result shape is the broadcast shape. |

**Syntax Rules:**

- The index array can be a Python list or a NumPy integer array.
- When a single index array is used, it selects along the first axis.
- When multiple index arrays are used, they must be broadcastable to a common shape.
- Negative indices are allowed in integer arrays.

**Constraints and Limitations:**

- Integer-array indexing always returns a copy, not a view.
- Index arrays must be integers; floats raise `IndexError`.
- Out-of-bounds indices raise `IndexError`.
- The result shape is determined by the broadcast shape of the index arrays, not the original array shape.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Integer-Array Indexing in 1-D and 2-D Arrays**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create a 1-D array.
arr = np.array([10, 20, 30, 40, 50])
print("Original array:", arr)
print()

# Step 3: Select elements at indices 0, 2, and 4.
selected = arr[[0, 2, 4]]
print("arr[[0, 2, 4]] ->", selected)
print()

# Step 4: Select elements in arbitrary order (including duplicates).
arbitrary = arr[[4, 0, 2, 0]]
print("arr[[4, 0, 2, 0]] ->", arbitrary)
print("  (element at index 0 appears twice)")
print()

# Step 5: Create a 2-D array and select specific rows.
matrix = np.array([[1,  2,  3],
                   [4,  5,  6],
                   [7,  8,  9],
                   [10, 11, 12]])
print("Matrix:\n", matrix)
rows = matrix[[0, 2, 3]]
print("matrix[[0, 2, 3]] ->\n", rows)
print("Shape:", rows.shape)
```

**Expected Output:**

```
Original array: [10 20 30 40 50]

arr[[0, 2, 4]] -> [10 30 50]

arr[[4, 0, 2, 0]] -> [50 10 30 10]
  (element at index 0 appears twice)

Matrix:
 [[ 1  2  3]
 [ 4  5  6]
 [ 7  8  9]
 [10 11 12]]
matrix[[0, 2, 3]] ->
 [[ 1  2  3]
 [ 7  8  9]
 [10 11 12]]
Shape: (3, 3)
```

**Why This Result Occurs:** The index array `[0, 2, 4]` tells NumPy to select elements at positions 0, 2, and 4 from the original array, producing `[10, 30, 50]`. The index array `[4, 0, 2, 0]` selects elements in the specified order, including index 0 twice, producing `[50, 10, 30, 10]`. For the 2-D array, `matrix[[0, 2, 3]]` selects rows 0, 2, and 3, returning a 3×3 array.

**Example 2: Multiple Index Arrays for Coordinate Pairs**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create a 3x3 matrix.
x = np.array([[1, 2, 3],
              [4, 5, 6],
              [7, 8, 9]])
print("Matrix:\n", x)
print()

# Step 3: Select specific elements using row and column index arrays.
# This selects (0,0), (1,1), and (2,2) -> the main diagonal.
diagonal = x[[0, 1, 2], [0, 1, 2]]
print("x[[0,1,2], [0,1,2]] ->", diagonal)
print("  (the main diagonal: 1, 5, 9)")
print()

# Step 4: Select different elements: (0,2), (1,0), (2,1).
specific = x[[0, 1, 2], [2, 0, 1]]
print("x[[0,1,2], [2,0,1]] ->", specific)
print("  (elements at (0,2), (1,0), (2,1): 3, 4, 8)")
```

**Expected Output:**

```
Matrix:
 [[1 2 3]
 [4 5 6]
 [7 8 9]]

x[[0,1,2], [0,1,2]] -> [1 5 9]
  (the main diagonal: 1, 5, 9)

x[[0,1,2], [2,0,1]] -> [3 4 8]
  (elements at (0,2), (1,0), (2,1): 3, 4, 8)
```

**Why This Result Occurs:** When multiple index arrays are provided, NumPy pairs them element-wise: the first element of the row index array is paired with the first element of the column index array, and so on. So `x[[0,1,2], [0,1,2]]` selects `x[0,0]`, `x[1,1]`, and `x[2,2]`, producing the diagonal. `x[[0,1,2], [2,0,1]]` selects `x[0,2]`, `x[1,0]`, and `x[2,1]`.

### Real-World Cases

- **Data Reordering:** Shuffle the rows of a dataset before training a model: `X_shuffled = X[permutation]`.
- **Cross-Validation:** Select specific fold indices: `X_train = X[train_indices]`.
- **Image Processing:** Select specific pixels by coordinate arrays for custom effects.
- **Neural Networks:** Gather specific elements from a tensor for embedding lookups.

### References

- NumPy Indexing (Advanced Indexing) – https://numpy.org/doc/1.18/reference/arrays.indexing.html#advanced-indexing
- NumPy How to index ndarrays – https://numpy.org/doc/2.5/user/how-to-index.html
- NumPy Copies and Views – https://numpy.org/doc/1.22/user/basics.copies.html

---

## Core Concept 2: Boolean Indexing

### Definitions

**Core Definition:** Boolean indexing uses a boolean array (or a boolean expression) of the same shape as the target array to select elements where the mask evaluates to `True`.

**Technical Definition:** Boolean array indexing occurs when the index object is an array of Boolean type, such as may be returned from comparison operators. The result is a 1-D array containing the elements of the original array corresponding to `True` values in the mask. This is a form of advanced indexing and always returns a copy.

**Beginner-Friendly Explanation:** Boolean indexing is like putting a stencil over your data. The stencil has holes where you want to keep data and solid areas where you don't. You create the stencil by asking a question—like "is this number greater than 5?"—and NumPy gives you back only the numbers where the answer is "yes."

### Purposes

- To filter array elements based on a condition.
- To select elements that satisfy one or more logical criteria.
- To modify elements in place using a boolean mask on the left-hand side of an assignment.
- To count or summarize elements meeting a condition.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Basic boolean mask
mask = array > threshold
filtered = array[mask]

# Direct expression
filtered = array[array > threshold]

# In-place modification
array[array > threshold] = new_value
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `mask` | A boolean array of the same shape as `array`. |
| `array[mask]` | Returns a 1-D array of elements where `mask` is `True`. |
| Comparison operators | `<`, `>`, `<=`, `>=`, `==`, `!=` produce boolean arrays. |
| Logical operators | `&` (and), `|` (or), `~` (not) combine boolean arrays. |

**Syntax Rules:**

- The boolean mask must be broadcastable to the shape of the array.
- The result of boolean indexing is always a 1-D array, even if the original array is multidimensional.
- Use `&`, `|`, `~` instead of `and`, `or`, `not` for array comparisons.
- Parentheses are required around comparisons when combining with logical operators due to precedence rules.

**Constraints and Limitations:**

- Boolean indexing always returns a copy, not a view.
- The mask must have exactly the same shape as the array being indexed (or be broadcastable to it).
- Boolean indexing with assignment (e.g., `arr[mask] = value`) modifies the original array in place, which is a special exception to the copy rule.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Basic Boolean Filtering**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create an array of values.
arr = np.array([1, 5, 3, 8, 2, 7, 4, 9, 6])
print("Original array:", arr)
print()

# Step 3: Create a boolean mask for values greater than 5.
mask = arr > 5
print("Mask (arr > 5):", mask)
print()

# Step 4: Apply the mask to filter the array.
filtered = arr[mask]
print("Filtered (arr[mask]):", filtered)
print()

# Step 5: Use the mask directly in one expression.
direct = arr[arr > 5]
print("Direct (arr[arr > 5]):", direct)
```

**Expected Output:**

```
Original array: [1 5 3 8 2 7 4 9 6]

Mask (arr > 5): [False False False  True False  True False  True False]

Filtered (arr[mask]): [8 7 9]

Direct (arr[arr > 5]): [8 7 9]
```

**Why This Result Occurs:** The comparison `arr > 5` produces a boolean array where each element is `True` if the corresponding element in `arr` is greater than 5. Indexing `arr` with this mask returns only the elements where the mask is `True`: 8, 7, and 9. Both `arr[mask]` and `arr[arr > 5]` produce the same result.

**Example 2: Boolean Indexing with 2-D Arrays**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create a 3x3 matrix.
matrix = np.array([[1, 2, 3],
                   [4, 5, 6],
                   [7, 8, 9]])
print("Matrix:\n", matrix)
print()

# Step 3: Create a boolean mask for values greater than 5.
mask = matrix > 5
print("Mask (matrix > 5):\n", mask)
print()

# Step 4: Apply the mask.
filtered = matrix[mask]
print("Filtered (matrix[mask]):", filtered)
print("Shape:", filtered.shape)
print("  (Note: result is 1-D, not 2-D)")
print()

# Step 5: Use boolean indexing to modify values in place.
matrix[matrix > 5] = 0
print("After matrix[matrix > 5] = 0:\n", matrix)
```

**Expected Output:**

```
Matrix:
 [[1 2 3]
 [4 5 6]
 [7 8 9]]

Mask (matrix > 5):
 [[False False False]
 [False False  True]
 [ True  True  True]]

Filtered (matrix[mask]): [6 7 8 9]
Shape: (4,)
  (Note: result is 1-D, not 2-D)

After matrix[matrix > 5] = 0:
 [[1 2 3]
 [4 5 0]
 [0 0 0]]
```

**Why This Result Occurs:** The mask `matrix > 5` is a 3×3 boolean array. Indexing `matrix[mask]` returns a 1-D array containing all elements where the mask is `True`, regardless of their original positions. The assignment `matrix[matrix > 5] = 0` uses the same mask to select elements in the original array and replaces them with 0 in place.

### Real-World Cases

- **Data Cleaning:** Remove outliers: `data = data[data < upper_bound]`.
- **Image Segmentation:** Select pixels above a brightness threshold: `bright_pixels = image[image > 200]`.
- **Financial Analysis:** Filter transactions above a certain amount: `large_transactions = transactions[transactions > 1000]`.
- **Scientific Data:** Select valid measurements: `valid = data[data != -999]`.

### References

- NumPy Indexing (Boolean array indexing) – https://numpy.org/doc/1.18/reference/arrays.indexing.html#boolean-array-indexing
- NumPy Copies and Views – https://numpy.org/doc/1.22/user/basics.copies.html
- NumPy Boolean Indexing (Discussion) – https://mail.python.org/pipermail/numpy-discussion/2011-January/054087.html

---

## Core Concept 3: Fancy Indexing

### Definitions

**Core Definition:** Fancy indexing is the general term for using integer arrays or lists to index into an array, allowing arbitrary, non-contiguous selection of elements along one or more axes.

**Technical Definition:** Fancy indexing occurs when the index is a non-tuple sequence object or an `ndarray` of integer type. When multiple index arrays are provided, they are broadcast against each other, and the result has the shape of the broadcast index arrays concatenated with the shape of any unindexed dimensions. Fancy indexing always returns a copy.

**Beginner-Friendly Explanation:** Fancy indexing is like giving NumPy a shopping list of exactly which elements you want. You can say "give me the element at row 2, column 5, then row 0, column 3, then row 7, column 1." NumPy gathers them all up and returns them in a new array.

### Purposes

- To select specific coordinate pairs from a multidimensional array.
- To gather elements from arbitrary positions in one operation.
- To construct new arrays from scattered elements of existing arrays.
- To perform vectorized lookups and gathers.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Single index array
result = array[[i1, i2, i3, ...]]

# Multiple index arrays (coordinate pairs)
result = array[[i1, i2, ...], [j1, j2, ...]]

# Mixing with slices
result = array[[i1, i2, ...], :, [k1, k2, ...]]
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `[i1, i2, ...]` | Integer index array for axis 0. |
| `[j1, j2, ...]` | Integer index array for axis 1. |
| Broadcast shape | Result shape is the broadcast shape of all index arrays. |
| Unindexed axes | Included in the result after the broadcast dimensions. |

**Syntax Rules:**

- All index arrays must be broadcastable to a common shape.
- The result shape is the broadcast shape of the index arrays followed by the shape of any unindexed axes.
- Fancy indexing always returns a copy.

**Constraints and Limitations:**

- Index arrays must have matching shapes (or be broadcastable).
- Using more index arrays than dimensions raises an error.
- The result is always a copy, never a view.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Fancy Indexing with Multiple Coordinate Arrays**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create a 4x5 matrix.
x = np.arange(20).reshape(4, 5)
print("Matrix (4x5):\n", x)
print()

# Step 3: Select elements at specific (row, column) coordinates.
row_indices = [0, 1, 2, 3]
col_indices = [0, 1, 2, 3]
diagonal = x[row_indices, col_indices]
print("Diagonal (x[[0,1,2,3], [0,1,2,3]]):", diagonal)
print("  (elements at (0,0), (1,1), (2,2), (3,3): 0, 6, 12, 18)")
print()

# Step 4: Select arbitrary coordinate pairs.
rows = [0, 3, 1]
cols = [4, 0, 2]
specific = x[rows, cols]
print("Specific elements (x[[0,3,1], [4,0,2]]):", specific)
print("  (elements at (0,4), (3,0), (1,2): 4, 15, 7)")
print()

# Step 5: Use broadcasting with a scalar.
partial = x[[0, 1, 2], 1]
print("x[[0,1,2], 1] ->", partial)
print("  (elements at (0,1), (1,1), (2,1): 1, 6, 11)")
```

**Expected Output:**

```
Matrix (4x5):
 [[ 0  1  2  3  4]
 [ 5  6  7  8  9]
 [10 11 12 13 14]
 [15 16 17 18 19]]

Diagonal (x[[0,1,2,3], [0,1,2,3]]): [ 0  6 12 18]
  (elements at (0,0), (1,1), (2,2), (3,3): 0, 6, 12, 18)

Specific elements (x[[0,3,1], [4,0,2]]): [ 4 15  7]
  (elements at (0,4), (3,0), (1,2): 4, 15, 7)

x[[0,1,2], 1] -> [ 1  6 11]
  (elements at (0,1), (1,1), (2,1): 1, 6, 11)
```

**Why This Result Occurs:** When multiple index arrays are provided, NumPy pairs them element-wise: `(0,0)`, `(1,1)`, `(2,2)`, `(3,3)` for the diagonal. The arbitrary selection pairs `(0,4)`, `(3,0)`, `(1,2)`. When a scalar is used for one axis (`1`), it is broadcast against the index array for the other axis, selecting `(0,1)`, `(1,1)`, `(2,1)`.

### Real-World Cases

- **Batch Gathering:** Selecting specific samples from a batch based on indices: `X[batch_indices]`.
- **Embedding Lookups:** Looking up word embeddings by token indices: `embeddings[token_ids]`.
- **Image Sampling:** Extracting pixel values at specific coordinates: `image[ys, xs]`.
- **Recommendation Systems:** Gathering user-item interaction scores at specific coordinates.

### References

- NumPy Indexing (Advanced Indexing) – https://numpy.org/doc/1.18/reference/arrays.indexing.html#advanced-indexing
- NumPy How to index ndarrays – https://numpy.org/doc/2.5/user/how-to-index.html
- NEP 21: Simplified and Explicit Advanced Indexing – https://numpy.org/neps/nep-0021-advanced-indexing.html

---

## Core Concept 4: Combining Indexing Techniques

### Definitions

**Core Definition:** Combining indexing techniques means mixing basic slices (`:`, integers) with advanced index arrays (integer or boolean) within a single bracket operation.

**Technical Definition:** Integer- and boolean-valued arrays can be used in conjunction with `slice`, `numpy.newaxis`, and `int` objects to form indices that combine basic and advanced indexing schemes. The two cases to distinguish are: (1) advanced indices separated by a slice, Ellipsis, or newaxis (e.g., `x[arr1, :, arr2]`), and (2) advanced indices all next to each other (e.g., `x[..., arr1, arr2, :]`).

**Beginner-Friendly Explanation:** You don't have to choose between basic slicing and advanced indexing—you can use both in the same operation. For example, you can say "take rows 1 through 3 (a slice) and columns 0, 2, and 4 (an index array)." NumPy figures out how to combine them into one result.

### Purposes

- To select a subset of rows using a slice and a subset of columns using an index array.
- To combine the efficiency of slicing with the flexibility of advanced indexing.
- To perform complex selections in a single operation without intermediate steps.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Advanced index separated from basic index
result = array[advanced_index, :, other_advanced_index]

# Advanced indices adjacent
result = array[..., advanced_index1, advanced_index2, :]
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `advanced_index` | An integer or boolean array. |
| `:` | A basic slice selecting all elements along that axis. |
| `...` | Ellipsis, selecting all remaining axes. |

**Syntax Rules:**

- When advanced indices are separated by a slice, the dimensions from the advanced indexing operation come first in the result array, and the subspace dimensions come after.
- When advanced indices are all next to each other, the result dimensions are placed in the same position as the advanced indices.
- Mixing slices and integer arrays is allowed and common.

**Constraints and Limitations:**

- The behavior of combined indexing can be confusing; always verify the result shape.
- Advanced indices are broadcast against each other, which can lead to unexpected shapes.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Combining Slices and Integer Arrays**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create a 4x5 matrix.
x = np.arange(20).reshape(4, 5)
print("Matrix (4x5):\n", x)
print()

# Step 3: Select rows 1:3 (slice) and columns [1, 3] (integer array).
combined = x[1:3, [1, 3]]
print("x[1:3, [1, 3]] ->\n", combined)
print("Shape:", combined.shape)
print("  (rows 1 and 2, columns 1 and 3)")
print()

# Step 4: Select all rows and specific columns.
all_rows = x[:, [0, 2, 4]]
print("x[:, [0, 2, 4]] ->\n", all_rows)
print("Shape:", all_rows.shape)
print()

# Step 5: Combine advanced indices with a slice in between.
z = np.arange(24).reshape(2, 3, 4)
print("3-D array shape:", z.shape)
result = z[[0, 1], :, [1, 3]]
print("z[[0, 1], :, [1, 3]] ->\n", result)
print("Shape:", result.shape)
```

**Expected Output:**

```
Matrix (4x5):
 [[ 0  1  2  3  4]
 [ 5  6  7  8  9]
 [10 11 12 13 14]
 [15 16 17 18 19]]

x[1:3, [1, 3]] ->
 [[ 6  8]
 [11 13]]
Shape: (2, 2)
  (rows 1 and 2, columns 1 and 3)

x[:, [0, 2, 4]] ->
 [[ 0  2  4]
 [ 5  7  9]
 [10 12 14]
 [15 17 19]]
Shape: (4, 3)

3-D array shape: (2, 3, 4)
z[[0, 1], :, [1, 3]] ->
 [[[ 1  2  3]
  [ 5  6  7]
  [ 9 10 11]]

 [[13 14 15]
  [17 18 19]
  [21 22 23]]]
Shape: (2, 3, 3)
```

**Why This Result Occurs:** `x[1:3, [1, 3]]` combines a slice (rows 1 and 2) with an integer array (columns 1 and 3), producing a 2×2 result. `x[:, [0, 2, 4]]` selects all rows and columns 0, 2, and 4. For the 3-D array, `z[[0, 1], :, [1, 3]]` has advanced indices separated by a slice; the advanced dimensions come first in the result, producing a shape of `(2, 3, 3)`.

### Real-World Cases

- **Dataset Slicing:** Select specific rows (a range) and specific columns (features of interest): `X[100:200, [0, 5, 10]]`.
- **Image Processing:** Crop a region and select specific channels: `image[50:150, 50:150, [0, 2]]`.
- **Time Series:** Select a time window (slice) and specific sensors (integer array): `data[1000:2000, [2, 5, 7]]`.

### References

- NumPy Combining Advanced and Basic Indexing – https://numpy.org/doc/stable/user/basics.indexing.html#combining-advanced-and-basic-indexing
- NumPy How to index ndarrays – https://numpy.org/doc/2.5/user/how-to-index.html
- GitHub Issue: DOC: Confusing advanced indexing when mixing tuple, integer, and range – https://github.com/numpy/numpy/issues/30022

---

## Core Concept 5: Conditional Selection

### Definitions

**Core Definition:** Conditional selection uses logical operators (`&`, `|`, `~`) combined with comparison operations to create boolean masks that select elements satisfying complex criteria.

**Technical Definition:** NumPy comparison operators (`<`, `>`, `<=`, `>=`, `==`, `!=`) produce boolean arrays. These boolean arrays can be combined using bitwise logical operators `&` (and), `|` (or), and `~` (not) to create compound conditions. The resulting mask is then used for boolean indexing.

**Beginner-Friendly Explanation:** Conditional selection is like asking multiple questions about your data at once. For example, "give me all the numbers that are greater than 5 AND less than 10" or "give me all the values that are either negative OR greater than 100." You combine the questions with `&`, `|`, and `~` to get exactly what you want.

### Purposes

- To filter data based on multiple conditions simultaneously.
- To select elements within a range or outside a range.
- To exclude specific values or categories.
- To implement complex data cleaning and subsetting logic.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
mask = (condition1) & (condition2) | ~(condition3)
result = array[mask]
```

**Component Breakdown:**

| Operator | Meaning | Example |
|----------|---------|---------|
| `&` | Logical AND | `(arr > 5) & (arr < 10)` |
| `\|` | Logical OR | `(arr < 0) \| (arr > 100)` |
| `~` | Logical NOT | `~(arr == 0)` |
| `==` | Equality | `arr == 5` |
| `!=` | Inequality | `arr != 0` |

**Syntax Rules:**

- Use `&`, `|`, `~` instead of `and`, `or`, `not`.
- Parentheses are required around each comparison when combining with logical operators.
- The resulting mask is a boolean array that can be used for indexing.

**Constraints and Limitations:**

- Operator precedence: `&` has higher precedence than `|`, so parentheses are essential.
- Using `and`/`or`/`not` on arrays raises `ValueError` because their truth value is ambiguous.
- The mask must be broadcastable to the array's shape.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Combining Conditions with Logical Operators**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create an array of values.
arr = np.array([1, 5, 3, 8, 2, 7, 4, 9, 6, 0])
print("Original array:", arr)
print()

# Step 3: Select values greater than 3 AND less than 8.
mask_and = (arr > 3) & (arr < 8)
print("Mask ((arr > 3) & (arr < 8)):", mask_and)
print("Result:", arr[mask_and])
print()

# Step 4: Select values less than 3 OR greater than 7.
mask_or = (arr < 3) | (arr > 7)
print("Mask ((arr < 3) | (arr > 7)):", mask_or)
print("Result:", arr[mask_or])
print()

# Step 5: Select values that are NOT equal to 0.
mask_not = ~(arr == 0)
print("Mask (~(arr == 0)):", mask_not)
print("Result:", arr[mask_not])
```

**Expected Output:**

```
Original array: [1 5 3 8 2 7 4 9 6 0]

Mask ((arr > 3) & (arr < 8)): [False  True False False False  True  True False  True False]
Result: [5 7 4 6]

Mask ((arr < 3) | (arr > 7)): [ True False False  True  True False False  True False  True]
Result: [1 8 2 9 0]

Mask (~(arr == 0)): [ True  True  True  True  True  True  True  True  True False]
Result: [1 5 3 8 2 7 4 9 6]
```

**Why This Result Occurs:** The `&` operator combines two boolean masks, keeping only elements where both conditions are `True`. The `|` operator keeps elements where at least one condition is `True`. The `~` operator inverts the mask. Each result is a 1-D array of the elements that satisfy the compound condition.

**Example 2: Conditional Selection on 2-D Arrays**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create a 3x3 matrix.
matrix = np.array([[1, 2, 3],
                   [4, 5, 6],
                   [7, 8, 9]])
print("Matrix:\n", matrix)
print()

# Step 3: Select elements that are even AND greater than 4.
mask = (matrix % 2 == 0) & (matrix > 4)
print("Mask ((matrix % 2 == 0) & (matrix > 4)):\n", mask)
print("Result:", matrix[mask])
print()

# Step 4: Use conditional selection to set values.
matrix[(matrix > 3) & (matrix < 7)] = 0
print("After setting values between 3 and 7 to 0:\n", matrix)
```

**Expected Output:**

```
Matrix:
 [[1 2 3]
 [4 5 6]
 [7 8 9]]

Mask ((matrix % 2 == 0) & (matrix > 4)):
 [[False False False]
 [False False  True]
 [False  True False]]
Result: [6 8]

After setting values between 3 and 7 to 0:
 [[1 2 3]
 [0 0 0]
 [7 8 9]]
```

**Why This Result Occurs:** The mask identifies elements that are both even and greater than 4: 6 and 8. The assignment `matrix[(matrix > 3) & (matrix < 7)] = 0` sets all elements between 3 and 7 (exclusive) to 0, modifying the original array in place.

### Real-World Cases

- **Data Cleaning:** Remove outliers: `data = data[(data > lower) & (data < upper)]`.
- **Filtering Transactions:** Select transactions in a specific range: `valid = transactions[(transactions > 100) & (transactions < 5000)]`.
- **Image Processing:** Select pixels in a specific color range: `mask = (r > 100) & (g < 50) & (b < 50)`.
- **Scientific Data:** Select measurements within acceptable bounds: `good = data[~((data < min_val) | (data > max_val))]`.

### References

- NumPy Logical Operations – https://numpy.org/doc/stable/reference/routines.logic.html
- NumPy Boolean Indexing (Discussion) – https://mail.python.org/pipermail/numpy-discussion/2010-September/052990.html
- NumPy Bitwise Operations – https://numpy.org/doc/stable/reference/routines.bitwise.html

---

## Core Concept 6: Copy Guarantee

### Definitions

**Core Definition:** Advanced indexing always returns a copy of the data, meaning the resulting array has its own independent data buffer and modifications to it do not affect the original array.

**Technical Definition:** Advanced indexing cannot return a view because the selected elements may be non-contiguous in memory and there is no regular stride pattern that can describe them. Therefore, NumPy allocates a new array and copies the selected data into it. This is in contrast to basic slicing, which always returns a view because it can be described by offsets and strides.

**Beginner-Friendly Explanation:** When you use advanced indexing to pick out scattered elements, NumPy can't just point to the original memory—the elements aren't next to each other. So it makes a brand-new array with copies of those elements. This means if you change something in the new array, the original stays the same. With basic slicing, on the other hand, you get a "window" into the original data, so changes show up in both places.

### Purposes

- To ensure data integrity when working with subsets of arrays.
- To allow independent modification of extracted data without affecting the source.
- To avoid the overhead of manually copying data after selection.
- To provide predictable behavior when the selected elements are non-contiguous.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Advanced indexing (returns a copy)
copy_array = array[[0, 2, 4]]
copy_array = array[array > 5]
copy_array = array[[0, 1], [0, 1]]

# Basic slicing (returns a view)
view_array = array[1:5]
view_array = array[:, 2]
```

**Component Breakdown:**

| Operation | Return Type | `.base` |
|-----------|-------------|---------|
| Basic slicing | View | Original array |
| Integer-array indexing | Copy | `None` |
| Boolean indexing | Copy | `None` |
| Fancy indexing | Copy | `None` |

**Syntax Rules:**

- Advanced indexing (integer arrays, boolean arrays) always returns a copy.
- Basic slicing (slices, integers, ellipsis) always returns a view.
- The `.base` attribute of a copy is `None`; the `.base` of a view is the original array.
- Use `.copy()` explicitly if you want a copy of a basic slice.

**Constraints and Limitations:**

- Copies require additional memory; for very large selections, this can be a performance consideration.
- The copy guarantee is a fundamental design decision of NumPy; it cannot be changed.
- Assignment with advanced indexing on the left-hand side (e.g., `arr[mask] = value`) modifies the original array in place, which is a special exception to the copy rule.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Demonstrating the Copy Guarantee**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create an array.
x = np.arange(10)
print("Original x:", x)
print()

# Step 3: Basic slicing creates a view.
view = x[2:6]
print("Basic slice x[2:6]:", view)
print("view.base is x:", view.base is x)
print()

# Step 4: Modify the view and observe the effect on the original.
view[0] = 99
print("After view[0] = 99:")
print("  view:", view)
print("  x:", x)
print("  (x[2] changed because view shares memory)")
print()

# Step 5: Reset x.
x = np.arange(10)

# Step 6: Advanced indexing creates a copy.
copy_arr = x[[2, 3, 4, 5]]
print("Advanced index x[[2, 3, 4, 5]]:", copy_arr)
print("copy_arr.base is None:", copy_arr.base is None)
print()

# Step 7: Modify the copy and observe no effect on the original.
copy_arr[0] = 99
print("After copy_arr[0] = 99:")
print("  copy_arr:", copy_arr)
print("  x:", x)
print("  (x[2] unchanged because copy_arr is independent)")
```

**Expected Output:**

```
Original x: [0 1 2 3 4 5 6 7 8 9]

Basic slice x[2:6]: [2 3 4 5]
view.base is x: True

After view[0] = 99:
  view: [99  3  4  5]
  x: [ 0  1 99  3  4  5  6  7  8  9]
  (x[2] changed because view shares memory)

Advanced index x[[2, 3, 4, 5]]: [2 3 4 5]
copy_arr.base is None: True

After copy_arr[0] = 99:
  copy_arr: [99  3  4  5]
  x: [0 1 2 3 4 5 6 7 8 9]
  (x[2] unchanged because copy_arr is independent)
```

**Why This Result Occurs:** Basic slicing `x[2:6]` creates a view because the selected elements are contiguous and can be described by offsets and strides. The view's `.base` is `x`, confirming they share memory. Modifying the view modifies `x`. Advanced indexing `x[[2, 3, 4, 5]]` creates a copy because the index array is a Python list (an advanced index). The copy's `.base` is `None`, and modifying it does not affect `x`.

### Real-World Cases

- **Data Independence:** When extracting a subset of data for modification, advanced indexing ensures the original remains unchanged.
- **Memory Management:** Understanding the copy guarantee helps avoid unintended memory sharing and data corruption.
- **Performance Planning:** Knowing that advanced indexing creates a copy helps estimate memory usage for large selections.

### References

- NumPy Copies and Views – https://numpy.org/doc/1.22/user/basics.copies.html
- NumPy Indexing (Advanced Indexing) – https://numpy.org/doc/1.18/reference/arrays.indexing.html#advanced-indexing
- NEP 21: Simplified and Explicit Advanced Indexing – https://numpy.org/neps/nep-0021-advanced-indexing.html
- NumPy Discussion: Advanced Indexing and Copies – https://mail.python.org/pipermail/numpy-discussion/2011-January/054086.html

---

## Summary of Deprecated and Version-Specific Features

| Feature | Status | Recommendation |
|---------|--------|----------------|
| Basic slicing with non-tuple sequences containing slices | Deprecated since NumPy 1.15.0 | Use tuples of slice objects and integers |
| `arr[i][j]` chained indexing | Functional but inefficient | Use `arr[i, j]` |
| Boolean indexing with `and`/`or`/`not` | Raises `ValueError` | Use `&`, `|`, `~` |
| Fancy indexing on structured arrays (field names) | Returning copy deprecated in favor of view | Check NumPy version; use view where available |

---

## Consolidated Reference List

- NumPy Indexing (Advanced Indexing) – https://numpy.org/doc/1.18/reference/arrays.indexing.html#advanced-indexing
- NumPy How to index ndarrays – https://numpy.org/doc/2.5/user/how-to-index.html
- NumPy Copies and Views – https://numpy.org/doc/1.22/user/basics.copies.html
- NumPy Combining Advanced and Basic Indexing – https://numpy.org/doc/stable/user/basics.indexing.html#combining-advanced-and-basic-indexing
- NEP 21: Simplified and Explicit Advanced Indexing – https://numpy.org/neps/nep-0021-advanced-indexing.html
- NumPy Logical Operations – https://numpy.org/doc/stable/reference/routines.logic.html
- NumPy Bitwise Operations – https://numpy.org/doc/stable/reference/routines.bitwise.html
- NumPy Boolean Indexing (Discussion) – https://mail.python.org/pipermail/numpy-discussion/2011-January/054087.html
- NumPy Advanced Indexing and Copies (Discussion) – https://mail.python.org/pipermail/numpy-discussion/2011-January/054086.html
- NumPy Discussion: Restrictions on Fancy Indexing – https://mail.python.org/pipermail/numpy-discussion/2010-September/052990.html
- GitHub Issue: DOC: Confusing advanced indexing when mixing tuple, integer, and range – https://github.com/numpy/numpy/issues/30022