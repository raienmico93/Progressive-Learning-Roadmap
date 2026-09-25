# Advanced Queue Variants & Hybrid Structures: Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**
Advanced queue variants extend the basic FIFO abstraction with additional access points, priority-based ordering, or hybrid behaviors that combine multiple data structure properties to solve specialized algorithmic problems.

**Technical Definition**
These structures—deques, priority queues, monotonic queues, and queue-backed stacks—modify the standard queue's constraints on insertion and deletion locations or element ordering to achieve different time complexity and access pattern trade-offs.

**Beginner-Friendly Explanation**
Think of a regular queue as a single-file line. Advanced variants are like special lines: some let people enter or leave from both ends (deque), some let the most important person go first regardless of arrival time (priority queue), and some maintain a "always sorted" property for efficient window queries (monotonic queue).

### Key Characteristics

- **Flexible Access Points**: Deques allow insertion and deletion at both ends, breaking the single-entry/single-exit constraint of standard queues .
- **Priority-Based Ordering**: Priority queues replace FIFO ordering with priority weights, ensuring the highest-priority element is always accessible .
- **Amortized Efficiency**: Monotonic queues maintain sorted order with O(1) amortized operations, enabling O(n) sliding window algorithms .
- **Structural Adaptation**: Complex structures (like stacks) can be simulated using simpler primitives (queues), demonstrating the expressive power of basic ADTs .

### Prerequisites

- Mastery of basic stack and queue implementations (array-based and linked-list-based)
- Understanding of Big-O notation and amortized analysis
- Familiarity with doubly linked lists and circular array mechanics
- Basic knowledge of binary tree properties (for priority queue heaps)

### Related Programming Areas

- **Algorithm Design**: Sliding window problems, graph algorithms (Dijkstra, A*)
- **Operating Systems**: Priority-based process scheduling, I/O buffering
- **Real-Time Systems**: Deadline-driven task management
- **Competitive Programming**: Efficient window queries, monotonic stack/queue patterns

### Core Concepts

| Concept | Description |
|---------|-------------|
| Double-Ended Queue (Deque) | Queue with O(1) insertion/deletion at both front and rear |
| Input/Output-Restricted Deques | Deques with constrained operations at one end |
| Priority Queue | Elements ordered by priority weight rather than arrival time |
| Binary Heap | Array-based tree structure providing O(log n) priority queue operations |
| Monotonic Queue | Deque maintaining elements in sorted order for window queries |
| Stack via Queues | Simulating LIFO behavior using one or two FIFO queues |


## 1. Double-Ended Queue (Deque)

### Definitions

**Core Definition**
A double-ended queue (deque, pronounced "deck") is a linear data structure that allows insertion and deletion at both the front and the rear in O(1) time .

**Technical Definition**
A deque is a generalization of both stack and queue, providing `push_front`, `push_back`, `pop_front`, and `pop_back` operations, all achievable in constant time with appropriate underlying implementations (doubly linked list or circular array) .

**Beginner-Friendly Explanation**
Imagine a line where people can enter or leave from either end. You can add someone to the front or back, and you can remove someone from the front or back. This flexibility makes it useful for many problems .

### Purposes (All begin with "To")

- **To** provide O(1) insertion and deletion at both ends of a sequence
- **To** enable efficient implementation of sliding window algorithms
- **To** support both stack-like (LIFO) and queue-like (FIFO) access patterns within a single structure
- **To** serve as the underlying primitive for more specialized structures like monotonic queues

### Syntax Rules and Structure

**General Syntax (Python `collections.deque`)**

```python
from collections import deque

d = deque()                    # Create empty deque
d.append(x)                    # Add to right (rear)
d.appendleft(x)                # Add to left (front)
d.pop()                        # Remove and return from right
d.popleft()                    # Remove and return from left
d[0]                           # Peek at front
d[-1]                          # Peek at rear
```

**Component Breakdown**

| Operation | Description | Complexity |
|-----------|-------------|------------|
| `append(x)` | Add to rear | O(1) |
| `appendleft(x)` | Add to front | O(1) |
| `pop()` | Remove from rear | O(1) |
| `popleft()` | Remove from front | O(1) |
| `[0]` | Access front | O(1) |
| `[-1]` | Access rear | O(1) |

**Syntax Rules**

- Both ends must support O(1) insertion and deletion for a true deque 
- Doubly linked list is the natural implementation: each node has `prev` and `next` pointers 
- Circular array implementation is also possible with careful index management 
- Python's `collections.deque` uses a doubly linked list of blocks for memory efficiency 

**Constraints and Limitations**

- Singly linked list cannot support O(1) `pop_back` because finding the second-to-last node requires O(n) traversal 
- Array-based deque requires capacity management for circular wrapping
- Python list cannot serve as an efficient deque due to O(n) `pop(0)` and `insert(0, x)` 

### Annotated Code Examples

**Example 1: Python `collections.deque` Operations**

```python
from collections import deque

# Create a deque with initial elements
d = deque(['g', 'h', 'i'])

# Add to both ends
d.append('j')        # Add to right (rear)
d.appendleft('f')    # Add to left (front)
print(f"After append/appendleft: {d}")  # deque(['f', 'g', 'h', 'i', 'j'])

# Remove from both ends
print(f"pop() from right: {d.pop()}")       # 'j'
print(f"popleft() from left: {d.popleft()}") # 'f'
print(f"After pops: {d}")                    # deque(['g', 'h', 'i'])

# Peek at both ends without removal
print(f"Front: {d[0]}")    # 'g'
print(f"Rear: {d[-1]}")    # 'i'

# Rotation operation (unique to deque)
d.rotate(1)  # Rotate right by 1
print(f"After rotate(1): {d}")   # deque(['i', 'g', 'h'])

d.rotate(-1)  # Rotate left by 1
print(f"After rotate(-1): {d}")  # deque(['g', 'h', 'i'])
```

**Expected Output:**
```
After append/appendleft: deque(['f', 'g', 'h', 'i', 'j'])
pop() from right: j
popleft() from left: f
After pops: deque(['g', 'h', 'i'])
Front: g
Rear: i
After rotate(1): deque(['i', 'g', 'h'])
After rotate(-1): deque(['g', 'h', 'i'])
```

**Why This Output Occurs**: The `append` and `appendleft` operations add to opposite ends in O(1). `pop` and `popleft` remove from opposite ends. The `rotate` method shifts all elements right or left by the specified amount, wrapping around .

**Example 2: Doubly Linked List Deque Implementation**

```python
class DequeNode:
    def __init__(self, value):
        self.value = value
        self.prev = None
        self.next = None

class DoublyLinkedDeque:
    def __init__(self):
        self._head = None  # Front
        self._tail = None  # Rear
        self._size = 0

    def push_front(self, value):
        new_node = DequeNode(value)
        if self._head is None:
            self._head = self._tail = new_node
        else:
            new_node.next = self._head
            self._head.prev = new_node
            self._head = new_node
        self._size += 1

    def push_back(self, value):
        new_node = DequeNode(value)
        if self._tail is None:
            self._head = self._tail = new_node
        else:
            new_node.prev = self._tail
            self._tail.next = new_node
            self._tail = new_node
        self._size += 1

    def pop_front(self):
        if self._head is None:
            raise IndexError("Deque is empty")
        value = self._head.value
        self._head = self._head.next
        if self._head is None:
            self._tail = None
        else:
            self._head.prev = None
        self._size -= 1
        return value

    def pop_back(self):
        if self._tail is None:
            raise IndexError("Deque is empty")
        value = self._tail.value
        self._tail = self._tail.prev
        if self._tail is None:
            self._head = None
        else:
            self._tail.next = None
        self._size -= 1
        return value

    def front(self):
        if self._head is None:
            raise IndexError("Deque is empty")
        return self._head.value

    def back(self):
        if self._tail is None:
            raise IndexError("Deque is empty")
        return self._tail.value


# Demonstration
dq = DoublyLinkedDeque()
dq.push_front(3)
dq.push_back(4)
dq.push_front(2)
dq.push_back(5)

print(f"Front: {dq.front()}, Back: {dq.back()}")  # 2, 5
print(f"pop_front: {dq.pop_front()}")  # 2
print(f"pop_back: {dq.pop_back()}")    # 5
print(f"Now front: {dq.front()}, back: {dq.back()}")  # 3, 4
```

**Expected Output:**
```
Front: 2, Back: 5
pop_front: 2
pop_back: 5
Now front: 3, back: 4
```

**Why This Output Occurs**: The doubly linked list allows O(1) access to both ends through `_head` and `_tail` pointers. Each push/pop operation updates the appropriate pointers, maintaining constant time complexity .

### Real-World Cases

- **Browser History**: Back/forward navigation uses a deque to track visited pages, allowing O(1) addition to either end .
- **Undo/Redo Systems**: A deque with limited capacity can track recent actions, with new actions added to one end and oldest actions removed from the other .
- **Sliding Window Algorithms**: Maintaining a window of elements where both ends change frequently benefits from deque's O(1) operations at both ends .

### References

- University of Waterloo - Deques and Their Implementations - https://ece.uwaterloo.ca/~dwharder/aads/Lecture_materials/3.04.Deques.pdf
- USC CS 103 - Deque Implementation using Doubly-Linked Lists - https://bytes.usc.edu/files/cs103/fa25/slides/Unit4c_DoublyLinkedLists.pdf
- Python Documentation - collections.deque - https://docs.python.org/fr/3.5/library/collections.html#collections.deque


## 2. Input-Restricted and Output-Restricted Deques

### Definitions

**Core Definition**
Input-restricted deques allow insertion at only one end but deletion at both ends. Output-restricted deques allow deletion at only one end but insertion at both ends .

**Technical Definition**
An input-restricted deque restricts `push_front` or `push_back` to a single end while permitting `pop_front` and `pop_back`. An output-restricted deque restricts one of the pop operations while permitting both push operations .

**Beginner-Friendly Explanation**
Imagine a line where people can leave from either end, but new people can only join from the back (input-restricted). Or a line where people can join from either end, but can only leave from the front (output-restricted) .

### Purposes (All begin with "To")

- **To** model scenarios where only one direction of insertion or deletion is logically meaningful
- **To** reduce the state space in algorithms that require only constrained deque operations
- **To** provide a simpler interface for problems that do not need full deque flexibility
- **To** demonstrate the design space between stack, queue, and full deque abstractions

### Syntax Rules and Structure

**Input-Restricted Deque Operations**

| Operation | Allowed |
|-----------|---------|
| `push_back(x)` | Yes |
| `push_front(x)` | **No** (restricted) |
| `pop_front()` | Yes |
| `pop_back()` | Yes |

**Output-Restricted Deque Operations**

| Operation | Allowed |
|-----------|---------|
| `push_back(x)` | Yes |
| `push_front(x)` | Yes |
| `pop_front()` | Yes |
| `pop_back()` | **No** (restricted) |

**Syntax Rules**

- An input-restricted deque supports all stack operations (`push_back`, `pop_back`) and queue operations (`push_back`, `pop_front`) 
- The restriction is on insertion location, not deletion
- Output-restricted deque (steque) supports push at both ends but pop at only one end 

**Constraints and Limitations**

- Restricted deques cannot solve problems requiring the full range of deque operations
- The restriction must be enforced at the interface level; underlying implementation may support all operations

### Annotated Code Examples

**Example 1: Input-Restricted Deque Simulation**

```python
from collections import deque

class InputRestrictedDeque:
    """Deque allowing insertion only at back, deletion from both ends."""
    def __init__(self):
        self._data = deque()

    def push_back(self, value):
        """Only insertion method allowed."""
        self._data.append(value)

    def pop_front(self):
        if not self._data:
            raise IndexError("Empty")
        return self._data.popleft()

    def pop_back(self):
        if not self._data:
            raise IndexError("Empty")
        return self._data.pop()

    def front(self):
        if not self._data:
            raise IndexError("Empty")
        return self._data[0]

    def back(self):
        if not self._data:
            raise IndexError("Empty")
        return self._data[-1]


# Demonstration: This structure can act as either a stack or a queue
ird = InputRestrictedDeque()

# Stack behavior: push_back + pop_back
ird.push_back(1)
ird.push_back(2)
ird.push_back(3)
print(f"Stack: pop_back gives {ird.pop_back()}")  # 3 (LIFO)

# Queue behavior: push_back + pop_front
ird.push_back(4)
print(f"Queue: pop_front gives {ird.pop_front()}")  # 1 (FIFO)
print(f"Remaining: front={ird.front()}, back={ird.back()}")  # 2, 4
```

**Expected Output:**
```
Stack: pop_back gives 3
Queue: pop_front gives 1
Remaining: front=2, back=4
```

**Why This Output Occurs**: The input-restricted deque supports both LIFO (using `pop_back`) and FIFO (using `pop_front`) behaviors because it allows deletion from both ends while restricting insertion to one end .

### Real-World Cases

- **Undo with Bounded History**: Input-restricted deque can add new actions only at the back while allowing removal from either end for undo/redo .
- **Priority Buffers**: When new items always arrive at the rear but processing may occur from either end based on priority.

### References

- University of Illinois - Variants of Ended Lists - http://jeffe.cs.illinois.edu/teaching/225H/notes/02-05.pdf
- Data Structures in C++ (Textbook) - Deque Variations - 
- Data Structures and Algorithms Using C++ (Textbook) - Deque Types - 


## 3. Priority Queue Foundations

### Definitions

**Core Definition**
A priority queue is an abstract data type where each element has an associated priority, and elements are served in order of priority rather than arrival time .

**Technical Definition**
A priority queue maintains a collection of elements with comparable keys (priorities), supporting insertion and extraction of the minimum (or maximum) key element in efficient time, typically O(log n) with a binary heap .

**Beginner-Friendly Explanation**
Imagine a hospital emergency room. Patients are not treated in arrival order—the most critical patients are seen first, regardless of when they arrived. A priority queue is the data structure that models this behavior .

### Purposes (All begin with "To")

- **To** ensure the highest-priority element is always accessible in O(1) or O(log n) time
- **To** enable efficient algorithms like Dijkstra's shortest path and A* search
- **To** model scheduling systems where importance outweighs arrival order
- **To** support dynamic insertion and extraction based on comparable keys

### Syntax Rules and Structure

**General Syntax (Python `heapq`)**

```python
import heapq

pq = []                          # Empty heap (priority queue)
heapq.heappush(pq, (priority, item))  # Insert with priority
priority, item = heapq.heappop(pq)    # Extract minimum priority
pq[0]                            # Peek at minimum without removal
heapq.heapify(list)              # Convert list to heap in O(n)
```

**Component Breakdown**

| Operation | Description | Complexity |
|-----------|-------------|------------|
| `heappush(pq, item)` | Insert element | O(log n) |
| `heappop(pq)` | Remove and return minimum | O(log n) |
| `pq[0]` | Peek at minimum | O(1) |
| `heapify(list)` | Build heap from list | O(n) |

**Syntax Rules**

- Python's `heapq` implements a **min-heap**: smallest priority is extracted first 
- For max-priority, negate priorities or use a wrapper class 
- Elements must be comparable; for tuples, comparison proceeds lexicographically
- To ensure FIFO ordering for equal priorities, include an entry counter as tie-breaker 

**Constraints and Limitations**

- `heapq` does not support efficient decrease-key or arbitrary deletion 
- Removing arbitrary elements requires a separate dictionary mapping and a "removed" marker 
- Python's heap is a list, not a separate class; functions operate on lists

### Annotated Code Examples

**Example 1: Priority Queue with `heapq` and Tie-Breaking**

```python
import heapq
import itertools

# Using a counter as tie-breaker for stable priority ordering
counter = itertools.count()  # Unique sequence count

pq = []

# Add tasks with priorities (lower number = higher priority)
tasks = [
    (3, 'write code'),
    (1, 'write spec'),
    (2, 'create tests'),
    (1, 'review spec')  # Same priority as 'write spec'
]

for priority, task in tasks:
    # Entry: [priority, count, task]
    # Count ensures FIFO for equal priorities
    count = next(counter)
    entry = [priority, count, task]
    heapq.heappush(pq, entry)

# Extract in priority order
print("Tasks processed in priority order:")
while pq:
    priority, count, task = heapq.heappop(pq)
    print(f"  Priority {priority}: {task}")
```

**Expected Output:**
```
Tasks processed in priority order:
  Priority 1: write spec
  Priority 1: review spec
  Priority 2: create tests
  Priority 3: write code
```

**Why This Output Occurs**: The counter breaks ties for equal priorities. "write spec" was added before "review spec", both with priority 1, so it is extracted first. This ensures stable FIFO ordering within the same priority level .

**Example 2: Max-Heap Using Negated Priorities**

```python
import heapq

# Python's heapq is a min-heap; negate priorities for max-heap behavior
max_pq = []

# Insert with negated priorities
values = [(5, 'A'), (3, 'B'), (7, 'C'), (1, 'D')]
for priority, item in values:
    heapq.heappush(max_pq, (-priority, item))

# Extract in descending priority order
print("Max-priority extraction:")
while max_pq:
    neg_priority, item = heapq.heappop(max_pq)
    print(f"  Priority { -neg_priority }: {item}")
```

**Expected Output:**
```
Max-priority extraction:
  Priority 7: C
  Priority 5: A
  Priority 3: B
  Priority 1: D
```

**Why This Output Occurs**: Negating priorities converts the min-heap into a max-heap. The smallest negated value (corresponding to the largest original priority) is extracted first .

### Real-World Cases

- **Dijkstra's Algorithm**: Priority queue selects the unvisited node with the smallest tentative distance .
- **Task Schedulers**: Operating systems use priority queues to select which process to run next.
- **Event Simulation**: Discrete event simulations process events in time order using a priority queue.

### References

- CMU 15-210 - Priority Queues and Leftist Heaps - https://www.cs.cmu.edu/afs/cs/academic/class/15210-f12/www/lectures/lecture27.pdf
- Python Documentation - heapq: Heap Queue Algorithm - https://docs.python.org/3.14/library/heapq.html
- Stack Overflow - Binary Heap vs Linked List Priority Queue - https://stackoverflow.com/questions/64740904/when-to-use-a-binary-heap-based-priority-queue-vs-a-linked-list-based-priority-q


## 4. Array, Linked List, and Binary Heap Implementation Trade-offs

### Definitions

**Core Definition**
Priority queues can be implemented using unsorted arrays, sorted arrays, linked lists, or binary heaps, each offering different time complexity trade-offs for insertion and extraction .

**Technical Definition**
The choice of underlying structure determines whether insertion or extraction dominates the complexity: unsorted structures favor O(1) insert but O(n) extract; sorted structures reverse this; binary heaps balance both at O(log n) .

**Beginner-Friendly Explanation**
Think of organizing a to-do list: you can throw tasks in a pile (fast to add, slow to find the most important), keep the list sorted (slow to add, fast to find), or use a special tree structure (balanced speed for both) .

### Purposes (All begin with "To")

- **To** select the appropriate priority queue implementation based on operation frequency
- **To** understand the time-space trade-offs between different underlying structures
- **To** recognize when a binary heap is optimal versus when simpler structures suffice
- **To** appreciate the role of data structure choice in algorithm performance

### Complexity Comparison Table

| Implementation | Insert | Extract Min/Max | Peek | Space per Element |
|----------------|--------|-----------------|------|-------------------|
| **Unsorted Array** | O(1) | O(n) | O(n) | 1 slot |
| **Sorted Array** | O(n) | O(1) | O(1) | 1 slot |
| **Unsorted Linked List** | O(1) | O(n) | O(n) | Value + pointer |
| **Sorted Linked List** | O(n) | O(1) | O(1) | Value + pointer |
| **Binary Heap** | O(log n) | O(log n) | O(1) | 1 slot (array-based) |

### Annotated Code Examples

**Example 1: Binary Heap Priority Queue (Python)**

```python
import heapq

class PriorityQueue:
    """Min-priority queue backed by a binary heap."""
    def __init__(self):
        self._heap = []
        self._entry_finder = {}  # For tracking tasks
        self._counter = 0

    def push(self, item, priority):
        """Insert with priority. O(log n)."""
        # Negate for max-heap behavior? No, this is min-heap.
        # Use counter as tie-breaker for stability.
        entry = [priority, self._counter, item]
        self._counter += 1
        heapq.heappush(self._heap, entry)
        self._entry_finder[item] = entry

    def pop(self):
        """Remove and return (item, priority). O(log n)."""
        while self._heap:
            priority, count, item = heapq.heappop(self._heap)
            if item is not None:  # Not a removed entry
                del self._entry_finder[item]
                return item, priority
        raise KeyError("pop from empty priority queue")

    def peek(self):
        """Return minimum priority item without removal. O(1)."""
        while self._heap:
            priority, count, item = self._heap[0]
            if item is not None:
                return item, priority
            heapq.heappop(self._heap)  # Clean removed entries
        raise KeyError("peek from empty priority queue")

    def __len__(self):
        return len(self._heap)


# Demonstration
pq = PriorityQueue()
pq.push("Task A", 3)
pq.push("Task B", 1)
pq.push("Task C", 2)
pq.push("Task D", 1)  # Same priority as B

print(f"Min priority: {pq.peek()}")  # Task B (priority 1, added first)
print(f"Popped: {pq.pop()}")         # Task B
print(f"Popped: {pq.pop()}")         # Task D (same priority, added later)
print(f"Popped: {pq.pop()}")         # Task C
print(f"Popped: {pq.pop()}")         # Task A
```

**Expected Output:**
```
Min priority: ('Task B', 1)
Popped: ('Task B', 1)
Popped: ('Task D', 1)
Popped: ('Task C', 2)
Popped: ('Task A', 3)
```

**Why This Output Occurs**: The binary heap maintains the heap invariant: each parent has priority ≤ its children. The counter ensures FIFO ordering for equal priorities (B before D). All operations are O(log n) except `peek` which is O(1) .

### Real-World Cases

- **Dijkstra's Algorithm**: Binary heap is the standard choice for efficient extraction of minimum-distance nodes .
- **Huffman Coding**: Priority queues (often binary heaps) build optimal prefix codes by repeatedly extracting two minimum-frequency nodes.
- **Load Balancing**: Priority queues select the least-loaded server.

### References

- CMU 15-210 - Leftist Heaps and Meld Operation - https://www.cs.cmu.edu/afs/cs/academic/class/15210-f12/www/lectures/lecture27.pdf
- Data Structures, Algorithms, and Program Style Using C (Textbook) - Leftist Trees - 
- Stack Overflow - Priority Queue Implementation Trade-offs - https://stackoverflow.com/questions/64740904/when-to-use-a-binary-heap-based-priority-queue-vs-a-linked-list-based-priority-q


## 5. Monotonic Queue Pattern

### Definitions

**Core Definition**
A monotonic queue is a deque that maintains its elements in sorted order, allowing O(1) access to the minimum or maximum element within a sliding window .

**Technical Definition**
A monotonic queue enforces that elements are in monotonically increasing or decreasing order from front to rear. When a new element arrives, all elements that violate the order are removed from the rear before insertion .

**Beginner-Friendly Explanation**
Imagine a line where people are arranged by height, shortest at the front. When someone new arrives, anyone shorter than them at the back is asked to leave, then the new person joins. This way, the front always has the "most extreme" (shortest) person .

### Purposes (All begin with "To")

- **To** answer sliding window minimum/maximum queries in O(1) amortized time
- **To** reduce O(n × k) sliding window algorithms to O(n)
- **To** maintain a "leaderboard" of elements where only recent extremes matter
- **To** support efficient processing of streaming data with bounded window size

### Syntax Rules and Structure

**General Syntax (Python)**

```python
from collections import deque

class MonotonicQueue:
    def __init__(self):
        self._data = deque()  # Stores indices or values

    def push(self, value):
        # Remove from rear all elements ≤ value (for max queue)
        while self._data and self._data[-1] < value:
            self._data.pop()
        self._data.append(value)

    def pop(self, value):
        # Remove from front if the value is the current maximum
        if self._data and self._data[0] == value:
            self._data.popleft()

    def max(self):
        return self._data[0]  # Front is always the maximum
```

**Component Breakdown**

| Operation | Description | Complexity |
|-----------|-------------|------------|
| `push(value)` | Insert while maintaining monotonicity | O(1) amortized |
| `pop(value)` | Remove value if it's at front (leaving window) | O(1) |
| `max()` / `min()` | Access extreme at front | O(1) |

**Syntax Rules**

- For **maximum** queue: maintain decreasing order (front is largest); remove from rear all elements ≤ new value 
- For **minimum** queue: maintain increasing order (front is smallest); remove from rear all elements ≥ new value
- Store indices rather than values when the window position matters 
- Each element is pushed and popped at most once, giving O(n) total for n operations 

**Constraints and Limitations**

- Only the extreme (min or max) is accessible in O(1); other elements are not queryable
- The queue must process elements in order (not random access)
- Elements removed during `push` are permanently lost (they can never be the extreme while the new element is in the window) 

### Annotated Code Examples

**Example 1: Sliding Window Maximum (Python)**

```python
from collections import deque

def max_sliding_window(nums, k):
    """
    Return the max of each sliding window of size k.
    Uses monotonic deque to achieve O(n) total time.
    """
    result = []
    dq = deque()  # Stores indices; values in decreasing order

    for i, num in enumerate(nums):
        # Remove from rear all indices whose values ≤ current num
        # They can never be the maximum while current num is in window
        while dq and nums[dq[-1]] <= num:
            dq.pop()

        dq.append(i)  # Add current index

        # Remove from front if it's outside the window
        if dq[0] <= i - k:
            dq.popleft()

        # Window has k elements; record maximum (front of deque)
        if i >= k - 1:
            result.append(nums[dq[0]])

    return result


# Demonstration
nums = [1, 3, -1, -3, 5, 3, 6, 7]
k = 3

print(f"Array: {nums}")
print(f"Window size: {k}")
print(f"Sliding window maximums: {max_sliding_window(nums, k)}")

# Step-by-step explanation for first few windows:
# Window [1, 3, -1]: max = 3
# Window [3, -1, -3]: max = 3
# Window [-1, -3, 5]: max = 5
# Window [-3, 5, 3]: max = 5
# Window [5, 3, 6]: max = 6
# Window [3, 6, 7]: max = 7
```

**Expected Output:**
```
Array: [1, 3, -1, -3, 5, 3, 6, 7]
Window size: 3
Sliding window maximums: [3, 3, 5, 5, 6, 7]
```

**Why This Output Occurs**: The deque stores indices of elements in decreasing value order. When a new element is larger than elements at the rear, those smaller elements are removed because they can never be the maximum in any window containing the new element. The front always holds the index of the current window's maximum .

### Real-World Cases

- **Stock Price Analysis**: Finding maximum price in sliding time windows .
- **Network Traffic Monitoring**: Tracking peak bandwidth usage over rolling time intervals.
- **Signal Processing**: Computing moving maximum/minimum for smoothing or peak detection.

### References

- LeetCode - Sliding Window Maximum Solutions (Monotonic Queue) - https://leetcode.com/problems/sliding-window-maximum/solutions/2642665/Simple-TypeScript-Solution-using-Monotonic-Queue/
- GitHub (raul-sauco) - Sliding Window Maximum with Monotonic Queue - https://raw.githubusercontent.com/raul-sauco/coding-challenges/main/leetcode/sliding-window-maximum.rs


## 6. Implementing a Stack using Queues

### Definitions

**Core Definition**
A stack can be simulated using one or two queues by manipulating the FIFO ordering of the underlying queues to produce LIFO behavior .

**Technical Definition**
The two-queue approach uses an auxiliary queue to reverse the order during `push`: new elements are enqueued to the auxiliary queue, all elements from the primary queue are moved behind it, then the queues are swapped .

**Beginner-Friendly Explanation**
Imagine you want to make a line behave like a stack. Each time someone new arrives, you ask everyone already in line to step aside, let the new person go to the front, then have everyone else rejoin behind them. Now the newest person is always at the front .

### Purposes (All begin with "To")

- **To** demonstrate the expressiveness of queue primitives
- **To** provide a stack implementation when only queue operations are available
- **To** explore the relationship between LIFO and FIFO abstractions
- **To** practice algorithm design with constrained primitives

### Syntax Rules and Structure

**Two-Queue Approach (Python)**

```python
from collections import deque

class MyStack:
    def __init__(self):
        self._q1 = deque()  # Primary queue (holds elements in stack order)
        self._q2 = deque()  # Auxiliary queue

    def push(self, x):
        # Add new element to auxiliary queue
        self._q2.append(x)
        # Move all elements from primary to auxiliary (behind new element)
        while self._q1:
            self._q2.append(self._q1.popleft())
        # Swap queues: primary now has new element at front
        self._q1, self._q2 = self._q2, self._q1

    def pop(self):
        return self._q1.popleft()  # O(1)

    def top(self):
        return self._q1[0]  # O(1)

    def empty(self):
        return len(self._q1) == 0
```

**Component Breakdown**

| Operation | Description | Complexity |
|-----------|-------------|------------|
| `push(x)` | Enqueue to q2, move q1 behind it, swap | O(n) |
| `pop()` | Dequeue from q1 | O(1) |
| `top()` | Peek at q1 front | O(1) |
| `empty()` | Check q1 length | O(1) |

**Syntax Rules**

- The key insight: after `push`, the new element must be at the **front** of the primary queue (so it pops first) 
- The auxiliary queue holds the new element first; all existing elements are appended behind it
- Swapping queue references makes the auxiliary queue the new primary 
- Alternative approach: use a single queue by rotating elements after each push 

**Constraints and Limitations**

- `push` is O(n), making the overall stack less efficient than a native implementation
- The alternative single-queue approach also has O(n) `push` 
- This is primarily an educational exercise; production code should use a native stack

### Annotated Code Examples

**Example 1: Stack Using Two Queues**

```python
from collections import deque

class MyStack:
    """Stack implemented using two FIFO queues."""
    def __init__(self):
        self._q1 = deque()  # Primary: elements in LIFO order
        self._q2 = deque()  # Auxiliary

    def push(self, x: int) -> None:
        """Push element x onto stack. Time: O(n)."""
        # Step 1: Add new element to auxiliary queue
        self._q2.append(x)

        # Step 2: Move all existing elements from q1 to q2
        # This places the new element at the FRONT of q2
        while self._q1:
            self._q2.append(self._q1.popleft())

        # Step 3: Swap queues
        # Now q1 has the new element at front, followed by older elements
        self._q1, self._q2 = self._q2, self._q1

    def pop(self) -> int:
        """Remove and return top element. Time: O(1)."""
        return self._q1.popleft()

    def top(self) -> int:
        """Return top element without removal. Time: O(1)."""
        return self._q1[0]

    def empty(self) -> bool:
        """Return True if stack is empty."""
        return len(self._q1) == 0


# Demonstration
stack = MyStack()

# Push elements
stack.push(1)
stack.push(2)
stack.push(3)

print(f"Top: {stack.top()}")      # 3 (last pushed)
print(f"Pop: {stack.pop()}")      # 3
print(f"Pop: {stack.pop()}")      # 2
print(f"Top now: {stack.top()}")  # 1
print(f"Empty: {stack.empty()}")  # False
stack.pop()
print(f"Empty: {stack.empty()}")  # True
```

**Expected Output:**
```
Top: 3
Pop: 3
Pop: 2
Top now: 1
Empty: False
Empty: True
```

**Why This Output Occurs**: After each `push`, the newly added element is at the front of `_q1`. When `1` is pushed, `_q1 = [1]`. When `2` is pushed, `_q2 = [2]`, then `1` is moved behind it: `_q2 = [2, 1]`, and swap gives `_q1 = [2, 1]`. When `3` is pushed, `_q2 = [3]`, then `2, 1` are moved: `_q2 = [3, 2, 1]`, swap gives `_q1 = [3, 2, 1]`. The front of `_q1` is always the top of the stack .

### Real-World Cases

- **Interview Questions**: Classic problem testing understanding of stack/queue duality .
- **Constrained Environments**: When only queue primitives are available but stack behavior is needed.
- **Educational Demonstrations**: Illustrating abstraction and primitive composition.

### References

- AlgoMonster - Implement Stack using Queues - https://algo.monster/liteproblems/225
- Stack Overflow - Implement Stack using Two Queues - https://browse.library.kiwix.org/content/stackoverflow.com_en_all_nopic_2022-07/questions/688276/implement-stack-using-two-queues


## Consolidated References

- University of Waterloo - Deques and Their Implementations - https://ece.uwaterloo.ca/~dwharder/aads/Lecture_materials/3.04.Deques.pdf
- University of Illinois - Variants of Ended Lists (Input/Output-Restricted Deques) - http://jeffe.cs.illinois.edu/teaching/225H/notes/02-05.pdf
- CMU 15-210 - Priority Queues and Leftist Heaps - https://www.cs.cmu.edu/afs/cs/academic/class/15210-f12/www/lectures/lecture27.pdf
- LeetCode - Sliding Window Maximum (Monotonic Queue Solutions) - https://leetcode.com/problems/sliding-window-maximum/solutions/2642665/Simple-TypeScript-Solution-using-Monotonic-Queue/
- AlgoMonster - Implement Stack using Queues - https://algo.monster/liteproblems/225
- Python.org Mailing List - collections.deque Implementation - https://mail.python.org/pipermail/python-list/2010-January/565242.html
- Python Documentation - heapq: Heap Queue Algorithm - https://docs.python.org/3.14/library/heapq.html
- Python Documentation - collections.deque - https://docs.python.org/fr/3.5/library/collections.html#collections.deque
- Python Documentation - heapq (3.11) - https://docs.python.org/3.11/library/heapq.html
- Virginia Tech ECE 2574 - Queue ADT and Deque - https://filebox.ece.vt.edu/~ECE2574/meeting/26-queue/slides.pdf
- Data Structures in C++ (Textbook) - Deque Variations - 
- Stack Overflow - Binary Heap vs Linked List Priority Queue - https://stackoverflow.com/questions/64740904/when-to-use-a-binary-heap-based-priority-queue-vs-a-linked-list-based-priority-q
- GitHub (raul-sauco) - Sliding Window Maximum with Monotonic Queue - https://raw.githubusercontent.com/raul-sauco/coding-challenges/main/leetcode/sliding-window-maximum.rs
- Stack Overflow - Implement Stack using Two Queues - https://browse.library.kiwix.org/content/stackoverflow.com_en_all_nopic_2022-07/questions/688276/implement-stack-using-two-queues
- Python-Dev Mailing List - Document Performance Requirements (collections.deque O(1)) - https://mail.python.org/archives/list/python-dev@python.org/message/LED332QKC2JZBCGHUGZSMCSPZTJJYETX/
- Python Documentation - heapq 优先队列实现说明 - https://docs.python.org/zh-cn/3.14/library/heapq.html
- Python Documentation (Legacy) - collections.defaultdict - 
- USC CS 103 - Deque Implementation using Doubly-Linked Lists - https://bytes.usc.edu/files/cs103/fa25/slides/Unit4c_DoublyLinkedLists.pdf
- Data Structures and Algorithms Using C++ (Textbook) - Deque Types - 
- Data Structures, Algorithms, and Program Style Using C (Textbook) - Leftist Trees and Priority Queues -