# DSA Core Array Algorithmic Patterns: Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**
Array algorithmic patterns are reusable computational strategies that exploit the contiguous, indexed nature of arrays to solve common problems—such as finding pairs, contiguous subarrays, or cumulative ranges—in optimal time and space.

**Technical Definition**
These patterns constitute template solutions characterized by specific pointer movement invariants, window maintenance rules, or precomputation schemes that reduce naive O(n²) approaches to O(n) or O(n log n) while maintaining O(1) or O(n) auxiliary space.

**Beginner-Friendly Explanation**
Think of array patterns as recipes. Instead of inventing a new way to cook every time, you recognize "this problem looks like a stir-fry" and apply the stir-fry technique. Each pattern has a specific situation where it works best.

### Key Characteristics

- **Reusability**: Each pattern applies to a broad class of problems with similar structural cues
- **Optimal Complexity**: Patterns typically reduce brute-force O(n²) or O(n³) to O(n) or O(n log n)
- **In-Place Capability**: Most patterns operate with O(1) extra space
- **Recognition-Based**: Success depends on identifying problem cues that match a known pattern

### Prerequisites

- Static and dynamic array fundamentals
- Zero-based indexing and bounds checking
- Big O notation and complexity analysis
- Loop invariants and conditional logic

### Related Programming Areas

- **Competitive Programming**: Interview problems and algorithmic contests
- **Systems Programming**: Buffer manipulation, memory scanning
- **Data Processing**: Stream analytics, batch aggregation
- **Computational Geometry**: Sweep-line algorithms

### Core Concepts / Features

| Pattern | Primary Use Case | Complexity |
|---------|-----------------|------------|
| Two Pointers | Pairs, palindromes, partitioning | O(n) |
| Sliding Window | Contiguous subarrays, substrings | O(n) |
| Binary Search | Sorted array search | O(log n) |
| Prefix/Suffix Sums | Range queries | O(1) query after O(n) precompute |
| Frequency Arrays | Counting, duplicates | O(n) |
| Difference Arrays | Range updates | O(1) update after O(n) precompute |
| Array Rotation | Cyclic shifts | O(n) |
| Kadane's Algorithm | Maximum subarray | O(n) |

---

## 1. Two-Pointer Technique (Opposite Ends and Same Direction)

### Definitions

**Core Definition**
The two-pointer technique maintains two indices that traverse an array, either from opposite ends moving inward or from the same side moving in the same direction, to solve search, pairing, and partitioning problems in a single pass.

**Technical Definition**
This pattern uses two integer indices with a movement invariant that guarantees each step eliminates at least one candidate, yielding O(n) time and O(1) extra space .

**Beginner-Friendly Explanation**
Imagine reading a book with two fingers—one at the beginning and one at the end. You move your fingers toward each other based on what you find, and you never need to go back.

### Purposes (All begin with "To")

- **To** find pairs or triplets satisfying a condition in sorted arrays
- **To** remove duplicates or partition elements in-place
- **To** check palindromes or reverse array portions
- **To** reduce nested-loop O(n²) solutions to O(n)

### Syntax Rules and Structure

**General Syntax (Opposite Direction)**
```
left = 0
right = length - 1
while left < right:
    current = arr[left] + arr[right]
    if current == target:
        return (left, right)
    elif current < target:
        left += 1
    else:
        right -= 1
```

**General Syntax (Same Direction)**
```
slow = 0
for fast in range(length):
    if isValid(arr[fast]):
        arr[slow] = arr[fast]
        slow += 1
return slow
```

**Component Breakdown**

| Component | Description | Opposite | Same Direction |
|-----------|-------------|----------|----------------|
| `left`/`slow` | Leading or marking pointer | Starts at 0 | Starts at 0 |
| `right`/`fast` | Trailing or scanning pointer | Starts at end | Starts at 0 |
| Movement | Pointer advancement rule | Based on comparison | Based on validity |
| Invariant | Guaranteed property | Sorted order maintained | Valid prefix maintained |

**Syntax Rules**

- Opposite direction requires sorted input to decide pointer movement 
- Same direction works on unsorted input for partitioning and duplicate removal 
- Both pointers must move monotonically (never backward)
- Loop terminates when pointers meet or scan completes

**Constraints and Limitations**

- Opposite direction: array must be sorted (or have monotonic property)
- Same direction: only applies to problems with an in-place validity criterion
- Two pointers alone cannot solve problems requiring unordered pair search (use hash map instead)

### Annotated Code Examples

**Example 1: Two Sum II (Sorted Array, Opposite Direction)**
```python
def two_sum_sorted(numbers, target):
    """
    Find two numbers in a sorted array that sum to target.
    Returns 1-based indices (as per LeetCode convention).
    """
    left = 0                    # Start pointer at beginning
    right = len(numbers) - 1    # End pointer at last element
    
    while left < right:         # Continue until pointers meet
        current_sum = numbers[left] + numbers[right]
        
        if current_sum == target:
            # Found the pair; return 1-based indices
            return [left + 1, right + 1]
        elif current_sum < target:
            # Sum too small: need larger number
            # Move left pointer right to increase sum
            left += 1
        else:
            # Sum too large: need smaller number
            # Move right pointer left to decrease sum
            right -= 1
    
    return []  # No pair found

# Test cases
print(two_sum_sorted([1, 2, 3, 4, 6], 6))   # Expected: [2, 4] (2+4=6)
print(two_sum_sorted([2, 7, 11, 15], 9))    # Expected: [1, 2] (2+7=9)
print(two_sum_sorted([1, 3, 4, 5], 100))    # Expected: []
```

**Expected Output:**
```
[2, 4]
[1, 2]
[]
```

**Why This Output Occurs**: The array is sorted, so `numbers[left]` is the smallest unused value and `numbers[right]` is the largest. If the sum is too small, only moving `left` right can increase it. If too large, only moving `right` left can decrease it. Each comparison eliminates at least one candidate, giving O(n) time .

**Example 2: Remove Duplicates from Sorted Array (Same Direction)**
```python
def remove_duplicates(nums):
    """
    Remove duplicates in-place from sorted array.
    Returns new length of unique elements.
    """
    if not nums:
        return 0
    
    slow = 0  # Marks position of last unique element
    
    for fast in range(1, len(nums)):
        # If current element differs from last unique
        if nums[fast] != nums[slow]:
            slow += 1              # Advance unique boundary
            nums[slow] = nums[fast]  # Write unique value
    
    # slow + 1 is the count of unique elements
    return slow + 1

# Test cases
nums1 = [1, 1, 2]
k1 = remove_duplicates(nums1)
print(f"k = {k1}, nums = {nums1[:k1]}")

nums2 = [0, 0, 1, 1, 1, 2, 2, 3, 3, 4]
k2 = remove_duplicates(nums2)
print(f"k = {k2}, nums = {nums2[:k2]}")
```

**Expected Output:**
```
k = 2, nums = [1, 2]
k = 5, nums = [0, 1, 2, 3, 4]
```

**Why This Output Occurs**: The `slow` pointer marks the boundary of unique elements. The `fast` pointer scans ahead. When a new unique value is found, it's written at `slow + 1`. The relative order is preserved because we only move forward .

### Real-World Cases

- **Database Merge Joins**: Merging two sorted lists with opposite-direction pointers
- **Text Processing**: Palindrome checking for string validation
- **Memory Compaction**: Removing invalid entries in-place
- **Container Optimization**: Finding pairs in sorted trade data

### References

- GitHub - Two Pointers Technique - https://raw.githubusercontent.com/pertrai1/coding-challenges/e6b3f21417c61df7acc728a6b785c002d81e8da9/docs/techniques/TWO_POINTERS.md
- GitHub - Two Pointers Pattern - https://raw.githubusercontent.com/thuva4/Algorithms/refs/heads/master/patterns/two-pointers.md

---

## 2. Sliding Window Technique (Fixed and Dynamic Sizes)

### Definitions

**Core Definition**
The sliding window technique maintains a contiguous subarray (the "window") that slides through the array, expanding and contracting based on problem constraints, to solve subarray problems in O(n) time.

**Technical Definition**
A window defined by `[left, right]` indices is maintained such that elements within satisfy a given property, with `right` advancing to include new elements and `left` advancing to restore validity when constraints are violated .

**Beginner-Friendly Explanation**
Imagine looking at a train through a window. The window can be a fixed size or can stretch and shrink depending on what you need to see. You slide it along the train without ever looking back.

### Purposes (All begin with "To")

- **To** find contiguous subarrays satisfying sum, length, or content constraints
- **To** compute running statistics over consecutive elements
- **To** solve substring problems in strings (longest, shortest, containing all)
- **To** avoid recomputing overlapping subarray properties

### Syntax Rules and Structure

**General Syntax (Variable Window)**
```
left = 0
for right in range(length):
    # Expand window by including arr[right]
    add(arr[right])
    
    # Shrink window while invalid
    while not valid():
        remove(arr[left])
        left += 1
    
    # Window [left, right] is now valid
    update_best()
```

**General Syntax (Fixed Window)**
```
window_size = k
# Compute first window
for i in range(window_size):
    add(arr[i])
process_window()

# Slide window
for i in range(window_size, length):
    add(arr[i])           # Include new element
    remove(arr[i - window_size])  # Exclude old element
    process_window()
```

**Component Breakdown**

| Component | Description | Fixed | Variable |
|-----------|-------------|-------|----------|
| `left` | Window start | Moves with `right` | Moves when invalid |
| `right` | Window end | Iterates all | Iterates all |
| Window size | Number of elements | Constant `k` | Varies |
| Shrink condition | When to move `left` | Automatic | Problem-specific |

**Syntax Rules**

- Fixed window: both pointers move together, maintaining constant size 
- Variable window: `right` always advances; `left` advances only when needed
- Window invariant must be clearly defined (e.g., sum ≤ target, all distinct)
- Best value updated after window becomes valid

**Constraints and Limitations**

- Works only for contiguous subarray/substring problems
- Window validity condition must be monotonic (adding elements never makes an invalid window valid)
- O(n) time relies on each element being added and removed at most once

### Annotated Code Examples

**Example 1: Minimum Size Subarray Sum (Dynamic Window)**
```python
def min_subarray_len(target, nums):
    """
    Find minimum length of contiguous subarray with sum >= target.
    Returns 0 if no such subarray exists.
    """
    left = 0
    current_sum = 0
    min_length = float('inf')
    
    for right in range(len(nums)):
        # Expand: add element at right
        current_sum += nums[right]
        
        # Shrink: while sum >= target, try to minimize window
        while current_sum >= target:
            # Update minimum length found so far
            min_length = min(min_length, right - left + 1)
            
            # Remove leftmost element to try smaller window
            current_sum -= nums[left]
            left += 1
    
    return 0 if min_length == float('inf') else min_length

# Test cases
print(min_subarray_len(7, [2, 3, 1, 2, 4, 3]))  # Expected: 2 (subarray [4,3])
print(min_subarray_len(4, [1, 4, 4]))             # Expected: 1 (subarray [4])
print(min_subarray_len(11, [1, 1, 1, 1, 1]))      # Expected: 0 (impossible)
```

**Expected Output:**
```
2
1
0
```

**Why This Output Occurs**: The `right` pointer expands the window until sum ≥ target. Then `left` shrinks it while maintaining validity, recording the smallest valid window. For `[2,3,1,2,4,3]` with target 7, the window `[4,3]` has sum 7 and length 2, which is minimal .

**Example 2: Maximum Sum of Fixed-Size Window**
```python
def max_sum_fixed_window(nums, k):
    """
    Find maximum sum of any contiguous subarray of size k.
    """
    if len(nums) < k:
        return None
    
    # Compute sum of first window
    window_sum = sum(nums[:k])
    max_sum = window_sum
    
    # Slide window: subtract outgoing, add incoming
    for i in range(k, len(nums)):
        window_sum = window_sum - nums[i - k] + nums[i]
        max_sum = max(max_sum, window_sum)
    
    return max_sum

# Test cases
print(max_sum_fixed_window([2, 1, 5, 1, 3, 2], 3))  # Expected: 9 ([5,1,3])
print(max_sum_fixed_window([1, 2, 3, 4, 5], 2))     # Expected: 9 ([4,5])
```

**Expected Output:**
```
9
9
```

**Why This Output Occurs**: For fixed window size, the window slides one position at a time. Each slide removes the leftmost element and adds the new rightmost element in O(1), avoiding recomputation of the entire window sum .

### Real-World Cases

- **Network Monitoring**: Maximum packets in any time window
- **Finance**: Best k-day moving average
- **String Matching**: Longest substring without repeating characters
- **Signal Processing**: Moving average filters

### References

- GitHub - Sliding Window Technique - https://raw.githubusercontent.com/Sirros/leetcode/master/thinkings/slide-window.en.en.md

---

## 3. Binary Search on Sorted Arrays

### Definitions

**Core Definition**
Binary search repeatedly divides a sorted array in half, comparing the target against the middle element to determine which half to search next, achieving O(log n) time.

**Technical Definition**
Binary search maintains a search interval `[lo, hi)` with the invariant that the target, if present, lies within this interval. Each comparison eliminates half the remaining elements .

**Beginner-Friendly Explanation**
Think of looking up a word in a dictionary. You open to the middle, see if your word comes before or after, then repeat with the correct half. You never look at the other half.

### Purposes (All begin with "To")

- **To** locate an element in a sorted array in logarithmic time
- **To** find insertion points for maintaining sorted order
- **To** search rotated sorted arrays (with modifications)
- **To** solve optimization problems with monotonic predicates

### Syntax Rules and Structure

**General Syntax (Iterative)**
```
lo = 0
hi = length
while lo < hi:
    mid = (lo + hi) // 2
    if arr[mid] == target:
        return mid
    elif arr[mid] < target:
        lo = mid + 1
    else:
        hi = mid
return -1  # Not found
```

**Component Breakdown**

| Component | Description | Example |
|-----------|-------------|---------|
| `lo` | Lower bound (inclusive) | 0 |
| `hi` | Upper bound (exclusive) | `len(arr)` |
| `mid` | Middle index | `(lo + hi) // 2` |
| Comparison | Determines search half | `<`, `==`, `>` |

**Syntax Rules**

- Array must be sorted in ascending order 
- `mid` calculation must avoid integer overflow in fixed-width languages (`lo + (hi - lo) // 2`)
- The `hi` bound is exclusive in the half-open interval convention
- Return value is index if found, -1 or insertion point if not

**Constraints and Limitations**

- Requires sorted input (O(n log n) to sort if not already)
- Only works on arrays with O(1) random access
- Modifications needed for rotated arrays, duplicates, or 2D matrices

### Annotated Code Examples

**Example 1: Classic Binary Search**
```python
def binary_search(arr, target):
    """
    Search for target in sorted array.
    Returns index if found, -1 otherwise.
    """
    lo = 0
    hi = len(arr)  # Exclusive upper bound
    
    while lo < hi:
        mid = (lo + hi) // 2  # Middle index
        
        if arr[mid] == target:
            return mid         # Found target
        elif arr[mid] < target:
            lo = mid + 1       # Search right half
        else:
            hi = mid           # Search left half
    
    return -1  # Target not in array

# Test cases
sorted_arr = [12, 15, 33, 35, 42, 45, 51, 62, 73, 75, 86, 98]
print(binary_search(sorted_arr, 42))   # Expected: 4
print(binary_search(sorted_arr, 12))   # Expected: 0
print(binary_search(sorted_arr, 98))   # Expected: 11
print(binary_search(sorted_arr, 50))   # Expected: -1
```

**Expected Output:**
```
4
0
11
-1
```

**Why This Output Occurs**: Each iteration compares the target to the middle element. For target 42, the search path is: mid=5 (45), 42<45, hi=5; mid=2 (33), 42>33, lo=3; mid=4 (42), found. This takes ⌈log₂(12)⌉ = 4 comparisons .

### Real-World Cases

- **Database Indexing**: B-tree search for record lookup
- **Version Control**: Finding first bad commit (git bisect)
- **Numerical Methods**: Root finding on sorted function values
- **Autocomplete**: Finding prefix ranges in sorted dictionaries

### References

- Princeton University - Sorting and Searching Lecture - https://www.cs.princeton.edu/courses/archive/spring13/cos234/www/lib/exe/fetch.php?media=cos233-234-lecture3-search-sort.pptx.pdf
- Cornell University - Divide and Conquer: Binary Search - https://www.cs.cornell.edu/courses/cs1112/2009sp/Notes/egL25/L25post.pdf

---

## 4. Prefix Sums and Suffix Sums

### Definitions

**Core Definition**
A prefix sum array stores cumulative sums from the start of the array up to each index, enabling O(1) range sum queries after O(n) precomputation.

**Technical Definition**
Prefix sum `P[k] = sum(arr[0..k-1])` with `P[0] = 0`. The sum of subarray `arr[x..y]` equals `P[y+1] - P[x]` .

**Beginner-Friendly Explanation**
If you're adding up numbers as you walk along a road, the prefix sum at each point is the running total so far. To find the sum between two points, just subtract the totals.

### Purposes (All begin with "To")

- **To** answer range sum queries in constant time after linear precomputation
- **To** compute running averages or cumulative statistics
- **To** detect subarrays with specific sum properties (e.g., sum = k)
- **To** enable difference array techniques

### Syntax Rules and Structure

**General Syntax**
```
# Build prefix sums
P = [0] * (n + 1)
for k in range(1, n + 1):
    P[k] = P[k-1] + arr[k-1]

# Range sum query [x, y] inclusive
range_sum = P[y+1] - P[x]
```

**Component Breakdown**

| Component | Description | Value |
|-----------|-------------|-------|
| `P[0]` | Empty prefix | 0 |
| `P[k]` | Sum of first k elements | `P[k-1] + arr[k-1]` |
| `P[y+1] - P[x]` | Sum of `arr[x..y]` | Range query |
| Precompute time | Building P | O(n) |
| Query time | Each range sum | O(1) |

**Syntax Rules**

- `P` has length `n+1` to handle empty prefix and full array sum
- `P[0] = 0` is essential for correct range queries starting at 0
- Suffix sums follow the same principle from the right
- Prefix sums work with any associative operation (sum, product, min, max)

**Constraints and Limitations**

- Requires O(n) extra space for the prefix array
- Only useful for multiple range queries; single query is O(n) anyway
- Array must be immutable between queries (or rebuilt after updates)

### Annotated Code Examples

**Example 1: Range Sum Queries**
```python
def build_prefix_sums(arr):
    """Build prefix sum array."""
    n = len(arr)
    P = [0] * (n + 1)
    for k in range(1, n + 1):
        P[k] = P[k-1] + arr[k-1]
    return P

def range_sum(P, x, y):
    """Sum of arr[x..y] inclusive."""
    return P[y+1] - P[x]

# Test
arr = [2, 3, 7, 5, 1, 3, 9]
P = build_prefix_sums(arr)

print(f"Prefix sums: {P}")
print(f"Sum [0..2]: {range_sum(P, 0, 2)}")  # 2+3+7 = 12
print(f"Sum [2..5]: {range_sum(P, 2, 5)}")  # 7+5+1+3 = 16
print(f"Sum [4..6]: {range_sum(P, 4, 6)}")  # 1+3+9 = 13
```

**Expected Output:**
```
Prefix sums: [0, 2, 5, 12, 17, 18, 21, 30]
Sum [0..2]: 12
Sum [2..5]: 16
Sum [4..6]: 13
```

**Why This Output Occurs**: `P[3] = 12` (sum of first 3 elements), `P[0] = 0`. So `arr[0..2] = P[3] - P[0] = 12`. Each query is O(1) after the O(n) precomputation .

### Real-World Cases

- **Financial Analysis**: Cumulative returns or running balances
- **Image Processing**: Integral images for fast rectangle sums
- **Game Development**: Cumulative experience or score thresholds
- **Bioinformatics**: Genomic range queries

### References

- Codility - Prefix Sums Lesson - https://codility.com/media/train/3-PrefixSums.pdf
- CodeChef - Prefix and Suffix Sum Practice - https://www.codechef.com/practice/prefix-sums

---

## 5. Frequency Arrays and Hash Mapping

### Definitions

**Core Definition**
A frequency array (or hash map) counts occurrences of each distinct element, enabling O(1) lookup of how many times a value appears.

**Technical Definition**
For bounded integer domains, an array `freq[value]` stores the count. For unbounded or non-integer domains, a hash map maps element values to their frequencies .

**Beginner-Friendly Explanation**
Think of a tally sheet. Every time you see a value, you make a mark next to it. At the end, you know exactly how many times each value appeared.

### Purposes (All begin with "To")

- **To** detect duplicates or unique elements
- **To** check if two arrays are permutations (anagrams)
- **To** find most/least frequent elements
- **To** implement counting sort or bucket sort

### Syntax Rules and Structure

**General Syntax (Array)**
```
freq = [0] * (max_value + 1)
for x in arr:
    freq[x] += 1

count = freq[target]
```

**General Syntax (Hash Map)**
```
from collections import Counter
freq = Counter(arr)
count = freq[target]
```

**Component Breakdown**

| Component | Description | Array | Hash Map |
|-----------|-------------|-------|----------|
| Storage | Count per value | `freq[value]` | `freq[value]` |
| Lookup | Get count | O(1) | O(1) avg |
| Space | Total slots | O(max value) | O(distinct values) |
| Best for | Bounded integers | Unbounded/strings |

**Syntax Rules**

- Array-based: value must be a non-negative integer within bounds
- Hash-based: any hashable type (integers, strings, tuples)
- Initialize counts to 0 before counting
- Iterate over distinct keys, not all possible values

**Constraints and Limitations**

- Frequency array wastes space if value range is large but sparse
- Hash map has overhead but handles arbitrary keys
- Both require O(n) time to build

### Annotated Code Examples

**Example 1: Duplicate Detection**
```python
def has_duplicate(nums):
    """Check if array contains any duplicate."""
    seen = set()
    for num in nums:
        if num in seen:
            return True
        seen.add(num)
    return False

def find_duplicates(nums):
    """Find all elements that appear more than once."""
    from collections import Counter
    freq = Counter(nums)
    return [num for num, count in freq.items() if count > 1]

# Test cases
print(has_duplicate([1, 2, 3, 4]))      # False
print(has_duplicate([1, 2, 3, 1]))      # True
print(find_duplicates([1, 2, 3, 1, 2, 4]))  # [1, 2]
```

**Expected Output:**
```
False
True
[1, 2]
```

**Why This Output Occurs**: The set tracks seen elements. When a duplicate is encountered, it's already in the set. The Counter builds a frequency map, and elements with count > 1 are duplicates .

### Real-World Cases

- **Anagram Detection**: Comparing character frequencies
- **Voting Systems**: Counting votes per candidate
- **Network Analysis**: Packet counts per IP address
- **Text Analysis**: Word frequency in documents

### References

- Educative - Knowing What to Track Pattern - https://www.educative.io/courses/grokking-coding-interview-in-cpp/lta/introduction-to-knowing-what-to-track

---

## 6. Difference Arrays for Range Updates

### Definitions

**Core Definition**
A difference array encodes range updates as point updates at boundaries, enabling O(1) range updates followed by O(n) reconstruction via prefix sums.

**Technical Definition**
For array `a`, define `diff[i] = a[i] - a[i-1]` (with `diff[0] = a[0]`). A range update `a[l..r] += val` becomes `diff[l] += val` and `diff[r+1] -= val`. Reconstructing `a` requires a prefix sum over `diff` .

**Beginner-Friendly Explanation**
Instead of updating every element in a range, you just mark where the change starts and where it ends. Later, you sweep through once to apply all changes.

### Purposes (All begin with "To")

- **To** apply multiple range updates efficiently
- **To** answer point queries after many updates
- **To** solve offline range-update problems
- **To** implement sweep-line algorithms

### Syntax Rules and Structure

**General Syntax**
```
# Build difference array
diff = [0] * (n + 1)
diff[0] = arr[0]
for i in range(1, n):
    diff[i] = arr[i] - arr[i-1]

# Range update [l, r] += val
diff[l] += val
if r + 1 < n:
    diff[r+1] -= val

# Reconstruct: prefix sum
result = [0] * n
result[0] = diff[0]
for i in range(1, n):
    result[i] = result[i-1] + diff[i]
```

**Component Breakdown**

| Component | Description | Value |
|-----------|-------------|-------|
| `diff[l] += val` | Start of range update | Boundary mark |
| `diff[r+1] -= val` | End of range update | Cancel after range |
| Prefix sum | Reconstructs final values | O(n) |
| Update cost | Per range update | O(1) |

**Syntax Rules**

- `r+1` boundary update is skipped when `r == n-1` 
- Works for additive updates (for multiplication, use modular inverse)
- All updates can be applied first, then one prefix pass reconstructs
- For interleaved queries, use a Fenwick tree on the difference array 

**Constraints and Limitations**

- Only works offline (all updates before all queries) without additional data structure
- Additive operations only (sum, XOR); not for min/max unless monotonic
- Requires O(n) extra space for the difference array

### Annotated Code Examples

**Example 1: Range Add, Point Query (Offline)**
```python
def apply_range_updates(arr, updates):
    """
    Apply multiple range additions, then return final array.
    updates: list of (l, r, val) meaning arr[l..r] += val
    """
    n = len(arr)
    diff = [0] * (n + 1)
    
    # Initialize difference array from arr
    diff[0] = arr[0]
    for i in range(1, n):
        diff[i] = arr[i] - arr[i-1]
    
    # Apply each range update in O(1)
    for l, r, val in updates:
        diff[l] += val
        if r + 1 < n:
            diff[r+1] -= val
    
    # Reconstruct final array with prefix sum
    result = [0] * n
    result[0] = diff[0]
    for i in range(1, n):
        result[i] = result[i-1] + diff[i]
    
    return result

# Test
arr = [1, 2, 3, 4, 5]
updates = [(0, 2, 10), (1, 3, 5)]
# After: [11, 17, 18, 9, 5]
result = apply_range_updates(arr, updates)
print(result)
```

**Expected Output:**
```
[11, 17, 18, 9, 5]
```

**Why This Output Occurs**: Update `(0,2,10)` adds 10 to indices 0,1,2. Update `(1,3,5)` adds 5 to indices 1,2,3. Combined: index 0: 1+10=11; index 1: 2+10+5=17; index 2: 3+10+5=18; index 3: 4+5=9; index 4: 5. The difference array encodes all updates at boundaries, and one prefix pass reconstructs .

### Real-World Cases

- **Event Scheduling**: Adding attendees to time ranges
- **Resource Allocation**: Memory reservation over address ranges
- **Weather Data**: Temperature anomalies over date ranges
- **Competitive Programming**: CSES Range Update Queries

### References

- GitHub - Range Update Queries (Difference Arrays) - https://raw.githubusercontent.com/mtuann/competitive-programming-cpp/refs/heads/main/practice/ladders/foundations/difference-arrays/rangeupdatequeries.md

---

## 7. Array Rotation and Array Reversal

### Definitions

**Core Definition**
Array rotation shifts all elements by k positions, wrapping around. Array reversal inverts element order. The reversal algorithm rotates an array using three reversals.

**Technical Definition**
The reversal algorithm: to rotate left by k, reverse `[0..k-1]`, reverse `[k..n-1]`, then reverse the entire array .

**Beginner-Friendly Explanation**
To rotate a line of people, you can have the first group turn around, the second group turn around, then everyone turn around. The result is a rotation.

### Purposes (All begin with "To")

- **To** cyclically shift array elements in O(1) extra space
- **To** implement circular buffers and queues
- **To** solve rotation-based interview problems
- **To** enable block swap operations

### Syntax Rules and Structure

**General Syntax (Reversal Algorithm)**
```
# Rotate left by k positions
k = k % n  # Normalize

reverse(arr, 0, k-1)    # Reverse first k
reverse(arr, k, n-1)    # Reverse remaining
reverse(arr, 0, n-1)    # Reverse entire array
```

**Component Breakdown**

| Component | Description | Range |
|-----------|-------------|-------|
| Normalize | Handle k > n | `k = k % n` |
| First reverse | Left block | `[0, k-1]` |
| Second reverse | Right block | `[k, n-1]` |
| Third reverse | Whole array | `[0, n-1]` |

**Syntax Rules**

- Normalize `k` to avoid unnecessary rotations 
- Three reversals always produce the correct rotation
- Each reversal is O(length) and in-place
- Reversal algorithm is cache-friendly compared to juggling 

**Constraints and Limitations**

- O(n) time regardless of k
- Requires three passes over the array
- Alternative: block swap (Gries-Mills) may be faster for specific k values

### Annotated Code Examples

**Example 1: Rotate Array Left by k**
```python
def reverse(arr, start, end):
    """Reverse arr[start..end] in-place."""
    while start < end:
        arr[start], arr[end] = arr[end], arr[start]
        start += 1
        end -= 1

def rotate_left(arr, k):
    """Rotate array left by k positions using reversal algorithm."""
    n = len(arr)
    if n == 0:
        return arr
    
    k = k % n  # Normalize k
    if k == 0:
        return arr
    
    reverse(arr, 0, k-1)      # Reverse first k elements
    reverse(arr, k, n-1)      # Reverse remaining elements
    reverse(arr, 0, n-1)      # Reverse entire array
    
    return arr

# Test
arr = [1, 2, 3, 4, 5, 6, 7]
print(f"Original: {arr}")
print(f"Rotate left by 2: {rotate_left(arr.copy(), 2)}")
print(f"Rotate left by 3: {rotate_left(arr.copy(), 3)}")
```

**Expected Output:**
```
Original: [1, 2, 3, 4, 5, 6, 7]
Rotate left by 2: [3, 4, 5, 6, 7, 1, 2]
Rotate left by 3: [4, 5, 6, 7, 1, 2, 3]
```

**Why This Output Occurs**: For left rotation by 2: reverse `[1,2]` → `[2,1,3,4,5,6,7]`; reverse `[3,4,5,6,7]` → `[2,1,7,6,5,4,3]`; reverse all → `[3,4,5,6,7,1,2]`. The three reversals compose to a rotation .

### Real-World Cases

- **Circular Buffers**: Ring buffers in embedded systems
- **Cryptography**: Simple cipher operations
- **Image Processing**: Cyclic shifts of pixel rows
- **Load Balancing**: Round-robin scheduling

### References

- Wikipedia - Block Swap Algorithms - https://en.wikipedia.org/wiki/Block_swap_algorithms
- Stack Overflow - Cyclically Shift Array In-Place - https://stackoverflow.com/questions/13736683/how-can-i-cyclically-shift-an-array-without-additional-heap-memory

---

## 8. Rearrangement and Partitioning Algorithms

### Definitions

**Core Definition**
Partitioning rearranges elements into groups based on a predicate (e.g., negatives before positives, Dutch national flag three-way partition) without changing relative order within groups.

**Technical Definition**
The Dutch National Flag algorithm partitions an array into three regions using three pointers, while simpler partitions use two pointers for two-way splits.

**Beginner-Friendly Explanation**
Imagine sorting laundry into piles by color. Partitioning is doing this in-place without extra baskets.

### Purposes (All begin with "To")

- **To** separate elements by category in O(n) time
- **To** prepare arrays for further processing (e.g., quicksort partition)
- **To** implement stable or unstable partitioning as needed
- **To** solve classification problems without extra space

### Syntax Rules and Structure

**General Syntax (Two-Way Partition)**
```
left = 0
right = n - 1
while left <= right:
    while left <= right and predicate(arr[left]):
        left += 1
    while left <= right and not predicate(arr[right]):
        right -= 1
    if left <= right:
        swap(arr[left], arr[right])
        left += 1
        right -= 1
```

**Component Breakdown**

| Component | Description | Two-Way | Three-Way |
|-----------|-------------|---------|-----------|
| Pointers | Boundary markers | 2 | 3 |
| Regions | Partitions | 2 | 3 |
| Use case | Binary classification | Dutch flag |

### Annotated Code Examples

**Example 1: Move Zeroes (Same-Direction Partition)**
```python
def move_zeroes(nums):
    """
    Move all zeros to end while maintaining relative order
    of non-zero elements. In-place O(n) time.
    """
    slow = 0  # Position for next non-zero element
    
    for fast in range(len(nums)):
        if nums[fast] != 0:
            # Swap non-zero to slow position
            nums[slow], nums[fast] = nums[fast], nums[slow]
            slow += 1
    
    return nums

# Test
nums = [0, 1, 0, 3, 12]
print(move_zeroes(nums))  # [1, 3, 12, 0, 0]
```

**Expected Output:**
```
[1, 3, 12, 0, 0]
```

**Why This Output Occurs**: The `slow` pointer tracks where the next non-zero should go. When `fast` finds a non-zero, it swaps with `slow`, effectively moving zeros right. Relative order of non-zeros is preserved .

### Real-World Cases

- **Quicksort**: Partition around pivot
- **Data Cleaning**: Separating valid/invalid records
- **Color Sorting**: Dutch national flag problem
- **Load Balancing**: Separating light/heavy tasks

---

## 9. Duplicate Detection and Tracking

### Definitions

**Core Definition**
Duplicate detection identifies elements appearing more than once in an array, using frequency counting, sorting, or hash-based tracking.

**Technical Definition**
Detection can be O(n) with O(n) space (hash set) or O(n log n) with O(1) space (sort then adjacent compare), or O(n) with O(1) space for bounded ranges (cyclic sort).

**Beginner-Friendly Explanation**
Think of checking whether a guest list has duplicate names. You could mark each name as you see it, or sort the list and check neighbors.

### Purposes (All begin with "To")

- **To** validate data integrity (no duplicates allowed)
- **To** find all duplicate values
- **To** find the first duplicate or the only duplicate
- **To** prepare data for deduplication

### Annotated Code Examples

**Example 1: Find All Duplicates (Bounded Range, O(1) Space)**
```python
def find_duplicates_cyclic(nums):
    """
    Find all duplicates in array where 1 <= nums[i] <= n.
    Uses cyclic sort: place each value at its correct index.
    """
    duplicates = []
    i = 0
    n = len(nums)
    
    while i < n:
        correct_idx = nums[i] - 1  # Where this value should be
        if nums[i] != nums[correct_idx]:
            # Swap to correct position
            nums[i], nums[correct_idx] = nums[correct_idx], nums[i]
        else:
            i += 1
    
    # After sorting, indices where nums[i] != i+1 are duplicates
    for i in range(n):
        if nums[i] != i + 1:
            duplicates.append(nums[i])
    
    return duplicates

# Test
print(find_duplicates_cyclic([4, 3, 2, 7, 8, 2, 3, 1]))  # [2, 3]
```

**Expected Output:**
```
[2, 3]
```

**Why This Output Occurs**: Cyclic sort places each value at index `value-1`. After sorting, any position where `nums[i] != i+1` indicates a duplicate. This achieves O(n) time and O(1) space for bounded ranges.

### Real-World Cases

- **User Registration**: Detecting duplicate email addresses
- **Data Migration**: Finding duplicate records
- **Inventory**: Tracking duplicate serial numbers
- **Log Analysis**: Finding repeated event IDs

---

## 10. Maximum Subarray Sum Using Kadane's Algorithm

### Definitions

**Core Definition**
Kadane's algorithm finds the contiguous subarray with the maximum sum in O(n) time by tracking the best sum ending at each position and the global best.

**Technical Definition**
The recurrence is `max_ending_here = max(arr[i], max_ending_here + arr[i])` and `max_so_far = max(max_so_far, max_ending_here)` .

**Beginner-Friendly Explanation**
Imagine walking along a road with money in each spot. You can start collecting at any point. If your running total goes negative, you might as well start fresh at the next spot. Kadane's algorithm tracks the best total you could have ending at each spot.

### Purposes (All begin with "To")

- **To** find maximum sum contiguous subarray in linear time
- **To** solve stock profit maximization (one transaction)
- **To** analyze genomic sequences (maximum scoring segment)
- **To** provide dynamic programming template for subarray problems

### Syntax Rules and Structure

**General Syntax**
```
max_ending_here = arr[0]
max_so_far = arr[0]

for i in range(1, n):
    max_ending_here = max(arr[i], max_ending_here + arr[i])
    max_so_far = max(max_so_far, max_ending_here)

return max_so_far
```

**Component Breakdown**

| Component | Description | Value |
|-----------|-------------|-------|
| `max_ending_here` | Best sum ending at current position | Local optimum |
| `max_so_far` | Best sum seen anywhere | Global optimum |
| Recurrence | Decision: extend or restart | `max(arr[i], prev + arr[i])` |
| Time | Single pass | O(n) |

**Syntax Rules**

- Initialize both to `arr[0]` (not 0, to handle all-negative arrays) 
- At each step, either extend previous subarray or start new at current element
- Track global maximum separately from local maximum

**Constraints and Limitations**

- The algorithm as commonly implemented does not return the subarray indices (Kadane's intended version does) 
- For all-negative arrays, the standard variant returns the maximum single element, while Kadane's intended version returns 0 for empty subarray 

### Annotated Code Examples

**Example 1: Maximum Subarray Sum**
```python
def kadane(arr):
    """
    Find maximum sum of contiguous subarray using Kadane's algorithm.
    """
    if not arr:
        return 0
    
    max_ending_here = arr[0]  # Best sum ending at current position
    max_so_far = arr[0]       # Best sum found anywhere
    
    for i in range(1, len(arr)):
        # Either extend previous subarray or start new at arr[i]
        max_ending_here = max(arr[i], max_ending_here + arr[i])
        # Update global best
        max_so_far = max(max_so_far, max_ending_here)
    
    return max_so_far

# Test cases
print(kadane([-2, 1, -3, 4, -1, 2, 1, -5, 4]))  # Expected: 6 ([4,-1,2,1])
print(kadane([1, 2, 3, 4]))                        # Expected: 10 (entire array)
print(kadane([-5, -2, -8]))                        # Expected: -2 (max single)
```

**Expected Output:**
```
6
10
-2
```

**Why This Output Occurs**: For `[-2,1,-3,4,-1,2,1,-5,4]`, the optimal subarray is `[4,-1,2,1]` with sum 6. The algorithm correctly resets when the running sum becomes less than the current element alone. For all-negative input, it returns the largest (least negative) single element .

### Real-World Cases

- **Stock Trading**: Maximum profit from one buy-sell transaction
- **Genomics**: Maximum scoring segment in DNA sequences 
- **Signal Processing**: Peak energy segment detection
- **Image Processing**: Brightest region detection

### References

- OUCI - Two Kadane Algorithms for Maximum Sum Subarray - https://ouci.dntb.gov.ua/en/works/40yKEgpl/
- R Project - Fast Maximum Subarray Computation - https://search.r-project.org/CRAN/refmans/MESS/html/maximum_subarray.html

---

## References (Consolidated)

- GitHub - Two Pointers Technique - https://raw.githubusercontent.com/pertrai1/coding-challenges/e6b3f21417c61df7acc728a6b785c002d81e8da9/docs/techniques/TWO_POINTERS.md
- GitHub - Two Pointers Pattern - https://raw.githubusercontent.com/thuva4/Algorithms/refs/heads/master/patterns/two-pointers.md
- GitHub - Sliding Window Technique - https://raw.githubusercontent.com/Sirros/leetcode/master/thinkings/slide-window.en.en.md
- Princeton University - Sorting and Searching - https://www.cs.princeton.edu/courses/archive/spring13/cos234/www/lib/exe/fetch.php?media=cos233-234-lecture3-search-sort.pptx.pdf
- Cornell University - Divide and Conquer: Binary Search - https://www.cs.cornell.edu/courses/cs1112/2009sp/Notes/egL25/L25post.pdf
- Codility - Prefix Sums - https://codility.com/media/train/3-PrefixSums.pdf
- CodeChef - Prefix and Suffix Sum Practice - https://www.codechef.com/practice/prefix-sums
- Educative - Knowing What to Track - https://www.educative.io/courses/grokking-coding-interview-in-cpp/lta/introduction-to-knowing-what-to-track
- GitHub - Range Update Queries (Difference Arrays) - https://raw.githubusercontent.com/mtuann/competitive-programming-cpp/refs/heads/main/practice/ladders/foundations/difference-arrays/rangeupdatequeries.md
- Wikipedia - Block Swap Algorithms - https://en.wikipedia.org/wiki/Block_swap_algorithms
- Stack Overflow - Cyclically Shift Array In-Place - https://stackoverflow.com/questions/13736683/how-can-i-cyclically-shift-an-array-without-additional-heap-memory
- OUCI - Two Kadane Algorithms for Maximum Sum Subarray - https://ouci.dntb.gov.ua/en/works/40yKEgpl/
- R Project - Fast Maximum Subarray Computation - https://search.r-project.org/CRAN/refmans/MESS/html/maximum_subarray.html