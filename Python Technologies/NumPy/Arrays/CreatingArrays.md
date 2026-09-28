# Creating NumPy Arrays: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Array creation in NumPy is the process of instantiating `ndarray` objects from existing Python data structures, intrinsic constructor functions, or algorithmic generators.

**Technical Definition:** NumPy provides six general mechanisms for creating arrays: conversion from other Python structures (lists, tuples), intrinsic NumPy array creation functions (e.g., `arange`, `ones`, `zeros`), replicating/joining existing arrays, reading arrays from disk, creating arrays from raw bytes, and using special library functions such as random generators.

**Beginner-Friendly Explanation:** Creating a NumPy array is like building a container for your numbers. You can build it from a Python list you already have, or you can ask NumPy to build one for you—say, a container full of zeros, or a sequence of evenly spaced numbers, or a grid of random values. Each method produces a different kind of container suited to different tasks.

### Key Characteristics

- **Dtype Inference:** When creating arrays from Python sequences, NumPy infers the minimum dtype required to hold all elements.
- **Contiguity:** Arrays created from sequences are stored in contiguous memory (C-order by default).
- **View vs. Copy:** Most creation functions return a new array; some (like `reshape`) may return views.
- **Shape Flexibility:** Arrays can be created with any number of dimensions, from 0-D scalars to high-dimensional tensors.

### Prerequisites

- NumPy installed (`pip install numpy`).
- Basic Python knowledge: lists, tuples, and indexing.
- Familiarity with the `import numpy as np` convention.

### Related Programming Areas

- **Data Science:** Loading and preparing datasets.
- **Scientific Computing:** Generating simulation grids and coordinate arrays.
- **Machine Learning:** Creating random weight matrices and batch inputs.
- **Image Processing:** Constructing pixel coordinate grids and masks.

### Core Concepts / Features

The following core concepts are explored in detail: (1) Creating arrays from Python lists, (2) From tuples, (3) From nested sequences, (4) Constructor functions (`zeros`, `ones`, `empty`, `full`), (5) Numeric sequences (`arange`, `linspace`, `logspace`, `geomspace`), (6) Identity matrices, (7) Diagonal arrays, (8) Repeating arrays, and (9) Modern random sampling.

---

## Core Concept 1: Creating Arrays from Python Lists

### Definitions

**Core Definition:** Passing a Python list to `np.array()` converts the list into a 1-D NumPy array.

**Technical Definition:** `numpy.array(object, dtype=None)` creates an `ndarray` from any array-like object. A flat list of numbers produces a 1-D array whose dtype is inferred from the list's elements.

**Beginner-Friendly Explanation:** If you have a list of numbers in Python—like `[1, 2, 3, 4]`—you can turn it into a NumPy array by wrapping it in `np.array()`. NumPy reads the list and builds a compact, typed container from it.

### Purposes

- To convert existing Python data into NumPy's efficient array format.
- To leverage NumPy's vectorized operations on data that originated as Python lists.
- To specify an explicit dtype for type control during conversion.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
arr = np.array(list_object, dtype=None, copy=True, order='K')
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `list_object` | A Python list of numbers or other array-like elements. |
| `dtype` | Optional data type; if `None`, inferred from elements. |
| `copy` | If `True` (default), the array is copied. |
| `order` | Memory layout: `'K'` (keep), `'C'`, `'F'`. |

**Syntax Rules:**

- The list can contain integers, floats, booleans, or complex numbers.
- All elements should be convertible to a common dtype.
- Mixed-type lists (e.g., `[1, 'a']`) may result in an object dtype.

**Constraints and Limitations:**

- Creating arrays from Python lists is slower than using intrinsic constructors like `np.zeros()` because the data must be parsed element-by-element.
- Overflow can occur if an explicit dtype is too narrow (e.g., `np.int8` with values outside −128 to 127).

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Basic List-to-Array Conversion**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Define a Python list of integers.
py_list = [10, 20, 30, 40, 50]

# Step 3: Convert the list to a NumPy array.
arr = np.array(py_list)

# Step 4: Inspect the result.
print("Array:", arr)
print("Type:", type(arr))
print("dtype:", arr.dtype)
print("Shape:", arr.shape)
```

**Expected Output:**

```
Array: [10 20 30 40 50]
Type: <class 'numpy.ndarray'>
dtype: int64
Shape: (5,)
```

**Why This Result Occurs:** NumPy reads each element of the list, determines that all are integers, selects `int64` as the default integer dtype, and stores the five values in a contiguous block. The shape is `(5,)` because the list is one-dimensional.

**Example 2: Specifying dtype Explicitly**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create an array with an explicit float32 dtype.
arr_f32 = np.array([1, 2, 3, 4], dtype=np.float32)

# Step 3: Print the array and its dtype.
print("Array:", arr_f32)
print("dtype:", arr_f32.dtype)
print("itemsize:", arr_f32.itemsize, "bytes")
```

**Expected Output:**

```
Array: [1. 2. 3. 4.]
dtype: float32
itemsize: 4 bytes
```

**Why This Result Occurs:** The `dtype` argument forces NumPy to store each element as a 32-bit float, even though the input list contains integers. The values are displayed with decimal points because the dtype is floating-point.

### Real-World Cases

- **Data Loading:** After reading a CSV column into a Python list, convert it to a NumPy array for vectorized analysis.
- **Quick Prototyping:** When experimenting in a REPL, wrapping a list in `np.array()` is the fastest way to get an array for testing.
- **Interoperability:** Libraries that return Python lists (e.g., some JSON parsers) can be converted to NumPy arrays for numerical work.

### References

- NumPy Array creation – https://numpy.org/doc/1.25/user/basics.creation.html
- NumPy array() – https://numpy.org/doc/stable/reference/generated/numpy.array.html
- NumPy Quickstart – https://numpy.org/doc/stable/user/quickstart.html

---

## Core Concept 2: Creating Arrays from Tuples

### Definitions

**Core Definition:** A Python tuple can be passed to `np.array()` to create a 1-D NumPy array, just like a list.

**Technical Definition:** `numpy.array()` accepts any array-like object, including tuples. The tuple's elements are read and stored in a new `ndarray` with an inferred or specified dtype.

**Beginner-Friendly Explanation:** A tuple is just another way to group numbers in Python—like `(1, 2, 3)`. NumPy treats it the same way it treats a list: it reads the numbers and builds an array. The difference is that tuples are immutable in Python, but once converted to a NumPy array, the data can be modified (unless you set the array to read-only).

### Purposes

- To convert immutable Python data structures into mutable NumPy arrays.
- To create arrays from function return values that are tuples.
- To preserve the original tuple while working with array operations.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
arr = np.array(tuple_object, dtype=None)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `tuple_object` | A Python tuple of numbers or elements. |
| `dtype` | Optional data type; inferred if omitted. |

**Syntax Rules:**

- Tuples of numbers behave identically to lists in `np.array()`.
- Nested tuples create multidimensional arrays, just like nested lists.
- A tuple of tuples can create a 2-D array.

**Constraints and Limitations:**

- There is no performance difference between converting a list versus a tuple; both are parsed element-by-element.
- Tuples themselves remain immutable; only the resulting NumPy array is mutable.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Tuple-to-Array Conversion**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Define a Python tuple.
py_tuple = (3.14, 2.71, 1.41, 0.58)

# Step 3: Convert the tuple to a NumPy array.
arr = np.array(py_tuple)

# Step 4: Inspect the array.
print("Array:", arr)
print("dtype:", arr.dtype)
print("Shape:", arr.shape)
print("Original tuple unchanged:", py_tuple)
```

**Expected Output:**

```
Array: [3.14 2.71 1.41 0.58]
dtype: float64
Shape: (4,)
Original tuple unchanged: (3.14, 2.71, 1.41, 0.58)
```

**Why This Result Occurs:** NumPy reads the tuple's elements, infers `float64` because all values are floating-point numbers, and creates a new array. The original tuple remains unchanged because tuples are immutable in Python.

### Real-World Cases

- **Coordinate Pairs:** A tuple like `(x, y)` representing a point can be converted to an array for vectorized geometry operations.
- **Function Returns:** Functions that return multiple values as tuples can be directly converted to arrays.
- **Configuration Data:** Fixed configuration parameters stored as tuples can be converted for numerical processing.

### References

- NumPy Array creation – https://numpy.org/doc/1.25/user/basics.creation.html
- NumPy array() – https://numpy.org/doc/stable/reference/generated/numpy.array.html

---

## Core Concept 3: Creating Arrays from Nested Sequences

### Definitions

**Core Definition:** Nested sequences (lists of lists, tuples of tuples, or mixtures) are converted into multidimensional arrays, where the nesting depth determines the number of dimensions.

**Technical Definition:** `np.array()` recursively parses nested sequences. A list of lists creates a 2-D array, a list of lists of lists creates a 3-D array, and so on. All sub-sequences at the same level must have the same length (rectangular structure).

**Beginner-Friendly Explanation:** If you have a list that contains other lists—like `[[1, 2], [3, 4]]`—NumPy sees the outer list as rows and the inner lists as columns, creating a 2-D table. The deeper the nesting, the more dimensions the resulting array has.

### Purposes

- To construct multidimensional arrays from structured Python data.
- To represent matrices, tables, and higher-dimensional grids directly from nested lists.
- To validate the structural consistency of nested data.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
arr = np.array(nested_sequence, dtype=None)
```

**Component Breakdown:**

| Nesting Depth | Resulting Shape | Example Input |
|---------------|-----------------|---------------|
| 1 level | `(n,)` | `[1, 2, 3]` |
| 2 levels | `(m, n)` | `[[1, 2], [3, 4]]` |
| 3 levels | `(k, m, n)` | `[[[1, 2], [3, 4]], [[5, 6], [7, 8]]]` |

**Syntax Rules:**

- All inner sequences at the same depth must have the same length.
- Ragged (non-uniform) nested sequences are deprecated and will raise an error in recent NumPy versions unless `dtype=object` is explicitly specified.
- Mixing lists and tuples at different nesting levels is allowed.

**Constraints and Limitations:**

- **Ragged Arrays:** Creating an array from a ragged nested sequence (e.g., `[[1, 2], [3]]`) without specifying `dtype=object` raises a `ValueError` in NumPy 1.24 and later.
- The maximum number of dimensions is practically limited by memory and performance.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Creating a 2-D Array from Nested Lists**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Define a nested list (list of lists).
nested = [[1, 2, 3],
          [4, 5, 6]]

# Step 3: Convert to a 2-D array.
arr = np.array(nested)

# Step 4: Inspect the result.
print("Array:\n", arr)
print("Shape:", arr.shape)
print("ndim:", arr.ndim)
print("Total elements:", arr.size)
```

**Expected Output:**

```
Array:
 [[1 2 3]
 [4 5 6]]
Shape: (2, 3)
ndim: 2
Total elements: 6
```

**Why This Result Occurs:** The outer list has two elements (the two rows), and each inner list has three elements (the three columns). NumPy interprets this as a 2×3 matrix. The shape tuple `(2, 3)` reflects the number of elements at each nesting level.

**Example 2: Handling Ragged Nested Sequences Safely**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Define a ragged nested list (inconsistent inner lengths).
ragged = [[1, 2], [3, 4, 5]]

# Step 3: Attempt to create an array without specifying dtype.
# This will raise a ValueError in NumPy 1.24+.
try:
    arr = np.array(ragged)
except ValueError as e:
    print("ValueError caught:", e)

# Step 4: Create an object array explicitly (safe workaround).
arr_obj = np.array(ragged, dtype=object)
print("\nObject array:", arr_obj)
print("dtype:", arr_obj.dtype)
print("Shape:", arr_obj.shape)
```

**Expected Output:**

```
ValueError caught: setting an array element with a sequence. The requested array has an inhomogeneous shape after 1 dimensions. The detected shape was (2,) + inhomogeneous part.

Object array: [list([1, 2]) list([3, 4, 5])]
dtype: object
Shape: (2,)
```

**Why This Result Occurs:** NumPy cannot create a regular multidimensional array from a ragged sequence because the inner lists have different lengths. The `ValueError` enforces the rectangularity requirement. Specifying `dtype=object` creates an object array that stores each inner list as a Python object, preserving the ragged structure.

### Real-World Cases

- **Tabular Data:** A list of rows from a CSV file is a nested sequence that converts naturally to a 2-D array.
- **Batch Data:** A list of samples, each a list of features, becomes a 2-D feature matrix.
- **Image Data:** A 3-D nested sequence (height × width × channels) represents a color image.
- **Ragged Data:** Variable-length sequences (e.g., sentences of different lengths) require object arrays or specialized structures like `np.ragged` (experimental).

### References

- NumPy Array creation – https://numpy.org/doc/1.25/user/basics.creation.html
- NEP 34 – Disallow inferring dtype=object from sequences – https://numpy.org/neps/nep-0034.html
- NumPy array() – https://numpy.org/doc/stable/reference/generated/numpy.array.html

---

## Core Concept 4: Constructor Functions (zeros, ones, empty, full)

### Definitions

**Core Definition:** Constructor functions create arrays of a specified shape and dtype without requiring an input sequence. They fill the array with zeros, ones, uninitialized values, or a specified constant.

**Technical Definition:** `np.zeros(shape)`, `np.ones(shape)`, `np.empty(shape)`, and `np.full(shape, fill_value)` allocate new memory for an array of the given shape and return it. `empty` does not initialize the memory, making it the fastest; the others initialize every element.

**Beginner-Friendly Explanation:** These are like ordering pre-made containers. `zeros` gives you a box full of 0s, `ones` gives you a box full of 1s, `empty` gives you an empty box (which might contain old data), and `full` lets you choose what to fill the box with. They are the fastest way to create an array when you know its size but not its values yet.

### Purposes

- To pre-allocate arrays for computations where the output values will be filled in later.
- To create mask arrays (all ones or all zeros) for boolean indexing.
- To initialize weight matrices in machine learning with zeros or constants.
- To create buffers for image processing operations.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.zeros(shape, dtype=float, order='C')
np.ones(shape, dtype=float, order='C')
np.empty(shape, dtype=float, order='C')
np.full(shape, fill_value, dtype=None, order='C')
```

**Component Breakdown:**

| Function | Description |
|----------|-------------|
| `shape` | An integer or tuple of integers specifying dimensions. |
| `dtype` | Data type (default `float64` for `zeros`/`ones`/`empty`; inferred for `full`). |
| `fill_value` | The scalar value to fill the array with (`full` only). |
| `order` | `'C'` (row-major) or `'F'` (column-major). |

**Syntax Rules:**

- `shape` can be an integer (for 1-D) or a tuple (for N-D).
- `dtype` defaults to `float64` unless specified.
- `empty` is the fastest because it does not initialize memory.

**Constraints and Limitations:**

- `empty` returns uninitialized memory; the values are arbitrary and may vary between runs.
- All functions allocate new memory; they do not modify existing arrays.
- `full` requires a `fill_value` argument.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Using All Four Constructors**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create a 2x3 array of zeros.
zeros_arr = np.zeros((2, 3))
print("zeros:\n", zeros_arr)
print()

# Step 3: Create a 2x3 array of ones.
ones_arr = np.ones((2, 3))
print("ones:\n", ones_arr)
print()

# Step 4: Create an empty 2x3 array (uninitialized).
empty_arr = np.empty((2, 3))
print("empty (values are arbitrary):\n", empty_arr)
print()

# Step 5: Create a 2x3 array filled with a constant value.
full_arr = np.full((2, 3), 7.5)
print("full with 7.5:\n", full_arr)
```

**Expected Output (empty values may vary):**

```
zeros:
 [[0. 0. 0.]
 [0. 0. 0.]]

ones:
 [[1. 1. 1.]
 [1. 1. 1.]]

empty (values are arbitrary):
 [[1. 2. 3.]
 [4. 5. 6.]]

full with 7.5:
 [[7.5 7.5 7.5]
 [7.5 7.5 7.5]]
```

**Why This Result Occurs:** `zeros` and `ones` explicitly initialize every element. `empty` allocates memory but does not write to it, so the displayed values are whatever happened to be in that memory location before allocation. `full` fills every element with the specified scalar.

### Real-World Cases

- **Neural Network Weights:** `np.zeros((n_inputs, n_neurons))` initializes a weight matrix before training.
- **Image Buffers:** `np.empty((height, width, 3))` allocates a buffer for image processing operations.
- **Mask Arrays:** `np.ones((height, width), dtype=bool)` creates a mask for selecting all pixels.
- **Constant Fields:** `np.full((100, 100), 273.15)` creates a temperature field initialized to 0°C in Kelvin.

### References

- NumPy Array creation routines – https://numpy.org/doc/2.5/reference/routines.array-creation.html
- NumPy zeros – https://numpy.org/doc/stable/reference/generated/numpy.zeros.html
- NumPy ones – https://numpy.org/doc/stable/reference/generated/numpy.ones.html
- NumPy empty – https://numpy.org/doc/stable/reference/generated/numpy.empty.html
- NumPy full – https://numpy.org/doc/stable/reference/generated/numpy.full.html

---

## Core Concept 5: Numeric Sequences (arange, linspace, logspace, geomspace)

### Definitions

**Core Definition:** Numeric sequence functions generate arrays of evenly spaced values over a specified interval, using either a step size (`arange`) or a fixed number of samples (`linspace`, `logspace`, `geomspace`).

**Technical Definition:** `np.arange([start,] stop[, step])` returns evenly spaced values within the half-open interval `[start, stop)`. `np.linspace(start, stop, num)` returns `num` evenly spaced samples over the closed interval `[start, stop]` (by default). `np.logspace(start, stop, num, base)` returns samples spaced evenly on a log scale. `np.geomspace(start, stop, num)` returns samples spaced evenly on a geometric progression.

**Beginner-Friendly Explanation:** These functions are like rulers with different tick-mark rules. `arange` says "start here, stop there, and jump by this much." `linspace` says "start here, end there, and give me exactly this many points evenly spread." `logspace` and `geomspace` do the same but on logarithmic or geometric scales, which is useful when your data spans many orders of magnitude.

### Purposes

- To generate coordinate arrays for plotting and simulation.
- To create sampling grids for numerical integration and interpolation.
- To generate frequency bands for signal processing.
- To create exponentially spaced values for logarithmic plots.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.arange([start,] stop[, step], dtype=None)
np.linspace(start, stop, num=50, endpoint=True, retstep=False, dtype=None)
np.logspace(start, stop, num=50, endpoint=True, base=10.0, dtype=None)
np.geomspace(start, stop, num=50, endpoint=True, dtype=None)
```

**Component Breakdown:**

| Function | Key Parameters |
|----------|---------------|
| `arange` | `start` (default 0), `stop` (exclusive), `step` (default 1). |
| `linspace` | `start`, `stop` (inclusive if `endpoint=True`), `num` (default 50). |
| `logspace` | `start` = base^start, `stop` = base^stop, `base` (default 10). |
| `geomspace` | `start`, `stop` (must be positive), `num` samples. |

**Syntax Rules:**

- `arange` excludes the endpoint; `linspace` includes it by default.
- Use `arange` for integer steps; use `linspace` for non-integer steps or when you need a specific number of points.
- `geomspace` requires positive start and stop values.

**Constraints and Limitations:**

- Floating-point inaccuracies can make `arange` results with floating-point numbers confusing; `linspace` is preferred in those cases.
- `logspace` and `geomspace` cannot include zero or negative values.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Comparing arange and linspace**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: arange with integer step.
arr_arange = np.arange(0, 10, 2)
print("arange(0, 10, 2):", arr_arange)
print("  Number of elements:", arr_arange.size)
print()

# Step 3: linspace with a specified number of points.
arr_linspace = np.linspace(0, 10, num=6)
print("linspace(0, 10, num=6):", arr_linspace)
print("  Number of elements:", arr_linspace.size)
print()

# Step 4: linspace excluding the endpoint.
arr_no_endpoint = np.linspace(0, 10, num=5, endpoint=False)
print("linspace(0, 10, num=5, endpoint=False):", arr_no_endpoint)
```

**Expected Output:**

```
arange(0, 10, 2): [0 2 4 6 8]
  Number of elements: 5

linspace(0, 10, num=6): [ 0.  2.  4.  6.  8. 10.]
  Number of elements: 6

linspace(0, 10, num=5, endpoint=False): [0. 2. 4. 6. 8.]
```

**Why This Result Occurs:** `arange(0, 10, 2)` starts at 0, stops before 10, and steps by 2, producing 5 elements. `linspace(0, 10, num=6)` divides the interval `[0, 10]` into 5 equal subintervals, producing 6 points that include both endpoints. Setting `endpoint=False` excludes the final point, so the interval is divided into 5 equal parts and only the first 5 points are returned.

**Example 2: Logarithmic and Geometric Sequences**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: logspace with base 10 (default).
log_arr = np.logspace(0, 3, num=4)
print("logspace(0, 3, num=4):", log_arr)
print("  (10^0, 10^1, 10^2, 10^3)")
print()

# Step 3: geomspace with explicit values.
geo_arr = np.geomspace(1, 1000, num=4)
print("geomspace(1, 1000, num=4):", geo_arr)
print("  (1, 10, 100, 1000)")
```

**Expected Output:**

```
logspace(0, 3, num=4): [   1.   10.  100. 1000.]
  (10^0, 10^1, 10^2, 10^3)

geomspace(1, 1000, num=4): [   1.   10.  100. 1000.]
  (1, 10, 100, 1000)
```

**Why This Result Occurs:** `logspace(0, 3, num=4)` returns `base**start` through `base**stop` with 4 samples, i.e., 10⁰=1, 10¹=10, 10²=100, 10³=1000. `geomspace(1, 1000, num=4)` produces values that form a geometric progression from 1 to 1000 with a common ratio of 10, yielding the same result.

### Real-World Cases

- **Signal Processing:** `np.linspace(0, 1, 1000)` generates a time axis for a 1000-sample signal.
- **Frequency Analysis:** `np.logspace(0, 4, num=100)` generates frequency bins for a logarithmic spectrogram.
- **Physics Simulations:** `np.arange(0, 10, 0.01)` creates a spatial grid with 0.01 resolution.
- **Machine Learning:** `np.linspace(0, 1, num=10)` generates learning rate schedules.

### References

- NumPy arange – https://numpy.org/doc/stable/reference/generated/numpy.arange.html
- NumPy linspace – https://numpy.org/doc/stable/reference/generated/numpy.linspace.html
- NumPy logspace – https://numpy.org/doc/stable/reference/generated/numpy.logspace.html
- NumPy geomspace – https://numpy.org/doc/stable/reference/generated/numpy.geomspace.html
- How to create arrays with regularly-spaced values – https://numpy.org/doc/stable/user/how-to-partition.html

---

## Core Concept 6: Identity Matrices (eye, identity)

### Definitions

**Core Definition:** `np.eye(N)` creates a 2-D square array with ones on the main diagonal and zeros elsewhere. `np.identity(n)` creates the same array but is restricted to square matrices with the main diagonal only.

**Technical Definition:** `np.eye(N, M=None, k=0, dtype=float)` returns a 2-D array with ones on the k-th diagonal (default 0, the main diagonal) and zeros elsewhere. `np.identity(n, dtype=None)` returns a square n×n array with ones on the main diagonal. Internally, `identity` calls `eye` with the appropriate parameters.

**Beginner-Friendly Explanation:** An identity matrix is like a multiplication "do-nothing" matrix. When you multiply any matrix by the identity matrix, you get the original matrix back. It's a square grid with 1s running diagonally from the top-left to the bottom-right, and 0s everywhere else.

### Purposes

- To create the multiplicative identity matrix for linear algebra operations.
- To initialize transformation matrices in computer graphics and robotics.
- To create one-hot encoding matrices for categorical data.
- To verify matrix inversion and eigenvalue routines.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.eye(N, M=None, k=0, dtype=float, order='C')
np.identity(n, dtype=None)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `N` | Number of rows. |
| `M` | Number of columns (defaults to `N`). |
| `k` | Diagonal offset: 0 (main), positive (upper), negative (lower). |
| `dtype` | Data type (default `float64`). |
| `n` | Size of the square matrix (`identity` only). |

**Syntax Rules:**

- `eye` can create non-square matrices by specifying different `N` and `M`.
- `identity` always creates a square matrix.
- The `k` parameter in `eye` allows creating matrices with ones on a shifted diagonal.

**Constraints and Limitations:**

- `identity` is essentially a convenience wrapper around `eye` with no additional functionality.
- Both functions default to `float64` dtype, which may need to be changed for integer or boolean identity matrices.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Creating Identity and Eye Matrices**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create a 3x3 identity matrix.
identity_3 = np.identity(3)
print("identity(3):\n", identity_3)
print()

# Step 3: Create a 3x4 eye matrix (non-square).
eye_3x4 = np.eye(3, 4)
print("eye(3, 4):\n", eye_3x4)
print()

# Step 4: Create an eye matrix with a shifted diagonal (k=1).
eye_shifted = np.eye(4, k=1)
print("eye(4, k=1):\n", eye_shifted)
```

**Expected Output:**

```
identity(3):
 [[1. 0. 0.]
 [0. 1. 0.]
 [0. 0. 1.]]

eye(3, 4):
 [[1. 0. 0. 0.]
 [0. 1. 0. 0.]
 [0. 0. 1. 0.]]

eye(4, k=1):
 [[0. 1. 0. 0.]
 [0. 0. 1. 0.]
 [0. 0. 0. 1.]
 [0. 0. 0. 0.]]
```

**Why This Result Occurs:** `identity(3)` creates a square 3×3 matrix with ones on the main diagonal. `eye(3, 4)` creates a 3×4 matrix because the row count (3) and column count (4) are specified separately. `eye(4, k=1)` creates a 4×4 matrix with ones on the first upper diagonal (offset 1 from the main diagonal).

### Real-World Cases

- **Linear Algebra:** Verifying that `A @ inv(A)` is close to the identity matrix.
- **Computer Graphics:** Initializing transformation matrices for 3D rendering pipelines.
- **Machine Learning:** Creating one-hot encoded targets for classification tasks.
- **Control Systems:** Representing the identity matrix in state-space models.

### References

- NumPy eye – https://numpy.org/doc/stable/reference/generated/numpy.eye.html
- NumPy identity – https://numpy.org/doc/stable/reference/generated/numpy.identity.html
- NumPy Array creation routines – https://numpy.org/doc/2.5/reference/routines.array-creation.html

---

## Core Concept 7: Diagonal Arrays (diag, diagflat)

### Definitions

**Core Definition:** `np.diag()` extracts or constructs a diagonal matrix, while `np.diagflat()` flattens the input and uses it to construct a 2-D array with the flattened input on the diagonal.

**Technical Definition:** `np.diag(v, k=0)` behaves differently depending on the input: if `v` is a 2-D array, it returns the k-th diagonal; if `v` is 1-D, it returns a 2-D array with `v` on the k-th diagonal. `np.diagflat(v, k=0)` first flattens `v` and then places the flattened values on the k-th diagonal of a square matrix.

**Beginner-Friendly Explanation:** `diag` is a dual-purpose tool: give it a matrix and it pulls out the diagonal; give it a vector and it builds a diagonal matrix. `diagflat` is similar but always flattens the input first, so even if you give it a 2-D array, it turns it into a flat vector and then builds a diagonal matrix from it.

### Purposes

- To extract diagonal elements from a matrix for analysis.
- To construct diagonal matrices from vectors (e.g., for scaling operations).
- To create banded matrices for numerical solutions of differential equations.
- To build covariance matrices from variance vectors.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.diag(v, k=0)
np.diagflat(v, k=0)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `v` | Input array. If 2-D, `diag` extracts; if 1-D, `diag` constructs. |
| `k` | Diagonal offset: 0 (main), positive (upper), negative (lower). |

**Syntax Rules:**

- `diag` with a 2-D input returns a 1-D array (the diagonal).
- `diag` with a 1-D input returns a 2-D array (the diagonal matrix).
- `diagflat` always flattens the input before constructing the diagonal matrix.

**Constraints and Limitations:**

- `diag` requires at least a 1-D input; passing a 0-D scalar raises an error.
- `diagflat` on a 2-D input loses the 2-D structure before construction.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Extracting and Constructing Diagonals**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create a 3x3 matrix.
matrix = np.array([[1, 2, 3],
                   [4, 5, 6],
                   [7, 8, 9]])

# Step 3: Extract the main diagonal.
main_diag = np.diag(matrix)
print("Main diagonal:", main_diag)

# Step 4: Extract the upper diagonal (k=1).
upper_diag = np.diag(matrix, k=1)
print("Upper diagonal (k=1):", upper_diag)
print()

# Step 5: Construct a diagonal matrix from a vector.
vector = np.array([10, 20, 30])
diag_matrix = np.diag(vector)
print("Diagonal matrix from vector:\n", diag_matrix)
```

**Expected Output:**

```
Main diagonal: [1 5 9]
Upper diagonal (k=1): [2 6]

Diagonal matrix from vector:
 [[10  0  0]
 [ 0 20  0]
 [ 0  0 30]]
```

**Why This Result Occurs:** `np.diag(matrix)` extracts the elements at positions (0,0), (1,1), and (2,2) from the 2-D input. With `k=1`, it extracts elements at positions (0,1) and (1,2). When given a 1-D vector, `np.diag` constructs a square matrix with the vector's elements on the main diagonal and zeros elsewhere.

**Example 2: Using diagflat with Nested Input**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create a nested list (2-D structure).
nested = [[1, 2], [3, 4]]

# Step 3: Use diagflat to flatten and create a diagonal matrix.
diagflat_result = np.diagflat(nested)
print("diagflat([[1,2],[3,4]]):\n", diagflat_result)
```

**Expected Output:**

```
diagflat([[1,2],[3,4]]):
 [[1 0 0 0]
 [0 2 0 0]
 [0 0 3 0]
 [0 0 0 4]]
```

**Why This Result Occurs:** `diagflat` first flattens the input into `[1, 2, 3, 4]` and then creates a 4×4 matrix with these values on the main diagonal. Unlike `diag`, which would treat the 2-D input as a matrix and extract its diagonal, `diagflat` always flattens first.

### Real-World Cases

- **Covariance Matrices:** Building a diagonal covariance matrix from a vector of variances.
- **Scaling Operations:** Creating diagonal scaling matrices for linear transformations.
- **Finite Difference Methods:** Constructing banded matrices for solving differential equations.
- **Principal Component Analysis:** Extracting eigenvalues (diagonal elements) from a covariance matrix.

### References

- NumPy diag – https://numpy.org/doc/stable/reference/generated/numpy.diag.html
- NumPy diagflat – https://numpy.org/doc/stable/reference/generated/numpy.diagflat.html
- NumPy diagonal – https://numpy.org/doc/stable/reference/generated/numpy.diagonal.html

---

## Core Concept 8: Repeating Arrays (repeat, tile)

### Definitions

**Core Definition:** `np.repeat()` repeats individual elements of an array, while `np.tile()` repeats the entire array structure across specified dimensions.

**Technical Definition:** `np.repeat(a, repeats, axis=None)` replicates each element of `a` `repeats` times along the specified axis. `np.tile(a, reps)` constructs an array by repeating `a` the number of times given by `reps`, which can be a scalar or a tuple specifying repetitions along each axis.

**Beginner-Friendly Explanation:** `repeat` is like making photocopies of each individual item in a list. If you have `[1, 2, 3]` and repeat each element twice, you get `[1, 1, 2, 2, 3, 3]`. `tile` is like making copies of the entire list. If you tile `[1, 2, 3]` twice, you get `[1, 2, 3, 1, 2, 3]`.

### Purposes

- To upsample data by replicating elements or patterns.
- To create larger arrays from smaller building blocks.
- To construct periodic boundary conditions in simulations.
- To expand feature vectors for broadcasting operations.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.repeat(a, repeats, axis=None)
np.tile(a, reps)
```

**Component Breakdown:**

| Function | Key Parameters |
|----------|---------------|
| `repeat` | `a`: input array; `repeats`: number of repetitions per element; `axis`: axis along which to repeat (default flattens). |
| `tile` | `a`: input array; `reps`: number of repetitions (scalar or tuple). |

**Syntax Rules:**

- `repeat` with `axis=None` flattens the array before repeating.
- `tile` with a scalar `reps` repeats the array that many times along each axis.
- `tile` with a tuple `reps` specifies repetitions per dimension.

**Constraints and Limitations:**

- `repeat` and `tile` both return new arrays; they do not modify the input.
- `tile` can produce very large arrays if `reps` values are large.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Comparing repeat and tile**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create a base array.
arr = np.array([1, 2, 3])

# Step 3: Repeat each element 3 times.
repeated = np.repeat(arr, 3)
print("repeat(arr, 3):", repeated)

# Step 4: Tile the whole array 3 times.
tiled = np.tile(arr, 3)
print("tile(arr, 3):", tiled)
```

**Expected Output:**

```
repeat(arr, 3): [1 1 1 2 2 2 3 3 3]
tile(arr, 3): [1 2 3 1 2 3 1 2 3]
```

**Why This Result Occurs:** `repeat` replicates each element individually: the first element (1) appears three times, then the second element (2) appears three times, and so on. `tile` repeats the entire array as a block: the full sequence `[1, 2, 3]` appears three times in succession.

**Example 2: Multi-dimensional repeat and tile**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create a 2x2 matrix.
mat = np.array([[1, 2], [3, 4]])

# Step 3: Repeat along axis=0 (rows).
repeat_axis0 = np.repeat(mat, 2, axis=0)
print("repeat(axis=0):\n", repeat_axis0)
print()

# Step 4: Tile with different repetitions per axis.
tiled_2d = np.tile(mat, (2, 3))
print("tile((2, 3)):\n", tiled_2d)
```

**Expected Output:**

```
repeat(axis=0):
 [[1 2]
 [1 2]
 [3 4]
 [3 4]]

tile((2, 3)):
 [[1 2 1 2 1 2]
 [3 4 3 4 3 4]
 [1 2 1 2 1 2]
 [3 4 3 4 3 4]]
```

**Why This Result Occurs:** With `axis=0`, `repeat` replicates each row twice: row 0 `[1, 2]` becomes two rows, and row 1 `[3, 4]` becomes two rows. `tile` with `reps=(2, 3)` repeats the entire matrix 2 times vertically and 3 times horizontally, producing a 4×6 array.

### Real-World Cases

- **Data Augmentation:** Repeating training samples to balance class distributions.
- **Signal Processing:** Upsampling a signal by repeating each sample (zero-order hold).
- **Image Processing:** Tiling a small texture across a larger canvas.
- **Neural Networks:** Expanding feature maps via nearest-neighbor upsampling.

### References

- NumPy repeat – https://numpy.org/doc/stable/reference/generated/numpy.repeat.html
- NumPy tile – https://numpy.org/doc/stable/reference/generated/numpy.tile.html
- NumPy Array creation routines – https://numpy.org/doc/2.5/reference/routines.array-creation.html

---

## Core Concept 9: Shape-Matching Constructors (zeros_like, ones_like, empty_like, full_like)

### Definitions

**Core Definition:** The `*_like` functions create new arrays with the same shape and dtype as a given prototype array, filled with zeros, ones, uninitialized values, or a specified constant.

**Technical Definition:** `np.zeros_like(a)`, `np.ones_like(a)`, `np.empty_like(a)`, and `np.full_like(a, fill_value)` return new arrays with the same shape and dtype as `a`. They are convenience functions that avoid manually specifying the shape and dtype of an existing array.

**Beginner-Friendly Explanation:** If you already have an array and want another one just like it—same size, same type—but filled with different values, these functions do exactly that. You don't have to remember the shape or dtype; you just say "make me one like this."

### Purposes

- To allocate output arrays that match the shape and dtype of input arrays.
- To create masks and buffers with the same structure as existing data.
- To avoid errors from manually specifying mismatched shapes or dtypes.
- To initialize arrays for in-place operations on existing data.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
np.zeros_like(a, dtype=None, order='K')
np.ones_like(a, dtype=None, order='K')
np.empty_like(a, dtype=None, order='K')
np.full_like(a, fill_value, dtype=None, order='K')
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `a` | The prototype array whose shape and dtype are copied. |
| `dtype` | Optional override; if `None`, uses `a.dtype`. |
| `fill_value` | Scalar value to fill the array (`full_like` only). |
| `order` | Memory layout: `'K'` (keep), `'C'`, `'F'`. |

**Syntax Rules:**

- The shape of the new array always matches `a.shape`.
- The dtype defaults to `a.dtype` unless explicitly overridden.
- `full_like` requires a `fill_value` argument.

**Constraints and Limitations:**

- `empty_like` returns uninitialized memory, just like `empty`.
- These functions create new arrays; they do not modify `a`.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Using All Four *_like Functions**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create a prototype array.
prototype = np.array([[1, 2, 3], [4, 5, 6]], dtype=np.int32)
print("Prototype:\n", prototype)
print("Shape:", prototype.shape, "| dtype:", prototype.dtype)
print()

# Step 3: Create zeros_like.
zeros_arr = np.zeros_like(prototype)
print("zeros_like:\n", zeros_arr)
print()

# Step 4: Create ones_like.
ones_arr = np.ones_like(prototype)
print("ones_like:\n", ones_arr)
print()

# Step 5: Create full_like with a custom fill value.
full_arr = np.full_like(prototype, 99)
print("full_like with 99:\n", full_arr)
```

**Expected Output:**

```
Prototype:
 [[1 2 3]
 [4 5 6]]
Shape: (2, 3) | dtype: int32

zeros_like:
 [[0 0 0]
 [0 0 0]]

ones_like:
 [[1 1 1]
 [1 1 1]]

full_like with 99:
 [[99 99 99]
 [99 99 99]]
```

**Why This Result Occurs:** Each `*_like` function reads the `shape` and `dtype` of the prototype (`(2, 3)` and `int32`) and creates a new array with those attributes. The fill values differ based on the function used. Note that `full_like` preserves the `int32` dtype because `dtype=None` defaults to the prototype's dtype.

### Real-World Cases

- **Gradient Descent:** `np.zeros_like(weights)` initializes gradient accumulators with the same shape as the weight matrix.
- **Mask Creation:** `np.ones_like(image, dtype=bool)` creates a mask of the same shape as an image.
- **Output Buffers:** `np.empty_like(input_data)` allocates an output buffer for a computation that will fill it.
- **Bias Initialization:** `np.full_like(features, 0.01)` initializes biases with a small constant.

### References

- NumPy zeros_like – https://numpy.org/doc/stable/reference/generated/numpy.zeros_like.html
- NumPy ones_like – https://numpy.org/doc/stable/reference/generated/numpy.ones_like.html
- NumPy empty_like – https://numpy.org/doc/stable/reference/generated/numpy.empty_like.html
- NumPy full_like – https://numpy.org/doc/stable/reference/generated/numpy.full_like.html

---

## Core Concept 10: Modern Random Sampling (Generator Subsystem)

### Definitions

**Core Definition:** The modern NumPy random sampling subsystem uses a `Generator` object created via `np.random.default_rng()`, which provides methods for sampling from various probability distributions.

**Technical Definition:** Since NumPy 1.17.0, the `Generator` class has been the recommended interface for random number generation. `default_rng(seed)` returns a `Generator` instance that uses `PCG64` by default. Methods include `.random()`, `.normal()`, `.uniform()`, and `.integers()`. The legacy `RandomState` and global `np.random` functions remain available for backward compatibility but are not recommended for new code.

**Beginner-Friendly Explanation:** The old way of generating random numbers in NumPy used global functions like `np.random.rand()`. The new way uses a `Generator` object that you create once and then call methods on. This is better because it keeps your random number generation isolated and reproducible, and it uses a better underlying algorithm.

### Purposes

- To generate reproducible random samples for simulations and experiments.
- To create random initializations for machine learning models.
- To sample from specific probability distributions (uniform, normal, integer).
- To avoid the global state issues of the legacy random API.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
rng = np.random.default_rng(seed=None)

# Uniform floats in [0, 1)
samples = rng.random(size)

# Normal (Gaussian) distribution
samples = rng.normal(loc=0.0, scale=1.0, size=None)

# Uniform distribution over [low, high)
samples = rng.uniform(low=0.0, high=1.0, size=None)

# Random integers from low (inclusive) to high (exclusive)
samples = rng.integers(low, high=None, size=None, dtype=int)
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `.random(size)` | Uniform floats in `[0, 1)`. |
| `.normal(loc, scale, size)` | Normal distribution with mean `loc` and std `scale`. |
| `.uniform(low, high, size)` | Uniform distribution over `[low, high)`. |
| `.integers(low, high, size)` | Random integers from `low` to `high-1`. |

**Syntax Rules:**

- Always create a `Generator` with `default_rng()` before sampling.
- The `seed` argument controls reproducibility; use an integer for reproducible results.
- `.integers()` is the canonical method for random integers (replaces legacy `.randint()`).

**Constraints and Limitations:**

- The legacy `np.random.rand()`, `np.random.randn()`, and `np.random.randint()` are deprecated for new code.
- `Generator` methods are not thread-safe by default; use separate generators for parallel sampling.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Modern Random Sampling with Generator**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create a Generator with a fixed seed for reproducibility.
rng = np.random.default_rng(seed=42)

# Step 3: Generate 5 uniform floats in [0, 1).
uniform_floats = rng.random(5)
print("Uniform floats [0, 1):", uniform_floats)
print()

# Step 4: Generate 5 samples from a normal distribution.
normal_samples = rng.normal(loc=0.0, scale=1.0, size=5)
print("Normal samples (mean=0, std=1):", normal_samples)
print()

# Step 5: Generate 5 random integers between 1 and 100 (exclusive).
random_ints = rng.integers(low=1, high=100, size=5)
print("Random integers [1, 100):", random_ints)
print()

# Step 6: Generate 5 uniform samples over [10, 20).
uniform_range = rng.uniform(low=10.0, high=20.0, size=5)
print("Uniform [10, 20):", uniform_range)
```

**Expected Output:**

```
Uniform floats [0, 1): [0.77395605 0.43887844 0.85859792 0.69736803 0.09417735]

Normal samples (mean=0, std=1): [ 0.76103773  0.12167502  0.44386323  0.33367433  1.49407907]

Random integers [1, 100): [52 93 15 72 61]

Uniform [10, 20): [15.60141963 18.11353196 10.98815875 12.93234678 17.94749072]
```

**Why This Result Occurs:** The `Generator` object uses the PCG64 bit generator, which produces high-quality pseudo-random numbers. The `seed=42` ensures that the same sequence of numbers is produced every time the code is run. Each method transforms the underlying random bits according to the specified distribution: uniform for `.random()` and `.uniform()`, Gaussian for `.normal()`, and discrete uniform for `.integers()`.

### Real-World Cases

- **Model Initialization:** `rng.normal(0, 0.01, size=(100, 50))` initializes neural network weights.
- **Monte Carlo Simulations:** `rng.random(10000)` generates random points for numerical integration.
- **Data Augmentation:** `rng.integers(0, 256, size=(32, 32, 3))` generates random noise images.
- **Cross-Validation:** `rng.permutation(n)` shuffles dataset indices for train/test splits.

### References

- NumPy Random sampling – https://numpy.org/doc/stable/reference/random/index.html
- NumPy default_rng – https://numpy.org/doc/stable/reference/random/generator.html
- NEP 19 – Random Number Generator Policy – https://numpy.org/neps/nep-0019-rng-policy.html

---

## Summary of Deprecated and Version-Specific Features

| Feature | Status | Recommendation |
|---------|--------|----------------|
| Ragged array creation without `dtype=object` | Raises `ValueError` in NumPy 1.24+ | Use `dtype=object` or ensure rectangular structure |
| `np.random.rand()`, `np.random.randn()`, `np.random.randint()` | Legacy; not recommended for new code | Use `np.random.default_rng()` and `Generator` methods |
| `np.matrix` class | Should no longer be used | Use regular 2-D `ndarray` |
| Setting `arr.shape` directly | Deprecated | Use `arr.reshape()` |

---

## Consolidated Reference List

- NumPy Array creation – https://numpy.org/doc/1.25/user/basics.creation.html
- NumPy Array creation routines – https://numpy.org/doc/2.5/reference/routines.array-creation.html
- NumPy array() – https://numpy.org/doc/stable/reference/generated/numpy.array.html
- NumPy zeros – https://numpy.org/doc/stable/reference/generated/numpy.zeros.html
- NumPy ones – https://numpy.org/doc/stable/reference/generated/numpy.ones.html
- NumPy empty – https://numpy.org/doc/stable/reference/generated/numpy.empty.html
- NumPy full – https://numpy.org/doc/stable/reference/generated/numpy.full.html
- NumPy arange – https://numpy.org/doc/stable/reference/generated/numpy.arange.html
- NumPy linspace – https://numpy.org/doc/stable/reference/generated/numpy.linspace.html
- NumPy logspace – https://numpy.org/doc/stable/reference/generated/numpy.logspace.html
- NumPy geomspace – https://numpy.org/doc/stable/reference/generated/numpy.geomspace.html
- NumPy eye – https://numpy.org/doc/stable/reference/generated/numpy.eye.html
- NumPy identity – https://numpy.org/doc/stable/reference/generated/numpy.identity.html
- NumPy diag – https://numpy.org/doc/stable/reference/generated/numpy.diag.html
- NumPy diagflat – https://numpy.org/doc/stable/reference/generated/numpy.diagflat.html
- NumPy repeat – https://numpy.org/doc/stable/reference/generated/numpy.repeat.html
- NumPy tile – https://numpy.org/doc/stable/reference/generated/numpy.tile.html
- NumPy zeros_like – https://numpy.org/doc/stable/reference/generated/numpy.zeros_like.html
- NumPy ones_like – https://numpy.org/doc/stable/reference/generated/numpy.ones_like.html
- NumPy empty_like – https://numpy.org/doc/stable/reference/generated/numpy.empty_like.html
- NumPy full_like – https://numpy.org/doc/stable/reference/generated/numpy.full_like.html
- NumPy Random sampling – https://numpy.org/doc/stable/reference/random/index.html
- NumPy default_rng – https://numpy.org/doc/stable/reference/random/generator.html
- NEP 34 – Disallow inferring dtype=object from sequences – https://numpy.org/neps/nep-0034.html
- NEP 19 – Random Number Generator Policy – https://numpy.org/neps/nep-0019-rng-policy.html