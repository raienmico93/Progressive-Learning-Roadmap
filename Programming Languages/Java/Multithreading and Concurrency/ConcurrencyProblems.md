# Concurrency Faults, Deadlocks, & Performance Problems: A Comprehensive Research Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**
Concurrency faults are defects that arise in multithreaded programs when threads interact incorrectly through shared memory, locks, or scheduling. They include race conditions (data corruption), deadlocks (permanent lockouts), starvation (indefinite resource denial), and livelocks (futile state changes). Performance problems arise from excessive thread contention, where threads spend more time waiting for locks than doing useful work.

**Technical Definition**
A race condition occurs when two or more threads access shared data concurrently and at least one access is a write, without proper synchronization establishing a happens-before relationship. A deadlock occurs when two or more threads are blocked forever, each waiting for a lock held by another, forming a circular wait. A livelock occurs when threads are not blocked but are too busy responding to each other to make progress. Starvation occurs when a thread is perpetually denied access to shared resources because other threads monopolize them. Thread contention is the performance degradation that results when multiple threads compete for the same lock or shared resource, causing threads to spend time waiting rather than executing.

**Beginner-Friendly Explanation**
Imagine a busy intersection with no traffic lights. A **race condition** is two cars trying to enter the same lane simultaneously and crashing. A **deadlock** is four cars at a four-way stop, each waiting for the others to go first, and nobody ever moves. A **livelock** is two people stepping aside to let each other pass, then stepping back, repeatedly, never actually passing. **Starvation** is a pedestrian who never gets a chance to cross because the cars never stop coming. **Contention** is the traffic jam that forms when too many cars try to use a single-lane road.

### Key Characteristics

- **Four Necessary Deadlock Conditions**: Deadlock requires mutual exclusion, hold-and-wait, no preemption, and circular wait; all four must hold simultaneously.
- **Race Condition Detectability**: Race conditions are notoriously difficult to detect because they depend on scheduling timing and may only manifest under specific load conditions.
- **Livelock Resemblance to Deadlock**: Both prevent progress, but livelocked threads are actively executing (changing state) rather than blocked.
- **Starvation vs. Deadlock**: Starvation may eventually resolve if the greedy thread finishes; deadlock never resolves without external intervention.
- **Contention as a Performance Bottleneck**: High contention turns a multithreaded application into a de facto single-threaded one, as threads queue up for locks.
- **Tooling Support**: Java provides `ThreadMXBean.findDeadlockedThreads()` for programmatic deadlock detection, and `jstack` for thread dump analysis.

### Prerequisites

- Understanding of Java threads, `synchronized`, and `ReentrantLock`
- Familiarity with the Java Memory Model and happens-before relationships
- Knowledge of concurrent collections and the Executor framework
- Awareness of compare-and-swap (CAS) and atomic variables

### Related Programming Areas

- **Java Memory Model**: Defines visibility and ordering guarantees that prevent race conditions.
- **Executor Framework**: Thread pools can exacerbate or mitigate contention depending on pool sizing.
- **Concurrent Collections**: `ConcurrentHashMap` and `BlockingQueue` reduce contention through lock striping and lock-free algorithms.
- **Database Transactions**: Concepts like two-phase locking and deadlock detection mirror JVM-level synchronization.

### Core Concepts / Features

Five core concepts are covered: (1) race conditions, (2) deadlocks, (3) starvation, (4) livelocks, and (5) thread contention mitigation, thread dump analysis, and thread confinement.

---

## Core Concept 1: Data Synchronization Failures — Identifying and Resolving Race Conditions

### Definitions

**Core Definition**
A race condition occurs when the correctness of a computation depends on the relative timing or interleaving of multiple threads accessing shared data without proper synchronization.

**Technical Definition**
A race condition exists when two or more threads access the same shared variable (heap memory) concurrently, at least one access is a write, and the accesses are not properly synchronized. In Java, the absence of a happens-before relationship between conflicting accesses means the program has a data race, and the Java Memory Model does not guarantee visibility or ordering. Race conditions can cause lost updates, stale reads, and inconsistent state. Common patterns include check-then-act (e.g., lazy initialization without `volatile`), read-modify-write (e.g., `count++`), and compound operations on thread-safe collections that are not atomic as a group.

**Beginner-Friendly Explanation**
A race condition is like two people editing the same shared document at the same time. Person A reads a sentence, then Person B changes it, then Person A saves their version based on the old sentence—now B's change is lost. The final result depends on who "raced" faster.

### Purposes

- To identify unsafe access patterns that depend on thread scheduling.
- To recognize compound operations that must be atomic but are not.
- To apply appropriate synchronization (intrinsic locks, atomic variables, or concurrent collections) to eliminate the race.
- To avoid the subtle trap of assuming that individually atomic operations compose into an atomic group.

### Syntax Rules and Structure

**Complete General Syntax — Race Condition vs. Safe Pattern**

```java
// RACE CONDITION: check-then-act
if (map.get(key) == null) {        // Thread A checks
    map.put(key, value);           // Thread B also checks and puts first
}
// Result: duplicate put, lost update

// SAFE: atomic compound operation
map.putIfAbsent(key, value);       // atomic check-and-put

// RACE CONDITION: read-modify-write
counter++;                          // read, increment, write (3 steps, not atomic)

// SAFE: atomic increment
atomicInteger.incrementAndGet();   // single atomic operation

// RACE CONDITION: non-atomic compound on thread-safe collection
if (!list.contains(x)) {           // check
    list.add(x);                   // act (another thread may have added x)
}

// SAFE: synchronized block or atomic compound
synchronized (lock) {
    if (!list.contains(x)) list.add(x);
}
```

**Component Breakdown**

| Pattern | Risk | Solution |
|---------|------|----------|
| Check-then-act | Two threads both pass the check | `putIfAbsent`, `computeIfAbsent` |
| Read-modify-write | Lost updates | `AtomicInteger`, `LongAdder`, `synchronized` |
| Compound on concurrent collection | Non-atomic group of calls | Explicit synchronization or atomic compound method |
| Lazy initialization | Partially constructed object visible | `volatile` + double-checked locking |

**Syntax Rules**

1. A race condition requires shared mutable state; immutable objects and thread-confined data are race-free.
2. Individual atomic operations do not compose into atomic groups; VNA03-J warns against assuming that a group of calls to independently atomic methods is atomic.
3. The `java.util.concurrent.atomic` package provides atomic compound operations for single variables.
4. `ConcurrentHashMap` provides `putIfAbsent`, `computeIfAbsent`, `merge`, and `compute` as atomic compound methods.

**Constraints and Limitations**

- Race conditions may not manifest in every execution; they are timing-dependent and often appear only under load.
- Detecting race conditions statically is difficult; tools like ThreadRadar and formal analysis are research-grade.
- Adding `synchronized` to every method causes contention and can reduce performance dramatically.

### Annotated Code Examples

**Example 1: Race Condition and Its Fix**

```java
import java.util.concurrent.atomic.AtomicInteger;

public class RaceConditionDemo {
    // UNSAFE: plain int, read-modify-write is not atomic
    static int unsafeCounter = 0;

    // SAFE: AtomicInteger uses CAS
    static AtomicInteger safeCounter = new AtomicInteger(0);

    public static void main(String[] args) throws InterruptedException {
        Runnable unsafeTask = () -> {
            for (int i = 0; i < 100000; i++) unsafeCounter++; // RACE
        };
        Runnable safeTask = () -> {
            for (int i = 0; i < 100000; i++) safeCounter.incrementAndGet();
        };

        Thread t1 = new Thread(unsafeTask);
        Thread t2 = new Thread(unsafeTask);
        t1.start(); t2.start();
        t1.join(); t2.join();
        System.out.println("Unsafe counter: " + unsafeCounter); // often < 200000

        Thread t3 = new Thread(safeTask);
        Thread t4 = new Thread(safeTask);
        t3.start(); t4.start();
        t3.join(); t4.join();
        System.out.println("Safe counter: " + safeCounter.get()); // exactly 200000
    }
}
```

**Expected Output (unsafe value varies)**

```
Unsafe counter: 173421
Safe counter: 200000
```

**Why This Output Occurs**

`unsafeCounter++` is not atomic; it reads the current value, increments it, and writes it back. When two threads interleave these three steps, one thread's write may overwrite the other's, causing lost updates. `AtomicInteger.incrementAndGet()` uses a hardware CAS loop: it reads the current value, computes the new value, and attempts to set it; if another thread changed the value in between, the CAS fails and the loop retries. No updates are lost.

---

**Example 2: Check-Then-Act with ConcurrentHashMap**

```java
import java.util.concurrent.ConcurrentHashMap;

public class CheckThenActDemo {
    static ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();

    public static void main(String[] args) throws InterruptedException {
        Runnable unsafe = () -> {
            for (int i = 0; i < 1000; i++) {
                String key = "key-" + (i % 10);
                if (map.get(key) == null) {      // check
                    map.put(key, 1);             // act — RACE
                }
            }
        };

        Thread t1 = new Thread(unsafe);
        Thread t2 = new Thread(unsafe);
        t1.start(); t2.start();
        t1.join(); t2.join();

        // May have fewer than 10 entries or unexpected values
        System.out.println("Map size: " + map.size());
        map.forEach((k, v) -> System.out.println(k + "=" + v));
    }
}
```

**Expected Output (varies; may show size < 10 or values > 1)**

```
Map size: 10
key-0=1
key-1=1
...
```

**Why This Output Occurs**

The `if (map.get(key) == null) { map.put(key, 1); }` sequence is a check-then-act race. Two threads may both see `null` for the same key and both put a value. The `put` overwrites the previous value, so the final value is still 1, but the check was redundant and the operation is not atomic. The safe alternative is `map.putIfAbsent(key, 1)` or `map.computeIfAbsent(key, k -> 1)`.

### Real-World Cases

- **Lazy Initialization**: A singleton `getInstance()` method without synchronization can create multiple instances under concurrent access.
- **Counter Updates**: A web server's request counter using `int` instead of `AtomicInteger` loses counts under load.
- **Cache Population**: Two threads both compute and cache the same expensive value; one overwrites the other.

---

## Core Concept 2: Terminal Execution Lockouts — Deadlocks

### Definitions

**Core Definition**
A deadlock occurs when two or more threads are blocked forever, each waiting for a lock or resource held by another thread in a circular dependency.

**Technical Definition**
A deadlock requires four simultaneous conditions (Coffman conditions): (1) mutual exclusion—resources are non-sharable; (2) hold-and-wait—a thread holds at least one resource while waiting for another; (3) no preemption—resources cannot be forcibly taken from a thread; (4) circular wait—a cycle of threads exists where each holds a lock requested by the next. In Java, mutual exclusion, hold-and-wait, and no preemption are inherent to intrinsic locks, so the existence of a circular wait leads to deadlock. Deadlocks are almost always revealed in thread dumps, though they are not always shown as lock-ordering deadlocks. `ThreadMXBean.findDeadlockedThreads()` finds cycles of threads deadlocked waiting to acquire object monitors or ownable synchronizers.

**Beginner-Friendly Explanation**
Deadlock is like two people at a narrow doorway, each saying "after you." Person A holds the door open for B, but B holds the door open for A, and neither moves. In Java, Thread A holds lock 1 and waits for lock 2; Thread B holds lock 2 and waits for lock 1. Neither can proceed.

### Purposes

- To identify and diagnose circular wait conditions in multithreaded applications.
- To prevent deadlocks through lock ordering, timed lock acquisition, and deadlock detection.
- To understand why deadlocks occur and how to design lock acquisition protocols that avoid them.
- To use thread dumps and `ThreadMXBean` for programmatic deadlock detection.

### Syntax Rules and Structure

**Complete General Syntax — Deadlock vs. Prevention**

```java
// DEADLOCK: inconsistent lock ordering
Thread 1: synchronized(lockA) { synchronized(lockB) { ... } }
Thread 2: synchronized(lockB) { synchronized(lockA) { ... } }

// PREVENTION 1: Global lock ordering
// Always acquire locks in a fixed order (e.g., by object identity hash)
int hashA = System.identityHashCode(lockA);
int hashB = System.identityHashCode(lockB);
if (hashA < hashB) {
    synchronized(lockA) { synchronized(lockB) { ... } }
} else {
    synchronized(lockB) { synchronized(lockA) { ... } }
}

// PREVENTION 2: tryLock with timeout (ReentrantLock)
if (lock1.tryLock(100, TimeUnit.MILLISECONDS)) {
    try {
        if (lock2.tryLock(100, TimeUnit.MILLISECONDS)) {
            try { /* work */ }
            finally { lock2.unlock(); }
        }
    } finally { lock1.unlock(); }
}
// If tryLock fails, release and retry — no permanent block

// DETECTION: ThreadMXBean
ThreadMXBean tmx = ManagementFactory.getThreadMXBean();
long[] deadlockedIds = tmx.findDeadlockedThreads();
if (deadlockedIds != null) {
    ThreadInfo[] infos = tmx.getThreadInfo(deadlockedIds);
    for (ThreadInfo info : infos) {
        System.out.println(info.getThreadName() + " is deadlocked");
    }
}
```

**Component Breakdown**

| Prevention Technique | Description | Trade-off |
|----------------------|-------------|-----------|
| Global lock ordering | Acquire locks in a consistent order | Requires global ordering knowledge |
| `tryLock` with timeout | Acquire with timeout; release and retry on failure | May cause livelock if retries collide |
| Lock timeout | Use `tryLock(timeout)` instead of `lock()` | Reduces throughput but eliminates deadlock |
| Deadlock detection | Use `ThreadMXBean.findDeadlockedThreads()` | Detection only; does not prevent |

**Syntax Rules**

1. The four Coffman conditions must all hold for deadlock; breaking any one prevents deadlock.
2. Lock ordering is the most practical prevention technique: if all threads acquire locks in the same global order, circular wait cannot occur.
3. `tryLock(timeout)` breaks the no-preemption condition by allowing a thread to give up waiting.
4. `ThreadMXBean.findDeadlockedThreads()` returns thread IDs involved in monitor or ownable synchronizer deadlocks; it may not detect deadlocks involving `ReentrantReadWriteLock` in all JDK versions.

**Constraints and Limitations**

- Lock ordering requires a consistent global ordering scheme; `System.identityHashCode()` may collide, causing false ordering.
- `tryLock` with random backoff can cause livelock if threads repeatedly collide.
- `ThreadMXBean.findDeadlockedThreads()` has overhead and may not detect all deadlock types.
- Deadlocks may be hidden by intermediate threads or complex lock hierarchies.

### Annotated Code Examples

**Example 1: Deadlock with Two Locks**

```java
public class DeadlockDemo {
    private final Object lockA = new Object();
    private final Object lockB = new Object();

    public void method1() {
        synchronized (lockA) {
            System.out.println(Thread.currentThread().getName() + " holds lockA");
            try { Thread.sleep(100); } catch (InterruptedException e) { }
            synchronized (lockB) {
                System.out.println(Thread.currentThread().getName() + " acquired lockB");
            }
        }
    }

    public void method2() {
        synchronized (lockB) {
            System.out.println(Thread.currentThread().getName() + " holds lockB");
            try { Thread.sleep(100); } catch (InterruptedException e) { }
            synchronized (lockA) {
                System.out.println(Thread.currentThread().getName() + " acquired lockA");
            }
        }
    }

    public static void main(String[] args) {
        DeadlockDemo demo = new DeadlockDemo();
        new Thread(demo::method1, "Thread-1").start();
        new Thread(demo::method2, "Thread-2").start();
    }
}
```

**Expected Output (deadlock; program hangs)**

```
Thread-1 holds lockA
Thread-2 holds lockB
(deadlock: neither thread can acquire the other's lock)
```

**Why This Output Occurs**

Thread-1 acquires `lockA` and waits for `lockB`. Thread-2 acquires `lockB` and waits for `lockA`. Each holds a lock the other needs. The circular wait condition is satisfied, and neither thread can proceed. The program hangs indefinitely.

---

**Example 2: Deadlock Prevention via Lock Ordering**

```java
public class LockOrderingDemo {
    private final Object lockA = new Object();
    private final Object lockB = new Object();

    private void acquireLocks(Object first, Object second, Runnable work) {
        synchronized (first) {
            synchronized (second) {
                work.run();
            }
        }
    }

    public void method1() {
        // Always acquire in order: lockA then lockB
        acquireLocks(lockA, lockB, () ->
            System.out.println("Thread-1 acquired both locks"));
    }

    public void method2() {
        // Also acquire in order: lockA then lockB (same order!)
        acquireLocks(lockA, lockB, () ->
            System.out.println("Thread-2 acquired both locks"));
    }

    public static void main(String[] args) throws InterruptedException {
        LockOrderingDemo demo = new LockOrderingDemo();
        Thread t1 = new Thread(demo::method1, "Thread-1");
        Thread t2 = new Thread(demo::method2, "Thread-2");
        t1.start(); t2.start();
        t1.join(); t2.join();
        System.out.println("No deadlock: both threads completed");
    }
}
```

**Expected Output**

```
Thread-1 acquired both locks
Thread-2 acquired both locks
No deadlock: both threads completed
```

**Why This Output Occurs**

Both threads acquire locks in the same global order: `lockA` first, then `lockB`. A circular wait cannot form because both threads follow the same acquisition sequence. Thread-1 acquires both locks, completes, releases them, and then Thread-2 acquires both. No deadlock occurs.

---

**Example 3: Programmatic Deadlock Detection with ThreadMXBean**

```java
import java.lang.management.*;
import java.util.concurrent.*;

public class DeadlockDetectionDemo {
    public static void main(String[] args) throws InterruptedException {
        Object lock1 = new Object();
        Object lock2 = new Object();

        Thread t1 = new Thread(() -> {
            synchronized (lock1) {
                try { Thread.sleep(100); } catch (InterruptedException e) { }
                synchronized (lock2) { }
            }
        }, "Deadlock-1");

        Thread t2 = new Thread(() -> {
            synchronized (lock2) {
                try { Thread.sleep(100); } catch (InterruptedException e) { }
                synchronized (lock1) { }
            }
        }, "Deadlock-2");

        t1.start(); t2.start();
        Thread.sleep(500); // let deadlock form

        ThreadMXBean tmx = ManagementFactory.getThreadMXBean();
        long[] deadlockedIds = tmx.findDeadlockedThreads();

        if (deadlockedIds != null) {
            ThreadInfo[] infos = tmx.getThreadInfo(deadlockedIds);
            System.out.println("Deadlock detected:");
            for (ThreadInfo info : infos) {
                System.out.println("  " + info.getThreadName()
                    + " is blocked on " + info.getLockName());
            }
        } else {
            System.out.println("No deadlock detected");
        }
    }
}
```

**Expected Output**

```
Deadlock detected:
  Deadlock-1 is blocked on lock2
  Deadlock-2 is blocked on lock1
```

**Why This Output Occurs**

`findDeadlockedThreads()` analyzes the lock graph and detects the circular wait between the two threads. It returns the thread IDs of the deadlocked threads, and `getThreadInfo()` provides details about each thread's blocked lock. This allows a monitoring system to detect deadlocks programmatically.

### Real-World Cases

- **Bank Transfers**: Transferring money between two accounts without lock ordering causes deadlock when concurrent transfers go in opposite directions.
- **Dining Philosophers**: Five philosophers and five forks; without a global ordering, deadlock is guaranteed.
- **Connection Pools**: Two threads each holding a connection and waiting for a second connection to the same database can deadlock if the pool is exhausted.

---

## Core Concept 3: Execution Starvation — Threads Permanently Denied Resources

### Definitions

**Core Definition**
Starvation describes a situation where a thread is unable to gain regular access to shared resources and is unable to make progress, even though the resources may be available at times.

**Technical Definition**
Starvation occurs when shared resources are made unavailable for long periods by "greedy" threads. In Java, starvation can result from improper thread priorities, non-fair locks, or synchronized methods that take a long time to return. The Java platform does not demand a fair scheduler; higher-priority threads can consume all resources, preventing lower-priority threads from running. Unlike deadlock, starvation may resolve if the greedy thread eventually finishes. Holding locks while performing time-consuming or blocking operations (network I/O, file I/O, `Thread.sleep()`) can severely degrade performance and result in starvation. The `finalize()` method is a sophisticated example: if finalizers spend too much time, thread starvation can occur.

**Beginner-Friendly Explanation**
Starvation is like waiting in line at a coffee shop where a group of customers keeps ordering new drinks before you get to the counter. The barista is making drinks (resources are available), but the group monopolizes the barista's attention. You never get served.

### Purposes

- To recognize when one or more threads are being perpetually denied CPU time or lock access.
- To identify greedy patterns such as holding locks during blocking operations.
- To apply fairness policies (fair locks, thread priorities) that prevent starvation.
- To minimize lock hold time by extracting non-critical operations outside synchronized blocks.

### Syntax Rules and Structure

**Complete General Syntax — Starvation Prevention**

```java
// STARVATION RISK: holding lock during blocking operation
public synchronized void badMethod() throws InterruptedException {
    Thread.sleep(10000); // holds lock for 10 seconds!
}

// FIX: Release lock before blocking
public void goodMethod() throws InterruptedException {
    synchronized (this) { /* quick critical section */ }
    Thread.sleep(10000); // outside the lock
}

// STARVATION RISK: non-fair lock with high contention
ReentrantLock unfairLock = new ReentrantLock(false); // barging allowed

// FIX: Fair lock ensures longest-waiting thread gets lock
ReentrantLock fairLock = new ReentrantLock(true);

// STARVATION RISK: priority inversion
highPriorityThread.setPriority(Thread.MAX_PRIORITY);

// FIX: Avoid relying on priorities; use fair scheduling
// (Java priorities are platform-dependent and unreliable)
```

**Component Breakdown**

| Cause | Description | Prevention |
|-------|-------------|------------|
| Greedy thread | One thread monopolizes lock | Minimize lock hold time |
| Blocking under lock | `sleep()`, I/O while synchronized | Move blocking outside lock |
| Non-fair lock | Barging allows repeated acquisition | Use fair `ReentrantLock` |
| Priority inversion | Low-priority thread holds lock needed by high-priority | Avoid priorities; use priority inheritance (OS-level) |

**Syntax Rules**

1. Java's thread scheduler is not required to be fair; higher-priority threads may starve lower-priority ones.
2. Fair locks (`new ReentrantLock(true)`) grant access to the longest-waiting thread under contention.
3. Blocking operations (`Thread.sleep()`, `Object.wait()`, I/O) should never be performed while holding a lock.
4. `Object.wait()` releases the monitor, unlike `Thread.sleep()`, which retains it.

**Constraints and Limitations**

- Fair locks have lower throughput than non-fair locks due to queue management overhead.
- Java thread priorities are hints; the JVM and OS may ignore them.
- Starvation can be difficult to detect because the thread is not blocked in a visible state; it is simply not scheduled.

### Annotated Code Examples

**Example 1: Starvation from Blocking Under Lock**

```java
public class StarvationDemo {
    // BAD: holds lock while sleeping
    public synchronized void greedyMethod() {
        System.out.println(Thread.currentThread().getName() + " entered greedy");
        try { Thread.sleep(3000); } catch (InterruptedException e) { }
        System.out.println(Thread.currentThread().getName() + " exiting greedy");
    }

    public static void main(String[] args) {
        StarvationDemo demo = new StarvationDemo();

        // Greedy thread calls the synchronized method repeatedly
        Thread greedy = new Thread(() -> {
            for (int i = 0; i < 3; i++) demo.greedyMethod();
        }, "Greedy");

        // Patient thread waits its turn
        Thread patient = new Thread(() -> {
            for (int i = 0; i < 3; i++) demo.greedyMethod();
        }, "Patient");

        greedy.start();
        patient.start();
    }
}
```

**Expected Output (Patient may wait a long time)**

```
Greedy entered greedy
Greedy exiting greedy
Greedy entered greedy
Greedy exiting greedy
Greedy entered greedy
Greedy exiting greedy
Patient entered greedy
Patient exiting greedy
...
```

**Why This Output Occurs**

The `greedyMethod()` is `synchronized` and sleeps for 3 seconds while holding the monitor. The `Greedy` thread calls it three times in a row, effectively monopolizing the lock. The `Patient` thread cannot enter the synchronized method until the Greedy thread finishes all three iterations. The fix is to move the `Thread.sleep()` outside the synchronized block.

---

**Example 2: Fair Lock Preventing Starvation**

```java
import java.util.concurrent.locks.ReentrantLock;

public class FairLockDemo {
    private final ReentrantLock fairLock = new ReentrantLock(true);

    public void doWork() {
        fairLock.lock();
        try {
            System.out.println(Thread.currentThread().getName() + " working");
        } finally {
            fairLock.unlock();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        FairLockDemo demo = new FairLockDemo();

        Thread[] threads = new Thread[5];
        for (int i = 0; i < 5; i++) {
            threads[i] = new Thread(() -> {
                for (int j = 0; j < 3; j++) demo.doWork();
            }, "Worker-" + i);
            threads[i].start();
        }

        for (Thread t : threads) t.join();
        System.out.println("All workers completed");
    }
}
```

**Expected Output (interleaved fairly)**

```
Worker-0 working
Worker-1 working
Worker-2 working
Worker-3 working
Worker-4 working
Worker-0 working
...
All workers completed
```

**Why This Output Occurs**

The fair lock grants access in the order threads requested it, preventing any single thread from monopolizing the lock. All five threads make progress and complete.

### Real-World Cases

- **Web Servers**: A long-running request holding a lock can starve other requests; using timeouts and lock-free data structures prevents this.
- **GUI Applications**: The event dispatch thread can starve background threads if it holds locks during rendering.
- **Database Connections**: A fair connection pool ensures all request threads eventually get a connection.

---

## Core Concept 4: Constant State Manipulation Failures — Livelocks

### Definitions

**Core Definition**
A livelock occurs when two or more threads are not blocked but are too busy responding to each other's actions to make any real progress.

**Technical Definition**
A livelock is similar to a deadlock; both prevent two or more threads from proceeding, but in livelock the threads are unable to proceed for different reasons—they are actively executing but continually changing state in response to each other. A thread often acts in response to the action of another thread; if the other thread's action is also a response, livelock may result. Livelocked threads are not blocked—they are simply too busy responding to each other to resume work. Livelock can occur when threads retry failed operations without randomization; for example, two threads repeatedly acquiring and releasing locks in response to each other.

**Beginner-Friendly Explanation**
Livelock is like two people in a hallway, each stepping aside to let the other pass, then stepping back, repeatedly. They are both moving (not blocked), but neither makes progress. In Java, this happens when threads keep retrying operations in response to each other without any randomization or backoff.

### Purposes

- To recognize when threads are active but not progressing.
- To apply randomization and backoff to break livelock cycles.
- To distinguish livelock from deadlock (livelock threads are RUNNABLE, not BLOCKED).
- To design retry logic that avoids synchronized retry patterns.

### Syntax Rules and Structure

**Complete General Syntax — Livelock vs. Solution**

```java
// LIVELOCK: synchronized retry without randomization
while (true) {
    if (lock1.tryLock()) {
        try {
            if (lock2.tryLock()) {
                try { /* work */ return; }
                finally { lock2.unlock(); }
            }
            // if lock2 fails, release lock1 and retry immediately
        } finally { lock1.unlock(); }
    }
    // No backoff: threads retry in lockstep, repeatedly colliding
}

// SOLUTION: random backoff before retry
Random random = new Random();
while (true) {
    if (lock1.tryLock()) {
        try {
            if (lock2.tryLock()) {
                try { /* work */ return; }
                finally { lock2.unlock(); }
            }
        } finally { lock1.unlock(); }
    }
    Thread.sleep(random.nextInt(100)); // random backoff
}
```

**Component Breakdown**

| Cause | Description | Prevention |
|-------|-------------|------------|
| Synchronized retry | Threads retry in lockstep | Random backoff |
| Over-polite threads | Each yields to the other | Introduce asymmetry or priority |
| Busy-wait retry loops | `while` loop with no delay | `Thread.sleep()` or `yield()` |

**Syntax Rules**

1. Livelock threads are RUNNABLE (or TIMED_WAITING), not BLOCKED; they appear active in thread dumps.
2. Randomization (e.g., `Thread.sleep(random.nextInt(100))`) breaks the synchronization that causes livelock.
3. `Thread.yield()` may worsen livelock if both threads yield in lockstep.
4. Livelock is often a consequence of well-intended deadlock avoidance (e.g., `tryLock` retry loops).

**Constraints and Limitations**

- Livelock is difficult to detect automatically; thread dumps show active threads, not blocked ones.
- There is no universal solution; the retry logic must be designed with backoff and randomization.
- Livelock can waste CPU cycles indefinitely.

### Annotated Code Examples

**Example 1: Livelock with Two Polite Threads**

```java
public class LivelockDemo {
    static class Spoon {
        private Diner owner;
        public Spoon(Diner d) { owner = d; }
        public Diner getOwner() { return owner; }
        public synchronized void setOwner(Diner d) { owner = d; }
        public synchronized void use() {
            System.out.printf("%s has eaten!%n", owner.name);
        }
    }

    static class Diner {
        private String name;
        private boolean isHungry;
        public Diner(String n) { name = n; isHungry = true; }
        public String getName() { return name; }
        public boolean isHungry() { return isHungry; }

        public void eatWith(Spoon spoon, Diner spouse) {
            while (isHungry) {
                // Don't have the spoon, so wait patiently for spouse
                if (spoon.getOwner() != this) {
                    try { Thread.sleep(1); } catch (InterruptedException e) { continue; }
                    continue;
                }
                // If spouse is hungry, insist upon passing the spoon
                if (spouse.isHungry()) {
                    System.out.printf("%s: You eat first my darling %s!%n",
                        name, spouse.getName());
                    spoon.setOwner(spouse);
                    continue; // LIVELOCK: both keep passing the spoon
                }
                // Spouse wasn't hungry, so finally eat
                spoon.use();
                isHungry = false;
                spoon.setOwner(spouse);
            }
        }
    }

    public static void main(String[] args) {
        final Diner husband = new Diner("Bob");
        final Diner wife = new Diner("Alice");
        final Spoon s = new Spoon(husband);

        new Thread(() -> husband.eatWith(s, wife)).start();
        new Thread(() -> wife.eatWith(s, husband)).start();
    }
}
```

**Expected Output (livelock; program runs indefinitely)**

```
Bob: You eat first my darling Alice!
Alice: You eat first my darling Bob!
Bob: You eat first my darling Alice!
Alice: You eat first my darling Bob!
... (repeats forever)
```

**Why This Output Occurs**

Both diners are "polite" and keep passing the spoon to each other because they see the other as hungry. Neither actually eats. The threads are active (not blocked), but no progress is made. The fix is to introduce asymmetry, such as a random delay or a rule that one diner always eats first.

---

**Example 2: Livelock Prevention with Random Backoff**

```java
import java.util.Random;
import java.util.concurrent.locks.ReentrantLock;

public class LivelockFixDemo {
    private static final ReentrantLock lock1 = new ReentrantLock();
    private static final ReentrantLock lock2 = new ReentrantLock();
    private static final Random random = new Random();

    public static void main(String[] args) throws InterruptedException {
        Runnable task1 = () -> acquireBoth(lock1, lock2, "Thread-1");
        Runnable task2 = () -> acquireBoth(lock2, lock1, "Thread-2");

        Thread t1 = new Thread(task1, "Thread-1");
        Thread t2 = new Thread(task2, "Thread-2");
        t1.start(); t2.start();
        t1.join(); t2.join();
    }

    static void acquireBoth(ReentrantLock first, ReentrantLock second,
                            String name) {
        while (true) {
            if (first.tryLock()) {
                try {
                    if (second.tryLock()) {
                        try {
                            System.out.println(name + " acquired both locks");
                            return; // success
                        } finally { second.unlock(); }
                    }
                } finally { first.unlock(); }
            }
            // Random backoff breaks livelock
            try {
                Thread.sleep(random.nextInt(100));
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                return;
            }
        }
    }
}
```

**Expected Output**

```
Thread-1 acquired both locks
Thread-2 acquired both locks
```

**Why This Output Occurs**

The random backoff (`Thread.sleep(random.nextInt(100))`) breaks the synchronization between the two threads. When one thread fails to acquire both locks, it sleeps for a random time before retrying. This reduces the chance that both threads retry in lockstep and collide again. Eventually, one thread acquires both locks, completes, releases them, and the other thread succeeds.

### Real-World Cases

- **Ethernet Collision Detection**: Network devices use random backoff to avoid repeated collisions, a real-world livelock prevention mechanism.
- **Optimistic Concurrency**: CAS retry loops without backoff can livelock under high contention.
- **Polite Robots**: Robots programmed to avoid collisions by stepping aside can livelock if both step in the same direction.

---

## Core Concept 5: System Scale Limitations — Contention Mitigation, Thread Dump Analysis, and Thread Confinement

### Definitions

**Core Definition**
Thread contention occurs when multiple threads compete for the same shared resource, forcing some threads to wait while others hold locks. Mitigation techniques include lock striping, reducing lock hold time, and using lock-free data structures. Thread confinement eliminates sharing entirely by ensuring that data is accessed by only one thread.

**Technical Definition**
Thread contention occurs when multiple threads compete for the same shared resource; this waiting time adds up under load, turning a multithreaded application into a de facto single-threaded bottleneck. Thread dumps show waiting threads blocked on the same lock; multiple threads blocked on the same object indicates contention. Lock striping reduces contention by partitioning a data structure into independently locked segments, so threads accessing different segments do not contend. Thread confinement avoids sharing by ensuring that data is accessed by only one thread; the three forms are ad-hoc confinement, stack confinement, and `ThreadLocal`.

**Beginner-Friendly Explanation**
Contention is like a supermarket with one checkout counter and twenty shoppers—everyone queues up. Lock striping is like opening ten checkout counters, each handling a subset of shoppers. Thread confinement is like giving each shopper their own personal checkout lane that no one else can use.

### Purposes

- To identify contention bottlenecks using thread dumps and lock profiling.
- To reduce contention through lock striping, lock splitting, and lock-free algorithms.
- To eliminate sharing altogether using thread confinement patterns (`ThreadLocal`, stack confinement).
- To analyze thread dumps for BLOCKED, WAITING, and deadlocked threads.

### Syntax Rules and Structure

**Complete General Syntax — Contention Mitigation**

```java
// CONTENTION: single lock for entire map
public class SingleLockMap<K, V> {
    private final Map<K, V> map = new HashMap<>();
    public synchronized V get(K key) { return map.get(key); }
    public synchronized void put(K key, V value) { map.put(key, value); }
}

// MITIGATION 1: Lock striping (manual)
public class StripedMap<K, V> {
    private final int stripes = 16;
    private final List<Map<K, V>> maps;
    private final List<Object> locks;

    public V get(K key) {
        int stripe = Math.abs(key.hashCode()) % stripes;
        synchronized (locks.get(stripe)) {
            return maps.get(stripe).get(key);
        }
    }
    public void put(K key, V value) {
        int stripe = Math.abs(key.hashCode()) % stripes;
        synchronized (locks.get(stripe)) {
            maps.get(stripe).put(key, value);
        }
    }
}

// MITIGATION 2: ConcurrentHashMap (built-in lock striping + lock-free reads)
ConcurrentHashMap<K, V> map = new ConcurrentHashMap<>();

// MITIGATION 3: ThreadLocal (thread confinement)
private static final ThreadLocal<SimpleDateFormat> formatter =
    ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyy-MM-dd"));
// Each thread gets its own SimpleDateFormat instance — no contention

// MITIGATION 4: Stack confinement (local variables)
public void process() {
    StringBuilder sb = new StringBuilder(); // local to this thread
    // sb is confined to the stack of the current thread
}
```

**Component Breakdown**

| Technique | Description | When to Use |
|-----------|-------------|-------------|
| Lock striping | Partition data into independently locked segments | Large maps, caches |
| Lock splitting | Use separate locks for different fields | Objects with independent state |
| Lock-free structures | `ConcurrentLinkedQueue`, `AtomicInteger` | High-throughput queues, counters |
| ThreadLocal | Per-thread copy of a value | Non-thread-safe formatters, connections |
| Stack confinement | Local variables only | Any method-local object |

**Syntax Rules for Thread Dump Analysis**

1. `jstack <pid>` captures a thread dump; the output shows each thread's state (RUNNABLE, BLOCKED, WAITING, TIMED_WAITING) and lock information.
2. Multiple threads in BLOCKED state on the same lock indicate contention.
3. `ThreadMXBean.findDeadlockedThreads()` programmatically detects deadlocks.
4. `jstack -l` includes additional lock information and deadlock detection results.

**Constraints and Limitations**

- Lock striping introduces a trade-off: more stripes reduce contention but increase memory overhead.
- `ThreadLocal` values in thread pools must be reinitialized or removed to prevent data leakage between tasks (TPS04-J).
- Stack confinement is automatic for primitives but requires care for object references that may escape.
- Thread dumps are a snapshot; contention patterns may change over time.

### Annotated Code Examples

**Example 1: Lock Striping vs. Single Lock**

```java
import java.util.*;
import java.util.concurrent.*;

public class LockStripingBenchmark {
    static final int STRIPES = 16;
    static final int OPERATIONS = 1_000_000;

    public static void main(String[] args) throws InterruptedException {
        // Single lock
        Map<Integer, Integer> singleLockMap =
            Collections.synchronizedMap(new HashMap<>());
        long singleTime = benchmark(singleLockMap, OPERATIONS);

        // Striped locks
        List<Map<Integer, Integer>> stripedMaps = new ArrayList<>();
        List<Object> locks = new ArrayList<>();
        for (int i = 0; i < STRIPES; i++) {
            stripedMaps.add(new HashMap<>());
            locks.add(new Object());
        }
        long stripedTime = benchmarkStriped(stripedMaps, locks, OPERATIONS);

        System.out.println("Single lock time: " + singleTime + " ms");
        System.out.println("Striped lock time: " + stripedTime + " ms");
    }

    static long benchmark(Map<Integer, Integer> map, int ops)
            throws InterruptedException {
        ExecutorService pool = Executors.newFixedThreadPool(8);
        long start = System.currentTimeMillis();
        for (int t = 0; t < 8; t++) {
            pool.submit(() -> {
                for (int i = 0; i < ops / 8; i++) map.put(i % 100, i);
            });
        }
        pool.shutdown(); pool.awaitTermination(30, TimeUnit.SECONDS);
        return System.currentTimeMillis() - start;
    }

    static long benchmarkStriped(List<Map<Integer, Integer>> maps,
                                  List<Object> locks, int ops)
            throws InterruptedException {
        ExecutorService pool = Executors.newFixedThreadPool(8);
        long start = System.currentTimeMillis();
        for (int t = 0; t < 8; t++) {
            pool.submit(() -> {
                for (int i = 0; i < ops / 8; i++) {
                    int key = i % 100;
                    int stripe = key % STRIPES;
                    synchronized (locks.get(stripe)) {
                        maps.get(stripe).put(key, i);
                    }
                }
            });
        }
        pool.shutdown(); pool.awaitTermination(30, TimeUnit.SECONDS);
        return System.currentTimeMillis() - start;
    }
}
```

**Expected Output (timing varies)**

```
Single lock time: 850 ms
Striped lock time: 320 ms
```

**Why This Output Occurs**

With a single lock, all 8 threads compete for the same monitor, causing serialization and context-switching overhead. With 16 stripes, threads accessing different keys (and therefore different stripes) can proceed concurrently. The striped version has roughly 2.5× higher throughput.

---

**Example 2: ThreadLocal for Thread Confinement**

```java
import java.text.SimpleDateFormat;
import java.util.Date;

public class ThreadLocalDemo {
    // UNSAFE: SimpleDateFormat is not thread-safe
    private static final SimpleDateFormat sharedFormatter =
        new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

    // SAFE: each thread gets its own formatter
    private static final ThreadLocal<SimpleDateFormat> localFormatter =
        ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));

    public static void main(String[] args) throws InterruptedException {
        Runnable unsafeTask = () -> {
            for (int i = 0; i < 1000; i++) {
                String result = sharedFormatter.format(new Date());
                // May produce garbage or throw NumberFormatException
            }
            System.out.println("Unsafe task done");
        };

        Runnable safeTask = () -> {
            for (int i = 0; i < 1000; i++) {
                String result = localFormatter.get().format(new Date());
                // Each thread has its own formatter: safe
            }
            System.out.println("Safe task done");
        };

        Thread t1 = new Thread(safeTask, "Thread-1");
        Thread t2 = new Thread(safeTask, "Thread-2");
        t1.start(); t2.start();
        t1.join(); t2.join();
    }
}
```

**Expected Output**

```
Safe task done
Safe task done
```

**Why This Output Occurs**

`SimpleDateFormat` is not thread-safe; concurrent calls to `format()` on the same instance can corrupt internal state and produce incorrect results or exceptions. `ThreadLocal.withInitial()` creates a separate `SimpleDateFormat` instance for each thread. No sharing occurs, so no contention and no data corruption. This is stack confinement applied to a non-thread-safe object.

---

**Example 3: Thread Dump Analysis**

```java
// This example is run and then a thread dump is captured externally
public class ContentionExample {
    private static final Object lock = new Object();

    public static void main(String[] args) throws InterruptedException {
        // One thread holds the lock for a long time
        new Thread(() -> {
            synchronized (lock) {
                try { Thread.sleep(60000); } catch (InterruptedException e) { }
            }
        }, "LockHolder").start();

        Thread.sleep(100); // let LockHolder acquire the lock

        // Many threads block on the lock
        for (int i = 0; i < 10; i++) {
            new Thread(() -> {
                synchronized (lock) {
                    System.out.println("Acquired");
                }
            }, "Waiter-" + i).start();
        }
    }
}
```

**Thread Dump Excerpt (from `jstack -l <pid>`)**

```
"Waiter-0" #12 prio=5 os_prio=0 tid=0x00007f... nid=0x... waiting for monitor entry
   java.lang.Thread.State: BLOCKED (on object monitor)
        at ContentionExample.lambda$main$1(ContentionExample.java:20)
        - waiting to lock <0x000000076b1c8a90> (a java.lang.Object)
        - locked by "LockHolder"

"Waiter-1" #13 ... BLOCKED ...
"Waiter-2" #14 ... BLOCKED ...
...
"LockHolder" #11 ... RUNNABLE
   java.lang.Thread.State: TIMED_WAITING (sleeping)
        at java.lang.Thread.sleep(Native Method)
        - locked <0x000000076b1c8a90> (a java.lang.Object)
```

**Why This Thread Dump Is Diagnostic**

The dump shows 10 threads in BLOCKED state, all waiting to lock the same object (`0x000000076b1c8a90`). The `LockHolder` thread holds that lock and is in TIMED_WAITING (sleeping). This immediately reveals the contention bottleneck: a single lock held by a sleeping thread. The fix is to move the sleep outside the synchronized block or use a different concurrency strategy.

### Real-World Cases

- **`ConcurrentHashMap`**: Uses lock striping and CAS internally to support high-concurrency access without a single global lock.
- **`SimpleDateFormat` in Web Apps**: A common contention point; `ThreadLocal` or `DateTimeFormatter` (immutable) solves it.
- **Database Connection Pools**: Lock striping on connection IDs reduces contention when many threads acquire connections.
- **Thread Dumps in Production**: `jstack` is the primary tool for diagnosing hangs, deadlocks, and contention in live JVMs.

---

## Comparison Summary

| Fault | Thread State | Progress? | Detection | Primary Fix |
|-------|-------------|-----------|-----------|-------------|
| Race condition | RUNNABLE | Wrong results | Race detectors, load testing | Synchronize or use atomic operations |
| Deadlock | BLOCKED | Never | `ThreadMXBean`, `jstack` | Lock ordering, `tryLock` timeout |
| Starvation | RUNNABLE (not scheduled) | Never for starved thread | Fairness metrics, thread dumps | Fair locks, minimize lock hold time |
| Livelock | RUNNABLE | Never (but active) | CPU profiling, code review | Random backoff, asymmetry |
| Contention | BLOCKED / WAITING | Slow progress | Thread dumps, lock profilers | Lock striping, lock-free structures, confinement |

---

## References

- Starvation and Livelock (Oracle Java Tutorials) - https://docs.oracle.com/javase/tutorial/essential/concurrency/starvelive.html
- LCK09-J. Do not perform operations that can block while holding a lock (CERT Oracle Secure Coding Standard) - https://wiki.sei.cmu.edu/confluence/display/java/LCK09-J.+Do+not+perform+operations+that+can+block+while+holding+a+lock
- LCK07-J. Avoid deadlock by requesting and releasing locks in the same order (CERT Oracle Secure Coding Standard) - https://wiki.sei.cmu.edu/confluence/display/java/LCK07-J.+Avoid+deadlock+by+requesting+and+releasing+locks+in+the+same+order
- VNA03-J. Do not assume that a group of calls to independently atomic methods is atomic (CERT Oracle Secure Coding Standard) - https://wiki.sei.cmu.edu/confluence/display/java/VNA03-J.+Do+not+assume+that+a+group+of+calls+to+independently+atomic+methods+is+atomic
- TPS04-J. Ensure ThreadLocal variables are reinitialized when using thread pools (CERT Oracle Secure Coding Standard) - https://wiki.sei.cmu.edu/confluence/display/java/TPS04-J.+Ensure+ThreadLocal+variables+are+reinitialized+when+using+thread+pools
- ThreadMXBean (Java Platform SE 21 & JDK 21) - https://docs.oracle.com/en/java/javase/21/docs/api/java.management/java/lang/management/ThreadMXBean.html
- Thread Dump Analysis (OneUptime) - https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-01-30-thread-dump-analysis/README.md
- How to Fix 'Thread Contention' Issues (OneUptime) - https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-01-24-fix-thread-contention/README.md
- Goetz, Brian, et al. *Java Concurrency in Practice*. Addison-Wesley, 2006. (Chapters 10–12: Avoiding Liveness Hazards, Performance and Scalability, Testing for Concurrency)
- Oracle Java Tutorials: Deadlock - https://docs.oracle.com/javase/tutorial/essential/concurrency/deadlock.html
- Oracle Java Tutorials: Thread Confinement (Java Concurrency in Practice, Chapter 3) - https://jcip.net/listings/