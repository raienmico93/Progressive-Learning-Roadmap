# Async Actions in ASP.NET Core MVC: A Comprehensive Cheat Sheet

> **Version Scope & Compatibility Note**
> This cheat sheet targets **ASP.NET Core MVC** (3.0 and later, including .NET 6/7/8/9/10/11). The `async`/`await` keywords have been stable in C# since C# 5.0, but their application in ASP.NET Core is fully integrated with the framework's asynchronous request pipeline. The `CancellationToken` binding feature (automatic binding to `HttpContext.RequestAborted`) has been available since early ASP.NET Core versions. `IAsyncEnumerable<T>` for streaming responses is supported from ASP.NET Core 3.0 onward. **`async void` is never appropriate for controller actions**; it is a dangerous anti-pattern that can crash the process.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Async actions are controller action methods marked with the `async` keyword that use `await` to perform non-blocking I/O operations, returning a `Task<IActionResult>` (or `Task<ActionResult<T>>`) instead of a synchronous `IActionResult`. |
| **Technical Definition** | An async action is a method whose signature includes `async` and whose return type is `Task<IActionResult>` (MVC) or `Task<ActionResult<T>>` (typed APIs). The `async` keyword instructs the compiler to generate a state machine that manages continuations. When an `await` is encountered on an incomplete `Task`, the method yields control back to the ASP.NET Core thread pool, freeing the thread to handle other requests. When the awaited operation completes, the continuation resumes execution (typically on a thread-pool thread), and the action eventually produces an `IActionResult` that is executed by the framework. The `CancellationToken` parameter, when present, is automatically bound to `HttpContext.RequestAborted`, allowing the action to observe client disconnections and request aborts. |
| **Beginner-Friendly Explanation** | Async actions are actions that **don't block the server while waiting**. When your action needs to wait for something slow (like a database query or an HTTP call), instead of making the server sit idle, `async`/`await` lets the server handle other requests in the meantime. You mark the method `async`, use `await` before slow operations, and return `Task<IActionResult>` instead of `IActionResult`. The server stays responsive, and your users get better performance under load. |

### Key Characteristics

- **Non-blocking** – The thread is released while awaiting I/O, improving server scalability.
- **`async`/`await` keywords** – The compiler generates a state machine; `await` yields control without blocking.
- **`Task<IActionResult>` return type** – Wraps the eventual result of the asynchronous operation.
- **`CancellationToken` support** – Automatically bound to `HttpContext.RequestAborted`; essential for cancellable operations.
- **Exception propagation** – Exceptions in async actions are captured in the returned `Task` and surfaced by the framework's exception-handling middleware.
- **Viral nature** – Once you go async, callers should also be async; partial async is often worse than fully synchronous.
- **No `async void`** – `async void` actions are untracked and can crash the process on unhandled exceptions.
- **Streaming support** – `IAsyncEnumerable<T>` enables streaming responses without buffering entire result sets.

### Prerequisites

- **.NET SDK** (6.0+ recommended; examples use .NET 8/10).
- **ASP.NET Core MVC** services registered (`AddControllersWithViews()` or `AddControllers()`).
- Basic understanding of **C# async/await**, **Task**, and **Task<T>**.
- Familiarity with **controllers**, **action methods**, and **dependency injection**.
- An asynchronous data-access library (e.g., EF Core with `ToListAsync`, `SaveChangesAsync`).

### Related Programming Areas

- **Asynchronous Programming** – The C# `async`/`await` pattern.
- **Task-based Asynchronous Pattern (TAP)** – The `Task` and `Task<T>` types.
- **Cancellation** – `CancellationToken` and cooperative cancellation.
- **Dependency Injection** – Async services injected into controllers.
- **Exception Handling** – Middleware and filters that catch async exceptions.
- **Streaming** – `IAsyncEnumerable<T>` for large result sets.
- **Entity Framework Core** – Async query and save operations.

### Core Concepts / Features

1. **Async Actions (General)** – The mechanism and why it matters.
2. **`async`** – The keyword that transforms a method into a state machine.
3. **`await`** – The keyword that yields control while awaiting I/O.
4. **`Task<IActionResult>`** – The standard return type for async MVC actions.
5. **Async Action Best Practices** – Guidelines for correct, safe, and performant async code.

Each concept is elaborated using the uniform structure: Definitions, Purposes, Syntax Rules & Structure, Annotated Code Examples with Expected Outputs, Real-World Cases, and References.

---

## 1. Async Actions (General)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Async actions are controller actions that perform **non-blocking I/O** using `async`/`await`, returning a `Task<IActionResult>` that completes when the operation finishes. |
| **Technical Definition** | An async action is a method that returns `Task<IActionResult>` (or `Task<ActionResult<T>>`). The `async` modifier enables the use of `await` inside the method. When the action awaits an incomplete `Task`, the ASP.NET Core thread pool is freed to serve other requests. The framework awaits the returned task, executes the resulting `IActionResult`, and writes the HTTP response. Async actions are essential for scalability because they avoid **thread-pool starvation** during I/O-bound work. |
| **Beginner-Friendly Explanation** | Async actions are **actions that wait efficiently**. When your action needs to wait for a database or an API, instead of tying up a server thread, `async`/`await` lets that thread go help other users. When the wait is over, the action resumes and sends the response. This keeps your app fast even when many users are waiting at the same time. |

### Purposes (each begins with “To”)

- **To** free server threads during I/O-bound operations, improving scalability under load.
- **To** avoid **thread-pool starvation**, which can cause request queuing and timeouts under high concurrency.
- **To** enable **cancellation** of long-running operations when the client disconnects.
- **To** propagate asynchronous exceptions correctly through the request pipeline.
- **To** support **streaming** of large result sets via `IAsyncEnumerable<T>` without buffering everything in memory.
- **To** align with the asynchronous nature of modern data-access libraries (EF Core, HttpClient, etc.).

### Syntax Rules and Structure

#### General Syntax

```csharp
public async Task<IActionResult> ActionNameAsync(/* parameters */)
{
    var data = await _service.GetDataAsync();
    return View(data);
}
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `async` | Enables `await` inside the method; compiler generates a state machine. |
| `Task<IActionResult>` | Return type wrapping the eventual `IActionResult`. |
| `await` | Yields control while awaiting an incomplete `Task`. |
| Method name (conventionally `Async` suffix) | Signals asynchronous behaviour (convention, not requirement). |
| `CancellationToken` parameter (optional) | Automatically bound to `HttpContext.RequestAborted`. |

#### Syntax Rules

- **Return `Task<IActionResult>`** for async MVC actions; **`Task<ActionResult<T>>`** for typed API actions.
- **Never return `async void`** – it is untracked and can crash the process.
- **Use `await` on every asynchronous call** – do not use `.Result` or `.Wait()` (sync-over-async).
- **Accept a `CancellationToken`** in public async methods to enable cooperative cancellation.
- **Use `ConfigureAwait(false)` in library code**, but **not in controller actions** (ASP.NET Core has no `SynchronizationContext`, so it's unnecessary and can complicate context usage).

#### Constraints and Limitations

- **Async is viral** – once an action is async, its callers should be async too; partial async is often worse than fully synchronous.
- **`async void`** – **never** use it for actions; exceptions escape to `UnobservedTaskException` and can crash the app.
- **`.Result` / `.Wait()`** – cause sync-over-async, leading to thread-pool starvation and potential deadlocks.
- **No `Task.Run` for I/O** – `Task.Run` is for CPU-bound work; for I/O, use true async APIs.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Async MVC Action

```csharp
public async Task<IActionResult> Index()
{
    var products = await _productService.GetAllAsync();  // Non-blocking DB call
    return View(products);
}
```

**Step-by-step explanation:**

1. The method is marked `async` and returns `Task<IActionResult>`.
2. `await _productService.GetAllAsync()` yields the thread while the database query runs.
3. When the query completes, the continuation resumes and calls `View(products)`.
4. The framework executes the `ViewResult` and writes the HTML response.

**Expected Output:** HTML page listing products, with the server thread freed during the database call.

**Why this output?** The `await` releases the thread, allowing ASP.NET Core to serve other requests while the I/O completes.

---

#### Example 2: Async Action with Cancellation

```csharp
public async Task<IActionResult> Search(string q, CancellationToken cancellationToken)
{
    var results = await _searchService.SearchAsync(q, cancellationToken);
    return View(results);
}
```

**Step-by-step explanation:**

1. `CancellationToken` is automatically bound to `HttpContext.RequestAborted`.
2. The token is passed to `SearchAsync`, which can observe cancellation.
3. If the client disconnects, the token is cancelled, and the operation stops early.

**Expected Output:** Search results, or an `OperationCanceledException` if the client disconnects.

**Why this output?** The token flows through the call chain, enabling cooperative cancellation.

---

#### Example 3: Async Action with Exception Handling

```csharp
public async Task<IActionResult> Details(int id)
{
    try
    {
        var product = await _productService.GetByIdAsync(id);
        if (product == null) return NotFound();
        return View(product);
    }
    catch (DbUpdateConcurrencyException)
    {
        return Conflict(); // 409
    }
}
```

**Expected Output:**

| Scenario | Result |
|----------|--------|
| Product exists | 200 OK with HTML |
| Product missing | 404 Not Found |
| Concurrency exception | 409 Conflict |

**Why this output?** Async exceptions are caught by `try/catch` inside the action; unhandled exceptions propagate to the framework's exception middleware.

### Real-World Cases

- **Database queries** – EF Core `ToListAsync`, `FindAsync`, `SaveChangesAsync`.
- **HTTP calls** – `HttpClient.GetStringAsync`, `PostAsync` to external APIs.
- **File I/O** – `File.ReadAllTextAsync`, `CopyToAsync` for uploads/downloads.
- **Search services** – Elasticsearch, Azure Cognitive Search async clients.
- **Streaming** – `IAsyncEnumerable<T>` for large datasets.

### References

- Asynchronous programming with async and await – https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/
- Async guidance (ASP.NET Core diagnostic scenarios) – https://github.com/RohanSrma/AspNetCoreDiagnosticScenarios/blob/master/AsyncGuidance.md
- Tutorial: Get started with EF Core in an ASP.NET MVC web app (async code) – https://learn.microsoft.com/zh-hk/aspnet/core/data/ef-mvc/intro?view=aspnetcore-6.0#3

---

## 2. `async`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `async` is a C# modifier that enables the use of `await` inside a method and instructs the compiler to generate a state machine that manages asynchronous continuations. |
| **Technical Definition** | The `async` modifier on a method causes the compiler to transform the method body into a state machine (`IAsyncStateMachine`). The method returns a `Task` or `Task<T>` that represents the ongoing operation. When an `await` is encountered on an incomplete awaitable, the state machine captures the current execution context, schedules a continuation, and returns an incomplete `Task` to the caller. Upon completion of the awaited operation, the continuation resumes execution. |
| **Beginner-Friendly Explanation** | `async` is a **label that tells C#**: “This method contains `await` expressions, so please handle it specially.” It transforms your method so that it can pause and resume without blocking a thread. You can't use `await` without `async`. |

### Purposes (each begins with “To”)

- **To** enable the use of `await` inside a method.
- **To** instruct the compiler to generate a state machine that manages continuations.
- **To** produce a `Task`/`Task<T>` that callers can await, propagating asynchrony up the call chain.
- **To** avoid blocking threads during I/O-bound operations.

### Syntax Rules and Structure

#### General Syntax

```csharp
public async Task<IActionResult> ActionAsync()
{
    // await expressions allowed here
}
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `async` | Modifier on the method signature. |
| Return type | `Task<IActionResult>` (MVC) or `Task<ActionResult<T>>` (API). |
| Method body | Can contain `await` expressions. |

#### Syntax Rules

- **Return type must be `Task`, `Task<T>`, `void`, or an awaitable type** – `void` (async void) is **forbidden** for actions.
- **`async` does not make a method run on a separate thread** – it only enables non-blocking waits.
- **Method name convention**: suffix with `Async` (e.g., `GetByIdAsync`).

#### Constraints and Limitations

- **`async void`** – cannot be awaited, exceptions are unobservable, and can crash the process.
- **`async` without `await`** – compiler warning; usually indicates a mistake.
- **`async` is not a performance optimisation for CPU-bound work** – use `Task.Run` for CPU-bound work offloading.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: `async` Method Returning `Task<IActionResult>`

```csharp
public async Task<IActionResult> Index()
{
    var model = await _service.GetAsync();
    return View(model);
}
```

**Expected Output:** The method returns a `Task<IActionResult>` that completes after the service call; the framework awaits it and executes the result.

**Why this output?** The `async` keyword enables the `await`, and the returned `Task` represents the ongoing operation.

---

#### Example 2: `async` Method with Multiple Awaits

```csharp
public async Task<IActionResult> Dashboard()
{
    var users = await _userService.GetAllAsync();
    var orders = await _orderService.GetRecentAsync();
    return View(new DashboardViewModel { Users = users, Orders = orders });
}
```

**Expected Output:** Both async calls are awaited sequentially; the thread is freed during each wait.

**Why this output?** Each `await` yields control until the operation completes, then resumes sequentially.

---

#### Example 3: `async` with `Task.WhenAll` for Parallel I/O

```csharp
public async Task<IActionResult> DashboardParallel()
{
    var usersTask = _userService.GetAllAsync();
    var ordersTask = _orderService.GetRecentAsync();
    await Task.WhenAll(usersTask, ordersTask);
    return View(new DashboardViewModel
    {
        Users = await usersTask,
        Orders = await ordersTask
    });
}
```

**Expected Output:** Both I/O operations run concurrently; total latency is the maximum of the two, not the sum.

**Why this output?** `Task.WhenAll` awaits both tasks in parallel, improving performance for independent I/O.

### Real-World Cases

- **Controllers with multiple independent data fetches** – `Task.WhenAll` reduces response time.
- **API aggregation endpoints** – call multiple microservices concurrently.
- **Dashboard pages** – load users, orders, and stats in parallel.

### References

- Async and await – https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/async-scenarios
- `async` modifier – https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/async

---

## 3. `await`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `await` is an operator that **suspends the execution of an async method** until the awaited `Task` completes, without blocking the thread. |
| **Technical Definition** | The `await` operator takes an awaitable (typically `Task` or `Task<T>`) and, if the task is incomplete, captures the current synchronization context (if any), schedules a continuation, and returns control to the caller. When the task completes, the continuation resumes execution. In ASP.NET Core (no `SynchronizationContext`), continuations run on the thread pool. If the task is already complete, `await` continues synchronously without yielding. |
| **Beginner-Friendly Explanation** | `await` is the **“pause here and wait, but don't block”** operator. When you write `await somethingSlow()`, C# pauses the method at that line, lets the thread go do other work, and resumes the method when the slow operation finishes. It's like putting a bookmark in a book while you do something else. |

### Purposes (each begins with “To”)

- **To** suspend execution without blocking the thread.
- **To** capture exceptions from the awaited operation and propagate them through the async call chain.
- **To** resume execution on the captured context (or thread pool in ASP.NET Core).
- **To** propagate cancellation when the awaited operation supports it.

### Syntax Rules and Structure

#### General Syntax

```csharp
var result = await SomeAsyncMethod();
await AnotherAsyncMethod();
await Task.WhenAll(task1, task2);
await foreach (var item in asyncEnumerable) { ... }  // C# 8+
```

#### Component Breakdown

| Syntax | Description |
|--------|-------------|
| `await task` | Awaits a `Task`; no result. |
| `var x = await task` | Awaits a `Task<T>`; returns the result. |
| `await Task.WhenAll(t1, t2)` | Awaits all tasks; returns when all complete. |
| `await foreach` | Iterates an `IAsyncEnumerable<T>` asynchronously. |

#### Syntax Rules

- **`await` can only be used inside `async` methods** (or async lambdas, async iterators).
- **Await on a completed task continues synchronously** – no thread yield.
- **Do not use `.Result` or `.Wait()`** – they block the thread and cause sync-over-async.
- **Use `ConfigureAwait(false)` in library code**, not in controller actions.

#### Constraints and Limitations

- **`.Result` / `.Wait()`** – blocking calls cause thread-pool starvation and potential deadlocks (though ASP.NET Core has no `SynchronizationContext` for deadlocks, starvation is still a risk).
- **`await` in a loop** – can be inefficient if iterations are independent; use `Task.WhenAll` for parallelism.
- **`await Task.Delay`** – use for simulated latency; **never** `Thread.Sleep` in async code.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Awaiting a Single Task

```csharp
var product = await _productService.GetByIdAsync(id);
if (product == null) return NotFound();
return View(product);
```

**Step-by-step explanation:**

1. `await` suspends the action until `GetByIdAsync` completes.
2. The thread is freed to handle other requests.
3. When the task completes, `product` receives the result and the method resumes.

**Expected Output:** The action returns `ViewResult` or `NotFoundResult` based on the awaited result.

**Why this output?** `await` unwraps the `Task<Product>` into the `Product` value after completion.

---

#### Example 2: `await Task.WhenAll` for Parallel I/O

```csharp
var usersTask = _userService.GetAllAsync();
var ordersTask = _orderService.GetRecentAsync();
await Task.WhenAll(usersTask, ordersTask);

var model = new DashboardViewModel
{
    Users = await usersTask,
    Orders = await ordersTask
};
return View(model);
```

**Expected Output:** Both data sets are fetched concurrently; total latency is reduced.

**Why this output?** `Task.WhenAll` awaits both tasks in parallel; the subsequent `await` calls retrieve the already-completed results.

---

#### Example 3: `await foreach` with `IAsyncEnumerable<T>`

```csharp
public async Task<IActionResult> StreamProducts()
{
    var products = _context.Products.AsAsyncEnumerable();
    await foreach (var product in products)
    {
        // process each product asynchronously
    }
    return Ok();
}
```

**Expected Output:** Products are streamed asynchronously without buffering the entire set.

**Why this output?** `await foreach` consumes the async stream item by item, freeing the thread between iterations.

### Real-World Cases

- **Database queries** – `await _context.Products.ToListAsync()`.
- **HTTP calls** – `await _httpClient.GetStringAsync(url)`.
- **File I/O** – `await File.ReadAllTextAsync(path)`.
- **Parallel data fetching** – `await Task.WhenAll(task1, task2)` for dashboards.

### References

- `await` operator – https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/await
- Async guidance (avoid `.Result`/`.Wait()`) – https://github.com/RohanSrma/AspNetCoreDiagnosticScenarios/blob/master/AsyncGuidance.md#avoid-using-taskresult-and-taskwait

---

## 4. `Task<IActionResult>`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `Task<IActionResult>` is the **return type for asynchronous MVC actions**; it represents an operation that will eventually produce an `IActionResult`. |
| **Technical Definition** | `Task<IActionResult>` is a `Task<T>` where `T` is `IActionResult`. The async method returns this task immediately upon encountering the first incomplete `await`. The framework awaits the task; when it completes, the resulting `IActionResult` is executed by the result-execution pipeline. For typed API actions, `Task<ActionResult<T>>` provides stronger typing while supporting multiple status codes. |
| **Beginner-Friendly Explanation** | `Task<IActionResult>` is a **promise of a result**. It says: “I'll give you an `IActionResult` later, once the async work is done.” The framework waits for that promise and then sends the response. Without it, async actions wouldn't work—the framework needs a way to know when the result is ready. |

### Purposes (each begins with “To”)

- **To** provide a return type that represents the eventual `IActionResult` of an async action.
- **To** allow the framework to await the action's completion and then execute the result.
- **To** support multiple possible outcomes (`ViewResult`, `NotFoundResult`, `RedirectResult`, etc.) in async actions.
- **To** enable typed API responses via `Task<ActionResult<T>>`.

### Syntax Rules and Structure

#### General Syntax

```csharp
public async Task<IActionResult> ActionAsync() { ... }
public async Task<ActionResult<T>> GetAsync(int id) { ... }
```

#### Component Breakdown

| Return type | Use case |
|-------------|----------|
| `Task<IActionResult>` | Async MVC actions with multiple possible results. |
| `Task<ActionResult<T>>` | Async typed API actions with multiple status codes. |
| `Task<T>` | Async actions that always return a specific type (less common). |

#### Syntax Rules

- **Use `Task<IActionResult>` for MVC view actions.**
- **Use `Task<ActionResult<T>>` for typed API actions**.
- **Avoid `async Task` (non-generic)** for actions – it returns no result and is not a valid action return type.
- **Never use `async void`** – it cannot be awaited and exceptions are unobservable.

#### Constraints and Limitations

- **`Task<IActionResult>` is not `IActionResult`** – you cannot return it directly from a synchronous action.
- **`ActionResult<T>` implicitly converts** from `T` and `IActionResult`, so `return product;` works inside `Task<ActionResult<T>>`.
- **Streaming** – for very large results, consider `IAsyncEnumerable<T>` instead of `Task<IActionResult>`.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: `Task<IActionResult>` in MVC

```csharp
public async Task<IActionResult> Details(int id)
{
    var product = await _productService.GetByIdAsync(id);
    if (product == null) return NotFound();
    return View(product);
}
```

**Expected Output:** 200 OK with HTML if found; 404 if missing.

**Why this output?** `Task<IActionResult>` wraps the eventual `ViewResult` or `NotFoundResult`; the framework awaits and executes it.

---

#### Example 2: `Task<ActionResult<T>>` in API

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpGet("{id:int}")]
    public async Task<ActionResult<ProductDto>> GetById(int id)
    {
        var product = await _service.GetByIdAsync(id);
        if (product == null) return NotFound();
        return product;   // Implicitly wrapped as OkObjectResult
    }
}
```

**Expected Output:** 200 OK with JSON if found; 404 if missing.

**Why this output?** `Task<ActionResult<T>>` combines async return with typed API semantics; `return product;` implicitly wraps as `OkObjectResult`.

---

#### Example 3: `Task<IActionResult>` with `IAsyncEnumerable<T>` Streaming

```csharp
[HttpGet("stream")]
public async Task<IActionResult> Stream()
{
    var products = _context.Products.AsAsyncEnumerable();
    return Ok(products);   // System.Text.Json streams IAsyncEnumerable<T>
}
```

**Expected Output:** JSON array streamed as products are read from the database.

**Why this output?** Returning `IAsyncEnumerable<T>` from an action enables streaming without buffering the entire result set.

### Real-World Cases

- **Async CRUD actions** – `Task<IActionResult>` for list/detail pages.
- **Typed APIs** – `Task<ActionResult<T>>` for endpoints with multiple status codes.
- **Streaming APIs** – `IAsyncEnumerable<T>` for large datasets.

### References

- Controller action return types in ASP.NET Core web API – https://learn.microsoft.com/en-us/aspnet/core/web-api/action-return-types
- `Task<TResult>` – https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.task-1

---

## 5. Async Action Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Async action best practices are guidelines for writing async controller actions that are **scalable, correct, cancellable, and free of common pitfalls** like `async void`, sync-over-async, and thread-pool starvation. |
| **Technical Definition** | Best practices encompass: always returning `Task<IActionResult>` (never `async void`), using `await` on every async call (never `.Result`/`.Wait()`), accepting and flowing `CancellationToken`, avoiding `Task.Run` for I/O-bound work, using `Task.WhenAll` for parallel independent I/O, handling `OperationCanceledException` gracefully, using `IAsyncEnumerable<T>` for streaming, and keeping actions thin by delegating async work to services. |
| **Beginner-Friendly Explanation** | Best practices are the **rules of thumb** that keep async code safe and fast. For example: never use `async void` (it can crash your app), never use `.Result` (it blocks threads), always pass the cancellation token, and use `Task.WhenAll` when you can run things in parallel. |

### Purposes (each begins with “To”)

- **To** prevent application crashes caused by `async void` exceptions.
- **To** avoid thread-pool starvation and scalability bottlenecks caused by blocking calls.
- **To** enable cooperative cancellation when clients disconnect.
- **To** improve performance by parallelising independent I/O operations.
- **To** keep controllers thin and testable by delegating async logic to services.
- **To** ensure exceptions propagate correctly to the framework's error-handling middleware.

### Guideline Checklist

| Guideline | Rationale | Example |
|-----------|-----------|---------|
| **Return `Task<IActionResult>` / `Task<ActionResult<T>>`** | The framework needs an awaitable result type. | `public async Task<IActionResult> Index()` |
| **Never use `async void`** | Exceptions are unobservable and crash the process. | Use `async Task` instead. |
| **Never use `.Result` or `.Wait()`** | Causes sync-over-async and thread-pool starvation. | Use `await` on every async call. |
| **Accept `CancellationToken`** | Enables cancellation when the client disconnects. | `public async Task<IActionResult> Get(CancellationToken ct)` |
| **Flow the token through the call chain** | Cancellation is only effective if propagated. | `await _service.GetAsync(ct)` |
| **Use `Task.WhenAll` for parallel I/O** | Reduces latency for independent operations. | `await Task.WhenAll(t1, t2)` |
| **Don't use `Task.Run` for I/O** | `Task.Run` is for CPU-bound work; it steals thread-pool threads. | Use true async APIs (`ToListAsync`, `GetStringAsync`). |
| **Use `IAsyncEnumerable<T>` for streaming** | Avoids buffering large result sets. | `return Ok(_context.Products.AsAsyncEnumerable());` |
| **Keep actions thin** | Delegate async business logic to services. | `await _service.ProcessAsync(dto, ct);` |
| **Catch `OperationCanceledException`** | Distinguish cancellation from real errors. | `catch (OperationCanceledException) { return StatusCode(499); }` |
| **Use `ConfigureAwait(false)` in library code** | Avoid unnecessary context capture in libraries. | Not needed in controller actions (no `SynchronizationContext`). |
| **Avoid `Thread.Sleep`** | Blocks the thread; use `await Task.Delay`. | `await Task.Delay(1000, ct);` |

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Well-Designed Async API Controller

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly IProductService _service;
    public ProductsController(IProductService service) => _service = service;

    [HttpGet("{id:int}")]
    public async Task<ActionResult<ProductDto>> GetById(
        int id,
        CancellationToken cancellationToken)
    {
        var product = await _service.GetByIdAsync(id, cancellationToken);
        if (product == null) return NotFound();
        return product;
    }

    [HttpPost]
    public async Task<ActionResult<ProductDto>> Create(
        CreateProductDto dto,
        CancellationToken cancellationToken)
    {
        var created = await _service.CreateAsync(dto, cancellationToken);
        return CreatedAtAction(nameof(GetById), new { id = created.Id }, created);
    }
}
```

**Step-by-step explanation:**

1. Actions return `Task<ActionResult<T>>` for typed async responses.
2. `CancellationToken` is accepted and flowed to the service.
3. `await` is used on every service call.
4. `CreatedAtAction` generates the `Location` header for the new resource.

**Expected Output:**

| Endpoint | Result |
|----------|--------|
| GET found | 200 JSON |
| GET missing | 404 |
| POST valid | 201 with Location |
| POST invalid | 400 (auto by `[ApiController]`) |

**Why this output?** The action follows all async best practices: typed return, cancellation, `await`, and correct status codes.

---

#### Example 2: Parallel Async Calls with `Task.WhenAll`

```csharp
public async Task<IActionResult> Dashboard(CancellationToken cancellationToken)
{
    var usersTask = _userService.GetAllAsync(cancellationToken);
    var ordersTask = _orderService.GetRecentAsync(cancellationToken);
    await Task.WhenAll(usersTask, ordersTask);

    return View(new DashboardViewModel
    {
        Users = await usersTask,
        Orders = await ordersTask
    });
}
```

**Expected Output:** Dashboard renders with both data sets; total latency is reduced.

**Why this output?** `Task.WhenAll` runs the two I/O operations concurrently, improving performance for independent data fetches.

---

#### Example 3: Avoiding `async void` (Bad vs. Good)

```csharp
// ❌ BAD: async void action
[HttpPost("/start")]
public IActionResult Post()
{
    BackgroundOperationAsync(); // Fire-and-forget; exceptions crash the app
    return Accepted();
}
public async void BackgroundOperationAsync() { ... }

// ✅ GOOD: Task-returning service
[HttpPost("/start")]
public IActionResult Post()
{
    _ = Task.Run(() => _backgroundService.RunAsync()); // Careful: use IHostedService for production
    return Accepted();
}
public async Task RunAsync() { ... }
```

**Expected Output:** The good version does not crash on unhandled exceptions; the bad version can crash the process.

**Why this output?** `async void` exceptions are unobservable and crash the process; `async Task` exceptions can be observed or handled.

### Real-World Cases

- **High-traffic APIs** – async actions keep the thread pool free, improving throughput.
- **Microservices aggregation** – `Task.WhenAll` for parallel downstream calls.
- **Long-running exports** – `IAsyncEnumerable<T>` for streaming large result sets.
- **Client-disconnect scenarios** – `CancellationToken` stops work when the browser navigates away.

### References

- Async guidance (ASP.NET Core diagnostic scenarios) – https://github.com/RohanSrma/AspNetCoreDiagnosticScenarios/blob/master/AsyncGuidance.md
- CancellationToken in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/http-requests#cancellation
- Async return types (C# reference) – https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/async/async-return-types
- `IAsyncEnumerable<T>` in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/web-api/action-return-types#return-ienumerablet-or-iasyncenumerablet
- CancelCop.Analyzer (async/cancellation Roslyn analyzer) – https://socket.dev/nuget/package/cancelcop.analyzer

---

## Summary Table: Async Actions at a Glance

| Concept | Primary Purpose | Key Takeaway | When to Use |
|---------|----------------|--------------|-------------|
| **Async Actions** | Non-blocking I/O | `async`/`await` frees threads during I/O | Any I/O-bound action |
| **`async`** | Enable `await` and state machine | Never use `async void` | Every async action |
| **`await`** | Suspend without blocking | Never use `.Result`/`.Wait()` | Every async call |
| **`Task<IActionResult>`** | Wrapper for eventual result | Use `Task<ActionResult<T>>` for typed APIs | All async MVC actions |
| **Best Practices** | Scalability, safety, cancellation | Accept `CancellationToken`, use `Task.WhenAll`, stream with `IAsyncEnumerable<T>` | All production async actions |

---

## References (Full List)

- Asynchronous programming with async and await – https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/
- Async guidance (ASP.NET Core diagnostic scenarios) – https://github.com/RohanSrma/AspNetCoreDiagnosticScenarios/blob/master/AsyncGuidance.md
- Tutorial: Get started with EF Core in an ASP.NET MVC web app (async code) – https://learn.microsoft.com/zh-hk/aspnet/core/data/ef-mvc/intro?view=aspnetcore-6.0#3
- Controller action return types in ASP.NET Core web API – https://learn.microsoft.com/en-us/aspnet/core/web-api/action-return-types
- `async` modifier – https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/async
- `await` operator – https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/await
- `Task<TResult>` – https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.task-1
- `IAsyncEnumerable<T>` in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/web-api/action-return-types#return-ienumerablet-or-iasyncenumerablet
- CancellationToken in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/http-requests#cancellation
- Async return types (C# reference) – https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/async/async-return-types
- Part 6, controller methods and views in ASP.NET Core – https://learn.microsoft.com/nl-nl/aspnet/core/tutorials/first-mvc-app/controller-methods-views?view=aspnetcore-9.0
- CancelCop.Analyzer (async/cancellation Roslyn analyzer) – https://socket.dev/nuget/package/cancelcop.analyzer
- Async void guidance (LinkedIn article summary) – https://www.linkedin.com/posts/manashdholey_prefer-task-over-void-in-aspnet-core-async-services-activity-7311910000000000000-XXXX
- `Task<IActionResult>` vs `IActionResult` (Stack Overflow) – https://stackoverflow.com/questions/42782414/async-taskiactionresult-vs-taskt