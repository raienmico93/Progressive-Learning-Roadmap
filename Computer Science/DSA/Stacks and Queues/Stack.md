# DSA Python Stack Architecture & Implementation Models

## Topic Overview

### Definitions

**Core Definition**
A stack is a linear abstract data type (ADT) that stores and retrieves elements according to the Last-In-First-Out (LIFO) principle, where insertion and deletion occur exclusively at one designated end called the top.

**Technical Definition**
A stack is a sequential collection with restricted access: `push` adds an element to the top, and `pop` removes the most recently added element from the top. The stack maintains a pointer or index to the top element, and all operations are defined relative to this position.

**Beginner-Friendly Explanation**
Imagine a stack of plates in a cafeteria. You can only add a new plate on top, and you can only take the topmost plate off. The plate you put on last is the first one you take off—that's Last-In-First-Out.

### Key Characteristics

- **LIFO Ordering**: The most recently added element is the first to be removed.
- **Single Access Point**: All operations occur at one end (the top).
- **O(1) Core Operations**: Push, pop, and peek execute in constant time.
- **Two Implementation Models**: Array-based (contiguous) and linked-list-based (dynamic nodes).
- **System vs. User Stacks**: The call stack (system) manages function execution; user-defined stacks solve algorithmic problems.

### Prerequisites

- Understanding of Python classes and object references
- Familiarity with Big-O notation and amortized analysis
- Basic knowledge of arrays (lists) and linked nodes
- Awareness of Python's memory model (references vs. values)

### Related Programming Areas

- **Recursion and Call Stack Management**
- **Expression Evaluation** (infix, postfix, prefix)
- **Backtracking Algorithms** (DFS, maze solving)
- **Undo/Redo Functionality** in applications
- **Balanced Parentheses and Syntax Parsing**

### Core Concepts

| Concept | Description |
|---------|-------------|
| LIFO Principle | Last-In-First-Out ordering for all operations |
| Core Operations | push, pop, peek/top, isEmpty, isFull |
| Array-Based Implementation | Contiguous storage with a top index |
| Linked-List Implementation | Dynamic nodes with head tracking |
| Complexity Profile | O(1) for all core operations (amortized for dynamic arrays) |
| System vs. User Stack | Call stack for execution vs. user-defined data structure |


## 1. The LIFO (Last In, First Out) Abstract Data Type Principle

### Definitions

**Core Definition**
LIFO is a principle where the element most recently added to a collection is the first one to be removed.

**Technical Definition**
In a LIFO structure, the only accessible element at any time is the one at the top of the collection; all insertions and deletions are constrained to this single access point.

**Beginner-Friendly Explanation**
Think of a stack of books on your desk. When you finish reading a book, you place it on top. When you want another book, you take the top one. The last book you stacked is the first one you grab.

### Purposes (All begin with "To")

- **To** enforce restricted access patterns that simplify algorithm design and prevent accidental misuse of data ordering.
- **To** enable efficient reversal of sequences (e.g., reversing strings, backtracking).
- **To** model naturally nested or hierarchical processes such as function calls and expression evaluation.
- **To** provide a predictable, constrained interface that guarantees only the most recent element can be accessed.

### Syntax Rules and Structure

**General LIFO Interface (Python)**
```python
class Stack:
    """Abstract LIFO container."""
    def push(self, item: Any) -> None: ...      # Add to top
    def pop(self) -> Any: ...                   # Remove and return top
    def peek(self) -> Any: ...                  # Return top without removal
    def is_empty(self) -> bool: ...             # Check for elements
    def is_full(self) -> bool: ...              # Check for capacity (fixed-size only)
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `push(item)` | Adds `item` to the top; may raise `OverflowError` if fixed capacity reached |
| `pop()` | Removes and returns the top element; raises `IndexError` on empty stack |
| `peek()` | Returns the top element without removing; raises `IndexError` on empty stack |
| `is_empty()` | Returns `True` if no elements remain |
| `is_full()` | Returns `True` if the stack has reached its fixed capacity |

**Syntax Rules**

- Only the top element is accessible; no direct index-based access in the abstract interface.
- All operations are atomic with respect to the LIFO guarantee.
- Error conditions (empty/full) must be checked before performing `pop`, `peek`, or `push` (for fixed-size).

**Constraints and Limitations**

- No random access to arbitrary elements.
- No efficient search for a specific value (O(n) traversal required, breaking LIFO abstraction).
- Fixed-size implementations cannot exceed their capacity without resizing or overflow.

### Annotated Code Examples

**Example 1: LIFO Behavior Demonstration (Python)**

```python
class ArrayStack:
    """Stack backed by a Python list; top is the end of the list."""
    def __init__(self) -> None:
        self._items: list = []

    def push(self, item) -> None:
        self._items.append(item)  # Appending places item at the end (top)

    def pop(self):
        if self.is_empty():
            raise IndexError("Pop from empty stack")
        return self._items.pop()  # Removes and returns the last element

    def peek(self):
        if self.is_empty():
            raise IndexError("Peek from empty stack")
        return self._items[-1]  # Returns last element without removal

    def is_empty(self) -> bool:
        return len(self._items) == 0

    def __len__(self) -> int:
        return len(self._items)


# Demonstration
stack = ArrayStack()
for value in [10, 20, 30]:
    stack.push(value)

print(f"Stack size: {len(stack)}")
print(f"Top element (peek): {stack.peek()}")  # Should be 30 (last pushed)

print(f"Popped: {stack.pop()}")  # 30
print(f"Popped: {stack.pop()}")  # 20
print(f"Stack size after pops: {len(stack)}")
print(f"Top element now: {stack.peek()}")  # 10

# Attempting to pop from an empty stack (after one more pop)
stack.pop()
try:
    stack.pop()
except IndexError as e:
    print(f"Error caught: {e}")
```

**Expected Output:**
```
Stack size: 3
Top element (peek): 30
Popped: 30
Popped: 20
Stack size after pops: 1
Top element now: 10
Error caught: Pop from empty stack
```

**Why This Output Occurs**: The LIFO principle dictates that `30` (last pushed) is the first `pop`ped. The `peek` operation confirms the top without removal. After removing `30` and `20`, the only remaining element is `10`, which becomes the new top. The final `pop` empties the stack, and a subsequent `pop` raises an error rather than silently returning `None`.

### Real-World Cases

- **Undo Functionality**: Each user action is pushed onto a stack; `Ctrl+Z` pops the most recent action to reverse.
- **Browser Back Button**: Visited pages are pushed; clicking Back pops the current page to reveal the previous one.
- **Expression Evaluation**: Compilers and calculators use stacks to evaluate postfix expressions and check balanced parentheses.
- **Depth-First Search**: DFS algorithms use an implicit or explicit stack to track the traversal path.

### References

- GeeksforGeeks - Stack Data Structure - https://www.geeksforgeeks.org/stack-data-structure/
- Cambridge University Press - Data Structures and Algorithms Using Python - https://www.cambridge.org/highereducation/books/data-structures-and-algorithms-using-python/86C296AE2484AF8B34B0C35F909B1077/stack/EB501FE2B35A2B31705DD7F75E44C843


## 2. Core Operations: push(), pop(), peek()/top(), isEmpty(), and isFull()

### Definitions

**Core Definition**
Core stack operations are the fundamental methods that define the stack ADT: adding elements (`push`), removing elements (`pop`), inspecting the top (`peek`), and checking state (`isEmpty`, `isFull`).

**Technical Definition**
These operations constitute the complete functional interface of a stack, with `push` and `pop` as the primary mutators, `peek` as an inspector, and `isEmpty`/`isFull` as state predicates.

**Beginner-Friendly Explanation**
These are the five buttons on a stack's control panel: put something on (push), take something off (pop), look at the top without taking it (peek), and check if it's empty or full.

### Purposes (All begin with "To")

- **To** provide a minimal, complete interface for LIFO data manipulation.
- **To** enable safe operations by checking preconditions (non-empty for pop/peek, non-full for push in fixed-size stacks).
- **To** support algorithmic patterns that require last-in-first-out access.
- **To** maintain encapsulation by exposing only necessary state information.

### Syntax Rules and Structure

**General Operation Syntax (Python)**

```python
class Stack:
    def push(self, item) -> None:
        """Add item to top. Raises OverflowError if full (fixed-size)."""
        ...

    def pop(self):
        """Remove and return top. Raises IndexError if empty."""
        ...

    def peek(self):
        """Return top without removal. Raises IndexError if empty."""
        ...

    def is_empty(self) -> bool:
        """Return True if stack has no elements."""
        ...

    def is_full(self) -> bool:
        """Return True if stack has reached capacity (fixed-size only)."""
        ...
```

**Component Breakdown**

| Operation | Precondition | Postcondition | Complexity |
|-----------|--------------|---------------|------------|
| `push(item)` | Not full (fixed-size) | `item` is at top; size + 1 | O(1) amortized |
| `pop()` | Not empty | Top removed; size - 1 | O(1) |
| `peek()` | Not empty | Top returned; stack unchanged | O(1) |
| `is_empty()` | None | `True` if size == 0 | O(1) |
| `is_full()` | None | `True` if size == capacity | O(1) |

**Syntax Rules**

- `pop` and `peek` must check `is_empty()` first.
- `push` on a fixed-size stack must check `is_full()` first.
- `isFull` is not applicable to dynamically resizing stacks (unless a capacity limit is imposed).

**Constraints and Limitations**

- `isFull` only makes sense for fixed-capacity implementations; dynamic stacks do not have a fixed maximum.
- `peek` on an empty stack is an error; some implementations return `None` instead, but this conflates "empty" with "contains `None`."

### Annotated Code Examples

**Example 1: Complete Stack Operations with Error Handling (Python)**

```python
from typing import Any, Optional

class FixedStack:
    """Fixed-capacity stack using a Python list with explicit capacity."""
    def __init__(self, capacity: int = 10) -> None:
        if capacity <= 0:
            raise ValueError("Capacity must be positive")
        self._capacity: int = capacity
        self._items: list = []
        self._top: int = -1  # -1 indicates empty

    def push(self, item: Any) -> None:
        if self.is_full():
            raise OverflowError("Stack overflow: cannot push to full stack")
        self._items.append(item)
        self._top += 1

    def pop(self) -> Any:
        if self.is_empty():
            raise IndexError("Stack underflow: cannot pop from empty stack")
        self._top -= 1
        return self._items.pop()

    def peek(self) -> Any:
        if self.is_empty():
            raise IndexError("Cannot peek at empty stack")
        return self._items[-1]

    def is_empty(self) -> bool:
        return self._top == -1

    def is_full(self) -> bool:
        return self._top == self._capacity - 1

    def __len__(self) -> int:
        return len(self._items)


# Demonstration
stack = FixedStack(capacity=3)

# Push elements
stack.push("A")
stack.push("B")
stack.push("C")
print(f"Stack full? {stack.is_full()}")  # True

# Attempt overflow
try:
    stack.push("D")
except OverflowError as e:
    print(f"Overflow caught: {e}")

# Peek and pop
print(f"Top element: {stack.peek()}")  # C
print(f"Popped: {stack.pop()}")  # C
print(f"Stack full now? {stack.is_full()}")  # False

# Empty the stack and attempt underflow
stack.pop()
stack.pop()
print(f"Stack empty? {stack.is_empty()}")  # True

try:
    stack.pop()
except IndexError as e:
    print(f"Underflow caught: {e}")
```

**Expected Output:**
```
Stack full? True
Overflow caught: Stack overflow: cannot push to full stack
Top element: C
Popped: C
Stack full now? False
Stack empty? True
Underflow caught: Stack underflow: cannot pop from empty stack
```

**Why This Output Occurs**: The stack has capacity 3. After pushing A, B, C, `_top` is 2 (index of C), so `is_full()` returns `True`. Pushing D raises `OverflowError`. After popping C, `_top` is 1, so `is_full()` is `False`. Popping A and B empties the stack (`_top` becomes -1), and further `pop` raises `IndexError`.

### Real-World Cases

- **Text Editor Undo**: Each edit is `push`ed; `pop` reverses the most recent change; `peek` might preview the action to be undone.
- **Call Stack Management**: Function calls are `push`ed with return addresses; `pop` restores execution context.
- **Validating Nested Structures**: `push` opening brackets; `pop` and verify matching closing brackets; `isEmpty` confirms all brackets are matched.

### References

- Cambridge University Press - Operations Associated with Stacks - https://www.cambridge.org/highereducation/books/data-structures-and-algorithms-using-python/86C296AE2484AF8B34B0C35F909B1077/stack/EB501FE2B35A2B31705DD7F75E44C843
- Save My Exams - Stack Operations Overview - https://www.savemyexams.com/a-level/computer-science/cie/19/revision-notes/10-data-types-and-structures-/introduction-to-abstract-data-types-adt/adt-overview/


## 3. Array-Based Implementation

### Definitions

**Core Definition**
An array-based stack uses a contiguous block of memory (typically a Python list) to store elements, with a `top` index tracking the most recently added element.

**Technical Definition**
The array-backed stack maps stack positions directly to array indices, where `top` serves as both the logical stack pointer and the array index of the current top element.

**Beginner-Friendly Explanation**
Imagine a row of numbered boxes. You put items into box 0 first, then box 1, then box 2. The "top" is whichever box was filled last. To remove an item, you take it from that last filled box.

### Purposes (All begin with "To")

- **To** leverage the O(1) random access and cache-friendly contiguous memory of arrays.
- **To** provide the simplest and most performant stack implementation in most languages.
- **To** enable amortized O(1) `push` through dynamic array growth (Python list).
- **To** support fixed-capacity stacks when memory must be bounded (embedded systems).

### Syntax Rules and Structure

**General Syntax: Fixed-Size Array Stack (Python)**

```python
class ArrayStack:
    def __init__(self, capacity: int = 10) -> None:
        self._capacity: int = capacity
        self._arr: list = [None] * capacity  # Pre-allocated array
        self._top: int = -1                   # Empty indicator

    def push(self, item) -> None:
        if self.is_full():
            raise OverflowError("Stack is full")
        self._top += 1
        self._arr[self._top] = item

    def pop(self):
        if self.is_empty():
            raise IndexError("Stack is empty")
        item = self._arr[self._top]
        self._top -= 1
        return item
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `_arr` | The underlying list/array storing elements |
| `_top` | Integer index of the top element; `-1` means empty |
| `_capacity` | Maximum number of elements (fixed-size) |
| `is_full()` | Returns `_top == _capacity - 1` |
| `is_empty()` | Returns `_top == -1` |

**Syntax Rules**

- `_top` starts at `-1` to indicate an empty stack.
- `push`: increment `_top`, then assign to `_arr[_top]`.
- `pop`: read `_arr[_top]`, then decrement `_top`.
- `peek`: return `_arr[_top]` without modifying `_top`.

**Constraints and Limitations**

- Fixed-size: `push` fails when `_top == capacity - 1`.
- Python's built-in `list` is dynamic and automatically resizes, making explicit capacity management optional in practice.
- Pre-allocating `[None] * capacity` wastes memory if the stack remains small.

### Annotated Code Examples

**Example 1: Fixed-Size Array Stack with Overflow/Underflow**

```python
class ArrayStack:
    """Fixed-capacity stack using a pre-allocated array."""
    def __init__(self, capacity: int = 5) -> None:
        self._capacity: int = capacity
        self._arr: list = [None] * capacity
        self._top: int = -1

    def push(self, item) -> None:
        if self.is_full():
            raise OverflowError(f"Stack overflow: capacity {self._capacity} reached")
        self._top += 1
        self._arr[self._top] = item

    def pop(self):
        if self.is_empty():
            raise IndexError("Stack underflow: cannot pop from empty stack")
        item = self._arr[self._top]
        self._top -= 1  # Note: item remains in array but is logically removed
        return item

    def peek(self):
        if self.is_empty():
            raise IndexError("Cannot peek at empty stack")
        return self._arr[self._top]

    def is_empty(self) -> bool:
        return self._top == -1

    def is_full(self) -> bool:
        return self._top == self._capacity - 1

    def __len__(self) -> int:
        return self._top + 1


# Demonstration
stack = ArrayStack(capacity=3)

print("Pushing 10, 20, 30...")
stack.push(10)
stack.push(20)
stack.push(30)
print(f"Stack size: {len(stack)}, Top: {stack.peek()}")

print(f"\nStack full? {stack.is_full()}")
try:
    stack.push(40)
except OverflowError as e:
    print(f"Error: {e}")

print(f"\nPopping all elements:")
while not stack.is_empty():
    print(f"  Popped: {stack.pop()}")
print(f"Stack empty? {stack.is_empty()}")

try:
    stack.pop()
except IndexError as e:
    print(f"Error: {e}")
```

**Expected Output:**
```
Pushing 10, 20, 30...
Stack size: 3, Top: 30

Stack full? True
Error: Stack overflow: capacity 3 reached

Popping all elements:
  Popped: 30
  Popped: 20
  Popped: 10
Stack empty? True
Error: Stack underflow: cannot pop from empty stack
```

**Why This Output Occurs**: The stack has capacity 3. After three pushes, `_top` equals 2 (`capacity - 1`), so `is_full()` returns `True` and the fourth push raises `OverflowError`. Popping decrements `_top` but leaves values in the array (logically removed). When `_top` reaches -1, `is_empty()` returns `True` and further `pop` raises `IndexError`.

**Example 2: Dynamic Array Stack (Python List)**

```python
class DynamicArrayStack:
    """Stack backed by Python's dynamically resizing list."""
    def __init__(self) -> None:
        self._items: list = []

    def push(self, item) -> None:
        self._items.append(item)  # Amortized O(1); occasionally O(n) resize

    def pop(self):
        if self.is_empty():
            raise IndexError("Pop from empty stack")
        return self._items.pop()  # O(1) from end

    def peek(self):
        if self.is_empty():
            raise IndexError("Peek from empty stack")
        return self._items[-1]

    def is_empty(self) -> bool:
        return len(self._items) == 0

    def __len__(self) -> int:
        return len(self._items)


# Timing demonstration (conceptual — actual timing varies)
import sys

stack = DynamicArrayStack()

print("Pushing 1,000,000 items...")
for i in range(1_000_000):
    stack.push(i)

print(f"Stack size: {len(stack)}")
print(f"List capacity (implementation detail): {len(stack) if hasattr(stack, '_items') else 'N/A'}")

print(f"Popping all items...")
popped_count = 0
while not stack.is_empty():
    stack.pop()
    popped_count += 1

print(f"Popped {popped_count} items")
print(f"Stack empty: {stack.is_empty()}")
```

**Expected Output:**
```
Pushing 1,000,000 items...
Stack size: 1000000
List capacity (implementation detail): 1000000
Popping all items...
Popped 1000000 items
Stack empty: True
```

**Why This Output Occurs**: Python's list uses a dynamic array internally with amortized O(1) `append` and O(1) `pop` from the end. The occasional resize (doubling capacity) is amortized across many operations, resulting in efficient overall performance. The `pop` from the end is O(1) because it only decrements the list's internal length.

### Real-World Cases

- **Interview Coding Problems**: The array-backed stack is the standard implementation when asked to implement a stack from scratch.
- **Expression Evaluators**: Parsing infix to postfix uses an operator stack backed by arrays.
- **Undo Systems with Bounded History**: Fixed-size array stacks limit undo depth to a known maximum.

### References

- Educative - Implementing a Stack with Arrays - https://www.educative.io/courses/learn-data-structures-and-algorithms/implementing-a-stack
- Python.org Mailing List - Python list append/pop complexity - https://mail.python.org/pipermail/python-list/2002-June/165207.html


## 4. Linked List-Based Implementation

### Definitions

**Core Definition**
A linked-list stack uses a chain of nodes, where each node holds a value and a reference to the next node, with the head of the list serving as the stack's top.

**Technical Definition**
The linked-list stack prepends new nodes at the head (top) for `push` and removes the head node for `pop`, achieving strict O(1) operations without resizing overhead.

**Beginner-Friendly Explanation**
Think of a stack of papers where each paper has a note saying "the paper below me is...". To add a paper, you put it on top and note the previous top. To remove a paper, you take the top and follow its note to find the new top.

### Purposes (All begin with "To")

- **To** achieve strict O(1) push and pop without amortized resize costs.
- **To** avoid the memory overhead of a pre-allocated array with unused capacity.
- **To** provide a natural implementation when nodes are already available (e.g., linked list problems).
- **To** support unbounded stack growth subject only to available heap memory.

### Syntax Rules and Structure

**General Syntax (Python)**

```python
class StackNode:
    def __init__(self, value, next_node=None):
        self.value = value
        self.next = next_node  # Reference toward bottom of stack

class LinkedStack:
    def __init__(self):
        self._top: StackNode | None = None
        self._size: int = 0

    def push(self, item) -> None:
        self._top = StackNode(item, self._top)  # New node points to old top
        self._size += 1

    def pop(self):
        if self.is_empty():
            raise IndexError("Pop from empty stack")
        value = self._top.value
        self._top = self._top.next  # Move top to next node
        self._size -= 1
        return value
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `StackNode` | Holds `value` and `next` reference to the node below |
| `_top` | Reference to the head node (most recent push) |
| `_size` | Tracks number of elements for O(1) `len()` |
| `push` | Creates new node pointing to current `_top`; updates `_top` |
| `pop` | Saves `_top.value`, advances `_top`, returns saved value |

**Syntax Rules**

- `_top` is `None` when the stack is empty.
- `push`: always creates a new node, even if a previous node was popped (no reuse without explicit pooling).
- `pop`: must check `is_empty()` before accessing `_top.value`.
- No `isFull()` exists inherently; the stack grows until memory is exhausted.

**Constraints and Limitations**

- Higher memory overhead per element (each node stores a value plus a reference).
- Poor cache locality compared to contiguous arrays.
- Python's object creation overhead makes linked stacks slower than array stacks in practice for most workloads.

### Annotated Code Examples

**Example 1: Linked-List Stack Implementation**

```python
from typing import Any, Optional

class StackNode:
    """A node in the linked stack; 'next' points toward the bottom."""
    def __init__(self, value: Any, next_node: Optional["StackNode"] = None) -> None:
        self.value = value
        self.next = next_node

class LinkedStack:
    """Stack backed by a singly linked list. Strict O(1) push/pop."""
    def __init__(self) -> None:
        self._top: Optional[StackNode] = None
        self._size: int = 0

    def push(self, item: Any) -> None:
        """Create new node; its next points to the previous top."""
        self._top = StackNode(item, self._top)
        self._size += 1

    def pop(self) -> Any:
        """Remove head node; new top is the previous second node."""
        if self.is_empty():
            raise IndexError("Pop from empty stack")
        value = self._top.value
        self._top = self._top.next
        self._size -= 1
        return value

    def peek(self) -> Any:
        if self.is_empty():
            raise IndexError("Peek from empty stack")
        return self._top.value

    def is_empty(self) -> bool:
        return self._top is None

    def __len__(self) -> int:
        return self._size


# Demonstration
stack = LinkedStack()

# Push elements
for ch in "STACK":
    stack.push(ch)

print(f"Stack size: {len(stack)}")
print(f"Top element: {stack.peek()}")  # K (last pushed)

# Pop in LIFO order
print("Popping all elements:")
while not stack.is_empty():
    print(f"  {stack.pop()}")

print(f"Stack empty: {stack.is_empty()}")
```

**Expected Output:**
```
Stack size: 5
Top element: K
Popping all elements:
  K
  C
  A
  T
  S
Stack empty: True
```

**Why This Output Occurs**: The string "STACK" is pushed character by character. Each new node becomes the head (`_top`). The last pushed character, 'K', is at the top. Popping follows LIFO: K, C, A, T, S. After all pops, `_top` is `None`, so `is_empty()` returns `True`.

### Real-World Cases

- **Undo Systems with Unbounded History**: Linked stacks grow as needed without pre-allocating capacity.
- **Algorithm Interviews**: Linked-list stacks demonstrate pointer manipulation and node management.
- **Memory-Constrained Environments**: When array resizing is undesirable, linked nodes allow fine-grained allocation.

### References

- GitHub - algo-stacks-queues (Pavel-Kravchenko) - https://raw.githubusercontent.com/Pavel-Kravchenko/Bioinformatics/refs/heads/main/Skills/algo-stacks-queues/SKILL.md
- Python.org Mailing List - Linked List in Python - https://mail.python.org/pipermail/python-list/2007-January/462978.html


## 5. Time and Space Complexity Profiles

### Definitions

**Core Definition**
Time complexity describes how operation time scales with stack size; space complexity describes memory overhead per element and overall.

**Technical Definition**
Array-based stacks achieve O(1) amortized push and strict O(1) pop/peek. Linked-list stacks achieve strict O(1) push/pop/peek. Space overhead differs: arrays may waste capacity; linked nodes add reference overhead.

**Beginner-Friendly Explanation**
Both implementations are fast for core operations (constant time regardless of stack size). The difference is in memory usage: arrays pre-allocate space that might go unused, while linked lists use exactly the memory needed but with extra bookkeeping.

### Complexity Comparison Table

| Operation | Array Stack (Dynamic) | Linked Stack | Notes |
|-----------|----------------------|--------------|-------|
| **push** | O(1) amortized | O(1) | Array resizes occasionally; linked always O(1) |
| **pop** | O(1) | O(1) | Both remove from one end |
| **peek** | O(1) | O(1) | Direct access to top |
| **is_empty** | O(1) | O(1) | Length check or `_top` comparison |
| **is_full** | O(1) (if fixed) | N/A | Linked stacks have no fixed capacity |
| **Space (per element)** | 1 reference (list slot) | Value + reference (node) | Linked uses ~2× memory in Python |
| **Space (overall)** | Up to 2× capacity | Exactly n nodes | Array may overallocate |

### Annotated Code Examples

**Example 1: Timing Comparison (Conceptual)**

```python
import time

# Array-backed stack (Python list)
class ArrayStack:
    def __init__(self):
        self._items = []
    def push(self, x):
        self._items.append(x)
    def pop(self):
        return self._items.pop()
    def is_empty(self):
        return len(self._items) == 0

# Linked stack (simplified node)
class Node:
    __slots__ = ('value', 'next')  # Memory optimization
    def __init__(self, value, next_node=None):
        self.value = value
        self.next = next_node

class LinkedStack:
    def __init__(self):
        self._top = None
    def push(self, x):
        self._top = Node(x, self._top)
    def pop(self):
        if self._top is None:
            raise IndexError
        val = self._top.value
        self._top = self._top.next
        return val
    def is_empty(self):
        return self._top is None

# Benchmark (note: actual timing varies by machine and Python version)
N = 500_000

# Array stack
arr_stack = ArrayStack()
start = time.perf_counter()
for i in range(N):
    arr_stack.push(i)
for i in range(N):
    arr_stack.pop()
arr_time = time.perf_counter() - start

# Linked stack
link_stack = LinkedStack()
start = time.perf_counter()
for i in range(N):
    link_stack.push(i)
for i in range(N):
    link_stack.pop()
link_time = time.perf_counter() - start

print(f"Array stack: {arr_time:.4f} seconds for {N} push+pop pairs")
print(f"Linked stack: {link_time:.4f} seconds for {N} push+pop pairs")
print(f"Array is {link_time/arr_time:.1f}x faster in this benchmark")
```

**Expected Output** (timing varies; ratio is illustrative):
```
Array stack: 0.0842 seconds for 500000 push+pop pairs
Linked stack: 0.3215 seconds for 500000 push+pop pairs
Array is 3.8x faster in this benchmark
```

**Why This Output Occurs**: Python's `list` is implemented in C with contiguous memory and efficient resizing, while the linked stack creates a new Python object for every node. The array stack benefits from cache locality and lower per-element overhead. This benchmark illustrates why Python developers typically prefer `list` for stack implementations despite the theoretical O(1) advantage of linked nodes.

### Real-World Cases

- **Python `list` as Stack**: The standard library's `list` is the idiomatic stack in Python due to its performance and simplicity.
- **Embedded Systems**: Fixed array stacks provide predictable memory bounds; linked stacks are avoided due to allocation overhead.
- **Language Runtimes**: The call stack uses a contiguous array-like structure for speed and cache efficiency.

### References

- University of Northern Iowa - Stack Implementation Big-O Analysis - http://www.cs.uni.edu//~fienup/cs052f10/labs/lab4.pdf
- Python.org Mailing List - List Performance Characteristics - https://mail.python.org/pipermail/python-list/2002-June/165207.html


## 6. System Stack vs. User Stack

### Definitions

**Core Definition**
The system stack (call stack) is a runtime structure managed by the language/OS for function call management. A user stack is any stack data structure created explicitly by the programmer for algorithmic purposes.

**Technical Definition**
The system stack stores activation records (stack frames) containing return addresses, saved registers, local variables, and arguments. Stack overflow occurs when this fixed-size region is exhausted. User stacks are heap-allocated and subject to available memory.

**Beginner-Friendly Explanation**
The system stack is like the computer's to-do list for function calls—each time you call a function, a note is added, and when it returns, the note is removed. A user stack is a tool you build yourself to solve problems, like keeping track of your path in a maze.

### Purposes (All begin with "To")

- **To** manage function execution, return addresses, and local variable scope automatically.
- **To** enable recursion by preserving each call's state on the system stack.
- **To** allow programmers to implement LIFO algorithms explicitly when the call stack is insufficient or inappropriate.
- **To** distinguish between runtime-managed (system) and program-managed (user) LIFO structures.

### Syntax Rules and Structure

**System Stack (Implicit)**

```python
def recursive_factorial(n: int) -> int:
    """Each call pushes an activation record onto the system stack."""
    if n <= 1:
        return 1
    return n * recursive_factorial(n - 1)

# Python's default recursion limit (system stack depth)
import sys
print(sys.getrecursionlimit())  # Typically 1000
```

**User Stack (Explicit)**

```python
class UserStack:
    """Programmer-managed LIFO structure on the heap."""
    def __init__(self):
        self._items = []
    def push(self, item):
        self._items.append(item)
    def pop(self):
        return self._items.pop()
    def is_empty(self):
        return len(self._items) == 0
```

**Component Breakdown**

| Aspect | System Stack | User Stack |
|--------|--------------|------------|
| Management | Language runtime / OS | Programmer |
| Location | Fixed-size memory region | Heap (dynamic) |
| Overflow | `RecursionError` / `StackOverflow` | `MemoryError` / `OverflowError` |
| Visibility | Implicit (call frames) | Explicit (variables) |
| Use Case | Function calls, recursion | Algorithms, backtracking |

**Syntax Rules**

- The system stack is not directly accessible in Python; it is managed by the interpreter.
- Python's recursion limit can be adjusted with `sys.setrecursionlimit()`, but this risks actual stack overflow at the C level.
- User stacks are ordinary objects with explicit method calls.

**Constraints and Limitations**

- System stack size is fixed (often 1–8 MB); deep recursion causes `RecursionError`.
- User stacks are limited only by heap memory.
- Python's system stack is less directly controllable than in C/C++.

### Annotated Code Examples

**Example 1: Demonstrating System Stack Overflow**

```python
import sys

def infinite_recursion(n=0):
    """Recursive function without base case; exhausts system stack."""
    return infinite_recursion(n + 1)

# Check default limit
print(f"Default recursion limit: {sys.getrecursionlimit()}")

try:
    infinite_recursion()
except RecursionError as e:
    print(f"RecursionError caught: {e}")
    print("This is the system stack (call stack) overflowing.")
```

**Expected Output:**
```
Default recursion limit: 1000
RecursionError caught: maximum recursion depth exceeded
This is the system stack (call stack) overflowing.
```

**Why This Output Occurs**: Each recursive call pushes an activation record onto the system stack. Python's default limit of 1000 prevents the C-level stack from overflowing (which would crash the interpreter). When the limit is exceeded, `RecursionError` is raised.

**Example 2: User Stack Replacing Recursion**

```python
def recursive_reverse(s: str) -> str:
    """Recursive string reversal using the system stack."""
    if len(s) <= 1:
        return s
    return recursive_reverse(s[1:]) + s[0]

def iterative_reverse(s: str) -> str:
    """Iterative string reversal using an explicit user stack."""
    stack = []  # User stack (Python list as stack)
    for ch in s:
        stack.append(ch)
    result = []
    while stack:
        result.append(stack.pop())
    return ''.join(result)

# Both produce the same result
text = "HELLO"
print(f"Recursive reverse: {recursive_reverse(text)}")
print(f"Iterative reverse: {iterative_reverse(text)}")

# The iterative version avoids system stack depth proportional to len(s)
# For very long strings, recursive_reverse would hit RecursionError
long_text = "A" * 5000
try:
    recursive_reverse(long_text)
except RecursionError:
    print(f"Recursive reverse failed for length {len(long_text)}")
    print(f"Iterative reverse works: {len(iterative_reverse(long_text))} chars")
```

**Expected Output:**
```
Recursive reverse: OLLEH
Iterative reverse: OLLEH
Recursive reverse failed for length 5000
Iterative reverse works: 5000 chars
```

**Why This Output Occurs**: `recursive_reverse` uses the system stack, which is limited to ~1000 frames by default. For a 5000-character string, the recursion depth exceeds this limit. `iterative_reverse` uses a user stack (Python list), which is limited only by heap memory, so it handles the long string without error.

### Real-World Cases

- **Recursion vs. Iteration**: Converting recursive algorithms to iterative ones with explicit stacks avoids system stack limits.
- **Debugging Stack Traces**: Understanding activation records helps interpret tracebacks and debug recursive functions.
- **Embedded Systems**: System stack size is a critical design constraint; avoiding deep recursion is mandatory.

### References

- CMU 15-213 - Stack Frame Layout and Function Calls - http://www.cs.cmu.edu/afs/cs/academic/class/15213-s01/lectures/class07.pdf
- Python Documentation - sys.setrecursionlimit - https://docs.python.org/3/search.html?q=sys.setrecursionlimit


## Consolidated References

- Cambridge University Press - Data Structures and Algorithms Using Python (Stack Chapter) - https://www.cambridge.org/highereducation/books/data-structures-and-algorithms-using-python/86C296AE2484AF8B34B0C35F909B1077/stack/EB501FE2B35A2B31705DD7F75E44C843
- Save My Exams - ADT Overview: Stack Operations - https://www.savemyexams.com/a-level/computer-science/cie/19/revision-notes/10-data-types-and-structures-/introduction-to-abstract-data-types-adt/adt-overview/
- Educative - Implementing a Stack with Arrays and Linked Lists - https://www.educative.io/courses/learn-data-structures-and-algorithms/implementing-a-stack
- GitHub (Pavel-Kravchenko) - algo-stacks-queues Skill - https://raw.githubusercontent.com/Pavel-Kravchenko/Bioinformatics/refs/heads/main/Skills/algo-stacks-queues/SKILL.md
- CMU 15-213 - Stack Frame and Function Call Management - http://www.cs.cmu.edu/afs/cs/academic/class/15213-s01/lectures/class07.pdf
- University of Northern Iowa - Stack Implementation Lab (Big-O Analysis) - http://www.cs.uni.edu//~fienup/cs052f10/labs/lab4.pdf
- Python.org Mailing List - Python List Performance (Append/Pop) - https://mail.python.org/pipermail/python-list/2002-June/165207.html
- Python.org Mailing List - Linked List Implementation Discussion - https://mail.python.org/pipermail/python-list/2007-January/462978.html
- Python Documentation - Data Structures Tutorial - https://docs.python.org/3/tutorial/datastructures.html
- Python Documentation - sys.setrecursionlimit - https://docs.python.org/3/search.html?q=sys.setrecursionlimit