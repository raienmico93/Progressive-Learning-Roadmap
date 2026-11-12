# C++ Vectors — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
`std::vector` is a sequence container in the C++ Standard Library that encapsulates a dynamically sized array, storing its elements contiguously in memory.

**Technical Definition**
`std::vector<T, Allocator = std::allocator<T>>` is a class template that satisfies the requirements of *Container*, *AllocatorAwareContainer*, and *SequenceContainer*. It provides random-access iterators, amortized constant-time insertion and removal at the end, and linear-time insertion and removal elsewhere. A pointer to an element of a `vector` may be passed to any C function expecting a pointer to an array element, because the storage is contiguous.

**Beginner-Friendly Explanation**
Think of `std::vector` as a “smart array” that can grow and shrink automatically. You can access any element instantly by index, add items to the end quickly, and let the vector handle all memory management for you.

### Key Characteristics

- **Contiguous storage** – elements are stored in a single contiguous block, enabling pointer arithmetic and cache-friendly access.
- **Dynamic size** – the vector grows and shrinks at runtime as elements are inserted or removed.
- **Amortized O(1) `push_back`** – appending to the end is fast on average.
- **Random access** – `operator[]` and `at()` provide O(1) access to any element.
- **Iterator invalidation** – reallocation invalidates all iterators, pointers, and references; insertion/erasure in the middle invalidates iterators at or after the operation point.
- **Value semantics** – copying a vector copies all its elements.
- **Allocator-aware** – custom memory allocators can be supplied.

### Prerequisites

- Basic C++ syntax (variables, types, control flow).
- Understanding of arrays and memory layout.
- Familiarity with iterators and the range-based `for` loop.
- Elementary knowledge of Big-O notation.

### Related Programming Areas

- **STL algorithms** – `std::sort`, `std::find`, `std::transform` operate seamlessly on vectors.
- **Generic programming** – vectors are class templates parameterised on element type.
- **Memory management** – allocators and capacity management (`reserve`, `shrink_to_fit`).
- **Performance optimization** – cache locality, reallocation avoidance.
- **API design** – passing vectors by `const&` or `span`.

### Core Concepts / Features

1. `std::vector` (the container itself)
2. Vector declaration
3. Vector initialization
4. Vector operations
   - `push_back()`
   - `pop_back()`
   - `emplace_back()`
   - `insert()`
   - `erase()`
   - `clear()`
   - `resize()`
   - `reserve()`
   - `shrink_to_fit()`
   - `at()`
   - `operator[]`
   - `front()`
   - `back()`
   - `data()`
   - `size()`
   - `capacity()`
   - `empty()`
5. Vector iteration
6. Vector performance
7. Vector best practices

---

## 1. `std::vector`

### Definitions

**Core Definition**
`std::vector` is a sequence container that provides dynamic array functionality with automatic memory management.

**Technical Definition**
The template is declared in `<vector>` as:
```cpp
template<class T, class Allocator = std::allocator<T>>
class vector;
```
It meets the requirements of *Container*, *AllocatorAwareContainer*, and *SequenceContainer*. Elements are stored contiguously, and the container supports random-access iterators.

**Beginner-Friendly Explanation**
`std::vector` is the default container you should reach for when you need a resizable array. It handles all the memory allocation and deallocation for you.

### Purposes

- To provide a dynamically sized array with contiguous storage.
- To offer fast random access by index.
- To support efficient insertion and removal at the end.
- To serve as the default sequence container in most C++ applications.
- To interoperate with C APIs via `data()`.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <vector>
std::vector<T> v;                     // empty vector
std::vector<T> v(n);                  // n default-constructed elements
std::vector<T> v(n, value);           // n copies of value
std::vector<T> v = {a, b, c};        // initializer-list
std::vector<T> v(other);              // copy
std::vector<T> v(std::move(other));   // move
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `T` | The type of elements stored. |
| `Allocator` | Optional allocator type (defaults to `std::allocator<T>`). |

**Syntax Rules**
- The header `<vector>` must be included.
- `std::vector` is defined in namespace `std`.
- Elements are value-initialized when a size is specified without a value.
- The vector can be constructed from an initializer list, iterators, or another vector.

**Constraints and Limitations**
- `std::vector<bool>` is a partial specialisation that packs bits and does not provide normal reference semantics.
- Reallocation invalidates all iterators, pointers, and references to elements.
- Insertion or erasure in the middle is O(n) because subsequent elements must be shifted.

### Annotated Code Examples

**Example 1: Creating a vector**

```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {10, 20, 30};
    std::cout << v.size() << '\n';       // 3
    std::cout << v[1] << '\n';           // 20
}
```

**Expected Output**
```
3
20
```

**Why this output?**  
The initializer-list constructor creates a vector with three elements. `size()` returns 3, and `v[1]` accesses the second element (`20`).

**Example 2: Vector of a custom type**

```cpp
#include <vector>
#include <iostream>
#include <string>

struct Point { int x, y; };

int main() {
    std::vector<Point> pts = {{1, 2}, {3, 4}};
    std::cout << pts[1].x << ',' << pts[1].y << '\n';
}
```

**Expected Output**
```
3,4
```

**Why this output?**  
Aggregate initialization creates `Point` objects; `pts[1]` accesses the second point.

### Real-World Cases

- **Dynamic arrays** – storing sensor readings, user records, or any variable-length sequence.
- **Buffers** – `std::vector<char>` as a byte buffer for I/O.
- **Default container** – recommended by the C++ Core Guidelines as the default choice for sequence data.

---

## 2. Vector Declaration

### Definitions

**Core Definition**
Vector declaration introduces a `std::vector` variable into a scope, specifying its element type and optionally its allocator.

**Technical Definition**
A declaration of a `std::vector` object specifies the template arguments (element type and optionally allocator) and the variable name. It may or may not include an initializer.

**Beginner-Friendly Explanation**
Declaring a vector means telling the compiler, “I want a vector that holds this type of item,” and giving it a name.

### Purposes

- To create a vector variable of a specific element type.
- To reserve a name for later use.
- To allow the compiler to allocate the appropriate type.
- To support generic code via `auto`.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::vector<T> name;
std::vector<T> name(initializer);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `std::vector` | The class template. |
| `<T>` | The element type. |
| `name` | The variable identifier. |

**Syntax Rules**
- The element type must be complete at the point of instantiation (with some exceptions for incomplete types since C++17).
- `auto` can be used for type deduction: `auto v = std::vector<int>{1, 2, 3};`.
- A vector cannot be declared with `const` if it will be modified.

**Constraints and Limitations**
- A vector cannot be declared with an array-of-unknown-bound element type.
- The element type must be *Erasable* (destructible).

### Annotated Code Examples

**Example 1: Simple declaration**

```cpp
#include <vector>

int main() {
    std::vector<int> numbers;              // empty vector of int
    std::vector<double> values(10);        // 10 doubles, value-initialized
}
```

**Expected Output**  
(No output; vectors are constructed.)

**Why this output?**  
`numbers` is empty; `values` has 10 elements, each `0.0`.

**Example 2: `auto` declaration**

```cpp
#include <vector>
#include <iostream>

int main() {
    auto v = std::vector<int>{1, 2, 3};
    std::cout << v.size() << '\n';
}
```

**Expected Output**
```
3
```

**Why this output?**  
`auto` deduces `std::vector<int>` from the initializer list.

### Real-World Cases

- **Local variables** – declaring a vector to hold intermediate results.
- **Class members** – `std::vector<Item> items_;`.
- **Function parameters** – `void process(const std::vector<int>& data);`.

---

## 3. Vector Initialization

### Definitions

**Core Definition**
Vector initialization is the process of providing initial values or a size to a vector at construction time.

**Technical Definition**
`std::vector` offers multiple constructors: default, fill (size and optional value), range (iterator pair), copy, move, and initializer-list. The choice of constructor determines the initial size and contents.

**Beginner-Friendly Explanation**
Initialization is how you set up a vector when you create it — you can leave it empty, give it a size, fill it with a value, or list the elements you want.

### Purposes

- To create a vector with a known size.
- To fill a vector with a repeated value.
- To copy elements from another range.
- To list elements explicitly.
- To move from another vector efficiently.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::vector<T> v;                      // empty
std::vector<T> v(n);                   // n value-initialized elements
std::vector<T> v(n, value);            // n copies of value
std::vector<T> v(first, last);         // from iterator range
std::vector<T> v = {a, b, c};         // initializer-list
std::vector<T> v(other);               // copy
std::vector<T> v(std::move(other));    // move
```

**Component Breakdown**

| Constructor | Effect |
|-------------|--------|
| Default | Empty vector. |
| Fill (`n`) | `n` value-initialized elements. |
| Fill (`n, value`) | `n` copies of `value`. |
| Range (`first, last`) | Copies elements from `[first, last)`. |
| Initializer-list | Copies the listed elements. |
| Copy | Copies all elements from another vector. |
| Move | Transfers ownership of storage (since C++11). |

**Syntax Rules**
- Use `()` for size/value initialization; use `{}` for element-list initialization (Core Guidelines SL.con.2).
- `std::vector<int> v(20);` creates 20 zeros; `std::vector<int> v{20};` creates one element `20`.
- Range initialization requires input iterators.

**Constraints and Limitations**
- Initializer-list initialization prefers the initializer-list constructor over the fill constructor, which can be surprising.
- Copy initialization from a braced list of elements uses the initializer-list constructor.

### Annotated Code Examples

**Example 1: Fill vs. list initialization**

```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> a(5);          // 5 zeros
    std::vector<int> b{5};          // 1 element: 5
    std::vector<int> c(5, 7);       // 5 sevens
    std::vector<int> d{1, 2, 3};    // 1, 2, 3

    std::cout << a.size() << ' ' << b.size() << ' '
              << c.size() << ' ' << d.size() << '\n';
}
```

**Expected Output**
```
5 1 5 3
```

**Why this output?**  
Parentheses create a vector of the given size; braces create a vector from the listed elements.

**Example 2: Range initialization**

```cpp
#include <vector>
#include <iostream>

int main() {
    int arr[] = {1, 2, 3, 4};
    std::vector<int> v(arr, arr + 4);
    for (int x : v) std::cout << x << ' ';
}
```

**Expected Output**
```
1 2 3 4
```

**Why this output?**  
The range constructor copies elements from `arr[0]` through `arr[3]`.

### Real-World Cases

- **Reading data** – initializing a vector from a file or network buffer.
- **Configuration** – creating a vector of default values.
- **Testing** – constructing vectors with known contents for unit tests.

---

## 4. Vector Operations

### 4.1 `push_back()`

#### Definitions

**Core Definition**
`push_back()` appends a copy of its argument to the end of the vector.

**Technical Definition**
`void push_back(const T& value);` and `void push_back(T&& value);` (since C++11). The operation has amortized O(1) complexity. If reallocation occurs, all iterators, pointers, and references to elements are invalidated.

**Beginner-Friendly Explanation**
`push_back` adds an item to the end of the vector. It is fast, but every so often the vector may need to grow, which can invalidate iterators.

#### Purposes

- To add elements to the end of a vector.
- To build a vector incrementally.
- To append existing objects.
- To support stack-like usage.

#### Syntax Rules and Structure

**General Syntax**
```cpp
v.push_back(value);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `v` | The vector object. |
| `value` | The element to copy (or move) into the vector. |

**Syntax Rules**
- The argument must be convertible to `T`.
- If the vector is full (`size() == capacity()`), reallocation occurs, doubling the capacity in most implementations.
- `push_back` always copies or moves an existing object; use `emplace_back` to construct in place.

**Constraints and Limitations**
- Reallocation invalidates all iterators, pointers, and references.
- `push_back` cannot be used on a `const` vector.

#### Annotated Code Examples

**Example 1: Appending elements**

```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v;
    v.push_back(10);
    v.push_back(20);
    v.push_back(30);
    for (int x : v) std::cout << x << ' ';
}
```

**Expected Output**
```
10 20 30
```

**Why this output?**  
Each `push_back` appends the argument to the end of the vector.

**Example 2: Reallocation invalidates iterators**

```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {1, 2, 3};
    int* p = v.data();
    v.push_back(4);                   // may reallocate
    // p may now be dangling; using it is undefined behaviour
    std::cout << v.capacity() << '\n';
}
```

**Expected Output** (capacity value is implementation-dependent)
```
6
```

**Why this output?**  
`push_back` may reallocate, invalidating `p`. The capacity grows according to the implementation's strategy.

#### Real-World Cases

- **Reading lines from a file** – `lines.push_back(line);`.
- **Building result sets** – appending computed values.
- **Dynamic stacks** – using `push_back`/`pop_back` as a stack.

---

### 4.2 `pop_back()`

#### Definitions

**Core Definition**
`pop_back()` removes the last element from the vector.

**Technical Definition**
`void pop_back();` removes the last element in O(1) time. The vector’s size decreases by one; capacity is unchanged. Iterators to the removed element are invalidated.

**Beginner-Friendly Explanation**
`pop_back` removes the item at the end of the vector. It is the counterpart to `push_back`.

#### Purposes

- To remove the last element.
- To implement LIFO (stack) behaviour.
- To shrink the vector from the back.
- To discard the most recently added element.

#### Syntax Rules and Structure

**General Syntax**
```cpp
v.pop_back();
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `v` | The vector object. |

**Syntax Rules**
- The vector must not be empty; calling `pop_back()` on an empty vector is undefined behaviour.
- `pop_back()` does not return the removed element; use `back()` before calling it if needed.

**Constraints and Limitations**
- No return value.
- Does not reduce capacity.

#### Annotated Code Examples

**Example 1: Removing the last element**

```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {1, 2, 3};
    v.pop_back();
    for (int x : v) std::cout << x << ' ';
}
```

**Expected Output**
```
1 2
```

**Why this output?**  
`pop_back` removes the last element (`3`).

**Example 2: Checking before popping**

```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v;
    if (!v.empty()) v.pop_back();      // safe
    std::cout << v.size() << '\n';
}
```

**Expected Output**
```
0
```

**Why this output?**  
The empty check prevents undefined behaviour.

#### Real-World Cases

- **Undo/redo stacks** – popping the last action.
- **Parsing** – removing the most recently matched token.
- **Backtracking** – discarding the latest choice.

---

### 4.3 `emplace_back()`

#### Definitions

**Core Definition**
`emplace_back()` constructs a new element directly at the end of the vector, forwarding its arguments to the element’s constructor.

**Technical Definition**
```cpp
template<class... Args>
reference emplace_back(Args&&... args);
```
(since C++11). The element is constructed in place via `std::allocator_traits::construct`, avoiding a temporary object and a copy/move operation.

**Beginner-Friendly Explanation**
`emplace_back` builds the object right inside the vector from the arguments you give it — no temporary is created, which can be faster than `push_back`.

#### Purposes

- To avoid a temporary object and a copy/move.
- To construct non-copyable/non-movable types in place.
- To improve performance when appending complex objects.
- To forward constructor arguments directly.

#### Syntax Rules and Structure

**General Syntax**
```cpp
v.emplace_back(args...);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `args...` | Arguments forwarded to the element’s constructor. |

**Syntax Rules**
- The element type must be *MoveInsertable* for vectors, because reallocation may move existing elements.
- `emplace_back` returns a reference to the inserted element (since C++17).
- No temporary object is created.

**Constraints and Limitations**
- If the element type has an initializer-list constructor, `emplace_back` may select it unexpectedly.
- Overuse of `emplace_back` can make code harder to read when the constructor arguments are not obvious.

#### Annotated Code Examples

**Example 1: `emplace_back` vs. `push_back`**

```cpp
#include <vector>
#include <iostream>
#include <string>

int main() {
    std::vector<std::string> v;
    v.push_back(std::string("hello"));   // creates temporary, then moves
    v.emplace_back("world");             // constructs in place
    for (const auto& s : v) std::cout << s << ' ';
}
```

**Expected Output**
```
hello world
```

**Why this output?**  
`push_back` requires an existing `std::string`; `emplace_back` constructs the string directly from the string literal.

**Example 2: Constructing a complex object**

```cpp
#include <vector>
#include <iostream>

struct Point {
    int x, y;
    Point(int x, int y) : x(x), y(y) {}
};

int main() {
    std::vector<Point> pts;
    pts.emplace_back(1, 2);               // constructs Point(1, 2) in place
    std::cout << pts[0].x << ',' << pts[0].y << '\n';
}
```

**Expected Output**
```
1,2
```

**Why this output?**  
`emplace_back(1, 2)` forwards `1` and `2` to `Point`’s constructor.

#### Real-World Cases

- **Emplacing expensive objects** – avoiding temporary construction.
- **Non-copyable types** – constructing directly in the vector.
- **High-performance code** – reducing allocations and moves.

---

### 4.4 `insert()`

#### Definitions

**Core Definition**
`insert()` inserts one or more elements at a specified position in the vector.

**Technical Definition**
Multiple overloads exist: `insert(const_iterator pos, const T& value)`, `insert(const_iterator pos, T&& value)`, `insert(const_iterator pos, size_type count, const T& value)`, and range insertion. Complexity is O(n) because elements after the insertion point must be shifted.

**Beginner-Friendly Explanation**
`insert` puts an item (or several items) into the middle of the vector at a position you choose. It is slower than adding to the end because the vector may have to move elements out of the way.

#### Purposes

- To insert elements at an arbitrary position.
- To insert multiple copies of a value.
- To insert a range of elements.
- To maintain sorted order when inserting into a sorted vector.

#### Syntax Rules and Structure

**General Syntax**
```cpp
v.insert(pos, value);
v.insert(pos, count, value);
v.insert(pos, first, last);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `pos` | Iterator to the position before which to insert. |
| `value` | The element to insert. |
| `count` | Number of copies to insert. |
| `first, last` | Iterator range to insert. |

**Syntax Rules**
- `pos` must be a valid iterator into the vector.
- Insertion invalidates iterators and references at or after `pos`.
- If reallocation occurs, all iterators are invalidated.

**Constraints and Limitations**
- O(n) time complexity.
- Inserting into the middle of a large vector is expensive.

#### Annotated Code Examples

**Example 1: Inserting a single element**

```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {1, 2, 4};
    v.insert(v.begin() + 2, 3);           // insert 3 before position 2
    for (int x : v) std::cout << x << ' ';
}
```

**Expected Output**
```
1 2 3 4
```

**Why this output?**  
`v.begin() + 2` points to `4`; `insert` places `3` before it.

**Example 2: Inserting multiple copies**

```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {1, 2};
    v.insert(v.end(), 3, 0);              // append three zeros
    for (int x : v) std::cout << x << ' ';
}
```

**Expected Output**
```
1 2 0 0 0
```

**Why this output?**  
`insert` with a count appends three copies of `0` at the end.

#### Real-World Cases

- **Maintaining sorted order** – inserting into a sorted vector at the correct position.
- **Building sequences** – inserting elements at specific indices.
- **Text editing** – inserting characters at a cursor position.

---

### 4.5 `erase()`

#### Definitions

**Core Definition**
`erase()` removes one or more elements from the vector.

**Technical Definition**
`iterator erase(const_iterator pos);` and `iterator erase(const_iterator first, const_iterator last);`. The complexity is O(n) because subsequent elements are shifted. Iterators and references at or after the erased elements are invalidated.

**Beginner-Friendly Explanation**
`erase` deletes an item (or a range of items) from the vector. The remaining elements are moved to fill the gap.

#### Purposes

- To remove a single element at a known position.
- To remove a range of elements.
- To maintain a clean sequence after deletions.
- To support algorithms that remove elements by position.

#### Syntax Rules and Structure

**General Syntax**
```cpp
v.erase(pos);
v.erase(first, last);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `pos` | Iterator to the element to erase. |
| `first, last` | Range of elements to erase. |

**Syntax Rules**
- `pos` must be valid and dereferenceable.
- Erasing does not reduce capacity.
- The returned iterator points to the element after the last erased element.

**Constraints and Limitations**
- All iterators, pointers, and references at or after the erasure point are invalidated.
- Erasing from a vector while iterating requires careful iterator updates.

#### Annotated Code Examples

**Example 1: Erasing a single element**

```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {1, 2, 3, 4};
    v.erase(v.begin() + 1);                // erase 2
    for (int x : v) std::cout << x << ' ';
}
```

**Expected Output**
```
1 3 4
```

**Why this output?**  
`v.begin() + 1` points to `2`; erasing it shifts `3` and `4` left.

**Example 2: Erasing a range**

```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {1, 2, 3, 4, 5};
    v.erase(v.begin() + 1, v.begin() + 3); // erase 2 and 3
    for (int x : v) std::cout << x << ' ';
}
```

**Expected Output**
```
1 4 5
```

**Why this output?**  
The range `[begin+1, begin+3)` covers `2` and `3`; erasing shifts `4` and `5` left.

#### Real-World Cases

- **Removing invalid entries** – erasing elements that fail a condition.
- **Game state management** – removing destroyed entities.
- **Data cleanup** – deleting outdated records.

---

### 4.6 `clear()`

#### Definitions

**Core Definition**
`clear()` removes all elements from the vector, leaving it empty.

**Technical Definition**
`void clear() noexcept;` destroys all elements and sets `size()` to zero. Capacity is unchanged; the storage is not deallocated.

**Beginner-Friendly Explanation**
`clear` empties the vector — all items are removed — but the vector keeps its allocated memory so it can be reused without reallocating.

#### Purposes

- To remove all elements at once.
- To reuse the same vector object for new data.
- To reset a vector to an empty state without freeing capacity.
- To release element resources while keeping storage.

#### Syntax Rules and Structure

**General Syntax**
```cpp
v.clear();
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `v` | The vector object. |

**Syntax Rules**
- All iterators, pointers, and references are invalidated.
- The capacity remains unchanged.
- The vector can be reused immediately.

**Constraints and Limitations**
- Does not free memory; use `shrink_to_fit()` afterwards if memory release is desired.
- Does not affect the vector’s capacity.

#### Annotated Code Examples

**Example 1: Clearing a vector**

```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {1, 2, 3};
    v.clear();
    std::cout << v.size() << ' ' << v.capacity() << '\n';
}
```

**Expected Output** (capacity value is implementation-dependent)
```
0 3
```

**Why this output?**  
`clear` sets size to 0 but leaves capacity unchanged.

**Example 2: Reusing after clear**

```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {1, 2, 3};
    v.clear();
    v.push_back(10);                       // no reallocation needed
    std::cout << v.size() << '\n';
}
```

**Expected Output**
```
1
```

**Why this output?**  
The capacity is still sufficient, so `push_back` does not reallocate.

#### Real-World Cases

- **Frame-based processing** – clearing a buffer each frame.
- **Resetting state** – emptying a vector between test cases.
- **Reusing storage** – avoiding repeated allocations.

---

### 4.7 `resize()`

#### Definitions

**Core Definition**
`resize()` changes the number of elements in the vector.

**Technical Definition**
`void resize(size_type count);` and `void resize(size_type count, const value_type& value);`. If `count > size()`, new elements are appended (default-constructed or copies of `value`). If `count < size()`, excess elements are erased. Capacity may change if growth requires reallocation.

**Beginner-Friendly Explanation**
`resize` changes how many items the vector holds. If you make it larger, new items are added; if smaller, extra items are removed.

#### Purposes

- To set the vector to a specific size.
- To append default-constructed elements.
- To truncate the vector to a smaller size.
- To fill new elements with a specified value.

#### Syntax Rules and Structure

**General Syntax**
```cpp
v.resize(count);
v.resize(count, value);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `count` | The new size. |
| `value` | Value for new elements (optional). |

**Syntax Rules**
- If `count > size()`, elements are appended.
- If `count < size()`, elements are removed.
- If `count > capacity()`, reallocation occurs.
- Iterators may be invalidated if reallocation happens.

**Constraints and Limitations**
- `resize` does not shrink capacity.
- Old iterators may become invalid if reallocation occurs.

#### Annotated Code Examples

**Example 1: Growing a vector**

```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {1, 2, 3};
    v.resize(5);                          // adds two default zeros
    for (int x : v) std::cout << x << ' ';
}
```

**Expected Output**
```
1 2 3 0 0
```

**Why this output?**  
Two new elements are value-initialized to `0`.

**Example 2: Shrinking a vector**

```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {1, 2, 3, 4, 5};
    v.resize(2);                          // keeps first two
    for (int x : v) std::cout << x << ' ';
}
```

**Expected Output**
```
1 2
```

**Why this output?**  
Elements after the second are erased.

#### Real-World Cases

- **Fixed-size buffers** – resizing to a required size.
- **Simulation grids** – growing or shrinking the grid.
- **Data padding** – adding default elements.

---

### 4.8 `reserve()`

#### Definitions

**Core Definition**
`reserve()` requests that the vector’s capacity be at least a specified value, preventing reallocation until the size exceeds that capacity.

**Technical Definition**
`void reserve(size_type new_cap);` increases the capacity to at least `new_cap`. If `new_cap > capacity()`, reallocation occurs; otherwise, no change. Iterators are invalidated only if reallocation happens.

**Beginner-Friendly Explanation**
`reserve` tells the vector, “Make room for at least this many items now,” so that subsequent `push_back` calls do not need to reallocate.

#### Purposes

- To avoid reallocation during repeated insertions.
- To improve performance by pre-allocating memory.
- To control capacity growth.
- To keep iterators stable during a known number of insertions.

#### Syntax Rules and Structure

**General Syntax**
```cpp
v.reserve(new_cap);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `new_cap` | The requested minimum capacity. |

**Syntax Rules**
- `reserve` does not change `size()`.
- If `new_cap < size()`, the request is ignored (no effect) in most implementations.
- Reallocation invalidates all iterators if capacity changes.

**Constraints and Limitations**
- Non-binding in the sense that the allocator may fail, but the standard requires capacity to be at least `new_cap` if the operation succeeds.
- Does not shrink capacity.

#### Annotated Code Examples

**Example 1: Reserving capacity**

```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v;
    v.reserve(100);
    for (int i = 0; i < 100; ++i) v.push_back(i);
    std::cout << v.size() << ' ' << v.capacity() << '\n';
}
```

**Expected Output** (capacity at least 100)
```
100 100
```

**Why this output?**  
`reserve(100)` ensures capacity for 100 elements; no reallocation occurs during the loop.

**Example 2: Reserve prevents iterator invalidation**

```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v;
    v.reserve(3);
    v.push_back(1);
    auto it = v.begin();
    v.push_back(2);                        // no reallocation
    v.push_back(3);                        // no reallocation
    std::cout << *it << '\n';              // still valid
}
```

**Expected Output**
```
1
```

**Why this output?**  
Because capacity was reserved, the iterators remain valid.

#### Real-World Cases

- **Building large vectors** – reserving to avoid repeated reallocations.
- **Performance-critical code** – pre-allocating known sizes.
- **Stable iterator requirements** – reserving before taking iterators.

---

### 4.9 `shrink_to_fit()`

#### Definitions

**Core Definition**
`shrink_to_fit()` requests that the vector reduce its capacity to match its size, releasing unused memory.

**Technical Definition**
`void shrink_to_fit();` is a **non-binding** request to reduce `capacity()` to `size()`. The implementation may or may not fulfill the request. If reallocation occurs, all iterators, pointers, and references are invalidated.

**Beginner-Friendly Explanation**
`shrink_to_fit` asks the vector to give back memory it isn’t using. It is only a request — the compiler/library may ignore it.

#### Purposes

- To release unused memory after elements have been removed.
- To reduce memory footprint.
- To make the vector’s capacity match its size.
- To optimise memory usage in long-running applications.

#### Syntax Rules and Structure

**General Syntax**
```cpp
v.shrink_to_fit();
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `v` | The vector object. |

**Syntax Rules**
- Non-binding; the implementation may choose not to shrink.
- If reallocation occurs, all iterators are invalidated.
- Complexity is at most linear in `size()`.

**Constraints and Limitations**
- Not guaranteed to have any effect.
- May be expensive if it does reallocate.

#### Annotated Code Examples

**Example 1: Shrinking after clear**

```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v(1000);
    v.clear();
    v.shrink_to_fit();
    std::cout << v.capacity() << '\n';     // may be 0
}
```

**Expected Output** (implementation-dependent)
```
0
```

**Why this output?**  
The implementation may honour the request and reduce capacity to 0.

**Example 2: After resize**

```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v(1000);
    v.resize(10);
    v.shrink_to_fit();
    std::cout << v.size() << ' ' << v.capacity() << '\n';
}
```

**Expected Output** (capacity may equal size)
```
10 10
```

**Why this output?**  
`shrink_to_fit` may reduce capacity to 10.

#### Real-World Cases

- **Memory-sensitive applications** – embedded systems, mobile devices.
- **Long-running servers** – releasing memory after large temporary vectors.
- **Post-processing** – freeing memory after filtering a large dataset.

---

### 4.10 `at()`

#### Definitions

**Core Definition**
`at()` provides bounds-checked access to an element at a specified index.

**Technical Definition**
`reference at(size_type pos);` and `const_reference at(size_type pos) const;`. Throws `std::out_of_range` if `pos >= size()`.

**Beginner-Friendly Explanation**
`at()` is like `operator[]`, but it checks the index and throws an exception if you go out of bounds, making it safer.

#### Purposes

- To safely access elements with bounds checking.
- To catch out-of-range errors at runtime.
- To provide a safer alternative to `operator[]`.
- To satisfy safety-critical coding standards.

#### Syntax Rules and Structure

**General Syntax**
```cpp
v.at(index);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `index` | Zero-based position of the element. |

**Syntax Rules**
- Throws `std::out_of_range` if `index >= size()`.
- Complexity O(1).
- Can be used on `const` vectors.

**Constraints and Limitations**
- Slightly slower than `operator[]` due to bounds checking.
- Requires exception handling for out-of-range access.

#### Annotated Code Examples

**Example 1: Safe access**

```cpp
#include <vector>
#include <iostream>
#include <stdexcept>

int main() {
    std::vector<int> v = {10, 20, 30};
    try {
        std::cout << v.at(1) << '\n';      // 20
        std::cout << v.at(5) << '\n';      // throws
    } catch (const std::out_of_range& e) {
        std::cout << "Out of range: " << e.what() << '\n';
    }
}
```

**Expected Output**
```
20
Out of range: vector::_M_range_check: __n (which is 5) >= this->size() (which is 3)
```

**Why this output?**  
`at(1)` returns `20`; `at(5)` throws because 5 is out of bounds.

**Example 2: Const access**

```cpp
#include <vector>
#include <iostream>

int main() {
    const std::vector<int> v = {1, 2, 3};
    std::cout << v.at(2) << '\n';
}
```

**Expected Output**
```
3
```

**Why this output?**  
`at` works on const vectors and returns a const reference.

#### Real-World Cases

- **User input validation** – safely indexing with user-provided indices.
- **Safety-critical systems** – avoiding undefined behaviour.
- **Debug builds** – bounds-checking during development.

---

### 4.11 `operator[]`

#### Definitions

**Core Definition**
`operator[]` provides unchecked access to an element at a specified index.

**Technical Definition**
`reference operator[](size_type pos);` and `const_reference operator[](size_type pos) const;`. No bounds checking is performed; accessing an out-of-range index is undefined behaviour.

**Beginner-Friendly Explanation**
`operator[]` is the fastest way to access an element by index — but it does not check whether the index is valid, so you must be careful.

#### Purposes

- To provide fast, unchecked element access.
- To enable array-like syntax.
- To support high-performance indexing.
- To allow direct modification of elements.

#### Syntax Rules and Structure

**General Syntax**
```cpp
v[index];
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `index` | Zero-based position. |

**Syntax Rules**
- No bounds checking.
- Undefined behaviour if `index >= size()`.
- Complexity O(1).

**Constraints and Limitations**
- Unsafe for untrusted indices.
- Cannot be used on a `const` vector for modification.

#### Annotated Code Examples

**Example 1: Fast access**

```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {10, 20, 30};
    std::cout << v[0] << ' ' << v[2] << '\n';
    v[1] = 99;
    std::cout << v[1] << '\n';
}
```

**Expected Output**
```
10 30
99
```

**Why this output?**  
`operator[]` accesses and modifies elements directly.

**Example 2: Undefined behaviour**

```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {1, 2, 3};
    std::cout << v[5] << '\n';             // undefined behaviour
}
```

**Expected Output**  
(Undefined behaviour; may crash or print garbage.)

**Why this output?**  
No bounds check is performed, so accessing index 5 is undefined.

#### Real-World Cases

- **Performance-critical loops** – indexing without bounds checks.
- **Mathematical code** – matrix/vector element access.
- **Known-valid indices** – after explicit size checks.

---

### 4.12 `front()`

#### Definitions

**Core Definition**
`front()` returns a reference to the first element of the vector.

**Technical Definition**
`reference front();` and `const_reference front() const;`. Undefined behaviour if the vector is empty.

**Beginner-Friendly Explanation**
`front()` gives you the first item in the vector.

#### Purposes

- To access the first element directly.
- To modify the first element.
- To read the first element without copying.
- To implement queue-like behaviour (with `pop_front` emulated).

#### Syntax Rules and Structure

**General Syntax**
```cpp
v.front();
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `v` | The vector object. |

**Syntax Rules**
- Undefined behaviour if `v.empty()`.
- Returns a reference, so it can be used to modify the element.

**Constraints and Limitations**
- Not safe on empty vectors.

#### Annotated Code Examples

**Example 1: Accessing the first element**

```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {10, 20, 30};
    std::cout << v.front() << '\n';        // 10
    v.front() = 99;
    std::cout << v[0] << '\n';            // 99
}
```

**Expected Output**
```
10
99
```

**Why this output?**  
`front()` returns a reference to the first element, allowing modification.

#### Real-World Cases

- **Queue front access** – reading the next item.
- **State inspection** – checking the first element.

---

### 4.13 `back()`

#### Definitions

**Core Definition**
`back()` returns a reference to the last element of the vector.

**Technical Definition**
`reference back();` and `const_reference back() const;`. Undefined behaviour if the vector is empty.

**Beginner-Friendly Explanation**
`back()` gives you the last item in the vector.

#### Purposes

- To access the last element directly.
- To modify the last element.
- To read the most recently added element.
- To implement stack-like `top()`.

#### Syntax Rules and Structure

**General Syntax**
```cpp
v.back();
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `v` | The vector object. |

**Syntax Rules**
- Undefined behaviour if `v.empty()`.
- Returns a reference.

**Constraints and Limitations**
- Not safe on empty vectors.

#### Annotated Code Examples

**Example 1: Accessing the last element**

```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {10, 20, 30};
    std::cout << v.back() << '\n';         // 30
    v.back() = 99;
    std::cout << v[2] << '\n';            // 99
}
```

**Expected Output**
```
30
99
```

**Why this output?**  
`back()` returns a reference to the last element.

#### Real-World Cases

- **Stack top** – reading the most recent item.
- **Last-inspection** – checking the final element.

---

### 4.14 `data()`

#### Definitions

**Core Definition**
`data()` returns a pointer to the underlying contiguous array of the vector.

**Technical Definition**
`T* data() noexcept;` and `const T* data() const noexcept;`. The pointer points to the first element, or may be null if the vector is empty.

**Beginner-Friendly Explanation**
`data()` gives you a raw pointer to the vector’s internal array, which is useful for passing to C functions.

#### Purposes

- To access the underlying array for C interoperability.
- To pass vector data to legacy APIs.
- To perform pointer arithmetic on the contiguous storage.
- To obtain a raw view of the elements.

#### Syntax Rules and Structure

**General Syntax**
```cpp
T* ptr = v.data();
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `v` | The vector object. |

**Syntax Rules**
- Returns a pointer to the first element.
- The pointer is invalidated by operations that reallocate.
- For an empty vector, the returned pointer may be null (implementation-dependent).

**Constraints and Limitations**
- The pointer must not be used after reallocation.
- The pointer must not be freed by the caller.

#### Annotated Code Examples

**Example 1: Passing data to a C function**

```cpp
#include <vector>
#include <iostream>
#include <cstring>

void print_c_array(const int* arr, int n) {
    for (int i = 0; i < n; ++i) std::cout << arr[i] << ' ';
}

int main() {
    std::vector<int> v = {1, 2, 3};
    print_c_array(v.data(), v.size());
}
```

**Expected Output**
```
1 2 3
```

**Why this output?**  
`data()` provides a pointer to the contiguous elements.

**Example 2: Modifying through `data()`**

```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {1, 2, 3};
    int* p = v.data();
    p[1] = 99;
    std::cout << v[1] << '\n';
}
```

**Expected Output**
```
99
```

**Why this output?**  
The pointer modifies the underlying storage directly.

#### Real-World Cases

- **C API integration** – passing vector data to `memcpy`, `printf`, etc.
- **Graphics programming** – uploading vertex data.
- **Network protocols** – sending contiguous buffers.

---

### 4.15 `size()`

#### Definitions

**Core Definition**
`size()` returns the number of elements currently stored in the vector.

**Technical Definition**
`size_type size() const noexcept;`. Returns the number of elements, which is always ≤ `capacity()`.

**Beginner-Friendly Explanation**
`size()` tells you how many items are in the vector right now.

#### Purposes

- To determine the number of elements.
- To loop over elements safely.
- To check whether the vector is empty.
- To compute indices.

#### Syntax Rules and Structure

**General Syntax**
```cpp
v.size();
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `v` | The vector object. |

**Syntax Rules**
- Complexity O(1).
- Returns a `size_type` (usually `std::size_t`).

**Constraints and Limitations**
- `size()` may be less than `capacity()`.

#### Annotated Code Examples

**Example 1: Using `size()` in a loop**

```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {1, 2, 3};
    for (std::size_t i = 0; i < v.size(); ++i)
        std::cout << v[i] << ' ';
}
```

**Expected Output**
```
1 2 3
```

**Why this output?**  
`size()` returns 3, so the loop runs three times.

#### Real-World Cases

- **Loop bounds** – iterating over elements.
- **Validation** – checking minimum size requirements.
- **Logging** – reporting collection size.

---

### 4.16 `capacity()`

#### Definitions

**Core Definition**
`capacity()` returns the number of elements the vector can hold without reallocating.

**Technical Definition**
`size_type capacity() const noexcept;`. The capacity is always ≥ `size()`. When `size()` exceeds `capacity()`, the vector reallocates.

**Beginner-Friendly Explanation**
`capacity()` tells you how many items the vector can hold before it needs to allocate more memory.

#### Purposes

- To understand the vector’s allocated storage.
- To decide when to call `reserve()`.
- To avoid unexpected reallocations.
- To reason about iterator invalidation.

#### Syntax Rules and Structure

**General Syntax**
```cpp
v.capacity();
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `v` | The vector object. |

**Syntax Rules**
- Complexity O(1).
- Capacity grows automatically (often doubling).
- Capacity never decreases automatically.

**Constraints and Limitations**
- `capacity()` may be larger than `size()`.
- Does not include unused memory beyond capacity.

#### Annotated Code Examples

**Example 1: Capacity growth**

```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v;
    std::cout << v.capacity() << '\n';     // 0
    v.push_back(1);
    std::cout << v.capacity() << '\n';     // 1
    v.push_back(2);
    std::cout << v.capacity() << '\n';     // 2 or more
}
```

**Expected Output** (implementation-dependent)
```
0
1
2
```

**Why this output?**  
Capacity starts at 0 and grows as elements are added.

#### Real-World Cases

- **Memory planning** – estimating allocated memory.
- **Performance tuning** – deciding whether to `reserve`.
- **Debugging** – understanding reallocation behaviour.

---

### 4.17 `empty()`

#### Definitions

**Core Definition**
`empty()` returns `true` if the vector contains no elements.

**Technical Definition**
`bool empty() const noexcept;`. Equivalent to `size() == 0`, but may be more efficient.

**Beginner-Friendly Explanation**
`empty()` checks whether the vector is empty.

#### Purposes

- To check whether the vector has any elements.
- To guard operations that require a non-empty vector.
- To control loops and conditions.
- To provide a readable alternative to `size() == 0`.

#### Syntax Rules and Structure

**General Syntax**
```cpp
if (v.empty()) { ... }
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `v` | The vector object. |

**Syntax Rules**
- Returns `true` if `size() == 0`.
- Complexity O(1).

**Constraints and Limitations**
- Does not check capacity.

#### Annotated Code Examples

**Example 1: Checking emptiness**

```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v;
    std::cout << std::boolalpha << v.empty() << '\n'; // true
    v.push_back(1);
    std::cout << v.empty() << '\n';        // false
}
```

**Expected Output**
```
true
false
```

**Why this output?**  
`empty()` returns `true` when size is 0.

#### Real-World Cases

- **Guarding `pop_back()`** – avoid undefined behaviour.
- **Loop conditions** – `while (!v.empty())`.
- **Validation** – checking required data.

---

## 5. Vector Iteration

### Definitions

**Core Definition**
Vector iteration is the process of traversing the elements of a vector, typically using iterators or range-based `for`.

**Technical Definition**
`std::vector` provides random-access iterators (`begin()`, `end()`, `cbegin()`, `cend()`, `rbegin()`, `rend()`). Iterators are invalidated by operations that reallocate or modify the vector beyond the iteration point.

**Beginner-Friendly Explanation**
Iteration means going through each item in the vector one by one. You can use a `for` loop with an index, iterators, or the modern range-based `for`.

### Purposes

- To process every element.
- To search for elements.
- To transform elements.
- To accumulate results.
- To work with STL algorithms.

### Syntax Rules and Structure

**General Syntax**
```cpp
// Index-based
for (std::size_t i = 0; i < v.size(); ++i) { ... }

// Iterator-based
for (auto it = v.begin(); it != v.end(); ++it) { ... }

// Range-based for
for (const auto& elem : v) { ... }
```

**Component Breakdown**

| Method | Description |
|--------|-------------|
| Index | Uses `operator[]` with an index. |
| Iterator | Uses `begin()`/`end()` and `++`. |
| Range-based `for` | Syntactic sugar for iterator loop. |

**Syntax Rules**
- `begin()` points to the first element; `end()` points past the last.
- Iterators are invalidated by reallocation and by insertion/erasure.
- Range-based `for` is preferred for readability.

**Constraints and Limitations**
- Modifying the vector during iteration can invalidate iterators.
- `const_iterator` prevents modification.

### Annotated Code Examples

**Example 1: Range-based for**

```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {1, 2, 3};
    for (int x : v) std::cout << x << ' ';
}
```

**Expected Output**
```
1 2 3
```

**Why this output?**  
The range-based `for` iterates over each element in order.

**Example 2: Iterator-based modification**

```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {1, 2, 3};
    for (auto it = v.begin(); it != v.end(); ++it)
        *it *= 2;
    for (int x : v) std::cout << x << ' ';
}
```

**Expected Output**
```
2 4 6
```

**Why this output?**  
Each element is doubled through the iterator.

#### Real-World Cases

- **Data processing** – applying a function to each element.
- **Searching** – `std::find` uses iterators.
- **Transformation** – `std::transform`.

---

## 6. Vector Performance

### Definitions

**Core Definition**
Vector performance refers to the time and space complexity characteristics of `std::vector` operations.

**Technical Definition**
The standard mandates specific complexity guarantees: random access O(1), `push_back` amortized O(1), `insert`/`erase` O(n), `size`/`capacity`/`empty` O(1). Reallocation invalidates iterators but preserves element values (via move or copy).

**Beginner-Friendly Explanation**
Vectors are fast at adding to the end and accessing by index, but slower at inserting or removing in the middle. They use contiguous memory, which is cache-friendly.

### Complexity Table

| Operation | Complexity | Notes |
|-----------|------------|-------|
| `operator[]`, `at` | O(1) | Random access |
| `front`, `back` | O(1) | |
| `push_back` | Amortized O(1) | May reallocate |
| `pop_back` | O(1) | |
| `insert`, `erase` | O(n) | Shifts elements |
| `clear` | O(n) | Destroys elements |
| `resize` | O(n) | May reallocate |
| `reserve` | O(n) if reallocation | |
| `shrink_to_fit` | O(n) if reallocation | Non-binding |
| `size`, `capacity`, `empty` | O(1) | |

### Key Performance Factors

- **Contiguous storage** – excellent cache locality and vectorisation friendliness.
- **Reallocation** – geometric growth strategy (often doubling) gives amortized O(1) `push_back`.
- **Insertion/erasure in the middle** – O(n) because elements must be shifted.
- **`reserve`** – eliminates reallocation for a known number of insertions.
- **`emplace_back`** – avoids temporary objects and copies/moves.

### Real-World Cases

- **High-performance computing** – vectors are preferred for numeric arrays.
- **Game engines** – contiguous storage for vertex buffers.
- **Real-time systems** – predictable O(1) access.

---

## 7. Vector Best Practices

### Definitions

**Core Definition**
Vector best practices are guidelines for using `std::vector` correctly, safely, and efficiently.

**Technical Definition**
These practices are codified in the C++ Core Guidelines (SL.con.1, SL.con.2) and community consensus: prefer `vector` by default, use `reserve` when size is known, prefer `emplace_back` for construction, be aware of iterator invalidation, and avoid `std::vector<bool>` pitfalls.

**Beginner-Friendly Explanation**
Best practices are good habits: use `vector` as your default container, reserve memory when you can, use `emplace_back` for efficiency, and be careful with iterators after modifying the vector.

### Key Guidelines

| Guideline | Summary |
|-----------|---------|
| **SL.con.2** | Prefer `vector` by default unless you have a reason to use another container. |
| **SL.con.1** | Prefer `vector` over C arrays for variable-length data. |
| **`reserve` when size is known** | Avoid reallocations by reserving capacity upfront. |
| **Prefer `emplace_back`** | Construct elements in place to avoid temporaries. |
| **Be aware of iterator invalidation** | Reallocation invalidates all iterators; insertion/erasure invalidates at or after the point. |
| **Use `at()` for safety** | When bounds checking is required. |
| **Avoid `std::vector<bool>`** | It is a bitset-like specialisation with non-standard reference semantics. |
| **Pass by `const&`** | Avoid copying vectors into functions. |
| **Use range-based `for`** | More readable than index or iterator loops. |
| **Use `data()` for C interoperability** | But ensure the pointer is used before any reallocation. |

### Annotated Code Example

```cpp
#include <vector>
#include <iostream>

void process(const std::vector<int>& v) {   // const reference: no copy
    for (int x : v) std::cout << x << ' ';
}

int main() {
    std::vector<int> v;
    v.reserve(10);                           // pre-allocate
    for (int i = 0; i < 10; ++i)
        v.emplace_back(i * i);               // in-place construction
    process(v);
}
```

**Expected Output**
```
0 1 4 9 16 25 36 49 64 81
```

**Why this output?**  
`reserve` avoids reallocations; `emplace_back` constructs each square in place; `process` receives a const reference to avoid copying.

### Real-World Cases

- **Code reviews** – checking for common vector pitfalls.
- **Static analysis** – tools like clang-tidy enforce Core Guidelines.
- **Library design** – interfaces returning/passing vectors safely.

---

## References

- `std::vector` — cppreference – https://en.cppreference.com/w/cpp/container/vector
- `std::vector::push_back` — cppreference – https://en.cppreference.com/w/cpp/container/vector/push_back
- `std::vector::emplace_back` — cppreference – https://en.cppreference.com/w/cpp/container/vector/emplace_back
- `std::vector::shrink_to_fit` — cppreference – https://en.cppreference.com/w/cpp/container/vector/shrink_to_fit
- Iterator invalidation — cppreference – https://en.cppreference.com/w/cpp/container/vector#Iterator_invalidation
- C++ Core Guidelines — SL.con: Containers – https://cpp-core-guidelines-docs.vercel.app/stdlib
- SEI CERT C++ Coding Standard — CTR51-CPP – https://wiki.sei.cmu.edu/confluence/display/cplusplus/CTR51-CPP.+Use+valid+references%2C+pointers%2C+and+iterators+to+reference+elements+of+a+container
- C++ Working Draft (N4950) — Vector – https://eel.is/c++draft/vector

---

This cheat sheet provides a comprehensive, structured reference for `std::vector`, covering every listed topic with definitions, purposes, syntax rules, annotated code examples, expected outputs, real-world cases, performance analysis, and best practices, all grounded in authoritative sources.