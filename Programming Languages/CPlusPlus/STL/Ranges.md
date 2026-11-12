# C++20 Ranges — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
The C++20 ranges library is an extension and generalization of the algorithms and iterator libraries that makes them more powerful by making them composable and less error-prone, introducing lightweight *views* that indirectly represent iterable sequences.

**Technical Definition**
The `<ranges>` header defines the namespace `std::ranges` and the namespace alias `std::views` (shorthand for `std::ranges::views`). The library creates and manipulates *range views*—lightweight objects that indirectly represent iterable sequences (ranges). Ranges are an abstraction on top of `[begin, end)` iterator pairs, `begin + [0, size)` counted sequences, `[begin, predicate)` conditionally-terminated sequences, and `[begin, ..)` unbounded sequences. The library includes range algorithms, which are applied to ranges eagerly, and range adaptors, which are applied to views lazily. Adaptors can be composed into pipelines, so that their actions take place as the view is iterated.

**Beginner-Friendly Explanation**
The C++20 ranges library is a modern way to work with sequences of data. Instead of passing pairs of iterators (`begin` and `end`) to algorithms, you can pass the container (or any "range") directly. More importantly, it lets you build *pipelines* of operations—like "take these numbers, keep only the even ones, square them, and take the first five"—without creating temporary containers. These pipelines are lazy: they only do work when you actually read the results.

### Key Characteristics

- **Composable** – range adaptors can be chained together using the pipe operator `|` to form processing pipelines.
- **Lazy evaluation** – range adaptors applied to views perform work only as the view is iterated; no intermediate containers are created.
- **Eager vs. lazy** – range algorithms (e.g., `std::ranges::sort`) are applied eagerly to ranges; range adaptors (e.g., `std::views::filter`) are applied lazily to views.
- **Concept-constrained** – the library uses C++20 concepts to constrain templates, producing clearer error messages and safer code.
- **Range-based** – most algorithms accept a single range argument instead of an iterator pair, and support projections and pointer-to-member callables.
- **Views are cheap** – a view is an O(1) copy/move/assign/destroy object that does not own its elements (except `owning_view`).

### Prerequisites

- Basic C++ syntax (variables, types, control flow).
- Familiarity with STL containers and iterators.
- Understanding of templates and C++20 concepts.
- Knowledge of lambda expressions and function objects.
- Awareness of the classic `<algorithm>` library.

### Related Programming Areas

- **STL algorithms** – `std::ranges::` provides constrained, range-accepting versions of classic algorithms.
- **Iterators** – ranges are built on top of iterator and sentinel abstractions.
- **Containers** – any standard container can be used as a range.
- **Functional programming** – pipelines mirror map/filter/reduce patterns.
- **Lazy evaluation** – views enable deferred computation.

### Core Concepts / Features

1. Ranges
2. `<ranges>` (the header and namespace)
3. Range concepts
4. Range views
5. Range adaptors
6. Range algorithms
7. Range factories
8. Pipes
9. Lazy evaluation
10. Range best practices

---

## 1. Ranges

### Definitions

**Core Definition**
A range is an abstraction that represents an iterable sequence of elements, defined by a begin iterator and an end sentinel.

**Technical Definition**
The `std::ranges::range` concept specifies that a type `T` models a range if `ranges::begin(t)` and `ranges::end(t)` are valid expressions, where `t` is an lvalue of type `T`. The iterator and sentinel must denote the elements of the range, and both `begin` and `end` must be amortized constant time.

**Beginner-Friendly Explanation**
A range is anything you can loop over: a vector, an array, a string, or even an infinite sequence of numbers. The ranges library lets you treat all of these uniformly.

### Purposes

- To provide a uniform abstraction over different kinds of sequences.
- To allow algorithms to accept a single range argument instead of an iterator pair.
- To enable composition through views and adaptors.
- To support both bounded and unbounded sequences.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <ranges>
// A range is any type that provides begin() and end()
std::vector<int> v = {1, 2, 3};          // v is a range
std::ranges::sort(v);                     // range algorithm
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `begin()` | Returns an iterator to the first element. |
| `end()` | Returns a sentinel indicating the end. |
| `range` concept | Requires valid `begin`/`end` expressions. |

**Syntax Rules**
- A typical range class provides member `begin()` and `end()`, or non-member functions found by ADL.
- `ranges::begin` and `ranges::end` are customization point objects (CPOs) that work with member, non-member, and array types.
- Forward iterators must be equality-preserving for multi-pass algorithms.

**Constraints and Limitations**
- A type without `begin`/`end` is not a range.
- `begin` must return a type modelling `input_or_output_iterator`; `end` must return a type modelling `sentinel_for`.

### Annotated Code Examples

**Example 1: A vector is a range**

```cpp
#include <ranges>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {3, 1, 4};
    std::ranges::sort(v);                  // accepts the range directly
    for (int x : v) std::cout << x << ' ';
}
```

**Expected Output**
```
1 3 4
```

**Why this output?**  
`std::ranges::sort` accepts the vector as a range and sorts it in place.

**Example 2: A C array is a range**

```cpp
#include <ranges>
#include <iostream>

int main() {
    int arr[] = {5, 2, 8};
    std::ranges::sort(arr);
    for (int x : arr) std::cout << x << ' ';
}
```

**Expected Output**
```
2 5 8
```

**Why this output?**  
C arrays provide `begin`/`end` via `std::ranges::begin`/`std::ranges::end`, so they model the `range` concept.

### Real-World Cases

- **Container-agnostic APIs** – functions accepting `std::ranges::range auto&&` work with any container.
- **Infinite sequences** – `views::iota` produces an unbounded range.
- **Algorithm simplification** – no more `v.begin(), v.end()` boilerplate.

---

## 2. `<ranges>`

### Definitions

**Core Definition**
`<ranges>` is the standard library header that declares the ranges library, including range concepts, views, adaptors, factories, and range algorithms.

**Technical Definition**
The header defines the namespace `std::ranges` and the namespace alias `std::views` (equivalent to `std::ranges::views`). It provides range access CPOs (`begin`, `end`, `cbegin`, `cend`, `rbegin`, `rend`, `size`, `empty`, `data`, etc.), range primitives, range concepts, views, adaptors, and factories.

**Beginner-Friendly Explanation**
`<ranges>` is the header you include to use the C++20 ranges features. It brings in everything you need for range-based programming.

### Purposes

- To provide a single header for all ranges functionality.
- To define the `std::ranges` and `std::views` namespaces.
- To expose range access customization point objects.
- To declare range algorithms and adaptors.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <ranges>
// Use std::ranges:: and std::views:: namespaces
```

**Component Breakdown**

| Namespace | Purpose |
|-----------|---------|
| `std::ranges` | Range concepts, algorithms, access CPOs, primitives. |
| `std::views` | Range adaptors and factories (alias for `std::ranges::views`). |

**Syntax Rules**
- `std::views` is a namespace alias for `std::ranges::views` (since C++20).
- Range algorithms live in `std::ranges`; range adaptors live in `std::views`.

### Annotated Code Example

```cpp
#include <ranges>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {1, 2, 3};
    auto it = std::ranges::begin(v);       // CPO
    std::cout << *it << '\n';              // 1
}
```

**Expected Output**
```
1
```

**Why this output?**  
`std::ranges::begin` is a customization point object that returns an iterator to the first element.

### Real-World Cases

- **Library development** – exposing range-based interfaces.
- **Generic utilities** – using range access CPOs for container-agnostic code.

---

## 3. Range Concepts

### Definitions

**Core Definition**
Range concepts are C++20 concepts that classify types by their range capabilities, from basic iterability to view semantics.

**Technical Definition**
The ranges library defines several key concepts: `std::ranges::range` (has `begin`/`end`), `std::ranges::borrowed_range` (safe to use after the range object dies), `std::ranges::sized_range` (O(1) `size()`), `std::ranges::viewable_range` (can be safely converted to a view), and `std::ranges::view` (O(1) copy/move, non-owning).

**Beginner-Friendly Explanation**
Range concepts are labels that tell you what a range can do. A `view` is a cheap, non-owning range. A `viewable_range` is something you can pipe into a view adaptor. A `borrowed_range` is safe to use even after the original range object is gone.

### Purposes

- To constrain templates to types with specific range capabilities.
- To enable compile-time checking of range operations.
- To distinguish owning containers from lightweight views.
- To support safe composition in pipelines.

### Syntax Rules and Structure

**General Syntax**
```cpp
template <std::ranges::range R>
void process(R&& r);
```

**Concept Hierarchy**

| Concept | Key Requirement | Example YES | Example NO |
|---------|-----------------|-------------|------------|
| `range` | Has `begin()` + `end()` | `vector`, `string`, `span` | `int` |
| `view` | O(1) move, non-owning | `string_view`, `span`, `filter_view` | `vector` (O(n) copy) |
| `viewable_range` | Can be passed to `\|` | `vector&`, `string_view`, views | rvalue `vector` (C++20) |
| `borrowed_range` | Iterators outlive range | `span`, `string_view`, `subrange` | `vector` |

*Source: Dr-Sergey/learn_cpp concept hierarchy table.*

**Syntax Rules**
- `view` requires constant-time move construction and (if copyable) constant-time copy construction.
- `viewable_range` is a refinement of `range` that can be converted to a view via `views::all`.
- `borrowed_range` allows algorithms to return iterators safely.

**Constraints and Limitations**
- Containers like `std::vector` are **not** views because copying is O(n).
- An lvalue of a move-only view is rejected by `viewable_range` (since C++20 DR).

### Annotated Code Examples

**Example 1: Checking range concepts**

```cpp
#include <ranges>
#include <vector>
#include <string_view>
#include <iostream>

int main() {
    std::vector<int> v = {1, 2, 3};
    std::string_view sv = "hello";

    std::cout << std::boolalpha;
    std::cout << std::ranges::range<decltype(v)> << '\n';       // true
    std::cout << std::ranges::view<decltype(v)> << '\n';        // false
    std::cout << std::ranges::view<decltype(sv)> << '\n';       // true
}
```

**Expected Output**
```
true
false
true
```

**Why this output?**  
A vector is a range but not a view; a string_view is both a range and a view.

**Example 2: Using a range concept as a constraint**

```cpp
#include <ranges>
#include <vector>
#include <iostream>

template <std::ranges::range R>
void print_all(R&& r) {
    for (auto&& x : r) std::cout << x << ' ';
}

int main() {
    std::vector<int> v = {1, 2, 3};
    print_all(v);                          // works
}
```

**Expected Output**
```
1 2 3
```

**Why this output?**  
The `print_all` template is constrained to accept any type modelling `std::ranges::range`.

### Real-World Cases

- **API constraints** – functions accepting only ranges.
- **View vs. container overloads** – dispatching based on `view` concept.
- **Safe iterator returns** – using `borrowed_range` to avoid dangling.

---

## 4. Range Views

### Definitions

**Core Definition**
A view is a lightweight, non-owning range that provides a indirect representation of an underlying sequence of elements.

**Technical Definition**
The `std::ranges::view` concept requires a type to model `range` and `std::movable`, and to have constant-time move construction, move assignment, and destruction (and constant-time copy operations if copyable). Views are the fundamental building blocks of range pipelines; they do not own their elements (except `owning_view`) and are cheap to copy, assign, and destroy regardless of the number of elements involved.

**Beginner-Friendly Explanation**
A view is like a window onto a sequence. It doesn't contain the data; it just knows how to look at it. Because it's so lightweight, you can pass it around, copy it, and combine it with other views without worrying about performance.

### Purposes

- To provide a cheap, non-owning representation of a sequence.
- To enable lazy, composable transformations.
- To avoid intermediate containers in pipelines.
- To serve as the return type of range adaptors.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::views::filter(pred)               // creates a view
std::views::transform(fn)              // creates a view
```

**Component Breakdown**

| Property | Requirement |
|----------|-------------|
| Move construction | O(1) |
| Move assignment | O(1) |
| Destruction | O(1) |
| Copy construction | O(1) if copyable |
| Ownership | Non-owning (except `owning_view`) |

**Syntax Rules**
- Views model `std::ranges::view`.
- They are returned by range adaptors (`std::views::filter`, `std::views::transform`, etc.).
- They can be composed into pipelines.

**Constraints and Limitations**
- Views do not own their elements (except `owning_view`); the underlying range must outlive the view.
- Modifying elements through a view modifies the underlying range.

### Annotated Code Example

```cpp
#include <ranges>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {1, 2, 3, 4, 5};
    auto evens = v | std::views::filter([](int x){ return x % 2 == 0; });

    for (int x : evens) std::cout << x << ' ';
}
```

**Expected Output**
```
2 4
```

**Why this output?**  
`std::views::filter` creates a view over the even elements of `v`; no new container is created.

### Real-World Cases

- **Lazy filtering** – processing only elements that match a condition.
- **Transformation pipelines** – mapping functions over sequences.
- **Memory efficiency** – avoiding temporary containers.

---

## 5. Range Adaptors

### Definitions

**Core Definition**
Range adaptors are utilities that transform a range into a view, optionally applying filtering, transformation, truncation, or other operations.

**Technical Definition**
Range adaptors accept a `viewable_range` as their first argument and return a `view` (or a view adaptor closure object when partially applied). They are customization point objects (CPOs) in the `std::views` namespace. Adaptors can be composed into pipelines, so that their actions take place as the view is iterated (lazily).

**Beginner-Friendly Explanation**
Range adaptors are the building blocks of pipelines. `views::filter` keeps only elements that pass a test; `views::transform` applies a function to each element; `views::take` limits the number of elements. You chain them together with `|`.

### Purposes

- To transform a range into a view with specific behaviour.
- To enable lazy, composable data processing.
- To avoid intermediate copies and allocations.
- To provide a declarative style for sequence manipulation.

### Common Range Adaptors (C++20)

| Adaptor | Description |
|---------|-------------|
| `views::filter(pred)` | View of elements satisfying `pred`. |
| `views::transform(fn)` | View of elements transformed by `fn`. |
| `views::take(n)` | View of the first `n` elements. |
| `views::drop(n)` | View skipping the first `n` elements. |
| `views::reverse` | View of elements in reverse order. |
| `views::join` | View that flattens a range of ranges. |
| `views::split(delim)` | View split by a delimiter. |
| `views::common` | View adapted to a common range. |
| `views::counted(it, n)` | Counted view from an iterator. |

*Source: Microsoft Learn range adaptors list, cppreference.*

**Syntax Rules**
- Adaptors are in `std::views` (alias for `std::ranges::views`).
- They accept a `viewable_range` as their first argument.
- Adaptors can be partially applied (e.g., `views::filter(pred)` returns a closure).
- Adaptors compose via `|`.

**Constraints and Limitations**
- Some adaptors require specific iterator categories (e.g., `views::reverse` requires bidirectional).
- The underlying range must outlive the view.

### Annotated Code Examples

**Example 1: `filter` and `transform` pipeline**

```cpp
#include <ranges>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10};

    auto result = v
        | std::views::filter([](int x){ return x % 3 == 0; })
        | std::views::transform([](int x){ return x * x; });

    for (int x : result) std::cout << x << ' ';
}
```

**Expected Output**
```
0 9 36 81
```

**Why this output?**  
`filter` keeps multiples of 3 (`0, 3, 6, 9`), and `transform` squares them, all lazily as the pipeline is iterated.

**Example 2: `take` and `drop`**

```cpp
#include <ranges>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {1, 2, 3, 4, 5, 6};

    for (int x : v | std::views::drop(2) | std::views::take(3))
        std::cout << x << ' ';
}
```

**Expected Output**
```
3 4 5
```

**Why this output?**  
`drop(2)` skips the first two elements; `take(3)` limits the result to three elements.

### Real-World Cases

- **Data filtering** – selecting records that match criteria.
- **Transformation pipelines** – applying calculations to data streams.
- **Pagination** – using `drop`/`take` to page through results.

---

## 6. Range Algorithms

### Definitions

**Core Definition**
Range algorithms are constrained versions of the classic STL algorithms that accept a range argument instead of an iterator pair, and support projections and pointer-to-member callables.

**Technical Definition**
C++20 provides `std::ranges::` versions of most algorithms in the `std::ranges` namespace. These algorithms accept either an iterator-sentinel pair or a single range argument. They are algorithm function objects (AFOs, informally *niebloids*), which means they cannot be called with explicit template arguments and are not found by argument-dependent lookup. Return types have been changed to return all potentially useful information computed during execution.

**Beginner-Friendly Explanation**
Range algorithms are the same algorithms you know (`sort`, `find`, `count`, etc.), but you can call them with a whole container instead of a pair of iterators, and they support extra conveniences like member pointers.

### Purposes

- To simplify algorithm calls by accepting a range directly.
- To support projections for member access.
- To provide clearer compile-time error messages via concepts.
- To return richer information (e.g., iterators to found elements).

### Syntax Rules and Structure

**General Syntax**
```cpp
std::ranges::algorithm_name(range, ...);       // range overload
std::ranges::algorithm_name(first, last, ...); // iterator overload
```

**Component Breakdown**

| Feature | Description |
|---------|-------------|
| Range argument | A single object with `begin()`/`end()`. |
| Projection | A unary callable applied to each element before comparison. |
| Return type | Often returns a richer result (e.g., `in_found_result`). |

**Syntax Rules**
- Range algorithms live in `std::ranges`.
- They accept either a range or an iterator-sentinel pair.
- Projections are supported (e.g., `&Person::name`).

**Constraints and Limitations**
- Not all classic algorithms have range versions.
- They are AFOs: no explicit template arguments, no ADL.

### Annotated Code Examples

**Example 1: `ranges::sort` vs. `std::sort`**

```cpp
#include <ranges>
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {3, 1, 4, 1, 5};

    std::ranges::sort(v);                  // range overload
    for (int x : v) std::cout << x << ' '; // 1 1 3 4 5
}
```

**Expected Output**
```
1 1 3 4 5
```

**Why this output?**  
`std::ranges::sort` accepts the vector directly; no iterator pair is needed.

**Example 2: Projection with `ranges::find`**

```cpp
#include <ranges>
#include <vector>
#include <iostream>
#include <string>

struct Person { std::string name; int age; };

int main() {
    std::vector<Person> people = {{"Alice", 30}, {"Bob", 25}};
    auto it = std::ranges::find(people, 25, &Person::age);
    if (it != people.end())
        std::cout << it->name << '\n';     // Bob
}
```

**Expected Output**
```
Bob
```

**Why this output?**  
The projection `&Person::age` tells `find` to compare against the `age` member.

### Real-World Cases

- **Simplifying algorithm calls** – no more `begin()`/`end()`.
- **Member access** – projections for sorting or searching by a field.
- **Safer code** – concepts produce clearer errors.

---

## 7. Range Factories

### Definitions

**Core Definition**
Range factories are utilities that create views from scratch, generating elements on the fly without storing them in a container.

**Technical Definition**
Range factories are defined in the `std::ranges` namespace. Key factories include `views::iota` (generates a sequence by repeatedly incrementing an initial value; can be bounded or unbounded), `views::repeat` (C++23; generates the same value repeatedly), `views::empty` (empty range), and `views::single` (single-element range).

**Beginner-Friendly Explanation**
Range factories are ways to create sequences out of thin air. `views::iota(0, 5)` gives you `0, 1, 2, 3, 4`. `views::repeat(42)` gives you an infinite stream of `42`s. These are perfect for generating data without storing it.

### Purposes

- To create sequences without an underlying container.
- To generate infinite or bounded sequences.
- To serve as starting points for pipelines.
- To provide building blocks for range construction.

### Common Range Factories (C++20/C++23)

| Factory | Standard | Generates | Bounded? |
|---------|----------|-----------|----------|
| `views::iota(start)` | C++20 | `start, start+1, start+2, ...` | Infinite |
| `views::iota(start, end)` | C++20 | `start, start+1, ..., end-1` | Finite |
| `views::repeat(val)` | C++23 | `val, val, val, ...` | Infinite |
| `views::repeat(val, n)` | C++23 | `val` repeated `n` times | Finite |
| `views::empty<T>` | C++20 | Nothing | Finite (0 elements) |
| `views::single(val)` | C++20 | Just `val` | Finite (1 element) |

*Source: Dr-Sergey/learn_cpp generating views summary.*

**Syntax Rules**
- Factories are in `std::views`.
- `views::iota` works with any `weakly_incrementable` type (integers, pointers, iterators).
- Infinite ranges must be truncated (e.g., with `views::take`) before consuming.

**Constraints and Limitations**
- Passing an infinite range to `ranges::to<vector>()` attempts infinite allocation.
- `views::repeat` is C++23, not C++20.

### Annotated Code Examples

**Example 1: Bounded `iota`**

```cpp
#include <ranges>
#include <iostream>

int main() {
    for (int x : std::views::iota(1, 6))
        std::cout << x << ' ';
}
```

**Expected Output**
```
1 2 3 4 5
```

**Why this output?**  
`iota(1, 6)` generates the sequence `1, 2, 3, 4, 5`.

**Example 2: Infinite `iota` with `take`**

```cpp
#include <ranges>
#include <iostream>

int main() {
    for (int x : std::views::iota(2) | std::views::take(5))
        std::cout << x << ' ';
}
```

**Expected Output**
```
2 3 4 5 6
```

**Why this output?**  
`iota(2)` is infinite; `take(5)` truncates it to five elements.

### Real-World Cases

- **Generating indices** – `views::iota(0, n)` for iteration.
- **Simulation** – repeating a value.
- **Testing** – creating bounded sequences for test cases.

---

## 8. Pipes

### Definitions

**Core Definition**
The pipe operator `|` composes a range with a range adaptor (or a range adaptor closure) to produce a new view.

**Technical Definition**
The pipe operator is syntactic sugar for function application: `range | adaptor` is equivalent to `adaptor(range)`. Because adaptors can be partially applied, `range | views::filter(pred) | views::transform(fn)` chains operations left-to-right, creating a pipeline that is evaluated lazily.

**Beginner-Friendly Explanation**
The pipe `|` lets you write data processing pipelines in a readable, left-to-right style: “take this data, then filter it, then transform it.” It reads like a series of steps.

### Purposes

- To compose range adaptors in a readable left-to-right order.
- To avoid deeply nested function calls.
- To enable a functional pipeline style.
- To make lazy pipelines easy to construct.

### Syntax Rules and Structure

**General Syntax**
```cpp
range | adaptor1 | adaptor2 | adaptor3
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `range` | Any `viewable_range`. |
| `adaptor` | A range adaptor or closure. |
| `|` | Pipe operator. |

**Syntax Rules**
- `range | adaptor` is equivalent to `adaptor(range)`.
- Multiple pipes chain left-to-right.
- The result is a view (lazy).

**Constraints and Limitations**
- The left operand must be a `viewable_range`.
- Some adaptors require specific iterator categories.

### Annotated Code Example

```cpp
#include <ranges>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {1, 2, 3, 4, 5, 6};

    auto result = v
        | std::views::filter([](int x){ return x % 2 == 0; })
        | std::views::transform([](int x){ return x * 10; })
        | std::views::take(2);

    for (int x : result) std::cout << x << ' ';
}
```

**Expected Output**
```
20 40
```

**Why this output?**  
The pipeline keeps evens (`2, 4, 6`), multiplies by 10 (`20, 40, 60`), and takes the first two (`20, 40`). All operations are lazy and composed via `|`.

### Real-World Cases

- **Data processing** – filtering and transforming streams.
- **Pagination** – `data | views::drop(page * size) | views::take(size)`.
- **Log analysis** – chaining filters and projections.

---

## 9. Lazy Evaluation

### Definitions

**Core Definition**
Lazy evaluation means that the operations in a view pipeline are performed only when elements are actually accessed (iterated), not when the pipeline is constructed.

**Technical Definition**
Range adaptors applied to views are lazy: they produce elements on demand with no intermediate containers allocated. The work is done as the view is iterated, element by element. This contrasts with range algorithms, which are eager—they process the entire range immediately.

**Beginner-Friendly Explanation**
Lazy evaluation means the pipeline doesn't do any work upfront. It just sets up the steps. When you loop over the result, each element flows through the pipeline one at a time, and only the elements you actually read are processed.

### Purposes

- To avoid unnecessary computation on unused elements.
- To avoid allocating intermediate containers.
- To enable processing of infinite ranges.
- To improve performance for large or partially consumed ranges.

### Syntax Rules and Structure

**Conceptual Model**
```
Lazy pipeline:
range → [filter] → [transform] → [take] → output
          ↑            ↑           ↑
       no work      no work     no work until iterated
```

**Component Breakdown**

| Aspect | Eager (algorithms) | Lazy (adaptors) |
|--------|-------------------|-----------------|
| Work done | Immediately | On iteration |
| Intermediate containers | May be created | None |
| Infinite ranges | Not supported | Supported (with truncation) |
| Return type | Algorithm result | View |

**Syntax Rules**
- Adaptors are lazy; algorithms are eager.
- Pipelines are only evaluated when iterated.
- Infinite ranges must be truncated before full consumption.

**Constraints and Limitations**
- Lazy pipelines may recompute elements if iterated multiple times.
- Dangling references can occur if the underlying range is destroyed.

### Annotated Code Example

```cpp
#include <ranges>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {1, 2, 3, 4, 5};

    auto pipeline = v
        | std::views::filter([](int x){ std::cout << "filter " << x << '\n'; return x % 2 == 0; })
        | std::views::transform([](int x){ std::cout << "transform " << x << '\n'; return x * 10; });

    std::cout << "Pipeline created (no work done yet)\n";

    for (int x : pipeline) std::cout << "Result: " << x << '\n';
}
```

**Expected Output** (illustrative)
```
Pipeline created (no work done yet)
filter 1
filter 2
transform 2
Result: 20
filter 3
filter 4
transform 4
Result: 40
filter 5
```

**Why this output?**  
No filtering or transformation occurs until the pipeline is iterated. Each element is processed on demand, and the interleaved output shows that work happens lazily, element by element.

### Real-World Cases

- **Large data sets** – processing only the needed prefix.
- **Infinite streams** – generating and consuming on demand.
- **Expensive transformations** – avoiding work on filtered-out elements.

---

## 10. Range Best Practices

### Definitions

**Core Definition**
Range best practices are guidelines for using the C++20 ranges library correctly, safely, and idiomatically.

**Technical Definition**
These practices are grounded in the C++ Core Guidelines and community consensus: prefer range algorithms over iterator-pair algorithms, use views for lazy composition, be mindful of lifetime and dangling issues, and understand the difference between eager algorithms and lazy adaptors.

**Beginner-Friendly Explanation**
Best practices for ranges are simple: use `std::ranges::` algorithms instead of the old iterator-pair versions, build pipelines with `|` for lazy processing, and make sure the underlying data lives long enough.

### Key Guidelines

| Guideline | Summary |
|-----------|---------|
| **Prefer range algorithms** | Use `std::ranges::sort(v)` instead of `std::sort(v.begin(), v.end())`. |
| **Use views for lazy pipelines** | Compose with `|` to avoid intermediate containers. |
| **Be aware of lifetime** | Views do not own data; ensure the underlying range outlives the view. |
| **Truncate infinite ranges** | Use `views::take` or `views::take_while` before consuming. |
| **Know eager vs. lazy** | Algorithms are eager; adaptors are lazy. |
| **Use projections** | Simplify member access in algorithms (`&Person::name`). |
| **Constrain with concepts** | Use `std::ranges::range` and related concepts in templates. |
| **Avoid dangling returns** | Return `borrowed_range` or owning types, not views over locals. |
| **Prefer `views::all` for type erasure** | Convert a range to a view explicitly when needed. |
| **Test with infinite ranges carefully** | Use `take` to bound them. |

### Annotated Code Example

```cpp
#include <ranges>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {5, 2, 8, 1, 9};

    // Range algorithm with projection
    std::ranges::sort(v);
    for (int x : v) std::cout << x << ' '; // 1 2 5 8 9

    // Lazy pipeline
    auto top3 = v | std::views::reverse | std::views::take(3);
    for (int x : top3) std::cout << x << ' '; // 9 8 5
}
```

**Expected Output**
```
1 2 5 8 9 9 8 5
```

**Why this output?**  
`ranges::sort` eagerly sorts the vector; the lazy pipeline reverses and takes the top three elements without creating a temporary container.

### Real-World Cases

- **Code reviews** – checking that range pipelines don't dangle.
- **Performance tuning** – using lazy views to avoid allocations.
- **API design** – accepting `std::ranges::range auto&&` parameters.

---

## References

- Ranges library (since C++20) — cppreference – https://en.cppreference.com/w/cpp/ranges
- `std::ranges::range` — cppreference – https://en.cppreference.com/w/cpp/ranges/range
- `std::ranges::view` — cppreference – https://en.cppreference.com/w/cpp/ranges/view
- `std::ranges::viewable_range` — cppreference – https://en.cppreference.com/w/cpp/ranges/viewable_range
- Constrained algorithms (since C++20) — cppreference – https://en.cppreference.com/w/cpp/algorithm/ranges
- `std::ranges::views::iota` — cppreference – https://en.cppreference.com/w/cpp/ranges/iota_view
- `std::ranges::views::repeat` — cppreference – https://en.cppreference.com/w/cpp/ranges/repeat_view
- Range adaptors — Microsoft Learn – https://learn.microsoft.com/en-us/cpp/standard-library/range-adaptors
- Understand the ranges concept hierarchy: range, view, viewable_range — Dr-Sergey/learn_cpp – https://raw.githubusercontent.com/Dr-Sergey/learn_cpp/203d4ffc38c82d34fa170c6083184d9fec383dd9/14_Ranges_Cpp20/Understand_the_ranges_concept_hierarchy_range_view_viewable_range.md
- Use `views::iota`, `views::repeat`, and other generating views — Dr-Sergey/learn_cpp – https://github.com/Dr-Sergey/learn_cpp/blob/08279578ce23b1cbf2d3394357620600db618eca/14_Ranges_Cpp20/Use_viewsiota_viewsrepeat_and_other_generating_views.md
- Use range algorithms instead of iterator-pair algorithms — Dr-Sergey/learn_cpp – https://raw.githubusercontent.com/Dr-Sergey/learn_cpp/c7f616bfee7b04f48d1c7513de508afa375d6a11/14_Ranges_Cpp20/Use_range_algorithms_instead_of_iterator-pair_algorithms.md
- C++ Core Guidelines — SL.con: Containers – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-stdlib

---

This cheat sheet provides a comprehensive, structured reference for the C++20 ranges library, covering ranges, `<ranges>`, range concepts, views, adaptors, algorithms, factories, pipes, lazy evaluation, and best practices, with annotated code examples, expected outputs, real-world cases, and authoritative references.