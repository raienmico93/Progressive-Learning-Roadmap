# NumPy Uniqueness & Set Operations: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** NumPy uniqueness and set operations are a collection of routines that identify distinct values within arrays, compute counts and indices of those values, and perform mathematical set operations (intersection, union, difference, symmetric difference, and membership testing) on one-dimensional arrays.

**Technical Definition:** These operations are implemented in the `numpy.lib.arraysetops` module and are based on sorting. The core function `numpy.unique()` returns the sorted unique elements of an array, optionally along with the indices of first occurrences, the indices to reconstruct the original array, and the counts of each unique value. The set operations (`intersect1d`, `union1d`, `setdiff1d`, `setxor1d`) treat input arrays as mathematical sets, returning sorted arrays of unique values. `numpy.isin()` provides element-wise membership testing with broadcasting support.

**Beginner-Friendly Explanation:** NumPy's set operations let you answer questions like "which values appear in both lists?", "which values are in one list but not the other?", and "how many times does each value appear?" They work like mathematical sets, automatically removing duplicates and returning sorted results. The `unique()` function is the foundation, giving you the distinct values in an array along with useful metadata about them.

### Key Characteristics

- **Sort-Based:** All set operations are based on sorting, which determines their performance characteristics.
- **1-D Focus:** Most set operations flatten their inputs and operate on 1-D arrays.
- **Deduplication:** Set operations automatically remove duplicates from their results.
- **Sorted Output:** Results are returned in sorted order (unless `assume_unique=True` and inputs are sorted).
- **Metadata Rich:** `unique()` can return counts, indices, and inverse indices alongside the unique values.
- **Broadcasting:** `isin()` supports broadcasting over the element array while flattening the test array.
- **Performance Trade-offs:** `assume_unique=True` speeds up computation but produces incorrect results if inputs contain duplicates.

### Prerequisites

- NumPy installed (`pip install numpy`).
- Understanding of array shapes, axes, and indexing.
- Familiarity with the `import numpy as np` convention.
- Conceptual understanding of sets and mathematical set operations.

### Related Programming Areas

- **Data Analysis:** Identifying distinct categories, counting frequencies, and finding overlaps between datasets.
- **Database Operations:** Implementing JOIN, INTERSECT, and EXCEPT logic on array-backed data.
- **Machine Learning:** Computing class distributions, finding common samples, and filtering datasets.
- **Signal Processing:** Identifying unique frequency components and removing duplicates.
- **Bioinformatics:** Finding common genes, unique mutations, and set differences between sample groups.

### Core Concepts / Features

The following concepts are explored in detail: (1) `unique()` arrays, (2) Core set mechanics (`intersect1d`, `setdiff1d`, `union1d`), (3) Exclusive variations (`setxor1d`), and (4) Membership testing (`isin`).

---

## Core Concept 1: unique() Arrays

### Definitions

**Core Definition:** `np.unique()` returns the sorted unique elements of an array, with optional outputs for counts, indices of first occurrences, and indices for reconstructing the original array.

**Technical Definition:** `numpy.unique(ar, return_index=False, return_inverse=False, return_counts=False, axis=None, *, equal_nan=True)` finds the unique elements of an array. The input array is flattened unless `axis` is specified. The function returns the sorted unique values, and optionally: `unique_indices` (indices of first occurrences of unique values in the original array), `unique_inverse` (indices to reconstruct the original array from the unique array), and `unique_counts` (number of times each unique value appears). When `axis` is specified, subarrays indexed by the axis are sorted lexicographically and treated as elements.

**Beginner-Friendly Explanation:** `np.unique()` tells you what distinct values are in your array, sorted from smallest to largest. You can also ask it to tell you how many times each value appears (`return_counts`), where each unique value first appears in the original array (`return_index`), and how to rebuild the original array from the unique values (`return_inverse`).

### Purposes

- To extract distinct values from an array.
- To count the frequency of each unique value (frequency table).
- To map original values to their unique counterparts (encoding).
- To find the first occurrence index of each unique value.
- To reconstruct the original array from unique values and inverse indices.
- To find unique rows or subarrays along a specified axis.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.unique(ar, return_index=False, return_inverse=False, return_counts=False, axis=None, *, equal_nan=True)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `ar` | Input array. Flattened unless `axis` is specified. |
| `return_index` | If `True`, also return the indices of `ar` that result in the unique array. |
| `return_inverse` | If `True`, also return the indices of the unique array that can reconstruct `ar`. |
| `return_counts` | If `True`, also return the number of times each unique item appears. |
| `axis` | Axis to operate on. If `None`, `ar` is flattened. |
| `equal_nan` | If `True` (default), collapse multiple NaN values into one. |

**Syntax Rules:**

- The result is always sorted (unless `axis` is specified, in which case it is lexicographically sorted).
- `return_index` gives the indices of the **first** occurrences of the unique values in the original array.
- `return_inverse` can be used with fancy indexing: `unique[inverse]` reconstructs the original array.
- `return_counts` gives the number of occurrences of each unique value.
- When `axis` is specified, object arrays and structured arrays containing objects are not supported.

**Constraints and Limitations:**

- `unique` with `axis` can be memory-intensive for large arrays.
- `return_counts` was added in NumPy 1.9.0.
- `equal_nan` was added in NumPy 1.24.0.
- The `axis` parameter was added in NumPy 1.13.0.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Basic unique() with All Return Options**

```python
import numpy as np

# Step 2: Create an array with duplicates.
a = np.array([1, 2, 6, 4, 2, 3, 2])
print("Original array:", a)
print()

# Step 3: Get unique values only.
unique_vals = np.unique(a)
print("np.unique(a):", unique_vals)
print()

# Step 4: Get unique values with counts.
values, counts = np.unique(a, return_counts=True)
print("Unique values:", values)
print("Counts:", counts)
print("  (value 2 appears 3 times)")
print()

# Step 5: Get unique values with first-occurrence indices.
values, indices = np.unique(a, return_index=True)
print("Unique values:", values)
print("First occurrence indices:", indices)
print()

# Step 6: Get unique values with inverse indices (for reconstruction).
values, inverse = np.unique(a, return_inverse=True)
print("Unique values:", values)
print("Inverse indices:", inverse)
print("Reconstructed:", values[inverse])
print("Reconstruction matches original:", np.array_equal(values[inverse], a))
```

**Expected Output:**

```
Original array: [1 2 6 4 2 3 2]

np.unique(a): [1 2 3 4 6]

Unique values: [1 2 3 4 6]
Counts: [1 3 1 1 1]
  (value 2 appears 3 times)

Unique values: [1 2 3 4 6]
First occurrence indices: [0 1 5 3 2]

Unique values: [1 2 3 4 6]
Inverse indices: [0 1 4 3 1 2 1]
Reconstructed: [1 2 6 4 2 3 2]
Reconstruction matches original: True
```

**Why This Result Occurs:** 
- `np.unique(a)` returns `[1, 2, 3, 4, 6]` — the distinct values sorted. 
- `return_counts=True` shows that 2 appears 3 times and all others appear once. 
- `return_index=True` shows the first occurrence of each unique value: 1 at index 0, 2 at index 1, 3 at index 5, 4 at index 3, and 6 at index 2. 
- `return_inverse=True` provides the mapping from unique values back to the original array: `[0, 1, 4, 3, 1, 2, 1]`, meaning the original array can be reconstructed as `unique[inverse]`.

**Example 2: Unique Rows in a 2-D Array**

```python
import numpy as np

# Step 2: Create a 2-D array with duplicate rows.
a = np.array([[1, 0, 0],
              [1, 0, 0],
              [2, 3, 4]])
print("Original array:\n", a)
print()

# Step 3: Find unique rows along axis=0.
unique_rows = np.unique(a, axis=0)
print("Unique rows:\n", unique_rows)
print()

# Step 4: Get unique rows with counts.
values, counts = np.unique(a, axis=0, return_counts=True)
print("Unique rows:\n", values)
print("Counts:", counts)
```

**Expected Output:**

```
Original array:
 [[1 0 0]
 [1 0 0]
 [2 3 4]]

Unique rows:
 [[1 0 0]
 [2 3 4]]

Unique rows:
 [[1 0 0]
 [2 3 4]]
Counts: [2 1]
```

**Why This Result Occurs:** 
= With `axis=0`, the function treats each row as an element. 
= The duplicate row `[1, 0, 0]` appears twice, so it is collapsed into a single unique row with count 2.

### Real-World Cases

- **Frequency Tables:** Counting how many times each category appears in a dataset.
- **Encoding Categorical Data:** Mapping string labels to integer codes using `return_inverse`.
- **Duplicate Detection:** Finding and removing duplicate rows from a dataset.
- **Histogram Preparation:** Computing bin counts for discrete data.
- **Unique Identifiers:** Extracting distinct IDs from a transaction log.

### References

- numpy.unique – https://numpy.org/doc/stable/reference/generated/numpy.unique.html
- Set routines – https://numpy.org/doc/1.23/reference/routines.set.html
- numpy.unique (v1.14) – https://numpy.org/doc/1.14/reference/generated/numpy.unique.html

---

## Core Concept 2: Core Set Mechanics (intersect1d, setdiff1d, union1d)

### Definitions

**Core Definition:** `intersect1d()` finds common elements, `setdiff1d()` finds elements in the first array not in the second, and `union1d()` finds all unique elements in either array.

**Technical Definition:** `numpy.intersect1d(ar1, ar2, assume_unique=False, return_indices=False)` returns the sorted, unique values that are in both input arrays. `numpy.setdiff1d(ar1, ar2, assume_unique=False)` returns the unique values in `ar1` that are not in `ar2`. `numpy.union1d(ar1, ar2)` returns the unique, sorted union of the two input arrays. All inputs are flattened if not already 1-D. The `assume_unique` parameter, when `True`, can speed up computation by skipping the unique step, but produces incorrect results if inputs contain duplicates.

**Beginner-Friendly Explanation:** These three functions implement the classic mathematical set operations. `intersect1d` gives you what two lists have in common (the overlap). `setdiff1d` gives you what's in the first list but not the second (the difference). `union1d` gives you everything from both lists, with duplicates removed (the combination).

### Purposes

- To find common elements between two datasets (intersection).
- To find elements exclusive to one dataset (difference).
- To combine two datasets while removing duplicates (union).
- To implement database-style JOIN and EXCEPT operations.
- To compare two sets of identifiers, categories, or measurements.
- To find and return the indices of common elements (`return_indices` in `intersect1d`).

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.intersect1d(ar1, ar2, assume_unique=False, return_indices=False)
np.setdiff1d(ar1, ar2, assume_unique=False)
np.union1d(ar1, ar2)
```

**Component Breakdown:**

| Function | Returns | Key Parameter |
|----------|---------|---------------|
| `intersect1d` | Sorted unique common elements | `assume_unique`, `return_indices` |
| `setdiff1d` | Sorted unique elements in `ar1` not in `ar2` | `assume_unique` |
| `union1d` | Sorted unique union of `ar1` and `ar2` | — |

**Syntax Rules:**

- All functions flatten their inputs to 1-D if not already 1-D.
- `assume_unique=True` speeds up computation but produces incorrect results if inputs contain duplicates.
- `setdiff1d` returns a sorted result when `assume_unique=False`; otherwise, only sorted if the input is sorted.
- `intersect1d` with `return_indices=True` returns the indices of first occurrences of common values in both input arrays.

**Constraints and Limitations:**

- `union1d` always returns sorted output.
- `setdiff1d` uses `in1d` internally, which is generally O(N log N) but can be O(N²) in rare pathological cases.
- For string dtypes, `setdiff1d` can be extremely slow (known issue #21804).

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Intersection, Difference, and Union**

```python
import numpy as np

# Step 2: Create two arrays.
a = np.array([1, 3, 4, 3, 2])
b = np.array([3, 1, 2, 1, 5])
print("Array a:", a)
print("Array b:", b)
print()

# Step 3: Intersection (common elements).
intersection = np.intersect1d(a, b)
print("np.intersect1d(a, b):", intersection)
print("  (values in both a and b)")
print()

# Step 4: Set difference (in a but not b).
difference = np.setdiff1d(a, b)
print("np.setdiff1d(a, b):", difference)
print("  (values in a but not b)")
print()

# Step 5: Union (all unique elements).
union = np.union1d(a, b)
print("np.union1d(a, b):", union)
print("  (all unique values from both)")
print()

# Step 6: Intersection with indices.
xy, x_ind, y_ind = np.intersect1d(a, b, return_indices=True)
print("Intersection with indices:")
print("  Common values:", xy)
print("  Indices in a:", x_ind)
print("  Indices in b:", y_ind)
print("  a[x_ind]:", a[x_ind])
print("  b[y_ind]:", b[y_ind])
```

**Expected Output:**

```
Array a: [1 3 4 3 2]
Array b: [3 1 2 1 5]

np.intersect1d(a, b): [1 2 3]
  (values in both a and b)

np.setdiff1d(a, b): [4]
  (values in a but not b)

np.union1d(a, b): [1 2 3 4 5]
  (all unique values from both)

Intersection with indices:
  Common values: [1 2 3]
  Indices in a: [0 4 1]
  Indices in b: [1 2 0]
  a[x_ind]: [1 2 3]
  b[y_ind]: [1 2 3]
```

**Why This Result Occurs:** The common values between `a` and `b` are 1, 2, and 3 (intersection). The only value in `a` but not `b` is 4 (difference). The union of both arrays contains all unique values: 1, 2, 3, 4, 5. The indices show the first occurrence of each common value in the original arrays.

### Real-World Cases

- **Customer Analysis:** Finding customers who purchased from both categories A and B (`intersect1d`).
- **Inventory Management:** Finding products that are in stock but not on sale (`setdiff1d`).
- **Data Merging:** Combining two lists of email addresses while removing duplicates (`union1d`).
- **Fraud Detection:** Finding transactions that appear in both a watchlist and a transaction log.
- **Bioinformatics:** Finding genes that are differentially expressed in both conditions.

### References

- numpy.intersect1d – https://numpy.org/doc/1.20/reference/generated/numpy.intersect1d.html
- numpy.setdiff1d – https://numpy.org/doc/1.21/reference/generated/numpy.setdiff1d.html
- numpy.union1d – https://numpy.org/doc/1.20//reference/generated/numpy.union1d.html
- Set routines – https://numpy.org/doc/1.23/reference/routines.set.html

---

## Core Concept 3: Exclusive Variations (setxor1d)

### Definitions

**Core Definition:** `np.setxor1d()` returns the sorted, unique values that are in exactly one of the two input arrays, but not both (the symmetric difference).

**Technical Definition:** `numpy.setxor1d(ar1, ar2, assume_unique=False)` finds the set exclusive-or (symmetric difference) of two arrays. It returns the sorted, unique values that are in only one (not both) of the input arrays. The `assume_unique` parameter, when `True`, speeds up calculation but produces incorrect results if inputs contain duplicates. Inputs are flattened if not already 1-D.

**Beginner-Friendly Explanation:** `setxor1d` gives you the "exclusive or" of two sets: everything that is in one set or the other, but not in both. If two values appear in both arrays, they are excluded from the result. It's the opposite of intersection.

### Purposes

- To find values exclusive to one of two datasets (not shared).
- To implement the symmetric difference of two sets.
- To identify elements that are unique to each dataset.
- To detect differences between two versions of data.
- To complement `intersect1d` for a complete set comparison.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.setxor1d(ar1, ar2, assume_unique=False)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `ar1, ar2` | Input arrays. Flattened if not already 1-D. |
| `assume_unique` | If `True`, inputs are assumed to contain no duplicates. Speeds up calculation. |
| Returns | Sorted 1D array of unique values that are in only one of the input arrays. |

**Syntax Rules:**

- The result is sorted.
- Elements that appear in both arrays are excluded.
- `assume_unique=True` assumes both inputs are unique sets; if they are not, results are incorrect.
- The result is equivalent to `union1d(ar1, ar2) - intersect1d(ar1, ar2)`.

**Constraints and Limitations:**

- `assume_unique=True` with non-unique inputs produces incorrect results (may include duplicates).
- For string dtypes, performance may be poor.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Symmetric Difference with setxor1d**

```python
import numpy as np

# Step 2: Create two arrays.
a = np.array([1, 2, 3, 2, 4])
b = np.array([2, 3, 5, 7, 5])
print("Array a:", a)
print("Array b:", b)
print()

# Step 3: Symmetric difference.
xor_result = np.setxor1d(a, b)
print("np.setxor1d(a, b):", xor_result)
print("  (values in exactly one of a or b, not both)")
print()

# Step 4: Verify with union - intersection.
union = np.union1d(a, b)
intersection = np.intersect1d(a, b)
xor_manual = np.setdiff1d(union, intersection)
print("Union:", union)
print("Intersection:", intersection)
print("Union - Intersection:", xor_manual)
print("Matches setxor1d:", np.array_equal(xor_result, xor_manual))
print()

# Step 5: Demonstrate assume_unique speedup.
a_unique = np.array([1, 2, 3, 4])
b_unique = np.array([3, 4, 5, 6])
xor_fast = np.setxor1d(a_unique, b_unique, assume_unique=True)
print("With assume_unique=True:", xor_fast)
```

**Expected Output:**

```
Array a: [1 2 3 2 4]
Array b: [2 3 5 7 5]

np.setxor1d(a, b): [1 4 5 7]
  (values in exactly one of a or b, not both)

Union: [1 2 3 4 5 7]
Intersection: [2 3]
Union - Intersection: [1 4 5 7]
Matches setxor1d: True

With assume_unique=True: [1 2 5 6]
```

**Why This Result Occurs:** The values 2 and 3 appear in both arrays, so they are excluded from the symmetric difference. The remaining values (1, 4 from `a`; 5, 7 from `b`) form the result `[1, 4, 5, 7]`. The `assume_unique` example shows that when inputs are unique, the result differs because the inputs themselves are different (no duplicates to remove).

### Real-World Cases

- **Version Comparison:** Finding records that changed between two versions of a dataset.
- **Network Analysis:** Identifying nodes that are in one network but not the other.
- **Inventory Reconciliation:** Finding products that are unique to one warehouse.
- **A/B Testing:** Finding users who are in one test group but not the other.

### References

- numpy.setxor1d – https://numpy.org/doc/2.1/reference/generated/numpy.setxor1d.html
- Set routines – https://numpy.org/doc/1.23/reference/routines.set.html

---

## Core Concept 4: Membership Testing (isin)

### Definitions

**Core Definition:** `np.isin()` tests whether each element of an array is present in a second array of test values, returning a boolean mask of the same shape as the input.

**Technical Definition:** `numpy.isin(element, test_elements, assume_unique=False, invert=False)` calculates element in `test_elements`, broadcasting over `element` only. It returns a boolean array of the same shape as `element` that is `True` where an element of `element` is in `test_elements` and `False` otherwise. The `invert` parameter, when `True`, returns the inverse of the mask. `np.isin` is an element-wise function version of the Python keyword `in`.

**Beginner-Friendly Explanation:** `np.isin()` asks "is this value in that list?" for every element in your array. It returns a True/False mask that you can use to filter, count, or modify data. It's like the `in` operator in Python, but vectorized.

### Purposes

- To test whether elements of one array are present in another array.
- To create boolean masks for filtering data based on membership.
- To implement "is in" checks for entire arrays without loops.
- To efficiently test membership when the test set is large.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.isin(element, test_elements, assume_unique=False, invert=False)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `element` | Input array. Broadcasting is applied over this array. |
| `test_elements` | Values against which to test each element of `element`. Flattened if an array. |
| `assume_unique` | If `True`, both arrays are assumed unique. Speeds up calculation. |
| `invert` | If `True`, returns the inverse of the mask (element not in test_elements). |
| Returns | Boolean array of the same shape as `element`. |

**Syntax Rules:**

- `element` and `test_elements` are converted to arrays if they are not already.
- `test_elements` is flattened before testing.
- `invert=True` is equivalent to `np.invert(np.isin(...))` but faster.
- If `test_elements` is a set, it is converted to a 0-D object array; convert to a list first for correct behavior.

**Constraints and Limitations:**

- Performance is O(N log M) where N is the size of `element` and M is the size of `test_elements`.
- For very large test sets, `np.isin` is more efficient than Python's `in` within a loop.
- The `assume_unique` parameter can produce incorrect results if inputs contain duplicates.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Basic Membership Testing with isin()**

```python
import numpy as np

# Step 2: Create an element array.
element = np.array([[0, 2], [4, 6]])
print("Element array:\n", element)
print()

# Step 3: Define test elements.
test_elements = [1, 2, 4, 8]
print("Test elements:", test_elements)
print()

# Step 4: Test membership.
mask = np.isin(element, test_elements)
print("np.isin(element, test_elements):\n", mask)
print()

# Step 5: Extract matching values.
print("element[mask]:", element[mask])
print()

# Step 6: Get coordinates of matches.
coords = np.nonzero(mask)
print("Coordinates of matches:", coords)
print()

# Step 7: Inverted membership test.
inverted = np.isin(element, test_elements, invert=True)
print("np.isin(..., invert=True):\n", inverted)
print("element[inverted]:", element[inverted])
```

**Expected Output:**

```
Element array:
 [[0 2]
 [4 6]]

Test elements: [1, 2, 4, 8]

np.isin(element, test_elements):
 [[False  True]
 [ True False]]

element[mask]: [2 4]

Coordinates of matches: (array([0, 1]), array([1, 0]))

np.isin(..., invert=True):
 [[ True False]
 [False  True]]
element[inverted]: [0 6]
```

**Why This Result Occurs:** The values 2 and 4 are in the test set, so the mask is `True` at their positions. The coordinates `(0, 1)` and `(1, 0)` correspond to the positions of 2 and 4. The inverted mask selects the values that are *not* in the test set: 0 and 6.

### Real-World Cases

- **Data Validation:** Checking whether all IDs in a dataset are present in a reference table.
- **Filtering:** Selecting rows where a column value is in a list of allowed values.
- **Anomaly Detection:** Identifying values that are not in a set of expected values.
- **Recommendation Systems:** Testing whether a user has interacted with any item in a candidate list.

### References

- numpy.isin – https://numpy.org/doc/1.20/reference/generated/numpy.isin.html
- Set routines – https://numpy.org/doc/1.23/reference/routines.set.html

---

## Comparison Summary

| Function | Purpose | Returns | Key Parameter |
|----------|---------|---------|---------------|
| `np.unique` | Distinct values | Sorted unique array (optional metadata) | `return_counts`, `return_index`, `return_inverse`, `axis` |
| `np.intersect1d` | Common elements | Sorted unique common elements | `assume_unique`, `return_indices` |
| `np.setdiff1d` | Difference | Sorted unique elements in `ar1` not in `ar2` | `assume_unique` |
| `np.union1d` | Union | Sorted unique union | — |
| `np.setxor1d` | Symmetric difference | Sorted unique elements in exactly one array | `assume_unique` |
| `np.isin` | Membership test | Boolean mask | `assume_unique`, `invert` |

---

## Summary of Deprecated and Version-Specific Features

| Feature | Status | Recommendation |
|---------|--------|----------------|
| `return_counts` in `unique` | New in NumPy 1.9.0 | Available in NumPy ≥ 1.9 |
| `axis` in `unique` | New in NumPy 1.13.0 | Available in NumPy ≥ 1.13 |
| `equal_nan` in `unique` | New in NumPy 1.24.0 | Available in NumPy ≥ 1.24 |
| `return_indices` in `intersect1d` | New in NumPy 1.15.0 | Available in NumPy ≥ 1.15 |
| `np.in1d` | Deprecated in favor of `np.isin` | Use `np.isin` |
| `setdiff1d` with string dtypes | Known performance issue (#21804) | Consider using `np.isin` with boolean masking instead |
| `assume_unique=True` | Available for all set operations | Use only when inputs are verified to be unique |

---

## Consolidated Reference List

- numpy.unique – https://numpy.org/doc/stable/reference/generated/numpy.unique.html
- numpy.unique (v1.14) – https://numpy.org/doc/1.14/reference/generated/numpy.unique.html
- numpy.intersect1d – https://numpy.org/doc/1.20/reference/generated/numpy.intersect1d.html
- numpy.setdiff1d – https://numpy.org/doc/1.21/reference/generated/numpy.setdiff1d.html
- numpy.union1d – https://numpy.org/doc/1.20//reference/generated/numpy.union1d.html
- numpy.setxor1d – https://numpy.org/doc/2.1/reference/generated/numpy.setxor1d.html
- numpy.isin – https://numpy.org/doc/1.20/reference/generated/numpy.isin.html
- Set routines – https://numpy.org/doc/1.23/reference/routines.set.html
- numpy.lib.arraysetops – https://numpy.org/doc/1.21/reference/generated/numpy.lib.arraysetops.html
- GitHub Issue #21804: Speed issue with setdiff1d – https://github.com/numpy/numpy/issues/21804