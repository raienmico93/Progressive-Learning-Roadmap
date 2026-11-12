# C++ Arrays — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
`std::array` is a fixed-size sequence container in the C++ Standard Library that encapsulates a C-style array while providing a standard container interface.

**Technical Definition**
`std::array<T, N>` is an aggregate type with the same semantics as a struct holding a C-style array `T[N]` as its only non-static data member. It satisfies the requirements of *Container* and *ReversibleContainer* (except that a default-constructed array is not empty and swapping is linear), *ContiguousContainer* (since C++17), and partially satisfies *SequenceContainer*. Unlike a C-style array, it does not decay to `T*` automatically.

**Beginner-Friendly Explanation**
`std::array` is a safer, more convenient version of a C array. Its size is fixed at compile time, but it provides member functions like `size()`, `at()`, and iterators, so you don’t have to rely on raw pointers or fragile `sizeof` calculations.

### Key Characteristics

- **Fixed size** – `N` is a compile-time constant and cannot change.
- **Contiguous storage** – elements are stored in a single contiguous block, enabling pointer arithmetic and cache-friendly access.
- **Zero overhead** – memory layout is bit-for-bit identical to a C-style array `T[N]`; no hidden size field or pointer.
- **Does not decay** – `std::array` does not implicitly convert to `T*`, preserving size information when passed to functions.
- **Aggregate type** – supports aggregate initialization and structured bindings (C++17).
- **Iterator stability** – iterators are never invalidated throughout the lifetime of the array; `swap` changes values but iterators continue to point to the same element.
- **Stack allocation** – like a built-in array, a stack-allocated `std::array` keeps its elements on the stack.

### Prerequisites

- Basic C++ syntax (variables, types, control flow).
- Understanding of C-style arrays and memory layout.
- Familiarity with iterators and the range-based `for` loop.
- Elementary knowledge of templates (`std::array` is a class template).

### Related Programming Areas

- **STL algorithms** – `std::sort`, `std::find`, `std::transform` operate seamlessly on `std::array`.
- **Generic programming** – `std::array` is a class template parameterised on element type and size.
- **Compile-time computation** – `constexpr` support makes `std::array` usable in constant expressions.
- **API design** – passing `std::array` by reference preserves size information.
- **Embedded systems** – fixed-size, stack-allocated storage is ideal for constrained environments.

### Core Concepts / Features

1. `std::array` (the container itself)
2. Array declaration
3. Array initialization
4. Array operations
5. Array vs. C-style array
6. Array best practices

---

## 1. `std::array`

### Definitions

**Core Definition**
`std::array<T, N>` is a class template that represents a fixed-size array of `N` elements of type `T`.

**Technical Definition**
The template is declared in `<array>` as:
```cpp
template<class T, std::size_t N>
struct array;
```
(since C++11). It is an aggregate type that combines the performance and accessibility of a C-style array with the benefits of a standard container: it knows its own size, supports assignment, provides random-access iterators, and can be compared with `==` and `<`.

**Beginner-Friendly Explanation**
`std::array` is like a box with a fixed number of compartments — you decide the size when you write the code, and it never changes. Unlike a raw C array, it knows how many compartments it has and can be copied and compared like a normal object.

### Purposes

- To provide a fixed-size array with a standard container interface.
- To avoid the pitfalls of C arrays (pointer decay, lack of size information).
- To enable stack allocation of fixed-size sequences.
- To allow seamless use with STL algorithms and range-based `for`.
- To support compile-time evaluation via `constexpr`.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <array>
std::array<T, N> arr;                 // declaration
std::array<T, N> arr = {a, b, c};    // aggregate initialization
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `T` | The type of elements stored. |
| `N` | The number of elements (a `std::size_t` constant expression). |
| `arr` | The variable identifier. |

**Syntax Rules**
- The header `<array>` must be included.
- `N` must be a constant expression; it cannot be a runtime value.
- `std::array` is defined in namespace `std`.
- `N` may be zero; `std::array<T, 0>` is valid (an empty array).
- Aggregate initialization allows at most `N` initializers convertible to `T`.

**Constraints and Limitations**
- The size cannot change at runtime.
- Default construction of `std::array` of non-class type leaves elements uninitialised (indeterminate values).
- `front()` and `back()` on a zero-sized array are undefined behaviour.
- Swapping is linear (O(N)), not constant time, because elements are swapped individually.

### Annotated Code Examples

**Example 1: Creating a `std::array`**

```cpp
#include <array>
#include <iostream>

int main() {
    std::array<int, 5> a = {10, 20, 30, 40, 50};
    std::cout << a.size() << '\n';        // 5
    std::cout << a[2] << '\n';            // 30
}
```

**Expected Output**
```
5
30
```

**Why this output?**  
The aggregate initializer fills the array with five values. `size()` returns `N` (5), and `a[2]` accesses the third element (`30`).

**Example 2: `std::array` with a custom type**

```cpp
#include <array>
#include <iostream>
#include <string>

struct Point { int x, y; };

int main() {
    std::array<Point, 2> pts = {{{1, 2}, {3, 4}}};
    std::cout << pts[1].x << ',' << pts[1].y << '\n';
}
```

**Expected Output**
```
3,4
```

**Why this output?**  
Aggregate initialization constructs `Point` objects; `pts[1]` accesses the second point.

### Real-World Cases

- **Fixed-size buffers** – embedded systems, device drivers.
- **Mathematical vectors** – small fixed-size numeric arrays.
- **Lookup tables** – compile-time-sized tables.
- **Stack-allocated collections** – where heap allocation is undesirable.

---

## 2. Array Declaration

### Definitions

**Core Definition**
Array declaration introduces a `std::array` variable into a scope, specifying its element type and size.

**Technical Definition**
A declaration of a `std::array` object specifies the template arguments `T` (element type) and `N` (size), plus the variable name. It may or may not include an initializer.

**Beginner-Friendly Explanation**
Declaring a `std::array` means telling the compiler, “I want a fixed-size array that holds this type of item and this many of them,” and giving it a name.

### Purposes

- To create a `std::array` variable of a specific type and size.
- To reserve a name for later use.
- To allow the compiler to allocate the appropriate amount of stack storage.
- To support generic code via `auto` or type aliases.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::array<T, N> name;
std::array<T, N> name = { ... };
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `std::array` | The class template. |
| `<T, N>` | Element type and size. |
| `name` | The variable identifier. |

**Syntax Rules**
- `N` must be a constant expression (e.g., literal, `constexpr` variable, template parameter).
- The element type must be a complete type at the point of instantiation.
- `auto` cannot deduce `N` from an initializer list; the size must be explicit unless using `std::to_array` (C++20).

**Constraints and Limitations**
- A `std::array` cannot be declared with a runtime-variable size.
- The size is part of the type; two arrays with different `N` have different types.

### Annotated Code Examples

**Example 1: Simple declaration**

```cpp
#include <array>

int main() {
    std::array<int, 3> numbers;            // uninitialized ints
    std::array<double, 5> values{};        // value-initialized to 0.0
}
```

**Expected Output**  
(No output; arrays are constructed.)

**Why this output?**  
`numbers` is default-initialized (indeterminate values for `int`). `values` is value-initialized to zero via `{}`.

**Example 2: Declaration with `constexpr` size**

```cpp
#include <array>
#include <iostream>

int main() {
    constexpr std::size_t size = 4;
    std::array<int, size> arr = {1, 2, 3, 4};
    std::cout << arr.size() << '\n';
}
```

**Expected Output**
```
4
```

**Why this output?**  
`size` is a constant expression, so it can be used as the array size.

### Real-World Cases

- **Class members** – `std::array<int, 3> coordinates_;`.
- **Function parameters** – `void process(const std::array<int, 5>& data);`.
- **Local buffers** – stack-allocated fixed-size storage.

---

## 3. Array Initialization

### Definitions

**Core Definition**
Array initialization is the process of providing initial values to a `std::array` at construction time.

**Technical Definition**
`std::array` is an aggregate type and is initialized using *aggregate initialization*. It can be given at most `N` initializers that are convertible to `T`. The forms include copy-list-initialization (`= {...}`) and direct-list-initialization (`{...}`).

**Beginner-Friendly Explanation**
Initialization is how you fill the array with values when you create it. You list the values inside braces, and the array stores them in order.

### Purposes

- To populate a `std::array` with known values.
- To value-initialize elements to zero.
- To support compile-time initialization of constant arrays.
- To allow aggregate initialization without writing a constructor.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::array<T, N> a = {v1, v2, ..., vN};   // copy-list-initialization
std::array<T, N> a{ v1, v2, ..., vN };    // direct-list-initialization
std::array<T, N> a{};                     // value-initialization (zeros)
```

**Component Breakdown**

| Form | Effect |
|------|--------|
| `= { ... }` | Copy-list-initialization (double braces not required after `=` since C++14) |
| `{ ... }` | Direct-list-initialization (C++11 may require double braces in some cases) |
| `{}` | Value-initialization: all elements are zero-initialized |

**Syntax Rules**
- Aggregate initialization requires at most `N` initializers.
- Elements are initialized in order; omitted trailing elements are value-initialized.
- For `std::array` of class type, each element is copy-initialized from the corresponding initializer.
- Since C++14, a single set of braces suffices in most cases; older C++11 code sometimes required double braces (`{{...}}`) due to aggregate rules.

**Constraints and Limitations**
- Default initialization (without `{}`) of a `std::array` of non-class type leaves elements with indeterminate values.
- You cannot initialize with more than `N` initializers.

### Annotated Code Examples

**Example 1: List initialization**

```cpp
#include <array>
#include <iostream>

int main() {
    std::array<int, 5> a = {1, 2, 3, 4, 5};
    for (int x : a) std::cout << x << ' ';
}
```

**Expected Output**
```
1 2 3 4 5
```

**Why this output?**  
The initializer list fills the array in order.

**Example 2: Value-initialization**

```cpp
#include <array>
#include <iostream>

int main() {
    std::array<int, 4> a{};                // all zeros
    for (int x : a) std::cout << x << ' ';
}
```

**Expected Output**
```
0 0 0 0
```

**Why this output?**  
`{}` value-initializes each element to zero.

**Example 3: Partial initialization**

```cpp
#include <array>
#include <iostream>

int main() {
    std::array<int, 5> a = {1, 2};        // remaining elements are zero
    for (int x : a) std::cout << x << ' ';
}
```

**Expected Output**
```
1 2 0 0 0
```

**Why this output?**  
Omitted trailing elements are value-initialized to zero.

### Real-World Cases

- **Lookup tables** – initializing with known values at compile time.
- **Configuration defaults** – value-initializing to zero.
- **Testing** – constructing arrays with known contents for unit tests.

---

## 4. Array Operations

### Definitions

**Core Definition**
Array operations are the member functions and non-member functions that allow access, modification, and querying of `std::array` elements.

**Technical Definition**
`std::array` provides the standard container interface: element access (`at`, `operator[]`, `front`, `back`, `data`), iterators (`begin`, `end`, `rbegin`, `rend`, etc.), capacity (`size`, `max_size`, `empty`), and operations (`fill`, `swap`). Most operations are `constexpr` and `noexcept`.

**Beginner-Friendly Explanation**
Even though `std::array` is fixed-size, it still gives you many useful commands — you can check its size, access elements safely or quickly, fill it with a value, and iterate through it.

### Key Operations

| Operation | Description | Complexity |
|-----------|-------------|------------|
| `at(pos)` | Bounds-checked access; throws `std::out_of_range` | O(1) |
| `operator[]` | Unchecked access | O(1) |
| `front()` | Reference to first element | O(1) |
| `back()` | Reference to last element | O(1) |
| `data()` | Pointer to underlying array | O(1) |
| `size()` | Number of elements (`N`) | O(1) |
| `max_size()` | Maximum number of elements (`N`) | O(1) |
| `empty()` | Always `false` for `N > 0`; `true` for `N == 0` | O(1) |
| `fill(value)` | Sets all elements to `value` | O(N) |
| `swap(other)` | Swaps contents with another array | O(N) |
| `begin()` / `end()` | Iterators to first / past-the-end | O(1) |

**Syntax Rules**
- `at()` performs bounds checking and throws `std::out_of_range` on invalid access.
- `operator[]` does not check bounds; accessing out of range is undefined behaviour.
- `fill()` sets every element to the given value.
- `swap()` is linear because elements are swapped individually.
- `data()` returns a pointer to the first element; for `N == 0`, the pointer may be null or non-null (implementation-defined).

**Constraints and Limitations**
- `front()` and `back()` on a zero-sized array are undefined.
- `swap()` is O(N), unlike the O(1) swap of most other containers.

### Annotated Code Examples

**Example 1: Element access with `at()` and `operator[]`**

```cpp
#include <array>
#include <iostream>
#include <stdexcept>

int main() {
    std::array<int, 3> a = {10, 20, 30};
    std::cout << a[1] << '\n';             // 20 (no bounds check)
    std::cout << a.at(2) << '\n';          // 30 (bounds checked)
    try {
        std::cout << a.at(5) << '\n';      // throws
    } catch (const std::out_of_range& e) {
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
`operator[]` and `at()` return the element at the given index. `at(5)` throws because index 5 is out of bounds.

**Example 2: `fill()` and `swap()`**

```cpp
#include <array>
#include <iostream>

int main() {
    std::array<int, 4> a = {1, 2, 3, 4};
    std::array<int, 4> b;
    b.fill(9);                             // b = {9, 9, 9, 9}
    a.swap(b);                             // linear swap
    for (int x : a) std::cout << x << ' ';
}
```

**Expected Output**
```
9 9 9 9
```

**Why this output?**  
`fill(9)` sets every element of `b` to `9`; `swap` exchanges the contents of `a` and `b`.

**Example 3: Using `data()` for C interoperability**

```cpp
#include <array>
#include <iostream>

void print_c_array(const int* arr, int n) {
    for (int i = 0; i < n; ++i) std::cout << arr[i] << ' ';
}

int main() {
    std::array<int, 5> a = {1, 2, 3, 4, 5};
    print_c_array(a.data(), a.size());
}
```

**Expected Output**
```
1 2 3 4 5
```

**Why this output?**  
`data()` returns a pointer to the contiguous storage, which can be passed to C-style functions.

### Real-World Cases

- **Algorithm input** – passing `std::array` to `std::sort`.
- **Data serialization** – accessing `data()` for binary I/O.
- **State reset** – using `fill()` to reset a buffer.

---

## 5. Array vs. C-Style Array

### Definitions

**Core Definition**
A C-style array is a built-in language construct (`T arr[N]`), while `std::array` is a standard-library container that wraps a C-style array with a richer interface.

**Technical Definition**
A C-style array is not a first-class object: it decays to a pointer when passed to a function, cannot be copied or compared with `=`, and provides no member functions. `std::array` is a first-class object with value semantics, size awareness, bounds-checked access, and full container support.

**Beginner-Friendly Explanation**
A C-style array is like a raw block of memory with sharp edges — it doesn’t know its own size, it turns into a pointer when you pass it around, and you can’t copy it easily. `std::array` is the same block of memory but wrapped in a safe, convenient object.

### Comparison Table

| Feature | `int arr[5]` | `std::array<int, 5>` |
|---------|--------------|----------------------|
| Size known at compile time | Via fragile `sizeof` calculation | `.size()` — always correct |
| Bounds checking | No | `.at()` throws `std::out_of_range` |
| Decays to pointer | Yes (dangerous!) | No — explicit `.data()` needed |
| Copyable | No (must use `memcpy`) | Yes — `operator=` works |
| Comparable | No | Yes — `==`, `<`, etc. |
| Works with STL algorithms | Via raw pointers | Via `.begin()` / `.end()` |
| Structured bindings (C++17) | No | Yes |
| `constexpr` friendly | Limited | Fully (C++17+) |
| Zero overhead | Baseline | Same — no overhead |
| Stack allocation | Yes | Yes |

**Source:** Adapted from.

### Annotated Code Examples

**Example 1: Size awareness**

```cpp
#include <array>
#include <iostream>

void process_c(int* arr) {
    // cannot determine size here
}

template <std::size_t N>
void process_cpp(const std::array<int, N>& arr) {
    std::cout << arr.size() << '\n';       // size available
}

int main() {
    int c_arr[5] = {1, 2, 3, 4, 5};
    std::array<int, 5> cpp_arr = {1, 2, 3, 4, 5};
    process_cpp(cpp_arr);
    // process_c(c_arr);                    // size lost
}
```

**Expected Output**
```
5
```

**Why this output?**  
`std::array` preserves its size through the function parameter, while a C-style array would decay to a pointer.

**Example 2: Copy and comparison**

```cpp
#include <array>
#include <iostream>

int main() {
    std::array<int, 3> a = {1, 2, 3};
    std::array<int, 3> b = a;              // copy works
    std::cout << std::boolalpha << (a == b) << '\n'; // true
}
```

**Expected Output**
```
true
```

**Why this output?**  
`std::array` supports copy assignment and comparison; C-style arrays do not.

### Real-World Cases

- **API design** – preferring `std::array` parameters to preserve size information.
- **Generic code** – using `std::array` with templates and STL algorithms.
- **Safety-critical systems** – avoiding pointer decay and buffer overruns.

---

## 6. Array Best Practices

### Definitions

**Core Definition**
Array best practices are guidelines for using `std::array` correctly, safely, and efficiently.

**Technical Definition**
These practices are codified in the C++ Core Guidelines, particularly **SL.con.1** (“Prefer using STL `array` or `vector` instead of a C array”) and related rules. The rationale is that C arrays are less safe, have no advantages over `std::array` and `std::vector`, and `std::array` does not degenerate to a pointer and knows its size.

**Beginner-Friendly Explanation**
Best practices are good habits: use `std::array` instead of raw C arrays, prefer `at()` when safety matters, and remember that `std::array` is the right choice for fixed-size collections.

### Key Guidelines

| Guideline | Summary |
|-----------|---------|
| **SL.con.1** | Prefer `std::array` or `std::vector` over C arrays. |
| **Use `std::array` for fixed size** | When the size is known at compile time, `std::array` is the right tool. |
| **Use `std::vector` for dynamic size** | When the size can change at runtime, use `vector`. |
| **Prefer `at()` for bounds safety** | When safety is more important than raw speed. |
| **Use `data()` for C interoperability** | But ensure the pointer is used before the array’s lifetime ends. |
| **Prefer range-based `for`** | More readable than index or iterator loops. |
| **Use `constexpr`** | `std::array` operations are largely `constexpr`; exploit compile-time evaluation. |
| **Be aware of zero-size arrays** | `std::array<T, 0>` is valid, but `front()` / `back()` are undefined. |

### Annotated Code Example

```cpp
#include <array>
#include <iostream>

void process(const std::array<int, 5>& data) {   // const reference: no copy
    for (int x : data) std::cout << x << ' ';
}

int main() {
    std::array<int, 5> arr = {1, 2, 3, 4, 5};
    process(arr);
}
```

**Expected Output**
```
1 2 3 4 5
```

**Why this output?**  
`std::array` is passed by `const` reference, avoiding a copy and preserving size information. The range-based `for` iterates safely over all elements.

### Real-World Cases

- **Code reviews** – checking for C arrays that should be `std::array`.
- **Static analysis** – tools like clang-tidy enforce Core Guidelines.
- **Embedded systems** – preferring `std::array` for fixed-size, stack-allocated storage.

---

## References

- `std::array` — cppreference – https://en.cppreference.com/w/cpp/container/array
- `std::array` (printable version) — cppreference – https://en.cppreference.com/mwiki/index.php?title=cpp/container/array&oldid=76618&printable=yes
- C++ Core Guidelines — SL.con.1: Prefer using STL `array` or `vector` instead of a C array – https://cpp-core-guidelines-docs.vercel.app/stdlib
- `std::array` — libstdc++ man page – https://manpages.debian.org/unstable/libstdc++-15-doc/std::array.3cxx.en.html
- Use `std::array` for fixed-size stack-allocated arrays — Learn C++ – https://raw.githubusercontent.com/Dr-Sergey/learn_cpp/a285937786f8ea542f74de40134f69efeaa45174/07_Standard_Library_Containers/Use_stdarray_for_fixed-size_stack-allocated_arrays.md
- C++ `std::array` — Programiz – https://www.programiz.com/cpp-programming/std-array
- `std::array` — Microsoft Learn – https://learn.microsoft.com/en-us/cpp/standard-library/array-class-stl
- Aggregate initialization — cppreference – https://en.cppreference.com/w/cpp/language/aggregate_initialization

---

This cheat sheet provides a comprehensive, structured reference for C++ `std::array`, covering definitions, declaration, initialization, operations, comparisons with C-style arrays, and best practices, with annotated code examples and authoritative references.