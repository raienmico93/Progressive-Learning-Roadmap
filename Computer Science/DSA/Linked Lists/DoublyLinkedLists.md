# DSA Doubly Linked Lists

## Topic Overview

### Definitions

**Core Definition**
A doubly linked list is a linear data structure where each node contains a data element and two references: one pointing to the previous node and one pointing to the next node, enabling bidirectional traversal .

**Technical Definition**
A doubly linked list is a concrete data structure composed of dynamically allocated nodes, where each node contains a `data` field, a `prev` pointer referencing the preceding node (or NULL), and a `next` pointer referencing the subsequent node (or NULL). The list maintains a `head` reference to the first node and optionally a `tail` reference to the last node. The `prev` of the head is NULL, and the `next` of the tail is NULL .

**Beginner-Friendly Explanation**
Think of a doubly linked list as a two-way street where each house (node) knows not only who lives next door (the next node) but also who lives on the other side (the previous node). In a singly linked list, you can only walk forward. In a doubly linked list, you can walk both forward and backward, which makes certain operations—like deleting a node or implementing undo/redo—much easier .

### Key Characteristics

- **Bidirectional Traversal**: Each node knows both its predecessor and successor, allowing movement in both directions 
- **Two Pointers per Node**: The `prev` and `next` references add memory overhead compared to singly linked lists 
- **O(1) Deletion with Node Reference**: If you have a direct reference to a node, you can delete it in O(1) time because you can access its predecessor via the `prev` pointer 
- **Symmetric Operations**: Insertion and deletion at both ends are equally efficient, making doubly linked lists ideal for deques 
- **No Direct Indexing**: Accessing the i-th element still requires O(n) traversal from either end 

### Prerequisites

- Understanding of singly linked lists and node structures
- Familiarity with pointers/references and dynamic memory allocation
- Awareness of NULL/None semantics
- Knowledge of basic pointer manipulation (linking and unlinking)

### Related Programming Areas

- **Deque Implementation**: Double-ended queues benefit from O(1) operations at both ends 
- **LRU Caches**: Combining a hash map with a doubly linked list enables O(1) access and eviction 
- **Text Editors**: Gap buffers use doubly linked lists for efficient cursor movement 
- **Undo/Redo Systems**: Bidirectional navigation is natural for history management 
- **Memory Management**: Operating systems use doubly linked lists for free/allocated block tracking 

### Core Concepts / Features

| Concept | Description |
|---------|-------------|
| Node Structure | Data + prev pointer + next pointer |
| Bidirectional Traversal | Forward via `next`, backward via `prev` |
| Head/Tail Sentinels | NULL endpoints or anchor nodes |
| O(1) Insert/Delete with Reference | Pointer rewiring without traversal |
| Memory Overhead | Extra pointer per node vs. singly linked |

---

## 1. Previous References

### Definitions

**Core Definition**
The previous reference (`prev` pointer) in a node holds the memory address of the node that comes immediately before it in the list. For the head node, `prev` is NULL (or None), indicating the beginning of the list .

**Technical Definition**
Each node contains a `prev` field of the same node type, forming a reverse linkage. The invariant `node.next.prev == node` holds for all internal nodes, ensuring the list's structural integrity in both directions .

**Beginner-Friendly Explanation**
The `prev` pointer is like having a bookmark that not only shows where you are but also where you just came from. This lets you walk backward through the list without starting over from the beginning .

### Purposes (All begin with "To")

- **To** enable backward traversal from any node without restarting from the head
- **To** facilitate O(1) deletion of a node when a direct reference to it is available
- **To** support symmetric operations at both ends of the list
- **To** maintain structural integrity for bidirectional navigation

### Syntax Rules and Structure

**General Syntax (Node Definition)**
```c
typedef struct DLLNode {
    void* data;              // Data field
    struct DLLNode* prev;    // Pointer to previous node
    struct DLLNode* next;    // Pointer to next node
} DLLNode;
```

```java
class Node<T> {
    T data;
    Node<T> prev;  // Previous reference
    Node<T> next;  // Next reference
    
    Node(T data, Node<T> prev, Node<T> next) {
        this.data = data;
        this.prev = prev;
        this.next = next;
    }
}
```

**Component Breakdown**

| Component | Description | Value |
|-----------|-------------|-------|
| `prev` | Reference to predecessor | Address of previous Node or NULL |
| Head's `prev` | Sentinel value | NULL (marks beginning) |
| Symmetry invariant | `node.next.prev == node` | Must hold for all internal nodes |

**Syntax Rules**

- The `prev` pointer of the head node is always NULL 
- When inserting a node, update both `prev` and `next` pointers of affected nodes
- The symmetry invariant must be maintained: `node.next.prev == node` and `node.prev.next == node` 

**Constraints and Limitations**

- Adds one pointer of memory overhead per node compared to singly linked lists
- Must be carefully maintained during insertions and deletions to avoid list corruption
- Languages with garbage collection handle reference cycles automatically; manual memory management requires explicit cleanup

### Annotated Code Examples

**Example 1: Creating Nodes with Previous References (Python)**
```python
class Node:
    def __init__(self, data):
        self.data = data
        self.prev = None  # Previous reference
        self.next = None  # Next reference

# Create three nodes
n1 = Node(10)
n2 = Node(20)
n3 = Node(30)

# Link them forward
n1.next = n2
n2.next = n3

# Link them backward (previous references)
n2.prev = n1
n3.prev = n2

# Traverse forward
print("Forward:")
cur = n1
while cur:
    print(cur.data, end=" -> ")
    cur = cur.next
print("NULL")

# Traverse backward using prev references
print("Backward:")
cur = n3
while cur:
    print(cur.data, end=" -> ")
    cur = cur.prev
print("NULL")
```

**Expected Output:**
```
Forward:
10 -> 20 -> 30 -> NULL
Backward:
30 -> 20 -> 10 -> NULL
```

**Why This Output Occurs**: The `next` pointers create forward links (10→20→30), while the `prev` pointers create backward links (30→20→10). The head's `prev` is NULL, and the tail's `next` is NULL, marking both boundaries .

### Real-World Cases

- **Music Players**: "Previous Track" button requires backward navigation 
- **Browser History**: Back button traverses history in reverse 
- **Text Editors**: Cursor movement in both directions 

### References

- CMU 15-123 - Doubly Linked Lists - https://www.cs.cmu.edu/~guna/15-123S11/Lectures/Lecture12.pdf

---

## 2. Next References

### Definitions

**Core Definition**
The next reference (`next` pointer) in a node holds the memory address of the node that comes immediately after it in the list. For the tail node, `next` is NULL, indicating the end of the list .

**Technical Definition**
The `next` field is identical in purpose and behavior to the `next` pointer in a singly linked list, forming the forward linkage of the doubly linked list. Together with `prev`, it enables full bidirectional traversal .

**Beginner-Friendly Explanation**
The `next` pointer is the same as in a singly linked list—it tells you where the next item is. The difference is that now you also have a `prev` pointer going the other way .

### Purposes (All begin with "To")

- **To** enable forward traversal from head to tail
- **To** provide the primary access pattern for most list operations
- **To** support insertion at the tail (with tail pointer)
- **To** maintain the sequential ordering of elements

### Syntax Rules and Structure

**General Syntax**
```c
// Next pointer access
Node* nextNode = current->next;

// Link new node at end
tail->next = newNode;
newNode->prev = tail;
newNode->next = NULL;
tail = newNode;
```

**Component Breakdown**

| Component | Description | Value |
|-----------|-------------|-------|
| `next` | Reference to successor | Address of next Node or NULL |
| Tail's `next` | Sentinel value | NULL (marks end) |
| Forward traversal | Follow `next` from head | O(n) to reach any position |

**Syntax Rules**

- The `next` pointer of the tail node is always NULL 
- Insertion at tail requires updating `tail->next`, `newNode->prev`, `newNode->next`, and `tail` 
- The forward traversal pattern is identical to singly linked lists

**Constraints and Limitations**

- Without a tail pointer, appending requires O(n) traversal
- The `next` pointer alone cannot support backward navigation
- Must be updated alongside `prev` to maintain symmetry

### Annotated Code Examples

**Example 1: Appending with Next References (Java)**
```java
public class DoublyLinkedList {
    Node head;
    Node tail;
    
    void append(int data) {
        Node newNode = new Node(data, null, null);
        
        if (head == null) {
            // Empty list: new node is both head and tail
            head = tail = newNode;
        } else {
            // Link new node after current tail
            tail.next = newNode;     // Old tail points to new
            newNode.prev = tail;     // New node points back
            tail = newNode;          // Update tail reference
        }
    }
    
    public static void main(String[] args) {
        DoublyLinkedList list = new DoublyLinkedList();
        list.append(10);
        list.append(20);
        list.append(30);
        
        // Print forward
        Node cur = list.head;
        while (cur != null) {
            System.out.print(cur.data + " -> ");
            cur = cur.next;
        }
        System.out.println("NULL");
    }
}
```

**Expected Output:**
```
10 -> 20 -> 30 -> NULL
```

**Why This Output Occurs**: Each `append` creates a new node and links it after the current tail. The `next` pointer of the old tail is updated to reference the new node, and the `tail` reference moves to the new node .

### Real-World Cases

- **Queue Implementation**: Enqueue at tail via `next` 
- **Playlist**: "Next Track" button follows `next` pointers 
- **Linked List Iteration**: Standard forward iteration pattern

### References

- William Fiset DoublyLinkedList Implementation - https://raw.githubusercontent.com/williamfiset/Algorithms/refs/heads/master/src/main/java/com/williamfiset/algorithms/datastructures/linkedlist/DoublyLinkedList.java

---

## 3. Bidirectional Traversal

### Definitions

**Core Definition**
Bidirectional traversal is the ability to navigate a doubly linked list in both forward (head-to-tail) and backward (tail-to-head) directions using the `next` and `prev` pointers respectively .

**Technical Definition**
Forward traversal follows `node.next` starting from head until NULL. Backward traversal follows `node.prev` starting from tail until NULL. Both traversals visit all n nodes in O(n) time .

**Beginner-Friendly Explanation**
Imagine reading a book that you can flip through both forwards and backwards. You can start at the beginning and read to the end, or start at the end and read to the beginning—both are equally easy .

### Purposes (All begin with "To")

- **To** enable navigation in either direction without restarting from an endpoint
- **To** support algorithms that require bidirectional scanning (e.g., palindrome checking)
- **To** facilitate symmetric operations at both ends of the list
- **To** improve efficiency when the target is closer to one end than the other

### Syntax Rules and Structure

**General Syntax (Forward Traversal)**
```
cur = head
while cur != NULL:
    process(cur.data)
    cur = cur.next
```

**General Syntax (Backward Traversal)**
```
cur = tail
while cur != NULL:
    process(cur.data)
    cur = cur.prev
```

**Component Breakdown**

| Direction | Start | Pointer Used | Termination |
|-----------|-------|--------------|-------------|
| Forward | head | `next` | `cur == NULL` |
| Backward | tail | `prev` | `cur == NULL` |

**Syntax Rules**

- Forward traversal starts at `head` and follows `next` 
- Backward traversal starts at `tail` and follows `prev` 
- Both traversals are O(n) and visit each node exactly once
- The `prev` of head and `next` of tail are both NULL, serving as sentinels 

**Constraints and Limitations**

- Backward traversal requires a `tail` reference (or O(n) to find the tail first)
- Not all operations benefit from bidirectional access; searching still requires O(n) in the worst case
- Memory overhead for the extra pointer must be justified by the use case

### Annotated Code Examples

**Example 1: Bidirectional Traversal (Python)**
```python
class Node:
    def __init__(self, data):
        self.data = data
        self.prev = None
        self.next = None

# Build list: 10 <-> 20 <-> 30
head = Node(10)
middle = Node(20)
tail = Node(30)

head.next = middle
middle.prev = head
middle.next = tail
tail.prev = middle

# Forward traversal
print("Forward traversal:")
cur = head
while cur:
    print(cur.data, end=" ")
    cur = cur.next
print()

# Backward traversal
print("Backward traversal:")
cur = tail
while cur:
    print(cur.data, end=" ")
    cur = cur.prev
print()
```

**Expected Output:**
```
Forward traversal:
10 20 30 
Backward traversal:
30 20 10 
```

**Why This Output Occurs**: Forward traversal follows `next` from head (10→20→30). Backward traversal follows `prev` from tail (30→20→10). The bidirectional pointers enable both directions without additional data structures .

### Real-World Cases

- **Music Player**: Skip forward/backward through tracks 
- **Browser History**: Forward and back navigation 
- **Subway Line Navigation**: Moving between stations in both directions 
- **Deque**: Double-ended queue operations 

### References

- CMU 15-123 - Doubly Linked Lists - https://www.cs.cmu.edu/~guna/15-123S11/Lectures/Lecture12.pdf

---

## 4. Insertion

### Definitions

**Core Definition**
Insertion in a doubly linked list adds a new node at a specified position—at the head, at the tail, or between two existing nodes—by updating the `prev` and `next` pointers of the new node and its neighbors .

**Technical Definition**
Insertion requires four pointer updates: the new node's `prev` and `next` must be set, the previous node's `next` must point to the new node, and the next node's `prev` must point back to the new node. At boundaries, NULL or head/tail references replace the missing neighbors .

**Beginner-Friendly Explanation**
Inserting a node is like splicing a new car into a train. You connect the new car to the car in front and the car behind, and you update those cars to point to the new one. In a doubly linked list, both the forward and backward connections need to be updated .

### Purposes (All begin with "To")

- **To** add elements to the list while maintaining bidirectional links
- **To** support O(1) insertion at the head and (with tail pointer) at the tail 
- **To** enable insertion at arbitrary positions after traversal
- **To** maintain the symmetry invariant during modification

### Syntax Rules and Structure

**General Syntax (Insert at Head)**
```
newNode.next = head
newNode.prev = NULL
if head != NULL:
    head.prev = newNode
head = newNode
if tail == NULL:    # Was empty
    tail = newNode
```

**General Syntax (Insert at Tail)**
```
newNode.next = NULL
newNode.prev = tail
if tail != NULL:
    tail.next = newNode
tail = newNode
if head == NULL:    # Was empty
    head = newNode
```

**General Syntax (Insert After Given Node)**
```
newNode.next = givenNode.next
newNode.prev = givenNode
if givenNode.next != NULL:
    givenNode.next.prev = newNode
givenNode.next = newNode
```

**Component Breakdown**

| Position | Pointer Updates | Complexity |
|----------|-----------------|------------|
| Head | 2-3 updates | O(1) |
| Tail | 2-3 updates | O(1) with tail pointer |
| After node | 3-4 updates | O(1) with node reference |

**Syntax Rules**

- Always update `prev` and `next` of the new node first, then update neighbors 
- When inserting at boundaries, handle NULL neighbors appropriately
- For empty list insertion, both `head` and `tail` must be set to the new node 
- The symmetry invariant must hold after insertion: `newNode.next.prev == newNode` and `newNode.prev.next == newNode` 

**Constraints and Limitations**

- Insertion at arbitrary position requires O(n) traversal to find the position
- Pointer updates must be ordered carefully to avoid losing references
- Memory must be allocated for the new node (C/C++) or the garbage collector handles it (Java/Python)

### Annotated Code Examples

**Example 1: Insertion at Head and Tail (Java)**
```java
class DoublyLinkedList {
    Node head;
    Node tail;
    
    // O(1) insert at head
    void addFirst(int data) {
        Node newNode = new Node(data);
        newNode.next = head;
        newNode.prev = null;
        
        if (head != null) {
            head.prev = newNode;
        }
        head = newNode;
        
        if (tail == null) {  // Was empty
            tail = newNode;
        }
    }
    
    // O(1) insert at tail (with tail pointer)
    void addLast(int data) {
        Node newNode = new Node(data);
        newNode.next = null;
        newNode.prev = tail;
        
        if (tail != null) {
            tail.next = newNode;
        }
        tail = newNode;
        
        if (head == null) {  // Was empty
            head = newNode;
        }
    }
    
    public static void main(String[] args) {
        DoublyLinkedList list = new DoublyLinkedList();
        list.addLast(20);   // [20]
        list.addFirst(10);  // [10, 20]
        list.addLast(30);   // [10, 20, 30]
        list.addFirst(5);   // [5, 10, 20, 30]
        
        // Print forward
        Node cur = list.head;
        while (cur != null) {
            System.out.print(cur.data + " <-> ");
            cur = cur.next;
        }
        System.out.println("NULL");
    }
}
```

**Expected Output:**
```
5 <-> 10 <-> 20 <-> 30 <-> NULL
```

**Why This Output Occurs**: `addLast` appends at the tail, updating `tail.next` and the new node's `prev`. `addFirst` prepends at the head, updating `head.prev` and the new node's `next`. Both operations are O(1) because they don't require traversal .

### Real-World Cases

- **Deque**: Insert at both ends in O(1) 
- **LRU Cache**: Move accessed items to front 
- **Music Playlist**: Add songs at any position

### References

- William Fiset DoublyLinkedList - https://raw.githubusercontent.com/williamfiset/Algorithms/refs/heads/master/src/main/java/com/williamfiset/algorithms/datastructures/linkedlist/DoublyLinkedList.java

---

## 5. Deletion

### Definitions

**Core Definition**
Deletion in a doubly linked list removes a node by updating its neighbors' pointers to bypass it and deallocating its memory. The `prev` pointer makes deletion O(1) when a direct reference to the target node is available .

**Technical Definition**
To delete a node `target`: set `target.prev.next = target.next` and `target.next.prev = target.prev`. Handle boundary cases where `target` is the head or tail. Free/deallocate `target` .

**Beginner-Friendly Explanation**
Deleting a node is like removing a car from a train. You connect the car in front to the car behind, and the car behind to the car in front. Then you remove the middle car. The `prev` pointer is what makes this easy—you don't need to search for the previous car .

### Purposes (All begin with "To")

- **To** remove elements from the list while maintaining bidirectional links
- **To** achieve O(1) deletion when a node reference is available 
- **To** support deletion at both ends and arbitrary positions
- **To** maintain the symmetry invariant during removal

### Syntax Rules and Structure

**General Syntax (Delete Given Node)**
```
if target.prev != NULL:
    target.prev.next = target.next
else:
    head = target.next  # target was head

if target.next != NULL:
    target.next.prev = target.prev
else:
    tail = target.prev  # target was tail

free(target)  # Deallocate
```

**General Syntax (Delete Head)**
```
if head == NULL: return
old = head
head = head.next
if head != NULL:
    head.prev = NULL
else:
    tail = NULL  # List became empty
free(old)
```

**General Syntax (Delete Tail)**
```
if tail == NULL: return
old = tail
tail = tail.prev
if tail != NULL:
    tail.next = NULL
else:
    head = NULL  # List became empty
free(old)
```

**Component Breakdown**

| Position | Pointer Updates | Complexity | Note |
|----------|-----------------|------------|------|
| Given node | 2-4 updates | O(1) | Requires node reference  |
| Head | 1-2 updates | O(1) | Update new head's prev |
| Tail | 1-2 updates | O(1) | Update new tail's next |

**Syntax Rules**

- Update neighbors before freeing the target node 
- Handle head/tail boundary cases explicitly
- When deleting the only node, set both `head` and `tail` to NULL 
- O(1) deletion of an arbitrary node requires having a direct reference to it; finding the node by value still requires O(n) search 

**Constraints and Limitations**

- Deletion by value requires O(n) search (unless a node reference is already available)
- In C/C++, must free memory to avoid leaks; Java/Python rely on garbage collection
- Pointer updates must be ordered to avoid losing references to the rest of the list

### Annotated Code Examples

**Example 1: O(1) Deletion with Node Reference (Java)**
```java
class DoublyLinkedList {
    Node head;
    Node tail;
    
    // O(1) deletion given a reference to the node
    void deleteNode(Node target) {
        if (target.prev != null) {
            target.prev.next = target.next;
        } else {
            head = target.next;  // target was head
        }
        
        if (target.next != null) {
            target.next.prev = target.prev;
        } else {
            tail = target.prev;  // target was tail
        }
        
        // Clear references (help GC)
        target.prev = null;
        target.next = null;
    }
    
    public static void main(String[] args) {
        DoublyLinkedList list = new DoublyLinkedList();
        list.addLast(10);
        list.addLast(20);
        list.addLast(30);
        list.addLast(40);
        
        // Get reference to node with value 20
        Node target = list.head.next;  // O(n) to find, but we have it
        
        // Delete in O(1)
        list.deleteNode(target);
        
        // Print
        Node cur = list.head;
        while (cur != null) {
            System.out.print(cur.data + " <-> ");
            cur = cur.next;
        }
        System.out.println("NULL");
    }
}
```

**Expected Output:**
```
10 <-> 30 <-> 40 <-> NULL
```

**Why This Output Occurs**: `deleteNode` updates `target.prev.next` (node 10's next) to point to `target.next` (node 30), and `target.next.prev` (node 30's prev) to point to `target.prev` (node 10). Node 20 is bypassed and removed .

### Real-World Cases

- **LRU Cache**: Evict least recently used item in O(1) 
- **Undo/Redo**: Remove history entries 
- **Music Playlist**: Remove songs from playlist 
- **Free List Management**: OS removes memory blocks from free list 

### References

- William Fiset DoublyLinkedList - https://raw.githubusercontent.com/williamfiset/Algorithms/refs/heads/master/src/main/java/com/williamfiset/algorithms/datastructures/linkedlist/DoublyLinkedList.java
- GitHub CS-Fundamentals Doubly Linked List - https://raw.githubusercontent.com/manishkumar8312/CS-Fundamentals/refs/heads/main/Data%20Structure%20and%20Algorithms/Ch%2004%20-%20Linked%20List.md
- SFU CMPT 307 Complexity Table - https://www2.cs.sfu.ca/CourseCentral/307/manuelz/a/3/a3s.pdf

---

## References (Consolidated)

- CMU 15-123 - Doubly Linked Lists - https://www.cs.cmu.edu/~guna/15-123S11/Lectures/Lecture12.pdf
- William Fiset DoublyLinkedList Implementation - https://raw.githubusercontent.com/williamfiset/Algorithms/refs/heads/master/src/main/java/com/williamfiset/algorithms/datastructures/linkedlist/DoublyLinkedList.java
- GitHub CS-Fundamentals Doubly Linked List - https://raw.githubusercontent.com/manishkumar8312/CS-Fundamentals/refs/heads/main/Data%20Structure%20and%20Algorithms/Ch%2004%20-%20Linked%20List.md
- SFU CMPT 307 Complexity Table - https://www2.cs.sfu.ca/CourseCentral/307/manuelz/a/3/a3s.pdf
- Dart DoublyLinkedList Package (LRU Cache Example) - https://pub.dev/packages/doubly_linked_list/versions/1.0.0/example
- Data Structures and Algorithms in Java (Internet Archive) - https://archive.org/stream/datastructuresandalgorithmsinjava2ndedition/Data_Structures_and_Algorithms_in_Java_2nd_Edition_djvu.txt
- Python.org Mailman - Double-Linked Lists in Python - https://mail.python.org/archives/list/python-list@python.org/thread/CGCWKZGTHLUETFASLRIHTHFYG3O34QXX/
- French Education - Listes doublement chainées - https://pratique.forge.apps.education.fr/lineaire/6-liste/5-doublement/
- CNRS POGL - Listes doublement chaînées - https://usr.lmf.cnrs.fr/~blsk/Archives/POGL/TDTP/TP5-ListesDoublementChainees.pdf
- Bohrium - Doubly-Linked List Applications - https://scipedia.bohrium.com/en/sciencepedia/feynman/keyword/doubly_linked_list_2
- Wikipedia - Two-way linked list - https://en.m.wikipedia.org/wiki/Two-way_list
- Stack Overflow - Why Doubly Linked List for Music Player - https://browse.library.kiwix.org/content/stackoverflow.com_en_all_2023-11/questions/68698823/why-is-a-doubly-linked-list-used-in-a-music-player-with-next-and-prev-butt
- Educative.io Doubly Linked Lists in Python - https://github.com/zzhutianyu/educative.io_courses/blob/f66baed0c9f27a612b69b9ac5b1e1add224f58d0/Data%20Structures%20and%20Algorithms%20in%20Python%20-%20Learn%20Interactively/33_Append_and_Prepend.pdf
- Garuda - Doubly Linked List Research (Indonesian) - https://garuda.kemdiktisaintek.go.id/author/view/3839455?page=1&jid=32479&jname=jriin+jurnal+riset+informatika+dan+inovasi
- Python.org Tutor - Linked Lists - https://mail.python.org/pipermail/tutor/2003-August/024968.html
- Archive.org - Programming in C (Polynomial Representation) - https://www.archive.org/download/AlgorithmAndComputation_201808/%5BPradip_Dey%2C_Manas_Ghosh%5D_Programming_in_C%28b-ok.org%29.pdf
- GitHub - Data Structures and Algorithms Made Easy (Memory-Efficient DLL) - https://raw.githubusercontent.com/careermonk/data-structures-and-algorithms-made-easy/master/Sample%20Chapter%20Data%20Structures%20and%20Algorithms%20Made%20Easy%20Fifth%20Edition.pdf