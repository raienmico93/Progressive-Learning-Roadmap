# C++ Condition Variables — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A condition variable is a synchronisation primitive that allows one or more threads to wait until another thread notifies them that a shared condition has become true.

**Technical Definition**
Condition variables are defined in `<condition_variable>`. The standard provides two types: `std::condition_variable` (used with `std::unique_lock<std::mutex>`) and `std::condition_variable_any` (used with any lockable type, including `std::shared_lock`). A condition variable is always used together with a mutex and a shared predicate. Waiting atomically releases the mutex, blocks the thread until notified, and reacquires the mutex before returning. Spurious wakeups mean waits must always be performed in a loop or with a predicate overload.

**Beginner-Friendly Explanation**
Imagine a worker who checks whether there's work to do. If there isn't, instead of constantly checking (busy-waiting), the worker goes to sleep. When a boss adds work, they wake the worker. A condition variable is that “sleep until notified” mechanism, combined with the mutex that protects the work queue.

### Key Characteristics

- **Always paired with a mutex** – the mutex protects the shared condition; the condition variable signals changes.
- **Atomic wait-and-release** – waiting releases the mutex and blocks atomically; waking reacquires the mutex.
- **Spurious wakeups** – waits can return without a notification; always re-check the predicate.
- **Predicate overloads** – `wait(lock, pred)`, `wait_for(lock, timeout, pred)`, `wait_until(lock, deadline, pred)` loop internally until the predicate is true or the timeout expires.
- **Two variants** – `condition_variable` (requires `unique_lock<mutex>`) and `condition_variable_any` (any lockable type).
- **Notifications are not stored** – a notification with no waiter is lost; use a predicate and a shared flag to avoid missed wakeups.

### Prerequisites

- Solid understanding of threads and mutexes.
- Familiarity with `std::unique_lock` and `std::lock_guard`.
- Knowledge of RAII and exception safety.
- Understanding of data races and happens-before.

### Related Programming Areas

- **Mutexes** – condition variables require a mutex to protect the shared condition.
- **Producer-consumer patterns** – the canonical use case.
- **Thread pools** – workers wait for tasks on a condition variable.
- **Semaphores, latches, barriers** – alternative synchronisation primitives (C++20).
- **Futures and promises** – higher-level alternatives for one-shot results.

### Core Concepts / Features

1. `std::condition_variable`
2. `std::condition_variable_any`
3. Condition variable operations (`wait`, `wait_for`, `wait_until`, `notify_one`, `notify_all`)
4. Predicate wait
5. Condition variable best practices

---

## 1. `std::condition_variable`

### Definitions

**Core Definition**
`std::condition_variable` is a synchronisation primitive that blocks one or more threads until notified, and is used together with a `std::unique_lock<std::mutex>`.

**Technical Definition**
`std::condition_variable` (since C++11, in `<condition_variable>`) provides `wait`, `wait_for`, `wait_until`, `notify_one`, and `notify_all`. It requires the waiting thread to hold a `std::unique_lock<std::mutex>`, which is atomically released during the wait and reacquired upon wake-up. It is not copyable.

**Beginner-Friendly Explanation**
`std::condition_variable` is the standard “sleep until notified” tool. It requires a `unique_lock` and works with a `std::mutex`.

### Purposes

- To block a thread until a condition becomes true.
- To implement producer-consumer queues.
- To avoid busy-waiting.
- To coordinate thread pools and task queues.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <condition_variable>
#include <mutex>

std::mutex mtx;
std::condition_variable cv;

void waiter() {
    std::unique_lock<std::mutex> lock(mtx);
    cv.wait(lock, []{ return /* condition */; });
    // ...
}

void notifier() {
    {
        std::lock_guard<std::mutex> lock(mtx);
        // modify shared state
    }
    cv.notify_one();
}
```

**Component Breakdown**

| Member | Description |
|--------|-------------|
| `wait(lock)` | Releases the lock and blocks until notified; reacquires the lock. |
| `wait(lock, pred)` | Loops until `pred()` is true (handles spurious wakeups). |
| `wait_for(lock, duration)` | Waits until notified or timeout. |
| `wait_until(lock, time_point)` | Waits until notified or deadline. |
| `notify_one()` | Wakes one waiting thread. |
| `notify_all()` | Wakes all waiting threads. |

**Syntax Rules**
- The mutex must be locked before calling `wait` (via `unique_lock`).
- The predicate must be checked while holding the mutex.
- `notify_one`/`notify_all` can be called without holding the mutex, but the shared condition should be updated under the mutex.

**Constraints and Limitations**
- Requires `std::unique_lock<std::mutex>`; cannot be used with `shared_lock`.
- Notifications are not stored; missed notifications are lost.
- Spurious wakeups require predicate checks.

### Annotated Code Example

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <queue>

std::mutex mtx;
std::condition_variable cv;
std::queue<int> tasks;
bool done = false;

void producer() {
    for (int i = 0; i < 5; ++i) {
        {
            std::lock_guard<std::mutex> lock(mtx);
            tasks.push(i);
            std::cout << "Produced " << i << '\n';
        }
        cv.notify_one();
    }
    {
        std::lock_guard<std::mutex> lock(mtx);
        done = true;
    }
    cv.notify_all();
}

void consumer() {
    while (true) {
        std::unique_lock<std::mutex> lock(mtx);
        cv.wait(lock, []{ return !tasks.empty() || done; });

        if (tasks.empty() && done) return;

        int task = tasks.front();
        tasks.pop();
        lock.unlock();
        std::cout << "Consumed " << task << '\n';
    }
}

int main() {
    std::thread p(producer);
    std::thread c(consumer);
    p.join(); c.join();
}
```

**Expected Output** (interleaving may vary)
```
Produced 0
Produced 1
Consumed 0
Consumed 1
...
```

**Why this output?**  
The producer pushes tasks and notifies the consumer; the consumer waits on the condition variable until tasks are available or the producer signals completion.

### Real-World Cases

- **Producer-consumer queues** – task queues in thread pools.
- **Event loops** – waiting for incoming events.
- **Resource pools** – waiting for a free resource.

---

## 2. `std::condition_variable_any`

### Definitions

**Core Definition**
`std::condition_variable_any` is a condition variable that can be used with any lockable type, not just `std::unique_lock<std::mutex>`.

**Technical Definition**
`std::condition_variable_any` (since C++11) provides the same interface as `condition_variable` but accepts any `Lockable` type, including `std::shared_lock`, `std::unique_lock<std::shared_mutex>`, and user-defined locks. It is more flexible but typically has slightly higher overhead because it may allocate or use a mutex internally to coordinate.

**Beginner-Friendly Explanation**
`condition_variable_any` is the flexible version: it works with any kind of lock, not just `unique_lock<mutex>`.

### Purposes

- To work with custom lock types.
- To work with `shared_lock` and reader-writer mutexes.
- To support non-standard lockable types.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::condition_variable_any cv;
std::shared_lock lock(shm);
cv.wait(lock, pred);
```

**Constraints and Limitations**
- Slightly higher overhead than `condition_variable`.
- The lock must satisfy the `Lockable` requirements.

### Annotated Code Example

```cpp
#include <iostream>
#include <thread>
#include <shared_mutex>
#include <condition_variable>

std::shared_mutex shm;
std::condition_variable_any cv;
bool ready = false;

void waiter() {
    std::shared_lock lock(shm);
    cv.wait(lock, []{ return ready; });
    std::cout << "Ready!\n";
}

void notifier() {
    {
        std::unique_lock lock(shm);
        ready = true;
    }
    cv.notify_one();
}

int main() {
    std::thread w(waiter);
    std::thread n(notifier);
    w.join(); n.join();
}
```

**Expected Output**
```
Ready!
```

**Why this output?**  
`condition_variable_any` works with a `shared_lock`; the notifier sets `ready` and wakes the waiter.

### Real-World Cases

- **Reader-writer scenarios** – waiting on a shared mutex.
- **Custom lock types** – integrating with non-standard synchronisation.

---

## 3. Condition Variable Operations

### Definitions

**Core Definition**
Condition variable operations are the member functions used to wait for notifications and to notify waiting threads.

**Technical Definition**
The operations are `wait`, `wait_for`, `wait_until`, `notify_one`, and `notify_all`. The waiting operations atomically release the associated lock, block the thread, and reacquire the lock upon wake-up. The notification operations unblock one or all waiting threads.

**Beginner-Friendly Explanation**
These are the commands you use with a condition variable: wait for a notification, wait with a timeout, wake one waiter, or wake all waiters.

---

### 3.1 `wait`

**Core Definition**
`wait(lock)` blocks the thread until notified, releasing and reacquiring the lock atomically.

**Syntax**
```cpp
cv.wait(lock);                           // without predicate
cv.wait(lock, pred);                     // with predicate
```

**Constraints**
- Spurious wakeups are possible; always use the predicate overload or a loop.

**Example**
```cpp
std::unique_lock lock(mtx);
cv.wait(lock, []{ return ready; });
```

---

### 3.2 `wait_for`

**Core Definition**
`wait_for(lock, duration)` blocks until notified or the duration expires.

**Syntax**
```cpp
cv.wait_for(lock, std::chrono::milliseconds(100));
cv.wait_for(lock, std::chrono::milliseconds(100), pred);
```

**Constraints**
- Returns `std::cv_status::timeout` on timeout (without predicate) or a `bool` (with predicate).

**Example**
```cpp
std::unique_lock lock(mtx);
if (cv.wait_for(lock, std::chrono::milliseconds(100), []{ return ready; })) {
    // ready
} else {
    // timeout
}
```

---

### 3.3 `wait_until`

**Core Definition**
`wait_until(lock, time_point)` blocks until notified or the deadline is reached.

**Syntax**
```cpp
cv.wait_until(lock, std::chrono::steady_clock::now() + std::chrono::seconds(1));
```

**Example**
```cpp
std::unique_lock lock(mtx);
auto deadline = std::chrono::steady_clock::now() + std::chrono::milliseconds(100);
if (cv.wait_until(lock, deadline, []{ return ready; })) {
    // ready
}
```

---

### 3.4 `notify_one`

**Core Definition**
`notify_one()` wakes one thread waiting on the condition variable.

**Syntax**
```cpp
cv.notify_one();
```

**Constraints**
- If no threads are waiting, the notification is lost.

---

### 3.5 `notify_all`

**Core Definition**
`notify_all()` wakes all threads waiting on the condition variable.

**Syntax**
```cpp
cv.notify_all();
```

**Constraints**
- Wakes all waiters; each must re-check its predicate.

---

## 4. Predicate Wait

### Definitions

**Core Definition**
A predicate wait is the use of the predicate overloads of `wait`, `wait_for`, or `wait_until` to loop until the condition is true, handling spurious wakeups automatically.

**Technical Definition**
The predicate overloads are equivalent to:
```cpp
while (!pred()) {
    wait(lock);
}
```
but implemented efficiently. They return `true` if the predicate became true, and `false` if the wait timed out (for the timed variants). They are the recommended way to use condition variables.

**Beginner-Friendly Explanation**
A predicate wait means “wait until this condition is true.” It handles spurious wakeups for you, so you don't need to write the loop yourself.

### Purposes

- To handle spurious wakeups.
- To express the waiting condition clearly.
- To avoid missed notifications.

### Syntax Rules and Structure

**General Syntax**
```cpp
cv.wait(lock, pred);
```

**Constraints and Limitations**
- The predicate must be safe to call while holding the lock.
- The predicate should check the shared condition, not just a flag set by the notifier.

### Annotated Code Example

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>

std::mutex mtx;
std::condition_variable cv;
bool ready = false;

void waiter() {
    std::unique_lock lock(mtx);
    cv.wait(lock, []{ return ready; });    // predicate wait
    std::cout << "Ready!\n";
}

void notifier() {
    {
        std::lock_guard lock(mtx);
        ready = true;
    }
    cv.notify_one();
}

int main() {
    std::thread w(waiter);
    std::thread n(notifier);
    w.join(); n.join();
}
```

**Expected Output**
```
Ready!
```

**Why this output?**  
The predicate wait loops until `ready` is true, handling any spurious wakeups.

### Real-World Cases

- **All condition variable usage** – predicate waits are the recommended pattern.
- **Producer-consumer** – wait until tasks are available.

---

## 5. Condition Variable Best Practices

### Definitions

**Core Definition**
Condition variable best practices are guidelines for using condition variables correctly, safely, and efficiently.

**Technical Definition**
These practices are informed by the C++ Core Guidelines (CP.42: “Don't wait without a condition”; CP.43: “Consider passing a `lock_guard` to `condition_variable::wait`”; CP.41: “Minimize thread creation”) and community consensus: always use predicate waits; always protect the shared condition with a mutex; update the condition under the mutex; notify after unlocking when possible; use `notify_all` for multiple waiters; and be aware of lost wakeups.

**Beginner-Friendly Explanation**
Best practices for condition variables are simple: always use predicate waits, always hold the mutex when checking the condition, and notify after updating the shared state.

### Key Guidelines

| Guideline | Summary |
|-----------|---------|
| **CP.42** | Don't wait without a condition (always use a predicate). |
| **CP.43** | Pass a `lock_guard` to `wait` when possible (use `unique_lock`). |
| **Protect the condition with a mutex** | The shared state must be safe to check. |
| **Update the condition under the mutex** | Ensures visibility and prevents races. |
| **Notify after unlocking** | Reduces contention (the waiter can acquire the lock immediately). |
| **Handle spurious wakeups** | Use predicate overloads. |
| **Use `notify_all` for multiple waiters** | Ensures all relevant threads wake. |
| **Avoid lost wakeups** | The predicate prevents missed notifications. |

### Annotated Code Example

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <queue>

std::mutex mtx;
std::condition_variable cv;
std::queue<int> tasks;
bool done = false;

void producer() {
    for (int i = 0; i < 3; ++i) {
        {
            std::lock_guard lock(mtx);
            tasks.push(i);
        } // unlock before notify
        cv.notify_one();
    }
    {
        std::lock_guard lock(mtx);
        done = true;
    }
    cv.notify_all();
}

void consumer() {
    while (true) {
        std::unique_lock lock(mtx);
        cv.wait(lock, []{ return !tasks.empty() || done; });
        if (tasks.empty() && done) return;
        int task = tasks.front();
        tasks.pop();
        lock.unlock();
        std::cout << "Consumed " << task << '\n';
    }
}

int main() {
    std::thread p(producer);
    std::thread c(consumer);
    p.join(); c.join();
}
```

**Expected Output** (interleaving may vary)
```
Consumed 0
Consumed 1
Consumed 2
```

**Why this output?**  
The producer updates `tasks` under the mutex, unlocks, then notifies. The consumer uses a predicate wait to check the condition, and handles the `done` flag correctly.

### Real-World Cases

- **Code reviews** – flagging waits without predicates.
- **Static analysis** – ThreadSanitizer and clang-tidy.
- **Library design** – implementing producer-consumer queues.

---

## References

- `std::condition_variable` — cppreference – https://en.cppreference.com/w/cpp/thread/condition_variable
- `std::condition_variable_any` — cppreference – https://en.cppreference.com/w/cpp/thread/condition_variable_any
- `std::condition_variable::wait` — cppreference – https://en.cppreference.com/w/cpp/thread/condition_variable/wait
- `std::condition_variable::wait_for` — cppreference – https://en.cppreference.com/w/cpp/thread/condition_variable/wait_for
- `std::condition_variable::wait_until` — cppreference – https://en.cppreference.com/w/cpp/thread/condition_variable/wait_until
- `std::condition_variable::notify_one` — cppreference – https://en.cppreference.com/w/cpp/thread/condition_variable/notify_one
- `std::condition_variable::notify_all` — cppreference – https://en.cppreference.com/w/cpp/thread/condition_variable/notify_all
- C++ Core Guidelines — CP.42: Don't wait without a condition – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-wait
- C++ Core Guidelines — CP.43: Consider passing a `lock_guard` to `condition_variable::wait` – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-waitfor
- Condition variable — Wikipedia – https://en.wikipedia.org/wiki/Monitor_(synchronization)

---

This cheat sheet provides a comprehensive, structured reference for C++ condition variables, covering `std::condition_variable`, `std::condition_variable_any`, the operations (`wait`, `wait_for`, `wait_until`, `notify_one`, `notify_all`), predicate waits, and best practices, with annotated code examples, expected outputs, real-world cases, and authoritative references.