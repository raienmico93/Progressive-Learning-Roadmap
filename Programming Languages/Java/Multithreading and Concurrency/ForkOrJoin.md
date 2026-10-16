# Java Fork/Join Framework: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** The Fork/Join framework, introduced in Java 7, is an implementation of the `ExecutorService` interface designed for work that can be broken into smaller pieces recursively. It uses a work-stealing algorithm to efficiently utilise multiple processors.

**Technical Definition:** The Fork/Join framework is an implementation of the `ExecutorService` interface that helps you take advantage of multiple processors. It is designed for work that can be broken into smaller pieces recursively. The goal is to use all the available processing power to enhance the performance of your application. The framework is distinct because it uses a work-stealing algorithm: worker threads that run out of things to do can steal tasks from other threads that are still busy. The center of the framework is the `ForkJoinPool` class, an extension of `AbstractExecutorService`, which implements the core work-stealing algorithm and can execute `ForkJoinTask` processes.

**Beginner-Friendly Explanation:** Imagine a large project that one person could do alone but would take a very long time. The Fork/Join framework is like hiring a team of workers. The project is broken into smaller and smaller pieces until each piece is small enough for one worker to handle quickly. Workers who finish their tasks early look for other workers who are still busy and “steal” some of their tasks to help out. This keeps everyone busy and finishes the project much faster. The `ForkJoinPool` is the manager who coordinates the team, and `ForkJoinTask` is the work item that can be split into smaller tasks.

### Key Characteristics

- **Divide-and-Conquer:** Designed for tasks that can be recursively broken into subtasks.
- **Work-Stealing:** Idle worker threads steal tasks from the tail of busy threads' deques to balance workload.
- **Lightweight Tasks:** `ForkJoinTask` is a thread-like entity much lighter than a normal thread; huge numbers of tasks can be hosted by a small number of actual threads.
- **Common Pool:** A static `commonPool()` is available and appropriate for most applications, reducing resource usage.
- **Automatic Parallelism Tuning:** The pool dynamically adds, suspends, or resumes internal worker threads to maintain enough active threads.
- **Daemon Threads:** All worker threads are initialized with `Thread.isDaemon()` set to true.
- **Pure Computation Focus:** Fork/Join tasks should ideally avoid blocking I/O and synchronization; they are designed for computational tasks calculating pure functions or operating on isolated objects.

### Prerequisites

- Basic understanding of Java threads and the `Thread` class
- Familiarity with `Runnable` and `Callable` interfaces
- Knowledge of the `java.util.concurrent` package
- Understanding of recursion and divide-and-conquer algorithms

### Related Programming Areas

- **Concurrency:** The broader topic of multithreaded programming.
- **Executors:** The Fork/Join framework is an implementation of `ExecutorService`.
- **Parallel Streams:** `ForkJoinPool.commonPool()` powers Java's parallel streams.
- **CompletableFuture:** Uses the common Fork/Join pool for async operations.
- **Recursive Algorithms:** Merge sort, quicksort, tree traversal, and matrix operations.

### Core Concepts / Features

---

## 1. Fork/Join Framework

### Definitions

**Core Definition:** The Fork/Join framework is a concurrency framework that recursively splits tasks into subtasks (fork), executes them in parallel, and combines the results (join).

**Technical Definition:** The Fork/Join framework is an implementation of the `ExecutorService` interface that helps you take advantage of multiple processors. It is designed for work that can be broken into smaller pieces recursively. The basic pattern is: if the portion of work is small enough, do the work directly; otherwise, split the work into two pieces, invoke the two pieces, and wait for the results. This pattern is wrapped in a `ForkJoinTask` subclass, typically `RecursiveTask` (which returns a result) or `RecursiveAction` (which does not).

**Beginner-Friendly Explanation:** The Fork/Join framework is a way to solve big problems by breaking them into smaller problems, solving each smaller problem in parallel, and then combining the answers. It's like a team of workers where each worker can say “this job is too big for me alone, so I'll split it into two smaller jobs and hand one to a teammate.”

### Purposes

- To efficiently utilise multiple processors for recursive, divide-and-conquer tasks
- To provide automatic workload balancing through work-stealing
- To handle large numbers of small tasks with minimal thread overhead
- To serve as the foundation for parallel streams and `CompletableFuture`
- To enable high-throughput computation without manual thread management

### Syntax Rules and Structure

**Fork/Join Pattern:**

```
if (task is small enough) {
    compute directly
} else {
    split task into subtasks
    fork subtasks (execute asynchronously)
    join subtasks (wait for results)
    combine results
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `ForkJoinPool` | Manages worker threads and executes `ForkJoinTask`s |
| `ForkJoinTask<V>` | Abstract base class for tasks |
| `RecursiveTask<V>` | A task that returns a result |
| `RecursiveAction` | A task that does not return a result |
| `fork()` | Arranges asynchronous execution |
| `join()` | Waits for the task's result |
| `compute()` | The method that performs the task's computation |

**Syntax Rules:**

- Tasks must extend `ForkJoinTask`, typically via `RecursiveTask` or `RecursiveAction`.
- The `compute()` method is the main hook method that subclasses must override.
- `fork()` and `join()` are the primary coordination mechanisms.
- Computations should ideally avoid synchronized methods/blocks and minimize blocking synchronization apart from joining other tasks.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Basic Fork/Join with RecursiveTask**

```java
// File: ForkJoinBasicDemo.java
import java.util.concurrent.*;

public class ForkJoinBasicDemo {
    // Step 1: Create a RecursiveTask subclass
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
            // Step 2: Check if task is small enough
            if (end - start <= THRESHOLD) {
                long sum = 0;
                for (int i = start; i < end; i++) {
                    sum += array[i];
                }
                return sum;
            }

            // Step 3: Split into subtasks
            int mid = (start + end) / 2;
            SumTask left = new SumTask(array, start, mid);
            SumTask right = new SumTask(array, mid, end);

            // Step 4: Fork the left task (execute asynchronously)
            left.fork();

            // Step 5: Compute right task directly
            long rightResult = right.compute();

            // Step 6: Join the left task (wait for result)
            long leftResult = left.join();

            // Step 7: Combine results
            return leftResult + rightResult;
        }
    }

    public static void main(String[] args) {
        // Step 8: Create data
        int[] array = new int[1_000_000];
        for (int i = 0; i < array.length; i++) {
            array[i] = i + 1;
        }

        // Step 9: Create ForkJoinPool and invoke the task
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

1. `SumTask` extends `RecursiveTask<Long>`, meaning it returns a `Long` result.
2. The `compute()` method checks if the segment is small enough (`THRESHOLD = 10000`). If so, it computes the sum directly.
3. If the segment is too large, it splits into left and right halves.
4. `left.fork()` submits the left task for asynchronous execution.
5. `right.compute()` computes the right task directly in the current thread.
6. `left.join()` waits for the left task to complete and retrieves its result.
7. The results are combined and returned.
8. The `ForkJoinPool.invoke()` method starts the computation and waits for the final result.
9. The sum of integers from 1 to 1,000,000 is 500,000,500,000.

### Real-World Cases

- **Parallel Sorting:** Merge sort and quicksort using Fork/Join.
- **Matrix Multiplication:** Dividing matrices into blocks for parallel computation.
- **Tree Traversal:** Parallel processing of tree structures.
- **Image Processing:** Applying filters to image segments in parallel.

---

## 2. `ForkJoinPool`

### Definitions

**Core Definition:** `ForkJoinPool` is an `ExecutorService` for running `ForkJoinTask`s, providing the entry point for submissions and employing a work-stealing algorithm.

**Technical Definition:** An `ExecutorService` for running `ForkJoinTask`s. A `ForkJoinPool` provides the entry point for submissions from non-`ForkJoinTask` clients, as well as management and monitoring operations. A `ForkJoinPool` differs from other kinds of `ExecutorService` mainly by virtue of employing work-stealing: all threads in the pool attempt to find and execute tasks submitted to the pool and/or created by other active tasks. All worker threads are initialized with `Thread.isDaemon()` set to true. A static `commonPool()` is available and appropriate for most applications.

**Beginner-Friendly Explanation:** `ForkJoinPool` is the manager of the Fork/Join team. It creates worker threads, distributes tasks, and ensures that idle workers help busy ones. You can use the default common pool or create your own with a specific parallelism level.

### Purposes

- To provide the entry point for submitting `ForkJoinTask`s
- To manage worker threads and their work-stealing queues
- To offer management and monitoring operations (e.g., `getStealCount()`)
- To serve as the execution engine for parallel streams and `CompletableFuture`
- To enable custom parallelism levels for different workloads

### Syntax Rules and Structure

**Creating a ForkJoinPool:**

```java
// Use the common pool
ForkJoinPool pool = ForkJoinPool.commonPool();

// Create a custom pool
ForkJoinPool pool = new ForkJoinPool();
ForkJoinPool pool = new ForkJoinPool(int parallelism);
ForkJoinPool pool = new ForkJoinPool(int parallelism, ForkJoinWorkerThreadFactory factory,
    Thread.UncaughtExceptionHandler handler, boolean asyncMode);
```

**Key Methods:**

| Method | Description |
|--------|-------------|
| `invoke(ForkJoinTask)` | Executes a task and returns its result |
| `execute(ForkJoinTask)` | Submits a task for async execution |
| `submit(ForkJoinTask)` | Submits a task and returns a `Future` |
| `commonPool()` | Returns the common pool instance |
| `shutdown()` | Initiates orderly shutdown |
| `awaitTermination(long, TimeUnit)` | Waits for shutdown |
| `getParallelism()` | Returns the target parallelism level |
| `getStealCount()` | Returns the total number of steals |

**Syntax Rules:**

- The common pool is used by any `ForkJoinTask` not explicitly submitted to a specified pool.
- Custom pools can be constructed with a given target parallelism level; by default, equal to the number of available processors.
- The pool dynamically adjusts worker threads but does not guarantee adjustments in the face of blocked I/O.

**Constraints and Limitations:**

- The common pool can be overloaded if used for all tasks; use custom pools for long-running or special tasks.
- Worker threads are daemon threads, so they do not prevent JVM shutdown.
- Blocking I/O is not well-supported; use `ManagedBlocker` for managed blocking.

### Annotated Complete Step-by-Step Code Examples

```java
// File: ForkJoinPoolDemo.java
import java.util.concurrent.*;

public class ForkJoinPoolDemo {
    public static void main(String[] args) throws Exception {
        // Create a custom pool with 4 threads
        ForkJoinPool pool = new ForkJoinPool(4);

        // Submit a Callable task
        Future<String> future = pool.submit(() -> {
            return "Task result from " + Thread.currentThread().getName();
        });

        System.out.println(future.get());

        // Monitor pool state
        System.out.println("Parallelism: " + pool.getParallelism());
        System.out.println("Steal count: " + pool.getStealCount());
        System.out.println("Active threads: " + pool.getActiveThreadCount());

        pool.shutdown();
        pool.awaitTermination(5, TimeUnit.SECONDS);
    }
}
```

**Expected Output (varies):**
```
Task result from ForkJoinPool-1-worker-1
Parallelism: 4
Steal count: 0
Active threads: 0
```

### Real-World Cases

- **Parallel Streams:** The common pool powers `list.parallelStream()`.
- **CompletableFuture:** Async methods use the common pool by default.
- **Custom Parallel Algorithms:** Using custom pools for specific parallelism requirements.

---

## 3. `ForkJoinTask`

### Definitions

**Core Definition:** `ForkJoinTask<V>` is the abstract base class for tasks that run within a `ForkJoinPool`, providing `fork()` and `join()` coordination mechanisms.

**Technical Definition:** Abstract base class for tasks that run within a `ForkJoinPool`. A `ForkJoinTask` is a thread-like entity that is much lighter weight than a normal thread. Huge numbers of tasks and subtasks may be hosted by a small number of actual threads in a `ForkJoinPool`, at the price of some usage limitations. A “main” `ForkJoinTask` begins execution when it is explicitly submitted to a `ForkJoinPool`, or, if not already engaged in a Fork/Join computation, commenced in the `ForkJoinPool.commonPool()` via `fork()`, `invoke()`, or related methods.

**Beginner-Friendly Explanation:** `ForkJoinTask` is the work item in the Fork/Join framework. It's like a task on a to-do list that can be split into smaller tasks. It's very lightweight — you can have thousands or millions of them without the overhead of thousands of threads.

### Purposes

- To represent a unit of work in the Fork/Join framework
- To provide `fork()` and `join()` methods for coordination
- To serve as the base class for `RecursiveTask` and `RecursiveAction`
- To enable lightweight task management with minimal overhead

### Syntax Rules and Structure

**Key Methods:**

| Method | Description |
|--------|-------------|
| `fork()` | Arranges asynchronous execution |
| `join()` | Waits for and returns the task's result |
| `invoke()` | Executes the task and returns its result |
| `compute()` | The abstract hook method for task computation |
| `isDone()` | Returns `true` if completed |
| `cancel(boolean)` | Attempts to cancel the task |
| `get(long, TimeUnit)` | Waits with a timeout for completion |

**Syntax Rules:**

- `compute()` must be implemented by subclasses.
- `fork()` and `join()` are the primary coordination mechanisms.
- Computations should ideally avoid synchronized methods/blocks and minimize blocking synchronization.
- Checked exceptions cannot be thrown; unchecked exceptions are rethrown to callers attempting to join.

### Annotated Complete Step-by-Step Code Examples

```java
// File: ForkJoinTaskDemo.java
import java.util.concurrent.*;

public class ForkJoinTaskDemo {
    static class RecursiveSum extends RecursiveTask<Long> {
        private static final int THRESHOLD = 1000;
        private final int[] array;
        private final int start, end;

        RecursiveSum(int[] array, int start, int end) {
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
            RecursiveSum left = new RecursiveSum(array, start, mid);
            RecursiveSum right = new RecursiveSum(array, mid, end);
            left.fork();
            long rightResult = right.compute();
            long leftResult = left.join();
            return leftResult + rightResult;
        }
    }

    public static void main(String[] args) {
        int[] array = new int[10000];
        for (int i = 0; i < array.length; i++) array[i] = i + 1;

        ForkJoinPool pool = new ForkJoinPool();
        long sum = pool.invoke(new RecursiveSum(array, 0, array.length));
        System.out.println("Sum: " + sum);  // 50005000
        pool.shutdown();
    }
}
```

**Expected Output:**
```
Sum: 50005000
```

### Real-World Cases

- **Parallel Reduction:** Summing, averaging, or finding maximum values.
- **Recursive Algorithms:** Sorting, searching, and tree processing.
- **Data Processing:** Parallel transformation of large datasets.

---

## 4. `RecursiveTask`

### Definitions

**Core Definition:** `RecursiveTask<V>` is a recursive result-bearing `ForkJoinTask` that returns a value of type `V`.

**Technical Definition:** A recursive result-bearing `ForkJoinTask`. For example, here is a task-based program for computing factorials or Fibonacci numbers. Subclasses override `compute()` to perform the task's main computation. `RecursiveTask` extends `ForkJoinTask<V>` and is used for computations that return results.

**Beginner-Friendly Explanation:** `RecursiveTask` is a task that gives you something back when it's done. If you're computing a sum, a maximum, or a sorted list, you use `RecursiveTask`.

### Purposes

- To define recursive tasks that produce a result
- To enable parallel divide-and-conquer computations with return values
- To serve as the primary task type for parallel reductions and transformations
- To combine subtask results into a final result

### Syntax Rules and Structure

```java
public abstract class RecursiveTask<V> extends ForkJoinTask<V> {
    protected abstract V compute();
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `V` | The type of the result |
| `compute()` | The main computation performed by this task |

**Syntax Rules:**

- Subclasses must override `compute()`.
- The result is returned from `compute()` and retrieved via `join()` or `invoke()`.
- `RecursiveTask` is used when the computation produces a result.

### Annotated Complete Step-by-Step Code Examples

```java
// File: RecursiveTaskDemo.java
import java.util.concurrent.*;

public class RecursiveTaskDemo {
    // Task to find the maximum value in an array
    static class MaxTask extends RecursiveTask<Integer> {
        private static final int THRESHOLD = 1000;
        private final int[] array;
        private final int start, end;

        MaxTask(int[] array, int start, int end) {
            this.array = array;
            this.start = start;
            this.end = end;
        }

        @Override
        protected Integer compute() {
            if (end - start <= THRESHOLD) {
                int max = Integer.MIN_VALUE;
                for (int i = start; i < end; i++) {
                    if (array[i] > max) max = array[i];
                }
                return max;
            }
            int mid = (start + end) / 2;
            MaxTask left = new MaxTask(array, start, mid);
            MaxTask right = new MaxTask(array, mid, end);
            left.fork();
            int rightResult = right.compute();
            int leftResult = left.join();
            return Math.max(leftResult, rightResult);
        }
    }

    public static void main(String[] args) {
        int[] array = new int[100000];
        for (int i = 0; i < array.length; i++) array[i] = i + 1;
        array[54321] = 999999;  // Set a maximum

        ForkJoinPool pool = new ForkJoinPool();
        int max = pool.invoke(new MaxTask(array, 0, array.length));
        System.out.println("Max: " + max);  // 999999
        pool.shutdown();
    }
}
```

**Expected Output:**
```
Max: 999999
```

### Real-World Cases

- **Parallel Search:** Finding the maximum, minimum, or specific element.
- **Parallel Sorting:** Merge sort returns a sorted array.
- **Parallel Reduction:** Summing, averaging, or counting elements.
- **Fibonacci/Factorial:** Computing mathematical sequences.

---

## 5. `RecursiveAction`

### Definitions

**Core Definition:** `RecursiveAction` is a recursive resultless `ForkJoinTask` that performs an action without returning a value.

**Technical Definition:** A recursive resultless `ForkJoinTask`. This class establishes conventions to parameterize resultless actions as `Void` `ForkJoinTask`s. Because `null` is the only valid value of type `Void`, methods such as `join` always return `null` upon completion. `RecursiveAction` extends `ForkJoinTask<Void>` and is used for computations that do not return results.

**Beginner-Friendly Explanation:** `RecursiveAction` is a task that does something but doesn't give you anything back. If you're sorting an array in place, incrementing values, or applying a transformation, you use `RecursiveAction`.

### Purposes

- To define recursive tasks that perform actions without returning results
- To enable parallel in-place modifications of data structures
- To serve as the task type for operations like sorting, incrementing, and transforming
- To avoid the overhead of returning and combining results when not needed

### Syntax Rules and Structure

```java
public abstract class RecursiveAction extends ForkJoinTask<Void> {
    protected abstract void compute();
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `compute()` | The main computation performed by this task; returns `void` |

**Syntax Rules:**

- Subclasses must override `compute()`.
- `join()` always returns `null` for `RecursiveAction`.
- Used when the computation modifies data in place and does not produce a return value.

### Annotated Complete Step-by-Step Code Examples

```java
// File: RecursiveActionDemo.java
import java.util.concurrent.*;

public class RecursiveActionDemo {
    // Task to increment all elements in an array
    static class IncrementTask extends RecursiveAction {
        private static final int THRESHOLD = 1000;
        private final int[] array;
        private final int start, end;

        IncrementTask(int[] array, int start, int end) {
            this.array = array;
            this.start = start;
            this.end = end;
        }

        @Override
        protected void compute() {
            if (end - start <= THRESHOLD) {
                for (int i = start; i < end; i++) {
                    array[i]++;
                }
            } else {
                int mid = (start + end) / 2;
                IncrementTask left = new IncrementTask(array, start, mid);
                IncrementTask right = new IncrementTask(array, mid, end);
                left.fork();
                right.compute();
                left.join();
            }
        }
    }

    public static void main(String[] args) {
        int[] array = new int[10000];
        for (int i = 0; i < array.length; i++) array[i] = i;

        ForkJoinPool pool = new ForkJoinPool();
        pool.invoke(new IncrementTask(array, 0, array.length));

        // Verify
        boolean correct = true;
        for (int i = 0; i < array.length; i++) {
            if (array[i] != i + 1) { correct = false; break; }
        }
        System.out.println("All incremented correctly: " + correct);  // true
        pool.shutdown();
    }
}
```

**Expected Output:**
```
All incremented correctly: true
```

### Real-World Cases

- **Parallel Sorting:** Merge sort and quicksort that modify arrays in place.
- **Image Processing:** Applying filters to image pixels.
- **Matrix Operations:** In-place transformations of matrices.
- **Data Cleaning:** Normalising or transforming data elements.

---

## 6. Work Stealing

### Definitions

**Core Definition:** Work stealing is a scheduling algorithm where idle worker threads “steal” tasks from the tail of busy threads' deques, balancing workload dynamically.

**Technical Definition:** The Fork/Join framework is distinct because it uses a work-stealing algorithm. Worker threads that run out of things to do can steal tasks from other threads that are still busy. Subtasks generated by worker threads (using `fork`) are pushed onto their own deque. The worker threads process their own deque LIFO (youngest-first), while workers steal from other deques FIFO (oldest-first). This minimizes contention with the worker thread owning the deque and ensures that an older stolen task may provide a larger unit of work.

**Beginner-Friendly Explanation:** Work stealing is like a team of workers where each has a personal to-do list. When a worker finishes their list, they look at other workers' lists and take the oldest tasks (from the bottom of the pile). This keeps everyone busy and prevents some workers from being overloaded while others are idle.

### Purposes

- To dynamically balance workload across worker threads
- To maximise CPU utilisation by keeping all workers busy
- To minimise contention by having workers primarily process their own tasks
- To enable efficient processing of irregular, recursive workloads

### How It Works

**Algorithm Steps:**

1. Each worker thread has its own deque (double-ended queue) of tasks.
2. When a task is forked, it is pushed onto the **tail** of the worker's own deque (LIFO order).
3. The worker processes tasks from the **tail** of its own deque (youngest-first).
4. When a worker's deque is empty, it becomes a thief and randomly selects a victim worker.
5. The thief steals a task from the **head** of the victim's deque (FIFO order — oldest-first).
6. This minimizes contention because the victim works on the tail while the thief steals from the head.

**Key Properties:**

| Property | Description |
|----------|-------------|
| **LIFO for own tasks** | Workers process their own tasks youngest-first |
| **FIFO for stolen tasks** | Thieves steal oldest tasks first |
| **Random victim selection** | Reduces contention by spreading steals |
| **Work-stealing queues** | Special deques supporting push, pop, and steal (poll) operations |

### Annotated Code Example

```java
// Work stealing is internal to ForkJoinPool — no direct API.
// The following demonstrates the concept through performance characteristics.

// File: WorkStealingDemo.java
import java.util.concurrent.*;

public class WorkStealingDemo {
    public static void main(String[] args) throws Exception {
        ForkJoinPool pool = new ForkJoinPool(4);

        // Create tasks with unequal workloads to demonstrate stealing
        for (int i = 1; i <= 8; i++) {
            final int taskId = i;
            pool.execute(() -> {
                long start = System.nanoTime();
                // Simulate variable workload
                long work = taskId * 100_000_000L;
                for (long j = 0; j < work; j++) { /* busy work */ }
                long elapsed = (System.nanoTime() - start) / 1_000_000;
                System.out.println("Task " + taskId + " completed in " +
                    elapsed + "ms on " + Thread.currentThread().getName());
            });
        }

        Thread.sleep(500);
        System.out.println("Steal count: " + pool.getStealCount());
        System.out.println("Active threads: " + pool.getActiveThreadCount());

        pool.shutdown();
        pool.awaitTermination(10, TimeUnit.SECONDS);
    }
}
```

**Expected Output (varies):**
```
Steal count: 12
Active threads: 4
Task 1 completed in 50ms on ForkJoinPool-1-worker-1
Task 2 completed in 100ms on ForkJoinPool-1-worker-2
...
```

### Real-World Cases

- **Parallel Streams:** Work stealing balances the workload across cores.
- **Recursive Algorithms:** Merge sort and quicksort benefit from dynamic balancing.
- **Irregular Workloads:** Tasks with varying execution times are automatically balanced.

---

## 7. Parallelism

### Definitions

**Core Definition:** Parallelism in the Fork/Join framework refers to the number of worker threads that can execute tasks simultaneously, typically set to the number of available processors.

**Technical Definition:** A `ForkJoinPool` may be constructed with a given target parallelism level; by default, equal to the number of available processors. The pool attempts to maintain enough active (or available) threads by dynamically adding, suspending, or resuming internal worker threads, even if some tasks are stalled waiting to join others. The common pool's parallelism can be controlled via the system property `java.util.concurrent.ForkJoinPool.common.parallelism`.

**Beginner-Friendly Explanation:** Parallelism is the number of workers on your team. By default, it's set to the number of CPU cores your computer has. If you have 8 cores, you get 8 workers. You can change this number if you have specific needs.

### Purposes

- To control the number of concurrent threads in the pool
- To match the pool size to the available hardware
- To balance resource usage and performance
- To avoid excessive thread creation and context switching

### Syntax Rules and Structure

```java
// Default: number of available processors
ForkJoinPool pool = new ForkJoinPool();

// Custom parallelism
ForkJoinPool pool = new ForkJoinPool(8);

// Common pool parallelism via system property
System.setProperty("java.util.concurrent.ForkJoinPool.common.parallelism", "8");
```

**Key Methods:**

| Method | Description |
|--------|-------------|
| `getParallelism()` | Returns the target parallelism level |
| `setParallelism(int)` | Changes the target parallelism (Java 21+) |

**Syntax Rules:**

- The default parallelism is `Runtime.getRuntime().availableProcessors()`.
- The common pool's parallelism can be set via system property before first use.
- The pool dynamically adjusts threads but does not guarantee adjustments for blocked I/O.

**Constraints and Limitations:**

- Setting parallelism too high can cause excessive context switching and overhead.
- Setting it too low can underutilise available cores.
- For I/O-bound tasks, consider a different approach (e.g., virtual threads).

### Annotated Code Example

```java
// File: ParallelismDemo.java
import java.util.concurrent.*;

public class ParallelismDemo {
    public static void main(String[] args) {
        // Default parallelism (number of processors)
        ForkJoinPool defaultPool = new ForkJoinPool();
        System.out.println("Default parallelism: " + defaultPool.getParallelism());

        // Custom parallelism
        ForkJoinPool customPool = new ForkJoinPool(4);
        System.out.println("Custom parallelism: " + customPool.getParallelism());

        // Common pool parallelism
        ForkJoinPool common = ForkJoinPool.commonPool();
        System.out.println("Common pool parallelism: " + common.getParallelism());

        defaultPool.shutdown();
        customPool.shutdown();
    }
}
```

**Expected Output (varies by CPU):**
```
Default parallelism: 8
Custom parallelism: 4
Common pool parallelism: 7
```

### Real-World Cases

- **CPU-Bound Tasks:** Set parallelism to the number of cores.
- **Mixed Workloads:** Adjust parallelism based on task characteristics.
- **Testing:** Reduce parallelism to reproduce concurrency issues.

---

## 8. Fork/Join Best Practices

### Definitions

**Core Definition:** Fork/Join best practices are guidelines for designing, implementing, and tuning Fork/Join tasks to achieve optimal performance, correctness, and maintainability.

**Technical Definition:** Best practices derive from authoritative sources including Oracle's Java documentation and community experience. Key principles include choosing appropriate thresholds, avoiding blocking operations, using associative operations, minimising shared mutable data, and selecting the right pool for the workload.

**Beginner-Friendly Explanation:** Best practices help you avoid common pitfalls like making tasks too small (overhead outweighs benefits), blocking workers (which stalls the entire pool), or using non-associative operations (which produce incorrect results). They're like a tuning manual for the Fork/Join framework.

### Key Best Practices Table

| Practice | Description | Source |
|----------|-------------|--------|
| Choose the Right Threshold | For very small tasks, sequential execution may be faster than forking; experiment to find the optimal threshold | Open-Std |
| Avoid Blocking Operations | Fork/Join tasks should not perform blocking I/O or synchronization; use `ManagedBlocker` if blocking is unavoidable | Oracle |
| Use Associative Operations | The combine operation must be associative to produce correct results regardless of grouping | Open-Std |
| Minimise Shared Mutable Data | Reduce shared state to minimise synchronization and potential bugs | Open-Std |
| Use Custom Pools for Special Tasks | The common pool can be overloaded; use custom pools for long-running or special tasks | Open-Std |
| Avoid Excessive Thread Creation | Use a suitable parallelism level, typically based on available processors | Open-Std |
| Ensure Tasks Are Balanced | Reasonably sized and balanced tasks avoid contention and work-stealing overhead | Open-Std |
| Use Thread-Safe Data Structures | When sharing data, use `java.util.concurrent` data structures | Open-Std |
| Prefer `RecursiveTask` for Results | Use `RecursiveTask` when the computation returns a value; use `RecursiveAction` otherwise | Oracle |
| Avoid Deep Recursion | Overly deep recursion can cause `StackOverflowError`; balance fork/join depth | Best practice |
| Test with Different Parallelism Levels | Performance characteristics change with parallelism; test across configurations | Best practice |
| Monitor Pool Metrics | Use `getStealCount()`, `getActiveThreadCount()`, and `toString()` for tuning | Oracle |

### Annotated Complete Step-by-Step Code Examples

```java
// File: ForkJoinBestPracticesDemo.java
import java.util.concurrent.*;

public class ForkJoinBestPracticesDemo {
    static class SafeSumTask extends RecursiveTask<Long> {
        // BEST PRACTICE: Choose a threshold that balances overhead and parallelism
        private static final int THRESHOLD = 5000;
        private final int[] array;
        private final int start, end;

        SafeSumTask(int[] array, int start, int end) {
            this.array = array;
            this.start = start;
            this.end = end;
        }

        @Override
        protected Long compute() {
            // BEST PRACTICE: Base case — compute directly when small
            if (end - start <= THRESHOLD) {
                long sum = 0;
                for (int i = start; i < end; i++) sum += array[i];
                return sum;
            }

            int mid = (start + end) / 2;
            SafeSumTask left = new SafeSumTask(array, start, mid);
            SafeSumTask right = new SafeSumTask(array, mid, end);

            // BEST PRACTICE: Fork one, compute the other directly
            left.fork();
            long rightResult = right.compute();
            long leftResult = left.join();

            // BEST PRACTICE: Associative operation (addition is associative)
            return leftResult + rightResult;
        }
    }

    public static void main(String[] args) throws Exception {
        int[] array = new int[1_000_000];
        for (int i = 0; i < array.length; i++) array[i] = i + 1;

        // BEST PRACTICE: Use a custom pool for controlled parallelism
        ForkJoinPool pool = new ForkJoinPool(4);

        long start = System.nanoTime();
        long sum = pool.invoke(new SafeSumTask(array, 0, array.length));
        long elapsed = (System.nanoTime() - start) / 1_000_000;

        System.out.println("Sum: " + sum);
        System.out.println("Time: " + elapsed + "ms");
        System.out.println("Steal count: " + pool.getStealCount());
        System.out.println("Pool state: " + pool);

        pool.shutdown();
        pool.awaitTermination(5, TimeUnit.SECONDS);
    }
}
```

**Expected Output (varies):**
```
Sum: 500000500000
Time: 45ms
Steal count: 7
Pool state: java.util.concurrent.ForkJoinPool@... [Running, parallelism = 4, size = 4, active = 0, running = 0, steals = 7, tasks = 0, submissions = 0]
```

**Step-by-Step Explanation:**

1. A threshold of 5000 is chosen to balance fork overhead against parallelism.
2. The base case computes directly when the segment is small enough.
3. One subtask is forked; the other is computed directly to reduce overhead.
4. Addition is associative, so combining left and right results is correct regardless of grouping.
5. A custom pool with 4 threads provides controlled parallelism.
6. `getStealCount()` shows that work stealing occurred, indicating balanced workload.

### Real-World Cases

- **Parallel Sorting:** Choosing appropriate thresholds for merge sort.
- **Image Processing:** Balancing image segments for filter application.
- **Data Aggregation:** Using associative operations for parallel sums and averages.

---

## References Links

- ForkJoinPool — Java Platform SE 21 API Specification - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ForkJoinPool.html
- Fork/Join — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/essential/concurrency/forkjoin.html
- ForkJoinTask — Java Platform SE 8 API Specification - https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ForkJoinTask.html
- RecursiveTask — Java Platform SE 21 API Specification - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/RecursiveTask.html
- RecursiveAction — Java Platform SE 21 API Specification - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/RecursiveAction.html
- Guide to Work Stealing in Java — Baeldung - https://www.baeldung.com/java-work-stealing
- Java Fork/Join Framework Internals — Douglas C. Schmidt, Vanderbilt University - https://www.dre.vanderbilt.edu/~schmidt/cs891f/2018-PDFs/fork-join-framework-internals.pdf
- The Java Fork-Join Pool Framework — Douglas C. Schmidt, Vanderbilt University - https://www.dre.vanderbilt.edu/~schmidt/cs891f/2018-PDFs/fork-join-framework.pdf
- ISO/IEC JTC1 SC22 WG23 Concurrency Update (Java) — Open-Std - https://www.open-std.org/jtc1/sc22/wg23/docs/ISO-IECJTC1-SC22-WG23_N1489-concurrency-update-java-20250604.pdf