# Java Synchronization & Traditional Mutual Exclusion: A Comprehensive Research Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**
Java synchronization is the language-level mechanism for controlling access to shared resources by multiple threads, ensuring that only one thread at a time executes a critical section of code that manipulates shared mutable state.

**Technical Definition**
Synchronization in Java is implemented through *intrinsic locks* (also called *monitor locks* or *monitors*), which are associated with every object. When a thread enters a `synchronized` method or block, it must acquire the monitor associated with the specified object. Only one thread can own a monitor at a time; other threads attempting to acquire the same monitor are blocked until it is released. The Java Memory Model (JLS Chapter 17) defines a *happens-before* relationship: an unlock action on monitor *m* synchronizes-with all subsequent lock actions on *m*, ensuring that memory writes performed by one thread are visible to another thread that subsequently acquires the same lock.

**Beginner-Friendly Explanation**
Imagine a single-occupancy bathroom in a shared apartment. The bathroom door has a lock, and only one person can be inside at a time. When someone is inside, others must wait outside until the door is unlocked. In Java, a `synchronized` block is like that bathroom: only one thread can be "inside" the block at a time, and other threads that try to enter must wait. The "key" to the lock is the object's monitor.

### Key Characteristics

- **Mutual Exclusion**: Only one thread can hold a monitor at a time, ensuring that critical sections execute atomically with respect to other threads competing for the same lock.
- **Reentrancy**: A thread that already holds a monitor can re-enter any synchronized code guarded by the same monitor without deadlocking itself.
- **Happens-Before Guarantee**: A monitor unlock *happens-before* every subsequent lock on the same monitor, ensuring memory visibility of writes made inside synchronized blocks.
- **Cooperative Waiting**: The `wait()`, `notify()`, and `notifyAll()` methods allow threads to communicate about condition changes while inside synchronized code.
- **Lock Object Flexibility**: The `synchronized` keyword can lock on `this` (instance methods), the `Class` object (static methods), or any explicitly specified object (synchronized blocks).
- **`Thread.stop()` Removed**: The historically unsafe `Thread.stop()` method has been deprecated since Java 1.2 and removed in Java 21; cooperative interruption and synchronization-based signaling are the correct alternatives.

### Prerequisites

- Basic understanding of Java threads and the `Thread` class
- Familiarity with the `Runnable` and `Callable` interfaces
- Awareness of shared mutable state and race conditions
- Knowledge of the `Object` class as the root of all Java classes

### Related Programming Areas

- **Java Memory Model**: Defines happens-before relationships that synchronization relies upon.
- **`java.util.concurrent`**: Provides higher-level concurrency utilities (`ReentrantLock`, `Condition`, `AtomicInteger`) that complement or replace intrinsic synchronization.
- **Design Patterns**: Producer–Consumer, Guarded Suspension, and Monitor Object patterns rely on synchronization.
- **Deadlock and Liveness**: Incorrect synchronization can cause deadlock, starvation, or livelock.

### Core Concepts / Features

Five core concepts are covered: (1) critical sections and thread-safe execution boundaries, (2) the `synchronized` keyword (method-level vs. block-level), (3) intrinsic locks (monitors) and reentrancy, (4) strict mutual exclusion, and (5) `wait()`, `notify()`, and `notifyAll()`.

---

## Core Concept 1: Thread-Safe Execution Boundaries — Explicit Declaration of Critical Sections

### Definitions

**Core Definition**
A critical section is a region of code that accesses shared mutable state and must not be executed by more than one thread concurrently. Thread safety means that a class or method behaves correctly when accessed by multiple threads.

**Technical Definition**
Mutual exclusion is the property that at most one thread executes a critical section at any given time. In Java, critical sections are declared using the `synchronized` keyword, which associates the code region with a monitor. Thread-safe code is code that can handle multiple threads using it simultaneously without producing incorrect results or corrupting shared state.

**Beginner-Friendly Explanation**
A critical section is like a narrow bridge that only one car can cross at a time. If two cars try to cross simultaneously, they will crash. In Java, you mark the bridge as `synchronized` to ensure only one "car" (thread) is on it at a time.

### Purposes

- To prevent thread interference when multiple threads read and write shared variables.
- To ensure memory consistency by establishing happens-before relationships between thread actions.
- To protect invariants of shared objects from being violated by concurrent access.
- To provide a clear, language-level declaration of which code regions require exclusive access.

### Syntax Rules and Structure

**Complete General Syntax**

```java
// Synchronized method (instance)
public synchronized void methodName() { }

// Synchronized static method
public static synchronized void methodName() { }

// Synchronized block
synchronized (lockObject) {
    // critical section
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `synchronized` | Keyword declaring mutual exclusion |
| `lockObject` | The object whose monitor is acquired |
| `this` | Implicit lock for instance synchronized methods |
| `ClassName.class` | Implicit lock for static synchronized methods |

**Syntax Rules**

1. Only methods and blocks of code can be synchronized; declaring variables or classes as `synchronized` results in a compilation error.
2. A `synchronized` instance method acquires the monitor of `this`.
3. A `synchronized` static method acquires the monitor of the `Class` object for the declaring class; this lock is distinct from any instance lock.
4. A `synchronized` block acquires the monitor of the specified object; the object must not be `null`.
5. The lock is released when the synchronized block or method exits normally or abruptly (via exception or `return`).

**Constraints and Limitations**

- Synchronization can introduce thread contention, causing slower execution or starvation.
- Holding locks during time-consuming or blocking operations can severely degrade performance and cause deadlock.
- Intrinsic locks are not interruptible (except via `synchronized` block with `ReentrantLock` alternatives).
- Synchronizing on `this` or public objects exposes the lock to untrusted code, enabling denial-of-service attacks.

### Annotated Code Examples

**Example 1: Race Condition Without Synchronization**

```java
public class UnsafeCounter {
    private int count = 0;

    public void increment() {
        count++; // read-modify-write: not atomic!
    }

    public int getCount() {
        return count;
    }

    public static void main(String[] args) throws InterruptedException {
        UnsafeCounter counter = new UnsafeCounter();
        Runnable task = () -> {
            for (int i = 0; i < 10_000; i++) {
                counter.increment();
            }
        };

        Thread t1 = new Thread(task);
        Thread t2 = new Thread(task);
        t1.start(); t2.start();
        t1.join(); t2.join();
        System.out.println("Final count: " + counter.getCount());
    }
}
```

**Expected Output (varies; often less than 20000)**

```
Final count: 17342
```

**Why This Output Occurs**

`count++` is not atomic; it consists of reading `count`, incrementing it, and writing it back. Without synchronization, both threads may read the same value and overwrite each other's increments, resulting in a lost update. The final count is unpredictable and typically less than 20,000.

---

**Example 2: Thread-Safe Counter with Synchronized Block**

```java
public class SafeCounter {
    private int count = 0;
    private final Object lock = new Object(); // private final lock object

    public void increment() {
        synchronized (lock) { // critical section: only one thread at a time
            count++;
        }
    }

    public int getCount() {
        synchronized (lock) {
            return count;
        }
    }

    public static void main(String[] args) throws InterruptedException {
        SafeCounter counter = new SafeCounter();
        Runnable task = () -> {
            for (int i = 0; i < 10_000; i++) {
                counter.increment();
            }
        };

        Thread t1 = new Thread(task);
        Thread t2 = new Thread(task);
        t1.start(); t2.start();
        t1.join(); t2.join();
        System.out.println("Final count: " + counter.getCount());
    }
}
```

**Expected Output**

```
Final count: 20000
```

**Why This Output Occurs**

The `synchronized (lock)` block ensures that only one thread executes `count++` at a time. The `lock` object is `private final`, so external code cannot interfere with it. The happens-before relationship guarantees that all increments are visible to the main thread after both workers terminate.

### Real-World Cases

- **Bank Account Transfers**: A transfer method must debit one account and credit another atomically; without synchronization, concurrent transfers can corrupt balances.
- **Inventory Management**: Decrementing stock must be atomic to prevent overselling.
- **Cached Values**: Lazy initialization of a shared cache requires synchronization to prevent duplicate computation.

---

## Core Concept 2: The `synchronized` Keyword — Method-Level vs. Structural Blocks

### Definitions

**Core Definition**
The `synchronized` keyword can be applied to methods (implicit lock) or to blocks of code (explicit lock), providing two levels of granularity for controlling access to critical sections.

**Technical Definition**
A `synchronized` method automatically performs a lock action when invoked and an unlock action when it completes. A `synchronized` statement (block) acquires the monitor of the specified object, executes the block, and releases the monitor. Per JLS §14.19, a synchronized statement is executed by first evaluating the expression to obtain the object reference, then acquiring the monitor, executing the block, and releasing the monitor.

**Beginner-Friendly Explanation**
Method-level synchronization is like locking the entire house when you go inside—simple but coarse. Block-level synchronization is like locking only the room you need—more precise and allows other threads to use different rooms simultaneously.

### Purposes

- To choose between simple, coarse-grained locking (method-level) and fine-grained, minimal-scope locking (block-level).
- To reduce lock contention by synchronizing only the operations that require mutual exclusion.
- To decouple locking from method structure, allowing atomic operations to be extracted from larger methods.
- To protect static state independently from instance state using separate locks.

### Syntax Rules and Structure

**Complete General Syntax — Method-Level**

```java
// Instance method: locks this
public synchronized void instanceMethod() { }

// Static method: locks ClassName.class
public static synchronized void staticMethod() { }
```

**Complete General Syntax — Block-Level**

```java
// Lock on a specific object
synchronized (lockObject) {
    // critical section
}

// Lock on this
synchronized (this) {
    // critical section
}

// Lock on the Class object
synchronized (ClassName.class) {
    // critical section
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `synchronized` | Keyword declaring mutual exclusion |
| `lockObject` | Explicit object whose monitor is acquired |
| `this` | Instance lock for instance synchronized methods |
| `ClassName.class` | Class-level lock for static synchronized methods |

**Syntax Rules**

1. A `synchronized` method is equivalent to a method whose body is wrapped in `synchronized (this)` for instance methods, or `synchronized (ClassName.class)` for static methods.
2. Static and instance synchronized methods use different monitors; a thread can hold both simultaneously without deadlock.
3. A `synchronized` block can lock on any object reference, including a dedicated lock object.
4. The lock is automatically released when the block exits, even if an exception is thrown.

**Constraints and Limitations**

- Method-level synchronization synchronizes the entire method body, which may include non-critical operations, increasing lock hold time.
- Synchronizing on `this` or `ClassName.class` exposes the lock to external code (subclasses, untrusted code), enabling denial-of-service via lock contention.
- Block-level synchronization requires explicit lock object management and careful documentation of the locking policy.

### Annotated Code Examples

**Example 1: Method-Level Synchronization**

```java
public class MethodSyncCounter {
    private int count = 0;

    public synchronized void increment() { // locks this
        count++;
    }

    public synchronized int getCount() {   // locks this
        return count;
    }
}
```

**Expected Behavior**

Calls to `increment()` and `getCount()` are mutually exclusive across threads. The lock is held for the entire method duration.

---

**Example 2: Block-Level Synchronization with Private Lock**

```java
public class BlockSyncCounter {
    private int count = 0;
    private final Object lock = new Object(); // private final lock object

    public void increment() {
        // Non-critical work outside the lock
        System.out.println("Preparing to increment...");
        synchronized (lock) { // only this block is synchronized
            count++;
        }
        // More non-critical work
    }

    public int getCount() {
        synchronized (lock) {
            return count;
        }
    }
}
```

**Why This Approach Is Preferred**

The `lock` object is `private final`, so no external class can acquire it. Only the `count++` operation is inside the synchronized block, minimizing lock hold time. Non-critical operations (logging, validation) execute outside the lock, improving throughput.

---

**Example 3: Static vs. Instance Lock Separation**

```java
public class DualLockCounter {
    private int instanceCount = 0;
    private static int staticCount = 0;

    public synchronized void incrementInstance() { // locks this
        instanceCount++;
    }

    public static synchronized void incrementStatic() { // locks DualLockCounter.class
        staticCount++;
    }
}
```

**Why This Works**

Instance and static synchronized methods use different monitors. A thread can hold both the instance monitor of an object and the class monitor simultaneously without deadlock.

### Real-World Cases

- **`StringBuffer`**: All methods are `synchronized`, providing thread safety at the cost of performance.
- **`StringBuilder`**: The non-synchronized counterpart, faster but unsafe for concurrent use.
- **`Collections.synchronizedList()`**: Wraps methods in synchronized blocks with a private lock, allowing safe concurrent access while delegating to an unsynchronized backing list.

---

## Core Concept 3: Under-the-Hood Synchronization — Intrinsic Locks, Acquisition, and Reentrancy

### Definitions

**Core Definition**
Every Java object has an associated intrinsic lock (monitor) that is acquired when a thread enters synchronized code guarded by that object and released when the thread exits.

**Technical Definition**
The JVM associates a monitor with each object. When a thread executes a `monitorenter` instruction (via `synchronized`), it attempts to acquire the monitor. If the monitor is unowned, the thread becomes the owner. If the monitor is already owned by another thread, the acquiring thread blocks. If the monitor is owned by the *same* thread, the acquisition succeeds immediately (reentrancy). The monitor maintains a recursion count; the lock is released only when the count returns to zero. Per JLS §17.4.4, a lock action on monitor *m* synchronizes-with all subsequent unlock actions on *m*, establishing the happens-before relationship that guarantees memory visibility.

**Beginner-Friendly Explanation**
Think of an intrinsic lock as a key attached to an object. When you enter a synchronized block, you take the key. If someone else has it, you wait. If you already have it (because you're already inside another synchronized block on the same object), you can enter again—the lock is "reentrant." Each time you leave a synchronized block, you give back one "layer" of the key until the last layer is returned.

### Purposes

- To enforce mutual exclusion at the object level, ensuring that only one thread at a time executes code guarded by a given monitor.
- To support reentrant locking, allowing a thread to call synchronized methods from within other synchronized methods on the same object without deadlocking itself.
- To establish happens-before relationships that guarantee memory visibility of writes made inside synchronized blocks.
- To provide a simple, built-in locking mechanism without requiring explicit lock objects or imports.

### Syntax Rules and Structure

**Complete General Syntax**

```java
// Acquiring a monitor (implicit via synchronized)
synchronized (obj) {
    // monitor of obj is acquired on entry
    // and released on exit
}

// Reentrant acquisition
public synchronized void outer() {
    inner(); // same thread can re-enter
}
public synchronized void inner() { }
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Monitor | The intrinsic lock associated with an object |
| Recursion count | Tracks how many times the same thread has acquired the monitor |
| `monitorenter` | JVM instruction executed when entering synchronized code |
| `monitorexit` | JVM instruction executed when exiting synchronized code |

**Syntax Rules**

1. Every object (except primitives) has an associated monitor.
2. A thread that already owns a monitor can re-acquire it without blocking; this is called reentrancy.
3. The monitor is released only when the recursion count reaches zero (all nested synchronized blocks on the same monitor have exited).
4. An unlock action on monitor *m* happens-before every subsequent lock action on *m* (JLS §17.4.4).
5. If a synchronized method throws an exception, the monitor is automatically released.

**Constraints and Limitations**

- Intrinsic locks are not interruptible; a thread blocked waiting for a monitor cannot be interrupted (except via `ReentrantLock.lockInterruptibly()`).
- There is no way to query whether the current thread holds a particular monitor.
- Intrinsic locks do not support timeouts or fairness policies.
- Synchronizing on publicly accessible objects (`this`, `ClassName.class`, public fields) exposes the lock to external interference.

### Annotated Code Examples

**Example 1: Reentrant Synchronization**

```java
public class ReentrantDemo {
    public synchronized void outer() {
        System.out.println("Entering outer");
        inner(); // calls another synchronized method on this
        System.out.println("Exiting outer");
    }

    public synchronized void inner() {
        System.out.println("Entering inner");
        // do work
        System.out.println("Exiting inner");
    }

    public static void main(String[] args) {
        new ReentrantDemo().outer();
    }
}
```

**Expected Output**

```
Entering outer
Entering inner
Exiting inner
Exiting outer
```

**Why This Output Occurs**

The thread acquires the monitor of `this` when entering `outer()`. When `outer()` calls `inner()`, the same thread attempts to acquire the same monitor again. Because the monitor is reentrant, the acquisition succeeds immediately without blocking. The monitor's recursion count becomes 2, then decreases to 1 when `inner()` exits, and finally to 0 when `outer()` exits.

---

**Example 2: Happens-Before Visibility**

```java
public class VisibilityDemo {
    private int value = 0;
    private boolean ready = false;
    private final Object lock = new Object();

    public void writer() {
        synchronized (lock) {
            value = 42;       // write inside synchronized
            ready = true;     // write inside synchronized
        }
    }

    public void reader() {
        synchronized (lock) {
            if (ready) {
                System.out.println("Value: " + value); // guaranteed to see 42
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        VisibilityDemo demo = new VisibilityDemo();
        Thread writer = new Thread(demo::writer);
        Thread reader = new Thread(() -> {
            try { Thread.sleep(100); } catch (InterruptedException e) { }
            demo.reader();
        });

        writer.start();
        reader.start();
        writer.join();
        reader.join();
    }
}
```

**Expected Output**

```
Value: 42
```

**Why This Output Occurs**

The writer thread writes `value = 42` and `ready = true` inside a synchronized block on `lock`. When it exits the block, the monitor unlock happens-before any subsequent lock of the same monitor. The reader thread acquires the same monitor, so all writes made by the writer before the unlock are visible to the reader. The reader is guaranteed to see `value = 42` when `ready` is `true`.

### Real-World Cases

- **Lazy Initialization**: Double-checked locking (with `volatile`) uses synchronized blocks to ensure a singleton is initialized only once.
- **Producer–Consumer**: Both producers and consumers synchronize on a shared buffer's monitor to coordinate access.
- **`Hashtable`**: All methods are synchronized, using the intrinsic lock of each `Hashtable` instance.

---

## Core Concept 4: Achieving Strict Mutual Exclusion Across Shared Resources

### Definitions

**Core Definition**
Strict mutual exclusion guarantees that at most one thread executes a critical section at any time, protecting shared resources from concurrent access that could corrupt state.

**Technical Definition**
Mutual exclusion is achieved when every access to a shared resource is guarded by acquisition of the same monitor. If two code regions access the same shared variable but synchronize on *different* locks, mutual exclusion is not enforced between them. The JLS defines a *synchronized* relationship between lock and unlock actions on the same monitor; only when all accesses to a shared resource use the same monitor is strict mutual exclusion guaranteed.

**Beginner-Friendly Explanation**
It's like having a single key for a storage room. If different people have different keys, they can enter simultaneously and collide. Only when everyone uses the *same* key does the room truly allow one person at a time.

### Purposes

- To guarantee that compound operations (read-modify-write) on shared state are atomic.
- To prevent data races and lost updates in concurrent programs.
- To ensure that all threads observe a consistent view of shared memory.
- To protect object invariants from being violated by interleaved thread execution.

### Syntax Rules and Structure

**Complete General Syntax**

```java
// All access points must use the same lock
public class SharedResource {
    private final Object lock = new Object();
    private int value;

    public void update() {
        synchronized (lock) { value++; }
    }

    public int read() {
        synchronized (lock) { return value; }
    }
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Shared lock | The single monitor used by all accessors |
| Critical section | The code inside synchronized blocks |
| Atomicity | The guarantee that the section executes as one unit |

**Syntax Rules**

1. All methods that access a shared variable must synchronize on the same monitor.
2. If a variable is accessed from multiple methods, every access (read and write) must be synchronized.
3. Using different locks for the same shared variable provides no mutual exclusion between those accesses.
4. Synchronizing on `this` means all synchronized instance methods on that object share the lock.

**Constraints and Limitations**

- Synchronizing on different objects for the same shared data fails to provide mutual exclusion.
- Holding a lock while calling a method that acquires another lock can lead to deadlock.
- If a synchronized method calls a non-synchronized method that accesses shared state, the non-synchronized access is unprotected.

### Annotated Code Examples

**Example 1: Incorrect Mutual Exclusion**

```java
public class BrokenCounter {
    private int count = 0;
    private final Object lock1 = new Object();
    private final Object lock2 = new Object();

    public void increment() {
        synchronized (lock1) { count++; }
    }

    public int getCount() {
        synchronized (lock2) { return count; } // DIFFERENT lock!
    }
}
```

**Why This Is Broken**

`increment()` and `getCount()` use different locks. A thread can execute `increment()` while another thread executes `getCount()`. The count may be read mid-increment or stale. Mutual exclusion between these two methods is not enforced.

---

**Example 2: Correct Mutual Exclusion**

```java
public class CorrectCounter {
    private int count = 0;
    private final Object lock = new Object(); // single shared lock

    public void increment() {
        synchronized (lock) { count++; }
    }

    public int getCount() {
        synchronized (lock) { return count; } // same lock
    }
}
```

**Expected Behavior**

All access to `count` is serialized through the same monitor. `increment()` and `getCount()` are mutually exclusive. The count is always consistent.

### Real-World Cases

- **Bank Transfer**: Both the debit and credit operations must be guarded by the same lock to prevent interleaved transfers from corrupting balances.
- **Bounded Buffer**: The `put()` and `take()` methods must synchronize on the same lock to coordinate buffer state and capacity.
- **Session Management**: A session map must be accessed under a single lock to prevent duplicate session creation or loss of session state.

---

## Core Concept 5: Low-Level Signaling — `.wait()`, `.notify()`, and `.notifyAll()`

### Definitions

**Core Definition**
`wait()`, `notify()`, and `notifyAll()` are methods of the `Object` class that allow threads to coordinate inside synchronized code by suspending execution until a condition is met and signaling when the condition changes.

**Technical Definition**
The `wait()` method causes the current thread to wait until another thread invokes `notify()` or `notifyAll()` on the same object, or until a specified timeout elapses. The thread must own the object's monitor before calling `wait()`; `wait()` atomically releases the monitor and suspends the thread. When notified, the thread reacquires the monitor before returning from `wait()`. `notify()` wakes one waiting thread; `notifyAll()` wakes all waiting threads. Per the API documentation, waiting threads should always call `wait()` inside a loop that tests the condition predicate, to guard against spurious wakeups and notifications intended for other conditions.

**Beginner-Friendly Explanation**
`wait()` is like telling a cashier "I'll wait here until my order is ready." You step aside (release the lock) so others can be served. `notify()` is like the cook shouting "Order up!" — it wakes one waiting customer. `notifyAll()` wakes everyone waiting, and each checks whether their own order is ready.

### Purposes

- To allow a thread to suspend execution until a specific condition becomes true, avoiding busy-waiting.
- To coordinate producer–consumer interactions where producers signal consumers when items are available and consumers signal producers when space is available.
- To implement the Guarded Suspension pattern, where a thread waits for a precondition before proceeding.
- To provide a low-level signaling mechanism that works with intrinsic locks, complementing the mutual exclusion provided by `synchronized`.

### Syntax Rules and Structure

**Complete General Syntax**

```java
synchronized (lock) {
    while (!condition) {
        lock.wait(); // releases lock and waits
    }
    // condition is true, proceed
    lock.notifyAll(); // or notify() if conditions are met
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `wait()` | Releases the monitor and suspends the thread until notified |
| `wait(long timeout)` | Waits at most `timeout` milliseconds |
| `notify()` | Wakes one waiting thread (unspecified which) |
| `notifyAll()` | Wakes all waiting threads |
| `while (!condition)` | Loop that guards against spurious wakeups and rechecks the predicate |

**Syntax Rules**

1. `wait()`, `notify()`, and `notifyAll()` must be called from within a synchronized block or method on the same object; otherwise `IllegalMonitorStateException` is thrown.
2. `wait()` atomically releases the monitor and suspends the thread; when it returns, the thread reacquires the monitor.
3. `wait()` must always be called inside a loop (`while (!condition)`) to guard against spurious wakeups and notifications for other conditions.
4. `notifyAll()` is generally preferred over `notify()` unless all waiting threads have identical condition predicates and perform identical actions after waking.
5. `wait(long millis)` returns when notified, when the timeout expires, or when the thread is interrupted (throwing `InterruptedException`).

**Constraints and Limitations**

- `notify()` does not guarantee which thread is awakened; if the awakened thread's condition is not satisfied, it re-enters the wait state, potentially leaving no thread to process the condition.
- Using `notify()` when multiple threads wait on different conditions can cause lost wakeups and system hangs.
- `wait()` releases the monitor but not any other locks the thread may hold, which can contribute to deadlock if not carefully designed.
- Forgetting to use a `while` loop around `wait()` can lead to processing a false condition after a spurious wakeup.

### Annotated Code Examples

**Example 1: Producer–Consumer with wait/notifyAll**

```java
import java.util.ArrayDeque;
import java.util.Deque;

public class ProducerConsumer {
    private final Deque<Integer> queue = new ArrayDeque<>();
    private final int capacity;
    private final Object lock = new Object();

    public ProducerConsumer(int capacity) {
        this.capacity = capacity;
    }

    public void produce(int value) throws InterruptedException {
        synchronized (lock) {
            while (queue.size() == capacity) { // wait for space
                lock.wait();
            }
            queue.addLast(value);
            System.out.println("Produced: " + value);
            lock.notifyAll(); // signal consumers
        }
    }

    public int consume() throws InterruptedException {
        synchronized (lock) {
            while (queue.isEmpty()) { // wait for item
                lock.wait();
            }
            int value = queue.removeFirst();
            System.out.println("Consumed: " + value);
            lock.notifyAll(); // signal producers
            return value;
        }
    }

    public static void main(String[] args) {
        ProducerConsumer pc = new ProducerConsumer(3);

        Thread producer = new Thread(() -> {
            try {
                for (int i = 1; i <= 5; i++) pc.produce(i);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });

        Thread consumer = new Thread(() -> {
            try {
                for (int i = 0; i < 5; i++) pc.consume();
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });

        producer.start();
        consumer.start();
    }
}
```

**Expected Output (order may vary)**

```
Produced: 1
Produced: 2
Produced: 3
Consumed: 1
Consumed: 2
Consumed: 3
Produced: 4
Consumed: 4
Produced: 5
Consumed: 5
```

**Why This Output Occurs**

Producers wait when the queue is full; consumers wait when the queue is empty. When a producer adds an item, it calls `notifyAll()` to wake waiting consumers. When a consumer removes an item, it calls `notifyAll()` to wake waiting producers. The `while` loops ensure that threads recheck the condition after waking, guarding against spurious wakeups and race conditions.

---

**Example 2: Guarded Suspension with Timeout**

```java
public class GuardedSuspension {
    private boolean ready = false;
    private final Object lock = new Object();

    public void waitForReady() throws InterruptedException {
        synchronized (lock) {
            long deadline = System.currentTimeMillis() + 2000; // 2-second timeout
            while (!ready) {
                long remaining = deadline - System.currentTimeMillis();
                if (remaining <= 0) {
                    System.out.println("Timeout waiting for ready");
                    return;
                }
                lock.wait(remaining);
            }
            System.out.println("Ready condition met");
        }
    }

    public void setReady() {
        synchronized (lock) {
            ready = true;
            lock.notifyAll();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        GuardedSuspension gs = new GuardedSuspension();

        Thread waiter = new Thread(() -> {
            try { gs.waitForReady(); } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });

        Thread setter = new Thread(() -> {
            try { Thread.sleep(3000); } catch (InterruptedException e) { }
            gs.setReady();
        });

        waiter.start();
        setter.start();
        waiter.join();
        setter.join();
    }
}
```

**Expected Output**

```
Timeout waiting for ready
```

**Why This Output Occurs**

The waiter thread enters the synchronized block and calls `wait(2000)`. The setter sleeps for 3 seconds before calling `setReady()`. The waiter's timeout expires after 2 seconds, before the setter signals. The `while` loop detects that `ready` is still `false` and that the deadline has passed, prints the timeout message, and returns.

### Real-World Cases

- **`BlockingQueue` Implementations**: `ArrayBlockingQueue` uses `ReentrantLock` and `Condition` (which offer `await()`/`signal()` instead of `wait()`/`notify()`) for the same producer–consumer semantics.
- **Thread Pool Coordination**: Worker threads wait for tasks to become available; the pool manager signals when a task is submitted.
- **GUI Event Dispatch**: The AWT event queue uses synchronization to coordinate event posting and dispatch.
- **Connection Pools**: Threads wait for a connection to become available; a returning connection signals waiters.

---

## References

- Synchronization (The Java Tutorials) - https://docs.oracle.com/javase/tutorial/essential/concurrency/sync.html
- Intrinsic Locks and Synchronization (The Java Tutorials) - https://docs.oracle.com/javase/tutorial/essential/concurrency/locksync.html
- Java Language Specification, Chapter 17: Threads and Locks - https://docs.oracle.com/javase/specs/jls/se23/html/jls-17.html
- Object (Java Platform SE 21 & JDK 21) - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Object.html
- Java Deprecated Thread Primitives (Thread.stop) - https://docs.oracle.com/javase/jp/21/docs/api/java.base/java/lang/doc-files/threadPrimitiveDeprecation.html
- LCK00-J. Use private final lock objects to synchronize classes that may interact with untrusted code (CERT Oracle Secure Coding Standard) - https://wiki.sei.cmu.edu/confluence/display/java/LCK00-J.+Use+private+final+lock+objects+to+synchronize+classes+that+may+interact+with+untrusted+code
- THI02-J. Notify all waiting threads rather than a single thread (CERT Oracle Secure Coding Standard) - https://wiki.sei.cmu.edu/confluence/display/java/THI02-J.+Notify+all+waiting+threads+rather+than+a+single+thread
- THI03-J. Always invoke wait() and await() methods inside a loop (CERT Oracle Secure Coding Standard) - https://wiki.sei.cmu.edu/confluence/display/java/THI03-J.+Always+invoke+wait%28%29+and+await%28%29+methods+inside+a+loop
- Goetz, Brian, et al. *Java Concurrency in Practice*. Addison-Wesley, 2006.