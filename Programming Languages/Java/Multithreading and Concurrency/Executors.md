# Java Executors: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** The Executor framework, introduced in Java 5, is a high-level concurrency framework in `java.util.concurrent` that decouples task submission from execution mechanics. Instead of manually creating and managing `Thread` objects, developers submit `Runnable` or `Callable` tasks to an executor that manages a pool of worker threads.

**Technical Definition:** "The `java.util.concurrent` package provides the Executor framework which offers a mechanism for executing tasks concurrently. A task is a logical unit of work encapsulated by a class that implements `Runnable` or `Callable`. The Executor framework allows task submission to be decoupled from low level scheduling and thread management details. It provides the thread pool mechanism that allows a system to degrade gracefully when presented with more requests than the system can handle simultaneously". "The `Executor` interface is the core interface of the framework and is extended by the `ExecutorService` interface that provides facilities for thread pool termination and obtaining return values of tasks (Futures)". "The `ExecutorService` interface is further extended by the `ScheduledExecutorService` interface that provides a way to run tasks periodically or after some delay". "The `Executors` class provides several factory and utility methods that are pre-configured with commonly used configurations".

**Beginner-Friendly Explanation:** Imagine a restaurant kitchen. Without an executor framework, each order would require hiring a new chef, training them, and firing them after the order — extremely wasteful. The Executor framework is like having a permanent team of chefs (a thread pool). When orders (tasks) come in, they're placed on a queue. Available chefs pick up orders and prepare them. If the restaurant gets busy, more chefs can be added (up to a limit). If it's quiet, some chefs can go home. The `Executor` interface is the job assignment desk; `ExecutorService` adds the ability to track orders (Futures) and close the kitchen gracefully. The `Executors` factory methods are pre-set kitchen configurations for common scenarios.

### Key Characteristics

- **Decoupling:** Task submission is decoupled from thread creation and management.
- **Thread Pooling:** Threads are reused across tasks, reducing the overhead of thread creation.
- **Bounded Resources:** Thread pools limit the number of concurrent threads, preventing resource exhaustion.
- **Task Queuing:** Tasks are queued when all threads are busy, providing backpressure.
- **Future Results:** `ExecutorService` returns `Future` objects for tracking task progress and retrieving results.
- **Scheduled Execution:** `ScheduledExecutorService` supports delayed and periodic task execution.
- **Work Stealing:** `ForkJoinPool` uses a work-stealing algorithm for efficient parallel processing.
- **Graceful Shutdown:** Executors support orderly shutdown, allowing submitted tasks to complete.

### Prerequisites

- Basic understanding of Java threads and the `Thread` class
- Familiarity with `Runnable` and `Callable` interfaces
- Knowledge of `java.util.concurrent` package basics
- Understanding of blocking queues

### Related Programming Areas

- **Concurrency Fundamentals:** The broader topic of concurrent programming.
- **Thread Pools:** Managing worker threads for efficient task execution.
- **CompletableFuture:** Asynchronous programming with composable futures.
- **Parallel Streams:** The Fork/Join framework powers Java's parallel streams.
- **Blocking Queues:** Used internally by thread pools to hold tasks.

### Core Concepts / Features

---

## 1. `Executor`

### Definitions

**Core Definition:** `Executor` is the simplest interface in the framework, defining a single method `execute(Runnable)` that replaces the usual thread creation idiom.

**Technical Definition:** "An object that executes submitted `Runnable` tasks. This interface provides a way of decoupling task submission from the mechanics of how each task will be run, including details of thread use, scheduling, etc. An `Executor` is normally used instead of explicitly creating threads". The `Executor` interface defines a single method: `void execute(Runnable command)`.

**Beginner-Friendly Explanation:** `Executor` is the most basic contract: "I can run a task for you." It doesn't say how, when, or where — just that it will execute the task. This simplicity allows different implementations: run it immediately, run it in a thread pool, run it in the calling thread, etc.

### Purposes

- To decouple task submission from execution mechanics
- To provide a standard interface for executing `Runnable` tasks
- To allow different execution strategies (immediate, pooled, threaded) without changing the calling code
- To serve as the foundation for the more feature-rich `ExecutorService`

### Syntax Rules and Structure

**Complete General Syntax:**

```java
Executor executor = Executors.newFixedThreadPool(5);
executor.execute(() -> System.out.println("Task running"));
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `execute(Runnable)` | Submits a `Runnable` task for execution |
| No return value | `execute` returns `void` |

**Syntax Rules:**

- `execute()` accepts only `Runnable`, not `Callable`.
- `execute()` does not return a `Future`; there's no way to track the task.
- Exceptions thrown by the task propagate to the thread's uncaught exception handler.

### Annotated Code Example

```java
// File: ExecutorDemo.java
import java.util.concurrent.Executor;

public class ExecutorDemo {
    public static void main(String[] args) {
        // Create a simple executor that runs tasks in the calling thread
        Executor directExecutor = Runnable::run;

        // Execute tasks
        directExecutor.execute(() -> System.out.println("Task 1"));
        directExecutor.execute(() -> System.out.println("Task 2"));

        // Using a thread pool executor
        Executor poolExecutor = java.util.concurrent.Executors.newFixedThreadPool(2);
        poolExecutor.execute(() -> System.out.println("Pool task 1"));
        poolExecutor.execute(() -> System.out.println("Pool task 2"));
    }
}
```

**Expected Output (order may vary):**
```
Task 1
Task 2
Pool task 1
Pool task 2
```

**Step-by-Step Explanation:**

1. `Runnable::run` creates a direct executor that runs tasks in the calling thread — no new thread is created.
2. `poolExecutor` uses a fixed thread pool with 2 threads.
3. Both executors implement the `Executor` interface, so calling code doesn't need to know the execution strategy.

### Real-World Cases

- **Framework Integration:** Libraries that accept an `Executor` for task execution.
- **Simple Task Submission:** When you only need to submit tasks and don't care about results.
- **Testing:** Using a direct executor in tests to run tasks synchronously.

---

## 2. `ExecutorService`

### Definitions

**Core Definition:** `ExecutorService` extends `Executor` with methods for lifecycle management, task tracking (Futures), and batch task submission.

**Technical Definition:** "An `Executor` that provides methods to manage termination and methods that can produce a `Future` for tracking progress of one or more asynchronous tasks". It adds methods like `shutdown()`, `shutdownNow()`, `submit()`, `invokeAll()`, and `invokeAny()`.

**Beginner-Friendly Explanation:** `ExecutorService` is like a full-service project manager. Not only can it run tasks (like `Executor`), but it can also tell you when tasks are done (Futures), shut down gracefully, and manage batches of tasks.

### Purposes

- To provide lifecycle management (shutdown, awaitTermination)
- To support task submission with results (via `Future`)
- To enable batch task execution (`invokeAll`, `invokeAny`)
- To provide a richer API than the basic `Executor`

### Syntax Rules and Structure

**Key Methods:**

| Method | Description |
|--------|-------------|
| `submit(Runnable)` | Submits a task, returns a `Future` |
| `submit(Callable<T>)` | Submits a value-returning task, returns `Future<T>` |
| `invokeAll(Collection)` | Executes all tasks, returns list of Futures |
| `invokeAny(Collection)` | Executes all, returns result of the first successful |
| `shutdown()` | Initiates orderly shutdown |
| `shutdownNow()` | Attempts to stop all tasks, returns pending tasks |
| `awaitTermination(long, TimeUnit)` | Waits for shutdown to complete |
| `isShutdown()` / `isTerminated()` | Lifecycle state checks |

**Syntax Rules:**

- `submit()` returns a `Future` that can be used to check completion, get results, or cancel.
- `shutdown()` stops accepting new tasks but lets submitted tasks finish.
- `shutdownNow()` attempts to cancel running tasks and returns queued tasks.
- `awaitTermination()` should be called after `shutdown()` to wait for completion.

### Annotated Code Example

```java
// File: ExecutorServiceDemo.java
import java.util.concurrent.*;

public class ExecutorServiceDemo {
    public static void main(String[] args) throws Exception {
        ExecutorService executor = Executors.newFixedThreadPool(3);

        // Submit a Runnable (no result)
        Future<?> future1 = executor.submit(() -> {
            System.out.println("Runnable task running");
        });

        // Submit a Callable (with result)
        Future<Integer> future2 = executor.submit(() -> {
            Thread.sleep(500);
            return 42;
        });

        // Get the result (blocks until available)
        System.out.println("Callable result: " + future2.get());

        // Graceful shutdown
        executor.shutdown();
        if (!executor.awaitTermination(5, TimeUnit.SECONDS)) {
            executor.shutdownNow();
        }

        System.out.println("Executor terminated: " + executor.isTerminated());
    }
}
```

**Expected Output:**
```
Runnable task running
Callable result: 42
Executor terminated: true
```

**Step-by-Step Explanation:**

1. `submit(Runnable)` returns a `Future<?>` that completes when the task finishes.
2. `submit(Callable)` returns a `Future<Integer>` whose `get()` method blocks until the result is available.
3. `shutdown()` initiates orderly shutdown; no new tasks are accepted.
4. `awaitTermination()` waits up to 5 seconds for all tasks to complete.
5. `isTerminated()` returns `true` after all tasks have finished.

### Real-World Cases

- **Web Servers:** Handling concurrent requests with a thread pool.
- **Batch Processing:** Submitting batches of tasks and collecting results.
- **Background Jobs:** Running scheduled or asynchronous tasks.

---

## 3. `Executors`

### Definitions

**Core Definition:** `Executors` is a utility class that provides factory methods for creating pre-configured `ExecutorService` and `ScheduledExecutorService` instances.

**Technical Definition:** "The `Executors` class provides several factory and utility methods that are pre-configured with commonly used configurations of `Executor`, `ExecutorService` and other related interfaces".

**Beginner-Friendly Explanation:** `Executors` is like a vending machine for thread pools. You select the type you want (fixed, cached, single, scheduled, work-stealing), and it dispenses a ready-to-use executor with sensible defaults.

### Purposes

- To provide convenient factory methods for common thread pool configurations
- To eliminate the need for manual `ThreadPoolExecutor` configuration for common cases
- To offer thread factory and callable utility methods
- To standardise the creation of executors

### Factory Methods

| Method | Description |
|--------|-------------|
| `newFixedThreadPool(int nThreads)` | Creates a thread pool with a fixed number of threads |
| `newCachedThreadPool()` | Creates a thread pool that creates new threads as needed, reusing idle ones |
| `newSingleThreadExecutor()` | Creates an executor with a single worker thread |
| `newScheduledThreadPool(int corePoolSize)` | Creates a thread pool for delayed/periodic tasks |
| `newSingleThreadScheduledExecutor()` | Creates a single-threaded scheduled executor |
| `newWorkStealingPool()` | Creates a work-stealing thread pool (Java 8+) |
| `newVirtualThreadPerTaskExecutor()` | Creates an executor that starts a new virtual thread per task (Java 21+) |

### Annotated Code Example

```java
// File: ExecutorsFactoryDemo.java
import java.util.concurrent.*;

public class ExecutorsFactoryDemo {
    public static void main(String[] args) throws Exception {
        // Fixed thread pool
        ExecutorService fixed = Executors.newFixedThreadPool(4);
        System.out.println("Fixed pool: " + fixed.getClass().getSimpleName());

        // Cached thread pool
        ExecutorService cached = Executors.newCachedThreadPool();
        System.out.println("Cached pool: " + cached.getClass().getSimpleName());

        // Single thread executor
        ExecutorService single = Executors.newSingleThreadExecutor();
        System.out.println("Single executor: " + single.getClass().getSimpleName());

        // Scheduled thread pool
        ScheduledExecutorService scheduled = Executors.newScheduledThreadPool(2);
        System.out.println("Scheduled pool: " + scheduled.getClass().getSimpleName());

        // Work-stealing pool (Java 8+)
        ExecutorService workStealing = Executors.newWorkStealingPool();
        System.out.println("Work-stealing pool: " + workStealing.getClass().getSimpleName());

        // Shut down all
        fixed.shutdown();
        cached.shutdown();
        single.shutdown();
        scheduled.shutdown();
        workStealing.shutdown();
    }
}
```

**Expected Output:**
```
Fixed pool: ThreadPoolExecutor
Cached pool: ThreadPoolExecutor
Single executor: FinalizableDelegatedExecutorService
Scheduled pool: ScheduledThreadPoolExecutor
Work-stealing pool: ForkJoinPool
```

**Step-by-Step Explanation:**

1. Each factory method returns an `ExecutorService` implementation configured for a specific use case.
2. `newFixedThreadPool` and `newCachedThreadPool` return `ThreadPoolExecutor` instances.
3. `newSingleThreadExecutor` returns a wrapped executor that ensures a single thread.
4. `newScheduledThreadPool` returns a `ScheduledThreadPoolExecutor`.
5. `newWorkStealingPool` returns a `ForkJoinPool`.

### Real-World Cases

- **Web Servers:** `newFixedThreadPool` for bounded request handling.
- **Short-Lived Tasks:** `newCachedThreadPool` for bursts of short tasks.
- **Background Processing:** `newSingleThreadExecutor` for sequential background work.
- **Scheduled Jobs:** `newScheduledThreadPool` for periodic tasks.

---

## 4. Thread Pools

### Definitions

**Core Definition:** A thread pool is a managed collection of worker threads that execute submitted tasks, reusing threads to avoid the overhead of creation and destruction.

**Technical Definition:** "Thread pools address two different problems: they usually provide improved performance when executing large numbers of asynchronous tasks, due to reduced per-task invocation overhead, and they provide a means of bounding and managing the resources, including threads, consumed when executing a collection of tasks".

**Beginner-Friendly Explanation:** A thread pool is like a team of workers who stay on staff. When work arrives, an available worker handles it. When the work is done, the worker doesn't leave — they wait for the next task. This is much more efficient than hiring and firing a worker for every single task.

### Purposes

- To reduce the overhead of thread creation and destruction
- To bound the number of concurrent threads and manage resources
- To provide task queuing and backpressure
- To improve performance for large numbers of small tasks

### Key Concepts

| Concept | Description |
|---------|-------------|
| Core Pool Size | Number of threads to keep in the pool, even if idle |
| Maximum Pool Size | Maximum number of threads allowed in the pool |
| Keep-Alive Time | How long excess idle threads wait before terminating |
| Work Queue | Queue holding tasks before they are executed |
| Thread Factory | Factory for creating new threads |
| Rejection Handler | Policy for handling tasks when the queue is full |

### Annotated Code Example

```java
// File: ThreadPoolDemo.java
import java.util.concurrent.*;

public class ThreadPoolDemo {
    public static void main(String[] args) throws Exception {
        // Create a thread pool with 2 core threads, max 4, and a bounded queue
        ThreadPoolExecutor executor = new ThreadPoolExecutor(
            2,                              // corePoolSize
            4,                              // maximumPoolSize
            60, TimeUnit.SECONDS,           // keepAliveTime
            new ArrayBlockingQueue<>(10),   // workQueue (bounded)
            Executors.defaultThreadFactory(),
            new ThreadPoolExecutor.CallerRunsPolicy()  // rejection handler
        );

        // Submit 20 tasks
        for (int i = 1; i <= 20; i++) {
            final int taskId = i;
            executor.execute(() -> {
                System.out.println(Thread.currentThread().getName() +
                    " executing task " + taskId);
                try { Thread.sleep(100); } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            });
        }

        executor.shutdown();
        executor.awaitTermination(10, TimeUnit.SECONDS);
        System.out.println("All tasks completed");
    }
}
```

**Expected Output (interleaved):**
```
pool-1-thread-1 executing task 1
pool-1-thread-2 executing task 2
main executing task 11
pool-1-thread-1 executing task 3
...
All tasks completed
```

**Step-by-Step Explanation:**

1. The pool starts with 2 core threads.
2. Tasks 1-2 run on the core threads.
3. Tasks 3-12 are queued in the `ArrayBlockingQueue` (capacity 10).
4. Tasks 13-20 cause the pool to grow to the maximum of 4 threads.
5. Task 11 is rejected (queue full, max threads busy) and handled by `CallerRunsPolicy` — it runs in the main thread.
6. After all tasks complete, `shutdown()` and `awaitTermination()` clean up.

### Real-World Cases

- **Web Servers:** Bounded thread pools for request handling.
- **Data Processing:** Parallel processing of large datasets.
- **Background Services:** Managed thread pools for asynchronous tasks.

---

## 5. `newFixedThreadPool()`

### Definitions

**Core Definition:** `newFixedThreadPool(int nThreads)` creates a thread pool with a fixed number of threads that are reused to execute tasks from an unbounded queue.

**Technical Definition:** "Creates a thread pool that reuses a fixed number of threads operating off a shared unbounded queue. At any point, at most `nThreads` threads will be active processing tasks. If additional tasks are submitted when all threads are active, they will wait in the queue until a thread is available".

**Beginner-Friendly Explanation:** `newFixedThreadPool` is like having exactly N permanent workers. If more than N tasks arrive, the extras wait in a queue. The number of workers never changes.

### Purposes

- To bound the number of concurrent threads to a fixed value
- To provide predictable resource usage
- To reuse threads for multiple tasks
- To handle bursts of tasks with queuing

### Syntax Rules and Structure

```java
ExecutorService executor = Executors.newFixedThreadPool(4);
```

**Constraints and Limitations:**

- Uses an unbounded `LinkedBlockingQueue` internally, which can lead to `OutOfMemoryError` if tasks accumulate faster than they are processed.
- The number of threads cannot grow or shrink.
- For production, consider using `ThreadPoolExecutor` with a bounded queue.

### Annotated Code Example

```java
// File: FixedThreadPoolDemo.java
import java.util.concurrent.*;

public class FixedThreadPoolDemo {
    public static void main(String[] args) throws Exception {
        ExecutorService executor = Executors.newFixedThreadPool(3);

        for (int i = 1; i <= 10; i++) {
            final int taskId = i;
            executor.submit(() -> {
                System.out.println(Thread.currentThread().getName() +
                    " processing task " + taskId);
                try { Thread.sleep(200); } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            });
        }

        executor.shutdown();
        executor.awaitTermination(10, TimeUnit.SECONDS);
        System.out.println("Done");
    }
}
```

**Expected Output (interleaved):**
```
pool-1-thread-1 processing task 1
pool-1-thread-2 processing task 2
pool-1-thread-3 processing task 3
pool-1-thread-1 processing task 4
...
Done
```

### Real-World Cases

- **Web Servers:** Fixed pool of request handler threads.
- **Database Connections:** Fixed pool of connection worker threads.
- **Image Processing:** Fixed number of worker threads for parallel image operations.

---

## 6. `newCachedThreadPool()`

### Definitions

**Core Definition:** `newCachedThreadPool()` creates a thread pool that creates new threads as needed and reuses idle threads, with no upper bound on the number of threads.

**Technical Definition:** "Creates a thread pool that creates new threads as needed, but will reuse previously constructed threads when they are available. These pools will typically improve the performance of programs that execute many short-lived asynchronous tasks. Calls to `execute` will reuse previously constructed threads if available. If no existing thread is available, a new thread will be created and added to the pool. Threads that have not been used for sixty seconds are terminated and removed from the cache. Thus, a pool that remains idle for long enough will not consume any resources".

**Beginner-Friendly Explanation:** `newCachedThreadPool` is like a temp agency. If you need workers, they're hired immediately. When work is done, they wait for 60 seconds. If no more work arrives, they're let go. If a burst of work arrives, more temps are hired instantly.

### Purposes

- To handle bursts of short-lived asynchronous tasks
- To provide immediate thread creation when needed
- To automatically reclaim idle threads
- To avoid queuing delays for short tasks

### Syntax Rules and Structure

```java
ExecutorService executor = Executors.newCachedThreadPool();
```

**Constraints and Limitations:**

- No upper bound on threads — can cause resource exhaustion under high load.
- Uses a `SynchronousQueue` internally, so tasks are handed directly to threads.
- Threads are terminated after 60 seconds of inactivity.
- **Not recommended for production** without careful consideration; consider `ThreadPoolExecutor` with a maximum pool size.

### Annotated Code Example

```java
// File: CachedThreadPoolDemo.java
import java.util.concurrent.*;

public class CachedThreadPoolDemo {
    public static void main(String[] args) throws Exception {
        ExecutorService executor = Executors.newCachedThreadPool();

        for (int i = 1; i <= 5; i++) {
            final int taskId = i;
            executor.submit(() -> {
                System.out.println(Thread.currentThread().getName() +
                    " running task " + taskId);
                try { Thread.sleep(500); } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            });
        }

        executor.shutdown();
        executor.awaitTermination(5, TimeUnit.SECONDS);
        System.out.println("Done");
    }
}
```

**Expected Output (interleaved):**
```
pool-1-thread-1 running task 1
pool-1-thread-2 running task 2
pool-1-thread-3 running task 3
pool-1-thread-4 running task 4
pool-1-thread-5 running task 5
Done
```

### Real-World Cases

- **Short Tasks:** Processing many small, independent tasks.
- **Bursty Workloads:** Handling occasional bursts of activity.
- **Testing:** Quick-and-dirty thread pools for prototypes.

---

## 7. `newSingleThreadExecutor()`

### Definitions

**Core Definition:** `newSingleThreadExecutor()` creates an executor with a single worker thread that processes tasks sequentially from an unbounded queue.

**Technical Definition:** "Creates an Executor that uses a single worker thread operating off an unbounded queue. (Note however that if this single thread terminates due to a failure during execution prior to shutdown, a new one will take its place if needed to execute subsequent tasks.) Tasks are guaranteed to execute sequentially, and no more than one task will be active at any given time".

**Beginner-Friendly Explanation:** `newSingleThreadExecutor` is like having exactly one worker. Tasks are processed one at a time, in order. It's perfect when you need sequential execution but still want the benefits of the executor framework.

### Purposes

- To ensure sequential task execution
- To replace a dedicated background thread with a managed executor
- To provide a single-threaded event loop
- To guarantee FIFO task ordering

### Syntax Rules and Structure

```java
ExecutorService executor = Executors.newSingleThreadExecutor();
```

**Constraints and Limitations:**

- Uses an unbounded `LinkedBlockingQueue`.
- Only one thread, so no parallelism.
- If the thread dies, a new one is created.

### Annotated Code Example

```java
// File: SingleThreadExecutorDemo.java
import java.util.concurrent.*;

public class SingleThreadExecutorDemo {
    public static void main(String[] args) throws Exception {
        ExecutorService executor = Executors.newSingleThreadExecutor();

        for (int i = 1; i <= 5; i++) {
            final int taskId = i;
            executor.submit(() -> {
                System.out.println("Task " + taskId + " on " +
                    Thread.currentThread().getName());
            });
        }

        executor.shutdown();
        executor.awaitTermination(5, TimeUnit.SECONDS);
    }
}
```

**Expected Output:**
```
Task 1 on pool-1-thread-1
Task 2 on pool-1-thread-1
Task 3 on pool-1-thread-1
Task 4 on pool-1-thread-1
Task 5 on pool-1-thread-1
```

### Real-World Cases

- **Sequential Processing:** Tasks that must run in order.
- **Event Loops:** Single-threaded event processing.
- **Database Writes:** Ensuring serialised writes.

---

## 8. `newScheduledThreadPool()`

### Definitions

**Core Definition:** `newScheduledThreadPool(int corePoolSize)` creates a thread pool that can schedule tasks to run after a delay or periodically.

**Technical Definition:** "Creates a thread pool that can schedule commands to run after a given delay, or to execute periodically". Returns a `ScheduledExecutorService`.

**Beginner-Friendly Explanation:** `newScheduledThreadPool` is like an alarm clock service. You can set tasks to run once after a delay, repeatedly at a fixed rate, or repeatedly with a fixed delay between completions.

### Purposes

- To schedule tasks for delayed execution
- To execute tasks periodically (fixed rate or fixed delay)
- To replace `Timer` with a more robust, multi-threaded solution
- To implement heartbeats, polling, and cleanup jobs

### Syntax Rules and Structure

```java
ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(2);

// Run once after delay
scheduler.schedule(task, 5, TimeUnit.SECONDS);

// Run repeatedly at fixed rate
scheduler.scheduleAtFixedRate(task, initialDelay, period, TimeUnit.SECONDS);

// Run repeatedly with fixed delay between completions
scheduler.scheduleWithFixedDelay(task, initialDelay, delay, TimeUnit.SECONDS);
```

**Key Methods:**

| Method | Description |
|--------|-------------|
| `schedule(Runnable, delay, unit)` | Runs once after the specified delay |
| `schedule(Callable, delay, unit)` | Runs once, returns a `ScheduledFuture` |
| `scheduleAtFixedRate(Runnable, init, period, unit)` | Runs periodically at a fixed rate |
| `scheduleWithFixedDelay(Runnable, init, delay, unit)` | Runs with a fixed delay between completions |

### Annotated Code Example

```java
// File: ScheduledThreadPoolDemo.java
import java.util.concurrent.*;

public class ScheduledThreadPoolDemo {
    public static void main(String[] args) throws Exception {
        ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(2);

        // Run once after 2 seconds
        scheduler.schedule(() -> System.out.println("Delayed task at " +
            System.currentTimeMillis()), 2, TimeUnit.SECONDS);

        // Run every 1 second, starting after 1 second
        ScheduledFuture<?> periodic = scheduler.scheduleAtFixedRate(() ->
            System.out.println("Periodic task at " + System.currentTimeMillis()),
            1, 1, TimeUnit.SECONDS);

        // Let it run for 4 seconds
        Thread.sleep(4000);

        // Cancel the periodic task
        periodic.cancel(false);

        scheduler.shutdown();
        scheduler.awaitTermination(5, TimeUnit.SECONDS);
        System.out.println("Scheduler stopped");
    }
}
```

**Expected Output (timestamps vary):**
```
Periodic task at 1700000001000
Delayed task at 1700000002000
Periodic task at 1700000002000
Periodic task at 1700000003000
Periodic task at 1700000004000
Scheduler stopped
```

### Real-World Cases

- **Heartbeat Monitoring:** Periodic health checks.
- **Cache Cleanup:** Scheduled eviction of expired entries.
- **Polling:** Regularly checking a resource for changes.
- **Delayed Retries:** Retrying failed operations after a delay.

---

## 9. `newWorkStealingPool()`

### Definitions

**Core Definition:** `newWorkStealingPool()` creates a work-stealing thread pool based on the Fork/Join framework, suitable for CPU-intensive parallel tasks.

**Technical Definition:** "Creates a work-stealing thread pool using all available processors as its target parallelism level". Internally, it creates a `ForkJoinPool` that uses a work-stealing algorithm: "Worker threads that run out of things to do can steal tasks from other threads that are still busy".

**Beginner-Friendly Explanation:** `newWorkStealingPool` is like a team where idle workers help busy workers. If one worker has a long queue of tasks and another is idle, the idle worker "steals" some of the busy worker's tasks. This keeps everyone busy and improves overall throughput.

### Purposes

- To efficiently process recursive, divide-and-conquer tasks
- To utilise all available CPU cores
- To balance workload dynamically through work-stealing
- To power parallel streams and `CompletableFuture`

### Syntax Rules and Structure

```java
ExecutorService executor = Executors.newWorkStealingPool();
// or with explicit parallelism
ExecutorService executor = Executors.newWorkStealingPool(4);
```

**Constraints and Limitations:**

- Uses a `ForkJoinPool` internally.
- Tasks should be non-blocking and CPU-intensive.
- Not ideal for I/O-bound tasks or tasks with dependencies.
- Does not guarantee task execution order.

### Annotated Code Example

```java
// File: WorkStealingPoolDemo.java
import java.util.concurrent.*;

public class WorkStealingPoolDemo {
    public static void main(String[] args) throws Exception {
        ExecutorService executor = Executors.newWorkStealingPool(4);

        // Submit CPU-intensive tasks
        for (int i = 1; i <= 8; i++) {
            final int taskId = i;
            executor.submit(() -> {
                long sum = 0;
                for (int j = 0; j < 10_000_000; j++) {
                    sum += j;
                }
                System.out.println("Task " + taskId + " completed on " +
                    Thread.currentThread().getName());
            });
        }

        executor.shutdown();
        executor.awaitTermination(30, TimeUnit.SECONDS);
        System.out.println("All tasks completed");
    }
}
```

**Expected Output (order may vary):**
```
Task 1 completed on ForkJoinPool-1-worker-1
Task 2 completed on ForkJoinPool-1-worker-2
Task 3 completed on ForkJoinPool-1-worker-3
Task 4 completed on ForkJoinPool-1-worker-4
Task 5 completed on ForkJoinPool-1-worker-1
...
All tasks completed
```

### Real-World Cases

- **Parallel Streams:** `list.parallelStream()` uses the common `ForkJoinPool`.
- **Recursive Algorithms:** Merge sort, quicksort, tree traversal.
- **Image Processing:** Applying filters to image segments in parallel.
- **Scientific Computing:** Monte Carlo simulations, matrix operations.

---

## 10. `ThreadPoolExecutor`

### Definitions

**Core Definition:** `ThreadPoolExecutor` is the core implementation of `ExecutorService`, providing a flexible, tunable thread pool with configurable core/max pool sizes, keep-alive times, work queues, and rejection policies.

**Technical Definition:** "An `ExecutorService` that executes each submitted task using one of possibly several pooled threads, normally configured using `Executors` factory methods". "To be useful across a wide range of contexts, this class provides many adjustable parameters and extensibility hooks".

**Beginner-Friendly Explanation:** `ThreadPoolExecutor` is the engine under the hood of the `Executors` factory methods. It's fully configurable: you decide how many core threads to keep, how many maximum threads to allow, how long idle threads wait before terminating, what kind of queue to use, and what to do when the queue is full.

### Constructor Parameters

| Parameter | Description |
|-----------|-------------|
| `corePoolSize` | Number of threads to keep in the pool, even if idle |
| `maximumPoolSize` | Maximum number of threads allowed in the pool |
| `keepAliveTime` | Time excess idle threads wait before terminating |
| `unit` | Time unit for `keepAliveTime` |
| `workQueue` | Queue for holding tasks before execution |
| `threadFactory` | Factory for creating new threads |
| `handler` | Handler for rejected tasks |

### Rejection Policies

| Policy | Description |
|--------|-------------|
| `AbortPolicy` | Throws `RejectedExecutionException` (default) |
| `CallerRunsPolicy` | Runs the task in the caller's thread |
| `DiscardPolicy` | Silently discards the task |
| `DiscardOldestPolicy` | Discards the oldest queued task, then retries |

### Annotated Code Example

```java
// File: ThreadPoolExecutorDemo.java
import java.util.concurrent.*;

public class ThreadPoolExecutorDemo {
    public static void main(String[] args) throws Exception {
        ThreadPoolExecutor executor = new ThreadPoolExecutor(
            2,                              // corePoolSize
            4,                              // maximumPoolSize
            60, TimeUnit.SECONDS,           // keepAliveTime
            new ArrayBlockingQueue<>(10),   // bounded work queue
            new ThreadFactory() {
                private int count = 0;
                public Thread newThread(Runnable r) {
                    return new Thread(r, "CustomWorker-" + (++count));
                }
            },
            new ThreadPoolExecutor.CallerRunsPolicy()
        );

        // Submit tasks
        for (int i = 1; i <= 20; i++) {
            final int taskId = i;
            executor.execute(() -> {
                System.out.println(Thread.currentThread().getName() +
                    " executing task " + taskId);
                try { Thread.sleep(100); } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            });
        }

        executor.shutdown();
        executor.awaitTermination(30, TimeUnit.SECONDS);
        System.out.println("All done. Pool size: " + executor.getPoolSize());
    }
}
```

**Expected Output (interleaved):**
```
CustomWorker-1 executing task 1
CustomWorker-2 executing task 2
CustomWorker-3 executing task 11
CustomWorker-4 executing task 12
main executing task 21
...
All done. Pool size: 0
```

**Step-by-Step Explanation:**

1. The pool starts with 2 core threads.
2. Tasks 1-2 run on core threads.
3. Tasks 3-12 are queued (capacity 10).
4. Tasks 13-20 cause the pool to grow to 4 threads.
5. Tasks beyond capacity are handled by `CallerRunsPolicy` (run in `main`).
6. After shutdown, the pool size drops to 0 as threads terminate.

### Real-World Cases

- **Production Web Servers:** Fine-tuned thread pools with bounded queues.
- **Database Connection Pools:** Managed worker threads for database operations.
- **Custom Thread Naming:** Thread factories for debugging and monitoring.

---

## 11. `ScheduledThreadPoolExecutor`

### Definitions

**Core Definition:** `ScheduledThreadPoolExecutor` extends `ThreadPoolExecutor` to support delayed and periodic task execution.

**Technical Definition:** "A `ThreadPoolExecutor` that can additionally schedule commands to run after a given delay, or to execute periodically. This class is preferable to `Timer` when multiple worker threads are needed, or when the additional flexibility or capabilities of `ThreadPoolExecutor` (which this class extends) are required".

**Beginner-Friendly Explanation:** `ScheduledThreadPoolExecutor` is like a `ThreadPoolExecutor` with a built-in alarm clock. You can tell it "run this task in 5 seconds" or "run this task every 10 seconds."

### Key Methods

| Method | Description |
|--------|-------------|
| `schedule(Runnable, delay, unit)` | Runs once after the delay |
| `scheduleAtFixedRate(Runnable, init, period, unit)` | Runs at a fixed rate |
| `scheduleWithFixedDelay(Runnable, init, delay, unit)` | Runs with fixed delay between completions |
| `setRemoveOnCancelPolicy(boolean)` | Removes cancelled tasks from the queue immediately |

### Syntax Rules and Structure

```java
ScheduledThreadPoolExecutor scheduler = new ScheduledThreadPoolExecutor(2);

// Run once after 5 seconds
scheduler.schedule(task, 5, TimeUnit.SECONDS);

// Run every 10 seconds starting after 1 second
scheduler.scheduleAtFixedRate(task, 1, 10, TimeUnit.SECONDS);

// Run 3 seconds after the previous run completes, starting after 1 second
scheduler.scheduleWithFixedDelay(task, 1, 3, TimeUnit.SECONDS);
```

### Annotated Code Example

```java
// File: ScheduledExecutorDemo.java
import java.util.concurrent.*;

public class ScheduledExecutorDemo {
    public static void main(String[] args) throws Exception {
        ScheduledThreadPoolExecutor scheduler = new ScheduledThreadPoolExecutor(2);

        // Fixed-rate: runs at 0s, 1s, 2s, 3s, ...
        ScheduledFuture<?> fixedRate = scheduler.scheduleAtFixedRate(() ->
            System.out.println("Fixed rate: " + System.currentTimeMillis()),
            0, 1, TimeUnit.SECONDS);

        // Fixed-delay: runs 1s after previous completes
        ScheduledFuture<?> fixedDelay = scheduler.scheduleWithFixedDelay(() ->
            System.out.println("Fixed delay: " + System.currentTimeMillis()),
            0, 1, TimeUnit.SECONDS);

        Thread.sleep(3500);
        fixedRate.cancel(false);
        fixedDelay.cancel(false);

        scheduler.shutdown();
        scheduler.awaitTermination(5, TimeUnit.SECONDS);
        System.out.println("Scheduler stopped");
    }
}
```

**Expected Output (timestamps vary):**
```
Fixed rate: 1700000000000
Fixed delay: 1700000000000
Fixed rate: 1700000001000
Fixed delay: 1700000001000
Fixed rate: 1700000002000
Fixed delay: 1700000002000
Fixed rate: 1700000003000
Fixed delay: 1700000003000
Scheduler stopped
```

### Real-World Cases

- **Heartbeat Monitoring:** Periodic health checks.
- **Cache Eviction:** Scheduled cleanup of expired entries.
- **Polling:** Regularly checking external resources.
- **Retry Logic:** Scheduling retries with delays.

---

## 12. `ForkJoinPool`

### Definitions

**Core Definition:** `ForkJoinPool` is a specialised `ExecutorService` implementation that uses a work-stealing algorithm for efficient parallel processing of recursive, divide-and-conquer tasks.

**Technical Definition:** "The fork/join framework is an implementation of the `ExecutorService` interface that helps you take advantage of multiple processors. It is designed for work that can be broken into smaller pieces recursively. The goal is to use all the available processing power to enhance the performance of your application". "The fork/join framework is distinct because it uses a work-stealing algorithm. Worker threads that run out of things to do can steal tasks from other threads that are still busy".

**Beginner-Friendly Explanation:** `ForkJoinPool` is like a team of workers where each worker has their own to-do list. When a worker finishes their list, they look at other workers' lists and "steal" tasks from the bottom. This keeps everyone busy and balances the workload automatically.

### Purposes

- To efficiently parallelise recursive, divide-and-conquer algorithms
- To utilise all available CPU cores
- To dynamically balance workload through work-stealing
- To power parallel streams and `CompletableFuture`
- To provide high throughput for CPU-intensive tasks

### Key Classes

| Class | Description |
|-------|-------------|
| `ForkJoinPool` | The pool that executes `ForkJoinTask`s |
| `ForkJoinTask<V>` | Abstract base class for tasks |
| `RecursiveTask<V>` | A task that returns a result |
| `RecursiveAction` | A task that does not return a result |

### Syntax Rules and Structure

```java
ForkJoinPool pool = new ForkJoinPool();

class MyTask extends RecursiveTask<Integer> {
    protected Integer compute() {
        if (smallEnough) {
            return computeDirectly();
        }
        MyTask left = new MyTask(leftHalf);
        MyTask right = new MyTask(rightHalf);
        left.fork();              // Submit left for async execution
        Integer rightResult = right.compute();  // Compute right directly
        Integer leftResult = left.join();       // Wait for left
        return combine(leftResult, rightResult);
    }
}

Integer result = pool.invoke(new MyTask(data));
```

### Annotated Code Example

```java
// File: ForkJoinPoolDemo.java
import java.util.concurrent.*;

public class ForkJoinPoolDemo {
    static class SumTask extends RecursiveTask<Long> {
        private static final int THRESHOLD = 10000;
        private final int[] array;
        private final int start, end;

        SumTask(int[] array, int start, int end) {
            this.array = array;
            this.start = start;
            this.end = end;
        }

        @Override
        protected Long compute() {
            if (end - start <= THRESHOLD) {
                long sum = 0;
                for (int i = start; i < end; i++) sum += array[i];
                return sum;
            }
            int mid = (start + end) / 2;
            SumTask left = new SumTask(array, start, mid);
            SumTask right = new SumTask(array, mid, end);
            left.fork();                    // Submit left asynchronously
            long rightResult = right.compute();  // Compute right directly
            long leftResult = left.join();       // Wait for left
            return leftResult + rightResult;
        }
    }

    public static void main(String[] args) {
        int[] array = new int[1_000_000];
        for (int i = 0; i < array.length; i++) array[i] = i + 1;

        ForkJoinPool pool = new ForkJoinPool();
        long sum = pool.invoke(new SumTask(array, 0, array.length));

        System.out.println("Sum: " + sum);  // 500000500000
        pool.shutdown();
    }
}
```

**Expected Output:**
```
Sum: 500000500000
```

**Step-by-Step Explanation:**

1. `SumTask` extends `RecursiveTask<Long>`.
2. If the segment is small enough (`THRESHOLD`), it computes the sum directly.
3. Otherwise, it splits the segment in half, forks the left half, computes the right half directly, and joins the left result.
4. The `ForkJoinPool` executes the task using work-stealing threads.

### Real-World Cases

- **Parallel Streams:** `list.parallelStream()` uses the common `ForkJoinPool`.
- **Recursive Algorithms:** Merge sort, quicksort, tree traversal.
- **Image Processing:** Parallel filtering and transformations.
- **Scientific Computing:** Monte Carlo simulations, matrix multiplication.

---

## 13. Executor Best Practices

### Definitions

**Core Definition:** Executor best practices are guidelines for using the Executor framework safely, efficiently, and maintainably.

**Technical Definition:** Best practices derive from authoritative sources including the SEI CERT Oracle Coding Standard for Java and community experience. Key principles include using bounded queues, implementing graceful shutdown, ensuring tasks are interruptible, and avoiding unbounded thread pools.

**Beginner-Friendly Explanation:** Best practices help you avoid common pitfalls like resource exhaustion, deadlocks, and memory leaks. They're like a safety manual for using thread pools correctly.

### Key Best Practices Table

| Practice | Description | Source |
|----------|-------------|--------|
| Use bounded queues | Prevent `OutOfMemoryError` from unbounded task accumulation | SEI CERT |
| Implement graceful shutdown | Call `shutdown()` then `awaitTermination()`; use `shutdownNow()` as fallback | Oracle |
| Ensure tasks are interruptible | Tasks must support interruption for `shutdownNow()` to work | SEI CERT |
| Prefer `ThreadPoolExecutor` over `Executors` for production | Factory methods use unbounded queues or unbounded threads | Alibaba |
| Set appropriate pool sizes | Match pool size to workload (CPU-bound vs I/O-bound) | Best practice |
| Use `CallerRunsPolicy` for backpressure | Prevents task loss and applies natural backpressure | Best practice |
| Name threads meaningfully | Use a custom `ThreadFactory` for debugging | Best practice |
| Use `try-with-resources` (Java 19+) | `ExecutorService` implements `AutoCloseable` | Java 19+ |
| Avoid blocking operations in pool threads | Blocking can starve the pool | Best practice |
| Monitor pool metrics | Track queue size, active threads, completed tasks | Best practice |
| Use virtual threads for I/O-bound tasks | `newVirtualThreadPerTaskExecutor()` for high-concurrency I/O | Java 21+ |
| Handle exceptions properly | Use `Future.get()` or an `UncaughtExceptionHandler` | Best practice |

### Annotated Code Example

```java
// File: ExecutorBestPractices.java
import java.util.concurrent.*;

public class ExecutorBestPractices {
    public static void main(String[] args) throws Exception {
        // BEST PRACTICE: Use ThreadPoolExecutor with bounded queue
        ThreadPoolExecutor executor = new ThreadPoolExecutor(
            4,                              // corePoolSize
            8,                              // maximumPoolSize
            60, TimeUnit.SECONDS,           // keepAliveTime
            new ArrayBlockingQueue<>(100),  // bounded queue
            new ThreadFactory() {
                private int count = 0;
                public Thread newThread(Runnable r) {
                    return new Thread(r, "AppWorker-" + (++count));
                }
            },
            new ThreadPoolExecutor.CallerRunsPolicy()  // backpressure
        );

        // Submit tasks
        for (int i = 1; i <= 50; i++) {
            final int taskId = i;
            executor.execute(() -> {
                System.out.println(Thread.currentThread().getName() +
                    " task " + taskId);
            });
        }

        // BEST PRACTICE: Graceful shutdown
        executor.shutdown();
        try {
            if (!executor.awaitTermination(10, TimeUnit.SECONDS)) {
                executor.shutdownNow();  // Force shutdown if timeout
                if (!executor.awaitTermination(5, TimeUnit.SECONDS)) {
                    System.err.println("Pool did not terminate");
                }
            }
        } catch (InterruptedException e) {
            executor.shutdownNow();
            Thread.currentThread().interrupt();
        }

        System.out.println("Shutdown complete");
    }
}
```

**Expected Output (interleaved):**
```
AppWorker-1 task 1
AppWorker-2 task 2
...
Shutdown complete
```

**Step-by-Step Explanation:**

1. A `ThreadPoolExecutor` is created with a bounded queue to prevent memory exhaustion.
2. `CallerRunsPolicy` provides backpressure when the queue is full.
3. A custom `ThreadFactory` names threads meaningfully.
4. Graceful shutdown: `shutdown()` then `awaitTermination()`, with `shutdownNow()` as a fallback.
5. `InterruptedException` is handled by restoring the interrupt status.

### Real-World Cases

- **Web Servers:** Bounded pools with graceful shutdown for request handling.
- **Microservices:** Thread pools for database queries, HTTP calls, and background jobs.
- **Batch Processing:** Custom thread pools for large-scale data processing.

---

## References Links

- Package java.util.concurrent — Java Platform SE 8 API Specification - https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/package-summary.html
- Interface Executor — Java Platform SE 8 API Specification - https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Executor.html
- Interface ExecutorService — Java Platform SE 8 API Specification - https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ExecutorService.html
- Class Executors — Java Platform SE 8 API Specification - https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Executors.html
- Class ThreadPoolExecutor — Java Platform SE 8 API Specification - https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ThreadPoolExecutor.html
- Class ScheduledThreadPoolExecutor — Java Platform SE 11 API Specification - https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/ScheduledThreadPoolExecutor.html
- Class ForkJoinPool — Java Platform SE 8 API Specification - https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ForkJoinPool.html
- Fork/Join — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/essential/concurrency/forkjoin.html
- CON36-J. Ensure that tasks submitted to a thread pool are interruptible — SEI CERT - https://wiki.sei.cmu.edu/confluence/display/java/CON36-J.+Ensure+that+tasks+submitted+to+a+thread+pool+are+interruptible
- Java Concurrency Best Practices — jabrena/cursor-rules-examples (GitHub) - https://github.com/jabrena/cursor-rules-examples/blob/main/.agents/skills/125-java-concurrency/references/125-java-concurrency.md
- 线程池初始化严禁使用Executors — Alibaba Cloud Developer Community - https://developer.aliyun.com/article/1624198