# NumPy Cumulative Operations: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Cumulative operations in NumPy are functions that produce a running result at each position of an array, where each output element represents the accumulation of all preceding elements up to and including the current position along a specified axis.

**Technical Definition:** Cumulative operations are implemented through the `.accumulate()` method of universal functions (ufuncs), which returns an array of intermediate results rather than collapsing the array to a single value. NumPy provides convenience wrappers such as `np.cumsum` and `np.cumprod`, and cumulative minimum/maximum via `np.minimum.accumulate` and `np.maximum.accumulate`. Complementary statistical functions like `np.percentile` and `np.quantile` provide order-statistic summaries, while ufunc methods `.reduce()`, `.accumulate()`, and `.reduceat()` offer low-level control over custom accumulations.

**Beginner-Friendly Explanation:** A cumulative operation is like watching a running total. Instead of giving you just the final sum, it gives you the sum after each step. For example, if you have daily sales figures, a cumulative sum tells you the total sales up to each day. NumPy provides similar running operations for products, minimums, and maximums, plus tools for computing percentiles and custom accumulations.

### Key Characteristics

- **Running Results:** Each output element is the accumulation of all elements up to that position along the specified axis.
- **Same Shape:** Unlike reductions, cumulative operations return an array of the same size as the input.
- **Axis-Aware:** The accumulation can be performed along any axis of a multidimensional array.
- **Integer Overflow:** Arithmetic is modular for integer types; no error is raised on overflow.
- **NaN Handling:** The `nan`-prefixed variants ignore NaN values during accumulation.
- **Ufunc Foundation:** All cumulative operations are built on the `.accumulate()` method of ufuncs.

### Prerequisites

- NumPy installed (`pip install numpy`).
- Understanding of array shapes, axes, and reduction operations.
- Familiarity with the `import numpy as np` convention.
- Conceptual understanding of universal functions (ufuncs).

### Related Programming Areas

- **Financial Analysis:** Computing running totals, cumulative returns, and drawdowns.
- **Signal Processing:** Computing cumulative energy, moving averages, and cumulative distributions.
- **Machine Learning:** Cumulative error tracking and learning rate schedules.
- **Scientific Computing:** Cumulative distribution functions, running statistics, and integration.

### Core Concepts / Features

The following concepts are explored in detail: (1) `cumsum()` & `cumprod()`, (2) Cumulative minimum & maximum, (3) Axis-specific tracking, (4) Advanced summary statistics (`percentile()`, `quantile()`, and NaN-safe variants), and (5) Custom accumulations & reductions.

---

## Core Concept 1: cumsum() & cumprod()

### Definitions

**Core Definition:** `np.cumsum()` computes the cumulative sum of array elements along a given axis, and `np.cumprod()` computes the cumulative product.

**Technical Definition:** `numpy.cumsum(a, axis=None, dtype=None, out=None)` returns the cumulative sum of the elements along a given axis. The default (`axis=None`) computes the cumsum over the flattened array. `numpy.cumprod()` returns the cumulative product. For both functions, the result has the same size as the input, and the same shape if `axis` is not `None` or `a` is a 1-D array. Arithmetic is modular when using integer types, and no error is raised on overflow.

**Beginner-Friendly Explanation:** `np.cumsum` gives you a running total: the first element is the first value, the second is the sum of the first two, the third is the sum of the first three, and so on. `np.cumprod` does the same with multiplication. They are like keeping a running tally as you go through a list.

### Purposes

- To compute running totals of sequential data.
- To track cumulative returns or growth over time.
- To implement cumulative distribution functions (CDFs).
- To compute running products for compound growth calculations.
- To prepare data for further analysis that depends on accumulated values.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.cumsum(a, axis=None, dtype=None, out=None)
np.cumprod(a, axis=None, dtype=None, out=None)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `a` | Input array (array_like). |
| `axis` | Axis along which the cumulative sum/product is computed. Default `None` flattens the array. |
| `dtype` | Type of the returned array and accumulator. Defaults to the dtype of `a`, unless `a` has an integer dtype with less precision than the default platform integer, in which case the platform integer is used. |
| `out` | Alternative output array. Must have the same shape and buffer length as the expected output. |

**Syntax Rules:**

- The result has the same size as `a`, and the same shape as `a` if `axis` is not `None` or `a` is a 1-D array.
- `cumsum(a)[-1]` may not equal `sum(a)` for floating-point values because `sum` may use pairwise summation to reduce roundoff error.
- Arithmetic is modular for integer types; no overflow error is raised.

**Constraints and Limitations:**

- Integer overflow wraps silently.
- For floating-point values, cumulative summation may accumulate more roundoff error than a direct `sum`.
- The `axis=None` default flattens the array before accumulation.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Basic cumsum and cumprod**

```python
import numpy as np

# Step 2: Create an array.
a = np.array([1, 2, 3, 4, 5])
print("Original:", a)

# Step 3: Cumulative sum.
cumsum_result = np.cumsum(a)
print("np.cumsum(a):", cumsum_result)
print("  (1, 1+2, 1+2+3, 1+2+3+4, 1+2+3+4+5)")

# Step 4: Cumulative product.
cumprod_result = np.cumprod(a)
print("np.cumprod(a):", cumprod_result)
print("  (1, 1*2, 1*2*3, 1*2*3*4, 1*2*3*4*5)")

# Step 5: 2-D array with axis parameter.
m = np.array([[1, 2, 3], [4, 5, 6]])
print("\nMatrix:\n", m)
print("np.cumsum(m, axis=0):\n", np.cumsum(m, axis=0))
print("np.cumsum(m, axis=1):\n", np.cumsum(m, axis=1))
```

**Expected Output:**

```
Original: [1 2 3 4 5]
np.cumsum(a): [ 1  3  6 10 15]
  (1, 1+2, 1+2+3, 1+2+3+4, 1+2+3+4+5)
np.cumprod(a): [  1   2   6  24 120]
  (1, 1*2, 1*2*3, 1*2*3*4, 1*2*3*4*5)

Matrix:
 [[1 2 3]
 [4 5 6]]
np.cumsum(m, axis=0):
 [[1 2 3]
 [5 7 9]]
np.cumsum(m, axis=1):
 [[ 1  3  6]
 [ 4  9 15]]
```

**Why This Result Occurs:** `np.cumsum(a)` produces running totals: after the first element it is 1, after the second it is 3 (1+2), after the third it is 6 (1+2+3), etc. `np.cumprod(a)` produces running products: 1, 2, 6, 24, 120. With `axis=0`, the accumulation goes down the columns; with `axis=1`, it goes across the rows.

**Example 2: dtype Control and Overflow**

```python
import numpy as np

# Step 2: Create an integer array with small dtype.
x = np.array([100, 200, 300], dtype=np.int8)
print("Array dtype:", x.dtype)

# Step 3: Cumsum with default dtype (platform integer).
cumsum_default = np.cumsum(x)
print("Default cumsum dtype:", cumsum_default.dtype)
print("Result:", cumsum_default)

# Step 4: Cumprod with explicit dtype.
cumprod_int64 = np.cumprod(x, dtype=np.int64)
print("Cumprod with int64:", cumprod_int64)
```

**Expected Output:**

```
Array dtype: int8
Default cumsum dtype: int64
Result: [100 300 600]
Cumprod with int64: [100 20000 6000000]
```

**Why This Result Occurs:** For `int8` input, NumPy uses the platform integer (`int64`) as the accumulator to prevent overflow. The `dtype` parameter explicitly forces the accumulator type for `cumprod`, ensuring the product fits.

### Real-World Cases

- **Financial Analysis:** Computing cumulative returns from a series of daily returns: `np.cumprod(1 + returns) - 1`.
- **Sales Tracking:** Running total of sales throughout a month: `np.cumsum(daily_sales)`.
- **Probability:** Computing cumulative distribution from a probability mass function: `np.cumsum(pmf)`.
- **Signal Processing:** Cumulative energy of a signal: `np.cumsum(signal ** 2)`.

### References

- numpy.cumsum – https://numpy.org/doc/2.3/reference/generated/numpy.cumsum.html
- numpy.cumprod – https://numpy.org/doc/stable/reference/generated/numpy.cumprod.html
- numpy.ufunc.accumulate – https://numpy.org/doc/stable/reference/generated/numpy.ufunc.accumulate.html

---

## Core Concept 2: Cumulative Minimum & Maximum

### Definitions

**Core Definition:** Cumulative minimum and maximum track the running minimum or maximum value encountered up to each position in an array, using `np.minimum.accumulate` and `np.maximum.accumulate`.

**Technical Definition:** NumPy does not provide dedicated `cummin` or `cummax` functions. Instead, cumulative minimum and maximum are obtained by calling the `.accumulate()` method on the `np.minimum` and `np.maximum` ufuncs: `np.minimum.accumulate(y, axis)` and `np.maximum.accumulate(y, axis)`. For any two-argument NumPy universal function, its `.accumulate()` method is the cumulative version of that function.

**Beginner-Friendly Explanation:** Cumulative minimum is like keeping track of the lowest price you've seen so far. At each point, it tells you the smallest value encountered up to that position. Cumulative maximum does the same for the largest value.

### Purposes

- To track running minimums and maximums over sequences.
- To compute drawdowns in financial time series.
- To find the highest or lowest value seen so far.
- To implement running bounds for threshold detection.
- To compute cumulative extrema for signal processing.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.minimum.accumulate(array, axis=0, dtype=None, out=None)
np.maximum.accumulate(array, axis=0, dtype=None, out=None)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `array` | Input array. |
| `axis` | Axis along which to accumulate. Default 0. |
| `dtype` | Data type of the accumulator. |
| `out` | Alternative output array. |

**Syntax Rules:**

- `np.minimum.accumulate` returns the running minimum at each position.
- `np.maximum.accumulate` returns the running maximum at each position.
- The result has the same shape as the input.

**Constraints and Limitations:**

- These are not standalone functions; they are methods of the `np.minimum` and `np.maximum` ufuncs.
- NaN values propagate; use `np.fmin.accumulate` for NaN-ignoring behavior.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Cumulative Minimum and Maximum**

```python
import numpy as np

# Step 2: Create an array of values.
a = np.array([5, 3, 8, 1, 9, 2, 7])
print("Original:", a)

# Step 3: Cumulative minimum.
cummin = np.minimum.accumulate(a)
print("np.minimum.accumulate(a):", cummin)
print("  (running minimum: 5, 3, 3, 1, 1, 1, 1)")

# Step 4: Cumulative maximum.
cummax = np.maximum.accumulate(a)
print("np.maximum.accumulate(a):", cummax)
print("  (running maximum: 5, 5, 8, 8, 9, 9, 9)")

# Step 5: 2-D array along axis 1.
m = np.array([[3, 1, 4], [1, 5, 9]])
print("\nMatrix:\n", m)
print("Cumulative min (axis=1):\n", np.minimum.accumulate(m, axis=1))
print("Cumulative max (axis=1):\n", np.maximum.accumulate(m, axis=1))
```

**Expected Output:**

```
Original: [5 3 8 1 9 2 7]
np.minimum.accumulate(a): [5 3 3 1 1 1 1]
  (running minimum: 5, 3, 3, 1, 1, 1, 1)
np.maximum.accumulate(a): [5 5 8 8 9 9 9]
  (running maximum: 5, 5, 8, 8, 9, 9, 9)

Matrix:
 [[3 1 4]
 [1 5 9]]
Cumulative min (axis=1):
 [[3 1 1]
 [1 1 1]]
Cumulative max (axis=1):
 [[3 3 4]
 [1 5 9]]
```

**Why This Result Occurs:** `np.minimum.accumulate` keeps the smallest value seen so far: after 5 it is 5, after 3 it drops to 3, after 8 it stays 3, after 1 it drops to 1, and so on. `np.maximum.accumulate` keeps the largest value seen so far.

### Real-World Cases

- **Financial Drawdown:** Computing the maximum drawdown of an investment portfolio: `(price / np.maximum.accumulate(price)) - 1`.
- **Temperature Tracking:** Finding the lowest temperature recorded up to each day.
- **Signal Processing:** Detecting the running peak of a signal.

### References

- numpy.minimum.accumulate – https://numpy.org/doc/stable/reference/generated/numpy.ufunc.accumulate.html
- Calculating cumulative minimum with numpy arrays – https://stackoverflow.com/questions/23519611
- Summarizing data (cummin/cummax) – https://github.com

---

## Core Concept 3: Axis-Specific Tracking

### Definitions

**Core Definition:** Axis-specific tracking means applying cumulative operations along a particular dimension of a multidimensional array, allowing independent running calculations along rows, columns, or higher-dimensional slices.

**Technical Definition:** All cumulative functions and ufunc methods accept an `axis` parameter that specifies the dimension along which the accumulation is performed. For a 2-D array of shape `(M, N)`, `axis=0` accumulates down the rows (producing results for each column), while `axis=1` accumulates across the columns (producing results for each row). For higher-dimensional arrays, the same principle applies to any chosen axis.

**Beginner-Friendly Explanation:** If you have a table of data, you can compute running totals down each column (`axis=0`) or across each row (`axis=1`). For a 3-D array, you can choose any of the three directions to accumulate along.

### Purposes

- To compute running statistics along a specific dimension.
- To produce per-column or per-row cumulative results.
- To apply cumulative operations to time-series data organized by rows.
- To handle multidimensional data where accumulation direction matters.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.cumsum(a, axis=<int>)
np.cumprod(a, axis=<int>)
np.minimum.accumulate(a, axis=<int>)
np.add.accumulate(a, axis=<int>)
```

**Component Breakdown:**

| `axis` value | Effect |
|--------------|--------|
| `0` | Accumulate down the rows (per column). |
| `1` | Accumulate across the columns (per row). |
| `-1` | Accumulate along the last axis. |
| `None` | Flatten and accumulate. |

**Syntax Rules:**

- `axis` must be a valid axis index in `[-ndim, ndim-1]`.
- The result has the same shape as the input.
- The accumulation is independent along each slice perpendicular to the chosen axis.

**Constraints and Limitations:**

- Only one axis can be specified at a time for cumulative operations (unlike reductions, which accept tuples).
- `axis=None` flattens the array.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Axis-Specific Cumulative Sum on a 3-D Array**

```python
import numpy as np

# Step 2: Create a 3-D array.
a = np.arange(24).reshape(2, 3, 4)
print("Shape:", a.shape)

# Step 3: Cumsum along axis 0.
print("cumsum(axis=0) shape:", np.cumsum(a, axis=0).shape)

# Step 4: Cumsum along axis 1.
print("cumsum(axis=1) shape:", np.cumsum(a, axis=1).shape)

# Step 5: Cumsum along axis 2.
print("cumsum(axis=2) shape:", np.cumsum(a, axis=2).shape)

# Step 6: Show the result for axis=2 on the first layer.
print("\nFirst layer of cumsum(axis=2):\n", np.cumsum(a, axis=2)[0])
```

**Expected Output:**

```
Shape: (2, 3, 4)
cumsum(axis=0) shape: (2, 3, 4)
cumsum(axis=1) shape: (2, 3, 4)
cumsum(axis=2) shape: (2, 3, 4)

First layer of cumsum(axis=2):
 [[ 0  1  3  6]
 [ 4  9 15 22]
 [ 8 17 27 38]]
```

**Why This Result Occurs:** The cumulative sum is computed along the specified axis while all other dimensions remain intact. For `axis=2`, each row of 4 elements is accumulated independently, producing running sums within each row of the first layer.

### Real-World Cases

- **Time Series Analysis:** Computing cumulative returns along the time axis: `np.cumsum(returns, axis=0)`.
- **Image Processing:** Cumulative histogram along the intensity axis.
- **Batch Processing:** Accumulating results across batches: `np.cumsum(batch_outputs, axis=0)`.

### References

- numpy.cumsum – https://numpy.org/doc/2.3/reference/generated/numpy.cumsum.html
- numpy.ufunc.accumulate – https://numpy.org/doc/stable/reference/generated/numpy.ufunc.accumulate.html

---

## Core Concept 4: Advanced Summary Statistics

### Definitions

**Core Definition:** `np.percentile()` and `np.quantile()` compute order statistics that divide a dataset into intervals, while `np.nanpercentile()` and `np.nanquantile()` provide the same computations while ignoring NaN values.

**Technical Definition:** `numpy.percentile(a, q, axis=None, ...)` computes the q-th percentile of the data along the specified axis, where q is in the range [0, 100]. `numpy.quantile(a, q, axis=None, ...)` is equivalent but uses q in the range [0, 1]. Both support multiple `method` options for estimating the quantile, including 'linear' (default), 'lower', 'higher', 'midpoint', 'nearest', and others. The `nan`-prefixed variants ignore NaN values during computation. Since NumPy 1.22.0, the `method` parameter replaced the deprecated `interpolation` parameter.

**Beginner-Friendly Explanation:** Percentiles and quantiles tell you where a value sits relative to the rest of the data. The 50th percentile is the median, the 25th percentile is the first quartile, and so on. These functions help you understand the distribution of your data, and the NaN-safe versions let you work with datasets that have missing values.

### Purposes

- To compute median, quartiles, and other order statistics.
- To understand the distribution of data.
- To detect outliers using percentile thresholds.
- To compute confidence intervals from bootstrap samples.
- To handle missing data without removing it.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.percentile(a, q, axis=None, out=None, overwrite_input=False, method='linear', keepdims=False, *, weights=None)
np.quantile(a, q, axis=None, out=None, overwrite_input=False, method='linear', keepdims=False, *, weights=None)
np.nanpercentile(a, q, axis=None, out=None, overwrite_input=False, method='linear', keepdims=False, *, weights=None)
np.nanquantile(a, q, axis=None, out=None, overwrite_input=False, method='linear', keepdims=False, *, weights=None)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `a` | Input array. |
| `q` | Percentile(s) (0–100) or quantile(s) (0–1) to compute. |
| `axis` | Axis along which to compute. Default flattens the array. |
| `method` | Estimation method: 'linear' (default), 'lower', 'higher', 'midpoint', 'nearest', 'inverted_cdf', etc. |
| `keepdims` | If `True`, reduced axes are retained as size-1 dimensions. |
| `weights` | Optional weights for the data (only supported by 'inverted_cdf'). |

**Syntax Rules:**

- `q` can be a scalar or a sequence of values.
- If `q` is a single value and `axis=None`, the result is a scalar.
- If multiple quantiles are given, the first axis of the result corresponds to the quantiles.
- `np.percentile` is equivalent to `np.quantile` with q scaled by 100.
- `np.nanpercentile` and `np.nanquantile` ignore NaN values.

**Constraints and Limitations:**

- The `interpolation` parameter is deprecated since NumPy 1.22.0; use `method` instead.
- Weights are only supported with `method='inverted_cdf'`.
- Performance improved significantly in NumPy 1.22.0, with `percentile` implemented in terms of `np.partition`, reducing complexity from O(n log n) to O(n).

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Percentiles and Quantiles**

```python
import numpy as np

# Step 2: Create a dataset.
data = np.array([1, 2, 3, 4, 5, 6, 7, 8, 9, 10])
print("Data:", data)

# Step 3: Compute percentiles.
print("25th percentile:", np.percentile(data, 25))
print("50th percentile (median):", np.percentile(data, 50))
print("75th percentile:", np.percentile(data, 75))

# Step 4: Compute quantiles (equivalent).
print("\n0.25 quantile:", np.quantile(data, 0.25))
print("0.50 quantile:", np.quantile(data, 0.50))
print("0.75 quantile:", np.quantile(data, 0.75))

# Step 5: Multiple quantiles at once.
print("\nMultiple quantiles:", np.quantile(data, [0.25, 0.5, 0.75]))

# Step 6: NaN-safe percentiles.
data_nan = np.array([1, 2, np.nan, 4, 5, np.nan, 7, 8, 9, 10])
print("\nWith NaN:", data_nan)
print("np.percentile (NaN):", np.percentile(data_nan, 50))
print("np.nanpercentile:", np.nanpercentile(data_nan, 50))
```

**Expected Output:**

```
Data: [ 1  2  3  4  5  6  7  8  9 10]
25th percentile: 3.25
50th percentile (median): 5.5
75th percentile: 7.75

0.25 quantile: 3.25
0.50 quantile: 5.5
0.75 quantile: 7.75

Multiple quantiles: [3.25 5.5  7.75]

With NaN: [ 1.  2. nan  4.  5. nan  7.  8.  9. 10.]
np.percentile (NaN): nan
np.nanpercentile: 5.5
```

**Why This Result Occurs:** The 25th percentile of 1–10 is 3.25, the median is 5.5, and the 75th percentile is 7.75. `np.percentile` returns `nan` when NaN values are present, while `np.nanpercentile` ignores them and computes the correct median of the non-NaN values.

### Real-World Cases

- **Data Analysis:** Computing quartiles and the interquartile range (IQR) for outlier detection.
- **Machine Learning:** Computing prediction intervals from ensemble outputs.
- **Finance:** Computing Value at Risk (VaR) from return distributions.
- **Quality Control:** Establishing acceptable ranges based on percentiles.

### References

- numpy.percentile – https://numpy.org/doc/2.1/reference/generated/numpy.percentile.html
- numpy.quantile – https://numpy.org/doc/1.22/reference/generated/numpy.quantile.html
- numpy.nanpercentile – https://numpy.org/doc/stable/reference/generated/numpy.nanpercentile.html
- numpy.nanquantile – https://numpy.org/doc/stable/reference/generated/numpy.nanquantile.html
- Statistics – https://numpy.org/doc/1.22/reference/routines.statistics.html

---

## Core Concept 5: Custom Accumulations & Reductions

### Definitions

**Core Definition:** Custom accumulations and reductions use the `.reduce()`, `.accumulate()`, and `.reduceat()` methods of universal functions to perform specialized operations, including high-performance windowing and segmented reductions.

**Technical Definition:** `ufunc.reduce(array, axis=0, dtype=None, out=None, keepdims=False, initial=<no value>, where=True)` reduces an array's dimension by one, applying the ufunc along one axis. For example, `np.add.reduce()` is equivalent to `np.sum()`. `ufunc.accumulate(array, axis=0, dtype=None, out=None)` returns the intermediate results of the reduction. `ufunc.reduceat(array, indices, axis=0, dtype=None, out=None)` performs a (local) reduce with specified slices over a single axis. For each `i` in `range(len(indices))`, `reduceat` computes `ufunc.reduce(array[indices[i]:indices[i+1]])`; if `indices[i] >= indices[i+1]`, the i-th generalized row is simply `array[indices[i]]`.

**Beginner-Friendly Explanation:** These methods give you fine-grained control over how NumPy accumulates values. `.reduce()` collapses an array to a single value (like sum or product). `.accumulate()` gives you all the intermediate results (like cumsum). `.reduceat()` lets you reduce over specific slices of an array, which is useful for segmented or windowed aggregations.

### Purposes

- To perform custom reductions with non-standard ufuncs (e.g., `np.logical_and.reduce`).
- To compute running results with any binary ufunc.
- To implement segmented reductions over specified index ranges.
- To achieve high-performance windowing without loops.
- To write results directly into pre-allocated buffers via the `out` parameter.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
ufunc.reduce(array, axis=0, dtype=None, out=None, keepdims=False, initial=<no value>, where=True)
ufunc.accumulate(array, axis=0, dtype=None, out=None)
ufunc.reduceat(array, indices, axis=0, dtype=None, out=None)
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `.reduce()` | Collapses the array by repeatedly applying the ufunc. |
| `.accumulate()` | Returns the intermediate results of the reduction. |
| `.reduceat()` | Performs a local reduce over slices specified by `indices`. |

**Syntax Rules:**

- `.reduce()` accepts `axis` as an integer, tuple of integers (to reduce over several axes), or `None` (reduce over all axes).
- `.accumulate()` applies along only one axis (axis zero by default).
- `.reduceat()` uses paired indices to specify slices. The output shape depends on the size of `indices` and may be larger than the input array.
- All methods accept `out` for writing results into pre-allocated arrays.

**Constraints and Limitations:**

- `.accumulate()` cannot handle multiple axes at once.
- `.reduceat()` raises an error if `indices[i] >= len(array)` or `indices[i] < 0`.
- Integer overflow is modular for `.reduce()` and `.accumulate()` with integer types.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: ufunc.reduce and ufunc.accumulate**

```python
import numpy as np

# Step 2: Create an array.
a = np.arange(1, 6)
print("Array:", a)

# Step 3: Reduce (equivalent to sum).
print("np.add.reduce(a):", np.add.reduce(a))
print("np.multiply.reduce(a):", np.multiply.reduce(a))

# Step 4: Accumulate (equivalent to cumsum).
print("np.add.accumulate(a):", np.add.accumulate(a))
print("np.multiply.accumulate(a):", np.multiply.accumulate(a))

# Step 5: Logical reduction.
b = np.array([True, True, False, True])
print("\nBoolean array:", b)
print("np.logical_and.reduce(b):", np.logical_and.reduce(b))
print("np.logical_or.reduce(b):", np.logical_or.reduce(b))
```

**Expected Output:**

```
Array: [1 2 3 4 5]
np.add.reduce(a): 15
np.multiply.reduce(a): 120
np.add.accumulate(a): [ 1  3  6 10 15]
np.multiply.accumulate(a): [  1   2   6  24 120]

Boolean array: [ True  True False  True]
np.logical_and.reduce(b): False
np.logical_or.reduce(b): True
```

**Why This Result Occurs:** `np.add.reduce` sums all elements (15), `np.multiply.reduce` multiplies them (120). `np.add.accumulate` produces the running sum, and `np.multiply.accumulate` produces the running product. Logical reductions apply `AND` and `OR` across the boolean array.

**Example 2: ufunc.reduceat for Segmented Windowing**

```python
import numpy as np

# Step 2: Create an array.
a = np.arange(10)
print("Array:", a)

# Step 3: Reduce over segments defined by indices.
# Segment 1: indices 0 to 3 (sum = 0+1+2+3 = 6)
# Segment 2: indices 4 to 6 (sum = 4+5+6 = 15)
# Segment 3: index 7 (sum = 7)
# Segment 4: index 8 to end (sum = 8+9 = 17)
indices = [0, 4, 7, 8]
result = np.add.reduceat(a, indices)
print("np.add.reduceat(a, [0, 4, 7, 8]):", result)
print("  (segments: [0:4], [4:7], [7:8], [8:])")

# Step 4: Overlapping segments.
indices_overlap = [0, 5, 2, 7]
result_overlap = np.add.reduceat(a, indices_overlap)
print("\nOverlapping segments [0, 5, 2, 7]:", result_overlap)
print("  (note: when indices[i] >= indices[i+1], result is array[indices[i]])")
```

**Expected Output:**

```
Array: [0 1 2 3 4 5 6 7 8 9]
np.add.reduceat(a, [0, 4, 7, 8]): [ 6 15  7 17]
  (segments: [0:4], [4:7], [7:8], [8:])

Overlapping segments [0, 5, 2, 7]: [10  2 27  9]
  (note: when indices[i] >= indices[i+1], result is array[indices[i]])
```

**Why This Result Occurs:** `reduceat` computes the reduction for each consecutive pair of indices. For `[0, 4]`, it sums elements 0–3 (6). For `[4, 7]`, it sums elements 4–6 (15). For `[7, 8]`, it sums element 7 (7). For `[8, end]`, it sums elements 8–9 (17). For overlapping indices, when `indices[i] >= indices[i+1]`, the result is just `array[indices[i]]`.

### Real-World Cases

- **Segmented Aggregation:** Computing sums or means over variable-length segments without loops.
- **Custom Reductions:** Using `np.logical_and.reduce` to check if all conditions are met.
- **Windowing:** Applying reductions over sliding or fixed windows in signal processing.
- **Performance Optimization:** Using `reduceat` with pre-computed indices for fast segmented statistics.

### References

- numpy.ufunc.reduce – https://numpy.org/doc/stable/reference/generated/numpy.ufunc.reduce.html
- numpy.ufunc.accumulate – https://numpy.org/doc/stable/reference/generated/numpy.ufunc.accumulate.html
- numpy.ufunc.reduceat – https://numpy.org/doc/stable/reference/generated/numpy.ufunc.reduceat.html
- Universal functions (ufunc) basics – https://numpy.org/doc/stable/user/basics.ufuncs.html

---

## Comparison Summary

| Operation | Function / Method | Returns | Key Use |
|-----------|------------------|---------|---------|
| Cumulative sum | `np.cumsum(a, axis)` | Array of running sums | Running totals |
| Cumulative product | `np.cumprod(a, axis)` | Array of running products | Compound growth |
| Cumulative min | `np.minimum.accumulate(a)` | Array of running minimums | Drawdowns, lowest values |
| Cumulative max | `np.maximum.accumulate(a)` | Array of running maximums | Peak tracking |
| Percentile | `np.percentile(a, q)` | Scalar or array | Distribution analysis |
| Quantile | `np.quantile(a, q)` | Scalar or array | Distribution analysis |
| NaN-safe percentile | `np.nanpercentile(a, q)` | Scalar or array | Missing data handling |
| NaN-safe quantile | `np.nanquantile(a, q)` | Scalar or array | Missing data handling |
| Custom reduce | `np.add.reduce(a)` | Scalar | Sum, product, logical reductions |
| Custom accumulate | `np.add.accumulate(a)` | Array | Running results with any ufunc |
| Segmented reduce | `np.add.reduceat(a, idx)` | Array | Windowed aggregation |

---

## Summary of Deprecated and Version-Specific Features

| Feature | Status | Recommendation |
|---------|--------|----------------|
| `interpolation` parameter in `percentile`/`quantile` | Deprecated since NumPy 1.22.0 | Use `method` instead |
| `method` parameter in `percentile`/`quantile` | New in NumPy 1.22.0 | Provides full set of estimation methods |
| `weights` in `percentile`/`quantile` | New in NumPy 2.0.0 | Only supported with `method='inverted_cdf'` |
| `cummin`/`cummax` as standalone functions | Do not exist in NumPy | Use `np.minimum.accumulate` and `np.maximum.accumulate` |
| `np.percentile` performance | Improved in NumPy 1.22.0 | O(n) complexity via `np.partition` |

---

## Consolidated Reference List

- numpy.cumsum – https://numpy.org/doc/2.3/reference/generated/numpy.cumsum.html
- numpy.cumprod – https://numpy.org/doc/stable/reference/generated/numpy.cumprod.html
- numpy.ufunc.accumulate – https://numpy.org/doc/stable/reference/generated/numpy.ufunc.accumulate.html
- numpy.ufunc.reduce – https://numpy.org/doc/stable/reference/generated/numpy.ufunc.reduce.html
- numpy.ufunc.reduceat – https://numpy.org/doc/stable/reference/generated/numpy.ufunc.reduceat.html
- numpy.percentile – https://numpy.org/doc/2.1/reference/generated/numpy.percentile.html
- numpy.quantile – https://numpy.org/doc/1.22/reference/generated/numpy.quantile.html
- numpy.nanpercentile – https://numpy.org/doc/stable/reference/generated/numpy.nanpercentile.html
- numpy.nanquantile – https://numpy.org/doc/stable/reference/generated/numpy.nanquantile.html
- Statistics – https://numpy.org/doc/1.22/reference/routines.statistics.html
- Universal functions (ufunc) basics – https://numpy.org/doc/stable/user/basics.ufuncs.html
- Calculating cumulative minimum with numpy arrays – https://stackoverflow.com/questions/23519611