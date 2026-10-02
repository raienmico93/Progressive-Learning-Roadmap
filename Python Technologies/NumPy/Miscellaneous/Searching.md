# NumPy Searching: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** NumPy searching is the set of functions that locate elements within an array based on value comparisons, conditions, or sorted-order properties, returning either the positions (indices) of matching elements or the values themselves.

**Technical Definition:** NumPy implements searching through a collection of routines in the `numpy` namespace, including `argmax`, `argmin`, `where`, `nonzero`, `argwhere`, `searchsorted`, and `extract`. These functions operate on ndarrays, support multidimensional structures, and leverage compiled C loops for vectorized performance. They are grouped under the "Sorting, searching, and counting" category of the NumPy API.

**Beginner-Friendly Explanation:** NumPy searching is how you find things in an array. You can ask "where is the biggest number?", "which elements are greater than 5?", or "where would this value fit if the array were sorted?" NumPy answers these questions quickly, even for arrays with millions of elements.

### Key Characteristics

- **Index vs. Value:** Some functions return indices (`argmax`, `argmin`, `where`, `nonzero`, `argwhere`, `searchsorted`); others return values (`extract`).
- **Vectorized:** All searching functions operate on entire arrays without Python loops.
- **Multidimensional Support:** Functions accept `axis` parameters to search along specific dimensions.
- **Condition-Based:** `where`, `nonzero`, `argwhere`, and `extract` work with boolean conditions.
- **Binary Search:** `searchsorted` uses binary search for O(log n) performance on sorted arrays.
- **NaN Handling:** `nanargmax` and `nanargmin` ignore NaN values.

### Prerequisites

- NumPy installed (`pip install numpy`).
- Understanding of array shapes, axes, and indexing.
- Familiarity with the `import numpy as np` convention.
- Conceptual understanding of boolean masks and comparison operations.

### Related Programming Areas

- **Data Analysis:** Finding outliers, extremes, and matching records.
- **Machine Learning:** Locating predictions, computing accuracy, and selecting samples.
- **Signal Processing:** Peak detection and threshold crossing identification.
- **Database Operations:** Implementing search and filter logic on array-backed data.

### Core Concepts / Features

The following concepts are explored in detail: (1) Location-based search (`argmax`, `argmin`), (2) Conditional coordinate extraction (`where`, `nonzero`, `argwhere`), (3) Finding matching elements & values, (4) Binary search scanning (`searchsorted`), and (5) Condition extraction (`extract`).

---

## Core Concept 1: Location-Based Search (argmax & argmin)

### Definitions

**Core Definition:** `np.argmax()` returns the indices of the maximum values and `np.argmin()` returns the indices of the minimum values along a specified axis.

**Technical Definition:** `numpy.argmax(a, axis=None, out=None, *, keepdims=<no value>)` returns the indices of the maximum values along an axis. If `axis=None`, the index is into the flattened array. Otherwise, it is along the specified axis. `numpy.argmin()` returns the indices of the minimum values. In case of multiple occurrences of the maximum/minimum values, the indices corresponding to the first occurrence are returned. `nanargmax` and `nanargmin` are NaN-safe variants that ignore NaN values.

**Beginner-Friendly Explanation:** Instead of giving you the largest number, `argmax` tells you *where* the largest number is. For example, if you have a list of test scores and want to know which student got the highest score, `argmax` gives you the student's position in the list.

### Purposes

- To find the position of the maximum or minimum value in an array.
- To implement one-hot encoding from model logits (the predicted class is the argmax).
- To locate peaks in signals and extrema in data.
- To identify the best or worst performing item in a dataset.
- To work with NaN-containing data using `nanargmax` and `nanargmin`.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.argmax(a, axis=None, out=None, *, keepdims=<no value>)
np.argmin(a, axis=None, out=None, *, keepdims=<no value>)
np.nanargmax(a, axis=None, out=None, *, keepdims=<no value>)
np.nanargmin(a, axis=None, out=None, *, keepdims=<no value>)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `a` | Input array. |
| `axis` | Axis along which to find the index. `None` (default) flattens the array. |
| `out` | Optional output array for the result. |
| `keepdims` | If `True`, reduced axes are retained as size-1 dimensions. |
| Returns | Index or array of indices. |

**Syntax Rules:**

- If `axis=None`, the index is into the flattened array; otherwise, it is along the specified axis.
- In case of multiple occurrences, the first occurrence is returned.
- `keepdims` is available since NumPy 1.22.0.
- `nanargmax` and `nanargmin` ignore NaN values but raise `ValueError` if all values are NaN.

**Constraints and Limitations:**

- `argmax` and `argmin` do not ignore NaN; use `nanargmax` and `nanargmin` for NaN-safe behavior.
- The `out` parameter must have the correct shape and dtype.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Finding Extremes in 1-D and 2-D Arrays**

```python
import numpy as np

# Step 2: Create a 1-D array.
a = np.array([10, 3, 25, 7, 25])
print("Array:", a)

# Step 3: Find global argmin and argmax.
print("np.argmin(a):", np.argmin(a))
print("np.argmax(a):", np.argmax(a))
print("  (argmax returns 2, the first occurrence of 25)")

# Step 4: Create a 2-D array and reduce along axes.
m = np.array([[4, 2, 3], [1, 0, 3]])
print("\nMatrix:\n", m)
print("np.argmax(m, axis=0):", np.argmax(m, axis=0))
print("np.argmax(m, axis=1):", np.argmax(m, axis=1))

# Step 5: keepdims for broadcasting.
print("np.argmax(m, axis=1, keepdims=True):\n", np.argmax(m, axis=1, keepdims=True))
```

**Expected Output:**

```
Array: [10  3 25  7 25]
np.argmin(a): 1
np.argmax(a): 2
  (argmax returns 2, the first occurrence of 25)

Matrix:
 [[4 2 3]
 [1 0 3]]
np.argmax(m, axis=0): [0 0 1]
np.argmax(m, axis=1): [0 2]
np.argmax(m, axis=1, keepdims=True):
 [[0]
 [2]]
```

**Why This Result Occurs:** `np.argmin(a)` returns 1 because the minimum value 3 is at index 1. `np.argmax(a)` returns 2 because the first occurrence of the maximum value 25 is at index 2. Along `axis=0`, the maximum in each column is found; along `axis=1`, the maximum in each row.

### Real-World Cases

- **Classification:** `np.argmax(logits, axis=1)` gives the predicted class for each sample.
- **Peak Detection:** `np.argmax(signal)` finds the index of the maximum signal amplitude.
- **Ranking:** Identifying the best and worst performers in a dataset.
- **Model Selection:** Choosing the model with the highest validation accuracy.

### References

- numpy.argmax – https://numpy.org/doc/stable/reference/generated/numpy.argmax.html
- numpy.argmin – https://numpy.org/doc/stable/reference/generated/numpy.argmin.html
- Sorting, searching, and counting – https://numpy.org/doc/1.22/reference/routines.sort.html

---

## Core Concept 2: Conditional Coordinate Extraction (where, nonzero, argwhere)

### Definitions

**Core Definition:** `np.where()` returns elements chosen from two arrays based on a condition, or the indices where a condition is true. `np.nonzero()` returns the indices of non-zero elements, and `np.argwhere()` returns the indices of non-zero elements grouped by element.

**Technical Definition:** `numpy.where(condition, [x, y])` returns elements from `x` or `y` depending on `condition`. With a single argument, it returns `condition.nonzero()`. `numpy.nonzero(a)` returns a tuple of arrays, one for each dimension, containing the indices of non-zero elements. `numpy.argwhere(a)` returns a 2-D array where each row is the index of a non-zero element; it is equivalent to `np.transpose(np.nonzero(a))` but produces a result of the correct shape for 0-D arrays. The output of `argwhere` is not suitable for indexing arrays; use `nonzero` for that purpose.

**Beginner-Friendly Explanation:** These functions help you find *where* something is in an array. `where` is like a vectorized if-else that can also tell you the positions of matching elements. `nonzero` gives you the coordinates of all non-zero (or `True`) elements. `argwhere` does the same but organizes the results as a list of coordinates, which is easier to read.

### Purposes

- To find the indices of elements that meet a condition.
- To replace elements conditionally using `where`.
- To extract coordinates of non-zero or `True` elements.
- To track positions of matching elements in multidimensional arrays.
- To implement masked operations and conditional selection.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.where(condition, [x, y])
np.nonzero(a)
np.argwhere(a)
np.flatnonzero(a)
```

**Component Breakdown:**

| Function | Description |
|----------|-------------|
| `np.where(cond, x, y)` | Returns elements from `x` or `y` based on `cond`. |
| `np.where(cond)` | Returns indices where `cond` is `True` (equivalent to `cond.nonzero()`). |
| `np.nonzero(a)` | Returns a tuple of index arrays for each dimension. |
| `np.argwhere(a)` | Returns an (N, ndim) array of indices grouped by element. |
| `np.flatnonzero(a)` | Returns indices of non-zero elements in the flattened array. |

**Syntax Rules:**

- `where` with three arguments requires `condition`, `x`, and `y` to be broadcastable.
- `nonzero` returns a tuple of arrays, one per dimension.
- `argwhere` returns a 2-D array with shape `(N, a.ndim)` where N is the number of non-zero elements.
- The output of `argwhere` is not suitable for indexing; use `nonzero` for indexing.

**Constraints and Limitations:**

- `argwhere` produces a copy, not a view.
- For 0-D arrays, `argwhere` produces a result of the correct shape, unlike `transpose(nonzero(a))`.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Using where, nonzero, and argwhere**

```python
import numpy as np

# Step 2: Create an array.
a = np.array([1, 0, 3, 0, 5])
print("Array:", a)

# Step 3: Find indices where a > 1.
indices = np.where(a > 1)
print("np.where(a > 1):", indices)
print("Values:", a[indices])

# Step 4: Replace elements conditionally.
replaced = np.where(a > 1, a, -1)
print("np.where(a > 1, a, -1):", replaced)

# Step 5: nonzero and argwhere on a 2-D array.
m = np.array([[0, 1, 2], [3, 0, 5]])
print("\nMatrix:\n", m)
print("np.nonzero(m):", np.nonzero(m))
print("np.argwhere(m > 1):\n", np.argwhere(m > 1))
```

**Expected Output:**

```
Array: [1 0 3 0 5]
np.where(a > 1): (array([2, 4]),)
Values: [3 5]
np.where(a > 1, a, -1): [-1 -1  3 -1  5]

Matrix:
 [[0 1 2]
 [3 0 5]]
np.nonzero(m): (array([0, 0, 1, 1]), array([1, 2, 0, 2]))
np.argwhere(m > 1):
 [[0 2]
 [1 0]
 [1 2]]
```

**Why This Result Occurs:** `np.where(a > 1)` returns a tuple containing the indices where `a > 1` (positions 2 and 4). `np.where(a > 1, a, -1)` replaces values not greater than 1 with -1. `np.nonzero(m)` returns row and column indices of non-zero elements. `np.argwhere(m > 1)` returns the coordinates of elements greater than 1 as rows in a 2-D array.

### Real-World Cases

- **Data Cleaning:** `np.where(np.isnan(data), 0, data)` replaces NaN values with 0.
- **Conditional Assignment:** `np.where(score > 60, 'pass', 'fail')` assigns labels.
- **Image Processing:** `np.argwhere(image > threshold)` finds bright pixel coordinates.
- **Sparse Data:** `np.nonzero(sparse_matrix)` locates non-zero entries.

### References

- numpy.where – https://numpy.org/doc/stable/reference/generated/numpy.where.html
- numpy.nonzero – https://numpy.org/doc/stable/reference/generated/numpy.nonzero.html
- numpy.argwhere – https://numpy.org/doc/2.0/reference/generated/numpy.argwhere.html
- numpy.flatnonzero – https://numpy.org/doc/stable/reference/generated/numpy.flatnonzero.html

---

## Core Concept 3: Finding Matching Elements & Values

### Definitions

**Core Definition:** Finding matching elements involves locating array positions whose values satisfy a comparison or equality condition, using functions like `np.where`, `np.nonzero`, or boolean indexing.

**Technical Definition:** Equality comparisons (`==`) produce boolean masks, which can be used directly for indexing or passed to `np.where`/`np.nonzero` to obtain indices. For structured arrays, comparisons can be performed on field values. `np.isin` provides an efficient way to test whether elements of an array are present in a second array.

**Beginner-Friendly Explanation:** Finding matching elements means asking "which items equal this value?" or "which items satisfy this condition?" You create a boolean mask by comparing the array to a value, then use that mask to extract either the indices or the values themselves.

### Purposes

- To locate all positions of a specific value in an array.
- To extract matching values from a dataset.
- To implement lookup and join operations.
- To filter data based on equality or membership.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
mask = array == value
indices = np.where(mask)
values = array[mask]
matches = np.isin(array, test_values)
```

**Component Breakdown:**

| Technique | Description |
|-----------|-------------|
| `array == value` | Boolean mask of equality matches. |
| `np.where(mask)` | Indices where mask is `True`. |
| `array[mask]` | Values where mask is `True`. |
| `np.isin(array, values)` | Mask indicating membership in a set of values. |

**Syntax Rules:**

- The comparison mask has the same shape as the array.
- Boolean indexing returns a 1-D array of matching values.
- `np.isin` can be used with `invert=True` to test for non-membership.

**Constraints and Limitations:**

- Floating-point equality is unreliable; use `np.isclose` for approximate matching.
- For NaN values, `==` always returns `False`; use `np.isnan` instead.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Finding Matching Values and Indices**

```python
import numpy as np

# Step 2: Create an array.
data = np.array([10, 20, 30, 20, 40, 20])
print("Data:", data)

# Step 3: Find all occurrences of 20.
matches = data == 20
print("data == 20:", matches)
print("Indices:", np.where(matches)[0])
print("Values:", data[matches])

# Step 4: Use np.isin for membership testing.
values_to_find = [20, 40]
membership = np.isin(data, values_to_find)
print("\nnp.isin(data, [20, 40]):", membership)
print("Matching values:", data[membership])

# Step 5: Search in a 2-D array.
m = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])
print("\nMatrix:\n", m)
print("Positions of 5:", np.argwhere(m == 5))
```

**Expected Output:**

```
Data: [10 20 30 20 40 20]
data == 20: [False  True False  True False  True]
Indices: [1 3 5]
Values: [20 20 20]

np.isin(data, [20, 40]): [False  True False  True  True False]
Matching values: [20 20 40]

Matrix:
 [[1 2 3]
 [4 5 6]
 [7 8 9]]
Positions of 5: [[1 1]]
```

**Why This Result Occurs:** The mask `data == 20` identifies positions 1, 3, and 5. `np.where` returns these indices, and boolean indexing extracts the values. `np.isin` checks membership in the set `{20, 40}`. `np.argwhere(m == 5)` returns the coordinate `[1, 1]` where the value 5 is located.

### Real-World Cases

- **Data Validation:** Checking whether a value exists in a reference set.
- **Record Lookup:** Finding all records with a specific ID.
- **Anomaly Detection:** Locating values outside an expected range.
- **Inventory Management:** Finding products with specific SKUs.

### References

- numpy.where – https://numpy.org/doc/stable/reference/generated/numpy.where.html
- numpy.nonzero – https://numpy.org/doc/stable/reference/generated/numpy.nonzero.html
- numpy.isin – https://numpy.org/doc/stable/reference/generated/numpy.isin.html

---

## Core Concept 4: Binary Search Scanning (searchsorted)

### Definitions

**Core Definition:** `np.searchsorted()` performs a binary search on a sorted array to find the indices where elements should be inserted to maintain order.

**Technical Definition:** `numpy.searchsorted(a, v, side='left', sorter=None)` finds the indices into a sorted array `a` such that, if the corresponding elements in `v` were inserted before the indices, the order of `a` would be preserved. With `side='left'`, the returned index `i` satisfies `a[i-1] < v <= a[i]`; with `side='right'`, it satisfies `a[i-1] <= v < a[i]`. Binary search is used, giving O(log n) performance. As of NumPy 1.4.0, `searchsorted` works with real/complex arrays containing NaN values.

**Beginner-Friendly Explanation:** If you have a sorted list and want to know where a new number would fit, `searchsorted` tells you the position. It's like finding the right place to insert a card in a sorted hand of cards. It uses binary search, so it's very fast even for huge arrays.

### Purposes

- To find insertion points in sorted arrays.
- To implement efficient lookup tables and binning.
- To classify values into bins (e.g., histogram binning).
- To perform range queries on sorted data.
- To compute percentiles and quantiles efficiently.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.searchsorted(a, v, side='left', sorter=None)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `a` | 1-D sorted array. If `sorter` is provided, it must be sorted in ascending order according to the sorter. |
| `v` | Values to insert into `a`. |
| `side` | `'left'` (default) or `'right'`. Controls tie-breaking. |
| `sorter` | Optional array of integer indices that sort `a` into ascending order (typically the result of `argsort`). |
| Returns | Array of insertion points with the same shape as `v`. |

**Syntax Rules:**

- `a` must be sorted in ascending order unless `sorter` is provided.
- `side='left'` returns the first suitable index; `side='right'` returns the last.
- If no suitable index exists, returns either 0 or N (where N is the length of `a`).
- The `sorter` parameter allows searching in an argsorted array.

**Constraints and Limitations:**

- `searchsorted` does not verify that `a` is sorted; passing an unsorted array produces undefined results.
- For unsorted arrays, sort first with `np.sort` or use `argsort` with the `sorter` parameter.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Finding Insertion Points**

```python
import numpy as np

# Step 2: Create a sorted array.
sorted_arr = np.array([1, 2, 3, 4, 5])
print("Sorted array:", sorted_arr)

# Step 3: Find insertion point for 3 (left side).
idx_left = np.searchsorted(sorted_arr, 3)
print("np.searchsorted(sorted_arr, 3):", idx_left)
print("  (insert before index 2, where value 3 is)")

# Step 4: Find insertion point with side='right'.
idx_right = np.searchsorted(sorted_arr, 3, side='right')
print("np.searchsorted(sorted_arr, 3, side='right'):", idx_right)
print("  (insert after index 2)")

# Step 5: Multiple values at once.
values = np.array([-10, 10, 2, 3])
indices = np.searchsorted(sorted_arr, values)
print("\nnp.searchsorted(sorted_arr, [-10, 10, 2, 3]):", indices)

# Step 6: Binning example.
bins = np.array([0, 10, 20, 30, 40, 50])
data = np.array([5, 15, 25, 35, 45])
bin_indices = np.searchsorted(bins, data, side='right') - 1
print("\nBinning data:", data)
print("Bin indices:", bin_indices)
```

**Expected Output:**

```
Sorted array: [1 2 3 4 5]
np.searchsorted(sorted_arr, 3): 2
  (insert before index 2, where value 3 is)
np.searchsorted(sorted_arr, 3, side='right'): 3
  (insert after index 2)

np.searchsorted(sorted_arr, [-10, 10, 2, 3]): [0 5 1 2]

Binning data: [ 5 15 25 35 45]
Bin indices: [0 1 2 3 4]
```

**Why This Result Occurs:** `searchsorted([1,2,3,4,5], 3)` returns 2 because 3 should be inserted at index 2 (before the existing 3). With `side='right'`, it returns 3 (after the existing 3). For the multiple values, -10 goes to index 0, 10 goes to index 5 (end), 2 goes to index 1, and 3 goes to index 2. The binning example uses `side='right'` and subtracts 1 to assign each value to its bin.

### Real-World Cases

- **Histogram Binning:** Assigning data points to histogram bins using `searchsorted`.
- **Percentile Calculation:** Finding the index corresponding to a percentile.
- **Range Queries:** Finding all elements between two values in a sorted array.
- **Merge Operations:** Merging two sorted arrays efficiently.

### References

- numpy.searchsorted – https://numpy.org/doc/1.26/reference/generated/numpy.searchsorted.html
- numpy.ndarray.searchsorted – https://numpy.org/doc/stable/reference/generated/numpy.ndarray.searchsorted.html
- Sorting, searching, and counting – https://numpy.org/doc/1.22/reference/routines.sort.html

---

## Core Concept 5: Condition Extraction (extract)

### Definitions

**Core Definition:** `np.extract()` returns the elements of an array that satisfy a given condition, flattening the result into a 1-D array.

**Technical Definition:** `numpy.extract(condition, arr)` returns the elements of `arr` that satisfy `condition`. It is equivalent to `np.compress(ravel(condition), ravel(arr))`. If `condition` is boolean, `np.extract` is equivalent to `arr[condition]`. The function is the opposite of `np.place`, which replaces elements that satisfy a condition.

**Beginner-Friendly Explanation:** `np.extract` pulls out the elements that meet a condition. It's similar to boolean indexing but flattens the result. If you have a matrix and want all elements greater than 5, `np.extract` gives you a 1-D array of those values.

### Purposes

- To extract values that satisfy a condition from a multidimensional array.
- To flatten and filter data in a single operation.
- To implement conditional value extraction.
- To complement `np.place` for conditional replacement.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.extract(condition, arr)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `condition` | Array whose non-zero or `True` entries indicate the elements to extract. |
| `arr` | Input array of the same size as `condition`. |
| Returns | Rank 1 array of values from `arr` where `condition` is `True`. |

**Syntax Rules:**

- `condition` and `arr` must have the same size (they are both raveled before extraction).
- The result is always a 1-D array.
- If `condition` is boolean, `np.extract` is equivalent to `arr[condition]`.

**Constraints and Limitations:**

- `extract` flattens the input, so the spatial structure is lost.
- For multidimensional extraction, use boolean indexing instead.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Extracting Conditionally**

```python
import numpy as np

# Step 2: Create a 2-D array.
arr = np.arange(12).reshape((3, 4))
print("Array:\n", arr)

# Step 3: Create a condition (multiples of 3).
condition = np.mod(arr, 3) == 0
print("\nCondition (multiples of 3):\n", condition)

# Step 4: Extract matching values.
extracted = np.extract(condition, arr)
print("np.extract(condition, arr):", extracted)
print("  (flattened 1-D array)")

# Step 5: Compare with boolean indexing.
print("arr[condition]:", arr[condition])
print("Match:", np.array_equal(extracted, arr[condition]))
```

**Expected Output:**

```
Array:
 [[ 0  1  2  3]
 [ 4  5  6  7]
 [ 8  9 10 11]]

Condition (multiples of 3):
 [[ True False False  True]
 [False False  True False]
 [False  True False False]]
np.extract(condition, arr): [0 3 6 9]
  (flattened 1-D array)
arr[condition]: [0 3 6 9]
Match: True
```

**Why This Result Occurs:** The condition identifies elements divisible by 3. `np.extract` ravels both the condition and the array, then selects matching elements, producing `[0, 3, 6, 9]`. This is equivalent to `arr[condition]`, which also returns the same 1-D array.

### Real-World Cases

- **Data Filtering:** Extracting all valid measurements from a sensor array.
- **Image Processing:** Extracting pixel values above a threshold.
- **Financial Analysis:** Extracting all transactions above a certain amount.
- **Scientific Computing:** Extracting non-zero elements from sparse data.

### References

- numpy.extract – https://numpy.org/doc/2.0/reference/generated/numpy.extract.html
- numpy.place – https://numpy.org/doc/2.0/reference/generated/numpy.place.html
- Sorting, searching, and counting – https://numpy.org/doc/1.22/reference/routines.sort.html

---

## Comparison Summary

| Function | Returns | Input Requirement | Best For |
|----------|---------|-------------------|----------|
| `np.argmax` / `np.argmin` | Indices of extremes | Any array | Finding positions of min/max |
| `np.where` (3-arg) | Values from x or y | Broadcastable arrays | Conditional replacement |
| `np.where` (1-arg) | Indices where True | Boolean condition | Finding matching positions |
| `np.nonzero` | Tuple of index arrays | Any array | Indexing-based selection |
| `np.argwhere` | (N, ndim) index array | Any array | Human-readable coordinates |
| `np.searchsorted` | Insertion indices | Sorted 1-D array | Binary search, binning |
| `np.extract` | 1-D values | Same-size condition | Flattened conditional extraction |

---

## Summary of Deprecated and Version-Specific Features

| Feature | Status | Recommendation |
|---------|--------|----------------|
| `keepdims` in `argmax`/`argmin` | New in NumPy 1.22.0 | Available in NumPy ≥ 1.22 |
| `searchsorted` with NaN | Works since NumPy 1.4.0 | Safe to use with real/complex arrays |
| `argwhere` on 0-D arrays | Produces correct shape | Use `argwhere` instead of `transpose(nonzero(a))` |
| `interpolation` in `percentile`/`quantile` | Deprecated | Use `method` instead |

---

## Consolidated Reference List

- numpy.argmax – https://numpy.org/doc/stable/reference/generated/numpy.argmax.html
- numpy.argmin – https://numpy.org/doc/stable/reference/generated/numpy.argmin.html
- numpy.nanargmax – https://numpy.org/doc/stable/reference/generated/numpy.nanargmax.html
- numpy.nanargmin – https://numpy.org/doc/stable/reference/generated/numpy.nanargmin.html
- numpy.where – https://numpy.org/doc/stable/reference/generated/numpy.where.html
- numpy.nonzero – https://numpy.org/doc/stable/reference/generated/numpy.nonzero.html
- numpy.argwhere – https://numpy.org/doc/2.0/reference/generated/numpy.argwhere.html
- numpy.flatnonzero – https://numpy.org/doc/stable/reference/generated/numpy.flatnonzero.html
- numpy.searchsorted – https://numpy.org/doc/1.26/reference/generated/numpy.searchsorted.html
- numpy.extract – https://numpy.org/doc/2.0/reference/generated/numpy.extract.html
- numpy.isin – https://numpy.org/doc/stable/reference/generated/numpy.isin.html
- Sorting, searching, and counting – https://numpy.org/doc/1.22/reference/routines.sort.html
- Python Numpy where() - Conditional Element Search – https://docs.vultr.com/python/third-party/numpy/where