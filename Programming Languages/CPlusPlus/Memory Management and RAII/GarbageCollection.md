# C++ Garbage Collection — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Garbage collection (GC) is automatic memory management that reclaims memory occupied by objects that are no longer reachable by the program, without requiring explicit deallocation by the programmer.

**Technical Definition**
C++ provides a *minimal* garbage collection interface (C++11, in `<memory>`) consisting of `std::pointer_safety`, `std::declare_reachable`, `std::undeclare_reachable`, `std::declare_no_pointers`, `std::undeclare_no_pointers`, `std::get_pointer_safety`, and `std::garbage_collect`. This interface **does not require** an implementation to provide a garbage collector; it standardises the *interface* so that programs written with GC in mind remain portable to implementations with and without GC. The C++ standard intentionally does **not** provide a general-purpose garbage-collected heap for ordinary objects (unlike Java or C#). Instead, C++ relies on RAII, smart pointers, and reference counting for deterministic memory management.

**Beginner-Friendly Explanation**
Garbage collection is a system where the computer automatically cleans up memory you no longer use. Languages like Java and C# have full garbage collection built in. C++ does **not**—C++ prefers deterministic cleanup (RAII), where objects clean up after themselves. C++'s GC interface exists mainly to support *optional* collectors and to make certain low-level code safe on GC-enabled platforms. In everyday C++, you use smart pointers instead of a garbage collector.

### Key Characteristics

- **Optional, not required** – the C++ standard permits but does not require a garbage collector.
- **Minimal interface** – the standard provides only a small set of functions for GC-aware code.
- **Deterministic alternatives** – RAII and smart pointers provide deterministic, predictable cleanup.
- **Reference counting** – `std::shared_ptr` implements a form of automatic reclamation (not a tracing GC).
- **No `std::gc`** – there is **no** `std::gc` namespace or class in standard C++; this is a common misconception.
- **Conservative collectors exist** – third-party collectors (Boehm GC, `libgc`) can be used with C++.

### Prerequisites

- Solid understanding of RAII and smart pointers.
- Familiarity with object lifetime and storage duration.
- Knowledge of reference counting (`std::shared_ptr`).
- Awareness of the C++ standard library's memory management facilities.

### Related Programming Areas

- **RAII** – the primary C++ technique for deterministic resource management.
- **Smart pointers** – `unique_ptr`, `shared_ptr`, `weak_ptr`.
- **Allocators** – custom memory management strategies.
- **Reference counting** – the mechanism behind `shared_ptr`.
- **Third-party GC** – Boehm GC, `libgc`, and other conservative collectors.

### Core Concepts / Features

1. Garbage collection (the concept)
2. C++ GC (the standard's minimal interface)
3. `std::gc` (clarification: does not exist)
4. Reference counting
5. Smart pointers
6. GC alternatives
7. GC best practices

---

## 1. Garbage Collection (The Concept)

### Definitions

**Core Definition**
Garbage collection is automatic reclamation of memory that is no longer reachable by the program.

**Technical Definition**
A garbage collector identifies objects that are no longer reachable from roots (stack, globals, registers) and reclaims their memory. Tracing collectors (mark-and-sweep, copying, generational) traverse the object graph; reference counting collectors (like `std::shared_ptr`) reclaim objects when their reference count drops to zero. C++ does not mandate a tracing collector, but the language and library are designed to be *GC-friendly* for implementations that provide one.

**Beginner-Friendly Explanation**
Garbage collection is the “automatic cleanup crew” of a programming language. It finds memory you can no longer reach and frees it. C++ doesn't have this built in—it gives you tools (RAII, smart pointers) to do it yourself deterministically.

### Purposes

- To eliminate manual `delete` and the bugs associated with it (leaks, double frees, use-after-free).
- To simplify memory management in complex object graphs.
- To support languages and runtimes that require GC.
- To enable conservative collectors for legacy or interop code.

### Syntax Rules and Structure

**GC Models**

| Model | Description | C++ Support |
|-------|-------------|-------------|
| Tracing GC | Traverses object graph from roots | Optional (third-party) |
| Reference counting | Frees when count reaches zero | `std::shared_ptr` |
| RAII | Deterministic cleanup via destructors | Core language |

**Constraints and Limitations**
- C++ has no standard tracing GC.
- GC can introduce pauses and non-deterministic timing.
- GC interacts poorly with manual memory management if not carefully integrated.

### Annotated Code Example (Conceptual)

```cpp
// Conceptual comparison: RAII vs. GC
// RAII (C++):
{
    auto p = std::make_unique<int>(42);   // deterministic cleanup
} // freed here

// GC (hypothetical, not standard C++):
// auto p = gcnew int(42);
// // freed when unreachable
```

**Expected Output**  
(No output; conceptual illustration.)

### Real-World Cases

- **Java/C# interop** – C++/CLI and JNI bridge GC and non-GC memory.
- **Scripting engines** – embedding a GC-based language in C++.
- **Legacy systems** – conservative collectors for old codebases.

---

## 2. C++ GC (The Standard's Minimal Interface)

### Definitions

**Core Definition**
C++'s garbage collection interface is a minimal, optional set of functions that allow programs to cooperate with a garbage collector if one is present.

**Technical Definition**
The C++11 standard (and later) defines in `<memory>`: `std::pointer_safety` (an enumeration with `relaxed`, `preferred`, `strict`), `std::get_pointer_safety()`, `std::declare_reachable(T*)`, `std::undeclare_reachable(T*)`, `std::declare_no_pointers(char*, size_t)`, `std::undeclare_no_pointers(char*, size_t)`, and `std::garbage_collect()`. These functions are **no-ops** on implementations without a garbage collector. The standard does not require a collector to exist.

**Beginner-Friendly Explanation**
C++ has a tiny “GC interface” that lets you write code that works whether or not a garbage collector is present. On normal C++ implementations, these functions do nothing.

### Purposes

- To allow GC-aware code to be portable across implementations.
- To let programmers declare reachability and pointer-free regions.
- To support optional collectors without changing the language.
- To provide a standard hook for GC integration.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <memory>

void* p = std::malloc(100);
std::declare_reachable(p);
// ...
std::undeclare_reachable(p);

std::declare_no_pointers(buffer, size);
std::undeclare_no_pointers(buffer, size);

std::pointer_safety ps = std::get_pointer_safety();
std::garbage_collect();
```

**Component Breakdown**

| Function | Purpose |
|----------|---------|
| `declare_reachable(p)` | Tells the GC that `p` is reachable (prevents collection). |
| `undeclare_reachable(p)` | Reverses `declare_reachable`. |
| `declare_no_pointers(p, n)` | Tells the GC that `[p, p+n)` contains no pointers. |
| `undeclare_no_pointers(p, n)` | Reverses `declare_no_pointers`. |
| `get_pointer_safety()` | Returns the pointer safety level. |
| `garbage_collect()` | Requests a collection (implementation-defined). |

**Constraints and Limitations**
- All these functions are **no-ops** on implementations without GC.
- The standard does not require `garbage_collect()` to do anything.
- `declare_reachable`/`undeclare_reachable` must be balanced.

### Annotated Code Example

```cpp
#include <memory>
#include <iostream>

int main() {
    void* p = std::malloc(100);
    std::declare_reachable(p);         // no-op without GC
    // ...
    std::undeclare_reachable(p);
    std::free(p);

    std::cout << "Pointer safety: "
              << static_cast<int>(std::get_pointer_safety()) << '\n';
}
```

**Expected Output** (typical, without GC)
```
Pointer safety: 0
```

**Why this output?**  
On a non-GC implementation, `declare_reachable` is a no-op, and `get_pointer_safety()` returns `pointer_safety::relaxed` (value 0).

### Real-World Cases

- **GC-enabled platforms** – code that must work with a conservative collector.
- **Portable libraries** – avoiding GC hazards without requiring GC.

---

## 3. `std::gc` (Clarification: Does Not Exist)

### Definitions

**Core Definition**
`std::gc` is **not** a standard C++ namespace, class, or function; it does not exist in the C++ standard library.

**Technical Definition**
There is no `std::gc` in the C++ standard. Some languages (e.g., C++/CLI) use `gcnew` and a `System::GC` class, but these are **not** standard C++. The standard's GC interface lives in `<memory>` and consists of the functions listed in §2. Do not expect `std::gc` to compile.

**Beginner-Friendly Explanation**
If you have seen `std::gc` in a tutorial or a code snippet, it is either a mistake, a third-party library, or a non-standard extension (like C++/CLI's `gcnew`). Standard C++ has no `std::gc`.

### Purposes

- To prevent confusion and incorrect code.
- To direct users to the actual standard GC interface.
- To clarify the difference between standard C++ and managed extensions.

### Syntax Rules and Structure

**What Does Exist**

| Facility | Header | Standard? |
|----------|--------|-----------|
| `std::declare_reachable` | `<memory>` | Yes (C++11) |
| `std::garbage_collect` | `<memory>` | Yes (C++11) |
| `gcnew` / `System::GC` | C++/CLI | **No** (non-standard) |
| `std::gc` | — | **No** |

**Constraints and Limitations**
- `std::gc` is not part of any C++ standard.

### Annotated Code Example

```cpp
// This does NOT compile: std::gc does not exist.
// #include <memory>
// int main() { std::gc::collect(); }   // ERROR

// What you can use:
#include <memory>
int main() {
    std::garbage_collect();            // standard, but often a no-op
}
```

**Expected Output**  
(Compiles; `garbage_collect()` is likely a no-op.)

### Real-World Cases

- **Code reviews** – catching non-standard GC usage.
- **Porting** – replacing C++/CLI `gcnew` with standard C++ idioms.

---

## 4. Reference Counting

### Definitions

**Core Definition**
Reference counting is a memory management technique where each object tracks how many references point to it, and the object is destroyed when the count reaches zero.

**Technical Definition**
Reference counting is the mechanism used by `std::shared_ptr`. Each `shared_ptr` increments a control block's strong count on copy and decrements it on destruction. When the strong count reaches zero, the managed object is destroyed. Reference counting is a form of automatic memory management, but it is **not** a tracing garbage collector: it cannot reclaim cycles (see Circular References in the smart pointers sheet).

**Beginner-Friendly Explanation**
Reference counting is like a tally sheet: every time someone takes a shared pointer, the tally goes up; every time someone lets go, the tally goes down. When the tally hits zero, the object is deleted.

### Purposes

- To provide automatic shared ownership.
- To avoid premature destruction while owners still exist.
- To provide deterministic cleanup.
- To serve as a C++ alternative to tracing GC.

### Syntax Rules and Structure

**Key Operations**

| Operation | Effect on Count |
|-----------|-----------------|
| Copy constructor | +1 |
| Copy assignment | +1 (and −1 on the old target) |
| Move constructor | +0 (source becomes empty) |
| Destructor | −1 |
| `reset()` | −1 on the old target |

**Constraints and Limitations**
- Cannot handle cycles.
- Atomic increments/decrements have a performance cost.
- Not a full GC; cycles require `weak_ptr`.

### Annotated Code Example

```cpp
#include <memory>
#include <iostream>

int main() {
    auto sp1 = std::make_shared<int>(100);
    std::cout << sp1.use_count() << '\n';  // 1

    {
        auto sp2 = sp1;                    // +1
        std::cout << sp1.use_count() << '\n'; // 2
    } // -1

    std::cout << sp1.use_count() << '\n';  // 1
}
```

**Expected Output**
```
1
2
1
```

**Why this output?**  
Each copy increments the strong count; each destruction decrements it.

### Real-World Cases

- **Shared caches** – objects kept alive as long as any user holds a `shared_ptr`.
- **Graph algorithms** – nodes shared between multiple edges.

---

## 5. Smart Pointers

### Definitions

**Core Definition**
Smart pointers are RAII wrappers around raw pointers that automatically manage object lifetime.

**Technical Definition**
C++ provides three standard smart pointers: `std::unique_ptr` (exclusive ownership), `std::shared_ptr` (shared ownership via reference counting), and `std::weak_ptr` (non-owning observer). Smart pointers are the primary C++ alternative to garbage collection, providing deterministic, predictable cleanup without a tracing GC.

**Beginner-Friendly Explanation**
Smart pointers are C++'s answer to garbage collection. They clean up automatically when they go out of scope, so you rarely need to call `delete` yourself.

### Purposes

- To eliminate manual `new`/`delete`.
- To provide deterministic cleanup.
- To express ownership semantics clearly.
- To avoid leaks, double frees, and dangling pointers.

### Syntax Rules and Structure

**General Syntax**
```cpp
auto u = std::make_unique<int>(42);       // exclusive
auto s = std::make_shared<int>(10);       // shared
std::weak_ptr<int> w = s;                  // non-owning
```

**Component Breakdown**

| Smart Pointer | Ownership | Copyable |
|---------------|-----------|----------|
| `unique_ptr` | Exclusive | No (move only) |
| `shared_ptr` | Shared | Yes |
| `weak_ptr` | Non-owning | Yes |

**Constraints and Limitations**
- `shared_ptr` cycles leak; use `weak_ptr`.
- `shared_ptr` has reference-counting overhead.

### Annotated Code Example

```cpp
#include <memory>
#include <iostream>

int main() {
    auto up = std::make_unique<int>(42);
    std::cout << *up << '\n';              // 42

    auto sp = std::make_shared<int>(10);
    std::cout << sp.use_count() << '\n';    // 1
}
```

**Expected Output**
```
42
1
```

**Why this output?**  
`unique_ptr` owns `42`; `shared_ptr` owns `10` with one owner.

### Real-World Cases

- **Factory functions** – returning `unique_ptr`.
- **Shared configuration** – `shared_ptr` for shared resources.
- **Pimpl idiom** – `unique_ptr<Impl>`.

---

## 6. GC Alternatives

### Definitions

**Core Definition**
GC alternatives are the C++ techniques—RAII, smart pointers, reference counting, allocators, and third-party collectors—used in place of a standard tracing garbage collector.

**Technical Definition**
C++ relies on deterministic memory management rather than tracing GC. The primary alternatives are: RAII (scope-based cleanup), smart pointers (ownership-aware wrappers), reference counting (`shared_ptr`), custom allocators (pools, arenas), and third-party conservative collectors (Boehm GC, `libgc`). Each has different trade-offs in determinism, performance, and complexity.

**Beginner-Friendly Explanation**
C++ doesn't have built-in garbage collection, but it has many tools that solve the same problems: smart pointers clean up automatically, RAII ties cleanup to scope, and pools/arenas manage memory in bulk.

### Alternatives Comparison

| Alternative | Determinism | Cycles | Overhead | Standard? |
|-------------|-------------|--------|----------|-----------|
| RAII | Deterministic | N/A | Zero | Yes |
| `unique_ptr` | Deterministic | N/A | Zero | Yes |
| `shared_ptr` | Deterministic | No (needs `weak_ptr`) | Atomic refcount | Yes |
| Allocators / pools | Deterministic | N/A | Low | Yes |
| Boehm GC | Non-deterministic | Yes | Tracing overhead | No (third-party) |

**Constraints and Limitations**
- Third-party GC requires integration and may not be portable.
- RAII and smart pointers are the preferred C++ approach.

### Annotated Code Example (Boehm GC, conceptual)

```cpp
// Requires libgc (Boehm GC)
#include <gc/gc.h>

int main() {
    GC_INIT();
    int* p = static_cast<int*>(GC_MALLOC(sizeof(int)));
    *p = 42;
    // No free needed; GC reclaims unreachable memory
}
```

**Expected Output**  
(No output; memory is reclaimed by the collector.)

### Real-World Cases

- **Legacy C code** – conservative GC for old codebases.
- **Embedded scripting** – GC-based language runtimes.

---

## 7. GC Best Practices

### Definitions

**Core Definition**
GC best practices are guidelines for using garbage collection (or its C++ alternatives) correctly, safely, and efficiently.

**Technical Definition**
These practices are informed by the C++ Core Guidelines (R.1–R.11) and community consensus: prefer RAII and smart pointers over GC; avoid raw `new`/`delete`; use `shared_ptr` only for genuine shared ownership; break cycles with `weak_ptr`; use `declare_reachable` only when interoperating with a GC; and avoid relying on `garbage_collect()` for correctness.

**Beginner-Friendly Explanation**
Best practices for GC in C++ are simple: don't use GC—use RAII and smart pointers. If you must interoperate with a GC, follow the rules carefully.

### Key Guidelines

| Guideline | Summary |
|-----------|---------|
| **Prefer RAII** | Deterministic cleanup is the C++ way. |
| **Use smart pointers** | `unique_ptr` by default; `shared_ptr` for sharing. |
| **Avoid cycles** | Use `weak_ptr` to break `shared_ptr` cycles. |
| **Use `declare_reachable` sparingly** | Only when interoperating with a GC. |
| **Do not rely on `garbage_collect()`** | It may be a no-op. |
| **Use custom allocators for performance** | Pools and arenas reduce overhead. |
| **Benchmark before adopting GC** | Third-party GC adds complexity and non-determinism. |

### Annotated Code Example

```cpp
#include <memory>
#include <iostream>

int main() {
    // Preferred: RAII + smart pointers
    auto data = std::make_unique<int[]>(100);
    data[0] = 42;
    std::cout << data[0] << '\n';          // 42
    // Memory freed automatically
}
```

**Expected Output**
```
42
```

**Why this output?**  
`unique_ptr` manages the array; no explicit `delete[]` is needed, and no GC is involved.

### Real-World Cases

- **Code reviews** – flagging manual `new`/`delete`.
- **Static analysis** – clang-tidy `cppcoreguidelines-owning-memory`.
- **Library design** – exposing RAII-based interfaces.

---

## References

- Garbage collection support — cppreference – https://en.cppreference.com/w/cpp/memory/gc
- `std::pointer_safety` — cppreference – https://en.cppreference.com/w/cpp/memory/gc/pointer_safety
- `std::declare_reachable` — cppreference – https://en.cppreference.com/w/cpp/memory/gc/declare_reachable
- `std::garbage_collect` — cppreference – https://en.cppreference.com/w/cpp/memory/gc/garbage_collect
- C++ Core Guidelines — Resource management – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-resource
- Boehm-Demers-Weiser conservative garbage collector – https://www.hboehm.info/gc/
- Smart pointers — Microsoft Learn – https://learn.microsoft.com/en-us/cpp/cpp/smart-pointers-modern-cpp
- `std::shared_ptr` — cppreference – https://en.cppreference.com/w/cpp/memory/shared_ptr
- Garbage collection (computer science) — Wikipedia – https://en.wikipedia.org/wiki/Garbage_collection_(computer_science)
- C++/CLI `gcnew` — Microsoft Learn – https://learn.microsoft.com/en-us/cpp/extensions/handle-to-object-operator-hat-cpp-component-extensions
