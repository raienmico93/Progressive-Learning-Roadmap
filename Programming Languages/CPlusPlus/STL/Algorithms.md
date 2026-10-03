# Modern Constrained Algorithms & Parallelism — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Modern C++ algorithms encompass two major evolutionary shifts: the C++20 Ranges library, which replaces iterator pairs with composable range wrappers and adds projections and concepts, and the C++17 Parallelism TS, which introduces execution policies that let standard algorithms run across multiple threads and leverage SIMD vectorization. Together, these features transform how developers write searching, sorting, transforming, and numeric operations.

**Technical Definition**
The Ranges library (C++20) provides constrained versions of most standard algorithms in the namespace `std::ranges`. These algorithms accept either an iterator-sentinel pair or a single range argument, support projections (a callable applied to each element before comparison or predicate evaluation), and return structured result types instead of raw iterators. The parallel algorithms library (C++17) adds an `ExecutionPolicy` template parameter to most standard algorithms, allowing them to execute sequentially (`std::execution::seq`), in parallel (`std::execution::par`), or in parallel with vectorization (`std::execution::par_unseq`). The `<numeric>` header has also evolved, adding `std::reduce` (C++17) as a parallel-friendly alternative to `std::accumulate`, and `std::midpoint` (C++20) as an overflow-safe midpoint computation.

**Beginner-Friendly Explanation**
Imagine you have a toolbox full of algorithms (sort, find, copy, etc.). The classic toolbox requires you to say "start here, end there" every single time — like telling a librarian "search between shelf 3 and shelf 7" instead of just "search the shelf." The Ranges library lets you say "search the shelf" and the algorithm figures out the boundaries. The Parallelism library is like having multiple librarians work on the same shelf simultaneously, or even having each librarian use a magnifying glass to process multiple books at once (SIMD). And the numeric additions give you safer, faster ways to do math on whole collections.

---

### Key Characteristics

- **Range-based algorithms** accept containers, arrays, and views directly, eliminating repetitive `.begin()`/`.end()` calls.
- **Projections** allow sorting or searching by a member, a transformation, or any callable without writing a custom comparator lambda.
- **Concepts** provide clearer compile errors (e.g., “does not satisfy sortable”) and prevent misuse of algorithms on incompatible ranges.
- **Structured return types** (e.g., `in_out_result`) replace raw iterator returns, making multi-output algorithms easier to use.
- **Execution policies** enable multi-threading (`par`), SIMD vectorization (`par_unseq`), and pure sequential execution (`seq`).
- **Parallel numeric algorithms** like `std::reduce` permit arbitrary reordering and reassociation, unlocking parallel speedups.
- **`std::midpoint`** avoids integer overflow when computing midpoints, a common source of bugs in binary search implementations.

---

### Prerequisites

- Basic C++ syntax: templates, lambdas, iterators, and containers.
- Familiarity with classic `<algorithm>` and `<numeric>` functions (e.g., `std::sort`, `std::find`, `std::accumulate`).
- Understanding of iterators and their categories.
- (For parallel algorithms) A compiler supporting C++17 parallelism (e.g., GCC with TBB, MSVC with `/std:c++17` and the Windows Thread Pool).
- (For ranges) A compiler supporting C++20 or later.

---

### Related Programming Areas

- **Generic programming**: Concepts and ranges make generic code safer and more expressive.
- **High-performance computing**: Execution policies bring standard-library algorithms to multi-core and SIMD hardware.
- **Data processing pipelines**: Range adaptors (views) enable lazy, composable transformations.
- **Numerical computing**: `std::reduce`, `std::transform_reduce`, and `std::midpoint` improve correctness and parallelism for numeric workloads.

---

### Core Concepts / Features

1. **Classic vs. Ranges Algorithms** — Moving from iterator pairs to cleaner range wrappers.
2. **Execution Policies & Parallelism** — Accelerating algorithms via multi-threading and SIMD vectorization.
3. **Functional Sub-categories** — Searching/sorting, mutating/copying, partitioning, heap actions, and modern numeric operations.

---

## 1. Classic vs. Ranges Algorithms

### Definitions

**Core Definition**
Classic algorithms take an iterator pair (`first`, `last`) that defines a range. Range algorithms (C++20) take a single range object (a container, view, or any type with `begin()`/`end()`) and optionally a projection and comparator, eliminating boilerplate and enabling composability.

**Technical Definition**
The Ranges library, defined in `<ranges>`, provides constrained algorithm function objects (niebloids) in the namespace `std::ranges`. These algorithms accept either `(I first, S last)` or `R&& r` where `R` models a range. They support a projection parameter (`Proj proj = std::identity{}`), which is applied to each element before the comparator or predicate. Return types are often structured (e.g., `std::ranges::in_out_result`) rather than raw iterators. Projections and concepts are first-class features, and algorithms are constrained via concepts like `std::sortable` and `std::random_access_range`. Classic algorithms in namespace `std` remain available and are not deprecated.

**Beginner-Friendly Explanation**
Classic C++ algorithms require you to always pass two iterators: `std::sort(v.begin(), v.end())`. Range algorithms let you just say `std::ranges::sort(v)`. More importantly, range algorithms let you sort by a member without writing a lambda: `std::ranges::sort(people, {}, &Person::age)`. They also give clearer error messages and protect against dangling references.

---

### Sub-Feature: Range-Based Algorithm Syntax

#### Definitions

**Core Definition**
Range-based syntax is the call pattern where a range is passed as a single argument, and projections/comparators are optional trailing arguments.

**Technical Definition**
The general form is `std::ranges::algorithm_name(range, comparator, projection)`. The comparator defaults to `std::ranges::less{}` and the projection defaults to `std::identity{}`. For algorithms that write output, the range is passed first, then output iterators. Range algorithms are niebloids — function objects that inhibit ADL and prevent explicit template argument specification. They are constrained by concepts and return structured types when multiple outputs are produced.

**Beginner-Friendly Explanation**
Instead of `std::sort(v.begin(), v.end())`, you write `std::ranges::sort(v)`. If you want to sort by a field, you write `std::ranges::sort(v, {}, &T::field)`. The `{}` is the default comparator, and `&T::field` is the projection that extracts the field to compare.

#### Purposes (all begin with "To")

- **To** reduce boilerplate by eliminating repetitive `.begin()`/`.end()` pairs.
- **To** prevent mismatched iterator errors (e.g., passing `begin()` from one container and `end()` from another).
- **To** enable sorting/searching by a member or transformation without writing a lambda.
- **To** provide clearer compile-time errors through concept constraints.
- **To** protect against dangling references with `std::ranges::dangling`.

#### Syntax Rules and Structure

```cpp
// Classic (C++98–C++17)
std::sort(v.begin(), v.end());
std::find(v.begin(), v.end(), 42);
std::count_if(v.begin(), v.end(), pred);

// Ranges (C++20)
std::ranges::sort(v);
std::ranges::find(v, 42);
std::ranges::count_if(v, pred);

// Ranges with projection
std::ranges::sort(people, std::less{}, &Person::age);
```

**Component breakdown:**
- `std::ranges::sort` — the constrained algorithm function object.
- `v` — the range (container, view, array, etc.).
- `std::less{}` — the comparator (default is `std::ranges::less{}`).
- `&Person::age` — the projection (applied to each element before comparison).

**Constraints and limitations:**
- Requires C++20 or later.
- Not all classic algorithms have range equivalents (e.g., `std::accumulate` has no `std::ranges` counterpart; use `std::reduce` or `std::fold_left` in C++23).
- Range algorithms are niebloids: explicit template arguments cannot be specified, and ADL is inhibited.
- Passing a temporary where the result would dangle returns `std::ranges::dangling` rather than a silent bug.

#### Annotated Code Example

```cpp
// File: ranges_vs_classic.cpp
// Compile with: g++ -std=c++20 ranges_vs_classic.cpp -o ranges_vs_classic

#include <algorithm>
#include <iostream>
#include <ranges>
#include <string>
#include <vector>

struct Person {
    std::string name;
    int age;
};

int main() {
    // Step 1: Classic iterator-pair algorithms
    std::vector<int> v1 = {5, 2, 8, 1, 9};
    std::sort(v1.begin(), v1.end());
    std::cout << "Classic sort: ";
    for (int x : v1) std::cout << x << " ";
    std::cout << std::endl;

    // Step 2: Range-based algorithms — no iterators
    std::vector<int> v2 = {5, 2, 8, 1, 9};
    std::ranges::sort(v2);
    std::cout << "Ranges sort: ";
    for (int x : v2) std::cout << x << " ";
    std::cout << std::endl;

    // Step 3: Projection — sort by member without a lambda
    std::vector<Person> people = {
        {"Alice", 30}, {"Bob", 25}, {"Charlie", 35}
    };
    std::ranges::sort(people, {}, &Person::age);
    std::cout << "Sorted by age:" << std::endl;
    for (const auto& p : people) {
        std::cout << "  " << p.name << " (" << p.age << ")" << std::endl;
    }

    // Step 4: Range algorithms work with subranges
    std::vector<int> v3 = {9, 7, 5, 3, 1, 8, 6, 4, 2};
    std::ranges::sort(std::ranges::subrange(v3.begin() + 2, v3.begin() + 7));
    std::cout << "Partial sort: ";
    for (int x : v3) std::cout << x << " ";
    std::cout << std::endl;

    return 0;
}
```

**Expected Output:**
```
Classic sort: 1 2 5 8 9
Ranges sort: 1 2 5 8 9
Sorted by age:
  Bob (25)
  Alice (30)
  Charlie (35)
Partial sort: 9 7 1 3 5 8 6 4 2
```

**Why this output:** `std::ranges::sort(v2)` deduces the range boundaries from the container. The projection `&Person::age` extracts the age field for comparison, so people are sorted by age. `std::ranges::subrange` creates a view over a portion of the vector, and sorting that subrange reorders only elements at indices 2–6.

#### Real-World Cases with Explanation

**Case 1 — Sorting records by a field**: Instead of writing `std::sort(v.begin(), v.end(), [](const auto& a, const auto& b) { return a.age < b.age; })`, you write `std::ranges::sort(v, {}, &Person::age)`.

**Case 2 — Finding in a container**: `std::ranges::find(container, value)` replaces `std::find(container.begin(), container.end(), value)`, eliminating a common source of bugs.

**Case 3 — Pipeline composition**: Range algorithms can be combined with range adaptors (views) for lazy processing: `auto result = data | std::views::filter(pred) | std::views::transform(fn);`.

---

## 2. Execution Policies & Parallelism

### Definitions

**Core Definition**
Execution policies are tags passed as the first argument to standard algorithms to specify whether they should run sequentially, in parallel, or in parallel with SIMD vectorization.

**Technical Definition**
C++17 introduces three execution policies in `<execution>`: `std::execution::seq` (sequential), `std::execution::par` (parallel), and `std::execution::par_unseq` (parallel + unsequenced/vectorized). C++20 adds `std::execution::unseq` (unsequenced, single-threaded vectorization). Algorithms overloaded with an `ExecutionPolicy&&` parameter execute according to the policy’s guarantees. `par` permits multiple threads; `par_unseq` additionally permits vectorization and migration across threads. The programmer is responsible for ensuring that element access functions are thread-safe and free of data races when using `par` or `par_unseq`.

**Beginner-Friendly Explanation**
Normally, `std::sort` uses one CPU core. By adding `std::execution::par`, you tell the library it may use multiple cores. Adding `par_unseq` also allows the compiler to use SIMD instructions — processing multiple data elements with one instruction. This can make algorithms dramatically faster on large datasets, but you must ensure your comparison functions and predicates are safe to run concurrently.

---

### Sub-Feature: Execution Policies

#### Definitions

**Core Definition**
An execution policy is a tag type that conveys to an algorithm the level of parallelism and vectorization permitted during execution.

**Technical Definition**
The execution policies are defined in namespace `std::execution` in header `<execution>`:

- `sequenced_policy` (`std::execution::seq`): Execution may not be parallelized. Element access functions are indeterminately sequenced in the calling thread.
- `parallel_policy` (`std::execution::par`): Execution may be parallelized. Element access functions may run in the invoking thread or in threads implicitly created by the library. Any two invocations in the same thread are indeterminately sequenced.
- `parallel_unsequenced_policy` (`std::execution::par_unseq`): Execution may be parallelized, vectorized, or migrated across threads. Element access functions are unsequenced with respect to one another within each thread and unordered across threads. Vectorization-unsafe operations (e.g., `std::atomic` synchronisation) are not permitted.
- `unsequenced_policy` (`std::execution::unseq`, C++20): Execution may be vectorized on a single thread but not parallelized.

**Beginner-Friendly Explanation**
- `seq`: “Run this on one core, one operation at a time.”
- `par`: “You may use multiple cores, but each core does one operation at a time.”
- `par_unseq`: “Use multiple cores and process multiple data elements per instruction (SIMD).”
- `unseq`: “Use SIMD on a single core.”

#### Purposes (all begin with "To")

- **To** accelerate standard algorithms on multi-core hardware without rewriting them.
- **To** enable SIMD vectorization for data-parallel workloads.
- **To** let the library choose the optimal parallel decomposition based on the algorithm and data size.
- **To** provide a portable, standard interface for parallel execution across compilers and platforms.
- **To** allow the same algorithm code to run sequentially or in parallel by changing one argument.

#### Syntax Rules and Structure

```cpp
#include <execution>
#include <algorithm>
#include <vector>

std::vector<int> v = /* ... */;

// Sequential (default, same as no policy)
std::sort(std::execution::seq, v.begin(), v.end());

// Parallel (multi-threaded)
std::sort(std::execution::par, v.begin(), v.end());

// Parallel + vectorized (SIMD)
std::sort(std::execution::par_unseq, v.begin(), v.end());
```

**Component breakdown:**
- `std::execution::par` — the execution policy tag.
- `v.begin(), v.end()` — the range to process (or use ranges with policies in C++26).

**Constraints and limitations:**
- Parallel algorithms require random-access iterators; other iterator types fall back to serial execution.
- The programmer must avoid data races: all element access functions must be thread-safe when using `par` or `par_unseq`.
- Exceptions thrown in parallel element access functions cause `std::terminate` to be called.
- `par_unseq` forbids vectorization-unsafe operations like locks and atomics.
- Compiler/library support varies: GCC requires Intel TBB; MSVC uses the Windows Thread Pool.
- Not all algorithms support parallel execution efficiently (e.g., `includes`, `inplace_merge`, `merge`, `nth_element`).

#### Annotated Code Example

```cpp
// File: execution_policies.cpp
// Compile with: g++ -std=c++17 -ltbb execution_policies.cpp -o execution_policies
// Or with MSVC: cl /std:c++17 /EHsc execution_policies.cpp

#include <algorithm>
#include <chrono>
#include <execution>
#include <iostream>
#include <numeric>
#include <vector>

int main() {
    constexpr int N = 10'000'000;
    std::vector<double> data(N);
    std::iota(data.begin(), data.end(), 1.0);

    // Step 1: Sequential transform
    auto start = std::chrono::high_resolution_clock::now();
    std::transform(std::execution::seq,
                   data.begin(), data.end(),
                   data.begin(),
                   [](double x) { return std::sqrt(x); });
    auto end = std::chrono::high_resolution_clock::now();
    std::cout << "seq time: "
              << std::chrono::duration<double>(end - start).count()
              << "s" << std::endl;

    // Step 2: Parallel transform
    start = std::chrono::high_resolution_clock::now();
    std::transform(std::execution::par,
                   data.begin(), data.end(),
                   data.begin(),
                   [](double x) { return std::sqrt(x); });
    end = std::chrono::high_resolution_clock::now();
    std::cout << "par time: "
              << std::chrono::duration<double>(end - start).count()
              << "s" << std::endl;

    // Step 3: Parallel + unsequenced (SIMD) transform
    start = std::chrono::high_resolution_clock::now();
    std::transform(std::execution::par_unseq,
                   data.begin(), data.end(),
                   data.begin(),
                   [](double x) { return std::sqrt(x); });
    end = std::chrono::high_resolution_clock::now();
    std::cout << "par_unseq time: "
              << std::chrono::duration<double>(end - start).count()
              << "s" << std::endl;

    // Step 4: Parallel reduce (sum)
    double sum = std::reduce(std::execution::par,
                             data.begin(), data.end(),
                             0.0);
    std::cout << "Parallel sum: " << sum << std::endl;

    return 0;
}
```

**Expected Output (timings vary):**
```
seq time: 0.045s
par time: 0.012s
par_unseq time: 0.008s
Parallel sum: 6.66667e+10
```

**Why this output:** The sequential transform processes 10 million elements one at a time. The parallel version splits the work across threads, reducing wall-clock time. `par_unseq` additionally uses SIMD instructions, processing multiple elements per instruction. The parallel `std::reduce` computes the sum by splitting the range into chunks, summing each chunk in parallel, then combining the partial sums.

#### Real-World Cases with Explanation

**Case 1 — Image processing**: Applying a filter to a large pixel buffer using `std::transform` with `par_unseq` for SIMD-accelerated per-pixel operations.

**Case 2 — Financial Monte Carlo simulations**: Using `std::reduce` with `par` to compute portfolio risk metrics across millions of scenarios.

**Case 3 — Data analytics**: Sorting large datasets with `std::sort(std::execution::par, ...)` to reduce query latency.

**Case 4 — Scientific computing**: Computing element-wise operations on large arrays with `std::for_each(par_unseq, ...)`.

---

## 3. Functional Sub-categories

### Definitions

**Core Definition**
Functional sub-categories group standard algorithms by the kind of operation they perform: searching/sorting, mutating/copying, partitioning, heap operations, and numeric operations.

**Technical Definition**
The C++ standard classifies algorithms into several sub-clauses: non-modifying sequence operations (searching, counting, comparing), mutating sequence operations (copying, transforming, removing, replacing), partitioning operations, sorting operations, binary search operations, set operations, heap operations, and numeric operations (`<numeric>`). The Ranges library provides constrained versions in `std::ranges` with the same categorisation. Modern numeric operations include `std::reduce` (C++17), `std::transform_reduce` (C++17), and `std::midpoint` (C++20).

**Beginner-Friendly Explanation**
Think of algorithms as tools in a workshop. Some are for finding things (searching), some for rearranging (sorting), some for copying data (mutating), some for splitting data into groups (partitioning), some for building special tree-like structures (heaps), and some for doing math (numeric).

---

### Sub-Feature: Searching & Sorting Algorithms

#### Definitions

**Core Definition**
Searching algorithms locate elements or subranges matching a value or predicate. Sorting algorithms arrange elements in a specified order.

**Technical Definition**
Searching algorithms include `std::ranges::find`, `std::ranges::find_if`, `std::ranges::binary_search`, `std::ranges::lower_bound`, and `std::ranges::upper_bound`. Sorting algorithms include `std::ranges::sort`, `std::ranges::stable_sort`, `std::ranges::partial_sort`, and `std::ranges::nth_element`. Binary search algorithms require the range to be sorted.

**Beginner-Friendly Explanation**
Searching is like looking for a specific book in a library. Sorting is like arranging the books in alphabetical order so you can find them faster next time.

#### Purposes

- **To** locate elements efficiently using linear or binary search.
- **To** arrange elements in ascending or custom order.
- **To** support binary search operations on sorted data.
- **To** provide stable sorting that preserves the relative order of equivalent elements.

#### Syntax Rules and Structure

```cpp
// Searching
std::ranges::find(range, value);
std::ranges::find_if(range, predicate);
std::ranges::binary_search(range, value);

// Sorting
std::ranges::sort(range);
std::ranges::stable_sort(range, comp, proj);
std::ranges::nth_element(range, nth);
```

#### Annotated Code Example

```cpp
// File: searching_sorting.cpp
// Compile with: g++ -std=c++20 searching_sorting.cpp -o searching_sorting

#include <algorithm>
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> v = {50, 20, 80, 10, 40, 70, 30};

    // Step 1: Sort the range
    std::ranges::sort(v);
    std::cout << "Sorted: ";
    for (int x : v) std::cout << x << " ";
    std::cout << std::endl;

    // Step 2: Binary search
    bool found = std::ranges::binary_search(v, 40);
    std::cout << "40 found: " << found << std::endl;

    // Step 3: lower_bound / upper_bound
    auto lb = std::ranges::lower_bound(v, 40);
    std::cout << "lower_bound(40): " << *lb << std::endl;

    // Step 4: find_if with projection
    struct Item { int id; };
    std::vector<Item> items = {{1}, {2}, {3}, {4}};
    auto it = std::ranges::find_if(items, [](int x) { return x > 2; }, &Item::id);
    if (it != items.end()) {
        std::cout << "First id > 2: " << it->id << std::endl;
    }

    return 0;
}
```

**Expected Output:**
```
Sorted: 10 20 30 40 50 70 80
40 found: 1
lower_bound(40): 40
First id > 2: 3
```

#### Real-World Cases

- **Database indexing**: Binary search on sorted indexes for O(log n) lookups.
- **Leaderboards**: Sorting player scores with `std::ranges::sort` and projection.
- **Log analysis**: Finding entries matching a timestamp with `std::ranges::find_if`.

---

### Sub-Feature: Mutating & Copying Algorithms

#### Definitions

**Core Definition**
Mutating algorithms modify the elements of a range (e.g., transform, replace, remove). Copying algorithms copy elements from a source range to a destination.

**Technical Definition**
Mutating operations include `std::ranges::transform`, `std::ranges::replace`, `std::ranges::remove`, `std::ranges::fill`, and `std::ranges::generate`. Copying operations include `std::ranges::copy`, `std::ranges::copy_if`, `std::ranges::move`, and `std::ranges::reverse_copy`. Many return structured results (e.g., `in_out_result`) indicating the end of the input and output ranges.

**Beginner-Friendly Explanation**
Mutating algorithms change the data in place. Copying algorithms duplicate data to a new location.

#### Purposes

- **To** apply a function to every element in a range.
- **To** copy elements conditionally to a destination.
- **To** replace or remove elements matching a criterion.
- **To** fill a range with a generated sequence.

#### Annotated Code Example

```cpp
// File: mutating_copying.cpp
// Compile with: g++ -std=c++20 mutating_copying.cpp -o mutating_copying

#include <algorithm>
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> v = {1, 2, 3, 4, 5};

    // Step 1: Transform in place
    std::ranges::transform(v, v.begin(), [](int x) { return x * x; });
    std::cout << "Squared: ";
    for (int x : v) std::cout << x << " ";
    std::cout << std::endl;

    // Step 2: Copy_if to a new vector
    std::vector<int> evens;
    std::ranges::copy_if(v, std::back_inserter(evens),
                         [](int x) { return x % 2 == 0; });
    std::cout << "Evens: ";
    for (int x : evens) std::cout << x << " ";
    std::cout << std::endl;

    // Step 3: Remove_if (in-place removal)
    std::vector<int> w = {1, 2, 3, 4, 5, 6};
    auto [in, out] = std::ranges::remove_if(w, [](int x) { return x % 2 == 0; });
    w.erase(out, w.end());
    std::cout << "After remove_if: ";
    for (int x : w) std::cout << x << " ";
    std::cout << std::endl;

    return 0;
}
```

**Expected Output:**
```
Squared: 1 4 9 16 25
Evens: 4 16
After remove_if: 1 3 5
```

#### Real-World Cases

- **Data cleaning**: `std::ranges::remove_if` to filter out invalid records.
- **Image processing**: `std::ranges::transform` to apply a colour filter to every pixel.
- **Bulk initialisation**: `std::ranges::fill` to set all elements to a default value.

---

### Sub-Feature: Partitioning Algorithms

#### Definitions

**Core Definition**
Partitioning algorithms rearrange elements so that those satisfying a predicate come before those that do not.

**Technical Definition**
`std::ranges::partition` reorders elements based on a predicate and returns a subrange from the partition point to the end. `std::ranges::stable_partition` preserves the relative order of elements within each group. `std::ranges::is_partitioned` checks whether a range is partitioned. `std::ranges::partition_point` finds the partition point in a partitioned range.

**Beginner-Friendly Explanation**
Partitioning is like separating a deck of cards into red and black piles. The partition algorithm does this in one pass, without fully sorting.

#### Purposes

- **To** separate elements into two groups based on a predicate.
- **To** support quicksort’s partitioning step.
- **To** enable efficient “find all elements satisfying X” operations.
- **To** maintain stability (relative order) when required.

#### Annotated Code Example

```cpp
// File: partitioning.cpp
// Compile with: g++ -std=c++20 partitioning.cpp -o partitioning

#include <algorithm>
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> v = {1, 2, 3, 4, 5, 6, 7, 8};

    // Step 1: Partition into even and odd
    auto result = std::ranges::partition(v, [](int x) { return x % 2 == 0; });

    std::cout << "Partitioned: ";
    for (int x : v) std::cout << x << " ";
    std::cout << std::endl;

    // Step 2: The subrange [result.begin(), result.end()) contains odds
    std::cout << "Odds: ";
    for (auto it = result.begin(); it != result.end(); ++it) {
        std::cout << *it << " ";
    }
    std::cout << std::endl;

    // Step 3: Check if partitioned
    bool is_part = std::ranges::is_partitioned(v, [](int x) { return x % 2 == 0; });
    std::cout << "Is partitioned: " << is_part << std::endl;

    return 0;
}
```

**Expected Output:**
```
Partitioned: 8 2 6 4 5 3 7 1
Odds: 5 3 7 1
Is partitioned: 1
```

**Why this output:** `std::ranges::partition` moves all even numbers to the front and odd numbers to the back. The returned subrange points to the first odd element through the end. `is_partitioned` confirms the property holds.

#### Real-World Cases

- **Quicksort**: Partitioning is the core step of quicksort.
- **Load balancing**: Separating “heavy” and “light” tasks before scheduling.
- **Data filtering**: Quickly grouping records that meet a condition.

---

### Sub-Feature: Heap Operations

#### Definitions

**Core Definition**
Heap operations construct and manipulate a binary heap — a tree-like structure stored in a contiguous range where every parent satisfies a comparison relationship with its children.

**Technical Definition**
`std::ranges::make_heap` arranges a range into a heap in O(n) time. `std::ranges::push_heap` and `std::ranges::pop_heap` maintain the heap property after insertion or removal. `std::ranges::sort_heap` sorts a heap in O(n log n). `std::ranges::is_heap` checks the heap property.

**Beginner-Friendly Explanation**
A heap is like a tournament bracket stored in an array. `make_heap` builds the bracket, `push_heap` adds a new contestant, `pop_heap` removes the winner, and `sort_heap` turns the bracket into a fully sorted list.

#### Purposes

- **To** implement priority queues efficiently.
- **To** sort data using heap sort.
- **To** support algorithms like Dijkstra’s and Huffman’s that need a priority queue.
- **To** maintain a dynamic set with fast access to the maximum element.

#### Annotated Code Example

```cpp
// File: heap_ops.cpp
// Compile with: g++ -std=c++20 heap_ops.cpp -o heap_ops

#include <algorithm>
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> v = {3, 1, 4, 1, 5, 9, 2, 6};

    // Step 1: Make a heap (max-heap by default)
    std::ranges::make_heap(v);
    std::cout << "Heap: ";
    for (int x : v) std::cout << x << " ";
    std::cout << std::endl;

    // Step 2: Push a new element
    v.push_back(10);
    std::ranges::push_heap(v);
    std::cout << "After push(10): ";
    for (int x : v) std::cout << x << " ";
    std::cout << std::endl;

    // Step 3: Pop the max
    std::ranges::pop_heap(v);
    int max = v.back();
    v.pop_back();
    std::cout << "Popped max: " << max << std::endl;

    // Step 4: Sort the heap
    std::ranges::sort_heap(v);
    std::cout << "Sorted heap: ";
    for (int x : v) std::cout << x << " ";
    std::cout << std::endl;

    return 0;
}
```

**Expected Output:**
```
Heap: 9 6 4 1 5 3 2 1
After push(10): 10 9 4 6 5 3 2 1 1
Popped max: 10
Sorted heap: 1 1 2 3 4 5 6 9
```

#### Real-World Cases

- **Priority queues**: `std::priority_queue` is built on heap operations.
- **Heap sort**: `std::ranges::sort_heap` provides an in-place O(n log n) sort.
- **Event simulation**: Using a heap to process events in priority order.

---

### Sub-Feature: Modern Numeric Operations (`std::reduce`, `std::midpoint`)

#### Definitions

**Core Definition**
`std::reduce` (C++17) is a parallel-friendly numeric algorithm that combines elements using an associative and commutative binary operation. `std::midpoint` (C++20) computes the midpoint between two integers, floating-point numbers, or pointers without overflow.

**Technical Definition**
`std::reduce` is defined in `<numeric>`. It behaves like `std::accumulate` but permits arbitrary reordering and reassociation of elements, enabling parallel execution via execution policies. The overloads include `reduce(first, last)`, `reduce(first, last, init)`, `reduce(first, last, init, binary_op)`, and execution-policy variants. `std::midpoint(a, b)` returns half the sum of `a` and `b`, rounded towards `a` if the sum is odd. It works for arithmetic types and pointers, and never overflows.

**Beginner-Friendly Explanation**
`std::reduce` is like `std::accumulate`, but it doesn’t care about the order in which elements are combined. This makes it suitable for parallel execution. `std::midpoint` solves a subtle bug: computing `(a + b) / 2` can overflow for large integers. `std::midpoint` avoids that.

#### Purposes

- **To** perform parallel reductions over large ranges.
- **To** compute sums, products, or custom reductions with associative operations.
- **To** safely compute the midpoint of two values without overflow.
- **To** support numeric algorithms in parallel execution policies.

#### Syntax Rules and Structure

```cpp
// std::reduce
template<class InputIt>
typename iterator_traits<InputIt>::value_type
reduce(InputIt first, InputIt last);

template<class InputIt, class T>
T reduce(InputIt first, InputIt last, T init);

template<class InputIt, class T, class BinaryOperation>
T reduce(InputIt first, InputIt last, T init, BinaryOperation binary_op);

// With execution policy
template<class ExecutionPolicy, class ForwardIt>
typename iterator_traits<ForwardIt>::value_type
reduce(ExecutionPolicy&& exec, ForwardIt first, ForwardIt last);

// std::midpoint
template<class T>
constexpr T midpoint(T a, T b) noexcept;

template<class T>
constexpr T* midpoint(T* a, T* b);
```

**Constraints and limitations:**
- `std::reduce` requires the binary operation to be associative and commutative; otherwise, the result is non-deterministic.
- `std::reduce` may reorder operands and reassociate arbitrarily.
- `std::midpoint` for integers rounds towards `a` when the sum is odd.
- `std::midpoint` for pointers requires both pointers to point to elements of the same array (or one past the end).

#### Annotated Code Example

```cpp
// File: numeric_modern.cpp
// Compile with: g++ -std=c++20 numeric_modern.cpp -o numeric_modern

#include <iostream>
#include <numeric>
#include <vector>

int main() {
    // Step 1: std::reduce — sequential
    std::vector<int> v = {1, 2, 3, 4, 5};
    int sum = std::reduce(v.begin(), v.end(), 0);
    std::cout << "Sequential sum: " << sum << std::endl;

    // Step 2: std::reduce with multiplication
    int product = std::reduce(v.begin(), v.end(), 1, std::multiplies<int>());
    std::cout << "Product: " << product << std::endl;

    // Step 3: std::midpoint — integer
    int a = 2'000'000'000;
    int b = 2'000'000'002;
    // (a + b) / 2 would overflow on 32-bit int
    int mid = std::midpoint(a, b);
    std::cout << "Midpoint: " << mid << std::endl;

    // Step 4: std::midpoint — pointer
    int arr[10] = {};
    int* p1 = &arr[2];
    int* p2 = &arr[8];
    int* pm = std::midpoint(p1, p2);
    std::cout << "Pointer midpoint index: " << (pm - arr) << std::endl;

    // Step 5: std::midpoint — floating point
    double x = 1.0, y = 2.0;
    std::cout << "Float midpoint: " << std::midpoint(x, y) << std::endl;

    return 0;
}
```

**Expected Output:**
```
Sequential sum: 15
Product: 120
Midpoint: 2000000001
Pointer midpoint index: 5
Float midpoint: 1.5
```

**Why this output:** `std::reduce` sums the vector elements. `std::midpoint(2'000'000'000, 2'000'000'002)` returns 2'000'000'001 without overflow, whereas `(a + b) / 2` would overflow 32-bit `int`. For pointers, `std::midpoint(p1, p2)` returns a pointer to the element halfway between them.

#### Real-World Cases with Explanation

**Case 1 — Parallel summation**: Computing the sum of a large array using `std::reduce(std::execution::par, ...)` to split the work across threads.

**Case 2 — Binary search**: Using `std::midpoint(lo, hi)` to compute the midpoint safely in binary search implementations.

**Case 3 — Image averaging**: Using `std::reduce` with a custom binary operation to average pixel values in parallel.

**Case 4 — Financial calculations**: Using `std::transform_reduce` (C++17) to compute weighted sums or dot products in parallel.

---

## Summary Table: Classic vs. Ranges vs. Parallel

| Feature | Classic (C++98–C++17) | Ranges (C++20) | Parallel (C++17) |
|---|---|---|---|
| Call syntax | `std::sort(v.begin(), v.end())` | `std::ranges::sort(v)` | `std::sort(std::execution::par, v.begin(), v.end())` |
| Projections | Manual lambda | `std::ranges::sort(v, {}, &T::member)` | Not available in classic API |
| Return types | Iterator | Structured (`in_out_result`, etc.) | Iterator |
| Concept constraints | None | `std::sortable`, `std::random_access_range`, etc. | None |
| Dangling protection | None | `std::ranges::dangling` | None |
| Execution policies | Not supported | C++26 adds range+policy overloads | `seq`, `par`, `par_unseq`, `unseq` |
| Parallel numeric | `std::accumulate` (sequential only) | N/A | `std::reduce`, `std::transform_reduce` |

---

## References

- Constrained algorithms (since C++20) — cppreference.com - https://en.cppreference.com/w/cpp/algorithm/ranges
- std::ranges::sort — cppreference.com - https://en.cppreference.com/w/cpp/algorithm/ranges/sort
- Ranges library (C++20) — cppreference.com - https://en.cppreference.com/w/cpp/ranges
- std::execution::sequenced_policy, std::execution::parallel_policy, std::execution::parallel_unsequenced_policy — cppreference.com - https://en.cppreference.com/w/cpp/algorithm/execution_policy_tag_t
- Execution policy — cppreference.com - https://en.cppreference.com/w/cpp/algorithm/execution_policy_tag
- std::reduce — cppreference.com - https://en.cppreference.com/w/cpp/algorithm/reduce
- std::midpoint — cppreference.com - https://en.cppreference.com/w/cpp/numeric/midpoint
- Standard library header <numeric> — cppreference.com - https://en.cppreference.com/w/cpp/header/numeric
- <execution> — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/standard-library/execution
- Parallel algorithms — cppreference.com - https://en.cppreference.com/w/cpp/algorithm#Parallel_algorithms
- Use range algorithms instead of iterator-pair algorithms — Dr-Sergey/learn_cpp (GitHub) - https://github.com/Dr-Sergey/learn_cpp/blob/main/14_Ranges_Cpp20/Use_range_algorithms_instead_of_iterator-pair_algorithms.md
- Use parallel execution policies (C++17) for algorithm parallelism — Dr-Sergey/learn_cpp (GitHub) - https://github.com/Dr-Sergey/learn_cpp/blob/main/08_Standard_Library_Algorithms/Use_parallel_execution_policies_C++17_for_algorithm_parallelism.md
- P0350R1: Integrating SIMD with Parallel Algorithms — Open-Std - https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/p0350r1.pdf
- N4861: Working Draft, Standard for Programming Language C++ (C++20) — Open-Std - https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2020/n4861.pdf
- NVIDIA HPC Compilers User's Guide: Parallel Algorithms — NVIDIA - https://docs.nvidia.com/hpc-sdk/compilers/hpc-compilers-user-guide/index.html