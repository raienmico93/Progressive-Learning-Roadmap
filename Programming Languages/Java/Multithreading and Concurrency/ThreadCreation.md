# Java Threads: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A thread is the smallest unit of execution within a process, representing an independent path of execution through a program's code. Java threads allow a program to perform multiple tasks concurrently, sharing the same memory space and system resources.

**Technical Definition:** "A thread is a thread of execution in a program. The Java virtual machine allows an application to have multiple threads of execution running concurrently". "Thread defines constructors and a Thread.Builder to create threads. Starting a thread schedules it to execute its run method. The newly started thread executes concurrently with the thread that caused it to start". "A thread terminates if either its run method completes normally, or if its run method completes abruptly and the appropriate uncaught exception handler completes normally or abruptly". Threads share the process's resources, including memory and open files, making communication between threads efficient but requiring synchronization to prevent data corruption.

**Beginner-Friendly Explanation:** A thread is like a worker on an assembly line. The assembly line (the process) has a shared workspace, tools, and materials. Each worker (thread) performs their own task, and multiple workers can be active at the same time, coordinating their efforts. When you write a simple Java program, you're already using one worker — the main thread. Creating additional threads is like hiring more workers to get more done simultaneously. For example, one worker can download a file while another updates the user interface, so the program doesn't freeze.

### Key Characteristics

- **Concurrent Execution:** Multiple threads within a process can run concurrently, interleaving their execution.
- **Shared Memory:** Threads share the process's memory space, enabling efficient communication but requiring synchronization.
- **Independent Paths:** Each thread has its own call stack, program counter, and local variables.
- **Lightweight:** Creating a thread is less expensive than creating a process.
- **Platform Threads:** Mapped 1:1 to OS kernel threads, scheduled by the operating system.
- **Virtual Threads (Java 21+):** JVM-managed lightweight threads suitable for I/O-bound tasks; a single JVM may support millions.
- **Daemon Status:** Daemon threads do not prevent the JVM from shutting down when all non-daemon threads terminate.
- **Priority Hints:** Thread priorities (1–10) provide hints to the scheduler but are platform-dependent.
- **Thread Groups:** Threads can be organised into groups for bulk operations and exception handling.

### Prerequisites

- Basic understanding of Java classes, objects, and interfaces
- Familiarity with the `java.lang` package
- Knowledge of exceptions and try-catch-finally
- Understanding of the `Runnable` interface (or willingness to learn)

### Related Programming Areas

- **Concurrency Fundamentals:** The broader topic of concurrent and parallel programming.
- **Synchronization:** Coordinating access to shared resources (`synchronized`, locks).
- **Thread Pools:** Reusing threads via `ExecutorService` for efficiency.
- **Concurrent Collections:** Thread-safe data structures in `java.util.concurrent`.
- **Asynchronous Programming:** `CompletableFuture`, reactive streams.
- **Virtual Threads (Project Loom):** Lightweight threads for high-throughput concurrency.

### Core Concepts / Features

---

## 1. `Thread` Class

### Definitions

**Core Definition:** The `Thread` class in `java.lang` represents a thread of execution and provides the API for creating, starting, and managing threads.

**Technical Definition:** "A thread is a thread of execution in a program. The Java virtual machine allows an application to have multiple threads of execution running concurrently. Thread defines constructors and a Thread.Builder to create threads". The `Thread` class implements `Runnable` and provides methods for starting (`start()`), pausing (`sleep()`), waiting (`join()`), interrupting (`interrupt()`), and querying state (`getState()`). "Threads have a unique identifier and a name. The identifier is generated when a Thread is created and cannot be changed. The thread name can be specified when creating a thread or can be changed at a later time".

**Beginner-Friendly Explanation:** The `Thread` class is the blueprint for creating workers in Java. You create a `Thread` object, give it a task (via `run()` or a `Runnable`), and then call `start()` to hire the worker. The worker then executes the task in parallel with your main program. The `Thread` class also provides tools to manage workers — pausing them, waiting for them to finish, and asking them to stop.

### Purposes

- To create and manage threads of execution within a Java program
- To execute tasks concurrently, improving responsiveness and throughput
- To provide lifecycle methods for controlling thread execution
- To support thread naming, prioritisation, daemon status, and grouping
- To serve as the foundation for higher-level concurrency utilities

### Syntax Rules and Structure

**Complete General Syntax (Creating a Thread):**

```java
// Approach 1: Extend Thread
class MyThread extends Thread {
    public void run() {
        // task code
    }
}
new MyThread().start();

// Approach 2: Implement Runnable (preferred)
Runnable task = () -> { /* task code */ };
new Thread(task).start();

// Approach 3: Thread.Builder (Java 21+)
Thread thread = Thread.ofPlatform().name("worker").start(() -> { /* task */ });
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Thread` | The class representing a thread of execution |
| `start()` | Schedules the thread to begin execution |
| `run()` | Contains the thread's task |
| `Runnable` | A functional interface for the task |
| `Thread.Builder` | Modern API for creating threads (Java 21+) |

**Syntax Rules:**

- Calling `start()` schedules the thread; the JVM invokes `run()` when the thread executes.
- Calling `run()` directly (instead of `start()`) executes the task in the current thread — a common mistake.
- A thread can be started only once; calling `start()` twice throws `IllegalThreadStateException`.
- Threads support `ThreadLocal` variables, which are local to each thread.

**Constraints and Limitations:**

- Platform threads are a limited resource; creating too many causes resource exhaustion.
- Virtual threads (Java 21+) are suitable for I/O-bound tasks, not CPU-intensive operations.
- "Thread may be extended for customization and other advanced reasons although most applications should have little need to do this".

### Annotated Complete Step-by-Step Code Examples

**Example 1: Creating and Starting a Thread**

```java
// File: ThreadBasicDemo.java
public class ThreadBasicDemo {
    public static void main(String[] args) {
        // Step 1: Create a Runnable task
        Runnable task = () -> {
            for (int i = 1; i <= 3; i++) {
                System.out.println(Thread.currentThread().getName() + ": " + i);
                try { Thread.sleep(500); } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            }
        };

        // Step 2: Create a Thread with a name
        Thread worker = new Thread(task, "Worker-A");

        // Step 3: Start the thread
        worker.start();

        // Step 4: Main thread continues concurrently
        System.out.println("Main thread: running concurrently");
    }
}
```

**Expected Output (order may vary):**
```
Main thread: running concurrently
Worker-A: 1
Worker-A: 2
Worker-A: 3
```

**Step-by-Step Explanation:**

1. A `Runnable` is defined with a lambda that prints the current thread's name and a counter.
2. A `Thread` is created with the `Runnable` and a descriptive name `"Worker-A"`.
3. `start()` schedules the thread for execution; the JVM invokes `run()` concurrently.
4. The main thread continues immediately, printing its message.
5. The worker thread executes its loop, interleaving with the main thread.

### Real-World Cases

- **Web Servers:** Handling each client request in a separate thread.
- **Background Tasks:** Performing logging, cleanup, or scheduled jobs without blocking the main thread.
- **GUI Applications:** Keeping the UI responsive while background work runs.
- **Parallel Processing:** Dividing a large computation across multiple threads.

---

## 2. `Runnable` Interface

### Definitions

**Core Definition:** `Runnable` is a functional interface with a single abstract method `run()` that represents a task to be executed by a thread.

**Technical Definition:** "The Runnable interface should be implemented by any class whose instances are intended to be executed by a thread. The class must define a method of no arguments called run. This interface is designed to provide a common protocol for objects that wish to execute code while they are active". "Runnable provides the means for a class to be active while not subclassing Thread". "In most cases, the Runnable interface should be used if you are only planning to override the run() method and no other Thread methods".

**Beginner-Friendly Explanation:** `Runnable` is a job description. It says "whoever implements this interface must know how to perform this task." When you give a `Runnable` to a `Thread`, you're hiring a worker and handing them the job description. The worker will follow the instructions in `run()`.

### Purposes

- To define a task that can be executed by a thread without subclassing `Thread`
- To separate the task (what to do) from the thread (how to execute it)
- To allow a class to be active while still extending another class
- To enable lambda expressions for defining tasks concisely
- To serve as the target type for thread creation

### Syntax Rules and Structure

**Complete General Syntax:**

```java
// Approach 1: Implement Runnable
public class MyTask implements Runnable {
    @Override
    public void run() {
        // task code
    }
}
new Thread(new MyTask()).start();

// Approach 2: Lambda (Runnable is a functional interface)
Runnable task = () -> { /* task code */ };
new Thread(task).start();
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Runnable` | The functional interface |
| `run()` | The single abstract method containing the task |
| `Thread(Runnable)` | Constructor that accepts a `Runnable` target |

**Syntax Rules:**

- `Runnable` is a `@FunctionalInterface`, so it can be implemented with a lambda or method reference.
- The `run()` method takes no arguments and returns `void`.
- "Runnable is implemented by class Thread".

**Constraints and Limitations:**

- `run()` cannot return a value or throw checked exceptions (use `Callable` for that).
- "Do not subclass Thread if you can use a Runnable instead" (SEI CERT).

### Annotated Complete Step-by-Step Code Examples

```java
// File: RunnableDemo.java
public class RunnableDemo {
    public static void main(String[] args) {
        // Approach 1: Implement Runnable with a named class
        Runnable task1 = new Runnable() {
            @Override
            public void run() {
                System.out.println("Task 1 running in: " +
                    Thread.currentThread().getName());
            }
        };

        // Approach 2: Lambda expression (preferred)
        Runnable task2 = () -> System.out.println("Task 2 running in: " +
            Thread.currentThread().getName());

        // Create and start threads
        new Thread(task1, "Thread-1").start();
        new Thread(task2, "Thread-2").start();

        System.out.println("Main thread: " + Thread.currentThread().getName());
    }
}
```

**Expected Output (order may vary):**
```
Main thread: main
Task 1 running in: Thread-1
Task 2 running in: Thread-2
```

**Step-by-Step Explanation:**

1. `task1` is an anonymous `Runnable` implementation that prints the current thread's name.
2. `task2` is a lambda `Runnable` that does the same thing more concisely.
3. Two threads are created with the tasks and started.
4. The main thread prints its own name and continues.

### Real-World Cases

- **Event Handlers:** `button.addActionListener(e -> { /* handle click */ })` uses a functional interface similar to `Runnable`.
- **Background Tasks:** `executor.submit(() -> { /* task */ })` accepts a `Runnable`.
- **Thread Pools:** `ExecutorService.execute(Runnable)` executes tasks.

---

## 3. Thread Creation

### Definitions

**Core Definition:** Thread creation is the process of instantiating a `Thread` object with a task to be executed.

**Technical Definition:** Threads can be created by extending `Thread` or by implementing `Runnable` and passing it to a `Thread` constructor. "Thread defines public constructors for creating platform threads and the start method to schedule threads to execute. Thread may be extended for customization and other advanced reasons although most applications should have little need to do this". "Thread defines a Thread.Builder API for creating and starting both platform and virtual threads".

**Beginner-Friendly Explanation:** Creating a thread is like hiring a new worker. You can either hire someone who already knows the job (extend `Thread` and override `run()`), or you can write a job description (`Runnable`) and hire a general worker to follow it. The second approach is more flexible because the worker isn't defined by the job.

### Purposes

- To create a new thread of execution for a specific task
- To enable concurrent execution of multiple tasks
- To support different threading models (platform vs virtual)
- To provide flexibility through multiple creation approaches

### Syntax Rules and Structure

**Thread Creation Approaches:**

| Approach | Syntax | Pros | Cons |
|----------|--------|------|------|
| Extend `Thread` | `class MyThread extends Thread` | Simple | Cannot extend other classes |
| Implement `Runnable` | `new Thread(new MyTask())` | Flexible, functional | Slightly more verbose |
| Lambda `Runnable` | `new Thread(() -> {})` | Concise | Limited to simple tasks |
| `Thread.Builder` | `Thread.ofPlatform().start(...)` | Modern, named, flexible | Java 21+ |

**Syntax Rules:**

- The `Thread` constructor accepts a `Runnable` or a thread name.
- `Thread.Builder` allows configuration of name, daemon status, priority, and thread group.
- Virtual threads are created with `Thread.ofVirtual()`.

**Constraints and Limitations:**

- A thread can be started only once.
- Extending `Thread` prevents extending other classes.

### Annotated Complete Step-by-Step Code Examples

```java
// File: ThreadCreationDemo.java
public class ThreadCreationDemo {
    public static void main(String[] args) {
        // Approach 1: Extend Thread
        Thread t1 = new Thread() {
            @Override
            public void run() {
                System.out.println("Thread 1 (extended): " + getName());
            }
        };
        t1.start();

        // Approach 2: Implement Runnable
        Thread t2 = new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("Thread 2 (Runnable): " +
                    Thread.currentThread().getName());
            }
        }, "Runnable-Thread");
        t2.start();

        // Approach 3: Lambda
        Thread t3 = new Thread(() -> System.out.println("Thread 3 (lambda): " +
            Thread.currentThread().getName()), "Lambda-Thread");
        t3.start();

        // Approach 4: Thread.Builder (Java 21+)
        Thread t4 = Thread.ofPlatform().name("Builder-Thread")
            .start(() -> System.out.println("Thread 4 (builder): " +
                Thread.currentThread().getName()));
    }
}
```

**Expected Output (order may vary):**
```
Thread 1 (extended): Thread-0
Thread 2 (Runnable): Runnable-Thread
Thread 3 (lambda): Lambda-Thread
Thread 4 (builder): Builder-Thread
```

**Step-by-Step Explanation:**

1. Approach 1 uses an anonymous subclass of `Thread` with an overridden `run()`.
2. Approach 2 uses an anonymous `Runnable` implementation passed to a `Thread` constructor.
3. Approach 3 uses a lambda expression with a custom thread name.
4. Approach 4 uses `Thread.ofPlatform().name(...).start(...)` for a modern, fluent API.

### Real-World Cases

- **Web Servers:** Each request handled by a thread created from a pool.
- **Background Processing:** Threads created for logging, cleanup, and scheduled tasks.
- **Parallel Algorithms:** Multiple threads created to divide work.

---

## 4. Thread Start

### Definitions

**Core Definition:** `start()` is the method that schedules a thread to begin execution, causing the JVM to invoke the thread's `run()` method in a new thread of control.

**Technical Definition:** "Starting a thread schedules it to execute its run method. The newly started thread executes concurrently with the thread that caused it to start". "The start method can be called only once; calling it a second time throws an IllegalThreadStateException".

**Beginner-Friendly Explanation:** `start()` is like clocking in a worker. The worker doesn't begin until you press the clock-in button. Once clocked in, the worker starts doing their job (`run()`) while you continue with your own work. Without `start()`, the worker just stands there doing nothing.

### Purposes

- To schedule a thread for execution
- To begin concurrent execution of a task
- To transition a thread from the `NEW` state to `RUNNABLE`

### Syntax Rules and Structure

```java
Thread thread = new Thread(task);
thread.start();  // Schedules the thread
```

**Syntax Rules:**

- `start()` can be called only once per thread.
- Calling `start()` a second time throws `IllegalThreadStateException`.
- Calling `run()` directly executes the task in the current thread — it does not create a new thread.

**Constraints and Limitations:**

- A thread can be started only once.
- The order in which started threads execute is not deterministic.

### Annotated Complete Step-by-Step Code Examples

```java
// File: ThreadStartDemo.java
public class ThreadStartDemo {
    public static void main(String[] args) {
        Thread thread = new Thread(() -> {
            System.out.println("Thread running: " +
                Thread.currentThread().getName());
        }, "MyThread");

        System.out.println("Before start: " + thread.getState());  // NEW
        thread.start();
        System.out.println("After start: " + thread.getState());   // RUNNABLE
    }
}
```

**Expected Output (order may vary):**
```
Before start: NEW
After start: RUNNABLE
Thread running: MyThread
```

**Step-by-Step Explanation:**

1. A thread is created with a lambda task and a name.
2. Before `start()`, the thread is in the `NEW` state.
3. `start()` schedules the thread; it transitions to `RUNNABLE`.
4. The JVM invokes `run()` in the new thread, printing the message.

### Real-World Cases

- **Task Execution:** Starting threads to handle background jobs.
- **Parallel Processing:** Starting multiple threads to divide work.
- **Server Request Handling:** Starting a thread for each incoming connection.

---

## 5. Thread Join

### Definitions

**Core Definition:** `join()` is a method that causes the calling thread to wait until the target thread terminates.

**Technical Definition:** "The join method allows one thread to wait for the completion of another. If t is a Thread object whose thread is currently executing, t.join(); causes the current thread to pause execution until t's thread terminates. Overloads of join allow the programmer to specify a waiting period. However, as with sleep, join is dependent on the OS for timing, so you should not assume that join will wait exactly as long as you specify. Like sleep, join responds to an interrupt by exiting with an InterruptedException".

**Beginner-Friendly Explanation:** `join()` is like waiting for a colleague to finish their part of a project before you can continue with yours. You say "I'll wait here until you're done." This is useful when you need the results of another thread before proceeding.

### Purposes

- To synchronise the completion of multiple threads
- To ensure a thread has finished before proceeding
- To implement a simple form of coordination between threads
- To collect results from worker threads

### Syntax Rules and Structure

```java
// Wait indefinitely for the thread to die
targetThread.join();

// Wait at most 1000 milliseconds
targetThread.join(1000);

// Wait at most 1000 milliseconds + 500 nanoseconds
targetThread.join(1000, 500);
```

**Syntax Rules:**

- `join()` without arguments waits indefinitely.
- `join(long millis)` waits up to the specified milliseconds.
- `join(long millis, int nanos)` provides nanosecond precision.
- `join()` throws `InterruptedException` if the current thread is interrupted while waiting.

**Constraints and Limitations:**

- Timing is not precise; it depends on the OS scheduler.
- A thread that has already terminated returns immediately from `join()`.

### Annotated Complete Step-by-Step Code Examples

```java
// File: ThreadJoinDemo.java
public class ThreadJoinDemo {
    public static void main(String[] args) throws InterruptedException {
        Thread worker = new Thread(() -> {
            System.out.println("Worker started");
            try {
                Thread.sleep(2000);  // Simulate work
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
            System.out.println("Worker finished");
        }, "Worker");

        worker.start();

        System.out.println("Main: waiting for worker...");
        worker.join();  // Main thread waits here
        System.out.println("Main: worker has finished, proceeding");
    }
}
```

**Expected Output:**
```
Worker started
Main: waiting for worker...
Worker finished
Main: worker has finished, proceeding
```

**Step-by-Step Explanation:**

1. The worker thread starts and prints "Worker started".
2. The main thread prints "Main: waiting for worker..." and calls `worker.join()`.
3. The main thread blocks until the worker finishes (after 2 seconds).
4. The worker prints "Worker finished" and terminates.
5. `join()` returns, and the main thread continues.

### Real-World Cases

- **Result Collection:** Waiting for worker threads to finish computing results.
- **Sequential Dependencies:** Ensuring a thread completes before starting the next stage.
- **Testing:** Waiting for threads to finish before asserting results.

---

## 6. Thread Sleep

### Definitions

**Core Definition:** `Thread.sleep()` is a static method that causes the currently executing thread to pause its execution for a specified duration.

**Technical Definition:** "Thread.sleep causes the current thread to suspend execution for a specified period. This is an efficient means of making processor time available to the other threads of an application or other applications that might be running on a computer system. The sleep method can also be used for pacing... Two overloaded versions of sleep are provided: one that specifies the sleep time to the millisecond and one that specifies the sleep time to the nanosecond. However, these sleep times are not guaranteed to be precise, because they are limited by the facilities provided by the underlying OS. Also, the sleep period can be terminated by interrupts". "The thread does not lose ownership of any monitors".

**Beginner-Friendly Explanation:** `sleep()` is like taking a nap. You tell your worker to rest for a specified time. During the nap, the worker isn't using the CPU, so other workers can get more done. But the worker still holds onto any resources they were using (like locks). When the nap is over, the worker wakes up and continues.

### Purposes

- To pause the current thread for a specified duration
- To yield processor time to other threads
- To implement pacing (e.g., polling intervals)
- To simulate delays in testing

### Syntax Rules and Structure

```java
// Sleep for 1000 milliseconds (1 second)
Thread.sleep(1000);

// Sleep for 1000 milliseconds + 500 nanoseconds
Thread.sleep(1000, 500);

// With interrupt handling
try {
    Thread.sleep(1000);
} catch (InterruptedException e) {
    Thread.currentThread().interrupt();  // Restore interrupted status
}
```

**Syntax Rules:**

- `sleep()` is a static method — it always affects the current thread.
- `sleep()` throws `InterruptedException` if another thread interrupts the sleeping thread.
- "The interrupted status of the current thread is cleared when this exception is thrown".
- "The thread does not lose ownership of any monitors".

**Constraints and Limitations:**

- Sleep duration is not guaranteed to be precise; it depends on the OS timer.
- Sleep can be interrupted, so you must handle `InterruptedException`.
- Do not use `sleep()` for synchronisation — use `wait()`/`notify()` or higher-level constructs.

### Annotated Complete Step-by-Step Code Examples

```java
// File: ThreadSleepDemo.java
public class ThreadSleepDemo {
    public static void main(String[] args) throws InterruptedException {
        System.out.println("Starting...");

        for (int i = 1; i <= 3; i++) {
            System.out.println("Tick " + i);
            Thread.sleep(1000);  // Pause for 1 second
        }

        System.out.println("Done!");
    }
}
```

**Expected Output (with 1-second intervals):**
```
Starting...
Tick 1
Tick 2
Tick 3
Done!
```

**Step-by-Step Explanation:**

1. The main thread prints "Starting...".
2. In each iteration, it prints "Tick N" and then sleeps for 1000 ms.
3. During sleep, other threads (if any) can execute.
4. After three ticks, it prints "Done!".

### Real-World Cases

- **Polling:** Checking a resource periodically with a sleep between checks.
- **Rate Limiting:** Pacing API calls to avoid overwhelming a service.
- **Testing:** Simulating delays in tests.

---

## 7. Thread Interruption

### Definitions

**Core Definition:** Thread interruption is a cooperative mechanism for requesting that a thread stop what it is doing and terminate or handle the request appropriately.

**Technical Definition:** "Interrupting a thread sets its interrupt status. If the thread is blocked in a method such as sleep, wait, or join, the block is cleared and an InterruptedException is thrown. If the thread is blocked in an I/O operation or a Selector, the interrupt status is set and the operation returns immediately. If none of the previous conditions hold, the interrupt status is set". "Interrupting a thread that is not alive need not have any effect". "A thread interruption ignored because a thread was not alive at the time of the interrupt will be reflected by this method returning false".

**Beginner-Friendly Explanation:** Interruption is like tapping a colleague on the shoulder while they're working. You're not forcing them to stop — you're just letting them know they should stop when they can. If they're in the middle of a nap (`sleep()`), the tap wakes them up and they throw an `InterruptedException`. If they're doing something that can't be interrupted immediately, they finish what they're doing and then notice the tap.

### Purposes

- To request that a thread stop what it is doing
- To wake up a sleeping or waiting thread
- To implement graceful shutdown of threads
- To cancel long-running tasks

### Syntax Rules and Structure

```java
// Interrupt a thread
targetThread.interrupt();

// Check if the current thread is interrupted (clears flag)
boolean wasInterrupted = Thread.interrupted();

// Check if a thread is interrupted (does not clear flag)
boolean isInterrupted = targetThread.isInterrupted();

// Handle InterruptedException
try {
    Thread.sleep(1000);
} catch (InterruptedException e) {
    Thread.currentThread().interrupt();  // Restore interrupted status
    // Clean up and exit
}
```

**Key Methods:**

| Method | Description |
|--------|-------------|
| `interrupt()` | Sets the interrupt status of the thread |
| `interrupted()` | Tests and clears the current thread's interrupt status |
| `isInterrupted()` | Tests a thread's interrupt status without clearing |

**Syntax Rules:**

- `interrupt()` sets the interrupt flag; it does not forcibly stop the thread.
- When a thread is blocked in `sleep()`, `wait()`, or `join()`, interruption causes `InterruptedException`.
- When `InterruptedException` is caught, the interrupt status is cleared; call `Thread.currentThread().interrupt()` to restore it.

**Constraints and Limitations:**

- Interruption is cooperative; threads must check for it.
- "Do not use Thread.stop() to terminate threads" (SEI CERT).
- Tasks must support interruption to be cancellable in thread pools.

### Annotated Complete Step-by-Step Code Examples

```java
// File: ThreadInterruptDemo.java
public class ThreadInterruptDemo {
    public static void main(String[] args) throws InterruptedException {
        Thread worker = new Thread(() -> {
            try {
                System.out.println("Worker: sleeping for 10 seconds...");
                Thread.sleep(10000);  // Will be interrupted
                System.out.println("Worker: woke up normally");
            } catch (InterruptedException e) {
                System.out.println("Worker: interrupted!");
                Thread.currentThread().interrupt();  // Restore status
            }
            System.out.println("Worker: exiting");
        }, "Worker");

        worker.start();

        Thread.sleep(2000);  // Main thread waits 2 seconds
        System.out.println("Main: interrupting worker...");
        worker.interrupt();  // Interrupt the worker

        worker.join();
        System.out.println("Main: done");
    }
}
```

**Expected Output:**
```
Worker: sleeping for 10 seconds...
Main: interrupting worker...
Worker: interrupted!
Worker: exiting
Main: done
```

**Step-by-Step Explanation:**

1. The worker thread starts and calls `Thread.sleep(10000)`.
2. After 2 seconds, the main thread calls `worker.interrupt()`.
3. The worker's `sleep()` throws `InterruptedException`.
4. The worker catches the exception, prints "interrupted!", and restores the interrupt status.
5. The worker exits its `run()` method and terminates.
6. `worker.join()` returns, and the main thread prints "done".

### Real-World Cases

- **Graceful Shutdown:** Interrupting worker threads during application shutdown.
- **Task Cancellation:** Cancelling long-running tasks in thread pools.
- **Timeout Handling:** Interrupting a thread that exceeds a time limit.

---

## 8. Thread Priorities

### Definitions

**Core Definition:** Thread priority is a hint to the scheduler about the relative importance of a thread, represented as an integer from 1 (minimum) to 10 (maximum).

**Technical Definition:** "Every thread has a priority. Threads with higher priority are executed in preference to threads with lower priority. Each thread may or may not also be marked as a daemon. When code running in some thread creates a new Thread object, the new thread has its priority initially set equal to the priority of the creating thread". "The JVM defines a range of ten logical priorities for Java threads: MIN_PRIORITY = 1, NORM_PRIORITY = 5, MAX_PRIORITY = 10". "On Linux thread priorities are ignored because the OS does not support static priority in SCHED_OTHER scheduling class".

**Beginner-Friendly Explanation:** Thread priority is like setting the volume on different workers' voices. A higher priority means "I want to be heard more often." But the actual effect depends on the operating system — some systems listen carefully, others ignore it. You shouldn't rely on priorities to guarantee correct behaviour.

### Purposes

- To provide a hint to the scheduler about relative thread importance
- To influence scheduling decisions on platforms that support priorities
- To differentiate critical tasks from background tasks

### Syntax Rules and Structure

```java
Thread t = new Thread(task);
t.setPriority(Thread.MAX_PRIORITY);  // 10
int priority = t.getPriority();

// Constants
Thread.MIN_PRIORITY  // 1
Thread.NORM_PRIORITY // 5
Thread.MAX_PRIORITY  // 10
```

**Syntax Rules:**

- Priority must be between `MIN_PRIORITY` and `MAX_PRIORITY`; otherwise `IllegalArgumentException` is thrown.
- A new thread inherits the priority of its creating thread.
- Priority is a hint; its effect is platform-dependent.

**Constraints and Limitations:**

- "On Linux thread priorities are ignored".
- Do not rely on priorities for correctness.
- Priority inversion can occur when a high-priority thread waits for a low-priority thread.

### Annotated Complete Step-by-Step Code Examples

```java
// File: ThreadPriorityDemo.java
public class ThreadPriorityDemo {
    public static void main(String[] args) {
        Runnable task = () -> {
            for (int i = 0; i < 3; i++) {
                System.out.println(Thread.currentThread().getName() +
                    " (priority=" + Thread.currentThread().getPriority() + "): " + i);
            }
        };

        Thread low = new Thread(task, "Low");
        Thread normal = new Thread(task, "Normal");
        Thread high = new Thread(task, "High");

        low.setPriority(Thread.MIN_PRIORITY);
        normal.setPriority(Thread.NORM_PRIORITY);
        high.setPriority(Thread.MAX_PRIORITY);

        low.start();
        normal.start();
        high.start();
    }
}
```

**Expected Output (order may vary; priorities may have no effect on some platforms):**
```
High (priority=10): 0
Normal (priority=5): 0
Low (priority=1): 0
High (priority=10): 1
Normal (priority=5): 1
Low (priority=1): 1
High (priority=10): 2
Normal (priority=5): 2
Low (priority=1): 2
```

**Step-by-Step Explanation:**

1. Three threads are created with the same task.
2. `setPriority()` assigns different priorities to each thread.
3. The threads are started; the scheduler decides the execution order.
4. Higher-priority threads may run more often, but the exact behaviour depends on the OS.

### Real-World Cases

- **GUI Applications:** Giving the UI thread higher priority for responsiveness.
- **Background Tasks:** Using lower priorities for non-critical work.
- **Real-Time Systems:** Prioritising critical tasks (where the OS supports it).

---

## 9. Thread Daemon

### Definitions

**Core Definition:** A daemon thread is a thread that does not prevent the JVM from terminating when all non-daemon threads have finished.

**Technical Definition:** "The Java virtual machine continues to execute threads until either of the following occurs: the exit method of class Runtime has been called and the security manager has permitted the exit operation to take place; or all threads that are not daemon threads have died, either by returning from the call to the run method or by throwing an exception that propagates beyond the run method". "The shutdown sequence begins when all started non-daemon threads have terminated". "Virtual threads are daemon threads and so do not prevent the shutdown sequence from beginning".

**Beginner-Friendly Explanation:** A daemon thread is like a background helper that doesn't keep the office open. When all the regular employees (non-daemon threads) go home, the janitor (daemon thread) stops working too. The program ends, and the daemon thread is terminated automatically. Daemon threads are useful for background services like garbage collection or logging.

### Purposes

- To provide background services without preventing JVM shutdown
- To run tasks that should not keep the application alive
- To implement system-level services (e.g., garbage collection, finalisation)

### Syntax Rules and Structure

```java
Thread daemon = new Thread(task);
daemon.setDaemon(true);  // Must be called before start()
daemon.start();

boolean isDaemon = daemon.isDaemon();
```

**Syntax Rules:**

- `setDaemon(true)` must be called before `start()`; otherwise `IllegalThreadStateException` is thrown.
- A new thread inherits daemon status from its creating thread.
- "Virtual threads are daemon threads and so do not prevent the shutdown sequence from beginning".
- "The daemon status of a virtual thread is always true and cannot be changed to false".

**Constraints and Limitations:**

- Daemon threads are terminated abruptly when the JVM shuts down; they may not complete their work.
- "Daemon threads should NOT perform I/O or hold resources" (GitHub).
- Do not use daemon threads for tasks that must complete.

### Annotated Complete Step-by-Step Code Examples

```java
// File: ThreadDaemonDemo.java
public class ThreadDaemonDemo {
    public static void main(String[] args) {
        // Create a daemon thread
        Thread daemon = new Thread(() -> {
            while (true) {
                System.out.println("Daemon: still running...");
                try { Thread.sleep(1000); } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    break;
                }
            }
        }, "Daemon-Thread");

        daemon.setDaemon(true);  // Set before start
        daemon.start();

        System.out.println("Daemon: " + daemon.isDaemon());  // true

        // Main thread sleeps briefly, then exits
        try { Thread.sleep(3000); } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }

        System.out.println("Main thread exiting — daemon will be terminated");
    }
}
```

**Expected Output:**
```
Daemon: true
Daemon: still running...
Daemon: still running...
Daemon: still running...
Main thread exiting — daemon will be terminated
```

**Step-by-Step Explanation:**

1. A daemon thread is created with an infinite loop that prints a message every second.
2. `setDaemon(true)` is called before `start()`.
3. The daemon thread starts and prints its status.
4. The main thread sleeps for 3 seconds, allowing the daemon to run.
5. The main thread exits, and the JVM terminates the daemon thread.

### Real-World Cases

- **Garbage Collection:** JVM internal threads are daemon threads.
- **Background Logging:** A logging thread that writes to disk periodically.
- **Monitoring:** A thread that reports system metrics at intervals.

---

## 10. Thread Groups

### Definitions

**Core Definition:** A `ThreadGroup` represents a set of threads and can also include other thread groups, forming a tree structure.

**Technical Definition:** "A thread group represents a set of threads. In addition, a thread group can also include other thread groups. The thread groups form a tree in which every thread group except the initial thread group has a parent. A thread group has a name and maximum priority. The name is specified when creating the group and cannot be changed". "A thread group is weakly reachable from its parent group so that it is eligible for garbage collection when there are no live threads in the group and the thread group is otherwise unreachable". "The Java runtime creates a special thread group for virtual threads. This group is returned by the Thread.getThreadGroup method when invoked on a virtual thread".

**Beginner-Friendly Explanation:** A thread group is like a department in a company. It's a way to organise threads into related teams. You can perform operations on the entire group at once (like interrupting all threads in a group) and set a maximum priority for the group. However, thread groups are largely obsolete — modern applications should use thread pools and executors instead.

### Purposes

- To organise threads into logical groups
- To perform bulk operations (interrupt, enumerate) on a group
- To set a maximum priority for all threads in a group
- To provide a default uncaught exception handler for group members

### Syntax Rules and Structure

```java
// Create a thread group
ThreadGroup group = new ThreadGroup("MyGroup");

// Create a thread in the group
Thread t = new Thread(group, task, "Thread-1");

// Bulk operations
group.interrupt();  // Interrupt all threads in the group
int count = group.activeCount();

// Set maximum priority
group.setMaxPriority(Thread.MAX_PRIORITY);
```

**Key Methods:**

| Method | Description |
|--------|-------------|
| `ThreadGroup(String name)` | Creates a new thread group |
| `activeCount()` | Returns the estimated number of active threads |
| `interrupt()` | Interrupts all threads in the group |
| `setMaxPriority(int)` | Sets the maximum priority for the group |
| `getParent()` | Returns the parent group |

**Syntax Rules:**

- Every thread belongs to a thread group; if not specified, the current thread's group is used.
- The initial thread group has no parent.
- "Thread groups provided a way in early Java releases to group threads and provide a form of job control" (API Note).

**Constraints and Limitations:**

- "Do not invoke ThreadGroup methods" (SEI CERT THI01-J) — thread groups are largely obsolete and error-prone.
- Thread groups do not provide reliable bulk operation guarantees.
- Virtual threads are not included in `activeCount()` or `enumerate()`.

### Annotated Complete Step-by-Step Code Examples

```java
// File: ThreadGroupDemo.java
public class ThreadGroupDemo {
    public static void main(String[] args) {
        // Create a thread group
        ThreadGroup group = new ThreadGroup("Workers");

        // Create threads in the group
        Thread t1 = new Thread(group, () -> {
            System.out.println("Worker 1 in group: " +
                Thread.currentThread().getThreadGroup().getName());
        }, "Worker-1");

        Thread t2 = new Thread(group, () -> {
            System.out.println("Worker 2 in group: " +
                Thread.currentThread().getThreadGroup().getName());
        }, "Worker-2");

        t1.start();
        t2.start();

        // Group information
        System.out.println("Active threads: " + group.activeCount());
        System.out.println("Group name: " + group.getName());
        System.out.println("Max priority: " + group.getMaxPriority());
    }
}
```

**Expected Output (order may vary):**
```
Worker 1 in group: Workers
Worker 2 in group: Workers
Active threads: 2
Group name: Workers
Max priority: 10
```

**Step-by-Step Explanation:**

1. A `ThreadGroup` named `"Workers"` is created.
2. Two threads are created with the group as the first constructor argument.
3. The threads start and print their group name.
4. `activeCount()` returns the number of active threads in the group.

### Real-World Cases

- **Legacy Code:** Older applications may use thread groups; new code should use executors.
- **Debugging:** Thread dumps show thread groups for organisation.
- **Applet Sandboxing:** Thread groups were used in applets for security (obsolete).

---

## 11. Thread Naming

### Definitions

**Core Definition:** Thread naming is the practice of assigning descriptive names to threads for debugging, monitoring, and logging purposes.

**Technical Definition:** "Threads have a unique identifier and a name. The identifier is generated when a Thread is created and cannot be changed. The thread name can be specified when creating a thread or can be changed at a later time". "Platform threads get an automatically generated thread name by default. Virtual threads do not have a thread name by default; the getName method returns the empty string if a thread name is not set".

**Beginner-Friendly Explanation:** Naming threads is like giving employees name badges. Without names, all workers are just "Thread-145" — impossible to tell apart. With meaningful names like "OrderProcessor" or "HttpListener", you can instantly see what each thread is doing in logs and debuggers.

### Purposes

- To identify threads in logs, debuggers, and thread dumps
- To distinguish between different types of worker threads
- To simplify debugging and performance analysis
- To provide context in error messages

### Syntax Rules and Structure

```java
// Set name in constructor
Thread t = new Thread(task, "Worker-1");

// Set or change name later
t.setName("NewName");

// Get name
String name = t.getName();
```

**Syntax Rules:**

- Thread names are not required to be unique.
- "More than one thread may have the same name. Giving multiple threads the same name may make things worse and cause confusion instead of helping debugging".
- Virtual threads have no default name; `getName()` returns an empty string.

**Naming Conventions:**

| Convention | Example | Use Case |
|------------|---------|----------|
| Component-based | `http-listener-1` | Server components |
| Pool-based | `pool-1-thread-3` | Thread pools |
| Task-based | `order-processor-1` | Task-specific workers |
| Module-based | `billing-worker-1` | Module-specific threads |

**Constraints and Limitations:**

- Names should be short but meaningful.
- Avoid extremely long names with irrelevant clauses.
- "Put the most significant part first" — e.g., `Listener-http-01` rather than `01-http-Listener`.

### Annotated Complete Step-by-Step Code Examples

```java
// File: ThreadNamingDemo.java
public class ThreadNamingDemo {
    public static void main(String[] args) {
        // Named thread via constructor
        Thread t1 = new Thread(() -> {
            System.out.println("Running: " + Thread.currentThread().getName());
        }, "OrderProcessor-1");

        // Named thread via Thread.Builder (Java 21+)
        Thread t2 = Thread.ofPlatform().name("HttpListener-1")
            .start(() -> {
                System.out.println("Running: " + Thread.currentThread().getName());
            });

        // Rename a thread
        Thread t3 = new Thread(() -> {
            System.out.println("Running: " + Thread.currentThread().getName());
        }, "TempName");
        t3.setName("DatabaseWorker-1");

        t1.start();
        t3.start();
    }
}
```

**Expected Output (order may vary):**
```
Running: OrderProcessor-1
Running: DatabaseWorker-1
Running: HttpListener-1
```

**Step-by-Step Explanation:**

1. `t1` is created with the name `"OrderProcessor-1"`.
2. `t2` uses `Thread.ofPlatform().name(...)` for a fluent naming API.
3. `t3` is initially named `"TempName"` but renamed to `"DatabaseWorker-1"` before starting.
4. Each thread prints its name when running.

### Real-World Cases

- **Web Servers:** Naming threads after the request type (`http-worker-1`, `admin-worker-1`).
- **Thread Pools:** Using `ThreadFactory` to name pool threads (`pool-1-thread-1`).
- **Logging:** Identifying which thread produced a log entry.

---

## 12. Thread Best Practices

### Definitions

**Core Definition:** Thread best practices are guidelines for creating, managing, and using threads safely, efficiently, and maintainably.

**Technical Definition:** Best practices derive from authoritative sources including the SEI CERT Oracle Coding Standard for Java. Key principles include preferring `Runnable` over extending `Thread`, using thread pools instead of creating threads ad-hoc, ensuring tasks are interruptible, and avoiding deprecated methods like `Thread.stop()`.

**Beginner-Friendly Explanation:** Thread best practices help you avoid common pitfalls like deadlocks, resource exhaustion, and unpredictable behaviour. They're like a safety manual for multithreaded programming.

### Purposes

- To prevent deadlocks, race conditions, and resource exhaustion
- To ensure correct and predictable concurrent behaviour
- To improve performance and scalability
- To write maintainable concurrent code

### Key Best Practices Table

| Practice | Description | Source |
|----------|-------------|--------|
| Prefer `Runnable` over extending `Thread` | "Do not subclass Thread if you can use a Runnable instead". | SEI CERT |
| Use thread pools | "Facilitate thread reuse by using Thread Pools". | SEI CERT |
| Ensure tasks are interruptible | "Ensure that tasks submitted to a thread pool are interruptible". | SEI CERT |
| Do not use `Thread.stop()` | "Do not use Thread.stop() to terminate threads". | SEI CERT |
| Name threads meaningfully | Choose short, meaningful, unique names for debugging. | Community |
| Use `join()` for synchronisation | "The join method allows one thread to wait for the completion of another". | Oracle |
| Restore interrupt status | Call `Thread.currentThread().interrupt()` when catching `InterruptedException`. | Best practice |
| Avoid `Thread.sleep()` for synchronisation | Use `wait()`/`notify()` or higher-level constructs. | Best practice |
| Use daemon threads for background services | Daemon threads do not prevent JVM shutdown. | Oracle |
| Avoid thread groups | "Do not invoke ThreadGroup methods". | SEI CERT |
| Set daemon status before start | `setDaemon()` must be called before `start()`. | Oracle |
| Use `Thread.Builder` (Java 21+) | Modern API for creating platform and virtual threads. | Oracle |

### Annotated Complete Step-by-Step Code Examples

```java
// File: ThreadBestPracticesDemo.java
public class ThreadBestPracticesDemo {
    public static void main(String[] args) throws InterruptedException {
        // BEST PRACTICE: Use Runnable, not Thread subclassing
        Runnable task = () -> {
            try {
                System.out.println(Thread.currentThread().getName() +
                    " is working");
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                // BEST PRACTICE: Restore interrupt status
                Thread.currentThread().interrupt();
                System.out.println(Thread.currentThread().getName() +
                    " was interrupted");
            }
            System.out.println(Thread.currentThread().getName() + " finished");
        };

        // BEST PRACTICE: Name threads meaningfully
        Thread worker = new Thread(task, "DataProcessor-1");
        worker.start();

        // BEST PRACTICE: Use join for synchronisation
        worker.join();

        System.out.println("Main: all workers completed");
    }
}
```

**Expected Output:**
```
DataProcessor-1 is working
DataProcessor-1 finished
Main: all workers completed
```

**Step-by-Step Explanation:**

1. A `Runnable` task is defined instead of extending `Thread`.
2. The thread is named `"DataProcessor-1"` for clarity.
3. `InterruptedException` is caught and the interrupt status is restored.
4. `join()` is used to wait for the worker to complete before proceeding.

### Real-World Cases

- **Web Servers:** Thread pools handle requests without creating threads ad-hoc.
- **Background Processing:** Named daemon threads handle logging and cleanup.
- **Task Cancellation:** Interruptible tasks respond to shutdown requests.

---

## References Links

- Class Thread — Java Platform SE 27 API Specification - https://docs.oracle.com/en/java/javase/27/docs/api/java.base/java/lang/Thread.html
- Interface Runnable — Java Platform SE 8 API Specification - https://docs.oracle.com/javase/8/docs/api/java/lang/Runnable.html
- Enum Thread.State — Java Platform SE 11 API Specification - https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Thread.State.html
- Pausing Execution with Sleep — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/essential/concurrency/sleep.html
- Joins — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/essential/concurrency/join.html
- Class ThreadGroup — Java Platform SE 21 API Specification - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/ThreadGroup.html
- 13. Thread APIs (THI) — SEI CERT Oracle Coding Standard for Java - https://wiki.sei.cmu.edu/confluence/pages/viewpage.action?pageId=88491697
- CON31-J. Ensure that tasks submitted to a thread pool are interruptible — SEI CERT - https://wiki.sei.cmu.edu/confluence/display/java/CON31-J.+Ensure+that+tasks+submitted+to+a+thread+pool+are+interruptible
- Java Thread.interrupt() Method — OpenJDK Source — https://cr.openjdk.org/~alanb/8198841/webrev/src/java.base/share/classes/java/lang/Thread.java.sdiff.html
- Should threads in Java be named for easier debugging? — Stack Overflow - https://stackoverflow.com/questions/3157852/should-threads-in-java-be-named-for-easier-debugging
- Daemon Threads in Java — Baeldung - https://www.baeldung.com/java-daemon-thread