# C++ Lock-Free Programming — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Lock-free programming is a concurrency paradigm in which multiple threads operate on shared data without using mutual-exclusion locks, relying instead on atomic hardware instructions to guarantee that **at least one** thread always makes progress.

**Technical Definition**
A lock-free data structure guarantees that **some** thread operating on it completes its operation in a bounded number of steps, even if other threads are stalled, preempted, or contending for the same resources. This is a **system-wide** progress guarantee: it does not guarantee that *every* thread makes progress (individual threads can starve), but it does guarantee that the system as a whole never deadlocks. Lock-free algorithms are built on atomic read-modify-write (RMW) primitives—most commonly compare-and-swap (CAS)—and loop until their atomic update succeeds. The stronger guarantee, **wait-free**, guarantees that **every** thread completes in a bounded number of steps.

**Beginner-Friendly Explanation**
Lock-free programming is “concurrency without locks.” Instead of a thread grabbing a mutex and blocking everyone else, threads optimistically attempt their operation and retry if someone else got there first. The key promise is: even if one thread is slow, the system as a whole keeps moving. The cost is complexity—and a class of subtle bugs (ABA, memory reclamation) that do not exist with mutexes.

### Key Characteristics

- **Non-blocking progress** – no thread can be blocked indefinitely by another thread’s failure or suspension.
- **CAS-centric** – compare-and-swap (or compare-exchange) is the fundamental building block.
- **Retry loops** – operations read state, compute an update, and CAS; if the CAS fails, they retry.
- **No deadlock** – by construction, no thread can hold a “lock” that another needs.
- **Starvation possible** – a single thread can theoretically be starved indefinitely under contention.
- **Memory reclamation is the hard part** – safely freeing removed nodes while other threads may still hold raw pointers is the central challenge.
- **Progress guarantees form a hierarchy** – obstruction-free < lock-free < wait-free.

### Prerequisites

- Solid understanding of `std::atomic` and memory ordering (`acquire`/`release`/`relaxed`/`seq_cst`).
- Knowledge of the C++ memory model (happens-before, synchronizes-with).
- Familiarity with CAS (`std::atomic::compare_exchange_weak/strong`).
- Understanding of raw pointers and node-based data structures.
- Awareness of the ABA problem and memory reclamation hazards.

### Related Programming Areas

- **Atomics** – the primitives on which lock-free code is built.
- **Memory model** – defines the ordering guarantees that lock-free algorithms rely on.
- **Mutexes** – the blocking alternative; lock-free is chosen when mutexes are too slow or unacceptable.
- **Hazard pointers / RCU** – safe memory reclamation mechanisms (C++26).
- **Concurrent data structures** – queues, stacks, hash maps, ring buffers.

### Core Concepts / Features

1. Lock-free programming
2. Wait-free programming
3. Lock-free data structures
4. Compare-and-swap (CAS)
5. ABA problem
6. Memory reclamation
7. Hazard pointers
8. RCU (Read-Copy-Update)
9. Lock-free best practices

---

## 1. Lock-Free Programming

### Definitions

**Core Definition**
A lock-free algorithm guarantees that **at least one** thread makes progress in a bounded number of steps, regardless of the behaviour of other threads.

**Technical Definition**
A data structure is lock-free if **at least one thread** operating on it completes its operation in a bounded number of steps, even if other threads are stalled, preempted, or contending for the same resources. This is a **system-wide** progress guarantee: it does not guarantee that every thread makes progress, but it guarantees that the system never deadlocks. The canonical lock-free pattern is the **CAS loop**: read the current state, compute an updated state, and attempt to install it via `compare_exchange`; if the exchange fails because another thread modified the state, retry.

**Beginner-Friendly Explanation**
Lock-free means “the system never gets stuck.” Even if one thread is slow, another thread will get its work done. It does not mean “every thread finishes quickly”—a single unlucky thread can theoretically be starved forever.

### Purposes

- To eliminate deadlock and priority inversion.
- To provide system-wide progress even when a thread is preempted.
- To reduce latency in high-contention scenarios.
- To build data structures that remain responsive under unpredictable scheduling.
- To avoid the overhead and blocking behaviour of mutexes.

### Syntax Rules and Structure

**The CAS Loop Pattern**
```cpp
void lock_free_update(std::atomic<T*>& head, T* new_node) {
    T* old_head = head.load(std::memory_order_acquire);
    do {
        new_node->next = old_head;
    } while (!head.compare_exchange_weak(
        old_head, new_node,
        std::memory_order_release,
        std::memory_order_acquire));
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `load` | Reads the current state (usually `acquire`). |
| `compare_exchange_weak` | Attempts to install the update; fails if the state changed. |
| Loop | Retries until the CAS succeeds. |
| Memory orders | `release` on success, `acquire` on failure. |

**Constraints and Limitations**
- Lock-freedom does **not** guarantee starvation-freedom.
- CAS loops suffer under high contention: repeated failures waste CPU.
- Memory reclamation is not solved by lock-freedom alone.

### Annotated Code Examples

**Example 1: Lock-free stack push (Treiber stack)**

```cpp
#include <atomic>
#include <iostream>

struct Node {
    int value;
    Node* next;
};

std::atomic<Node*> head{nullptr};

void push(int value) {
    Node* new_node = new Node{value, nullptr};
    Node* old_head = head.load(std::memory_order_relaxed);
    do {
        new_node->next = old_head;
    } while (!head.compare_exchange_weak(
        old_head, new_node,
        std::memory_order_release,
        std::memory_order_relaxed));
}

int main() {
    push(42);
    Node* h = head.load(std::memory_order_acquire);
    std::cout << h->value << '\n';         // 42
}
```

**Expected Output**
```
42
```

**Why this output?**  
`push` uses a CAS loop to atomically install the new node at the head of the stack. If another thread modifies `head` between the load and the CAS, the loop retries.

**Example 2: Lock-free counter**

```cpp
#include <atomic>
#include <thread>
#include <iostream>

std::atomic<int> counter{0};

void increment() {
    for (int i = 0; i < 1000; ++i) {
        int old = counter.load(std::memory_order_relaxed);
        while (!counter.compare_exchange_weak(
            old, old + 1,
            std::memory_order_relaxed,
            std::memory_order_relaxed));
    }
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
Each thread uses CAS to increment the counter. `compare_exchange_weak` may fail spuriously, but the loop retries until success.

### Real-World Cases

- **High-frequency trading** – low-latency queues where mutex blocking is unacceptable.
- **Kernel data structures** – routing tables and configuration updates (RCU).
- **Lock-free memory allocators** – per-thread pools with lock-free free lists.

---

## 2. Wait-Free Programming

### Definitions

**Core Definition**
A wait-free algorithm guarantees that **every** thread completes its operation in a bounded number of steps, regardless of other threads.

**Technical Definition**
Wait-freedom is the strongest progress guarantee: every operation is guaranteed to complete in a bounded number of steps, **even if other threads are stalled, preempted, or contending**. Wait-free implies lock-free, but not vice versa. The canonical technique for transforming a lock-free algorithm into a wait-free one is **helping**: when a thread encounters another thread’s in-progress operation, it helps complete that operation before proceeding with its own.

**Beginner-Friendly Explanation**
Wait-free means “everyone finishes, and quickly.” Unlike lock-free, where an individual thread can be starved, wait-free guarantees that every thread completes in a bounded number of steps. It is the gold standard of non-blocking progress, but it is significantly harder to design and implement.

### Purposes

- To guarantee bounded latency for every operation (low tail latency).
- To eliminate starvation entirely.
- To provide deterministic progress guarantees in real-time systems.
- To overcome the contention problems of CAS loops.

### Syntax Rules and Structure

**Helping Pattern (Conceptual)**
```cpp
// Conceptual: an operation that encounters an in-progress
// operation helps complete it before retrying its own.
while (!try_complete_own_operation()) {
    if (auto* in_progress = find_in_progress_operation()) {
        help_complete(in_progress);
    }
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Descriptor | A record of an in-progress operation. |
| Helping | Other threads complete the operation described by the descriptor. |
| Bounded steps | Each thread completes in a bounded number of steps. |

**Constraints and Limitations**
- Significantly more complex than lock-free.
- Requires per-operation descriptors and careful memory management.
- Often has higher constant overhead than lock-free in the uncontended case.

### Annotated Code Example (Conceptual)

```cpp
// Conceptual wait-free counter using helping (simplified).
// Real implementations use descriptors and CAS on a state word.
#include <atomic>

struct Descriptor {
    int index;
    int increment;
};

std::atomic<int> counter{0};
// A full implementation would maintain a list of active descriptors
// and allow any thread to help complete pending increments.
```

**Expected Output**  
(Conceptual; no executable output.)

**Why this output?**  
A wait-free counter would allow any thread to help complete another thread’s pending increment, ensuring bounded progress for every thread.

### Real-World Cases

- **Real-time systems** – bounded latency for every operation.
- **Wait-free queues** – MPMC ring buffers with helping.
- **Wait-free memory reclamation** – hazard pointers with helping (C++26).

---

## 3. Lock-Free Data Structures

### Definitions

**Core Definition**
A lock-free data structure is a concurrent data structure that uses atomic operations (typically CAS) instead of locks to coordinate access, guaranteeing system-wide progress.

**Technical Definition**
Common lock-free data structures include the **Treiber stack** (lock-free push/pop with CAS on the head pointer), the **Michael-Scott queue** (lock-free MPMC queue using CAS on head and tail), the **SPSC ring buffer** (single-producer single-consumer queue with acquire/release indices), and lock-free linked lists and hash maps. Each requires careful attention to ABA and memory reclamation.

**Beginner-Friendly Explanation**
Lock-free data structures are the “no-lock” versions of queues, stacks, and lists. They use CAS loops instead of mutexes and must handle the tricky problems of safe memory reclamation.

### Purposes

- To provide concurrent queues, stacks, and maps without blocking.
- To achieve low latency under contention.
- To avoid deadlock and priority inversion.
- To scale across many cores.

### Syntax Rules and Structure

**Lock-Free Stack (Treiber)**
```cpp
struct Node { int value; Node* next; };
std::atomic<Node*> head{nullptr};

void push(int v) {
    Node* n = new Node{v, nullptr};
    Node* old = head.load(std::memory_order_relaxed);
    do { n->next = old; }
    while (!head.compare_exchange_weak(old, n,
        std::memory_order_release, std::memory_order_relaxed));
}

bool pop(int& out) {
    Node* old = head.load(std::memory_order_acquire);
    while (old && !head.compare_exchange_weak(old, old->next,
        std::memory_order_acquire, std::memory_order_acquire));
    if (!old) return false;
    out = old->value;
    // UNSAFE: old may still be referenced by another thread
    delete old;
    return true;
}
```

**Component Breakdown**

| Structure | Core Technique |
|-----------|----------------|
| Treiber stack | CAS on `head` pointer. |
| Michael-Scott queue | CAS on `head` and `tail` with a dummy node. |
| SPSC ring buffer | Acquire/release on `head` and `tail` indices. |

**Constraints and Limitations**
- The `pop` above is **not** memory-reclamation safe; see §6.
- ABA can corrupt the stack if nodes are reused; see §5.
- Contention can cause repeated CAS failures.

### Annotated Code Example

```cpp
#include <atomic>
#include <iostream>

struct Node { int value; Node* next; };
std::atomic<Node*> head{nullptr};

void push(int v) {
    Node* n = new Node{v, nullptr};
    Node* old = head.load(std::memory_order_relaxed);
    do { n->next = old; }
    while (!head.compare_exchange_weak(old, n,
        std::memory_order_release, std::memory_order_relaxed));
}

int main() {
    push(1);
    push(2);
    Node* h = head.load(std::memory_order_acquire);
    std::cout << h->value << '\n';         // 2 (LIFO)
}
```

**Expected Output**
```
2
```

**Why this output?**  
The stack is LIFO: the last pushed value (`2`) is at the head.

### Real-World Cases

- **SPSC ring buffers** – high-frequency trading event streams.
- **MPMC queues** – task queues in thread pools.
- **Lock-free stacks** – free-list allocators.

---

## 4. Compare-and-Swap (CAS)

### Definitions

**Core Definition**
Compare-and-swap (CAS) is an atomic instruction that compares a memory location’s current value to an expected value and, if they match, replaces it with a new value—all in one indivisible step.

**Technical Definition**
In C++, CAS is exposed as `std::atomic<T>::compare_exchange_weak` and `compare_exchange_strong`. Both take an `expected` reference and a `desired` value: if the current value equals `expected`, it is replaced with `desired` and the function returns `true`; otherwise, `expected` is updated with the current value and the function returns `false`. `weak` may fail spuriously (even when the values match); `strong` does not. CAS is the fundamental building block of almost all lock-free algorithms.

**Beginner-Friendly Explanation**
CAS is an atomic “check and swap”: “If the value is still what I expect, change it to this new value. Otherwise, tell me what the current value is.” It is the primitive that makes lock-free updates possible.

### Purposes

- To atomically update a shared variable only if it has not changed.
- To implement lock-free push/pop, enqueue/dequeue, and counters.
- To build the CAS loop pattern.

### Syntax Rules and Structure

**General Syntax**
```cpp
bool compare_exchange_weak(T& expected, T desired,
                           memory_order success,
                           memory_order failure);
bool compare_exchange_strong(T& expected, T desired,
                             memory_order success,
                             memory_order failure);
```

**Component Breakdown**

| Parameter | Description |
|-----------|-------------|
| `expected` | In/out: the value you expect; updated on failure. |
| `desired` | The new value to install. |
| `success` | Memory order on success. |
| `failure` | Memory order on failure (must not be `release` or `acq_rel`). |

**Constraints and Limitations**
- `weak` may fail spuriously; use it in a loop.
- `strong` is for one-shot CAS where a spurious failure would be a bug.
- CAS on pointers can suffer from ABA.

### Annotated Code Example

```cpp
#include <atomic>
#include <iostream>

std::atomic<int> value{10};

int main() {
    int expected = 10;
    bool ok = value.compare_exchange_strong(expected, 20);
    std::cout << std::boolalpha << ok << ' ' << value.load() << '\n'; // true 20

    expected = 10;                          // stale
    ok = value.compare_exchange_strong(expected, 30);
    std::cout << ok << ' ' << expected << ' ' << value.load() << '\n'; // false 20 20
}
```

**Expected Output**
```
true 20
false 20 20
```

**Why this output?**  
The first CAS succeeds because `value` is `10`; the second fails because `value` is now `20`, and `expected` is updated to `20`.

### Real-World Cases

- **Lock-free counters** – `fetch_add` is often implemented with CAS.
- **Reference counting** – `shared_ptr` control block updates.
- **Lock-free stacks and queues** – head/tail pointer updates.

---

## 5. ABA Problem

### Definitions

**Core Definition**
The ABA problem occurs when a memory location changes from value A to B and back to A between a thread’s read and its CAS, causing the CAS to succeed even though the state has changed in a way that makes the operation incorrect.

**Technical Definition**
In a CAS loop, a thread reads a value A, computes an update, and then performs CAS expecting A. If, in the meantime, another thread changes the value to B and then back to A, the CAS succeeds—but the thread’s assumption that the state has not changed is false. In node-based data structures, this typically means a node was popped, freed, and a **new node was allocated at the same address**; the CAS succeeds but the node’s `next` pointer now points to freed or unrelated memory.

**Beginner-Friendly Explanation**
ABA is a sneaky bug: you check that the value is still A, but it went A → B → A while you weren’t looking. You think nothing changed, but everything did. In a stack, this can make you link a node to memory that has already been freed.

### Purposes (Problem Awareness)

- To understand why naive CAS loops can be incorrect.
- To motivate version counters, hazard pointers, and RCU.
- To design safe lock-free data structures.

### Syntax Rules and Structure

**ABA Scenario**
```
Thread A: reads head = X
Thread A: preempted
Thread B: pops X, pops Y, pushes new node at same address X
Thread A: resumes, CAS(head, X, Y) → succeeds
          but Y was already freed → use-after-free
```

**Common Solutions**

| Solution | Mechanism |
|----------|-----------|
| Tagged pointers | Pack a version counter with the pointer in a double-width CAS. |
| Hazard pointers | Prevent reclamation, so addresses are not reused while readers hold them. |
| RCU | Defer reclamation until a grace period; addresses are not reused during the period. |

**Constraints and Limitations**
- Tagged pointers require double-width CAS (`cmpxchg16b` on x86-64).
- Hazard pointers and RCU solve ABA as a side effect of safe reclamation.

### Annotated Code Example (Tagged Pointer Concept)

```cpp
// Conceptual: pack pointer and version into a single 128-bit value.
struct TaggedPtr {
    Node* ptr;
    uint64_t tag;
};
std::atomic<TaggedPtr> head;               // requires double-width CAS

void push(Node* n) {
    TaggedPtr old = head.load();
    TaggedPtr desired;
    do {
        n->next = old.ptr;
        desired = {n, old.tag + 1};        // increment tag on every update
    } while (!head.compare_exchange_weak(old, desired));
}
```

**Expected Output**  
(Conceptual; no executable output.)

**Why this output?**  
Every update increments the tag, so an A→B→A sequence produces a different tag and the CAS fails, preventing ABA.

### Real-World Cases

- **Lock-free stacks** – ABA corrupts the head pointer.
- **Lock-free free lists** – nodes are reused, making ABA likely.
- **Tagged pointers in Boost.Lockfree** – uses the high bits of 64-bit pointers for a tag.

---

## 6. Memory Reclamation

### Definitions

**Core Definition**
Memory reclamation is the problem of safely freeing memory that may still be referenced by other threads in a lock-free data structure.

**Technical Definition**
In lock-free data structures, when a node is removed from the structure, other threads may still hold raw pointers to it (having read the pointer before the removal). Freeing the node immediately causes a **use-after-free**. Safe reclamation requires a mechanism to defer deletion until no thread can still access the node. The main approaches are **hazard pointers** (per-thread announcements of protected pointers) and **RCU** (grace-period-based deferral).

**Beginner-Friendly Explanation**
When you remove a node from a lock-free structure, you cannot just `delete` it—someone else might still be reading it. Memory reclamation is the set of techniques for figuring out when it is finally safe to free the node.

### Purposes

- To prevent use-after-free in lock-free data structures.
- To prevent ABA by avoiding premature address reuse.
- To bound memory usage (or accept unbounded usage with RCU).
- To provide deterministic or batched reclamation.

### Syntax Rules and Structure

**Reclamation Approaches**

| Approach | Mechanism | Bounded Memory? |
|----------|-----------|-----------------|
| Hazard pointers | Readers publish pointers; writers scan before freeing. | Yes (O(threads × pointers)) |
| RCU | Writers wait for a grace period; readers are lock-free. | No (unbounded during grace period) |
| Epoch-based | Defer until all threads advance past an epoch. | No (unbounded) |
| Reference counting | Atomic count; free when zero. | Yes (but cycles leak) |

**Constraints and Limitations**
- Immediate `delete` after removal is **never** safe without one of these mechanisms.
- Boost.Lockfree avoids the problem by never returning memory to the OS (free-list reuse).

### Annotated Code Example (Unsafe vs. Safe Concept)

```cpp
// UNSAFE: immediate delete after pop
bool pop_unsafe(int& out) {
    Node* old = head.load();
    while (old && !head.compare_exchange_weak(old, old->next));
    if (!old) return false;
    out = old->value;
    delete old;                            // UNSAFE: another thread may still read it
    return true;
}

// SAFE: defer reclamation (conceptual; see hazard pointers §7)
void pop_safe(int& out) {
    Node* old = head.load();
    while (old && !head.compare_exchange_weak(old, old->next));
    if (!old) return false;
    out = old->value;
    retired.push(old);                     // defer; free only when safe
    return true;
}
```

**Expected Output**  
(Conceptual; illustrates the hazard.)

**Why this output?**  
Immediate deletion causes use-after-free; deferred reclamation (via hazard pointers or RCU) prevents it.

### Real-World Cases

- **Lock-free queues** – nodes are removed and must be reclaimed safely.
- **Lock-free hash maps** – buckets and nodes require reclamation.
- **Kernel RCU** – routing tables and configuration data.

---

## 7. Hazard Pointers

### Definitions

**Core Definition**
A hazard pointer is a per-thread slot in which a reader publishes the address it is currently accessing, allowing writers to defer reclamation of that address until no reader is using it.

**Technical Definition**
Hazard pointers (proposed for C++26, `<hazard_pointer>`) solve the reclamation problem by letting each reader **announce** which pointers it is currently accessing. A writer that wants to reclaim memory checks all published hazard pointers and defers deletion until no reader holds the address. The protocol has three steps: **protect** (publish the pointer), **verify** (re-read the source to confirm it has not changed), and **access** (safely dereference). Each `hazard_pointer` object protects **one** address at a time. Because a protected address cannot be freed, it also cannot be reallocated—**hazard pointers prevent ABA as a side effect**.

**Beginner-Friendly Explanation**
A hazard pointer is a “do not delete” sign. Before reading a node, a thread posts the node’s address in its hazard pointer slot. Writers check all slots and only delete nodes that are not posted. This prevents use-after-free and ABA.

### Purposes

- To safely reclaim memory in lock-free data structures.
- To prevent ABA by preventing address reuse.
- To provide bounded memory reclamation (O(threads × pointers)).
- To offer deterministic reclamation (unlike epoch-based approaches).

### Syntax Rules and Structure

**Protocol (3 Steps)**
```cpp
// Reader:
auto hp = std::make_hazard_pointer();
Node* safe = hp.protect(head);             // publish pointer
if (safe) { use(safe); }                  // safe to dereference
hp.reset_protection();                     // release

// Writer:
Node* old = head.exchange(new_node);
old->retire();                             // defer; scan hazard list before freeing
```

**Component Breakdown**

| API (C++26) | Purpose |
|-------------|---------|
| `std::make_hazard_pointer()` | Acquire a per-thread hazard slot. |
| `hp.protect(src)` | Atomically load and publish a pointer. |
| `hp.reset_protection()` | Clear the hazard slot. |
| `obj->retire()` | Schedule deferred reclamation. |
| `std::hazard_pointer_obj_base<T>` | CRTP base that adds `retire()`. |

**Constraints and Limitations**
- Each hazard pointer protects **one** address.
- Requires C++26 (or a third-party implementation).
- Writers must scan all hazard slots before freeing.

### Annotated Code Example (Manual Simulation)

```cpp
#include <atomic>
#include <iostream>

template<typename T>
struct HazardSlot {
    std::atomic<T*> protected_ptr{nullptr};

    T* protect(const std::atomic<T*>& source) {
        T* ptr = nullptr;
        do {
            ptr = source.load(std::memory_order_acquire);
            protected_ptr.store(ptr, std::memory_order_release);
        } while (ptr != source.load(std::memory_order_acquire)); // verify
        return ptr;
    }
    void reset() { protected_ptr.store(nullptr, std::memory_order_release); }
};

struct Node { int value; Node* next; };

int main() {
    std::atomic<Node*> head{new Node{42, nullptr}};
    HazardSlot<Node> hp;

    Node* safe = hp.protect(head);          // protect
    if (safe) std::cout << safe->value << '\n'; // 42
    hp.reset();                             // release
}
```

**Expected Output**
```
42
```

**Why this output?**  
The hazard slot publishes the pointer before dereferencing, ensuring the node cannot be freed while it is being read.

### Real-World Cases

- **Lock-free queues and stacks** – safe node reclamation.
- **Lock-free hash maps** – safe bucket/node reclamation.
- **C++26 standard library** – `std::hazard_pointer` is being standardized.

---

## 8. RCU (Read-Copy-Update)

### Definitions

**Core Definition**
RCU (Read-Copy-Update) is a synchronization mechanism optimized for **read-mostly** data: readers access data without locks or atomic operations, while writers make a copy, publish it, and defer reclamation of the old version until a **grace period** has elapsed.

**Technical Definition**
RCU provides three guarantees: a **grace-period guarantee** (updaters can wait for pre-existing readers), a **publish-subscribe guarantee** (new data is published safely), and **memory-ordering guarantees**. Readers are delimited by `rcu_read_lock()` and `rcu_read_unlock()` (which may be nested). Writers use `synchronize_rcu()` to block until all pre-existing readers have completed, or `call_rcu()` to schedule an asynchronous callback after a grace period. Once the grace period completes, the old data can be safely freed. RCU is ideal for read-often, update-seldom workloads.

**Beginner-Friendly Explanation**
RCU is “copy, swap, wait, free.” Readers never block or lock. A writer copies the data structure, modifies the copy, atomically swaps it in, and then waits for all readers that started before the swap to finish before freeing the old version. It is extremely fast for readers but can use unbounded memory during the grace period.

### Purposes

- To provide extremely low-overhead reads in read-mostly workloads.
- To avoid reader-side locks and atomic operations.
- To allow concurrent readers and writers without blocking.
- To defer reclamation until it is safe.

### Syntax Rules and Structure

**General Syntax (Conceptual)**
```cpp
// Reader:
rcu_read_lock();
auto* p = rcu_dereference(shared_ptr);
use(*p);
rcu_read_unlock();

// Writer:
auto* new_version = copy_and_modify(old);
rcu_assign_pointer(shared_ptr, new_version);
synchronize_rcu();                         // wait for pre-existing readers
free(old);
```

**Component Breakdown**

| Operation | Description |
|-----------|-------------|
| `rcu_read_lock` / `rcu_read_unlock` | Delimit a read-side critical section. |
| `rcu_dereference` | Safely read an RCU-protected pointer. |
| `rcu_assign_pointer` | Safely publish a new pointer. |
| `synchronize_rcu` | Block until a grace period completes. |
| `call_rcu` | Asynchronously invoke a callback after a grace period. |

**Constraints and Limitations**
- Readers must not block or sleep inside the read-side critical section.
- Memory usage can grow unbounded during a long grace period.
- Not suitable for write-heavy workloads.

### Annotated Code Example (Conceptual)

```cpp
// Conceptual RCU-style update (not standard C++; illustrates the pattern).
#include <atomic>
#include <memory>

struct Config { int value; };

std::atomic<Config*> current{new Config{42}};

void reader() {
    Config* p = current.load(std::memory_order_acquire);
    // use p->value safely (p will not be freed during this read)
}

void writer() {
    Config* old = current.load(std::memory_order_relaxed);
    Config* fresh = new Config{old->value + 1};
    current.store(fresh, std::memory_order_release);
    // wait for pre-existing readers, then delete old
    delete old;
}
```

**Expected Output**  
(Conceptual; no executable output.)

**Why this output?**  
The reader loads the current pointer atomically; the writer publishes a new version and defers deletion of the old version until a grace period (simplified here).

### Real-World Cases

- **Linux kernel** – routing tables, configuration data, and security policies.
- **Userspace RCU (liburcu)** – high-performance read-mostly data structures.
- **C++26 `std::rcu`** – standardized RCU facility (in progress).

---

## 9. Lock-Free Best Practices

### Definitions

**Core Definition**
Lock-free best practices are guidelines for determining when lock-free programming is appropriate, how to implement it correctly, and how to validate it.

**Technical Definition**
The C++ Core Guidelines codify the most important rule: **CP.100: “Don't use lock-free programming unless you absolutely have to.”** Lock-free code is notoriously difficult to get right, and the performance benefits are often smaller than expected. When lock-free is justified, the guidelines recommend: **CP.101: “Distrust your hardware/compiler combination”** (verify assumptions), and **CP.102: “Carefully study the literature”** (understand the algorithms you are implementing). Additional practices include using `seq_cst` initially, validating with ThreadSanitizer, and never underestimating memory reclamation.

**Beginner-Friendly Explanation**
The best practice for lock-free programming is: don't do it unless you really have to. If you do, study the algorithms, test exhaustively, and remember that memory reclamation is the hard part.

### Key Guidelines

| Guideline | Summary |
|-----------|---------|
| **CP.100** | Don't use lock-free programming unless you absolutely have to. |
| **CP.101** | Distrust your hardware/compiler combination. |
| **CP.102** | Carefully study the literature before implementing lock-free algorithms. |
| **CP.9** | Whenever feasible, use tools to validate your concurrent code. |
| **Start with `seq_cst`** | Use the strongest memory order; relax only with justification. |
| **Use `weak` CAS in loops** | `weak` is faster and may fail spuriously; loop until success. |
| **Never immediately `delete` after removal** | Use hazard pointers or RCU. |
| **Prefer existing libraries** | Boost.Lockfree, Folly, and C++26 standard facilities. |
| **Measure before committing** | The CAS-loop retry cost can outweigh mutex overhead. |
| **Consider wait-free only when necessary** | Wait-free is even harder; use it for bounded latency requirements. |

### Annotated Code Example (Safe vs. Unsafe)

```cpp
#include <atomic>
#include <iostream>

std::atomic<int> counter{0};

void safe_increment() {
    int old = counter.load(std::memory_order_relaxed);
    while (!counter.compare_exchange_weak(
        old, old + 1,
        std::memory_order_relaxed,
        std::memory_order_relaxed));
}

int main() {
    safe_increment();
    std::cout << counter.load() << '\n';   // 1
}
```

**Expected Output**
```
1
```

**Why this output?**  
The CAS loop is the canonical safe lock-free increment; it retries until the CAS succeeds.

### Real-World Cases

- **Code reviews** – flagging unnecessary lock-free code.
- **Static analysis** – ThreadSanitizer and model checkers.
- **Library selection** – preferring proven implementations over custom code.

---

## References

- C++ Core Guidelines — CP.100: Don't use lock-free programming unless you absolutely have to — https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-lockfree
- C++ Core Guidelines — CP.101: Distrust your hardware/compiler combination — https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-distrust
- C++ Core Guidelines — CP.102: Carefully study the literature — https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-literature
- CppCon 2024: Introduction to Wait-free Algorithms in C++ Programming (Daniel Anderson) — https://isocpp.org/blog/2025/06/cppcon-2024-introduction-to-wait-free-algorithms-in-cpp-programming-daniel
- `std::atomic::compare_exchange` — cppreference — https://en.cppreference.com/w/cpp/atomic/atomic/compare_exchange
- Hazard Pointers (P0233R1) — https://rap.no/JTC1/SC22/WG21/docs/papers/2016/p0233r1.pdf
- Read-Copy Update (RCU) for C++ (P0279R1) — https://rap.no/JTC1/SC22/WG21/docs/papers/2016/p0279r1.pdf
- Understand std::hazard_pointer for lock-free memory reclamation — Dr-Sergey/learn_cpp — https://github.com/Dr-Sergey/learn_cpp/blob/main/26_Standard_Library_New_Cpp23_26/Understand_stdhazard_pointer_for_lock-free_memory_reclamation.md
- Understand std::hazard_pointer and std::rcu (C++26 preview) — Dr-Sergey/learn_cpp — https://github.com/Dr-Sergey/learn_cpp/blob/main/10_Concurrency_and_Parallelism/Understand_stdhazard_pointer_and_stdrcu_C++26_preview_for_lock-free_reclamation.md
- Boost.Lockfree rationale (ABA and memory management) — https://lcgpackages.web.cern.ch/tarFiles/sources/boost_1_59_0/doc/html/lockfree/rationale.html
- Lock-free programming lecture (ETH Zürich) — https://lec.inf.ethz.ch/DA/2022/slides/daLecture26.en.pdf

---

This cheat sheet provides a comprehensive, structured reference for C++ lock-free programming, covering lock-free and wait-free progress guarantees, lock-free data structures, compare-and-swap, the ABA problem, memory reclamation, hazard pointers, RCU, and best practices, with annotated code examples, expected outputs, real-world cases, and authoritative references.