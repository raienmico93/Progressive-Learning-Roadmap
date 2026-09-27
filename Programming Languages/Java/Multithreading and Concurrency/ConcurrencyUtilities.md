# Concurrency Utilities & The Executor Framework: A Comprehensive Research Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**
The Java Executor Framework is a set of interfaces and classes in the `java.util.concurrent` package that standardize the invocation, scheduling, execution, and control of asynchronous tasks according to configurable execution policies. It decouples task submission from the mechanics of how each task will be run, including details of thread use, scheduling, and lifecycle management.

**Technical Definition**
The framework is built around the `Executor` interface, which provides a single method `execute(Runnable)`. The `ExecutorService` subinterface adds lifecycle management methods (`shutdown`, `shutdownNow`, `awaitTermination`) and the ability to submit `Callable` tasks that return `Future` objects. `ScheduledExecutorService` extends these capabilities with support for delayed and periodic execution. Concrete implementations include `ThreadPoolExecutor` (tunable thread pool), `ForkJoinPool` (work-stealing pool for divide-and-conquer algorithms), and the factory methods in `Executors`. The framework implements the Producer-Consumer design pattern: the application thread acts as producer, submitting tasks to a work queue consumed by worker threads.

**Beginner-Friendly Explanation**
Imagine a restaurant where customers (tasks) place orders at a counter. Instead of each customer hiring their own personal chef (creating a new `Thread` for every task), the restaurant has a team of chefs (a thread pool) who share the orders. The counter clerk (the `Executor`) takes orders and passes them to the kitchen. The customer doesn't need to know which chef cooks their dish or how many chefs are working. This is what the Executor Framework does: it manages a pool of worker threads and lets you focus on writing tasks, not on managing threads.

### Key Characteristics

- **Decoupling**: Task submission is separated from task execution policy, allowing the same task to run in different execution contexts (single thread, thread pool, scheduled executor, virtual thread per task).
- **Thread Reuse**: Thread pools reuse worker threads across multiple tasks, reducing the overhead of thread creation and destruction.
- **Resource Bounding**: The framework allows bounding the number of threads and the size of the work queue, preventing resource exhaustion.
- **Lifecycle Management**: `ExecutorService` provides methods to shut down gracefully (`shutdown()`), forcefully (`shutdownNow()`), and to await termination (`awaitTermination()`).
- **Asynchronous Results**: `Callable` tasks return `Future` objects that allow checking completion, retrieving results, and cancelling tasks.
- **Composable Pipelines**: `CompletableFuture` implements `Future` and `CompletionStage`, providing 40+ methods for chaining, combining, and composing asynchronous computations.
- **Structured Concurrency**: Java 19+ makes `ExecutorService` implement `AutoCloseable`, enabling deterministic cleanup via try-with-resources.

### Prerequisites

- Basic understanding of Java threads and the `Thread` class
- Familiarity with `Runnable` and `Callable` interfaces
- Knowledge of the Java Memory Model and happens-before relationships
- Awareness of blocking queues and producer-consumer patterns

### Related Programming Areas

- **Java Memory Model**: The framework's task submission and completion rely on happens-before guarantees.
- **`java.util.concurrent.atomic`**: Atomic variables complement the framework for lock-free state management.
- **Structured Concurrency**: Java 21+ `StructuredTaskScope` builds on the Executor Framework for scoped task management.
- **Reactive Programming**: `CompletableFuture` serves as a bridge between imperative and reactive styles.

### Core Concepts / Features

Six core concepts are covered: (1) the Executor abstraction, (2) ExecutorService and ScheduledExecutorService lifecycles, (3) core thread pools (ThreadPoolExecutor and ForkJoinPool), (4) Future handlers, (5) CompletableFuture chaining, and (6) AutoCloseable executor scopes.

---

## Core Concept 1: Decoupling Task Submission from Thread Execution — The Executor Abstraction

### Definitions

**Core Definition**
The `Executor` interface is the root abstraction of the framework. It represents an object that executes submitted `Runnable` tasks, providing a way to decouple task submission from the mechanics of how each task will be run.

**Technical Definition**
Per the Java API specification, `Executor` provides a way of decoupling task submission from the mechanics of how each task will be run, including details of thread use, scheduling, etc. An `Executor` is normally used instead of explicitly creating threads. In the simplest case, an executor can run the submitted task immediately in the caller's thread (`class DirectExecutor implements Executor { public void execute(Runnable r) { r.run(); } }`). More generally, tasks are executed in some thread other than the caller's thread. The memory consistency effect is that actions in a thread prior to submitting a `Runnable` object to an `Executor` happen-before its execution begins, perhaps in another thread.

**Beginner-Friendly Explanation**
`Executor` is like a delivery service. You hand over a package (a `Runnable` task) to the service (the `Executor`), and the service decides how and when to deliver it. You don't need to know whether the delivery is done by bicycle, car, or truck (whether it runs on a new thread, a pooled thread, or the caller's thread). You just know it will get done.

### Purposes

- To decouple task submission from execution policy, enabling flexible and reusable task definitions.
- To replace the direct creation of threads with a higher-level abstraction that centralizes thread management.
- To enable the same task to be executed by different execution strategies (direct, pooled, scheduled, virtual thread) without changing the task code.
- To provide a uniform interface for executing tasks across diverse concurrency implementations.

### Syntax Rules and Structure

**Complete General Syntax**

```java
public interface Executor {
    void execute(Runnable command);
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `execute(Runnable command)` | The sole method; submits a task for execution |
| `Runnable command` | The task to be executed; contains a `run()` method |

**Syntax Rules**

1. `Executor.execute()` returns `void`; it provides no handle for tracking task completion or retrieving results.
2. The `Executor` interface does not strictly require asynchronous execution; an executor may run the task in the caller's thread.
3. A common convention is `Executor executor = anExecutor(); executor.execute(new RunnableTask1());`.
4. The `Executor` interface does not provide lifecycle management methods; these are added by `ExecutorService`.

**Constraints and Limitations**

- Cannot shut down an `Executor` or interrupt/cancel running tasks.
- Cannot handle runtime configuration changes gracefully.
- Cannot retrieve results from tasks or check their completion status.
- Cannot schedule tasks for future or periodic execution.

### Annotated Code Examples

**Example 1: DirectExecutor (Simplest Implementation)**

```java
import java.util.concurrent.Executor;

public class DirectExecutorDemo {
    // DirectExecutor runs the task in the caller's thread
    static class DirectExecutor implements Executor {
        @Override
        public void execute(Runnable r) {
            r.run(); // no new thread: runs immediately in caller's thread
        }
    }

    public static void main(String[] args) {
        Executor executor = new DirectExecutor();
        System.out.println("Before execute");
        executor.execute(() -> System.out.println("Task running in: "
            + Thread.currentThread().getName()));
        System.out.println("After execute");
    }
}
```

**Expected Output**

```
Before execute
Task running in: main
After execute
```

**Why This Output Occurs**

`DirectExecutor` implements `execute()` by calling `r.run()` directly. No new thread is created. The task executes synchronously in the `main` thread. "Before execute" prints first, then the task prints its thread name (`main`), then "After execute" prints. This demonstrates that the `Executor` interface does not mandate asynchronous execution.

---

**Example 2: ThreadPerTaskExecutor**

```java
import java.util.concurrent.Executor;

public class ThreadPerTaskExecutorDemo {
    // Each task gets its own new thread
    static class ThreadPerTaskExecutor implements Executor {
        @Override
        public void execute(Runnable r) {
            new Thread(r).start(); // creates a new thread per task
        }
    }

    public static void main(String[] args) {
        Executor executor = new ThreadPerTaskExecutor();
        System.out.println("Submitting task 1");
        executor.execute(() -> System.out.println("Task 1 in: "
            + Thread.currentThread().getName()));
        System.out.println("Submitting task 2");
        executor.execute(() -> System.out.println("Task 2 in: "
            + Thread.currentThread().getName()));
    }
}
```

**Expected Output (order may vary)**

```
Submitting task 1
Submitting task 2
Task 1 in: Thread-0
Task 2 in: Thread-1
```

**Why This Output Occurs**

`ThreadPerTaskExecutor` creates a new `Thread` for each task and starts it. The main thread submits both tasks quickly and exits. The two worker threads run concurrently, each printing its own name. The submission order is preserved in the main thread's output, but task execution order is not guaranteed.

---

**Example 3: Executor with a Real Thread Pool**

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class PooledExecutorDemo {
    public static void main(String[] args) {
        // Create a fixed pool of 2 threads
        ExecutorService executor = Executors.newFixedThreadPool(2);

        for (int i = 1; i <= 5; i++) {
            final int taskId = i;
            executor.execute(() -> {
                System.out.println("Task " + taskId + " running in "
                    + Thread.currentThread().getName());
                try { Thread.sleep(100); } catch (InterruptedException e) { }
            });
        }

        executor.shutdown(); // no more tasks accepted
    }
}
```

**Expected Output (order may vary)**

```
Task 1 running in pool-1-thread-1
Task 2 running in pool-1-thread-2
Task 3 running in pool-1-thread-1
Task 4 running in pool-1-thread-2
Task 5 running in pool-1-thread-1
```

**Why This Output Occurs**

The fixed thread pool contains two worker threads. Tasks 1 and 2 are immediately assigned to both threads. Tasks 3, 4, and 5 wait in the work queue. When a thread finishes a task (after the 100 ms sleep), it picks up the next task from the queue. The thread names (`pool-1-thread-1`, `pool-1-thread-2`) confirm that only two threads are reused across five tasks.

### Real-World Cases

- **Web Servers**: A servlet container uses an `Executor` to process incoming HTTP requests without creating a new thread per request.
- **Task Queues**: Background job processors submit tasks to an `Executor` that runs them on a bounded pool.
- **Parallel Data Processing**: A batch processor submits chunks of data as `Runnable` tasks to an `Executor` for concurrent processing.

---

## Core Concept 2: Managing Task Lifecycles and Production Pipelines — ExecutorService and ScheduledExecutorService

### Definitions

**Core Definition**
`ExecutorService` is a subinterface of `Executor` that adds lifecycle management for both individual tasks and the executor itself. `ScheduledExecutorService` extends `ExecutorService` to support delayed and periodic task execution.

**Technical Definition**
`ExecutorService` provides methods to manage termination (`shutdown()`, `shutdownNow()`, `awaitTermination()`) and methods that produce a `Future` for tracking progress of one or more asynchronous tasks (`submit()`). `ScheduledExecutorService` adds `schedule(Runnable, long, TimeUnit)`, `schedule(Callable<V>, long, TimeUnit)`, `scheduleAtFixedRate(Runnable, long, long, TimeUnit)`, and `scheduleWithFixedDelay(Runnable, long, long, TimeUnit)`. The lifecycle of an executor service has three states: RUNNING (accepts new tasks), SHUTDOWN (no new tasks accepted, existing tasks complete), and TERMINATED (all tasks complete, executor fully stopped).

**Beginner-Friendly Explanation**
`ExecutorService` is like a full-service restaurant manager. Not only can they assign orders to chefs (execute tasks), but they can also close the kitchen (shutdown), tell all chefs to stop immediately (shutdownNow), and wait until everyone has left before locking the doors (awaitTermination). `ScheduledExecutorService` is like a manager who can also schedule orders for a specific time or have them repeat regularly, like a bakery that bakes fresh bread every morning at 6 AM.

### Purposes

- To manage the lifecycle of an executor, including graceful shutdown and forced termination.
- To submit tasks that return results via `Future`, enabling asynchronous computation with result retrieval.
- To schedule tasks for future execution or periodic repetition (e.g., heartbeats, polling, cleanup jobs).
- To await termination of all tasks before proceeding with program shutdown.

### Syntax Rules and Structure

**Complete General Syntax**

```java
// ExecutorService lifecycle
public interface ExecutorService extends Executor, AutoCloseable {
    void shutdown();
    List<Runnable> shutdownNow();
    boolean isShutdown();
    boolean isTerminated();
    boolean awaitTermination(long timeout, TimeUnit unit)
        throws InterruptedException;
    <T> Future<T> submit(Callable<T> task);
    <T> Future<T> submit(Runnable task, T result);
    Future<?> submit(Runnable task);
}

// ScheduledExecutorService
public interface ScheduledExecutorService extends ExecutorService {
    ScheduledFuture<?> schedule(Runnable command, long delay, TimeUnit unit);
    <V> ScheduledFuture<V> schedule(Callable<V> callable, long delay, TimeUnit unit);
    ScheduledFuture<?> scheduleAtFixedRate(Runnable command,
        long initialDelay, long period, TimeUnit unit);
    ScheduledFuture<?> scheduleWithFixedDelay(Runnable command,
        long initialDelay, long delay, TimeUnit unit);
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `shutdown()` | Initiates orderly shutdown; previously submitted tasks execute, no new tasks accepted |
| `shutdownNow()` | Attempts to stop all actively executing tasks; returns list of never-commenced tasks |
| `awaitTermination(timeout, unit)` | Blocks until all tasks complete, timeout expires, or current thread interrupted |
| `submit(Callable)` | Submits a value-returning task and returns a `Future` |
| `schedule(...)` | Executes a task after a specified delay |
| `scheduleAtFixedRate(...)` | Repeats a task at a fixed rate (period between start times) |
| `scheduleWithFixedDelay(...)` | Repeats a task with a fixed delay (delay between end and next start) |

**Syntax Rules**

1. `shutdown()` allows previously submitted tasks to execute but rejects new tasks with `RejectedExecutionException`.
2. `shutdownNow()` attempts to stop running tasks via interruption and returns tasks that never began execution.
3. `awaitTermination()` must be called after `shutdown()` or `shutdownNow()`; it blocks until termination or timeout.
4. `scheduleAtFixedRate` executes at a fixed rate regardless of task duration (if task takes longer than period, next execution starts immediately after).
5. `scheduleWithFixedDelay` executes with a fixed delay between the end of one execution and the start of the next.
6. If a periodic task throws an exception, subsequent executions are suppressed.

**Constraints and Limitations**

- `shutdownNow()` does not guarantee that running tasks will stop; it only sends interrupts.
- Periodic tasks that throw unchecked exceptions are cancelled and will not run again.
- `awaitTermination()` without a timeout can block indefinitely if tasks never complete.
- Before Java 19, `ExecutorService` did not implement `AutoCloseable`; try-with-resources required manual wrapping.

### Annotated Code Examples

**Example 1: ExecutorService Lifecycle**

```java
import java.util.concurrent.*;

public class LifecycleDemo {
    public static void main(String[] args) throws InterruptedException {
        ExecutorService executor = Executors.newFixedThreadPool(2);

        // Submit tasks
        for (int i = 1; i <= 3; i++) {
            final int id = i;
            executor.submit(() -> {
                System.out.println("Task " + id + " started");
                try { Thread.sleep(500); } catch (InterruptedException e) {
                    System.out.println("Task " + id + " interrupted");
                }
                System.out.println("Task " + id + " finished");
            });
        }

        System.out.println("isShutdown: " + executor.isShutdown()); // false
        executor.shutdown(); // orderly shutdown
        System.out.println("isShutdown after shutdown(): " + executor.isShutdown()); // true
        System.out.println("isTerminated: " + executor.isTerminated()); // false

        // Wait for all tasks to complete
        boolean terminated = executor.awaitTermination(5, TimeUnit.SECONDS);
        System.out.println("Terminated within 5s: " + terminated); // true
        System.out.println("isTerminated: " + executor.isTerminated()); // true
    }
}
```

**Expected Output**

```
isShutdown: false
isShutdown after shutdown(): true
isTerminated: false
Task 1 started
Task 2 started
Task 1 finished
Task 2 finished
Task 3 started
Task 3 finished
Terminated within 5s: true
isTerminated: true
```

**Why This Output Occurs**

`shutdown()` transitions the executor to SHUTDOWN state (no new tasks accepted) but does not wait for existing tasks. `isTerminated()` returns `false` immediately after shutdown because tasks are still running. `awaitTermination(5, TimeUnit.SECONDS)` blocks until all three tasks complete (approximately 1.5 seconds total), then returns `true`. The executor is now TERMINATED.

---

**Example 2: ScheduledExecutorService — Fixed Rate vs. Fixed Delay**

```java
import java.util.concurrent.*;

public class ScheduledDemo {
    public static void main(String[] args) throws InterruptedException {
        ScheduledExecutorService scheduler =
            Executors.newScheduledThreadPool(1);

        // Fixed rate: every 1 second, regardless of task duration
        System.out.println("--- Fixed Rate ---");
        ScheduledFuture<?> fixedRate = scheduler.scheduleAtFixedRate(
            () -> System.out.println("FixedRate: " + System.currentTimeMillis() % 10000),
            0, 1, TimeUnit.SECONDS);

        // Let it run for 3 seconds
        Thread.sleep(3000);
        fixedRate.cancel(false);

        // Fixed delay: 1 second between end of one and start of next
        System.out.println("--- Fixed Delay ---");
        ScheduledFuture<?> fixedDelay = scheduler.scheduleWithFixedDelay(
            () -> {
                System.out.println("FixedDelay: " + System.currentTimeMillis() % 10000);
                try { Thread.sleep(500); } catch (InterruptedException e) { }
            },
            0, 1, TimeUnit.SECONDS);

        Thread.sleep(4000);
        fixedDelay.cancel(false);
        scheduler.shutdown();
    }
}
```

**Expected Output (timestamps vary)**

```
--- Fixed Rate ---
FixedRate: 1234
FixedRate: 2234
FixedRate: 3234
--- Fixed Delay ---
FixedDelay: 4234
FixedDelay: 5734  (500ms task + 1000ms delay = 1500ms later)
FixedDelay: 7234
```

**Why This Output Occurs**

`scheduleAtFixedRate` executes at fixed intervals regardless of task duration. The task runs every 1 second. `scheduleWithFixedDelay` waits 1 second *after* the task finishes. Since the task sleeps 500 ms, the next execution starts 1500 ms after the previous start (500 ms work + 1000 ms delay).

### Real-World Cases

- **Cache Refresh**: A `ScheduledExecutorService` refreshes a cache every 5 minutes using `scheduleAtFixedRate`.
- **Heartbeat Monitoring**: A scheduled task pings a server every 30 seconds with `scheduleWithFixedDelay` to avoid overlap if a ping takes long.
- **Nightly Cleanup**: A database cleanup job runs at 2 AM daily using `schedule`.
- **Graceful Server Shutdown**: A web server calls `shutdown()` then `awaitTermination()` to finish processing in-flight requests before exiting.

---

## Core Concept 3: Core Thread Pools — Tuning ThreadPoolExecutor, Sizing ForkJoinPool, and Work-Stealing

### Definitions

**Core Definition**
`ThreadPoolExecutor` is a tunable thread pool implementation that manages a pool of worker threads and a work queue. `ForkJoinPool` is a specialized `ExecutorService` that employs a work-stealing algorithm for efficient execution of divide-and-conquer tasks.

**Technical Definition**
`ThreadPoolExecutor` automatically adjusts pool size according to bounds set by `corePoolSize` and `maximumPoolSize`. When a new task is submitted and fewer than `corePoolSize` threads are running, a new thread is created to handle the request, even if other worker threads are idle. If more than `corePoolSize` but fewer than `maximumPoolSize` threads are running, a new thread is created only if the queue is full. `ForkJoinPool` differs from other `ExecutorService` implementations mainly by employing work-stealing: all threads in the pool attempt to find and execute tasks submitted to the pool and/or created by other active tasks. Worker threads maintain thread-local deques, pushing tasks LIFO and stealing from other deques FIFO.

**Beginner-Friendly Explanation**
`ThreadPoolExecutor` is like a restaurant with a fixed number of permanent chefs (`corePoolSize`) and the ability to hire temporary chefs (`maximumPoolSize`) when the order queue gets too long. `ForkJoinPool` is like a team of chefs where each chef works on their own pile of orders, but if one chef finishes early, they steal orders from the bottom of another chef's pile (the oldest orders) so everyone stays busy.

### Purposes

- To tune thread pool behavior to match workload characteristics (CPU-bound vs. I/O-bound tasks).
- To bound resource consumption by limiting the maximum number of threads and the queue capacity.
- To maximize CPU utilization through work-stealing for divide-and-conquer algorithms.
- To reduce contention between threads by using thread-local deques in ForkJoinPool.

### Syntax Rules and Structure

**Complete General Syntax — ThreadPoolExecutor**

```java
public ThreadPoolExecutor(
    int corePoolSize,        // number of threads to keep in pool
    int maximumPoolSize,     // maximum number of threads
    long keepAliveTime,      // max idle time for excess threads
    TimeUnit unit,           // time unit for keepAliveTime
    BlockingQueue<Runnable> workQueue,  // queue for pending tasks
    ThreadFactory threadFactory,         // factory for creating threads
    RejectedExecutionHandler handler     // policy for rejected tasks
)
```

**Complete General Syntax — ForkJoinPool**

```java
public ForkJoinPool()  // default parallelism = available processors
public ForkJoinPool(int parallelism)
public ForkJoinPool(int parallelism, ForkJoinWorkerThreadFactory factory,
    UncaughtExceptionHandler handler, boolean asyncMode)
```

**Component Breakdown**

| Parameter | Description |
|-----------|-------------|
| `corePoolSize` | Number of threads kept in the pool even when idle |
| `maximumPoolSize` | Maximum number of threads the pool can create |
| `keepAliveTime` | Idle time before excess threads (> core) are terminated |
| `workQueue` | Blocking queue holding tasks before execution |
| `threadFactory` | Factory for creating new threads (default: `Executors.defaultThreadFactory()`) |
| `handler` | Policy for tasks that cannot be accepted (default: `AbortPolicy`) |
| `parallelism` | Target parallelism level in ForkJoinPool (default: CPU count) |
| `asyncMode` | If true, uses FIFO scheduling for never-joined event-style tasks |

**Syntax Rules**

1. `corePoolSize` and `maximumPoolSize` can be the same for a fixed-size pool; `maximumPoolSize` can be `Integer.MAX_VALUE` for unbounded pools.
2. New threads are created only when a new task arrives and fewer than `corePoolSize` threads are running.
3. If `corePoolSize` threads are running and the queue is full, new threads are created up to `maximumPoolSize`.
4. If the pool is at `maximumPoolSize` and the queue is full, the `RejectedExecutionHandler` is invoked.
5. `keepAliveTime` applies only to threads exceeding `corePoolSize` unless `allowCoreThreadTimeOut(true)` is set.
6. In `ForkJoinPool`, worker threads process their own deques LIFO (youngest-first) and steal from other deques FIFO (oldest-first).
7. `ForkJoinPool.commonPool()` provides a static common pool suitable for most applications.

**Constraints and Limitations**

- Thread pool sizing is workload-dependent; no single formula works for all scenarios.
- CPU-bound tasks benefit from `corePoolSize` ≈ number of processors; I/O-bound tasks may need higher values.
- A work queue that is too small may cause task rejection; too large may delay detection of overload.
- `ForkJoinPool` is optimized for divide-and-conquer; it is not ideal for blocking I/O operations.
- `ForkJoinPool` worker threads are daemon threads by default.

### Annotated Code Examples

**Example 1: Tuning ThreadPoolExecutor**

```java
import java.util.concurrent.*;

public class ThreadPoolTuningDemo {
    public static void main(String[] args) throws InterruptedException {
        // Custom thread pool: 2 core threads, max 4, 10-second idle keepalive
        ThreadPoolExecutor executor = new ThreadPoolExecutor(
            2,                          // corePoolSize
            4,                          // maximumPoolSize
            10, TimeUnit.SECONDS,       // keepAliveTime
            new LinkedBlockingQueue<>(2), // workQueue capacity = 2
            Executors.defaultThreadFactory(),
            new ThreadPoolExecutor.AbortPolicy() // reject when full
        );

        // Submit 8 tasks: 2 core + 2 queued + 2 extra = 6 accepted, 2 rejected
        for (int i = 1; i <= 8; i++) {
            final int id = i;
            try {
                executor.execute(() -> {
                    System.out.println("Task " + id + " in "
                        + Thread.currentThread().getName());
                    try { Thread.sleep(2000); } catch (InterruptedException e) { }
                });
            } catch (RejectedExecutionException e) {
                System.out.println("Task " + id + " rejected");
            }
        }

        executor.shutdown();
        executor.awaitTermination(10, TimeUnit.SECONDS);
    }
}
```

**Expected Output (order may vary)**

```
Task 1 in pool-1-thread-1
Task 2 in pool-1-thread-2
Task 3 in pool-1-thread-3
Task 4 in pool-1-thread-4
Task 7 rejected
Task 8 rejected
Task 5 in pool-1-thread-1
Task 6 in pool-1-thread-2
```

**Why This Output Occurs**

With `corePoolSize = 2`, two threads are created immediately for tasks 1 and 2. The queue (capacity 2) holds tasks 3 and 4. When tasks 5 and 6 are submitted and the queue is full, the pool creates two more threads (up to `maximumPoolSize = 4`) to handle them. Tasks 7 and 8 are rejected because the queue is full and the pool is at maximum size. When tasks complete, worker threads pick up queued tasks 3 and 4.

---

**Example 2: ForkJoinPool Work-Stealing**

```java
import java.util.concurrent.*;

public class ForkJoinDemo {
    // Recursive task that sums an array
    static class SumTask extends RecursiveTask<Long> {
        private static final int THRESHOLD = 10_000;
        private final int[] array;
        private final int start, end;

        SumTask(int[] array, int start, int end) {
            this.array = array; this.start = start; this.end = end;
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
            left.fork();         // push to own deque (LIFO)
            long rightResult = right.compute(); // process directly
            long leftResult = left.join();      // wait for stolen task
            return leftResult + rightResult;
        }
    }

    public static void main(String[] args) {
        int[] data = new int[1_000_000];
        for (int i = 0; i < data.length; i++) data[i] = i;

        ForkJoinPool pool = new ForkJoinPool();
        long sum = pool.invoke(new SumTask(data, 0, data.length));
        System.out.println("Sum: " + sum); // sum 0..999999 = 499999500000

        System.out.println("Steal count: " + pool.getStealCount());
        pool.shutdown();
    }
}
```

**Expected Output**

```
Sum: 499999500000
Steal count: (some positive number, varies)
```

**Why This Output Occurs**

`SumTask` recursively splits the array until the segment size is ≤ 10,000 elements. The left subtask is forked (pushed to the worker's own deque), and the right subtask is computed directly. When the right subtask completes, the worker tries to join the left subtask. If the left subtask was stolen by another worker, the joining worker blocks. `getStealCount()` returns the number of successful steal operations, which is non-zero because idle workers steal the forked left subtasks.

### Real-World Cases

- **Web Crawler**: A `ForkJoinPool` splits a URL frontier into chunks and recursively processes links.
- **Image Processing**: A `ForkJoinPool` divides a large image into tiles for parallel filtering.
- **Parallel Sort**: `Arrays.parallelSort()` uses `ForkJoinPool.commonPool()` for divide-and-conquer sorting.
- **Database Connection Pool**: A `ThreadPoolExecutor` manages database connections with a bounded pool and a queue.

---

## Core Concept 4: Managing Async Computational Outcomes — Future<V> Handlers

### Definitions

**Core Definition**
`Future<V>` represents the result of an asynchronous computation, providing methods to check completion, wait for the result, retrieve the result, and cancel the task.

**Technical Definition**
`Future<V>` provides `get()` (blocks until ready, throws `ExecutionException` if the task failed), `get(long timeout, TimeUnit unit)` (blocks with timeout), `isDone()` (returns true if complete in any way), `isCancelled()` (returns true if cancelled before completion), and `cancel(boolean mayInterruptIfRunning)` (attempts to cancel). The `Future` interface is implemented by `FutureTask` and returned by `ExecutorService.submit()`. A `Future` cannot be manually completed; for that, `CompletableFuture` is required.

**Beginner-Friendly Explanation**
`Future` is like a restaurant pager. When you order (submit a task), you get a pager (a `Future`). You can check if your order is ready (`isDone()`), wait for it and pick it up (`get()`), or cancel your order before it's ready (`cancel()`). But once you pick it up, you can't put it back.

### Purposes

- To check the completion status of an asynchronous task without blocking.
- To retrieve the result of a `Callable` task after it completes.
- To wait for a task to complete with an optional timeout.
- To cancel a task that is no longer needed.

### Syntax Rules and Structure

**Complete General Syntax**

```java
public interface Future<V> {
    boolean cancel(boolean mayInterruptIfRunning);
    boolean isCancelled();
    boolean isDone();
    V get() throws InterruptedException, ExecutionException;
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

**Component Breakdown**

| Method | Description |
|--------|-------------|
| `cancel(boolean)` | Attempts to cancel; returns true if successful |
| `isCancelled()` | Returns true if cancelled before normal completion |
| `isDone()` | Returns true if complete (normally, exceptionally, or cancelled) |
| `get()` | Blocks until complete; returns result or throws exception |
| `get(timeout, unit)` | Blocks at most timeout; throws `TimeoutException` if not ready |

**Syntax Rules**

1. `get()` throws `ExecutionException` if the task threw an exception; the cause is accessible via `getCause()`.
2. `get(timeout, unit)` throws `TimeoutException` if the result is not available within the timeout.
3. `cancel(true)` attempts to interrupt the running task; `cancel(false)` cancels only if not yet started.
4. `isDone()` returns true for normal completion, exceptional completion, and cancellation.
5. A `Future`'s result can be retrieved multiple times after completion; `get()` returns immediately on subsequent calls.

**Constraints and Limitations**

- `Future` cannot be manually completed; only the task can complete it.
- `get()` blocks the calling thread; using it prematurely can reduce concurrency benefits.
- `Future` cannot be chained or composed; `CompletableFuture` addresses this limitation.
- `cancel(true)` does not guarantee cancellation; it sends an interrupt.

### Annotated Code Examples

**Example 1: Future with Timeout**

```java
import java.util.concurrent.*;

public class FutureTimeoutDemo {
    public static void main(String[] args) {
        ExecutorService executor = Executors.newSingleThreadExecutor();

        Callable<String> slowTask = () -> {
            Thread.sleep(5000); // 5 seconds
            return "Result";
        };

        Future<String> future = executor.submit(slowTask);

        try {
            String result = future.get(2, TimeUnit.SECONDS); // 2-second timeout
            System.out.println("Result: " + result);
        } catch (TimeoutException e) {
            System.out.println("Task timed out after 2 seconds");
            future.cancel(true); // interrupt the task
        } catch (InterruptedException | ExecutionException e) {
            e.printStackTrace();
        } finally {
            executor.shutdown();
        }
    }
}
```

**Expected Output**

```
Task timed out after 2 seconds
```

**Why This Output Occurs**

The task sleeps for 5 seconds, but `get(2, TimeUnit.SECONDS)` waits only 2 seconds. The `TimeoutException` is caught, and the task is cancelled with `cancel(true)`, which interrupts the sleeping thread.

---

**Example 2: Checking Status Without Blocking**

```java
import java.util.concurrent.*;

public class FutureStatusDemo {
    public static void main(String[] args) throws InterruptedException {
        ExecutorService executor = Executors.newSingleThreadExecutor();

        Future<Integer> future = executor.submit(() -> {
            Thread.sleep(2000);
            return 42;
        });

        System.out.println("isDone: " + future.isDone()); // false
        System.out.println("isCancelled: " + future.isCancelled()); // false

        Thread.sleep(2500); // wait for task to complete

        System.out.println("isDone: " + future.isDone()); // true
        System.out.println("Result: " + future.get()); // 42 (returns immediately)

        executor.shutdown();
    }
}
```

**Expected Output**

```
isDone: false
isCancelled: false
isDone: true
Result: 42
```

**Why This Output Occurs**

`isDone()` returns `false` immediately after submission because the task is still sleeping. After 2500 ms, the task has completed, so `isDone()` returns `true`. `get()` then returns the cached result `42` immediately without blocking.

### Real-World Cases

- **Parallel API Calls**: Multiple `Future` objects track concurrent HTTP requests; `get()` collects responses.
- **Batch Processing**: A batch job submits chunks as `Callable` tasks and collects results via `Future.get()`.
- **Timeout Enforcement**: A framework uses `Future.get(timeout)` to enforce deadlines on external calls.

---

## Core Concept 5: Asynchronous Functional Programming — CompletableFuture<T> Chaining and Composition

### Definitions

**Core Definition**
`CompletableFuture<T>` is a class that implements both `Future<T>` and `CompletionStage<T>`, providing a rich API for chaining, combining, and composing asynchronous computations in a functional style.

**Technical Definition**
`CompletableFuture` provides over 40 methods for asynchronous composition. Key methods include `thenApply` (transform result, like `map`), `thenCompose` (chain dependent async task, like `flatMap`), `thenCombine` (combine two independent futures), `thenAccept` (consume result), `thenRun` (run action after completion), `exceptionally` (recover from exception), `handle` (transform result or exception), and `allOf`/`anyOf` (combine multiple futures). Completion stages are executed in the thread that completes the previous stage unless an `Async` variant is used with an explicit `Executor`.

**Beginner-Friendly Explanation**
`CompletableFuture` is like a relay race baton that can be passed through a series of runners. Each runner (`thenApply`) transforms the baton, or starts a new race (`thenCompose`), or waits for another baton to arrive so they can be combined (`thenCombine`). You can also set up a backup plan if a runner drops the baton (`exceptionally`).

### Purposes

- To chain asynchronous computations where each stage depends on the result of the previous stage.
- To combine the results of multiple independent asynchronous computations.
- To handle exceptions and provide fallback values in asynchronous pipelines.
- To compose multiple `CompletableFuture` instances into a single result (e.g., `allOf`, `anyOf`).
- To provide a non-blocking alternative to `Future.get()` for building reactive-style pipelines.

### Syntax Rules and Structure

**Complete General Syntax**

```java
// Create
CompletableFuture<String> cf = CompletableFuture.supplyAsync(() -> "Hello");

// Transform (map)
CompletableFuture<Integer> length = cf.thenApply(String::length);

// Chain (flatMap)
CompletableFuture<String> chained = cf.thenCompose(s ->
    CompletableFuture.supplyAsync(() -> s + " World"));

// Combine
CompletableFuture<String> combined = cf.thenCombine(
    CompletableFuture.supplyAsync(() -> "!!!"),
    (a, b) -> a + b);

// Exception handling
CompletableFuture<String> recovered = cf.exceptionally(ex -> "Fallback");

// Wait for all
CompletableFuture.allOf(cf1, cf2, cf3).join();
```

**Component Breakdown**

| Method | Description | Analogy |
|--------|-------------|---------|
| `supplyAsync(Supplier)` | Starts an async task returning a value | Start a race |
| `thenApply(Function)` | Transforms the result synchronously | `map` |
| `thenCompose(Function)` | Chains a dependent async task | `flatMap` |
| `thenCombine(other, BiFunction)` | Combines two independent futures | Merge two streams |
| `thenAccept(Consumer)` | Consumes the result | `forEach` |
| `thenRun(Runnable)` | Runs after completion | `onComplete` |
| `exceptionally(Function)` | Recovers from exception | `catch` |
| `handle(BiFunction)` | Transforms result or exception | `map` + `catch` |
| `allOf(cfs...)` | Completes when all complete | `Promise.all` |
| `anyOf(cfs...)` | Completes when any completes | `Promise.race` |

**Syntax Rules**

1. `thenApply` transforms the result; `thenCompose` flattens a nested `CompletableFuture`.
2. Without the `Async` suffix, the function runs in the thread that completed the previous stage (or the caller's thread if already complete).
3. With `Async`, the function runs in a default executor (`ForkJoinPool.commonPool()`) or a specified executor.
4. `allOf` returns `CompletableFuture<Void>`; you must manually retrieve results from individual futures.
5. `exceptionally` provides a fallback value; `handle` can transform both success and failure.
6. `join()` is like `get()` but throws unchecked `CompletionException`.

**Constraints and Limitations**

- Debugging chained `CompletableFuture` can be difficult due to stack traces spanning multiple threads.
- Without `Async`, the executing thread is unpredictable.
- `allOf` does not propagate results; individual `get()` calls are needed.
- Nested `CompletableFuture` (e.g., `CompletableFuture<CompletableFuture<T>>`) is a common mistake; use `thenCompose` to flatten.

### Annotated Code Examples

**Example 1: Basic Chaining**

```java
import java.util.concurrent.*;

public class ChainDemo {
    public static void main(String[] args) throws Exception {
        CompletableFuture<String> result = CompletableFuture
            .supplyAsync(() -> {
                System.out.println("Stage 1 in: " + Thread.currentThread().getName());
                return "Hello";
            })
            .thenApply(s -> {
                System.out.println("Stage 2 in: " + Thread.currentThread().getName());
                return s + " World";
            })
            .thenApply(String::toUpperCase);

        System.out.println("Result: " + result.get()); // HELLO WORLD
    }
}
```

**Expected Output**

```
Stage 1 in: ForkJoinPool.commonPool-worker-1
Stage 2 in: ForkJoinPool.commonPool-worker-1
Result: HELLO WORLD
```

**Why This Output Occurs**

`supplyAsync` starts the first stage in the common `ForkJoinPool`. `thenApply` chains a transformation; since the previous stage completed in the same thread, `thenApply` runs in the same worker thread (no `Async` suffix). The final `get()` blocks until the chain completes.

---

**Example 2: thenCompose vs. thenApply**

```java
import java.util.concurrent.*;

public class ComposeVsApplyDemo {
    static CompletableFuture<String> fetchUser() {
        return CompletableFuture.supplyAsync(() -> "Alice");
    }

    static CompletableFuture<String> fetchOrder(String user) {
        return CompletableFuture.supplyAsync(() -> "Order for " + user);
    }

    public static void main(String[] args) throws Exception {
        // WRONG: thenApply returns nested CompletableFuture
        CompletableFuture<CompletableFuture<String>> nested = fetchUser()
            .thenApply(user -> fetchOrder(user)); // nested future!

        // CORRECT: thenCompose flattens
        CompletableFuture<String> flat = fetchUser()
            .thenCompose(user -> fetchOrder(user)); // flattened

        System.out.println("Flat result: " + flat.get()); // Order for Alice
    }
}
```

**Expected Output**

```
Flat result: Order for Alice
```

**Why This Output Occurs**

`thenApply` with a function that returns a `CompletableFuture` produces a nested structure (`CompletableFuture<CompletableFuture<String>>`). `thenCompose` is designed for this: it takes a function that returns a `CompletionStage` and flattens the result, producing `CompletableFuture<String>` directly.

---

**Example 3: Combining with thenCombine and allOf**

```java
import java.util.concurrent.*;

public class CombineDemo {
    public static void main(String[] args) throws Exception {
        CompletableFuture<String> user = CompletableFuture
            .supplyAsync(() -> "Alice");
        CompletableFuture<Integer> age = CompletableFuture
            .supplyAsync(() -> 30);

        // Combine two independent futures
        CompletableFuture<String> profile = user.thenCombine(age,
            (u, a) -> u + " is " + a + " years old");
        System.out.println(profile.get()); // Alice is 30 years old

        // Wait for all to complete
        CompletableFuture<Void> all = CompletableFuture.allOf(
            CompletableFuture.supplyAsync(() -> { sleep(500); return "A"; }),
            CompletableFuture.supplyAsync(() -> { sleep(300); return "B"; }),
            CompletableFuture.supplyAsync(() -> { sleep(100); return "C"; })
        );
        all.join();
        System.out.println("All done");
    }

    static void sleep(long ms) {
        try { Thread.sleep(ms); } catch (InterruptedException e) { }
    }
}
```

**Expected Output**

```
Alice is 30 years old
All done
```

**Why This Output Occurs**

`thenCombine` waits for both `user` and `age` to complete, then applies the combining function. `allOf` returns a `CompletableFuture<Void>` that completes when all three provided futures complete, regardless of their individual results. `join()` blocks until `allOf` completes.

### Real-World Cases

- **Microservice Orchestration**: A `CompletableFuture` chain calls multiple services, combines results, and handles failures.
- **E-commerce Checkout**: `thenCombine` joins user profile and cart data; `thenCompose` chains payment processing.
- **Data Enrichment Pipeline**: `supplyAsync` fetches raw data, `thenApply` transforms it, `thenCompose` enriches with external data.

---

## Core Concept 6: Modern Task Lifecycles — Auto-Closing Executor Scopes via AutoCloseable

### Definitions

**Core Definition**
Starting with Java 19, `ExecutorService` extends `AutoCloseable`, allowing executors to be used in try-with-resources blocks for deterministic shutdown and cleanup.

**Technical Definition**
Per JEP 436, `ExecutorService` was enhanced to extend `AutoCloseable`. The default `close()` method invokes `shutdown()` and waits for tasks to complete with `awaitTermination()` in a loop. This enables structured concurrency: the scope of the executor is bounded by the try-with-resources block, and no tasks can escape the scope. The executor is guaranteed to be shut down and all tasks terminated when the block exits, whether normally or exceptionally.

**Beginner-Friendly Explanation**
Before Java 19, using an executor was like leaving a restaurant without telling the staff to clean up—you had to remember to call `shutdown()` manually, or the restaurant would stay open indefinitely. With try-with-resources, the restaurant automatically closes and cleans up when you leave the block, even if you leave in a hurry (exception).

### Purposes

- To ensure deterministic shutdown of executors without manual `shutdown()` calls.
- To prevent resource leaks by guaranteeing executor termination when the block exits.
- To enable structured concurrency where all tasks are confined to a lexical scope.
- To simplify code by eliminating boilerplate shutdown and awaitTermination logic.

### Syntax Rules and Structure

**Complete General Syntax**

```java
try (ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor()) {
    // submit tasks to executor
    Future<?> future = executor.submit(() -> { ... });
    // executor is automatically closed (shutdown + awaitTermination)
} // executor is terminated here
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `try (...)` | Try-with-resources block |
| `ExecutorService executor` | The executor resource to be managed |
| `executor.close()` | Implicitly called on block exit; calls `shutdown()` + `awaitTermination()` |

**Syntax Rules**

1. `ExecutorService` extends `AutoCloseable` from Java 19 onward.
2. The `close()` method is equivalent to `shutdown()` followed by `awaitTermination()` in a loop.
3. All tasks submitted within the try-with-resources block must complete before the block exits.
4. If the block exits exceptionally, the executor is still closed.
5. `Executors.newVirtualThreadPerTaskExecutor()` is commonly used with try-with-resources for structured concurrency.

**Constraints and Limitations**

- `close()` blocks until all tasks complete; it does not interrupt running tasks.
- A long-running task will delay the close operation until it finishes.
- The try-with-resources block cannot be exited until all submitted tasks complete.
- Virtual thread executors are ideal for try-with-resources; fixed thread pools also work but may block longer.

### Annotated Code Examples

**Example 1: Basic Try-with-Resources**

```java
import java.util.concurrent.*;

public class AutoCloseableDemo {
    public static void main(String[] args) {
        // Executor is automatically closed at the end of the block
        try (ExecutorService executor =
                 Executors.newFixedThreadPool(2)) {

            Future<String> f1 = executor.submit(() -> {
                Thread.sleep(500);
                return "Task 1 done";
            });
            Future<String> f2 = executor.submit(() -> {
                Thread.sleep(300);
                return "Task 2 done";
            });

            System.out.println(f1.get());
            System.out.println(f2.get());

        } catch (InterruptedException | ExecutionException e) {
            e.printStackTrace();
        }
        // executor is automatically shut down and terminated here
        System.out.println("Executor closed");
    }
}
```

**Expected Output**

```
Task 1 done
Task 2 done
Executor closed
```

**Why This Output Occurs**

The try-with-resources block creates an executor, submits two tasks, retrieves their results, and then exits the block. On exit, `close()` is called automatically, which invokes `shutdown()` and `awaitTermination()`. Both tasks have already completed, so `close()` returns immediately. The main thread then prints "Executor closed".

---

**Example 2: Structured Concurrency with Virtual Threads**

```java
import java.util.concurrent.*;

public class StructuredConcurrencyDemo {
    public static void main(String[] args) {
        try (ExecutorService executor =
                 Executors.newVirtualThreadPerTaskExecutor()) {

            // Submit multiple tasks within the scope
            Future<String> user = executor.submit(() -> fetchUser());
            Future<String> order = executor.submit(() -> fetchOrder());
            Future<String> payment = executor.submit(() -> fetchPayment());

            // All tasks must complete before block exits
            System.out.println(user.get());
            System.out.println(order.get());
            System.out.println(payment.get());

        } catch (InterruptedException | ExecutionException e) {
            e.printStackTrace();
        }
        // All virtual threads are terminated here
    }

    static String fetchUser() {
        try { Thread.sleep(200); } catch (InterruptedException e) { }
        return "User: Alice";
    }

    static String fetchOrder() {
        try { Thread.sleep(300); } catch (InterruptedException e) { }
        return "Order: #12345";
    }

    static String fetchPayment() {
        try { Thread.sleep(100); } catch (InterruptedException e) { }
        return "Payment: Paid";
    }
}
```

**Expected Output**

```
User: Alice
Order: #12345
Payment: Paid
```

**Why This Output Occurs**

Each `executor.submit()` starts a virtual thread. The main thread calls `get()` on each future, blocking until each task completes. The try-with-resources block ensures that all virtual threads are terminated when the block exits. Virtual threads are lightweight, so creating three is inexpensive.

### Real-World Cases

- **Request Handling**: A web server uses try-with-resources to scope per-request executors that are cleaned up automatically.
- **Batch Data Processing**: A batch job wraps its executor in try-with-resources to ensure all workers finish before the job reports completion.
- **Structured Concurrency**: Java 21's `StructuredTaskScope` builds on this pattern for fail-fast task orchestration.

---

## References

- Executor (Java Platform SE 21 & JDK 21) - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/Executor.html
- ExecutorService (Java Platform SE 21 & JDK 21) - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ExecutorService.html
- ScheduledExecutorService (Java Platform SE 21 & JDK 21) - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ScheduledExecutorService.html
- ThreadPoolExecutor (Java Platform SE 21 & JDK 21) - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ThreadPoolExecutor.html
- ForkJoinPool (Java Platform SE 21 & JDK 21) - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ForkJoinPool.html
- CompletableFuture (Java Platform SE 21 & JDK 21) - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/CompletableFuture.html
- Future (Java Platform SE 21 & JDK 21) - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/Future.html
- JEP 436: Virtual Threads - https://openjdk.org/jeps/436
- Oracle Java Tutorials: Executors - https://docs.oracle.com/javase/tutorial/essential/concurrency/executors.html
- Java Concurrency in Practice (Chapter 6: Task Execution) - https://www.oreilly.com/library/view/java-concurrency-in/0321349601/