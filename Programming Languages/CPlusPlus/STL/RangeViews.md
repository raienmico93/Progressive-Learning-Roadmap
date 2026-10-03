# The Ranges & Views Revolution (C++20 / C++23) — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
The Ranges library (C++20) and its C++23 extensions provide a modern, composable framework for processing sequences of data. The core innovation is the separation of *containers* (owning, eager data structures) from *views* (non-owning, lazy, lightweight descriptors of computation). Views are composed using the pipe operator (`|`) to form declarative, readable data-processing pipelines that execute on demand.

**Technical Definition**
The Ranges library (header `<ranges>`, namespace `std::ranges`) introduces three pillars: **ranges** (types with `begin()` and `end()`), **views** (ranges that are cheap to copy/move and do not own their elements), and **range adaptors** (customisation point objects that transform a view into another view). Range adaptors are *niebloids* — function objects that inhibit ADL and can be partially applied to form *range adaptor closures*. The pipe operator (`operator|`) is overloaded to compose adaptor closures with ranges, enabling left-to-right, lazy pipelines. Views satisfy the `std::ranges::view` concept, requiring O(1) copy/move/assignment operations. Since P2415 (applied to C++20 as a defect report), views may also own elements (e.g., `views::owning_view`), so `view` no longer strictly means "non-owning." C++23 expanded the adaptor set with `views::zip`, `views::chunk`, `views::slide`, `views::stride`, `views::chunk_by`, `views::enumerate`, and `views::adjacent`.

**Beginner-Friendly Explanation**
Imagine you have a box of Lego bricks (a container). You want to build a model. The traditional way is to take every brick out, examine it, and build a new pile. The Ranges way is to describe what you want — "take only the red bricks, make them twice as big, take the first 10" — and only when you finally ask for the result does the library look at each brick. Views are like a description of work, not the work itself. The pipe operator (`|`) lets you chain these descriptions together like a recipe: "bricks | filter(red) | transform(double) | take(10)."

---

### Key Characteristics

- **Lazy evaluation**: Views compute elements only when iterated; operations are not performed until the result is consumed.
- **Non-owning (mostly)**: Views do not own their underlying data; they reference existing containers or other views. `views::owning_view` and `ranges::to` provide escape hatches for owning views.
- **O(1) copy and move**: Views are required to be cheap to copy, move, and assign (O(1) complexity).
- **Composability**: Range adaptors compose via the pipe operator (`|`) to form pipelines that read left-to-right.
- **Infinite range support**: Lazy views can safely operate on infinite sources like `views::iota(1)` when bounded by `views::take`.
- **C++23 expansion**: New adaptors (`zip`, `chunk`, `slide`, `stride`, `enumerate`) cover multi-range, windowing, and indexed iteration patterns.

---

### Prerequisites

- Basic C++ syntax: containers, iterators, lambdas, and templates.
- Familiarity with the STL algorithms and sequence containers.
- Understanding of iterators and their categories.
- (For C++20) A compiler supporting C++20 or later.
- (For C++23 views) A compiler supporting C++23 or later.
- (For `ranges::to`) A compiler supporting C++23 and `<ranges>`.

---

### Related Programming Areas

- **Functional programming**: Views and pipelines bring functional composition to C++.
- **Data processing**: ETL pipelines, log analysis, and stream processing benefit from lazy views.
- **Numerical computing**: `views::iota` and `views::transform` replace manual index loops.
- **Generic programming**: Concepts (`std::ranges::view`, `std::ranges::range`) constrain algorithms at compile time.

---

### Core Concepts / Features

1. **Views vs. Containers** — Non-owning, lazy-evaluated range structures that do not allocate or copy data.
2. **The Pipe Operator (`|`)** — Composing clean, readable processing pipelines.
3. **Key Modern Views** — `views::filter`, `views::transform`, `views::take`, `views::drop`, and C++23 additions (`views::zip`, `views::chunk`, `views::slide`).

---

## 1. Views vs. Containers

### Definitions

**Core Definition**
A container owns its elements and manages their storage eagerly. A view is a lightweight, non-owning (or optionally owning) range that provides a lazy, composable description of how to access elements from an underlying data source.

**Technical Definition**
The `std::ranges::view` concept requires that a type `V` satisfies `std::ranges::range<V>`, is move-constructible, and has O(1) copy, move, and assignment complexity. Containers like `std::vector<T>` own their elements and are not views. Views like `std::ranges::filter_view<V, Pred>` store a reference (or copy) of the underlying range and a predicate, and generate elements on demand. `std::ranges::owning_view<R>` (C++20 DR, P2415) wraps an rvalue range and takes ownership, enabling views to own their data when necessary. `std::ranges::to<C>()` (C++23) materialises a view into an owning container.

**Beginner-Friendly Explanation**
A container is like a physical photo album — it holds the photos, and they exist as long as the album does. A view is like a magnifying glass you hold over the album — it doesn't hold the photos, it just describes which ones to look at and how. If you move the album, the view is still pointing at the same place (which might be a problem if the album is gone). A view is cheap to copy: copying the magnifying glass doesn't copy the photos.

---

### Purposes (all begin with "To")

- **To** provide a non-owning, lightweight abstraction over existing data without copying or allocating.
- **To** enable lazy evaluation, deferring computation until elements are actually needed.
- **To** allow composition of multiple data transformations without creating intermediate containers.
- **To** support infinite ranges by pulling elements on demand rather than materialising them.
- **To** reduce memory overhead by eliminating intermediate storage in processing pipelines.

---

### Syntax Rules and Structure

```cpp
// A view is any type modeling std::ranges::view
template<class V>
concept view = std::ranges::range<V> &&
               std::movable<V> &&
               /* O(1) copy/move/assign */;

// Owning view (C++20 DR)
std::ranges::owning_view<R> ov(std::move(container));

// Materialise a view into a container (C++23)
auto vec = view | std::ranges::to<std::vector>();
```

**Component breakdown:**
- `std::ranges::range<V>` — the type has `begin()` and `end()`.
- `std::movable<V>` — the type is move-constructible and move-assignable.
- `std::ranges::owning_view<R>` — wraps an rvalue range and owns it.
- `std::ranges::to<C>()` — creates a container of type `C` from a view.

**Constraints and limitations:**
- Views reference data owned by something else; if the underlying container is destroyed, the view dangles.
- Views may not be `const`-iterable (e.g., `filter_view` with a stateful predicate).
- Not all algorithms are available as views (e.g., sorting cannot be lazy).
- Copying a view is O(1), but copying an `owning_view` may copy the underlying data.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — Container vs. View: Lazy vs. Eager

```cpp
// File: views_vs_containers.cpp
// Compile with: g++ -std=c++20 views_vs_containers.cpp -o views_vs_containers

#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> v = {1, 2, 3, 4, 5, 6};

    // Step 1: Eager transformation (creates a new vector)
    std::vector<int> eager;
    for (int x : v) {
        if (x % 2 == 0) {
            eager.push_back(x * x);
        }
    }
    std::cout << "Eager result: ";
    for (int x : eager) std::cout << x << " ";
    std::cout << std::endl;

    // Step 2: Lazy view (no new container, no computation yet)
    auto lazy = v
        | std::views::filter([](int x) { return x % 2 == 0; })
        | std::views::transform([](int x) { return x * x; });

    // Nothing has been computed yet!
    std::cout << "Lazy view created (no computation yet)" << std::endl;

    // Step 3: Computation happens when iterated
    std::cout << "Lazy result: ";
    for (int x : lazy) std::cout << x << " ";
    std::cout << std::endl;

    // Step 4: Materialise the view into a container (C++23)
    auto materialized = lazy | std::ranges::to<std::vector>();
    std::cout << "Materialized size: " << materialized.size() << std::endl;

    return 0;
}
```

**Expected Output:**
```
Eager result: 4 16 36
Lazy view created (no computation yet)
Lazy result: 4 16 36
Materialized size: 3
```

**Why this output:** The eager version creates a new vector and computes all results immediately. The lazy view stores only the description of the computation; no work is done until the `for` loop iterates over it. `ranges::to` then materialises the view into a container.

---

#### Example 2 — Infinite Range with Lazy Views

```cpp
// File: infinite_view.cpp
// Compile with: g++ -std=c++20 infinite_view.cpp -o infinite_view

#include <iostream>
#include <ranges>

int main() {
    // Step 1: Infinite source (iota without bound)
    auto infinite = std::views::iota(1);  // 1, 2, 3, 4, ...

    // Step 2: Lazy pipeline with a bound
    auto pipeline = infinite
        | std::views::filter([](int n) { return n % 2 == 0; })
        | std::views::transform([](int n) { return n * n; })
        | std::views::take(5);

    // Step 3: Only 5 elements are ever computed
    std::cout << "First 5 even squares: ";
    for (int x : pipeline) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

**Expected Output:**
```
First 5 even squares: 4 16 36 64 100
```

**Why this output:** `views::iota(1)` represents an infinite sequence. The pipeline is lazy, so `take(5)` stops the evaluation after 5 elements. Only the first 10 integers (1–10) are ever examined from the infinite source.

---

### Real-World Cases with Explanation

**Case 1 — Log file processing**: A `views::filter` selects only error-level log entries, and `views::transform` extracts timestamps. No intermediate vector is created, saving memory for large log files.

**Case 2 — Real-time data streams**: A view over a sensor data buffer filters out invalid readings and transforms raw values into engineering units, all without copying the raw data.

**Case 3 — Numerical simulation**: `views::iota(0, N)` replaces a manual index loop; `views::transform` applies a mathematical function lazily to each index, and `views::take_while` stops when a convergence criterion is met.

**Case 4 — CSV parsing**: A view over a string of CSV data, split by lines and fields, filters and transforms entries without allocating intermediate strings.

---

## 2. The Pipe Operator (`|`)

### Definitions

**Core Definition**
The pipe operator (`|`) is an overloaded operator that composes a range on the left with a range adaptor closure on the right, producing a new view. It enables declarative, left-to-right data-processing pipelines.

**Technical Definition**
A *range adaptor closure* is a function object created by partially applying a range adaptor (e.g., `std::views::filter(pred)` returns an adaptor closure). The expression `range | adaptor_closure` is defined as `adaptor_closure(range)`. The `operator|` overload is constrained to types modeling `std::ranges::range` on the left and `std::ranges::range_adaptor_closure` on the right. This design allows chaining: `r | a | b | c` is equivalent to `c(b(a(r)))` but reads in the order the operations are applied.

**Beginner-Friendly Explanation**
The pipe operator is like an assembly line. Data enters on the left, passes through each stage (`filter`, `transform`, `take`), and comes out on the right. You read the pipeline from left to right, just like you think about the data flowing through it.

---

### Purposes (all begin with "To")

- **To** provide a readable, left-to-right syntax for composing range transformations.
- **To** eliminate deeply nested function calls (e.g., `take(transform(filter(v)))`).
- **To** enable declarative, pipeline-style programming that mirrors the data flow.
- **To** allow adaptors to be reused and partially applied as closures.
- **To** separate the definition of a pipeline from its execution (lazy evaluation).

---

### Syntax Rules and Structure

```cpp
// Basic pipeline
source | adaptor1(args) | adaptor2(args) | adaptor3(args)

// Example
v | std::views::filter(pred) | std::views::transform(fn) | std::views::take(n)

// Equivalent nested form
std::views::take(n,
    std::views::transform(fn,
        std::views::filter(pred, v)))
```

**Component breakdown:**
- `source` — any range (container, view, iota, etc.).
- `|` — the pipe operator, composing a range with an adaptor closure.
- `adaptor(args)` — a range adaptor closure produced by partially applying a range adaptor.

**Constraints and limitations:**
- The left operand must model `std::ranges::range`.
- The right operand must be a `range_adaptor_closure`.
- Adaptors that take multiple ranges (e.g., `views::zip`) cannot be partially applied with `|` alone; they require explicit call syntax.
- Pipelines are lazy; they produce a view, not a container, unless materialised with `ranges::to`.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — Basic Pipeline Composition

```cpp
// File: pipe_operator_basic.cpp
// Compile with: g++ -std=c++20 pipe_operator_basic.cpp -o pipe_operator_basic

#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> v = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};

    // Step 1: Define the pipeline
    auto pipeline = v
        | std::views::filter([](int x) { return x % 2 == 0; })
        | std::views::transform([](int x) { return x * x; })
        | std::views::take(3);

    // Step 2: Iterate the pipeline (computation happens here)
    std::cout << "Pipeline result: ";
    for (int x : pipeline) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    // Step 3: Compare with nested call syntax
    auto nested = std::views::take(3,
        std::views::transform([](int x) { return x * x; },
            std::views::filter([](int x) { return x % 2 == 0; }, v)));

    std::cout << "Nested result:  ";
    for (int x : nested) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

**Expected Output:**
```
Pipeline result: 4 16 36
Nested result:  4 16 36
```

**Why this output:** The pipeline filters even numbers (2, 4, 6, 8, 10), squares them (4, 16, 36, 64, 100), and takes the first three (4, 16, 36). The nested form is equivalent but harder to read.

---

#### Example 2 — Pipeline with Stateful Adaptor Closure

```cpp
// File: pipe_closure.cpp
// Compile with: g++ -std=c++20 pipe_closure.cpp -o pipe_closure

#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> v = {5, 2, 8, 1, 9, 3, 7};

    // Step 1: Create a reusable adaptor closure
    auto even_and_squared = std::views::filter([](int x) {
        return x % 2 == 0;
    }) | std::views::transform([](int x) {
        return x * x;
    });

    // Step 2: Apply to different ranges
    auto result1 = v | even_and_squared;
    std::cout << "From v: ";
    for (int x : result1) std::cout << x << " ";
    std::cout << std::endl;

    std::vector<int> w = {10, 11, 12, 13, 14};
    auto result2 = w | even_and_squared;
    std::cout << "From w: ";
    for (int x : result2) std::cout << x << " ";
    std::cout << std::endl;

    return 0;
}
```

**Expected Output:**
```
From v: 4 64
From w: 100 144 196
```

**Why this output:** The closure `even_and_squared` composes `filter` and `transform` into a reusable pipeline stage. It is then applied to two different ranges (`v` and `w`), demonstrating the composability and reusability of adaptor closures.

---

### Real-World Cases with Explanation

**Case 1 — Data analysis**: `data | views::filter(is_valid) | views::transform(normalize) | views::take(100)` selects the first 100 valid, normalised records without creating intermediate vectors.

**Case 2 — Image processing**: A pipeline of pixel transformations (grayscale, blur, edge detect) is expressed as a chain of views, applied lazily to each pixel on demand.

**Case 3 — JSON processing**: A JSON array is viewed as a range, filtered by field values, transformed into domain objects, and materialised only when needed.

**Case 4 — Config file parsing**: Lines of a config file are filtered by section, transformed into key-value pairs, and taken until a terminator is reached.

---

## 3. Key Modern Views

### Definitions

**Core Definition**
Modern views are standard range adaptors that transform, filter, truncate, or restructure ranges. C++20 provides the core set (`filter`, `transform`, `take`, `drop`, `reverse`, `iota`, `join`, `split`), and C++23 adds multi-range and windowing adaptors (`zip`, `chunk`, `slide`, `stride`, `chunk_by`, `enumerate`, `adjacent`).

**Technical Definition**
Each view is a class template (e.g., `std::ranges::filter_view<V, Pred>`) with a corresponding range adaptor object (e.g., `std::views::filter`). The adaptor object is a niebloid that, when called with its arguments, returns either a view (if a range is passed) or an adaptor closure (if only the predicate is passed). The views satisfy the `std::ranges::view` concept and provide iterators whose category depends on the underlying range and the adaptor.

**Beginner-Friendly Explanation**
Views are the building blocks of pipelines. `filter` keeps only elements that pass a test, `transform` changes each element, `take` grabs the first N, and `drop` skips the first N. C++23 adds tools like `zip` (combine multiple ranges), `chunk` (split into fixed-size groups), and `slide` (moving windows).

---

### Sub-Feature: C++20 Core Views (`filter`, `transform`, `take`, `drop`)

#### Definitions

**Core Definition**
`views::filter(pred)` yields only elements for which `pred` returns `true`. `views::transform(fn)` applies `fn` to each element. `views::take(n)` yields the first `n` elements. `views::drop(n)` skips the first `n` elements.

**Technical Definition**
`filter_view<V, Pred>` stores a view of `V` and a predicate `Pred`; its iterator advances until the predicate is satisfied. `transform_view<V, F>` stores a view of `V` and a function `F`; its iterator applies `F` on dereference. `take_view<V>` and `drop_view<V>` store a count and a view; `take` stops iteration after `n` elements, `drop` advances the begin iterator by `n`.

**Beginner-Friendly Explanation**
- `filter`: "Give me only the elements that pass this test."
- `transform`: "Apply this function to every element."
- `take`: "Give me the first N elements."
- `drop`: "Skip the first N elements."

#### Purposes (all begin with "To")

- **To** filter elements based on a predicate without creating a new container.
- **To** apply a transformation to each element lazily.
- **To** limit a range to its first N elements.
- **To** skip the first N elements of a range.
- **To** compose these operations into readable pipelines.

#### Syntax Rules and Structure

```cpp
// Filter
range | std::views::filter(pred);

// Transform
range | std::views::transform(fn);

// Take
range | std::views::take(n);

// Drop
range | std::views::drop(n);
```

**Constraints and limitations:**
- `filter` requires the predicate to be invocable with the element type and return something convertible to `bool`.
- `transform` requires the function to be invocable.
- `take` and `drop` require the range to be at least input-range for `take`, and forward-range for `drop` (in C++20; relaxed in C++23).
- `take` and `drop` can be combined with `iota` for index-based generation.

#### Annotated Code Example

```cpp
// File: core_views.cpp
// Compile with: g++ -std=c++20 core_views.cpp -o core_views

#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> v = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};

    // Step 1: filter + transform + take
    auto result = v
        | std::views::filter([](int x) { return x % 2 == 0; })
        | std::views::transform([](int x) { return x * x; })
        | std::views::take(3);

    std::cout << "filter+transform+take: ";
    for (int x : result) std::cout << x << " ";
    std::cout << std::endl;

    // Step 2: drop + take (skip first 3, take next 4)
    auto window = v
        | std::views::drop(3)
        | std::views::take(4);

    std::cout << "drop(3)+take(4): ";
    for (int x : window) std::cout << x << " ";
    std::cout << std::endl;

    // Step 3: Infinite iota with filter + take
    auto even_squares = std::views::iota(1)
        | std::views::filter([](int x) { return x % 2 == 0; })
        | std::views::transform([](int x) { return x * x; })
        | std::views::take(5);

    std::cout << "Infinite iota pipeline: ";
    for (int x : even_squares) std::cout << x << " ";
    std::cout << std::endl;

    return 0;
}
```

**Expected Output:**
```
filter+transform+take: 4 16 36
drop(3)+take(4): 4 5 6 7
Infinite iota pipeline: 4 16 36 64 100
```

**Why this output:** The first pipeline filters even numbers (2, 4, 6, 8, 10), squares them, and takes the first three. The second skips the first three elements (1, 2, 3) and takes the next four (4, 5, 6, 7). The third demonstrates an infinite source bounded by `take(5)`.

---

#### Real-World Cases with Explanation

**Case 1 — Pagination**: `views::drop(page * page_size) | views::take(page_size)` implements pagination over any range without copying the entire dataset.

**Case 2 — Data validation**: `views::filter(is_valid) | views::transform(parse)` filters out invalid records and parses the remaining ones lazily.

**Case 3 — Top-N queries**: `views::filter(condition) | views::take(N)` efficiently retrieves the first N elements matching a condition.

---

### Sub-Feature: C++23 Windowing and Multi-Range Views (`zip`, `chunk`, `slide`)

#### Definitions

**Core Definition**
C++23 introduces adaptors for combining multiple ranges (`zip`), splitting a range into fixed-size non-overlapping groups (`chunk`), and creating sliding windows (`slide`). These enable expressive multi-sequence and windowed processing.

**Technical Definition**
`zip_view` (C++23) takes one or more ranges and produces a view of tuples, where the i-th element contains the i-th element from each range. `chunk_view` (C++23) splits a range into subranges of a specified size (non-overlapping, last chunk may be smaller). `slide_view` (C++23) produces overlapping subranges of a specified size, advancing by one element at a time. All are lazy views that do not allocate.

**Beginner-Friendly Explanation**
- `zip`: Combine two or more ranges element-by-element, like a zipper.
- `chunk`: Split a range into equal-sized blocks, like cutting a loaf into slices.
- `slide`: Create a moving window over a range, like a sliding glass door.

#### Purposes (all begin with "To")

- **To** combine multiple ranges into a single range of tuples for parallel iteration.
- **To** split a range into fixed-size chunks for batch processing.
- **To** create overlapping windows for algorithms like moving averages or n-gram extraction.
- **To** avoid manual index arithmetic when processing grouped or windowed data.
- **To** enable expressive multi-range algorithms without intermediate containers.

#### Syntax Rules and Structure

```cpp
// Zip — combine ranges
std::views::zip(range1, range2, ...);

// Chunk — split into non-overlapping groups
range | std::views::chunk(n);

// Slide — create sliding windows
range | std::views::slide(n);

// Chunk_by — split where predicate is false between adjacent elements
range | std::views::chunk_by(pred);
```

**Component breakdown:**
- `std::views::zip(a, b, c)` — takes multiple ranges; produces tuples `(a[i], b[i], c[i])`.
- `std::views::chunk(n)` — takes a size `n`; produces subranges of length `n` (last may be shorter).
- `std::views::slide(n)` — takes a window size `n`; produces overlapping subranges of length `n`.

**Constraints and limitations:**
- `zip` requires all ranges to be at least input ranges; the resulting view's iterator category is the minimum of the input ranges' categories.
- `chunk` requires a forward range in C++23 (relaxed from input range in C++20).
- `slide` requires a forward range.
- `chunk` and `slide` produce subranges, not vectors; materialisation requires `ranges::to`.

#### Annotated Code Example

```cpp
// File: cpp23_views.cpp
// Compile with: g++ -std=c++23 cpp23_views.cpp -o cpp23_views

#include <iostream>
#include <ranges>
#include <vector>
#include <string>

int main() {
    // Step 1: zip — combine names and scores
    std::vector<std::string> names = {"Alice", "Bob", "Charlie"};
    std::vector<int> scores = {85, 92, 78};

    std::cout << "Zipped:" << std::endl;
    for (auto [name, score] : std::views::zip(names, scores)) {
        std::cout << "  " << name << ": " << score << std::endl;
    }

    // Step 2: chunk — split into groups of 3
    std::vector<int> v = {1, 2, 3, 4, 5, 6, 7, 8};
    std::cout << "Chunked (size 3):" << std::endl;
    for (auto chunk : v | std::views::chunk(3)) {
        std::cout << "  [";
        for (int x : chunk) std::cout << x << " ";
        std::cout << "]" << std::endl;
    }

    // Step 3: slide — sliding window of size 3
    std::cout << "Sliding (size 3):" << std::endl;
    for (auto window : v | std::views::slide(3)) {
        std::cout << "  [";
        for (int x : window) std::cout << x << " ";
        std::cout << "]" << std::endl;
    }

    // Step 4: chunk_by — group consecutive even numbers
    std::vector<int> w = {1, 2, 4, 5, 6, 8, 9};
    std::cout << "Chunk_by (even runs):" << std::endl;
    for (auto group : w | std::views::chunk_by(
             [](int a, int b) { return a % 2 == b % 2; })) {
        std::cout << "  [";
        for (int x : group) std::cout << x << " ";
        std::cout << "]" << std::endl;
    }

    return 0;
}
```

**Expected Output:**
```
Zipped:
  Alice: 85
  Bob: 92
  Charlie: 78
Chunked (size 3):
  [1 2 3 ]
  [4 5 6 ]
  [7 8 ]
Sliding (size 3):
  [1 2 3 ]
  [2 3 4 ]
  [3 4 5 ]
  [4 5 6 ]
  [5 6 7 ]
  [6 7 8 ]
Chunk_by (even runs):
  [1 ]
  [2 4 ]
  [5 ]
  [6 8 ]
  [9 ]
```

**Why this output:** `views::zip` pairs names with scores. `views::chunk(3)` splits into non-overlapping groups of 3 (last group has 2 elements). `views::slide(3)` creates overlapping windows of 3, advancing by 1 each time. `views::chunk_by` groups adjacent elements where the predicate holds (consecutive even or consecutive odd).

---

#### Real-World Cases with Explanation

**Case 1 — Moving average**: `data | views::slide(5) | views::transform(average)` computes a 5-point moving average without manual index management.

**Case 2 — Batch API calls**: `ids | views::chunk(100) | views::transform(fetch_batch)` splits a large list of IDs into batches of 100 for API calls.

**Case 3 — Parallel arrays**: `views::zip(names, ages, scores)` iterates over three parallel arrays simultaneously, eliminating index arithmetic errors.

**Case 4 — N-gram extraction**: `text | views::slide(3)` produces all 3-character n-grams for language modelling.

**Case 5 — Pagination**: `views::chunk(page_size)` combined with `views::drop(page_number)` implements pagination without manual offset calculations.

---

## Summary Table: Key Views

| View | C++ Version | Description | Complexity | Requires |
|---|---|---|---|---|
| `filter(pred)` | C++20 | Keep elements satisfying `pred` | Lazy, O(1) per element | Input range |
| `transform(fn)` | C++20 | Apply `fn` to each element | Lazy, O(1) per element | Input range |
| `take(n)` | C++20 | First `n` elements | Lazy, O(1) per element | Input range |
| `drop(n)` | C++20 | Skip first `n` elements | Lazy, O(1) per element | Forward range |
| `reverse` | C++20 | Reverse the range | Lazy, O(1) per element | Bidirectional range |
| `iota(start)` | C++20 | Generate sequence from `start` | Lazy, O(1) per element | N/A |
| `join` | C++20 | Flatten a range of ranges | Lazy | Input range of ranges |
| `zip(r1, r2, ...)` | C++23 | Combine ranges into tuples | Lazy, O(1) per element | Input ranges |
| `chunk(n)` | C++23 | Non-overlapping groups of size `n` | Lazy, O(1) per group | Forward range |
| `slide(n)` | C++23 | Overlapping windows of size `n` | Lazy, O(1) per window | Forward range |
| `chunk_by(pred)` | C++23 | Split where predicate changes | Lazy | Forward range |
| `stride(n)` | C++23 | Every n-th element | Lazy, O(1) per element | Input range |
| `enumerate` | C++23 | Index + value pairs | Lazy, O(1) per element | Input range |

---

## References

- Ranges library (C++20) — cppreference.com - https://en.cppreference.com/w/cpp/ranges
- Range adaptors — cppreference.com - https://en.cppreference.com/w/cpp/ranges#Range_adaptors
- std::ranges::views Symbol Index — cppreference.com - https://en.cppreference.com/w/cpp/symbol_index/views
- std::ranges::filter_view — cppreference.com - https://en.cppreference.com/w/cpp/ranges/filter_view
- std::ranges::transform_view — cppreference.com - https://en.cppreference.com/w/cpp/ranges/transform_view
- std::ranges::take_view — cppreference.com - https://en.cppreference.com/w/cpp/ranges/take_view
- std::ranges::drop_view — cppreference.com - https://en.cppreference.com/w/cpp/ranges/drop_view
- std::ranges::zip_view — cppreference.com - https://en.cppreference.com/w/cpp/ranges/zip_view
- std::ranges::chunk_view — cppreference.com - https://en.cppreference.com/w/cpp/ranges/chunk_view
- std::ranges::slide_view — cppreference.com - https://en.cppreference.com/w/cpp/ranges/slide_view
- P0896R4: The One Ranges Proposal — Open-Std - https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/p0896r4.pdf
- P2214R2: A Plan for C++23 Ranges — Open-Std - https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2214r2.html
- P2415R2: What is a view? — Open-Std - https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p2415r2.html
- Owning and non-owning C++ Ranges — Hannes Hauswedell, isocpp.org - https://isocpp.org/blog/2025/05/owning-and-non-owning-cpp-ranges
- Use range adaptors: filter, transform, take, drop, and their composition — Dr-Sergey/learn_cpp - https://github.com/Dr-Sergey/learn_cpp/blob/main/14_Ranges_Cpp20/Use_range_adaptors_filter_transform_take_drop_and_their_composition.md
- Use std::chunk_by, stride, and cartesian_product Views (C++23) — Dr-Sergey/learn_cpp - https://github.com/Dr-Sergey/learn_cpp/blob/main/26_Standard_Library_New_Cpp23_26/Use_stdchunk_by_stride_and_cartesian_product_views_C++23.md