# Core Linked List Algorithmic Patterns: Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**
Linked list algorithmic patterns are reusable computational strategies that exploit the pointer-based, sequential nature of linked lists to solve problems involving cycle detection, structural manipulation, and ordered merging in optimal time and space.

**Technical Definition**
These patterns constitute template solutions characterized by specific pointer movement invariants (fast/slow speeds, dummy nodes, multi-pointer coordination) that reduce naive O(n²) approaches to O(n) while maintaining O(1) auxiliary space where possible.

**Beginner-Friendly Explanation**
Think of linked list patterns as recipes for navigating a chain of clues. Some recipes use two runners at different speeds to find loops, others use a temporary placeholder to simplify edge cases, and others reverse the chain link by link without needing extra paper.

### Key Characteristics

- **Pointer-Centric**: Solutions manipulate `next` references rather than indices or values
- **In-Place Capability**: Most patterns achieve O(1) extra space
- **Linear Time**: Optimal solutions visit each node a constant number of times
- **Edge-Case Sensitivity**: Head/tail/null handling is critical for correctness

### Prerequisites

- Singly and doubly linked list fundamentals
- Node structure and pointer manipulation
- Big O notation and complexity analysis
- Loop invariants and conditional logic

### Related Programming Areas

- **Interview Preparation**: Linked list problems dominate technical interviews
- **Systems Programming**: Free list management, memory pools
- **Algorithm Design**: Hash table chaining, adjacency lists
- **Functional Programming**: Persistent data structures

### Core Concepts / Features

| Pattern | Primary Use Case | Complexity |
|---------|-----------------|------------|
| Fast/Slow Pointers | Cycle detection, middle finding | O(n) time, O(1) space |
| Reversal | Structural transformation | O(n) time, O(1) space |
| Merging | Combining sorted lists | O(n+m) time |
| Duplicate Removal | Deduplication | O(n) or O(n²) |
| Intersection | Finding junction | O(n+m) time, O(1) space |
| Palindrome Check | Symmetry validation | O(n) time, O(1) space |

---

## 1. Fast and Slow Pointers (Tortoise and Hare)

### Definitions

**Core Definition**
The fast and slow pointer technique uses two references traversing a linked list at different speeds—typically one node per step (slow) and two nodes per step (fast)—to detect cycles, find the middle, or solve structural problems in a single pass.

**Technical Definition**
Two pointers `slow` and `fast` are initialized at the head. `slow` advances by one node (`slow = slow.next`), while `fast` advances by two (`fast = fast.next.next`). The speed ratio creates predictable relative positions: when `fast` reaches the end, `slow` is at the middle; if a cycle exists, `fast` will eventually meet `slow` inside the cycle .

**Beginner-Friendly Explanation**
Imagine two runners on a track. The slow runner takes one step at a time; the fast runner takes two. If the track is a straight line, when the fast runner finishes, the slow runner is exactly halfway. If the track is a loop, the fast runner will eventually lap and catch the slow runner.

### Purposes (All begin with "To")

- **To** detect cycles in a linked list without using extra memory 
- **To** find the middle node in a single pass 
- **To** locate the starting point of a cycle 
- **To** determine the length of a cycle 
- **To** enable palindrome checking and other symmetric operations 

### Syntax Rules and Structure

**General Syntax (Middle Finding)**
```
slow = head
fast = head
while fast != null and fast.next != null:
    slow = slow.next
    fast = fast.next.next
# slow is now at middle
```

**General Syntax (Cycle Detection)**
```
slow = head
fast = head
while fast != null and fast.next != null:
    slow = slow.next
    fast = fast.next.next
    if slow == fast:
        return True  # Cycle detected
return False
```

**Component Breakdown**

| Component | Description | Value |
|-----------|-------------|-------|
| `slow` | Tortoise pointer | Advances by 1 |
| `fast` | Hare pointer | Advances by 2 |
| Loop condition | Safety check | `fast != null && fast.next != null` |
| Meeting point | Cycle detected | `slow == fast` |

**Syntax Rules**

- Initialize both pointers at `head` 
- Loop condition must check both `fast != null` and `fast.next != null` to prevent null dereference 
- For even-length lists, the algorithm returns the second middle node 
- Speed ratio of 2:1 is natural for cycle detection; other ratios can work but change the meeting time 

**Constraints and Limitations**

- Requires O(1) extra space
- Finds cycle existence in O(n) time
- For cycle starting point, a second phase is required 
- For cycle length, an additional loop after detection is needed 

### Annotated Code Examples

**Example 1: Finding the Middle Node (Python)**
```python
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

def find_middle(head):
    """
    Find middle node using slow/fast pointers.
    For even-length lists, returns the second middle.
    """
    slow = head
    fast = head
    
    # Fast moves 2x, slow moves 1x
    while fast is not None and fast.next is not None:
        slow = slow.next      # Move 1 step
        fast = fast.next.next # Move 2 steps
    
    # When fast reaches end, slow is at middle
    return slow

# Test: Odd length
n1 = ListNode(1)
n2 = ListNode(2)
n3 = ListNode(3)
n4 = ListNode(4)
n5 = ListNode(5)
n1.next = n2; n2.next = n3; n3.next = n4; n4.next = n5

print(f"Middle of [1,2,3,4,5]: {find_middle(n1).val}")  # 3

# Test: Even length
n6 = ListNode(6)
n5.next = n6
print(f"Middle of [1,2,3,4,5,6]: {find_middle(n1).val}")  # 4 (second middle)
```

**Expected Output:**
```
Middle of [1,2,3,4,5]: 3
Middle of [1,2,3,4,5,6]: 4
```

**Why This Output Occurs**: For odd-length lists, `fast` reaches the end exactly when `slow` is at the center. For even-length lists, `fast` becomes null after `slow` passes the first middle, so `slow` ends at the second middle .

**Example 2: Cycle Detection and Finding Cycle Start (Python)**
```python
def detect_cycle_start(head):
    """
    Detect cycle and return the starting node of the cycle.
    Phase 1: Find meeting point inside cycle.
    Phase 2: Find cycle entry point.
    """
    if head is None:
        return None
    
    slow = head
    fast = head
    has_cycle = False
    
    # Phase 1: Detect cycle
    while fast is not None and fast.next is not None:
        slow = slow.next
        fast = fast.next.next
        if slow == fast:
            has_cycle = True
            break
    
    if not has_cycle:
        return None
    
    # Phase 2: Find cycle start
    # Reset slow to head; move both at same speed
    slow = head
    while slow != fast:
        slow = slow.next
        fast = fast.next
    
    return slow  # Both meet at cycle start

# Test: Create cycle
# 1 -> 2 -> 3 -> 4 -> 5 -> (back to 3)
n1 = ListNode(1); n2 = ListNode(2); n3 = ListNode(3)
n4 = ListNode(4); n5 = ListNode(5)
n1.next = n2; n2.next = n3; n3.next = n4; n4.next = n5
n5.next = n3  # Cycle back to node 3

cycle_start = detect_cycle_start(n1)
print(f"Cycle starts at node: {cycle_start.val}")  # 3
```

**Expected Output:**
```
Cycle starts at node: 3
```

**Why This Output Occurs**: After detection, `slow` and `fast` meet somewhere inside the cycle. Resetting `slow` to head and moving both at speed 1 causes them to meet exactly at the cycle entry point because the distance from head to cycle start equals the distance from meeting point to cycle start (modulo cycle length) .

### Real-World Cases

- **Memory Leak Detection**: Finding circular references in object graphs
- **Pollard's Rho Algorithm**: Integer factorization uses cycle detection
- **Deadlock Detection**: Operating systems detect circular waits in resource allocation graphs
- **Middle Element Access**: Quick access to central data in a stream

### References

- Carnegie Mellon University - Calculating Transients and Periods - http://www.cs.cmu.edu/afs/andrew.cmu.edu/course/15/354/www/pdf/Iteration3.pdf 
- Stack Overflow - Logic Behind Cycle Detection - https://browse.library.kiwix.org/content/stackoverflow.com_en_all_2023-11/questions/8238593/logic-behind-the-method-to-identify-cycle-in-a-linked-list 
- LeetCode - Middle of the Linked List Solutions - https://leetcode.com/problems/middle-of-the-linked-list/solutions/8337898/middle-of-the-linked-list-java-slow-fast-pointer-one-pass-solution/ 
- GitHub - Find Middle Element in One Pass - https://raw.githubusercontent.com/harishvc/challenges/d1031eeb76209ea25ce3fe52da69c8c1559f527c/linked-list-middle-element.py 

---

## 2. Multi-Pointer & Structural Manipulations

### Definitions

**Core Definition**
Multi-pointer structural manipulation uses three or more references simultaneously to reverse, rearrange, or restructure linked list nodes in place, typically employing a "previous, current, next" triple to track context during pointer rewiring.

**Technical Definition**
A common multi-pointer pattern maintains `prev` (predecessor), `cur` (current node being processed), and `nxt` (successor saved before rewiring). This enables O(n) single-pass transformations without auxiliary data structures.

**Beginner-Friendly Explanation**
When you need to rewire a chain, you hold onto three links at once: where you came from, where you are, and where you're going. This way, you never lose your place when you change a link's direction.

### Purposes (All begin with "To")

- **To** reverse linked list segments without losing references
- **To** rearrange nodes based on conditions (partition, group)
- **To** swap adjacent nodes or reorder in groups
- **To** enable in-place structural transformations

### Syntax Rules and Structure

**General Syntax (Three-Pointer Pattern)**
```
prev = null
cur = head

while cur != null:
    nxt = cur.next      # Save next
    cur.next = prev     # Rewire
    prev = cur          # Advance prev
    cur = nxt           # Advance cur (using saved next)
```

**Component Breakdown**

| Pointer | Role | Update Order |
|---------|------|--------------|
| `prev` | Predecessor context | After rewiring |
| `cur` | Current node | Last |
| `nxt` | Successor safety | First (before rewiring) |

**Syntax Rules**

- Always save `cur.next` before modifying `cur.next` 
- Update `prev` before `cur` to maintain correct ordering
- Return `prev` (new head) after the loop completes

### Annotated Code Examples

**Example 1: Iterative Reversal (C++)**
```cpp
ListNode* reverseList(ListNode* head) {
    ListNode* prev = nullptr;
    ListNode* cur = head;
    
    while (cur != nullptr) {
        ListNode* nxt = cur->next;  // Save next
        cur->next = prev;           // Reverse link
        prev = cur;                 // Move prev forward
        cur = nxt;                  // Move cur forward
    }
    
    return prev;  // New head
}
```

**Expected Output** (for `1->2->3->4->5`):
```
5->4->3->2->1->null
```

**Why This Output Occurs**: Each iteration reverses one link. The saved `nxt` ensures we can advance after `cur->next` is changed .

### Real-World Cases

- **Undo Functionality**: Reversing action history
- **Expression Evaluation**: Reversing Polish notation
- **Data Pipeline Reversal**: Processing streams in reverse

---

## 3. Reversing a Linked List (Iterative and Recursive)

### Definitions

**Core Definition**
Reversal transforms a linked list so that each node's `next` pointer references its former predecessor, making the tail the new head. Iterative reversal uses three pointers; recursive reversal uses the call stack.

**Technical Definition**
Iterative: O(n) time, O(1) space using `prev`, `cur`, `nxt` pointers. Recursive: O(n) time, O(n) stack space, where `head.next.next = head` and `head.next = null` after recursive call returns .

**Beginner-Friendly Explanation**
Reversal is like turning a conga line around—the last person becomes the first, and everyone faces the opposite direction.

### Purposes (All begin with "To")

- **To** prepare lists for certain algorithms (e.g., adding numbers)
- **To** implement undo/redo or history navigation
- **To** solve palindrome and symmetry problems

### Syntax Rules and Structure

**Iterative Syntax**
```
prev = null
cur = head
while cur:
    nxt = cur.next
    cur.next = prev
    prev = cur
    cur = nxt
return prev
```

**Recursive Syntax**
```
def reverse(head):
    if not head or not head.next:
        return head
    new_head = reverse(head.next)
    head.next.next = head
    head.next = null
    return new_head
```

**Component Breakdown**

| Approach | Time | Space | Key Operation |
|----------|------|-------|---------------|
| Iterative | O(n) | O(1) | Three-pointer rewiring |
| Recursive | O(n) | O(n) | `head.next.next = head` |

**Syntax Rules**

- Iterative: Save `nxt` before overwriting `cur.next` 
- Recursive: Base case is empty list or single node 
- Recursive: After recursive call, `head.next` is the new tail; set its `next` to `head` 

### Annotated Code Examples

**Example 1: Recursive Reversal (Python)**
```python
def reverse_recursive(head):
    """Reverse linked list recursively."""
    # Base case: empty or single node
    if head is None or head.next is None:
        return head
    
    # Reverse the rest
    new_head = reverse_recursive(head.next)
    
    # Make next node point back to current
    head.next.next = head
    head.next = None  # Prevent cycle
    
    return new_head
```

**Expected Output** (for `1->2->3`):
```
3->2->1->null
```

**Why This Output Occurs**: The recursion goes to the end (node 3), which becomes the new head. As the stack unwinds, each node's successor is made to point back, and its own `next` is set to null .

### Real-World Cases

- **Big Integer Arithmetic**: Adding numbers stored in reverse order
- **Browser History**: Reversing navigation direction
- **Undo Stacks**: Replaying operations in reverse

### References

- LeetCode - Reverse Linked List Solutions - https://leetcode.com/problems/reverse-linked-list/solutions/58130/C++-Iterative-and-Recursive/ 
- Digital Lab - Advanced Linked List Operations - https://learn.digilabdte.com/books/algorithm-programming-ee/page/7-advanced-linked-list-operations/export/pdf 

---

## 4. Reversing a Sub-List (Between Positions m and n)

### Definitions

**Core Definition**
Sub-list reversal reverses only the nodes between positions m and n (1-indexed), leaving nodes before m and after n in their original order.

**Technical Definition**
A dummy node simplifies head-edge cases. After reaching position m-1, the standard reversal is applied for `n - m` iterations, then the reversed segment is reconnected .

**Beginner-Friendly Explanation**
Imagine reversing a section of a train while keeping the engine and caboose attached. Only the cars in the middle get flipped.

### Purposes (All begin with "To")

- **To** reverse specific portions of a list without affecting the whole
- **To** solve problems requiring local structural changes
- **To** implement complex rearrangements as building blocks

### Syntax Rules and Structure

**General Syntax**
```
dummy = Node(0)
dummy.next = head
prev = dummy

# Move to node before m
for i in range(m - 1):
    prev = prev.next

start = prev.next
then = start.next

# Reverse n - m times
for i in range(n - m):
    start.next = then.next
    then.next = prev.next
    prev.next = then
    then = start.next

return dummy.next
```

**Component Breakdown**

| Variable | Role | Value |
|----------|------|-------|
| `dummy` | Head safety | `dummy.next = head` |
| `prev` | Node before m | Reached after m-1 steps |
| `start` | First node to reverse | `prev.next` |
| `then` | Node to move to front | `start.next` |

**Syntax Rules**

- Use dummy node to handle m=1 uniformly 
- The loop runs `n - m` times (number of nodes after the first in the segment)
- Each iteration moves one node from after `start` to before `start` 

### Annotated Code Examples

**Example 1: Reverse Between m and n (Python)**
```python
def reverse_between(head, m, n):
    """Reverse nodes from position m to n (1-indexed)."""
    if not head or m == n:
        return head
    
    dummy = ListNode(0)
    dummy.next = head
    prev = dummy
    
    # Reach node before position m
    for _ in range(m - 1):
        prev = prev.next
    
    start = prev.next
    then = start.next
    
    # Reverse by moving 'then' to front of segment
    for _ in range(n - m):
        start.next = then.next
        then.next = prev.next
        prev.next = then
        then = start.next
    
    return dummy.next
```

**Expected Output** (for `1->2->3->4->5`, m=2, n=4):
```
1->4->3->2->5
```

**Why This Output Occurs**: Nodes 2, 3, 4 are reversed to 4, 3, 2. Node 1 stays before, node 5 stays after. The dummy node ensures uniform handling if m=1 .

### Real-World Cases

- **Text Editing**: Reversing a selected portion
- **Playlist Reordering**: Reversing a range of tracks
- **Data Segment Transformations**: Reversing byte ranges

### References

- Codepath - Reverse Sublist of a Linked List - https://raw.githubusercontent.com/wiki/codepath/compsci_guides/Reverse-Sublist-of-a-Linked-List.md 
- GitHub - ReverseSublist.java - https://raw.githubusercontent.com/madela-team/Java/refs/heads/master/leet-code-algorithms/src/main/java/org/madela/ReverseSublist.java 

---

## 5. Reversing a Linked List in K-Group Segments

### Definitions

**Core Definition**
K-group reversal reverses nodes in consecutive groups of k, leaving any remaining nodes (fewer than k) unchanged. This is a generalization of full reversal.

**Technical Definition**
Count total nodes to determine complete groups (`count / k`). For each group, isolate it, reverse it, and reconnect it with the previous and next portions .

**Beginner-Friendly Explanation**
If you have a group of 6 people and k=3, you reverse the first 3 and the next 3 separately. If there are 7 people, the last person stays put because there aren't enough to form a group.

### Purposes (All begin with "To")

- **To** reverse lists in fixed-size blocks
- **To** solve problems requiring periodic structural changes
- **To** implement batch processing with reversal

### Syntax Rules and Structure

**General Syntax**
```
count = total nodes
groups = count // k

dummy = Node(0)
dummy.next = head
prev_group = dummy

for _ in range(groups):
    # Find end of current group
    group_head = prev_group.next
    group_tail = group_head
    for _ in range(k - 1):
        group_tail = group_tail.next
    
    next_group = group_tail.next
    
    # Reverse group
    prev = next_group
    cur = group_head
    for _ in range(k):
        nxt = cur.next
        cur.next = prev
        prev = cur
        cur = nxt
    
    # Connect
    prev_group.next = group_tail
    prev_group = group_head

return dummy.next
```

**Component Breakdown**

| Component | Description | Value |
|-----------|-------------|-------|
| `groups` | Number of complete groups | `count // k` |
| `group_head` | First node of group | `prev_group.next` |
| `group_tail` | Last node of group | Reached after k-1 steps |
| `next_group` | First node after group | `group_tail.next` |

**Syntax Rules**

- Count total nodes first to determine how many complete groups exist 
- After reversing, `group_head` becomes the tail, `group_tail` becomes the head 
- Remaining nodes (less than k) are left unchanged
- Dummy node simplifies the first group's connection

### Annotated Code Examples

**Example 1: Reverse K-Group (Python)**
```python
def reverse_k_group(head, k):
    """Reverse nodes in groups of k, leave remainder unchanged."""
    # Count total nodes
    count = 0
    cur = head
    while cur:
        count += 1
        cur = cur.next
    
    dummy = ListNode(0)
    dummy.next = head
    prev_group = dummy
    
    groups = count // k
    
    for _ in range(groups):
        # Find group boundaries
        group_head = prev_group.next
        group_tail = group_head
        for _ in range(k - 1):
            group_tail = group_tail.next
        
        next_group = group_tail.next
        
        # Reverse group
        prev = next_group
        cur = group_head
        for _ in range(k):
            nxt = cur.next
            cur.next = prev
            prev = cur
            cur = nxt
        
        # Connect reversed group
        prev_group.next = group_tail
        prev_group = group_head
    
    return dummy.next
```

**Expected Output** (for `1->2->3->4->5`, k=2):
```
2->1->4->3->5
```

**Why This Output Occurs**: Two complete groups of size 2: (1,2) becomes (2,1), (3,4) becomes (4,3). Node 5 remains because it's in an incomplete group .

### Real-World Cases

- **Batch Processing**: Reversing chunks of data
- **Network Packet Reordering**: Reversing groups of packets
- **Image Processing**: Reversing blocks of pixels

### References

- LeetCode - Reverse Nodes in k-Group Solutions - https://leetcode.com/problems/reverse-nodes-in-k-group/solutions/8303172/o-n-time-o-1-space-cut-reverse-and-reconnect-groups-intuitive-approach/ 

---

## 6. Merging Two Sorted Linked Lists (Iterative and Recursive)

### Definitions

**Core Definition**
Merging combines two sorted linked lists into a single sorted list by repeatedly selecting the smaller head node.

**Technical Definition**
Iterative: Use a dummy node and compare heads, appending the smaller to the result. Recursive: The smaller head becomes the result, and its `next` is the merge of the remaining lists .

**Beginner-Friendly Explanation**
Merging is like shuffling two sorted decks of cards into one sorted deck—you compare the top cards and place the smaller one down first.

### Purposes (All begin with "To")

- **To** combine sorted data from multiple sources
- **To** implement merge sort on linked lists
- **To** merge results from parallel computations

### Syntax Rules and Structure

**Iterative Syntax**
```
dummy = Node(0)
cur = dummy

while l1 and l2:
    if l1.val <= l2.val:
        cur.next = l1
        l1 = l1.next
    else:
        cur.next = l2
        l2 = l2.next
    cur = cur.next

cur.next = l1 or l2
return dummy.next
```

**Recursive Syntax**
```
if not l1: return l2
if not l2: return l1
if l1.val <= l2.val:
    l1.next = merge(l1.next, l2)
    return l1
else:
    l2.next = merge(l1, l2.next)
    return l2
```

**Component Breakdown**

| Approach | Time | Space | Key |
|----------|------|-------|-----|
| Iterative | O(n+m) | O(1) | Dummy node |
| Recursive | O(n+m) | O(n+m) | Call stack |

### Annotated Code Examples

**Example 1: Iterative Merge (Python)**
```python
def merge_two_lists(l1, l2):
    """Merge two sorted lists iteratively."""
    dummy = ListNode(0)
    cur = dummy
    
    while l1 and l2:
        if l1.val <= l2.val:
            cur.next = l1
            l1 = l1.next
        else:
            cur.next = l2
            l2 = l2.next
        cur = cur.next
    
    # Attach remaining
    cur.next = l1 if l1 else l2
    
    return dummy.next
```

**Expected Output** (for `1->2->4` and `1->3->4`):
```
1->1->2->3->4->4
```

**Why This Output Occurs**: The algorithm compares heads, always selecting the smaller. When one list is exhausted, the remainder is appended .

### Real-World Cases

- **Merge Sort**: The merge step of linked list merge sort
- **Database Joins**: Merging sorted result sets
- **Log Aggregation**: Combining timestamped logs

### References

- GitHub - Merge Two Sorted Linked Lists - https://raw.githubusercontent.com/peterxichen/python-algorithms/refs/heads/main/merge-two-sorted-linked-lists.py 
- GitHub - Tranquil Beach Merge Two Lists - https://github.com/yokolet/tranquil-beach/blob/301c87683d3e7e3963367b32b907faa4b1fbb22d/linked_list/merge_two_lists.md 

---

## 7. Merging K Sorted Linked Lists (Divide and Conquer vs. Heap)

### Definitions

**Core Definition**
Merging k sorted lists produces one sorted list. Divide-and-conquer merges pairs iteratively; heap-based uses a min-heap of heads.

**Technical Definition**
Divide-and-conquer: Merge lists pairwise until one remains. O(N log k) time, O(1) space. Heap: Push all heads, pop min, push its next. O(N log k) time, O(k) space .

**Beginner-Friendly Explanation**
Divide-and-conquer is like a tournament—lists merge in pairs, then pairs of pairs, until one champion emerges. Heap is like having all candidates in a priority line, always taking the smallest.

### Purposes (All begin with "To")

- **To** combine results from k sorted streams
- **To** implement external sorting
- **To** merge distributed sorted data

### Syntax Rules and Structure

**Divide and Conquer**
```
while len(lists) > 1:
    merged = []
    for i in range(0, len(lists), 2):
        l1 = lists[i]
        l2 = lists[i+1] if i+1 < len(lists) else None
        merged.append(merge_two(l1, l2))
    lists = merged
return lists[0]
```

**Heap**
```
heap = [(l.val, i, l) for i, l in enumerate(lists) if l]
heapify(heap)
dummy = Node(0)
cur = dummy
while heap:
    val, i, node = heappop(heap)
    cur.next = node
    cur = cur.next
    if node.next:
        heappush(heap, (node.next.val, i, node.next))
return dummy.next
```

**Component Breakdown**

| Approach | Time | Space | Best For |
|----------|------|-------|----------|
| Divide & Conquer | O(N log k) | O(1) | Memory-constrained |
| Heap | O(N log k) | O(k) | Streaming data |

### Annotated Code Examples

**Example 1: Divide and Conquer (Python)**
```python
def merge_k_lists(lists):
    """Merge k sorted lists via divide and conquer."""
    if not lists:
        return None
    
    while len(lists) > 1:
        merged = []
        for i in range(0, len(lists), 2):
            l1 = lists[i]
            l2 = lists[i+1] if i+1 < len(lists) else None
            merged.append(merge_two_lists(l1, l2))
        lists = merged
    
    return lists[0]
```

**Expected Output** (for `[[1,4,5],[1,3,4],[2,6]]`):
```
1->1->2->3->4->4->5->6
```

**Why This Output Occurs**: Lists merge pairwise: [1,4,5] and [1,3,4] become [1,1,3,4,4,5], then merged with [2,6] to produce the final sorted list. The number of merge rounds is log k .

### Real-World Cases

- **External Sorting**: Merging sorted chunks from disk
- **MapReduce**: Reducing sorted mapper outputs
- **Database Systems**: Merging sorted runs

### References

- GitHub - Merge K Sorted Lists - https://raw.githubusercontent.com/Intervue/Data-structures-algorithms-for-interviews/6ae916ae27da0ae0a48cd2b5aa2e4cc6c7eb0550/linked-lists/question15.c 

---

## 8. In-Place Removal of Duplicates

### Definitions

**Core Definition**
Duplicate removal eliminates nodes with repeated values. Sorted lists allow O(n) single-pass; unsorted lists require O(n²) brute force or a hash set.

**Technical Definition**
Sorted: Compare `cur.val` with `cur.next.val`; skip duplicates. Unsorted: For each node, scan ahead and remove matching values .

**Beginner-Friendly Explanation**
In a sorted list, duplicates are adjacent—just skip them. In an unsorted list, you must check every pair.

### Purposes (All begin with "To")

- **To** deduplicate data streams
- **To** clean imported data
- **To** implement set-like behavior

### Syntax Rules and Structure

**Sorted List (O(n))**
```
cur = head
while cur and cur.next:
    if cur.val == cur.next.val:
        cur.next = cur.next.next  # Skip duplicate
    else:
        cur = cur.next
```

**Unsorted List (O(n²))**
```
cur = head
while cur:
    runner = cur
    while runner.next:
        if runner.next.val == cur.val:
            runner.next = runner.next.next
        else:
            runner = runner.next
    cur = cur.next
```

**Component Breakdown**

| List Type | Time | Space | Method |
|-----------|------|-------|--------|
| Sorted | O(n) | O(1) | Compare adjacent |
| Unsorted | O(n²) | O(1) | Nested scan |
| Unsorted | O(n) | O(n) | Hash set |

### Annotated Code Examples

**Example 1: Sorted Duplicate Removal (Python)**
```python
def remove_duplicates_sorted(head):
    """Remove duplicates from sorted list."""
    cur = head
    while cur and cur.next:
        if cur.val == cur.next.val:
            cur.next = cur.next.next  # Remove duplicate
        else:
            cur = cur.next
    return head
```

**Expected Output** (for `1->1->2->3->3`):
```
1->2->3
```

**Why This Output Occurs**: Adjacent duplicates are detected and skipped. Since the list is sorted, all duplicates are contiguous .

### Real-World Cases

- **Data Cleaning**: Removing duplicate records
- **Log Processing**: Deduplicating events
- **Set Implementation**: Maintaining unique elements

### References

- Hacettepe University - Remove Duplicates - https://web.cs.hacettepe.edu.tr/~celikcan/201/vize2.pdf 

---

## 9. Intersecting Linked Lists (Finding the Junction Point)

### Definitions

**Core Definition**
Two lists intersect if they share a common tail. The junction is the first common node. The two-pointer switch technique finds it in O(n+m) time and O(1) space.

**Technical Definition**
Pointer A traverses list A then list B; pointer B traverses list B then list A. Both travel the same total distance, so they meet at the intersection or both become null .

**Beginner-Friendly Explanation**
Two runners on different-length paths switch paths at the end. By the time they meet, they've covered the same distance, so they meet at the junction.

### Purposes (All begin with "To")

- **To** find where two lists merge
- **To** detect shared data structures
- **To** solve problems with unknown length differences

### Syntax Rules and Structure

**General Syntax**
```
pa = headA
pb = headB
while pa != pb:
    pa = pa.next if pa else headB
    pb = pb.next if pb else headA
return pa
```

**Component Breakdown**

| Pointer | Path | Total Distance |
|---------|------|----------------|
| `pa` | A then B | len(A) + len(B) |
| `pb` | B then A | len(B) + len(A) |

### Annotated Code Examples

**Example 1: Intersection Finding (Python)**
```python
def get_intersection_node(headA, headB):
    """Find intersection node using two-pointer switch."""
    pa, pb = headA, headB
    
    while pa != pb:
        pa = pa.next if pa else headB
        pb = pb.next if pb else headA
    
    return pa  # Intersection or None
```

**Expected Output** (for intersecting lists):
```
Intersection at node with value: 8
```

**Why This Output Occurs**: Both pointers travel `len(A) + len(B)` total distance. The length difference is canceled out, so they arrive at the intersection simultaneously .

### Real-World Cases

- **Git Merge Base**: Finding common ancestor commits
- **Dependency Graphs**: Finding shared dependencies
- **File System Links**: Finding common path prefixes

### References

- LeetCode - Intersection of Two Linked Lists - https://leetcode.com/problems/intersection-of-two-linked-lists/solutions/8496730/intersection-of-two-linked-lists-leetcod-2lhw/ 

---

## 10. Palindrome Validation

### Definitions

**Core Definition**
Palindrome validation checks if a linked list reads the same forwards and backwards. The optimal approach uses slow/fast pointers to find the middle, reverses the second half, and compares.

**Technical Definition**
Phase 1: Find middle with slow/fast pointers. Phase 2: Reverse second half. Phase 3: Compare first and second halves .

**Beginner-Friendly Explanation**
Split the list in half, reverse the second half, then check if the halves match like a mirror.

### Purposes (All begin with "To")

- **To** validate symmetric sequences
- **To** solve symmetry-based problems
- **To** combine multiple patterns (middle finding, reversal, comparison)

### Syntax Rules and Structure

**General Syntax**
```
# Phase 1: Find middle
slow = fast = head
while fast and fast.next:
    slow = slow.next
    fast = fast.next.next

# Phase 2: Reverse second half
prev = None
while slow:
    nxt = slow.next
    slow.next = prev
    prev = slow
    slow = nxt

# Phase 3: Compare
left, right = head, prev
while right:
    if left.val != right.val:
        return False
    left = left.next
    right = right.next
return True
```

**Component Breakdown**

| Phase | Operation | Purpose |
|-------|-----------|---------|
| 1 | Slow/Fast pointers | Find middle |
| 2 | Reverse | Enable comparison |
| 3 | Compare | Validate palindrome |

### Annotated Code Examples

**Example 1: Palindrome Check (Python)**
```python
def is_palindrome(head):
    """Check if linked list is palindrome in O(n) time, O(1) space."""
    if not head or not head.next:
        return True
    
    # Find middle
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
    
    # Reverse second half
    prev = None
    while slow:
        nxt = slow.next
        slow.next = prev
        prev = slow
        slow = nxt
    
    # Compare
    left, right = head, prev
    while right:
        if left.val != right.val:
            return False
        left = left.next
        right = right.next
    return True
```

**Expected Output** (for `1->2->2->1`):
```
True
```

**Why This Output Occurs**: Middle is node 2 (second). Second half `2->1` reversed becomes `1->2`. Comparing `1->2` with `1->2` succeeds .

### Real-World Cases

- **String Validation**: Checking palindromic sequences
- **DNA Analysis**: Palindromic sequences in genomes
- **Data Integrity**: Checking symmetric structures

### References

- LeetCode - Palindrome Linked List Solutions - https://leetcode.com/problems/palindrome-linked-list/solutions/1137027/js-python-java-c-easy-floyd-s-reversal-solution-w-explanation/ 

---

## 11. Swapping Nodes in Pairs and Arbitrary Rearrangement

### Definitions

**Core Definition**
Pairwise swapping exchanges adjacent nodes (1↔2, 3↔4, etc.). Arbitrary rearrangement covers other structural changes like odd-even grouping, partition, and rotation.

**Technical Definition**
Pairwise: Use a dummy node, swap `prev.next` and `prev.next.next` pointers iteratively. Other rearrangements use multi-pointer tracking with condition-based relinking.

**Beginner-Friendly Explanation**
Pairwise swapping is like reversing dance partners two at a time. Other rearrangements move nodes based on rules (evens before odds, smaller before larger).

### Purposes (All begin with "To")

- **To** reorder lists based on structural rules
- **To** implement partition operations
- **To** prepare data for specific access patterns

### Syntax Rules and Structure

**Pairwise Swap**
```
dummy = Node(0)
dummy.next = head
prev = dummy

while prev.next and prev.next.next:
    first = prev.next
    second = first.next
    first.next = second.next
    second.next = first
    prev.next = second
    prev = first

return dummy.next
```

### Annotated Code Examples

**Example 1: Pairwise Swap (Python)**
```python
def swap_pairs(head):
    """Swap adjacent nodes pairwise."""
    dummy = ListNode(0)
    dummy.next = head
    prev = dummy
    
    while prev.next and prev.next.next:
        first = prev.next
        second = first.next
        
        # Swap
        first.next = second.next
        second.next = first
        prev.next = second
        
        # Advance
        prev = first
    
    return dummy.next
```

**Expected Output** (for `1->2->3->4`):
```
2->1->4->3
```

**Why This Output Occurs**: Each iteration swaps the two nodes after `prev`. The first pair (1,2) becomes (2,1), second pair (3,4) becomes (4,3).

### Real-World Cases

- **Data Reordering**: Shuffling adjacent pairs
- **Tournament Seeding**: Pairwise matchup arrangement
- **Image Processing**: Swapping pixel pairs

---

## References (Consolidated)

- Carnegie Mellon University - Calculating Transients and Periods - http://www.cs.cmu.edu/afs/andrew.cmu.edu/course/15/354/www/pdf/Iteration3.pdf 
- GitHub - Find Middle Element in One Pass - https://raw.githubusercontent.com/harishvc/challenges/d1031eeb76209ea25ce3fe52da69c8c1559f527c/linked-list-middle-element.py 
- LeetCode - Reverse Linked List Solutions - https://leetcode.com/problems/reverse-linked-list/solutions/58130/C++-Iterative-and-Recursive/ 
- Codepath - Reverse Sublist of a Linked List - https://raw.githubusercontent.com/wiki/codepath/compsci_guides/Reverse-Sublist-of-a-Linked-List.md 
- LeetCode - Reverse Nodes in k-Group Solutions - https://leetcode.com/problems/reverse-nodes-in-k-group/solutions/8303172/o-n-time-o-1-space-cut-reverse-and-reconnect-groups-intuitive-approach/ 
- GitHub - Merge Two Sorted Linked Lists - https://raw.githubusercontent.com/peterxichen/python-algorithms/refs/heads/main/merge-two-sorted-linked-lists.py 
- GitHub - Merge K Sorted Lists - https://raw.githubusercontent.com/Intervue/Data-structures-algorithms-for-interviews/6ae916ae27da0ae0a48cd2b5aa2e4cc6c7eb0550/linked-lists/question15.c 
- LeetCode - Palindrome Linked List Solutions - https://leetcode.com/problems/palindrome-linked-list/solutions/1137027/js-python-java-c-easy-floyd-s-reversal-solution-w-explanation/ 
- Stack Overflow - Logic Behind Cycle Detection - https://browse.library.kiwix.org/content/stackoverflow.com_en_all_2023-11/questions/8238593/logic-behind-the-method-to-identify-cycle-in-a-linked-list 
- LeetCode - Middle of the Linked List Solutions - https://leetcode.com/problems/middle-of-the-linked-list/solutions/8337898/middle-of-the-linked-list-java-slow-fast-pointer-one-pass-solution/ 
- Digital Lab - Advanced Linked List Operations - https://learn.digilabdte.com/books/algorithm-programming-ee/page/7-advanced-linked-list-operations/export/pdf 
- GitHub - ReverseSublist.java - https://raw.githubusercontent.com/madela-team/Java/refs/heads/master/leet-code-algorithms/src/main/java/org/madela/ReverseSublist.java 
- GitHub - Tranquil Beach Merge Two Lists - https://github.com/yokolet/tranquil-beach/blob/301c87683d3e7e3963367b32b907faa4b1fbb22d/linked_list/merge_two_lists.md 
- Hacettepe University - Remove Duplicates - https://web.cs.hacettepe.edu.tr/~celikcan/201/vize2.pdf 
- LeetCode - Intersection of Two Linked Lists - https://leetcode.com/problems/intersection-of-two-linked-lists/solutions/8496730/intersection-of-two-linked-lists-leetcod-2lhw/ 