# C++ Threads — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A thread is an independent sequence of execution within a process. In C++, threads are represented by `std::thread` (C++11) and `std::jthread` (C++20), which allow a program to run multiple functions concurrently while sharing the same address space.

**Technical Definition**
`std::thread` is a class that represents a single thread of execution. A thread begins execution when the `std::thread` object is constructed with a callable (function, lambda, function object, or member function) and its arguments. The thread runs concurrently with the constructing thread. `std::thread` is move-only, must be either joined or detached before destruction, and provides `get_id()` to obtain the thread's unique identifier. C++20 adds `std::jthread`, which automatically joins on destruction and carries a `std::stop_source` for cooperative cancellation.

**Beginner-Friendly Explanation**
A thread is a “worker” inside your program. `std::thread` lets you start a new worker that runs a function at the same time as the main program. You must either wait for it to finish (`join`) or let it run on its own (`detach`). `std::jthread` is a smarter worker that cleans itself up when it's done.

### Key Characteristics

- **Concurrent execution** – threads run in parallel or interleaved.
- **Shared address space** – threads share the process's memory, file descriptors, and other resources.
- **Move-only** – `std::thread` cannot be copied, only moved.
- **Joinable or detached** – every thread must be joined or detached before the `std::thread` object is destroyed.
- **Non-deterministic scheduling** – the OS decides when and where threads run.
- **`std::jthread`** – automatic join, cooperative cancellation via `std::stop_token`.
- **Thread-local storage** – `thread_local` variables are unique to each thread.

### Prerequisites

- Basic C++ syntax (functions, lambdas, move semantics).
- Understanding of the memory model and data races.
- Familiarity with `std::mutex` and `std::atomic` (for synchronisation).
- Knowledge of exception safety and RAII.

### Related Programming Areas

- **Concurrency fundamentals** – threads, processes, scheduling.
- **Memory model** – happens-before, data races.
- **Synchronisation** – mutexes, condition variables, atomics.
- **Async programming** – `std::async`, `std::future`, coroutines.
- **Thread pools** – managing many threads efficiently.

### Core Concepts / Features

1. `std::thread`
2. Thread creation
3. Thread join
4. Thread detach
5. `std::jthread` (C++20)
6. Thread ID
7. `std::this_thread`
8. `std::this_thread::sleep_for`
9. `std::this_thread::sleep_until`
10. `std::this_thread::yield`
11. Thread best practices

---

## 1. `std::thread`

### Definitions

**Core Definition**
`std::thread` is a class that represents a single thread of execution in a C++ program.

**Technical Definition**
`std::thread` (since C++11, in `<thread>`) manages a native OS thread. A thread object is *joinable* if it represents an active thread. Every joinable thread must be joined or detached before the `std::thread` object is destroyed; otherwise, `std::terminate` is called. The class is move-only and provides `join()`, `detach()`, `joinable()`, `get_id()`, `native_handle()`, and `swap()`.

**Beginner-Friendly Explanation**
`std::thread` is the standard C++ way to start a new thread. You give it a function to run, and it runs that function concurrently.

### Purposes

- To run code concurrently with the main thread.
- To exploit multiple cores (parallelism).
- To overlap I/O with computation.
- To structure programs around independent tasks.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <thread>

std::thread t(function, args...);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `function` | A callable (function, lambda, function object, member function). |
| `args...` | Arguments forwarded to the callable. |
| `join()` | Waits for the thread to finish. |
| `detach()` | Separates the thread from the thread object. |
| `joinable()` | Returns `true` if the thread is joinable. |
| `get_id()` | Returns the thread's ID. |

**Syntax Rules**
- Arguments are copied or moved into the thread's storage; use `std::ref` to pass by reference.
- A `std::thread` object must be joined or detached before destruction.
- Exceptions thrown in a thread are not propagated to the parent; they call `std::terminate` unless caught within the thread.

**Constraints and Limitations**
- Thread creation has overhead; avoid creating threads for trivial tasks.
- Too many threads can degrade performance.
- Data races are undefined behaviour.

### Annotated Code Example

```cpp
#include <iostream>
#include <thread>

void hello(int id) {
    std::cout << "Hello from thread " << id << '\n';
}

int main() {
    std::thread t(hello, 42);
    t.join();                              // wait for completion
    std::cout << "Main done\n";
}
```

**Expected Output**
```
Hello from thread 42
Main done
```

**Why this output?**  
`std::thread t(hello, 42)` starts a new thread running `hello(42)`; `join()` waits for it to finish before printing the final message.

### Real-World Cases

- **Servers** – one thread per connection (or a thread pool).
- **GUI** – background threads for long-running tasks.
- **Games** – separate threads for rendering, physics, and audio.

---

## 2. Thread Creation

### Definitions

**Core Definition**
Thread creation is the act of starting a new thread of execution by constructing a `std::thread` object with a callable and its arguments.

**Technical Definition**
A `std::thread` is constructed with a callable and (optionally) arguments. The new thread begins execution as if it had been invoked via `std::invoke` with *decayed copies* of the arguments (unless `std::ref`/`std::cref` is used). The callable can be a free function, lambda, function object, member function pointer, or any type meeting the *Callable* requirements.

**Beginner-Friendly Explanation**
Creating a thread means “starting a new worker.” You tell it what function to run and what arguments to give it.

### Purposes

- To start concurrent work.
- To run independent tasks in parallel.
- To overlap computation and I/O.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::thread t(function, args...);
std::thread t([]{ /* lambda */ });
std::thread t(&Class::method, object, args...);
```

**Component Breakdown**

| Form | Example |
|------|---------|
| Free function | `std::thread t(func, arg);` |
| Lambda | `std::thread t([]{ ... });` |
| Member function | `std::thread t(&C::f, obj, arg);` |
| Function object | `std::thread t(Functor{}, arg);` |

**Syntax Rules**
- Arguments are copied or moved; use `std::ref`/`std::cref` for references.
- The callable is copied/moved into the thread's storage.
- The thread starts executing immediately (subject to OS scheduling).

**Constraints and Limitations**
- The callable must be copy-constructible or move-constructible.
- Arguments must be copy-constructible or move-constructible unless wrapped in `std::ref`/`std::cref`.

### Annotated Code Example

```cpp
#include <iostream>
#include <thread>
#include <string>

void greet(const std::string& name) {
    std::cout << "Hello, " << name << '\n';
}

int main() {
    std::thread t1(greet, "Alice");       // copies the string
    std::thread t2([]{ std::cout << "Lambda thread\n"; });
    t1.join();
    t2.join();
}
```

**Expected Output** (order may vary)
```
Hello, Alice
Lambda thread
```

**Why this output?**  
`t1` runs `greet("Alice")`; `t2` runs a lambda. The output order depends on scheduling.

### Real-World Cases

- **Worker threads** – background computation.
- **Parallel algorithms** – dividing work across threads.
- **Event handling** – separate threads for different event sources.

---

## 3. Thread Join

### Definitions

**Core Definition**
`join()` blocks the calling thread until the thread represented by the `std::thread` object finishes execution.

**Technical Definition**
`join()` waits for the thread to complete and then releases the thread object (it is no longer joinable). After `join()`, the thread object represents no thread. Joining a non-joinable thread throws `std::system_error`. Joining the current thread (`t.join()` where `t` represents the calling thread) is undefined behaviour.

**Beginner-Friendly Explanation**
`join` means “wait for this worker to finish.” It blocks the caller until the thread is done.

### Purposes

- To wait for a thread to complete.
- To synchronise with the thread's results.
- To ensure resources are released before the program exits.

### Syntax Rules and Structure

**General Syntax**
```cpp
t.join();
```

**Component Breakdown**

| Aspect | Description |
|--------|-------------|
| Effect | Blocks until the thread finishes. |
| Postcondition | `t.joinable()` returns `false`. |
| Exception | Throws `std::system_error` if not joinable. |

**Syntax Rules**
- `join()` must be called at most once per thread.
- After `join()`, the thread object is no longer joinable.
- Joining from the thread itself is undefined behaviour.

**Constraints and Limitations**
- `join()` blocks the caller; use it when you need to wait.

### Annotated Code Example

```cpp
#include <iostream>
#include <thread>
#include <chrono>

void work() {
    std::this_thread::sleep_for(std::chrono::milliseconds(100));
    std::cout << "Work done\n";
}

int main() {
    std::thread t(work);
    std::cout << "Waiting...\n";
    t.join();                              // wait for completion
    std::cout << "Joined\n";
}
```

**Expected Output**
```
Waiting...
Work done
Joined
```

**Why this output?**  
`join()` blocks `main` until `work` finishes, so “Work done” appears before “Joined”.

### Real-World Cases

- **Program shutdown** – joining worker threads before exiting.
- **Result collection** – waiting for threads to produce results.
- **Thread pools** – joining pool threads on destruction.

---

## 4. Thread Detach

### Definitions

**Core Definition**
`detach()` separates the thread from the `std::thread` object, allowing the thread to run independently in the background.

**Technical Definition**
`detach()` makes the `std::thread` object non-joinable and allows the thread to continue running independently. The thread's resources are released automatically when it finishes. After `detach()`, there is no way to wait for the thread or obtain its result. Detaching a non-joinable thread throws `std::system_error`.

**Beginner-Friendly Explanation**
`detach` means “let this worker run on its own; I won't wait for it.” The thread continues in the background and cleans up after itself when done.

### Purposes

- To run background tasks that don't need to be waited for.
- To avoid blocking the main thread.
- To fire-and-forget operations.

### Syntax Rules and Structure

**General Syntax**
```cpp
t.detach();
```

**Component Breakdown**

| Aspect | Description |
|--------|-------------|
| Effect | Thread runs independently. |
| Postcondition | `t.joinable()` returns `false`. |
| Exception | Throws `std::system_error` if not joinable. |

**Syntax Rules**
- The thread's resources are released when it finishes.
- You cannot `join()` a detached thread.
- Detached threads may outlive the main thread; the program may terminate before they finish.

**Constraints and Limitations**
- Detached threads cannot be waited for.
- If the program exits, detached threads are terminated abruptly.

### Annotated Code Example

```cpp
#include <iostream>
#include <thread>
#include <chrono>

void background() {
    std::this_thread::sleep_for(std::chrono::seconds(1));
    std::cout << "Background task done\n";
}

int main() {
    std::thread t(background);
    t.detach();                            // run independently
    std::cout << "Main continues\n";
    std::this_thread::sleep_for(std::chrono::seconds(2));
    std::cout << "Main done\n";
}
```

**Expected Output**
```
Main continues
Background task done
Main done
```

**Why this output?**  
The detached thread runs independently; `main` continues immediately, and the background task finishes later (the program waits long enough to see it).

### Real-World Cases

- **Background logging** – detached logging thread.
- **Fire-and-forget tasks** – notifications, analytics.
- **Daemon threads** – background maintenance.

---

## 5. `std::jthread` (C++20)

### Definitions

**Core Definition**
`std::jthread` is a thread class that automatically joins on destruction and supports cooperative cancellation via `std::stop_token`.

**Technical Definition**
`std::jthread` (C++20) behaves like `std::thread` but provides two key improvements: its destructor automatically calls `join()` (unless the thread has been detached), and it carries an internal `std::stop_source` whose `std::stop_token` can be used for cooperative cancellation. The thread's callable can optionally accept a `std::stop_token` as its first parameter.

**Beginner-Friendly Explanation**
`std::jthread` is a safer, smarter thread: it joins automatically when destroyed, and it can be asked to stop gracefully.

### Purposes

- To eliminate the risk of forgetting to join a thread.
- To provide built-in cooperative cancellation.
- To simplify thread lifetime management.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <thread>
#include <stop_token>

std::jthread t([](std::stop_token st) {
    while (!st.stop_requested()) {
        // work
    }
});
// t joins automatically at scope exit
```

**Component Breakdown**

| Feature | Description |
|---------|-------------|
| Auto-join | Destructor calls `join()`. |
| `stop_token` | Cooperative cancellation token. |
| `request_stop()` | Requests cancellation. |

**Syntax Rules**
- The callable may take a `std::stop_token` as its first argument.
- `request_stop()` requests cancellation; the thread must poll the token.

**Constraints and Limitations**
- Requires C++20.
- Cooperative cancellation requires the thread to check the token.

### Annotated Code Example

```cpp
#include <iostream>
#include <thread>
#include <stop_token>
#include <chrono>

int main() {
    std::jthread t([](std::stop_token st) {
        int i = 0;
        while (!st.stop_requested()) {
            std::cout << i++ << ' ';
            std::this_thread::sleep_for(std::chrono::milliseconds(100));
        }
    });
    std::this_thread::sleep_for(std::chrono::milliseconds(350));
    t.request_stop();
} // t joins automatically
```

**Expected Output** (approximately)
```
0 1 2 3
```

**Why this output?**  
The thread prints numbers until `request_stop()` is called; the `jthread` then joins automatically at scope exit.

### Real-World Cases

- **Background workers** – safe lifetime management.
- **Server loops** – graceful shutdown on request.
- **Game engines** – cooperative task cancellation.

---

## 6. Thread ID

### Definitions

**Core Definition**
A thread ID is a unique identifier associated with each thread of execution.

**Technical Definition**
`std::thread::id` (since C++11) is a lightweight, copyable type that uniquely identifies a thread. It is obtained via `std::thread::get_id()` or `std::this_thread::get_id()`. IDs can be compared, hashed, and printed. The ID of a non-joinable thread is the default-constructed `std::thread::id{}`.

**Beginner-Friendly Explanation**
A thread ID is like a name tag for a thread. It lets you tell threads apart and check which thread is running.

### Purposes

- To identify threads in logging and debugging.
- To compare thread identities.
- To store thread IDs in maps or sets.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::thread::id id = std::this_thread::get_id();
std::cout << id << '\n';
```

**Component Breakdown**

| Operation | Description |
|-----------|-------------|
| `get_id()` | Returns the thread's ID. |
| `std::this_thread::get_id()` | Returns the current thread's ID. |
| Comparison | `==`, `!=`, `<` (for ordered containers) |
| Hashing | `std::hash<std::thread::id>` |

**Syntax Rules**
- IDs are unique for each running thread.
- IDs may be reused after a thread terminates.
- The default-constructed ID is not associated with any thread.

**Constraints and Limitations**
- The string representation of a thread ID is implementation-defined.

### Annotated Code Example

```cpp
#include <iostream>
#include <thread>
#include <vector>
#include <mutex>

std::mutex mtx;

void worker(int n) {
    std::lock_guard<std::mutex> lock(mtx);
    std::cout << "Thread " << n << " id: "
              << std::this_thread::get_id() << '\n';
}

int main() {
    std::vector<std::thread> threads;
    for (int i = 0; i < 3; ++i)
        threads.emplace_back(worker, i);
    for (auto& t : threads) t.join();
}
```

**Expected Output** (IDs vary)
```
Thread 0 id: 12345
Thread 1 id: 12346
Thread 2 id: 12347
```

**Why this output?**  
Each thread prints its own ID, obtained via `std::this_thread::get_id()`.

### Real-World Cases

- **Logging** – tagging log entries with thread IDs.
- **Debugging** – identifying which thread produced output.
- **Thread-local registries** – mapping IDs to per-thread data.

---

## 7. `std::this_thread`

### Definitions

**Core Definition**
`std::this_thread` is a namespace containing functions that operate on the calling thread.

**Technical Definition**
`std::this_thread` (since C++11) provides `get_id()`, `yield()`, `sleep_for()`, and `sleep_until()`. These functions affect the current thread only.

**Beginner-Friendly Explanation**
`std::this_thread` is “the current thread's toolbox.” It lets the running thread identify itself, yield the CPU, or sleep.

### Purposes

- To identify the current thread.
- To yield the CPU.
- To sleep for a duration or until a time point.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::this_thread::get_id();
std::this_thread::yield();
std::this_thread::sleep_for(duration);
std::this_thread::sleep_until(time_point);
```

**Component Breakdown**

| Function | Description |
|----------|-------------|
| `get_id()` | Returns the current thread's ID. |
| `yield()` | Hints that the thread is willing to yield the CPU. |
| `sleep_for(duration)` | Sleeps for a duration. |
| `sleep_until(time_point)` | Sleeps until a time point. |

**Constraints and Limitations**
- Sleep durations are minimums, not guarantees.
- `yield()` is a hint; the OS may ignore it.

### Annotated Code Example

```cpp
#include <iostream>
#include <thread>
#include <chrono>

int main() {
    std::cout << "Current thread id: "
              << std::this_thread::get_id() << '\n';
    std::this_thread::sleep_for(std::chrono::milliseconds(100));
    std::this_thread::yield();
    std::cout << "Done\n";
}
```

**Expected Output** (ID varies)
```
Current thread id: 12345
Done
```

**Why this output?**  
The current thread prints its ID, sleeps for 100 ms, yields, and prints “Done”.

### Real-World Cases

- **Rate limiting** – sleeping between operations.
- **Polling loops** – yielding or sleeping between checks.
- **Debugging** – identifying the current thread.

---

## 8. `std::this_thread::sleep_for`

### Definitions

**Core Definition**
`std::this_thread::sleep_for` blocks the current thread for at least the specified duration.

**Technical Definition**
`sleep_for(duration)` (since C++11) blocks the calling thread for at least `duration`, subject to OS scheduling and clock resolution. The actual sleep may be longer.

**Beginner-Friendly Explanation**
`sleep_for` means “pause this thread for this long.”

### Purposes

- To delay execution.
- To rate-limit operations.
- To implement timeouts and polling.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::this_thread::sleep_for(std::chrono::milliseconds(100));
```

**Constraints and Limitations**
- The actual sleep may be longer than requested.
- Not suitable for precise timing.

### Annotated Code Example

```cpp
#include <iostream>
#include <thread>
#include <chrono>

int main() {
    std::cout << "Start\n";
    std::this_thread::sleep_for(std::chrono::milliseconds(200));
    std::cout << "After 200ms\n";
}
```

**Expected Output**
```
Start
After 200ms
```

**Why this output?**  
The thread sleeps for at least 200 ms between the two prints.

### Real-World Cases

- **Rate limiting** – pausing between API calls.
- **Simulation** – advancing time in steps.
- **Retry loops** – waiting before retrying.

---

## 9. `std::this_thread::sleep_until`

### Definitions

**Core Definition**
`std::this_thread::sleep_until` blocks the current thread until a specified time point.

**Technical Definition**
`sleep_until(time_point)` (since C++11) blocks the calling thread until the specified time point is reached. It is useful for periodic tasks with absolute deadlines.

**Beginner-Friendly Explanation**
`sleep_until` means “pause until this specific moment in time.”

### Purposes

- To synchronise with absolute deadlines.
- To implement periodic tasks.
- To avoid drift in repeated delays.

### Syntax Rules and Structure

**General Syntax**
```cpp
auto deadline = std::chrono::steady_clock::now() + std::chrono::seconds(1);
std::this_thread::sleep_until(deadline);
```

**Constraints and Limitations**
- Uses the clock associated with the time point.
- The actual wake-up may be later than requested.

### Annotated Code Example

```cpp
#include <iostream>
#include <thread>
#include <chrono>

int main() {
    auto start = std::chrono::steady_clock::now();
    auto deadline = start + std::chrono::milliseconds(300);
    std::cout << "Start\n";
    std::this_thread::sleep_until(deadline);
    std::cout << "Deadline reached\n";
}
```

**Expected Output**
```
Start
Deadline reached
```

**Why this output?**  
The thread sleeps until the absolute deadline is reached.

### Real-World Cases

- **Periodic tasks** – running at fixed intervals.
- **Deadlines** – waiting until a specific time.
- **Schedulers** – precise timing without drift.

---

## 10. `std::this_thread::yield`

### Definitions

**Core Definition**
`std::this_thread::yield` hints to the scheduler that the current thread is willing to give up the CPU.

**Technical Definition**
`yield()` (since C++11) is a hint to the OS scheduler to reschedule, potentially allowing other threads to run. It does not guarantee a context switch; the OS may ignore the hint. It is used in spin-wait loops and cooperative multitasking.

**Beginner-Friendly Explanation**
`yield` means “I'm willing to let someone else run now.” It is a polite hint to the scheduler.

### Purposes

- To reduce CPU spinning in busy-wait loops.
- To improve fairness between threads.
- To cooperate with other threads.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::this_thread::yield();
```

**Constraints and Limitations**
- It is a hint, not a guarantee.
- Not a substitute for proper synchronisation.

### Annotated Code Example

```cpp
#include <iostream>
#include <thread>
#include <atomic>

std::atomic<bool> ready{false};

void producer() {
    ready.store(true);
}

void consumer() {
    while (!ready.load()) {
        std::this_thread::yield();         // yield while waiting
    }
    std::cout << "Ready\n";
}

int main() {
    std::thread t1(producer);
    std::thread t2(consumer);
    t1.join(); t2.join();
}
```

**Expected Output**
```
Ready
```

**Why this output?**  
The consumer yields the CPU while waiting for `ready`, allowing the producer to run.

### Real-World Cases

- **Spin-wait loops** – yielding to reduce CPU usage.
- **Cooperative multitasking** – yielding between tasks.
- **Debugging** – improving fairness in test scenarios.

---

## 11. Thread Best Practices

### Definitions

**Core Definition**
Thread best practices are guidelines for creating, managing, and synchronising threads correctly, safely, and efficiently in C++.

**Technical Definition**
These practices are codified in the C++ Core Guidelines (CP.1–CP.4, CP.20–CP.21, CP.100+) and community consensus: prefer `std::jthread` over `std::thread` when possible (automatic join and cancellation); use `std::async` for simple task-based concurrency; avoid raw threads for complex coordination; use RAII locks (`std::lock_guard`, `std::scoped_lock`); never use `volatile` for synchronisation; and validate with ThreadSanitizer or Helgrind.

**Beginner-Friendly Explanation**
Best practices for threads are simple: prefer `std::jthread` over `std::thread`, always join or detach, use RAII locks, and test with race-detection tools.

### Key Guidelines

| Guideline | Summary |
|-----------|---------|
| **Prefer `std::jthread`** | Automatic join and cooperative cancellation (C++20). |
| **Always join or detach** | Every `std::thread` must be joined or detached. |
| **Prefer `std::async` for simple tasks** | Higher-level abstraction. |
| **Use RAII locks** | `std::lock_guard`, `std::scoped_lock`. |
| **Never use `volatile` for sync** | Use atomics or mutexes. |
| **Minimise shared state** | Less sharing means fewer races. |
| **Use `std::ref` for references** | Avoid accidental copies. |
| **Catch exceptions inside threads** | Uncaught exceptions call `std::terminate`. |
| **Validate with tools** | ThreadSanitizer, Helgrind. |
| **Avoid detach in libraries** | Detached threads are hard to manage. |

### Annotated Code Example

```cpp
#include <iostream>
#include <thread>
#include <stop_token>
#include <chrono>

int main() {
    std::jthread worker([](std::stop_token st) {
        while (!st.stop_requested()) {
            std::cout << "Working\n";
            std::this_thread::sleep_for(std::chrono::milliseconds(100));
        }
    });
    std::this_thread::sleep_for(std::chrono::milliseconds(250));
    worker.request_stop();
} // joins automatically
```

**Expected Output** (approximately)
```
Working
Working
Working
```

**Why this output?**  
`std::jthread` runs the worker, which prints until `request_stop()` is called; the thread joins automatically at scope exit.

### Real-World Cases

- **Code reviews** – flagging unjoined threads and raw `lock()`/`unlock()`.
- **Static analysis** – ThreadSanitizer and Helgrind.
- **Library design** – exposing safe thread wrappers.

---

## References

- `std::thread` — cppreference – https://en.cppreference.com/w/cpp/thread/thread
- `std::jthread` — cppreference – https://en.cppreference.com/w/cpp/thread/jthread
- `std::this_thread` — cppreference – https://en.cppreference.com/w/cpp/thread/this_thread
- `std::this_thread::sleep_for` — cppreference – https://en.cppreference.com/w/cpp/thread/sleep_for
- `std::this_thread::sleep_until` — cppreference – https://en.cppreference.com/w/cpp/thread/sleep_until
- `std::this_thread::yield` — cppreference – https://en.cppreference.com/w/cpp/thread/yield
- `std::thread::id` — cppreference – https://en.cppreference.com/w/cpp/thread/thread/id
- C++ Core Guidelines — Concurrency – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-concurrency
- C++ Core Guidelines — CP.4: Think in terms of tasks, not threads – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-task
- `std::stop_token` — cppreference – https://en.cppreference.com/w/cpp/thread/stop_token
- `std::async` — cppreference – https://en.cppreference.com/w/cpp/thread/async

---

This cheat sheet provides a comprehensive, structured reference for C++ threads, covering `std::thread`, thread creation, join, detach, `std::jthread`, thread ID, `std::this_thread`, `sleep_for`, `sleep_until`, `yield`, and best practices, with annotated code examples, expected outputs, real-world cases, and authoritative references.