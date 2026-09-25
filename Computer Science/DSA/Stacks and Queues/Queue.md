# Queue Architecture & Linear Implementations: Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**
A queue is a linear abstract data type (ADT) that stores and retrieves elements according to the First-In-First-Out (FIFO) principle, where insertion occurs at one end (the rear) and deletion occurs at the opposite end (the front).

**Technical Definition**
A queue is an ordered collection of homogeneous elements where insertion (enqueue) happens at the rear and deletion (dequeue) happens at the front, ensuring that the element with the longest residence time in the structure is removed first.

**Beginner-Friendly Explanation**
Think of a line of people waiting at a bank counter. The person who arrived first is served first, and new arrivals join the end of the line. You cannot serve someone in the middle of the line—only the person at the front .

### Key Characteristics

- **FIFO Ordering**: The first element inserted is the first element removed 
- **Two Access Points**: Enqueue operates at the rear; dequeue operates at the front 
- **O(1) Core Operations**: Enqueue, dequeue, front, and isEmpty execute in constant time with proper implementation 
- **Fixed vs. Dynamic Capacity**: Array-based queues may have fixed capacity; linked-list queues grow dynamically
- **Dead Space Problem**: Naive linear array implementations waste space as elements are dequeued from the front 

### Prerequisites

- Understanding of arrays and linked lists
- Familiarity with pointers/references and index manipulation
- Basic knowledge of Big-O notation
- Awareness of modular arithmetic (for circular queues)

### Related Programming Areas

- **Operating Systems**: CPU scheduling, print spooling, interrupt handling 
- **Networking**: Packet buffering, request queuing
- **Graph Algorithms**: Breadth-First Search (BFS) 
- **Real-Time Systems**: Call center systems, traffic signal control 

### Core Concepts

| Concept | Description |
|---------|-------------|
| FIFO Principle | First-In-First-Out ordering for all operations |
| Core Operations | enqueue, dequeue, front, rear, isEmpty, isFull |
| Linear Array Implementation | Fixed-size array with front/rear indices and dead space limitation |
| Circular Queue | Modulo arithmetic wraps indices to reuse space |
| Linked List Implementation | Dynamic nodes with explicit head and tail pointers |
| Complexity Profile | O(1) for all core operations |


## 1. The FIFO (First In, First Out) Abstract Data Type Principle

### Definitions

**Core Definition**
FIFO is a principle where the element that has been in the collection the longest is the first one to be removed .

**Technical Definition**
In a FIFO structure, elements are appended at one end (the rear) and removed from the opposite end (the front), preserving the arrival order of all elements .

**Beginner-Friendly Explanation**
Think of a queue at a movie theater. The person who bought their ticket first gets into the theater first. New people join the back of the line, and the usher only lets people in from the front .

### Purposes (All begin with "To")

- **To** preserve temporal ordering, ensuring fair processing of elements based on arrival time
- **To** enable breadth-first traversal in graph and tree algorithms where level order matters
- **To** manage resources in scheduling systems where fair access is required
- **To** buffer data streams where processing order must match arrival order

### Syntax Rules and Structure

**General FIFO Interface (Python)**
```python
class Queue:
    """Abstract FIFO container."""
    def enqueue(self, item) -> None: ...   # Add to rear
    def dequeue(self): ...                 # Remove from front
    def front(self): ...                   # Inspect front without removal
    def rear(self): ...                    # Inspect rear without removal
    def is_empty(self) -> bool: ...        # Check for elements
    def is_full(self) -> bool: ...         # Check capacity (fixed-size only)
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `enqueue(item)` | Adds `item` to the rear; may raise `OverflowError` if fixed capacity reached |
| `dequeue()` | Removes and returns the front element; raises `IndexError` on empty queue |
| `front()` | Returns the front element without removing; raises `IndexError` on empty queue |
| `rear()` | Returns the rear element without removing; raises `IndexError` on empty queue |
| `is_empty()` | Returns `True` if no elements remain |
| `is_full()` | Returns `True` if the queue has reached its fixed capacity |

**Syntax Rules**

- Insertion occurs exclusively at the rear; deletion occurs exclusively at the front 
- The front element is the one that has been in the queue the longest
- Error conditions (empty/full) must be checked before performing operations

**Constraints and Limitations**

- No random access to middle elements
- Linear search breaks FIFO abstraction
- Fixed-size implementations may overflow; underflow occurs on empty dequeue

### Annotated Code Examples

**Example 1: FIFO Behavior Demonstration (Python)**

```python
class ArrayQueue:
    """Queue backed by Python list; front is index 0, rear is index -1."""
    def __init__(self) -> None:
        self._items: list = []

    def enqueue(self, item) -> None:
        self._items.append(item)  # Add to rear (end of list)

    def dequeue(self):
        if self.is_empty():
            raise IndexError("Dequeue from empty queue")
        return self._items.pop(0)  # Remove from front (index 0)

    def front(self):
        if self.is_empty():
            raise IndexError("Queue is empty")
        return self._items[0]

    def is_empty(self) -> bool:
        return len(self._items) == 0

    def __len__(self) -> int:
        return len(self._items)


# Demonstration
queue = ArrayQueue()
for value in ["First", "Second", "Third"]:
    queue.enqueue(value)

print(f"Queue size: {len(queue)}")
print(f"Front element: {queue.front()}")  # "First" (earliest arrival)

print(f"Dequeued: {queue.dequeue()}")  # "First"
print(f"Dequeued: {queue.dequeue()}")  # "Second"
print(f"Front now: {queue.front()}")   # "Third"
```

**Expected Output:**
```
Queue size: 3
Front element: First
Dequeued: First
Dequeued: Second
Front now: Third
```

**Why This Output Occurs**: Elements are enqueued in order: "First", "Second", "Third". The FIFO principle dictates that "First" (the earliest) is dequeued first, followed by "Second". The `front()` operation confirms "Third" is now at the front .

### Real-World Cases

- **Bank Teller Lines**: Customers are served in arrival order 
- **Print Spooling**: Print jobs are processed in the order they were submitted 
- **BFS Traversal**: Graph nodes are visited level by level, using a queue to track neighbors 
- **Call Centers**: Calls are answered in the order they arrive 

### References

- Oracle Help Center - Stack and Queue Overview - https://docs.oracle.com/cd/E19205-01/820-2985/general/10_1.htm 
- University of Mustansiriyah - Queue Introduction - https://uomustansiriyah.edu.iq/media/lectures/9/9_2020_05_23!08_22_15_AM.pdf 


## 2. Core Operations: enqueue(), dequeue(), front(), rear(), isEmpty(), and isFull()

### Definitions

**Core Definition**
Core queue operations are the fundamental methods defining the queue ADT: adding to the rear (`enqueue`), removing from the front (`dequeue`), inspecting both ends (`front`, `rear`), and checking state (`isEmpty`, `isFull`).

**Technical Definition**
These operations constitute the complete functional interface, with `enqueue` and `dequeue` as primary mutators, `front`/`rear` as inspectors, and `isEmpty`/`isFull` as state predicates .

**Beginner-Friendly Explanation**
These are the six buttons on a queue's control panel: add to the back (enqueue), remove from the front (dequeue), look at the front or back without removing (front/rear), and check if it's empty or full.

### Purposes (All begin with "To")

- **To** provide a complete, minimal interface for FIFO data manipulation
- **To** enable safe operations by checking preconditions before mutating state
- **To** support algorithmic patterns requiring ordered, fair processing
- **To** maintain encapsulation by exposing only necessary state information

### Syntax Rules and Structure

**General Operation Syntax (Python)**

```python
class Queue:
    def enqueue(self, item) -> None:
        """Add item to rear. Raises OverflowError if full (fixed-size)."""
        ...

    def dequeue(self):
        """Remove and return front. Raises IndexError if empty."""
        ...

    def front(self):
        """Return front without removal. Raises IndexError if empty."""
        ...

    def rear(self):
        """Return rear without removal. Raises IndexError if empty."""
        ...

    def is_empty(self) -> bool:
        """Return True if queue has no elements."""
        ...

    def is_full(self) -> bool:
        """Return True if queue has reached capacity (fixed-size only)."""
        ...
```

**Component Breakdown**

| Operation | Precondition | Postcondition | Complexity |
|-----------|--------------|---------------|------------|
| `enqueue(item)` | Not full (fixed-size) | `item` at rear; size + 1 | O(1) |
| `dequeue()` | Not empty | Front removed; size - 1 | O(1)* |
| `front()` | Not empty | Front returned; unchanged | O(1) |
| `rear()` | Not empty | Rear returned; unchanged | O(1) |
| `is_empty()` | None | `True` if size == 0 | O(1) |
| `is_full()` | None | `True` if size == capacity | O(1) |

*O(1) with circular or linked implementations; O(n) with naive linear array shift 

**Syntax Rules**

- `dequeue`, `front`, and `rear` must check `is_empty()` first
- `enqueue` on fixed-size queue must check `is_full()` first
- `isFull` is not applicable to dynamically resizing queues

**Constraints and Limitations**

- Naive linear array `dequeue` requires shifting, making it O(n) 
- Circular array requires careful boundary handling (empty vs. full ambiguity) 
- Linked list requires maintaining tail pointer for O(1) enqueue 

### Annotated Code Examples

**Example 1: Complete Queue Operations with Error Handling (Python)**

```python
from typing import Any

class LinearQueue:
    """Fixed-capacity linear queue using array with shift-based dequeue."""
    def __init__(self, capacity: int = 5) -> None:
        if capacity <= 0:
            raise ValueError("Capacity must be positive")
        self._capacity: int = capacity
        self._arr: list = [None] * capacity
        self._front: int = 0   # Index of front element
        self._rear: int = -1   # Index of rear element (-1 = empty)
        self._size: int = 0

    def enqueue(self, item: Any) -> None:
        if self.is_full():
            raise OverflowError("Queue overflow")
        self._rear += 1
        self._arr[self._rear] = item
        self._size += 1

    def dequeue(self) -> Any:
        if self.is_empty():
            raise IndexError("Queue underflow")
        item = self._arr[self._front]
        # Shift all remaining elements left
        for i in range(self._front, self._rear):
            self._arr[i] = self._arr[i + 1]
        self._rear -= 1
        self._size -= 1
        return item

    def front(self) -> Any:
        if self.is_empty():
            raise IndexError("Queue is empty")
        return self._arr[self._front]

    def rear(self) -> Any:
        if self.is_empty():
            raise IndexError("Queue is empty")
        return self._arr[self._rear]

    def is_empty(self) -> bool:
        return self._size == 0

    def is_full(self) -> bool:
        return self._size == self._capacity


# Demonstration
q = LinearQueue(4)

# Enqueue elements
for ch in "ABCD":
    q.enqueue(ch)

print(f"Queue full? {q.is_full()}")      # True
print(f"Front: {q.front()}, Rear: {q.rear()}")  # A, D

try:
    q.enqueue("E")
except OverflowError as e:
    print(f"Error: {e}")

# Dequeue (shifts elements left)
print(f"Dequeued: {q.dequeue()}")  # A
print(f"Front now: {q.front()}")   # B
print(f"Rear now: {q.rear()}")     # D

# Empty the queue
for _ in range(3):
    q.dequeue()

print(f"Queue empty? {q.is_empty()}")

try:
    q.dequeue()
except IndexError as e:
    print(f"Error: {e}")
```

**Expected Output:**
```
Queue full? True
Front: A, Rear: D
Error: Queue overflow
Dequeued: A
Front now: B
Rear now: D
Queue empty? True
Error: Queue underflow
```

**Why This Output Occurs**: The queue has capacity 4. After enqueuing A, B, C, D, `is_full()` returns `True` and enqueuing E raises `OverflowError`. Dequeuing A causes all remaining elements (B, C, D) to shift left, making B the new front. After dequeuing all elements, `is_empty()` is `True` and further `dequeue` raises `IndexError`.

### Real-World Cases

- **Task Processing**: Each task is enqueued; workers dequeue in FIFO order
- **Request Handling**: Web servers queue incoming requests for fair processing
- **Event Simulation**: Events are processed in chronological order

### References

- BMS College - Queue ADT Operations - https://collegewebsite.bmsce.in/Content/CS/DS-UNIT-2.pdf 
- Stack Overflow - Array Queue Space Wastage - https://stackoverflow.com/feeds/question/36969450 


## 3. Linear Array Implementation and the Structural Limitation of "Dead Space"

### Definitions

**Core Definition**
A linear array queue uses a fixed-size contiguous array with front and rear indices that only increase, causing wasted space at the beginning of the array as elements are dequeued.

**Technical Definition**
In the naive linear implementation, `front` and `rear` both start at -1 and increment on every operation. Dequeue removes from `front` and increments it, but the array positions before `front` become inaccessible, creating "dead space" .

**Beginner-Friendly Explanation**
Imagine a long shelf with numbered slots. You put items at the right end and remove from the left end. As you remove items, the empty slots on the left are "dead" because you can never put new items there—you can only add to the right until you run out of shelf .

### Purposes (All begin with "To")

- **To** provide a simple, straightforward queue implementation for educational purposes
- **To** demonstrate the fundamental limitations of naive array-based queues
- **To** motivate the circular queue solution
- **To** serve as a baseline for complexity comparison

### Syntax Rules and Structure

**General Syntax (Python)**

```python
class NaiveArrayQueue:
    def __init__(self, capacity):
        self._arr = [None] * capacity
        self._front = -1
        self._rear = -1

    def enqueue(self, item):
        if self.is_full():
            raise OverflowError
        if self.is_empty():
            self._front = 0
        self._rear += 1
        self._arr[self._rear] = item

    def dequeue(self):
        if self.is_empty():
            raise IndexError
        item = self._arr[self._front]
        # Shift remaining elements left (O(n))
        for i in range(self._front, self._rear):
            self._arr[i] = self._arr[i + 1]
        self._rear -= 1
        return item
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `_front` | Index of front element; increases on dequeue |
| `_rear` | Index of rear element; increases on enqueue |
| `_arr` | Fixed-size array holding elements |
| Dead space | Positions before `_front` that cannot be reused |

**Syntax Rules**

- `front` and `rear` both start at -1 (empty indicator)
- Enqueue increments `rear` and stores at `_arr[rear]`
- Dequeue reads `_arr[front]`, then shifts all elements left OR increments `front` (causing dead space)

**Constraints and Limitations**

- **Dead Space**: After dequeuing k elements, k array positions are permanently wasted 
- **Fixed Capacity**: Cannot enqueue beyond `capacity` elements even if dead space exists
- **Shift Overhead**: If using shift-based dequeue, each dequeue is O(n) 
- **Full Detection**: `rear == capacity - 1` indicates full, even if front > 0 (space available before front)

### Annotated Code Examples

**Example 1: Demonstrating Dead Space (Python)**

```python
class DeadSpaceQueue:
    """Linear queue with front-incrementing dequeue (no shift)."""
    def __init__(self, capacity: int = 5) -> None:
        self._capacity = capacity
        self._arr = [None] * capacity
        self._front = -1
        self._rear = -1

    def enqueue(self, item) -> None:
        if self._rear == self._capacity - 1:
            raise OverflowError(f"Queue full: rear at {self._rear}, "
                                f"dead space before front={self._front}")
        if self._front == -1:
            self._front = 0
        self._rear += 1
        self._arr[self._rear] = item

    def dequeue(self):
        if self._front == -1 or self._front > self._rear:
            raise IndexError("Queue empty")
        item = self._arr[self._front]
        self._front += 1  # Advances front; positions before are dead
        return item

    def front(self):
        if self._front == -1 or self._front > self._rear:
            raise IndexError("Queue empty")
        return self._arr[self._front]

    def is_empty(self) -> bool:
        return self._front == -1 or self._front > self._rear


# Demonstration
q = DeadSpaceQueue(5)

# Fill the queue
for i in range(5):
    q.enqueue(i * 10)

print(f"Queue full. Front={q._front}, Rear={q._rear}")
print(f"Array contents: {q._arr}")

# Dequeue two elements
print(f"Dequeued: {q.dequeue()}")  # 0
print(f"Dequeued: {q.dequeue()}")  # 10
print(f"After 2 dequeues: Front={q._front}, Rear={q._rear}")
print(f"Array contents: {q._arr}")
print(f"DEAD SPACE: indices 0 and 1 are wasted!")

# Attempt to enqueue new element
try:
    q.enqueue(99)
except OverflowError as e:
    print(f"Error: {e}")
```

**Expected Output:**
```
Queue full. Front=0, Rear=4
Array contents: [0, 10, 20, 30, 40]
Dequeued: 0
Dequeued: 10
After 2 dequeues: Front=2, Rear=4
Array contents: [0, 10, 20, 30, 40]
DEAD SPACE: indices 0 and 1 are wasted!
Error: Queue full: rear at 4, dead space before front=2
```

**Why This Output Occurs**: After dequeuing two elements, `front` advances to index 2, but indices 0 and 1 still hold old values and cannot be reused. The `rear` remains at index 4 (capacity - 1). Enqueuing fails with `OverflowError` even though only 3 elements remain in the queue—indices 0 and 1 are "dead space" .

### Real-World Cases

- **Educational Context**: Teaching the limitations that motivate circular queues
- **Fixed-Partition Systems**: When queue will be completely emptied before refilling
- **Simple Embedded Systems**: Where simplicity outweighs efficiency concerns

### References

- Stack Overflow - Array Implementation Wastage - https://stackoverflow.com/feeds/question/36969450 
- Colby College - Array-based Queue Limitations - https://cs.colby.edu/courses/S18/cs231/notes/outlines15.pdf 
- UMBC - Queue Implementation Limitations - https://redirect.cs.umbc.edu/courses/undergraduate/201/spring99/lectures/lec22/queueimp.shtml 


## 4. Circular Queue Implementation

### Definitions

**Core Definition**
A circular queue uses modulo arithmetic to wrap the front and rear indices around the array's end, reusing dead space and allowing indefinite operation as long as the queue is not full.

**Technical Definition**
In a circular queue, indices advance via `(index + 1) % capacity`, treating the array as a ring. The queue is full when `(rear + 1) % capacity == front`, and empty when `front == rear` (with one slot sacrificed) or when a size counter indicates zero .

**Beginner-Friendly Explanation**
Imagine a circular conveyor belt with numbered slots. When you reach the end, you wrap around to the beginning. This way, you can keep adding items to the back and removing from the front indefinitely, as long as the belt isn't completely full .

### Purposes (All begin with "To")

- **To** eliminate dead space by reusing array positions after dequeue
- **To** achieve O(1) enqueue and dequeue without shifting elements
- **To** provide a fixed-capacity queue with efficient space utilization
- **To** support scenarios where the queue operates continuously (e.g., streaming data)

### Syntax Rules and Structure

**General Syntax (Python)**

```python
class CircularQueue:
    def __init__(self, capacity):
        self._arr = [None] * capacity
        self._capacity = capacity
        self._front = 0
        self._rear = 0
        self._count = 0  # Track size to disambiguate empty vs. full

    def enqueue(self, item):
        if self.is_full():
            raise OverflowError
        self._arr[self._rear] = item
        self._rear = (self._rear + 1) % self._capacity
        self._count += 1

    def dequeue(self):
        if self.is_empty():
            raise IndexError
        item = self._arr[self._front]
        self._front = (self._front + 1) % self._capacity
        self._count -= 1
        return item
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `_front` | Index of front element |
| `_rear` | Index where next element will be placed |
| `_count` | Number of elements (disambiguates empty/full) |
| `(index + 1) % capacity` | Modulo arithmetic for wrapping  |
| `is_full()` | `_count == capacity` |
| `is_empty()` | `_count == 0` |

**Syntax Rules**

- Initialize `front = rear = 0`; `count = 0`
- Enqueue: place at `_rear`, then `_rear = (rear + 1) % capacity`, increment `count`
- Dequeue: read `_front`, then `_front = (front + 1) % capacity`, decrement `count` 
- Full condition: `count == capacity`
- Empty condition: `count == 0`

**Constraints and Limitations**

- Without a count variable, one slot must be sacrificed to distinguish empty from full 
- Fixed capacity still limits maximum elements
- Modulo operation has slight overhead compared to simple increment

### Annotated Code Examples

**Example 1: Circular Queue with Count (Python)**

```python
class CircularQueue:
    """Circular queue using modulo arithmetic and size counter."""
    def __init__(self, capacity: int = 5) -> None:
        if capacity <= 0:
            raise ValueError("Capacity must be positive")
        self._capacity = capacity
        self._arr = [None] * capacity
        self._front = 0   # Index of front element
        self._rear = 0    # Next available slot for enqueue
        self._count = 0   # Current number of elements

    def enqueue(self, item) -> None:
        if self.is_full():
            raise OverflowError("Circular queue is full")
        self._arr[self._rear] = item
        # Wrap rear: after capacity-1 comes 0
        self._rear = (self._rear + 1) % self._capacity
        self._count += 1

    def dequeue(self):
        if self.is_empty():
            raise IndexError("Circular queue is empty")
        item = self._arr[self._front]
        self._arr[self._front] = None  # Optional: clear reference
        self._front = (self._front + 1) % self._capacity
        self._count -= 1
        return item

    def front(self):
        if self.is_empty():
            raise IndexError("Queue is empty")
        return self._arr[self._front]

    def rear(self):
        if self.is_empty():
            raise IndexError("Queue is empty")
        # Rear index points to next slot; actual rear is one before
        actual_rear = (self._rear - 1 + self._capacity) % self._capacity
        return self._arr[actual_rear]

    def is_empty(self) -> bool:
        return self._count == 0

    def is_full(self) -> bool:
        return self._count == self._capacity

    def __len__(self) -> int:
        return self._count


# Demonstration of wraparound
q = CircularQueue(5)

# Enqueue 5 elements (fills the queue)
print("Enqueuing 1-5...")
for i in range(1, 6):
    q.enqueue(i * 10)
print(f"Full: {q.is_full()}, Front: {q.front()}, Rear: {q.rear()}")

# Dequeue 2 elements (creates space at front)
print(f"\nDequeued: {q.dequeue()}")  # 10
print(f"Dequeued: {q.dequeue()}")    # 20
print(f"Front now: {q.front()}, Count: {len(q)}")

# Enqueue 2 more (wraps around to reuse indices 0 and 1)
print("\nEnqueuing 60 and 70 (wraparound)...")
q.enqueue(60)
q.enqueue(70)
print(f"Front: {q.front()}, Rear: {q.rear()}, Count: {len(q)}")

# Display queue contents in order
print("\nQueue contents (front to rear):")
temp = CircularQueue(5)
while not q.is_empty():
    val = q.dequeue()
    print(f"  {val}")
    temp.enqueue(val)

# Restore
while not temp.is_empty():
    q.enqueue(temp.dequeue())

# Demonstrate dead space elimination
print(f"\nAfter wraparound, queue has {len(q)} elements at full capacity")
```

**Expected Output:**
```
Enqueuing 1-5...
Full: True, Front: 10, Rear: 50

Dequeued: 10
Dequeued: 20
Front now: 30, Count: 3

Enqueuing 60 and 70 (wraparound)...
Front: 30, Rear: 70, Count: 5

Queue contents (front to rear):
  30
  40
  50
  60
  70

After wraparound, queue has 5 elements at full capacity
```

**Why This Output Occurs**: After dequeuing 10 and 20, `front` advances to index 2 while `rear` is at index 0 (wrapped). Enqueuing 60 and 70 places them at indices 0 and 1—the formerly "dead" space. The modulo arithmetic `(rear + 1) % 5` wraps index 4 back to 0. The queue now has 5 elements (30, 40, 50, 60, 70) in correct FIFO order, demonstrating full space utilization .

### Real-World Cases

- **Traffic Signal Control**: Cyclic scheduling of traffic phases 
- **Streaming Media Buffers**: Fixed-size buffers that wrap around for continuous data
- **CPU Scheduling**: Round-robin scheduling uses circular queues 
- **Keyboard Buffers**: Keystroke buffers in operating systems

### References

- Colby College - Circular Array Queue Implementation - https://cs.colby.edu/courses/S18/cs231/notes/outlines15.pdf 
- BMS College - Circular Queue and Modulo Arithmetic - https://collegewebsite.bmsce.in/Content/CS/DS-UNIT-2.pdf 
- CSJM University - Circular Arrays Boundary Conditions - https://gyansanchay.csjmu.ac.in/wp-content/uploads/2021/11/Queue-by-programming-code-part-2-of-3.pdf 


## 5. Linked List Implementation of Queues

### Definitions

**Core Definition**
A linked-list queue uses a singly linked list where the head node represents the front of the queue and the tail node represents the rear, with both pointers maintained for O(1) operations.

**Technical Definition**
The linked queue maintains `head` and `tail` references to the first and last nodes respectively. Enqueue appends a new node after `tail` and updates `tail`; dequeue removes the `head` node and advances `head` to `head.next` .

**Beginner-Friendly Explanation**
Imagine a chain of people holding hands. The person at the front (head) leaves first, and new people join at the back (tail) by grabbing the last person's hand. You always know who's at the front and who's at the back .

### Purposes (All begin with "To")

- **To** provide dynamic queue growth without pre-allocating capacity
- **To** achieve strict O(1) enqueue and dequeue without resizing overhead
- **To** avoid wasted space by allocating exactly the needed nodes
- **To** handle unbounded queues subject only to available memory

### Syntax Rules and Structure

**General Syntax (Python)**

```python
class QueueNode:
    def __init__(self, value, next_node=None):
        self.value = value
        self.next = next_node

class LinkedQueue:
    def __init__(self):
        self._head = None  # Front of queue
        self._tail = None  # Rear of queue
        self._size = 0

    def enqueue(self, item):
        new_node = QueueNode(item)
        if self._tail is None:
            self._head = self._tail = new_node
        else:
            self._tail.next = new_node
            self._tail = new_node
        self._size += 1

    def dequeue(self):
        if self._head is None:
            raise IndexError
        value = self._head.value
        self._head = self._head.next
        if self._head is None:
            self._tail = None  # Queue became empty
        self._size -= 1
        return value
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `_head` | Reference to front node (first in queue) |
| `_tail` | Reference to rear node (last in queue) |
| `enqueue` | Creates node, links after tail, updates tail  |
| `dequeue` | Saves head value, advances head, updates tail if empty  |
| Empty condition | `_head is None` |

**Syntax Rules**

- Initialize both `_head` and `_tail` to `None`
- Enqueue: if empty, both `_head` and `_tail` point to new node; else link `_tail.next` to new node and update `_tail`
- Dequeue: save `_head.value`, advance `_head = _head.next`; if `_head` becomes `None`, set `_tail = None` 
- No `isFull()` operation; limited only by memory

**Constraints and Limitations**

- Higher memory overhead per element (value + next reference)
- Poorer cache locality than contiguous arrays
- Python object creation overhead affects performance 

### Annotated Code Examples

**Example 1: Linked Queue Implementation (Python)**

```python
from typing import Any, Optional

class QueueNode:
    """A node in the linked queue."""
    def __init__(self, value: Any, next_node: Optional["QueueNode"] = None) -> None:
        self.value = value
        self.next = next_node


class LinkedQueue:
    """Queue backed by singly linked list with head and tail pointers."""
    def __init__(self) -> None:
        self._head: Optional[QueueNode] = None  # Front
        self._tail: Optional[QueueNode] = None  # Rear
        self._size: int = 0

    def enqueue(self, item: Any) -> None:
        """Add to rear. O(1)."""
        new_node = QueueNode(item)
        if self._tail is None:
            # Queue was empty; new node is both head and tail
            self._head = self._tail = new_node
        else:
            # Link current tail to new node, then update tail
            self._tail.next = new_node
            self._tail = new_node
        self._size += 1

    def dequeue(self) -> Any:
        """Remove from front. O(1)."""
        if self._head is None:
            raise IndexError("Dequeue from empty queue")
        value = self._head.value
        self._head = self._head.next
        if self._head is None:
            # Queue became empty; reset tail
            self._tail = None
        self._size -= 1
        return value

    def front(self) -> Any:
        """Return front without removal. O(1)."""
        if self._head is None:
            raise IndexError("Queue is empty")
        return self._head.value

    def rear(self) -> Any:
        """Return rear without removal. O(1)."""
        if self._tail is None:
            raise IndexError("Queue is empty")
        return self._tail.value

    def is_empty(self) -> bool:
        return self._head is None

    def __len__(self) -> int:
        return self._size


# Demonstration
q = LinkedQueue()

# Enqueue elements
for name in ["Alice", "Bob", "Charlie", "Diana"]:
    q.enqueue(name)

print(f"Queue size: {len(q)}")
print(f"Front: {q.front()}")   # Alice (first in)
print(f"Rear: {q.rear()}")     # Diana (last in)

# Dequeue in FIFO order
print("\nDequeuing all elements:")
while not q.is_empty():
    print(f"  {q.dequeue()}")

print(f"Queue empty: {q.is_empty()}")

# Attempt underflow
try:
    q.dequeue()
except IndexError as e:
    print(f"Error: {e}")
```

**Expected Output:**
```
Queue size: 4
Front: Alice
Rear: Diana

Dequeuing all elements:
  Alice
  Bob
  Charlie
  Diana
Queue empty: True
Error: Dequeue from empty queue
```

**Why This Output Occurs**: Elements are enqueued in order. The `head` points to Alice (first), and `tail` points to Diana (last). Dequeue removes Alice first, then advances `head` to Bob. When Diana is dequeued, `head` becomes `None`, and `tail` is also set to `None` to maintain the empty invariant .

### Real-World Cases

- **Unbounded Event Queues**: When the maximum number of pending events is unknown
- **BFS with Large Graphs**: Linked queues handle arbitrarily large frontier sets
- **Message Passing Systems**: Dynamic queues for inter-process communication
- **Task Schedulers**: When task arrival rate is unpredictable

### References

- JHU DSA - Linked Implementation of Queue - https://jhu-dsa.github.io/notes/13-queue/step03.html 
- GitHub (npatel007) - NodeQueue Implementation - https://raw.githubusercontent.com/npatel007/Computer-Science-Notes/refs/heads/master/Advance%20Data%20Structure%20and%20algorithim%20using%20Java/NodeQueue.java 
- York University - Queue Running Times - https://wiki.eecs.yorku.ca/course_archive/2012-13/S/2011/_media/09Queues.pdf 


## 6. Time and Space Complexity Profiles

### Definitions

**Core Definition**
Time complexity measures how operation time scales with queue size; space complexity measures memory overhead per element and overall.

**Technical Definition**
With proper implementation (circular array or linked list), all core queue operations achieve O(1) time. Space overhead differs: circular arrays pre-allocate fixed capacity; linked queues add pointer overhead per node .

**Beginner-Friendly Explanation**
Both circular array and linked list queues are fast for all operations. The difference is memory: arrays reserve space in advance, while linked lists use exactly what's needed but with extra bookkeeping .

### Complexity Comparison Table

| Operation | Circular Array Queue | Linked Queue | Linear Array (naive) |
|-----------|---------------------|--------------|---------------------|
| **enqueue** | O(1) | O(1) | O(1) |
| **dequeue** | O(1) | O(1) | O(n) (shift) |
| **front** | O(1) | O(1) | O(1) |
| **rear** | O(1) | O(1) | O(1) |
| **is_empty** | O(1) | O(1) | O(1) |
| **is_full** | O(1) | N/A | O(1) |
| **Space (per element)** | 1 slot | Value + pointer | 1 slot |
| **Space (overall)** | Fixed capacity | Exactly n nodes | Fixed capacity |

### Annotated Code Examples

**Example 1: Timing Comparison (Conceptual)**

```python
import time

# Circular array queue
class CircularQueue:
    def __init__(self, cap):
        self._arr = [None] * cap
        self._cap = cap
        self._front = self._rear = self._count = 0
    def enqueue(self, x):
        self._arr[self._rear] = x
        self._rear = (self._rear + 1) % self._cap
        self._count += 1
    def dequeue(self):
        val = self._arr[self._front]
        self._front = (self._front + 1) % self._cap
        self._count -= 1
        return val
    def is_empty(self):
        return self._count == 0

# Linked queue
class Node:
    __slots__ = ('val', 'next')
    def __init__(self, val, next_node=None):
        self.val = val
        self.next = next_node

class LinkedQueue:
    def __init__(self):
        self._head = self._tail = None
    def enqueue(self, x):
        node = Node(x)
        if self._tail is None:
            self._head = self._tail = node
        else:
            self._tail.next = node
            self._tail = node
    def dequeue(self):
        val = self._head.val
        self._head = self._head.next
        if self._head is None:
            self._tail = None
        return val
    def is_empty(self):
        return self._head is None

# Benchmark
N = 500_000

# Circular queue (large enough capacity)
cq = CircularQueue(N)
start = time.perf_counter()
for i in range(N):
    cq.enqueue(i)
for i in range(N):
    cq.dequeue()
cq_time = time.perf_counter() - start

# Linked queue
lq = LinkedQueue()
start = time.perf_counter()
for i in range(N):
    lq.enqueue(i)
for i in range(N):
    lq.dequeue()
lq_time = time.perf_counter() - start

print(f"Circular queue: {cq_time:.4f}s for {N} enqueue+dequeue")
print(f"Linked queue: {lq_time:.4f}s for {N} enqueue+dequeue")
print(f"Circular is {lq_time/cq_time:.1f}x faster in this benchmark")
```

**Expected Output** (timing varies):
```
Circular queue: 0.1523s for 500000 enqueue+dequeue
Linked queue: 0.4871s for 500000 enqueue+dequeue
Circular is 3.2x faster in this benchmark
```

**Why This Output Occurs**: The circular queue uses a pre-allocated list with contiguous memory and modulo arithmetic. The linked queue creates a Python object per node, incurring allocation overhead and poorer cache locality. While both are O(1), constants differ significantly .

### Real-World Cases

- **High-Performance Systems**: Circular queues preferred for predictable memory and speed
- **Unbounded Input**: Linked queues when maximum size is unknown
- **Embedded Systems**: Fixed circular queues for deterministic memory usage

### References

- Montana State University - Queue Complexity O(1) - https://www.cs.montana.edu/pearsall/classes/spring2023/132/lectures/slides/24-StackAndQueueWrapup.pdf 
- JHU DSA - Linked Queue O(1) Operations - https://jhu-dsa.github.io/notes/13-queue/step03.html 
- York University - Queue Implementation Trade-offs - https://wiki.eecs.yorku.ca/course_archive/2012-13/S/2011/_media/09Queues.pdf 


## Consolidated References

- Oracle Help Center - Stack and Queue Overview (FIFO Definition) - https://docs.oracle.com/cd/E19205-01/820-2985/general/10_1.htm
- University of Mustansiriyah - Queue Introduction and Applications - https://uomustansiriyah.edu.iq/media/lectures/9/9_2020_05_23!08_22_15_AM.pdf
- Stack Overflow - Array Queue Space Wastage - https://stackoverflow.com/feeds/question/36969450
- Colby College - Array-based Queue and Circular Arrays - https://cs.colby.edu/courses/S18/cs231/notes/outlines15.pdf
- BMS College - Queue ADT and Circular Queue - https://collegewebsite.bmsce.in/Content/CS/DS-UNIT-2.pdf
- CSJM University - Circular Arrays Boundary Conditions - https://gyansanchay.csjmu.ac.in/wp-content/uploads/2021/11/Queue-by-programming-code-part-2-of-3.pdf
- JHU DSA - Linked Implementation of Queue - https://jhu-dsa.github.io/notes/13-queue/step03.html
- GitHub (npatel007) - NodeQueue Implementation - https://raw.githubusercontent.com/npatel007/Computer-Science-Notes/refs/heads/master/Advance%20Data%20Structure%20and%20algorithim%20using%20Java/NodeQueue.java
- York University - Queue Running Times and Trade-offs - https://wiki.eecs.yorku.ca/course_archive/2012-13/S/2011/_media/09Queues.pdf
- Montana State University - Queue Complexity - https://www.cs.montana.edu/pearsall/classes/spring2023/132/lectures/slides/24-StackAndQueueWrapup.pdf
- UMBC - Queue Implementation Limitations - https://redirect.cs.umbc.edu/courses/undergraduate/201/spring99/lectures/lec22/queueimp.shtml
- GitHub (devanshusomani99) - Indexed Queue Operations - https://raw.githubusercontent.com/devanshusomani99/DS_repo/master/TA1