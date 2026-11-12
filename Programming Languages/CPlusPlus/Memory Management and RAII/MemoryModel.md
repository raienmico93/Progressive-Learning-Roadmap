# C++ Memory Model — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
The C++ memory model is a specification that defines how objects are laid out in memory, how their lifetimes are managed, and how concurrent threads observe and interact with shared memory. It is the contract between the programmer, the compiler, and the hardware that makes reasoning about memory and concurrency possible.

**Technical Definition**
The C++ memory model, formalized in C++11 and refined in subsequent standards, consists of two major parts: the **object model** (structural), which specifies how objects are represented, aligned, stored, and kept alive in memory, and the **concurrency model**, which specifies the rules for data races, atomic operations, and memory visibility across threads. Together they define the set of allowed executions of a C++ program, including the happens-before relationships between operations.

**Beginner-Friendly Explanation**
The memory model is C++'s rulebook for how memory works. It tells you where variables live, how long they live, what their bytes look like, and—crucially—what different threads can and cannot see when they read and write shared data. Without these rules, compilers and CPUs could reorder or eliminate memory accesses in ways that break your programs.

### Key Characteristics

- **Object-centric** – C++ programs create, destroy, refer to, access, and manipulate *objects*; every object has a size, alignment, storage duration, lifetime, type, and value.
- **Storage duration determines lifetime** – objects have automatic, static, dynamic, or thread-local storage duration, which bounds their lifetime.
- **Lifetime is a runtime property** – every object has a moment when its lifetime begins and ends; using an object outside its lifetime is undefined behaviour.
- **Type-based aliasing rules** – accessing an object through a glvalue of a different type is generally undefined behaviour, enabling type-based alias analysis.
- **Happens-before defines visibility** – the memory model defines a happens-before relation that determines which writes a thread is guaranteed to observe.
- **Atomics and memory orders** – `std::atomic` and `std::memory_order` provide fine-grained control over memory visibility and instruction reordering.
- **Fences provide standalone ordering** – `std::atomic_thread_fence` orders all memory operations across it, not just a single atomic variable.

### Prerequisites

- Basic C++ syntax (variables, pointers, references).
- Understanding of the stack vs. the heap.
- Familiarity with `new`/`delete` and smart pointers.
- Basic knowledge of threads and data races (for the concurrency sections).

### Related Programming Areas

- **Object-oriented programming** – object lifetime and construction/destruction.
- **Concurrency and parallelism** – data races, atomics, and memory ordering.
- **Compiler optimisation** – strict aliasing, type-based alias analysis, and reordering.
- **Low-level programming** – alignment, `reinterpret_cast`, and `std::bit_cast`.
- **Resource management** – RAII and storage duration.

### Core Concepts / Features

1. Memory model (overarching framework)
2. Object model
3. Storage duration
4. Object lifetime
5. Object representation
6. Alignment
7. Strict aliasing
8. Effective type
9. Memory ordering
10. Memory barriers
11. Memory model best practices

---

## 1. Memory Model (Overarching Framework)

### Definitions

**Core Definition**
The C++ memory model is the specification of how memory accesses behave, especially in multi-threaded programs, defining the set of allowed executions and the visibility guarantees between threads.

**Technical Definition**
The C++ memory model (since C++11) defines memory locations, data races, atomic operations, and a happens-before relation. A data race occurs when two threads access the same memory location concurrently, at least one access is a write, and neither is atomic; a data race results in undefined behaviour. The model provides `std::atomic` types and `std::memory_order` operations to avoid data races and control visibility.

**Beginner-Friendly Explanation**
The memory model is the big-picture rulebook: it says that if two threads touch the same memory without proper synchronisation, the program is broken (undefined behaviour), but if you use atomics correctly, the program has well-defined behaviour.

### Purposes

- To define which program executions are valid in the presence of threads.
- To provide a foundation for data-race-free concurrent programming.
- To allow compilers to optimise while preserving observable behaviour.
- To abstract over different hardware memory models (x86-TSO, ARM weak, etc.).

### Syntax Rules and Structure

**Key Concepts**

| Concept | Description |
|---------|-------------|
| Memory location | An object of scalar type, or a maximal sequence of adjacent bit-fields. |
| Data race | Concurrent conflicting accesses without synchronisation → undefined behaviour. |
| Happens-before | A relation that determines visibility: if A happens-before B, B sees A's effects. |
| Synchronises-with | An atomic release store synchronises with an acquire load of the same variable. |
| Sequenced-before | A single-thread ordering of operations. |

**Constraints and Limitations**
- The memory model is a specification, not an implementation; compilers and hardware must conform.
- Different hardware architectures implement different (weaker or stronger) memory models.

### Annotated Code Example

```cpp
#include <atomic>
#include <thread>
#include <cassert>

std::atomic<int> flag{0};
int data = 0;

void producer() {
    data = 42;                         // non-atomic write
    flag.store(1, std::memory_order_release);  // release: publishes data
}

void consumer() {
    while (flag.load(std::memory_order_acquire) != 1) ;  // acquire: sees data
    assert(data == 42);                // guaranteed to see 42
}

int main() {
    std::thread t1(producer), t2(consumer);
    t1.join(); t2.join();
}
```

**Expected Output**  
(No output; assertion passes.)

**Why this output?**  
The release store to `flag` happens-before the acquire load from `flag`, so the write to `data` is visible to the consumer. Without the memory orders, this would be a data race.

### Real-World Cases

- **Lock-free data structures** – designing concurrent queues and stacks.
- **Thread synchronisation** – ensuring visibility of shared data.
- **Cross-platform concurrency** – writing code that works on x86 and ARM.

---

## 2. Object Model

### Definitions

**Core Definition**
The object model specifies that C++ programs create, destroy, refer to, access, and manipulate objects, where an object is a region of storage with a type, size, alignment, storage duration, lifetime, and value.

**Technical Definition**
An object in C++ has: size (via `sizeof`), alignment requirement (via `alignof`), storage duration (automatic, static, dynamic, thread-local), lifetime (bounded by storage duration or temporary), type, value (which may be indeterminate), and optionally a name. The following are *not* objects: values, references, functions, enumerators, types, non-static class members, templates, namespaces, parameter packs, and `this`.

**Beginner-Friendly Explanation**
Everything in C++ that occupies memory is an object (in the technical sense). A variable is an object, a `new`-allocated buffer is an object, and even a temporary `std::string` is an object. References are *not* objects; they are aliases.

### Purposes

- To provide a uniform abstraction for memory-resident entities.
- To define the properties (size, alignment, lifetime) that every object has.
- To distinguish objects from non-object entities (references, functions, types).

### Syntax Rules and Structure

**Object Properties**

| Property | Determined By |
|----------|---------------|
| Size | `sizeof(obj)` |
| Alignment | `alignof(T)` |
| Storage duration | Declaration (`auto`, `static`, `thread_local`, `new`) |
| Lifetime | Runtime property |
| Type | The declared or deduced type |
| Value | May be indeterminate |

**Syntax Rules**
- Objects are created by definitions, `new`-expressions, `throw`-expressions, union member activation, and temporary object creation.
- Objects of implicit-lifetime types can be implicitly created by certain operations (e.g., `malloc`, `std::memcpy`).

**Constraints and Limitations**
- References are not objects; they have no size or address of their own.
- Functions and types are not objects.

### Annotated Code Example

```cpp
#include <iostream>
#include <cstddef>

struct Point { int x, y; };

int main() {
    Point p{1, 2};                    // p is an object
    int& r = p.x;                     // r is a reference, not an object
    std::cout << sizeof(p) << '\n';   // size of Point
    std::cout << alignof(Point) << '\n'; // alignment requirement
}
```

**Expected Output** (typical)
```
8
4
```

**Why this output?**  
`Point` has two `int` members (8 bytes total on typical platforms) and alignment 4 (alignof `int`).

### Real-World Cases

- **Serialisation** – treating objects as byte sequences.
- **Generic programming** – writing code that works with any object type.
- **Memory debugging** – understanding object layout.

---

## 3. Storage Duration

### Definitions

**Core Definition**
Storage duration determines how long the storage for an object remains allocated: automatic (block scope), static (program duration), dynamic (until explicitly freed), or thread-local (thread duration).

**Technical Definition**
C++ defines four storage durations: **automatic** (allocated at the beginning of a block, deallocated at the end), **static** (allocated when the program begins, deallocated when it ends), **thread** (allocated when the thread begins, deallocated when it ends; since C++11), and **dynamic** (allocated by `new`, deallocated by `delete` or the end of the program). Storage duration is associated with objects introduced by declarations and with temporary objects.

**Beginner-Friendly Explanation**
Storage duration answers: “How long does this variable’s memory stick around?” A local variable lives until its block ends (automatic). A global lives for the whole program (static). A thread-local lives for the thread. A `new`-allocated object lives until you `delete` it (dynamic).

### Purposes

- To determine when storage is allocated and deallocated.
- To bound an object’s lifetime.
- To control memory usage and resource management.

### Syntax Rules and Structure

**Storage Duration Summary**

| Duration | Allocated | Deallocated | Example |
|----------|-----------|-------------|---------|
| Automatic | Block entry | Block exit | `int x;` inside a function |
| Static | Program start | Program end | `static int x;`, global |
| Thread-local | Thread start | Thread end | `thread_local int x;` |
| Dynamic | `new` | `delete` or program end | `new int` |

**Syntax Rules**
- `static` and `thread_local` can be combined.
- `auto` (storage class specifier) is deprecated; automatic duration is the default for block-scope variables.
- Dynamic storage duration is associated with `new`-expressions.

**Constraints and Limitations**
- The lifetime of an object is equal to or nested within the lifetime of its storage.
- Using an object outside its lifetime is undefined behaviour.

### Annotated Code Example

```cpp
#include <iostream>

int global = 1;                        // static storage duration

void f() {
    int local = 2;                     // automatic storage duration
    static int static_local = 3;       // static storage duration
    thread_local int tl = 4;           // thread-local storage duration
    std::cout << local << ' ' << static_local << ' ' << tl << '\n';
}

int main() {
    f();
    int* dyn = new int(5);             // dynamic storage duration
    std::cout << global << ' ' << *dyn << '\n';
    delete dyn;
}
```

**Expected Output**
```
2 3 4
1 5
```

**Why this output?**  
Each variable has a different storage duration, but all are valid within their respective scopes.

### Real-World Cases

- **Singleton patterns** – static storage duration for global state.
- **Thread-local caches** – `thread_local` for per-thread data.
- **RAII** – automatic storage duration for resource management.

---

## 4. Object Lifetime

### Definitions

**Core Definition**
Object lifetime is the runtime period during which an object exists; it begins when storage with the proper alignment and size is obtained and initialization completes, and ends when the object is destroyed or its storage is released or reused.

**Technical Definition**
The lifetime of an object of type `T` begins when: (1) storage with the proper alignment and size for `T` is obtained, and (2) if `T` is a class type with a non-trivial constructor, the constructor call has completed. The lifetime ends when: (1) for non-class types, the object is destroyed, or (2) for class types, the destructor call begins, or (3) the storage is released or reused by another object. The lifetime of an object is equal to or nested within the lifetime of its storage.

**Beginner-Friendly Explanation**
Object lifetime is the “alive” period of an object. It starts when the object is fully constructed and ends when it is destroyed or its memory is reused. Using an object after its lifetime ends (a dangling pointer) is undefined behaviour.

### Purposes

- To define when an object can be safely accessed.
- To bound the validity of pointers and references.
- To enable resource management (RAII).

### Syntax Rules and Structure

**Lifetime Begins**
- Storage obtained (proper size and alignment).
- Initialization complete (constructor finished, if applicable).

**Lifetime Ends**
- Non-class type: object destroyed.
- Class type: destructor call begins.
- Storage released or reused.

**Special Cases**
- Union member lifetime begins when the member is activated.
- Implicit-lifetime types can be implicitly created by operations like `malloc`.

**Constraints and Limitations**
- Using an object outside its lifetime (dangling pointer, use-after-free) is undefined behaviour.
- References can outlive the objects they refer to, creating dangling references.

### Annotated Code Example

```cpp
#include <iostream>

struct Widget {
    Widget() { std::cout << "Constructed\n"; }
    ~Widget() { std::cout << "Destroyed\n"; }
};

int main() {
    {
        Widget w;                      // lifetime begins
    }                                  // lifetime ends (destructor called)

    Widget* p = new Widget();          // lifetime begins
    delete p;                          // lifetime ends
}
```

**Expected Output**
```
Constructed
Destroyed
Constructed
Destroyed
```

**Why this output?**  
The automatic `Widget` is constructed at block entry and destroyed at block exit. The dynamic `Widget` is constructed by `new` and destroyed by `delete`.

### Real-World Cases

- **RAII** – tying resource lifetime to object lifetime.
- **Smart pointers** – managing dynamic object lifetime.
- **Placement new** – explicitly controlling lifetime.

---

## 5. Object Representation

### Definitions

**Core Definition**
The object representation of an object is the sequence of `unsigned char` bytes it occupies, while the value representation is the subset of those bits that actually hold the object's value.

**Technical Definition**
The **object representation** of an object of type `T` is the sequence of `N` `unsigned char` objects taken up by the object, where `N == sizeof(T)`. The **value representation** is the set of bits that hold the value of type `T`. Bits in the object representation that are not part of the value representation are **padding bits**. For `char`, `signed char`, and `unsigned char`, every bit of the object representation participates in the value representation (no padding allowed).

**Beginner-Friendly Explanation**
The object representation is the raw bytes of an object. The value representation is the meaningful bits within those bytes. Padding bits are the “wasted” bits that don't affect the value—often inserted to satisfy alignment requirements.

### Purposes

- To define what it means to copy an object byte-for-byte.
- To enable `memcpy`-based copying for trivially copyable types.
- To explain padding and object size.

### Syntax Rules and Structure

**Component Breakdown**

| Concept | Description |
|---------|-------------|
| Object representation | `sizeof(T)` bytes. |
| Value representation | Bits that participate in the value. |
| Padding bits | Bits not part of the value representation. |

**Syntax Rules**
- For trivially copyable types, copying the object representation produces an object with the same value.
- `char`, `signed char`, and `unsigned char` have no padding bits.
- `std::byte` (C++17) can also be used to inspect object representation.

**Constraints and Limitations**
- Reading padding bits via `unsigned char*` is allowed, but the values of padding bits are unspecified.
- Writing to padding bits may not preserve the value.

### Annotated Code Example

```cpp
#include <iostream>
#include <cstring>

struct S {
    char c;                            // 1 byte
    int i;                             // 4 bytes (typically)
    // 3 bytes of padding between c and i
};

int main() {
    S s{'a', 42};
    unsigned char bytes[sizeof(S)];
    std::memcpy(bytes, &s, sizeof(S));

    std::cout << sizeof(S) << '\n';    // typically 8
    std::cout << (int)bytes[0] << '\n'; // 'a' (97)
}
```

**Expected Output** (typical)
```
8
97
```

**Why this output?**  
`S` has size 8 due to padding after `c`; `memcpy` copies the object representation byte-for-byte.

### Real-World Cases

- **Serialisation** – writing objects to binary files.
- **Networking** – sending raw object bytes.
- **Checksumming** – hashing object representations.

---

## 6. Alignment

### Definitions

**Core Definition**
Alignment is a restriction on the memory addresses at which an object can be stored; an object of type `T` must be placed at an address that is a multiple of its alignment requirement.

**Technical Definition**
Each object type imposes an alignment requirement on every object of that type. The alignment requirement of a type can be queried with `alignof` or `std::alignment_of`. Since C++11, stricter alignment (a larger alignment requirement) can be requested using the `alignas` specifier. `alignof` returns the greatest power of 2 to which something is aligned.

**Beginner-Friendly Explanation**
Alignment means that some types must be stored at addresses that are multiples of a certain number (e.g., a 4-byte `int` must start at an address divisible by 4). Misaligned access can be slow or even illegal on some hardware.

### Purposes

- To ensure efficient memory access.
- To satisfy hardware requirements.
- To control memory layout for performance or interoperability.

### Syntax Rules and Structure

**General Syntax**
```cpp
alignas(N) T obj;                      // request N-byte alignment
std::size_t a = alignof(T);            // query alignment requirement
```

**Component Breakdown**

| Specifier/Operator | Meaning |
|--------------------|---------|
| `alignas(N)` | Requests alignment of at least `N` bytes. |
| `alignof(T)` | Returns the alignment requirement of `T`. |
| `std::align` | Adjusts a pointer for alignment (C++11). |

**Syntax Rules**
- `alignas` cannot reduce alignment below the natural alignment.
- `alignas` can take a type or a constant expression.
- `alignof` returns a `std::size_t`.

**Constraints and Limitations**
- Over-alignment may waste memory.
- Misaligned access is undefined behaviour for some types.

### Annotated Code Example

```cpp
#include <iostream>
#include <cstddef>

struct alignas(16) Vec4 {
    float x, y, z, w;
};

int main() {
    std::cout << alignof(Vec4) << '\n';    // 16
    std::cout << sizeof(Vec4) << '\n';     // 16
    std::cout << alignof(int) << '\n';     // typically 4
}
```

**Expected Output**
```
16
16
4
```

**Why this output?**  
`alignas(16)` forces `Vec4` to be 16-byte aligned; `sizeof` is also 16.

### Real-World Cases

- **SIMD programming** – 16- or 32-byte alignment for vector types.
- **Cache-line alignment** – avoiding false sharing.
- **Hardware registers** – specific alignment requirements.

---

## 7. Strict Aliasing

### Definitions

**Core Definition**
The strict aliasing rule says that an object of type `T` may only be accessed through a glvalue of a type that is *similar* to `T`, a signed/unsigned variant of `T`, or `char`/`unsigned char`/`std::byte`; otherwise the behaviour is undefined.

**Technical Definition**
Whenever an attempt is made to read or modify the value of an object of type `DynamicType` through a glvalue of type `AliasedType`, the behaviour is undefined unless one of the following is true: (1) `AliasedType` and `DynamicType` are *similar* (same type after stripping cv-qualifications at every level), (2) `AliasedType` is the signed or unsigned variant of `DynamicType`, or (3) `AliasedType` is `std::byte` (since C++17), `char`, or `unsigned char`. This rule enables **type-based alias analysis**, in which the compiler assumes that writes through one type do not affect objects of a different type.

**Beginner-Friendly Explanation**
Strict aliasing means you cannot pretend a `float` is an `int` by casting its pointer and dereferencing it. The compiler assumes pointers to different types never point to the same memory, and violating this assumption leads to undefined behaviour.

### Purposes

- To enable type-based alias analysis and optimisation.
- To define the conditions under which type punning is safe.
- To prevent subtle miscompilations.

### Syntax Rules and Structure

**Allowed Aliasing**

| Aliased Type | Allowed? |
|--------------|----------|
| Same type (cv-qualified) | Yes |
| Signed/unsigned variant | Yes |
| `char`, `unsigned char`, `std::byte` | Yes (to inspect object representation) |
| Unrelated type | No (undefined behaviour) |

**Safe Type Punning Alternatives**
- `std::memcpy` (C++11) – copy bytes between objects.
- `std::bit_cast` (C++20) – bitwise reinterpretation.

**Constraints and Limitations**
- `reinterpret_cast` between unrelated pointer types and dereferencing is undefined behaviour.
- Many compilers relax the rule, but relying on that is non-portable.

### Annotated Code Example

```cpp
#include <cstring>
#include <iostream>

int main() {
    float f = 3.14f;
    // BAD: strict aliasing violation
    // int i = *reinterpret_cast<int*>(&f);

    // GOOD: memcpy
    int i;
    std::memcpy(&i, &f, sizeof(i));
    std::cout << std::hex << i << '\n';
}
```

**Expected Output** (IEEE 754 representation)
```
4048f5c3
```

**Why this output?**  
`memcpy` copies the object representation without violating strict aliasing; `reinterpret_cast` would have been undefined behaviour.

### Real-World Cases

- **Serialisation** – using `memcpy` for byte-level copying.
- **Low-level programming** – inspecting object representations.
- **Compiler optimisation** – understanding why type-based alias analysis matters.

---

## 8. Effective Type

### Definitions

**Core Definition**
The effective type of an object is the type used for accesses to its stored value, which is either the object's declared type or the type determined by the most recent store to it (in C; C++ primarily relies on the declared type and strict aliasing).

**Technical Definition**
In C (and adopted conceptually in C++ discussions), the **effective type** of an object for an access to its stored value is the declared type of the object, if any; for objects with no declared type (e.g., `malloc`'d memory), the effective type is the type of the lvalue used for the most recent store. C++ does not formally use the term “effective type” in the same way; instead, it relies on the declared type and the strict aliasing rule. However, the concept is relevant when interoperating with C and when reasoning about dynamically allocated memory.

**Beginner-Friendly Explanation**
The effective type is “the type that memory currently holds.” If you `malloc` a block and write an `int` to it, the effective type becomes `int`. Writing a `float` later may change it (in C). In C++, the rules are stricter and based on declared types.

### Purposes

- To determine which type may be used to access memory.
- To understand C/C++ interoperability.
- To reason about type punning and aliasing.

### Syntax Rules and Structure

**Effective Type Rules (C-style)**

| Situation | Effective Type |
|-----------|----------------|
| Declared object | Declared type. |
| `malloc`'d memory, no store yet | No effective type. |
| `malloc`'d memory, after store | Type of the store. |
| `memcpy`'d memory | Source's effective type (copied). |

**C++ Note**
- C++ primarily uses declared types and the strict aliasing rule.
- The term “effective type” appears in C and in discussions of low-level memory.

### Annotated Code Example

```cpp
#include <cstdlib>
#include <cstring>
#include <iostream>

int main() {
    void* mem = std::malloc(sizeof(int));
    // effective type is "none" until a store
    *static_cast<int*>(mem) = 42;      // effective type becomes int
    std::cout << *static_cast<int*>(mem) << '\n';
    std::free(mem);
}
```

**Expected Output**
```
42
```

**Why this output?**  
After the store, the effective type of the `malloc`'d memory is `int`, so reading it as `int` is valid.

### Real-World Cases

- **C interoperability** – understanding C's effective type rules.
- **Dynamic memory** – reasoning about type-punning in `malloc`'d buffers.

---

## 9. Memory Ordering

### Definitions

**Core Definition**
Memory ordering specifies the constraints on how atomic operations and surrounding memory accesses may be reordered by the compiler and hardware, ranging from `relaxed` (no ordering) to `seq_cst` (sequential consistency).

**Technical Definition**
`std::memory_order` is an enumeration with six values: `relaxed`, `consume`, `acquire`, `release`, `acq_rel`, and `seq_cst`. Each atomic operation carries a memory order that determines its synchronisation properties. `relaxed` guarantees only atomicity; `acquire`/`release` establish happens-before relationships; `seq_cst` provides a single total order of all sequentially consistent operations.

**Beginner-Friendly Explanation**
Memory ordering controls how much freedom the compiler and CPU have to reorder memory operations. `relaxed` means “do whatever, just keep the atomic operation itself atomic.” `acquire`/`release` means “make sure the operations before/after this one are visible in the right order.” `seq_cst` is the strictest and easiest to reason about, but also the slowest.

### Purposes

- To control visibility of memory writes across threads.
- To balance performance and correctness in concurrent code.
- To reason about happens-before relationships.

### Syntax Rules and Structure

**Memory Order Summary**

| Order | Semantics | Typical Use |
|-------|-----------|-------------|
| `relaxed` | Atomicity only; no ordering | Counters, statistics |
| `consume` | Data dependency ordering (discouraged) | Rarely used |
| `acquire` | Prevents later reads/writes from moving before | Load side of a lock |
| `release` | Prevents earlier reads/writes from moving after | Store side of a lock |
| `acq_rel` | Both acquire and release | Read-modify-write operations |
| `seq_cst` | Total order across all threads | Default; easiest to reason about |

**Syntax Rules**
- Atomic operations default to `seq_cst` unless otherwise specified.
- `acquire` is for loads; `release` is for stores; `acq_rel` is for read-modify-write operations.
- `consume` is discouraged due to implementation difficulties.

**Constraints and Limitations**
- Weak memory orders can lead to subtle bugs if not used carefully.
- `seq_cst` is the safest but may be slower on some architectures.

### Annotated Code Example

```cpp
#include <atomic>
#include <thread>
#include <cassert>

std::atomic<int> flag{0};
int data = 0;

void producer() {
    data = 42;
    flag.store(1, std::memory_order_release);   // release
}

void consumer() {
    while (flag.load(std::memory_order_acquire) != 1) ; // acquire
    assert(data == 42);
}

int main() {
    std::thread t1(producer), t2(consumer);
    t1.join(); t2.join();
}
```

**Expected Output**  
(No output; assertion passes.)

**Why this output?**  
The release store and acquire load establish happens-before, making `data = 42` visible to the consumer.

### Real-World Cases

- **Lock-free queues** – using `acquire`/`release` for head/tail pointers.
- **Reference counting** – using `relaxed` for increments, `acq_rel` for decrements.
- **Publishing data** – `release` store to a flag after writing data.

---

## 10. Memory Barriers

### Definitions

**Core Definition**
A memory barrier (fence) is an instruction that enforces ordering constraints on memory operations, preventing the compiler and CPU from reordering accesses across the barrier.

**Technical Definition**
`std::atomic_thread_fence(order)` is a standalone fence that orders all memory operations across it, not just a single atomic variable. Unlike per-variable memory ordering, which annotates individual atomic operations, a fence applies to all prior and subsequent memory operations in the same thread. Fence types correspond to memory orders: acquire, release, acq_rel, and seq_cst.

**Beginner-Friendly Explanation**
A memory barrier is like a wall that memory operations cannot cross. Operations before the wall stay before; operations after stay after. This prevents the CPU and compiler from reordering memory accesses across the barrier.

### Purposes

- To order multiple memory operations without per-variable annotations.
- To batch synchronisation (one fence instead of many ordered atomics).
- To separate data access from synchronisation logic.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::atomic_thread_fence(std::memory_order_release);
std::atomic_thread_fence(std::memory_order_acquire);
```

**Fence Types**

| Fence | Effect |
|-------|--------|
| `acquire` | Prevents reads/writes after the fence from moving before it. |
| `release` | Prevents reads/writes before the fence from moving after it. |
| `acq_rel` | Both. |
| `seq_cst` | Total order. |

**Constraints and Limitations**
- Fences are rarely needed when per-variable ordering suffices.
- `seq_cst` fences are expensive.

### Annotated Code Example

```cpp
#include <atomic>
#include <thread>
#include <cassert>

std::atomic<int> flag{0};
int data1 = 0, data2 = 0;

void producer() {
    data1 = 10;
    data2 = 20;
    std::atomic_thread_fence(std::memory_order_release);  // fence
    flag.store(1, std::memory_order_relaxed);
}

void consumer() {
    while (flag.load(std::memory_order_relaxed) != 1) ;
    std::atomic_thread_fence(std::memory_order_acquire);   // fence
    assert(data1 == 10 && data2 == 20);
}

int main() {
    std::thread t1(producer), t2(consumer);
    t1.join(); t2.join();
}
```

**Expected Output**  
(No output; assertions pass.)

**Why this output?**  
The release fence orders both `data1` and `data2` writes before the relaxed flag store; the acquire fence orders the reads after the relaxed flag load.

### Real-World Cases

- **Batching synchronisation** – one fence for multiple relaxed operations.
- **Hardware interop** – mapping to `DMB` (ARM) or `LWSYNC` (POWER).
- **Separating concerns** – data code uses relaxed; synchronisation code adds fences.

---

## 11. Memory Model Best Practices

### Definitions

**Core Definition**
Memory model best practices are guidelines for writing correct, efficient, and maintainable code that respects the C++ object model and memory model, especially in concurrent programs.

**Technical Definition**
These practices are informed by the C++ Core Guidelines (especially the concurrency and resource management sections), which emphasise RAII, avoiding raw `new`/`delete`, preferring `seq_cst` until proven otherwise, and using tools to validate concurrent code. Key rules include CP.8 (“Don’t try to use `volatile` for synchronization”), CP.9 (“Whenever feasible use tools to validate your concurrent code”), and CP.100 (“Don’t use lock-free programming unless you absolutely have to”).

**Beginner-Friendly Explanation**
The best practices boil down to: use RAII, don’t use `volatile` for synchronisation, start with `seq_cst` atomics, and get your lock-free code reviewed and tested—it’s much harder than it looks.

### Key Guidelines

| Guideline | Summary |
|-----------|---------|
| **CP.8** | Don’t use `volatile` for synchronisation. |
| **CP.9** | Use tools (ThreadSanitizer, etc.) to validate concurrent code. |
| **CP.100** | Don’t use lock-free programming unless you absolutely have to. |
| **CP.101** | Distrust your hardware/compiler combination; verify assumptions. |
| **Use RAII** | Tie resource lifetime to object lifetime. |
| **Prefer `seq_cst`** | Start with sequential consistency; relax only with justification. |
| **Avoid raw owning pointers** | Use smart pointers (R.20–R.23). |
| **Use `std::atomic`** | Never use plain data for cross-thread communication. |
| **Avoid type punning** | Use `memcpy` or `std::bit_cast` instead of `reinterpret_cast`. |
| **Respect alignment** | Use `alignas` and `alignof` when necessary. |

### Annotated Code Example

```cpp
#include <atomic>
#include <thread>
#include <cassert>

std::atomic<int> counter{0};

void increment() {
    for (int i = 0; i < 1000; ++i)
        counter.fetch_add(1, std::memory_order_relaxed);
}

int main() {
    std::thread t1(increment), t2(increment);
    t1.join(); t2.join();
    assert(counter.load() == 2000);
}
```

**Expected Output**  
(No output; assertion passes.)

**Why this output?**  
`std::atomic` with `relaxed` ordering provides atomicity for the counter; no data race occurs. For a simple counter, `relaxed` is sufficient because no other memory needs ordering.

### Real-World Cases

- **Code reviews** – checking for data races and aliasing violations.
- **Static analysis** – tools like ThreadSanitizer and clang-tidy.
- **Performance tuning** – relaxing memory orders where justified.

---

## References

- Objects — cppreference – https://en.cppreference.com/w/cpp/language/object
- Object lifetime — cppreference – https://en.cppreference.com/w/cpp/language/lifetime
- Storage duration — cppreference – https://en.cppreference.com/w/cpp/language/storage_duration
- Strict aliasing — cppreference – https://en.cppreference.com/w/cpp/language/reinterpret_cast#Type_aliasing
- `std::memory_order` — cppreference – https://en.cppreference.com/w/cpp/atomic/memory_order
- `std::atomic_thread_fence` — cppreference – https://en.cppreference.com/w/cpp/atomic/atomic_thread_fence
- Alignment — Microsoft Learn – https://learn.microsoft.com/en-us/cpp/cpp/alignment
- C++ Core Guidelines — Concurrency – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-concurrency
- C++ Core Guidelines — Resource management – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-resource
- Understand the C++ memory model and happens-before relationships — Dr-Sergey/learn_cpp – https://github.com/Dr-Sergey/learn_cpp/blob/main/10_Concurrency_and_Parallelism/Understand_the_C%2B%2B_memory_model_and_happens-before_relationships.md
- Understand memory fences and `std::atomic_thread_fence` — Dr-Sergey/learn_cpp – https://github.com/Dr-Sergey/learn_cpp/blob/main/10_Concurrency_and_Parallelism/Understand_memory_fences_and_stdatomic_thread_fence.md
- Understand strict aliasing and type punning safely — Dr-Sergey/learn_cpp – https://github.com/Dr-Sergey/learn_cpp/blob/main/09_Standard_Library_Utilities/Understand_strict_aliasing_and_type_punning_safely.md
- Effective types: examples (P1796R0) — Open Standards – https://cdn.jsdelivr.net
- Memory Models for C/C++ Programmers — arXiv – https://ar5iv.labs.arxiv.org

---

This cheat sheet provides a comprehensive, structured reference for the C++ memory model, covering the object model, storage duration, object lifetime, representation, alignment, strict aliasing, effective type, memory ordering, memory barriers, and best practices, with annotated code examples, expected outputs, real-world cases, and authoritative references.