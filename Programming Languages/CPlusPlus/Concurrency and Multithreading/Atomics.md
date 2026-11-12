# C++ Atomics — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Atomics are types and operations that guarantee indivisible, data-race-free access to shared data across threads, forming the foundation of lock-free concurrent programming in C++.

**Technical Definition**
The `<atomic>` header (since C++11) provides the `std::atomic<T>` class template and related facilities. An atomic operation is one that is guaranteed to be indivisible—no other thread can observe it in a partially completed state. Atomics also support **memory ordering** specifications that control how surrounding memory operations may be reordered by the compiler and hardware. `std::atomic_flag` is the only guaranteed lock-free atomic type; `std::atomic<T>` is lock-free for most trivially copyable types on most platforms. C++20 adds `std::atomic_ref<T>`, which provides atomic operations on non-atomic objects.

**Beginner-Friendly Explanation**
An atomic is like a “one-at-a-time” variable: no matter how many threads try to read or write it simultaneously, each operation happens as a single indivisible step. This prevents data races and is the building block for lock-free data structures. Memory ordering controls how much the compiler and CPU are allowed to reorder memory operations around the atomic operation.

### Key Characteristics

- **Data-race-free** – atomic operations on the same variable do not cause data races.
- **Indivisible** – no thread can observe a partially completed atomic operation.
- **Lock-free (usually)** – most atomic types are lock-free on most platforms; `std::atomic_flag` is guaranteed lock-free.
- **Memory ordering** – six memory orders control visibility and reordering.
- **Non-copyable** – atomic types cannot be copied or moved.
- **Specialisations** – `std::atomic<bool>`, `std::atomic<int>`, `std::atomic<T*>`, and `std::atomic<std::shared_ptr<T>>` (C++20) have additional operations.
- **`std::atomic_ref`** – provides atomic access to non-atomic objects (C++20).

### Prerequisites

- Solid understanding of threads and the memory model.
- Familiarity with data races and happens-before.
- Knowledge of mutexes and condition variables (for comparison).
- Understanding of compiler and hardware reordering.

### Related Programming Areas

- **Memory model** – atomics define happens-before relationships.
- **Lock-free programming** – atomics are the building blocks.
- **Mutexes** – atomics are an alternative for simple shared state.
- **Reference counting** – `std::shared_ptr` uses atomics internally.
- **Concurrency primitives** – latches, barriers, and semaphores may use atomics.

### Core Concepts / Features

1. `std::atomic`
2. Atomic types
3. Atomic operations
4. Memory ordering
5. `std::atomic_flag`
6. `std::atomic_ref` (C++20)
7. Atomic best practices

---

## 1. `std::atomic`

### Definitions

**Core Definition**
`std::atomic<T>` is a class template that provides atomic operations on objects of type `T`.

**Technical Definition**
`std::atomic<T>` (since C++11) is a template that wraps a value of type `T` and provides atomic load, store, exchange, compare-exchange, and read-modify-write operations. `T` must be trivially copyable. The type is non-copyable and non-movable. Specialisations exist for integral types, floating-point types, pointer types, and `std::shared_ptr` (C++20).

**Beginner-Friendly Explanation**
`std::atomic<int>` is an integer that can be safely read and written from multiple threads without locks.

### Purposes

- To provide data-race-free access to shared variables.
- To build lock-free data structures.
- To implement reference counting, flags, and counters.
- To provide fine-grained synchronisation.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <atomic>

std::atomic<int> counter{0};
counter.store(42);
int value = counter.load();
counter.fetch_add(1);
```

**Component Breakdown**

| Member | Description |
|--------|-------------|
| `load()` | Atomically reads the value. |
| `store(value)` | Atomically writes the value. |
| `exchange(value)` | Atomically replaces and returns the old value. |
| `compare_exchange_weak/strong` | Conditional atomic update. |
| `fetch_add/sub/and/or/xor` | Atomic read-modify-write operations. |

**Syntax Rules**
- `std::atomic<T>` requires `T` to be trivially copyable.
- Atomics are non-copyable and non-movable.
- Operations default to `memory_order_seq_cst`.

**Constraints and Limitations**
- Not all types are lock-free; check `is_lock_free()`.
- `std::atomic<T>` for large `T` may use an internal lock.

### Annotated Code Example

```cpp
#include <atomic>
#include <thread>
#include <iostream>

std::atomic<int> counter{0};

void increment() {
    for (int i = 0; i < 1000; ++i)
        counter.fetch_add(1, std::memory_order_relaxed);
}

int main() {
    std::thread t1(increment), t2(increment);
    t1.join(); t2.join();
    std::cout << counter.load() << '\n';   // 2000
}
```

**Expected Output**
```
2000
```

**Why this output?**  
`fetch_add` is atomic, so both threads increment `counter` without a data race.

### Real-World Cases

- **Counters and statistics** – thread-safe counters.
- **Flags** – signalling between threads.
- **Reference counting** – internal to `shared_ptr`.

---

## 2. Atomic Types

### Definitions

**Core Definition**
Atomic types are the set of types for which `std::atomic` provides specialisations or that are guaranteed to be lock-free.

**Technical Definition**
`std::atomic` is specialised for integral types, floating-point types (C++20), pointer types, and `std::shared_ptr` (C++20). `std::atomic_flag` is the only type guaranteed to be always lock-free. The `std::atomic<T>` primary template works for any trivially copyable `T`.

**Beginner-Friendly Explanation**
There are atomic versions of common types: `atomic<int>`, `atomic<bool>`, `atomic<T*>`, and so on. `atomic_flag` is the simplest and is always lock-free.

### Common Atomic Types

| Type | Notes |
|------|-------|
| `std::atomic<bool>` | Boolean flag. |
| `std::atomic<int>`, `std::atomic<long>`, etc. | Integral counters and flags. |
| `std::atomic<float>`, `std::atomic<double>` | Floating-point atomics (C++20). |
| `std::atomic<T*>` | Atomic pointers. |
| `std::atomic<std::shared_ptr<T>>` | Atomic shared pointers (C++20). |
| `std::atomic_flag` | Guaranteed lock-free boolean flag. |

### Annotated Code Example

```cpp
#include <atomic>
#include <iostream>
#include <thread>

std::atomic<bool> ready{false};
int data = 0;

void producer() {
    data = 42;
    ready.store(true, std::memory_order_release);
}

void consumer() {
    while (!ready.load(std::memory_order_acquire)) ;
    std::cout << data << '\n';             // 42
}

int main() {
    std::thread t1(producer), t2(consumer);
    t1.join(); t2.join();
}
```

**Expected Output**
```
42
```

**Why this output?**  
The release store and acquire load establish happens-before, making `data = 42` visible to the consumer.

---

## 3. Atomic Operations

### Definitions

**Core Definition**
Atomic operations are the indivisible operations provided by atomic types: load, store, exchange, compare-exchange, and read-modify-write.

**Technical Definition**
The standard atomic operations include `load`, `store`, `exchange`, `compare_exchange_weak`, `compare_exchange_strong`, `fetch_add`, `fetch_sub`, `fetch_and`, `fetch_or`, `fetch_xor`, `operator++`, `operator--`, `operator+=`, `operator-=`, `operator&=`, `operator|=`, and `operator^=` (for appropriate types). Each operation accepts a memory order.

**Beginner-Friendly Explanation**
These are the “safe” operations you can perform on an atomic: read it, write it, swap it, compare-and-swap, and add/subtract.

### Key Operations

| Operation | Description |
|-----------|-------------|
| `load(order)` | Atomically reads the value. |
| `store(value, order)` | Atomically writes the value. |
| `exchange(value, order)` | Atomically replaces and returns the old value. |
| `compare_exchange_weak(expected, desired, order)` | If current == expected, stores desired; otherwise updates expected. May fail spuriously. |
| `compare_exchange_strong` | Same, but does not fail spuriously. |
| `fetch_add(n, order)` | Atomically adds `n` and returns the old value. |

**Constraints and Limitations**
- `compare_exchange_weak` may fail spuriously; use it in a loop.
- `fetch_*` operations are available only for appropriate types (integral, pointer).

### Annotated Code Example

```cpp
#include <atomic>
#include <iostream>

int main() {
    std::atomic<int> x{10};

    std::cout << x.load() << '\n';          // 10
    x.store(20);
    std::cout << x.load() << '\n';          // 20

    int old = x.exchange(30);
    std::cout << old << ' ' << x.load() << '\n'; // 20 30

    int expected = 30;
    bool ok = x.compare_exchange_strong(expected, 40);
    std::cout << ok << ' ' << x.load() << '\n'; // 1 40
}
```

**Expected Output**
```
10
20
20 30
1 40
```

**Why this output?**  
`exchange` replaces the value and returns the old one; `compare_exchange_strong` updates only if the expected value matches.

---

## 4. Memory Ordering

### Definitions

**Core Definition**
Memory ordering specifies the constraints on how atomic operations and surrounding memory accesses may be reordered by the compiler and hardware.

**Technical Definition**
`std::memory_order` is an enumeration with six values: `relaxed`, `consume`, `acquire`, `release`, `acq_rel`, and `seq_cst`. Each atomic operation carries a memory order that determines its synchronisation properties.

**Beginner-Friendly Explanation**
Memory ordering controls how much freedom the compiler and CPU have to reorder memory operations. Stronger orders are safer but slower; weaker orders are faster but require careful reasoning.

### 4.1 `memory_order_relaxed`

**Core Definition**
`relaxed` guarantees atomicity only; no ordering constraints.

**Example**
```cpp
counter.fetch_add(1, std::memory_order_relaxed);
```

**Use** – counters, statistics where ordering does not matter.

---

### 4.2 `memory_order_consume`

**Core Definition**
`consume` provides data-dependency ordering (discouraged).

**Example**
```cpp
auto* p = ptr.load(std::memory_order_consume);
```

**Use** – rarely used; discouraged due to implementation difficulties.

---

### 4.3 `memory_order_acquire`

**Core Definition**
`acquire` prevents later reads/writes from moving before the operation.

**Example**
```cpp
while (!flag.load(std::memory_order_acquire)) ;
```

**Use** – load side of a lock or flag.

---

### 4.4 `memory_order_release`

**Core Definition**
`release` prevents earlier reads/writes from moving after the operation.

**Example**
```cpp
data = 42;
flag.store(true, std::memory_order_release);
```

**Use** – store side of a lock or flag.

---

### 4.5 `memory_order_acq_rel`

**Core Definition**
`acq_rel` combines acquire and release semantics.

**Example**
```cpp
counter.fetch_add(1, std::memory_order_acq_rel);
```

**Use** – read-modify-write operations.

---

### 4.6 `memory_order_seq_cst`

**Core Definition**
`seq_cst` provides a single total order across all sequentially consistent operations.

**Example**
```cpp
flag.store(true);                      // default: seq_cst
```

**Use** – default; easiest to reason about.

---

### Memory Order Summary

| Order | Semantics | Typical Use |
|-------|-----------|-------------|
| `relaxed` | Atomicity only | Counters, statistics |
| `consume` | Data dependency (discouraged) | Rarely used |
| `acquire` | Prevents later ops from moving before | Load side of a lock |
| `release` | Prevents earlier ops from moving after | Store side of a lock |
| `acq_rel` | Both acquire and release | Read-modify-write |
| `seq_cst` | Total order | Default; easiest to reason about |

### Annotated Code Example

```cpp
#include <atomic>
#include <thread>
#include <iostream>

std::atomic<bool> flag{false};
int data = 0;

void producer() {
    data = 42;
    flag.store(true, std::memory_order_release);
}

void consumer() {
    while (!flag.load(std::memory_order_acquire)) ;
    std::cout << data << '\n';             // 42
}

int main() {
    std::thread t1(producer), t2(consumer);
    t1.join(); t2.join();
}
```

**Expected Output**
```
42
```

**Why this output?**  
The release store and acquire load establish happens-before, making `data = 42` visible to the consumer.

---

## 5. `std::atomic_flag`

### Definitions

**Core Definition**
`std::atomic_flag` is the simplest atomic type and the only one guaranteed to be lock-free.

**Technical Definition**
`std::atomic_flag` (since C++11, improved in C++20) is a boolean flag with `test_and_set` and `clear`. Since C++20, it is default-initialized to `false` and has `test()`. It is used to build spinlocks and other low-level primitives.

**Beginner-Friendly Explanation**
`atomic_flag` is the most basic atomic: it's a flag that you can set and test atomically. It is guaranteed to be lock-free, making it the foundation for spinlocks.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::atomic_flag flag = ATOMIC_FLAG_INIT;  // pre-C++20
std::atomic_flag flag;                      // C++20: clear by default
flag.test_and_set();
flag.clear();
bool value = flag.test();                   // C++20
```

**Constraints and Limitations**
- `atomic_flag` is not a general-purpose atomic; it has no load/store.
- Use `test_and_set` and `clear` for spinlocks.

### Annotated Code Example

```cpp
#include <atomic>
#include <thread>
#include <iostream>

std::atomic_flag lock = ATOMIC_FLAG_INIT;

void spinlock_acquire() {
    while (lock.test_and_set(std::memory_order_acquire)) ;
}

void spinlock_release() {
    lock.clear(std::memory_order_release);
}

int main() {
    std::thread t1([]{ 
        spinlock_acquire(); 
        std::cout << "T1\n"; spinlock_release(); 
    });

    std::thread t2([]{ 
        spinlock_acquire(); 
        std::cout << "T2\n"; spinlock_release(); 
    });
    
    t1.join(); t2.join();
}
```

**Expected Output** (order may vary)
```
T1
T2
```

**Why this output?**  
`test_and_set` acquires the flag atomically; the spinlock ensures mutual exclusion.

---

## 6. `std::atomic_ref` (C++20)

### Definitions

**Core Definition**
`std::atomic_ref<T>` provides atomic operations on a non-atomic object of type `T`.

**Technical Definition**
`std::atomic_ref<T>` (C++20) is a class template that provides atomic operations on a referenced object. The referenced object must be aligned and must not be concurrently accessed through non-atomic means. It is useful for applying atomic operations to existing data without changing its type.

**Beginner-Friendly Explanation**
`atomic_ref` lets you treat an ordinary variable atomically for a specific scope, without declaring it as `std::atomic`.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <atomic>

int data = 0;
std::atomic_ref<int> ref(data);
ref.fetch_add(1);
```

**Constraints and Limitations**
- The referenced object must be aligned.
- The object must not be concurrently accessed through non-atomic means.

### Annotated Code Example

```cpp
#include <atomic>
#include <thread>
#include <iostream>

int counter = 0;

void increment() {
    for (int i = 0; i < 1000; ++i) {
        std::atomic_ref<int> ref(counter);
        ref.fetch_add(1, std::memory_order_relaxed);
    }
}

int main() {
    std::thread t1(increment), t2(increment);
    t1.join(); t2.join();
    std::cout << counter << '\n';          // 2000
}
```

**Expected Output**
```
2000
```

**Why this output?**  
`atomic_ref` provides atomic access to the non-atomic `counter`, preventing a data race.

---

## 7. Atomic Best Practices

### Definitions

**Core Definition**
Atomic best practices are guidelines for using atomics correctly, safely, and efficiently.

**Technical Definition**
These practices are informed by the C++ Core Guidelines (CP.100: “Don't use lock-free programming unless you absolutely have to”; CP.101: “Distrust your hardware/compiler combination”; CP.102: “Study the literature”) and community consensus: prefer `seq_cst` until proven otherwise; use `relaxed` only when ordering does not matter; use `atomic_ref` for existing data; prefer higher-level synchronisation for complex invariants; and validate with ThreadSanitizer.

**Beginner-Friendly Explanation**
Best practices for atomics are simple: start with `seq_cst`, use `relaxed` only when you are sure, prefer higher-level tools for complex logic, and test with ThreadSanitizer.

### Key Guidelines

| Guideline | Summary |
|-----------|---------|
| **CP.100** | Don't use lock-free programming unless you absolutely have to. |
| **CP.101** | Distrust your hardware/compiler combination. |
| **CP.102** | Study the literature before writing lock-free code. |
| **Prefer `seq_cst`** | Default; easiest to reason about. |
| **Use `relaxed` carefully** | Only for counters and statistics. |
| **Prefer `std::atomic_flag` for spinlocks** | Guaranteed lock-free. |
| **Use `atomic_ref` for existing data** | Avoid changing layouts. |
| **Validate with tools** | ThreadSanitizer, model checkers. |

### Annotated Code Example

```cpp
#include <atomic>
#include <thread>
#include <iostream>

std::atomic<int> counter{0};

void increment() {
    for (int i = 0; i < 1000; ++i)
        counter.fetch_add(1, std::memory_order_relaxed);
}

int main() {
    std::thread t1(increment), t2(increment);
    t1.join(); t2.join();
    std::cout << counter.load() << '\n';   // 2000
}
```

**Expected Output**
```
2000
```

**Why this output?**  
The atomic counter is safely incremented by both threads; `relaxed` is sufficient because no other memory needs ordering.

---

## References

- `std::atomic` — cppreference – https://en.cppreference.com/w/cpp/atomic/atomic
- `std::memory_order` — cppreference – https://en.cppreference.com/w/cpp/atomic/memory_order
- `std::atomic_flag` — cppreference – https://en.cppreference.com/w/cpp/atomic/atomic_flag
- `std::atomic_ref` — cppreference – https://en.cppreference.com/w/cpp/atomic/atomic_ref
- `std::atomic_thread_fence` — cppreference – https://en.cppreference.com/w/cpp/atomic/atomic_thread_fence
- C++ Core Guidelines — CP.100: Don't use lock-free programming unless you absolutely have to – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-lockfree
- C++ Core Guidelines — CP.101: Distrust your hardware/compiler combination – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-distrust
- C++ Core Guidelines — CP.102: Study the literature – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-literature
- Understand the C++ memory model and happens-before relationships — Dr-Sergey/learn_cpp – https://github.com/Dr-Sergey/learn_cpp/blob/main/10_Concurrency_and_Parallelism/Understand_the_C%2B%2B_memory_model_and_happens-before_relationships.md
- Understand memory fences and `std::atomic_thread_fence` — Dr-Sergey/learn_cpp – https://github.com/Dr-Sergey/learn_cpp/blob/main/10_Concurrency_and_Parallelism/Understand_memory_fences_and_stdatomic_thread_fence.md
- `std::atomic` — Microsoft Learn – https://learn.microsoft.com/en-us/cpp/standard-library/atomic

---

This cheat sheet provides a comprehensive, structured reference for C++ atomics, covering `std::atomic`, atomic types, atomic operations, memory ordering (all six orders), `std::atomic_flag`, `std::atomic_ref`, and best practices, with annotated code examples, expected outputs, real-world cases, and authoritative references.