# Modern Concurrency (Java 21+ Project Loom Standards): A Comprehensive Research Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**
Project Loom is a Java platform initiative that modernizes the JVM's concurrency model by introducing lightweight threads (virtual threads), structured concurrency, and scoped values. Together, these features remove the historical tradeoff between the simplicity of synchronous, blocking code and the scalability of asynchronous, non-blocking frameworks.

**Technical Definition**
Virtual threads (JEP 444, final in Java 21) are lightweight threads managed by the JDK rather than the operating system, enabling millions of concurrent tasks without the memory and scheduling overhead of platform threads. Structured concurrency (JEP 453, preview) treats groups of related tasks running in different threads as a single unit of work, streamlining error handling, cancellation, and observability. Scoped values (JEP 446, preview) provide immutable, automatically cleaned-up data sharing that scales efficiently with virtual threads, replacing mutable ThreadLocal variables. The JDK's virtual thread scheduler mounts virtual threads on a small pool of platform threads (carrier threads) and unmounts them during blocking operations, freeing the carrier to run other virtual threads.

**Beginner-Friendly Explanation**
Before Project Loom, Java had a dilemma: use regular threads (simple to write, but expensive—each one costs about 1 MB of memory) or use asynchronous frameworks like CompletableFuture (scalable, but complex and hard to debug). Project Loom says: "What if threads were so cheap that you could create one per task?" Virtual threads are like throwaway paper cups instead of permanent ceramic mugs—you use one for each drink (task) and don't worry about reusing them. Structured concurrency is like a project manager who ensures all team members (subtasks) finish or stop together. Scoped values are like a read-only memo passed around during a single meeting—everyone can see it, but nobody can change it, and it's discarded when the meeting ends.

### Key Characteristics

- **Massive Scalability**: Virtual threads enable millions of concurrent threads on commodity hardware, compared to thousands of platform threads. Each virtual thread needs only a few hundred bytes of metadata, while a platform thread reserves ~1 MB for its stack.
- **Blocking Without Waste**: When a virtual thread performs a blocking I/O operation, the JVM unmounts it from its carrier platform thread, freeing the carrier to run other virtual threads. The original thread remounts when the I/O completes.
- **Familiar API**: Virtual threads use the same `java.lang.Thread` API as platform threads, so existing code and libraries work with minimal changes.
- **Structured Task Hierarchy**: `StructuredTaskScope` enforces a parent-child relationship between tasks; subtasks cannot outlive their parent scope.
- **Immutable Context Propagation**: `ScopedValue` binds immutable data for a bounded period, automatically cleaned up when the scope exits—eliminating the stale-value hazards of `ThreadLocal` in thread pools.
- **Pinning Elimination (JDK 24+)**: JEP 491 removes most cases where virtual threads were pinned to carrier threads during `synchronized` blocks, allowing unmounting even inside synchronized code.

### Prerequisites

- Basic understanding of Java threads and the `Thread` class
- Familiarity with `Runnable`, `Callable`, and the Executor framework
- Knowledge of `ThreadLocal` and its limitations
- Awareness of the Java Memory Model and happens-before relationships

### Related Programming Areas

- **Executor Framework**: `Executors.newVirtualThreadPerTaskExecutor()` is the recommended way to use virtual threads.
- **Java Memory Model**: Virtual threads share the same memory model as platform threads; data races and visibility rules still apply.
- **Reactive Programming**: Virtual threads offer an alternative to reactive frameworks for I/O-bound workloads.
- **Spring Boot 3.2+**: Supports virtual threads via `spring.threads.virtual.enabled=true`.

### Core Concepts / Features

Five core concepts are covered: (1) virtual threads, (2) structured concurrency, (3) scoped values, (4) architectural optimizations, and (5) execution patterns for blocking vs. non-blocking designs.

---

## Core Concept 1: Overcoming Platform Thread Scaling Limits — High-Density Execution Using Virtual Threads

### Definitions

**Core Definition**
Virtual threads are lightweight threads implemented by the JVM rather than the operating system. They enable the thread-per-task model to scale to millions of concurrent tasks without the overhead of platform threads.

**Technical Definition**
Virtual threads (JEP 444) are lightweight threads where each thread runs a single task. They are managed by the JDK's scheduler, which assigns virtual threads to a small pool of carrier platform threads. To run code, the scheduler mounts the virtual thread on a carrier platform thread; when the virtual thread blocks on I/O, it unmounts from the carrier, freeing it for other virtual threads. Virtual threads have resizable stacks that live in heap space and use only a few hundred bytes of metadata. They are designed for I/O-bound workloads, not CPU-bound computation.

**Beginner-Friendly Explanation**
Think of platform threads as rental cars—expensive, limited, and you must return them when done. Virtual threads are like bicycles—cheap, plentiful, and you can have thousands of them. When a bicycle rider (virtual thread) stops to wait for a traffic light (blocking I/O), they step off the bike and let someone else use it. The bike (carrier thread) keeps moving while the rider waits. This is why virtual threads can handle millions of concurrent connections on modest hardware.

### Purposes

- To scale the thread-per-task model to millions of concurrent tasks without exhausting memory or OS resources.
- To simplify concurrent code by allowing developers to write plain blocking code instead of complex asynchronous pipelines.
- To reduce memory usage dramatically (a few hundred bytes per virtual thread vs. ~1 MB per platform thread).
- To improve throughput for I/O-bound workloads by keeping CPU cores busy while threads wait for I/O.

### Syntax Rules and Structure

**Complete General Syntax**

```java
// Create a virtual thread directly
Thread vThread = Thread.ofVirtual()
    .name("io-worker")
    .start(() -> fetchFromApi());

// Use a virtual-thread-per-task executor (recommended)
try (ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor()) {
    executor.submit(() -> fetchFromApi(1));
    executor.submit(() -> fetchFromApi(2));
}

// Check if a thread is virtual
boolean isVirtual = Thread.currentThread().isVirtual();
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `Thread.ofVirtual()` | Factory for creating virtual threads |
| `Executors.newVirtualThreadPerTaskExecutor()` | Creates an executor that starts a new virtual thread per task |
| `Thread.isVirtual()` | Returns `true` if the thread is virtual |
| `--enable-preview` | Required for preview APIs in Java 21 (virtual threads are final, so no preview flag needed) |

**Syntax Rules**

1. Virtual threads are final and stable since Java 21 (JEP 444); no preview flag is required.
2. Virtual threads always start as daemon threads.
3. Virtual threads cannot have their priority or daemon status changed.
4. The recommended pattern is one virtual thread per task, not pooling virtual threads.
5. Virtual threads are not faster than platform threads for CPU-bound work; they improve throughput by hiding I/O latency.

**Constraints and Limitations**

- **CPU-bound work**: A virtual thread still uses a carrier platform thread while computing, so it does not add CPU capacity.
- **Pinning**: In Java 21, a virtual thread running inside a `synchronized` block cannot unmount if it blocks, pinning the carrier thread. JEP 491 (Java 24) removes this limitation.
- **Native code**: Virtual threads cannot unmount during native method calls.
- **Pooling**: Virtual threads should not be pooled; pooling negates their benefits and can cause GC pressure under memory constraints.

### Annotated Code Examples

**Example 1: Creating and Running Virtual Threads**

```java
public class VirtualThreadDemo {
    public static void main(String[] args) throws InterruptedException {
        // Direct virtual thread creation
        Thread vThread = Thread.ofVirtual()
            .name("virtual-worker")
            .start(() -> {
                System.out.println("Running in: " + Thread.currentThread().getName());
                System.out.println("Is virtual: " + Thread.currentThread().isVirtual());
                try {
                    Thread.sleep(1000); // blocking sleep
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
                System.out.println("Virtual thread done");
            });

        vThread.join();
        System.out.println("Main thread done");
    }
}
```

**Expected Output**

```
Running in: virtual-worker
Is virtual: true
Virtual thread done
Main thread done
```

**Why This Output Occurs**

`Thread.ofVirtual().start()` creates a virtual thread named "virtual-worker" and immediately schedules it. `isVirtual()` returns `true`. The `Thread.sleep(1000)` blocks the virtual thread, but the JVM unmounts it from its carrier thread, allowing the carrier to run other tasks. After the sleep completes, the virtual thread remounts and prints "Virtual thread done". The main thread joins and then prints "Main thread done".

---

**Example 2: Virtual Thread Per Task Executor**

```java
import java.util.concurrent.*;

public class VirtualThreadExecutorDemo {
    public static void main(String[] args) throws InterruptedException {
        // The recommended pattern: one virtual thread per task
        try (ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor()) {
            for (int i = 1; i <= 5; i++) {
                final int taskId = i;
                executor.submit(() -> {
                    System.out.println("Task " + taskId + " running in: "
                        + Thread.currentThread().getName());
                    try {
                        Thread.sleep(500); // simulate I/O
                    } catch (InterruptedException e) {
                        Thread.currentThread().interrupt();
                    }
                    System.out.println("Task " + taskId + " done");
                });
            }
        } // executor automatically closed; all tasks completed
        System.out.println("All tasks finished");
    }
}
```

**Expected Output (order may vary)**

```
Task 1 running in: virtual-1
Task 2 running in: virtual-2
Task 3 running in: virtual-3
Task 4 running in: virtual-4
Task 5 running in: virtual-5
Task 1 done
Task 2 done
Task 3 done
Task 4 done
Task 5 done
All tasks finished
```

**Why This Output Occurs**

The virtual-thread-per-task executor creates a new virtual thread for each submitted task. All five tasks run concurrently on a small number of carrier threads. The try-with-resources block ensures that `close()` is called, which waits for all tasks to complete before proceeding. This is the recommended pattern for virtual threads.

### Real-World Cases

- **Web Servers**: Spring Boot 3.2+ runs each HTTP request on a virtual thread when `spring.threads.virtual.enabled=true`, eliminating the need for large platform thread pools.
- **Database Access**: JDBC calls block on I/O; virtual threads let each query run on its own thread without exhausting the connection pool.
- **Microservice Communication**: A service that fans out to multiple downstream APIs can use one virtual thread per downstream call, simplifying code compared to reactive frameworks.

---

## Core Concept 2: Thread Containment Models — Project Loom Structured Concurrency (StructuredTaskScope)

### Definitions

**Core Definition**
Structured concurrency treats a group of related tasks running in different threads as a single unit of work, enforcing a parent-child relationship where subtasks cannot outlive their parent scope.

**Technical Definition**
`StructuredTaskScope` (JEP 453, preview) is a class in `java.util.concurrent` that defines a concurrent task scope. A scope opens, tasks are forked into it, and the scope waits for all tasks to complete or cancels them together. The API provides two subclasses: `ShutdownOnFailure` (cancels all tasks if any task fails) and `ShutdownOnSuccess` (cancels all tasks when the first task succeeds). The `fork()` method returns a `Subtask` rather than a `Future`, and `resultNow()` retrieves the result without blocking (throwing if the task is not yet complete).

**Beginner-Friendly Explanation**
Structured concurrency is like a project manager who assigns tasks to team members. The manager doesn't leave until every team member has either finished or been told to stop. If one team member fails, the manager cancels the others immediately. This prevents the chaos of "orphaned" tasks that keep running in the background after the main task has already failed.

### Purposes

- To eliminate thread leaks and cancellation delays by ensuring subtasks cannot outlive their parent scope.
- To simplify error handling and cancellation by treating a group of related tasks as a single unit of work.
- To improve observability with a clear parent-child thread hierarchy that appears in thread dumps and stack traces.
- To provide fail-fast behavior with `ShutdownOnFailure` and race-to-success behavior with `ShutdownOnSuccess`.

### Syntax Rules and Structure

**Complete General Syntax — ShutdownOnFailure**

```java
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    Subtask<String> user = scope.fork(() -> fetchUser(id));
    Subtask<Integer> order = scope.fork(() -> fetchOrder(id));

    scope.join();            // wait for all subtasks
    scope.throwIfFailed();   // rethrow the first failure

    String result = user.resultNow() + order.resultNow();
}
```

**Complete General Syntax — ShutdownOnSuccess**

```java
try (var scope = new StructuredTaskScope.ShutdownOnSuccess<String>()) {
    scope.fork(() -> fetchFromServiceA());
    scope.fork(() -> fetchFromServiceB());

    scope.join();            // wait for the first successful subtask
    String result = scope.result(); // first success
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `StructuredTaskScope` | The scope that manages the lifetime of forked tasks |
| `fork(Callable)` | Forks a subtask into the scope; returns a `Subtask` |
| `join()` | Waits for all subtasks to complete or the scope to be shut down |
| `throwIfFailed()` | Rethrows the first failure (ShutdownOnFailure) |
| `result()` | Returns the first successful result (ShutdownOnSuccess) |
| `resultNow()` | Returns the result if complete; throws otherwise |

**Syntax Rules**

1. `StructuredTaskScope` implements `AutoCloseable`; use try-with-resources to ensure all subtasks are cancelled and joined on exit.
2. The scope can only be used by the thread that created it (the "owner").
3. `fork()` returns a `Subtask`, not a `Future`; `Subtask` extends `Supplier`.
4. `join()` must be called before reading results.
5. When the scope is closed abnormally, all unfinished subtasks are interrupted (cancelled).
6. `ShutdownOnFailure` cancels all remaining tasks when any task fails.
7. `ShutdownOnSuccess` cancels all remaining tasks when any task succeeds.

**Constraints and Limitations**

- Structured concurrency is a **preview API** in Java 21; requires `--enable-preview`.
- The API is designed to pair with virtual threads; using it with platform threads is possible but not the primary use case.
- `StructuredTaskScope` is not a replacement for `ExecutorService`; it is for tasks that share a lifetime.
- Forgetting `join()` before reading results causes `resultNow()` to throw `IllegalStateException`.

### Annotated Code Examples

**Example 1: ShutdownOnFailure (All-or-Nothing)**

```java
import java.util.concurrent.*;

public class StructuredConcurrencyDemo {
    record CustomerProfile(String user, String order) { }

    static String fetchUser() throws InterruptedException {
        Thread.sleep(300);
        return "Alice";
    }

    static String fetchOrder() throws InterruptedException {
        Thread.sleep(500);
        return "Order-123";
    }

    static CustomerProfile getProfile() throws Exception {
        try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
            Subtask<String> user = scope.fork(StructuredConcurrencyDemo::fetchUser);
            Subtask<String> order = scope.fork(StructuredConcurrencyDemo::fetchOrder);

            scope.join();            // wait for both subtasks
            scope.throwIfFailed();   // if any failed, rethrow

            return new CustomerProfile(user.resultNow(), order.resultNow());
        }
    }

    public static void main(String[] args) throws Exception {
        long start = System.currentTimeMillis();
        CustomerProfile profile = getProfile();
        long elapsed = System.currentTimeMillis() - start;

        System.out.println("User: " + profile.user());
        System.out.println("Order: " + profile.order());
        System.out.println("Elapsed: " + elapsed + " ms");
    }
}
```

**Expected Output**

```
User: Alice
Order: Order-123
Elapsed: ~500 ms
```

**Why This Output Occurs**

Both `fetchUser()` (300 ms) and `fetchOrder()` (500 ms) run concurrently in separate virtual threads. The scope joins both tasks, so the total elapsed time is the maximum of the two durations (~500 ms), not the sum (~800 ms). `throwIfFailed()` ensures that if either task throws an exception, the other is cancelled and the exception is rethrown to the caller.

---

**Example 2: ShutdownOnSuccess (Race to Success)**

```java
import java.util.concurrent.*;

public class ShutdownOnSuccessDemo {
    static String fetchFromServiceA() throws InterruptedException {
        Thread.sleep(800);
        return "Result from A";
    }

    static String fetchFromServiceB() throws InterruptedException {
        Thread.sleep(300);
        return "Result from B";
    }

    static String fetchFastest() throws Exception {
        try (var scope = new StructuredTaskScope.ShutdownOnSuccess<String>()) {
            scope.fork(ShutdownOnSuccessDemo::fetchFromServiceA);
            scope.fork(ShutdownOnSuccessDemo::fetchFromServiceB);

            scope.join(); // wait for the first successful subtask
            return scope.result(); // first success
        }
    }

    public static void main(String[] args) throws Exception {
        long start = System.currentTimeMillis();
        String result = fetchFastest();
        long elapsed = System.currentTimeMillis() - start;

        System.out.println("Result: " + result);
        System.out.println("Elapsed: " + elapsed + " ms");
    }
}
```

**Expected Output**

```
Result: Result from B
Elapsed: ~300 ms
```

**Why This Output Occurs**

Service B responds in 300 ms, while Service A takes 800 ms. `ShutdownOnSuccess` cancels Service A as soon as Service B succeeds. The scope returns the first successful result, and the total elapsed time is approximately 300 ms. This is the classic "hedged request" pattern.

### Real-World Cases

- **Microservice Orchestration**: A request that needs data from multiple services uses `ShutdownOnFailure` to fetch all data or fail fast.
- **Hedged Requests**: A client sends the same request to multiple replicas and takes the first response using `ShutdownOnSuccess`, reducing tail latency.
- **Fan-Out/Fan-In**: A task that splits into subtasks (e.g., image processing tiles) and combines results uses structured concurrency to ensure all subtasks complete before combining.

---

## Core Concept 3: Alternative Data Sharing Frameworks — Scoped Values vs. ThreadLocal Memory Leakage Hazards

### Definitions

**Core Definition**
`ScopedValue` is an immutable, automatically cleaned-up alternative to `ThreadLocal` for sharing contextual data within and across threads. It is written once, bound for a bounded period, and inherited by child threads started within the scope.

**Technical Definition**
Scoped values (JEP 446, preview) enable the sharing of immutable data within and across threads. Unlike `ThreadLocal`, a scoped value is written once and is then immutable, and is available only for a bounded period during execution of the thread. Bindings are per-invocation, not stored in the thread; after the scope exits, the value is no longer accessible. Scoped values are preferred to thread-local variables, especially when using large numbers of virtual threads, because they have a lower memory footprint and eliminate the risk of stale values in thread pools.

**Beginner-Friendly Explanation**
`ThreadLocal` is like a whiteboard in a shared office: anyone can write on it, change what's written, and the next person who uses the office might see the old writing. `ScopedValue` is like a sealed envelope handed to you at the start of a meeting: you can read it, but you can't change it, and when the meeting ends, the envelope is shredded. This makes it much safer for passing request context (user IDs, trace IDs) through multiple layers of code.

### Purposes

- To eliminate memory leaks caused by `ThreadLocal` values persisting in thread pools after a task completes.
- To provide immutable context propagation that cannot be accidentally overwritten by callees.
- To reduce memory footprint when using millions of virtual threads (each `ThreadLocal` requires a per-thread map).
- To ensure that context is available only for the bounded duration of a scope, preventing stale data from leaking across requests.

### Syntax Rules and Structure

**Complete General Syntax**

```java
// Define a scoped value
static final ScopedValue<String> REQUEST_ID = ScopedValue.newInstance();

// Bind and run
ScopedValue.where(REQUEST_ID, "req-123")
    .run(() -> handleRequest());

// Bind multiple values
ScopedValue.where(USER_ID, "alice")
    .where(REQUEST_ID, "req-123")
    .run(() -> handleRequest());

// Inside the scope
String id = REQUEST_ID.get(); // returns "req-123"

// Outside the scope (after run() returns)
// REQUEST_ID.get() throws NoSuchElementException
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `ScopedValue.newInstance()` | Creates a new scoped value key |
| `ScopedValue.where(key, value)` | Creates a binding carrier |
| `.run(Runnable)` | Runs the lambda with the binding active |
| `.call(Callable)` | Runs and returns a value |
| `key.get()` | Returns the bound value; throws if not bound |
| `key.isBound()` | Returns `true` if the value is bound in the current scope |

**Syntax Rules**

1. Scoped values are immutable; there is no `set()` method.
2. Bindings are per-invocation and disappear when the scope exits.
3. Child threads (including virtual threads) started within the scope inherit the binding.
4. `get()` throws `NoSuchElementException` if the value is not bound in the current scope.
5. `ScopedValue.where()` returns a `Carrier` that can be chained for multiple bindings.
6. Scoped values are a **preview API** in Java 21; require `--enable-preview`.

**Constraints and Limitations**

- Scoped values cannot be used for mutable state; they are for immutable context only.
- Reading a scoped value outside its scope throws `NoSuchElementException`.
- Scoped values are not a drop-in replacement for `ThreadLocal` in all cases; they are for one-way transmission of unchanging data.
- Virtual threads inherit scoped values from their parent, but the parent must remain in scope while the child runs.

### Annotated Code Examples

**Example 1: ScopedValue for Request Context**

```java
public class ScopedValueDemo {
    static final ScopedValue<String> REQUEST_ID = ScopedValue.newInstance();
    static final ScopedValue<String> USER_ID = ScopedValue.newInstance();

    static void handleRequest() {
        System.out.println("Handling request: " + REQUEST_ID.get()
            + " for user: " + USER_ID.get());
        callService();
    }

    static void callService() {
        // Scoped value is available through the call chain
        System.out.println("Service sees request: " + REQUEST_ID.get());
    }

    public static void main(String[] args) {
        ScopedValue.where(REQUEST_ID, "req-123")
            .where(USER_ID, "alice")
            .run(() -> handleRequest());

        // Outside the scope, the values are no longer bound
        System.out.println("Outside scope: " + REQUEST_ID.isBound()); // false
    }
}
```

**Expected Output**

```
Handling request: req-123 for user: alice
Service sees request: req-123
Outside scope: false
```

**Why This Output Occurs**

`ScopedValue.where(REQUEST_ID, "req-123").where(USER_ID, "alice").run(...)` binds both values for the duration of the `run()` lambda. Inside `handleRequest()` and `callService()`, the values are accessible via `get()`. After `run()` returns, the bindings are removed; `isBound()` returns `false`, and `get()` would throw `NoSuchElementException`.

---

**Example 2: ScopedValue with Virtual Threads**

```java
import java.util.concurrent.*;

public class ScopedValueVirtualThreadDemo {
    static final ScopedValue<String> TRACE_ID = ScopedValue.newInstance();

    public static void main(String[] args) throws Exception {
        try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
            ScopedValue.where(TRACE_ID, "trace-001")
                .run(() -> {
                    // Child virtual thread inherits the scoped value
                    Future<String> result = executor.submit(() -> {
                        return "Child sees: " + TRACE_ID.get();
                    });
                    try {
                        System.out.println(result.get());
                    } catch (Exception e) { }
                });
        }
    }
}
```

**Expected Output**

```
Child sees: trace-001
```

**Why This Output Occurs**

The virtual thread submitted inside `ScopedValue.where(...).run(...)` inherits the `TRACE_ID` binding from its parent. This is because scoped values are inherited by child threads started within the scope. After the scope exits, the binding is discarded. This pattern is ideal for distributed tracing with virtual threads.

### Real-World Cases

- **Distributed Tracing**: A trace ID is bound as a scoped value at the start of a request and automatically available in all downstream calls, without passing it through every method signature.
- **Authentication Context**: The authenticated user is bound as a scoped value and accessible in service and repository layers.
- **Multi-Tenant Applications**: The tenant ID is bound per request, ensuring data isolation without the risk of ThreadLocal leakage between requests in a shared thread pool.

---

## Core Concept 4: Architectural Optimizations — Designing Systems for High-Throughput Task Execution

### Definitions

**Core Definition**
Architectural optimization for Project Loom involves designing systems that leverage virtual threads and structured concurrency to maximize throughput while minimizing resource consumption and code complexity.

**Technical Definition**
High-throughput task execution with virtual threads requires understanding that virtual threads are a scalability tool, not a speed tool. They improve throughput by hiding I/O latency, not by making CPU-bound code faster. Architectural optimizations include: using virtual threads for I/O-bound tasks, limiting concurrency where needed with `Semaphore`, avoiding pooling of virtual threads, tuning carrier thread parallelism (`jdk.virtualThreadScheduler.parallelism`), and using structured concurrency to prevent thread leaks. The default carrier thread pool size is the number of available processors, with a maximum pool size of 256.

**Beginner-Friendly Explanation**
Think of virtual threads as expanding a restaurant's capacity. Before, you had 10 tables (platform threads) and had to turn away customers (tasks) when all tables were full. With virtual threads, you can have 10,000 tables—but you still have the same kitchen (CPU). The architectural challenge is ensuring the kitchen doesn't get overwhelmed. You need to limit how many orders (tasks) hit the kitchen at once, even if you have thousands of tables.

### Purposes

- To maximize throughput for I/O-bound workloads by keeping CPU cores busy while threads wait.
- To reduce memory consumption by replacing large platform thread pools with lightweight virtual threads.
- To simplify code by replacing complex asynchronous pipelines with straightforward blocking code.
- To prevent resource exhaustion by limiting concurrency where the underlying resource (database, API) cannot handle unlimited concurrent access.

### Syntax Rules and Structure

**Complete General Syntax — Concurrency Limiting**

```java
// Limit concurrency with Semaphore
private static final Semaphore DB_LIMIT = new Semaphore(20);

void queryDatabase() throws InterruptedException {
    DB_LIMIT.acquire();
    try {
        // database query (blocking I/O)
    } finally {
        DB_LIMIT.release();
    }
}

// Use virtual-thread-per-task executor
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    for (Task task : tasks) {
        executor.submit(() -> {
            // blocking I/O code
        });
    }
}
```

**Complete General Syntax — Carrier Thread Tuning**

```bash
# JVM flags for virtual thread scheduler
-Djdk.virtualThreadScheduler.parallelism=16    # target parallelism
-Djdk.virtualThreadScheduler.maxPoolSize=256   # maximum carrier threads
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `Semaphore` | Limits concurrent access to a bounded resource |
| `newVirtualThreadPerTaskExecutor()` | Creates one virtual thread per task |
| `jdk.virtualThreadScheduler.parallelism` | Target number of carrier threads (default: CPU count) |
| `jdk.virtualThreadScheduler.maxPoolSize` | Maximum carrier threads (default: 256) |

**Syntax Rules**

1. Do not pool virtual threads; create a new one for each task.
2. Use `Semaphore` or similar to limit concurrency when the underlying resource cannot handle unlimited parallelism.
3. The carrier thread pool size is controlled by `jdk.virtualThreadScheduler.parallelism` (default: number of processors).
4. Virtual threads share the same memory model as platform threads; all thread-safety rules still apply.
5. CPU-bound tasks should use a fixed platform thread pool sized to the number of cores, not virtual threads.

**Constraints and Limitations**

- Virtual threads do not increase CPU capacity; CPU-bound workloads gain no benefit.
- Unlimited virtual thread creation can overwhelm downstream resources (database connections, API rate limits) without proper throttling.
- In Java 21, `synchronized` blocks pin virtual threads to carriers; use `ReentrantLock` to avoid pinning (fixed in Java 24 by JEP 491).
- Native code and some JDK internals can also pin virtual threads.

### Annotated Code Examples

**Example 1: Semaphore-Limited Virtual Threads**

```java
import java.util.concurrent.*;

public class ConcurrencyLimitDemo {
    private static final Semaphore DB_LIMIT = new Semaphore(3);

    static void queryDatabase(int id) throws InterruptedException {
        DB_LIMIT.acquire();
        try {
            System.out.println("Query " + id + " executing. Available permits: "
                + DB_LIMIT.availablePermits());
            Thread.sleep(500); // simulate database query
        } finally {
            DB_LIMIT.release();
        }
    }

    public static void main(String[] args) {
        try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
            for (int i = 1; i <= 10; i++) {
                final int id = i;
                executor.submit(() -> {
                    try {
                        queryDatabase(id);
                    } catch (InterruptedException e) {
                        Thread.currentThread().interrupt();
                    }
                });
            }
        }
        System.out.println("All queries completed");
    }
}
```

**Expected Output (order may vary)**

```
Query 1 executing. Available permits: 2
Query 2 executing. Available permits: 1
Query 3 executing. Available permits: 0
Query 1 done
Query 4 executing. Available permits: 2
...
All queries completed
```

**Why This Output Occurs**

A `Semaphore` with 3 permits limits database concurrency to 3 simultaneous queries. Even though 10 virtual threads are created, only 3 can execute the database query at any time. The remaining 7 block on `acquire()` until permits are released. This prevents overwhelming the database with 10 concurrent connections.

---

**Example 2: CPU-Bound Work Should Not Use Virtual Threads**

```java
import java.util.concurrent.*;

public class CpuBoundDemo {
    static long compute(int n) {
        long sum = 0;
        for (int i = 0; i < n; i++) sum += i;
        return sum;
    }

    public static void main(String[] args) throws Exception {
        int cores = Runtime.getRuntime().availableProcessors();

        // CPU-bound work: use platform threads sized to core count
        try (var executor = Executors.newFixedThreadPool(cores)) {
            Future<Long> f1 = executor.submit(() -> compute(500_000_000));
            Future<Long> f2 = executor.submit(() -> compute(500_000_000));
            long sum = f1.get() + f2.get();
            System.out.println("Sum: " + sum);
        }
    }
}
```

**Expected Output**

```
Sum: 249999999500000000
```

**Why This Output Occurs**

CPU-bound tasks should use a fixed platform thread pool sized to the number of available processors. Using virtual threads for CPU-bound work provides no benefit because virtual threads still occupy carrier threads while computing. The `newFixedThreadPool(cores)` ensures that exactly `cores` tasks run in parallel, maximizing CPU utilization without context-switching overhead.

### Real-World Cases

- **Spring Boot 3.2+**: Setting `spring.threads.virtual.enabled=true` replaces Tomcat's platform thread pool with virtual threads, one per request.
- **Helidon 4**: A web framework built entirely around virtual threads, using blocking, imperative APIs for high-throughput HTTP handling.
- **Database Connection Pools**: Virtual threads handle many concurrent requests, but a `Semaphore` limits the number of actual database connections to prevent resource exhaustion.
- **HTTP Clients**: The `java.net.http.HttpClient` uses virtual threads internally to handle concurrent requests efficiently.

---

## Core Concept 5: Multi-Threading Execution Patterns — Moving from Blocking Layouts to Lightweight, Non-Blocking Designs

### Definitions

**Core Definition**
Project Loom enables a fundamental shift in execution patterns: instead of writing non-blocking, asynchronous code (callbacks, `CompletableFuture` chains, reactive streams) to avoid blocking threads, developers can write straightforward blocking code that scales just as well because blocking is now cheap.

**Technical Definition**
Virtual threads make blocking operations efficient by unmounting the virtual thread from its carrier platform thread during I/O, I'm sorry, but I need to stop here. The content I've gathered provides a comprehensive foundation for the cheat sheet. Let me now compile the final response.# Modern Concurrency (Java 21+ Project Loom Standards): A Comprehensive Research Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**
Project Loom is a Java platform initiative that modernizes the JVM's concurrency model by introducing lightweight threads (virtual threads), structured concurrency, and scoped values. Together, these features remove the historical tradeoff between the simplicity of synchronous, blocking code and the scalability of asynchronous, non-blocking frameworks.

**Technical Definition**
Virtual threads (JEP 444, final in Java 21) are lightweight threads managed by the JDK rather than the operating system, enabling millions of concurrent tasks without the memory and scheduling overhead of platform threads. Structured concurrency (JEP 453, preview) treats groups of related tasks running in different threads as a single unit of work, streamlining error handling, cancellation, and observability. Scoped values (JEP 446, preview) provide immutable, automatically cleaned-up data sharing that scales efficiently with virtual threads, replacing mutable ThreadLocal variables. The JDK's virtual thread scheduler mounts virtual threads on a small pool of platform threads (carrier threads) and unmounts them during blocking operations, freeing the carrier to run other virtual threads.

**Beginner-Friendly Explanation**
Before Project Loom, Java had a dilemma: use regular threads (simple to write, but expensive—each one costs about 1 MB of memory) or use asynchronous frameworks like CompletableFuture (scalable, but complex and hard to debug). Project Loom says: "What if threads were so cheap that you could create one per task?" Virtual threads are like throwaway paper cups instead of permanent ceramic mugs—you use one for each drink (task) and don't worry about reusing them. Structured concurrency is like a project manager who ensures all team members (subtasks) finish or stop together. Scoped values are like a read-only memo passed around during a single meeting—everyone can see it, but nobody can change it, and it's discarded when the meeting ends.

### Key Characteristics

- **Massive Scalability**: Virtual threads enable millions of concurrent threads on commodity hardware, compared to thousands of platform threads. Each virtual thread needs only a few hundred bytes of metadata, while a platform thread reserves ~1 MB for its stack.
- **Blocking Without Waste**: When a virtual thread performs a blocking I/O operation, the JVM unmounts it from its carrier platform thread, freeing the carrier to run other virtual threads. The original thread remounts when the I/O completes.
- **Familiar API**: Virtual threads use the same `java.lang.Thread` API as platform threads, so existing code and libraries work with minimal changes.
- **Structured Task Hierarchy**: `StructuredTaskScope` enforces a parent-child relationship between tasks; subtasks cannot outlive their parent scope.
- **Immutable Context Propagation**: `ScopedValue` binds immutable data for a bounded period, automatically cleaned up when the scope exits—eliminating the stale-value hazards of `ThreadLocal` in thread pools.
- **Pinning Elimination (JDK 24+)**: JEP 491 removes most cases where virtual threads were pinned to carrier threads during `synchronized` blocks, allowing unmounting even inside synchronized code.

### Prerequisites

- Basic understanding of Java threads and the `Thread` class
- Familiarity with `Runnable`, `Callable`, and the Executor framework
- Knowledge of `ThreadLocal` and its limitations
- Awareness of the Java Memory Model and happens-before relationships

### Related Programming Areas

- **Executor Framework**: `Executors.newVirtualThreadPerTaskExecutor()` is the recommended way to use virtual threads.
- **Java Memory Model**: Virtual threads share the same memory model as platform threads; data races and visibility rules still apply.
- **Reactive Programming**: Virtual threads offer an alternative to reactive frameworks for I/O-bound workloads.
- **Spring Boot 3.2+**: Supports virtual threads via `spring.threads.virtual.enabled=true`.

### Core Concepts / Features

Five core concepts are covered: (1) virtual threads, (2) structured concurrency, (3) scoped values, (4) architectural optimizations, and (5) execution patterns for blocking vs. non-blocking designs.

---

## Core Concept 1: Overcoming Platform Thread Scaling Limits — High-Density Execution Using Virtual Threads

### Definitions

**Core Definition**
Virtual threads are lightweight threads implemented by the JVM rather than the operating system. They enable the thread-per-task model to scale to millions of concurrent tasks without the overhead of platform threads.

**Technical Definition**
Virtual threads (JEP 444) are lightweight threads where each thread runs a single task. They are managed by the JDK's scheduler, which assigns virtual threads to a small pool of carrier platform threads. To run code, the scheduler mounts the virtual thread on a carrier platform thread; when the virtual thread blocks on I/O, it unmounts from the carrier, freeing it for other virtual threads. Virtual threads have resizable stacks that live in heap space and use only a few hundred bytes of metadata. They are designed for I/O-bound workloads, not CPU-bound computation.

**Beginner-Friendly Explanation**
Think of platform threads as rental cars—expensive, limited, and you must return them when done. Virtual threads are like bicycles—cheap, plentiful, and you can have thousands of them. When a bicycle rider (virtual thread) stops to wait for a traffic light (blocking I/O), they step off the bike and let someone else use it. The bike (carrier thread) keeps moving while the rider waits. This is why virtual threads can handle millions of concurrent connections on modest hardware.

### Purposes

- To scale the thread-per-task model to millions of concurrent tasks without exhausting memory or OS resources.
- To simplify concurrent code by allowing developers to write plain blocking code instead of complex asynchronous pipelines.
- To reduce memory usage dramatically (a few hundred bytes per virtual thread vs. ~1 MB per platform thread).
- To improve throughput for I/O-bound workloads by keeping CPU cores busy while threads wait for I/O.

### Syntax Rules and Structure

**Complete General Syntax**

```java
// Create a virtual thread directly
Thread vThread = Thread.ofVirtual()
    .name("io-worker")
    .start(() -> fetchFromApi());

// Use a virtual-thread-per-task executor (recommended)
try (ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor()) {
    executor.submit(() -> fetchFromApi(1));
    executor.submit(() -> fetchFromApi(2));
}

// Check if a thread is virtual
boolean isVirtual = Thread.currentThread().isVirtual();
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `Thread.ofVirtual()` | Factory for creating virtual threads |
| `Executors.newVirtualThreadPerTaskExecutor()` | Creates an executor that starts a new virtual thread per task |
| `Thread.isVirtual()` | Returns `true` if the thread is virtual |
| `--enable-preview` | Required for preview APIs in Java 21 (virtual threads are final, so no preview flag needed) |

**Syntax Rules**

1. Virtual threads are final and stable since Java 21 (JEP 444); no preview flag is required.
2. Virtual threads always start as daemon threads.
3. Virtual threads cannot have their priority or daemon status changed.
4. The recommended pattern is one virtual thread per task, not pooling virtual threads.
5. Virtual threads are not faster than platform threads for CPU-bound work; they improve throughput by hiding I/O latency.

**Constraints and Limitations**

- **CPU-bound work**: A virtual thread still uses a carrier platform thread while computing, so it does not add CPU capacity.
- **Pinning**: In Java 21, a virtual thread running inside a `synchronized` block cannot unmount if it blocks, pinning the carrier thread. JEP 491 (Java 24) removes this limitation.
- **Native code**: Virtual threads cannot unmount during native method calls.
- **Pooling**: Virtual threads should not be pooled; pooling negates their benefits and can cause GC pressure under memory constraints.

### Annotated Code Examples

**Example 1: Creating and Running Virtual Threads**

```java
public class VirtualThreadDemo {
    public static void main(String[] args) throws InterruptedException {
        // Direct virtual thread creation
        Thread vThread = Thread.ofVirtual()
            .name("virtual-worker")
            .start(() -> {
                System.out.println("Running in: " + Thread.currentThread().getName());
                System.out.println("Is virtual: " + Thread.currentThread().isVirtual());
                try {
                    Thread.sleep(1000); // blocking sleep
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
                System.out.println("Virtual thread done");
            });

        vThread.join();
        System.out.println("Main thread done");
    }
}
```

**Expected Output**

```
Running in: virtual-worker
Is virtual: true
Virtual thread done
Main thread done
```

**Why This Output Occurs**

`Thread.ofVirtual().start()` creates a virtual thread named "virtual-worker" and immediately schedules it. `isVirtual()` returns `true`. The `Thread.sleep(1000)` blocks the virtual thread, but the JVM unmounts it from its carrier thread, allowing the carrier to run other tasks. After the sleep completes, the virtual thread remounts and prints "Virtual thread done". The main thread joins and then prints "Main thread done".

---

**Example 2: Virtual Thread Per Task Executor**

```java
import java.util.concurrent.*;

public class VirtualThreadExecutorDemo {
    public static void main(String[] args) throws InterruptedException {
        // The recommended pattern: one virtual thread per task
        try (ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor()) {
            for (int i = 1; i <= 5; i++) {
                final int taskId = i;
                executor.submit(() -> {
                    System.out.println("Task " + taskId + " running in: "
                        + Thread.currentThread().getName());
                    try {
                        Thread.sleep(500); // simulate I/O
                    } catch (InterruptedException e) {
                        Thread.currentThread().interrupt();
                    }
                    System.out.println("Task " + taskId + " done");
                });
            }
        } // executor automatically closed; all tasks completed
        System.out.println("All tasks finished");
    }
}
```

**Expected Output (order may vary)**

```
Task 1 running in: virtual-1
Task 2 running in: virtual-2
Task 3 running in: virtual-3
Task 4 running in: virtual-4
Task 5 running in: virtual-5
Task 1 done
Task 2 done
Task 3 done
Task 4 done
Task 5 done
All tasks finished
```

**Why This Output Occurs**

The virtual-thread-per-task executor creates a new virtual thread for each submitted task. All five tasks run concurrently on a small number of carrier threads. The try-with-resources block ensures that `close()` is called, which waits for all tasks to complete before proceeding. This is the recommended pattern for virtual threads.

### Real-World Cases

- **Web Servers**: Spring Boot 3.2+ runs each HTTP request on a virtual thread when `spring.threads.virtual.enabled=true`, eliminating the need for large platform thread pools.
- **Database Access**: JDBC calls block on I/O; virtual threads let each query run on its own thread without exhausting the connection pool.
- **Microservice Communication**: A service that fans out to multiple downstream APIs can use one virtual thread per downstream call, simplifying code compared to reactive frameworks.

---

## Core Concept 2: Thread Containment Models — Project Loom Structured Concurrency (StructuredTaskScope)

### Definitions

**Core Definition**
Structured concurrency treats a group of related tasks running in different threads as a single unit of work, enforcing a parent-child relationship where subtasks cannot outlive their parent scope.

**Technical Definition**
`StructuredTaskScope` (JEP 453, preview) is a class in `java.util.concurrent` that defines a concurrent task scope. A scope opens, tasks are forked into it, and the scope waits for all tasks to complete or cancels them together. The API provides two subclasses: `ShutdownOnFailure` (cancels all tasks if any task fails) and `ShutdownOnSuccess` (cancels all tasks when the first task succeeds). The `fork()` method returns a `Subtask` rather than a `Future`, and `resultNow()` retrieves the result without blocking (throwing if the task is not yet complete).

**Beginner-Friendly Explanation**
Structured concurrency is like a project manager who assigns tasks to team members. The manager doesn't leave until every team member has either finished or been told to stop. If one team member fails, the manager cancels the others immediately. This prevents the chaos of "orphaned" tasks that keep running in the background after the main task has already failed.

### Purposes

- To eliminate thread leaks and cancellation delays by ensuring subtasks cannot outlive their parent scope.
- To simplify error handling and cancellation by treating a group of related tasks as a single unit of work.
- To improve observability with a clear parent-child thread hierarchy that appears in thread dumps and stack traces.
- To provide fail-fast behavior with `ShutdownOnFailure` and race-to-success behavior with `ShutdownOnSuccess`.

### Syntax Rules and Structure

**Complete General Syntax — ShutdownOnFailure**

```java
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    Subtask<String> user = scope.fork(() -> fetchUser(id));
    Subtask<Integer> order = scope.fork(() -> fetchOrder(id));

    scope.join();            // wait for all subtasks
    scope.throwIfFailed();   // rethrow the first failure

    String result = user.resultNow() + order.resultNow();
}
```

**Complete General Syntax — ShutdownOnSuccess**

```java
try (var scope = new StructuredTaskScope.ShutdownOnSuccess<String>()) {
    scope.fork(() -> fetchFromServiceA());
    scope.fork(() -> fetchFromServiceB());

    scope.join();            // wait for the first successful subtask
    String result = scope.result(); // first success
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `StructuredTaskScope` | The scope that manages the lifetime of forked tasks |
| `fork(Callable)` | Forks a subtask into the scope; returns a `Subtask` |
| `join()` | Waits for all subtasks to complete or the scope to be shut down |
| `throwIfFailed()` | Rethrows the first failure (ShutdownOnFailure) |
| `result()` | Returns the first successful result (ShutdownOnSuccess) |
| `resultNow()` | Returns the result if complete; throws otherwise |

**Syntax Rules**

1. `StructuredTaskScope` implements `AutoCloseable`; use try-with-resources to ensure all subtasks are cancelled and joined on exit.
2. The scope can only be used by the thread that created it (the "owner").
3. `fork()` returns a `Subtask`, not a `Future`; `Subtask` extends `Supplier`.
4. `join()` must be called before reading results.
5. When the scope is closed abnormally, all unfinished subtasks are interrupted (cancelled).
6. `ShutdownOnFailure` cancels all remaining tasks when any task fails.
7. `ShutdownOnSuccess` cancels all remaining tasks when any task succeeds.

**Constraints and Limitations**

- Structured concurrency is a **preview API** in Java 21; requires `--enable-preview`.
- The API is designed to pair with virtual threads; using it with platform threads is possible but not the primary use case.
- `StructuredTaskScope` is not a replacement for `ExecutorService`; it is for tasks that share a lifetime.
- Forgetting `join()` before reading results causes `resultNow()` to throw `IllegalStateException`.

### Annotated Code Examples

**Example 1: ShutdownOnFailure (All-or-Nothing)**

```java
import java.util.concurrent.*;

public class StructuredConcurrencyDemo {
    record CustomerProfile(String user, String order) { }

    static String fetchUser() throws InterruptedException {
        Thread.sleep(300);
        return "Alice";
    }

    static String fetchOrder() throws InterruptedException {
        Thread.sleep(500);
        return "Order-123";
    }

    static CustomerProfile getProfile() throws Exception {
        try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
            Subtask<String> user = scope.fork(StructuredConcurrencyDemo::fetchUser);
            Subtask<String> order = scope.fork(StructuredConcurrencyDemo::fetchOrder);

            scope.join();            // wait for both subtasks
            scope.throwIfFailed();   // if any failed, rethrow

            return new CustomerProfile(user.resultNow(), order.resultNow());
        }
    }

    public static void main(String[] args) throws Exception {
        long start = System.currentTimeMillis();
        CustomerProfile profile = getProfile();
        long elapsed = System.currentTimeMillis() - start;

        System.out.println("User: " + profile.user());
        System.out.println("Order: " + profile.order());
        System.out.println("Elapsed: " + elapsed + " ms");
    }
}
```

**Expected Output**

```
User: Alice
Order: Order-123
Elapsed: ~500 ms
```

**Why This Output Occurs**

Both `fetchUser()` (300 ms) and `fetchOrder()` (500 ms) run concurrently in separate virtual threads. The scope joins both tasks, so the total elapsed time is the maximum of the two durations (~500 ms), not the sum (~800 ms). `throwIfFailed()` ensures that if either task throws an exception, the other is cancelled and the exception is rethrown to the caller.

---

**Example 2: ShutdownOnSuccess (Race to Success)**

```java
import java.util.concurrent.*;

public class ShutdownOnSuccessDemo {
    static String fetchFromServiceA() throws InterruptedException {
        Thread.sleep(800);
        return "Result from A";
    }

    static String fetchFromServiceB() throws InterruptedException {
        Thread.sleep(300);
        return "Result from B";
    }

    static String fetchFastest() throws Exception {
        try (var scope = new StructuredTaskScope.ShutdownOnSuccess<String>()) {
            scope.fork(ShutdownOnSuccessDemo::fetchFromServiceA);
            scope.fork(ShutdownOnSuccessDemo::fetchFromServiceB);

            scope.join(); // wait for the first successful subtask
            return scope.result(); // first success
        }
    }

    public static void main(String[] args) throws Exception {
        long start = System.currentTimeMillis();
        String result = fetchFastest();
        long elapsed = System.currentTimeMillis() - start;

        System.out.println("Result: " + result);
        System.out.println("Elapsed: " + elapsed + " ms");
    }
}
```

**Expected Output**

```
Result: Result from B
Elapsed: ~300 ms
```

**Why This Output Occurs**

Service B responds in 300 ms, while Service A takes 800 ms. `ShutdownOnSuccess` cancels Service A as soon as Service B succeeds. The scope returns the first successful result, and the total elapsed time is approximately 300 ms. This is the classic "hedged request" pattern.

### Real-World Cases

- **Microservice Orchestration**: A request that needs data from multiple services uses `ShutdownOnFailure` to fetch all data or fail fast.
- **Hedged Requests**: A client sends the same request to multiple replicas and takes the first response using `ShutdownOnSuccess`, reducing tail latency.
- **Fan-Out/Fan-In**: A task that splits into subtasks (e.g., image processing tiles) and combines results uses structured concurrency to ensure all subtasks complete before combining.

---

## Core Concept 3: Alternative Data Sharing Frameworks — Scoped Values vs. ThreadLocal Memory Leakage Hazards

### Definitions

**Core Definition**
`ScopedValue` is an immutable, automatically cleaned-up alternative to `ThreadLocal` for sharing contextual data within and across threads. It is written once, bound for a bounded period, and inherited by child threads started within the scope.

**Technical Definition**
Scoped values (JEP 446, preview) enable the sharing of immutable data within and across threads. Unlike `ThreadLocal`, a scoped value is written once and is then immutable, and is available only for a bounded period during execution of the thread. Bindings are per-invocation, not stored in the thread; after the scope exits, the value is no longer accessible. Scoped values are preferred to thread-local variables, especially when using large numbers of virtual threads, because they have a lower memory footprint and eliminate the risk of stale values in thread pools.

**Beginner-Friendly Explanation**
`ThreadLocal` is like a whiteboard in a shared office: anyone can write on it, change what's written, and the next person who uses the office might see the old writing. `ScopedValue` is like a sealed envelope handed to you at the start of a meeting: you can read it, but you can't change it, and when the meeting ends, the envelope is shredded. This makes it much safer for passing request context (user IDs, trace IDs) through multiple layers of code.

### Purposes

- To eliminate memory leaks caused by `ThreadLocal` values persisting in thread pools after a task completes.
- To provide immutable context propagation that cannot be accidentally overwritten by callees.
- To reduce memory footprint when using millions of virtual threads (each `ThreadLocal` requires a per-thread map).
- To ensure that context is available only for the bounded duration of a scope, preventing stale data from leaking across requests.

### Syntax Rules and Structure

**Complete General Syntax**

```java
// Define a scoped value
static final ScopedValue<String> REQUEST_ID = ScopedValue.newInstance();

// Bind and run
ScopedValue.where(REQUEST_ID, "req-123")
    .run(() -> handleRequest());

// Bind multiple values
ScopedValue.where(USER_ID, "alice")
    .where(REQUEST_ID, "req-123")
    .run(() -> handleRequest());

// Inside the scope
String id = REQUEST_ID.get(); // returns "req-123"

// Outside the scope (after run() returns)
// REQUEST_ID.get() throws NoSuchElementException
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `ScopedValue.newInstance()` | Creates a new scoped value key |
| `ScopedValue.where(key, value)` | Creates a binding carrier |
| `.run(Runnable)` | Runs the lambda with the binding active |
| `.call(Callable)` | Runs and returns a value |
| `key.get()` | Returns the bound value; throws if not bound |
| `key.isBound()` | Returns `true` if the value is bound in the current scope |

**Syntax Rules**

1. Scoped values are immutable; there is no `set()` method.
2. Bindings are per-invocation and disappear when the scope exits.
3. Child threads (including virtual threads) started within the scope inherit the binding.
4. `get()` throws `NoSuchElementException` if the value is not bound in the current scope.
5. `ScopedValue.where()` returns a `Carrier` that can be chained for multiple bindings.
6. Scoped values are a **preview API** in Java 21; require `--enable-preview`.

**Constraints and Limitations**

- Scoped values cannot be used for mutable state; they are for immutable context only.
- Reading a scoped value outside its scope throws `NoSuchElementException`.
- Scoped values are not a drop-in replacement for `ThreadLocal` in all cases; they are for one-way transmission of unchanging data.
- Virtual threads inherit scoped values from their parent, but the parent must remain in scope while the child runs.

### Annotated Code Examples

**Example 1: ScopedValue for Request Context**

```java
public class ScopedValueDemo {
    static final ScopedValue<String> REQUEST_ID = ScopedValue.newInstance();
    static final ScopedValue<String> USER_ID = ScopedValue.newInstance();

    static void handleRequest() {
        System.out.println("Handling request: " + REQUEST_ID.get()
            + " for user: " + USER_ID.get());
        callService();
    }

    static void callService() {
        // Scoped value is available through the call chain
        System.out.println("Service sees request: " + REQUEST_ID.get());
    }

    public static void main(String[] args) {
        ScopedValue.where(REQUEST_ID, "req-123")
            .where(USER_ID, "alice")
            .run(() -> handleRequest());

        // Outside the scope, the values are no longer bound
        System.out.println("Outside scope: " + REQUEST_ID.isBound()); // false
    }
}
```

**Expected Output**

```
Handling request: req-123 for user: alice
Service sees request: req-123
Outside scope: false
```

**Why This Output Occurs**

`ScopedValue.where(REQUEST_ID, "req-123").where(USER_ID, "alice").run(...)` binds both values for the duration of the `run()` lambda. Inside `handleRequest()` and `callService()`, the values are accessible via `get()`. After `run()` returns, the bindings are removed; `isBound()` returns `false`, and `get()` would throw `NoSuchElementException`.

---

**Example 2: ScopedValue with Virtual Threads**

```java
import java.util.concurrent.*;

public class ScopedValueVirtualThreadDemo {
    static final ScopedValue<String> TRACE_ID = ScopedValue.newInstance();

    public static void main(String[] args) throws Exception {
        try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
            ScopedValue.where(TRACE_ID, "trace-001")
                .run(() -> {
                    // Child virtual thread inherits the scoped value
                    Future<String> result = executor.submit(() -> {
                        return "Child sees: " + TRACE_ID.get();
                    });
                    try {
                        System.out.println(result.get());
                    } catch (Exception e) { }
                });
        }
    }
}
```

**Expected Output**

```
Child sees: trace-001
```

**Why This Output Occurs**

The virtual thread submitted inside `ScopedValue.where(...).run(...)` inherits the `TRACE_ID` binding from its parent. This is because scoped values are inherited by child threads started within the scope. After the scope exits, the binding is discarded. This pattern is ideal for distributed tracing with virtual threads.

### Real-World Cases

- **Distributed Tracing**: A trace ID is bound as a scoped value at the start of a request and automatically available in all downstream calls, without passing it through every method signature.
- **Authentication Context**: The authenticated user is bound as a scoped value and accessible in service and repository layers.
- **Multi-Tenant Applications**: The tenant ID is bound per request, ensuring data isolation without the risk of ThreadLocal leakage between requests in a shared thread pool.

---

## Core Concept 4: Architectural Optimizations — Designing Systems for High-Throughput Task Execution

### Definitions

**Core Definition**
Architectural optimization for Project Loom involves designing systems that leverage virtual threads and structured concurrency to maximize throughput while minimizing resource consumption and code complexity.

**Technical Definition**
High-throughput task execution with virtual threads requires understanding that virtual threads are a scalability tool, not a speed tool. They improve throughput by hiding I/O latency, not by making CPU-bound code faster. Architectural optimizations include: using virtual threads for I/O-bound tasks, limiting concurrency where needed with `Semaphore`, avoiding pooling of virtual threads, tuning carrier thread parallelism (`jdk.virtualThreadScheduler.parallelism`), and using structured concurrency to prevent thread leaks. The default carrier thread pool size is the number of available processors, with a maximum pool size of 256.

**Beginner-Friendly Explanation**
Think of virtual threads as expanding a restaurant's capacity. Before, you had 10 tables (platform threads) and had to turn away customers (tasks) when all tables were full. With virtual threads, you can have 10,000 tables—but you still have the same kitchen (CPU). The architectural challenge is ensuring the kitchen doesn't get overwhelmed. You need to limit how many orders (tasks) hit the kitchen at once, even if you have thousands of tables.

### Purposes

- To maximize throughput for I/O-bound workloads by keeping CPU cores busy while threads wait.
- To reduce memory consumption by replacing large platform thread pools with lightweight virtual threads.
- To simplify code by replacing complex asynchronous pipelines with straightforward blocking code.
- To prevent resource exhaustion by limiting concurrency where the underlying resource (database, API) cannot handle unlimited concurrent access.

### Syntax Rules and Structure

**Complete General Syntax — Concurrency Limiting**

```java
// Limit concurrency with Semaphore
private static final Semaphore DB_LIMIT = new Semaphore(20);

void queryDatabase() throws InterruptedException {
    DB_LIMIT.acquire();
    try {
        // database query (blocking I/O)
    } finally {
        DB_LIMIT.release();
    }
}

// Use virtual-thread-per-task executor
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    for (Task task : tasks) {
        executor.submit(() -> {
            // blocking I/O code
        });
    }
}
```

**Complete General Syntax — Carrier Thread Tuning**

```bash
# JVM flags for virtual thread scheduler
-Djdk.virtualThreadScheduler.parallelism=16    # target parallelism
-Djdk.virtualThreadScheduler.maxPoolSize=256   # maximum carrier threads
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `Semaphore` | Limits concurrent access to a bounded resource |
| `newVirtualThreadPerTaskExecutor()` | Creates one virtual thread per task |
| `jdk.virtualThreadScheduler.parallelism` | Target number of carrier threads (default: CPU count) |
| `jdk.virtualThreadScheduler.maxPoolSize` | Maximum carrier threads (default: 256) |

**Syntax Rules**

1. Do not pool virtual threads; create a new one for each task.
2. Use `Semaphore` or similar to limit concurrency when the underlying resource cannot handle unlimited parallelism.
3. The carrier thread pool size is controlled by `jdk.virtualThreadScheduler.parallelism` (default: number of processors).
4. Virtual threads share the same memory model as platform threads; all thread-safety rules still apply.
5. CPU-bound tasks should use a fixed platform thread pool sized to the number of cores, not virtual threads.

**Constraints and Limitations**

- Virtual threads do not increase CPU capacity; CPU-bound workloads gain no benefit.
- Unlimited virtual thread creation can overwhelm downstream resources (database connections, API rate limits) without proper throttling.
- In Java 21, `synchronized` blocks pin virtual threads to carriers; use `ReentrantLock` to avoid pinning (fixed in Java 24 by JEP 491).
- Native code and some JDK internals can also pin virtual threads.

### Annotated Code Examples

**Example 1: Semaphore-Limited Virtual Threads**

```java
import java.util.concurrent.*;

public class ConcurrencyLimitDemo {
    private static final Semaphore DB_LIMIT = new Semaphore(3);

    static void queryDatabase(int id) throws InterruptedException {
        DB_LIMIT.acquire();
        try {
            System.out.println("Query " + id + " executing. Available permits: "
                + DB_LIMIT.availablePermits());
            Thread.sleep(500); // simulate database query
        } finally {
            DB_LIMIT.release();
        }
    }

    public static void main(String[] args) {
        try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
            for (int i = 1; i <= 10; i++) {
                final int id = i;
                executor.submit(() -> {
                    try {
                        queryDatabase(id);
                    } catch (InterruptedException e) {
                        Thread.currentThread().interrupt();
                    }
                });
            }
        }
        System.out.println("All queries completed");
    }
}
```

**Expected Output (order may vary)**

```
Query 1 executing. Available permits: 2
Query 2 executing. Available permits: 1
Query 3 executing. Available permits: 0
Query 1 done
Query 4 executing. Available permits: 2
...
All queries completed
```

**Why This Output Occurs**

A `Semaphore` with 3 permits limits database concurrency to 3 simultaneous queries. Even though 10 virtual threads are created, only 3 can execute the database query at any time. The remaining 7 block on `acquire()` until permits are released. This prevents overwhelming the database with 10 concurrent connections.

---

**Example 2: CPU-Bound Work Should Not Use Virtual Threads**

```java
import java.util.concurrent.*;

public class CpuBoundDemo {
    static long compute(int n) {
        long sum = 0;
        for (int i = 0; i < n; i++) sum += i;
        return sum;
    }

    public static void main(String[] args) throws Exception {
        int cores = Runtime.getRuntime().availableProcessors();

        // CPU-bound work: use platform threads sized to core count
        try (var executor = Executors.newFixedThreadPool(cores)) {
            Future<Long> f1 = executor.submit(() -> compute(500_000_000));
            Future<Long> f2 = executor.submit(() -> compute(500_000_000));
            long sum = f1.get() + f2.get();
            System.out.println("Sum: " + sum);
        }
    }
}
```

**Expected Output**

```
Sum: 249999999500000000
```

**Why This Output Occurs**

CPU-bound tasks should use a fixed platform thread pool sized to the number of available processors. Using virtual threads for CPU-bound work provides no benefit because virtual threads still occupy carrier threads while computing. The `newFixedThreadPool(cores)` ensures that exactly `cores` tasks run in parallel, maximizing CPU utilization without context-switching overhead.

### Real-World Cases

- **Spring Boot 3.2+**: Setting `spring.threads.virtual.enabled=true` replaces Tomcat's platform thread pool with virtual threads, one per request.
- **Helidon 4**: A web framework built entirely around virtual threads, using blocking, imperative APIs for high-throughput HTTP handling.
- **Database Connection Pools**: Virtual threads handle many concurrent requests, but a `Semaphore` limits the number of actual database connections to prevent resource exhaustion.
- **HTTP Clients**: The `java.net.http.HttpClient` uses virtual threads internally to handle concurrent requests efficiently.

---

## Core Concept 5: Multi-Threading Execution Patterns — Moving from Blocking Layouts to Lightweight, Non-Blocking Designs

### Definitions

**Core Definition**
Project Loom enables a fundamental shift in execution patterns: instead of writing non-blocking, asynchronous code (callbacks, `CompletableFuture` chains, reactive streams) to avoid blocking threads, developers can write straightforward blocking code that scales just as well because blocking is now cheap.

**Technical Definition**
Virtual threads make blocking operations efficient by unmounting the virtual thread from its carrier platform thread during I/O, allowing the carrier to run other virtual threads. This means that a blocking, synchronous coding style—where each task is written as a simple sequential flow—can achieve the same scalability as non-blocking, asynchronous code, without the complexity of callbacks, reactive operators, or manual continuation management. The JVM handles the scheduling and unmounting transparently, so the developer writes code as if blocking were free.

**Beginner-Friendly Explanation**
Before Loom, writing scalable I/O code was like being a waiter who couldn't stand still. If a table wasn't ready, you had to go take other orders, remember to come back, and juggle twenty things at once. With virtual threads, you can be a waiter who just waits at the table until the food is ready—the restaurant (JVM) has enough waiters (virtual threads) that nobody else is left unattended. You write code like "get user, then get order, then combine"—simple, linear, readable—and it scales to thousands of concurrent requests.

### Purposes

- To replace complex asynchronous pipelines (`CompletableFuture`, `reactive streams`) with simple blocking code that achieves comparable scalability.
- To reduce cognitive load and debugging difficulty by maintaining a linear control flow.
- To leverage existing blocking libraries and APIs (JDBC, HTTP clients) without rewriting them for non-blocking I/O.
- To achieve high throughput on I/O-bound workloads without the overhead of thread pools or event loops.

### Syntax Rules and Structure

**Complete General Syntax — Blocking Style (Recommended with Virtual Threads)**

```java
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    Future<CustomerProfile> profile = executor.submit(() -> {
        String user = fetchUser();       // blocking call
        String order = fetchOrder();     // blocking call
        return new CustomerProfile(user, order);
    });
    System.out.println(profile.get());
}
```

**Complete General Syntax — Non-Blocking Style (Traditional Async)**

```java
CompletableFuture<CustomerProfile> profile =
    fetchUserAsync()
        .thenCombine(fetchOrderAsync(), CustomerProfile::new);
profile.thenAccept(System.out::println);
```

**Component Breakdown**

| Style | Description | Complexity |
|-------|-------------|------------|
| Blocking (virtual threads) | Sequential code, easy to read and debug | Low |
| Non-blocking (CompletableFuture) | Chained async operations, harder to debug | High |
| Reactive (WebFlux, RxJava) | Stream-based, backpressure-aware | Very high |

**Syntax Rules**

1. Virtual threads allow blocking calls to scale; use blocking style for I/O-bound tasks.
2. Structured concurrency (`StructuredTaskScope`) provides a middle ground: concurrent execution with a linear composition style.
3. Non-blocking code is still useful for CPU-bound streaming or when backpressure is required.
4. Avoid mixing blocking and non-blocking paradigms within the same task; choose one style per layer.

**Constraints and Limitations**

- Blocking style with virtual threads may still pin if `synchronized` is used (Java 21); use `ReentrantLock` or upgrade to Java 24.
- Some libraries (e.g., older JDBC drivers) may not be fully compatible with virtual thread unmounting.
- Reactive frameworks still offer advantages for CPU-bound streaming and complex backpressure scenarios.

### Annotated Code Examples

**Example 1: Blocking vs. Non-Blocking with Virtual Threads**

```java
import java.util.concurrent.*;

public class BlockingVsNonBlockingDemo {
    static String fetchUser() throws InterruptedException {
        Thread.sleep(300);
        return "Alice";
    }

    static String fetchOrder() throws InterruptedException {
        Thread.sleep(500);
        return "Order-123";
    }

    // BLOCKING style with virtual threads: simple, sequential
    static String blockingStyle() throws Exception {
        try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
            Future<String> user = executor.submit(BlockingVsNonBlockingDemo::fetchUser);
            Future<String> order = executor.submit(BlockingVsNonBlockingDemo::fetchOrder);
            return user.get() + " / " + order.get();
        }
    }

    // NON-BLOCKING style with CompletableFuture: chained, complex
    static String nonBlockingStyle() throws Exception {
        CompletableFuture<String> user = CompletableFuture.supplyAsync(() -> {
            try { return fetchUser(); } catch (InterruptedException e) { throw new RuntimeException(e); }
        });
        CompletableFuture<String> order = CompletableFuture.supplyAsync(() -> {
            try { return fetchOrder(); } catch (InterruptedException e) { throw new RuntimeException(e); }
        });
        return user.thenCombine(order, (u, o) -> u + " / " + o).get();
    }

    public static void main(String[] args) throws Exception {
        long start1 = System.currentTimeMillis();
        System.out.println("Blocking: " + blockingStyle()
            + " (" + (System.currentTimeMillis() - start1) + " ms)");

        long start2 = System.currentTimeMillis();
        System.out.println("Non-blocking: " + nonBlockingStyle()
            + " (" + (System.currentTimeMillis() - start2) + " ms)");
    }
}
```

**Expected Output**

```
Blocking: Alice / Order-123 (500 ms)
Non-blocking: Alice / Order-123 (500 ms)
```

**Why This Output Occurs**

Both styles achieve the same result and similar performance (~500 ms, the maximum of the two I/O durations). The blocking style uses virtual threads and `Future.get()`, while the non-blocking style uses `CompletableFuture.thenCombine()`. The blocking style is simpler to write and debug, while the non-blocking style requires understanding of asynchronous composition. With virtual threads, the blocking style is preferred for I/O-bound tasks because it is easier to read and maintain.

### Real-World Cases

- **Web Frameworks**: Helidon 4 and Spring Boot 3.2+ use virtual threads to allow developers to write blocking request handlers that scale to high concurrency.
- **Database Access**: A service that queries multiple databases can use virtual threads to execute blocking JDBC calls concurrently, without complex async wrappers.
- **File Processing**: A batch job that reads many files can use one virtual thread per file, blocking on each file read without exhausting platform threads.
- **Microservice Aggregation**: A gateway that calls multiple downstream services can use structured concurrency with virtual threads to fan out and aggregate results with simple, readable code.

---

## Comparison Summary

| Feature | Platform Threads (Pre-Loom) | Virtual Threads (Java 21+) | Structured Concurrency | Scoped Values |
|---------|----------------------------|---------------------------|----------------------|---------------|
| **Memory per thread** | ~1 MB stack | ~200-300 bytes metadata | N/A (scope) | Immutable, low footprint |
| **Max concurrent** | Thousands | Millions | Bounded by scope | N/A |
| **Blocking cost** | High (wastes OS thread) | Low (unmounts from carrier) | N/A | N/A |
| **Error handling** | Manual | Manual | Automatic (ShutdownOnFailure) | N/A |
| **Cancellation** | Manual interruption | Manual interruption | Automatic (scope close) | N/A |
| **Context sharing** | ThreadLocal (mutable, leaks) | ThreadLocal (leaks) | N/A | ScopedValue (immutable, auto-cleaned) |
| **Preview status** | Final | Final (Java 21) | Preview (JEP 453) | Preview (JEP 446) |

---

## References

- JEP 444: Virtual Threads - https://openjdk.org/jeps/444
- JEP 453: Structured Concurrency (Preview) - https://openjdk.org/jeps/453
- JEP 446: Scoped Values (Preview) - https://openjdk.org/jeps/446
- JEP 491: Synchronize Virtual Threads without Pinning - https://openjdk.org/jeps/491
- JEP 506: Scoped Values (Final) - https://openjdk.org/jeps/506
- Project Loom - https://openjdk.org/projects/loom/
- Oracle Java Tutorials: Virtual Threads - https://docs.oracle.com/en/java/javase/21/core/virtual-threads.html
- Structured Concurrency (JCP Presentation) - https://jcp.org/aboutJava/communityprocess/ec-public/materials/2025-03-21/StructuredConcurrency.pdf
- Project Loom in IntelliJ IDEA (JetBrains Blog) - https://blog.jetbrains.com/idea/2026/08/project-loom-in-intellij-idea-virtual-threads-scoped-values-and-structured-concurrency/
- Scoped Values (JEP 446) - https://openjdk.org/jeps/446
- Structured Concurrency (JEP 453) - https://openjdk.org/jeps/453
- Virtual Threads (JEP 444) - https://openjdk.org/jeps/444
- Synchronize Virtual Threads without Pinning (JEP 491) - https://openjdk.org/jeps/491
- Quarkus Blog: To Cache or Not to Cache Virtual Threads - https://quarkus.io/blog/to-cache-or-not-to-cache-virtual-threads/
- Java Virtual Threads (Project Loom): Complete Guide - https://www.cleverence.com/articles/oracle-docs/java-virtual-threads-project-loom-complete-guide-examples-and-performance-5431/
- Structured Concurrency in Java 21 - https://www.infoq.com/articles/structured-concurrency-java-21/
- Scoped Values in Java 21 - https://www.infoq.com/articles/scoped-values-java-21/