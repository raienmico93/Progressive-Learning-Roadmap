# C++ Allocators — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
An allocator is a class template that encapsulates a memory allocation and deallocation strategy, allowing standard containers and other allocator-aware components to separate the *what* of memory management (how much, what alignment) from the *how* (which memory resource, which pooling strategy).

**Technical Definition**
The C++ Standard Library defines the *Allocator* named requirement: a type `A` that provides `allocate(n)` and `deallocate(p, n)` for objects of type `T`, along with associated member types accessed through `std::allocator_traits`. Since C++11, custom allocators may contain state, and every allocator-aware container stores an allocator instance and uses it—via `allocator_traits`—for all memory acquisition and object construction/destruction. C++17 introduced the *polymorphic memory resource* (PMR) library, which separates the allocator's static type (`polymorphic_allocator<T>`) from its runtime behaviour (`memory_resource`), enabling containers with different allocation strategies to interoperate seamlessly.

**Beginner-Friendly Explanation**
Think of an allocator as a “memory supplier.” Standard containers like `std::vector` and `std::map` need memory to store their elements, but they don't care *where* that memory comes from. An allocator is the object that actually goes and gets the memory—from the global heap, from a pre-allocated buffer, from a thread-local pool, or from anywhere else. By swapping the allocator, you can change how a container manages memory without changing the container itself.

### Key Characteristics

- **Separation of concerns** – containers handle element storage logic; allocators handle raw memory acquisition.
- **Allocator-aware containers** – all standard containers except `std::array` (and `std::inplace_vector` in C++26) meet the *AllocatorAwareContainer* requirements.
- **Traits-based access** – since C++11, containers access allocators exclusively through `std::allocator_traits`, which supplies sensible defaults for optional allocator members.
- **Stateful allocators** – allocators may carry state (e.g., a pointer to a memory resource); each container stores its own allocator instance.
- **Runtime polymorphism (PMR)** – `std::pmr::polymorphic_allocator<T>` has a fixed static type but delegates to a `memory_resource` chosen at runtime, allowing containers with different allocation strategies to be interoperable.
- **Custom resources** – `std::pmr` provides ready-made resources: `monotonic_buffer_resource` (fast, allocate-only), `unsynchronized_pool_resource` (fast, single-threaded pools), and `synchronized_pool_resource` (thread-safe pools).

### Prerequisites

- Solid understanding of C++ templates and the standard containers.
- Familiarity with dynamic memory (`new`/`delete`) and object lifetime.
- Knowledge of the `Allocator` and `AllocatorAwareContainer` named requirements.
- Understanding of runtime polymorphism (virtual functions) for PMR.
- Awareness of thread-safety concepts (for pool resources).

### Related Programming Areas

- **Containers** – every standard container (except `std::array`) is allocator-aware.
- **Memory management** – custom pools, arenas, and memory resources.
- **Performance engineering** – reducing allocation overhead and fragmentation.
- **Embedded systems** – deterministic memory usage via monotonic buffers.
- **Concurrency** – thread-safe pool resources and thread-local allocators.

### Core Concepts / Features

1. Allocators (the overarching concept)
2. `std::allocator`
3. Allocator traits
4. Custom allocators
5. Allocator-aware containers
6. `std::allocator_traits`
7. Polymorphic allocators
8. `std::pmr`
9. `std::pmr::polymorphic_allocator`
10. `std::pmr::memory_resource`
11. `std::pmr::monotonic_buffer_resource`
12. `std::pmr::unsynchronized_pool_resource`
13. `std::pmr::synchronized_pool_resource`
14. Allocator best practices

---

## 1. Allocators (The Overarching Concept)

### Definitions

**Core Definition**
An allocator is an object that provides a policy for allocating and deallocating raw memory, which containers use to manage their element storage.

**Technical Definition**
The *Allocator* named requirement specifies that a type `A` must provide `A::value_type`, `a.allocate(n)`, and `a.deallocate(p, n)`, along with equality comparison. Since C++11, containers access allocators through `std::allocator_traits<A>`, which supplies defaults for optional members such as `construct`, `destroy`, `select_on_container_copy_construction`, and the `propagate_on_container_*` typedefs. Custom allocators need only implement the minimal required interface.

**Beginner-Friendly Explanation**
An allocator is the “memory policy” for a container. It answers the question: “Where should the container get its memory from, and how should it give it back?” By supplying a different allocator, you change the memory strategy without changing the container.

### Purposes

- To decouple memory allocation strategy from container logic.
- To enable custom memory management (pools, arenas, shared memory).
- To support allocator-aware design in generic libraries.
- To allow containers to interoperate when they use different allocation strategies (via PMR).

### Syntax Rules and Structure

**Minimal Allocator Interface (C++11+)**
```cpp
template<class T>
struct MyAllocator {
    using value_type = T;
    MyAllocator() = default;
    template<class U> MyAllocator(const MyAllocator<U>&) {}
    T* allocate(std::size_t n);
    void deallocate(T* p, std::size_t n);
    bool operator==(const MyAllocator&) const noexcept;
    bool operator!=(const MyAllocator&) const noexcept;
};
```

**Component Breakdown**

| Component | Required? | Description |
|-----------|-----------|-------------|
| `value_type` | Yes | The element type. |
| `allocate(n)` | Yes | Allocates storage for `n` objects. |
| `deallocate(p, n)` | Yes | Releases storage. |
| `operator==` / `!=` | Yes | Allocator equality. |
| `construct` / `destroy` | No (defaulted by traits) | Object construction/destruction. |
| `propagate_on_container_*` | No (defaulted to `false`) | Copy/move/swap propagation. |
| `is_always_equal` | No (defaulted to `is_empty`) | Statelessness hint. |

**Constraints and Limitations**
- Allocators must be copy-constructible and (since C++11) may be stateful.
- `allocate` must return storage suitably aligned for `T`.
- `deallocate` must only be called with memory from the same allocator (or an equal one).

### Annotated Code Example

```cpp
#include <memory>
#include <vector>
#include <iostream>

template<class T>
struct LoggingAllocator {
    using value_type = T;
    LoggingAllocator() = default;
    template<class U> LoggingAllocator(const LoggingAllocator<U>&) {}

    T* allocate(std::size_t n) {
        std::cout << "Allocating " << n << " objects\n";
        return std::allocator<T>{}.allocate(n);
    }
    void deallocate(T* p, std::size_t n) {
        std::cout << "Deallocating " << n << " objects\n";
        std::allocator<T>{}.deallocate(p, n);
    }
    bool operator==(const LoggingAllocator&) const noexcept { return true; }
    bool operator!=(const LoggingAllocator&) const noexcept { return false; }
};

int main() {
    std::vector<int, LoggingAllocator<int>> v;
    v.push_back(1);
    v.push_back(2);
    v.push_back(3);
}
```

**Expected Output** (illustrative)
```
Allocating 1 objects
Allocating 2 objects
Deallocating 1 objects
Allocating 4 objects
Deallocating 2 objects
...
```

**Why this output?**  
The custom allocator logs every allocation and deallocation. The vector starts with capacity 0, grows to 1, then 2, then 4, triggering multiple allocations as it resizes.

### Real-World Cases

- **Game engines** – arena allocators for per-frame allocations.
- **Embedded systems** – fixed-pool allocators to avoid heap fragmentation.
- **Shared memory** – allocators backed by shared memory segments.
- **Debugging** – logging allocators to track memory usage.

---

## 2. `std::allocator`

### Definitions

**Core Definition**
`std::allocator<T>` is the default allocator used by all standard library containers when no user-specified allocator is provided.

**Technical Definition**
`std::allocator<T>` is a stateless allocator that uses the global `operator new` and `operator delete` for memory acquisition and release. All instances of `std::allocator<T>` compare equal and can deallocate memory allocated by any other instance of the same type. The `void` specialisation was deprecated in C++17 and removed in C++20.

**Beginner-Friendly Explanation**
`std::allocator` is the “plain vanilla” allocator—it just calls `new` and `delete` behind the scenes. It is what you get by default, and it is perfectly fine for most code.

### Purposes

- To provide the default memory allocation strategy for standard containers.
- To serve as a stateless, interchangeable allocator.
- To act as a fallback when no custom allocator is needed.
- To provide a reference implementation for custom allocators.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <memory>

std::allocator<int> alloc;
int* p = alloc.allocate(10);          // raw storage for 10 ints
alloc.deallocate(p, 10);
```

**Component Breakdown**

| Member | Description |
|--------|-------------|
| `allocate(n)` | Allocates storage for `n` objects. |
| `deallocate(p, n)` | Releases storage. |
| `value_type` | `T`. |
| `is_always_equal` | `std::true_type` (stateless). |
| `propagate_on_container_move_assignment` | `std::true_type` (since C++14). |

**Constraints and Limitations**
- Stateless; cannot be customized per-instance.
- Uses global `operator new`/`operator delete`.
- The `void` specialisation was removed in C++20.

### Annotated Code Example

```cpp
#include <memory>
#include <iostream>

int main() {
    std::allocator<int> alloc;
    int* p = alloc.allocate(3);        // allocate raw storage
    for (int i = 0; i < 3; ++i) p[i] = i * 10;
    for (int i = 0; i < 3; ++i) std::cout << p[i] << ' ';
    alloc.deallocate(p, 3);
}
```

**Expected Output**
```
0 10 20
```

**Why this output?**  
`allocate(3)` obtains raw storage for three `int`s; the program writes and reads through the pointer, then deallocates.

### Real-World Cases

- **Default container construction** – `std::vector<int> v;` uses `std::allocator<int>`.
- **Baseline comparison** – benchmarking custom allocators against the default.

---

## 3. Allocator Traits

### Definitions

**Core Definition**
Allocator traits are a uniform interface—provided by `std::allocator_traits`—for querying and using the properties of any allocator type, filling in defaults for optional members.

**Technical Definition**
`std::allocator_traits<Alloc>` (since C++11) is the standardised way to access allocator properties. It provides member typedefs (`pointer`, `const_pointer`, `size_type`, `difference_type`, `propagate_on_container_copy_assignment`, etc.) and static member functions (`allocate`, `deallocate`, `construct`, `destroy`, `max_size`, `select_on_container_copy_construction`) that work uniformly with both minimal and fully featured allocators.

**Beginner-Friendly Explanation**
Allocator traits are a “smart adapter” for allocators. They let containers talk to any allocator in the same way, even if the allocator doesn't implement every optional feature—the traits supply sensible defaults.

### Purposes

- To provide a uniform interface for all allocators.
- To allow minimal custom allocators to work with standard containers.
- To fill in defaults for optional allocator members.
- To enable generic allocator-aware programming.

### Syntax Rules and Structure

**General Syntax**
```cpp
using Traits = std::allocator_traits<MyAlloc>;
auto p = Traits::allocate(alloc, n);
Traits::construct(alloc, p, args...);
Traits::destroy(alloc, p);
```

**Component Breakdown**

| Trait Member | Description |
|--------------|-------------|
| `allocator_type` | The allocator type. |
| `value_type` | The element type. |
| `pointer` | Pointer type (defaults to `value_type*`). |
| `size_type` | Size type. |
| `propagate_on_container_copy_assignment` | Copy-assignment propagation (default `false`). |
| `propagate_on_container_move_assignment` | Move-assignment propagation (default `false`). |
| `propagate_on_container_swap` | Swap propagation (default `false`). |
| `is_always_equal` | Statelessness hint (default `is_empty`). |
| `allocate(a, n)` | Allocates `n` objects. |
| `deallocate(a, p, n)` | Deallocates. |
| `construct(a, p, args...)` | Constructs an object. |
| `destroy(a, p)` | Destroys an object. |

**Constraints and Limitations**
- Explicit or partial specialisations of `std::allocator_traits` are ill-formed since C++23.
- `construct` and `destroy` are deprecated in C++20 (use `std::construct_at`/`std::destroy_at`).

### Annotated Code Example

```cpp
#include <memory>
#include <iostream>

template<class T>
struct MinimalAlloc {
    using value_type = T;
    T* allocate(std::size_t n) {
        return static_cast<T*>(::operator new(n * sizeof(T)));
    }
    void deallocate(T* p, std::size_t) {
        ::operator delete(p);
    }
};

int main() {
    using Traits = std::allocator_traits<MinimalAlloc<int>>;
    MinimalAlloc<int> alloc;

    int* p = Traits::allocate(alloc, 1);
    Traits::construct(alloc, p, 42);       // uses placement new
    std::cout << *p << '\n';               // 42
    Traits::destroy(alloc, p);
    Traits::deallocate(alloc, p, 1);
}
```

**Expected Output**
```
42
```

**Why this output?**  
`MinimalAlloc` implements only `allocate`/`deallocate`/`value_type`; `allocator_traits` supplies `construct` and `destroy` using placement new and explicit destructor calls.

### Real-World Cases

- **Library implementation** – containers internally use `allocator_traits`.
- **Generic utilities** – writing code that works with any allocator.
- **Custom allocator development** – implementing only the minimal interface.

---

## 4. Custom Allocators

### Definitions

**Core Definition**
A custom allocator is a user-defined type that satisfies the *Allocator* requirements and provides a specialized memory allocation strategy.

**Technical Definition**
A custom allocator is a class template `A<T>` (or a non-template type with a `rebind` mechanism) that provides `value_type`, `allocate`, and `deallocate`, and optionally `construct`, `destroy`, and propagation traits. Since C++11, `allocator_traits` fills in the optional parts, so a minimal custom allocator needs only the three required members.

**Beginner-Friendly Explanation**
A custom allocator is your own “memory supplier.” You write it when the default `std::allocator` isn't good enough—for example, when you want to use a memory pool, a shared memory segment, or a logging allocator.

### Purposes

- To implement specialized memory strategies (pools, arenas, shared memory).
- To add instrumentation (logging, leak detection).
- To reduce fragmentation and allocation overhead.
- To support embedded and real-time constraints.

### Syntax Rules and Structure

**General Syntax**
```cpp
template<class T>
struct CustomAllocator {
    using value_type = T;
    CustomAllocator() = default;
    template<class U> CustomAllocator(const CustomAllocator<U>&) {}

    T* allocate(std::size_t n);
    void deallocate(T* p, std::size_t n);
    bool operator==(const CustomAllocator&) const noexcept;
    bool operator!=(const CustomAllocator&) const noexcept;
};
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `value_type` | The element type. |
| `allocate(n)` | Allocates `n` objects' worth of memory. |
| `deallocate(p, n)` | Releases the memory. |
| Equality operators | Required for allocator comparison. |
| Rebinding constructor | Template constructor for `A<U>` → `A<T>`. |

**Constraints and Limitations**
- The allocator must be copy-constructible.
- If stateful, the state must be properly propagated according to the `propagate_on_container_*` traits.
- `allocate` must return storage aligned for `T`.

### Annotated Code Example

```cpp
#include <memory>
#include <iostream>
#include <vector>

template<class T>
struct PoolAllocator {
    using value_type = T;
    PoolAllocator() = default;
    template<class U> PoolAllocator(const PoolAllocator<U>&) {}

    T* allocate(std::size_t n) {
        std::cout << "Pool alloc: " << n << '\n';
        return static_cast<T*>(::operator new(n * sizeof(T)));
    }
    void deallocate(T* p, std::size_t n) {
        std::cout << "Pool dealloc: " << n << '\n';
        ::operator delete(p);
    }
    bool operator==(const PoolAllocator&) const noexcept { return true; }
    bool operator!=(const PoolAllocator&) const noexcept { return false; }
};

int main() {
    std::vector<int, PoolAllocator<int>> v;
    v.reserve(10);                      // single allocation
    for (int i = 0; i < 10; ++i) v.push_back(i);
}
```

**Expected Output** (illustrative)
```
Pool alloc: 10
```

**Why this output?**  
`reserve(10)` requests storage for 10 elements in one allocation; the allocator logs it. Without `reserve`, the vector would allocate multiple times as it grows.

### Real-World Cases

- **Memory pools** – `std::pmr::unsynchronized_pool_resource` (see §12).
- **Arena allocators** – `std::pmr::monotonic_buffer_resource` (see §11).
- **Shared memory** – allocators backed by `mmap` or `shm_open`.
- **Debugging** – logging and tracking allocations.

---

## 5. Allocator-Aware Containers

### Definitions

**Core Definition**
An allocator-aware container is a container that stores an allocator instance and uses it for all memory allocation, deallocation, construction, and destruction operations.

**Technical Definition**
The *AllocatorAwareContainer* named requirement (since C++11) specifies that a container holds an allocator instance and uses it—through `allocator_traits`—for all memory management. All standard containers except `std::array` meet this requirement. Containers provide constructors that accept an allocator, and `get_allocator()` returns a copy of the stored allocator.

**Beginner-Friendly Explanation**
An allocator-aware container is a container that “knows” about its allocator and uses it consistently. You can pass a custom allocator to its constructor, and it will use that allocator for all its memory needs.

### Purposes

- To allow containers to use custom memory strategies.
- To enable per-container allocator state.
- To support allocator propagation on copy/move/swap.
- To provide a uniform interface for memory management.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::vector<int, MyAlloc<int>> v{MyAlloc<int>{}};
auto alloc = v.get_allocator();
```

**Component Breakdown**

| Feature | Description |
|---------|-------------|
| Allocator constructor | `vector(const Allocator&)`. |
| `get_allocator()` | Returns the stored allocator. |
| Propagation traits | Control copy/move/swap behaviour. |
| `select_on_container_copy_construction` | Determines allocator for copies. |

**Constraints and Limitations**
- `std::array` is **not** allocator-aware (fixed-size, stack-allocated).
- `std::inplace_vector` (C++26) is also not allocator-aware.

### Annotated Code Example

```cpp
#include <vector>
#include <iostream>
#include <memory>

template<class T>
struct DummyAlloc {
    using value_type = T;
    DummyAlloc() = default;
    template<class U> DummyAlloc(const DummyAlloc<U>&) {}
    T* allocate(std::size_t n) { return std::allocator<T>{}.allocate(n); }
    void deallocate(T* p, std::size_t n) { std::allocator<T>{}.deallocate(p, n); }
    bool operator==(const DummyAlloc&) const noexcept { return true; }
    bool operator!=(const DummyAlloc&) const noexcept { return false; }
};

int main() {
    std::vector<int, DummyAlloc<int>> v;
    auto alloc = v.get_allocator();
    std::cout << typeid(alloc).name() << '\n';
}
```

**Expected Output** (implementation-dependent)
```
DummyAlloc<int>
```

**Why this output?**  
`get_allocator()` returns the allocator instance stored by the vector.

### Real-World Cases

- **Per-container pools** – each container gets its own pool allocator.
- **Shared-memory containers** – containers backed by shared memory allocators.
- **Custom container development** – implementing allocator-aware semantics.

---

## 6. `std::allocator_traits`

### Definitions

**Core Definition**
`std::allocator_traits<Alloc>` is a traits class that provides a uniform, standardised interface for accessing and using any allocator type's properties and operations.

**Technical Definition**
`std::allocator_traits` (since C++11) supplies member typedefs and static functions that adapt any allocator—minimal or fully featured—to the interface expected by standard containers. It fills in defaults for optional members: `pointer` defaults to `value_type*`, `propagate_on_container_copy_assignment` defaults to `std::false_type`, `is_always_equal` defaults to `std::is_empty<Alloc>::type`, and `construct`/`destroy` default to placement new and explicit destructor calls.

**Beginner-Friendly Explanation**
`allocator_traits` is the “universal remote” for allocators. It lets containers control any allocator using the same buttons, regardless of which optional features the allocator implements.

### Purposes

- To provide a uniform interface for all allocators.
- To supply defaults for optional allocator members.
- To enable minimal custom allocators to work with standard containers.
- To support generic allocator-aware programming.

### Syntax Rules and Structure

**General Syntax**
```cpp
using Traits = std::allocator_traits<Alloc>;
auto p = Traits::allocate(alloc, n);
Traits::construct(alloc, p, args...);
```

**Component Breakdown**

| Member | Default (if not provided by `Alloc`) |
|--------|--------------------------------------|
| `pointer` | `value_type*` |
| `const_pointer` | `pointer_traits<pointer>::rebind<const value_type>` |
| `size_type` | `make_unsigned<difference_type>::type` |
| `propagate_on_container_copy_assignment` | `false_type` |
| `propagate_on_container_move_assignment` | `false_type` |
| `propagate_on_container_swap` | `false_type` |
| `is_always_equal` | `is_empty<Alloc>::type` |

**Constraints and Limitations**
- Since C++23, explicit/partial specialisations of `allocator_traits` are ill-formed.
- `construct`/`destroy` are deprecated in C++20.

### Annotated Code Example

```cpp
#include <memory>
#include <iostream>
#include <type_traits>

struct Minimal {
    using value_type = int;
    int* allocate(std::size_t n) { return new int[n]; }
    void deallocate(int* p, std::size_t) { delete[] p; }
};

int main() {
    using Traits = std::allocator_traits<Minimal>;
    std::cout << std::boolalpha;
    std::cout << std::is_same_v<Traits::pointer, int*> << '\n';       // true
    std::cout << std::is_same_v<Traits::propagate_on_container_swap,
                                std::false_type> << '\n';              // true
}
```

**Expected Output**
```
true
true
```

**Why this output?**  
`Minimal` does not define `pointer` or propagation traits; `allocator_traits` supplies the defaults.

### Real-World Cases

- **Standard library implementation** – containers use `allocator_traits` internally.
- **Generic allocator adapters** – wrapping existing allocators with new behaviour.

---

## 7. Polymorphic Allocators

### Definitions

**Core Definition**
A polymorphic allocator is an allocator whose behaviour is determined at runtime by an associated `memory_resource`, rather than being fixed by its static type.

**Technical Definition**
`std::pmr::polymorphic_allocator<T>` (since C++17) is an allocator that delegates all memory operations to a `std::pmr::memory_resource*` supplied at construction. Because the resource is a runtime value, containers with `polymorphic_allocator<T>` as their static allocator type can use different underlying allocation strategies—and even be assigned to each other—while remaining type-compatible.

**Beginner-Friendly Explanation**
A polymorphic allocator is a “universal adapter” that can use any memory resource at runtime. You can create two `std::pmr::vector`s with completely different allocation strategies, and they still have the same static type—so they can be assigned to each other.

### Purposes

- To decouple allocation strategy from container type.
- To allow runtime selection of memory resources.
- To enable interoperability between containers with different strategies.
- To simplify generic code that works with different allocators.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <memory_resource>

std::pmr::monotonic_buffer_resource pool;
std::pmr::vector<int> v{&pool};       // uses pool for allocation
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `polymorphic_allocator<T>` | Allocator type with runtime delegation. |
| `memory_resource` | Abstract base class for allocation strategies. |
| Default resource | `std::pmr::get_default_resource()`. |

**Constraints and Limitations**
- `polymorphic_allocator` copy assignment is deleted (since C++20).
- The associated `memory_resource` must outlive the allocator and all containers using it.

### Annotated Code Example

```cpp
#include <memory_resource>
#include <vector>
#include <iostream>

int main() {
    std::pmr::monotonic_buffer_resource pool;
    std::pmr::vector<int> v1{&pool};
    std::pmr::vector<int> v2{&pool};

    v1.push_back(1);
    v1.push_back(2);
    v2.push_back(10);

    // v1 and v2 have the same type, different resources possible
    std::cout << v1.size() << ' ' << v2.size() << '\n';
}
```

**Expected Output**
```
2 1
```

**Why this output?**  
Both vectors use the same `monotonic_buffer_resource` for allocation; they remain type-compatible because their static allocator type is identical.

### Real-World Cases

- **Plugin systems** – different plugins using different memory resources.
- **Arena-based processing** – per-frame arenas for game engines.
- **Interop between libraries** – libraries exposing PMR-based interfaces.

---

## 8. `std::pmr`

### Definitions

**Core Definition**
`std::pmr` is the namespace containing the Polymorphic Memory Resource library introduced in C++17.

**Technical Definition**
The `std::pmr` namespace (defined in `<memory_resource>`) contains `memory_resource`, `polymorphic_allocator`, and the standard memory resource classes (`monotonic_buffer_resource`, `unsynchronized_pool_resource`, `synchronized_pool_resource`), along with `pool_options` and `get_default_resource()`/`set_default_resource()`. It also provides `std::pmr::vector`, `std::pmr::string`, and other aliases for standard containers that use `polymorphic_allocator`.

**Beginner-Friendly Explanation**
`std::pmr` is the “toolbox” for polymorphic memory resources. It contains everything you need to choose and use different memory strategies at runtime.

### Purposes

- To provide a standard, ready-to-use PMR library.
- To offer aliases for PMR-enabled containers.
- To supply standard memory resource implementations.
- To enable runtime-selectable memory strategies.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <memory_resource>

std::pmr::vector<int> v;              // uses default resource
std::pmr::string s{&pool};            // uses custom resource
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `std::pmr::vector<T>` | Alias for `std::vector<T, std::pmr::polymorphic_allocator<T>>`. |
| `std::pmr::string` | PMR-enabled string. |
| `std::pmr::memory_resource` | Abstract allocation strategy. |
| `std::pmr::get_default_resource()` | Returns the default resource. |

**Constraints and Limitations**
- Requires C++17 or later.
- PMR containers have slightly larger object size due to the stored resource pointer.

### Annotated Code Example

```cpp
#include <memory_resource>
#include <string>
#include <iostream>

int main() {
    std::pmr::string s = "hello";
    s += " world";
    std::cout << s << '\n';
}
```

**Expected Output**
```
hello world
```

**Why this output?**  
`std::pmr::string` is a PMR-enabled string that uses the default memory resource for its allocations.

### Real-World Cases

- **Generic libraries** – exposing PMR-based APIs for custom memory strategies.
- **High-performance code** – selecting resources at runtime based on workload.

---

## 9. `std::pmr::polymorphic_allocator`

### Definitions

**Core Definition**
`std::pmr::polymorphic_allocator<T>` is the allocator type that delegates all memory operations to a `memory_resource` chosen at runtime.

**Technical Definition**
The class template `polymorphic_allocator<T>` (since C++17) stores a `memory_resource*` and forwards `allocate`, `deallocate`, `construct`, and `destroy` to it. Since C++20, the default template argument is `std::byte`. All specialisations meet the allocator completeness requirements. The `construct` member uses *uses-allocator construction*, so container elements that themselves are allocator-aware (e.g., nested containers) receive the same resource.

**Beginner-Friendly Explanation**
`polymorphic_allocator` is the “bridge” between containers and memory resources. It is the allocator type that standard containers use when you want PMR behaviour.

### Purposes

- To connect containers to runtime-selected memory resources.
- To provide a uniform allocator type for different strategies.
- To propagate the resource to nested allocator-aware elements.
- To enable interoperable containers with different allocation strategies.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::pmr::polymorphic_allocator<T> alloc{resource_ptr};
```

**Component Breakdown**

| Member | Description |
|--------|-------------|
| `resource()` | Returns the underlying `memory_resource*`. |
| `allocate(n)` | Delegates to `resource()->allocate(...)`. |
| `deallocate(p, n)` | Delegates to `resource()->deallocate(...)`. |
| `construct` | Uses-allocator construction. |

**Constraints and Limitations**
- The resource must outlive the allocator and all containers using it.
- `polymorphic_allocator<void>` was removed in C++20.

### Annotated Code Example

```cpp
#include <memory_resource>
#include <vector>
#include <iostream>

int main() {
    std::pmr::monotonic_buffer_resource pool;
    std::pmr::polymorphic_allocator<int> alloc{&pool};

    std::pmr::vector<int> v{alloc};
    v.push_back(42);
    std::cout << v[0] << '\n';
}
```

**Expected Output**
```
42
```

**Why this output?**  
The vector uses the `polymorphic_allocator` backed by the monotonic buffer resource; the `42` is stored in memory from the pool.

### Real-World Cases

- **Shared-memory containers** – allocator backed by a shared-memory resource.
- **Arena-allocated containers** – monotonic buffer resource.
- **Pool-allocated containers** – pool resource for small objects.

---

## 10. `std::pmr::memory_resource`

### Definitions

**Core Definition**
`std::pmr::memory_resource` is an abstract base class that defines the interface for all PMR memory resources.

**Technical Definition**
`std::pmr::memory_resource` (since C++17) declares pure virtual `do_allocate`, `do_deallocate`, and `do_is_equal`. Public non-virtual functions `allocate`, `deallocate`, and `is_equal` dispatch to these. Concrete resources (monotonic, pool, etc.) override the `do_*` functions to implement specific strategies.

**Beginner-Friendly Explanation**
`memory_resource` is the “abstract blueprint” for memory strategies. It says: “Any memory resource must be able to allocate, deallocate, and compare itself to another resource.” Concrete resources like pools and arenas implement these operations.

### Purposes

- To define a common interface for all PMR resources.
- To enable runtime polymorphism in memory allocation.
- To allow custom memory strategies to be plugged into PMR containers.
- To support resource comparison and equality.

### Syntax Rules and Structure

**General Syntax**
```cpp
class MyResource : public std::pmr::memory_resource {
protected:
    void* do_allocate(std::size_t bytes, std::size_t align) override;
    void do_deallocate(void* p, std::size_t bytes, std::size_t align) override;
    bool do_is_equal(const memory_resource& other) const noexcept override;
};
```

**Component Breakdown**

| Virtual Function | Description |
|------------------|-------------|
| `do_allocate(bytes, align)` | Allocates storage. |
| `do_deallocate(p, bytes, align)` | Releases storage. |
| `do_is_equal(other)` | Compares resources. |

**Constraints and Limitations**
- `do_allocate` must return storage aligned to `align`.
- `do_deallocate` must accept pointers from the same resource (or an equal one).

### Annotated Code Example

```cpp
#include <memory_resource>
#include <iostream>

struct CountingResource : std::pmr::memory_resource {
    std::size_t allocated = 0;
protected:
    void* do_allocate(std::size_t bytes, std::size_t) override {
        allocated += bytes;
        return ::operator new(bytes);
    }
    void do_deallocate(void* p, std::size_t, std::size_t) override {
        ::operator delete(p);
    }
    bool do_is_equal(const memory_resource& other) const noexcept override {
        return this == &other;
    }
};

int main() {
    CountingResource res;
    std::pmr::vector<int> v{&res};
    v.push_back(1);
    v.push_back(2);
    std::cout << res.allocated << '\n';    // bytes allocated
}
```

**Expected Output** (value depends on implementation)
```
8
```

**Why this output?**  
The `CountingResource` tracks the total bytes allocated; the vector's first allocation for two `int`s is 8 bytes (on typical platforms).

### Real-World Cases

- **Custom memory trackers** – counting bytes or allocations.
- **Shared-memory resources** – backed by shared memory segments.
- **Debugging resources** – logging and leak detection.

---

## 11. `std::pmr::monotonic_buffer_resource`

### Definitions

**Core Definition**
`std::pmr::monotonic_buffer_resource` is a special-purpose memory resource that allocates memory monotonically and releases it all at once when the resource is destroyed.

**Technical Definition**
`monotonic_buffer_resource` (since C++17) allocates from a buffer and never frees individual allocations; `do_deallocate` is a no-op. It is intended for very fast allocation in situations where memory is used to build up a few objects and then released all at once. It can be constructed with an initial buffer; when exhausted, additional buffers are obtained from an upstream resource in geometric progression. It is not thread-safe.

**Beginner-Friendly Explanation**
A monotonic buffer is like a notepad: you write on it until it's full, then get another notepad. You never erase individual notes—you just throw the whole notepad away when you're done. It is extremely fast because deallocation is a no-op.

### Purposes

- To provide extremely fast, allocation-only memory management.
- To avoid per-allocation overhead and fragmentation.
- To support arena-based allocation patterns.
- To enable deterministic, bulk deallocation.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::pmr::monotonic_buffer_resource pool;
std::pmr::vector<int> v{&pool};
```

**Component Breakdown**

| Member | Description |
|--------|-------------|
| `release()` | Releases all allocated memory. |
| `upstream_resource()` | Returns the upstream resource. |
| `do_allocate` | Bump-pointer allocation. |
| `do_deallocate` | No-op. |

**Constraints and Limitations**
- Not thread-safe.
- Memory is only released when the resource is destroyed or `release()` is called.
- Individual deallocations do nothing.

### Annotated Code Example

```cpp
#include <memory_resource>
#include <vector>
#include <iostream>

int main() {
    std::pmr::monotonic_buffer_resource pool;
    {
        std::pmr::vector<int> v{&pool};
        for (int i = 0; i < 100; ++i) v.push_back(i);
        std::cout << v.size() << '\n';     // 100
    } // vector destroyed, but memory not released yet
    pool.release();                        // release all at once
}
```

**Expected Output**
```
100
```

**Why this output?**  
The vector stores 100 elements using the monotonic buffer; individual elements are never deallocated—memory is released only by `release()` or resource destruction.

### Real-World Cases

- **Per-frame game allocations** – allocate all frame data, release at end of frame.
- **Parsing and compilation** – build AST nodes, release all at once.
- **Batch processing** – allocate intermediate results, discard all at end.

---

## 12. `std::pmr::unsynchronized_pool_resource`

### Definitions

**Core Definition**
`std::pmr::unsynchronized_pool_resource` is a general-purpose PMR resource that organizes memory into pools of different block sizes, with **no thread-safety overhead**.

**Technical Definition**
`unsynchronized_pool_resource` (since C++17) maintains a set of free lists for different block sizes. Allocation goes to the smallest pool that fits; deallocation returns the block to its pool for reuse. It is **not thread-safe**; it must not be accessed from multiple threads simultaneously. This lack of synchronization makes it the fastest PMR pool option for single-threaded or thread-local use.

**Beginner-Friendly Explanation**
An unsynchronized pool is a “fast, single-threaded memory recycler.” It keeps separate piles of different-sized blocks and reuses them. Because it skips locking, it is very fast—but you must not share it between threads.

### Purposes

- To provide fast, recycled block allocation for single-threaded use.
- To reduce fragmentation compared to the global heap.
- To avoid synchronization overhead in thread-local scenarios.
- To support efficient alloc/dealloc workloads.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::pmr::unsynchronized_pool_resource pool;
std::pmr::vector<int> v{&pool};
```

**Component Breakdown**

| Feature | Description |
|---------|-------------|
| Thread-safe | **No** |
| Deallocation | Recycles blocks |
| Best for | Single-thread alloc/dealloc |

**Constraints and Limitations**
- Must not be accessed from multiple threads simultaneously; doing so is a data race and undefined behaviour.
- Large allocations that don't fit any pool fall through to the upstream resource.

### Annotated Code Example

```cpp
#include <memory_resource>
#include <vector>
#include <iostream>

int main() {
    std::pmr::unsynchronized_pool_resource pool;
    std::pmr::vector<int> v{&pool};

    for (int i = 0; i < 100; ++i) v.push_back(i);
    v.clear();                              // deallocation recycles blocks
    v.push_back(42);                        // reuses a recycled block

    std::cout << v[0] << '\n';              // 42
}
```

**Expected Output**
```
42
```

**Why this output?**  
The pool resource recycles blocks after `clear()`; the subsequent `push_back` reuses a recycled block from the pool.

### Real-World Cases

- **Thread-local caches** – per-thread pool for fast allocation.
- **Single-threaded parsers** – AST node allocation with recycling.
- **Game engines** – per-thread pools for entity data.

---

## 13. `std::pmr::synchronized_pool_resource`

### Definitions

**Core Definition**
`std::pmr::synchronized_pool_resource` is a thread-safe PMR resource that organizes memory into pools of different block sizes, with internal synchronization for concurrent access.

**Technical Definition**
`synchronized_pool_resource` (since C++17) has the same pooling behaviour as `unsynchronized_pool_resource` but may be accessed from multiple threads without external synchronization. It may have thread-specific pools to reduce synchronization costs. It is a general-purpose memory resource suitable for multi-threaded shared allocation.

**Beginner-Friendly Explanation**
A synchronized pool is a “thread-safe memory recycler.” Multiple threads can use it simultaneously; it handles the locking for you.

### Purposes

- To provide thread-safe pooled allocation.
- To reduce synchronization costs via thread-specific pools.
- To support multi-threaded shared allocation workloads.
- To combine pooling benefits with concurrency safety.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::pmr::synchronized_pool_resource pool;
std::pmr::vector<int> v{&pool};          // usable from multiple threads
```

**Component Breakdown**

| Feature | Description |
|---------|-------------|
| Thread-safe | **Yes** |
| Deallocation | Recycles blocks |
| Best for | Multi-threaded shared allocation |

**Constraints and Limitations**
- Higher synchronization overhead than the unsynchronized version.
- May still have thread-specific pools to reduce contention.

### Annotated Code Example

```cpp
#include <memory_resource>
#include <vector>
#include <thread>
#include <iostream>

std::pmr::synchronized_pool_resource pool;

void worker() {
    std::pmr::vector<int> v{&pool};
    for (int i = 0; i < 100; ++i) v.push_back(i);
}

int main() {
    std::thread t1(worker), t2(worker);
    t1.join(); t2.join();
    std::cout << "Done\n";
}
```

**Expected Output**
```
Done
```

**Why this output?**  
Both threads safely use the same `synchronized_pool_resource` for their vectors; the resource handles synchronization internally.

### Real-World Cases

- **Multi-threaded servers** – shared pools for per-request allocations.
- **Parallel algorithms** – thread-safe allocation of temporary buffers.
- **Concurrent data structures** – pooled node allocation across threads.

---

## 14. Allocator Best Practices

### Definitions

**Core Definition**
Allocator best practices are guidelines for choosing, implementing, and using allocators correctly, safely, and efficiently.

**Technical Definition**
These practices are informed by the C++ Core Guidelines (especially R.10: “Avoid `malloc()`/`free()`” and R.11: “Avoid calling `new` and `delete` explicitly”) and the PMR design principles. Key recommendations include: prefer standard containers and their default allocators unless there is a measurable need; use PMR for runtime-selected strategies; use `monotonic_buffer_resource` for allocate-only workloads; use `unsynchronized_pool_resource` for single-threaded allocation; and use `synchronized_pool_resource` for multi-threaded shared allocation.

**Beginner-Friendly Explanation**
Best practices for allocators are simple: don't write a custom allocator unless you have a specific performance or memory problem. When you do, use `std::pmr` and pick the right resource for your workload.

### Key Guidelines

| Guideline | Summary |
|-----------|---------|
| **R.10** | Avoid `malloc()`/`free()`. |
| **R.11** | Avoid explicit `new`/`delete`. |
| **Prefer standard containers** | Let them manage memory with their default allocators. |
| **Use PMR for runtime strategies** | When allocation strategy must be selectable at runtime. |
| **Use `monotonic_buffer_resource`** | For allocate-only, bulk-release workloads. |
| **Use `unsynchronized_pool_resource`** | For single-threaded allocation with recycling. |
| **Use `synchronized_pool_resource`** | For multi-threaded shared allocation. |
| **Measure before optimizing** | Custom allocators add complexity; justify with benchmarks. |
| **Respect thread-safety** | Never share an unsynchronized resource between threads. |

### Annotated Code Example

```cpp
#include <memory_resource>
#include <vector>
#include <iostream>

int main() {
    // Good: use PMR with an appropriate resource
    std::pmr::monotonic_buffer_resource arena;
    std::pmr::vector<int> v{&arena};

    for (int i = 0; i < 1000; ++i)
        v.push_back(i);

    std::cout << v.size() << '\n';          // 1000
    // All memory released when arena is destroyed
}
```

**Expected Output**
```
1000
```

**Why this output?**  
The monotonic buffer resource handles 1000 allocations efficiently; no individual deallocation occurs—memory is released all at once at scope exit.

### Real-World Cases

- **Code reviews** – flagging unnecessary custom allocators.
- **Performance profiling** – identifying allocation hot spots before optimizing.
- **Static analysis** – clang-tidy `cppcoreguidelines-no-malloc` and `cppcoreguidelines-owning-memory`.

---

## References

- `std::allocator` — cppreference – https://en.cppreference.com/w/cpp/memory/allocator
- `std::allocator_traits` — cppreference – https://en.cppreference.com/w/cpp/memory/allocator_traits
- `std::pmr::polymorphic_allocator` — cppreference – https://en.cppreference.com/w/cpp/memory/polymorphic_allocator
- `std::pmr::memory_resource` — cppreference – https://en.cppreference.com/w/cpp/memory/memory_resource
- `std::pmr::monotonic_buffer_resource` — cppreference – https://en.cppreference.com/w/cpp/memory/monotonic_buffer_resource
- `std::pmr::unsynchronized_pool_resource` — cppreference – https://en.cppreference.com/w/cpp/memory/unsynchronized_pool_resource
- `std::pmr::synchronized_pool_resource` — cppreference – https://en.cppreference.com/w/cpp/memory/synchronized_pool_resource
- Allocator named requirement — cppreference – https://en.cppreference.com/w/cpp/named_req/Allocator
- AllocatorAwareContainer named requirement — cppreference – https://en.cppreference.com/w/cpp/named_req/AllocatorAwareContainer
- C++ Core Guidelines — R.10: Avoid `malloc()` and `free()` – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rr-mallocfree
- C++ Core Guidelines — R.11: Avoid calling `new` and `delete` explicitly – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rr-newdelete
- Use `std::pmr::unsynchronized_pool_resource` for Fast Thread-Local Pool Allocation — Dr-Sergey/learn_cpp – https://github.com/Dr-Sergey/learn_cpp/blob/main/03_Memory_and_Ownership/Use_stdpmrunsynchronized_pool_resource_for_fast_thread-local_pool_allocation.md
- Know how to use `std::allocator_traits` to write allocator-aware code — Dr-Sergey/learn_cpp – https://github.com/Dr-Sergey/learn_cpp/blob/main/03_Memory_and_Ownership/Know_how_to_use_stdallocator_traits_to_write_allocator-aware_code.md
- Implementing a custom memory allocator — Vulcanexus documentation – https://docs.vulcanexus.org/en/latest/rst/Tutorials/Advanced/Allocator-Template-Tutorial.html

---

This cheat sheet provides a comprehensive, structured reference for C++ allocators, covering the Allocator concept, `std::allocator`, allocator traits, custom allocators, allocator-aware containers, `std::allocator_traits`, polymorphic allocators, `std::pmr`, the PMR resource types, and best practices, with annotated code examples, expected outputs, real-world cases, and authoritative references.