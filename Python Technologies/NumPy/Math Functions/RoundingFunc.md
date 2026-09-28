# NumPy Rounding Functions: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** NumPy rounding functions are universal functions (ufuncs) that transform floating-point array elements into rounded values according to specific rounding rules, such as rounding to the nearest integer, rounding down, rounding up, or truncating toward zero.

**Technical Definition:** These functions are implemented as `numpy.ufunc` objects — compiled C functions that operate element-by-element on ndarrays, supporting broadcasting, type casting, and the `out`/`where` parameters. They include `np.around` (alias `np.round`), `np.floor`, `np.ceil`, `np.trunc`, `np.fix`, and `np.rint`. Each implements a distinct rounding mode with specific tie-breaking and sign-handling behavior.

**Beginner-Friendly Explanation:** NumPy rounding functions let you simplify decimal numbers by cutting off or adjusting their fractional parts. Some round to the nearest whole number, some always round down, some always round up, and some just chop off the decimals. Each function follows a different rule, so choosing the right one depends on what you want to happen at the boundaries.

### Key Characteristics

- **Element-Wise:** Each function operates independently on every element of an array.
- **Vectorized:** Implemented in compiled C for high performance.
- **Banker's Rounding:** `np.around` and `np.rint` use round-half-to-even, minimizing statistical bias.
- **Directional Rounding:** `np.floor` rounds toward negative infinity; `np.ceil` rounds toward positive infinity.
- **Truncation:** `np.trunc` and `np.fix` round toward zero, discarding the fractional part.
- **Return Type:** Most functions return floats even when the result is a whole number.

### Prerequisites

- NumPy installed (`pip install numpy`).
- Basic Python syntax and understanding of variables.
- Familiarity with NumPy array creation and the `import numpy as np` convention.
- Conceptual understanding of floating-point representation.

### Related Programming Areas

- **Data Analysis:** Rounding measurements and statistics for reporting.
- **Machine Learning:** Quantizing model weights and activations.
- **Financial Computing:** Rounding currency amounts to the nearest cent.
- **Scientific Computing:** Reducing precision of simulation outputs.

### Core Concepts / Features

The following rounding functions are explored in detail: (1) `np.around()` / `np.round()`, (2) `np.floor()`, (3) `np.ceil()`, (4) `np.trunc()`, (5) `np.fix()`, and (6) `np.rint()`.

---

## Core Concept 1: np.around() / np.round()

### Definitions

**Core Definition:** `np.around()` (alias `np.round()`) rounds array elements to a specified number of decimal places using round-half-to-even (banker's rounding) behavior.

**Technical Definition:** `numpy.around(a, decimals=0, out=None)` rounds an array to the given number of decimals. `around` is an alias of `round`. For values exactly halfway between rounded decimal values, NumPy rounds to the nearest even value. Thus 1.5 and 2.5 round to 2.0, and -0.5 and 0.5 round to 0.0 .

**Beginner-Friendly Explanation:** `np.round()` rounds numbers to the nearest value at a specified decimal place. When a number is exactly halfway between two possible results (like 2.5), it rounds to the nearest even number — this is called "banker's rounding" and helps avoid bias in statistical calculations.

### Purposes

- To round numbers to a specified number of decimal places for reporting or display.
- To reduce numerical precision in a controlled way.
- To round to negative decimals (e.g., rounding to the nearest ten or hundred).
- To apply consistent rounding rules across entire arrays.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.around(a, decimals=0, out=None)
np.round(a, decimals=0, out=None)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `a` | Input array (array_like). |
| `decimals` | Number of decimal places to round to (default 0). If negative, specifies positions to the left of the decimal point. |
| `out` | Optional output array. |

**Syntax Rules:**

- `np.around` and `np.round` are aliases; they behave identically.
- Negative `decimals` rounds to the left of the decimal point (e.g., `decimals=-1` rounds to the nearest 10).
- The real and imaginary parts of complex numbers are rounded separately.
- For integer arrays, `np.round` returns the array unchanged (no rounding needed).

**Constraints and Limitations:**

- Uses a fast but sometimes inexact algorithm for floating-point values; `np.round(56294995342131.5, 3)` may produce `56294995342131.51` due to floating-point representation .
- Python's built-in `round()` uses a more accurate but slower algorithm for 64-bit floats.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Basic Rounding with np.round()**

```python
import numpy as np

# Step 2: Create an array of values.
arr = np.array([1.234, 2.567, 3.891, 4.123])
print("Original:", arr)
print()

# Step 3: Round to 0 decimals (nearest integer).
print("np.round(arr):", np.round(arr))
print()

# Step 4: Round to 1 decimal place.
print("np.round(arr, 1):", np.round(arr, 1))
print()

# Step 5: Round to 2 decimal places.
print("np.round(arr, 2):", np.round(arr, 2))
print()

# Step 6: Demonstrate banker's rounding at .5 boundaries.
halves = np.array([0.5, 1.5, 2.5, 3.5, 4.5])
print("Halves:", halves)
print("np.round(halves):", np.round(halves))
print("  (rounds to nearest even: 0, 2, 2, 4, 4)")
print()

# Step 7: Negative decimals (round to nearest 10).
tens = np.array([11, 15, 24, 25, 26])
print("np.round(tens, -1):", np.round(tens, -1))
```

**Expected Output:**

```
Original: [1.234 2.567 3.891 4.123]

np.round(arr): [1. 3. 4. 4.]

np.round(arr, 1): [1.2 2.6 3.9 4.1]

np.round(arr, 2): [1.23 2.57 3.89 4.12]

Halves: [0.5 1.5 2.5 3.5 4.5]
np.round(halves): [0. 2. 2. 4. 4.]
  (rounds to nearest even: 0, 2, 2, 4, 4)

np.round(tens, -1): [10 20 20 20 30]
```

**Why This Result Occurs:** `np.round(arr)` rounds each element to the nearest integer. `np.round(arr, 1)` keeps one decimal place. The `halves` example demonstrates banker's rounding: 0.5 → 0 (even), 1.5 → 2 (even), 2.5 → 2 (even), 3.5 → 4 (even), 4.5 → 4 (even). Negative decimals round to the left of the decimal point.

### Real-World Cases

- **Financial Reports:** Rounding currency amounts to two decimal places.
- **Data Display:** Limiting decimal places for readable output.
- **Statistics:** Reducing precision of computed statistics for publication.

### References

- numpy.around – https://numpy.org/doc/stable/reference/generated/numpy.around.html
- numpy.round – https://numpy.org/doc/stable/reference/generated/numpy.round.html

---

## Core Concept 2: np.floor()

### Definitions

**Core Definition:** `np.floor()` returns the largest integer less than or equal to each element, rounding toward negative infinity.

**Technical Definition:** `numpy.floor(x)` returns the floor of the input, element-wise. The floor of a scalar \(x\) is the largest integer \(i\) such that \(i \le x\). NumPy uses the mathematical definition of floor, where `floor(-2.5) == -3` (unlike some spreadsheet programs that use "floor-towards-zero") .

**Beginner-Friendly Explanation:** `np.floor()` always rounds down to the next lower integer. For positive numbers, this is like chopping off the decimal. For negative numbers, it goes further down: `floor(-2.5) = -3`.

### Purposes

- To round down to the nearest integer.
- To implement binning and discretization with lower boundaries.
- To compute the integer part of a division (combined with `np.floor_divide`).
- To model processes that cannot exceed a lower integer bound.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.floor(x, out=None, where=True)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `x` | Input array or scalar. |
| `out` | Optional output array. |
| `where` | Boolean condition for selective computation. |

**Syntax Rules:**

- The result is always a float, even for integer inputs.
- `np.floor` rounds toward negative infinity, regardless of sign.
- For integer inputs, the result is the same integer (as a float).

**Constraints and Limitations:**

- Returns `nan` for `nan` inputs.
- Returns `-inf` for `-inf` inputs.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Floor Rounding with Positive and Negative Values**

```python
import numpy as np

# Step 2: Create an array.
a = np.array([-1.7, -1.5, -0.2, 0.2, 1.5, 1.7, 2.0])
print("Original:", a)
print()

# Step 3: Apply floor.
print("np.floor(a):", np.floor(a))
print("  (always rounds toward -infinity)")
print()

# Step 4: Contrast with truncation.
print("np.trunc(a):", np.trunc(a))
print("  (truncates toward zero)")
print()

# Step 5: Demonstrate the difference at negative values.
print("floor(-2.5):", np.floor(-2.5))
print("trunc(-2.5):", np.trunc(-2.5))
print("  (floor gives -3, trunc gives -2)")
```

**Expected Output:**

```
Original: [-1.7 -1.5 -0.2  0.2  1.5  1.7  2. ]

np.floor(a): [-2. -2. -1.  0.  1.  1.  2.]
  (always rounds toward -infinity)

np.trunc(a): [-1. -1. -0.  0.  1.  1.  2.]
  (truncates toward zero)

floor(-2.5): -3.0
trunc(-2.5): -2.0
  (floor gives -3, trunc gives -2)
```

**Why This Result Occurs:** `np.floor(-1.7) = -2` because -2 is the largest integer ≤ -1.7. `np.floor(-0.2) = -1`. The difference between `floor` and `trunc` is most visible for negative numbers: `floor(-2.5) = -3` (toward -infinity) while `trunc(-2.5) = -2` (toward zero).

### Real-World Cases

- **Pagination:** Computing the number of full pages needed: `np.floor(total_items / items_per_page)`.
- **Binning:** Assigning data points to discrete bins with lower-bound labels.
- **Time Calculations:** Computing elapsed whole hours from fractional hours.

### References

- numpy.floor – https://numpy.org/doc/1.20/reference/generated/numpy.floor.html

---

## Core Concept 3: np.ceil()

### Definitions

**Core Definition:** `np.ceil()` returns the smallest integer greater than or equal to each element, rounding toward positive infinity.

**Technical Definition:** `numpy.ceil(x)` returns the ceiling of the input, element-wise. The ceil of a scalar \(x\) is the smallest integer \(i\) such that \(i \ge x\) .

**Beginner-Friendly Explanation:** `np.ceil()` always rounds up to the next higher integer. For positive numbers, it rounds up even if the decimal is small: `ceil(2.1) = 3`. For negative numbers, it rounds toward zero: `ceil(-2.5) = -2`.

### Purposes

- To round up to the nearest integer.
- To compute the minimum number of containers or batches needed.
- To implement upper-bound binning.
- To ensure values meet a minimum integer threshold.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.ceil(x, out=None, where=True)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `x` | Input array or scalar. |
| `out` | Optional output array. |
| `where` | Boolean condition for selective computation. |

**Syntax Rules:**

- The result is always a float.
- `np.ceil` rounds toward positive infinity.
- For integer inputs, the result is the same integer (as a float).

**Constraints and Limitations:**

- Returns `nan` for `nan` inputs.
- Returns `inf` for `inf` inputs.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Ceiling Rounding with Positive and Negative Values**

```python
import numpy as np

# Step 2: Create an array.
a = np.array([-1.7, -1.5, -0.2, 0.2, 1.5, 1.7, 2.0])
print("Original:", a)
print()

# Step 3: Apply ceiling.
print("np.ceil(a):", np.ceil(a))
print("  (always rounds toward +infinity)")
print()

# Step 4: Demonstrate the difference with negative values.
print("ceil(-2.5):", np.ceil(-2.5))
print("floor(-2.5):", np.floor(-2.5))
print("  (ceil goes toward zero, floor goes away from zero)")
print()

# Step 5: Practical example: computing minimum boxes needed.
items = np.array([10, 25, 30, 47, 50])
per_box = 12
boxes_needed = np.ceil(items / per_box).astype(int)
print(f"Items: {items}, per box: {per_box}")
print(f"Boxes needed: {boxes_needed}")
```

**Expected Output:**

```
Original: [-1.7 -1.5 -0.2  0.2  1.5  1.7  2. ]

np.ceil(a): [-1. -1. -0.  1.  2.  2.  2.]
  (always rounds toward +infinity)

ceil(-2.5): -2.0
floor(-2.5): -3.0
  (ceil goes toward zero, floor goes away from zero)

Items: [10 25 30 47 50], per box: 12
Boxes needed: [1 3 3 4 5]
```

**Why This Result Occurs:** `np.ceil(-1.7) = -1` because -1 is the smallest integer ≥ -1.7. `np.ceil(0.2) = 1` because 1 is the smallest integer ≥ 0.2. For the practical example, `ceil(25/12) = ceil(2.083) = 3` boxes are needed to hold 25 items when each box holds 12.

### Real-World Cases

- **Inventory Management:** Computing the number of shipping containers needed.
- **Scheduling:** Determining the number of time slots required for a set of tasks.
- **Resource Allocation:** Computing the number of servers needed to handle a load.

### References

- numpy.ceil – https://numpy.org/doc/1.20/reference/generated/numpy.ceil.html

---

## Core Concept 4: np.trunc()

### Definitions

**Core Definition:** `np.trunc()` returns the integer part of each element by rounding toward zero, discarding the fractional component.

**Technical Definition:** `numpy.trunc(x)` returns the truncated value of the input, element-wise. The truncated value is the nearest integer \(i\) such that \(i\) is closer to zero than \(x\) or equal to \(x\). It is a ufunc, faster than `np.fix`, and follows the Array API standard .

**Beginner-Friendly Explanation:** `np.trunc()` simply chops off the decimal part. It always rounds toward zero: `trunc(2.9) = 2`, `trunc(-2.9) = -2`.

### Purposes

- To extract the integer part of a number without rounding.
- To implement truncation-based quantization.
- To compute integer parts in a way consistent with C99 `trunc()`.
- To serve as the modern replacement for `np.fix`.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.trunc(x, out=None, where=True)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `x` | Input array or scalar. |
| `out` | Optional output array. |
| `where` | Boolean condition for selective computation. |

**Syntax Rules:**

- `np.trunc` rounds toward zero (truncation).
- It is a ufunc, so it supports all ufunc features (`out`, `where`, etc.).
- The result is a float (or same type as input for integer inputs).

**Constraints and Limitations:**

- For integer inputs, `np.trunc` returns the input unchanged (already truncated).
- Returns `nan` for `nan` inputs.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Truncation vs. Floor and Ceiling**

```python
import numpy as np

# Step 2: Create an array with mixed positive and negative values.
a = np.array([-3.7, -2.5, -0.5, 0.5, 2.5, 3.7])
print("Original:", a)
print()

# Step 3: Apply trunc.
print("np.trunc(a):", np.trunc(a))
print("  (rounds toward zero)")
print()

# Step 4: Compare all rounding modes.
print("Comparison table:")
print(f"{'Value':>8} | {'trunc':>8} | {'floor':>8} | {'ceil':>8} | {'round':>8}")
print("-" * 48)
for v in a:
    print(f"{v:>8} | {np.trunc(v):>8} | {np.floor(v):>8} | {np.ceil(v):>8} | {np.round(v):>8}")
```

**Expected Output:**

```
Original: [-3.7 -2.5 -0.5  0.5  2.5  3.7]

np.trunc(a): [-3. -2. -0.  0.  2.  3.]
  (rounds toward zero)

Comparison table:
   Value |    trunc |    floor |     ceil |    round
------------------------------------------------
    -3.7 |       -3 |       -4 |       -3 |       -4
    -2.5 |       -2 |       -3 |       -2 |       -2
    -0.5 |       -0 |       -1 |       -0 |       -0
     0.5 |        0 |        0 |        1 |        0
     2.5 |        2 |        2 |        3 |        2
     3.7 |        3 |        3 |        4 |        4
```

**Why This Result Occurs:** `np.trunc(-3.7) = -3` (toward zero), while `np.floor(-3.7) = -4` (toward -infinity). `np.ceil(-3.7) = -3` (toward +infinity). `np.round(-3.7) = -4` (nearest). The table shows how each function behaves differently depending on the sign and fractional part.

### Real-World Cases

- **Data Cleaning:** Removing decimal parts from measurements where fractional values are meaningless.
- **Index Computation:** Converting floating-point indices to integers toward zero.
- **Signal Processing:** Truncating sample values to a fixed number of bits.

### References

- numpy.trunc – https://numpy.org/doc/stable/reference/generated/numpy.trunc.html
- NumPy-Discussion: Deprecating numpy.fix – https://mail.python.org/archives/list/numpy-discussion@python.org/thread/LYRHETXPN32S7FNHKNLJFDIYWGQHLODJ/

---

## Core Concept 5: np.fix()

### Definitions

**Core Definition:** `np.fix()` rounds array elements toward zero, acting as a shortcut alias for truncation logic. It is deprecated in favor of `np.trunc`.

**Technical Definition:** `numpy.fix(x, out=None)` rounds an array of floats element-wise to the nearest integer toward zero. It was added for MATLAB compatibility. As of NumPy 2.5, `np.fix` is deprecated; `np.trunc` should be used instead because it is faster and follows the Array API standard .

**Beginner-Friendly Explanation:** `np.fix()` does the same thing as `np.trunc()` — it chops off the decimal part, rounding toward zero. However, it is now deprecated, meaning you should use `np.trunc()` instead in new code.

### Purposes

- To provide MATLAB-compatible truncation behavior (historical).
- To round toward zero in legacy code.
- **Note:** New code should use `np.trunc` instead.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.fix(x, out=None)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `x` | Input array (array_like). |
| `out` | Optional output array. |

**Syntax Rules:**

- `np.fix` rounds toward zero (truncation).
- It returns an array of floats with the same dimensions and data type as the input.
- Deprecated since NumPy 2.5: use `np.trunc` instead.

**Constraints and Limitations:**

- `np.fix` is slower than `np.trunc` (approximately 50% slower on ARM Mac for million-element arrays) .
- It is not a ufunc, unlike `np.trunc`.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: np.fix vs np.trunc**

```python
import numpy as np

# Step 2: Create an array.
a = np.array([2.1, 2.9, -2.1, -2.9])
print("Original:", a)
print()

# Step 3: Apply np.fix (deprecated).
print("np.fix(a):", np.fix(a))
print("  (rounds toward zero)")
print()

# Step 4: Apply np.trunc (recommended).
print("np.trunc(a):", np.trunc(a))
print("  (same result, faster, not deprecated)")
print()

# Step 5: Verify they produce identical results.
print("Results match:", np.array_equal(np.fix(a), np.trunc(a)))
```

**Expected Output:**

```
Original: [ 2.1  2.9 -2.1 -2.9]

np.fix(a): [ 2.  2. -2. -2.]
  (rounds toward zero)

np.trunc(a): [ 2.  2. -2. -2.]
  (same result, faster, not deprecated)

Results match: True
```

**Why This Result Occurs:** `np.fix` and `np.trunc` both round toward zero, so `2.1 → 2`, `2.9 → 2`, `-2.1 → -2`, `-2.9 → -2`. The results are identical, but `np.trunc` is preferred because it is faster and not deprecated.

### Real-World Cases

- **Legacy Code:** Maintaining older code that uses `np.fix` for MATLAB compatibility.
- **Migration:** Replacing `np.fix` with `np.trunc` in new code to avoid deprecation warnings.

### References

- numpy.fix – https://numpy.org/doc/stable//reference/generated/numpy.fix.html
- Mailman 3 Deprecating numpy.fix – https://mail.python.org/archives/list/numpy-discussion@python.org/thread/LYRHETXPN32S7FNHKNLJFDIYWGQHLODJ/

---

## Core Concept 6: np.rint()

### Definitions

**Core Definition:** `np.rint()` rounds array elements to the nearest integer, returning a float array with the same shape and type as the input.

**Technical Definition:** `numpy.rint(x, /, out=None, *, where=True, casting='same_kind', order='K', dtype=None, subok=True[, signature])` rounds elements of the array to the nearest integer. For values exactly halfway between rounded decimal values, NumPy rounds to the nearest even value .

**Beginner-Friendly Explanation:** `np.rint()` rounds each number to the nearest whole number. If a number is exactly halfway (like 2.5), it rounds to the nearest even number. The result is always a float, not an integer, even though the values are whole numbers.

### Purposes

- To round to the nearest integer while preserving the float dtype.
- To implement consistent round-half-to-even behavior.
- To avoid the dtype conversion that would occur with `np.round().astype(int)`.
- To round arrays where the result must remain float for downstream operations.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.rint(x, out=None, where=True, casting='same_kind', order='K', dtype=None)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `x` | Input array or scalar. |
| `out` | Optional output array. |
| `where` | Boolean condition for selective computation. |
| `dtype` | Optional target dtype. |

**Syntax Rules:**

- The output has the same shape and type as the input (`x`).
- `np.rint` uses round-half-to-even (banker's rounding).
- It is a ufunc, so it supports all ufunc features.

**Constraints and Limitations:**

- Unlike `np.round`, `np.rint` does not accept a `decimals` parameter; it always rounds to the nearest integer.
- The result is a float, not an integer.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: np.rint vs np.round**

```python
import numpy as np

# Step 2: Create an array.
a = np.array([-1.7, -1.5, -0.2, 0.2, 1.5, 1.7, 2.0, 2.5])
print("Original:", a)
print()

# Step 3: Apply np.rint (always to nearest integer).
print("np.rint(a):", np.rint(a))
print("  (rounds to nearest integer, ties to even)")
print()

# Step 4: Compare with np.round (default 0 decimals).
print("np.round(a):", np.round(a))
print("  (same behavior for 0 decimals)")
print()

# Step 5: Demonstrate the difference: rint doesn't accept decimals.
print("np.rint always returns the same dtype:")
print("  Input dtype:", a.dtype)
print("  Output dtype:", np.rint(a).dtype)
print()

# Step 6: Show that rint can be used for scaling and unscaling.
scaled = np.rint(a * 10) / 10
print("np.rint(a * 10) / 10:", scaled)
```

**Expected Output:**

```
Original: [-1.7 -1.5 -0.2  0.2  1.5  1.7  2.   2.5]

np.rint(a): [-2. -2. -0.  0.  2.  2.  2.  2.]
  (rounds to nearest integer, ties to even)

np.round(a): [-2. -2. -0.  0.  2.  2.  2.  2.]
  (same behavior for 0 decimals)

np.rint always returns the same dtype:
  Input dtype: float64
  Output dtype: float64

np.rint(a * 10) / 10: [-1.7 -1.5 -0.2  0.2  1.5  1.7  2.   2.5]
```

**Why This Result Occurs:** `np.rint` rounds to the nearest integer with ties going to even: `-1.5 → -2`, `1.5 → 2`, `2.5 → 2`. The output dtype is `float64`, same as the input. `np.rint(a * 10) / 10` demonstrates a common pattern: scale, round, unscale.

### Real-World Cases

- **Neural Network Quantization:** Rounding weights and activations to the nearest integer while keeping float dtype.
- **Physics Simulations:** Rounding positions to the nearest integer grid point.
- **Data Processing:** Converting floating-point indices to nearest integers.

### References

- numpy.rint – https://numpy.org/doc/stable/reference/generated/numpy.rint.html

---

## Comparison Summary

| Function | Rounding Rule | Result dtype | Notes |
|----------|--------------|-------------|-------|
| `np.around` / `np.round` | Nearest, ties to even | Float (or int for int input) | Accepts `decimals` parameter. |
| `np.floor` | Toward -infinity | Float | Always rounds down. |
| `np.ceil` | Toward +infinity | Float | Always rounds up. |
| `np.trunc` | Toward zero | Float | Chopping off decimals. |
| `np.fix` | Toward zero | Float | **Deprecated**; use `np.trunc`. |
| `np.rint` | Nearest, ties to even | Float (same as input) | No `decimals` parameter. |

---

## Summary of Deprecated and Version-Specific Features

| Feature | Status | Recommendation |
|---------|--------|----------------|
| `np.fix` | Deprecated since NumPy 2.5 | Use `np.trunc` (faster, ufunc, Array API standard) |
| `np.round` vs `np.around` | Both available; `around` is alias | Use `np.round` for brevity |
| `np.rint` with `decimals` | Not supported | Use `np.round` for decimal rounding |
| `np.floor` on integers | Returns float | Cast back to int if needed |

---

## Consolidated Reference List

- numpy.around – https://numpy.org/doc/stable/reference/generated/numpy.around.html
- numpy.round – https://numpy.org/doc/stable/reference/generated/numpy.round.html
- numpy.floor – https://numpy.org/doc/1.20/reference/generated/numpy.floor.html
- numpy.ceil – https://numpy.org/doc/1.20/reference/generated/numpy.ceil.html
- numpy.trunc – https://numpy.org/doc/stable/reference/generated/numpy.trunc.html
- numpy.fix – https://numpy.org/doc/stable//reference/generated/numpy.fix.html
- numpy.rint – https://numpy.org/doc/stable/reference/generated/numpy.rint.html
- Mailman 3 Deprecating numpy.fix – https://mail.python.org/archives/list/numpy-discussion@python.org/thread/LYRHETXPN32S7FNHKNLJFDIYWGQHLODJ/
- Stack Overflow: numpy trunc vs fix – https://stackoverflow.com/questions/21420901
- NumPy Rounding Functions – https://numpy.org/numpy-quaddtype/api/functions.html