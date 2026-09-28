# NumPy Mathematical Functions: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** NumPy mathematical functions are universal functions (ufuncs) that perform element-wise mathematical operations on arrays, including absolute values, roots, powers, exponentials, logarithms, trigonometric, inverse trigonometric, and hyperbolic functions.

**Technical Definition:** These functions are implemented as `numpy.ufunc` objects — compiled C functions that operate element-by-element on ndarrays, supporting broadcasting, type casting, and the `out`/`where` parameters. They are accessed either as standalone functions (e.g., `np.sqrt(x)`) or as methods on ndarray objects (e.g., `x.sqrt()` is not standard; use `np.sqrt(x)`).

**Beginner-Friendly Explanation:** NumPy provides a complete toolbox of mathematical functions that work on entire arrays at once. Instead of computing `sin` of one number at a time, you can compute `sin` of a million numbers in a single call. These functions are the building blocks for scientific computing, signal processing, machine learning, and engineering simulations.

### Key Characteristics

- **Element-Wise:** Each function operates independently on every element of an array.
- **Vectorized:** Implemented in compiled C for high performance.
- **Broadcasting Support:** Scalars and arrays of different shapes are automatically aligned.
- **Domain Handling:** Negative inputs to `sqrt` return `nan` for real dtypes; complex dtypes yield complex results.
- **Numerical Stability Variants:** `expm1` and `log1p` avoid precision loss near zero.
- **Radians by Default:** Trigonometric functions use radians; `degrees`/`radians` convert units.

### Prerequisites

- NumPy installed (`pip install numpy`).
- Basic Python syntax and understanding of variables.
- Familiarity with NumPy array creation and the `import numpy as np` convention.
- Conceptual understanding of broadcasting.

### Related Programming Areas

- **Scientific Computing:** Solving differential equations, simulating physical systems.
- **Signal Processing:** Fourier transforms, filtering, waveform generation.
- **Machine Learning:** Activation functions (sigmoid, tanh, ReLU), loss functions, gradient computation.
- **Physics and Engineering:** Wave propagation, oscillatory systems, thermodynamics.
- **Data Analysis:** Logarithmic scaling, normalization, distance computations.

### Core Concepts / Features

The following function categories are explored in detail: (1) Absolute values, (2) Square roots, (3) Powers, (4) Exponentials, (5) Logarithms, (6) Trigonometric functions, (7) Inverse trigonometric functions, and (8) Hyperbolic functions.

---

## Core Concept 1: Absolute Values

### Definitions

**Core Definition:** `np.abs()` (alias `np.absolute()`) computes the absolute (non-negative) value of each element, and `np.angle()` returns the phase angle of complex numbers.

**Technical Definition:** `numpy.absolute(x, /, out=None, *, where=True, casting='same_kind', order='K', dtype=None, subok=True)` calculates the absolute value element-wise. `np.abs` is a shorthand for this function. For complex input `a + ib`, the absolute value is \(\sqrt{a^2 + b^2}\). `numpy.angle(z)` returns the argument (angle) of a complex number, between \(-\pi\) and \(\pi\).

**Beginner-Friendly Explanation:** `np.abs()` strips the sign from a number, making it always non-negative. For complex numbers, it returns the "magnitude" — the distance from the origin. `np.angle()` tells you the direction of a complex number as an angle.

### Purposes

- To compute magnitudes and distances regardless of sign.
- To extract the phase of complex-valued signals.
- To measure errors or deviations in absolute terms.
- To compute norms and losses in machine learning.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.absolute(x, out=None, where=True)
np.abs(x)                    # Shorthand
np.angle(z, deg=False)       # Phase of complex number
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `x` | Input array or scalar. |
| `out` | Optional output array. |
| `where` | Boolean condition for selective computation. |
| `z` | Complex-valued input for `angle`. |
| `deg` | If `True`, return angle in degrees instead of radians. |

**Syntax Rules:**

- `np.abs` handles complex numbers; `np.fabs` does not.
- For complex input, the absolute value is the magnitude \(r = \sqrt{a^2 + b^2}\).
- `np.angle` returns values in \((-\pi, \pi]\).

**Constraints and Limitations:**

- `np.fabs` is faster but does not handle complex numbers.
- `np.angle(0)` returns 0 by convention.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Absolute Values and Complex Magnitudes**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Real-valued absolute values.
x = np.array([-1.2, 1.2, -3.5, 0.0])
print("np.abs(x):", np.abs(x))
print()

# Step 3: Complex absolute values.
z = np.array([3 + 4j, 1 - 1j])
print("Complex array:", z)
print("np.abs(z) (magnitudes):", np.abs(z))
print("np.angle(z) (radians):", np.angle(z))
print("np.angle(z, deg=True):", np.angle(z, deg=True))
```

**Expected Output:**

```
np.abs(x): [1.2 1.2 3.5 0. ]

Complex array: [3.+4.j 1.-1.j]
np.abs(z) (magnitudes): [5.         1.41421356]
np.angle(z) (radians): [0.92729522  -0.78539816]
np.angle(z, deg=True): [ 53.13010235 -45.        ]
```

**Why This Result Occurs:** `np.abs` on `[−1.2, 1.2, −3.5, 0.0]` removes signs. For `3+4j`, the magnitude is \(\sqrt{3^2+4^2}=5\). The angle of `3+4j` is \(\arctan(4/3) \approx 0.927\) radians ≈ 53.13°.

### Real-World Cases

- **Signal Processing:** Extracting the envelope of a complex signal via `np.abs`.
- **Machine Learning:** Computing L1 loss with `np.abs(predictions - targets)`.
- **Physics:** Computing magnitudes of complex wavefunctions.

### References

- numpy.absolute – https://numpy.org/doc/stable/reference/generated/numpy.absolute.html
- numpy.angle – https://numpy.org/doc/stable/reference/generated/numpy.angle.html

---

## Core Concept 2: Square Roots

### Definitions

**Core Definition:** `np.sqrt()` computes the non-negative square root, and `np.cbrt()` computes the cube root (which accepts negative inputs).

**Technical Definition:** `numpy.sqrt(x)` returns the positive square root of each element. If any element is complex, a complex array is returned (negative reals yield complex square roots). If all elements are real, negative elements return `nan`. `numpy.cbrt(x)` returns the cube root and correctly handles negative inputs, returning negative results.

**Beginner-Friendly Explanation:** `np.sqrt` gives the square root — the number that, when multiplied by itself, equals the input. It doesn't work on negative numbers (returns `nan` for real dtypes). `np.cbrt` gives the cube root and works on negatives too, returning a negative cube root.

### Purposes

- To compute distances, norms, and standard deviations.
- To normalize data using root-based transformations.
- To compute Euclidean distances in machine learning.
- To evaluate root-based mathematical formulas.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.sqrt(x, out=None, where=True)
np.cbrt(x, out=None, where=True)
```

**Component Breakdown:**

| Function | Negative Input | Complex Input |
|----------|---------------|---------------|
| `np.sqrt` | Returns `nan` (real dtype) | Returns complex sqrt |
| `np.cbrt` | Returns negative cube root | Returns complex cube root |

**Syntax Rules:**

- `np.sqrt` of a negative real returns `nan` and a `RuntimeWarning`.
- `np.sqrt` of a complex number returns a complex result.
- `np.cbrt` handles negative reals correctly (returns negative).
- `np.lib.scimath.sqrt` returns complex results for negative reals.

**Constraints and Limitations:**

- `np.sqrt` on negative integers may raise a warning and return `nan`.
- `np.cbrt` is not available in older NumPy versions (new in 1.10.0).

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Square Roots and Cube Roots**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Square roots.
x = np.array([1, 4, 9, 16])
print("np.sqrt(x):", np.sqrt(x))
print()

# Step 3: Square root of negative (real dtype).
neg = np.array([-1.0, 4.0])
print("np.sqrt([-1, 4]):", np.sqrt(neg))
print("  (nan for negative real)")
print()

# Step 4: Square root of complex.
z = np.array([-1 + 0j])
print("np.sqrt(-1+0j):", np.sqrt(z))
print()

# Step 5: Cube roots.
y = np.array([1, 8, 27, -8])
print("np.cbrt(y):", np.cbrt(y))
print("  (negative inputs handled correctly)")
```

**Expected Output:**

```
np.sqrt(x): [1. 2. 3. 4.]

np.sqrt([-1, 4]): [nan  2.]
  (nan for negative real)

np.sqrt(-1+0j): [0.+1.j]

np.cbrt(y): [ 1.  2.  3. -2.]
  (negative inputs handled correctly)
```

**Why This Result Occurs:** `np.sqrt` of `[1, 4, 9, 16]` gives `[1, 2, 3, 4]`. For `-1.0`, it returns `nan` because the square root of a negative real is not real. Using the complex `-1+0j` yields the complex square root `1j`. `np.cbrt` handles `-8` correctly, returning `-2` because `(-2)³ = -8`.

### Real-World Cases

- **Distance Calculations:** Euclidean distance uses `np.sqrt(np.sum((a - b) ** 2))`.
- **Standard Deviation:** `np.sqrt(np.var(x))`.
- **Geometry:** Computing the hypotenuse of right triangles with `np.hypot` (which is `sqrt(x² + y²)`).

### References

- numpy.sqrt – https://numpy.org/doc/1.20/reference/generated/numpy.sqrt.html
- numpy.cbrt – https://numpy.org/doc/1.19/reference/generated/numpy.cbrt.html
- numpy.lib.scimath.sqrt – https://numpy.org/doc/stable/reference/generated/numpy.lib.scimath.sqrt.html

---

## Core Concept 3: Powers

### Definitions

**Core Definition:** `np.power()` raises each base to a corresponding exponent, element-wise, with `**` as a shorthand operator.

**Technical Definition:** `numpy.power(x1, x2)` raises each base in `x1` to the positionally-corresponding power in `x2`. `x1` and `x2` must be broadcastable. An integer type raised to a negative integer power raises a `ValueError`. Negative values raised to a non-integral value result in `nan`.

**Beginner-Friendly Explanation:** `np.power(a, b)` computes `a` raised to the power `b` for each element. You can square numbers (`x ** 2`), take square roots (`x ** 0.5`), or raise each element to a different power. The `**` operator is shorthand for `np.power`.

### Purposes

- To square or cube values for polynomial computations.
- To compute roots using fractional exponents.
- To implement power transformations for statistical normalization.
- To compute distances in Euclidean space.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.power(x1, x2, out=None, where=True, dtype=None)
x1 ** x2                    # Shorthand
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `x1` | Base array or scalar. |
| `x2` | Exponent array or scalar. |
| `out` | Optional output array. |
| `dtype` | Optional target dtype. |

**Syntax Rules:**

- `x1` and `x2` must be broadcastable to a common shape.
- Integer base raised to a negative integer power raises `ValueError`.
- Negative base with fractional exponent returns `nan` (real dtype).
- Use `np.float_power` to promote integers to floats for power operations.

**Constraints and Limitations:**

- `0 ** 0` is defined as 1 in NumPy.
- Large exponents can cause overflow to `inf`.
- `np.power` preserves type; `np.float_power` promotes to float.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Powers with Scalars and Arrays**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Square each element.
x = np.array([1, 2, 3, 4, 5])
print("x ** 2:", x ** 2)
print()

# Step 3: Raise to different exponents.
exponents = np.array([1, 2, 3, 4, 5])
print("np.power(x, exponents):", np.power(x, exponents))
print("  (1^1, 2^2, 3^3, 4^4, 5^5)")
print()

# Step 4: Fractional exponent (square root).
print("x ** 0.5:", x ** 0.5)
print()

# Step 5: Negative base with fractional exponent (real).
neg = np.array([-8.0])
print("np.power(-8, 1/3):", np.power(neg, 1/3))
print("  (nan because negative base with fractional exponent)")
```

**Expected Output:**

```
x ** 2: [ 1  4  9 16 25]

np.power(x, exponents): [   1    4   27  256 3125]
  (1^1, 2^2, 3^3, 4^4, 5^5)

x ** 0.5: [1.         1.41421356 1.73205081 2.         2.23606798]

np.power(-8, 1/3): [nan]
  (nan because negative base with fractional exponent)
```

**Why This Result Occurs:** `x ** 2` squares each element. `np.power(x, exponents)` raises each element to the corresponding exponent. `x ** 0.5` takes the square root. `np.power(-8, 1/3)` returns `nan` because the real cube root of a negative number requires an odd-integer denominator, and floating-point `1/3` is not exact.

### Real-World Cases

- **Distance Calculations:** `np.sqrt(np.sum((a - b) ** 2))`.
- **Physics:** Computing kinetic energy: `0.5 * m * v ** 2`.
- **Statistics:** Power transformations for variance stabilization.

### References

- numpy.power – https://numpy.org/doc/stable/reference/generated/numpy.power.html
- numpy.float_power – https://numpy.org/doc/stable/reference/generated/numpy.float_power.html

---

## Core Concept 4: Exponentials

### Definitions

**Core Definition:** `np.exp()` computes \(e^x\) and `np.expm1()` computes \(e^x - 1\) with higher numerical stability for small \(x\).

**Technical Definition:** `numpy.exp(x)` calculates the exponential of all elements. `numpy.expm1(x)` computes \(e^x - 1\) more accurately than `np.exp(x) - 1` when \(x\) is close to zero, avoiding the loss of precision from subtracting nearly equal numbers.

**Beginner-Friendly Explanation:** `np.exp(x)` computes \(e\) (≈2.718) raised to the power `x`. `np.expm1(x)` computes \(e^x - 1\), which is important because for very small `x`, `np.exp(x)` is very close to 1, and subtracting 1 loses precision. The `expm1` function avoids this loss.

### Purposes

- To compute exponential growth and decay.
- To implement sigmoid and softmax activation functions.
- To compute probabilities and likelihoods.
- To solve differential equations involving exponential terms.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.exp(x, out=None, where=True)
np.expm1(x, out=None, where=True)
```

**Component Breakdown:**

| Function | Formula | Best For |
|----------|---------|----------|
| `np.exp(x)` | \(e^x\) | General exponential. |
| `np.expm1(x)` | \(e^x - 1\) | Small \(x\) (avoids precision loss). |

**Syntax Rules:**

- `np.exp` of very large values overflows to `inf`.
- `np.expm1` is recommended when the result is used in a formula involving `exp(x) - 1`.
- Both functions operate element-wise.

**Constraints and Limitations:**

- `np.expm1` has no benefit over `np.exp` if the result is not used in a `-1` expression.
- For `x > 1`, `np.exp(x) - 1` is accurate enough.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Exponentials and Numerical Stability**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Basic exponential.
x = np.array([0, 1, 2, 3])
print("np.exp(x):", np.exp(x))
print()

# Step 3: expm1 for small x.
small = np.array([1e-10, 1e-8, 1e-5])
print("np.expm1(small):", np.expm1(small))
print("np.exp(small) - 1:", np.exp(small) - 1)
print("  (expm1 is more accurate)")
print()

# Step 4: Larger x.
large = np.array([1.0, 2.0])
print("np.expm1(large):", np.expm1(large))
print("np.exp(large) - 1:", np.exp(large) - 1)
print("  (both accurate for larger x)")
```

**Expected Output:**

```
np.exp(x): [ 1.          2.71828183  7.3890561  20.08553692]

np.expm1(small): [1.00000000e-10 1.00000000e-08 1.00000500e-05]
np.exp(small) - 1: [1.00000008e-10 1.00000008e-08 1.00000500e-05]
  (expm1 is more accurate)

np.expm1(large): [ 1.71828183  6.3890561 ]
np.exp(large) - 1: [ 1.71828183  6.3890561 ]
  (both accurate for larger x)
```

**Why This Result Occurs:** For `x = 1e-10`, `np.exp(x)` is approximately `1.0000000001`, and subtracting 1 loses significant digits. `np.expm1` computes the result directly without this subtraction, preserving accuracy. For larger `x`, the difference is negligible.

### Real-World Cases

- **Machine Learning:** Sigmoid activation uses `1 / (1 + np.exp(-x))`.
- **Physics:** Radioactive decay, population growth models.
- **Finance:** Continuous compound interest calculations.

### References

- numpy.exp – https://numpy.org/doc/stable/reference/generated/numpy.exp.html
- numpy.expm1 – https://numpy.org/doc/stable/reference/generated/numpy.expm1.html
- Stack Overflow: expm1 vs exp-1 – https://stackoverflow.com/questions/47725508

---

## Core Concept 5: Logarithms

### Definitions

**Core Definition:** NumPy provides natural logarithm (`np.log`), base-10 (`np.log10`), base-2 (`np.log2`), and `np.log1p` for \(\log(1+x)\) with high precision near zero.

**Technical Definition:** `numpy.log(x)` computes the natural logarithm (base \(e\)), the inverse of `np.exp`. `numpy.log10(x)` computes base-10 logarithms. `numpy.log2(x)` computes base-2 logarithms. `numpy.log1p(x)` computes \(\log(1+x)\) accurately for small \(x\), avoiding precision loss from adding 1 to a small number.

**Beginner-Friendly Explanation:** A logarithm answers the question "what power must I raise the base to, to get this number?" `np.log` uses base \(e\), `np.log10` uses base 10, and `np.log2` uses base 2. `np.log1p` computes \(\log(1+x)\) accurately when \(x\) is very small.

### Purposes

- To compress wide-ranging data for visualization.
- To compute log-likelihoods and information-theoretic quantities.
- To transform exponential relationships into linear ones.
- To handle probabilities that are very close to 1.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.log(x, out=None, where=True)      # Natural log
np.log10(x, out=None, where=True)    # Base-10 log
np.log2(x, out=None, where=True)     # Base-2 log
np.log1p(x, out=None, where=True)    # log(1 + x)
```

**Component Breakdown:**

| Function | Base | Domain | Best For |
|----------|------|--------|----------|
| `np.log` | \(e\) | \(x > 0\) | Natural phenomena |
| `np.log10` | 10 | \(x > 0\) | Decibel, pH scales |
| `np.log2` | 2 | \(x > 0\) | Information theory |
| `np.log1p` | \(e\) | \(x > -1\) | Small \(x\) near 0 |

**Syntax Rules:**

- `np.log(0)` returns `-inf`; `np.log(negative)` returns `nan` for real dtypes.
- `np.log1p` is accurate for small \(x\) where `np.log(1 + x)` would lose precision.
- `np.log2` is approximately 3× faster than `np.log` and `np.log10` on some platforms.

**Constraints and Limitations:**

- Logarithms are not defined for negative numbers or zero in the real domain.
- For complex inputs, `np.log` returns the principal value.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Logarithms and Numerical Stability**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Natural logarithm.
x = np.array([1, np.e, np.e**2, 0])
print("np.log(x):", np.log(x))
print()

# Step 3: Base-10 and base-2 logarithms.
y = np.array([1, 10, 100, 1000])
print("np.log10(y):", np.log10(y))
print("np.log2(y):", np.log2(y))
print()

# Step 4: log1p for small values.
small = np.array([1e-10, 1e-8, 1e-5])
print("np.log1p(small):", np.log1p(small))
print("np.log(1 + small):", np.log(1 + small))
print("  (log1p is more accurate for small x)")
```

**Expected Output:**

```
np.log(x): [ 0.  1.  2. -inf]

np.log10(y): [0. 1. 2. 3.]
np.log2(y): [0. 1. 2. 3.]

np.log1p(small): [1.00000000e-10 1.00000000e-08 1.00000500e-05]
np.log(1 + small): [1.00000008e-10 1.00000008e-08 1.00000500e-05]
  (log1p is more accurate for small x)
```

**Why This Result Occurs:** `np.log` of `e` is 1, and `np.log(0)` is `-inf`. `np.log10` of `[1, 10, 100, 1000]` gives `[0, 1, 2, 3]`. For small `x`, `np.log1p(x)` is more accurate than `np.log(1 + x)` because adding 1 to a small number loses precision.

### Real-World Cases

- **Machine Learning:** Cross-entropy loss uses `np.log` for probabilities.
- **Signal Processing:** Decibel calculations use `np.log10`.
- **Information Theory:** Entropy calculations use `np.log2`.

### References

- numpy.log – https://numpy.org/doc/1.22/reference/generated/numpy.log.html
- numpy.log10 – https://numpy.org/doc/stable/reference/generated/numpy.log10.html
- numpy.log2 – https://numpy.org/doc/stable/reference/generated/numpy.log2.html
- numpy.log1p – https://numpy.org/doc/stable/reference/generated/numpy.log1p.html

---

## Core Concept 6: Trigonometric Functions

### Definitions

**Core Definition:** NumPy provides `np.sin()`, `np.cos()`, and `np.tan()` for trigonometric computations, with `np.degrees()` and `np.radians()` for unit conversion.

**Technical Definition:** All trigonometric functions take angles in radians. `numpy.sin(x)`, `numpy.cos(x)`, and `numpy.tan(x)` compute the sine, cosine, and tangent element-wise. `numpy.degrees(x)` converts radians to degrees; `numpy.radians(x)` converts degrees to radians. `numpy.deg2rad` and `numpy.rad2deg` are aliases.

**Beginner-Friendly Explanation:** Trigonometry deals with the relationships between angles and sides of triangles. `np.sin`, `np.cos`, and `np.tan` compute these ratios for any angle. NumPy expects angles in radians, but you can convert with `np.degrees` and `np.radians`.

### Purposes

- To generate and analyze waveforms.
- To model periodic phenomena.
- To compute angles and distances in geometry.
- To implement rotations and transformations in computer graphics.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.sin(x, out=None, where=True)
np.cos(x, out=None, where=True)
np.tan(x, out=None, where=True)
np.degrees(x)      # Radians -> Degrees
np.radians(x)      # Degrees -> Radians
```

**Component Breakdown:**

| Function | Input | Output |
|----------|-------|--------|
| `np.sin` | Radians | \([-1, 1]\) |
| `np.cos` | Radians | \([-1, 1]\) |
| `np.tan` | Radians | \((-\infty, \infty)\) |
| `np.degrees` | Radians | Degrees |
| `np.radians` | Degrees | Radians |

**Syntax Rules:**

- All trigonometric functions take radians by default.
- `np.degrees` and `np.radians` are for unit conversion.
- `np.tan` has singularities at odd multiples of \(\pi/2\).

**Constraints and Limitations:**

- Floating-point precision causes `np.sin(np.pi)` to be a very small number, not exactly 0.
- `np.tan` of \(\pi/2\) returns a very large number, not exactly infinity.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Trigonometric Functions and Unit Conversion**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Compute sine, cosine, tangent for common angles.
angles_deg = np.array([0, 30, 45, 60, 90])
angles_rad = np.radians(angles_deg)
print("Angles (radians):", angles_rad)
print()

print("np.sin:", np.sin(angles_rad))
print("np.cos:", np.cos(angles_rad))
print("np.tan:", np.tan(angles_rad))
print()

# Step 3: Convert radians back to degrees.
print("np.degrees(angles_rad):", np.degrees(angles_rad))
print()

# Step 4: Note the floating-point precision issue.
print("np.sin(np.pi):", np.sin(np.pi))
print("  (not exactly 0 due to floating-point precision)")
```

**Expected Output:**

```
Angles (radians): [0.         0.52359878 0.78539816 1.04719755 1.57079633]

np.sin: [0.         0.5        0.70710678 0.8660254  1.        ]
np.cos: [1.00000000e+00 8.66025404e-01 7.07106781e-01 5.00000000e-01
 6.12323400e-17]
np.tan: [0.00000000e+00 5.77350269e-01 1.00000000e+00 1.73205081e+00
 1.63312394e+16]

np.degrees(angles_rad): [ 0. 30. 45. 60. 90.]

np.sin(np.pi): 1.2246467991473532e-16
  (not exactly 0 due to floating-point precision)
```

**Why This Result Occurs:** `np.sin(30°) = 0.5`, `np.cos(60°) = 0.5`, etc. The tangent of 90° is a very large number (not exactly infinity) due to floating-point representation. `np.sin(np.pi)` is `1.22e-16`, not 0, because `np.pi` is not exactly \(\pi\).

### Real-World Cases

- **Signal Processing:** Generating sine waves for audio and communication signals.
- **Computer Graphics:** Rotating objects using sine and cosine.
- **Physics:** Modeling oscillatory motion (pendulums, springs).

### References

- numpy.sin – https://numpy.org/doc/2.0/reference/generated/numpy.sin.html
- numpy.cos – https://numpy.org/doc/stable/reference/generated/numpy.cos.html
- numpy.tan – https://numpy.org/doc/stable/reference/generated/numpy.tan.html
- numpy.degrees – https://numpy.org/doc/stable/reference/generated/numpy.degrees.html
- numpy.radians – https://numpy.org/doc/stable/reference/generated/numpy.radians.html

---

## Core Concept 7: Inverse Trigonometric Functions

### Definitions

**Core Definition:** NumPy provides inverse trigonometric functions `np.arcsin()`, `np.arccos()`, `np.arctan()`, and `np.arctan2()` for recovering angles from trigonometric ratios.

**Technical Definition:** `numpy.arcsin(x)` returns the inverse sine, in radians and in the closed interval \([-\pi/2, \pi/2]\). `numpy.arccos(x)` returns the inverse cosine in \([0, \pi]\). `numpy.arctan(x)` returns the inverse tangent in \([-\pi/2, \pi/2]\). `numpy.arctan2(y, x)` returns the signed angle of the point \((x, y)\), correctly choosing the quadrant and handling division by zero.

**Beginner-Friendly Explanation:** Inverse trig functions answer "what angle produces this sine/cosine/tangent?" `np.arcsin` gives an angle from a sine value, and so on. `np.arctan2(y, x)` is special: it takes two arguments and figures out the correct quadrant of the angle, avoiding the ambiguity of a single-argument arctangent.

### Purposes

- To recover angles from trigonometric ratios.
- To compute directions and bearings in navigation.
- To resolve quadrants correctly in signal processing and robotics.
- To avoid division-by-zero errors in angle computations.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.arcsin(x, out=None, where=True)
np.arccos(x, out=None, where=True)
np.arctan(x, out=None, where=True)
np.arctan2(y, x, out=None, where=True)
```

**Component Breakdown:**

| Function | Domain | Range | Description |
|----------|--------|-------|-------------|
| `np.arcsin` | \([-1, 1]\) | \([-\pi/2, \pi/2]\) | Inverse sine |
| `np.arccos` | \([-1, 1]\) | \([0, \pi]\) | Inverse cosine |
| `np.arctan` | \((-\infty, \infty)\) | \([-\pi/2, \pi/2]\) | Inverse tangent |
| `np.arctan2` | \((y, x)\) | \([-\pi, \pi]\) | Four-quadrant arctangent |

**Syntax Rules:**

- `np.arcsin` and `np.arccos` return `nan` for inputs outside \([-1, 1]\).
- `np.arctan2(y, x)` chooses the quadrant based on the signs of both arguments.
- `np.arctan2(0, 0)` returns 0.

**Constraints and Limitations:**

- `np.arcsin` and `np.arccos` are only defined for \([-1, 1]\).
- `np.arctan2` is preferred over `np.arctan(y/x)` because it handles all quadrants and avoids division by zero.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Inverse Trigonometric Functions and Quadrants**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Inverse sine, cosine, tangent.
print("np.arcsin(1):", np.arcsin(1))
print("np.arcsin(-1):", np.arcsin(-1))
print("np.arccos(0):", np.arccos(0))
print("np.arctan(1):", np.arctan(1))
print()

# Step 3: arctan2 for quadrant resolution.
# Point (1, 1) in Q1, (-1, 1) in Q2, (-1, -1) in Q3, (1, -1) in Q4.
points = [(1, 1), (-1, 1), (-1, -1), (1, -1)]
for x, y in points:
    angle = np.arctan2(y, x)
    print(f"arctan2({y}, {x}) = {angle:.4f} rad = {np.degrees(angle):.2f}°")
print()

# Step 4: Comparison with arctan(y/x) — loses quadrant information.
print("np.arctan(1/1):", np.arctan(1/1))
print("np.arctan(-1/-1):", np.arctan(-1/-1))
print("  (both give 0.785, but arctan2 distinguishes quadrants)")
```

**Expected Output:**

```
np.arcsin(1): 1.5707963267948966
np.arcsin(-1): -1.5707963267948966
np.arccos(0): 1.5707963267948966
np.arctan(1): 0.7853981633974483

arctan2(1, 1) = 0.7854 rad = 45.00°
arctan2(1, -1) = 2.3562 rad = 135.00°
arctan2(-1, -1) = -2.3562 rad = -135.00°
arctan2(-1, 1) = -0.7854 rad = -45.00°

np.arctan(1/1): 0.7853981633974483
np.arctan(-1/-1): 0.7853981633974483
  (both give 0.785, but arctan2 distinguishes quadrants)
```

**Why This Result Occurs:** `np.arcsin(1)` returns \(\pi/2\). `np.arctan2` correctly distinguishes the four quadrants: `(1, 1)` gives 45°, `(-1, 1)` gives 135°, etc. `np.arctan(y/x)` cannot distinguish Q1 from Q3 because both give the same ratio.

### Real-World Cases

- **Robotics:** Computing joint angles from end-effector positions.
- **Navigation:** Computing bearings from coordinate differences.
- **Signal Processing:** Phase extraction from I/Q data.

### References

- numpy.arcsin – https://numpy.org/doc/stable/reference/generated/numpy.arcsin.html
- numpy.arccos – https://numpy.org/doc/stable/reference/generated/numpy.arccos.html
- numpy.arctan – https://numpy.org/doc/stable/reference/generated/numpy.arctan.html
- numpy.arctan2 – https://numpy.org/doc/stable/reference/generated/numpy.arctan2.html

---

## Core Concept 8: Hyperbolic Functions

### Definitions

**Core Definition:** NumPy provides hyperbolic functions `np.sinh()`, `np.cosh()`, `np.tanh()` and their inverses `np.arcsinh()`, `np.arccosh()`, `np.arctanh()`.

**Technical Definition:** `numpy.sinh(x)` computes the hyperbolic sine: \(\frac{1}{2}(e^x - e^{-x})\). `numpy.cosh(x)` computes \(\frac{1}{2}(e^x + e^{-x})\). `numpy.tanh(x)` computes \(\frac{\sinh(x)}{\cosh(x)}\). Their inverses (`arcsinh`, `arccosh`, `arctanh`) return the corresponding angle in radians.

**Beginner-Friendly Explanation:** Hyperbolic functions are like trigonometric functions but based on the hyperbola instead of the circle. `np.tanh` is widely used in machine learning as an activation function. The inverse hyperbolic functions recover the angle from a hyperbolic ratio.

### Purposes

- To implement activation functions in neural networks (`np.tanh`).
- To model catenary curves, heat transfer, and special relativity.
- To solve differential equations involving hyperbolic terms.
- To compute inverse hyperbolic relationships.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.sinh(x, out=None, where=True)
np.cosh(x, out=None, where=True)
np.tanh(x, out=None, where=True)
np.arcsinh(x, out=None, where=True)
np.arccosh(x, out=None, where=True)
np.arctanh(x, out=None, where=True)
```

**Component Breakdown:**

| Function | Formula | Domain | Range |
|----------|---------|--------|-------|
| `np.sinh` | \((e^x - e^{-x})/2\) | \((-\infty, \infty)\) | \((-\infty, \infty)\) |
| `np.cosh` | \((e^x + e^{-x})/2\) | \((-\infty, \infty)\) | \([1, \infty)\) |
| `np.tanh` | \(\sinh/\cosh\) | \((-\infty, \infty)\) | \((-1, 1)\) |
| `np.arcsinh` | Inverse sinh | \((-\infty, \infty)\) | \((-\infty, \infty)\) |
| `np.arccosh` | Inverse cosh | \([1, \infty)\) | \([0, \infty)\) |
| `np.arctanh` | Inverse tanh | \((-1, 1)\) | \((-\infty, \infty)\) |

**Syntax Rules:**

- `np.tanh` maps all real numbers to \((-1, 1)\).
- `np.arccosh` requires input \(\geq 1\).
- `np.arctanh` requires input in \((-1, 1)\).

**Constraints and Limitations:**

- `np.cosh` grows exponentially for large \(x\).
- `np.arccosh` returns `nan` for inputs less than 1.
- `np.arctanh` returns `inf` for inputs equal to ±1.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Hyperbolic Functions and Activation**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Hyperbolic sine, cosine, tangent.
x = np.array([0, 1, 2, 3])
print("np.sinh(x):", np.sinh(x))
print("np.cosh(x):", np.cosh(x))
print("np.tanh(x):", np.tanh(x))
print()

# Step 3: tanh as an activation function (range -1 to 1).
z = np.linspace(-3, 3, 7)
print("z:", z)
print("np.tanh(z):", np.tanh(z))
print("  (squashes values to (-1, 1))")
print()

# Step 4: Inverse hyperbolic functions.
print("np.arcsinh(1):", np.arcsinh(1))
print("np.arccosh(1):", np.arccosh(1))
print("np.arctanh(0.5):", np.arctanh(0.5))
```

**Expected Output:**

```
np.sinh(x): [ 0.          1.17520119  3.62686041 10.01787493]
np.cosh(x): [ 1.          1.54308063  3.76219569 10.067662  ]
np.tanh(x): [0.         0.76159416 0.96402758 0.99505475]

z: [-3. -2. -1.  0.  1.  2.  3.]
np.tanh(z): [-0.99505475 -0.96402758 -0.76159416  0.          0.76159416
  0.96402758  0.99505475]
  (squashes values to (-1, 1))

np.arcsinh(1): 0.881373587019543
np.arccosh(1): 0.0
np.arctanh(0.5): 0.5493061443340549
```

**Why This Result Occurs:** `np.tanh` maps all inputs to the interval \((-1, 1)\), making it useful as an activation function. `np.cosh` grows exponentially. The inverse functions recover the original angles.

### Real-World Cases

- **Machine Learning:** `np.tanh` is used as an activation function in neural networks.
- **Physics:** Special relativity uses hyperbolic functions for rapidity.
- **Engineering:** Catenary curves (hanging cables) are modeled with `np.cosh`.

### References

- numpy.sinh – https://numpy.org/doc/stable/reference/generated/numpy.sinh.html
- numpy.cosh – https://numpy.org/doc/stable/reference/generated/numpy.cosh.html
- numpy.tanh – https://numpy.org/doc/stable/reference/generated/numpy.tanh.html
- numpy.arcsinh – https://numpy.org/doc/stable/reference/generated/numpy.arcsinh.html
- numpy.arccosh – https://numpy.org/doc/stable/reference/generated/numpy.arccosh.html
- numpy.arctanh – https://numpy.org/doc/stable/reference/generated/numpy.arctanh.html

---

## Summary of Mathematical Functions

| Category | Key Functions | Notes |
|----------|--------------|-------|
| Absolute values | `np.abs`, `np.absolute`, `np.angle` | Complex-aware. |
| Square roots | `np.sqrt`, `np.cbrt` | `sqrt` returns `nan` for negative reals. |
| Powers | `np.power`, `**` | Negative base with fractional exponent → `nan`. |
| Exponentials | `np.exp`, `np.expm1` | `expm1` for numerical stability near 0. |
| Logarithms | `np.log`, `np.log10`, `np.log2`, `np.log1p` | `log1p` for small \(x\). |
| Trigonometric | `np.sin`, `np.cos`, `np.tan`, `np.degrees`, `np.radians` | Radians by default. |
| Inverse trig | `np.arcsin`, `np.arccos`, `np.arctan`, `np.arctan2` | `arctan2` for quadrants. |
| Hyperbolic | `np.sinh`, `np.cosh`, `np.tanh`, `np.arcsinh`, `np.arccosh`, `np.arctanh` | `tanh` for activation. |

---

## Summary of Deprecated and Version-Specific Features

| Feature | Status | Recommendation |
|---------|--------|----------------|
| `np.cbrt` | New in NumPy 1.10.0 | Use for negative cube roots. |
| `np.lib.scimath.sqrt` | Available | Use for complex square roots of negative reals. |
| `np.expm1` / `np.log1p` | Available | Use for numerical stability near zero. |
| `np.arcsin` vs `np.asin` | Both available | `asin` is an alias. |

---

## Consolidated Reference List

- numpy.absolute – https://numpy.org/doc/stable/reference/generated/numpy.absolute.html
- numpy.angle – https://numpy.org/doc/stable/reference/generated/numpy.angle.html
- numpy.sqrt – https://numpy.org/doc/1.20/reference/generated/numpy.sqrt.html
- numpy.cbrt – https://numpy.org/doc/1.19/reference/generated/numpy.cbrt.html
- numpy.power – https://numpy.org/doc/stable/reference/generated/numpy.power.html
- numpy.float_power – https://numpy.org/doc/stable/reference/generated/numpy.float_power.html
- numpy.exp – https://numpy.org/doc/stable/reference/generated/numpy.exp.html
- numpy.expm1 – https://numpy.org/doc/stable/reference/generated/numpy.expm1.html
- numpy.log – https://numpy.org/doc/1.22/reference/generated/numpy.log.html
- numpy.log10 – https://numpy.org/doc/stable/reference/generated/numpy.log10.html
- numpy.log2 – https://numpy.org/doc/stable/reference/generated/numpy.log2.html
- numpy.log1p – https://numpy.org/doc/stable/reference/generated/numpy.log1p.html
- numpy.sin – https://numpy.org/doc/2.0/reference/generated/numpy.sin.html
- numpy.cos – https://numpy.org/doc/stable/reference/generated/numpy.cos.html
- numpy.tan – https://numpy.org/doc/stable/reference/generated/numpy.tan.html
- numpy.degrees – https://numpy.org/doc/stable/reference/generated/numpy.degrees.html
- numpy.radians – https://numpy.org/doc/stable/reference/generated/numpy.radians.html
- numpy.arcsin – https://numpy.org/doc/stable/reference/generated/numpy.arcsin.html
- numpy.arccos – https://numpy.org/doc/stable/reference/generated/numpy.arccos.html
- numpy.arctan – https://numpy.org/doc/stable/reference/generated/numpy.arctan.html
- numpy.arctan2 – https://numpy.org/doc/stable/reference/generated/numpy.arctan2.html
- numpy.sinh – https://numpy.org/doc/stable/reference/generated/numpy.sinh.html
- numpy.cosh – https://numpy.org/doc/stable/reference/generated/numpy.cosh.html
- numpy.tanh – https://numpy.org/doc/stable/reference/generated/numpy.tanh.html
- numpy.arcsinh – https://numpy.org/doc/stable/reference/generated/numpy.arcsinh.html
- numpy.arccosh – https://numpy.org/doc/stable/reference/generated/numpy.arccosh.html
- numpy.arctanh – https://numpy.org/doc/stable/reference/generated/numpy.arctanh.html
- Stack Overflow: expm1 vs exp-1 – https://stackoverflow.com/questions/47725508
- Mathematical functions – https://numpy.org/doc/2.4/reference/routines.math.html