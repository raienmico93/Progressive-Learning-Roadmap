# Recursive Problem Patterns: Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**
Recursive problem patterns are recurring structural templates where a problem can be solved by decomposing it into smaller instances of the same problem, solving those recursively, and combining their results.

**Technical Definition**
A recursive pattern is characterized by a recurrence relation that expresses the solution for input size \(n\) in terms of solutions for smaller inputs, combined with one or more base cases that terminate the recursion.

**Beginner-Friendly Explanation**
Recursive patterns are like recipes that say "to make a large cake, make two smaller cakes and stack them." Each pattern has a characteristic way of breaking the problem down and putting the answers back together.

### Key Characteristics

- **Self-Similar Structure**: The solution to a problem of size \(n\) depends on solutions to problems of smaller size.
- **Base Case Foundation**: Every pattern requires at least one trivial case that can be solved directly.
- **Combination Step**: Results from subproblems must be combined to form the final answer.
- **Call Stack Depth**: The pattern's recursion depth determines auxiliary space complexity.

### Prerequisites

- Understanding of recursion fundamentals (base case, recursive case, call stack)
- Basic knowledge of arrays, linked lists, and trees
- Familiarity with Big-O notation and recurrence relations

### Related Programming Areas

- **Algorithm Design**: Divide-and-conquer, dynamic programming, backtracking
- **Data Structure Operations**: Tree traversal, searching, sorting
- **Parsing**: JSON, XML, file systems, expressions
- **Artificial Intelligence**: Game trees, decision trees

### Core Concepts

| Pattern | Recursive Structure | Typical Complexity |
|---------|---------------------|-------------------|
| Factorial | Linear reduction (\(n \to n-1\)) | O(n) time, O(n) space |
| Fibonacci | Branching reduction (\(n \to n-1, n-2\)) | O(2^n) naive, O(n) memoized |
| Tree Traversal | Divide by subtrees | O(n) time, O(h) space |
| Divide-and-Conquer | Split, recurse, combine | O(n log n) for sorting |
| Recursive Search | Halve search space | O(log n) time |
| Nested Structures | Recurse into children | O(n) total nodes |


## 1. Factorial

### Definitions

**Core Definition**
The factorial of a non-negative integer \(n\), denoted \(n!\), is the product of all positive integers from 1 to \(n\), defined recursively as \(n! = n \times (n-1)!\) with base case \(0! = 1\) .

**Technical Definition**
The factorial recurrence is:
\[
F(n) = \begin{cases} 1 & \text{if } n \le 1 \\ n \times F(n-1) & \text{if } n > 1 \end{cases}
\]
This is a linear recursion with a single recursive call per invocation .

**Beginner-Friendly Explanation**
Factorial is the classic "countdown" recursion. To find 5!, you need 4!, which needs 3!, and so on down to 0!, which is 1 by definition. Then you multiply back up: 1 × 1 = 1, × 2 = 2, × 3 = 6, × 4 = 24, × 5 = 120 .

### Purposes (All begin with "To")

- **To** demonstrate the simplest linear recursion pattern
- **To** illustrate the relationship between iteration and recursion
- **To** serve as a building block for combinatorics and probability
- **To** introduce the concept of stack unwinding

### Syntax Rules and Structure

**General Syntax (Python/C++)**
```python
def factorial(n):
    if n <= 1:          # Base case
        return 1
    return n * factorial(n - 1)  # Recursive case
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Base case | `n <= 1` returns 1 |
| Reduction | `n - 1` |
| Combination | Multiply `n` by subresult |

### Annotated Code Examples

**Example 1: Factorial with Stack Trace (Python)**

```python
def factorial(n, depth=0):
    """Compute n! with indented trace to visualize recursion."""
    indent = "  " * depth
    print(f"{indent}factorial({n}) called")

    if n <= 1:
        print(f"{indent}  base case: return 1")
        return 1

    subresult = factorial(n - 1, depth + 1)
    result = n * subresult
    print(f"{indent}  combining: {n} * {subresult} = {result}")
    return result


print(f"\nFinal result: {factorial(5)}")
```

**Expected Output:**
```
factorial(5) called
  factorial(4) called
    factorial(3) called
      factorial(2) called
        factorial(1) called
          base case: return 1
        combining: 2 * 1 = 2
      combining: 3 * 2 = 6
    combining: 4 * 6 = 24
  combining: 5 * 24 = 120

Final result: 120
```

**Why This Output Occurs**: The recursion descends from 5 to 1 (stack growth), then unwinds from 1 back to 5 (stack unwinding). Each level multiplies its `n` by the result from the level below. The indentation shows the call stack depth at each point .

### Real-World Cases

- **Combinatorics**: Calculating permutations and combinations (\(\binom{n}{k} = \frac{n!}{k!(n-k)!}\))
- **Probability**: Computing probabilities in discrete distributions
- **Taylor Series**: Factorial appears in denominators of power series expansions

### References

- Vultr Docs - C++ Program to Calculate Factorial Using Recursion - https://docs.vultr.com/cpp/examples/calculate-factorial-of-a-number-using-recursion 
- OGM Materyal - Recursive Factorial Explanation - http://ogmmateryal.eba.gov.tr/panel/upload/etkilesimli/kitap/bilgisayarbilimikur1/sec/unite1/files/basic-html/page194.html 


## 2. Fibonacci

### Definitions

**Core Definition**
The Fibonacci sequence is defined by \(F(n) = F(n-1) + F(n-2)\) with base cases \(F(0) = 0\) and \(F(1) = 1\) .

**Technical Definition**
The Fibonacci recurrence is a binary branching recursion:
\[
F(n) = \begin{cases} n & \text{if } n < 2 \\ F(n-1) + F(n-2) & \text{if } n \ge 2 \end{cases}
\]
Naive evaluation has \(T(n) = T(n-1) + T(n-2) + O(1)\), yielding exponential time \(O(\phi^n)\) where \(\phi \approx 1.618\) .

**Beginner-Friendly Explanation**
To find a Fibonacci number, you need the two numbers before it. To find those, you need their predecessors, and so on. The naive approach recalculates the same values many times, making it very slow for large \(n\) .

### Purposes (All begin with "To")

- **To** demonstrate branching recursion and exponential blowup
- **To** motivate memoization and dynamic programming
- **To** model natural phenomena (rabbit populations, spiral growth)
- **To** introduce the concept of overlapping subproblems

### Syntax Rules and Structure

**Naive Recursive Syntax**
```python
def fib(n):
    if n < 2:
        return n
    return fib(n - 1) + fib(n - 2)
```

**Memoized Syntax**
```python
def fib_memo(n, memo={}):
    if n in memo:
        return memo[n]
    if n < 2:
        return n
    memo[n] = fib_memo(n - 1, memo) + fib_memo(n - 2, memo)
    return memo[n]
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Base cases | `n < 2` returns `n` |
| Two recursive calls | `fib(n-1)` and `fib(n-2)` |
| Memo cache | Dictionary storing computed values |

### Annotated Code Examples

**Example 1: Naive vs. Memoized Performance**

```python
import time

# Naive: exponential time
def fib_naive(n):
    if n < 2:
        return n
    return fib_naive(n - 1) + fib_naive(n - 2)


# Memoized: linear time
def fib_memo(n, memo=None):
    if memo is None:
        memo = {}
    if n in memo:
        return memo[n]
    if n < 2:
        return n
    memo[n] = fib_memo(n - 1, memo) + fib_memo(n - 2, memo)
    return memo[n]


n = 35

start = time.perf_counter()
result_naive = fib_naive(n)
naive_time = time.perf_counter() - start

start = time.perf_counter()
result_memo = fib_memo(n)
memo_time = time.perf_counter() - start

print(f"fib({n}) = {result_memo}")
print(f"Naive:    {naive_time:.4f}s")
print(f"Memoized: {memo_time:.6f}s")
print(f"Speedup:  {naive_time/memo_time:.0f}x")
```

**Expected Output:**
```
fib(35) = 9227465
Naive:    2.3456s
Memoized: 0.000089s
Speedup:  26355x
```

**Why This Output Occurs**: The naive recursion recomputes `fib(30)` over 100,000 times for `fib(35)`. Memoization stores each result once, reducing time from O(2^n) to O(n). The memo dictionary prevents redundant work .

### Real-World Cases

- **Dynamic Programming Introduction**: Fibonacci is the canonical DP example
- **Algorithm Analysis**: Demonstrates exponential vs. polynomial time
- **Financial Modeling**: Fibonacci retracement in technical analysis

### References

- ETH Zurich - Dynamic Programming Examples (Fibonacci) - https://lec.inf.ethz.ch/baug/informatik2/2020/slides/dp.pdf 
- ETH Zurich - Fibonacci Memoization Analysis - https://lec.inf.ethz.ch/baug/informatik2/2019/slides/infIILecture8.pdf 
- University of Houston - Dynamic Programming and Memoization - https://www2.cs.uh.edu/~panruowu/2022f_cosc3320/lec4_dp.pdf 


## 3. Tree Traversal (In-order, Pre-order, Post-order)

### Definitions

**Core Definition**
Tree traversal is the process of visiting every node in a tree exactly once, where the order of visiting the root relative to its subtrees defines three standard patterns: pre-order (root, left, right), in-order (left, root, right), and post-order (left, right, root) .

**Technical Definition**
For a binary tree node with left child L and right child R:
- **Pre-order**: Visit node, then traverse L, then traverse R
- **In-order**: Traverse L, visit node, then traverse R
- **Post-order**: Traverse L, traverse R, then visit node

**Beginner-Friendly Explanation**
Tree traversal is like reading a book with nested chapters. Pre-order reads the chapter title first, then the sub-chapters. In-order reads all left sub-chapters, then the title, then right sub-chapters. Post-order reads all sub-chapters before the title .

### Purposes (All begin with "To")

- **To** process all nodes in a tree in a systematic order
- **To** enable operations like searching, copying, and evaluating expression trees
- **To** produce sorted output from a binary search tree (in-order)
- **To** support tree destruction (post-order: children before parent)

### Syntax Rules and Structure

**General Syntax (C)**
```c
// Pre-order: Root, Left, Right
void preorder(Node* root) {
    if (root != NULL) {
        printf("%d ", root->data);   // Visit
        preorder(root->left);         // Left
        preorder(root->right);        // Right
    }
}

// In-order: Left, Root, Right
void inorder(Node* root) {
    if (root != NULL) {
        inorder(root->left);          // Left
        printf("%d ", root->data);   // Visit
        inorder(root->right);         // Right
    }
}

// Post-order: Left, Right, Root
void postorder(Node* root) {
    if (root != NULL) {
        postorder(root->left);        // Left
        postorder(root->right);       // Right
        printf("%d ", root->data);   // Visit
    }
}
```

**Component Breakdown**

| Traversal | Order | Primary Use |
|-----------|-------|-------------|
| Pre-order | Root → L → R | Copy tree, prefix expressions |
| In-order | L → Root → R | Sorted output (BST) |
| Post-order | L → R → Root | Delete tree, postfix expressions |

### Annotated Code Examples

**Example 1: Tree Traversal with Sample Tree (Python)**

```python
class TreeNode:
    def __init__(self, value):
        self.value = value
        self.left = None
        self.right = None


def preorder(node):
    """Root, Left, Right"""
    if node is None:
        return []
    return [node.value] + preorder(node.left) + preorder(node.right)


def inorder(node):
    """Left, Root, Right"""
    if node is None:
        return []
    return inorder(node.left) + [node.value] + inorder(node.right)


def postorder(node):
    """Left, Right, Root"""
    if node is None:
        return []
    return postorder(node.left) + postorder(node.right) + [node.value]


# Build sample tree:
#         27
#        /  \
#      14    35
#     / \   / \
#   10  19 31  42
root = TreeNode(27)
root.left = TreeNode(14)
root.right = TreeNode(35)
root.left.left = TreeNode(10)
root.left.right = TreeNode(19)
root.right.left = TreeNode(31)
root.right.right = TreeNode(42)

print(f"Pre-order:  {preorder(root)}")
print(f"In-order:   {inorder(root)}")
print(f"Post-order: {postorder(root)}")

# In-order on BST produces sorted sequence
print(f"\nIn-order on BST is sorted: {inorder(root) == sorted(inorder(root))}")
```

**Expected Output:**
```
Pre-order:  [27, 14, 10, 19, 35, 31, 42]
In-order:   [10, 14, 19, 27, 31, 35, 42]
Post-order: [10, 19, 14, 31, 42, 35, 27]

In-order on BST is sorted: True
```

**Why This Output Occurs**: The tree structure determines the output order. Pre-order visits 27 first, then its left subtree (14, 10, 19), then right subtree (35, 31, 42). In-order on a binary search tree always produces sorted output because smaller values are in left subtrees .

### Real-World Cases

- **Expression Evaluation**: Post-order traversal evaluates arithmetic expression trees
- **File System Display**: Pre-order traversal lists directories before contents
- **Database Indexing**: In-order traversal of B-trees produces sorted key sequences
- **Tree Serialization**: Pre-order with null markers enables reconstruction

### References

- NIELIT Gorakhpur - Binary Tree Traversal Methods - https://www.nielit.in/gorakhpur/sites/default/files/Gorakhpur/ALevel_2_DS_14May%202020_RJS.pdf 
- IARE - Tree Traversal Recursive and Iterative - https://iare.ac.in/sites/default/files/PPT/DS%20PPTS_2.pdf 


## 4. Divide-and-Conquer Problems (Merge Sort, Quick Sort)

### Definitions

**Core Definition**
Divide-and-conquer is a recursive problem-solving paradigm that breaks a problem into two or more smaller subproblems of the same type, solves them recursively, and combines their solutions to solve the original problem .

**Technical Definition**
A divide-and-conquer algorithm follows three steps:
1. **Divide**: Partition the input into smaller subproblems
2. **Conquer**: Recursively solve the subproblems
3. **Combine**: Merge the subproblem solutions into a solution for the original problem

For sorting, Merge Sort divides arbitrarily and combines thoughtfully (merge); Quick Sort divides thoughtfully (partition) and combines trivially .

**Beginner-Friendly Explanation**
Divide-and-conquer is like organizing a large pile of papers. You split the pile in half, give each half to a friend to organize, then merge the two organized piles back together. Recursively, your friends do the same with their halves .

### Purposes (All begin with "To")

- **To** achieve O(n log n) sorting instead of O(n²)
- **To** exploit parallelizable problem structure
- **To** reduce complex problems to simpler instances
- **To** provide optimal algorithms for many classical problems

### Syntax Rules and Structure

**Merge Sort Syntax**
```python
def merge_sort(arr):
    if len(arr) <= 1:          # Base case
        return arr
    mid = len(arr) // 2
    left = merge_sort(arr[:mid])    # Divide & Conquer
    right = merge_sort(arr[mid:])
    return merge(left, right)       # Combine
```

**Quick Sort Syntax**
```python
def quick_sort(arr):
    if len(arr) <= 1:          # Base case
        return arr
    pivot = arr[len(arr) // 2]
    left = [x for x in arr if x < pivot]   # Divide
    mid = [x for x in arr if x == pivot]
    right = [x for x in arr if x > pivot]
    return quick_sort(left) + mid + quick_sort(right)  # Conquer & Combine
```

**Component Breakdown**

| Algorithm | Divide Step | Combine Step | Complexity |
|-----------|-------------|--------------|------------|
| Merge Sort | Split in half | Merge two sorted lists | O(n log n) always |
| Quick Sort | Partition around pivot | Concatenate | O(n log n) average |

### Annotated Code Examples

**Example 1: Merge Sort with Trace (Python)**

```python
def merge_sort(arr, depth=0):
    indent = "  " * depth
    print(f"{indent}merge_sort({arr})")

    if len(arr) <= 1:
        print(f"{indent}  base: {arr}")
        return arr

    mid = len(arr) // 2
    left = merge_sort(arr[:mid], depth + 1)
    right = merge_sort(arr[mid:], depth + 1)

    merged = merge(left, right)
    print(f"{indent}  merge({left}, {right}) -> {merged}")
    return merged


def merge(left, right):
    result = []
    i = j = 0
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            result.append(left[i])
            i += 1
        else:
            result.append(right[j])
            j += 1
    result.extend(left[i:])
    result.extend(right[j:])
    return result


print(f"\nFinal: {merge_sort([38, 27, 43, 3, 9, 82, 10])}")
```

**Expected Output:**
```
merge_sort([38, 27, 43, 3, 9, 82, 10])
  merge_sort([38, 27, 43])
    merge_sort([38])
      base: [38]
    merge_sort([27, 43])
      merge_sort([27])
        base: [27]
      merge_sort([43])
        base: [43]
      merge([27], [43]) -> [27, 43]
    merge([38], [27, 43]) -> [27, 38, 43]
  merge_sort([3, 9, 82, 10])
    ...
  merge([27, 38, 43], [3, 9, 10, 82]) -> [3, 9, 10, 27, 38, 43, 82]

Final: [3, 9, 10, 27, 38, 43, 82]
```

**Why This Output Occurs**: The array is recursively split until single elements remain (base cases), then merged pairwise in sorted order. Each merge takes O(k) for k elements, and there are O(log n) levels, giving O(n log n) total .

### Real-World Cases

- **External Sorting**: Merge sort is used for sorting data too large for memory
- **Database Systems**: Quick sort for in-memory sorting, merge sort for disk-based
- **Parallel Computing**: Both algorithms parallelize naturally
- **Python's Timsort**: Hybrid of merge sort and insertion sort

### References

- UMass Amherst - QuickSort and MergeSort - https://people.cs.umass.edu/~barring/cs187f11/lecture/25.pdf 
- ANU COMP 1100 - Divide and Conquer Algorithms - https://comp.anu.edu.au/pages/2020-S2/courses/comp1100/lectures/Week11/Lecture-11A.pdf 
- Duke COMPSCI 330 - Divide and Conquer Notes - https://courses.cs.duke.edu/spring18/compsci330/Notes/divideandconquer.pdf 


## 5. Recursive Search (Binary Search)

### Definitions

**Core Definition**
Binary search is a recursive divide-and-conquer algorithm that finds a target value in a sorted array by repeatedly halving the search range .

**Technical Definition**
Given sorted array `arr` and target `x`, binary search computes `mid = low + (high - low) / 2`. If `arr[mid] == x`, return `mid`. If `arr[mid] > x`, search the left half recursively. Otherwise, search the right half. The recurrence is \(T(n) = T(n/2) + O(1) = O(\log n)\) .

**Beginner-Friendly Explanation**
Binary search is like looking up a word in a dictionary. You open to the middle, check if your word comes before or after, then repeat with the appropriate half. Each step eliminates half the remaining possibilities .

### Purposes (All begin with "To")

- **To** search sorted data in O(log n) time instead of O(n)
- **To** demonstrate logarithmic recursion depth
- **To** serve as the foundation for binary search trees and other structures
- **To** enable efficient range queries and insertion point finding

### Syntax Rules and Structure

**General Syntax (C++)**
```cpp
int binarySearch(int arr[], int l, int r, int x) {
    if (r >= l) {
        int mid = l + (r - l) / 2;  // Avoid overflow
        if (arr[mid] == x) return mid;
        if (arr[mid] > x) return binarySearch(arr, l, mid - 1, x);
        return binarySearch(arr, mid + 1, r, x);
    }
    return -1;  // Not found
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `l`, `r` | Current search bounds |
| `mid` | `l + (r - l) / 2` (overflow-safe) |
| Base case | `r < l` (not found) or `arr[mid] == x` (found) |
| Reduction | Halve the range |

### Annotated Code Examples

**Example 1: Binary Search with Trace (Python)**

```python
def binary_search(arr, target, low, high, depth=0):
    indent = "  " * depth
    print(f"{indent}search({low}, {high})")

    if low > high:
        print(f"{indent}  not found")
        return -1

    mid = low + (high - low) // 2
    print(f"{indent}  mid={mid}, arr[{mid}]={arr[mid]}")

    if arr[mid] == target:
        print(f"{indent}  FOUND at {mid}")
        return mid
    elif arr[mid] > target:
        return binary_search(arr, target, low, mid - 1, depth + 1)
    else:
        return binary_search(arr, target, mid + 1, high, depth + 1)


arr = [2, 3, 4, 10, 40]
target = 10
print(f"Searching for {target} in {arr}")
result = binary_search(arr, target, 0, len(arr) - 1)
print(f"Result: index {result}")
```

**Expected Output:**
```
Searching for 10 in [2, 3, 4, 10, 40]
search(0, 4)
  mid=2, arr[2]=4
  search(3, 4)
    mid=3, arr[3]=10
    FOUND at 3
Result: index 3
```

**Why This Output Occurs**: First comparison at mid=2 (value 4) is less than 10, so search right half. Second comparison at mid=3 (value 10) matches. Two recursive calls for 5 elements, demonstrating O(log n) behavior .

### Real-World Cases

- **Database Indexing**: B-tree searches use binary search at each node
- **Version Control**: `git bisect` uses binary search to find bug-introducing commits
- **Numerical Methods**: Root-finding via bisection method

### References

- UCLA Statistics - Binary Search Pseudocode - http://www.stat.ucla.edu/~dinov/courses_students.dir/PIC10B_CPP_Summer01.dir/PIC10B_CPP_Summer01.dir/Dinov_PIC10B_UCLA_wk4_files/slide0665.htm 
- GitHub - Recursive Binary Search Implementation - https://raw.githubusercontent.com/maheshacc0/binary-seach/refs/heads/main/newBinarySearch.cpp 
- GitHub - Binary Search with Overflow Prevention - https://raw.githubusercontent.com/MakeContributions/DSA/6620f32d9c79477ddb3e38ee390a81e16c35a79a/algorithms/CPlusPlus/Searching/binary-search.cpp 


## 6. Nested / Deeply Nested Structures

### Definitions

**Core Definition**
Nested structures are data organizations where elements can contain sub-elements of the same type, recursively, to arbitrary depth—such as JSON objects, XML trees, and file system directories .

**Technical Definition**
A nested structure is defined by a recursive grammar: a container holds values, where each value is either a leaf (primitive) or another container. Traversal requires recursion to descend into nested containers.

**Beginner-Friendly Explanation**
Nested structures are like Russian dolls. Each doll might contain another doll, which might contain another. To find the smallest doll, you keep opening until you find one that doesn't open further .

### Purposes (All begin with "To")

- **To** represent hierarchical data naturally (JSON, XML, file systems)
- **To** enable arbitrary-depth traversal and search
- **To** support extraction of paths or values from unknown-depth structures
- **To** model recursive data schemas

### Syntax Rules and Structure

**General Recursive Traversal Pattern (Python)**
```python
def traverse(node, path=""):
    if is_leaf(node):
        yield path, node.value
    else:
        for key, child in node.children():
            yield from traverse(child, path + "/" + key)
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `is_leaf(node)` | Check if node has no children |
| `node.children()` | Iterate over sub-elements |
| `path` | Accumulated path from root |
| `yield from` | Recursive generator delegation |

### Annotated Code Examples

**Example 1: JSON Path Extraction (Python)**

```python
def extract_paths(data, current_path=""):
    """Recursively extract all leaf paths from nested JSON."""
    if isinstance(data, dict):
        for key, value in data.items():
            new_path = f"{current_path}/{key}" if current_path else key
            yield from extract_paths(value, new_path)
    elif isinstance(data, list):
        for index, item in enumerate(data):
            new_path = f"{current_path}[{index}]"
            yield from extract_paths(item, new_path)
    else:
        # Leaf value
        yield current_path, data


# Nested JSON-like structure
data = {
    "users": [
        {
            "name": "Alice",
            "address": {"city": "NYC", "zip": "10001"}
        },
        {
            "name": "Bob",
            "address": {"city": "LA", "zip": "90001"}
        }
    ],
    "count": 2
}

print("All leaf paths:")
for path, value in extract_paths(data):
    print(f"  {path} = {value}")
```

**Expected Output:**
```
All leaf paths:
  users[0]/name = Alice
  users[0]/address/city = NYC
  users[0]/address/zip = 10001
  users[1]/name = Bob
  users[1]/address/city = LA
  users[1]/address/zip = 90001
  count = 2
```

**Why This Output Occurs**: The function recurses into dictionaries and lists, accumulating path components. At each leaf (non-container value), it yields the complete path and value. This handles arbitrary nesting depth .

### Real-World Cases

- **File System Crawling**: Recursively listing all files in nested directories
- **JSON API Processing**: Extracting fields from deeply nested API responses
- **HTML DOM Traversal**: Navigating nested elements in web pages
- **Configuration Parsing**: Reading nested YAML or TOML configs

### References

- Stack Overflow - Recursive JSON Path Extraction - https://stackoverflow.com/revisions/507fb8ee-5c6c-45a6-8e99-052736ded0cb/view-source 
- Stack Overflow - Nested JSON Traversal with Generators - https://stackoverflow.com/revisions/fccbe3d0-9db2-40cb-b2d2-7c17c4f74ce2/view-source 


## Consolidated References

- Vultr Docs - C++ Program to Calculate Factorial Using Recursion - https://docs.vultr.com/cpp/examples/calculate-factorial-of-a-number-using-recursion 
- OGM Materyal - Recursive Factorial Explanation - http://ogmmateryal.eba.gov.tr/panel/upload/etkilesimli/kitap/bilgisayarbilimikur1/sec/unite1/files/basic-html/page194.html 
- ETH Zurich - Dynamic Programming Examples (Fibonacci) - https://lec.inf.ethz.ch/baug/informatik2/2020/slides/dp.pdf 
- ETH Zurich - Fibonacci Memoization Analysis - https://lec.inf.ethz.ch/baug/informatik2/2019/slides/infIILecture8.pdf 
- University of Houston - Dynamic Programming and Memoization - https://www2.cs.uh.edu/~panruowu/2022f_cosc3320/lec4_dp.pdf 
- NIELIT Gorakhpur - Binary Tree Traversal Methods - https://www.nielit.in/gorakhpur/sites/default/files/Gorakhpur/ALevel_2_DS_14May%202020_RJS.pdf 
- IARE - Tree Traversal Recursive and Iterative - https://iare.ac.in/sites/default/files/PPT/DS%20PPTS_2.pdf 
- UMass Amherst - QuickSort and MergeSort - https://people.cs.umass.edu/~barring/cs187f11/lecture/25.pdf 
- ANU COMP 1100 - Divide and Conquer Algorithms - https://comp.anu.edu.au/pages/2020-S2/courses/comp1100/lectures/Week11/Lecture-11A.pdf 
- Duke COMPSCI 330 - Divide and Conquer Notes - https://courses.cs.duke.edu/spring18/compsci330/Notes/divideandconquer.pdf 
- UCLA Statistics - Binary Search Pseudocode - http://www.stat.ucla.edu/~dinov/courses_students.dir/PIC10B_CPP_Summer01.dir/PIC10B_CPP_Summer01.dir/Dinov_PIC10B_UCLA_wk4_files/slide0665.htm 
- GitHub - Recursive Binary Search Implementation - https://raw.githubusercontent.com/maheshacc0/binary-seach/refs/heads/main/newBinarySearch.cpp 
- GitHub - Binary Search with Overflow Prevention - https://raw.githubusercontent.com/MakeContributions/DSA/6620f32d9c79477ddb3e38ee390a81e16c35a79a/algorithms/CPlusPlus/Searching/binary-search.cpp 
- Stack Overflow - Recursive JSON Path Extraction - https://stackoverflow.com/revisions/507fb8ee-5c6c-45a6-8e99-052736ded0cb/view-source 
- Stack Overflow - Nested JSON Traversal with Generators - https://stackoverflow.com/revisions/fccbe3d0-9db2-40cb-b2d2-7c17c4f74ce2/view-source 