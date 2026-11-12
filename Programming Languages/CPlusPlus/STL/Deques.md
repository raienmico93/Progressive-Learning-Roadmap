# C++ Deques — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
`std::deque` (double-ended queue) is an indexed sequence container in the C++ Standard Library that allows fast insertion and deletion at both its beginning and its end.

**Technical Definition**
`std::deque<T, Allocator = std::allocator<T>>` is a class template that satisfies the requirements of *Container*, *AllocatorAwareContainer*, *SequenceContainer*, and *ReversibleContainer*. Unlike `std::vector`, its elements are not stored contiguously; typical implementations use a sequence of individually allocated fixed-size arrays (blocks) with additional bookkeeping, which means indexed access requires two pointer dereferences. All member functions are `constexpr` since C++26.

**Beginner-Friendly Explanation**
Think of `std::deque` as a “two-way dynamic array.” You can add or remove items instantly at either the front or the back, and you can still access any item by its index. It achieves this by managing multiple memory blocks behind the scenes, so it never has to shift all existing elements when it grows at the front.

### Key Characteristics

- **O(1) insertion/removal at both ends** – `push_front`, `pop_front`, `push_back`, and `pop_back` are constant time.
- **O(1) random access** – `operator[]` and `at()` provide constant-time indexed access (with two pointer dereferences).
- **Reference stability at the ends** – insertion or deletion at either end never invalidates pointers or references to the rest of the elements.
- **Iterator invalidation** – while references remain stable, all iterators are invalidated by insertion at either end.
- **Non-contiguous storage** – elements are organised in fixed-size blocks; `data()` is not available.
- **Cheaper expansion than `vector`** – growth does not require copying existing elements to a new memory location.
- **Larger minimal memory footprint** – a deque holding one element may allocate a full internal array (e.g., 8× object size on libstdc++, or 16× / 4096 bytes on libc++).

### Prerequisites

- Basic C++ syntax (variables, types, control flow).
- Understanding of arrays and dynamic memory concepts.
- Familiarity with iterators and the range-based `for` loop.
- Awareness of Big-O notation and algorithmic complexity.

### Related Programming Areas

- **STL algorithms** – `std::sort`, `std::find`, and `std::transform` operate on deque iterators.
- **Container adaptors** – `std::stack` and `std::queue` use `std::deque` as their default underlying container.
- **Memory management** – block-based allocation strategy.
- **Performance optimisation** – cache locality and reference stability trade-offs.

### Core Concepts / Features

1. `std::deque` (the container itself)
2. Double-ended queue (underlying structure)
3. Deque operations
4. Deque performance
5. Deque best practices

---

## 1. `std::deque`

### Definitions

**Core Definition**
`std::deque` is an indexed sequence container that allows fast insertion and deletion at both its beginning and its end.

**Technical Definition**
The template is declared in `<deque>` as:
```cpp
template<class T, class Allocator = std::allocator<T>>
class deque;
```
It meets the requirements of *Container*, *AllocatorAwareContainer*, *SequenceContainer*, and *ReversibleContainer*. The storage is automatically expanded and contracted as needed.

**Beginner-Friendly Explanation**
`std::deque` is the standard C++ container that behaves like a dynamic array you can grow or shrink from both ends.

### Purposes

- To provide O(1) insertion and removal at both the front and the back.
- To offer random access by index.
- To preserve references to elements when modifying the ends.
- To serve as the default underlying container for `std::stack` and `std::queue`.
- To avoid the O(n) cost of front insertion that `std::vector` incurs.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <deque>
std::deque<T> dq;                     // empty
std::deque<T> dq(n);                  // n default-constructed elements
std::deque<T> dq(n, value);           // n copies of value
std::deque<T> dq = {a, b, c};        // initializer-list
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `T` | The type of elements stored. |
| `Allocator` | Optional allocator type (defaults to `std::allocator<T>`). |

**Syntax Rules**
- The header `<deque>` must be included.
- `std::deque` is defined in namespace `std`.
- Elements are value-initialised when a size is specified without a value.
- The container can be constructed from an initializer list, iterators, or another deque.

**Constraints and Limitations**
- `std::deque` does not provide `data()` because storage is not contiguous.
- Insertion in the middle is O(n) and invalidates all iterators and references.
- A deque holding a single element may allocate a large internal array.

### Annotated Code Examples

**Example 1: Creating a deque**

```cpp
#include <deque>
#include <iostream>

int main() {
    std::deque<int> dq = {10, 20, 30};
    std::cout << dq.size() << '\n';        // 3
    std::cout << dq[1] << '\n';            // 20
}
```

**Expected Output**
```
3
20
```

**Why this output?**  
The initializer-list constructor creates a deque with three elements. `size()` returns 3, and `dq[1]` accesses the second element (`20`).

**Example 2: Deque of a custom type**

```cpp
#include <deque>
#include <iostream>
#include <string>

struct Point { int x, y; };

int main() {
    std::deque<Point> pts = {{1, 2}, {3, 4}};
    std::cout << pts[1].x << ',' << pts[1].y << '\n';
}
```

**Expected Output**
```
3,4
```

**Why this output?**  
Aggregate initialisation constructs `Point` objects; `pts[1]` accesses the second point.

### Real-World Cases

- **Sliding-window algorithms** – efficient front removal and back insertion.
- **Task queues** – adding jobs at both ends.
- **Undo/redo systems** – double-ended access to history states.
- **Default for adaptors** – `std::stack` and `std::queue` use `deque` internally.

---

## 2. Double-Ended Queue (Underlying Structure)

### Definitions

**Core Definition**
A double-ended queue is a linear data structure that allows insertion and removal of elements at both ends in constant time.

**Technical Definition**
`std::deque` is typically implemented as a *map* (a dynamically allocated array of pointers) to fixed-size memory blocks (chunks). Each block holds a fixed number of elements (often 512 bytes / `sizeof(T)`). New blocks are added to the front or back of the map as needed. Expansion never moves existing elements; only pointers in the map are updated.

**Beginner-Friendly Explanation**
Imagine a bookshelf with removable shelves. You can add a new shelf at the left or right end without moving the books on other shelves. Similarly, a deque manages several memory blocks and can add a new block at either end without disturbing existing elements.

### Purposes

- To enable O(1) insertion and removal at both ends.
- To avoid copying existing elements during expansion.
- To provide random access with acceptable performance.
- To offer reference stability for elements not at the modified end.

### Syntax Rules and Structure

**Conceptual Layout**
```
[Block*] [Block*] [Block*] [Block*] [Block*]
   ↓         ↓         ↓         ↓         ↓
[a][b]    [c][d]    [e][f]    [g][h]    [i][ ]
                                           ^ end (spare space)
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Map | Dynamically allocated array of pointers to blocks. |
| Blocks | Fixed-size memory chunks holding elements. |
| Front/back growth | New blocks added to the ends of the map. |

**Syntax Rules**
- The block-based structure is an implementation detail; the standard specifies behaviour, not the exact layout.
- `push_front` may allocate a new block and add it to the front of the map.
- Elements are never relocated when the deque grows at either end.

**Constraints and Limitations**
- Indexed access requires two pointer dereferences (map → block → element), making it slightly slower than `vector`'s single dereference.
- Minimal memory cost is higher than `vector` for small deques.

### Annotated Code Example (Conceptual)

```cpp
// Conceptual illustration (not actual std::deque internals)
struct Block {
    int data[4];
};

struct Deque {
    Block** map;          // array of block pointers
    int front_index;      // index of first element
    int back_index;       // one past the last element
};

// push_front: place at --front_index, allocate new block if needed
// push_back: place at back_index++, allocate new block if needed
```

**Expected Output**  
(No output; conceptual illustration.)

### Real-World Cases

- **High-frequency trading systems** – rapid insertion/removal at both ends.
- **Cache implementations** – sliding windows over data streams.
- **Job schedulers** – adding urgent jobs at the front, normal jobs at the back.

---

## 3. Deque Operations

### Definitions

**Core Definition**
Deque operations are the member functions that allow insertion, removal, access, and manipulation of `std::deque` elements.

**Technical Definition**
`std::deque` provides the standard sequence-container interface: element access (`at`, `operator[]`, `front`, `back`), iterators (`begin`, `end`, `rbegin`, `rend`, etc.), capacity (`size`, `empty`, `max_size`, `shrink_to_fit`), and modifiers (`push_front`, `pop_front`, `push_back`, `pop_back`, `insert`, `emplace`, `erase`, `clear`, `resize`, `swap`).

**Beginner-Friendly Explanation**
`std::deque` gives you all the usual container commands, plus specialised commands for adding and removing items at the front.

### Key Operations

| Operation | Description | Complexity |
|-----------|-------------|------------|
| `push_front(value)` | Insert at the front | O(1) |
| `push_back(value)` | Insert at the back | O(1) |
| `pop_front()` | Remove the first element | O(1) |
| `pop_back()` | Remove the last element | O(1) |
| `emplace_front(args...)` | Construct in place at the front | O(1) |
| `emplace_back(args...)` | Construct in place at the back | O(1) |
| `insert(pos, value)` | Insert before `pos` | O(n) |
| `erase(pos)` | Remove element at `pos` | O(n) |
| `at(pos)` | Bounds-checked access | O(1) |
| `operator[]` | Unchecked access | O(1) |
| `front()` | Reference to first element | O(1) |
| `back()` | Reference to last element | O(1) |
| `size()` | Number of elements | O(1) |
| `empty()` | Whether the deque is empty | O(1) |
| `clear()` | Remove all elements | O(n) |
| `shrink_to_fit()` | Request reduction of memory usage | O(n) |
| `swap(other)` | Exchange contents | O(1) |

**Syntax Rules**
- `push_front` and `push_back` do not invalidate references to existing elements, but they do invalidate all iterators.
- `pop_front` and `pop_back` invalidate only iterators and references to the erased element (the past-the-end iterator may also be invalidated).
- `insert` and `erase` in the middle invalidate all iterators and references.
- `emplace_front` and `emplace_back` (since C++11) construct elements directly, avoiding temporaries.

**Constraints and Limitations**
- No `data()` member function because storage is not contiguous.
- `insert` and `erase` in the middle are O(n) and invalidate all iterators and references.
- `shrink_to_fit` is a non-binding request.

### Annotated Code Examples

**Example 1: Double-ended insertion and removal**

```cpp
#include <deque>
#include <iostream>

int main() {
    std::deque<int> dq;
    dq.push_back(2);                       // [2]
    dq.push_front(1);                      // [1, 2]
    dq.push_back(3);                       // [1, 2, 3]
    dq.push_front(0);                      // [0, 1, 2, 3]

    dq.pop_front();                        // [1, 2, 3]
    dq.pop_back();                         // [1, 2]

    for (int x : dq) std::cout << x << ' ';
}
```

**Expected Output**
```
1 2
```

**Why this output?**  
`push_front` and `push_back` add elements at the respective ends; `pop_front` and `pop_back` remove them.

**Example 2: Random access and `at()`**

```cpp
#include <deque>
#include <iostream>
#include <stdexcept>

int main() {
    std::deque<int> dq = {10, 20, 30};
    std::cout << dq[1] << '\n';            // 20
    std::cout << dq.at(2) << '\n';         // 30
    try {
        std::cout << dq.at(5) << '\n';     // throws
    } catch (const std::out_of_range&) {
        std::cout << "Out of range\n";
    }
}
```

**Expected Output**
```
20
30
Out of range
```

**Why this output?**  
`operator[]` and `at()` return the element at the given index; `at(5)` throws because index 5 is out of bounds.

**Example 3: `emplace_front` and `emplace_back`**

```cpp
#include <deque>
#include <iostream>
#include <string>

int main() {
    std::deque<std::string> dq;
    dq.emplace_back("world");             // construct in place at back
    dq.emplace_front("hello");            // construct in place at front
    for (const auto& s : dq) std::cout << s << ' ';
}
```

**Expected Output**
```
hello world
```

**Why this output?**  
`emplace_front` and `emplace_back` construct the strings directly in the deque’s storage, avoiding temporary objects.

### Real-World Cases

- **Sliding-window buffers** – `push_back` new data, `pop_front` old data.
- **BFS (breadth-first search)** – queue operations at both ends.
- **Event processing** – adding high-priority events at the front.

---

## 4. Deque Performance

### Definitions

**Core Definition**
Deque performance refers to the time and space complexity characteristics of `std::deque` operations.

**Technical Definition**
The standard mandates: random access O(1), insertion/removal at either end O(1), insertion/removal in the middle O(n). Expansion is cheaper than `vector` because existing elements are not copied. However, indexed access involves two pointer dereferences, and the minimal memory footprint is larger than `vector`'s.

**Beginner-Friendly Explanation**
Deques are fast at adding or removing items at the ends and fast at accessing items by index. They are slower than vectors for indexed access and use more memory when holding very few items.

### Complexity Summary

| Operation | Complexity | Notes |
|-----------|------------|-------|
| Random access | O(1) | Two pointer dereferences |
| `push_front` / `pop_front` | O(1) | References stable; iterators invalidated |
| `push_back` / `pop_back` | O(1) | References stable; iterators invalidated |
| `insert` / `erase` (middle) | O(n) | All iterators and references invalidated |
| `size` / `empty` | O(1) | |
| `clear` | O(n) | Destroys all elements |

### Comparison with `std::vector`

| Feature | `std::vector` | `std::deque` |
|---------|---------------|--------------|
| `push_back` | Amortised O(1) | Amortised O(1) |
| `push_front` | **O(n)** (shift all) | **O(1)** |
| `pop_front` | **O(n)** (shift all) | **O(1)** |
| Random access `[i]` | O(1) (one dereference) | O(1) (two dereferences) |
| Memory layout | Contiguous | Non-contiguous (chunked) |
| Cache performance | Excellent | Good but worse than `vector` |
| Reference stability on `push_back`/`push_front` | No (may invalidate) | **Yes** (refs stable) |
| `data()` pointer | Yes | **No** |
| `reserve()` | Yes | No |

*Source: adapted from*.

### Key Performance Factors

- **Cache locality** – deques have worse cache behaviour than vectors because blocks are not contiguous.
- **Expansion cost** – deques avoid copying elements during growth, making expansion cheaper than `vector`'s reallocation.
- **Memory overhead** – deques have a larger minimal memory cost, especially for small element counts.
- **Reference stability** – the key advantage: references to existing elements survive `push_front` and `push_back`.

### Real-World Cases

- **When front insertion dominates** – deques outperform vectors significantly.
- **When reference stability matters** – deques are preferred over vectors.
- **When contiguous memory is required** – vectors or arrays must be used instead.

---

## 5. Deque Best Practices

### Definitions

**Core Definition**
Deque best practices are guidelines for choosing and using `std::deque` appropriately, avoiding common pitfalls, and achieving correct and efficient code.

**Technical Definition**
These practices are grounded in the C++ Core Guidelines (SL.con.2: “Prefer `vector` by default unless you have a reason to use a different container”) and community consensus: use `deque` when you need efficient front insertion/removal or reference stability during end operations; otherwise prefer `vector` for its better cache behaviour.

**Beginner-Friendly Explanation**
Use `deque` when you frequently add or remove items at the front, or when you need references to elements to stay valid when you add or remove at the ends. Otherwise, `vector` is usually the better default.

### Key Guidelines

| Guideline | Summary |
|-----------|---------|
| **Prefer `vector` by default** | Contiguous memory gives better cache behaviour. |
| **Use `deque` for front insertion/removal** | When O(1) `push_front`/`pop_front` is essential. |
| **Use `deque` for reference stability** | When references must survive `push_back`/`push_front`. |
| **Do not assume contiguity** | `deque` has no `data()`; do not pass it to APIs expecting a contiguous buffer. |
| **Be aware of iterator invalidation** | `push_front`/`push_back` invalidate all iterators, even though references stay valid. |
| **Prefer `emplace_front`/`emplace_back`** | Construct in place to avoid temporaries. |
| **Consider `deque` for adaptors** | `stack` and `queue` default to `deque` for good reason. |
| **Benchmark before committing** | The theoretical advantages may be outweighed by cache effects in practice. |

### Annotated Code Example

```cpp
#include <deque>
#include <iostream>

void process(const std::deque<int>& dq) {   // const reference: no copy
    for (int x : dq) std::cout << x << ' ';
}

int main() {
    std::deque<int> dq;
    dq.push_front(1);                      // O(1)
    dq.push_back(2);                       // O(1)
    dq.emplace_front(0);                   // construct in place
    process(dq);
}
```

**Expected Output**
```
0 1 2
```

**Why this output?**  
`push_front` and `emplace_front` add elements at the front; `push_back` adds at the back. The function receives a const reference to avoid copying the deque.

### Real-World Cases

- **Code reviews** – checking that `deque` is used only when its advantages are needed.
- **Performance profiling** – confirming that front operations or reference stability actually matter.
- **Library design** – exposing `deque` when the interface requires double-ended operations.

---

## References

- `std::deque` — cppreference – https://en.cppreference.com/w/cpp/container/deque
- `std::deque` (printable) — cppreference – https://www.cppreference.com/cpp/container/deque
- `std::deque` — cppreference.dev – https://cppreference.dev/w/cpp/container/deque
- Standard library header `<deque>` — cppreference – https://en.cppreference.com/w/cpp/header/deque
- CTR51-CPP: Use valid references, pointers, and iterators — SEI CERT – https://wiki.sei.cmu.edu/confluence/display/cplusplus/CTR51-CPP.+Use+valid+references%2C+pointers%2C+and+iterators+to+reference+elements+of+a+container
- C++ Core Guidelines — SL.con: Containers – https://cpp-core-guidelines-docs.vercel.app/stdlib
- Know `std::deque` internals and when to prefer it over `vector` — Dr-Sergey/learn_cpp – https://github.com/Dr-Sergey/learn_cpp/blob/e2c54b8163b274e4f724168226553ae2ab4d7ddd/07_Standard_Library_Containers/Know_stddeque_internals_and_when_to_prefer_it_over_vector.md
- `deque::push_front` and `deque::pop_front` — Microsoft Learn – https://learn.microsoft.com/en-us/cpp/standard-library/deque-push-front-and-deque-pop-front
- Iterator invalidation rules for standard containers — Dr-Sergey/learn_cpp – https://raw.githubusercontent.com/Dr-Sergey/learn_cpp/refs/heads/main/07_Standard_Library_Containers/Understand_iterator_invalidation_rules_for_all_standard_containers.md
- C++ Working Draft (N4950) — Containers library – https://eel.is/c++draft/containers

---

This cheat sheet provides a comprehensive, structured reference for `std::deque`, covering the container itself, its underlying double-ended queue structure, operations, performance characteristics, comparisons with `std::vector`, and best practices, with annotated code examples, expected outputs, real-world cases, and authoritative references.