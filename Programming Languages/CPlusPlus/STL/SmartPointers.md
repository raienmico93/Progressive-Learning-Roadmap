# C++ Smart Pointers — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Smart pointers are class templates in the C++ Standard Library (`<memory>`) that behave like pointers but automatically manage the lifetime of the object they point to, ensuring proper deallocation and preventing memory leaks.

**Technical Definition**
Smart pointers are RAII (Resource Acquisition Is Initialization) wrappers around raw pointers. They own the object they point to and are responsible for calling the appropriate deleter (by default, `delete`) when the smart pointer goes out of scope or is reset. The three primary smart pointers are `std::unique_ptr` (exclusive ownership), `std::shared_ptr` (shared ownership via reference counting), and `std::weak_ptr` (non-owning observer of a `shared_ptr`-managed object).

**Beginner-Friendly Explanation**
A smart pointer is like a "self-cleaning" pointer. Instead of manually calling `delete` (which is easy to forget or do twice), you use a smart pointer that automatically deletes the object when it's no longer needed. This prevents memory leaks and dangling pointers.

### Key Characteristics

- **Automatic lifetime management** – objects are destroyed when the owning smart pointer goes out of scope.
- **Ownership semantics** – `unique_ptr` models exclusive ownership, `shared_ptr` models shared ownership, `weak_ptr` models non-owning observation.
- **Exception safety** – smart pointers guarantee deletion even when exceptions are thrown.
- **Customizable deleters** – both `unique_ptr` and `shared_ptr` support custom cleanup functions.
- **Move semantics** – `unique_ptr` is move-only; `shared_ptr` is copyable and movable.

### Prerequisites

- Basic C++ syntax (classes, templates, pointers).
- Understanding of dynamic allocation (`new`/`delete`).
- Familiarity with move semantics and RAII.
- Knowledge of exception handling.

### Related Programming Areas

- **RAII** – smart pointers are the canonical example of resource management via object lifetime.
- **Exception safety** – smart pointers are essential for writing exception-safe code.
- **Move semantics** – `unique_ptr` relies on move semantics for ownership transfer.
- **Concurrency** – `shared_ptr` reference counting is thread-safe (but the pointed-to object is not automatically synchronized).

### Core Concepts / Features

1. `std::unique_ptr`
2. `std::shared_ptr`
3. `std::weak_ptr`
4. Smart pointer operations
5. Custom deleters
6. Smart pointer best practices

---

## 1. `std::unique_ptr`

### Definitions

**Core Definition**
`std::unique_ptr` is a smart pointer that retains sole ownership of an object through a pointer and destroys that object when the `unique_ptr` goes out of scope.

**Technical Definition**
`std::unique_ptr<T, Deleter = std::default_delete<T>>` is a move-only class template that manages a single object (or array) via a raw pointer. No two `unique_ptr` instances can manage the same object. The object is destroyed using the stored deleter when the `unique_ptr` is destroyed or assigned a new pointer. A `unique_ptr` may also own no object, in which case it is called *empty* .

**Beginner-Friendly Explanation**
`unique_ptr` is the "default" smart pointer. It means "I am the only one who owns this object, and when I'm gone, the object goes with me." You cannot copy a `unique_ptr`, only move it to transfer ownership.

### Purposes

- To provide exclusive ownership of a dynamically allocated object.
- To guarantee deletion on scope exit, including during exception unwinding.
- To transfer ownership between functions and containers safely.
- To serve as the default smart pointer for most use cases (Core Guidelines R.21).

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <memory>
std::unique_ptr<T> p = std::make_unique<T>(args...);
std::unique_ptr<T> p(new T(args...));           // direct (less preferred)
std::unique_ptr<T[]> p = std::make_unique<T[]>(n); // array
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `T` | The type of the managed object. |
| `Deleter` | The deleter type (default `std::default_delete<T>`). |
| `make_unique` | Factory function (C++14) that creates a `unique_ptr` and forwards arguments to `T`'s constructor. |

**Syntax Rules**
- `unique_ptr` is move-only; copy construction and copy assignment are deleted.
- `std::make_unique<T>(args...)` is the preferred way to create a `unique_ptr` (avoids raw `new`).
- `unique_ptr<T[]>` manages arrays and uses `delete[]`.
- Only a non-const `unique_ptr` can transfer ownership.

**Constraints and Limitations**
- Cannot be copied; must be moved (`std::move`).
- Does not support shared ownership; use `shared_ptr` for that.
- The deleter is part of the type, so different deleters produce different `unique_ptr` types.

### Annotated Code Examples

**Example 1: Basic ownership**

```cpp
#include <memory>
#include <iostream>

struct Widget {
    Widget() { std::cout << "Widget created\n"; }
    ~Widget() { std::cout << "Widget destroyed\n"; }
};

int main() {
    {
        std::unique_ptr<Widget> p = std::make_unique<Widget>();
        // Widget is alive here
    } // Widget destroyed automatically here
}
```

**Expected Output**
```
Widget created
Widget destroyed
```

**Why this output?**  
`make_unique` creates the `Widget`; when `p` goes out of scope, its destructor automatically destroys the `Widget`.

**Example 2: Transferring ownership**

```cpp
#include <memory>
#include <iostream>

void take(std::unique_ptr<int> p) {
    std::cout << "Received: " << *p << '\n';
} // p is destroyed here

int main() {
    auto p = std::make_unique<int>(42);
    take(std::move(p));                    // ownership transferred
    std::cout << std::boolalpha << (p == nullptr) << '\n'; // true
}
```

**Expected Output**
```
Received: 42
true
```

**Why this output?**  
`std::move(p)` transfers ownership to the parameter; after the function returns, `p` is empty (`nullptr`).

### Real-World Cases

- **Polymorphic objects** – `std::unique_ptr<Base> p = std::make_unique<Derived>();`.
- **Factory functions** – returning `unique_ptr` to transfer ownership to the caller.
- **Pimpl idiom** – `std::unique_ptr<Impl> pimpl_;`.

---

## 2. `std::shared_ptr`

### Definitions

**Core Definition**
`std::shared_ptr` is a smart pointer that retains shared ownership of an object through a pointer; multiple `shared_ptr` instances may own the same object.

**Technical Definition**
`std::shared_ptr<T>` is a copyable smart pointer that uses reference counting to manage a shared object. The object is destroyed and its memory deallocated when the last remaining `shared_ptr` owning it is destroyed or is assigned another pointer via `operator=` or `reset()`. A `shared_ptr` may also own no objects, in which case it is called *empty* .

**Beginner-Friendly Explanation**
`shared_ptr` is a smart pointer that allows multiple owners. It keeps a count of how many `shared_ptr`s are pointing to the object; when the count drops to zero, the object is deleted. This is useful when different parts of a program need to share the same data.

### Purposes

- To provide shared ownership of a dynamically allocated object.
- To automatically manage the lifetime of objects shared across multiple components.
- To avoid premature deletion while other parts of the program still need the object.
- To enable safe sharing of resources between threads (reference counting is thread-safe).

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <memory>
std::shared_ptr<T> p = std::make_shared<T>(args...);
std::shared_ptr<T> p2 = p;                  // copy: increments ref count
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `T` | The type of the managed object. |
| `make_shared` | Factory function (C++11) that creates both the object and the control block in a single allocation. |

**Syntax Rules**
- `shared_ptr` is copyable; each copy increments the reference count.
- The object is destroyed when the last `shared_ptr` is destroyed or reset.
- `std::make_shared<T>(args...)` is the preferred creation method (single allocation, exception-safe).

**Constraints and Limitations**
- Reference counting has overhead (atomic operations for thread safety).
- Cyclic references between `shared_ptr`s cause memory leaks; use `weak_ptr` to break cycles.
- `shared_ptr<T[]>` support exists since C++17, but `make_shared` does not support arrays.

### Annotated Code Examples

**Example 1: Shared ownership**

```cpp
#include <memory>
#include <iostream>

int main() {
    auto p1 = std::make_shared<int>(42);
    std::cout << p1.use_count() << '\n';   // 1

    {
        auto p2 = p1;                      // copy: shared ownership
        std::cout << p1.use_count() << '\n'; // 2
    } // p2 destroyed; ref count drops to 1

    std::cout << p1.use_count() << '\n';   // 1
}
```

**Expected Output**
```
1
2
1
```

**Why this output?**  
`use_count()` reports the number of `shared_ptr` instances owning the object. Copying increases the count; destroying a copy decreases it.

**Example 2: Object destroyed when last owner dies**

```cpp
#include <memory>
#include <iostream>

struct Widget {
    ~Widget() { std::cout << "Widget destroyed\n"; }
};

int main() {
    auto p1 = std::make_shared<Widget>();
    auto p2 = p1;                          // shared
    std::cout << "End of scope\n";
} // both destroyed; Widget destroyed once
```

**Expected Output**
```
End of scope
Widget destroyed
```

**Why this output?**  
The `Widget` is destroyed only when the last `shared_ptr` (`p2`) is destroyed.

### Real-World Cases

- **Shared configuration** – multiple components reading the same config object.
- **Graph nodes** – nodes shared between edges.
- **Caches** – objects kept alive as long as any user holds a reference.

---

## 3. `std::weak_ptr`

### Definitions

**Core Definition**
`std::weak_ptr` is a smart pointer that holds a non-owning ("weak") reference to an object managed by `std::shared_ptr`.

**Technical Definition**
`std::weak_ptr<T>` is a smart pointer that tracks an object managed by `shared_ptr` without contributing to the reference count. It must be converted to `shared_ptr` (via `lock()`) to access the object. `weak_ptr` models *temporary ownership*: it allows access to an object only if it still exists, and is used to break circular references between `shared_ptr`s .

**Beginner-Friendly Explanation**
`weak_ptr` is like a "peek" at a `shared_ptr`'s object. It doesn't keep the object alive. If the object has already been deleted, `weak_ptr` knows it (it's "expired") and you cannot access it. You use `lock()` to try to get a temporary `shared_ptr`.

### Purposes

- To break circular references between `shared_ptr`s (which would otherwise leak).
- To observe an object without affecting its lifetime.
- To safely check whether a shared object still exists.
- To implement caches and observer patterns without ownership.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <memory>
std::weak_ptr<T> wp = sp;                  // observe sp's object
if (auto locked = wp.lock()) {             // try to obtain shared_ptr
    // use *locked
} else {
    // object has expired
}
```

**Component Breakdown**

| Member Function | Description |
|-----------------|-------------|
| `lock()` | Returns a `shared_ptr` if the object still exists, or an empty `shared_ptr` otherwise. |
| `expired()` | Returns `true` if the object has been destroyed. |
| `use_count()` | Returns the number of `shared_ptr`s owning the object. |
| `reset()` | Releases the weak reference. |

**Syntax Rules**
- `weak_ptr` is constructed from a `shared_ptr` (or another `weak_ptr`).
- `lock()` is the only way to access the object.
- `expired()` is a quick check but not a guarantee (the object could be destroyed immediately after).

**Constraints and Limitations**
- Cannot be dereferenced directly.
- Does not own the object; the object may disappear at any time.

### Annotated Code Examples

**Example 1: Checking expiration**

```cpp
#include <memory>
#include <iostream>

int main() {
    std::weak_ptr<int> wp;

    {
        auto sp = std::make_shared<int>(42);
        wp = sp;                           // weak reference
        std::cout << "use_count: " << wp.use_count() << '\n'; // 1
        if (auto locked = wp.lock())
            std::cout << "Value: " << *locked << '\n';          // 42
    } // sp destroyed; object destroyed

    if (wp.expired())
        std::cout << "Object has expired\n";
}
```

**Expected Output**
```
use_count: 1
Value: 42
Object has expired
```

**Why this output?**  
`weak_ptr` does not increase `use_count`; `lock()` succeeds while the object exists, and `expired()` becomes true after the `shared_ptr` is destroyed.

**Example 2: Breaking a cycle**

```cpp
#include <memory>
#include <iostream>

struct Node {
    std::shared_ptr<Node> next;
    std::weak_ptr<Node> prev;              // breaks the cycle
    ~Node() { std::cout << "Node destroyed\n"; }
};

int main() {
    auto a = std::make_shared<Node>();
    auto b = std::make_shared<Node>();
    a->next = b;
    b->prev = a;                           // weak reference, no cycle
}
```

**Expected Output**
```
Node destroyed
Node destroyed
```

**Why this output?**  
Without `weak_ptr`, `a` and `b` would reference each other, preventing destruction. The `weak_ptr` breaks the cycle.

### Real-World Cases

- **Observer patterns** – observers hold `weak_ptr` to subjects.
- **Caches** – cache entries hold `weak_ptr` to values.
- **Parent-child relationships** – children hold `weak_ptr` to parents to avoid cycles.

---

## 4. Smart Pointer Operations

### Definitions

**Core Definition**
Smart pointer operations are the member functions and non-member functions that allow querying, modifying, and transferring ownership of the managed object.

**Technical Definition**
Common operations include `get()` (returns the raw pointer without releasing ownership), `reset()` (replaces the managed object), `release()` (releases ownership and returns the raw pointer—`unique_ptr` only), `swap()` (exchanges managed objects), and `use_count()` (`shared_ptr`). `unique_ptr` also supports `operator*` and `operator->` for access.

**Beginner-Friendly Explanation**
These are the commands you use to work with smart pointers: checking what they point to, replacing the object, giving up ownership, or counting how many owners there are.

### Key Operations

| Operation | `unique_ptr` | `shared_ptr` | `weak_ptr` |
|-----------|-------------|--------------|------------|
| `get()` | ✓ | ✓ | — |
| `reset()` | ✓ | ✓ | ✓ (releases reference) |
| `release()` | ✓ | — | — |
| `swap()` | ✓ | ✓ | ✓ |
| `use_count()` | — | ✓ | ✓ |
| `lock()` | — | — | ✓ |
| `expired()` | — | — | ✓ |
| `operator*` | ✓ | ✓ | — |
| `operator->` | ✓ | ✓ | — |

### Annotated Code Example

```cpp
#include <memory>
#include <iostream>

int main() {
    auto p = std::make_unique<int>(10);
    std::cout << *p << '\n';               // 10
    std::cout << p.get() << '\n';          // address

    p.reset(new int(20));                  // old object deleted
    std::cout << *p << '\n';               // 20

    int* raw = p.release();                // ownership released
    std::cout << (p == nullptr) << '\n';   // true
    delete raw;                            // must delete manually
}
```

**Expected Output** (address value is illustrative)
```
10
0x...
20
true
```

**Why this output?**  
`reset` deletes the old object and takes ownership of the new one; `release` returns the raw pointer and makes the `unique_ptr` empty.

---

## 5. Custom Deleters

### Definitions

**Core Definition**
A custom deleter is a callable object or function that defines how a smart pointer should destroy its managed object, instead of the default `delete`.

**Technical Definition**
For `unique_ptr`, the deleter is part of the type (`std::unique_ptr<T, Deleter>`). For `shared_ptr`, the deleter is passed to the constructor and stored in the control block; it is not part of the type. Custom deleters enable smart pointers to manage resources other than heap memory (file handles, sockets, C library allocations) or to use non-standard deallocation functions.

**Beginner-Friendly Explanation**
Sometimes you need to clean up a resource in a special way—for example, closing a file with `fclose` instead of `delete`. A custom deleter lets you tell the smart pointer exactly how to clean up.

### Purposes

- To manage resources that are not freed with `delete` (e.g., `FILE*`, `malloc`'d memory).
- To use array deallocation (`delete[]`) correctly with `shared_ptr`.
- To integrate C library resources into RAII.
- To add logging or side effects to resource destruction.

### Syntax Rules and Structure

**General Syntax (unique_ptr)**
```cpp
std::unique_ptr<T, DeleterType> p(ptr, deleter);
// DeleterType is a template parameter
```

**General Syntax (shared_ptr)**
```cpp
std::shared_ptr<T> p(ptr, deleter);
// DeleterType is a constructor argument, not a template parameter
```

**Component Breakdown**

| Smart Pointer | Deleter Location | Example |
|---------------|------------------|---------|
| `unique_ptr` | Template parameter | `unique_ptr<FILE, decltype(&fclose)>` |
| `shared_ptr` | Constructor argument | `shared_ptr<FILE>(fopen(...), &fclose)` |

**Syntax Rules**
- For `unique_ptr`, the deleter type must be specified in the template argument.
- For `shared_ptr`, the deleter is deduced from the constructor argument.
- The deleter must be callable with a pointer to the managed type.

**Constraints and Limitations**
- `unique_ptr` deleters must be stateless or cheap to copy if the `unique_ptr` is to be moved.
- `shared_ptr` deleters are stored in the control block; the deleter type is erased.

### Annotated Code Examples

**Example 1: Custom deleter for `FILE*` with `unique_ptr`**

```cpp
#include <memory>
#include <cstdio>
#include <iostream>

int main() {
    auto file_deleter = [](std::FILE* f) {
        if (f) { std::fclose(f); std::cout << "File closed\n"; }
    };

    std::unique_ptr<std::FILE, decltype(file_deleter)>
        file(std::fopen("test.txt", "w"), file_deleter);

    if (file) std::fputs("hello", file.get());
} // file closed by custom deleter
```

**Expected Output**
```
File closed
```

**Why this output?**  
The custom deleter calls `fclose` instead of `delete`.

**Example 2: Custom deleter with `shared_ptr`**

```cpp
#include <memory>
#include <cstdlib>
#include <iostream>

int main() {
    auto free_deleter = [](int* p) {
        std::free(p);
        std::cout << "Memory freed\n";
    };

    std::shared_ptr<int> sp(static_cast<int*>(std::malloc(sizeof(int))),
                            free_deleter);
    *sp = 42;
    std::cout << *sp << '\n';
} // free_deleter called
```

**Expected Output**
```
42
Memory freed
```

**Why this output?**  
The `shared_ptr` uses `free` (not `delete`) to release the `malloc`'d memory.

### Real-World Cases

- **File handles** – `FILE*` managed with `fclose`.
- **Sockets** – `close()` instead of `delete`.
- **C library allocations** – `free()` for `malloc`'d memory.
- **Array deallocation** – `delete[]` for arrays managed by `shared_ptr` (pre-C++17).

---

## 6. Smart Pointer Best Practices

### Definitions

**Core Definition**
Smart pointer best practices are guidelines for choosing and using smart pointers correctly, safely, and efficiently, as codified in the C++ Core Guidelines (rules R.20–R.37).

**Technical Definition**
These practices are derived from the C++ Core Guidelines' resource management section: use `unique_ptr` for exclusive ownership (R.20), prefer `unique_ptr` over `shared_ptr` unless sharing is genuinely needed (R.21), use `make_shared` to create `shared_ptr`s (R.22), and take smart pointers as parameters only to express lifetime semantics (R.30, R.34).

**Beginner-Friendly Explanation**
Best practices for smart pointers are simple: use `unique_ptr` by default, use `shared_ptr` only when you really need sharing, use `make_unique` and `make_shared`, and don't pass smart pointers around unnecessarily.

### Key Guidelines

| Guideline | Summary |
|-----------|---------|
| **R.20** | Use `unique_ptr` or `shared_ptr` to represent ownership. |
| **R.21** | Prefer `unique_ptr` over `shared_ptr` unless you need to share ownership. |
| **R.22** | Use `make_shared` to make `shared_ptr`s. |
| **R.23** | Use `make_unique` to make `unique_ptr`s. |
| **R.30** | Take smart pointers as parameters only to express lifetime semantics. |
| **R.34** | Take `shared_ptr<widget>&` parameters to express that a function may modify the shared ownership. |
| **R.35** | Take `shared_ptr<widget>&` to express that a function might reseat the shared pointer. |
| **R.36** | Take `const shared_ptr<widget>&` to express that a function may retain a reference count. |
| **R.37** | Do not pass a pointer or reference obtained from an aliased smart pointer. |

### Annotated Code Example

```cpp
#include <memory>
#include <iostream>

// Good: unique_ptr for exclusive ownership
std::unique_ptr<int> create() {
    return std::make_unique<int>(42);
}

// Good: shared_ptr only when sharing is needed
void share(std::shared_ptr<int> sp) {
    std::cout << "Shared value: " << *sp << '\n';
}

int main() {
    auto u = create();                     // unique_ptr
    std::cout << *u << '\n';               // 42

    auto s = std::make_shared<int>(10);    // shared_ptr
    share(s);                              // copy shares ownership
}
```

**Expected Output**
```
42
Shared value: 10
```

**Why this output?**  
`create` returns a `unique_ptr` (exclusive ownership); `share` takes a `shared_ptr` by value, demonstrating shared ownership.

### Real-World Cases

- **Code reviews** – checking for raw `new`/`delete` that should be smart pointers.
- **Static analysis** – tools like clang-tidy enforce Core Guidelines rules.
- **API design** – choosing `unique_ptr` vs `shared_ptr` for ownership semantics.

---

## References

- `std::unique_ptr` — cppreference – https://en.cppreference.com/w/cpp/memory/unique_ptr
- `std::shared_ptr` — cppreference – https://en.cppreference.com/w/cpp/memory/shared_ptr
- `std::weak_ptr` — cppreference – https://en.cppreference.com/w/cpp/memory/weak_ptr
- `std::make_unique` — cppreference – https://en.cppreference.com/w/cpp/memory/unique_ptr/make_unique
- `std::make_shared` — cppreference – https://en.cppreference.com/w/cpp/memory/shared_ptr/make_shared
- C++ Core Guidelines — R.20: Use `unique_ptr` or `shared_ptr` to represent ownership – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rr-owner
- C++ Core Guidelines — R.21: Prefer `unique_ptr` over `shared_ptr` unless you need to share ownership – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rr-unique
- C++ Core Guidelines — R.22: Use `make_shared()` to make `shared_ptr`s – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rr-make_shared
- C++ Core Guidelines — R.23: Use `make_unique()` to make `unique_ptr`s – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rr-make_unique
- `std::weak_ptr::lock` — cppreference – https://en.cppreference.com/w/cpp/memory/weak_ptr/lock
- Smart pointers — Microsoft Learn – https://learn.microsoft.com/en-us/cpp/cpp/smart-pointers-modern-cpp
- Custom deleters — Microsoft Learn – https://learn.microsoft.com/en-us/cpp/standard-library/memory-functions

---

This cheat sheet provides a comprehensive, structured reference for C++ smart pointers, covering `unique_ptr`, `shared_ptr`, `weak_ptr`, their operations, custom deleters, and best practices, with annotated code examples, expected outputs, real-world cases, and authoritative references.