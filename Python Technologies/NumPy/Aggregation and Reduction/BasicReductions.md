# NumPy Basic Reductions: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** NumPy reductions are functions that collapse an array along one or more axes into a smaller array (or a scalar) by repeatedly applying an associative operation, such as summation or multiplication.

**Technical Definition:** A reduction is implemented through a ufunc's `.reduce()` method, which applies a binary operation cumulatively across an axis. NumPy provides convenience functions like `np.sum`, `np.prod`, `np.mean`, `np.average`, `np.min`, `np.max`, `np.std`, and `np.var` that wrap these reduce operations, along with NaN-safe counterparts (`np.nansum`, `np.nanmean`, etc.) and index-returning functions (`np.argmin`, `np.argmax`).

**Beginner-Friendly Explanation:** A reduction takes a whole array and boils it down to a single value (or a smaller array). For example, `np.sum(arr)` adds up all the numbers, `np.max(arr)` finds the largest one, and `np.mean(arr)` computes the average. You can also reduce along a specific axis—for instance, summing each column of a matrix.

### Key Characteristics

- **Axis Parameter:** All reductions accept `axis` to control which dimension is collapsed. `axis=None` (default) reduces the entire array.
- **dtype Control:** `sum` and `prod` accept a `dtype` parameter to specify the accumulator type, preventing integer overflow.
- **NaN Awareness:** Functions prefixed with `nan` (`nansum`, `nanmean`, etc.) ignore NaN values during computation.
- **keepdims:** When `True`, the reduced axis is retained as a size-1 dimension, preserving broadcast compatibility.
- **Index Returns:** `argmin` and `argmax` return the indices of extreme values, not the values themselves.

### Prerequisites

- NumPy installed (`pip install numpy`).
- Basic Python syntax and understanding of arrays.
- Familiarity with the `import numpy as np` convention.
- Conceptual understanding of array axes and shapes.

### Related Programming Areas

- **Data Analysis:** Computing summary statistics for datasets.
- **Machine Learning:** Loss aggregation, gradient averaging, and model evaluation metrics.
- **Scientific Computing:** Total energy, mass, or probability calculations.
- **Image Processing:** Pixel intensity statistics and histogram computation.

### Core Concepts / Features

The following reductions are explored in detail: (1) `sum()` & `prod()`, (2) `mean()` & `average()`, (3) `min()` & `max()`, (4) `std()` & `var()`, (5) NaN-safe reductions, and (6) extreme location reductions (`argmin` & `argmax`).

---

## Core Concept 1: sum() & prod()

### Definitions

**Core Definition:** `np.sum()` computes the sum of array elements along a specified axis, and `np.prod()` computes their product.

**Technical Definition:** `numpy.sum(a, axis=None, dtype=None, out=None, keepdims=<no value>, initial=<no value>, where=<no value>)` returns the sum of array elements over a given axis. `numpy.prod()` returns the product. Both default to reducing the entire array (flattened) and accept a `dtype` parameter to control the accumulator type. The default dtype for integer inputs of lower precision than the platform integer is the platform integer (signed or unsigned).

**Beginner-Friendly Explanation:** `np.sum` adds up all the numbers, and `np.prod` multiplies them all together. For a matrix, you can sum each column (`axis=0`) or each row (`axis=1`).

### Purposes

- To compute totals and cumulative products.
- To aggregate data along specific dimensions.
- To compute likelihoods as products of probabilities.
- To control accumulator precision via `dtype`.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.sum(a, axis=None, dtype=None, out=None, keepdims=<no value>, initial=<no value>, where=<no value>)
np.prod(a, axis=None, dtype=None, out=None, keepdims=<no value>, initial=<no value>, where=<no value>)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `a` | Input array. |
| `axis` | Axis or axes along which to reduce. `None` (default) reduces all. |
| `dtype` | Type of the accumulator and returned array. |
| `out` | Alternative output array. |
| `keepdims` | If `True`, reduced axes are kept as size-1 dimensions. |
| `initial` | Starting value for the reduction. |
| `where` | Elements to include in the reduction. |

**Syntax Rules:**

- For integer inputs of lower precision than the platform integer, the default accumulator is the platform integer (signed or unsigned).
- Arithmetic is modular on integer overflow; no error is raised.
- The product of an empty array is 1.0; the sum of an empty array is 0.0.

**Constraints and Limitations:**

- Integer overflow wraps silently.
- `initial` and `where` are not supported in older NumPy versions.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Sum and Product with dtype Control**

```python
import numpy as np

# Step 2: Create an array.
a = np.array([1, 2, 3, 4, 5])
print("Array:", a)

# Step 3: Sum and product.
print("np.sum(a):", np.sum(a))
print("np.prod(a):", np.prod(a))

# Step 4: Reduce a 2-D array along axes.
m = np.array([[1, 2, 3], [4, 5, 6]])
print("\nMatrix:\n", m)
print("np.sum(m, axis=0):", np.sum(m, axis=0))
print("np.sum(m, axis=1):", np.sum(m, axis=1))

# Step 5: Control dtype to prevent overflow.
x = np.array([1, 2, 3], dtype=np.uint8)
print("\nnp.prod(x) dtype:", np.prod(x).dtype)
print("np.prod(x, dtype=np.int64):", np.prod(x, dtype=np.int64))
```

**Expected Output:**

```
Array: [1 2 3 4 5]
np.sum(a): 15
np.prod(a): 120

Matrix:
 [[1 2 3]
 [4 5 6]]
np.sum(m, axis=0): [5 7 9]
np.sum(m, axis=1): [ 6 15]

np.prod(x) dtype: uint64
np.prod(x, dtype=np.int64): 6
```

**Why This Result Occurs:** `np.sum` adds all elements. `np.prod` multiplies them. `axis=0` sums each column; `axis=1` sums each row. The `dtype` parameter forces the accumulator to `int64`, preventing unsigned overflow.

### Real-World Cases

- **Total Revenue:** Summing a column of transaction amounts.
- **Compound Probability:** Multiplying independent probabilities.
- **Weighted Sums:** Using `np.sum` with `where` to include only selected elements.

### References

- numpy.sum – https://numpy.org/doc/2.2/numpy-ref.pdf#277#181
- numpy.prod – https://numpy.org/doc/2.2/numpy-ref.pdf#277#181

---

## Core Concept 2: mean() & average()

### Definitions

**Core Definition:** `np.mean()` computes the arithmetic mean (sum divided by count), while `np.average()` computes a weighted average when weights are provided.

**Technical Definition:** `numpy.mean(a, axis=None, dtype=None, out=None, keepdims=<no value>, where=<no value>)` returns the arithmetic mean along the specified axis. `numpy.average(a, axis=None, weights=None, returned=False, *, keepdims=<no value>)` computes the weighted average `sum(a * weights) / sum(weights)`. If `weights=None`, the two functions are equivalent.

**Beginner-Friendly Explanation:** `np.mean` is the familiar average. `np.average` does the same thing but lets you give more importance to some values than others using a `weights` array.

### Purposes

- To compute averages of datasets.
- To compute weighted averages where some values matter more.
- To center data for normalization.
- To aggregate metrics across batches.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.mean(a, axis=None, dtype=None, out=None, keepdims=<no value>, where=<no value>)
np.average(a, axis=None, weights=None, returned=False, *, keepdims=<no value>)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `a` | Input array. |
| `axis` | Axis along which to compute the mean. |
| `weights` | Array of weights (for `average`). Must be same shape as `a` or 1-D. |
| `returned` | If `True`, return `(average, sum_of_weights)`. |

**Syntax Rules:**

- For floating-point input, the mean is computed with the same precision as the input.
- The arithmetic mean is sum divided by the number of elements.
- `average` with `weights=None` is equivalent to `mean`.

**Constraints and Limitations:**

- `np.mean` does not support weights.
- `np.average` requires weights to have the same shape as `a` (or be 1-D).

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Mean vs. Weighted Average**

```python
import numpy as np

# Step 2: Create an array.
data = np.array([10, 20, 30, 40])
print("Data:", data)

# Step 3: Arithmetic mean.
print("np.mean(data):", np.mean(data))

# Step 4: Weighted average with equal weights.
w_equal = np.array([1, 1, 1, 1])
print("np.average(data, weights=w_equal):", np.average(data, weights=w_equal))

# Step 5: Weighted average with unequal weights.
w = np.array([1, 2, 3, 4])
print("np.average(data, weights=w):", np.average(data, weights=w))

# Step 6: Mean along an axis.
m = np.array([[1, 2, 3], [4, 5, 6]])
print("\nMatrix mean (axis=0):", np.mean(m, axis=0))
```

**Expected Output:**

```
Data: [10 20 30 40]
np.mean(data): 25.0
np.average(data, weights=w_equal): 25.0
np.average(data, weights=w): 30.0

Matrix mean (axis=0): [2.5 3.5 4.5]
```

**Why This Result Occurs:** `np.mean` gives (10+20+30+40)/4 = 25. With weights [1,2,3,4], the weighted average is (10·1 + 20·2 + 30·3 + 40·4) / (1+2+3+4) = 300/10 = 30.

### Real-World Cases

- **Grade Point Average:** Weighted by credit hours.
- **Portfolio Return:** Weighted by asset allocation.
- **Batch Normalization:** Computing mean activations across a batch.

### References

- numpy.mean – https://numpy.org/doc/1.22/reference/routines.statistics.html
- numpy.average – https://numpy.org/doc/1.22/reference/routines.statistics.html

---

## Core Concept 3: min() & max()

### Definitions

**Core Definition:** `np.min()` returns the minimum value and `np.max()` returns the maximum value along a specified axis.

**Technical Definition:** `numpy.amin(a, axis=None, out=None, keepdims=<no value>, initial=<no value>, where=<no value>)` returns the minimum along an axis. `numpy.amax()` returns the maximum. `np.min` and `np.max` are aliases for `amin` and `amax`.

**Beginner-Friendly Explanation:** `np.min` finds the smallest number in an array, and `np.max` finds the largest. You can also find the minimum or maximum along each row or column.

### Purposes

- To find extreme values for normalization (min-max scaling).
- To detect outliers.
- To implement clipping bounds.
- To aggregate results across dimensions.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.min(a, axis=None, out=None, keepdims=<no value>, initial=<no value>, where=<no value>)
np.max(a, axis=None, out=None, keepdims=<no value>, initial=<no value>, where=<no value>)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `a` | Input array. |
| `axis` | Axis along which to find extremes. |
| `keepdims` | If `True`, reduced axes are kept as size-1. |
| `where` | Elements to include. |

**Syntax Rules:**

- `np.min` and `np.max` propagate NaN values (return `nan` if any element is NaN).
- Use `np.nanmin` / `np.nanmax` to ignore NaN.
- `initial` provides a starting value for the comparison.

**Constraints and Limitations:**

- Cannot compare complex numbers with `<` or `>`.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Finding Extremes Along Axes**

```python
import numpy as np

# Step 2: Create an array.
m = np.array([[3, 1, 4], [1, 5, 9]])
print("Matrix:\n", m)

# Step 3: Global extremes.
print("np.min(m):", np.min(m))
print("np.max(m):", np.max(m))

# Step 4: Extremes along axes.
print("np.min(m, axis=0):", np.min(m, axis=0))
print("np.max(m, axis=1):", np.max(m, axis=1))

# Step 5: keepdims for broadcasting.
print("np.max(m, axis=1, keepdims=True):\n", np.max(m, axis=1, keepdims=True))
```

**Expected Output:**

```
Matrix:
 [[3 1 4]
 [1 5 9]]
np.min(m): 1
np.max(m): 9
np.min(m, axis=0): [1 1 4]
np.max(m, axis=1): [4 9]
np.max(m, axis=1, keepdims=True):
 [[4]
 [9]]
```

**Why This Result Occurs:** `axis=0` finds the minimum in each column; `axis=1` finds the maximum in each row. `keepdims=True` retains the reduced dimension, enabling broadcasting in subsequent operations.

### Real-World Cases

- **Min-Max Normalization:** `(x - np.min(x)) / (np.max(x) - np.min(x))`.
- **Clipping:** Setting bounds with `np.clip(x, np.min(x), np.max(x))`.
- **Peak Detection:** Finding the maximum signal amplitude.

### References

- numpy.amin – https://numpy.org/doc/stable/reference/generated/numpy.amin.html
- numpy.amax – https://numpy.org/doc/stable/reference/generated/numpy.amax.html

---

## Core Concept 4: std() & var()

### Definitions

**Core Definition:** `np.std()` computes the standard deviation and `np.var()` computes the variance along a specified axis, with the `ddof` parameter controlling the divisor.

**Technical Definition:** `numpy.std(a, axis=None, dtype=None, out=None, ddof=0, keepdims=<no value>, where=<no value>)` returns the standard deviation. The divisor used is `N - ddof`, where `N` is the number of elements. `numpy.var()` returns the variance. The default `ddof=0` provides a maximum likelihood estimate; `ddof=1` provides an unbiased estimator of the variance of a hypothetical infinite population.

**Beginner-Friendly Explanation:** Standard deviation measures how spread out the numbers are around the mean. Variance is the square of that. `ddof` lets you choose between the population formula (divide by N) and the sample formula (divide by N−1).

### Purposes

- To quantify data spread and variability.
- To standardize features (z-score normalization).
- To assess risk in financial returns.
- To compute error bars.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.std(a, axis=None, dtype=None, out=None, ddof=0, keepdims=<no value>, where=<no value>)
np.var(a, axis=None, dtype=None, out=None, ddof=0, keepdims=<no value>, where=<no value>)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `a` | Input array. |
| `ddof` | Delta Degrees of Freedom: divisor is `N - ddof`. |
| `axis` | Axis along which to compute. |
| `dtype` | Type for computation. |

**Syntax Rules:**

- The default `ddof=0` gives the population standard deviation/variance.
- `ddof=1` gives the sample standard deviation/variance.
- The standard deviation is the square root of the variance.

**Constraints and Limitations:**

- For complex numbers, the absolute value is taken before squaring.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Population vs. Sample Standard Deviation**

```python
import numpy as np

# Step 2: Create data.
data = np.array([2, 4, 4, 4, 5, 5, 7, 9])
print("Data:", data)

# Step 3: Population variance (ddof=0).
print("np.var(data, ddof=0):", np.var(data))
print("np.std(data, ddof=0):", np.std(data))

# Step 4: Sample variance (ddof=1).
print("np.var(data, ddof=1):", np.var(data))
print("np.std(data, ddof=1):", np.std(data))
```

**Expected Output:**

```
Data: [2 4 4 4 5 5 7 9]
np.var(data, ddof=0): 4.0
np.std(data, ddof=0): 2.0
np.var(data, ddof=1): 4.571428571428571
np.std(data, ddof=1): 2.1380899352993947
```

**Why This Result Occurs:** With `ddof=0`, the divisor is N=8. With `ddof=1`, the divisor is N−1=7, producing a larger variance and standard deviation.

### Real-World Cases

- **Feature Scaling:** `(X - X.mean(axis=0)) / X.std(axis=0)`.
- **Risk Assessment:** Computing portfolio volatility.
- **Quality Control:** Measuring process consistency.

### References

- numpy.std – https://numpy.org/doc/1.22/reference/routines.statistics.html
- numpy.var – https://numpy.org/doc/1.22/reference/routines.statistics.html

---

## Core Concept 5: NaN-Safe Reductions

### Definitions

**Core Definition:** NaN-safe reductions (`nansum`, `nanmean`, `nanmin`, `nanmax`, `nanstd`, `nanvar`) compute their respective statistics while ignoring NaN values.

**Technical Definition:** These functions replace NaN values with a fill value (typically 0), compute the reduction, and then restore NaN in the output where appropriate. `nansum` returns the sum of non-NaN values; `nanmean` returns the mean of non-NaN values; `nanmin`/`nanmax` return the minimum/maximum non-NaN value; `nanstd`/`nanvar` return the standard deviation/variance of non-NaN values.

**Beginner-Friendly Explanation:** When your data has missing values (NaN), regular reductions return NaN. The `nan`-prefixed versions skip those missing values and compute the result from the remaining data.

### Purposes

- To handle missing data in datasets.
- To compute statistics on incomplete records.
- To avoid NaN propagation in aggregation pipelines.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.nansum(a, axis=None, dtype=None, out=None, keepdims=<no value>)
np.nanmean(a, axis=None, dtype=None, out=None, keepdims=<no value>)
np.nanmin(a, axis=None, out=None, keepdims=<no value>)
np.nanmax(a, axis=None, out=None, keepdims=<no value>)
np.nanstd(a, axis=None, dtype=None, out=None, ddof=0, keepdims=<no value>)
np.nanvar(a, axis=None, dtype=None, out=None, ddof=0, keepdims=<no value>)
```

**Component Breakdown:**

| Function | Ignores NaN | Notes |
|----------|-------------|-------|
| `nansum` | Yes | Sum of non-NaN values. |
| `nanmean` | Yes | Mean of non-NaN values. |
| `nanmin`/`nanmax` | Yes | Min/max of non-NaN values. |
| `nanstd`/`nanvar` | Yes | Std/var of non-NaN values. |

**Syntax Rules:**

- If all values along an axis are NaN, `nanmean` returns `nan` and emits a `RuntimeWarning`.
- The result for an all-NaN slice is NaN.

**Constraints and Limitations:**

- These functions are slower than their non-NaN counterparts due to the extra masking step.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: NaN-Safe Reductions**

```python
import numpy as np

# Step 2: Create an array with NaN.
a = np.array([1.0, 2.0, np.nan, 4.0, 5.0])
print("Array:", a)

# Step 3: Regular sum vs. nansum.
print("np.sum(a):", np.sum(a))
print("np.nansum(a):", np.nansum(a))

# Step 4: Regular mean vs. nanmean.
print("np.mean(a):", np.mean(a))
print("np.nanmean(a):", np.nanmean(a))

# Step 5: NaN-safe min/max.
print("np.nanmin(a):", np.nanmin(a))
print("np.nanmax(a):", np.nanmax(a))
```

**Expected Output:**

```
Array: [ 1.  2. nan  4.  5.]
np.sum(a): nan
np.nansum(a): 12.0
np.mean(a): nan
np.nanmean(a): 3.0
np.nanmin(a): 1.0
np.nanmax(a): 5.0
```

**Why This Result Occurs:** `np.sum` returns NaN because NaN propagates. `np.nansum` ignores the NaN and sums 1+2+4+5 = 12. `np.nanmean` divides by 4 (the count of non-NaN values), yielding 3.0.

### Real-World Cases

- **Sensor Data:** Handling missing readings in time series.
- **Survey Data:** Computing averages with unanswered questions.
- **Financial Data:** Dealing with missing trading days.

### References

- numpy.nansum – https://numpy.org/doc/stable/reference/generated/numpy.nansum.html
- numpy.nanmean – https://numpy.org/doc/stable/reference/generated/numpy.nanmean.html
- numpy.nanmin – https://numpy.org/doc/stable/reference/generated/numpy.nanmin.html
- numpy.nanmax – https://numpy.org/doc/stable/reference/generated/numpy.nanmax.html
- numpy.nanstd – https://numpy.org/doc/stable/reference/generated/numpy.nanstd.html
- numpy.nanvar – https://numpy.org/doc/stable/reference/generated/numpy.nanvar.html

---

## Core Concept 6: Extreme Location Reductions (argmin & argmax)

### Definitions

**Core Definition:** `np.argmin()` returns the indices of the minimum values and `np.argmax()` returns the indices of the maximum values along an axis.

**Technical Definition:** `numpy.argmin(a, axis=None, out=None, *, keepdims=<no value>)` returns the indices of the minimum values along an axis. `numpy.argmax()` returns the indices of the maximum values. If `axis=None`, the index is into the flattened array. Otherwise, it is along the specified axis.

**Beginner-Friendly Explanation:** Instead of giving you the smallest or largest value, `argmin` and `argmax` tell you *where* that value is located. This is useful when you need the position of the extreme value, not just the value itself.

### Purposes

- To find the position of the maximum or minimum in an array.
- To implement one-hot encoding from logits.
- To locate peaks in signals.
- To identify the best or worst performing item in a dataset.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.argmin(a, axis=None, out=None, *, keepdims=<no value>)
np.argmax(a, axis=None, out=None, *, keepdims=<no value>)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `a` | Input array. |
| `axis` | Axis along which to find the index. |
| `keepdims` | If `True`, reduced axes are kept as size-1. |

**Syntax Rules:**

- If `axis=None`, the index is into the flattened array.
- In case of multiple occurrences of the minimum/maximum, the first occurrence is returned.
- `keepdims` is available since NumPy 1.22.0.

**Constraints and Limitations:**

- `argmin`/`argmax` do not ignore NaN; use `nanargmin`/`nanargmax` for NaN-safe versions.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Finding Indices of Extremes**

```python
import numpy as np

# Step 2: Create an array.
a = np.array([10, 3, 25, 7, 25])
print("Array:", a)

# Step 3: Global argmin and argmax.
print("np.argmin(a):", np.argmin(a))
print("np.argmax(a):", np.argmax(a))

# Step 4: Along an axis in 2-D.
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

Matrix:
 [[4 2 3]
 [1 0 3]]
np.argmax(m, axis=0): [0 0 1]
np.argmax(m, axis=1): [0 2]
np.argmax(m, axis=1, keepdims=True):
 [[0]
 [2]]
```

**Why This Result Occurs:** `np.argmin(a)` returns 1 because the minimum value 3 is at index 1. `np.argmax(a)` returns 2 because the first occurrence of the maximum value 25 is at index 2 (even though 25 also appears at index 4). Along `axis=0`, the maximum in each column is found; along `axis=1`, the maximum in each row.

### Real-World Cases

- **Classification:** `np.argmax(logits)` gives the predicted class.
- **Peak Detection:** Finding the index of the maximum signal amplitude.
- **Ranking:** Identifying the best and worst performers.

### References

- numpy.argmin – https://numpy.org/doc/stable/reference/generated/numpy.argmin.html
- numpy.argmax – https://numpy.org/doc/stable/reference/generated/numpy.argmax.html

---

## Comparison Summary

| Function | Returns | NaN-Safe Counterpart | Key Parameter |
|----------|---------|---------------------|---------------|
| `np.sum` | Sum of elements | `np.nansum` | `dtype` |
| `np.prod` | Product of elements | `np.nanprod` | `dtype` |
| `np.mean` | Arithmetic mean | `np.nanmean` | — |
| `np.average` | Weighted average | — | `weights` |
| `np.min` / `np.max` | Extreme values | `np.nanmin` / `np.nanmax` | — |
| `np.std` / `np.var` | Spread | `np.nanstd` / `np.nanvar` | `ddof` |
| `np.argmin` / `np.argmax` | Indices of extremes | `np.nanargmin` / `np.nanargmax` | `axis` |

---

## Summary of Deprecated and Version-Specific Features

| Feature | Status | Recommendation |
|---------|--------|----------------|
| `np.mean`/`np.std`/`np.var` on complex numbers | Documented behavior | Uses absolute value for variance |
| `ddof` default | `0` (population) | Use `ddof=1` for sample statistics |
| `initial` and `where` in reductions | New in NumPy 1.15–1.17 | Available in modern NumPy |
| `keepdims` in `argmin`/`argmax` | New in NumPy 1.22.0 | Available in NumPy ≥ 1.22 |

---

## Consolidated Reference List

- numpy.sum – https://numpy.org/doc/2.2/numpy-ref.pdf#277#181
- numpy.prod – https://numpy.org/doc/2.2/numpy-ref.pdf#277#181
- numpy.mean – https://numpy.org/doc/1.22/reference/routines.statistics.html
- numpy.average – https://numpy.org/doc/1.22/reference/routines.statistics.html
- numpy.std – https://numpy.org/doc/1.22/reference/routines.statistics.html
- numpy.var – https://numpy.org/doc/1.22/reference/routines.statistics.html
- numpy.nansum – https://numpy.org/doc/stable/reference/generated/numpy.nansum.html
- numpy.nanmean – https://numpy.org/doc/stable/reference/generated/numpy.nanmean.html
- numpy.nanmin – https://numpy.org/doc/stable/reference/generated/numpy.nanmin.html
- numpy.nanmax – https://numpy.org/doc/stable/reference/generated/numpy.nanmax.html
- numpy.nanstd – https://numpy.org/doc/stable/reference/generated/numpy.nanstd.html
- numpy.nanvar – https://numpy.org/doc/stable/reference/generated/numpy.nanvar.html
- numpy.argmin – https://numpy.org/doc/stable/reference/generated/numpy.argmin.html
- numpy.argmax – https://numpy.org/doc/stable/reference/generated/numpy.argmax.html
- numpy.amin – https://numpy.org/doc/stable/reference/generated/numpy.amin.html
- numpy.amax – https://numpy.org/doc/stable/reference/generated/numpy.amax.html