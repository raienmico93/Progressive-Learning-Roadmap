# C++ Futures and Promises — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Futures and promises are a C++ mechanism for asynchronous communication between threads: a *promise* or *async task* produces a value or exception, and a *future* receives it once it becomes available.

**Technical Definition**
The `<future>` header (since C++11) provides `std::future<T>`, `std::shared_future<T>`, `std::promise<T>`, `std::packaged_task<Sig>`, and `std::async`. A future represents a value that will be available at some point in the future. A promise or packaged task stores a value or exception into a shared state, which the associated future retrieves via `get()`. Futures support blocking waits (`wait`, `wait_for`, `wait_until`) and validity checks (`valid`). A future is move-only; `shared_future` is copyable and allows multiple receivers.

**Beginner-Friendly Explanation**
A promise is like a “voucher” for a value that will be ready later. The future is the “claim ticket” that lets you pick up the value when it's ready. You create a promise, hand its future to another thread, and that thread sets the value; when you call `get()` on the future, it waits until the value is available.

### Key Characteristics

- **Asynchronous communication** – transfers values or exceptions between threads.
- **Shared state** – promise and future share a hidden state that holds the result.
- **Move-only futures** – `std::future` is move-only; `std::shared_future` is copyable.
- **Blocking retrieval** – `get()` blocks until the result is available.
- **Exception propagation** – exceptions thrown in the producer are rethrown by `get()`.
- **Single-use vs. multi-use** – `future::get()` consumes the result; `shared_future::get()` returns a copy and can be called multiple times.
- **`std::async`** – the highest-level facility; launches a task and returns a future.

### Prerequisites

- Solid understanding of threads and concurrency.
- Familiarity with mutexes and condition variables.
- Knowledge of move semantics and RAII.
- Understanding of exception handling.

### Related Programming Areas

- **Threads** – futures often coordinate work between threads.
- **Async programming** – `std::async` launches background tasks.
- **Exceptions** – futures propagate exceptions across threads.
- **C++20 coroutines** – coroutines often use future-like types.
- **Senders/receivers (C++26)** – `std::execution` generalises async composition.

### Core Concepts / Features

1. `std::future`
2. `std::promise`
3. `std::packaged_task`
4. `std::async`
5. Future operations (`get`, `wait`, `wait_for`, `wait_until`, `valid`)
6. `std::shared_future`
7. Future best practices

---

## 1. `std::future`

### Definitions

**Core Definition**
`std::future<T>` is a class template that represents a value (or exception) that will be available asynchronously.

**Technical Definition**
`std::future<T>` (since C++11, in `<future>`) provides access to the result of an asynchronous operation. It is associated with a shared state that is set by a `std::promise`, `std::packaged_task`, or `std::async`. The future is move-only; `get()` retrieves the result (blocking if necessary) and invalidates the future. Futures also provide `wait()`, `wait_for()`, `wait_until()`, and `valid()`.

**Beginner-Friendly Explanation**
A `std::future` is a “claim ticket” for a value that will be computed later. You call `get()` to wait for and retrieve the value.

### Purposes

- To receive results from asynchronous tasks.
- To propagate exceptions from producer to consumer.
- To coordinate threads without manual mutexes and condition variables.
- To support higher-level async abstractions.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <future>

std::future<int> f = /* ... */;
int value = f.get();                    // blocks until ready
```

**Component Breakdown**

| Member | Description |
|--------|-------------|
| `get()` | Returns the result (blocking); invalidates the future. |
| `wait()` | Blocks until the result is available. |
| `wait_for(duration)` | Waits for a duration; returns a status. |
| `wait_until(time_point)` | Waits until a deadline; returns a status. |
| `valid()` | Returns `true` if the future is associated with a shared state. |

**Syntax Rules**
- `get()` can be called only once (the future becomes invalid).
- `wait_for`/`wait_until` return `std::future_status` (`ready`, `timeout`, `deferred`).
- Futures are move-only.

**Constraints and Limitations**
- Cannot copy a `std::future`.
- Destroying a valid future from `std::async` may block (implementation-defined).

### Annotated Code Example

```cpp
#include <future>
#include <iostream>
#include <thread>

int compute() {
    std::this_thread::sleep_for(std::chrono::milliseconds(100));
    return 42;
}

int main() {
    std::future<int> f = std::async(std::launch::async, compute);
    std::cout << "Waiting...\n";
    int value = f.get();                   // blocks
    std::cout << "Result: " << value << '\n';
}
```

**Expected Output**
```
Waiting...
Result: 42
```

**Why this output?**  
`std::async` launches `compute` asynchronously; `f.get()` blocks until the result is ready and returns `42`.

### Real-World Cases

- **Background computations** – returning results to the main thread.
- **Asynchronous I/O** – receiving data when it arrives.
- **Parallel algorithms** – collecting results from multiple tasks.

---

## 2. `std::promise`

### Definitions

**Core Definition**
`std::promise<T>` is a class template that allows a thread to store a value or exception into a shared state, which an associated future can retrieve.

**Technical Definition**
`std::promise<T>` (since C++11) provides `set_value`, `set_exception`, `set_value_at_thread_exit`, `set_exception_at_thread_exit`, and `get_future`. The promise and its future share a state; the future is created via `get_future()`. A promise can set its value only once; a second attempt throws `std::future_error`.

**Beginner-Friendly Explanation**
A promise is the “producer side” of a future. You create a promise, get its future, hand the future to a consumer, and later set the value. The consumer's `get()` returns that value.

### Purposes

- To produce a value or exception asynchronously.
- To communicate between threads.
- To implement custom async primitives.
- To signal completion of a task.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::promise<int> p;
std::future<int> f = p.get_future();

// Producer thread:
p.set_value(42);

// Consumer thread:
int value = f.get();
```

**Component Breakdown**

| Member | Description |
|--------|-------------|
| `get_future()` | Returns the associated future. |
| `set_value(v)` | Stores the value. |
| `set_exception(e)` | Stores an exception. |
| `set_value_at_thread_exit(v)` | Stores the value when the thread exits. |

**Syntax Rules**
- `get_future()` can be called only once.
- `set_value`/`set_exception` can be called only once.
- The promise must not be destroyed without setting a value or exception (it sets a `broken_promise` exception).

**Constraints and Limitations**
- Setting a value twice throws `std::future_error`.
- Destroying a promise without setting a value causes the future to throw `std::future_error`.

### Annotated Code Example

```cpp
#include <future>
#include <iostream>
#include <thread>

void producer(std::promise<int> p) {
    std::this_thread::sleep_for(std::chrono::milliseconds(100));
    p.set_value(42);
}

int main() {
    std::promise<int> p;
    std::future<int> f = p.get_future();
    std::thread t(producer, std::move(p));
    std::cout << "Result: " << f.get() << '\n';   // 42
    t.join();
}
```

**Expected Output**
```
Result: 42
```

**Why this output?**  
The producer thread sets the promise's value; `f.get()` blocks until the value is set and returns `42`.

### Real-World Cases

- **Custom async primitives** – building futures on top of promises.
- **Event signalling** – notifying completion.
- **Thread handoff** – passing results between threads.

---

## 3. `std::packaged_task`

### Definitions

**Core Definition**
`std::packaged_task<Sig>` is a class template that wraps a callable and stores its result (or exception) in a shared state, which an associated future can retrieve.

**Technical Definition**
`std::packaged_task<Sig>` (since C++11) wraps a callable with signature `Sig`. When the task is invoked, it stores the return value or exception in a shared state. `get_future()` returns the associated future. The task is move-only and can be invoked on any thread.

**Beginner-Friendly Explanation**
A packaged task is a “callable with a built-in future.” You wrap a function in a packaged task, get its future, and invoke it whenever and wherever you like. The future receives the result.

### Purposes

- To wrap a callable for asynchronous execution.
- To decouple task creation from task execution.
- To pass tasks to thread pools.
- To obtain results from tasks.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::packaged_task<int()> task([]{ return 42; });
std::future<int> f = task.get_future();
std::thread t(std::move(task));
int result = f.get();
t.join();
```

**Component Breakdown**

| Member | Description |
|--------|-------------|
| `get_future()` | Returns the associated future. |
| `operator()` | Invokes the task and stores the result. |
| `reset()` | Resets the shared state (C++11). |

**Syntax Rules**
- `get_future()` can be called only once.
- The task is move-only.
- Invoking a task without an associated future is allowed, but the result is discarded.

**Constraints and Limitations**
- Calling `get_future()` twice throws `std::future_error`.
- Calling an empty packaged task throws `std::future_error`.

### Annotated Code Example

```cpp
#include <future>
#include <iostream>
#include <thread>

int compute(int x) { return x * 2; }

int main() {
    std::packaged_task<int(int)> task(compute);
    std::future<int> f = task.get_future();

    std::thread t(std::move(task), 21);
    std::cout << "Result: " << f.get() << '\n';   // 42
    t.join();
}
```

**Expected Output**
```
Result: 42
```

**Why this output?**  
The packaged task wraps `compute`; invoking it with `21` stores `42` in the shared state, and `f.get()` returns it.

### Real-World Cases

- **Thread pools** – queuing packaged tasks for execution.
- **Deferred execution** – creating tasks and running them later.
- **GUI frameworks** – posting tasks to the UI thread.

---

## 4. `std::async`

### Definitions

**Core Definition**
`std::async` is a function template that runs a callable asynchronously and returns a future for its result.

**Technical Definition**
`std::async(policy, callable, args...)` (since C++11) launches `callable(args...)` according to the launch policy and returns a `std::future` for the result. Launch policies are `std::launch::async` (run on a new thread), `std::launch::deferred` (run lazily on `get()`), or the default (`async | deferred`, implementation chooses).

**Beginner-Friendly Explanation**
`std::async` is the easiest way to run a function in the background. It returns a future you can use to get the result.

### Purposes

- To run a task asynchronously.
- To obtain the result via a future.
- To simplify simple async patterns.
- To leverage the standard library's task launching.

### Syntax Rules and Structure

**General Syntax**
```cpp
auto f = std::async(std::launch::async, callable, args...);
```

**Component Breakdown**

| Policy | Description |
|--------|-------------|
| `std::launch::async` | Run on a new thread. |
| `std::launch::deferred` | Run lazily on `get()`/`wait()`. |
| `async | deferred` (default) | Implementation chooses. |

**Syntax Rules**
- The callable and arguments are decayed copies.
- The returned future's destructor may block (for `async` policy) until the task completes.
- `std::async` may throw `std::system_error` if it cannot start a thread.

**Constraints and Limitations**
- Overuse of `std::async` can create too many threads.
- The default policy's behaviour is implementation-defined.

### Annotated Code Example

```cpp
#include <future>
#include <iostream>

int compute(int x) { return x * x; }

int main() {
    auto f = std::async(std::launch::async, compute, 7);
    std::cout << "Result: " << f.get() << '\n';   // 49
}
```

**Expected Output**
```
Result: 49
```

**Why this output?**  
`std::async` runs `compute(7)` asynchronously; `f.get()` returns `49`.

### Real-World Cases

- **Background computations** – offloading work from the main thread.
- **Parallel processing** – launching multiple `std::async` tasks.
- **Simple async APIs** – the easiest async abstraction.

---

## 5. Future Operations

### Definitions

**Core Definition**
Future operations are the member functions used to wait for, retrieve, or check the result of a future.

**Technical Definition**
The operations are `get`, `wait`, `wait_for`, `wait_until`, and `valid`. `get` blocks until the result is available and returns it; `wait` blocks without returning the result; the timed waits return a `std::future_status`.

**Beginner-Friendly Explanation**
These are the commands for using a future: wait for the result, wait with a timeout, retrieve the result, and check if the future is valid.

---

### 5.1 `get`

**Core Definition**
`get()` blocks until the result is available and returns it (or rethrows the stored exception).

**Syntax**
```cpp
int value = f.get();
```

**Constraints**
- `get()` can be called only once; the future becomes invalid.

---

### 5.2 `wait`

**Core Definition**
`wait()` blocks until the result is available without retrieving it.

**Syntax**
```cpp
f.wait();
```

**Constraints**
- Does not invalidate the future.

---

### 5.3 `wait_for`

**Core Definition**
`wait_for(duration)` waits for a duration and returns a status.

**Syntax**
```cpp
auto status = f.wait_for(std::chrono::milliseconds(100));
```

**Return Values**
- `std::future_status::ready` – result is available.
- `std::future_status::timeout` – timed out.
- `std::future_status::deferred` – deferred task.

---

### 5.4 `wait_until`

**Core Definition**
`wait_until(time_point)` waits until a deadline and returns a status.

**Syntax**
```cpp
auto deadline = std::chrono::steady_clock::now() + std::chrono::seconds(1);
auto status = f.wait_until(deadline);
```

---

### 5.5 `valid`

**Core Definition**
`valid()` returns `true` if the future is associated with a shared state.

**Syntax**
```cpp
if (f.valid()) { /* ... */ }
```

**Constraints**
- After `get()`, `valid()` returns `false`.

---

### Annotated Code Example

```cpp
#include <future>
#include <iostream>
#include <chrono>

int compute() {
    std::this_thread::sleep_for(std::chrono::milliseconds(200));
    return 42;
}

int main() {
    auto f = std::async(std::launch::async, compute);

    if (f.wait_for(std::chrono::milliseconds(50)) == std::future_status::timeout) {
        std::cout << "Not ready yet\n";
    }

    std::cout << "Result: " << f.get() << '\n';   // 42
    std::cout << std::boolalpha << "Valid: " << f.valid() << '\n'; // false
}
```

**Expected Output**
```
Not ready yet
Result: 42
Valid: false
```

**Why this output?**  
`wait_for` times out after 50 ms; `get()` waits for the result; after `get()`, the future is invalid.

---

## 6. `std::shared_future`

### Definitions

**Core Definition**
`std::shared_future<T>` is a copyable future that allows multiple threads to wait for and retrieve the same result.

**Technical Definition**
`std::shared_future<T>` (since C++11) provides the same interface as `std::future` but is copyable. `get()` returns a copy of the value and can be called multiple times. It is constructed from a `std::future` (via `share()` or move) or another `shared_future`.

**Beginner-Friendly Explanation**
A shared future is a future that multiple threads can wait on. Unlike a regular future, it doesn't get consumed by `get()`.

### Purposes

- To share a result among multiple threads.
- To allow multiple retrievals of the same value.
- To broadcast a result.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::shared_future<int> sf = f.share();
```

**Constraints and Limitations**
- `get()` returns `const T&` (or a copy for `void`).
- The shared state must remain alive.

### Annotated Code Example

```cpp
#include <future>
#include <iostream>
#include <thread>
#include <vector>

int main() {
    std::promise<int> p;
    std::shared_future<int> sf = p.get_future().share();

    std::vector<std::thread> threads;
    for (int i = 0; i < 3; ++i) {
        threads.emplace_back([sf, i] {
            std::cout << "Thread " << i << ": " << sf.get() << '\n';
        });
    }

    p.set_value(42);
    for (auto& t : threads) t.join();
}
```

**Expected Output** (order may vary)
```
Thread 0: 42
Thread 1: 42
Thread 2: 42
```

**Why this output?**  
All three threads share the same `shared_future` and retrieve the value `42` after the promise sets it.

### Real-World Cases

- **Broadcasting results** – multiple consumers of one result.
- **Fan-out patterns** – one producer, many consumers.

---

## 7. Future Best Practices

### Definitions

**Core Definition**
Future best practices are guidelines for using futures, promises, and async tasks correctly, safely, and efficiently.

**Technical Definition**
These practices are informed by the C++ Core Guidelines (CP.60: “Use a `future` to return a value from a concurrent task”; CP.61: “Use `async()` to spawn concurrent tasks”; CP.62: “Use `future` to delay execution or to communicate a value between threads”) and community consensus: prefer `std::async` for simple tasks; use promises for custom communication; use `shared_future` for multiple consumers; avoid destroying futures without waiting; and be aware of the blocking destructor of `std::async` futures.

**Beginner-Friendly Explanation**
Best practices for futures are simple: use `std::async` for simple tasks, use promises for custom communication, and don't let futures be destroyed before you've handled their results.

### Key Guidelines

| Guideline | Summary |
|-----------|---------|
| **CP.60** | Use a `future` to return a value from a concurrent task. |
| **CP.61** | Use `async()` to spawn concurrent tasks. |
| **CP.62** | Use `future` to delay execution or communicate between threads. |
| **Prefer `std::async`** | For simple async tasks. |
| **Use `std::promise` for custom communication** | When you need fine control. |
| **Use `shared_future` for multiple consumers** | Share results safely. |
| **Handle exceptions** | `get()` rethrows stored exceptions. |
| **Be aware of the blocking destructor** | Futures from `std::async` may block in their destructor. |
| **Avoid `std::launch::deferred` unless needed** | Deferred tasks run on `get()`, which may surprise. |

### Annotated Code Example

```cpp
#include <future>
#include <iostream>
#include <vector>

int compute(int x) { return x * x; }

int main() {
    std::vector<std::future<int>> futures;
    for (int i = 1; i <= 5; ++i)
        futures.push_back(std::async(std::launch::async, compute, i));

    for (auto& f : futures)
        std::cout << f.get() << ' ';        // 1 4 9 16 25
}
```

**Expected Output**
```
1 4 9 16 25
```

**Why this output?**  
Each `std::async` launches `compute(i)`; the futures are collected and retrieved in order.

### Real-World Cases

- **Parallel computations** – launching multiple tasks.
- **Producer-consumer** – using promises and futures.
- **Fan-out/fan-in** – `shared_future` for multiple consumers.

---

## References

- `std::future` — cppreference – https://en.cppreference.com/w/cpp/thread/future
- `std::promise` — cppreference – https://en.cppreference.com/w/cpp/thread/promise
- `std::packaged_task` — cppreference – https://en.cppreference.com/w/cpp/thread/packaged_task
- `std::async` — cppreference – https://en.cppreference.com/w/cpp/thread/async
- `std::shared_future` — cppreference – https://en.cppreference.com/w/cpp/thread/shared_future
- `std::future_status` — cppreference – https://en.cppreference.com/w/cpp/thread/future_status
- `std::launch` — cppreference – https://en.cppreference.com/w/cpp/thread/launch
- C++ Core Guidelines — CP.60: Use a `future` to return a value from a concurrent task – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-future
- C++ Core Guidelines — CP.61: Use `async()` to spawn concurrent tasks – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-async
- C++ Core Guidelines — CP.62: Use `future` to delay execution or to communicate a value between threads – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-future

---

This cheat sheet provides a comprehensive, structured reference for C++ futures and promises, covering `std::future`, `std::promise`, `std::packaged_task`, `std::async`, the future operations (`get`, `wait`, `wait_for`, `wait_until`, `valid`), `std::shared_future`, and best practices, with annotated code examples, expected outputs, real-world cases, and authoritative references.