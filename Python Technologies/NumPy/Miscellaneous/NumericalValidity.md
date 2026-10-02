# NumPy Numerical Validity & Data Cleaning: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Numerical validity and data cleaning in NumPy is the set of techniques and functions used to detect, handle, and correct invalid, missing, or anomalous numerical values—such as NaN (Not a Number), infinity, and out-of-range outliers—within arrays.

**Technical Definition:** NumPy provides a collection of functions for detecting special floating-point values (`np.isnan`, `np.isinf`, `np.isfinite`, `np.isnat`), replacing or imputing them (`np.nan_to_num`, `np.where`), bounding extreme values (`np.clip`), dynamically masking invalid data (`np.ma.masked_array`), and configuring the runtime behavior of floating-point exceptions (`np.seterr`, `np.errstate`). These tools enable robust numerical pipelines that handle incomplete or corrupted data without producing cascading errors.

**Beginner-Friendly Explanation:** Real-world data is messy—it has missing values, impossible numbers, and extreme outliers. NumPy gives you tools to find these problems, fix them, or work around them. You can detect where the bad values are, replace them with sensible defaults, limit how extreme values can get, and even tell NumPy how to react when something goes wrong numerically.

### Key Characteristics

- **Comprehensive Detection:** Dedicated functions identify NaN, infinity, NaT (Not a Time), and finite values.
- **Flexible Imputation:** `np.nan_to_num` and `np.where` allow replacing invalid values with constants or conditionally computed values.
- **Non-Destructive Outlier Handling:** `np.clip` bounds values without removing records, preserving data integrity.
- **Masked Arrays:** `numpy.ma` provides a memory-efficient way to mark invalid entries so they are ignored in computations.
- **Configurable Error Handling:** `np.seterr` and `np.errstate` control whether floating-point errors are ignored, warned, or raised as exceptions.
- **Float-Only Special Values:** NaN and infinity can only exist in floating-point (and complex) arrays; integer arrays cannot hold them.

### Prerequisites

- NumPy installed (`pip install numpy`).
- Understanding of array shapes, dtypes, and basic indexing.
- Familiarity with the `import numpy as np` convention.
- Conceptual understanding of floating-point representation (IEEE 754).

### Related Programming Areas

- **Data Science:** Cleaning datasets before analysis or machine learning.
- **Machine Learning:** Handling missing features and detecting exploding gradients.
- **Financial Modeling:** Managing division-by-zero and missing price data.
- **Scientific Computing:** Validating simulation outputs and detecting numerical instability.
- **Signal Processing:** Filtering invalid sensor readings and clipping noise.

### Core Concepts / Features

The following concepts are explored in detail: (1) Comprehensive detection, (2) Replacing and imputing missing values, (3) Clipping extreme values & outliers, (4) Masked arrays (`np.ma`), and (5) Floating-point error handling.

---

## Core Concept 1: Comprehensive Detection

### Definitions

**Core Definition:** Comprehensive detection uses NumPy's suite of detection functions to identify NaN, infinity, finite values, and NaT (Not a Time) in arrays, along with anomalous entries.

**Technical Definition:** `numpy.isnan(x)` tests element-wise for NaN and returns a boolean array. `numpy.isinf(x)` tests for positive or negative infinity. `numpy.isfinite(x)` tests for finiteness (not NaN and not infinity). `numpy.isnat(x)` tests for NaT in datetime64 and timedelta64 arrays. Counting occurrences is done by summing the boolean masks: `np.isnan(array).sum()`. For counting, `np.count_nonzero(np.isnan(array))` is also available.

**Beginner-Friendly Explanation:** These functions are your "problem detectors." `np.isnan()` finds missing numeric values, `np.isinf()` finds values that overflowed, `np.isfinite()` finds values that are normal, and `np.isnat()` finds missing dates. They all return True/False masks that you can use to locate and handle problems.

### Purposes

- To detect NaN values in floating-point arrays.
- To detect positive and negative infinity.
- To identify finite values (valid numbers).
- To detect NaT (missing datetime) values.
- To count the number of invalid entries in a dataset.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.isnan(x, out=None, where=True)
np.isinf(x, out=None, where=True)
np.isfinite(x, out=None, where=True)
np.isnat(x, out=None, where=True)
np.count_nonzero(np.isnan(x))
```

**Component Breakdown:**

| Function | Returns | Description |
|----------|---------|-------------|
| `np.isnan` | Boolean array | `True` where x is NaN. |
| `np.isinf` | Boolean array | `True` where x is ±inf. |
| `np.isfinite` | Boolean array | `True` where x is finite. |
| `np.isnat` | Boolean array | `True` where x is NaT. |
| `np.count_nonzero(mask)` | Integer | Counts `True` values. |

**Syntax Rules:**

- `np.isnan` only works on floating-point and complex arrays.
- `np.isnat` only works on datetime64 and timedelta64 arrays.
- For integer arrays, `np.isnan` returns `False` for all elements.
- Boolean masks can be summed to count invalid entries.

**Constraints and Limitations:**

- `np.isnat` raises a `TypeError` for non-datetime arrays.
- `np.isnan` on integer arrays does not raise an error in all versions; it returns `False`.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Detecting NaN, Inf, and NaT**

```python
import numpy as np

# Step 2: Create an array with NaN and Inf.
a = np.array([1.0, np.nan, np.inf, -np.inf, 3.0])
print("Array:", a)
print()

# Step 3: Detect NaN.
print("np.isnan(a):", np.isnan(a))
print("  (True where value is NaN)")
print()

# Step 4: Detect infinity.
print("np.isinf(a):", np.isinf(a))
print("  (True where value is ±inf)")
print()

# Step 5: Detect finite values.
print("np.isfinite(a):", np.isfinite(a))
print("  (True where value is finite)")
print()

# Step 6: Count invalid entries.
print("Number of NaN:", np.count_nonzero(np.isnan(a)))
print("Number of Inf:", np.count_nonzero(np.isinf(a)))
print()

# Step 7: Detect NaT in datetime arrays.
dates = np.array(['2024-01-01', 'NaT', '2024-01-03'], dtype='datetime64[D]')
print("Dates:", dates)
print("np.isnat(dates):", np.isnat(dates))
print("  (True where date is NaT)")
```

**Expected Output:**

```
Array: [ 1. nan inf -inf  3.]
np.isnan(a): [False  True False False False]
  (True where value is NaN)

np.isinf(a): [False False  True  True False]
  (True where value is ±inf)

np.isfinite(a): [ True False False False  True]
  (True where value is finite)

Number of NaN: 1
Number of Inf: 2

Dates: ['2024-01-01' 'NaT' '2024-01-03']
np.isnat(dates): [False  True False]
  (True where date is NaT)
```

**Why This Result Occurs:** `np.isnan` identifies the NaN at index 1. `np.isinf` identifies the infinities at indices 2 and 3. `np.isfinite` returns `True` only for indices 0 and 4. `np.count_nonzero` counts the number of `True` values in each mask. `np.isnat` correctly identifies the missing date.

### Real-World Cases

- **Data Loading:** Detecting missing values when reading CSV files into arrays.
- **Model Validation:** Checking whether predictions contain NaN or infinity before evaluation.
- **Time Series:** Identifying missing timestamps in datetime arrays.

### References

- numpy.isnan – https://numpy.org/doc/stable/reference/generated/numpy.isnan.html
- numpy.isinf – https://numpy.org/doc/stable/reference/generated/numpy.isinf.html
- numpy.isfinite – https://numpy.org/doc/stable/reference/generated/numpy.isfinite.html
- numpy.isnat – https://numpy.org/doc/stable/reference/generated/numpy.isnat.html
- NumPy Handling Missing Data – https://m.yisu.com/zixun/974089.html

---

## Core Concept 2: Replacing and Imputing Missing Values

### Definitions

**Core Definition:** Replacing and imputing missing values involves substituting NaN, infinity, or other invalid entries with meaningful replacements—such as constants, boundary values, or conditionally computed values.

**Technical Definition:** `numpy.nan_to_num(x, copy=True, nan=0.0, posinf=None, neginf=None)` replaces NaN with zero (or a user-defined value) and infinity with the largest (or smallest) finite floating-point values (or user-defined values). For conditional imputation, `numpy.where(condition, x, y)` selects values from `x` or `y` depending on a boolean condition, enabling strategies such as mean imputation or conditional replacement based on multiple criteria.

**Beginner-Friendly Explanation:** When you find bad values, you can either replace them with a fixed number (like 0 or the mean) or use a conditional rule to decide what each missing value should become. `np.nan_to_num` is the quick fix for NaN and infinity, while `np.where` gives you fine-grained control.

### Purposes

- To replace NaN with zero or a custom constant.
- To replace infinity with large finite numbers.
- To impute missing values with the mean, median, or a conditional value.
- To prepare data for algorithms that cannot handle NaN or infinity.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.nan_to_num(x, copy=True, nan=0.0, posinf=None, neginf=None)
np.where(condition, x, y)
```

**Component Breakdown:**

| Function | Parameter | Description |
|----------|-----------|-------------|
| `np.nan_to_num` | `nan` | Value to replace NaN with (default 0.0). |
| | `posinf` | Value to replace +inf with (default max float). |
| | `neginf` | Value to replace −inf with (default min float). |
| `np.where` | `condition` | Boolean array selecting values. |
| | `x` | Values where `condition` is `True`. |
| | `y` | Values where `condition` is `False`. |

**Syntax Rules:**

- `np.nan_to_num` with `copy=False` modifies the input array in place (if possible).
- For complex dtypes, replacements apply to real and imaginary components separately.
- `np.where` evaluates both `x` and `y` eagerly; for expensive computations, consider alternatives.

**Constraints and Limitations:**

- `np.nan_to_num` does not distinguish between different types of missing values.
- `np.where` creates a new array; the original is not modified unless assigned back.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Replacing NaN and Inf with nan_to_num**

```python
import numpy as np

# Step 2: Create an array with NaN and Inf.
a = np.array([1.0, np.nan, np.inf, -np.inf, 3.0])
print("Original:", a)
print()

# Step 3: Replace with defaults.
default = np.nan_to_num(a)
print("np.nan_to_num(a):")
print(default)
print("  (NaN -> 0.0, +inf -> max float, -inf -> min float)")
print()

# Step 4: Replace with custom values.
custom = np.nan_to_num(a, nan=-1.0, posinf=100.0, neginf=-100.0)
print("Custom replacement:", custom)
print()

# Step 5: Conditional imputation with np.where (replace NaN with mean).
b = np.array([1.0, 2.0, np.nan, 4.0, np.nan, 6.0])
print("Array with NaN:", b)
mean_val = np.nanmean(b)
imputed = np.where(np.isnan(b), mean_val, b)
print(f"Mean value: {mean_val}")
print("After imputation:", imputed)
```

**Expected Output:**

```
Original: [ 1. nan inf -inf  3.]
np.nan_to_num(a):
[ 1.00000000e+00  0.00000000e+00  1.79769313e+308 -1.79769313e+308
  3.00000000e+00]
  (NaN -> 0.0, +inf -> max float, -inf -> min float)

Custom replacement: [  1.  -1. 100. -100.   3.]

Array with NaN: [ 1.  2. nan  4. nan  6.]
Mean value: 3.25
After imputation: [1.  2.  3.25 4.  3.25 6. ]
```

**Why This Result Occurs:** `np.nan_to_num` with defaults replaces NaN with 0 and infinities with the largest/smallest finite float64 values. Custom values override these defaults. In the conditional example, `np.nanmean` computes the mean of non-NaN values (3.25), and `np.where` replaces each NaN with that mean.

### Real-World Cases

- **Data Preprocessing:** Replacing missing values with the column mean before training a model.
- **Financial Data:** Replacing missing returns with 0 or the previous day's value.
- **Sensor Data:** Replacing invalid readings with the last known good value.

### References

- numpy.nan_to_num – https://numpy.org/doc/1.20/reference/generated/numpy.nan_to_num.html
- numpy.where – https://numpy.org/doc/stable/reference/generated/numpy.where.html
- NumPy Handling Missing Data – https://m.yisu.com/zixun/974089.html

---

## Core Concept 3: Clipping Extreme Values & Outliers

### Definitions

**Core Definition:** Clipping is the process of bounding array values within a specified minimum and maximum range, replacing values outside the range with the boundary values.

**Technical Definition:** `numpy.clip(a, a_min, a_max, out=None)` limits the values in an array. Given an interval, values outside the interval are clipped to the interval edges. For example, if an interval of [0, 1] is specified, values smaller than 0 become 0, and values larger than 1 become 1. The `out` parameter allows in-place clipping. Dynamic thresholds can be computed from the data using percentiles: `np.percentile(data, [5, 95])`.

**Beginner-Friendly Explanation:** Clipping is like putting a fence around your data. Values that are too small get raised to the fence, and values that are too large get lowered to the fence. Nothing is removed—the extreme values are simply capped. This is useful for handling outliers without deleting data.

### Purposes

- To bound data within a reasonable range without removing records.
- To handle outliers by capping them at percentile-based thresholds.
- To prepare data for algorithms sensitive to extreme values.
- To implement gradient clipping in neural network training.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.clip(a, a_min, a_max, out=None)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `a` | Input array. |
| `a_min` | Minimum value; values below this become `a_min`. Can be `None`. |
| `a_max` | Maximum value; values above this become `a_max`. Can be `None`. |
| `out` | Optional output array; can be `a` for in-place clipping. |

**Syntax Rules:**

- At most one of `a_min` and `a_max` may be `None`.
- `a_min` and `a_max` are broadcast against `a`.
- The result has the same dtype as the input.

**Constraints and Limitations:**

- If `a_min > a_max`, the result is undefined.
- Clipping does not remove outliers; it caps them.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Static and Dynamic Clipping**

```python
import numpy as np

# Step 2: Create an array with outliers.
data = np.array([1, 2, 3, 10, 20, 30, -5, 100])
print("Original:", data)
print()

# Step 3: Static clipping to [0, 20].
clipped_static = np.clip(data, 0, 20)
print("np.clip(data, 0, 20):", clipped_static)
print("  (values below 0 become 0, above 20 become 20)")
print()

# Step 4: Dynamic clipping using percentiles.
lo, hi = np.percentile(data, [10, 90])
print(f"10th percentile: {lo}, 90th percentile: {hi}")
clipped_dynamic = np.clip(data, lo, hi)
print("Dynamic clip:", clipped_dynamic)
print()

# Step 5: In-place clipping.
data_copy = data.copy()
np.clip(data_copy, 0, 15, out=data_copy)
print("In-place clip to [0, 15]:", data_copy)
```

**Expected Output:**

```
Original: [  1   2   3  10  20  30  -5 100]

np.clip(data, 0, 20): [ 1  2  3 10 20 20  0 20]
  (values below 0 become 0, above 20 become 20)

10th percentile: -2.0, 90th percentile: 51.0
Dynamic clip: [ 1.  2.  3. 10. 20. 30. -2. 51.]

In-place clip to [0, 15]: [ 1  2  3 10 15 15  0 15]
```

**Why This Result Occurs:** Static clipping bounds values to [0, 20]. Dynamic clipping uses data-derived percentiles to set bounds. In-place clipping modifies the original array. Clipping preserves the number of records while limiting extreme values.

### Real-World Cases

- **Image Processing:** Clipping pixel values to [0, 255] after filtering.
- **Neural Networks:** Gradient clipping to prevent exploding gradients.
- **Financial Data:** Capping returns at a reasonable maximum to limit outlier influence.
- **Sensor Data:** Bounding readings to the sensor's valid range.

### References

- numpy.clip – https://numpy.org/doc/stable/reference/generated/numpy.clip.html
- Python Numpy clip() – https://docs.vultr.com/python/third-party/numpy/clip
- Handling Outliers with Clip – https://dmccreary.github.io/data-science-course/chapters/04-data-cleaning/

---

## Core Concept 4: Masked Arrays (np.ma Module)

### Definitions

**Core Definition:** Masked arrays are arrays that carry a boolean mask alongside their data, marking certain entries as invalid or missing so that they are ignored in computations.

**Technical Definition:** The `numpy.ma` module provides the `MaskedArray` class, a subclass of `ndarray` that combines a standard array with a mask. A mask is either `nomask` (no invalid entries) or a boolean array where `True` indicates a masked (invalid) element. Masked entries are excluded from computations. `np.ma.masked_array(x, mask)` constructs a masked array. `np.ma.getmask()` retrieves the mask. The `fill_value` attribute specifies the value used to replace masked entries when converting back to a regular array.

**Beginner-Friendly Explanation:** A masked array is like a regular array with sticky notes on some entries saying "ignore this one." When you compute statistics, NumPy skips the marked entries. This is more flexible than replacing values with NaN because you can keep the original values for inspection while excluding them from calculations.

### Purposes

- To handle missing or invalid data dynamically.
- To perform computations that automatically ignore masked entries.
- To preserve the original data while marking entries as invalid.
- To control how masked values are filled when converting to a regular array.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
import numpy.ma as ma
mx = ma.masked_array(x, mask=condition)
mask = ma.getmask(mx)
mx.fill_value = value
filled = mx.filled(fill_value)
```

**Component Breakdown:**

| Function/Attribute | Description |
|-------------------|-------------|
| `ma.masked_array(data, mask)` | Creates a masked array. |
| `ma.getmask(mx)` | Returns the mask of a masked array. |
| `mx.fill_value` | Value used when filling masked entries. |
| `mx.filled(val)` | Returns a regular array with masked entries replaced. |
| `mx.mean()` | Computes mean ignoring masked entries. |

**Syntax Rules:**

- The mask has the same shape as the data.
- `True` in the mask indicates an invalid (masked) element.
- Masked entries are ignored in aggregations.
- `nomask` (default) means no elements are masked.

**Constraints and Limitations:**

- Masked arrays are a subclass of `ndarray`, so they work with most NumPy functions.
- Some operations may return regular ndarrays, losing the mask.
- Performance is slightly lower than regular arrays due to mask handling.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Creating and Using Masked Arrays**

```python
import numpy as np
import numpy.ma as ma

# Step 2: Create data with an invalid entry.
x = np.array([1, 2, 3, -1, 5])
print("Original data:", x)
print()

# Step 3: Create a masked array, masking the -1.
mx = ma.masked_array(x, mask=[0, 0, 0, 1, 0])
print("Masked array:", mx)
print("  (the fourth entry is masked with --)")
print()

# Step 4: Compute statistics ignoring the masked value.
print("Mean (ignoring masked):", mx.mean())
print("  (sum of 1+2+3+5 = 11, divided by 4 = 2.75)")
print()

# Step 5: Access the mask and fill value.
print("Mask:", ma.getmask(mx))
print("Fill value:", mx.fill_value)
print()

# Step 6: Fill masked entries with a custom value.
filled = mx.filled(0)
print("Filled with 0:", filled)
print()

# Step 7: Create a masked array from a NaN condition.
data = np.array([1.0, np.nan, 3.0, np.nan, 5.0])
masked_nan = ma.masked_array(data, np.isnan(data))
print("Masked NaN array:", masked_nan)
print("Mean (ignoring NaN):", masked_nan.mean())
```

**Expected Output:**

```
Original data: [ 1  2  3 -1  5]

Masked array: [1 2 3 -- 5]
  (the fourth entry is masked with --)

Mean (ignoring masked): 2.75
  (sum of 1+2+3+5 = 11, divided by 4 = 2.75)

Mask: [False False False  True False]
Fill value: 999999

Filled with 0: [1 2 3 0 5]

Masked NaN array: [1.0 -- 3.0 -- 5.0]
Mean (ignoring NaN): 3.0
```

**Why This Result Occurs:** `ma.masked_array` creates an array where the fourth element is marked as invalid. `mx.mean()` ignores the masked value and computes the mean of the remaining four values. `filled(0)` replaces the masked entry with 0. The NaN-based mask automatically identifies and masks NaN values.

### Real-World Cases

- **Sensor Networks:** Masking failed sensor readings while preserving other data.
- **Survey Analysis:** Masking skipped questions and computing statistics on valid responses.
- **Financial Data:** Masking missing trading days and computing returns on available data.
- **Image Processing:** Masking corrupted pixels and computing statistics on valid regions.

### References

- The numpy.ma module – https://numpy.org/doc/2.2/reference/maskedarray.generic.html
- numpy.ma.masked_array – https://numpy.org/doc/stable/reference/generated/numpy.ma.masked_array.html
- numpy.ma.getmask – https://numpy.org/doc/stable/reference/generated/numpy.ma.getmask.html
- Masked Array Fill Values – https://numpy.org/doc/stable/reference/generated/numpy.ma.MaskedArray.fill_value.html

---

## Core Concept 5: Floating-Point Error Handling

### Definitions

**Core Definition:** Floating-point error handling in NumPy is the configuration of how the library responds to numerical exceptions such as division by zero, overflow, underflow, and invalid operations.

**Technical Definition:** `numpy.seterr(all=None, divide=None, over=None, under=None, invalid=None)` sets how floating-point errors are handled globally. `numpy.errstate(**kwargs)` is a context manager that temporarily sets error handling behavior for a block of code. The available behaviors are `'ignore'`, `'warn'`, `'raise'`, `'call'`, `'print'`, and `'log'`. The default is `'warn'` for invalid, divide, and overflow, and `'ignore'` for underflow. Error handling settings are stored in `contextvars`, allowing different threads or async tasks to have independent configurations.

**Beginner-Friendly Explanation:** When NumPy encounters a numerical problem—like dividing by zero or taking the square root of a negative number—it needs to know what to do. By default, it prints a warning. You can change this to ignore the problem entirely, raise an exception, or call a custom function. The `errstate` context manager lets you change this behavior temporarily for a specific block of code.

### Purposes

- To suppress warnings for expected numerical edge cases (e.g., division by zero in a masked computation).
- To raise exceptions for debugging numerical instability.
- To customize the response to floating-point errors.
- To isolate error handling behavior to specific code blocks.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.seterr(all=None, divide=None, over=None, under=None, invalid=None)
with np.errstate(divide='ignore', invalid='ignore'):
    # code with modified error handling
    ...
```

**Component Breakdown:**

| Behavior | Description |
|----------|-------------|
| `'ignore'` | Take no action. |
| `'warn'` | Print a `RuntimeWarning`. |
| `'raise'` | Raise a `FloatingPointError`. |
| `'call'` | Call a specified function. |
| `'print'` | Print directly to stdout. |
| `'log'` | Record in a Log object. |

| Error Type | Description |
|------------|-------------|
| `divide` | Division by zero. |
| `over` | Floating-point overflow. |
| `under` | Floating-point underflow. |
| `invalid` | Invalid operations (e.g., sqrt(-1)). |

**Syntax Rules:**

- `seterr` changes global settings and returns the previous settings.
- `errstate` is a context manager that restores settings on exit.
- `errstate` should be used instead of `seterr` when possible to avoid global side effects.
- Settings can be applied to all errors or specific types.

**Constraints and Limitations:**

- `seterr` affects global state; use `errstate` for temporary changes.
- `'raise'` mode turns warnings into exceptions, which may require try/except handling.
- Integer division by zero is handled by the same machinery, even though it is not a floating-point operation.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Using errstate to Suppress Warnings**

```python
import numpy as np

# Step 2: Perform division by zero with default behavior.
a = np.array([1.0, 2.0, 0.0])
b = np.array([1.0, 0.0, 0.0])
print("Default behavior (warning):")
with np.errstate(divide='warn'):
    result = a / b
    print("Result:", result)
print()

# Step 3: Suppress the warning.
print("Suppressed warning:")
with np.errstate(divide='ignore'):
    result = a / b
    print("Result:", result)
print()

# Step 4: Raise an exception.
print("Raise exception:")
try:
    with np.errstate(divide='raise'):
        result = a / b
except FloatingPointError as e:
    print("FloatingPointError:", e)
print()

# Step 5: Use seterr to check current settings.
print("Current error settings:", np.geterr())
```

**Expected Output:**

```
Default behavior (warning):
Result: [ 1. inf nan]

Suppressed warning:
Result: [ 1. inf nan]

Raise exception:
FloatingPointError: divide by zero encountered in divide

Current error settings: {'divide': 'warn', 'over': 'warn', 'under': 'ignore', 'invalid': 'warn'}
```

**Why This Result Occurs:** With `divide='warn'`, NumPy prints a `RuntimeWarning` (not shown in the output for brevity) but still produces the result. With `divide='ignore'`, no warning is printed. With `divide='raise'`, a `FloatingPointError` is raised instead of producing a result. `np.geterr()` returns the current global settings.

### Real-World Cases

- **Pipeline Development:** Suppressing warnings for expected edge cases while debugging.
- **Production Code:** Raising exceptions for numerical instability to catch bugs early.
- **Testing:** Using `errstate` to verify that specific operations produce expected warnings.

### References

- Floating point error handling – https://numpy.org/doc/2.5/reference/routines.err.html
- numpy.seterr – https://numpy.org/doc/stable/reference/generated/numpy.seterr.html
- numpy.errstate – https://numpy.org/doc/stable/reference/generated/numpy.errstate.html
- numpy.geterr – https://numpy.org/doc/stable/reference/generated/numpy.geterr.html

---

## Comparison Summary

| Technique | Function | Purpose | Preserves Data |
|-----------|----------|---------|----------------|
| Detection | `np.isnan`, `np.isinf`, `np.isfinite`, `np.isnat` | Find invalid values | Yes |
| Replacement | `np.nan_to_num` | Replace NaN/Inf with defaults | No |
| Conditional imputation | `np.where` | Replace based on conditions | No |
| Clipping | `np.clip` | Bound extreme values | Yes |
| Masking | `np.ma.masked_array` | Mark invalid entries | Yes |
| Error handling | `np.seterr`, `np.errstate` | Control numerical exception behavior | Yes |

---

## Summary of Deprecated and Version-Specific Features

| Feature | Status | Recommendation |
|---------|--------|----------------|
| `np.nan_to_num` with `nan`, `posinf`, `neginf` | New in NumPy 1.17 | Use for custom replacements |
| `np.isnat` | Available since NumPy 1.7 | Use for datetime NaT detection |
| `np.errstate` as context manager | Available since NumPy 1.0 | Preferred over `np.seterr` for temporary changes |
| `np.ma.masked_array` | Available | Use for masked data handling |
| Integer arrays with NaN | Not supported | Use floating-point arrays |

---

## Consolidated Reference List

- numpy.isnan – https://numpy.org/doc/stable/reference/generated/numpy.isnan.html
- numpy.isinf – https://numpy.org/doc/stable/reference/generated/numpy.isinf.html
- numpy.isfinite – https://numpy.org/doc/stable/reference/generated/numpy.isfinite.html
- numpy.isnat – https://numpy.org/doc/stable/reference/generated/numpy.isnat.html
- numpy.nan_to_num – https://numpy.org/doc/1.20/reference/generated/numpy.nan_to_num.html
- numpy.where – https://numpy.org/doc/stable/reference/generated/numpy.where.html
- numpy.clip – https://numpy.org/doc/stable/reference/generated/numpy.clip.html
- The numpy.ma module – https://numpy.org/doc/2.2/reference/maskedarray.generic.html
- numpy.ma.masked_array – https://numpy.org/doc/stable/reference/generated/numpy.ma.masked_array.html
- numpy.ma.getmask – https://numpy.org/doc/stable/reference/generated/numpy.ma.getmask.html
- Floating point error handling – https://numpy.org/doc/2.5/reference/routines.err.html
- numpy.seterr – https://numpy.org/doc/stable/reference/generated/numpy.seterr.html
- numpy.errstate – https://numpy.org/doc/stable/reference/generated/numpy.errstate.html
- numpy.geterr – https://numpy.org/doc/stable/reference/generated/numpy.geterr.html
- NumPy Handling Missing Data – https://m.yisu.com/zixun/974089.html
- Python Numpy clip() – https://docs.vultr.com/python/third-party/numpy/clip
- Handling Outliers with Clip – https://dmccreary.github.io/data-science-course/chapters/04-data-cleaning/