# STL Fundamentals — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
The Standard Template Library (STL) is a C++ library providing a collection of **generic containers, iterators, algorithms, function objects, adaptors, and allocators** that work together through a consistent, template-based interface.

**Technical Definition**
The STL is a generic library of class templates and function templates that provides common data structures and algorithms. It is built on the principle of **generic programming**: algorithms are decoupled from the data structures they operate on through the **iterator** abstraction. The STL is not object-oriented in the traditional sense; it relies on **static binding** and **templates** rather than virtual functions and dynamic polymorphism. Its six core component categories are **containers**, **iterators**, **algorithms**, **function objects**, **adaptors**, and **allocators**. The library was designed by Alexander Stepanov in the early 1990s and was incorporated into the C++ standard in 1998.

**Beginner-Friendly Explanation**
The STL is a **toolbox of ready-made data structures and algorithms** that you can plug together. Instead of writing your own dynamic array, linked list, sorting function, or search routine, you use `std::vector`, `std::list`, `std::sort`, and `std::find`. The magic is that the algorithms **don't know what kind of container they're working on**—they just use **iterators** (which behave like pointers) to walk through the data. This means the same `std::sort` works on arrays, vectors, and deques. The STL has six parts: containers (hold data), iterators (navigate data), algorithms (process data), function objects (customize operations), adaptors (change interfaces), and allocators (manage memory).

### Key Characteristics

- **Generic programming**: Algorithms and containers are parameterized by type.
- **Decoupling**: Iterators separate algorithms from containers.
- **Efficiency**: The STL provides well-optimized implementations of common data structures and algorithms.
- **Templates everywhere**: All components are templates.
- **Static binding**: No virtual dispatch; operations are resolved at compile time.
- **Composability**: Components are designed to plug together seamlessly.
- **No runtime safety guarantees**: The STL is optimized for performance and does not check for errors (e.g., out-of-bounds access).

### Prerequisites

- Understanding of C++ templates and type parameters.
- Familiarity with classes, objects, and operator overloading.
- Knowledge of pointers and references (iterators generalize pointers).
- Basic awareness of dynamic memory and exception safety.

### Related Programming Areas

- **Generic programming** – the paradigm the STL embodies.
- **Containers and data structures** – vectors, lists, maps, sets.
- **Algorithms** – sorting, searching, transforming.
- **Iterators** – the glue between containers and algorithms.
- **Functional programming** – function objects and lambdas.
- **Memory management** – allocators and custom memory models.
- **Modern C++ (C++20 Ranges)** – the ranges library extends the STL's ideas.

### Core Concepts / Features

1. Containers
2. Iterators
3. Algorithms
4. Function Objects
5. Adaptors
6. Allocators
7. STL Components
8. STL Best Practices

---

## 1. Containers

### Definitions

**Core Definition**
A container is a class template that **stores and manages a collection of other objects** (its elements), providing member functions to access, insert, and erase elements.

**Technical Definition**
The Containers library is a generic collection of class templates and algorithms that allow programmers to easily implement common data structures like queues, lists, and stacks. There are three classes of containers: **sequence containers**, **associative containers**, and **unordered associative containers**, each designed to support a different set of operations. The container manages the storage space allocated for its elements and provides member functions to access them, either directly or through iterators.

**Beginner-Friendly Explanation**
A container is a **box that holds things**. You can put items in, take them out, look at them, and ask how many there are. Different containers are optimized for different tasks: `std::vector` is a fast, growable array; `std::list` is a linked list for frequent insertions in the middle; `std::map` is a sorted dictionary; `std::unordered_map` is a hash table.

### Purposes

- To store and organize collections of objects.
- To provide efficient access, insertion, and removal of elements.
- To manage memory for their elements automatically.
- To offer different performance trade-offs for different workloads.
- To serve as the data side of the STL's generic programming model.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <vector>
#include <list>
#include <map>
#include <unordered_map>
#include <set>

std::vector<int> v;          // sequence container
std::list<std::string> l;   // sequence container
std::map<int, double> m;     // associative container
std::unordered_map<int, double> um;  // unordered associative container
std::set<int> s;             // associative container
```

**Container Categories**

| Category | Examples | Characteristics |
|----------|----------|-----------------|
| **Sequence** | `vector`, `deque`, `list`, `array` | Elements in a specific order |
| **Associative** | `set`, `map`, `multiset`, `multimap` | Sorted, O(log n) lookup |
| **Unordered associative** | `unordered_set`, `unordered_map` | Hashed, O(1) average lookup |
| **Adaptors** | `stack`, `queue`, `priority_queue` | Restricted interfaces (see §5) |

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `std::vector<T>` | Resizable contiguous array. |
| `std::deque<T>` | Double-ended queue. |
| `std::list<T>` | Doubly-linked list. |
| `std::map<K, V>` | Sorted key-value pairs. |
| `std::unordered_map<K, V>` | Hashed key-value pairs. |
| `std::set<T>` | Sorted unique keys. |

**Syntax Rules**
- Containers are class templates parameterized by element type.
- They manage their own memory via allocators.
- They provide iterators via `begin()` and `end()`.
- They provide member functions for size, capacity, and modification.

**Constraints and Limitations**
- Containers do not perform bounds checking (use `at()` for checked access).
- Different containers have different iterator invalidation rules.
- Choosing the wrong container can lead to severe performance problems.

### Annotated Code Examples

**Example 1: `std::vector` — the default sequence container**

```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> v = {10, 20, 30};
    v.push_back(40);                      // O(1) amortized
    for (int x : v) std::cout << x << ' '; // 10 20 30 40
    std::cout << '\n';
    std::cout << "size: " << v.size() << '\n';   // 4
    std::cout << "front: " << v.front() << '\n';  // 10
}
```

**Expected Output**
```
10 20 30 40 
size: 4
front: 10
```

**Why this output?**  
`std::vector` is a resizable contiguous array. `push_back` appends `40`, and range-based `for` iterates over all elements.

**Example 2: `std::map` — sorted associative container**

```cpp
#include <iostream>
#include <map>
#include <string>

int main() {
    std::map<std::string, int> ages;
    ages["Alice"] = 30;
    ages["Bob"] = 25;
    for (const auto& [name, age] : ages) {
        std::cout << name << ": " << age << '\n';
    }
}
```

**Expected Output**
```
Alice: 30
Bob: 25
```

**Why this output?**  
`std::map` stores sorted key-value pairs. Iteration order is lexicographic by key (`Alice` before `Bob`).

### Real-World Cases

- **`std::vector`** – default container for most tasks.
- **`std::map`** – configuration stores, symbol tables, ordered lookups.
- **`std::unordered_map`** – caches, frequency counts, fast lookups.
- **`std::list`** – frequent insertions/removals in the middle.
- **`std::set`** – unique element tracking.

---

## 2. Iterators

### Definitions

**Core Definition**
An iterator is a **generalization of a pointer** that allows algorithms to traverse and access elements of a container or range in a uniform manner.

**Technical Definition**
Iterators are a generalization of pointers that allow a C++ program to work with different data structures (for example, containers and ranges since C++20) in a uniform manner. The iterator library provides definitions for iterators, as well as iterator traits, adaptors, and utility functions. Iterators are categorized into six kinds: **LegacyInputIterator**, **LegacyOutputIterator**, **LegacyForwardIterator**, **LegacyBidirectionalIterator**, **LegacyRandomAccessIterator**, and **LegacyContiguousIterator** (since C++17). Each category is defined by the operations it supports, not by a specific type.

**Beginner-Friendly Explanation**
An iterator is a **smart pointer that knows how to walk through a container**. `begin()` gives you an iterator to the first element; `end()` gives you a “past-the-end” sentinel. You can increment (`++it`), dereference (`*it`), and compare iterators. Because all containers provide iterators that follow the same rules, algorithms like `std::sort` can work with any container that provides random-access iterators.

### Purposes

- To provide uniform access to elements of different data structures.
- To decouple algorithms from containers.
- To define a standard interface for traversal, access, and modification.
- To enable generic algorithms to work with any compatible container.
- To support iterator categories with different capabilities.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <iterator>

auto it = container.begin();   // iterator to first element
auto end = container.end();     // past-the-end sentinel
for (; it != end; ++it) {
    *it;                        // access element
}
```

**Iterator Categories**

| Category | Operations | Example Container |
|----------|------------|-------------------|
| Input | `++`, `*`, `==`, `!=` | `istream_iterator` |
| Output | `++`, `*` | `ostream_iterator` |
| Forward | Input + multi-pass | `forward_list` |
| Bidirectional | Forward + `--` | `list`, `set`, `map` |
| Random Access | Bidirectional + `+`, `-`, `<` | `vector`, `deque` |
| Contiguous (C++17) | Random access + contiguous storage | `vector`, `array` |

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `begin()` | Iterator to the first element. |
| `end()` | Past-the-end sentinel. |
| `*it` | Dereference. |
| `++it` | Advance to next element. |
| `it != end` | Comparison. |

**Syntax Rules**
- Iterators are obtained from containers via `begin()`/`end()` or `cbegin()`/`cend()`.
- The dereference operator `*` accesses the element.
- Increment `++` advances to the next element.
- Comparison `==`/`!=` checks for equality.
- Iterator categories define which operations are valid.

**Constraints and Limitations**
- Iterator invalidation rules vary by container (e.g., `vector` iterators are invalidated on reallocation).
- Not all iterators support all operations (e.g., `list` iterators do not support `+`).
- Using an invalidated iterator is undefined behavior.

### Annotated Code Examples

**Example 1: Manual iteration**

```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> v = {1, 2, 3, 4, 5};
    for (auto it = v.begin(); it != v.end(); ++it) {
        std::cout << *it << ' ';
    }
    std::cout << '\n';
}
```

**Expected Output**
```
1 2 3 4 5 
```

**Why this output?**  
`begin()` returns an iterator to the first element; `end()` is the past-the-end sentinel. The loop dereferences and increments the iterator until it reaches `end()`.

**Example 2: Iterator categories in action**

```cpp
#include <iostream>
#include <vector>
#include <list>

template <typename Container>
void print(const Container& c) {
    for (auto it = c.begin(); it != c.end(); ++it) {
        std::cout << *it << ' ';
    }
    std::cout << '\n';
}

int main() {
    std::vector<int> v = {1, 2, 3};
    std::list<int> l = {4, 5, 6};
    print(v);   // vector: random-access iterator
    print(l);   // list: bidirectional iterator
}
```

**Expected Output**
```
1 2 3 
4 5 6 
```

**Why this output?**  
The `print` function works with both `vector` and `list` because both provide at least bidirectional iterators.

### Real-World Cases

- **`std::sort`** – requires random-access iterators.
- **`std::find`** – works with input iterators.
- **`std::list::splice`** – uses bidirectional iterators.
- **Range-based for loops** – implicitly use iterators.

---

## 3. Algorithms

### Definitions

**Core Definition**
An algorithm is a **function template that operates on a range of elements** (defined by iterators) to perform operations such as searching, sorting, counting, transforming, and more.

**Technical Definition**
The algorithms library defines functions for a variety of purposes (e.g., searching, sorting, counting, manipulating) that operate on ranges of elements. A range is defined by a pair of iterators `[first, last)`. Since C++20, constrained algorithms in `std::ranges` can accept a single range argument and support projections. The algorithms are decoupled from containers: the same algorithm works with any container whose iterators satisfy the algorithm's requirements.

**Beginner-Friendly Explanation**
Algorithms are **ready-made operations you can apply to any container**. Instead of writing a loop to find an element, you call `std::find`. Instead of writing a sorting routine, you call `std::sort`. The algorithms use iterators, so they work with vectors, lists, arrays, and even raw C arrays.

### Purposes

- To provide reusable, well-tested implementations of common operations.
- To decouple operations from specific containers.
- To replace hand-written loops with clearer, more correct code.
- To enable the optimizer to apply known transformations.
- To support generic programming across data structures.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <algorithm>

std::algorithm_name(first_iterator, last_iterator, arguments...);
```

**Common Algorithm Categories**

| Category | Examples |
|----------|----------|
| Search | `find`, `find_if`, `count`, `count_if` |
| Sort | `sort`, `stable_sort`, `partial_sort` |
| Transform | `transform`, `copy`, `copy_if`, `replace` |
| Remove | `remove`, `remove_if`, `unique` |
| Numeric | `accumulate`, `reduce`, `inner_product` |
| Binary search | `lower_bound`, `upper_bound`, `binary_search` |
| Set operations | `set_union`, `set_intersection`, `set_difference` |

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `first`, `last` | Iterator pair defining the range `[first, last)`. |
| `predicate` | A callable returning `bool` for filtering. |
| `operation` | A callable for transformation. |
| `comparator` | A callable for ordering. |

**Syntax Rules**
- Algorithms take iterators, not containers (except C++20 range algorithms).
- The range is `[first, last)` — `last` is exclusive.
- Predicates and operations can be function objects, lambdas, or function pointers.
- Many algorithms return iterators to relevant positions.

**Constraints and Limitations**
- Algorithms do not perform bounds checking.
- Iterator invalidation rules apply.
- Some algorithms require specific iterator categories.

### Annotated Code Examples

**Example 1: Searching with `std::find_if`**

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main() {
    std::vector<int> v = {1, 2, 3, 4, 5};
    auto it = std::find_if(v.begin(), v.end(), [](int x) { return x > 3; });
    if (it != v.end()) {
        std::cout << "Found: " << *it << '\n';   // 4
    }
}
```

**Expected Output**
```
Found: 4
```

**Why this output?**  
`std::find_if` returns an iterator to the first element satisfying the predicate (`x > 3`), which is `4`.

**Example 2: Sorting and transforming**

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main() {
    std::vector<int> v = {5, 2, 4, 1, 3};
    std::sort(v.begin(), v.end());
    for (int x : v) std::cout << x << ' ';   // 1 2 3 4 5
    std::cout << '\n';
}
```

**Expected Output**
```
1 2 3 4 5 
```

**Why this output?**  
`std::sort` orders the range in ascending order using `operator<`.

### Real-World Cases

- **Data processing** – filtering, transforming, aggregating.
- **Competitive programming** – sorting, binary search, permutations.
- **Business logic** – finding, counting, removing.
- **Numerical computing** – accumulate, inner_product.

---

## 4. Function Objects

### Definitions

**Core Definition**
A function object (functor) is **any object for which the function call operator (`operator()`) is defined**, allowing it to be called as if it were a function.

**Technical Definition**
A function object is any object for which the function call operator is defined. C++ provides many built-in function objects as well as support for creation and manipulation of new function objects. Function objects include arithmetic operations (`plus`, `minus`, `multiplies`), comparisons (`equal_to`, `less`, `greater`), logical operations (`logical_and`, `logical_or`), and adaptors (`std::bind`, `std::not_fn`). Lambdas are syntactic sugar for function objects.

**Beginner-Friendly Explanation**
A function object is a **callable object** — something you can call like a function. Regular functions are function objects, but so are objects of classes that define `operator()`. The STL uses function objects to customize algorithms: you can pass `std::greater<>` to sort in descending order, or a lambda to filter with a custom condition.

### Purposes

- To customize algorithm behavior (sorting order, filtering conditions).
- To encapsulate state alongside behavior.
- To provide compile-time polymorphism (templates) without virtual functions.
- To enable functional programming style in C++.
- To serve as predicates, comparators, and operations.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <functional>

struct MyFunctor {
    bool operator()(int x) const { return x > 0; }
};

std::sort(v.begin(), v.end(), std::greater<>{});   // built-in functor
```

**Common Function Objects**

| Category | Examples |
|----------|----------|
| Arithmetic | `plus`, `minus`, `multiplies`, `divides` |
| Comparison | `equal_to`, `not_equal_to`, `less`, `greater` |
| Logical | `logical_and`, `logical_or`, `logical_not` |
| Wrappers | `std::function`, `std::bind`, `std::not_fn` |

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `operator()` | The function call operator. |
| Predicate | A function object returning `bool`. |
| Comparator | A function object defining ordering. |
| Operation | A function object performing a transformation. |

**Syntax Rules**
- Function objects are passed by value to algorithms.
- They can hold state (e.g., a threshold).
- Lambdas are the most common way to create function objects in modern C++.
- `std::function` provides type erasure for callables.

**Constraints and Limitations**
- Function objects must be copyable (most are).
- Stateful function objects may have surprising behavior if copied by algorithms.
- `std::function` has overhead compared to direct functors or lambdas.

### Annotated Code Examples

**Example 1: Custom comparator with a function object**

```cpp
#include <algorithm>
#include <functional>
#include <iostream>
#include <vector>

int main() {
    std::vector<int> v = {3, 1, 4, 1, 5};
    std::sort(v.begin(), v.end(), std::greater<>{});   // descending
    for (int x : v) std::cout << x << ' ';   // 5 4 3 1 1
    std::cout << '\n';
}
```

**Expected Output**
```
5 4 3 1 1 
```

**Why this output?**  
`std::greater<>` is a function object that compares two values with `>`, producing descending order.

**Example 2: Stateful functor**

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

struct AboveThreshold {
    int threshold;
    bool operator()(int x) const { return x > threshold; }
};

int main() {
    std::vector<int> v = {1, 5, 2, 8, 3};
    auto it = std::find_if(v.begin(), v.end(), AboveThreshold{3});
    if (it != v.end()) std::cout << "First > 3: " << *it << '\n';   // 5
}
```

**Expected Output**
```
First > 3: 5
```

**Why this output?**  
`AboveThreshold{3}` is a function object holding a threshold; `find_if` uses it to find the first element greater than `3`.

### Real-World Cases

- **Sorting with custom orders** – `std::greater<>`, `std::less<>`, custom comparators.
- **Filtering** – predicates for `find_if`, `remove_if`, `copy_if`.
- **Transforming** – operations for `transform`.
- **Binding** – `std::bind` for partial application.
- **Type erasure** – `std::function` for storing callables.

---

## 5. Adaptors

### Definitions

**Core Definition**
An adaptor is a component that **converts one interface into another**, allowing a component to be used in a context that expects a different interface.

**Technical Definition**
Container adaptors provide a different interface for sequential containers. The three main container adaptors are `std::stack` (LIFO, adapted from `deque` by default), `std::queue` (FIFO, adapted from `deque` by default), and `std::priority_queue` (heap-ordered, adapted from `vector` by default). Iterator adaptors (e.g., `std::reverse_iterator`, `std::back_insert_iterator`) adapt iterators to behave differently. Function adaptors (e.g., `std::bind`, `std::not_fn`) adapt callables.

**Beginner-Friendly Explanation**
An adaptor is a **wrapper that changes how something behaves**. `std::stack` is an adaptor that takes a `deque` (or `vector`, or `list`) and restricts its interface to **last-in, first-out** operations (`push`, `pop`, `top`). `std::queue` restricts it to **first-in, first-out** (`push`, `pop`, `front`). `std::reverse_iterator` makes an iterator go backwards.

### Purposes

- To provide restricted or specialized interfaces over general containers.
- To adapt containers to specific abstract data types (stack, queue, priority queue).
- To adapt iterators for different traversal patterns.
- To adapt callables for binding, negation, or composition.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <stack>
#include <queue>

std::stack<int> s;               // LIFO, backed by deque<int>
std::queue<int> q;               // FIFO, backed by deque<int>
std::priority_queue<int> pq;     // max-heap, backed by vector<int>
```

**Component Breakdown**

| Adaptor | Underlying Container | Operations |
|---------|---------------------|------------|
| `std::stack<T>` | `deque<T>` (default) | `push`, `pop`, `top`, `empty`, `size` |
| `std::queue<T>` | `deque<T>` (default) | `push`, `pop`, `front`, `back`, `empty`, `size` |
| `std::priority_queue<T>` | `vector<T>` (default) | `push`, `pop`, `top`, `empty`, `size` |

**Syntax Rules**
- Container adaptors wrap a sequence container and restrict its interface.
- The underlying container can be specified as a template parameter.
- Iterator adaptors transform iterators (e.g., `std::back_inserter`).
- Function adaptors transform callables.

**Constraints and Limitations**
- Adaptors restrict functionality (e.g., no random access in `stack`).
- The underlying container must satisfy the adaptor's requirements.
- Iterator invalidation rules depend on the underlying container.

### Annotated Code Examples

**Example 1: `std::stack` — LIFO adaptor**

```cpp
#include <iostream>
#include <stack>

int main() {
    std::stack<int> s;
    s.push(1);
    s.push(2);
    s.push(3);
    while (!s.empty()) {
        std::cout << s.top() << ' ';   // 3 2 1
        s.pop();
    }
    std::cout << '\n';
}
```

**Expected Output**
```
3 2 1 
```

**Why this output?**  
`std::stack` provides LIFO access: the last element pushed (`3`) is the first popped.

**Example 2: `std::back_inserter` — iterator adaptor**

```cpp
#include <algorithm>
#include <iostream>
#include <iterator>
#include <vector>

int main() {
    std::vector<int> src = {1, 2, 3};
    std::vector<int> dst;
    std::copy(src.begin(), src.end(), std::back_inserter(dst));
    for (int x : dst) std::cout << x << ' ';   // 1 2 3
    std::cout << '\n';
}
```

**Expected Output**
```
1 2 3 
```

**Why this output?**  
`std::back_inserter(dst)` returns an iterator adaptor that calls `dst.push_back()` when written to, allowing `std::copy` to append to `dst`.

### Real-World Cases

- **`std::stack`** – undo/redo, expression evaluation, backtracking.
- **`std::queue`** – BFS, task scheduling, event handling.
- **`std::priority_queue`** – Dijkstra's algorithm, task prioritization.
- **`std::back_inserter`** – appending to containers in algorithms.
- **`std::reverse_iterator`** – reverse traversal.

---

## 6. Allocators

### Definitions

**Core Definition**
An allocator is an object that **encapsulates a memory allocation and deallocation strategy**, allowing containers to acquire and release memory without knowing the details of the underlying memory model.

**Technical Definition**
The `std::allocator` class template is the default allocator used by all standard library containers if no user-specified allocator is provided. The default allocator is stateless: all instances are interchangeable, compare equal, and can deallocate memory allocated by any other instance of the same allocator type. Custom allocators may contain state (since C++11) and control allocator replacement through `std::allocator_traits`. Allocators encapsulate memory models, decoupling algorithms from assumptions about a particular model.

**Beginner-Friendly Explanation**
An allocator is a **memory manager** that containers use to get and return memory. The default `std::allocator` uses `new` and `delete`. But you can plug in a custom allocator — for example, a pool allocator that reuses fixed-size blocks, or an allocator that places objects in shared memory. Allocators are the STL's way of letting you customize memory management without changing the container.

### Purposes

- To decouple containers from specific memory allocation strategies.
- To allow custom memory management (pools, arenas, shared memory).
- To support specialized memory models (embedded systems, high-performance computing).
- To enable tracking and debugging of memory usage.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <memory>

template <typename T, typename Allocator = std::allocator<T>>
class vector;

std::vector<int, MyAllocator<int>> v;   // custom allocator
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `std::allocator<T>` | Default allocator using `new`/`delete`. |
| `Allocator` template parameter | Custom allocator type. |
| `allocate(n)` | Allocates memory for `n` objects. |
| `deallocate(p, n)` | Deallocates memory. |

**Syntax Rules**
- Containers are templated on an allocator type (default: `std::allocator<T>`).
- Custom allocators must satisfy the *Allocator* requirements.
- Stateful allocators are supported since C++11.
- `std::allocator_traits` provides a uniform interface.

**Constraints and Limitations**
- Writing a correct custom allocator is nontrivial.
- Allocator propagation rules affect container copy/move behavior.
- The default allocator is stateless and interchangeable.

### Annotated Code Examples

**Example 1: Default allocator**

```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> v = {1, 2, 3};   // uses std::allocator<int> implicitly
    std::cout << v.size() << '\n';    // 3
}
```

**Expected Output**
```
3
```

**Why this output?**  
`std::vector<int>` uses `std::allocator<int>` by default to allocate memory for its elements.

**Example 2: Custom allocator (simplified)**

```cpp
#include <iostream>
#include <vector>
#include <memory>

template <typename T>
struct LoggingAllocator {
    using value_type = T;
    LoggingAllocator() = default;
    template <typename U> LoggingAllocator(const LoggingAllocator<U>&) {}
    T* allocate(std::size_t n) {
        std::cout << "alloc " << n << '\n';
        return std::allocator<T>{}.allocate(n);
    }
    void deallocate(T* p, std::size_t n) {
        std::allocator<T>{}.deallocate(p, n);
    }
    bool operator==(const LoggingAllocator&) const { return true; }
};

int main() {
    std::vector<int, LoggingAllocator<int>> v;
    v.push_back(1);
    v.push_back(2);
}
```

**Expected Output**
```
alloc 1
alloc 2
```

**Why this output?**  
The custom allocator logs each allocation. The vector uses it to acquire memory for its elements.

### Real-World Cases

- **Pool allocators** – game engines, real-time systems.
- **Shared-memory allocators** – inter-process communication.
- **Tracking allocators** – debugging and profiling.
- **`std::pmr`** – polymorphic memory resources (C++17).

---

## 7. STL Components

### Definitions

**Core Definition**
The STL components are the six interoperating categories of the library: **containers, iterators, algorithms, function objects, adaptors, and allocators**.

**Technical Definition**
The STL has six kinds of components: containers, iterators, generic algorithms, function objects, adaptors, and allocators. All of these are provided via C++ templates. Containers hold data; iterators provide uniform access; algorithms operate on ranges defined by iterators; function objects customize operations; adaptors transform interfaces; and allocators manage memory.

**Beginner-Friendly Explanation**
The STL is built from **six kinds of building blocks** that plug together. Containers hold your data. Iterators let algorithms walk through the data. Algorithms do the work. Function objects customize what “work” means. Adaptors change how containers or iterators behave. Allocators handle memory.

### How the Components Interact

| Component | Role | Interacts With |
|-----------|------|----------------|
| Containers | Hold data | Iterators, Allocators |
| Iterators | Traverse data | Containers, Algorithms |
| Algorithms | Process data | Iterators, Function Objects |
| Function Objects | Customize operations | Algorithms |
| Adaptors | Change interfaces | Containers, Iterators |
| Allocators | Manage memory | Containers |

**Syntax Rules**
- Algorithms receive iterators (not containers).
- Function objects are passed to algorithms.
- Containers use allocators for memory.
- Adaptors wrap containers or iterators.

**Constraints and Limitations**
- Components are designed to be composable but not all combinations are valid.
- Iterator category requirements must be satisfied.

### Annotated Code Examples

**Example 1: All six components in one program**

```cpp
#include <algorithm>
#include <functional>
#include <iostream>
#include <iterator>
#include <vector>

int main() {
    // Container: vector
    std::vector<int> v = {5, 2, 4, 1, 3};

    // Iterator: begin/end
    // Algorithm: sort, with Function Object: greater<>
    std::sort(v.begin(), v.end(), std::greater<>{});

    // Adaptor: back_inserter (iterator adaptor)
    std::vector<int> out;
    std::copy(v.begin(), v.end(), std::back_inserter(out));

    for (int x : out) std::cout << x << ' ';   // 5 4 3 2 1
    std::cout << '\n';
}
```

**Expected Output**
```
5 4 3 2 1 
```

**Why this output?**  
The `vector` container holds the data; iterators define the range; `sort` is the algorithm; `std::greater<>` is the function object; `back_inserter` is the iterator adaptor; the default allocator manages memory behind the scenes.

### Real-World Cases

- **Standard library** – the entire `<algorithm>`, `<vector>`, `<functional>` ecosystem.
- **Competitive programming** – using all components together.
- **Business applications** – data processing pipelines.

---

## 8. STL Best Practices

### Definitions

**Core Definition**
STL best practices are guidelines for using the Standard Template Library effectively, safely, and efficiently.

**Technical Definition**
Key practices include: prefer named STL algorithms to hand-written loops; use `std::vector` as the default container; reserve capacity when the size is known; choose the right container for the workload; avoid iterator invalidation; use range-based for loops for simple iteration; prefer `const` references and `auto&` appropriately; and understand iterator categories. These practices are widely documented in community resources and style guides.

**Beginner-Friendly Explanation**
Best practices for the STL are simple: **use `std::vector` by default**, **prefer algorithms over loops**, **reserve memory when you know the size**, **pick the right container for the job**, and **don't invalidate iterators**.

### Key Guidelines (Summary Table)

| Guideline | Summary | Recommendation |
|-----------|---------|----------------|
| **Prefer algorithms** | Named algorithms are clearer and less error-prone than hand-written loops. | `std::find_if` instead of a manual loop. |
| **`std::vector` is the default** | Reach for another container only with a reason. | Use `vector` unless you need specific properties. |
| **Reserve capacity** | Avoid reallocation overhead. | `v.reserve(n)` before `push_back` loops. |
| **Choose the right container** | Match container to workload. | `list` for mid-insertions; `map` for sorted lookups; `unordered_map` for fast lookups. |
| **Avoid iterator invalidation** | Modifying a container can invalidate iterators. | Re-obtain iterators after modification. |
| **Use range-based for** | Simple iteration is clearer. | `for (const auto& x : v)`. |
| **Use `const auto&` for read-only** | Avoid copies. | `for (const auto& x : v)`. |
| **Use `auto&` to modify** | Modify in place. | `for (auto& x : v) x *= 2;`. |
| **Understand iterator categories** | Algorithms have different requirements. | `sort` needs random access; `find` needs input. |
| **Avoid mixing `new`/`delete` with STL** | Containers manage memory. | Use containers and smart pointers. |

### Annotated Code Examples

**Example 1: Algorithm over hand-written loop**

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main() {
    std::vector<int> v = {1, 2, 3, 4, 5};
    // GOOD: named algorithm
    auto it = std::find_if(v.begin(), v.end(), [](int x) { return x > 3; });
    if (it != v.end()) std::cout << *it << '\n';   // 4
}
```

**Expected Output**
```
4
```

**Why this is best practice?**  
`std::find_if` communicates intent clearly and avoids manual loop errors.

**Example 2: Reserve capacity**

```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> v;
    v.reserve(100);   // avoid reallocations
    for (int i = 0; i < 100; ++i) v.push_back(i);
    std::cout << v.size() << '\n';   // 100
}
```

**Expected Output**
```
100
```

**Why this is best practice?**  
`reserve(100)` pre-allocates memory, avoiding the cost of repeated reallocations as the vector grows.

**Example 3: Choosing the right container**

```cpp
#include <iostream>
#include <map>
#include <unordered_map>
#include <vector>

int main() {
    // Ordered lookups: std::map
    std::map<std::string, int> ordered;
    ordered["b"] = 2;
    ordered["a"] = 1;

    // Fast lookups: std::unordered_map
    std::unordered_map<std::string, int> hashed;
    hashed["a"] = 1;

    // Default: std::vector
    std::vector<int> vec = {1, 2, 3};

    std::cout << "ok\n";
}
```

**Expected Output**
```
ok
```

**Why this is best practice?**  
Each container is chosen for its strengths: `map` for ordering, `unordered_map` for speed, `vector` for general-purpose sequences.

### Real-World Cases

- **Standard library** – all standard containers and algorithms follow these guidelines.
- **Code review** – algorithm preference and container choice are common review points.
- **Performance optimization** – reserve and container selection are critical.

---

## References

- Containers library (cppreference) — https://en.cppreference.com/w/cpp/container
- Iterator library (cppreference) — https://en.cppreference.com/w/cpp/iterator
- Algorithms library (cppreference) — https://en.cppreference.com/w/cpp/algorithm
- Function objects (cppreference) — https://en.cppreference.com/w/cpp/utility/functional
- `std::allocator` (cppreference) — https://en.cppreference.com/w/cpp/memory/allocator
- Standard Template Library (Virginia Tech) — http://courses.cs.vt.edu/~cs2704/spring00/phanouri/Slides/Topic08_6p.pdf
- Programming with the Standard Template Library (Pace University) — http://csis.pace.edu/~bergin/papers/STL.txt
- C++ STL Algorithms and Containers (rocm-systems) — https://github.com/rocm/rocm-systems/blob/77f7e9e6d92649fcbd7c9f6e64703214bf783f98/projects/rocprofiler-compute/.ai/rules/cpp/stl-algorithms.md
- C++ STL Master Guide (LeetCode Discuss) — https://leetcode.com/discuss/post/7345713/cpp-stl-master-guide-containers-algorithms-and-patterns-you-must-know/
- Introduction to the C++ Standard Library (UET Taxila) — http://web.uettaxila.edu.pk/CMS/teOOPbsSp09/notes%5CL4_stl_types.pdf
- 301AA Advanced Programming — STL (University of Pisa) — https://pages.di.unipi.it/corradini/Didattica/AP-23/SLIDES/AP-2023-12-STL.pdf

---

This cheat sheet provides a complete, structured reference for C++ STL fundamentals, covering containers, iterators, algorithms, function objects, adaptors, allocators, the six-component model, and best practices. Each section follows a uniform template with definitions, purposes, syntax rules, annotated examples, expected outputs, and authoritative references.