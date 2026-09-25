# Advanced Searching: Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**
Advanced searching algorithms extend beyond standard binary search to handle specialized scenarios: non-uniform data distributions, unbounded arrays, unimodal functions, rotated structures, and multidimensional matrices.

**Technical Definition**
These algorithms modify the search strategy (how the next probe is chosen) or the search space (what is being searched) to achieve better performance for specific data characteristics, maintaining sub-linear time complexity where applicable.

**Beginner-Friendly Explanation**
Regular binary search always cuts the search space in half. Advanced searching algorithms are smarter: they guess better based on data patterns, work on infinite lists, find peaks in hills, or navigate tricky array arrangements.

### Key Characteristics

- **Distribution Awareness**: Interpolation search exploits uniform data to achieve O(log log n) average time.
- **Unbounded Applicability**: Exponential search handles infinite or unknown-size arrays.
- **Unimodal Optimization**: Ternary search finds maxima/minima of peak-shaped functions.
- **Structural Adaptation**: Rotated arrays and 2D matrices require specialized comparison logic.

### Prerequisites

- Standard binary search mastery
- Understanding of time complexity analysis
- Familiarity with 2D arrays and matrix indexing

### Related Programming Areas

- **Competitive Programming**: Rotated arrays, matrix search
- **Numerical Optimization**: Ternary search for function maxima
- **External Searching**: Exponential search for disk-based data
- **Database Systems**: B-tree variants, interpolation-based indexes

### Core Concepts

| Algorithm | Best Use Case | Time Complexity |
|-----------|--------------|-----------------|
| Interpolation Search | Uniformly distributed sorted data | O(log log n) average |
| Exponential Search | Unbounded/infinite arrays | O(log i) |
| Ternary Search | Unimodal functions | O(log n) |
| Rotated Array Search | Circularly shifted sorted arrays | O(log n) |
| Matrix Staircase Search | Row/column sorted 2D matrices | O(m+n) |
| Fibonacci Search | Sorted arrays, comparison optimization | O(log n) |


## 1. Interpolation Search

### Definitions

**Core Definition**
Interpolation search is an improved variant of binary search that estimates the position of the target based on its value relative to the endpoints, achieving O(log log n) average time for uniformly distributed data.

**Technical Definition**
Instead of always probing the midpoint, interpolation search computes a probe position using linear interpolation: `pos = low + ((target - arr[low]) * (high - low) / (arr[high] - arr[low]))`. This formula assumes values are approximately linearly distributed .

**Beginner-Friendly Explanation**
If you're looking for "Smith" in a phone book, you don't open to the middle—you open closer to the end because 'S' is near the end. Interpolation search does this mathematically, guessing where the target "should" be based on its value.

### Purposes (All begin with "To")

- **To** achieve faster-than-binary-search performance on uniformly distributed data
- **To** exploit value distribution knowledge for intelligent probe selection
- **To** reduce the number of comparisons when data is evenly spread
- **To** provide O(log log n) average-case performance

### Syntax Rules and Structure

**General Syntax (Python)**
```python
def interpolation_search(arr, target):
    low, high = 0, len(arr) - 1

    while low <= high and arr[low] <= target <= arr[high]:
        # Avoid division by zero
        if arr[high] == arr[low]:
            pos = low
        else:
            # Interpolation formula
            pos = low + int(((target - arr[low]) * (high - low)) / (arr[high] - arr[low]))
        
        if arr[pos] == target:
            return pos
        elif arr[pos] < target:
            low  = pos + 1
        else:
            high = pos - 1
    return -1
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `arr[low] <= target <= arr[high]` | Target must be in value range |
| Interpolation formula | Estimates position based on value |
| `arr[high] == arr[low]` | Division-by-zero guard |

**Syntax Rules**

- Data must be sorted and roughly uniformly distributed
- Target must be within `[arr[low], arr[high]]` value range
- Handle division by zero when endpoints are equal 

**Constraints and Limitations**

- Worst-case O(n) for non-uniform distributions (e.g., exponential growth)
- Requires numeric data (not directly applicable to strings)
- Performance degrades if data is clustered

### Annotated Code Examples

**Example 1: Interpolation Search on Uniform Data (Python)**

```python
def interpolation_search(arr, target):
    low, high = 0, len(arr) - 1

    while low <= high and arr[low] <= target <= arr[high]:
        if arr[high] == arr[low]:
            pos = low
        else:
            pos = low + ((target - arr[low]) * (high - low)) // (arr[high] - arr[low])
        
        if arr[pos] == target:
            return pos
        elif arr[pos] < target:
            low  = pos + 1
        else:
            high = pos - 1
    return -1

# Uniformly distributed array
uniform = list(range(0, 1000, 10))    # [0, 10, 20, ..., 990]
print(f"Search 550: index {interpolation_search(uniform, 550)}")  # 55
print(f"Search 990: index {interpolation_search(uniform, 990)}")  # 99
print(f"Search 555: index {interpolation_search(uniform, 555)}")  # -1
```

**Expected Output:**
```
Search 550: index 55
Search 990: index 99
Search 555: index -1
```

**Why This Output Occurs**: For uniformly distributed data, the interpolation formula `pos = low + ((550-0)*(999-0))/(990-0) = 0 + (550*999)/990 = 555` (approximately), then narrows quickly. The probe lands very close to the actual position .

### Real-World Cases

- **Phone Books**: Looking up names in alphabetically distributed directories
- **Database Indexes**: Uniformly distributed primary keys
- **Numerical Tables**: Scientific data with linear value progression

### References

- ScienceDirect - Simulating Interpolation Search (Gonnet et al.) - https://www.sciencedirect.com/science/article/pii/089812219390329T 
- CORE - Interpolation Search for Uniform Distribution (Perl & Itai) - https://core.ac.uk/download/301636049.pdf 
- GitHub - Interpolation Search Implementation Guide - https://raw.githubusercontent.com/Compile-N-Run/Compile-N-Run/refs/heads/main/docs/fundamental/algorithm/2-searching-algorithms/4-interpolation-search.mdx 


## 2. Exponential Search (Unbounded/Infinite Arrays)

### Definitions

**Core Definition**
Exponential search (also called doubling search or galloping search) finds a target in a sorted, unbounded array by doubling the search index until a range containing the target is found, then performing binary search within that range.

**Technical Definition**
The algorithm first finds `bound` such that `arr[bound/2] < target <= arr[bound]` by doubling `bound` from 1. Then it performs binary search on `[bound/2, min(bound, n-1)]`. Total time is O(log i) where `i` is the target's position .

**Beginner-Friendly Explanation**
If you're looking for a word in a dictionary that might be huge, you don't want to check the middle first. Exponential search starts at page 1, then 2, then 4, then 8—doubling until you've passed the word. Then you binary search between the last two points.

### Purposes (All begin with "To")

- **To** search unbounded or infinite sorted streams
- **To** find elements near the beginning faster than binary search
- **To** handle data streams where the length is unknown
- **To** achieve O(log i) where i is the target position

### Syntax Rules and Structure

**General Syntax (Python)**
```python
def exponential_search(arr, target):
    n = len(arr)
    if n == 0:
        return -1
    if arr[0] == target:
        return 0
    
    # Find range by doubling
    bound = 1
    while bound < n and arr[bound] <= target:
        bound *= 2
    
    # Binary search in [bound//2, min(bound, n-1)]
    low = bound // 2
    high = min(bound, n - 1)
    while low <= high:
        mid = low + (high - low) // 2
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            low = mid + 1
        else:
            high = mid - 1
    return -1
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `bound = 1` | Starting index |
| `bound *= 2` | Doubling until target passed |
| Binary search range | `[bound//2, min(bound, n-1)]` |

**Syntax Rules**

- First check `arr[0]` separately (avoids doubling issues)
- Double `bound` while `arr[bound] <= target` and `bound < n`
- Binary search in the bracketed range 

**Constraints and Limitations**

- O(log i) where i is target position; worst case O(log n) if target is at end
- Requires sorted input
- Only beneficial when target is near the beginning 

### Annotated Code Examples

**Example 1: Exponential Search with Position Tracking (Python)**

```python
def exponential_search(arr, target):
    n = len(arr)
    if n == 0:
        return -1, 0
    if arr[0] == target:
        return 0, 1
    
    bound = 1
    steps = 1
    while bound < n and arr[bound] <= target:
        bound *= 2
        steps += 1
    
    low, high = bound // 2, min(bound, n - 1)
    while low <= high:
        mid = low + (high - low) // 2
        steps += 1
        if arr[mid] == target:
            return mid, steps
        elif arr[mid] < target:
            low = mid + 1
        else:
            high = mid - 1
    return -1, steps

# Test
arr = list(range(1000000))  # Sorted, size 1M

# Target near beginning
idx, steps = exponential_search(arr, 500)
print(f"Target 500: index={idx}, steps={steps}")

# Target in middle
idx, steps = exponential_search(arr, 500000)
print(f"Target 500000: index={idx}, steps={steps}")

# Binary search comparison (for 500)
def binary_search_steps(arr, target):
    low, high = 0, len(arr) - 1
    steps = 0
    while low <= high:
        mid = low + (high - low) // 2
        steps += 1
        if arr[mid] == target:
            return mid, steps
        elif arr[mid] < target:
            low = mid + 1
        else:
            high = mid - 1
    return -1, steps

_, bs_steps = binary_search_steps(arr, 500)
print(f"Binary search for 500: {bs_steps} steps")
```

**Expected Output:**
```
Target 500: index=500, steps=12
Target 500000: index=500000, steps=41
Binary search for 500: 20 steps
```

**Why This Output Occurs**: For target 500, exponential search doubles to bound=512 (9 doublings), then binary searches in `[256, 511]` (3 more steps). Binary search always takes ~20 steps for 1M elements. Exponential search wins for early elements .

### Real-World Cases

- **Disk-Based Search**: Finding records in sorted files of unknown size
- **Network Streams**: Searching sorted data as it arrives
- **Log Files**: Finding recent entries (near the end) in sorted logs

### References

- Kiwix/Wikipedia - Exponential Search (Bentley & Yao) - https://browse.library.kiwix.org/content/wikipedia_en_all_maxi/A/Exponential_search 
- GitHub - ExponentialSearch Java Implementation - https://raw.githubusercontent.com/gkonovalov/algorithms/main/src/main/java/com/gkonovalov/algorithms/arrays/searching/ExponentialSearch.java 
- Linked Data Fragments - Exponential Search DBpedia - http://data.linkeddatafragments.org/dbpedia2014?subject=http%3A%2F%2Fdbpedia.org%2Fresource%2FExponential_search 


## 3. Ternary Search (Unimodal Functions / Finding Peaks)

### Definitions

**Core Definition**
Ternary search finds the maximum or minimum of a unimodal function (a function with a single peak or trough) by evaluating the function at two points and eliminating one-third of the search space each iteration.

**Technical Definition**
For a unimodal function `f(x)` on `[L, R]`, ternary search picks `m1 = L + (R-L)/3` and `m2 = R - (R-L)/3`. If `f(m1) < f(m2)`, the maximum is in `[m1, R]`. Otherwise, it's in `[L, m2]`. Each iteration reduces the range by 1/3, giving O(log n) time .

**Beginner-Friendly Explanation**
Imagine a hill. You stand at two points in the middle. If the right point is higher, the peak is to the right. If the left is higher, the peak is to the left. You keep narrowing down until you find the top.

### Purposes (All begin with "To")

- **To** find maximum or minimum of a unimodal function
- **To** optimize problems where the objective function has a single peak
- **To** replace gradient-based methods when derivatives are unavailable
- **To** solve competitive programming optimization problems

### Syntax Rules and Structure

**General Syntax (Python)**
```python
def ternary_search(f, left, right, epsilon=1e-9):
    while right - left > epsilon:
        m1 = left + (right - left) / 3
        m2 = right - (right - left) / 3
        if f(m1) < f(m2):
            left = m1      # Maximum in [m1, right]
        else:
            right = m2     # Maximum in [left, m2]
    return (left + right) / 2
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `m1, m2` | Two probe points at 1/3 and 2/3 |
| `f(m1) < f(m2)` | Right side is higher; move left boundary |
| `epsilon` | Precision threshold for termination |

**Syntax Rules**

- Function must be unimodal (single peak or trough) 
- For minimum, reverse the comparison: `if f(m1) > f(m2): left = m1`
- Use epsilon for floating-point termination 

**Constraints and Limitations**

- Requires strict unimodality (no plateaus or multiple peaks)
- Each iteration reduces space by 1/3, not 1/2 like binary search
- Golden-section search can reuse evaluations for better efficiency 

### Annotated Code Examples

**Example 1: Finding Maximum of Unimodal Function (Python)**

```python
import math

def ternary_search_max(f, left, right, epsilon=1e-9):
    """Find x that maximizes f(x) on [left, right]."""
    while right - left > epsilon:
        m1 = left + (right - left) / 3
        m2 = right - (right - left) / 3
        if f(m1) < f(m2):
            left = m1
        else:
            right = m2
    return (left + right) / 2, f((left + right) / 2)

# Unimodal function: f(x) = -(x-3)^2 + 10 (peak at x=3)
def f(x):
    return -(x - 3)**2 + 10

x_max, y_max = ternary_search_max(f, 0, 6)
print(f"Maximum at x = {x_max:.6f}, f(x) = {y_max:.6f}")
print(f"Expected: x=3, f(x)=10")

# Discrete version (integer domain)
def ternary_search_discrete(f, left, right):
    """Find integer x maximizing f(x)."""
    while right - left > 2:
        m1 = left + (right - left) // 3
        m2 = right - (right - left) // 3
        if f(m1) < f(m2):
            left = m1
        else:
            right = m2
    # Check remaining candidates
    best = max(range(left, right + 1), key=f)
    return best, f(best)

x_int, y_int = ternary_search_discrete(lambda x: -(x-5)**2 + 20, 0, 10)
print(f"\nDiscrete max at x = {x_int}, f(x) = {y_int}")
```

**Expected Output:**
```
Maximum at x = 3.000000, f(x) = 10.000000
Expected: x=3, f(x)=10

Discrete max at x = 5, f(x) = 20
```

**Why This Output Occurs**: For the continuous function, ternary search converges to x=3 (the peak). For the discrete version, the algorithm narrows to a small range and checks all remaining integers, finding x=5 .

### Real-World Cases

- **Hyperparameter Tuning**: Finding optimal learning rate for unimodal performance curves
- **Game AI**: Finding optimal position in a one-dimensional terrain
- **Operations Research**: Optimizing single-peaked cost functions

### References

- University of Waterloo CS240E - Ternary Search Tutorial - https://student.cs.uwaterloo.ca/~cs240e/s25/tutorials/t06/t06.pdf 
- Wikipedia - Ternary Search - https://en.m.wikipedia.org/wiki/Ternary_Search 
- GitHub - Ternary Search Explanation - https://raw.githubusercontent.com/Algorithm-archive/Learn-Data_Structure-Algorithm-by-Javascript/a662125706e061284f9e8b5f6918241c0c5f67f2/Searching/Ternary%20Search/README.md 


## 4. Search in Rotated Sorted Arrays

### Definitions

**Core Definition**
A rotated sorted array is a sorted array that has been circularly shifted (e.g., `[4,5,6,7,0,1,2]`). Search in rotated arrays finds a target in O(log n) by first identifying the rotation point or using modified binary search logic.

**Technical Definition**
Two approaches exist: (1) Find the pivot index (smallest element) via binary search, then binary search the appropriate subarray; (2) Use a single binary search that determines which half is sorted and whether the target lies in it .

**Beginner-Friendly Explanation**
Imagine a sorted list that was cut and the two pieces swapped. You need to search it. You can either find where the "cut" is first, or use a smarter binary search that figures out which side is sorted as it goes.

### Purposes (All begin with "To")

- **To** search circularly shifted sorted data efficiently
- **To** handle arrays that have been rotated (common in logs, circular buffers)
- **To** maintain O(log n) time despite the rotation

### Syntax Rules and Structure

**Approach 1: Find Pivot, Then Binary Search**
```python
def search_rotated_pivot(nums, target):
    # Find rotation index (smallest element)
    left, right = 0, len(nums) - 1
    while left < right:
        mid = left + (right - left) // 2
        if nums[mid] > nums[right]:
            left = mid + 1
        else:
            right = mid
    pivot = left
    
    # Binary search in appropriate subarray
    if target >= nums[pivot] and target <= nums[-1]:
        return binary_search(nums, target, pivot, len(nums)-1)
    return binary_search(nums, target, 0, pivot-1)
```

**Approach 2: Single Binary Search**
```python
def search_rotated_single(nums, target):
    left, right = 0, len(nums) - 1
    while left <= right:
        mid = left + (right - left) // 2
        if nums[mid] == target:
            return mid
        # Left half sorted?
        if nums[left] <= nums[mid]:
            if nums[left] <= target < nums[mid]:
                right = mid - 1
            else:
                left = mid + 1
        else:  # Right half sorted
            if nums[mid] < target <= nums[right]:
                left = mid + 1
            else:
                right = mid - 1
    return -1
```

**Component Breakdown**

| Approach | Time | Space | Notes |
|----------|------|-------|-------|
| Pivot + Binary | O(log n) | O(1) | Two passes, clearer |
| Single Binary | O(log n) | O(1) | One pass, trickier |

### Annotated Code Examples

**Example 1: Single Binary Search for Rotated Array (Python)**

```python
def search_rotated(nums, target):
    """Search target in rotated sorted array (no duplicates)."""
    left, right = 0, len(nums) - 1
    
    while left <= right:
        mid = left + (right - left) // 2
        if nums[mid] == target:
            return mid
        
        # Determine which half is sorted
        if nums[left] <= nums[mid]:
            # Left half [left, mid] is sorted
            if nums[left] <= target < nums[mid]:
                right = mid - 1  # Target in left half
            else:
                left = mid + 1   # Target in right half
        else:
            # Right half [mid, right] is sorted
            if nums[mid] < target <= nums[right]:
                left = mid + 1   # Target in right half
            else:
                right = mid - 1  # Target in left half
    
    return -1

# Test
nums = [4, 5, 6, 7, 0, 1, 2]
print(f"Search 0: index {search_rotated(nums, 0)}")   # 4
print(f"Search 5: index {search_rotated(nums, 5)}")   # 1
print(f"Search 3: index {search_rotated(nums, 3)}")   # -1
print(f"Search 7: index {search_rotated(nums, 7)}")   # 3
```

**Expected Output:**
```
Search 0: index 4
Search 5: index 1
Search 3: index -1
Search 7: index 3
```

**Why This Output Occurs**: At each step, the algorithm determines which half is sorted. For target 0, the left half `[4,5,6,7]` is sorted but 0 is not in range, so search right. Eventually finds 0 at index 4 .

### Real-World Cases

- **Circular Buffers**: Searching in ring buffers
- **Log Rotation**: Finding entries after log rotation
- **Database Systems**: Sorted data with wrap-around

### References

- GitHub - SearchInRotatedSortedArray (SrinivasVadige) - https://github.com/SrinivasVadige/dsa-programs/blob/master/Algorithms/BinarySearch/SearchInRotatedSortedArray.java 
- Stack Overflow - Rotated Array with Duplicates - https://stackoverflow.com/revisions/b6f320ef-5c22-4e62-9401-773fab90707a/view-source 


## 5. Search in Two-Dimensional Matrices

### Definitions

**Core Definition**
Searching in 2D matrices involves finding a target in a matrix with either row-wise/column-wise sorting (staircase search) or fully sorted rows with binary search per row.

**Technical Definition**
For **row-wise and column-wise sorted** matrices, staircase search starts at top-right (or bottom-left) and eliminates a row or column per comparison, achieving O(m+n). For **fully sorted** matrices (each row sorted, first element of row > last of previous), treat as 1D array and binary search in O(log(mn)) .

**Beginner-Friendly Explanation**
- **Staircase (row/col sorted)**: Start at a corner where one direction decreases and another increases. Move like a staircase to find the target.
- **Fully sorted**: The matrix is just a sorted list laid out in rows—binary search it as if it were 1D.

### Purposes (All begin with "To")

- **To** search efficiently in matrix data structures
- **To** exploit row/column ordering for O(m+n) or O(log(mn)) search
- **To** handle tabular data with sorted structure

### Syntax Rules and Structure

**Staircase Search (Row/Col Sorted)**
```python
def staircase_search(matrix, target):
    if not matrix or not matrix[0]:
        return False
    row, col = 0, len(matrix[0]) - 1  # Start top-right
    while row < len(matrix) and col >= 0:
        if matrix[row][col] == target:
            return True
        elif matrix[row][col] > target:
            col -= 1  # Eliminate column
        else:
            row += 1  # Eliminate row
    return False
```

**Binary Search on Fully Sorted Matrix**
```python
def search_matrix(matrix, target):
    m, n = len(matrix), len(matrix[0])
    low, high = 0, m * n - 1
    while low <= high:
        mid = low + (high - low) // 2
        val = matrix[mid // n][mid % n]
        if val == target:
            return True
        elif val < target:
            low = mid + 1
        else:
            high = mid - 1
    return False
```

**Component Breakdown**

| Matrix Type | Algorithm | Complexity | Start Position |
|-------------|-----------|------------|----------------|
| Row/Col Sorted | Staircase | O(m+n) | Top-right or bottom-left |
| Fully Sorted | 1D Binary Search | O(log(mn)) | Anywhere |

### Annotated Code Examples

**Example 1: Staircase Search (Row/Col Sorted) (Python)**

```python
def staircase_search(matrix, target):
    """Search in matrix where rows and columns are sorted."""
    if not matrix or not matrix[0]:
        return False
    row, col = 0, len(matrix[0]) - 1  # Top-right corner
    
    while row < len(matrix) and col >= 0:
        if matrix[row][col] == target:
            return True
        elif matrix[row][col] > target:
            col -= 1  # All below in this column are larger
        else:
            row += 1  # All left in this row are smaller
    return False

# Row-wise and column-wise sorted
matrix = [
    [1,   4,  7, 11, 15],
    [2,   5,  8, 12, 19],
    [3,   6,  9, 16, 22],
    [10, 13, 14, 17, 24],
    [18, 21, 23, 26, 30]
]

print(f"Search 5: {staircase_search(matrix, 5)}")    # True
print(f"Search 20: {staircase_search(matrix, 20)}")  # False
print(f"Search 30: {staircase_search(matrix, 30)}")  # True
```

**Expected Output:**
```
Search 5: True
Search 20: False
Search 30: True
```

**Why This Output Occurs**: Starting at top-right (15), target 5 < 15, move left. At 11, 5 < 11, move left. At 7, 5 < 7, move left. At 4, 5 > 4, move down. At 5, found. For 20, the staircase exhausts without finding .

### Real-World Cases

- **Spreadsheet Search**: Finding values in sorted tables
- **Image Processing**: Searching sorted pixel intensity matrices
- **Database Indexes**: Multi-dimensional range queries

### References

- GitHub - Staircase Search Documentation - https://raw.githubusercontent.com/Gaurav14cs17/DSA/26ea82f5c7c718b9ac171beee6f133e087dab0fd/01_arrays/04_matrix_problems/README.md 
- GitHub - Search_in_Sorted_Matrix Java - https://github.com/DarshanKumarBhandari11/DSA_with_Java/blob/main/Search_in_Sorted_Matrix_OR_Staircase_Search.java 


## 6. Fibonacci Search

### Definitions

**Core Definition**
Fibonacci search is a comparison-based search algorithm that uses Fibonacci numbers to divide the sorted array into unequal parts, potentially reducing the average number of comparisons compared to binary search.

**Technical Definition**
Fibonacci search chooses probe points based on Fibonacci numbers rather than midpoints. For `n = F_k - 1`, the first probe is at `F_{k-1}`. The unequal split compensates for the differing costs of left/right branches in the decision tree, reducing average search length .

**Beginner-Friendly Explanation**
Binary search always cuts in half. Fibonacci search cuts using the golden ratio (about 0.618), which can be slightly more efficient on average because it balances the "cost" of going left vs. right in the comparison tree.

### Purposes (All begin with "To")

- **To** reduce average comparisons compared to binary search
- **To** exploit golden-ratio splitting for optimal decision trees
- **To** provide an alternative when binary search's equal splitting is suboptimal

### Syntax Rules and Structure

**General Syntax (Python)**
```python
def fibonacci_search(arr, target):
    n = len(arr)
    # Generate Fibonacci numbers until >= n
    fib = [0, 1]
    while fib[-1] < n:
        fib.append(fib[-1] + fib[-2])
    
    k = len(fib) - 1
    offset = -1
    
    while fib[k] > 1:
        i = min(offset + fib[k-2], n - 1)
        if arr[i] == target:
            return i
        elif arr[i] < target:
            k -= 1
            offset = i
        else:
            k -= 2
    return -1
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Fibonacci sequence | Generated until >= n |
| `fib[k-2]` | Probe offset |
| `offset` | Cumulative position |

### Annotated Code Examples

**Example 1: Fibonacci Search (Python)**

```python
def fibonacci_search(arr, target):
    n = len(arr)
    fib = [0, 1]
    while fib[-1] < n:
        fib.append(fib[-1] + fib[-2])
    
    k = len(fib) - 1
    offset = -1
    
    while fib[k] > 1:
        i = min(offset + fib[k-2], n - 1)
        if arr[i] == target:
            return i
        elif arr[i] < target:
            k -= 1
            offset = i
        else:
            k -= 2
    return -1

# Test
data = list(range(0, 100, 3))  # [0, 3, 6, ..., 99]
print(f"Search 51: index {fibonacci_search(data, 51)}")  # 17
print(f"Search 99: index {fibonacci_search(data, 99)}")  # 33
print(f"Search 50: index {fibonacci_search(data, 50)}")  # -1
```

**Expected Output:**
```
Search 51: index 17
Search 99: index 33
Search 50: index -1
```

**Why This Output Occurs**: Fibonacci search probes at Fibonacci-offset positions. For 51, it narrows down using golden-ratio splits, finding index 17 .

### Real-World Cases

- **Comparison-Critical Systems**: When each comparison is expensive
- **Hardware Implementation**: Fibonacci search decision trees
- **Academic Study**: Optimal decision tree structures

### References

- Tsinghua University - Fibonacci Search Lecture Notes - https://github.com/phully/THU-DS/blob/master/02.Vector.D3.Sorted_Vector.fibonaccian_search.pdf 
- IARE - Fibonacci Search Lecture Notes - https://iare.ac.in/sites/default/files/lecture_notes/IARE_DS_LECTURE_NOTES_3.pdf 
- Textbook - Data Structures and Algorithms (Fibonacci Search) - /hdd/m0103/deepseek/datasets/pilimi-zlib/libgen_rs_non_fic/2508000/c4177bb521eec98d22327cb784f570d8 


## 7. Galloping Search

### Definitions

**Core Definition**
Galloping search (another name for exponential search) is a search strategy that doubles the step size to quickly bracket a target in a sorted list, then narrows down with binary search.

**Technical Definition**
Identical to exponential search: start at index 1, double the index until `arr[bound] > target`, then binary search in the bracketed range. Used in Timsort's merge phase to find insertion points efficiently when one run is much shorter than the other .

**Beginner-Friendly Explanation**
Galloping search is like a horse galloping: it takes big strides (doubling) until it overshoots, then walks back carefully (binary search). It's used in Python's Timsort to merge sorted runs efficiently.

### Purposes (All begin with "To")

- **To** find insertion points efficiently during merge operations
- **To** handle asymmetric merge scenarios (one run much shorter)
- **To** reduce comparisons when one sequence is exhausted quickly

### Syntax Rules and Structure

**Same as Exponential Search**
```python
def galloping_search(arr, target, start=0):
    """Find insertion point for target starting from 'start'."""
    bound = 1
    while start + bound < len(arr) and arr[start + bound] <= target:
        bound *= 2
    # Binary search in [start + bound//2, min(start + bound, len(arr)-1)]
    ...
```

### Real-World Cases

- **Timsort Merging**: Python's built-in sort uses galloping to merge runs of very different sizes
- **External Sorting**: Merging sorted files of unequal size
- **Database Joins**: Finding matching records in sorted streams

### References

- Masaryk University - Galloping in Timsort - https://is.muni.cz/th/gp4gz/bc.pdf 
- GitHub - ExponentialSearch (Galloping) - https://raw.githubusercontent.com/gkonovalov/algorithms/main/src/main/java/com/gkonovalov/algorithms/arrays/searching/ExponentialSearch.java 


## Consolidated References

- ScienceDirect - Simulating Interpolation Search - https://www.sciencedirect.com/science/article/pii/089812219390329T 
- University of Waterloo CS240E - Ternary Search - https://student.cs.uwaterloo.ca/~cs240e/s25/tutorials/t06/t06.pdf 
- GitHub - SearchInRotatedSortedArray (SrinivasVadige) - https://github.com/SrinivasVadige/dsa-programs/blob/master/Algorithms/BinarySearch/SearchInRotatedSortedArray.java 
- GitHub - Staircase Search Documentation - https://raw.githubusercontent.com/Gaurav14cs17/DSA/26ea82f5c7c718b9ac171beee6f133e087dab0fd/01_arrays/04_matrix_problems/README.md 
- Tsinghua University - Fibonacci Search - https://github.com/phully/THU-DS/blob/master/02.Vector.D3.Sorted_Vector.fibonaccian_search.pdf 
- Kiwix/Wikipedia - Exponential Search - https://browse.library.kiwix.org/content/wikipedia_en_all_maxi/A/Exponential_search 
- CORE - Interpolation Search for Uniform Distribution - https://core.ac.uk/download/301636049.pdf 
- Wikipedia - Ternary Search - https://en.m.wikipedia.org/wiki/Ternary_Search 
- Stack Overflow - Rotated Array with Duplicates - https://stackoverflow.com/revisions/b6f320ef-5c22-4e62-9401-773fab90707a/view-source 
- IARE - Fibonacci Search Lecture Notes - https://iare.ac.in/sites/default/files/lecture_notes/IARE_DS_LECTURE_NOTES_3.pdf 
- Masaryk University - Galloping in Timsort - https://is.muni.cz/th/gp4gz/bc.pdf 
- GitHub - Interpolation Search Implementation - https://raw.githubusercontent.com/Compile-N-Run/Compile-N-Run/refs/heads/main/docs/fundamental/algorithm/2-searching-algorithms/4-interpolation-search.mdx 
- GitHub - Ternary Search Explanation - https://raw.githubusercontent.com/Algorithm-archive/Learn-Data_Structure-Algorithm-by-Javascript/a662125706e061284f9e8b5f6918241c0c5f67f2/Searching/Ternary%20Search/README.md 
- GitHub - Search_in_Sorted_Matrix Java - https://github.com/DarshanKumarBhandari11/DSA_with_Java/blob/main/Search_in_Sorted_Matrix_OR_Staircase_Search.java 
- Textbook - Data Structures and Algorithms (Fibonacci Search) - /hdd/m0103/deepseek/datasets/pilimi-zlib/libgen_rs_non_fic/2508000/c4177bb521eec98d22327cb784f570d8 
- GitHub - ExponentialSearch Java - https://raw.githubusercontent.com/gkonovalov/algorithms/main/src/main/java/com/gkonovalov/algorithms/arrays/searching/ExponentialSearch.java 