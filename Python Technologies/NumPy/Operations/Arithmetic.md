# NumPy Element-Wise Arithmetic: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Element-wise arithmetic in NumPy refers to performing mathematical operations independently on each corresponding pair of elements across two arrays or between an array and a scalar, producing a result array of the same shape.

**Technical Definition:** Element-wise arithmetic is implemented through NumPy's universal functions (ufuncs), which are C-level functions that operate element by element on whole arrays. For each binary arithmetic operation, the arrays are broadcast to a common shape, the operation is applied to each pair of corresponding elements, and the result is returned as a new array (or written to a provided output array).

**Beginner-Friendly Explanation:** Element-wise arithmetic is like pairing up items from two lists and doing math on each pair separately. If you have one list of prices and another list of taxes, you add them element by element to get a list of final prices. NumPy does this for entire arrays at once without needing Python loops, making it extremely fast.

### Key Characteristics

- **Vectorized Execution:** Operations are performed in compiled C code, not Python loops, giving speedups of 100× or more over naive Python iteration.
- **Broadcasting:** Arrays of different shapes are automatically aligned using NumPy's broadcasting rules, allowing scalars and lower-dimensional arrays to be combined with larger arrays.
- **Ufunc Foundation:** Every arithmetic operator (`+`, `-`, `*`, `/`, `**`, `%`) has a corresponding ufunc (`np.add`, `np.subtract`, etc.) that can be called explicitly.
- **In-Place Variants:** Compound assignment operators (`+=`, `-=`, `*=`, `/=`) modify the left operand in place, avoiding new memory allocations.
- **Error Handling:** Division by zero and other numerical exceptions are handled by NumPy's floating-point error machinery, which can be configured via `np.seterr` or `np.errstate`.

### Prerequisites

- NumPy installed (`pip install numpy`).
- Basic Python syntax and understanding of variables.
- Familiarity with NumPy array creation and the `import numpy as np` convention.
- Conceptual understanding of broadcasting.

### Related Programming Areas

- **Data Science:** Computing derived features, normalizing data, and performing statistical transformations.
- **Machine Learning:** Computing gradients, applying activation functions, and updating weights.
- **Image Processing:** Adjusting brightness, contrast, and combining image channels.
- **Scientific Computing:** Solving numerical equations and simulating physical systems.

### Core Concepts / Features

The following operations are explored in detail: (1) Addition, (2) Subtraction, (3) Multiplication, (4) Division (true, floor), (5) Exponentiation, (6) Modulo, and (7) In-place operations.

---

## Core Concept 1: Addition

### Definitions

**Core Definition:** Addition computes the sum of each pair of corresponding elements in two arrays, or between an array and a scalar.

**Technical Definition:** `numpy.add(x1, x2)` is a ufunc that adds its arguments element-wise. It is equivalent to `x1 + x2` in terms of array broadcasting. The result is a new array whose shape is determined by broadcasting `x1.shape` and `x2.shape`. If both inputs are scalars, the result is a scalar.

**Beginner-Friendly Explanation:** Addition is the simplest element-wise operation—you just add numbers together. If you have an array of numbers and want to add 10 to each one, you use `arr + 10`. If you have two arrays of the same shape, you add them element by element.

### Purposes

- To combine two datasets element by element (e.g., adding bias terms to features).
- To shift all values in an array by a constant amount.
- To accumulate results across multiple arrays.
- To implement the forward pass of linear models where bias is added to each feature.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Using the + operator
result = array1 + array2
result = array + scalar

# Using the np.add ufunc
result = np.add(array1, array2, out=None, where=True)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `x1, x2` | Input arrays or scalars. Must be broadcastable to a common shape. |
| `out` | Optional output array. If provided, the result is stored here. |
| `where` | Boolean array selecting which elements to compute. |
| `casting` | Casting mode. Default `'same_kind'`. |

**Syntax Rules:**

- Shapes must be broadcastable: either equal, or one dimension is 1, or one array has fewer dimensions.
- The `+` operator and `np.add()` are functionally identical for ndarrays.
- The result dtype follows NumPy's promotion rules.

**Constraints and Limitations:**

- Arrays with incompatible shapes raise a `ValueError`.
- The result is always a new array unless `out` is provided.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Scalar and Array Addition**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create an array.
arr = np.array([1, 2, 3, 4, 5])
print("Original array:", arr)
print()

# Step 3: Add a scalar to every element.
result_scalar = arr + 10
print("arr + 10:", result_scalar)
print("  (each element increased by 10)")
print()

# Step 4: Add two arrays element-wise.
other = np.array([10, 20, 30, 40, 50])
result_array = arr + other
print("arr + other:", result_array)
print("  (1+10, 2+20, 3+30, 4+40, 5+50)")
print()

# Step 5: Use np.add explicitly.
result_ufunc = np.add(arr, other)
print("np.add(arr, other):", result_ufunc)
```

**Expected Output:**

```
Original array: [1 2 3 4 5]

arr + 10: [11 12 13 14 15]
  (each element increased by 10)

arr + other: [11 22 33 44 55]
  (1+10, 2+20, 3+30, 4+40, 5+50)

np.add(arr, other): [11 22 33 44 55]
```

**Why This Result Occurs:** Adding a scalar to an array broadcasts the scalar to every element. Adding two arrays of the same shape performs element-wise addition: the first element of `arr` (1) is added to the first element of `other` (10), and so on.

### Real-World Cases

- **Feature Engineering:** Adding a bias term to each sample in a dataset.
- **Image Processing:** Adding a constant to every pixel to adjust brightness.
- **Financial Analysis:** Adding transaction fees to each trade amount.

### References

- numpy.add – https://numpy.org/doc/2.0/reference/generated/numpy.add.html
- NumPy Arithmetic operations – https://numpy.org/doc/2.2/numpy-ref.pdf

---

## Core Concept 2: Subtraction

### Definitions

**Core Definition:** Subtraction computes the difference between each pair of corresponding elements in two arrays, or between an array and a scalar.

**Technical Definition:** `numpy.subtract(x1, x2)` is a ufunc that subtracts `x2` from `x1` element-wise. It is equivalent to `x1 - x2`. The result is a new array with the broadcast shape of the inputs.

**Beginner-Friendly Explanation:** Subtraction works like addition, but instead of adding, you take away. `arr - 5` subtracts 5 from every element. `arr1 - arr2` subtracts each element of `arr2` from the corresponding element of `arr1`.

### Purposes

- To compute differences between two datasets (e.g., before/after measurements).
- To center data by subtracting the mean.
- To compute residuals in regression analysis.
- To adjust values by removing a baseline.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
result = array1 - array2
result = array - scalar
result = np.subtract(array1, array2, out=None, where=True)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `x1` | Minuend (the array being subtracted from). |
| `x2` | Subtrahend (the array being subtracted). |
| `out` | Optional output array. |
| `where` | Boolean condition for selective computation. |

**Syntax Rules:**

- Subtraction is not commutative: `a - b` is not the same as `b - a`.
- Shapes must be broadcastable.
- The result dtype follows promotion rules.

**Constraints and Limitations:**

- Subtracting a larger array from a smaller one requires broadcasting compatibility.
- Subtracting nearly equal floating-point numbers can cause catastrophic cancellation.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Subtracting Arrays and Scalars**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create two arrays.
a = np.array([10, 20, 30, 40])
b = np.array([1, 2, 3, 4])
print("Array a:", a)
print("Array b:", b)
print()

# Step 3: Subtract b from a.
result = a - b
print("a - b:", result)
print("  (10-1, 20-2, 30-3, 40-4)")
print()

# Step 4: Subtract a scalar.
result_scalar = a - 5
print("a - 5:", result_scalar)
print()

# Step 5: Demonstrate non-commutativity.
print("b - a:", b - a)
print("  (different from a - b)")
```

**Expected Output:**

```
Array a: [10 20 30 40]
Array b: [1 2 3 4]

a - b: [ 9 18 27 36]
  (10-1, 20-2, 30-3, 40-4)

a - 5: [ 5 15 25 35]

b - a: [ -9 -18 -27 -36]
  (different from a - b)
```

**Why This Result Occurs:** `a - b` subtracts each element of `b` from the corresponding element of `a`. `b - a` subtracts in the opposite order, producing negative results. Scalar subtraction broadcasts the scalar to all elements.

### Real-World Cases

- **Signal Processing:** Computing the difference between a signal and its baseline.
- **Quality Control:** Comparing measurements before and after a treatment.
- **Finance:** Calculating profit as revenue minus costs.

### References

- NumPy Arithmetic operations – https://numpy.org/doc/2.2/numpy-ref.pdf
- numpy.subtract – https://numpy.org/doc/stable/reference/generated/numpy.subtract.html

---

## Core Concept 3: Multiplication

### Definitions

**Core Definition:** Element-wise multiplication computes the product of each pair of corresponding elements in two arrays, distinct from matrix multiplication.

**Technical Definition:** `numpy.multiply(x1, x2)` is a ufunc that multiplies its arguments element-wise. It is equivalent to `x1 * x2`. This is fundamentally different from matrix multiplication (`@` or `np.dot`), which performs a linear algebra operation.

**Beginner-Friendly Explanation:** Element-wise multiplication is like pairing up numbers from two lists and multiplying each pair. If you have a list of quantities and a list of prices, element-wise multiplication gives you the total cost for each item. Matrix multiplication is different—it combines rows and columns in a specific way.

### Purposes

- To scale each element of an array by a corresponding factor.
- To apply weights to features in a linear model.
- To compute element-wise products in signal processing (e.g., windowing).
- To implement Hadamard products in linear algebra.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
result = array1 * array2
result = array * scalar
result = np.multiply(array1, array2, out=None, where=True)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `x1, x2` | Input arrays or scalars. |
| `out` | Optional output array. |
| `where` | Boolean condition for selective computation. |

**Syntax Rules:**

- The `*` operator performs element-wise multiplication, not matrix multiplication.
- Use `@` or `np.dot()` for matrix multiplication.
- Shapes must be broadcastable.

**Constraints and Limitations:**

- Confusing `*` with matrix multiplication is a common source of bugs.
- Multiplying very large numbers can cause overflow.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Element-Wise vs. Matrix Multiplication**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create two 2x2 matrices.
A = np.array([[1, 2],
              [3, 4]])
B = np.array([[5, 6],
              [7, 8]])
print("Matrix A:\n", A)
print("Matrix B:\n", B)
print()

# Step 3: Element-wise multiplication.
elementwise = A * B
print("A * B (element-wise):\n", elementwise)
print("  (1*5, 2*6, 3*7, 4*8)")
print()

# Step 4: Matrix multiplication for comparison.
matrix_product = A @ B
print("A @ B (matrix multiplication):\n", matrix_product)
print("  (1*5+2*7, 1*6+2*8, 3*5+4*7, 3*6+4*8)")
print()

# Step 5: Scalar multiplication.
scalar_mult = A * 2
print("A * 2:\n", scalar_mult)
```

**Expected Output:**

```
Matrix A:
 [[1 2]
 [3 4]]
Matrix B:
 [[5 6]
 [7 8]]

A * B (element-wise):
 [[ 5 12]
 [21 32]]
  (1*5, 2*6, 3*7, 4*8)

A @ B (matrix multiplication):
 [[19 22]
 [43 50]]
  (1*5+2*7, 1*6+2*8, 3*5+4*7, 3*6+4*8)

A * 2:
 [[2 4]
 [6 8]]
```

**Why This Result Occurs:** `A * B` multiplies corresponding elements: `A[0,0]*B[0,0] = 1*5 = 5`, and so on. `A @ B` performs matrix multiplication: the (0,0) entry is `1*5 + 2*7 = 19`. Scalar multiplication broadcasts the scalar to every element.

### Real-World Cases

- **Feature Weighting:** Multiplying each feature by its corresponding weight.
- **Signal Windowing:** Applying a window function element-wise to a signal.
- **Image Blending:** Combining two images by multiplying corresponding pixels.

### References

- numpy.multiply – https://numpy.org/doc/stable/reference/generated/numpy.multiply.html
- NumPy Arithmetic operations – https://numpy.org/doc/2.2/numpy-ref.pdf

---

## Core Concept 4: Division

### Definitions

**Core Definition:** Division computes the quotient of each pair of corresponding elements. NumPy provides true division (`/`) and floor division (`//`).

**Technical Definition:** `numpy.divide(x1, x2)` (alias `true_divide`) returns the true division of the inputs, element-wise. `numpy.floor_divide(x1, x2)` returns the largest integer smaller than or equal to the division. In Python 3, `/` always performs true division; `//` performs floor division.

**Beginner-Friendly Explanation:** True division gives you the exact decimal result (e.g., `7 / 2 = 3.5`). Floor division gives you the largest whole number less than or equal to the result (e.g., `7 // 2 = 3`). Both operate element by element.

### Purposes

- To compute rates, ratios, and proportions.
- To normalize data by dividing by a sum or standard deviation.
- To implement integer binning using floor division.
- To distribute resources evenly across groups.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# True division
result = array1 / array2
result = np.divide(array1, array2)

# Floor division
result = array1 // array2
result = np.floor_divide(array1, array2)
```

**Component Breakdown:**

| Operator | Ufunc | Description |
|----------|-------|-------------|
| `/` | `np.divide` / `np.true_divide` | True division (returns float). |
| `//` | `np.floor_divide` | Floor division (returns integer or float). |

**Syntax Rules:**

- Division by zero produces `inf`, `-inf`, or `nan` for floats, and a warning (by default).
- Integer division by zero raises a `RuntimeWarning` and returns 0 (for integers).
- The `np.errstate` context manager controls error handling.

**Constraints and Limitations:**

- Division by zero is the most common source of numerical warnings.
- Floor division of negative numbers rounds toward negative infinity, not toward zero.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: True Division, Floor Division, and Division by Zero**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create arrays.
a = np.array([10, 20, 30, 40])
b = np.array([3, 4, 5, 6])
print("Array a:", a)
print("Array b:", b)
print()

# Step 3: True division.
true_div = a / b
print("a / b (true division):", true_div)
print()

# Step 4: Floor division.
floor_div = a // b
print("a // b (floor division):", floor_div)
print()

# Step 5: Division by zero with warning handling.
with np.errstate(divide='warn'):
    result = np.array([1.0, 2.0]) / np.array([0.0, 1.0])
    print("1/0 and 2/1:", result)
print("  (division by zero produces inf)")
```

**Expected Output:**

```
Array a: [10 20 30 40]
Array b: [3 4 5 6]

a / b (true division): [3.33333333 5.         6.         6.66666667]

a // b (floor division): [3 5 6 6]

1/0 and 2/1: [inf 2.]
  (division by zero produces inf)
```

**Why This Result Occurs:** `10 / 3 = 3.333...` (true division), while `10 // 3 = 3` (floor division). Dividing 1.0 by 0.0 produces `inf` with a `RuntimeWarning`. NumPy's error handling machinery manages this behavior and can be configured via `np.seterr` or `np.errstate`.

### Real-World Cases

- **Normalization:** Dividing each feature by its maximum value to scale to [0, 1].
- **Rate Calculation:** Computing per-capita metrics by dividing totals by population.
- **Integer Binning:** Using floor division to assign data points to bins.

### References

- Floating point error handling – https://numpy.org/doc/2.5/reference/routines.err.html
- numpy.seterr – https://numpy.org/doc/1.26/reference/generated/numpy.seterr.html
- Supported NumPy Functions – https://numpy.org/numpy-quaddtype/api/functions.html

---

## Core Concept 5: Exponentiation

### Definitions

**Core Definition:** Exponentiation raises each element of an array to a specified power, element-wise.

**Technical Definition:** `numpy.power(x1, x2)` raises each base in `x1` to the positionally-corresponding power in `x2`, element-wise. It is equivalent to `x1 ** x2`. An integer type raised to a negative integer power raises a `ValueError`. Negative values raised to a non-integral power return `nan` unless the dtype is complex.

**Beginner-Friendly Explanation:** Exponentiation is like taking powers: `arr ** 2` squares every element, `arr ** 0.5` takes the square root of every element. You can also raise each element to a different power by providing an array of exponents.

### Purposes

- To square or cube values for polynomial computations.
- To compute roots (square root, cube root) using fractional exponents.
- To implement power transformations for statistical normalization.
- To compute distances in Euclidean space.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
result = array ** power
result = np.power(base_array, exponent_array)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `x1` | Base array. |
| `x2` | Exponent array or scalar. Must be broadcastable with `x1`. |

**Syntax Rules:**

- Integer arrays raised to negative powers raise `ValueError`.
- Negative bases with fractional exponents produce `nan` (for real dtypes).
- Use `np.float_power` to promote integers to floats for power operations.

**Constraints and Limitations:**

- `0 ** 0` is defined as 1 in NumPy.
- Large exponents can cause overflow to `inf`.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Exponentiation with Scalars and Arrays**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create a base array.
x = np.array([1, 2, 3, 4, 5])
print("Base array:", x)
print()

# Step 3: Square each element.
squared = x ** 2
print("x ** 2:", squared)
print()

# Step 4: Compute square roots using fractional exponent.
sqrt_x = x ** 0.5
print("x ** 0.5:", sqrt_x)
print()

# Step 5: Raise to element-wise different powers.
exponents = np.array([1, 2, 3, 4, 5])
variable_power = np.power(x, exponents)
print("np.power(x, exponents):", variable_power)
print("  (1^1, 2^2, 3^3, 4^4, 5^5)")
```

**Expected Output:**

```
Base array: [1 2 3 4 5]

x ** 2: [ 1  4  9 16 25]

x ** 0.5: [1.         1.41421356 1.73205081 2.         2.23606798]

np.power(x, exponents): [   1    4   27  256 3125]
  (1^1, 2^2, 3^3, 4^4, 5^5)
```

**Why This Result Occurs:** `x ** 2` squares each element. `x ** 0.5` computes the square root. `np.power(x, exponents)` raises each element of `x` to the corresponding element of `exponents`: `1¹=1`, `2²=4`, `3³=27`, `4⁴=256`, `5⁵=3125`.

### Real-World Cases

- **Distance Calculations:** Computing squared distances in k-nearest neighbors.
- **Statistical Modeling:** Applying Box-Cox power transformations.
- **Physics:** Computing kinetic energy as `0.5 * m * v ** 2`.

### References

- numpy.power – https://numpy.org/doc/stable/reference/generated/numpy.power.html

---

## Core Concept 6: Modulo

### Definitions

**Core Definition:** Modulo computes the remainder of division for each pair of corresponding elements.

**Technical Definition:** `numpy.remainder(x1, x2)` (alias `np.mod`) returns the element-wise remainder of division. It is equivalent to the Python modulus operator `x1 % x2` and has the same sign as the divisor `x2`. It is the complement to `floor_divide`.

**Beginner-Friendly Explanation:** Modulo gives you the remainder after division. `7 % 3 = 1` because 7 divided by 3 is 2 with a remainder of 1. It's useful for checking divisibility, cycling through values, and wrapping around arrays.

### Purposes

- To check divisibility (e.g., finding even numbers).
- To implement cyclic indexing (wrapping around array bounds).
- To extract the fractional part of a number.
- To implement periodic boundary conditions in simulations.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
result = array1 % array2
result = np.remainder(array1, array2)
result = np.mod(array1, array2)
```

**Component Breakdown:**

| Function | Description |
|----------|-------------|
| `np.remainder` | Computes the element-wise remainder. |
| `np.mod` | Alias for `np.remainder`. |
| `np.fmod` | Similar, but sign follows the dividend (not the divisor). |

**Syntax Rules:**

- The result has the same sign as the divisor (`x2`).
- When `x2` is 0 and both inputs are integers, the result is 0.
- The `%` operator is shorthand for `np.remainder`.

**Constraints and Limitations:**

- `np.remainder` and `np.fmod` differ in sign behavior for negative operands.
- Floating-point modulo can produce unexpected results due to precision issues.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Modulo Operations**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create an array.
x = np.arange(10)
print("Array:", x)
print()

# Step 3: Compute modulo 3.
mod_3 = x % 3
print("x % 3:", mod_3)
print("  (remainders: 0,1,2,0,1,2,...)")
print()

# Step 4: Test divisibility (even numbers).
is_even = (x % 2 == 0)
print("x % 2 == 0 (even mask):", is_even)
print()

# Step 5: Demonstrate sign behavior with negative numbers.
neg = np.array([-7, -3, 3, 7])
print("neg % 3:", neg % 3)
print("  (result has same sign as divisor 3)")
print()

# Step 6: Compare with fmod.
print("np.fmod(neg, 3):", np.fmod(neg, 3))
print("  (result has same sign as dividend)")
```

**Expected Output:**

```
Array: [0 1 2 3 4 5 6 7 8 9]

x % 3: [0 1 2 0 1 2 0 1 2 0]
  (remainders: 0,1,2,0,1,2,...)

x % 2 == 0 (even mask): [ True False  True False  True False  True False  True False]

neg % 3: [2 0 0 1]
  (result has same sign as divisor 3)

np.fmod(neg, 3): [-1 -0  0  1]
  (result has same sign as dividend)
```

**Why This Result Occurs:** `x % 3` gives the remainder after dividing each element by 3. The even mask uses the modulo result. For negative numbers, `np.remainder` gives results with the same sign as the divisor (3), so `-7 % 3 = 2`. `np.fmod` gives results with the same sign as the dividend, so `fmod(-7, 3) = -1`.

### Real-World Cases

- **Cyclic Indexing:** Wrapping array indices using `% len(array)`.
- **Time Calculations:** Converting seconds to hours, minutes, and seconds.
- **Game Development:** Implementing wrap-around movement on a grid.

### References

- numpy.mod – https://numpy.org/doc/1.22/reference/generated/numpy.mod.html
- numpy.remainder – https://numpy.org/doc/stable/reference/generated/numpy.remainder.html

---

## Core Concept 7: In-Place Operations

### Definitions

**Core Definition:** In-place operations modify an array's data buffer directly using compound assignment operators (`+=`, `-=`, `*=`, `/=`, etc.), without creating a new array.

**Technical Definition:** In-place arithmetic is implemented through augmented assignment operators, which map to ufuncs with the `out` parameter set to the left operand. For example, `a += b` is equivalent to `np.add(a, b, out=a)`. This avoids allocating a new array, saving memory and time.

**Beginner-Friendly Explanation:** In-place operations are like editing a document directly instead of creating a new copy. `a += 5` adds 5 to every element of `a` and stores the result back in `a`. The original array is modified; no new array is created.

### Purposes

- To save memory by avoiding the allocation of new arrays.
- To improve performance by reducing memory bandwidth usage.
- To update array values incrementally in loops or iterative algorithms.
- To implement gradient descent weight updates efficiently.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
array += other
array -= other
array *= other
array /= other
array **= other
array %= other
array //= other
```

**Component Breakdown:**

| Operator | Ufunc Equivalent | Description |
|----------|-----------------|-------------|
| `+=` | `np.add(out=a)` | Add and assign. |
| `-=` | `np.subtract(out=a)` | Subtract and assign. |
| `*=` | `np.multiply(out=a)` | Multiply and assign. |
| `/=` | `np.divide(out=a)` | Divide and assign. |
| `**=` | `np.power(out=a)` | Power and assign. |
| `%=` | `np.remainder(out=a)` | Modulo and assign. |
| `//=` | `np.floor_divide(out=a)` | Floor divide and assign. |

**Syntax Rules:**

- The left operand is modified in place; its shape and dtype must remain unchanged.
- The operation may raise an error if the result cannot be cast to the left operand's dtype.
- In-place operations are faster and more memory-efficient than their out-of-place counterparts.

**Constraints and Limitations:**

- The dtype of the left operand must be able to hold the result (e.g., `int_array += float_array` raises an error).
- In-place operations on views modify the original array.
- Not all operations have in-place variants (e.g., there is no `@=` for all cases).

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: In-Place vs. Out-of-Place Operations**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create an array.
a = np.array([1, 2, 3, 4])
print("Original a:", a)
print()

# Step 3: Out-of-place addition (creates a new array).
b = a + 10
print("b = a + 10:")
print("  a:", a, "(unchanged)")
print("  b:", b, "(new array)")
print()

# Step 4: In-place addition (modifies a directly).
a += 10
print("After a += 10:")
print("  a:", a, "(modified in place)")
print()

# Step 5: In-place multiplication.
a *= 2
print("After a *= 2:", a)
print()

# Step 6: Demonstrate memory efficiency.
c = np.ones(1_000_000)
d = np.ones(1_000_000)
# Out-of-place: creates a new 8 MB array
e = c + d
# In-place: modifies c directly, no new array
c += d
print("Out-of-place result shape:", e.shape)
print("In-place result shape:", c.shape)
```

**Expected Output:**

```
Original a: [1 2 3 4]

b = a + 10:
  a: [1 2 3 4] (unchanged)
  b: [11 12 13 14] (new array)

After a += 10:
  a: [11 12 13 14] (modified in place)

After a *= 2: [22 24 26 28]

Out-of-place result shape: (1000000,)
In-place result shape: (1000000,)
```

**Why This Result Occurs:** `a + 10` creates a new array `b` without modifying `a`. `a += 10` modifies `a` directly. The memory efficiency example shows that in-place operations avoid allocating a new 8 MB array for a million-element float64 array.

### Real-World Cases

- **Gradient Descent:** `weights -= learning_rate * gradients` updates weights in place.
- **Image Processing:** `image *= 0.5` darkens an image in place.
- **Running Averages:** `running_sum += new_data` accumulates sums without extra memory.

### References

- numpy.ufunc – https://numpy.org/doc/2.3/reference/generated/numpy.ufunc.html
- In-place operations – https://numpy.org/doc/stable/user/basics.ufuncs.html

---

## Comparison of Arithmetic Operators and Ufuncs

| Operator | Ufunc | Description | In-Place |
|----------|-------|-------------|----------|
| `+` | `np.add` | Addition | `+=` |
| `-` | `np.subtract` | Subtraction | `-=` |
| `*` | `np.multiply` | Element-wise multiplication | `*=` |
| `/` | `np.divide` / `np.true_divide` | True division | `/=` |
| `//` | `np.floor_divide` | Floor division | `//=` |
| `**` | `np.power` | Exponentiation | `**=` |
| `%` | `np.remainder` / `np.mod` | Modulo | `%=` |
| `@` | `np.matmul` | Matrix multiplication | `@=` |

---

## Summary of Deprecated and Version-Specific Features

| Feature | Status | Recommendation |
|---------|--------|----------------|
| Python 2 division behavior | Removed in Python 3 | `/` always performs true division; use `//` for floor division |
| `np.divide` on integers (Python 2) | Behaved like floor division | In Python 3, it behaves like true division |
| `UPDATEIFCOPY` flag | Deprecated | Do not use in new code |
| Setting `arr.shape` directly | Deprecated | Use `arr.reshape()` |

---

## Consolidated Reference List

- numpy.add – https://numpy.org/doc/2.0/reference/generated/numpy.add.html
- numpy.subtract – https://numpy.org/doc/stable/reference/generated/numpy.subtract.html
- numpy.multiply – https://numpy.org/doc/stable/reference/generated/numpy.multiply.html
- numpy.divide – https://numpy.org/doc/stable/reference/generated/numpy.divide.html
- numpy.power – https://numpy.org/doc/stable/reference/generated/numpy.power.html
- numpy.mod – https://numpy.org/doc/1.22/reference/generated/numpy.mod.html
- numpy.remainder – https://numpy.org/doc/stable/reference/generated/numpy.remainder.html
- numpy.ufunc – https://numpy.org/doc/2.3/reference/generated/numpy.ufunc.html
- Broadcasting – https://numpy.org/devdocs/user/basics.broadcasting.html
- Floating point error handling – https://numpy.org/doc/2.5/reference/routines.err.html
- numpy.seterr – https://numpy.org/doc/1.26/reference/generated/numpy.seterr.html
- Supported NumPy Functions – https://numpy.org/numpy-quaddtype/api/functions.html
- NumPy Arithmetic operations – https://numpy.org/doc/2.2/numpy-ref.pdf