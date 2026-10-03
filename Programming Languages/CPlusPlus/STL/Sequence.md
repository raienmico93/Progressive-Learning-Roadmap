# C++ Sequence Containers & Contiguous Memory — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Sequence containers are standard library class templates that store objects of the same type in a strictly linear arrangement, preserving the order of insertion. They provide different performance trade-offs for access, insertion, and deletion operations.

**Technical Definition**
A SequenceContainer is a Container that stores objects of the same type in a linear arrangement. The standard library provides six sequence containers: `std::array` (fixed-size contiguous array), `std::vector` (dynamic contiguous array), `std::deque` (double-ended queue), `std::list` (doubly-linked list), `std::forward_list` (singly-linked list), and `std::basic_string` (character sequence). All sequence containers except `std::array` satisfy the full SequenceContainer requirements, which include operations like `insert`, `erase`, `emplace`, `push_back`/`pop_back` (where applicable), and `push_front`/`pop_front` (where applicable).

**Beginner-Friendly Explanation**
Imagine you need to store a list of items — numbers, names, game objects — in order. C++ gives you several "containers" to choose from, like choosing between a notebook (array), a stretchable list (vector), a two-way queue (deque), or a chain of linked boxes (list). Each has different strengths: some are fast to read from but slow to insert in the middle; others are fast to insert but slow to search. Choosing the right one depends on what your program does most often.

---

### Key Characteristics

- **Linear ordering**: Elements are stored in a strictly linear arrangement, preserving insertion order.
- **Contiguous vs. node-based storage**: `std::array`, `std::vector`, and `std::basic_string` store elements contiguously; `std::deque` stores elements in fixed-size blocks; `std::list` and `std::forward_list` store elements as individual linked nodes.
- **Different complexity guarantees**: `std::vector` provides amortized O(1) `push_back`; `std::deque` provides amortized O(1) insertion at both ends; `std::list` and `std::forward_list` provide O(1) insertion anywhere given an iterator.
- **Iterator categories**: `std::array`, `std::vector`, and `std::deque` provide random-access iterators; `std::list` provides bidirectional iterators; `std::forward_list` provides forward iterators.
- **Non-owning views**: C++20 `std::span` and C++23 `std::mdspan` provide non-owning views over contiguous memory, generalizing pointers and arrays.

---

### Prerequisites

- Basic C++ syntax: templates, iterators, and the standard library.
- Understanding of memory layout: stack vs. heap, contiguous vs. pointer-based structures.
- Familiarity with iterators and the range-based for loop.
- (For `std::span`) A compiler supporting C++20 or later.
- (For `std::mdspan`) A compiler supporting C++23 or later.

---

### Related Programming Areas

- **Algorithm design and complexity analysis**: Choosing the right container affects algorithmic complexity.
- **Cache-efficient programming**: Contiguous containers (vector, array) exploit CPU cache lines far better than node-based containers (list).
- **Generic programming**: Containers are used as template parameters in generic algorithms.
- **Memory management and allocators**: `std::vector` and `std::deque` use allocators for dynamic memory.
- **Exception safety and RAII**: Containers manage resource lifetimes and must provide appropriate exception guarantees.

---

### Core Concepts / Features

1. **Fixed vs. Dynamic Storage** — `std::array` and `std::vector` (reallocation mechanics and optimization strategies)
2. **Double-Ended & Linked Lists** — `std::deque`, `std::list`, and `std::forward_list`
3. **Modern Multidimensional Views** — `std::span` (C++20) and `std::mdspan` (C++23)
4. **Exception Safety Guarantees** — `noexcept` move constructors and strong exception safety in vector resizing

---

## 1. Fixed vs. Dynamic Storage

### Definitions

**Core Definition**
Fixed storage means the container's size is determined at compile time and cannot change. Dynamic storage means the container can grow or shrink at runtime, managing memory automatically.

**Technical Definition**
`std::array<T, N>` is an aggregate type that wraps a built-in array of `N` elements of type `T`. Its size is a template parameter and is part of its type; no extra storage is reserved at runtime. `std::vector<T>` is a dynamic contiguous array that manages a dynamically allocated buffer, with amortized O(1) `push_back` and automatic reallocation when capacity is exceeded.

**Beginner-Friendly Explanation**
`std::array` is like a fixed-size egg carton — you decide how many eggs it holds when you make it, and that's it. `std::vector` is like a stretchable bag — you can keep adding items, and when it gets full, the bag automatically gets a bigger replacement.

---

### Sub-Feature: `std::array` (Fixed-Size Contiguous Storage)

#### Definitions

**Core Definition**
`std::array` is a fixed-size sequence container that encapsulates a C-style array with added member functions and iterators.

**Technical Definition**
`template<class T, std::size_t N> struct array;` (since C++11). It is an aggregate: it contains a single built-in array data member and no other data. `std::array` satisfies the ContiguousContainer requirement for `T` other than `bool`. Its size is known at compile time and is embedded in the type itself.

**Beginner-Friendly Explanation**
`std::array` is a safer, more convenient version of a C array. It knows its own size, supports iterators and algorithms, and never decays to a pointer when passed to a function.

#### Purposes

- **To** provide a zero-overhead abstraction over C-style arrays with added safety and functionality.
- **To** store a fixed number of elements contiguously on the stack (or wherever the object is placed).
- **To** enable use with standard algorithms and range-based for loops without losing size information.
- **To** avoid dynamic memory allocation entirely for fixed-size collections.

#### Syntax Rules and Structure

```cpp
template<class T, std::size_t N>
struct array;
```

**Component breakdown:**
- `T` — the element type.
- `N` — the number of elements (a compile-time constant).

**Common operations:**
- `arr.size()` — returns `N` (constexpr).
- `arr[i]` — element access (no bounds checking).
- `arr.at(i)` — element access with bounds checking.
- `arr.data()` — returns a pointer to the underlying array.
- `arr.fill(value)` — fills all elements with `value`.

**Constraints and limitations:**
- Size must be known at compile time. Cannot be resized.
- `std::array<T, 0>` is valid but has limited usefulness.
- Aggregate initialization requires braces: `std::array<int, 3> a = {1, 2, 3};`.
- C++20 added parenthesized aggregate initialization: `std::array<int, 3> a(1, 2, 3);`.

#### Annotated Code Example

```cpp
// File: array_example.cpp
// Compile with: g++ -std=c++17 array_example.cpp -o array_example

#include <array>
#include <iostream>
#include <algorithm>

int main() {
    // Step 1: Declare a fixed-size array of 5 integers
    std::array<int, 5> arr = {50, 20, 40, 10, 30};

    // Step 2: size() is constexpr — known at compile time
    std::cout << "Size: " << arr.size() << std::endl;

    // Step 3: Access elements with operator[]
    std::cout << "First element: " << arr[0] << std::endl;

    // Step 4: Use with standard algorithms
    std::sort(arr.begin(), arr.end());

    // Step 5: Range-based for loop
    std::cout << "Sorted: ";
    for (int v : arr) {
        std::cout << v << " ";
    }
    std::cout << std::endl;

    // Step 6: Bounds-checked access
    try {
        std::cout << arr.at(10) << std::endl;  // throws std::out_of_range
    } catch (const std::out_of_range& e) {
        std::cout << "Caught: " << e.what() << std::endl;
    }

    return 0;
}
```

**Expected Output:**
```
Size: 5
First element: 50
Sorted: 10 20 30 40 50
Caught: array::at: __n (which is 10) >= _Nm (which is 5)
```

**Why this output:** `arr.size()` returns the template parameter `N` (5). `arr[0]` accesses the first element. `std::sort` works because `std::array` provides random-access iterators. `arr.at(10)` throws `std::out_of_range` because index 10 is beyond the array's size.

#### Real-World Cases

- **Embedded systems**: Fixed-size buffers for sensor readings where dynamic allocation is forbidden.
- **Mathematical code**: Fixed-dimension vectors (3D points, 4×4 matrices) where size is part of the type.
- **Lookup tables**: Compile-time constant tables of strings or values.

---

### Sub-Feature: `std::vector` (Dynamic Contiguous Storage)

#### Definitions

**Core Definition**
`std::vector` is a dynamic sequence container that stores elements contiguously and can grow or shrink at runtime, managing memory automatically.

**Technical Definition**
`template<class T, class Allocator = std::allocator<T>> class vector;`. The elements are stored contiguously, meaning that for a `vector<T>` (where `T` is not `bool`), `&v[n] == &v[0] + n` for all `0 <= n < v.size()`. `std::vector` satisfies Container, AllocatorAwareContainer, SequenceContainer, ContiguousContainer (for non-`bool` `T`), and ReversibleContainer requirements.

**Beginner-Friendly Explanation**
`std::vector` is the default "go-to" container in C++. It's like an array that can grow. When you add an element and there's no room, it automatically allocates a bigger block, moves everything over, and continues. It's fast for most operations, especially adding at the end.

#### Purposes

- **To** provide a dynamic array with amortized O(1) insertion at the end.
- **To** offer contiguous storage for interoperability with C APIs and cache-friendly iteration.
- **To** serve as the default sequence container when no special requirements exist.
- **To** allow random access to any element in O(1) time.

#### Syntax Rules and Structure

```cpp
template<class T, class Allocator = std::allocator<T>>
class vector;
```

**Common operations:**
- `v.push_back(x)` — adds an element at the end (amortized O(1)).
- `v.pop_back()` — removes the last element (O(1)).
- `v.reserve(n)` — pre-allocates capacity for at least `n` elements.
- `v.shrink_to_fit()` — non-binding request to reduce capacity to fit size.
- `v.size()` — number of elements.
- `v.capacity()` — number of elements that can be held without reallocation.
- `v[i]` — element access (no bounds checking).

**Reallocation mechanics:**
When `push_back` is called and `size() == capacity()`, the vector allocates a new, larger buffer (typically 1.5× or 2× the current capacity — this is implementation-defined), transfers elements from the old buffer to the new one, and deallocates the old buffer. The growth factor is chosen to ensure amortized O(1) `push_back`.

**Exception safety during reallocation:**
- If the element's move constructor is `noexcept`, the vector moves elements (fast).
- If the move constructor may throw but the type is copy-insertable, the vector copies elements (slower but provides the strong exception guarantee).
- If neither, the vector may move and the strong exception guarantee is not provided.

**Constraints and limitations:**
- Reallocation invalidates all pointers, references, and iterators to elements.
- `std::vector<bool>` is a specialized template that packs bits; it does not provide the same guarantees as other vector specializations (no `data()` that returns `bool*`).
- Growth factor is implementation-defined; do not rely on a specific factor.
- Insertion or removal in the middle takes O(n) time.

#### Annotated Code Example

```cpp
// File: vector_example.cpp
// Compile with: g++ -std=c++17 vector_example.cpp -o vector_example

#include <vector>
#include <iostream>

int main() {
    // Step 1: Create an empty vector
    std::vector<int> v;

    // Step 2: Reserve capacity to avoid reallocations
    v.reserve(5);
    std::cout << "After reserve(5): size=" << v.size()
              << " capacity=" << v.capacity() << std::endl;

    // Step 3: Add elements
    for (int i = 1; i <= 5; ++i) {
        v.push_back(i * 10);
    }
    std::cout << "After 5 push_backs: size=" << v.size()
              << " capacity=" << v.capacity() << std::endl;

    // Step 4: Add one more — this triggers reallocation
    v.push_back(60);
    std::cout << "After 6th push_back: size=" << v.size()
              << " capacity=" << v.capacity() << std::endl;

    // Step 5: Access elements
    std::cout << "v[0]=" << v[0] << " v[5]=" << v[5] << std::endl;

    // Step 6: Contiguous storage — data() returns pointer to first element
    int* raw = v.data();
    std::cout << "raw[2]=" << raw[2] << std::endl;

    // Step 7: shrink_to_fit
    v.shrink_to_fit();
    std::cout << "After shrink_to_fit: capacity=" << v.capacity() << std::endl;

    return 0;
}
```

**Expected Output:**
```
After reserve(5): size=0 capacity=5
After 5 push_backs: size=5 capacity=5
After 6th push_back: size=6 capacity=10
v[0]=10 v[5]=60
raw[2]=30
After shrink_to_fit: capacity=6
```

**Why this output:** `reserve(5)` allocates space for 5 elements without changing size. After 5 `push_back`s, size equals capacity (5). The 6th `push_back` triggers reallocation; with GCC/libstdc++, the capacity doubles to 10. `data()` returns a pointer to the contiguous buffer. `shrink_to_fit` reduces capacity to match size (6).

#### Real-World Cases

- **General-purpose dynamic arrays**: The default choice for storing sequences of elements when size is not known in advance.
- **Image processing**: Storing pixel data in a contiguous buffer for cache-efficient algorithms.
- **Networking**: Building byte buffers for protocol serialization.
- **Interfacing with C APIs**: `v.data()` provides a pointer to a contiguous array compatible with C functions.

---

## 2. Double-Ended & Linked Lists

### Definitions

**Core Definition**
`std::deque` is a double-ended queue that allows O(1) insertion and deletion at both ends. `std::list` is a doubly-linked list supporting O(1) insertion and deletion anywhere given an iterator. `std::forward_list` is a singly-linked list that trades backward traversal for reduced memory overhead.

**Technical Definition**
`std::deque<T>` maintains an array of pointers to fixed-size blocks (often called a "map"), with elements stored in those blocks. `std::list<T>` is a doubly-linked list where each node contains a `T` and two pointers (prev, next). `std::forward_list<T>` is a singly-linked list where each node contains a `T` and one pointer (next).

**Beginner-Friendly Explanation**
`std::deque` is like a train with cars at both ends — you can add or remove cars at the front or back quickly. `std::list` is like a chain of boxes where each box knows both the previous and next box. `std::forward_list` is like a chain where each box only knows the next box — lighter but you can only go forward.

---

### Sub-Feature: `std::deque`

#### Definitions

**Core Definition**
`std::deque` (double-ended queue) is an indexed sequence container that allows fast insertion and deletion at both its beginning and its end.

**Technical Definition**
`template<class T, class Allocator = std::allocator<T>> class deque;`. Elements are not stored contiguously as a whole; instead, the deque maintains a "map" (array of pointers) to fixed-size blocks. Insertion and deletion at either end are amortized O(1). Random access is O(1). Insertion or deletion in the middle is O(n).

**Beginner-Friendly Explanation**
`std::deque` is like a book with pages glued in chunks. You can add or remove pages at the front or back easily, and you can jump to any page quickly. But inserting a page in the middle requires shifting everything.

#### Purposes

- **To** provide efficient insertion and deletion at both the front and the back.
- **To** support random access with O(1) complexity.
- **To** offer a middle ground between `std::vector` (fast at back only) and `std::list` (fast anywhere but poor cache locality).
- **To** serve as the default underlying container for `std::stack` and `std::queue`.

#### Syntax Rules and Structure

```cpp
template<class T, class Allocator = std::allocator<T>>
class deque;
```

**Common operations:**
- `dq.push_back(x)` / `dq.push_front(x)` — add at back/front (amortized O(1)).
- `dq.pop_back()` / `dq.pop_front()` — remove from back/front (O(1)).
- `dq[i]` — random access (O(1)).
- `dq.insert(pos, x)` — insert at position (O(n) for middle).

**Constraints and limitations:**
- Not contiguous as a whole; `&dq[i] != &dq[0] + i` across block boundaries.
- Block size is implementation-defined (typically 512 bytes in libstdc++, 4096 bytes in libc++, 16 bytes in MSVC).
- Reallocation of the map may occur, invalidating iterators but not references to existing elements.

#### Annotated Code Example

```cpp
// File: deque_example.cpp
// Compile with: g++ -std=c++17 deque_example.cpp -o deque_example

#include <deque>
#include <iostream>

int main() {
    // Step 1: Create a deque
    std::deque<int> dq;

    // Step 2: Push at both ends
    dq.push_back(10);
    dq.push_front(5);
    dq.push_back(20);
    dq.push_front(1);

    // Step 3: Iterate (from front to back)
    std::cout << "Deque: ";
    for (int v : dq) {
        std::cout << v << " ";
    }
    std::cout << std::endl;

    // Step 4: Random access
    std::cout << "dq[2] = " << dq[2] << std::endl;

    // Step 5: Pop from both ends
    dq.pop_front();
    dq.pop_back();
    std::cout << "After pops: ";
    for (int v : dq) {
        std::cout << v << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

**Expected Output:**
```
Deque: 1 5 10 20
dq[2] = 10
After pops: 5 10
```

**Why this output:** `push_front` adds to the front; `push_back` adds to the back. The deque maintains order. `dq[2]` accesses the third element (10) in O(1). `pop_front` removes 1, `pop_back` removes 20.

#### Real-World Cases

- **Task schedulers**: Adding tasks at both ends (high priority at front, normal at back).
- **Sliding window algorithms**: Efficiently maintaining a window of elements.
- **Undo/redo systems**: Pushing states at both ends.
- **`std::queue` and `std::stack`**: `std::deque` is the default container adapter.

---

### Sub-Feature: `std::list`

#### Definitions

**Core Definition**
`std::list` is a doubly-linked list that supports bidirectional iteration and O(1) insertion and deletion anywhere in the sequence.

**Technical Definition**
`template<class T, class Allocator = std::allocator<T>> class list;`. Each node stores a `T` and two pointers (previous and next). It provides bidirectional iterators. Compared to `std::forward_list`, it uses more memory per node but allows backward traversal.

**Beginner-Friendly Explanation**
`std::list` is like a chain where each link knows both its predecessor and successor. You can go forward or backward, and inserting or removing a link is fast if you're already at that link. But finding a link in the middle requires walking the chain.

#### Purposes

- **To** provide O(1) insertion and deletion anywhere in the sequence given an iterator.
- **To** support bidirectional iteration (forward and backward).
- **To** guarantee that iterators remain valid after insertion and deletion (except for erased elements).
- **To** offer `splice` for O(1) transfer of nodes between lists.

#### Syntax Rules and Structure

```cpp
template<class T, class Allocator = std::allocator<T>>
class list;
```

**Common operations:**
- `lst.push_back(x)` / `lst.push_front(x)` — add at back/front (O(1)).
- `lst.insert(pos, x)` — insert before `pos` (O(1)).
- `lst.erase(pos)` — remove element at `pos` (O(1)).
- `lst.splice(pos, other)` — transfer elements from `other` to `lst` (O(1) if same list; O(n) otherwise).
- `lst.sort()` — sorts the list (O(n log n)).
- `lst.merge(other)` — merges two sorted lists.

**Constraints and limitations:**
- No random access; accessing the nth element is O(n).
- Poor cache locality; each node is a separate heap allocation.
- Per-node overhead: two pointers (16 bytes on 64-bit) plus the element itself. For `std::list<int>`, node size is typically 24 bytes on a 64-bit system.

#### Annotated Code Example

```cpp
// File: list_example.cpp
// Compile with: g++ -std=c++17 list_example.cpp -o list_example

#include <list>
#include <iostream>

int main() {
    // Step 1: Create a list
    std::list<int> lst = {10, 20, 30, 40};

    // Step 2: Insert at position
    auto it = lst.begin();
    ++it;  // point to 20
    lst.insert(it, 15);  // insert 15 before 20

    // Step 3: Print the list
    std::cout << "List: ";
    for (int v : lst) {
        std::cout << v << " ";
    }
    std::cout << std::endl;

    // Step 4: Splice from another list
    std::list<int> other = {100, 200};
    lst.splice(lst.end(), other);  // transfer all elements from other
    std::cout << "After splice: ";
    for (int v : lst) {
        std::cout << v << " ";
    }
    std::cout << std::endl;
    std::cout << "other size: " << other.size() << std::endl;

    return 0;
}
```

**Expected Output:**
```
List: 10 15 20 30 40
After splice: 10 15 20 30 40 100 200
other size: 0
```

**Why this output:** `insert` adds 15 before 20. `splice` transfers all nodes from `other` to `lst` in O(1) time (since the lists are different, the standard allows linear time, but libstdc++ implements it in O(1) by relinking nodes). `other` becomes empty.

#### Real-World Cases

- **LRU caches**: A list plus a hash map for O(1) access and O(1) move-to-front.
- **Text editors**: Storing lines of text where insertion/deletion in the middle is frequent.
- **Adjacency lists**: Graph representations where edges are added/removed dynamically.
- **Splicing**: Moving elements between lists without copying or reallocating.

---

### Sub-Feature: `std::forward_list`

#### Definitions

**Core Definition**
`std::forward_list` is a singly-linked list that supports forward iteration and O(1) insertion and deletion after a given position.

**Technical Definition**
`template<class T, class Allocator = std::allocator<T>> class forward_list;`. Each node stores a `T` and a single pointer to the next node. It provides forward iterators. It trades backward traversal, `push_back`, and O(1) `size()` for 50% less pointer overhead per node compared to `std::list`.

**Beginner-Friendly Explanation**
`std::forward_list` is like a treasure hunt where each clue tells you only where the next clue is. You can only go forward, and you can't ask "how many clues are left?" But each clue is lighter than in a doubly-linked list.

#### Purposes

- **To** minimize per-node memory overhead for linked-list use cases.
- **To** provide O(1) insertion and deletion after a given position.
- **To** support forward-only traversal where backward traversal is unnecessary.
- **To** offer efficient `merge`, `sort`, and `splice_after` operations.

#### Syntax Rules and Structure

```cpp
template<class T, class Allocator = std::allocator<T>>
class forward_list;
```

**Common operations:**
- `fl.push_front(x)` — add at front (O(1)).
- `fl.insert_after(pos, x)` — insert after `pos` (O(1)).
- `fl.erase_after(pos)` — remove element after `pos` (O(1)).
- `fl.before_begin()` — iterator before the first element (needed for `insert_after`).
- `fl.merge(other)` — merge sorted lists.
- `fl.sort()` — sort the list.

**Constraints and limitations:**
- No `size()` member function by design.
- No `push_back`, `pop_back`, or backward iteration.
- Only forward iterators (not bidirectional or random-access).
- Per-node overhead: one pointer (8 bytes on 64-bit) plus the element.

#### Annotated Code Example

```cpp
// File: forward_list_example.cpp
// Compile with: g++ -std=c++17 forward_list_example.cpp -o forward_list_example

#include <forward_list>
#include <iostream>

int main() {
    // Step 1: Create a forward_list
    std::forward_list<int> fl;

    // Step 2: Insert elements in natural order using insert_after
    auto it = fl.before_begin();  // before the first element
    it = fl.insert_after(it, 10);
    it = fl.insert_after(it, 20);
    it = fl.insert_after(it, 30);

    // Step 3: Print
    std::cout << "forward_list: ";
    for (int v : fl) {
        std::cout << v << " ";
    }
    std::cout << std::endl;

    // Step 4: Insert at front
    fl.push_front(5);
    std::cout << "After push_front(5): ";
    for (int v : fl) {
        std::cout << v << " ";
    }
    std::cout << std::endl;

    // Step 5: Note: no size() member; use std::distance
    std::cout << "Size via distance: "
              << std::distance(fl.begin(), fl.end()) << std::endl;

    return 0;
}
```

**Expected Output:**
```
forward_list: 10 20 30
After push_front(5): 5 10 20 30
Size via distance: 4
```

**Why this output:** `insert_after` inserts after the given position. `before_begin()` returns an iterator to the position before the first element. `push_front` adds at the front. Since `forward_list` has no `size()`, we use `std::distance`.

#### Real-World Cases

- **Sparse data structures**: Representing sparse matrices or high-dimensional arrays with minimal memory overhead.
- **Memory-constrained systems**: Embedded systems where every byte counts.
- **Hash table buckets**: Chaining with singly-linked lists.
- **Forward-only pipelines**: Processing stages that only move forward.

---

## 3. Modern Multidimensional Views

### Definitions

**Core Definition**
`std::span` (C++20) is a non-owning view over a contiguous sequence of objects. `std::mdspan` (C++23) is a non-owning multidimensional view over a contiguous sequence, generalizing `std::span` to arbitrary rank.

**Technical Definition**
`template<class T, std::size_t Extent = std::dynamic_extent> class span;`. A span is a pointer-length pair that provides a view over contiguous memory. `template<class T, class Extents, class LayoutPolicy = std::layout_right, class AccessorPolicy = std::default_accessor<T>> class mdspan;`. An mdspan maps a multidimensional index to an element of a contiguous array, with configurable layout and accessor policies.

**Beginner-Friendly Explanation**
`std::span` is like a window into an existing array or vector — it doesn't copy anything, just lets you look at (and possibly modify) the underlying data safely with bounds-aware access. `std::mdspan` is like a window into a multidimensional grid, letting you treat a flat block of memory as a 2D, 3D, or N-dimensional array.

---

### Sub-Feature: `std::span` (C++20)

#### Definitions

**Core Definition**
`std::span` is a non-owning view over a contiguous sequence of objects, providing safe access and size information.

**Technical Definition**
A `span<T, Extent>` is essentially a `(T* data, size_t size)` pair. If `Extent` is `std::dynamic_extent`, the size is stored at runtime. If `Extent` is a positive integer, the size is known at compile time and stored in the type. `std::span` provides random-access iterators and supports subviews via `first()`, `last()`, and `subspan()`.

**Beginner-Friendly Explanation**
`std::span` is a lightweight way to pass a contiguous sequence to a function without copying it or losing its size. It replaces the old `(pointer, length)` pair idiom and C-style array decay.

#### Purposes

- **To** provide a non-owning view over contiguous memory without copying.
- **To** replace `(pointer, length)` function parameters with a safe, bounds-aware abstraction.
- **To** preserve size information when passing arrays to functions.
- **To** enable subviews (slices) over contiguous data.
- **To** serve as a vocabulary type for contiguous ranges.

#### Syntax Rules and Structure

```cpp
template<class T, std::size_t Extent = std::dynamic_extent>
class span;
```

**Component breakdown:**
- `T` — the element type (may be `const`).
- `Extent` — the number of elements; `std::dynamic_extent` for runtime size.

**Common operations:**
- `sp.size()` — number of elements.
- `sp[i]` — element access.
- `sp.data()` — pointer to the first element.
- `sp.first<N>()` / `sp.first(count)` — subspan of first N elements.
- `sp.last<N>()` / `sp.last(count)` — subspan of last N elements.
- `sp.subspan(offset, count)` — arbitrary subspan.

**Constraints and limitations:**
- Does not own the underlying data; the programmer must ensure the data outlives the span.
- Requires contiguous memory; cannot be used with `std::list` or `std::forward_list`.
- `std::span<const T>` is read-only; `std::span<T>` is mutable.
- A fixed-extent span can be constructed from a dynamic-extent span, but not vice versa (unless the dynamic size matches).

#### Annotated Code Example

```cpp
// File: span_example.cpp
// Compile with: g++ -std=c++20 span_example.cpp -o span_example

#include <span>
#include <iostream>
#include <vector>
#include <array>

// Step 1: Function taking a span (works with vector, array, C array)
void print_span(std::span<const int> sp) {
    std::cout << "Span (" << sp.size() << "): ";
    for (int v : sp) {
        std::cout << v << " ";
    }
    std::cout << std::endl;
}

int main() {
    // Step 2: Create spans from different sources
    int carr[] = {1, 2, 3, 4, 5};
    std::vector<int> vec = {10, 20, 30};
    std::array<int, 3> arr = {100, 200, 300};

    print_span(carr);  // C array
    print_span(vec);   // vector
    print_span(arr);   // std::array

    // Step 3: Subviews
    std::span<int> sp(carr);
    auto first3 = sp.first(3);
    auto last2 = sp.last(2);
    auto mid = sp.subspan(1, 3);  // offset 1, length 3

    print_span(first3);  // 1 2 3
    print_span(last2);   // 4 5
    print_span(mid);     // 2 3 4

    // Step 4: Modify through a mutable span
    std::span<int> mut(vec);
    mut[0] = 99;
    std::cout << "vec[0] after span modification: " << vec[0] << std::endl;

    return 0;
}
```

**Expected Output:**
```
Span (5): 1 2 3 4 5
Span (3): 10 20 30
Span (3): 100 200 300
Span (3): 1 2 3
Span (2): 4 5
Span (3): 2 3 4
vec[0] after span modification: 99
```

**Why this output:** `print_span` accepts any contiguous source. Subviews create new spans referencing the same underlying data. A mutable span allows modifying the original vector.

#### Real-World Cases

- **API boundaries**: Passing buffers to functions without copying or losing size.
- **Parsing and serialization**: Viewing slices of a byte buffer.
- **Numerical computing**: Passing matrix rows to BLAS/LAPACK routines.
- **String processing**: `std::span<const char>` as a non-owning string view (complementing `std::string_view`).

---

### Sub-Feature: `std::mdspan` (C++23)

#### Definitions

**Core Definition**
`std::mdspan` is a non-owning multidimensional view over a contiguous sequence of objects, mapping a multidimensional index to a linear offset.

**Technical Definition**
`template<class T, class Extents, class LayoutPolicy = std::layout_right, class AccessorPolicy = std::default_accessor<T>> class mdspan;`. `Extents` is a `std::extents` specialization that describes the size of each dimension (static or dynamic). `LayoutPolicy` controls how multidimensional indices map to linear offsets (`layout_right` for row-major, `layout_left` for column-major, `layout_stride` for custom strides). `AccessorPolicy` controls how elements are accessed.

**Beginner-Friendly Explanation**
`std::mdspan` lets you treat a flat block of memory (like a `std::vector`) as a 2D, 3D, or N-dimensional array. You can specify whether the data is stored row-major (C-style) or column-major (Fortran-style), and you can access elements with `md[i, j, k]` syntax.

#### Purposes

- **To** provide a multidimensional view over contiguous memory without copying.
- **To** support both static and dynamic extents (dimension sizes known at compile time or runtime).
- **To** allow configurable memory layouts (row-major, column-major, strided).
- **To** serve as a zero-overhead abstraction for multidimensional array access.
- **To** enable generic multidimensional algorithms.

#### Syntax Rules and Structure

```cpp
template<class T, class Extents,
         class LayoutPolicy = std::layout_right,
         class AccessorPolicy = std::default_accessor<T>>
class mdspan;
```

**Component breakdown:**
- `T` — element type.
- `Extents` — a `std::extents` specialization describing each dimension.
- `LayoutPolicy` — mapping policy (`layout_right`, `layout_left`, `layout_stride`).
- `AccessorPolicy` — element access policy (usually default).

**Common operations:**
- `ms.rank()` — number of dimensions.
- `ms.extent(i)` — size of dimension `i`.
- `ms[i, j, ...]` — element access (C++23 multidimensional subscript).
- `ms.data_handle()` — pointer to the underlying data.

**Constraints and limitations:**
- Requires C++23 or later.
- Does not own the data; programmer must manage lifetime.
- `layout_right` is row-major (last index varies fastest); `layout_left` is column-major.
- `layout_stride` allows arbitrary strides but requires careful setup to avoid overlapping or out-of-bounds access.

#### Annotated Code Example

```cpp
// File: mdspan_example.cpp
// Compile with: g++ -std=c++23 mdspan_example.cpp -o mdspan_example

#include <mdspan>
#include <vector>
#include <iostream>

int main() {
    // Step 1: Create a flat buffer of 12 elements
    std::vector<int> data = {1, 2, 3, 4, 5, 6,
                             7, 8, 9, 10, 11, 12};

    // Step 2: Create a 3x4 mdspan (row-major by default)
    std::mdspan<int, std::extents<int, 3, 4>> ms(data.data());

    // Step 3: Access elements with multidimensional subscript
    std::cout << "Element (0,0): " << ms[0, 0] << std::endl;
    std::cout << "Element (1,2): " << ms[1, 2] << std::endl;
    std::cout << "Element (2,3): " << ms[2, 3] << std::endl;

    // Step 4: Print the 3x4 matrix
    std::cout << "Matrix:" << std::endl;
    for (int i = 0; i < ms.extent(0); ++i) {
        for (int j = 0; j < ms.extent(1); ++j) {
            std::cout << ms[i, j] << " ";
        }
        std::cout << std::endl;
    }

    // Step 5: Column-major layout
    std::mdspan<int, std::extents<int, 3, 4>,
                std::layout_left> cm(data.data());
    std::cout << "Column-major (1,2): " << cm[1, 2] << std::endl;

    return 0;
}
```

**Expected Output:**
```
Element (0,0): 1
Element (1,2): 7
Element (2,3): 12
Matrix:
1 2 3 4
5 6 7 8
9 10 11 12
Column-major (1,2): 10
```

**Why this output:** With `layout_right` (row-major), `ms[1,2]` maps to linear index `1*4 + 2 = 6`, which is value 7. With `layout_left` (column-major), `cm[1,2]` maps to linear index `2*3 + 1 = 7`, which is value 10.

#### Real-World Cases

- **Linear algebra**: Treating vectors as matrices for BLAS/LAPACK.
- **Image processing**: Viewing a flat pixel buffer as a 2D image.
- **Scientific computing**: Multi-dimensional arrays in simulation code.
- **Machine learning**: Tensors as multidimensional views over contiguous memory.

---

## 4. Exception Safety Guarantees

### Definitions

**Core Definition**
Exception safety guarantees describe the state of a program or object after an exception is thrown. The three standard levels are basic, strong, and nothrow.

**Technical Definition**
- **Basic guarantee**: The invariants of the component are preserved, and no resources are leaked. Objects remain in a valid but unspecified state.
- **Strong guarantee**: The operation either succeeds completely or throws an exception, leaving the program state exactly as it was before the operation started (commit-or-rollback semantics).
- **Nothrow guarantee**: The function never throws exceptions.

`std::vector` provides the strong exception guarantee for `push_back` and `resize` under certain conditions related to the element type's move constructor's `noexcept` status.

**Beginner-Friendly Explanation**
When something goes wrong (an exception is thrown), you want your data to not be corrupted. The "basic" guarantee means nothing is leaked. The "strong" guarantee means it's as if the failed operation never happened. `std::vector` tries to provide the strong guarantee, but it can only do so efficiently if your type's move constructor is marked `noexcept`.

---

### Sub-Feature: `noexcept` Move Constructors and Vector Reallocation

#### Definitions

**Core Definition**
A `noexcept` move constructor is a move constructor declared with the `noexcept` specifier, promising not to throw exceptions. `std::vector` uses this information to decide whether to move or copy elements during reallocation.

**Technical Definition**
When `std::vector` needs to reallocate (e.g., during `push_back` when `size() == capacity()`), it uses `std::move_if_noexcept` to transfer existing elements to the new buffer. If the element type's move constructor is `noexcept` (i.e., `std::is_nothrow_move_constructible_v<T>` is `true`), the vector moves elements. Otherwise, if the type is copy-insertable, the vector copies elements to preserve the strong exception guarantee. If neither, the vector may move and the strong guarantee is not provided.

**Beginner-Friendly Explanation**
Imagine you're moving books from an old shelf to a new one. If you can move them without risk of dropping any (noexcept move), you move them quickly. If there's a risk of dropping (throwing move), you carefully copy each book instead, so if something goes wrong, the original shelf is still intact. Marking your move constructor `noexcept` tells `std::vector` it can safely use the fast path.

#### Purposes

- **To** enable `std::vector` to use fast move operations during reallocation instead of slow copies.
- **To** preserve the strong exception guarantee when moves may throw by falling back to copies.
- **To** signal to the compiler and library that a move operation is safe and cannot throw.
- **To** improve performance of container operations that rely on moving elements.

#### Syntax Rules and Structure

```cpp
class MyType {
public:
    // Move constructor declared noexcept
    MyType(MyType&& other) noexcept
        : data_(other.data_), size_(other.size_) {
        other.data_ = nullptr;
        other.size_ = 0;
    }
    // ...
private:
    int* data_;
    std::size_t size_;
};
```

**Component breakdown:**
- `MyType(MyType&& other)` — move constructor taking an rvalue reference.
- `noexcept` — promises the constructor will not throw.
- The body steals resources from `other` and leaves it in a valid, destructible state.

**Constraints and limitations:**
- If a `noexcept` function throws, `std::terminate` is called immediately — no stack unwinding.
- Only mark a move constructor `noexcept` if it truly cannot throw.
- The `noexcept` status affects `std::vector`'s choice between move and copy, which can have a dramatic performance impact (up to 84× difference in some benchmarks).

#### Annotated Code Example

```cpp
// File: noexcept_move_example.cpp
// Compile with: g++ -std=c++17 noexcept_move_example.cpp -o noexcept_move_example

#include <vector>
#include <iostream>
#include <string>

// Step 1: A type with a throwing move constructor (NO noexcept)
class SlowType {
public:
    std::string name;
    SlowType(std::string n) : name(std::move(n)) {}
    SlowType(const SlowType& other) : name(other.name) {
        std::cout << "  COPY constructed" << std::endl;
    }
    SlowType(SlowType&& other) /* no noexcept */
        : name(std::move(other.name)) {
        std::cout << "  MOVE constructed (throwing)" << std::endl;
    }
};

// Step 2: A type with a noexcept move constructor
class FastType {
public:
    std::string name;
    FastType(std::string n) : name(std::move(n)) {}
    FastType(const FastType& other) : name(other.name) {
        std::cout << "  COPY constructed" << std::endl;
    }
    FastType(FastType&& other) noexcept
        : name(std::move(other.name)) {
        std::cout << "  MOVE constructed (noexcept)" << std::endl;
    }
};

int main() {
    std::cout << "=== SlowType (throwing move) ===" << std::endl;
    std::vector<SlowType> slow;
    slow.reserve(1);  // force reallocation on second push
    slow.emplace_back("A");
    std::cout << "Pushing second element:" << std::endl;
    slow.emplace_back("B");  // triggers reallocation — will COPY

    std::cout << "\n=== FastType (noexcept move) ===" << std::endl;
    std::vector<FastType> fast;
    fast.reserve(1);
    fast.emplace_back("A");
    std::cout << "Pushing second element:" << std::endl;
    fast.emplace_back("B");  // triggers reallocation — will MOVE

    return 0;
}
```

**Expected Output:**
```
=== SlowType (throwing move) ===
  COPY constructed
Pushing second element:
  COPY constructed
  MOVE constructed (throwing)

=== FastType (noexcept move) ===
  MOVE constructed (noexcept)
Pushing second element:
  MOVE constructed (noexcept)
  MOVE constructed (noexcept)
```

**Why this output:** For `SlowType`, the move constructor is not `noexcept`, so `std::vector` copies elements during reallocation to preserve the strong exception guarantee. For `FastType`, the move constructor is `noexcept`, so the vector moves elements, which is much faster.

#### Real-World Cases

- **Resource-owning types**: A type that owns a heap-allocated buffer should have a `noexcept` move constructor, enabling fast vector reallocation.
- **`std::unique_ptr`**: Its move constructor is `noexcept`, making `std::vector<std::unique_ptr<T>>` efficient during reallocation.
- **High-performance computing**: Marking move constructors `noexcept` avoids accidental deep copies in vectors of large data structures.
- **Library design**: The C++ Core Guidelines recommend marking move operations `noexcept` when they cannot throw.

---

### Sub-Feature: Strong Exception Safety in `std::vector::push_back` and `resize`

#### Definitions

**Core Definition**
The strong exception guarantee for `push_back` means that if an exception is thrown during the operation, the vector is left unchanged.

**Technical Definition**
For `std::vector::push_back`, the standard guarantees that if an exception is thrown (by the allocator, the element's copy/move constructor, or assignment), the function has no effect — the vector remains in its original state — provided that `T` is `CopyInsertable` or `std::is_nothrow_move_constructible_v<T>` is `true`. If `T` is move-constructible but not copy-insertable and its move constructor may throw, the strong guarantee is not provided and the effects are unspecified.

For `std::vector::resize`, a similar guarantee applies. The libstdc++ implementation constructs new elements before moving existing ones so that if a default constructor throws, the existing elements are not left in a moved-from state.

**Beginner-Friendly Explanation**
If you're adding an element to a vector and something goes wrong (like running out of memory), the strong guarantee means the vector looks exactly like it did before you tried. This is like a transaction: either it fully succeeds, or it fully rolls back.

#### Purposes

- **To** ensure that container operations are atomic from the programmer's perspective.
- **To** prevent data loss or corruption when exceptions occur during insertion.
- **To** allow safe use of vectors in exception-prone code (e.g., allocating memory, constructing objects).
- **To** provide a predictable, reliable programming model for dynamic arrays.

#### Syntax Rules and Structure

The strong exception guarantee is not something you write — it is a guarantee provided by the standard library implementation, contingent on the element type's properties:

- **If `T` is `nothrow_move_constructible`**: Reallocation moves elements. Strong guarantee provided.
- **If `T` is `CopyInsertable` (but not nothrow-move)**: Reallocation copies elements. Strong guarantee provided.
- **If `T` is move-only and its move constructor may throw**: Reallocation moves elements. Strong guarantee NOT provided.

#### Annotated Code Example

```cpp
// File: strong_guarantee_example.cpp
// Compile with: g++ -std=c++17 strong_guarantee_example.cpp -o strong_guarantee_example

#include <vector>
#include <iostream>
#include <stdexcept>

// Step 1: A type that throws when copied after a certain count
class ThrowOnCopy {
    static int copy_count;
    int value_;
public:
    ThrowOnCopy(int v) : value_(v) {}
    ThrowOnCopy(const ThrowOnCopy& other) : value_(other.value_) {
        ++copy_count;
        if (copy_count > 2) {
            throw std::runtime_error("Copy limit exceeded");
        }
        std::cout << "  Copy constructed " << value_ << std::endl;
    }
    int value() const { return value_; }
};
int ThrowOnCopy::copy_count = 0;

int main() {
    std::vector<ThrowOnCopy> v;
    v.reserve(2);  // capacity 2
    v.emplace_back(1);
    v.emplace_back(2);

    std::cout << "Vector size: " << v.size() << std::endl;

    // Step 2: Try to add a third element — triggers reallocation
    // Since ThrowOnCopy has no noexcept move, vector will COPY.
    // The third copy will throw.
    try {
        std::cout << "Attempting push_back(3)..." << std::endl;
        v.emplace_back(3);
    } catch (const std::exception& e) {
        std::cout << "Caught: " << e.what() << std::endl;
    }

    // Step 3: Verify strong guarantee — vector unchanged
    std::cout << "After exception, size: " << v.size() << std::endl;
    std::cout << "Elements: ";
    for (const auto& elem : v) {
        std::cout << elem.value() << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

**Expected Output:**
```
Vector size: 2
Attempting push_back(3)...
  Copy constructed 1
  Copy constructed 2
Caught: Copy limit exceeded
After exception, size: 2
Elements: 1 2
```

**Why this output:** When `emplace_back(3)` triggers reallocation, the vector copies existing elements to the new buffer. The first two copies succeed, but the third (for the new element) throws. Because `ThrowOnCopy` is copy-insertable, the strong guarantee applies: the vector is left unchanged with its original two elements.

#### Real-World Cases

- **Financial systems**: Transactions that must not leave partial state.
- **Game engines**: Entity component systems where adding a component must be atomic.
- **Database engines**: Ensuring index updates are atomic.
- **Any code using `std::vector` in a try-catch block**: The strong guarantee provides predictable recovery.

---

## Summary Table

| Container | Storage | Access | Insert Front | Insert Back | Insert Middle | Cache Locality | Per-Element Overhead |
|---|---|---|---|---|---|---|---|
| `std::array` | Contiguous (fixed) | O(1) | N/A | N/A | N/A | Excellent | None |
| `std::vector` | Contiguous (dynamic) | O(1) | O(n) | Amortized O(1) | O(n) | Excellent | Minimal |
| `std::deque` | Block-based | O(1) | Amortized O(1) | Amortized O(1) | O(n) | Good | Block overhead |
| `std::list` | Doubly-linked | O(n) | O(1) | O(1) | O(1) | Poor | 2 pointers (16 bytes) |
| `std::forward_list` | Singly-linked | O(n) | O(1) | N/A | O(1)* | Poor | 1 pointer (8 bytes) |
| `std::span` | Non-owning view | O(1) | N/A | N/A | N/A | Excellent | None (view) |
| `std::mdspan` | Non-owning view | O(1) | N/A | N/A | N/A | Excellent | None (view) |

*Insert after a given position.

---

## References

- Sequence containers — cppreference.com - https://en.cppreference.com/w/cpp/container
- std::array — cppreference.com - https://en.cppreference.com/w/cpp/container/array
- std::vector — cppreference.com - https://en.cppreference.com/w/cpp/container/vector
- std::deque — cppreference.com - https://en.cppreference.com/w/cpp/container/deque
- std::list — cppreference.com - https://en.cppreference.com/w/cpp/container/list
- std::forward_list — cppreference.com - https://en.cppreference.com/w/cpp/container/forward_list
- std::span — cppreference.com - https://en.cppreference.com/w/cpp/container/span
- std::mdspan — cppreference.com - https://en.cppreference.com/w/cpp/container/mdspan
- std::move_if_noexcept — cppreference.com - https://en.cppreference.com/w/cpp/utility/move_if_noexcept
- std::is_nothrow_move_constructible — cppreference.com - https://en.cppreference.com/w/cpp/types/is_move_constructible
- C++ Core Guidelines: Provide noexcept guarantees on move operations — https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-move-noexcept
- Inside STL: The deque, implementation — Microsoft Learn - https://devblogs.microsoft.com/oldnewthing/20230810-00/?p=108584
- Use std::span Instead of C-style Arrays — Sandor Dargo - https://www.sandordargo.com/blog/2024/11/06/std-span
- Details of std::mdspan from C++23 — Bartlomiej Filipek - https://www.cppstories.com/2025/cpp23_mdspan/
- Exception Safety: Concepts and Techniques — Bjarne Stroustrup - https://www.stroustrup.com/except.pdf