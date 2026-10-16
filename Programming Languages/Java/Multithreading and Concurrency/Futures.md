# Java Futures and CompletableFuture: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** `Future` represents the result of an asynchronous computation, providing methods to check if the computation is complete, wait for its completion, and retrieve the result. `CompletableFuture` is an enhanced implementation that implements both `Future` and `CompletionStage`, enabling non-blocking, composable asynchronous programming with explicit completion, chaining, combining, and exception handling capabilities.

**Technical Definition:** "A `Future` represents the result of an asynchronous computation. Methods are provided to check if the computation is complete, to wait for its completion, and to retrieve the result of the computation. The result can only be retrieved using method `get` when the computation has completed, blocking if necessary until it is ready. Cancellation is performed by the `cancel` method. Additional methods are provided to determine if the task completed normally or was cancelled. Once a computation has completed, the computation cannot be cancelled". `CompletableFuture` extends this: "A `Future` that may be explicitly completed (setting its value and status), and may be used as a `CompletionStage`, supporting dependent functions and actions that trigger upon its completion. When two or more threads attempt to `complete`, `completeExceptionally`, or `cancel` a `CompletableFuture`, only one of them succeeds". "All async methods without an explicit `Executor` argument are performed using the `ForkJoinPool.commonPool()` (unless it does not support a parallelism level of at least two, in which case, a new `Thread` is created to run each task)".

**Beginner-Friendly Explanation:** Imagine ordering food at a restaurant. With `Future`, you place your order and get a receipt (the `Future`). You can go do other things, but when you want your food, you have to stand at the counter and wait (blocking) until it's ready. With `CompletableFuture`, you can say "when my food is ready, add extra sauce, then notify me." You don't have to wait at the counter — you just tell the system what to do next, and it happens automatically. `CompletableFuture` lets you chain together multiple steps, combine results from multiple orders, and handle problems gracefully, all without blocking.

### Key Characteristics

- **Asynchronous Execution:** Tasks run in separate threads, allowing the main thread to continue working.
- **Non-Blocking Chaining:** `CompletableFuture` supports chaining dependent actions that execute automatically upon completion without blocking.
- **Explicit Completion:** Unlike `Future`, `CompletableFuture` can be explicitly completed by any thread via `complete()` or `completeExceptionally()`.
- **Composition:** Supports composing multiple futures using `thenApply`, `thenCompose`, `thenCombine`, `allOf`, and `anyOf`.
- **Exception Handling:** Provides `exceptionally()`, `handle()`, and `whenComplete()` for robust error handling.
- **Executor Flexibility:** Async methods can use the common ForkJoinPool or a custom `Executor`.
- **`Callable` Support:** `Callable` tasks can return results and throw exceptions, unlike `Runnable`.
- **`FutureTask`:** A concrete implementation of `Future` that also implements `Runnable`, allowing it to be executed by an `Executor`.

### Prerequisites

- Basic understanding of Java threads and the `Thread` class
- Familiarity with `Runnable` and `Callable` interfaces
- Knowledge of the `java.util.concurrent` package
- Understanding of lambda expressions and functional interfaces

### Related Programming Areas

- **Concurrency:** The broader topic of multithreaded programming.
- **Executors:** Thread pools manage the threads that execute futures.
- **Asynchronous Programming:** Non-blocking, event-driven code.
- **Reactive Programming:** `CompletableFuture` is a building block for reactive systems.
- **Parallel Streams:** The Fork/Join framework powers both parallel streams and `CompletableFuture`.

### Core Concepts / Features

---

## 1. `Future`

### Definitions

**Core Definition:** `Future` is an interface representing the result of an asynchronous computation, providing methods to check completion, wait for results, and cancel tasks.

**Technical Definition:** "A `Future` represents the result of an asynchronous computation. Methods are provided to check if the computation is complete, to wait for its completion, and to retrieve the result of the computation. The result can only be retrieved using method `get` when the computation has completed, blocking if necessary until it is ready. Cancellation is performed by the `cancel` method. Additional methods are provided to determine if the task completed normally or was cancelled".

**Beginner-Friendly Explanation:** `Future` is like a claim ticket at a coat check. When you hand over your coat, you get a ticket (the `Future`). Later, you can use the ticket to retrieve your coat. If the coat isn't ready yet, you wait. You can also ask "is my coat ready?" or cancel your request if you change your mind.

### Purposes

- To represent the result of an asynchronous computation
- To check whether a task is complete, cancelled, or still running
- To wait for and retrieve the result of a computation
- To cancel a task that has not yet completed
- To serve as the foundation for higher-level asynchronous APIs

### Syntax Rules and Structure

**Key Methods:**

| Method | Description |
|--------|-------------|
| `get()` | Waits if necessary and retrieves the result |
| `get(long, TimeUnit)` | Waits for at most the given time |
| `cancel(boolean)` | Attempts to cancel the task |
| `isCancelled()` | Returns `true` if cancelled before completion |
| `isDone()` | Returns `true` if completed (normally, exceptionally, or cancelled) |

**Syntax Rules:**

- `get()` blocks until the result is available.
- `get()` throws `ExecutionException` if the computation threw an exception.
- `get()` throws `CancellationException` if the computation was cancelled.
- Once completed, a `Future` cannot be cancelled.

### Annotated Complete Step-by-Step Code Examples

```java
// File: FutureDemo.java
import java.util.concurrent.*;

public class FutureDemo {
    public static void main(String[] args) throws Exception {
        // Create an executor
        ExecutorService executor = Executors.newSingleThreadExecutor();

        // Submit a Callable task
        Future<Integer> future = executor.submit(() -> {
            System.out.println("Task running in: " + Thread.currentThread().getName());
            Thread.sleep(1000);
            return 42;
        });

        // Check if done (non-blocking)
        System.out.println("Is done: " + future.isDone());  // false

        // Do other work while task runs
        System.out.println("Main thread doing other work...");

        // Retrieve result (blocking)
        Integer result = future.get();
        System.out.println("Result: " + result);  // 42

        // Check if done (now true)
        System.out.println("Is done: " + future.isDone());  // true

        executor.shutdown();
    }
}
```

**Expected Output:**
```
Is done: false
Main thread doing other work...
Task running in: pool-1-thread-1
Result: 42
Is done: true
```

**Step-by-Step Explanation:**

1. `executor.submit(Callable)` returns a `Future<Integer>` representing the pending result.
2. `future.isDone()` returns `false` because the task is still running.
3. The main thread continues executing ("doing other work") while the task runs asynchronously.
4. `future.get()` blocks until the task completes, then returns `42`.
5. `future.isDone()` now returns `true`.

### Real-World Cases

- **Web Service Calls:** Submitting HTTP requests asynchronously and retrieving responses.
- **Database Queries:** Running long queries asynchronously while the main thread continues.
- **File Processing:** Reading and parsing large files in the background.

---

## 2. `Callable`

### Definitions

**Core Definition:** `Callable<V>` is a functional interface representing a task that returns a result and may throw a checked exception.

**Technical Definition:** `Callable` is similar to `Runnable`, but it can return a value and throw checked exceptions. The `call()` method returns `V` and declares `throws Exception`. `Callable` tasks are typically submitted to an `ExecutorService`, which returns a `Future` representing the pending result.

**Beginner-Friendly Explanation:** `Callable` is like a worker who can both do a job and bring back a result. Unlike a `Runnable` (which is like a worker who does a job but brings back nothing), a `Callable` can say "here's what I found" when finished. It can also report problems (throw exceptions).

### Purposes

- To define tasks that return a result
- To allow tasks to throw checked exceptions
- To provide a value-returning alternative to `Runnable`
- To serve as the target type for `ExecutorService.submit()`

### Syntax Rules and Structure

```java
@FunctionalInterface
public interface Callable<V> {
    V call() throws Exception;
}
```

**Key Differences from `Runnable`:**

| Aspect | `Runnable` | `Callable<V>` |
|--------|-----------|---------------|
| Return value | `void` | `V` |
| Exceptions | Cannot throw checked | Can throw checked |
| Method | `run()` | `call()` |
| Executor method | `execute()` | `submit()` |

### Annotated Code Example

```java
// File: CallableDemo.java
import java.util.concurrent.*;

public class CallableDemo {
    public static void main(String[] args) throws Exception {
        ExecutorService executor = Executors.newFixedThreadPool(2);

        // Callable with a return value
        Callable<String> task = () -> {
            Thread.sleep(500);
            return "Callable result";
        };

        // Submit and get Future
        Future<String> future = executor.submit(task);

        // Retrieve result
        System.out.println(future.get());  // Callable result

        // Callable that throws an exception
        Callable<Integer> failingTask = () -> {
            throw new RuntimeException("Task failed");
        };

        Future<Integer> failingFuture = executor.submit(failingTask);

        try {
            failingFuture.get();
        } catch (ExecutionException e) {
            System.out.println("Caught: " + e.getCause().getMessage());
            // Output: Caught: Task failed
        }

        executor.shutdown();
    }
}
```

**Expected Output:**
```
Callable result
Caught: Task failed
```

### Real-World Cases

- **Data Fetching:** Callables that retrieve data from remote services.
- **Computation:** Callables that perform calculations and return results.
- **File Parsing:** Callables that parse files and return parsed data.

---

## 3. `FutureTask`

### Definitions

**Core Definition:** `FutureTask<V>` is a concrete implementation of `Future` that also implements `Runnable`, allowing it to be executed by an `Executor` or a `Thread`.

**Technical Definition:** "The `FutureTask` class is an implementation of `Future` that implements `Runnable`, and so may be executed by an `Executor`". "`FutureTask` provides the basic implementation of `Future`, often used to wrap `Callable` and `Runnable`, with methods to cancel the task, check if the task is completed, and obtain the task's execution result".

**Beginner-Friendly Explanation:** `FutureTask` is like a hybrid — it's both a task (you can run it) and a promise (you can get the result). It's useful when you need to run a task without an executor, or when you need a `Future` that you can also pass to `Thread`.

### Purposes

- To provide a concrete `Future` implementation that can be executed directly
- To wrap `Callable` or `Runnable` tasks for execution
- To enable creating `Future` instances without an `ExecutorService`
- To support both cancellation and result retrieval

### Syntax Rules and Structure

```java
// Wrap a Callable
FutureTask<String> task = new FutureTask<>(() -> "result");

// Execute via Executor
executor.execute(task);

// Or via Thread
new Thread(task).start();

// Retrieve result
String result = task.get();
```

### Annotated Code Example

```java
// File: FutureTaskDemo.java
import java.util.concurrent.*;

public class FutureTaskDemo {
    public static void main(String[] args) throws Exception {
        // Create a FutureTask wrapping a Callable
        FutureTask<String> task = new FutureTask<>(() -> {
            Thread.sleep(500);
            return "FutureTask result";
        });

        // Execute via a Thread
        Thread thread = new Thread(task);
        thread.start();

        // Do other work
        System.out.println("Main thread continuing...");

        // Get the result
        System.out.println(task.get());  // FutureTask result

        thread.join();
    }
}
```

**Expected Output:**
```
Main thread continuing...
FutureTask result
```

### Real-World Cases

- **Thread Pools:** `FutureTask` is used internally by `ThreadPoolExecutor`.
- **Simple Async Tasks:** When you need a `Future` without an executor.
- **Caching:** Storing computed results for later retrieval.

---

## 4. `CompletableFuture`

### Definitions

**Core Definition:** `CompletableFuture<T>` is an implementation of `Future` and `CompletionStage` that supports explicit completion, non-blocking chaining, composition, and exception handling.

**Technical Definition:** "A `Future` that may be explicitly completed (setting its value and status), and may be used as a `CompletionStage`, supporting dependent functions and actions that trigger upon its completion. When two or more threads attempt to `complete`, `completeExceptionally`, or `cancel` a `CompletableFuture`, only one of them succeeds". "All async methods without an explicit `Executor` argument are performed using the `ForkJoinPool.commonPool()` (unless it does not support a parallelism level of at least two, in which case, a new `Thread` is created to run each task)".

**Beginner-Friendly Explanation:** `CompletableFuture` is the evolution of `Future`. It doesn't just let you wait for a result — it lets you say "when this is done, do that, then do this other thing, and if anything goes wrong, do this instead." It's like a workflow engine for asynchronous tasks.

### Purposes

- To provide a non-blocking, composable asynchronous programming model
- To enable chaining of dependent actions without blocking
- To support combining results from multiple asynchronous tasks
- To provide robust exception handling for asynchronous pipelines
- To serve as the standard async programming tool in Java

### Syntax Rules and Structure

**Creation Methods:**

| Method | Description |
|--------|-------------|
| `supplyAsync(Supplier)` | Returns `CompletableFuture<U>` with a result |
| `supplyAsync(Supplier, Executor)` | Same, using a custom executor |
| `runAsync(Runnable)` | Returns `CompletableFuture<Void>` (no result) |
| `runAsync(Runnable, Executor)` | Same, using a custom executor |
| `completedFuture(U)` | Returns an already-completed future |

**Composition Methods:**

| Method | Description |
|--------|-------------|
| `thenApply(Function)` | Transforms the result (sync) |
| `thenApplyAsync(Function)` | Transforms the result (async) |
| `thenAccept(Consumer)` | Consumes the result (no return) |
| `thenRun(Runnable)` | Runs an action after completion |
| `thenCompose(Function)` | Chains another `CompletableFuture` (flatMap) |
| `thenCombine(other, BiFunction)` | Combines two futures' results |
| `allOf(CompletableFuture...)` | Waits for all futures |
| `anyOf(CompletableFuture...)` | Waits for any future |

**Exception Handling:**

| Method | Description |
|--------|-------------|
| `exceptionally(Function)` | Recovers from an exception |
| `handle(BiFunction)` | Processes both result and exception |
| `whenComplete(BiConsumer)` | Performs a side effect on completion |

### Annotated Complete Step-by-Step Code Examples

**Example 1: `supplyAsync` and `thenApply`**

```java
// File: CompletableFutureBasic.java
import java.util.concurrent.*;

public class CompletableFutureBasic {
    public static void main(String[] args) throws Exception {
        // Create an async task with a result
        CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
            System.out.println("Task running in: " + Thread.currentThread().getName());
            return "Hello";
        });

        // Chain a transformation (sync)
        CompletableFuture<String> transformed = future.thenApply(s -> s + " World");

        // Chain another transformation
        CompletableFuture<Integer> length = transformed.thenApply(String::length);

        // Get the final result (blocking)
        System.out.println("Length: " + length.get());  // 11

        // Non-blocking: accept the result
        transformed.thenAccept(s -> System.out.println("Result: " + s));
        // Output: Result: Hello World
    }
}
```

**Expected Output:**
```
Task running in: ForkJoinPool.commonPool-worker-1
Length: 11
Result: Hello World
```

**Step-by-Step Explanation:**

1. `supplyAsync` starts an async task that returns `"Hello"`.
2. `thenApply(s -> s + " World")` chains a transformation that runs when the first task completes.
3. `thenApply(String::length)` chains another transformation.
4. `get()` blocks until the entire chain completes and returns `11`.
5. `thenAccept` attaches a non-blocking consumer that prints the result.

**Example 2: `thenCompose` for Async Chaining**

```java
// File: ThenComposeDemo.java
import java.util.concurrent.*;

public class ThenComposeDemo {
    public static void main(String[] args) throws Exception {
        // First async task
        CompletableFuture<String> userId = CompletableFuture.supplyAsync(() -> {
            return "user-123";
        });

        // thenCompose: chain another async task that depends on the first
        CompletableFuture<String> profile = userId.thenCompose(id ->
            CompletableFuture.supplyAsync(() -> {
                return "Profile for " + id;
            })
        );

        System.out.println(profile.get());  // Profile for user-123
    }
}
```

**Expected Output:**
```
Profile for user-123
```

**Step-by-Step Explanation:**

1. `userId` returns `"user-123"` asynchronously.
2. `thenCompose` is used instead of `thenApply` because the mapping function returns another `CompletableFuture`.
3. The second async task depends on the result of the first.
4. `thenCompose` flattens the nested `CompletableFuture<CompletableFuture<String>>` into `CompletableFuture<String>`.

**Example 3: `thenCombine` for Parallel Composition**

```java
// File: ThenCombineDemo.java
import java.util.concurrent.*;

public class ThenCombineDemo {
    public static void main(String[] args) throws Exception {
        CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> {
            System.out.println("Future 1 running");
            return "Hello";
        });

        CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> {
            System.out.println("Future 2 running");
            return "World";
        });

        // Combine results when both complete
        CompletableFuture<String> combined = future1.thenCombine(future2,
            (s1, s2) -> s1 + " " + s2);

        System.out.println(combined.get());  // Hello World
    }
}
```

**Expected Output (order may vary):**
```
Future 1 running
Future 2 running
Hello World
```

**Step-by-Step Explanation:**

1. Two futures run in parallel.
2. `thenCombine` waits for both to complete and combines their results using a `BiFunction`.
3. The combined result is `"Hello World"`.

### Real-World Cases

- **Microservices Orchestration:** Combining responses from multiple services.
- **E-commerce:** Fetching product details, user profile, and recommendations in parallel.
- **Data Aggregation:** Collecting data from multiple sources and combining results.
- **Async Pipelines:** Processing data through multiple stages asynchronously.

---

## 5. CompletableFuture Creation

### 5.1 `supplyAsync()`

#### Definitions

**Core Definition:** `supplyAsync(Supplier<U>)` creates a `CompletableFuture<U>` that is asynchronously completed by a task running in the `ForkJoinPool.commonPool()` with the value obtained by calling the given `Supplier`.

**Technical Definition:** "Returns a new `CompletableFuture` that is asynchronously completed by a task running in the `ForkJoinPool.commonPool()` with the value obtained by calling the given `Supplier`".

#### Purposes

- To start an asynchronous task that produces a result
- To execute a `Supplier` in a separate thread
- To serve as the entry point for asynchronous pipelines

#### Annotated Code Example

```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    return "Async result";
});
System.out.println(future.get());  // Async result
```

### 5.2 `runAsync()`

#### Definitions

**Core Definition:** `runAsync(Runnable)` creates a `CompletableFuture<Void>` that is asynchronously completed by a task running in the `ForkJoinPool.commonPool()` after running the given `Runnable`.

**Technical Definition:** "Returns a new `CompletableFuture` that is asynchronously completed by a task running in the `ForkJoinPool.commonPool()` after it runs the given action".

#### Purposes

- To start an asynchronous task that does not return a result
- To perform fire-and-forget operations
- To trigger side effects asynchronously

#### Annotated Code Example

```java
CompletableFuture<Void> future = CompletableFuture.runAsync(() -> {
    System.out.println("Async task running");
});
future.get();  // Waits for completion
```

### 5.3 `completedFuture()`

#### Definitions

**Core Definition:** `completedFuture(U)` returns a new `CompletableFuture` that is already completed with the given value.

**Technical Definition:** "Returns a new `CompletableFuture` that is already completed with the given value".

#### Purposes

- To create a pre-completed future for testing
- To provide a default value in asynchronous pipelines
- To avoid null checks in composition chains

#### Annotated Code Example

```java
CompletableFuture<String> future = CompletableFuture.completedFuture("Already done");
System.out.println(future.get());  // Already done
```

### Real-World Cases

- **`supplyAsync`:** Fetching data from a database or API.
- **`runAsync`:** Sending notifications, logging, or cleaning up resources.
- **`completedFuture`:** Returning cached values in an async pipeline.

---

## 6. CompletableFuture Composition

### 6.1 `thenApply()`, `thenAccept()`, `thenRun()`

#### Definitions

**`thenApply(Function)`:** Transforms the result of the previous stage using a function.
**`thenAccept(Consumer)`:** Consumes the result of the previous stage without returning a value.
**`thenRun(Runnable)`:** Runs a `Runnable` after the previous stage completes, ignoring the result.

#### Purposes

- To transform, consume, or react to the result of a `CompletableFuture`
- To chain synchronous operations after an async task
- To perform side effects on the result

#### Annotated Code Example

```java
CompletableFuture.supplyAsync(() -> "Hello")
    .thenApply(s -> s + " World")           // Transform
    .thenApply(String::length)               // Transform again
    .thenAccept(len -> System.out.println("Length: " + len))  // Consume
    .thenRun(() -> System.out.println("Done"));  // Run action
// Output: Length: 11, Done
```

### 6.2 `thenCompose()`

#### Definitions

**Core Definition:** `thenCompose(Function)` chains a function that returns another `CompletableFuture`, flattening the result.

**Technical Definition:** `thenCompose` is like `thenApply`, but the mapping function returns a `CompletableFuture<U>` instead of a plain value `U`. This avoids nested `CompletableFuture<CompletableFuture<U>>`.

#### Purposes

- To chain two asynchronous operations sequentially
- To avoid nested futures
- To implement async flatMap semantics

#### Annotated Code Example

```java
CompletableFuture.supplyAsync(() -> "user-123")
    .thenCompose(id -> CompletableFuture.supplyAsync(() -> "Profile: " + id))
    .thenAccept(System.out::println);  // Profile: user-123
```

### 6.3 `thenCombine()` and `thenCombineAsync()`

#### Definitions

**Core Definition:** `thenCombine(other, BiFunction)` combines the results of two `CompletableFuture`s when both complete.

**Technical Definition:** "Returns a new `CompletableFuture` that is completed when this and the other given stage both complete, with the results combined using the given `BiFunction`" (paraphrased).

#### Purposes

- To combine results from two independent asynchronous tasks
- To implement parallel aggregation
- To merge data from two sources

#### Annotated Code Example

```java
CompletableFuture<String> f1 = CompletableFuture.supplyAsync(() -> "Hello");
CompletableFuture<String> f2 = CompletableFuture.supplyAsync(() -> "World");

CompletableFuture<String> combined = f1.thenCombine(f2, (a, b) -> a + " " + b);
System.out.println(combined.get());  // Hello World
```

### 6.4 `allOf()` and `anyOf()`

#### Definitions

**`allOf(CompletableFuture...)`:** Returns a `CompletableFuture<Void>` that completes when all given futures complete.
**`anyOf(CompletableFuture...)`:** Returns a `CompletableFuture<Object>` that completes when any of the given futures completes.

#### Purposes

- To wait for multiple futures to complete (allOf)
- To implement racing patterns (anyOf)
- To coordinate bulk asynchronous operations

#### Annotated Code Example

```java
CompletableFuture<String> f1 = CompletableFuture.supplyAsync(() -> "A");
CompletableFuture<String> f2 = CompletableFuture.supplyAsync(() -> "B");
CompletableFuture<String> f3 = CompletableFuture.supplyAsync(() -> "C");

// Wait for all
CompletableFuture<Void> all = CompletableFuture.allOf(f1, f2, f3);
all.thenRun(() -> System.out.println("All done"));

// Wait for any
CompletableFuture<Object> any = CompletableFuture.anyOf(f1, f2, f3);
System.out.println("First: " + any.get());  // A (or B or C)
```

### Real-World Cases

- **API Aggregation:** Fetching data from multiple services and combining.
- **Parallel Validation:** Running multiple validations and waiting for all to pass.
- **Caching:** Using `thenCompose` to check cache before fetching.

---

## 7. CompletableFuture Error Handling

### 7.1 `exceptionally()`

#### Definitions

**Core Definition:** `exceptionally(Function<Throwable, ? extends T>)` returns a new `CompletableFuture` that recovers from an exception by providing a fallback value.

**Technical Definition:** "Returns a new `CompletableFuture` that is completed when this stage completes, with the same result or exception, and the given function is applied to the exception if this stage completed exceptionally" (paraphrased).

#### Purposes

- To recover from exceptions with a fallback value
- To provide default results when errors occur
- To prevent exceptions from propagating

#### Annotated Code Example

```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    throw new RuntimeException("Something went wrong");
}).exceptionally(ex -> {
    System.out.println("Recovered: " + ex.getMessage());
    return "Fallback";
});

System.out.println(future.get());  // Fallback
```

### 7.2 `handle()`

#### Definitions

**Core Definition:** `handle(BiFunction<T, Throwable, U>)` returns a new `CompletableFuture` that processes both the result and the exception, allowing transformation of either.

**Technical Definition:** "Returns a new `CompletionStage` that, when this stage completes either normally or exceptionally, is executed with this stage's result and exception as arguments to the supplied function. The function's result is used to complete the returned stage" (paraphrased).

#### Purposes

- To process both success and failure paths in a single function
- To transform exceptions into results
- To implement comprehensive error handling

#### Annotated Code Example

```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    throw new RuntimeException("Error");
}).handle((result, ex) -> {
    if (ex != null) return "Handled: " + ex.getMessage();
    return "Success: " + result;
});

System.out.println(future.get());  // Handled: Error
```

### 7.3 `whenComplete()`

#### Definitions

**Core Definition:** `whenComplete(BiConsumer<T, Throwable>)` returns a new `CompletableFuture` that performs a side effect when the stage completes, without changing the result.

**Technical Definition:** "Returns a new `CompletionStage` with the same result or exception as this stage, that executes the given action when this stage completes. Unlike `handle`, this method is not designed to translate completion outcomes, so the supplied action should not throw an exception" (paraphrased).

#### Purposes

- To perform side effects (logging, cleanup) on completion
- To observe the result or exception without modifying it
- To implement finalisation logic

#### Annotated Code Example

```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> "Hello")
    .whenComplete((result, ex) -> {
        if (ex != null) System.out.println("Failed: " + ex);
        else System.out.println("Completed: " + result);
    });
// Output: Completed: Hello
```

### Real-World Cases

- **`exceptionally`:** Returning cached data when an API call fails.
- **`handle`:** Converting errors into user-friendly messages.
- **`whenComplete`:** Logging task completion for monitoring.

---

## 8. CompletableFuture Best Practices

### Definitions

**Core Definition:** CompletableFuture best practices are guidelines for writing correct, efficient, and maintainable asynchronous code.

**Technical Definition:** Best practices derive from authoritative sources including community experience, production patterns, and Java documentation. Key principles include using custom executors for I/O-bound tasks, avoiding blocking calls inside async tasks, implementing proper error handling, and understanding the threading model.

**Beginner-Friendly Explanation:** Best practices help you avoid common pitfalls like deadlocks, thread starvation, and unhandled exceptions. They're like a safety manual for asynchronous programming.

### Purposes

- To prevent thread starvation and deadlocks
- To ensure proper error handling in async pipelines
- To optimise thread pool usage for different workloads
- To write maintainable and testable asynchronous code

### Key Best Practices Table

| Practice | Description | Source |
|----------|-------------|--------|
| Use custom executors for I/O-bound tasks | "IO-intensive tasks must use custom thread pools" — the common ForkJoinPool is for CPU-bound tasks |  |
| Never block inside async tasks | "Never call `join()`/`get()` inside async tasks. For fire-and-forget, always add error handling" |  |
| Always handle exceptions | "For fire-and-forget, always add error handling" |  |
| Use `thenCompose` for async chaining | "Use `thenApply` for synchronous transformations, `thenCompose` for async chaining" |  |
| Run independent I/O operations in parallel | "Independent I/O operations should run in parallel" |  |
| Avoid the common ForkJoinPool in production | "Always use a custom thread pool, avoid the common thread pool" |  |
| Understand the threading model | "Methods without Async suffix run in the previous task's thread; methods with Async suffix run in the common pool or custom executor" |  |
| Use `exceptionally` for recovery | "`exceptionally` = recovery, exception path only" |  |
| Use `handle` for transformation | "`handle` = always runs, can transform result and handle exception" |  |
| Use `whenComplete` for side effects | "`whenComplete` = side effect only, cannot change outcome" |  |
| Test async code with controlled executors | Use test executors and `CountDownLatch` for deterministic tests | Best practice |
| Monitor thread pool metrics | Track queue size, active threads, completed tasks | Best practice |

### Annotated Complete Step-by-Step Code Examples

```java
// File: CompletableFutureBestPractices.java
import java.util.concurrent.*;

public class CompletableFutureBestPractices {
    public static void main(String[] args) throws Exception {
        // BEST PRACTICE: Use a custom executor for I/O-bound tasks
        ExecutorService ioExecutor = Executors.newFixedThreadPool(10);

        try {
            // BEST PRACTICE: Use supplyAsync with a custom executor
            CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
                // Simulate I/O operation
                return "Data from API";
            }, ioExecutor);

            // BEST PRACTICE: Handle exceptions with exceptionally
            CompletableFuture<String> withFallback = future
                .thenApply(data -> data.toUpperCase())
                .exceptionally(ex -> {
                    System.err.println("Error: " + ex.getMessage());
                    return "FALLBACK";
                });

            // BEST PRACTICE: Never block inside async tasks
            // (get() is called on the main thread here, not inside an async task)
            System.out.println(withFallback.get());

            // BEST PRACTICE: Run independent operations in parallel
            CompletableFuture<String> f1 = CompletableFuture.supplyAsync(
                () -> "Result 1", ioExecutor);
            CompletableFuture<String> f2 = CompletableFuture.supplyAsync(
                () -> "Result 2", ioExecutor);

            CompletableFuture<String> combined = f1.thenCombine(f2,
                (r1, r2) -> r1 + " + " + r2);

            System.out.println(combined.get());  // Result 1 + Result 2

        } finally {
            // BEST PRACTICE: Graceful shutdown
            ioExecutor.shutdown();
            ioExecutor.awaitTermination(5, TimeUnit.SECONDS);
        }
    }
}
```

**Expected Output:**
```
DATA FROM API
Result 1 + Result 2
```

**Step-by-Step Explanation:**

1. A custom `ExecutorService` is created for I/O-bound tasks (10 threads).
2. `supplyAsync` is called with the custom executor to avoid using the common ForkJoinPool.
3. `exceptionally` provides a fallback value if the pipeline fails.
4. `thenCombine` combines two parallel futures.
5. The executor is shut down gracefully in a `finally` block.

### Real-World Cases

- **Microservices:** Using custom executors for HTTP calls to avoid blocking the common pool.
- **E-commerce:** Running product, user, and recommendation queries in parallel.
- **Data Pipelines:** Chaining transformations and handling errors at each stage.

---

## References Links

- Class CompletableFuture<T> — Java Platform, Standard Edition & Java Development Kit Version 21 API Specification - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/CompletableFuture.html
- Interface Future<V> — Java Platform, Standard Edition 8 API Specification - https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Future.html
- Interface Callable<V> — Java Platform, Standard Edition 8 API Specification - https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Callable.html
- Class FutureTask<V> — Java Platform, Standard Edition 8 API Specification - https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/FutureTask.html
- Interface CompletionStage<T> — Java Platform, Standard Edition 8 API Specification - https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletionStage.html
- Java CompletableFuture - Progressive Learning Path — mo0rti/Learning-Completable-Future (GitHub) - https://github.com/mo0rti/Learning-Completable-Future
- CompletableFuture 异步编程全解 — Alibaba Cloud Developer Community - https://developer.aliyun.com/article/1725807
- JUC并发—13.Future模式和异步编程简介 — Alibaba Cloud Developer Community - https://developer.aliyun.com/article/1662727
- Overview of Advanced Java 8 CompletableFuture Features (Part 1) — Douglas C. Schmidt, Vanderbilt University - http://www.dre.vanderbilt.edu/~schmidt/cs891f/2017-PDFs/25-overview-of-advanced-Java-8-completablefuture-features-pt1.pdf
- Overview of Advanced Java 8 CompletableFuture Features (Part 4) — Douglas C. Schmidt, Vanderbilt University - http://www.dre.vanderbilt.edu/~schmidt/cs891f/2017-PDFs/28-overview-of-advanced-Java-8-completablefuture-features-pt4.pdf
- Java CompletableFuture — Baeldung - https://www.baeldung.com/java-completablefuture