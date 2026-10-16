# Advanced Synchronization Utilities & Lock-Free Tools: A Comprehensive Research Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**
Advanced synchronization utilities are the classes in `java.util.concurrent.locks` and `java.util.concurrent` that provide explicit locking, phase coordination, and lock-free atomic operations beyond what `synchronized` and `volatile` alone can offer.

**Technical Definition**
The `java.util.concurrent.locks` package provides a framework for locking and waiting for conditions that is distinct from built-in synchronization and monitors. `Lock` implementations provide more extensive locking operations than can be obtained using `synchronized` methods and statements; they allow more flexible structuring, may have quite different properties, and may support multiple associated `Condition` objects. The `java.util.concurrent.atomic` package provides a small toolkit of classes that support lock-free thread-safe programming on single variables. These utilities build on hardware-level compare-and-swap (CAS) instructions and the Java Memory Model's happens-before guarantees.

**Beginner-Friendly Explanation**
Think of synchronization in layers. The `synchronized` keyword is like a basic door lock—simple, reliable, but you have to lock and unlock it in the same room. The advanced utilities are like a full security system: you can schedule when doors lock, give different keys to different people, set alarms if someone waits too long, and coordinate multiple doors opening at once. The atomic classes are like self-updating counters that never miss a beat, even when thousands of people press them simultaneously.

### Key Characteristics

- **Explicit Locking**: `Lock` implementations allow acquisition and release in different scopes, unlike block-structured `synchronized`.
- **Non-Blocking Attempts**: `tryLock()` provides a non-blocking attempt to acquire a lock, enabling deadlock-avoidance strategies.
- **Interruptible Acquisition**: `lockInterruptibly()` allows a thread to be interrupted while waiting for a lock.
- **Timed Acquisition**: `tryLock(timeout, unit)` avoids indefinite blocking.
- **Fairness Control**: `ReentrantLock` and `ReentrantReadWriteLock` accept an optional fairness parameter.
- **Read/Write Separation**: `ReentrantReadWriteLock` and `StampedLock` allow concurrent readers while maintaining exclusive write access.
- **Permit-Based Throttling**: `Semaphore` limits the number of threads accessing a resource.
- **Phase Coordination**: `CountDownLatch` and `CyclicBarrier` coordinate thread groups at synchronization points.
- **Lock-Free Atomics**: `AtomicInteger`, `LongAdder`, and related classes use hardware CAS instead of locks.

### Prerequisites

- Understanding of Java threads and the `synchronized` keyword
- Familiarity with the Java Memory Model and happens-before relationships
- Knowledge of `ReentrantLock` basics (reentrancy, lock/unlock pairing)
- Awareness of compare-and-swap (CAS) as a hardware primitive

### Related Programming Areas

- **Java Memory Model**: All synchronization utilities rely on happens-before guarantees.
- **Executor Framework**: Thread pools use `BlockingQueue` and `Lock` internally.
- **Concurrent Collections**: `ConcurrentHashMap` uses CAS and lock striping.
- **Design Patterns**: Guarded Suspension, Producer-Consumer, Read-Write Lock patterns.

### Core Concepts / Features

Six core concepts are covered: (1) `Lock` interface vs. intrinsic synchronization, (2) `ReentrantLock` advanced configuration, (3) `ReentrantReadWriteLock` and `StampedLock`, (4) `Semaphore`, (5) `CountDownLatch` vs. `CyclicBarrier`, and (6) atomic variables including `LongAdder`.

---

## Core Concept 1: Explicit Lock Frameworks — Lock Interface Implementations vs. Intrinsic Synchronization

### Definitions

**Core Definition**
The `Lock` interface is an explicit locking mechanism that provides more extensive operations than the implicit monitor lock accessed via `synchronized`. It allows locks to be acquired and released in different scopes and in any order.

**Technical Definition**
`Lock` implementations provide more extensive locking operations than can be obtained using `synchronized` methods and statements. They allow more flexible structuring, may have quite different properties, and may support multiple associated `Condition` objects. A lock is a tool for controlling access to a shared resource by multiple threads. Commonly, a lock provides exclusive access to a shared resource: only one thread at a time can acquire the lock and all access to the shared resource requires that the lock be acquired first. Unlike `synchronized`, the absence of block-structured locking removes the automatic release of locks; the idiom `lock(); try { ... } finally { unlock(); }` should be used. All `Lock` implementations must enforce the same memory synchronization semantics as provided by the built-in monitor lock: a successful lock operation acts like a successful `monitorEnter`, and a successful unlock acts like a successful `monitorExit`.

**Beginner-Friendly Explanation**
`synchronized` is like a hotel room key card that automatically locks the door when you leave the room—it's simple and safe. `Lock` is like a physical key you must remember to return to the front desk. It gives you more control (you can leave the room, come back, and lock it from outside) but also more responsibility (if you forget to return the key, the room stays locked forever). The trade-off is flexibility versus convenience.

### Purposes

- To provide non-blocking lock acquisition attempts (`tryLock()`), enabling deadlock-avoidance strategies.
- To allow interruptible lock acquisition (`lockInterruptibly()`), so waiting threads can respond to cancellation.
- To support timed lock acquisition (`tryLock(timeout, unit)`), avoiding indefinite blocking.
- To enable lock acquisition and release in different scopes, supporting hand-over-hand locking algorithms.
- To provide fairness guarantees and multiple condition variables.

### Syntax Rules and Structure

**Complete General Syntax**

```java
public interface Lock {
    void lock();                                    // blocks until acquired
    void lockInterruptibly() throws InterruptedException; // interruptible
    boolean tryLock();                              // non-blocking; returns immediately
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
    void unlock();                                  // releases the lock
    Condition newCondition();                       // creates a condition variable
}
```

**Component Breakdown**

| Method | Description | Blocking Behavior |
|--------|-------------|-------------------|
| `lock()` | Acquires the lock | Blocks indefinitely |
| `lockInterruptibly()` | Acquires the lock unless interrupted | Blocks until acquired or interrupted |
| `tryLock()` | Attempts to acquire without blocking | Returns `true`/`false` immediately |
| `tryLock(time, unit)` | Attempts within timeout | Blocks up to timeout |
| `unlock()` | Releases the lock | — |
| `newCondition()` | Creates a `Condition` bound to this lock | — |

**Syntax Rules**

1. The standard idiom is `lock.lock(); try { ... } finally { lock.unlock(); }`.
2. A `Lock` implementation may detect erroneous use (e.g., deadlock) and throw an unchecked exception.
3. `Lock` instances are normal objects and can be used as targets in a `synchronized` statement, but this has no specified relationship with `lock()`; it is recommended never to do this except within the lock's own implementation.
4. The three acquisition forms (interruptible, non-interruptible, timed) may differ in performance and ordering guarantees.

**Constraints and Limitations**

- `unlock()` must be called in a `finally` block; otherwise, the lock may never be released.
- `synchronized` automatically releases the lock on exception; `Lock` does not.
- `Lock` is not reentrant unless implemented as such (`ReentrantLock` is; the raw interface contract does not require it).
- Using `Lock` requires more code and discipline than `synchronized`.

### Annotated Code Examples

**Example 1: Basic Lock Idiom**

```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class LockIdiomDemo {
    private final Lock lock = new ReentrantLock();
    private int sharedValue = 0;

    public void increment() {
        lock.lock();                    // acquire
        try {
            sharedValue++;              // critical section
        } finally {
            lock.unlock();              // MUST release in finally
        }
    }

    public int get() {
        lock.lock();
        try { return sharedValue; }
        finally { lock.unlock(); }
    }

    public static void main(String[] args) throws InterruptedException {
        LockIdiomDemo demo = new LockIdiomDemo();
        Runnable task = () -> {
            for (int i = 0; i < 10000; i++) demo.increment();
        };
        Thread t1 = new Thread(task);
        Thread t2 = new Thread(task);
        t1.start(); t2.start();
        t1.join(); t2.join();
        System.out.println("Value: " + demo.get()); // 20000
    }
}
```

**Expected Output**

```
Value: 20000
```

**Why This Output Occurs**

The `lock.lock()` / `finally { lock.unlock(); }` idiom ensures that the critical section executes atomically and the lock is released even if an exception occurs. Both threads increment the shared value exactly 10,000 times each, yielding 20,000.

### Real-World Cases

- **Hand-Over-Hand Locking**: Traversing a concurrent linked list requires acquiring the lock of the current node, then the next, then releasing the previous—impossible with `synchronized`'s block structure.
- **Deadlock Avoidance**: Using `tryLock(timeout)` allows a thread to back off and retry rather than blocking forever.
- **Cancellable Operations**: `lockInterruptibly()` allows a thread waiting for a lock to be cancelled by another thread.

---

## Core Concept 2: Reentrant Coordination — Advanced ReentrantLock Configuration

### Definitions

**Core Definition**
`ReentrantLock` is a reentrant mutual exclusion `Lock` with the same basic behavior and semantics as the implicit monitor lock accessed using `synchronized` methods and statements, but with extended capabilities including fairness control, timed acquisition, and interruptible acquisition.

**Technical Definition**
A `ReentrantLock` is owned by the thread last successfully locking, but not yet unlocking it. A thread invoking `lock` will return, successfully acquiring the lock, when the lock is not owned by another thread. The method will return immediately if the current thread already owns the lock. The constructor for this class accepts an optional fairness parameter. When set `true`, under contention, locks favor granting access to the longest-waiting thread. Otherwise, the lock does not guarantee any particular access order. `tryLock(long timeout, TimeUnit unit)` acquires the lock if it is not held by another thread within the given waiting time and the current thread has not been interrupted.

**Beginner-Friendly Explanation**
`ReentrantLock` is like a bank counter with a numbered queue ticket system. You can choose the "fair" option (the person who has waited longest gets served next) or the "fast" option (someone who just arrived might get served before a long-waiting customer). It also has a "I'll wait 30 seconds" button (`tryLock` with timeout) so you don't stand in line forever.

### Purposes

- To provide a reentrant lock with the same semantics as `synchronized` but with additional flexibility.
- To enable fairness policies that prevent thread starvation.
- To support timed lock acquisition, allowing threads to give up after a timeout.
- To support interruptible lock acquisition, enabling cooperative cancellation.
- To provide multiple `Condition` objects for sophisticated wait/notify scenarios.

### Syntax Rules and Structure

**Complete General Syntax**

```java
// Constructors
ReentrantLock lock = new ReentrantLock();           // non-fair (default)
ReentrantLock lock = new ReentrantLock(boolean fair); // fair or non-fair

// Acquisition
lock.lock();                                        // blocks until acquired
lock.lockInterruptibly();                           // interruptible
boolean acquired = lock.tryLock();                  // non-blocking
boolean acquired = lock.tryLock(5, TimeUnit.SECONDS); // timed

// Release
lock.unlock();                                      // must be called by owner

// Introspection
boolean held = lock.isLocked();
boolean heldByCurrent = lock.isHeldByCurrentThread();
int count = lock.getHoldCount();                    // reentrancy count
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `fair` | When `true`, favors longest-waiting thread under contention |
| `tryLock()` | Non-blocking acquisition attempt |
| `tryLock(time, unit)` | Timed acquisition attempt |
| `lockInterruptibly()` | Interruptible acquisition attempt |
| `isHeldByCurrentThread()` | Returns true if current thread holds the lock |
| `getHoldCount()` | Returns the number of holds on this lock by the current thread |

**Syntax Rules**

1. Fairness: when set to `true`, locks favor granting access to the longest-waiting thread; otherwise no particular order is guaranteed.
2. `ReentrantLock` is reentrant: the same thread can acquire the lock multiple times, and each `unlock()` decrements the hold count.
3. `tryLock()` returns `false` immediately if the lock is held by another thread.
4. `tryLock(timeout, unit)` blocks up to the timeout and throws `InterruptedException` if interrupted.
5. `unlock()` must be called by the thread that acquired the lock.

**Constraints and Limitations**

- Fair locks typically have lower throughput than non-fair locks due to queue management overhead.
- `ReentrantLock` must be manually unlocked; unlike `synchronized`, forgetting `unlock()` causes a permanent lock.
- Fairness does not guarantee absolute FIFO order; it only prevents starvation.

### Annotated Code Examples

**Example 1: Fair vs. Non-Fair Lock**

```java
import java.util.concurrent.locks.ReentrantLock;

public class FairnessDemo {
    public static void main(String[] args) throws InterruptedException {
        ReentrantLock fairLock = new ReentrantLock(true);    // fair
        ReentrantLock nonFairLock = new ReentrantLock(false); // non-fair

        System.out.println("--- Fair Lock ---");
        runWithLock(fairLock);
        System.out.println("--- Non-Fair Lock ---");
        runWithLock(nonFairLock);
    }

    static void runWithLock(ReentrantLock lock) throws InterruptedException {
        Runnable task = () -> {
            for (int i = 0; i < 2; i++) {
                lock.lock();
                try {
                    System.out.println(Thread.currentThread().getName()
                        + " acquired");
                    Thread.sleep(50);
                } catch (InterruptedException e) { }
                finally { lock.unlock(); }
            }
        };

        Thread[] threads = new Thread[5];
        for (int i = 0; i < 5; i++) {
            threads[i] = new Thread(task, "Thread-" + i);
            threads[i].start();
        }
        for (Thread t : threads) t.join();
    }
}
```

**Expected Output (fair lock favors FIFO)**

```
--- Fair Lock ---
Thread-0 acquired
Thread-1 acquired
Thread-2 acquired
Thread-3 acquired
Thread-4 acquired
...
--- Non-Fair Lock ---
Thread-0 acquired
Thread-0 acquired
Thread-1 acquired
Thread-2 acquired
...
```

**Why This Output Occurs**

With a fair lock, threads acquire the lock in roughly the order they requested it, preventing starvation. With a non-fair lock, a thread can "barge" and re-acquire the lock immediately, potentially starving other threads. The fair lock output shows more balanced acquisition.

---

**Example 2: tryLock with Timeout to Avoid Deadlock**

```java
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.ReentrantLock;

public class TryLockTimeoutDemo {
    public static void main(String[] args) {
        ReentrantLock lock1 = new ReentrantLock();
        ReentrantLock lock2 = new ReentrantLock();

        Thread t1 = new Thread(() -> {
            try {
                if (lock1.tryLock(1, TimeUnit.SECONDS)) {
                    System.out.println("T1 acquired lock1");
                    Thread.sleep(500);
                    if (lock2.tryLock(1, TimeUnit.SECONDS)) {
                        System.out.println("T1 acquired lock2");
                        lock2.unlock();
                    } else {
                        System.out.println("T1 failed to acquire lock2, backing off");
                    }
                    lock1.unlock();
                }
            } catch (InterruptedException e) { }
        });

        Thread t2 = new Thread(() -> {
            try {
                if (lock2.tryLock(1, TimeUnit.SECONDS)) {
                    System.out.println("T2 acquired lock2");
                    Thread.sleep(500);
                    if (lock1.tryLock(1, TimeUnit.SECONDS)) {
                        System.out.println("T2 acquired lock1");
                        lock1.unlock();
                    } else {
                        System.out.println("T2 failed to acquire lock1, backing off");
                    }
                    lock2.unlock();
                }
            } catch (InterruptedException e) { }
        });

        t1.start(); t2.start();
    }
}
```

**Expected Output**

```
T1 acquired lock1
T2 acquired lock2
T1 failed to acquire lock2, backing off
T2 failed to acquire lock1, backing off
```

**Why This Output Occurs**

Both threads acquire their first lock but fail to acquire the second within the timeout, avoiding a permanent deadlock. Without `tryLock(timeout)`, both threads would block forever.

### Real-World Cases

- **Database Connection Pools**: Fair locks prevent connection starvation when many threads compete for limited connections.
- **Trading Systems**: Timed lock acquisition prevents a thread from blocking forever on a lock held by a stalled transaction.
- **UI Frameworks**: `lockInterruptibly()` allows a background thread to be cancelled when the user navigates away.

---

## Core Concept 3: Segregating Read and Write Access — ReentrantReadWriteLock and StampedLock

### Definitions

**Core Definition**
`ReentrantReadWriteLock` maintains a pair of associated locks—one for read-only operations and one for writing—allowing multiple readers concurrently while ensuring exclusive write access. `StampedLock` is a capability-based lock with three modes (write, read, optimistic read) that often provides higher throughput than `ReentrantReadWriteLock`.

**Technical Definition**
A `ReadWriteLock` maintains a pair of associated locks, one for read-only operations and one for writing. The read lock may be held simultaneously by multiple reader threads, so long as there are no writers. The write lock is exclusive. `StampedLock` is a capability-based lock with three modes for controlling read/write access. The state of a `StampedLock` consists of a version and mode. Lock acquisition methods return a stamp that represents and controls access with respect to a lock state; `tryOptimisticRead()` returns a non-zero stamp only if the lock is not currently held in write mode. `StampedLock` is often much faster than `ReentrantReadWriteLock` because it uses fewer instructions for read lock acquisition.

**Beginner-Friendly Explanation**
`ReentrantReadWriteLock` is like a library with two types of doors: many people can enter through the "reading room" door simultaneously, but when someone needs to "reorganize the shelves" (write), they lock the entire library and everyone else must wait. `StampedLock` is like a library with a "checkout slip" system: you can grab a slip and read the shelves optimistically, but if someone reorganizes the shelves while you're reading, your slip becomes invalid and you must re-read.

### Purposes

- To allow concurrent read access while maintaining exclusive write access, improving throughput for read-heavy workloads.
- To provide a high-throughput alternative to `ReentrantReadWriteLock` via optimistic reads.
- To support lock downgrading (write lock to read lock) in `ReentrantReadWriteLock`.
- To enable optimistic read patterns that avoid the overhead of acquiring a read lock in `StampedLock`.

### Syntax Rules and Structure

**Complete General Syntax — ReentrantReadWriteLock**

```java
ReentrantReadWriteLock rwLock = new ReentrantReadWriteLock();
Lock readLock = rwLock.readLock();
Lock writeLock = rwLock.writeLock();

readLock.lock();
try { /* read */ } finally { readLock.unlock(); }

writeLock.lock();
try { /* write */ } finally { writeLock.unlock(); }
```

**Complete General Syntax — StampedLock**

```java
StampedLock lock = new StampedLock();

// Write mode
long stamp = lock.writeLock();
try { /* write */ } finally { lock.unlockWrite(stamp); }

// Read mode
long stamp = lock.readLock();
try { /* read */ } finally { lock.unlockRead(stamp); }

// Optimistic read
long stamp = lock.tryOptimisticRead();
/* read fields into locals */
if (!lock.validate(stamp)) {
    // data may have changed; retry with a read lock
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `readLock()` | Returns the read lock (shared) |
| `writeLock()` | Returns the write lock (exclusive) |
| `tryOptimisticRead()` | Returns a non-zero stamp if not write-locked |
| `validate(stamp)` | Returns true if the lock has not been write-locked since the stamp |
| `unlockRead(stamp)` / `unlockWrite(stamp)` | Release the corresponding mode |

**Syntax Rules**

1. `ReentrantReadWriteLock` supports up to 65,535 recursive write locks and 65,535 read locks.
2. `ReentrantReadWriteLock` does **not** allow an upgrade from read lock to write lock.
3. `StampedLock` is **not reentrant**; a thread holding a lock and trying to acquire it again will deadlock.
4. `StampedLock` optimistic reads must be validated with `validate(stamp)`; if validation fails, retry with a read lock.
5. `StampedLock` is designed for use as an internal utility in the development of thread-safe components.

**Constraints and Limitations**

- `StampedLock` is not reentrant; do not use it in recursive algorithms.
- Optimistic reads may fail under contention; the retry logic must be implemented.
- `ReentrantReadWriteLock` can cause writer starvation under heavy read loads.
- `StampedLock` does not implement the `Lock` or `ReadWriteLock` interfaces.

### Annotated Code Examples

**Example 1: ReentrantReadWriteLock**

```java
import java.util.concurrent.locks.*;

public class RWLockDemo {
    private final ReentrantReadWriteLock rwLock = new ReentrantReadWriteLock();
    private int value = 0;

    public int read() {
        rwLock.readLock().lock();
        try {
            System.out.println(Thread.currentThread().getName() + " reading: " + value);
            return value;
        } finally {
            rwLock.readLock().unlock();
        }
    }

    public void write(int newValue) {
        rwLock.writeLock().lock();
        try {
            System.out.println(Thread.currentThread().getName() + " writing: " + newValue);
            value = newValue;
        } finally {
            rwLock.writeLock().unlock();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        RWLockDemo demo = new RWLockDemo();

        // Multiple readers can run concurrently
        Runnable reader = demo::read;
        Thread r1 = new Thread(reader, "Reader-1");
        Thread r2 = new Thread(reader, "Reader-2");
        r1.start(); r2.start();

        // Writer is exclusive
        Thread w = new Thread(() -> demo.write(42), "Writer");
        w.start();

        r1.join(); r2.join(); w.join();
    }
}
```

**Expected Output (order may vary)**

```
Reader-1 reading: 0
Reader-2 reading: 0
Writer writing: 42
```

**Why This Output Occurs**

Both readers acquire the read lock simultaneously and run concurrently. The writer must wait until both readers release their read locks before acquiring the write lock. This demonstrates the shared-read, exclusive-write semantics.

---

**Example 2: StampedLock Optimistic Read**

```java
import java.util.concurrent.locks.StampedLock;

public class StampedLockDemo {
    private double x, y;
    private final StampedLock lock = new StampedLock();

    void move(double deltaX, double deltaY) {
        long stamp = lock.writeLock();
        try {
            x += deltaX;
            y += deltaY;
        } finally {
            lock.unlockWrite(stamp);
        }
    }

    double distanceFromOrigin() {
        long stamp = lock.tryOptimisticRead(); // optimistic read
        double currentX = x, currentY = y;     // read into locals

        if (!lock.validate(stamp)) {           // check if write occurred
            stamp = lock.readLock();            // fall back to read lock
            try {
                currentX = x;
                currentY = y;
            } finally {
                lock.unlockRead(stamp);
            }
        }
        return Math.sqrt(currentX * currentX + currentY * currentY);
    }

    public static void main(String[] args) {
        StampedLockDemo point = new StampedLockDemo();
        point.move(3, 4);
        System.out.println("Distance: " + point.distanceFromOrigin()); // 5.0
    }
}
```

**Expected Output**

```
Distance: 5.0
```

**Why This Output Occurs**

`tryOptimisticRead()` returns a stamp without blocking. The method reads `x` and `y` into local variables, then calls `validate(stamp)`. If no write occurred during the read, the values are consistent and the distance is computed. If a write occurred, the method retries with a proper read lock. This avoids the overhead of acquiring a read lock in the common case.

### Real-World Cases

- **Caching Systems**: `ReentrantReadWriteLock` allows many threads to read cached values while a single thread refreshes the cache.
- **Configuration Management**: `StampedLock` provides high-throughput reads of configuration values with occasional writes.
- **Geometric Computations**: The `StampedLock` example above is from the official Java documentation—computing distances from a mutable point.

---

## Core Concept 4: Resource Access Throttling — Semaphores

### Definitions

**Core Definition**
A counting semaphore maintains a set of permits. Each `acquire()` blocks until a permit is available and then takes it; each `release()` adds a permit and may release a blocking acquirer.

**Technical Definition**
A `Semaphore` is a counting semaphore. Conceptually, a semaphore maintains a set of permits. Each `acquire()` blocks if necessary until a permit is available, and then takes it. Each `release()` adds a permit, potentially releasing a blocking acquirer. However, no actual permit objects are used; the `Semaphore` just keeps a count of the number available and acts accordingly. Semaphores are often used to restrict the number of threads that can access some (physical or logical) resource. A semaphore initialized to one can be used as a mutual exclusion lock, sometimes called a binary semaphore. Unlike many `Lock` implementations, a binary semaphore has the property that the lock can be released by a thread other than the owner (since semaphores have no notion of ownership).

**Beginner-Friendly Explanation**
A semaphore is like a parking lot with a fixed number of spaces. Cars (threads) can enter only if a space is available. When a car leaves, it frees a space for another car. The parking lot doesn't care which car parks in which space; it only tracks how many spaces are free. This is perfect for limiting concurrent access to a resource pool.

### Purposes

- To restrict the number of threads that can concurrently access a resource (connection pool, file handles, network sockets).
- To implement a binary semaphore (mutex) that can be released by a different thread than the one that acquired it.
- To throttle request rates in servers and APIs.
- To signal between threads using permit counts.

### Syntax Rules and Structure

**Complete General Syntax**

```java
// Constructors
Semaphore sem = new Semaphore(int permits);
Semaphore sem = new Semaphore(int permits, boolean fair);

// Acquisition
sem.acquire();                          // blocks until permit available
sem.acquire(int n);                     // acquires n permits
boolean acquired = sem.tryAcquire();    // non-blocking
boolean acquired = sem.tryAcquire(1, TimeUnit.SECONDS); // timed

// Release
sem.release();                          // releases one permit
sem.release(int n);                     // releases n permits

// Introspection
int available = sem.availablePermits();
```

**Component Breakdown**

| Method | Description |
|--------|-------------|
| `acquire()` | Blocks until a permit is available, then takes it |
| `release()` | Adds a permit, potentially releasing a blocking acquirer |
| `tryAcquire()` | Non-blocking acquisition attempt |
| `tryAcquire(timeout, unit)` | Timed acquisition attempt |
| `availablePermits()` | Returns the current number of permits |

**Syntax Rules**

1. A semaphore initialized to one acts as a binary semaphore (mutex).
2. Unlike `Lock`, a semaphore has no ownership concept; any thread can call `release()`.
3. Fairness: when `false`, the semaphore does not guarantee any particular order of permit acquisition; barging is allowed.
4. `acquire()` does not hold any synchronization lock when called; it only manages the permit count.

**Constraints and Limitations**

- Semaphores do not protect the consistency of a resource; they only limit access. Additional synchronization may be needed for the resource itself.
- Forgetting to call `release()` in a `finally` block causes permit leaks.
- Fair semaphores have lower throughput than non-fair ones.

### Annotated Code Examples

**Example 1: Connection Pool with Semaphore**

```java
import java.util.concurrent.Semaphore;

public class ConnectionPool {
    private final Semaphore permits;
    private final int maxConnections;

    public ConnectionPool(int maxConnections) {
        this.maxConnections = maxConnections;
        this.permits = new Semaphore(maxConnections, true); // fair
    }

    public void useConnection(String threadName) throws InterruptedException {
        permits.acquire(); // wait for an available permit
        try {
            System.out.println(threadName + " using connection. Available: "
                + permits.availablePermits());
            Thread.sleep(500); // simulate work
        } finally {
            permits.release(); // always release in finally
            System.out.println(threadName + " released connection");
        }
    }

    public static void main(String[] args) {
        ConnectionPool pool = new ConnectionPool(3);

        Runnable task = () -> {
            try {
                pool.useConnection(Thread.currentThread().getName());
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        };

        for (int i = 0; i < 6; i++) {
            new Thread(task, "Thread-" + i).start();
        }
    }
}
```

**Expected Output (order may vary)**

```
Thread-0 using connection. Available: 2
Thread-1 using connection. Available: 1
Thread-2 using connection. Available: 0
Thread-0 released connection
Thread-1 released connection
Thread-2 released connection
Thread-3 using connection. Available: 2
...
```

**Why This Output Occurs**

The pool has 3 permits. The first three threads acquire permits and use connections simultaneously. The next three threads block on `acquire()` until the first three release their permits. The output shows exactly 3 concurrent connections, with the remaining threads waiting.

### Real-World Cases

- **Database Connection Pools**: `Semaphore` limits the number of active connections to a database.
- **Rate Limiting**: An API server uses a semaphore to limit concurrent requests.
- **File Descriptor Management**: A semaphore limits open file handles.
- **Thread Pool Sizing**: Semaphores coordinate the maximum number of concurrent workers.

---

## Core Concept 5: Multi-Thread Task Gating — CountDownLatch vs. CyclicBarrier

### Definitions

**Core Definition**
`CountDownLatch` allows one or more threads to wait until a set of operations being performed in other threads completes. `CyclicBarrier` allows a set of threads to all wait for each other to reach a common barrier point.

**Technical Definition**
A `CountDownLatch` is initialized with a given count. The `await` methods block until the current count reaches zero due to invocations of the `countDown()` method, after which all waiting threads are released. A `CountDownLatch` initialized with a count of one serves as a simple on/off latch, or gate. A `CyclicBarrier` is a synchronization aid that allows a set of threads to all wait for each other to reach a common barrier point. The barrier is called cyclic because it can be reused after the waiting threads are released. `CyclicBarrier` supports an optional `Runnable` command that is run once per barrier point, after the last thread in the party arrives, but before any threads are released. The key semantic difference is that `CyclicBarrier` maintains a count of threads, whereas `CountDownLatch` maintains a count of tasks.

**Beginner-Friendly Explanation**
`CountDownLatch` is like waiting for all the guests to arrive at a party. The host (waiting thread) waits at the door until every guest (task) has arrived. Once all guests are in, the host proceeds, but the party cannot be "re-run" with the same guest list. `CyclicBarrier` is like a group of friends meeting at a restaurant. Each friend (thread) waits at the meeting point until everyone else arrives. Only when all friends are there do they proceed together. The next time they meet, the same barrier can be reused.

### Syntax Rules and Structure

**Complete General Syntax — CountDownLatch**

```java
CountDownLatch latch = new CountDownLatch(int count);

latch.countDown();        // decrement count
latch.await();            // block until count reaches zero
latch.await(timeout, unit); // block up to timeout
long remaining = latch.getCount(); // current count
```

**Complete General Syntax — CyclicBarrier**

```java
CyclicBarrier barrier = new CyclicBarrier(int parties);
CyclicBarrier barrier = new CyclicBarrier(int parties, Runnable barrierAction);

barrier.await();          // wait for all parties
barrier.await(timeout, unit);
int waiting = barrier.getNumberWaiting();
boolean broken = barrier.isBroken();
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `countDown()` | Decrements the latch count |
| `await()` | Blocks until count reaches zero (latch) or all parties arrive (barrier) |
| `barrierAction` | Runnable executed when the last thread arrives (barrier only) |
| `getCount()` | Current latch count |
| `getNumberWaiting()` | Current number of waiting threads (barrier only) |

**Comparison Table**

| Aspect | CountDownLatch | CyclicBarrier |
|--------|----------------|---------------|
| Reusable? | No (single-use) | Yes (cyclic) |
| Counts | Tasks | Threads |
| One thread can count down multiple times? | Yes | No |
| Barrier action? | No | Yes (optional) |
| Typical use | Start gate, task completion | Phased computation, multi-stage algorithms |

**Syntax Rules**

1. `CountDownLatch` is single-use; once the count reaches zero, it cannot be reset.
2. `CyclicBarrier` is reusable; after all parties arrive, the barrier resets automatically.
3. In `CyclicBarrier`, a single thread cannot count down the barrier twice; each thread must call `await()` exactly once per cycle.
4. `CountDownLatch.await()` blocks until the count reaches zero; `CyclicBarrier.await()` blocks until all parties have invoked `await()`.
5. `CyclicBarrier` throws `BrokenBarrierException` if the barrier is broken (e.g., a thread times out).

**Constraints and Limitations**

- `CountDownLatch` cannot be reused; a new instance is needed for each synchronization event.
- `CyclicBarrier` requires all parties to call `await()`; if one thread fails to arrive, all others block.
- `CountDownLatch` counts tasks, not threads; one thread can count down multiple times.
- Neither class provides mutual exclusion; they are coordination aids only.

### Annotated Code Examples

**Example 1: CountDownLatch as a Start Gate**

```java
import java.util.concurrent.CountDownLatch;

public class CountDownLatchDemo {
    public static void main(String[] args) throws InterruptedException {
        CountDownLatch startGate = new CountDownLatch(1);
        CountDownLatch doneGate = new CountDownLatch(3);

        Runnable worker = () -> {
            try {
                startGate.await(); // all workers wait for the signal
                System.out.println(Thread.currentThread().getName() + " started");
                Thread.sleep(300);
                System.out.println(Thread.currentThread().getName() + " finished");
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            } finally {
                doneGate.countDown(); // signal completion
            }
        };

        for (int i = 1; i <= 3; i++) {
            new Thread(worker, "Worker-" + i).start();
        }

        System.out.println("Main: releasing start gate");
        startGate.countDown(); // release all workers simultaneously
        doneGate.await();      // wait for all workers to finish
        System.out.println("Main: all workers done");
    }
}
```

**Expected Output**

```
Main: releasing start gate
Worker-1 started
Worker-2 started
Worker-3 started
Worker-1 finished
Worker-2 finished
Worker-3 finished
Main: all workers done
```

**Why This Output Occurs**

`startGate` (count 1) blocks all workers until the main thread calls `countDown()`. This releases all three workers nearly simultaneously. `doneGate` (count 3) allows the main thread to wait until all three workers have finished. The latch is perfect for this "one-time start signal" pattern.

---

**Example 2: CyclicBarrier for Phased Computation**

```java
import java.util.concurrent.CyclicBarrier;

public class CyclicBarrierDemo {
    public static void main(String[] args) {
        int parties = 3;
        // Barrier action runs when all threads arrive
        CyclicBarrier barrier = new CyclicBarrier(parties,
            () -> System.out.println("--- All threads reached barrier ---"));

        Runnable task = () -> {
            try {
                for (int phase = 1; phase <= 3; phase++) {
                    System.out.println(Thread.currentThread().getName()
                        + " working on phase " + phase);
                    Thread.sleep(200 * phase);
                    barrier.await(); // wait for all threads
                    System.out.println(Thread.currentThread().getName()
                        + " proceeding past phase " + phase);
                }
            } catch (Exception e) {
                Thread.currentThread().interrupt();
            }
        };

        for (int i = 1; i <= parties; i++) {
            new Thread(task, "Worker-" + i).start();
        }
    }
}
```

**Expected Output**

```
Worker-1 working on phase 1
Worker-2 working on phase 1
Worker-3 working on phase 1
--- All threads reached barrier ---
Worker-1 proceeding past phase 1
Worker-2 proceeding past phase 1
Worker-3 proceeding past phase 1
Worker-1 working on phase 2
...
```

**Why This Output Occurs**

All three workers perform phase 1 at different speeds. When each finishes, it calls `barrier.await()`. The barrier action runs only after the last worker arrives. Then all threads proceed to phase 2 together. The barrier resets automatically and is ready for the next phase. This is the classic phased computation pattern.

### Real-World Cases

- **Parallel Merge Sort**: `CyclicBarrier` synchronizes threads between sorting and merging phases.
- **Game Loop**: A game engine uses `CyclicBarrier` to wait for all physics, AI, and rendering threads before advancing the frame.
- **Start Gate for Benchmarks**: `CountDownLatch` ensures all benchmark threads start at the same instant for fair comparison.
- **Service Startup**: A `CountDownLatch` waits for all services to report ready before the application accepts traffic.

---

## Core Concept 6: Low-Level Lock-Free Atomic Scaling — Atomic Variables and LongAdder

### Definitions

**Core Definition**
Atomic variables are classes in `java.util.concurrent.atomic` that support lock-free thread-safe programming on single variables. `LongAdder` is a specialized atomic class optimized for high-contention scenarios where updates are frequent and reads are infrequent.

**Technical Definition**
The `java.util.concurrent.atomic` package provides a small toolkit of classes that support lock-free thread-safe programming on single variables. `AtomicInteger` is an `int` value that may be updated atomically. `LongAdder` maintains one or more variables that together maintain an initially zero `long` sum. When updates (method `add`) are contended across threads, the set of variables may grow dynamically to reduce contention. Method `sum()` (or equivalently, `longValue()`) returns the current total sum; the returned value is not an atomic snapshot. Atomic classes use hardware-level compare-and-swap (CAS) instructions—on x86, a single `LOCK XADD` instruction—to achieve atomicity without locks.

**Beginner-Friendly Explanation**
`AtomicInteger` is like a single shared counter with a "compare and swap" rule: "If the counter is still 5, set it to 6; otherwise, try again." `LongAdder` is like having multiple counters in different rooms. Each thread updates its own counter, and when you want the total, you add up all the counters. This is much faster when many threads are updating simultaneously, because they don't all fight over the same counter.

### Purposes

- To provide atomic operations on single variables without using locks, avoiding blocking and deadlock.
- To enable lock-free counters, sequence generators, and state flags.
- To provide high-throughput counters via `LongAdder` for statistics and metrics collection.
- To support compare-and-set loops for building custom lock-free data structures.

### Syntax Rules and Structure

**Complete General Syntax — AtomicInteger**

```java
AtomicInteger counter = new AtomicInteger(0);

int value = counter.get();                  // volatile read
counter.set(10);                             // volatile write
int newValue = counter.incrementAndGet();    // atomic increment
int oldValue = counter.getAndIncrement();    // atomic get-then-increment
boolean swapped = counter.compareAndSet(10, 20); // CAS
int updated = counter.updateAndGet(x -> x * 2);  // atomic update
```

**Complete General Syntax — LongAdder**

```java
LongAdder adder = new LongAdder();

adder.add(5);            // atomic add (contention-reducing)
adder.increment();       // equivalent to add(1)
long total = adder.sum(); // current total (not atomic snapshot)
adder.reset();           // reset to zero
```

**Component Breakdown**

| Class | Method | Description |
|-------|--------|-------------|
| `AtomicInteger` | `incrementAndGet()` | Atomically increments and returns new value |
| `AtomicInteger` | `compareAndSet(expect, update)` | CAS: sets to update if current equals expect |
| `AtomicInteger` | `getAndUpdate(fn)` | Atomically applies function and returns old value |
| `LongAdder` | `add(long)` | Adds value, may distribute across cells |
| `LongAdder` | `sum()` | Returns current total (not atomic) |
| `LongAdder` | `reset()` | Resets to zero |

**Syntax Rules**

1. Atomic classes use CAS loops internally; they do not use locks.
2. `compareAndSet(expected, update)` returns `true` only if the current value equals `expected` and the update was successful.
3. `LongAdder.sum()` is not an atomic snapshot; it is approximate under concurrent updates.
4. `LongAdder` is preferable to `AtomicLong` when writes are much more frequent than reads and high throughput is needed.
5. Atomic classes support `getAndUpdate`, `updateAndGet`, `getAndAccumulate`, and `accumulateAndGet` for functional atomic updates.

**Constraints and Limitations**

- Atomic variables protect only a single variable; they do not protect multiple related variables.
- `compareAndSet` may suffer from the ABA problem (a value changes A→B→A; use `AtomicStampedReference` to detect).
- `LongAdder.sum()` is not atomic; do not use it for precise synchronization.
- Atomic classes are not a replacement for `Integer`; they are specialized for concurrent counters.

### Annotated Code Examples

**Example 1: AtomicInteger Counter**

```java
import java.util.concurrent.atomic.AtomicInteger;

public class AtomicCounterDemo {
    private final AtomicInteger counter = new AtomicInteger(0);

    public void increment() {
        counter.incrementAndGet(); // lock-free atomic increment
    }

    public int get() {
        return counter.get(); // volatile read
    }

    public static void main(String[] args) throws InterruptedException {
        AtomicCounterDemo demo = new AtomicCounterDemo();
        Runnable task = () -> {
            for (int i = 0; i < 100000; i++) demo.increment();
        };

        Thread t1 = new Thread(task);
        Thread t2 = new Thread(task);
        t1.start(); t2.start();
        t1.join(); t2.join();
        System.out.println("Counter: " + demo.get()); // 200000
    }
}
```

**Expected Output**

```
Counter: 200000
```

**Why This Output Occurs**

`incrementAndGet()` is implemented as a CAS loop: it reads the current value, computes the new value, and attempts to set it. If another thread changed the value in between, the CAS fails and the loop retries. No locks are used, so neither thread ever blocks. The final count is exactly 200,000.

---

**Example 2: LongAdder vs. AtomicLong Under Contention**

```java
import java.util.concurrent.atomic.*;

public class LongAdderVsAtomicLong {
    public static void main(String[] args) throws InterruptedException {
        int threads = 8;
        int iterations = 1_000_000;

        // AtomicLong
        AtomicLong atomicLong = new AtomicLong(0);
        long t1 = runBenchmark(atomicLong::incrementAndGet, threads, iterations);

        // LongAdder
        LongAdder longAdder = new LongAdder();
        long t2 = runBenchmark(longAdder::increment, threads, iterations);

        System.out.println("AtomicLong time: " + t1 + " ms, value: "
            + atomicLong.get());
        System.out.println("LongAdder time: " + t2 + " ms, value: "
            + longAdder.sum());
    }

    static long runBenchmark(Runnable action, int threads, int iterations)
            throws InterruptedException {
        ExecutorService pool = Executors.newFixedThreadPool(threads);
        long start = System.currentTimeMillis();

        for (int i = 0; i < threads; i++) {
            pool.submit(() -> {
                for (int j = 0; j < iterations; j++) action.run();
            });
        }

        pool.shutdown();
        pool.awaitTermination(30, TimeUnit.SECONDS);
        return System.currentTimeMillis() - start;
    }
}
```

**Expected Output (timing varies)**

```
AtomicLong time: 420 ms, value: 8000000
LongAdder time: 95 ms, value: 8000000
```

**Why This Output Occurs**

`AtomicLong` uses a single CAS target, so all threads contend for the same value, causing many retries. `LongAdder` distributes updates across multiple cells (one per thread in the best case), eliminating contention. The total is the sum of all cells. `LongAdder` is typically 4–10× faster under high contention.

### Real-World Cases

- **Metrics Collection**: `LongAdder` counts requests, errors, and bytes transferred in high-throughput servers.
- **Sequence Generators**: `AtomicLong` generates unique IDs in concurrent systems.
- **Lock-Free Stacks/Queues**: `AtomicReference` manages linked nodes in lock-free data structures.
- **State Flags**: `AtomicBoolean` controls one-time initialization or shutdown flags.

---

## Comparison Summary

| Utility | Purpose | Reentrant? | Fairness? | Blocking? | Best For |
|---------|---------|------------|-----------|-----------|----------|
| `synchronized` | Implicit mutual exclusion | Yes | No | Yes | Simple critical sections |
| `ReentrantLock` | Explicit mutual exclusion | Yes | Optional | Yes | Advanced lock features |
| `ReentrantReadWriteLock` | Read/write separation | Yes | Optional | Yes | Read-heavy workloads |
| `StampedLock` | Read/write + optimistic | No | No | Yes | High-throughput read-heavy |
| `Semaphore` | Permit-based throttling | N/A | Optional | Yes | Resource pools, rate limiting |
| `CountDownLatch` | One-time task completion | N/A | N/A | Yes | Start gates, completion waits |
| `CyclicBarrier` | Reusable phase synchronization | N/A | N/A | Yes | Phased computation |
| `AtomicInteger` | Lock-free single-variable atomic | N/A | N/A | No | Counters, flags, CAS loops |
| `LongAdder` | High-contention counter | N/A | N/A | No | Metrics, statistics |

---

## References

- Lock (Java SE 10 & JDK 10) - https://docs.oracle.com/javase/jp/10/docs/api/java/util/concurrent/locks/Lock.html
- ReentrantLock (Java SE 8) - https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/ReentrantLock.html
- ReentrantReadWriteLock (Java SE 8) - https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/ReentrantReadWriteLock.html
- StampedLock (Java SE 8) - https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/StampedLock.html
- Semaphore (Java SE 21 & JDK 21) - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/Semaphore.html
- CountDownLatch (Java SE 21 & JDK 21) - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/CountDownLatch.html
- CyclicBarrier (Java SE 21 & JDK 21) - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/CyclicBarrier.html
- AtomicInteger (Java SE 24) - https://docs.oracle.com/en/java/javase/24/docs/api/java.base/java/util/concurrent/atomic/AtomicInteger.html
- LongAdder (Java SE 21 & JDK 21) - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/atomic/LongAdder.html
- Package java.util.concurrent.atomic - https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/atomic/package-summary.html
- Java CyclicBarrier vs CountDownLatch (Baeldung) - https://www.baeldung.com/java-cyclicbarrier-countdownlatch
- Java Concurrency in Practice (Chapter 13: Explicit Locks, Chapter 14: Building Custom Synchronizers) - https://www.oreilly.com/library/view/java-concurrency-in/0321349601/