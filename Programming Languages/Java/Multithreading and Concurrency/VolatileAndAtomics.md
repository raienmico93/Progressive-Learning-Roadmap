# Java Volatile and Atomics: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** `volatile` is a Java keyword that guarantees memory visibility and ordering for a single variable, while atomic classes in `java.util.concurrent.atomic` provide lock-free thread-safe operations on single variables that also guarantee atomicity for read-modify-write sequences.

**Technical Definition:** A field may be declared `volatile`, in which case the Java Memory Model (§17) ensures that all threads see a consistent value for the variable. A write to a volatile variable establishes a happens-before relationship with all subsequent reads of that volatile variable. The `java.util.concurrent.atomic` package is "a small toolkit of classes that support lock-free thread-safe programming on single variables. Instances of Atomic classes maintain values that are accessed and updated using methods otherwise available for fields using associated atomic VarHandle operations". In essence, the classes in this package extend the notion of volatile values, fields, and array elements to those that also provide an atomic conditional update operation of the form `compareAndSet`.

**Beginner-Friendly Explanation:** `volatile` is like a shared whiteboard in an office — when someone writes on it, everyone immediately sees the new value. But `volatile` only guarantees that you see the latest value; it doesn't stop two people from trying to write on the whiteboard at the same time and making a mess. Atomic classes solve that problem. They're like a whiteboard with a lock — only one person can update it at a time, and the update is guaranteed to be atomic (all or nothing). Think of `AtomicInteger` as a counter that multiple people can safely increment without losing count.

### Key Characteristics

- **Volatile Visibility:** `volatile` guarantees that writes are immediately visible to other threads and prevents instruction reordering.
- **Volatile Limitation:** `volatile` does not guarantee atomicity for composite operations (e.g., `i++`).
- **Atomic Classes:** Provide atomicity for single-variable read-modify-write operations without locks.
- **CAS Foundation:** Atomic classes are built on the compare-and-swap (CAS) instruction.
- **Lock-Free:** Atomic operations use hardware-level CAS rather than locks, avoiding deadlocks and reducing contention.
- **Adders and Accumulators (Java 8+):** `LongAdder` and `LongAccumulator` are optimised for high-contention scenarios where writes vastly outnumber reads.
- **Array Atomicity:** `AtomicIntegerArray`, `AtomicLongArray`, and `AtomicReferenceArray` provide atomic access to array elements.
- **Not General Replacements:** Atomic classes do not define `equals`, `hashCode`, or `compareTo`, and are poor choices for hash table keys.

### Prerequisites

- Basic understanding of Java classes, methods, and fields
- Familiarity with threads and the `Thread` class
- Knowledge of synchronization and the Java Memory Model
- Understanding of the happens-before relationship

### Related Programming Areas

- **Concurrency:** The broader topic of multithreaded programming.
- **Synchronization:** `synchronized` blocks and locks provide mutual exclusion.
- **Locks:** `ReentrantLock` and other explicit lock implementations.
- **Concurrent Collections:** `ConcurrentHashMap` and other thread-safe collections.
- **Thread Pools:** `ExecutorService` for managing worker threads.

### Core Concepts / Features

---

## 1. `volatile`

### Definitions

**Core Definition:** `volatile` is a field modifier that guarantees memory visibility and prevents certain compiler and hardware reorderings, but does not guarantee atomicity for compound operations.

**Technical Definition:** "A field may be declared `volatile`, in which case the Java Memory Model ensures that all threads see a consistent value for the variable". "A happens-before relationship exists between a thread that writes to a volatile variable and a thread that subsequently reads it". "Declaring a variable volatile guarantees the happens-before relationship so that writes are always visible to subsequent reads from any thread". "The volatile keyword in Java is used as an indicator to Java compiler and Thread that do not cache value of this variable and always read it from main memory".

**Beginner-Friendly Explanation:** `volatile` is like a noticeboard in a shared office. When one person writes a new notice, everyone sees it immediately — they don't have to check back later. But if two people try to write on the noticeboard at the same time, their notes get mixed up. `volatile` ensures visibility, not atomicity.

### Purposes

- To ensure that writes to a variable are immediately visible to all threads
- To prevent the compiler and CPU from reordering instructions in ways that would break visibility
- To provide a lightweight alternative to synchronization for simple flags and state variables
- To establish happens-before relationships between volatile writes and subsequent volatile reads

### Syntax Rules and Structure

**Complete General Syntax:**

```java
private volatile boolean flag = false;
private volatile int counter = 0;
private volatile Object reference = null;
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `volatile` | Field modifier that guarantees visibility |
| `type` | Any primitive type or reference type |
| `fieldName` | The variable name |

**Syntax Rules:**

- `volatile` can be applied only to fields (instance, static, or array elements).
- `volatile` cannot be applied to local variables.
- `volatile` cannot be applied to `final` fields (redundant) or method parameters.
- "Reads and writes are atomic for all variables declared volatile (including long and double variables)".

**Constraints and Limitations:**

- "Declaring a shared mutable variable volatile ensures the visibility of the latest updates on it across other threads but does not guarantee the atomicity of composite operations".
- "Volatile variables are unsuitable when more than one load/store operation needs to be atomic".
- "Declaring an object volatile to ensure visibility of its state does not work without the use of synchronization, unless the object is immutable".
- `volatile` does not prevent race conditions in compound operations like `i++` or `check-then-act`.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Volatile Flag for Thread Coordination**

```java
// File: VolatileFlagDemo.java
public class VolatileFlagDemo {
    // Volatile flag ensures visibility across threads
    private static volatile boolean running = true;

    public static void main(String[] args) throws InterruptedException {
        Thread worker = new Thread(() -> {
            while (running) {
                // Busy wait — volatile ensures the flag is re-read each iteration
            }
            System.out.println("Worker stopped");
        }, "Worker");

        worker.start();
        Thread.sleep(1000);
        System.out.println("Main: setting running to false");
        running = false;  // Volatile write — immediately visible
        worker.join();
        System.out.println("Done");
    }
}
```

**Expected Output:**
```
Main: setting running to false
Worker stopped
Done
```

**Step-by-Step Explanation:**

1. `running` is declared `volatile`, so the worker thread always reads the latest value from main memory.
2. The worker loops while `running` is `true`.
3. After 1 second, the main thread sets `running = false`.
4. Because `running` is volatile, the worker immediately sees the change and exits the loop.
5. Without `volatile`, the worker might cache the old value and loop forever.

**Example 2: Volatile Does Not Guarantee Atomicity**

```java
// File: VolatileCounterDemo.java
public class VolatileCounterDemo {
    private static volatile int counter = 0;

    public static void main(String[] args) throws InterruptedException {
        Runnable task = () -> {
            for (int i = 0; i < 10000; i++) {
                counter++;  // NOT atomic — read, increment, write
            }
        };

        Thread t1 = new Thread(task);
        Thread t2 = new Thread(task);
        t1.start();
        t2.start();
        t1.join();
        t2.join();

        System.out.println("Counter: " + counter);
        // Expected: 20000, Actual: less due to race condition
    }
}
```

**Expected Output (varies):**
```
Counter: 17432
```

**Step-by-Step Explanation:**

1. `counter` is declared `volatile`, so all threads see the latest value.
2. However, `counter++` is a compound operation: read, increment, write.
3. Two threads can read the same value, increment it, and write it back — one write overwrites the other.
4. The final count is less than 20,000, demonstrating that `volatile` does not provide atomicity.

### Real-World Cases

- **Stop Flags:** Using a volatile boolean to signal threads to stop.
- **State Publishing:** Publishing immutable objects safely across threads.
- **Double-Checked Locking:** Using volatile in the singleton pattern.

---

## 2. Memory Visibility and Happens-Before Relationship

### Definitions

**Core Definition:** Memory visibility is the guarantee that changes made by one thread to shared data are visible to other threads. The happens-before relationship defines when one action is guaranteed to be visible to another.

**Technical Definition:** "A happens-before relationship exists between a thread that writes to a volatile variable and a thread that subsequently reads it". "If two accesses follow the happens-before relationship, data races cannot occur". The Java Memory Model defines several happens-before rules, including: program order (within a thread), monitor lock (unlock happens-before subsequent lock), volatile write (happens-before subsequent read of the same variable), and thread start/join rules.

**Beginner-Friendly Explanation:** Imagine a shared notebook. Without any coordination, one person might write in the notebook while another reads an old copy. The happens-before relationship is like a rule that says "if you write something and tell me you're done, I'll see it when I read next." Volatile establishes this rule for a single variable.

### Purposes

- To ensure that changes made by one thread are visible to others
- To prevent memory consistency errors
- To define ordering guarantees between operations in different threads
- To provide the foundation for safe publication

### Syntax Rules and Structure

**Happens-Before Rules:**

| Rule | Description |
|------|-------------|
| Program Order | Each action in a thread happens-before every subsequent action in that thread. |
| Monitor Lock | An unlock on a monitor happens-before every subsequent lock on that monitor. |
| Volatile Variable | A write to a volatile field happens-before every subsequent read of that field. |
| Thread Start | `Thread.start()` happens-before any action in the started thread. |
| Thread Join | All actions in a thread happen-before any other thread successfully returns from `join()`. |

**Syntax Rules:**

- A volatile write creates a happens-before relationship with all subsequent volatile reads of the same variable.
- "Declaring a variable volatile guarantees the happens-before relationship so that writes are always visible to subsequent reads from any thread".

### Annotated Code Example

```java
// File: HappensBeforeDemo.java
public class HappensBeforeDemo {
    private static volatile boolean ready = false;
    private static int data = 0;

    public static void main(String[] args) throws InterruptedException {
        Thread producer = new Thread(() -> {
            data = 42;        // 1. Write data
            ready = true;     // 2. Volatile write — establishes happens-before
        });

        Thread consumer = new Thread(() -> {
            while (!ready) { }  // 3. Volatile read — sees ready = true
            System.out.println("Data: " + data);  // 4. Guaranteed to see 42
        });

        consumer.start();
        producer.start();
        producer.join();
        consumer.join();
    }
}
```

**Expected Output:**
```
Data: 42
```

**Step-by-Step Explanation:**

1. The producer writes `data = 42` (non-volatile).
2. The producer writes `ready = true` (volatile) — this establishes a happens-before relationship.
3. The consumer reads `ready` (volatile) and sees `true`.
4. Because of the happens-before relationship, the consumer is guaranteed to see `data = 42`.

### Real-World Cases

- **Safe Publication:** Publishing an object reference and its state safely across threads.
- **Configuration Updates:** Making configuration changes visible to worker threads.
- **Producer-Consumer:** Using volatile flags to coordinate simple handoff.

---

## 3. Atomic Classes

### Definitions

**Core Definition:** Atomic classes in `java.util.concurrent.atomic` provide lock-free thread-safe operations on single variables, including atomic read-modify-write sequences.

**Technical Definition:** "A small toolkit of classes that support lock-free thread-safe programming on single variables. Instances of Atomic classes maintain values that are accessed and updated using methods otherwise available for fields using associated atomic VarHandle operations". "The classes in this package extend the notion of volatile values, fields, and array elements to those that also provide an atomic conditional update operation of the form `compareAndSet`".

**Beginner-Friendly Explanation:** Atomic classes are like having a magic counter that multiple people can safely increment at the same time without losing count. The counter uses hardware-level magic (CAS) to ensure that every increment is atomic — all or nothing. No locks, no waiting, no lost updates.

### Purposes

- To provide atomic operations on single variables without using locks
- To avoid the overhead and deadlock risks of synchronization
- To enable lock-free concurrent algorithms and data structures
- To serve as building blocks for higher-level concurrency utilities

### Core Atomic Classes

| Class | Description |
|-------|-------------|
| `AtomicBoolean` | A `boolean` value that may be updated atomically |
| `AtomicInteger` | An `int` value that may be updated atomically |
| `AtomicLong` | A `long` value that may be updated atomically |
| `AtomicReference<V>` | An object reference that may be updated atomically |
| `AtomicIntegerArray` | An `int` array whose elements may be updated atomically |
| `AtomicLongArray` | A `long` array whose elements may be updated atomically |
| `AtomicReferenceArray<V>` | An object reference array whose elements may be updated atomically |
| `LongAdder` | A high-performance adder for `long` values |
| `LongAccumulator` | A high-performance accumulator for `long` values |
| `DoubleAdder` | A high-performance adder for `double` values |
| `DoubleAccumulator` | A high-performance accumulator for `double` values |

### Syntax Rules and Structure

**Complete General Syntax:**

```java
// AtomicInteger
AtomicInteger counter = new AtomicInteger(0);
int value = counter.get();
counter.set(10);
counter.incrementAndGet();
counter.getAndIncrement();
counter.addAndGet(5);
counter.compareAndSet(15, 20);

// AtomicReference
AtomicReference<String> ref = new AtomicReference<>("initial");
ref.set("updated");
ref.compareAndSet("updated", "final");

// AtomicIntegerArray
AtomicIntegerArray array = new AtomicIntegerArray(10);
array.set(0, 42);
int element = array.get(0);
array.compareAndSet(0, 42, 100);
```

**Key Methods:**

| Method | Description |
|--------|-------------|
| `get()` | Returns the current value |
| `set(value)` | Sets the value |
| `getAndSet(value)` | Atomically sets and returns the previous value |
| `compareAndSet(expected, update)` | Atomically sets if current value equals expected |
| `incrementAndGet()` | Atomically increments and returns the new value |
| `getAndIncrement()` | Atomically increments and returns the previous value |
| `addAndGet(delta)` | Atomically adds and returns the new value |
| `getAndAdd(delta)` | Atomically adds and returns the previous value |
| `updateAndGet(function)` | Atomically updates with a function and returns the new value |

### Annotated Complete Step-by-Step Code Examples

**Example 1: AtomicInteger Counter**

```java
// File: AtomicCounterDemo.java
import java.util.concurrent.atomic.AtomicInteger;

public class AtomicCounterDemo {
    private static final AtomicInteger counter = new AtomicInteger(0);

    public static void main(String[] args) throws InterruptedException {
        Runnable task = () -> {
            for (int i = 0; i < 10000; i++) {
                counter.incrementAndGet();  // Atomic operation
            }
        };

        Thread t1 = new Thread(task);
        Thread t2 = new Thread(task);
        t1.start();
        t2.start();
        t1.join();
        t2.join();

        System.out.println("Counter: " + counter.get());  // Always 20000
    }
}
```

**Expected Output:**
```
Counter: 20000
```

**Step-by-Step Explanation:**

1. `AtomicInteger` is created with an initial value of `0`.
2. Two threads each call `incrementAndGet()` 10,000 times.
3. `incrementAndGet()` is atomic — it uses CAS to ensure no lost updates.
4. The final count is always 20,000, regardless of thread scheduling.

**Example 2: AtomicReference for Lock-Free Updates**

```java
// File: AtomicReferenceDemo.java
import java.util.concurrent.atomic.AtomicReference;

public class AtomicReferenceDemo {
    static class User {
        final String name;
        final int age;
        User(String name, int age) { this.name = name; this.age = age; }
        public String toString() { return name + " (" + age + ")"; }
    }

    public static void main(String[] args) {
        AtomicReference<User> userRef = new AtomicReference<>(new User("Alice", 30));

        // Atomically update the reference
        User oldUser = userRef.get();
        User newUser = new User("Alice", 31);
        boolean updated = userRef.compareAndSet(oldUser, newUser);

        System.out.println("Updated: " + updated);  // true
        System.out.println("User: " + userRef.get());  // Alice (31)
    }
}
```

**Expected Output:**
```
Updated: true
User: Alice (31)
```

**Step-by-Step Explanation:**

1. `AtomicReference` holds a reference to a `User` object.
2. `compareAndSet` atomically checks if the current reference equals `oldUser`, and if so, sets it to `newUser`.
3. The update succeeds because the reference still points to `oldUser`.
4. The new user object is visible to all threads.

### Real-World Cases

- **Counters and Sequence Generators:** Atomic sequence numbers for IDs.
- **Lock-Free Stacks and Queues:** `AtomicReference` for head/tail pointers.
- **State Machines:** Atomic state transitions.
- **Reference Counting:** Atomic reference counts in resource pools.

---

## 4. Compare-and-Swap (CAS)

### Definitions

**Core Definition:** Compare-and-swap (CAS) is an atomic instruction that updates a memory location only if it currently holds an expected value, returning whether the operation succeeded.

**Technical Definition:** "CAS(addr, old, new) writes the value new to location addr if this location contains the value old at the time of the write". In Java, the atomic classes use CAS via methods like `compareAndSet(expectedValue, newValue)`: "if value doesn't equal expectedValue, return false; if equal, store newValue in value and return true — executes as a single atomic action".

**Beginner-Friendly Explanation:** CAS is like a conditional update: "If the current value is what I expect, change it to the new value. Otherwise, don't change anything." It's a hardware-level operation that makes atomic updates possible without locks.

### Purposes

- To provide the foundation for lock-free atomic operations
- To enable optimistic concurrency without blocking
- To implement non-blocking data structures
- To avoid the overhead and risks of locks

### Syntax Rules and Structure

**CAS in Java:**

```java
// AtomicInteger
boolean success = atomicInt.compareAndSet(expectedValue, newValue);

// AtomicReference
boolean success = atomicRef.compareAndSet(expectedRef, newRef);

// AtomicIntegerArray (per element)
boolean success = atomicArray.compareAndSet(index, expectedValue, newValue);
```

**CAS Algorithm:**

```
1. Read the current value (V).
2. Compare V with the expected value (A).
3. If V == A, write the new value (B) atomically and return true.
4. If V != A, do not write; return false (another thread changed the value).
5. Retry the operation if needed (CAS loop).
```

**Syntax Rules:**

- `compareAndSet` returns `true` if the swap succeeded, `false` otherwise.
- CAS operates on a single memory location.
- CAS loops are used to retry when contention occurs.

**Constraints and Limitations:**

- "Compare and swap is god here — but it only atomically updates one variable at a time".
- The ABA problem can occur if a value changes A → B → A between the read and CAS.
- CAS loops can spin under high contention, wasting CPU cycles.

### Annotated Code Example

```java
// File: CasDemo.java
import java.util.concurrent.atomic.AtomicInteger;

public class CasDemo {
    public static void main(String[] args) {
        AtomicInteger counter = new AtomicInteger(10);

        // CAS loop: retry until successful
        int expected, newValue;
        do {
            expected = counter.get();
            newValue = expected + 1;
        } while (!counter.compareAndSet(expected, newValue));

        System.out.println("Counter: " + counter.get());  // 11

        // Failed CAS: expected value doesn't match
        boolean failed = counter.compareAndSet(999, 100);
        System.out.println("Failed CAS: " + failed);  // false
        System.out.println("Counter still: " + counter.get());  // 11
    }
}
```

**Expected Output:**
```
Counter: 11
Failed CAS: false
Counter still: 11
```

**Step-by-Step Explanation:**

1. `AtomicInteger` starts at 10.
2. The CAS loop reads the current value (10), computes the new value (11), and attempts CAS.
3. If CAS succeeds, the loop exits.
4. A second CAS with an incorrect expected value (999) fails, and the counter remains 11.

### Real-World Cases

- **Lock-Free Counters:** Implementing atomic counters without locks.
- **Non-Blocking Data Structures:** `ConcurrentLinkedQueue` uses CAS for head/tail updates.
- **Optimistic Concurrency:** Retrying operations when conflicts occur.

---

## 5. `LongAdder` and `LongAccumulator`

### Definitions

**Core Definition:** `LongAdder` and `LongAccumulator` are Java 8+ classes designed for high-performance atomic accumulation under high contention, where writes vastly outnumber reads.

**Technical Definition:** `LongAdder` maintains a set of internal variables (cells) that are summed to produce the final value. This reduces contention because different threads can update different cells. "In cases of multithreading applications that update frequently but read less frequently, it is recommended to rely on LongAdder, DoubleAdder, LongAccumulator, and DoubleAccumulator, instead of the AtomicFoo classes".

**Beginner-Friendly Explanation:** Imagine a busy restaurant where customers are constantly adding to the bill. Instead of one person writing on a single bill (contention), `LongAdder` uses multiple bills, and the final total is the sum of all of them. This is much faster when many threads are updating simultaneously.

### Purposes

- To provide high-performance atomic accumulation for high-contention scenarios
- To reduce contention by spreading updates across multiple cells
- To optimise for write-heavy workloads where reads are infrequent
- To offer a more scalable alternative to `AtomicLong` for counting

### Syntax Rules and Structure

**Complete General Syntax:**

```java
// LongAdder
LongAdder adder = new LongAdder();
adder.add(1);
adder.increment();
long sum = adder.sum();

// LongAccumulator
LongAccumulator accumulator = new LongAccumulator(Long::sum, 0);
accumulator.accumulate(10);
long result = accumulator.get();
```

**Key Differences:**

| Aspect | AtomicLong | LongAdder |
|--------|------------|-----------|
| Write performance | Low under contention | High under contention |
| Read performance | Constant | Sums all cells (slower) |
| Memory footprint | Small | Larger (multiple cells) |
| Use case | Balanced read/write | Write-heavy, rare reads |

**Syntax Rules:**

- `LongAdder` only supports addition (`add`, `increment`, `decrement`).
- `LongAccumulator` supports arbitrary accumulation functions via `LongBinaryOperator`.
- `sum()` and `get()` return the current total by summing all cells.

### Annotated Code Example

```java
// File: LongAdderDemo.java
import java.util.concurrent.atomic.LongAdder;
import java.util.concurrent.atomic.LongAccumulator;

public class LongAdderDemo {
    public static void main(String[] args) throws InterruptedException {
        LongAdder adder = new LongAdder();

        Runnable task = () -> {
            for (int i = 0; i < 100000; i++) {
                adder.increment();
            }
        };

        Thread t1 = new Thread(task);
        Thread t2 = new Thread(task);
        Thread t3 = new Thread(task);

        long start = System.nanoTime();
        t1.start(); t2.start(); t3.start();
        t1.join(); t2.join(); t3.join();
        long elapsed = System.nanoTime() - start;

        System.out.println("LongAdder: " + adder.sum() + " (" + elapsed / 1_000_000 + " ms)");

        // LongAccumulator for maximum
        LongAccumulator maxAcc = new LongAccumulator(Long::max, Long.MIN_VALUE);
        maxAcc.accumulate(10);
        maxAcc.accumulate(25);
        maxAcc.accumulate(15);
        System.out.println("Max: " + maxAcc.get());  // 25
    }
}
```

**Expected Output (times vary):**
```
LongAdder: 300000 (12 ms)
Max: 25
```

**Step-by-Step Explanation:**

1. `LongAdder` is used to accumulate 300,000 increments across three threads.
2. Each thread increments 100,000 times.
3. `sum()` returns the total by summing all internal cells.
4. `LongAccumulator` with `Long::max` finds the maximum value.

### Real-World Cases

- **High-Throughput Metrics:** Counting requests or events in web servers.
- **Statistics Collection:** Aggregating counters in monitoring systems.
- **Frequency Maps:** Counting word occurrences in concurrent text processing.

---

## 6. Array Atomic Classes

### Definitions

**Core Definition:** `AtomicIntegerArray`, `AtomicLongArray`, and `AtomicReferenceArray` provide atomic operations on individual elements of arrays.

**Technical Definition:** "The `AtomicIntegerArray`, `AtomicLongArray`, and `AtomicReferenceArray` classes further extend atomic operation support to arrays of these types. These classes are also notable in providing volatile access semantics for their array elements".

**Beginner-Friendly Explanation:** These classes are like regular arrays, but each element is a mini atomic variable. You can safely update individual elements from multiple threads without locking the entire array.

### Purposes

- To provide atomic operations on individual array elements
- To enable lock-free concurrent data structures based on arrays
- To avoid the overhead of synchronizing the entire array
- To support per-element atomic updates

### Syntax Rules and Structure

```java
// AtomicIntegerArray
AtomicIntegerArray array = new AtomicIntegerArray(10);
array.set(0, 42);
int val = array.get(0);
array.incrementAndGet(0);
array.compareAndSet(0, 43, 100);

// AtomicReferenceArray
AtomicReferenceArray<String> refArray = new AtomicReferenceArray<>(5);
refArray.set(0, "Hello");
refArray.compareAndSet(0, "Hello", "World");
```

### Annotated Code Example

```java
// File: AtomicArrayDemo.java
import java.util.concurrent.atomic.AtomicIntegerArray;

public class AtomicArrayDemo {
    public static void main(String[] args) throws InterruptedException {
        AtomicIntegerArray array = new AtomicIntegerArray(5);

        // Two threads incrementing different elements
        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                array.incrementAndGet(0);  // Element 0
            }
        });

        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                array.incrementAndGet(1);  // Element 1
            }
        });

        t1.start();
        t2.start();
        t1.join();
        t2.join();

        System.out.println("Element 0: " + array.get(0));  // 1000
        System.out.println("Element 1: " + array.get(1));  // 1000
    }
}
```

**Expected Output:**
```
Element 0: 1000
Element 1: 1000
```

**Step-by-Step Explanation:**

1. An `AtomicIntegerArray` of size 5 is created.
2. Thread 1 increments element 0 atomically 1000 times.
3. Thread 2 increments element 1 atomically 1000 times.
4. Both elements reach 1000 correctly, without any locks.

### Real-World Cases

- **Resource Pools:** Tracking usage of individual resources.
- **Parallel Histograms:** Counting occurrences per bucket.
- **Game Boards:** Atomic updates to individual cells.

---

## 7. Atomic Best Practices

### Definitions

**Core Definition:** Atomic best practices are guidelines for using `volatile` and atomic classes correctly, efficiently, and safely.

**Technical Definition:** Best practices derive from authoritative sources including the SEI CERT Oracle Coding Standard for Java. Key principles include using `volatile` only for visibility (not atomicity), preferring atomic classes for compound operations, and using adders/accumulators for high-contention scenarios.

**Beginner-Friendly Explanation:** Best practices help you choose the right tool for the job: `volatile` for simple flags, atomic classes for counters and references, and adders for high-contention scenarios.

### Key Best Practices Table

| Practice | Description | Source |
|----------|-------------|--------|
| Use `volatile` for visibility only | "Declaring a shared mutable variable volatile ensures visibility but does not guarantee atomicity of composite operations". | SEI CERT |
| Use atomic classes for compound operations | "Volatile variables are unsuitable when more than one load/store operation needs to be atomic". | SEI CERT |
| Prefer `LongAdder` over `AtomicLong` for high contention | "In cases of multithreading applications that update frequently but read less frequently, it is recommended to rely on LongAdder". | O'Reilly |
| Use `AtomicReference` for object references | Provides atomic updates for reference types. | Oracle |
| Use array atomic classes for per-element atomicity | "AtomicIntegerArray, AtomicLongArray, and AtomicReferenceArray classes further extend atomic operation support to arrays". | Oracle |
| Do not use atomic classes as hash keys | "They do not define methods such as equals, hashCode and compareTo". | Oracle |
| Use `compareAndSet` for conditional updates | CAS is the foundation of lock-free algorithms. | Best practice |
| Avoid CAS loops under high contention | Spin loops can waste CPU; consider locks or adders. | Best practice |
| Document your concurrency strategy | Explain why `volatile` or atomic classes are used. | Best practice |
| Prefer immutable objects with volatile references | "Declaring the reference volatile suffices to guarantee safe publication of the members of the referent". | SEI CERT |

### Annotated Complete Step-by-Step Code Examples

```java
// File: AtomicBestPracticesDemo.java
import java.util.concurrent.atomic.AtomicInteger;
import java.util.concurrent.atomic.LongAdder;
import java.util.concurrent.atomic.AtomicReference;

public class AtomicBestPracticesDemo {
    // BEST PRACTICE: Use AtomicInteger for compound operations
    private static final AtomicInteger atomicCounter = new AtomicInteger(0);

    // BEST PRACTICE: Use LongAdder for high-contention counting
    private static final LongAdder highContentionCounter = new LongAdder();

    // BEST PRACTICE: Use AtomicReference for object references
    private static final AtomicReference<String> state =
        new AtomicReference<>("initial");

    public static void main(String[] args) throws InterruptedException {
        Runnable atomicTask = () -> {
            for (int i = 0; i < 10000; i++) {
                atomicCounter.incrementAndGet();  // Atomic — no lost updates
            }
        };

        Thread t1 = new Thread(atomicTask);
        Thread t2 = new Thread(atomicTask);
        t1.start();
        t2.start();
        t1.join();
        t2.join();

        System.out.println("AtomicCounter: " + atomicCounter.get());  // 20000

        // AtomicReference update
        boolean updated = state.compareAndSet("initial", "running");
        System.out.println("State updated: " + updated);  // true
        System.out.println("State: " + state.get());      // running

        // LongAdder for high contention
        Runnable adderTask = () -> {
            for (int i = 0; i < 100000; i++) {
                highContentionCounter.increment();
            }
        };

        Thread t3 = new Thread(adderTask);
        Thread t4 = new Thread(adderTask);
        t3.start();
        t4.start();
        t3.join();
        t4.join();

        System.out.println("LongAdder: " + highContentionCounter.sum());  // 200000
    }
}
```

**Expected Output:**
```
AtomicCounter: 20000
State updated: true
State: running
LongAdder: 200000
```

**Step-by-Step Explanation:**

1. `AtomicInteger` correctly counts to 20,000 without lost updates.
2. `AtomicReference` atomically transitions the state from `"initial"` to `"running"`.
3. `LongAdder` efficiently accumulates 200,000 increments under high contention.

### Real-World Cases

- **Web Servers:** Using `LongAdder` for request counters under high load.
- **State Machines:** Using `AtomicReference` for safe state transitions.
- **Resource Pools:** Using `AtomicIntegerArray` for per-resource tracking.

---

## References Links

- Package java.util.concurrent.atomic — Java Platform SE 17 API Specification - https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/concurrent/atomic/package-summary.html
- VNA06-J. Do not assume that declaring an object reference volatile guarantees visibility of its members — SEI CERT - https://wiki.sei.cmu.edu/confluence/pages/viewpage.action?pageId=88872069
- CON00-J. Declare shared variables as volatile to ensure visibility and prevent reordering of statements — SEI CERT - https://wiki.sei.cmu.edu/confluence/pages/viewpage.action?pageId=88505558
- Java Coding Problems — Adders and Accumulators (O'Reilly) - https://www.oreilly.com/library/view/java-coding-problems/9781789801415/
- Compare and Set (CAS) — Cornell University - https://www.cs.cornell.edu
- VNA00-J. Ensure visibility when accessing shared primitive variables — SEI CERT - https://wiki.sei.cmu.edu/confluence/pages/viewpage.action?pageId=88505573
- Java Volatile Keyword — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/essential/concurrency/atomic.html
- AtomicReferenceArray Example — LeetCode Discussion - https://leetcode.com/discuss/post/8483792/java-concurrency-question-design-thread-dhxd0/