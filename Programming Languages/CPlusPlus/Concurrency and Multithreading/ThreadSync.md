# C++ Thread Synchronization — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Thread synchronization is the coordination of concurrent threads to ensure correct ordering of operations and safe access to shared resources, using primitives such as mutexes, condition variables, barriers, latches, and semaphores.

**Technical Definition**
C++ provides synchronization primitives in `<mutex>`, `<condition_variable>`, `<barrier>`, `<latch>`, and `<semaphore>`. Mutexes and condition variables (C++11) provide mutual exclusion and waiting/notification. C++20 adds **latches** (single-use countdown barriers), **barriers** (reusable phase synchronization), and **semaphores** (counting resource guards). These primitives are built on the memory model and establish happens-before relationships between threads.

**Beginner-Friendly Explanation**
Synchronization is how threads “take turns” or “wait for each other.” Mutexes ensure only one thread accesses data at a time; condition variables let threads sleep until notified; barriers and latches make threads wait at a rendezvous point; semaphores limit how many threads can access a resource simultaneously.

### Key Characteristics

- **Happens-before relationships** – synchronization primitives establish ordering guarantees.
- **RAII wrappers** – lock guards ensure proper release.
- **Blocking vs. non-blocking** – some primitives block, others support `try_` variants.
- **Single-use vs. reusable** – latches are single-use; barriers are reusable.
- **Thread-safe primitives** – the primitives themselves are safe to use from multiple threads.
- **Composability** – primitives can be combined (e.g., mutex + condition variable).

### Prerequisites

- Solid understanding of threads and the memory model.
- Familiarity with mutexes and condition variables.
- Knowledge of data races and happens-before.
- Understanding of RAII and exception safety.

### Related Programming Areas

- **Mutexes** – mutual exclusion.
- **Condition variables** – waiting and notification.
- **Atomics** – lock-free synchronization.
- **Futures and promises** – asynchronous result passing.
- **Parallel algorithms** – implicit synchronization via execution policies.

### Core Concepts / Features

1. Synchronization primitives (overview)
2. Barriers
   - `std::barrier` (C++20)
   - `std::latch` (C++20)
3. Semaphores
   - `std::counting_semaphore` (C++20)
   - `std::binary_semaphore` (C++20)
4. Synchronization best practices

---

## 1. Synchronization Primitives (Overview)

### Definitions

**Core Definition**
Synchronization primitives are the language and library facilities that coordinate thread execution and protect shared data.

**Technical Definition**
C++ synchronization primitives include mutexes (`std::mutex`, `std::shared_mutex`), condition variables (`std::condition_variable`), atomics (`std::atomic`), futures/promises, and the C++20 synchronization types (`std::barrier`, `std::latch`, `std::counting_semaphore`, `std::binary_semaphore`). Each provides specific guarantees about ordering and mutual exclusion.

**Beginner-Friendly Explanation**
Synchronization primitives are the “traffic rules” for threads. Different primitives solve different problems: mutexes for exclusive access, condition variables for waiting, barriers for rendezvous, semaphores for resource limits.

### Purposes

- To prevent data races.
- To establish happens-before relationships.
- To coordinate thread ordering.
- To limit concurrent access to resources.

### Primitive Comparison

| Primitive | Purpose | Standard |
|-----------|---------|----------|
| `std::mutex` | Exclusive access | C++11 |
| `std::shared_mutex` | Reader-writer access | C++17 |
| `std::condition_variable` | Wait/notify | C++11 |
| `std::atomic` | Lock-free access | C++11 |
| `std::future`/`promise` | Async result | C++11 |
| `std::latch` | Single-use countdown | C++20 |
| `std::barrier` | Reusable phase sync | C++20 |
| `std::counting_semaphore` | Resource counting | C++20 |

### Annotated Code Example

```cpp
#include <mutex>
#include <iostream>

std::mutex mtx;

int main() {
    std::lock_guard<std::mutex> lock(mtx);
    std::cout << "Synchronized\n";
}
```

**Expected Output**
```
Synchronized
```

**Why this output?**  
The `lock_guard` acquires the mutex, ensuring exclusive access for the scope.

---

## 2. Barriers

### Definitions

**Core Definition**
A barrier is a synchronization point where a fixed number of threads must arrive before any can proceed.

**Technical Definition**
C++20 provides two barrier-like types: `std::latch` (single-use countdown) and `std::barrier` (reusable phase synchronization). A latch counts down to zero once; a barrier resets after each phase and can invoke a completion function.

**Beginner-Friendly Explanation**
Barriers are rendezvous points: everyone must arrive before anyone can continue. Latches are one-shot; barriers can be used repeatedly for multi-phase algorithms.

---

### 2.1 `std::barrier` (C++20)

#### Definitions

**Core Definition**
`std::barrier` is a reusable synchronization point for a fixed number of threads, with an optional completion function.

**Technical Definition**
`std::barrier<CompletionFunction>` (since C++20, in `<barrier>`) is a reusable barrier. Threads call `arrive_and_wait()` (or `arrive()` and `wait()`). When the required number of threads have arrived, the completion function (if any) is invoked, and the barrier resets for the next phase. The completion function runs on one of the participating threads.

#### Purposes

- To synchronize iterative parallel algorithms.
- To implement multi-phase computations.
- To coordinate repeated rendezvous points.

#### Syntax Rules and Structure

**General Syntax**
```cpp
#include <barrier>

std::barrier<> bar(n);                    // n participating threads
bar.arrive_and_wait();
```

**Component Breakdown**

| Operation | Description |
|-----------|-------------|
| `arrive()` | Signals arrival without waiting. |
| `wait()` | Waits for the current phase to complete. |
| `arrive_and_wait()` | Arrives and waits. |

**Constraints and Limitations**
- The number of participating threads is fixed at construction.
- Cannot be copied or moved.
- The completion function must not throw.

#### Annotated Code Example

```cpp
#include <barrier>
#include <iostream>
#include <thread>
#include <vector>

std::barrier bar(3);

void worker(int id) {
    for (int phase = 0; phase < 2; ++phase) {
        std::cout << "Worker " << id << " phase " << phase << '\n';
        bar.arrive_and_wait();
    }
}

int main() {
    std::vector<std::thread> threads;
    for (int i = 0; i < 3; ++i)
        threads.emplace_back(worker, i);
    for (auto& t : threads) t.join();
}
```

**Expected Output** (interleaving may vary)
```
Worker 0 phase 0
Worker 1 phase 0
Worker 2 phase 0
Worker 0 phase 1
Worker 1 phase 1
Worker 2 phase 1
```

**Why this output?**  
All workers synchronize at the barrier between phases; no worker can enter phase 1 until all have completed phase 0.

#### Real-World Cases

- **Iterative solvers** – synchronizing phases of a numerical algorithm.
- **Simulation steps** – advancing time steps in lockstep.

---

### 2.2 `std::latch` (C++20)

#### Definitions

**Core Definition**
`std::latch` is a single-use countdown synchronization primitive that blocks threads until a counter reaches zero.

**Technical Definition**
`std::latch` (since C++20, in `<latch>`) is a downward counter. Threads decrement the counter via `count_down()` and wait until it reaches zero via `wait()` or `arrive_and_wait()`. Once zero, all waiting threads are released, and the latch cannot be reused.

#### Purposes

- To synchronize a fixed number of tasks once.
- To implement one-time barriers.
- To coordinate startup and shutdown.

#### Syntax Rules and Structure

**General Syntax**
```cpp
#include <latch>
std::latch latch(n);
latch.count_down();
latch.wait();
latch.arrive_and_wait();
```

**Constraints and Limitations**
- Single-use; cannot be reset.
- Cannot be copied or moved.

#### Annotated Code Example

```cpp
#include <latch>
#include <thread>
#include <iostream>
#include <vector>

std::latch latch(3);

void worker(int id) {
    std::cout << "Worker " << id << " ready\n";
    latch.arrive_and_wait();
    std::cout << "Worker " << id << " running\n";
}

int main() {
    std::vector<std::thread> threads;
    for (int i = 0; i < 3; ++i)
        threads.emplace_back(worker, i);
    for (auto& t : threads) t.join();
}
```

**Expected Output** (order may vary)
```
Worker 0 ready
Worker 1 ready
Worker 2 ready
Worker 0 running
Worker 1 running
Worker 2 running
```

**Why this output?**  
All three workers call `arrive_and_wait()`; the latch releases them only after all three have arrived.

#### Real-World Cases

- **Startup synchronization** – waiting for all subsystems to initialize.
- **Parallel test setup** – ensuring all fixtures are ready.

---

## 3. Semaphores

### Definitions

**Core Definition**
A semaphore is a synchronization primitive that models a non-negative resource count, blocking threads when the count is zero.

**Technical Definition**
C++20 provides `std::counting_semaphore<N>` and its alias `std::binary_semaphore` (in `<semaphore>`). `acquire()` decrements the counter (blocking if zero); `release()` increments it. `try_acquire()` is non-blocking.

**Beginner-Friendly Explanation**
A semaphore is like a set of permits: threads take a permit to proceed and return it when done. If no permits are available, they wait.

---

### 3.1 `std::counting_semaphore` (C++20)

#### Definitions

**Core Definition**
`std::counting_semaphore<N>` is a semaphore with a maximum count of at least `N`.

**Technical Definition**
`std::counting_semaphore<LeastMaxValue>` (since C++20) maintains an internal counter. `acquire()` decrements the counter, blocking if it is zero; `release()` increments it. `try_acquire()` and timed variants are also available.

#### Purposes

- To limit concurrency to a fixed number of threads.
- To signal between threads.
- To model resource pools.

#### Syntax Rules and Structure

**General Syntax**
```cpp
#include <semaphore>
std::counting_semaphore<3> sem(3);
sem.acquire();
sem.release();
```

**Constraints and Limitations**
- Over-releasing (releasing more times than acquired) is undefined behaviour.
- Cannot be copied or moved.

#### Annotated Code Example

```cpp
#include <semaphore>
#include <thread>
#include <iostream>
#include <vector>

std::counting_semaphore<2> sem(2);

void worker(int id) {
    sem.acquire();
    std::cout << "Worker " << id << " acquired\n";
    std::this_thread::sleep_for(std::chrono::milliseconds(100));
    sem.release();
}

int main() {
    std::vector<std::thread> threads;
    for (int i = 0; i < 4; ++i)
        threads.emplace_back(worker, i);
    for (auto& t : threads) t.join();
}
```

**Expected Output** (order may vary)
```
Worker 0 acquired
Worker 1 acquired
Worker 2 acquired
Worker 3 acquired
```

**Why this output?**  
At most two workers hold the semaphore at a time; the others wait until a permit is released.

#### Real-World Cases

- **Connection pools** – limiting concurrent connections.
- **Producer-consumer** – signalling availability.
- **Resource throttling** – limiting parallel tasks.

---

### 3.2 `std::binary_semaphore` (C++20)

#### Definitions

**Core Definition**
`std::binary_semaphore` is an alias for `std::counting_semaphore<1>`, modelling a mutex-like or signal-like primitive.

**Technical Definition**
`std::binary_semaphore` is a type alias for `std::counting_semaphore<1>`. It is commonly used for signalling (e.g., waking a worker) and simple mutual exclusion.

#### Purposes

- To signal between threads.
- To implement simple mutual exclusion.
- To coordinate producer-consumer patterns.

#### Syntax Rules and Structure

**General Syntax**
```cpp
std::binary_semaphore sem(0);            // initially unavailable
sem.acquire();
sem.release();
```

#### Annotated Code Example

```cpp
#include <semaphore>
#include <thread>
#include <iostream>

std::binary_semaphore sem(0);

void worker() {
    sem.acquire();                         // wait for signal
    std::cout << "Worker woke up\n";
}

int main() {
    std::thread t(worker);
    std::this_thread::sleep_for(std::chrono::milliseconds(100));
    std::cout << "Signalling\n";
    sem.release();
    t.join();
}
```

**Expected Output**
```
Signalling
Worker woke up
```

**Why this output?**  
The worker blocks on `acquire()`; `main` releases the semaphore after 100 ms, waking the worker.

#### Real-World Cases

- **Event signalling** – waking a worker thread.
- **Simple mutex** – protecting a critical section.
- **Producer-consumer** – signalling item availability.

---

## 4. Synchronization Best Practices

### Definitions

**Core Definition**
Synchronization best practices are guidelines for choosing, using, and combining synchronization primitives correctly, safely, and efficiently.

**Technical Definition**
These practices are informed by the C++ Core Guidelines (CP.20–CP.22, CP.40, CP.50, CP.100) and community consensus: use the least powerful primitive that solves the problem; use RAII locks; avoid holding locks while waiting on other primitives; minimise critical sections; prefer higher-level abstractions (`std::async`, `std::jthread`, barriers) over raw threads; and validate with ThreadSanitizer or Helgrind.

**Beginner-Friendly Explanation**
Best practices for synchronization are simple: use the right tool for the job, hold locks briefly, and always test with race-detection tools.

### Key Guidelines

| Guideline | Summary |
|-----------|---------|
| **Use the least powerful primitive** | Mutex for exclusive access; semaphore for counting; barrier for phases. |
| **Prefer RAII locks** | `std::lock_guard`, `std::scoped_lock`. |
| **Minimise critical sections** | Hold locks briefly. |
| **Avoid nested locks** | Use `std::scoped_lock` for multiple mutexes. |
| **Do not hold locks while waiting** | Avoid holding a lock while waiting on a condition variable or barrier. |
| **Use `std::latch` for one-shot** | Use `std::barrier` for repeated phases. |
| **Use semaphores for resource limits** | Prefer semaphores over manual counting. |
| **Validate with tools** | ThreadSanitizer, Helgrind. |

### Annotated Code Example

```cpp
#include <barrier>
#include <semaphore>
#include <thread>
#include <iostream>
#include <vector>

std::barrier bar(2);
std::counting_semaphore<1> sem(1);

void worker(int id) {
    bar.arrive_and_wait();                 // synchronize start
    sem.acquire();                         // limit concurrency
    std::cout << "Worker " << id << " running\n";
    sem.release();
}

int main() {
    std::vector<std::thread> threads;
    for (int i = 0; i < 4; ++i)
        threads.emplace_back(worker, i);
    for (auto& t : threads) t.join();
}
```

**Expected Output** (interleaving may vary)
```
Worker 0 running
Worker 1 running
Worker 2 running
Worker 3 running
```

**Why this output?**  
The barrier synchronizes thread starts; the semaphore limits concurrent execution to one worker at a time.

### Real-World Cases

- **Code reviews** – checking for missing synchronization.
- **Static analysis** – ThreadSanitizer and clang-tidy.
- **Library design** – exposing safe synchronization primitives.

---

## References

- Synchronization primitives — cppreference – https://en.cppreference.com/w/cpp/thread
- `std::barrier` — cppreference – https://en.cppreference.com/w/cpp/thread/barrier
- `std::latch` — cppreference – https://en.cppreference.com/w/cpp/thread/latch
- `std::counting_semaphore` — cppreference – https://en.cppreference.com/w/cpp/thread/counting_semaphore
- `std::binary_semaphore` — cppreference – https://en.cppreference.com/w/cpp/thread/binary_semaphore
- C++ Core Guidelines — Concurrency – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-concurrency
- C++ Core Guidelines — CP.20: Use RAII, never plain `lock()`/`unlock()` – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-raii
- C++ Core Guidelines — CP.21: Use `std::lock()` or `std::scoped_lock` for multiple mutexes – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-lock
- C++ Core Guidelines — CP.40: Minimise shared data and lock duration – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-minimize
- C++ Core Guidelines — CP.100: Don't use lock-free programming unless you absolutely have to – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-lockfree
- `std::barrier` — Microsoft Learn – https://learn.microsoft.com/en-us/cpp/standard-library/barrier
- `std::latch` — Microsoft Learn – https://learn.microsoft.com/en-us/cpp/standard-library/latch

---

This cheat sheet provides a comprehensive, structured reference for C++ thread synchronization, covering synchronization primitives, barriers (`std::barrier`, `std::latch`), semaphores (`std::counting_semaphore`, `std::binary_semaphore`), and best practices, with annotated code examples, expected outputs, real-world cases, and authoritative references.