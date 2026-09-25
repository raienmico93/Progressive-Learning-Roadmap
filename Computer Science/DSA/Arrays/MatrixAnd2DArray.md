# DSA Matrix and 2D Array Techniques: Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**
A matrix (or 2D array) is a rectangular collection of elements organized in rows and columns, where each element is accessed via a pair of indices representing its row and column position. Matrix techniques encompass traversal patterns, transformation algorithms, and optimization strategies specific to grid-based data.

**Technical Definition**
An `m × n` matrix constitutes a mathematical object visualized as a 2D table of values, typically represented in programming as a nested array `matrix[i][j]` where `i` is the row index (0 to m-1) and `j` is the column index (0 to n-1). In row-major storage (C, C++, Java), elements are stored row by row in contiguous memory, with the element at `[i][j]` located at `base + (i * n + j) * element_size`.

**Beginner-Friendly Explanation**
Think of a matrix as a spreadsheet or a chessboard. You need two coordinates to find a specific cell: which row it's in, and which column. Matrix techniques are simply organized ways to walk through, modify, or analyze these grids efficiently.

### Key Characteristics

- **Two-Dimensional Indexing**: Elements accessed via `matrix[row][col]` with zero-based indices
- **Rectangular Structure**: All rows typically have the same number of columns (jagged arrays are possible in some languages)
- **Contiguous Memory (Row-Major)**: In C/C++/Java, rows are stored sequentially, making row-wise traversal cache-friendly
- **Boundary Sensitivity**: Traversal algorithms must carefully manage row/column bounds to avoid out-of-bounds access
- **In-Place Transformation Capability**: Many operations (rotation, transpose) can be performed without extra space

### Prerequisites

- One-dimensional array fundamentals
- Nested loops and conditional logic
- Zero-based indexing and bounds checking
- Big O notation for complexity analysis

### Related Programming Areas

- **Image Processing**: Pixel matrices for filters and transformations
- **Game Development**: Grid-based game boards and pathfinding
- **Scientific Computing**: Linear algebra and numerical methods
- **Graph Algorithms**: Adjacency matrices and grid-based search

### Core Concepts / Features

| Concept | Primary Application | Key Technique |
|---------|---------------------|---------------|
| Matrix Traversal | Visiting all elements | Row-wise, column-wise, zigzag |
| Rotation | Image/grid transformation | Transpose + Reverse, Layer-by-Layer |
| Spiral Traversal | Boundary-based processing | 4-wall boundary shrinking |
| Prefix Sum Matrix | O(1) range queries | 2D cumulative sum precomputation |
| Grid DP | Pathfinding, counting paths | Bottom-up or top-down DP |
| Sparse Representation | Memory-efficient storage | CSR, CSC, COO formats |

---

## 1. Matrix Traversal Patterns

### Definitions

**Core Definition**
Matrix traversal is the systematic process of visiting every element in a 2D array exactly once, following a defined order such as row-by-row, column-by-column, or a custom pattern.

**Technical Definition**
Traversal involves nested loops where the outer loop typically iterates over rows (`i` from `0` to `m-1`) and the inner loop iterates over columns (`j` from `0` to `n-1`), accessing `matrix[i][j]` at each iteration.

**Beginner-Friendly Explanation**
Traversal is like reading a book. You can read normally (left to right, top to bottom), read column by column, or read in a zigzag pattern. The order you choose depends on what you need to accomplish.

### Purposes (All begin with "To")

- **To** process every element in a matrix for computation or analysis
- **To** search for specific values or conditions within a grid
- **To** transform matrix data into a different format (e.g., 1D array)
- **To** validate matrix properties (symmetry, identity, etc.)

### Syntax Rules and Structure

**General Syntax (Row-Wise Traversal)**
```
for i from 0 to rows - 1:
    for j from 0 to cols - 1:
        process(matrix[i][j])
```

**General Syntax (Column-Wise Traversal)**
```
for j from 0 to cols - 1:
    for i from 0 to rows - 1:
        process(matrix[i][j])
```

**Component Breakdown**

| Component | Description | Row-Wise | Column-Wise |
|-----------|-------------|----------|-------------|
| Outer loop | Controls primary index | Rows (`i`) | Columns (`j`) |
| Inner loop | Controls secondary index | Columns (`j`) | Rows (`i`) |
| Access pattern | Cache efficiency | High (contiguous) | Low (strided) |
| Use case | Row operations | Column operations |

**Syntax Rules**

- Rows are indexed `0` to `m-1`, columns `0` to `n-1`
- Row-major languages (C/C++/Java) favor row-wise traversal for cache efficiency
- In Python, NumPy defaults to C-order (row-major), while Fortran uses column-major
- Traversal order affects which operations are efficient (e.g., summing rows vs. columns)

**Constraints and Limitations**

- Column-wise traversal is cache-inefficient in row-major languages
- Jagged arrays (Java) may have rows of different lengths
- Traversal must respect boundaries to avoid `IndexOutOfBounds` exceptions

### Annotated Code Examples

**Example 1: Row-Wise and Column-Wise Traversal (Python)**
```python
matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
]

rows = len(matrix)
cols = len(matrix[0])

# Row-wise traversal (cache-friendly for row-major)
print("Row-wise traversal:")
for i in range(rows):
    for j in range(cols):
        # Access each element row by row
        print(matrix[i][j], end=" ")
    print()  # New line after each row

# Column-wise traversal (cache-unfriendly but sometimes needed)
print("\nColumn-wise traversal:")
for j in range(cols):
    for i in range(rows):
        # Access each element column by column
        print(matrix[i][j], end=" ")
    print()
```

**Expected Output:**
```
Row-wise traversal:
1 2 3 
4 5 6 
7 8 9 

Column-wise traversal:
1 4 7 
2 5 8 
3 6 9 
```

**Why This Output Occurs**: Row-wise traversal accesses elements in memory order (1,2,3,4,5,6,7,8,9). Column-wise traversal jumps through memory (1,4,7,2,5,8,3,6,9), which is less cache-efficient in row-major storage but necessary for column-oriented operations.

### Real-World Cases

- **Image Processing**: Applying filters to every pixel in row or column order
- **Spreadsheet Software**: Iterating cells for recalculation
- **Game Boards**: Scanning for win conditions (row, column, diagonal)
- **Data Migration**: Converting between row-major and column-major formats

### References

- IIT Kanpur - Multi-dimensional Arrays - https://www.cgs.iitk.ac.in/user/hk/csd101/2021/inClass/l14.pdf
- Educative - Matrix Traversals and Mapping - https://www.educative.io/courses/beginner-to-advanced-computing-and-logic-building/lta/problem-solving-matrix-traversals-i

---

## 2. Row and Column Operations

### Definitions

**Core Definition**
Row and column operations are computations that treat entire rows or columns as units, such as summing a row, finding the maximum in a column, or swapping rows.

**Technical Definition**
Row operations access elements with a fixed row index while varying column indices (`matrix[i][0..n-1]`). Column operations access with a fixed column index while varying row indices (`matrix[0..m-1][j]`).

**Beginner-Friendly Explanation**
Think of rows as horizontal lines and columns as vertical lines. Row operations work with complete horizontal slices; column operations work with complete vertical slices.

### Purposes (All begin with "To")

- **To** compute aggregate statistics per row or column (sum, max, min)
- **To** swap or reorder rows/columns for matrix manipulation
- **To** implement Gaussian elimination and linear algebra operations
- **To** detect patterns or anomalies in tabular data

### Syntax Rules and Structure

**General Syntax (Row Sum)**
```
for i from 0 to rows - 1:
    row_sum = 0
    for j from 0 to cols - 1:
        row_sum += matrix[i][j]
    output row_sum
```

**General Syntax (Column Sum)**
```
for j from 0 to cols - 1:
    col_sum = 0
    for i from 0 to rows - 1:
        col_sum += matrix[i][j]
    output col_sum
```

**Component Breakdown**

| Component | Description | Row Operation | Column Operation |
|-----------|-------------|---------------|------------------|
| Fixed index | Constant dimension | Row `i` | Column `j` |
| Varying index | Iterated dimension | Column `j` | Row `i` |
| Inner loop | Accumulation | Sum across columns | Sum across rows |
| Complexity | Time | O(m × n) | O(m × n) |

**Syntax Rules**

- Row operations are cache-friendly in row-major storage
- Column operations require strided access (slower in row-major)
- Nested loops can be swapped to change operation type
- Intermediate storage may be needed for results (e.g., array of row sums)

**Constraints and Limitations**

- Column operations in row-major languages have poor cache performance
- Row/column swaps require element-by-element copying (O(n) or O(m))
- Matrix must be rectangular for uniform operations

### Annotated Code Examples

**Example 1: Row Sums and Column Sums (C++)**
```cpp
#include <iostream>
using namespace std;

int main() {
    int matrix[3][4] = {
        {1, 2, 3, 4},
        {5, 6, 7, 8},
        {9, 10, 11, 12}
    };
    int rows = 3, cols = 4;
    
    // Row sums: iterate rows, accumulate across columns
    cout << "Row sums:" << endl;
    for (int i = 0; i < rows; i++) {
        int row_sum = 0;
        for (int j = 0; j < cols; j++) {
            row_sum += matrix[i][j];  // Sum across columns
        }
        cout << "Row " << i << ": " << row_sum << endl;
    }
    
    // Column sums: iterate columns, accumulate across rows
    cout << "\nColumn sums:" << endl;
    for (int j = 0; j < cols; j++) {
        int col_sum = 0;
        for (int i = 0; i < rows; i++) {
            col_sum += matrix[i][j];  // Sum across rows
        }
        cout << "Col " << j << ": " << col_sum << endl;
    }
    
    return 0;
}
```

**Expected Output:**
```
Row sums:
Row 0: 10
Row 1: 26
Row 2: 42

Column sums:
Col 0: 15
Col 1: 18
Col 2: 21
Col 3: 24
```

**Why This Output Occurs**: Row 0 sums 1+2+3+4=10; Row 1 sums 5+6+7+8=26; Row 2 sums 9+10+11+12=42. Column 0 sums 1+5+9=15; Column 1 sums 2+6+10=18, etc. The row sums are cache-efficient (contiguous access), while column sums are strided.

### Real-World Cases

- **Gradebooks**: Computing student averages (row sums) or class averages per assignment (column sums)
- **Financial Reports**: Row totals (account balances) and column totals (category sums)
- **Image Histograms**: Column-wise intensity sums for vertical profiles
- **Board Games**: Row/column win detection in Connect Four or tic-tac-toe

### References

- IIT Kanpur - Matrix Operations - https://www.cgs.iitk.ac.in/user/hk/csd101/2021/inClass/l14.pdf
- Educative - Matrix Traversals and Mapping - https://www.educative.io/courses/beginner-to-advanced-computing-and-logic-building/lta/problem-solving-matrix-traversals-i

---

## 3. Diagonal and Anti-Diagonal Traversal

### Definitions

**Core Definition**
Diagonal traversal visits elements along lines that run from top-left to bottom-right (main diagonals) or from top-right to bottom-left (anti-diagonals).

**Technical Definition**
The main diagonal of a square matrix consists of elements where `row == col`. Anti-diagonal elements satisfy `row + col == n - 1` for an `n × n` matrix. General diagonals are identified by the constant `row - col` (main direction) or `row + col` (anti-direction).

**Beginner-Friendly Explanation**
Imagine drawing lines across a grid at 45-degree angles. Diagonal traversal follows these lines.

### Purposes (All begin with "To")

- **To** process elements in diagonal bands (e.g., dynamic programming on diagonals)
- **To** detect diagonal win conditions in games (tic-tac-toe, Connect Four)
- **To** perform matrix transposition or rotation operations
- **To** compute diagonal sums or products

### Syntax Rules and Structure

**General Syntax (Main Diagonal)**
```
# For square matrix: main diagonal where i == j
for i from 0 to n - 1:
    process(matrix[i][i])
```

**General Syntax (Anti-Diagonal)**
```
# For square matrix: anti-diagonal where i + j == n - 1
for i from 0 to n - 1:
    j = n - 1 - i
    process(matrix[i][j])
```

**General Syntax (All Diagonals)**
```
# Iterate by diagonal index d (from 0 to m+n-2)
for d from 0 to m + n - 2:
    # Determine starting row and column
    start_row = max(0, d - (n - 1))
    start_col = d - start_row
    count = min(m - start_row, n - start_col)
    
    for k from 0 to count - 1:
        i = start_row + k
        j = start_col + k
        process(matrix[i][j])
```

**Component Breakdown**

| Diagonal Type | Condition | Direction | Use Case |
|---------------|-----------|-----------|----------|
| Main | `i == j` | Top-left to bottom-right | Trace, identity |
| Anti | `i + j == n - 1` | Top-right to bottom-left | Anti-diagonal sum |
| General | `i - j == d` | Constant offset | DP, convolution |

**Syntax Rules**

- Main diagonal exists only in square matrices (`m == n`)
- Anti-diagonal condition for square: `i + j == n - 1`
- General diagonal indices range from `-(m-1)` to `(n-1)` for main-direction diagonals
- Diagonal traversal often requires boundary checks for non-square matrices

**Constraints and Limitations**

- Square matrix assumption for simple `i == j` condition
- Non-square matrices require careful boundary handling
- Diagonal traversal is cache-unfriendly (strided access)

### Annotated Code Examples

**Example 1: Main and Anti-Diagonal of Square Matrix (C)**
```c
#include <stdio.h>

int main() {
    int matrix[3][3] = {
        {1, 2, 3},
        {4, 5, 6},
        {7, 8, 9}
    };
    int n = 3;
    
    // Main diagonal: i == j
    printf("Main diagonal: ");
    for (int i = 0; i < n; i++) {
        printf("%d ", matrix[i][i]);
    }
    printf("\n");
    
    // Anti-diagonal: i + j == n - 1
    printf("Anti-diagonal: ");
    for (int i = 0; i < n; i++) {
        int j = n - 1 - i;  // Compute column for anti-diagonal
        printf("%d ", matrix[i][j]);
    }
    printf("\n");
    
    return 0;
}
```

**Expected Output:**
```
Main diagonal: 1 5 9 
Anti-diagonal: 3 5 7 
```

**Why This Output Occurs**: Main diagonal elements are at (0,0)=1, (1,1)=5, (2,2)=9. Anti-diagonal elements are at (0,2)=3, (1,1)=5, (2,0)=7. The condition `i + j == n - 1` gives j = 2-i for n=3.

### Real-World Cases

- **Dynamic Programming**: Matrix chain multiplication uses diagonal iteration
- **Game AI**: Checking diagonal threats in chess or tic-tac-toe
- **Convolutional Neural Networks**: Diagonal feature extraction
- **Image Processing**: Diagonal edge detection filters

### References

- NIELIT - Anti-diagonal of Square Matrix - http://nielit.gov.in/gorakhpur/sites/default/files/Gorakhpur/olevel_2_C_Language_15May2020_RJS.pdf
- IIT Guwahati - Dynamic Programming on Diagonals - http://www.iitg.ac.in/psm/indexing_ma353/y09/LectureNoteMA515Oct12.pdf

---

## 4. Matrix Rotation (90, 180, 270 Degrees)

### Definitions

**Core Definition**
Matrix rotation transforms a matrix by rotating all elements around its center by a specified angle (90°, 180°, or 270°), changing the positions of elements while preserving the rectangular structure.

**Technical Definition**
For an `n × n` matrix, 90° clockwise rotation maps element at `(i, j)` to `(j, n-1-i)`. This can be achieved in-place by first transposing (`swap matrix[i][j]` with `matrix[j][i]`) and then reversing each row.

**Beginner-Friendly Explanation**
Rotating a matrix is like physically turning a picture frame 90 degrees clockwise. What was on the left edge is now on the top, what was on top is now on the right.

### Purposes (All begin with "To")

- **To** transform image data (rotate images by 90° increments)
- **To** solve matrix manipulation problems in-place
- **To** convert between row-major and column-major perspectives
- **To** prepare data for different algorithmic access patterns

### Syntax Rules and Structure

**General Syntax (90° Clockwise In-Place)**
```
# Step 1: Transpose (swap across main diagonal)
for i from 0 to n - 1:
    for j from i + 1 to n - 1:
        swap(matrix[i][j], matrix[j][i])

# Step 2: Reverse each row
for each row in matrix:
    reverse(row)
```

**General Syntax (180°)**
```
# Reverse rows and columns
for i from 0 to n - 1:
    for j from 0 to n/2 - 1:
        swap(matrix[i][j], matrix[i][n-1-j])

for i from 0 to n/2 - 1:
    swap rows i and n-1-i
```

**Component Breakdown**

| Rotation | Transformation | Steps | In-Place? |
|----------|---------------|-------|-----------|
| 90° CW | `(i,j) → (j, n-1-i)` | Transpose + Reverse rows | Yes |
| 90° CCW | `(i,j) → (n-1-j, i)` | Transpose + Reverse columns | Yes |
| 180° | `(i,j) → (n-1-i, n-1-j)` | Reverse rows + Reverse columns | Yes |
| 270° CW | Same as 90° CCW | Transpose + Reverse columns | Yes |

**Syntax Rules**

- In-place rotation requires `O(1)` extra space (excluding loop variables)
- Transpose swaps `matrix[i][j]` with `matrix[j][i]` for `i < j`
- 90° clockwise = Transpose + Reverse each row
- 90° counter-clockwise = Transpose + Reverse each column
- 180° = Reverse rows and columns (or apply 90° twice)

**Constraints and Limitations**

- Layer-by-layer rotation is an alternative O(n²) in-place method
- Only square matrices can be rotated in-place with simple algorithms
- Non-square rotation requires dimension swapping (m×n becomes n×m)

### Annotated Code Examples

**Example 1: 90° Clockwise In-Place Rotation (Python)**
```python
def rotate_90_clockwise(matrix):
    """Rotate n x n matrix 90 degrees clockwise in-place."""
    n = len(matrix)
    
    # Step 1: Transpose (swap across main diagonal)
    for i in range(n):
        for j in range(i + 1, n):  # Only upper triangle
            matrix[i][j], matrix[j][i] = matrix[j][i], matrix[i][j]
    
    # Step 2: Reverse each row horizontally
    for i in range(n):
        matrix[i].reverse()

# Test
matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
]

print("Original:")
for row in matrix:
    print(row)

rotate_90_clockwise(matrix)

print("\nAfter 90° clockwise:")
for row in matrix:
    print(row)
```

**Expected Output:**
```
Original:
[1, 2, 3]
[4, 5, 6]
[7, 8, 9]

After 90° clockwise:
[7, 4, 1]
[8, 5, 2]
[9, 6, 3]
```

**Why This Output Occurs**: Transpose swaps `matrix[0][1]`↔`matrix[1][0]` (2↔4), `matrix[0][2]`↔`matrix[2][0]` (3↔7), `matrix[1][2]`↔`matrix[2][1]` (6↔8), yielding `[[1,4,7],[2,5,8],[3,6,9]]`. Reversing each row gives `[[7,4,1],[8,5,2],[9,6,3]]`—the correct 90° clockwise rotation.

### Real-World Cases

- **Image Editing**: Rotating photos by 90° increments
- **Game Development**: Rotating game boards or tile maps
- **Matrix Libraries**: BLAS/LAPACK rotation operations
- **Computer Vision**: Normalizing image orientation

### References

- GitHub - Two Pointers: Transpose and Reverse Technique - https://raw.githubusercontent.com/Stewie-pixel/claude-with-leetcode/refs/heads/main/c/48-rotate-image/ANALYSIS.md
- GitHub - Matrix Rotation Patterns - https://github.com/sajidhamdard/dsa/commit/348d65aa6e876a4bf0afaa2537c880528d40364f
- LeetCode - Spiral Matrix Solutions - https://leetcode.com/problems/spiral-matrix/solutions/20573/a-concise-c-implementation-based-on-dire-zls9/

---

## 5. Spiral Traversal

### Definitions

**Core Definition**
Spiral traversal visits all elements of a matrix in a clockwise spiral pattern, starting from the top-left corner and moving right, down, left, up, then inward.

**Technical Definition**
Spiral traversal maintains four boundary variables (`top`, `bottom`, `left`, `right`) that define the current perimeter. After traversing each edge, the corresponding boundary is shrunk inward.

**Beginner-Friendly Explanation**
Imagine walking around the outside edge of a rectangular garden, then stepping inward and walking around the next inner edge, continuing until you reach the center.

### Purposes (All begin with "To")

- **To** process matrix elements in a perimeter-to-center order
- **To** solve spiral matrix generation problems
- **To** implement boundary-based algorithms (e.g., image peeling)
- **To** convert 2D data to 1D in a specific ordering

### Syntax Rules and Structure

**General Syntax**
```
top = 0, bottom = rows - 1
left = 0, right = cols - 1

while top <= bottom and left <= right:
    # Traverse right along top row
    for j from left to right:
        process(matrix[top][j])
    top += 1
    
    # Traverse down along right column
    for i from top to bottom:
        process(matrix[i][right])
    right -= 1
    
    # Traverse left along bottom row (if valid)
    if top <= bottom:
        for j from right downto left:
            process(matrix[bottom][j])
        bottom -= 1
    
    # Traverse up along left column (if valid)
    if left <= right:
        for i from bottom downto top:
            process(matrix[i][left])
        left += 1
```

**Component Breakdown**

| Boundary | Initial | After Right | After Down | After Left | After Up |
|----------|---------|-------------|------------|------------|----------|
| top | 0 | +1 | — | — | — |
| bottom | rows-1 | — | — | -1 | — |
| left | 0 | — | — | — | +1 |
| right | cols-1 | — | -1 | — | — |

**Syntax Rules**

- Four boundaries define the current spiral layer
- Each edge traversal shrinks one boundary inward
- Boundary checks prevent duplicate traversal of single-row/column matrices
- Direction order: RIGHT → DOWN → LEFT → UP (clockwise)

**Constraints and Limitations**

- Boundary checks (`if top <= bottom`, `if left <= right`) are essential for non-square matrices
- Without checks, single-row or single-column matrices cause duplicate elements
- Time complexity: O(m × n), Space: O(1) excluding output

### Annotated Code Examples

**Example 1: Spiral Traversal of 3×4 Matrix**
```python
def spiral_order(matrix):
    """Return elements in spiral order."""
    if not matrix or not matrix[0]:
        return []
    
    result = []
    top, bottom = 0, len(matrix) - 1
    left, right = 0, len(matrix[0]) - 1
    
    while top <= bottom and left <= right:
        # Move right along top row
        for j in range(left, right + 1):
            result.append(matrix[top][j])
        top += 1
        
        # Move down along right column
        for i in range(top, bottom + 1):
            result.append(matrix[i][right])
        right -= 1
        
        # Move left along bottom row (check valid)
        if top <= bottom:
            for j in range(right, left - 1, -1):
                result.append(matrix[bottom][j])
            bottom -= 1
        
        # Move up along left column (check valid)
        if left <= right:
            for i in range(bottom, top - 1, -1):
                result.append(matrix[i][left])
            left += 1
    
    return result

# Test
matrix = [
    [1, 2, 3, 4],
    [5, 6, 7, 8],
    [9, 10, 11, 12]
]

print(spiral_order(matrix))
```

**Expected Output:**
```
[1, 2, 3, 4, 8, 12, 11, 10, 9, 5, 6, 7]
```

**Why This Output Occurs**: The traversal follows the perimeter: top row (1,2,3,4), right column (8,12), bottom row (11,10,9), left column (5). Then the inner layer: top (6,7). The boundary checks prevent re-visiting elements when the remaining layer is a single row or column.

### Real-World Cases

- **Image Processing**: Spiral scanning for compression or feature detection
- **Game Development**: Spiral spawn patterns or enemy placement
- **Data Serialization**: Converting 2D grids to 1D streams
- **Robotics**: Spiral search patterns for area coverage

### References

- LeetCode - Concise Spiral Matrix Implementation - https://leetcode.com/problems/spiral-matrix/solutions/20573/a-concise-c-implementation-based-on-dire-zls9/
- Educative - Spiral Matrix Algorithm - https://www.educative.io/answers/spiral-matrix-algorithm
- LeetCode - 2D Matrix & Grid Patterns Cheat Sheet - https://leetcode.com/discuss/post/8533041/set-28-2d-matrix-grid-patterns-python-ds-76aw/

---

## 6. Prefix-Sum Matrices (2D Cumulative Sums)

### Definitions

**Core Definition**
A 2D prefix sum matrix stores cumulative sums such that each cell `prefix[i][j]` represents the sum of all elements in the submatrix from `(0,0)` to `(i-1, j-1)`, enabling O(1) range sum queries.

**Technical Definition**
The 2D prefix sum is defined as `P[i][j] = matrix[i-1][j-1] + P[i-1][j] + P[i][j-1] - P[i-1][j-1]`. The sum of submatrix `(r1,c1)` to `(r2,c2)` is `P[r2+1][c2+1] - P[r1][c2+1] - P[r2+1][c1] + P[r1][c1]`.

**Beginner-Friendly Explanation**
Imagine keeping a running total of everything above and to the left of each cell. To find the sum of any rectangle, you just do some subtraction with four corners.

### Purposes (All begin with "To")

- **To** answer 2D range sum queries in O(1) time after O(m×n) precomputation
- **To** solve submatrix sum problems efficiently
- **To** enable difference array techniques for 2D range updates
- **To** compute integral images for computer vision

### Syntax Rules and Structure

**General Syntax (Building Prefix Matrix)**
```
# P has (rows+1) x (cols+1) dimensions
P = [[0] * (cols + 1) for _ in range(rows + 1)]

for i from 1 to rows:
    for j from 1 to cols:
        P[i][j] = matrix[i-1][j-1] + P[i-1][j] + P[i][j-1] - P[i-1][j-1]
```

**General Syntax (Range Query)**
```
def sumRegion(r1, c1, r2, c2):
    return P[r2+1][c2+1] - P[r1][c2+1] - P[r2+1][c1] + P[r1][c1]
```

**Component Breakdown**

| Component | Description | Value |
|-----------|-------------|-------|
| `P[0][j]` | Empty top row | 0 |
| `P[i][0]` | Empty left column | 0 |
| `P[i][j]` | Sum of `(0,0)` to `(i-1,j-1)` | 4-term formula |
| Query | Sum of `(r1,c1)` to `(r2,c2)` | 4-term inclusion-exclusion |

**Syntax Rules**

- Prefix matrix dimensions are `(rows+1) × (cols+1)` to handle empty ranges
- The inclusion-exclusion principle prevents double-counting the overlap
- Building takes O(m × n) time and space
- Each query is O(1) regardless of submatrix size

**Constraints and Limitations**

- Requires O(m × n) extra space for the prefix matrix
- Only useful for multiple queries; single query is O(m × n) anyway
- Matrix must be immutable between queries (or rebuilt after updates)
- For 2D range updates, use 2D difference arrays

### Annotated Code Examples

**Example 1: Build 2D Prefix Sum and Query (Python)**
```python
def build_prefix_2d(matrix):
    """Build 2D prefix sum matrix."""
    if not matrix or not matrix[0]:
        return []
    
    rows, cols = len(matrix), len(matrix[0])
    # Initialize with extra row and column for empty ranges
    P = [[0] * (cols + 1) for _ in range(rows + 1)]
    
    for i in range(1, rows + 1):
        for j in range(1, cols + 1):
            P[i][j] = (matrix[i-1][j-1] 
                       + P[i-1][j]      # Sum above
                       + P[i][j-1]      # Sum left
                       - P[i-1][j-1])   # Subtract overlap
    
    return P

def sum_region(P, r1, c1, r2, c2):
    """Sum of submatrix (r1,c1) to (r2,c2) inclusive."""
    return (P[r2+1][c2+1] 
            - P[r1][c2+1] 
            - P[r2+1][c1] 
            + P[r1][c1])

# Test
matrix = [
    [3, 0, 1, 4],
    [5, 6, 3, 2],
    [1, 2, 0, 1]
]

P = build_prefix_2d(matrix)

print("Prefix matrix:")
for row in P:
    print(row)

# Query: sum of submatrix from (1,1) to (2,2)
print(f"\nSum (1,1) to (2,2): {sum_region(P, 1, 1, 2, 2)}")
# Expected: 6+3+2+0 = 11
```

**Expected Output:**
```
Prefix matrix:
[0, 0, 0, 0, 0]
[0, 3, 3, 4, 8]
[0, 8, 14, 18, 24]
[0, 9, 17, 21, 28]

Sum (1,1) to (2,2): 11
```

**Why This Output Occurs**: The prefix matrix accumulates sums from origin. `P[3][3] = 21` = sum of first 3 rows and 3 columns. The query `(1,1)` to `(2,2)` = `P[3][3] - P[1][3] - P[3][1] + P[1][1]` = 21 - 4 - 9 + 3 = 11, which equals 6+3+2+0 = 11.

### Real-World Cases

- **Image Processing**: Integral images for fast rectangle feature computation
- **Financial Analysis**: Cumulative sales by region and product
- **Game Development**: Fast area-of-effect damage calculations
- **Data Analytics**: Submatrix aggregation for heatmaps

### References

- GitHub - Prefix Sum Technique - https://raw.githubusercontent.com/pluginagentmarketplace/custom-plugin-data-structures-algorithms/refs/heads/main/skills/arrays/prefix-sum.md
- Algorithm Anthology - 2D Difference Arrays - https://raw.githubusercontent.com/alxli/algorithm-anthology/master/Book/A3C5-v1.0.pdf

---

## 7. Grid-Based Pathfinding and Dynamic Programming

### Definitions

**Core Definition**
Grid-based pathfinding finds optimal paths from a start cell to an end cell in a matrix, where movement is typically restricted to adjacent cells and each cell has an associated cost. Dynamic programming on grids solves problems by breaking them into overlapping subproblems.

**Technical Definition**
Grid DP uses a recurrence relation `dp[i][j] = f(dp[i-1][j], dp[i][j-1], ...)` based on allowed movement directions (4 cardinal or 8 directional). The base case is typically `dp[0][0] = matrix[0][0]`, and the answer is at `dp[rows-1][cols-1]`.

**Beginner-Friendly Explanation**
Imagine a robot on a grid trying to reach a goal while avoiding obstacles. Grid DP is like solving a maze where you figure out the best path to each cell by building on solutions to smaller problems.

### Purposes (All begin with "To")

- **To** find minimum cost paths in grids with obstacles
- **To** count the number of distinct paths between two cells
- **To** solve maximum/minimum sum path problems
- **To** implement flood-fill and connectivity algorithms

### Syntax Rules and Structure

**General Syntax (Bottom-Up DP)**
```
# Initialize base cases
dp[0][0] = matrix[0][0]

# Fill first row
for j from 1 to cols - 1:
    dp[0][j] = dp[0][j-1] + matrix[0][j]

# Fill first column
for i from 1 to rows - 1:
    dp[i][0] = dp[i-1][0] + matrix[i][0]

# Fill remaining cells
for i from 1 to rows - 1:
    for j from 1 to cols - 1:
        dp[i][j] = min(dp[i-1][j], dp[i][j-1]) + matrix[i][j]

return dp[rows-1][cols-1]
```

**General Syntax (DFS with Memoization)**
```
memo = {}

def dfs(i, j):
    if (i, j) in memo:
        return memo[(i, j)]
    if i >= rows or j >= cols:
        return infinity
    if i == rows - 1 and j == cols - 1:
        return matrix[i][j]
    
    result = matrix[i][j] + min(dfs(i+1, j), dfs(i, j+1))
    memo[(i, j)] = result
    return result
```

**Component Breakdown**

| Component | Description | Value |
|-----------|-------------|-------|
| State | Cell position | `(i, j)` |
| Recurrence | How to compute from neighbors | `min(dp[i-1][j], dp[i][j-1]) + matrix[i][j]` |
| Base case | Starting cell(s) | `dp[0][0]` |
| Answer | Target cell | `dp[rows-1][cols-1]` |

**Syntax Rules**

- Movement directions determine which neighbors to consider
- 4-directional: up, down, left, right
- 8-directional: includes diagonals
- Obstacles typically represented as infinity or negative markers
- Memoization avoids recomputing overlapping subproblems

**Constraints and Limitations**

- Bottom-up requires O(m × n) space (can be optimized to O(n))
- DFS with memoization may hit recursion limits for large grids
- Grid DP works only when subproblems have optimal substructure

### Annotated Code Examples

**Example 1: Minimum Path Sum (Python)**
```python
def min_path_sum(grid):
    """
    Find minimum sum path from top-left to bottom-right.
    Movement: right or down only.
    """
    if not grid or not grid[0]:
        return 0
    
    rows, cols = len(grid), len(grid[0])
    
    # Create DP table
    dp = [[0] * cols for _ in range(rows)]
    
    # Base case: starting cell
    dp[0][0] = grid[0][0]
    
    # Fill first row (only can come from left)
    for j in range(1, cols):
        dp[0][j] = dp[0][j-1] + grid[0][j]
    
    # Fill first column (only can come from above)
    for i in range(1, rows):
        dp[i][0] = dp[i-1][0] + grid[i][0]
    
    # Fill remaining cells (from top or left)
    for i in range(1, rows):
        for j in range(1, cols):
            dp[i][j] = min(dp[i-1][j], dp[i][j-1]) + grid[i][j]
    
    return dp[rows-1][cols-1]

# Test
grid = [
    [1, 3, 1],
    [1, 5, 1],
    [4, 2, 1]
]

print(f"Minimum path sum: {min_path_sum(grid)}")
# Path: 1→3→1→1→1 = 7 (right, right, down, down)
```

**Expected Output:**
```
Minimum path sum: 7
```

**Why This Output Occurs**: The DP table computes the minimum cost to reach each cell. For the example, the optimal path is 1 (start) → 3 → 1 → 1 → 1, with sum 7. The recurrence `min(top, left) + current` ensures the cheapest path is found at each step.

### Real-World Cases

- **Robotics**: Path planning on occupancy grids
- **Game AI**: NPC navigation around obstacles
- **Network Routing**: Packet routing on grid topologies
- **Image Segmentation**: Minimum-cost boundary detection

### References

- GitHub - Graph DFS on Grids - https://git.mashibing.com/msb-public/tech-interview-handbook/raw/commit/01bcbfc49f9be65ba182469e5f55c2fd5a56d8f7/utilities/python/graph_dfs.py
- NASA - Dynamic Programming on Grid Lattice Graphs - https://ntrs.nasa.gov/api/citations/19880010064/downloads/19880010064.pdf

---

## 8. Sparse Matrix Representation and Compression

### Definitions

**Core Definition**
A sparse matrix is a matrix in which most elements are zero (or a default value). Sparse representations store only the non-zero elements, saving significant memory for large matrices.

**Technical Definition**
Common sparse formats include COO (Coordinate), CSR (Compressed Sparse Row), and CSC (Compressed Sparse Column). CSR stores three arrays: values `AA`, column indices `JA`, and row pointers `IA`.

**Beginner-Friendly Explanation**
If you have a huge spreadsheet where almost every cell is empty, you don't need to store all the empty cells. Sparse formats only record where the actual values are.

### Purposes (All begin with "To")

- **To** reduce memory usage for matrices with many zeros
- **To** speed up matrix operations by skipping zero elements
- **To** enable efficient storage of large-scale scientific data
- **To** optimize graph adjacency matrix representation

### Syntax Rules and Structure

**CSR Format Components**
```
AA = [non-zero values, row by row]
JA = [column indices for each value]
IA = [starting index in AA for each row, plus total count]
```

**COO Format**
```
row_indices = [row for each non-zero]
col_indices = [col for each non-zero]
values = [value for each non-zero]
```

**Component Breakdown**

| Format | Arrays | Best For | Conversion |
|--------|--------|----------|------------|
| COO | row, col, value | Building/editing | → CSR/CSC |
| CSR | value, col, row_ptr | Row operations | From COO |
| CSC | value, row, col_ptr | Column operations | Transpose of CSR |

**Syntax Rules**

- `IA` has `rows + 1` entries; `IA[i]` to `IA[i+1]-1` are indices for row `i`
- `JA` stores column indices corresponding to each value in `AA`
- COO is simplest but less efficient for arithmetic
- CSR is efficient for row-wise operations and matrix-vector multiplication

**Constraints and Limitations**

- CSR/CSC are difficult to modify (insertion/deletion is O(nnz))
- ELLPACK format is inefficient when rows have varying non-zero counts
- Format conversion requires O(nnz) time and space

### Annotated Code Examples

**Example 1: CSR Representation (Python)**
```python
def dense_to_csr(matrix):
    """Convert dense matrix to CSR format."""
    rows = len(matrix)
    AA = []  # non-zero values
    JA = []  # column indices
    IA = [0]  # row pointers
    
    for i in range(rows):
        for j in range(len(matrix[i])):
            if matrix[i][j] != 0:
                AA.append(matrix[i][j])
                JA.append(j)
        IA.append(len(AA))  # End of this row
    
    return AA, JA, IA

def csr_to_dense(AA, JA, IA, rows, cols):
    """Convert CSR back to dense matrix."""
    matrix = [[0] * cols for _ in range(rows)]
    
    for i in range(rows):
        for k in range(IA[i], IA[i+1]):
            j = JA[k]
            matrix[i][j] = AA[k]
    
    return matrix

# Test
dense = [
    [1, 0, 0, 0],
    [0, 2, 0, 0],
    [0, 0, 3, 0],
    [0, 0, 0, 4]
]

AA, JA, IA = dense_to_csr(dense)
print(f"AA (values): {AA}")
print(f"JA (col indices): {JA}")
print(f"IA (row pointers): {IA}")

# Reconstruct
reconstructed = csr_to_dense(AA, JA, IA, 4, 4)
print("\nReconstructed:")
for row in reconstructed:
    print(row)
```

**Expected Output:**
```
AA (values): [1, 2, 3, 4]
JA (col indices): [0, 1, 2, 3]
IA (row pointers): [0, 1, 2, 3, 4]

Reconstructed:
[1, 0, 0, 0]
[0, 2, 0, 0]
[0, 0, 3, 0]
[0, 0, 0, 4]
```

**Why This Output Occurs**: The CSR format stores only non-zero values (1,2,3,4), their column indices (0,1,2,3), and row pointers indicating where each row's values start. `IA[1] = 1` means row 0's values are at indices 0-0 in `AA`. This saves memory: 3 arrays of 4 elements vs. 16 dense elements.

### Real-World Cases

- **Scientific Computing**: Finite element method stiffness matrices
- **Graph Algorithms**: Sparse adjacency matrices for social networks
- **Machine Learning**: Sparse feature matrices for text classification
- **Recommendation Systems**: User-item interaction matrices

### References

- HAL Science - Sparse Matrix Storage Formats - https://theses.hal.science/tel-02918210/file/These_ACheikAhamed_2015.pdf
- GNU Scientific Library - Sparse Matrix Formats - https://gitweb.git.savannah.gnu.org/gitweb/?p=gsl.git;a=blob_plain;f=doc/spmatrix.rst;hb=HEAD

---

## References (Consolidated)

- IIT Kanpur - Multi-dimensional Arrays - https://www.cgs.iitk.ac.in/user/hk/csd101/2021/inClass/l14.pdf
- GitHub - Two Pointers: Transpose and Reverse Technique - https://raw.githubusercontent.com/Stewie-pixel/claude-with-leetcode/refs/heads/main/c/48-rotate-image/ANALYSIS.md
- LeetCode - Concise Spiral Matrix Implementation - https://leetcode.com/problems/spiral-matrix/solutions/20573/a-concise-c-implementation-based-on-dire-zls9/
- GitHub - Prefix Sum Technique - https://raw.githubusercontent.com/pluginagentmarketplace/custom-plugin-data-structures-algorithms/refs/heads/main/skills/arrays/prefix-sum.md
- NASA - Dynamic Programming on Grid Lattice Graphs - https://ntrs.nasa.gov/api/citations/19880010064/downloads/19880010064.pdf
- HAL Science - Sparse Matrix Storage Formats - https://theses.hal.science/tel-02918210/file/These_ACheikAhamed_2015.pdf
- Educative - Matrix Traversals and Mapping - https://www.educative.io/courses/beginner-to-advanced-computing-and-logic-building/lta/problem-solving-matrix-traversals-i
- NIELIT - Anti-diagonal of Square Matrix - http://nielit.gov.in/gorakhpur/sites/default/files/Gorakhpur/olevel_2_C_Language_15May2020_RJS.pdf
- GitHub - Matrix Rotation Patterns - https://github.com/sajidhamdard/dsa/commit/348d65aa6e876a4bf0afaa2537c880528d40364f
- Educative - Spiral Matrix Algorithm - https://www.educative.io/answers/spiral-matrix-algorithm
- Algorithm Anthology - 2D Difference Arrays - https://raw.githubusercontent.com/alxli/algorithm-anthology/master/Book/A3C5-v1.0.pdf
- IIT Guwahati - Dynamic Programming on Diagonals - http://www.iitg.ac.in/psm/indexing_ma353/y09/LectureNoteMA515Oct12.pdf
- GNU Scientific Library - Sparse Matrix Formats - https://gitweb.git.savannah.gnu.org/gitweb/?p=gsl.git;a=blob_plain;f=doc/spmatrix.rst;hb=HEAD
- LeetCode - 2D Matrix & Grid Patterns Cheat Sheet - https://leetcode.com/discuss/post/8533041/set-28-2d-matrix-grid-patterns-python-ds-76aw/
- GitHub - Graph DFS on Grids - https://git.mashibing.com/msb-public/tech-interview-handbook/raw/commit/01bcbfc49f9be65ba182469e5f55c2fd5a56d8f7/utilities/python/graph_dfs.py