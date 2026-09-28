# Choosing Appropriate dtypes in NumPy: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Choosing an appropriate dtype in NumPy is the process of selecting the data type for an array that best balances memory consumption, numerical precision, computational speed, and range requirements for a given task.

**Technical Definition:** A dtype (data type object) is an instance of `numpy.dtype` that specifies the type of data (integer, float, complex, boolean, string, object, etc.), its size in bytes, byte order, and—for structured types—the names and types of fields. Every element in a NumPy array shares the same dtype, and this choice directly determines the array's memory footprint, the range of representable values, the precision of floating-point operations, and the efficiency of hardware-level vectorization.

**Beginner-Friendly Explanation:** A dtype is like a label on a box that says what's inside and how much space each item takes. If you choose a smaller box (like `float32`), you save shelf space but might not fit very precise numbers. If you choose a larger box (like `float64`), you have plenty of room but use more shelf space. The goal is to pick the right box for your data—not too big, not too small.

### Key Characteristics

- **Memory Footprint:** `float64` uses 8 bytes per element; `float32` uses 4 bytes; `int8` uses 1 byte. Halving the bit-width halves the memory requirement.
- **Numerical Precision:** `float64` provides ~15–16 decimal digits of precision; `float32` provides ~7; `float16` provides ~3. Lower precision increases the risk of rounding errors and catastrophic cancellation.
- **Range:** Integer types have fixed ranges (e.g., `int8`: −128 to 127; `int32`: ±2.1×10⁹; `int64`: ±9.2×10¹⁸). Floating-point types have exponent-dependent ranges.
- **Hardware Alignment:** Modern CPUs and GPUs are optimized for specific bit-widths (e.g., 32-bit and 16-bit operations on GPUs). Aligning dtypes with native hardware register sizes improves throughput.
- **Type Promotion:** When arrays of different dtypes are combined, NumPy promotes to a common dtype that can represent both, following NEP 50 rules.

### Prerequisites

- NumPy installed (`pip install numpy`).
- Understanding of NumPy dtypes (int, float, complex, bool, etc.).
- Familiarity with the `import numpy as np` convention.
- Conceptual understanding of computer memory (bytes, bit-widths) and binary representation.

### Related Programming Areas

- **Machine Learning:** Selecting `float32` or `float16` for GPU training and inference to reduce memory and improve throughput.
- **Image Processing:** Using `uint8` for pixel values (0–255) and `float32` for normalized image data.
- **Scientific Computing:** Using `float64` for high-precision simulations and numerical integration.
- **Data Engineering:** Using `int32` or `int64` for identifiers, counters, and timestamps.

### Core Concepts / Features

The following core concepts are explored in detail: (1) Memory considerations, (2) Numerical precision, (3) Computational efficiency, (4) Integer range, and (5) Floating-point limitations.

---

## Core Concept 1: Memory Considerations

### Definitions

**Core Definition:** Memory considerations in dtype selection refer to the impact of dtype bit-width on the total RAM consumed by an array.

**Technical Definition:** The memory footprint of an array is calculated as `nbytes = itemsize × size`, where `itemsize` is the number of bytes per element (determined by dtype) and `size` is the total number of elements. Switching from `float64` to `float32` halves `itemsize` from 8 to 4 bytes, halving `nbytes`. This matters most for large arrays that approach RAM limits.

**Beginner-Friendly Explanation:** If you have a million numbers and each one takes 8 bytes, you need 8 MB of RAM. If you switch to 4 bytes per number, you only need 4 MB. For small arrays, this doesn't matter; for large ones, it's the difference between fitting in memory and crashing.

### Purposes

- To reduce RAM usage for large arrays, allowing bigger datasets to fit in memory.
- To lower memory bandwidth requirements, improving cache utilization.
- To enable larger batch sizes in machine learning training.
- To reduce data transfer costs when loading or saving arrays.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Check memory footprint
arr.nbytes          # total bytes
arr.itemsize        # bytes per element
arr.size            # number of elements

# Create arrays with specific dtypes
arr_f32 = np.zeros((1000, 768), dtype=np.float32)  # 3 MB
arr_f64 = np.zeros((1000, 768), dtype=np.float64)  # 6 MB
```

**Component Breakdown:**

| Dtype | Itemsize (bytes) | Memory for 1M Elements |
|-------|-----------------|----------------------|
| `float64` | 8 | 8 MB |
| `float32` | 4 | 4 MB |
| `float16` | 2 | 2 MB |
| `int64` | 8 | 8 MB |
| `int32` | 4 | 4 MB |
| `int8` | 1 | 1 MB |
| `bool` | 1 | 1 MB |

**Syntax Rules:**

- `nbytes = itemsize × size`.
- Downcasting (e.g., `float64` → `float32`) always copies the array because the memory layout changes.
- Use `astype()` to change dtype; note that this creates a new array.

**Constraints and Limitations:**

- Downcasting can lose precision or range; verify that the target dtype can represent all values.
- Memory savings from downcasting are only realized when the original array can be freed.
- Some operations may temporarily upcast to a larger dtype, negating memory savings.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Measuring Memory Impact of dtype Choice**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create a large array with float64 (default).
arr_f64 = np.zeros((10_000, 10_000), dtype=np.float64)
print("float64 array:")
print(f"  Shape: {arr_f64.shape}")
print(f"  itemsize: {arr_f64.itemsize} bytes")
print(f"  nbytes: {arr_f64.nbytes / 1e6:.1f} MB")
print()

# Step 3: Create the same array with float32.
arr_f32 = np.zeros((10_000, 10_000), dtype=np.float32)
print("float32 array:")
print(f"  itemsize: {arr_f32.itemsize} bytes")
print(f"  nbytes: {arr_f32.nbytes / 1e6:.1f} MB")
print()

# Step 4: Compare memory savings.
savings = (arr_f64.nbytes - arr_f32.nbytes) / arr_f64.nbytes * 100
print(f"Memory savings: {savings:.0f}%")
```

**Expected Output:**

```
float64 array:
  Shape: (10000, 10000)
  itemsize: 8 bytes
  nbytes: 800.0 MB

float32 array:
  itemsize: 4 bytes
  nbytes: 400.0 MB

Memory savings: 50%
```

**Why This Result Occurs:** The `float64` array stores each of its 100 million elements in 8 bytes, totaling 800 MB. The `float32` array uses 4 bytes per element, totaling 400 MB. Halving the itemsize halves the total memory footprint.

### Real-World Cases

- **Deep Learning:** A model with 100 million parameters stored as `float64` requires 800 MB; as `float32`, only 400 MB—enabling larger batch sizes on the same GPU.
- **Image Processing:** A batch of 10,000 RGB images (256×256×3) as `float64` uses ~15 GB; as `uint8`, only ~1.9 GB.
- **Time Series:** A 10-year minute-level financial dataset as `float64` uses ~4.2 GB; as `float32`, ~2.1 GB.

### References

- NumPy dtypes and Precision — Memory, Speed, and ML – https://github.com/ReddyBytes/Python-DSA-API-Mastery/blob/main/01_Python_Mastery/22_numpy_for_ai/01_dtype_and_precision.md

---

## Core Concept 2: Numerical Precision

### Definitions

**Core Definition:** Numerical precision refers to the number of significant digits a floating-point dtype can represent, and the resulting accuracy of computations.

**Technical Definition:** `float64` has 52 mantissa bits (≈15.9 decimal digits), `float32` has 23 mantissa bits (≈7.2 digits), and `float16` has 10 mantissa bits (≈3.3 digits). Precision loss occurs when values cannot be represented exactly, leading to rounding errors that accumulate across operations. Catastrophic cancellation occurs when subtracting two nearly equal numbers, causing a loss of significance.

**Beginner-Friendly Explanation:** Precision is about how many digits of a number you can trust. `float64` can store about 15 digits; `float32` about 7; `float16` about 3. If you're doing calculations where small differences matter, you need higher precision. If you're just storing rough approximations, lower precision is fine.

### Purposes

- To ensure that computed results are accurate enough for the application.
- To avoid catastrophic cancellation in subtraction-heavy algorithms.
- To prevent rounding errors from accumulating in recursive or iterative computations.
- To balance the cost of higher precision against the need for accuracy.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Check precision limits
np.finfo(np.float16).eps   # 9.77e-04
np.finfo(np.float32).eps   # 1.19e-07
np.finfo(np.float64).eps   # 2.22e-16

# Check decimal precision
np.finfo(np.float32).precision   # 6
np.finfo(np.float64).precision   # 15
```

**Component Breakdown:**

| Dtype | Mantissa Bits | Decimal Digits | Machine Epsilon |
|-------|--------------|----------------|-----------------|
| `float16` | 10 | ~3.3 | 9.77×10⁻⁴ |
| `float32` | 23 | ~7.2 | 1.19×10⁻⁷ |
| `float64` | 52 | ~15.9 | 2.22×10⁻¹⁶ |

**Syntax Rules:**

- Machine epsilon (`eps`) is the difference between 1.0 and the next representable float.
- Use `np.finfo(dtype).precision` for the approximate number of decimal digits.
- Catastrophic cancellation occurs when subtracting nearly equal numbers; the result loses precision relative to the operands.

**Constraints and Limitations:**

- `float32` cannot represent 0.1 exactly; accumulated errors can become significant.
- `float16` has very limited precision and range; use only when memory is critical.
- Higher precision does not eliminate all errors; it only reduces their magnitude.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Precision Loss and Catastrophic Cancellation**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Demonstrate precision loss in float32.
x = np.float32(1e10)
y = x + 1 - x
print("float32: 1e10 + 1 - 1e10 =", y)
print("  Expected: 1.0, but got 0.0 due to precision loss")
print()

# Step 3: The same computation in float64.
x64 = np.float64(1e10)
y64 = x64 + 1 - x64
print("float64: 1e10 + 1 - 1e10 =", y64)
print("  Expected: 1.0, but got 0.0 because 1e10 exceeds float64 precision")
print()

# Step 4: Catastrophic cancellation with square roots.
a = 1e8
result_f32 = np.float32(np.sqrt(a + 1)) - np.float32(np.sqrt(a))
result_f64 = np.float64(np.sqrt(a + 1)) - np.float64(np.sqrt(a))
print(f"float32: sqrt(1e8+1) - sqrt(1e8) = {result_f32:.10f}")
print(f"float64: sqrt(1e8+1) - sqrt(1e8) = {result_f64:.10f}")
print("  (float64 captures the small difference; float32 loses it)")
print()

# Step 5: Machine epsilon comparison.
for dtype in [np.float16, np.float32, np.float64]:
    print(f"{dtype.__name__}: eps = {np.finfo(dtype).eps:.6e}, "
          f"precision = {np.finfo(dtype).precision} digits")
```

**Expected Output:**

```
float32: 1e10 + 1 - 1e10 = 0.0
  Expected: 1.0, but got 0.0 due to precision loss

float64: 1e10 + 1 - 1e10 = 0.0
  Expected: 1.0, but got 0.0 because 1e10 exceeds float64 precision

float32: sqrt(1e8+1) - sqrt(1e8) = 0.0000000000
float64: sqrt(1e8+1) - sqrt(1e8) = 0.0000000018
  (float64 captures the small difference; float32 loses it)

float16: eps = 9.765625e-04, precision = 3 digits
float32: eps = 1.192093e-07, precision = 6 digits
float64: eps = 2.220446e-16, precision = 15 digits
```

**Why This Result Occurs:** In `float32`, `1e10` has only about 7 significant digits, so adding 1 (which is at the 11th digit place) is lost, and subtracting `1e10` returns 0. In `float64`, `1e10` still exceeds the 15–16 digit precision, so the same issue occurs. The square root example shows that `float64` can capture a difference of ~1.8×10⁻⁹, while `float32` cannot.

### Real-World Cases

- **Scientific Simulations:** Long-running iterative simulations require `float64` to prevent error accumulation.
- **Financial Calculations:** Summing many small transactions requires `float64` or integer cents to avoid rounding drift.
- **Neural Networks:** `float32` is sufficient for most training; `float16` is used for mixed-precision training with careful scaling.

### References

- Lab 5: Numerical Stability in Python – https://pages.llf-paris.fr/~gwisniewski/assets/2025/lab_pro_skills/pro_skills_lab5/Numerical_Stability_Lama_YASSIN_Code_and_Report.pdf
- Common dtype Pitfalls – https://fintechpython.pages.oit.duke.edu/jupyternotebooks/_sources/6-Data%20Science/2-numpy-datatype_pitfalls.ipynb

---

## Core Concept 3: Computational Efficiency

### Definitions

**Core Definition:** Computational efficiency in dtype selection refers to how well a dtype aligns with the underlying CPU/GPU hardware, affecting the speed of arithmetic and memory operations.

**Technical Definition:** Modern CPUs and GPUs have native support for specific bit-widths. 32-bit and 64-bit floating-point operations are typically hardware-accelerated. GPUs are often optimized for 32-bit and 16-bit operations (especially for AI workloads). Memory alignment—whether the data's memory offset is a multiple of its natural alignment—also affects speed. NumPy's `dtype.alignment` attribute reflects the "true alignment" of the type.

**Beginner-Friendly Explanation:** Computers have "native" sizes for numbers—usually 32 or 64 bits. If your data matches those sizes, the computer can process it faster. Using a dtype that doesn't match can slow things down because the computer has to do extra work to break it into pieces.

### Purposes

- To maximize throughput of vectorized operations by matching native hardware widths.
- To improve cache utilization and reduce memory bandwidth pressure.
- To leverage SIMD (Single Instruction, Multiple Data) instructions that operate on specific bit-widths.
- To reduce energy consumption in large-scale computations.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Check dtype alignment
np.dtype(np.float64).alignment   # typically 8
np.dtype(np.float32).alignment   # typically 4

# Check array alignment flag
arr.flags['ALIGNED']   # True if all elements are aligned
```

**Component Breakdown:**

| Dtype | Typical Alignment (bytes) | Hardware Support |
|-------|--------------------------|------------------|
| `float64` | 8 | Native on x64 |
| `float32` | 4 | Native on most CPUs/GPUs |
| `float16` | 2 | GPU-optimized (AI workloads) |
| `int32` | 4 | Native |
| `int64` | 8 | Native |

**Syntax Rules:**

- Aligned arrays are faster to process than unaligned ones.
- NumPy uses "true alignment" (architecture-dependent) and "uint alignment" (for copy operations).
- Structured dtypes with `align=True` match C-struct layout but increase size.

**Constraints and Limitations:**

- Alignment is architecture-dependent; code optimized for one platform may not be optimal on another.
- Unaligned arrays can be up to 4× slower for certain operations.
- GPU performance depends on the specific hardware; `float16` is fast on modern GPUs but not on older ones.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Alignment and Performance**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Check dtype alignment.
for dtype in [np.float64, np.float32, np.int64, np.int32]:
    print(f"{dtype.__name__}: alignment = {np.dtype(dtype).alignment} bytes")

print()

# Step 3: Create aligned and unaligned arrays.
aligned = np.zeros(1000, dtype=np.float64)
print("Aligned array flags:", aligned.flags['ALIGNED'])

# Create an unaligned view by slicing a byte array.
byte_arr = np.zeros(1001, dtype=np.uint8)
unaligned = byte_arr[1:].view(np.float64)
print("Unaligned array flags:", unaligned.flags['ALIGNED'])
print()

# Step 4: Compare memory layout sizes for structured dtypes.
dt_packed = np.dtype([('a', 'u1'), ('b', 'u8')])
dt_aligned = np.dtype([('a', 'u1'), ('b', 'u8')], align=True)
print(f"Packed dtype itemsize: {dt_packed.itemsize} bytes")
print(f"Aligned dtype itemsize: {dt_aligned.itemsize} bytes")
print("  (aligned adds padding for better CPU access)")
```

**Expected Output:**

```
float64: alignment = 8 bytes
float32: alignment = 4 bytes
int64: alignment = 8 bytes
int32: alignment = 4 bytes

Aligned array flags: True
Unaligned array flags: False

Packed dtype itemsize: 9 bytes
Aligned dtype itemsize: 16 bytes
  (aligned adds padding for better CPU access)
```

**Why This Result Occurs:** `float64` requires 8-byte alignment; `float32` requires 4-byte alignment. The aligned array satisfies these requirements; the unaligned view does not. For structured dtypes, `align=True` adds padding to match C-struct layout, increasing size from 9 to 16 bytes but enabling faster field access.

### Real-World Cases

- **Deep Learning on GPUs:** `float32` and `float16` operations are hardware-accelerated on NVIDIA GPUs, offering significant speedups over `float64`.
- **Numerical Linear Algebra:** BLAS/LAPACK routines are optimized for aligned, contiguous `float64` arrays.
- **Image Processing:** Using `uint8` for pixel data aligns with the native byte-level operations of image processing libraries.

### References

- Memory alignment – https://numpy.org/doc/2.2/dev/alignment.html
- NumPy dtypes and Precision — Memory, Speed, and ML – https://github.com/ReddyBytes/Python-DSA-API-Mastery/blob/main/01_Python_Mastery/22_numpy_for_ai/01_dtype_and_precision.md

---

## Core Concept 4: Integer Range

### Definitions

**Core Definition:** Integer range refers to the set of values representable by a given integer dtype, determined by its bit-width and signedness.

**Technical Definition:** Signed integers use two's complement representation with ranges from −2⁽ⁿ⁻¹⁾ to 2⁽ⁿ⁻¹⁾−1. Unsigned integers range from 0 to 2ⁿ−1. NumPy provides `int8`, `int16`, `int32`, `int64` (signed) and `uint8`, `uint16`, `uint32`, `uint64` (unsigned). When a value exceeds the range, NumPy silently wraps around (overflow), producing incorrect results without raising an error.

**Beginner-Friendly Explanation:** Each integer type has a maximum and minimum value it can hold. If you try to store a number outside that range, it wraps around to the other end—like a car odometer rolling over. Choosing the right range means picking a type that's big enough for your largest values but not unnecessarily large.

### Purposes

- To ensure that all values in a dataset can be represented without overflow.
- To select the smallest integer type that accommodates the required range, saving memory.
- To use unsigned integers for data that is naturally non-negative (e.g., pixel values, ages, counts).
- To avoid silent data corruption from integer overflow.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Check integer range
np.iinfo(np.int8).min    # -128
np.iinfo(np.int8).max    # 127
np.iinfo(np.uint8).max   # 255

# Create arrays with specific integer dtypes
arr = np.array([100, 200, 255], dtype=np.uint8)
```

**Component Breakdown:**

| Type | Bit-Width | Signed Range | Unsigned Range |
|------|-----------|-------------|----------------|
| `int8` / `uint8` | 8 | −128 to 127 | 0 to 255 |
| `int16` / `uint16` | 16 | −32,768 to 32,767 | 0 to 65,535 |
| `int32` / `uint32` | 32 | ±2.1×10⁹ | 0 to 4.3×10⁹ |
| `int64` / `uint64` | 64 | ±9.2×10¹⁸ | 0 to 1.8×10¹⁹ |

**Syntax Rules:**

- `np.int_` is an alias for the platform's default integer (typically `int64` on 64-bit systems).
- Integer overflow wraps around silently; no exception is raised.
- Use `np.iinfo(dtype)` to inspect the exact range for any integer dtype.

**Constraints and Limitations:**

- Overflow is silent and can produce completely wrong results.
- Mixing signed and unsigned integers in operations can produce unexpected promotion.
- `uint8` is critical for image byte representations (0–255).

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Integer Range and Overflow**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Check integer ranges.
for dtype in [np.int8, np.uint8, np.int16, np.int32, np.int64]:
    info = np.iinfo(dtype)
    print(f"{dtype.__name__}: {info.min} to {info.max}")

print()

# Step 3: Demonstrate overflow.
a = np.array([127], dtype=np.int8)
print("int8 127 + 1:", a + 1)
print("  (wraps to -128)")

b = np.array([255], dtype=np.uint8)
print("uint8 255 + 1:", b + 1)
print("  (wraps to 0)")

c = np.array([2**31 - 1], dtype=np.int32)
print("int32 max + 1:", c + 1)
print("  (wraps to negative minimum)")

print()

# Step 4: Prevent overflow by upcasting.
safe = a.astype(np.int32) + 1
print("After upcasting to int32:", safe)
```

**Expected Output:**

```
int8: -128 to 127
uint8: 0 to 255
int16: -32768 to 32767
int32: -2147483648 to 2147483647
int64: -9223372036854775808 to 9223372036854775807

int8 127 + 1: [-128]
  (wraps to -128)
uint8 255 + 1: [0]
  (wraps to 0)
int32 max + 1: [-2147483648]
  (wraps to negative minimum)

After upcasting to int32: [128]
```

**Why This Result Occurs:** `int8` holds −128 to 127; 127 + 1 wraps to −128. `uint8` holds 0 to 255; 255 + 1 wraps to 0. `int32` max + 1 wraps to its minimum. Upcasting to `int32` before the addition avoids overflow because `int32` can hold 128.

### Real-World Cases

- **Image Processing:** Pixel values 0–255 fit perfectly in `uint8`.
- **Database IDs:** Auto-incrementing IDs may exceed `int32` (2.1 billion) and require `int64`.
- **Sensor Readings:** 16-bit ADCs produce values 0–65,535, fitting `uint16`.
- **Counters:** Use `int64` for counters that may exceed 2 billion.

### References

- Common dtype Pitfalls – https://fintechpython.pages.oit.duke.edu/jupyternotebooks/_sources/6-Data%20Science/2-numpy-datatype_pitfalls.ipynb
- NumPy Data Types – https://www.datacamp.com/nl/doc/numpy/data-types

---

## Core Concept 5: Floating-Point Limitations

### Definitions

**Core Definition:** Floating-point limitations are the inherent constraints of IEEE 754 floating-point representation, including machine epsilon, NaN behavior, and precision limits.

**Technical Definition:** IEEE 754 floating-point numbers have finite precision (machine epsilon) and limited range. Special values include NaN (Not a Number), +Inf, and −Inf. NaN has the unique property that `NaN != NaN` evaluates to `True`. Machine epsilon (`eps`) is the difference between 1.0 and the next representable float. `np.finfo()` provides these limits.

**Beginner-Friendly Explanation:** Floating-point numbers can't represent every real number exactly. There's a smallest difference they can detect (machine epsilon), and some operations produce special values like NaN and infinity. Understanding these limits helps avoid subtle bugs.

### Purposes

- To understand the smallest difference a float type can detect (machine epsilon).
- To handle NaN and infinity correctly in computations.
- To compare floating-point numbers safely (avoiding `==` with NaN).
- To choose the appropriate precision for a given application.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Check machine limits
np.finfo(np.float32).eps        # 1.19e-07
np.finfo(np.float64).eps        # 2.22e-16
np.finfo(np.float32).max        # 3.40e+38
np.finfo(np.float32).min        # -3.40e+38
np.finfo(np.float32).tiny       # 1.18e-38 (smallest normal)
np.finfo(np.float32).precision  # 6 (decimal digits)

# NaN comparison
np.nan == np.nan   # False
np.isnan(np.nan)   # True
```

**Component Breakdown:**

| Attribute | Description |
|-----------|-------------|
| `eps` | Difference between 1.0 and next representable float. |
| `max` | Largest representable finite value. |
| `min` | Most negative finite value. |
| `tiny` | Smallest positive normal value. |
| `precision` | Approximate number of decimal digits. |

**Syntax Rules:**

- Use `np.isnan()` to test for NaN; never use `==`.
- Use `np.isinf()` to test for infinity.
- Use `np.finfo(dtype)` to inspect limits for any float dtype.
- Subnormal numbers exist between 0 and `tiny` but with reduced precision.

**Constraints and Limitations:**

- NaN propagates through most operations.
- Comparisons with NaN are always `False` (except `!=`).
- `float16` has very limited precision and range; use with caution.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Floating-Point Limits and NaN Handling**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Check machine epsilon and precision.
for dtype in [np.float16, np.float32, np.float64]:
    info = np.finfo(dtype)
    print(f"{dtype.__name__}:")
    print(f"  eps: {info.eps:.6e}")
    print(f"  precision: {info.precision} digits")
    print(f"  max: {info.max:.4e}")
    print(f"  tiny: {info.tiny:.4e}")
    print()

# Step 3: Demonstrate NaN behavior.
a = np.array([1.0, np.nan, 3.0])
print("Array with NaN:", a)
print("NaN == NaN:", np.nan == np.nan)
print("np.isnan(a):", np.isnan(a))
print()

# Step 4: Safe NaN handling.
clean = a[~np.isnan(a)]
print("Array without NaN:", clean)
print("Sum (NaN-aware):", np.nansum(a))
```

**Expected Output:**

```
float16:
  eps: 9.765625e-04
  precision: 3 digits
  max: 6.5504e+04
  tiny: 6.1035e-05

float32:
  eps: 1.192093e-07
  precision: 6 digits
  max: 3.4028e+38
  tiny: 1.1755e-38

float64:
  eps: 2.220446e-16
  precision: 15 digits
  max: 1.7977e+308
  tiny: 2.2251e-308

Array with NaN: [ 1. nan  3.]
NaN == NaN: False
np.isnan(a): [False  True False]

Array without NaN: [1. 3.]
Sum (NaN-aware): 4.0
```

**Why This Result Occurs:** `np.finfo()` reveals the machine limits for each float type. `float16` has only 3 decimal digits of precision, while `float64` has 15. `NaN == NaN` is `False` because NaN is not equal to anything, including itself. Using `np.isnan()` and `np.nansum()` correctly handles NaN values.

### Real-World Cases

- **Data Cleaning:** Detecting and removing NaN values from datasets.
- **Numerical Algorithms:** Using machine epsilon to set convergence tolerances.
- **Scientific Computing:** Avoiding equality comparisons with floating-point numbers; using `np.isclose()` instead.

### References

- numpy.finfo – https://numpy.org/doc/1.22/reference/generated/numpy.finfo.html
- Common dtype Pitfalls – https://fintechpython.pages.oit.duke.edu/jupyternotebooks/_sources/6-Data%20Science/2-numpy-datatype_pitfalls.ipynb
- NumPy Data Types – https://www.datacamp.com/nl/doc/numpy/data-types

---

## Summary of Deprecated and Version-Specific Features

| Feature | Status | Recommendation |
|---------|--------|----------------|
| `np.find_common_type` | Deprecated | Use `np.result_type` or `np.promote_types` |
| Value-based promotion (pre-NEP 50) | Removed in NumPy 2.0 | Use NEP 50 rules; Python scalars are "weakly typed" |
| `np.can_cast` with Python scalars | Removed in NumPy 2.0 | Use dtypes or NumPy scalars explicitly |
| `float16` for general computation | Not recommended | Use `float32` or `float64` unless memory is critical |

---

## Consolidated Reference List

- NumPy dtypes and Precision — Memory, Speed, and ML – https://github.com/ReddyBytes/Python-DSA-API-Mastery/blob/main/01_Python_Mastery/22_numpy_for_ai/01_dtype_and_precision.md
- Memory alignment – https://numpy.org/doc/2.2/dev/alignment.html
- Common dtype Pitfalls – https://fintechpython.pages.oit.duke.edu/jupyternotebooks/_sources/6-Data%20Science/2-numpy-datatype_pitfalls.ipynb
- numpy.finfo – https://numpy.org/doc/1.22/reference/generated/numpy.finfo.html
- NumPy Data Types – https://www.datacamp.com/nl/doc/numpy/data-types
- Lab 5: Numerical Stability in Python – https://pages.llf-paris.fr/~gwisniewski/assets/2025/lab_pro_skills/pro_skills_lab5/Numerical_Stability_Lama_YASSIN_Code_and_Report.pdf
- Data type promotion in NumPy – https://numpy.org/doc/2.1/reference/arrays.promotion.html
- NEP 50 — Promotion rules for Python scalars – https://numpy.org/neps/nep-0050-scalar-promotion.html
- numpy.iinfo – https://numpy.org/doc/stable/reference/generated/numpy.iinfo.html