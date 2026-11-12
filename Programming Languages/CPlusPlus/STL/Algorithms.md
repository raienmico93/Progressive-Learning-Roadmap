# C++ Algorithms — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
The C++ algorithms library provides a collection of function templates that operate on ranges of elements (typically defined by iterator pairs) to perform common operations such as searching, sorting, counting, and modifying data.

**Technical Definition**
The `<algorithm>` and `<numeric>` headers define a large set of function templates that operate on ranges defined as `[first, last)`. These algorithms are decoupled from the containers they operate on—they accept iterators as arguments, allowing them to work with any sequence that satisfies the required iterator category. The standard specifies complexity guarantees for each algorithm, and many algorithms are `constexpr` (since C++20) and support execution policies for parallelisation (since C++17) .

**Beginner-Friendly Explanation**
Think of the algorithms library as a toolbox of ready-made operations you can apply to any collection of data—vectors, lists, arrays, or even raw C arrays. Instead of writing loops by hand to search, sort, or transform data, you call a named algorithm that does the job correctly and efficiently.

### Key Characteristics

- **Range-based** – algorithms operate on half-open ranges `[first, last)`, where `first` points to the first element and `last` points one past the last element .
- **Iterator-driven** – algorithms are parameterised by iterator types, so they work with any container that provides suitable iterators.
- **Complexity guarantees** – each algorithm has a specified time complexity (e.g., O(n), O(log n), O(n log n)).
- **Non-modifying vs. modifying** – some algorithms only read elements; others modify them or write to output ranges.
- **Policy-based** – many algorithms accept an optional execution policy for parallel execution (C++17).
- **Overload sets** – most algorithms come in multiple overloads (with and without predicates/comparators).

### Prerequisites

- Basic C++ syntax (variables, types, control flow).
- Understanding of iterators and iterator categories.
- Familiarity with STL containers (`vector`, `list`, `map`, etc.).
- Awareness of Big-O notation and algorithmic complexity.

### Related Programming Areas

- **Iterators** – the interface between algorithms and containers.
- **Containers** – the data structures algorithms operate on.
- **Ranges (C++20)** – a higher-level abstraction built on top of iterators and algorithms.
- **Numeric computation** – `<numeric>` provides accumulation, reduction, and scan operations.
- **Parallelism** – execution policies enable parallel algorithm execution.

### Core Concepts / Features

1. `<algorithm>` (the header and library)
2. Non-modifying algorithms
3. Modifying algorithms
4. Sorting algorithms
5. Binary search algorithms
6. Set operations
7. Heap operations
8. Min/max operations
9. Numeric algorithms
10. Algorithm best practices

---

## 1. `<algorithm>` — The Algorithms Header

### Definitions

**Core Definition**
`<algorithm>` is the standard library header that declares the non-modifying, modifying, sorting, searching, set, heap, and min/max algorithms.

**Technical Definition**
The header `<algorithm>` provides the majority of the general-purpose algorithm function templates. The related header `<numeric>` provides numeric algorithms such as `accumulate`, `reduce`, and `partial_sum`. Most algorithms are function templates that accept iterator ranges and optional predicates or comparison functions .

**Beginner-Friendly Explanation**
`<algorithm>` is the main toolbox header. If you want to sort, search, count, or transform data, you include this header.

### Purposes

- To provide reusable, tested implementations of common operations.
- To replace hand-written loops with clear, declarative code.
- To offer complexity guarantees and optimised implementations.
- To support generic programming across all container types.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <algorithm>
std::algorithm_name(first, last, ...);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `first`, `last` | Iterators defining the range `[first, last)`. |
| Optional arguments | Predicates, comparators, values, output iterators. |

**Syntax Rules**
- The range `[first, last)` must be valid; `last` must be reachable from `first`.
- Predicates must be callable with the element type and return a value convertible to `bool`.
- Comparators must impose a strict weak ordering.
- Algorithms do not resize containers; output must fit in pre-allocated space or use insert iterators.

**Constraints and Limitations**
- Algorithms cannot change the size of a container; use container member functions for resizing.
- Iterator invalidation rules of the underlying container still apply.
- Some algorithms require specific iterator categories (e.g., `std::sort` requires random access).

### Annotated Code Example

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {3, 1, 4, 1, 5};
    std::sort(v.begin(), v.end());         // sort the range
    for (int x : v) std::cout << x << ' ';
}
```

**Expected Output**
```
1 1 3 4 5
```

**Why this output?**  
`std::sort` reorders the elements in `[v.begin(), v.end())` into ascending order.

---

## 2. Non-Modifying Algorithms

### Definitions

**Core Definition**
Non-modifying algorithms read elements from a range without changing them.

**Technical Definition**
These algorithms satisfy the `NonModifyingSequenceOperation` requirements: they operate on input ranges and do not modify any element. They include checks (`all_of`, `any_of`, `none_of`), iteration (`for_each`), counting (`count`, `count_if`), searching (`find`, `find_if`, `find_if_not`, `find_end`, `find_first_of`, `adjacent_find`, `search`, `search_n`), and comparison (`mismatch`, `equal`, `is_permutation`) .

**Beginner-Friendly Explanation**
Non-modifying algorithms answer questions about your data or visit each element without changing anything.

---

### 2.1 `std::all_of`, `std::any_of`, `std::none_of`

#### Definitions

**Core Definition**
These algorithms check whether a predicate holds for all, any, or none of the elements in a range.

**Technical Definition**
`all_of` returns `true` if the predicate returns `true` for every element (or if the range is empty). `any_of` returns `true` if the predicate returns `true` for at least one element. `none_of` returns `true` if the predicate returns `false` for every element .

**Beginner-Friendly Explanation**
These are “yes/no” questions about a whole range: “Are all elements even?” “Is any element negative?” “Are none of them zero?”

#### Purposes

- To test a condition across an entire range.
- To replace explicit loops with declarative checks.
- To short-circuit as soon as the answer is known.

#### Syntax Rules and Structure

**General Syntax**
```cpp
bool all_of(InputIt first, InputIt last, Predicate pred);
bool any_of(InputIt first, InputIt last, Predicate pred);
bool none_of(InputIt first, InputIt last, Predicate pred);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `first`, `last` | Input iterators defining the range. |
| `pred` | Unary predicate returning `bool`. |

**Complexity**
- At most `last - first` applications of `pred`.

#### Annotated Code Example

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {2, 4, 6};
    std::cout << std::boolalpha;
    std::cout << std::all_of(v.begin(), v.end(), [](int x){ return x % 2 == 0; }) << '\n'; // true
    std::cout << std::any_of(v.begin(), v.end(), [](int x){ return x > 5; }) << '\n';       // true
    std::cout << std::none_of(v.begin(), v.end(), [](int x){ return x < 0; }) << '\n';      // true
}
```

**Expected Output**
```
true
true
true
```

**Why this output?**  
All elements are even; at least one is greater than 5; none are negative.

---

### 2.2 `std::for_each`

#### Definitions

**Core Definition**
`std::for_each` applies a function to every element in a range.

**Technical Definition**
`for_each(first, last, f)` applies `f` to the result of dereferencing every iterator in `[first, last)` and returns `std::move(f)` (since C++11). The function object may modify elements if the iterator is mutable.

**Beginner-Friendly Explanation**
`for_each` is like a `for` loop that visits every element and does something with it.

#### Purposes

- To apply an operation to each element.
- To replace explicit loops with a named algorithm.
- To support side effects (printing, accumulating, modifying).

#### Syntax Rules and Structure

**General Syntax**
```cpp
Function for_each(InputIt first, InputIt last, Function f);
```

**Complexity**
- Exactly `last - first` applications of `f`.

#### Annotated Code Example

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {1, 2, 3};
    std::for_each(v.begin(), v.end(), [](int x){ std::cout << x * 2 << ' '; });
}
```

**Expected Output**
```
2 4 6
```

**Why this output?**  
The lambda is applied to each element, printing its double.

---

### 2.3 `std::count`, `std::count_if`

#### Definitions

**Core Definition**
`count` counts elements equal to a value; `count_if` counts elements satisfying a predicate.

**Technical Definition**
`count(first, last, value)` returns the number of iterators `i` in `[first, last)` for which `*i == value`. `count_if(first, last, pred)` returns the number of iterators for which `pred(*i)` is `true`.

**Beginner-Friendly Explanation**
These algorithms tell you how many items match a condition or equal a particular value.

#### Syntax Rules and Structure

**General Syntax**
```cpp
difference_type count(InputIt first, InputIt last, const T& value);
difference_type count_if(InputIt first, InputIt last, Predicate pred);
```

**Complexity**
- Exactly `last - first` comparisons/applications.

#### Annotated Code Example

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {1, 2, 2, 3, 2};
    std::cout << std::count(v.begin(), v.end(), 2) << '\n';           // 3
    std::cout << std::count_if(v.begin(), v.end(), [](int x){ return x > 1; }) << '\n'; // 4
}
```

**Expected Output**
```
3
4
```

**Why this output?**  
There are three 2s; four elements are greater than 1.

---

### 2.4 `std::find`, `std::find_if`, `std::find_if_not`

#### Definitions

**Core Definition**
These algorithms locate the first element that matches a value or satisfies (or fails) a predicate.

**Technical Definition**
`find` returns an iterator to the first element equal to `value`, or `last` if none. `find_if` returns an iterator to the first element for which `pred` is `true`, or `last`. `find_if_not` returns an iterator to the first element for which `pred` is `false`, or `last` .

**Beginner-Friendly Explanation**
These are search functions: they find the first item that meets a condition.

#### Syntax Rules and Structure

**General Syntax**
```cpp
InputIt find(InputIt first, InputIt last, const T& value);
InputIt find_if(InputIt first, InputIt last, Predicate pred);
InputIt find_if_not(InputIt first, InputIt last, Predicate pred);
```

**Complexity**
- At most `last - first` comparisons/applications.

#### Annotated Code Example

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {10, 20, 30, 40};
    auto it = std::find(v.begin(), v.end(), 30);
    if (it != v.end()) std::cout << "Found: " << *it << '\n';

    auto it2 = std::find_if(v.begin(), v.end(), [](int x){ return x > 25; });
    std::cout << "First > 25: " << *it2 << '\n';
}
```

**Expected Output**
```
Found: 30
First > 25: 30
```

---

### 2.5 `std::find_end`, `std::find_first_of`, `std::adjacent_find`

#### Definitions

**Core Definition**
These algorithms search for subsequences or adjacent equal elements within a range.

**Technical Definition**
`find_end` searches for the last occurrence of a subsequence `[s_first, s_last)`. `find_first_of` searches for the first element that matches any element in a given set. `adjacent_find` finds the first pair of adjacent elements that are equal (or satisfy a predicate) .

**Complexity**
- `find_end`: at most `(last1 - first1) * (last2 - first2)` comparisons.
- `find_first_of`: at most `(last1 - first1) * (last2 - first2)` comparisons.
- `adjacent_find`: exactly `min((last - first) - 1, ...)` applications.

#### Annotated Code Example

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {1, 2, 3, 2, 3, 4};
    std::vector<int> sub = {2, 3};

    auto it = std::find_end(v.begin(), v.end(), sub.begin(), sub.end());
    std::cout << "Last occurrence starts at index: " << (it - v.begin()) << '\n'; // 3

    std::vector<int> targets = {3, 5};
    auto it2 = std::find_first_of(v.begin(), v.end(), targets.begin(), targets.end());
    std::cout << "First match: " << *it2 << '\n'; // 3

    std::vector<int> adj = {1, 1, 2, 2, 3};
    auto it3 = std::adjacent_find(adj.begin(), adj.end());
    std::cout << "Adjacent equal: " << *it3 << '\n'; // 1
}
```

**Expected Output**
```
Last occurrence starts at index: 3
First match: 3
Adjacent equal: 1
```

---

### 2.6 `std::search`, `std::search_n`

#### Definitions

**Core Definition**
`search` finds the first occurrence of a subsequence; `search_n` finds the first occurrence of `count` consecutive copies of a value.

**Technical Definition**
`search` returns an iterator to the beginning of the first occurrence of the subsequence `[s_first, s_last)` in `[first, last)`. `search_n` returns an iterator to the beginning of the first sequence of `count` consecutive elements equal to `value` (or satisfying a predicate).

**Complexity**
- `search`: at most `(last1 - first1) * (last2 - first2)` comparisons.
- `search_n`: at most `(last - first) * count` comparisons.

#### Annotated Code Example

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {1, 2, 3, 4, 5, 3, 4};
    std::vector<int> sub = {3, 4};

    auto it = std::search(v.begin(), v.end(), sub.begin(), sub.end());
    std::cout << "First occurrence at index: " << (it - v.begin()) << '\n'; // 2

    auto it2 = std::search_n(v.begin(), v.end(), 2, 3);
    std::cout << "Two consecutive 3s: " << (it2 != v.end()) << '\n'; // 0 (false)
}
```

**Expected Output**
```
First occurrence at index: 2
Two consecutive 3s: 0
```

---

### 2.7 `std::mismatch`, `std::equal`, `std::is_permutation`

#### Definitions

**Core Definition**
These algorithms compare two ranges.

**Technical Definition**
`mismatch` returns a pair of iterators pointing to the first position where two ranges differ (or `last` if they are equal up to that point). `equal` returns `true` if two ranges are element-wise equal. `is_permutation` returns `true` if one range is a permutation of the other.

**Complexity**
- `mismatch`/`equal`: at most `min(last1 - first1, last2 - first2)` comparisons.
- `is_permutation`: at most O(n²) comparisons in the worst case.

#### Annotated Code Example

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> a = {1, 2, 3, 4};
    std::vector<int> b = {1, 2, 9, 4};

    auto [mi1, mi2] = std::mismatch(a.begin(), a.end(), b.begin());
    std::cout << "Mismatch: " << *mi1 << " vs " << *mi2 << '\n'; // 3 vs 9

    std::cout << std::boolalpha;
    std::cout << std::equal(a.begin(), a.end(), b.begin(), b.end()) << '\n'; // false

    std::vector<int> p = {3, 1, 4, 2};
    std::cout << std::is_permutation(a.begin(), a.end(), p.begin(), p.end()) << '\n'; // true
}
```

**Expected Output**
```
Mismatch: 3 vs 9
false
true
```

---

## 3. Modifying Algorithms

### Definitions

**Core Definition**
Modifying algorithms change the elements of a range or write elements to an output range.

**Technical Definition**
These algorithms include copy/move operations, swap operations, transformation operations (`replace`, `transform`), generation operations (`fill`, `generate`), removal operations (`remove`, `remove_if`, `unique`), order-changing operations (`reverse`, `rotate`, `shuffle`, `sample`), and assignment operations .

**Beginner-Friendly Explanation**
Modifying algorithms change your data—they copy, replace, fill, remove, reverse, or shuffle elements.

---

### 3.1 `std::copy`, `std::copy_if`, `std::copy_n`, `std::copy_backward`

#### Definitions

**Core Definition**
These algorithms copy elements from a source range to a destination range.

**Technical Definition**
`copy` copies all elements from `[first, last)` to `d_first`. `copy_if` copies only elements satisfying a predicate. `copy_n` copies exactly `n` elements. `copy_backward` copies elements in reverse order (starting from the end) to handle overlapping ranges.

**Complexity**
- `copy`, `copy_if`, `copy_n`: exactly `last - first` (or `n`) assignments.
- `copy_backward`: exactly `last - first` assignments.

#### Annotated Code Example

```cpp
#include <algorithm>
#include <vector>
#include <iostream>
#include <iterator>

int main() {
    std::vector<int> src = {1, 2, 3, 4, 5};
    std::vector<int> dst;

    std::copy(src.begin(), src.end(), std::back_inserter(dst));
    // dst = {1, 2, 3, 4, 5}

    std::vector<int> evens;
    std::copy_if(src.begin(), src.end(), std::back_inserter(evens),
                 [](int x){ return x % 2 == 0; });
    // evens = {2, 4}

    for (int x : evens) std::cout << x << ' ';
}
```

**Expected Output**
```
2 4
```

---

### 3.2 `std::move`, `std::move_backward`

#### Definitions

**Core Definition**
These algorithms move elements from a source range to a destination range, leaving the source in a moved-from state.

**Technical Definition**
`move` moves all elements from `[first, last)` to `d_first` using move assignment. `move_backward` moves elements in reverse order, useful for overlapping ranges.

**Complexity**
- Exactly `last - first` move assignments.

#### Annotated Code Example

```cpp
#include <algorithm>
#include <vector>
#include <iostream>
#include <string>

int main() {
    std::vector<std::string> src = {"hello", "world"};
    std::vector<std::string> dst(2);

    std::move(src.begin(), src.end(), dst.begin());

    std::cout << dst[0] << ' ' << dst[1] << '\n'; // hello world
    std::cout << "src[0] empty: " << src[0].empty() << '\n'; // 1
}
```

**Expected Output**
```
hello world
src[0] empty: 1
```

---

### 3.3 `std::fill`, `std::fill_n`

#### Definitions

**Core Definition**
`fill` assigns a value to every element in a range; `fill_n` assigns a value to `n` elements starting from a given iterator.

**Complexity**
- Exactly `last - first` (or `n`) assignments.

#### Annotated Code Example

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v(5);
    std::fill(v.begin(), v.end(), 7);
    for (int x : v) std::cout << x << ' '; // 7 7 7 7 7
}
```

**Expected Output**
```
7 7 7 7 7
```

---

### 3.4 `std::transform`

#### Definitions

**Core Definition**
`transform` applies a function to each element of a range and stores the result in an output range.

**Technical Definition**
The unary version applies a unary operation to each element in `[first, last)` and writes the result to `d_first`. The binary version applies a binary operation to corresponding elements from two input ranges.

**Complexity**
- Exactly `last - first` applications of the operation.

#### Annotated Code Example

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {1, 2, 3};
    std::vector<int> out(v.size());

    std::transform(v.begin(), v.end(), out.begin(),
                   [](int x){ return x * x; });

    for (int x : out) std::cout << x << ' '; // 1 4 9
}
```

**Expected Output**
```
1 4 9
```

---

### 3.5 `std::generate`, `std::generate_n`

#### Definitions

**Core Definition**
`generate` assigns the result of successive calls to a generator function to every element in a range; `generate_n` does so for `n` elements.

**Complexity**
- Exactly `last - first` (or `n`) invocations of the generator.

#### Annotated Code Example

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v(5);
    int counter = 0;
    std::generate(v.begin(), v.end(), [&counter]{ return counter++; });
    for (int x : v) std::cout << x << ' '; // 0 1 2 3 4
}
```

**Expected Output**
```
0 1 2 3 4
```

---

### 3.6 `std::remove`, `std::remove_if`, `std::remove_copy`, `std::remove_copy_if`

#### Definitions

**Core Definition**
`remove` and `remove_if` logically remove elements by moving non-removed elements to the front of the range (they do not actually erase elements from the container). The `_copy` variants copy non-removed elements to a destination range.

**Technical Definition**
`remove(first, last, value)` moves all elements not equal to `value` to the front and returns an iterator to the new logical end. `remove_if` uses a predicate instead of a value. `remove_copy` and `remove_copy_if` write the kept elements to an output range .

**Complexity**
- Exactly `last - first` applications of the predicate/comparison.

#### Annotated Code Example

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {1, 2, 3, 2, 4};
    auto new_end = std::remove(v.begin(), v.end(), 2);

    // Actually erase the removed elements
    v.erase(new_end, v.end());

    for (int x : v) std::cout << x << ' '; // 1 3 4
}
```

**Expected Output**
```
1 3 4
```

**Important note:** `std::remove` does not change the container’s size; you must call `erase` to physically remove the elements.

---

### 3.7 `std::replace`, `std::replace_if`, `std::replace_copy`, `std::replace_copy_if`

#### Definitions

**Core Definition**
`replace` replaces all elements equal to a value with a new value. `replace_if` replaces elements satisfying a predicate. The `_copy` variants write the result to a destination range.

**Complexity**
- Exactly `last - first` comparisons/applications.

#### Annotated Code Example

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {1, 2, 3, 2, 4};
    std::replace(v.begin(), v.end(), 2, 99);
    for (int x : v) std::cout << x << ' '; // 1 99 3 99 4
}
```

**Expected Output**
```
1 99 3 99 4
```

---

### 3.8 `std::swap`, `std::swap_ranges`, `std::iter_swap`

#### Definitions

**Core Definition**
`swap` exchanges the values of two objects; `swap_ranges` exchanges the elements of two ranges; `iter_swap` swaps the values pointed to by two iterators.

**Complexity**
- `swap`: O(1) for most types.
- `swap_ranges`: exactly `last1 - first1` swaps.
- `iter_swap`: O(1).

#### Annotated Code Example

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> a = {1, 2, 3};
    std::vector<int> b = {4, 5, 6};

    std::swap_ranges(a.begin(), a.end(), b.begin());
    for (int x : a) std::cout << x << ' '; // 4 5 6
}
```

**Expected Output**
```
4 5 6
```

---

### 3.9 `std::reverse`, `std::reverse_copy`

#### Definitions

**Core Definition**
`reverse` reverses the order of elements in a range. `reverse_copy` writes the reversed sequence to a destination range.

**Complexity**
- `reverse`: exactly `(last - first) / 2` swaps.
- `reverse_copy`: exactly `last - first` assignments.

#### Annotated Code Example

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {1, 2, 3, 4};
    std::reverse(v.begin(), v.end());
    for (int x : v) std::cout << x << ' '; // 4 3 2 1
}
```

**Expected Output**
```
4 3 2 1
```

---

### 3.10 `std::rotate`, `std::rotate_copy`

#### Definitions

**Core Definition**
`rotate` performs a left rotation on a range such that `middle` becomes the new first element. `rotate_copy` writes the rotated sequence to a destination range.

**Complexity**
- `rotate`: at most `last - first` swaps.
- `rotate_copy`: exactly `last - first` assignments.

#### Annotated Code Example

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {1, 2, 3, 4, 5};
    std::rotate(v.begin(), v.begin() + 2, v.end());
    for (int x : v) std::cout << x << ' '; // 3 4 5 1 2
}
```

**Expected Output**
```
3 4 5 1 2
```

---

### 3.11 `std::shuffle`, `std::sample`

#### Definitions

**Core Definition**
`shuffle` randomly reorders the elements of a range using a uniform random bit generator. `sample` selects `n` elements from a range without replacement.

**Complexity**
- `shuffle`: O(n) swaps.
- `sample`: O(n) applications.

#### Annotated Code Example

```cpp
#include <algorithm>
#include <vector>
#include <iostream>
#include <random>

int main() {
    std::vector<int> v = {1, 2, 3, 4, 5};
    std::random_device rd;
    std::mt19937 g(rd());
    std::shuffle(v.begin(), v.end(), g);
    for (int x : v) std::cout << x << ' '; // some permutation
}
```

**Expected Output** (example)
```
3 1 5 2 4
```

---

### 3.12 `std::unique`, `std::unique_copy`

#### Definitions

**Core Definition**
`unique` eliminates consecutive duplicate elements from a range (logically, by moving kept elements to the front). `unique_copy` writes the unique elements to a destination range.

**Technical Definition**
`unique` returns an iterator to the new logical end; the elements between the new end and the original end are unspecified. To physically remove them, call `erase` .

**Complexity**
- Exactly `last - first - 1` comparisons (or predicate applications).

#### Annotated Code Example

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {1, 1, 2, 2, 3, 3, 3, 4};
    auto new_end = std::unique(v.begin(), v.end());
    v.erase(new_end, v.end());
    for (int x : v) std::cout << x << ' '; // 1 2 3 4
}
```

**Expected Output**
```
1 2 3 4
```

---

## 4. Sorting Algorithms

### Definitions

**Core Definition**
Sorting algorithms rearrange the elements of a range into a specified order (by default, ascending).

**Technical Definition**
The standard provides several sorting algorithms with different complexity and stability guarantees: `sort` (O(n log n) average, not stable), `stable_sort` (O(n log n) with memory, stable), `partial_sort` (sorts only the first k elements), `partial_sort_copy`, `nth_element` (places the n-th element in its sorted position), `is_sorted`, and `is_sorted_until` .

**Beginner-Friendly Explanation**
Sorting algorithms put your data in order—either all of it, part of it, or just enough to find the n-th element.

---

### 4.1 `std::sort`

#### Definitions

**Core Definition**
`std::sort` sorts the elements of a range in ascending order (or according to a comparator).

**Technical Definition**
`sort(first, last)` sorts `[first, last)` using an introsort variant. Average complexity is O(n log n); worst case is O(n log n). The algorithm is **not stable**—equal elements may change relative order.

**Complexity**
- O(n log n) comparisons.

#### Annotated Code Example

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {3, 1, 4, 1, 5};
    std::sort(v.begin(), v.end());
    for (int x : v) std::cout << x << ' '; // 1 1 3 4 5
}
```

**Expected Output**
```
1 1 3 4 5
```

---

### 4.2 `std::stable_sort`

#### Definitions

**Core Definition**
`std::stable_sort` sorts the range while preserving the relative order of equivalent elements.

**Technical Definition**
Uses a merge-sort variant. Complexity is O(n log n) with additional memory, or O(n log²n) without.

**Complexity**
- O(n log n) with O(n) extra memory, or O(n log²n) in-place.

#### Annotated Code Example

```cpp
#include <algorithm>
#include <vector>
#include <iostream>
#include <string>

struct Person { std::string name; int age; };

int main() {
    std::vector<Person> people = {{"Alice", 30}, {"Bob", 25}, {"Charlie", 30}};
    std::stable_sort(people.begin(), people.end(),
                     [](const Person& a, const Person& b){ return a.age < b.age; });
    for (const auto& p : people) std::cout << p.name << ' '; // Bob Alice Charlie
}
```

**Expected Output**
```
Bob Alice Charlie
```

**Why stable?** Alice and Charlie have the same age; Alice appears before Charlie because stable_sort preserves their original relative order.

---

### 4.3 `std::partial_sort`, `std::partial_sort_copy`

#### Definitions

**Core Definition**
`partial_sort` sorts only the first `k` elements of a range, leaving the rest unspecified. `partial_sort_copy` copies and partially sorts into a destination range.

**Complexity**
- `partial_sort`: O(n log k) comparisons.
- `partial_sort_copy`: O(n log k) comparisons.

#### Annotated Code Example

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {5, 2, 4, 1, 3};
    std::partial_sort(v.begin(), v.begin() + 3, v.end());
    for (int x : v) std::cout << x << ' '; // 1 2 3 ? ?
}
```

**Expected Output** (first three sorted; remainder unspecified)
```
1 2 3 5 4
```

---

### 4.4 `std::is_sorted`, `std::is_sorted_until`

#### Definitions

**Core Definition**
`is_sorted` checks whether a range is sorted. `is_sorted_until` returns an iterator to the first element that violates sorted order.

**Complexity**
- `is_sorted`: at most `last - first - 1` comparisons.
- `is_sorted_until`: at most `last - first - 1` comparisons.

#### Annotated Code Example

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {1, 2, 3, 5, 4};
    std::cout << std::boolalpha;
    std::cout << std::is_sorted(v.begin(), v.end()) << '\n'; // false

    auto it = std::is_sorted_until(v.begin(), v.end());
    std::cout << "First unsorted element: " << *it << '\n'; // 4
}
```

**Expected Output**
```
false
First unsorted element: 4
```

---

### 4.5 `std::nth_element`

#### Definitions

**Core Definition**
`nth_element` places the element that would be at position `n` if the range were fully sorted, with all smaller elements before it and all larger elements after it.

**Technical Definition**
Average complexity is O(n). The relative order of elements before and after `n` is unspecified.

**Complexity**
- O(n) comparisons on average.

#### Annotated Code Example

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {5, 2, 4, 1, 3};
    std::nth_element(v.begin(), v.begin() + 2, v.end());
    std::cout << "Median: " << v[2] << '\n'; // 3
}
```

**Expected Output**
```
Median: 3
```

---

## 5. Binary Search Algorithms

### Definitions

**Core Definition**
Binary search algorithms operate on sorted (or partitioned) ranges to find elements or insertion points in logarithmic time.

**Technical Definition**
The standard provides `lower_bound` (first element not less than a value), `upper_bound` (first element greater than a value), `binary_search` (whether a value exists), and `equal_range` (pair of `lower_bound` and `upper_bound`). They require the range to be partitioned with respect to the search key .

**Beginner-Friendly Explanation**
Binary search is like looking up a word in a dictionary—you halve the search space each time. These algorithms require the data to already be sorted.

---

### 5.1 `std::lower_bound`, `std::upper_bound`

#### Definitions

**Core Definition**
`lower_bound` returns an iterator to the first element that is **not less than** a given value. `upper_bound` returns an iterator to the first element that is **greater than** the value.

**Complexity**
- O(log n) comparisons for random-access iterators.
- O(n) for non-random-access iterators (due to linear advancement).

#### Annotated Code Example

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {10, 20, 30, 40, 50};

    auto lb = std::lower_bound(v.begin(), v.end(), 30);
    std::cout << "lower_bound(30): index " << (lb - v.begin()) << '\n'; // 2

    auto ub = std::upper_bound(v.begin(), v.end(), 30);
    std::cout << "upper_bound(30): index " << (ub - v.begin()) << '\n'; // 3
}
```

**Expected Output**
```
lower_bound(30): index 2
upper_bound(30): index 3
```

---

### 5.2 `std::binary_search`

#### Definitions

**Core Definition**
`binary_search` checks whether a given value exists in a sorted range.

**Technical Definition**
Returns `true` if there is an element equivalent to `value`; otherwise `false`.

**Complexity**
- O(log n) comparisons.

#### Annotated Code Example

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {10, 20, 30, 40};
    std::cout << std::boolalpha;
    std::cout << std::binary_search(v.begin(), v.end(), 30) << '\n'; // true
    std::cout << std::binary_search(v.begin(), v.end(), 35) << '\n'; // false
}
```

**Expected Output**
```
true
false
```

---

### 5.3 `std::equal_range`

#### Definitions

**Core Definition**
`equal_range` returns a pair of iterators delimiting the subrange of elements equivalent to a given value.

**Technical Definition**
Equivalent to `{lower_bound(first, last, value), upper_bound(first, last, value)}`.

**Complexity**
- O(log n) comparisons.

#### Annotated Code Example

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {10, 20, 20, 20, 30};
    auto [lo, hi] = std::equal_range(v.begin(), v.end(), 20);
    std::cout << "Number of 20s: " << (hi - lo) << '\n'; // 3
}
```

**Expected Output**
```
Number of 20s: 3
```

---

## 6. Set Operations

### Definitions

**Core Definition**
Set operations compute unions, intersections, differences, and related operations on sorted ranges.

**Technical Definition**
The set algorithms (`set_union`, `set_intersection`, `set_difference`, `set_symmetric_difference`, `includes`, `merge`, `inplace_merge`) require sorted input ranges and produce sorted output. Their semantics generalise to multisets: `set_union` contains the maximum number of occurrences, `set_intersection` the minimum, and so on .

**Beginner-Friendly Explanation**
Set operations treat your sorted data as mathematical sets, computing combinations like “everything in both sets” or “everything in one but not the other.”

---

### 6.1 `std::merge`, `std::inplace_merge`

#### Definitions

**Core Definition**
`merge` combines two sorted ranges into a single sorted output range. `inplace_merge` merges two consecutive sorted subranges within the same range.

**Complexity**
- `merge`: O(n + m) comparisons.
- `inplace_merge`: O(n log n) worst case, O(n) with extra memory.

#### Annotated Code Example

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> a = {1, 3, 5};
    std::vector<int> b = {2, 4, 6};
    std::vector<int> out;

    std::merge(a.begin(), a.end(), b.begin(), b.end(),
               std::back_inserter(out));

    for (int x : out) std::cout << x << ' '; // 1 2 3 4 5 6
}
```

**Expected Output**
```
1 2 3 4 5 6
```

---

### 6.2 `std::includes`

#### Definitions

**Core Definition**
`includes` checks whether one sorted range is a subset of another.

**Complexity**
- O(n + m) comparisons.

#### Annotated Code Example

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> big = {1, 2, 3, 4, 5};
    std::vector<int> sub = {2, 4};

    std::cout << std::boolalpha;
    std::cout << std::includes(big.begin(), big.end(), sub.begin(), sub.end()) << '\n'; // true
}
```

**Expected Output**
```
true
```

---

### 6.3 `std::set_difference`, `std::set_intersection`, `std::set_symmetric_difference`, `std::set_union`

#### Definitions

**Core Definition**
These algorithms compute the difference, intersection, symmetric difference, and union of two sorted ranges.

**Complexity**
- All are O(n + m) comparisons.

#### Annotated Code Example

```cpp
#include <algorithm>
#include <vector>
#include <iostream>
#include <iterator>

int main() {
    std::vector<int> a = {1, 3, 5, 7};
    std::vector<int> b = {2, 3, 5, 8};

    std::vector<int> u, i, d, sd;
    std::set_union(a.begin(), a.end(), b.begin(), b.end(), std::back_inserter(u));
    std::set_intersection(a.begin(), a.end(), b.begin(), b.end(), std::back_inserter(i));
    std::set_difference(a.begin(), a.end(), b.begin(), b.end(), std::back_inserter(d));
    std::set_symmetric_difference(a.begin(), a.end(), b.begin(), b.end(), std::back_inserter(sd));

    std::cout << "union: "; for (int x : u) std::cout << x << ' '; std::cout << '\n';
    std::cout << "intersection: "; for (int x : i) std::cout << x << ' '; std::cout << '\n';
    std::cout << "difference: "; for (int x : d) std::cout << x << ' '; std::cout << '\n';
    std::cout << "symmetric: "; for (int x : sd) std::cout << x << ' '; std::cout << '\n';
}
```

**Expected Output**
```
union: 1 2 3 5 7 8
intersection: 3 5
difference: 1 7
symmetric: 1 2 7 8
```

---

## 7. Heap Operations

### Definitions

**Core Definition**
Heap operations maintain a heap data structure within a random-access range.

**Technical Definition**
A heap is a tree-like structure stored in an array where each parent is greater than or equal to its children (max-heap by default). The standard provides `make_heap`, `push_heap`, `pop_heap`, `sort_heap`, `is_heap`, and `is_heap_until` .

**Beginner-Friendly Explanation**
A heap is a special arrangement of data where the largest element is always at the front. These algorithms let you build and maintain that arrangement.

---

### 7.1 `std::make_heap`, `std::push_heap`, `std::pop_heap`, `std::sort_heap`, `std::is_heap`, `std::is_heap_until`

#### Definitions

**Core Definition**
These algorithms create, maintain, and query a heap within a range.

**Technical Definition**
`make_heap` rearranges a range into a heap in O(n). `push_heap` inserts an element (already placed at the end) into the heap in O(log n). `pop_heap` moves the largest element to the end and restores the heap in O(log n). `sort_heap` sorts a heap into ascending order in O(n log n). `is_heap` checks whether a range is a heap in O(n) .

#### Annotated Code Example

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {3, 1, 4, 1, 5};

    std::make_heap(v.begin(), v.end());    // O(n)
    std::cout << "Max: " << v.front() << '\n'; // 5

    v.push_back(6);
    std::push_heap(v.begin(), v.end());    // O(log n)
    std::cout << "Max after push: " << v.front() << '\n'; // 6

    std::pop_heap(v.begin(), v.end());     // O(log n)
    v.pop_back();
    std::cout << "Max after pop: " << v.front() << '\n'; // 5

    std::sort_heap(v.begin(), v.end());    // O(n log n)
    for (int x : v) std::cout << x << ' '; // 1 1 3 4 5
}
```

**Expected Output**
```
Max: 5
Max after push: 6
Max after pop: 5
1 1 3 4 5
```

---

## 8. Min/Max Operations

### Definitions

**Core Definition**
Min/max algorithms find the smallest or largest of given values or elements in a range.

**Technical Definition**
The standard provides `min`, `max`, `minmax` (for two values or an initializer list), `min_element`, `max_element`, `minmax_element` (for ranges), and `clamp` (to bound a value within a range) .

**Beginner-Friendly Explanation**
These algorithms answer “what is the smallest/largest value?” or “keep this value between these limits.”

---

### 8.1 `std::min`, `std::max`, `std::minmax`, `std::clamp`

#### Definitions

**Core Definition**
`min`/`max` return the smaller/larger of two values. `minmax` returns both. `clamp` bounds a value to a range.

**Complexity**
- O(1) for two-value versions; O(n) for initializer-list versions.

#### Annotated Code Example

```cpp
#include <algorithm>
#include <iostream>

int main() {
    std::cout << std::min(3, 7) << '\n';      // 3
    std::cout << std::max(3, 7) << '\n';      // 7

    auto [lo, hi] = std::minmax({4, 1, 9, 2});
    std::cout << lo << ' ' << hi << '\n';    // 1 9

    std::cout << std::clamp(15, 0, 10) << '\n'; // 10
}
```

**Expected Output**
```
3
7
1 9
10
```

---

### 8.2 `std::min_element`, `std::max_element`, `std::minmax_element`

#### Definitions

**Core Definition**
These algorithms return iterators to the smallest/largest element(s) in a range.

**Complexity**
- `min_element`/`max_element`: exactly `max(last - first - 1, 0)` comparisons.
- `minmax_element`: at most `max(floor(3/2 * (last - first - 1)), 0)` comparisons.

#### Annotated Code Example

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {4, 1, 9, 2, 7};

    auto min_it = std::min_element(v.begin(), v.end());
    auto max_it = std::max_element(v.begin(), v.end());
    std::cout << "min: " << *min_it << ", max: " << *max_it << '\n'; // min: 1, max: 9

    auto [lo, hi] = std::minmax_element(v.begin(), v.end());
    std::cout << "minmax: " << *lo << ' ' << *hi << '\n'; // 1 9
}
```

**Expected Output**
```
min: 1, max: 9
minmax: 1 9
```

---

## 9. Numeric Algorithms

### Definitions

**Core Definition**
Numeric algorithms perform accumulation, reduction, inner product, partial sums, and scan operations on numeric ranges.

**Technical Definition**
Defined in `<numeric>`: `accumulate` (sequential fold), `reduce` (unordered fold, parallelisable), `inner_product`, `adjacent_difference`, `partial_sum`, `exclusive_scan`, `inclusive_scan`, `transform_reduce`, and `transform_scan` variants .

**Beginner-Friendly Explanation**
Numeric algorithms crunch numbers: they sum, multiply, accumulate, and transform sequences of values.

---

### 9.1 `std::accumulate`

#### Definitions

**Core Definition**
`accumulate` computes a sequential fold (by default, a sum) of a range.

**Technical Definition**
`accumulate(first, last, init)` returns the result of successively applying `operator+` to `init` and each element. An overload accepts a binary operation.

**Complexity**
- Exactly `last - first` applications of the operation.

#### Annotated Code Example

```cpp
#include <numeric>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {1, 2, 3, 4};
    int sum = std::accumulate(v.begin(), v.end(), 0);
    std::cout << sum << '\n'; // 10
}
```

**Expected Output**
```
10
```

---

### 9.2 `std::reduce` (C++17)

#### Definitions

**Core Definition**
`reduce` is an unordered, parallelisable version of `accumulate`.

**Technical Definition**
The order of operations is unspecified, allowing parallel execution. The operation must be associative and commutative.

**Complexity**
- O(n) applications.

#### Annotated Code Example

```cpp
#include <numeric>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {1, 2, 3, 4};
    int sum = std::reduce(v.begin(), v.end(), 0);
    std::cout << sum << '\n'; // 10
}
```

**Expected Output**
```
10
```

---

### 9.3 `std::inner_product`

#### Definitions

**Core Definition**
`inner_product` computes the dot product of two ranges (or a generalised version with custom operations).

**Complexity**
- Exactly `last1 - first1` applications of the operations.

#### Annotated Code Example

```cpp
#include <numeric>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> a = {1, 2, 3};
    std::vector<int> b = {4, 5, 6};
    int dot = std::inner_product(a.begin(), a.end(), b.begin(), 0);
    std::cout << dot << '\n'; // 32 (1*4 + 2*5 + 3*6)
}
```

**Expected Output**
```
32
```

---

### 9.4 `std::partial_sum`, `std::adjacent_difference`

#### Definitions

**Core Definition**
`partial_sum` computes running totals; `adjacent_difference` computes differences between adjacent elements.

**Complexity**
- Exactly `last - first - 1` applications.

#### Annotated Code Example

```cpp
#include <numeric>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {1, 2, 3, 4};
    std::vector<int> ps(v.size());
    std::partial_sum(v.begin(), v.end(), ps.begin());
    for (int x : ps) std::cout << x << ' '; // 1 3 6 10
}
```

**Expected Output**
```
1 3 6 10
```

---

### 9.5 `std::exclusive_scan`, `std::inclusive_scan`

#### Definitions

**Core Definition**
Scan algorithms compute prefix sums (or generalised folds). Inclusive scan includes the current element; exclusive scan excludes it.

**Complexity**
- O(n) applications.

#### Annotated Code Example

```cpp
#include <numeric>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {1, 2, 3, 4};
    std::vector<int> inc(v.size()), exc(v.size());

    std::inclusive_scan(v.begin(), v.end(), inc.begin());
    std::exclusive_scan(v.begin(), v.end(), exc.begin(), 0);

    for (int x : inc) std::cout << x << ' '; // 1 3 6 10
    std::cout << '\n';
    for (int x : exc) std::cout << x << ' '; // 0 1 3 6
}
```

**Expected Output**
```
1 3 6 10
0 1 3 6
```

---

### 9.6 `std::transform_reduce`, `std::transform_exclusive_scan`, `std::transform_inclusive_scan`

#### Definitions

**Core Definition**
These combine a transformation with a reduction or scan.

**Complexity**
- O(n) applications.

#### Annotated Code Example

```cpp
#include <numeric>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {1, 2, 3};
    int result = std::transform_reduce(
        v.begin(), v.end(), 0,
        std::plus<>(),
        [](int x){ return x * x; }
    );
    std::cout << result << '\n'; // 14 (1 + 4 + 9)
}
```

**Expected Output**
```
14
```

---

## 10. Algorithm Best Practices

### Definitions

**Core Definition**
Algorithm best practices are guidelines for using standard algorithms effectively, safely, and idiomatically.

**Technical Definition**
These practices are grounded in the C++ Core Guidelines, which recommend preferring standard algorithms over hand-written loops (ES.5, ES.3), using the appropriate algorithm for the task, and understanding complexity and iterator requirements .

**Beginner-Friendly Explanation**
Best practices mean choosing the right algorithm, using it correctly, and letting the library do the work instead of writing error-prone loops.

### Key Guidelines

| Guideline | Summary |
|-----------|---------|
| **Prefer algorithms to hand-written loops** | They are clearer, safer, and often faster. |
| **Choose the right algorithm** | `find` for search, `sort` for ordering, `transform` for mapping. |
| **Know the complexity** | Avoid accidentally quadratic algorithms. |
| **Respect iterator requirements** | `sort` needs random access; `list` has its own `sort`. |
| **Use predicates and lambdas** | Customise behaviour without writing new functions. |
| **Remember `remove` does not erase** | Call `erase` after `remove` to physically remove elements. |
| **Use `back_inserter` for output ranges** | When the destination size is unknown. |
| **Be aware of stability** | Use `stable_sort` when relative order of equal elements matters. |
| **Use execution policies for parallelism** | `std::execution::par` (C++17). |
| **Prefer `const` iterators for read-only algorithms** | `cbegin()`/`cend()`. |

### Annotated Code Example

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {5, 2, 8, 1, 9};

    // Use algorithm instead of manual loop
    auto it = std::find_if(v.cbegin(), v.cend(),
                           [](int x){ return x > 5; });
    if (it != v.cend())
        std::cout << "First > 5: " << *it << '\n'; // 8

    // Sort with a custom comparator
    std::sort(v.begin(), v.end(), std::greater<>());
    for (int x : v) std::cout << x << ' '; // 9 8 5 2 1
}
```

**Expected Output**
```
First > 5: 8
9 8 5 2 1
```

---

## References

- Algorithms library — cppreference – https://en.cppreference.com/w/cpp/algorithm
- Non-modifying sequence operations — cppreference – https://en.cppreference.com/w/cpp/algorithm#Non-modifying_sequence_operations
- Modifying sequence operations — cppreference – https://en.cppreference.com/w/cpp/algorithm#Modifying_sequence_operations
- Sorting operations — cppreference – https://en.cppreference.com/w/cpp/algorithm#Sorting_operations
- Binary search operations — cppreference – https://en.cppreference.com/w/cpp/algorithm#Binary_search_operations
- Set operations — cppreference – https://en.cppreference.com/w/cpp/algorithm#Set_operations
- Heap operations — cppreference – https://en.cppreference.com/w/cpp/algorithm#Heap_operations
- Minimum/maximum operations — cppreference – https://en.cppreference.com/w/cpp/algorithm#Minimum/maximum_operations
- Numeric operations — cppreference – https://en.cppreference.com/w/cpp/header/numeric
- C++ Core Guidelines — ES.5: Keep scopes small – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Res-scope
- C++ Core Guidelines — SL.con: Containers – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-stdlib
- Understand and use `std::sort`, `partial_sort`, `nth_element`, and `stable_sort` — Dr-Sergey/learn_cpp – https://github.com/Dr-Sergey/learn_cpp/blob/575e0775d6aa9df1b80d1343af811215cf9c1745/08_Standard_Library_Algorithms/Understand_and_use_stdsort_partial_sort_nth_element_and_stable_sort.md
- Use binary search algorithms on sorted ranges — Dr-Sergey/learn_cpp – https://github.com/Dr-Sergey/learn_cpp/blob/e2c54b8163b274e4f724168226553ae2ab4d7ddd/08_Standard_Library_Algorithms/Use_binary_search_algorithms_on_sorted_ranges.md
- Know set and heap algorithms — Dr-Sergey/learn_cpp – https://github.com/Dr-Sergey/learn_cpp/blob/575e0775d6aa9df1b80d1343af811215cf9c1745/08_Standard_Library_Algorithms/Know_set_and_heap_algorithms_set_union_merge_push_heap_pop_heap.md
- C++ Working Draft (N4950) — Algorithms library – https://eel.is/c++draft/algorithms

---

This cheat sheet provides a comprehensive, structured reference for the C++ algorithms library, covering all major categories with definitions, purposes, syntax rules, annotated code examples with expected outputs, real-world cases, and authoritative references.