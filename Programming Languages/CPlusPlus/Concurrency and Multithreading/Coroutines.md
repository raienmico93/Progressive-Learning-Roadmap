# C++20 Coroutines — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A coroutine is a function that can suspend execution to be resumed later, preserving its local state across suspensions, enabling sequential-style code for asynchronous operations and lazy sequence generation.

**Technical Definition**
C++20 coroutines are **stackless**: they suspend by returning to the caller, and the data required to resume execution is stored separately from the stack in a heap-allocated (usually) **coroutine state** object. A function is a coroutine if its definition contains any of three keywords: `co_await`, `co_yield`, or `co_return`. The compiler transforms the function into a state machine that interacts with a user-supplied **promise type** and is controlled externally via a **coroutine handle**.

**Beginner-Friendly Explanation**
A coroutine is a function that can pause in the middle and later continue where it left off, keeping its local variables alive. Think of it as a “resumable function”: it runs until it hits a suspension point, returns control to the caller, and can be resumed later to continue from exactly that point. This is the foundation for writing asynchronous code that reads like synchronous code, and for generating sequences lazily.

### Key Characteristics

- **Stackless** – suspension returns to the caller; state is stored in a separate coroutine frame, not on the stack.
- **Three keywords** – `co_await` (suspend until resumed), `co_yield` (produce a value and suspend), `co_return` (complete with a value).
- **Promise type** – a user-defined policy object that controls coroutine behaviour (start, end, value flow, exception handling).
- **Coroutine handle** – a non-owning pointer-like token used to resume or destroy the coroutine from outside.
- **Awaitable protocol** – any type implementing `await_ready`, `await_suspend`, and `await_resume` can be used with `co_await`.
- **Generators and tasks** – standard library provides `std::generator` (C++23) and `std::execution::task` (C++26); custom types are common.

### Prerequisites

- Solid understanding of functions, lambdas, and move semantics.
- Familiarity with templates and type traits.
- Knowledge of exception handling and `std::coroutine_handle`.
- Awareness of the C++20 standard (coroutines are a core language feature).

### Related Programming Areas

- **Asynchronous I/O** – coroutines for non-blocking sockets and files.
- **Generators and lazy sequences** – `co_yield` for on-demand value production.
- **Task-based concurrency** – `co_await` for composable async tasks.
- **Ranges (C++20/23)** – `std::generator` models `std::ranges::input_range`.
- **Senders/receivers (C++26)** – `std::execution` provides a structured async model that interoperates with coroutines.

### Core Concepts / Features

1. Coroutines (the language feature)
2. `co_await`
3. `co_yield`
4. `co_return`
5. Coroutine framework (promise type, awaitable, coroutine handle)
6. Promise type
7. Awaitable
8. Coroutine handles
9. Generators
10. Async tasks
11. Coroutine best practices

---

## 1. Coroutines

### Definitions

**Core Definition**
A coroutine is a function that can suspend its execution and later resume it, maintaining its local state across suspensions.

**Technical Definition**
A coroutine is a function whose body contains at least one of `co_await`, `co_yield`, or `co_return`. The compiler transforms it into a state machine. Each coroutine is associated with a **promise object** (manipulated from inside), a **coroutine handle** (manipulated from outside), and a **coroutine state** (heap-allocated storage holding the promise, parameter copies, suspension point, and live locals). Coroutines are stackless and cannot use variadic arguments, plain `return` statements, or placeholder return types (`auto`, concepts); constructors, destructors, `main`, `constexpr`, and `consteval` functions cannot be coroutines.

**Beginner-Friendly Explanation**
A coroutine is a function that can pause and resume. It keeps its local variables alive while paused, and you control when it continues.

### Purposes

- To write asynchronous code that reads like sequential code (no callback hell).
- To implement lazy generators and infinite sequences.
- To support cooperative multitasking and task composition.
- To separate suspension logic from business logic.

### Syntax Rules and Structure

**General Syntax**
```cpp
ReturnType coroutine_function(Parameters) {
    // ...
    co_await expression;      // suspend until resumed
    co_yield value;           // produce a value and suspend
    co_return value;          // complete with a value
}
```

**Component Breakdown**

| Keyword | Effect |
|---------|--------|
| `co_await expr` | Suspends until the awaitable is ready; resumes with a result. |
| `co_yield value` | Suspends and produces `value` to the caller (equivalent to `co_await promise.yield_value(value)`). |
| `co_return value` | Completes the coroutine with a value (calls `promise.return_value(value)`). |
| `co_return;` | Completes with no value (calls `promise.return_void()`). |

**Constraints and Limitations**
- Cannot be `constexpr`, `consteval`, constructors, destructors, or `main`.
- Cannot use variadic arguments or plain `return` statements.
- The return type must satisfy the coroutine promise requirements.

### Annotated Code Example

```cpp
#include <coroutine>
#include <iostream>

struct SimpleCoro {
    struct promise_type {
        SimpleCoro get_return_object() { return {}; }
        std::suspend_never initial_suspend() { return {}; }
        std::suspend_never final_suspend() noexcept { return {}; }
        void return_void() {}
        void unhandled_exception() { std::terminate(); }
    };
};

SimpleCoro hello() {
    std::cout << "Hello from coroutine\n";
    co_return;
}

int main() {
    hello();
    std::cout << "Back in main\n";
}
```

**Expected Output**
```
Hello from coroutine
Back in main
```

**Why this output?**  
`initial_suspend` returns `suspend_never`, so the coroutine starts eagerly; it prints and completes without suspending.

### Real-World Cases

- **Async I/O** – `co_await socket.async_read_some(...)` in a TCP echo server.
- **Lazy sequences** – `generator<int> iota() { while(true) co_yield n++; }`.

---

## 2. `co_await`

### Definitions

**Core Definition**
`co_await` is an expression that suspends the coroutine until the awaited operation completes, then resumes with a result.

**Technical Definition**
`co_await expr` converts `expr` to an **awaitable** (possibly via `promise.await_transform`), obtains an **awaiter** object, and calls three methods: `await_ready()` (if `true`, no suspension), `await_suspend(handle)` (schedules resumption), and `await_resume()` (produces the result). The awaitable protocol enables any type to customise suspension behaviour.

**Beginner-Friendly Explanation**
`co_await` means “pause here until this operation finishes, then continue with its result.” It is how a coroutine waits for something without blocking a thread.

### Purposes

- To suspend a coroutine until an asynchronous operation completes.
- To integrate custom awaitable types into coroutine control flow.
- To write sequential-style async code.

### Syntax Rules and Structure

**General Syntax**
```cpp
auto result = co_await awaitable_expression;
```

**Component Breakdown**

| Awaiter Method | Role |
|----------------|------|
| `await_ready()` | Returns `true` if ready (no suspension). |
| `await_suspend(handle)` | Schedules resumption; may return `void`, `bool`, or another coroutine handle. |
| `await_resume()` | Returns the result (or rethrows an exception). |

**Constraints and Limitations**
- `co_await` cannot appear in handlers, default arguments, or certain static initialisers.
- If `await_suspend` throws, the coroutine is resumed and the exception is rethrown.

### Annotated Code Example

```cpp
#include <coroutine>
#include <iostream>

struct MyAwaitable {
    bool await_ready() const noexcept {
        std::cout << "await_ready\n";
        return false;                       // always suspend
    }
    void await_suspend(std::coroutine_handle<>) const noexcept {
        std::cout << "await_suspend\n";
    }
    int await_resume() const noexcept {
        std::cout << "await_resume\n";
        return 42;
    }
};

struct Task {
    struct promise_type {
        Task get_return_object() { return {}; }
        std::suspend_never initial_suspend() { return {}; }
        std::suspend_never final_suspend() noexcept { return {}; }
        void return_void() {}
        void unhandled_exception() { std::terminate(); }
    };
};

Task demo() {
    int value = co_await MyAwaitable{};
    std::cout << "Got: " << value << '\n';
}

int main() {
    demo();
}
```

**Expected Output**
```
await_ready
await_suspend
await_resume
Got: 42
```

**Why this output?**  
The awaiter suspends, then `await_resume` returns `42`, which the coroutine uses.

### Real-World Cases

- **Custom timers** – an awaitable timer suspends and resumes after a delay.
- **Async I/O** – awaiting socket operations in a TCP server.

---

## 3. `co_yield`

### Definitions

**Core Definition**
`co_yield` produces a value to the caller and suspends the coroutine, enabling lazy generation of sequences.

**Technical Definition**
`co_yield expr` is equivalent to `co_await promise.yield_value(expr)`. A typical `yield_value` stores the value in the coroutine object and returns `std::suspend_always`, transferring control to the caller/resumer. The standard library provides `std::generator` (C++23) as a ready-made generator type.

**Beginner-Friendly Explanation**
`co_yield` means “produce this value and pause.” When the caller asks for the next value, the coroutine resumes and produces the next one.

### Purposes

- To build lazy generators that produce values on demand.
- To implement infinite or large sequences without storing them.
- To support pipeline-style data processing.

### Syntax Rules and Structure

**General Syntax**
```cpp
generator<T> sequence() {
    while (condition) {
        co_yield value;
    }
}
```

**Constraints and Limitations**
- The coroutine's return type must provide a `promise_type` with `yield_value` and appropriate suspension.
- `std::generator` (C++23) models `std::ranges::input_range` and can be used in range-based `for` loops.

### Annotated Code Example

```cpp
#include <coroutine>
#include <iostream>
#include <vector>

template <typename T>
struct Generator {
    struct promise_type {
        T current_value;
        Generator get_return_object() {
            return Generator{std::coroutine_handle<promise_type>::from_promise(*this)};
        }
        std::suspend_always initial_suspend() { return {}; }
        std::suspend_always final_suspend() noexcept { return {}; }
        std::suspend_always yield_value(T value) {
            current_value = value;
            return {};
        }
        void return_void() {}
        void unhandled_exception() { std::terminate(); }
    };

    std::coroutine_handle<promise_type> handle;
    explicit Generator(std::coroutine_handle<promise_type> h) : handle(h) {}
    ~Generator() { if (handle) handle.destroy(); }

    bool next() {
        handle.resume();
        return !handle.done();
    }
    T value() const { return handle.promise().current_value; }
};

Generator<int> count_to(int n) {
    for (int i = 1; i <= n; ++i)
        co_yield i;
}

int main() {
    auto g = count_to(3);
    while (g.next())
        std::cout << g.value() << ' ';
}
```

**Expected Output**
```
1 2 3
```

**Why this output?**  
`count_to` yields `1`, `2`, `3`; `next()` resumes and checks for completion, and `value()` retrieves each yielded value.

### Real-World Cases

- **Fibonacci sequences** – `std::generator` produces values on demand.
- **Tree traversal** – `std::generator` recursively yields elements.

---

## 4. `co_return`

### Definitions

**Core Definition**
`co_return` completes a coroutine, optionally returning a final value.

**Technical Definition**
`co_return value` calls `promise.return_value(value)`; `co_return;` calls `promise.return_void()`. After `co_return`, the coroutine destroys automatic variables in reverse order, then calls `final_suspend()` and co_awaits it. A coroutine that flows off the end without `co_return` is equivalent to `co_return;` (if `return_void` exists).

**Beginner-Friendly Explanation**
`co_return` is how a coroutine finishes. It can hand back a final value or simply end.

### Purposes

- To complete a coroutine with a final result.
- To signal completion of an async task.
- To provide the return value of a task type.

### Syntax Rules and Structure

**General Syntax**
```cpp
co_return value;      // if promise has return_value
co_return;            // if promise has return_void
```

**Constraints and Limitations**
- The promise type must define either `return_value` or `return_void`, not both.
- The return value type must be convertible to the promise's expected type.

### Annotated Code Example

```cpp
#include <coroutine>
#include <iostream>

struct Task {
    struct promise_type {
        Task get_return_object() { return {}; }
        std::suspend_never initial_suspend() { return {}; }
        std::suspend_never final_suspend() noexcept { return {}; }
        void return_value(int v) {
            std::cout << "Returned: " << v << '\n';
        }
        void unhandled_exception() { std::terminate(); }
    };
};

Task compute() {
    co_return 42;
}

int main() {
    compute();
}
```

**Expected Output**
```
Returned: 42
```

**Why this output?**  
`co_return 42` calls `promise.return_value(42)`, which prints the value.

### Real-World Cases

- **Async tasks** – `task<int> compute() { co_return 42; }`.
- **Generators** – `co_return` ends a generator after yielding values.

---

## 5. Coroutine Framework (Promise, Awaitable, Handle)

### Definitions

**Core Definition**
The coroutine framework is the set of three collaborating pieces: the promise type (policy object), the awaitable protocol (suspension control), and the coroutine handle (external control token).

**Technical Definition**
Every coroutine is built from three pillars:
- **Promise type** – controls initial/final suspend, return value, yield, and unhandled exceptions.
- **Awaitable** – implements `await_ready`, `await_suspend`, and `await_resume` to control each `co_await`.
- **Coroutine handle** – a raw pointer-like token (`std::coroutine_handle`) that lets you resume or destroy the coroutine from outside.

**Beginner-Friendly Explanation**
The framework is the “machinery” behind a coroutine: the promise decides how it starts and ends, the awaitable decides how it pauses, and the handle lets you control it from outside.

### Purposes

- To provide full control over coroutine behaviour.
- To enable custom coroutine types (generators, tasks, etc.).
- To support symmetric transfer and custom scheduling.

### Syntax Rules and Structure

**Promise Type Required Members**

| Member | Role |
|--------|------|
| `get_return_object()` | Creates the user-facing coroutine object. |
| `initial_suspend()` | Returns `suspend_always` (lazy) or `suspend_never` (eager). |
| `final_suspend()` | Must be `noexcept`; controls cleanup. |
| `return_value(v)` / `return_void()` | Handles `co_return`. |
| `yield_value(v)` | Handles `co_yield`. |
| `unhandled_exception()` | Handles uncaught exceptions. |

**Coroutine Handle Operations**

| Operation | Description |
|-----------|-------------|
| `resume()` | Resumes execution. |
| `destroy()` | Destroys the coroutine frame. |
| `done()` | Checks if the coroutine is at its final suspend point. |
| `promise()` | Accesses the promise object. |

### Annotated Code Example

```cpp
#include <coroutine>
#include <iostream>

struct Task {
    struct promise_type {
        Task get_return_object() {
            return Task{std::coroutine_handle<promise_type>::from_promise(*this)};
        }
        std::suspend_always initial_suspend() { return {}; }
        std::suspend_always final_suspend() noexcept { return {}; }
        void return_void() {}
        void unhandled_exception() { std::terminate(); }
    };

    std::coroutine_handle<promise_type> handle;
    explicit Task(std::coroutine_handle<promise_type> h) : handle(h) {}
    ~Task() { if (handle) handle.destroy(); }

    void resume() { handle.resume(); }
};

Task demo() {
    std::cout << "Coroutine running\n";
    co_return;
}

int main() {
    auto t = demo();
    std::cout << "Before resume\n";
    t.resume();
    std::cout << "After resume\n";
}
```

**Expected Output**
```
Before resume
Coroutine running
After resume
```

**Why this output?**  
`initial_suspend` returns `suspend_always`, so the coroutine does not start until `resume()` is called.

### Real-World Cases

- **Custom task types** – building async task coroutines with full control.
- **Symmetric transfer** – chaining coroutines without stack growth.

---

## 6. Promise Type

### Definitions

**Core Definition**
The promise type is a user-defined class that acts as the coroutine's policy object, controlling its lifecycle and value flow.

**Technical Definition**
The promise type is nested inside the coroutine's return type (or found via `coroutine_traits`). It must provide `get_return_object`, `initial_suspend`, `final_suspend`, `return_value`/`return_void`, `yield_value` (if `co_yield` is used), and `unhandled_exception`. The compiler constructs the promise object inside the coroutine frame and interacts with it at each suspension point.

**Beginner-Friendly Explanation**
The promise type is the “brain” of the coroutine. It decides what happens when the coroutine starts, suspends, yields, returns, or throws.

### Purposes

- To customise coroutine behaviour for different use cases (generators, tasks).
- To store yielded values or final results.
- To handle exceptions and finalisation.

### Syntax Rules and Structure

**General Syntax**
```cpp
struct ReturnType {
    struct promise_type {
        ReturnType get_return_object();
        std::suspend_always initial_suspend();
        std::suspend_always final_suspend() noexcept;
        void return_void();              // or return_value(T)
        std::suspend_always yield_value(T); // if co_yield is used
        void unhandled_exception();
    };
};
```

**Constraints and Limitations**
- `final_suspend` must be `noexcept`.
- The promise type must be default-constructible or constructible from the coroutine parameters.

### Annotated Code Example

```cpp
#include <coroutine>
#include <iostream>

struct Generator {
    struct promise_type {
        int current_value;
        Generator get_return_object() {
            return Generator{std::coroutine_handle<promise_type>::from_promise(*this)};
        }
        std::suspend_always initial_suspend() { return {}; }
        std::suspend_always final_suspend() noexcept { return {}; }
        std::suspend_always yield_value(int v) {
            current_value = v;
            return {};
        }
        void return_void() {}
        void unhandled_exception() { std::terminate(); }
    };

    std::coroutine_handle<promise_type> handle;
    explicit Generator(std::coroutine_handle<promise_type> h) : handle(h) {}
    ~Generator() { if (handle) handle.destroy(); }

    bool next() { handle.resume(); return !handle.done(); }
    int value() const { return handle.promise().current_value; }
};

Generator count() {
    co_yield 10;
    co_yield 20;
}

int main() {
    auto g = count();
    while (g.next()) std::cout << g.value() << ' '; // 10 20
}
```

**Expected Output**
```
10 20
```

**Why this output?**  
The promise's `yield_value` stores each yielded value in `current_value`, which the generator retrieves.

### Real-World Cases

- **Generator promise** – stores yielded values.
- **Task promise** – stores the final result or exception.

---

## 7. Awaitable

### Definitions

**Core Definition**
An awaitable is any type that can be used with `co_await`, either directly (if it implements the awaiter protocol) or via `operator co_await` or `promise.await_transform`.

**Technical Definition**
An awaitable must produce an **awaiter** object with three methods: `await_ready()` (returns `bool`), `await_suspend(handle)` (returns `void`, `bool`, or a `coroutine_handle`), and `await_resume()` (returns the result). Any type implementing these three methods is an awaiter. The awaitable protocol allows custom suspension logic for timers, I/O, or other asynchronous operations.

**Beginner-Friendly Explanation**
An awaitable is anything you can `co_await`. It defines what “waiting” means: whether to suspend, what to do while suspended, and what value to produce when resumed.

### Purposes

- To customise suspension and resumption behaviour.
- To integrate asynchronous operations (timers, I/O) with coroutines.
- To support lazy evaluation and scheduling.

### Syntax Rules and Structure

**General Syntax**
```cpp
struct MyAwaiter {
    bool await_ready() const noexcept;
    void await_suspend(std::coroutine_handle<>) const noexcept;
    int await_resume() const noexcept;
};
```

**Component Breakdown**

| Method | Role |
|--------|------|
| `await_ready()` | `true` = no suspension. |
| `await_suspend(handle)` | Schedules resumption; may return `void`, `bool`, or another handle. |
| `await_resume()` | Produces the result. |

**Constraints and Limitations**
- If `await_suspend` throws, the coroutine is resumed and the exception is rethrown.
- Returning `false` from `await_suspend` resumes the coroutine immediately.

### Annotated Code Example

```cpp
#include <coroutine>
#include <iostream>
#include <thread>

struct TimerAwaiter {
    int ms;
    bool await_ready() const noexcept { return false; }
    void await_suspend(std::coroutine_handle<> h) const {
        std::thread([h, ms = ms] {
            std::this_thread::sleep_for(std::chrono::milliseconds(ms));
            h.resume();
        }).detach();
    }
    void await_resume() const noexcept {
        std::cout << "Timer fired\n";
    }
};

struct Task {
    struct promise_type {
        Task get_return_object() { return {}; }
        std::suspend_never initial_suspend() { return {}; }
        std::suspend_never final_suspend() noexcept { return {}; }
        void return_void() {}
        void unhandled_exception() { std::terminate(); }
    };
};

Task demo() {
    co_await TimerAwaiter{100};
    std::cout << "After timer\n";
}

int main() {
    demo();
    std::this_thread::sleep_for(std::chrono::milliseconds(200));
}
```

**Expected Output** (order may vary)
```
Timer fired
After timer
```

**Why this output?**  
The awaiter spawns a thread that sleeps and resumes the coroutine, which then prints “After timer”.

### Real-World Cases

- **Async timers** – suspend and resume after a delay.
- **I/O awaitables** – integrate with non-blocking sockets.

---

## 8. Coroutine Handles

### Definitions

**Core Definition**
`std::coroutine_handle` is a non-owning, pointer-like token that refers to a coroutine frame and provides operations to resume, destroy, or query it.

**Technical Definition**
`std::coroutine_handle<Promise>` is a type-erased handle to a coroutine frame. It provides `resume()`, `destroy()`, `done()`, `promise()`, and `operator bool`. The handle is typically obtained via `coroutine_handle<promise_type>::from_promise(*this)` inside `get_return_object`. It does **not** own the coroutine; destroying the handle does not destroy the frame unless `destroy()` is called.

**Beginner-Friendly Explanation**
A coroutine handle is a “remote control” for a coroutine. It lets you resume it, destroy it, or check if it is done.

### Purposes

- To resume a suspended coroutine.
- To destroy a coroutine frame.
- To access the promise object.
- To support symmetric transfer.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::coroutine_handle<Promise> h = /* ... */;
h.resume();
h.destroy();
if (h.done()) { /* ... */ }
```

**Constraints and Limitations**
- Resuming a coroutine that is not suspended is undefined behaviour.
- Destroying a coroutine that is not suspended is undefined behaviour.
- Handles do not own the coroutine; you must call `destroy()` explicitly (or use an RAII wrapper).

### Annotated Code Example

```cpp
#include <coroutine>
#include <iostream>

struct Task {
    struct promise_type {
        Task get_return_object() {
            return Task{std::coroutine_handle<promise_type>::from_promise(*this)};
        }
        std::suspend_always initial_suspend() { return {}; }
        std::suspend_always final_suspend() noexcept { return {}; }
        void return_void() {}
        void unhandled_exception() { std::terminate(); }
    };

    std::coroutine_handle<promise_type> handle;
    explicit Task(std::coroutine_handle<promise_type> h) : handle(h) {}
    ~Task() { if (handle) handle.destroy(); }

    void resume() { handle.resume(); }
    bool done() const { return handle.done(); }
};

Task demo() {
    std::cout << "Step 1\n";
    co_await std::suspend_always{};
    std::cout << "Step 2\n";
}

int main() {
    auto t = demo();
    std::cout << "Before resume\n";
    t.resume();
    std::cout << "After first resume\n";
    t.resume();
    std::cout << "Done: " << std::boolalpha << t.done() << '\n';
}
```

**Expected Output**
```
Before resume
Step 1
After first resume
Step 2
Done: true
```

**Why this output?**  
The coroutine suspends after “Step 1”; resuming again prints “Step 2” and completes.

### Real-World Cases

- **Task scheduling** – storing handles in a queue and resuming them.
- **Symmetric transfer** – returning a handle from `await_suspend` to resume another coroutine.

---

## 9. Generators

### Definitions

**Core Definition**
A generator is a coroutine that produces a sequence of values lazily via `co_yield`, computing each value only when requested.

**Technical Definition**
A generator is a coroutine whose promise type stores the yielded value and returns `std::suspend_always` from `yield_value`. C++23 provides `std::generator<T>` in `<generator>`, which models `std::ranges::view` and `std::ranges::input_range`, and can be used in range-based `for` loops. `std::generator` generates elements by repeatedly resuming the coroutine from which it was returned.

**Beginner-Friendly Explanation**
A generator is a “lazy sequence factory.” It produces one value at a time, only when you ask for it, so it can represent infinite sequences without using infinite memory.

### Purposes

- To produce lazy sequences on demand.
- To implement infinite or large sequences without storing them.
- To simplify iteration over generated data.

### Syntax Rules and Structure

**General Syntax (std::generator, C++23)**
```cpp
#include <generator>

std::generator<int> iota(int n = 0) {
    while (true)
        co_yield n++;
}
```

**Component Breakdown**

| Feature | Description |
|---------|-------------|
| `std::generator<T>` | Coroutine return type for generators (C++23). |
| `co_yield` | Produces a value and suspends. |
| Range interface | Models `input_range`, usable in range-based `for`. |

**Constraints and Limitations**
- `std::generator` requires C++23.
- Custom generators require a full promise type and handle management.

### Annotated Code Example (std::generator)

```cpp
#include <generator>
#include <iostream>

std::generator<int> count_to(int n) {
    for (int i = 1; i <= n; ++i)
        co_yield i;
}

int main() {
    for (int x : count_to(5))
        std::cout << x << ' ';
}
```

**Expected Output**
```
1 2 3 4 5
```

**Why this output?**  
`count_to` yields `1` through `5`; the range-based `for` consumes them lazily.

### Real-World Cases

- **Fibonacci** – `std::generator` produces Fibonacci numbers on demand.
- **Tree traversal** – recursively yielding tree elements.

---

## 10. Async Tasks

### Definitions

**Core Definition**
An async task is a coroutine type that represents an asynchronous computation, typically producing a single result (or `void`) that can be awaited.

**Technical Definition**
Task types (e.g., `std::execution::task<T>` in C++26) are coroutines whose promise types store the result or exception and schedule resumption. They support `co_await` on other tasks, enabling structured async composition. C++26's `std::execution::task` integrates with the senders/receivers model, allowing tasks to be launched and awaited within `std::execution` pipelines.

**Beginner-Friendly Explanation**
An async task is a coroutine that does asynchronous work and produces a result. You can `co_await` it to wait for the result without blocking a thread.

### Purposes

- To represent asynchronous computations with a single result.
- To compose async operations with `co_await`.
- To integrate with structured concurrency frameworks.

### Syntax Rules and Structure

**General Syntax (Conceptual)**
```cpp
task<int> compute() {
    int x = co_await async_operation();
    co_return x * 2;
}
```

**Constraints and Limitations**
- Task types are library-specific (standard `std::execution::task` is C++26).
- Requires careful lifetime management (task frames must outlive their use).

### Annotated Code Example (Custom Task)

```cpp
#include <coroutine>
#include <iostream>
#include <thread>

struct Task {
    struct promise_type {
        int result;
        Task get_return_object() {
            return Task{std::coroutine_handle<promise_type>::from_promise(*this)};
        }
        std::suspend_never initial_suspend() { return {}; }
        std::suspend_always final_suspend() noexcept { return {}; }
        void return_value(int v) { result = v; }
        void unhandled_exception() { std::terminate(); }
    };

    std::coroutine_handle<promise_type> handle;
    explicit Task(std::coroutine_handle<promise_type> h) : handle(h) {}
    ~Task() { if (handle) handle.destroy(); }

    int get() {
        if (!handle.done()) handle.resume();
        return handle.promise().result;
    }
};

Task compute() {
    co_return 42;
}

int main() {
    auto t = compute();
    std::cout << t.get() << '\n';          // 42
}
```

**Expected Output**
```
42
```

**Why this output?**  
`compute` returns `42` via `co_return`; the task stores the result and `get()` retrieves it.

### Real-World Cases

- **Structured async pipelines** – combining coroutines with `std::execution`.
- **Network servers** – awaiting client requests in a task.

---

## 11. Coroutine Best Practices

### Definitions

**Core Definition**
Coroutine best practices are guidelines for using coroutines safely, efficiently, and idiomatically, avoiding lifetime hazards and ensuring correct suspension semantics.

**Technical Definition**
These practices are informed by the C++ Core Guidelines (CP.51, CP.53) and community consensus: avoid capturing lambdas in coroutines (captures may dangle after suspension); avoid reference parameters in coroutines (references may dangle after suspension); prefer passing by value so parameters are copied into the coroutine frame; ensure coroutine handles are destroyed (use RAII wrappers); and be mindful of allocation (coroutine frames are heap-allocated unless elided).

**Beginner-Friendly Explanation**
Best practices for coroutines are about lifetimes: avoid referencing things that might disappear while the coroutine is suspended, and make sure coroutine frames are cleaned up.

### Key Guidelines

| Guideline | Summary |
|-----------|---------|
| **CP.51** | Avoid capturing lambdas in coroutines; captures may dangle after suspension. |
| **CP.53** | Avoid reference parameters in coroutines; references may dangle after suspension. |
| **Pass by value** | Parameters are copied into the coroutine frame and live as long as the coroutine. |
| **Use RAII for handles** | Wrap `coroutine_handle` in a destructor to avoid leaks. |
| **Be aware of allocation** | Coroutine frames are heap-allocated unless the compiler can elide. |
| **Use `std::generator`** | Prefer the standard generator (C++23) over custom implementations. |
| **Validate with tools** | AddressSanitizer catches use-after-free in coroutine lambdas. |
| **Avoid resuming destroyed coroutines** | Resuming a non-suspended or destroyed coroutine is UB. |

### Annotated Code Example (Safe vs. Unsafe)

```cpp
#include <coroutine>
#include <iostream>

struct SafeTask {
    struct promise_type {
        SafeTask get_return_object() { return {}; }
        std::suspend_never initial_suspend() { return {}; }
        std::suspend_never final_suspend() noexcept { return {}; }
        void return_void() {}
        void unhandled_exception() { std::terminate(); }
    };
};

SafeTask safe_coro(int value) {           // by value: safe across suspension
    co_await std::suspend_never{};
    std::cout << value << '\n';
}

int main() {
    safe_coro(42);
}
```

**Expected Output**
```
42
```

**Why this output?**  
The parameter is passed by value and copied into the coroutine frame, so it remains valid across suspension.

### Real-World Cases

- **Code reviews** – flagging coroutine lambdas with captures.
- **Static analysis** – clang-tidy checks for CP.51 and CP.53.
- **Library design** – providing RAII wrappers for coroutine handles.

---

## References

- Coroutines (C++20) — cppreference — https://en.cppreference.com/w/cpp/language/coroutines
- `std::generator` — cppreference — https://en.cppreference.com/w/cpp/coroutine/generator
- Coroutines (C++20) — cppreference (printable) — https://www.cppreference.com/cpp/language/coroutines
- Understand the coroutine model: promise_type, coroutine_handle, and suspension — Dr-Sergey/learn_cpp — https://github.com/Dr-Sergey/learn_cpp/blob/5c03ef304b020f95190f4443b5f56e4ade270e83/16_Coroutines_Cpp20/Understand_the_coroutine_model_promise_type_coroutine_handle_and_suspension.md
- C++ Core Guidelines — CP.51: Avoid capturing lambdas in coroutines — https://github.com/intel/llvm/blob/32a089363166e1be8be4db66e046f7d6d34a1772/clang-tools-extra/docs/clang-tidy/checks/cppcoreguidelines/avoid-capturing-lambda-coroutines.rst
- C++ Core Guidelines — CP.53: Avoid reference parameters in coroutines — https://github.com/intel/llvm/blob/32a089363166e1be8be4db66e046f7d6d34a1772/clang-tools-extra/docs/clang-tidy/checks/cppcoreguidelines/avoid-reference-coroutine-parameters.rst
- P0912R5: Merge Coroutines TS into C++20 working draft — https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p0912r5.html
- `std::execution::task` (C++26) — cppreference — https://en.cppreference.com/w/cpp/execution/task

---

This cheat sheet provides a comprehensive, structured reference for C++20 coroutines, covering the language feature, the three keywords (`co_await`, `co_yield`, `co_return`), the framework (promise type, awaitable, coroutine handle), generators, async tasks, and best practices, with annotated code examples, expected outputs, real-world cases, and authoritative references.