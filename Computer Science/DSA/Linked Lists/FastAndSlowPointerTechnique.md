# DSA Fast and Slow Pointer Technique

## Topic Overview

### Definitions

**Core Definition**
The fast and slow pointer technique (also known as the tortoise-and-hare method) uses two references traversing a sequence at different speeds—typically one step per iteration for the slow pointer and two steps for the fast pointer—to solve problems involving cycles, midpoints, and relative positioning in a single pass with constant space.

**Technical Definition**
Two pointers `slow` and `fast` are initialized at the same starting position. `slow` advances by one node per iteration (`slow = slow.next`), while `fast` advances by two (`fast = fast.next.next`). This 2:1 speed ratio creates predictable relative dynamics: in an acyclic sequence, `fast` reaches the end when `slow` is at the midpoint; in a cyclic sequence, `fast` will eventually meet `slow` inside the cycle because the gap between them decreases by one node per iteration .

**Beginner-Friendly Explanation**
Imagine two runners on a track. The slow runner takes one step at a time; the fast runner takes two. On a straight track, when the fast runner finishes, the slow runner is exactly halfway. On a looped track, the fast runner will eventually lap and catch the slow runner—and the place they meet reveals important information about the loop .

### Key Characteristics

- **Constant Space**: Uses only two pointer variables regardless of list length—O(1) auxiliary space 
- **Single Pass**: Solves problems that would otherwise require multiple traversals or extra data structures 
- **Relative Speed Invariant**: The fast pointer gains exactly one node per iteration on the slow pointer, ensuring predictable catch-up 
- **Null-Safety Critical**: Loop conditions must guard against dereferencing null pointers, especially when fast advances by two steps 

### Prerequisites

- Linked list node structure and traversal
- Pointer/reference manipulation
- Basic loop invariants and conditional logic

### Related Programming Areas

- **Cycle Detection**: Floyd's algorithm for linked lists and functional graphs 
- **Midpoint Finding**: One-pass middle element detection 
- **Palindrome Checking**: Combining midpoint finding with reversal 
- **Duplicate Detection**: Floyd's algorithm applied to array-based cycle detection 

### Core Concepts / Features

| Application | Slow Pointer Role | Fast Pointer Role | Key Insight |
|-------------|-------------------|-------------------|-------------|
| Middle Detection | Ends at midpoint | Reaches end | Slow travels half the distance |
| Cycle Detection | Meets fast inside cycle | Catches slow | Gap closes by 1 per step |
| Cycle Start | Resets to head | Stays at meeting point | Equal distance to entry |
| Cycle Length | Counts steps to return | — | Walk around cycle once |


## 1. Tortoise-and-Hare Method

### Definitions

**Core Definition**
The tortoise-and-hare method is the foundational pattern where a slow pointer (tortoise) moves one step and a fast pointer (hare) moves two steps per iteration, exploiting their speed differential to extract structural information about a sequence.

**Technical Definition**
Given a sequence where each element has a single successor (linked list, functional graph), the tortoise-and-hare algorithm maintains two iterators. The hare's position after k iterations is 2k steps from the start, while the tortoise's is k steps. If the sequence contains a cycle, the hare will eventually enter it, and because the hare gains one position per iteration on the tortoise, they must meet within the cycle .

**Beginner-Friendly Explanation**
Think of two people walking through a maze. One walks slowly (one room per minute), the other walks quickly (two rooms per minute). If the maze is a straight line, the fast walker finishes first. If the maze has a loop, the fast walker will eventually catch up to the slow walker from behind—and the place they meet helps you understand the loop's location .

### Purposes (All begin with "To")

- **To** detect whether a linked list or functional graph contains a cycle in O(n) time and O(1) space 
- **To** find the middle element of a linked list in a single pass 
- **To** determine the starting node and length of a cycle when one exists 
- **To** enable other algorithms (palindrome checking, duplicate detection) by providing structural information

### Syntax Rules and Structure

**General Syntax (Basic Pattern)**
```
slow = head
fast = head

while fast is not NULL and fast.next is not NULL:
    slow = slow.next          # Move 1 step
    fast = fast.next.next     # Move 2 steps
    # Optional: check if slow == fast for cycle detection
```

**Component Breakdown**

| Component | Description | Value |
|-----------|-------------|-------|
| `slow` | Tortoise pointer | Advances by 1 |
| `fast` | Hare pointer | Advances by 2 |
| Loop condition | Null safety guard | `fast != NULL && fast.next != NULL` |
| Speed ratio | Fast/slow ratio | 2:1 (most common) |

**Syntax Rules**

- **Initialize both at head**: Both pointers start at the same position 
- **Guard both `fast` and `fast.next`**: The expression `fast.next.next` requires both to be non-null; checking them in the `while` condition prevents null dereference 
- **Slow never null-checked**: The slow pointer trails the fast pointer, so it never reaches null before the fast pointer does 
- **Meeting is guaranteed if cycle exists**: Once both enter the cycle, the gap decreases by exactly one per iteration, so they must meet 

**Constraints and Limitations**

- **Cycle detection is O(n)**: In the worst case, the fast pointer traverses the list once before reaching the end or catching the slow pointer 
- **Meeting point is not the cycle start**: The first meeting occurs somewhere inside the cycle, not necessarily at the entry 
- **Not a general-purpose algorithm**: Designed specifically for sequences with single-successor structure


## 2. Middle-Element Detection

### Definitions

**Core Definition**
Middle-element detection finds the central node of a linked list in a single pass by exploiting the 2:1 speed ratio: when the fast pointer reaches the end, the slow pointer has traveled exactly half the distance, placing it at the middle.

**Technical Definition**
For a list of length n, the fast pointer reaches the end after approximately n/2 iterations. Since the slow pointer advances half as fast, it reaches position n/2 at the same time. For odd n, this is the exact middle; for even n, this is the second of the two middle nodes .

**Beginner-Friendly Explanation**
If you walk through a list at half the speed of your friend, when your friend reaches the end, you are automatically at the middle. You don't need to count the length first—you just walk .

### Purposes (All begin with "To")

- **To** find the midpoint of a linked list in a single traversal without pre-computing length 
- **To** split a list for merge sort or divide-and-conquer algorithms
- **To** enable palindrome checking by identifying the second half for reversal 
- **To** locate the node before the middle for deletion problems 

### Syntax Rules and Structure

**General Syntax**
```
slow = head
fast = head

while fast is not NULL and fast.next is not NULL:
    slow = slow.next
    fast = fast.next.next

# slow is now at the middle
```

**Component Breakdown**

| Component | Description | Value |
|-----------|-------------|-------|
| `slow` | Middle candidate | Advances 1 step |
| `fast` | End detector | Advances 2 steps |
| Loop condition | End detection | `fast != NULL && fast.next != NULL` |
| Result | Middle node | `slow` after loop |

**Syntax Rules**

- **Even-length lists**: The loop returns the **second** middle node, not the first 
- **Odd-length lists**: The loop returns the exact center node 
- **Single-node list**: The loop exits immediately; `slow` is the head 
- **To get the first middle for even lists**: Use `while fast.next != NULL && fast.next.next != NULL` and stop one step earlier 

**Constraints and Limitations**

- **Even vs. odd ambiguity**: The standard condition returns the second middle for even-length lists; adjust the loop condition if the first middle is needed 
- **Deletion requires predecessor tracking**: To delete the middle node, an additional `prev` pointer must trail `slow` 
- **No index access**: Unlike arrays, linked lists cannot directly access `n/2`; the single-pass method is the optimal approach 

### Annotated Code Examples

**Example 1: Finding Middle Node (Python)**
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

# Test: Odd length (1->2->3->4->5)
n1 = ListNode(1); n2 = ListNode(2); n3 = ListNode(3)
n4 = ListNode(4); n5 = ListNode(5)
n1.next = n2; n2.next = n3; n3.next = n4; n4.next = n5
print(f"Middle of odd list: {find_middle(n1).val}")  # 3

# Test: Even length (1->2->3->4)
n1.next = n2; n2.next = n3; n3.next = n4; n4.next = None
print(f"Middle of even list: {find_middle(n1).val}")  # 3 (second middle)
```

**Expected Output:**
```
Middle of odd list: 3
Middle of even list: 3
```

**Why This Output Occurs**: For the odd list (1→2→3→4→5), the fast pointer reaches the end after 2 iterations, and the slow pointer is at node 3. For the even list (1→2→3→4), the fast pointer becomes null after 1.5 iterations, and the slow pointer is at node 3—the second middle node .


## 3. Cycle Detection

### Definitions

**Core Definition**
Cycle detection determines whether a linked list or functional graph contains a cycle (a node reachable from itself) using the tortoise-and-hare method. If the fast pointer ever equals the slow pointer, a cycle exists; if the fast pointer reaches NULL, no cycle exists.

**Technical Definition**
Floyd's cycle-finding algorithm relies on the pigeonhole principle: in a finite sequence where each element maps to exactly one successor, repetition is inevitable if and only if a cycle exists. The fast and slow pointers must meet inside the cycle because the fast pointer gains exactly one position per iteration on the slow pointer, and the cycle is finite .

**Beginner-Friendly Explanation**
If you have two runners on a track that has a loop, the faster runner will eventually catch the slower runner from behind. If there's no loop, the faster runner just finishes and stops. That's the entire idea .

### Purposes (All begin with "To")

- **To** detect cycles in linked lists without using extra memory (O(1) space) 
- **To** determine whether a functional graph (e.g., array where `i -> arr[i]`) contains a cycle 
- **To** solve duplicate detection problems where the array represents a functional graph 
- **To** detect infinite loops in state machines or iterative processes

### Syntax Rules and Structure

**General Syntax**
```
slow = head
fast = head

while fast is not NULL and fast.next is not NULL:
    slow = slow.next
    fast = fast.next.next
    
    if slow == fast:
        return True  # Cycle detected

return False  # No cycle
```

**Component Breakdown**

| Component | Description | Value |
|-----------|-------------|-------|
| `slow` | Tortoise | 1 step per iteration |
| `fast` | Hare | 2 steps per iteration |
| Meeting check | Cycle indicator | `slow == fast` |
| Null check | No-cycle indicator | `fast == NULL` or `fast.next == NULL` |

**Syntax Rules**

- **Check meeting after movement**: Compare `slow == fast` after both pointers have moved 
- **Null check order**: The `while` condition must check `fast != NULL` **before** `fast.next != NULL` to short-circuit safely 
- **Return false on null**: If `fast` or `fast.next` becomes null, no cycle exists 
- **Meeting guarantees cycle**: If `slow == fast`, a cycle definitely exists—there is no false positive 

**Constraints and Limitations**

- **O(n) time**: The fast pointer traverses the list at most once before reaching the end or catching the slow pointer 
- **Meeting point unknown location**: The meeting point is not necessarily the cycle start; a second phase is required to find the entry 
- **Requires single-successor structure**: Not applicable to general graphs with multiple outgoing edges

### Annotated Code Examples

**Example 1: Cycle Detection (Python)**
```python
def has_cycle(head):
    """
    Detect if linked list has a cycle using Floyd's algorithm.
    Returns True if cycle exists, False otherwise.
    """
    slow = head
    fast = head
    
    while fast is not None and fast.next is not None:
        slow = slow.next          # 1 step
        fast = fast.next.next     # 2 steps
        
        if slow == fast:
            return True           # Cycle detected
    
    return False  # Fast reached end, no cycle

# Test: Create cycle
# 1 -> 2 -> 3 -> 4 -> 5 -> (back to 3)
n1 = ListNode(1); n2 = ListNode(2); n3 = ListNode(3)
n4 = ListNode(4); n5 = ListNode(5)
n1.next = n2; n2.next = n3; n3.next = n4; n4.next = n5
n5.next = n3  # Cycle back to node 3

print(f"Has cycle: {has_cycle(n1)}")  # True

# Test: No cycle
n5.next = None
print(f"Has cycle: {has_cycle(n1)}")  # False
```

**Expected Output:**
```
Has cycle: True
Has cycle: False
```

**Why This Output Occurs**: In the cyclic case, both pointers enter the cycle (nodes 3→4→5). The fast pointer gains one position per iteration, eventually meeting the slow pointer inside the cycle. In the acyclic case, the fast pointer reaches `NULL` after traversing the list .


## 4. Cycle-Related Problems

### Definitions

**Core Definition**
Cycle-related problems extend basic cycle detection to find **where** the cycle begins (entry node), **how long** the cycle is (period), and **how far** the cycle is from the head (transient). These require additional phases after the initial meeting point is found.

**Technical Definition**
After the fast and slow pointers meet inside the cycle, resetting one pointer to the head and advancing both at speed 1 causes them to meet exactly at the cycle entry node. This follows from the algebraic identity: if `F` = distance from head to entry, `C` = cycle length, and `a` = distance from entry to meeting point, then `2(F + a) = F + a + C`, giving `F = C - a` .

**Beginner-Friendly Explanation**
Once you know a loop exists, you can find where it starts: send one runner back to the beginning, have both walk at the same speed, and they'll meet exactly at the loop's entrance. To find the loop's length, just walk around it once and count .

### Purposes (All begin with "To")

- **To** find the starting node (entry point) of a cycle in O(n) time and O(1) space 
- **To** compute the length (period) of the cycle 
- **To** determine the distance from the head to the cycle entry (transient length) 
- **To** solve problems like "Find the Duplicate Number" that reduce to cycle entry finding 

### Syntax Rules and Structure

**General Syntax (Cycle Start Finding)**
```
# Phase 1: Find meeting point
slow = fast = head
while fast and fast.next:
    slow = slow.next
    fast = fast.next.next
    if slow == fast:
        break

if not fast or not fast.next:
    return None  # No cycle

# Phase 2: Find cycle entry
slow = head
while slow != fast:
    slow = slow.next
    fast = fast.next

return slow  # Cycle entry node
```

**General Syntax (Cycle Length Finding)**
```
# After finding meeting point (slow == fast inside cycle)
cycle_length = 1
temp = slow.next
while temp != slow:
    temp = temp.next
    cycle_length += 1
```

**Component Breakdown**

| Phase | Purpose | Action |
|-------|---------|--------|
| 1 | Find meeting point | Fast/slow at 2:1 ratio |
| 2 | Find entry node | Reset slow to head, both at 1:1 |
| 3 (optional) | Find cycle length | Walk around cycle from meeting point |
| 4 (optional) | Find transient length | Distance from head to entry |

**Syntax Rules**

- **Phase 2 relies on distance equality**: The distance from head to entry equals the distance from meeting point to entry (modulo cycle length) 
- **Both pointers move at speed 1 in Phase 2**: The fast pointer is no longer faster; both advance one step 
- **Meeting point is inside cycle**: The first meeting always occurs within the cycle, never before entry 
- **Cycle length requires one full loop**: Start from the meeting point and count steps until returning 

**Constraints and Limitations**

- **Cycle entry finding adds O(F) time**: Phase 2 takes at most the distance from head to entry 
- **No cycle means no entry**: Phase 1 must confirm cycle existence before Phase 2 
- **Functional graph semantics**: This approach works for any single-successor structure, including arrays where `i -> arr[i]` 

### Annotated Code Examples

**Example 1: Finding Cycle Start and Length (Python)**
```python
def find_cycle_info(head):
    """
    Find cycle entry node and cycle length.
    Returns (entry_node, cycle_length) or (None, 0) if no cycle.
    """
    # Phase 1: Detect cycle and find meeting point
    slow = fast = head
    has_cycle = False
    
    while fast is not None and fast.next is not None:
        slow = slow.next
        fast = fast.next.next
        if slow == fast:
            has_cycle = True
            break
    
    if not has_cycle:
        return None, 0
    
    # Phase 2: Find cycle entry
    slow = head
    while slow != fast:
        slow = slow.next
        fast = fast.next
    
    entry = slow  # Both meet at entry
    
    # Phase 3: Find cycle length
    length = 1
    temp = entry.next
    while temp != entry:
        temp = temp.next
        length += 1
    
    return entry, length

# Test: Create cycle
# 1 -> 2 -> 3 -> 4 -> 5 -> 6 -> (back to 3)
n1 = ListNode(1); n2 = ListNode(2); n3 = ListNode(3)
n4 = ListNode(4); n5 = ListNode(5); n6 = ListNode(6)
n1.next = n2; n2.next = n3; n3.next = n4
n4.next = n5; n5.next = n6; n6.next = n3  # Cycle

entry, length = find_cycle_info(n1)
print(f"Cycle entry: {entry.val}")  # 3
print(f"Cycle length: {length}")    # 4 (nodes 3,4,5,6)
```

**Expected Output:**
```
Cycle entry: 3
Cycle length: 4
```

**Why This Output Occurs**: In Phase 1, the fast and slow pointers meet somewhere inside the cycle (nodes 3→4→5→6). In Phase 2, resetting `slow` to head and moving both at speed 1 causes them to meet at node 3—the cycle entry. The cycle length is 4 because the cycle includes nodes 3, 4, 5, and 6 .

### Real-World Cases

- **Duplicate Detection**: The "Find the Duplicate Number" problem models the array as a functional graph where `i -> nums[i]`, and the duplicate value is the cycle entry 
- **Memory Leak Detection**: Finding circular references in object graphs
- **State Machine Analysis**: Detecting loops in finite state machines
- **Pollard's Rho Algorithm**: Integer factorization using cycle detection 


## References

- CMU 15-122 Principles of Imperative Computation - Pointers & Linked Lists - http://www.cs.cmu.edu/afs/cs.cmu.edu/academic/class/15122-s14/www/10-pointers.pdf 
- CMU 15-122 Iterated Functions (Floyd's Algorithm) - http://www.cs.cmu.edu/~cdm/pdf/30-iteration.pdf 
- ACM Digital Library - Formal Reasoning About Tortoise-and-Hare - https://dl.acm.org/doi/pdf/10.1145/3572862 
- GitHub - Find Middle Element in One Pass - https://raw.githubusercontent.com/harishvc/challenges/d1031eeb76209ea25ce3fe52da69c8c1559f527c/linked-list-middle-element.py 
- Stack Overflow - How Does Finding a Cycle Start Node Work - https://stackoverflow.com/feeds/question/2936213 
- Data Structures and Algorithms Made Easy - Linked List Middle Element - /hdd/m0103/deepseek/datasets/pilimi-zlib/libgen_rs_non_fic/2555000/560817c82e2310a5099e858e749424d7 
- CMU 15-122 Recitation 11 Solutions - http://www.cs.cmu.edu/afs/cs.cmu.edu/academic/class/15122-s14/www/rec/11-soln.pdf 
- TAMU CSCE 411 - Iterated Functions and Floyd's Algorithm - https://people.engr.tamu.edu/andreas-klappenecker/csce411-s17/AlgProblems1.pdf 
- GitHub - Fast and Slow Pointers Pattern - https://raw.githubusercontent.com/thuva4/Algorithms/refs/heads/master/patterns/fast-slow-pointers.md 
- LeetCode - Delete Middle Node Solutions - https://leetcode.com/problems/delete-the-middle-node-of-a-linked-list/solutions/8335255/ 
- Kiwix Stack Overflow - Finding Starting Node of Loop - https://browse.library.kiwix.org/content/stackoverflow.com_en_all_2023-11/questions/10880672/ 
- Kiwix Stack Overflow - Logic Behind Cycle Detection - https://browse.library.kiwix.org/content/stackoverflow.com_en_all_2023-11/questions/8238593/ 
- Princeton COS 226 - Tortoise and Hare - https://www.cs.princeton.edu/courses/archive/spring18/cos226/meetings/cm07-interview-contd.pdf 
- LeetCode - Linked List Cycle Solutions - https://leetcode.com/problems/linked-list-cycle/solutions/8293145/ 
- GitHub - Midpoint of Linked List - https://raw.githubusercontent.com/shravankumar0811/Coding_Ninjas_In_Python/refs/heads/master/Data-Structures-and-Algorithms-in-Python-master/10%20Linked%20List-2/10.05%20Mid%20Point%20of%20a%20Linked%20List.ipynb 
- LeetCode - Fast and Slow Pointer When, Why, Questions - https://leetcode.com/discuss/post/7437291/ 
- Kiwix Stack Overflow - Cycle Start Node Analysis - https://browse.library.kiwix.org/content/stackoverflow.com_en_all_2023-11/questions/2936213/ 