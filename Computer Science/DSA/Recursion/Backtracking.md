# Backtracking: Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**
Backtracking is a systematic algorithmic strategy for finding solutions to constraint satisfaction problems by incrementally building candidates and abandoning (backtracking) a candidate as soon as it is determined that it cannot be completed to a valid solution.

**Technical Definition**
Backtracking performs a depth-first search of a state-space tree, where each node represents a partial candidate solution. At each node, the algorithm checks whether the partial solution can be extended to a valid complete solution (via a bounding/rejection function). If not, the entire subtree rooted at that node is pruned. Otherwise, the algorithm recursively explores all extensions of the current candidate.

**Beginner-Friendly Explanation**
Backtracking is like solving a maze by trying paths one at a time. When you hit a dead end, you go back to the last fork and try a different path. You keep a notebook of your current path so you never lose track of where you are.

### Key Characteristics

- **Depth-First Exploration**: Backtracking traverses the state-space tree depth-first, using recursion (or an explicit stack) as the natural implementation.
- **Incremental Construction**: Solutions are built step by step, with each step adding one decision to the partial candidate.
- **Pruning Power**: The algorithm's efficiency depends critically on detecting invalid partial solutions early and abandoning their subtrees.
- **Memory Efficiency**: Backtracking uses O(depth) auxiliary space, storing only the current path, not the entire explored tree.

### Prerequisites

- Recursion fundamentals (base case, recursive case, call stack)
- Tree traversal concepts (DFS)
- Basic set operations (add, remove, contains)

### Related Programming Areas

- **Constraint Satisfaction**: N-Queens, Sudoku, crossword puzzles
- **Combinatorial Generation**: Subsets, permutations, combinations
- **Path Finding**: Maze solving, word search, graph coloring
- **Optimization**: Knapsack problem, branch-and-bound

### Core Concepts

| Concept | Description |
|---------|-------------|
| State-Space Search | Exploring all possible configurations systematically |
| Decision Trees | Tree where each level represents a choice |
| Constraint Satisfaction | Problems requiring all constraints to be satisfied |
| Pruning (Bounding) | Cutting branches that cannot yield valid solutions |
| Choose-Explore-Undo | The three-step backtracking pattern |
| Bitmasking | Using integer bits for ultra-fast state tracking |

## 1. State-Space Search

### Definitions

**Core Definition**
State-space search is the process of exploring all possible states of a problem, where each state represents a partial or complete configuration, and transitions between states correspond to making decisions.

**Technical Definition**
A state-space tree (or decision tree) has nodes representing problem states and edges representing decisions. The root is the empty state, and leaves are either complete solutions or dead ends. Backtracking performs a depth-first traversal of this tree, pruning subtrees that cannot lead to solutions.

**Beginner-Friendly Explanation**
Imagine a game of 20 Questions. Each question narrows down the possibilities. The state-space tree is like a map of all possible question sequences and their outcomes. Backtracking explores this map depth-first, stopping down branches that can't lead to the answer.

### Purposes (All begin with "To")

- **To** systematically explore all possible configurations of a problem
- **To** guarantee finding all solutions (or proving none exist) in a finite search space
- **To** provide a framework for applying pruning optimizations
- **To** model combinatorial problems as tree traversals

### Syntax Rules and Structure

**General State-Space Model**

```
State: (x₁, x₂, ..., xₖ) — a k-tuple representing partial decisions
Decision Set: D(x₁, ..., xₖ₋₁) — valid choices for the next decision
Root: empty tuple ()
Goal: complete tuple (x₁, ..., xₙ) satisfying all constraints
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| State | Partial assignment of variables |
| Decision Set | Available choices at each state |
| Transition | Adding a choice to the state |
| Goal Test | Checking if state is a complete solution |

### Annotated Code Examples

**Example 1: Subset Sum State-Space Tree (Python)**

```python
def subset_sum(nums, target):
    """
    Find all subsets of nums that sum to target.
    State-space tree: each level decides include/exclude for one number.
    """
    results = []
    
    def backtrack(index, current_sum, path):
        # Base case: complete decision sequence
        if index == len(nums):
            if current_sum == target:
                results.append(path[:])
            return
        
        # Decision 1: EXCLUDE nums[index]
        backtrack(index + 1, current_sum, path)
        
        # Decision 2: INCLUDE nums[index]
        # Prune: only include if sum doesn't exceed target
        if current_sum + nums[index] <= target:
            path.append(nums[index])
            backtrack(index + 1, current_sum + nums[index], path)
            path.pop()  # Undo
    
    backtrack(0, 0, [])
    return results


# Demonstration
nums = [3, 34, 4, 12, 5, 2]
target = 9

solutions = subset_sum(nums, target)
print(f"Numbers: {nums}, Target: {target}")
print(f"Subsets summing to {target}:")
for sol in solutions:
    print(f"  {sol} = {sum(sol)}")
```

**Expected Output:**
```
Numbers: [3, 34, 4, 12, 5, 2], Target: 9
Subsets summing to 9:
  [4, 5] = 9
  [3, 4, 2] = 9
```

**Why This Output Occurs**: The state-space tree has depth 6 (one level per number). At each node, the algorithm branches into "exclude" and "include" decisions. The pruning condition `current_sum + nums[index] <= target` cuts branches where the sum already exceeds the target, reducing the search space.

### Real-World Cases

- **Knapsack Problem**: Each item is included or excluded (binary decision tree)
- **Graph Coloring**: Each vertex is assigned a color (branching factor = number of colors)
- **Sudoku**: Each empty cell is assigned a digit 1-9

### References

- UiO INF4130 - Search in State-spaces - https://www.uio.no/studier/emner/matnat/ifi/INF4130/h16/undervisningsmateriale/forelesning4.pdf
- METU CENG567 - Backtracking and B&B Design Technique - https://saksagan.ceng.metu.edu.tr/courses/secondprog/ceng567/lect/Yazici-Backtrack-B&B-2007.pdf

## 2. Decision Trees

### Definitions

**Core Definition**
A decision tree is a tree structure where each internal node represents a decision point, each branch represents a choice, and each leaf represents a complete or terminal state.

**Technical Definition**
For a problem with n decision variables, the decision tree has depth n. At level k, each node represents a partial assignment of the first k variables, and its children represent the possible values for variable k+1. The decision tree may be static (fixed structure) or dynamic (structure depends on problem input).

**Beginner-Friendly Explanation**
A decision tree is like a flowchart of "what if" scenarios. At each step, you choose one option from several. The tree shows all possible paths you could take from the start.

### Purposes (All begin with "To")

- **To** visualize all possible decision sequences for a problem
- **To** provide a framework for analyzing search space size
- **To** identify where pruning can eliminate large subtrees
- **To** guide the choice of variable and value ordering heuristics

### Syntax Rules and Structure

**Decision Tree Structure**

| Level | Nodes Represent | Children Represent |
|-------|-----------------|-------------------|
| 0 | Empty state | Choices for variable 1 |
| 1 | Assignments to variable 1 | Choices for variable 2 |
| k | Assignments to variables 1..k | Choices for variable k+1 |
| n | Complete assignments | None (leaves) |

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Branching Factor | Number of choices at each level |
| Depth | Number of decision variables (n) |
| Leaf | Complete assignment or dead end |
| Path | Sequence of decisions from root to node |

### Annotated Code Examples

**Example 1: N-Queens Decision Tree (Python)**

```python
def solve_n_queens(n):
    """
    N-Queens: Place n queens on n×n board, no two attacking.
    Decision tree: each level = one row; children = column choices.
    """
    solutions = []
    
    def is_safe(board, row, col):
        # Check column and diagonals with previous queens
        for r in range(row):
            c = board[r]
            if c == col:  # Same column
                return False
            if abs(c - col) == abs(r - row):  # Same diagonal
                return False
        return True
    
    def backtrack(row, board):
        # Base case: all queens placed
        if row == n:
            solutions.append(board[:])
            return
        
        # Try each column in current row
        for col in range(n):
            if is_safe(board, row, col):  # PRUNE invalid placements
                board.append(col)          # CHOOSE
                backtrack(row + 1, board)  # EXPLORE
                board.pop()                # UNCHOOSE
    
    backtrack(0, [])
    return solutions


# Demonstration
n = 4
solutions = solve_n_queens(n)
print(f"Number of {n}-Queens solutions: {len(solutions)}")
for sol in solutions:
    print(f"  Columns for rows 0-{n-1}: {sol}")
    # Visualize
    for row in range(n):
        line = ""
        for col in range(n):
            line += "Q " if sol[row] == col else ". "
        print(f"    {line}")
    print()
```

**Expected Output:**
```
Number of 4-Queens solutions: 2
  Columns for rows 0-3: [1, 3, 0, 2]
    . Q . . 
    . . . Q 
    Q . . . 
    . . Q . 

  Columns for rows 0-3: [2, 0, 3, 1]
    . . Q . 
    Q . . . 
    . . . Q 
    . Q . . 
```

**Why This Output Occurs**: The decision tree has depth 4 (one level per row) and branching factor 4 (column choices). The `is_safe` function prunes branches where queens attack each other. Only 2 of the 4⁴ = 256 possible assignments are valid solutions.

### Real-World Cases

- **N-Queens**: Branching factor = n, depth = n
- **Sudoku**: Branching factor = 9 (digits), depth = number of empty cells
- **Subset Generation**: Branching factor = 2 (include/exclude), depth = n

### References

- UiO INF4130 - Decision Sequences and State Space Trees - https://www.uio.no/studier/emner/matnat/ifi/INF4130/h16/undervisningsmateriale/forelesning4.pdf
- UiO IN3130 - Backtracking and Branch-and-Bound - https://www.uio.no/studier/emner/matnat/ifi/IN3130/h22/material/chapter-10.pdf

## 3. Constraint Satisfaction

### Definitions

**Core Definition**
Constraint satisfaction problems (CSPs) require assigning values to variables such that all given constraints are satisfied simultaneously.

**Technical Definition**
A CSP consists of a set of variables X = {x₁, ..., xₙ}, a domain D(xᵢ) for each variable, and a set of constraints C = {c₁, ..., cₘ} where each constraint specifies allowed combinations of values. A solution is an assignment where every constraint is satisfied.

**Beginner-Friendly Explanation**
Constraint satisfaction is like scheduling meetings: each meeting needs a time slot, but no two meetings with the same attendee can overlap. The constraints (no overlapping) guide which assignments are valid.

### Purposes (All begin with "To")

- **To** model real-world problems with hard constraints (scheduling, planning)
- **To** find all valid configurations satisfying multiple conditions
- **To** provide a framework for systematic search with pruning
- **To** enable early detection of infeasible partial assignments

### Syntax Rules and Structure

**CSP Components**

| Component | Description | Example (N-Queens) |
|-----------|-------------|-------------------|
| Variables | Decision points | Row positions |
| Domains | Allowed values | Columns 0..n-1 |
| Constraints | Restrictions | No two queens attack |
| Solution | Complete valid assignment | All n queens placed safely |

**Component Breakdown**

| Constraint Type | Description |
|-----------------|-------------|
| Unary | Restricts single variable |
| Binary | Relates two variables |
| Global | Involves many variables |

### Annotated Code Examples

**Example 1: Sudoku Solver with Constraints (Python)**

```python
def solve_sudoku(board):
    """
    Sudoku: Fill 9×9 grid with digits 1-9.
    Constraints: Each row, column, and 3×3 box has 1-9 exactly once.
    """
    def is_valid(board, row, col, num):
        # Check row
        if num in board[row]:
            return False
        # Check column
        for r in range(9):
            if board[r][col] == num:
                return False
        # Check 3×3 box
        box_r, box_c = 3 * (row // 3), 3 * (col // 3)
        for r in range(box_r, box_r + 3):
            for c in range(box_c, box_c + 3):
                if board[r][c] == num:
                    return False
        return True
    
    def backtrack():
        # Find first empty cell
        for r in range(9):
            for c in range(9):
                if board[r][c] == 0:
                    # Try each digit
                    for num in range(1, 10):
                        if is_valid(board, r, c, num):  # PRUNE
                            board[r][c] = num           # CHOOSE
                            if backtrack():             # EXPLORE
                                return True
                            board[r][c] = 0             # UNCHOOSE
                    return False  # No valid digit works
        return True  # All cells filled
    
    if backtrack():
        return board
    return None


# Demonstration
puzzle = [
    [5,3,0,0,7,0,0,0,0],
    [6,0,0,1,9,5,0,0,0],
    [0,9,8,0,0,0,0,6,0],
    [8,0,0,0,6,0,0,0,3],
    [4,0,0,8,0,3,0,0,1],
    [7,0,0,0,2,0,0,0,6],
    [0,6,0,0,0,0,2,8,0],
    [0,0,0,4,1,9,0,0,5],
    [0,0,0,0,8,0,0,7,9]
]

solution = solve_sudoku(puzzle)
if solution:
    print("Solution found:")
    for row in solution:
        print(" ".join(map(str, row)))
```

**Expected Output:**
```
Solution found:
5 3 4 6 7 8 9 1 2
6 7 2 1 9 5 3 4 8
1 9 8 3 4 2 5 6 7
8 5 9 7 6 1 4 2 3
4 2 6 8 5 3 7 9 1
7 1 3 9 2 4 8 5 6
9 6 1 5 3 7 2 8 4
2 8 7 4 1 9 6 3 5
3 4 5 2 8 6 1 7 9
```

**Why This Output Occurs**: The `is_valid` function enforces all three constraints (row, column, box). When a digit violates any constraint, that branch is pruned immediately. The algorithm only explores assignments that maintain validity, dramatically reducing the search space compared to checking constraints only at complete assignments.

### Real-World Cases

- **Scheduling**: Assigning time slots to meetings without conflicts
- **Map Coloring**: Coloring regions so adjacent regions differ
- **Circuit Design**: Assigning signals to wires without interference

### References

- Canada.ca - Backtracking (Constraint Satisfaction) - https://webarchiveweb.wayback.bac-lac.canada.ca/web/20051215000000/http://en.wikipedia.org/wiki/Backtracking
- METU CENG567 - Constraint Satisfaction and Backtracking - https://saksagan.ceng.metu.edu.tr/courses/secondprog/ceng567/lect/Yazici-Backtrack-B&B-2007.pdf

## 4. Pruning (Bounding Functions)

### Definitions

**Core Definition**
Pruning (or bounding) is the process of eliminating branches of the state-space tree that cannot possibly lead to valid or optimal solutions, based on a bounding function.

**Technical Definition**
A bounding function B(X) for a partial solution X satisfies B(X) ≥ P(X), where P(X) is the maximum profit (or minimum cost) achievable by any descendant of X. If B(X) ≤ OptP (the best solution found so far), the subtree rooted at X can be pruned because it cannot improve the current best.

**Beginner-Friendly Explanation**
Pruning is like checking the weather forecast before hiking. If the forecast says a storm is coming, you don't even start the trail. You save time by avoiding paths that can't reach your destination.

### Purposes (All begin with "To")

- **To** reduce the effective search space without losing solutions
- **To** detect infeasible partial solutions early and abandon them
- **To** improve performance from exponential toward polynomial in practice
- **To** enable optimal solutions by eliminating provably suboptimal branches

### Syntax Rules and Structure

**Pruning Conditions**

| Pruning Type | Condition | Action |
|--------------|-----------|--------|
| Feasibility | Partial solution violates constraint | Skip subtree |
| Bound | B(X) ≤ OptP (for maximization) | Skip subtree |
| Duplicate | Same state reached via different path | Skip subtree |

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Bounding Function | Estimates best possible outcome from X |
| OptP | Best solution found so far |
| Rejection Test | Returns true if subtree should be pruned |

### Annotated Code Examples

**Example 1: Branch-and-Bound for 0/1 Knapsack (Python)**

```python
def knapsack_branch_bound(weights, values, capacity):
    """
    0/1 Knapsack with branch-and-bound pruning.
    Bounding function: fractional knapsack relaxation (greedy).
    """
    n = len(weights)
    # Sort by value/weight ratio descending
    items = sorted(zip(weights, values), key=lambda x: x[1]/x[0], reverse=True)
    weights = [w for w, v in items]
    values = [v for w, v in items]
    
    best_value = 0
    best_items = []
    
    def bound(index, current_weight, current_value):
        """
        Upper bound: assume we can take fractional items.
        Greedy: take items until capacity exhausted.
        """
        total = current_value
        remaining = capacity - current_weight
        for i in range(index, n):
            if weights[i] <= remaining:
                total += values[i]
                remaining -= weights[i]
            else:
                total += values[i] * (remaining / weights[i])
                break
        return total
    
    def backtrack(index, current_weight, current_value, selected):
        nonlocal best_value, best_items
        
        # Update best if we have a better complete solution
        if current_value > best_value:
            best_value = current_value
            best_items = selected[:]
        
        # Base case
        if index >= n:
            return
        
        # PRUNE: if upper bound can't beat best, skip
        if bound(index, current_weight, current_value) <= best_value:
            return
        
        # Decision 1: INCLUDE item (if weight allows)
        if current_weight + weights[index] <= capacity:
            selected.append(index)
            backtrack(index + 1, 
                     current_weight + weights[index], 
                     current_value + values[index], 
                     selected)
            selected.pop()  # UNCHOOSE
        
        # Decision 2: EXCLUDE item
        backtrack(index + 1, current_weight, current_value, selected)
    
    backtrack(0, 0, 0, [])
    return best_value, best_items


# Demonstration
weights = [10, 20, 30]
values = [60, 100, 120]
capacity = 50

max_val, items = knapsack_branch_bound(weights, values, capacity)
print(f"Capacity: {capacity}")
print(f"Maximum value: {max_val}")
print(f"Items selected (indices): {items}")
```

**Expected Output:**
```
Capacity: 50
Maximum value: 220
Items selected (indices): [1, 2]
```

**Why This Output Occurs**: The bounding function computes the fractional knapsack upper bound. When this bound cannot exceed the current best, the entire subtree is pruned. For this instance, the optimal is items 1 and 2 (20+30=50 weight, 100+120=220 value). The branch-and-bound prunes branches that cannot improve this.

### Real-World Cases

- **Knapsack Problem**: Fractional relaxation provides tight bounds
- **Maximum Clique**: Graph coloring provides bounds
- **Traveling Salesman**: Minimum spanning tree relaxation

### References

- SFU MATH 343 - Bounding Functions - https://people.math.sfu.ca/~kya17/teaching/math343/20-343.pdf
- Concordia University - Bounding Functions in Backtracking - https://spectrum.library.concordia.ca/id/eprint/1290/1/MQ59327.pdf

## 5. Choosing, Exploring, Undoing Choices

### Definitions

**Core Definition**
The choose-explore-undo pattern is the fundamental three-step structure of every backtracking algorithm: make a decision, recurse to explore its consequences, then undo the decision to restore state before trying the next option.

**Technical Definition**
At each node in the state-space tree:
1. **Choose**: Apply a decision (append to list, mark visited, place piece)
2. **Explore**: Recursively search from the new state
3. **Unchoose**: Reverse the decision exactly (pop, unmark, remove)

State restoration is critical: the state must be identical before and after the recursive call as seen by the caller.

**Beginner-Friendly Explanation**
It's like trying on clothes. You pick a shirt (choose), look in the mirror (explore), then take it off (unchoose) before trying the next one. You always return to the same starting point.

### Purposes (All begin with "To")

- **To** ensure each branch of the decision tree starts from a clean state
- **To** prevent state corruption across sibling branches
- **To** enable correct exploration of all possibilities
- **To** maintain the invariant that state reflects only the current path

### Syntax Rules and Structure

**Universal Template**

```python
def backtrack(state, choices):
    if is_solution(state):
        record(state)
        return
    
    for choice in choices:
        if is_valid(state, choice):  # PRUNE
            apply(state, choice)      # CHOOSE
            backtrack(state, ...)      # EXPLORE
            undo(state, choice)        # UNCHOOSE
```

**Component Breakdown**

| Step | Operation | Example (List) |
|------|-----------|----------------|
| Choose | Add element | `path.append(x)` |
| Explore | Recurse | `backtrack(...)` |
| Unchoose | Remove element | `path.pop()` |

### Annotated Code Examples

**Example 1: Subset Generation with State Restoration (Python)**

```python
def generate_subsets(nums):
    """
    Generate all subsets using choose-explore-unchoose.
    State: current path list.
    """
    result = []
    path = []  # Shared mutable state
    
    def backtrack(index):
        # At every node, record current subset
        result.append(path[:])  # Copy!
        
        # Try including each remaining element
        for i in range(index, len(nums)):
            path.append(nums[i])      # CHOOSE
            backtrack(i + 1)          # EXPLORE
            path.pop()                # UNCHOOSE (critical!)
    
    backtrack(0)
    return result


# Demonstration
nums = [1, 2, 3]
subsets = generate_subsets(nums)
print(f"All subsets of {nums}:")
for s in subsets:
    print(f"  {s}")
```

**Expected Output:**
```
All subsets of [1, 2, 3]:
  []
  [1]
  [1, 2]
  [1, 2, 3]
  [1, 3]
  [2]
  [2, 3]
  [3]
```

**Why This Output Occurs**: The `path` list is shared across all recursive calls. After exploring `[1, 2]`, `path.pop()` removes `2`, restoring `path` to `[1]`. Without the pop, the next branch would incorrectly start with `[1, 2]` instead of `[1]`. This state restoration is what makes backtracking correct.

### Real-World Cases

- **Permutations**: Swap elements, recurse, swap back
- **N-Queens**: Place queen, recurse, remove queen
- **Word Search**: Mark cell, recurse, unmark cell

### References

- GitHub - Backtracking Fundamentals (State Restoration) - https://raw.githubusercontent.com/archman-dev/website/refs/heads/main/docs/algorithms-and-data-structures/backtracking/backtracking-fundamentals.mdx
- GitHub - Backtracking Theory (Choose-Explore-Undo) - https://raw.githubusercontent.com/ReddyBytes/Python-DSA-API-Mastery/refs/heads/main/02_DSA_Mastery/20_backtracking/theory.md
- LeetCode - Backtracking Cheat Sheet - https://leetcode.com/discuss/post/8531445/set-14-backtracking-python-dsa-cheat-she-0q8i/

## 6. Bitmasking for Ultra-Fast State Tracking

### Definitions

**Core Definition**
Bitmasking uses individual bits of an integer to represent binary states of multiple elements, enabling O(1) state checks and updates through bitwise operations.

**Technical Definition**
For a set of n elements, a bitmask of n bits represents a subset: bit i is 1 if element i is included/used, 0 otherwise. Operations: set bit (`mask |= 1 << i`), clear bit (`mask ^= 1 << i`), test bit (`(mask >> i) & 1`).

**Beginner-Friendly Explanation**
A bitmask is like a row of light switches. Each switch (bit) represents one element. Flipping a switch is a single fast operation, much faster than updating an array or set.

### Purposes (All begin with "To")

- **To** replace visited arrays with a single integer for constant-space tracking
- **To** enable O(1) state checks and updates using bitwise operations
- **To** reduce memory overhead and improve cache performance
- **To** enable elegant, concise state representation for problems with ≤ 32 (or 64) elements

### Syntax Rules and Structure

**Bitmask Operations**

| Operation | Syntax | Description |
|-----------|--------|-------------|
| Set bit i | `mask \|= (1 << i)` | Mark element i as used |
| Clear bit i | `mask ^= (1 << i)` | Unmark element i |
| Test bit i | `(mask >> i) & 1` | Check if element i is used |
| Check all used | `mask == (1 << n) - 1` | All n elements used |
| Count used | `mask.bit_count()` | Number of set bits |

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `mask` | Integer holding bit states |
| `1 << i` | Bit at position i |
| `\|=` | Set (OR) |
| `^=` | Toggle (XOR) |

### Annotated Code Examples

**Example 1: N-Queens with Bitmasking (Python)**

```python
def solve_n_queens_bitmask(n):
    """
    N-Queens using bitmasks for columns and diagonals.
    Each mask tracks which positions are under attack.
    """
    solutions = []
    
    def backtrack(row, cols, diag1, diag2, board):
        # Base case: all queens placed
        if row == n:
            solutions.append(board[:])
            return
        
        # Available columns: bits not set in cols, diag1, diag2
        # Diagonals: row+col for one direction, row-col+n for other
        available = ((1 << n) - 1) & ~(cols | diag1 | diag2)
        
        while available:
            # Get lowest set bit
            col_bit = available & -available
            available ^= col_bit  # Clear this bit
            col = col_bit.bit_length() - 1
            
            # CHOOSE: place queen, update masks
            board.append(col)
            
            # EXPLORE: recurse with updated masks
            # Diagonal shifts propagate attacks
            backtrack(row + 1,
                     cols | col_bit,
                     (diag1 | col_bit) << 1,  # Left diagonal shifts left
                     (diag2 | col_bit) >> 1,  # Right diagonal shifts right
                     board)
            
            # UNCHOOSE: remove queen
            board.pop()
    
    backtrack(0, 0, 0, 0, [])
    return solutions


# Demonstration
n = 8
solutions = solve_n_queens_bitmask(n)
print(f"Number of {n}-Queens solutions: {len(solutions)}")
print(f"First solution columns: {solutions[0]}")
```

**Expected Output:**
```
Number of 8-Queens solutions: 92
First solution columns: [0, 4, 7, 5, 2, 6, 1, 3]
```

**Why This Output Occurs**: The bitmask `cols` tracks occupied columns, `diag1` tracks one diagonal direction (shifted left each row), and `diag2` tracks the other (shifted right). The expression `(cols | diag1 | diag2)` gives all attacked positions in the current row in a single bitwise OR. The `available` mask isolates valid positions, and the while loop iterates only over valid columns.

### Real-World Cases

- **N-Queens**: Track columns and diagonals with 3 integers
- **Maximum Independent Set**: Bitmask represents available vertices
- **Traveling Salesman (small n)**: Bitmask represents visited cities

### References

- LeetCode - Bitmasking in Backtracking - https://leetcode.com/discuss/post/6939814/day-27-bitmasking-in-backtracking-shrink-z680/
- LeetCode - N-Queens Pure Bitmasking Solution - https://leetcode.com/problems/n-queens/solutions/8342980/0ms-100-beats-pure-bitmasking-no-extra-a-u71b/
- Competitive Programming 3 - Bitmask Backtracking - /hdd/m0103/deepseek/datasets/pilimi-zlib/libgen_rs_non_fic/1594000/eabccec887295e5da350302deca61d7c

## Consolidated References

- GitHub - Backtracking Fundamentals - https://raw.githubusercontent.com/archman-dev/website/refs/heads/main/docs/algorithms-and-data-structures/backtracking/backtracking-fundamentals.mdx
- MIT CSAIL - Constraint Satisfaction and Games - https://courses.csail.mit.edu/6.034s/handouts/csp-games-handout-07.pdf
- University of Tübingen - Branch and Bound Trajectory Planning - https://publikationen.uni-tuebingen.de/xmlui/bitstream/handle/10900/159754/Dissertation_messmer.pdf
- LeetCode - How to Solve ANY Backtracking Problem - https://leetcode.com/discuss/post/8346110/how-to-solve-any-backtracking-problem-st-bel5/
- LeetCode - Bitmasking in Backtracking - https://leetcode.com/discuss/post/6939814/day-27-bitmasking-in-backtracking-shrink-z680/
- UiO INF4130 - Search in State-spaces - https://www.uio.no/studier/emner/matnat/ifi/INF4130/h16/undervisningsmateriale/forelesning4.pdf
- Canada.ca - Backtracking (Wikipedia Archive) - https://webarchiveweb.wayback.bac-lac.canada.ca/web/20051215000000/http://en.wikipedia.org/wiki/Backtracking
- METU CENG567 - Backtracking and B&B Design Technique - https://saksagan.ceng.metu.edu.tr/courses/secondprog/ceng567/lect/Yazici-Backtrack-B&B-2007.pdf
- LeetCode - Backtracking Python DSA Cheat Sheet - https://leetcode.com/discuss/post/8531445/set-14-backtracking-python-dsa-cheat-she-0q8i/
- Competitive Programming 3 (Archive) - /hdd/m0103/deepseek/datasets/pilimi-zlib/libgen_rs_non_fic/1594000/eabccec887295e5da350302deca61d7c
- UiO IN3130 - Search Strategies - https://www.uio.no/studier/emner/matnat/ifi/IN3130/h24/slides/forelesning-7---search-strategies.pdf
- Arquivo.pt - Backtracking (Wikipedia Archive) - https://preprod.arquivo.pt/noFrame/patching/patching/record/20171208040947mp_/https://en.wikipedia.org/wiki/Backtracking
- SFU MATH 343 - Bounding Functions - https://people.math.sfu.ca/~kya17/teaching/math343/20-343.pdf
- GitHub - Backtracking Theory (State Restoration) - https://raw.githubusercontent.com/ReddyBytes/Python-DSA-API-Mastery/refs/heads/main/02_DSA_Mastery/20_backtracking/theory.md
- LeetCode - N-Queens Pure Bitmasking - https://leetcode.com/problems/n-queens/solutions/8342980/0ms-100-beats-pure-bitmasking-no-extra-a-u71b/
- UiO IN3130 - Chapter 10: Backtracking and Branch-and-Bound - https://www.uio.no/studier/emner/matnat/ifi/IN3130/h22/material/chapter-10.pdf
- Charles University - Constraint Programming Slides - http://ktiml.mff.cuni.cz/%7ebartak/downloads/CPschool05slides.pdf
- Concordia University - Bounding Functions Thesis - https://spectrum.library.concordia.ca/id/eprint/1290/1/MQ59327.pdf
- GitHub - Backtracking Quick Reference Cheatsheet - https://raw.githubusercontent.com/ReddyBytes/Python-DSA-API-Mastery/refs/heads/main/02_DSA_Mastery/20_backtracking/cheetsheet.md
- Competitive Programming 3 - Bitmask Backtracking - /hdd/m0103/deepseek/datasets/pilimi-zlib/libgen_rs_non_fic/1594000/eabccec887295e5da350302deca61d7c