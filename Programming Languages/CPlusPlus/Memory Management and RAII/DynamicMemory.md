# C++ Dynamic Memory — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Dynamic memory management is the process by which a C++ program explicitly requests memory from the free store (heap) at runtime and releases it when it is no longer needed.

**Technical Definition**
C++ provides two families of dynamic memory facilities: the **language-level** `new`/`delete` expressions (which both allocate memory *and* construct/destroy objects) and the **C-style** `malloc`/`free`/`calloc`/`realloc` functions (which only manage raw memory). The `new`-expression is the only way to create an object or array of objects with dynamic storage duration—lifetime not restricted to the scope in which it is created. The `delete`-expression destroys the object(s) and calls the deallocation function. `malloc` allocates `size` bytes of **uninitialized** storage, while `calloc` allocates and zero-initializes, and `realloc` resizes an existing block.

**Beginner-Friendly Explanation**
Dynamic memory is “memory on demand.” Instead of deciding at compile time how big an array is, you ask the operating system at runtime for exactly the amount you need. When you are done, you must give it back. C++ gives you two toolkits for this: `new`/`delete` (which also run constructors and destructors) and `malloc`/`free` (which only hand you raw bytes).

### Key Characteristics

- **Runtime-sized** – allocation size can be determined while the program is running.
- **Lifetime control** – dynamically allocated objects live until explicitly destroyed (or the program ends), unlike automatic objects.
- **Dual responsibility** – `new`/`delete` manage both memory and object lifetime; `malloc`/`free` manage only memory.
- **Matching requirement** – every allocation must be matched with the correct form of deallocation (`new`↔`delete`, `new[]`↔`delete[]`, `malloc`↔`free`).
- **Error-prone by nature** – leaks, double frees, use-after-free, and mismatched forms are all undefined behaviour.
- **Modern replacement** – smart pointers and standard containers wrap dynamic memory in RAII, eliminating most manual calls.

### Prerequisites

- Basic C++ syntax (variables, pointers, references).
- Understanding of stack vs. heap memory.
- Familiarity with constructors and destructors.
- Knowledge of object lifetime and storage duration.
- Awareness of undefined behaviour and exception safety.

### Related Programming Areas

- **Object lifetime** – dynamic storage duration is one of the four storage durations.
- **RAII and smart pointers** – the modern, safe alternative to raw `new`/`delete`.
- **Exception safety** – dynamic memory must be released even when exceptions are thrown.
- **Allocators and memory pools** – custom strategies for reducing fragmentation.
- **C interoperability** – `malloc`/`free` are the C allocation primitives.

### Core Concepts / Features

1. `new`
2. `delete`
3. `new[]` / `delete[]`
4. Placement new
5. `operator new` / `operator delete`
6. `malloc` / `free`
7. `calloc` / `realloc`
8. Memory leaks
9. Memory fragmentation
10. Dynamic memory best practices

---

## 1. `new`

### Definitions

**Core Definition**
`new` is a C++ expression that allocates storage from the free store and constructs an object (or array of objects) in that storage, returning a pointer to the constructed object.

**Technical Definition**
A `new`-expression first calls an allocation function (`operator new`) to obtain raw storage, then initializes the object(s) in that storage. If initialization throws, the storage is automatically deallocated by calling the corresponding `operator delete`. The result is a prvalue pointer to the constructed object (or to the first element of a constructed array). The default `operator new` throws `std::bad_alloc` on failure unless the `nothrow` form is used.

**Beginner-Friendly Explanation**
`new` is the C++ way of saying “give me memory for a new object.” It does two things: it reserves the memory, and it constructs the object in that memory. If construction fails (throws), the memory is automatically returned—you do not need to clean up.

### Purposes

- To create a single object with a lifetime that outlives the current scope.
- To allocate memory whose size is determined at runtime.
- To construct objects that are too large for the stack.
- To create objects that must persist beyond the function that created them.

### Syntax Rules and Structure

**General Syntax**
```cpp
new type;                           // default-initialized
new type(initializer);              // direct-initialized
new type{initializer};              // list-initialized (C++11)
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `new` | Keyword introducing the new-expression. |
| `type` | The type of object to create. |
| `initializer` | Optional arguments for the constructor. |

**Syntax Rules**
- The allocated type may be an array type, a class template with deduced arguments (C++17), or a placeholder type (`auto`).
- The `new`-expression returns a pointer of the allocated type.
- If allocation fails, `std::bad_alloc` is thrown (default form) or `nullptr` is returned (`nothrow` form).

**Constraints and Limitations**
- Every `new` must be matched by exactly one `delete` (or `delete[]` for arrays).
- The allocated object has dynamic storage duration and no automatic cleanup.

### Annotated Code Examples

**Example 1: Allocating a single object**

```cpp
#include <iostream>

int main() {
    int* p = new int(42);             // allocate and initialize to 42
    std::cout << *p << '\n';          // 42
    delete p;                         // release memory
}
```

**Expected Output**
```
42
```

**Why this output?**  
`new int(42)` allocates space for one `int` and initializes it to `42`; `delete p` destroys the object and releases the memory.

**Example 2: Allocating a class object**

```cpp
#include <iostream>
#include <string>

struct Widget {
    std::string name;
    Widget(const std::string& n) : name(n) {
        std::cout << "Constructed " << name << '\n';
    }
    ~Widget() { std::cout << "Destroyed " << name << '\n'; }
};

int main() {
    Widget* w = new Widget("alpha");
    delete w;
}
```

**Expected Output**
```
Constructed alpha
Destroyed alpha
```

**Why this output?**  
`new Widget("alpha")` allocates memory, then calls the constructor. `delete w` calls the destructor, then releases memory.

### Real-World Cases

- **Polymorphic objects** – `Base* p = new Derived();` to use runtime polymorphism.
- **Long-lived data** – objects that must survive the function that created them.
- **Large objects** – avoiding stack overflow for large local arrays.

---

## 2. `delete`

### Definitions

**Core Definition**
`delete` is a C++ expression that destroys an object previously created by a `new`-expression and releases the associated memory.

**Technical Definition**
A `delete`-expression first calls the destructor of the object (if it has one), then calls the deallocation function (`operator delete`) to release the storage. For a single object, the non-array form `delete` must be used; using `delete[]` on a non-array or `delete` on an array is undefined behaviour.

**Beginner-Friendly Explanation**
`delete` is the counterpart to `new`. It tells C++ “I’m done with this object”—it runs the destructor (if any) and gives the memory back to the system.

### Purposes

- To destroy an object created by `new` and release its memory.
- To run the destructor of a dynamically allocated class object.
- To prevent memory leaks.

### Syntax Rules and Structure

**General Syntax**
```cpp
delete pointer;                    // single object
delete[] pointer;                 // array
```

**Component Breakdown**

| Form | Destroys |
|------|----------|
| `delete p` | A single object created by `new`. |
| `delete[] p` | An array created by `new[]`. |

**Syntax Rules**
- The pointer must point to an object created by the matching form of `new`.
- `delete nullptr` is a no-op (safe).
- Deleting the same pointer twice is undefined behaviour (double free).

**Constraints and Limitations**
- Must match the allocation form exactly.
- Must not be used on memory obtained from `malloc`/`calloc`/`realloc`.

### Annotated Code Examples

**Example 1: Deleting a single object**

```cpp
#include <iostream>

int main() {
    int* p = new int(10);
    std::cout << *p << '\n';          // 10
    delete p;                         // destroy and free
    // *p is now dangling; do not use
}
```

**Expected Output**
```
10
```

**Why this output?**  
`delete p` destroys the `int` object and releases its memory.

**Example 2: Double delete (undefined behaviour)**

```cpp
#include <iostream>

int main() {
    int* p = new int(5);
    delete p;
    delete p;                         // UNDEFINED BEHAVIOUR
}
```

**Expected Output**  
(Undefined behaviour; may crash or corrupt the heap.)

**Why this output?**  
The second `delete` operates on a pointer whose object has already been destroyed; this is undefined behaviour.

### Real-World Cases

- **RAII wrappers** – smart pointers call `delete` automatically in their destructors.
- **Manual resource management** – when smart pointers are not used.

---

## 3. `new[]` / `delete[]`

### Definitions

**Core Definition**
`new[]` allocates an array of objects with dynamic storage duration; `delete[]` destroys every element of that array and releases the memory.

**Technical Definition**
A `new[]`-expression allocates storage for an array and default-constructs (or list-initializes) each element. The compiler records the number of elements so that `delete[]` can call the correct number of destructors. Using the wrong form of `delete` (e.g., `delete` instead of `delete[]`) results in undefined behaviour, typically because only the first element's destructor is called and the allocation bookkeeping is corrupted.

**Beginner-Friendly Explanation**
`new[]` creates an entire array at once, and `delete[]` destroys the whole array. You must use `delete[]`—not plain `delete`—when you used `new[]`.

### Purposes

- To allocate an array whose size is determined at runtime.
- To create a dynamically sized collection of objects with constructors.
- To manage arrays of class objects whose destructors must all run.

### Syntax Rules and Structure

**General Syntax**
```cpp
T* arr = new T[n];                // array of n default-constructed elements
T* arr = new T[n]{init};          // array with initializer (C++11)
delete[] arr;                     // destroy all elements and free memory
```

**Component Breakdown**

| Form | Meaning |
|------|---------|
| `new T[n]` | Allocates `n` elements. |
| `new T[]{...}` | Deduces `n` from the initializer (C++11). |
| `delete[] p` | Destroys every element and frees memory. |

**Syntax Rules**
- `n` must be a valid size (for arrays of known bound, dimensions other than the first must be constant expressions).
- For class types, the default constructor must be accessible.
- The array form of `delete` must be used.

**Constraints and Limitations**
- Cannot be used with non-default-constructible types unless initialized.
- Overhead for storing the element count is implementation-defined.

### Annotated Code Examples

**Example 1: Dynamic array of integers**

```cpp
#include <iostream>

int main() {
    int* arr = new int[5]{1, 2, 3, 4, 5};
    for (int i = 0; i < 5; ++i)
        std::cout << arr[i] << ' ';
    delete[] arr;
}
```

**Expected Output**
```
1 2 3 4 5
```

**Why this output?**  
`new int[5]{...}` allocates five integers and initializes them; `delete[]` destroys and frees the whole array.

**Example 2: Mismatched delete (undefined behaviour)**

```cpp
int* arr = new int[10];
delete arr;                       // WRONG: should be delete[]
```

**Expected Output**  
(Undefined behaviour; likely heap corruption.)

**Why this output?**  
Using `delete` instead of `delete[]` fails to destroy all elements and corrupts the allocator's bookkeeping.

### Real-World Cases

- **Buffer pools** – dynamically sized byte arrays.
- **Legacy APIs** – functions returning `new[]`-allocated arrays.

---

## 4. Placement New

### Definitions

**Core Definition**
Placement new constructs an object at a pre-existing, caller-supplied memory address without allocating new storage.

**Technical Definition**
The standard placement form is `void* operator new(std::size_t, void* ptr)`, which simply returns `ptr`. The expression `new (ptr) T(args...)` constructs a `T` object at `ptr` using the provided arguments. Because no memory is allocated, there is no matching placement `delete`; the programmer must explicitly call the destructor (`p->~T()`) when the object's lifetime should end.

**Beginner-Friendly Explanation**
Placement new lets you say “build this object right here, in this memory I already have.” It is useful for custom allocators, memory pools, and reusing buffers—but it comes with manual destructor responsibility.

### Purposes

- To construct an object in a pre-allocated buffer or memory pool.
- To separate memory allocation from object construction.
- To implement custom containers and allocators.
- To reuse storage for objects of different types.

### Syntax Rules and Structure

**General Syntax**
```cpp
new (address) Type(args...);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `address` | A `void*` pointing to sufficient, suitably aligned storage. |
| `Type(args...)` | The object to construct. |

**Syntax Rules**
- The buffer must be large enough and correctly aligned.
- The object's lifetime begins after placement new completes.
- Destruction requires an explicit destructor call.

**Constraints and Limitations**
- No automatic deallocation; memory is the caller's responsibility.
- Cannot use `delete` on a placement-new object; use `p->~T()`.

### Annotated Code Example

```cpp
#include <iostream>
#include <new>

struct Widget {
    int value;
    Widget(int v) : value(v) { std::cout << "Constructed\n"; }
    ~Widget() { std::cout << "Destroyed\n"; }
};

int main() {
    alignas(Widget) char buffer[sizeof(Widget)];   // raw storage
    Widget* w = new (buffer) Widget(42);            // placement new
    std::cout << w->value << '\n';                  // 42
    w->~Widget();                                   // explicit destructor
}
```

**Expected Output**
```
Constructed
42
Destroyed
```

**Why this output?**  
Placement new constructs `Widget` inside `buffer`; the destructor is called explicitly because `delete` would attempt to free non-heap memory.

### Real-World Cases

- **Memory pools** – constructing objects in pre-allocated blocks.
- **`std::vector` internals** – constructing elements in raw capacity.
- **Optional/variant implementations** – managing in-place storage.

---

## 5. `operator new` / `operator delete`

### Definitions

**Core Definition**
`operator new` and `operator delete` are the low-level allocation and deallocation functions that `new`- and `delete`-expressions call to obtain and release raw memory.

**Technical Definition**
`operator new(std::size_t)` allocates `count` bytes and returns a `void*`; `operator delete(void*)` releases the block. These functions can be replaced globally or overloaded per class (for `new`) to customize allocation behaviour. The default `operator new` throws `std::bad_alloc` on failure; the `nothrow` overload returns `nullptr` instead. Replacing them is a common technique for debugging, leak detection, and custom allocators.

**Beginner-Friendly Explanation**
`new` is the expression; `operator new` is the function that actually does the memory grabbing. You can replace it with your own version to control where memory comes from—for example, to add logging or use a custom pool.

### Purposes

- To customize memory allocation globally or per class.
- To add allocation tracking and debugging.
- To implement memory pools and arena allocators.
- To route allocations through a custom memory resource.

### Syntax Rules and Structure

**General Syntax (replaceable)**
```cpp
void* operator new(std::size_t size);
void operator delete(void* ptr) noexcept;
void* operator new[](std::size_t size);
void operator delete[](void* ptr) noexcept;
```

**Component Breakdown**

| Function | Role |
|----------|------|
| `operator new(size_t)` | Allocates raw bytes. |
| `operator delete(void*)` | Releases raw bytes. |
| `operator new[](size_t)` | Allocates raw bytes for arrays. |
| `operator delete[](void*)` | Releases array storage. |

**Syntax Rules**
- The global versions are implicitly declared in every translation unit.
- They can be replaced at global scope; class-specific overloads are also allowed.
- `operator delete` must not throw.

**Constraints and Limitations**
- Replacement must be consistent: if you replace `operator new`, replace `operator delete` too.
- Class-specific `operator new` must be a `static` public member.

### Annotated Code Example

```cpp
#include <iostream>
#include <new>

void* operator new(std::size_t size) {
    std::cout << "Allocating " << size << " bytes\n";
    if (void* p = std::malloc(size)) return p;
    throw std::bad_alloc();
}

void operator delete(void* p) noexcept {
    std::cout << "Deallocating\n";
    std::free(p);
}

int main() {
    int* x = new int(5);
    delete x;
}
```

**Expected Output**
```
Allocating 4 bytes
Deallocating
```

**Why this output?**  
The replaced `operator new` logs the allocation size; `operator delete` logs the deallocation.

### Real-World Cases

- **Leak detection** – logging every allocation and deallocation.
- **Custom allocators** – arena and pool allocators.
- **Embedded systems** – routing allocations to fixed memory regions.

---

## 6. `malloc` / `free`

### Definitions

**Core Definition**
`malloc` allocates a block of uninitialized memory of a specified size; `free` releases a block previously obtained from `malloc`, `calloc`, or `realloc`.

**Technical Definition**
`std::malloc(size)` allocates `size` bytes of uninitialized storage, suitably aligned for any object with fundamental alignment. It does **not** construct objects. `std::free(ptr)` deallocates memory previously allocated by `malloc`, `calloc`, or `realloc`; passing `nullptr` is a no-op, and double free is undefined behaviour. In C++, `malloc`/`free` are available but should generally be avoided in favour of `new`/`delete` or smart pointers (Core Guidelines R.10).

**Beginner-Friendly Explanation**
`malloc` and `free` are C's memory functions. `malloc` gives you a raw block of bytes—no constructors, no initialization. `free` gives it back. In C++, they are legal but usually the wrong tool unless you are interfacing with C.

### Purposes

- To allocate raw memory without constructing objects.
- To interoperate with C libraries that return `malloc`'d memory.
- To implement low-level allocators.

### Syntax Rules and Structure

**General Syntax**
```cpp
void* p = std::malloc(size);      // allocate
std::free(p);                     // deallocate
```

**Component Breakdown**

| Function | Behaviour |
|----------|-----------|
| `malloc(size)` | Allocates `size` uninitialized bytes; returns `nullptr` on failure. |
| `free(ptr)` | Releases memory; `free(nullptr)` is safe. |

**Syntax Rules**
- `malloc` returns `void*`; cast is required in C++.
- Memory from `malloc` must be freed with `free`, not `delete`.
- `malloc(0)` is implementation-defined (may return `nullptr` or a unique pointer).

**Constraints and Limitations**
- Does not call constructors or destructors.
- Cannot be used on memory from `new`.

### Annotated Code Example

```cpp
#include <cstdlib>
#include <iostream>

int main() {
    int* p = static_cast<int*>(std::malloc(sizeof(int)));
    if (!p) return 1;
    *p = 42;                          // no constructor
    std::cout << *p << '\n';          // 42
    std::free(p);
}
```

**Expected Output**
```
42
```

**Why this output?**  
`malloc` allocates raw storage; the `int` is assigned directly; `free` releases it.

### Real-World Cases

- **C API integration** – `strdup`-style functions.
- **Custom allocators** – building blocks for pools.

---

## 7. `calloc` / `realloc`

### Definitions

**Core Definition**
`calloc` allocates zero-initialized memory for an array; `realloc` resizes an existing allocation, potentially moving it.

**Technical Definition**
`std::calloc(nmemb, size)` allocates `nmemb * size` bytes and initializes all bits to zero. `std::realloc(ptr, new_size)` attempts to resize the block pointed to by `ptr`; if it cannot resize in place, it allocates a new block, copies the old contents, and frees the old block. On failure, `realloc` returns `nullptr` and leaves the original block untouched (a common leak source if the return value overwrites the only pointer).

**Beginner-Friendly Explanation**
`calloc` is like `malloc` but gives you zeroed memory. `realloc` changes the size of an existing block—it might extend it in place or move it somewhere bigger.

### Purposes

- To allocate zero-initialized arrays (`calloc`).
- To grow or shrink an existing dynamic buffer (`realloc`).
- To manage resizable buffers in C-style code.

### Syntax Rules and Structure

**General Syntax**
```cpp
void* p = std::calloc(nmemb, size);
void* q = std::realloc(p, new_size);
```

**Component Breakdown**

| Function | Behaviour |
|----------|-----------|
| `calloc` | Allocates `nmemb * size` zeroed bytes. |
| `realloc` | Resizes block; may move it. |

**Syntax Rules**
- `realloc` returns the new pointer; the old pointer may be invalid after the call.
- Always assign to a temporary first: `void* tmp = realloc(p, n); if (tmp) p = tmp;`.
- `realloc(nullptr, size)` is equivalent to `malloc(size)`.

**Constraints and Limitations**
- `realloc` may copy the entire block.
- Failure leaves the original block intact but returns `nullptr`.

### Annotated Code Example

```cpp
#include <cstdlib>
#include <iostream>

int main() {
    int* p = static_cast<int*>(std::calloc(3, sizeof(int)));
    if (!p) return 1;
    std::cout << p[0] << ' ' << p[1] << ' ' << p[2] << '\n'; // 0 0 0

    int* tmp = static_cast<int*>(std::realloc(p, 5 * sizeof(int)));
    if (!tmp) { std::free(p); return 1; }
    p = tmp;
    p[3] = 42;
    std::cout << p[3] << '\n';        // 42
    std::free(p);
}
```

**Expected Output**
```
0 0 0
42
```

**Why this output?**  
`calloc` zeroes the first three elements; `realloc` grows the block to five elements, preserving the first three.

### Real-World Cases

- **Dynamic buffers** – growing a buffer as data arrives.
- **C data structures** – resizable arrays in C-compatible code.

---

## 8. Memory Leaks

### Definitions

**Core Definition**
A memory leak occurs when dynamically allocated memory is never released, so the program gradually consumes more and more memory until it exhausts available resources.

**Technical Definition**
A memory leak is a failure to deallocate memory that is no longer reachable. Common causes include: forgetting to call `delete`/`delete[]`, losing the only pointer to an allocation, early returns or exceptions bypassing deallocation, and double ownership confusion. Leaks are especially dangerous in long-running processes. Tools such as Valgrind, AddressSanitizer, and `_CrtDumpMemoryLeaks` (MSVC) detect them.

**Beginner-Friendly Explanation**
A memory leak is like borrowing a book from the library and never returning it. Eventually, the library runs out of books. In C++, every `new` should have a matching `delete`.

### Purposes

- To understand why leaks happen and how to prevent them.
- To recognize leak-prone code patterns.
- To select the right detection tool.

### Common Causes

| Cause | Example |
|-------|---------|
| Missing `delete` | `int* p = new int;` with no `delete` |
| Lost pointer | `p = new int;` overwrites the only pointer |
| Early return | `return;` before `delete` |
| Exception path | `throw` before `delete` |
| Double ownership | Two pointers think they own the same object |

### Annotated Code Example

```cpp
#include <iostream>

void leak() {
    int* p = new int(42);
    if (*p == 42) return;             // LEAK: p never deleted
    delete p;
}

int main() {
    for (int i = 0; i < 1000; ++i) leak(); // leaks grow
}
```

**Expected Output**  
(No output; memory usage grows silently.)

**Why this output?**  
The early return bypasses `delete p`, leaking one `int` per call.

### Real-World Cases

- **Long-running servers** – leaks accumulate over days.
- **Game engines** – per-frame leaks cause rapid exhaustion.
- **Embedded systems** – limited memory makes leaks fatal.

---

## 9. Memory Fragmentation

### Definitions

**Core Definition**
Memory fragmentation is the condition where free memory exists in many small, non-contiguous blocks, making it impossible to satisfy large allocation requests even though the total free memory is sufficient.

**Technical Definition**
Fragmentation comes in two forms: **external fragmentation** (free blocks are scattered, so a contiguous request cannot be met) and **internal fragmentation** (allocated blocks are larger than requested due to alignment or allocator overhead). Frequent allocation and deallocation of varying sizes worsens fragmentation. Mitigations include object pools, arena allocators, and PMR (polymorphic memory resources, C++17).

**Beginner-Friendly Explanation**
Fragmentation is like having enough total free space in a parking lot, but no single space large enough for a truck. The memory is “holey.” Pools and arenas prevent this by reusing fixed-size chunks or freeing everything at once.

### Purposes

- To understand why allocation can fail despite free memory.
- To recognize workloads that cause fragmentation.
- To choose the right mitigation strategy.

### Mitigation Strategies

| Strategy | How It Helps |
|----------|--------------|
| Object pool | Recycles fixed-size blocks; no external fragmentation. |
| Arena allocator | Frees all memory at once; no holes. |
| PMR | Pluggable memory resources for standard containers. |
| Pre-allocation | Reserves a large block up front. |

### Annotated Code Example (Conceptual)

```cpp
// Pool allocator: fixed-size blocks, no fragmentation
std::pmr::monotonic_buffer_resource pool;
std::pmr::vector<int> v{&pool};       // allocations come from pool
// All memory released when pool is destroyed
```

**Expected Output**  
(No output; allocations are served from the pool.)

### Real-World Cases

- **Game engines** – per-frame allocations cause fragmentation.
- **Trading systems** – deterministic latency requires pools.
- **Long-running servers** – fragmentation degrades performance over time.

---

## 10. Dynamic Memory Best Practices

### Definitions

**Core Definition**
Dynamic memory best practices are guidelines for using `new`/`delete` and `malloc`/`free` correctly, safely, and minimally, with RAII and smart pointers as the default approach.

**Technical Definition**
The C++ Core Guidelines recommend: prefer scoped objects and avoid unnecessary heap allocation (R.5); avoid `malloc()`/`free()` (R.10); perform at most one explicit resource allocation per statement (R.13); and use `unique_ptr`/`shared_ptr` for ownership (R.20–R.23). In modern C++, manual `new`/`delete` should be rare; containers and smart pointers manage memory automatically.

**Beginner-Friendly Explanation**
The golden rule is: don't use `new`/`delete` unless you absolutely have to. Use `std::vector`, `std::string`, and smart pointers—they clean up after themselves. When you must allocate manually, follow strict matching and RAII.

### Key Guidelines

| Guideline | Summary |
|-----------|---------|
| **R.5** | Prefer scoped objects; don't heap-allocate unnecessarily. |
| **R.10** | Avoid `malloc()`/`free()`. |
| **R.11** | Avoid explicit `new` and `delete`. |
| **R.13** | Perform at most one explicit resource allocation per statement. |
| **R.20** | Use `unique_ptr` or `shared_ptr` to represent ownership. |
| **R.21** | Prefer `unique_ptr` over `shared_ptr` unless sharing is needed. |
| **Match forms** | `new`↔`delete`, `new[]`↔`delete[]`, `malloc`↔`free`. |
| **Null after delete** | Reduces double-free risk (optional but helpful). |
| **Use `make_unique`/`make_shared`** | Avoids raw `new` and improves exception safety. |

### Annotated Code Example

```cpp
#include <memory>
#include <vector>
#include <iostream>

int main() {
    // Modern approach: no manual new/delete
    std::vector<int> v = {1, 2, 3};
    auto p = std::make_unique<int>(42);

    std::cout << v.size() << ' ' << *p << '\n';
} // v and p cleaned up automatically
```

**Expected Output**
```
3 42
```

**Why this output?**  
`std::vector` and `std::unique_ptr` manage their own memory; no explicit `delete` is needed. This eliminates leaks and double frees.

### Real-World Cases

- **Code reviews** – flagging raw `new`/`delete`.
- **Static analysis** – clang-tidy `cppcoreguidelines-owning-memory` and `cppcoreguidelines-no-malloc`.
- **Library design** – exposing RAII-based interfaces.

---

## References

- Low-level memory management — cppreference – https://en.cppreference.com/w/cpp/memory/new
- `operator new`, `operator new[]` — cppreference – https://en.cppreference.com/w/cpp/memory/new/operator_new
- `new` expression — cppreference – https://en.cppreference.com/w/cpp/language/new
- `delete` expression — cppreference – https://en.cppreference.com/w/cpp/language/delete
- `std::malloc` — cppreference – https://en.cppreference.com/w/c/memory/malloc
- `std::free` — cppreference – https://en.cppreference.com/w/c/memory/free
- `std::calloc` — cppreference – https://en.cppreference.com/w/c/memory/calloc
- `std::realloc` — cppreference – https://en.cppreference.com/w/c/memory/realloc
- `std::set_new_handler` — cppreference – https://en.cppreference.com/w/cpp/memory/new/set_new_handler
- `new` and `delete` operators — Microsoft Learn – https://learn.microsoft.com/en-us/cpp/cpp/new-and-delete-operators
- C++ Core Guidelines — Resource management – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-resource
- Understand placement `new` and its valid use cases — Dr-Sergey/learn_cpp – https://github.com/Dr-Sergey/learn_cpp/blob/main/03_Memory_and_Ownership/Understand_placement_new_and_its_valid_use_cases.md
- Design memory management architecture — arenas, pools, and PMR — Dr-Sergey/learn_cpp – https://github.com/Dr-Sergey/learn_cpp/blob/main/31_Project_Architecture/Design_memory_management_architecture_-_arenas_pools_and_PMR.md
- MEM51-CPP. Properly deallocate dynamically allocated resources — SEI CERT – https://wiki.sei.cmu.edu/confluence/display/cplusplus/MEM51-CPP.+Properly+deallocate+dynamically+allocated+resources

---

This cheat sheet provides a comprehensive, structured reference for C++ dynamic memory, covering `new`, `delete`, `new[]`/`delete[]`, placement new, `operator new`/`operator delete`, `malloc`/`free`/`calloc`/`realloc`, memory leaks, fragmentation, and best practices, with annotated code examples, expected outputs, real-world cases, and authoritative references.