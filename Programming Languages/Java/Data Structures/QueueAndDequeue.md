# Java Queue and Deque Framework: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition:** The Java Queue and Deque framework is a unified architecture within the Java Collections Framework for representing and manipulating collections of elements in which the position of each element relative to others is significant and governed by well-defined insertion and removal ordering policies.

**Technical Definition:** The Java Queue and Deque framework comprises the `Queue`, `Deque`, `BlockingQueue`, `BlockingDeque`, and `TransferQueue` interfaces together with their standard implementations in `java.util` and `java.util.concurrent`. These interfaces extend `Collection<E>` and specify a contract for element insertion, extraction, and inspection, with the head of the collection defined as the element that would be removed by a call to `remove()` or `poll()`. Implementations are classified by their ordering discipline (FIFO, LIFO, priority-based, or delay-based), their concurrency characteristics (thread-safe or not), and their capacity semantics (bounded or unbounded).

**Beginner-Friendly Explanation:** Imagine you are managing a queue of people waiting for a service. Some queues work on a first-come, first-served basis (FIFO). Others, like a stack of plates, work on a last-in, first-out basis (LIFO). Some queues let you place a person at either the front or the back (Deque). In a busy office, one person might have to wait for another to hand off a document (BlockingQueue). The Java Queue and Deque framework provides ready-made, well-tested building blocks for all these scenarios, so you do not have to implement the underlying data structures yourself.

### Key Characteristics

- The framework is built around five core interfaces: `Queue`, `Deque`, `BlockingQueue`, `BlockingDeque`, and `TransferQueue`. 
- The `Queue` interface provides two forms of each operation: one that throws an exception on failure and one that returns a special value (`null` or `false`). 
- The `Deque` interface extends `Queue` and adds methods for insertion, removal, and examination at both the head and the tail, and it defines equivalent methods for using a deque as a stack (LIFO) or as a queue (FIFO). 
- The `BlockingQueue` interface adds operations that wait for the queue to become non-empty when retrieving an element and wait for space to become available when storing an element, making it suitable for producer–consumer designs. 
- The `BlockingDeque` interface combines `Deque` and `BlockingQueue`. The `TransferQueue` interface adds a `transfer` operation in which a producer may wait for a consumer to receive an element. 
- Standard implementations include `ArrayDeque`, `LinkedList`, and `PriorityQueue` in `java.util`, and `ConcurrentLinkedQueue`, `ConcurrentLinkedDeque`, `LinkedBlockingQueue`, `LinkedBlockingDeque`, `ArrayBlockingQueue`, `PriorityBlockingQueue`, `DelayQueue`, `SynchronousQueue`, and `LinkedTransferQueue` in `java.util.concurrent`.

### Prerequisites

- Readers should be familiar with Java generics, the `Collection` interface, and basic exception handling. 
- Familiarity with the `Comparable` and `Comparator` interfaces is necessary for understanding `PriorityQueue` and `PriorityBlockingQueue`. 
- For concurrent implementations, a basic understanding of threads, the `Runnable` interface, and the `java.util.concurrent` package is assumed.

### Related Programming Areas with Explanation

- The Queue and Deque framework is related to data structures and algorithms (particularly the study of abstract data types such as queues, stacks, and priority queues)
- Concurrent programming (through the blocking and non-blocking concurrent implementations)
- Operating systems (through producer–consumer and scheduling patterns), 
- Software design patterns (such as the pipeline, worker pool, and task scheduling patterns). 
- It is also closely related to the Java Collections Framework as a whole, since all queue and deque types extend `Collection<E>`.

## Core Concepts and Key Features

### 1. The Queue Interface

#### Core Definitions

**Core Definition:** A collection designed for holding elements prior to processing, where elements are typically ordered in a first-in-first-out (FIFO) manner.

**Technical Definition:** `public interface Queue<E> extends Collection<E>`. Besides basic `Collection` operations, queues provide additional insertion, extraction, and inspection operations. Each of these methods exists in two forms: one throws an exception if the operation fails, and the other returns a special value (either `null` or `false`, depending on the operation).

**Beginner-Friendly Explanation:** Think of a queue at a ticket counter. The first person to join the line is the first person to be served. The `Queue` interface captures this behaviour, along with the ability to check who is at the front without removing them.

#### 1.1 Element Insertion

**Definitions**

- **Core Definition:** Adding an element to the tail of the queue.
- **Technical Definition:** `boolean add(E e)` inserts the specified element into this queue if it is possible to do so immediately without violating capacity restrictions, returning `true` upon success and throwing an `IllegalStateException` if no space is currently available. `boolean offer(E e)` inserts the specified element into this queue if it is possible to do so immediately without violating capacity restrictions.
- **Beginner-Friendly Explanation:** `add` is like forcing your way into a queue that has a strict limit—if it is full, you get an error. `offer` is more polite: it tries to join the queue and simply says “no” (returns `false`) if there is no room.

**Purposes**

- To insert an element into the queue at the tail.
- To provide a non-throwing alternative (`offer`) for capacity-restricted queues.
- To support the producer side of producer–consumer patterns.

**Syntax Structures and Rules**

Complete general syntaxes:

```java
boolean add(E e)
boolean offer(E e)
```

Component breakdown:

- `E`: the type of elements held in this queue.
- `e`: the element to be added.
- Return value: `true` if the element was added; `add` throws `IllegalStateException` if the queue is full; `offer` returns `false` if the queue is full.

Syntax rules: The `Queue` interface does not define blocking methods; those are defined in `BlockingQueue`. The `offer` method is designed for use when failure is a normal occurrence, such as in fixed-capacity queues.

Constraints and limitations: `add` may throw `IllegalStateException`; `offer` returns `false` if the element cannot be added. Neither method accepts `null` in implementations that prohibit null elements.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Basic `add` and `offer` with `ArrayDeque`**

```java
import java.util.ArrayDeque;
import java.util.Queue;

public class QueueInsertionExample {
    public static void main(String[] args) {
        // Step 1: Create a queue using ArrayDeque
        Queue<String> queue = new ArrayDeque<>();

        // Step 2: Use add() to insert elements
        queue.add("Alice");
        queue.add("Bob");
        System.out.println("Queue after add: " + queue);

        // Step 3: Use offer() to insert an element
        boolean offered = queue.offer("Charlie");
        System.out.println("offer returned: " + offered);
        System.out.println("Queue after offer: " + queue);
    }
}
```

Expected Output:

```
Queue after add: [Alice, Bob]
offer returned: true
Queue after offer: [Alice, Bob, Charlie]
```

**Why the code produces this result:** `ArrayDeque` is an unbounded deque, so `add` and `offer` always succeed unless memory is exhausted. The queue preserves insertion order, so “Alice” is at the head and “Charlie” is at the tail. `offer` returns `true` because there is no capacity restriction.

**Example 2: `offer` on a capacity-restricted queue**

```java
import java.util.Queue;
import java.util.concurrent.ArrayBlockingQueue;

public class QueueOfferFullExample {
    public static void main(String[] args) {
        // Step 1: Create a bounded queue with capacity 2
        Queue<Integer> queue = new ArrayBlockingQueue<>(2);

        // Step 2: Add two elements
        queue.offer(10);
        queue.offer(20);

        // Step 3: Attempt to add a third element
        boolean third = queue.offer(30);
        System.out.println("Third offer succeeded: " + third);
        System.out.println("Queue contents: " + queue);
    }
}
```

Expected Output:

```
Third offer succeeded: false
Queue contents: [10, 20]
```

**Why the code produces this result:** `ArrayBlockingQueue` is a bounded queue. Once it reaches capacity, `offer` returns `false` without modifying the queue. The element 30 is not added.

**Example 3: `add` on a full queue throws an exception**

```java
import java.util.Queue;
import java.util.concurrent.ArrayBlockingQueue;

public class QueueAddFullExample {
    public static void main(String[] args) {
        Queue<Integer> queue = new ArrayBlockingQueue<>(1);
        queue.add(1);
        try {
            queue.add(2); // This will throw IllegalStateException
        } catch (IllegalStateException e) {
            System.out.println("Caught: " + e.getClass().getSimpleName());
        }
    }
}
```

Expected Output:

```
Caught: IllegalStateException
```

**Why the code produces this result:** `ArrayBlockingQueue` is bounded, and `add` throws `IllegalStateException` when the queue is full.

**Real-World Cases with Explanation**

In a print spooler, documents are added to a queue in the order they are submitted. Using `offer` allows the spooler to gracefully reject new jobs when the queue is full rather than crashing. In a web server request queue, `add` may be used when the queue is unbounded, while `offer` is preferred when the queue has a fixed capacity and the server must shed load.

**References Links**

- Queue (Java Platform SE 8) – https://docs.oracle.com/javase/8/docs/api/java/util/Queue.html
- Queue (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Queue.html

#### 1.2 Element Removal

**Definitions**

- **Core Definition:** Removing and returning the head of the queue.
- **Technical Definition:** `E remove()` retrieves and removes the head of this queue, throwing a `NoSuchElementException` if this queue is empty. `E poll()` retrieves and removes the head of this queue, or returns `null` if this queue is empty.
- **Beginner-Friendly Explanation:** `remove` is like asking for the next person in line and getting an error if the line is empty. `poll` is more forgiving: it returns `null` if there is no one waiting.

**Purposes**

- To retrieve and remove the head of the queue.
- To provide a non-throwing alternative (`poll`) for empty queues.
- To support the consumer side of producer–consumer patterns.

**Syntax Structures and Rules**

Complete general syntaxes:

```java
E remove()
E poll()
```

Component breakdown:

- Return value: the head of the queue.
- `remove` throws `NoSuchElementException` if the queue is empty.
- `poll` returns `null` if the queue is empty.

Syntax rules: Exactly which element is removed depends on the queue's ordering policy. The `remove()` and `poll()` methods differ only in their behaviour when the queue is empty.

Constraints and limitations: `remove` may throw `NoSuchElementException`; `poll` returns `null` for an empty queue. Neither method blocks.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: `poll` on a non-empty queue**

```java
import java.util.ArrayDeque;
import java.util.Queue;

public class QueuePollExample {
    public static void main(String[] args) {
        Queue<String> queue = new ArrayDeque<>();
        queue.add("First");
        queue.add("Second");
        queue.add("Third");

        String head = queue.poll();
        System.out.println("Polled: " + head);
        System.out.println("Remaining: " + queue);
    }
}
```

Expected Output:

```
Polled: First
Remaining: [Second, Third]
```

**Why the code produces this result:** `ArrayDeque` maintains FIFO order when used as a queue. The head is “First”, which is removed and returned by `poll`.

**Example 2: `poll` on an empty queue returns `null`**

```java
import java.util.ArrayDeque;
import java.util.Queue;

public class QueuePollEmptyExample {
    public static void main(String[] args) {
        Queue<String> queue = new ArrayDeque<>();
        String result = queue.poll();
        System.out.println("Poll on empty queue: " + result);
    }
}
```

Expected Output:

```
Poll on empty queue: null
```

**Why the code produces this result:** `poll` returns `null` when the queue is empty, rather than throwing an exception.

**Example 3: `remove` on an empty queue throws an exception**

```java
import java.util.ArrayDeque;
import java.util.Queue;

public class QueueRemoveEmptyExample {
    public static void main(String[] args) {
        Queue<String> queue = new ArrayDeque<>();
        try {
            queue.remove();
        } catch (java.util.NoSuchElementException e) {
            System.out.println("Caught: " + e.getClass().getSimpleName());
        }
    }
}
```

Expected Output:

```
Caught: NoSuchElementException
```

**Why the code produces this result:** `remove` throws `NoSuchElementException` when the queue is empty.

**Real-World Cases with Explanation**

In a message processing system, `poll` is used in a loop to drain the queue without the risk of an exception when the queue becomes empty. In a task scheduler, `remove` might be used when the scheduler guarantees that at least one task is present.

**References Links**

- Queue (Java Platform SE 8) – https://docs.oracle.com/javase/8/docs/api/java/util/Queue.html
- Queue (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Queue.html

#### 1.3 Element Examination

**Definitions**

- **Core Definition:** Retrieving, but not removing, the head of the queue.
- **Technical Definition:** `E element()` retrieves, but does not remove, the head of this queue, throwing a `NoSuchElementException` if this queue is empty. `E peek()` retrieves, but does not remove, the head of this queue, or returns `null` if this queue is empty.
- **Beginner-Friendly Explanation:** `element` lets you look at the person at the front of the line without asking them to leave. If the line is empty, you get an error. `peek` does the same but returns `null` if the line is empty.

**Purposes**

- To inspect the head of the queue without modifying it.
- To provide a non-throwing alternative (`peek`) for empty queues.
- To support algorithms that need to examine the next element before deciding whether to remove it.

**Syntax Structures and Rules**

Complete general syntaxes:

```java
E element()
E peek()
```

Component breakdown:

- Return value: the head of the queue.
- `element` throws `NoSuchElementException` if the queue is empty.
- `peek` returns `null` if the queue is empty.

Syntax rules: These methods return, but do not remove, the head of the queue.

Constraints and limitations: `element` may throw `NoSuchElementException`; `peek` returns `null` for an empty queue.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: `peek` on a non-empty queue**

```java
import java.util.ArrayDeque;
import java.util.Queue;

public class QueuePeekExample {
    public static void main(String[] args) {
        Queue<String> queue = new ArrayDeque<>();
        queue.add("Alpha");
        queue.add("Beta");

        String head = queue.peek();
        System.out.println("Peeked: " + head);
        System.out.println("Queue unchanged: " + queue);
    }
}
```

Expected Output:

```
Peeked: Alpha
Queue unchanged: [Alpha, Beta]
```

**Why the code produces this result:** `peek` returns the head element without removing it, so the queue contents remain unchanged.

**Example 2: `peek` on an empty queue returns `null`**

```java
import java.util.ArrayDeque;
import java.util.Queue;

public class QueuePeekEmptyExample {
    public static void main(String[] args) {
        Queue<String> queue = new ArrayDeque<>();
        System.out.println("Peek on empty: " + queue.peek());
    }
}
```

Expected Output:

```
Peek on empty: null
```

**Example 3: `element` on an empty queue throws an exception**

```java
import java.util.ArrayDeque;
import java.util.Queue;

public class QueueElementEmptyExample {
    public static void main(String[] args) {
        Queue<String> queue = new ArrayDeque<>();
        try {
            queue.element();
        } catch (java.util.NoSuchElementException e) {
            System.out.println("Caught: " + e.getClass().getSimpleName());
        }
    }
}
```

Expected Output:

```
Caught: NoSuchElementException
```

**Real-World Cases with Explanation**

In a rate-limiting system, `peek` is used to check the next request's timestamp without removing it, so the system can decide whether to process it or delay. In a breadth-first search algorithm, `peek` is used to examine the next node before deciding whether to expand it.

**References Links**

- Queue (Java Platform SE 8) – https://docs.oracle.com/javase/8/docs/api/java/util/Queue.html
- Queue (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Queue.html

### 2. The Deque Interface

#### Core Definitions

**Core Definition:** A linear collection that supports element insertion and removal at both ends.

**Technical Definition:** `public interface Deque<E> extends Queue<E>, SequencedCollection<E>`. The name deque is short for “double ended queue” and is usually pronounced “deck”. Most `Deque` implementations place no fixed limits on the number of elements they may contain, but this interface supports capacity-restricted deques as well as those with no fixed size limit. This interface defines methods to access the elements at both ends of the deque.

**Beginner-Friendly Explanation:** A deque is like a queue where you can join or leave from either end. It can act as a queue (first-in, first-out) or as a stack (last-in, first-out). It is more flexible than a plain queue.

#### 2.1 Insertion at Both Ends

**Definitions**

- **Core Definition:** Adding an element at the front or the back of the deque.
- **Technical Definition:** The `Deque` interface provides six insertion methods: `addFirst(E e)`, `offerFirst(E e)`, `addLast(E e)`, `offerLast(E e)`, `push(E e)`, and the inherited `add(E e)` and `offer(E e)` methods which are equivalent to `addLast` and `offerLast` respectively. Methods that throw exceptions on failure are `addFirst`, `addLast`, and `push`; methods that return a special value are `offerFirst`, `offerLast`, `add`, and `offer`.
- **Beginner-Friendly Explanation:** You can add an element to the front using `addFirst` or `offerFirst`, or to the back using `addLast` or `offerLast`. `push` is equivalent to `addFirst` and is used when the deque is used as a stack.

**Purposes**

- To insert an element at the head (front) of the deque.
- To insert an element at the tail (back) of the deque.
- To provide non-throwing alternatives (`offerFirst`, `offerLast`) for capacity-restricted deques.
- To support both FIFO (queue) and LIFO (stack) usage patterns.

**Syntax Structures and Rules**

Complete general syntaxes:

```java
void addFirst(E e)
boolean offerFirst(E e)
void addLast(E e)
boolean offerLast(E e)
void push(E e)
```

Component breakdown:

- `e`: the element to be inserted.
- `addFirst` and `addLast` throw `IllegalStateException` if the deque is full and `NullPointerException` if the element is `null`.
- `offerFirst` and `offerLast` return `true` on success and `false` if the deque is full.
- `push` is equivalent to `addFirst`.

Syntax rules: When a deque is used as a queue, elements are added at the end of the deque and removed from the beginning. When used as a stack, elements are pushed and popped from the beginning.

Constraints and limitations: `addFirst`, `addLast`, and `push` throw exceptions on failure; `offerFirst`, `offerLast`, `add`, and `offer` return special values. Null elements are prohibited in most implementations.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Insertion at both ends with `ArrayDeque`**

```java
import java.util.ArrayDeque;
import java.util.Deque;

public class DequeInsertionExample {
    public static void main(String[] args) {
        // Step 1: Create an ArrayDeque
        Deque<String> deque = new ArrayDeque<>();

        // Step 2: Add elements at both ends
        deque.addFirst("Middle");
        deque.addLast("End");
        deque.addFirst("Start");

        System.out.println("Deque: " + deque);

        // Step 3: Use offerFirst and offerLast
        deque.offerFirst("NewStart");
        deque.offerLast("NewEnd");
        System.out.println("After offers: " + deque);
    }
}
```

Expected Output:

```
Deque: [Start, Middle, End]
After offers: [NewStart, Start, Middle, End, NewEnd]
```

**Why the code produces this result:** `addFirst` inserts at the head, `addLast` inserts at the tail. The deque maintains the relative order of insertions at each end. `offerFirst` and `offerLast` behave identically to `addFirst` and `addLast` in an unbounded deque.

**Example 2: Using `push` and `pop` as a stack**

```java
import java.util.ArrayDeque;
import java.util.Deque;

public class DequeStackExample {
    public static void main(String[] args) {
        Deque<Integer> stack = new ArrayDeque<>();
        stack.push(1);
        stack.push(2);
        stack.push(3);
        System.out.println("Stack: " + stack);
        System.out.println("Pop: " + stack.pop());
        System.out.println("After pop: " + stack);
    }
}
```

Expected Output:

```
Stack: [3, 2, 1]
Pop: 3
After pop: [2, 1]
```

**Why the code produces this result:** `push` is equivalent to `addFirst`, so the most recently pushed element (3) is at the head. `pop` is equivalent to `removeFirst`, so it removes and returns the head.

**Example 3: Capacity-restricted deque with `offerFirst`**

```java
import java.util.Deque;
import java.util.concurrent.LinkedBlockingDeque;

public class DequeOfferFullExample {
    public static void main(String[] args) {
        Deque<Integer> deque = new LinkedBlockingDeque<>(2);
        deque.offerFirst(1);
        deque.offerLast(2);
        boolean result = deque.offerFirst(3);
        System.out.println("offerFirst succeeded: " + result);
        System.out.println("Deque: " + deque);
    }
}
```

Expected Output:

```
offerFirst succeeded: false
Deque: [1, 2]
```

**Why the code produces this result:** `LinkedBlockingDeque` is optionally bounded. With capacity 2, the deque is full after two insertions, so `offerFirst` returns `false`.

**Real-World Cases with Explanation**

In a browser history implementation, a deque can be used so that new pages are added at the front and the “back” button removes from the front, while the “forward” button can examine the tail. In a work-stealing scheduler, each worker has a deque of tasks and can steal tasks from either end.

**References Links**

- Deque (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Deque.html
- Deque (Java Platform SE 8) – https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html

#### 2.2 Removal from Both Ends

**Definitions**

- **Core Definition:** Removing and returning an element from the front or the back of the deque.
- **Technical Definition:** The `Deque` interface provides `removeFirst()`, `pollFirst()`, `removeLast()`, `pollLast()`, and `pop()`. `removeFirst` and `removeLast` throw `NoSuchElementException` if the deque is empty; `pollFirst` and `pollLast` return `null` if the deque is empty. `pop` is equivalent to `removeFirst`.
- **Beginner-Friendly Explanation:** You can take an element from the front using `removeFirst` or `pollFirst`, or from the back using `removeLast` or `pollLast`. `pop` is equivalent to `removeFirst` and is used with stacks.

**Purposes**

- To remove and return an element from the head (front) of the deque.
- To remove and return an element from the tail (back) of the deque.
- To provide non-throwing alternatives (`pollFirst`, `pollLast`) for empty deques.
- To support both FIFO (queue) and LIFO (stack) removal patterns.

**Syntax Structures and Rules**

Complete general syntaxes:

```java
E removeFirst()
E pollFirst()
E removeLast()
E pollLast()
E pop()
```

Component breakdown:

- Return value: the element removed.
- `removeFirst` and `removeLast` throw `NoSuchElementException` if the deque is empty.
- `pollFirst` and `pollLast` return `null` if the deque is empty.
- `pop` is equivalent to `removeFirst`.

Syntax rules: When a deque is used as a queue, elements are removed from the beginning. When used as a stack, elements are popped from the beginning.

Constraints and limitations: `removeFirst`, `removeLast`, and `pop` throw exceptions on empty deques; `pollFirst` and `pollLast` return `null`.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Removal from both ends**

```java
import java.util.ArrayDeque;
import java.util.Deque;

public class DequeRemovalExample {
    public static void main(String[] args) {
        Deque<String> deque = new ArrayDeque<>();
        deque.addLast("A");
        deque.addLast("B");
        deque.addLast("C");

        System.out.println("Initial: " + deque);
        System.out.println("removeFirst: " + deque.removeFirst());
        System.out.println("removeLast: " + deque.removeLast());
        System.out.println("Remaining: " + deque);
    }
}
```

Expected Output:

```
Initial: [A, B, C]
removeFirst: A
removeLast: C
Remaining: [B]
```

**Why the code produces this result:** `removeFirst` removes the head (A), `removeLast` removes the tail (C), leaving only B.

**Example 2: `pollFirst` on an empty deque returns `null`**

```java
import java.util.ArrayDeque;
import java.util.Deque;

public class DequePollEmptyExample {
    public static void main(String[] args) {
        Deque<String> deque = new ArrayDeque<>();
        System.out.println("pollFirst: " + deque.pollFirst());
        System.out.println("pollLast: " + deque.pollLast());
    }
}
```

Expected Output:

```
pollFirst: null
pollLast: null
```

**Example 3: `pop` on an empty deque throws an exception**

```java
import java.util.ArrayDeque;
import java.util.Deque;

public class DequePopEmptyExample {
    public static void main(String[] args) {
        Deque<String> deque = new ArrayDeque<>();
        try {
            deque.pop();
        } catch (java.util.NoSuchElementException e) {
            System.out.println("Caught: " + e.getClass().getSimpleName());
        }
    }
}
```

Expected Output:

```
Caught: NoSuchElementException
```

**Real-World Cases with Explanation**

In an undo–redo system, the undo stack can be modelled as a deque where `pop` removes the most recent action, while the redo stack can be modelled as a deque where `pollFirst` retrieves the next action to redo. In a double-ended priority queue, elements can be removed from either the highest or lowest priority end.

**References Links**

- Deque (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Deque.html
- Deque (Java Platform SE 8) – https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html

#### 2.3 Examination at Both Ends

**Definitions**

- **Core Definition:** Retrieving, but not removing, an element from the front or the back of the deque.
- **Technical Definition:** The `Deque` interface provides `getFirst()`, `peekFirst()`, `getLast()`, and `peekLast()`. `getFirst` and `getLast` throw `NoSuchElementException` if the deque is empty; `peekFirst` and `peekLast` return `null` if the deque is empty.
- **Beginner-Friendly Explanation:** You can look at the first or last element without removing it. `getFirst` and `getLast` throw an error if the deque is empty; `peekFirst` and `peekLast` return `null`.

**Purposes**

- To inspect the element at the head (front) of the deque without modifying it.
- To inspect the element at the tail (back) of the deque without modifying it.
- To provide non-throwing alternatives (`peekFirst`, `peekLast`) for empty deques.
- To support algorithms that need to examine both ends before deciding which element to remove.

**Syntax Structures and Rules**

Complete general syntaxes:

```java
E getFirst()
E peekFirst()
E getLast()
E peekLast()
```

Component breakdown:

- Return value: the element examined.
- `getFirst` and `getLast` throw `NoSuchElementException` if the deque is empty.
- `peekFirst` and `peekLast` return `null` if the deque is empty.

Syntax rules: These methods return, but do not remove, the element at the specified end.

Constraints and limitations: `getFirst` and `getLast` may throw `NoSuchElementException`; `peekFirst` and `peekLast` return `null` for an empty deque.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Examination at both ends**

```java
import java.util.ArrayDeque;
import java.util.Deque;

public class DequeExamineExample {
    public static void main(String[] args) {
        Deque<String> deque = new ArrayDeque<>();
        deque.addLast("First");
        deque.addLast("Middle");
        deque.addLast("Last");

        System.out.println("First element: " + deque.getFirst());
        System.out.println("Last element: " + deque.getLast());
        System.out.println("Deque unchanged: " + deque);
    }
}
```

Expected Output:

```
First element: First
Last element: Last
Deque unchanged: [First, Middle, Last]
```

**Why the code produces this result:** `getFirst` returns the head, `getLast` returns the tail, and neither modifies the deque.

**Example 2: `peekFirst` on an empty deque returns `null`**

```java
import java.util.ArrayDeque;
import java.util.Deque;

public class DequePeekEmptyExample {
    public static void main(String[] args) {
        Deque<String> deque = new ArrayDeque<>();
        System.out.println("peekFirst: " + deque.peekFirst());
        System.out.println("peekLast: " + deque.peekLast());
    }
}
```

Expected Output:

```
peekFirst: null
peekLast: null
```

**Example 3: `getFirst` on an empty deque throws an exception**

```java
import java.util.ArrayDeque;
import java.util.Deque;

public class DequeGetEmptyExample {
    public static void main(String[] args) {
        Deque<String> deque = new ArrayDeque<>();
        try {
            deque.getFirst();
        } catch (java.util.NoSuchElementException e) {
            System.out.println("Caught: " + e.getClass().getSimpleName());
        }
    }
}
```

Expected Output:

```
Caught: NoSuchElementException
```

**Real-World Cases with Explanation**

In a sliding window maximum algorithm, a deque is used to maintain a window of elements, and `peekFirst` and `peekLast` are used to examine the ends of the window without removing elements. In a palindrome checker, `peekFirst` and `peekLast` are used to compare characters from both ends.

**References Links**

- Deque (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Deque.html
- Deque (Java Platform SE 8) – https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html

### 3. The BlockingQueue Interface

#### Core Definitions

**Core Definition:** A queue that additionally supports operations that wait for the queue to become non-empty when retrieving an element, and wait for space to become available in the queue when storing an element.

**Technical Definition:** `public interface BlockingQueue<E> extends Queue<E>`. `BlockingQueue` methods come in four forms: one throws an exception, the second returns a special value, the third blocks the current thread indefinitely until the operation can succeed, and the fourth blocks for only a given maximum time limit before giving up. These methods are summarized in the following table: `add(e)` / `offer(e)` / `put(e)` / `offer(e, time, unit)` for insertion; `remove()` / `poll()` / `take()` / `poll(time, unit)` for removal; and `element()` / `peek()` for examination.

**Beginner-Friendly Explanation:** A blocking queue is like a queue at a bank where the teller waits until a customer arrives before serving, and a customer waits until a teller is free before being served. This makes it perfect for producer–consumer scenarios where one thread produces data and another consumes it.

#### 3.1 Blocking Insertion (`put`)

**Definitions**

- **Core Definition:** Inserting an element into the queue, waiting if necessary for space to become available.
- **Technical Definition:** `void put(E e) throws InterruptedException` inserts the specified element into this queue, waiting if necessary for space to become available. The method blocks the current thread indefinitely until the operation can succeed, and throws `InterruptedException` if interrupted while waiting.
- **Beginner-Friendly Explanation:** `put` is like trying to place a letter in a full mailbox: you wait until someone removes a letter to make room. If you are interrupted while waiting, you get an `InterruptedException`.

**Purposes**

- To insert an element into a possibly full queue, blocking until space is available.
- To support the producer side of producer–consumer patterns without busy-waiting.
- To provide a thread-safe, blocking alternative to `add` and `offer`.

**Syntax Structures and Rules**

Complete general syntaxes:

```java
void put(E e) throws InterruptedException
```

Component breakdown:

- `e`: the element to be inserted.
- Throws `InterruptedException` if interrupted while waiting.
- Throws `NullPointerException` if the element is `null`.
- Blocks indefinitely until space is available.

Syntax rules: `BlockingQueue` implementations are thread-safe. All queuing methods achieve their effects atomically using internal locks or other forms of concurrency control.

Constraints and limitations: `put` may block indefinitely. `BlockingQueue` does not accept `null` elements; implementations throw `NullPointerException` on attempts to add, put, or offer a `null`.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Producer–consumer with `put` and `take`**

```java
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;

public class BlockingQueueExample {
    public static void main(String[] args) throws InterruptedException {
        // Step 1: Create a bounded blocking queue with capacity 2
        BlockingQueue<Integer> queue = new ArrayBlockingQueue<>(2);

        // Step 2: Producer thread puts elements into the queue
        Thread producer = new Thread(() -> {
            try {
                queue.put(1);
                System.out.println("Produced 1");
                queue.put(2);
                System.out.println("Produced 2");
                // This put will block until consumer takes an element
                queue.put(3);
                System.out.println("Produced 3");
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });

        // Step 3: Consumer thread takes elements from the queue
        Thread consumer = new Thread(() -> {
            try {
                Thread.sleep(1000); // Simulate delay
                System.out.println("Consumed " + queue.take());
                System.out.println("Consumed " + queue.take());
                System.out.println("Consumed " + queue.take());
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });

        producer.start();
        consumer.start();
        producer.join();
        consumer.join();
    }
}
```

Expected Output (order may vary due to thread scheduling):

```
Produced 1
Produced 2
Consumed 1
Produced 3
Consumed 2
Consumed 3
```

**Why the code produces this result:** The producer can put two elements immediately (capacity 2). The third `put` blocks because the queue is full. The consumer takes elements, which unblocks the producer. The exact interleaving depends on thread scheduling.

**Example 2: `put` with timeout**

```java
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.TimeUnit;

public class BlockingQueueTimeoutExample {
    public static void main(String[] args) throws InterruptedException {
        BlockingQueue<Integer> queue = new ArrayBlockingQueue<>(1);
        queue.put(1);
        // This offer will time out after 500 milliseconds
        boolean result = queue.offer(2, 500, TimeUnit.MILLISECONDS);
        System.out.println("Offer timed out: " + !result);
    }
}
```

Expected Output:

```
Offer timed out: true
```

**Why the code produces this result:** The queue is full, and no consumer is taking elements. The timed `offer` waits for 500 ms and then returns `false`.

**Real-World Cases with Explanation**

In a web server, a `BlockingQueue` is used to hold incoming requests. Worker threads call `take` to retrieve requests; if the queue is empty, they block until a request arrives. In a logging system, a producer thread `put`s log messages into a queue, and a consumer thread `take`s them and writes them to disk.

**References Links**

- BlockingQueue (Java SE 11) – https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/BlockingQueue.html
- BlockingQueue (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/BlockingQueue.html

#### 3.2 Blocking Removal (`take`)

**Definitions**

- **Core Definition:** Removing and returning the head of the queue, waiting if necessary for an element to become available.
- **Technical Definition:** `E take() throws InterruptedException` retrieves and removes the head of this queue, waiting if necessary until an element becomes available. The method blocks the current thread indefinitely until an element is available, and throws `InterruptedException` if interrupted while waiting.
- **Beginner-Friendly Explanation:** `take` is like waiting at a conveyor belt for a package to arrive. If the belt is empty, you wait. If you are interrupted while waiting, you get an `InterruptedException`.

**Purposes**

- To retrieve and remove the head of a possibly empty queue, blocking until an element is available.
- To support the consumer side of producer–consumer patterns without busy-waiting.
- To provide a thread-safe, blocking alternative to `remove` and `poll`.

**Syntax Structures and Rules**

Complete general syntaxes:

```java
E take() throws InterruptedException
```

Component breakdown:

- Return value: the head of the queue.
- Throws `InterruptedException` if interrupted while waiting.
- Blocks indefinitely until an element is available.

Syntax rules: `take` waits for the queue to become non-empty when retrieving an element.

Constraints and limitations: `take` may block indefinitely.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: `take` blocks until an element is available**

```java
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;

public class BlockingTakeExample {
    public static void main(String[] args) throws InterruptedException {
        BlockingQueue<String> queue = new ArrayBlockingQueue<>(10);

        Thread consumer = new Thread(() -> {
            try {
                System.out.println("Waiting for element...");
                String item = queue.take();
                System.out.println("Took: " + item);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });

        consumer.start();
        Thread.sleep(1000); // Consumer is now blocked
        queue.put("Hello");
        consumer.join();
    }
}
```

Expected Output:

```
Waiting for element...
Took: Hello
```

**Why the code produces this result:** The consumer calls `take` on an empty queue and blocks. After one second, the main thread puts “Hello” into the queue, which unblocks the consumer.

**Example 2: `take` with interrupt**

```java
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;

public class BlockingTakeInterruptExample {
    public static void main(String[] args) throws InterruptedException {
        BlockingQueue<String> queue = new ArrayBlockingQueue<>(10);
        Thread consumer = new Thread(() -> {
            try {
                queue.take();
            } catch (InterruptedException e) {
                System.out.println("Consumer interrupted");
            }
        });
        consumer.start();
        Thread.sleep(500);
        consumer.interrupt();
        consumer.join();
    }
}
```

Expected Output:

```
Consumer interrupted
```

**Real-World Cases with Explanation**

In a message broker, consumer threads call `take` on a `BlockingQueue`. When a message is published, the broker `put`s it into the queue, and a waiting consumer is unblocked and processes the message. In a thread pool, worker threads call `take` on a task queue, blocking until a task is submitted.

**References Links**

- BlockingQueue (Java SE 11) – https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/BlockingQueue.html
- BlockingQueue (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/BlockingQueue.html

### 4. Standard Implementations

#### 4.1 ArrayDeque

**Definitions**

- **Core Definition:** A resizable-array implementation of the `Deque` interface.
- **Technical Definition:** `public class ArrayDeque<E> extends AbstractCollection<E> implements Deque<E>, Cloneable, Serializable`. Array deques have no capacity restrictions; they grow as necessary to support usage. They are not thread-safe; in the absence of external synchronization, they do not support concurrent access by multiple threads. Null elements are prohibited. This class is likely to be faster than `Stack` when used as a stack, and faster than `LinkedList` when used as a queue.
- **Beginner-Friendly Explanation:** `ArrayDeque` is a fast, array-backed deque. It is not thread-safe, but it is usually the best choice for single-threaded queue or stack operations.

**Purposes**

- To provide a fast, array-backed implementation of the `Deque` interface.
- To serve as a superior alternative to `Stack` and `LinkedList` for queue and stack operations.
- To avoid the overhead of linked nodes.

**Syntax Structures and Rules**

Complete general syntaxes:

```java
ArrayDeque()
ArrayDeque(int numElements)
ArrayDeque(Collection<? extends E> c)
```

Component breakdown:

- `numElements`: the initial capacity sufficient to hold the specified number of elements.
- `c`: the collection whose elements are to be placed into the deque.

Syntax rules: `ArrayDeque` prohibits `null` elements. Most operations run in amortized constant time; exceptions include `remove`, `removeFirstOccurrence`, `removeLastOccurrence`, `contains`, `iterator.remove`, and bulk operations, which run in linear time.

Constraints and limitations: Not thread-safe. Null elements are prohibited.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Basic `ArrayDeque` usage**

```java
import java.util.ArrayDeque;
import java.util.Deque;

public class ArrayDequeExample {
    public static void main(String[] args) {
        Deque<Integer> deque = new ArrayDeque<>();
        deque.addFirst(1);
        deque.addLast(2);
        deque.addFirst(0);
        System.out.println("Deque: " + deque);
        System.out.println("First: " + deque.getFirst());
        System.out.println("Last: " + deque.getLast());
    }
}
```

Expected Output:

```
Deque: [0, 1, 2]
First: 0
Last: 2
```

**Example 2: Using `ArrayDeque` as a stack**

```java
import java.util.ArrayDeque;
import java.util.Deque;

public class ArrayDequeStackExample {
    public static void main(String[] args) {
        Deque<String> stack = new ArrayDeque<>();
        stack.push("a");
        stack.push("b");
        stack.push("c");
        System.out.println("Stack: " + stack);
        System.out.println("Pop: " + stack.pop());
        System.out.println("After pop: " + stack);
    }
}
```

Expected Output:

```
Stack: [c, b, a]
Pop: c
After pop: [b, a]
```

**Real-World Cases with Explanation**

`ArrayDeque` is used in algorithms such as breadth-first search (as a queue), depth-first search (as a stack), and sliding window problems (as a deque). It is also used in Java's own `ArrayDeque`-based `Spliterator` implementations.

**References Links**

- ArrayDeque (Java SE 26) – https://docs.oracle.com/en/java/javase/26/docs/api/java.base/java/util/ArrayDeque.html
- ArrayDeque (Java Platform SE 8) – https://docs.oracle.com/javase/8/docs/api/java/util/ArrayDeque.html

#### 4.2 LinkedList

**Definitions**

- **Core Definition:** A doubly-linked list implementation of the `List` and `Deque` interfaces.
- **Technical Definition:** `public class LinkedList<E> extends AbstractSequentialList<E> implements List<E>, Deque<E>, Cloneable, Serializable`. It implements all optional list operations and permits all elements (including `null`).
- **Beginner-Friendly Explanation:** `LinkedList` is a linked-list implementation that can be used as a list, a queue, or a deque. It allows `null` elements, but `ArrayDeque` is generally preferred for pure queue/stack operations.

**Purposes**

- To provide a linked-list implementation of the `Deque` interface.
- To support `null` elements in a deque.
- To serve as a list implementation that can also be used as a queue or stack.

**Syntax Structures and Rules**

Complete general syntaxes:

```java
LinkedList()
LinkedList(Collection<? extends E> c)
```

Component breakdown:

- `c`: the collection whose elements are to be placed into the list.

Syntax rules: `LinkedList` implements both `List` and `Deque`. It permits `null` elements.

Constraints and limitations: Not thread-safe. Generally slower than `ArrayDeque` for pure queue/stack operations due to node allocation overhead.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: `LinkedList` as a queue**

```java
import java.util.LinkedList;
import java.util.Queue;

public class LinkedListQueueExample {
    public static void main(String[] args) {
        Queue<String> queue = new LinkedList<>();
        queue.offer("First");
        queue.offer("Second");
        queue.offer("Third");
        System.out.println("Queue: " + queue);
        System.out.println("Poll: " + queue.poll());
        System.out.println("After poll: " + queue);
    }
}
```

Expected Output:

```
Queue: [First, Second, Third]
Poll: First
After poll: [Second, Third]
```

**Example 2: `LinkedList` as a deque**

```java
import java.util.LinkedList;
import java.util.Deque;

public class LinkedListDequeExample {
    public static void main(String[] args) {
        Deque<Integer> deque = new LinkedList<>();
        deque.addFirst(1);
        deque.addLast(2);
        deque.addFirst(0);
        System.out.println("Deque: " + deque);
    }
}
```

Expected Output:

```
Deque: [0, 1, 2]
```

**Real-World Cases with Explanation**

`LinkedList` is used when a deque needs to allow `null` elements or when list operations are also required. In practice, `ArrayDeque` is preferred for queue and stack operations due to better performance.

**References Links**

- LinkedList (Java Platform SE 8) – https://docs.oracle.com/javase/8/docs/api/java/util/LinkedList.html
- LinkedList (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/LinkedList.html

#### 4.3 PriorityQueue

**Definitions**

- **Core Definition:** An unbounded priority queue based on a priority heap.
- **Technical Definition:** `public class PriorityQueue<E> extends AbstractQueue<E> implements Serializable`. The elements of the priority queue are ordered according to their natural ordering, or by a `Comparator` provided at queue construction time. A priority queue does not permit `null` elements. A priority queue relying on natural ordering also does not permit insertion of non-comparable objects.
- **Beginner-Friendly Explanation:** `PriorityQueue` is a queue that serves elements based on their priority, not their arrival order. The smallest element (according to natural ordering or a comparator) is always at the head.

**Purposes**

- To provide a queue that orders elements by priority rather than insertion order.
- To support algorithms that repeatedly need the smallest (or largest) element.
- To serve as a building block for priority-based scheduling and graph algorithms.

**Syntax Structures and Rules**

Complete general syntaxes:

```java
PriorityQueue()
PriorityQueue(int initialCapacity)
PriorityQueue(Comparator<? super E> comparator)
PriorityQueue(int initialCapacity, Comparator<? super E> comparator)
PriorityQueue(Collection<? extends E> c)
PriorityQueue(PriorityQueue<? extends E> c)
PriorityQueue(SortedSet<? extends E> c)
```

Component breakdown:

- `initialCapacity`: the initial capacity for the internal array.
- `comparator`: the comparator used to order elements.
- `c`: the collection whose elements are to be placed into the priority queue.

Syntax rules: The head of the queue is the least element with respect to the specified ordering. If multiple elements are tied for least value, the head is one of those elements—ties are broken arbitrarily. The `iterator()` is not guaranteed to traverse elements in any particular order.

Constraints and limitations: Not synchronized. Multiple threads should not access a `PriorityQueue` instance concurrently if any thread modifies it. Use `PriorityBlockingQueue` for concurrent use.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Natural ordering**

```java
import java.util.PriorityQueue;
import java.util.Queue;

public class PriorityQueueNaturalExample {
    public static void main(String[] args) {
        Queue<Integer> pq = new PriorityQueue<>();
        pq.add(5);
        pq.add(1);
        pq.add(3);
        System.out.println("Head: " + pq.peek()); // 1
        System.out.println("Poll: " + pq.poll()); // 1
        System.out.println("Poll: " + pq.poll()); // 3
        System.out.println("Poll: " + pq.poll()); // 5
    }
}
```

Expected Output:

```
Head: 1
Poll: 1
Poll: 3
Poll: 5
```

**Why the code produces this result:** `PriorityQueue` orders elements by natural ordering. The smallest element (1) is at the head. `poll` removes elements in ascending order.

**Example 2: Custom comparator (reverse order)**

```java
import java.util.Comparator;
import java.util.PriorityQueue;
import java.util.Queue;

public class PriorityQueueComparatorExample {
    public static void main(String[] args) {
        Queue<Integer> pq = new PriorityQueue<>(Comparator.reverseOrder());
        pq.add(5);
        pq.add(1);
        pq.add(3);
        System.out.println("Poll: " + pq.poll()); // 5
        System.out.println("Poll: " + pq.poll()); // 3
        System.out.println("Poll: " + pq.poll()); // 1
    }
}
```

Expected Output:

```
Poll: 5
Poll: 3
Poll: 1
```

**Real-World Cases with Explanation**

`PriorityQueue` is used in Dijkstra's shortest path algorithm, Huffman coding, task scheduling (where tasks with higher priority are executed first), and event-driven simulation.

**References Links**

- PriorityQueue (Java SE 25) – https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/PriorityQueue.html
- PriorityQueue (Java Platform SE 8) – https://docs.oracle.com/javase/8/docs/api/java/util/PriorityQueue.html

### 5. Concurrent Implementations

#### 5.1 ConcurrentLinkedQueue

**Definitions**

- **Core Definition:** An unbounded thread-safe queue based on linked nodes.
- **Technical Definition:** `public class ConcurrentLinkedQueue<E> extends AbstractQueue<E> implements Queue<E>, Serializable`. This queue orders elements FIFO. The head of the queue is that element that has been on the queue the longest time. The tail is that element that has been on the queue the shortest time. New elements are inserted at the tail, and retrieval operations obtain elements at the head.
- **Beginner-Friendly Explanation:** `ConcurrentLinkedQueue` is a thread-safe queue that uses a lock-free algorithm. It is suitable for scenarios where many threads share a common collection.

**Purposes**

- To provide a thread-safe, non-blocking FIFO queue.
- To support high-throughput concurrent producer–consumer scenarios without locks.
- To avoid the overhead of blocking synchronisation.

**Syntax Structures and Rules**

Complete general syntaxes:

```java
ConcurrentLinkedQueue()
ConcurrentLinkedQueue(Collection<? extends E> c)
```

Component breakdown:

- `c`: the collection whose elements are to be placed into the queue.

Syntax rules: This implementation employs an efficient non-blocking algorithm. Iterators are weakly consistent and do not throw `ConcurrentModificationException`.

Constraints and limitations: The `size` method is not a constant-time operation. `null` elements are not permitted.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Basic concurrent queue usage**

```java
import java.util.concurrent.ConcurrentLinkedQueue;

public class ConcurrentLinkedQueueExample {
    public static void main(String[] args) {
        ConcurrentLinkedQueue<String> queue = new ConcurrentLinkedQueue<>();
        queue.offer("A");
        queue.offer("B");
        queue.offer("C");
        System.out.println("Queue: " + queue);
        System.out.println("Poll: " + queue.poll());
        System.out.println("Peek: " + queue.peek());
        System.out.println("Queue after: " + queue);
    }
}
```

Expected Output:

```
Queue: [A, B, C]
Poll: A
Peek: B
Queue after: [B, C]
```

**Example 2: Multiple threads offering to the queue**

```java
import java.util.concurrent.ConcurrentLinkedQueue;

public class ConcurrentLinkedQueueThreadsExample {
    public static void main(String[] args) throws InterruptedException {
        ConcurrentLinkedQueue<Integer> queue = new ConcurrentLinkedQueue<>();
        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 5; i++) queue.offer(i);
        });
        Thread t2 = new Thread(() -> {
            for (int i = 5; i < 10; i++) queue.offer(i);
        });
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println("Queue size: " + queue.size());
        System.out.println("Queue contents: " + queue);
    }
}
```

Expected Output (order may vary):

```
Queue size: 10
Queue contents: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

**Real-World Cases with Explanation**

`ConcurrentLinkedQueue` is used in event processing systems where multiple producers generate events and multiple consumers process them. It is also used in thread pools and task queues that require high throughput without blocking.

**References Links**

- ConcurrentLinkedQueue (Java SE 11) – https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/ConcurrentLinkedQueue.html
- ConcurrentLinkedQueue (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ConcurrentLinkedQueue.html

#### 5.2 LinkedBlockingQueue

**Definitions**

- **Core Definition:** An optionally-bounded blocking queue based on linked nodes.
- **Technical Definition:** `public class LinkedBlockingQueue<E> extends AbstractQueue<E> implements BlockingQueue<E>, Serializable`. This queue orders elements FIFO. The head is the element that has been on the queue the longest time. New elements are inserted at the tail. Linked queues typically have higher throughput than array-based queues but less predictable performance.
- **Beginner-Friendly Explanation:** `LinkedBlockingQueue` is a thread-safe blocking queue that uses a linked-list structure. It can be bounded or unbounded, and it is suitable for producer–consumer scenarios.

**Purposes**

- To provide a thread-safe, optionally bounded blocking queue.
- To support high-throughput producer–consumer scenarios with linked nodes.
- To allow flexible capacity management.

**Syntax Structures and Rules**

Complete general syntaxes:

```java
LinkedBlockingQueue()
LinkedBlockingQueue(int capacity)
LinkedBlockingQueue(Collection<? extends E> c)
```

Component breakdown:

- `capacity`: the maximum number of elements the queue can hold.
- `c`: the collection whose elements are to be placed into the queue.

Syntax rules: The optional capacity bound constructor argument serves as a way to prevent excessive queue expansion. The capacity, if unspecified, is equal to `Integer.MAX_VALUE`.

Constraints and limitations: `null` elements are not permitted.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Bounded `LinkedBlockingQueue`**

```java
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.BlockingQueue;

public class LinkedBlockingQueueExample {
    public static void main(String[] args) throws InterruptedException {
        BlockingQueue<Integer> queue = new LinkedBlockingQueue<>(3);
        queue.put(1);
        queue.put(2);
        queue.put(3);
        System.out.println("Queue: " + queue);
        System.out.println("Take: " + queue.take());
        System.out.println("Queue after take: " + queue);
    }
}
```

Expected Output:

```
Queue: [1, 2, 3]
Take: 1
Queue after take: [2, 3]
```

**Example 2: Producer–consumer with `LinkedBlockingQueue`**

```java
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.BlockingQueue;

public class LinkedBlockingQueuePCExample {
    public static void main(String[] args) throws InterruptedException {
        BlockingQueue<String> queue = new LinkedBlockingQueue<>(2);
        Thread producer = new Thread(() -> {
            try {
                queue.put("Message1");
                System.out.println("Produced Message1");
                queue.put("Message2");
                System.out.println("Produced Message2");
                queue.put("Message3"); // Blocks until consumer takes
                System.out.println("Produced Message3");
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });
        Thread consumer = new Thread(() -> {
            try {
                Thread.sleep(1000);
                System.out.println("Consumed " + queue.take());
                System.out.println("Consumed " + queue.take());
                System.out.println("Consumed " + queue.take());
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });
        producer.start();
        consumer.start();
        producer.join();
        consumer.join();
    }
}
```

Expected Output (order may vary):

```
Produced Message1
Produced Message2
Consumed Message1
Produced Message3
Consumed Message2
Consumed Message3
```

**Real-World Cases with Explanation**

`LinkedBlockingQueue` is used in thread pool implementations (such as `ThreadPoolExecutor`), message brokers, and any scenario where producers and consumers operate at different rates and need to be decoupled.

**References Links**

- LinkedBlockingQueue (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/LinkedBlockingQueue.html
- LinkedBlockingQueue (Java Platform SE 8) – https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/LinkedBlockingQueue.html

#### 5.3 ArrayBlockingQueue

**Definitions**

- **Core Definition:** A bounded blocking queue backed by an array.
- **Technical Definition:** `public class ArrayBlockingQueue<E> extends AbstractQueue<E> implements BlockingQueue<E>, Serializable`. This queue orders elements FIFO. Once created, the capacity cannot be changed. Attempts to `put` an element into a full queue will result in the operation blocking; attempts to `take` an element from an empty queue will similarly block.
- **Beginner-Friendly Explanation:** `ArrayBlockingQueue` is a fixed-size blocking queue backed by an array. It is a classic bounded buffer and supports an optional fairness policy.

**Purposes**

- To provide a bounded, array-backed blocking queue.
- To serve as a classic bounded buffer for producer–consumer scenarios.
- To support optional fairness for waiting producer and consumer threads.

**Syntax Structures and Rules**

Complete general syntaxes:

```java
ArrayBlockingQueue(int capacity)
ArrayBlockingQueue(int capacity, boolean fair)
ArrayBlockingQueue(int capacity, boolean fair, Collection<? extends E> c)
```

Component breakdown:

- `capacity`: the fixed capacity of the queue.
- `fair`: if `true`, grants threads access in FIFO order.
- `c`: the collection whose elements are to be placed into the queue.

Syntax rules: The capacity cannot be changed after creation. Fairness generally decreases throughput but reduces variability and avoids starvation.

Constraints and limitations: `null` elements are not permitted.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Bounded `ArrayBlockingQueue`**

```java
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;

public class ArrayBlockingQueueExample {
    public static void main(String[] args) throws InterruptedException {
        BlockingQueue<String> queue = new ArrayBlockingQueue<>(2);
        queue.put("A");
        queue.put("B");
        System.out.println("Queue: " + queue);
        System.out.println("Take: " + queue.take());
        System.out.println("Queue after: " + queue);
    }
}
```

Expected Output:

```
Queue: [A, B]
Take: A
Queue after: [B]
```

**Example 2: Fair `ArrayBlockingQueue`**

```java
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;

public class ArrayBlockingQueueFairExample {
    public static void main(String[] args) throws InterruptedException {
        BlockingQueue<Integer> queue = new ArrayBlockingQueue<>(1, true);
        Thread t1 = new Thread(() -> {
            try {
                queue.put(1);
                System.out.println("Thread 1 produced 1");
            } catch (InterruptedException e) {}
        });
        Thread t2 = new Thread(() -> {
            try {
                queue.put(2);
                System.out.println("Thread 2 produced 2");
            } catch (InterruptedException e) {}
        });
        t1.start();
        Thread.sleep(100);
        t2.start();
        Thread.sleep(100);
        System.out.println("Take: " + queue.take());
        System.out.println("Take: " + queue.take());
        t1.join();
        t2.join();
    }
}
```

Expected Output (order may vary):

```
Thread 1 produced 1
Take: 1
Thread 2 produced 2
Take: 2
```

**Real-World Cases with Explanation**

`ArrayBlockingQueue` is used in bounded buffer scenarios, such as a fixed-size request queue in a web server, a bounded task queue in a thread pool, and any producer–consumer system where the buffer size must be limited to prevent memory exhaustion.

**References Links**

- ArrayBlockingQueue (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ArrayBlockingQueue.html
- ArrayBlockingQueue (Java Platform SE 8) – https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ArrayBlockingQueue.html

#### 5.4 PriorityBlockingQueue

**Definitions**

- **Core Definition:** An unbounded blocking queue that uses the same ordering rules as `PriorityQueue` and supplies blocking retrieval operations.
- **Technical Definition:** `public class PriorityBlockingQueue<E> extends AbstractQueue<E> implements BlockingQueue<E>, Serializable`. This class does not permit `null` elements. A priority queue relying on natural ordering also does not permit insertion of non-comparable objects.
- **Beginner-Friendly Explanation:** `PriorityBlockingQueue` is a thread-safe priority queue. It combines the ordering of `PriorityQueue` with the blocking behaviour of `BlockingQueue`.

**Purposes**

- To provide a thread-safe, unbounded blocking priority queue.
- To support concurrent algorithms that require priority-based retrieval.
- To serve as a building block for concurrent task scheduling with priorities.

**Syntax Structures and Rules**

Complete general syntaxes:

```java
PriorityBlockingQueue()
PriorityBlockingQueue(int initialCapacity)
PriorityBlockingQueue(int initialCapacity, Comparator<? super E> comparator)
PriorityBlockingQueue(Collection<? extends E> c)
```

Component breakdown:

- `initialCapacity`: the initial capacity for the internal heap.
- `comparator`: the comparator used to order elements.
- `c`: the collection whose elements are to be placed into the queue.

Syntax rules: The iterator is not guaranteed to traverse elements in any particular order. `drainTo` can be used to remove elements in priority order.

Constraints and limitations: Logically unbounded, but attempted additions may fail due to resource exhaustion. `null` elements are not permitted.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Basic `PriorityBlockingQueue`**

```java
import java.util.concurrent.PriorityBlockingQueue;
import java.util.concurrent.BlockingQueue;

public class PriorityBlockingQueueExample {
    public static void main(String[] args) throws InterruptedException {
        BlockingQueue<Integer> pq = new PriorityBlockingQueue<>();
        pq.put(5);
        pq.put(1);
        pq.put(3);
        System.out.println("Take: " + pq.take()); // 1
        System.out.println("Take: " + pq.take()); // 3
        System.out.println("Take: " + pq.take()); // 5
    }
}
```

Expected Output:

```
Take: 1
Take: 3
Take: 5
```

**Example 2: Custom comparator**

```java
import java.util.Comparator;
import java.util.concurrent.PriorityBlockingQueue;
import java.util.concurrent.BlockingQueue;

public class PriorityBlockingQueueComparatorExample {
    public static void main(String[] args) throws InterruptedException {
        BlockingQueue<Integer> pq = new PriorityBlockingQueue<>(11, Comparator.reverseOrder());
        pq.put(5);
        pq.put(1);
        pq.put(3);
        System.out.println("Take: " + pq.take()); // 5
        System.out.println("Take: " + pq.take()); // 3
        System.out.println("Take: " + pq.take()); // 1
    }
}
```

Expected Output:

```
Take: 5
Take: 3
Take: 1
```

**Real-World Cases with Explanation**

`PriorityBlockingQueue` is used in concurrent task schedulers where tasks have different priorities, in concurrent Dijkstra implementations, and in any concurrent algorithm that requires priority-based retrieval.

**References Links**

- PriorityBlockingQueue (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/PriorityBlockingQueue.html
- PriorityBlockingQueue (Java Platform SE 8) – https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/PriorityBlockingQueue.html

#### 5.5 DelayQueue

**Definitions**

- **Core Definition:** An unbounded blocking queue of `Delayed` elements, in which an element generally becomes eligible for removal when its delay has expired.
- **Technical Definition:** `public class DelayQueue<E extends Delayed> extends AbstractQueue<E> implements BlockingQueue<E>`. While this class implements the `BlockingQueue` interface, it intentionally violates the general contract of `BlockingQueue`, in that the following methods disregard the presence of unexpired elements and only ever remove the expired head.
- **Beginner-Friendly Explanation:** `DelayQueue` is a queue where elements cannot be taken until a certain amount of time has passed. It is useful for scheduling tasks that should be executed after a delay.

**Purposes**

- To provide a time-based scheduling queue.
- To support scenarios where elements become available only after a specified delay.
- To serve as a building block for scheduled task execution systems.

**Syntax Structures and Rules**

Complete general syntaxes:

```java
DelayQueue()
```

Component breakdown:

- Elements must implement the `Delayed` interface.

Syntax rules: Elements are ordered by their delay. The `peek` method may return a non-null head even when `take` would block waiting for that element to expire.

Constraints and limitations: `null` elements are not permitted.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: `DelayQueue` with delayed elements**

```java
import java.util.concurrent.DelayQueue;
import java.util.concurrent.Delayed;
import java.util.concurrent.TimeUnit;

class DelayedItem implements Delayed {
    private final String name;
    private final long delayTime;

    public DelayedItem(String name, long delay, TimeUnit unit) {
        this.name = name;
        this.delayTime = System.currentTimeMillis() + unit.toMillis(delay);
    }

    @Override
    public long getDelay(TimeUnit unit) {
        return unit.convert(delayTime - System.currentTimeMillis(), TimeUnit.MILLISECONDS);
    }

    @Override
    public int compareTo(Delayed o) {
        return Long.compare(this.delayTime, ((DelayedItem) o).delayTime);
    }

    @Override
    public String toString() { return name; }
}

public class DelayQueueExample {
    public static void main(String[] args) throws InterruptedException {
        DelayQueue<DelayedItem> queue = new DelayQueue<>();
        queue.put(new DelayedItem("Task1", 2, TimeUnit.SECONDS));
        queue.put(new DelayedItem("Task2", 1, TimeUnit.SECONDS));
        System.out.println("Taking...");
        System.out.println("Took: " + queue.take()); // Task2 after 1 second
        System.out.println("Took: " + queue.take()); // Task1 after another second
    }
}
```

Expected Output (after ~2 seconds total):

```
Taking...
Took: Task2
Took: Task1
```

**Real-World Cases with Explanation**

`DelayQueue` is used in scheduled thread pools, cache expiration systems (where entries expire after a certain time), and session timeout management.

**References Links**

- DelayQueue (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/DelayQueue.html
- DelayQueue (Java Platform SE 8) – https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/DelayQueue.html

#### 5.6 SynchronousQueue

**Definitions**

- **Core Definition:** A blocking queue in which each insert operation must wait for a corresponding remove operation by another thread, and vice versa.
- **Technical Definition:** `public class SynchronousQueue<E> extends AbstractQueue<E> implements BlockingQueue<E>, Serializable`. A synchronous queue does not have any internal capacity, not even a capacity of one. You cannot `peek` at a synchronous queue because an element is only present when you try to remove it.
- **Beginner-Friendly Explanation:** `SynchronousQueue` is a meeting point for two threads. One thread offers an element, and the other takes it. There is no storage—the handoff is direct.

**Purposes**

- To provide a zero-capacity handoff queue.
- To support rendezvous-style communication between threads.
- To serve as a building block for handoff designs.

**Syntax Structures and Rules**

Complete general syntaxes:

```java
SynchronousQueue()
SynchronousQueue(boolean fair)
```

Component breakdown:

- `fair`: if `true`, grants threads access in FIFO order.

Syntax rules: `put` blocks until another thread calls `take`, and `take` blocks until another thread calls `put`. `peek` always returns `null`.

Constraints and limitations: No internal capacity. `null` elements are not permitted.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Handoff with `SynchronousQueue`**

```java
import java.util.concurrent.SynchronousQueue;
import java.util.concurrent.BlockingQueue;

public class SynchronousQueueExample {
    public static void main(String[] args) throws InterruptedException {
        BlockingQueue<String> queue = new SynchronousQueue<>();
        Thread producer = new Thread(() -> {
            try {
                System.out.println("Producer: putting...");
                queue.put("Handoff");
                System.out.println("Producer: put completed");
            } catch (InterruptedException e) {}
        });
        Thread consumer = new Thread(() -> {
            try {
                Thread.sleep(1000);
                System.out.println("Consumer: taking...");
                String item = queue.take();
                System.out.println("Consumer: took " + item);
            } catch (InterruptedException e) {}
        });
        producer.start();
        consumer.start();
        producer.join();
        consumer.join();
    }
}
```

Expected Output:

```
Producer: putting...
Consumer: taking...
Consumer: took Handoff
Producer: put completed
```

**Real-World Cases with Explanation**

`SynchronousQueue` is used in thread pools (such as `Executors.newCachedThreadPool()`), where tasks are handed off directly to waiting worker threads without being stored in a queue.

**References Links**

- SynchronousQueue (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/SynchronousQueue.html
- SynchronousQueue (Java Platform SE 8) – https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/SynchronousQueue.html

#### 5.7 TransferQueue and LinkedTransferQueue

**Definitions**

- **Core Definition:** A `BlockingQueue` in which producers may wait for consumers to receive elements.
- **Technical Definition:** `public interface TransferQueue<E> extends BlockingQueue<E>`. A `TransferQueue` may be useful for example in message passing applications in which producers sometimes (using method `transfer(E)`) await receipt of elements by consumers invoking `take` or `poll`, while at other times enqueue elements (via method `put`) without waiting for receipt.
- **Beginner-Friendly Explanation:** `TransferQueue` is like a blocking queue but with an extra guarantee: when you `transfer` an element, you know that a consumer has received it. `LinkedTransferQueue` is the standard implementation.

**Purposes**

- To provide a queue where producers can wait for consumers to receive elements.
- To support message-passing applications with guaranteed delivery.
- To offer both blocking and non-blocking transfer operations.

**Syntax Structures and Rules**

Complete general syntaxes:

```java
void transfer(E e) throws InterruptedException
boolean tryTransfer(E e)
boolean tryTransfer(E e, long timeout, TimeUnit unit) throws InterruptedException
boolean hasWaitingConsumer()
int getWaitingConsumerCount()
```

Component breakdown:

- `e`: the element to transfer.
- `transfer`: blocks until the element is received by a consumer.
- `tryTransfer`: transfers immediately if a consumer is waiting; otherwise returns `false` without enqueuing.
- `hasWaitingConsumer`: returns `true` if at least one consumer is waiting.

Syntax rules: `LinkedTransferQueue` is an unbounded `TransferQueue` based on linked nodes. It is the only standard implementation of `TransferQueue`.

Constraints and limitations: `transfer` may block indefinitely. `null` elements are not permitted.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: `LinkedTransferQueue` with `transfer`**

```java
import java.util.concurrent.LinkedTransferQueue;
import java.util.concurrent.TransferQueue;

public class TransferQueueExample {
    public static void main(String[] args) throws InterruptedException {
        TransferQueue<String> queue = new LinkedTransferQueue<>();
        Thread producer = new Thread(() -> {
            try {
                System.out.println("Producer: transferring...");
                queue.transfer("Data");
                System.out.println("Producer: transfer completed");
            } catch (InterruptedException e) {}
        });
        Thread consumer = new Thread(() -> {
            try {
                Thread.sleep(1000);
                System.out.println("Consumer: taking...");
                String item = queue.take();
                System.out.println("Consumer: took " + item);
            } catch (InterruptedException e) {}
        });
        producer.start();
        consumer.start();
        producer.join();
        consumer.join();
    }
}
```

Expected Output:

```
Producer: transferring...
Consumer: taking...
Consumer: took Data
Producer: transfer completed
```

**Real-World Cases with Explanation**

`LinkedTransferQueue` is used in message-passing frameworks where producers need confirmation that a message has been received, and in systems that require both blocking and non-blocking transfer semantics.

**References Links**

- TransferQueue (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/TransferQueue.html
- LinkedTransferQueue (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/LinkedTransferQueue.html

### 6. The BlockingDeque Interface

**Core Definition:** A `Deque` that additionally supports blocking operations that wait for the deque to become non-empty when retrieving an element, and wait for space to become available in the deque when storing an element.

**Technical Definition:** `public interface BlockingDeque<E> extends BlockingQueue<E>, Deque<E>`. `BlockingDeque` methods come in four forms: one throws an exception, the second returns a special value, the third blocks the current thread indefinitely, and the fourth blocks for a given maximum time limit. These methods are summarized for both the head and tail ends.

**Beginner-Friendly Explanation:** A `BlockingDeque` is a deque that can block. It combines the flexibility of a double-ended queue with the blocking behaviour of a `BlockingQueue`.

**Purposes**

- To provide a thread-safe, blocking double-ended queue.
- To support both FIFO and LIFO blocking operations.
- To serve as a building block for work-stealing algorithms with blocking.

**Syntax Structures and Rules**

Complete general syntaxes (selected):

```java
void putFirst(E e) throws InterruptedException
void putLast(E e) throws InterruptedException
E takeFirst() throws InterruptedException
E takeLast() throws InterruptedException
```

Component breakdown:

- `e`: the element to be inserted.
- `putFirst` and `putLast` block until space is available.
- `takeFirst` and `takeLast` block until an element is available.

Syntax rules: Like any `BlockingQueue`, a `BlockingDeque` is thread safe, does not permit `null` elements, and may be capacity-constrained. A `BlockingDeque` implementation may be used directly as a FIFO `BlockingQueue`.

Constraints and limitations: `null` elements are not permitted. `LinkedBlockingDeque` is the standard implementation.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: `LinkedBlockingDeque` with blocking operations**

```java
import java.util.concurrent.LinkedBlockingDeque;
import java.util.concurrent.BlockingDeque;

public class BlockingDequeExample {
    public static void main(String[] args) throws InterruptedException {
        BlockingDeque<String> deque = new LinkedBlockingDeque<>(2);
        deque.putFirst("First");
        deque.putLast("Last");
        System.out.println("Deque: " + deque);
        System.out.println("takeFirst: " + deque.takeFirst());
        System.out.println("takeLast: " + deque.takeLast());
    }
}
```

Expected Output:

```
Deque: [First, Last]
takeFirst: First
takeLast: Last
```

**Example 2: Blocking `putFirst` on a full deque**

```java
import java.util.concurrent.LinkedBlockingDeque;
import java.util.concurrent.BlockingDeque;

public class BlockingDequePutBlockExample {
    public static void main(String[] args) throws InterruptedException {
        BlockingDeque<Integer> deque = new LinkedBlockingDeque<>(1);
        deque.putFirst(1);
        Thread producer = new Thread(() -> {
            try {
                System.out.println("Producer: putFirst blocking...");
                deque.putFirst(2);
                System.out.println("Producer: putFirst succeeded");
            } catch (InterruptedException e) {}
        });
        Thread consumer = new Thread(() -> {
            try {
                Thread.sleep(1000);
                System.out.println("Consumer: takeFirst: " + deque.takeFirst());
            } catch (InterruptedException e) {}
        });
        producer.start();
        consumer.start();
        producer.join();
        consumer.join();
    }
}
```

Expected Output:

```
Producer: putFirst blocking...
Consumer: takeFirst: 1
Producer: putFirst succeeded
```

**Real-World Cases with Explanation**

`LinkedBlockingDeque` is used in work-stealing schedulers where workers can steal tasks from either end of a deque, and in systems that require both FIFO and LIFO blocking semantics.

**References Links**

- BlockingDeque (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/BlockingDeque.html
- LinkedBlockingDeque (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/LinkedBlockingDeque.html

## Summary Table of Implementations

| Class | Type | Bounded | Thread-Safe | Blocking | Ordering |
|---|---|---|---|---|---|
| ArrayDeque | Deque | No | No | No | FIFO / LIFO |
| LinkedList | Deque | No | No | No | FIFO / LIFO |
| PriorityQueue | Queue | No | No | No | Priority |
| ConcurrentLinkedQueue | Queue | No | Yes | No | FIFO |
| ConcurrentLinkedDeque | Deque | No | Yes | No | FIFO / LIFO |
| LinkedBlockingQueue | BlockingQueue | Optional | Yes | Yes | FIFO |
| LinkedBlockingDeque | BlockingDeque | Optional | Yes | Yes | FIFO / LIFO |
| ArrayBlockingQueue | BlockingQueue | Yes | Yes | Yes | FIFO |
| PriorityBlockingQueue | BlockingQueue | No | Yes | Yes | Priority |
| DelayQueue | BlockingQueue | No | Yes | Yes | Delay |
| SynchronousQueue | BlockingQueue | Zero | Yes | Yes | Handoff |
| LinkedTransferQueue | TransferQueue | No | Yes | Yes | FIFO |

## Deprecated, Unsafe, and Version-Specific Notes

- The legacy `Stack` class is deprecated in favour of `Deque` implementations such as `ArrayDeque`. The `Deque` interface should be used in preference to `Stack` when a stack is needed.
- `ArrayDeque`, `LinkedList`, and `PriorityQueue` are not thread-safe. They must not be accessed concurrently by multiple threads without external synchronisation.
- `ConcurrentLinkedQueue` and `ConcurrentLinkedDeque` are non-blocking and do not support `null` elements. Their `size()` method is not a constant-time operation.
- `LinkedBlockingQueue`, `ArrayBlockingQueue`, `PriorityBlockingQueue`, `DelayQueue`, and `SynchronousQueue` are thread-safe blocking queues. They do not accept `null` elements.
- `TransferQueue` and `LinkedTransferQueue` were introduced in Java 7.
- `BlockingDeque` and `LinkedBlockingDeque` were introduced in Java 6.
- `Queue`, `BlockingQueue`, `PriorityQueue`, and `ConcurrentLinkedQueue` were introduced in Java 5.

## References

- Queue (Java Platform SE 8) – https://docs.oracle.com/javase/8/docs/api/java/util/Queue.html
- Queue (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Queue.html
- Deque (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Deque.html
- Deque (Java Platform SE 8) – https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html
- BlockingQueue (Java SE 11) – https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/BlockingQueue.html
- BlockingQueue (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/BlockingQueue.html
- ArrayDeque (Java SE 26) – https://docs.oracle.com/en/java/javase/26/docs/api/java.base/java/util/ArrayDeque.html
- LinkedList (Java Platform SE 8) – https://docs.oracle.com/javase/8/docs/api/java/util/LinkedList.html
- PriorityQueue (Java SE 25) – https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/PriorityQueue.html
- ConcurrentLinkedQueue (Java SE 11) – https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/ConcurrentLinkedQueue.html
- LinkedBlockingQueue (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/LinkedBlockingQueue.html
- ArrayBlockingQueue (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ArrayBlockingQueue.html
- PriorityBlockingQueue (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/PriorityBlockingQueue.html
- DelayQueue (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/DelayQueue.html
- SynchronousQueue (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/SynchronousQueue.html
- TransferQueue (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/TransferQueue.html
- BlockingDeque (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/BlockingDeque.html
- LinkedTransferQueue (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/LinkedTransferQueue.html
- LinkedBlockingDeque (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/LinkedBlockingDeque.html
- Collections Framework Enhancements – https://docs.oracle.com/javase/8/docs/technotes/guides/collections/changes5.html