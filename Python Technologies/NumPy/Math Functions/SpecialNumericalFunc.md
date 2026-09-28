# NumPy Special Numerical Functions: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** NumPy special numerical functions are a collection of universal functions (ufuncs) that handle specialized numerical operations beyond basic arithmetic—including sign detection, value clamping, remainder computation, floating-point decomposition, and IEEE 754 special value (NaN/Inf) management.

**Technical Definition:** These functions are implemented as `numpy.ufunc` objects—compiled C functions that operate element-wise on ndarrays, supporting broadcasting, type casting, and the `out`/`where` parameters. They include `np.sign`, `np.clip`, `np.mod`, `np.fmod`, `np.modf`, `np.frexp`, `np.ldexp`, `np.isnan`, `np.isinf`, `np.isfinite`, and `np.nan_to_num`.

**Beginner-Friendly Explanation:** Beyond basic math, NumPy provides tools for handling special situations: detecting the sign of numbers, limiting values to a range, working with remainders, breaking floating-point numbers into their components, and safely dealing with "Not a Number" (NaN) and infinity values. These functions are essential for robust numerical computing.

### Key Characteristics

- **Element-Wise:** Each function operates independently on every element of an array.
- **Vectorized:** Implemented in compiled C for high performance.
- **IEEE 754 Awareness:** Functions like `isnan`, `isinf`, and `nan_to_num` handle special floating-point values explicitly.
- **Sign Convention Control:** `np.mod` and `np.fmod` differ in how they handle negative operands.
- **Decomposition Utilities:** `np.modf`, `np.frexp`, and `np.ldexp` expose the internal structure of floating-point numbers.
- **Broadcasting Support:** All functions accept array-like inputs and broadcast automatically.

### Prerequisites

- NumPy installed (`pip install numpy`).
- Basic Python syntax and understanding of variables.
- Familiarity with NumPy array creation and the `import numpy as np` convention.
- Conceptual understanding of floating-point representation (IEEE 754).

### Related Programming Areas

- **Data Cleaning:** Detecting and replacing NaN/Inf values in datasets.
- **Signal Processing:** Sign detection and clipping for waveform manipulation.
- **Numerical Analysis:** Floating-point decomposition for precision studies.
- **Machine Learning:** Gradient clipping and sign-based optimization.
- **Scientific Computing:** Handling special values in simulation outputs.

### Core Concepts / Features

The following functions are explored in detail: (1) `np.sign()`, (2) `np.clip()`, (3) `np.mod()` and `np.fmod()`, (4) `np.modf()`, (5) `np.frexp()` and `np.ldexp()`, and (6) Infinity and NaN management (`np.isnan`, `np.isinf`, `np.isfinite`, `np.nan_to_num`).

---

## Core Concept 1: np.sign()

### Definitions

**Core Definition:** `np.sign()` returns an element-wise indication of the sign of a number: −1 for negative, 0 for zero, and 1 for positive.

**Technical Definition:** `numpy.sign(x)` returns −1 if x < 0, 0 if x == 0, and 1 if x > 0. NaN is returned for NaN inputs. For complex inputs, the sign function returns `x / abs(x)`, the generalization of the above (and 0 if x == 0).

**Beginner-Friendly Explanation:** `np.sign()` tells you whether a number is negative, zero, or positive. It returns −1, 0, or 1 respectively. For complex numbers, it returns a complex number with the same direction but unit magnitude.

### Purposes

- To extract the directional component of a value without its magnitude.
- To implement sign-based logic (e.g., gradient sign updates).
- To classify values into negative, zero, and positive categories.
- To compute unit vectors from arbitrary vectors.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.sign(x, out=None, where=True, casting='same_kind', order='K', dtype=None)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `x` | Input array or scalar. |
| `out` | Optional output array. |
| `where` | Boolean condition for selective computation. |
| Returns | Sign indicator: −1, 0, or 1 (or complex equivalent). |

**Syntax Rules:**

- `np.sign` returns `nan` for `nan` inputs.
- For complex inputs, the definition changed in NumPy 2.0.0 to follow the Array API standard (`x / |x|`). Prior to 2.0, it used `x / sqrt(x*x)`.
- The result is a float array even for integer inputs.

**Constraints and Limitations:**

- `np.sign(0)` returns 0, which may be ambiguous in some contexts.
- For complex numbers, the sign is a direction on the complex plane, not a real number.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Sign Detection for Real and Complex Values**

```python
import numpy as np

# Step 2: Create an array with mixed signs.
a = np.array([-5.0, -0.0, 0.0, 4.5, 3.0])
print("Original        :", a)

# Step 3: Apply np.sign.
signs = np.sign(a)
print("np.sign(a)      :", signs)

# Step 4: Complex sign detection.
z = np.array([3 - 4j, 8j, 0 + 0j])
print("\nComplex array :", z)
print("np.sign(z)      :", np.sign(z))
print("  (unit-magnitude direction)")
```

**Expected Output:**

```
Original      : [-5. -0.  0.  4.5  3. ]
np.sign(a)    : [-1. -0.  0.  1.  1.]

Complex array : [3.-4.j 0.+8.j 0.+0.j]
np.sign(z)    : [0.6-0.8j 0. +1.j 0. +0.j]
  (unit-magnitude direction)
```

**Why This Result Occurs:** `np.sign` maps negative values to −1 and positive values to 1. For the complex number `3−4j`, the magnitude is 5, so the sign is `(3−4j)/5 = 0.6−0.8j`, a unit-magnitude complex number.

### Real-World Cases

- **Optimization:** Sign-based gradient methods (e.g., signSGD) use `np.sign(gradients)` for update directions.
- **Unit Vectors:** Computing unit vectors from arbitrary vectors in physics and graphics.
- **Signal Processing:** Zero-crossing detection for frequency estimation.

### References

- numpy.sign – https://numpy.org/doc/stable/reference/generated/numpy.sign.html

---

## Core Concept 2: np.clip()

### Definitions

**Core Definition:** `np.clip()` limits the values in an array to a specified interval, replacing values outside the interval with the interval boundaries.

**Technical Definition:** `numpy.clip(a, a_min, a_max, out=None)` clips (limits) the values in an array. Given an interval, values outside the interval are clipped to the interval edges. For example, if an interval of [0, 1] is specified, values smaller than 0 become 0, and values larger than 1 become 1. It is equivalent to but faster than `np.minimum(a_max, np.maximum(a, a_min))`.

**Beginner-Friendly Explanation:** `np.clip()` is like putting a fence around your data. Any values below the minimum fence get raised to the fence, and any values above the maximum fence get lowered to the fence. Values already inside the fence stay unchanged.

### Purposes

- To enforce bounds on data (e.g., clipping probabilities to [0, 1]).
- To prevent overflow or underflow in downstream computations.
- To implement gradient clipping in neural network training.
- To handle outliers by capping them at reasonable limits.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.clip(a, a_min, a_max, out=None)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `a` | Input array. |
| `a_min` | Minimum value; values below this become `a_min`. Can be `None` to disable lower clipping. |
| `a_max` | Maximum value; values above this become `a_max`. Can be `None` to disable upper clipping. |
| `out` | Optional output array; can be the input array for in-place clipping. |

**Syntax Rules:**

- At most one of `a_min` and `a_max` may be `None`.
- `a_min` and `a_max` are broadcast against `a`.
- No check is performed to ensure `a_min < a_max`.
- The result has the same dtype as the input.

**Constraints and Limitations:**

- If `a_min > a_max`, the result is undefined (all values become `a_max` in practice).
- In-place clipping (`out=a`) modifies the original array.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Clipping to a Range**

```python
import numpy as np

# Step 2: Create an array.
a = np.arange(10)
print("Original                   :", a)

# Step 3: Clip to [1, 8].
clipped = np.clip(a, 1, 8)
print("np.clip(a, 1, 8)           :", clipped)

# Step 4: In-place clipping.
b = np.arange(10)
np.clip(b, 3, 6, out=b)
print("In-place clip to [3, 6]    :", b)

# Step 5: Array-like bounds (broadcasting).
c = np.arange(10)
bounds_min = np.array([3, 4, 1, 1, 1, 4, 4, 4, 4, 4])
clipped_bounds = np.clip(c, bounds_min, 8)
print("Clip with array min bounds :", clipped_bounds)
```

**Expected Output:**

```
Original                   : [0 1 2 3 4 5 6 7 8 9]
np.clip(a, 1, 8)           : [1 1 2 3 4 5 6 7 8 8]
In-place clip to [3, 6]    : [3 3 3 3 4 5 6 6 6 6]
Clip with array min bounds : [3 4 2 3 4 5 6 7 8 8]
```

**Why This Result Occurs:** Values below 1 become 1, values above 8 become 8. In-place clipping modifies the original array. Array-like bounds allow per-element minimum values, producing different clipping for each position.

### Real-World Cases

- **Neural Networks:** Gradient clipping prevents exploding gradients.
- **Image Processing:** Clipping pixel values to [0, 255] after filtering.
- **Probability Calibration:** Clipping predicted probabilities to [ε, 1−ε] for numerical stability.

### References

- numpy.clip – https://numpy.org/doc/1.19/reference/generated/numpy.clip.html

---

## Core Concept 3: np.mod() and np.fmod()

### Definitions

**Core Definition:** `np.mod()` computes the remainder with the same sign as the divisor (matching Python's `%`), while `np.fmod()` computes the remainder with the same sign as the dividend (matching C's `fmod()`).

**Technical Definition:** `numpy.mod(x1, x2)` returns the element-wise remainder of division. It is equivalent to the Python modulus operator `x1 % x2` and has the same sign as the divisor `x2`. `numpy.fmod(x1, x2)` is the NumPy implementation of the C library function `fmod`, and the remainder has the same sign as the dividend `x1`.

**Beginner-Friendly Explanation:** When you divide two numbers and take the remainder, the sign of the result depends on which convention you use. `np.mod` follows Python's convention (result has the sign of the divisor), while `np.fmod` follows C's convention (result has the sign of the dividend).

### Purposes

- To compute remainders for cyclic operations (e.g., clock arithmetic).
- To implement modular arithmetic for hashing and indexing.
- To extract fractional parts of division with specific sign conventions.
- To wrap values into a periodic range.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.mod(x1, x2, out=None, where=True)
np.fmod(x1, x2, out=None, where=True)
```

**Component Breakdown:**

| Function | Sign Convention | Python Equivalent |
|----------|----------------|-------------------|
| `np.mod` | Same sign as divisor (x2) | `x1 % x2` |
| `np.fmod` | Same sign as dividend (x1) | C `fmod` |

**Syntax Rules:**

- `np.mod` returns 0 when `x2` is 0 and both are integers.
- Both functions accept broadcasting for `x1` and `x2`.
- `np.mod` is an alias of `np.remainder`.

**Constraints and Limitations:**

- The results differ for negative operands; choose based on your mathematical convention.
- `np.fmod` is equivalent to MATLAB's `rem` function.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Comparing mod and fmod**

```python
import numpy as np

# Step 2: Create arrays with negative values.
x1 = np.array([-3, -2, -1, 1, 2, 3])
x2 = 2

# Step 3: Apply np.mod.
mod_result = np.mod(x1, x2)
print("np.mod([-3,-2,-1,1,2,3], 2):", mod_result)

# Step 4: Apply np.fmod.
fmod_result = np.fmod(x1, x2)
print("np.fmod([-3,-2,-1,1,2,3], 2):", fmod_result)

# Step 5: Show the difference clearly.
print("\nDifference:")
print(f"{'x':>4} | {'mod':>5} | {'fmod':>5}")
print("-" * 18)
for x in x1:
    print(f"{x:>4} | {np.mod(x, x2):>5} | {np.fmod(x, x2):>5}")
```

**Expected Output:**

```
np.mod([-3,-2,-1,1,2,3], 2): [1 0 1 1 0 1]
np.fmod([-3,-2,-1,1,2,3], 2): [-1  0 -1  1  0  1]

Difference:
   x |   mod |  fmod
------------------
  -3 |     1 |    -1
  -2 |     0 |     0
  -1 |     1 |    -1
   1 |     1 |     1
   2 |     0 |     0
   3 |     1 |     1
```

**Why This Result Occurs:** For `−3`, `np.mod(−3, 2) = 1` because the result has the sign of the divisor (2), while `np.fmod(−3, 2) = −1` because the result has the sign of the dividend (−3).

### Real-World Cases

- **Clock Arithmetic:** `np.mod(hour + offset, 12)` for 12-hour clock.
- **Hash Tables:** `np.mod(hash_value, table_size)` for indexing.
- **Signal Wrapping:** Wrapping phase angles into [−π, π].

### References

- numpy.mod – https://numpy.org/doc/1.21/reference/generated/numpy.mod.html
- numpy.fmod – https://numpy.org/doc/1.16/reference/generated/numpy.fmod.html

---

## Core Concept 4: np.modf()

### Definitions

**Core Definition:** `np.modf()` returns the fractional and integral parts of an array, element-wise, with both parts having the same sign as the input.

**Technical Definition:** `numpy.modf(x)` returns a tuple `(fractional_part, integral_part)` where each element of the input is split into its fractional and integer components. The fractional and integral parts are negative if the given number is negative. For integer inputs, the return values are floats.

**Beginner-Friendly Explanation:** `np.modf()` takes a decimal number and splits it into two pieces: the part before the decimal point and the part after. For example, `3.75` becomes `0.75` (fractional) and `3.0` (integral).

### Purposes

- To separate the integer and fractional components of measurements.
- To extract the fractional part for phase computations.
- To decompose values for custom rounding or formatting.
- To implement fixed-point arithmetic.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.modf(x, out1=None, out2=None, out=(None, None), where=True)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `x` | Input array. |
| `out1` | Output for fractional part (optional). |
| `out2` | Output for integral part (optional). |
| Returns | Tuple: `(fractional, integral)` |

**Syntax Rules:**

- Both parts have the same sign as the input.
- `divmod(x, 1)` is equivalent to `modf` with the return values switched, except it always has a positive remainder.
- For integer inputs, the return values are floats.

**Constraints and Limitations:**

- The fractional part is always in (−1, 1).
- The sum of the two parts equals the original value.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Decomposing Values with modf**

```python
import numpy as np

# Step 2: Create an array.
a = np.array([0.0, 3.5, -2.25, 7.0])
print("Original      :", a)

# Step 3: Decompose with np.modf.
frac, integ = np.modf(a)
print("Fractional    :", frac)
print("Integral      :", integ)

# Step 4: Verify reconstruction.
reconstructed = frac + integ
print("Reconstructed :", reconstructed)
print("Match         :", np.allclose(a, reconstructed))
```

**Expected Output:**

```
Original      : [ 0.    3.5  -2.25  7.  ]
Fractional    : [ 0.    0.5  -0.25  0.  ]
Integral      : [ 0.    3.   -2.    7.  ]
Reconstructed : [ 0.    3.5  -2.25  7.  ]
Match         : True
```

**Why This Result Occurs:** `np.modf(3.5)` returns `(0.5, 3.0)`. For `−2.25`, both parts are negative: `(−0.25, −2.0)`. The reconstruction `frac + integ` always equals the original value.

### Real-World Cases

- **Time Decomposition:** Separating hours and minutes from a fractional hour value.
- **Currency Handling:** Splitting dollars and cents for integer arithmetic.
- **Phase Extraction:** Extracting the fractional part of a phase measurement.

### References

- numpy.modf – https://numpy.org/doc/2.0/reference/generated/numpy.modf.html

---

## Core Concept 5: np.frexp() and np.ldexp()

### Definitions

**Core Definition:** `np.frexp()` decomposes a floating-point number into its normalized mantissa and base-2 exponent, and `np.ldexp()` reconstructs the original number from these components.

**Technical Definition:** `numpy.frexp(x)` returns `(mantissa, exponent)` such that `x = mantissa * 2**exponent`, where `mantissa` is in the range [0.5, 1). `numpy.ldexp(x1, x2)` returns `x1 * 2**x2`, element-wise, and is the inverse of `frexp`.

**Beginner-Friendly Explanation:** Every floating-point number can be written as a mantissa (a number between 0.5 and 1) times 2 raised to an exponent. `np.frexp` extracts these two pieces, and `np.ldexp` puts them back together.

### Purposes

- To analyze the internal representation of floating-point numbers.
- To implement custom floating-point arithmetic.
- To scale numbers by powers of 2 efficiently.
- To study numerical precision and range.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.frexp(x, out1=None, out2=None)
np.ldexp(x1, x2, out=None, where=True)
```

**Component Breakdown:**

| Function | Input | Output |
|----------|-------|--------|
| `np.frexp` | `x` | `(mantissa, exponent)` |
| `np.ldexp` | `x1` (mantissa), `x2` (exponent) | `x1 * 2**x2` |

**Syntax Rules:**

- `np.frexp` returns a tuple of two arrays.
- `np.ldexp` broadcasts `x1` and `x2`.
- Complex dtypes are not supported by `np.ldexp` (raises `TypeError`).

**Constraints and Limitations:**

- The mantissa is always in [0.5, 1) for nonzero finite inputs.
- `np.frexp(0)` returns `(0.0, 0)`.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Frexp and Ldexp Round-Trip**

```python
import numpy as np

# Step 2: Create an array.
x = np.array([1.0, 2.0, 3.0, 4.0, 0.5])
print("Original:", x)

# Step 3: Decompose with frexp.
mantissa, exponent = np.frexp(x)
print("Mantissa:", mantissa)
print("Exponent:", exponent)

# Step 4: Reconstruct with ldexp.
reconstructed = np.ldexp(mantissa, exponent)
print("Reconstructed:", reconstructed)
print("Match:", np.allclose(x, reconstructed))

# Step 5: Show the decomposition formula.
print("\nDecomposition:")
for i in range(len(x)):
    print(f"  {x[i]} = {mantissa[i]} * 2^{exponent[i]}")
```

**Expected Output:**

```
Original: [1.  2.  3.  4.  0.5]
Mantissa: [0.5  0.5  0.75 0.5  0.5 ]
Exponent: [1 2 2 3 0]
Reconstructed: [1.  2.  3.  4.  0.5]
Match: True

Decomposition:
  1.0 = 0.5 * 2^1
  2.0 = 0.5 * 2^2
  3.0 = 0.75 * 2^2
  4.0 = 0.5 * 2^3
  0.5 = 0.5 * 2^0
```

**Why This Result Occurs:** `3.0 = 0.75 × 2² = 0.75 × 4`. The mantissa is always in [0.5, 1), and the exponent adjusts to reconstruct the original value. `np.ldexp` inverts the process exactly.

### Real-World Cases

- **Numerical Analysis:** Studying floating-point representation and precision.
- **Signal Processing:** Scaling by powers of 2 for efficient hardware implementation.
- **Scientific Computing:** Implementing custom logarithm or exponentiation algorithms.

### References

- numpy.frexp – https://docs.scipy.org/doc/numpy-1.7.0/reference/generated/numpy.frexp.html
- numpy.ldexp – https://numpy.org/doc/1.20/reference/generated/numpy.ldexp.html

---

## Core Concept 6: Infinity and NaN Management

### Definitions

**Core Definition:** NumPy provides specialized functions for detecting and handling IEEE 754 special values: `np.isnan()` (True where NaN), `np.isinf()` (True where infinite), `np.isfinite()` (True where neither NaN nor infinite), and `np.nan_to_num()` (replaces NaN/Inf with finite values).

**Technical Definition:** `numpy.isnan(x)` tests element-wise for NaN. `numpy.isinf(x)` tests element-wise for positive or negative infinity. `numpy.isfinite(x)` tests element-wise for finiteness (not infinity and not NaN). `numpy.nan_to_num(x)` replaces NaN with zero and infinity with large finite numbers (default behavior) or with user-defined values using the `nan`, `posinf`, and `neginf` keywords.

**Beginner-Friendly Explanation:** NaN means "Not a Number" (e.g., 0/0 or sqrt(−1)), and Inf means infinity (e.g., 1/0). NumPy provides tools to detect these special values and replace them with safe defaults so your calculations don't break.

### Purposes

- To detect invalid or missing data in datasets.
- To filter out NaN/Inf values before computation.
- To replace special values with safe defaults.
- To validate numerical results after operations.

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
| `np.isnan` | Boolean array | True where x is NaN |
| `np.isinf` | Boolean array | True where x is ±inf |
| `np.isfinite` | Boolean array | True where x is finite |
| `np.nan_to_num` | Float array | Replaces NaN/Inf with finite values |

**Syntax Rules:**

- `np.isnan` returns `False` for infinite values.
- `np.nan_to_num` default: NaN → 0.0, +inf → max float, −inf → min float.
- `np.nan_to_num` with `copy=False` modifies the input array in place.
- For complex dtypes, `np.nan_to_num` applies replacements to real and imaginary parts separately.

**Constraints and Limitations:**

- `np.isnan` only works on floating-point and complex arrays; it raises an error for integer arrays.
- NaN propagates through most arithmetic operations.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Detecting and Replacing Special Values**

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

# Step 6: Replace with defaults.
safe = np.nan_to_num(a)
print("np.nan_to_num(a):", safe)

# Step 7: Replace with custom values.
custom = np.nan_to_num(a, nan=-1.0, posinf=100.0, neginf=-100.0)
print("Custom replacement:", custom)
```

**Expected Output:**

```
Original: [ 1. nan inf -inf  3.]
np.isnan(a): [False  True False False False]
np.isinf(a): [False False  True  True False]
np.isfinite(a): [ True False False False  True]
np.nan_to_num(a): [ 1.00000000e+00  0.00000000e+00  1.79769313e+308
 -1.79769313e+308  3.00000000e+00]
Custom replacement: [  1.  -1. 100. -100.   3.]
```

**Why This Result Occurs:** `np.isnan` identifies the NaN at index 1. `np.isinf` identifies the infinities at indices 2 and 3. `np.nan_to_num` replaces NaN with 0 and infinities with the largest finite float64 values. Custom replacements override the defaults.

**Example 2: Safe Division with np.errstate**

```python
import numpy as np

# Step 2: Perform division by zero safely.
a = np.array([1.0, 2.0, 3.0])
b = np.array([0.0, 1.0, 0.0])

with np.errstate(divide='ignore', invalid='ignore'):
    result = a / b
    print("Raw result:", result)

# Step 3: Replace non-finite values.
safe_result = np.nan_to_num(result, posinf=0.0, neginf=0.0)
print("Safe result:", safe_result)
```

**Expected Output:**

```
Raw result: [inf  2. inf]
Safe result: [0. 2. 0.]
```

**Why This Result Occurs:** The `np.errstate` context manager suppresses warnings for division by zero. The raw result contains `inf` values, which are then replaced with 0 using `np.nan_to_num` with `posinf=0.0`.

### Real-World Cases

- **Data Cleaning:** Replacing NaN values in datasets before training ML models.
- **Financial Modeling:** Detecting and handling division-by-zero in return calculations.
- **Scientific Computing:** Validating simulation outputs for non-finite values.

### References

- numpy.isnan – https://numpy.org/doc/stable/reference/generated/numpy.isnan.html
- numpy.isinf – https://numpy.org/doc/stable/reference/generated/numpy.isinf.html
- numpy.isfinite – https://numpy.org/doc/stable/reference/generated/numpy.isfinite.html
- numpy.nan_to_num – https://numpy.org/doc/1.20/reference/generated/numpy.nan_to_num.html
- Miscellaneous (IEEE 754 special values) – https://numpy.org/doc/stable/user/misc.html

---

## Comparison Summary

| Function | Purpose | Sign Convention | Returns |
|----------|---------|----------------|---------|
| `np.sign` | Sign indicator | — | −1, 0, 1 |
| `np.clip` | Bound values | — | Clipped array |
| `np.mod` | Remainder | Sign of divisor | Remainder |
| `np.fmod` | Remainder | Sign of dividend | Remainder |
| `np.modf` | Decompose | Both parts same sign | (fractional, integral) |
| `np.frexp` | Decompose | — | (mantissa, exponent) |
| `np.ldexp` | Reconstruct | — | x1 * 2^x2 |
| `np.isnan` | Detect NaN | — | Boolean array |
| `np.isinf` | Detect Inf | — | Boolean array |
| `np.isfinite` | Detect finite | — | Boolean array |
| `np.nan_to_num` | Replace special | — | Finite array |

---

## Summary of Deprecated and Version-Specific Features

| Feature | Status | Recommendation |
|---------|--------|----------------|
| `np.sign` complex definition | Changed in NumPy 2.0.0 | Now uses `x / |x|` (Array API standard) |
| `np.nan_to_num` `copy` parameter | New in 1.13 | Use to control in-place vs. copy |
| `np.nan_to_num` `nan`, `posinf`, `neginf` | New in 1.17 | Use for custom replacements |
| `np.modf` with integer inputs | Returns floats | Cast back to int if needed |

---

## Consolidated Reference List

- numpy.sign – https://numpy.org/doc/stable/reference/generated/numpy.sign.html
- numpy.clip – https://numpy.org/doc/1.19/reference/generated/numpy.clip.html
- numpy.mod – https://numpy.org/doc/1.21/reference/generated/numpy.mod.html
- numpy.fmod – https://numpy.org/doc/1.16/reference/generated/numpy.fmod.html
- numpy.modf – https://numpy.org/doc/2.0/reference/generated/numpy.modf.html
- numpy.frexp – https://docs.scipy.org/doc/numpy-1.7.0/reference/generated/numpy.frexp.html
- numpy.ldexp – https://numpy.org/doc/1.20/reference/generated/numpy.ldexp.html
- numpy.isnan – https://numpy.org/doc/stable/reference/generated/numpy.isnan.html
- numpy.isinf – https://numpy.org/doc/stable/reference/generated/numpy.isinf.html
- numpy.isfinite – https://numpy.org/doc/stable/reference/generated/numpy.isfinite.html
- numpy.nan_to_num – https://numpy.org/doc/1.20/reference/generated/numpy.nan_to_num.html
- Miscellaneous (IEEE 754 special values) – https://numpy.org/doc/stable/user/misc.html
- numpy.errstate – https://numpy.org/doc/stable/reference/generated/numpy.errstate.html