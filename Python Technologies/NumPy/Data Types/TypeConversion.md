# NumPy Type Conversion & Promotion: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Type conversion in NumPy is the process of changing an array's data type from one dtype to another, while type promotion is the automatic determination of a common dtype when operations involve arrays of different types.

**Technical Definition:** Type conversion is performed explicitly via `ndarray.astype()` or implicitly during operations that require a common dtype. Type promotion follows NumPy's rules (defined in NEP 50) to find a result dtype that can safely represent both inputs. Casting safety is controlled by the `casting` parameter, which determines whether a conversion is permitted based on value-preservation guarantees.

**Beginner-Friendly Explanation:** Sometimes you need to change the kind of numbers in an array—say, from integers to floats, or from high-precision floats to lower-precision ones. Type conversion lets you do this explicitly. Type promotion happens automatically when you combine arrays of different types; NumPy picks a common type that can hold both. But these conversions aren't always safe—some lose precision, some overflow, and some fail entirely.

### Key Characteristics

- **Explicit vs. Implicit:** `astype()` forces a conversion; promotion happens automatically during operations.
- **Copy Guarantee:** `astype()` returns a new array (a copy), never a view.
- **Casting Safety Modes:** `'safe'`, `'same_kind'`, `'unsafe'`, `'equiv'`, `'no'` control what conversions are permitted.
- **Value-Based Promotion (Legacy):** Before NumPy 2.0, promotion depended on the values of scalars; NEP 50 changed this to be value-independent.
- **Overflow and Underflow:** Integer overflow wraps silently; float underflow flushes to zero (or denormal numbers).

### Prerequisites

- NumPy installed (`pip install numpy`).
- Understanding of NumPy dtypes (int, float, bool, complex, etc.).
- Familiarity with the `import numpy as np` convention.
- Conceptual understanding of binary representation and precision.

### Related Programming Areas

- **Numerical Computing:** Ensuring precision and avoiding overflow in scientific calculations.
- **Machine Learning:** Converting between float32 and float16 for memory efficiency.
- **Image Processing:** Converting between uint8 pixel values and float32 normalized values.
- **Data Engineering:** Loading data with mixed types and coercing to a common dtype.

### Core Concepts / Features

The following core concepts are explored in detail: (1) `astype()`, (2) Integer-to-float conversion, (3) Float-to-integer conversion, (4) Boolean conversion, (5) Casting safety modes, (6) Precision implications, (7) Overflow, (8) Underflow, and (9) Type promotion and coercion.

---

## Core Concept 1: astype()

### Definitions

**Core Definition:** `astype()` is the primary method for explicitly converting an array from one dtype to another.

**Technical Definition:** `ndarray.astype(dtype, order='K', casting='unsafe', subok=True, copy=True)` returns a copy of the array cast to the specified type. By default, `astype` always returns a newly allocated array. If `copy=False` and the dtype, order, and subok requirements are already satisfied, the input array is returned instead of a copy.

**Beginner-Friendly Explanation:** `astype()` is the "change the type" button. You tell it what type you want, and it converts the array. It always makes a new copy by default, so the original array is untouched.

### Purposes

- To explicitly convert an array's dtype when you need a specific type for an operation.
- To reduce memory usage by converting to a smaller dtype (e.g., float64 → float32).
- To increase precision by converting to a larger dtype (e.g., int32 → int64).
- To ensure compatibility with functions or libraries that require a specific dtype.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
ndarray.astype(dtype, order='K', casting='unsafe', subok=True, copy=True)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `dtype` | Target data type (string or dtype object). |
| `order` | Memory layout: `'C'`, `'F'`, `'A'`, `'K'` (default `'K'`). |
| `casting` | Casting safety mode: `'no'`, `'equiv'`, `'safe'`, `'same_kind'`, `'unsafe'` (default `'unsafe'`). |
| `subok` | If `True` (default), subclasses are passed through. |
| `copy` | If `False`, returns the input array if requirements are satisfied. |

**Syntax Rules:**

- `astype()` always returns a new array unless `copy=False` and all conditions are met.
- The default casting mode is `'unsafe'` for backward compatibility.
- Casting from complex to float or int raises a `ComplexWarning`; use `.real.astype(t)` to avoid it.

**Constraints and Limitations:**

- `astype()` cannot change the shape of the array; use `reshape()` for that.
- Converting from a larger to a smaller dtype can silently lose data (e.g., float64 → float32).
- String dtype casting requires the target string length to be sufficient in `'safe'` mode.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Basic astype() Conversions**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create an array of integers.
a = np.array([1, 2, 3, 4])
print("Original:", a, "| dtype:", a.dtype)
print()

# Step 3: Convert to float64.
b = a.astype(np.float64)
print("astype(float64):", b, "| dtype:", b.dtype)
print()

# Step 4: Convert float array to int (truncation).
c = np.array([1.7, 2.3, 3.9])
d = c.astype(int)
print("astype(int) of [1.7, 2.3, 3.9]:", d)
print("  (truncated toward zero)")
print()

# Step 5: Convert with copy=False (returns same array if possible).
e = a.astype(np.int64, copy=False)
print("astype(int64, copy=False):", e)
print("e is a:", e is a)
```

**Expected Output:**

```
Original: [1 2 3 4] | dtype: int64

astype(float64): [1. 2. 3. 4.] | dtype: float64

astype(int) of [1.7, 2.3, 3.9]: [1 2 3]
  (truncated toward zero)

astype(int64, copy=False): [1 2 3 4]
e is a: True
```

**Why This Result Occurs:** `astype(np.float64)` converts each integer to a float. `astype(int)` truncates the fractional part toward zero. `astype(np.int64, copy=False)` returns the same array because the input is already `int64`, satisfying the dtype requirement.

### Real-World Cases

- **Memory Reduction:** Converting a float64 array to float32 to halve memory usage in a neural network.
- **Image Processing:** Converting uint8 pixel values to float32 for normalization.
- **Data Serialization:** Converting arrays to a specific dtype before saving to disk.

### References

- numpy.ndarray.astype – https://numpy.org/doc/2.2/reference/generated/numpy.ndarray.astype.html
- numpy.astype – https://numpy.org/doc/stable/reference/generated/numpy.astype.html

---

## Core Concept 2: Converting Integer to Float

### Definitions

**Core Definition:** Converting an integer array to a floating-point dtype upcasts each integer value to its floating-point representation.

**Technical Definition:** Integer-to-float conversion is a "safe" cast when the float dtype has enough mantissa bits to exactly represent all integers in the range. For example, `int32` → `float64` is safe because float64 has 52 mantissa bits (sufficient for 32-bit integers), but `int32` → `float32` is not safe because float32 has only 23 mantissa bits. Large integers may be rounded.

**Beginner-Friendly Explanation:** Converting integers to floats is usually straightforward, but if the integer is very large, the float may not be able to store it exactly. This is because floats have limited precision—they can only represent a certain number of significant digits.

### Purposes

- To enable floating-point arithmetic on integer data (e.g., division).
- To prepare integer data for algorithms that require float inputs.
- To normalize integer-valued data to a standard range.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
float_arr = int_arr.astype(np.float64)
float_arr = int_arr.astype(np.float32)
```

**Component Breakdown:**

| Conversion | Safe? | Notes |
|------------|-------|-------|
| `int8` → `float16` | No | float16 has 10 mantissa bits, int8 needs 8 (ok for 8-bit, but not all). |
| `int16` → `float32` | Yes | float32 has 23 mantissa bits ≥ 16. |
| `int32` → `float64` | Yes | float64 has 52 mantissa bits ≥ 32. |
| `int64` → `float64` | No | float64 has 52 mantissa bits < 64. |

**Syntax Rules:**

- Use `astype(np.float64)` for maximum precision.
- Use `np.can_cast(from, to, casting='safe')` to check if a conversion is safe.
- Large integers beyond 2⁵³ may lose precision when converted to float64.

**Constraints and Limitations:**

- `int64` → `float64` can lose precision for values above 2⁵³.
- `int32` → `float32` can lose precision for values above 2²³.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Integer to Float Precision**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create a large integer array.
large_int = np.array([2**24 + 1, 2**53 + 1], dtype=np.int64)
print("Large integers:", large_int)
print()

# Step 3: Convert to float32 (loses precision for 2^24+1).
f32 = large_int.astype(np.float32)
print("astype(float32):", f32)
print("  (2^24+1 is not exactly representable in float32)")
print()

# Step 4: Convert to float64 (loses precision for 2^53+1).
f64 = large_int.astype(np.float64)
print("astype(float64):", f64)
print("  (2^53+1 is not exactly representable in float64)")
print()

# Step 5: Check safe casting.
print("can_cast(int64, float64, 'safe'):", np.can_cast(np.int64, np.float64, 'safe'))
```

**Expected Output:**

```
Large integers: [16777217, 9007199254740993]

astype(float32): [1.6777216e+07 9.0071993e+15]
  (2^24+1 is not exactly representable in float32)

astype(float64): [1.6777217e+07 9.0071993e+15]
  (2^53+1 is not exactly representable in float64)

can_cast(int64, float64, 'safe'): False
```

**Why This Result Occurs:** `2²⁴ + 1 = 16777217` requires 25 bits of precision, but float32 has only 24 bits of significand (23 explicit + 1 implicit), so it rounds to 16777216. `2⁵³ + 1` requires 54 bits, but float64 has only 53 bits of significand. `np.can_cast` confirms that `int64` → `float64` is not a safe cast.

### Real-World Cases

- **Normalization:** Converting integer pixel values (0–255) to float32 (0.0–1.0) for neural network input.
- **Scientific Computing:** Converting integer measurements to floats for statistical analysis.
- **Financial Calculations:** Converting integer cents to float dollars.

### References

- Data type promotion in NumPy – https://numpy.org/doc/2.1/reference/arrays.promotion.html
- numpy.can_cast – https://numpy.org/doc/2.0/reference/generated/numpy.can_cast.html

---

## Core Concept 3: Converting Float to Integer

### Definitions

**Core Definition:** Converting a float array to an integer dtype truncates the fractional part toward zero.

**Technical Definition:** When a finite floating value is converted to an integer type, the fractional part is discarded (truncated toward zero). NaN and infinity values produce platform-dependent results (often the minimum integer value or zero), and no error is raised by default.

**Beginner-Friendly Explanation:** Converting floats to integers simply drops everything after the decimal point. So 2.9 becomes 2, and -2.9 becomes -2. But be careful with NaN and infinity—they don't have meaningful integer representations.

### Purposes

- To convert floating-point measurements to discrete integer values.
- To prepare float data for algorithms that require integer indices.
- To store float data as integers when fractional precision is not needed.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
int_arr = float_arr.astype(np.int32)
```

**Component Breakdown:**

| Input | Result | Notes |
|-------|--------|-------|
| `2.9` | `2` | Truncated toward zero. |
| `-2.9` | `-2` | Truncated toward zero. |
| `NaN` | Platform-dependent | Often `np.iinfo(dtype).min` or `0`. |
| `Inf` | Platform-dependent | Often `np.iinfo(dtype).min` or `0`. |

**Syntax Rules:**

- Truncation, not rounding, is the default behavior.
- Use `np.round()` before `astype()` for round-to-nearest behavior.
- Use `np.nan_to_num()` to handle NaN and Inf before conversion.

**Constraints and Limitations:**

- NaN and Inf conversions are not well-defined and can produce unexpected results.
- The exact behavior for NaN/Inf depends on the platform and NumPy version.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Float to Integer Truncation and NaN Handling**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create a float array with fractional values.
f = np.array([1.7, 2.3, -3.9, 0.5])
print("Float array:", f)
print()

# Step 3: Convert to integer (truncation).
i = f.astype(np.int32)
print("astype(int32):", i)
print("  (truncated toward zero: 1, 2, -3, 0)")
print()

# Step 4: Demonstrate NaN and Inf conversion.
special = np.array([np.nan, np.inf, -np.inf])
print("Special values:", special)
special_int = special.astype(np.int32)
print("astype(int32):", special_int)
print("  (platform-dependent results)")
print()

# Step 5: Safe conversion using nan_to_num.
safe = np.nan_to_num(special, nan=0.0, posinf=2**31 - 1, neginf=-2**31)
print("After nan_to_num:", safe)
safe_int = safe.astype(np.int32)
print("Safe astype(int32):", safe_int)
```

**Expected Output:**

```
Float array: [ 1.7  2.3 -3.9  0.5]

astype(int32): [ 1  2 -3  0]
  (truncated toward zero: 1, 2, -3, 0)

Special values: [nan inf -inf]
astype(int32): [-2147483648 -2147483648 -2147483648]
  (platform-dependent results)

After nan_to_num: [ 0.00000000e+00  2.14748365e+09 -2.14748365e+09]
Safe astype(int32): [         0 2147483647 -2147483648]
```

**Why This Result Occurs:** Truncation drops the fractional part. NaN and Inf become `np.iinfo(np.int32).min` (-2147483648) on this platform. Using `np.nan_to_num()` first replaces NaN with 0 and Inf with the specified finite values, producing predictable results.

### Real-World Cases

- **Discretization:** Converting continuous measurements to integer bins.
- **Index Generation:** Converting float coordinates to integer pixel indices.
- **Data Storage:** Storing float measurements as scaled integers to save memory.

### References

- Mailman: Nice float -> integer conversion? – https://mail.python.org/archives/list/numpy-discussion@python.org/message/EMOCZLYKQN3CZP3T3ZXZRE5PVF6NL7ZI/
- numpy.nan_to_num – https://numpy.org/doc/stable/reference/generated/numpy.nan_to_num.html

---

## Core Concept 4: Boolean Conversion

### Definitions

**Core Definition:** Boolean conversion evaluates numeric or other values to logical flags, where zero maps to `False` and all non-zero values map to `True`.

**Technical Definition:** When converting to `bool_`, any non-zero value becomes `True`, and zero becomes `False`. When converting from `bool_` to numeric types, `True` becomes 1 and `False` becomes 0. For strings, non-empty strings are `True` (though string-to-bool casting has specific rules).

**Beginner-Friendly Explanation:** Converting to boolean is like asking "is this value zero or not?" Zero is false; everything else is true.

### Purposes

- To create masks for boolean indexing.
- To represent binary states compactly.
- To convert logical conditions to numeric 0/1 values.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
bool_arr = numeric_arr.astype(bool)
```

**Component Breakdown:**

| Input | Result |
|-------|--------|
| `0` | `False` |
| `0.0` | `False` |
| `1` (or any non-zero) | `True` |
| `-1` | `True` |
| `np.nan` | `True` (NaN is not zero) |

**Syntax Rules:**

- Zero values (integer 0, float 0.0) become `False`.
- All non-zero values become `True`.
- Converting `bool` back to numeric gives 1 for `True` and 0 for `False`.

**Constraints and Limitations:**

- `np.nan` is not zero, so it converts to `True`.
- String-to-bool conversion rules are more complex and version-dependent.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Boolean Conversion of Numeric Arrays**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create an integer array with zeros and non-zeros.
ints = np.array([0, 1, 2, -3, 0])
print("Integer array:", ints)

# Step 3: Convert to boolean.
bools = ints.astype(bool)
print("astype(bool):", bools)
print()

# Step 4: Create a float array with NaN.
floats = np.array([0.0, 1.5, np.nan, -0.0])
print("Float array:", floats)
bools_f = floats.astype(bool)
print("astype(bool):", bools_f)
print("  (NaN is True because it is not zero)")
print()

# Step 5: Convert boolean back to integer.
back_to_int = bools.astype(np.int32)
print("Back to int32:", back_to_int)
```

**Expected Output:**

```
Integer array: [ 0  1  2 -3  0]
astype(bool): [False  True  True  True False]

Float array: [ 0.  1.5 nan -0.]
astype(bool): [False  True  True False]
  (NaN is True because it is not zero)

Back to int32: [0 1 1 1 0]
```

**Why This Result Occurs:** Zero maps to `False`; all other values (including NaN) map to `True`. Converting back to integer gives 1 for `True` and 0 for `False`.

### Real-World Cases

- **Masking:** Creating boolean masks from threshold conditions.
- **Counting:** Summing boolean arrays to count `True` values.
- **Logic Operations:** Combining conditions with `&`, `|`, `~`.

### References

- Numpy-discussion: nan_to_num and bool arrays – https://mail.python.org/pipermail/numpy-discussion/2009-November/046314.html

---

## Core Concept 5: Casting Safety Modes

### Definitions

**Core Definition:** Casting safety modes are rules that determine whether a type conversion is permitted, based on how much value preservation is guaranteed.

**Technical Definition:** The `casting` parameter accepts one of five values: `'no'` (no casting), `'equiv'` (only byte-order changes), `'safe'` (only value-preserving casts), `'same_kind'` (safe casts or casts within a kind), and `'unsafe'` (any conversion). These are implemented by `numpy.can_cast()` and used by `astype()` and ufuncs.

**Beginner-Friendly Explanation:** Casting safety modes are like a safety rating for type conversions. "Safe" means no data will be lost. "Same_kind" allows conversions within the same family (like float64 to float32). "Unsafe" means anything goes—potentially losing data.

### Purposes

- To prevent silent data corruption from unsafe conversions.
- To document the intent of a conversion in code.
- To enforce strict type rules in numerical pipelines.
- To catch potential bugs at runtime rather than producing wrong results.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Checking if a cast is allowed
np.can_cast(from_dtype, to_dtype, casting='safe')

# Using casting in astype
arr.astype(target_dtype, casting='safe')
```

**Component Breakdown:**

| Mode | Description | Example Allowed | Example Denied |
|------|-------------|-----------------|----------------|
| `'no'` | No casting at all | Same dtype | Any different dtype |
| `'equiv'` | Byte-order changes only | `<f8` → `>f8` | `f8` → `f4` |
| `'safe'` | Value-preserving casts | `i4` → `i8` | `f8` → `i4` |
| `'same_kind'` | Safe or same-kind casts | `f8` → `f4` | `f8` → `i4` |
| `'unsafe'` | Any conversion | `f8` → `i1` | None |

**Syntax Rules:**

- The default for `astype()` is `'unsafe'` for backward compatibility.
- The default for `can_cast()` is `'safe'`.
- Ufuncs typically use `'same_kind'`.

**Constraints and Limitations:**

- `'safe'` mode may reject conversions that are actually fine for specific values (value-independent).
- `'same_kind'` still allows lossy conversions within the same kind (e.g., float64 → float16).

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Testing Casting Safety Modes**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Check safe casting between various types.
print("np.can_cast('i4', 'i8', 'safe'):", np.can_cast('i4', 'i8', 'safe'))
print("np.can_cast('f8', 'i4', 'safe'):", np.can_cast('f8', 'i4', 'safe'))
print("np.can_cast('f8', 'f4', 'same_kind'):", np.can_cast('f8', 'f4', 'same_kind'))
print("np.can_cast('f8', 'i4', 'unsafe'):", np.can_cast('f8', 'i4', 'unsafe'))
print()

# Step 3: Demonstrate astype with different casting modes.
a = np.array([1.5, 2.7, 3.1])

try:
    a.astype(np.int32, casting='safe')
except TypeError as e:
    print("astype(int32, casting='safe') raises:", e)

result = a.astype(np.int32, casting='unsafe')
print("astype(int32, casting='unsafe'):", result)
```

**Expected Output:**

```
np.can_cast('i4', 'i8', 'safe'): True
np.can_cast('f8', 'i4', 'safe'): False
np.can_cast('f8', 'f4', 'same_kind'): True
np.can_cast('f8', 'i4', 'unsafe'): True

astype(int32, casting='safe') raises: Cannot cast array data from dtype('float64') to dtype('int32') according to the rule 'safe'
astype(int32, casting='unsafe'): [1 2 3]
```

**Why This Result Occurs:** `int32` → `int64` is safe (no value loss). `float64` → `int32` is not safe (fractional parts are lost). `float64` → `float32` is allowed under `'same_kind'` because they are the same kind. Using `casting='safe'` in `astype` raises an error, while `'unsafe'` permits the conversion.

### Real-World Cases

- **Data Validation:** Using `casting='safe'` to ensure that a conversion will not lose data.
- **Scientific Pipelines:** Enforcing `'same_kind'` to prevent accidental integer/float mixups.
- **Performance Optimization:** Using `'unsafe'` when you know the conversion is safe for your data.

### References

- numpy.can_cast – https://numpy.org/doc/2.0/reference/generated/numpy.can_cast.html
- Data type routines – https://numpy.org/doc/1.26/reference/routines.dtype.html

---

## Core Concept 6: Precision Implications

### Definitions

**Core Definition:** Precision implications are the effects on numerical accuracy and range when converting between floating-point dtypes of different bit widths.

**Technical Definition:** `float16` has 10 mantissa bits (≈3.3 decimal digits) and 5 exponent bits (range ±6.5×10⁴). `float32` has 23 mantissa bits (≈7.2 digits) and 8 exponent bits (range ±3.4×10³⁸). `float64` has 52 mantissa bits (≈15.9 digits) and 11 exponent bits (range ±1.8×10³⁰⁸). Converting from a larger to a smaller float dtype loses precision and reduces range.

**Beginner-Friendly Explanation:** Not all floats are created equal. `float16` is very imprecise and has a small range. `float32` is a good middle ground. `float64` is the most precise. When you convert from float64 to float32, you lose about 8 decimal digits of precision.

### Purposes

- To understand the trade-off between memory usage and numerical accuracy.
- To choose the appropriate float dtype for a given application.
- To predict when precision loss will cause problems in computations.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Downcasting loses precision
f32 = f64_array.astype(np.float32)

# Upcasting preserves precision
f64 = f32_array.astype(np.float64)
```

**Component Breakdown:**

| Dtype | Mantissa Bits | Decimal Digits | Exponent Bits | Range |
|-------|--------------|----------------|---------------|-------|
| `float16` | 10 | ~3.3 | 5 | ±6.5×10⁴ |
| `float32` | 23 | ~7.2 | 8 | ±3.4×10³⁸ |
| `float64` | 52 | ~15.9 | 11 | ±1.8×10³⁰⁸ |

**Syntax Rules:**

- Downcasting (float64 → float32) always loses precision.
- Upcasting (float32 → float64) preserves the existing precision but does not add precision that was never there.
- Use `np.finfo(dtype).eps` to check the machine epsilon for a dtype.

**Constraints and Limitations:**

- Repeated downcasting can accumulate errors.
- Operations in lower precision are faster but less accurate.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Precision Loss in Downcasting**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create a float64 array with a value needing high precision.
a = np.array([1.0 / 3.0], dtype=np.float64)
print("float64:", a)
print()

# Step 3: Downcast to float32.
b = a.astype(np.float32)
print("float32:", b)
print("Difference:", abs(a - b.astype(np.float64)))
print()

# Step 4: Downcast to float16.
c = a.astype(np.float16)
print("float16:", c)
print("Difference:", abs(a - c.astype(np.float64)))
print()

# Step 5: Show machine epsilon for each dtype.
for dtype in [np.float16, np.float32, np.float64]:
    print(f"{dtype.__name__} eps: {np.finfo(dtype).eps:.6e}")
```

**Expected Output:**

```
float64: [0.33333333]

float32: [0.33333334]
Difference: [5.55111512e-09]

float16: [0.3333]
Difference: [3.33333333e-05]

float16 eps: 9.765625e-04
float32 eps: 1.192093e-07
float64 eps: 2.220446e-16
```

**Why This Result Occurs:** `1/3` is not exactly representable in binary. `float64` stores the closest approximation with 52 mantissa bits. `float32` stores a less precise approximation, and `float16` is even less precise. The machine epsilon decreases with increasing precision.

### Real-World Cases

- **Neural Networks:** Using float16 for inference reduces memory and speeds up computation with minimal accuracy loss.
- **Scientific Simulations:** Using float64 for long-running simulations to avoid accumulated rounding errors.
- **Graphics:** Using float32 for GPU computations where precision requirements are lower.

### References

- Numerical Stability in Python – https://pages.llf-paris.fr/~gwisniewski/assets/2025/lab_pro_skills/pro_skills_lab5/Numerical_Stability_Lama_YASSIN_Code_and_Report.pdf

---

## Core Concept 7: Overflow

### Definitions

**Core Definition:** Overflow occurs when a value exceeds the maximum representable value of a dtype, causing it to wrap around (for integers) or become infinity (for floats).

**Technical Definition:** For fixed-width integer types, arithmetic that exceeds the range wraps around modulo 2ⁿ. For floating-point types, values exceeding the maximum become `inf`. NumPy does not raise an error by default for overflow; the behavior is silent.

**Beginner-Friendly Explanation:** If you try to put a number that's too big into a small container, it overflows. For integers, it wraps around like a car odometer. For floats, it becomes infinity.

### Purposes

- To understand when numerical results may be incorrect due to range limitations.
- To choose dtypes with sufficient range for your data.
- To detect and handle overflow conditions proactively.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Integer overflow wraps around
np.uint8(255) + 1  # -> 0

# Float overflow becomes inf
np.float32(3.4e38) * 10  # -> inf
```

**Component Breakdown:**

| Dtype | Max Value | Overflow Behavior |
|-------|-----------|-------------------|
| `int8` | 127 | Wraps to -128 |
| `uint8` | 255 | Wraps to 0 |
| `int32` | 2,147,483,647 | Wraps to -2,147,483,648 |
| `float32` | ~3.4×10³⁸ | Becomes `inf` |
| `float64` | ~1.8×10³⁰⁸ | Becomes `inf` |

**Syntax Rules:**

- Integer overflow wraps silently; no exception is raised.
- Float overflow produces `inf`; no exception is raised.
- Use `np.seterr(over='raise')` to raise an error on float overflow.

**Constraints and Limitations:**

- Integer overflow is undefined behavior in C for signed integers but NumPy guarantees wraparound.
- Overflow detection is not automatic for integer operations.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Integer and Float Overflow**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Demonstrate integer overflow.
a = np.array([255], dtype=np.uint8)
print("uint8 255 + 1:", a + 1)
print("  (wraps to 0)")
print()

b = np.array([127], dtype=np.int8)
print("int8 127 + 1:", b + 1)
print("  (wraps to -128)")
print()

# Step 3: Demonstrate float overflow.
c = np.array([3.4e38], dtype=np.float32)
print("float32 3.4e38 * 10:", c * 10)
print("  (becomes inf)")
print()

# Step 4: Enable overflow warnings for floats.
np.seterr(over='warn')
d = np.array([1e308], dtype=np.float64)
result = d * 10
print("float64 1e308 * 10:", result)
```

**Expected Output:**

```
uint8 255 + 1: [0]
  (wraps to 0)

int8 127 + 1: [-128]
  (wraps to -128)

float32 3.4e38 * 10: [inf]
  (becomes inf)

float64 1e308 * 10: [inf]
```

**Why This Result Occurs:** `uint8` can hold 0–255; 255 + 1 wraps to 0. `int8` holds −128 to 127; 127 + 1 wraps to −128. Float overflow produces `inf`. Enabling `seterr(over='warn')` would issue a warning for float overflow.

### Real-World Cases

- **Counting:** Using `int32` for counters that might exceed 2 billion.
- **Financial Calculations:** Using `int64` for monetary amounts to avoid overflow.
- **Deep Learning:** Using float32 can overflow during accumulation; float64 is safer for gradients.

### References

- Numpy-discussion: Overflow and wraparound – https://mail.python.org/pipermail/numpy-discussion/2011-March/054999.html

---

## Core Concept 8: Underflow

### Definitions

**Core Definition:** Underflow occurs when a floating-point result is smaller in magnitude than the smallest representable positive value, causing it to be flushed to zero (or represented as a denormal number with reduced precision).

**Technical Definition:** For IEEE 754 floats, the smallest normalized positive number is 2⁻¹²⁶ for float32 and 2⁻¹⁰²² for float64. Values smaller than this may be represented as denormal (subnormal) numbers, which have reduced precision. If they become too small even for denormals, they flush to zero.

**Beginner-Friendly Explanation:** Underflow happens when a number gets so close to zero that the computer can't represent it anymore, so it just becomes zero. This can happen when multiplying many small numbers together.

### Purposes

- To understand when tiny values might be lost in computations.
- To detect underflow conditions in numerical simulations.
- To choose appropriate dtypes for applications involving very small numbers.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Underflow flush to zero
np.float32(1e-45) / 10  # -> 0.0

# Use seterr to control underflow handling
np.seterr(under='warn')
```

**Component Breakdown:**

| Dtype | Smallest Normalized | Smallest Denormal |
|-------|---------------------|-------------------|
| `float16` | 6.1×10⁻⁵ | 6.0×10⁻⁸ |
| `float32` | 1.2×10⁻³⁸ | 1.4×10⁻⁴⁵ |
| `float64` | 2.2×10⁻³⁰⁸ | 4.9×10⁻³²⁴ |

**Syntax Rules:**

- Underflow does not raise an error by default; it silently flushes to zero.
- Use `np.seterr(under='raise')` to raise an exception on underflow.
- Denormal numbers have reduced precision (fewer significant bits).

**Constraints and Limitations:**

- Underflow can cause gradients in neural networks to vanish.
- Repeated underflow can accumulate errors.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Underflow in Floating-Point Arithmetic**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create a tiny float64 value.
tiny = np.float64(1e-308)
print("Tiny value:", tiny)
print()

# Step 3: Multiply by a small factor repeatedly.
for i in range(3):
    tiny = tiny / 1e10
    print(f"After division {i+1}: {tiny}")
print()

# Step 4: Demonstrate float32 underflow.
tiny32 = np.float32(1e-38)
print("float32 tiny:", tiny32)
tiny32 = tiny32 / 1e10
print("After division:", tiny32)
print("  (flushed to zero)")
print()

# Step 5: Check with seterr.
np.seterr(under='warn')
result = np.float32(1e-40) * np.float32(1e-10)
print("Underflow warning result:", result)
```

**Expected Output:**

```
Tiny value: 1e-308

After division 1: 1e-318
After division 2: 0.0
After division 3: 0.0

float32 tiny: 1e-38
After division: 0.0
  (flushed to zero)

Underflow warning result: 0.0
```

**Why This Result Occurs:** `1e-308` is near the limit of float64 precision. Dividing by 1e10 pushes it below the smallest representable value, so it flushes to zero. float32 flushes to zero much earlier. Underflow warnings can be enabled but are often ignored in practice.

### Real-World Cases

- **Deep Learning:** Gradients can underflow to zero in float16, causing training to stall.
- **Probability Calculations:** Multiplying many small probabilities can underflow; use log-probabilities instead.
- **Physics Simulations:** Very small quantities may underflow; use appropriate scaling.

### References

- Stack Overflow: At what point should I worry about underflow in numpy values? – https://stackoverflow.com/questions/34925396
- NumPy: How numpy handles numerical exceptions – https://numpy.org/doc/stable/reference/generated/numpy.seterr.html

---

## Core Concept 9: Type Promotion & Coercion

### Definitions

**Core Definition:** Type promotion is NumPy's automatic determination of a common dtype when combining arrays of different types; type coercion is the implicit conversion of one or both operands to that common dtype.

**Technical Definition:** NumPy uses `np.result_type()` and `np.promote_types()` to determine the result dtype. Promotion follows a hierarchy: unsigned integers → signed integers → floats → complex. The result dtype is always of equal or higher kind and precision. NEP 50 (NumPy 2.0+) changed Python scalar behavior: scalars are "weakly typed" and adopt the array's dtype.

**Beginner-Friendly Explanation:** When you add an integer array to a float array, NumPy has to pick one type for the result. It always picks the "bigger" or more precise type. This is called promotion. NEP 50 made this more predictable by making Python scalars adapt to the array's type.

### Purposes

- To understand the result dtype of mixed-type operations.
- To predict and control type promotion in numerical pipelines.
- To avoid unexpected precision loss or overflow from type promotion.
- To use `result_type()` and `promote_types()` for explicit dtype planning.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Determine result type of arrays/dtypes
result_dtype = np.result_type(arr1, arr2)

# Promote two dtypes
common_dtype = np.promote_types(dtype1, dtype2)
```

**Component Breakdown:**

| Function | Description |
|----------|-------------|
| `np.result_type(*arrays_and_dtypes)` | Returns the dtype that results from applying NumPy's promotion rules. |
| `np.promote_types(type1, type2)` | Returns the smallest dtype to which both types can be safely cast. |

**Syntax Rules:**

- Promotion hierarchy: bool → uint → int → float → complex.
- The result always has precision ≥ both inputs.
- Python scalars (int, float) are "weakly typed" since NumPy 2.0; they adopt the array's dtype.

**Constraints and Limitations:**

- `int64` + `uint64` promotes to `float64` (no higher integer type available).
- `int64` + `float16` promotes to `float64` (integer precision forces higher float precision).
- Before NEP 50, scalar values could influence promotion; this is no longer the case.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Type Promotion in Practice**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Promote int32 and float64.
result = np.result_type(np.int32, np.float64)
print("result_type(int32, float64):", result)
print()

# Step 3: Promote int8 and int64.
result2 = np.promote_types(np.int8, np.int64)
print("promote_types(int8, int64):", result2)
print()

# Step 4: Demonstrate NEP 50 scalar behavior.
arr = np.array([1, 2, 3], dtype=np.uint8)
print("uint8 array + 1:", (arr + 1).dtype)
print("  (Python int adopts uint8 dtype)")
print()

arr_f32 = np.array([1.0, 2.0, 3.0], dtype=np.float32)
print("float32 array + 2.0:", (arr_f32 + 2.0).dtype)
print("  (Python float adopts float32 dtype)")
print()

# Step 5: Show int64 + uint64 promotion.
result3 = np.result_type(np.int64, np.uint64)
print("result_type(int64, uint64):", result3)
print("  (promotes to float64, since no higher integer type exists)")
```

**Expected Output:**

```
result_type(int32, float64): float64

promote_types(int8, int64): int64

uint8 array + 1: uint8
  (Python int adopts uint8 dtype)

float32 array + 2.0: float32
  (Python float adopts float32 dtype)

result_type(int64, uint64): float64
  (promotes to float64, since no higher integer type exists)
```

**Why This Result Occurs:** `int32` and `float64` promote to `float64` because float is a higher kind. `int8` and `int64` promote to `int64` (the larger integer type). Under NEP 50, Python scalars adopt the array's dtype, so adding `1` to a `uint8` array keeps the result `uint8`. `int64` and `uint64` cannot promote to a higher integer type, so they promote to `float64`.

### Real-World Cases

- **Mixed-Type Data:** Combining integer IDs with float measurements in a computation.
- **Neural Networks:** Adding a Python float learning rate to a float32 array keeps the result float32.
- **Data Cleaning:** Understanding why an operation between int and float arrays produces a float result.

### References

- Data type promotion in NumPy – https://numpy.org/doc/2.1/reference/arrays.promotion.html
- NEP 50 — Promotion rules for Python scalars – https://numpy.org/neps/nep-0050-scalar-promotion.html
- numpy.result_type – https://numpy.org/doc/stable/reference/generated/numpy.result_type.html
- numpy.promote_types – https://numpy.org/doc/stable/reference/generated/numpy.promote_types.html
- Data type routines – https://numpy.org/doc/1.26/reference/routines.dtype.html

---

## Summary of Deprecated and Version-Specific Features

| Feature | Status | Recommendation |
|---------|--------|----------------|
| Value-based promotion (pre-NEP 50) | Removed in NumPy 2.0 | Use NEP 50 rules; Python scalars are "weakly typed" |
| `np.find_common_type` | Deprecated | Use `np.result_type` or `np.promote_types` |
| `astype` with `casting='safe'` on strings | Requires sufficient string length since NumPy 1.9 | Ensure string dtype is long enough |
| `np.can_cast` with Python scalars | Removed in NumPy 2.0 | Use dtypes or NumPy scalars explicitly |

---

## Consolidated Reference List

- numpy.ndarray.astype – https://numpy.org/doc/2.2/reference/generated/numpy.ndarray.astype.html
- numpy.can_cast – https://numpy.org/doc/2.0/reference/generated/numpy.can_cast.html
- Data type routines – https://numpy.org/doc/1.26/reference/routines.dtype.html
- Data type promotion in NumPy – https://numpy.org/doc/2.1/reference/arrays.promotion.html
- NEP 50 — Promotion rules for Python scalars – https://numpy.org/neps/nep-0050-scalar-promotion.html
- numpy.result_type – https://numpy.org/doc/stable/reference/generated/numpy.result_type.html
- numpy.promote_types – https://numpy.org/doc/stable/reference/generated/numpy.promote_types.html
- Mailman: Nice float -> integer conversion? – https://mail.python.org/archives/list/numpy-discussion@python.org/message/EMOCZLYKQN3CZP3T3ZXZRE5PVF6NL7ZI/
- Numerical Stability in Python – https://pages.llf-paris.fr/~gwisniewski/assets/2025/lab_pro_skills/pro_skills_lab5/Numerical_Stability_Lama_YASSIN_Code_and_Report.pdf
- Stack Overflow: At what point should I worry about underflow in numpy values? – https://stackoverflow.com/questions/34925396
- Numpy-discussion: Overflow and wraparound – https://mail.python.org/pipermail/numpy-discussion/2011-March/054999.html