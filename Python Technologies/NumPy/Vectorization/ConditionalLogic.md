# NumPy Vectorized Conditional Logic: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Vectorized conditional logic in NumPy refers to the set of functions and techniques that apply conditional (if-else) selection across entire arrays in a single operation, without Python-level loops.

**Technical Definition:** NumPy implements vectorized conditional logic through functions like `np.where`, `np.select`, and `np.piecewise`, along with boolean masking and in-place assignment. These operations evaluate conditions element-wise across arrays, leveraging broadcasting and compiled C loops to perform selection and transformation at native speed. Unlike Python's ternary expressions or if-elif-else chains, which operate on scalars, these functions operate on entire arrays simultaneously.

**Beginner-Friendly Explanation:** Vectorized conditional logic lets you say "if this condition is true for an element, use this value; otherwise use that value" — but for millions of elements at once. Instead of writing a loop with if-else statements, you write a single expression that NumPy applies to every element in the array. This is faster, cleaner, and less error-prone than Python loops.

### Key Characteristics

- **Element-Wise Selection:** Conditions are evaluated for every element independently, and the corresponding value is selected.
- **Broadcasting Support:** Conditions and value arrays are broadcast to a common shape, allowing scalars and lower-dimensional arrays to be combined with larger arrays.
- **No Python Loops:** All selection and transformation happens in compiled C code, bypassing per-element Python overhead.
- **Multiple Functions:** `np.where` handles binary (two-way) selection; `np.select` handles multi-way selection; `np.piecewise` applies different functions to different regions.
- **Boolean Masking:** Boolean arrays can be used directly as indices for filtering and in-place modification.

### Prerequisites

- NumPy installed (`pip install numpy`).
- Basic Python syntax: variables, lists, and conditional statements.
- Familiarity with NumPy array creation, indexing, and the `import numpy as np` convention.
- Conceptual understanding of broadcasting and boolean arrays.

### Related Programming Areas

- **Data Cleaning:** Replacing missing or invalid values based on conditions.
- **Machine Learning:** Implementing activation functions (ReLU, sigmoid thresholds) and loss functions.
- **Image Processing:** Applying different transformations to different regions of an image.
- **Financial Modeling:** Computing tiered pricing, tax brackets, and conditional returns.

### Core Concepts / Features

The following core concepts are explored in detail: (1) `np.where()`, (2) Nested conditions, (3) Boolean masks, (4) Conditional transformations, (5) `np.select()`, and (6) `np.piecewise()`.

---

## Core Concept 1: np.where()

### Definitions

**Core Definition:** `np.where()` is a vectorized ternary operator that selects elements from one of two arrays based on a condition.

**Technical Definition:** `numpy.where(condition[, x, y])` returns elements chosen from `x` or `y` depending on `condition`. If only `condition` is given, it returns the indices where the condition is true. If both `x` and `y` are specified, the output array contains elements of `x` where `condition` is `True`, and elements from `y` elsewhere. The `condition`, `x`, and `y` must be broadcastable to a common shape.

**Beginner-Friendly Explanation:** `np.where` is like a vectorized if-else statement. For each element, it checks a condition: if true, it picks a value from one array; if false, it picks from another. It's the NumPy equivalent of Python's `x if condition else y`, but applied to whole arrays at once.

### Purposes

- To perform element-wise conditional selection between two arrays in a single vectorized operation.
- To implement thresholding, binary classification, and activation functions.
- To replace values that meet a condition without modifying the original array.
- To find the indices of elements that satisfy a condition.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
result = np.where(condition, x, y)
indices = np.where(condition)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `condition` | Boolean array. When `True`, yield `x`; otherwise yield `y`. |
| `x, y` | Values from which to choose. Must be broadcastable with `condition`. |
| Returns (3-arg) | Array with elements from `x` where `condition` is `True`, from `y` elsewhere. |
| Returns (1-arg) | Tuple of index arrays (equivalent to `condition.nonzero()`). |

**Syntax Rules:**

- `condition`, `x`, and `y` need to be broadcastable to some shape.
- Both `x` and `y` are evaluated before the condition is applied (not lazy).
- When called with only a condition, returns the indices where the condition is `True`.

**Constraints and Limitations:**

- Both branches (`x` and `y`) are evaluated eagerly, so expensive computations in unused branches still execute.
- For division, `np.where(denom != 0, num / denom, 0)` will still compute `num / denom` for all elements, including where `denom == 0`.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Basic np.where() Selection**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create an array of scores.
scores = np.array([45, 78, 92, 33, 67, 88, 51, 95])
print("Scores:", scores)
print()

# Step 3: Apply np.where to assign pass/fail labels.
labels = np.where(scores >= 60, "pass", "fail")
print("Labels:", labels)
print()

# Step 4: Use np.where to replace values.
adjusted = np.where(scores < 60, 60, scores)
print("Adjusted scores (minimum 60):", adjusted)
print()

# Step 5: Use single-argument np.where to find indices.
failing_indices = np.where(scores < 60)
print("Failing indices:", failing_indices)
print("Failing scores:", scores[failing_indices])
```

**Expected Output:**

```
Scores: [45 78 92 33 67 88 51 95]

Labels: ['fail' 'pass' 'pass' 'fail' 'pass' 'pass' 'fail' 'pass']

Adjusted scores (minimum 60): [60 78 92 60 67 88 60 95]

Failing indices: (array([0, 3, 6]),)
Failing scores: [45 33 51]
```

**Why This Result Occurs:** `np.where(scores >= 60, "pass", "fail")` evaluates the condition `scores >= 60` element-wise. Where `True` (scores 78, 92, 67, 88, 95), it selects "pass"; where `False` (45, 33, 51), it selects "fail". The single-argument form returns the indices where the condition is `True` as a tuple of arrays.

**Example 2: Broadcasting in np.where()**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create a 2-D array.
x = np.array([[1, 2, 3],
              [4, 5, 6]])
print("x:\n", x)
print()

# Step 3: Apply np.where with a scalar threshold.
result = np.where(x > 3, x * 10, x)
print("np.where(x > 3, x * 10, x):\n", result)
print()

# Step 4: Use broadcasting with different-shaped arrays.
row_val = np.array([[10], [20]])
result2 = np.where(x > 3, row_val, 0)
print("np.where(x > 3, row_val, 0):\n", result2)
```

**Expected Output:**

```
x:
 [[1 2 3]
 [4 5 6]]

np.where(x > 3, x * 10, x):
 [[ 1  2  3]
 [40 50 60]]

np.where(x > 3, row_val, 0):
 [[ 0  0  0]
 [20 20 20]]
```

**Why This Result Occurs:** The condition `x > 3` produces a boolean array. Where `True`, `x * 10` is selected; where `False`, `x` is retained. In the second example, `row_val` has shape `(2, 1)` and broadcasts against the `(2, 3)` condition, filling the selected positions with 20 for the second row.

### Real-World Cases

- **Machine Learning:** `np.where(logits >= 0.5, 1, 0)` for binary classification predictions.
- **ReLU Activation:** `np.where(x > 0, x, 0)` or `np.maximum(x, 0)` for neural network activation.
- **Data Cleaning:** `np.where(np.isnan(data), 0, data)` to replace NaN values with 0.

### References

- numpy.where – https://docs.scipy.org/doc/numpy-1.14.1/reference/generated/numpy.where.html
- How to Use Conditional Expressions With NumPy where() – https://realpython.com

---

## Core Concept 2: Nested Conditions

### Definitions

**Core Definition:** Nested conditions in NumPy involve embedding multiple `np.where()` calls inside one another to implement multi-way branching (if-elif-else logic).

**Technical Definition:** Because `np.where` only handles binary selection, multiple conditions can be chained by nesting `np.where` calls. Each inner `np.where` provides the value for one branch of the outer condition. The result is equivalent to an if-elif-else chain, evaluated element-wise.

**Beginner-Friendly Explanation:** Nested conditions let you handle more than two outcomes. For example, "if the score is above 90, assign 'A'; else if it's above 80, assign 'B'; else assign 'C'." You do this by putting one `np.where` inside another.

### Purposes

- To implement multi-way branching (if-elif-else) in vectorized form.
- To assign different values based on multiple overlapping conditions.
- To handle tiered or graded classifications.
- To replace complex Python conditional logic with a single vectorized expression.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
result = np.where(condition1, value1,
         np.where(condition2, value2,
         np.where(condition3, value3, default)))
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `condition1` | First (highest-priority) condition. |
| `value1` | Value where `condition1` is `True`. |
| Inner `np.where` | Provides the fallback for `condition1` being `False`. |

**Syntax Rules:**

- The innermost `np.where` provides the default value for elements that satisfy no condition.
- Conditions are evaluated in order; the first `True` condition wins.
- Nesting more than 3 levels becomes difficult to read; use `np.select` instead.

**Constraints and Limitations:**

- Deeply nested `np.where` is hard to read and maintain.
- All branches are evaluated eagerly, so performance degrades with many nested levels.
- `np.select` is more efficient and readable for 3+ conditions.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Grade Assignment with Nested np.where**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create an array of scores.
scores = np.array([95, 82, 67, 45, 78, 91, 55])
print("Scores:", scores)
print()

# Step 3: Assign letter grades using nested np.where.
grades = np.where(scores >= 90, 'A',
         np.where(scores >= 80, 'B',
         np.where(scores >= 70, 'C',
         np.where(scores >= 60, 'D', 'F'))))
print("Grades:", grades)
print()

# Step 4: Assign grade points using nested np.where.
points = np.where(scores >= 90, 4.0,
         np.where(scores >= 80, 3.0,
         np.where(scores >= 70, 2.0,
         np.where(scores >= 60, 1.0, 0.0))))
print("Grade points:", points)
```

**Expected Output:**

```
Scores: [95 82 67 45 78 91 55]

Grades: ['A' 'B' 'D' 'F' 'C' 'A' 'F']

Grade points: [4. 3. 1. 0. 2. 4. 0.]
```

**Why This Result Occurs:** The outermost `np.where` checks `scores >= 90`. For scores that don't meet this (82, 67, 45, 78, 55), it falls through to the next `np.where`, which checks `scores >= 80`. This continues until the innermost `np.where` provides the default value ('F' or 0.0). The result is a complete if-elif-else evaluation for each element.

### Real-World Cases

- **Grading Systems:** Assigning letter grades based on score thresholds.
- **Pricing Tiers:** Computing prices based on quantity brackets.
- **Risk Assessment:** Classifying items into low/medium/high risk categories.

### References

- Nested use of numpy.where – https://stackoverflow.com
- np.select vs nested np.where – https://cloud.tencent.com

---

## Core Concept 3: Boolean Masks

### Definitions

**Core Definition:** Boolean masks are boolean arrays that can be used to select elements from an array (for reading) or assign new values to selected elements (for in-place modification).

**Technical Definition:** Boolean array indexing occurs when the index object is an array of Boolean type. The result is a 1-D array containing the elements of the original array corresponding to `True` values. For assignment, `array[mask] = value` modifies the original array in place, changing only the elements where `mask` is `True`.

**Beginner-Friendly Explanation:** A boolean mask is like a stencil. You create it by asking a question (e.g., "which numbers are negative?"), and NumPy gives you a True/False pattern. You can then use that stencil to pick out matching elements or to paint over them with new values.

### Purposes

- To filter array elements based on a condition.
- To modify specific elements of an array in place without creating a new array.
- To implement conditional transformations selectively.
- To combine multiple conditions for complex filtering.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Reading with a mask
selected = array[mask]

# Assignment with a mask
array[mask] = new_value
array[mask] += 1
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `mask` | Boolean array of the same shape as `array` (or broadcastable). |
| `array[mask]` | Returns a 1-D array of elements where `mask` is `True`. |
| `array[mask] = val` | Assigns `val` to elements where `mask` is `True`. |

**Syntax Rules:**

- The mask must be broadcastable to the shape of the array.
- Assignment always modifies the original array in place (unlike fancy indexing assignment).
- Chained indexing (e.g., `mask[data['col'] == i][2] = True`) creates a copy and does not modify the original; use a single indexing expression instead.

**Constraints and Limitations:**

- Boolean masks always produce a 1-D result when used for reading, regardless of the original array's dimensionality.
- For in-place modification, use `np.putmask(a, mask, values)` or direct boolean assignment.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Boolean Mask Filtering and Assignment**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create an array of temperatures.
temps = np.array([-5, 12, -3, 18, 25, -1, 30, 8])
print("Temperatures:", temps)
print()

# Step 3: Create a mask for below-freezing temperatures.
freezing = temps < 0
print("Freezing mask:", freezing)
print()

# Step 4: Read values using the mask.
freezing_temps = temps[freezing]
print("Freezing temperatures:", freezing_temps)
print()

# Step 5: Modify values in place using the mask.
temps[freezing] = 0
print("After setting freezing temps to 0:", temps)
print()

# Step 6: Apply conditional transformation with the mask.
temps[temps > 20] = temps[temps > 20] * 2
print("After doubling temps above 20:", temps)
```

**Expected Output:**

```
Temperatures: [-5 12 -3 18 25 -1 30  8]

Freezing mask: [ True False  True False False  True False False]

Freezing temperatures: [-5 -3 -1]

After setting freezing temps to 0: [ 0 12  0 18 25  0 30  8]

After doubling temps above 20: [ 0 12  0 18 50  0 60  8]
```

**Why This Result Occurs:** The mask `temps < 0` identifies freezing temperatures. Reading `temps[freezing]` returns the matching values. The assignment `temps[freezing] = 0` modifies the original array in place, setting those elements to 0. The transformation `temps[temps > 20] = temps[temps > 20] * 2` doubles only the values above 20.

### Real-World Cases

- **Image Processing:** Setting all pixels below a brightness threshold to black: `image[image < 50] = 0`.
- **Data Cleaning:** Replacing negative values with zero: `data[data < 0] = 0`.
- **Quality Control:** Flagging defective items: `products[products['defect'] == True]['status'] = 'rejected'` (with proper single indexing).

### References

- numpy boolean indexing assignment – https://numpy.org
- Modify numpy array section in-place using boolean indexing – https://stackoverflow.com

---

## Core Concept 4: Conditional Transformations

### Definitions

**Core Definition:** Conditional transformations apply mathematical changes selectively to specific subsets of an array that meet chosen criteria.

**Technical Definition:** Conditional transformations combine boolean masking with arithmetic operations to modify only the elements that satisfy a condition. This can be done in-place (`array[mask] *= factor`) or out-of-place (`result = np.where(mask, transformed, original)`). The key is that the transformation is applied only where the condition is true.

**Beginner-Friendly Explanation:** Conditional transformations let you say "apply this math only to the numbers that meet this condition." For example, "double all values greater than 100" or "take the square root of all positive numbers."

### Purposes

- To apply mathematical transformations selectively to subsets of data.
- To normalize or scale specific regions of an array.
- To implement piecewise mathematical functions.
- To handle outliers or special cases differently from the rest of the data.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# In-place transformation
array[mask] = operation(array[mask])

# Out-of-place with np.where
result = np.where(mask, operation(array), array)
```

**Component Breakdown:**

| Approach | Syntax | Memory |
|----------|--------|--------|
| In-place | `arr[mask] = arr[mask] * 2` | Modifies original |
| np.where | `np.where(mask, arr * 2, arr)` | Creates new array |

**Syntax Rules:**

- In-place transformations modify the original array and do not allocate a new array for the full result.
- `np.where` creates a new array but leaves the original unchanged.
- The operation must be applicable to the selected elements.

**Constraints and Limitations:**

- In-place transformations require the result to be castable to the array's dtype.
- Using `np.where` evaluates both branches, which may be inefficient for expensive operations.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Conditional Mathematical Transformation**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create an array of values.
data = np.array([-4, 9, -16, 25, -36, 49, -64, 81])
print("Original data:", data)
print()

# Step 3: Apply square root only to positive values.
result = np.where(data >= 0, np.sqrt(data), data)
print("np.where(data >= 0, np.sqrt(data), data):", result)
print("  (negative values unchanged)")
print()

# Step 4: In-place conditional transformation.
data2 = data.copy()
data2[data2 >= 0] = np.sqrt(data2[data2 >= 0])
print("In-place sqrt of positives:", data2)
print()

# Step 5: Conditional scaling.
scaled = np.where(data > 0, data * 0.1, data * 0.5)
print("Scaled (positive *0.1, negative *0.5):", scaled)
```

**Expected Output:**

```
Original data: [ -4   9 -16  25 -36  49 -64  81]

np.where(data >= 0, np.sqrt(data), data): [ -4.   3. -16.   5. -36.   7. -64.   9.]
  (negative values unchanged)

In-place sqrt of positives: [ -4.   3. -16.   5. -36.   7. -64.   9.]

Scaled (positive *0.1, negative *0.5): [ -2.    0.9  -8.    2.5 -18.    4.9 -32.    8.1]
```

**Why This Result Occurs:** `np.where(data >= 0, np.sqrt(data), data)` computes the square root of positive values (9→3, 25→5, 49→7, 81→9) and leaves negative values unchanged. The in-place version achieves the same result by modifying only the positive elements. The scaling example applies different factors to positive and negative elements.

### Real-World Cases

- **Financial Modeling:** Applying different tax rates to different income brackets.
- **Image Processing:** Gamma-correcting only the bright regions of an image.
- **Scientific Computing:** Applying different constitutive laws to different regions of a simulation.

### References

- How to modify an array using numpy.where() – https://stackoverflow.com
- NumPy Where: Conditional Array Operations Made Simple – https://docs.kanaries.net

---

## Core Concept 5: np.select()

### Definitions

**Core Definition:** `np.select()` performs multi-condition selection by matching an ordered list of conditions against a corresponding list of choices.

**Technical Definition:** `numpy.select(condlist, choicelist, default=0)` returns an array drawn from elements in `choicelist`, depending on conditions. The output at position `m` is the `m`-th element of the array in `choicelist` where the `m`-th element of the corresponding array in `condlist` is `True`. When multiple conditions are satisfied, the first one encountered in `condlist` is used.

**Beginner-Friendly Explanation:** `np.select` is like a vectorized if-elif-else chain. You provide a list of conditions and a list of corresponding values. For each element, NumPy checks the conditions in order and picks the value from the first condition that is true. If none are true, it uses the default value.

### Purposes

- To implement multi-way branching (if-elif-else) cleanly without deeply nested `np.where` calls.
- To assign values based on an ordered priority of conditions.
- To replace verbose nested conditional logic with a readable, declarative structure.
- To handle tiered pricing, grading, and classification tasks.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
result = np.select(condlist, choicelist, default=0)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `condlist` | List of boolean ndarrays. Conditions determine which choice is used. |
| `choicelist` | List of ndarrays from which output elements are taken. Same length as `condlist`. |
| `default` | Scalar inserted when all conditions evaluate to `False`. Default is 0. |

**Syntax Rules:**

- `condlist` and `choicelist` must have the same length.
- Conditions are evaluated in order; the first `True` condition wins.
- The default value applies to elements where no condition is `True`.

**Constraints and Limitations:**

- `np.select` is more efficient and readable than nested `np.where` for 3+ conditions.
- All conditions are evaluated (not short-circuited), which may be less efficient for expensive conditions.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Multi-Condition Selection with np.select()**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create an array of values.
x = np.arange(1, 11)
print("Array:", x)
print()

# Step 3: Define conditions and choices.
conditions = [x < 3, x < 6, x < 9]
choices = [x, x**2, x**3]
print("Conditions:")
print("  x < 3:", conditions[0])
print("  x < 6:", conditions[1])
print("  x < 9:", conditions[2])
print()

# Step 4: Apply np.select.
result = np.select(conditions, choices, default=0)
print("np.select result:", result)
print("  (x for x<3, x^2 for 3<=x<6, x^3 for 6<=x<9, 0 for x>=9)")
print()

# Step 5: Compare with nested np.where (less readable).
nested = np.where(x < 3, x,
         np.where(x < 6, x**2,
         np.where(x < 9, x**3, 0)))
print("Nested np.where result:", nested)
print("Results match:", np.array_equal(result, nested))
```

**Expected Output:**

```
Array: [ 1  2  3  4  5  6  7  8  9 10]

Conditions:
  x < 3: [ True  True False False False False False False False False]
  x < 6: [ True  True  True  True  True False False False False False]
  x < 9: [ True  True  True  True  True  True  True  True False False]

np.select result: [  1   2   9  16  25 216 343 512   0   0]
  (x for x<3, x^2 for 3<=x<6, x^3 for 6<=x<9, 0 for x>=9)

Nested np.where result: [  1   2   9  16  25 216 343 512   0   0]
Results match: True
```

**Why This Result Occurs:** `np.select` evaluates the conditions in order. For `x=1` and `x=2`, the first condition (`x < 3`) is `True`, so the value is `x`. For `x=3, 4, 5`, the first condition is `False`, but the second (`x < 6`) is `True`, so the value is `x²`. For `x=6, 7, 8`, the third condition (`x < 9`) is `True`, so the value is `x³`. For `x=9, 10`, no condition is `True`, so the default `0` is used.

### Real-World Cases

- **Grading Systems:** Assigning letter grades based on multiple score thresholds.
- **Pricing Tiers:** Computing prices based on quantity brackets.
- **Risk Classification:** Classifying data into multiple risk categories.

### References

- numpy.select – https://docs.scipy.org/doc/numpy-1.17.0/reference/generated/numpy.select.html
- Conditional Operations — np.where, np.select – https://github.com/ReddyBytes/Python-DSA-API-Mastery

---

## Core Concept 6: np.piecewise()

### Definitions

**Core Definition:** `np.piecewise()` evaluates a piecewise-defined function by applying different functions to different regions of an array based on conditions.

**Technical Definition:** `numpy.piecewise(x, condlist, funclist, *args, **kw)` evaluates a piecewise-defined function. Given a set of conditions and corresponding functions, evaluate each function on the input data wherever its condition is true. Each boolean array in `condlist` selects a piece of `x`, and `funclist[i](x)` is used as the output value where `condlist[i]` is `True`. If `len(funclist) == len(condlist) + 1`, the extra function is the default value used wherever all conditions are false.

**Beginner-Friendly Explanation:** `np.piecewise` lets you define a function that behaves differently in different regions. For example, "if x is negative, return -x; if x is positive, return x" defines the absolute value function. You provide the conditions and the functions to apply in each region, and NumPy evaluates them piece by piece.

### Purposes

- To evaluate mathematical piecewise functions across an array.
- To apply different mathematical operations to different regions of data.
- To implement functions like absolute value, sigma function, or custom piecewise formulas.
- To provide a more mathematical, function-oriented alternative to `np.select`.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
result = np.piecewise(x, condlist, funclist, *args, **kw)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `x` | Input ndarray or scalar. |
| `condlist` | List of boolean arrays or scalars, each corresponding to a function in `funclist`. |
| `funclist` | List of callables or scalars. Each function is evaluated over `x` where its condition is `True`. |
| `*args, **kw` | Additional arguments passed to the functions. |
| Returns | ndarray of the same shape and type as `x`. |

**Syntax Rules:**

- The length of `condlist` must correspond to that of `funclist`, unless one extra function is given (the default).
- If a scalar is provided instead of a callable, a constant function is assumed.
- Portions not covered by any condition have a default value of 0 (unless an extra default function is provided).

**Constraints and Limitations:**

- The functions in `funclist` should take a 1-D array as input and return a 1-D array or scalar.
- `np.piecewise` is less flexible than `np.select` for non-mathematical conditional assignment.
- The output is always the same shape and type as `x`.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Piecewise Function Evaluation**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create an input array.
x = np.linspace(-3, 3, 7)
print("Input x:", x)
print()

# Step 3: Define the sigma function: -1 for x < 0, +1 for x >= 0.
sigma = np.piecewise(x, [x < 0, x >= 0], [-1, 1])
print("Sigma function:", sigma)
print()

# Step 4: Define the absolute value function using lambdas.
abs_val = np.piecewise(x, [x < 0, x >= 0], [lambda x: -x, lambda x: x])
print("Absolute value:", abs_val)
print()

# Step 5: Define a custom piecewise function.
# f(x) = x^2 for x < 0, x for 0 <= x < 2, 5 for x >= 2
custom = np.piecewise(x, [x < 0, (x >= 0) & (x < 2), x >= 2],
                      [lambda x: x**2, lambda x: x, lambda x: 5])
print("Custom piecewise:", custom)
```

**Expected Output:**

```
Input x: [-3. -2. -1.  0.  1.  2.  3.]

Sigma function: [-1. -1. -1.  1.  1.  1.  1.]

Absolute value: [3. 2. 1. 0. 1. 2. 3.]

Custom piecewise: [9. 4. 1. 0. 1. 5. 5.]
```

**Why This Result Occurs:** For the sigma function, `x < 0` selects the first three elements (which get -1), and `x >= 0` selects the rest (which get 1). The absolute value function applies `-x` to negative values and `x` to non-negative values. The custom function applies `x²` for negative values, `x` for values in [0, 2), and `5` for values ≥ 2.

### Real-World Cases

- **Physics:** Defining potential energy functions with different forms in different regions.
- **Engineering:** Applying different stress-strain relationships in elastic and plastic regions.
- **Finance:** Implementing tiered tax brackets or progressive pricing models.

### References

- numpy.piecewise – https://docs.scipy.org/doc/numpy-1.14.2/reference/generated/numpy.piecewise.html
- How to write a function for numpy array input – https://stackoverflow.com

---

## Comparison Summary

| Function | Use Case | Conditions | Output |
|----------|----------|-----------|--------|
| `np.where` | Binary selection | 1 condition | 2 choices |
| Nested `np.where` | Multi-way selection | 2-3 conditions | 2+ choices |
| Boolean masks | Filtering & in-place modification | 1 condition | Modified array |
| `np.select` | Multi-way selection | Any number | Any number of choices |
| `np.piecewise` | Piecewise mathematical functions | Any number | Any number of functions |

---

## Summary of Deprecated and Version-Specific Features

| Feature | Status | Recommendation |
|---------|--------|----------------|
| Python `and`/`or` in conditions | Raises `ValueError` | Use `&`, `\|`, `~` |
| Chained boolean indexing for assignment | Creates copy, does not modify original | Use single indexing expression |
| `np.where` with single argument | Returns index tuple | Use `np.nonzero` for clarity |

---

## Consolidated Reference List

- numpy.where – https://docs.scipy.org/doc/numpy-1.14.1/reference/generated/numpy.where.html
- numpy.select – https://docs.scipy.org/doc/numpy-1.17.0/reference/generated/numpy.select.html
- numpy.piecewise – https://docs.scipy.org/doc/numpy-1.14.2/reference/generated/numpy.piecewise.html
- Conditional Operations — np.where, np.select – https://github.com/ReddyBytes/Python-DSA-API-Mastery
- How to Use Conditional Expressions With NumPy where() – https://realpython.com
- NumPy Where: Conditional Array Operations Made Simple – https://docs.kanaries.net
- Nested use of numpy.where – https://stackoverflow.com
- np.select vs nested np.where – https://cloud.tencent.com
- numpy boolean indexing assignment – https://numpy.org
- Modify numpy array section in-place using boolean indexing – https://stackoverflow.com
- How to modify an array using numpy.where() – https://stackoverflow.com
- How to write a function for numpy array input – https://stackoverflow.com