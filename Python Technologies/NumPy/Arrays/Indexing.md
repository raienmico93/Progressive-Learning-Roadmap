# NumPy Basic Indexing: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Basic indexing in NumPy refers to accessing individual elements, rows, columns, or subarrays of an `ndarray` using integer indices, slices, and tuples of indices, following Python's standard 0-based indexing conventions.

**Technical Definition:** Basic indexing occurs when the indexing object is an integer, a slice object (constructed by `start:stop:step` notation), an Ellipsis (`...`), or a tuple of these objects. Basic indexing always returns a **view** of the original array, meaning the returned array shares the same data buffer and any modifications affect both arrays.

**Beginner-Friendly Explanation:** Basic indexing is how you pick out pieces of a NumPy array—just like you would pick items out of a list. You can grab a single number, a whole row, a whole column, or a block of values. NumPy lets you do this with simple square brackets and coordinates. The key difference from Python lists is that NumPy lets you use a single pair of brackets with comma-separated indices for multiple dimensions, like `arr[1, 2]` instead of `arr[1][2]`.

### Key Characteristics

- **0-Based:** Indexing starts at 0; the first element is at index 0.
- **Negative Indices Supported:** `-1` refers to the last element, `-2` to the second-to-last, etc..
- **View Semantics:** Basic indexing returns views, not copies; modifying the view modifies the original array.
- **Multidimensional in One Bracket:** Use `arr[i, j]` instead of `arr[i][j]` for multidimensional arrays.
- **C-Order Memory:** The last index varies fastest in memory (row-major).

### Prerequisites

- NumPy installed (`pip install numpy`).
- Basic Python knowledge: lists, indexing, and slicing.
- Familiarity with the `import numpy as np` convention.

### Related Programming Areas

- **Data Analysis:** Extracting subsets of datasets for inspection or transformation.
- **Image Processing:** Accessing pixel values and regions of interest.
- **Machine Learning:** Selecting batches, features, or samples from data matrices.
- **Scientific Computing:** Retrieving specific measurements from simulation grids.

### Core Concepts / Features

The following core concepts are explored in detail: (1) Single-element indexing, (2) Positive indices, (3) Negative indices, (4) Multidimensional indexing, (5) Row selection, (6) Column selection, and (7) Tuple vs. chained indexing.

---

## Core Concept 1: Single-Element Indexing

### Definitions

**Core Definition:** Single-element indexing extracts a single scalar value from an array using one or more integer coordinates.

**Technical Definition:** For a 1-D array, `x[i]` returns the element at position `i`. For an N-D array, `x[i, j, k, ...]` returns the scalar at the specified coordinates. The return type is a NumPy scalar (e.g., `numpy.int64`, `numpy.float64`), not a Python scalar, though it behaves similarly.

**Beginner-Friendly Explanation:** Single-element indexing is like pointing to one specific cell in a spreadsheet and reading its value. You give the row and column (and depth, if applicable), and NumPy hands you back just that one number.

### Purposes

- To retrieve a specific value from an array for inspection or computation.
- To update a single element in place.
- To extract a scalar for use in conditional logic or as a function argument.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# 1-D array
value = array[index]

# N-D array
value = array[i, j, k, ...]
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `array` | Any NumPy `ndarray` object. |
| `index` | A non-negative integer (0 to size-1) or negative integer (−size to −1). |
| `i, j, k, ...` | Comma-separated integers, one per dimension. |

**Syntax Rules:**

- Indices must be integers; floats raise `IndexError`.
- The number of indices must not exceed the number of dimensions.
- Fewer indices than dimensions returns a subdimensional view (not a scalar).

**Constraints and Limitations:**

- Out-of-bounds indices raise `IndexError`.
- Single-element indexing on a non-contiguous array still works but may be slower.
- The returned scalar is a NumPy scalar, which may behave differently from Python scalars in some contexts (e.g., overflow).

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Single-Element Indexing in 1-D and 2-D Arrays**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create a 1-D array.
arr_1d = np.array([10, 20, 30, 40, 50])
print("1-D array:", arr_1d)

# Step 3: Access the first element (index 0).
print("First element (index 0):", arr_1d[0])

# Step 4: Access the third element (index 2).
print("Third element (index 2):", arr_1d[2])
print()

# Step 5: Create a 2-D array.
arr_2d = np.array([[1, 2, 3],
                   [4, 5, 6],
                   [7, 8, 9]])
print("2-D array:\n", arr_2d)

# Step 6: Access the element at row 1, column 2.
print("Element at (1, 2):", arr_2d[1, 2])

# Step 7: Access the element at row 0, column 0.
print("Element at (0, 0):", arr_2d[0, 0])
```

**Expected Output:**

```
1-D array: [10 20 30 40 50]
First element (index 0): 10
Third element (index 2): 30

2-D array:
 [[1 2 3]
 [4 5 6]
 [7 8 9]]
Element at (1, 2): 6
Element at (0, 0): 1
```

**Why This Result Occurs:** The 1-D array stores values in a flat sequence; index 0 is the first value (10), and index 2 is the third (30). The 2-D array is indexed by row then column: `arr_2d[1, 2]` selects row 1 (the second row) and column 2 (the third column), which contains 6.

### Real-World Cases

- **Reading Sensor Data:** Extract the temperature at a specific time step and sensor location.
- **Game Development:** Retrieve the tile type at a specific (x, y) coordinate on a map.
- **Financial Analysis:** Get the closing price of a stock on a specific day.

### References

- NumPy Indexing on ndarrays – https://numpy.org/doc/stable/user/basics.indexing.html
- NumPy Quickstart – https://numpy.org/doc/stable/user/quickstart.html

---

## Core Concept 2: Positive Indices

### Definitions

**Core Definition:** Positive indices count forward from the beginning of an axis, starting at 0 for the first element.

**Technical Definition:** A positive index `i` refers to the element at position `i` in the array's logical ordering along the specified axis. The valid range for a positive index is `0 <= i < n`, where `n` is the length of that axis.

**Beginner-Friendly Explanation:** Positive indexing is the "normal" way to count: first item is 0, second item is 1, third item is 2, and so on. It's like counting positions on a shelf from left to right.

### Purposes

- To access elements in their natural forward order.
- To iterate over arrays from the beginning.
- To select specific positions when the order is known.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
value = array[positive_integer]
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `positive_integer` | An integer `i` such that `0 <= i < n` (where `n` is the axis length). |

**Syntax Rules:**

- The first element is always at index 0.
- The last element is at index `n-1`.
- Positive indices are the default; no special syntax is required.

**Constraints and Limitations:**

- Using an index `>= n` raises `IndexError`.
- For large arrays, positive indexing is O(1) but still slower than Python list indexing.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Positive Indexing Across Array Types**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create a 1-D array.
arr = np.array([100, 200, 300, 400, 500])
print("Array:", arr)
print("Length:", len(arr))
print()

# Step 3: Access elements with positive indices.
print("Index 0:", arr[0])   # First element
print("Index 1:", arr[1])   # Second element
print("Index 4:", arr[4])   # Last element (n-1 = 4)
print()

# Step 4: Demonstrate out-of-bounds access.
try:
    print(arr[5])  # Index 5 is out of bounds for length 5
except IndexError as e:
    print("IndexError:", e)
```

**Expected Output:**

```
Array: [100 200 300 400 500]
Length: 5

Index 0: 100
Index 1: 200
Index 4: 500

IndexError: index 5 is out of bounds for axis 0 with size 5
```

**Why This Result Occurs:** The array has indices 0 through 4. Index 0 gives the first element (100), index 1 gives the second (200), and index 4 gives the last (500). Attempting index 5 raises an `IndexError` because the valid range is 0 to 4.

### Real-World Cases

- **Iterating Over Data:** Loop through array elements in order using `for i in range(len(arr))`.
- **Batch Processing:** Select the first N samples from a dataset.
- **Time Series:** Access the earliest measurements in a chronological sequence.

### References

- NumPy Indexing on ndarrays – https://numpy.org/doc/stable/user/basics.indexing.html
- NumPy Quickstart – https://numpy.org/doc/stable/user/quickstart.html

---

## Core Concept 3: Negative Indices

### Definitions

**Core Definition:** Negative indices count backward from the end of an axis, starting at −1 for the last element.

**Technical Definition:** A negative index `i` is interpreted as `n + i`, where `n` is the length of the axis. Thus, `-1` refers to the last element, `-2` to the second-to-last, and so on.

**Beginner-Friendly Explanation:** Negative indexing is a shortcut for counting from the end. Instead of writing `arr[len(arr)-1]` to get the last element, you can write `arr[-1]`. It's like saying "the last one" or "the second from the end."

### Purposes

- To access elements from the end of an array without knowing its length.
- To simplify code when working with the last N elements.
- To handle arrays of variable length uniformly.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
value = array[negative_integer]
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `negative_integer` | An integer `i` such that `-n <= i <= -1`, where `n` is the axis length. |

**Syntax Rules:**

- `-1` is the last element, `-2` is the second-to-last, etc.
- Negative indices are converted internally to `n + i`.
- Negative indices work identically to positive ones after conversion.

**Constraints and Limitations:**

- Using an index `< -n` raises `IndexError`.
- Negative indices cannot be used with certain advanced indexing constructs without care.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Negative Indexing in 1-D and 2-D Arrays**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create a 1-D array.
arr = np.array([10, 20, 30, 40, 50])
print("Array:", arr)
print("Length:", len(arr))
print()

# Step 3: Access elements with negative indices.
print("Last element (index -1):", arr[-1])      # 50
print("Second-to-last (index -2):", arr[-2])    # 40
print("First element (index -5):", arr[-5])     # 10
print()

# Step 4: Create a 2-D array and use negative indices.
arr_2d = np.array([[1, 2, 3],
                   [4, 5, 6],
                   [7, 8, 9]])
print("2-D array:\n", arr_2d)
print("Last row, last column ([-1, -1]):", arr_2d[-1, -1])  # 9
print("First row, last column ([0, -1]):", arr_2d[0, -1])   # 3
```

**Expected Output:**

```
Array: [10 20 30 40 50]
Length: 5

Last element (index -1): 50
Second-to-last (index -2): 40
First element (index -5): 10

2-D array:
 [[1 2 3]
 [4 5 6]
 [7 8 9]]
Last row, last column ([-1, -1]): 9
First row, last column ([0, -1]): 3
```

**Why This Result Occurs:** For a length-5 array, `-1` maps to `5 + (-1) = 4` (the last element), `-2` maps to `5 + (-2) = 3`, and `-5` maps to `5 + (-5) = 0` (the first element). For the 2-D array, `[-1, -1]` maps to `[2, 2]`, which is the bottom-right element (9).

### Real-World Cases

- **Data Validation:** Check the last record in a dataset with `data[-1]` without needing its length.
- **Time Series:** Access the most recent measurement using `series[-1]`.
- **Image Processing:** Access the bottom-right pixel with `image[-1, -1]`.

### References

- NumPy Indexing on ndarrays – https://numpy.org/doc/stable/user/basics.indexing.html
- NIELIT Access NumPy Array Elements – http://nielit.gov.in/gorakhpur/sites/default/files/Gorakhpur/ALevel_1_Python_02June2020_SS.pdf

---

## Core Concept 4: Multidimensional Indexing

### Definitions

**Core Definition:** Multidimensional indexing accesses elements in arrays with more than one dimension using comma-separated coordinate syntax within a single pair of square brackets.

**Technical Definition:** For an N-D array, `x[i, j, k, ...]` selects the element at coordinates `(i, j, k, ...)`. This is equivalent to `x[(i, j, k, ...)]`; the tuple is implicit. NumPy's `__getitem__` and `__setitem__` methods receive the comma-separated indices as a single tuple argument.

**Beginner-Friendly Explanation:** Instead of writing `arr[1][2]` to get an element from a 2-D array, you write `arr[1, 2]`. Both give the same result, but `arr[1, 2]` is one operation instead of two, making it faster and more efficient. Think of it as giving both coordinates at once rather than asking for a row first and then a column.

### Purposes

- To access elements in multidimensional arrays efficiently with a single indexing operation.
- To avoid creating unnecessary intermediate arrays.
- To enable simultaneous selection along multiple axes.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
value = array[i, j, k, ...]
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `i, j, k, ...` | Comma-separated integers, one per dimension. |
| The tuple | `(i, j, k, ...)` is passed internally as a single index object. |

**Syntax Rules:**

- The number of indices must not exceed `ndim`.
- Fewer indices return a subdimensional view.
- Negative indices work in any position.
- Slices (`:`) can be mixed with integers in the same bracket.

**Constraints and Limitations:**

- Using more indices than dimensions raises `IndexError`.
- `arr[i][j]` and `arr[i, j]` produce the same values, but `arr[i][j]` creates an intermediate array and is slower.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Multidimensional Indexing in 2-D and 3-D Arrays**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create a 3-D array (2 layers, 2 rows, 3 columns).
arr_3d = np.arange(24).reshape(2, 3, 4)
print("3-D array shape:", arr_3d.shape)
print("3-D array:\n", arr_3d)
print()

# Step 3: Access a specific element with three indices.
element = arr_3d[1, 2, 3]
print("Element at (1, 2, 3):", element)
print()

# Step 4: Access a subdimensional view by using fewer indices.
layer_1 = arr_3d[1]
print("Layer 1 (arr_3d[1]):\n", layer_1)
print("Shape of layer 1:", layer_1.shape)
print()

# Step 5: Access a row within a layer.
row = arr_3d[1, 2]
print("Row at (1, 2):", row)
print("Shape of row:", row.shape)
```

**Expected Output:**

```
3-D array shape: (2, 3, 4)
3-D array:
 [[[ 0  1  2  3]
  [ 4  5  6  7]
  [ 8  9 10 11]]

 [[12 13 14 15]
  [16 17 18 19]
  [20 21 22 23]]]

Element at (1, 2, 3): 23

Layer 1 (arr_3d[1]):
 [[12 13 14 15]
 [16 17 18 19]
 [20 21 22 23]]
Shape of layer 1: (3, 4)

Row at (1, 2): [20 21 22 23]
Shape of row: (4,)
```

**Why This Result Occurs:** The 3-D array has shape `(2, 3, 4)`. The index `[1, 2, 3]` selects layer 1, row 2, column 3, which contains 23. Using `[1]` alone returns the entire second layer (shape `(3, 4)`), and `[1, 2]` returns the third row of the second layer (shape `(4,)`).

### Real-World Cases

- **Image Processing:** Access a pixel at `(row, column, channel)` in a color image array.
- **Neural Networks:** Select a specific element from a 4-D tensor `(batch, height, width, channels)`.
- **Climate Data:** Access a temperature value at `(time, latitude, longitude)`.

### References

- NumPy Indexing on ndarrays – https://numpy.org/doc/stable/user/basics.indexing.html
- NumPy How to index ndarrays – https://numpy.org/doc/2.5/user/how-to-index.html

---

## Core Concept 5: Row Selection

### Definitions

**Core Definition:** Row selection extracts an entire horizontal subarray (a row) from a 2-D array by indexing along the first axis.

**Technical Definition:** For a 2-D array `arr` with shape `(m, n)`, `arr[i]` returns a 1-D view of row `i` with shape `(n,)`. This is because indexing with a single integer selects along axis 0 and leaves the remaining axes unspecified.

**Beginner-Friendly Explanation:** If your array is a table, selecting a row means picking one horizontal line of data. You specify just the row number, and NumPy gives you all the columns in that row as a 1-D array.

### Purposes

- To extract a single row of data for processing or analysis.
- To iterate over rows in a matrix.
- To select a subset of rows using slices (e.g., `arr[1:4]`).

### Syntax Rules and Structure

**Complete General Syntax:**

```python
row = array[row_index]
rows = array[start:stop:step]
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `row_index` | Integer index of the row (0-based). |
| `start:stop:step` | Slice notation to select multiple rows. |

**Syntax Rules:**

- `arr[i]` returns row `i` as a 1-D view.
- `arr[i:j]` returns rows `i` through `j-1` as a 2-D view.
- The returned row is a view; modifying it modifies the original array.

**Constraints and Limitations:**

- Row selection on a 1-D array returns a scalar, not an array.
- The returned view shares memory with the original; use `.copy()` for independence.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Selecting Rows from a 2-D Array**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create a 3x4 matrix.
matrix = np.array([[1,  2,  3,  4],
                   [5,  6,  7,  8],
                   [9, 10, 11, 12]])
print("Matrix:\n", matrix)
print()

# Step 3: Select the second row (index 1).
row_1 = matrix[1]
print("Row 1 (matrix[1]):", row_1)
print("Shape:", row_1.shape)
print()

# Step 4: Select the last row using negative indexing.
last_row = matrix[-1]
print("Last row (matrix[-1]):", last_row)
print()

# Step 5: Select multiple rows with a slice.
first_two = matrix[0:2]
print("First two rows (matrix[0:2]):\n", first_two)
print("Shape:", first_two.shape)
```

**Expected Output:**

```
Matrix:
 [[ 1  2  3  4]
 [ 5  6  7  8]
 [ 9 10 11 12]]

Row 1 (matrix[1]): [5 6 7 8]
Shape: (4,)

Last row (matrix[-1]): [ 9 10 11 12]

First two rows (matrix[0:2]):
 [[1 2 3 4]
 [5 6 7 8]]
Shape: (2, 4)
```

**Why This Result Occurs:** `matrix[1]` selects the second row (index 1) and returns a 1-D array of shape `(4,)`. `matrix[-1]` selects the last row. `matrix[0:2]` returns a 2-D view containing rows 0 and 1, preserving the column dimension.

### Real-World Cases

- **Dataset Inspection:** View the first few rows of a data matrix to understand its structure.
- **Row-wise Operations:** Apply a function to each row of a matrix.
- **Batch Selection:** Select a batch of samples from a feature matrix.

### References

- NumPy Indexing on ndarrays – https://numpy.org/doc/stable/user/basics.indexing.html
- NumPy How to index ndarrays – https://numpy.org/doc/2.5/user/how-to-index.html

---

## Core Concept 6: Column Selection

### Definitions

**Core Definition:** Column selection extracts a vertical subarray (a column) from a 2-D array by combining a full-slice (`:`) for the first axis with an integer index for the second axis.

**Technical Definition:** For a 2-D array `arr` with shape `(m, n)`, `arr[:, j]` returns a 1-D view of column `j` with shape `(m,)`. The colon `:` selects all rows, and the integer `j` selects the desired column.

**Beginner-Friendly Explanation:** To get a column from a table, you need to say "all rows, but only this column." In NumPy, you write `arr[:, j]`, where `:` means "all rows" and `j` is the column number. This gives you a vertical slice of the table.

### Purposes

- To extract a single column of data for analysis or processing.
- To select a feature from a feature matrix.
- To apply column-wise operations on a matrix.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
column = array[:, column_index]
columns = array[:, start:stop:step]
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `:` | Slice indicating "all rows" along axis 0. |
| `column_index` | Integer index of the column (0-based). |
| `start:stop:step` | Slice notation to select multiple columns. |

**Syntax Rules:**

- `arr[:, j]` returns column `j` as a 1-D view.
- `arr[:, j:k]` returns columns `j` through `k-1` as a 2-D view.
- The returned column is a view; modifying it modifies the original array.

**Constraints and Limitations:**

- Column selection on a 1-D array is not applicable (1-D arrays have no columns).
- The returned view is not C-contiguous, which may affect performance in some operations.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Selecting Columns from a 2-D Array**

```python
# Step 1: Import NumPy.
import numpy as np

# Step 2: Create a 3x4 matrix.
matrix = np.array([[1,  2,  3,  4],
                   [5,  6,  7,  8],
                   [9, 10, 11, 12]])
print("Matrix:\n", matrix)
print()

# Step 3: Select the second column (index 1).
col_1 = matrix[:, 1]
print("Column 1 (matrix[:, 1]):", col_1)
print("Shape:", col_1.shape)
print()

# Step 4: Select the last column using negative indexing.
last_col = matrix[:, -1]
print("Last column (matrix[:, -1]):", last_col)
print()

# Step 5: Select multiple columns with a slice.
first_two_cols = matrix[:, 0:2]
print("First two columns (matrix[:, 0:2]):\n", first_two_cols)
print("Shape:", first_two_cols.shape)
```

**Expected Output:**

```
Matrix:
 [[ 1  2  3  4]
 [ 5  6  7  8]
 [ 9 10 11 12]]

Column 1 (matrix[:, 1]): [ 2  6 10]
Shape: (3,)

Last column (matrix[:, -1]): [ 4  8 12]

First two columns (matrix[:, 0:2]):
 [[ 1  2]
 [ 5  6]
 [ 9 10]]
Shape: (3, 2)
```

**Why This Result Occurs:** `matrix[:, 1]` selects all rows (`:`) and column 1, returning a 1-D array of shape `(3,)` containing the values 2, 6, and 10. `matrix[:, -1]` selects the last column. `matrix[:, 0:2]` returns a 2-D view containing the first two columns.

### Real-World Cases

- **Feature Selection:** Extract a single feature column from a machine learning feature matrix.
- **Data Cleaning:** Isolate a column to check for missing values or outliers.
- **Time Series Analysis:** Extract a specific variable from a multivariate time series.

### References

- NumPy How to index ndarrays – https://numpy.org/doc/2.5/user/how-to-index.html
- NumPy Indexing on ndarrays – https://numpy.org/doc/stable/user/basics.indexing.html

---

## Core Concept 7: Tuple vs. Chained Indexing

### Definitions

**Core Definition:** Tuple indexing (`arr[i, j]`) performs a single indexing operation using a tuple of coordinates, while chained indexing (`arr[i][j]`) performs two sequential indexing operations, each creating an intermediate array.

**Technical Definition:** In tuple indexing, `arr[i, j]` passes the tuple `(i, j)` to `ndarray.__getitem__`, which directly computes the memory offset for the target element. In chained indexing, `arr[i]` first returns a view of row `i`, and then `[j]` indexes into that view. The intermediate view creation adds overhead.

**Beginner-Friendly Explanation:** `arr[i, j]` says "give me the element at row i, column j" in one step. `arr[i][j]` says "first give me row i, then give me element j from that row" in two steps. Both get you the same value, but the one-step version is faster because NumPy doesn't have to create an intermediate array.

### Purposes

- To understand the performance and memory implications of different indexing notations.
- To choose the most efficient indexing method for performance-critical code.
- To avoid subtle bugs that can arise from chained indexing (e.g., in assignment contexts).

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Tuple indexing (preferred)
value = array[i, j]

# Chained indexing (less efficient)
value = array[i][j]
```

**Component Breakdown:**

| Notation | Operations | Intermediate Array |
|----------|-----------|-------------------|
| `arr[i, j]` | 1 | No |
| `arr[i][j]` | 2 | Yes (row view) |

**Syntax Rules:**

- Both notations return the same value for basic indexing.
- `arr[i, j]` is syntactic sugar for `arr[(i, j)]`.
- Chained indexing creates an intermediate view after the first index.

**Constraints and Limitations:**

- Chained indexing is approximately 2× slower than tuple indexing.
- In assignment contexts, chained indexing can sometimes produce unexpected results (though basic indexing views behave predictably).
- The performance difference grows with array size and access frequency.

### Multiple Annotated Complete Step by Step Code Examples

**Example 1: Performance Comparison of Tuple vs. Chained Indexing**

```python
# Step 1: Import NumPy and timeit.
import numpy as np
import timeit

# Step 2: Create a 3x3 array.
x = np.array([[1, 2, 3],
              [4, 5, 6],
              [7, 8, 9]])

# Step 3: Define functions for each indexing style.
def tuple_index():
    """Access element using tuple indexing arr[i, j]."""
    return x[1, 2]

def chained_index():
    """Access element using chained indexing arr[i][j]."""
    return x[1][2]

# Step 4: Verify both return the same value.
print("Tuple indexing result:", tuple_index())
print("Chained indexing result:", chained_index())
print()

# Step 5: Time both approaches over 100,000 iterations.
tuple_time = timeit.timeit(tuple_index, number=100_000)
chained_time = timeit.timeit(chained_index, number=100_000)

print(f"Tuple indexing time (100k iters):   {tuple_time:.4f} seconds")
print(f"Chained indexing time (100k iters): {chained_time:.4f} seconds")
print(f"Chained indexing is {chained_time / tuple_time:.2f}x slower")
```

**Expected Output (approximate; times vary by machine):**

```
Tuple indexing result: 6
Chained indexing result: 6

Tuple indexing time (100k iters):   0.0456 seconds
Chained indexing time (100k iters): 0.0891 seconds
Chained indexing is 1.95x slower
```

**Why This Result Occurs:** Both indexing styles return the same value (6), but tuple indexing passes both coordinates in a single operation, while chained indexing performs two separate operations: `x[1]` creates an intermediate view of row 1, and then `[2]` indexes into that view. The intermediate view creation adds overhead, making chained indexing approximately 2× slower.

### Real-World Cases

- **Performance-Critical Loops:** When indexing arrays repeatedly in tight loops, tuple indexing can significantly reduce execution time.
- **Code Readability:** Tuple indexing `arr[i, j]` is generally considered more readable and Pythonic for multidimensional arrays.
- **Assignment:** Both notations work for assignment, but tuple indexing is clearer and less error-prone.

### References

- NumPy Indexing on ndarrays – https://numpy.org/doc/stable/user/basics.indexing.html
- Stack Overflow: Why is nparray[i][j] slower than nparray[i, j]? – https://stackoverflow.com/questions/41791253
- Stack Overflow: Timeline for numpy 2d array indexing – https://stackoverflow.com/posts/5178009/timeline

---

## Summary of Deprecated and Version-Specific Features

| Feature | Status | Recommendation |
|---------|--------|----------------|
| `arr[i][j]` for multidimensional access | Functional but inefficient | Use `arr[i, j]` instead |
| Setting `arr.shape` directly | Deprecated | Use `arr.reshape()` instead |
| `UPDATEIFCOPY` flag | Deprecated | Do not use in new code |

---

## Consolidated Reference List

- NumPy Indexing on ndarrays – https://numpy.org/doc/stable/user/basics.indexing.html
- NumPy How to index ndarrays – https://numpy.org/doc/2.5/user/how-to-index.html
- NumPy Copies and Views – https://numpy.org/doc/1.22/user/basics.copies.html
- NumPy Quickstart – https://numpy.org/doc/stable/user/quickstart.html
- NIELIT Access NumPy Array Elements – http://nielit.gov.in/gorakhpur/sites/default/files/Gorakhpur/ALevel_1_Python_02June2020_SS.pdf
- Stack Overflow: Why is nparray[i][j] slower than nparray[i, j]? – https://stackoverflow.com/questions/41791253
- Stack Overflow: Timeline for numpy 2d array indexing – https://stackoverflow.com/posts/5178009/timeline
- NumPy Indexing routines – https://numpy.org/doc/stable/reference/routines.indexing.html