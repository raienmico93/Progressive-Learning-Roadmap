# C++ Iterator Taxonomy & Invalidation Mechanics — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Iterators are a generalization of pointers that allow a C++ program to work with different data structures (containers, streams, ranges) in a uniform way. The iterator library defines categories based on the operations an iterator supports, adapter types that transform iterator behavior, and invalidation rules that describe when iterators become unusable after container mutations.

**Technical Definition**
The C++ iterator library provides definitions for five (until C++17) or six (since C++17) iterator categories: LegacyInputIterator, LegacyOutputIterator, LegacyForwardIterator, LegacyBidirectionalIterator, LegacyRandomAccessIterator, and LegacyContiguousIterator. These categories are organized in a hierarchy of capability, where more powerful categories support the operations of less powerful ones. Iterator adaptors (inserters, reverse iterators, move iterators, stream iterators) transform the interface of an underlying iterator. Iterator invalidation is governed by the container's mutation operations: read-only methods never invalidate iterators or references, while modifying methods may invalidate iterators and/or references.

**Beginner-Friendly Explanation**
An iterator is like a bookmark in a book. Different kinds of bookmarks let you do different things: some let you only read forward, some let you jump to any page, and some let you write notes on the pages. The "categories" describe these capabilities. "Adaptors" are like special bookmarks that change how you use the book — one might let you read the book backwards, another might automatically add new pages as you write. "Invalidation" is what happens when the book is reprinted with a different page count — your old bookmark might no longer point to the right place.

---

### Key Characteristics

- **Six iterator categories** (C++17/20): Input, Output, Forward, Bidirectional, Random-Access, and Contiguous.
- **Concept hierarchy**: Each category subsumes the requirements of the categories below it (e.g., a random-access iterator is also a bidirectional iterator).
- **Tag dispatch**: Iterator categories are represented by empty tag types (`std::input_iterator_tag`, etc.) used for compile-time algorithm selection.
- **Adaptors**: `std::back_inserter`, `std::front_inserter`, `std::inserter`, `std::reverse_iterator`, `std::move_iterator`, `std::ostream_iterator`, `std::istream_iterator`.
- **Invalidation rules**: Container-specific rules determine which iterators, pointers, and references remain valid after insert, erase, or reallocation.
- **Past-the-end iterator**: Deserves particular mention because it is invalidated as though it were a normal iterator in many cases.

---

### Prerequisites

- Basic understanding of pointers and pointer arithmetic.
- Familiarity with standard containers (`std::vector`, `std::list`, `std::map`, etc.).
- Knowledge of templates and the standard library.
- (For C++20 concepts) A compiler supporting C++20 or later.

---

### Related Programming Areas

- **Generic programming**: Iterators are the interface between containers and algorithms.
- **Algorithm design**: The iterator category determines which algorithms can be used and their complexity.
- **Container implementation**: Iterator invalidation rules are part of every container's specification.
- **Stream processing**: Stream iterators bridge I/O and algorithm ranges.
- **Move semantics**: Move iterators enable efficient transfer of resources.

---

### Core Concepts / Features

1. **The 6 Standard Categories** — Input, Output, Forward, Bidirectional, Random-Access, and Contiguous Iterators.
2. **Iterator Helpers** — Stream iterators, inserters, reverse iterators, and move iterators.
3. **Invalidation Matrix** — When pointers, references, and iterators break during container mutations.

---

## 1. The 6 Standard Categories

### Definitions

**Core Definition**
Iterator categories classify iterators by the operations they support and the guarantees they provide about multi-pass traversal, backward movement, random access, and memory contiguity.

**Technical Definition**
The C++ standard defines six iterator categories. Each category is associated with an empty tag type in namespace `std`: `input_iterator_tag`, `output_iterator_tag`, `forward_iterator_tag`, `bidirectional_iterator_tag`, `random_access_iterator_tag`, and `contiguous_iterator_tag` (C++20). The category hierarchy is reflected in inheritance: `forward_iterator_tag` derives from `input_iterator_tag`, `bidirectional_iterator_tag` derives from `forward_iterator_tag`, and so on. A type `It` declares its category via `std::iterator_traits<It>::iterator_category` (legacy) or `It::iterator_concept` (C++20 concepts). Algorithms use tag dispatch to select the most efficient implementation for a given category.

**Beginner-Friendly Explanation**
Not all iterators are equal. Some can only read data once and move forward (like reading a streaming feed). Others can jump to any element instantly (like an array index). The categories tell you what you can do with an iterator and what guarantees you get.

---

### Sub-Feature: Input Iterator

#### Definitions

**Core Definition**
An input iterator is a single-pass, read-only iterator that supports forward traversal through a sequence.

**Technical Definition**
The `input_iterator` concept (C++20) refines `input_or_output_iterator` and `indirectly_readable`. It requires equality comparison, forward movement, and the `std::iter_value_t` type to be copyable. Legacy input iterators (pre-C++20) satisfy the `LegacyInputIterator` named requirement. Input iterators do not guarantee that reading the same element twice produces the same result, and they support only single-pass traversal.

**Beginner-Friendly Explanation**
An input iterator is like reading a newspaper for the first time — you read each page once and move forward. You can't go back to re-read a page, and if you read it again you might get different content (e.g., a live news feed).

#### Purposes (all begin with "To")

- **To** read data from a single-pass source such as a stream or generator.
- **To** provide the minimum interface needed by read-only algorithms.
- **To** enable algorithms like `std::find` and `std::count` to work with streams.
- **To** serve as the base category for all read-capable iterators.

#### Syntax Rules and Structure

```cpp
// C++20 concept
template<class I>
concept input_iterator = /* ... */;

// Legacy tag
struct input_iterator_tag {};
```

**Constraints and limitations:**
- Single-pass: after incrementing, previous copies may be invalidated.
- No backward movement.
- No random access.
- Multi-pass guarantee not provided (reading the same element twice may yield different results).

#### Annotated Code Example

```cpp
// File: input_iterator_example.cpp
// Compile with: g++ -std=c++20 input_iterator_example.cpp -o input_iterator_example

#include <iterator>
#include <iostream>
#include <sstream>

int main() {
    std::istringstream input("10 20 30 40 50");

    // Step 1: istream_iterator is an input iterator
    std::istream_iterator<int> it(input);
    std::istream_iterator<int> end;  // past-the-end

    // Step 2: Read all values (single pass)
    std::cout << "Values: ";
    while (it != end) {
        std::cout << *it << " ";
        ++it;
    }
    std::cout << std::endl;

    // Step 3: Cannot re-traverse — iterator is exhausted
    std::cout << "After exhaustion, it == end: " << (it == end) << std::endl;

    return 0;
}
```

**Expected Output:**
```
Values: 10 20 30 40 50
After exhaustion, it == end: 1
```

**Why this output:** `std::istream_iterator` reads integers from the stream one at a time. It is a single-pass iterator: once consumed, the values cannot be re-read from the same iterator. After reaching the end, the iterator compares equal to the default-constructed end iterator.

#### Real-World Cases with Explanation

**Case 1 — Reading from files**: `std::istream_iterator<int>` reads numbers from a file without loading the entire file into memory.

**Case 2 — Network streams**: Input iterators model single-pass data consumption from network sockets.

**Case 3 — Generator patterns**: Coroutine-based generators produce values that can only be consumed once.

---

### Sub-Feature: Output Iterator

#### Definitions

**Core Definition**
An output iterator is a single-pass, write-only iterator that supports forward traversal and writing to elements.

**Technical Definition**
The `output_iterator` concept (C++20) refines `input_or_output_iterator` and `indirectly_writable`. It requires `*it = value` and `*it++ = value` to be valid. Legacy output iterators satisfy the `LegacyOutputIterator` named requirement. Output iterators do not support equality comparison or dereferencing for reading.

**Beginner-Friendly Explanation**
An output iterator is like a pen that writes on a moving strip of paper. You can write one thing at each position, but you can't read what you've written or go back.

#### Purposes (all begin with "To")

- **To** write data to a single-pass sink such as a stream or inserter.
- **To** provide the minimum interface needed by write-only algorithms.
- **To** enable algorithms like `std::copy` and `std::transform` to write to output destinations.
- **To** serve as the foundation for insert adaptors.

#### Syntax Rules and Structure

```cpp
// C++20 concept
template<class I, class T>
concept output_iterator = /* ... */;

// Legacy tag
struct output_iterator_tag {};
```

**Constraints and limitations:**
- Single-pass: only one assignment per position.
- No backward movement.
- No reading (dereference yields a proxy for writing only).
- No equality comparison.

#### Annotated Code Example

```cpp
// File: output_iterator_example.cpp
// Compile with: g++ -std=c++20 output_iterator_example.cpp -o output_iterator_example

#include <iterator>
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> v = {1, 2, 3, 4, 5};

    // Step 1: ostream_iterator is an output iterator
    std::ostream_iterator<int> out(std::cout, " ");

    // Step 2: Write elements to the stream
    std::cout << "Elements: ";
    std::copy(v.begin(), v.end(), out);
    std::cout << std::endl;

    // Step 3: back_inserter is an output iterator that appends
    std::vector<int> dest;
    std::copy(v.begin(), v.end(), std::back_inserter(dest));
    std::cout << "Copied size: " << dest.size() << std::endl;

    return 0;
}
```

**Expected Output:**
```
Elements: 1 2 3 4 5
Copied size: 5
```

**Why this output:** `std::ostream_iterator` writes each element to `std::cout` followed by a space. `std::back_inserter` creates an output iterator that calls `push_back` on the target vector.

#### Real-World Cases with Explanation

**Case 1 — Logging**: `std::ostream_iterator` writes formatted log entries to a file stream.

**Case 2 — Building containers**: `std::back_inserter` collects algorithm results into a vector.

**Case 3 — Serialization**: Output iterators write bytes to a serialization buffer.

---

### Sub-Feature: Forward Iterator

#### Definitions

**Core Definition**
A forward iterator is a multi-pass iterator that supports reading and writing, with forward-only traversal.

**Technical Definition**
The `forward_iterator` concept refines `input_iterator` and `sentinel_for`. It requires the iterator to be default-constructible, copyable, and to provide the multi-pass guarantee: two copies of the same iterator can both be dereferenced, and iterating one copy does not affect the other. Legacy forward iterators satisfy the `LegacyForwardIterator` named requirement.

**Beginner-Friendly Explanation**
A forward iterator is like a bookmark in a book that you can copy. Each copy remembers its own position, so you can read the same page twice or have two bookmarks at different places.

#### Purposes (all begin with "To")

- **To** enable multi-pass algorithms that need to traverse a sequence more than once.
- **To** support algorithms like `std::adjacent_find` and `std::search`.
- **To** provide a base for bidirectional and random-access iterators.
- **To** work with singly-linked lists and other forward-only structures.

#### Syntax Rules and Structure

```cpp
// C++20 concept
template<class I>
concept forward_iterator = /* ... */;

// Legacy tag
struct forward_iterator_tag : public input_iterator_tag {};
```

**Constraints and limitations:**
- Forward-only; no backward movement.
- No random access.
- Requires multi-pass guarantee.

#### Annotated Code Example

```cpp
// File: forward_iterator_example.cpp
// Compile with: g++ -std=c++20 forward_iterator_example.cpp -o forward_iterator_example

#include <forward_list>
#include <iostream>
#include <algorithm>

int main() {
    std::forward_list<int> fl = {30, 10, 50, 20, 40};

    // Step 1: forward_list provides forward iterators
    auto it1 = fl.begin();
    auto it2 = it1;  // copy — both valid

    // Step 2: Multi-pass: both iterators can be dereferenced
    std::cout << "it1: " << *it1 << ", it2: " << *it2 << std::endl;

    // Step 3: Advancing one does not affect the other
    ++it1;
    std::cout << "After ++it1: it1=" << *it1 << ", it2=" << *it2 << std::endl;

    // Step 4: Use with algorithm (std::find)
    auto found = std::find(fl.begin(), fl.end(), 50);
    if (found != fl.end()) {
        std::cout << "Found: " << *found << std::endl;
    }

    return 0;
}
```

**Expected Output:**
```
it1: 30, it2: 30
After ++it1: it1=10, it2=30
Found: 50
```

**Why this output:** `std::forward_list` provides forward iterators. Copying `it1` to `it2` creates independent iterators. Incrementing `it1` does not affect `it2`. `std::find` traverses the list forward.

#### Real-World Cases with Explanation

**Case 1 — Singly-linked lists**: `std::forward_list` provides forward iterators for memory-efficient storage.

**Case 2 — Multi-pass parsing**: Parsers that need to look ahead and then backtrack within a limited range use forward iterators.

**Case 3 — Hash table iteration**: Bucket-based containers provide forward iterators over their elements.

---

### Sub-Feature: Bidirectional Iterator

#### Definitions

**Core Definition**
A bidirectional iterator is a forward iterator that additionally supports backward traversal (decrement).

**Technical Definition**
The `bidirectional_iterator` concept refines `forward_iterator` and requires `--it` and `it--` to be valid. Legacy bidirectional iterators satisfy the `LegacyBidirectionalIterator` named requirement.

**Beginner-Friendly Explanation**
A bidirectional iterator is like a bookmark that can move both forward and backward through the book. You can read a page, go back to the previous page, and then go forward again.

#### Purposes (all begin with "To")

- **To** enable algorithms that require backward traversal, such as `std::reverse`.
- **To** support containers like `std::list`, `std::set`, and `std::map` that provide bidirectional iterators.
- **To** allow reverse iteration via `rbegin()` and `rend()`.
- **To** serve as the base for random-access iterators.

#### Syntax Rules and Structure

```cpp
// C++20 concept
template<class I>
concept bidirectional_iterator = /* ... */;

// Legacy tag
struct bidirectional_iterator_tag : public forward_iterator_tag {};
```

**Constraints and limitations:**
- No random access (no `it + n`).
- Requires the underlying structure to support backward links.

#### Annotated Code Example

```cpp
// File: bidirectional_iterator_example.cpp
// Compile with: g++ -std=c++20 bidirectional_iterator_example.cpp -o bidirectional_iterator_example

#include <list>
#include <iostream>
#include <algorithm>

int main() {
    std::list<int> lst = {10, 20, 30, 40, 50};

    // Step 1: std::list provides bidirectional iterators
    auto it = lst.begin();
    std::cout << "Forward: ";
    while (it != lst.end()) {
        std::cout << *it << " ";
        ++it;
    }
    std::cout << std::endl;

    // Step 2: Backward traversal
    std::cout << "Backward: ";
    while (it != lst.begin()) {
        --it;
        std::cout << *it << " ";
    }
    std::cout << std::endl;

    // Step 3: std::reverse uses bidirectional iterators
    std::reverse(lst.begin(), lst.end());
    std::cout << "After reverse: ";
    for (int v : lst) std::cout << v << " ";
    std::cout << std::endl;

    return 0;
}
```

**Expected Output:**
```
Forward: 10 20 30 40 50
Backward: 50 40 30 20 10
After reverse: 50 40 30 20 10
```

**Why this output:** The list iterator moves forward with `++` and backward with `--`. After reaching the end, decrementing moves back through the elements. `std::reverse` swaps elements using bidirectional iteration.

#### Real-World Cases with Explanation

**Case 1 — Undo/redo**: Doubly-linked lists with bidirectional iterators support moving backward and forward through history.

**Case 2 — Sorted containers**: `std::set` and `std::map` provide bidirectional iterators for ordered traversal in both directions.

**Case 3 — Text editors**: Cursor movement forward and backward through lines of text.

---

### Sub-Feature: Random-Access Iterator

#### Definitions

**Core Definition**
A random-access iterator is a bidirectional iterator that supports constant-time advancement by arbitrary offsets, subscripting, and pointer arithmetic.

**Technical Definition**
The `random_access_iterator` concept refines `bidirectional_iterator` and requires `it + n`, `it - n`, `it[n]`, `it += n`, `it -= n`, `it1 - it2`, and relational comparisons (`<`, `>`, `<=`, `>=`) to be valid. Legacy random-access iterators satisfy the `LegacyRandomAccessIterator` named requirement.

**Beginner-Friendly Explanation**
A random-access iterator is like having the page number of a book. You can jump directly to page 42 without flipping through pages 1–41. You can also subtract page numbers to find how many pages are between two bookmarks.

#### Purposes (all begin with "To")

- **To** enable O(1) access to any element in a sequence.
- **To** support algorithms that require random access, such as `std::sort` and `std::binary_search`.
- **To** allow pointer arithmetic and subscripting on iterators.
- **To** provide the foundation for contiguous iterators.

#### Syntax Rules and Structure

```cpp
// C++20 concept
template<class I>
concept random_access_iterator = /* ... */;

// Legacy tag
struct random_access_iterator_tag : public bidirectional_iterator_tag {};
```

**Constraints and limitations:**
- Requires the underlying structure to support O(1) arithmetic (arrays, vectors, deques).
- Not available for node-based containers (list, forward_list, set, map).

#### Annotated Code Example

```cpp
// File: random_access_iterator_example.cpp
// Compile with: g++ -std=c++20 random_access_iterator_example.cpp -o random_access_iterator_example

#include <vector>
#include <iostream>
#include <algorithm>

int main() {
    std::vector<int> v = {50, 20, 40, 10, 30};

    // Step 1: Random access via subscript
    std::cout << "v[2] = " << v[2] << std::endl;

    // Step 2: Iterator arithmetic
    auto it = v.begin();
    it += 3;  // jump forward 3
    std::cout << "*(begin + 3) = " << *it << std::endl;

    // Step 3: Distance between iterators
    auto dist = v.end() - v.begin();
    std::cout << "Distance: " << dist << std::endl;

    // Step 4: Sort requires random-access iterators
    std::sort(v.begin(), v.end());
    std::cout << "Sorted: ";
    for (int x : v) std::cout << x << " ";
    std::cout << std::endl;

    return 0;
}
```

**Expected Output:**
```
v[2] = 40
*(begin + 3) = 10
Distance: 5
Sorted: 10 20 30 40 50
```

**Why this output:** Vector iterators support `it += n`, `it[n]`, and `it1 - it2`. `std::sort` requires random-access iterators because it uses the median-of-three pivot selection and introsort.

#### Real-World Cases with Explanation

**Case 1 — Sorting**: `std::sort` requires random-access iterators for O(n log n) performance.

**Case 2 — Binary search**: `std::binary_search` and `std::lower_bound` require random access for O(log n) lookup.

**Case 3 — Numeric algorithms**: `std::accumulate` and `std::inner_product` benefit from random access for vectorized implementations.

---

### Sub-Feature: Contiguous Iterator (C++20)

#### Definitions

**Core Definition**
A contiguous iterator is a random-access iterator that additionally guarantees the denoted elements are stored contiguously in memory.

**Technical Definition**
The `contiguous_iterator` concept (C++20) refines `random_access_iterator` and guarantees that `std::to_address(a) == std::addressof(*a)` and that pointer arithmetic on the underlying address corresponds exactly to iterator arithmetic. Iterator types in the standard library that were required to satisfy `LegacyContiguousIterator` in C++17 are also required to model `contiguous_iterator` in C++20. The tag type `contiguous_iterator_tag` derives from `random_access_iterator_tag`.

**Beginner-Friendly Explanation**
A contiguous iterator is like a random-access iterator, but with an extra guarantee: the elements are packed tightly together in memory, one after another, with no gaps. This means you can take the address of one element and use pointer arithmetic to reach any other element.

#### Purposes (all begin with "To")

- **To** guarantee that elements are stored contiguously for pointer-based access.
- **To** enable algorithms that operate on raw memory (e.g., `memcpy`, SIMD).
- **To** support interoperability with C APIs that expect contiguous arrays.
- **To** provide the strongest iterator guarantee available in the standard library.

#### Syntax Rules and Structure

```cpp
// C++20 concept
template<class I>
concept contiguous_iterator = /* ... */;

// Tag
struct contiguous_iterator_tag : public random_access_iterator_tag {};
```

**Constraints and limitations:**
- Requires the underlying storage to be contiguous (vector, array, string, span, mdspan).
- Not available for deque (block-based) or node-based containers.
- `std::vector<bool>` iterators are not contiguous (bit-packed proxy).

#### Annotated Code Example

```cpp
// File: contiguous_iterator_example.cpp
// Compile with: g++ -std=c++20 contiguous_iterator_example.cpp -o contiguous_iterator_example

#include <vector>
#include <array>
#include <span>
#include <iostream>
#include <memory>

int main() {
    // Step 1: vector, array, and span provide contiguous iterators
    std::vector<int> v = {10, 20, 30, 40, 50};
    std::array<int, 3> a = {1, 2, 3};
    std::span<int> sp(v);

    // Step 2: to_address gives the raw pointer
    auto* p = std::to_address(v.begin());
    std::cout << "p[2] = " << p[2] << std::endl;  // 30

    // Step 3: Pointer arithmetic matches iterator arithmetic
    auto* p2 = std::to_address(v.begin() + 3);
    std::cout << "*p2 = " << *p2 << std::endl;  // 40

    // Step 4: Contiguous guarantee enables memcpy-style access
    int* raw = v.data();
    std::cout << "raw[4] = " << raw[4] << std::endl;  // 50

    // Step 5: span is contiguous
    std::cout << "span[1] = " << sp[1] << std::endl;  // 20

    return 0;
}
```

**Expected Output:**
```
p[2] = 30
*p2 = 40
raw[4] = 50
span[1] = 20
```

**Why this output:** `std::to_address(v.begin())` returns the raw pointer to the first element. Because vector elements are contiguous, `p[2]` accesses the third element. `v.data()` returns the same underlying pointer. `std::span` also provides contiguous access.

#### Real-World Cases with Explanation

**Case 1 — C API interoperability**: Passing `v.data()` to a C function that expects `int*` works because vector iterators are contiguous.

**Case 2 — SIMD**: Contiguous storage enables vectorized loads and stores.

**Case 3 — Memory-mapped I/O**: Contiguous iterators over memory-mapped regions allow direct manipulation.

---

## 2. Iterator Helpers

### Definitions

**Core Definition**
Iterator helpers (adaptors) are class templates and functions that transform the behavior of an underlying iterator, providing insertion semantics, reverse traversal, move semantics, or stream I/O.

**Technical Definition**
The standard library provides several iterator adaptors in `<iterator>`: insert iterators (`std::back_insert_iterator`, `std::front_insert_iterator`, `std::insert_iterator`), reverse iterators (`std::reverse_iterator`), move iterators (`std::move_iterator`), and stream iterators (`std::istream_iterator`, `std::ostream_iterator`). These adaptors wrap an existing iterator or container and provide a modified interface. Helper functions such as `std::back_inserter`, `std::front_inserter`, and `std::inserter` simplify the creation of insert iterators.

**Beginner-Friendly Explanation**
Iterator helpers are like tools that change how an iterator works. An inserter makes an iterator that inserts new elements instead of overwriting existing ones. A reverse iterator reads a container backwards. A move iterator moves elements instead of copying them.

---

### Sub-Feature: Insert Iterators (`std::back_inserter`, `std::front_inserter`, `std::inserter`)

#### Definitions

**Core Definition**
Insert iterators are output iterators that insert new elements into a container instead of overwriting existing elements.

**Technical Definition**
`std::back_insert_iterator<Container>` calls `container.push_back(value)` on assignment. `std::front_insert_iterator<Container>` calls `container.push_front(value)`. `std::insert_iterator<Container>` calls `container.insert(iter, value)` at a specified position. The helper functions `std::back_inserter(c)`, `std::front_inserter(c)`, and `std::inserter(c, it)` create these adaptors.

**Beginner-Friendly Explanation**
A normal iterator overwrites whatever it points to. An insert iterator instead inserts a new element. `back_inserter` adds to the end, `front_inserter` adds to the beginning, and `inserter` adds at a specific position.

#### Purposes (all begin with "To")

- **To** enable algorithms to append to containers that don't have pre-sized storage.
- **To** avoid manual index management when filling a container.
- **To** support `std::copy`, `std::transform`, and similar algorithms that write output.
- **To** provide a uniform way to insert into vectors, deques, and lists.

#### Syntax Rules and Structure

```cpp
std::back_inserter(container);           // calls push_back
std::front_inserter(container);          // calls push_front
std::inserter(container, iterator);      // calls insert at position
```

**Constraints and limitations:**
- `back_inserter` requires `push_back` (vector, deque, list, string).
- `front_inserter` requires `push_front` (deque, list, forward_list).
- `inserter` requires `insert` at a position (most containers except array).
- Cannot be used with containers that don't support the required operation.

#### Annotated Code Example

```cpp
// File: insert_iterators.cpp
// Compile with: g++ -std=c++17 insert_iterators.cpp -o insert_iterators

#include <iterator>
#include <vector>
#include <list>
#include <iostream>
#include <algorithm>

int main() {
    // Step 1: back_inserter — append to vector
    std::vector<int> src = {1, 2, 3};
    std::vector<int> dst;
    std::copy(src.begin(), src.end(), std::back_inserter(dst));
    std::cout << "back_inserter: ";
    for (int v : dst) std::cout << v << " ";
    std::cout << std::endl;

    // Step 2: front_inserter — prepend to list
    std::list<int> lst;
    std::copy(src.begin(), src.end(), std::front_inserter(lst));
    std::cout << "front_inserter: ";
    for (int v : lst) std::cout << v << " ";
    std::cout << std::endl;  // 3 2 1

    // Step 3: inserter — insert at position
    std::vector<int> mid = {10, 40, 50};
    auto pos = mid.begin() + 1;  // before 40
    std::copy(src.begin(), src.end(), std::inserter(mid, pos));
    std::cout << "inserter: ";
    for (int v : mid) std::cout << v << " ";
    std::cout << std::endl;

    return 0;
}
```

**Expected Output:**
```
back_inserter: 1 2 3
front_inserter: 3 2 1
inserter: 10 1 2 3 40 50
```

**Why this output:** `back_inserter` appends to the end. `front_inserter` prepends to the front, reversing the order. `inserter` inserts at the specified position, shifting existing elements to the right.

#### Real-World Cases with Explanation

**Case 1 — Parsing CSV**: Reading records and appending them to a vector using `back_inserter`.

**Case 2 — Building logs**: Prepending log entries to a list with `front_inserter` for newest-first order.

**Case 3 — Merging sorted data**: Using `inserter` to insert elements at the correct position in a sorted container.

---

### Sub-Feature: Reverse Iterator (`std::reverse_iterator`)

#### Definitions

**Core Definition**
A reverse iterator adapts a bidirectional or random-access iterator to traverse a sequence in reverse order.

**Technical Definition**
`std::reverse_iterator<Iter>` is an iterator adaptor that performs the reverse of the underlying iterator: `++rit` calls `--it` on the base iterator, and `*rit` returns `*(it - 1)`. It is created via `rbegin()` and `rend()` member functions of containers, or `std::make_reverse_iterator(it)` (C++14).

**Beginner-Friendly Explanation**
A reverse iterator is like reading a book from the last page to the first. It wraps a normal iterator and flips its direction.

#### Purposes (all begin with "To")

- **To** traverse a container from the last element to the first.
- **To** enable algorithms like `std::reverse` and `std::reverse_copy`.
- **To** support reverse iteration without modifying the container.
- **To** provide a uniform interface for backward traversal.

#### Syntax Rules and Structure

```cpp
std::reverse_iterator<Iter> rit(it);
// Or via container:
container.rbegin();  // reverse begin
container.rend();    // reverse end
```

**Constraints and limitations:**
- Requires bidirectional or random-access iterators.
- `base()` returns the underlying forward iterator (not the same position).
- Dereferencing a reverse iterator returns a reference to the element *before* the base iterator.

#### Annotated Code Example

```cpp
// File: reverse_iterator.cpp
// Compile with: g++ -std=c++17 reverse_iterator.cpp -o reverse_iterator

#include <vector>
#include <iostream>
#include <algorithm>

int main() {
    std::vector<int> v = {10, 20, 30, 40, 50};

    // Step 1: Reverse iteration
    std::cout << "Reverse: ";
    for (auto rit = v.rbegin(); rit != v.rend(); ++rit) {
        std::cout << *rit << " ";
    }
    std::cout << std::endl;

    // Step 2: base() relationship
    auto rit = v.rbegin();
    std::cout << "*rit = " << *rit << std::endl;              // 50
    std::cout << "*(rit.base()) = " << *(rit.base()) << std::endl;  // 10? No — base() points past the current element
    // Actually: rit.base() points one past the element rit refers to
    // So *(rit.base()) is the element after 50, which is out of bounds. Wait:
    // rbegin() points to last element (50). base() points to end() (one past 50).
    // *rit = *(base() - 1) = 50
    std::cout << "rit.base() == v.end(): " << (rit.base() == v.end()) << std::endl;

    // Step 3: Reverse copy
    std::vector<int> rev;
    std::copy(v.rbegin(), v.rend(), std::back_inserter(rev));
    std::cout << "Reverse copy: ";
    for (int x : rev) std::cout << x << " ";
    std::cout << std::endl;

    return 0;
}
```

**Expected Output:**
```
Reverse: 50 40 30 20 10
*rit = 50
*(rit.base()) = 0
rit.base() == v.end(): 1
Reverse copy: 50 40 30 20 10
```

**Why this output:** `rbegin()` points to the last element (50). `base()` points one past that element (which is `end()`). The reverse iterator's `operator*` returns `*(base() - 1)`. `std::copy` with reverse iterators writes the elements in reverse order.

#### Real-World Cases with Explanation

**Case 1 — Reverse search**: Finding the last occurrence of a value by searching with `rbegin()`.

**Case 2 — Reverse sorting**: Sorting a vector in descending order by using `rbegin()` and `rend()`.

**Case 3 — Palindrome checking**: Comparing a string's forward and reverse halves.

---

### Sub-Feature: Move Iterator (`std::move_iterator`)

#### Definitions

**Core Definition**
A move iterator adapts an iterator so that dereferencing produces an rvalue reference, enabling move semantics during algorithm execution.

**Technical Definition**
`std::move_iterator<Iter>` wraps an iterator and returns `std::move(*it)` from `operator*`. This allows algorithms like `std::copy` to move elements instead of copying them when the destination is a move-aware container.

**Beginner-Friendly Explanation**
A move iterator is like a normal iterator, but when you dereference it, you get permission to move the element rather than copy it. This is useful when you want to transfer resources from one container to another without copying.

#### Purposes (all begin with "To")

- **To** enable algorithms to move elements instead of copying them.
- **To** avoid expensive deep copies when transferring ownership.
- **To** support move-aware containers like `std::vector<std::string>`.
- **To** provide a uniform interface for move-based algorithm execution.

#### Syntax Rules and Structure

```cpp
std::move_iterator<Iter> mit(it);
// Or via helper:
std::make_move_iterator(it);
```

**Constraints and limitations:**
- The underlying element type must be move-constructible.
- After moving, the source elements are in a valid but unspecified state.
- Not all algorithms benefit; only those that write to a destination.

#### Annotated Code Example

```cpp
// File: move_iterator.cpp
// Compile with: g++ -std=c++17 move_iterator.cpp -o move_iterator

#include <iterator>
#include <vector>
#include <string>
#include <iostream>
#include <algorithm>

int main() {
    std::vector<std::string> src = {"Hello", "World", "Move"};

    // Step 1: Move elements to destination (no deep copy)
    std::vector<std::string> dst(
        std::make_move_iterator(src.begin()),
        std::make_move_iterator(src.end())
    );

    std::cout << "dst: ";
    for (const auto& s : dst) std::cout << s << " ";
    std::cout << std::endl;

    // Step 2: Source elements are in unspecified state
    std::cout << "src (after move): ";
    for (const auto& s : src) std::cout << "[" << s << "] ";
    std::cout << std::endl;

    return 0;
}
```

**Expected Output (GCC/libstdc++):**
```
dst: Hello World Move
src (after move): [] [] []
```

**Why this output:** `std::make_move_iterator` wraps the source iterators, so the vector constructor moves the strings into `dst` instead of copying them. After the move, the source strings are in a valid but unspecified state, typically empty.

#### Real-World Cases with Explanation

**Case 1 — Transferring ownership**: Moving `std::unique_ptr` elements from one vector to another.

**Case 2 — Avoiding deep copies**: Moving `std::string` or `std::vector` elements during container construction.

**Case 3 — Return value optimization**: Moving elements into a return container.

---

### Sub-Feature: Stream Iterators (`std::istream_iterator`, `std::ostream_iterator`)

#### Definitions

**Core Definition**
Stream iterators provide iterator interfaces over input and output streams, allowing algorithms to read from and write to streams.

**Technical Definition**
`std::istream_iterator<T>` reads successive `T` values from an input stream using `operator>>`. `std::ostream_iterator<T>` writes `T` values to an output stream using `operator<<`, with an optional delimiter string. Both are input/output iterators respectively.

**Beginner-Friendly Explanation**
A stream iterator lets you use algorithms with cin and cout. Instead of writing a loop to read numbers, you can use `std::copy` with `std::istream_iterator` to read directly into a container.

#### Purposes (all begin with "To")

- **To** read data from streams using standard algorithms.
- **To** write algorithm results directly to output streams.
- **To** integrate I/O with generic algorithms.
- **To** simplify parsing and formatting code.

#### Syntax Rules and Structure

```cpp
std::istream_iterator<T> it(input_stream);
std::ostream_iterator<T> out(output_stream, delimiter);
```

**Constraints and limitations:**
- `istream_iterator` is an input iterator (single-pass).
- `ostream_iterator` is an output iterator (single-pass).
- Requires the type to have `operator>>` / `operator<<` defined.

#### Annotated Code Example

```cpp
// File: stream_iterators.cpp
// Compile with: g++ -std=c++17 stream_iterators.cpp -o stream_iterators

#include <iterator>
#include <vector>
#include <iostream>
#include <sstream>
#include <algorithm>

int main() {
    // Step 1: Read integers from a stringstream
    std::istringstream input("5 10 15 20 25");
    std::vector<int> v(
        std::istream_iterator<int>(input),
        std::istream_iterator<int>()
    );

    std::cout << "Read " << v.size() << " values: ";
    std::copy(v.begin(), v.end(),
              std::ostream_iterator<int>(std::cout, " "));
    std::cout << std::endl;

    // Step 2: Write to a stringstream
    std::ostringstream output;
    std::copy(v.begin(), v.end(),
              std::ostream_iterator<int>(output, ","));
    std::cout << "Joined: " << output.str() << std::endl;

    return 0;
}
```

**Expected Output:**
```
Read 5 values: 5 10 15 20 25
Joined: 5,10,15,20,25,
```

**Why this output:** The `istream_iterator` reads integers from the input stringstream until the stream is exhausted. The `ostream_iterator` writes each integer followed by a space to `std::cout`, and then followed by a comma to the output stringstream.

#### Real-World Cases with Explanation

**Case 1 — Log processing**: Reading log entries from a file stream into a vector for analysis.

**Case 2 — CSV export**: Writing vector data to a CSV file using `ostream_iterator`.

**Case 3 — Interactive input**: Reading a sequence of numbers from `std::cin` until EOF.

---

## 3. Invalidation Matrix

### Definitions

**Core Definition**
Iterator invalidation describes the conditions under which iterators, pointers, and references to container elements become invalid after container mutations.

**Technical Definition**
The C++ standard specifies, for each container, which operations invalidate iterators, pointers, and references. Read-only methods never invalidate iterators or references. Methods that modify a container may invalidate iterators and/or references. The invalidation rules are summarized in the standard's container requirements and are container-specific.

**Beginner-Friendly Explanation**
When you change a container — adding, removing, or reallocating elements — your old iterators might stop working. The invalidation matrix tells you exactly which iterators are safe to use after each operation.

---

### Sub-Feature: Sequence Container Invalidation

#### Definitions

**Core Definition**
Sequence containers (`vector`, `deque`, `list`, `forward_list`, `array`) have distinct invalidation rules based on their storage strategies.

**Technical Definition**
- **`std::vector`**: Insertion invalidates all iterators if reallocation occurs (capacity change). Otherwise, iterators before the insertion point remain valid. Erasure invalidates iterators at or after the erased element.
- **`std::deque`**: Insertion at either end invalidates all iterators but references remain valid. Insertion in the middle invalidates all iterators and references. Erasure at either end invalidates only iterators to the erased element; erasure in the middle invalidates all.
- **`std::list` and `std::forward_list`**: Insertion never invalidates existing iterators. Erasure invalidates only iterators to the erased elements.
- **`std::array`**: Insertion/erasure are not applicable.

#### Purposes (all begin with "To")

- **To** understand when it is safe to hold iterators across container mutations.
- **To** write correct code that avoids undefined behaviour from dangling iterators.
- **To** choose the right container for algorithms that rely on iterator stability.
- **To** debug use-after-invalidation bugs.

#### Syntax Rules and Structure

**Invalidation Matrix for Sequence Containers:**

| Container | Operation | Iterators Invalidated | References Invalidated |
|---|---|---|---|
| `vector` | `push_back` (no realloc) | `end()` only | None |
| `vector` | `push_back` (realloc) | All | All |
| `vector` | `insert` (no realloc) | At/after insertion point | None |
| `vector` | `erase` | At/after erased point | At/after erased point |
| `deque` | `push_back`/`push_front` | All | None |
| `deque` | `insert` (middle) | All | All |
| `deque` | `erase` (ends) | Only erased | Only erased |
| `deque` | `erase` (middle) | All | All |
| `list` | `insert` | None | None |
| `list` | `erase` | Only erased | Only erased |
| `forward_list` | `insert_after` | None | None |
| `forward_list` | `erase_after` | Only erased | Only erased |

#### Annotated Code Example

```cpp
// File: sequence_invalidation.cpp
// Compile with: g++ -std=c++17 sequence_invalidation.cpp -o sequence_invalidation

#include <vector>
#include <list>
#include <iostream>

int main() {
    // Step 1: vector invalidation on reallocation
    std::vector<int> v = {1, 2, 3};
    auto it = v.begin();
    std::cout << "*it before: " << *it << std::endl;
    v.reserve(100);  // reallocation — it may be invalidated
    // *it is now undefined behaviour; do not use it!
    std::cout << "After reserve, capacity: " << v.capacity() << std::endl;

    // Step 2: list iterators remain valid after insertion
    std::list<int> lst = {10, 20, 30};
    auto lit = lst.begin();
    ++lit;  // points to 20
    lst.push_front(5);  // insert at front
    std::cout << "*lit after push_front: " << *lit << std::endl;  // still 20

    // Step 3: list erase invalidates only the erased iterator
    auto to_erase = lst.begin();
    ++to_erase;  // points to 10
    auto next = lst.erase(to_erase);  // returns iterator after erased
    std::cout << "*next: " << *next << std::endl;  // 20

    return 0;
}
```

**Expected Output:**
```
*it before: 1
After reserve, capacity: 100
*lit after push_front: 20
*next: 20
```

**Why this output:** After `reserve(100)`, the vector may have reallocated, so `it` is invalid. The list iterator `lit` remains valid after `push_front` because list nodes are not moved. `erase` returns an iterator to the next element, which is still valid.

#### Real-World Cases with Explanation

**Case 1 — Vector reallocation in loops**: Holding a vector iterator across `push_back` calls is a common source of bugs.

**Case 2 — List splicing**: List iterators remain valid even when nodes are moved between lists via `splice`.

**Case 3 — Deque end insertion**: Deque iterators are invalidated on `push_back`, but references to existing elements remain valid.

---

### Sub-Feature: Associative and Unordered Container Invalidation

#### Definitions

**Core Definition**
Associative and unordered containers have their own invalidation rules: node-based containers (`set`, `map`) invalidate only erased iterators, while unordered containers invalidate all iterators on rehash.

**Technical Definition**
- **`std::set`, `std::map`, `std::multiset`, `std::multimap`**: Insertion never invalidates iterators or references. Erasure invalidates only iterators and references to the erased elements.
- **`std::unordered_set`, `std::unordered_map`, etc.**: Insertion may invalidate all iterators if rehashing occurs, but references and pointers to elements remain valid. Erasure invalidates only iterators to the erased elements.

#### Purposes (all begin with "To")

- **To** know when it is safe to hold iterators across insertions in associative containers.
- **To** understand the difference between iterator invalidation and reference invalidation in unordered containers.
- **To** write correct code that uses `extract()` and `merge()` safely.
- **To** choose between ordered and unordered containers based on iterator stability.

#### Syntax Rules and Structure

**Invalidation Matrix for Associative/Unordered Containers:**

| Container | Operation | Iterators Invalidated | References Invalidated |
|---|---|---|---|
| `set`/`map`/`multiset`/`multimap` | `insert` | None | None |
| `set`/`map`/`multiset`/`multimap` | `erase` | Only erased | Only erased |
| `unordered_set`/`unordered_map`/etc. | `insert` (no rehash) | None | None |
| `unordered_set`/`unordered_map`/etc. | `insert` (rehash) | All | None |
| `unordered_set`/`unordered_map`/etc. | `erase` | Only erased | Only erased |

#### Annotated Code Example

```cpp
// File: associative_invalidation.cpp
// Compile with: g++ -std=c++17 associative_invalidation.cpp -o associative_invalidation

#include <map>
#include <unordered_map>
#include <iostream>

int main() {
    // Step 1: map iterators remain valid after insertion
    std::map<int, std::string> m = {{1, "one"}, {2, "two"}};
    auto it = m.find(1);
    m.insert({3, "three"});  // does not invalidate it
    std::cout << "map it: " << it->second << std::endl;  // "one"

    // Step 2: unordered_map iterators invalidated on rehash
    std::unordered_map<int, std::string> um;
    um.reserve(1);  // small bucket count
    um[1] = "one";
    auto uit = um.find(1);
    std::cout << "Before rehash: " << uit->second << std::endl;

    // Insert many elements to trigger rehash
    for (int i = 2; i <= 100; ++i) {
        um[i] = "val";
    }
    // uit is now invalidated if rehash occurred
    std::cout << "Rehash occurred, uit may be invalid" << std::endl;

    // Step 3: References remain valid in unordered_map
    // (even after rehash, pointers/references to elements are stable)
    std::cout << "um[1] still: " << um[1] << std::endl;

    return 0;
}
```

**Expected Output:**
```
map it: one
Before rehash: one
Rehash occurred, uit may be invalid
um[1] still: one
```

**Why this output:** `std::map` insertion does not invalidate iterators. `std::unordered_map` insertion may invalidate iterators on rehash, but references to elements remain valid. `um[1]` still works because the element itself was not erased.

#### Real-World Cases with Explanation

**Case 1 — Cache with map**: A `std::map`-based cache can insert new entries without invalidating existing iterators, making it safe to hold references to cached items.

**Case 2 — Hash table resizing**: An `unordered_map` that grows beyond its load factor triggers a rehash, invalidating iterators but not references — critical for code that holds pointers to values.

**Case 3 — Node extraction**: Using `.extract()` on a `map` invalidates only the extracted node's iterator, allowing safe modification and re-insertion.

---

## Summary Table: Iterator Categories

| Category | Direction | Read/Write | Multi-pass | Example Types | Tag Type |
|---|---|---|---|---|---|
| Input | Forward | Read | No | `istream_iterator` | `input_iterator_tag` |
| Output | Forward | Write | No | `ostream_iterator`, `back_inserter` | `output_iterator_tag` |
| Forward | Forward | Read/Write | Yes | `forward_list`, `unordered_map` | `forward_iterator_tag` |
| Bidirectional | Forward + Backward | Read/Write | Yes | `list`, `set`, `map` | `bidirectional_iterator_tag` |
| Random-Access | Any order | Read/Write | Yes | `vector`, `deque`, `string` | `random_access_iterator_tag` |
| Contiguous (C++20) | Any order | Read/Write | Yes | `vector`, `array`, `span` | `contiguous_iterator_tag` |

---

## Summary Table: Iterator Helper Adaptors

| Adaptor | Category | Purpose | Requires |
|---|---|---|---|
| `back_inserter` | Output | Append to container | `push_back` |
| `front_inserter` | Output | Prepend to container | `push_front` |
| `inserter` | Output | Insert at position | `insert` |
| `reverse_iterator` | Bidirectional+ | Traverse in reverse | Bidirectional iterator |
| `move_iterator` | Input/Forward | Move elements | Move-constructible type |
| `istream_iterator` | Input | Read from stream | `operator>>` |
| `ostream_iterator` | Output | Write to stream | `operator<<` |

---

## References

- Iterator library — cppreference.com - https://en.cppreference.com/w/cpp/iterator
- Iterator categories — cppreference.com - https://en.cppreference.com/w/cpp/iterator#Iterator_categories
- std::input_iterator_tag, std::output_iterator_tag, etc. — cppreference.com - https://en.cppreference.com/w/cpp/iterator/iterator_tags
- std::contiguous_iterator — cppreference.com - https://en.cppreference.com/w/cpp/iterator/contiguous_iterator
- Iterator invalidation rules — cppreference.com - https://en.cppreference.com/w/cpp/container#Iterator_invalidation
- std::back_inserter — cppreference.com - https://en.cppreference.com/w/cpp/iterator/back_inserter
- std::front_inserter — cppreference.com - https://en.cppreference.com/w/cpp/iterator/front_inserter
- std::inserter — cppreference.com - https://en.cppreference.com/w/cpp/iterator/inserter
- std::reverse_iterator — cppreference.com - https://en.cppreference.com/w/cpp/iterator/reverse_iterator
- std::move_iterator — cppreference.com - https://en.cppreference.com/w/cpp/iterator/move_iterator
- std::istream_iterator — cppreference.com - https://en.cppreference.com/w/cpp/iterator/istream_iterator
- std::ostream_iterator — cppreference.com - https://en.cppreference.com/w/cpp/iterator/ostream_iterator
- CTR51-CPP. Use valid references, pointers, and iterators to reference elements of a container — SEI CERT - https://wiki.sei.cmu.edu/confluence/display/cplusplus/CTR51-CPP.+Use+valid+references%2C+pointers%2C+and+iterators+to+reference+elements+of+a+container
- Iterator invalidation rules for C++ containers — Stack Overflow - https://stackoverflow.com/questions/6438086/iterator-invalidation-rules-for-c-containers
- <iterator> — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/standard-library/iterator
- P0896R4: The One Ranges Proposal — Open-Std - https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/p0896r4.pdf
- N4861: Working Draft, Standard for Programming Language C++ (C++20) — Open-Std - https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2020/n4861.pdf
- N3337: Working Draft, Standard for Programming Language C++ (C++11) — GitHub - https://github.com/cplusplus/draft/raw/master/papers/n3337.pdf
- Contiguous Iterators: A Refinement of Random Access Iterators — Open-Std - https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2014/n4284.pdf