# Java Locks: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** The `java.util.concurrent.locks` package provides a framework for locking and waiting for conditions that is distinct from built-in synchronization and monitors. Lock implementations provide more extensive locking operations than can be obtained using `synchronized` methods and statements, allowing more flexible structuring, different properties, and multiple associated `Condition` objects.

**Technical Definition:** "A lock is a tool for controlling access to a shared resource by multiple threads. Commonly, a lock provides exclusive access to a shared resource: only one thread at a time can acquire the lock and all access to the shared resource requires that the lock be acquired first. However, some locks may allow concurrent access to a shared resource, such as the read lock of a `ReadWriteLock`". The `Lock` interface provides methods for acquiring (`lock()`, `lockInterruptibly()`, `tryLock()`) and releasing (`unlock()`) locks, as well as creating `Condition` objects for thread coordination.

**Beginner-Friendly Explanation:** Imagine a busy restaurant with a single restroom. Without any rules, two people might try to enter at the same time, causing chaos. A lock is like the lock on the restroom door — only one person can enter at a time, and everyone else must wait. The `synchronized` keyword is like a simple lock built into the door. The `Lock` interface is like a more advanced lock system: you can try the door without waiting (`tryLock()`), wait with a timeout, or be interrupted while waiting (`lockInterruptibly()`). Different types of locks serve different purposes: `ReentrantLock` is the standard lock, `ReentrantReadWriteLock` allows multiple readers or one writer, and `StampedLock` is an even more optimised version for read-heavy workloads.

### Key Characteristics

- **Explicit Locking:** Unlike `synchronized`, locks must be explicitly acquired and released, typically using `try/finally` blocks.
- **Flexible Acquisition:** Locks support non-blocking attempts (`tryLock()`), interruptible acquisition (`lockInterruptibly()`), and timeout-based acquisition (`tryLock(long, TimeUnit)`).
- **Multiple Conditions:** A `Lock` can have multiple associated `Condition` objects, giving the effect of having multiple wait-sets per object.
- **Reentrancy:** `ReentrantLock` and `ReentrantReadWriteLock` support reentrant locking — a thread can acquire the same lock multiple times.
- **Read-Write Separation:** `ReentrantReadWriteLock` maintains a pair of associated locks, one for read-only operations and one for writing.
- **Optimistic Reads:** `StampedLock` supports optimistic reads that don't block writers, improving throughput in read-heavy scenarios.
- **Fairness Option:** `ReentrantLock` and `ReentrantReadWriteLock` can be created with a fairness policy (FIFO wait queues).

### Prerequisites

- Solid understanding of Java threads and the `Thread` class
- Familiarity with `synchronized` and intrinsic locks
- Knowledge of the `java.util.concurrent` package
- Understanding of `try/finally` blocks for resource management

### Related Programming Areas

- **Concurrency:** The broader topic of multithreaded programming.
- **Synchronization:** `synchronized` provides implicit locking with less flexibility.
- **Atomic Classes:** Lock-free alternatives for single-variable operations.
- **Concurrent Collections:** `ConcurrentHashMap` and other thread-safe collections.
- **Thread Pools:** `ExecutorService` for managing worker threads.
- **Condition Variables:** `Condition` for inter-thread communication with locks.

### Core Concepts / Features

---

## 1. `Lock` Interface

### Definitions

**Core Definition:** The `Lock` interface is the fundamental abstraction for locking in `java.util.concurrent.locks`, providing methods for acquiring and releasing locks with more flexibility than `synchronized`.

**Technical Definition:** "Lock implementations provide more extensive locking operations than can be obtained using synchronized methods and statements. They allow more flexible structuring, may have quite different properties, and may support multiple associated Condition objects". "The use of synchronized methods or statements provides access to the implicit monitor lock associated with every object, but forces all lock acquisition and release to occur in a block-structured way: when multiple locks are acquired they must be released in the opposite order, and all locks must be released in the same lexical scope in which they were acquired". Lock implementations enable hand-over-hand locking, where a lock can be acquired and released in different scopes, and multiple locks can be acquired and released in any order.

**Beginner-Friendly Explanation:** The `Lock` interface is like a contract that says "I can lock and unlock things, but I'm more flexible than `synchronized`." Different lock implementations (like `ReentrantLock`) fulfil this contract in different ways. The key advantage is that you can try to acquire a lock without waiting, wait with a timeout, or be interrupted while waiting.

### Purposes

- To provide flexible locking that is not restricted to block-structured scoping
- To allow non-blocking, interruptible, and timeout-based lock acquisition
- To support multiple `Condition` objects for fine-grained thread coordination
- To enable advanced locking techniques such as hand-over-hand locking
- To serve as the foundation for higher-level synchronisers

### Syntax Rules and Structure

**Complete General Syntax:**

```java
Lock lock = new ReentrantLock();
lock.lock();
try {
    // critical section
} finally {
    lock.unlock();
}
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `lock()` | Acquires the lock, blocking until available |
| `lockInterruptibly()` | Acquires the lock, allowing interruption |
| `tryLock()` | Acquires the lock only if available immediately |
| `tryLock(long, TimeUnit)` | Acquires the lock if available within the timeout |
| `unlock()` | Releases the lock |
| `newCondition()` | Returns a `Condition` bound to this lock |

**Syntax Rules:**

- "The absence of block-structured locking removes the automatic release of locks that occurs with synchronized methods and statements. In most cases, the following idiom should be used: `l.lock(); try { ... } finally { l.unlock(); }`".
- The `unlock()` method must be called in a `finally` block to ensure the lock is released even if an exception occurs.
- "Lock instances are just normal objects and can themselves be used as the target in a synchronized statement. Acquiring the monitor lock of a Lock instance has no specified relationship with invoking any of the lock() methods of that instance. It is recommended that to avoid confusion you never use Lock instances in this way".

**Constraints and Limitations:**

- "Synchronizing on the intrinsic lock of high-level concurrency utilities can cause non-deterministic behavior because the class can end up with two different locking policies".
- "Do not synchronize on the intrinsic locks of high-level concurrency objects" (SEI CERT LCK03-J).
- Lock implementations may have different fairness, reentrancy, and ordering properties.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Basic Lock Usage**

```java
// File: BasicLockDemo.java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class BasicLockDemo {
    private static int counter = 0;
    private static final Lock lock = new ReentrantLock();

    public static void increment() {
        lock.lock();  // Acquire the lock
        try {
            counter++;  // Critical section
        } finally {
            lock.unlock();  // Always release in finally
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Runnable task = () -> {
            for (int i = 0; i < 10000; i++) {
                increment();
            }
        };

        Thread t1 = new Thread(task);
        Thread t2 = new Thread(task);
        t1.start();
        t2.start();
        t1.join();
        t2.join();

        System.out.println("Counter: " + counter);  // Always 20000
    }
}
```

**Expected Output:**
```
Counter: 20000
```

**Step-by-Step Explanation:**

1. A `ReentrantLock` is created as a static final field.
2. `lock.lock()` acquires the lock before entering the critical section.
3. The `counter++` operation is protected by the lock.
4. `lock.unlock()` is called in the `finally` block, ensuring the lock is released even if an exception occurs.
5. Two threads each increment 10,000 times; the lock ensures no lost updates.

### Real-World Cases

- **Custom Synchronizers:** Building advanced concurrency utilities.
- **Hand-Over-Hand Locking:** Traversing linked data structures while releasing locks on previous nodes.
- **Interruptible Tasks:** Allowing tasks to be cancelled while waiting for a lock.

---

## 2. `ReentrantLock`

### Definitions

**Core Definition:** `ReentrantLock` is a reentrant mutual exclusion lock with the same basic behaviour and semantics as the implicit monitor lock accessed using `synchronized` methods and statements, but with extended capabilities.

**Technical Definition:** "A reentrant mutual exclusion Lock with the same basic behavior and semantics as the implicit monitor lock accessed using synchronized methods and statements, but with extended capabilities". "Reentrant" means that the thread holding the lock can reacquire it without deadlock. `ReentrantLock` supports fairness policies (FIFO wait queues), and provides methods like `tryLock()`, `lockInterruptibly()`, and `tryLock(long, TimeUnit)`.

**Beginner-Friendly Explanation:** `ReentrantLock` is the standard lock in Java. It works like `synchronized` — only one thread can hold the lock at a time — but it's more flexible. You can try to acquire it without waiting, wait with a timeout, or be interrupted. "Reentrant" means that if a thread already holds the lock, it can acquire it again without getting stuck.

### Purposes

- To provide a flexible alternative to `synchronized` with advanced features
- To support reentrant locking for recursive algorithms
- To enable fair locking (FIFO ordering) when needed
- To allow interruptible and timeout-based lock acquisition
- To serve as the foundation for `Condition`-based coordination

### Syntax Rules and Structure

**Complete General Syntax:**

```java
ReentrantLock lock = new ReentrantLock();           // Non-fair (default)
ReentrantLock fairLock = new ReentrantLock(true);   // Fair

lock.lock();
try {
    // critical section
} finally {
    lock.unlock();
}
```

**Key Methods:**

| Method | Description |
|--------|-------------|
| `lock()` | Acquires the lock |
| `unlock()` | Releases the lock |
| `tryLock()` | Acquires only if available |
| `tryLock(long, TimeUnit)` | Acquires if available within timeout |
| `lockInterruptibly()` | Acquires, allowing interruption |
| `isHeldByCurrentThread()` | Tests if current thread holds the lock |
| `getHoldCount()` | Returns the hold count for the current thread |
| `newCondition()` | Creates a `Condition` for this lock |

**Syntax Rules:**

- `ReentrantLock` must be explicitly unlocked in a `finally` block.
- Fair locks (`new ReentrantLock(true)`) guarantee FIFO ordering but may have lower throughput.
- "Reentrant" means the same thread can acquire the lock multiple times; each `lock()` must be matched with an `unlock()`.

### Annotated Complete Step-by-Step Code Examples

```java
// File: ReentrantLockDemo.java
import java.util.concurrent.locks.ReentrantLock;

public class ReentrantLockDemo {
    private static final ReentrantLock lock = new ReentrantLock();
    private static int count = 0;

    public static void outerMethod() {
        lock.lock();
        try {
            System.out.println("Outer: hold count = " + lock.getHoldCount());
            innerMethod();  // Reentrant acquisition
        } finally {
            lock.unlock();
        }
    }

    private static void innerMethod() {
        lock.lock();  // Same thread acquires again
        try {
            count++;
            System.out.println("Inner: hold count = " + lock.getHoldCount());
        } finally {
            lock.unlock();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(ReentrantLockDemo::outerMethod, "Thread-1");
        Thread t2 = new Thread(ReentrantLockDemo::outerMethod, "Thread-2");

        t1.start();
        t2.start();
        t1.join();
        t2.join();

        System.out.println("Final count: " + count);
    }
}
```

**Expected Output (order may vary):**
```
Outer: hold count = 1
Inner: hold count = 2
Outer: hold count = 1
Inner: hold count = 2
Final count: 2
```

**Step-by-Step Explanation:**

1. `outerMethod()` acquires the lock (`hold count = 1`).
2. `innerMethod()` acquires the same lock again (`hold count = 2`) — reentrancy.
3. The inner method releases one hold; the outer method releases the final hold.
4. The next thread then acquires the lock.
5. The counter correctly reaches 2.

### Real-World Cases

- **Recursive Algorithms:** Tree traversals with locked nodes.
- **Fair Scheduling:** Ensuring FIFO access to a shared resource.
- **Interruptible Tasks:** Allowing tasks to be cancelled while waiting for a lock.

---

## 3. `ReentrantReadWriteLock`

### Definitions

**Core Definition:** `ReentrantReadWriteLock` maintains a pair of associated locks, one for read-only operations and one for writing, allowing multiple readers or a single writer but never both.

**Technical Definition:** "A ReadWriteLock maintains a pair of associated locks, one for read-only operations and one for writing. The read lock may be held simultaneously by multiple reader threads, so long as there are no writers. The write lock is exclusive". "Read-write locks allow shared state to be accessed by multiple readers or a single writer but never both". `ReentrantReadWriteLock` supports lock downgrading (holding the write lock, then acquiring the read lock, then releasing the write lock) but not upgrading (holding the read lock, then trying to acquire the write lock).

**Beginner-Friendly Explanation:** Imagine a library with a rare book. Many people can read the book at the same time (readers), but if someone wants to edit the book, they need exclusive access (writer). `ReentrantReadWriteLock` allows this: multiple readers can access the resource simultaneously, but a writer must have exclusive access. This is ideal for data structures that are read frequently but written rarely.

### Purposes

- To allow concurrent read access while ensuring exclusive write access
- To improve throughput in read-heavy scenarios compared to exclusive locks
- To support lock downgrading for safe state transitions
- To provide a structured way to manage read and write access to shared resources

### Syntax Rules and Structure

```java
ReentrantReadWriteLock rwLock = new ReentrantReadWriteLock();
Lock readLock = rwLock.readLock();
Lock writeLock = rwLock.writeLock();

// Read operation
readLock.lock();
try {
    // read shared data
} finally {
    readLock.unlock();
}

// Write operation
writeLock.lock();
try {
    // modify shared data
} finally {
    writeLock.unlock();
}
```

**Key Methods:**

| Method | Description |
|--------|-------------|
| `readLock()` | Returns the read lock |
| `writeLock()` | Returns the write lock |
| `isWriteLocked()` | Tests if the write lock is held |
| `getReadLockCount()` | Returns the number of read locks held |

**Syntax Rules:**

- Multiple threads can hold the read lock simultaneously if no writer holds the write lock.
- The write lock is exclusive — no other thread can hold either lock while a writer holds the write lock.
- Lock downgrading is allowed; lock upgrading is not.

### Annotated Complete Step-by-Step Code Examples

```java
// File: ReadWriteLockDemo.java
import java.util.concurrent.locks.ReentrantReadWriteLock;

public class ReadWriteLockDemo {
    private static final ReentrantReadWriteLock rwLock = new ReentrantReadWriteLock();
    private static int data = 0;

    public static int readData() {
        rwLock.readLock().lock();
        try {
            System.out.println(Thread.currentThread().getName() +
                " reading: " + data);
            return data;
        } finally {
            rwLock.readLock().unlock();
        }
    }

    public static void writeData(int value) {
        rwLock.writeLock().lock();
        try {
            data = value;
            System.out.println(Thread.currentThread().getName() +
                " writing: " + data);
        } finally {
            rwLock.writeLock().unlock();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        // Multiple readers can run concurrently
        Thread r1 = new Thread(ReadWriteLockDemo::readData, "Reader-1");
        Thread r2 = new Thread(ReadWriteLockDemo::readData, "Reader-2");
        Thread w1 = new Thread(() -> writeData(42), "Writer-1");

        r1.start();
        r2.start();
        w1.start();

        r1.join();
        r2.join();
        w1.join();
    }
}
```

**Expected Output (order may vary):**
```
Reader-1 reading: 0
Reader-2 reading: 0
Writer-1 writing: 42
```

**Step-by-Step Explanation:**

1. Two reader threads acquire the read lock concurrently.
2. Both read the data (0) simultaneously — read locks are shared.
3. The writer thread acquires the write lock, blocking all readers.
4. The writer modifies the data and releases the write lock.
5. Readers can resume if they wish.

### Real-World Cases

- **Caching Systems:** Many reads, occasional cache invalidation.
- **Configuration Management:** Frequent reads, infrequent updates.
- **Database Access:** Read-heavy workloads with periodic writes.

---

## 4. `StampedLock`

### Definitions

**Core Definition:** `StampedLock` is a capability-based lock with three modes for controlling read/write access: writing, reading, and optimistic reading.

**Technical Definition:** "A capability-based lock with three modes for controlling read/write access". `StampedLock` was introduced in Java 8 and is more efficient than `ReentrantReadWriteLock`. It supports optimistic reads: a read that doesn't block writers, with a stamp returned that must be validated. "StampedLock supports upgrading readLock to writeLock using method `tryConvertToWriteLock(long)`". It solves the write starvation problem of `ReentrantReadWriteLock` where many readers can starve writers.

**Beginner-Friendly Explanation:** `StampedLock` is like a super-fast version of `ReentrantReadWriteLock`. The key innovation is "optimistic reading" — you can read data without locking, and then check if your read was valid (i.e., no writer intervened). If it was valid, you got your data without any blocking. If not, you can retry with a real read lock. This is extremely efficient for read-heavy workloads. The "stamp" is like a receipt — it proves you held the lock at a particular time.

### Purposes

- To provide higher throughput than `ReentrantReadWriteLock` in read-heavy scenarios
- To support optimistic reads that don't block writers
- To avoid writer starvation in read-heavy workloads
- To allow lock conversion between read and write modes

### Syntax Rules and Structure

```java
StampedLock lock = new StampedLock();

// Write lock
long stamp = lock.writeLock();
try {
    // modify
} finally {
    lock.unlockWrite(stamp);
}

// Read lock
long stamp = lock.readLock();
try {
    // read
} finally {
    lock.unlockRead(stamp);
}

// Optimistic read
long stamp = lock.tryOptimisticRead();
int current = data;  // read data
if (!lock.validate(stamp)) {
    // Read was invalid — a writer intervened
    stamp = lock.readLock();
    try {
        current = data;
    } finally {
        lock.unlockRead(stamp);
    }
}
```

**Key Methods:**

| Method | Description |
|--------|-------------|
| `writeLock()` | Acquires the write lock, returns a stamp |
| `readLock()` | Acquires the read lock, returns a stamp |
| `tryOptimisticRead()` | Returns a stamp for optimistic reading |
| `validate(long stamp)` | Validates an optimistic read stamp |
| `unlockWrite(long stamp)` | Releases the write lock |
| `unlockRead(long stamp)` | Releases the read lock |
| `tryConvertToWriteLock(long stamp)` | Converts a read stamp to a write stamp |

**Syntax Rules:**

- `StampedLock` is not reentrant — a thread cannot acquire the same lock twice.
- Stamps must be used to unlock; they are not interchangeable.
- Optimistic reads must be validated before using the data.
- "StampedLock is not reentrant, so it cannot be used in recursive algorithms".

### Annotated Complete Step-by-Step Code Examples

```java
// File: StampedLockDemo.java
import java.util.concurrent.locks.StampedLock;

public class StampedLockDemo {
    private static final StampedLock lock = new StampedLock();
    private static int data = 0;

    // Optimistic read
    public static int optimisticRead() {
        long stamp = lock.tryOptimisticRead();
        int current = data;  // Read without locking
        if (!lock.validate(stamp)) {
            // A writer intervened — fall back to a real read lock
            stamp = lock.readLock();
            try {
                current = data;
            } finally {
                lock.unlockRead(stamp);
            }
        }
        return current;
    }

    // Write
    public static void write(int value) {
        long stamp = lock.writeLock();
        try {
            data = value;
            System.out.println("Written: " + data);
        } finally {
            lock.unlockWrite(stamp);
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Thread w = new Thread(() -> write(42), "Writer");
        Thread r = new Thread(() -> {
            System.out.println("Read: " + optimisticRead());
        }, "Reader");

        r.start();
        w.start();
        r.join();
        w.join();
    }
}
```

**Expected Output (order may vary):**
```
Read: 0
Written: 42
```

**Step-by-Step Explanation:**

1. The reader thread performs an optimistic read (no locking).
2. The writer thread may acquire the write lock and modify the data.
3. If the writer intervenes, `validate()` returns `false`, and the reader falls back to a real read lock.
4. The reader gets consistent data.

### Real-World Cases

- **High-Performance Caching:** Read-mostly data with occasional updates.
- **Real-Time Systems:** Low-latency reads without blocking writers.
- **Concurrent Data Structures:** Building custom lock-free or near-lock-free structures.

---

## 5. `Condition`

### Definitions

**Core Definition:** `Condition` factors out the `Object` monitor methods (`wait`, `notify`, and `notifyAll`) into distinct objects to give the effect of having multiple wait-sets per object, by combining them with the use of arbitrary `Lock` implementations.

**Technical Definition:** "Condition factors out the Object monitor methods (wait, notify and notifyAll) into distinct objects to give the effect of having multiple wait-sets per object, by combining them with the use of arbitrary Lock implementations. Where a Lock replaces the use of synchronized methods and statements, a Condition replaces the use of the Object monitor methods". "Conditions (also known as condition queues or condition variables) provide a means for one thread to suspend execution (to 'wait') until notified by another thread that some state condition may now be true". "The key property that waiting for a condition provides is that it atomically releases the associated lock and suspends the current thread, just like Object.wait".

**Beginner-Friendly Explanation:** `Condition` is like `wait()` and `notify()` but more powerful. With `synchronized`, you have only one wait-set per object. With `Lock` and `Condition`, you can have multiple wait-sets — for example, one for "buffer not full" and one for "buffer not empty." This allows more precise signalling: you can wake up only the threads waiting for a specific condition.

### Purposes

- To provide multiple wait-sets per lock for fine-grained thread coordination
- To replace `Object.wait()`, `notify()`, and `notifyAll()` in lock-based synchronisation
- To enable precise signalling of specific thread groups
- To support interruptible and timeout-based waiting

### Syntax Rules and Structure

```java
Lock lock = new ReentrantLock();
Condition condition = lock.newCondition();

// Waiting
lock.lock();
try {
    while (!conditionMet) {
        condition.await();  // Atomically releases lock and waits
    }
    // proceed
} finally {
    lock.unlock();
}

// Signalling
lock.lock();
try {
    conditionMet = true;
    condition.signal();    // Wake one waiting thread
    // or condition.signalAll();  // Wake all
} finally {
    lock.unlock();
}
```

**Key Methods:**

| Method | Description |
|--------|-------------|
| `await()` | Waits until signalled or interrupted |
| `await(long, TimeUnit)` | Waits with a timeout |
| `awaitUninterruptibly()` | Waits without responding to interrupts |
| `signal()` | Wakes one waiting thread |
| `signalAll()` | Wakes all waiting threads |

**Syntax Rules:**

- `Condition` methods must be called while holding the associated lock.
- "The key property that waiting for a condition provides is that it atomically releases the associated lock and suspends the current thread".
- Always use `while` (not `if`) before `await()` to guard against spurious wakeups.
- "It is safer to always `signalAll()`, even if this wakes threads up from the 'wait set' unnecessarily".

### Annotated Complete Step-by-Step Code Examples

```java
// File: ConditionDemo.java
import java.util.concurrent.locks.*;

public class ConditionDemo {
    private final Lock lock = new ReentrantLock();
    private final Condition notFull = lock.newCondition();
    private final Condition notEmpty = lock.newCondition();
    private final int[] buffer = new int[5];
    private int count = 0, in = 0, out = 0;

    public void put(int value) throws InterruptedException {
        lock.lock();
        try {
            while (count == buffer.length) {
                notFull.await();  // Wait until buffer has space
            }
            buffer[in] = value;
            in = (in + 1) % buffer.length;
            count++;
            System.out.println("Put: " + value + " (count=" + count + ")");
            notEmpty.signal();  // Signal that buffer is not empty
        } finally {
            lock.unlock();
        }
    }

    public int take() throws InterruptedException {
        lock.lock();
        try {
            while (count == 0) {
                notEmpty.await();  // Wait until buffer has data
            }
            int value = buffer[out];
            out = (out + 1) % buffer.length;
            count--;
            System.out.println("Take: " + value + " (count=" + count + ")");
            notFull.signal();  // Signal that buffer has space
            return value;
        } finally {
            lock.unlock();
        }
    }

    public static void main(String[] args) {
        ConditionDemo buffer = new ConditionDemo();

        Thread producer = new Thread(() -> {
            for (int i = 1; i <= 5; i++) {
                try { buffer.put(i); Thread.sleep(100); }
                catch (InterruptedException e) { Thread.currentThread().interrupt(); }
            }
        }, "Producer");

        Thread consumer = new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                try { buffer.take(); Thread.sleep(150); }
                catch (InterruptedException e) { Thread.currentThread().interrupt(); }
            }
        }, "Consumer");

        producer.start();
        consumer.start();
    }
}
```

**Expected Output (interleaved):**
```
Put: 1 (count=1)
Take: 1 (count=0)
Put: 2 (count=1)
Take: 2 (count=0)
...
```

**Step-by-Step Explanation:**

1. Two `Condition` objects are created: `notFull` (for producers) and `notEmpty` (for consumers).
2. Producers wait on `notFull` when the buffer is full; consumers wait on `notEmpty` when empty.
3. `notEmpty.signal()` wakes a waiting consumer; `notFull.signal()` wakes a waiting producer.
4. This provides precise signalling, unlike `Object.notifyAll()` which wakes all threads.

### Real-World Cases

- **Producer-Consumer Buffers:** Coordinating producers and consumers.
- **Thread Pools:** Workers waiting for tasks; the pool signalling when tasks arrive.
- **Connection Pools:** Threads waiting for available connections.

---

## 6. Lock vs synchronized

### Definitions

**Core Definition:** `Lock` and `synchronized` are both mechanisms for providing mutual exclusion in Java, but they differ in flexibility, features, and usage patterns.

**Technical Definition:** "The use of synchronized methods or statements provides access to the implicit monitor lock associated with every object, but forces all lock acquisition and release to occur in a block-structured way: when multiple locks are acquired they must be released in the opposite order, and all locks must be released in the same lexical scope in which they were acquired". "Lock implementations provide more extensive locking operations than can be obtained using synchronized methods and statements. They allow more flexible structuring, may have quite different properties, and may support multiple associated Condition objects".

**Beginner-Friendly Explanation:** `synchronized` is the simple, built-in option — like a basic lock on a door. `Lock` is the advanced option — like a smart lock that can be opened with a key, a code, or a phone app. `synchronized` is easier to use but less flexible. `Lock` requires more care (you must unlock it yourself) but gives you more control.

### Comparison Table

| Aspect | `synchronized` | `Lock` |
|--------|----------------|--------|
| **Acquisition** | Implicit, block-structured | Explicit (`lock()`, `tryLock()`, `lockInterruptibly()`) |
| **Release** | Automatic (on block exit) | Manual (in `finally`) |
| **Non-blocking try** | Not possible | `tryLock()` |
| **Timeout** | Not possible | `tryLock(long, TimeUnit)` |
| **Interruptible** | Not possible | `lockInterruptibly()` |
| **Multiple conditions** | One wait-set per object | Multiple `Condition` objects |
| **Fairness** | Not specified | Optional FIFO via `new ReentrantLock(true)` |
| **Lock ordering** | Must be block-structured | Any order, any scope |
| **Reentrancy** | Yes (intrinsic) | Yes (`ReentrantLock`) |
| **Performance** | Good for low contention | Better for high contention (with optimisations) |
| **Ease of use** | Simpler, less error-prone | More complex, requires discipline |

### When to Use Which

**Use `synchronized` when:**

- You need simple, block-structured mutual exclusion
- You don't need advanced features like `tryLock()` or timeouts
- You want automatic lock release (less error-prone)
- The lock scope is naturally block-structured

**Use `Lock` when:**

- You need non-blocking, interruptible, or timeout-based acquisition
- You need multiple `Condition` objects for fine-grained coordination
- You need fair locking (FIFO ordering)
- You need hand-over-hand locking or non-block-structured locking
- You're building advanced concurrency utilities

### Annotated Code Example

```java
// File: LockVsSynchronizedDemo.java
import java.util.concurrent.locks.*;

public class LockVsSynchronizedDemo {
    private final Object syncLock = new Object();
    private final Lock explicitLock = new ReentrantLock();
    private int syncCounter = 0;
    private int lockCounter = 0;

    // synchronized version
    public void incrementSync() {
        synchronized (syncLock) {
            syncCounter++;
        }
    }

    // Lock version with tryLock
    public void incrementLock() {
        if (explicitLock.tryLock()) {  // Non-blocking attempt
            try {
                lockCounter++;
            } finally {
                explicitLock.unlock();
            }
        } else {
            System.out.println("Could not acquire lock — doing something else");
        }
    }

    public static void main(String[] args) throws InterruptedException {
        LockVsSynchronizedDemo demo = new LockVsSynchronizedDemo();

        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 10000; i++) demo.incrementSync();
        });
        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 10000; i++) demo.incrementLock();
        });

        t1.start();
        t2.start();
        t1.join();
        t2.join();

        System.out.println("Sync counter: " + demo.syncCounter);
        System.out.println("Lock counter: " + demo.lockCounter);
    }
}
```

**Expected Output (varies):**
```
Sync counter: 10000
Could not acquire lock — doing something else
Lock counter: 9998
```

**Step-by-Step Explanation:**

1. `incrementSync()` uses `synchronized` — it always waits for the lock.
2. `incrementLock()` uses `tryLock()` — if the lock is unavailable, it does something else instead of waiting.
3. The sync counter always reaches 10,000; the lock counter may be less if `tryLock()` fails.

### Real-World Cases

- **High-Concurrency Systems:** `Lock` for fine-grained control in servers.
- **Simple Synchronization:** `synchronized` for straightforward mutual exclusion.
- **Timeout-Sensitive Operations:** `Lock` when waiting indefinitely is unacceptable.

---

## 7. Lock Best Practices

### Definitions

**Core Definition:** Lock best practices are guidelines for using `Lock` implementations correctly, efficiently, and safely in concurrent Java programs.

**Technical Definition:** Best practices derive from authoritative sources including the SEI CERT Oracle Coding Standard for Java. Key principles include always unlocking in `finally`, not synchronizing on the intrinsic locks of `Lock` objects, using private final lock objects, and preferring higher-level concurrency utilities when possible.

**Beginner-Friendly Explanation:** Lock best practices help you avoid common mistakes like forgetting to unlock (causing deadlock) or using the wrong lock object. They're like a safety manual for using locks correctly.

### Key Best Practices Table

| Practice | Description | Source |
|----------|-------------|--------|
| Always unlock in `finally` | "In most cases, the following idiom should be used: `l.lock(); try { ... } finally { l.unlock(); }`" | Oracle |
| Never synchronize on `Lock` objects | "Do not synchronize on the intrinsic locks of high-level concurrency objects" | SEI CERT |
| Use private final lock objects | "Use private final lock objects to synchronize classes that may interact with untrusted code" | SEI CERT |
| Use the lock's `lock()`/`unlock()` methods | "Instead of using the intrinsic locks of objects that implement the Lock interface, such as ReentrantLock, use the lock() and unlock() methods" | SEI CERT |
| Consider `tryLock` for deadlock avoidance | Non-blocking acquisition prevents deadlock | Best practice |
| Use `Condition` with `while` loops | Guard against spurious wakeups | Best practice |
| Prefer `synchronized` for simple cases | "If there is no requirement for using the advanced functionality... it is better to use the Executor framework or other concurrency primitives" | SEI CERT |
| Document locking policy | Explain which locks protect which state | Effective Java |
| Avoid excessive lock granularity | Too many locks increase complexity | Best practice |
| Use fair locks judiciously | Fairness reduces throughput | Best practice |

### Annotated Complete Step-by-Step Code Examples

```java
// File: LockBestPracticesDemo.java
import java.util.concurrent.locks.*;

public class LockBestPracticesDemo {
    // BEST PRACTICE: Private final lock object
    private final Lock lock = new ReentrantLock();
    private int count = 0;

    public void increment() {
        lock.lock();
        try {
            count++;
        } finally {
            // BEST PRACTICE: Always unlock in finally
            lock.unlock();
        }
    }

    // BEST PRACTICE: Use tryLock for deadlock avoidance
    public boolean tryIncrement() {
        if (lock.tryLock()) {
            try {
                count++;
                return true;
            } finally {
                lock.unlock();
            }
        }
        return false;  // Could not acquire — do something else
    }

    // BEST PRACTICE: Use Condition with while loop
    private final Condition condition = lock.newCondition();
    private boolean ready = false;

    public void waitForReady() throws InterruptedException {
        lock.lock();
        try {
            while (!ready) {  // Always use while, not if
                condition.await();
            }
        } finally {
            lock.unlock();
        }
    }

    public void setReady() {
        lock.lock();
        try {
            ready = true;
            condition.signalAll();  // Safer than signal()
        } finally {
            lock.unlock();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        LockBestPracticesDemo demo = new LockBestPracticesDemo();

        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) demo.increment();
        });
        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) demo.tryIncrement();
        });

        t1.start();
        t2.start();
        t1.join();
        t2.join();

        System.out.println("Count: " + demo.count);  // At least 1000
    }
}
```

**Expected Output:**
```
Count: 2000
```

**Step-by-Step Explanation:**

1. A private final `ReentrantLock` is used for all locking.
2. `increment()` always unlocks in `finally`.
3. `tryIncrement()` uses `tryLock()` to avoid waiting indefinitely.
4. The `Condition` is used with a `while` loop to guard against spurious wakeups.
5. `signalAll()` is used instead of `signal()` for safety.

### Real-World Cases

- **High-Performance Servers:** Using `tryLock` to avoid deadlocks under load.
- **Enterprise Applications:** Consistent locking policies across services.
- **Security-Sensitive Code:** Using private lock objects to prevent interference.

---

## References Links

- Package java.util.concurrent.locks — Java Platform SE 8 API Specification - https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/compact2-package-summary.html
- Interface Lock — Java Platform SE 11 API Specification - https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/locks/Lock.html
- Interface Condition — Java SE 11 API Specification - https://cr.openjdk.org/~iris/se/11/pr/java-se-11-pr-spec-02/api/java.base/java/util/concurrent/locks/Condition.html
- LCK03-J. Do not synchronize on the intrinsic locks of high-level concurrency objects — SEI CERT - https://wiki.sei.cmu.edu/confluence/display/java/LCK03-J.+Do+not+synchronize+on+the+intrinsic+locks+of+high-level+concurrency+objects
- LCK00-J. Use private final lock objects to synchronize classes that may interact with untrusted code — SEI CERT - https://wiki.sei.cmu.edu/confluence/pages/viewpage.action?pageId=88888126
- StampedLock Example — GitHub (alxkm/java-concurrency-patterns) - https://github.com/alxkm/java-concurrency-patterns
- Java Concurrency in Practice — Brian Goetz et al., Addison-Wesley
- Effective Java (3rd Edition) by Joshua Bloch — Chapter 11: Concurrency