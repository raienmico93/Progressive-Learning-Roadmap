# NumPy dtypes: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A NumPy data type (`dtype`) is an object that describes how the bytes in a fixed-size block of memory corresponding to an array item should be interpreted, specifying the type of data (integer, float, Python object, etc.), its size in bytes, and the byte order.

**Technical Definition:** A `dtype` object is an instance of the `numpy.dtype` class that describes the data type, size, byte order, and—for structured data types—the names and types of fields. It may also describe sub-array shapes and structured aggregates of other data types.

**Beginner-Friendly Explanation:** A `dtype` is like a label on a box that says what's inside: "all 32-bit integers," or "all 64-bit floating-point numbers." Because every element in a NumPy array must have the same dtype, the computer knows exactly how many bytes each element occupies and can process the array uniformly and efficiently.

### Key Characteristics

- **Homogeneity:** All elements in a NumPy array share the same dtype.
- **Fixed Size:** Most NumPy dtypes have a fixed byte width (e.g., `int32` is always 4 bytes).
- **Byte Order:** Dtypes specify endianness (little-endian `<` or big-endian `>`).
- **Flexible Width:** String and void dtypes can have variable byte widths; the new `StringDType` supports variable-width UTF-8 strings.
- **Structured Types:** Dtypes can be composed of named fields of different types, enabling record-like arrays.
- **Platform Dependence:** Some types (e.g., `int_`, `float_`) are aliases whose exact width depends on the platform.

### Prerequisites

- NumPy installed (`pip install numpy`).
- Basic Python knowledge: variables, lists, and type concepts.
- Familiarity with the `import numpy as np` convention.
- Conceptual understanding of computer memory (bytes, bit-widths).

### Related Programming Areas

- **Data Science:** Choosing appropriate dtypes for memory efficiency and numerical precision.
- **Machine Learning:** Using `float16`/`float32` to reduce memory footprints in neural networks.
- **Image Processing:** Using `uint8` for pixel values (0–255).
- **Scientific Computing:** Using `float64` for high-precision calculations.
- **Database and Tabular Data:** Using structured dtypes to represent rows with mixed types.

### Core Concepts / Features

The following dtype categories are explored in detail: (1) Integer types, (2) Floating-point types, (3) Complex types, (4) Boolean, (5) String and Unicode, (6) Object dtype, (7) Datetime types, and (8) Structured and Record Arrays.

---

## Core Concept 1: Integer Types

### Definitions

**Core Definition:** Integer dtypes represent whole numbers, either signed (allowing negative values) or unsigned (non-negative only), with bit-widths ranging from 8 to 64 bits.

**Technical Definition:** NumPy provides bit-sized integer types: `int8`, `int16`, `int32`, `int64` (signed) and `uint8`, `uint16`, `uint32`, `uint64` (unsigned). Each has a corresponding DType class in `numpy.dtypes` (e.g., `Int8DType`, `UInt8DType`). Signed integers use two's complement notation.

**Beginner-Friendly Explanation:** Integer dtypes store whole numbers. Signed types can hold negative values; unsigned types can only hold zero and positive values. The number (8, 16, 32, 64) tells you how many bits each value uses—larger numbers mean a wider range but more memory.

### Purposes

- To store discrete counts, indices, and identifiers.
- To select the smallest bit-width that accommodates the required range, minimizing memory usage.
- To use unsigned integers for data that is naturally non-negative (e.g., pixel values, ages, counts).
- To enable bitwise operations and array indexing.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Signed integers
np.int8, np.int16, np.int32, np.int64

# Unsigned integers
np.uint8, np.uint16, np.uint32, np.uint64

# Creating arrays with integer dtypes
arr = np.array([1, 2, 3], dtype=np.int32)
```

**Component Breakdown:**

| Type | Bit-Width | Signed Range | Unsigned Range |
|------|-----------|-------------|----------------|
| `int8` / `uint8` | 8 | −128 to 127 | 0 to 255 |
| `int16` / `uint16` | 16 | −32,768 to 32,767 | 0 to 65,535 |
| `int32` / `uint32` | 32 | −2,147,483,648 to 2,147,483,647 | 0 to 4,294,967,295 |
| `int64` / `uint64` | 64 | −9,223,372,036,854,775,808 to 9,223,372,036,854,775,807 | 0 to 18,446,744,073,709,551,615 |

**Syntax Rules:**

- `np.int_` is an alias for the platform's default integer (typically `int64` on 64-bit systems).
- Signed integers use two's complement notation.
- Unsigned integers wrap around modulo 2ⁿ on overflow.

**Constraints and Limitations:**

- Overflow behavior: arithmetic on integers wraps around (no error is raised).
- Mixing signed and unsigned integers in operations can produce unexpected results due to type promotion rules.
- `uint8` is critical for image byte representations (0–255).

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Integer Dtype Ranges and Overflow**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create arrays with different integer dtypes.
a = np.array([127], dtype=np.int8)
b = np.array([255], dtype=np.uint8)
print("int8 127:", a, "| dtype:", a.dtype)
print("uint8 255:", b, "| dtype:", b.dtype)
print()

# Step 3: Demonstrate overflow (wraparound).
a_overflow = np.array([128], dtype=np.int8)
print("int8 128 (overflows to -128):", a_overflow)
b_overflow = np.array([256], dtype=np.uint8)
print("uint8 256 (overflows to 0):", b_overflow)
print()

# Step 4: Check ranges using iinfo.
print("int8 range:", np.iinfo(np.int8).min, "to", np.iinfo(np.int8).max)
print("uint8 range:", np.iinfo(np.uint8).min, "to", np.iinfo(np.uint8).max)
print("int32 range:", np.iinfo(np.int32).min, "to", np.iinfo(np.int32).max)
print()

# Step 5: Demonstrate type promotion between signed and unsigned.
x = np.array([1], dtype=np.int8)
y = np.array([2], dtype=np.uint8)
result = x + y
print("int8 + uint8 -> dtype:", result.dtype)
```

**Expected Output:**

```
int8 127: [127] | dtype: int8
uint8 255: [255] | dtype: uint8

int8 128 (overflows to -128): [-128]
uint8 256 (overflows to 0): [0]

int8 range: -128 to 127
uint8 range: 0 to 255
int32 range: -2147483648 to 2147483647

int8 + uint8 -> dtype: int16
```

**Why This Result Occurs:** `int8` can hold values from −128 to 127; assigning 128 causes an overflow that wraps to −128 (two's complement). `uint8` holds 0 to 255; assigning 256 wraps to 0. `np.iinfo()` provides the exact range for any integer dtype. When adding `int8` and `uint8`, NumPy promotes both to `int16` to accommodate the full range of possible results.

### Real-World Cases

- **Image Processing:** Pixel values are stored as `uint8` (0–255), the natural format for image files.
- **Sensor Data:** 16-bit integers are common for ADC readings from sensors.
- **Database IDs:** 64-bit integers for large-scale identifiers.

### References

- Data type classes (numpy.dtypes) – https://numpy.org/devdocs/reference/routines.dtypes.html
- Data type objects (dtype) – https://numpy.org/doc/stable/reference/arrays.dtypes.html
- numpy.iinfo – https://numpy.org/doc/stable/reference/generated/numpy.iinfo.html

---

## Core Concept 2: Floating-Point Types

### Definitions

**Core Definition:** Floating-point dtypes represent real numbers with fractional parts, following the IEEE 754 standard with different precision levels: `float16` (half), `float32` (single), and `float64` (double).

**Technical Definition:** NumPy floating-point types correspond to IEEE 754 binary formats: `float16` (1 sign bit, 5 exponent bits, 10 mantissa bits), `float32` (1 sign bit, 8 exponent bits, 23 mantissa bits), and `float64` (1 sign bit, 11 exponent bits, 52 mantissa bits). `float64` is the default floating-point dtype.

**Beginner-Friendly Explanation:** Floating-point types store numbers with decimal points. `float16` uses the least memory but has limited precision and range; `float32` is a good balance; `float64` is the most precise and is the default.

### Purposes

- To represent real-valued measurements, probabilities, and scientific data.
- To balance memory usage against numerical precision (e.g., `float16` for neural networks).
- To ensure compatibility with IEEE 754 hardware and libraries.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.float16, np.float32, np.float64
arr = np.array([1.5, 2.7], dtype=np.float32)
```

**Component Breakdown:**

| Type | Total Bits | Sign | Exponent | Mantissa | Decimal Digits | Range |
|------|-----------|------|----------|----------|---------------|-------|
| `float16` | 16 | 1 | 5 | 10 | ~3.3 | ±6.5×10⁴ |
| `float32` | 32 | 1 | 8 | 23 | ~7.2 | ±3.4×10³⁸ |
| `float64` | 64 | 1 | 11 | 52 | ~15.9 | ±1.8×10³⁰⁸ |

**Syntax Rules:**

- `np.float_` is an alias for `float64` (the default).
- `float16` has limited precision; values may not be exactly representable.
- Use `np.finfo()` to inspect precision and range properties.

**Constraints and Limitations:**

- `float16` has limited range (~6.5×10⁴) and precision (~3.3 decimal digits).
- Floating-point arithmetic is not associative; `(a + b) + c` may differ from `a + (b + c)`.
- Subnormal numbers exist near zero but with reduced precision.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Floating-Point Precision and Range**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create arrays with different float dtypes.
a = np.array([1.0 / 3.0], dtype=np.float16)
b = np.array([1.0 / 3.0], dtype=np.float32)
c = np.array([1.0 / 3.0], dtype=np.float64)
print("float16 1/3:", a)
print("float32 1/3:", b)
print("float64 1/3:", c)
print()

# Step 3: Check precision using finfo.
for dtype in [np.float16, np.float32, np.float64]:
    info = np.finfo(dtype)
    print(f"{dtype.__name__}: eps={info.eps:.6e}, max={info.max:.4e}")
print()

# Step 4: Demonstrate float16 range limitation.
large = np.float16(70000.0)
print("float16 70000:", large)
print("  (overflows to inf)")
```

**Expected Output:**

```
float16 1/3: [0.3333]
float32 1/3: [0.33333334]
float64 1/3: [0.33333333]

float16: eps=9.765625e-04, max=6.5504e+04
float32: eps=1.192093e-07, max=3.4028e+38
float64: eps=2.220446e-16, max=1.7977e+308

float16 70000: inf
  (overflows to inf)
```

**Why This Result Occurs:** `float16` can only represent about 3.3 decimal digits, so `1/3` is rounded to 0.3333. `float32` gives about 7 digits, and `float64` about 16. The `eps` value shows the smallest representable difference between numbers. `float16` cannot represent 70000 (above its maximum ~65504), so it overflows to infinity.

### Real-World Cases

- **Neural Networks:** `float16` (half precision) reduces memory and speeds up training on GPUs with minimal accuracy loss.
- **Scientific Computing:** `float64` (double precision) is the standard for numerical simulations and physics calculations.
- **Graphics:** `float32` is commonly used in GPU shaders and computer graphics.

### References

- Data type objects (dtype) – https://numpy.org/doc/stable/reference/arrays.dtypes.html
- numpy.finfo – https://numpy.org/doc/stable/reference/generated/numpy.finfo.html
- Floating-Point Guide – https://raw.githubusercontent.com

---

## Core Concept 3: Complex Types

### Definitions

**Core Definition:** Complex dtypes represent complex numbers with real and imaginary components, stored as pairs of floating-point values.

**Technical Definition:** `complex64` stores two 32-bit floats (real and imaginary parts), and `complex128` stores two 64-bit floats. `np.complex_` is an alias for `complex128`.

**Beginner-Friendly Explanation:** Complex types store numbers like `3 + 4j`, where `3` is the real part and `4` is the imaginary part. They're useful in engineering and physics where complex arithmetic is needed.

### Purposes

- To perform complex arithmetic in signal processing and electrical engineering.
- To represent phasors, impedance, and frequency-domain data.
- To solve differential equations in physics and engineering.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.complex64, np.complex128
arr = np.array([1+2j, 3+4j], dtype=np.complex128)
```

**Component Breakdown:**

| Type | Real Part | Imaginary Part | Total Bits |
|------|-----------|----------------|------------|
| `complex64` | `float32` | `float32` | 64 |
| `complex128` | `float64` | `float64` | 128 |

**Syntax Rules:**

- Complex numbers can be created using `j` notation (e.g., `1+2j`).
- Access real and imaginary parts via `.real` and `.imag` attributes.
- `complex128` is the default complex dtype.

**Constraints and Limitations:**

- Complex types do not support comparison operators like `<` or `>`.
- Operations like `abs()` return the magnitude (a float).

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Complex Arithmetic**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create a complex array.
z = np.array([1+2j, 3+4j], dtype=np.complex128)
print("Complex array:", z)
print("dtype:", z.dtype)
print()

# Step 3: Access real and imaginary parts.
print("Real parts:", z.real)
print("Imaginary parts:", z.imag)
print()

# Step 4: Perform complex arithmetic.
z_squared = z ** 2
print("z squared:", z_squared)
print()

# Step 5: Compute magnitude.
magnitude = np.abs(z)
print("Magnitude:", magnitude)
```

**Expected Output:**

```
Complex array: [1.+2.j 3.+4.j]
dtype: complex128

Real parts: [1. 3.]
Imaginary parts: [2. 4.]

z squared: [-3.+4.j -7.+24.j]

Magnitude: [2.23606798 5.        ]
```

**Why This Result Occurs:** `(1+2j)² = 1 + 4j + 4j² = 1 + 4j − 4 = −3 + 4j`. The magnitude of a complex number `a+bi` is `√(a² + b²)`.

### Real-World Cases

- **Signal Processing:** FFT results are complex arrays.
- **Electrical Engineering:** Impedance calculations use complex numbers.
- **Quantum Mechanics:** Wavefunctions are complex-valued.

### References

- Data type objects (dtype) – https://numpy.org/doc/stable/reference/arrays.dtypes.html

---

## Core Concept 4: Boolean

### Definitions

**Core Definition:** The boolean dtype (`bool_`) stores `True` or `False` values, using 1 byte per element.

**Technical Definition:** `numpy.bool_` is stored as a byte, where `False` is represented by 0 and `True` by any non-zero value (typically 1).

**Beginner-Friendly Explanation:** Boolean dtypes store true/false values. They're the result of comparison operations and are used for masking and filtering.

### Purposes

- To store the results of comparison operations.
- To create masks for boolean indexing.
- To represent binary states in data.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.bool_
arr = np.array([True, False, True], dtype=bool)
```

**Component Breakdown:**

| Value | Stored As |
|-------|-----------|
| `False` | 0 |
| `True` | 1 (or any non-zero) |

**Syntax Rules:**

- Booleans are stored as 1 byte per element.
- Comparison operators return boolean arrays.
- Logical operations use `&`, `|`, `~` (not `and`, `or`, `not`).

**Constraints and Limitations:**

- Boolean arrays cannot be used for arithmetic in the usual sense (though they can be summed to count `True` values).

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Boolean Arrays and Masking**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create a boolean array.
mask = np.array([True, False, True, False])
print("Boolean array:", mask)
print("dtype:", mask.dtype)
print()

# Step 3: Use as a mask for indexing.
data = np.array([10, 20, 30, 40])
filtered = data[mask]
print("Filtered data:", filtered)
print()

# Step 4: Count True values.
count = np.sum(mask)
print("Number of True values:", count)
```

**Expected Output:**

```
Boolean array: [ True False  True False]
dtype: bool

Filtered data: [10 30]

Number of True values: 2
```

**Why This Result Occurs:** The boolean mask selects elements where it is `True` (indices 0 and 2), producing `[10, 30]`. Summing a boolean array counts the `True` values (True is treated as 1).

### Real-World Cases

- **Data Filtering:** Selecting rows where a condition is met.
- **Image Segmentation:** Creating masks for regions of interest.
- **Quality Control:** Flagging valid/invalid measurements.

### References

- Data type objects (dtype) – https://numpy.org/doc/stable/reference/arrays.dtypes.html

---

## Core Concept 5: String and Unicode

### Definitions

**Core Definition:** String dtypes store text data. NumPy provides fixed-width types (`S` for ASCII bytes, `U` for UTF-32 Unicode) and the modern variable-width `StringDType` for UTF-8 strings.

**Technical Definition:** The `S` dtype (e.g., `'S5'`) stores fixed-width byte strings. The `U` dtype (e.g., `'U5'`) stores fixed-width Unicode strings using UTF-32 encoding. `numpy.dtypes.StringDType` (new in NumPy 2.0) stores variable-width UTF-8 encoded strings, using inline storage for short strings and heap allocation for longer ones.

**Beginner-Friendly Explanation:** Fixed-width strings waste memory because every element is padded to the same length. The new `StringDType` stores only as much memory as each string needs, making it much more efficient for real-world text data.

### Purposes

- To store labels, names, and categorical data.
- To work with text data loaded from files or databases.
- To provide a memory-efficient alternative to object arrays for variable-length strings.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Fixed-width Unicode (UTF-32)
arr = np.array(["hello", "world"], dtype='U10')

# Fixed-width bytes (ASCII)
arr = np.array([b"hello", b"world"], dtype='S10')

# Variable-width UTF-8 (NumPy 2.0+)
from numpy.dtypes import StringDType
arr = np.array(["hello", "world"], dtype=StringDType())
```

**Component Breakdown:**

| Type | Character Code | Encoding | Width |
|------|---------------|----------|-------|
| `str_` | `U` | UTF-32 | Fixed |
| `bytes_` | `S` | ASCII | Fixed |
| `StringDType` | — | UTF-8 | Variable |

**Syntax Rules:**

- The `U` and `S` types require a maximum length; NumPy infers it if not specified.
- `StringDType` supports `na_object` (for missing values) and `coerce` parameters.
- `StringDType` is not as extensively tested as fixed-width types.

**Constraints and Limitations:**

- Fixed-width types waste memory for variable-length strings.
- `StringDType` is new in NumPy 2.0 and may not be supported by all downstream libraries.
- `StringDType` is not the default for `str` data; fixed-width `U` remains the default.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Fixed-Width vs. Variable-Width Strings**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create fixed-width Unicode array.
u_arr = np.array(["hello", "world"])
print("Fixed-width Unicode:", u_arr)
print("dtype:", u_arr.dtype)
print("itemsize:", u_arr.itemsize, "bytes")
print()

# Step 3: Create variable-width StringDType array (NumPy 2.0+).
try:
    from numpy.dtypes import StringDType
    s_arr = np.array(["hello", "world"], dtype=StringDType())
    print("Variable-width StringDType:", s_arr)
    print("dtype:", s_arr.dtype)
except ImportError:
    print("StringDType requires NumPy 2.0+")
```

**Expected Output:**

```
Fixed-width Unicode: ['hello' 'world']
dtype: <U5
itemsize: 20 bytes

Variable-width StringDType: ['hello' 'world']
dtype: StringDType()
```

**Why This Result Occurs:** The fixed-width `U5` type allocates 20 bytes per element (5 Unicode code points × 4 bytes each in UTF-32). `StringDType` stores the same strings in UTF-8 with variable width, using less memory for ASCII text.

### Real-World Cases

- **Categorical Data:** Storing labels in machine learning datasets.
- **Natural Language Processing:** Storing tokens and text data.
- **Database Integration:** Mapping string fields from SQL databases.

### References

- Working with Arrays of Strings And Bytes – https://numpy.org/doc/stable/user/basics.strings.html
- NEP 55 — Add a UTF-8 variable-width string DType to NumPy – https://numpy.org/neps/nep-0055-string_dtype.html

---

## Core Concept 6: Object dtype

### Definitions

**Core Definition:** The object dtype (`object` or `O`) stores references to arbitrary Python objects, bypassing NumPy's type homogeneity and performance optimizations.

**Technical Definition:** An object array stores pointers to Python objects rather than the objects themselves. The array's `itemsize` is the size of a pointer (typically 8 bytes on 64-bit systems), but the pointed-to objects can be of any type and size. Operations on object arrays fall back to Python-level loops, eliminating vectorization benefits.

**Beginner-Friendly Explanation:** Object arrays are like Python lists that happen to be wrapped in a NumPy array. They can hold anything—strings, lists, dictionaries—but lose all the speed advantages of NumPy's typed arrays.

### Purposes

- To store heterogeneous data that cannot be represented by a single numeric dtype.
- To work with Python objects (e.g., `Decimal`, custom classes) in an array context.
- To serve as a fallback when no better dtype exists (e.g., ragged arrays).

### Syntax Rules and Structure

**Complete General Syntax:**

```python
arr = np.array([1, "hello", 3.14], dtype=object)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `dtype=object` | Stores references to Python objects. |
| `itemsize` | Size of a pointer (typically 8 bytes). |
| Operations | Fall back to Python-level loops. |

**Syntax Rules:**

- Object arrays can hold any Python object.
- Arithmetic operations on object arrays use Python's operators, not NumPy's vectorized versions.
- Ragged nested sequences must use `dtype=object` in NumPy 1.24+.

**Constraints and Limitations:**

- No vectorized performance; operations are Python-speed.
- Memory usage includes both the pointer array and the objects themselves.
- Comparisons and sorting may not work as expected for heterogeneous types.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Object Array with Mixed Types**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create an object array with mixed types.
arr = np.array([1, "hello", 3.14, [1, 2]], dtype=object)
print("Object array:", arr)
print("dtype:", arr.dtype)
print("itemsize:", arr.itemsize, "bytes")
print()

# Step 3: Demonstrate that operations are Python-level.
result = arr[0] + arr[2]  # 1 + 3.14
print("arr[0] + arr[2]:", result)
print()

# Step 4: Show ragged array creation with dtype=object.
ragged = np.array([[1, 2], [3, 4, 5]], dtype=object)
print("Ragged object array:", ragged)
print("Shape:", ragged.shape)
```

**Expected Output:**

```
Object array: [1 'hello' 3.14 list([1, 2])]
dtype: object
itemsize: 8 bytes

arr[0] + arr[2]: 4.14

Ragged object array: [list([1, 2]) list([3, 4, 5])]
Shape: (2,)
```

**Why This Result Occurs:** The object array stores pointers to Python objects of different types. The `itemsize` is 8 bytes (pointer size), but the pointed-to objects consume additional memory. The addition `arr[0] + arr[2]` uses Python's `+` operator. The ragged array has shape `(2,)` because each element is a Python list object.

### Real-World Cases

- **Ragged Data:** Variable-length sequences (e.g., sentences of different lengths).
- **Custom Objects:** Storing instances of user-defined classes in an array.
- **Heterogeneous Data:** Mixing numbers and strings in a single array when no structured dtype is appropriate.

### References

- Data type objects (dtype) – https://numpy.org/doc/stable/reference/arrays.dtypes.html
- NEP 55 — Add a UTF-8 variable-width string DType to NumPy – https://numpy.org/neps/nep-0055-string_dtype.html

---

## Core Concept 7: Datetime Types

### Definitions

**Core Definition:** NumPy provides `datetime64` for absolute time points and `timedelta64` for time intervals, both stored as 64-bit integers with a specified unit of resolution.

**Technical Definition:** `datetime64` encodes dates and times as 64-bit integers offset from a reference epoch. The unit of resolution is specified by a string code: `Y` (years), `M` (months), `W` (weeks), `D` (days), `h` (hours), `m` (minutes), `s` (seconds), `ms` (milliseconds), `us` (microseconds), `ns` (nanoseconds), `ps` (picoseconds), `fs` (femtoseconds), and `as` (attoseconds). `timedelta64` uses the same unit codes to represent durations.

**Beginner-Friendly Explanation:** `datetime64` stores a specific moment in time, and `timedelta64` stores a duration. You choose the precision—from years down to attoseconds—by specifying a unit code. The finer the precision, the shorter the total time span that can be represented.

### Purposes

- To represent timestamps for time series analysis.
- To compute durations between events.
- To perform vectorized date arithmetic.
- To index arrays by time.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Creating datetimes from strings
dt = np.datetime64('2024-01-15T12:30:00', 's')

# Creating timedeltas
td = np.timedelta64(5, 'D')  # 5 days

# Arrays of datetimes
dates = np.array(['2024-01-01', '2024-01-02'], dtype='datetime64[D]')
```

**Component Breakdown:**

| Unit Code | Meaning | Range (approx.) |
|-----------|---------|-----------------|
| `Y` | Years | ±2.9×10¹¹ years |
| `M` | Months | ±3.5×10¹² months |
| `W` | Weeks | ±1.5×10¹³ weeks |
| `D` | Days | ±1.1×10¹⁴ days |
| `h` | Hours | ±2.6×10¹⁵ hours |
| `m` | Minutes | ±1.5×10¹⁷ minutes |
| `s` | Seconds | ±9.2×10¹⁸ seconds |
| `ms` | Milliseconds | ±9.2×10¹⁵ ms |
| `us` | Microseconds | ±9.2×10¹² us |
| `ns` | Nanoseconds | ±9.2×10⁹ ns |
| `ps` | Picoseconds | ±9.2×10⁶ ps |
| `fs` | Femtoseconds | ±9.2×10³ fs |
| `as` | Attoseconds | ±9.2 as |

**Syntax Rules:**

- The unit is specified as a string suffix (e.g., `'datetime64[D]'`).
- `datetime64` supports the string `'NaT'` for "Not a Time" values.
- Arithmetic between `datetime64` and `timedelta64` produces `datetime64`.
- Subtraction of two `datetime64` values produces a `timedelta64`.

**Constraints and Limitations:**

- The choice of unit determines the trade-off between resolution and maximum representable time span.
- `datetime64` is "naive" (no timezone information).
- Units finer than nanoseconds have very limited range.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Datetime Arithmetic and Resolution**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create a datetime array.
dates = np.array(['2024-01-01', '2024-01-15', '2024-02-01'], dtype='datetime64[D]')
print("Dates:", dates)
print("dtype:", dates.dtype)
print()

# Step 3: Compute differences (timedeltas).
diffs = np.diff(dates)
print("Differences:", diffs)
print("dtype:", diffs.dtype)
print()

# Step 4: Add a timedelta to a datetime.
future = dates[0] + np.timedelta64(30, 'D')
print("2024-01-01 + 30 days:", future)
print()

# Step 5: Demonstrate resolution trade-off.
ns_date = np.datetime64('2024-01-01T00:00:00.123456789', 'ns')
print("Nanosecond precision:", ns_date)
print()

# Step 6: Create a datetime with finer resolution.
as_date = np.datetime64('2024-01-01T00:00:00.123456789123456789', 'as')
print("Attosecond precision:", as_date)
```

**Expected Output:**

```
Dates: ['2024-01-01' '2024-01-15' '2024-02-01']
dtype: datetime64[D]

Differences: [14 17]
dtype: timedelta64[D]

2024-01-01 + 30 days: 2024-01-31

Nanosecond precision: 2024-01-01T00:00:00.123456789

Attosecond precision: 2024-01-01T00:00:00.123456789123456789
```

**Why This Result Occurs:** The `datetime64[D]` array stores dates at day resolution. Differences between dates produce `timedelta64[D]` values. Adding a timedelta shifts the date. The nanosecond and attosecond examples show that finer units capture more decimal places, but with a reduced total representable range.

### Real-World Cases

- **Financial Time Series:** Timestamps of trades and quotes.
- **Scientific Measurements:** High-precision timestamps for experiments.
- **Log Analysis:** Parsing and comparing log entry times.

### References

- Datetimes and timedeltas – https://numpy.org/doc/stable/reference/arrays.datetime

---

## Core Concept 8: Structured and Record Arrays

### Definitions

**Core Definition:** Structured arrays are ndarrays whose dtype is a composition of simpler dtypes organized as a sequence of named fields, allowing each element to hold multiple heterogeneous values.

**Technical Definition:** A structured dtype is created by passing a list of tuples (field name, dtype) to `np.dtype()`. Each field has a name, a data type, and optionally a shape. Record arrays (`np.recarray`) are a subclass of structured arrays that allow field access via attribute notation (e.g., `arr.field` instead of `arr['field']`).

**Beginner-Friendly Explanation:** Structured arrays are like tables or spreadsheets. Each element is a row with named columns of different types—an integer ID, a float score, and a string name, for example. Record arrays are a convenience wrapper that lets you access columns with dot notation.

### Purposes

- To represent tabular data with heterogeneous column types.
- To map database row schemas into NumPy arrays.
- To store and manipulate records with named fields.
- To enable field-wise operations on structured data.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Creating a structured dtype
dt = np.dtype([('name', 'U10'), ('age', 'i4'), ('score', 'f4')])

# Creating a structured array
arr = np.array([('Alice', 25, 92.5), ('Bob', 30, 88.0)], dtype=dt)

# Accessing fields
names = arr['name']
ages = arr['age']

# Record array (attribute access)
rec = arr.view(np.recarray)
names = rec.name
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `dtype` list | List of `(field_name, field_dtype)` tuples. |
| Field access | `arr['field_name']` returns a view of that field. |
| Record array | `np.recarray` allows `arr.field_name` access. |

**Syntax Rules:**

- Field names must be unique strings.
- Field dtypes can be any valid NumPy dtype, including nested structured types.
- Accessing a field returns a view, not a copy.
- Record arrays are created by viewing a structured array as `np.recarray`.

**Constraints and Limitations:**

- Structured arrays are less memory-efficient than separate arrays for each field.
- Operations on structured arrays are limited compared to regular numeric arrays.
- Record arrays are legacy; regular structured arrays with field-name indexing are preferred.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Creating and Using Structured Arrays**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Define a structured dtype.
dt = np.dtype([('name', 'U10'), ('age', 'i4'), ('score', 'f4')])
print("Structured dtype:", dt)
print("Field names:", dt.names)
print()

# Step 3: Create a structured array.
data = np.array([('Alice', 25, 92.5),
                 ('Bob', 30, 88.0),
                 ('Charlie', 28, 95.5)], dtype=dt)
print("Structured array:\n", data)
print()

# Step 4: Access individual fields.
names = data['name']
ages = data['age']
scores = data['score']
print("Names:", names)
print("Ages:", ages)
print("Scores:", scores)
print()

# Step 5: Filter by field.
high_scorers = data[data['score'] > 90]
print("High scorers:\n", high_scorers)
print()

# Step 6: Create a record array for attribute access.
rec = data.view(np.recarray)
print("Record array attribute access:")
print("  rec.name:", rec.name)
print("  rec.age:", rec.age)
```

**Expected Output:**

```
Structured dtype: [('name', '<U10'), ('age', '<i4'), ('score', '<f4')]
Field names: ('name', 'age', 'score')

Structured array:
 [('Alice', 25, 92.5) ('Bob', 30, 88. ) ('Charlie', 28, 95.5)]

Names: ['Alice' 'Bob' 'Charlie']
Ages: [25 30 28]
Scores: [92.5 88.  95.5]

High scorers:
 [('Alice', 25, 92.5) ('Charlie', 28, 95.5)]

Record array attribute access:
  rec.name: ['Alice' 'Bob' 'Charlie']
  rec.age: [25 30 28]
```

**Why This Result Occurs:** The structured dtype defines three fields with different types. The array stores each record as a single element with the three fields packed together. Accessing `data['score']` returns a view of the score field as a regular float array. Filtering with `data[data['score'] > 90]` uses boolean indexing to select records. The record array view allows `rec.name` instead of `rec['name']`.

### Real-World Cases

- **Database Integration:** Mapping SQL table rows to structured arrays.
- **CSV Data:** Loading CSV files with mixed column types into structured arrays.
- **Sensor Data:** Storing readings with timestamps, sensor IDs, and values as named fields.
- **Scientific Data:** Storing experiment results with multiple measured quantities.

### References

- Structured arrays – https://numpy.org/doc/stable/user/basics.rec.html
- Data type objects (dtype) – https://numpy.org/doc/stable/reference/arrays.dtypes.html
- numpy.recarray – https://numpy.org/doc/stable/reference/generated/numpy.recarray.html

---

## Summary of Deprecated and Version-Specific Features

| Feature | Status | Recommendation |
|---------|--------|----------------|
| `StringDType` | New in NumPy 2.0 | Use for variable-width UTF-8 strings; not default for `str` |
| `np.recarray` | Legacy convenience class | Use structured arrays with field-name indexing |
| Fixed-width `U` and `S` types | Still supported; default for strings | Use `StringDType` for variable-width text |
| `np.complex_` | Alias for `complex128` | Use `complex128` explicitly |
| `np.float_` | Alias for `float64` | Use `float64` explicitly |

---

## Consolidated Reference List

- Data type objects (dtype) – https://numpy.org/doc/stable/reference/arrays.dtypes.html
- Data type classes (numpy.dtypes) – https://numpy.org/devdocs/reference/routines.dtypes.html
- Working with Arrays of Strings And Bytes – https://numpy.org/doc/stable/user/basics.strings.html
- NEP 55 — Add a UTF-8 variable-width string DType to NumPy – https://numpy.org/neps/nep-0055-string_dtype.html
- Datetimes and timedeltas – https://numpy.org/doc/stable/reference/arrays.datetime
- Structured arrays – https://numpy.org/doc/stable/user/basics.rec.html
- numpy.iinfo – https://numpy.org/doc/stable/reference/generated/numpy.iinfo.html
- numpy.finfo – https://numpy.org/doc/stable/reference/generated/numpy.finfo.html
- numpy.recarray – https://numpy.org/doc/stable/reference/generated/numpy.recarray.html
- NumPy Basics: Data Types – https://numpy.org/doc/stable/user/basics.types.html