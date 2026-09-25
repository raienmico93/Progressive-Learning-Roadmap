# Singly Linked Lists & Memory Architecture

## Topic Overview

### Definitions

**Core Definition**
A singly linked list is a linear data structure consisting of nodes where each node contains a data element and a pointer (reference) to the next node in the sequence, with the last node pointing to NULL (or None).

**Technical Definition**
A singly linked list is a concrete data structure composed of dynamically allocated nodes, each containing a `data` field and a `next` pointer. The list is accessed via a `head` pointer to the first node; traversal proceeds only in one direction until the `next` pointer equals NULL, which serves as the sentinel for list termination .

**Beginner-Friendly Explanation**
Think of a singly linked list as a treasure hunt where each clue tells you where the next clue is. You start at the first clue (the head), and each clue contains a piece of information (the data) and directions to the next clue (the pointer). The last clue says "this is the end" (NULL). Unlike an array where all items are in numbered boxes next to each other, a linked list's items can be scattered anywhere in memory—you just follow the chain of pointers.

### Key Characteristics

- **Non-Contiguous Memory**: Nodes are allocated dynamically and may reside anywhere in the heap; logical order is maintained by pointers, not physical adjacency 
- **Dynamic Size**: The list grows and shrinks at runtime by allocating and freeing individual nodes, with no pre-declared capacity 
- **Unidirectional Traversal**: Each node only knows about its successor; backward traversal is impossible without additional data structures 
- **O(1) Head Insertion/Deletion**: Adding or removing at the front requires only pointer updates 
- **O(n) Access**: Reaching the i-th element requires traversing from the head 

### Prerequisites

- Understanding of pointers/references and dynamic memory allocation
- Familiarity with struct/class definitions
- Basic knowledge of NULL/None semantics
- Awareness of stack vs. heap memory

### Related Programming Areas

- **Abstract Data Types**: Stacks, queues, symbol tables
- **Memory Management**: Heap allocation, garbage collection
- **Algorithm Design**: Hash tables (chaining), adjacency lists
- **Systems Programming**: Free list management, memory pools

### Core Concepts / Features

| Concept | Description |
|---------|-------------|
| Node Structure | Data field + next pointer |
| Head/Tail Pointers | References to first/last nodes |
| Dynamic Allocation | Per-node heap allocation at runtime |
| Pointer Traps | Memory leaks, dangling pointers, null dereferences |
| Dummy/Sentinel Nodes | Temporary helper nodes to unify edge cases |

---

## 1. Node Structure (Data and Next Pointer)

### Definitions

**Core Definition**
A node is the fundamental building block of a linked list, containing a data field to store the element and a next pointer that references the subsequent node in the sequence.

**Technical Definition**
A node is typically defined as a self-referential struct or class: `struct Node { DataType data; Node* next; }`. The `next` field holds the memory address of the next node, or NULL/None if this is the last node .

**Beginner-Friendly Explanation**
A node is like a box with two compartments: one holds your item (the data), and the other holds a slip of paper with the address of the next box. If it's the last box, the slip says "nothing" (NULL).

### Purposes (All begin with "To")

- **To** encapsulate a data element together with the information needed to locate the next element
- **To** enable dynamic memory allocation of individual list elements
- **To** provide a self-referential structure that can link to other nodes of the same type

### Syntax Rules and Structure

**General Syntax (C)**
```c
typedef struct Node {
    int data;           // Data field
    struct Node* next;  // Pointer to next node (self-referential)
} Node;
```

**General Syntax (Java)**
```java
class Node {
    int data;
    Node next;
    
    Node(int data) {
        this.data = data;
        this.next = null;
    }
}
```

**Component Breakdown**

| Component | Description | Value |
|-----------|-------------|-------|
| `data` | The stored element | Any type (int, char, object) |
| `next` | Pointer to successor | Address of next Node or NULL |
| Self-reference | Struct/class references its own type | Required for linking |

**Syntax Rules**

- In C, the struct tag must be used inside the definition (`struct Node* next`) because the typedef isn't complete yet 
- In Java/Python, the class can reference itself directly within its definition
- `next` is initialized to NULL (or None) when creating a standalone node
- A node with `next == NULL` marks the end of the list 

**Constraints and Limitations**

- Each node carries pointer overhead (typically 4 or 8 bytes) in addition to data
- No built-in bounds checking; pointer errors cause undefined behavior
- Recursive data structures require careful memory management

### Annotated Code Examples

**Example 1: Creating and Linking Nodes (C)**
```c
#include <stdio.h>
#include <stdlib.h>

typedef struct Node {
    int data;
    struct Node* next;
} Node;

int main() {
    // Allocate three nodes on the heap
    Node* first = (Node*)malloc(sizeof(Node));
    Node* second = (Node*)malloc(sizeof(Node));
    Node* third = (Node*)malloc(sizeof(Node));
    
    // Assign data
    first->data = 10;
    second->data = 20;
    third->data = 30;
    
    // Link nodes
    first->next = second;
    second->next = third;
    third->next = NULL;  // End of list
    
    // Traverse and print
    Node* current = first;
    while (current != NULL) {
        printf("%d -> ", current->data);
        current = current->next;
    }
    printf("NULL\n");
    
    // Free all nodes
    free(first);
    free(second);
    free(third);
    
    return 0;
}
```

**Expected Output:**
```
10 -> 20 -> 30 -> NULL
```

**Why This Output Occurs**: Each node is allocated separately on the heap. The `next` pointers create the chain: first → second → third → NULL. Traversal follows these pointers until NULL is reached .

### Real-World Cases

- **Implementing Stacks**: Push/pop at head uses node allocation 
- **Symbol Tables**: Chaining for hash table collision resolution
- **Polynomial Representation**: Each node stores coefficient and exponent

### References

- IIT Kharagpur - Dynamic Memory Allocation and Linked Lists - http://cse.iitkgp.ac.in/%7Epallab/course/2022/spring%202022/pds%20theory%202022/PPT/12%20-%20Linked%20Lists.pdf
- Huawei Cloud - Singly Linked List Implementation - https://bbs.huaweicloud.com/blogs/d3bf1dac4d3044c29bb852066140d3fe

---

## 2. Dynamic Allocation vs. Contiguous Memory

### Definitions

**Core Definition**
Arrays allocate a single contiguous block of memory at creation, while linked lists allocate each node individually at runtime, scattering nodes across the heap.

**Technical Definition**
Array elements are stored at addresses computed as `base + index × element_size`, enabling O(1) random access . Linked list nodes are allocated via `malloc`/`new`, with each node's address determined by the memory allocator; access requires following pointers from the head .

**Beginner-Friendly Explanation**
An array is like a row of numbered seats in a theater—everyone sits together, and you can walk directly to seat 42. A linked list is like a scavenger hunt—each clue tells you where the next one is, and they could be anywhere.

### Purposes (All begin with "To")

- **To** provide fast O(1) random access (arrays) versus flexible dynamic sizing (linked lists)
- **To** avoid large contiguous allocation requirements when memory is fragmented
- **To** enable efficient insertion/deletion without shifting elements

### Syntax Rules and Structure

**Memory Layout Comparison**
```
Array: [data0][data1][data2][data3]  ← Contiguous, adjacent addresses

Linked List: [data|ptr] → [data|ptr] → [data|ptr] → NULL
              ↑ scattered anywhere in heap
```

**Component Breakdown**

| Aspect | Array | Linked List |
|--------|-------|-------------|
| Allocation | Single block, compile/runtime | Per-node, runtime |
| Address computation | `base + i × size` | Follow `next` pointers |
| Random access | O(1) | O(n) |
| Insert/delete | O(n) shifting | O(1) pointer update (at head) |

**Syntax Rules**

- Arrays require the size (or capacity) known at allocation time
- Linked lists allocate nodes as needed; no pre-declared capacity 
- Linked list nodes have pointer overhead (extra memory per element) 
- Arrays are cache-friendly; linked lists have poor cache locality 

**Constraints and Limitations**

- Arrays need contiguous memory; large arrays may fail to allocate even with enough total free memory 
- Linked lists waste memory on pointers
- Linked lists cannot be indexed directly (no `list[i]` in O(1))
- Array insertion/deletion requires O(n) element shifting 

### Annotated Code Examples

**Example 1: Memory Address Comparison (Conceptual)**
```c
#include <stdio.h>
#include <stdlib.h>

int main() {
    // Array: contiguous memory
    int arr[3] = {10, 20, 30};
    printf("Array addresses:\n");
    for (int i = 0; i < 3; i++) {
        printf("  arr[%d] at %p\n", i, (void*)&arr[i]);
    }
    
    // Linked list: non-contiguous
    typedef struct Node { int data; struct Node* next; } Node;
    Node* a = malloc(sizeof(Node));
    Node* b = malloc(sizeof(Node));
    Node* c = malloc(sizeof(Node));
    a->data = 10; b->data = 20; c->data = 30;
    a->next = b; b->next = c; c->next = NULL;
    
    printf("Node addresses:\n");
    printf("  Node a at %p (data=%d)\n", (void*)a, a->data);
    printf("  Node b at %p (data=%d)\n", (void*)b, b->data);
    printf("  Node c at %p (data=%d)\n", (void*)c, c->data);
    
    free(a); free(b); free(c);
    return 0;
}
```

**Expected Output** (addresses vary):
```
Array addresses:
  arr[0] at 0x7ffd5c8a4a50
  arr[1] at 0x7ffd5c8a4a54
  arr[2] at 0x7ffd5c8a4a58
Node addresses:
  Node a at 0x557a8e9b42a0 (data=10)
  Node b at 0x557a8e9b42c0 (data=20)
  Node c at 0x557a8e9b42e0 (data=30)
```

**Why This Output Occurs**: Array elements are separated by `sizeof(int)` (4 bytes), confirming contiguous allocation. Linked list nodes are at arbitrary heap addresses, possibly with gaps between them .

### Real-World Cases

- **Arrays**: Image pixel buffers, lookup tables, fixed-size records
- **Linked Lists**: Free memory pools, adjacency lists for sparse graphs, undo/redo stacks 

### References

- Carnegie Mellon University - Arrays and Linked Lists - https://www.cs.cmu.edu/~15110-n15/lectures/unit06-1-Arrays_handout.pdf
- Cambridge University Press - Linked List Overview - https://www.cambridge.org/highereducation/books/data-structures-and-algorithms-using-python/86C296AE2484AF8B34B0C35F909B1077/linked-list/38BB2574881F8F0848086C3413B8587A

---

## 3. Head and Tail Pointer Mechanics

### Definitions

**Core Definition**
The head pointer references the first node of the list. An optional tail pointer references the last node, enabling O(1) insertion at the end.

**Technical Definition**
`head` is a pointer to the first `Node`. When the list is empty, `head = NULL`. A `tail` pointer (when maintained) references the last node, whose `next` is NULL. Without a tail pointer, appending requires O(n) traversal .

**Beginner-Friendly Explanation**
The head is the entrance to the list—you must start there to go anywhere. The tail is like having a bookmark at the end, so you can add something quickly without walking through the whole list.

### Purposes (All begin with "To")

- **To** provide a stable entry point for accessing the list
- **To** enable O(1) insertion at the tail when maintained 
- **To** allow empty-list detection (`head == NULL`)
- **To** support queue-like operations efficiently

### Syntax Rules and Structure

**General Syntax (Head Only)**
```c
Node* head = NULL;  // Empty list

// Insert at head (O(1))
Node* newNode = malloc(sizeof(Node));
newNode->data = value;
newNode->next = head;
head = newNode;
```

**General Syntax (Head + Tail)**
```c
Node* head = NULL;
Node* tail = NULL;

// Insert at tail (O(1))
Node* newNode = malloc(sizeof(Node));
newNode->data = value;
newNode->next = NULL;

if (tail == NULL) {  // Empty list
    head = tail = newNode;
} else {
    tail->next = newNode;
    tail = newNode;
}
```

**Component Breakdown**

| Pointer | Description | Empty Value | Update on Insert Head | Update on Insert Tail |
|---------|-------------|-------------|----------------------|----------------------|
| `head` | First node | NULL | New node | Unchanged (unless empty) |
| `tail` | Last node | NULL | Unchanged (unless empty) | New node |

**Syntax Rules**

- Empty list: `head == NULL` (and `tail == NULL` if maintained)
- Single-node list: `head == tail`
- Removing the only node must reset both `head` and `tail` to NULL 
- Tail pointer does NOT help with tail deletion in singly linked lists (still O(n)) 

**Constraints and Limitations**

- Tail pointer adds one pointer of memory overhead
- Tail deletion remains O(n) because you cannot find the previous node 
- List corruption if pointers are not updated consistently

### Annotated Code Examples

**Example 1: Head + Tail Insertion (Java)**
```java
class SinglyLinkedList {
    Node head;
    Node tail;
    int size;
    
    void addFirst(int data) {
        Node newNode = new Node(data);
        newNode.next = head;
        head = newNode;
        if (tail == null) {  // Was empty
            tail = newNode;
        }
        size++;
    }
    
    void addLast(int data) {
        Node newNode = new Node(data);
        if (tail == null) {  // Empty list
            head = tail = newNode;
        } else {
            tail.next = newNode;  // Old tail points to new
            tail = newNode;        // Update tail
        }
        size++;
    }
    
    public static void main(String[] args) {
        SinglyLinkedList list = new SinglyLinkedList();
        list.addFirst(10);   // [10]
        list.addLast(20);    // [10, 20]
        list.addLast(30);    // [10, 20, 30]
        list.addFirst(5);    // [5, 10, 20, 30]
        
        // Print
        Node current = list.head;
        while (current != null) {
            System.out.print(current.data + " -> ");
            current = current.next;
        }
        System.out.println("NULL (size=" + list.size + ")");
    }
}
```

**Expected Output:**
```
5 -> 10 -> 20 -> 30 -> NULL (size=4)
```

**Why This Output Occurs**: `addFirst` links the new node before the current head. `addLast` appends after the current tail and updates the tail pointer. The tail pointer makes `addLast` O(1) instead of O(n) .

### Real-World Cases

- **Queue Implementation**: Head for dequeue, tail for enqueue 
- **Undo Stack**: Head insertion and removal 
- **LRU Cache**: Maintaining recently used items at head

### References

- Auckland COMPSCI 105 - Singly Linked Lists with Head and Tail - https://www.cs.auckland.ac.nz/courses/compsci105s1c/lectures/Bruce/19-LinkedList(3).pdf
- UC Irvine - Linked List Variations Analysis - https://canvas.eee.uci.edu/courses/57346/files/23640903/download

---

## 4. Pointer Manipulation and Common Traps

### Definitions

**Core Definition**
Pointer traps in linked lists include memory leaks (allocated memory never freed), dangling pointers (pointing to deallocated memory), and null pointer dereferences (accessing a node through a NULL pointer).

**Technical Definition**
A memory leak occurs when heap-allocated nodes are no longer reachable but never deallocated. A dangling pointer results when `free()` is called but a reference to that memory is retained. A null dereference attempts to access `ptr->data` or `ptr->next` when `ptr == NULL` .

**Beginner-Friendly Explanation**
Think of a memory leak as losing the key to a storage unit—you paid for it but can't use or remove its contents. A dangling pointer is like having a key to a demolished building. A null dereference is trying to enter a building that doesn't exist.

### Purposes (All begin with "To")

- **To** understand and avoid critical bugs in linked list implementation
- **To** ensure proper memory management (no leaks)
- **To** prevent crashes from invalid pointer access

### Syntax Rules and Structure

**Memory Leak Pattern (WRONG)**
```c
// WRONG: loses reference to old head without freeing
void deleteHead() {
    head = head->next;  // Old head is now leaked!
}
```

**Correct Pattern**
```c
void deleteHead() {
    if (head == NULL) return;
    Node* old = head;      // Save reference
    head = head->next;     // Update head
    free(old);             // Free old head
}
```

**Dangling Pointer Pattern (WRONG)**
```c
Node* p = head;
free(head);
head = NULL;    // Prevent dangling
// p is now dangling if not NULLed
p->data = 5;    // CRASH or undefined behavior
```

**Component Breakdown**

| Trap | Cause | Prevention |
|------|-------|------------|
| Memory leak | Lost reference before free | Save temp pointer before re-linking |
| Dangling pointer | Using memory after free | Set pointer to NULL after free |
| Null dereference | Accessing NULL node | Check `if (ptr != NULL)` |

**Syntax Rules**

- Always save a temporary reference before modifying `head` or `tail` if the old node needs freeing 
- Set pointers to NULL after freeing to prevent dangling references 
- Check `head == NULL` before dereferencing
- In C/C++, every `malloc`/`new` must have a corresponding `free`/`delete` 

**Constraints and Limitations**

- Languages with garbage collection (Java, Python) eliminate leaks and dangling pointers but not null dereferences
- Valgrind can detect memory leaks and dangling pointer uses in C/C++ 
- Double-free (calling free twice) is another common error

### Annotated Code Examples

**Example 1: Memory Leak vs. Correct Deletion (C)**
```c
#include <stdio.h>
#include <stdlib.h>

typedef struct Node { int data; struct Node* next; } Node;

// WRONG: memory leak
void deleteHeadLeak(Node** head) {
    if (*head == NULL) return;
    *head = (*head)->next;  // Old head leaked!
}

// CORRECT: proper deletion
void deleteHeadCorrect(Node** head) {
    if (*head == NULL) return;
    Node* old = *head;       // Save old head
    *head = (*head)->next;   // Update head
    free(old);               // Free old head
}

int main() {
    // Create list: 10 -> 20
    Node* head = malloc(sizeof(Node));
    head->data = 10;
    head->next = malloc(sizeof(Node));
    head->next->data = 20;
    head->next->next = NULL;
    
    // deleteHeadLeak(&head);  // Would leak the 10-node
    deleteHeadCorrect(&head);  // Properly frees 10-node
    
    printf("Head data: %d\n", head->data);  // 20
    free(head);
    return 0;
}
```

**Expected Output:**
```
Head data: 20
```

**Why This Output Occurs**: `deleteHeadCorrect` saves the old head in `old`, updates `head` to point to the next node, then frees `old`. The leak version would lose the address of the 10-node, making it unreachable but still allocated .

### Real-World Cases

- **Embedded Systems**: Manual memory management requires careful free/delete
- **Long-Running Servers**: Leaks accumulate and exhaust memory
- **Security**: Dangling pointers can be exploited for arbitrary code execution

### References

- UCSB CS16 - Linked Lists and Dynamic Memory Errors - https://ucsb-cs16.github.io/w20/lectures/CS16_Lecture14_ann.pdf
- Princeton COS 217 - Dynamaid Dangling Pointer Detection - https://www.cs.princeton.edu/%7Erdondero/iw/14Morrison.pdf
- Stack Overflow - Why Use Temporary Pointer Before delete - https://browse.library.kiwix.org/content/stackoverflow.com_en_all_nopic_2022-07/questions/46102696/pointers-in-a-linked-list-example

---

## 5. Time and Space Complexity Profiles

### Definitions

**Core Definition**
Singly linked lists provide O(1) insertion/deletion at the head but O(n) access to arbitrary positions, with O(n) space overhead due to per-node pointers.

**Technical Definition**
The time complexity of `get(i)` is O(i) because traversal from head is required. Insertion at head is O(1); at tail with tail pointer is O(1); at arbitrary position is O(n) for traversal plus O(1) pointer update. Space is O(n) for n nodes, each with data + pointer overhead .

**Beginner-Friendly Explanation**
Getting the first item is instant. Getting the 100th item requires walking through 99 items. Adding at the front is instant. Adding at the end requires walking to the end (unless you have a tail bookmark).

### Purposes (All begin with "To")

- **To** select the appropriate data structure based on operation frequency
- **To** predict performance characteristics for algorithm design
- **To** understand trade-offs versus arrays

### Syntax Rules and Structure

**Complexity Summary**

| Operation | Time | Space | Notes |
|-----------|------|-------|-------|
| Access `get(i)` | O(i) ≈ O(n) | O(1) | Traverse from head |
| Search `find(x)` | O(n) | O(1) | Linear scan |
| Insert at head | O(1) | O(1) | Pointer update only |
| Insert at tail (no tail ptr) | O(n) | O(1) | Traverse to end |
| Insert at tail (with tail ptr) | O(1) | O(1) | Update tail.next and tail |
| Insert at position i | O(i) | O(1) | Traverse + link |
| Delete at head | O(1) | O(1) | Free old head |
| Delete at tail | O(n) | O(1) | Find prev, cannot go backward  |
| Delete at position i | O(i) | O(1) | Traverse to prev |

**Component Breakdown**

| Metric | Array | Singly Linked List |
|--------|-------|-------------------|
| Access | O(1) | O(n) |
| Insert head | O(n) | O(1) |
| Insert tail | O(1)* | O(n) or O(1)** |
| Delete head | O(n) | O(1) |
| Delete tail | O(1)* | O(n) |
| Space per element | Data only | Data + pointer |
| Cache locality | Excellent | Poor |

*Amortized for dynamic arrays; **with tail pointer

### Annotated Code Examples

**Example 1: Complexity Demonstration (Timing Concept)**
```java
class Node {
    int data;
    Node next;
    Node(int data) { this.data = data; }
}

public class ComplexityDemo {
    static Node head;
    static Node tail;
    
    // O(1) head insertion
    static void insertHead(int data) {
        Node n = new Node(data);
        n.next = head;
        head = n;
        if (tail == null) tail = n;
    }
    
    // O(1) tail insertion (with tail pointer)
    static void insertTail(int data) {
        Node n = new Node(data);
        if (tail == null) { head = tail = n; }
        else { tail.next = n; tail = n; }
    }
    
    // O(n) access
    static int get(int index) {
        Node cur = head;
        for (int i = 0; i < index; i++) cur = cur.next;
        return cur.data;
    }
    
    public static void main(String[] args) {
        // Build list of 1000 elements
        for (int i = 1; i <= 1000; i++) insertTail(i);
        
        long start = System.nanoTime();
        insertHead(0);  // O(1)
        long headTime = System.nanoTime() - start;
        
        start = System.nanoTime();
        get(999);  // O(n) — traverse 999 nodes
        long getTime = System.nanoTime() - start;
        
        System.out.println("insertHead (O(1)): " + headTime + " ns");
        System.out.println("get(999) (O(n)): " + getTime + " ns");
    }
}
```

**Expected Output** (timing varies):
```
insertHead (O(1)): 200 ns
get(999) (O(n)): 45000 ns
```

**Why This Output Occurs**: `insertHead` performs a constant number of operations regardless of list size. `get(999)` must traverse 999 nodes, so its time scales linearly with the index .

### Real-World Cases

- **Stack (LIFO)**: Head operations are O(1)—perfect for linked lists 
- **Queue (FIFO)**: Head + tail combination gives O(1) enqueue/dequeue 
- **Frequent Middle Insertion**: Linked lists beat arrays when insertion position is known via reference 

### References

- York University - Singly Linked List Complexity Analysis - https://www.eecs.yorku.ca/%7Ewangcw/teaching/lectures/2025/W/EECS2101/slides/EECS2101_W25_All_Slides-4up.pdf
- DSA.js Data Structures - Linear Data Structures Complexity - https://raw.githubusercontent.com/luastoned/dsa.js-data-structures-algorithms-javascript/cc1ccc0183ea5d4675815413b7d1d4cc14da348c/book/chapters/linear-data-structures-outro.adoc
- METU - Singly Linked List Implementation - https://users.metu.edu.tr/degerli/DSA/DSA-2-LinkedLists.pdf

---

## 6. Basic Operations

### Definitions

**Core Definition**
Basic linked list operations include traversal (visiting all nodes), insertion (adding a node), deletion (removing a node), searching (finding a value), and updating (modifying a value), performed at the head, tail, or arbitrary positions.

**Technical Definition**
Insertion at head: create node, set `new->next = head`, update `head = new`. Insertion at tail: traverse to last node, set `last->next = new`. Deletion: find predecessor, update `pred->next = target->next`, free target .

**Beginner-Friendly Explanation**
Adding at the front is like pushing a new car onto the front of a train. Adding at the end requires walking to the last car. Removing is like unlinking a car from the middle by connecting its neighbors together.

### Purposes (All begin with "To")

- **To** maintain a dynamic collection with efficient head operations
- **To** implement stack, queue, and other abstract data types
- **To** provide in-place modification without shifting elements

### Syntax Rules and Structure

**General Syntax (Insert at Head)**
```
new->next = head
head = new
```

**General Syntax (Insert at Tail)**
```
new->next = NULL
if head == NULL:
    head = tail = new
else:
    tail->next = new
    tail = new
```

**General Syntax (Delete at Head)**
```
if head != NULL:
    old = head
    head = head->next
    free(old)
```

**General Syntax (Search)**
```
cur = head
while cur != NULL:
    if cur->data == target: return cur
    cur = cur->next
return NULL
```

**Component Breakdown**

| Operation | Steps | Complexity |
|-----------|-------|------------|
| Traversal | `cur = head; while cur: process; cur = cur->next` | O(n) |
| Insert head | Link new to head, update head | O(1) |
| Insert tail | Traverse to end (or use tail), link | O(n) / O(1) |
| Delete head | Save head, advance head, free | O(1) |
| Delete tail | Traverse to second-last, unlink | O(n) |
| Search | Linear scan | O(n) |
| Update | Traverse to node, change data | O(n) |

### Annotated Code Examples

**Example 1: Complete Operations (C++)**
```cpp
#include <iostream>
using namespace std;

struct Node {
    int data;
    Node* next;
    Node(int d) : data(d), next(nullptr) {}
};

class SinglyLinkedList {
public:
    Node* head = nullptr;
    Node* tail = nullptr;
    int size = 0;
    
    // O(1) insert at head
    void pushFront(int data) {
        Node* n = new Node(data);
        n->next = head;
        head = n;
        if (tail == nullptr) tail = n;
        size++;
    }
    
    // O(1) insert at tail (with tail pointer)
    void pushBack(int data) {
        Node* n = new Node(data);
        if (tail == nullptr) {
            head = tail = n;
        } else {
            tail->next = n;
            tail = n;
        }
        size++;
    }
    
    // O(1) delete head
    void popFront() {
        if (head == nullptr) return;
        Node* old = head;
        head = head->next;
        if (head == nullptr) tail = nullptr;
        delete old;
        size--;
    }
    
    // O(n) delete by value
    void remove(int data) {
        if (head == nullptr) return;
        
        // Special case: head
        if (head->data == data) {
            popFront();
            return;
        }
        
        // Find predecessor
        Node* prev = head;
        while (prev->next != nullptr && prev->next->data != data) {
            prev = prev->next;
        }
        
        if (prev->next != nullptr) {
            Node* target = prev->next;
            prev->next = target->next;
            if (target == tail) tail = prev;
            delete target;
            size--;
        }
    }
    
    // O(n) search
    Node* find(int data) {
        Node* cur = head;
        while (cur != nullptr) {
            if (cur->data == data) return cur;
            cur = cur->next;
        }
        return nullptr;
    }
    
    // O(n) update
    bool update(int oldVal, int newVal) {
        Node* node = find(oldVal);
        if (node == nullptr) return false;
        node->data = newVal;
        return true;
    }
    
    void print() {
        Node* cur = head;
        while (cur != nullptr) {
            cout << cur->data << " -> ";
            cur = cur->next;
        }
        cout << "NULL (size=" << size << ")" << endl;
    }
};

int main() {
    SinglyLinkedList list;
    list.pushBack(10);
    list.pushBack(20);
    list.pushFront(5);
    list.pushBack(30);
    list.print();  // 5 -> 10 -> 20 -> 30 -> NULL
    
    list.remove(20);
    list.print();  // 5 -> 10 -> 30 -> NULL
    
    list.popFront();
    list.print();  // 10 -> 30 -> NULL
    
    list.update(30, 99);
    list.print();  // 10 -> 99 -> NULL
    
    return 0;
}
```

**Expected Output:**
```
5 -> 10 -> 20 -> 30 -> NULL (size=4)
5 -> 10 -> 30 -> NULL (size=3)
10 -> 30 -> NULL (size=2)
10 -> 99 -> NULL (size=2)
```

**Why This Output Occurs**: `pushBack` appends at tail (O(1) with tail pointer). `pushFront` inserts at head (O(1)). `remove(20)` finds 20's predecessor (10) and unlinks 20. `popFront` removes 5. `update` finds 30 and changes it to 99 .

### Real-World Cases

- **Browser History**: Back/forward navigation using doubly linked lists (singly for forward-only)
- **Music Playlist**: Add/remove songs, next track
- **Task Queues**: Enqueue at tail, dequeue from head 

### References

- IIT Kharagpur - List Operations (Insert, Delete, Search, Free) - http://cse.iitkgp.ac.in/%7Epallab/course/2022/spring%202022/pds%20theory%202022/PPT/12%20-%20Linked%20Lists.pdf
- McMaster University - Linked Lists Insertion and Deletion - http://www.cas.mcmaster.ca/~kahl/SE2S03/2005/SE2S-2005_Lists_4up.pdf
- C.S.S. University - Singly Linked List Operations - https://bridge-library.ccsuniversity.ac.in/pdf/btech-cs/Linklist%20L-44,45,46,47,48-converted.pdf

---

## 7. The Dummy Node / Sentinel Node Technique

### Definitions

**Core Definition**
A dummy (sentinel) node is a temporary auxiliary node placed before the real head of a linked list to unify edge cases, particularly deletions and insertions that may affect the head.

**Technical Definition**
A dummy node `dummy` with `dummy.next = head` provides a consistent predecessor for the first real node. Operations that modify `head` can be expressed as modifications to `dummy.next`, treating head deletion the same as middle deletion .

**Beginner-Friendly Explanation**
A dummy node is like a placeholder bookmark at the very beginning of a book. It's not a real page, but it makes it easier to handle operations at the front because you always have a "previous" page to work with.

### Purposes (All begin with "To")

- **To** eliminate special-case code for head insertion/deletion
- **To** unify edge cases (empty list, single node, head modification)
- **To** simplify linked list algorithms in interviews and competitive programming
- **To** reduce bug potential from duplicated logic

### Syntax Rules and Structure

**General Syntax (Without Dummy)**
```
// Delete value requires special case for head
if head != NULL && head.data == target:
    head = head.next
else:
    find predecessor of target
    update predecessor.next
```

**General Syntax (With Dummy)**
```
dummy = Node(0)
dummy.next = head
prev = dummy

while prev.next != NULL:
    if prev.next.data == target:
        prev.next = prev.next.next  // Delete
    else:
        prev = prev.next

return dummy.next  // Real head
```

**Component Breakdown**

| Aspect | Without Dummy | With Dummy |
|--------|--------------|------------|
| Head deletion | Special case | Uniform |
| Empty list | Special case | Uniform |
| Predecessor | May not exist for head | Always exists (dummy) |
| Return value | `head` | `dummy.next` |
| Extra memory | None | One node |

**Syntax Rules**

- Dummy node is a local variable; never return `&dummy`, return `dummy.next` 
- Dummy node's data is irrelevant (often set to 0 or None)
- After processing, `dummy.next` points to the actual head
- The dummy node itself is typically on the stack (C++) or garbage-collected (Java/Python)

**Constraints and Limitations**

- Requires one extra node allocation (usually negligible)
- Must remember to return `dummy.next`, not `dummy`
- Dummy node cannot be returned by address (it's a local variable) 

### Annotated Code Examples

**Example 1: Delete All Occurrences (With and Without Dummy)**
```java
class Node {
    int val;
    Node next;
    Node(int val) { this.val = val; }
}

public class DummyNodeDemo {
    // WITHOUT dummy: special case for head
    static Node removeElementsWithoutDummy(Node head, int val) {
        // Remove leading nodes with target value
        while (head != null && head.val == val) {
            head = head.next;
        }
        
        if (head == null) return null;
        
        // Remove non-head occurrences
        Node cur = head;
        while (cur.next != null) {
            if (cur.next.val == val) {
                cur.next = cur.next.next;
            } else {
                cur = cur.next;
            }
        }
        return head;
    }
    
    // WITH dummy: unified logic
    static Node removeElementsWithDummy(Node head, int val) {
        Node dummy = new Node(0);
        dummy.next = head;
        Node cur = dummy;
        
        while (cur.next != null) {
            if (cur.next.val == val) {
                cur.next = cur.next.next;  // Delete: same for head or middle
            } else {
                cur = cur.next;
            }
        }
        
        return dummy.next;  // Real head
    }
    
    public static void main(String[] args) {
        // Build: 1 -> 2 -> 6 -> 3 -> 4 -> 5 -> 6
        Node head = new Node(1);
        head.next = new Node(2);
        head.next.next = new Node(6);
        head.next.next.next = new Node(3);
        head.next.next.next.next = new Node(4);
        head.next.next.next.next.next = new Node(5);
        head.next.next.next.next.next.next = new Node(6);
        
        // Test with dummy
        Node result = removeElementsWithDummy(head, 6);
        
        // Print
        Node cur = result;
        while (cur != null) {
            System.out.print(cur.val + " -> ");
            cur = cur.next;
        }
        System.out.println("NULL");
    }
}
```

**Expected Output:**
```
1 -> 2 -> 3 -> 4 -> 5 -> NULL
```

**Why This Output Occurs**: The dummy node `dummy.next = head` ensures `cur` always has a valid predecessor. When `cur.next.val == 6`, we skip it with `cur.next = cur.next.next`. If the head had value 6, `dummy.next` would be updated just like any other node's `next`. Without the dummy, we'd need a separate `while` loop to remove leading 6s .

### Real-World Cases

- **LeetCode Problems**: Remove Linked List Elements, Delete Node in a Linked List
- **Interview Coding**: Simplifying edge cases under time pressure
- **Merging Lists**: Dummy node for result list construction
- **Partitioning Lists**: Stable partition around a pivot 

### References

- GitHub - Dummy Node Pattern for Linked Lists - https://github.com/davidchen0970/algorithm_note/blob/3012a5d5221ce3cc265aaaaa4fd9b46d7eec11d6/02_data_structures/07_linked_list.md
- GitHub - Sentinel Node Techniques - https://github.com/ash-iiiiish/master-repo-maths-cs-ai-compendium/blob/main/chapter%2014%20-%20data%20structures%20and%20algorithms/02.%20linked%20lists%2C%20stacks%2C%20and%20queues.md

---

## References (Consolidated)

- IIT Kharagpur - Dynamic Memory Allocation and Linked Lists - http://cse.iitkgp.ac.in/%7Epallab/course/2022/spring%202022/pds%20theory%202022/PPT/12%20-%20Linked%20Lists.pdf
- Carnegie Mellon University - Arrays and Linked Lists - https://www.cs.cmu.edu/~15110-n15/lectures/unit06-1-Arrays_handout.pdf
- Auckland COMPSCI 105 - Singly Linked Lists with Head and Tail - https://www.cs.auckland.ac.nz/courses/compsci105s1c/lectures/Bruce/19-LinkedList(3).pdf
- UC Irvine - Linked List Variations Analysis - https://canvas.eee.uci.edu/courses/57346/files/23640903/download
- UCSB CS16 - Linked Lists and Dynamic Memory Errors - https://ucsb-cs16.github.io/w20/lectures/CS16_Lecture14_ann.pdf
- Princeton COS 217 - Dynamaid Dangling Pointer Detection - https://www.cs.princeton.edu/%7Erdondero/iw/14Morrison.pdf
- Stack Overflow - Why Use Temporary Pointer Before delete - https://browse.library.kiwix.org/content/stackoverflow.com_en_all_nopic_2022-07/questions/46102696/pointers-in-a-linked-list-example
- York University - Singly Linked List Complexity Analysis - https://www.eecs.yorku.ca/%7Ewangcw/teaching/lectures/2025/W/EECS2101/slides/EECS2101_W25_All_Slides-4up.pdf
- METU - Singly Linked List Implementation - https://users.metu.edu.tr/degerli/DSA/DSA-2-LinkedLists.pdf
- McMaster University - Linked Lists Insertion and Deletion - http://www.cas.mcmaster.ca/~kahl/SE2S03/2005/SE2S-2005_Lists_4up.pdf
- GitHub - Dummy Node Pattern for Linked Lists - https://github.com/davidchen0970/algorithm_note/blob/3012a5d5221ce3cc265aaaaa4fd9b46d7eec11d6/02_data_structures/07_linked_list.md
- GitHub - Sentinel Node Techniques - https://github.com/ash-iiiiish/master-repo-maths-cs-ai-compendium/blob/main/chapter%2014%20-%20data%20structures%20and%20algorithms/02.%20linked%20lists%2C%20stacks%2C%20and%20queues.md
- Huawei Cloud - Singly Linked List Implementation - https://bbs.huaweicloud.com/blogs/d3bf1dac4d3044c29bb852066140d3fe
- Cambridge University Press - Linked List Overview - https://www.cambridge.org/highereducation/books/data-structures-and-algorithms-using-python/86C296AE2484AF8B34B0C35F909B1077/linked-list/38BB2574881F8F0848086C3413B8587A
- DSA.js Data Structures - Linear Data Structures Complexity - https://raw.githubusercontent.com/luastoned/dsa.js-data-structures-algorithms-javascript/cc1ccc0183ea5d4675815413b7d1d4cc14da348c/book/chapters/linear-data-structures-outro.adoc
- C.S.S. University - Singly Linked List Operations - https://bridge-library.ccsuniversity.ac.in/pdf/btech-cs/Linklist%20L-44,45,46,47,48-converted.pdf