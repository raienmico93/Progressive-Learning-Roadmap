# NumPy Comparison Operations: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** NumPy comparison operations are element-wise ufuncs that compare corresponding elements of two arrays (or an array and a scalar) and return a boolean array indicating where the comparison is true.

**Technical Definition:** NumPy implements the six standard comparison operators (`==`, `!=`, `<`, `>`, `<=`, `>=`) as element-wise universal functions (ufuncs): `np.equal`, `np.not_equal`, `np.less`, `np.greater`, `np.less_equal`, and `np.greater_equal`. Each accepts two array-like inputs, broadcasts them to a common shape, and returns a boolean array of the same broadcast shape. For scalar inputs, a scalar boolean is returned.

**Beginner-Friendly Explanation:** Comparison operations let you ask questions about your data: "Which numbers are greater than 5?", "Which elements are equal to zero?", "Which values are not equal to each other?" NumPy answers these questions for every element at once, giving you back a true/false mask that you can use to filter, count, or modify your data.

### Key Characteristics

- **Element-Wise:** Each comparison is performed independently on corresponding elements.
- **Broadcasting:** Arrays of different shapes are aligned using NumPy's broadcasting rules.
- **Boolean Output:** The result is always a boolean array (or scalar), regardless of input dtype.
- **Ufunc Foundation:** Every comparison operator has a corresponding ufunc that can be called explicitly.
- **NaN Handling:** NaN is never equal to anything, including itself; use `np.isnan()` for NaN checks.
- **Reductions:** `np.all()` and `np.any()` collapse boolean masks to a single truth value.

### Prerequisites

- NumPy installed (`pip install numpy`).
- Basic Python syntax and understanding of variables.
- Familiarity with NumPy array creation and the `import numpy as np` convention.
- Conceptual understanding of broadcasting.

### Related Programming Areas

- **Data Filtering:** Selecting rows or elements that meet a condition.
- **Masking:** Creating boolean masks for indexing and assignment.
- **Validation:** Checking whether arrays satisfy certain properties.
- **Machine Learning:** Computing accuracy, error rates, and confusion matrices.
- **Scientific Computing:** Detecting anomalies, thresholding, and segmenting data.

### Core Concepts / Features

The following operations are explored in detail: (1) Equality, (2) Inequality, (3) Greater than / Less than, (4) Greater/Less than or equal, (5) Array-wide logical reductions (`np.all`, `np.any`), and (6) NaN comparison quirks.

---

## Core Concept 1: Equality

### Definitions

**Core Definition:** Equality comparison checks whether corresponding elements of two arrays are equal, returning a boolean mask.

**Technical Definition:** `numpy.equal(x1, x2)` returns `(x1 == x2)` element-wise. The `==` operator can be used as a shorthand. The result is a boolean array unless both inputs are scalars, in which case a scalar boolean is returned. What is compared are values, not types; an integer `1` and a float `1.0` evaluate as equal.

**Beginner-Friendly Explanation:** Equality comparison asks "are these two things the same?" For arrays, it asks the question for every pair of corresponding elements. You get back a true/false answer for each position.

### Purposes

- To find matching elements between two arrays.
- To create masks for selecting elements equal to a specific value.
- To verify that two arrays are element-wise identical.
- To implement one-hot encoding validation or label matching.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Using the == operator
result = array1 == array2

# Using the np.equal ufunc
result = np.equal(x1, x2, out=None, where=True)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `x1, x2` | Input arrays or scalars. Must be broadcastable to a common shape. |
| `out` | Optional output array for the result. |
| `where` | Boolean array selecting which elements to compute. |
| Returns | Boolean array (or scalar if both inputs are scalars). |

**Syntax Rules:**

- The `==` operator and `np.equal()` are functionally identical for ndarrays.
- Shapes must be broadcastable: either equal, or one dimension is 1, or one array has fewer dimensions.
- What is compared are values, not types (e.g., `1 == 1.0` is `True`).

**Constraints and Limitations:**

- For floating-point arrays, exact equality can be misleading due to precision issues.
- For object arrays, equality uses Python's `==` operator, which may not behave as expected for custom objects.
- NaN is never equal to anything, including itself (see Core Concept 6).

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Equality Comparison Between Arrays and Scalars**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create two arrays.
a = np.array([1, 2, 3, 4, 5])
b = np.array([1, 0, 3, 0, 5])
print("Array a:", a)
print("Array b:", b)
print()

# Step 3: Compare arrays element-wise.
equal_mask = a == b
print("a == b:", equal_mask)
print("  (True where elements match)")
print()

# Step 4: Compare array to a scalar.
scalar_mask = a == 3
print("a == 3:", scalar_mask)
print()

# Step 5: Use np.equal explicitly.
ufunc_mask = np.equal(a, b)
print("np.equal(a, b):", ufunc_mask)
```

**Expected Output:**

```
Array a: [1 2 3 4 5]
Array b: [1 0 3 0 5]

a == b: [ True False  True False  True]
  (True where elements match)

a == 3: [False False  True False False]

np.equal(a, b): [ True False  True False  True]
```

**Why This Result Occurs:** The `==` operator compares each element of `a` with the corresponding element of `b`. Where they match (indices 0, 2, 4), the result is `True`; elsewhere, `False`. Comparing to a scalar broadcasts the scalar to all positions.

### Real-World Cases

- **Label Matching:** Checking whether predicted labels match true labels in classification.
- **Data Validation:** Verifying that two data sources contain identical values.
- **Filtering:** Selecting rows where a specific column equals a target value.

### References

- numpy.equal – https://numpy.org/doc/1.26/reference/generated/numpy.equal.html

---

## Core Concept 2: Inequality

### Definitions

**Core Definition:** Inequality comparison checks whether corresponding elements of two arrays are not equal, returning a boolean mask.

**Technical Definition:** `numpy.not_equal(x1, x2)` returns `(x1 != x2)` element-wise. The `!=` operator can be used as a shorthand.

**Beginner-Friendly Explanation:** Inequality is the opposite of equality—it asks "are these two things different?" It returns `True` where elements differ and `False` where they match.

### Purposes

- To find mismatches between two arrays.
- To create masks for selecting elements that differ from a reference value.
- To detect changes between two versions of data.
- To implement error detection in data pipelines.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
result = array1 != array2
result = np.not_equal(x1, x2)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `x1, x2` | Input arrays or scalars. |
| Returns | Boolean array where `True` indicates inequality. |

**Syntax Rules:**

- `!=` is the shorthand for `np.not_equal`.
- The result is the logical negation of `==`.

**Constraints and Limitations:**

- NaN is not equal to anything, so `NaN != x` is always `True` for any `x`, including `NaN` itself.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Inequality Comparison**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create arrays.
a = np.array([1, 2, 3, 4])
b = np.array([1, 0, 3, 0])
print("a:", a)
print("b:", b)
print()

# Step 3: Inequality comparison.
neq = a != b
print("a != b:", neq)
print()

# Step 4: Compare to a scalar.
neq_scalar = a != 2
print("a != 2:", neq_scalar)
```

**Expected Output:**

```
a: [1 2 3 4]
b: [1 0 3 0]

a != b: [False  True False  True]

a != 2: [ True False  True  True]
```

**Why This Result Occurs:** `a != b` is `True` where the elements differ (indices 1 and 3) and `False` where they match (indices 0 and 2). `a != 2` is `True` everywhere except where `a` equals 2.

### Real-World Cases

- **Change Detection:** Finding rows that changed between two datasets.
- **Error Detection:** Identifying mismatched values in data validation.
- **Anomaly Detection:** Flagging values that differ from an expected baseline.

### References

- numpy.not_equal – https://numpy.org/doc/stable/reference/generated/numpy.not_equal.html

---

## Core Concept 3: Greater Than / Less Than

### Definitions

**Core Definition:** Greater-than and less-than comparisons check whether each element of one array is strictly greater or strictly less than the corresponding element of another array.

**Technical Definition:** `numpy.greater(x1, x2)` returns `(x1 > x2)` element-wise. `numpy.less(x1, x2)` returns `(x1 < x2)` element-wise. The operators `>` and `<` are shorthands for these ufuncs.

**Beginner-Friendly Explanation:** These comparisons ask "is this number bigger than that one?" or "is this number smaller than that one?" They return true/false masks showing where the condition holds.

### Purposes

- To threshold data (e.g., find values above a limit).
- To compare scores, rankings, or measurements.
- To implement range checks (e.g., values between a lower and upper bound).
- To create masks for conditional assignments.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
result = array1 > array2
result = np.greater(x1, x2)

result = array1 < array2
result = np.less(x1, x2)
```

**Component Breakdown:**

| Operator | Ufunc | Description |
|----------|-------|-------------|
| `>` | `np.greater` | True where `x1 > x2`. |
| `<` | `np.less` | True where `x1 < x2`. |

**Syntax Rules:**

- Both operands must be broadcastable.
- The result is a boolean array.

**Constraints and Limitations:**

- Comparisons with NaN always return `False` (except `!=`).

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Greater Than and Less Than Comparisons**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create an array.
x = np.array([1, 5, 3, 8, 2, 7])
print("Array x:", x)
print()

# Step 3: Greater than comparison.
greater_mask = x > 4
print("x > 4:", greater_mask)
print()

# Step 4: Less than comparison.
less_mask = x < 4
print("x < 4:", less_mask)
print()

# Step 5: Combine with logical operators.
between = (x > 2) & (x < 7)
print("(x > 2) & (x < 7):", between)
```

**Expected Output:**

```
Array x: [1 5 3 8 2 7]

x > 4: [False  True False  True False  True]

x < 4: [ True False  True False  True False]

(x > 2) & (x < 7): [False  True  True False False False]
```

**Why This Result Occurs:** `x > 4` is `True` for elements 5, 8, and 7. `x < 4` is `True` for 1, 3, and 2. The combined mask uses `&` (element-wise AND) to find elements that satisfy both conditions simultaneously.

### Real-World Cases

- **Thresholding:** Selecting pixels above a brightness threshold in image processing.
- **Scoring:** Finding students who scored above 90 on an exam.
- **Finance:** Identifying trades above a certain value.

### References

- numpy.greater – https://numpy.org/doc/stable/reference/generated/numpy.greater.html
- numpy.less – https://numpy.org/doc/stable/reference/generated/numpy.less.html

---

## Core Concept 4: Greater/Less Than or Equal

### Definitions

**Core Definition:** Greater-than-or-equal and less-than-or-equal comparisons check whether each element is greater than or equal to, or less than or equal to, the corresponding element.

**Technical Definition:** `numpy.greater_equal(x1, x2)` returns `(x1 >= x2)` element-wise. `numpy.less_equal(x1, x2)` returns `(x1 <= x2)` element-wise. The operators `>=` and `<=` are shorthands.

**Beginner-Friendly Explanation:** These are like the strict comparisons but include equality. "Is this number at least as big as that one?" or "Is this number at most that big?"

### Purposes

- To implement inclusive range checks.
- To find elements that meet or exceed a threshold.
- To compare rankings where ties matter.
- To validate that values are within acceptable bounds.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
result = array1 >= array2
result = np.greater_equal(x1, x2)

result = array1 <= array2
result = np.less_equal(x1, x2)
```

**Component Breakdown:**

| Operator | Ufunc | Description |
|----------|-------|-------------|
| `>=` | `np.greater_equal` | True where `x1 >= x2`. |
| `<=` | `np.less_equal` | True where `x1 <= x2`. |

**Syntax Rules:**

- The `>=` operator is a shorthand for `np.greater_equal`.
- The `<=` operator is a shorthand for `np.less_equal`.

**Constraints and Limitations:**

- NaN comparisons always return `False`.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Inclusive Comparisons**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create arrays.
a = np.array([4, 2, 1])
b = np.array([2, 2, 2])
print("a:", a)
print("b:", b)
print()

# Step 3: Greater than or equal.
ge = a >= b
print("a >= b:", ge)
print()

# Step 4: Less than or equal.
le = a <= b
print("a <= b:", le)
print()

# Step 5: Inclusive range check.
x = np.array([1, 5, 3, 8, 2, 7])
in_range = (x >= 2) & (x <= 7)
print("(x >= 2) & (x <= 7):", in_range)
```

**Expected Output:**

```
a: [4 2 1]
b: [2 2 2]

a >= b: [ True  True False]

a <= b: [False  True  True]

(x >= 2) & (x <= 7): [False  True  True False  True  True]
```

**Why This Result Occurs:** `a >= b` is `True` where `a` is at least as large as `b`. `a <= b` is `True` where `a` is at most as large as `b`. The inclusive range check finds elements between 2 and 7 (inclusive).

### Real-World Cases

- **Grade Boundaries:** Finding students who scored 60 or above.
- **Inventory Management:** Identifying items with stock levels at or below the reorder point.
- **Finance:** Finding transactions at or above a threshold.

### References

- numpy.greater_equal – https://numpy.org/doc/stable/reference/generated/numpy.greater_equal.html
- numpy.less_equal – https://numpy.org/doc/stable/reference/generated/numpy.less_equal.html

---

## Core Concept 5: Array-Wide Logical Reductions

### Definitions

**Core Definition:** Array-wide logical reductions collapse a boolean mask into a single truth value using `np.all()` (all elements must be `True`) or `np.any()` (at least one element must be `True`).

**Technical Definition:** `numpy.all(a, axis=None, out=None, keepdims=False, *, where=True)` tests whether all array elements along a given axis evaluate to `True`. `numpy.any(a, axis=None, out=None, keepdims=False, *, where=True)` tests whether at least one element evaluates to `True`. Both return a single boolean if `axis` is `None`. Not a Number (NaN), positive infinity, and negative infinity evaluate to `True` because these are not equal to zero.

**Beginner-Friendly Explanation:** These functions answer two simple questions about a boolean mask: "Is everything True?" (`np.all`) and "Is anything True?" (`np.any`). They're useful for validation checks and conditional logic.

### Purposes

- To validate that all elements of an array satisfy a condition.
- To check whether any element satisfies a condition.
- To implement short-circuit-like logic for arrays.
- To reduce boolean masks to scalar truth values for conditional statements.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.all(a, axis=None, out=None, keepdims=False, *, where=True)
np.any(a, axis=None, out=None, keepdims=False, *, where=True)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `a` | Input array or object convertible to an array. |
| `axis` | Axis or axes along which the reduction is performed. Default `None` (all axes). |
| `out` | Optional output array. |
| `keepdims` | If `True`, reduced axes are kept as dimensions of size one. |
| `where` | Elements to include in the check. |
| Returns | Boolean scalar or array. |

**Syntax Rules:**

- `axis=None` performs the reduction over all dimensions.
- `axis` can be an integer or a tuple of integers.
- NaN, Inf, and -Inf evaluate to `True` because they are not zero.

**Constraints and Limitations:**

- NumPy's `any` and `all` do not short-circuit like Python's built-in versions; they evaluate all elements.
- For object dtype arrays, behavior may differ in older NumPy versions.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Using np.all() and np.any()**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create a boolean mask.
mask = np.array([True, True, False, True])
print("Mask:", mask)
print()

# Step 3: Check if all elements are True.
all_true = np.all(mask)
print("np.all(mask):", all_true)
print("  (False because one element is False)")
print()

# Step 4: Check if any element is True.
any_true = np.any(mask)
print("np.any(mask):", any_true)
print("  (True because at least one element is True)")
print()

# Step 5: Use along an axis.
mask_2d = np.array([[True, False],
                     [True, True]])
print("2-D mask:\n", mask_2d)
print("np.all(mask_2d, axis=0):", np.all(mask_2d, axis=0))
print("  (column-wise: all True?)")
print("np.any(mask_2d, axis=1):", np.any(mask_2d, axis=1))
print("  (row-wise: any True?)")
```

**Expected Output:**

```
Mask: [ True  True False  True]

np.all(mask): False
  (False because one element is False)

np.any(mask): True
  (True because at least one element is True)

2-D mask:
 [[ True False]
 [ True  True]]
np.all(mask_2d, axis=0): [ True False]
  (column-wise: all True?)
np.any(mask_2d, axis=1): [ True  True]
  (row-wise: any True?)
```

**Why This Result Occurs:** `np.all(mask)` returns `False` because the third element is `False`. `np.any(mask)` returns `True` because at least one element is `True`. With `axis=0`, the reduction is performed down each column; with `axis=1`, across each row.

### Real-World Cases

- **Validation:** Checking that all values in a dataset are within expected bounds.
- **Error Detection:** Checking whether any value is NaN or infinite.
- **Machine Learning:** Computing whether all predictions are correct or whether any prediction is wrong.

### References

- numpy.all – https://numpy.org/doc/2.2/numpy-ref.pdf#277#172
- numpy.any – https://numpy.org/doc/2.2/numpy-ref.pdf#277#172

---

## Core Concept 6: NaN Comparison Quirks

### Definitions

**Core Definition:** NaN (Not a Number) has the unique property that it is never equal to anything, including itself, so `NaN == NaN` evaluates to `False`. Special functions like `np.isnan()` must be used to detect NaN values.

**Technical Definition:** According to IEEE 754, any comparison involving NaN (except `!=`) returns `False`. `NaN != NaN` returns `True`. NumPy provides `np.isnan(x)` to test element-wise for NaN, returning a boolean array. Related functions include `np.isinf`, `np.isfinite`, and `np.nan_to_num`.

**Beginner-Friendly Explanation:** NaN is a special "not a number" value that represents missing or undefined data. Because it's not a number, it doesn't equal anything—not even another NaN. To check for NaN, you must use `np.isnan()` instead of `==`.

### Purposes

- To detect missing or invalid values in datasets.
- To handle NaN values before performing calculations.
- To filter out NaN values from arrays.
- To implement NaN-aware computations.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Detect NaN
mask = np.isnan(x)

# Replace NaN
x[np.isnan(x)] = replacement

# Check if any NaN exists
has_nan = np.isnan(x).any()
```

**Component Breakdown:**

| Function | Description |
|----------|-------------|
| `np.isnan(x)` | Returns `True` where `x` is NaN. |
| `np.isinf(x)` | Returns `True` where `x` is infinite. |
| `np.isfinite(x)` | Returns `True` where `x` is neither NaN nor infinite. |
| `np.nan_to_num(x)` | Replaces NaN with 0, Inf with large finite values. |

**Syntax Rules:**

- Never use `==` to test for NaN; it always returns `False`.
- Use `np.isnan()` for element-wise NaN detection.
- Use `np.isnan(x).any()` to check whether an array contains any NaN.

**Constraints and Limitations:**

- `np.isnan()` only works on floating-point and complex arrays; it raises an error for integer arrays.
- NaN propagates through most arithmetic operations.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Detecting and Handling NaN**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create an array with NaN.
arr = np.array([1.0, np.nan, 3.0, np.nan, 5.0])
print("Array with NaN:", arr)
print()

# Step 3: Demonstrate that NaN == NaN is False.
print("np.nan == np.nan:", np.nan == np.nan)
print("arr[1] == arr[1]:", arr[1] == arr[1])
print()

# Step 4: Detect NaN using np.isnan().
nan_mask = np.isnan(arr)
print("np.isnan(arr):", nan_mask)
print()

# Step 5: Check if any NaN exists.
print("np.isnan(arr).any():", np.isnan(arr).any())
print()

# Step 6: Replace NaN values.
arr[np.isnan(arr)] = 0.0
print("After replacing NaN with 0:", arr)
print()

# Step 7: Use nan-aware functions.
arr2 = np.array([1.0, np.nan, 3.0])
print("np.nansum(arr2):", np.nansum(arr2))
print("  (sum ignores NaN)")
```

**Expected Output:**

```
Array with NaN: [ 1. nan  3. nan  5.]

np.nan == np.nan: False
arr[1] == arr[1]: False

np.isnan(arr): [False  True False  True False]

np.isnan(arr).any(): True

After replacing NaN with 0: [1. 0. 3. 0. 5.]

np.nansum(arr2): 4.0
  (sum ignores NaN)
```

**Why This Result Occurs:** NaN is never equal to anything, including itself, so `arr[1] == arr[1]` is `False`. `np.isnan()` correctly identifies NaN positions. Setting `arr[np.isnan(arr)] = 0.0` replaces NaN with 0. `np.nansum()` ignores NaN values when computing the sum.

### Real-World Cases

- **Data Cleaning:** Detecting and removing or replacing missing values.
- **Financial Analysis:** Handling missing price data.
- **Sensor Data:** Filtering out invalid readings.

### References

- numpy.isnan – https://numpy.org/doc/2.0/reference/generated/numpy.isnan.html
- NumPy Miscellaneous – https://numpy.org/devdocs/user/misc.html

---

## Comparison of Comparison Operations

| Operation | Operator | Ufunc | Description |
|-----------|----------|-------|-------------|
| Equality | `==` | `np.equal` | True where elements are equal. |
| Inequality | `!=` | `np.not_equal` | True where elements differ. |
| Greater than | `>` | `np.greater` | True where `x1 > x2`. |
| Less than | `<` | `np.less` | True where `x1 < x2`. |
| Greater or equal | `>=` | `np.greater_equal` | True where `x1 >= x2`. |
| Less or equal | `<=` | `np.less_equal` | True where `x1 <= x2`. |
| NaN check | — | `np.isnan` | True where value is NaN. |
| All True | — | `np.all` | True if all elements are True. |
| Any True | — | `np.any` | True if at least one element is True. |

---

## Consolidated Reference List

- numpy.equal – https://numpy.org/doc/1.26/reference/generated/numpy.equal.html
- numpy.not_equal – https://numpy.org/doc/stable/reference/generated/numpy.not_equal.html
- numpy.greater – https://numpy.org/doc/stable/reference/generated/numpy.greater.html
- numpy.less – https://numpy.org/doc/stable/reference/generated/numpy.less.html
- numpy.greater_equal – https://numpy.org/doc/stable/reference/generated/numpy.greater_equal.html
- numpy.less_equal – https://numpy.org/doc/stable/reference/generated/numpy.less_equal.html
- numpy.all – https://numpy.org/doc/2.2/numpy-ref.pdf#277#172
- numpy.any – https://numpy.org/doc/2.2/numpy-ref.pdf#277#172
- numpy.isnan – https://numpy.org/doc/2.0/reference/generated/numpy.isnan.html
- NumPy Miscellaneous (IEEE 754 special values) – https://numpy.org/devdocs/user/misc.html