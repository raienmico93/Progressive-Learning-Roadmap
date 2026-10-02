# NumPy Selection: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Selection in NumPy is the set of operations that pick specific elements from an array based on conditions, index maps, or ranking criteria, producing either a new array or a filtered view.

**Technical Definition:** NumPy selection encompasses multi-condition branching (`np.select`), index-driven mapping (`np.choose`), advanced boolean and fancy indexing, partial sorting for top-K extraction (`np.partition`/`np.argpartition`), and random sampling (`np.random.choice` / `Generator.choice`). These operations leverage compiled C loops, broadcasting, and O(N) selection algorithms to avoid full sorts when only a subset is needed.

**Beginner-Friendly Explanation:** Selection is how you pick out the parts of your data that you care about. You can pick based on conditions ("give me everything above 10"), based on a lookup table ("use this index to choose from these options"), based on ranking ("give me the top 5 values"), or randomly ("give me a random sample"). NumPy does all of this quickly without loops.

### Key Characteristics

- **Condition-Driven:** `np.select` handles multi-condition branching with an ordered checklist of criteria.
- **Index-Driven:** `np.choose` maps an index array to a list of choice arrays.
- **Mask-Driven:** Boolean and fancy indexing filter elements based on conditions or explicit index arrays.
- **O(N) Partial Sorting:** `np.partition` and `np.argpartition` find top-K elements in O(N) instead of O(N log N).
- **Random Sampling:** `Generator.choice` samples with or without replacement, with optional weights and axis control.

### Prerequisites

- NumPy installed (`pip install numpy`).
- Understanding of array shapes, axes, and indexing.
- Familiarity with the `import numpy as np` convention.
- Conceptual understanding of boolean masks and broadcasting.

### Related Programming Areas

- **Data Analysis:** Conditional labeling, filtering, and ranking.
- **Machine Learning:** Top-K accuracy, sampling batches, and conditional activation.
- **Image Processing:** LUT-based color mapping and region masking.
- **Scientific Computing:** Piecewise function evaluation and random sampling.

### Core Concepts / Features

The following concepts are explored in detail: (1) Multi-condition selection with `np.select`, (2) Index-driven mapping with `np.choose`, (3) Advanced Boolean filtering & masking, (4) Top-K selection & partial sorting, and (5) Sub-array sampling with random selection.

---

## Core Concept 1: Multi-Condition Selection with np.select

### Definitions

**Core Definition:** `np.select()` returns an array drawn from elements in a choice list, depending on an ordered list of conditions.

**Technical Definition:** `numpy.select(condlist, choicelist, default=0)` returns an array drawn from elements in `choicelist`, depending on conditions. The output at position `m` is the `m`-th element of the array in `choicelist` where the `m`-th element of the corresponding array in `condlist` is `True`. When multiple conditions are satisfied, the first one encountered in `condlist` is used. The `default` value applies to any element where none of the conditions matched.

**Beginner-Friendly Explanation:** `np.select` is like a vectorized if-elif-else chain. You give it a list of conditions and a list of corresponding values, and it picks the value from the first condition that is true for each element.

### Purposes

- To implement multi-way branching (if-elif-else) in a vectorized form.
- To assign values based on an ordered priority of conditions.
- To avoid deeply nested `np.where` calls for 3+ conditions.
- To handle tiered classification, pricing, and grading.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.select(condlist, choicelist, default=0)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `condlist` | List of boolean ndarrays. Conditions determine which choice is used. |
| `choicelist` | List of ndarrays from which output elements are taken. Same length as `condlist`. |
| `default` | Scalar inserted when all conditions evaluate to `False`. Default is 0. |
| Returns | ndarray with the same shape as the broadcast condition arrays. |

**Syntax Rules:**

- `condlist` and `choicelist` must have the same length.
- Conditions are evaluated in order; the first `True` condition wins.
- The default value applies to elements where no condition is `True`.

**Constraints and Limitations:**

- `np.select` is more efficient and readable than nested `np.where` for 3+ conditions.
- All conditions are evaluated (not short-circuited), which may be less efficient for expensive conditions.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Multi-Condition Selection with np.select()**

```python
import numpy as np

# Step 2: Create an array of values.
x = np.arange(1, 11)
print("Array:", x)
print()

# Step 3: Define conditions and choices.
conditions = [x < 3, x < 6, x < 9]
choices = [x, x**2, x**3]
print("Conditions:")
print("  x < 3:", conditions[0])
print("  x < 6:", conditions[1])
print("  x < 9:", conditions[2])
print()

# Step 4: Apply np.select.
result = np.select(conditions, choices, default=0)
print("np.select result:", result)
print("  (x for x<3, x^2 for 3<=x<6, x^3 for 6<=x<9, 0 for x>=9)")
print()

# Step 5: Compare with nested np.where (less readable).
nested = np.where(x < 3, x,
         np.where(x < 6, x**2,
         np.where(x < 9, x**3, 0)))
print("Nested np.where result:", nested)
print("Results match:", np.array_equal(result, nested))
```

**Expected Output:**

```
Array: [ 1  2  3  4  5  6  7  8  9 10]

Conditions:
  x < 3: [ True  True False False False False False False False False]
  x < 6: [ True  True  True  True  True False False False False False]
  x < 9: [ True  True  True  True  True  True  True  True False False]

np.select result: [  1   2   9  16  25 216 343 512   0   0]
  (x for x<3, x^2 for 3<=x<6, x^3 for 6<=x<9, 0 for x>=9)

Nested np.where result: [  1   2   9  16  25 216 343 512   0   0]
Results match: True
```

**Why This Result Occurs:** `np.select` evaluates conditions in order. For `x=1` and `x=2`, the first condition (`x < 3`) is `True`, so the value is `x`. For `x=3, 4, 5`, the first condition is `False`, but the second (`x < 6`) is `True`, so the value is `x²`. For `x=6, 7, 8`, the third condition (`x < 9`) is `True`, so the value is `x³`. For `x=9, 10`, no condition is `True`, so the default `0` is used.

### Real-World Cases

- **Grading Systems:** Assigning letter grades based on multiple score thresholds.
- **Pricing Tiers:** Computing prices based on quantity brackets.
- **Risk Classification:** Classifying data into multiple risk categories.
- **Conditional Labeling:** Assigning labels like 'high_conf', 'medium_conf', 'low_conf' based on probability thresholds.

### References

- numpy.select – https://docs.scipy.org/doc/numpy-1.17.0/reference/generated/numpy.select.html
- Conditional Operations — np.where, np.select, np.clip – https://raw.githubusercontent.com

---

## Core Concept 2: Index-Driven Mapping with np.choose

### Definitions

**Core Definition:** `np.choose()` constructs an array from an index array and a list of choice arrays, using the indices to select values from the choices.

**Technical Definition:** `numpy.choose(a, choices, out=None, mode='raise')` constructs an array from an index array and a list of arrays to choose from. Given an "index" array `a` of integers and a sequence of `n` arrays (`choices`), `a` and each choice array are first broadcast to arrays of a common shape. Then, a new array with that shape is created where each element is the value in `choices[i]` at the same position, where `i` is the value of `a` at that position. The `mode` parameter controls how out-of-range indices are handled: `'raise'` (default) raises an exception, `'wrap'` uses modular arithmetic, and `'clip'` maps negative values to 0 and values greater than `n-1` to `n-1`.

**Beginner-Friendly Explanation:** `np.choose` is like a lookup table. You have an array of indices (0, 1, 2, ...) and a list of arrays. For each position, `np.choose` looks at the index and picks the value from the corresponding array.

### Purposes

- To implement index-driven mapping and lookup tables.
- To apply different transformations to different elements based on a category index.
- To perform vectorized LUT (lookup table) operations.
- To broadcast and merge multiple choice arrays.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.choose(a, choices, out=None, mode='raise')
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `a` | Integer array with values in `[0, n-1]` (unless `mode='wrap'` or `'clip'`). |
| `choices` | Sequence of arrays. `a` and all choices must be broadcastable to the same shape. |
| `out` | Optional output array. |
| `mode` | `'raise'` (default), `'wrap'`, or `'clip'`. Controls out-of-range indices. |

**Syntax Rules:**

- `a` and each choice array are broadcast to a common shape.
- In `'raise'` mode, all indices must be in `[0, n-1]`.
- In `'wrap'` mode, indices are taken modulo `n`.
- In `'clip'` mode, indices < 0 become 0, and indices > n-1 become n-1.

**Constraints and Limitations:**

- The `out` parameter is always buffered in `'raise'` mode; use other modes for better performance.
- `choices` must be a sequence of arrays, not a single 2-D array (though the outermost dimension of a single array is used as the sequence if passed directly).

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Basic np.choose() Mapping**

```python
import numpy as np

# Step 2: Create an index array.
indices = np.array([0, 1, 2, 1, 0])
print("Indices:", indices)
print()

# Step 3: Define choice arrays.
choice0 = np.array([10, 20, 30, 40, 50])
choice1 = np.array([100, 200, 300, 400, 500])
choice2 = np.array([1000, 2000, 3000, 4000, 5000])

# Step 4: Apply np.choose.
result = np.choose(indices, [choice0, choice1, choice2])
print("np.choose(indices, [choice0, choice1, choice2]):", result)
print("  (picks from choice[i] where indices == i)")
print()

# Step 5: Demonstrate mode='clip' and mode='wrap'.
out_of_range = np.array([0, 1, 5, -1])
print("Out-of-range indices:", out_of_range)
print("mode='clip':", np.choose(out_of_range, [choice0, choice1], mode='clip'))
print("mode='wrap':", np.choose(out_of_range, [choice0, choice1], mode='wrap'))
```

**Expected Output:**

```
Indices: [0 1 2 1 0]

np.choose(indices, [choice0, choice1, choice2]): [  10  200 3000  400   50]
  (picks from choice[i] where indices == i)

Out-of-range indices: [ 0  1  5 -1]
mode='clip': [10 20 20 10]
mode='wrap': [10 20 20 20]
```

**Why This Result Occurs:** At position 0, the index is 0, so `choice0[0] = 10` is selected. At position 1, the index is 1, so `choice1[1] = 200` is selected. At position 2, the index is 2, so `choice2[2] = 3000`. With `mode='clip'`, index 5 becomes 1 (max index) and −1 becomes 0. With `mode='wrap'`, index 5 becomes 1 (5 mod 2), and −1 becomes 1 (−1 mod 2).

### Real-World Cases

- **Color Lookup Tables:** Mapping an index image to RGB values via `np.choose`.
- **Categorical Transformations:** Applying different scaling factors to different categories.
- **Piecewise Functions:** Selecting function outputs based on region indices.

### References

- numpy.choose – https://numpy.org/doc/2.0/reference/generated/numpy.choose.html
- numpy.ndarray.choose – https://numpy.org/doc/2.0/reference/generated/numpy.ndarray.choose.html

---

## Core Concept 3: Advanced Boolean Filtering & Masking

### Definitions

**Core Definition:** Advanced boolean filtering uses boolean arrays (masks) to select elements that satisfy one or more conditions, and fancy indexing uses integer arrays to select elements at specified positions.

**Technical Definition:** Boolean array indexing occurs when the index is a boolean ndarray of the same shape as the array being indexed. It is equivalent to `x[obj.nonzero()]` and returns a 1-D array of elements where the mask is `True`. Fancy indexing uses integer arrays to select elements; unlike basic slicing, it returns a copy, not a view. Advanced indexing includes all cases where arrays are indexed by other arrays.

**Beginner-Friendly Explanation:** Boolean masks are like stencils — you ask a question ("which numbers are positive?") and NumPy gives you a True/False pattern. Fancy indexing lets you pick arbitrary positions by giving a list of indices.

### Purposes

- To filter array elements based on complex conditions.
- To select arbitrary, non-contiguous elements by index.
- To combine multiple conditions with `&`, `|`, and `~`.
- To modify selected elements in place.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
mask = (condition1) & (condition2)
selected = array[mask]
selected = array[[0, 2, 5]]   # Fancy indexing
selected = array[integer_array]
```

**Component Breakdown:**

| Technique | Description | Return Type |
|-----------|-------------|-------------|
| Boolean indexing | `array[array > 5]` | Copy (1-D) |
| Fancy indexing | `array[[0, 2, 5]]` | Copy |
| Multi-condition | `array[(a > 1) & (a < 5)]` | Copy |

**Syntax Rules:**

- Boolean masks must be broadcastable to the array's shape.
- Use `&`, `|`, `~` for logical operations; never `and`, `or`, `not`.
- Fancy indexing with integer arrays always returns a copy.

**Constraints and Limitations:**

- Boolean indexing returns a 1-D array, losing the original shape.
- Fancy indexing cannot return a view; it always returns a copy.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Combining Boolean and Fancy Indexing**

```python
import numpy as np

# Step 2: Create an array.
a = np.array([10, 25, 3, 47, 8, 19, 62])
print("Array:", a)

# Step 2: Boolean filtering.
mask = a > 20
print("a > 20:", mask)
print("a[a > 20]:", a[mask])

# Step 3: Multi-condition boolean filtering.
between = (a > 10) & (a < 50)
print("\n(a > 10) & (a < 50):", between)
print("a[(a > 10) & (a < 50)]:", a[between])

# Step 4: Fancy indexing.
indices = [0, 3, 6]
print("\nFancy indexing with indices", indices, ":", a[indices])

# Step 5: Modify elements via boolean mask.
a[a < 10] = 0
print("\nAfter a[a < 10] = 0:", a)
```

**Expected Output:**

```
Array: [10 25  3 47  8 19 62]
a > 20: [False  True False  True False False  True]
a[a > 20]: [25 47 62]

(a > 10) & (a < 50): [False  True False  True False  True False]
a[(a > 10) & (a < 50)]: [25 47 19]

Fancy indexing with indices [0, 3, 6] : [10 47 62]

After a[a < 10] = 0: [10 25  0 47  0 19 62]
```

**Why This Result Occurs:** Boolean filtering selects elements where the condition is `True`. The multi-condition mask combines two conditions with `&`. Fancy indexing selects elements at the specified positions. Assignment with a boolean mask modifies the original array in place.

### Real-World Cases

- **Data Cleaning:** Removing outliers: `data = data[(data > lower) & (data < upper)]`.
- **Image Processing:** Selecting bright pixels: `image[image > 200]`.
- **Feature Selection:** Picking specific columns by index: `X[:, [0, 3, 5]]`.

### References

- Boolean array indexing – https://numpy.org/doc/stable/user/basics.indexing.html#boolean-array-indexing
- Advanced NumPy Indexing: A Comprehensive Guide – https://raw.githubusercontent.com
- NEP 21 – Advanced indexing – https://chromium.journaldev.googlesource.com

---

## Core Concept 4: Top-K Selection & Partial Sorting

### Definitions

**Core Definition:** `np.partition()` and `np.argpartition()` partially sort an array so that the k-th element is in its final sorted position, with all smaller elements before it and all larger elements after it, in O(N) time.

**Technical Definition:** `numpy.partition(a, kth, axis=-1, kind='introselect', order=None)` returns a partitioned copy of an array. `numpy.argpartition()` returns the indices that would partition the array. The k-th element will be in its final sorted position and all smaller elements will be moved before it and all larger elements behind it. The ordering of elements within the partitions is undefined. This is achieved using the introselect algorithm, which has a linear time complexity of O(N).

**Beginner-Friendly Explanation:** If you only need the top 5 values, you don't need to sort the whole array. `np.partition` rearranges the array so the k-th smallest element is in its correct position, with smaller elements before it and larger after. This is much faster than a full sort — O(N) instead of O(N log N).

### Purposes

- To find the top-K or bottom-K elements efficiently.
- To compute order statistics (median, percentiles) without a full sort.
- To select the N largest values in O(N) time.
- To implement ranking and selection algorithms.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
partitioned = np.partition(a, kth, axis=-1)
indices = np.argpartition(a, kth, axis=-1)
top_k = a[np.argpartition(a, -k)[-k:]]
```

**Component Breakdown:**

| Function | Description |
|----------|-------------|
| `np.partition(a, kth)` | Returns a partitioned copy. |
| `np.argpartition(a, kth)` | Returns the indices that partition the array. |
| `kth` | Element index (or sequence of indices) to partition by. |
| `axis` | Axis along which to partition. Default -1. |

**Syntax Rules:**

- The k-th element will be in its final sorted position.
- All elements before k are smaller or equal; all after are larger or equal.
- The order within partitions is undefined.
- Can pass a sequence of kth values to partition multiple positions at once.

**Constraints and Limitations:**

- `argpartition` does not return a fully sorted top-K; use `np.sort` on the top-K slice for ordered results.
- The algorithm is `introselect`, which switches to median-of-medians for worst-case O(N).

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Finding Top-K Elements with argpartition**

```python
import numpy as np

# Step 2: Create an array.
a = np.array([30, 10, 50, 20, 40, 70, 60])
print("Array:", a)

# Step 3: Partition around the 3rd smallest element.
partitioned = np.partition(a, 3)
print("np.partition(a, 3):", partitioned)
print("  (element at index 3 is in its sorted position)")

# Step 4: Find the top-3 elements.
k = 3
top_k_indices = np.argpartition(a, -k)[-k:]
top_k_values = a[top_k_indices]
print("\nTop-3 indices:", top_k_indices)
print("Top-3 values (unsorted):", top_k_values)

# Step 5: Sort the top-K for ordered results.
ordered_top_k = np.sort(top_k_values)
print("Top-3 values (sorted):", ordered_top_k)

# Step 6: Find the median using partition.
median_idx = len(a) // 2
median = np.partition(a, median_idx)[median_idx]
print("\nMedian (via partition):", median)
```

**Expected Output:**

```
Array: [30 10 50 20 40 70 60]
np.partition(a, 3): [10 20 30 40 50 70 60]
  (element at index 3 is in its sorted position)

Top-3 indices: [5 6 2]
Top-3 values (unsorted): [70 60 50]
Top-3 values (sorted): [50 60 70]

Median (via partition): 40
```

**Why This Result Occurs:** `np.partition(a, 3)` places the 3rd smallest element (40) at index 3, with all smaller elements to the left and larger to the right. `argpartition(a, -3)[-3:]` gives the indices of the 3 largest elements. Sorting those top-3 values gives the ordered result. The median is found by partitioning around the middle index.

### Real-World Cases

- **Top-K Accuracy:** Computing the top-5 accuracy of a classifier without sorting all predictions.
- **Data Analysis:** Finding the N largest sales figures efficiently.
- **Competitive Programming:** Selecting the K-th largest element in O(N) time.
- **Signal Processing:** Finding the K strongest frequency components.

### References

- numpy.partition – https://numpy.org/doc/1.20/reference/generated/numpy.ndarray.partition.html
- numpy.argpartition – https://numpy.org/doc/1.26/reference/generated/numpy.ndarray.argpartition.html
- New functions partition and argpartition – https://numpy.org/doc/1.26/reference/generated/numpy.argpartition.html
- How to get indices of top-K values – https://stackoverflow.com

---

## Core Concept 5: Sub-Array Sampling with Random Selection

### Definitions

**Core Definition:** `Generator.choice()` generates a random sample from a given array, with options for sampling with or without replacement, weighted probabilities, and axis-specific row sampling.

**Technical Definition:** `numpy.random.Generator.choice(a, size=None, replace=True, p=None, axis=0, shuffle=True)` generates a random sample from a given array. If `a` is an int, the sample is drawn from `np.arange(a)`. The `size` parameter controls the output shape. `replace` controls whether sampling is with or without replacement. `p` specifies probabilities for each element. `axis` selects the axis along which to sample (for multidimensional arrays). `shuffle` controls whether the sample is shuffled when sampling without replacement.

**Beginner-Friendly Explanation:** `Generator.choice` randomly picks items from an array. You can pick with or without replacement, give different items different probabilities of being picked, and even sample entire rows from a 2-D array.

### Purposes

- To generate random samples for simulations and bootstrapping.
- To implement random train/test splits.
- To sample batches from a dataset.
- To perform weighted random selection.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
rng = np.random.default_rng()
rng.choice(a, size=None, replace=True, p=None, axis=0, shuffle=True)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `a` | 1-D array-like or int. If int, sample from `np.arange(a)`. |
| `size` | Output shape. Default `None` (single value). |
| `replace` | `True` (default) for sampling with replacement. |
| `p` | Probabilities associated with each entry in `a`. |
| `axis` | Axis along which to sample (for multidimensional `a`). Default 0. |
| `shuffle` | Whether to shuffle the sample when sampling without replacement. |

**Syntax Rules:**

- If `a` is an int, the sample is drawn from `np.arange(a)`.
- `p` must sum to 1 and have the same length as `a`.
- When `replace=False`, the sample size cannot exceed the population size.
- `axis` allows sampling rows from a 2-D array.

**Constraints and Limitations:**

- `np.random.choice` is legacy; new code should use `Generator.choice`.
- Sampling random rows from a 2-D array is not possible with the legacy `np.random.choice` but is possible with `Generator.choice` through its `axis` keyword.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Sampling With and Without Replacement**

```python
import numpy as np

# Step 2: Create a Generator with a fixed seed.
rng = np.random.default_rng(seed=42)

# Step 3: Sample with replacement.
with_repl = rng.choice(5, 3, replace=True)
print("With replacement (size=3):", with_repl)

# Step 4: Sample without replacement.
without_repl = rng.choice(5, 3, replace=False)
print("Without replacement (size=3):", without_repl)

# Step 5: Weighted sampling.
weighted = rng.choice(5, 10, p=[0.1, 0.0, 0.3, 0.6, 0.0])
print("Weighted sample:", weighted)

# Step 6: Sample rows from a 2-D array.
data = np.array([[1, 2], [3, 4], [5, 6], [7, 8]])
row_sample = rng.choice(data, 2, replace=False)
print("\n2-D array row sample:\n", row_sample)
```

**Expected Output:**

```
With replacement (size=3): [0 3 4]
Without replacement (size=3): [3 1 0]
Weighted sample: [3 3 3 2 3 3 3 2 3 3]

2-D array row sample:
 [[5 6]
 [1 2]]
```

**Why This Result Occurs:** With replacement, the same index can be drawn multiple times. Without replacement, each index is drawn at most once. The weighted sample follows the specified probabilities (index 3 has 60% probability). The 2-D example samples entire rows.

### Real-World Cases

- **Train/Test Split:** Sampling indices for training and testing sets.
- **Bootstrap:** Sampling with replacement for statistical inference.
- **Batch Sampling:** Drawing random mini-batches from a dataset.
- **Monte Carlo:** Generating random inputs for simulation.

### References

- numpy.random.choice – https://numpy.org/doc/1.26/reference/random/generated/numpy.random.choice.html
- numpy.random.Generator.choice – https://numpy.org/doc/1.25/reference/random/generated/numpy.random.Generator.choice.html
- NumPy-Discussion: Efficient vectorized sampling without replacement – https://mail.python.org

---

## Comparison Summary

| Function | Purpose | Return Type | Key Parameter |
|----------|---------|-------------|---------------|
| `np.select` | Multi-condition branching | ndarray | `default` |
| `np.choose` | Index-driven lookup | ndarray | `mode` |
| Boolean indexing | Conditional filtering | Copy (1-D) | `&`, `\|`, `~` |
| Fancy indexing | Arbitrary index selection | Copy | Integer array |
| `np.partition` | Partial sort for top-K | Partitioned copy | `kth` |
| `np.argpartition` | Indices for partial sort | Index array | `kth` |
| `Generator.choice` | Random sampling | Sample array | `replace`, `p`, `axis` |

---

## Summary of Deprecated and Version-Specific Features

| Feature | Status | Recommendation |
|---------|--------|----------------|
| `np.random.choice` | Legacy | Use `Generator.choice` in new code |
| `mode='raise'` in `np.choose` | Default but always buffered | Use `'wrap'` or `'clip'` for performance |
| `np.partition` with sequence `kth` | Available since NumPy 1.8 | Use for multiple order statistics |
| `np.select` with `default=0` | Default | Specify explicitly for clarity |

---

## Consolidated Reference List

- numpy.select – https://docs.scipy.org/doc/numpy-1.17.0/reference/generated/numpy.select.html
- numpy.choose – https://numpy.org/doc/2.0/reference/generated/numpy.choose.html
- numpy.ndarray.choose – https://numpy.org/doc/2.0/reference/generated/numpy.ndarray.choose.html
- numpy.partition – https://numpy.org/doc/1.20/reference/generated/numpy.ndarray.partition.html
- numpy.argpartition – https://numpy.org/doc/1.26/reference/generated/numpy.ndarray.argpartition.html
- numpy.random.choice – https://numpy.org/doc/1.26/reference/random/generated/numpy.random.choice.html
- numpy.random.Generator.choice – https://numpy.org/doc/1.25/reference/random/generated/numpy.random.Generator.choice.html
- Boolean array indexing – https://numpy.org/doc/stable/user/basics.indexing.html#boolean-array-indexing
- Advanced NumPy Indexing: A Comprehensive Guide – https://raw.githubusercontent.com
- NEP 21 – Advanced indexing – https://chromium.journaldev.googlesource.com
- New functions partition and argpartition – https://numpy.org/doc/1.26/reference/generated/numpy.argpartition.html
- Conditional Operations — np.where, np.select, np.clip – https://raw.githubusercontent.com