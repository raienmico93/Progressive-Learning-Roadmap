# Binary Search on Answer: Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**
Binary Search on Answer is an algorithmic paradigm that applies binary search not to a given data array, but to the **range of possible answers**. It finds the optimal value (minimum or maximum) satisfying a given condition by repeatedly testing candidate answers with a feasibility function.

**Technical Definition**
Given a monotonic predicate \( P(x) \rightarrow \{0, 1\} \), Binary Search on Answer finds the boundary of the feasible region in the search space \([low, high]\), where the answer is the smallest \( x \) such that \( P(x) = 1 \) (for minimization) or the largest \( x \) such that \( P(x) = 1 \) (for maximization). The search space itself is not an array, but an abstract range of candidate values.

**Beginner-Friendly Explanation**
Normally, binary search finds a number inside a sorted array. Binary Search on Answer flips this: instead of searching for a number *in* a list, you're searching for the best number *from a range*. You guess a value, ask "Is this good enough?", and based on the answer, you narrow down your range. It works when "good enough" stays true once it becomes true.

### Key Characteristics

- **Monotonic Feasibility**: The predicate function \( P(x) \) must be monotonic — once \( P(x) \) becomes true, it stays true for all larger \( x \) (or all smaller \( x \), depending on the problem).
- **Abstract Search Space**: The search space is a numeric range \([low, high]\), not an array with indices.
- **Feasibility Check**: A helper function `isPossible(mid)` determines whether a candidate answer `mid` is valid.
- **No Direct Answer Computation**: The answer itself is never computed directly; it is found by narrowing the range.

### Prerequisites

- Standard binary search (on sorted arrays)
- Understanding of monotonic functions
- Familiarity with greedy algorithms (for feasibility checks)

### Related Programming Areas

- **Optimization Problems**: Minimizing maximum, maximizing minimum
- **Competitive Programming**: Common pattern in Codeforces, LeetCode problems
- **Resource Allocation**: Ship capacity, split arrays, scheduling
- **Numerical Methods**: Finding square roots, Nth roots

### Core Concepts

| Concept | Description |
|---------|-------------|
| Feasibility Function | `isPossible(x)` returns whether candidate `x` satisfies the goal |
| Monotonic Search Space | Feasibility changes once and stays changed |
| Minimum Feasible Value | Smallest `x` where `isPossible(x) == true` |
| Maximum Feasible Value | Largest `x` where `isPossible(x) == true` |
| Optimization Problems | Minimax or maximin problems solvable via this pattern |


## 1. Feasibility Function / Predicate Function (\( P(x) \rightarrow \{0, 1\} \))

### Definitions

**Core Definition**
A feasibility function (or predicate) is a boolean-valued function `isPossible(x)` that determines whether a candidate answer `x` satisfies the problem's requirements.

**Technical Definition**
The predicate \( P: S \rightarrow \{0, 1\} \) maps each candidate value in the search space to a boolean, where \( P(x) = 1 \) indicates feasibility. For Binary Search on Answer to work, \( P \) must be **monotonic**: if \( P(x) = 1 \), then \( P(y) = 1 \) for all \( y > x \) (for minimization problems) or all \( y < x \) (for maximization problems).

**Beginner-Friendly Explanation**
The feasibility function is the "checker" that answers: "If the answer were X, would that work?" It's the easy part — once you can answer this quickly, binary search finds the best X.

### Purposes (All begin with "To")

- **To** provide a fast, boolean test for any candidate answer
- **To** enable binary search by establishing the monotonic boundary
- **To** separate the feasibility logic from the search logic
- **To** reduce the problem to a series of decision problems

### Syntax Rules and Structure

**General Syntax (Python)**
```python
def isPossible(candidate, params) -> bool:
    """
    Check if 'candidate' answer satisfies the problem.
    Must be monotonic: once True, stays True.
    """
    # Implement feasibility logic
    # Usually greedy, BFS, or simple iteration
    return feasibility_result
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `candidate` | The mid value being tested |
| `params` | Original problem data |
| Return value | `True` if feasible, `False` otherwise |
| Monotonicity | `isPossible(x) == True` implies `isPossible(y) == True` for all `y > x` |

**Syntax Rules**

- The function must run in **O(n)** or **O(n log n)** to keep overall complexity manageable
- It must be **monotonic**: the feasible region must be a contiguous block, not interspersed
- Early termination within the function is encouraged (e.g., stop when a bound is exceeded)

**Constraints and Limitations**

- If the predicate is not monotonic, binary search will produce incorrect results
- The predicate's complexity determines the overall algorithm's complexity

### Annotated Code Examples

**Example 1: Feasibility for Koko Eating Bananas (Python)**

```python
import math

def can_eat_all(piles, speed, h):
    """
    Feasibility check: Can Koko eat all piles at 'speed' within 'h' hours?
    Monotonic: higher speed always takes <= hours.
    """
    total_hours = 0
    for pile in piles:
        # Ceiling division: hours to finish this pile
        total_hours += (pile + speed - 1) // speed  # Equivalent to ceil(pile / speed)
        if total_hours > h:
            return False  # Early exit
    return total_hours <= h

# Test
piles = [3, 6, 7, 11]
h = 8
for speed in [1, 2, 3, 4, 5, 6]:
    print(f"Speed {speed}: {can_eat_all(piles, speed, h)}")
```

**Expected Output:**
```
Speed 1: False
Speed 2: False
Speed 3: False
Speed 4: True
Speed 5: True
Speed 6: True
```

**Why This Output Occurs**: At speed 3, total hours = ceil(3/3) + ceil(6/3) + ceil(7/3) + ceil(11/3) = 1+2+3+4 = 10 > 8. At speed 4: 1+2+2+3 = 8 ≤ 8. The predicate is monotonic — once true at speed 4, stays true for all higher speeds.

**Example 2: Feasibility for Aggressive Cows (Python)**

```python
def can_place_cows(stalls, min_dist, k):
    """
    Feasibility: Can we place k cows with at least 'min_dist' apart?
    Monotonic: smaller distance is easier to satisfy.
    """
    cows_placed = 1
    last_position = stalls[0]
    for i in range(1, len(stalls)):
        if stalls[i] - last_position >= min_dist:
            cows_placed += 1
            last_position = stalls[i]
            if cows_placed >= k:
                return True
    return False

# Test
stalls = [1, 2, 4, 8, 9]
k = 3
for dist in [1, 2, 3, 4, 5]:
    print(f"Min distance {dist}: {can_place_cows(stalls, dist, k)}")
```

**Expected Output:**
```
Min distance 1: True
Min distance 2: True
Min distance 3: True
Min distance 4: False
Min distance 5: False
```

**Why This Output Occurs**: At distance 3, cows can be placed at 1, 4, 8 (distances 3 and 4). At distance 4, placing at 1, then next ≥ 5 (4 is only 3 away), so 8 or 9 works but then no third cow. The predicate is monotonic in the opposite direction: once false, stays false for larger distances.

### Real-World Cases

- **Load Balancing**: Can `k` processors handle the workload if each has capacity `X`?
- **Scheduling**: Can all tasks finish within `T` time given `k` machines?
- **Network Design**: Can we place `k` routers with minimum separation `d`?

### References

- Codeforces - Binary Search on Answer (Feasibility Check) - https://codeforces.com/topic/143666/en9
- LeetCode - Hidden Pattern 5: Binary Search on Answer - https://leetcode.com/discuss/post/8468320/hidden-pattern-5-binary-search-on-answer-qk0v/


## 2. Monotonic Search Space

### Definitions

**Core Definition**
A monotonic search space is a range of candidate answers where the feasibility predicate, once it becomes true, remains true for all larger (or smaller) values.

**Technical Definition**
For a minimization problem, the search space exhibits the pattern `False, False, ..., False, True, True, ..., True`. The boundary between `False` and `True` is the answer. For maximization, the pattern is reversed.

**Beginner-Friendly Explanation**
Think of a dimmer switch: below a certain level, the light is off; at and above that level, it's on. Once it turns on, it stays on for all higher levels. The "on point" is what you're searching for.

### Purposes (All begin with "To")

- **To** guarantee that binary search will find the correct boundary
- **To** enable the search to discard half the space at each step
- **To** identify whether a problem is suitable for Binary Search on Answer

### Syntax Rules and Structure

**Monotonic Patterns**

| Problem Type | Pattern | Answer |
|--------------|---------|--------|
| Minimization (find smallest feasible) | `FFFTTT` | First `T` |
| Maximization (find largest feasible) | `TTTFFF` | Last `T` |

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Feasible Region | Contiguous block of `True` values |
| Infeasible Region | Contiguous block of `False` values |
| Boundary | The answer value |

### Annotated Code Examples

**Example 1: Visualizing Monotonicity (Python)**

```python
def is_monotonic(search_space, predicate):
    """Check if predicate results form a valid monotonic pattern."""
    results = [predicate(x) for x in search_space]
    print(f"Values:   {search_space}")
    print(f"Feasible: {['T' if r else 'F' for r in results]}")
    
    # Check for at most one transition
    transitions = sum(1 for i in range(1, len(results)) 
                      if results[i] != results[i-1])
    print(f"Transitions: {transitions} ({'Monotonic' if transitions <= 1 else 'NOT Monotonic'})")
    return transitions <= 1


# Minimization example (Koko)
print("=== Minimization Pattern ===")
is_monotonic(range(1, 9), lambda speed: (sum((p + speed - 1)//speed for p in [3,6,7,11])) <= 8)

# Maximization example (Aggressive Cows)
print("\n=== Maximization Pattern ===")
is_monotonic(range(1, 6), lambda d: (lambda stalls, k: (
    lambda: True if sum(1 for i in range(1, len(stalls)) if stalls[i] - stalls[i-1] >= d) >= k-1 else False
))([1,2,4,8,9], 3)())
```

**Expected Output:**
```
=== Minimization Pattern ===
Values:   [1, 2, 3, 4, 5, 6, 7, 8]
Feasible: ['F', 'F', 'F', 'T', 'T', 'T', 'T', 'T']
Transitions: 1 (Monotonic)

=== Maximization Pattern ===
Values:   [1, 2, 3, 4, 5]
Feasible: ['T', 'T', 'T', 'F', 'F']
Transitions: 1 (Monotonic)
```

**Why This Output Occurs**: Both predicates exhibit exactly one transition. Koko's feasibility flips from False to True at speed 4. Aggressive Cows flips from True to False at distance 4. This single-transition property is what makes binary search valid.

### Real-World Cases

- **Quality Control**: Testing if a product passes at various tolerance levels
- **Budget Planning**: Determining minimum budget for project feasibility
- **Performance Tuning**: Finding minimum configuration that meets requirements

### References

- Codeforces - Monotonicity Requirement - https://codeforces.com/blog/entry/143666
- Bohrium - Monotonic "Yes/No" Property - https://scipedia.bohrium.com/en/sciencepedia/feynman/keyword/binary_search_on_the_answer


## 3. Minimum Feasible Value

### Definitions

**Core Definition**
The minimum feasible value is the smallest candidate answer for which the feasibility predicate returns `True`. This is the answer to minimization problems.

**Technical Definition**
Given a monotonic predicate \( P(x) \) with pattern `FFFTTT`, the minimum feasible value is \( \min\{x \in [low, high] : P(x) = 1\} \). Binary search maintains the invariant that the answer lies in `[low, high]` and narrows until `low == high`.

**Beginner-Friendly Explanation**
If speeds 1-3 fail but speed 4 works, then 4 is the minimum feasible speed. You want the *smallest* number that gets the job done.

### Purposes (All begin with "To")

- **To** solve minimization problems (minimum speed, capacity, time)
- **To** find the smallest resource allocation that meets requirements
- **To** minimize costs or maximum loads

### Syntax Rules and Structure

**General Syntax (Python)**
```python
def min_feasible(low, high, is_possible):
    """Find smallest x in [low, high] where is_possible(x) is True."""
    while low < high:
        mid = low + (high - low) // 2
        if is_possible(mid):
            high = mid      # Try smaller (minimization)
        else:
            low = mid + 1   # Need larger
    return low
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `low < high` | Loop until converged |
| `is_possible(mid)` | Feasibility check |
| `high = mid` | On success, search left for smaller |
| `low = mid + 1` | On failure, search right for larger |

### Annotated Code Examples

**Example 1: Minimum Speed (Koko Eating Bananas)**

```python
def min_eating_speed(piles, h):
    """Find minimum speed to eat all bananas within h hours."""
    def can_finish(speed):
        return sum((pile + speed - 1) // speed for pile in piles) <= h
    
    low, high = 1, max(piles)
    while low < high:
        mid = low + (high - low) // 2
        if can_finish(mid):
            high = mid      # Try slower
        else:
            low = mid + 1   # Need faster
    return low

piles = [3, 6, 7, 11]
h = 8
print(f"Minimum speed: {min_eating_speed(piles, h)}")  # 4
```

**Expected Output:**
```
Minimum speed: 4
```

**Why This Output Occurs**: The binary search narrows from `[1, 11]` to find speed 4, the smallest value for which `can_finish(4)` is True.

### Real-World Cases

- **Minimum Ship Capacity**: Smallest capacity to ship all packages in D days
- **Minimum Days for Bouquets**: Smallest wait time for m bouquets
- **Minimum Divisor**: Smallest divisor given a threshold

### References

- LeetCode - Binary Search on Answer Template (Minimization) - https://leetcode.com/discuss/post/7085149/binary-search-on-answers-by-mr_saivardha-jmtn/


## 4. Maximum Feasible Value

### Definitions

**Core Definition**
The maximum feasible value is the largest candidate answer for which the feasibility predicate returns `True`. This is the answer to maximization problems.

**Technical Definition**
Given a monotonic predicate \( P(x) \) with pattern `TTTFFF`, the maximum feasible value is \( \max\{x \in [low, high] : P(x) = 1\} \). Binary search maintains the invariant that the answer lies in `[low, high]` and narrows until convergence.

**Beginner-Friendly Explanation**
If distance 3 works but distance 4 fails, then 3 is the maximum feasible distance. You want the *largest* number that still works.

### Purposes (All begin with "To")

- **To** solve maximization problems (maximum distance, minimum separation)
- **To** find the largest value satisfying constraints
- **To** maximize minimum distances or minimum values

### Syntax Rules and Structure

**General Syntax (Python)**
```python
def max_feasible(low, high, is_possible):
    """Find largest x in [low, high] where is_possible(x) is True."""
    result = low
    while low <= high:
        mid = low + (high - low) // 2
        if is_possible(mid):
            result = mid     # Record and try larger
            low = mid + 1
        else:
            high = mid - 1   # Too large
    return result
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `low <= high` | Loop until exhausted |
| `is_possible(mid)` | Feasibility check |
| `low = mid + 1` | On success, search right for larger |
| `high = mid - 1` | On failure, search left |

### Annotated Code Examples

**Example 1: Maximum Minimum Distance (Aggressive Cows)**

```python
def aggressive_cows(stalls, k):
    """Find maximum minimum distance to place k cows."""
    stalls.sort()
    
    def can_place(dist):
        cows = 1
        last = stalls[0]
        for i in range(1, len(stalls)):
            if stalls[i] - last >= dist:
                cows += 1
                last = stalls[i]
                if cows >= k:
                    return True
        return False
    
    low, high = 1, stalls[-1] - stalls[0]
    result = 1
    while low <= high:
        mid = low + (high - low) // 2
        if can_place(mid):
            result = mid      # Valid, try larger
            low = mid + 1
        else:
            high = mid - 1    # Too large
    return result

stalls = [1, 2, 4, 8, 9]
k = 3
print(f"Max min distance: {aggressive_cows(stalls, k)}")  # 3
```

**Expected Output:**
```
Max min distance: 3
```

**Why This Output Occurs**: Binary search from `[1, 8]` finds that distance 3 is the largest for which 3 cows can be placed.

### Real-World Cases

- **Maximum Minimum Distance**: Aggressive Cows, Magnetic Force Between Balls
- **Maximum Capacity**: Largest feasible container size
- **Maximum Threshold**: Largest value satisfying quality constraints

### References

- LeetCode - Aggressive Cows Binary Search - https://leetcode.com/discuss/post/8494555/java-aggressive-cows-binary-search-by-ha-cwv5/


## 5. Optimization Problems (Koko Eating Bananas, Aggressive Cows)

### Definitions

**Core Definition**
Optimization problems in this context are problems asking for the minimum or maximum value satisfying a condition, where the condition can be checked efficiently for any candidate value.

**Technical Definition**
These are problems of the form: find \( x^* \in [low, high] \) such that \( x^* = \arg\min\{x : P(x)\} \) or \( x^* = \arg\max\{x : P(x)\} \), where \( P \) is monotonic and efficiently computable.

**Beginner-Friendly Explanation**
These are "find the best X" problems. Instead of computing the answer directly, you test candidates: "Is X good enough?" and use the answer to narrow down.

### Purposes (All begin with "To")

- **To** solve minimax and maximin optimization problems
- **To** avoid complex DP or greedy proofs by using binary search
- **To** reduce O(n²) or brute force solutions to O(n log n)
- **To** handle problems where the answer is a threshold value

### Syntax Rules and Structure

**General Problem Identification**

| Clue | Problem Type |
|------|-------------|
| "minimum possible value" | Minimization |
| "maximum possible value" | Maximization |
| "minimize the maximum" | Minimization |
| "maximize the minimum" | Maximization |
| "can we achieve X?" | Feasibility check |

### Annotated Code Examples

**Example 1: Koko Eating Bananas (Complete Solution)**

```python
def min_eating_speed(piles, h):
    """
    Koko eats bananas at speed k per hour.
    Find minimum k to finish all piles within h hours.
    """
    def can_finish(k):
        hours = 0
        for pile in piles:
            hours += (pile + k - 1) // k  # ceil(pile/k)
            if hours > h:
                return False
        return True
    
    low, high = 1, max(piles)
    while low < high:
        mid = low + (high - low) // 2
        if can_finish(mid):
            high = mid
        else:
            low = mid + 1
    return low

# Test
print(min_eating_speed([3, 6, 7, 11], 8))   # 4
print(min_eating_speed([30, 11, 23, 4, 20], 5))  # 30
print(min_eating_speed([30, 11, 23, 4, 20], 6))  # 23
```

**Expected Output:**
```
4
30
23
```

**Why This Output Occurs**: The search space is `[1, max(piles)]`. Feasibility is monotonic: if Koko can finish at speed k, she can finish at any speed > k.

**Example 2: Aggressive Cows (Complete Solution)**

```python
def aggressive_cows(stalls, k):
    """
    Place k cows in stalls to maximize minimum distance.
    """
    stalls.sort()
    
    def can_place(min_dist):
        count = 1
        last = stalls[0]
        for i in range(1, len(stalls)):
            if stalls[i] - last >= min_dist:
                count += 1
                last = stalls[i]
                if count >= k:
                    return True
        return False
    
    low, high = 1, stalls[-1] - stalls[0]
    ans = 1
    while low <= high:
        mid = low + (high - low) // 2
        if can_place(mid):
            ans = mid
            low = mid + 1   # Try larger distance
        else:
            high = mid - 1  # Reduce distance
    return ans

# Test
print(aggressive_cows([1, 2, 4, 8, 9], 3))  # 3
print(aggressive_cows([1, 2, 3, 4, 5], 3))  # 2
```

**Expected Output:**
```
3
2
```

**Why This Output Occurs**: Binary search on distance. Feasibility is monotonic in reverse: if distance d works, any smaller distance also works.

### Real-World Cases

- **Ship Capacity**: Minimum ship capacity to deliver packages in D days
- **Split Array Largest Sum**: Minimize maximum subarray sum when splitting into k parts
- **Magnetic Force**: Maximum minimum distance between placed balls
- **Factory Machines**: Minimum time to produce t products

### References

- LeetCode - Hidden Pattern 5: Binary Search on Answer - https://leetcode.com/discuss/post/8468320/hidden-pattern-5-binary-search-on-answer-qk0v/
- LeetCode - Koko Eating Bananas (Binary Search on Answer) - https://leetcode.com/problems/koko-eating-bananas/solutions/7466306/binary-search-on-answer-on-log-m-intuiti-58p4/
- Codeforces - Binary Search on Answer (Factory Machines) - https://codeforces.com/topic/144710/en1
- GitHub - Factory Machines (Monotone Feasibility) - https://github.com/mtuann/competitive-programming-cpp/blob/main/practice/ladders/foundations/binary-search/factorymachines.md


## Consolidated References

- LeetCode - Hidden Pattern 5: Binary Search on Answer - https://leetcode.com/discuss/post/8468320/hidden-pattern-5-binary-search-on-answer-qk0v/
- Codeforces - Binary Search on the Answer - https://codeforces.com/topic/143666/en9
- Bohrium - Binary Search on the Answer - https://scipedia.bohrium.com/en/sciencepedia/feynman/keyword/binary_search_on_the_answer
- LeetCode - Koko Eating Bananas Solutions - https://leetcode.com/problems/koko-eating-bananas/solutions/8402570/clear-explanation-by-pact-1451/
- LeetCode - Binary Search On Answers (Mr_Saivardha) - https://leetcode.com/discuss/post/7085149/binary-search-on-answers-by-mr_saivardha-jmtn/
- LeetCode - Aggressive Cows (Java) - https://leetcode.com/discuss/post/8494555/java-aggressive-cows-binary-search-by-ha-cwv5/
- GitHub - Factory Machines (Binary Search on Answer) - https://github.com/mtuann/competitive-programming-cpp/blob/main/practice/ladders/foundations/binary-search/factorymachines.md
- LeetCode - Binary Search On Answer Template - https://leetcode.com/discuss/post/3444552/binary-search-on-answer-template-generic-ysn4/
- Codeforces - Binary Search on Answer (Book Allocation) - https://codeforces.com/topic/144710/en1
- LeetCode - Koko Eating Bananas (Binary Search on Answer) - https://leetcode.com/problems/koko-eating-bananas/solutions/7466306/binary-search-on-answer-on-log-m-intuiti-58p4/
- GitHub - A2Z-DSA-Solutions (Koko Eating Bananas) - https://github.com/The-EleetCoder/A2Z-DSA-Solutions/blob/main/04_BinarySearch/02_BinarySearchOnAnswers/03_KokoEatingBananas.cpp
- GitHub - A2Z-DSA-Solutions (Aggressive Cows) - https://github.com/The-EleetCoder/A2Z-DSA-Solutions/blob/main/04_BinarySearch/02_BinarySearchOnAnswers/08_AggressiveCows.cpp
- LeetCode - Binary Search On Answer (KOKO Type) - https://leetcode.com/discuss/post/3725477/binary-search-on-answer-koko-type-by-sam-a350/
- GitHub - Aggressive Cows (Java) - https://raw.githubusercontent.com/Sanjanayadav07/Java-DSA-Coding-Ninja/refs/heads/main/Chapter%207%20-%20Binary%20search/Aggressive%20Cows.md
- GitHub - Aggressive Cows (Striver) - https://github.com/Bharat2044/Striver-A2Z-DSA-Sheet-Solution-using-JAVA/blob/main/Step%2004%20%3A%20Binary%20Search%20%5B1D%2C%202D%20Arrays%2C%20Search%20Space%5D/Lec%202%3A%20BS%20on%20Answers/08_AgressiveCows.java