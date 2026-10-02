# NumPy Sorting: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Sorting in NumPy is the process of rearranging array elements into a specified order (typically ascending) using optimized algorithms implemented in compiled C code, either by modifying the array in place or by producing a new sorted copy.

**Technical Definition:** NumPy implements sorting through three primary functions — `numpy.sort()` (returns a sorted copy), `ndarray.sort()` (sorts in place), and `numpy.argsort()` (returns the indices that would sort an array) — all of which accept `axis`, `kind`, and `order` parameters. Sorting algorithms are characterized by average speed, worst-case performance, work space size, and stability.

**Beginner-Friendly Explanation:** Sorting is how you put your data in order, from smallest to largest (or alphabetically, or by any other rule). NumPy gives you several ways to do this: one that changes your original array, one that leaves the original alone and gives you a new sorted array, and one that tells you the positions of the sorted items.

### Key Characteristics

- **In-Place vs. Copy:** `ndarray.sort()` modifies the array in place; `np.sort()` returns a new sorted copy.
- **Indirect Sorting:** `np.argsort()` returns indices that would sort the array, useful for aligning multiple arrays.
- **Axis-Controlled:** The `axis` parameter controls which dimension is sorted independently.
- **Algorithm Selection:** The `kind` parameter selects among quicksort, mergesort, heapsort, and stable (timsort).
- **Structured Arrays:** The `order` parameter specifies which fields to sort by, first, second, etc.
- **Stability:** Stable sorts preserve the relative order of equal elements.

### Prerequisites

- NumPy installed (`pip install numpy`).
- Understanding of array shapes, axes, and indexing.
- Familiarity with the `import numpy as np` convention.
- Conceptual understanding of sorting algorithms and time complexity.

### Related Programming Areas

- **Data Analysis:** Ranking records, finding top-N values, and ordering datasets.
- **Machine Learning:** Sorting predictions, computing ranking metrics, and organizing batches.
- **Finance:** Ranking assets by return, sorting transactions by date.
- **Scientific Computing:** Ordering measurement data and aligning parallel arrays.

### Core Concepts / Features

The following concepts are explored in detail: (1) In-place vs. copy sorting, (2) Indirect sorting with `argsort`, (3) Axis-controlled sorting, (4) Sorting algorithms and stability, and (5) Structured array sorting.

---

## Core Concept 1: In-Place vs. Copy Sorting

### Definitions

**Core Definition:** `ndarray.sort()` sorts an array in place, modifying the original data buffer, while `np.sort()` returns a new sorted copy, leaving the original array unchanged.

**Technical Definition:** `numpy.sort(a, axis=-1, kind=None, order=None)` returns a sorted copy of the input array. `ndarray.sort(axis=-1, kind='quicksort', order=None)` sorts the array in place and returns `None` (the original array is modified). When sorting arrays in place, if the array is a view on a different ndarray, the original array will be modified.

**Beginner-Friendly Explanation:** `np.sort()` is the safe option: it gives you a sorted version and your original array stays exactly as it was. `ndarray.sort()` is the memory-efficient option: it rearranges the array itself, saving the cost of allocating a new array — but the original order is lost forever.

### Purposes

- To obtain a sorted version of an array without altering the original (`np.sort`).
- To sort an array efficiently when the original order is no longer needed (`ndarray.sort`).
- To avoid memory allocation for large arrays by sorting in place.
- To intentionally modify shared data through a view.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
sorted_copy = np.sort(a, axis=-1, kind=None, order=None)
a.sort(axis=-1, kind='quicksort', order=None)   # In-place, returns None
```

**Component Breakdown:**

| Function | Returns | Memory Impact | Original Modified |
|----------|---------|---------------|-------------------|
| `np.sort(a)` | Sorted copy | Allocates new array | No |
| `a.sort()` | `None` | Modifies in place | Yes |

**Syntax Rules:**

- `np.sort` returns an array of the same type and shape as `a`.
- `ndarray.sort` returns `None`; the sorted result is in the original array.
- If the array is a view, in-place sorting modifies the base array.

**Constraints and Limitations:**

- In-place sorting on a view modifies the underlying data, which can have unintended side effects.
- `np.sort` uses more memory but preserves the original data.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: In-Place vs. Copy Sorting**

```python
import numpy as np

# Step 2: Create an array.
arr = np.array([3, 1, 4, 1, 5, 9, 2, 6])
print("Original:", arr)

# Step 3: Sort a copy (original unchanged).
sorted_copy = np.sort(arr)
print("np.sort(arr):", sorted_copy)
print("Original after np.sort:", arr)
print()

# Step 4: Sort in place (original modified).
arr.sort()
print("arr after arr.sort():", arr)
print("  (original array is now sorted)")
print()

# Step 5: Demonstrate the view trap.
base = np.array([5, 2, 8, 1])
view = base[:3]   # View of the first 3 elements
print("Base before view sort:", base)
view.sort()
print("Base after view.sort():", base)
print("  (sorting the view modified the base array)")
```

**Expected Output:**

```
Original: [3 1 4 1 5 9 2 6]
np.sort(arr): [1 1 2 3 4 5 6 9]
Original after np.sort: [3 1 4 1 5 9 2 6]

arr after arr.sort(): [1 1 2 3 4 5 6 9]
  (original array is now sorted)

Base before view sort: [5 2 8 1]
Base after view.sort(): [2 5 8 1]
  (sorting the view modified the base array)
```

**Why This Result Occurs:** `np.sort(arr)` returns a new sorted array and leaves `arr` unchanged. `arr.sort()` modifies `arr` directly. In the view example, `view` shares memory with `base`, so sorting `view` rearranges the first three elements of `base`.

### Real-World Cases

- **Data Preservation:** Using `np.sort` when you need both the original and sorted versions.
- **Memory-Constrained Environments:** Using `ndarray.sort` to avoid allocating a second array.
- **Pipeline Processing:** Sorting intermediate results in place when the original order is no longer needed.

### References

- numpy.sort – https://numpy.org/doc/1.26/reference/generated/numpy.sort.html
- numpy.ndarray.sort – https://numpy.org/doc/2.0/reference/generated/numpy.ndarray.sort.html

---

## Core Concept 2: Indirect Sorting with argsort

### Definitions

**Core Definition:** `np.argsort()` returns the indices that would sort an array, rather than the sorted values themselves.

**Technical Definition:** `numpy.argsort(a, axis=-1, kind=None, order=None)` performs an indirect sort along the given axis. It returns an array of indices of the same shape as `a` that index data along the given axis in sorted order. If `a` is one-dimensional, `a[index_array]` yields a sorted `a`. More generally, `np.take_along_axis(a, index_array, axis=axis)` always yields the sorted `a`, irrespective of dimensionality.

**Beginner-Friendly Explanation:** `argsort` doesn't give you the sorted list — it gives you the *positions* of the items in sorted order. This is useful when you need to sort one array while keeping other arrays aligned to the same order.

### Purposes

- To obtain the permutation that sorts an array.
- To sort multiple parallel arrays by the same key.
- To implement ranking and ordering without rearranging the original data.
- To use the sorted indices for `take_along_axis` or fancy indexing.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
index_array = np.argsort(a, axis=-1, kind=None, order=None)
sorted_a = a[index_array]   # For 1-D arrays
sorted_a = np.take_along_axis(a, index_array, axis=axis)   # For N-D arrays
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `a` | Input array. |
| `axis` | Axis along which to sort. Default `-1`. |
| `kind` | Sorting algorithm. |
| `order` | Fields to sort by (structured arrays). |
| Returns | Integer index array of the same shape as `a`. |

**Syntax Rules:**

- The returned index array has the same shape as `a`.
- For 1-D arrays, `a[index_array]` produces the sorted array.
- For N-D arrays, use `np.take_along_axis` instead of direct indexing.

**Constraints and Limitations:**

- `argsort` does not modify the original array.
- The resulting indices must be used carefully with multidimensional arrays.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Using argsort for Indirect Sorting**

```python
import numpy as np

# Step 2: Create an array.
a = np.array([30, 10, 20, 50, 40])
print("Original:", a)

# Step 3: Get the sorting indices.
indices = np.argsort(a)
print("np.argsort(a):", indices)
print("  (indices that would sort the array)")

# Step 4: Apply the indices to get the sorted array.
print("a[indices]:", a[indices])
print()

# Step 5: Use argsort to align multiple arrays.
names = np.array(['Alice', 'Bob', 'Charlie', 'Dave', 'Eve'])
scores = np.array([85, 92, 78, 95, 88])
order = np.argsort(scores)[::-1]   # Descending order
print("Names sorted by score (desc):", names[order])
print("Scores sorted (desc):", scores[order])
```

**Expected Output:**

```
Original: [30 10 20 50 40]
np.argsort(a): [1 2 0 4 3]
  (indices that would sort the array)
a[indices]: [10 20 30 40 50]

Names sorted by score (desc): ['Dave' 'Bob' 'Eve' 'Alice' 'Charlie']
Scores sorted (desc): [95 92 88 85 78]
```

**Why This Result Occurs:** `np.argsort(a)` returns `[1, 2, 0, 4, 3]` because the smallest element (10) is at index 1, the next (20) at index 2, and so on. Applying these indices to `a` yields the sorted array. Using `[::-1]` reverses the order for descending sort.

### Real-World Cases

- **Ranking:** Assigning ranks to students based on scores.
- **Data Alignment:** Sorting one column while keeping corresponding rows aligned.
- **Top-N Selection:** Finding the indices of the top-N values.

### References

- numpy.argsort – https://numpy.org/doc/1.18/reference/generated/numpy.argsort.html
- numpy.take_along_axis – https://numpy.org/doc/stable/reference/generated/numpy.take_along_axis.html

---

## Core Concept 3: Axis-Controlled Sorting

### Definitions

**Core Definition:** The `axis` parameter controls which dimension of a multidimensional array is sorted, allowing independent sorting along rows, columns, or any specified axis.

**Technical Definition:** `np.sort(a, axis=-1)` sorts along the last axis by default. `axis=0` sorts each column independently; `axis=1` sorts each row independently. `axis=None` flattens the array before sorting. All sort algorithms make temporary copies of the data when sorting along any but the last axis; consequently, sorting along the last axis is faster and uses less space than sorting along any other axis.

**Beginner-Friendly Explanation:** If you have a table of data, you can sort each column separately (`axis=0`) or each row separately (`axis=1`). The `axis` parameter tells NumPy which direction to sort along.

### Purposes

- To sort each row or column of a matrix independently.
- To sort along a specific dimension of a higher-dimensional array.
- To flatten and sort an entire array when the structure is irrelevant.
- To prepare data for row-wise or column-wise processing.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.sort(a, axis=-1)    # Sort along last axis
np.sort(a, axis=0)     # Sort each column
np.sort(a, axis=1)     # Sort each row
np.sort(a, axis=None)  # Flatten and sort
```

**Component Breakdown:**

| `axis` value | Effect |
|--------------|--------|
| `-1` (default) | Sort along the last axis. |
| `0` | Sort each column (down the rows). |
| `1` | Sort each row (across the columns). |
| `None` | Flatten the array and sort. |

**Syntax Rules:**

- The default `axis=-1` sorts along the last axis.
- Sorting along non-last axes requires temporary copies, which uses more memory and time.
- The result has the same shape as the input for `axis != None`.

**Constraints and Limitations:**

- Sorting along non-last axes is slower due to temporary copies.
- For higher-dimensional arrays, the axis numbering follows the shape tuple order.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Sorting Along Different Axes**

```python
import numpy as np

# Step 2: Create a 2-D array.
a = np.array([[3, 1, 4],
              [1, 5, 9],
              [2, 6, 5]])
print("Original:\n", a)

# Step 3: Sort along axis=0 (each column).
print("np.sort(a, axis=0):\n", np.sort(a, axis=0))
print()

# Step 4: Sort along axis=1 (each row).
print("np.sort(a, axis=1):\n", np.sort(a, axis=1))
print()

# Step 5: Flatten and sort.
print("np.sort(a, axis=None):", np.sort(a, axis=None))
```

**Expected Output:**

```
Original:
 [[3 1 4]
 [1 5 9]
 [2 6 5]]
np.sort(a, axis=0):
 [[1 1 4]
 [2 5 5]
 [3 6 9]]

np.sort(a, axis=1):
 [[1 3 4]
 [1 5 9]
 [2 5 6]]

np.sort(a, axis=None): [1 1 2 3 4 5 5 6 9]
```

**Why This Result Occurs:** `axis=0` sorts each column independently: the first column [3, 1, 2] becomes [1, 2, 3]. `axis=1` sorts each row independently: the first row [3, 1, 4] becomes [1, 3, 4]. `axis=None` flattens the entire array and sorts all 9 elements.

### Real-World Cases

- **Table Sorting:** Sorting each column of a data table independently.
- **Image Processing:** Sorting pixel values along the channel axis.
- **Batch Processing:** Sorting each sample's features independently.

### References

- numpy.sort – https://numpy.org/doc/1.26/reference/generated/numpy.sort.html
- numpy.ndarray.sort – https://numpy.org/doc/1.16/reference/generated/numpy.ndarray.sort.html

---

## Core Concept 4: Sorting Algorithms & Stability

### Definitions

**Core Definition:** NumPy supports four sorting algorithm choices via the `kind` parameter: `'quicksort'`, `'mergesort'`, `'heapsort'`, and `'stable'`, each with different performance and stability characteristics.

**Technical Definition:** The algorithms are characterized by average speed, worst-case performance, work space size, and stability. A stable sort keeps items with the same key in the same relative order. The four algorithms have the following properties: `'quicksort'` (speed 1, worst case O(n²), no extra space, not stable), `'heapsort'` (speed 3, worst case O(n log n), no extra space, not stable), `'mergesort'` (speed 2, worst case O(n log n), ~n/2 extra space, stable), and `'timsort'` (speed 2, worst case O(n log n), ~n/2 extra space, stable). Both `'stable'` and `'mergesort'` use timsort or radix sort under the covers.

**Beginner-Friendly Explanation:** Different sorting algorithms have different strengths. Quicksort is usually fastest but can be slow in the worst case and doesn't preserve the order of equal elements. Mergesort and timsort are stable — equal elements keep their original relative order — which is important for multi-key sorting. Heapsort is a good all-rounder with guaranteed performance.

### Purposes

- To choose the appropriate algorithm for the data size and requirements.
- To ensure stability when sorting by multiple keys or when equal elements must preserve order.
- To optimize for speed (`quicksort`), guaranteed performance (`heapsort`), or stability (`mergesort`/`stable`).
- To avoid worst-case O(n²) behavior with `heapsort` or `mergesort`.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.sort(a, kind='quicksort')   # Default
np.sort(a, kind='heapsort')
np.sort(a, kind='mergesort')
np.sort(a, kind='stable')
```

**Component Breakdown:**

| Algorithm | Speed | Worst Case | Extra Space | Stable |
|-----------|-------|-----------|-------------|--------|
| `'quicksort'` | 1 (fastest) | O(n²) | 0 | No |
| `'heapsort'` | 3 (slowest) | O(n log n) | 0 | No |
| `'mergesort'` | 2 | O(n log n) | ~n/2 | Yes |
| `'stable'` | 2 | O(n log n) | ~n/2 | Yes |

**Syntax Rules:**

- The default is `'quicksort'`.
- `'stable'` automatically chooses the best stable sorting algorithm for the data type being sorted.
- `'mergesort'` is retained for backwards compatibility; it and `'stable'` currently map to timsort or radix sort.
- The datatype determines which of `'mergesort'` or `'timsort'` is actually used, even if `'mergesort'` is specified.

**Constraints and Limitations:**

- `'quicksort'` has O(n²) worst-case performance, though this is rare in practice due to introsort (it switches to heapsort when recursion depth is too deep).
- Stable sorts require extra memory (~n/2).

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Comparing Sorting Algorithms**

```python
import numpy as np
import time

# Step 2: Create a large array.
arr = np.random.rand(1_000_000)
print("Array size:", arr.size)

# Step 3: Time quicksort.
start = time.time()
_ = np.sort(arr, kind='quicksort')
qt = time.time() - start
print(f"quicksort: {qt:.4f}s")

# Step 4: Time heapsort.
start = time.time()
_ = np.sort(arr, kind='heapsort')
ht = time.time() - start
print(f"heapsort: {ht:.4f}s")

# Step 5: Time mergesort.
start = time.time()
_ = np.sort(arr, kind='mergesort')
mt = time.time() - start
print(f"mergesort: {mt:.4f}s")

# Step 6: Demonstrate stability.
data = np.array([(1, 'b'), (1, 'a'), (2, 'c'), (2, 'a')],
                dtype=[('key', int), ('value', 'U1')])
stable_sorted = np.sort(data, kind='stable')
print("\nStable sort (preserves original order of equal keys):")
print(stable_sorted)
```

**Expected Output (approximate):**

```
Array size: 1000000
quicksort: 0.0852s
heapsort: 0.1523s
mergesort: 0.0987s

Stable sort (preserves original order of equal keys):
[(1, 'b') (1, 'a') (2, 'c') (2, 'a')]
```

**Why This Result Occurs:** Quicksort is fastest on average. Heapsort is slower but has guaranteed O(n log n) performance. Mergesort is stable, preserving the relative order of records with equal keys (the two `(1, 'b')` and `(1, 'a')` records remain in their original order).

### Real-World Cases

- **Multi-Key Sorting:** Using `'stable'` to sort by a primary key, then a secondary key, preserving the first sort's order.
- **Large Datasets:** Using `'quicksort'` for speed when stability is not required.
- **Real-Time Systems:** Using `'heapsort'` for guaranteed worst-case performance.

### References

- numpy.sort – https://numpy.org/doc/1.26/reference/generated/numpy.sort.html
- Sorting algorithms – https://numpy.org/doc/1.26/reference/generated/numpy.sort.html#numpy.sort

---

## Core Concept 5: Structured Array Sorting

### Definitions

**Core Definition:** Structured array sorting uses the `order` parameter to specify which named fields to sort by, allowing complex records to be ordered by multiple criteria.

**Technical Definition:** When `a` is a structured array, the `order` argument specifies which fields to compare first, second, etc. A single field can be specified as a string, and not all fields need be specified, but unspecified fields will still be used, in the order in which they come up in the dtype, to break ties. Structured arrays are sorted lexically by `argsort`.

**Beginner-Friendly Explanation:** If your array is like a database table with named columns, `order` lets you sort by one column, then another, and so on. For example, sort by last name, then by first name.

### Purposes

- To sort structured records by one or more named fields.
- To implement multi-column sorting similar to SQL `ORDER BY`.
- To organize complex datasets with heterogeneous field types.
- To align records by multiple criteria.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.sort(a, order='field_name')
np.sort(a, order=['field1', 'field2'])
a.sort(order=['field1', 'field2'])
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `order` | String (single field) or list of strings (multiple fields). |
| Field order | Fields are compared in the order listed. |
| Tie-breaking | Unspecified fields are used in dtype order. |

**Syntax Rules:**

- `order` can be a single field name (string) or a list of field names.
- The fields are compared lexicographically in the specified order.
- Unspecified fields are used in the order they appear in the dtype to break ties.
- Works with both `np.sort` and `ndarray.sort`.

**Constraints and Limitations:**

- The `order` parameter only applies to structured arrays.
- Sorting by multiple fields uses a stable algorithm internally.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Sorting a Structured Array by Multiple Fields**

```python
import numpy as np

# Step 2: Create a structured array.
data = np.array([('Alice', 85, 'A'),
                 ('Bob', 92, 'B'),
                 ('Charlie', 78, 'A'),
                 ('Alice', 92, 'A'),
                 ('Bob', 85, 'B')],
                dtype=[('name', 'U10'), ('score', 'i4'), ('grade', 'U1')])
print("Original:\n", data)

# Step 3: Sort by name only.
sorted_by_name = np.sort(data, order='name')
print("\nSorted by name:\n", sorted_by_name)

# Step 4: Sort by name, then by score.
sorted_multi = np.sort(data, order=['name', 'score'])
print("\nSorted by name, then score:\n", sorted_multi)

# Step 5: Sort in place.
data.sort(order=['score', 'name'])
print("\nIn-place sort by score, then name:\n", data)
```

**Expected Output:**

```
Original:
 [('Alice', 85, 'A') ('Bob', 92, 'B') ('Charlie', 78, 'A')
 ('Alice', 92, 'A') ('Bob', 85, 'B')]

Sorted by name:
 [('Alice', 85, 'A') ('Alice', 92, 'A') ('Bob', 92, 'B')
 ('Bob', 85, 'B') ('Charlie', 78, 'A')]

Sorted by name, then score:
 [('Alice', 85, 'A') ('Alice', 92, 'A') ('Bob', 85, 'B')
 ('Bob', 92, 'B') ('Charlie', 78, 'A')]

In-place sort by score, then name:
 [('Charlie', 78, 'A') ('Alice', 85, 'A') ('Bob', 85, 'B')
 ('Alice', 92, 'A') ('Bob', 92, 'B')]
```

**Why This Result Occurs:** `order='name'` sorts records by the name field. `order=['name', 'score']` sorts first by name, then by score within each name group. The in-place sort by `['score', 'name']` first orders by score, then by name for records with the same score.

### Real-World Cases

- **Database Operations:** Sorting records by multiple columns similar to SQL `ORDER BY`.
- **Employee Records:** Sorting by department, then by salary.
- **Product Catalogs:** Sorting by category, then by price.

### References

- numpy.ndarray.sort – https://numpy.org/doc/2.0/reference/generated/numpy.ndarray.sort.html
- numpy.sort – https://numpy.org/doc/1.26/reference/generated/numpy.sort.html
- numpy.lexsort – https://numpy.org/doc/stable/reference/generated/numpy.lexsort.html

---

## Comparison Summary

| Function | Returns | Modifies Original | Key Parameter |
|----------|---------|-------------------|---------------|
| `np.sort(a)` | Sorted copy | No | `axis`, `kind`, `order` |
| `a.sort()` | `None` | Yes | `axis`, `kind`, `order` |
| `np.argsort(a)` | Index array | No | `axis`, `kind`, `order` |
| `np.lexsort(keys)` | Index array | No | Multiple keys |

---

## Summary of Deprecated and Version-Specific Features

| Feature | Status | Recommendation |
|---------|--------|----------------|
| `'mergesort'` kind | Retained for backwards compatibility | Use `'stable'` for clarity |
| `'stable'` kind | New in NumPy 1.15.0 | Use for stable sorts |
| `kind=None` in `argsort` | Defaults to `'quicksort'` | Specify explicitly for clarity |
| `axis=None` in `sort` | Available | Flattens before sorting |

---

## Consolidated Reference List

- numpy.sort – https://numpy.org/doc/1.26/reference/generated/numpy.sort.html
- numpy.ndarray.sort – https://numpy.org/doc/2.0/reference/generated/numpy.ndarray.sort.html
- numpy.argsort – https://numpy.org/doc/1.18/reference/generated/numpy.argsort.html
- numpy.lexsort – https://numpy.org/doc/stable/reference/generated/numpy.lexsort.html
- numpy.searchsorted – https://numpy.org/doc/1.26/reference/generated/numpy.searchsorted.html
- numpy.partition – https://numpy.org/doc/stable/reference/generated/numpy.partition.html
- Sorting, searching, and counting – https://numpy.org/doc/1.22/reference/routines.sort.html
- numpy.take_along_axis – https://numpy.org/doc/stable/reference/generated/numpy.take_along_axis.html