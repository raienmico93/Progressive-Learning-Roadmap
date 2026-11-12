# C++ Memory Model — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
The C++ memory model is a formal specification that defines how memory operations behave in a multi-threaded program, determining which values a thread can observe and when, and specifying the conditions under which concurrent accesses are safe.

**Technical Definition**
The C++ memory model (formalized in C++11 and refined in subsequent standards) defines the set of allowed executions of a program, including how memory operations from different threads interact. It specifies relations such as **sequenced-before** (within a thread), **synchronizes-with** (between threads via atomic operations), and **happens-before** (the transitive combination of both). A program with a **data race** (conflicting accesses without synchronization) has undefined behaviour. The model provides atomic types and memory ordering specifications to allow programmers to write data-race-free concurrent programs while still permitting compiler and hardware optimizations.

**Beginner-Friendly Explanation**
The memory model is C++'s rulebook for how different threads see each other's memory operations. Without these rules, the compiler and CPU could reorder or optimize memory accesses in ways that make multithreaded programs behave unpredictably. The model guarantees that if you follow the rules (using atomics and proper synchronization), your program will behave correctly on any conforming implementation.

### Key Characteristics

- **Object-centric** – C++ programs manipulate objects; each object has a size, alignment, storage duration, lifetime, and type.
- **Partial order** – the happens-before relation is a partial order, not a total order; some operations are unordered relative to others.
- **Data-race-free by construction** – programs that avoid data races have well-defined behaviour.
- **Compiler and hardware freedom** – the model allows extensive reordering as long as observable behaviour is preserved.
- **Atomic escape hatch** – `std::atomic` provides indivisible operations that participate in the memory model.
- **Fences enable standalone ordering** – `std::atomic_thread_fence` orders all memory operations across it, not just a single atomic variable.

### Prerequisites

- Solid understanding of threads and concurrency.
- Familiarity with `std::atomic` and basic atomic operations.
- Knowledge of data races and the difference between atomic and non-atomic accesses.
- Awareness of compiler optimization and hardware reordering.

### Related Programming Areas

- **Atomics** – the primary tools for participating in the memory model.
- **Mutexes** – higher-level synchronization built on the memory model.
- **Thread synchronization** – barriers, latches, semaphores.
- **Lock-free programming** – relies directly on the memory model.
- **Compiler optimization** – the memory model defines what optimizations are legal.

### Core Concepts / Features

1. C++ memory model (the overarching framework)
2. Memory ordering (`relaxed`, `consume`, `acquire`, `release`, `acq_rel`, `seq_cst`)
3. Happens-before
4. Synchronizes-with
5. Data races
6. Atomic operations
7. Memory fences (`std::atomic_thread_fence`)
8. `std::atomic_signal_fence`
9. Memory model best practices

---

## 1. C++ Memory Model (The Framework)

### Definitions

**Core Definition**
The C++ memory model specifies the set of allowed behaviours for multi-threaded programs, defining how memory operations from different threads interact and when a program has undefined behaviour due to a data race.

**Technical Definition**
The memory model defines the relations **sequenced-before** (program order within a thread), **synchronizes-with** (between threads via atomic operations with appropriate memory orders), and **happens-before** (the transitive closure of sequenced-before and synchronizes-with). A data race occurs when two threads access the same memory location concurrently, at least one access is a write, and neither is atomic. Programs with data races have undefined semantics.

**Beginner-Friendly Explanation**
The memory model is the big-picture rulebook: it says that if two threads touch the same memory without proper synchronization, the program is broken. If you use atomics correctly, the program has well-defined behaviour and you can reason about what each thread will see.

### Purposes

- To define which program executions are valid in the presence of threads.
- To provide a foundation for data-race-free concurrent programming.
- To allow compilers and hardware to optimize while preserving observable behaviour.
- To abstract over different hardware memory models (x86-TSO, ARM weak, etc.).

### Syntax Rules and Structure

**Key Relations**

| Relation | Scope | Description |
|----------|-------|-------------|
| Sequenced-before | Single thread | Program order of evaluations. |
| Synchronizes-with | Between threads | Established by atomic release/acquire pairs. |
| Happens-before | Between threads | Transitive closure of sequenced-before and synchronizes-with. |

**Constraints and Limitations**
- The memory model is a specification, not an implementation.
- Different hardware architectures implement different (weaker or stronger) memory models.

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
    assert(data == 42);                          // guaranteed to see 42
}

int main() {
    std::thread t1(producer), t2(consumer);
    t1.join(); t2.join();
}
```

**Expected Output**  
(No output; assertion passes.)

**Why this output?**  
The release store to `flag` synchronizes-with the acquire load, establishing happens-before; the write to `data` is visible to the consumer.

### Real-World Cases

- **Lock-free data structures** – designing concurrent queues and stacks.
- **Thread synchronisation** – ensuring visibility of shared data.
- **Cross-platform concurrency** – writing code that works on x86 and ARM.

---

## 2. Memory Ordering

### Definitions

**Core Definition**
Memory ordering specifies the constraints on how atomic operations and surrounding memory accesses may be reordered by the compiler and hardware.

**Technical Definition**
`std::memory_order` is an enumeration with six values: `relaxed`, `consume`, `acquire`, `release`, `acq_rel`, and `seq_cst`. Each atomic operation carries a memory order that determines its synchronisation properties. The default for all atomic operations is `seq_cst` (sequentially consistent), which is the strongest and easiest to reason about but may have performance costs.

**Beginner-Friendly Explanation**
Memory ordering controls how much freedom the compiler and CPU have to reorder memory operations. Stronger orders are safer but slower; weaker orders are faster but require careful reasoning.

### Memory Order Summary

| Order | Semantics | Typical Use |
|-------|-----------|-------------|
| `relaxed` | Atomicity only; no ordering constraints | Counters, statistics |
| `consume` | Data-dependency ordering (discouraged) | Rarely used |
| `acquire` | Prevents later reads/writes from moving before | Load side of a lock |
| `release` | Prevents earlier reads/writes from moving after | Store side of a lock |
| `acq_rel` | Both acquire and release | Read-modify-write operations |
| `seq_cst` | Total order across all threads | Default; easiest to reason about |

### Annotated Code Example

```cpp
#include <atomic>
#include <thread>
#include <cassert>

std::atomic<int> x{0}, y{0};
int r1, r2;

void thread1() {
    x.store(1, std::memory_order_relaxed);
    r1 = y.load(std::memory_order_relaxed);
}

void thread2() {
    y.store(1, std::memory_order_relaxed);
    r2 = x.load(std::memory_order_relaxed);
}

int main() {
    std::thread t1(thread1), t2(thread2);
    t1.join(); t2.join();
    // With relaxed ordering, r1 == 0 && r2 == 0 is possible
}
```

**Expected Output**  
(No output; `r1` and `r2` may both be 0.)

**Why this output?**  
Relaxed ordering allows the loads and stores to be reordered; both threads may read 0 before the other's store becomes visible.

### Real-World Cases

- **Reference counting** – `relaxed` for increments, `acq_rel` for decrements.
- **Lock implementations** – `acquire`/`release` for lock/unlock.
- **Publishing data** – `release` store to a flag after writing data.

---

## 3. Happens-Before

### Definitions

**Core Definition**
Happens-before is a relation between operations that guarantees the first operation's effects are visible to the second operation and that the first is sequenced before the second.

**Technical Definition**
The happens-before relation is the transitive closure of **sequenced-before** (within a thread) and **synchronizes-with** (between threads). If operation A happens-before operation B, then A's memory effects are visible to B, and B cannot observe a state where A has not yet occurred. The relation is a partial order, not a total order: some operations are unordered relative to others.

**Beginner-Friendly Explanation**
Happens-before means “this happens first, and its effects are visible.” If A happens-before B, then B sees everything A did. The relation is built from program order within a thread plus synchronization between threads.

### Purposes

- To define visibility guarantees between threads.
- To enable reasoning about data-race-free programs.
- To specify the semantics of synchronization primitives.

### Syntax Rules and Structure

**Key Rules**
- Sequenced-before: within a thread, operations are ordered by program order.
- Synchronizes-with: an atomic release store synchronizes-with an atomic acquire load that reads the stored value.
- Happens-before: transitive closure of the above.

**Constraints and Limitations**
- Happens-before is not a total order; concurrent operations may have no ordering.
- Without synchronization, there is no happens-before between threads.

### Annotated Code Example

```cpp
#include <atomic>
#include <thread>
#include <cassert>

std::atomic<bool> ready{false};
int data = 0;

void producer() {
    data = 42;                              // (1)
    ready.store(true, std::memory_order_release); // (2)
}

void consumer() {
    while (!ready.load(std::memory_order_acquire)) ; // (3)
    assert(data == 42);                     // (4)
}

int main() {
    std::thread t1(producer), t2(consumer);
    t1.join(); t2.join();
}
```

**Expected Output**  
(No output; assertion passes.)

**Why this output?**  
(1) sequenced-before (2); (2) synchronizes-with (3); (3) sequenced-before (4). By transitivity, (1) happens-before (4), so the consumer sees `data == 42`.

### Real-World Cases

- **Publishing objects** – safely handing off objects between threads.
- **Lock implementations** – mutex lock/unlock establishes happens-before.

---

## 4. Synchronizes-With

### Definitions

**Core Definition**
Synchronizes-with is a relation between an atomic release operation and an atomic acquire operation on the same atomic variable that read the value written by the release.

**Technical Definition**
An atomic store with `memory_order_release` synchronizes-with an atomic load with `memory_order_acquire` that reads the value stored (or a value later in the modification order). It can also be established between a release fence and an acquire fence, or between a release fence and an acquire operation, under specific conditions. Synchronizes-with is the mechanism that creates happens-before edges between threads.

**Beginner-Friendly Explanation**
Synchronizes-with is the “handshake” between two threads: one thread releases data, the other acquires it. When the handshake succeeds, the acquiring thread sees everything the releasing thread did before the release.

### Purposes

- To establish happens-before between threads.
- To make writes visible across threads.
- To implement synchronization primitives.

### Syntax Rules and Structure

**Forms of Synchronizes-With**

| Form | Description |
|------|-------------|
| Release store → acquire load | Same atomic variable. |
| Release fence → acquire fence | With relaxed operations between. |
| Release store → acquire fence | If the fence reads the stored value. |

**Constraints and Limitations**
- Only atomic operations can establish synchronizes-with.
- The acquire load must read the value written by the release store (or a later value in the modification order).

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
(No assertion failure.)

**Why this output?**  
The release store synchronizes-with the acquire load, creating a happens-before edge from the write to `data` to the read in the consumer.

---

## 5. Data Races

### Definitions

**Core Definition**
A data race occurs when two threads access the same memory location concurrently, at least one access is a write, and neither access is atomic, and the accesses are not ordered by happens-before.

**Technical Definition**
A data race is defined as two conflicting evaluations (one writes, the other reads or writes the same memory location) that are not sequenced-before and not in the same thread or signal handler, unless both are atomic. A program with a data race has undefined behaviour. This definition allows compilers to make aggressive optimizations on data-race-free code.

**Beginner-Friendly Explanation**
A data race is when two threads touch the same variable at the same time without synchronization, and at least one is writing. This is a bug—the program's behaviour is unpredictable and undefined.

### Purposes

- To define the boundary between well-defined and undefined concurrent programs.
- To allow compiler optimizations that would be invalid in the presence of races.

### Syntax Rules and Structure

**Data Race Conditions**

| Condition | Required |
|-----------|----------|
| Same memory location | Yes |
| Concurrent access | Yes |
| At least one write | Yes |
| Not atomic | Yes |
| No happens-before ordering | Yes |

**Constraints and Limitations**
- Reading a non-atomic variable concurrently with a write is a data race.
- Using `volatile` does **not** prevent data races.

### Annotated Code Example

```cpp
#include <thread>
#include <iostream>

int counter = 0;                          // non-atomic

void increment() {
    for (int i = 0; i < 1000; ++i)
        ++counter;                        // DATA RACE
}

int main() {
    std::thread t1(increment), t2(increment);
    t1.join(); t2.join();
    std::cout << counter << '\n';          // undefined result
}
```

**Expected Output**  
(Undefined; may print any value ≤ 2000.)

**Why this output?**  
Both threads modify `counter` without synchronization; this is a data race, so the program has undefined behaviour.

### Real-World Cases

- **Accidental sharing** – global variables accessed by multiple threads.
- **Missing locks** – forgetting to protect shared data.

---

## 6. Atomic Operations

### Definitions

**Core Definition**
Atomic operations are indivisible operations on atomic types that are guaranteed to be data-race-free and participate in the memory model's happens-before relation.

**Technical Definition**
`std::atomic<T>` provides `load`, `store`, `exchange`, `compare_exchange_weak/strong`, and `fetch_add/sub/and/or/xor`. Each operation accepts a memory order. Atomics are the only types that can safely be accessed concurrently without additional synchronization (though the *values* observed may depend on memory ordering).

**Beginner-Friendly Explanation**
Atomic operations are the “safe” operations for shared data. They happen as a single indivisible step, so no other thread can see them half-done.

### Purposes

- To provide data-race-free access to shared variables.
- To establish synchronizes-with and happens-before relations.
- To build lock-free data structures.

### Syntax Rules and Structure

**Common Operations**

| Operation | Description |
|-----------|-------------|
| `load(order)` | Atomically reads the value. |
| `store(value, order)` | Atomically writes the value. |
| `exchange(value, order)` | Atomically replaces and returns the old value. |
| `compare_exchange_weak/strong` | Conditional atomic update. |
| `fetch_add/sub/and/or/xor` | Atomic read-modify-write. |

**Constraints and Limitations**
- Not all types are lock-free; check `is_lock_free()`.
- Atomics are non-copyable and non-movable.

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
`fetch_add` is atomic, so both threads safely increment the counter without a data race.

---

## 7. Memory Fences (`std::atomic_thread_fence`)

### Definitions

**Core Definition**
`std::atomic_thread_fence` is a standalone fence that orders all memory operations across it, not just a single atomic variable.

**Technical Definition**
`std::atomic_thread_fence(order)` (since C++11, in `<atomic>`) establishes ordering constraints without being tied to a specific atomic object. An **acquire fence** prevents reads/writes after the fence from moving before it; a **release fence** prevents reads/writes before the fence from moving after it. Fences enable synchronization across multiple relaxed atomic operations in one instruction, which can be cheaper than per-variable ordering on some architectures.

**Beginner-Friendly Explanation**
A fence is like a “wall” that memory operations cannot cross. Operations before the wall stay before; operations after stay after. One fence can order many operations at once, unlike per-variable memory ordering.

### Purposes

- To order multiple memory operations without per-variable annotations.
- To batch synchronisation (one fence instead of many ordered atomics).
- To separate data access from synchronisation logic.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <atomic>

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
(No assertion failure.)

**Why this output?**  
The release fence orders both `data1` and `data2` writes before the relaxed flag store; the acquire fence orders the reads after the relaxed flag load.

### Real-World Cases

- **Batching synchronisation** – one fence for multiple relaxed operations.
- **Hardware interop** – mapping to `DMB` (ARM) or `LWSYNC` (POWER).
- **Separating concerns** – data code uses relaxed; synchronisation code adds fences.

---

## 8. `std::atomic_signal_fence`

### Definitions

**Core Definition**
`std::atomic_signal_fence` is a fence that orders memory operations between a thread and a signal handler executed in the same thread, but does not emit hardware fence instructions.

**Technical Definition**
`std::atomic_signal_fence(order)` (since C++11) is equivalent to `std::atomic_thread_fence(order)`, except that the resulting ordering constraints are established only between a thread and a signal handler executed in the same thread. It is a compiler-only fence: it prevents compiler reordering but does not generate hardware memory barriers. It is useful for ordering operations relative to asynchronous signals.

**Beginner-Friendly Explanation**
`atomic_signal_fence` is a “compiler-only” fence: it stops the compiler from reordering memory operations, but it doesn't emit hardware fence instructions. It is used to ensure that a signal handler sees a consistent view of memory.

### Purposes

- To order memory operations relative to signal handlers in the same thread.
- To prevent compiler reordering without hardware barrier overhead.
- To implement signal-safe communication.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::atomic_signal_fence(std::memory_order_seq_cst);
```

**Constraints and Limitations**
- Only affects ordering between a thread and its signal handler.
- Does not generate hardware fences.
- Cannot be used to synchronize between different threads.

### Annotated Code Example

```cpp
#include <atomic>
#include <csignal>
#include <iostream>

volatile std::sig_atomic_t flag = 0;
int data = 0;

void handler(int) {
    std::atomic_signal_fence(std::memory_order_acquire);
    if (flag) std::cout << "Data: " << data << '\n';
}

int main() {
    std::signal(SIGINT, handler);
    data = 42;
    std::atomic_signal_fence(std::memory_order_release);
    flag = 1;
    std::raise(SIGINT);
}
```

**Expected Output**
```
Data: 42
```

**Why this output?**  
The signal fence ensures the write to `data` is ordered before the write to `flag`, and the handler's fence ensures it reads `data` after seeing `flag`.

### Real-World Cases

- **Signal handling** – communicating between a thread and its signal handler.
- **Embedded systems** – interrupt handlers in the same thread context.

---

## 9. Memory Model Best Practices

### Definitions

**Core Definition**
Memory model best practices are guidelines for writing correct, efficient, and maintainable concurrent C++ programs that respect the memory model.

**Technical Definition**
These practices are informed by the C++ Core Guidelines (CP.8: “Don't try to use `volatile` for synchronization”; CP.100: “Don't use lock-free programming unless you absolutely have to”; CP.101: “Distrust your hardware/compiler combination”; CP.102: “Carefully study the literature”) and community consensus: start with `seq_cst` atomics; use `relaxed` only when justified; prefer higher-level synchronization for complex invariants; and validate with ThreadSanitizer or model checkers.

**Beginner-Friendly Explanation**
Best practices for the memory model are simple: start with the safest (`seq_cst`), only relax when you have a proven reason, and always test with race-detection tools.

### Key Guidelines

| Guideline | Summary |
|-----------|---------|
| **CP.8** | Don't use `volatile` for synchronization. |
| **CP.100** | Don't use lock-free programming unless you absolutely have to. |
| **CP.101** | Distrust your hardware/compiler combination. |
| **CP.102** | Carefully study the literature. |
| **Prefer `seq_cst`** | Start with the strongest ordering; relax only with justification. |
| **Use `relaxed` for counters** | When ordering does not matter. |
| **Use fences for batching** | One fence can order multiple relaxed operations. |
| **Validate with tools** | ThreadSanitizer, model checkers. |

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
(No assertion failure.)

**Why this output?**  
`std::atomic` with `relaxed` ordering provides atomicity for the counter; no data race occurs. For a simple counter, `relaxed` is sufficient because no other memory needs ordering.

### Real-World Cases

- **Code reviews** – checking for data races and missing synchronization.
- **Static analysis** – ThreadSanitizer and clang-tidy.
- **Performance tuning** – relaxing memory orders where justified.

---

## References

- C++ Threads (memory model overview) — ISO C++ – http://isocpp.open-std.org/JTC1/SC22/WG14/www/docs/n1196.htm
- `std::memory_order` — cppreference – https://en.cppreference.com/w/cpp/atomic/memory_order
- `std::atomic_thread_fence` — cppreference – https://en.cppreference.com/w/cpp/atomic/atomic_thread_fence
- `std::atomic_signal_fence` — cppreference – https://en.cppreference.com/w/cpp/atomic/atomic_signal_fence
- N1276: A Less Formal Explanation of the Proposed C++ Concurrency Memory Model — Open Standards – https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1276.htm
- C++ Core Guidelines — CP.8: Don't try to use `volatile` for synchronization – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-volatile
- C++ Core Guidelines — CP.100: Don't use lock-free programming unless you absolutely have to – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-lockfree
- C++ Core Guidelines — CP.101: Distrust your hardware/compiler combination – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-distrust
- C++ Core Guidelines — CP.102: Carefully study the literature – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-literature
- Understand memory fences and `std::atomic_thread_fence` — Dr-Sergey/learn_cpp – https://github.com/Dr-Sergey/learn_cpp/blob/main/10_Concurrency_and_Parallelism/Understand_memory_fences_and_stdatomic_thread_fence.md
- Atomic fences (Oracle man page) – https://docs.oracle.com/cd/E60778_01/html/E61993/atomic-fence-3a.html
- Issue 1480: Atomic fences don't have synchronizes with relation — WG21 – https://wg21.cmeerw.net/lwg/issue1480

---

This cheat sheet provides a comprehensive, structured reference for the C++ memory model, covering the framework, memory ordering, happens-before, synchronizes-with, data races, atomic operations, memory fences (`std::atomic_thread_fence`), `std::atomic_signal_fence`, and best practices, with annotated code examples, expected outputs, real-world cases, and authoritative references.