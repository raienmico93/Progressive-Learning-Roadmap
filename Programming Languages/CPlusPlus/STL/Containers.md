# C++ Containers — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A C++ container is a class template that manages a collection of objects (elements) and provides member functions for accessing, inserting, and erasing those elements.

**Technical Definition**
The C++ Standard Library defines a hierarchy of container class templates, each satisfying a set of named requirements (Container, SequenceContainer, AssociativeContainer, UnorderedAssociativeContainer, etc.). Containers own their elements and manage the storage allocated for them, providing iterators for traversal and member functions for element access and modification.

**Beginner-Friendly Explanation**
A container is like a “box” that holds many items of the same (or related) type. Different kinds of boxes are optimised for different jobs: some keep items in a row for fast lookup by position, some sort items automatically, and some use hash tables for near-instant lookups by key.

### Key Characteristics

- **Value semantics** – containers own their elements; copying a container copies its elements.
- **Allocator-aware** – most containers accept an allocator template parameter for custom memory management.
- **Iterator-based access** – every container provides iterators (begin/end) for traversal.
- **Complexity guarantees** – the standard mandates time-complexity bounds for operations (e.g., O(1) amortised `push_back` for `vector`).
- **Iterator invalidation rules** – modifying a container may invalidate iterators and references, with rules differing per container.
- **Since C++11** – move semantics and `constexpr` support (for many operations) are available.

### Prerequisites

- Basic knowledge of C++ templates and class templates.
- Familiarity with iterators and the range-based `for` loop.
- Understanding of Big-O notation and algorithmic complexity.
- Awareness of memory management (stack vs. heap).

### Related Programming Areas

- **STL algorithms** – containers are designed to work with `<algorithm>` functions.
- **Generic programming** – containers are templates parameterised on element type and allocator.
- **Memory management** – allocators and custom memory pools.
- **Concurrency** – thread-safety guarantees (or lack thereof) for standard containers.
- **Serialization** – persisting container contents.

### Core Concepts / Features

1. Sequence containers (`vector`, `array`, `deque`, `list`, `forward_list`)
2. Associative containers (`set`, `multiset`, `map`, `multimap`)
3. Unordered associative containers (`unordered_set`, `unordered_multiset`, `unordered_map`, `unordered_multimap`)
4. Container adaptors (`stack`, `queue`, `priority_queue`)
5. Container operations (common member functions)
6. Container performance (complexity guarantees)
7. Container selection (choosing the right container)
8. Container best practices

---

## 1. Sequence Containers

### Definitions

**Core Definition**
Sequence containers are containers that store elements in a linear order, where each element has a specific position (index) and the order of elements is determined by the order of insertion.

**Technical Definition**
Sequence containers satisfy the `SequenceContainer` named requirement: they support `emplace_front`/`push_front` (except `array` and `vector`), `emplace_back`/`push_back` (except `array` and `forward_list`), and provide iterators that are at least forward iterators. Elements are stored in the order they are inserted (unless explicitly reordered).

**Beginner-Friendly Explanation**
Sequence containers are like numbered slots in a row: the first item you put in is at position 0, the next at position 1, and so on. Different sequence containers differ in how they allocate memory and how fast different operations are.

### Purposes

- To store elements in a well-defined order.
- To provide positional access (via index or iterator).
- To support insertion and removal at specific positions.
- To serve as the foundation for container adaptors.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <container_header>
std::container_type<ElementType, Allocator = std::allocator<ElementType>> name;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `container_header` | The header that defines the container (e.g., `<vector>`). |
| `container_type` | The container template name (e.g., `vector`, `list`). |
| `ElementType` | The type of elements stored. |
| `Allocator` | Optional allocator type (defaults to `std::allocator`). |

**Syntax Rules**
- All sequence containers except `array` accept an allocator template parameter.
- `array` requires a non-type template parameter for its size.
- Elements are stored in contiguous memory for `array` and `vector`; non-contiguously for others.

**Constraints and Limitations**
- `array` has a fixed size determined at compile time.
- `vector` reallocation invalidates all iterators and references.
- `forward_list` does not provide `size()` member function (use `std::distance`).

### Annotated Code Example (overview)

```cpp
#include <vector>
#include <array>
#include <deque>
#include <list>
#include <forward_list>

int main() {
    std::vector<int> v = {1, 2, 3};          // dynamic contiguous array
    std::array<int, 3> a = {1, 2, 3};        // fixed-size contiguous array
    std::deque<int> d = {1, 2, 3};           // double-ended queue
    std::list<int> l = {1, 2, 3};            // doubly-linked list
    std::forward_list<int> fl = {1, 2, 3};   // singly-linked list
}
```

**Expected Output**  
(No output; containers are constructed.)

---

## 1.1 `std::vector`

### Definitions

**Core Definition**
`std::vector` is a sequence container that encapsulates a dynamically sized array, stored contiguously in memory.

**Technical Definition**
`std::vector<T, Allocator>` is a class template that provides fast random access (O(1)), amortised O(1) insertion/removal at the end, and O(n) insertion/removal elsewhere. Elements are stored contiguously, so a pointer to an element can be passed to C APIs expecting a raw array.

**Beginner-Friendly Explanation**
`std::vector` is like a smart array that can grow and shrink automatically. You can access any element instantly by index, and adding to the end is fast.

### Purposes

- To provide a dynamic array with contiguous storage.
- To enable fast random access by index.
- To support efficient iteration and use with STL algorithms.
- To serve as the default sequence container in most applications.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <vector>
std::vector<T> v;                    // empty
std::vector<T> v(n);                 // n default-constructed elements
std::vector<T> v(n, value);          // n copies of value
std::vector<T> v = {a, b, c};       // initializer list
```

**Component Breakdown**

| Member Function | Complexity | Description |
|-----------------|------------|-------------|
| `operator[]` | O(1) | Unchecked access |
| `at()` | O(1) | Checked access (throws `std::out_of_range`) |
| `front()`, `back()` | O(1) | Access first/last element |
| `push_back()` | Amortised O(1) | Insert at end |
| `pop_back()` | O(1) | Remove last element |
| `insert()` | O(n) | Insert at position |
| `erase()` | O(n) | Erase at position |
| `size()` | O(1) | Number of elements |
| `capacity()` | O(1) | Allocated storage |

**Syntax Rules**
- `vector` does not provide `push_front` or `pop_front` (use `deque` for that).
- Reallocation invalidates all iterators and references.
- `reserve()` prevents reallocation until capacity is exceeded.

**Constraints and Limitations**
- `std::vector<bool>` is a partial specialisation that packs bits and does not provide normal reference semantics.
- Insertion in the middle is O(n) because subsequent elements must be shifted.

### Annotated Code Examples

**Example 1: Basic `vector` operations**

```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {10, 20, 30};
    v.push_back(40);                       // O(1) amortised
    v.insert(v.begin() + 1, 15);          // O(n)

    for (int x : v) std::cout << x << ' ';
    std::cout << '\n';
}
```

**Expected Output**
```
10 15 20 30 40
```

**Why this output?**  
`push_back` appends `40`; `insert` at position 1 shifts elements and inserts `15`.

**Example 2: Reserve to avoid reallocation**

```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v;
    v.reserve(100);                        // pre-allocate capacity
    for (int i = 0; i < 100; ++i) v.push_back(i);
    std::cout << v.size() << ' ' << v.capacity() << '\n';
}
```

**Expected Output**
```
100 100
```

**Why this output?**  
`reserve(100)` allocates capacity for 100 elements, so no reallocation occurs during `push_back`.

### Real-World Cases

- **Dynamic arrays** – storing sensor readings, user records.
- **Buffers** – `vector<char>` as a byte buffer.
- **Default container** – recommended by the C++ Core Guidelines as the default choice.

---

## 1.2 `std::array`

### Definitions

**Core Definition**
`std::array` is a sequence container that encapsulates a fixed-size array.

**Technical Definition**
`std::array<T, N>` is an aggregate type with the same semantics as a struct holding a C-style array `T[N]` as its only non-static data member. Unlike a C array, it does not decay to `T*` and it knows its own size.

**Beginner-Friendly Explanation**
`std::array` is a safer, more convenient version of a C array. Its size is fixed at compile time, but it provides member functions like `size()`, `at()`, and iterators.

### Purposes

- To provide a fixed-size array with container-like interface.
- To avoid the pitfalls of C arrays (decay, lack of size information).
- To enable stack allocation of fixed-size sequences.
- To allow use with STL algorithms.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <array>
std::array<T, N> arr = { ... };
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `T` | Element type. |
| `N` | Number of elements (compile-time constant). |

**Syntax Rules**
- `N` must be a constant expression.
- `std::array` is an aggregate, so aggregate initialisation is used.
- `at()` performs bounds checking; `operator[]` does not.
- `swap()` on `array` is O(n), not O(1).

**Constraints and Limitations**
- Size cannot change at runtime.
- Default construction of `array` of non-class type leaves elements uninitialised.
- `N == 0` is allowed; `front()` and `back()` on a zero-sized array are undefined.

### Annotated Code Examples

**Example 1: Fixed-size array**

```cpp
#include <array>
#include <iostream>

int main() {
    std::array<int, 5> a = {1, 2, 3, 4, 5};
    std::cout << a.size() << '\n';          // 5
    std::cout << a.at(2) << '\n';           // 3 (checked)
    a[0] = 10;
    for (int x : a) std::cout << x << ' ';
}
```

**Expected Output**
```
5
3
10 2 3 4 5
```

**Why this output?**  
`size()` returns 5, `at(2)` returns the third element (index 2), and `a[0] = 10` modifies the first element.

**Example 2: `array` with STL algorithms**

```cpp
#include <array>
#include <algorithm>
#include <iostream>

int main() {
    std::array<int, 4> a = {4, 2, 3, 1};
    std::sort(a.begin(), a.end());
    for (int x : a) std::cout << x << ' ';
}
```

**Expected Output**
```
1 2 3 4
```

**Why this output?**  
`std::sort` reorders the elements in ascending order.

### Real-World Cases

- **Fixed-size buffers** – embedded systems, lookup tables.
- **Mathematical vectors** – small fixed-size numeric arrays.
- **Compile-time-sized collections** – where dynamic allocation is undesirable.

---

## 1.3 `std::deque`

### Definitions

**Core Definition**
`std::deque` (double-ended queue) is an indexed sequence container that allows fast insertion and deletion at both its beginning and its end.

**Technical Definition**
`std::deque<T, Allocator>` stores elements in a sequence of individually allocated fixed-size arrays (blocks), with additional bookkeeping. Random access is O(1) but requires two pointer dereferences. Insertion/removal at either end is O(1) and does not invalidate pointers or references to other elements.

**Beginner-Friendly Explanation**
`std::deque` is like a `vector` that can efficiently add and remove items at both the front and the back. It is not contiguous in memory, so it is slightly slower for indexed access than `vector`, but much faster for front insertions.

### Purposes

- To provide efficient insertion/removal at both ends.
- To avoid the O(n) cost of `vector::insert(begin())`.
- To serve as the default underlying container for `std::stack` and `std::queue`.
- To support random access with acceptable performance.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <deque>
std::deque<T> d;
```

**Component Breakdown**

| Operation | Complexity |
|-----------|------------|
| `push_back`, `pop_back` | O(1) |
| `push_front`, `pop_front` | O(1) |
| `operator[]`, `at` | O(1) (two dereferences) |
| `insert`, `erase` (middle) | O(n) |

**Syntax Rules**
- Elements are not stored contiguously.
- Insertion at either end does not invalidate references to other elements.
- Expansion is cheaper than `vector` because existing elements are not copied.

**Constraints and Limitations**
- Indexed access is slightly slower than `vector`.
- Deques typically have a large minimal memory cost (e.g., an internal array).

### Annotated Code Examples

**Example 1: Double-ended operations**

```cpp
#include <deque>
#include <iostream>

int main() {
    std::deque<int> d;
    d.push_back(2);
    d.push_front(1);
    d.push_back(3);
    for (int x : d) std::cout << x << ' ';
}
```

**Expected Output**
```
1 2 3
```

**Why this output?**  
`push_front` adds `1` at the front; `push_back` adds `2` and `3` at the back.

**Example 2: Deque as a queue**

```cpp
#include <deque>
#include <iostream>

int main() {
    std::deque<int> q = {1, 2, 3};
    q.pop_front();                          // O(1)
    q.push_back(4);
    for (int x : q) std::cout << x << ' ';
}
```

**Expected Output**
```
2 3 4
```

**Why this output?**  
`pop_front` removes the first element; `push_back` appends at the end.

### Real-World Cases

- **Task queues** – adding jobs at both ends.
- **Sliding window algorithms** – efficient front removal and back insertion.
- **Undo/redo stacks** – double-ended access.

---

## 1.4 `std::list`

### Definitions

**Core Definition**
`std::list` is a sequence container that supports constant-time insertion and removal of elements from anywhere in the container.

**Technical Definition**
`std::list<T, Allocator>` is usually implemented as a doubly-linked list. It provides bidirectional iterators but no random access. Insertion and removal of elements do not invalidate iterators or references to other elements.

**Beginner-Friendly Explanation**
`std::list` is like a chain of nodes: each element points to the previous and next one. You can insert or remove items very quickly if you already have a pointer to the right spot, but finding an element by position requires walking the chain.

### Purposes

- To provide O(1) insertion/removal at any position (given an iterator).
- To avoid iterator invalidation on insertion/removal.
- To support splicing (moving elements between lists).
- To serve as a stable container for elements that are frequently reordered.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <list>
std::list<T> lst;
```

**Component Breakdown**

| Operation | Complexity |
|-----------|------------|
| `push_back`, `pop_back` | O(1) |
| `push_front`, `pop_front` | O(1) |
| `insert`, `erase` (given iterator) | O(1) |
| `operator[]` | Not available |
| `size()` | O(1) (since C++11) |

**Syntax Rules**
- `list` provides bidirectional iterators.
- Splicing (`splice`) moves elements between lists in O(1).
- `sort()` member function sorts the list in O(n log n) without invalidating iterators.

**Constraints and Limitations**
- No random access; accessing the n-th element is O(n).
- Higher memory overhead per element (two pointers per node).
- Poor cache locality compared to contiguous containers.

### Annotated Code Examples

**Example 1: Constant-time insertion**

```cpp
#include <list>
#include <iostream>

int main() {
    std::list<int> lst = {1, 2, 4};
    auto it = lst.begin();
    std::advance(it, 2);                   // O(n) to find position
    lst.insert(it, 3);                     // O(1) insertion
    for (int x : lst) std::cout << x << ' ';
}
```

**Expected Output**
```
1 2 3 4
```

**Why this output?**  
`advance` walks to the third position; `insert` links the new node in O(1).

**Example 2: Splice between lists**

```cpp
#include <list>
#include <iostream>

int main() {
    std::list<int> a = {1, 2, 3};
    std::list<int> b = {4, 5};
    a.splice(a.end(), b);                  // O(1) splice
    for (int x : a) std::cout << x << ' ';
}
```

**Expected Output**
```
1 2 3 4 5
```

**Why this output?**  
`splice` moves all elements of `b` into `a` without copying.

### Real-World Cases

- **LRU caches** – frequent reordering and removal.
- **Adjacency lists** – graph representations.
- **Playlist management** – reordering songs.

---

## 1.5 `std::forward_list`

### Definitions

**Core Definition**
`std::forward_list` is a sequence container that supports fast insertion and removal of elements from anywhere in the container, implemented as a singly-linked list.

**Technical Definition**
`std::forward_list<T, Allocator>` provides forward iterators only. It is more space-efficient than `std::list` but lacks `size()` and bidirectional iteration. Insertion and removal do not invalidate iterators to other elements.

**Beginner-Friendly Explanation**
`std::forward_list` is like a one-way chain: you can only move forward through the elements. It uses less memory per element than `std::list` because each node stores only one pointer.

### Purposes

- To provide a memory-efficient linked list.
- To support forward-only traversal.
- To enable O(1) insertion/removal after a given position.
- To serve as the lightest-weight linked list in the standard library.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <forward_list>
std::forward_list<T> fl;
```

**Component Breakdown**

| Operation | Complexity |
|-----------|------------|
| `push_front`, `pop_front` | O(1) |
| `insert_after`, `erase_after` | O(1) |
| `size()` | Not available (O(n) via `std::distance`) |
| Random access | Not supported |

**Syntax Rules**
- Uses `insert_after` and `erase_after` instead of `insert` and `erase`.
- Provides `before_begin()` for operations at the front.
- No `back()` or `push_back()`.

**Constraints and Limitations**
- Only forward iteration.
- No direct access to the last element.
- `size()` is not provided for efficiency reasons.

### Annotated Code Examples

**Example 1: `forward_list` operations**

```cpp
#include <forward_list>
#include <iostream>

int main() {
    std::forward_list<int> fl = {1, 2, 3};
    fl.push_front(0);                      // O(1)
    fl.insert_after(fl.begin(), 10);      // O(1) after first element
    for (int x : fl) std::cout << x << ' ';
}
```

**Expected Output**
```
0 10 1 2 3
```

**Why this output?**  
`push_front` adds `0`; `insert_after` inserts `10` after the first element.

**Example 2: `erase_after`**

```cpp
#include <forward_list>
#include <iostream>

int main() {
    std::forward_list<int> fl = {1, 2, 3, 4};
    fl.erase_after(fl.begin());           // removes the element after the first
    for (int x : fl) std::cout << x << ' ';
}
```

**Expected Output**
```
1 3 4
```

**Why this output?**  
`erase_after` removes the node following the iterator, which is `2`.

### Real-World Cases

- **Memory-constrained environments** – embedded systems.
- **Hash table buckets** – separate chaining.
- **Forward-only processing pipelines**.

---

## 2. Associative Containers

### Definitions

**Core Definition**
Associative containers are containers that store elements in a sorted order and provide fast lookup based on keys.

**Technical Definition**
Associative containers implement sorted data structures (typically red-black trees) that support O(log n) search, insertion, and removal. They include `set`, `multiset`, `map`, and `multimap`.

**Beginner-Friendly Explanation**
Associative containers keep their elements sorted automatically. You can quickly find an element by its key, and iteration always visits elements in sorted order.

### Purposes

- To maintain elements in sorted order.
- To provide fast key-based lookup (O(log n)).
- To support range queries (lower_bound, upper_bound).
- To enable ordered iteration.

### Key Characteristics

| Feature | `set` / `multiset` | `map` / `multimap` |
|---------|-------------------|-------------------|
| Stores | Keys only | Key-value pairs |
| Key uniqueness | `set`: unique; `multiset`: duplicates allowed | `map`: unique; `multimap`: duplicates allowed |
| Ordering | Sorted by key | Sorted by key |

---

## 2.1 `std::set` and `std::multiset`

### Definitions

**Core Definition**
`std::set` is an associative container that contains a sorted set of unique objects of type `Key`. `std::multiset` is the same but allows duplicate keys.

**Technical Definition**
`std::set<Key, Compare, Allocator>` and `std::multiset<Key, Compare, Allocator>` store keys in sorted order according to the `Compare` function object (default `std::less<Key>`). `set` allows at most one element per key; `multiset` allows multiple.

**Beginner-Friendly Explanation**
`std::set` is like a sorted list that automatically removes duplicates. `std::multiset` is similar but keeps duplicates.

### Purposes

- To maintain a sorted collection of unique (or duplicate) keys.
- To perform fast membership tests.
- To support ordered range queries.
- To eliminate duplicates automatically.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <set>
std::set<Key> s;
std::multiset<Key> ms;
```

**Component Breakdown**

| Operation | Complexity |
|-----------|------------|
| `insert` | O(log n) |
| `erase` | O(log n) |
| `find` | O(log n) |
| `count` | O(log n) (for `set`, 0 or 1) |
| Iteration | O(n) sorted order |

**Syntax Rules**
- `set` does not allow modifying elements through iterators (keys are `const`).
- `multiset` allows multiple equivalent keys.
- `lower_bound` / `upper_bound` provide range queries in O(log n).

**Constraints and Limitations**
- No random access.
- Insertion may invalidate iterators (but not references, which remain valid).
- Requires a strict weak ordering (`Compare`).

### Annotated Code Examples

**Example 1: `set` uniqueness**

```cpp
#include <set>
#include <iostream>

int main() {
    std::set<int> s = {3, 1, 2, 3, 1};   // duplicates ignored
    for (int x : s) std::cout << x << ' ';
}
```

**Expected Output**
```
1 2 3
```

**Why this output?**  
`set` stores only unique keys in sorted order.

**Example 2: `multiset` with duplicates**

```cpp
#include <set>
#include <iostream>

int main() {
    std::multiset<int> ms = {3, 1, 2, 3, 1};
    for (int x : ms) std::cout << x << ' ';
}
```

**Expected Output**
```
1 1 2 3 3
```

**Why this output?**  
`multiset` preserves all duplicate keys, sorted.

### Real-World Cases

- **Deduplication** – removing duplicates from a collection.
- **Membership testing** – checking whether a value exists.
- **Sorted iteration** – processing elements in order.

---

## 2.2 `std::map` and `std::multimap`

### Definitions

**Core Definition**
`std::map` is an associative container that contains sorted key-value pairs with unique keys. `std::multimap` allows duplicate keys.

**Technical Definition**
`std::map<Key, T, Compare, Allocator>` stores `std::pair<const Key, T>` elements sorted by key. `operator[]` inserts a default-constructed value if the key is absent. `multimap` does not provide `operator[]` because keys need not be unique.

**Beginner-Friendly Explanation**
`std::map` is like a dictionary: each key is associated with a value, and keys are kept sorted. `std::multimap` is a dictionary that allows multiple entries for the same key.

### Purposes

- To associate values with keys.
- To provide fast key-based lookup.
- To maintain sorted order by key.
- To support frequency counting and indexing.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <map>
std::map<Key, T> m;
std::multimap<Key, T> mm;
```

**Component Breakdown**

| Operation | Complexity |
|-----------|------------|
| `operator[]` | O(log n) |
| `insert` | O(log n) |
| `find` | O(log n) |
| `erase` | O(log n) |

**Syntax Rules**
- `map::operator[]` requires default-constructible `T`.
- `map` keys are `const` and cannot be modified in place.
- `multimap` uses `equal_range` to access all values for a key.

**Constraints and Limitations**
- `map` cannot store duplicate keys; use `multimap` for duplicates.
- `operator[]` is not available for `multimap`.

### Annotated Code Examples

**Example 1: `map` basics**

```cpp
#include <map>
#include <iostream>
#include <string>

int main() {
    std::map<std::string, int> ages;
    ages["Alice"] = 30;
    ages["Bob"] = 25;
    ages["Alice"] = 31;                   // updates existing key
    for (const auto& [name, age] : ages)
        std::cout << name << ": " << age << '\n';
}
```

**Expected Output**
```
Alice: 31
Bob: 25
```

**Why this output?**  
Keys are sorted alphabetically; `ages["Alice"] = 31` updates the existing entry.

**Example 2: `multimap` with duplicate keys**

```cpp
#include <map>
#include <iostream>

int main() {
    std::multimap<char, int> mm;
    mm.insert({'a', 1});
    mm.insert({'a', 2});
    mm.insert({'b', 3});
    auto range = mm.equal_range('a');
    for (auto it = range.first; it != range.second; ++it)
        std::cout << it->second << ' ';
}
```

**Expected Output**
```
1 2
```

**Why this output?**  
`equal_range` returns the range of elements with key `'a'`.

### Real-World Cases

- **Dictionaries** – word-to-definition mappings.
- **Frequency counting** – `std::map<std::string, int>` word counts.
- **Symbol tables** – compilers and interpreters.

---

## 3. Unordered Associative Containers

### Definitions

**Core Definition**
Unordered associative containers are containers that store elements in a hash table, providing average O(1) lookup, insertion, and removal.

**Technical Definition**
Unordered associative containers (`unordered_set`, `unordered_multiset`, `unordered_map`, `unordered_multimap`) use a hash function to organise elements into buckets. Average complexity is O(1); worst-case is O(n).

**Beginner-Friendly Explanation**
These containers use a “hash” to find items almost instantly. They don’t keep elements in sorted order, but they are usually the fastest choice for pure lookup.

### Key Characteristics

| Container | Unique keys? | Stores key-value? |
|-----------|--------------|-------------------|
| `unordered_set` | Yes | No |
| `unordered_multiset` | No | No |
| `unordered_map` | Yes | Yes |
| `unordered_multimap` | No | Yes |

---

## 3.1 `std::unordered_set` and `std::unordered_multiset`

### Definitions

**Core Definition**
`std::unordered_set` is an unordered associative container that contains a set of unique objects of type `Key`. `std::unordered_multiset` allows duplicate keys.

**Technical Definition**
Both are hash-table-based containers parameterised on `Key`, `Hash`, and `KeyEqual`. They do not maintain sorted order. `unordered_set` ensures uniqueness; `unordered_multiset` allows multiple equivalent keys.

**Beginner-Friendly Explanation**
`unordered_set` is like a set that uses hashing for speed, but it doesn’t sort its elements. `unordered_multiset` is the same but allows duplicates.

### Purposes

- To provide O(1) average-time membership tests.
- To deduplicate elements without requiring sorting.
- To support fast insertion and removal.
- To serve as a hash-based collection of keys.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <unordered_set>
std::unordered_set<Key> us;
std::unordered_multiset<Key> ums;
```

**Component Breakdown**

| Operation | Average Complexity | Worst Case |
|-----------|-------------------|------------|
| `insert` | O(1) | O(n) |
| `erase` | O(1) | O(n) |
| `find` | O(1) | O(n) |
| `count` | O(1) | O(n) |

**Syntax Rules**
- Requires a `Hash` function object (default `std::hash<Key>`).
- Requires an equality predicate (default `std::equal_to<Key>`).
- Rehashing invalidates iterators but not references.

**Constraints and Limitations**
- Iteration order is unspecified and may change after rehashing.
- No sorted order; use `set` if order matters.
- Hash quality affects performance.

### Annotated Code Examples

**Example 1: `unordered_set` uniqueness**

```cpp
#include <unordered_set>
#include <iostream>

int main() {
    std::unordered_set<int> us = {3, 1, 2, 3, 1};
    std::cout << us.size() << '\n';        // 3 (duplicates removed)
    for (int x : us) std::cout << x << ' ';
}
```

**Expected Output** (order may vary)
```
3
1 2 3
```

**Why this output?**  
Duplicates are ignored; iteration order is unspecified.

**Example 2: `unordered_multiset`**

```cpp
#include <unordered_set>
#include <iostream>

int main() {
    std::unordered_multiset<int> ums = {1, 1, 2};
    std::cout << ums.count(1) << '\n';     // 2
}
```

**Expected Output**
```
2
```

**Why this output?**  
`count` returns the number of elements with the given key.

### Real-World Cases

- **Deduplication** – removing duplicate records.
- **Caches** – fast membership checks.
- **Graph algorithms** – visited-set tracking.

---

## 3.2 `std::unordered_map` and `std::unordered_multimap`

### Definitions

**Core Definition**
`std::unordered_map` is an unordered associative container that contains key-value pairs with unique keys. `std::unordered_multimap` allows duplicate keys.

**Technical Definition**
Both use hash tables. `unordered_map` provides `operator[]`; `unordered_multimap` does not. Average complexity for lookup, insertion, and removal is O(1).

**Beginner-Friendly Explanation**
`unordered_map` is a hash-based dictionary — extremely fast for looking up a value by its key. `unordered_multimap` is the same but allows multiple values per key.

### Purposes

- To provide fast key-value lookup.
- To implement associative arrays without ordering.
- To support high-performance counting and indexing.
- To serve as a hash-based dictionary.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <unordered_map>
std::unordered_map<Key, T> um;
std::unordered_multimap<Key, T> umm;
```

**Component Breakdown**

| Operation | Average Complexity | Worst Case |
|-----------|-------------------|------------|
| `operator[]` | O(1) | O(n) |
| `insert` | O(1) | O(n) |
| `find` | O(1) | O(n) |
| `erase` | O(1) | O(n) |

**Syntax Rules**
- `unordered_map::operator[]` inserts a default-constructed value if the key is missing.
- `unordered_multimap` uses `equal_range` to access multiple values for a key.
- Rehashing invalidates iterators.

**Constraints and Limitations**
- No sorted order.
- Worst-case O(n) if hash function is poor or adversarial.
- `operator[]` requires default-constructible `T`.

### Annotated Code Examples

**Example 1: `unordered_map` basics**

```cpp
#include <unordered_map>
#include <iostream>
#include <string>

int main() {
    std::unordered_map<std::string, int> um;
    um["Alice"] = 30;
    um["Bob"] = 25;
    std::cout << um["Alice"] << '\n';
}
```

**Expected Output**
```
30
```

**Why this output?**  
`operator[]` retrieves the value associated with `"Alice"`.

**Example 2: `unordered_multimap`**

```cpp
#include <unordered_map>
#include <iostream>

int main() {
    std::unordered_multimap<char, int> umm;
    umm.insert({'a', 1});
    umm.insert({'a', 2});
    auto range = umm.equal_range('a');
    for (auto it = range.first; it != range.second; ++it)
        std::cout << it->second << ' ';
}
```

**Expected Output** (order may vary)
```
1 2
```

**Why this output?**  
`equal_range` returns all values associated with key `'a'`.

### Real-World Cases

- **Caches** – memoization tables.
- **Frequency counting** – `unordered_map<std::string, int>`.
- **Symbol tables** – fast identifier lookup.

---

## 4. Container Adaptors

### Definitions

**Core Definition**
Container adaptors are class templates that provide a restricted interface on top of an underlying sequence container, implementing specific data structures (stack, queue, priority queue).

**Technical Definition**
Container adaptors wrap a sequence container (by default `deque` or `vector`) and expose only the operations appropriate for the target data structure (e.g., LIFO for `stack`, FIFO for `queue`).

**Beginner-Friendly Explanation**
Adaptors are like specialised “views” of a container: they hide most of the container’s functionality and give you only the operations that make sense for a stack, queue, or priority queue.

### Purposes

- To provide standard data-structure interfaces (stack, queue, priority queue).
- To restrict operations to those appropriate for the abstraction.
- To allow the underlying container to be customised.
- To simplify code that uses these classic structures.

---

## 4.1 `std::stack`

### Definitions

**Core Definition**
`std::stack` is a container adaptor that provides a LIFO (last-in, first-out) data structure.

**Technical Definition**
`std::stack<T, Container>` uses `Container` (default `std::deque<T>`) as its underlying container and provides `push`, `pop`, `top`, `empty`, and `size`. It exposes only the operations of a stack.

**Beginner-Friendly Explanation**
A stack is like a pile of plates: you add (push) to the top and remove (pop) from the top. The last item added is the first one removed.

### Purposes

- To model LIFO behaviour.
- To provide a simple, restricted interface.
- To support expression evaluation, backtracking, and undo mechanisms.
- To allow underlying container customisation.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <stack>
std::stack<T> s;
```

**Component Breakdown**

| Operation | Complexity |
|-----------|------------|
| `push` | O(1) |
| `pop` | O(1) |
| `top` | O(1) |

**Syntax Rules**
- `pop()` does not return the removed element; use `top()` first.
- Underlying container must support `back()`, `push_back()`, `pop_back()`.
- Default container is `std::deque<T>`.

**Constraints and Limitations**
- No iteration over elements.
- No direct access to elements other than the top.

### Annotated Code Examples

**Example 1: Basic stack**

```cpp
#include <stack>
#include <iostream>

int main() {
    std::stack<int> s;
    s.push(1);
    s.push(2);
    s.push(3);
    std::cout << s.top() << '\n';          // 3
    s.pop();
    std::cout << s.top() << '\n';          // 2
}
```

**Expected Output**
```
3
2
```

**Why this output?**  
LIFO order: the last pushed element (`3`) is on top.

**Example 2: Stack with `vector` underlying container**

```cpp
#include <stack>
#include <vector>
#include <iostream>

int main() {
    std::stack<int, std::vector<int>> s;
    s.push(10);
    s.push(20);
    std::cout << s.top() << '\n';
}
```

**Expected Output**
```
20
```

**Why this output?**  
The underlying container is `vector`, but the interface remains LIFO.

### Real-World Cases

- **Expression evaluation** – parsing and evaluating postfix expressions.
- **Undo/redo** – storing states.
- **Depth-first search** – iterative DFS.

---

## 4.2 `std::queue`

### Definitions

**Core Definition**
`std::queue` is a container adaptor that provides a FIFO (first-in, first-out) data structure.

**Technical Definition**
`std::queue<T, Container>` uses `Container` (default `std::deque<T>`) and provides `push`, `pop`, `front`, `back`, `empty`, and `size`.

**Beginner-Friendly Explanation**
A queue is like a line at a ticket counter: the first person to join is the first to be served.

### Purposes

- To model FIFO behaviour.
- To provide a simple queue interface.
- To support breadth-first search, task scheduling, and message passing.
- To allow underlying container customisation.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <queue>
std::queue<T> q;
```

**Component Breakdown**

| Operation | Complexity |
|-----------|------------|
| `push` | O(1) |
| `pop` | O(1) |
| `front` | O(1) |
| `back` | O(1) |

**Syntax Rules**
- `pop()` removes the front element without returning it.
- Underlying container must support `front()`, `back()`, `push_back()`, `pop_front()`.
- Default container is `std::deque<T>`.

**Constraints and Limitations**
- No iteration over elements.
- Only the front and back are accessible.

### Annotated Code Examples

**Example 1: Basic queue**

```cpp
#include <queue>
#include <iostream>

int main() {
    std::queue<int> q;
    q.push(1);
    q.push(2);
    q.push(3);
    std::cout << q.front() << '\n';        // 1
    q.pop();
    std::cout << q.front() << '\n';        // 2
}
```

**Expected Output**
```
1
2
```

**Why this output?**  
FIFO order: the first pushed element (`1`) is at the front.

**Example 2: Queue with `list` underlying container**

```cpp
#include <queue>
#include <list>
#include <iostream>

int main() {
    std::queue<int, std::list<int>> q;
    q.push(10);
    q.push(20);
    std::cout << q.front() << '\n';
}
```

**Expected Output**
```
10
```

**Why this output?**  
`list` supports the required operations; the FIFO interface is preserved.

### Real-World Cases

- **Breadth-first search** – exploring graph levels.
- **Printer spooling** – jobs processed in arrival order.
- **Message queues** – asynchronous communication.

---

## 4.3 `std::priority_queue`

### Definitions

**Core Definition**
`std::priority_queue` is a container adaptor that provides a priority queue, where the largest element (by default) is always at the top.

**Technical Definition**
`std::priority_queue<T, Container, Compare>` uses a heap-ordered underlying container (default `std::vector<T>` with `std::less<T>`) and provides `push`, `pop`, `top`, `empty`, and `size`. `top()` returns the greatest element according to `Compare`.

**Beginner-Friendly Explanation**
A priority queue is like a hospital emergency room: the most urgent case is treated first, regardless of arrival order.

### Purposes

- To model a priority-based data structure.
- To provide O(log n) insertion and O(1) access to the maximum.
- To support algorithms like Dijkstra’s and Huffman coding.
- To allow custom comparison for priority ordering.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <queue>
std::priority_queue<T> pq;
```

**Component Breakdown**

| Operation | Complexity |
|-----------|------------|
| `push` | O(log n) |
| `pop` | O(log n) |
| `top` | O(1) |

**Syntax Rules**
- Default ordering is max-heap (`std::less<T>`); use `std::greater<T>` for min-heap.
- Underlying container must support random access iterators and `front()`, `push_back()`, `pop_back()`.
- Default container is `std::vector<T>`.

**Constraints and Limitations**
- No iteration over elements.
- Only the top (highest-priority) element is accessible.

### Annotated Code Examples

**Example 1: Max-heap priority queue**

```cpp
#include <queue>
#include <iostream>

int main() {
    std::priority_queue<int> pq;
    pq.push(3);
    pq.push(1);
    pq.push(4);
    std::cout << pq.top() << '\n';         // 4
    pq.pop();
    std::cout << pq.top() << '\n';         // 3
}
```

**Expected Output**
```
4
3
```

**Why this output?**  
The largest element is always at the top.

**Example 2: Min-heap with `greater`**

```cpp
#include <queue>
#include <iostream>
#include <functional>

int main() {
    std::priority_queue<int, std::vector<int>, std::greater<int>> pq;
    pq.push(3);
    pq.push(1);
    pq.push(4);
    std::cout << pq.top() << '\n';         // 1
}
```

**Expected Output**
```
1
```

**Why this output?**  
`std::greater<int>` reverses the ordering, making the smallest element the highest priority.

### Real-World Cases

- **Dijkstra’s algorithm** – selecting the nearest unvisited node.
- **Huffman coding** – merging lowest-frequency nodes.
- **Task scheduling** – highest-priority tasks first.

---

## 5. Container Operations (Common Member Functions)

### Definitions

**Core Definition**
Container operations are the member functions shared by most standard containers for accessing, modifying, and querying the container.

**Technical Definition**
Most containers provide a common interface: `begin()`/`end()`, `size()`, `empty()`, `clear()`, `insert()`, `erase()`, `swap()`, and various constructors. The exact set depends on the container category.

**Beginner-Friendly Explanation**
Even though containers differ internally, they share many of the same “commands” — like `size()`, `empty()`, `clear()` — so you can use them in similar ways.

### Key Operations

| Operation | Description | Available in |
|-----------|-------------|--------------|
| `begin()` / `end()` | Iterators to first/past-the-end | All |
| `size()` | Number of elements | All except `forward_list` (before C++17) |
| `empty()` | Whether container is empty | All |
| `clear()` | Remove all elements | All except `array` |
| `insert()` | Insert element(s) | Most |
| `erase()` | Remove element(s) | Most |
| `push_back()` / `pop_back()` | Add/remove at end | `vector`, `deque`, `list` |
| `push_front()` / `pop_front()` | Add/remove at front | `deque`, `list`, `forward_list` |
| `emplace()` | Construct in-place | Most |
| `swap()` | Exchange contents | All |

### Syntax Rules

- `emplace_back` constructs the element directly in the container, avoiding a temporary.
- `erase` invalidates iterators to the erased elements.
- `swap` is O(1) for most containers (except `array`, which is O(n)).

---

## 6. Container Performance (Complexity Guarantees)

### Definitions

**Core Definition**
Container performance refers to the time and space complexity guarantees that the C++ standard mandates for container operations.

**Technical Definition**
The standard specifies worst-case or average-case complexity bounds (Big-O) for each container operation. These guarantees allow programmers to reason about performance independently of implementation details.

**Beginner-Friendly Explanation**
Different containers are fast at different things. The standard tells you, for example, that `vector` can access any element instantly (O(1)), while `list` takes linear time (O(n)) to find the n-th element.

### Comprehensive Complexity Table

| Operation | `vector` | `deque` | `list` | `forward_list` | `array` | `map`/`set` | `unordered_map`/`unordered_set` |
|-----------|----------|---------|--------|----------------|---------|-------------|----------------------------------|
| **Random access** | O(1) | O(1) | O(n) | O(n) | O(1) | — | — |
| **Insert/erase at front** | O(n) | O(1) | O(1) | O(1) | — | — | — |
| **Insert/erase at back** | Amort. O(1) | O(1) | O(1) | — | — | — | — |
| **Insert/erase in middle** | O(n) | O(n) | O(1)* | O(1)* | — | O(log n) | O(1) avg |
| **Lookup by key** | — | — | — | — | — | O(log n) | O(1) avg |
| **Iteration** | O(n) | O(n) | O(n) | O(n) | O(n) | O(n) | O(n) |
| **`find` (unsorted)** | O(n) | O(n) | O(n) | O(n) | O(n) | — | — |

\*O(1) if you already have an iterator to the position; finding the position is O(n).

### Key Takeaways

- `vector` is the fastest for most operations except front insertion.
- `list`/`forward_list` excel at O(1) insertion/erasure given an iterator.
- `map`/`set` provide O(log n) for all ordered operations.
- `unordered_*` provide O(1) average lookup but O(n) worst case.
- Constant factors and cache behaviour can make “worse” Big-O faster in practice for small sizes.

---

## 7. Container Selection

### Definitions

**Core Definition**
Container selection is the process of choosing the most appropriate standard container for a given use case based on required operations, performance, and semantics.

**Technical Definition**
Selection depends on the operations required (access pattern, insertion/removal locations), ordering requirements, iterator invalidation constraints, and memory/performance trade-offs.

**Beginner-Friendly Explanation**
Picking the right container is like picking the right tool: use a `vector` for a list you mostly read by index, a `list` if you insert/remove in the middle often, a `map` if you need sorted keys, and an `unordered_map` if you just need fast lookups.

### Selection Guide

| Requirement | Recommended Container |
|-------------|----------------------|
| Default choice, fast random access, dynamic size | `std::vector` |
| Fixed size known at compile time | `std::array` |
| Fast insertion/removal at both ends | `std::deque` |
| Frequent insertion/removal in the middle (with iterators) | `std::list` |
| Memory-efficient forward-only linked list | `std::forward_list` |
| Sorted unique keys | `std::set` |
| Sorted keys with duplicates | `std::multiset` |
| Sorted key-value pairs, unique keys | `std::map` |
| Sorted key-value pairs, duplicate keys | `std::multimap` |
| Fast lookup, no ordering required, unique keys | `std::unordered_set` |
| Fast lookup, no ordering, duplicate keys | `std::unordered_multiset` |
| Fast key-value lookup, no ordering, unique keys | `std::unordered_map` |
| Fast key-value lookup, no ordering, duplicate keys | `std::unordered_multimap` |
| LIFO data structure | `std::stack` |
| FIFO data structure | `std::queue` |
| Priority-based access | `std::priority_queue` |

**Core Guideline**  
The C++ Core Guidelines recommend: “Prefer using STL `vector` by default unless you have a reason to use a different container” (SL.con.2).

---

## 8. Container Best Practices

### Definitions

**Core Definition**
Container best practices are guidelines for using standard containers safely, efficiently, and idiomatically.

**Technical Definition**
These practices are codified in resources such as the C++ Core Guidelines, which include rules like SL.con.1 (“Prefer using STL `array` or `vector` instead of a C array”) and SL.con.2 (“Prefer using STL `vector` by default”).

**Beginner-Friendly Explanation**
Best practices are good habits: use containers instead of raw arrays, prefer `vector` unless you need something special, and be aware of iterator invalidation.

### Key Guidelines

| Guideline | Summary |
|-----------|---------|
| **SL.con.1** | Prefer `std::array` or `std::vector` over C arrays. |
| **SL.con.2** | Prefer `std::vector` by default unless you have a reason. |
| **Use `emplace_back`** | Construct elements in-place to avoid temporaries. |
| **Reserve when size is known** | Call `reserve()` on `vector` to avoid reallocations. |
| **Be aware of iterator invalidation** | Modifying a container may invalidate iterators. |
| **Prefer range-based `for`** | More readable and less error-prone. |
| **Use `at()` for checked access** | When bounds safety is required. |
| **Avoid `std::vector<bool>` pitfalls** | It is a bitset-like specialisation. |
| **Use `const` references** | When passing containers to functions to avoid copies. |
| **Choose the right container** | Match the container to the access pattern. |

### Annotated Code Example

```cpp
#include <vector>
#include <iostream>

void print(const std::vector<int>& v) {   // const reference: no copy
    for (int x : v) std::cout << x << ' ';
}

int main() {
    std::vector<int> v;
    v.reserve(10);                         // avoid reallocations
    for (int i = 0; i < 10; ++i)
        v.emplace_back(i);                 // in-place construction
    print(v);
}
```

**Expected Output**
```
0 1 2 3 4 5 6 7 8 9
```

**Why this output?**  
`reserve` pre-allocates storage; `emplace_back` constructs elements directly; `print` takes a `const` reference to avoid copying.

---

## References

- Containers library – https://en.cppreference.com/w/cpp/container
- `std::vector` – https://en.cppreference.com/w/cpp/container/vector
- `std::array` – https://en.cppreference.com/w/cpp/container/array
- `std::deque` – https://en.cppreference.com/w/cpp/container/deque
- `std::list` – https://en.cppreference.com/w/cpp/container/list
- `std::forward_list` – https://en.cppreference.com/w/cpp/container/forward_list
- UnorderedAssociativeContainer requirements – https://en.cppreference.com/w/cpp/named_req/UnorderedAssociativeContainer
- Container adaptors – https://en.cppreference.com/w/cpp/container#Container_adaptors
- C++ Core Guidelines — SL.con: Containers – https://cpp-core-guidelines-docs.vercel.app/stdlib
- Time complexity guarantees (reference table) – https://en.cppreference.com/w/cpp/container
- C++ Working Draft (N4950) — Containers library – https://eel.is/c++draft/containers

---

This cheat sheet provides a comprehensive, structured reference for C++ containers, covering all standard container categories, individual containers, operations, performance characteristics, selection guidance, and best practices, with annotated code examples and authoritative references.