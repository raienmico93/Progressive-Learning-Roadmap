# NumPy NaN (Not a Number): A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** NaN (Not a Number) is a special floating-point value defined by the IEEE 754 standard that represents undefined or unrepresentable results of arithmetic operations, such as 0/0, ∞−∞, or the square root of a negative number.

**Technical Definition:** NaN is an "unordered" floating-point value. In IEEE 754, four mutually exclusive relations are possible between two operands: less than, equal, greater than, and unordered. Unordered arises when at least one operand is a NaN, and every NaN compares unordered with everything, including itself. This means `NaN != NaN` is always `True`, and all other comparisons (`<`, `>`, `==`) return `False` when NaN is involved. NaNs propagate through subsequent arithmetic operations by default.

**Beginner-Friendly Explanation:** NaN is what a computer produces when it tries to do math that has no meaningful answer—like dividing zero by zero or taking the square root of a negative number. It's a "poison value": once it appears in a calculation, it usually spreads to the result. The trickiest thing about NaN is that it doesn't even equal itself, so you can't check for it with `==`. NumPy provides special functions like `np.isnan()` to detect it and a whole suite of `nan`-prefixed functions to work around it.

### Key Characteristics

- **IEEE 754 Standard:** NaN is defined by the IEEE 754 floating-point specification.
- **Unordered Comparisons:** NaN compares unordered with everything, including itself.
- **Propagation:** NaNs propagate through arithmetic operations.
- **Float-Only:** NaN can only exist in floating-point (and complex) arrays; integer arrays cannot hold NaN.
- **Special Detection Required:** Use `np.isnan()` to test for NaN, never `==`.
- **NaN-Safe Functions:** NumPy provides `nan`-prefixed alternatives to most reduction functions.

### Prerequisites

- NumPy installed (`pip install numpy`).
- Basic understanding of floating-point numbers and IEEE 754.
- Familiarity with the `import numpy as np` convention.
- Conceptual understanding of boolean masking and aggregation.

### Related Programming Areas

- **Data Cleaning:** Handling missing values in datasets.
- **Scientific Computing:** Managing undefined results in simulations.
- **Machine Learning:** Dealing with missing features and labels.
- **Financial Modeling:** Handling missing price or volume data.
- **Signal Processing:** Managing invalid sensor readings.

### Core Concepts / Features

The following concepts are explored in detail: (1) The structural meaning of NaN, (2) Detecting NaN variations, (3) Ignoring NaN during aggregation, and (4) Filtering and removing NaN.

---

## Core Concept 1: The Structural Meaning of NaN

### Definitions

**Core Definition:** NaN is a special IEEE 754 floating-point value that represents an undefined or unrepresentable result, characterized by the unique property that it is not equal to any value, including itself.

**Technical Definition:** IEEE 754 defines NaN as one of the "reserved operands" that serves as the default result of invalid operations and propagates through subsequent arithmetic operations. The standard specifies that every NaN shall compare unordered with everything, including itself. This means that all comparison operators (`<`, `>`, `<=`, `>=`, `==`) return `False` when NaN is involved, except for `!=`, which returns `True`. NaNs can be used as a "poor-man's mask" in NumPy arrays if you don't care about the original value.

**Beginner-Friendly Explanation:** NaN is the computer's way of saying "I don't know what this number is." It might come from dividing 0 by 0, taking the square root of a negative number, or a missing data point. Because NaN means "unknown," it can't be equal to anything—not even another NaN. That's why `NaN == NaN` is `False`. This property makes NaN both useful (as a marker for missing data) and tricky (you can't use normal equality checks).

### Purposes

- To represent undefined or unrepresentable results in floating-point computations.
- To mark missing or invalid data in arrays.
- To propagate errors through calculations without raising exceptions.
- To serve as a sentinel value that can be detected and handled specially.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.nan           # The NaN constant
np.NaN           # Alias
np.NAN           # Alias
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `np.nan` | The floating-point NaN constant. |
| Type | `float` (Python) / `numpy.float64`. |
| Comparison | `NaN != NaN` is `True`; all other comparisons are `False`. |
| Propagation | Arithmetic operations involving NaN produce NaN. |

**Syntax Rules:**

- Never use `==` to test for NaN; it always returns `False`.
- Use `np.isnan()` for element-wise NaN detection.
- NaN can only exist in floating-point and complex arrays; integer arrays cannot hold NaN.
- NaNs propagate through arithmetic operations: `np.nan + 1` is `nan`.

**Constraints and Limitations:**

- Integer arrays cannot natively hold NaN values.
- NaN detection requires special functions, not equality checks.
- Comparisons involving NaN are unordered, meaning sorting with NaN produces undefined ordering.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Demonstrating NaN's Unordered Comparison Property**

```python
import numpy as np

# Step 2: Create arrays with NaN.
a = np.array([1.0, np.nan, 3.0])
print("Array with NaN:", a)

# Step 3: Demonstrate that NaN != NaN.
print("np.nan == np.nan:", np.nan == np.nan)
print("np.nan != np.nan:", np.nan != np.nan)
print()

# Step 4: Show that equality checks with NaN fail.
mask = a == np.nan
print("a == np.nan:", mask)
print("  (all False because NaN is never equal to anything)")
print()

# Step 5: Demonstrate propagation.
result = np.nan + 1
print("np.nan + 1:", result)
print("np.nan * 0:", np.nan * 0)
print("  (NaN propagates through arithmetic)")
```

**Expected Output:**

```
Array with NaN: [ 1. nan  3.]
np.nan == np.nan: False
np.nan != np.nan: True

a == np.nan: [False False False]
  (all False because NaN is never equal to anything)

np.nan + 1: nan
np.nan * 0: nan
  (NaN propagates through arithmetic)
```

**Why This Result Occurs:** IEEE 754 specifies that NaN compares unordered with everything, including itself. This is why `NaN == NaN` is `False` and `NaN != NaN` is `True`. Any arithmetic operation involving NaN produces NaN because the result is undefined.

### Real-World Cases

- **Missing Data:** NaN is commonly used to represent missing values in datasets loaded from CSV files, databases, or sensors.
- **Invalid Operations:** NaN naturally arises from invalid computations like `0/0`, `sqrt(-1)`, or `inf - inf`.
- **Quality Flags:** NaN can be used as a sentinel value to flag invalid measurements in scientific data.

### References

- NumPy Miscellaneous (IEEE 754 special values) – https://numpy.org/doc/1.20/user/misc.html
- FreeBSD IEEE 754 Manual – https://man.freebsd.org
- IEEE 754-2019 Section 5.11 – https://stackoverflow.com

---

## Core Concept 2: Detecting NaN Variations

### Definitions

**Core Definition:** `np.isnan()` is the canonical function for detecting NaN values in floating-point arrays, returning a boolean mask where `True` indicates the presence of NaN.

**Technical Definition:** `numpy.isnan(x, out=None, where=True, casting='same_kind', order='K', dtype=None, subok=True)` tests element-wise for NaN and returns a boolean array of the same shape as the input. It only works on floating-point and complex arrays; for integer arrays, it returns `False` for all elements. Related functions include `np.isinf()` (tests for infinity), `np.isfinite()` (tests for neither NaN nor infinity), and `np.nan_to_num()` (replaces NaN with zero and infinity with large finite numbers).

**Beginner-Friendly Explanation:** `np.isnan()` is the "NaN detector." You pass it an array, and it gives you back a True/False mask showing exactly where the NaNs are. This is essential because you can't use `==` to find NaN.

### Purposes

- To detect NaN values in floating-point arrays.
- To create boolean masks for filtering or replacing NaN values.
- To check whether an array contains any NaN values.
- To complement `isinf` and `isfinite` for complete special-value detection.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.isnan(x, out=None, where=True)
np.isinf(x, out=None, where=True)
np.isfinite(x, out=None, where=True)
np.nan_to_num(x, copy=True, nan=0.0, posinf=None, neginf=None)
```

**Component Breakdown:**

| Function | Returns | Description |
|----------|---------|-------------|
| `np.isnan` | Boolean array | `True` where x is NaN. |
| `np.isinf` | Boolean array | `True` where x is ±inf. |
| `np.isfinite` | Boolean array | `True` where x is neither NaN nor inf. |
| `np.nan_to_num` | Float array | Replaces NaN/Inf with finite values. |

**Syntax Rules:**

- `np.isnan` only works on floating-point and complex arrays.
- For integer arrays, `np.isnan` returns `False` for all elements.
- `np.nan_to_num` default: NaN → 0.0, +inf → max float, −inf → min float.
- Use `np.isnan(x).any()` to check whether any NaN exists.

**Constraints and Limitations:**

- `np.isnan` raises a `TypeError` if applied to integer arrays in some contexts.
- `np.nan_to_num` with `copy=False` modifies the input array in place.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Detecting and Handling NaN**

```python
import numpy as np

# Step 2: Create an array with NaN and Inf.
a = np.array([1.0, np.nan, np.inf, -np.inf, 3.0])
print("Original:", a)

# Step 3: Detect NaN.
print("np.isnan(a):", np.isnan(a))

# Step 4: Detect Inf.
print("np.isinf(a):", np.isinf(a))

# Step 5: Detect finite values.
print("np.isfinite(a):", np.isfinite(a))

# Step 6: Check if any NaN exists.
print("Any NaN:", np.isnan(a).any())

# Step 7: Replace special values with defaults.
safe = np.nan_to_num(a)
print("np.nan_to_num(a):", safe)

# Step 8: Replace with custom values.
custom = np.nan_to_num(a, nan=-1.0, posinf=100.0, neginf=-100.0)
print("Custom replacement:", custom)
```

**Expected Output:**

```
Original: [ 1. nan inf -inf  3.]
np.isnan(a): [False  True False False False]
np.isinf(a): [False False  True  True False]
np.isfinite(a): [ True False False False  True]
Any NaN: True
np.nan_to_num(a): [ 1.00000000e+00  0.00000000e+00  1.79769313e+308
 -1.79769313e+308  3.00000000e+00]
Custom replacement: [  1.  -1. 100. -100.   3.]
```

**Why This Result Occurs:** `np.isnan` identifies the NaN at index 1. `np.isinf` identifies the infinities at indices 2 and 3. `np.isfinite` is `True` only where values are neither NaN nor infinite. `np.nan_to_num` replaces NaN with 0 and infinities with the largest finite float64 values. Custom replacements override the defaults.

### Real-World Cases

- **Data Cleaning:** Detecting and replacing missing values before analysis.
- **Validation:** Checking whether a computation produced any invalid results.
- **Preprocessing:** Converting NaN to a neutral value before feeding data to a model.

### References

- numpy.isnan – https://numpy.org/doc/stable/reference/generated/numpy.isnan.html
- numpy.isinf – https://numpy.org/doc/stable/reference/generated/numpy.isinf.html
- numpy.isfinite – https://numpy.org/doc/stable/reference/generated/numpy.isfinite.html
- numpy.nan_to_num – https://numpy.org/doc/1.20/reference/generated/numpy.nan_to_num.html
- Miscellaneous – https://numpy.org/devdocs/user/misc.html

---

## Core Concept 3: Ignoring NaN During Aggregation

### Definitions

**Core Definition:** NaN-safe aggregation functions compute statistics (sum, mean, min, max, std, var, argmin, argmax) while ignoring NaN values, treating them as if they were not present in the array.

**Technical Definition:** NumPy provides a complete suite of `nan`-prefixed functions that exclude NaN values from their calculations: `np.nansum`, `np.nanmean`, `np.nanmin`, `np.nanmax`, `np.nanstd`, `np.nanvar`, `np.nanargmin`, and `np.nanargmax`, among others. These functions replace NaN values with a fill value (0 for sum, +inf for min, -inf for max, etc.), perform the aggregation, and then restore NaN where appropriate. For all-NaN slices, `nanmean` returns NaN and raises a `RuntimeWarning`, while `nanargmin` and `nanargmax` raise a `ValueError`.

**Beginner-Friendly Explanation:** If your array has missing values (NaN), regular aggregation functions will return NaN because NaN "poisons" the result. The `nan`-prefixed versions ignore the NaN values and compute the result from the remaining data. For example, `np.nanmean([1, 2, NaN, 4])` returns 2.33 (the mean of 1, 2, and 4), while `np.mean([1, 2, NaN, 4])` returns NaN.

### Purposes

- To compute statistics on datasets with missing values.
- To avoid NaN propagation in aggregation pipelines.
- To get meaningful results from incomplete data.
- To implement robust data analysis workflows.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.nansum(a, axis=None, dtype=None, out=None, keepdims=<no value>)
np.nanmean(a, axis=None, dtype=None, out=None, keepdims=<no value>)
np.nanmin(a, axis=None, out=None, keepdims=<no value>)
np.nanmax(a, axis=None, out=None, keepdims=<no value>)
np.nanstd(a, axis=None, dtype=None, out=None, ddof=0, keepdims=<no value>)
np.nanvar(a, axis=None, dtype=None, out=None, ddof=0, keepdims=<no value>)
np.nanargmin(a, axis=None, out=None, keepdims=<no value>)
np.nanargmax(a, axis=None, out=None, keepdims=<no value>)
```

**Component Breakdown:**

| Function | Ignores NaN | Behavior on All-NaN Slice |
|----------|-------------|---------------------------|
| `nansum` | Yes | Returns 0 |
| `nanmean` | Yes | Returns NaN + RuntimeWarning |
| `nanmin` / `nanmax` | Yes | Returns NaN |
| `nanstd` / `nanvar` | Yes | Returns NaN + RuntimeWarning |
| `nanargmin` / `nanargmax` | Yes | Raises ValueError |

**Syntax Rules:**

- `nanmean` computes the sum of non-NaN elements divided by the count of non-NaN elements.
- `nansum` treats NaN as zero.
- `nanmin` and `nanmax` use `np.fmin` and `np.fmax` reduce operations internally.
- `nanargmin` and `nanargmax` raise `ValueError` if a slice contains only NaN.

**Constraints and Limitations:**

- These functions are slower than their non-NaN counterparts due to the extra masking step.
- `nanmean` on integer arrays is equivalent to regular mean (integers cannot hold NaN).
- For all-NaN slices, results vary (NaN, 0, or ValueError) depending on the function.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: NaN-Safe Aggregation Suite**

```python
import numpy as np

# Step 2: Create an array with NaN.
a = np.array([1.0, 2.0, np.nan, 4.0, 5.0])
print("Array:", a)
print()

# Step 3: Compare regular vs. NaN-safe sum.
print("np.sum(a):", np.sum(a))
print("np.nansum(a):", np.nansum(a))
print()

# Step 4: Compare regular vs. NaN-safe mean.
print("np.mean(a):", np.mean(a))
print("np.nanmean(a):", np.nanmean(a))
print()

# Step 5: NaN-safe min and max.
print("np.nanmin(a):", np.nanmin(a))
print("np.nanmax(a):", np.nanmax(a))
print()

# Step 6: NaN-safe std and var.
print("np.nanstd(a):", np.nanstd(a))
print("np.nanvar(a):", np.nanvar(a))
print()

# Step 7: NaN-safe argmin and argmax.
print("np.nanargmin(a):", np.nanargmin(a))
print("np.nanargmax(a):", np.nanargmax(a))
print()

# Step 8: 2-D array with axis parameter.
m = np.array([[1, np.nan], [3, 4]])
print("2-D array:\n", m)
print("np.nanmean(m):", np.nanmean(m))
print("np.nanmean(m, axis=0):", np.nanmean(m, axis=0))
print("np.nanmean(m, axis=1):", np.nanmean(m, axis=1))
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

np.nanstd(a): 1.5811388300841898
np.nanvar(a): 2.5

np.nanargmin(a): 0
np.nanargmax(a): 4

2-D array:
 [[ 1. nan]
 [ 3.  4.]]
np.nanmean(m): 2.6666666666666665
np.nanmean(m, axis=0): [2. 4.]
np.nanmean(m, axis=1): [1.  3.5]
```

**Why This Result Occurs:** `np.sum(a)` returns NaN because NaN propagates. `np.nansum(a)` ignores the NaN and sums 1+2+4+5 = 12. `np.nanmean(a)` divides by 4 (the count of non-NaN values), yielding 3.0. The `nan`-safe functions produce meaningful results despite the missing value.

### Real-World Cases

- **Sensor Data:** Computing average temperature when some sensors failed.
- **Survey Data:** Calculating mean responses when some questions were skipped.
- **Financial Data:** Computing average returns when some trading days had missing data.
- **Machine Learning:** Computing loss on batches with missing labels.

### References

- numpy.nanmean – https://numpy.org/doc/1.20//reference/generated/numpy.nanmean.html
- numpy.nansum – https://numpy.org/doc/2.2/reference/generated/numpy.nansum.html
- Functions that ignore NaN – https://chromium.cpp.hybrid-analysis.googlesource.com
- numpy nanargmin/nanargmax – https://chromium.googlesource.com

---

## Core Concept 4: Filtering and Removing NaN

### Definitions

**Core Definition:** Filtering and removing NaN involves using boolean masks derived from `np.isnan()` to strip out records or elements containing NaN values from arrays.

**Technical Definition:** Because `np.isnan()` returns a boolean mask, it can be inverted with `~` and used for boolean indexing to select only the non-NaN elements. For multidimensional arrays, `np.any(np.isnan(x), axis=1)` identifies rows containing at least one NaN, and `~np.any(np.isnan(x), axis=1)` selects rows where no NaN is present. This technique is essential for cleaning datasets before analysis or machine learning.

**Beginner-Friendly Explanation:** To remove NaN values, you create a mask that is `True` where the data is good (not NaN) and `False` where it's NaN, then use that mask to select only the good data. For rows in a 2-D array, you check each row for any NaN and keep only the rows that have none.

### Purposes

- To remove corrupted or incomplete records from datasets.
- To prepare clean data for analysis or machine learning.
- To filter out invalid measurements from sensor data.
- To align multiple arrays by removing rows where any array has NaN.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# 1-D: Remove NaN elements
clean = array[~np.isnan(array)]

# 2-D: Remove rows containing any NaN
mask = ~np.any(np.isnan(array), axis=1)
clean = array[mask]

# Multiple arrays: Remove rows where any array has NaN
mask = ~np.any(np.isnan(X), axis=1) & ~np.any(np.isnan(Y), axis=1)
X_clean = X[mask]
Y_clean = Y[mask]
```

**Component Breakdown:**

| Technique | Description |
|-----------|-------------|
| `~np.isnan(array)` | Mask of non-NaN elements. |
| `np.any(np.isnan(array), axis=1)` | `True` for rows with at least one NaN. |
| `~np.any(...)` | `True` for rows with no NaN. |
| Boolean indexing | Selects elements/rows where the mask is `True`. |

**Syntax Rules:**

- `np.isnan` must be applied to floating-point arrays.
- `np.any(..., axis=1)` checks each row for any NaN.
- For multiple arrays, combine masks with `&` to keep only rows that are clean in all arrays.

**Constraints and Limitations:**

- Removing NaN reduces the size of the array; ensure downstream code handles variable-length data.
- For structured arrays, check NaN per field.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Removing NaN from 1-D and 2-D Arrays**

```python
import numpy as np

# Step 2: Create a 1-D array with NaN.
a = np.array([1.0, 2.0, np.nan, 4.0, np.nan, 6.0])
print("Original 1-D array:", a)

# Step 3: Remove NaN elements.
clean_1d = a[~np.isnan(a)]
print("After removing NaN:", clean_1d)
print()

# Step 4: Create a 2-D array with NaN rows.
m = np.array([[1.0, 2.0],
              [np.nan, 4.0],
              [5.0, 6.0],
              [7.0, np.nan]])
print("Original 2-D array:\n", m)

# Step 5: Remove rows containing any NaN.
mask = ~np.any(np.isnan(m), axis=1)
print("Mask (rows without NaN):", mask)
clean_2d = m[mask]
print("After removing NaN rows:\n", clean_2d)
print()

# Step 6: Align multiple arrays.
X = np.array([[1.0, 2.0], [np.nan, 4.0], [5.0, 6.0]])
Y = np.array([10.0, 20.0, 30.0])
mask_align = ~np.any(np.isnan(X), axis=1)
print("X clean:\n", X[mask_align])
print("Y clean:", Y[mask_align])
```

**Expected Output:**

```
Original 1-D array: [ 1.  2. nan  4. nan  6.]
After removing NaN: [1. 2. 4. 6.]

Original 2-D array:
 [[ 1.  2.]
 [nan  4.]
 [ 5.  6.]
 [ 7. nan]]
Mask (rows without NaN): [ True False  True False]
After removing NaN rows:
 [[1. 2.]
 [5. 6.]]

X clean:
 [[1. 2.]
 [5. 6.]]
Y clean: [10. 30.]
```

**Why This Result Occurs:** `~np.isnan(a)` creates a mask that is `True` for non-NaN values, so `a[mask]` selects only the valid values. For the 2-D array, `np.any(np.isnan(m), axis=1)` identifies rows with at least one NaN, and the inverted mask selects clean rows. When aligning multiple arrays, the same mask is applied to both to keep corresponding rows.

### Real-World Cases

- **Data Preprocessing:** Removing incomplete records before training a machine learning model.
- **Sensor Networks:** Filtering out invalid readings from faulty sensors.
- **Financial Analysis:** Removing trading days with missing price data.
- **Survey Analysis:** Removing respondents with incomplete answers.

### References

- Stack Overflow: Remove rows containing NaN – https://stackoverflow.com
- Removing NaN values from NumPy arrays – https://browse.library.kiwix.org
- Masking and filtering with boolean arrays – https://numpy.org/doc/stable/user/basics.indexing.html

---

## Comparison Summary

| Function | Purpose | NaN Handling | All-NaN Behavior |
|----------|---------|-------------|------------------|
| `np.isnan` | Detect NaN | Returns boolean mask | — |
| `np.isinf` | Detect Inf | Returns boolean mask | — |
| `np.isfinite` | Detect finite | Returns boolean mask | — |
| `np.nan_to_num` | Replace special values | Replaces NaN/Inf | — |
| `np.nansum` | Sum ignoring NaN | NaN treated as 0 | Returns 0 |
| `np.nanmean` | Mean ignoring NaN | NaN excluded | Returns NaN + warning |
| `np.nanmin` / `np.nanmax` | Min/max ignoring NaN | NaN excluded | Returns NaN |
| `np.nanstd` / `np.nanvar` | Std/var ignoring NaN | NaN excluded | Returns NaN + warning |
| `np.nanargmin` / `np.nanargmax` | Indices of extremes | NaN excluded | Raises ValueError |

---

## Summary of Deprecated and Version-Specific Features

| Feature | Status | Recommendation |
|---------|--------|----------------|
| `np.nansum` on all-NaN slices | Returns 0 in NumPy > 1.9.0 | Check NumPy version; earlier versions return NaN |
| `keepdims` in `nanargmin`/`nanargmax` | New in NumPy 1.22.0 | Available in NumPy ≥ 1.22 |
| `where` and `initial` in `nansum` | New in NumPy 1.22.0 | Available in NumPy ≥ 1.22 |
| `np.nan_to_num` with `nan`, `posinf`, `neginf` | New in NumPy 1.17 | Use for custom replacements |
| Integer arrays with NaN | Not supported | Use floating-point arrays |

---

## Consolidated Reference List

- NumPy Miscellaneous (IEEE 754 special values) – https://numpy.org/doc/1.20/user/misc.html
- numpy.isnan – https://numpy.org/doc/stable/reference/generated/numpy.isnan.html
- numpy.isinf – https://numpy.org/doc/stable/reference/generated/numpy.isinf.html
- numpy.isfinite – https://numpy.org/doc/stable/reference/generated/numpy.isfinite.html
- numpy.nan_to_num – https://numpy.org/doc/1.20/reference/generated/numpy.nan_to_num.html
- numpy.nanmean – https://numpy.org/doc/1.20//reference/generated/numpy.nanmean.html
- numpy.nansum – https://numpy.org/doc/2.2/reference/generated/numpy.nansum.html
- numpy.nanmin – https://numpy.org/doc/stable/reference/generated/numpy.nanmin.html
- numpy.nanmax – https://numpy.org/doc/stable/reference/generated/numpy.nanmax.html
- numpy.nanstd – https://numpy.org/doc/stable/reference/generated/numpy.nanstd.html
- numpy.nanvar – https://numpy.org/doc/stable/reference/generated/numpy.nanvar.html
- numpy.nanargmin – https://numpy.org/doc/stable/reference/generated/numpy.nanargmin.html
- numpy.nanargmax – https://numpy.org/doc/stable/reference/generated/numpy.nanargmax.html
- Functions that ignore NaN – https://chromium.cpp.hybrid-analysis.googlesource.com
- FreeBSD IEEE 754 Manual – https://man.freebsd.org
- Stack Overflow: Removing NaN rows – https://stackoverflow.com