# Java Concurrency Fundamentals: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Concurrency is the ability of a program to execute multiple tasks in overlapping time periods, while parallelism is the simultaneous execution of multiple tasks on multiple processing units. Java provides first-class support for concurrency through its threading model, built into the `java.lang.Thread` class and the `java.util.concurrent` package.

**Technical Definition:** "A thread is a thread of execution in a program. The Java Virtual Machine allows an application to have multiple threads of execution running concurrently". "A process has a self-contained execution environment. A process generally has a complete, private set of basic run-time resources; in particular, each process has its own memory space". "Threads are sometimes called lightweight processes. Both processes and threads provide an execution environment, but creating a new thread requires fewer resources than creating a new process. Threads exist within a process — every process has at least one. Threads share the process's resources, including memory and open files".

**Beginner-Friendly Explanation:** Imagine a restaurant kitchen. A **process** is like the entire kitchen — it has its own space, equipment, and staff. A **thread** is like an individual chef working in that kitchen. Multiple chefs (threads) can work simultaneously, sharing the same kitchen space (memory), but each doing their own task. **Concurrency** is when the chefs take turns using the stove, giving the impression that everything is happening at once. **Parallelism** is when there are multiple stoves, so chefs can actually cook at the same time. Most Java programs start with one chef (the main thread), but you can hire more chefs (create threads) to get more work done.

### Key Characteristics

- **Two Units of Execution:** Processes (heavyweight, isolated) and threads (lightweight, shared memory).
- **Shared Memory Model:** Threads within a process share the same address space, enabling efficient communication but requiring synchronization.
- **Preemptive Scheduling:** The operating system (or JVM) decides when to switch between threads.
- **Concurrency ≠ Parallelism:** Concurrency is about dealing with many things at once; parallelism is about doing many things at once.
- **Platform vs Virtual Threads:** Platform threads are mapped 1:1 to OS threads; virtual threads (Java 21+) are lightweight, JVM-managed threads.
- **Java Memory Model (JMM):** Defines the rules for how threads interact through memory, ensuring visibility and ordering.

### Prerequisites

- Basic understanding of Java classes and objects
- Familiarity with the `java.lang` package
- Knowledge of exceptions and try-catch-finally
- Understanding of basic operating system concepts (processes, scheduling)

### Related Programming Areas

- **Multithreading:** Creating and managing multiple threads of execution.
- **Synchronization:** Coordinating access to shared resources.
- **Thread Pools:** Reusing threads for efficient task execution.
- **Concurrent Collections:** Thread-safe data structures in `java.util.concurrent`.
- **Asynchronous Programming:** `CompletableFuture` and reactive programming.
- **Virtual Threads (Project Loom):** Lightweight threads for high-throughput concurrency.

### Core Concepts / Features

---

## 1. Concurrency

### Definitions

**Core Definition:** Concurrency is the ability of a program to execute multiple tasks in overlapping time periods, where tasks may start, run, and complete in interleaved fashion.

**Technical Definition:** "Concurrency, in contrast, is the problem of scheduling some computational resources to a set of largely independent tasks that compete over those resources. The main performance measure we care about when we speak of concurrency isn't the duration (latency) of any single task, but throughput — the number of tasks we can process per time-unit". "Concurrency is the simultaneous execution of multiple threads of execution". "Concurrency uses multi-threading in order to start several execution units, called threads".

**Beginner-Friendly Explanation:** Concurrency is like a single chef juggling multiple dishes. The chef chops vegetables for one dish, then stirs the pot for another, then checks the oven for a third. The chef isn't doing all three at exactly the same instant, but all three dishes are progressing. This is what concurrency looks like — multiple tasks making progress by sharing time.

### Purposes

- To improve responsiveness in interactive applications
- To increase throughput by overlapping I/O and computation
- To model real-world systems that naturally have multiple activities
- To make better use of multi-core processors
- To handle multiple clients or requests simultaneously

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

// Approach 2: Implement Runnable
Runnable task = () -> {
    // task code
};
new Thread(task).start();

// Approach 3: Using Thread.Builder (Java 21+)
Thread thread = Thread.ofPlatform().start(() -> {
    // task code
});
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Thread` | The class representing a thread of execution |
| `Runnable` | A functional interface for the task to be executed |
| `start()` | Schedules the thread to begin execution |
| `run()` | The method containing the thread's task |
| `Thread.Builder` | Modern API for creating threads (Java 21+) |

**Syntax Rules:**

- `start()` schedules the thread; `run()` is called by the JVM when the thread executes.
- A thread terminates when `run()` completes normally or abruptly.
- Threads share the process's memory and resources.

**Constraints and Limitations:**

- Creating a new thread for every task has significant overhead.
- Excessive threads cause resource exhaustion and `OutOfMemoryError`.
- Thread scheduling and context switching introduce overhead.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Creating and Starting Threads**

```java
// File: ConcurrencyDemo.java
public class ConcurrencyDemo {
    public static void main(String[] args) {
        // Approach 1: Using a lambda with Runnable
        Runnable task = () -> {
            for (int i = 0; i < 3; i++) {
                System.out.println(Thread.currentThread().getName() + ": " + i);
            }
        };

        // Create two threads that run the same task
        Thread t1 = new Thread(task, "Thread-A");
        Thread t2 = new Thread(task, "Thread-B");

        // Start both threads
        t1.start();
        t2.start();

        System.out.println("Main thread continues...");
    }
}
```

**Expected Output (order of thread output may vary):**
```
Main thread continues...
Thread-A: 0
Thread-B: 0
Thread-A: 1
Thread-B: 1
Thread-A: 2
Thread-B: 2
```

**Step-by-Step Explanation:**

1. A `Runnable` is defined with a lambda that prints the current thread's name and a counter.
2. Two `Thread` objects are created with the same `Runnable` but different names.
3. `start()` is called on both threads, scheduling them for execution.
4. The main thread continues immediately after calling `start()` (it does not wait).
5. Both threads execute the `run()` method concurrently, interleaving their output.

### Real-World Cases

- **Web Servers:** Handling multiple HTTP requests concurrently.
- **GUI Applications:** Keeping the UI responsive while background tasks run.
- **Data Processing:** Overlapping I/O operations with computation.
- **Game Engines:** Managing rendering, physics, and input in parallel.

---

## 2. Parallelism

### Definitions

**Core Definition:** Parallelism is the simultaneous execution of multiple tasks on multiple processing units (CPU cores), where tasks literally run at the same instant.

**Technical Definition:** "Parallelism is the problem of doing a single job — say, inverting a matrix or sorting a list — faster by employing multiple processing units. This is done by breaking up the job into multiple cooperating subtasks, different subsets of which running on a different core". "Parallelism implies doing many things at once". "Parallelism is when different hardware units (e.g., cores) are doing work at the same time".

**Beginner-Friendly Explanation:** Parallelism is like having multiple chefs, each with their own stove, all cooking different dishes at exactly the same time. Unlike concurrency (one chef juggling tasks), parallelism requires multiple processing units. On a multi-core CPU, different threads can truly run simultaneously on different cores.

### Purposes

- To reduce latency of a single large computation
- To exploit multi-core processors for performance
- To speed up CPU-bound tasks
- To divide a large job into cooperating subtasks

### Syntax Rules and Structure

**Key Distinction:**

| Aspect | Concurrency | Parallelism |
|--------|-------------|-------------|
| Goal | Handle many tasks (throughput) | Do one job faster (latency) |
| Tasks | Largely independent, competing | Cooperating subtasks of one job |
| Hardware | Can exist on a single core | Requires multiple cores |
| Measure | Throughput | Latency reduction |
| Example | Web server handling requests | Sorting a huge list |

### Annotated Complete Step-by-Step Code Examples

```java
// File: ParallelismDemo.java
import java.util.stream.IntStream;

public class ParallelismDemo {
    public static void main(String[] args) {
        int n = 10_000_000;
        int[] numbers = IntStream.rangeClosed(1, n).toArray();

        // Sequential sum
        long start = System.nanoTime();
        long sequentialSum = 0;
        for (int num : numbers) {
            sequentialSum += num;
        }
        long seqTime = System.nanoTime() - start;

        // Parallel sum using parallel streams
        start = System.nanoTime();
        long parallelSum = IntStream.of(numbers).parallel().sum();
        long parTime = System.nanoTime() - start;

        System.out.println("Sequential: " + sequentialSum + " (" + seqTime / 1_000_000 + " ms)");
        System.out.println("Parallel:   " + parallelSum + " (" + parTime / 1_000_000 + " ms)");
    }
}
```

**Expected Output (times vary):**
```
Sequential: 50000005000000 (45 ms)
Parallel:   50000005000000 (12 ms)
```

**Step-by-Step Explanation:**

1. An array of 10 million integers is created.
2. The sequential sum iterates over the array using a loop.
3. The parallel sum uses `IntStream.parallel()` to divide the work across cores.
4. The parallel version is typically faster because multiple cores work simultaneously.
5. The result is the same, but the execution time differs due to parallelism.

### Real-World Cases

- **Scientific Computing:** Matrix multiplication, Monte Carlo simulations.
- **Image Processing:** Applying filters to large images in parallel.
- **Big Data:** MapReduce operations across distributed clusters.
- **Machine Learning:** Parallel gradient descent, batch processing.

---

## 3. Processes

### Definitions

**Core Definition:** A process is an instance of a running program that has its own isolated memory space and system resources.

**Technical Definition:** "A process has a self-contained execution environment. A process generally has a complete, private set of basic run-time resources; in particular, each process has its own memory space". "Processes are independent execution units that contain their own state information, use their own address spaces, and only interact via interprocess communication". "Most implementations of the Java virtual machine run as a single process".

**Beginner-Friendly Explanation:** A process is like a separate house. It has its own address (memory space), its own utilities, and its own front door. Processes cannot directly access each other's memory — they must communicate through special channels (pipes, sockets). In Java, when you run a program, the JVM itself runs as a single process, and your Java code runs inside it.

### Purposes

- To provide isolation between different applications
- To enable multiple programs to run simultaneously
- To allocate separate memory spaces for security and stability
- To support inter-process communication (IPC) when needed

### Syntax Rules and Structure

**Process vs Thread Comparison:**

| Aspect | Process | Thread |
|--------|---------|--------|
| Memory | Own address space | Shared with other threads |
| Creation cost | High | Low |
| Communication | IPC (pipes, sockets) | Shared memory |
| Isolation | Strong | None (shared memory) |
| Crash impact | Isolated | Can crash entire process |

**Constraints and Limitations:**

- Creating a process is expensive (requires a new JVM if Java).
- Inter-process communication is slower than thread communication.
- Java applications can create additional processes using `ProcessBuilder`.

### Annotated Complete Step-by-Step Code Examples

```java
// File: ProcessDemo.java
import java.io.IOException;

public class ProcessDemo {
    public static void main(String[] args) throws IOException, InterruptedException {
        // Create a new process (runs the OS 'echo' command)
        ProcessBuilder pb = new ProcessBuilder("echo", "Hello from a separate process!");
        Process process = pb.start();

        // Read the output of the process
        process.getInputStream().transferTo(System.out);

        // Wait for the process to finish
        int exitCode = process.waitFor();
        System.out.println("\nProcess exited with code: " + exitCode);

        // Information about the current JVM process
        ProcessHandle current = ProcessHandle.current();
        System.out.println("Current process PID: " + current.pid());
    }
}
```

**Expected Output:**
```
Hello from a separate process!
Process exited with code: 0
Current process PID: 12345
```

**Step-by-Step Explanation:**

1. `ProcessBuilder` is created with the `echo` command and an argument.
2. `pb.start()` launches a new OS process.
3. `getInputStream().transferTo()` reads the new process's output.
4. `waitFor()` waits for the process to finish and returns its exit code.
5. `ProcessHandle.current()` provides information about the JVM's own process.

### Real-World Cases

- **Running External Tools:** Invoking compilers, scripts, or system commands.
- **Microservices:** Independent processes communicating over the network.
- **Sandboxing:** Isolating untrusted code in separate processes.

---

## 4. Threads

### Definitions

**Core Definition:** A thread is the smallest unit of execution within a process, sharing the process's memory and resources with other threads.

**Technical Definition:** "A thread is a thread of execution in a program. The Java virtual machine allows an application to have multiple threads of execution running concurrently". "Threads are sometimes called lightweight processes... Threads share the process's resources, including memory and open files". "Every application has at least one thread — the main thread".

**Beginner-Friendly Explanation:** A thread is like a worker inside a factory (the process). Multiple workers share the same factory floor (memory), tools, and materials. They can work on different tasks simultaneously, but they must coordinate when using shared resources.

### Purposes

- To execute multiple tasks concurrently within a single process
- To keep applications responsive
- To improve performance through overlapping I/O and computation
- To take advantage of multi-core processors
- To model naturally concurrent activities

### Syntax Rules and Structure

**Complete General Syntax:**

```java
// Approach 1: Extend Thread
public class MyThread extends Thread {
    @Override
    public void run() {
        // task code
    }
}

// Approach 2: Implement Runnable
public class MyTask implements Runnable {
    @Override
    public void run() {
        // task code
    }
}
new Thread(new MyTask()).start();

// Approach 3: Lambda with Runnable
new Thread(() -> { /* task code */ }).start();

// Approach 4: Thread.Builder (Java 21+)
Thread thread = Thread.ofVirtual().name("my-thread").start(() -> { /* task */ });
```

**Key Thread Methods:**

| Method | Description |
|--------|-------------|
| `start()` | Schedules the thread to begin execution |
| `run()` | Contains the thread's task |
| `join()` | Waits for the thread to terminate |
| `isAlive()` | Tests if a started thread has terminated |
| `getName()` | Returns the thread's name |
| `setName(String)` | Changes the thread's name |
| `getId()` | Returns the unique thread identifier |
| `sleep(long)` | Pauses the thread for the specified milliseconds |
| `interrupt()` | Interrupts the thread |
| `currentThread()` | Returns the currently executing thread |

**Syntax Rules:**

- A thread is created but not started until `start()` is called.
- `start()` schedules the thread; the JVM calls `run()` when the thread executes.
- A thread terminates when `run()` completes normally or abruptly.
- Platform threads are mapped 1:1 to kernel threads; virtual threads are JVM-managed.

**Constraints and Limitations:**

- Platform threads are a limited resource (typically thousands at most).
- Virtual threads are suitable for I/O-bound tasks, not CPU-intensive operations.
- Threads share memory, requiring synchronization to avoid race conditions.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Thread Lifecycle and Methods**

```java
// File: ThreadDemo.java
public class ThreadDemo {
    public static void main(String[] args) throws InterruptedException {
        Thread worker = new Thread(() -> {
            System.out.println("Worker thread running: " + Thread.currentThread().getName());
            try {
                Thread.sleep(1000);  // Simulate work
            } catch (InterruptedException e) {
                System.out.println("Worker interrupted");
                Thread.currentThread().interrupt();  // Restore interrupt status
            }
            System.out.println("Worker thread finished");
        }, "Worker-1");

        System.out.println("Before start: " + worker.getState());  // NEW
        worker.start();
        System.out.println("After start: " + worker.getState());   // RUNNABLE or TIMED_WAITING

        worker.join();  // Wait for the worker to finish
        System.out.println("After join: " + worker.getState());    // TERMINATED
        System.out.println("Main thread finished");
    }
}
```

**Expected Output:**
```
Before start: NEW
After start: RUNNABLE
Worker thread running: Worker-1
Worker thread finished
After join: TERMINATED
Main thread finished
```

**Step-by-Step Explanation:**

1. A `Thread` is created with a lambda `Runnable` and a name `"Worker-1"`.
2. Before `start()`, the thread is in the `NEW` state.
3. After `start()`, the thread transitions to `RUNNABLE` and begins execution.
4. The worker sleeps for 1000 ms, entering the `TIMED_WAITING` state.
5. `worker.join()` blocks the main thread until the worker terminates.
6. After completion, the worker is in the `TERMINATED` state.

### Real-World Cases

- **Web Servers:** Each request handled by a separate thread.
- **Background Tasks:** Logging, cleanup, and scheduled jobs.
- **GUI Event Handling:** The Event Dispatch Thread processes UI events.
- **Parallel Algorithms:** Dividing work across multiple threads.

---

## 5. Thread Lifecycle

### Definitions

**Core Definition:** The thread lifecycle is the sequence of states a thread passes through from creation to termination.

**Technical Definition:** "A thread can be in one of the following states: NEW, RUNNABLE, BLOCKED, WAITING, TIMED_WAITING, TERMINATED". "A thread can be in only one state at a given point in time. These states are virtual machine states which do not reflect any operating system thread states".

**Beginner-Friendly Explanation:** A thread's life is like a person's day. It starts as NEW (born, not yet started), becomes RUNNABLE (ready to work), may become BLOCKED (waiting for a resource), WAITING (waiting indefinitely), TIMED_WAITING (waiting with a timeout), and finally TERMINATED (finished).

### Purposes

- To understand how threads progress through their existence
- To diagnose thread-related issues (deadlocks, starvation)
- To write correct code that handles thread state transitions
- To use `Thread.getState()` for monitoring and debugging

### Syntax Rules and Structure

**Thread States Table:**

| State | Description | How It's Entered |
|-------|-------------|------------------|
| `NEW` | Thread created but not started | `new Thread(...)` |
| `RUNNABLE` | Executing or ready to execute | `start()` called |
| `BLOCKED` | Waiting for a monitor lock | Waiting to enter `synchronized` |
| `WAITING` | Waiting indefinitely for another thread | `wait()`, `join()` without timeout |
| `TIMED_WAITING` | Waiting with a timeout | `sleep(ms)`, `wait(ms)`, `join(ms)` |
| `TERMINATED` | Thread has exited | `run()` completes |

**Syntax Rules:**

- Only one state at a time.
- JVM states do not map directly to OS thread states.
- Use `Thread.getState()` to query the current state.

**State Transition Diagram:**

```
NEW → start() → RUNNABLE → (scheduling) → RUNNABLE
                    ↓
              (lock contention) → BLOCKED → (lock acquired) → RUNNABLE
                    ↓
              (wait/join) → WAITING → (notify/join complete) → RUNNABLE
                    ↓
              (sleep/wait with timeout) → TIMED_WAITING → (timeout/notify) → RUNNABLE
                    ↓
              (run completes) → TERMINATED
```

### Annotated Complete Step-by-Step Code Examples

```java
// File: ThreadStateDemo.java
public class ThreadStateDemo {
    public static void main(String[] args) throws InterruptedException {
        Object lock = new Object();

        Thread t = new Thread(() -> {
            synchronized (lock) {
                try {
                    System.out.println("Thread is in BLOCKED/WAITING state");
                    lock.wait(2000);  // TIMED_WAITING
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            }
            System.out.println("Thread finished");
        }, "StateDemoThread");

        System.out.println("1. After creation: " + t.getState());  // NEW

        t.start();
        Thread.sleep(100);  // Give the thread time to run
        System.out.println("2. After start: " + t.getState());     // RUNNABLE or TIMED_WAITING

        t.join();
        System.out.println("3. After completion: " + t.getState()); // TERMINATED
    }
}
```

**Expected Output:**
```
1. After creation: NEW
Thread is in BLOCKED/WAITING state
2. After start: TIMED_WAITING
Thread finished
3. After completion: TERMINATED
```

**Step-by-Step Explanation:**

1. The thread is created but not started: state is `NEW`.
2. After `start()`, the thread enters `RUNNABLE` and acquires the lock.
3. `lock.wait(2000)` puts the thread into `TIMED_WAITING`.
4. After the wait completes, the thread finishes and enters `TERMINATED`.
5. `t.join()` waits for the thread to terminate.

### Real-World Cases

- **Debugging:** Diagnosing stuck threads via thread dumps.
- **Monitoring:** Tracking the number of runnable vs blocked threads.
- **Testing:** Verifying that a thread reaches a specific state.

---

## 6. Thread States

### Definitions

**Core Definition:** Thread states are the distinct conditions a thread can be in at any moment, as defined by the `Thread.State` enum.

**Technical Definition:** The `Thread.State` enum defines six states: `NEW`, `RUNNABLE`, `BLOCKED`, `WAITING`, `TIMED_WAITING`, and `TERMINATED`. "A thread in the runnable state is executing in the Java virtual machine but it may be waiting for other resources from the operating system such as processor". "A thread in the blocked state is waiting for a monitor lock to enter a synchronized block/method".

**Beginner-Friendly Explanation:** Thread states are like traffic lights for threads. `NEW` means the car hasn't started. `RUNNABLE` means the car is moving or ready to move. `BLOCKED` means the car is waiting at a red light (waiting for a lock). `WAITING` means the car is parked indefinitely. `TIMED_WAITING` means the car is parked with a timer. `TERMINATED` means the car has reached its destination.

### Purposes

- To understand exactly what a thread is doing at any moment
- To diagnose performance issues (e.g., many threads in BLOCKED state)
- To write correct synchronization code
- To use `Thread.getState()` for monitoring

### Syntax Rules and Structure

**State Details:**

| State | Meaning | Common Causes |
|-------|---------|---------------|
| `NEW` | Not yet started | Just created |
| `RUNNABLE` | Executing or ready to execute | Running, or waiting for CPU |
| `BLOCKED` | Waiting for a monitor lock | Contending for `synchronized` |
| `WAITING` | Waiting indefinitely | `wait()`, `join()`, `park()` |
| `TIMED_WAITING` | Waiting with timeout | `sleep()`, `wait(ms)`, `join(ms)` |
| `TERMINATED` | Exited | `run()` completed |

**Syntax Rules:**

- Use `Thread.getState()` to query the state.
- States are JVM-level, not OS-level.
- A thread can transition between states multiple times.

### Annotated Complete Step-by-Step Code Examples

```java
// File: ThreadStatesDemo.java
public class ThreadStatesDemo {
    public static void main(String[] args) throws InterruptedException {
        Object lock = new Object();

        Thread t = new Thread(() -> {
            synchronized (lock) {
                try {
                    lock.wait();  // WAITING
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            }
        }, "StatesDemo");

        // NEW
        System.out.println("Before start: " + t.getState());

        t.start();
        Thread.sleep(100);

        // RUNNABLE or WAITING
        System.out.println("During wait: " + t.getState());

        // Wake up the thread
        synchronized (lock) {
            lock.notify();
        }
        t.join();

        // TERMINATED
        System.out.println("After completion: " + t.getState());
    }
}
```

**Expected Output:**
```
Before start: NEW
During wait: WAITING
After completion: TERMINATED
```

**Step-by-Step Explanation:**

1. The thread is created: `NEW`.
2. `start()` is called: the thread enters `RUNNABLE`.
3. `lock.wait()` puts the thread into `WAITING`.
4. `lock.notify()` wakes the thread, returning it to `RUNNABLE`.
5. The thread completes: `TERMINATED`.

### Real-World Cases

- **Thread Dumps:** Analyzing states to find deadlocks (threads in `BLOCKED`).
- **Performance Tuning:** Reducing time spent in `BLOCKED` or `WAITING`.
- **Testing:** Asserting that a thread reaches a specific state.

---

## 7. Thread Scheduling

### Definitions

**Core Definition:** Thread scheduling is the process by which the operating system or JVM decides which thread to execute next and for how long.

**Technical Definition:** "The JVM defines a range of ten logical priorities for Java threads, including: `java.lang.Thread.MIN_PRIORITY = 1`, `java.lang.Thread.NORM_PRIORITY = 5`". "On Linux thread priorities are ignored because the OS does not support static priority in `SCHED_OTHER` scheduling class". "Embodiments of the Java thread scheduler implement a high-level thread scheduling policy (e.g., round robin, priority-based, and/or negotiated multimedia)".

**Beginner-Friendly Explanation:** Thread scheduling is like a referee deciding which player gets the ball. The referee can use different strategies — round-robin (everyone gets equal turns), priority-based (important players get more turns), or others. In Java, the JVM often delegates scheduling to the operating system, which means Java's thread priorities are sometimes just hints.

### Purposes

- To fairly allocate CPU time among threads
- To prioritise important tasks
- To maximise system throughput and responsiveness
- To provide predictable behaviour

### Syntax Rules and Structure

**Thread Priorities:**

| Priority | Constant | Value |
|----------|----------|-------|
| Minimum | `Thread.MIN_PRIORITY` | 1 |
| Normal | `Thread.NORM_PRIORITY` | 5 |
| Maximum | `Thread.MAX_PRIORITY` | 10 |

**Setting Priority:**

```java
Thread t = new Thread(task);
t.setPriority(Thread.MAX_PRIORITY);
```

**Syntax Rules:**

- Priorities are hints; their effect is platform-dependent.
- The JVM may ignore priorities on some operating systems.
- Higher-priority threads are generally scheduled before lower-priority threads, but this is not guaranteed.

**Constraints and Limitations:**

- "On Linux thread priorities are ignored".
- Relying on priorities for correctness is an anti-pattern.
- Starvation can occur if high-priority threads monopolise the CPU.

### Annotated Complete Step-by-Step Code Examples

```java
// File: SchedulingDemo.java
public class SchedulingDemo {
    public static void main(String[] args) {
        Runnable task = () -> {
            for (int i = 0; i < 3; i++) {
                System.out.println(Thread.currentThread().getName() +
                    " [priority=" + Thread.currentThread().getPriority() + "]: " + i);
            }
        };

        Thread low = new Thread(task, "Low");
        Thread normal = new Thread(task, "Normal");
        Thread high = new Thread(task, "High");

        low.setPriority(Thread.MIN_PRIORITY);    // 1
        normal.setPriority(Thread.NORM_PRIORITY); // 5
        high.setPriority(Thread.MAX_PRIORITY);    // 10

        low.start();
        normal.start();
        high.start();
    }
}
```

**Expected Output (order may vary):**
```
High [priority=10]: 0
Normal [priority=5]: 0
Low [priority=1]: 0
High [priority=10]: 1
Normal [priority=5]: 1
Low [priority=1]: 1
High [priority=10]: 2
Normal [priority=5]: 2
Low [priority=1]: 2
```

**Step-by-Step Explanation:**

1. Three threads are created with different priorities.
2. `setPriority()` sets the hint for the scheduler.
3. The threads are started; the scheduler decides the execution order.
4. Higher-priority threads may run more often, but the exact behaviour depends on the OS.

### Real-World Cases

- **Real-Time Systems:** Prioritising critical tasks.
- **GUI Applications:** Giving the UI thread higher priority.
- **Background Tasks:** Using lower priorities to avoid interfering with user interaction.

---

## 8. Context Switching

### Definitions

**Core Definition:** Context switching is the process of saving the state of one thread and restoring the state of another so that the CPU can switch between them.

**Technical Definition:** "The context switch halts execution of the currently executing execution flow and saves its context. The context switch then restores and activates an execution flow context specified by the execution flow activation method". "Context switching is more costly as it has to be done in the operating system instead of only in the JVM. Another disadvantage is that every Java thread consumes more resources, because it requires an operating system thread of its own".

**Beginner-Friendly Explanation:** Context switching is like switching between TV channels. When you change the channel, the TV saves your current position in the show (context) and loads the next channel's show from where you left off. This takes time and interrupts the flow. Similarly, when the CPU switches between threads, it must save the current thread's state and load another thread's state.

### Purposes

- To enable multiple threads to share a single CPU core
- To give the illusion of parallelism on single-core systems
- To respond to I/O events and interrupts
- To implement preemptive scheduling

### Syntax Rules and Structure

**Causes of Context Switching:**

| Cause | Description |
|-------|-------------|
| Time slice expiry | The OS preempts the running thread |
| Blocking I/O | The thread waits for I/O to complete |
| Synchronization | The thread waits for a lock |
| Higher-priority thread | A higher-priority thread becomes runnable |
| Interrupt | An interrupt handler runs |

**Cost of Context Switching:**

- "Context switch in the Linux kernel is pretty efficient compared with other Unix kernels, but still on the order of 20 to 40 microseconds".
- "A context switch in Java (generally based) is around 100 instructions".
- Frequent context switches reduce throughput.

**Syntax Rules:**

- Context switching is managed by the OS (for platform threads) or the JVM (for virtual threads).
- Reduce context switching by using thread pools and avoiding excessive blocking.
- Virtual threads have lower context-switching costs because they are JVM-managed.

### Annotated Complete Step-by-Step Code Examples

```java
// File: ContextSwitchDemo.java
public class ContextSwitchDemo {
    public static void main(String[] args) throws InterruptedException {
        // Simulate frequent context switching with many threads
        Runnable task = () -> {
            for (int i = 0; i < 1_000_000; i++) {
                // Minimal work to encourage context switching
                Math.sqrt(i);
            }
        };

        long start = System.nanoTime();
        Thread[] threads = new Thread[100];
        for (int i = 0; i < 100; i++) {
            threads[i] = new Thread(task);
            threads[i].start();
        }
        for (Thread t : threads) {
            t.join();
        }
        long elapsed = System.nanoTime() - start;

        System.out.println("100 threads completed in " +
            elapsed / 1_000_000 + " ms");
        System.out.println("Excessive threads increase context switching overhead");
    }
}
```

**Expected Output (times vary):**
```
100 threads completed in 2500 ms
Excessive threads increase context switching overhead
```

**Step-by-Step Explanation:**

1. 100 threads are created, each performing a small amount of work.
2. The OS must context-switch between these threads frequently.
3. The overhead of context switching reduces overall throughput.
4. Using a thread pool with fewer threads would reduce this overhead.

### Real-World Cases

- **Thread Pool Sizing:** Choosing the right number of threads to minimise context switching.
- **Performance Tuning:** Monitoring context switches with OS tools (`vmstat`, `perf`).
- **Virtual Threads:** Reducing context-switching overhead with JVM-managed threads.

---

## 9. Concurrency Best Practices

### Definitions

**Core Definition:** Concurrency best practices are guidelines for writing correct, efficient, and maintainable concurrent Java code.

**Technical Definition:** Best practices derive from authoritative sources including the SEI CERT Oracle Coding Standard for Java. Key principles include using thread pools instead of creating threads ad-hoc, avoiding blocking operations while holding locks, and ensuring thread safety through proper synchronization.

**Beginner-Friendly Explanation:** Concurrency best practices help you avoid common pitfalls like deadlocks, race conditions, and resource exhaustion. They're like a safety manual for multithreaded programming.

### Purposes

- To prevent deadlocks, race conditions, and data corruption
- To improve performance and scalability
- To ensure graceful degradation under load
- To write maintainable concurrent code
- To avoid resource exhaustion

### Key Best Practices Table

| Practice | Description | Source |
|----------|-------------|--------|
| Use Thread Pools | "Facilitate thread reuse by using Thread Pools". | SEI CERT |
| Avoid Blocking While Holding Locks | "Do not perform operations that may block while holding a lock". | SEI CERT |
| Use Bounded Queues | Prevent unbounded queue growth that can exhaust memory. | SEI CERT |
| Do Not Execute Interdependent Tasks in a Bounded Pool | "Do not execute interdependent tasks in a bounded thread pool". | SEI CERT |
| Ensure Tasks Are Interruptible | "Ensure that tasks submitted to a thread pool are interruptible". | SEI CERT |
| Reinitialize ThreadLocal Variables | "Ensure ThreadLocal variables are reinitialized when using thread pools". | SEI CERT |
| Use Consistent Locking Order | Prevent deadlocks by always acquiring locks in the same order. | Best practice |
| Prefer Higher-Level Concurrency Utilities | Use `java.util.concurrent` instead of `wait()`/`notify()` directly. | Best practice |
| Name Threads Meaningfully | Improves debugging and profiling. | Best practice |
| Minimize Lock Contention | Reduce the scope of synchronized blocks. | Best practice |
| Use Virtual Threads for I/O-Bound Tasks | "Virtual threads are suitable for executing tasks that spend most of the time blocked, often waiting for I/O". | Java documentation |

### Annotated Complete Step-by-Step Code Examples

```java
// File: BestPracticesDemo.java
import java.util.concurrent.*;

public class BestPracticesDemo {
    public static void main(String[] args) throws Exception {
        // BEST PRACTICE: Use a thread pool instead of creating threads ad-hoc
        ExecutorService executor = Executors.newFixedThreadPool(4);

        // BEST PRACTICE: Use meaningful thread names
        ThreadFactory namedFactory = r -> {
            Thread t = new Thread(r);
            t.setName("Worker-" + t.getId());
            return t;
        };

        // BEST PRACTICE: Use a bounded queue to prevent resource exhaustion
        ExecutorService boundedPool = new ThreadPoolExecutor(
            2, 4, 60L, TimeUnit.SECONDS,
            new ArrayBlockingQueue<>(100),  // Bounded queue
            namedFactory,
            new ThreadPoolExecutor.CallerRunsPolicy()  // Backpressure
        );

        // Submit tasks
        Future<Integer> future = executor.submit(() -> {
            // Task code
            return 42;
        });

        System.out.println("Result: " + future.get());

        // BEST PRACTICE: Shut down the executor properly
        executor.shutdown();
        executor.awaitTermination(5, TimeUnit.SECONDS);

        boundedPool.shutdown();
        boundedPool.awaitTermination(5, TimeUnit.SECONDS);

        System.out.println("All tasks completed");
    }
}
```

**Expected Output:**
```
Result: 42
All tasks completed
```

**Step-by-Step Explanation:**

1. A fixed thread pool is created with 4 threads.
2. A named thread factory gives threads meaningful names.
3. A bounded thread pool with a queue size of 100 prevents unbounded growth.
4. `CallerRunsPolicy` provides backpressure when the queue is full.
5. `shutdown()` and `awaitTermination()` ensure graceful shutdown.

### Real-World Cases

- **Web Servers:** Thread pools handle concurrent requests without resource exhaustion.
- **Background Processing:** Bounded queues prevent memory overflow during traffic spikes.
- **Database Connection Pools:** Reusing connections instead of creating new ones.

---

## References Links

- Processes and Threads — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/essential/concurrency/procthread.html
- Class Thread — Java Platform SE 27 API Specification - https://docs.oracle.com/en/java/javase/27/docs/api/java.base/java/lang/Thread.html
- Enum Thread.State — Java SE 8 API Specification - https://cr.openjdk.org/~mr/se/8/java-se-8-pr-spec/apidiffs/java/lang/Thread.State-report.html
- On Parallelism and Concurrency — Inside.java - https://inside.java/2021/11/30/on-parallelism-and-concurrency/
- CON02-J. Facilitate thread reuse by using Thread Pools — SEI CERT - https://wiki.sei.cmu.edu/confluence/pages/viewpage.action?pageId=88502000
- CON20-J. Do not perform operations that may block while holding a lock — SEI CERT - https://wiki.sei.cmu.edu/confluence/display/java/CON20-J.+Do+not+perform+operations+that+may+block+while+holding+a+lock
- Chapter 17. Threads and Locks — Java Language Specification - https://docs.oracle.com/javase/specs/jls/se21/html/jls-17.html
- Thread Priority on the Solaris Platform — Oracle - https://docs.oracle.com/cd/E19455-01/806-3461/6jck06gqm/index.html
- Project Loom — OpenJDK - https://openjdk.org/projects/loom/
- 14. Thread Pools (TPS) — SEI CERT - https://wiki.sei.cmu.edu/confluence/display/java/14.+Thread+Pools+%28TPS%29
- Java Concurrency and Parallelism — Duke University Libraries - https://find.library.duke.edu/catalog/DUKE99119487065008501