# Java Virtual Threads: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Virtual threads are lightweight threads introduced in Java 21 (JEP 444) that are managed by the JVM rather than the operating system, allowing millions of concurrent tasks with minimal overhead. They are instances of `java.lang.Thread` that are not tied to a specific OS thread.

**Technical Definition:** "Virtual threads are lightweight threads that dramatically reduce the effort of writing, maintaining, and observing high-throughput concurrent applications". A virtual thread is an instance of `java.lang.Thread` that is not tied to a particular OS thread. "A virtual thread still runs code on an OS thread. However, when code running in a virtual thread calls a blocking I/O operation, the Java runtime suspends the virtual thread until it can be resumed. The OS thread associated with the suspended virtual thread is now free to perform operations for other virtual threads". "To simulate a lot of threads, the Java runtime maps a large number of virtual threads to a small number of OS threads".

**Beginner-Friendly Explanation:** Imagine a busy restaurant. Platform threads are like full-time employees — each one takes up a desk and a salary, and you can only afford so many. Virtual threads are like freelancers who share desks. When one freelancer is waiting on a phone call (blocked), they step aside, and another freelancer uses the desk. You can have thousands of freelancers without needing thousands of desks. Virtual threads make blocking cheap, so you can write simple, sequential code and still handle massive concurrency.

### Key Characteristics

- **JVM-Managed:** Virtual threads are scheduled by the JVM, not the operating system.
- **Lightweight:** They require very little memory and can be created in huge numbers (millions).
- **Cheap Blocking:** When a virtual thread blocks on I/O, it unmounts from its carrier thread, freeing the OS thread for other work.
- **Not Faster, More Scalable:** "Virtual threads are not faster threads; they do not run code any faster than platform threads. They exist to provide scale (higher throughput), not speed (lower latency)".
- **I/O-Bound Focus:** "Virtual threads are suitable for running tasks that spend most of the time blocked, often waiting for I/O operations to complete. However, they aren't intended for long-running CPU-intensive operations".
- **Thread-Local Support:** Virtual threads support `ThreadLocal` variables, but their use should be carefully considered due to the large number of threads.

### Prerequisites

- Basic understanding of Java threads and the `Thread` class
- Familiarity with `Runnable`, `Callable`, and `ExecutorService`
- Knowledge of blocking I/O operations
- Understanding of the `java.util.concurrent` package

### Related Programming Areas

- **Concurrency:** The broader topic of multithreaded programming.
- **Executors:** Virtual threads integrate with the Executor framework.
- **Structured Concurrency:** A programming paradigm for managing groups of related tasks.
- **Scoped Values:** An alternative to `ThreadLocal` for sharing data within a task.
- **Reactive Programming:** Virtual threads offer a simpler alternative to reactive callbacks for I/O-bound workloads.

### Core Concepts / Features

---

## 1. Virtual Threads

### Definitions

**Core Definition:** A virtual thread is a lightweight thread managed by the JVM that is not tied to a specific OS thread, enabling high-throughput concurrency with simple blocking code.

**Technical Definition:** "A virtual thread is an instance of `java.lang.Thread` that is not tied to a particular OS thread. A platform thread is implemented as a thin wrapper around an operating system (OS) thread". "Unlike platform threads, virtual threads typically have a shallow call stack, performing as few as a single HTTP client call or a single JDBC query". Virtual threads were proposed as a preview feature by JEP 425 and delivered in JDK 19, previewed again in JDK 20, and finalized in JDK 21.

**Beginner-Friendly Explanation:** A virtual thread is like a task on a to-do list that the JVM manages. When the task needs to wait (e.g., for a database response), the JVM puts it aside and picks up another task. When the response arrives, the task is resumed. This allows a single OS thread to handle many virtual threads, much like a single chef can manage many dishes by switching between them while things simmer.

### Purposes

- To enable server applications to scale with near-optimal hardware utilization using the simple thread-per-request style
- To make blocking I/O cheap, eliminating the need for complex asynchronous callback-based code
- To reduce the effort of writing, maintaining, and debugging high-throughput concurrent applications
- To allow millions of concurrent tasks with minimal memory overhead
- To integrate seamlessly with existing `java.lang.Thread` API code

### Syntax Rules and Structure

**Creating Virtual Threads:**

```java
// Approach 1: Thread.ofVirtual().start()
Thread vThread = Thread.ofVirtual().start(() -> {
    System.out.println("Running in virtual thread");
});
vThread.join();

// Approach 2: Thread.startVirtualThread()
Thread vThread = Thread.startVirtualThread(() -> {
    System.out.println("Running in virtual thread");
});
vThread.join();

// Approach 3: Executors.newVirtualThreadPerTaskExecutor()
try (ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor()) {
    executor.submit(() -> System.out.println("Task 1"));
    executor.submit(() -> System.out.println("Task 2"));
}
```

**Key Methods:**

| Method | Description |
|--------|-------------|
| `Thread.ofVirtual()` | Returns a builder for creating virtual threads |
| `Thread.startVirtualThread(Runnable)` | Creates and starts a virtual thread |
| `Executors.newVirtualThreadPerTaskExecutor()` | Creates an executor that runs each task in a new virtual thread |

**Syntax Rules:**

- Virtual threads are always daemon threads and cannot be changed to non-daemon.
- Virtual threads have no default name; `getName()` returns an empty string unless a name is set.
- "Virtual threads are not faster threads; they do not run code any faster than platform threads".

**Constraints and Limitations:**

- **Pinning:** In Java 21–23, a virtual thread can become "pinned" to its carrier thread if it blocks while inside a `synchronized` block or calls a native method, preventing unmounting.
- **CPU-Bound Work:** Virtual threads offer no benefit for CPU-intensive tasks; use parallel streams or Fork/Join for those.
- **ThreadLocal:** Using `ThreadLocal` with virtual threads can cause memory issues because each virtual thread gets its own copy.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Creating a Virtual Thread**

```java
// File: VirtualThreadBasic.java
public class VirtualThreadBasic {
    public static void main(String[] args) throws InterruptedException {
        // Step 1: Create and start a virtual thread
        Thread vThread = Thread.ofVirtual()
            .name("my-virtual-thread")
            .start(() -> {
                System.out.println("Running in: " + Thread.currentThread());
                System.out.println("Is virtual: " + Thread.currentThread().isVirtual());
            });

        // Step 2: Wait for the virtual thread to complete
        vThread.join();

        System.out.println("Main thread: " + Thread.currentThread());
    }
}
```

**Expected Output:**
```
Running in: VirtualThread[#21,my-virtual-thread]/runnable@ForkJoinPool-1-worker-1
Is virtual: true
Main thread: Thread[#1,main,5,main]
```

**Step-by-Step Explanation:**

1. `Thread.ofVirtual()` returns a builder for virtual threads.
2. `.name("my-virtual-thread")` gives the thread a descriptive name.
3. `.start(Runnable)` creates and starts the virtual thread.
4. `Thread.currentThread().isVirtual()` returns `true` for virtual threads.
5. `vThread.join()` waits for the virtual thread to finish.
6. The main thread is a platform thread.

**Example 2: Using `Executors.newVirtualThreadPerTaskExecutor()`**

```java
// File: VirtualThreadExecutor.java
import java.util.concurrent.*;
import java.util.stream.IntStream;

public class VirtualThreadExecutor {
    public static void main(String[] args) throws Exception {
        // Create an executor that starts a new virtual thread per task
        try (ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor()) {

            // Submit 10,000 tasks
            IntStream.range(0, 10_000).forEach(i -> {
                executor.submit(() -> {
                    // Simulate blocking I/O
                    Thread.sleep(1000);
                    return i;
                });
            });
        } // executor.close() is called automatically, waiting for tasks to finish
        System.out.println("All tasks completed");
    }
}
```

**Expected Output:**
```
All tasks completed
```

**Step-by-Step Explanation:**

1. `Executors.newVirtualThreadPerTaskExecutor()` creates an `ExecutorService` that creates a new virtual thread for each task.
2. The try-with-resources block ensures the executor is closed, which waits for all tasks to complete.
3. 10,000 tasks are submitted, each sleeping for 1 second.
4. Because virtual threads are lightweight, all 10,000 tasks can run concurrently without exhausting resources.

### Real-World Cases

- **Web Servers:** Handling thousands of concurrent HTTP requests with simple thread-per-request code.
- **Microservices:** Making blocking calls to databases and other services without complex async code.
- **Data Processing:** Processing large numbers of I/O-bound tasks concurrently.

---

## 2. Platform Threads vs Virtual Threads

### Definitions

**Core Definition:** Platform threads are thin wrappers around OS threads, while virtual threads are JVM-managed lightweight threads that are not tied to OS threads.

**Technical Definition:** "A platform thread is implemented as a thin wrapper around an operating system (OS) thread. A platform thread runs Java code on its underlying OS thread, and the platform thread captures its OS thread for the platform thread's entire lifetime. Consequently, the number of available platform threads is limited to the number of OS threads". "A virtual thread isn't tied to a specific OS thread. A virtual thread still runs code on an OS thread. However, when code running in a virtual thread calls a blocking I/O operation, the Java runtime suspends the virtual thread until it can be resumed. The OS thread associated with the suspended virtual thread is now free to perform operations for other virtual threads".

**Beginner-Friendly Explanation:** Platform threads are like dedicated phone lines — each call needs its own line, and you can only have as many lines as the phone company provides. Virtual threads are like a modern VoIP system — thousands of calls can share a limited number of physical lines, with the system switching between them as needed.

### Comparison Table

| Aspect | Platform Threads | Virtual Threads |
|--------|-----------------|-----------------|
| **Management** | OS-managed | JVM-managed |
| **Memory** | ~1 MB per thread | ~ few hundred bytes initially |
| **Creation Cost** | Thousands of CPU instructions | Very cheap |
| **Max Concurrent** | Thousands | Millions |
| **Blocking** | Blocks OS thread | Unmounts from carrier thread |
| **Use Case** | CPU-bound tasks | I/O-bound tasks |
| **Thread-Local** | Safe for caching | Careful use advised |
| **Pinning** | N/A | Possible with synchronized/native |

**Syntax Rules:**

- `Thread.ofPlatform()` creates platform threads; `Thread.ofVirtual()` creates virtual threads.
- `Thread.currentThread().isVirtual()` distinguishes between the two.
- Both extend `java.lang.Thread` and support the same API.

### Annotated Code Example

```java
// File: ThreadComparison.java
public class ThreadComparison {
    public static void main(String[] args) throws Exception {
        // Platform thread
        Thread platform = Thread.ofPlatform()
            .name("platform-thread")
            .start(() -> System.out.println("Platform: " + Thread.currentThread().isVirtual()));

        // Virtual thread
        Thread virtual = Thread.ofVirtual()
            .name("virtual-thread")
            .start(() -> System.out.println("Virtual: " + Thread.currentThread().isVirtual()));

        platform.join();
        virtual.join();
    }
}
```

**Expected Output:**
```
Platform: false
Virtual: true
```

### Real-World Cases

- **Platform Threads:** CPU-intensive computations, tasks requiring native code, and scenarios with a limited number of long-lived threads.
- **Virtual Threads:** Web servers, microservices, database access, and any I/O-bound workload with high concurrency.

---

## 3. Creating Virtual Threads

### 3.1 `Thread.ofVirtual()`

#### Definitions

**Core Definition:** `Thread.ofVirtual()` returns a `Thread.Builder.OfVirtual` instance for creating and configuring virtual threads.

**Technical Definition:** "Call the `Thread.ofVirtual()` method to create an instance of `Thread.Builder` for creating virtual threads". The builder supports methods for naming threads, setting uncaught exception handlers, and starting threads.

#### Purposes

- To create virtual threads with a fluent builder API
- To configure thread names, inheritance of thread-locals, and exception handlers
- To provide a consistent API for creating both platform and virtual threads

#### Syntax Rules and Structure

```java
Thread vThread = Thread.ofVirtual()
    .name("my-thread")
    .uncaughtExceptionHandler((t, e) -> System.err.println("Error: " + e))
    .start(() -> { /* task */ });
```

**Key Methods:**

| Method | Description |
|--------|-------------|
| `name(String)` | Sets the thread name |
| `name(String prefix, long start)` | Sets the thread name with a counter |
| `inheritInheritableThreadLocals(boolean)` | Controls inheritance of thread-locals |
| `uncaughtExceptionHandler(Thread.UncaughtExceptionHandler)` | Sets the exception handler |
| `start(Runnable)` | Creates and starts the virtual thread |

#### Annotated Code Example

```java
// File: OfVirtualDemo.java
public class OfVirtualDemo {
    public static void main(String[] args) throws InterruptedException {
        Thread.Builder.OfVirtual builder = Thread.ofVirtual().name("worker-", 1);

        Thread t1 = builder.start(() -> 
            System.out.println("Thread: " + Thread.currentThread().getName()));
        Thread t2 = builder.start(() -> 
            System.out.println("Thread: " + Thread.currentThread().getName()));

        t1.join();
        t2.join();
    }
}
```

**Expected Output:**
```
Thread: worker-1
Thread: worker-2
```

### 3.2 `Thread.startVirtualThread()`

#### Definitions

**Core Definition:** `Thread.startVirtualThread(Runnable)` is a convenience method that creates and starts a virtual thread in a single call.

**Technical Definition:** "Creates a virtual thread to execute a task and schedules it to execute". This is a static factory method equivalent to `Thread.ofVirtual().start(task)`.

#### Purposes

- To create and start a virtual thread in one line
- To provide a simple API for fire-and-forget tasks
- To serve as the easiest entry point for virtual threads

#### Syntax Rules and Structure

```java
Thread vThread = Thread.startVirtualThread(() -> {
    System.out.println("Running in virtual thread");
});
vThread.join();
```

#### Annotated Code Example

```java
// File: StartVirtualThreadDemo.java
public class StartVirtualThreadDemo {
    public static void main(String[] args) throws InterruptedException {
        Thread vThread = Thread.startVirtualThread(() -> {
            System.out.println("Hello from virtual thread: " + 
                Thread.currentThread().isVirtual());
        });
        vThread.join();
    }
}
```

**Expected Output:**
```
Hello from virtual thread: true
```

### 3.3 `Executors.newVirtualThreadPerTaskExecutor()`

#### Definitions

**Core Definition:** `Executors.newVirtualThreadPerTaskExecutor()` creates an `ExecutorService` that starts a new virtual thread for each submitted task.

**Technical Definition:** "The `java.util.concurrent.Executors` class also defines methods to create an `ExecutorService` that starts a new virtual thread for each task". "The factory method `Executors.newVirtualThreadPerTaskExecutor()` yields an `ExecutorService` that runs each task in a separate virtual thread".

#### Purposes

- To integrate virtual threads with the familiar Executor framework
- To submit tasks without managing individual threads
- To provide a try-with-resources-friendly executor for structured concurrency
- To serve as the recommended way to use virtual threads in server applications

#### Syntax Rules and Structure

```java
try (ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor()) {
    executor.submit(task1);
    executor.submit(task2);
    // executor.close() waits for all tasks
}
```

**Syntax Rules:**

- The executor creates a new virtual thread for every task, no pooling.
- `ExecutorService` implements `AutoCloseable` since Java 19, so try-with-resources works.
- "The `ExecutorService` interface extends `AutoCloseable` so the `ExecutorService` instance returned from `Executors.newVirtualThreadPerTaskExecutor()` can be used in a try-with-resources".

**Constraints and Limitations:**

- "Using `Executors.newVirtualThreadPerTaskExecutor()` to create and execute tasks as virtual threads is largely a temporary solution" — consider structured concurrency for production code.
- Virtual threads created by this executor are not named by default, which can complicate debugging.

#### Annotated Code Example

```java
// File: VirtualThreadPerTaskDemo.java
import java.util.concurrent.*;
import java.util.stream.IntStream;

public class VirtualThreadPerTaskDemo {
    public static void main(String[] args) throws Exception {
        try (ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor()) {
            IntStream.range(0, 1000).forEach(i -> {
                executor.submit(() -> {
                    Thread.sleep(1000);  // Simulate I/O
                    return "Task " + i + " done";
                });
            });
        } // Waits for all 1000 tasks to complete
        System.out.println("All tasks completed");
    }
}
```

**Expected Output:**
```
All tasks completed
```

### Real-World Cases

- **Web Servers:** Using `newVirtualThreadPerTaskExecutor()` to handle each HTTP request in its own virtual thread.
- **Background Processing:** Submitting I/O-bound tasks to an executor without managing threads.
- **Testing:** Quickly running many concurrent tasks in tests.

---

## 4. Structured Concurrency and `StructuredTaskScope`

### Definitions

**Core Definition:** Structured concurrency is a programming paradigm that treats a group of related concurrent tasks as a single unit of work with a well-defined lifecycle. `StructuredTaskScope` is the API that implements this paradigm in Java.

**Technical Definition:** "StructuredTaskScope supports cases where a task splits into several concurrent subtasks, and where the subtasks must complete before the main task continues. A `StructuredTaskScope` can be used to ensure that the lifetime of a concurrent operation is confined by a syntax block, just like that of a sequential operation in structured programming". "The API is intended to be used with the try-with-resources statement. The intention is that code in the try block uses the `fork` method to fork threads to execute the subtasks, wait for the subtasks to finish with the `join` method, and then process the results".

**Beginner-Friendly Explanation:** Structured concurrency is like a project manager who creates a team for a specific project. All team members (subtasks) report to the manager and must finish before the project is considered complete. If one team member fails, the manager can cancel the others. The `StructuredTaskScope` is the manager — it keeps track of all subtasks, ensures they all complete, and handles failures gracefully.

### Purposes

- To ensure that concurrent subtasks are properly scoped and completed before the main task continues
- To prevent thread leaks by automatically cancelling remaining subtasks when one fails
- To provide a clear, structured way to manage groups of related concurrent tasks
- To improve observability and debugging of concurrent code
- To simplify error handling in concurrent scenarios

### Syntax Rules and Structure

**Basic Pattern:**

```java
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    Subtask<String> user = scope.fork(() -> fetchUser(id));
    Subtask<Integer> order = scope.fork(() -> fetchOrder(id));

    scope.join();           // Wait for all subtasks
    scope.throwIfFailed();  // Propagate any failure

    String userResult = user.get();
    Integer orderResult = order.get();
}
```

**Key Classes:**

| Class | Description |
|-------|-------------|
| `StructuredTaskScope<T>` | Base class for structured concurrency scopes |
| `StructuredTaskScope.ShutdownOnFailure` | Shuts down all subtasks if any fails |
| `StructuredTaskScope.ShutdownOnSuccess` | Shuts down all subtasks when one succeeds |
| `StructuredTaskScope.Subtask<T>` | Represents a forked subtask |

**Key Methods:**

| Method | Description |
|--------|-------------|
| `fork(Callable)` | Starts a subtask in a new virtual thread |
| `join()` | Waits for all subtasks to complete |
| `throwIfFailed()` | Throws if any subtask failed |
| `shutdown()` | Cancels all remaining subtasks |
| `close()` | Closes the scope (automatic with try-with-resources) |

**Syntax Rules:**

- `StructuredTaskScope` is a preview API in Java 21; compile with `--enable-preview`.
- The scope must be used with try-with-resources to ensure proper cleanup.
- Subtasks are forked using `fork(Callable)`, which returns a `Subtask<T>`.
- `join()` waits for all subtasks to finish (or for the policy to trigger).
- `throwIfFailed()` propagates the first exception if any subtask failed.

**Constraints and Limitations:**

- Preview feature in Java 21 (re-previewed in subsequent versions).
- Subtasks must be independent; dependencies between subtasks can cause deadlocks.
- The scope must not be used after it is closed.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Fetching Data in Parallel with `ShutdownOnFailure`**

```java
// File: StructuredConcurrencyDemo.java
import java.util.concurrent.StructuredTaskScope;
import java.util.concurrent.StructuredTaskScope.Subtask;

public class StructuredConcurrencyDemo {
    public static void main(String[] args) throws Exception {
        try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
            // Fork two subtasks
            Subtask<String> user = scope.fork(() -> fetchUser("123"));
            Subtask<Integer> order = scope.fork(() -> fetchOrder("123"));

            // Wait for all subtasks to complete
            scope.join();

            // Propagate any failure
            scope.throwIfFailed();

            // Use the results
            System.out.println("User: " + user.get());
            System.out.println("Order: " + order.get());
        }
    }

    static String fetchUser(String id) throws InterruptedException {
        Thread.sleep(500);
        return "User-" + id;
    }

    static Integer fetchOrder(String id) throws InterruptedException {
        Thread.sleep(700);
        return 42;
    }
}
```

**Expected Output:**
```
User: User-123
Order: 42
```

**Step-by-Step Explanation:**

1. `StructuredTaskScope.ShutdownOnFailure` creates a scope that cancels all subtasks if any fails.
2. `scope.fork(() -> fetchUser("123"))` starts a subtask in a new virtual thread.
3. `scope.fork(() -> fetchOrder("123"))` starts a second subtask.
4. `scope.join()` waits for both subtasks to complete.
5. `scope.throwIfFailed()` throws if either subtask threw an exception.
6. `user.get()` and `order.get()` retrieve the results.
7. The try-with-resources ensures the scope is closed and all subtasks are cleaned up.

**Example 2: Racing Tasks with `ShutdownOnSuccess`**

```java
// File: ShutdownOnSuccessDemo.java
import java.util.concurrent.StructuredTaskScope;

public class ShutdownOnSuccessDemo {
    public static void main(String[] args) throws Exception {
        try (var scope = new StructuredTaskScope.ShutdownOnSuccess<String>()) {
            scope.fork(() -> fetchFromPrimary());
            scope.fork(() -> fetchFromFallback());

            scope.join();  // Returns when one succeeds

            System.out.println("Result: " + scope.result());
        }
    }

    static String fetchFromPrimary() throws InterruptedException {
        Thread.sleep(1000);
        return "Primary result";
    }

    static String fetchFromFallback() throws InterruptedException {
        Thread.sleep(500);
        return "Fallback result";
    }
}
```

**Expected Output:**
```
Result: Fallback result
```

**Step-by-Step Explanation:**

1. `ShutdownOnSuccess` creates a scope that shuts down all subtasks when one succeeds.
2. Two subtasks are forked: one slow (primary), one fast (fallback).
3. `scope.join()` returns as soon as the fast subtask completes.
4. `scope.result()` returns the result of the first successful subtask.
5. The remaining subtasks are cancelled automatically.

### Real-World Cases

- **Microservices Orchestration:** Fetching data from multiple services in parallel and combining results.
- **API Aggregation:** Calling multiple APIs concurrently with automatic cancellation on failure.
- **Data Fetching:** Racing primary and fallback data sources for low latency.

---

## 5. Virtual Thread Best Practices

### Definitions

**Core Definition:** Virtual thread best practices are guidelines for using virtual threads effectively, avoiding common pitfalls, and maximizing throughput.

**Technical Definition:** Best practices derive from JEP 444 and community experience. Key principles include not pooling virtual threads, avoiding CPU-intensive work, limiting concurrency with semaphores, and being cautious with `ThreadLocal`.

**Beginner-Friendly Explanation:** Best practices help you use virtual threads correctly. The most important rule is: don't pool virtual threads. They're cheap and disposable — create a new one for each task. Also, don't use them for CPU-intensive work; they're designed for I/O-bound tasks.

### Key Best Practices Table

| Practice | Description | Source |
|----------|-------------|--------|
| **Do not pool virtual threads** | "Virtual threads are cheap and plentiful, and thus should never be pooled: A new virtual thread should be created for every application task" | Quarkus |
| **Use `Semaphore` to limit concurrency** | "A limit like 'up to 10 concurrent connections to an external API' is expressed with a semaphore, not with the size of a pool" | Zenodo |
| **Do not use for CPU-bound work** | "Virtual threads are suitable for running tasks that spend most of the time blocked... they aren't intended for long-running CPU-intensive operations" | Oracle |
| **Be cautious with `ThreadLocal`** | "Avoid caching costly resources in thread locals" when using virtual threads | OpenJDK |
| **Use structured concurrency** | "Structured concurrency groups related tasks and handles their lifecycle together" | OneUptime |
| **Use `Executors.newVirtualThreadPerTaskExecutor()`** | The recommended way to use virtual threads in server applications | Oracle |
| **Avoid `synchronized` blocking in Java 21–23** | Pinning can occur; use `ReentrantLock` instead | JEP 491 |
| **Set meaningful thread names** | Virtual threads have no default name; use `Thread.ofVirtual().name(...)` | OpenJDK |
| **Use for I/O-bound tasks** | Virtual threads excel at blocking I/O operations | Dev.java |
| **Consider virtual threads for high-throughput servers** | "Use virtual threads in high-throughput concurrent applications, especially those that consist of a great number of concurrent tasks that spend much of their time waiting" | Oracle |

### Annotated Complete Step-by-Step Code Examples

```java
// File: VirtualThreadBestPractices.java
import java.util.concurrent.*;
import java.util.stream.IntStream;

public class VirtualThreadBestPractices {
    public static void main(String[] args) throws Exception {
        // BEST PRACTICE: Do not pool virtual threads — create one per task
        try (ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor()) {

            // BEST PRACTICE: Limit concurrency with a Semaphore, not a pool
            Semaphore apiLimit = new Semaphore(10);

            IntStream.range(0, 1000).forEach(i -> {
                executor.submit(() -> {
                    apiLimit.acquire();  // Limit concurrent API calls
                    try {
                        // Simulate API call
                        Thread.sleep(100);
                        return "Result " + i;
                    } finally {
                        apiLimit.release();
                    }
                });
            });
        }
        System.out.println("All tasks completed with limited concurrency");

        // BEST PRACTICE: Use structured concurrency for related tasks
        try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
            var task1 = scope.fork(() -> fetchData("A"));
            var task2 = scope.fork(() -> fetchData("B"));

            scope.join();
            scope.throwIfFailed();

            System.out.println("Data: " + task1.get() + ", " + task2.get());
        }

        // BEST PRACTICE: Use meaningful thread names
        Thread named = Thread.ofVirtual()
            .name("data-fetcher-", 1)
            .start(() -> System.out.println("Named: " + Thread.currentThread().getName()));
        named.join();
    }

    static String fetchData(String source) throws InterruptedException {
        Thread.sleep(200);
        return "Data from " + source;
    }
}
```

**Expected Output:**
```
All tasks completed with limited concurrency
Data: Data from A, Data from B
Named: data-fetcher-1
```

**Step-by-Step Explanation:**

1. `Executors.newVirtualThreadPerTaskExecutor()` creates a new virtual thread for each task without pooling.
2. A `Semaphore` limits concurrent API calls to 10, preventing resource exhaustion.
3. `StructuredTaskScope.ShutdownOnFailure` groups related tasks and handles their lifecycle together.
4. `Thread.ofVirtual().name(...)` gives the virtual thread a meaningful name for debugging.

### Real-World Cases

- **Web Servers:** Using `newVirtualThreadPerTaskExecutor()` for request handling with semaphores for downstream limits.
- **Microservices:** Using structured concurrency for parallel service calls with automatic cancellation.
- **Data Pipelines:** Processing I/O-bound data with virtual threads and semaphores for backpressure.

---

## References Links

- Virtual Threads — Oracle Java Core Libraries Developer Guide - https://docs.oracle.com/en/java/javase/26/core/virtual-threads.html
- JEP 444: Virtual Threads — OpenJDK - https://openjdk.org/jeps/444
- Virtual Threads — Dev.java - https://dev.java/learn/api/virtual-threads/
- Thread.Builder.OfVirtual — Java Platform SE 21 API Specification - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Thread.Builder.OfVirtual.html
- StructuredTaskScope — JSR 396: Java SE 21 API Specification - https://cr.openjdk.org/~iris/se/21/latestSpec/apidiffs/java.base/java/util/concurrent/StructuredTaskScope.html
- To Cache or Not to Cache Virtual Threads — Quarkus - https://quarkus.io
- JEP 491: Synchronize Virtual Threads without Pinning — OpenJDK - https://openjdk.org/jeps/491
- Structured Concurrency — JCP - https://jcp.org
- Virtual Threads, Structured Concurrency, and Scoped Values — MPI ebooks - https://ebooks.mpdl.mpg.de