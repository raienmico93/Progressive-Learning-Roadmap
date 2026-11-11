# C# Async/Await — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
`async` and `await` are C# keywords that enable **asynchronous programming**: `async` marks a method as asynchronous, and `await` suspends the method until an awaited task completes, without blocking the calling thread.

**Technical Definition**
The `async` modifier specifies that a method, lambda expression, or anonymous method is asynchronous. The `await` operator suspends evaluation of the enclosing async method until the asynchronous operation represented by its operand completes. The C# compiler transforms `async` methods into **state machines** that implement suspension and resumption. Async methods return task types (`Task`, `Task<TResult>`, `ValueTask<TResult>`, or `void` for event handlers), and the caller can await them to get results or observe exceptions.

**Beginner-Friendly Explanation**
`async` says **"this method can pause."** `await` says **"pause here until this finishes."** You write code that looks sequential, but at each `await` the method yields control back to its caller and resumes later when the result is ready.

---

### Key Characteristics

- **Non-blocking** — `await` suspends without blocking the calling thread.
- **State-machine transformation** — The compiler rewrites async methods into state machines.
- **Task-returning** — Async methods (except event handlers) return `Task`, `Task<T>`, or `ValueTask<T>`.
- **`async` is contextual** — It is a keyword only when modifying a method, lambda, or anonymous method.
- **Runs synchronously until first `await`** — An async method executes synchronously until it hits the first incomplete awaitable.
- **Exception capture** — Exceptions are stored in the returned task and rethrown on `await`.

---

### Prerequisites

- Basic understanding of C# methods, return types, and exceptions.
- Familiarity with delegates and lambda expressions.
- Awareness of threads and the thread pool.
- Understanding of I/O-bound vs. CPU-bound work.

---

### Related Programming Areas

- **Task Parallel Library (TPL)** — The infrastructure underlying `Task`.
- **Threading** — Async is a higher-level abstraction over threads.
- **I/O operations** — File, network, and database operations are the primary use case.
- **UI programming** — Async keeps UIs responsive.

---

### Core Concepts / Features

1. `async` keyword
2. `await` keyword
3. Async method
4. Task-returning methods
5. `Task.Run()`
6. `Task.Factory.StartNew()`
7. `async void`
8. `async Task`
9. `async Task<T>`
10. `ConfigureAwait(false)`
11. Async best practices
12. Async pitfalls

---

## 1. `async` keyword

### Definitions

- **Core:** `async` marks a method, lambda, or anonymous method as asynchronous, enabling `await` inside.
- **Technical:** The `async` modifier specifies that the target is asynchronous. It is contextual — a keyword only when modifying a method, lambda expression, or anonymous method. In all other contexts, the compiler treats it as an identifier. An async method runs synchronously until it reaches its first `await` expression. If the method contains no `await`, it executes synchronously (and the compiler emits warning CS4014).
- **Beginner:** `async` is the **"this method can pause"** switch.

### Purposes

- To enable the use of `await` inside the method body.
- To tell the compiler to generate a state machine for suspension/resumption.
- To define asynchronous operations that do not block the caller.

### Syntax Rules and Structure

```csharp
public async Task<int> ExampleMethodAsync()
{
    // synchronous work
    var result = await SomeAsyncOperationAsync();
    return result;
}
```

- `async` is a modifier placed before the return type.
- The return type must be `void`, a task type (`Task`, `Task<T>`, `ValueTask<T>`), or an async iterator type.
- `async` is not part of the method signature; it is an implementation detail.

### Constraints and Limitations

- `async` cannot be used on the `Main` method prior to C# 7.1 (C# 7.1+ allows `async Task Main`).
- `async` methods cannot have `ref` or `out` parameters.
- Without `await`, the method executes synchronously and triggers a compiler warning.

### Annotated Example

```csharp
using System;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        Console.WriteLine("Before await");
        await Task.Delay(1000);
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
The async method runs synchronously until `await Task.Delay(1000)`, then yields control to the caller.

### Real-World Cases

- HTTP calls (`await httpClient.GetAsync(url)`).
- Database queries (`await dbContext.Users.ToListAsync()`).
- File I/O (`await File.ReadAllTextAsync(path)`).

### References

- async keyword — C# reference — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/async

---

## 2. `await` keyword

### Definitions

- **Core:** `await` suspends the enclosing async method until the awaited task completes, without blocking the thread.
- **Technical:** The `await` operator suspends evaluation of the enclosing async method until the asynchronous operation represented by its operand completes. When the operation completes, the operator returns the result (if any). The operator does not block the thread; when suspension occurs, control returns to the caller. `await` can only be used in a method, lambda, or anonymous method modified by `async`.
- **Beginner:** `await` is the **"pause here"** keyword.

### Purposes

- To suspend execution until an asynchronous operation completes.
- To extract the result of a `Task<T>`.
- To propagate exceptions from faulted tasks.
- To free the thread during I/O waits.

### Syntax Rules and Structure

```csharp
T result = await someTask;
await someTask;   // for non-generic Task
```

- The operand must be awaitable (typically `Task`, `Task<T>`, `ValueTask`, `ValueTask<T>`, or a custom awaitable).
- `await` returns the result of `Task<T>` or `void` for `Task`.
- Exceptions in the task are rethrown at the `await` point.

### Constraints and Limitations

- `await` cannot be used outside an `async` method.
- `await` cannot be used in a `catch` or `finally` block prior to C# 6 (allowed in C# 6+).
- `await` cannot be used in a `lock` block.

### Annotated Example

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

### Real-World Cases

- Awaiting HTTP responses.
- Awaiting database queries.
- Awaiting file I/O operations.

### References

- await operator — C# reference — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/await

---

## 3. Async method

### Definitions

- **Core:** An async method is a method declared with the `async` modifier and (usually) containing one or more `await` expressions.
- **Technical:** An async method is a method whose body is transformed by the compiler into a state machine to support suspension and resumption. It returns a task type (or `void` for event handlers). The method runs synchronously until the first incomplete `await`; at that point, it yields control to its caller. When the awaited operation completes, execution resumes inside the method.
- **Beginner:** An async method is a **method that can pause and resume**.

### Purposes

- To perform asynchronous operations without blocking the calling thread.
- To compose asynchronous workflows using familiar sequential syntax.
- To propagate results and exceptions via the returned task.

### Syntax Rules and Structure

```csharp
public async Task<int> GetDataAsync()
{
    var data = await FetchDataAsync();
    return data.Length;
}
```

- Must be marked `async`.
- Must return `void`, `Task`, `Task<T>`, `ValueTask<T>`, or an async iterator type.
- Should contain at least one `await` (otherwise compiler warning).
- Naming convention: suffix with `Async`.

### Constraints and Limitations

- Cannot have `ref` or `out` parameters.
- Cannot be `async` and `unsafe` simultaneously? (Actually allowed.)
- `async` is not part of the signature.

### Annotated Example

```csharp
public async Task<string> DownloadAsync(string url)
{
    using var client = new HttpClient();
    return await client.GetStringAsync(url);
}
```

**Expected Output (conceptual)**
```
// Returns the downloaded string content wrapped in a Task<string>.
```

**Why this output occurs**
The method returns a `Task<string>` that completes with the downloaded content.

### Real-World Cases

- Service methods that perform I/O.
- Controllers in ASP.NET Core.
- Repository methods that query databases.

### References

- Asynchronous programming scenarios — C# — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/async-scenarios

---

## 4. Task-returning methods

### Definitions

- **Core:** Task-returning methods are async (or non-async) methods that return `Task`, `Task<T>`, or `ValueTask<T>`, representing an asynchronous operation.
- **Technical:** Async methods can have the following return types: `Task` (no result), `Task<TResult>` (a result), `void` (event handlers only), `IAsyncEnumerable<T>` (async streams), or any type with an accessible `GetAwaiter` method. A method returning a task type is said to be task-returning.
- **Beginner:** Task-returning methods return a **promise** of future completion.

### Purposes

- To represent asynchronous operations uniformly.
- To enable `await` by callers.
- To support composition (`Task.WhenAll`, `Task.WhenAny`).

### Syntax Rules and Structure

```csharp
Task DoWorkAsync();           // no result
Task<int> GetValueAsync();     // result of int
ValueTask<double> ComputeAsync(); // lightweight result
```

- `Task` for operations without a return value.
- `Task<T>` for operations with a return value.
- `ValueTask<T>` for allocation-sensitive scenarios.

### Constraints and Limitations

- `void` return is only appropriate for event handlers.
- `Task` and `Task<T>` allocate on the heap; `ValueTask<T>` avoids allocation when completing synchronously.

### Annotated Example

```csharp
public Task<int> GetValueAsync()
{
    return Task.FromResult(42);
}
```

**Expected Output (conceptual)**
```
// Returns a completed Task<int> with value 42.
```

**Why this output occurs**
`Task.FromResult` creates a completed task with the specified result.

### Real-World Cases

- API methods returning `Task<T>` for async results.
- Event handlers returning `void` (async void).
- High-performance libraries using `ValueTask<T>`.

### References

- Async return types — C# — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/async-return-types

---

## 5. `Task.Run()`

### Definitions

- **Core:** `Task.Run()` queues work to the thread pool and returns a `Task` representing that work. It is the preferred way to offload CPU-bound work.
- **Technical:** `Task.Run(Action)` is a quick way to call `Task.Factory.StartNew` with default parameters. Starting with .NET Framework 4.5, `Task.Run` is recommended for offloading CPU-bound work. Unlike `StartNew`, `Task.Run` does not allow child tasks started with `TaskCreationOptions.AttachedToParent` to attach to the current task instance by default. `Task.Run` automatically unwraps `Task<Task>` returned by async lambdas.
- **Beginner:** `Task.Run` is the **"do this on a background thread"** helper.

### Purposes

- To offload CPU-bound work to the thread pool.
- To keep the UI thread responsive during heavy computation.
- To avoid blocking the current thread with synchronous CPU-bound operations.

### Syntax Rules and Structure

```csharp
Task<int> task = Task.Run(() => ComputeIntensive());
await task;
```

- Accepts an `Action` or `Func<T>` (or async equivalents).
- Returns `Task` or `Task<T>`.
- Automatically unwraps `Task<Task>` from async lambdas.

### Constraints and Limitations

- Not for I/O-bound work; use native async APIs for I/O.
- Does not guarantee a new thread; uses the thread pool.
- Can cause thread-pool starvation if overused.

### Annotated Example

```csharp
using System;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        int result = await Task.Run(() =>
        {
            int sum = 0;
            for (int i = 0; i < 1_000_000; i++) sum += i;
            return sum;
        });
        Console.WriteLine(result);
    }
}
```

**Expected Output**
```
499999500000
```

**Why this output occurs**
The CPU-bound loop runs on a thread-pool thread; `await` waits for completion.

### Real-World Cases

- Background calculations.
- Image processing.
- CPU-bound data transformations.

### References

- Task.Run Method — .NET API — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.task.run
- Task.Run vs Task.Factory.StartNew — Microsoft Learn — https://learn.microsoft.com/en-us/archive/blogs/stephentoub/task-run-vs-task-factory-startnew

---

## 6. `Task.Factory.StartNew()`

### Definitions

- **Core:** `Task.Factory.StartNew()` creates and starts a task, offering more control than `Task.Run` via `TaskCreationOptions` and a custom scheduler.
- **Technical:** Calling `StartNew` is functionally equivalent to creating a `Task` using a constructor and then calling `Start` to schedule it. Starting with .NET Framework 4.5, `Task.Run` is recommended as a quicker alternative for default parameters. The key behavioural difference is that `Task.Run` by default does not allow child tasks started with `TaskCreationOptions.AttachedToParent` to attach to the current task, whereas `StartNew` does. `StartNew` returns `Task<Task>` when passed an async lambda unless `.Unwrap()` is called.
- **Beginner:** `StartNew` is the **power-user** version of `Task.Run` with more knobs.

### Purposes

- To create tasks with custom `TaskCreationOptions` (e.g., `LongRunning`).
- To schedule tasks on a custom `TaskScheduler`.
- To allow child tasks to attach to the parent.

### Syntax Rules and Structure

```csharp
Task task = Task.Factory.StartNew(() => DoWork(),
    TaskCreationOptions.LongRunning);

Task unwrapped = Task.Factory.StartNew(async () => await WorkAsync()).Unwrap();
```

- Accepts an `Action` or `Func<T>`, optional `CancellationToken`, `TaskCreationOptions`, and `TaskScheduler`.
- For async lambdas, call `.Unwrap()` to get a `Task` representing the inner task.

### Constraints and Limitations

- Returns `Task<Task>` for async lambdas unless unwrapped.
- More complex than `Task.Run`; use only when extra control is needed.
- `Task.Run` is preferred for most scenarios.

### Annotated Example

```csharp
using System;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        await Task.Factory.StartNew(async () =>
        {
            await Task.Delay(1000);
        }).Unwrap();

        Console.WriteLine("Done");
    }
}
```

**Expected Output**
```
(1 second pause)
Done
```

**Why this output occurs**
`.Unwrap()` flattens the `Task<Task>` into a `Task` that completes when the inner async lambda finishes.

### Real-World Cases

- Long-running background tasks (`TaskCreationOptions.LongRunning`).
- Custom schedulers in UI frameworks.
- Advanced task hierarchies.

### References

- TaskFactory.StartNew Method — .NET API — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.taskfactory.startnew

---

## 7. `async void`

### Definitions

- **Core:** `async void` is an async method that returns `void` instead of a task. It is intended **only for top-level event handlers**.
- **Technical:** Event handlers must declare `void` return types and cannot use or return `Task`/`Task<T>` objects. `async void` methods have significant drawbacks: exceptions go unobserved (they propagate to the `SynchronizationContext`, not the caller), callers cannot track completion (no `Task`), and testing is difficult (cannot `await` the method).
- **Beginner:** `async void` is **"fire and forget"** — use it only for event handlers.

### Purposes

- To satisfy the `void`-returning signature required by event handlers.
- To allow event handlers to use `await` without changing their signature.

### Syntax Rules and Structure

```csharp
private async void Button_Click(object sender, EventArgs e)
{
    await Task.Delay(100);
}
```

- `async void` is used only for event handlers.
- Cannot be awaited.
- Exceptions are raised on the `SynchronizationContext`.

### Constraints and Limitations

- Exceptions cannot be caught by the caller.
- Completion cannot be tracked.
- Difficult to test.
- **Always a bad practice outside top-level event handlers** (and in ASP.NET Core).

### Annotated Example

```csharp
// BAD: async void outside event handler
public async void DoWorkBadAsync()
{
    await Task.Delay(100);
}

// GOOD: async Task — callers can await
public async Task DoWorkGoodAsync()
{
    await Task.Delay(100);
}
```

**Expected Output (conceptual)**
```
// DoWorkBadAsync cannot be awaited; DoWorkGoodAsync can.
```

**Why this output occurs**
`async void` does not return a `Task`, so callers have no way to wait for completion.

### Real-World Cases

- UI event handlers (`Button_Click`, `Form_Load`).
- ASP.NET Core event handlers (discouraged).
- **Never** in library or service code.

### References

- Common async/await bugs — .NET — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/common-async-bugs

---

## 8. `async Task`

### Definitions

- **Core:** `async Task` is an async method that returns a `Task` representing an operation with no result.
- **Technical:** An async method that performs an operation but returns no value typically returns `Task`. The method returns `void` if it runs synchronously, but the `Task` return type allows callers to `await` it and observe exceptions.
- **Beginner:** `async Task` is **"do this asynchronously, no result."**

### Purposes

- To represent asynchronous operations without a return value.
- To allow callers to await completion and catch exceptions.
- To enable composition with `Task.WhenAll` and `Task.WhenAny`.

### Syntax Rules and Structure

```csharp
public async Task DoWorkAsync()
{
    await Task.Delay(100);
}
```

- Return type is `Task`.
- Caller uses `await DoWorkAsync();`.

### Constraints and Limitations

- No result; use `Task<T>` if a value is needed.
- Should not be used for event handlers (use `async void` there).

### Annotated Example

```csharp
public async Task SaveAsync(string path, string content)
{
    await File.WriteAllTextAsync(path, content);
}
```

**Expected Output (conceptual)**
```
// File is written asynchronously; returns Task.
```

**Why this output occurs**
`File.WriteAllTextAsync` returns a `Task`; `SaveAsync` returns a `Task` that completes when the file is written.

### Real-World Cases

- Saving data asynchronously.
- Sending emails.
- Performing asynchronous initialization.

### References

- Async return types — C# — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/async-return-types

---

## 9. `async Task<T>`

### Definitions

- **Core:** `async Task<T>` is an async method that returns a `Task<T>` representing an operation that produces a result of type `T`.
- **Technical:** `Task<TResult>` represents a single operation that returns a value. The `await` operator extracts the result. Accessing `.Result` before completion blocks the thread and should be avoided in favor of `await`.
- **Beginner:** `async Task<T>` is **"do this asynchronously and give me a result."**

### Purposes

- To represent asynchronous operations that return a value.
- To allow callers to `await` and obtain the result.
- To propagate exceptions through the task.

### Syntax Rules and Structure

```csharp
public async Task<int> GetCountAsync()
{
    await Task.Delay(100);
    return 42;
}
```

- Return type is `Task<T>`.
- Caller: `int result = await GetCountAsync();`.

### Constraints and Limitations

- `.Result` blocks the thread; prefer `await`.
- `ValueTask<T>` can be used to avoid allocations when appropriate.

### Annotated Example

```csharp
public async Task<User> GetUserAsync(int id)
{
    return await _dbContext.Users.FindAsync(id);
}
```

**Expected Output (conceptual)**
```
// Returns Task<User> that completes with the User entity.
```

**Why this output occurs**
`FindAsync` returns `Task<User>`; the async method returns that task.

### Real-World Cases

- Fetching a single entity from a database.
- Calling an API that returns data.
- Computing a result asynchronously.

### References

- Task<TResult> Class — .NET API — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.task-1

---

## 10. `ConfigureAwait(false)`

### Definitions

- **Core:** `ConfigureAwait(false)` configures the awaiter to **not capture the current `SynchronizationContext`**, allowing the continuation to run on any available thread-pool thread.
- **Technical:** `Task.ConfigureAwait(Boolean)` allows a `Task` instance to configure the context for the state machine that implements the `await` expression. If `false`, the `SynchronizationContext` is ignored and the framework attempts to continue execution wherever the previous asynchronous operation completed. If `true` (default), the continuation marshals back to the original context (e.g., UI thread). Using `ConfigureAwait(false)` in library code is a general recommendation: it avoids blocking the application’s synchronization context and prevents deadlocks when consumers block. When set incorrectly, performance can degrade and deadlocks may occur.
- **Beginner:** `ConfigureAwait(false)` tells the runtime **"don't bother returning to the original context; any thread is fine."**

### Purposes

- To avoid unnecessary thread hops back to the original context.
- To improve performance in library code.
- To help prevent deadlocks when callers block on async code.

### Syntax Rules and Structure

```csharp
await SomeAsyncOperation().ConfigureAwait(false);
```

- `ConfigureAwait(false)` is placed on the awaited task.
- Affects the continuation of that specific `await`.

### Constraints and Limitations

- **Do not use** `ConfigureAwait(false)` in code that must resume on the captured context (e.g., UI event handlers, ASP.NET Core controllers? Actually ASP.NET Core has no SynchronizationContext, so it's not needed there but harmless).
- **Do use** it in library code that does not need the caller's context.
- Incorrect use can cause deadlocks or performance issues.

### Annotated Example

```csharp
public async Task<string> GetDataAsync()
{
    await Task.Delay(100).ConfigureAwait(false);
    return "data";
}
```

**Expected Output (conceptual)**
```
// Continuation runs on a thread-pool thread, not the original context.
```

**Why this output occurs**
`ConfigureAwait(false)` suppresses context capture, allowing the continuation to run on any thread-pool thread.

### Real-World Cases

- Library methods that do not need UI or request context.
- High-performance server code.
- Any async method that does not touch UI elements after `await`.

### References

- ConfigureAwait FAQ — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca2007
- An alternative to ConfigureAwait(false) everywhere — Microsoft Learn — https://learn.microsoft.com/en-us/archive/blogs/benwilli/an-alternative-to-configureawait-false-everywhere

---

## 11. Async best practices

### Definitions

- **Core:** Async best practices are the guidelines that ensure async code is **correct, scalable, and maintainable**, avoiding deadlocks, blocking, and hidden pitfalls.
- **Technical:** Key recommendations include: avoid blocking calls (`Task.Wait`, `Task.Result`, `GetAwaiter().GetResult()`); use `async`/`await` throughout the call stack; avoid `async void` except for event handlers; use `ConfigureAwait(false)` in library code; use `Task.WhenAll`/`Task.WhenAny` for composition; write stateless code; and use `ValueTask` only when profiling justifies it.
- **Beginner:** The best practices are: **don't block**, **await everywhere**, **avoid async void**, and **don't over-engineer**.

### Guidelines

```csharp
// 1. Avoid blocking
// BAD: task.Wait(); task.Result;
// GOOD: await task;

// 2. Async all the way
// BAD: sync method blocking on async
// GOOD: async Task MethodAsync() { await ...; }

// 3. Avoid async void (except event handlers)
// BAD: async void DoWork()
// GOOD: async Task DoWorkAsync()

// 4. Use ConfigureAwait(false) in library code
await SomeAsyncOperation().ConfigureAwait(false);

// 5. Use Task.WhenAll / Task.WhenAny
await Task.WhenAll(task1, task2);
await Task.WhenAny(task1, task2);

// 6. Stateless code
// Depend on return values, not global state.
```

### Purposes

- To prevent deadlocks and thread-pool starvation.
- To improve scalability.
- To keep code readable and testable.

### Annotated Example

```csharp
public async Task ProcessAsync()
{
    await Task.Delay(100).ConfigureAwait(false);
    // continuation on thread pool
}
```

**Expected Output (conceptual)**
```
// Runs asynchronously without blocking.
```

**Why this output occurs**
`ConfigureAwait(false)` avoids context capture; `await` is used throughout.

### Real-World Cases

- ASP.NET Core controllers and services.
- UI event handlers (async void).
- Library methods (ConfigureAwait(false)).

### References

- Asynchronous programming scenarios — C# — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/async-scenarios

---

## 12. Async pitfalls

### Definitions

- **Core:** Async pitfalls are the common mistakes that cause async code to deadlock, run synchronously, or lose exceptions.
- **Technical:** The five most common async bugs are: **async method runs synchronously** (no `await`, or all awaits already complete); **`async void` cannot be awaited** (exceptions unobserved); **deadlocks from blocking on async code** (`.Result`/`.Wait()` on a single-threaded context); **`Task<Task>` unwrapping** (`StartNew` with async lambdas returns `Task<Task>`); and **exceptions go unobserved**.
- **Beginner:** The pitfalls are: **blocking**, **async void**, **missing await**, and **not unwrapping**.

### Common Pitfalls

#### 1. Blocking on Async Code (Deadlock)

```csharp
// BAD: deadlock on UI thread
string result = GetDataAsync().Result;
```

**Why:** The UI thread blocks on `.Result`; the continuation needs the UI thread → deadlock. **Fix:** Use `await`.

#### 2. `async void`

```csharp
// BAD: async void — exceptions unobserved
public async void DoWorkBadAsync() { await Task.Delay(100); }
```

**Why:** Cannot be awaited; exceptions propagate to `SynchronizationContext`. **Fix:** Use `async Task`.

#### 3. Async Method Runs Synchronously

```csharp
// BAD: no await — runs synchronously
public async Task<int> ComputeAsync() { return 42; }
```

**Why:** The method completes synchronously; no yielding occurs. **Fix:** Use `Task.Run` for CPU-bound work, or ensure `await` on incomplete tasks.

#### 4. `Task<Task>` Unwrapping

```csharp
// BAD: StartNew returns Task<Task>
await Task.Factory.StartNew(async () => { await Task.Delay(1000); });
```

**Why:** The outer task completes when the lambda yields; the inner task is not awaited. **Fix:** Use `Task.Run` (auto-unwraps) or call `.Unwrap()`.

#### 5. Exceptions Go Unobserved

```csharp
// BAD: exception lost
async void FireAndForget() { throw new Exception(); }
```

**Why:** No task to observe the exception. **Fix:** Return `Task` and `await` it.

### Annotated Example

```csharp
// Deadlock example
public static async Task<string> GetDataAsync()
{
    await Task.Delay(100);   // no ConfigureAwait(false)
    return "data";
}

public static void CallerThatDeadlocks()
{
    string result = GetDataAsync().Result;   // DEADLOCK on UI thread
}
```

**Expected Output (conceptual)**
```
// Deadlock; never completes.
```

**Why this output occurs**
The UI thread blocks on `.Result`; the continuation needs the UI thread to resume → deadlock.

### Real-World Cases

- UI applications (deadlocks).
- ASP.NET Core (async void is always bad).
- Libraries (unobserved exceptions).

### References

- Common async/await bugs — .NET — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/common-async-bugs
- Synchronous wrappers for asynchronous methods — .NET — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/synchronous-wrappers-for-asynchronous-methods

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| `async`/`await` | Introduced in **C# 5.0 / .NET Framework 4.5**; stable. |
| `async Task Main` | Introduced in **C# 7.1**; allows async entry point. |
| `ValueTask` | Introduced in **.NET Core 2.0**; use only after profiling. |
| `async void` | **Only for event handlers**; exceptions unobserved; cannot be awaited. |
| `ConfigureAwait(false)` | **Recommended in library code**; avoids context capture and deadlocks. |
| `Task.Run` | Preferred over `StartNew` for default parameters and async unwrapping. |
| `Task.Factory.StartNew` with async lambda | Returns `Task<Task>`; call `.Unwrap()`. |
| Blocking on async (`.Result`, `.Wait()`) | **Can deadlock**; avoid. |

---

## References (Consolidated)

- async keyword — C# reference — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/async
- await operator — C# reference — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/await
- Async return types — C# — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/async-return-types
- Asynchronous programming scenarios — C# — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/async-scenarios
- Common async/await bugs — .NET — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/common-async-bugs
- Task.Run vs Task.Factory.StartNew — Microsoft Learn — https://learn.microsoft.com/en-us/archive/blogs/stephentoub/task-run-vs-task-factory-startnew
- TaskFactory.StartNew Method — .NET API — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.taskfactory.startnew
- ConfigureAwait FAQ — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca2007
- An alternative to ConfigureAwait(false) everywhere — Microsoft Learn — https://learn.microsoft.com/en-us/archive/blogs/benwilli/an-alternative-to-configureawait-false-everywhere
- Task<TResult> Class — .NET API — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.task-1
- Synchronous wrappers for asynchronous methods — .NET — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/synchronous-wrappers-for-asynchronous-methods