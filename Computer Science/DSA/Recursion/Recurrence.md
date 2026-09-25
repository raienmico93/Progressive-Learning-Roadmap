# Recurrence Relations: Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**
A recurrence relation is a mathematical equation that defines a function \(T(n)\) in terms of its own values at smaller inputs, commonly used to describe the running time of recursive algorithms.

**Technical Definition**
A recurrence relation for a divide-and-conquer algorithm expresses \(T(n)\) as a sum of costs from recursive subproblems plus the cost of dividing and combining: \(T(n) = \sum a_i T(b_i n) + f(n)\), where \(a_i\) is the number of subproblems, \(b_i\) is the size reduction factor, and \(f(n)\) is the non-recursive work .

**Beginner-Friendly Explanation**
A recurrence is like a recipe that says "to find the answer for size n, first find the answers for smaller sizes, then combine them." Solving the recurrence means finding a simple formula (like \(O(n \log n)\)) that tells you how the work grows as n increases.

### Key Characteristics

- **Self-Reference**: The function appears on both sides of the equation 
- **Base Cases**: At least one trivial case must be defined to terminate the recursion 
- **Asymptotic Focus**: In algorithm analysis, only the growth rate matters, not exact constants 
- **Multiple Solution Methods**: Recursion trees, substitution, Master Theorem, and Akra-Bazzi each suit different recurrence forms

### Prerequisites

- Big-O notation and asymptotic analysis
- Basic logarithms and exponents
- Familiarity with recursive algorithms (merge sort, binary search)
- Mathematical induction (for the substitution method)

### Related Programming Areas

- **Divide-and-Conquer Algorithms**: Merge sort, quick sort, Strassen's matrix multiplication
- **Dynamic Programming**: Overlapping subproblems and memoization
- **Algorithm Analysis**: Predicting performance before implementation
- **Computational Complexity Theory**: Reductions and lower bounds

### Core Concepts

| Method | Applies To | Key Output |
|--------|-----------|------------|
| Formulating Recurrences | Any recursive algorithm | Equation describing work |
| Solving Recurrences | Any recurrence | Closed-form or asymptotic bound |
| Recursion Trees | Any recurrence | Visual sum and guess |
| Substitution | Any recurrence | Proven bound via induction |
| Master Theorem | \(T(n) = aT(n/b) + f(n)\) | Direct asymptotic result |
| Akra-Bazzi | Unequal subproblem sizes | General asymptotic result |


## 1. Formulating Recurrences

### Definitions

**Core Definition**
Formulating a recurrence means translating a recursive algorithm into a mathematical equation that describes its running time as a function of input size.

**Technical Definition**
For a divide-and-conquer algorithm, the recurrence takes the form \(T(n) = a_1 T(n/b_1) + a_2 T(n/b_2) + \cdots + f(n)\), where each term represents a recursive call and \(f(n)\) represents the non-recursive work (dividing, combining, overhead) .

**Beginner-Friendly Explanation**
To formulate a recurrence, ask: "How many times does this function call itself, and on what size inputs?" and "How much work happens outside those calls?" The answers become the terms of the recurrence.

### Purposes (All begin with "To")

- **To** translate recursive code into a mathematical object that can be analyzed
- **To** capture the relationship between problem size and work performed
- **To** provide the input for solution methods like the Master Theorem
- **To** identify which part of an algorithm dominates its running time

### Syntax Rules and Structure

**General Formulation Process**

```
1. Identify the base case: T(1) = c (constant work)
2. Identify recursive calls: how many? (a) and what size? (n/b)
3. Identify non-recursive work: f(n)
4. Write: T(n) = a·T(n/b) + f(n)
```

**Component Breakdown**

| Component | Description | Example (Merge Sort) |
|-----------|-------------|---------------------|
| Base case | Work for smallest input | T(1) = O(1) |
| `a` | Number of recursive calls | 2 |
| `b` | Size reduction factor | 2 |
| `f(n)` | Non-recursive work | O(n) (merge) |

**Syntax Rules**

- Every recursive call must reduce the problem size (b > 1 or n - k)
- Base cases must be explicitly defined
- Floors and ceilings are typically ignored in asymptotic analysis 

### Annotated Code Examples

**Example 1: Binary Search**

```python
def binary_search(arr, target, low, high):
    if low > high:                     # Base case: O(1)
        return -1
    mid = (low + high) // 2
    if arr[mid] == target:
        return mid
    elif arr[mid] > target:
        return binary_search(arr, target, low, mid - 1)  # 1 call, size n/2
    else:
        return binary_search(arr, target, mid + 1, high) # 1 call, size n/2

# Recurrence: T(n) = T(n/2) + O(1)
```

**Expected Output** (analysis results):
```
Binary Search: T(n) = T(n/2) + O(1)     → O(log n)
```

**Why This Output Occurs**: Binary search halves the problem with constant overhead.

**Example 2: Merge Sort**

```python
def merge_sort(arr):
    if len(arr) <= 1:                  # Base case: O(1)
        return arr
    mid = len(arr) // 2
    left = merge_sort(arr[:mid])       # 1 call, size n/2
    right = merge_sort(arr[mid:])      # 1 call, size n/2
    return merge(left, right)          # Non-recursive work: O(n)

# Recurrence: T(n) = 2T(n/2) + O(n)
```

**Expected Output** (analysis results):
```
Merge Sort:    T(n) = 2T(n/2) + O(n)     → O(n log n)
```

**Why This Output Occurs**: Merge sort splits into two equal halves with linear merge cost.

**Example 3: Naive Fibonacci**

```python
def fib(n):
    if n <= 1:                         # Base case: O(1)
        return n
    return fib(n - 1) + fib(n - 2)     # 2 calls, sizes n-1 and n-2

# Recurrence: T(n) = T(n-1) + T(n-2) + O(1)
```

**Expected Output** (analysis results):
```
Fibonacci:     T(n) = T(n-1) + T(n-2) + c → O(2^n)
```

**Why This Output Occurs**: Fibonacci branches into two calls with constant overhead, causing exponential growth.

### Real-World Cases

- **Analyzing Library Functions**: Formulating recurrences for standard algorithms like `std::sort`
- **Optimizing Custom Algorithms**: Identifying whether recursive overhead dominates
- **Interview Preparation**: Deriving recurrences from code is a common whiteboard exercise

### References

- Stack Overflow - How to Write a Recurrence Relation - https://stackoverflow.com/questions/30201391/how-to-write-a-recurrence-relation-for-a-given-piece-of-code 
- UNM CS 362 - Divide and Conquer Recurrences - https://www.cs.unm.edu/~saia/classes/362-s24/lec/lec-divideAndConquer.pdf 


## 2. Solving Recurrences

### Definitions

**Core Definition**
Solving a recurrence means finding a closed-form expression (or asymptotic bound) for \(T(n)\) that does not involve \(T\) on the right-hand side.

**Technical Definition**
A solution to a recurrence is a function \(g(n)\) such that \(T(n) = \Theta(g(n))\) or \(T(n) = O(g(n))\), proven correct by induction or derived through summation .

**Beginner-Friendly Explanation**
Solving a recurrence is like finding the "final answer" to the recipe: instead of saying "the time for n is the time for n/2 plus n," you say "the time for n is proportional to n log n."

### Purposes (All begin with "To")

- **To** obtain a simple formula for algorithm running time
- **To** compare algorithms independent of implementation details
- **To** predict performance for large inputs without running code
- **To** identify the dominant cost term

### Syntax Rules and Structure

**General Solution Approaches**

| Method | Approach | Best For |
|--------|----------|----------|
| Iteration/Telescoping | Expand and find pattern | Simple linear recurrences |
| Recursion Tree | Visualize cost per level | Divide-and-conquer |
| Substitution | Guess and prove by induction | Any recurrence |
| Master Theorem | Match to standard form | \(T(n) = aT(n/b) + f(n)\) |
| Akra-Bazzi | Solve integral equation | Unequal subproblem sizes |

### Annotated Code Examples

**Example 1: Solving by Iteration (Python)**

```python
# Recurrence: T(n) = T(n-1) + c, T(1) = d
# Solve by expansion:
# T(n) = T(n-1) + c
#      = [T(n-2) + c] + c = T(n-2) + 2c
#      = T(n-3) + 3c
#      ...
#      = T(1) + (n-1)c
#      = d + (n-1)c
# Solution: T(n) = O(n)

# Recurrence: T(n) = T(n/2) + c
# T(n) = T(n/2) + c
#      = T(n/4) + 2c
#      = T(n/8) + 3c
#      ...
#      = T(1) + log(n) * c
# Solution: T(n) = O(log n)
```

**Expected Output:**
```
T(n) = T(n-1) + c  → T(n) = O(n)
T(n) = T(n/2) + c  → T(n) = O(log n)
```

**Why This Output Occurs**: Iteration reveals the pattern of accumulated work. The number of steps until reaching the base case determines the complexity: n-1 steps for linear reduction, log n steps for halving .

### Real-World Cases

- **Algorithm Selection**: Choosing between O(n log n) and O(n²) sorting
- **System Design**: Estimating query time for database indexes
- **Performance Modeling**: Predicting scaling behavior

### References

- Horowitz & Sahani - Fundamentals of Computer Algorithms (Substitution Method) - https://archive.org/download/compatitive-programming/Books/horowitz-and-sahani-fundamentals-of-computer-al_text.pdf 
- UNM CS 362 - Solving Recurrences - https://www.cs.unm.edu/~saia/classes/362-s24/lec/lec-divideAndConquer.pdf 


## 3. Recursion Trees

### Definitions

**Core Definition**
A recursion tree is a visual tool that expands a recurrence level by level, showing the cost at each level and summing these costs to find the total.

**Technical Definition**
For \(T(n) = aT(n/b) + f(n)\), the recursion tree has a root with cost \(f(n)\), \(a\) children each with cost \(f(n/b)\), \(a^2\) grandchildren with cost \(f(n/b^2)\), and so on until subproblems reach size 1. The total cost is the sum of all node costs .

**Beginner-Friendly Explanation**
A recursion tree is like a family tree of function calls. Each level shows how the work splits. By adding up the work at every level, you get the total work for the whole algorithm.

### Purposes (All begin with "To")

- **To** visualize how work is distributed across recursion levels
- **To** generate a good guess for the solution
- **To** identify which level dominates the total cost
- **To** provide intuition before formal proof

### Syntax Rules and Structure

**Recursion Tree Construction**

```
Level 0: f(n)                          [1 node, cost f(n)]
Level 1: f(n/b) + f(n/b) + ...         [a nodes, total a·f(n/b)]
Level 2: f(n/b²) + ...                 [a² nodes, total a²·f(n/b²)]
...
Level i: a^i nodes, each cost f(n/b^i)  [total a^i·f(n/b^i)]
...
Level log_b n: 1 node per subproblem   [a^(log_b n) nodes, each O(1)]
```

**Component Breakdown**

| Level | Nodes | Cost per Node | Total Level Cost |
|-------|-------|--------------|-----------------|
| 0 | 1 | f(n) | f(n) |
| 1 | a | f(n/b) | a·f(n/b) |
| i | a^i | f(n/b^i) | a^i·f(n/b^i) |
| log_b n | a^(log_b n) | O(1) | n^(log_b a) |

### Annotated Code Examples

**Example 1: Recursion Tree for T(n) = 3T(n/4) + cn²**

```python
# Recurrence: T(n) = 3T(n/4) + cn²
# Recursion tree analysis:

# Level 0: cn²
# Level 1: 3 · c(n/4)² = (3/16)cn²
# Level 2: 9 · c(n/16)² = (3/16)²cn²
# ...
# Level i: (3/16)^i · cn²
# ...
# Level log_4 n: 3^(log_4 n) · T(1) = n^(log_4 3) · O(1)

# Total cost:
# T(n) = cn² + (3/16)cn² + (3/16)²cn² + ... + n^(log_4 3)
#      = cn² · Σ(3/16)^i + Θ(n^(log_4 3))
#      = cn² · (1/(1 - 3/16)) + Θ(n^(log_4 3))  [geometric series]
#      = (16/13)cn² + Θ(n^(log_4 3))
#      = O(n²)

print("Recursion tree for T(n) = 3T(n/4) + cn²")
print("Level 0: cn²")
print("Level 1: (3/16)cn²")
print("Level 2: (3/16)²cn²")
print("...")
print("Level i: (3/16)^i cn²")
print("\nGeometric series converges → total = O(n²)")
```

**Expected Output:**
```
Recursion tree for T(n) = 3T(n/4) + cn²
Level 0: cn²
Level 1: (3/16)cn²
Level 2: (3/16)²cn²
...
Level i: (3/16)^i cn²

Geometric series converges → total = O(n²)
```

**Why This Output Occurs**: The per-level cost decreases geometrically because \(3/16 < 1\). The root cost \(cn^2\) dominates, and the bottom level \(n^{\log_4 3} \approx n^{0.793}\) is asymptotically smaller. Summing the geometric series gives \(O(n^2)\) .

### Real-World Cases

- **Teaching Recursion**: Visualizing merge sort's O(n log n) behavior
- **Quick Estimation**: Getting a solution guess before formal proof
- **Comparing Algorithms**: Seeing which recursion structure is more efficient

### References

- CLRS - The Recursion-Tree Method for Solving Recurrences - http://archive.org/stream/0262033844Algorithm/0262033844_Algorithm_djvu.txt 
- UNM CS 561 - Recursion Trees - https://www.cs.unm.edu/~saia/classes/561-fall2006/lec/lec3-2x2.pdf 


## 4. Substitution Method

### Definitions

**Core Definition**
The substitution method solves recurrences by guessing the form of the solution and proving it correct using mathematical induction.

**Technical Definition**
Given a recurrence \(T(n)\), guess \(T(n) = O(g(n))\), then prove by induction that \(T(n) \leq c \cdot g(n)\) for all \(n \geq n_0\), using the inductive hypothesis \(T(k) \leq c \cdot g(k)\) for \(k < n\) .

**Beginner-Friendly Explanation**
The substitution method is "guess and check." You guess what the answer might be (like \(O(n \log n)\)), then prove it by showing that if it works for smaller sizes, it works for the next size up.

### Purposes (All begin with "To")

- **To** prove a solution obtained by intuition or recursion trees
- **To** handle recurrences where the Master Theorem does not apply
- **To** establish tight bounds with explicit constants
- **To** develop mathematical rigor in algorithm analysis

### Syntax Rules and Structure

**Substitution Method Steps**

```
1. GUESS: Propose T(n) = O(g(n))
2. ASSUME: For all k < n, T(k) ≤ c·g(k)  [inductive hypothesis]
3. VERIFY: Show T(n) ≤ c·g(n) using the recurrence
4. BASE CASE: Verify for n = n_0
5. CONCLUSION: T(n) = O(g(n)) by induction
```

**Component Breakdown**

| Step | Description |
|------|-------------|
| Guess | Solution form from intuition/tree |
| Inductive hypothesis | Assume bound for smaller inputs |
| Verification | Substitute into recurrence |
| Base case | Confirm small n |
| Conclusion | Apply induction principle |

### Annotated Code Examples

**Example 1: Substitution for T(n) = T(n/2) + 1**

```python
# Recurrence: T(n) = T(n/2) + 1, T(1) = 1
# Guess: T(n) = O(log n)

# Proof:
# Assume T(k) ≤ c·log(k) for all k < n
# T(n) = T(n/2) + 1
#      ≤ c·log(n/2) + 1
#      = c·(log n - log 2) + 1
#      = c·log n - c + 1
#      ≤ c·log n    [for c ≥ 1]
#
# Base case: T(2) = T(1) + 1 = 2 ≤ c·log(2) = c → c ≥ 2 works
# Conclusion: T(n) = O(log n)

print("Substitution proof for T(n) = T(n/2) + 1")
print("Guess: T(n) = O(log n)")
print("Inductive step: T(n) ≤ c·log(n/2) + 1 = c·log n - c + 1 ≤ c·log n")
print("Result: T(n) = O(log n)")
```

**Expected Output:**
```
Substitution proof for T(n) = T(n/2) + 1
Guess: T(n) = O(log n)
Inductive step: T(n) ≤ c·log(n/2) + 1 = c·log n - c + 1 ≤ c·log n
Result: T(n) = O(log n)
```

**Why This Output Occurs**: The inductive hypothesis substitutes the bound for the smaller input. The algebraic manipulation shows the bound holds for \(n\) as long as \(c \geq 1\). The base case confirms the constant exists .

### Real-World Cases

- **Proving Algorithm Bounds**: Formal verification of complexity claims
- **Handling Unusual Recurrences**: When Master Theorem's conditions fail
- **Research Papers**: Providing rigorous proofs of new algorithms

### References

- UNM CS 362 - Substitution Method - https://www.cs.unm.edu/~saia/classes/362-s24/lec/lec-divideAndConquer.pdf 
- IIT Bombay CS 218 - Substitution Method Proofs - https://www.cse.iitb.ac.in/~pandya58/cs218m/Lect6ann.pdf 


## 5. Master Theorem

### Definitions

**Core Definition**
The Master Theorem provides a direct formula for solving recurrences of the form \(T(n) = aT(n/b) + f(n)\), where \(a \geq 1\), \(b > 1\), and \(f(n)\) is asymptotically positive.

**Technical Definition**
For \(T(n) = aT(n/b) + f(n)\), compare \(f(n)\) with \(n^{\log_b a}\):
- **Case 1**: If \(f(n) = O(n^{\log_b a - \epsilon})\) for some \(\epsilon > 0\), then \(T(n) = \Theta(n^{\log_b a})\)
- **Case 2**: If \(f(n) = \Theta(n^{\log_b a})\), then \(T(n) = \Theta(n^{\log_b a} \log n)\)
- **Case 3**: If \(f(n) = \Omega(n^{\log_b a + \epsilon})\) and \(af(n/b) \leq cf(n)\) for some \(c < 1\), then \(T(n) = \Theta(f(n))\) 

**Beginner-Friendly Explanation**
The Master Theorem compares two quantities: the "work of recursion" (how much the problem splits) and the "work at this level" (how much the combine step costs). Whichever is bigger determines the answer; if they're equal, you multiply by log n.

### Purposes (All begin with "To")

- **To** solve common divide-and-conquer recurrences in O(1) time
- **To** avoid lengthy substitution proofs for standard cases
- **To** quickly compare algorithm variants (e.g., different matrix multiplication strategies)
- **To** recognize when a recurrence falls into a "gap" and requires other methods

### Syntax Rules and Structure

**Master Theorem Decision Process**

```
Given T(n) = aT(n/b) + f(n):
1. Compute n^(log_b a)
2. Compare f(n) with n^(log_b a):
   - f(n) smaller by polynomial factor → Case 1: Θ(n^(log_b a))
   - f(n) equal → Case 2: Θ(n^(log_b a) log n)
   - f(n) larger by polynomial factor (with regularity) → Case 3: Θ(f(n))
```

**Component Breakdown**

| Case | Condition | Result | Example |
|------|-----------|--------|---------|
| 1 | \(f(n) = O(n^{\log_b a - \epsilon})\) | \(\Theta(n^{\log_b a})\) | T(n)=8T(n/2)+n² → n³ |
| 2 | \(f(n) = \Theta(n^{\log_b a})\) | \(\Theta(n^{\log_b a} \log n)\) | T(n)=2T(n/2)+n → n log n |
| 3 | \(f(n) = \Omega(n^{\log_b a + \epsilon})\) + regularity | \(\Theta(f(n))\) | T(n)=2T(n/2)+n² → n² |

**Syntax Rules**

- The recurrence must be of the form \(T(n) = aT(n/b) + f(n)\) (single subproblem size)
- Subproblems must be of roughly equal size
- \(f(n)\) must be polynomially comparable to \(n^{\log_b a}\) (no gaps like \(n \log n\) vs. \(n\))
- Regularity condition: \(af(n/b) \leq cf(n)\) for some \(c < 1\) and large \(n\)

### Annotated Code Examples

**Example 1: Applying Master Theorem (Python)**

```python
import math

def master_theorem(a, b, f_type, n_power=None):
    """
    Classify and solve T(n) = aT(n/b) + f(n).
    f_type: 'n^p', 'n^p log n', or 'constant'
    """
    log_b_a = math.log(a, b)
    critical = n_power if n_power else 0

    if f_type == 'n^p':
        if critical < log_b_a:
            return f"Case 1: Θ(n^{log_b_a:.2f})"
        elif critical == log_b_a:
            return f"Case 2: Θ(n^{log_b_a:.2f} log n)"
        else:
            return f"Case 3: Θ(n^{critical})"
    return "Requires manual analysis"


# Examples
print("Merge Sort: T(n) = 2T(n/2) + n")
print(f"  a=2, b=2, n^(log_2 2)=n, f(n)=n")
print(f"  {master_theorem(2, 2, 'n^p', 1)}")

print("\nBinary Search: T(n) = T(n/2) + 1")
print(f"  a=1, b=2, n^(log_2 1)=n^0=1, f(n)=1")
print(f"  {master_theorem(1, 2, 'n^p', 0)}")

print("\nStrassen: T(n) = 7T(n/2) + n²")
print(f"  a=7, b=2, n^(log_2 7)≈n^2.81, f(n)=n²")
print(f"  {master_theorem(7, 2, 'n^p', 2)}")
```

**Expected Output:**
```
Merge Sort: T(n) = 2T(n/2) + n
  a=2, b=2, n^(log_2 2)=n, f(n)=n
  Case 2: Θ(n^1.00 log n)

Binary Search: T(n) = T(n/2) + 1
  a=1, b=2, n^(log_2 1)=n^0=1, f(n)=1
  Case 2: Θ(n^0.00 log n)  [should be O(log n)]

Strassen: T(n) = 7T(n/2) + n²
  a=7, b=2, n^(log_2 7)≈n^2.81, f(n)=n²
  Case 1: Θ(n^2.81)
```

**Why This Output Occurs**: The Master Theorem compares the exponent of \(f(n)\) with \(\log_b a\). For merge sort, both are 1, so Case 2 gives \(n \log n\). For binary search, both are 0, giving \(\log n\) (the tool's output of "n^0 log n" simplifies to \(\log n\)). For Strassen, \(\log_2 7 \approx 2.81 > 2\), so Case 1 gives \(n^{2.81}\) .

### Real-World Cases

- **Matrix Multiplication**: Choosing between naive O(n³) and Strassen O(n^2.81)
- **Sorting Algorithms**: Confirming merge sort's O(n log n) bound
- **Fast Fourier Transform**: Analyzing divide-and-conquer FFT recurrences

### References

- CLRS - Using the Master Method - http://archive.org/stream/0262033844Algorithm/0262033844_Algorithm_djvu.txt 
- Princeton - Master Theorem Cases - https://www.cs.princeton.edu/%7Ewayne/kleinberg-tardos/pdf/05DivideAndConquerII-2x2.pdf 
- Rhodes College - Master Theorem Statement - https://www.cs.rhodes.edu/~welshc/CS355/F19/Lecture5.pdf 


## 6. Akra-Bazzi Method

### Definitions

**Core Definition**
The Akra-Bazzi method generalizes the Master Theorem to recurrences with unequal subproblem sizes, such as \(T(n) = T(n/3) + T(2n/3) + n\).

**Technical Definition**
For \(T(x) = g(x) + \sum_{i=1}^k a_i T(b_i x + h_i(x))\), find the unique real \(p\) satisfying \(\sum a_i b_i^p = 1\). Then:
\[
T(x) = \Theta\left(x^p \left(1 + \int_1^x \frac{g(u)}{u^{p+1}} du\right)\right)
\]
The \(h_i(x)\) terms represent small perturbations (floors, ceilings) and do not affect the asymptotic result .

**Beginner-Friendly Explanation**
The Akra-Bazzi method is the Master Theorem's more flexible cousin. It handles cases where subproblems are different sizes (like 1/3 and 2/3), which the Master Theorem cannot. You solve an equation to find an exponent \(p\), then compute an integral to get the final answer.

### Purposes (All begin with "To")

- **To** solve recurrences with unequal subproblem sizes
- **To** handle floor/ceiling perturbations rigorously
- **To** analyze algorithms that split unevenly (quickselect, some randomized algorithms)
- **To** generalize the Master Theorem to a broader class

### Syntax Rules and Structure

**Akra-Bazzi Steps**

```
Given T(x) = g(x) + Σ a_i T(b_i x + h_i(x)):

1. Verify conditions:
   - a_i > 0, 0 < b_i < 1
   - |g'(x)| = O(x^c) for some constant c
   - |h_i(x)| = O(x / log² x)

2. Solve for p: Σ a_i b_i^p = 1

3. Compute: T(x) = Θ(x^p (1 + ∫₁ˣ g(u)/u^(p+1) du))
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `a_i` | Number of subproblems of type i |
| `b_i` | Size fraction of subproblem i |
| `g(x)` | Non-recursive work function |
| `p` | Unique real exponent satisfying the equation |
| Integral | Captures the cumulative effect of g(x) |

**Syntax Rules**

- The equation \(\sum a_i b_i^p = 1\) has a unique real solution \(p\) 
- Floors and ceilings are absorbed into \(h_i(x)\) and do not affect the result 
- The integral can often be simplified by recognizing common forms

### Annotated Code Examples

**Example 1: Akra-Bazzi for T(n) = T(n/3) + T(2n/3) + n**

```python
# Recurrence: T(n) = T(n/3) + T(2n/3) + n
# a_1 = 1, b_1 = 1/3
# a_2 = 1, b_2 = 2/3
# g(n) = n

# Step 1: Solve for p: (1/3)^p + (2/3)^p = 1
# Try p = 1: 1/3 + 2/3 = 1 ✓
# So p = 1

# Step 2: Compute integral:
# ∫₁ⁿ u / u^(1+1) du = ∫₁ⁿ u/u² du = ∫₁ⁿ 1/u du = ln(n)

# Step 3: T(n) = Θ(n^1 (1 + ln n)) = Θ(n log n)

print("Akra-Bazzi for T(n) = T(n/3) + T(2n/3) + n")
print("Solve p: (1/3)^p + (2/3)^p = 1 → p = 1")
print("Integral: ∫ u/u² du = ln(n)")
print("Result: T(n) = Θ(n log n)")

# Example 2: T(n) = 2T(n/4) + n
# a = 2, b = 1/4, g(n) = n
# Solve: 2·(1/4)^p = 1 → (1/4)^p = 1/2 → 4^p = 2 → p = 1/2
# Integral: ∫ u/u^(3/2) du = ∫ u^(-1/2) du = 2√n
# T(n) = Θ(n^(1/2) (1 + √n)) = Θ(n)

print("\nAkra-Bazzi for T(n) = 2T(n/4) + n")
print("Solve p: 2·(1/4)^p = 1 → p = 1/2")
print("Integral: ∫ u/u^(3/2) du = 2√n")
print("Result: T(n) = Θ(n)")
```

**Expected Output:**
```
Akra-Bazzi for T(n) = T(n/3) + T(2n/3) + n
Solve p: (1/3)^p + (2/3)^p = 1 → p = 1
Integral: ∫ u/u² du = ln(n)
Result: T(n) = Θ(n log n)

Akra-Bazzi for T(n) = 2T(n/4) + n
Solve p: 2·(1/4)^p = 1 → p = 1/2
Integral: ∫ u/u^(3/2) du = 2√n
Result: T(n) = Θ(n)
```

**Why This Output Occurs**: For the first recurrence, \(p=1\) because \(1/3 + 2/3 = 1\). The integral evaluates to \(\ln n\), giving \(n \log n\). For the second, \(p=1/2\) because \(2 \cdot (1/4)^{1/2} = 2 \cdot 1/2 = 1\). The integral grows as \(\sqrt{n}\), which dominates the \(n^{1/2}\) factor, yielding \(O(n)\) .

### Real-World Cases

- **Quickselect**: Average-case analysis of median-finding with unequal partitions
- **Randomized Algorithms**: Recurrences from random pivot selection
- **Uneven Divide-and-Conquer**: Algorithms that split work asymmetrically

### References

- Wikipedia - Akra-Bazzi Method - https://en.m.wikipedia.org/wiki/Akra%E2%80%93Bazzi_theorem 
- MIT 6.042 - Akra-Bazzi Theorem Notes - http://courses.csail.mit.edu/6.042/fall08/rec13-sol.pdf 
- Stack Overflow - Master Theorem vs Akra-Bazzi - https://stackoverflow.com/questions/38058541/master-theorem-for-subproblems-of-different-sizes 


## Consolidated References

- Internet Archive - CLRS Algorithm Text (Master Theorem, Recursion Trees) - http://archive.org/stream/0262033844Algorithm/0262033844_Algorithm_djvu.txt 
- Wikipedia - Akra-Bazzi Method - https://en.m.wikipedia.org/wiki/Akra%E2%80%93Bazzi_theorem 
- Horowitz & Sahani - Fundamentals of Computer Algorithms (Substitution Method) - https://archive.org/download/compatitive-programming/Books/horowitz-and-sahani-fundamentals-of-computer-al_text.pdf 
- Stack Overflow - How to Write a Recurrence Relation - https://stackoverflow.com/questions/30201391/how-to-write-a-recurrence-relation-for-a-given-piece-of-code 
- MIT 6.042 - Akra-Bazzi Theorem Notes - http://courses.csail.mit.edu/6.042/fall08/rec13-sol.pdf 
- TU Darmstadt - Merge Sort Recurrence Solution - https://wwwevs.mathematik.tu-darmstadt.de/index.php?evsid=32&evsver=994&evsdir=1026&evsfile=Solution2.pdf 
- Wolfram MathWorld - Akra-Bazzi Method - https://mathworld.wolfram.com/Akra-BazziMethod.html 
- IIT Bombay CS 218 - Substitution Method - https://www.cse.iitb.ac.in/~pandya58/cs218m/Lect6ann.pdf 
- UNM CS 561 - Recursion Trees - https://www.cs.unm.edu/~saia/classes/561-fall2006/lec/lec3-2x2.pdf 
- Princeton - Master Theorem - https://www.cs.princeton.edu/%7Ewayne/kleinberg-tardos/pdf/05DivideAndConquerII-2x2.pdf 
- Western University - Master Theorem for Recurrences - https://www.csd.uwo.ca/~mmorenom/CS433-CS9624/Resources/NewMasterTheorem.pdf 
- WPI CS 2223 - Master Theorem Reference - http://web.cs.wpi.edu/~cs2223/b13/HW/HW3/Solutions/solutions_hw3_cs2223_b13.pdf 
- Boston University - Akra-Bazzi Problems - https://cs-people.bu.edu/aene/cs237sp17/handouts/mcs.pdf 
- UNM CS 362 - Divide and Conquer, Substitution, Recursion Trees - https://www.cs.unm.edu/~saia/classes/362-s24/lec/lec-divideAndConquer.pdf 
- Rhodes College - Solving Recurrences - https://www.cs.rhodes.edu/~welshc/CS355/F19/Lecture5.pdf 
- SFU CMPT 307 - Master Theorem Study Guide - https://www2.cs.sfu.ca/~binay/2019/cmpt307/D-C-StudyGuide.pdf 
- Stack Overflow - Master Theorem vs Akra-Bazzi - https://stackoverflow.com/questions/38058541/master-theorem-for-subproblems-of-different-sizes 