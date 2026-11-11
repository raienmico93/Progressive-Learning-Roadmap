# C# Async Patterns — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Async patterns are the **reusable design and coding conventions** that make asynchronous C# code correct, scalable, and maintainable. They cover how asynchronous methods should be structured, how they propagate through a call stack, how they handle errors, timeouts, and retries, and how they compose multiple asynchronous operations. Mastering these patterns is what separates code that merely *compiles* with `async`/`await` from code that is **truly asynchronous, deadlock-free, and production-ready**.

**Technical Definition**
Async patterns in C# are built on the **Task-based Asynchronous Pattern (TAP)** and the `async`/`await` language support. The core patterns include: **"async all the way"** (every method in a call chain that performs asynchronous work is `async` and returns a task type, never mixing blocking calls with asynchronous operations); **avoiding `async void`** (except for top-level event handlers) because exceptions in `async void` methods are unobservable; **`ConfigureAwait(false)`** to avoid capturing the synchronization context in library code; **task composition** (`Task.WhenAll`, `Task.WhenAny`, `Task.WhenEach`) for orchestrating concurrent operations; **structured error handling** with `try`/`catch` around `await` and `Task.WhenAll`'s exception aggregation; **timeouts** via linked `CancellationTokenSource` or `Task.WhenAny` with `Task.Delay`; and **retries** using exponential backoff and libraries such as Polly. These patterns interact with TAP semantics, `SynchronizationContext`, and `TaskScheduler` to determine correctness under concurrency.

**Beginner-Friendly Explanation**
Async patterns are the **rules of the road for asynchronous C#**. Once you know `async` and `await`, the next challenge is writing code that is **actually async end-to-end** (not accidentally synchronous), that **never loses exceptions**, that **doesn't deadlock**, and that **handles failures gracefully**. Think of these patterns as the **best-practices playbook**: don't mix blocking and async, never use `async void` for anything you want to await, add `ConfigureAwait(false)` in libraries, use `WhenAll` to wait for many tasks, wrap awaits in `try`/`catch`, add timeouts so nothing hangs forever, and retry transient failures with backoff.

---

### Key Characteristics

- **End-to-end asynchrony** — An async chain must be async at every layer; a single blocking call poisons the pipeline.
- **Exceptions are captured in tasks** — Async methods never throw synchronously (except argument validation); exceptions are stored in the returned task and rethrown on `await`.
- **Context capture is opt-out** — `await` captures the `SynchronizationContext` by default; libraries should call `ConfigureAwait(false)`.
- **Composition is the unit of parallelism** — `Task.WhenAll` and `Task.WhenAny` compose tasks without blocking threads.
- **Timeouts and retries are cancellable** — Both rely on `CancellationToken` and linked token sources.
- **`async void` is a trap** — It cannot be awaited, exceptions go unobserved, and testing is impossible.
- **Retries require idempotency and backoff** — Blind retries make outages worse; use exponential backoff with jitter.

---

### Prerequisites

- Solid understanding of `async` and `await` and the TAP model.
- Familiarity with `Task`, `Task<T>`, `ValueTask`, and the Task Parallel Library.
- Knowledge of `CancellationToken` and `CancellationTokenSource`.
- Awareness of `SynchronizationContext` and the UI/ASP.NET contexts.
- Basic understanding of exceptions and `try`/`catch`/`finally`.

---

### Related Programming Areas

- **TAP (Task-based Asynchronous Pattern)** — The design pattern underlying all these conventions.
- **Cancellation** — Timeouts and retries both build on cancellation tokens.
- **Polly / resilience libraries** — Provide retry, timeout, circuit-breaker policies.
- **ASP.NET Core** — Relies heavily on async all the way and `ConfigureAwait(false)` in libraries.
- **Async streams** — `IAsyncEnumerable<T>` extends async patterns to sequences.

---

### Core Concepts / Features

1. Async All the Way
2. Avoid `async void`
3. `ConfigureAwait(false)`
4. Task Composition (`WhenAll`, `WhenAny`, `WhenEach`)
5. Error Handling in Async Code
6. Timeouts
7. Retries
8. Async Patterns Best Practices

---

## 1. Async All the Way

### Definitions

**Core Definition**
"Async all the way" means that every method in a call chain that performs asynchronous work is **declared `async` and returns a task type**, and that no layer in the chain blocks on asynchronous results using `.Result`, `.Wait()`, or `.GetAwaiter().GetResult()`.

**Technical Definition**
An asynchronous call chain is **fully asynchronous** when each method returns `Task`, `Task<T>`, `ValueTask<T>`, or another awaitable type, and the caller `await`s the result. Mixing sync and async — for example, a synchronous method calling an async method and blocking on `.Result` — breaks the chain, can cause **thread-pool starvation**, and can cause **deadlocks** if a `SynchronizationContext` is captured. The general guidance from Microsoft is: **don't block, use `async` all the way**; if you have to block, use `GetAwaiter().GetResult()` as a last resort, which preserves the original exception rather than wrapping it in `AggregateException`.

**Beginner-Friendly Explanation**
"Async all the way" means **the async-ness must flow from top to bottom**. If a lower method is async, every caller up the chain must also be async and `await` it. The moment one caller blocks (`.Result`), you're no longer async—you've just added overhead and risk.

---

### Purposes

- To prevent **deadlocks** caused by blocking on async code with captured contexts.
- To prevent **thread-pool starvation** (blocking threads reduces available parallelism).
- To keep the entire pipeline truly non-blocking and scalable.
- To preserve the original exceptions without `AggregateException` wrapping.

---

### Syntax Rules and Structure

```
// GOOD: async all the way
public async Task<int> ControllerAsync()
{
    return await ServiceAsync();
}

public async Task<int> ServiceAsync()
{
    return await RepositoryAsync();
}

public async Task<int> RepositoryAsync()
{
    return await SomeIoAsync();
}
```

**Component breakdown**

- `async` — Every method in the chain is async.
- `Task<T>` — Task type flows up.
- `await` — Awaited at every level, no blocking.

#### Syntax Rules

- Every method in the call chain returns a task type and is `async` (unless it simply forwards the task).
- Never block on async results (`.Result`, `.Wait()`).
- If forced to block, use `GetAwaiter().GetResult()` to preserve the original exception.

#### Constraints and Limitations

- Some legacy APIs are sync-only; in those cases, use `Task.Run` at the boundary but be aware of the thread-pool cost.
- Some UI frameworks require the async entry point to return `async void` (event handlers); that is the **only** acceptable exception.

---

### Annotated Code Examples and Expected Outputs

#### Example: Fully Async Chain

```csharp
using System;
using System.Net.Http;
using System.Threading.Tasks;

public class Repositories
{
    private readonly HttpClient _http;
    public Repositories(HttpClient http) => _http = http;

    public async Task<string> FetchContentAsync(string url) =>
        await _http.GetStringAsync(url);
}

public class Services
{
    private readonly Repositories _repos;
    public Services(Repositories r) => _repos = r;

    public async Task<string> GetHomepageAsync() =>
        await _repos.FetchContentAsync("https://example.com");
}

class Program
{
    static async Task Main()
    {
        using var http = new HttpClient();
        var services = new Services(new Repositories(http));
        string content = await services.GetHomepageAsync();
        Console.WriteLine($"Length: {content.Length}");
    }
}
```

**Expected Output (approximate)**
```
Length: 1256
```

**Why this output occurs**
The chain is async at every layer: `Main` → `Services.GetHomepageAsync` → `Repositories.FetchContentAsync` → `HttpClient.GetStringAsync`. No thread is blocked at any point.

---

### Real-World Cases

- **ASP.NET Core controllers** — All actions return `Task<IActionResult>`.
- **Repositories** — Return `Task<User>` rather than `User` if the source is I/O-bound.
- **Application services** — Async from controller to repository.

---

### References

- Async/Await — Best Practices in Asynchronous Programming — Microsoft Learn — https://learn.microsoft.com/en-us/archive/msdn-magazine/2013/march/async-await-best-practices-in-asynchronous-programming
- GetAwaiter().GetResult() vs .Result — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca2007

---

## 2. Avoid `async void`

### Definitions

**Core Definition**
`async void` is an `async` method that returns `void` instead of a task. It should be used **only for top-level event handlers** because callers cannot `await` it, and exceptions thrown inside it are not observable.

**Technical Definition**
`async void` methods are transformed into state machines just like `async Task` methods, but there is **no `Task` object** returned to the caller. This creates three problems: (1) **exceptions** thrown after the first `await` propagate to the `SynchronizationContext` (often crashing the process or terminating the ASP.NET Core request silently), not to the caller; (2) **completion** cannot be tracked (no `Task` to await); (3) **testing** is difficult because the test cannot await the operation. Microsoft's guidance is: use `async void` only in top-level event handlers; everywhere else, use `async Task` or `async Task<T>`.

**Beginner-Friendly Explanation**
`async void` is **"fire and forget, but with no tracking"** — you cannot wait for it, and if it crashes, you may never know. Use `async Task` everywhere except the top-level event handler on a UI control.

---

### Purposes

- To satisfy **event handler signatures** that require `void` return types.
- To allow `await` inside UI event handlers without changing the signature.

---

### Syntax Rules and Structure

```
// OK: top-level UI event handler
private async void Button_Click(object sender, EventArgs e)
{
    await LoadDataAsync();
}

// BAD: everywhere else
public async void DoWorkBadAsync()   // ✗ cannot be awaited
{
    await Task.Delay(100);
}
```

**Component breakdown**

- `async void` — Signature for event handlers.
- `async Task` — Preferred everywhere else.

#### Syntax Rules

- `async void` is **only** acceptable for top-level event handlers.
- Exceptions in `async void` are raised on the `SynchronizationContext`; they are not catchable by the caller.
- `async void` methods cannot be awaited or tested reliably.

#### Constraints and Limitations

- In ASP.NET Core, `async void` is **never** acceptable.
- In libraries, `async void` is **never** acceptable.
- Static analyzers (VSTHRD100) flag `async void` outside event handlers.

---

### Annotated Code Examples and Expected Outputs

#### Example: `async void` vs. `async Task`

```csharp
using System;
using System.Threading.Tasks;

class Program
{
    // OK for event handlers only
    static async void AsyncVoid()
    {
        await Task.Delay(100);
        throw new InvalidOperationException("Lost!");
    }

    // Preferred
    static async Task AsyncTask()
    {
        await Task.Delay(100);
        throw new InvalidOperationException("Observable!");
    }

    static async Task Main()
    {
        try { await AsyncTask(); }
        catch (InvalidOperationException ex)
        {
            Console.WriteLine($"Caught: {ex.Message}");   // Caught: Observable!
        }

        AsyncVoid();   // Exception is unobservable; may crash
        await Task.Delay(500);
    }
}
```

**Expected Output**
```
Caught: Observable!
```

**Why this output occurs**
`AsyncTask` returns a task that carries the exception, and `await` rethrows it inside the `try` block. `AsyncVoid`'s exception goes to the synchronization context and is not caught here.

---

### Real-World Cases

- **UI event handlers** — The only place `async void` is acceptable.
- **Legacy event-driven APIs** — Where the delegate signature is `void`.
- **Anywhere else** — Use `async Task`.

---

### References

- Async/Await — Best Practices — Microsoft Learn — https://learn.microsoft.com/en-us/archive/msdn-magazine/2013/march/async-await-best-practices-in-asynchronous-programming
- VSTHRD100: Avoid async void methods — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/vsthrd100

---

## 3. `ConfigureAwait(false)`

### Definitions

**Core Definition**
`ConfigureAwait(false)` tells the runtime **not to capture the current `SynchronizationContext`** (or `TaskScheduler`), allowing the continuation after `await` to run on any thread-pool thread.

**Technical Definition**
`Task.ConfigureAwait(bool continueOnCapturedContext)` configures how `await` resumes after the awaited operation completes. With `true` (the default), the continuation is marshaled back to the captured `SynchronizationContext` or `TaskScheduler`; with `false`, the continuation runs on the thread that completed the operation (typically a thread-pool thread). Library code should use `ConfigureAwait(false)` because libraries do not need the caller's context; application code should generally use the default (`true`) so that UI or request context is preserved. `ConfigureAwait(false)` also helps avoid deadlocks when callers block on async code, because the continuation does not need the blocked thread.

**Beginner-Friendly Explanation**
`ConfigureAwait(false)` says: **"Don't bother coming back to the original thread after the await—any thread is fine."** It improves performance and prevents deadlocks in libraries, but in UI code you usually want to come back to the UI thread, so you leave it as default.

---

### Purposes

- To avoid unnecessary thread hops in library code.
- To improve performance by not forcing context marshaling.
- To prevent deadlocks when callers block on async code.
- To keep library code free of `SynchronizationContext` dependencies.

---

### Syntax Rules and Structure

```csharp
await SomeAsyncOperation().ConfigureAwait(false);
```

**Component breakdown**

- `ConfigureAwait(bool)` — Extension method on any task.
- `false` — Do not resume on the captured context.
- `true` (default) — Resume on the captured context.

#### Syntax Rules

- `ConfigureAwait(false)` applies to the **current `await` only**; it must be applied to every `await` in a method to be effective for the whole method.
- Use `ConfigureAwait(false)` in library code that does not touch UI or `HttpContext`.
- Use default (`true`) in UI event handlers and (traditionally) ASP.NET Core controllers if you need `HttpContext` after the await.

#### Constraints and Limitations

- Never use `ConfigureAwait(false)` in code that must resume on the UI thread.
- In ASP.NET Core, there is no `SynchronizationContext`, so `ConfigureAwait(false)` is technically redundant but harmless.
- The `ConfigureAwait` analyzer **CA2007** recommends `ConfigureAwait(false)` in libraries.

---

### Annotated Code Examples and Expected Outputs

#### Example: Library Method with `ConfigureAwait(false)`

```csharp
using System;
using System.Net.Http;
using System.Threading.Tasks;

public class DataFetcher
{
    private readonly HttpClient _http;
    public DataFetcher(HttpClient http) => _http = http;

    public async Task<string> FetchAsync(string url)
    {
        using var response = await _http.GetAsync(url).ConfigureAwait(false);
        response.EnsureSuccessStatusCode();
        return await response.Content.ReadAsStringAsync().ConfigureAwait(false);
    }
}

class Program
{
    static async Task Main()
    {
        using var http = new HttpClient();
        var fetcher = new DataFetcher(http);
        string content = await fetcher.FetchAsync("https://example.com");
        Console.WriteLine($"Length: {content.Length}");
    }
}
```

**Expected Output (approximate)**
```
Length: 1256
```

**Why this output occurs**
`ConfigureAwait(false)` ensures both `await`s resume on thread-pool threads, avoiding context marshaling—appropriate for library code that does not need UI or `HttpContext`.

---

### Real-World Cases

- **Library methods** — Always use `ConfigureAwait(false)`.
- **UI event handlers** — Use default (capture the UI context).
- **ASP.NET Core controllers** — `ConfigureAwait(false)` is optional (no `SynchronizationContext`).

---

### References

- CA2007: Do not directly await a Task — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca2007
- ConfigureAwait FAQ — Microsoft Learn — https://devblogs.microsoft.com/dotnet/configureawait-faq/

---

## 4. Task Composition (`WhenAll`, `WhenAny`, `WhenEach`)

### Definitions

**Core Definition**
Task composition is the use of combinators such as `Task.WhenAll`, `Task.WhenAny`, and `Task.WhenEach` to **orchestrate multiple asynchronous operations** into a single awaitable unit.

**Technical Definition**
`Task.WhenAll` returns a task that completes when all provided tasks complete (returning an array of results for `Task<T>`); `Task.WhenAny` returns a task that completes when any provided task completes (returning the first completed task); `Task.WhenEach` (.NET 9+) returns an `IAsyncEnumerable<Task>` that yields completed tasks one by one as they complete. Composition enables fan-out/fan-in concurrency, timeouts, redundancy, and interleaved processing — all without blocking threads.

**Beginner-Friendly Explanation**
Task composition is **"combine many tasks into one"**: `WhenAll` waits for everything, `WhenAny` waits for the first, `WhenEach` processes tasks as they finish.

---

### Purposes

- To run **concurrent operations** and aggregate their results.
- To implement **timeouts** (`WhenAny` with `Task.Delay`).
- To implement **redundancy** (race multiple operations, use the fastest).
- To process tasks **as they complete** (`WhenEach`).

---

### Syntax Rules and Structure

```csharp
// Wait for all
T[] results = await Task.WhenAll(task1, task2, task3);

// Wait for the first
Task firstCompleted = await Task.WhenAny(task1, task2);

// Process as each completes (.NET 9+)
await foreach (Task<T> task in Task.WhenEach(task1, task2, task3))
{
    T result = await task;
}
```

**Component breakdown**

- `WhenAll` — Aggregates all results (or faults with the first exception).
- `WhenAny` — Returns the first completed task.
- `WhenEach` — Streams completed tasks as they finish.

#### Syntax Rules

- `WhenAll` on `Task<T>[]` returns `Task<T[]>`.
- `WhenAny` returns `Task<Task<T>>`; you must `await` the inner task to get the result.
- `WhenAll` throws the first exception if any task faults; inspect each task for additional exceptions.

#### Constraints and Limitations

- `WhenAny` does not observe exceptions in the losing tasks; fire-and-forget losers may raise unobserved-exception events.
- `WhenEach` requires .NET 9+.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: `WhenAll` — Fan-Out / Fan-In

```csharp
using System;
using System.Linq;
using System.Net.Http;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        using var http = new HttpClient();
        var urls = new[] { "https://example.com", "https://example.org" };

        Task<string>[] downloads = urls.Select(u => http.GetStringAsync(u)).ToArray();
        string[] pages = await Task.WhenAll(downloads);
        Console.WriteLine($"Downloaded {pages.Length} pages");
    }
}
```

**Expected Output**
```
Downloaded 2 pages
```

**Why this output occurs**
`WhenAll` waits for both downloads and returns the result array.

---

#### Example 2: `WhenEach` — Process as They Complete

```csharp
using System;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        Task[] tasks = new[]
        {
            Task.Delay(200).ContinueWith(_ => Console.WriteLine("Fast")),
            Task.Delay(500).ContinueWith(_ => Console.WriteLine("Slow"))
        };

        await foreach (Task t in Task.WhenEach(tasks))
        {
            Console.WriteLine($"Completed: {t.Id}");
        }
    }
}
```

**Expected Output (approximate)**
```
Fast
Completed: <id1>
Slow
Completed: <id2>
```

**Why this output occurs**
`WhenEach` yields each task as it completes, so the fast task is processed first.

---

### Real-World Cases

- **Parallel HTTP requests** — `WhenAll` for multiple API calls.
- **Timeout + redundancy** — `WhenAny` to race operations.
- **Streaming completions** — `WhenEach` to process results as they become available.

---

### References

- Task.WhenAll — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.task.whenall
- Task.WhenAny — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.task.whenany
- Task.WhenEach — .NET 9 API — https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.task.wheneach

---

## 5. Error Handling in Async Code

### Definitions

**Core Definition**
Error handling in async code is the practice of **capturing, propagating, and observing exceptions** in `async` methods and their tasks — using `try`/`catch` around `await`, inspecting `AggregateException` for `WhenAll`, and not losing exceptions in fire-and-forget tasks.

**Technical Definition**
Exceptions thrown in `async` methods are captured in the returned `Task` (for `async Task`/`Task<T>`) or re-raised on the `SynchronizationContext` (for `async void`). Awaiting a faulted task rethrows the **original exception** (preserving the stack trace via `ExceptionDispatchInfo`), while `Task.Wait()` and `Task.Result` throw `AggregateException`. `Task.WhenAll` throws the **first** exception (which is an `AggregateException` whose `InnerExceptions` lists all failures). Callers should inspect the individual tasks to see all exceptions.

**Beginner-Friendly Explanation**
Async code captures exceptions in **tasks**, not on the calling thread. `await` rethrows them like regular exceptions; `Task.Wait()` and `.Result` wrap them in `AggregateException`. `WhenAll` throws only the first exception—check each task if you need all.

---

### Purposes

- To propagate failures correctly through async pipelines.
- To inspect and handle multiple failures from `WhenAll`.
- To prevent unobserved exceptions from crashing the process.
- To log and report errors accurately.

---

### Syntax Rules and Structure

```csharp
// Basic try/catch around await
try { await SomeAsync(); }
catch (SpecificException ex) { /* handle */ }

// Aggregation with WhenAll
Task[] tasks = { t1, t2, t3 };
try { await Task.WhenAll(tasks); }
catch
{
    foreach (var t in tasks)
        if (t.IsFaulted)
            Console.WriteLine(t.Exception?.InnerException?.Message);
}
```

**Component breakdown**

- `try`/`catch` — Wraps the `await`.
- `Task.Exception` — `AggregateException` on faulted tasks.
- `Task.WhenAll` — Throws the first exception; inspect individual tasks.

#### Syntax Rules

- `await` rethrows the original exception, preserving stack trace.
- `.Wait()` and `.Result` throw `AggregateException`.
- `WhenAll` throws the first exception; individual tasks' `Exception` properties hold all exceptions.

#### Constraints and Limitations

- Exceptions in `async void` propagate to the `SynchronizationContext` (often crashes).
- Fire-and-forget tasks with unobserved exceptions may raise `TaskScheduler.UnobservedTaskException` — handle or observe them.

---

### Annotated Code Examples and Expected Outputs

#### Example: `WhenAll` with Multiple Failures

```csharp
using System;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        Task[] tasks =
        {
            Task.FromException(new InvalidOperationException("One")),
            Task.FromException(new ArgumentException("Two"))
        };

        try
        {
            await Task.WhenAll(tasks);
        }
        catch (Exception)
        {
            foreach (var t in tasks)
                if (t.IsFaulted)
                    Console.WriteLine(t.Exception?.InnerException?.Message);
        }
    }
}
```

**Expected Output**
```
One
Two
```

**Why this output occurs**
`WhenAll` throws the first exception, but each task's `Exception` property still exposes its own failure.

---

### Real-World Cases

- **HTTP clients** — Wrapping `HttpRequestException` in domain exceptions.
- **Database access** — Catching `DbException` and retrying.
- **Parallel API calls** — Inspecting all failures from `WhenAll`.

---

### References

- Exception handling (Task Parallel Library) — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/parallel-programming/exception-handling-task-parallel-library

---

## 6. Timeouts

### Definitions

**Core Definition**
A timeout is a mechanism that **cancels an asynchronous operation if it does not complete within a specified duration**, preventing indefinite waits.

**Technical Definition**
Timeouts are typically implemented using one of two patterns: (1) **`CancellationTokenSource(TimeSpan)`** or **`CancelAfter(delay)`** to cancel the operation, propagated via `CancellationToken`; or (2) **`Task.WhenAny(operation, Task.Delay(timeout))`** to race the operation against a delay. The linked-token pattern (`CreateLinkedTokenSource`) is preferred because it composes user cancellation with the timeout. In .NET 8+, `CancellationTokenSource.CancelAfter` and `Task.WaitAsync` (with `TimeSpan` or `CancellationToken`) make timeouts first-class.

**Beginner-Friendly Explanation**
A timeout is **"cancel this if it takes too long."** Use a `CancellationTokenSource` with a timeout to cancel cooperatively; use `WaitAsync` or `WhenAny` to race the operation against the clock.

---

### Purposes

- To prevent operations from hanging indefinitely.
- To free resources and threads when a dependency is unresponsive.
- To fail fast and surface failures to callers.
- To compose with user cancellation (`RequestAborted`).

---

### Syntax Rules and Structure

```csharp
// Timeout via linked tokens
using var timeoutCts = new CancellationTokenSource(TimeSpan.FromSeconds(5));
using var linked = CancellationTokenSource.CreateLinkedTokenSource(userToken, timeoutCts.Token);
await operationAsync(linked.Token);

// Timeout via WaitAsync (.NET 6+)
await operationAsync().WaitAsync(TimeSpan.FromSeconds(5));

// Timeout via WhenAny
Task op = operationAsync();
Task winner = await Task.WhenAny(op, Task.Delay(TimeSpan.FromSeconds(5)));
if (winner != op) throw new TimeoutException();
await op;
```

**Component breakdown**

- `CancellationTokenSource(TimeSpan)` — Auto-cancels after a delay.
- `CreateLinkedTokenSource` — Combines tokens.
- `WaitAsync` — .NET 6+ convenience.
- `WhenAny` + `Task.Delay` — Legacy pattern (does not cancel the operation).

#### Syntax Rules

- Linked-token pattern cancels the operation and observes `OperationCanceledException`.
- `WhenAny` pattern does not cancel the underlying operation; the operation keeps running unless it observes a token.
- `WaitAsync` throws `TimeoutException` on timeout.

#### Constraints and Limitations

- `WhenAny` without cancellation leaks the running operation.
- Dispose `CancellationTokenSource` and linked sources.

---

### Annotated Code Examples and Expected Outputs

#### Example: Timeout with `WaitAsync`

```csharp
using System;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        try
        {
            await SlowAsync().WaitAsync(TimeSpan.FromMilliseconds(200));
        }
        catch (TimeoutException)
        {
            Console.WriteLine("Timed out");
        }
    }

    static async Task SlowAsync()
    {
        await Task.Delay(1000);
    }
}
```

**Expected Output**
```
Timed out
```

**Why this output occurs**
`WaitAsync` throws `TimeoutException` if the task does not complete within 200ms.

---

### Real-World Cases

- **HTTP calls** — Timeout external requests.
- **Database queries** — Cancel long-running queries.
- **Distributed services** — Fail fast to avoid cascading failures.

---

### References

- Cancellation in Managed Threads — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/threading/cancellation-in-managed-threads
- Task.WaitAsync — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.task.waitasync

---

## 7. Retries

### Definitions

**Core Definition**
Retries are a resilience pattern in which **failed asynchronous operations are re-attempted** (usually with a delay and backoff) until they succeed or a retry limit is reached.

**Technical Definition**
Retries are implemented by looping over the operation and catching transient exceptions. Best practice is to use **exponential backoff with jitter** to avoid thundering-herd effects, and to wrap the retry loop in a **`CancellationToken`-aware** helper. The `Polly` library is the de-facto standard in .NET, providing retry policies, circuit breakers, and timeouts. Retries should be applied only to **transient, idempotent** operations; retrying non-idempotent operations (e.g., `POST` without an idempotency key) can cause duplicate side effects.

**Beginner-Friendly Explanation**
A retry is **"if it fails, wait a bit and try again."** Only retry operations that are safe to repeat (idempotent) and only for transient failures (timeouts, 503s).

---

### Purposes

- To handle transient network or service failures.
- To improve resilience without surfacing one-off errors.
- To reduce the impact of brief outages.

---

### Syntax Rules and Structure

```csharp
// Simple retry loop
for (int attempt = 1; ; attempt++)
{
    try { return await OperationAsync(); }
    catch (TransientException) when (attempt < maxAttempts)
    {
        await Task.Delay(TimeSpan.FromMilliseconds(100 * Math.Pow(2, attempt)));
    }
}

// Polly retry policy
var policy = Policy
    .Handle<HttpRequestException>()
    .WaitAndRetryAsync(3, i => TimeSpan.FromSeconds(Math.Pow(2, i)));
await policy.ExecuteAsync(() => OperationAsync());
```

**Component breakdown**

- Retry loop — Catches transient exceptions and delays.
- Exponential backoff — Increases delay per attempt.
- Jitter — Randomizes delay to prevent synchronized retries.
- Polly — Library for declarative retry/circuit-breaker/timeout policies.

#### Syntax Rules

- Retry only idempotent operations.
- Use exponential backoff with jitter.
- Respect `CancellationToken`.
- Cap total retry time.

#### Constraints and Limitations

- Blind retries can amplify outages (thundering herd).
- Retrying non-idempotent operations can duplicate side effects.
- Retries must not exceed the caller's timeout budget.

---

### Annotated Code Examples and Expected Outputs

#### Example: Manual Retry with Backoff

```csharp
using System;
using System.Net.Http;
using System.Threading.Tasks;

class Program
{
    static async Task<string> FetchWithRetryAsync(HttpClient http, string url, int maxAttempts = 3)
    {
        for (int attempt = 1; ; attempt++)
        {
            try
            {
                return await http.GetStringAsync(url);
            }
            catch (HttpRequestException) when (attempt < maxAttempts)
            {
                Console.WriteLine($"Attempt {attempt} failed; retrying...");
                await Task.Delay(TimeSpan.FromMilliseconds(100 * Math.Pow(2, attempt)));
            }
        }
    }

    static async Task Main()
    {
        using var http = new HttpClient();
        try
        {
            string content = await FetchWithRetryAsync(http, "https://httpstat.us/503");
            Console.WriteLine(content.Length);
        }
        catch (HttpRequestException)
        {
            Console.WriteLine("All attempts failed");
        }
    }
}
```

**Expected Output (approximate)**
```
Attempt 1 failed; retrying...
Attempt 2 failed; retrying...
All attempts failed
```

**Why this output occurs**
The `catch` filter catches `HttpRequestException` and retries with exponential backoff until `maxAttempts` is reached.

---

### Real-World Cases

- **HTTP clients** — Retrying 5xx and timeout failures.
- **Database connections** — Retrying transient connection errors.
- **Distributed calls** — Retrying gRPC/HTTP calls with backoff.

---

### References

- Polly — GitHub — https://github.com/App-vNext/Polly
- Retry pattern — Microsoft Learn — https://learn.microsoft.com/en-us/azure/architecture/patterns/retry

---

## 8. Async Patterns Best Practices

### General Guidelines

```
// 1. Async all the way — never block
public async Task<IActionResult> GetAsync() => await _service.GetAsync();

// 2. Avoid async void (except event handlers)
private async void Button_Click(object s, EventArgs e) { ... }   // ok

// 3. Use ConfigureAwait(false) in library code
await _http.GetStringAsync(url).ConfigureAwait(false);

// 4. Compose with WhenAll / WhenAny / WhenEach
await Task.WhenAll(downloads);

// 5. Handle errors around await
try { await SomeAsync(); } catch (SpecificException ex) { ... }

// 6. Apply timeouts with linked tokens or WaitAsync
await OperationAsync().WaitAsync(TimeSpan.FromSeconds(5));

// 7. Retry only transient, idempotent operations
await Policy.Handle<HttpRequestException>()
    .WaitAndRetryAsync(3, i => TimeSpan.FromSeconds(Math.Pow(2, i)))
    .ExecuteAsync(() => OperationAsync());

// 8. Propagate CancellationToken everywhere
public async Task DoAsync(CancellationToken ct = default)
{
    await StepAsync(ct);
}
```

### Purposes

- To build async code that is **correct, scalable, and resilient**.
- To avoid the classic async pitfalls (deadlocks, lost exceptions, thread starvation).
- To align with .NET conventions and analyzers.

### Rules and Constraints

- **Do** make the entire chain async.
- **Do** use `async Task` over `async void`.
- **Do** use `ConfigureAwait(false)` in libraries.
- **Do** compose tasks with `WhenAll`/`WhenAny`/`WhenEach`.
- **Do** handle exceptions around `await`.
- **Do** apply timeouts and retries only to transient failures.
- **Do not** block on async code (`.Result`, `.Wait()`).
- **Do not** swallow `OperationCanceledException`.
- **Do not** retry non-idempotent operations.

### Real-World Cases

- **ASP.NET Core controllers** — Async all the way, no `async void`, `ConfigureAwait(false)` in libraries.
- **Resilient microservices** — Timeouts and retries with Polly.
- **Batch jobs** — `WhenAll` for fan-out/fan-in.

### References

- Async/Await — Best Practices in Asynchronous Programming — Microsoft Learn — https://learn.microsoft.com/en-us/archive/msdn-magazine/2013/march/async-await-best-practices-in-asynchronous-programming
- Task-based asynchronous pattern (TAP) — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/task-based-asynchronous-pattern-tap

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| `async`/`await` | Introduced in **C# 5.0**; stable. |
| `async void` | **Only for event handlers**; exceptions unobservable. |
| `ConfigureAwait(false)` | **Recommended in library code**; CA2007. |
| `Task.WhenEach` | Introduced in **.NET 9**; version-specific. |
| `Task.WaitAsync` | Introduced in **.NET 6**; version-specific. |
| `GetAwaiter().GetResult()` | **Last-resort blocking**; preserves original exception. |
| `.Result` / `.Wait()` | **Risk of deadlocks**; avoid. |
| Polly retries | **Not built into .NET**; use the Polly library. |
| Retrying non-idempotent operations | **Unsafe**; can duplicate side effects. |

---

## References (Consolidated)

- Async/Await — Best Practices in Asynchronous Programming — Microsoft Learn — https://learn.microsoft.com/en-us/archive/msdn-magazine/2013/march/async-await-best-practices-in-asynchronous-programming
- Task-based asynchronous pattern (TAP) — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/task-based-asynchronous-pattern-tap
- CA2007: Do not directly await a Task — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca2007
- VSTHRD100: Avoid async void methods — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/vsthrd100
- ConfigureAwait FAQ — Microsoft Learn — https://devblogs.microsoft.com/dotnet/configureawait-faq/
- Task.WhenAll — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.task.whenall
- Task.WhenAny — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.task.whenany
- Task.WhenEach — .NET 9 API — https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.task.wheneach
- Exception handling (Task Parallel Library) — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/parallel-programming/exception-handling-task-parallel-library
- Cancellation in Managed Threads — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/threading/cancellation-in-managed-threads
- Task.WaitAsync — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.task.waitasync
- Polly — GitHub — https://github.com/App-vNext/Polly
- Retry pattern — Microsoft Learn — https://learn.microsoft.com/en-us/azure/architecture/patterns/retry

---