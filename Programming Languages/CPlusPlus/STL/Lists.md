# C++ Lists — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
`std::list` and `std::forward_list` are sequence containers in the C++ Standard Library that store elements in non-contiguous, node-based structures, providing constant-time insertion and removal at any position.

**Technical Definition**
`std::list<T, Allocator>` is a doubly-linked list that meets the requirements of *Container*, *AllocatorAwareContainer*, *SequenceContainer*, and *ReversibleContainer*, supporting bidirectional iteration and O(1) insertion/erasure at any position given an iterator. `std::forward_list<T, Allocator>` (since C++11) is a singly-linked list that meets the requirements of *Container* (except for the `size` member function), *AllocatorAwareContainer*, and *SequenceContainer*, supporting forward-only iteration with even lower per-node memory overhead.

**Beginner-Friendly Explanation**
Think of a linked list as a chain of individual boxes (nodes). Each box holds an item and a pointer to the next box. In `std::forward_list`, each box points only forward; in `std::list`, each box points both forward and backward. This design makes it very fast to insert or remove boxes if you already have a “handle” to the right spot, but slow to find an item by its position because you have to walk the chain from the start.

### Key Characteristics

- **Node-based storage** – elements are not stored contiguously; each element lives in its own node.
- **O(1) insertion/removal** – given an iterator (or, for `forward_list`, an iterator to the *preceding* element), insertion and removal are constant time.
- **Iterator stability** – insertion, removal, and splicing do not invalidate iterators or references to *other* elements; only iterators to erased elements are invalidated.
- **No random access** – accessing the n-th element is O(n) via iteration.
- **Bidirectional vs. forward-only** – `list` supports `rbegin`/`rend` and `operator--`; `forward_list` supports only forward iteration.
- **Memory overhead** – `list` nodes carry two pointers (prev and next); `forward_list` nodes carry one pointer (next). On 64-bit systems, this is typically 16 bytes vs. 8 bytes of pointer overhead per node.
- **No O(1) `size()` in `forward_list`** – `forward_list` intentionally omits a `size()` member to remain as lightweight as possible; use `std::distance` if needed.

### Prerequisites

- Basic C++ syntax (variables, types, control flow).
- Understanding of pointers and dynamic memory.
- Familiarity with iterators and the range-based `for` loop.
- Awareness of Big-O notation and algorithmic complexity.
- Knowledge of the `std::vector` container as a baseline for comparison.

### Related Programming Areas

- **STL algorithms** – `std::find`, `std::for_each`, `std::copy` work with list iterators; `std::sort` does **not** (requires random access), so `list` provides its own member `sort()`.
- **Node-based data structures** – graphs (adjacency lists), LRU caches, hash table buckets.
- **Memory-constrained systems** – embedded systems where `forward_list`’s lower overhead matters.
- **Iterator stability requirements** – scenarios where references must remain valid across insertions/erasures.

### Core Concepts / Features

1. `std::list` (the container itself)
2. Doubly linked list (underlying structure)
3. List operations
4. List performance
5. List best practices
6. `std::forward_list`
7. Singly linked list (underlying structure)
8. Forward list best practices

---

## 1. `std::list`

### Definitions

**Core Definition**
`std::list` is a sequence container that supports constant-time insertion and removal of elements from anywhere in the container, usually implemented as a doubly-linked list.

**Technical Definition**
`std::list<T, Allocator>` stores elements in individually allocated nodes, each containing two pointers (to the previous and next nodes) and the element itself. It provides bidirectional iterators. Addition, removal, and moving of elements within the list or across several lists does not invalidate iterators or references; an iterator is invalidated only when the corresponding element is deleted.

**Beginner-Friendly Explanation**
`std::list` is the standard C++ doubly-linked list. It is the go-to container when you need to insert or remove items frequently in the middle of a sequence without disturbing the rest of the elements.

### Purposes

- To provide O(1) insertion and removal at any position (given an iterator).
- To guarantee iterator and reference stability across insertions, removals, and splices.
- To support bidirectional traversal.
- To offer list-specific algorithms (`sort`, `merge`, `splice`, `unique`, `remove`) that exploit the linked structure.
- To serve as a stable container for elements that are frequently reordered.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <list>
std::list<T> lst;                     // empty
std::list<T> lst(n);                  // n default-constructed elements
std::list<T> lst(n, value);           // n copies of value
std::list<T> lst = {a, b, c};        // initializer-list
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `T` | The type of elements stored. |
| `Allocator` | Optional allocator type (defaults to `std::allocator<T>`). |

**Member Function Complexity**

| Operation | Complexity |
|-----------|------------|
| `push_front`, `pop_front` | O(1) |
| `push_back`, `pop_back` | O(1) |
| `insert`, `erase` (given iterator) | O(1) |
| `splice` | O(1) |
| `sort` | O(n log n) |
| `merge` | O(n) (linear in the combined size) |
| `unique`, `remove` | O(n) |
| `size()` | O(1) (since C++11) |
| `begin`, `end`, `rbegin`, `rend` | O(1) |

**Syntax Rules**
- `list` does not provide `operator[]` or `at()`; positional access requires iteration.
- Insertion and erasure are expressed relative to an iterator: `insert(pos, value)` inserts **before** `pos`; `erase(pos)` removes the element at `pos`.
- Splicing (`splice`) moves elements between lists in O(1) without copying or allocating.
- `list` has its own member `sort()`, `merge()`, `unique()`, and `remove()`; these should be used instead of the generic algorithms where applicable.

**Constraints and Limitations**
- No random access; accessing the n-th element is O(n).
- Higher memory overhead per element (two pointers per node).
- Poor cache locality compared to contiguous containers like `vector`.
- `std::sort` cannot be used on a `list` because it requires random-access iterators.

### Annotated Code Examples

**Example 1: Basic `list` operations**

```cpp
#include <list>
#include <iostream>

int main() {
    std::list<int> lst = {1, 2, 3};

    lst.push_front(0);                     // O(1)
    lst.push_back(4);                      // O(1)

    auto it = lst.begin();
    std::advance(it, 2);                   // O(n) to reach position
    lst.insert(it, 99);                    // O(1) insertion

    for (int x : lst) std::cout << x << ' ';
    std::cout << '\n';
}
```

**Expected Output**
```
0 1 99 2 3 4
```

**Why this output?**  
`push_front(0)` adds `0` at the front; `push_back(4)` appends `4`. `advance` moves the iterator to the element `2` (third position), and `insert` places `99` before it.

**Example 2: Splice between lists**

```cpp
#include <list>
#include <iostream>

int main() {
    std::list<int> a = {1, 2, 3};
    std::list<int> b = {4, 5};

    a.splice(a.end(), b);                  // O(1): move all of b into a

    for (int x : a) std::cout << x << ' ';
}
```

**Expected Output**
```
1 2 3 4 5
```

**Why this output?**  
`splice` relinks the nodes of `b` into `a` without copying or allocating; `b` becomes empty.

**Example 3: Member `sort` and `unique`**

```cpp
#include <list>
#include <iostream>

int main() {
    std::list<int> lst = {3, 1, 2, 3, 1};
    lst.sort();                            // O(n log n)
    lst.unique();                          // O(n): removes consecutive duplicates

    for (int x : lst) std::cout << x << ' ';
}
```

**Expected Output**
```
1 2 3
```

**Why this output?**  
`sort` orders the elements; `unique` collapses adjacent duplicate values.

### Real-World Cases

- **LRU caches** – frequently accessed items are moved to the front; erasure from the middle is O(1).
- **Adjacency lists** – graph representations where edges are inserted and removed dynamically.
- **Playlist or task reordering** – `splice` and `sort` support efficient reordering.
- **Stable reference scenarios** – when other components hold references to elements that must remain valid across insertions.

---

## 2. Doubly Linked List (Underlying Structure)

### Definitions

**Core Definition**
A doubly linked list is a linear data structure in which each node contains a pointer to both the next and the previous node.

**Technical Definition**
Each node of a doubly linked list stores: (1) the element, (2) a pointer/reference to the previous node, and (3) a pointer/reference to the next node. The list maintains a head pointer (and, in `std::list`, typically a sentinel node) and supports O(1) insertion and removal at any node given a pointer to it, as well as O(1) traversal in both directions.

**Beginner-Friendly Explanation**
A doubly linked list is like a two-way chain: every link knows both the link before it and the link after it. This lets you walk forward or backward and makes insertion and deletion very fast if you already have a handle to the spot.

### Purposes

- To enable O(1) insertion and removal without shifting other elements.
- To support bidirectional traversal.
- To allow stable iterators and references across modifications.
- To serve as the basis for `std::list` and other node-based containers.

### Syntax Rules and Structure

**Conceptual Node Layout**
```
[prev | data | next] <-> [prev | data | next] <-> [prev | data | next]
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `prev` | Pointer to the previous node (or null/sentinel). |
| `data` | The stored element. |
| `next` | Pointer to the next node (or null/sentinel). |

**Syntax Rules**
- `std::list` does not expose the node structure directly; the doubly-linked nature is an implementation detail.
- The container provides bidirectional iterators that support `++` and `--`.
- Insertion before a given iterator is O(1); erasure of a given iterator is O(1).

**Constraints and Limitations**
- Two pointers per node increase memory overhead (typically 16 bytes on 64-bit systems for the pointers alone).
- Cache locality is poor because nodes are scattered in memory.
- No random access.

### Annotated Code Example (Conceptual)

```cpp
// Conceptual illustration (not actual std::list internals)
struct Node {
    int data;
    Node* prev;
    Node* next;
};

// Insertion before a given node:
// new_node->prev = node->prev;
// new_node->next = node;
// node->prev->next = new_node;
// node->prev = new_node;
```

**Expected Output**  
(No output; conceptual illustration.)

### Real-World Cases

- **Undo/redo stacks** – bidirectional traversal for navigating history.
- **Browser history** – back and forward navigation.
- **Music playlists** – skip forward or backward.

---

## 3. List Operations

### Definitions

**Core Definition**
List operations are the member functions and non-member functions that allow insertion, removal, access, and manipulation of `std::list` elements.

**Technical Definition**
`std::list` provides the standard sequence-container interface (`begin`/`end`, `size`, `empty`, `insert`, `erase`, `clear`, `push_front`/`pop_front`, `push_back`/`pop_back`) plus list-specific algorithms: `splice`, `sort`, `merge`, `unique`, and `remove`/`remove_if`.

**Beginner-Friendly Explanation**
`std::list` comes with a rich set of commands tailored for linked lists — not just adding and removing, but also splicing lists together, sorting in place, and merging two sorted lists efficiently.

### Key Operations

| Operation | Description | Complexity |
|-----------|-------------|------------|
| `push_front(value)` | Insert at the front | O(1) |
| `push_back(value)` | Insert at the back | O(1) |
| `pop_front()` | Remove the first element | O(1) |
| `pop_back()` | Remove the last element | O(1) |
| `insert(pos, value)` | Insert before `pos` | O(1) |
| `erase(pos)` | Remove element at `pos` | O(1) |
| `splice(pos, other)` | Move all elements of `other` before `pos` | O(1) |
| `splice(pos, other, it)` | Move a single element | O(1) |
| `splice(pos, other, first, last)` | Move a range | O(n) (for size update) |
| `sort()` | Sort in place | O(n log n) |
| `merge(other)` | Merge two sorted lists | O(n) |
| `unique()` | Remove consecutive duplicates | O(n) |
| `remove(value)` | Remove all elements equal to `value` | O(n) |
| `clear()` | Remove all elements | O(n) |
| `size()` | Number of elements | O(1) |
| `empty()` | Whether the list is empty | O(1) |

**Syntax Rules**
- `insert` and `erase` operate on iterators; they do not return iterators (unlike `vector::erase` which returns an iterator).
- `splice` does not copy or move elements; it relinks nodes.
- `sort`, `merge`, `unique`, and `remove` are member functions, not free algorithms; use them instead of `std::sort`, `std::merge`, etc..

**Constraints and Limitations**
- `splice` with a range may be O(n) if the implementation needs to update the size of the destination list.
- `remove` and `unique` invalidate only iterators to removed elements.

### Annotated Code Examples

**Example 1: Insert and erase**

```cpp
#include <list>
#include <iostream>

int main() {
    std::list<int> lst = {1, 2, 3, 4};

    auto it = lst.begin();
    std::advance(it, 2);                   // points to 3
    lst.insert(it, 99);                    // insert 99 before 3

    it = lst.begin();
    std::advance(it, 1);                   // points to 2
    lst.erase(it);                         // erase 2

    for (int x : lst) std::cout << x << ' ';
}
```

**Expected Output**
```
1 99 3 4
```

**Why this output?**  
`insert` places `99` before `3`; `erase` removes `2`. The remaining elements retain their relative order.

**Example 2: `merge` two sorted lists**

```cpp
#include <list>
#include <iostream>

int main() {
    std::list<int> a = {1, 3, 5};
    std::list<int> b = {2, 4, 6};

    a.merge(b);                            // both must be sorted

    for (int x : a) std::cout << x << ' ';
}
```

**Expected Output**
```
1 2 3 4 5 6
```

**Why this output?**  
`merge` interleaves the sorted elements of `b` into `a`, leaving `b` empty.

### Real-World Cases

- **LRU caches** – `splice` moves an item to the front; `erase` removes the least-recently used item.
- **Merge sort implementations** – `merge` provides the merging step.
- **Deduplication** – `sort` + `unique` removes duplicates efficiently.

---

## 4. List Performance

### Definitions

**Core Definition**
List performance refers to the time and space complexity characteristics of `std::list` operations.

**Technical Definition**
`std::list` provides O(1) insertion/erasure given an iterator, O(1) splice, O(n) traversal, and O(n log n) `sort`. Space overhead is two pointers per node. Iterators and references remain valid across insertions, erasures of other elements, and splices.

**Beginner-Friendly Explanation**
`std::list` is fast when you need to insert or remove in the middle, but slow when you need to find the n-th element or iterate through a large list because its nodes are scattered in memory.

### Complexity Summary

| Operation | Complexity | Notes |
|-----------|------------|-------|
| Random access | O(n) | No `operator[]`; must iterate |
| `push_front` / `pop_front` | O(1) | |
| `push_back` / `pop_back` | O(1) | |
| `insert` / `erase` (given iterator) | O(1) | |
| `splice` | O(1) | O(n) for a subrange if size update required |
| `sort` | O(n log n) | Member function, not `std::sort` |
| `merge` | O(n) | Linear in the combined size |
| `unique` / `remove` | O(n) | |
| `size` | O(1) | Since C++11 |

### Key Performance Factors

- **Cache behaviour** – nodes are individually allocated and may be scattered, causing frequent cache misses. In practice, `vector` often outperforms `list` even for middle insertions because of cache locality.
- **Memory overhead** – two pointers per node (16 bytes on 64-bit) plus allocator overhead.
- **Iterator stability** – valuable when references must remain valid, but comes at the cost of memory and cache performance.

### Real-World Cases

- **When iterator stability matters** – event systems where listeners hold references to nodes.
- **When splice is critical** – LRU caches, scheduler queues.
- **When to avoid `list`** – most general-purpose sequence processing, where `vector` is faster in practice.

---

## 5. List Best Practices

### Definitions

**Core Definition**
List best practices are guidelines for choosing and using `std::list` appropriately, avoiding common pitfalls, and achieving correct and efficient code.

**Technical Definition**
These practices are grounded in the C++ Core Guidelines (SL.con.2: “Prefer `vector` by default unless you have a reason to use a different container”) and community consensus: use `list` only when you specifically need iterator/reference stability or O(1) splice; otherwise prefer `vector`.

**Beginner-Friendly Explanation**
Best practices for `list` boil down to: don’t use it unless you really need it. For most tasks, `vector` is faster. Reach for `list` when you need stable references or fast splicing.

### Key Guidelines

| Guideline | Summary |
|-----------|---------|
| **SL.con.2** | Prefer `vector` by default; use `list` only with a specific reason. |
| **Use `list` for iterator stability** | When references/iterators must survive insertions and erasures. |
| **Use `splice` for O(1) transfers** | Moving elements between lists without copying. |
| **Use member `sort` / `merge` / `unique`** | Do not use `std::sort` on a list. |
| **Avoid `list` for large data sets** | Cache misses dominate; benchmark before committing. |
| **Consider `deque` as an alternative** | `deque` offers stable references (not iterators) and better cache behaviour. |
| **Benchmark** | The theoretical O(1) advantage is often outweighed by cache effects. |

### Annotated Code Example

```cpp
#include <list>
#include <iostream>

int main() {
    std::list<int> lst = {3, 1, 2};

    lst.sort();                            // correct: member sort
    // std::sort(lst.begin(), lst.end());  // ERROR: requires random access

    for (int x : lst) std::cout << x << ' ';
}
```

**Expected Output**
```
1 2 3
```

**Why this output?**  
The member `sort` is optimised for linked lists and works correctly; `std::sort` would fail to compile because `list` iterators are not random-access.

### Real-World Cases

- **Code reviews** – checking that `list` is not used where `vector` would suffice.
- **Performance profiling** – confirming that splice/iterator stability actually matters.
- **Library design** – exposing `list` only when the interface requires stable references.

---

## 6. `std::forward_list`

### Definitions

**Core Definition**
`std::forward_list` is a sequence container that supports fast insertion and removal of elements from anywhere in the container, implemented as a singly-linked list.

**Technical Definition**
`std::forward_list<T, Allocator>` (since C++11) stores elements in nodes that each contain one pointer (to the next node). It provides forward iterators only. Compared to `std::list`, it provides more space-efficient storage when bidirectional iteration is not needed. It meets the requirements of *Container* except for the `size` member function and the complexity of `operator==`.

**Beginner-Friendly Explanation**
`std::forward_list` is a one-way linked list. It uses less memory than `std::list` because each node stores only one pointer, but you can only walk forward, and it lacks some convenient features like `push_back` and `size()`.

### Purposes

- To provide a memory-efficient linked list.
- To support forward-only traversal with minimal overhead.
- To enable O(1) insertion and removal *after* a given position.
- To serve as the lightest-weight linked list in the standard library.
- To model data structures that naturally only need forward iteration (e.g., hash table buckets, stacks).

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <forward_list>
std::forward_list<T> fl;             // empty
std::forward_list<T> fl = {a, b, c};
```

**Component Breakdown**

| Feature | `std::forward_list` | `std::list` |
|---------|---------------------|-------------|
| Per-node overhead | 1 pointer (8 bytes on 64-bit) | 2 pointers (16 bytes) |
| Traversal | Forward only | Forward and backward |
| `push_front` | O(1) | O(1) |
| `push_back` | **Not available** | O(1) |
| `insert_after` / `erase_after` | O(1) | N/A (uses `insert`/`erase`) |
| `size()` | **O(n)** (not stored) | O(1) |
| `before_begin()` | Yes | N/A |
| Reverse iteration | **No** | Yes |

**Syntax Rules**
- `forward_list` uses `insert_after(pos, value)` and `erase_after(pos)` instead of `insert` and `erase`, because a singly-linked node cannot access its predecessor.
- `before_begin()` returns an iterator to the position *before* the first element, enabling insertion/erasure at the front with the `_after` API.
- There is no `size()` member; use `std::distance(fl.begin(), fl.end())` if the size is needed, but this is O(n).
- There is no `push_back` or `back()`.

**Constraints and Limitations**
- Forward-only iteration.
- No direct access to the last element.
- `size()` is not provided for efficiency reasons.
- `splice_after` moves elements after a given position.

### Annotated Code Examples

**Example 1: `insert_after` and `erase_after`**

```cpp
#include <forward_list>
#include <iostream>

int main() {
    std::forward_list<int> fl = {1, 2, 3};

    auto it = fl.begin();                  // points to 1
    fl.insert_after(it, 99);              // insert 99 after 1
    fl.insert_after(fl.before_begin(), 0); // insert 0 at the front

    fl.erase_after(fl.begin());            // erase the element after 0 (which is 1)

    for (int x : fl) std::cout << x << ' ';
}
```

**Expected Output**
```
0 99 2 3
```

**Why this output?**  
`insert_after` inserts after the given iterator; `before_begin()` allows insertion at the very front; `erase_after` removes the element following the given iterator.

**Example 2: `forward_list` sort and unique**

```cpp
#include <forward_list>
#include <iostream>

int main() {
    std::forward_list<int> fl = {3, 1, 2, 3, 1};
    fl.sort();
    fl.unique();

    for (int x : fl) std::cout << x << ' ';
}
```

**Expected Output**
```
1 2 3
```

**Why this output?**  
`sort` orders the elements in O(n log n); `unique` removes consecutive duplicates in O(n).

### Real-World Cases

- **Hash table bucket chains** – separate chaining with minimal memory overhead.
- **Embedded systems** – tight memory budgets where saving one pointer per node matters.
- **Stack implementations** – `push_front` and forward iteration are sufficient.
- **Intrusive lists** – when nodes are embedded in the element itself.

---

## 7. Singly Linked List (Underlying Structure)

### Definitions

**Core Definition**
A singly linked list is a linear data structure in which each node contains a pointer only to the next node.

**Technical Definition**
Each node stores: (1) the element, and (2) a pointer/reference to the next node. The list maintains a head pointer (and optionally a sentinel). Traversal is forward-only. Insertion and removal require a pointer to the *preceding* node (hence `insert_after` / `erase_after`).

**Beginner-Friendly Explanation**
A singly linked list is a one-way chain: each link points only to the next link. You can’t go backward, but each link uses less memory than in a doubly linked list.

### Purposes

- To minimise memory overhead per element.
- To support forward-only traversal efficiently.
- To enable O(1) insertion and removal after a known position.
- To serve as the basis for `std::forward_list` and many low-level data structures.

### Syntax Rules and Structure

**Conceptual Node Layout**
```
[data | next] -> [data | next] -> [data | next] -> nullptr
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `data` | The stored element. |
| `next` | Pointer to the next node (or null). |

**Syntax Rules**
- `forward_list` exposes only forward iterators.
- `before_begin()` provides a sentinel before the first element for front operations.
- `insert_after` and `erase_after` operate on the node *following* the given iterator.

**Constraints and Limitations**
- No backward traversal.
- No O(1) `size()`.
- Cannot insert *before* a given node without a reference to the predecessor.

### Annotated Code Example (Conceptual)

```cpp
// Conceptual illustration (not actual std::forward_list internals)
struct Node {
    int data;
    Node* next;
};

// Insert after `node`:
// Node* new_node = new Node{value, node->next};
// node->next = new_node;
```

**Expected Output**  
(No output; conceptual illustration.)

### Real-World Cases

- **Hash table buckets** – separate chaining with minimal per-node cost.
- **Memory-constrained embedded systems** – saving one pointer per node matters.
- **Forward-only pipelines** – processing streams where backward access is unnecessary.

---

## 8. Forward List Best Practices

### Definitions

**Core Definition**
Forward list best practices are guidelines for choosing and using `std::forward_list` appropriately, recognising its trade-offs and avoiding common pitfalls.

**Technical Definition**
These practices emphasise using `forward_list` only when the memory savings of a singly-linked list are important and backward traversal is not required. When in doubt, prefer `std::list` or `std::vector` depending on the operation profile.

**Beginner-Friendly Explanation**
Use `forward_list` when you need to save memory and only ever move forward. If you need to go backward, get the size quickly, or add to the end, use `std::list` instead.

### Key Guidelines

| Guideline | Summary |
|-----------|---------|
| **Use when memory matters** | `forward_list` saves ~50% of pointer overhead vs. `list`. |
| **Use when forward-only is sufficient** | No `rbegin`/`rend`; no `push_back`. |
| **Use `before_begin()` for front operations** | The canonical way to insert/erase at the front. |
| **Avoid if `size()` is needed frequently** | `size()` is O(n); store the size separately if necessary. |
| **Prefer `list` if any missing feature is needed** | Don’t fight the API restrictions. |
| **Benchmark** | For small element types, the memory savings may be outweighed by other factors. |
| **Remember: `std::vector` is usually faster** | Linked lists are rarely the best default choice. |

### Annotated Code Example

```cpp
#include <forward_list>
#include <iostream>

int main() {
    std::forward_list<int> fl = {3, 1, 2};

    fl.sort();                             // member sort
    fl.unique();                           // remove duplicates

    // Prepend using before_begin()
    fl.insert_after(fl.before_begin(), 0);

    for (int x : fl) std::cout << x << ' ';
}
```

**Expected Output**
```
0 1 2 3
```

**Why this output?**  
`sort` and `unique` order and deduplicate the elements; `insert_after(before_begin(), 0)` prepends `0` using the `_after` API.

### Real-World Cases

- **Embedded firmware** – saving 8 bytes per node across thousands of nodes.
- **Hash table implementations** – bucket chains with minimal overhead.
- **Forward-only parsers** – token streams processed sequentially.

---

## References

- `std::list` — cppreference – https://en.cppreference.com/w/cpp/container/list
- `std::forward_list` — cppreference – https://en.cppreference.com/w/cpp/container/forward_list
- Standard library header `<list>` — cppreference – https://en.cppreference.com/w/cpp/header/list
- Standard library header `<forward_list>` — cppreference – https://en.cppreference.com/w/cpp/header/forward_list
- `std::list::splice` — cppreference – https://en.cppreference.com/w/cpp/container/list/splice
- `std::forward_list::insert_after` — cppreference – https://en.cppreference.com/w/cpp/container/forward_list/insert_after
- `std::forward_list::erase_after` — cppreference – https://en.cppreference.com/w/cpp/container/forward_list/erase_after
- `std::forward_list::before_begin` — cppreference – https://en.cppreference.com/w/cpp/container/forward_list/before_begin
- C++ Core Guidelines — SL.con: Containers – https://cpp-core-guidelines-docs.vercel.app/stdlib
- Know when to use `std::forward_list` over `std::list` — Dr-Sergey/learn_cpp – https://raw.githubusercontent.com/Dr-Sergey/learn_cpp/refs/heads/main/07_Standard_Library_Containers/Know_when_to_use_stdforward_list_over_stdlist.md
- Use `std::list::splice` for O(1) node transfers — Dr-Sergey/learn_cpp – https://github.com/Dr-Sergey/learn_cpp/commit/41c7db3f55f5932ad8d2c9292542cc657a548c0b
- Iterator invalidation rules for standard containers — Dr-Sergey/learn_cpp – https://raw.githubusercontent.com/Dr-Sergey/learn_cpp/refs/heads/main/07_Standard_Library_Containers/Understand_iterator_invalidation_rules_for_all_standard_containers.md
- Time complexity guarantees of standard containers — Dr-Sergey/learn_cpp – https://raw.githubusercontent.com/Dr-Sergey/learn_cpp/refs/heads/main/07_Standard_Library_Containers/Know_the_time_complexity_guarantees_of_all_standard_containers.md
- C++ Working Draft (N4950) — Containers library – https://eel.is/c++draft/containers

---

This cheat sheet provides a comprehensive, structured reference for C++ linked lists, covering `std::list`, doubly linked lists, list operations and performance, `std::forward_list`, singly linked lists, and best practices, with annotated code examples, expected outputs, real-world cases, and authoritative references.