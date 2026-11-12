# C++ Iterators — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
An iterator is an object that provides uniform access to elements of a sequence, acting as a generalized pointer that allows algorithms to operate on different data structures without knowing their internal layout.

**Technical Definition**
Iterators are a generalization of pointers that allow a C++ program to work with different data structures (containers and ranges) in a uniform manner. The iterator library provides definitions for iterators, as well as iterator traits, adaptors, and utility functions. Since iterators are an abstraction of pointers, their semantics are a generalization of most of the semantics of pointers in C++, ensuring that every function template that takes iterators works as well with regular pointers. Iterators are categorized by the operations they support, not by specific types—any type that supports the necessary operations can be used as an iterator. For example, a pointer supports all operations required by a random-access iterator, so a pointer can be used wherever a random-access iterator is expected.

**Beginner-Friendly Explanation**
Think of an iterator as a “smart bookmark” that points to an element in a container. You can move the bookmark forward (and sometimes backward or by leaps), read the value it points to, and sometimes write a new value. Different containers provide different kinds of bookmarks: a vector’s bookmark can jump instantly to any position, while a list’s bookmark can only step forward or backward one element at a time.

### Key Characteristics

- **Uniform interface** – algorithms can operate on any container via iterators without knowing the container’s type.
- **Category-based capabilities** – iterators are classified by the operations they support (input, output, forward, bidirectional, random access, contiguous).
- **Traits-based introspection** – `std::iterator_traits` provides a uniform way to query an iterator’s associated types.
- **Adaptable** – iterator adaptors (reverse, insert, move) change an iterator’s behaviour without changing the underlying sequence.
- **Value semantics** – iterators are copyable objects; dereferencing yields a reference to an element.
- **Range-based** – most algorithms operate on half-open ranges `[first, last)`.

### Prerequisites

- Basic C++ syntax (variables, types, control flow).
- Familiarity with pointers and pointer arithmetic.
- Understanding of templates and class templates.
- Knowledge of STL containers and algorithms.
- Awareness of Big-O notation.

### Related Programming Areas

- **STL algorithms** – `<algorithm>` and `<numeric>` rely entirely on iterators.
- **Containers** – every standard container exposes iterators.
- **Ranges (C++20)** – the ranges library builds on iterator concepts.
- **Generic programming** – iterators are the interface between containers and algorithms.
- **Iterator adaptors** – reverse, insert, move iterators modify traversal or assignment behaviour.

### Core Concepts / Features

1. Iterators (the concept and library)
2. Iterator categories
   - Input iterators
   - Output iterators
   - Forward iterators
   - Bidirectional iterators
   - Random access iterators
   - Contiguous iterators (C++20)
3. Iterator operations
4. Iterator adaptors
   - `std::reverse_iterator`
   - `std::back_insert_iterator`
   - `std::front_insert_iterator`
   - `std::insert_iterator`
   - `std::move_iterator`
5. Iterator traits
6. Iterator best practices

---

## 1. Iterators (the Concept)

### Definitions

**Core Definition**
An iterator is an abstraction of a pointer that designates an element in a sequence and provides operations for traversal, access, and (for some categories) modification.

**Technical Definition**
The iterator library defines six iterator categories (five until C++17). Each category is defined by the operations that can be performed on it. All categories except output iterators can be organised into a hierarchy, where more powerful categories support the operations of less powerful ones. A mutable iterator satisfies both input and output requirements; a constant iterator does not support writing. Iterators are called `constexpr` iterators if all operations meeting category requirements are `constexpr` functions (since C++20).

**Beginner-Friendly Explanation**
An iterator is a generalised pointer that knows how to walk through a container. Different iterators have different “powers”: some can only read, some can write, some can go backward, and some can jump directly to any position.

### Purposes

- To decouple algorithms from the specific data structures they operate on.
- To provide a uniform interface for traversing containers and ranges.
- To enable generic programming through templates.
- To support a hierarchy of capabilities matched to container properties.
- To allow raw pointers to be used wherever iterators are expected.

### Syntax Rules and Structure

**General Syntax (using iterators)**
```cpp
auto it = container.begin();            // iterator to first element
auto end = container.end();             // past-the-end iterator
for (; it != end; ++it) { /* ... */ }
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `begin()` | Returns an iterator to the first element. |
| `end()` | Returns a past-the-end iterator. |
| `*it` | Dereferences the iterator to access the element. |
| `++it` | Advances the iterator to the next element. |
| `it != end` | Compares iterators to determine range termination. |

**Syntax Rules**
- A range is a pair of iterators `[first, last)`: the first element is included, the last is excluded.
- Past-the-end iterators are not dereferenceable; they only serve as sentinels.
- Singular iterators (not associated with any sequence) have undefined behaviour for most operations.
- Raw pointers satisfy random-access iterator requirements and can be used with STL algorithms.

**Constraints and Limitations**
- An iterator is invalidated when the underlying container is modified (rules vary by container).
- Comparing iterators from different containers is undefined behaviour.
- A range `[i, j)` is valid only if `j` is reachable from `i` via repeated `++i`.

### Annotated Code Examples

**Example 1: Iterating with a `vector`**

```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {10, 20, 30};
    for (auto it = v.begin(); it != v.end(); ++it)
        std::cout << *it << ' ';
}
```

**Expected Output**
```
10 20 30
```

**Why this output?**  
`begin()` points to the first element; `++it` advances through the vector; `end()` signals termination.

**Example 2: Iterating with a raw pointer (C array)**

```cpp
#include <iostream>
#include <algorithm>

int main() {
    int arr[] = {3, 1, 4, 1, 5};
    std::sort(arr, arr + 5);               // pointers as random-access iterators
    for (int x : arr) std::cout << x << ' ';
}
```

**Expected Output**
```
1 1 3 4 5
```

**Why this output?**  
Pointers satisfy random-access iterator requirements, so `std::sort` works directly on the C array.

### Real-World Cases

- **Algorithms** – `std::find`, `std::sort`, `std::accumulate` operate on iterators.
- **Container traversal** – every standard container exposes iterators.
- **Generic libraries** – APIs accept iterator pairs to support any container.
- **Ranges (C++20)** – iterator concepts underpin the ranges library.

---

## 2. Iterator Categories

### Definitions

**Core Definition**
Iterator categories classify iterators by the operations they support, forming a capability hierarchy from weakest (input/output) to strongest (contiguous).

**Technical Definition**
There are five categories until C++17 and six since C++17: LegacyInputIterator, LegacyOutputIterator, LegacyForwardIterator, LegacyBidirectionalIterator, LegacyRandomAccessIterator, and LegacyContiguousIterator (since C++17). All categories except output iterators form a hierarchy: random access implies bidirectional, which implies forward, which implies input. C++20 introduces a parallel system of iterator concepts (`input_iterator`, `forward_iterator`, etc.) with somewhat different requirements.

**Beginner-Friendly Explanation**
Iterator categories are like skill levels. An input iterator can only read once; a forward iterator can read multiple times and go forward; a bidirectional iterator can also go backward; a random-access iterator can jump anywhere instantly; and a contiguous iterator guarantees the elements are stored next to each other in memory.

### Purposes

- To allow algorithms to state the minimum iterator capability they require.
- To enable optimal implementations for more powerful iterators.
- To provide a vocabulary for describing iterator capabilities.
- To support tag dispatch and `if constexpr` specialisation.

### Syntax Rules and Structure

**Category Hierarchy**
```
InputIterator ──► ForwardIterator ──► BidirectionalIterator ──► RandomAccessIterator ──► ContiguousIterator
OutputIterator (separate, no hierarchy)
```

**Component Breakdown**

| Category | Read | Write | Increment | Decrement | Random Access | Contiguous |
|----------|------|-------|-----------|-----------|---------------|------------|
| Input | ✓ | — | ✓ | — | — | — |
| Output | — | ✓ | ✓ | — | — | — |
| Forward | ✓ | ✓ | ✓ | — | — | — |
| Bidirectional | ✓ | ✓ | ✓ | ✓ | — | — |
| Random Access | ✓ | ✓ | ✓ | ✓ | ✓ | — |
| Contiguous (C++17) | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |

*Source: cppreference iterator category table.*

**Syntax Rules**
- Categories are defined by operations, not by specific types; any type supporting the required operations qualifies.
- Raw pointers satisfy random-access (and contiguous) requirements.
- Output iterators are not part of the hierarchy; they support write-only traversal.

**Constraints and Limitations**
- A type supporting the operations of a category does not automatically *fall into* that category; the category page lists complete requirements.
- C++20 concepts may impose different requirements than C++17 named requirements.

---

### 2.1 Input Iterators

#### Definitions

**Core Definition**
An input iterator supports reading elements from a sequence in a single pass.

**Technical Definition**
`LegacyInputIterator` requires `*i` (read), `++i` (increment), equality comparison, and copy construction. It supports single-pass algorithms; after incrementing, previous copies may be invalidated.

**Beginner-Friendly Explanation**
An input iterator is like a one-way read-only stream: you can read the current item and move forward, but you cannot go back or read the same item twice.

#### Purposes

- To read data from streams (files, network, console).
- To support single-pass algorithms.
- To serve as the base of the forward-iterator hierarchy.

#### Syntax Rules and Structure

**General Syntax**
```cpp
InputIterator it = source.begin();
while (it != source.end()) {
    T value = *it;      // read
    ++it;               // advance
}
```

**Component Breakdown**

| Operation | Expression |
|-----------|------------|
| Read | `*it` |
| Increment | `++it` |
| Equality | `it1 == it2` |

**Constraints and Limitations**
- Single-pass only; multi-pass algorithms cannot be used.
- After increment, copies of the old iterator may be invalid.

#### Annotated Code Example

```cpp
#include <iostream>
#include <sstream>
#include <iterator>

int main() {
    std::istringstream input("10 20 30");
    std::istream_iterator<int> it(input), end;

    while (it != end) {
        std::cout << *it << ' ';
        ++it;
    }
}
```

**Expected Output**
```
10 20 30
```

**Why this output?**  
`istream_iterator` is an input iterator that reads integers from the stream one at a time.

#### Real-World Cases

- **Stream processing** – reading tokens from `std::cin` or files.
- **Single-pass parsers** – processing data sequentially.

---

### 2.2 Output Iterators

#### Definitions

**Core Definition**
An output iterator supports writing elements to a sequence in a single pass.

**Technical Definition**
`LegacyOutputIterator` requires `*i = value` (write), `++i` (increment), and copy construction. It is not part of the forward hierarchy and cannot be compared for equality in general.

**Beginner-Friendly Explanation**
An output iterator is like a one-way write-only pipe: you can send items into it and move forward, but you cannot read back what you wrote.

#### Purposes

- To write data to streams, files, or containers.
- To support algorithms that produce output (`std::copy`, `std::transform`).
- To serve as the destination in output algorithms.

#### Syntax Rules and Structure

**General Syntax**
```cpp
OutputIterator out = destination.begin();
*out = value;
++out;
```

**Component Breakdown**

| Operation | Expression |
|-----------|------------|
| Write | `*out = value` |
| Increment | `++out` |

**Constraints and Limitations**
- Write-only; cannot read through an output iterator.
- Single-pass; after increment, previous copies may be invalid.

#### Annotated Code Example

```cpp
#include <iostream>
#include <iterator>
#include <algorithm>
#include <vector>

int main() {
    std::vector<int> v = {1, 2, 3};
    std::ostream_iterator<int> out(std::cout, " ");
    std::copy(v.begin(), v.end(), out);
}
```

**Expected Output**
```
1 2 3
```

**Why this output?**  
`ostream_iterator` is an output iterator that writes each element to `std::cout` followed by a space.

#### Real-World Cases

- **Logging** – writing formatted output to streams.
- **Data export** – copying containers to files.
- **Transformation pipelines** – producing results into output containers.

---

### 2.3 Forward Iterators

#### Definitions

**Core Definition**
A forward iterator supports reading and writing, and can be traversed forward multiple times (multi-pass).

**Technical Definition**
`LegacyForwardIterator` satisfies `LegacyInputIterator` and additionally guarantees that copies of the iterator remain valid and that the same sequence can be traversed multiple times.

**Beginner-Friendly Explanation**
A forward iterator is like a read/write cursor that can move forward through a list, and you can save a copy of your position and come back to it later.

#### Purposes

- To support multi-pass algorithms (e.g., searching and then processing).
- To traverse singly-linked structures (`std::forward_list`).
- To serve as the base for bidirectional and random-access iterators.

#### Syntax Rules and Structure

**General Syntax**
```cpp
ForwardIterator it = container.begin();
auto saved = it;                        // copies remain valid
++it;
```

**Component Breakdown**

| Operation | Expression |
|-----------|------------|
| Read/write | `*it`, `*it = v` |
| Increment | `++it`, `it++` |
| Multi-pass | Copies remain valid |
| Equality | `it1 == it2` |

**Constraints and Limitations**
- No decrement (`--it` is not available).
- No random access.

#### Annotated Code Example

```cpp
#include <forward_list>
#include <iostream>

int main() {
    std::forward_list<int> fl = {1, 2, 3};
    auto it = fl.begin();
    auto saved = it;                       // copy is valid
    ++it;
    std::cout << *saved << ' ' << *it << '\n';
}
```

**Expected Output**
```
1 2
```

**Why this output?**  
Forward iterators support copying; `saved` still points to the first element after `it` is advanced.

#### Real-World Cases

- **Singly-linked lists** – `std::forward_list` provides forward iterators.
- **Hash table bucket traversal** – implementations often use forward iterators.
- **Multi-pass parsing** – scanning data more than once.

---

### 2.4 Bidirectional Iterators

#### Definitions

**Core Definition**
A bidirectional iterator supports all forward iterator operations plus backward traversal via `--`.

**Technical Definition**
`LegacyBidirectionalIterator` satisfies `LegacyForwardIterator` and additionally supports `--it` and `it--`. Standard containers `std::list`, `std::set`, `std::map`, and their multi/unordered counterparts provide bidirectional iterators.

**Beginner-Friendly Explanation**
A bidirectional iterator can walk both forward and backward through a sequence, like a cursor that can move left or right.

#### Purposes

- To traverse doubly-linked structures in both directions.
- To support algorithms that need to reverse direction.
- To serve as the base for random-access iterators.

#### Syntax Rules and Structure

**General Syntax**
```cpp
BidirectionalIterator it = container.end();
--it;                                   // move to last element
```

**Component Breakdown**

| Operation | Expression |
|-----------|------------|
| Forward | `++it` |
| Backward | `--it` |
| Read/write | `*it` |

**Constraints and Limitations**
- No random access (`it + n` is not available).

#### Annotated Code Example

```cpp
#include <list>
#include <iostream>

int main() {
    std::list<int> lst = {1, 2, 3};
    auto it = lst.end();
    --it;                                  // now points to 3
    std::cout << *it << '\n';
    --it;                                  // now points to 2
    std::cout << *it << '\n';
}
```

**Expected Output**
```
3
2
```

**Why this output?**  
Bidirectional iterators support `--`; starting from `end()` and decrementing reaches the last element.

#### Real-World Cases

- **Doubly-linked lists** – `std::list` iterators are bidirectional.
- **Ordered containers** – `std::set`/`std::map` provide bidirectional iterators.
- **Reverse traversal** – algorithms that scan a sequence backwards.

---

### 2.5 Random Access Iterators

#### Definitions

**Core Definition**
A random access iterator supports all bidirectional operations plus constant-time advancement by arbitrary offsets, subscripting, and relational comparison.

**Technical Definition**
`LegacyRandomAccessIterator` satisfies `LegacyBidirectionalIterator` and additionally supports `it + n`, `it - n`, `it[n]`, `it1 - it2`, and relational operators (`<`, `<=`, `>`, `>=`). Standard containers `std::vector`, `std::deque`, and `std::array` provide random-access iterators.

**Beginner-Friendly Explanation**
A random access iterator can jump directly to any position instantly, like an array index. You can say “give me the element 5 positions ahead” without walking through the intermediate elements.

#### Purposes

- To support efficient algorithms that require random access (`std::sort`, binary search).
- To enable O(1) advancement by arbitrary distances.
- To provide array-like indexing on containers.

#### Syntax Rules and Structure

**General Syntax**
```cpp
RandomAccessIterator it = container.begin();
it += 5;                                // O(1) jump
T x = it[3];                            // subscript
auto dist = it2 - it1;                  // distance
```

**Component Breakdown**

| Operation | Expression | Complexity |
|-----------|------------|------------|
| Offset addition | `it + n`, `it - n` | O(1) |
| Subscript | `it[n]` | O(1) |
| Distance | `it2 - it1` | O(1) |
| Relational comparison | `it1 < it2` | O(1) |

**Constraints and Limitations**
- Only available on containers with contiguous or near-contiguous access (vector, deque, array).
- `std::list` and associative containers do **not** provide random-access iterators.

#### Annotated Code Example

```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {10, 20, 30, 40, 50};
    auto it = v.begin();
    it += 3;                               // jump to element 40
    std::cout << *it << '\n';              // 40
    std::cout << it[1] << '\n';            // 50
}
```

**Expected Output**
```
40
50
```

**Why this output?**  
Random-access iterators support `+=` and `[]`; `it += 3` jumps directly to the fourth element.

#### Real-World Cases

- **Sorting** – `std::sort` requires random-access iterators.
- **Binary search** – `std::lower_bound` on a vector.
- **Numerical algorithms** – accessing elements by computed index.

---

### 2.6 Contiguous Iterators (C++20)

#### Definitions

**Core Definition**
A contiguous iterator is a random-access iterator whose elements are stored contiguously in memory, so the address of an element can be obtained by pointer arithmetic.

**Technical Definition**
`LegacyContiguousIterator` (formally specified in C++17) satisfies `LegacyRandomAccessIterator` and additionally guarantees that `*(it + n)` is equivalent to `*(std::addressof(*it) + n)`. The iterators of `std::vector`, `std::basic_string`, `std::array`, and `std::valarray`, as well as pointers into C arrays, satisfy this requirement. C++20 introduces the `contiguous_iterator` concept.

**Beginner-Friendly Explanation**
A contiguous iterator is the strongest kind: not only can it jump anywhere instantly, but the elements are packed side-by-side in memory, so a pointer to one element can be used to access the others by simple pointer arithmetic.

#### Purposes

- To guarantee that a range can be treated as a C array.
- To enable safe and efficient interoperability with C APIs.
- To support algorithms that rely on contiguous storage.

#### Syntax Rules and Structure

**General Syntax**
```cpp
ContiguousIterator it = vec.begin();
T* ptr = std::to_address(it);           // pointer to first element
T* ptr_n = ptr + n;                     // pointer arithmetic
```

**Component Breakdown**

| Property | Guarantee |
|----------|-----------|
| Random access | Yes |
| Contiguous storage | `&*(it + n) == &*it + n` |
| Pointer conversion | `std::to_address(it)` yields a pointer |

**Constraints and Limitations**
- Only available on contiguous containers (`vector`, `array`, `string`, C arrays).
- `std::deque` is random-access but **not** contiguous.

#### Annotated Code Example

```cpp
#include <vector>
#include <iostream>
#include <memory>

int main() {
    std::vector<int> v = {10, 20, 30};
    auto it = v.begin();
    int* ptr = std::to_address(it);
    std::cout << ptr[1] << '\n';           // 20
}
```

**Expected Output**
```
20
```

**Why this output?**  
The vector’s storage is contiguous; `std::to_address` obtains a raw pointer, and pointer arithmetic accesses the second element.

#### Real-World Cases

- **C API integration** – passing vector data to functions expecting `T*`.
- **Memory-mapped I/O** – treating a contiguous buffer as an array.
- **SIMD and low-level optimisation** – exploiting contiguous layout.

---

## 3. Iterator Operations

### Definitions

**Core Definition**
Iterator operations are the standard library functions and operators that advance, measure, and manipulate iterators.

**Technical Definition**
The `<iterator>` header provides `std::advance` (advance an iterator by a distance), `std::distance` (number of increments between two iterators), `std::next` (return an iterator advanced by n), and `std::prev` (return an iterator retreated by n). These functions adapt to the iterator category, providing the most efficient implementation possible.

**Beginner-Friendly Explanation**
Iterator operations are helper functions that let you move an iterator forward or backward by a certain amount, or measure how far apart two iterators are, without writing manual loops.

### Purposes

- To advance iterators by a specified distance.
- To measure the distance between two iterators.
- To obtain a new iterator position without modifying the original.
- To write category-generic code that adapts to iterator capabilities.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::advance(it, n);                    // advances it by n
auto d = std::distance(first, last);    // number of steps from first to last
auto it2 = std::next(it, n);            // new iterator n steps ahead
auto it3 = std::prev(it, n);            // new iterator n steps back
```

**Component Breakdown**

| Function | Description | Complexity |
|----------|-------------|------------|
| `advance(it, n)` | Advances `it` by `n` (positive or negative) | O(1) for random access; O(n) otherwise |
| `distance(first, last)` | Returns number of increments from `first` to `last` | O(1) for random access; O(n) otherwise |
| `next(it, n)` | Returns iterator advanced by `n` | Same as `advance` |
| `prev(it, n)` | Returns iterator retreated by `n` | Same as `advance` (requires bidirectional) |

**Syntax Rules**
- `advance` modifies its argument in place; `next` and `prev` return a new iterator.
- `distance` requires `last` to be reachable from `first`.
- `prev` requires at least bidirectional iterators.

**Constraints and Limitations**
- `advance` with negative `n` requires bidirectional iterators.
- Complexity depends on iterator category; random-access iterators are O(1), others are O(n).

### Annotated Code Examples

**Example 1: `advance` and `distance`**

```cpp
#include <list>
#include <iostream>
#include <iterator>

int main() {
    std::list<int> lst = {10, 20, 30, 40};
    auto it = lst.begin();
    std::advance(it, 2);                   // O(n) for list
    std::cout << *it << '\n';              // 30

    auto d = std::distance(lst.begin(), it);
    std::cout << d << '\n';                // 2
}
```

**Expected Output**
```
30
2
```

**Why this output?**  
`advance` moves the iterator two steps forward; `distance` measures the number of increments between `begin()` and `it`.

**Example 2: `next` and `prev`**

```cpp
#include <vector>
#include <iostream>
#include <iterator>

int main() {
    std::vector<int> v = {10, 20, 30, 40, 50};
    auto it = v.begin();

    auto n = std::next(it, 3);             // O(1) for vector
    std::cout << *n << '\n';              // 40

    auto p = std::prev(n, 1);
    std::cout << *p << '\n';              // 30
}
```

**Expected Output**
```
40
30
```

**Why this output?**  
`next` returns an iterator advanced by 3; `prev` returns an iterator retreated by 1.

### Real-World Cases

- **Generic algorithms** – `std::advance` and `std::distance` adapt to iterator category.
- **Position calculation** – finding the index of an element in a list.
- **Windowed processing** – using `next` to look ahead without disturbing the current iterator.

---

## 4. Iterator Adaptors

### Definitions

**Core Definition**
Iterator adaptors are class templates that wrap an existing iterator and modify its behaviour—changing traversal direction, assignment semantics, or both—without altering the underlying sequence.

**Technical Definition**
The standard library provides five iterator adaptors: `std::reverse_iterator` (reverses traversal direction), `std::back_insert_iterator` (appends to a container), `std::front_insert_iterator` (prepends to a container), `std::insert_iterator` (inserts at a specified position), and `std::move_iterator` (converts dereference to an rvalue reference). Convenience function templates `std::make_reverse_iterator`, `std::back_inserter`, `std::front_inserter`, `std::inserter`, and `std::make_move_iterator` deduce the adaptor type from their arguments.

**Beginner-Friendly Explanation**
An iterator adaptor is like a lens that changes how an iterator behaves. A reverse iterator makes a forward iterator go backward; an insert iterator turns assignments into insertions; a move iterator turns copies into moves.

### Purposes

- To traverse a container in reverse without changing the container.
- To turn assignment into insertion for use with `std::copy` and similar algorithms.
- To enable move semantics in algorithms that normally copy.
- To adapt iterators to different algorithm requirements.

---

### 4.1 `std::reverse_iterator`

#### Definitions

**Core Definition**
`std::reverse_iterator` is an iterator adaptor that reverses the direction of a given bidirectional iterator.

**Technical Definition**
`std::reverse_iterator<Iter>` requires `Iter` to be at least a `LegacyBidirectionalIterator` (or model `bidirectional_iterator` since C++20). It is the iterator returned by `rbegin()` and `rend()` of standard containers. The fundamental identity is `&*(reverse_iterator(i)) == &*(i - 1)`.

**Beginner-Friendly Explanation**
A reverse iterator lets you walk a container from the last element to the first, as if the container were reversed.

#### Purposes

- To traverse a sequence in reverse order.
- To use reverse algorithms without copying or reversing the container.
- To support `rbegin()`/`rend()` idioms.

#### Syntax Rules and Structure

**General Syntax**
```cpp
std::reverse_iterator<Iterator> rit(base_iterator);
auto rit = std::make_reverse_iterator(base_iterator);
```

**Constraints and Limitations**
- Requires at least bidirectional iterators.
- The base iterator is one position ahead of the reverse iterator’s target.

#### Annotated Code Example

```cpp
#include <vector>
#include <iostream>
#include <iterator>

int main() {
    std::vector<int> v = {1, 2, 3, 4};
    for (auto rit = v.rbegin(); rit != v.rend(); ++rit)
        std::cout << *rit << ' ';
}
```

**Expected Output**
```
4 3 2 1
```

**Why this output?**  
`rbegin()` returns a reverse iterator pointing to the last element; `++rit` moves toward the first element.

---

### 4.2 `std::back_insert_iterator`

#### Definitions

**Core Definition**
`std::back_insert_iterator` is an output iterator that appends elements to a container by calling its `push_back()` member function.

**Technical Definition**
`std::back_insert_iterator<Container>` is a `LegacyOutputIterator` that appends elements to the container for which it was constructed. The container’s `push_back()` member function is called whenever the iterator (whether dereferenced or not) is assigned to. Incrementing the iterator is a no-op.

**Beginner-Friendly Explanation**
A back-insert iterator turns assignments into “append to the end” operations. It is commonly used with `std::copy` to append elements to a vector.

#### Purposes

- To append elements to a container via assignment.
- To use output algorithms with containers that support `push_back()`.
- To avoid pre-sizing the destination container.

#### Syntax Rules and Structure

**General Syntax**
```cpp
std::back_insert_iterator<Container> it(container);
auto it = std::back_inserter(container);
```

**Constraints and Limitations**
- Requires `push_back()` on the container.
- Increment is a no-op.

#### Annotated Code Example

```cpp
#include <vector>
#include <iostream>
#include <algorithm>
#include <iterator>

int main() {
    std::vector<int> src = {1, 2, 3};
    std::vector<int> dst;
    std::copy(src.begin(), src.end(), std::back_inserter(dst));
    for (int x : dst) std::cout << x << ' ';
}
```

**Expected Output**
```
1 2 3
```

**Why this output?**  
`back_inserter` calls `dst.push_back()` for each element copied from `src`.

---

### 4.3 `std::front_insert_iterator`

#### Definitions

**Core Definition**
`std::front_insert_iterator` is an output iterator that prepends elements to a container by calling its `push_front()` member function.

**Technical Definition**
`std::front_insert_iterator<Container>` is a `LegacyOutputIterator` that inserts elements at the front of the container. The container’s `push_front()` member function is called on assignment.

**Beginner-Friendly Explanation**
A front-insert iterator turns assignments into “prepend to the front” operations.

#### Purposes

- To prepend elements to a container via assignment.
- To support algorithms that produce output in reverse order.

#### Syntax Rules and Structure

**General Syntax**
```cpp
std::front_insert_iterator<Container> it(container);
auto it = std::front_inserter(container);
```

**Constraints and Limitations**
- Requires `push_front()` on the container (`std::deque`, `std::list`, `std::forward_list`).
- Not available for `std::vector`.

#### Annotated Code Example

```cpp
#include <list>
#include <iostream>
#include <algorithm>
#include <iterator>

int main() {
    std::list<int> src = {1, 2, 3};
    std::list<int> dst;
    std::copy(src.begin(), src.end(), std::front_inserter(dst));
    for (int x : dst) std::cout << x << ' ';
}
```

**Expected Output**
```
3 2 1
```

**Why this output?**  
Each element is prepended, so the order is reversed.

---

### 4.4 `std::insert_iterator`

#### Definitions

**Core Definition**
`std::insert_iterator` is an output iterator that inserts elements at a specified position in a container.

**Technical Definition**
`std::insert_iterator<Container>` inserts elements before a fixed position using the container’s `insert()` member function. It is created with `std::inserter(container, position)`.

**Beginner-Friendly Explanation**
An insert iterator inserts elements at a position you choose, rather than at the front or back.

#### Purposes

- To insert elements at a specific position via assignment.
- To maintain sorted order when producing output.
- To support algorithms that write to a middle position.

#### Syntax Rules and Structure

**General Syntax**
```cpp
auto it = std::inserter(container, position);
```

**Constraints and Limitations**
- Requires `insert()` on the container.
- The insertion position is fixed at construction.

#### Annotated Code Example

```cpp
#include <vector>
#include <iostream>
#include <algorithm>
#include <iterator>

int main() {
    std::vector<int> v = {10, 20, 30};
    auto it = std::inserter(v, v.begin() + 1);
    *it = 15;                              // insert 15 at position 1
    for (int x : v) std::cout << x << ' ';
}
```

**Expected Output**
```
10 15 20 30
```

**Why this output?**  
`inserter` inserts `15` before the element at position 1, shifting subsequent elements.

---

### 4.5 `std::move_iterator`

#### Definitions

**Core Definition**
`std::move_iterator` is an iterator adaptor that behaves like the underlying iterator, except that dereferencing produces an rvalue reference, enabling move semantics.

**Technical Definition**
`std::move_iterator<Iter>` requires `Iter` to be at least a `LegacyInputIterator`. Its dereference operator converts the underlying iterator’s value into an rvalue reference, so algorithms that normally copy will move from the source instead. When used as an input iterator, the effect is that values are moved from, rather than copied from.

**Beginner-Friendly Explanation**
A move iterator turns copies into moves. Use it when you want to transfer elements out of a source range without deep-copying them.

#### Purposes

- To enable move semantics in algorithms that normally copy.
- To transfer elements efficiently from a source range.
- To avoid expensive copies of large objects.

#### Syntax Rules and Structure

**General Syntax**
```cpp
std::move_iterator<Iterator> it(base_iterator);
auto it = std::make_move_iterator(base_iterator);
```

**Constraints and Limitations**
- The source range is left in a moved-from state (valid but unspecified).
- Requires at least input iterators.

#### Annotated Code Example

```cpp
#include <vector>
#include <iostream>
#include <algorithm>
#include <iterator>
#include <string>

int main() {
    std::vector<std::string> src = {"hello", "world"};
    std::vector<std::string> dst;

    std::copy(std::make_move_iterator(src.begin()),
              std::make_move_iterator(src.end()),
              std::back_inserter(dst));

    std::cout << dst[0] << ' ' << dst[1] << '\n';
    std::cout << "src[0]: '" << src[0] << "'\n";
}
```

**Expected Output**
```
hello world
src[0]: ''
```

**Why this output?**  
`move_iterator` converts dereference to rvalue, so `std::copy` moves the strings into `dst`; the source strings become empty.

#### Real-World Cases

- **Transferring ownership** – moving elements between containers.
- **Performance optimisation** – avoiding deep copies of expensive types.
- **Resource management** – moving `unique_ptr` or file handles.

---

## 5. Iterator Traits

### Definitions

**Core Definition**
`std::iterator_traits` is a type trait class that provides a uniform interface to the properties of iterator types, enabling algorithms to query an iterator’s associated types.

**Technical Definition**
`std::iterator_traits<Iter>` provides the member types `difference_type`, `value_type`, `pointer`, `reference`, and `iterator_category` (or `iterator_concept` since C++20). It is SFINAE-friendly since C++17 and provides partial specialisations for pointer types `T*` and `const T*`, making it possible to use all iterator-based algorithms with raw pointers.

**Beginner-Friendly Explanation**
Iterator traits are a “specification sheet” for an iterator. They tell algorithms what type the iterator points to, what kind of reference it returns, how far it can jump, and which category it belongs to—so the algorithm can choose the right implementation.

### Purposes

- To query an iterator’s value type, difference type, pointer, and reference types.
- To determine an iterator’s category for tag dispatch.
- To enable algorithms to work uniformly with raw pointers and user-defined iterators.
- To support customisation for user-defined iterators.

### Syntax Rules and Structure

**General Syntax**
```cpp
typename std::iterator_traits<Iter>::value_type
typename std::iterator_traits<Iter>::difference_type
typename std::iterator_traits<Iter>::iterator_category
```

**Component Breakdown**

| Member Type | Meaning |
|-------------|---------|
| `difference_type` | Signed integer type for distances between iterators |
| `value_type` | Type of the elements pointed to |
| `pointer` | Pointer type (`T*` or proxy) |
| `reference` | Reference type (`T&` or proxy) |
| `iterator_category` | Category tag (input, forward, etc.) |

**Syntax Rules**
- Always use `std::iterator_traits` when accessing type information about an iterator, even if `decltype` or `auto` seem to work—custom specializations may change the result.
- Raw pointers `T*` are treated as random-access iterators via partial specialisation.

**Constraints and Limitations**
- If an iterator does not provide all five member types, `iterator_traits` may have no members by those names (SFINAE-friendly since C++17).
- C++20 introduces `iterator_concept` for concept-based conformance.

### Annotated Code Example

```cpp
#include <vector>
#include <iostream>
#include <iterator>

template <typename Iter>
void describe(Iter it) {
    using traits = std::iterator_traits<Iter>;
    std::cout << "value_type: " << typeid(typename traits::value_type).name() << '\n';
    std::cout << "difference_type: " << typeid(typename traits::difference_type).name() << '\n';
    std::cout << "category: " << typeid(typename traits::iterator_category).name() << '\n';
}

int main() {
    std::vector<int> v = {1, 2, 3};
    describe(v.begin());
}
```

**Expected Output** (type names are implementation-dependent)
```
value_type: i
difference_type: l
category: NSt3__111__wrap_iterIPiEE
```

**Why this output?**  
`iterator_traits` extracts the associated types from the vector’s iterator type.

### Real-World Cases

- **Generic algorithms** – `std::distance` uses `difference_type`.
- **Tag dispatch** – selecting an algorithm implementation based on `iterator_category`.
- **Custom iterators** – specialising `iterator_traits` for user-defined iterator types.

---

## 6. Iterator Best Practices

### Definitions

**Core Definition**
Iterator best practices are guidelines for using iterators safely, efficiently, and idiomatically in C++.

**Technical Definition**
These practices are grounded in the C++ Core Guidelines and community consensus: prefer standard algorithms over hand-written loops, use `const_iterator` when not modifying, compare iterators with `!=` rather than `<`, ensure ranges are valid, and avoid comparing iterators from different containers.

**Beginner-Friendly Explanation**
Best practices for iterators are simple rules: use algorithms instead of writing loops, use `const` iterators when you don’t need to modify, and never mix iterators from different containers. These habits prevent bugs and make code clearer.

### Key Guidelines

| Guideline | Summary |
|-----------|---------|
| **Prefer standard algorithms** | Use `std::find`, `std::sort`, etc., instead of hand-written loops. |
| **Use `const_iterator` when not modifying** | Prevents accidental modification and documents intent. |
| **Compare with `!=`, not `<`** | `!=` works for all iterator categories; `<` requires random access. |
| **Never mix iterators from different containers** | Undefined behaviour. |
| **Ensure ranges are valid** | `last` must be reachable from `first`. |
| **Use `std::distance` / `std::advance` for category-generic code** | They adapt to the iterator category. |
| **Invalidate iterators carefully** | Know the invalidation rules of the container you are using. |
| **Prefer `cbegin()` / `cend()` for read-only traversal** | Returns `const_iterator`. |
| **Use `auto` for iterator types** | Avoids verbose type names and improves readability. |
| **Consider `std::span` (C++20) for contiguous ranges** | Type-safe view over contiguous memory. |

### Annotated Code Example

```cpp
#include <vector>
#include <iostream>
#include <algorithm>

int main() {
    std::vector<int> v = {3, 1, 4, 1, 5};

    // Prefer algorithm over manual loop
    auto it = std::find(v.cbegin(), v.cend(), 4);
    if (it != v.cend())
        std::cout << "Found 4 at index "
                  << std::distance(v.cbegin(), it) << '\n';

    // Use const_iterator for read-only traversal
    for (auto cit = v.cbegin(); cit != v.cend(); ++cit)
        std::cout << *cit << ' ';
}
```

**Expected Output**
```
Found 4 at index 2
3 1 4 1 5
```

**Why this output?**  
`std::find` locates the element; `std::distance` computes the index; `cbegin()`/`cend()` provide read-only iterators.

### Real-World Cases

- **Code reviews** – checking for mixed-container iterator comparisons.
- **Static analysis** – tools like clang-tidy flag unsafe iterator use.
- **Performance tuning** – choosing algorithms that exploit random-access iterators.

---

## References

- Iterator library — cppreference – https://en.cppreference.com/w/cpp/iterator
- Iterator categories — cppreference – https://en.cppreference.com/w/cpp/iterator#Iterator_categories
- `std::reverse_iterator` — cppreference – https://en.cppreference.com/w/cpp/iterator/reverse_iterator
- `std::back_insert_iterator` — cppreference – https://en.cppreference.com/w/cpp/iterator/back_insert_iterator
- `std::front_insert_iterator` — cppreference – https://en.cppreference.com/w/cpp/iterator/front_insert_iterator
- `std::insert_iterator` — cppreference – https://en.cppreference.com/w/cpp/iterator/insert_iterator
- `std::move_iterator` — cppreference – https://en.cppreference.com/w/cpp/iterator/move_iterator
- `std::iterator_traits` — cppreference – https://en.cppreference.com/w/cpp/iterator/iterator_traits
- Iterator operations (`std::advance`, `std::distance`, `std::next`, `std::prev`) — cppreference – https://en.cppreference.com/w/cpp/iterator#Iterator_operations
- C++ Core Guidelines — SL.con: Containers – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-stdlib
- Iterator library — cppreference (printable overview) – https://en.cppreference.com/w/cpp/iterator

---

This cheat sheet provides a comprehensive, structured reference for C++ iterators, covering the iterator concept, all six categories, operations, adaptors, traits, and best practices, with annotated code examples, expected outputs, real-world cases, and authoritative references.