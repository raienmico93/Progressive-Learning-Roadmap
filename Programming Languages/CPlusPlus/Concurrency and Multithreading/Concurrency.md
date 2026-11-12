# C++ Concurrency Fundamentals — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Concurrency is the ability of a program to have multiple independent execution flows (threads or processes) that appear to run simultaneously, while parallelism is the actual simultaneous execution of multiple tasks on multiple processing units.

**Technical Definition**
Concurrency is the *composition* of independently executing tasks; parallelism is the *simultaneous execution* of those tasks. A program is concurrent if it is structured to handle multiple tasks logically at once (even on a single core via interleaving); it is parallel if those tasks physically execute at the same time on multiple cores. C++ models concurrency primarily through **threads** (`std::thread`), with **processes** provided by the operating system. The C++ memory model and standard library (since C++11) define the rules for data-race-free concurrent execution.

**Beginner-Friendly Explanation**
Concurrency means your program is *organised* to do multiple things at once—like a chef who chops vegetables while the sauce simmers. Parallelism means those things literally happen at the same instant—like two chefs working side by side. Threads are the “workers” inside your program; processes are separate programs with their own memory.

### Key Characteristics

- **Logical vs. physical** – concurrency is about structure; parallelism is about hardware execution.
- **Shared state** – threads in the same process share memory; processes do not (by default).
- **Data races** – concurrent conflicting accesses to shared memory without synchronisation cause undefined behaviour.
- **Synchronisation primitives** – mutexes, atomics, condition variables, and futures coordinate threads.
- **Scheduling** – the operating system decides when and where threads run; C++ does not guarantee scheduling order.
- **Context switching** – the OS saves and restores thread state when switching between threads, which has a cost.

### Prerequisites

- Basic C++ syntax (functions, classes, lambdas).
- Understanding of object lifetime and the memory model.
- Familiarity with `std::thread`, `std::mutex`, and `std::atomic` (or willingness to learn).
- Awareness of undefined behaviour and data races.

### Related Programming Areas

- **Memory model** – defines happens-before and data-race rules.
- **Atomics and memory ordering** – fine-grained synchronisation.
- **Thread synchronisation** – mutexes, condition variables, futures.
- **Parallel algorithms** – `std::execution` policies and C++17 parallel algorithms.
- **Asynchronous programming** – `std::future`, `std::async`, and coroutines.

### Core Concepts / Features

1. Concurrency
2. Parallelism
3. Threads
4. Processes
5. Thread lifecycle
6. Thread scheduling
7. Context switching
8. Concurrency best practices

---

## 1. Concurrency

### Definitions

**Core Definition**
Concurrency is the ability of a program to make progress on multiple tasks by interleaving their execution, creating the illusion of simultaneity.

**Technical Definition**
Concurrency is the decomposition of a program into independently executing tasks that can make progress without waiting for each other. In C++, concurrency is achieved through threads (and, at a coarser level, processes), with coordination via the standard library's synchronisation primitives. Concurrency does not require multiple cores—a single-core machine can run concurrent programs via time-slicing.

**Beginner-Friendly Explanation**
Concurrency is about *structure*: writing a program as multiple tasks that can make progress independently. It is like a chef who starts boiling water, then chops vegetables while the water heats—the chef is doing one thing at a time, but the *program* is structured to handle both tasks.

### Purposes

- To improve responsiveness (e.g., UI stays responsive while background work runs).
- To structure programs around independent tasks.
- To exploit multiple cores (parallelism).
- To overlap I/O with computation.
- To model real-world concurrent systems.

### Syntax Rules and Structure

**Concurrency Patterns in C++**

| Pattern | Mechanism |
|---------|-----------|
| OS threads | `std::thread` |
| Async tasks | `std::async`, `std::future` |
| Parallel algorithms | `std::execution::par` |
| Coroutines (C++20) | `co_await`, `co_yield` |

**Syntax Rules**
- Shared mutable state requires synchronisation (`std::mutex`, `std::atomic`).
- Data races result in undefined behaviour.
- `std::thread` must be joined or detached before destruction.

**Constraints and Limitations**
- Concurrency introduces non-determinism.
- Bugs (races, deadlocks) are hard to reproduce and debug.

### Annotated Code Example

```cpp
#include <iostream>
#include <thread>
#include <vector>

void task(int id) {
    std::cout << "Task " << id << " running\n";
}

int main() {
    std::vector<std::thread> threads;
    for (int i = 0; i < 4; ++i)
        threads.emplace_back(task, i);

    for (auto& t : threads) t.join();
    std::cout << "All tasks done\n";
}
```

**Expected Output** (order may vary)
```
Task 0 running
Task 2 running
Task 1 running
Task 3 running
All tasks done
```

**Why this output?**  
Four threads run concurrently; the order of their output is non-deterministic because the OS schedules them independently.

### Real-World Cases

- **GUI applications** – background threads keep the UI responsive.
- **Servers** – handling multiple client connections concurrently.
- **Data pipelines** – overlapping I/O and computation.

---

## 2. Parallelism

### Definitions

**Core Definition**
Parallelism is the simultaneous execution of multiple tasks on multiple processing units (cores, processors, or machines).

**Technical Definition**
Parallelism is a subset of concurrency in which tasks actually execute at the same time. C++ supports parallelism through multiple threads, parallel algorithms (`std::execution::par`), and SIMD (`std::simd` in C++26). Amdahl's law and Gustafson's law describe the limits of parallel speedup.

**Beginner-Friendly Explanation**
Parallelism is *actual* simultaneity: two or more things happening at the exact same moment on different cores. Concurrency is the broader idea; parallelism is the physical realisation.

### Purposes

- To reduce execution time by exploiting multiple cores.
- To process large data sets faster.
- To improve throughput in servers.
- To accelerate numerical and scientific computations.

### Syntax Rules and Structure

**Parallelism in C++**

| Mechanism | Description |
|-----------|-------------|
| `std::thread` | Manual parallelism. |
| `std::execution::par` | Parallel standard algorithms. |
| `std::execution` (senders/receivers) | Structured parallelism (C++26). |
| `std::simd` (C++26) | Data parallelism. |

**Syntax Rules**
- Parallel algorithms require thread-safe callables.
- Amdahl's law limits speedup based on the serial fraction.
- Data must be partitioned to avoid contention.

**Constraints and Limitations**
- Parallelism requires multiple cores.
- Overhead of thread creation and synchronisation can outweigh gains for small tasks.

### Annotated Code Example

```cpp
#include <algorithm>
#include <execution>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v(1'000'000, 1);
    std::for_each(std::execution::par, v.begin(), v.end(),
                  [](int& x) { x *= 2; });
    std::cout << v[0] << '\n';            // 2
}
```

**Expected Output**
```
2
```

**Why this output?**  
The parallel `for_each` doubles every element; the result is the same as sequential execution, but the work is distributed across cores.

### Real-World Cases

- **Scientific computing** – matrix operations on many cores.
- **Image processing** – parallel pixel operations.
- **Web servers** – handling many requests in parallel.

---

## 3. Threads

### Definitions

**Core Definition**
A thread is an independent sequence of execution within a process, sharing the process's memory and resources with other threads.

**Technical Definition**
In C++, a thread is represented by `std::thread` (since C++11). Threads within a process share the same address space, file descriptors, and other resources, but each has its own stack, registers, and thread-local storage. Threads are the primary unit of concurrency in C++.

**Beginner-Friendly Explanation**
A thread is a “worker” inside your program. Multiple threads can do different things at the same time, and they all share the same memory—which is powerful but dangerous if not synchronized.

### Purposes

- To run code concurrently within a process.
- To exploit multiple cores (parallelism).
- To overlap I/O with computation.
- To structure programs around independent tasks.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <thread>

std::thread t(function, args...);
t.join();     // wait for completion
// or
t.detach();   // run independently
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `std::thread` | Thread object. |
| `join()` | Waits for the thread to finish. |
| `detach()` | Separates the thread from the thread object. |
| `std::this_thread::get_id()` | Returns the current thread's ID. |

**Syntax Rules**
- Every `std::thread` must be joined or detached before destruction; otherwise `std::terminate` is called.
- Threads share memory; synchronisation is required for shared mutable data.
- Thread functions can be free functions, lambdas, function objects, or member functions.

**Constraints and Limitations**
- Thread creation has overhead.
- Too many threads can degrade performance.
- Data races are undefined behaviour.

### Annotated Code Example

```cpp
#include <iostream>
#include <thread>
#include <vector>

std::mutex mtx;
int counter = 0;

void increment(int n) {
    for (int i = 0; i < n; ++i) {
        std::lock_guard<std::mutex> lock(mtx);
        ++counter;
    }
}

int main() {
    std::thread t1(increment, 1000);
    std::thread t2(increment, 1000);
    t1.join();
    t2.join();
    std::cout << counter << '\n';         // 2000
}
```

**Expected Output**
```
2000
```

**Why this output?**  
The mutex ensures that only one thread increments `counter` at a time, preventing a data race.

### Real-World Cases

- **Servers** – one thread per connection (or a thread pool).
- **GUI** – background threads for long-running tasks.
- **Games** – separate threads for rendering, physics, and audio.

---

## 4. Processes

### Definitions

**Core Definition**
A process is an independent instance of a running program, with its own memory space, file descriptors, and resources.

**Technical Definition**
Processes are created and managed by the operating system, not by the C++ standard library (C++ provides no standard process API). Unlike threads, processes do not share memory by default; inter-process communication (IPC) requires pipes, sockets, shared memory, or other mechanisms. C++ programs can create processes via platform-specific APIs (`fork`/`exec` on POSIX, `CreateProcess` on Windows) or third-party libraries.

**Beginner-Friendly Explanation**
A process is a separate program with its own memory. Threads share memory; processes do not. Processes are isolated, which makes them safer but harder to coordinate.

### Purposes

- To provide isolation between tasks.
- To run separate programs.
- To sandbox untrusted code.
- To exploit multiple cores across program boundaries.

### Syntax Rules and Structure

**Process Creation (POSIX example)**
```cpp
#include <unistd.h>
#include <sys/wait.h>

pid_t pid = fork();
if (pid == 0) {
    execl("/bin/ls", "ls", nullptr);
} else {
    waitpid(pid, nullptr, 0);
}
```

**Component Breakdown**

| Mechanism | Platform |
|-----------|----------|
| `fork`/`exec` | POSIX |
| `CreateProcess` | Windows |
| `std::system` | Cross-platform (simple) |

**Syntax Rules**
- C++ has no standard process API.
- Processes do not share memory by default.
- IPC requires explicit mechanisms.

**Constraints and Limitations**
- Process creation is more expensive than thread creation.
- IPC adds complexity.

### Annotated Code Example

```cpp
#include <cstdlib>
#include <iostream>

int main() {
    int result = std::system("echo hello from a process");
    std::cout << "System returned: " << result << '\n';
}
```

**Expected Output** (POSIX-like)
```
hello from a process
System returned: 0
```

**Why this output?**  
`std::system` launches a separate process (`echo`), which runs independently and prints its output.

### Real-World Cases

- **Shells and build systems** – spawning compiler processes.
- **Sandboxing** – running untrusted code in a separate process.
- **Distributed systems** – separate processes communicating over sockets.

---

## 5. Thread Lifecycle

### Definitions

**Core Definition**
The thread lifecycle is the sequence of states a thread goes through: creation, running (or ready), blocked, and terminated.

**Technical Definition**
A thread starts when `std::thread` is constructed with a callable. It may be *running*, *ready* (runnable but waiting for CPU), or *blocked* (waiting for I/O, a mutex, a condition variable, etc.). It terminates when its function returns or when it is cancelled (not standard in C++). After termination, the thread must be joined or detached.

**Beginner-Friendly Explanation**
A thread's life: it is born (created), runs (or waits its turn), may pause (blocked), and eventually finishes. You must wait for it (`join`) or let it go (`detach`).

### Purposes

- To understand thread states and transitions.
- To avoid leaks (unjoined threads).
- To reason about thread termination.

### Syntax Rules and Structure

**Thread States**

| State | Description |
|-------|-------------|
| Created | `std::thread` constructed. |
| Ready | Runnable, waiting for CPU. |
| Running | Executing on a core. |
| Blocked | Waiting for a resource. |
| Terminated | Function returned. |
| Joined/Detached | Thread object released. |

**Syntax Rules**
- `join()` blocks until the thread terminates.
- `detach()` separates the thread; it runs independently.
- Destroying a joinable `std::thread` calls `std::terminate`.

**Constraints and Limitations**
- C++ does not provide thread cancellation (use `std::stop_token` with `std::jthread` in C++20).
- Threads cannot be restarted after termination.

### Annotated Code Example

```cpp
#include <iostream>
#include <thread>
#include <chrono>

void worker() {
    std::cout << "Worker started\n";
    std::this_thread::sleep_for(std::chrono::milliseconds(100));
    std::cout << "Worker finished\n";
}

int main() {
    std::thread t(worker);
    std::cout << "Main waiting\n";
    t.join();
    std::cout << "Main done\n";
}
```

**Expected Output**
```
Worker started
Main waiting
Worker finished
Main done
```

**Why this output?**  
The worker thread starts, sleeps, and finishes; `main` waits via `join()` and then prints the final message.

### Real-World Cases

- **Thread pools** – managing a set of worker threads.
- **Graceful shutdown** – joining threads on program exit.
- **`std::jthread`** – automatic join and cooperative cancellation.

---

## 6. Thread Scheduling

### Definitions

**Core Definition**
Thread scheduling is the operating system's policy for deciding which runnable thread executes on which CPU core and for how long.

**Technical Definition**
Thread scheduling is an OS responsibility, not a C++ one. C++ provides no standard scheduling API (though platform-specific APIs exist). Common policies include round-robin, priority-based, and fair-share scheduling. The OS may preempt running threads to give others a turn, and may migrate threads between cores.

**Beginner-Friendly Explanation**
The OS is the “traffic controller” for threads. It decides which thread runs when, for how long, and on which core. C++ doesn't control this—it just creates threads and lets the OS schedule them.

### Purposes

- To share CPU time fairly among threads.
- To meet real-time deadlines (with priority scheduling).
- To balance load across cores.
- To prevent starvation.

### Syntax Rules and Structure

**Scheduling in C++**

| Mechanism | Description |
|-----------|-------------|
| `std::thread` | OS-scheduled by default. |
| `std::this_thread::yield()` | Hint to yield the CPU. |
| `std::this_thread::sleep_for()` | Sleep for a duration. |
| Platform APIs | `sched_setscheduler`, `SetThreadPriority`. |

**Constraints and Limitations**
- C++ provides no standard scheduling control.
- Scheduling is non-deterministic from the program's perspective.

### Annotated Code Example

```cpp
#include <iostream>
#include <thread>
#include <chrono>

void worker(int id) {
    for (int i = 0; i < 3; ++i) {
        std::cout << "Thread " << id << ": " << i << '\n';
        std::this_thread::sleep_for(std::chrono::milliseconds(10));
    }
}

int main() {
    std::thread t1(worker, 1);
    std::thread t2(worker, 2);
    t1.join();
    t2.join();
}
```

**Expected Output** (interleaving may vary)
```
Thread 1: 0
Thread 2: 0
Thread 1: 1
Thread 2: 1
Thread 1: 2
Thread 2: 2
```

**Why this output?**  
The OS schedules the two threads; the interleaving depends on the scheduler. `sleep_for` yields the CPU to the other thread.

### Real-World Cases

- **Real-time systems** – priority scheduling for deadlines.
- **Servers** – fair scheduling for many connections.
- **Games** – dedicating threads to rendering or physics.

---

## 7. Context Switching

### Definitions

**Core Definition**
Context switching is the process by which the operating system saves the state of one thread (or process) and restores the state of another so that it can resume execution.

**Technical Definition**
A context switch saves the current thread's registers, program counter, and stack pointer, and loads the saved state of the next thread. It may also involve flushing caches and TLB entries (especially across processes). Context switches are triggered by time slices expiring, blocking operations, or explicit yields. They have a measurable cost (typically microseconds).

**Beginner-Friendly Explanation**
Context switching is the “pause and resume” mechanism: the OS pauses one thread, saves its place, and resumes another. It is fast but not free—too many switches slow down the program.

### Purposes

- To enable multitasking on limited cores.
- To allow threads to block without stalling the CPU.
- To share CPU time fairly.

### Syntax Rules and Structure

**Context Switch Triggers**

| Trigger | Description |
|---------|-------------|
| Time slice expiry | OS preempts the thread. |
| Blocking call | Thread waits for I/O or a lock. |
| Explicit yield | `std::this_thread::yield()`. |
| Sleep | `std::this_thread::sleep_for()`. |

**Constraints and Limitations**
- Context switches have overhead (registers, caches).
- Excessive switching (thrashing) degrades performance.
- C++ cannot control context switching directly.

### Annotated Code Example

```cpp
#include <iostream>
#include <thread>
#include <chrono>

void spinner() {
    for (int i = 0; i < 5; ++i) {
        std::this_thread::yield();       // hint to the scheduler
    }
}

int main() {
    std::thread t(spinner);
    t.join();
    std::cout << "Done\n";
}
```

**Expected Output**
```
Done
```

**Why this output?**  
`yield()` hints that the thread is willing to give up the CPU; the OS decides whether to perform a context switch.

### Real-World Cases

- **High-frequency trading** – minimising context switches for latency.
- **Real-time systems** – controlling scheduling to meet deadlines.
- **Servers** – balancing throughput and latency.

---

## 8. Concurrency Best Practices

### Definitions

**Core Definition**
Concurrency best practices are guidelines for writing correct, efficient, and maintainable concurrent C++ programs.

**Technical Definition**
These practices are codified in the C++ Core Guidelines (CP.1–CP.100+) and community consensus: prefer higher-level abstractions (`std::async`, `std::jthread`, senders/receivers) over raw threads (CP.4); use `std::thread` only when necessary; avoid data races (CP.2); share data rather than communicate (or vice versa); use `std::atomic` and locks correctly; minimise shared mutable state; and use tools (ThreadSanitizer, Helgrind) to validate (CP.9).

**Beginner-Friendly Explanation**
Best practices for concurrency are simple: prefer high-level tools over raw threads, never share mutable data without synchronisation, and always test with race-detection tools.

### Key Guidelines

| Guideline | Summary |
|-----------|---------|
| **CP.1** | Assume your code will run as part of a multi-threaded program. |
| **CP.2** | Avoid data races. |
| **CP.3** | Minimise explicit sharing of writable data. |
| **CP.4** | Think in terms of tasks, not threads. |
| **CP.8** | Don't use `volatile` for synchronisation. |
| **CP.9** | Use tools to validate concurrent code. |
| **CP.20** | Use RAII, never plain `lock()`/`unlock()`. |
| **CP.21** | Use `std::lock()` or `std::scoped_lock` for multiple mutexes. |
| **CP.100** | Don't use lock-free programming unless you absolutely have to. |
| **Prefer `std::jthread`** | Automatic join and cooperative cancellation (C++20). |
| **Prefer `std::async`** | For simple task-based concurrency. |

### Annotated Code Example

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <vector>

std::mutex mtx;
int counter = 0;

void increment(int n) {
    for (int i = 0; i < n; ++i) {
        std::lock_guard<std::mutex> lock(mtx);   // RAII lock
        ++counter;
    }
}

int main() {
    std::thread t1(increment, 1000);
    std::thread t2(increment, 1000);
    t1.join();
    t2.join();
    std::cout << counter << '\n';                // 2000
}
```

**Expected Output**
```
2000
```

**Why this output?**  
The `lock_guard` ensures only one thread increments `counter` at a time, preventing a data race.

### Real-World Cases

- **Code reviews** – checking for data races and missing locks.
- **Static analysis** – ThreadSanitizer and Helgrind.
- **Performance tuning** – reducing lock contention and context switches.

---

## References

- Concurrency support library — cppreference – https://en.cppreference.com/w/cpp/thread
- `std::thread` — cppreference – https://en.cppreference.com/w/cpp/thread/thread
- `std::mutex` — cppreference – https://en.cppreference.com/w/cpp/thread/mutex
- `std::atomic` — cppreference – https://en.cppreference.com/w/cpp/atomic/atomic
- C++ Core Guidelines — Concurrency – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-concurrency
- C++ Core Guidelines — CP.4: Think in terms of tasks, not threads – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-task
- C++ Core Guidelines — CP.9: Whenever feasible use tools to validate your concurrent code – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-tools
- `std::execution` policies — cppreference – https://en.cppreference.com/w/cpp/algorithm/execution_policy_tag
- `std::jthread` — cppreference – https://en.cppreference.com/w/cpp/thread/jthread
- Thread (computing) — Wikipedia – https://en.wikipedia.org/wiki/Thread_(computing)
- Process (computing) — Wikipedia – https://en.wikipedia.org/wiki/Process_(computing)
- Context switch — Wikipedia – https://en.wikipedia.org/wiki/Context_switch

---

This cheat sheet provides a comprehensive, structured reference for C++ concurrency fundamentals, covering concurrency, parallelism, threads, processes, thread lifecycle, thread scheduling, context switching, and best practices, with annotated code examples, expected outputs, real-world cases, and authoritative references.