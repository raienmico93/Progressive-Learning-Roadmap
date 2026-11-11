# C# Asynchronous Fundamentals — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Asynchronous programming in C# is a programming model that allows a program to **start a long-running operation and continue doing other work while waiting for it to complete**, instead of blocking the current thread until the operation finishes. It is built around the **Task-based Asynchronous Pattern (TAP)**, the `Task` and `ValueTask` types, and the `async` and `await` keywords. Asynchronous programming is essential for building responsive UIs, scalable web servers, and efficient I/O-bound applications.

**Technical Definition**
Asynchronous programming in C# is centered on the **Task-based Asynchronous Pattern (TAP)** , where asynchronous operations are represented by `Task`, `Task<TResult>`, `ValueTask`, or `ValueTask<TResult>` objects. The `async` modifier marks a method as an asynchronous function, and the `await` operator suspends the method's execution until the awaited task completes, without blocking the calling thread. The C# compiler transforms `async` methods into **state machines** that implement the suspension and resumption semantics. Asynchronous methods return task types; a method returning a task type is said to be **task-returning**. Task types can be in one of four states: **incomplete**, **succeeded**, **canceled**, or **faulted**. Tasks that are faulted record an exception; canceled tasks record an `OperationCanceledException`; succeeded `Task<TResult>` objects record a result of type `TResult`. The Task Parallel Library (TPL) is the underlying infrastructure, and it is the preferred API for writing multi-threaded, asynchronous, and parallel code in .NET.

**Beginner-Friendly Explanation**
Asynchronous programming is **“don't wait idly — do something else while you wait.”** Imagine ordering coffee at a café. In a synchronous world, you would stand at the counter staring at the barista until your coffee is ready (blocking). In an asynchronous world, you place your order, get a buzzer, and sit down to read your book until the buzzer goes off (non-blocking). You are not blocked — you are free to do other things. C# gives you two keywords, `async` and `await`, that let you write code that looks sequential but actually pauses at `await` and resumes when the operation completes, without tying up a thread. Under the hood, the compiler builds a **state machine** that remembers where you paused and resumes when the result is ready. This is how modern .NET applications handle thousands of concurrent requests without needing thousands of threads.

---

### Key Characteristics

- **Non-blocking** — `await` suspends the method without blocking the calling thread; the thread is freed to do other work.
- **Task-based** — Asynchronous operations are represented by `Task`, `Task<T>`, `ValueTask`, or `ValueTask<T>`.
- **Compiler-generated state machines** — The `async` modifier causes the compiler to rewrite the method into a state machine that handles suspension and resumption.
- **Task states** — Tasks can be incomplete, succeeded, canceled, or faulted; faulted tasks store exceptions.
- **Exception propagation** — Exceptions in async methods are captured in the returned `Task` and rethrown when the task is awaited.
- **Scalability** — Asynchronous I/O frees threads to handle other requests, improving scalability dramatically.
- **`async void` is for event handlers only** — `async void` methods cannot be awaited and exceptions cannot be caught by the caller.
- **`ValueTask` reduces allocations** — `ValueTask` is a struct that avoids heap allocation for synchronously completing async methods.

---

### Prerequisites

- Basic understanding of **C# methods**, return types, and exceptions.
- Familiarity with **delegates** and **lambda expressions** (for `Task.Run` and continuations).
- Awareness of **threads** and the **thread pool** (helpful but not strictly required).
- Understanding of **I/O-bound vs. CPU-bound** operations (for choosing the right approach).
- Basic knowledge of **try/catch/finally** and exception handling.

---

### Related Programming Areas

- **Task Parallel Library (TPL)** — The infrastructure underlying `Task` and PLINQ.
- **Threading** — Asynchronous programming is a higher-level abstraction over threads.
- **I/O operations** — File I/O, network calls, and database queries are the primary use case.
- **UI programming** — Async keeps UIs responsive by freeing the UI thread during awaits.
- **Web servers** — ASP.NET Core relies heavily on async to handle many concurrent requests.

---

### Core Concepts / Features

1. Asynchronous Programming
2. Synchronous vs. Asynchronous
3. Blocking vs. Non-Blocking
4. Threads
5. Tasks
6. `Task`
7. `Task<T>`
8. `ValueTask`
9. `ValueTask<T>`
10. `async` and `await`
11. Asynchronous Best Practices

---

## 1. Asynchronous Programming

### Definitions

**Core Definition**
Asynchronous programming is a programming model where operations that would otherwise block can be started, and control returned to the caller, with completion handled later.

**Technical Definition**
Asynchronous programming in C# is built on the Task-based Asynchronous Pattern (TAP). An `async` method can suspend at `await` expressions and resume when the awaited operation completes. The C# compiler rewrites `async` methods into state machines that implement this suspend/resume semantics. Asynchronous methods return task types (`Task`, `Task<TResult>`, `ValueTask`, or `ValueTask<TResult>`), and the caller can await them to get the result or observe exceptions.

**Beginner-Friendly Explanation**
Asynchronous programming is **doing other things while waiting** for slow operations like network calls or file reads.

---

### Purposes

- To keep UIs responsive during long-running operations.
- To scale web servers by freeing threads during I/O waits.
- To improve application throughput and resource utilization.
- To write non-blocking code that is easier to read than callback-based alternatives.

---

### Syntax Rules and Structure

```csharp
async Task<TResult> MethodNameAsync()
{
    // synchronous work
    var result = await SomeAsyncOperationAsync();
    // more work
    return result;
}
```

**Component breakdown**

- `async` — Marks the method as asynchronous; enables `await` inside.
- `Task<TResult>` — The return type (task type).
- `await` — Suspends until the awaited task completes.
- `Async` suffix — Convention for asynchronous method names.

#### Syntax Rules

- An `async` method must return `void`, a task type (`Task`, `Task<T>`, `ValueTask<T>`), or an asynchronous iterator type.
- `await` can only be used inside an `async` method.
- Exceptions in async methods are captured in the returned task.

#### Constraints and Limitations

- `async` is not a part of the method signature; it is an implementation detail.
- `async void` should only be used for event handlers.

---

### Annotated Code Examples and Expected Outputs

#### Example: Basic Async Method

```csharp
using System;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        Console.WriteLine("Before await");
        await Task.Delay(1000);   // non-blocking wait
        Console.WriteLine("After await");
    }
}
```

**Expected Output**
```
Before await
(1 second pause)
After await
```

**Why this output occurs**
`await Task.Delay(1000)` suspends the `Main` method without blocking the thread; the continuation runs after the delay.

---

### Real-World Cases

- **HTTP calls** — `await httpClient.GetAsync(url)`.
- **Database queries** — `await dbContext.Users.ToListAsync()`.
- **File I/O** — `await File.ReadAllTextAsync(path)`.

---

### References

- Asynchronous programming scenarios — C# — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/async-scenarios
- Task-based asynchronous programming — .NET — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/parallel-programming/task-based-asynchronous-programming

---

## 2. Synchronous vs. Asynchronous

### Definitions

**Core Definition**
**Synchronous** execution blocks the calling thread until the operation completes; **asynchronous** execution returns control to the caller before the operation completes, allowing other work to proceed.

**Technical Definition**
In synchronous execution, the calling thread waits (blocks) until the operation finishes and the result is available. In asynchronous execution, the operation is started, control returns to the caller immediately, and the caller can do other work or await the result later. Synchronous I/O blocks a thread for the entire duration of the I/O; asynchronous I/O frees the thread during the wait, allowing it to serve other work.

**Beginner-Friendly Explanation**
**Synchronous** is “stand in line and wait.” **Asynchronous** is “take a number and sit down.”

---

### Purposes

- To choose the right execution model for the workload.
- To understand why asynchronous I/O is more scalable than synchronous I/O.
- To avoid blocking threads unnecessarily.
- To write responsive and scalable applications.

---

### Syntax Rules and Structure (Comparison)

| Aspect | Synchronous | Asynchronous |
|---|---|---|
| **Thread usage** | Thread blocked during wait | Thread freed during wait |
| **Scalability** | Poor for I/O-bound work | Excellent for I/O-bound work |
| **Complexity** | Simpler to write | Requires `async`/`await` |
| **UI responsiveness** | UI freezes during long operations | UI remains responsive |

**Component breakdown**

- **Synchronous** — `Thread.Sleep`, `Task.Wait()`, `Task.Result`, synchronous file/network APIs.
- **Asynchronous** — `await Task.Delay`, `await` file/network async APIs.

#### Syntax Rules

- Synchronous methods return their result directly.
- Asynchronous methods return a task; the caller awaits it.

#### Constraints and Limitations

- Synchronous I/O on a UI thread freezes the UI.
- Synchronous I/O on a web server consumes a thread for the entire wait.

---

### Annotated Code Examples and Expected Outputs

#### Example: Synchronous vs. Asynchronous Delay

```csharp
using System;
using System.Diagnostics;
using System.Threading;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        var sw = Stopwatch.StartNew();

        // Synchronous: blocks the thread
        Thread.Sleep(1000);
        Console.WriteLine($"Synchronous: {sw.ElapsedMilliseconds} ms");

        sw.Restart();

        // Asynchronous: does not block the thread
        await Task.Delay(1000);
        Console.WriteLine($"Asynchronous: {sw.ElapsedMilliseconds} ms");
    }
}
```

**Expected Output**
```
Synchronous: 1000 ms
Asynchronous: 1000 ms
```

**Why this output occurs**
Both take the same wall-clock time, but `Thread.Sleep` blocks the thread, while `await Task.Delay` frees the thread during the wait.

---

### Real-World Cases

- **UI applications** — Use async to avoid freezing the UI.
- **Web servers** — Use async I/O to handle more concurrent requests.
- **Background services** — Use async to avoid consuming threads during waits.

---

### References

- Asynchronous programming scenarios — C# — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/async-scenarios

---

## 3. Blocking vs. Non-Blocking

### Definitions

**Core Definition**
**Blocking** means the current thread is prevented from doing other work until an operation completes; **non-blocking** means the thread is free to do other work while the operation is in progress.

**Technical Definition**
Blocking the current thread as a means of synchronously waiting for a `Task` to complete can lead to deadlocks and blocked context threads. In asynchronous programming, the goal is to **yield for tasks in a non-blocking way** — using `await` instead of `Task.Wait()` or `Task.Result`. Blocking calls reduce scalability, because each blocked thread is unavailable to perform other work.

**Beginner-Friendly Explanation**
**Blocking** is “sit and stare at the phone until it rings.” **Non-blocking** is “put the phone in your pocket and do something else until it rings.”

---

### Purposes

- To understand why blocking on async code is dangerous.
- To avoid deadlocks and thread starvation.
- To choose non-blocking alternatives (`await` instead of `.Wait()`/`.Result`).

---

### Syntax Rules and Structure (Blocking vs. Non-Blocking)

| Scenario | Blocking (avoid) | Non-Blocking (prefer) |
|---|---|---|
| Wait for a task | `Task.Wait()` / `Task.Result` | `await task` |
| Wait for any task | `Task.WaitAny()` | `await Task.WhenAny()` |
| Wait for all tasks | `Task.WaitAll()` | `await Task.WhenAll()` |
| Delay | `Thread.Sleep()` | `await Task.Delay()` |

**Component breakdown**

- **Blocking** — `Task.Wait()`, `Task.Result`, `Thread.Sleep`.
- **Non-blocking** — `await`, `Task.WhenAll`, `Task.WhenAny`, `Task.Delay`.

#### Syntax Rules

- Blocking on async code can cause deadlocks, especially on UI or ASP.NET contexts.
- `GetAwaiter().GetResult()` is the least-preferred blocking approach, preserving the original exception.

#### Constraints and Limitations

- Some legacy APIs are synchronous; blocking may be unavoidable at the boundary.

---

### Annotated Code Examples and Expected Outputs

#### Example: Blocking vs. Non-Blocking Wait

```csharp
using System;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        Task<int> task = ComputeAsync();

        // BLOCKING: blocks the current thread
        // int result = task.Result;

        // NON-BLOCKING: frees the thread
        int result = await task;
        Console.WriteLine(result);
    }

    static async Task<int> ComputeAsync()
    {
        await Task.Delay(100);
        return 42;
    }
}
```

**Expected Output**
```
42
```

**Why this output occurs**
`await task` yields the thread while waiting; `task.Result` would block the thread and could deadlock in certain contexts.

---

### Real-World Cases

- **UI event handlers** — Never use `.Result`; use `await`.
- **ASP.NET Core** — Never block on async calls; use `await` throughout.

---

### References

- Asynchronous programming scenarios — C# — Microsoft Learn (blocking guidance) - https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/async-scenarios

---

## 4. Threads

### Definitions

**Core Definition**
A thread is the smallest unit of execution within a process, representing a sequence of instructions that the operating system can schedule independently.

**Technical Definition**
A thread is a basic unit of execution within a process. In .NET, threads are managed by the CLR and scheduled by the operating system. The thread pool provides a pool of worker threads for executing tasks; the Task Parallel Library (TPL) manages the scheduling of tasks on the thread pool, handling partitioning of work, cancellation, state management, and other low-level details. Blocking a thread prevents it from executing other work; asynchronous I/O frees the thread during the wait, allowing it to serve other tasks.

**Beginner-Friendly Explanation**
A thread is a **worker**. The thread pool is a **team of workers**. Blocking a thread is keeping a worker idle; async frees the worker to do other jobs.

---

### Purposes

- To understand the execution infrastructure underlying async and tasks.
- To appreciate why blocking is expensive (it wastes a worker).
- To understand thread-pool scheduling and its implications.

---

### Syntax Rules and Structure

Threads are typically used indirectly through tasks and async/await:

```csharp
// Direct thread (rarely needed)
var thread = new Thread(() => Console.WriteLine("Running"));
thread.Start();

// Preferred: thread pool via Task
await Task.Run(() => Console.WriteLine("Running on thread pool"));
```

**Component breakdown**

- `Thread` — Manual thread creation.
- `ThreadPool` — Pool of reusable worker threads.
- `Task.Run` — Queues work to the thread pool.

#### Syntax Rules

- The thread pool reuses threads to avoid the cost of thread creation.
- Blocking a thread-pool thread reduces the pool's ability to service other work.

#### Constraints and Limitations

- Creating a new `Thread` for every operation is expensive; prefer `Task.Run` or async APIs.

---

### Annotated Code Examples and Expected Outputs

#### Example: Thread Pool via `Task.Run`

```csharp
using System;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        await Task.Run(() =>
        {
            Console.WriteLine($"Running on thread {Environment.CurrentManagedThreadId}");
        });
    }
}
```

**Expected Output (thread ID varies)**
```
Running on thread 5
```

**Why this output occurs**
`Task.Run` schedules the delegate on a thread-pool thread; the `await` ensures the main method waits for completion.

---

### Real-World Cases

- **CPU-bound work** — Offload to the thread pool with `Task.Run`.
- **I/O-bound work** — Use async APIs; do not occupy a thread during the wait.

---

### References

- Task Parallel Library (TPL) — .NET — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/parallel-programming/task-parallel-library-tpl

---

## 5. Tasks

### Definitions

**Core Definition**
A task is an abstraction representing an asynchronous operation that may complete in the future, possibly with a result or an exception.

**Technical Definition**
The Task Parallel Library (TPL) is based on the concept of a **task**, which represents an asynchronous operation. Tasks are the preferred API for writing multi-threaded, asynchronous, and parallel code in .NET. A task can be in one of several states (incomplete, succeeded, canceled, faulted) and can be awaited, combined (`WhenAll`, `WhenAny`), or continued. Tasks are scheduled on the thread pool by default.

**Beginner-Friendly Explanation**
A task is a **promise** that work is happening and will produce a result (or an error) later.

---

### Purposes

- To represent asynchronous operations uniformly.
- To compose multiple asynchronous operations (`WhenAll`, `WhenAny`).
- To propagate results and exceptions.
- To support cancellation and progress reporting.

---

### Syntax Rules and Structure

```csharp
// Start a task
Task task = Task.Run(() => DoWork());

// Await a task
await task;

// Combine tasks
await Task.WhenAll(task1, task2);
await Task.WhenAny(task1, task2);
```

**Component breakdown**

- `Task` — Non-generic task (no result).
- `Task<TResult>` — Task with a result.
- `Task.Run` — Queues a delegate to the thread pool.
- `Task.WhenAll` / `Task.WhenAny` — Combinators.

#### Syntax Rules

- Tasks are awaitable.
- `Task.WhenAll` completes when all tasks complete; `Task.WhenAny` completes when any task completes.
- Faulted tasks store exceptions that are rethrown on `await`.

#### Constraints and Limitations

- `Task.WhenAll` returns the unwrapped results of all successfully completed tasks; `Task.WhenAny` returns the completed task.

---

### Annotated Code Examples and Expected Outputs

#### Example: `Task.WhenAll` and `Task.WhenAny`

```csharp
using System;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        Task<int> t1 = Task.FromResult(1);
        Task<int> t2 = Task.FromResult(2);

        // WhenAll: wait for all
        int[] results = await Task.WhenAll(t1, t2);
        Console.WriteLine(string.Join(", ", results));   // 1, 2

        // WhenAny: wait for the first
        Task<int> first = await Task.WhenAny(t1, t2);
        Console.WriteLine(await first);   // 1 (or 2, whichever completed first)
    }
}
```

**Expected Output**
```
1, 2
1
```

**Why this output occurs**
`WhenAll` waits for all tasks and returns their results; `WhenAny` returns the first completed task.

---

### Real-World Cases

- **Parallel downloads** — `Task.WhenAll` for multiple HTTP requests.
- **Timeouts** — `Task.WhenAny` with `Task.Delay` to implement timeouts.
- **Fire-and-forget** — `Task.Run` for background work (with caution).

---

### References

- Task-based asynchronous programming — .NET — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/parallel-programming/task-based-asynchronous-programming
- Consuming the Task-based Asynchronous Pattern — .NET — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/consuming-the-task-based-asynchronous-pattern

---

## 6. `Task`

### Definitions

**Core Definition**
`Task` is the non-generic task type representing an asynchronous operation that does not produce a result.

**Technical Definition**
`System.Threading.Tasks.Task` represents a single asynchronous operation that does not return a value. It is used for asynchronous methods that perform work but do not produce a result, analogous to a `void`-returning synchronous method. `Task` can be awaited, combined, and inspected for status (`IsCompleted`, `IsFaulted`, `IsCanceled`).

**Beginner-Friendly Explanation**
`Task` is a **promise of work** that produces no result — just completion or failure.

---

### Purposes

- To represent asynchronous operations without a return value.
- To support awaiting and exception propagation.
- To enable composition (`WhenAll`, `WhenAny`).

---

### Syntax Rules and Structure

```csharp
async Task DoWorkAsync()
{
    await Task.Delay(100);
}
```

**Component breakdown**

- `Task` — Return type for async methods without a result.
- `await` — Suspends until the task completes.

#### Syntax Rules

- `Task` is a reference type.
- `await task` rethrows any exceptions stored in the task.

#### Constraints and Limitations

- `Task` allocates on the heap; use `ValueTask` when appropriate.

---

### Annotated Code Examples and Expected Outputs

#### Example: `Task` Without a Result

```csharp
using System;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        await DoWorkAsync();
        Console.WriteLine("Done");
    }

    static async Task DoWorkAsync()
    {
        await Task.Delay(100);
    }
}
```

**Expected Output**
```
Done
```

**Why this output occurs**
`DoWorkAsync` returns a `Task` that completes after the delay; `await` waits for completion.

---

### Real-World Cases

- **Saving a file asynchronously** — `await File.WriteAllTextAsync(...)` returns a `Task`.
- **Sending an email** — `await emailSender.SendAsync(...)`.

---

### References

- Task Class — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.task

---

## 7. `Task<T>`

### Definitions

**Core Definition**
`Task<TResult>` is the generic task type representing an asynchronous operation that produces a result of type `TResult`.

**Technical Definition**
`System.Threading.Tasks.Task<TResult>` represents a single operation that returns a value and usually executes asynchronously. It derives from `Task` and adds a `Result` property. `Task<TResult>` objects are central to TAP. The result is available after the task completes; accessing `Result` before completion blocks the calling thread and should be avoided in favor of `await`.

**Beginner-Friendly Explanation**
`Task<T>` is a **promise of a value** that will be available later.

---

### Purposes

- To represent asynchronous operations that return a value.
- To enable `await` to extract the result.
- To support composition and exception propagation.

---

### Syntax Rules and Structure

```csharp
async Task<int> GetValueAsync()
{
    await Task.Delay(100);
    return 42;
}

int value = await GetValueAsync();   // value == 42
```

**Component breakdown**

- `Task<int>` — Return type for async methods that produce an `int`.
- `await` — Extracts the result.

#### Syntax Rules

- `Task<T>` is a reference type.
- `await task` returns the `T` result or throws the stored exception.

#### Constraints and Limitations

- Accessing `.Result` blocks the thread; prefer `await`.

---

### Annotated Code Examples and Expected Outputs

#### Example: `Task<T>` with a Result

```csharp
using System;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        int result = await GetValueAsync();
        Console.WriteLine(result);   // 42
    }

    static async Task<int> GetValueAsync()
    {
        await Task.Delay(100);
        return 42;
    }
}
```

**Expected Output**
```
42
```

**Why this output occurs**
`await GetValueAsync()` waits for the task to complete and returns the result `42`.

---

### Real-World Cases

- **Database queries** — `await db.Users.FindAsync(id)` returns `Task<User>`.
- **HTTP responses** — `await httpClient.GetStringAsync(url)` returns `Task<string>`.

---

### References

- Task<TResult> Class — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.task-1

---

## 8. `ValueTask`

### Definitions

**Core Definition**
`ValueTask` is a **struct-based** awaitable type that represents an asynchronous operation without a result, designed to avoid heap allocations when the operation completes synchronously.

**Technical Definition**
`System.Threading.Tasks.ValueTask` is a readonly struct that provides an awaitable result of an asynchronous operation. It can wrap either a `Task` or an `IValueTaskSource` instance. A `ValueTask` instance may only be awaited once, and consumers may not call `GetAwaiter()` until the instance has completed. If these limitations are unacceptable, convert the `ValueTask` to a `Task` by calling `AsTask`. Returning a `ValueTask` that wraps an `IValueTaskSource` enables high-throughput applications to avoid allocations by using a pool of reusable `IValueTaskSource` objects.

**Beginner-Friendly Explanation**
`ValueTask` is a **lightweight promise** that avoids allocating a heap object when the operation completes immediately.

---

### Purposes

- To reduce allocations for async methods that often complete synchronously.
- To support high-throughput scenarios with pooling (`IValueTaskSource`).
- To optimize performance-critical paths.

---

### Syntax Rules and Structure

```csharp
async ValueTask DoWorkAsync()
{
    if (cache.TryGetValue(key, out var value))
        return;   // completes synchronously, no allocation
    await LoadAsync(key);
}
```

**Component breakdown**

- `ValueTask` — Struct-based awaitable.
- `AsTask()` — Converts to `Task` for multiple awaits.

#### Syntax Rules

- A `ValueTask` may only be awaited once.
- `GetAwaiter()` may only be called after completion.
- Convert to `Task` with `AsTask()` if multiple consumption is needed.

#### Constraints and Limitations

- Using `ValueTask` introduces overhead (copying more data than a single `Task` reference).
- Only use after performance analysis proves it worthwhile.

---

### Annotated Code Examples and Expected Outputs

#### Example: `ValueTask` for Synchronous Completion

```csharp
using System;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        await DoWorkAsync();
        Console.WriteLine("Done");
    }

    static ValueTask DoWorkAsync()
    {
        // Completes synchronously — no allocation
        return default;
    }
}
```

**Expected Output**
```
Done
```

**Why this output occurs**
`DoWorkAsync` returns a `default` `ValueTask`, which completes synchronously without allocating a `Task`.

---

### Real-World Cases

- **Caching** — Async methods that often return cached results.
- **High-throughput pipelines** — `IValueTaskSource` pooling.

---

### References

- ValueTask Struct — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.valuetask

---

## 9. `ValueTask<T>`

### Definitions

**Core Definition**
`ValueTask<TResult>` is the **generic struct-based** awaitable type representing an asynchronous operation that produces a result of type `TResult`, designed to avoid heap allocations when possible.

**Technical Definition**
`System.Threading.Tasks.ValueTask<TResult>` is a readonly struct that provides an awaitable result of an asynchronous operation that returns a value. Like `ValueTask`, it may only be awaited once and should be converted to `Task<TResult>` with `AsTask()` if multiple consumption is needed. The standard library types `Task<TResult>` and `ValueTask<TResult>` are both task types; `ValueTask<TResult>` is used when allocation reduction is important.

**Beginner-Friendly Explanation**
`ValueTask<T>` is a **lightweight promise of a value** that avoids heap allocation when the result is already available.

---

### Purposes

- To reduce allocations for async methods returning values that often complete synchronously.
- To optimize performance-critical paths.
- To support pooled `IValueTaskSource<T>` implementations.

---

### Syntax Rules and Structure

```csharp
async ValueTask<int> GetValueAsync()
{
    if (cache.TryGetValue(key, out var value))
        return value;   // synchronous completion
    return await LoadAsync(key);
}
```

**Component breakdown**

- `ValueTask<int>` — Struct-based awaitable returning `int`.
- `await` — Extracts the result.

#### Syntax Rules

- May only be awaited once.
- Use `AsTask()` for multiple consumption.

#### Constraints and Limitations

- Same single-await restriction as `ValueTask`.
- Overhead of copying struct fields.

---

### Annotated Code Examples and Expected Outputs

#### Example: `ValueTask<T>` with Synchronous Result

```csharp
using System;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        int value = await GetValueAsync();
        Console.WriteLine(value);   // 42
    }

    static ValueTask<int> GetValueAsync()
    {
        return new ValueTask<int>(42);   // no allocation
    }
}
```

**Expected Output**
```
42
```

**Why this output occurs**
`GetValueAsync` returns a `ValueTask<int>` wrapping the value `42` without allocating a `Task<int>`.

---

### Real-World Cases

- **Cached async lookups** — Return cached values without allocation.
- **High-performance libraries** — Use `IValueTaskSource<T>` for pooled async operations.

---

### References

- ValueTask<TResult> Struct — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.valuetask-1

---

## 10. `async` and `await`

### Definitions

**Core Definition**
`async` and `await` are C# keywords that enable asynchronous programming: `async` marks a method as asynchronous, and `await` suspends the method until an awaited task completes, without blocking the thread.

**Technical Definition**
The `async` modifier marks a method as an asynchronous function. The `await` operator suspends evaluation of the enclosing async method until the asynchronous operation represented by its operand completes. When the operation completes, the `await` operator returns the result of the operation, if any. The C# compiler transforms `async` methods into **state machines** that implement the suspend/resume semantics. The return type of an async method must be `void`, a task type (`Task`, `Task<T>`, `ValueTask<T>`), or an asynchronous iterator type. A method returning a task type is said to be **task-returning**.

**Beginner-Friendly Explanation**
`async` says **“this method can pause.”** `await` says **“pause here until this finishes.”**

---

### Purposes

- To write asynchronous code that reads like synchronous code.
- To avoid callbacks and manual continuations.
- To keep UIs responsive and servers scalable.
- To simplify exception handling in asynchronous code.

---

### Syntax Rules and Structure

```csharp
async Task<int> GetDataAsync()
{
    var data = await FetchDataAsync();
    return data.Length;
}
```

**Component breakdown**

- `async` — Modifier on the method.
- `Task<int>` — Return type.
- `await` — Suspension point.

#### Syntax Rules

- `await` can only be used in an `async` method.
- The return type must be `void`, a task type, or an asynchronous iterator type.
- Exceptions are captured in the returned task.
- `async` methods run synchronously until the first `await`.

#### Constraints and Limitations

- `async void` should only be used for event handlers.
- `await` cannot be used in a `catch` or `finally` block prior to C# 6 (allowed in C# 6+).

---

### Annotated Code Examples and Expected Outputs

#### Example: `async` and `await` in Practice

```csharp
using System;
using System.Net.Http;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        using var client = new HttpClient();
        string content = await client.GetStringAsync("https://example.com");
        Console.WriteLine($"Length: {content.Length}");
    }
}
```

**Expected Output (approximate)**
```
Length: 1256
```

**Why this output occurs**
`await client.GetStringAsync(...)` suspends `Main` until the HTTP request completes, then returns the response body.

---

### Real-World Cases

- **HTTP calls** — `await httpClient.GetAsync(url)`.
- **Database queries** — `await dbContext.Users.ToListAsync()`.
- **File I/O** — `await File.ReadAllTextAsync(path)`.

---

### References

- Asynchronous programming scenarios — C# — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/async-scenarios
- Asynchronous return types — C# — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/async-return-types

---

## 11. Asynchronous Best Practices

### Definitions

**Core Definition**
Asynchronous best practices are the guidelines that ensure async code is **correct, scalable, and maintainable**, avoiding deadlocks, unnecessary blocking, and hidden pitfalls.

**Technical Definition**
Best practices for async programming derive from Microsoft's guidance. Key recommendations include: **avoid blocking calls** (`Task.Wait`, `Task.Result`, `Thread.Sleep`); **use `async`/`await` throughout the call stack**; **avoid `async void` except for event handlers**; **use `ConfigureAwait(false)` in library code**; **use `ValueTask` only when profiling shows allocation pressure**; **write stateless code** that depends only on return values; **use `Task.WhenAll` and `Task.WhenAny` for composition**; and **use `GetAwaiter().GetResult()` only as a last resort for synchronous blocking**.

**Beginner-Friendly Explanation**
The best practices are: **do not block**, **await everywhere**, **avoid `async void`**, and **do not over-engineer**.

---

### Guidelines and Structure

#### General Guidelines

```csharp
// 1. Avoid blocking calls
// BAD: task.Wait(); task.Result; Thread.Sleep();
// GOOD: await task; await Task.Delay();

// 2. Use async/await throughout the call stack
// BAD: sync method calling async method and blocking
// GOOD: async all the way

// 3. Avoid async void except for event handlers
// BAD: async void DoWork() { ... }
// GOOD: async Task DoWorkAsync() { ... }

// 4. Use ConfigureAwait(false) in library code
await SomeAsyncOperation().ConfigureAwait(false);

// 5. Use ValueTask only when profiling shows allocation pressure
// Prefer Task by default.

// 6. Use Task.WhenAll / Task.WhenAny for composition
await Task.WhenAll(task1, task2);
await Task.WhenAny(task1, task2);

// 7. Write stateless code
// Depend only on return values, not on global state or method execution.
```

**Component breakdown**

- **No blocking** — `await` instead of `.Wait()`/`.Result`.
- **Async all the way** — Avoid mixing sync and async.
- **`async void` only for event handlers** — Exceptions cannot be caught by the caller.
- **`ConfigureAwait(false)`** — Avoids capturing the synchronization context in library code.
- **`ValueTask` sparingly** — Only after profiling.
- **Composition** — `WhenAll`/`WhenAny`.
- **Stateless code** — Easier to reason about and test.

#### Rules and Constraints

- **Do** use `async`/`await` throughout.
- **Do** use `ConfigureAwait(false)` in library code.
- **Do** use `Task.WhenAll` and `Task.WhenAny` for composition.
- **Do not** block on async code.
- **Do not** use `async void` outside event handlers.
- **Do not** use `ValueTask` without performance justification.

---

### Annotated Code Examples and Expected Outputs

#### Example: Correct Async Usage

```csharp
using System;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        await DoWorkAsync();
    }

    static async Task DoWorkAsync()
    {
        await Task.Delay(100).ConfigureAwait(false);
        Console.WriteLine("Done");
    }
}
```

**Expected Output**
```
Done
```

**Why this output occurs**
`ConfigureAwait(false)` avoids capturing the synchronization context; `await` is used throughout instead of blocking.

---

### Real-World Cases

- **Library code** — Use `ConfigureAwait(false)` to avoid context capture.
- **ASP.NET Core** — Use async all the way; never block.
- **UI applications** — Use `async void` only for event handlers; use `async Task` elsewhere.

---

### References

- Asynchronous programming scenarios — C# — Microsoft Learn (best practices) - https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/async-scenarios
- Async/Await best practices — Microsoft Learn (video) - https://learn.microsoft.com/en-us/shows/on-dotnet/brandon-minnick-async-await-best-practices
- ConfigureAwait FAQ — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca2007

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| `async`/`await` | Introduced in **C# 5.0 / .NET Framework 4.5**; stable. |
| `ValueTask` | Introduced in **.NET Core 2.0**; use only after profiling. |
| `ValueTask<T>` | Introduced in **.NET Core 2.1**; single-await restriction. |
| `async void` | **Only for event handlers**; exceptions cannot be caught by the caller. |
| `ConfigureAwait(false)` | **Recommended in library code**; avoids context capture. |
| Blocking on async (`Task.Wait`, `Task.Result`) | **Can cause deadlocks**; avoid. |
| `async` streams (`IAsyncEnumerable<T>`) | Introduced in **C# 8.0**; not covered here. |
| `Task.WhenAll` / `Task.WhenAny` | Core TAP combinators; stable. |

---

## References (Consolidated)

- Asynchronous programming scenarios — C# — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/async-scenarios
- Task-based asynchronous programming — .NET — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/parallel-programming/task-based-asynchronous-programming
- Task Parallel Library (TPL) — .NET — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/parallel-programming/task-parallel-library-tpl
- Consuming the Task-based Asynchronous Pattern — .NET — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/consuming-the-task-based-asynchronous-pattern
- Task<TResult> Class — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.task-1
- ValueTask Struct — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.valuetask
- ValueTask<TResult> Struct — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.valuetask-1
- C# async and await: A Deeper Dive — Microsoft Learn (archive) - https://learn.microsoft.com/en-us/archive/blogs/cdndevs/c-async-and-await-a-deeper-dive
- Brandon Minnick — Async/Await Best Practices — Microsoft Learn (video) - https://learn.microsoft.com/en-us/shows/on-dotnet/brandon-minnick-async-await-best-practices
- ConfigureAwait FAQ — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca2007
- Asynchronous return types — C# — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/async-return-types

---