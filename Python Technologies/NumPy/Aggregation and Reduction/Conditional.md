# NumPy Conditional Aggregation: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Conditional aggregation is the practice of applying reduction operations (sum, count, mean, etc.) to only those array elements that satisfy one or more logical conditions, typically expressed as boolean masks.

**Technical Definition:** Conditional aggregation combines NumPy's vectorized comparison operators, boolean mask generation, and reduction functions. A condition applied to an array produces a boolean array (mask) of the same shape. This mask can then be used in three primary ways: (1) directly as the input to a reduction function (since `True` evaluates to 1 and `False` to 0), (2) as a boolean index to filter the array before reduction, or (3) in conjunction with `np.where()` to construct conditional value arrays for partitioned aggregation.

**Beginner-Friendly Explanation:** Sometimes you don't want to summarize all your data—you only want to summarize the parts that meet certain criteria. For example, "What is the total sales for the East region?" or "How many students scored above 90?" NumPy lets you answer these questions efficiently using boolean masks and conditional reductions, all without writing loops.

### Key Characteristics

- **Vectorized Execution:** All operations run in compiled C loops, avoiding Python-level iteration.
- **Boolean Mask Foundation:** Conditions produce boolean arrays that serve as masks.
- **Dual Counting Mechanism:** Summing a boolean mask counts `True` values (since `True` = 1, `False` = 0).
- **Broadcasting Support:** Masks broadcast against arrays for element-wise conditional selection.
- **No Short-Circuiting:** `np.any` and `np.all` do not short-circuit like Python's built-in `any`/`all`; they evaluate all elements.
- **Composability:** Multiple conditions can be combined with `&`, `|`, and `~`.

### Prerequisites

- NumPy installed (`pip install numpy`).
- Understanding of boolean arrays and comparison operations.
- Familiarity with the `import numpy as np` convention.
- Conceptual understanding of broadcasting and reduction axes.

### Related Programming Areas

- **Data Analysis:** Computing conditional statistics (e.g., average sales for a region).
- **Machine Learning:** Computing accuracy, precision, and recall from prediction masks.
- **Financial Modeling:** Aggregating transactions that meet specific criteria.
- **Quality Control:** Counting defective items and computing conditional averages.

### Core Concepts / Features

The following concepts are explored in detail: (1) `any()` & `all()`, (2) Boolean masks & vectorized filtering, (3) Conditional sums & counts, and (4) Partitioned aggregations with `np.where()`.

---

## Core Concept 1: any() & all()

### Definitions

**Core Definition:** `np.any()` returns `True` if at least one element along an axis evaluates to `True`; `np.all()` returns `True` only if every element evaluates to `True`.

**Technical Definition:** `numpy.any(a, axis=None, out=None, keepdims=<no value>, *, where=<no value>)` performs a logical OR reduction. `numpy.all()` performs a logical AND reduction. Both return a single boolean scalar if `axis=None`, otherwise an array of booleans. NaN, positive infinity, and negative infinity evaluate to `True` because they are not equal to zero.

**Beginner-Friendly Explanation:** `np.any` asks "is there at least one True here?" and `np.all` asks "are all of them True?" They collapse a boolean array into a single answer (or an answer per axis).

### Purposes

- To validate that all elements satisfy a condition.
- To detect whether any element satisfies a condition.
- To reduce boolean masks to scalar decisions for conditional logic.
- To implement existence checks and universal quantification over arrays.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.any(a, axis=None, out=None, keepdims=<no value>, *, where=<no value>)
np.all(a, axis=None, out=None, keepdims=<no value>, *, where=<no value>)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `a` | Input array or object convertible to an array. |
| `axis` | Axis or axes along which the reduction is performed. Default `None` reduces all. |
| `out` | Optional output array. |
| `keepdims` | If `True`, reduced axes are left as size-1 dimensions. |
| `where` | Elements to include in the check. |

**Syntax Rules:**

- `axis=None` performs the reduction over all dimensions, returning a scalar boolean.
- `axis` can be an integer or a tuple of integers.
- NaN, Inf, and -Inf evaluate to `True` because they are not zero.

**Constraints and Limitations:**

- NumPy's `any` and `all` do not short-circuit like Python's built-in versions; they evaluate all elements even if the result is determined early.
- For object arrays, behavior may differ in older NumPy versions.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Using np.any() and np.all() on Boolean Arrays**

```python
import numpy as np

# Step 2: Create a boolean mask.
mask = np.array([True, True, False, True])
print("Mask:", mask)

# Step 3: Check if any element is True.
print("np.any(mask):", np.any(mask))
print("  (True because at least one element is True)")

# Step 4: Check if all elements are True.
print("np.all(mask):", np.all(mask))
print("  (False because one element is False)")

# Step 5: Apply along an axis on a 2-D mask.
mask_2d = np.array([[True, False],
                     [True, True]])
print("\n2-D mask:\n", mask_2d)
print("np.any(mask_2d, axis=0):", np.any(mask_2d, axis=0))
print("np.all(mask_2d, axis=1):", np.all(mask_2d, axis=1))
```

**Expected Output:**

```
Mask: [ True  True False  True]
np.any(mask): True
  (True because at least one element is True)
np.all(mask): False
  (False because one element is False)

2-D mask:
 [[ True False]
 [ True  True]]
np.any(mask_2d, axis=0): [ True  True]
np.all(mask_2d, axis=1): [False  True]
```

**Why This Result Occurs:** `np.any` returns `True` because at least one element in the mask is `True`. `np.all` returns `False` because the third element is `False`. With `axis=0`, the reduction is performed down each column; with `axis=1`, it is performed across each row.

### Real-World Cases

- **Validation:** `np.all(scores >= 0)` to check that no negative scores exist.
- **Anomaly Detection:** `np.any(np.isnan(data))` to check whether any missing values are present.
- **Quality Control:** `np.all(products['weight'] > 0)` to verify all products have positive weight.

### References

- numpy.any – https://numpy.org/doc/1.26/reference/generated/numpy.any.html
- numpy.all – https://numpy.org/doc/stable/reference/generated/numpy.all.html
- NumPy-Discussion: np.any and np.all short-circuiting – https://mail.python.org/archives/list/numpy-discussion@python.org/thread/2RESYGWSXH6UXMIJA6ZC5BJUZ5DEC3A2/

---

## Core Concept 2: Boolean Masks & Vectorized Filtering

### Definitions

**Core Definition:** A boolean mask is a boolean array that selects elements from an array based on whether each element satisfies a condition. Vectorized filtering uses these masks to extract subsets of data without loops.

**Technical Definition:** When a comparison operator (e.g., `>`, `<`, `==`) is applied to an ndarray, it returns a boolean array of the same shape. This boolean array can be used as an index: `array[mask]` returns a 1-D array containing only the elements where the mask is `True`. The mask can be combined with other masks using `&`, `|`, and `~` to express complex conditions.

**Beginner-Friendly Explanation:** A boolean mask is like a stencil. You create it by asking a question ("which numbers are greater than 5?"), and NumPy gives you a True/False pattern. You can then use that stencil to pick out the matching elements or to compute statistics on them.

### Purposes

- To filter array elements based on one or more conditions.
- To extract subsets of data for further analysis.
- To combine multiple conditions into a single selection criterion.
- To prepare data for conditional aggregation.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
mask = array > threshold
selected = array[mask]
combined_mask = (condition1) & (condition2)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `mask` | Boolean array of the same shape as `array`. |
| `array[mask]` | 1-D array of elements where `mask` is `True`. |
| `&`, `\|`, `~` | Element-wise logical AND, OR, NOT for combining masks. |
| Parentheses | Required around each comparison when combining. |

**Syntax Rules:**

- The mask must be broadcastable to the shape of the array.
- Boolean indexing always returns a copy, not a view.
- Parentheses are required because `&` has higher precedence than comparison operators.

**Constraints and Limitations:**

- Boolean masks always produce a 1-D result when used for reading, regardless of the original array's dimensionality.
- Combining masks with Python's `and`/`or` raises `ValueError`.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Basic Boolean Mask Filtering**

```python
import numpy as np

# Step 2: Create an array of values.
x = np.array([1, 5, 3, 8, 2, 7, 4, 9, 6])
print("Array:", x)

# Step 3: Create a single-condition mask.
mask_gt_4 = x > 4
print("x > 4:", mask_gt_4)
print("x[x > 4]:", x[mask_gt_4])

# Step 4: Combine two conditions.
combined = (x > 3) & (x < 8)
print("\n(x > 3) & (x < 8):", combined)
print("x[(x > 3) & (x < 8)]:", x[combined])

# Step 5: Use OR and NOT.
either = (x < 3) | (x > 7)
print("\n(x < 3) | (x > 7):", either)
print("x[(x < 3) | (x > 7)]:", x[either])
print("~(x > 4):", ~mask_gt_4)
```

**Expected Output:**

```
Array: [1 5 3 8 2 7 4 9 6]
x > 4: [False  True False  True False  True False  True  True]
x[x > 4]: [5 8 7 9 6]

(x > 3) & (x < 8): [False  True False False False  True  True False  True]
x[(x > 3) & (x < 8)]: [5 7 4 6]

(x < 3) | (x > 7): [ True False False  True  True False False  True False]
x[(x < 3) | (x > 7)]: [1 8 2 9]
~(x > 4): [ True False  True False  True False  True False False]
```

**Why This Result Occurs:** `x > 4` creates a mask where `True` indicates values greater than 4. `x[mask_gt_4]` extracts those values. The combined mask `(x > 3) & (x < 8)` selects values between 3 and 8 exclusive. The OR mask selects values outside the range [3, 7].

### Real-World Cases

- **Data Cleaning:** `data = data[~np.isnan(data)]` removes NaN values.
- **Threshold Filtering:** `high_scores = scores[scores >= 90]` extracts A+ grades.
- **Multi-Condition Selection:** `eligible = customers[(customers['age'] > 18) & (customers['income'] > 50000)]`.

### References

- Boolean array indexing – https://numpy.org/doc/stable/user/basics.indexing.html#boolean-array-indexing
- Masking and aggregation – https://csci4050u.science.ontariotechu.ca

---

## Core Concept 3: Conditional Sums & Counts

### Definitions

**Core Definition:** Conditional sums and counts use reduction functions (`np.sum`, `np.count_nonzero`) on boolean masks to compute totals and frequencies of elements that meet a condition.

**Technical Definition:** When a reduction is applied to a boolean array, NumPy treats `True` as 1 and `False` as 0. Thus, `np.sum(mask)` returns the number of `True` values. `np.count_nonzero(mask)` achieves the same result and is often preferred for counting. For conditional sums of values (not just counts), the mask is used as a boolean index: `np.sum(array[mask])`.

**Beginner-Friendly Explanation:** To count how many numbers are greater than 5, you create a mask (True/False array) and sum it—NumPy counts the Trues. To sum the actual values that are greater than 5, you filter the array with the mask and then sum the filtered values.

### Purposes

- To count occurrences of a condition without loops.
- To compute sums, means, and other statistics on filtered subsets.
- To implement frequency analysis and histogram-like counting.
- To compute conditional totals for reporting and analysis.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
count = np.count_nonzero(mask)
count = np.sum(mask)
conditional_sum = np.sum(array[mask])
conditional_sum = np.sum(array, where=mask)
```

**Component Breakdown:**

| Function | Description |
|----------|-------------|
| `np.count_nonzero(mask)` | Counts `True` values in the mask. |
| `np.sum(mask)` | Sums the mask (True=1, False=0). |
| `np.sum(array[mask])` | Sums the values selected by the mask. |
| `np.sum(array, where=mask)` | Sums only where mask is `True` (NumPy ≥ 1.17). |

**Syntax Rules:**

- `np.count_nonzero` is generally faster than `np.sum` for counting because it skips unnecessary arithmetic.
- Boolean indexing creates a copy; the `where=` parameter avoids this copy.
- Both approaches produce identical results for counting.

**Constraints and Limitations:**

- Boolean indexing allocates a temporary array, which can be memory-intensive for large arrays.
- The `where=` parameter in `np.sum` is not supported in NumPy versions before 1.17.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Counting and Summing Conditionally**

```python
import numpy as np

# Step 2: Create an array of values.
x = np.array([1, 5, 3, 8, 2, 7, 4, 9, 6])
print("Array:", x)

# Step 3: Create a mask.
mask = x > 4
print("Mask (x > 4):", mask)

# Step 4: Count elements meeting the condition.
print("np.count_nonzero(mask):", np.count_nonzero(mask))
print("np.sum(mask):", np.sum(mask))

# Step 5: Sum the values meeting the condition.
print("np.sum(x[mask]):", np.sum(x[mask]))
print("np.sum(x, where=mask):", np.sum(x, where=mask))

# Step 6: Count along an axis in a 2-D array.
m = np.array([[1, 5, 3],
              [8, 2, 7]])
mask_2d = m > 4
print("\n2-D mask:\n", mask_2d)
print("Count per row:", np.count_nonzero(mask_2d, axis=1))
```

**Expected Output:**

```
Array: [1 5 3 8 2 7 4 9 6]
Mask (x > 4): [False  True False  True False  True False  True  True]
np.count_nonzero(mask): 5
np.sum(mask): 5
np.sum(x[mask]): 35
np.sum(x, where=mask): 35

2-D mask:
 [[False  True False]
 [ True False  True]]
Count per row: [1 2]
```

**Why This Result Occurs:** The mask `x > 4` has five `True` values, so `count_nonzero` and `sum` both return 5. The sum of the selected values (5+8+7+9+6) is 35. With `axis=1` on the 2-D mask, each row's `True` count is returned.

### Real-World Cases

- **Accuracy Calculation:** `np.count_nonzero(predictions == labels) / len(labels)`.
- **Conditional Revenue:** `np.sum(revenue[region == 'East'])` for regional totals.
- **Quality Control:** `np.count_nonzero(defects > threshold)` to count defective items.

### References

- numpy.count_nonzero – https://numpy.org/doc/stable/reference/generated/numpy.count_nonzero.html
- How to Easily Count Element Occurrences in NumPy Arrays – https://scales.arabpsychology.com/stats/how-to-count-occurrences-of-elements-in-numpy/
- numpy.sum – https://numpy.org/doc/stable/reference/generated/numpy.sum.html

---

## Core Concept 4: Partitioned Aggregations with np.where()

### Definitions

**Core Definition:** Partitioned aggregation uses `np.where()` to construct conditional value arrays, enabling aggregation of different subsets of data with different rules in a single operation.

**Technical Definition:** `numpy.where(condition, x, y)` returns elements chosen from `x` or `y` depending on `condition`. By nesting `np.where` calls or using it to assign group-specific values, partitioned aggregation computes per-group statistics without explicit group-by loops. This is often combined with `np.sum`, `np.mean`, or other reductions to compute conditional totals or averages.

**Beginner-Friendly Explanation:** `np.where` is like a vectorized if-else. It lets you say "if this condition is true, use this value; otherwise use that value" for every element at once. You can use it to assign different weights or values to different groups, then aggregate the result.

### Purposes

- To implement multi-condition partitioned aggregation without loops.
- To assign different aggregation rules to different subsets of data.
- To compute weighted averages for different groups.
- To replace SQL-style CASE WHEN logic in NumPy.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
result = np.where(condition, x, y)
result = np.sum(np.where(condition, values, 0))
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `condition` | Boolean array. |
| `x` | Values to use where `condition` is `True`. |
| `y` | Values to use where `condition` is `False`. |
| Nested `np.where` | Chains multiple conditions for multi-way branching. |

**Syntax Rules:**

- `condition`, `x`, and `y` must be broadcastable to a common shape.
- Both `x` and `y` are evaluated eagerly, even if only one branch is used.
- `np.where` can be nested for multi-condition logic.

**Constraints and Limitations:**

- Deeply nested `np.where` becomes hard to read; use `np.select` for 3+ conditions.
- Both branches are computed, which may be inefficient for expensive operations.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Partitioned Aggregation with np.where**

```python
import numpy as np

# Step 2: Create data arrays.
sales = np.array([100, 250, 80, 320, 150, 400, 90])
region = np.array(['East', 'West', 'East', 'West', 'East', 'West', 'East'])
print("Sales:", sales)
print("Region:", region)

# Step 3: Compute total sales for East region only.
east_total = np.sum(np.where(region == 'East', sales, 0))
print("\nEast total (np.where):", east_total)

# Step 4: Compute total sales for West region only.
west_total = np.sum(np.where(region == 'West', sales, 0))
print("West total (np.where):", west_total)

# Step 5: Compare with boolean indexing approach.
print("East total (mask):", np.sum(sales[region == 'East']))
print("West total (mask):", np.sum(sales[region == 'West']))

# Step 6: Multi-condition partitioned aggregation.
# Sum sales for East with sales > 100.
east_high = np.sum(np.where((region == 'East') & (sales > 100), sales, 0))
print("\nEast sales > 100:", east_high)
```

**Expected Output:**

```
Sales: [100 250  80 320 150 400  90]
Region: ['East' 'West' 'East' 'West' 'East' 'West' 'East']

East total (np.where): 420
West total (np.where): 970
East total (mask): 420
West total (mask): 970

East sales > 100: 250
```

**Why This Result Occurs:** `np.where(region == 'East', sales, 0)` produces an array where West sales are replaced with 0, so summing gives the East total. The mask approach produces the same result by filtering. The multi-condition example adds another condition (`sales > 100`) to the selection.

### Real-World Cases

- **Tiered Pricing:** Computing total revenue where different products have different tax rates.
- **Segmented Analysis:** Aggregating metrics per customer segment (e.g., new vs. returning).
- **Conditional Discounts:** Applying different discount rates to different product categories before summing.

### References

- numpy.where – https://numpy.org/doc/stable/reference/generated/numpy.where.html
- np.where conditional aggregation – https://stackoverflow.com/revisions/ee96fd52-d351-4018-98db-a33502814be6/view-source

---

## Comparison Summary

| Concept | Key Functions | Output | Best For |
|---------|--------------|--------|----------|
| `any` / `all` | `np.any`, `np.all` | Boolean scalar/array | Validation, existence checks |
| Boolean masks | `array[mask]` | Filtered 1-D array | Subset extraction |
| Conditional counts | `np.count_nonzero`, `np.sum(mask)` | Integer count | Frequency analysis |
| Conditional sums | `np.sum(array[mask])`, `np.sum(array, where=mask)` | Scalar sum | Filtered totals |
| Partitioned aggregation | `np.where(cond, x, y)` | Conditional value array | Multi-group aggregation |

---

## Summary of Deprecated and Version-Specific Features

| Feature | Status | Recommendation |
|---------|--------|----------------|
| `np.sum(where=...)` | New in NumPy 1.17 | Use for memory-efficient conditional sums |
| Short-circuiting in `np.any`/`np.all` | Not supported | Use Python's `any`/`all` on generators for short-circuit |
| `np.count_nonzero` on boolean masks | Always available | Preferred over `np.sum(mask)` for counting |
| Nested `np.where` | Functional | Use `np.select` for 3+ conditions |

---

## Consolidated Reference List

- numpy.any – https://numpy.org/doc/1.26/reference/generated/numpy.any.html
- numpy.all – https://numpy.org/doc/stable/reference/generated/numpy.all.html
- numpy.count_nonzero – https://numpy.org/doc/stable/reference/generated/numpy.count_nonzero.html
- numpy.where – https://numpy.org/doc/stable/reference/generated/numpy.where.html
- numpy.sum – https://numpy.org/doc/stable/reference/generated/numpy.sum.html
- Boolean array indexing – https://numpy.org/doc/stable/user/basics.indexing.html#boolean-array-indexing
- Masking and aggregation – https://csci4050u.science.ontariotechu.ca
- NumPy-Discussion: np.any and np.all short-circuiting – https://mail.python.org/archives/list/numpy-discussion@python.org/thread/2RESYGWSXH6UXMIJA6ZC5BJUZ5DEC3A2/
- How to Easily Count Element Occurrences in NumPy Arrays – https://scales.arabpsychology.com/stats/how-to-count-occurrences-of-elements-in-numpy/
- np.where conditional aggregation – https://stackoverflow.com/revisions/ee96fd52-d351-4018-98db-a33502814be6/view-source