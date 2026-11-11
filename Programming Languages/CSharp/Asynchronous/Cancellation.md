# C# Cancellation — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Cancellation in .NET is a **cooperative model** for gracefully stopping asynchronous or long-running synchronous operations when they are no longer needed. It is built on two lightweight types: `CancellationTokenSource` (the **controller** that issues cancellation requests) and `CancellationToken` (the **notification** passed to operations so they can observe and respond to the request). Cancellation is **cooperative**—the requesting object signals that cancellation is desired, but each listener decides how and when to terminate.

**Technical Definition**
Starting with .NET Framework 4, .NET uses a unified model for cooperative cancellation of asynchronous or long-running synchronous operations, based on a lightweight object called a **cancellation token** . The pattern is: (1) instantiate a `CancellationTokenSource`, which manages and sends cancellation notification; (2) pass the token returned by `CancellationTokenSource.Token` to each task or thread; (3) provide a mechanism for each task to respond to cancellation; (4) call `CancellationTokenSource.Cancel` to issue the request . Only the requesting object can issue the cancellation request, and each listener is responsible for noticing the request and responding in an appropriate and timely manner . `CancellationTokenSource` implements `IDisposable` and must be disposed when no longer needed . Multiple tokens can be joined into a **linked token** via `CreateLinkedTokenSource` . For async streams, the `[EnumeratorCancellation]` attribute ensures the token flows from `await foreach ... WithCancellation(...)` into the iterator .

**Beginner-Friendly Explanation**
Cancellation is **“please stop, but I'll let you decide how.”** You create a `CancellationTokenSource` (the “stop button”), hand out `CancellationToken` copies (the “stop signals”) to operations, and when you call `Cancel()`, every operation holding a token sees the signal and gracefully winds down. No operation is forcibly killed—each one is responsible for checking the token and deciding when and how to stop. This is how .NET avoids the dangers of `Thread.Abort` while still allowing long-running work to be abandoned cleanly.

---

### Key Characteristics

- **Cooperative, not forceful** — Cancellation is a request; listeners decide how to respond .
- **Separation of concerns** — The requester (`CancellationTokenSource`) is distinct from the listener (`CancellationToken`) .
- **One request, many listeners** — A single `Cancel()` call notifies all copies of the token .
- **Linked tokens** — Multiple tokens can be joined so that cancellation of any one cancels the linked token .
- **First-class in async** — `CancellationToken` is a standard parameter on async APIs and flows through the call chain .
- **`OperationCanceledException` is the signal** — Listeners throw this exception (or call `ThrowIfCancellationRequested`) to acknowledge cancellation .
- **Async streams require `[EnumeratorCancellation]`** — Without it, `WithCancellation` does not reach the iterator .

---

### Prerequisites

- Understanding of `async` and `await` and the Task-based Asynchronous Pattern (TAP).
- Familiarity with `Task` and `Task<T>`.
- Basic knowledge of asynchronous streams (`IAsyncEnumerable<T>`, `await foreach`).
- Awareness of `IDisposable` and resource cleanup.

---

### Related Programming Areas

- **Async/await** — Cancellation tokens are the standard way to cancel asynchronous operations.
- **Async streams** — `[EnumeratorCancellation]` propagates cancellation into `IAsyncEnumerable<T>` iterators.
- **Parallel programming** — PLINQ and `Parallel.For` accept `CancellationToken`.
- **Timeouts** — `CancellationTokenSource(TimeSpan)` and `CancelAfter` implement timeouts .
- **Linked tokens** — Combining user cancellation with timeouts or multiple upstream sources .

---

### Core Concepts / Features

1. Cancellation (Overarching Concept)
2. `CancellationToken`
3. `CancellationTokenSource`
4. Cancellation Propagation
5. Cancellation Best Practices
6. Cancellation Pitfalls
7. Async Streams Cancellation (`[EnumeratorCancellation]`)

---

## 1. Cancellation (Overarching Concept)

### Definitions

**Core Definition**
Cancellation is the cooperative signalling mechanism by which an operation is told to stop, with the operation itself responsible for observing the signal and terminating gracefully.

**Technical Definition**
The .NET cancellation framework is implemented as a set of related types: `CancellationTokenSource` (creates the token and issues the request) and `CancellationToken` (the lightweight value passed to listeners), plus `OperationCanceledException` (thrown by listeners to acknowledge cancellation) . Listeners can be notified of cancellation requests by **polling** (`IsCancellationRequested`), **callback registration** (`Register`), or **wait-handle** waiting .

**Beginner-Friendly Explanation**
Cancellation is a **“stop sign”** that operations check periodically. When the stop sign appears, they clean up and exit.

---

### Purposes

- To stop long-running or unnecessary work and free resources.
- To implement timeouts gracefully.
- To allow users to cancel operations (UI “Cancel” buttons, request aborts).
- To propagate cancellation through a chain of asynchronous calls.
- To shut down background services and hosted services cleanly.

---

### Syntax Rules and Structure

```
// 1. Create the source
var cts = new CancellationTokenSource();

// 2. Obtain the token
CancellationToken token = cts.Token;

// 3. Pass the token to operations
await DoWorkAsync(token);

// 4. Request cancellation
cts.Cancel();

// 5. Dispose the source
cts.Dispose();
```

**Component breakdown**

- `CancellationTokenSource` — The controller; implements `IDisposable`.
- `CancellationToken` — The struct passed to listeners.
- `Cancel()` — Issues the cancellation request.
- `Dispose()` — Releases unmanaged resources held by the source.

#### Syntax Rules

- Create the source, pass `source.Token` to operations, call `Cancel()` to request cancellation, and `Dispose()` when done .
- `CancellationToken.None` represents a token that can never be cancelled (useful as a default).
- `OperationCanceledException` (or its subclass `TaskCanceledException`) is the standard exception for acknowledging cancellation .

#### Constraints and Limitations

- Cancellation is **cooperative**: if a listener never checks the token, it will run to completion.
- `Cancel()` is idempotent; multiple calls are safe.
- Cancelling after the operation has completed has no effect.

---

### Annotated Code Examples and Expected Outputs

#### Example: Basic Cancellation

```csharp
using System;
using System.Threading;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        using var cts = new CancellationTokenSource();
        Task work = DoWorkAsync(cts.Token);

        await Task.Delay(2000);
        Console.WriteLine("Cancelling...");
        cts.Cancel();

        try { await work; }
        catch (OperationCanceledException) { Console.WriteLine("Cancelled."); }
    }

    static async Task DoWorkAsync(CancellationToken ct)
    {
        for (int i = 0; i < 10; i++)
        {
            ct.ThrowIfCancellationRequested();   // respond to cancellation
            Console.WriteLine($"Working... {i + 1}/10");
            await Task.Delay(1000, ct);          // cancellable delay
        }
    }
}
```

**Expected Output (approximate)**
```
Working... 1/10
Working... 2/10
Cancelling...
Cancelled.
```

**Why this output occurs**
`ThrowIfCancellationRequested` checks the token before each iteration; `Task.Delay(1000, ct)` also observes cancellation. After `Cancel()`, the next check throws `OperationCanceledException`, which is caught and reported.

---

### Real-World Cases

- **HTTP request aborts** — Passing `HttpContext.RequestAborted` into controllers and services.
- **UI Cancel buttons** — Cancelling background data loading.
- **Hosted services** — Receiving `stoppingToken` in `ExecuteAsync`.
- **Timeouts** — Combining a user token with a timeout token via `CreateLinkedTokenSource`.

---

### References

- Cancellation in Managed Threads — Microsoft Learn (raw) — https://raw.githubusercontent.com/dotnet/docs/main/docs/standard/threading/cancellation-in-managed-threads.md
- 관리되는 스레드의 취소 — .NET — Microsoft Learn — https://learn.microsoft.com/ko-kr/dotnet/standard/threading/cancellation-in-managed-threads
- .NET Cancellation Tokens — Compile-N-Run (GitHub) — https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/dotnet/9-net-asynchronous-programming/5-net-cancellation-tokens.mdx

---

## 2. `CancellationToken`

### Definitions

**Core Definition**
`CancellationToken` is a **lightweight value type** that represents a cancellation notification. It is passed to listeners, which monitor it to detect cancellation requests.

**Technical Definition**
`System.Threading.CancellationToken` is a struct that carries the cancellation state of a `CancellationTokenSource`. Listeners monitor the `IsCancellationRequested` property (by polling, callback, or wait handle) to detect cancellation . The token is **immutable** and cannot be cancelled directly; only the associated source can cancel it. `CancellationToken.None` represents a token that can never be cancelled. The token also supports `Register(Action)` callbacks that fire when cancellation is requested, and `ThrowIfCancellationRequested()` which throws `OperationCanceledException` when cancellation is requested .

**Beginner-Friendly Explanation**
`CancellationToken` is the **stop signal** you hand to operations. They check it periodically to see if they should stop.

---

### Purposes

- To observe cancellation requests (polling, callback, wait handle).
- To pass cancellation intent through a call chain.
- To register callbacks that run when cancellation is requested.
- To throw `OperationCanceledException` conveniently via `ThrowIfCancellationRequested`.

---

### Syntax Rules and Structure

```csharp
// Default (non-cancellable) token
CancellationToken none = CancellationToken.None;

// From a source
CancellationToken token = cts.Token;

// Check cancellation
if (token.IsCancellationRequested) { ... }

// Throw if cancelled
token.ThrowIfCancellationRequested();

// Register a callback
token.Register(() => Console.WriteLine("Cancelled!"));
```

**Component breakdown**

- `IsCancellationRequested` — `bool` property indicating cancellation.
- `ThrowIfCancellationRequested()` — Throws `OperationCanceledException` if cancelled.
- `Register(Action)` — Registers a callback invoked on cancellation.
- `CancellationToken.None` — A token that can never be cancelled.

#### Syntax Rules

- `CancellationToken` is a struct; it is copied by value but shares the source's state.
- `ThrowIfCancellationRequested()` is the preferred way to acknowledge cancellation.
- `Register` returns a `CancellationTokenRegistration` that should be disposed to unregister .

#### Constraints and Limitations

- A `CancellationToken` cannot be cancelled directly; only its source can.
- Registering callbacks on `CancellationToken.None` is a no-op.
- `ThrowIfCancellationRequested` throws `OperationCanceledException` (or `TaskCanceledException` in task contexts).

---

### Annotated Code Examples and Expected Outputs

#### Example: Polling and Callback Registration

```csharp
using System;
using System.Threading;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        using var cts = new CancellationTokenSource();
        CancellationToken token = cts.Token;

        // Callback
        token.Register(() => Console.WriteLine("Callback: cancelled"));

        Task work = Task.Run(() =>
        {
            for (int i = 0; i < 100; i++)
            {
                if (token.IsCancellationRequested)   // polling
                {
                    Console.WriteLine("Polled: stopping");
                    return;
                }
                Thread.Sleep(50);
            }
        });

        await Task.Delay(200);
        cts.Cancel();
        await work;
    }
}
```

**Expected Output**
```
Callback: cancelled
Polled: stopping
```

**Why this output occurs**
The callback fires immediately on `Cancel()`, and the worker notices `IsCancellationRequested` on its next loop iteration.

---

### Real-World Cases

- **API method parameters** — Every async API accepts `CancellationToken cancellationToken = default`.
- **Loops** — Checking the token inside long-running loops.
- **Callback cleanup** — Registering callbacks to release resources on cancellation.

---

### References

- CancellationToken Struct — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.threading.cancellationtoken
- Cancellation in Managed Threads — Microsoft Learn (raw) — https://raw.githubusercontent.com/dotnet/docs/main/docs/standard/threading/cancellation-in-managed-threads.md

---

## 3. `CancellationTokenSource`

### Definitions

**Core Definition**
`CancellationTokenSource` is the **controller object** that creates `CancellationToken` instances and issues cancellation requests to all of them.

**Technical Definition**
`System.Threading.CancellationTokenSource` manages and sends cancellation notification to the individual cancellation tokens . It exposes a `Token` property (returns a `CancellationToken`), a `Cancel()` method (issues the request), a `CancelAfter(int millisecondsDelay)` / `CancelAfter(TimeSpan)` method (schedules cancellation), and constructors that accept a timeout. It implements `IDisposable` and should be disposed to free unmanaged resources . `CreateLinkedTokenSource(params CancellationToken[])` creates a source that is cancelled when any of the supplied tokens is cancelled .

**Beginner-Friendly Explanation**
`CancellationTokenSource` is the **stop button factory**. It creates tokens and, when you press `Cancel()`, every token it created is notified.

---

### Purposes

- To create and manage cancellation tokens.
- To issue cancellation requests.
- To implement timeouts (`CancelAfter`, constructor with `TimeSpan`).
- To link multiple tokens into a combined cancellation source.
- To dispose resources associated with cancellation.

---

### Syntax Rules and Structure

```csharp
// Basic
var cts = new CancellationTokenSource();
CancellationToken token = cts.Token;
cts.Cancel();                     // request cancellation
cts.Dispose();                    // release resources

// Timeout
var timeoutCts = new CancellationTokenSource(TimeSpan.FromSeconds(5));
// or
var timeoutCts2 = new CancellationTokenSource();
timeoutCts2.CancelAfter(5000);

// Linked tokens
using var linkedCts = CancellationTokenSource.CreateLinkedTokenSource(
    userToken, timeoutCts.Token);
```

**Component breakdown**

- `Token` — The `CancellationToken` to pass to operations.
- `Cancel()` — Issues the cancellation request.
- `CancelAfter(int/TimeSpan)` — Schedules cancellation after a delay.
- `CreateLinkedTokenSource(...)` — Combines multiple tokens.
- `Dispose()` — Releases resources.

#### Syntax Rules

- `CancellationTokenSource` implements `IDisposable`; dispose when done .
- `Cancel()` is idempotent.
- `CancelAfter` schedules cancellation; calling `Cancel()` earlier cancels sooner.
- Linked sources should also be disposed.

#### Constraints and Limitations

- Once disposed, the source cannot be reused.
- `CancelAfter` uses a timer; the timer is released on disposal.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Timeout with `CancellationTokenSource`

```csharp
using System;
using System.Threading;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        using var cts = new CancellationTokenSource(TimeSpan.FromSeconds(2));
        try
        {
            await Task.Delay(5000, cts.Token);
        }
        catch (OperationCanceledException)
        {
            Console.WriteLine("Timed out");
        }
    }
}
```

**Expected Output**
```
Timed out
```

**Why this output occurs**
The source cancels automatically after 2 seconds; `Task.Delay(5000, token)` observes cancellation and throws.

---

#### Example 2: Linking User Token with Timeout

```csharp
using System;
using System.Threading;
using System.Threading.Tasks;

public static class TimeoutPolicy
{
    public static async Task ExecuteWithLinkedTimeoutAsync(
        Func<CancellationToken, Task> operation,
        TimeSpan timeout,
        CancellationToken userToken)
    {
        using var timeoutCts = new CancellationTokenSource(timeout);
        using var linkedCts = CancellationTokenSource.CreateLinkedTokenSource(
            userToken, timeoutCts.Token);

        await operation(linkedCts.Token);
    }
}
```

**Expected Output (conceptual)**
```
// Operation is cancelled if the user cancels OR the timeout elapses.
```

**Why this pattern is used**
`CreateLinkedTokenSource` combines the user token and the timeout token so that either one cancels the linked token .

---

### Real-World Cases

- **Timeouts** — Wrapping operations with `CancellationTokenSource(TimeSpan)`.
- **Request aborts** — Linking `HttpContext.RequestAborted` with internal timeouts.
- **Background service shutdown** — Using the host's `stoppingToken`.

---

### References

- CancellationTokenSource Class — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.threading.cancellationtokensource
- Coalesce cancellation tokens from timeouts — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/coalesce-cancellation-tokens-from-timeouts

---

## 4. Cancellation Propagation

### Definitions

**Core Definition**
Cancellation propagation is the practice of **passing the `CancellationToken` from caller to callee** through every layer of asynchronous code, so that a cancellation request at the top reaches the deepest operation.

**Technical Definition**
Cancellation tokens are **propagated by passing them as parameters** to every async method in the call chain. Each method should accept a `CancellationToken` (typically with a default of `default`) and forward it to child operations . For async streams, the `[EnumeratorCancellation]` attribute marks the `CancellationToken` parameter so that the token supplied by `await foreach ... WithCancellation(token)` is combined with the iterator's token . Linked token sources combine multiple cancellation sources (e.g., user cancellation + timeout) into one token .

**Beginner-Friendly Explanation**
Propagation is **“pass the stop signal down the chain.”** Every method that can be cancelled accepts a token and forwards it to the methods it calls.

---

### Purposes

- To ensure cancellation reaches every layer of the operation.
- To avoid silent cancellation failures (missing token propagation).
- To combine multiple cancellation sources (user, timeout, shutdown).
- To support cancellation in async streams via `[EnumeratorCancellation]`.

---

### Syntax Rules and Structure

```csharp
// Propagate through async methods
public async Task<Order> ProcessOrderAsync(Order order, CancellationToken ct = default)
{
    await ValidateAsync(order, ct);
    await ChargeAsync(order.Total, ct);
    return await SaveAsync(order, ct);
}

// Async streams: [EnumeratorCancellation]
async IAsyncEnumerable<Row> ReadRowsAsync(
    [EnumeratorCancellation] CancellationToken ct = default)
{
    while (await reader.ReadAsync(ct))
        yield return reader.CurrentRow;
}

// Consumer:
await foreach (var row in ReadRowsAsync().WithCancellation(userToken))
    Process(row);
```

**Component breakdown**

- `CancellationToken ct = default` — Standard parameter on async methods.
- Forwarding — Passing `ct` to every child call.
- `[EnumeratorCancellation]` — Marks the token parameter on async iterators .
- `WithCancellation(token)` — Supplies the token to the async stream consumer.
- `CreateLinkedTokenSource` — Combines multiple tokens .

#### Syntax Rules

- Every async method that performs cancellable work should accept a `CancellationToken` (typically `= default`).
- Forward the token to all child async calls .
- For async iterators, mark the token parameter with `[EnumeratorCancellation]` .
- Use `CreateLinkedTokenSource` to combine tokens when multiple cancellation sources exist .

#### Constraints and Limitations

- Without `[EnumeratorCancellation]`, `WithCancellation` does **not** reach the iterator's token .
- Linked sources must be disposed .
- Propagation is manual; there is no automatic ambient token.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Propagating Through a Call Chain

```csharp
using System;
using System.Threading;
using System.Threading.Tasks;

public class OrderService
{
    public async Task<OrderResult> ProcessOrderAsync(Order order, CancellationToken ct = default)
    {
        await ValidateOrderAsync(order, ct);
        await ChargePaymentAsync(order.Total, ct);
        return await SaveOrderAsync(order, ct);
    }

    private Task ValidateOrderAsync(Order order, CancellationToken ct) =>
        Task.Delay(100, ct);

    private Task ChargePaymentAsync(decimal total, CancellationToken ct) =>
        Task.Delay(100, ct);

    private Task<OrderResult> SaveOrderAsync(Order order, CancellationToken ct) =>
        Task.FromResult(new OrderResult());
}
```

**Expected Output (conceptual)**
```
// Cancellation at the top propagates to all three child operations.
```

**Why this pattern is used**
Every child call receives the token, so a single `Cancel()` stops the entire pipeline .

---

#### Example 2: Async Stream Cancellation with `[EnumeratorCancellation]`

```csharp
using System;
using System.Collections.Generic;
using System.Runtime.CompilerServices;
using System.Threading;
using System.Threading.Tasks;

class Program
{
    static async IAsyncEnumerable<int> CountAsync(
        [EnumeratorCancellation] CancellationToken ct = default)
    {
        for (int i = 0; ; i++)
        {
            ct.ThrowIfCancellationRequested();
            await Task.Delay(100, ct);
            yield return i;
        }
    }

    static async Task Main()
    {
        using var cts = new CancellationTokenSource(500);
        try
        {
            await foreach (var n in CountAsync().WithCancellation(cts.Token))
                Console.WriteLine(n);
        }
        catch (OperationCanceledException) { Console.WriteLine("Cancelled"); }
    }
}
```

**Expected Output (approximate)**
```
0
1
2
3
4
Cancelled
```

**Why this output occurs**
`[EnumeratorCancellation]` ensures the token from `WithCancellation` is combined with the iterator's `ct`; when the source cancels, `ThrowIfCancellationRequested` throws .

---

### Real-World Cases

- **Web APIs** — Propagating `HttpContext.RequestAborted` through controllers, services, and repositories.
- **gRPC** — Propagating `ServerCallContext.CancellationToken`.
- **Message consumers** — Propagating the shutdown token through handlers.

---

### References

- .NET Code Review Patterns — Cancellation Token Propagation — GitHub (managedcode/dotnet-skills) — https://github.com/managedcode/dotnet-skills/blob/HEAD/catalog/Platform/Code-Review/skills/code-review/references/patterns.md
- C9-CRUNCH-SHARP — `[EnumeratorCancellation]` exercise — GitHub — https://github.com/CODECRUNCHWORLDWIDE/C9-CRUNCH-SHARP/blob/f9331d18afa92513d15ad5ae2ac9192f25ec35d1/curriculum/week-08-async-channels-in-production/exercises/exercise-02-iasyncenumerable-cancellation.cs
- Coalesce cancellation tokens from timeouts — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/coalesce-cancellation-tokens-from-timeouts

---

## 5. Cancellation Best Practices

### Definitions

**Core Definition**
Cancellation best practices are the guidelines that ensure cancellation is **propagated correctly, observed promptly, and cleaned up safely** without introducing bugs or resource leaks.

**Technical Definition**
Best practices include: **always accept and propagate `CancellationToken`** ; **check the token in long loops and before expensive operations** ; **pass the token to all async operations** so the framework can propagate cancellation ; **do not catch `OperationCanceledException` unless you need specific cleanup—let it bubble up** ; **dispose `CancellationTokenSource` and linked sources** ; **use `[EnumeratorCancellation]` on async iterators** ; **use `CreateLinkedTokenSource` for timeouts** ; and **use `ConfigureAwait(false)` in library code** .

**Beginner-Friendly Explanation**
The best practices are: **pass the token everywhere**, **check it often**, **don't swallow cancellation exceptions**, and **dispose sources**.

---

### Guidelines and Structure

#### General Guidelines

```csharp
// 1. Always accept and propagate CancellationToken
public async Task ProcessAsync(CancellationToken ct = default)
{
    await Step1Async(ct);
    await Step2Async(ct);
}

// 2. Check in long loops and before expensive work
ct.ThrowIfCancellationRequested();

// 3. Pass the token to all async operations
await Task.Delay(1000, ct);

// 4. Do not catch OperationCanceledException unless you need cleanup
// Let it bubble up to the caller.

// 5. Dispose CancellationTokenSource and linked sources
using var cts = new CancellationTokenSource();
using var linked = CancellationTokenSource.CreateLinkedTokenSource(cts.Token, otherToken);

// 6. Use [EnumeratorCancellation] on async iterators
async IAsyncEnumerable<int> StreamAsync([EnumeratorCancellation] CancellationToken ct = default) { ... }

// 7. Use ConfigureAwait(false) in library code
await SomeAsync(ct).ConfigureAwait(false);
```

**Component breakdown**

- **Propagation** — Every layer accepts and forwards the token .
- **Polling** — Check the token in loops and before expensive work .
- **Token passing** — Pass the token to child async operations .
- **Exception handling** — Do not catch `OperationCanceledException` unless necessary .
- **Disposal** — Dispose sources .
- **Async streams** — Use `[EnumeratorCancellation]` .
- **ConfigureAwait** — Use in library code .

#### Rules and Constraints

- **Do** accept `CancellationToken` in every async method.
- **Do** forward the token to child calls.
- **Do** check the token in long-running loops.
- **Do** dispose sources.
- **Do not** swallow `OperationCanceledException` without rethrowing.
- **Do not** forget `[EnumeratorCancellation]` on async iterators.
- **Do not** ignore the token in library code.

---

### Annotated Code Examples and Expected Outputs

#### Example: Well-Designed Cancellation

```csharp
public class DataProcessor
{
    private readonly HttpClient _http;

    public DataProcessor(HttpClient http) => _http = http;

    public async Task<ProcessedData> FetchAndProcessAsync(
        string url, CancellationToken ct = default)
    {
        var response = await _http.GetAsync(url, ct).ConfigureAwait(false);
        response.EnsureSuccessStatusCode();
        var content = await response.Content.ReadAsStringAsync(ct).ConfigureAwait(false);

        ct.ThrowIfCancellationRequested();   // check before expensive work

        return await ProcessContentAsync(content, ct).ConfigureAwait(false);
    }

    private async Task<ProcessedData> ProcessContentAsync(
        string content, CancellationToken ct)
    {
        ct.ThrowIfCancellationRequested();
        // processing...
        return await Task.FromResult(new ProcessedData(content)).ConfigureAwait(false);
    }
}
```

**Expected Output (conceptual)**
```
// Cancellation propagates from the caller through HTTP, content read, and processing.
```

**Why this pattern is used**
The token is propagated to every async call, checked before expensive work, and `ConfigureAwait(false)` is used in library code .

---

### Real-World Cases

- **Library code** — Propagating tokens and using `ConfigureAwait(false)`.
- **Web APIs** — Propagating `HttpContext.RequestAborted`.
- **Background services** — Accepting `stoppingToken` from `BackgroundService`.

---

### References

- .NET Cancellation Tokens — Compile-N-Run (GitHub) — https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/dotnet/9-net-asynchronous-programming/5-net-cancellation-tokens.mdx
- .NET Code Review Patterns — GitHub (managedcode/dotnet-skills) — https://github.com/managedcode/dotnet-skills/blob/HEAD/catalog/Platform/Code-Review/skills/code-review/references/patterns.md
- Cancellation in Managed Threads — Microsoft Learn (raw) — https://raw.githubusercontent.com/dotnet/docs/main/docs/standard/threading/cancellation-in-managed-threads.md

---

## 6. Cancellation Pitfalls

### Definitions

**Core Definition**
Cancellation pitfalls are the common mistakes that cause cancellation to be **silently ignored, swallowed, or mishandled**, leading to resource leaks or unresponsive applications.

**Technical Definition**
Common pitfalls include: **not propagating the token** (the operation continues running); **not checking the token** in long loops (cancellation is ignored) ; **swallowing `OperationCanceledException`** without rethrowing (the caller cannot observe cancellation) ; **forgetting `[EnumeratorCancellation]`** on async iterators (the token from `WithCancellation` never reaches the iterator) ; **not disposing `CancellationTokenSource`** (resource leak) ; and **blocking on async code** (`.Result`/`.Wait()`) which can cause deadlocks with cancellation .

**Beginner-Friendly Explanation**
The pitfalls are: **ignoring the token**, **swallowing the exception**, **forgetting the attribute**, and **not disposing the source**.

---

### Common Pitfalls

#### 1. Not Propagating the Token

```csharp
// BAD: token ignored
public async Task BadAsync(CancellationToken ct)
{
    await Task.Delay(1000);   // NOT cancellable
}
```

**Fix:** `await Task.Delay(1000, ct);`.

---

#### 2. Not Checking the Token in Loops

```csharp
// BAD: loop never checks cancellation
for (int i = 0; i < 1_000_000; i++) { Compute(i); }
```

**Fix:** `ct.ThrowIfCancellationRequested()` inside the loop .

---

#### 3. Swallowing `OperationCanceledException`

```csharp
// BAD: cancellation swallowed
try { await WorkAsync(ct); }
catch (OperationCanceledException) { /* ignore */ }
```

**Fix:** Rethrow (or let it bubble) unless you need cleanup .

---

#### 4. Forgetting `[EnumeratorCancellation]`

```csharp
// BAD: token not linked to WithCancellation
async IAsyncEnumerable<int> StreamAsync(CancellationToken ct = default) { ... }
```

**Fix:** Add `[EnumeratorCancellation]` to the `CancellationToken` parameter .

---

#### 5. Not Disposing `CancellationTokenSource`

```csharp
// BAD: resource leak
var cts = new CancellationTokenSource();
```

**Fix:** `using var cts = new CancellationTokenSource();` .

---

### Annotated Code Examples and Expected Outputs

#### Example: Swallowed Cancellation

```csharp
using System;
using System.Threading;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        using var cts = new CancellationTokenSource(100);
        try
        {
            await WorkAsync(cts.Token);
        }
        catch (OperationCanceledException)
        {
            // SWALLOWED: caller cannot observe cancellation
            Console.WriteLine("Swallowed");
        }
    }

    static async Task WorkAsync(CancellationToken ct)
    {
        await Task.Delay(1000, ct);
    }
}
```

**Expected Output**
```
Swallowed
```

**Why this output occurs**
The exception is caught and discarded; the caller has no way to know the operation was cancelled. The fix is to rethrow or not catch at all.

---

### Real-World Cases

- **Web APIs** — Swallowing cancellation can cause requests to hang.
- **UI applications** — Ignoring the token can freeze the UI.
- **Libraries** — Missing `[EnumeratorCancellation]` silently breaks stream cancellation.

---

### References

- .NET Code Review Patterns — GitHub (managedcode/dotnet-skills) — https://github.com/managedcode/dotnet-skills/blob/HEAD/catalog/Platform/Code-Review/skills/code-review/references/patterns.md
- C9-CRUNCH-SHARP — `[EnumeratorCancellation]` requirement — GitHub — https://github.com/CODECRUNCHWORLDWIDE/C9-CRUNCH-SHARP/blob/f9331d18afa92513d15ad5ae2ac9192f25ec35d1/curriculum/week-08-async-channels-in-production/exercises/exercise-02-iasyncenumerable-cancellation.cs
- Cancellation in Managed Threads — Microsoft Learn (raw) — https://raw.githubusercontent.com/dotnet/docs/main/docs/standard/threading/cancellation-in-managed-threads.md

---

## 7. Async Streams Cancellation (`[EnumeratorCancellation]`)

### Definitions

**Core Definition**
`[EnumeratorCancellation]` is an attribute applied to the `CancellationToken` parameter of an async iterator (`IAsyncEnumerable<T>`), instructing the compiler to wire the token supplied by `await foreach ... WithCancellation(token)` into the iterator's token.

**Technical Definition**
`System.Runtime.CompilerServices.EnumeratorCancellationAttribute` marks the parameter that should receive the cancellation token value from `GetAsyncEnumerator(CancellationToken)` . The C# compiler combines the parameter's value with the value passed to `WithCancellation` using `CancellationTokenSource.CreateLinkedTokenSource`, so the iterator is cancelled when either source is cancelled . **Without the attribute, `WithCancellation` does not reach the iterator's token**—a silent bug .

**Beginner-Friendly Explanation**
`[EnumeratorCancellation]` is the **“connect the cancellation token”** attribute for async streams. Without it, `WithCancellation` has no effect on the iterator.

---

### Purposes

- To propagate cancellation from `await foreach ... WithCancellation(...)` into the async iterator.
- To combine the token passed at method invocation with the token passed at enumeration.
- To prevent silent cancellation bugs in async streams.

---

### Syntax Rules and Structure

```csharp
async IAsyncEnumerable<T> StreamAsync(
    [EnumeratorCancellation] CancellationToken ct = default)
{
    while (...)
    {
        ct.ThrowIfCancellationRequested();
        yield return ...;
    }
}
```

**Component breakdown**

- `[EnumeratorCancellation]` — Attribute on the `CancellationToken` parameter.
- `CancellationToken ct = default` — The parameter.
- `WithCancellation(token)` — Supplies the token from the consumer side.

#### Syntax Rules

- The attribute can be applied to only one `CancellationToken` parameter per async iterator.
- The parameter typically has a default value (`= default`).
- The attribute is in the `System.Runtime.CompilerServices` namespace .

#### Constraints and Limitations

- Without the attribute, `WithCancellation` does not affect the iterator's token .
- The combined token is created via `CreateLinkedTokenSource`; the original tokens are not replaced.

---

### Annotated Code Examples and Expected Outputs

#### Example: Correct and Incorrect Cancellation

```csharp
// CORRECT
async IAsyncEnumerable<int> CorrectAsync(
    [EnumeratorCancellation] CancellationToken ct = default)
{
    for (int i = 0; ; i++)
    {
        ct.ThrowIfCancellationRequested();
        await Task.Delay(100, ct);
        yield return i;
    }
}

// INCORRECT: missing attribute
async IAsyncEnumerable<int> IncorrectAsync(CancellationToken ct = default)
{
    for (int i = 0; ; i++)
    {
        ct.ThrowIfCancellationRequested();
        await Task.Delay(100, ct);
        yield return i;
    }
}
```

**Expected Output (conceptual)**
```
// CorrectAsync: WithCancellation(token) cancels the iterator.
// IncorrectAsync: WithCancellation(token) has no effect on the iterator.
```

**Why this matters**
The `[EnumeratorCancellation]` attribute is what makes `WithCancellation` reach the iterator .

---

### Real-World Cases

- **Streaming database queries** — Cancelling when the client disconnects.
- **Web API streaming** — Propagating `RequestAborted`.
- **UI streaming** — Cancelling when the user navigates away.

---

### References

- EnumeratorCancellationAttribute Class — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.runtime.compilerservices.enumeratorcancellationattribute
- C9-CRUNCH-SHARP — `[EnumeratorCancellation]` lecture notes — GitHub — https://github.com/CODECRUNCHWORLDWIDE/C9-CRUNCH-SHARP/blob/f9331d18afa92513d15ad5ae2ac9192f25ec35d1/curriculum/week-08-async-channels-in-production/lecture-notes/02-iasyncenumerable-and-await-foreach.md
- Why is this an issue? — GitHub (rule documentation) — https://github.com/CODECRUNCHWORLDWIDE/C9-CRUNCH-SHARP/blob/f9331d18afa92513d15ad5ae2ac9192f25ec35d1/curriculum/week-08-async-channels-in-production/exercises/exercise-02-iasyncenumerable-cancellation.cs

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| Cancellation model | Unified cooperative model since **.NET Framework 4** . |
| `CancellationToken` | Struct; stable. |
| `CancellationTokenSource` | Implements `IDisposable`; must be disposed . |
| `CreateLinkedTokenSource` | Combines multiple tokens; stable . |
| `CancelAfter` | Schedules cancellation; stable. |
| `[EnumeratorCancellation]` | Required for async stream cancellation; stable . |
| `OperationCanceledException` | Standard cancellation exception . |
| Blocking on async (`.Result`/`.Wait()`) | **Can cause deadlocks**; avoid . |
| Swallowing `OperationCanceledException` | **Hides cancellation**; rethrow or let it bubble . |
| Not disposing `CancellationTokenSource` | **Resource leak** . |

---

## References (Consolidated)

- Cancellation in Managed Threads — Microsoft Learn (raw) — https://raw.githubusercontent.com/dotnet/docs/main/docs/standard/threading/cancellation-in-managed-threads.md
- 관리되는 스레드의 취소 — .NET — Microsoft Learn — https://learn.microsoft.com/ko-kr/dotnet/standard/threading/cancellation-in-managed-threads
- .NET Cancellation Tokens — Compile-N-Run (GitHub) — https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/dotnet/9-net-asynchronous-programming/5-net-cancellation-tokens.mdx
- Coalesce cancellation tokens from timeouts — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/coalesce-cancellation-tokens-from-timeouts
- CancellationToken Struct — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.threading.cancellationtoken
- CancellationTokenSource Class — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.threading.cancellationtokensource
- EnumeratorCancellationAttribute Class — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.runtime.compilerservices.enumeratorcancellationattribute
- .NET Code Review Patterns — GitHub (managedcode/dotnet-skills) — https://github.com/managedcode/dotnet-skills/blob/HEAD/catalog/Platform/Code-Review/skills/code-review/references/patterns.md
- C9-CRUNCH-SHARP — `[EnumeratorCancellation]` exercise — GitHub — https://github.com/CODECRUNCHWORLDWIDE/C9-CRUNCH-SHARP/blob/f9331d18afa92513d15ad5ae2ac9192f25ec35d1/curriculum/week-08-async-channels-in-production/exercises/exercise-02-iasyncenumerable-cancellation.cs
- C9-CRUNCH-SHARP — `[EnumeratorCancellation]` lecture notes — GitHub — https://github.com/CODECRUNCHWORLDWIDE/C9-CRUNCH-SHARP/blob/f9331d18afa92513d15ad5ae2ac9192f25ec35d1/curriculum/week-08-async-channels-in-production/lecture-notes/02-iasyncenumerable-and-await-foreach.md
- Why is this an issue? — GitHub (rule documentation) — https://github.com/CODECRUNCHWORLDWIDE/C9-CRUNCH-SHARP/blob/f9331d18afa92513d15ad5ae2ac9192f25ec35d1/curriculum/week-08-async-channels-in-production/exercises/exercise-02-iasyncenumerable-cancellation.cs
- Use cancellation tokens in Orleans grains — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/orleans/grains/cancellation-tokens
- CancellationToken Usage — GitHub (best practices) — https://github.com/CODECRUNCHWORLDWIDE/C9-CRUNCH-SHARP/blob/f9331d18afa92513d15ad5ae2ac9192f25ec35d1/curriculum/week-08-async-channels-in-production/exercises/exercise-02-iasyncenumerable-cancellation.cs