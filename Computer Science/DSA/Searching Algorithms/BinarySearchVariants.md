# Binary Search Variants: Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**
Binary search variants are modifications of the standard binary search algorithm that handle specific requirements such as finding the first or last occurrence of a duplicate value, computing insertion positions, or counting frequencies—all while maintaining O(log n) time complexity.

**Technical Definition**
These variants modify the update logic upon finding a match (`nums[mid] == target`): instead of returning immediately, they continue searching either leftward (for first occurrence/lower bound) or rightward (for last occurrence/upper bound), recording the candidate index and narrowing the search interval accordingly.

**Beginner-Friendly Explanation**
Standard binary search finds *any* occurrence of a value. But if the value appears multiple times, you often need to know where the *first* one is, where the *last* one is, or where you'd insert a new value to keep the array sorted. These variants answer those questions.

### Key Characteristics

- **Bias-Based Search**: The only difference from standard binary search is the direction of the bias when a match is found .
- **Two Searches for Range**: Finding both first and last occurrence requires two binary searches (or one search for lower bound and one for upper bound) .
- **Insertion Point Semantics**: Lower/upper bound functions return the index where a value *would* be inserted, not necessarily where it exists .
- **Frequency via Bounds**: The count of a value equals `upper_bound - lower_bound` .

### Prerequisites

- Standard binary search implementation
- Understanding of sorted arrays and duplicate values
- Familiarity with interval management (`low`, `high`)

### Related Programming Areas

- **Standard Library Implementation**: C++ `std::lower_bound`, `std::upper_bound`, `std::equal_range`; Python `bisect_left`, `bisect_right` 
- **Database Indexing**: Finding ranges of duplicate keys
- **Competitive Programming**: Range queries on sorted data

### Core Concepts

| Variant | Purpose | Key Modification |
|---------|---------|------------------|
| First Occurrence | Find leftmost index of target | On match, record and search left |
| Last Occurrence | Find rightmost index of target | On match, record and search right |
| Lower Bound | First position where value can be inserted | Return `low` after search |
| Upper Bound | Last position where value can be inserted | Return `low` after search |
| Frequency Counting | Count occurrences of target | `upper_bound - lower_bound` |


## 1. First Occurrence

### Definitions

**Core Definition**
First occurrence binary search finds the leftmost index at which the target value appears in a sorted array.

**Technical Definition**
When `nums[mid] == target`, the algorithm records `mid` as a candidate and continues searching the **left half** (`high = mid - 1`) to find any earlier occurrence .

**Beginner-Friendly Explanation**
If the array is `[1, 2, 2, 2, 3]` and you search for `2`, the first occurrence is at index 1, not index 2 or 3.

### Purposes (All begin with "To")

- **To** find the leftmost position of a duplicate value in sorted data
- **To** provide the starting index for range extraction
- **To** enable efficient frequency counting when paired with last occurrence
- **To** implement `lower_bound` semantics where the target exists

### Syntax Rules and Structure

**General Syntax (Python)**
```python
def first_occurrence(nums, target):
    low, high = 0, len(nums) - 1
    result = -1
    while low <= high:
        mid = low + (high - low) // 2
        if nums[mid] == target:
            result = mid          # Record candidate
            high = mid - 1        # Search LEFT for earlier
        elif nums[mid] < target:
            low = mid + 1
        else:
            high = mid - 1
    return result
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `result` | Stores the leftmost match found so far |
| `high = mid - 1` | Bias toward left half on match |
| Return | Leftmost index, or -1 if absent |

### Annotated Code Examples

**Example 1: First Occurrence with Trace (Python)**

```python
def first_occurrence(nums, target):
    low, high = 0, len(nums) - 1
    result = -1
    while low <= high:
        mid = low + (high - low) // 2
        if nums[mid] == target:
            result = mid
            high = mid - 1  # Keep searching left
        elif nums[mid] < target:
            low = mid + 1
        else:
            high = mid - 1
    return result

# Demo
nums = [5, 7, 7, 8, 8, 10]
print(f"First occurrence of 8: index {first_occurrence(nums, 8)}")   # 3
print(f"First occurrence of 7: index {first_occurrence(nums, 7)}")   # 1
print(f"First occurrence of 99: index {first_occurrence(nums, 99)}") # -1
```

**Expected Output:**
```
First occurrence of 8: index 3
First occurrence of 7: index 1
First occurrence of 99: index -1
```

**Why This Output Occurs**: When `nums[4] == 8` is found, the algorithm records index 4 but continues searching left. It then finds `nums[3] == 8`, records index 3, and continues left until `low > high`. The final recorded index is 3—the leftmost occurrence .

### Real-World Cases

- **Log Analysis**: Finding the first error timestamp in sorted logs
- **Database Queries**: `MIN(id)` for duplicate keys
- **Competitive Programming**: Range query starting points

### References

- LeetCode - Find First and Last Position of Element in Sorted Array (Microsoft) - https://leetcode.com/discuss/post/6343514/microsoft-find-first-and-last-position-o-y2kf/ 
- NeetCode - Binary Search with Left Bias - https://neetcode.io/solutions/find-first-and-last-position-of-element-in-sorted-array 


## 2. Last Occurrence

### Definitions

**Core Definition**
Last occurrence binary search finds the rightmost index at which the target value appears in a sorted array.

**Technical Definition**
When `nums[mid] == target`, the algorithm records `mid` as a candidate and continues searching the **right half** (`low = mid + 1`) to find any later occurrence .

**Beginner-Friendly Explanation**
If the array is `[1, 2, 2, 2, 3]` and you search for `2`, the last occurrence is at index 3, not index 1 or 2.

### Purposes (All begin with "To")

- **To** find the rightmost position of a duplicate value
- **To** provide the ending index for range extraction
- **To** enable frequency counting when paired with first occurrence
- **To** implement upper-bound-like behavior where the target exists

### Syntax Rules and Structure

**General Syntax (Python)**
```python
def last_occurrence(nums, target):
    low, high = 0, len(nums) - 1
    result = -1
    while low <= high:
        mid = low + (high - low) // 2
        if nums[mid] == target:
            result = mid          # Record candidate
            low = mid + 1         # Search RIGHT for later
        elif nums[mid] < target:
            low = mid + 1
        else:
            high = mid - 1
    return result
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `result` | Stores the rightmost match found so far |
| `low = mid + 1` | Bias toward right half on match |
| Return | Rightmost index, or -1 if absent |

### Annotated Code Examples

**Example 1: Last Occurrence with Trace (Python)**

```python
def last_occurrence(nums, target):
    low, high = 0, len(nums) - 1
    result = -1
    while low <= high:
        mid = low + (high - low) // 2
        if nums[mid] == target:
            result = mid
            low = mid + 1  # Keep searching right
        elif nums[mid] < target:
            low = mid + 1
        else:
            high = mid - 1
    return result

# Demo
nums = [5, 7, 7, 8, 8, 10]
print(f"Last occurrence of 8: index {last_occurrence(nums, 8)}")   # 4
print(f"Last occurrence of 7: index {last_occurrence(nums, 7)}")   # 2
print(f"Last occurrence of 99: index {last_occurrence(nums, 99)}") # -1
```

**Expected Output:**
```
Last occurrence of 8: index 4
Last occurrence of 7: index 2
Last occurrence of 99: index -1
```

**Why This Output Occurs**: When `nums[3] == 8` is found, the algorithm records index 3 but continues right. It finds `nums[4] == 8`, records index 4, then continues until `low > high`. The final recorded index is 4—the rightmost occurrence .

### Real-World Cases

- **Log Analysis**: Finding the last error timestamp in sorted logs
- **Database Queries**: `MAX(id)` for duplicate keys
- **Competitive Programming**: Range query ending points

### References

- LeetCode - Find First and Last Position (Meta Variants) - https://leetcode.com/discuss/post/6389602/meta-variants-for-find-first-last-positi-tlnk/ 
- GitHub - First and Last Position of an Element in Sorted Array (Java) - https://raw.githubusercontent.com/Sanjanayadav07/Java-DSA-Coding-Ninja/refs/heads/main/Chapter%207%20-%20Binary%20search/First%20and%20Last%20Position%20of%20an%20Element%20In%20Sorted%20Array.md 


## 3. Lower Bound (std::lower_bound Equivalent)

### Definitions

**Core Definition**
Lower bound finds the first position in a sorted array where a value could be inserted without violating the sorted order—equivalent to the position of the first element **not less than** the target.

**Technical Definition**
`lower_bound(first, last, value)` returns an iterator to the first element `e` in `[first, last)` such that `e >= value` (or `comp(e, value)` is false). If all elements are less than `value`, it returns `last` .

**Beginner-Friendly Explanation**
If you want to insert the number 5 into `[1, 3, 7, 9]`, the lower bound is index 2 (before 7)—the first spot where 5 can go without breaking sorted order.

### Purposes (All begin with "To")

- **To** find the first position where a value can be inserted
- **To** provide the starting index for `equal_range`
- **To** enable efficient search for the leftmost occurrence (even if absent)
- **To** implement sorted list insertion in O(log n) search time

### Syntax Rules and Structure

**General Syntax (Python Equivalent)**
```python
def lower_bound(nums, target):
    low, high = 0, len(nums)
    while low < high:
        mid = low + (high - low) // 2
        if nums[mid] < target:
            low = mid + 1
        else:
            high = mid
    return low  # First index where nums[i] >= target
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `high = len(nums)` | Exclusive upper bound |
| `nums[mid] < target` | Move right if strictly less |
| Return `low` | First index where `nums[i] >= target` |

**C++ Reference**
```cpp
auto it = std::lower_bound(nums.begin(), nums.end(), target);
// Returns iterator to first element >= target
```

### Annotated Code Examples

**Example 1: Lower Bound Behavior (Python)**

```python
def lower_bound(nums, target):
    low, high = 0, len(nums)
    while low < high:
        mid = low + (high - low) // 2
        if nums[mid] < target:
            low = mid + 1
        else:
            high = mid
    return low

# Demo
nums = [1, 3, 3, 5, 7]
print(f"lower_bound for 3: {lower_bound(nums, 3)}")  # 1 (first 3)
print(f"lower_bound for 4: {lower_bound(nums, 4)}")  # 3 (before 5)
print(f"lower_bound for 0: {lower_bound(nums, 0)}")  # 0 (before 1)
print(f"lower_bound for 9: {lower_bound(nums, 9)}")  # 5 (end)
```

**Expected Output:**
```
lower_bound for 3: 1
lower_bound for 4: 3
lower_bound for 0: 0
lower_bound for 9: 5
```

**Why This Output Occurs**: For target 3, the first index where `nums[i] >= 3` is 1. For target 4 (absent), it's 3—the position before 5. For 0, it's 0. For 9 (larger than all), it's the length 5 .

### Real-World Cases

- **Sorted Insertion**: Finding where to insert a new value in a sorted list
- **C++ STL**: `std::lower_bound` used in `equal_range`, `set` operations
- **Python**: `bisect_left` provides identical behavior 

### References

- C++ Standard Proposal P1718R0 - lower_bound/upper_bound semantics - https://rap.no/JTC1/SC22/WG21/docs/papers/2019/p1718r0.pdf 
- Oracle Help Center - lower_bound and upper_bound - https://docs.oracle.com/cd/E19205-01/820-2985/general/14_5.htm 
- Python Documentation - bisect_left - https://docs.python.org/3.11/library/bisect.html 


## 4. Upper Bound (std::upper_bound Equivalent)

### Definitions

**Core Definition**
Upper bound finds the first position in a sorted array where a value would be inserted **after** all existing elements equal to the target—equivalent to the position of the first element **greater than** the target.

**Technical Definition**
`upper_bound(first, last, value)` returns an iterator to the first element `e` in `[first, last)` such that `e > value` (or `comp(value, e)` is true). If no such element exists, returns `last` .

**Beginner-Friendly Explanation**
If you want to insert the number 5 into `[1, 3, 5, 5, 9]`, the upper bound is index 4 (before 9)—the first spot *after* all existing 5s.

### Purposes (All begin with "To")

- **To** find the last position where a value can be inserted
- **To** provide the ending index for `equal_range`
- **To** enable frequency counting via `upper_bound - lower_bound`
- **To** implement sorted list insertion after all duplicates

### Syntax Rules and Structure

**General Syntax (Python Equivalent)**
```python
def upper_bound(nums, target):
    low, high = 0, len(nums)
    while low < high:
        mid = low + (high - low) // 2
        if nums[mid] <= target:   # <= instead of <
            low = mid + 1
        else:
            high = mid
    return low  # First index where nums[i] > target
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `nums[mid] <= target` | Move right if less OR equal |
| Return `low` | First index where `nums[i] > target` |

**C++ Reference**
```cpp
auto it = std::upper_bound(nums.begin(), nums.end(), target);
// Returns iterator to first element > target
```

### Annotated Code Examples

**Example 1: Upper Bound Behavior (Python)**

```python
def upper_bound(nums, target):
    low, high = 0, len(nums)
    while low < high:
        mid = low + (high - low) // 2
        if nums[mid] <= target:
            low = mid + 1
        else:
            high = mid
    return low

# Demo
nums = [1, 3, 3, 5, 7]
print(f"upper_bound for 3: {upper_bound(nums, 3)}")  # 3 (after last 3)
print(f"upper_bound for 4: {upper_bound(nums, 4)}")  # 3 (before 5)
print(f"upper_bound for 0: {upper_bound(nums, 0)}")  # 0 (before 1)
print(f"upper_bound for 9: {upper_bound(nums, 9)}")  # 5 (end)
```

**Expected Output:**
```
upper_bound for 3: 3
upper_bound for 4: 3
upper_bound for 0: 0
upper_bound for 9: 5
```

**Why This Output Occurs**: For target 3, the first index where `nums[i] > 3` is 3 (after both 3s). For target 4 (absent), it's also 3—the position before 5. For 9, it's 5 .

### Real-World Cases

- **Frequency Counting**: `upper_bound - lower_bound` gives count
- **C++ STL**: `std::upper_bound` used in `equal_range`
- **Python**: `bisect_right` provides identical behavior 

### References

- Hungarian C++ Textbook - lower_bound and upper_bound examples - https://archive.org/download/bme-vik-konyvek/Szoftverfejleszt%C3%A9s%20Cpp%20nyelven%20-%20Levendovszky%20Tiham%C3%A9r%2C%20Benedek%20Zolt%C3%A1n_text.pdf 
- Python Documentation - bisect_right - https://docs.python.org/3.11/library/bisect.html 


## 5. Frequency Counting

### Definitions

**Core Definition**
Frequency counting uses binary search to determine how many times a target value appears in a sorted array by computing `last_occurrence - first_occurrence + 1` or `upper_bound - lower_bound`.

**Technical Definition**
Since duplicates are contiguous in a sorted array, the count of a value equals the number of elements in the half-open interval `[lower_bound, upper_bound)`. This requires two O(log n) binary searches .

**Beginner-Friendly Explanation**
If the array is `[1, 2, 2, 2, 3]` and you search for `2`, you find the first at index 1 and the last at index 3. The count is 3 - 1 + 1 = 3 occurrences.

### Purposes (All begin with "To")

- **To** count occurrences of a value in O(log n) instead of O(n)
- **To** answer range count queries on sorted data
- **To** provide the count without scanning all duplicates
- **To** enable statistical analysis on sorted datasets

### Syntax Rules and Structure

**General Syntax (Python)**
```python
def count_occurrences(nums, target):
    first = first_occurrence(nums, target)
    if first == -1:
        return 0
    last = last_occurrence(nums, target)
    return last - first + 1
```

**Alternative (Using Lower/Upper Bound)**
```python
def count_occurrences(nums, target):
    return upper_bound(nums, target) - lower_bound(nums, target)
```

### Annotated Code Examples

**Example 1: Frequency Counting with Bounds (Python)**

```python
def lower_bound(nums, target):
    low, high = 0, len(nums)
    while low < high:
        mid = low + (high - low) // 2
        if nums[mid] < target: low = mid + 1
        else: high = mid
    return low

def upper_bound(nums, target):
    low, high = 0, len(nums)
    while low < high:
        mid = low + (high - low) // 2
        if nums[mid] <= target: low = mid + 1
        else: high = mid
    return low

def count_occurrences(nums, target):
    return upper_bound(nums, target) - lower_bound(nums, target)

# Demo
nums = [1, 2, 2, 2, 3, 3, 3, 3, 4]
print(f"Count of 2: {count_occurrences(nums, 2)}")  # 3
print(f"Count of 3: {count_occurrences(nums, 3)}")  # 4
print(f"Count of 5: {count_occurrences(nums, 5)}")  # 0
```

**Expected Output:**
```
Count of 2: 3
Count of 3: 4
Count of 5: 0
```

**Why This Output Occurs**: `lower_bound(2) = 1`, `upper_bound(2) = 4`, count = 3. For 3: `lower = 4`, `upper = 8`, count = 4. For absent 5: `lower = upper = 9`, count = 0 .

### Real-World Cases

- **Analytics**: Counting occurrences of events in sorted logs
- **Database**: `COUNT(*)` for indexed keys
- **Meta Interview Variant**: Count of element using binary search 

### References

- Stack Overflow - Efficient way to count occurrences in sorted array - https://stackoverflow.com/questions/4322327/efficient-way-to-count-occurrences-of-a-key-in-a-sorted-array 
- LeetCode - Meta Variants (Count of Element) - https://leetcode.com/discuss/post/6389602/meta-variants-for-find-first-last-positi-tlnk/ 


## 6. Search Insertion Position

### Definitions

**Core Definition**
Search insertion position finds the index where a target value should be inserted into a sorted array to maintain order—equivalent to `lower_bound` when the target is absent, and the target's index when present.

**Technical Definition**
This is exactly the `lower_bound` operation: return the first index `i` such that `nums[i] >= target`. If the target exists, this is the first occurrence; if absent, it's the insertion point .

**Beginner-Friendly Explanation**
If you have `[1, 3, 5, 6]` and want to insert 4, it goes at index 2 (before 5). If you insert 5, it goes at index 2 (where 5 already is).

### Purposes (All begin with "To")

- **To** find where to insert a value while keeping the array sorted
- **To** implement `bisect_left` or `std::lower_bound` semantics
- **To** avoid O(n) insertion position search
- **To** enable binary search-based sorted data structures

### Syntax Rules and Structure

**General Syntax (Python)**
```python
def search_insert(nums, target):
    low, high = 0, len(nums)
    while low < high:
        mid = low + (high - low) // 2
        if nums[mid] < target:
            low = mid + 1
        else:
            high = mid
    return low
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `high = len(nums)` | Exclusive upper bound |
| `nums[mid] < target` | Move right if strictly less |
| Return `low` | First index where `nums[i] >= target` |

### Annotated Code Examples

**Example 1: Insertion Position (Python)**

```python
def search_insert(nums, target):
    low, high = 0, len(nums)
    while low < high:
        mid = low + (high - low) // 2
        if nums[mid] < target:
            low = mid + 1
        else:
            high = mid
    return low

# Demo
nums = [1, 3, 5, 6]
print(f"Insert 5 at index: {search_insert(nums, 5)}")  # 2 (5 exists)
print(f"Insert 2 at index: {search_insert(nums, 2)}")  # 1 (between 1,3)
print(f"Insert 7 at index: {search_insert(nums, 7)}")  # 4 (end)
print(f"Insert 0 at index: {search_insert(nums, 0)}")  # 0 (before 1)
```

**Expected Output:**
```
Insert 5 at index: 2
Insert 2 at index: 1
Insert 7 at index: 4
Insert 0 at index: 0
```

**Why This Output Occurs**: For target 5 (exists), the first index where `nums[i] >= 5` is 2. For target 2 (absent), it's 1. For 7, it's 4. For 0, it's 0 .

### Real-World Cases

- **Sorted Insertion**: Maintaining a sorted list without re-sorting
- **Python `bisect.insort`**: Uses `bisect_left` to find position 
- **C++ `std::lower_bound`**: Standard library insertion position

### References

- GitHub - Search Insert Position using lowerBound - https://raw.githubusercontent.com/stevenjlho/leetcode/refs/heads/main/solutions/34.%20Find%20First%20and%20Last%20Position%20of%20Element%20in%20Sorted%20Array/README.md 
- Programming Classics Textbook - Search-Insert Algorithm - /hf3fs-jd/hdd/deepseek/shared/kaidong/datasets/pilimi-zlib/ia20241105/annas_archive_data__aacid__ia2_acsmpdf_files__20240701T025359Z--20240701T025400Z/aacid__ia2_acsmpdf_files__20240701T025359Z__XpL6q3yjmZvstWmkVpgsSw 


## Consolidated References

- C++ Standard Proposal P1718R0 - lower_bound/upper_bound semantics - https://rap.no/JTC1/SC22/WG21/docs/papers/2019/p1718r0.pdf 
- LeetCode - Microsoft Find First and Last Position (Bias Binary Search) - https://leetcode.com/discuss/post/6343514/microsoft-find-first-and-last-position-o-y2kf/ 
- Python Documentation - bisect module (bisect_left, bisect_right) - https://docs.python.org/3.11/library/bisect.html 
- LeetCode - Meta Variants for LC34 (Count of Element) - https://leetcode.com/discuss/post/6389602/meta-variants-for-find-first-last-positi-tlnk/ 
- NeetCode - LC34 Solution (Left/Right Bias) - https://neetcode.io/solutions/find-first-and-last-position-of-element-in-sorted-array 
- Oracle Help Center - lower_bound and upper_bound - https://docs.oracle.com/cd/E19205-01/820-2985/general/14_5.htm 
- GitHub - First and Last Position (Java) - https://raw.githubusercontent.com/Sanjanayadav07/Java-DSA-Coding-Ninja/refs/heads/main/Chapter%207%20-%20Binary%20search/First%20and%20Last%20Position%20of%20an%20Element%20In%20Sorted%20Array.md 
- Stack Overflow - Count Occurrences in Sorted Array - https://stackoverflow.com/questions/4322327/efficient-way-to-count-occurrences-of-a-key-in-a-sorted-array 
- Hungarian C++ Textbook - lower_bound/upper_bound example - https://archive.org/download/bme-vik-konyvek/Szoftverfejleszt%C3%A9s%20Cpp%20nyelven%20-%20Levendovszky%20Tiham%C3%A9r%2C%20Benedek%20Zolt%C3%A1n_text.pdf 
- GitHub - Search Insert Position via lowerBound - https://raw.githubusercontent.com/stevenjlho/leetcode/refs/heads/main/solutions/34.%20Find%20First%20and%20Last%20Position%20of%20Element%20in%20Sorted%20Array/README.md 
- GitHub - LeetCode-Go Binary Search Template - https://deepwiki.com/halfrost/LeetCode-Go/1.1-how-to-use-this-repository 
- C++ Standard N4382 - nth_element and partition requirements - https://www.rap.no/JTC1/SC22/WG21/docs/papers/2015/n4382.pdf 
- GitHub - FindFirstAndLastPosition C++ Solution - https://raw.githubusercontent.com/The-EleetCoder/A2Z-DSA-Solutions/refs/heads/main/04_BinarySearch/01_BinarySearchOn1DArrays/06_FindFirstAndLastPositionOfElementInSortedArray.cpp 
- Python Documentation (Chinese) - bisect module - https://docs.python.org/zh-cn/3.14/library/bisect.html 
- LeetCode - LC34 Python Solution (Bias) - https://leetcode.com/problems/find-first-and-last-position-of-element-in-sorted-array/solutions/3918198/LC-34-Python/ 
- Programming Classics Textbook - Search-Insert Algorithm - /hf3fs-jd/hdd/deepseek/shared/kaidong/datasets/pilimi-zlib/ia20241105/annas_archive_data__aacid__ia2_acsmpdf_files__20240701T025359Z--20240701T025400Z/aacid__ia2_acsmpdf_files__20240701T025359Z__XpL6q3yjmZvstWmkVpgsSw