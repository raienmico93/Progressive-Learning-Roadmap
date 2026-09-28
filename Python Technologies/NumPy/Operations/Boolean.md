# NumPy Boolean Operations: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Boolean operations in NumPy are element-wise logical functions that combine or invert boolean arrays (masks), producing new boolean arrays that represent the result of logical AND, OR, NOT, or XOR operations.

**Technical Definition:** NumPy implements logical operations as universal functions (ufuncs): `np.logical_and`, `np.logical_or`, `np.logical_not`, and `np.logical_xor`. These ufuncs accept array-like inputs, broadcast them to a common shape, and return a boolean array of the broadcast shape. For boolean ndarrays, the bitwise operators `&`, `|`, `~`, and `^` are equivalent shorthands for the corresponding logical ufuncs.

**Beginner-Friendly Explanation:** Boolean operations let you combine conditions. If you have one mask that says "these numbers are greater than 5" and another that says "these numbers are less than 10," you can use `&` to find numbers that satisfy both conditions. Boolean operations are the building blocks for filtering, masking, and conditional selection in NumPy.

### Key Characteristics

- **Element-Wise:** Each logical operation is applied independently to corresponding elements.
- **Broadcasting:** Inputs of different shapes are aligned using NumPy's broadcasting rules.
- **Boolean Output:** The result is always a boolean array (or scalar), regardless of input dtype.
- **Shorthand Operators:** For boolean arrays, `&`, `|`, `~`, `^` are equivalent to the logical ufuncs.
- **Operator Precedence:** Bitwise operators have higher precedence than comparison operators, so parentheses are essential when combining masks.

### Prerequisites

- NumPy installed (`pip install numpy`).
- Basic Python syntax and understanding of variables.
- Familiarity with NumPy array creation and comparison operations.
- Conceptual understanding of broadcasting and boolean indexing.

### Related Programming Areas

- **Data Filtering:** Combining multiple conditions to select subsets of data.
- **Masking:** Creating and manipulating boolean masks for indexing and assignment.
- **Machine Learning:** Computing logical combinations of prediction thresholds.
- **Image Processing:** Segmenting regions based on multiple color or intensity conditions.

### Core Concepts / Features

The following operations are explored in detail: (1) `logical_and` (`&`), (2) `logical_or` (`|`), (3) `logical_not` (`~`), (4) `logical_xor` (`^`), and (5) Combining boolean masks.

---

## Core Concept 1: logical_and (&)

### Definitions

**Core Definition:** `logical_and` computes the element-wise logical AND of two arrays, returning `True` only where both corresponding elements are `True`.

**Technical Definition:** `numpy.logical_and(x1, x2, /, out=None, *, where=True, casting='same_kind', order='K', dtype=None, subok=True)` computes the truth value of `x1 AND x2` element-wise. The `&` operator can be used as a shorthand for `np.logical_and` on boolean ndarrays.

**Beginner-Friendly Explanation:** `logical_and` is the "both must be true" operation. If you have two conditions, `&` gives you `True` only where both conditions are satisfied simultaneously.

### Purposes

- To combine two boolean masks into one that satisfies both conditions.
- To implement intersection-like filtering (elements must meet all criteria).
- To narrow down selections by applying multiple constraints.
- To build compound conditions for boolean indexing.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
result = np.logical_and(x1, x2, out=None, where=True)
result = array1 & array2
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `x1, x2` | Input arrays or scalars. Must be broadcastable. |
| `out` | Optional output array. |
| `where` | Boolean condition selecting which elements to compute. |
| Returns | Boolean array of the broadcast shape. |

**Syntax Rules:**

- The `&` operator is a shorthand for `np.logical_and` on boolean ndarrays.
- For integer arrays, `&` performs bitwise AND, not logical AND.
- Both inputs must be broadcastable to a common shape.

**Constraints and Limitations:**

- `&` has higher precedence than comparison operators; use parentheses around comparisons.
- `&` cannot be used with Python scalars the same way as `and`.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Combining Two Masks with logical_and**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create an array of values.
x = np.array([1, 5, 3, 8, 2, 7, 4, 9, 6])
print("Array:", x)
print()

# Step 3: Create two individual masks.
mask_gt_3 = x > 3
mask_lt_8 = x < 8
print("x > 3:", mask_gt_3)
print("x < 8:", mask_lt_8)
print()

# Step 4: Combine with np.logical_and.
combined = np.logical_and(mask_gt_3, mask_lt_8)
print("np.logical_and(x > 3, x < 8):", combined)
print()

# Step 5: Use the & operator as shorthand.
combined_amp = (x > 3) & (x < 8)
print("(x > 3) & (x < 8):", combined_amp)
print()

# Step 6: Apply the combined mask.
result = x[combined]
print("x[(x > 3) & (x < 8)]:", result)
print("  (values between 4 and 7 inclusive)")
```

**Expected Output:**

```
Array: [1 5 3 8 2 7 4 9 6]

x > 3: [False  True False  True False  True  True  True  True]
x < 8: [ True  True  True False  True  True  True False  True]

np.logical_and(x > 3, x < 8): [False  True False False False  True  True False  True]

(x > 3) & (x < 8): [False  True False False False  True  True False  True]

x[(x > 3) & (x < 8)]: [5 7 4 6]
  (values between 4 and 7 inclusive)
```

**Why This Result Occurs:** The combined mask is `True` only where both `x > 3` and `x < 8` are `True`. This selects values 5, 7, 4, and 6 (indices 1, 5, 6, and 8), which are the elements greater than 3 and less than 8.

### Real-World Cases

- **Data Filtering:** Selecting rows where age > 18 AND income > 50,000.
- **Image Processing:** Finding pixels that are both bright (R > 200) and green (G > 150).
- **Quality Control:** Identifying products that meet both size and weight specifications.

### References

- numpy.logical_and – https://numpy.org/doc/1.26/reference/generated/numpy.logical_and.html
- Logic functions – https://numpy.org/devdocs/reference/routines.logic.html

---

## Core Concept 2: logical_or (|)

### Definitions

**Core Definition:** `logical_or` computes the element-wise logical OR of two arrays, returning `True` where at least one corresponding element is `True`.

**Technical Definition:** `numpy.logical_or(x1, x2, /, out=None, *, where=True, casting='same_kind', order='K', dtype=None, subok=True)` computes the truth value of `x1 OR x2` element-wise. The `|` operator can be used as a shorthand for boolean ndarrays.

**Beginner-Friendly Explanation:** `logical_or` is the "either one can be true" operation. If you have two conditions, `|` gives you `True` where either condition is satisfied.

### Purposes

- To combine two boolean masks into one that satisfies at least one condition.
- To implement union-like filtering (elements can meet any of several criteria).
- To broaden selections by applying alternative constraints.
- To build flexible conditions for boolean indexing.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
result = np.logical_or(x1, x2, out=None, where=True)
result = array1 | array2
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `x1, x2` | Input arrays or scalars. Must be broadcastable. |
| `out` | Optional output array. |
| `where` | Boolean condition selecting which elements to compute. |
| Returns | Boolean array of the broadcast shape. |

**Syntax Rules:**

- The `|` operator is a shorthand for `np.logical_or` on boolean ndarrays.
- For integer arrays, `|` performs bitwise OR.
- Both inputs must be broadcastable.

**Constraints and Limitations:**

- `|` has higher precedence than comparison operators; use parentheses.
- `|` cannot be used with Python scalars the same way as `or`.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Combining Two Masks with logical_or**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create an array of values.
x = np.array([1, 5, 3, 8, 2, 7, 4, 9, 6])
print("Array:", x)
print()

# Step 3: Create two individual masks.
mask_lt_3 = x < 3
mask_gt_7 = x > 7
print("x < 3:", mask_lt_3)
print("x > 7:", mask_gt_7)
print()

# Step 4: Combine with np.logical_or.
combined = np.logical_or(mask_lt_3, mask_gt_7)
print("np.logical_or(x < 3, x > 7):", combined)
print()

# Step 5: Use the | operator as shorthand.
combined_pipe = (x < 3) | (x > 7)
print("(x < 3) | (x > 7):", combined_pipe)
print()

# Step 6: Apply the combined mask.
result = x[combined]
print("x[(x < 3) | (x > 7)]:", result)
print("  (values less than 3 or greater than 7)")
```

**Expected Output:**

```
Array: [1 5 3 8 2 7 4 9 6]

x < 3: [ True False False False  True False False False False]
x > 7: [False False False  True False False False  True False]

np.logical_or(x < 3, x > 7): [ True False False  True  True False False  True False]

(x < 3) | (x > 7): [ True False False  True  True False False  True False]

x[(x < 3) | (x > 7)]: [1 8 2 9]
  (values less than 3 or greater than 7)
```

**Why This Result Occurs:** The combined mask is `True` where either `x < 3` or `x > 7` is `True`. This selects values 1, 8, 2, and 9 (indices 0, 3, 4, and 7), which are the elements outside the range [3, 7].

### Real-World Cases

- **Data Filtering:** Selecting rows where status is "active" OR "pending".
- **Anomaly Detection:** Flagging values that are either too high or too low.
- **Image Processing:** Finding pixels that are either very dark or very bright.

### References

- numpy.logical_or – https://numpy.org/doc/1.20/reference/generated/numpy.logical_or.html

---

## Core Concept 3: logical_not (~)

### Definitions

**Core Definition:** `logical_not` inverts a boolean array element-wise, turning `True` into `False` and `False` into `True`.

**Technical Definition:** `numpy.logical_not(x, /, out=None, *, where=True, casting='same_kind', order='K', dtype=None, subok=True)` computes the truth value of `NOT x` element-wise. The `~` operator can be used as a shorthand for boolean ndarrays.

**Beginner-Friendly Explanation:** `logical_not` is the "opposite" operation. It flips every `True` to `False` and every `False` to `True`. It's useful when you want to select everything that does not meet a condition.

### Purposes

- To invert a boolean mask (select elements that do not satisfy a condition).
- To negate a compound condition.
- To implement exclusion filtering.
- To complement masks for set operations.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
result = np.logical_not(x, out=None, where=True)
result = ~array
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `x` | Input array or scalar. |
| `out` | Optional output array. |
| `where` | Boolean condition selecting which elements to compute. |
| Returns | Boolean array with the same shape as `x`. |

**Syntax Rules:**

- The `~` operator is a shorthand for `np.logical_not` on boolean ndarrays.
- For integer arrays, `~` performs bitwise NOT (one's complement).
- The result has the same shape as the input.

**Constraints and Limitations:**

- `~` on integer arrays produces bitwise negation, not logical negation.
- `~` cannot be used with Python `not` for arrays.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Inverting a Boolean Mask**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create an array of values.
x = np.array([1, 5, 3, 8, 2, 7])
print("Array:", x)
print()

# Step 3: Create a mask for values greater than 4.
mask = x > 4
print("x > 4:", mask)
print()

# Step 4: Invert the mask with np.logical_not.
inverted = np.logical_not(mask)
print("np.logical_not(x > 4):", inverted)
print()

# Step 5: Use the ~ operator as shorthand.
inverted_tilde = ~(x > 4)
print("~(x > 4):", inverted_tilde)
print()

# Step 6: Apply the inverted mask.
result = x[~(x > 4)]
print("x[~(x > 4)]:", result)
print("  (values not greater than 4)")
```

**Expected Output:**

```
Array: [1 5 3 8 2 7]

x > 4: [False  True False  True False  True]

np.logical_not(x > 4): [ True False  True False  True False]

~(x > 4): [ True False  True False  True False]

x[~(x > 4)]: [1 3 2]
  (values not greater than 4)
```

**Why This Result Occurs:** `logical_not` flips the mask, so `True` becomes `False` and vice versa. Applying the inverted mask selects values 1, 3, and 2, which are not greater than 4.

### Real-World Cases

- **Data Exclusion:** Removing rows that meet a certain condition (e.g., `~is_outlier`).
- **Image Processing:** Selecting pixels that are not part of a background mask.
- **Quality Control:** Filtering out defective items (e.g., `~(status == 'defective')`).

### References

- numpy.logical_not – https://numpy.org/doc/1.20/reference/generated/numpy.logical_not.html

---

## Core Concept 4: logical_xor (^)

### Definitions

**Core Definition:** `logical_xor` computes the element-wise logical exclusive OR of two arrays, returning `True` where exactly one of the corresponding elements is `True`.

**Technical Definition:** `numpy.logical_xor(x1, x2, /, out=None, *, where=True, casting='same_kind', order='K', dtype=None, subok=True)` computes the truth value of `x1 XOR x2` element-wise. The `^` operator can be used as a shorthand for boolean ndarrays.

**Beginner-Friendly Explanation:** `logical_xor` is the "one or the other, but not both" operation. It gives `True` when exactly one of the two conditions is satisfied—not both, and not neither.

### Purposes

- To find elements that satisfy exactly one of two conditions.
- To implement exclusive filtering.
- To detect differences between two boolean masks.
- To implement parity checks in binary operations.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
result = np.logical_xor(x1, x2, out=None, where=True)
result = array1 ^ array2
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `x1, x2` | Input arrays or scalars. Must be broadcastable. |
| `out` | Optional output array. |
| `where` | Boolean condition selecting which elements to compute. |
| Returns | Boolean array of the broadcast shape. |

**Syntax Rules:**

- The `^` operator is a shorthand for `np.logical_xor` on boolean ndarrays.
- For integer arrays, `^` performs bitwise XOR.
- Both inputs must be broadcastable.

**Constraints and Limitations:**

- XOR is `True` only when the inputs differ.
- `^` has higher precedence than comparison operators.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Using logical_xor to Find Exclusive Differences**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create two boolean masks.
a = np.array([True, True, False, False])
b = np.array([True, False, True, False])
print("Mask a:", a)
print("Mask b:", b)
print()

# Step 3: Compute XOR with np.logical_xor.
xor_result = np.logical_xor(a, b)
print("np.logical_xor(a, b):", xor_result)
print("  (True where a and b differ)")
print()

# Step 4: Use the ^ operator as shorthand.
xor_caret = a ^ b
print("a ^ b:", xor_caret)
print()

# Step 5: Apply to numeric arrays with comparison.
x = np.arange(5)
xor_mask = np.logical_xor(x < 1, x > 3)
print("x =", x)
print("np.logical_xor(x < 1, x > 3):", xor_mask)
print("  (values less than 1 OR greater than 3, but not both)")
```

**Expected Output:**

```
Mask a: [ True  True False False]
Mask b: [ True False  True False]

np.logical_xor(a, b): [False  True  True False]
  (True where a and b differ)

a ^ b: [False  True  True False]

x = [0 1 2 3 4]
np.logical_xor(x < 1, x > 3): [ True False False False  True]
  (values less than 1 OR greater than 3, but not both)
```

**Why This Result Occurs:** XOR is `True` when exactly one input is `True`. In the first example, positions where `a` and `b` differ (indices 1 and 2) produce `True`. In the second example, values less than 1 or greater than 3 (but not both) are selected: 0 and 4.

### Real-World Cases

- **Difference Detection:** Finding elements that differ between two boolean masks.
- **Quality Control:** Identifying items that pass exactly one of two tests.
- **Game Development:** Implementing toggle logic where exactly one state must be active.

### References

- numpy.logical_xor – https://numpy.org/doc/1.20/reference/generated/numpy.logical_xor.html

---

## Core Concept 5: Combining Boolean Masks

### Definitions

**Core Definition:** Combining boolean masks means using logical operators (`&`, `|`, `~`, `^`) to construct compound filters that express complex conditions.

**Technical Definition:** When combining comparison expressions with bitwise logical operators, parentheses are required around each comparison because bitwise operators (`&`, `|`, `~`, `^`) have higher precedence than comparison operators (`<`, `>`, `==`, etc.). Without parentheses, the expression is evaluated incorrectly (e.g., `x > 5 & x < 10` is parsed as `x > (5 & x) < 10`).

**Beginner-Friendly Explanation:** Combining masks lets you express complex conditions like "values between 5 and 10" or "either this or that." The key rule is: always wrap each comparison in parentheses before combining with `&`, `|`, `~`, or `^`. This prevents Python from evaluating the bitwise operator before the comparison.

### Purposes

- To build compound conditions for filtering and selection.
- To express complex logical criteria in a readable way.
- To implement multi-condition data validation.
- To combine range checks, exclusions, and alternatives in a single expression.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Correct: parentheses around each comparison
mask = (condition1) & (condition2)
mask = (condition1) | (condition2)
mask = ~(condition1)

# Incorrect: no parentheses (precedence error)
mask = condition1 & condition2   # May parse incorrectly
```

**Component Breakdown:**

| Operator | Precedence | Use in Mask Combination |
|----------|------------|------------------------|
| `~` | Highest | Invert a mask. |
| `&` | High | Combine masks with AND. |
| `^` | Medium | Combine masks with XOR. |
| `\|` | Medium | Combine masks with OR. |
| Comparisons (`<`, `>`, `==`) | Lower | Must be parenthesized. |

**Syntax Rules:**

- Always wrap comparisons in parentheses before combining with `&`, `|`, `~`, `^`.
- Use `&` for AND, `|` for OR, `~` for NOT, `^` for XOR.
- Never use Python's `and`, `or`, `not` on arrays; they raise `ValueError`.

**Constraints and Limitations:**

- Forgetting parentheses is the most common mistake in mask combination.
- Operator precedence can produce silently incorrect results if not careful.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Correct vs. Incorrect Mask Combination**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create an array.
x = np.array([1, 5, 3, 8, 2, 7, 4, 9, 6])
print("Array:", x)
print()

# Step 3: Correct combination with parentheses.
correct = (x > 3) & (x < 8)
print("Correct: (x > 3) & (x < 8):", correct)
print("  Selected:", x[correct])
print()

# Step 4: Incorrect combination (missing parentheses).
try:
    incorrect = x > 3 & x < 8
    print("Incorrect: x > 3 & x < 8:", incorrect)
except ValueError as e:
    print("Incorrect raises ValueError:", e)
print()

# Step 5: Correct combination with OR and NOT.
condition = ((x < 3) | (x > 7)) & ~(x == 5)
print("((x < 3) | (x > 7)) & ~(x == 5):", condition)
print("  Selected:", x[condition])
```

**Expected Output:**

```
Array: [1 5 3 8 2 7 4 9 6]

Correct: (x > 3) & (x < 8): [False  True False False False  True  True False  True]
  Selected: [5 7 4 6]

Incorrect raises ValueError: The truth value of an array with more than one element is ambiguous. Use a.any() or a.all()

((x < 3) | (x > 7)) & ~(x == 5): [ True False False  True  True False False  True False]
  Selected: [1 8 2 9]
```

**Why This Result Occurs:** The correct expression uses parentheses to group each comparison, so `&` combines the resulting boolean masks. Without parentheses, `&` is evaluated before the comparisons, producing an ambiguous result that Python cannot resolve to a single truth value. The compound condition with `~` excludes the value 5 from the union of the two ranges.

### Real-World Cases

- **Data Filtering:** `(df['age'] > 18) & (df['income'] > 50000)` for selecting eligible customers.
- **Image Processing:** `(r > 200) & (g > 150) & (b < 100)` for detecting yellow pixels.
- **Quality Control:** `(size > 10) & ~(weight > 100)` for filtering products.

### References

- Numpy-discussion: Addressing arrays – https://mail.python.org/pipermail/numpy-discussion/2012-January/060032.html
- Numpy-discussion: where construct – https://mail.python.org/pipermail/numpy-discussion/2011-March/054999.html

---

## Comparison of Boolean Operations

| Operation | Ufunc | Operator | Description | Truth Table (x1, x2) |
|-----------|-------|----------|-------------|---------------------|
| AND | `np.logical_and` | `&` | True if both are True | T,T→T; all else→F |
| OR | `np.logical_or` | `\|` | True if either is True | F,F→F; all else→T |
| NOT | `np.logical_not` | `~` | Inverts the value | T→F; F→T |
| XOR | `np.logical_xor` | `^` | True if exactly one is True | T,F→T; F,T→T; else→F |

---

## Summary of Deprecated and Version-Specific Features

| Feature | Status | Recommendation |
|---------|--------|----------------|
| Using `and`/`or`/`not` on arrays | Raises `ValueError` | Use `&`, `\|`, `~` instead |
| `np.bitwise_and` on boolean arrays | Equivalent to `np.logical_and` | Use either; `&` is preferred for brevity |
| `np.bitwise_not` | Alias for `np.invert` | Use `~` or `np.logical_not` for booleans |
| Scalar `&` with Python ints | Performs bitwise AND | Use `np.logical_and` for logical behavior |

---

## Consolidated Reference List

- Logic functions – https://numpy.org/devdocs/reference/routines.logic.html
- numpy.logical_and – https://numpy.org/doc/1.26/reference/generated/numpy.logical_and.html
- numpy.logical_or – https://numpy.org/doc/1.20/reference/generated/numpy.logical_or.html
- numpy.logical_not – https://numpy.org/doc/1.20/reference/generated/numpy.logical_not.html
- numpy.logical_xor – https://numpy.org/doc/1.20/reference/generated/numpy.logical_xor.html
- Numpy-discussion: Addressing arrays – https://mail.python.org/pipermail/numpy-discussion/2012-January/060032.html
- Numpy-discussion: where construct – https://mail.python.org/pipermail/numpy-discussion/2011-March/054999.html
- Numpy-discussion: boolean operations on boolean arrays – https://mail.python.org/pipermail/numpy-discussion/2014-April/070233.html