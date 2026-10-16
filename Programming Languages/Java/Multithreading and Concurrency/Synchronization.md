# Java Synchronization: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition:** Synchronization is the mechanism in Java that controls access to shared resources by multiple threads, ensuring that only one thread at a time can execute a critical section of code. It is implemented through the `synchronized` keyword, which provides mutual exclusion and visibility guarantees.

**Technical Definition:** "Synchronization is built around an internal entity known as the intrinsic lock or monitor lock. (The API specification often refers to this entity simply as a 'monitor.') Intrinsic locks play a role in both aspects of synchronization: enforcing exclusive access to an object's state and establishing happens-before relationships that are essential to visibility. Every object has an intrinsic lock associated with it". "Each object in Java is associated with a monitor, which a thread can lock or unlock. Only one thread at a time may hold a lock on a monitor. Any other threads attempting to lock that monitor are blocked until they can obtain a lock on that monitor. A thread t may lock a particular monitor multiple times; each unlock reverses the effect of one lock operation".

**Beginner-Friendly Explanation:** Imagine a single bathroom in a shared apartment. Only one person can use it at a time. When someone enters, they lock the door, and anyone else who wants to use it must wait outside until the door is unlocked. In Java, a `synchronized` block or method is like that bathroom — only one thread can execute it at a time. The "lock" is the intrinsic lock (monitor) of the object. When a thread enters a synchronized region, it automatically acquires the lock; when it exits, it releases the lock, allowing the next waiting thread to enter. This prevents two threads from accidentally corrupting shared data by accessing it simultaneously.

### Key Characteristics

- **Mutual Exclusion:** Only one thread can hold a monitor lock at a time, preventing concurrent access to critical sections.
- **Reentrancy:** A thread that holds a lock can acquire it again, avoiding self-deadlock. "A thread t may lock a particular monitor multiple times; each unlock reverses the effect of one lock operation".
- **Visibility Guarantee:** "When a synchronized method exits, it automatically establishes a happens-before relationship with any subsequent invocation of a synchronized method for the same object. This guarantees that changes to the state of the object are visible to all threads".
- **Two Idioms:** Synchronized methods and synchronized statements (blocks).
- **Intrinsic Locks:** Every object has one; synchronized methods use the object's lock, static synchronized methods use the class object's lock.
- **Non-Interleaving:** "It is not possible for two invocations of synchronized methods on the same object to interleave".
- **Automatic Release:** The lock is released even if the method exits via an uncaught exception.
- **Constructors Cannot Be Synchronized:** Using `synchronized` with a constructor is a syntax error.

### Prerequisites

- Basic understanding of Java classes, methods, and objects
- Familiarity with threads and the `Thread` class
- Knowledge of the `Runnable` interface
- Understanding of the Java Memory Model (visibility, happens-before)

### Related Programming Areas

- **Concurrency:** The broader topic of multithreaded programming.
- **Locks:** `ReentrantLock`, `ReadWriteLock`, and other `java.util.concurrent.locks` implementations.
- **Atomic Variables:** `AtomicInteger`, `AtomicReference` for lock-free thread safety.
- **Concurrent Collections:** `ConcurrentHashMap`, `CopyOnWriteArrayList`.
- **Thread Pools:** `ExecutorService` for managing worker threads.
- **Deadlock Prevention:** Strategies for avoiding deadlock, livelock, and starvation.

### Core Concepts / Features

---

## 1. `synchronized`

### Definitions

**Core Definition:** `synchronized` is a Java keyword that provides mutual exclusion and memory visibility by acquiring an object's monitor lock before executing a block of code or method.

**Technical Definition:** "Asynchronized statement acquires a mutual-exclusion lock on behalf of the executing thread, executes a block, then releases the lock. While the executing thread owns the lock, no other thread may acquire the lock". "Asynchronized method acquires a monitor before it executes. For a class (static) method, the monitor associated with the Class object for the method's class is used. For an instance method, the monitor associated with this (the object for which the method was invoked) is used".

**Beginner-Friendly Explanation:** `synchronized` is the keyword you put on a method or a block of code to tell Java "only one thread at a time can run this." It's like putting a lock on a door. When a thread enters the synchronized area, it locks the door behind it. Other threads must wait until the door is unlocked. This prevents race conditions where two threads try to change the same data at the same time.

### Purposes

- To prevent thread interference when multiple threads access shared mutable state
- To establish happens-before relationships, ensuring memory visibility across threads
- To enforce mutual exclusion on critical sections of code
- To protect invariants from being violated by concurrent access
- To provide a simple, built-in locking mechanism without external libraries

### Syntax Rules and Structure

**Complete General Syntax (Synchronized Method):**

```java
public synchronized returnType methodName(parameters) {
    // critical section
}

public static synchronized returnType methodName(parameters) {
    // critical section (locks on Class object)
}
```

**Complete General Syntax (Synchronized Statement):**

```java
synchronized (lockObject) {
    // critical section
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `synchronized` | Keyword that acquires a monitor lock |
| `lockObject` | The object whose monitor is acquired |
| `{ }` | The critical section protected by the lock |

**Syntax Rules:**

- For instance methods, the lock is the object (`this`).
- For static methods, the lock is the `Class` object.
- The expression in a synchronized statement must be a reference type.
- "A single thread may acquire a lock more than once" (reentrant).
- Constructors cannot be synchronized.

**Constraints and Limitations:**

- "Acquiring the lock associated with an object does not in itself prevent other threads from accessing fields of the object or invoking un-synchronized methods on the object".
- Synchronized blocks can cause liveness problems (deadlock, starvation) if not used carefully.
- Excessive synchronization can reduce concurrency and performance.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Synchronized Method Protecting a Counter**

```java
// File: SynchronizedCounter.java
public class SynchronizedCounter {
    private int c = 0;

    // Synchronized method: only one thread at a time can increment
    public synchronized void increment() {
        c++;
    }

    public synchronized void decrement() {
        c--;
    }

    public synchronized int value() {
        return c;
    }

    public static void main(String[] args) throws InterruptedException {
        SynchronizedCounter counter = new SynchronizedCounter();

        // Two threads incrementing concurrently
        Runnable task = () -> {
            for (int i = 0; i < 10000; i++) {
                counter.increment();
            }
        };

        Thread t1 = new Thread(task, "Thread-1");
        Thread t2 = new Thread(task, "Thread-2");

        t1.start();
        t2.start();
        t1.join();
        t2.join();

        // Without synchronization, result would be less than 20000
        System.out.println("Final count: " + counter.value());
    }
}
```

**Expected Output:**
```
Final count: 20000
```

**Step-by-Step Explanation:**

1. `increment()` is declared `synchronized`, so only one thread can execute it at a time on the same `SynchronizedCounter` object.
2. Two threads each call `increment()` 10,000 times.
3. Because the method is synchronized, the increments are atomic with respect to each other — no lost updates occur.
4. Without synchronization, the final count would be less than 20,000 due to race conditions (lost updates).
5. The synchronized method guarantees that changes to `c` are visible to all threads, and the final value is 20,000.

**Example 2: Synchronized Block with Private Lock Object**

```java
// File: PrivateLockDemo.java
public class PrivateLockDemo {
    private int count = 0;
    private final Object lock = new Object();  // Private lock object

    public void increment() {
        synchronized (lock) {  // Block synchronization
            count++;
        }
    }

    public int getCount() {
        synchronized (lock) {
            return count;
        }
    }

    public static void main(String[] args) throws InterruptedException {
        PrivateLockDemo demo = new PrivateLockDemo();

        Runnable task = () -> {
            for (int i = 0; i < 10000; i++) {
                demo.increment();
            }
        };

        Thread t1 = new Thread(task);
        Thread t2 = new Thread(task);
        t1.start();
        t2.start();
        t1.join();
        t2.join();

        System.out.println("Count: " + demo.getCount());
    }
}
```

**Expected Output:**
```
Count: 20000
```

**Step-by-Step Explanation:**

1. A private final `Object` is used as the lock, rather than `this`.
2. `synchronized (lock)` acquires the monitor of the private object.
3. This prevents external code from interfering with the lock (a security best practice).
4. The count is correctly incremented to 20,000.

### Real-World Cases

- **Counter Services:** Protecting shared counters in web applications.
- **Singleton Pattern:** Synchronizing `getInstance()` to ensure a single instance.
- **Producer-Consumer:** Coordinating access to a shared buffer.
- **Cache Updates:** Protecting shared cache entries from concurrent modification.

---

## 2. Synchronized Methods

### Definitions

**Core Definition:** A synchronized method is a method declared with the `synchronized` keyword, which acquires the object's monitor before execution and releases it upon completion.

**Technical Definition:** "To make a method synchronized, simply add the synchronized keyword to its declaration". "A synchronized method acquires a monitor before it executes. For a class (static) method, the monitor associated with the Class object for the method's class is used. For an instance method, the monitor associated with this (the object for which the method was invoked) is used".

**Beginner-Friendly Explanation:** A synchronized method is like a private room in an office. Only one person (thread) can be in the room at a time. When someone enters, they lock the door. Others must wait outside until the door is unlocked. This ensures that only one thread at a time can execute the method's code, preventing interference.

### Purposes

- To provide a simple, declarative way to synchronize an entire method
- To protect instance state by locking on `this`
- To protect static state by locking on the `Class` object
- To eliminate the need for explicit synchronized blocks

### Syntax Rules and Structure

```java
// Instance method: locks on 'this'
public synchronized void methodName() { }

// Static method: locks on the Class object
public static synchronized void staticMethod() { }
```

**Syntax Rules:**

- The `synchronized` keyword goes in the method declaration.
- Instance synchronized methods lock on `this`.
- Static synchronized methods lock on the `Class` object.
- "Constructors cannot be synchronized — using the synchronized keyword with a constructor is a syntax error".

**Constraints and Limitations:**

- Synchronizing an entire method can reduce concurrency if only part of the method needs protection.
- "Synchronized methods enable a simple strategy for preventing thread interference and memory consistency errors... This strategy is effective, but can present problems with liveness".
- Method-level synchronization can be too coarse-grained.

### Annotated Complete Step-by-Step Code Examples

```java
// File: SynchronizedMethodDemo.java
public class SynchronizedMethodDemo {
    private int balance = 1000;

    // Synchronized instance method
    public synchronized void withdraw(int amount) {
        if (balance >= amount) {
            System.out.println(Thread.currentThread().getName() +
                " withdrawing " + amount);
            balance -= amount;
            System.out.println(Thread.currentThread().getName() +
                " balance after: " + balance);
        } else {
            System.out.println(Thread.currentThread().getName() +
                " insufficient funds");
        }
    }

    public static void main(String[] args) throws InterruptedException {
        SynchronizedMethodDemo account = new SynchronizedMethodDemo();

        Runnable task = () -> account.withdraw(700);

        Thread t1 = new Thread(task, "Alice");
        Thread t2 = new Thread(task, "Bob");

        t1.start();
        t2.start();
        t1.join();
        t2.join();

        System.out.println("Final balance: " + account.balance);
    }
}
```

**Expected Output:**
```
Alice withdrawing 700
Alice balance after: 300
Bob insufficient funds
Final balance: 300
```

**Step-by-Step Explanation:**

1. `withdraw` is synchronized, so only one thread can execute it at a time.
2. Alice acquires the lock, withdraws 700, and updates the balance to 300.
3. Bob then acquires the lock and finds insufficient funds (300 < 700).
4. Without synchronization, both threads might see the balance as 1000 and both withdraw, leaving a negative balance.
5. The synchronized method ensures atomicity and consistency.

### Real-World Cases

- **Bank Account Operations:** Synchronized `deposit()` and `withdraw()` methods.
- **Inventory Management:** Synchronized `decrementStock()` methods.
- **Singleton Pattern:** `public static synchronized Singleton getInstance()`.

---

## 3. Synchronized Blocks

### Definitions

**Core Definition:** A synchronized block (or synchronized statement) is a block of code within a method that is protected by a specified object's monitor lock.

**Technical Definition:** "Another way to create synchronized code is with synchronized statements. Unlike synchronized methods, synchronized statements must specify the object that provides the intrinsic lock". "Asynchronized statement acquires a mutual-exclusion lock on behalf of the executing thread, executes a block, then releases the lock".

**Beginner-Friendly Explanation:** A synchronized block is like a locked drawer inside a larger office. Only the person with the key can access the contents of the drawer, while the rest of the office remains accessible. This allows you to protect only the critical parts of a method, not the entire method.

### Purposes

- To protect only the critical section of a method, improving concurrency
- To use a specific lock object rather than `this` or the `Class` object
- To implement fine-grained locking with multiple lock objects
- To avoid synchronizing method calls that don't need protection

### Syntax Rules and Structure

```java
synchronized (lockObject) {
    // critical section
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `synchronized` | Keyword |
| `lockObject` | The object whose monitor is acquired |
| `{ }` | Critical section |

**Syntax Rules:**

- The expression must be a reference type.
- The lock is held only for the duration of the block.
- "Synchronized statements are also useful for improving concurrency with fine-grained synchronization".
- Multiple lock objects can be used for unrelated operations.

**Constraints and Limitations:**

- Must ensure the lock object is consistent across all threads accessing the shared resource.
- Using a public lock object exposes the lock to external interference.

### Annotated Complete Step-by-Step Code Examples

```java
// File: SynchronizedBlockDemo.java
public class SynchronizedBlockDemo {
    private final Object lock1 = new Object();
    private final Object lock2 = new Object();
    private int c1 = 0;
    private int c2 = 0;

    public void incrementC1() {
        synchronized (lock1) {
            c1++;
        }
    }

    public void incrementC2() {
        synchronized (lock2) {
            c2++;
        }
    }

    public static void main(String[] args) throws InterruptedException {
        SynchronizedBlockDemo demo = new SynchronizedBlockDemo();

        // Two threads incrementing c1
        Runnable task1 = () -> {
            for (int i = 0; i < 10000; i++) demo.incrementC1();
        };

        // Two threads incrementing c2 (can run concurrently with c1 updates)
        Runnable task2 = () -> {
            for (int i = 0; i < 10000; i++) demo.incrementC2();
        };

        Thread t1 = new Thread(task1);
        Thread t2 = new Thread(task2);
        t1.start();
        t2.start();
        t1.join();
        t2.join();

        System.out.println("c1: " + demo.c1 + ", c2: " + demo.c2);
    }
}
```

**Expected Output:**
```
c1: 10000, c2: 10000
```

**Step-by-Step Explanation:**

1. Two separate lock objects (`lock1` and `lock2`) protect `c1` and `c2` respectively.
2. `incrementC1()` and `incrementC2()` can execute concurrently because they use different locks.
3. This fine-grained locking improves concurrency compared to synchronizing the entire object.
4. Each counter correctly reaches 10,000.

### Real-World Cases

- **Fine-Grained Caching:** Protecting different cache segments with different locks.
- **Read-Write Locks:** Using separate locks for reads and writes.
- **Database Connection Pools:** Protecting pool acquisition separately from pool release.

---

## 4. Intrinsic Locks and Monitor Locks

### Definitions

**Core Definition:** An intrinsic lock (also called a monitor lock or monitor) is a built-in locking mechanism associated with every Java object, used by `synchronized` methods and statements.

**Technical Definition:** "Synchronization is built around an internal entity known as the intrinsic lock or monitor lock. (The API specification often refers to this entity simply as a 'monitor.') Every object has an intrinsic lock associated with it. By convention, a thread that needs exclusive and consistent access to an object's fields has to acquire the object's intrinsic lock before accessing them, and then release the intrinsic lock when it's done with them. A thread is said to own the intrinsic lock between the time it has acquired the lock and released the lock. As long as a thread owns an intrinsic lock, no other thread can acquire the same lock. The other thread will block when it attempts to acquire the lock".

**Beginner-Friendly Explanation:** Every Java object has a built-in lock, like every door has a lock. When you use `synchronized`, you're using that object's lock. If you lock the door, no one else can enter until you unlock it. Different objects have different locks, so locking one object doesn't lock another.

### Purposes

- To provide automatic mutual exclusion for synchronized code
- To establish happens-before relationships for memory visibility
- To serve as the foundation for `wait()`, `notify()`, and `notifyAll()`
- To enable reentrant locking (a thread can acquire the same lock multiple times)

### Syntax Rules and Structure

**Lock Acquisition and Release:**

| Operation | Mechanism |
|-----------|-----------|
| Acquire (instance method) | Locks on `this` |
| Acquire (static method) | Locks on `Class` object |
| Acquire (synchronized block) | Locks on the specified object |
| Release | Automatic when method/block exits |

**Syntax Rules:**

- "A thread t may lock a particular monitor multiple times; each unlock reverses the effect of one lock operation" (reentrancy).
- "The Java programming language neither prevents nor requires detection of deadlock conditions".
- Locks are released when the synchronized method or block exits, even via exception.

**Constraints and Limitations:**

- Intrinsic locks cannot be acquired interruptibly or with a timeout.
- There is no way to poll for lock availability.
- Locks are not fair; a waiting thread may starve.

### Annotated Complete Step-by-Step Code Examples

```java
// File: IntrinsicLockDemo.java
public class IntrinsicLockDemo {
    public static void main(String[] args) {
        IntrinsicLockDemo obj = new IntrinsicLockDemo();

        // Reentrant locking: same thread acquires the lock twice
        synchronized (obj) {
            System.out.println("Outer lock acquired");
            synchronized (obj) {
                System.out.println("Inner lock acquired (reentrant)");
            }
            System.out.println("Inner lock released");
        }
        System.out.println("Outer lock released");
    }
}
```

**Expected Output:**
```
Outer lock acquired
Inner lock acquired (reentrant)
Inner lock released
Outer lock released
```

**Step-by-Step Explanation:**

1. The main thread acquires `obj`'s intrinsic lock in the outer `synchronized` block.
2. It then acquires the same lock again in the inner block (reentrancy).
3. Because the thread already owns the lock, it can re-enter without blocking.
4. The inner block releases one level of the lock, then the outer block releases the final level.
5. "This program would deadlock if a single thread were not permitted to lock a monitor more than once".

### Real-World Cases

- **Recursive Algorithms:** Synchronized methods that call themselves.
- **Nested Synchronization:** Methods that call other synchronized methods on the same object.

---

## 5. Reentrant Locks

### Definitions

**Core Definition:** A reentrant lock is a lock that can be acquired multiple times by the same thread without causing a deadlock.

**Technical Definition:** "A thread t may lock a particular monitor multiple times; each unlock reverses the effect of one lock operation". Java's intrinsic locks are reentrant. The `ReentrantLock` class in `java.util.concurrent.locks` provides explicit reentrant locking with additional features.

**Beginner-Friendly Explanation:** Imagine you have a key to your house. You unlock the front door, then unlock the back door with the same key while still inside. Because it's the same key (same thread), you can use it multiple times without getting locked out. If someone else tried to use your key while you're inside, they'd have to wait.

### Purposes

- To allow a thread to re-acquire a lock it already holds
- To prevent self-deadlock in recursive synchronized calls
- To provide flexible locking with `ReentrantLock` (timeout, interruptibility, fairness)

### Syntax Rules and Structure

**Intrinsic (Automatic Reentrancy):**

```java
public synchronized void methodA() {
    methodB();  // methodB is also synchronized — reentrant
}
public synchronized void methodB() { }
```

**Explicit ReentrantLock:**

```java
ReentrantLock lock = new ReentrantLock();
lock.lock();
try {
    // critical section
    lock.lock();  // Reentrant acquisition
    try {
        // nested critical section
    } finally {
        lock.unlock();
    }
} finally {
    lock.unlock();
}
```

**Syntax Rules:**

- Intrinsic locks are automatically reentrant.
- `ReentrantLock` requires explicit `lock()` and `unlock()` calls.
- "The use of synchronized methods or statements provides access to the implicit monitor lock associated with every object, but forces all lock acquisition and release to occur in a block-structured way".
- `ReentrantLock` provides "more extensive locking operations than can be obtained using synchronized methods and statements".

**Constraints and Limitations:**

- `ReentrantLock` must be unlocked in a `finally` block to avoid lock leaks.
- "For simple uncontended mutual exclusion, synchronized often performs equally or better" than `ReentrantLock`.
- "Use ReentrantLock when you specifically need its features, not as a default replacement for synchronized".

### Annotated Complete Step-by-Step Code Examples

```java
// File: ReentrantLockDemo.java
import java.util.concurrent.locks.ReentrantLock;

public class ReentrantLockDemo {
    private final ReentrantLock lock = new ReentrantLock();
    private int count = 0;

    public void outerMethod() {
        lock.lock();
        try {
            System.out.println("Outer method: lock acquired");
            innerMethod();  // Reentrant acquisition
        } finally {
            lock.unlock();
            System.out.println("Outer method: lock released");
        }
    }

    private void innerMethod() {
        lock.lock();  // Reentrant — same thread already holds the lock
        try {
            count++;
            System.out.println("Inner method: count = " + count);
        } finally {
            lock.unlock();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        ReentrantLockDemo demo = new ReentrantLockDemo();

        Thread t1 = new Thread(demo::outerMethod, "Thread-1");
        Thread t2 = new Thread(demo::outerMethod, "Thread-2");

        t1.start();
        t2.start();
        t1.join();
        t2.join();

        System.out.println("Final count: " + demo.count);
    }
}
```

**Expected Output (order may vary):**
```
Outer method: lock acquired
Inner method: count = 1
Outer method: lock released
Outer method: lock acquired
Inner method: count = 2
Outer method: lock released
Final count: 2
```

**Step-by-Step Explanation:**

1. Thread-1 acquires the lock in `outerMethod`.
2. Thread-1 calls `innerMethod`, which acquires the same lock again (reentrant).
3. Thread-1 releases the lock twice (once in inner, once in outer).
4. Thread-2 then acquires the lock and repeats.
5. The count correctly reaches 2.

### Real-World Cases

- **Recursive Data Structures:** Tree traversal with synchronized methods.
- **Complex Operations:** Methods that call other synchronized methods on the same object.
- **Advanced Locking:** Using `ReentrantLock` for timeout, fairness, or interruptible locking.

---

## 6. `wait()`, `notify()`, and `notifyAll()`

### Definitions

**Core Definition:** `wait()`, `notify()`, and `notifyAll()` are methods of the `Object` class that enable inter-thread communication by allowing threads to wait for a condition and be notified when the condition changes.

**Technical Definition:** "The wait() method is employed to temporarily cede possession of a lock so that another thread can proceed". "The notifyAll method wakes up all threads waiting on the object in question. The awakened threads compete for the lock. One thread gets it, and the others go back to waiting". "The Object class also defines the notify method, which arbitrarily wakes up one of the threads waiting on this object".

**Beginner-Friendly Explanation:** Imagine a restaurant kitchen. The chef (producer) prepares a dish and places it on a counter. The waiter (consumer) waits at the counter until the dish is ready. `wait()` is the waiter waiting; `notify()` is the chef saying "the dish is ready"; `notifyAll()` is the chef ringing a bell to wake all waiting waiters. The waiters then compete to grab the dish.

### Purposes

- To coordinate producer-consumer scenarios
- To allow threads to wait for a condition to become true
- To avoid busy-waiting (polling) by suspending threads until notified
- To implement inter-thread communication without shared memory

### Syntax Rules and Structure

```java
// Waiting for a condition
synchronized (lock) {
    while (!condition) {
        lock.wait();  // Releases lock, waits for notification
    }
    // Condition is now true — proceed
}

// Notifying waiting threads
synchronized (lock) {
    // Change condition
    condition = true;
    lock.notifyAll();  // Wake all waiting threads
}
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `wait()` | Waits indefinitely for notification |
| `wait(long timeout)` | Waits up to timeout milliseconds |
| `notify()` | Wakes one arbitrarily chosen waiting thread |
| `notifyAll()` | Wakes all waiting threads |

**Syntax Rules:**

- "wait(), notify(), notifyAll() MUST be called from inside a synchronized block on the same object".
- "ALWAYS use while (not if) before wait()" to guard against spurious wakeups and missed signals.
- "It is safer to always notifyAll(), even if this wakes threads up from the 'wait set' unnecessarily".

**Constraints and Limitations:**

- `wait()` releases the lock; `sleep()` does not.
- `notify()` can wake the wrong thread if conditions are not checked in a loop.
- "Invocation of notify or notifyAll in another thread cannot pin point which waiting thread needs to be resumed".

### Annotated Complete Step-by-Step Code Examples

```java
// File: WaitNotifyDemo.java
public class WaitNotifyDemo {
    private boolean ready = false;

    public synchronized void produce() {
        System.out.println("Producer: producing...");
        ready = true;
        System.out.println("Producer: notifying consumer");
        notifyAll();  // Wake waiting consumer
    }

    public synchronized void consume() {
        while (!ready) {
            try {
                System.out.println("Consumer: waiting for producer...");
                wait();  // Release lock and wait
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                return;
            }
        }
        System.out.println("Consumer: consuming!");
        ready = false;
    }

    public static void main(String[] args) throws InterruptedException {
        WaitNotifyDemo demo = new WaitNotifyDemo();

        Thread consumer = new Thread(demo::consume, "Consumer");
        consumer.start();

        Thread.sleep(1000);  // Let consumer wait

        Thread producer = new Thread(demo::produce, "Producer");
        producer.start();

        consumer.join();
        producer.join();
    }
}
```

**Expected Output:**
```
Consumer: waiting for producer...
Producer: producing...
Producer: notifying consumer
Consumer: consuming!
```

**Step-by-Step Explanation:**

1. The consumer thread calls `consume()`, acquires the lock, and finds `ready == false`.
2. It calls `wait()`, which releases the lock and suspends the consumer.
3. The producer thread calls `produce()`, acquires the lock, sets `ready = true`, and calls `notifyAll()`.
4. The consumer wakes up, re-acquires the lock, re-checks the condition (now `true`), and consumes.
5. The `while` loop guards against spurious wakeups and missed signals.

### Real-World Cases

- **Producer-Consumer:** Coordinating producers and consumers of data.
- **Thread Pool Task Queues:** Workers waiting for tasks.
- **Event Handling:** Threads waiting for events to occur.

---

## 7. Inter-Thread Communication

### Definitions

**Core Definition:** Inter-thread communication is the mechanism by which threads exchange information and coordinate their activities, typically through shared memory with synchronization or through `wait()`/`notify()`.

**Technical Definition:** "The Java programming language provides multiple mechanisms for communicating between threads. The most basic of these methods is synchronization, which is implemented using monitors". "Other mechanisms, such as reads and writes of volatile variables and the use of classes in the java.util.concurrent package, provide alternative ways of synchronization".

**Beginner-Friendly Explanation:** Inter-thread communication is like a conversation between workers. One worker says "I've finished my part, you can start yours now." The other worker listens and responds. In Java, this conversation happens through shared variables (protected by synchronization) and `wait()`/`notify()`.

### Purposes

- To coordinate the order in which threads execute
- To signal that a condition has changed
- To transfer data between threads safely
- To implement producer-consumer, reader-writer, and other coordination patterns

### Syntax Rules and Structure

**Communication Mechanisms:**

| Mechanism | Description |
|-----------|-------------|
| `wait()`/`notify()` | Classic condition-based communication |
| `volatile` | Visibility for simple flags |
| `BlockingQueue` | Thread-safe producer-consumer queues |
| `SynchronousQueue` | Direct handoff between threads |
| `Exchanger` | Two threads exchange objects |
| `CountDownLatch` | Wait for N operations to complete |
| `CyclicBarrier` | Threads wait for each other |

### Annotated Complete Step-by-Step Code Examples

```java
// File: ProducerConsumerDemo.java
public class ProducerConsumerDemo {
    private int[] buffer = new int[5];
    private int count = 0;
    private int in = 0;
    private int out = 0;

    public synchronized void produce(int value) throws InterruptedException {
        while (count == buffer.length) {
            wait();  // Buffer full — wait for consumer
        }
        buffer[in] = value;
        in = (in + 1) % buffer.length;
        count++;
        System.out.println("Produced: " + value + " (count=" + count + ")");
        notifyAll();  // Wake consumer
    }

    public synchronized int consume() throws InterruptedException {
        while (count == 0) {
            wait();  // Buffer empty — wait for producer
        }
        int value = buffer[out];
        out = (out + 1) % buffer.length;
        count--;
        System.out.println("Consumed: " + value + " (count=" + count + ")");
        notifyAll();  // Wake producer
        return value;
    }

    public static void main(String[] args) {
        ProducerConsumerDemo buffer = new ProducerConsumerDemo();

        Thread producer = new Thread(() -> {
            for (int i = 1; i <= 10; i++) {
                try { buffer.produce(i); Thread.sleep(100); }
                catch (InterruptedException e) { Thread.currentThread().interrupt(); }
            }
        }, "Producer");

        Thread consumer = new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try { buffer.consume(); Thread.sleep(150); }
                catch (InterruptedException e) { Thread.currentThread().interrupt(); }
            }
        }, "Consumer");

        producer.start();
        consumer.start();
    }
}
```

**Expected Output (interleaved, order may vary):**
```
Produced: 1 (count=1)
Consumed: 1 (count=0)
Produced: 2 (count=1)
Consumed: 2 (count=0)
...
```

**Step-by-Step Explanation:**

1. The `buffer` array and counters are shared between producer and consumer.
2. `produce()` waits if the buffer is full; `consume()` waits if the buffer is empty.
3. `notifyAll()` wakes all waiting threads after each operation.
4. The `while` loops ensure conditions are re-checked after waking.
5. The producer and consumer alternate safely without race conditions.

### Real-World Cases

- **Message Queues:** Producers put messages; consumers take them.
- **Thread Pools:** Workers wait for tasks; the pool signals when tasks arrive.
- **Pipeline Processing:** Data flows through multiple stages connected by queues.

---

## 8. Deadlock

### Definitions

**Core Definition:** Deadlock is a situation where two or more threads are blocked forever, each waiting for a resource (lock) held by another.

**Technical Definition:** "A deadlock occurs when two or more threads are blocked forever, each waiting on the other to release a resource". "The Java programming language neither prevents nor requires detection of deadlock conditions. Programs where threads hold (directly or indirectly) locks on multiple objects should use conventional techniques for deadlock avoidance, creating higher-level locking primitives that don't deadlock, if necessary".

**Beginner-Friendly Explanation:** Deadlock is like two people in a narrow hallway, each waiting for the other to step aside. Neither moves, so both are stuck forever. In Java, Thread A holds Lock 1 and waits for Lock 2, while Thread B holds Lock 2 and waits for Lock 1. Neither can proceed.

### Purposes

- (Not a purpose — a problem to avoid)
- To understand the conditions that cause deadlock so they can be prevented

### Causes and Conditions

**Four Necessary Conditions (Coffman Conditions):**

| Condition | Description |
|-----------|-------------|
| Mutual Exclusion | At least one resource must be held exclusively |
| Hold and Wait | A thread holds a resource and waits for another |
| No Preemption | Resources cannot be forcibly taken away |
| Circular Wait | A cycle of threads waiting for each other's resources |

### Annotated Complete Step-by-Step Code Examples

```java
// File: DeadlockDemo.java
public class DeadlockDemo {
    private final Object lock1 = new Object();
    private final Object lock2 = new Object();

    public void methodA() {
        synchronized (lock1) {
            System.out.println("Thread A: holding lock1");
            try { Thread.sleep(100); } catch (InterruptedException e) { }
            synchronized (lock2) {
                System.out.println("Thread A: holding lock1 and lock2");
            }
        }
    }

    public void methodB() {
        synchronized (lock2) {
            System.out.println("Thread B: holding lock2");
            try { Thread.sleep(100); } catch (InterruptedException e) { }
            synchronized (lock1) {
                System.out.println("Thread B: holding lock2 and lock1");
            }
        }
    }

    public static void main(String[] args) {
        DeadlockDemo demo = new DeadlockDemo();

        Thread t1 = new Thread(demo::methodA, "Thread-A");
        Thread t2 = new Thread(demo::methodB, "Thread-B");

        t1.start();
        t2.start();
    }
}
```

**Expected Output:**
```
Thread A: holding lock1
Thread B: holding lock2
(program hangs — deadlock)
```

**Step-by-Step Explanation:**

1. Thread A acquires `lock1` and then tries to acquire `lock2`.
2. Thread B acquires `lock2` and then tries to acquire `lock1`.
3. Thread A waits for `lock2` (held by B); Thread B waits for `lock1` (held by A).
4. Neither can proceed — deadlock.
5. The program hangs indefinitely.

### Real-World Cases

- **Database Transactions:** Two transactions locking rows in opposite order.
- **Resource Pools:** Threads holding connections and waiting for others.
- **Nested Synchronization:** Multiple synchronized methods calling each other in different orders.

---

## 9. Livelock

### Definitions

**Core Definition:** Livelock is a situation where two or more threads are actively executing but not making progress because they are too busy responding to each other.

**Technical Definition:** "Similar to a deadlock, except that the states of the threads involved in the livelock constantly change with regard to one another, none progressing".

**Beginner-Friendly Explanation:** Livelock is like two people trying to pass each other in a corridor. One steps left, the other steps right, then they both step the other way. They keep moving but never get past each other. The threads are not blocked (unlike deadlock) but still make no progress.

### Purposes

- (Not a purpose — a problem to avoid)

### Annotated Complete Step-by-Step Code Examples

```java
// File: LivelockDemo.java
public class LivelockDemo {
    static class Worker {
        private String name;
        private boolean active = true;

        public Worker(String name) { this.name = name; }

        public void workWith(Worker other) {
            while (active && other.active) {
                System.out.println(name + " is trying to work");
                // Give way to the other worker
                try { Thread.sleep(100); } catch (InterruptedException e) { }
                // Both keep yielding — no progress
            }
        }

        public void stop() { active = false; }
    }

    public static void main(String[] args) {
        Worker alice = new Worker("Alice");
        Worker bob = new Worker("Bob");

        Thread t1 = new Thread(() -> alice.workWith(bob), "Alice");
        Thread t2 = new Thread(() -> bob.workWith(alice), "Bob");

        t1.start();
        t2.start();
    }
}
```

**Expected Output:**
```
Alice is trying to work
Bob is trying to work
Alice is trying to work
Bob is trying to work
...(continues indefinitely)
```

**Step-by-Step Explanation:**

1. Alice and Bob both check if the other is active.
2. Both are active, so both keep yielding to each other.
3. Neither makes progress — livelock.
4. The threads are not blocked; they are actively executing but not progressing.

### Real-World Cases

- **Optimistic Concurrency:** Threads retrying transactions that keep conflicting.
- **Message Passing:** Processes that keep sending messages to each other without progress.
- **Network Protocols:** Two nodes retrying transmissions simultaneously.

---

## 10. Starvation

### Definitions

**Core Definition:** Starvation is a situation where a thread is unable to gain regular access to shared resources and is unable to make progress.

**Technical Definition:** "Lock starvation occurs when a thread, having lesser priority than other ones, is constantly waiting for a lock". "Starvation is a situation where a thread is repeatedly denied the opportunity to make progress".

**Beginner-Friendly Explanation:** Starvation is like a person at a buffet who never gets to eat because other, more aggressive people keep grabbing all the food. The thread is not deadlocked (it's not blocked forever) but it rarely or never gets a turn.

### Purposes

- (Not a purpose — a problem to avoid)

### Causes of Starvation

| Cause | Description |
|-------|-------------|
| Priority differences | Low-priority threads rarely get scheduled |
| Unfair locks | A thread repeatedly loses lock acquisition races |
| Synchronized methods | A long-running synchronized method blocks others indefinitely |
| Thread groups | High-priority group monopolises resources |

### Annotated Complete Step-by-Step Code Examples

```java
// File: StarvationDemo.java
public class StarvationDemo {
    public static void main(String[] args) {
        Object lock = new Object();

        // High-priority thread that holds the lock for long periods
        Thread greedy = new Thread(() -> {
            while (true) {
                synchronized (lock) {
                    System.out.println("Greedy thread working...");
                    try { Thread.sleep(1000); } catch (InterruptedException e) { }
                }
            }
        }, "Greedy");

        // Low-priority thread that rarely gets the lock
        Thread starved = new Thread(() -> {
            while (true) {
                synchronized (lock) {
                    System.out.println("Starved thread finally got the lock!");
                }
            }
        }, "Starved");

        greedy.setPriority(Thread.MAX_PRIORITY);
        starved.setPriority(Thread.MIN_PRIORITY);

        greedy.start();
        starved.start();
    }
}
```

**Expected Output (varies):**
```
Greedy thread working...
Greedy thread working...
Greedy thread working...
...
(starved thread may never print)
```

**Step-by-Step Explanation:**

1. The greedy thread has maximum priority and holds the lock for 1 second at a time.
2. The starved thread has minimum priority and rarely gets a chance to acquire the lock.
3. The starved thread may never execute — starvation.

### Real-World Cases

- **Unfair Locks:** Threads with lower priority waiting indefinitely.
- **Thread Pools:** Certain tasks always losing the race for worker threads.
- **Resource Allocation:** Low-priority threads never getting CPU time.

---

## 11. Race Conditions

### Definitions

**Core Definition:** A race condition is a situation where the correctness of a program depends on the relative timing or interleaving of threads, leading to unpredictable results.

**Technical Definition:** "A race condition is a situation where two threads compete for the same resource and they try to change it at the same time, doing it in a way that causes unexpected results".

**Beginner-Friendly Explanation:** A race condition is like two people trying to write their name on the same line of a paper at the same time. The result is a mess — parts of both names are written, and neither is complete. In programming, race conditions cause data corruption because threads interfere with each other.

### Purposes

- (Not a purpose — a problem to avoid)

### Annotated Complete Step-by-Step Code Examples

```java
// File: RaceConditionDemo.java
public class RaceConditionDemo {
    private static int counter = 0;

    public static void increment() {
        counter++;  // Not atomic: read, increment, write
    }

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 10000; i++) increment();
        });
        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 10000; i++) increment();
        });

        t1.start();
        t2.start();
        t1.join();
        t2.join();

        System.out.println("Counter: " + counter);
        // Expected: 20000, Actual: less (race condition)
    }
}
```

**Expected Output (varies):**
```
Counter: 17432
```

**Step-by-Step Explanation:**

1. `counter++` is not atomic — it involves read, modify, and write.
2. Two threads execute `increment()` concurrently.
3. Both threads read the same value of `counter`, increment it, and write it back.
4. One write overwrites the other — lost updates.
5. The final count is less than 20,000, demonstrating the race condition.

### Real-World Cases

- **Banking:** Concurrent deposits and withdrawals corrupting balances.
- **Inventory:** Overselling products due to concurrent stock updates.
- **Caching:** Concurrent cache updates causing stale or corrupted data.

---

## 12. Thread Safety

### Definitions

**Core Definition:** Thread safety is the property of a class or method that guarantees correct behaviour when accessed by multiple threads simultaneously.

**Technical Definition:** "Thread-safe code is code that manipulates shared data structures in a manner that enables all threads to make progress without interference" (paraphrased). "If an object is visible to more than one thread, all reads or writes to that object's variables are done through synchronized methods. (An important exception: final fields, which cannot be modified after the object is constructed, can be safely read through non-synchronized methods, once the object is constructed)".

**Beginner-Friendly Explanation:** Thread safety means your code works correctly no matter how many threads are using it at the same time. A thread-safe class is like a well-designed kitchen — no matter how many cooks are working, the food comes out right. A non-thread-safe class is like a kitchen where two cooks might grab the same knife at the same time and cut themselves.

### Purposes

- To ensure correct behaviour under concurrent access
- To prevent race conditions and data corruption
- To provide visibility guarantees for shared data
- To enable safe reuse of objects across threads

### Strategies for Thread Safety

| Strategy | Description |
|----------|-------------|
| Immutability | Objects cannot be modified after creation |
| Synchronization | Using `synchronized` to protect critical sections |
| Thread-local storage | Each thread has its own copy of data |
| Atomic variables | `AtomicInteger`, `AtomicReference` |
| Concurrent collections | `ConcurrentHashMap`, `CopyOnWriteArrayList` |
| Confinement | Data is confined to a single thread |

### Annotated Complete Step-by-Step Code Examples

```java
// File: ThreadSafeDemo.java
public class ThreadSafeDemo {
    // Thread-safe: synchronized methods
    private int count = 0;

    public synchronized void increment() {
        count++;
    }

    public synchronized int getCount() {
        return count;
    }

    public static void main(String[] args) throws InterruptedException {
        ThreadSafeDemo demo = new ThreadSafeDemo();

        Runnable task = () -> {
            for (int i = 0; i < 10000; i++) {
                demo.increment();
            }
        };

        Thread t1 = new Thread(task);
        Thread t2 = new Thread(task);
        t1.start();
        t2.start();
        t1.join();
        t2.join();

        System.out.println("Count: " + demo.getCount());  // Always 20000
    }
}
```

**Expected Output:**
```
Count: 20000
```

**Step-by-Step Explanation:**

1. `increment()` and `getCount()` are synchronized, ensuring mutual exclusion.
2. Both threads safely increment the shared counter.
3. The synchronized methods provide both atomicity and visibility.
4. The final count is always 20,000, regardless of thread scheduling.

### Real-World Cases

- **Web Controllers:** Thread-safe handling of concurrent requests.
- **Caching Libraries:** Thread-safe cache implementations.
- **Database Connection Pools:** Thread-safe acquisition and release.

---

## 13. Synchronization Best Practices

### Definitions

**Core Definition:** Synchronization best practices are guidelines for using synchronization correctly, efficiently, and safely in concurrent Java programs.

**Technical Definition:** Best practices derive from authoritative sources including the SEI CERT Oracle Coding Standard for Java. Key principles include using private final lock objects, minimising synchronized scope, avoiding nested locks, and preferring higher-level concurrency utilities.

**Beginner-Friendly Explanation:** Synchronization best practices help you avoid common pitfalls like deadlocks, performance bottlenecks, and security vulnerabilities. They're like a safety manual for locking.

### Purposes

- To prevent deadlocks, livelocks, and starvation
- To improve concurrency and performance
- To protect against security vulnerabilities (e.g., denial of service)
- To write maintainable and correct synchronized code

### Key Best Practices Table

| Practice | Description | Source |
|----------|-------------|--------|
| Use private final lock objects | "Use the private lock object idiom instead of intrinsic synchronization". | SEI CERT |
| Minimise synchronized scope | "Do as little as possible in synchronized region". | CMU |
| Avoid nested locks | Prevent deadlock by avoiding acquiring multiple locks. | Best practice |
| Use consistent lock ordering | "Require and release locks in the same order to avoid deadlock". | JPCERT |
| Do not synchronize on `this` or `Class` | "Do not use an instance lock to protect shared static data". | SEI CERT |
| Prefer block synchronization over method synchronization | "Synchronized block statements allow the lock duration to be minimized". | OpenJDK |
| Use `notifyAll()` instead of `notify()` | "It is safer to always notifyAll()". | Cornell |
| Always use `while` (not `if`) with `wait()` | "ALWAYS use while (not if) before wait()". | Cornell |
| Restore interrupt status | Call `Thread.currentThread().interrupt()` when catching `InterruptedException`. | Best practice |
| Consider `ReentrantLock` for advanced features | "Use ReentrantLock when you specifically need its features". | Stack Overflow |
| Document locking policy | Explain which locks protect which state. | Effective Java |
| Avoid excessive synchronization | "Excessive method synchronization can induce a denial of service (DoS) vulnerability". | SEI CERT |

### Annotated Complete Step-by-Step Code Examples

```java
// File: SyncBestPracticesDemo.java
public class SyncBestPracticesDemo {
    // BEST PRACTICE: Private final lock object
    private final Object lock = new Object();
    private int count = 0;

    public void increment() {
        // BEST PRACTICE: Minimal synchronized scope
        synchronized (lock) {
            count++;
        }
        // Non-critical work outside the lock
        System.out.println("Incremented to " + count);
    }

    public int getCount() {
        synchronized (lock) {
            return count;
        }
    }

    // BEST PRACTICE: Consistent lock ordering
    private final Object lockA = new Object();
    private final Object lockB = new Object();

    public void method1() {
        synchronized (lockA) {
            synchronized (lockB) {
                // work
            }
        }
    }

    public void method2() {
        synchronized (lockA) {  // Same order: lockA then lockB
            synchronized (lockB) {
                // work
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        SyncBestPracticesDemo demo = new SyncBestPracticesDemo();

        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) demo.increment();
        });
        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) demo.increment();
        });

        t1.start();
        t2.start();
        t1.join();
        t2.join();

        System.out.println("Final: " + demo.getCount());
    }
}
```

**Expected Output:**
```
Final: 2000
```

**Step-by-Step Explanation:**

1. A private final lock object is used instead of `this`.
2. The synchronized block protects only the `count++` operation, minimizing lock duration.
3. Both `method1()` and `method2()` acquire `lockA` before `lockB`, preventing deadlock.
4. The counter reaches 2000 correctly.

### Real-World Cases

- **Enterprise Applications:** Consistent locking policies across services.
- **High-Performance Systems:** Minimizing lock contention for scalability.
- **Security-Sensitive Code:** Protecting against denial-of-service attacks via lock manipulation.

---

## References Links

- Synchronized Methods — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/essential/concurrency/syncmeth.html
- Intrinsic Locks and Synchronization — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/essential/concurrency/locksync.html
- Chapter 14. Blocks and Statements — Java Language Specification (SE 8) - https://docs.oracle.com/javase/specs/jls/se8/html/jls-14.html
- Chapter 8. Classes — Java Language Specification (SE 15) - https://docs.oracle.com/javase/specs/jls/se15/html/jls-8.html
- Chapter 17. Threads and Locks — Java Language Specification (SE 6) - https://docs.oracle.com/javase/specs/jls/se6/html/memory.html
- CON10-J. Use the private lock object idiom instead of intrinsic synchronization — SEI CERT - https://wiki.sei.cmu.edu/confluence/pages/viewpage.action?pageId=88500616
- LCK00-J. Use private final lock objects to synchronize classes that may interact with untrusted code — SEI CERT - https://wiki.sei.cmu.edu/confluence/pages/viewpage.action?pageId=88888126
- Using the notifyAll and wait Methods — IIT Bombay (Sun Java Tutorial) - https://www.cse.iitb.ac.in/~cs296/Java/SunJavaTutorial/essential/threads/waitAndNotify.html
- Java Concurrency Problems — OCPJP8 Notes - https://github.com/eh3rrera/ocpj8-notes
- Java Thread Synchronization — CMU - https://www.cs.cmu.edu
- ReentrantLock vs synchronized — Stack Overflow - https://stackoverflow.com
- Java Concurrency Best Practices — SEI CERT - https://wiki.sei.cmu.edu/confluence/display/java/Concurrency