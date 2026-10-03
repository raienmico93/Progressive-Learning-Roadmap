# Smart Ownership, Resource Management & Concurrency — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Smart pointers are class templates that provide automatic, exception-safe management of dynamically allocated resources. They model ownership semantics — unique, shared, or non-owning — and ensure that resources are released deterministically when no longer needed. Concurrency utilities extend these semantics to multi-threaded environments, enabling safe concurrent access to shared pointer structures.

**Technical Definition**
The C++ standard library provides three primary smart pointer templates in `<memory>`: `std::unique_ptr<T, Deleter>` (exclusive ownership, move-only), `std::shared_ptr<T>` (shared ownership via reference counting), and `std::weak_ptr<T>` (non-owning observer of a `shared_ptr`-managed object). `std::shared_ptr` stores a pointer to the managed object and a pointer to a dynamically allocated control block that holds the strong reference count, weak reference count, deleter, and allocator. C++20 adds a partial specialization `std::atomic<std::shared_ptr<T>>` that allows atomic manipulation of `shared_ptr` objects across threads. Factory functions `std::make_unique` (C++14) and `std::make_shared` (C++11) provide exception-safe construction, with `make_shared` performing a single allocation for both the object and its control block.

**Beginner-Friendly Explanation**
Imagine you have a key to a storage locker. A `unique_ptr` is like being the sole keyholder — when you're done, the locker is cleaned out automatically. A `shared_ptr` is like having multiple keyholders — the locker is only cleaned out when the last keyholder gives up their key. A `weak_ptr` is like having a visitor's pass — you can look inside if the locker is still open, but you can't keep it open. And when multiple threads are involved, `std::atomic<std::shared_ptr>` is like having a special lock that lets threads safely swap keys without stepping on each other's toes.

---

### Key Characteristics

- **Exclusive ownership**: `std::unique_ptr` enforces a single owner; ownership can be transferred but not copied.
- **Reference-counted sharing**: `std::shared_ptr` maintains a strong reference count; the managed object is destroyed when the count reaches zero.
- **Non-owning observation**: `std::weak_ptr` observes a `shared_ptr`-managed object without contributing to its lifetime, enabling cycle-breaking.
- **Custom deleters**: Both `unique_ptr` and `shared_ptr` accept custom cleanup logic, extending RAII to non-memory resources (file descriptors, database handles, GPU buffers).
- **Thread-safe reference counting**: `shared_ptr`'s control block is thread-safe for concurrent reference count modifications.
- **Atomic shared pointers**: C++20's `std::atomic<std::shared_ptr<T>>` provides lock-free or lock-based atomic operations on the entire pointer.
- **Zero-overhead allocation**: `std::make_unique` provides a thin, exception-safe wrapper over `new`; `std::make_shared` combines two allocations into one.

---

### Prerequisites

- Basic C++ syntax: templates, move semantics, lambdas, and RAII.
- Understanding of raw pointers, `new`/`delete`, and dynamic memory.
- Familiarity with copy and move constructors.
- (For custom deleters) Knowledge of function objects and lambdas.
- (For `std::atomic<std::shared_ptr>`) C++20 compiler support and basic multithreading concepts.

---

### Related Programming Areas

- **RAII and exception safety**: Smart pointers are the primary tool for resource acquisition is initialization.
- **Concurrency**: `std::atomic<std::shared_ptr>` and `shared_ptr`'s thread-safe control block support lock-free and lock-based concurrent data structures.
- **API design**: Ownership semantics communicated through smart pointer types in function signatures.
- **Resource management**: Custom deleters generalize RAII to any resource with a release operation.
- **Performance engineering**: `make_shared` optimizes cache locality and reduces allocator calls.

---

### Core Concepts / Features

1. **Unique Ownership** — `std::unique_ptr` and `std::make_unique`.
2. **Shared & Cyclic Ownership** — `std::shared_ptr`, `std::make_shared`, and `std::weak_ptr`.
3. **Custom Deleters** — Cleanup logic for non-memory resources.
4. **Smart Pointer Concurrency** — `std::atomic<std::shared_ptr<T>>`.
5. **Control Block Efficiencies** — Allocation layout and cache-locality trade-offs.

---

## 1. Unique Ownership — `std::unique_ptr`

### Definitions

**Core Definition**
`std::unique_ptr` is a smart pointer that owns and manages another object through a pointer and disposes of that object when the `unique_ptr` goes out of scope. Only one `unique_ptr` can own a given object at any time.

**Technical Definition**
`template<class T, class Deleter = std::default_delete<T>> class unique_ptr;` (C++11, `<memory>`). The primary template manages a single object; a partial specialization `unique_ptr<T[]>` manages an array allocated with `new[]`. `unique_ptr` satisfies MoveConstructible and MoveAssignable but not CopyConstructible or CopyAssignable. Ownership can be transferred via move construction or move assignment. The default deleter `std::default_delete` calls `delete` or `delete[]`. The stored deleter is part of the type and benefits from empty-base optimization (a zero-size deleter adds no overhead).

**Beginner-Friendly Explanation**
A `unique_ptr` is like a personal key to a locker. No one else has a copy. When you're done with it, the locker is automatically emptied. If you want to give the key to someone else, you have to hand it over — you can't both hold the same key. This is the default smart pointer you should reach for when you don't need sharing.

---

### Purposes (all begin with "To")

- **To** provide exclusive ownership of a dynamically allocated object with automatic cleanup.
- **To** replace raw pointers in ownership positions, preventing memory leaks.
- **To** enable exception-safe resource management through RAII.
- **To** transfer ownership of resources between scopes and functions via move semantics.
- **To** support custom cleanup logic for non-memory resources.
- **To** allow zero-overhead management of polymorphic objects through base-class pointers.

---

### Syntax Rules and Structure

```cpp
// Single object
template<class T, class Deleter = std::default_delete<T>>
class unique_ptr;

// Array
template<class T, class Deleter>
class unique_ptr<T[], Deleter>;
```

**Common operations:**
- `std::make_unique<T>(args...)` — creates a `unique_ptr<T>` with a constructed object (C++14).
- `up.get()` — returns the raw pointer (non-owning).
- `up.release()` — releases ownership and returns the raw pointer.
- `up.reset(ptr)` — replaces the managed object, deleting the old one.
- `*up` / `up->member` — access the managed object.
- `std::move(up)` — transfers ownership.

**Constraints and limitations:**
- Not copyable; must be moved.
- `unique_ptr<T[]>` supports `operator[]` but not `operator*` or `operator->`.
- Custom deleters increase the size of `unique_ptr` unless they are empty and use empty-base optimization.
- `make_unique` does not support custom deleters (use the constructor directly).

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — Basic Unique Ownership and Transfer

```cpp
// File: unique_ptr_basic.cpp
// Compile with: g++ -std=c++17 unique_ptr_basic.cpp -o unique_ptr_basic

#include <iostream>
#include <memory>
#include <string>

int main() {
    // Step 1: Create with make_unique (preferred)
    auto up = std::make_unique<std::string>("hello");

    // Step 2: Access the managed object
    std::cout << "Value: " << *up << std::endl;
    std::cout << "Length: " << up->size() << std::endl;

    // Step 3: Transfer ownership via move
    auto up2 = std::move(up);
    std::cout << "After move, up is null: " << (up == nullptr) << std::endl;
    std::cout << "up2 value: " << *up2 << std::endl;

    // Step 4: Release ownership (caller becomes responsible)
    std::string* raw = up2.release();
    std::cout << "After release, up2 is null: " << (up2 == nullptr) << std::endl;
    std::cout << "raw value: " << *raw << std::endl;

    // Step 5: Manually delete the released pointer
    delete raw;

    return 0;
}
```

**Expected Output:**
```
Value: hello
Length: 5
After move, up is null: 1
up2 value: hello
After release, up2 is null: 1
raw value: hello
```

**Why this output:** `make_unique` creates a `unique_ptr` owning a `std::string`. Moving transfers ownership, leaving the source null. `release()` returns the raw pointer without deleting; the caller must delete it manually. The destructor of `up2` would have deleted the object if `release()` had not been called.

---

#### Example 2 — `make_unique` vs. Raw `new` (Exception Safety)

```cpp
// File: make_unique_exception.cpp
// Compile with: g++ -std=c++17 make_unique_exception.cpp -o make_unique_exception

#include <iostream>
#include <memory>
#include <stdexcept>

class Resource {
public:
    Resource(int id) : id_(id) {
        std::cout << "Resource " << id_ << " created" << std::endl;
    }
    ~Resource() {
        std::cout << "Resource " << id_ << " destroyed" << std::endl;
    }
    int id() const { return id_; }
private:
    int id_;
};

void may_throw() {
    throw std::runtime_error("Something went wrong");
}

int main() {
    // Step 1: Safe with make_unique — object is always cleaned up
    try {
        auto r = std::make_unique<Resource>(1);
        may_throw();  // r's destructor runs during stack unwinding
    } catch (const std::exception& e) {
        std::cout << "Caught: " << e.what() << std::endl;
    }

    // Step 2: Unsafe with raw new — potential leak if exception occurs
    // between new and the unique_ptr constructor (not shown, but conceptually)
    std::cout << "Done" << std::endl;

    return 0;
}
```

**Expected Output:**
```
Resource 1 created
Resource 1 destroyed
Caught: Something went wrong
Done
```

**Why this output:** `make_unique` creates the `Resource` and immediately wraps it in a `unique_ptr`. When `may_throw()` throws, stack unwinding calls `r`'s destructor, which deletes the `Resource`. With raw `new` followed by a separate `unique_ptr` construction, an exception between the two operations could leak the resource.

---

### Real-World Cases with Explanation

**Case 1 — Polymorphic factories**: A factory returns `std::unique_ptr<Base>` to transfer ownership of a derived object without exposing the concrete type.

**Case 2 — Pimpl idiom**: A class holds `std::unique_ptr<Impl>` to hide implementation details and reduce compilation dependencies.

**Case 3 — Container of owned objects**: `std::vector<std::unique_ptr<Shape>>` stores polymorphic objects with automatic cleanup.

**Case 4 — Resource acquisition in constructors**: A class member `std::unique_ptr<Connection>` owns a database connection, ensuring release on destruction.

---

### References

- std::unique_ptr — cppreference.com - https://en.cppreference.com/w/cpp/memory/unique_ptr
- std::make_unique — cppreference.com - https://en.cppreference.com/w/cpp/memory/unique_ptr/make_unique
- std::default_delete — cppreference.com - https://en.cppreference.com/w/cpp/memory/default_delete
- std::unique_ptr<T,Deleter>::release — cppreference.com - https://en.cppreference.com/w/cpp/memory/unique_ptr/release

---

## 2. Shared & Cyclic Ownership — `std::shared_ptr` and `std::weak_ptr`

### Definitions

**Core Definition**
`std::shared_ptr` is a smart pointer that retains shared ownership of an object through a pointer. Multiple `shared_ptr` instances may own the same object; the object is destroyed when the last remaining `shared_ptr` is destroyed or reset. `std::weak_ptr` is a non-owning observer of a `shared_ptr`-managed object that can be used to break reference cycles.

**Technical Definition**
`std::shared_ptr<T>` stores two pointers: one to the managed object and one to a dynamically allocated control block. The control block holds the strong reference count (number of `shared_ptr` owners), the weak reference count (number of `weak_ptr` observers), the deleter, and the allocator. When the strong count reaches zero, the managed object is destroyed; when the weak count also reaches zero, the control block is deallocated. `std::weak_ptr<T>` participates in the weak count but not the strong count. `std::make_shared<T>(args...)` performs a single allocation for both the object and the control block. `std::weak_ptr::lock()` returns a `shared_ptr` if the object still exists, or an empty `shared_ptr` otherwise.

**Beginner-Friendly Explanation**
A `shared_ptr` is like a group of friends sharing a rented apartment. Each friend holds a key (a `shared_ptr`). The apartment is cleaned out only when the last friend leaves. A `weak_ptr` is like a visitor's pass — you can check if the apartment is still occupied, but you don't keep it open. If you have two friends who each hold keys to each other's apartments, neither can leave (a cyclic reference). A `weak_ptr` breaks the cycle by making one of them just a visitor.

---

### Sub-Feature: Breaking Cyclic References with `std::weak_ptr`

#### Definitions

**Core Definition**
A cyclic reference occurs when two or more `shared_ptr` objects own each other, preventing the reference count from reaching zero. `std::weak_ptr` breaks the cycle by replacing one direction of ownership with a non-owning observation.

**Technical Definition**
In a cycle where `A` holds `shared_ptr<B>` and `B` holds `shared_ptr<A>`, the strong count for both objects is at least 1 even when no external owner exists, causing a memory leak. Replacing one direction with `std::weak_ptr` removes that direction's contribution to the strong count, allowing the cycle to be broken when external owners release their `shared_ptr`s. `weak_ptr::lock()` atomically checks whether the object still exists and returns a `shared_ptr` that keeps it alive during use.

**Beginner-Friendly Explanation**
Imagine two people holding each other's keys. Neither can leave because the other still has their key. If one person holds a "visitor's pass" instead of a key, the other can leave when they're ready, and the first person's pass just becomes invalid. That's what `weak_ptr` does.

#### Purposes (all begin with "To")

- **To** break cyclic ownership graphs that would otherwise leak memory.
- **To** provide non-owning observation of a `shared_ptr`-managed object.
- **To** check whether a shared object still exists without extending its lifetime.
- **To** enable cache-like structures where entries may be evicted.
- **To** support parent-child relationships where children reference parents without owning them.

#### Syntax Rules and Structure

```cpp
// Create weak_ptr from shared_ptr
std::weak_ptr<T> wp = sp;

// Check if object still exists
if (auto locked = wp.lock()) {
    // locked is a shared_ptr; object is alive
    locked->method();
} else {
    // object has been destroyed
}

// Check expiration without locking
bool expired = wp.expired();
```

**Constraints and limitations:**
- `weak_ptr` does not provide `operator*` or `operator->`; you must `lock()` first.
- `lock()` is thread-safe with respect to the control block.
- A `weak_ptr` keeps the control block alive even after the object is destroyed.
- `weak_ptr` is not copyable from a raw pointer; it must be constructed from a `shared_ptr` or another `weak_ptr`.

#### Annotated Code Example

```cpp
// File: weak_ptr_cycle.cpp
// Compile with: g++ -std=c++17 weak_ptr_cycle.cpp -o weak_ptr_cycle

#include <iostream>
#include <memory>
#include <string>

struct Node {
    std::string name;
    std::shared_ptr<Node> next;   // strong reference (cycle risk)
    std::weak_ptr<Node> prev;     // weak reference (breaks cycle)

    Node(const std::string& n) : name(n) {
        std::cout << "Node " << name << " created" << std::endl;
    }
    ~Node() {
        std::cout << "Node " << name << " destroyed" << std::endl;
    }
};

int main() {
    // Step 1: Create a cycle with one weak reference
    auto a = std::make_shared<Node>("A");
    auto b = std::make_shared<Node>("B");

    a->next = b;      // A strongly owns B
    b->prev = a;      // B weakly observes A

    std::cout << "a use_count: " << a.use_count() << std::endl;
    std::cout << "b use_count: " << b.use_count() << std::endl;

    // Step 2: Access the weak reference
    if (auto locked = b->prev.lock()) {
        std::cout << "B's prev: " << locked->name << std::endl;
    }

    // Step 3: Reset a; B's weak reference expires
    a.reset();
    std::cout << "After a.reset(), b->prev expired: "
              << b->prev.expired() << std::endl;

    // b is still alive (owned by b itself? No, b is owned by the local b)
    // When b goes out of scope, both are destroyed

    return 0;
}
```

**Expected Output:**
```
Node A created
Node B created
a use_count: 1
b use_count: 2
B's prev: A
After a.reset(), b->prev expired: 1
Node A destroyed
Node B destroyed
```

**Why this output:** `a` owns `b` via `next`, and `b` observes `a` via `prev` (weak). The strong count of `a` is 1 (from local `a`); the strong count of `b` is 2 (local `b` + `a->next`). When `a.reset()` is called, `a`'s strong count drops to zero (since `b->prev` is weak), so `A` is destroyed. `b->prev` expires. When `b` goes out of scope, `B` is destroyed. Without the `weak_ptr`, both would leak.

---

#### Real-World Cases with Explanation

**Case 1 — Observer pattern**: Subjects hold `weak_ptr<Observer>` so that observers can be destroyed without the subject preventing cleanup.

**Case 2 — Cache with eviction**: A cache holds `weak_ptr` to cached objects; if an object is no longer referenced elsewhere, it is automatically evicted.

**Case 3 — Parent-child trees**: Children hold `weak_ptr` to their parent, avoiding ownership cycles while allowing upward traversal.

**Case 4 — Callback registration**: A callback system stores `weak_ptr` to the callback object, preventing dangling calls after the object is destroyed.

---

### References

- std::shared_ptr — cppreference.com - https://en.cppreference.com/w/cpp/memory/shared_ptr
- std::weak_ptr — cppreference.com - https://en.cppreference.com/w/cpp/memory/weak_ptr
- std::make_shared — cppreference.com - https://en.cppreference.com/w/cpp/memory/shared_ptr/make_shared
- std::weak_ptr::lock — cppreference.com - https://en.cppreference.com/w/cpp/memory/weak_ptr/lock

---

## 3. Custom Deleters

### Definitions

**Core Definition**
A custom deleter is a user-provided function object or lambda that is invoked instead of `delete` (or `delete[]`) when a smart pointer releases its managed resource. It extends RAII to non-memory resources such as file descriptors, database handles, and GPU buffers.

**Technical Definition**
For `std::unique_ptr`, the deleter is a template parameter: `unique_ptr<T, Deleter>`. The deleter must be callable with `T*` (or the pointer type) and is stored as a member. For `std::shared_ptr`, the deleter is a constructor argument and is stored in the control block as a type-erased callable. The deleter for `unique_ptr` affects the type and size of the pointer; for `shared_ptr`, it does not affect the type. `std::default_delete<T>` is the default deleter for both, calling `delete ptr` or `delete[] ptr`.

**Beginner-Friendly Explanation**
Normally, smart pointers call `delete` to clean up. But some resources aren't cleaned up with `delete` — files are closed with `fclose`, database connections are closed with a custom close function, and GPU buffers are freed with a graphics API call. A custom deleter lets you tell the smart pointer exactly how to clean up your resource.

---

### Sub-Feature: `unique_ptr` Custom Deleters

#### Definitions

**Core Definition**
A `unique_ptr` custom deleter is a callable type passed as the second template parameter that defines how the managed resource is released.

**Technical Definition**
The deleter type `Deleter` must be a FunctionObject or an lvalue reference to one, callable with an argument of type `unique_ptr<T, Deleter>::pointer`. The deleter is stored as a member of the `unique_ptr` and is invoked on destruction, `reset()`, or assignment. If the deleter is an empty class (stateless lambda or function object), the empty-base optimization ensures zero size overhead. A function pointer deleter, however, adds the size of a pointer to the `unique_ptr`.

**Beginner-Friendly Explanation**
For `unique_ptr`, you specify the deleter as part of the type. This means the deleter's type is baked into the pointer, which allows the compiler to optimize it away if it's stateless. It's like telling the compiler "when this pointer is destroyed, call this specific function."

#### Purposes (all begin with "To")

- **To** manage non-memory resources (file descriptors, sockets, database handles) with RAII.
- **To** provide custom cleanup logic for resources allocated by third-party libraries.
- **To** ensure exception-safe release of resources that don't use `delete`.
- **To** allow stateful cleanup (e.g., logging, statistics) during destruction.
- **To** support resources with complex deallocation sequences.

#### Syntax Rules and Structure

```cpp
// Function pointer deleter
void my_deleter(T* ptr);
std::unique_ptr<T, decltype(&my_deleter)> up(ptr, my_deleter);

// Lambda deleter (stateless)
auto deleter = [](T* ptr) { /* cleanup */ };
std::unique_ptr<T, decltype(deleter)> up(ptr, deleter);

// Function object deleter
struct MyDeleter {
    void operator()(T* ptr) const { /* cleanup */ }
};
std::unique_ptr<T, MyDeleter> up(ptr);
```

**Constraints and limitations:**
- The deleter must be callable with the pointer type.
- A function pointer deleter increases the `unique_ptr` size by the size of the pointer.
- The deleter is part of the type; `unique_ptr<T, D1>` and `unique_ptr<T, D2>` are different types and cannot be directly assigned.
- Custom deleters are not supported by `make_unique`; the pointer must be constructed manually.

#### Annotated Code Example

```cpp
// File: unique_custom_deleter.cpp
// Compile with: g++ -std=c++17 unique_custom_deleter.cpp -o unique_custom_deleter

#include <iostream>
#include <memory>
#include <cstdio>

// Step 1: Function object deleter for FILE*
struct FileDeleter {
    void operator()(std::FILE* fp) const {
        if (fp) {
            std::cout << "Closing file" << std::endl;
            std::fclose(fp);
        }
    }
};

// Step 2: Lambda deleter for file descriptor (POSIX)
struct FdWrapper {
    int fd;
    explicit FdWrapper(int f) : fd(f) {}
};

int main() {
    // Step 3: unique_ptr with FileDeleter
    {
        std::unique_ptr<std::FILE, FileDeleter> file(
            std::fopen("test.txt", "w"), FileDeleter{}
        );
        if (file) {
            std::fputs("hello", file.get());
        }
        // File is closed automatically when file goes out of scope
    }

    // Step 4: unique_ptr with lambda deleter for file descriptor
    auto fd_deleter = [](FdWrapper* w) {
        std::cout << "Closing fd " << w->fd << std::endl;
        // In real code: close(w->fd);
        delete w;
    };
    std::unique_ptr<FdWrapper, decltype(fd_deleter)> fd(
        new FdWrapper(42), fd_deleter
    );

    std::cout << "fd value: " << fd->fd << std::endl;

    return 0;
}
```

**Expected Output:**
```
fd value: 42
Closing fd 42
Closing file
```

**Why this output:** The `FileDeleter` closes the `FILE*` when the `unique_ptr` goes out of scope. The lambda deleter closes the file descriptor and deletes the wrapper. Note the order: the `fd` object is destroyed before the `file` object because it was declared later in the scope.

---

### Sub-Feature: `shared_ptr` Custom Deleters

#### Definitions

**Core Definition**
A `shared_ptr` custom deleter is a callable passed to the constructor that is stored in the control block and invoked when the last `shared_ptr` releases ownership.

**Technical Definition**
The deleter is a constructor argument: `shared_ptr<T>(ptr, deleter)`. It is type-erased and stored in the control block. Unlike `unique_ptr`, the deleter does not affect the type of the `shared_ptr`. This means `shared_ptr<T>` with different deleters are the same type and can be stored in the same container. The deleter is invoked exactly once, when the strong reference count reaches zero.

**Beginner-Friendly Explanation**
For `shared_ptr`, the deleter is passed at construction time and is hidden inside the control block. This makes `shared_ptr` more flexible but slightly less efficient than `unique_ptr` because the deleter is type-erased. It's like hiring a cleaning service — you specify what to do when the apartment is vacated, and the service handles it.

#### Purposes (all begin with "To")

- **To** manage shared resources that require custom cleanup.
- **To** allow different cleanup strategies for the same type of `shared_ptr`.
- **To** support resources with release functions that differ from `delete`.
- **To** enable type-erased cleanup in heterogeneous collections.
- **To** provide custom allocation and deallocation for shared objects.

#### Annotated Code Example

```cpp
// File: shared_custom_deleter.cpp
// Compile with: g++ -std=c++17 shared_custom_deleter.cpp -o shared_custom_deleter

#include <iostream>
#include <memory>

int main() {
    // Step 1: shared_ptr with lambda deleter
    auto deleter = [](int* p) {
        std::cout << "Custom deleter called for value " << *p << std::endl;
        delete p;
    };

    std::shared_ptr<int> sp(new int(42), deleter);
    std::cout << "sp use_count: " << sp.use_count() << std::endl;

    // Step 2: Copy the shared_ptr (deleter is shared)
    auto sp2 = sp;
    std::cout << "sp use_count after copy: " << sp.use_count() << std::endl;

    // Step 3: Both go out of scope — deleter called once
    std::cout << "End of main" << std::endl;

    return 0;
}
```

**Expected Output:**
```
sp use_count: 1
sp use_count after copy: 2
End of main
Custom deleter called for value 42
```

**Why this output:** The custom deleter is stored in the control block and invoked only once, when the last `shared_ptr` (`sp2`) is destroyed. Even though both `sp` and `sp2` point to the same object, the deleter is called exactly once.

---

### Real-World Cases with Explanation

**Case 1 — File handles**: A `unique_ptr<FILE, decltype(&fclose)>` manages a file opened with `fopen`, ensuring `fclose` is called.

**Case 2 — Database connections**: A `shared_ptr<sqlite3, decltype(&sqlite3_close)>` shares a database connection across multiple objects, closing it only when the last user is done.

**Case 3 — Graphics pipelines**: A `unique_ptr<VkPipeline, PipelineDeleter>` ensures a Vulkan pipeline is destroyed with `vkDestroyPipeline`.

**Case 4 — Network sockets**: A `shared_ptr<Socket, SocketCloser>` manages a socket descriptor, closing it when no longer needed.

---

### References

- std::unique_ptr<T,Deleter> — cppreference.com - https://en.cppreference.com/w/cpp/memory/unique_ptr
- std::shared_ptr<T>::shared_ptr — cppreference.com - https://en.cppreference.com/w/cpp/memory/shared_ptr/shared_ptr
- Use std::unique_ptr with custom deleters for non-heap resources — Dr-Sergey/learn_cpp - https://github.com/Dr-Sergey/learn_cpp/blob/main/03_Memory_and_Ownership/Use_stdunique_ptr_with_custom_deleters_for_non-heap_resources.md

---

## 4. Smart Pointer Concurrency

### Definitions

**Core Definition**
`std::atomic<std::shared_ptr<T>>` is a partial specialization of `std::atomic` (C++20) that allows atomic manipulation of `shared_ptr` objects across multiple threads without data races.

**Technical Definition**
`template<class T> struct atomic<std::shared_ptr<T>>;` (C++20, `<memory>`). It provides atomic operations (`load`, `store`, `exchange`, `compare_exchange_weak`, `compare_exchange_strong`) on the entire `shared_ptr` (both the stored pointer and the control block pointer). The reference count increments associated with the atomic operation are part of the atomic step; decrements are sequenced after. The specialization is not copyable or movable. The deprecated standalone functions `std::atomic_load`, `std::atomic_store`, etc., should be replaced by this specialization in new code.

**Beginner-Friendly Explanation**
A normal `shared_ptr` is not safe to modify from multiple threads simultaneously because it consists of two pointers (the object and the control block), and updating both cannot be done atomically with ordinary assignment. `std::atomic<std::shared_ptr<T>>` provides a thread-safe wrapper that lets multiple threads read, write, and swap the entire pointer without data races.

---

### Purposes (all begin with "To")

- **To** enable thread-safe concurrent read and write of shared pointer structures.
- **To** implement lock-free or lock-based concurrent data structures (e.g., shared configuration, caches).
- **To** allow atomic replacement of a shared object without stopping readers.
- **To** support publisher-subscriber patterns where multiple threads read a shared pointer while one writes.
- **To** replace deprecated `std::atomic_load`/`std::atomic_store` free functions.

---

### Syntax Rules and Structure

```cpp
#include <memory>
#include <atomic>

std::atomic<std::shared_ptr<T>> asp;

// Atomic load
std::shared_ptr<T> sp = asp.load();

// Atomic store
asp.store(std::make_shared<T>(args));

// Atomic exchange
std::shared_ptr<T> old = asp.exchange(new_sp);

// Compare-and-exchange
std::shared_ptr<T> expected = ...;
std::shared_ptr<T> desired = ...;
bool success = asp.compare_exchange_strong(expected, desired);
```

**Constraints and limitations:**
- Not copyable or movable.
- Initialization is not atomic (as with all `std::atomic` types).
- May use a lock internally (not necessarily lock-free).
- The bottom two bits of the control block pointer may be used as flags (implementation detail).
- Available in C++20; older code must use the deprecated free functions.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — Concurrent Read/Write with Atomic `shared_ptr`

```cpp
// File: atomic_shared_ptr.cpp
// Compile with: g++ -std=c++20 -pthread atomic_shared_ptr.cpp -o atomic_shared_ptr

#include <atomic>
#include <iostream>
#include <memory>
#include <thread>
#include <vector>

struct Config {
    int value;
    explicit Config(int v) : value(v) {}
};

int main() {
    // Step 1: Atomic shared_ptr to a Config
    std::atomic<std::shared_ptr<Config>> config(
        std::make_shared<Config>(1)
    );

    // Step 2: Reader threads — load the current config
    auto reader = [&config](int id) {
        for (int i = 0; i < 5; ++i) {
            auto sp = config.load();  // atomic load
            std::cout << "Reader " << id
                      << " sees value " << sp->value << std::endl;
            std::this_thread::yield();
        }
    };

    // Step 3: Writer thread — atomically replace the config
    auto writer = [&config]() {
        for (int i = 0; i < 5; ++i) {
            auto new_config = std::make_shared<Config>(i * 10);
            config.store(new_config);  // atomic store
            std::this_thread::yield();
        }
    };

    std::vector<std::thread> threads;
    threads.emplace_back(writer);
    threads.emplace_back(reader, 1);
    threads.emplace_back(reader, 2);

    for (auto& t : threads) t.join();

    return 0;
}
```

**Expected Output (interleaving varies):**
```
Reader 1 sees value 1
Reader 2 sees value 1
Reader 1 sees value 0
Reader 2 sees value 0
Reader 1 sees value 10
Reader 2 sees value 10
Reader 1 sees value 20
Reader 2 sees value 20
Reader 1 sees value 30
Reader 2 sees value 30
```

**Why this output:** The writer atomically replaces the `shared_ptr` using `store`, and readers atomically load it using `load`. No data race occurs because all accesses go through the atomic wrapper. The exact interleaving depends on thread scheduling, but each reader always sees a valid, fully constructed `Config`.

---

### Real-World Cases with Explanation

**Case 1 — Hot-reloadable configuration**: A server application holds `std::atomic<std::shared_ptr<const Config>>`; a config reload thread atomically swaps in a new immutable config while worker threads read the current one.

**Case 2 — Lock-free cache**: A cache holds `std::atomic<std::shared_ptr<CacheEntry>>`; readers atomically load the entry, and a background thread atomically replaces it when new data arrives.

**Case 3 — Shared logger**: A logging system holds `std::atomic<std::shared_ptr<Logger>>`; the logger can be swapped at runtime without stopping threads that are currently logging.

**Case 4 — Publisher-subscriber**: A publisher thread atomically updates a shared pointer to the latest data; subscriber threads atomically load the pointer and process the data.

---

### References

- std::atomic<std::shared_ptr> — cppreference.com - https://en.cppreference.com/w/cpp/memory/shared_ptr/atomic2
- std::atomic_...<std::shared_ptr> (deprecated) — cppreference.com - https://en.cppreference.com/w/cpp/memory/shared_ptr/atomic
- P0718R2: Revising atomic_shared_ptr for C++20 — Open-Std - https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2017/p0718r2.html
- Use std::atomic<std::shared_ptr<T>> (C++20) for safe concurrent pointer updates — Dr-Sergey/learn_cpp - https://github.com/Dr-Sergey/learn_cpp/blob/main/10_Concurrency_and_Parallelism/Use_stdatomicstdshared_ptrT_C%2B%2B20_for_safe_concurrent_pointer_updates.md

---

## 5. Control Block Efficiencies

### Definitions

**Core Definition**
The control block is a dynamically allocated data structure that stores the metadata for a `std::shared_ptr`: the strong reference count, the weak reference count, the deleter, and the allocator. The efficiency of control block allocation — whether it is separate or combined with the managed object — affects performance and cache locality.

**Technical Definition**
When a `shared_ptr` is created via `std::make_shared` or `std::allocate_shared`, the memory for both the managed object and the control block is allocated in a single block. The control block is typically placed first, followed by the managed object (or vice versa, depending on implementation). When a `shared_ptr` is created from a raw pointer (`shared_ptr<T>(new T)`), the control block and the managed object are allocated separately. The single-allocation approach improves cache locality (the control block and object are adjacent) and reduces allocator calls from two to one. However, it also means that the memory for the object cannot be freed until the last `weak_ptr` is destroyed, because the control block and object share the same allocation.

**Beginner-Friendly Explanation**
A `shared_ptr` needs a "control block" to keep track of how many owners there are and how to clean up. If you create the pointer with `new` and then wrap it, the control block is allocated separately — two trips to the memory allocator. If you use `make_shared`, the object and the control block are allocated together in one trip — faster and better for the CPU cache. The trade-off is that the memory stays alive until all weak pointers are gone.

---

### Sub-Feature: `make_shared` Single Allocation vs. Separate Allocation

#### Definitions

**Core Definition**
Single allocation (`make_shared`) combines the control block and managed object into one memory block. Separate allocation (`shared_ptr(new T)`) allocates them independently.

**Technical Definition**
The layout of a `make_shared` allocation is typically: `[control block][T object]` or `[T object][control block]`. The control block contains the strong count, weak count, and a copy of the deleter (which for `make_shared` is an internal deleter that destroys the object in place). In the separate allocation case, the control block contains a pointer to the managed object and the deleter. The single allocation improves locality because the reference count operations and object access hit the same cache line more often. The drawback is that the memory for the object is not freed until the weak count reaches zero (because the deallocation must free the combined block).

**Beginner-Friendly Explanation**
Think of `make_shared` as packing a lunchbox: the sandwich (object) and the napkin (control block) are in the same box. When you're done with the sandwich but still have the napkin (weak pointer), the whole box stays around. With separate allocation, the sandwich and napkin are in different boxes — the napkin can stay while the sandwich box is thrown away.

#### Purposes (all begin with "To")

- **To** reduce allocation overhead by combining two allocations into one.
- **To** improve cache locality by placing the control block and object adjacent in memory.
- **To** reduce memory fragmentation by allocating a single contiguous block.
- **To** eliminate the exception-safety window between separate allocations.
- **To** improve performance in allocation-heavy workloads.

#### Syntax Rules and Structure

```cpp
// Single allocation (make_shared)
auto sp1 = std::make_shared<T>(args...);

// Separate allocation (raw new + shared_ptr constructor)
std::shared_ptr<T> sp2(new T(args...));

// allocate_shared for custom allocator
auto sp3 = std::allocate_shared<T>(my_allocator, args...);
```

**Constraints and limitations:**
- `make_shared` does not support custom deleters.
- Memory for the object is not freed until all `weak_ptr`s are destroyed.
- The exact layout (control block before or after object) is implementation-defined.
- `make_shared` requires access to the constructor, which may be private for some types.

#### Annotated Code Example

```cpp
// File: control_block_layout.cpp
// Compile with: g++ -std=c++17 control_block_layout.cpp -o control_block_layout

#include <iostream>
#include <memory>
#include <chrono>

struct LargeObject {
    double data[1024];  // 8 KB
    LargeObject() { for (auto& d : data) d = 1.0; }
};

int main() {
    constexpr int N = 100'000;

    // Step 1: Separate allocation (two allocations per object)
    auto start = std::chrono::high_resolution_clock::now();
    for (int i = 0; i < N; ++i) {
        std::shared_ptr<LargeObject> sp(new LargeObject());
    }
    auto end = std::chrono::high_resolution_clock::now();
    std::cout << "Separate allocation: "
              << std::chrono::duration<double>(end - start).count()
              << "s" << std::endl;

    // Step 2: Single allocation (make_shared)
    start = std::chrono::high_resolution_clock::now();
    for (int i = 0; i < N; ++i) {
        auto sp = std::make_shared<LargeObject>();
    }
    end = std::chrono::high_resolution_clock::now();
    std::cout << "Single allocation:   "
              << std::chrono::duration<double>(end - start).count()
              << "s" << std::endl;

    // Step 3: Demonstrate weak_ptr lifetime effect
    std::weak_ptr<LargeObject> wp;
    {
        auto sp = std::make_shared<LargeObject>();
        wp = sp;
        std::cout << "Object alive, weak_ptr set" << std::endl;
    }
    std::cout << "After sp destroyed, weak expired: "
              << wp.expired() << std::endl;
    // The memory for the object is not freed until wp is destroyed

    return 0;
}
```

**Expected Output (timings vary):**
```
Separate allocation: 0.012s
Single allocation:   0.008s
Object alive, weak_ptr set
After sp destroyed, weak expired: 1
```

**Why this output:** The single-allocation version is faster because it makes one allocator call instead of two. The `weak_ptr` demonstrates that even after the `shared_ptr` is destroyed, the memory block remains allocated until the `weak_ptr` is also destroyed.

---

#### Real-World Cases with Explanation

**Case 1 — High-frequency object creation**: In systems that create and destroy many shared objects (e.g., network packets, game entities), `make_shared` reduces allocator pressure.

**Case 2 — Cache-friendly data structures**: Nodes in a concurrent linked list are allocated with `make_shared` so that the control block and node payload share a cache line.

**Case 3 — Weak reference caches**: When weak references are used for caching, the delayed deallocation of `make_shared` allocations can be a disadvantage if memory is tight.

**Case 4 — Memory-constrained environments**: Separate allocation allows the object memory to be freed while weak pointers still exist, which can be beneficial when memory is limited.

---

### References

- std::make_shared — cppreference.com - https://en.cppreference.com/w/cpp/memory/shared_ptr/make_shared
- std::allocate_shared — cppreference.com - https://en.cppreference.com/w/cpp/memory/shared_ptr/allocate_shared
- Inside STL: The shared_ptr constructor vs make_shared — Raymond Chen, Microsoft DevBlogs - https://devblogs.microsoft.com/oldnewthing/20230815-00/?p=108609
- Know when shared_ptr is appropriate and understand its overhead — Dr-Sergey/learn_cpp - https://github.com/Dr-Sergey/learn_cpp/blob/main/03_Memory_and_Ownership/Know_when_shared_ptr_is_appropriate_and_understand_its_overhead.md
- Control block — cppreference.com - https://en.cppreference.com/w/cpp/memory/shared_ptr#Control_block

---

## Summary Table

| Smart Pointer | Ownership | Copyable | Thread-Safe Control Block | Custom Deleter | Allocation |
|---|---|---|---|---|---|
| `std::unique_ptr` | Exclusive | No (move only) | N/A | Type parameter | Zero-overhead |
| `std::shared_ptr` | Shared | Yes | Yes (reference counts) | Constructor argument | Separate or combined |
| `std::weak_ptr` | Non-owning | Yes | Yes | N/A | Uses shared control block |
| `std::atomic<std::shared_ptr>` | Shared (atomic) | No | Atomic operations | Constructor argument | Same as shared_ptr |

---

## References (Consolidated)

- std::unique_ptr — cppreference.com - https://en.cppreference.com/w/cpp/memory/unique_ptr
- std::shared_ptr — cppreference.com - https://en.cppreference.com/w/cpp/memory/shared_ptr
- std::weak_ptr — cppreference.com - https://en.cppreference.com/w/cpp/memory/weak_ptr
- std::make_unique — cppreference.com - https://en.cppreference.com/w/cpp/memory/unique_ptr/make_unique
- std::make_shared — cppreference.com - https://en.cppreference.com/w/cpp/memory/shared_ptr/make_shared
- std::atomic<std::shared_ptr> — cppreference.com - https://en.cppreference.com/w/cpp/memory/shared_ptr/atomic2
- std::default_delete — cppreference.com - https://en.cppreference.com/w/cpp/memory/default_delete
- std::weak_ptr::lock — cppreference.com - https://en.cppreference.com/w/cpp/memory/weak_ptr/lock
- How to: Create and use weak_ptr instances — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/standard-library/how-to-create-and-use-weak-ptr-instances
- Use std::unique_ptr with custom deleters for non-heap resources — Dr-Sergey/learn_cpp - https://github.com/Dr-Sergey/learn_cpp/blob/main/03_Memory_and_Ownership/Use_stdunique_ptr_with_custom_deleters_for_non-heap_resources.md
- Use std::weak_ptr to break cyclic ownership — Dr-Sergey/learn_cpp - https://github.com/Dr-Sergey/learn_cpp/blob/main/03_Memory_and_Ownership/Use_stdweak_ptr_to_break_cyclic_ownership.md
- Use std::atomic<std::shared_ptr<T>> (C++20) for safe concurrent pointer updates — Dr-Sergey/learn_cpp - https://github.com/Dr-Sergey/learn_cpp/blob/main/10_Concurrency_and_Parallelism/Use_stdatomicstdshared_ptrT_C%2B%2B20_for_safe_concurrent_pointer_updates.md
- P0718R2: Revising atomic_shared_ptr for C++20 — Open-Std - https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2017/p0718r2.html
- Inside STL: The shared_ptr constructor vs make_shared — Raymond Chen - https://devblogs.microsoft.com/oldnewthing/20230815-00/?p=108609
- Control block — cppreference.com - https://en.cppreference.com/w/cpp/memory/shared_ptr#Control_block