# Java Concurrency Fundamentals: A Comprehensive Research Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**
Java concurrency fundamentals encompass the mechanisms by which multiple threads of execution coexist, share resources, and are managed by the Java Virtual Machine (JVM) and the underlying operating system. These fundamentals include the distinction between processes and threads, the difference between concurrency and parallelism, the formal thread lifecycle, thread identification and configuration, and the routing of uncaught thread-level exceptions.

**Technical Definition**
A thread is a thread of execution in a program; the JVM allows an application to have multiple threads of execution running concurrently. Platform threads are typically mapped 1:1 to kernel threads scheduled by the operating system and usually have a large stack and other resources maintained by the operating system. Virtual threads are user-mode threads scheduled by the Java runtime rather than the operating system, require few resources, and are suitable for executing tasks that spend most of the time blocked. Every thread has a unique identifier and a name, and platform threads additionally have a thread priority and are members of a thread group. Threads transition through six states defined by `Thread.State`: NEW, RUNNABLE, BLOCKED, WAITING, TIMED_WAITING, and TERMINATED. When a thread terminates due to an uncaught exception, the JVM queries the thread for its `UncaughtExceptionHandler` and invokes the handler's `uncaughtException` method.

**Beginner-Friendly Explanation**
Imagine a large office building. The building itself is a **process**—it has its own address, its own resources, and it's isolated from other buildings. Inside the building, several **workers** (threads) share the same office space, the same filing cabinets (heap memory), and the same coffee machine. Each worker has their own desk and notebook (stack memory) to keep track of their personal tasks. The building manager (operating system) decides which worker gets to use the shared printer (CPU) at any given moment. If a worker encounters an unresolvable problem (uncaught exception), the building's emergency protocol (UncaughtExceptionHandler) determines what happens next.

### Key Characteristics

- **Shared Heap, Private Stack**: All threads within a process share the process's heap memory, allowing them to access and modify the same objects, but each thread maintains its own private stack for local variables and method calls.
- **OS-Level Scheduling**: Platform threads are scheduled by the operating system's scheduler; the JVM does not control when a thread runs, only when it is eligible to run.
- **Lightweight Context Switching**: Thread context switching is less expensive than process context switching because threads share the same address space, requiring no change of memory mapping.
- **Six Formal States**: The JVM defines exactly six thread states via the `Thread.State` enumeration; these are virtual machine states that do not necessarily map to operating system thread states.
- **Daemon vs. User Threads**: The JVM shuts down when all started non-daemon threads have terminated; daemon threads do not prevent the shutdown sequence from beginning.
- **Centralized Exception Routing**: Uncaught exceptions are routed through a handler chain: the thread's specific handler, then its `ThreadGroup`, then the global default handler.

### Prerequisites

- Basic Java syntax (classes, methods, object instantiation)
- Familiarity with the `main` method and program entry point
- Awareness of the distinction between the call stack and the heap
- Understanding of exception handling (`try-catch`, `throws`)

### Related Programming Areas

- **Java Memory Model**: Defines happens-before relationships that govern visibility between threads.
- **Executor Framework**: Higher-level abstraction for managing pools of threads and asynchronous task execution.
- **Synchronization**: Mechanisms (`synchronized`, `volatile`, locks) for coordinating access to shared state.
- **Operating Systems**: Process scheduling, context switching, and memory management concepts directly inform Java's threading behavior.

### Core Concepts / Features

Five core concepts are covered: (1) process vs. thread, (2) concurrency vs. parallelism, (3) the thread lifecycle, (4) thread identification, naming, priorities, and daemon status, and (5) uncaught exception handling.

---

## Core Concept 1: Process vs. Thread — OS-Level Scheduling, Memory Allocation, and Context Switching

### Definitions

**Core Definition**
A process is an instance of a computer program being executed, with its own isolated memory space and resources. A thread is a smaller, lightweight unit of execution that lives inside a process and shares the process's memory and resources.

**Technical Definition**
A process is an executable program that is loaded into memory and has its own logical memory address space allocated by the kernel. A thread is a single sequential flow that runs in the address space of its process; all threads within a single process share the process's heap memory. Each thread maintains its own private stack memory to track local variables and method calls. In Java, a platform thread is typically mapped 1:1 to a kernel thread scheduled by the operating system, while a virtual thread is scheduled by the Java runtime and may be multiplexed onto a small set of carrier platform threads.

**Beginner-Friendly Explanation**
A process is like a house: it has its own address, its own rooms, and its own furniture. Nobody from outside can walk in uninvited. A thread is like a person living in that house. Several people (threads) can live in the same house (process), share the same kitchen and living room (heap memory), but each person has their own bedroom (stack) where they keep their personal belongings. When the house is sold (process terminates), everyone inside must leave.

### Purposes

- To understand why processes provide isolation while threads enable shared-memory communication.
- To recognize that thread creation and context switching are cheaper than process creation and switching.
- To reason about memory safety: process isolation prevents accidental corruption across programs, while thread sharing enables efficient data exchange but requires synchronization.
- To choose between multi-process and multi-threaded designs based on isolation, communication, and resource requirements.

### Syntax Rules and Structure

**Complete General Syntax — Process vs. Thread in Java**

```java
// A Java program runs as a process with at least one thread (the main thread)
public class ProcessVsThread {
    public static void main(String[] args) {
        // The JVM process contains the main thread by default
        Thread mainThread = Thread.currentThread();
        System.out.println("Current thread: " + mainThread.getName());

        // Additional threads share the process's heap
        Thread worker = new Thread(() -> {
            System.out.println("Worker thread: " + Thread.currentThread().getName());
        });
        worker.start();
    }
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Process | OS-level container with isolated address space and resources |
| Thread | Lightweight execution unit within a process; shares heap, has private stack |
| Heap memory | Shared among all threads in a process; stores objects |
| Stack memory | Private to each thread; stores local variables and method call frames |
| Context switch | The OS saving one thread's state and restoring another's |

**Syntax Rules**

1. Every Java program begins with a single non-daemon thread (the "main" thread) created by the JVM.
2. Threads created within a process share the same heap memory; they can access and modify the same objects.
3. Each thread has its own stack; local variables declared in one thread are not visible to other threads.
4. Thread context switching does not require a change of address space, making it faster than process context switching.

**Constraints and Limitations**

- Threads within a process are not protected against each other by the operating system; a bug in one thread can corrupt shared data seen by all.
- Creating a new process requires more resources (memory, file descriptors) than creating a new thread.
- Process isolation means inter-process communication requires special mechanisms (pipes, sockets, shared memory segments).
- Java does not provide direct APIs for creating new processes with shared heap memory; each Java process has its own independent JVM.

### Annotated Code Examples

**Example 1: Shared Heap vs. Private Stack**

```java
public class SharedHeapPrivateStack {
    // Shared among all threads (heap memory)
    static int sharedCounter = 0;

    public static void main(String[] args) throws InterruptedException {
        Runnable task = () -> {
            // Local variable: private to this thread (stack memory)
            int localCounter = 0;
            for (int i = 0; i < 1000; i++) {
                sharedCounter++;  // shared heap access
                localCounter++;   // private stack access
            }
            System.out.println(Thread.currentThread().getName()
                + " local: " + localCounter);
        };

        Thread t1 = new Thread(task, "Thread-A");
        Thread t2 = new Thread(task, "Thread-B");
        t1.start(); t2.start();
        t1.join(); t2.join();

        System.out.println("Shared counter: " + sharedCounter);
    }
}
```

**Expected Output (order may vary)**

```
Thread-A local: 1000
Thread-B local: 1000
Shared counter: 2000 (or less, due to race condition)
```

**Why This Output Occurs**

`sharedCounter` is a static field stored on the heap and shared by both threads. `localCounter` is a local variable stored on each thread's private stack, so each thread has its own independent copy. The final value of `sharedCounter` may be less than 2000 because `sharedCounter++` is not atomic and the two threads can interleave their read-modify-write operations. This demonstrates the shared-heap, private-stack model.

---

**Example 2: Main Thread as a Process's First Thread**

```java
public class MainThreadDemo {
    public static void main(String[] args) {
        Thread current = Thread.currentThread();
        System.out.println("Main thread name: " + current.getName());
        System.out.println("Main thread ID: " + current.threadId());
        System.out.println("Is daemon: " + current.isDaemon());
        System.out.println("Priority: " + current.getPriority());
        System.out.println("Thread group: " + current.getThreadGroup().getName());
    }
}
```

**Expected Output (values vary)**

```
Main thread name: main
Main thread ID: 1
Is daemon: false
Priority: 5
Thread group: main
```

**Why This Output Occurs**

The JVM creates the "main" thread to execute the `main()` method. It is a non-daemon thread with the default priority of 5 (`Thread.NORM_PRIORITY`) and belongs to the "main" thread group. This demonstrates that even the simplest Java program runs within a process that contains at least one thread.

### Real-World Cases

- **Web Servers**: A single server process handles thousands of concurrent requests using a thread pool; threads share the server's heap for caching and session data.
- **Database Systems**: A database server process uses multiple threads for query execution, transaction logging, and background maintenance, all sharing the buffer pool on the heap.
- **IDE Applications**: The IDE process uses a main thread for the UI and background threads for compilation, indexing, and version control operations.

---

## Core Concept 2: Concurrency vs. Parallelism — Interleaved Execution vs. Simultaneous Multi-Core Execution

### Definitions

**Core Definition**
Concurrency is the ability to deal with multiple things at once, often through interleaved execution on a single core. Parallelism is the ability to do multiple things at once, using multiple cores to execute tasks simultaneously.

**Technical Definition**
Concurrency means that two or more calculations happen within the same time frame, and there is usually some sort of dependency between them. Concurrency describes a problem (two things need to happen together), while parallelism describes a solution (two processor cores are used to execute two things simultaneously). On a single-core CPU, concurrency is achieved by rapidly switching between tasks (time-slicing), creating the illusion of simultaneous execution; at any given nanosecond, only one task is actually running. Parallelism requires multiple processor cores to execute tasks truly simultaneously.

**Beginner-Friendly Explanation**
Concurrency is like a single chef juggling multiple dishes: chopping vegetables for one dish, then stirring a pot for another, then checking the oven for a third. The chef is making progress on all dishes, but only doing one thing at a time. Parallelism is like having three chefs, each working on a different dish simultaneously. Concurrency is about *dealing with* many things; parallelism is about *doing* many things.

### Purposes

- To distinguish between the design goal (concurrency: handling multiple tasks) and the implementation strategy (parallelism: using multiple cores).
- To recognize that concurrent programs on a single core are still correct and useful, even without true parallelism.
- To understand that parallelism is one way to implement concurrency, but not the only one (interleaved processing is another).
- To reason about performance: concurrency can improve responsiveness, while parallelism can improve throughput for CPU-bound tasks.

### Syntax Rules and Structure

**Complete General Syntax — Concurrency vs. Parallelism in Java**

```java
// Concurrency: two tasks interleaved on a single thread (or single core)
Thread t1 = new Thread(() -> {
    for (int i = 0; i < 5; i++) System.out.println("Task A: " + i);
});
Thread t2 = new Thread(() -> {
    for (int i = 0; i < 5; i++) System.out.println("Task B: " + i);
});
t1.start(); t2.start();

// Parallelism: tasks explicitly submitted to a multi-core pool
ExecutorService pool = Executors.newFixedThreadPool(4);
Future<Integer> f1 = pool.submit(() -> computeHeavyTaskA());
Future<Integer> f2 = pool.submit(() -> computeHeavyTaskB());
// Both tasks may run simultaneously on different cores
```

**Component Breakdown**

| Concept | Description |
|---------|-------------|
| Concurrency | Dealing with multiple tasks; interleaved execution |
| Parallelism | Doing multiple tasks simultaneously; requires multiple cores |
| Time-slicing | OS rapidly switches between threads on a single core |
| Multi-core execution | Tasks run on different cores at the same instant |

**Syntax Rules**

1. Concurrency is a property of the program structure; parallelism is a property of the execution environment.
2. Java threads can be concurrent without being parallel (e.g., on a single-core CPU).
3. Java threads can be parallel only if the underlying hardware has multiple cores and the OS scheduler assigns threads to different cores.
4. `Runtime.getRuntime().availableProcessors()` returns the number of processors available to the JVM, which bounds the degree of parallelism.

**Constraints and Limitations**

- Concurrency on a single core does not reduce total execution time for CPU-bound tasks; it may even increase it due to context-switching overhead.
- Parallelism requires multiple cores; on a single-core system, parallel threads are executed concurrently (interleaved), not simultaneously.
- Writing correct concurrent code is difficult regardless of whether parallelism is available.
- Virtual threads (Java 21+) improve concurrency scalability but do not create additional CPU cores for parallelism.

### Annotated Code Examples

**Example 1: Concurrent Interleaving on a Single Core**

```java
public class ConcurrencyDemo {
    public static void main(String[] args) throws InterruptedException {
        Runnable taskA = () -> {
            for (int i = 1; i <= 3; i++) {
                System.out.println("A" + i);
                Thread.yield(); // hint to scheduler: allow other threads to run
            }
        };
        Runnable taskB = () -> {
            for (int i = 1; i <= 3; i++) {
                System.out.println("B" + i);
                Thread.yield();
            }
        };

        Thread t1 = new Thread(taskA);
        Thread t2 = new Thread(taskB);
        t1.start(); t2.start();
        t1.join(); t2.join();
    }
}
```

**Expected Output (interleaving varies)**

```
A1
B1
A2
B2
A3
B3
```

**Why This Output Occurs**

The two threads are concurrent. On a single-core system, the OS scheduler interleaves their execution via time-slicing. `Thread.yield()` suggests that the current thread is willing to yield its current use of a processor, increasing the likelihood of interleaving. Even on a multi-core system, the output may be interleaved because both threads are making progress concurrently.

---

**Example 2: Parallel Execution on Multiple Cores**

```java
import java.util.concurrent.*;

public class ParallelismDemo {
    static long computeSum(int start, int end) {
        long sum = 0;
        for (int i = start; i <= end; i++) sum += i;
        return sum;
    }

    public static void main(String[] args) throws Exception {
        int cores = Runtime.getRuntime().availableProcessors();
        System.out.println("Available processors: " + cores);

        ExecutorService pool = Executors.newFixedThreadPool(cores);

        long startTime = System.nanoTime();
        Future<Long> f1 = pool.submit(() -> computeSum(1, 500_000_000));
        Future<Long> f2 = pool.submit(() -> computeSum(500_000_001, 1_000_000_000));

        long sum = f1.get() + f2.get();
        long endTime = System.nanoTime();

        System.out.println("Sum: " + sum);
        System.out.println("Time (ms): " + (endTime - startTime) / 1_000_000);
        pool.shutdown();
    }
}
```

**Expected Output (timing varies)**

```
Available processors: 8
Sum: 500000000500000000
Time (ms): 350
```

**Why This Output Occurs**

The two halves of the sum are computed in parallel on different cores (if available). The total execution time is approximately half of what it would be if computed sequentially, demonstrating true parallelism. On a single-core system, the tasks would run concurrently (interleaved) and the total time would be roughly the same as sequential execution.

### Real-World Cases

- **Web Servers**: Handle many concurrent requests on a limited number of cores; concurrency ensures responsiveness while parallelism accelerates CPU-bound request processing.
- **Video Encoding**: A video is split into chunks, each encoded in parallel on a different core; this is true parallelism for CPU-bound work.
- **UI Applications**: A background thread performs I/O concurrently with the UI thread, keeping the interface responsive even on a single-core device.

---

## Core Concept 3: Thread Lifecycle — Transition States (NEW, RUNNABLE, BLOCKED, WAITING, TIMED_WAITING, TERMINATED)

### Definitions

**Core Definition**
A Java thread exists in one of six states at any given moment: NEW, RUNNABLE, BLOCKED, WAITING, TIMED_WAITING, or TERMINATED. These states are defined by the `Thread.State` enumeration.

**Technical Definition**
The `Thread.State` enum defines the possible states of a thread. A thread can be in only one state at a given point in time. These states are virtual machine states that do not reflect any operating system thread states. NEW: a thread that has not yet started. RUNNABLE: a thread executing in the Java virtual machine. BLOCKED: a thread that is blocked waiting for a monitor lock. WAITING: a thread that is waiting indefinitely for another thread to perform a particular action. TIMED_WAITING: a thread that is waiting for another thread to perform an action for up to a specified waiting time. TERMINATED: a thread that has exited.

**Beginner-Friendly Explanation**
Think of a thread's life like a person going through stages of a day. **NEW** is when you've just woken up but haven't gotten out of bed yet (the thread object is created but `start()` hasn't been called). **RUNNABLE** is when you're up and about, doing your tasks (the thread is executing or ready to execute). **BLOCKED** is when you're waiting for the bathroom key (waiting for a monitor lock). **WAITING** is when you're waiting for a phone call that has no set time (waiting indefinitely for another thread). **TIMED_WAITING** is when you're waiting for a timer to go off (waiting with a timeout). **TERMINATED** is when your day is over (the thread has finished).

### Purposes

- To diagnose thread-related issues (deadlocks, starvation, excessive blocking) by inspecting thread states.
- To understand which operations cause transitions between states (e.g., `start()`, `sleep()`, `wait()`, `notify()`).
- To write correct concurrent code by knowing when a thread is blocked versus waiting versus runnable.
- To use `Thread.getState()` for monitoring and debugging, especially in thread dumps.

### Syntax Rules and Structure

**Complete General Syntax — Thread State Transitions**

```java
Thread t = new Thread(task);          // NEW
t.start();                             // NEW -> RUNNABLE
// RUNNABLE -> BLOCKED: waiting to acquire a monitor lock
// RUNNABLE -> WAITING: calls Object.wait() or Thread.join() without timeout
// RUNNABLE -> TIMED_WAITING: calls Thread.sleep(), Object.wait(timeout), or Thread.join(timeout)
// BLOCKED/WAITING/TIMED_WAITING -> RUNNABLE: lock acquired, notify()/notifyAll(), timeout expires
// RUNNABLE -> TERMINATED: run() completes or throws uncaught exception

Thread.State state = t.getState();     // query current state
```

**Component Breakdown**

| State | Description | Entry Condition | Exit Condition |
|-------|-------------|-----------------|----------------|
| NEW | Not yet started | Thread object created | `start()` called |
| RUNNABLE | Executing in JVM | `start()` called; or lock acquired; or timeout expired; or notified | Blocked on lock; waiting; terminated |
| BLOCKED | Waiting for monitor lock | Attempts to enter `synchronized` block held by another | Lock acquired |
| WAITING | Waiting indefinitely | `wait()`, `join()`, `park()` | `notify()`/`notifyAll()`, `join()` target terminates |
| TIMED_WAITING | Waiting with timeout | `sleep()`, `wait(timeout)`, `join(timeout)` | Timeout expires, notified, target terminates |
| TERMINATED | Exited | `run()` completes or throws | — |

**Syntax Rules**

1. `Thread.getState()` returns a `Thread.State` enum value representing the thread's current state.
2. A thread can be in only one state at any given time.
3. A thread in the NEW state has not yet been started; calling `getState()` on it returns NEW.
4. A thread in the TERMINATED state has completed execution; it cannot be restarted.
5. A thread blocked in `synchronized` is in BLOCKED state; a thread blocked in `wait()` is in WAITING or TIMED_WAITING state.

**Constraints and Limitations**

- `Thread.getState()` is intended for monitoring and debugging, not for synchronization or control flow.
- The JVM thread states are virtual machine states and do not necessarily correspond to operating system thread states.
- A thread may transition between RUNNABLE and BLOCKED/WAITING/TIMED_WAITING many times during its lifetime.
- `Thread.getState()` on a thread that has not been started returns NEW, not an error.

### Annotated Code Examples

**Example 1: Observing Thread State Transitions**

```java
public class ThreadStateDemo {
    public static void main(String[] args) throws InterruptedException {
        Object lock = new Object();

        Thread t = new Thread(() -> {
            synchronized (lock) {
                try {
                    System.out.println("Thread: acquiring lock, then waiting");
                    lock.wait(2000); // TIMED_WAITING
                    System.out.println("Thread: notified or timed out");
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            }
        });

        System.out.println("State after creation: " + t.getState()); // NEW
        t.start();
        Thread.sleep(100); // let thread enter wait
        System.out.println("State while waiting: " + t.getState()); // TIMED_WAITING

        synchronized (lock) {
            lock.notifyAll(); // wake the thread
        }

        t.join();
        System.out.println("State after termination: " + t.getState()); // TERMINATED
    }
}
```

**Expected Output**

```
State after creation: NEW
Thread: acquiring lock, then waiting
State while waiting: TIMED_WAITING
Thread: notified or timed out
State after termination: TERMINATED
```

**Why This Output Occurs**

After creation, the thread is in NEW state. After `start()`, it enters RUNNABLE. It acquires the lock (RUNNABLE), then calls `wait(2000)`, transitioning to TIMED_WAITING. The main thread observes this state. `notifyAll()` wakes the thread, which reacquires the lock and returns to RUNNABLE. After `run()` completes, the thread transitions to TERMINATED.

---

**Example 2: BLOCKED State Due to Monitor Contention**

```java
public class BlockedStateDemo {
    public static void main(String[] args) throws InterruptedException {
        Object lock = new Object();

        Thread holder = new Thread(() -> {
            synchronized (lock) {
                try {
                    System.out.println("Holder: holding lock for 2 seconds");
                    Thread.sleep(2000); // keeps the lock while sleeping
                } catch (InterruptedException e) { }
            }
        });

        Thread waiter = new Thread(() -> {
            synchronized (lock) {
                System.out.println("Waiter: acquired lock");
            }
        });

        holder.start();
        Thread.sleep(100); // let holder acquire the lock
        waiter.start();
        Thread.sleep(100); // let waiter attempt to acquire

        System.out.println("Waiter state: " + waiter.getState()); // BLOCKED

        holder.join();
        waiter.join();
    }
}
```

**Expected Output**

```
Holder: holding lock for 2 seconds
Waiter state: BLOCKED
Waiter: acquired lock
```

**Why This Output Occurs**

The `holder` thread acquires the monitor of `lock` and sleeps for 2 seconds while holding it. The `waiter` thread attempts to enter the synchronized block but cannot acquire the monitor, so it transitions to BLOCKED. The main thread observes `waiter.getState()` returning `BLOCKED`. When the holder exits the synchronized block, the waiter acquires the lock and proceeds.

### Real-World Cases

- **Deadlock Diagnosis**: Thread dumps show thread states; a BLOCKED thread waiting for a lock held by a WAITING thread may indicate deadlock.
- **Performance Monitoring**: JVM monitoring tools track the number of threads in each state to detect contention.
- **Thread Pool Sizing**: Understanding how many threads are BLOCKED on I/O versus RUNNABLE helps tune pool sizes.

---

## Core Concept 4: Thread Identification, Naming Conventions, Priorities, and Daemon vs. User Threads

### Definitions

**Core Definition**
Every Java thread has a unique identifier (generated at creation and unchangeable) and a name (which can be set at creation or changed later). Platform threads additionally have a priority (an integer from 1 to 10) and are designated as either daemon or non-daemon (user) threads.

**Technical Definition**
Threads have a unique identifier and a name. The identifier is generated when a `Thread` is created and cannot be changed. The thread name can be specified when creating a thread or can be changed at a later time. Platform threads inherit the daemon status and thread priority from the thread that creates them. The shutdown sequence begins when all started non-daemon threads have terminated. Daemon threads are low-priority threads whose only role is to provide services to user threads. Virtual threads are always daemon threads and have a fixed priority that cannot be changed.

**Beginner-Friendly Explanation**
Think of threads as employees in a company. Each employee has a unique employee ID (thread ID) that never changes, and a name (thread name) that can be changed. Each employee also has a priority level (1 to 10) that hints at how urgently their work should be done. Some employees are permanent (user threads)—the company won't close until they all go home. Others are temporary contractors (daemon threads)—the company can close even if they're still working, because they're only there to support the permanent staff.

### Purposes

- To identify threads in logs, thread dumps, and debugging tools using meaningful names.
- To distinguish between threads that keep the JVM alive (user threads) and background service threads (daemon threads).
- To provide a scheduling hint to the operating system through thread priorities (though this is platform-dependent and unreliable).
- To set up thread-local storage and inheritable thread-local storage for per-thread data.

### Syntax Rules and Structure

**Complete General Syntax — Identification, Naming, Priority, and Daemon Status**

```java
// Thread identification
Thread t = new Thread(task);
long id = t.threadId();            // unique, unchangeable identifier
String name = t.getName();         // name (may be auto-generated or set)

// Naming
Thread named = new Thread(task, "MyWorker");
named.setName("RenamedWorker");

// Priority
Thread.MIN_PRIORITY;   // 1
Thread.NORM_PRIORITY;  // 5 (default)
Thread.MAX_PRIORITY;   // 10
t.setPriority(Thread.MAX_PRIORITY);
int priority = t.getPriority();

// Daemon status
t.setDaemon(true);     // must be called before start()
boolean isDaemon = t.isDaemon();
```

**Component Breakdown**

| Attribute | Description | Default | Changeable? |
|-----------|-------------|---------|-------------|
| Thread ID | Unique identifier generated at creation | Auto-generated | No |
| Thread name | Human-readable label | Auto-generated ("Thread-N") | Yes |
| Priority | Scheduling hint (1–10) | Inherited from creating thread (usually 5) | Yes |
| Daemon status | Whether thread prevents JVM shutdown | Inherited from creating thread | Only before `start()` |

**Syntax Rules**

1. The thread ID is generated when the `Thread` is created and cannot be changed.
2. The thread name can be set at creation via a constructor or changed later with `setName()`.
3. Priority values range from `Thread.MIN_PRIORITY` (1) to `Thread.MAX_PRIORITY` (10); `Thread.NORM_PRIORITY` is 5.
4. `setDaemon(true)` must be called before `start()`; otherwise `IllegalThreadStateException` is thrown.
5. Virtual threads are always daemon threads and have a fixed priority that cannot be changed.
6. A newly created platform thread inherits its priority and daemon status from the creating thread.

**Constraints and Limitations**

- Thread priorities are **not guaranteed** to affect scheduling; they are hints that the JVM and OS may ignore.
- Setting a thread's priority to `MAX_PRIORITY` does not guarantee it will run before lower-priority threads.
- Daemon threads are abruptly terminated when the JVM shuts down; they should not be used for tasks that require cleanup.
- A daemon thread cannot be changed to a user thread after it has been started.

### Annotated Code Examples

**Example 1: Naming and Identifying Threads**

```java
public class ThreadNamingDemo {
    public static void main(String[] args) {
        Thread unnamed = new Thread(() ->
            System.out.println("Unnamed thread: " + Thread.currentThread().getName()));
        unnamed.start();

        Thread named = new Thread(() ->
            System.out.println("Named thread: " + Thread.currentThread().getName()),
            "OrderProcessor");
        named.start();

        Thread renamed = new Thread(() ->
            System.out.println("Renamed thread: " + Thread.currentThread().getName()));
        renamed.setName("RenamedWorker");
        renamed.start();
    }
}
```

**Expected Output (order may vary)**

```
Unnamed thread: Thread-0
Named thread: OrderProcessor
Renamed thread: RenamedWorker
```

**Why This Output Occurs**

The first thread is created without a name, so the JVM assigns an auto-generated name ("Thread-0"). The second thread is created with the name "OrderProcessor" via the constructor. The third thread is created unnamed but renamed to "RenamedWorker" before `start()`. Meaningful names make debugging and monitoring easier.

---

**Example 2: Daemon vs. User Threads**

```java
public class DaemonDemo {
    public static void main(String[] args) {
        // User thread: JVM will wait for this to finish
        Thread userThread = new Thread(() -> {
            try {
                for (int i = 1; i <= 5; i++) {
                    System.out.println("User thread working: " + i);
                    Thread.sleep(400);
                }
            } catch (InterruptedException e) { }
            System.out.println("User thread finished");
        });

        // Daemon thread: JVM will NOT wait for this
        Thread daemonThread = new Thread(() -> {
            try {
                for (int i = 1; i <= 100; i++) {
                    System.out.println("Daemon thread working: " + i);
                    Thread.sleep(400);
                }
            } catch (InterruptedException e) { }
            System.out.println("Daemon thread finished");
        });
        daemonThread.setDaemon(true); // must be before start()

        userThread.start();
        daemonThread.start();

        System.out.println("Main thread finished");
    }
}
```

**Expected Output (daemon thread may be cut off)**

```
Main thread finished
User thread working: 1
Daemon thread working: 1
User thread working: 2
Daemon thread working: 2
...
User thread working: 5
User thread finished
```

**Why This Output Occurs**

The main thread finishes quickly, but the JVM waits for the user thread to complete (5 iterations × 400 ms = 2 seconds). The daemon thread runs in the background, but when the user thread finishes, the JVM shuts down and the daemon thread is terminated abruptly—it never reaches "Daemon thread finished." Daemon threads are suitable for background services (garbage collection, monitoring) but not for tasks that must complete.

---

**Example 3: Priority Inheritance**

```java
public class PriorityDemo {
    public static void main(String[] args) {
        Thread.currentThread().setPriority(8); // set main thread priority
        System.out.println("Main priority: " + Thread.currentThread().getPriority());

        Thread child = new Thread(() ->
            System.out.println("Child priority: " + Thread.currentThread().getPriority()));
        child.start(); // child inherits priority 8 from main

        Thread low = new Thread(() ->
            System.out.println("Low priority thread running"));
        low.setPriority(Thread.MIN_PRIORITY); // 1
        low.start();
    }
}
```

**Expected Output**

```
Main priority: 8
Child priority: 8
Low priority thread running
```

**Why This Output Occurs**

The main thread sets its priority to 8. The child thread is created after this change and inherits priority 8 from the creating thread. The low-priority thread is explicitly set to `MIN_PRIORITY` (1). The JVM may or may not schedule the high-priority child before the low-priority thread; priority is a hint, not a guarantee.

### Real-World Cases

- **Named Threads for Debugging**: Servers name threads after their roles ("HTTP-Worker-1", "DB-Connection-Pool") so thread dumps are readable.
- **Daemon Threads for Background Services**: JVM garbage collection, JIT compilation, and finalizer threads are daemon threads.
- **Priority for Responsiveness**: UI frameworks may give the event dispatch thread a higher priority than background workers to keep the interface responsive.
- **Virtual Threads for Scalability**: Applications with millions of concurrent tasks use virtual threads (always daemon) instead of platform threads.

---

## Core Concept 5: Core Exception Routing — Handling Uncaught Thread-Level Exceptions via UncaughtExceptionHandler

### Definitions

**Core Definition**
`Thread.UncaughtExceptionHandler` is a functional interface for handlers invoked when a thread abruptly terminates due to an uncaught exception. When a thread is about to terminate due to an uncaught exception, the JVM queries the thread for its `UncaughtExceptionHandler` and invokes the handler's `uncaughtException` method, passing the thread and the exception as arguments.

**Technical Definition**
Interface for handlers invoked when a Thread abruptly terminates due to an uncaught exception. If a thread has not had its UncaughtExceptionHandler explicitly set, then its ThreadGroup object acts as its UncaughtExceptionHandler. If the ThreadGroup object has no special requirements for dealing with the exception, it can forward the invocation to the default uncaught exception handler. The exception routing order is: (1) the thread's specific `UncaughtExceptionHandler`, (2) the thread's `ThreadGroup`, (3) the global default `UncaughtExceptionHandler`. Any exception thrown by the `uncaughtException` method itself is ignored by the JVM.

**Beginner-Friendly Explanation**
Imagine an employee (thread) who encounters a problem they can't solve (uncaught exception). The company has a protocol: first, the employee checks if they have a personal emergency contact (their specific handler). If not, they report to their department manager (ThreadGroup). If the manager doesn't know what to do, they call the company's general emergency line (default handler). If nobody handles it, the employee's work simply stops, and the company logs the incident.

### Purposes

- To prevent silent thread termination by logging uncaught exceptions for diagnosis.
- To implement centralized error reporting for all threads in an application.
- To perform cleanup or fallback actions when a thread fails unexpectedly.
- To customize the behavior of `ThreadGroup` for exception handling across groups of related threads.

### Syntax Rules and Structure

**Complete General Syntax — UncaughtExceptionHandler**

```java
// Define a handler
Thread.UncaughtExceptionHandler handler = (thread, exception) -> {
    System.err.println("Thread " + thread.getName()
        + " failed with: " + exception.getMessage());
};

// Set per-thread handler
Thread t = new Thread(task);
t.setUncaughtExceptionHandler(handler);

// Set global default handler
Thread.setDefaultUncaughtExceptionHandler(handler);

// Custom ThreadGroup override
class MyGroup extends ThreadGroup {
    @Override
    public void uncaughtException(Thread t, Throwable e) {
        System.err.println("Group caught: " + e);
    }
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `UncaughtExceptionHandler` | Functional interface with `uncaughtException(Thread, Throwable)` |
| `setUncaughtExceptionHandler` | Sets the handler for a specific thread |
| `setDefaultUncaughtExceptionHandler` | Sets the global default handler for all threads |
| `ThreadGroup.uncaughtException` | Default handler for threads in a group |

**Syntax Rules**

1. The exception routing order is: thread-specific handler → ThreadGroup → default handler.
2. If a thread has not had its handler explicitly set, its `ThreadGroup` acts as its handler.
3. `ThreadGroup.uncaughtException` can forward to the default handler or handle the exception itself.
4. Any exception thrown by the `uncaughtException` method is ignored by the JVM.
5. `Thread.setDefaultUncaughtExceptionHandler` sets the handler for all threads that do not have a specific handler.

**Constraints and Limitations**

- The `uncaughtException` method cannot prevent the thread from terminating.
- Exceptions thrown within `uncaughtException` are silently ignored.
- The handler is invoked only for uncaught exceptions that cause thread termination; it is not invoked for exceptions caught within the thread.
- Thread pool tasks submitted via `ExecutorService` do not use `UncaughtExceptionHandler` for task exceptions; those are captured by the `Future`.

### Annotated Code Examples

**Example 1: Per-Thread UncaughtExceptionHandler**

```java
public class PerThreadHandlerDemo {
    public static void main(String[] args) {
        Thread t = new Thread(() -> {
            System.out.println("Thread starting");
            throw new RuntimeException("Intentional failure");
        }, "FailingWorker");

        // Set a handler for this specific thread
        t.setUncaughtExceptionHandler((thread, ex) -> {
            System.err.println("Handler caught exception from: " + thread.getName());
            System.err.println("Exception: " + ex.getMessage());
        });

        t.start();
    }
}
```

**Expected Output**

```
Thread starting
Handler caught exception from: FailingWorker
Exception: Intentional failure
```

**Why This Output Occurs**

The thread throws a `RuntimeException` that is not caught. Before terminating, the JVM queries the thread's `UncaughtExceptionHandler`, which was set explicitly. The handler receives the thread reference and the exception, prints diagnostic information, and the thread terminates. Without the handler, the default behavior would print a stack trace to `System.err`.

---

**Example 2: Global Default Handler**

```java
public class GlobalHandlerDemo {
    public static void main(String[] args) {
        // Set a global default handler for all threads
        Thread.setDefaultUncaughtExceptionHandler((thread, ex) -> {
            System.err.println("[GLOBAL] Thread " + thread.getName()
                + " failed: " + ex.getClass().getSimpleName()
                + " - " + ex.getMessage());
        });

        new Thread(() -> {
            throw new IllegalArgumentException("Invalid input");
        }, "Validator").start();

        new Thread(() -> {
            throw new NullPointerException("Missing object");
        }, "Loader").start();
    }
}
```

**Expected Output (order may vary)**

```
[GLOBAL] Thread Validator failed: IllegalArgumentException - Invalid input
[GLOBAL] Thread Loader failed: NullPointerException - Missing object
```

**Why This Output Occurs**

Neither thread has a specific handler set, so the JVM falls back to the global default handler. The handler prints a formatted message with the thread name, exception class, and message. This provides centralized error reporting for all threads in the application.

---

**Example 3: Custom ThreadGroup Exception Handling**

```java
public class ThreadGroupHandlerDemo {
    static class LoggingGroup extends ThreadGroup {
        LoggingGroup(String name) { super(name); }

        @Override
        public void uncaughtException(Thread t, Throwable e) {
            System.err.println("[" + getName() + "] Thread " + t.getName()
                + " died: " + e);
            // Optionally forward to default handler
            Thread.UncaughtExceptionHandler defaultHandler =
                Thread.getDefaultUncaughtExceptionHandler();
            if (defaultHandler != null) {
                defaultHandler.uncaughtException(t, e);
            }
        }
    }

    public static void main(String[] args) {
        Thread.setDefaultUncaughtExceptionHandler((thread, ex) ->
            System.err.println("[DEFAULT] Also logging: " + ex.getMessage()));

        ThreadGroup group = new LoggingGroup("CriticalWorkers");
        Thread t = new Thread(group, () -> {
            throw new OutOfMemoryError("Simulated resource exhaustion");
        }, "ResourceLoader");

        t.start();
    }
}
```

**Expected Output**

```
[CriticalWorkers] Thread ResourceLoader died: java.lang.OutOfMemoryError: Simulated resource exhaustion
[DEFAULT] Also logging: Simulated resource exhaustion
```

**Why This Output Occurs**

The thread belongs to `LoggingGroup`, which overrides `uncaughtException`. When the thread throws an `OutOfMemoryError`, the group's handler is invoked first. It logs the error and then forwards it to the global default handler for additional logging. This demonstrates the handler chain: thread-specific → ThreadGroup → default.

### Real-World Cases

- **Server Logging**: A web server sets a global handler to log all uncaught exceptions with thread names and request context before the thread dies.
- **Thread Pool Monitoring**: Executors can be wrapped to set uncaught exception handlers on worker threads for centralized error tracking.
- **Crash Reporting**: A desktop application sets a global handler to save a crash report and display a user-friendly message when a background thread fails.
- **ThreadGroup for Application Modules**: An application server assigns each deployed application to a `ThreadGroup` with a custom handler that logs exceptions with the application's context.

---

## References

- Thread (Java Platform SE 21 & JDK 21) - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Thread.html
- Thread.State (Java SE 9 & JDK 9) - https://docs.oracle.com/javase/9/docs/api/java/lang/Thread.State.html
- Thread States for a Thread Dump (Java SE 8 Troubleshooting Guide) - https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/tooldescr034.html
- Thread.UncaughtExceptionHandler (Java Platform SE 11) - https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Thread.UncaughtExceptionHandler.html
- Java Language Specification, Chapter 17: Threads and Locks - https://docs.oracle.com/javase/specs/jls/se23/html/jls-17.html
- Oracle Java Tutorials: Concurrency - https://docs.oracle.com/javase/tutorial/essential/concurrency/
- Oracle Java Tutorials: Processes and Threads - https://docs.oracle.com/javase/tutorial/essential/concurrency/procthread.html
- Oracle Java Tutorials: Thread Objects - https://docs.oracle.com/javase/tutorial/essential/concurrency/threads.html
- Oracle Java Tutorials: Thread States - https://docs.oracle.com/javase/tutorial/essential/concurrency/locksync.html
- Java Concurrency in Practice (Chapter 1: Introduction, Chapter 6: Task Execution) - https://www.oreilly.com/library/view/java-concurrency-in/0321349601/
- Goetz, Brian. "Java Theory and Practice: Going Atomic." IBM developerWorks, 2004. - https://developer.ibm.com/articles/j-jtp11234/