# C# Task-Based Asynchronous Pattern (TAP) — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
The **Task-based Asynchronous Pattern (TAP)** is the recommended asynchronous design pattern in .NET for new development. It uses a single method to represent the **initiation and completion** of an asynchronous operation, returning a `Task`, `Task<TResult>`, `ValueTask`, or `ValueTask<TResult>` object that represents the operation's eventual completion.

**Technical Definition**
TAP is based on the `System.Threading.Tasks.Task` and `Task<TResult>` types in the `System.Threading.Tasks` namespace. TAP methods include the `Async` suffix after the operation name for methods that return awaitable types. TAP methods return either a `Task` (for operations corresponding to `void`-returning synchronous methods) or a `Task<TResult>` (for operations corresponding to `TResult`-returning synchronous methods). The parameters of a TAP method should match the parameters of its synchronous counterpart and be provided in the same order; `out` and `ref` parameters are exempt from this rule and should be avoided entirely. Methods devoted exclusively to the creation, manipulation, or combination of tasks (combinators such as `Task.WhenAll` and `Task.WhenAny`) need not follow the naming pattern.

**Beginner-Friendly Explanation**
TAP is **the modern way to write asynchronous code in C#**. Instead of the old `Begin`/`End` pattern (APM) or event-based pattern (EAP), TAP uses a single method that returns a **promise**—a `Task` object. You call the method, get back a `Task`, and either `await` it (which pauses your method without blocking the thread) or attach a continuation to run when it completes. The `async` and `await` keywords are built on top of TAP, making asynchronous code look almost like synchronous code.

---

### Key Characteristics

- **Single-method model** — One method represents both initiation and completion, unlike APM (requires `Begin`/`End`) or EAP (requires events).
- **Task return types** — Methods return `Task`, `Task<TResult>`, `ValueTask`, or `ValueTask<TResult>`.
- **`Async` suffix** — TAP methods use the `Async` suffix (or `TaskAsync` if an `Async`-suffixed EAP method already exists).
- **Parameter parity** — TAP method parameters match the synchronous counterpart; `out`/`ref` parameters are avoided and data is returned via `TResult`.
- **Cancellation support** — Consider adding a `CancellationToken` parameter even if the synchronous counterpart doesn't have one.
- **Combinators** — `Task.WhenAll` and `Task.WhenAny` compose multiple tasks into a single awaitable task.

---

### Prerequisites

- Basic understanding of C# methods, return types, and exceptions.
- Familiarity with `async` and `await` keywords.
- Awareness of the `System.Threading.Tasks` namespace.
- Understanding of I/O-bound vs. CPU-bound operations.
- Basic knowledge of delegates and lambda expressions (for continuations).

---

### Related Programming Areas

- **Async/await** — Language-level support built on TAP continuations.
- **Task Parallel Library (TPL)** — The infrastructure underlying TAP.
- **Asynchronous programming patterns** — TAP supersedes APM and EAP.
- **Cancellation** — `CancellationToken` integration for cooperative cancellation.
- **I/O-bound operations** — File, network, and database operations.

---

### Core Concepts / Features

1. TAP — Overarching Concept
2. Task Creation
3. Task Continuation
4. Task Composition
5. `Task.WhenAll()`
6. `Task.WhenAny()`
7. `Task.Delay()`
8. `Task.FromResult()`
9. `Task.CompletedTask`
10. TAP Best Practices

---

## 1. TAP — Overarching Concept

### Definitions

**Core Definition**
TAP is the recommended asynchronous design pattern in .NET, using a single method returning an awaitable task to represent an asynchronous operation.

**Technical Definition**
In .NET, the task-based asynchronous pattern is the recommended asynchronous design pattern for new development. It's based on the `Task` and `Task<TResult>` types, which represent asynchronous operations. TAP uses a single method to represent the initiation and completion of an asynchronous operation, contrasting with APM (which requires `Begin` and `End` methods) and EAP (which requires an `Async` suffix, events, event handler delegate types, and `EventArgs`-derived types).

**Beginner-Friendly Explanation**
TAP is **“one method, one task, one await.”** You call `GetDataAsync()`, get back a `Task<string>`, and `await` it to get the result.

---

### Purposes

- To provide a **unified, modern asynchronous programming model**.
- To enable **composition** of asynchronous operations via combinators.
- To support **cancellation** and **progress reporting** in a standard way.
- To be the foundation for `async`/`await` language support.

---

### Syntax Rules and Structure

```csharp
// TAP method signature pattern
public Task<TResult> OperationNameAsync(parameters, CancellationToken cancellationToken = default)
{
    // implementation
}
```

- Return type: `Task` (no result) or `Task<TResult>` (with result).
- Naming: `OperationNameAsync` (or `OperationNameTaskAsync` if `Async` is already taken).
- Parameters: match synchronous counterpart; avoid `out`/`ref`.
- Optional `CancellationToken` parameter.

#### Constraints and Limitations

- TAP methods should return **active** tasks (callers shouldn't need to call `Start`).
- In most TAP code, **do not dispose tasks**; tasks don't hold unmanaged resources in the typical case.

---

### Annotated Code Examples and Expected Outputs

#### Example: Basic TAP Method

```csharp
using System;
using System.Threading;
using System.Threading.Tasks;

public class DataService
{
    public async Task<string> GetDataAsync(string url, CancellationToken cancellationToken = default)
    {
        using var client = new HttpClient();
        return await client.GetStringAsync(url, cancellationToken);
    }
}

class Program
{
    static async Task Main()
    {
        var service = new DataService();
        string data = await service.GetDataAsync("https://example.com");
        Console.WriteLine($"Length: {data.Length}");
    }
}
```

**Expected Output (approximate)**
```
Length: 1256
```

**Why this output occurs**
`GetDataAsync` returns a `Task<string>`; `await` suspends `Main` until the HTTP request completes, then returns the response body length.

---

### Real-World Cases

- **ASP.NET Core controllers** — `Task<IActionResult>` methods.
- **Repository methods** — `Task<User> GetUserAsync(int id)`.
- **HTTP clients** — `Task<HttpResponseMessage> GetAsync(string url)`.

---

### References

- Task-based asynchronous pattern (TAP) in .NET: Introduction and overview — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/task-based-asynchronous-pattern-tap

---

## 2. Task Creation

### Definitions

**Core Definition**
Task creation is the process of producing a `Task` or `Task<TResult>` instance that represents an asynchronous operation, either by calling an async method, using `Task.Run`, `Task.Factory.StartNew`, `Task.FromResult`, or a `TaskCompletionSource`.

**Technical Definition**
Tasks can be created in several ways: (1) **compiler-generated** — any method marked with `async` is transformed into a TAP method returning a task; (2) **`Task.Run`** — queues work to the thread pool (preferred for CPU-bound work); (3) **`Task.Factory.StartNew`** — creates and starts a task with custom `TaskCreationOptions` and `TaskScheduler`; (4) **`Task.FromResult`** — returns a completed task with a result; (5) **`Task.CompletedTask`** — returns a cached successfully completed task; (6) **`TaskCompletionSource<T>`** — manually controls task completion.

**Beginner-Friendly Explanation**
You create tasks by **calling async methods** (the compiler builds the task for you), by **offloading work** with `Task.Run`, or by **returning pre-completed tasks** with `Task.FromResult`/`Task.CompletedTask`.

---

### Purposes

- To initiate asynchronous operations.
- To offload CPU-bound work to the thread pool.
- To return completed tasks from synchronous code paths.
- To bridge callback-based APIs to TAP.

---

### Syntax Rules and Structure

```csharp
// Compiler-generated (async method)
public async Task<int> GetValueAsync() { await Task.Delay(100); return 42; }

// Task.Run (CPU-bound offload)
Task<int> task = Task.Run(() => ComputeIntensive());

// Task.Factory.StartNew (advanced options)
Task task2 = Task.Factory.StartNew(() => DoWork(), TaskCreationOptions.LongRunning);

// Task.FromResult (completed with result)
Task<int> completed = Task.FromResult(42);

// Task.CompletedTask (completed without result)
Task completedVoid = Task.CompletedTask;

// TaskCompletionSource<T> (manual completion)
var tcs = new TaskCompletionSource<int>();
tcs.SetResult(42);
Task<int> fromTcs = tcs.Task;
```

#### Constraints and Limitations

- `Task.Run` should be used for CPU-bound work; use native async APIs for I/O-bound work.
- `Task.Factory.StartNew` with async lambdas returns `Task<Task>`; call `.Unwrap()` or use `Task.Run`.
- `Task.FromResult` may return a cached singleton in .NET 6+; do not rely on reference equality.

---

### Annotated Code Examples and Expected Outputs

#### Example: Different Task Creation Approaches

```csharp
using System;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        // Compiler-generated async method
        Task<int> t1 = GetValueAsync();

        // Task.Run for CPU-bound work
        Task<int> t2 = Task.Run(() => 21 * 2);

        // Task.FromResult for pre-computed result
        Task<int> t3 = Task.FromResult(42);

        // Task.CompletedTask for completed void task
        Task t4 = Task.CompletedTask;

        int[] results = await Task.WhenAll(t1, t2, t3);
        Console.WriteLine(string.Join(", ", results));   // 42, 42, 42
        Console.WriteLine(t4.IsCompleted);               // True
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
42, 42, 42
True
```

**Why this output occurs**
All four task creation methods produce tasks that complete with the value `42` (or complete successfully for `Task.CompletedTask`).

---

### Real-World Cases

- **Offloading CPU-bound calculations** — `Task.Run` in a UI application.
- **Returning cached results** — `Task.FromResult` in an async repository.
- **Implementing `Task`-returning interfaces** — `Task.CompletedTask` for no-op paths.
- **Wrapping event-based APIs** — `TaskCompletionSource<T>`.

---

### References

- Implementing the Task-based Asynchronous Pattern — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/implementing-the-task-based-asynchronous-pattern
- Task.Run Method — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.task.run
- Task.FromResult<TResult>(TResult) Method — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.task.fromresult
- Task.CompletedTask Property — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.task.completedtask

---

## 3. Task Continuation

### Definitions

**Core Definition**
A task continuation is an asynchronous task that is **invoked by another task (the antecedent) when the antecedent finishes**, either explicitly via `Task.ContinueWith` or implicitly via `await`.

**Technical Definition**
A continuation task (also known just as a continuation) is an asynchronous task that's invoked by another task, known as the antecedent, when the antecedent finishes. You create a continuation that executes when its antecedent has completed by calling the `Task.ContinueWith` method. The continuation is scheduled to run when the antecedent completes, regardless of whether it completed successfully, faulted, or was canceled. Alternatively, language support built on top of continuations (`await` in C#) provides an implicit continuation model.

**Beginner-Friendly Explanation**
A continuation is **“when this finishes, do that.”** You can chain tasks together so that task B runs after task A completes.

---

### Purposes

- To chain asynchronous operations sequentially.
- To run cleanup logic after a task completes.
- To compose multi-step asynchronous pipelines.
- To observe completion without blocking.

---

### Syntax Rules and Structure

```csharp
// Explicit continuation
Task<int> antecedent = GetValueAsync();
Task continuation = antecedent.ContinueWith(t =>
{
    Console.WriteLine($"Result: {t.Result}");
});

// Implicit continuation (await)
int result = await GetValueAsync();
Console.WriteLine($"Result: {result}");
```

- `ContinueWith` accepts an `Action<Task>` or `Func<Task, TResult>`.
- `TaskContinuationOptions` controls when the continuation runs (e.g., `OnlyOnRanToCompletion`, `NotOnCanceled`).
- `await` is the preferred, compiler-generated continuation mechanism.

#### Constraints and Limitations

- `ContinueWith` runs on the thread pool by default; use `TaskScheduler.FromCurrentSynchronizationContext()` to run on the UI thread.
- `ContinueWith` does not automatically unwrap nested tasks; use `Unwrap()` if needed.
- `await` is generally simpler and less error-prone than `ContinueWith`.

---

### Annotated Code Examples and Expected Outputs

#### Example: Explicit and Implicit Continuations

```csharp
using System;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        // Explicit continuation
        Task<int> antecedent = Task.Run(() => 42);
        Task explicitCont = antecedent.ContinueWith(t =>
            Console.WriteLine($"Explicit: {t.Result}"));

        await explicitCont;   // wait for continuation

        // Implicit continuation via await
        int result = await Task.Run(() => 42);
        Console.WriteLine($"Implicit: {result}");
    }
}
```

**Expected Output**
```
Explicit: 42
Implicit: 42
```

**Why this output occurs**
`ContinueWith` schedules a continuation that runs when the antecedent completes; `await` provides the same effect with simpler syntax.

---

### Real-World Cases

- **Sequential async steps** — Download → parse → save.
- **Cleanup after operations** — `ContinueWith` for logging or resource release.
- **Timeout handling** — `Task.WhenAny` with a delay task.

---

### References

- Chaining tasks using continuation tasks — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/parallel-programming/chaining-tasks-by-using-continuation-tasks
- Task.ContinueWith Method — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.task.continuewith

---

## 4. Task Composition

### Definitions

**Core Definition**
Task composition is the process of **combining multiple tasks into a single task** that represents the completion of all or some of the original tasks, using combinators such as `Task.WhenAll` and `Task.WhenAny`.

**Technical Definition**
The `Task` class provides methods for composing multiple tasks into a single task. The two primary combinators are `Task.WhenAll` (returns a task that completes when **all** tasks in a collection have completed) and `Task.WhenAny` (returns a task that completes when **any** task in a collection has completed). These combinators enable orchestration of concurrent operations without blocking threads.

**Beginner-Friendly Explanation**
Composition is **“combine many tasks into one.”** `WhenAll` waits for everything; `WhenAny` waits for the first to finish.

---

### Purposes

- To run multiple asynchronous operations concurrently.
- To wait for all operations to complete (`WhenAll`).
- To react to the first completed operation (`WhenAny`).
- To implement timeouts and redundancy patterns.

---

### Syntax Rules and Structure

```csharp
// WhenAll: wait for all tasks
TResult[] results = await Task.WhenAll(task1, task2, task3);

// WhenAny: wait for any task
Task<TResult> firstCompleted = await Task.WhenAny(task1, task2, task3);
```

- `WhenAll` returns an array of results (for `Task<T>`) or a `Task` (for non-generic tasks).
- `WhenAny` returns the first completed `Task` (or `Task<T>`).

#### Constraints and Limitations

- `WhenAll` throws the first exception if any task faults; examine all tasks for exceptions if needed.
- `WhenAny` does not automatically observe exceptions in the non-completed tasks.

---

### Annotated Code Examples and Expected Outputs

#### Example: WhenAll and WhenAny

```csharp
using System;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        Task<int> t1 = Task.FromResult(1);
        Task<int> t2 = Task.FromResult(2);

        // WhenAll
        int[] results = await Task.WhenAll(t1, t2);
        Console.WriteLine(string.Join(", ", results));   // 1, 2

        // WhenAny
        Task<int> first = await Task.WhenAny(t1, t2);
        Console.WriteLine(await first);                   // 1 (or 2, whichever is first)
    }
}
```

**Expected Output**
```
1, 2
1
```

**Why this output occurs**
`WhenAll` waits for both tasks and returns their results; `WhenAny` returns the first completed task.

---

### Real-World Cases

- **Parallel downloads** — `Task.WhenAll` for multiple HTTP requests.
- **Timeouts** — `Task.WhenAny` with `Task.Delay`.
- **Redundancy** — Query multiple services and use the fastest response.

---

### References

- Consuming the Task-based Asynchronous Pattern — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/consuming-the-task-based-asynchronous-pattern

---

## 5. `Task.WhenAll()`

### Definitions

**Core Definition**
`Task.WhenAll()` creates a task that completes when **all** tasks in a collection have completed.

**Technical Definition**
`Task.WhenAll` returns a task that completes when all tasks in a collection are complete. For `Task<TResult>[]`, it returns a `Task<TResult[]>` containing the unwrapped results of all successfully completed tasks. If any task faults, the returned task faults with the first exception (though all exceptions are accessible via the individual tasks).

**Beginner-Friendly Explanation**
`WhenAll` is **“wait for everyone to finish.”** It gives you an array of all results.

---

### Purposes

- To wait for multiple concurrent operations to complete.
- To aggregate results from multiple tasks.
- To implement fan-out/fan-in concurrency patterns.

---

### Syntax Rules and Structure

```csharp
Task<TResult[]> all = Task.WhenAll(task1, task2, task3);
TResult[] results = await all;
```

- Accepts `IEnumerable<Task<TResult>>` or `params Task<TResult>[]`.
- Returns `Task<TResult[]>` (or `Task` for non-generic tasks).

#### Constraints and Limitations

- If any task faults, `WhenAll` throws the first exception; use `task.Exception` on each task to see all exceptions.
- Tasks are not canceled automatically if one fails.

---

### Annotated Code Examples and Expected Outputs

#### Example: Downloading Multiple Pages Concurrently

```csharp
using System;
using System.Linq;
using System.Net.Http;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        string[] urls = { "https://example.com", "https://example.org" };
        using var client = new HttpClient();

        Task<string>[] tasks = urls.Select(url => client.GetStringAsync(url)).ToArray();
        string[] pages = await Task.WhenAll(tasks);

        Console.WriteLine($"Downloaded {pages.Length} pages");
        // Downloaded 2 pages
    }
}
```

**Expected Output**
```
Downloaded 2 pages
```

**Why this output occurs**
`WhenAll` waits for both downloads and returns their contents in order.

---

### Real-World Cases

- **Parallel data fetching** — Multiple API calls.
- **Batch processing** — Running independent work items concurrently.
- **Fan-out/fan-in** — Spreading work across tasks and aggregating results.

---

### References

- Task.WhenAll Method — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.task.whenall

---

## 6. `Task.WhenAny()`

### Definitions

**Core Definition**
`Task.WhenAny()` creates a task that completes when **any one** of the tasks in a collection has completed.

**Technical Definition**
`Task.WhenAny` returns a task that completes when any task in a collection is complete. Unlike `WhenAll`, which returns the unwrapped results of all successfully completed tasks, `WhenAny` returns the completed task itself. This is useful for redundancy (racing multiple operations), interleaving (processing tasks as they complete), throttling (limiting concurrency), and early bailout (timeouts).

**Beginner-Friendly Explanation**
`WhenAny` is **“wait for the first one to finish.”** It tells you which task completed first.

---

### Purposes

- To implement **timeouts** (`WhenAny` with `Task.Delay`).
- To race multiple operations and use the fastest result.
- To process tasks as they complete.
- To implement early cancellation.

---

### Syntax Rules and Structure

```csharp
Task<TResult> firstCompleted = await Task.WhenAny(task1, task2, task3);
TResult result = await firstCompleted;
```

- Accepts `IEnumerable<Task<TResult>>` or `params Task<TResult>[]`.
- Returns `Task<Task<TResult>>` (the first completed task).

#### Constraints and Limitations

- The returned task does not automatically observe exceptions in the non-completed tasks.
- The order of completion is not guaranteed.

---

### Annotated Code Examples and Expected Outputs

#### Example: Timeout with WhenAny

```csharp
using System;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        Task<string> download = SimulateDownloadAsync();
        Task timeout = Task.Delay(500);

        Task completed = await Task.WhenAny(download, timeout);

        if (completed == timeout)
            Console.WriteLine("Timed out");
        else
            Console.WriteLine(await download);   // Data
    }

    static async Task<string> SimulateDownloadAsync()
    {
        await Task.Delay(100);
        return "Data";
    }
}
```

**Expected Output**
```
Data
```

**Why this output occurs**
`WhenAny` returns the first completed task; the download completes before the timeout.

---

### Real-World Cases

- **Timeouts** — Combine a long-running task with `Task.Delay`.
- **Redundancy** — Query multiple mirrors and use the fastest.
- **Interleaving** — Process results as tasks complete.

---

### References

- Task.WhenAny Method — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.task.whenany

---

## 7. `Task.Delay()`

### Definitions

**Core Definition**
`Task.Delay()` creates a task that completes after a specified **time delay**, without blocking the calling thread.

**Technical Definition**
`Task.Delay` creates a task that will complete after a time delay. The method is typically used to delay the operation of all or part of a task for a specified interval of time. It can also be combined with `Task.WhenAny` to implement timeouts.

**Beginner-Friendly Explanation**
`Task.Delay` is **“asynchronous sleep.”** It pauses the logical flow without blocking the thread.

---

### Purposes

- To implement asynchronous pauses.
- To build polling loops.
- To implement timeouts with `WhenAny`.
- To simulate latency in tests.

---

### Syntax Rules and Structure

```csharp
await Task.Delay(milliseconds);
await Task.Delay(TimeSpan, cancellationToken);
```

- Accepts milliseconds or a `TimeSpan`.
- Optionally accepts a `CancellationToken`.

#### Constraints and Limitations

- `Task.Delay(0)` may return a cached completed task; do not rely on it for unique instances.
- Timer resolution is limited by the system clock.

---

### Annotated Code Examples and Expected Outputs

#### Example: Basic Delay

```csharp
using System;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        Console.WriteLine("Before delay");
        await Task.Delay(1000);
        Console.WriteLine("After delay");
    }
}
```

**Expected Output**
```
Before delay
(1 second pause)
After delay
```

**Why this output occurs**
`await Task.Delay(1000)` suspends the method for one second without blocking the thread.

---

### Real-World Cases

- **Polling loops** — `while (...) { await Task.Delay(1000); }`.
- **Timeouts** — `Task.WhenAny(operation, Task.Delay(timeout))`.
- **Testing** — Simulating network latency.

---

### References

- Task.Delay Method — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.task.delay

---

## 8. `Task.FromResult()`

### Definitions

**Core Definition**
`Task.FromResult()` creates a `Task<TResult>` that is **already completed successfully** with a specified result.

**Technical Definition**
`Task.FromResult<TResult>(TResult result)` creates a `Task<TResult>` that's completed successfully with the specified result. It is useful when implementing TAP methods that do not actually perform asynchronous work but must return a task to satisfy the method signature.

**Beginner-Friendly Explanation**
`Task.FromResult` is **“wrap this value in a completed task.”**

---

### Purposes

- To return a completed task from a synchronous code path.
- To implement async interface methods that have synchronous implementations.
- To provide cached/precomputed results as tasks.

---

### Syntax Rules and Structure

```csharp
Task<int> completed = Task.FromResult(42);
```

- Accepts any `TResult` value.
- Returns a completed `Task<TResult>`.

#### Constraints and Limitations

- In .NET 6+, `Task.FromResult` may return a cached singleton for some values; do not rely on reference equality.

---

### Annotated Code Examples and Expected Outputs

#### Example: Returning a Precomputed Result

```csharp
using System;
using System.Threading.Tasks;

class Program
{
    static Task<int> GetPrecomputedAsync() => Task.FromResult(42);

    static async Task Main()
    {
        int value = await GetPrecomputedAsync();
        Console.WriteLine(value);   // 42
    }
}
```

**Expected Output**
```
42
```

**Why this output occurs**
`Task.FromResult(42)` creates a completed task with the value `42`; `await` extracts it immediately.

---

### Real-World Cases

- **Async interface implementations** — Returning a completed task when the operation is synchronous.
- **Caching** — Returning cached results as tasks.
- **Testing** — Providing mock tasks.

---

### References

- Task.FromResult<TResult>(TResult) Method — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.task.fromresult

---

## 9. `Task.CompletedTask`

### Definitions

**Core Definition**
`Task.CompletedTask` is a **cached, successfully completed task** (without a result), useful for methods that return `Task` but have no asynchronous work to perform.

**Technical Definition**
`Task.CompletedTask` gets a task that has already completed successfully. This property returns a task whose `Status` property is set to `RanToCompletion`. To create a task that returns a value and runs to completion, call the `Task.FromResult` method.

**Beginner-Friendly Explanation**
`Task.CompletedTask` is **“a no-op completed task.”** Use it when you need to return a `Task` but there's nothing to do.

---

### Purposes

- To return a completed `Task` from synchronous methods.
- To implement async interface methods that have no asynchronous work.
- To avoid allocating a new task for no-op paths.

---

### Syntax Rules and Structure

```csharp
return Task.CompletedTask;
```

- Returns a cached `Task` with status `RanToCompletion`.

#### Constraints and Limitations

- Repeated calls may not return the same instance; do not rely on reference equality.
- Use `Task.FromResult` when a result is needed.

---

### Annotated Code Examples and Expected Outputs

#### Example: No-Op Async Method

```csharp
using System;
using System.Threading.Tasks;

class Program
{
    static Task DoNothingAsync()
    {
        return Task.CompletedTask;
    }

    static async Task Main()
    {
        await DoNothingAsync();
        Console.WriteLine("Done");
    }
}
```

**Expected Output**
```
Done
```

**Why this output occurs**
`DoNothingAsync` returns a completed task; `await` completes immediately.

---

### Real-World Cases

- **Interface implementations** — Returning `Task.CompletedTask` for synchronous no-ops.
- **Conditional async paths** — Avoiding allocations when no work is needed.

---

### References

- Task.CompletedTask Property — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.task.completedtask

---

## 10. TAP Best Practices

### Definitions

**Core Definition**
TAP best practices are the guidelines for implementing and consuming TAP methods correctly, ensuring correctness, performance, and maintainability.

**Technical Definition**
Best practices for TAP derive from Microsoft's guidance: **name methods with the `Async` suffix**; **return `Task` or `Task<TResult>`**; **avoid `out`/`ref` parameters**; **consider `CancellationToken` parameters**; **do not dispose tasks in typical TAP code**; **use `Task.Run` for CPU-bound work and native async APIs for I/O-bound work**; **avoid blocking calls**; and **use combinators (`WhenAll`, `WhenAny`) for composition**.

**Beginner-Friendly Explanation**
The best practices are: **name it Async**, **return Task**, **avoid out/ref**, **support cancellation**, and **don't block**.

---

### Guidelines and Structure

#### General Guidelines

```csharp
// 1. Name with Async suffix
public async Task<int> GetValueAsync() { ... }

// 2. Return Task or Task<T>
public Task DoWorkAsync() { ... }

// 3. Avoid out/ref — return values via TResult
// BAD: async Task GetValueAsync(out int result)
// GOOD: async Task<int> GetValueAsync()

// 4. Consider CancellationToken
public async Task DownloadAsync(string url, CancellationToken ct = default) { ... }

// 5. Do not dispose tasks in typical TAP code
// Tasks don't hold unmanaged resources in the typical case.

// 6. Use Task.Run for CPU-bound work; native async for I/O
// CPU-bound: await Task.Run(() => Compute());
// I/O-bound: await File.ReadAllTextAsync(path);

// 7. Avoid blocking calls
// BAD: task.Wait(); task.Result;
// GOOD: await task;

// 8. Use combinators for composition
await Task.WhenAll(task1, task2);
await Task.WhenAny(task1, Task.Delay(timeout));
```

#### Rules and Constraints

- **Do** name methods with the `Async` suffix.
- **Do** return `Task` or `Task<TResult>`.
- **Do** consider `CancellationToken` parameters.
- **Do** use `Task.Run` for CPU-bound work.
- **Do** use `Task.WhenAll`/`WhenAny` for composition.
- **Do not** use `out`/`ref` parameters in TAP methods.
- **Do not** dispose tasks unnecessarily.
- **Do not** block on async code (`.Wait()`, `.Result`).

---

### Annotated Code Examples and Expected Outputs

#### Example: Well-Designed TAP Method

```csharp
using System;
using System.Threading;
using System.Threading.Tasks;

public class DataService
{
    public async Task<string> GetDataAsync(string url, CancellationToken cancellationToken = default)
    {
        using var client = new HttpClient();
        return await client.GetStringAsync(url, cancellationToken);
    }
}

class Program
{
    static async Task Main()
    {
        var service = new DataService();
        string data = await service.GetDataAsync("https://example.com");
        Console.WriteLine(data.Length);
    }
}
```

**Expected Output (approximate)**
```
1256
```

**Why this output occurs**
The method follows TAP conventions: `Async` suffix, `Task<string>` return type, `CancellationToken` support, and no blocking.

---

### Real-World Cases

- **Library design** — Public async APIs follow TAP conventions.
- **ASP.NET Core** — Controllers and services return `Task`/`Task<T>`.
- **Cancellation** — Passing `CancellationToken` through the call chain.

---

### References

- Task-based asynchronous pattern (TAP) in .NET: Introduction and overview — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/task-based-asynchronous-pattern-tap
- Implementing the Task-based Asynchronous Pattern — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/implementing-the-task-based-asynchronous-pattern
- Consuming the Task-based Asynchronous Pattern — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/consuming-the-task-based-asynchronous-pattern

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| TAP | Introduced in **.NET 4.0 / .NET 4.5**; recommended pattern for new development. |
| `async`/`await` | Introduced in **C# 5.0 / .NET Framework 4.5**; built on TAP. |
| `ValueTask`/`ValueTask<T>` | Introduced in **.NET Core 2.0/2.1**; use only after profiling. |
| `Task.FromResult` caching | **.NET 6+** may return a cached singleton; do not rely on reference equality. |
| `Task.CompletedTask` | Available since **.NET 4.6**; cached task. |
| `Task.Factory.StartNew` with async lambdas | Returns `Task<Task>`; call `.Unwrap()` or use `Task.Run`. |
| Disposing tasks | **Not recommended** in typical TAP code; tasks don't hold unmanaged resources in the typical case. |
| `out`/`ref` parameters in TAP methods | **Avoided**; use `TResult` instead. |

---

## References (Consolidated)

- Task-based asynchronous pattern (TAP) in .NET: Introduction and overview — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/task-based-asynchronous-pattern-tap
- Implementing the Task-based Asynchronous Pattern — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/implementing-the-task-based-asynchronous-pattern
- Consuming the Task-based Asynchronous Pattern — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/consuming-the-task-based-asynchronous-pattern
- Chaining tasks using continuation tasks — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/parallel-programming/chaining-tasks-by-using-continuation-tasks
- Task.WhenAll Method — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.task.whenall
- Task.WhenAny Method — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.task.whenany
- Task.Delay Method — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.task.delay
- Task.FromResult<TResult>(TResult) Method — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.task.fromresult
- Task.CompletedTask Property — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.task.completedtask
- Task.Run Method — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.task.run
- TaskFactory.StartNew Method — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.taskfactory.startnew

---