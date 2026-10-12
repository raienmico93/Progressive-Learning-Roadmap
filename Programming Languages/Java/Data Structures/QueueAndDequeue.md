# Java Queues and Deques: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A `Queue` is a collection designed for holding elements prior to processing, typically ordering elements in a first-in-first-out (FIFO) manner. A `Deque` (double-ended queue) is a linear collection that supports element insertion and removal at both ends.

**Technical Definition:** Per the Java API specification, the `Queue` interface is "a collection designed for holding elements prior to processing. Besides basic `Collection` operations, queues provide additional insertion, extraction, and inspection operations". The `Deque` interface is "a linear collection that supports element insertion and removal at both ends. The name deque is short for 'double ended queue' and is usually pronounced 'deck'". Queues typically, but do not necessarily, order elements in a FIFO manner; exceptions include priority queues and LIFO stacks. The `Deque` interface extends `Queue` and provides methods for both ends, enabling use as both a queue and a stack.

**Beginner-Friendly Explanation:** A `Queue` is like a line at a ticket counter — the first person in line is the first person served (FIFO). A `Deque` is like a deck of cards — you can add or remove cards from either the top or the bottom. Java provides many queue implementations for different needs: some are fast and simple, some keep items in priority order, and some are designed for safe use by multiple threads at the same time.

### Key Characteristics

- **FIFO Ordering:** Most queues order elements in first-in-first-out order, though `PriorityQueue` orders by priority and `Deque` supports LIFO (stack) behaviour.
- **Dual-Form Operations:** Queue methods come in two forms — one throws an exception on failure, the other returns a special value (`null` or `false`).
- **No Null Elements:** Most queue implementations do not permit `null` elements, as `null` is used as a sentinel return value by `poll()` and `peek()`.
- **Blocking Operations:** `BlockingQueue` adds methods that wait for the queue to become non-empty or for space to become available, essential for producer-consumer scenarios.
- **Concurrent Implementations:** `ConcurrentLinkedQueue` and `ConcurrentLinkedDeque` provide thread-safe, lock-free operations for high-concurrency environments.
- **Priority Ordering:** `PriorityQueue` and `PriorityBlockingQueue` order elements by natural ordering or a supplied `Comparator`.

### Prerequisites

- Basic understanding of Java classes, objects, and interfaces
- Familiarity with generics (`Queue<String>`, `Deque<Integer>`, etc.)
- Knowledge of the `Collection` interface and the for-each loop
- Awareness of the `java.util` and `java.util.concurrent` packages
- Understanding of threads and concurrency (for blocking and concurrent queues)

### Related Programming Areas

- **Producer-Consumer Patterns:** Blocking queues are the backbone of producer-consumer coordination.
- **Thread Pools:** `ExecutorService` uses blocking queues to hold tasks awaiting execution.
- **Breadth-First Search:** Queues are used to explore graph or tree levels.
- **Task Scheduling:** `PriorityQueue` and `DelayQueue` schedule tasks by priority or time.
- **Message Passing:** `TransferQueue` enables handoff designs between threads.
- **Backpressure Handling:** Bounded queues provide natural backpressure in concurrent systems.

---

## Core Concepts / Features

---

## 1. Queue Interface

### Definitions

**Core Definition:** The `Queue` interface is the abstract data type that defines the contract for collections designed for holding elements prior to processing.

**Technical Definition:** `Queue<E>` extends `Collection<E>` and provides additional insertion, extraction, and inspection operations. Each operation exists in two forms: one throws an exception on failure, the other returns a special value (`null` or `false`). The latter form is designed for use with capacity-restricted implementations. The `offer` method inserts an element if possible, returning `false` otherwise; the `remove` and `poll` methods remove and return the head; and the `element` and `peek` methods return the head without removing it.

**Beginner-Friendly Explanation:** The `Queue` interface is the "rulebook" that all queue types follow. It says what you can do with a queue — add items, remove items, look at the first item — and defines two ways to do each thing: one that throws an error if something goes wrong, and one that returns a special value instead.

### Purposes

- To define a standard contract for collections that hold elements for processing
- To provide multiple ways of handling operations that may fail (exception vs. special value)
- To serve as the foundation for more specialized queue types (`Deque`, `BlockingQueue`)

### Syntax Rules and Structure

```java
public interface Queue<E> extends Collection<E> {
    // Insert
    boolean add(E e);      // Throws IllegalStateException if no space
    boolean offer(E e);    // Returns false if no space

    // Remove
    E remove();            // Throws NoSuchElementException if empty
    E poll();              // Returns null if empty

    // Examine
    E element();           // Throws NoSuchElementException if empty
    E peek();              // Returns null if empty
}
```

**Component Breakdown:**

| Method | Description | Failure Behaviour |
|--------|-------------|-------------------|
| `add(E e)` | Inserts element | Throws `IllegalStateException` |
| `offer(E e)` | Inserts element | Returns `false` |
| `remove()` | Removes and returns head | Throws `NoSuchElementException` |
| `poll()` | Removes and returns head | Returns `null` |
| `element()` | Returns head without removing | Throws `NoSuchElementException` |
| `peek()` | Returns head without removing | Returns `null` |

**Syntax Rules:**

- `Queue` is a generic interface (`Queue<E>`).
- `Queue` extends `Collection`, so all `Collection` methods are available.
- `null` elements are generally not permitted.

**Constraints and Limitations:**

- The `Queue` interface does not define blocking methods; these are defined in `BlockingQueue`.
- Every `Queue` implementation must specify its ordering properties.

### Annotated Code Example

```java
// File: QueueInterfaceDemo.java
import java.util.*;

public class QueueInterfaceDemo {
    public static void main(String[] args) {
        Queue<String> queue = new LinkedList<>();

        // offer() returns false if no space (never for unbounded queue)
        queue.offer("Task 1");
        queue.offer("Task 2");
        queue.offer("Task 3");
        System.out.println("Queue: " + queue);

        // peek() returns null if empty
        System.out.println("Head: " + queue.peek());

        // poll() returns null if empty
        System.out.println("Removed: " + queue.poll());
        System.out.println("Removed: " + queue.poll());
        System.out.println("Queue after polls: " + queue);
    }
}
```

**Expected Output:**
```
Queue: [Task 1, Task 2, Task 3]
Head: Task 1
Removed: Task 1
Removed: Task 2
Queue after polls: [Task 3]
```

**Step-by-Step Explanation:**

1. `queue.offer("Task 1")` adds elements in FIFO order.
2. `queue.peek()` returns "Task 1" without removing it.
3. `queue.poll()` removes and returns the head each time.
4. After two polls, "Task 3" remains.

### Real-World Cases

- **Task Scheduling:** A `Queue<Runnable>` holds tasks awaiting execution by a thread pool.
- **Breadth-First Search:** A `Queue<Node>` tracks nodes to visit in level order.
- **Print Spooler:** A `Queue<Document>` holds documents awaiting printing.

---

## 2. Deque Interface

### Definitions

**Core Definition:** The `Deque` interface is a linear collection that supports element insertion and removal at both ends.

**Technical Definition:** `Deque<E>` extends `Queue<E>` and provides methods for accessing elements at both the front and the back of the deque. The interface defines twelve methods for insertion, removal, and inspection, each in two forms (exception-throwing and special-value-returning). When used as a queue, `Deque` exhibits FIFO behaviour; when used as a stack, it exhibits LIFO behaviour. The legacy `Stack` class should be replaced by `Deque`.

**Beginner-Friendly Explanation:** A `Deque` is a "double-ended queue" — you can add or remove items from either the front or the back. This makes it versatile: it can act as a regular queue (FIFO) or as a stack (LIFO). It's like a deck of cards where you can take from the top or the bottom.

### Purposes

- To provide double-ended queue operations (insert/remove at both ends)
- To serve as both a queue (FIFO) and a stack (LIFO)
- To replace the legacy `Stack` class with a more consistent and complete API

### Syntax Rules and Structure

```java
public interface Deque<E> extends Queue<E> {
    // Insert
    void addFirst(E e);       // Throws if no space
    void addLast(E e);        // Throws if no space
    boolean offerFirst(E e);  // Returns false if no space
    boolean offerLast(E e);   // Returns false if no space

    // Remove
    E removeFirst();          // Throws if empty
    E removeLast();           // Throws if empty
    E pollFirst();            // Returns null if empty
    E pollLast();             // Returns null if empty

    // Examine
    E getFirst();             // Throws if empty
    E getLast();              // Throws if empty
    E peekFirst();            // Returns null if empty
    E peekLast();             // Returns null if empty
}
```

**Component Breakdown:**

| First Element (Head) | Last Element (Tail) |
|----------------------|---------------------|
| `addFirst(e)` / `offerFirst(e)` | `addLast(e)` / `offerLast(e)` |
| `removeFirst()` / `pollFirst()` | `removeLast()` / `pollLast()` |
| `getFirst()` / `peekFirst()` | `getLast()` / `peekLast()` |

**Syntax Rules:**

- `Deque` is a generic interface (`Deque<E>`).
- `Deque` extends `Queue`, so all `Queue` methods are available.
- When used as a stack, `push()` = `addFirst()` and `pop()` = `removeFirst()`.

**Constraints and Limitations:**

- `null` elements are strongly discouraged because `null` is used as a special return value.
- `Deque` does not support indexed access like `List`.

### Annotated Code Example

```java
// File: DequeInterfaceDemo.java
import java.util.*;

public class DequeInterfaceDemo {
    public static void main(String[] args) {
        Deque<String> deque = new ArrayDeque<>();

        // Adding at both ends
        deque.addFirst("Middle");
        deque.addFirst("Front");
        deque.addLast("Back");
        System.out.println("Deque: " + deque); // [Front, Middle, Back]

        // Stack operations (LIFO)
        deque.push("Top"); // Same as addFirst
        System.out.println("After push: " + deque);
        System.out.println("Pop: " + deque.pop()); // Removes "Top"

        // Queue operations (FIFO)
        System.out.println("Poll: " + deque.poll()); // Removes "Front"

        // Peek at both ends
        System.out.println("First: " + deque.peekFirst());
        System.out.println("Last: " + deque.peekLast());
    }
}
```

**Expected Output:**
```
Deque: [Front, Middle, Back]
After push: [Top, Front, Middle, Back]
Pop: Top
Poll: Front
First: Middle
Last: Back
```

**Step-by-Step Explanation:**

1. `deque.addFirst("Front")` adds at the front.
2. `deque.addLast("Back")` adds at the end.
3. `deque.push("Top")` is equivalent to `addFirst` (stack behaviour).
4. `deque.pop()` removes from the front (LIFO).
5. `deque.poll()` removes from the front (FIFO).

### Real-World Cases

- **Undo/Redo:** A `Deque` can store undo operations at one end and redo operations at the other.
- **Palindrome Checker:** Add characters to both ends and compare.
- **Sliding Window:** Maintain a window of elements at both ends.
- **Replacing Stack:** Use `ArrayDeque` instead of the legacy `Stack` class.

---

## 3. Queue Operations

### Definitions

**Core Definition:** Queue operations are the methods provided by the `Queue`, `Deque`, and `BlockingQueue` interfaces for inserting, removing, and inspecting elements.

**Technical Definition:** The `Queue` interface provides six core methods in two forms each: `add`/`offer` for insertion, `remove`/`poll` for removal, and `element`/`peek` for inspection. The `BlockingQueue` interface adds `put` (blocking insert) and `take` (blocking remove). The `Deque` interface adds twelve methods for accessing both ends.

**Beginner-Friendly Explanation:** Queue operations are the actions you can perform on a queue — adding items, removing items, and looking at the first item. Some operations throw an error if they fail, while others return a special value. Blocking queues add operations that wait until they can succeed.

### add() / offer()

**Definition:** Inserts an element into the queue. `add()` throws an exception if the queue is full; `offer()` returns `false`.

**Syntax:**
```java
boolean add(E e);      // Throws IllegalStateException if no space
boolean offer(E e);    // Returns false if no space
```

**Example:**
```java
Queue<String> queue = new LinkedList<>();
queue.offer("Task"); // Preferred for capacity-restricted queues
```

### remove() / poll()

**Definition:** Removes and returns the head of the queue. `remove()` throws an exception if the queue is empty; `poll()` returns `null`.

**Syntax:**
```java
E remove();    // Throws NoSuchElementException if empty
E poll();      // Returns null if empty
```

**Example:**
```java
String head = queue.poll(); // Returns null if empty
```

### element() / peek()

**Definition:** Returns the head of the queue without removing it. `element()` throws an exception if the queue is empty; `peek()` returns `null`.

**Syntax:**
```java
E element();   // Throws NoSuchElementException if empty
E peek();      // Returns null if empty
```

**Example:**
```java
String first = queue.peek(); // Returns null if empty
```

### put() / take()

**Definition:** Blocking operations defined in `BlockingQueue`. `put()` waits for space to become available; `take()` waits for an element to become available.

**Syntax:**
```java
void put(E e) throws InterruptedException;  // Blocks until space available
E take() throws InterruptedException;        // Blocks until element available
```

**Example:**
```java
BlockingQueue<String> queue = new ArrayBlockingQueue<>(10);
queue.put("Task");  // Blocks if queue is full
String task = queue.take(); // Blocks if queue is empty
```

---

## 4. PriorityQueue

### Definitions

**Core Definition:** `PriorityQueue` is an unbounded priority queue based on a priority heap, ordering elements by natural ordering or a supplied `Comparator`.

**Technical Definition:** `PriorityQueue<E>` extends `AbstractQueue<E>` and implements `Serializable`. The elements are ordered according to their natural ordering or by a `Comparator` provided at construction time. The head of the queue is the least element with respect to the specified ordering. It does not permit `null` elements and does not permit insertion of non-comparable objects when relying on natural ordering. This implementation provides O(log n) time for enqueueing and dequeueing, linear time for `remove(Object)` and `contains(Object)`, and constant time for retrieval methods.

**Beginner-Friendly Explanation:** `PriorityQueue` is like a hospital emergency room — patients with more urgent conditions are treated first, regardless of when they arrived. You can add items, and when you remove one, you always get the most important one.

### Purposes

- To process elements in priority order rather than insertion order
- To implement priority-based scheduling and task management
- To provide efficient O(log n) insertion and removal of the highest-priority element

### Syntax Rules and Structure

```java
public class PriorityQueue<E> extends AbstractQueue<E> implements Serializable {
    public PriorityQueue();
    public PriorityQueue(int initialCapacity);
    public PriorityQueue(Comparator<? super E> comparator);
    public PriorityQueue(Collection<? extends E> c);
}
```

**Syntax Rules:**

- Elements must be `Comparable` (for natural ordering) or a `Comparator` must be provided.
- `null` elements are not permitted.
- The iterator does not guarantee traversal in any particular order.
- Not thread-safe; use `PriorityBlockingQueue` for concurrent use.

**Constraints and Limitations:**

- `remove(Object)` and `contains(Object)` are linear time.
- Iteration order is not sorted order.

### Annotated Code Example

```java
// File: PriorityQueueDemo.java
import java.util.*;

public class PriorityQueueDemo {
    public static void main(String[] args) {
        PriorityQueue<Integer> pq = new PriorityQueue<>();

        pq.offer(5);
        pq.offer(2);
        pq.offer(8);
        pq.offer(1);
        pq.offer(9);

        System.out.println("PriorityQueue: " + pq); // Not sorted!
        System.out.println("Poll order:");
        while (!pq.isEmpty()) {
            System.out.println(pq.poll()); // 1, 2, 5, 8, 9
        }
    }
}
```

**Expected Output:**
```
PriorityQueue: [1, 2, 5, 8, 9] (order may vary)
Poll order:
1
2
5
8
9
```

**Step-by-Step Explanation:**

1. Elements are inserted in arbitrary order.
2. The internal heap structure ensures the smallest element is always at the head.
3. `poll()` removes the smallest element each time, producing sorted output.

### Real-World Cases

- **Task Scheduling:** Process high-priority tasks before low-priority ones.
- **Dijkstra's Algorithm:** Find the shortest path in a graph.
- **Event Simulation:** Process events in chronological order.

---

## 5. ArrayDeque

### Definitions

**Core Definition:** `ArrayDeque` is a resizable-array implementation of the `Deque` interface, providing fast operations at both ends.

**Technical Definition:** `ArrayDeque<E>` extends `AbstractCollection<E>` and implements `Deque<E>`, `Cloneable`, and `Serializable`. It has no capacity restrictions; it grows as necessary. It is not thread-safe and prohibits `null` elements. It is likely to be faster than `Stack` when used as a stack and faster than `LinkedList` when used as a queue. Most operations run in amortized constant time.

**Beginner-Friendly Explanation:** `ArrayDeque` is the go-to implementation for both stacks and queues. It's fast, efficient, and doesn't have the overhead of `LinkedList`'s node objects.

### Purposes

- To provide a fast, resizable-array implementation of `Deque`
- To replace both `Stack` (for LIFO) and `LinkedList` (for FIFO)
- To offer amortized O(1) operations at both ends

### Syntax Rules and Structure

```java
public class ArrayDeque<E> extends AbstractCollection<E>
        implements Deque<E>, Cloneable, Serializable {
    public ArrayDeque();
    public ArrayDeque(int numElements);
    public ArrayDeque(Collection<? extends E> c);
}
```

**Syntax Rules:**

- `ArrayDeque` is not thread-safe.
- `null` elements are prohibited.
- Iterators are fail-fast.

**Constraints and Limitations:**

- `remove(Object)`, `contains(Object)`, and bulk operations are linear time.
- Not suitable for concurrent use without external synchronization.

### Annotated Code Example

```java
// File: ArrayDequeDemo.java
import java.util.*;

public class ArrayDequeDemo {
    public static void main(String[] args) {
        ArrayDeque<String> deque = new ArrayDeque<>();

        // As a queue (FIFO)
        deque.offer("A");
        deque.offer("B");
        deque.offer("C");
        System.out.println("Queue: " + deque);
        System.out.println("Poll: " + deque.poll()); // A

        // As a stack (LIFO)
        deque.push("X");
        System.out.println("Stack: " + deque);
        System.out.println("Pop: " + deque.pop()); // X
    }
}
```

**Expected Output:**
```
Queue: [A, B, C]
Poll: A
Stack: [X, B, C]
Pop: X
```

**Step-by-Step Explanation:**

1. `offer()` adds elements to the tail (queue behaviour).
2. `poll()` removes from the head (FIFO).
3. `push()` adds to the head (stack behaviour).
4. `pop()` removes from the head (LIFO).

### Real-World Cases

- **General-Purpose Queue:** The default choice for non-concurrent queue operations.
- **Stack Replacement:** Replace the legacy `Stack` class.
- **Sliding Window:** Maintain a window of elements.

---

## 6. LinkedList

### Definitions

**Core Definition:** `LinkedList` is a doubly-linked list implementation of both `List` and `Deque` interfaces.

**Technical Definition:** `LinkedList<E>` extends `AbstractSequentialList<E>` and implements `List<E>`, `Deque<E>`, `Cloneable`, and `Serializable`. It provides efficient insertion and removal at both ends but has O(n) positional access. As a `Deque`, it can be used as a queue (FIFO) or stack (LIFO).

**Beginner-Friendly Explanation:** `LinkedList` is a queue implementation backed by a chain of nodes. It's flexible but generally slower than `ArrayDeque` for queue operations.

### Purposes

- To provide a linked-list-based implementation of `Deque`
- To serve as both a `List` and a `Deque`

### Syntax Rules and Structure

```java
public class LinkedList<E> extends AbstractSequentialList<E>
        implements List<E>, Deque<E>, Cloneable, Serializable {
    public LinkedList();
    public LinkedList(Collection<? extends E> c);
}
```

**Syntax Rules:**

- `LinkedList` is not thread-safe.
- It permits `null` elements.
- It implements both `List` and `Deque`.

**Constraints and Limitations:**

- Queue operations are generally slower than `ArrayDeque`.
- `LinkedList` almost never outperforms `ArrayDeque` for queue operations.

### Annotated Code Example

```java
// File: LinkedListQueueDemo.java
import java.util.*;

public class LinkedListQueueDemo {
    public static void main(String[] args) {
        Queue<String> queue = new LinkedList<>();
        queue.offer("Task 1");
        queue.offer("Task 2");
        queue.offer("Task 3");
        System.out.println("Queue: " + queue);
        System.out.println("Poll: " + queue.poll());
        System.out.println("Queue after poll: " + queue);
    }
}
```

**Expected Output:**
```
Queue: [Task 1, Task 2, Task 3]
Poll: Task 1
Queue after poll: [Task 2, Task 3]
```

**Step-by-Step Explanation:**

1. `offer()` adds elements to the tail.
2. `poll()` removes from the head.

### Real-World Cases

- **Legacy Code:** Old applications that use `LinkedList` as a queue.
- **List + Queue:** When both list and queue functionality are needed.

---

## 7. BlockingQueue

### Definitions

**Core Definition:** `BlockingQueue` is a `Queue` that additionally supports operations that wait for the queue to become non-empty when retrieving an element, and wait for space to become available when storing an element.

**Technical Definition:** `BlockingQueue<E>` extends `Queue<E>` and provides four forms of operations: one throws an exception, one returns a special value, one blocks indefinitely, and one blocks for a given time limit. It does not accept `null` elements. It is designed primarily for producer-consumer queues and is thread-safe.

**Beginner-Friendly Explanation:** A `BlockingQueue` is a queue that can make threads wait. If you try to take an item from an empty queue, your thread waits until an item appears. If you try to put an item into a full queue, your thread waits until space is available. This makes it perfect for producer-consumer scenarios.

### Purposes

- To coordinate producer and consumer threads
- To provide blocking operations for concurrent programming
- To serve as the foundation for thread pools and message queues

### Syntax Rules and Structure

```java
public interface BlockingQueue<E> extends Queue<E> {
    // Blocking insert
    void put(E e) throws InterruptedException;
    boolean offer(E e, long timeout, TimeUnit unit) throws InterruptedException;

    // Blocking remove
    E take() throws InterruptedException;
    E poll(long timeout, TimeUnit unit) throws InterruptedException;
}
```

**Component Breakdown:**

| Operation | Throws Exception | Special Value | Blocks | Times Out |
|-----------|-----------------|---------------|--------|-----------|
| Insert | `add(e)` | `offer(e)` | `put(e)` | `offer(e, time, unit)` |
| Remove | `remove()` | `poll()` | `take()` | `poll(time, unit)` |
| Examine | `element()` | `peek()` | N/A | N/A |

**Syntax Rules:**

- `BlockingQueue` does not accept `null` elements.
- Implementations are thread-safe.
- Bulk operations are not necessarily atomic.

**Constraints and Limitations:**

- Blocking operations can be interrupted.
- A `BlockingQueue` does not support "close" or "shutdown" operations.

### Annotated Code Example

```java
// File: BlockingQueueDemo.java
import java.util.concurrent.*;

public class BlockingQueueDemo {
    public static void main(String[] args) throws InterruptedException {
        BlockingQueue<String> queue = new ArrayBlockingQueue<>(5);

        // Producer thread
        Thread producer = new Thread(() -> {
            try {
                for (int i = 1; i <= 5; i++) {
                    queue.put("Item " + i);
                    System.out.println("Produced: Item " + i);
                }
            } catch (InterruptedException e) { Thread.currentThread().interrupt(); }
        });

        // Consumer thread
        Thread consumer = new Thread(() -> {
            try {
                for (int i = 1; i <= 5; i++) {
                    String item = queue.take();
                    System.out.println("Consumed: " + item);
                }
            } catch (InterruptedException e) { Thread.currentThread().interrupt(); }
        });

        producer.start();
        consumer.start();
        producer.join();
        consumer.join();
    }
}
```

**Expected Output:**
```
Produced: Item 1
Consumed: Item 1
Produced: Item 2
Consumed: Item 2
...
```

**Step-by-Step Explanation:**

1. The producer thread calls `put()` to add items.
2. The consumer thread calls `take()` to retrieve items.
3. `put()` blocks if the queue is full; `take()` blocks if the queue is empty.
4. The output shows interleaved production and consumption.

### Real-World Cases

- **Producer-Consumer:** Coordinating data production and consumption between threads.
- **Thread Pools:** `ThreadPoolExecutor` uses a `BlockingQueue` to hold tasks.
- **Message Queues:** In-memory message passing between components.

---

## 8. ArrayBlockingQueue

### Definitions

**Core Definition:** `ArrayBlockingQueue` is a bounded blocking queue backed by an array, ordering elements FIFO.

**Technical Definition:** `ArrayBlockingQueue<E>` extends `AbstractQueue<E>` and implements `BlockingQueue<E>`, `Serializable`. It is a classic "bounded buffer" in which a fixed-sized array holds elements inserted by producers and extracted by consumers. It orders elements FIFO. The queue's capacity is fixed at construction and cannot be changed.

**Beginner-Friendly Explanation:** `ArrayBlockingQueue` is a blocking queue with a fixed maximum size. Once it's full, producers must wait until a consumer removes an item.

### Purposes

- To provide a bounded blocking queue with predictable memory usage
- To implement the classic bounded buffer pattern
- To support fair or non-fair thread access

### Syntax Rules and Structure

```java
public class ArrayBlockingQueue<E> extends AbstractQueue<E>
        implements BlockingQueue<E>, Serializable {
    public ArrayBlockingQueue(int capacity);
    public ArrayBlockingQueue(int capacity, boolean fair);
    public ArrayBlockingQueue(int capacity, boolean fair, Collection<? extends E> c);
}
```

**Syntax Rules:**

- Capacity is fixed and must be specified at construction.
- `fair` parameter controls whether blocked threads are processed in FIFO order.
- Does not permit `null` elements.

**Constraints and Limitations:**

- Fixed capacity; cannot grow.
- Slightly lower throughput than `LinkedBlockingQueue`.

### Annotated Code Example

```java
// File: ArrayBlockingQueueDemo.java
import java.util.concurrent.*;

public class ArrayBlockingQueueDemo {
    public static void main(String[] args) throws InterruptedException {
        BlockingQueue<Integer> queue = new ArrayBlockingQueue<>(3);
        queue.put(1);
        queue.put(2);
        queue.put(3);
        System.out.println("Queue: " + queue);
        System.out.println("Take: " + queue.take()); // 1
        System.out.println("Queue after take: " + queue);
    }
}
```

**Expected Output:**
```
Queue: [1, 2, 3]
Take: 1
Queue after take: [2, 3]
```

**Step-by-Step Explanation:**

1. Three elements are inserted into the capacity-3 queue.
2. `take()` removes the head element.

### Real-World Cases

- **Bounded Buffers:** Classic producer-consumer with fixed buffer size.
- **Rate Limiting:** Limiting the number of pending tasks.
- **Backpressure:** Naturally slowing producers when consumers are slow.

---

## 9. LinkedBlockingQueue

### Definitions

**Core Definition:** `LinkedBlockingQueue` is an optionally-bounded blocking queue based on linked nodes, ordering elements FIFO.

**Technical Definition:** `LinkedBlockingQueue<E>` extends `AbstractQueue<E>` and implements `BlockingQueue<E>`, `Serializable`. It is based on linked nodes and is optionally bounded (default capacity is `Integer.MAX_VALUE`). Linked queues typically have higher throughput than array-based queues but less predictable performance.

**Beginner-Friendly Explanation:** `LinkedBlockingQueue` is a blocking queue that uses linked nodes instead of an array. It can be bounded or unbounded (with a very large default capacity).

### Purposes

- To provide a blocking queue with higher throughput than `ArrayBlockingQueue`
- To support optionally-bounded queues
- To serve as the default queue for thread pools

### Syntax Rules and Structure

```java
public class LinkedBlockingQueue<E> extends AbstractQueue<E>
        implements BlockingQueue<E>, Serializable {
    public LinkedBlockingQueue();
    public LinkedBlockingQueue(int capacity);
    public LinkedBlockingQueue(Collection<? extends E> c);
}
```

**Syntax Rules:**

- Default capacity is `Integer.MAX_VALUE` (effectively unbounded).
- Does not permit `null` elements.

**Constraints and Limitations:**

- Higher memory overhead per element (node objects).
- Less predictable performance than array-based queues.

### Annotated Code Example

```java
// File: LinkedBlockingQueueDemo.java
import java.util.concurrent.*;

public class LinkedBlockingQueueDemo {
    public static void main(String[] args) throws InterruptedException {
        BlockingQueue<String> queue = new LinkedBlockingQueue<>(10);
        queue.put("A");
        queue.put("B");
        System.out.println("Queue: " + queue);
        System.out.println("Take: " + queue.take());
    }
}
```

**Expected Output:**
```
Queue: [A, B]
Take: A
```

### Real-World Cases

- **Thread Pools:** Default queue for `Executors.newFixedThreadPool()`.
- **Message Queues:** High-throughput message passing.

---

## 10. PriorityBlockingQueue

### Definitions

**Core Definition:** `PriorityBlockingQueue` is an unbounded blocking queue that uses the same ordering rules as `PriorityQueue` and supplies blocking retrieval operations.

**Technical Definition:** `PriorityBlockingQueue<E>` extends `AbstractQueue<E>` and implements `BlockingQueue<E>`, `Serializable`. It is unbounded and orders elements by natural ordering or a `Comparator`. While logically unbounded, it may fail with `OutOfMemoryError` if too many elements are added.

**Beginner-Friendly Explanation:** `PriorityBlockingQueue` is a thread-safe priority queue. It's like `PriorityQueue` but with blocking operations for use in concurrent programs.

### Purposes

- To provide a thread-safe priority queue
- To support blocking retrieval in priority order
- To implement priority-based task scheduling in concurrent systems

### Syntax Rules and Structure

```java
public class PriorityBlockingQueue<E> extends AbstractQueue<E>
        implements BlockingQueue<E>, Serializable {
    public PriorityBlockingQueue();
    public PriorityBlockingQueue(int initialCapacity);
    public PriorityBlockingQueue(int initialCapacity, Comparator<? super E> comparator);
    public PriorityBlockingQueue(Collection<? extends E> c);
}
```

**Syntax Rules:**

- Elements must be `Comparable` or a `Comparator` must be provided.
- Does not permit `null` elements.
- Unbounded; `put()` never blocks.

**Constraints and Limitations:**

- Unbounded growth may lead to `OutOfMemoryError`.
- Iteration order is not sorted order.

### Annotated Code Example

```java
// File: PriorityBlockingQueueDemo.java
import java.util.concurrent.*;

public class PriorityBlockingQueueDemo {
    public static void main(String[] args) throws InterruptedException {
        PriorityBlockingQueue<Integer> pq = new PriorityBlockingQueue<>();
        pq.put(5);
        pq.put(2);
        pq.put(8);
        pq.put(1);

        System.out.println("Take order:");
        while (!pq.isEmpty()) {
            System.out.println(pq.take()); // 1, 2, 5, 8
        }
    }
}
```

**Expected Output:**
```
Take order:
1
2
5
8
```

### Real-World Cases

- **Priority Task Scheduling:** Process high-priority tasks first in concurrent systems.
- **Event Processing:** Process events in priority order.

---

## 11. SynchronousQueue

### Definitions

**Core Definition:** `SynchronousQueue` is a blocking queue in which each insert operation must wait for a corresponding remove operation by another thread, and vice versa.

**Technical Definition:** `SynchronousQueue<E>` extends `AbstractQueue<E>` and implements `BlockingQueue<E>`, `Serializable`. It has no internal capacity, not even a capacity of one. You cannot `peek()` at a synchronous queue because an element is only present when you try to remove it. It is well suited for handoff designs, where one thread must synchronize with another to hand off information, events, or tasks.

**Beginner-Friendly Explanation:** `SynchronousQueue` is a queue with zero capacity. A producer must wait for a consumer to be ready before it can hand off an item, and a consumer must wait for a producer. It's like a direct handoff — no intermediate storage.

### Purposes

- To provide a direct handoff mechanism between threads
- To implement rendezvous-style synchronization
- To serve as the work queue for cached thread pools

### Syntax Rules and Structure

```java
public class SynchronousQueue<E> extends AbstractQueue<E>
        implements BlockingQueue<E>, Serializable {
    public SynchronousQueue();
    public SynchronousQueue(boolean fair);
}
```

**Syntax Rules:**

- No internal capacity.
- `put()` blocks until another thread calls `take()`.
- `take()` blocks until another thread calls `put()`.
- Does not permit `null` elements.

**Constraints and Limitations:**

- Cannot `peek()` because no element is ever stored.
- `isEmpty()` always returns `true`.

### Annotated Code Example

```java
// File: SynchronousQueueDemo.java
import java.util.concurrent.*;

public class SynchronousQueueDemo {
    public static void main(String[] args) throws InterruptedException {
        SynchronousQueue<String> queue = new SynchronousQueue<>();

        Thread producer = new Thread(() -> {
            try {
                System.out.println("Producer: putting");
                queue.put("Handoff");
                System.out.println("Producer: put completed");
            } catch (InterruptedException e) { Thread.currentThread().interrupt(); }
        });

        Thread consumer = new Thread(() -> {
            try {
                Thread.sleep(1000);
                System.out.println("Consumer: taking");
                String item = queue.take();
                System.out.println("Consumer: got " + item);
            } catch (InterruptedException e) { Thread.currentThread().interrupt(); }
        });

        producer.start();
        consumer.start();
        producer.join();
        consumer.join();
    }
}
```

**Expected Output:**
```
Producer: putting
Consumer: taking
Consumer: got Handoff
Producer: put completed
```

**Step-by-Step Explanation:**

1. The producer calls `put()` and blocks because no consumer is waiting.
2. After 1 second, the consumer calls `take()`.
3. The handoff occurs, and both threads proceed.

### Real-World Cases

- **Cached Thread Pools:** `Executors.newCachedThreadPool()` uses `SynchronousQueue`.
- **Direct Handoff:** Handing off tasks directly between threads.
- **Rendezvous Synchronization:** Two threads meeting at a synchronization point.

---

## 12. DelayQueue

### Definitions

**Core Definition:** `DelayQueue` is an unbounded blocking queue of `Delayed` elements, in which an element can only be taken when its delay has expired.

**Technical Definition:** `DelayQueue<E extends Delayed>` extends `AbstractQueue<E>` and implements `BlockingQueue<E>`, `Serializable`. The head of the queue is the `Delayed` element whose delay expired furthest in the past. If no delay has expired, there is no head and `poll()` returns `null`. Expiration occurs when an element's `getDelay(TimeUnit.NANOSECONDS)` method returns a value less than or equal to zero.

**Beginner-Friendly Explanation:** `DelayQueue` is a queue where items become available only after a certain amount of time has passed. It's like setting a timer on each item — you can't take it out until the timer goes off.

### Purposes

- To schedule tasks to be executed after a delay
- To implement timeout-based processing
- To manage resources with expiration times

### Syntax Rules and Structure

```java
public class DelayQueue<E extends Delayed> extends AbstractQueue<E>
        implements BlockingQueue<E>, Serializable {
    public DelayQueue();
    public DelayQueue(Collection<? extends E> c);
}
```

**Syntax Rules:**

- Elements must implement the `Delayed` interface.
- `take()` blocks until an element's delay expires.
- Does not permit `null` elements.

**Constraints and Limitations:**

- Elements must implement `Delayed`.
- Unbounded; `put()` never blocks.

### Annotated Code Example

```java
// File: DelayQueueDemo.java
import java.util.concurrent.*;

class DelayedTask implements Delayed {
    private final String name;
    private final long startTime;

    public DelayedTask(String name, long delayMs) {
        this.name = name;
        this.startTime = System.currentTimeMillis() + delayMs;
    }

    @Override
    public long getDelay(TimeUnit unit) {
        long remaining = startTime - System.currentTimeMillis();
        return unit.convert(remaining, TimeUnit.MILLISECONDS);
    }

    @Override
    public int compareTo(Delayed other) {
        return Long.compare(this.startTime, ((DelayedTask) other).startTime);
    }

    @Override
    public String toString() { return name; }
}

public class DelayQueueDemo {
    public static void main(String[] args) throws InterruptedException {
        DelayQueue<DelayedTask> queue = new DelayQueue<>();
        queue.put(new DelayedTask("Task 1", 3000));
        queue.put(new DelayedTask("Task 2", 1000));
        queue.put(new DelayedTask("Task 3", 2000));

        while (!queue.isEmpty()) {
            System.out.println("Taking: " + queue.take());
        }
    }
}
```

**Expected Output:**
```
Taking: Task 2
Taking: Task 3
Taking: Task 1
```

**Step-by-Step Explanation:**

1. Three tasks are added with different delays (1s, 2s, 3s).
2. `take()` blocks until each task's delay expires.
3. Tasks are retrieved in order of expiration: Task 2, Task 3, Task 1.

### Real-World Cases

- **Timeout Management:** Managing session timeouts or connection timeouts.
- **Scheduled Tasks:** Implementing a simple task scheduler.
- **Cache Expiration:** Removing expired items from a cache.

---

## 13. TransferQueue

### Definitions

**Core Definition:** `TransferQueue` is a `BlockingQueue` in which producers may wait for consumers to receive elements.

**Technical Definition:** `TransferQueue<E>` extends `BlockingQueue<E>`. A `TransferQueue` may be useful in message-passing applications where producers sometimes (using method `transfer(E)`) await receipt of elements by consumers invoking `take` or `poll`. The `transfer()` method transfers an element immediately if a consumer is waiting; otherwise, it blocks until a consumer receives the element.

**Beginner-Friendly Explanation:** `TransferQueue` is like a `SynchronousQueue` with capacity. A producer can either hand off an element directly (using `transfer()`) or enqueue it (using `put()`).

### Purposes

- To provide a queue where producers can wait for consumers
- To support handoff semantics with optional buffering
- To implement message-passing patterns

### Syntax Rules and Structure

```java
public interface TransferQueue<E> extends BlockingQueue<E> {
    boolean tryTransfer(E e);
    void transfer(E e) throws InterruptedException;
    boolean tryTransfer(E e, long timeout, TimeUnit unit) throws InterruptedException;
    boolean hasWaitingConsumer();
    int getWaitingConsumerCount();
}
```

**Syntax Rules:**

- `transfer()` blocks until the element is consumed.
- `tryTransfer()` returns immediately.
- `hasWaitingConsumer()` checks if a consumer is waiting.

**Constraints and Limitations:**

- Only implemented by `LinkedTransferQueue`.

### Annotated Code Example

```java
// File: TransferQueueDemo.java
import java.util.concurrent.*;

public class TransferQueueDemo {
    public static void main(String[] args) throws InterruptedException {
        TransferQueue<String> queue = new LinkedTransferQueue<>();

        Thread producer = new Thread(() -> {
            try {
                System.out.println("Producer: transferring");
                queue.transfer("Direct Handoff");
                System.out.println("Producer: transfer completed");
            } catch (InterruptedException e) { Thread.currentThread().interrupt(); }
        });

        Thread consumer = new Thread(() -> {
            try {
                Thread.sleep(1000);
                System.out.println("Consumer: taking");
                System.out.println("Consumer: got " + queue.take());
            } catch (InterruptedException e) { Thread.currentThread().interrupt(); }
        });

        producer.start();
        consumer.start();
        producer.join();
        consumer.join();
    }
}
```

**Expected Output:**
```
Producer: transferring
Consumer: taking
Consumer: got Direct Handoff
Producer: transfer completed
```

### Real-World Cases

- **Message Passing:** Handing off messages between threads.
- **Task Handoff:** Directly transferring tasks to worker threads.

---

## 14. LinkedTransferQueue

### Definitions

**Core Definition:** `LinkedTransferQueue` is an unbounded `TransferQueue` based on linked nodes, ordering elements FIFO with respect to any given producer.

**Technical Definition:** `LinkedTransferQueue<E>` extends `AbstractQueue<E>` and implements `TransferQueue<E>`, `Serializable`. It orders elements FIFO with respect to any given producer. The head of the queue is that element that has been on the queue the longest time for some producer. It provides higher throughput than `SynchronousQueue` and is suitable for message-passing applications.

**Beginner-Friendly Explanation:** `LinkedTransferQueue` is the implementation of `TransferQueue`. It combines the handoff capability of `SynchronousQueue` with the buffering capability of a regular queue.

### Purposes

- To provide a concrete implementation of `TransferQueue`
- To support both direct handoff and buffered queuing
- To offer high throughput for message-passing scenarios

### Syntax Rules and Structure

```java
public class LinkedTransferQueue<E> extends AbstractQueue<E>
        implements TransferQueue<E>, Serializable {
    public LinkedTransferQueue();
    public LinkedTransferQueue(Collection<? extends E> c);
}
```

**Syntax Rules:**

- Unbounded; `put()` never blocks.
- Does not permit `null` elements.

**Constraints and Limitations:**

- `size()` is not constant-time.

### Annotated Code Example

See the `TransferQueueDemo` in Section 13.

### Real-World Cases

- **High-Throughput Messaging:** Inter-thread communication with optional handoff.
- **Reactive Streams:** Implementing backpressure-aware pipelines.

---

## 15. ConcurrentLinkedQueue

### Definitions

**Core Definition:** `ConcurrentLinkedQueue` is an unbounded thread-safe queue based on linked nodes, ordering elements FIFO.

**Technical Definition:** `ConcurrentLinkedQueue<E>` extends `AbstractQueue<E>` and implements `Queue<E>`, `Serializable`. It is an unbounded thread-safe queue based on linked nodes, ordering elements FIFO. The head of the queue is the element that has been on the queue the longest time. It is an appropriate choice when many threads will share access to a common collection. It does not permit `null` elements. It employs an efficient non-blocking algorithm.

**Beginner-Friendly Explanation:** `ConcurrentLinkedQueue` is a thread-safe queue that uses lock-free algorithms. Multiple threads can add and remove elements simultaneously without blocking each other.

### Purposes

- To provide a thread-safe, lock-free FIFO queue
- To support high-concurrency scenarios with many threads
- To avoid the overhead of blocking queues when blocking is not needed

### Syntax Rules and Structure

```java
public class ConcurrentLinkedQueue<E> extends AbstractQueue<E>
        implements Queue<E>, Serializable {
    public ConcurrentLinkedQueue();
    public ConcurrentLinkedQueue(Collection<? extends E> c);
}
```

**Syntax Rules:**

- Unbounded.
- Does not permit `null` elements.
- Iterators are weakly consistent and never throw `ConcurrentModificationException`.
- `size()` is NOT constant-time; it requires traversal.

**Constraints and Limitations:**

- `size()` may be inaccurate if the queue is modified during traversal.
- Bulk operations are not guaranteed to be atomic.

### Annotated Code Example

```java
// File: ConcurrentLinkedQueueDemo.java
import java.util.concurrent.*;

public class ConcurrentLinkedQueueDemo {
    public static void main(String[] args) {
        ConcurrentLinkedQueue<Integer> queue = new ConcurrentLinkedQueue<>();

        // Multiple threads can safely add elements
        Thread t1 = new Thread(() -> {
            for (int i = 1; i <= 5; i++) queue.offer(i);
        });

        Thread t2 = new Thread(() -> {
            for (int i = 6; i <= 10; i++) queue.offer(i);
        });

        t1.start();
        t2.start();

        try { t1.join(); t2.join(); } catch (InterruptedException e) { }

        System.out.println("Queue size: " + queue.size()); // 10
        System.out.println("Poll order:");
        while (!queue.isEmpty()) {
            System.out.println(queue.poll());
        }
    }
}
```

**Expected Output:**
```
Queue size: 10
Poll order:
1
2
3
4
5
6
7
8
9
10
```

### Real-World Cases

- **Producer-Consumer:** Non-blocking producer-consumer queues.
- **Work Stealing:** Distributing tasks among worker threads.
- **Event Queues:** High-throughput event processing.

---

## 16. ConcurrentLinkedDeque

### Definitions

**Core Definition:** `ConcurrentLinkedDeque` is an unbounded concurrent deque based on linked nodes, supporting concurrent insertion, removal, and access operations at both ends.

**Technical Definition:** `ConcurrentLinkedDeque<E>` extends `AbstractCollection<E>` and implements `Deque<E>`, `Serializable`. Concurrent insertion, removal, and access operations execute safely across multiple threads. It is an appropriate choice when many threads will share access to a common collection. It does not permit `null` elements.

**Beginner-Friendly Explanation:** `ConcurrentLinkedDeque` is a thread-safe double-ended queue. It supports lock-free operations at both ends.

### Purposes

- To provide a thread-safe, lock-free deque
- To support concurrent operations at both ends
- To serve as a concurrent stack or queue

### Syntax Rules and Structure

```java
public class ConcurrentLinkedDeque<E> extends AbstractCollection<E>
        implements Deque<E>, Serializable {
    public ConcurrentLinkedDeque();
    public ConcurrentLinkedDeque(Collection<? extends E> c);
}
```

**Syntax Rules:**

- Unbounded.
- Does not permit `null` elements.
- Iterators are weakly consistent.

**Constraints and Limitations:**

- `size()` is not constant-time.
- Bulk operations are not atomic.

### Annotated Code Example

```java
// File: ConcurrentLinkedDequeDemo.java
import java.util.concurrent.*;

public class ConcurrentLinkedDequeDemo {
    public static void main(String[] args) {
        ConcurrentLinkedDeque<String> deque = new ConcurrentLinkedDeque<>();

        // Multiple threads can safely add at both ends
        Thread t1 = new Thread(() -> {
            for (int i = 1; i <= 5; i++) deque.addFirst("F" + i);
        });

        Thread t2 = new Thread(() -> {
            for (int i = 1; i <= 5; i++) deque.addLast("L" + i);
        });

        t1.start();
        t2.start();

        try { t1.join(); t2.join(); } catch (InterruptedException e) { }

        System.out.println("Deque: " + deque);
    }
}
```

**Expected Output:**
```
Deque: [F5, F4, F3, F2, F1, L1, L2, L3, L4, L5]
```

### Real-World Cases

- **Concurrent Work Stealing:** Deques are used in work-stealing algorithms.
- **Concurrent Stacks and Queues:** Flexible concurrent data structure.
- **Undo/Redo:** Concurrent undo and redo operations.

---

## 17. Queue Implementations Comparison

| Feature | ArrayDeque | LinkedList | PriorityQueue | ArrayBlockingQueue | LinkedBlockingQueue | PriorityBlockingQueue | SynchronousQueue | DelayQueue | ConcurrentLinkedQueue | ConcurrentLinkedDeque |
|---------|-----------|------------|---------------|-------------------|---------------------|----------------------|-----------------|-----------|----------------------|----------------------|
| **Interface** | Deque | Deque | Queue | BlockingQueue | BlockingQueue | BlockingQueue | BlockingQueue | BlockingQueue | Queue | Deque |
| **Thread-Safe** | No | No | No | Yes | Yes | Yes | Yes | Yes | Yes | Yes |
| **Bounded** | No | No | No | Yes | Optionally | No | No (0 capacity) | No | No | No |
| **Ordering** | FIFO/LIFO | FIFO/LIFO | Priority | FIFO | FIFO | Priority | Handoff | Delay | FIFO | FIFO/LIFO |
| **Null Elements** | No | Yes | No | No | No | No | No | No | No | No |
| **Blocking** | No | No | No | Yes | Yes | Yes | Yes | Yes | No | No |
| **Best For** | Default queue/stack | Legacy | Priority tasks | Bounded buffers | High throughput | Priority + blocking | Handoff | Delayed tasks | High-concurrency | Concurrent deque |

---

## 18. Queue Best Practices

### Definitions

**Core Definition:** Queue best practices are guidelines for choosing and using queue implementations effectively in Java applications.

**Technical Definition:** Best practices include choosing the right implementation for the use case, preferring `offer`/`poll`/`peek` over `add`/`remove`/`element`, avoiding `null` elements, and being aware of the performance characteristics of different implementations. For concurrent scenarios, prefer lock-free implementations (`ConcurrentLinkedQueue`) over blocking ones when blocking is not required.

**Beginner-Friendly Explanation:** Best practices help you write queue code that is fast, safe, and easy to maintain.

### Key Best Practices

| Practice | Description |
|----------|-------------|
| **Choose the right implementation** | Use `ArrayDeque` for general use; `PriorityQueue` for priority; `BlockingQueue` for producer-consumer; `ConcurrentLinkedQueue` for high-concurrency non-blocking. |
| **Prefer `offer`/`poll`/`peek`** | These return special values instead of throwing exceptions, avoiding try/catch clutter. |
| **Avoid `null` elements** | Most queues forbid `null`; use `Optional` or sentinel objects instead. |
| **Prefer `ArrayDeque` over `Stack` and `LinkedList`** | `ArrayDeque` is faster for both stack and queue operations. |
| **Use `BlockingQueue` for producer-consumer** | `put()` and `take()` handle blocking and waiting automatically. |
| **Use bounded queues for backpressure** | `ArrayBlockingQueue` provides natural backpressure in concurrent systems. |
| **Be aware of `size()` cost** | `ConcurrentLinkedQueue.size()` is O(n), not O(1). |
| **Use `PriorityBlockingQueue` for concurrent priority** | `PriorityQueue` is not thread-safe; use `PriorityBlockingQueue` instead. |
| **Consider fairness in blocking queues** | `ArrayBlockingQueue` with `fair=true` ensures FIFO order for blocked threads. |
| **Use `DelayQueue` for scheduled processing** | Elements become available only after their delay expires. |

### Annotated Code Example

```java
// File: QueueBestPractices.java
import java.util.*;
import java.util.concurrent.*;

public class QueueBestPractices {
    public static void main(String[] args) throws InterruptedException {
        // GOOD: Use ArrayDeque for general-purpose queue
        Deque<String> deque = new ArrayDeque<>();
        deque.offer("A");
        deque.offer("B");
        System.out.println("Deque: " + deque);

        // GOOD: Use PriorityQueue for priority-based processing
        PriorityQueue<Integer> pq = new PriorityQueue<>();
        pq.offer(5);
        pq.offer(1);
        pq.offer(3);
        System.out.println("Priority head: " + pq.peek()); // 1

        // GOOD: Use BlockingQueue for producer-consumer
        BlockingQueue<String> blockingQueue = new ArrayBlockingQueue<>(10);
        blockingQueue.put("Task");
        System.out.println("BlockingQueue take: " + blockingQueue.take());

        // GOOD: Use ConcurrentLinkedQueue for high-concurrency non-blocking
        ConcurrentLinkedQueue<Integer> clq = new ConcurrentLinkedQueue<>();
        clq.offer(1);
        clq.offer(2);
        System.out.println("ConcurrentLinkedQueue: " + clq);

        // GOOD: Prefer offer/poll over add/remove
        if (!deque.isEmpty()) {
            String head = deque.poll(); // Returns null if empty
            System.out.println("Polled: " + head);
        }
    }
}
```

**Expected Output:**
```
Deque: [A, B]
Priority head: 1
BlockingQueue take: Task
ConcurrentLinkedQueue: [1, 2]
Polled: A
```

**Step-by-Step Explanation:**

1. `ArrayDeque` is used for general-purpose queue operations.
2. `PriorityQueue` ensures the smallest element is always at the head.
3. `ArrayBlockingQueue` handles producer-consumer blocking.
4. `ConcurrentLinkedQueue` provides thread-safe, non-blocking operations.
5. `offer`/`poll` are preferred over `add`/`remove` for their tolerant failure behaviour.

### Real-World Cases

- **Task Scheduling:** `PriorityBlockingQueue` for priority-based task execution.
- **Web Servers:** `BlockingQueue` for request processing queues.
- **Event Processing:** `ConcurrentLinkedQueue` for high-throughput event handling.
- **Session Timeout:** `DelayQueue` for managing session expiration.

---

## References Links

- Queue (Java SE 27) — Oracle API Documentation - https://docs.oracle.com/en/java/javase/27/docs/api/java.base/java/util/Queue.html
- Deque (Java SE 11) — Oracle API Documentation - https://docs.oracle.com/javase/jp/11/docs/api/java.base/java/util/Deque.html
- PriorityQueue (Java SE 25) — Oracle API Documentation - https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/PriorityQueue.html
- ArrayDeque (Java SE 21) — Oracle API Documentation - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/ArrayDeque.html
- BlockingQueue (Java SE 11) — Oracle API Documentation - https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/BlockingQueue.html
- ArrayBlockingQueue (Java SE 21) — Oracle API Documentation - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ArrayBlockingQueue.html
- LinkedBlockingQueue (Java SE 21) — Oracle API Documentation - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/LinkedBlockingQueue.html
- PriorityBlockingQueue (Java SE 21) — Oracle API Documentation - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/PriorityBlockingQueue.html
- SynchronousQueue (Java SE 21) — Oracle API Documentation - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/SynchronousQueue.html
- DelayQueue (Java SE 21) — Oracle API Documentation - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/DelayQueue.html
- TransferQueue (Java SE 8) — Oracle API Documentation - https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/TransferQueue.html
- LinkedTransferQueue (Java SE 21) — Oracle API Documentation - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/LinkedTransferQueue.html
- ConcurrentLinkedQueue (Java SE 11) — Oracle API Documentation - https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/ConcurrentLinkedQueue.html
- ConcurrentLinkedDeque (Java SE 21) — Oracle API Documentation - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ConcurrentLinkedDeque.html
- Queue Implementations — The Java Tutorials (Oracle) - https://docs.oracle.com/javase/tutorial/collections/implementations/queue.html
- Java Queue Implementations: Guide to Queue, Deque, PriorityQueue, and BlockingQueue — Cleverence - https://www.cleverence.com/articles/oracle-documentation/queue-implementations-the-java-tutorials-4927/