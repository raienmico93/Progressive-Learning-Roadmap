# C++ Mutexes — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A mutex (mutual exclusion) is a synchronisation primitive that allows only one thread at a time to access a shared resource, preventing data races and ensuring consistency.

**Technical Definition**
The C++ Standard Library provides several mutex types in `<mutex>` and `<shared_mutex>`: `std::mutex` (exclusive ownership), `std::recursive_mutex` (re-entrant), `std::timed_mutex` (exclusive with timeouts), `std::recursive_timed_mutex` (re-entrant with timeouts), `std::shared_mutex` (C++17, reader-writer), and `std::shared_timed_mutex` (C++14, reader-writer with timeouts). These types satisfy the *Mutex* and *TimedMutex* named requirements. Mutexes are typically paired with RAII lock guards (`std::lock_guard`, `std::unique_lock`, `std::shared_lock`, `std::scoped_lock`) to guarantee unlocking on all exit paths, including exceptions.

**Beginner-Friendly Explanation**
A mutex is like a lock on a door: only one thread can hold the key at a time. When a thread wants to access shared data, it locks the mutex; other threads wait until the mutex is unlocked. Lock guards are “smart locks” that release the mutex automatically when you're done, even if an exception is thrown.

### Key Characteristics

- **Mutual exclusion** – only one thread owns a `std::mutex` at a time.
- **RAII guards** – lock guards unlock automatically on scope exit.
- **Non-copyable** – mutexes cannot be copied or moved.
- **Ownership semantics** – a thread that locks a mutex should unlock it; RAII guards handle this.
- **Reader-writer variants** – `shared_mutex` allows multiple readers or one writer.
- **Recursive variants** – `recursive_mutex` allows the same thread to lock multiple times.
- **Timeouts** – timed variants support `try_lock_for` and `try_lock_until`.

### Prerequisites

- Solid understanding of threads and concurrency.
- Familiarity with the memory model and data races.
- Knowledge of RAII and exception safety.
- Awareness of deadlock and lock ordering.

### Related Programming Areas

- **Threads** – mutexes coordinate shared access between threads.
- **Atomics** – lock-free alternatives for simple shared variables.
- **Condition variables** – used with mutexes for waiting/notification.
- **RAII** – lock guards are RAII wrappers for mutexes.
- **Deadlock avoidance** – `std::lock` and `std::scoped_lock` prevent deadlocks.

### Core Concepts / Features

1. `std::mutex`
2. `std::recursive_mutex`
3. `std::timed_mutex`
4. `std::recursive_timed_mutex`
5. `std::shared_mutex` (C++17)
6. `std::shared_timed_mutex` (C++14)
7. Lock guards: `std::lock_guard`, `std::unique_lock`, `std::shared_lock`, `std::scoped_lock`
8. `std::lock` / `std::try_lock`
9. Mutex best practices

---

## 1. `std::mutex`

### Definitions

**Core Definition**
`std::mutex` is a mutual exclusion primitive that provides exclusive, non-recursive ownership.

**Technical Definition**
`std::mutex` (since C++11, in `<mutex>`) is a class that satisfies the *Mutex* requirements. It provides `lock()`, `try_lock()`, and `unlock()`. A thread that locks a `std::mutex` must unlock it before any other thread can lock it. Locking a mutex that the same thread already owns is undefined behaviour (deadlock). `std::mutex` is non-copyable and non-movable.

**Beginner-Friendly Explanation**
`std::mutex` is the standard lock. Only one thread can hold it at a time; others must wait.

### Purposes

- To protect shared data from concurrent access.
- To prevent data races.
- To serialise access to critical sections.
- To coordinate threads.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <mutex>

std::mutex mtx;
mtx.lock();
// critical section
mtx.unlock();
```

**Component Breakdown**

| Member | Description |
|--------|-------------|
| `lock()` | Acquires the mutex, blocking if necessary. |
| `try_lock()` | Attempts to acquire; returns `false` if unavailable. |
| `unlock()` | Releases the mutex. |

**Syntax Rules**
- `lock()` must be paired with `unlock()`.
- `try_lock()` returns `true` on success, `false` otherwise.
- Locking an already-owned mutex is undefined behaviour.

**Constraints and Limitations**
- Non-recursive; a thread cannot lock it twice.
- Non-copyable and non-movable.
- Manual lock/unlock is error-prone; use RAII guards.

### Annotated Code Example

```cpp
#include <iostream>
#include <mutex>
#include <thread>

std::mutex mtx;
int counter = 0;

void increment() {
    for (int i = 0; i < 1000; ++i) {
        mtx.lock();
        ++counter;
        mtx.unlock();
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
The mutex serialises access to `counter`, preventing a data race.

### Real-World Cases

- **Shared counters** – incrementing a global counter.
- **Logging** – serialising log output.
- **Caches** – protecting cache data.

---

## 2. `std::recursive_mutex`

### Definitions

**Core Definition**
`std::recursive_mutex` is a mutex that allows the same thread to lock it multiple times without deadlocking.

**Technical Definition**
`std::recursive_mutex` (since C++11) satisfies the *Mutex* requirements and additionally allows the owning thread to acquire the mutex recursively. The mutex must be unlocked as many times as it was locked. It is useful for recursive functions that need to protect shared state.

**Beginner-Friendly Explanation**
A recursive mutex is a lock that the same thread can re-acquire. This is useful when a locked function calls another function that also needs the same lock.

### Purposes

- To support recursive locking in recursive algorithms.
- To avoid deadlocks in re-entrant code.
- To simplify code that calls locked functions from within locked functions.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::recursive_mutex rmtx;
rmtx.lock();
rmtx.lock();                            // OK: same thread
rmtx.unlock();
rmtx.unlock();
```

**Component Breakdown**

| Member | Description |
|--------|-------------|
| `lock()` | Acquires recursively; increments the count. |
| `unlock()` | Decrements the count; releases when zero. |
| `try_lock()` | Attempts to acquire. |

**Syntax Rules**
- The same number of `unlock()` calls as `lock()` calls must be made.
- Other threads must wait until all locks are released.

**Constraints and Limitations**
- Slightly higher overhead than `std::mutex`.
- Often a sign of design issues; prefer non-recursive designs.

### Annotated Code Example

```cpp
#include <iostream>
#include <mutex>

std::recursive_mutex rmtx;
int depth = 0;

void recurse(int n) {
    std::lock_guard<std::recursive_mutex> lock(rmtx);
    ++depth;
    if (n > 0) recurse(n - 1);
    std::cout << "Depth: " << depth << '\n';
}

int main() {
    recurse(3);
}
```

**Expected Output**
```
Depth: 4
Depth: 4
Depth: 4
Depth: 4
```

**Why this output?**  
The recursive mutex allows the same thread to lock it at each recursion level; `depth` reaches 4.

### Real-World Cases

- **Recursive algorithms** – tree traversals with shared state.
- **Re-entrant callbacks** – callbacks that re-enter the same code.

---

## 3. `std::timed_mutex`

### Definitions

**Core Definition**
`std::timed_mutex` is a mutex that supports timed lock attempts (`try_lock_for` and `try_lock_until`).

**Technical Definition**
`std::timed_mutex` (since C++11) satisfies the *TimedMutex* requirements. It adds `try_lock_for(duration)` and `try_lock_until(time_point)` to the standard mutex interface. These functions block until the lock is acquired or the timeout expires, returning `true` on success and `false` on timeout.

**Beginner-Friendly Explanation**
A timed mutex is a lock that lets you say “try to acquire it, but give up after a certain amount of time.”

### Purposes

- To avoid indefinite blocking.
- To implement timeouts and retries.
- To prevent deadlocks in bounded-time scenarios.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::timed_mutex tmtx;
if (tmtx.try_lock_for(std::chrono::milliseconds(100))) {
    // acquired
    tmtx.unlock();
} else {
    // timeout
}
```

**Component Breakdown**

| Member | Description |
|--------|-------------|
| `try_lock_for(duration)` | Tries to acquire within a duration. |
| `try_lock_until(time_point)` | Tries to acquire by a deadline. |

**Constraints and Limitations**
- Timeouts are minimums; actual wait may be longer.
- Non-recursive.

### Annotated Code Example

```cpp
#include <iostream>
#include <mutex>
#include <thread>
#include <chrono>

std::timed_mutex tmtx;

void worker() {
    if (tmtx.try_lock_for(std::chrono::milliseconds(100))) {
        std::cout << "Locked\n";
        std::this_thread::sleep_for(std::chrono::milliseconds(200));
        tmtx.unlock();
    } else {
        std::cout << "Timeout\n";
    }
}

int main() {
    std::thread t1(worker), t2(worker);
    t1.join(); t2.join();
}
```

**Expected Output** (order may vary)
```
Locked
Timeout
```

**Why this output?**  
One thread acquires the lock; the other times out after 100 ms because the first holds it for 200 ms.

### Real-World Cases

- **Network protocols** – timeouts on resource acquisition.
- **Real-time systems** – bounded blocking.
- **Deadlock avoidance** – fallback on timeout.

---

## 4. `std::recursive_timed_mutex`

### Definitions

**Core Definition**
`std::recursive_timed_mutex` is a mutex that supports both recursive locking and timed lock attempts.

**Technical Definition**
`std::recursive_timed_mutex` (since C++11) satisfies the *TimedMutex* requirements and allows recursive locking by the owning thread. It combines the features of `recursive_mutex` and `timed_mutex`.

**Beginner-Friendly Explanation**
It's a recursive lock that also supports timeouts.

### Purposes

- To support recursive locking with timeouts.
- To combine recursive and timed behaviour.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::recursive_timed_mutex rtm;
rtm.try_lock_for(std::chrono::milliseconds(100));
rtm.unlock();
```

**Constraints and Limitations**
- Higher overhead.
- Rarely needed; prefer simpler designs.

### Annotated Code Example

```cpp
#include <iostream>
#include <mutex>
#include <chrono>

std::recursive_timed_mutex rtm;

int main() {
    if (rtm.try_lock_for(std::chrono::milliseconds(100))) {
        std::cout << "Locked\n";
        rtm.unlock();
    }
}
```

**Expected Output**
```
Locked
```

### Real-World Cases

- **Complex recursive algorithms** – with bounded waiting.
- **Legacy code** – where recursion and timeouts are both needed.

---

## 5. `std::shared_mutex` (C++17)

### Definitions

**Core Definition**
`std::shared_mutex` is a reader-writer mutex that allows multiple readers or one writer.

**Technical Definition**
`std::shared_mutex` (since C++17, in `<shared_mutex>`) satisfies the *SharedMutex* requirements. It provides `lock()`/`unlock()` for exclusive (write) ownership and `lock_shared()`/`unlock_shared()` for shared (read) ownership. Multiple threads may hold shared ownership simultaneously, but exclusive ownership excludes all others.

**Beginner-Friendly Explanation**
A shared mutex lets many readers read at the same time, but a writer must have exclusive access. It is useful for read-heavy data.

### Purposes

- To allow concurrent readers while ensuring exclusive writer access.
- To protect read-heavy data structures.
- To improve performance in read-mostly workloads.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <shared_mutex>

std::shared_mutex shm;
std::shared_lock lock(shm);            // read
std::unique_lock lock(shm);            // write
```

**Constraints and Limitations**
- Writers may starve if readers are frequent.
- Higher overhead than `std::mutex`.

### Annotated Code Example

```cpp
#include <iostream>
#include <shared_mutex>
#include <thread>
#include <vector>

std::shared_mutex shm;
int data = 0;

void reader(int id) {
    std::shared_lock lock(shm);
    std::cout << "Reader " << id << ": " << data << '\n';
}

void writer() {
    std::unique_lock lock(shm);
    ++data;
}

int main() {
    std::thread w(writer);
    w.join();
    std::thread r1(reader, 1), r2(reader, 2);
    r1.join(); r2.join();
}
```

**Expected Output** (order may vary)
```
Reader 1: 1
Reader 2: 1
```

**Why this output?**  
The writer increments `data` under exclusive lock; both readers acquire shared locks and see the updated value.

### Real-World Cases

- **Caches** – many readers, occasional updates.
- **Configuration data** – read by many threads, reloaded rarely.

---

## 6. `std::shared_timed_mutex` (C++14)

### Definitions

**Core Definition**
`std::shared_timed_mutex` is a reader-writer mutex with timeout support.

**Technical Definition**
`std::shared_timed_mutex` (since C++14) satisfies the *SharedTimedMutex* requirements. It provides timed exclusive and shared lock attempts: `try_lock_for`, `try_lock_until`, `try_lock_shared_for`, `try_lock_shared_until`.

**Beginner-Friendly Explanation**
A shared timed mutex is a reader-writer lock that also supports timeouts.

### Purposes

- To combine reader-writer semantics with timeouts.
- To avoid indefinite blocking in read-heavy scenarios.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::shared_timed_mutex stm;
stm.try_lock_shared_for(std::chrono::milliseconds(100));
stm.unlock_shared();
```

**Constraints and Limitations**
- Requires C++14.
- Higher overhead than `shared_mutex`.

### Annotated Code Example

```cpp
#include <iostream>
#include <shared_mutex>
#include <chrono>

std::shared_timed_mutex stm;

int main() {
    if (stm.try_lock_shared_for(std::chrono::milliseconds(100))) {
        std::cout << "Shared lock acquired\n";
        stm.unlock_shared();
    }
}
```

**Expected Output**
```
Shared lock acquired
```

### Real-World Cases

- **Read-heavy systems with deadlines** – time-bounded reads.
- **Legacy code** – where C++14 is the target.

---

## 7. Lock Guards

### Definitions

**Core Definition**
Lock guards are RAII wrappers that acquire a mutex on construction and release it on destruction, guaranteeing unlock on all exit paths.

**Technical Definition**
The standard provides four lock guards: `std::lock_guard` (simple exclusive lock), `std::unique_lock` (movable, flexible exclusive lock), `std::shared_lock` (shared lock), and `std::scoped_lock` (C++17, multi-mutex deadlock-avoiding lock). All are RAII wrappers that unlock the mutex in their destructor.

**Beginner-Friendly Explanation**
Lock guards are “smart locks” that automatically unlock when they go out of scope, even if an exception is thrown.

### Purposes

- To guarantee unlocking on all exit paths.
- To simplify lock management.
- To avoid deadlocks from forgotten unlocks.

---

### 7.1 `std::lock_guard`

**Core Definition**
`std::lock_guard` is a simple RAII wrapper that locks a mutex on construction and unlocks it on destruction.

**Syntax**
```cpp
std::lock_guard<std::mutex> lock(mtx);
```

**Constraints**
- Non-copyable, non-movable.
- Locks for the entire scope.

**Example**
```cpp
#include <mutex>
#include <iostream>

std::mutex mtx;

int main() {
    std::lock_guard<std::mutex> lock(mtx);
    std::cout << "Locked\n";
} // unlocked
```

**Expected Output**
```
Locked
```

---

### 7.2 `std::unique_lock`

**Core Definition**
`std::unique_lock` is a movable RAII wrapper that supports deferred locking, timed locking, and manual lock/unlock.

**Syntax**
```cpp
std::unique_lock<std::mutex> lock(mtx);
std::unique_lock<std::mutex> lock(mtx, std::defer_lock);
lock.lock();
lock.unlock();
```

**Constraints**
- Movable, not copyable.
- Higher overhead than `lock_guard`.

**Example**
```cpp
#include <mutex>
#include <iostream>

std::mutex mtx;

int main() {
    std::unique_lock<std::mutex> lock(mtx, std::defer_lock);
    lock.lock();
    std::cout << "Locked\n";
    lock.unlock();
    std::cout << "Unlocked\n";
}
```

**Expected Output**
```
Locked
Unlocked
```

---

### 7.3 `std::shared_lock`

**Core Definition**
`std::shared_lock` is a movable RAII wrapper for shared (read) ownership of a shared mutex.

**Syntax**
```cpp
std::shared_lock<std::shared_mutex> lock(shm);
```

**Example**
```cpp
#include <shared_mutex>
#include <iostream>

std::shared_mutex shm;

int main() {
    std::shared_lock lock(shm);
    std::cout << "Shared lock\n";
} // unlocked
```

**Expected Output**
```
Shared lock
```

---

### 7.4 `std::scoped_lock` (C++17)

**Core Definition**
`std::scoped_lock` is a RAII wrapper that can lock multiple mutexes simultaneously using a deadlock-avoidance algorithm.

**Syntax**
```cpp
std::scoped_lock lock(mtx1, mtx2);
```

**Example**
```cpp
#include <mutex>
#include <iostream>

std::mutex m1, m2;

int main() {
    std::scoped_lock lock(m1, m2);     // deadlock-avoiding
    std::cout << "Both locked\n";
} // both unlocked
```

**Expected Output**
```
Both locked
```

---

## 8. `std::lock` / `std::try_lock`

### Definitions

**Core Definition**
`std::lock` locks multiple mutexes without deadlock; `std::try_lock` attempts to lock multiple mutexes and returns an index of the first failure (or -1 on success).

**Technical Definition**
`std::lock(m1, m2, ...)` locks all mutexes using a deadlock-avoidance algorithm. `std::try_lock(m1, m2, ...)` attempts to lock all; on failure, it unlocks any acquired mutexes and returns the zero-based index of the mutex that failed (or -1 if all succeeded).

**Beginner-Friendly Explanation**
`std::lock` safely locks several mutexes at once. `std::try_lock` tries to lock several and tells you which one failed.

### Syntax
```cpp
std::lock(m1, m2);
int result = std::try_lock(m1, m2);
```

### Example
```cpp
#include <mutex>
#include <iostream>

std::mutex m1, m2;

int main() {
    std::lock(m1, m2);
    std::cout << "Both locked\n";
    m1.unlock();
    m2.unlock();

    int idx = std::try_lock(m1, m2);
    std::cout << "try_lock result: " << idx << '\n';   // -1 (success)
    m1.unlock();
    m2.unlock();
}
```

**Expected Output**
```
Both locked
try_lock result: -1
```

### Real-World Cases

- **Transfer between accounts** – locking both account mutexes.
- **Multi-resource acquisition** – deadlock-free locking.

---

## 9. Mutex Best Practices

### Definitions

**Core Definition**
Mutex best practices are guidelines for using mutexes correctly, safely, and efficiently in concurrent C++ programs.

**Technical Definition**
These practices are codified in the C++ Core Guidelines (CP.20–CP.22, CP.40, CP.50) and community consensus: use RAII lock guards, never lock manually; use `std::scoped_lock` or `std::lock` for multiple mutexes; hold locks for the shortest possible duration; avoid calling user code while holding a lock; never use `volatile` for synchronisation; prefer `std::atomic` for simple shared variables; and use `std::jthread` with stop tokens for cancellation.

**Beginner-Friendly Explanation**
Best practices for mutexes are simple: use RAII lock guards, lock multiple mutexes with `std::scoped_lock`, hold locks briefly, and never call unknown code while holding a lock.

### Key Guidelines

| Guideline | Summary |
|-----------|---------|
| **CP.20** | Use RAII, never plain `lock()`/`unlock()`. |
| **CP.21** | Use `std::lock()` or `std::scoped_lock` for multiple mutexes. |
| **CP.22** | Never call unknown code while holding a lock. |
| **CP.40** | Minimise shared data and lock duration. |
| **CP.50** | Define and use a mutex together with the data it protects. |
| **Prefer `std::atomic`** | For simple shared variables. |
| **Avoid recursive mutexes** | Prefer non-recursive designs. |
| **Use `std::shared_mutex` for read-heavy data** | Allow concurrent readers. |

### Annotated Code Example

```cpp
#include <mutex>
#include <thread>
#include <iostream>

class Counter {
    mutable std::mutex mtx_;
    int value_ = 0;
public:
    void increment() {
        std::lock_guard<std::mutex> lock(mtx_);
        ++value_;
    }
    int value() const {
        std::lock_guard<std::mutex> lock(mtx_);
        return value_;
    }
};

int main() {
    Counter c;
    std::thread t1([&]{ for (int i = 0; i < 1000; ++i) c.increment(); });
    std::thread t2([&]{ for (int i = 0; i < 1000; ++i) c.increment(); });
    t1.join(); t2.join();
    std::cout << c.value() << '\n';        // 2000
}
```

**Expected Output**
```
2000
```

**Why this output?**  
The `Counter` class encapsulates its mutex with its data; `lock_guard` ensures safe access.

### Real-World Cases

- **Code reviews** – flagging manual `lock()`/`unlock()`.
- **Static analysis** – ThreadSanitizer and clang-tidy.
- **Library design** – encapsulating mutexes with their data.

---

## References

- `std::mutex` — cppreference – https://en.cppreference.com/w/cpp/thread/mutex
- `std::recursive_mutex` — cppreference – https://en.cppreference.com/w/cpp/thread/recursive_mutex
- `std::timed_mutex` — cppreference – https://en.cppreference.com/w/cpp/thread/timed_mutex
- `std::recursive_timed_mutex` — cppreference – https://en.cppreference.com/w/cpp/thread/recursive_timed_mutex
- `std::shared_mutex` — cppreference – https://en.cppreference.com/w/cpp/thread/shared_mutex
- `std::shared_timed_mutex` — cppreference – https://en.cppreference.com/w/cpp/thread/shared_timed_mutex
- `std::lock_guard` — cppreference – https://en.cppreference.com/w/cpp/thread/lock_guard
- `std::unique_lock` — cppreference – https://en.cppreference.com/w/cpp/thread/unique_lock
- `std::shared_lock` — cppreference – https://en.cppreference.com/w/cpp/thread/shared_lock
- `std::scoped_lock` — cppreference – https://en.cppreference.com/w/cpp/thread/scoped_lock
- `std::lock` — cppreference – https://en.cppreference.com/w/cpp/thread/lock
- `std::try_lock` — cppreference – https://en.cppreference.com/w/cpp/thread/try_lock
- C++ Core Guidelines — CP.20: Use RAII, never plain `lock()`/`unlock()` – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-raii
- C++ Core Guidelines — CP.21: Use `std::lock()` or `std::scoped_lock` for multiple mutexes – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-lock
- C++ Core Guidelines — CP.22: Never call unknown code while holding a lock – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-unknown
- C++ Core Guidelines — CP.50: Define and use a mutex together with the data it protects – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-mutex

---

This cheat sheet provides a comprehensive, structured reference for C++ mutexes, covering `std::mutex`, `std::recursive_mutex`, `std::timed_mutex`, `std::recursive_timed_mutex`, `std::shared_mutex`, `std::shared_timed_mutex`, lock guards (`std::lock_guard`, `std::unique_lock`, `std::shared_lock`, `std::scoped_lock`), `std::lock`/`std::try_lock`, and best practices, with annotated code examples, expected outputs, real-world cases, and authoritative references.