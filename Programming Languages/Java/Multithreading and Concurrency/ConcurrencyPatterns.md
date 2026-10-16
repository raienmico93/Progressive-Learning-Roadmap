# Java Concurrency Patterns: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Concurrency patterns are reusable, proven solutions to common problems that arise when multiple threads or processes execute simultaneously, coordinating access to shared resources and managing task lifecycles.

**Technical Definition:** Concurrency patterns are design patterns specifically addressing the challenges of concurrent execution: race conditions, deadlocks, starvation, memory visibility, and resource contention. They provide structured approaches to coordinating threads, managing shared state, distributing work, and handling asynchronous results. In Java, these patterns are implemented using the `java.util.concurrent` package, `synchronized`, `Lock`, `ExecutorService`, `CompletableFuture`, and higher-level abstractions.

**Beginner-Friendly Explanation:** Imagine a busy restaurant kitchen. Without organisation, chaos reigns — chefs bump into each other, orders get mixed up, and food gets cold. Concurrency patterns are like standard kitchen procedures: one person takes orders (producer), another cooks (consumer), a head chef assigns tasks (worker pool), and a runner delivers food (future). These patterns make the kitchen run smoothly, even when it's packed with customers. In Java, concurrency patterns help you write multithreaded code that is correct, efficient, and maintainable.

### Key Characteristics

- **Proven Solutions:** Patterns are battle-tested approaches to common concurrency problems.
- **Reusability:** Patterns can be applied across different projects and domains.
- **Communication-Oriented:** Most patterns focus on how threads communicate and coordinate.
- **Resource-Aware:** Patterns manage shared resources (CPU, memory, I/O) efficiently.
- **Scalability-Focused:** Patterns help applications scale to handle more concurrent work.
- **Safety-First:** Patterns prioritise correctness (no race conditions, deadlocks, or data corruption).

### Prerequisites

- Solid understanding of Java threads and the `Thread` class
- Familiarity with `synchronized`, `Lock`, and `volatile`
- Knowledge of the `java.util.concurrent` package
- Understanding of `ExecutorService`, `Future`, and `CompletableFuture`
- Basic knowledge of blocking queues and concurrent collections

### Related Programming Areas

- **Concurrency Fundamentals:** Threads, synchronization, and the Java Memory Model.
- **Executors:** Thread pools and task scheduling.
- **Futures and CompletableFuture:** Asynchronous result handling.
- **Concurrent Collections:** Thread-safe data structures.
- **Reactive Programming:** Event-driven, non-blocking patterns.
- **Distributed Systems:** Patterns that extend to multiple JVMs (Actor model).

### Core Concepts / Features

---

## 1. Producer-Consumer

### Definitions

**Core Definition:** The producer-consumer pattern decouples data production from data consumption by using a shared buffer (queue) between producer threads and consumer threads.

**Technical Definition:** "The producer-consumer pattern is a concurrency design pattern where one set of threads (producers) generate data and place it into a shared buffer, while another set of threads (consumers) retrieve and process that data. The buffer decouples producers from consumers, allowing them to operate at different rates. In Java, this pattern is typically implemented using `BlockingQueue` implementations, which handle the synchronization and blocking automatically".

**Beginner-Friendly Explanation:** Imagine a bakery. Bakers (producers) make bread and place it on a shelf (the queue). Customers (consumers) take bread from the shelf. If the shelf is full, bakers wait. If the shelf is empty, customers wait. The shelf decouples the bakers from the customers — they don't need to coordinate directly. This is the producer-consumer pattern.

### Purposes

- To decouple data production from data consumption
- To allow producers and consumers to operate at different rates
- To buffer bursts of data without blocking producers
- To provide backpressure when the buffer is full
- To enable efficient use of resources by overlapping production and consumption

### Syntax Rules and Structure

**Complete General Syntax:**

```java
BlockingQueue<T> queue = new ArrayBlockingQueue<>(capacity);

// Producer
queue.put(item);  // Blocks if full

// Consumer
T item = queue.take();  // Blocks if empty
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `BlockingQueue` | The shared buffer |
| `put()` | Adds an item (blocks if full) |
| `take()` | Removes an item (blocks if empty) |
| `offer()` | Adds an item (returns false if full) |
| `poll()` | Removes an item (returns null if empty) |

**Syntax Rules:**

- Use `put()`/`take()` for blocking behaviour.
- Use `offer()`/`poll()` with timeouts for bounded waiting.
- Use `ArrayBlockingQueue` for bounded buffers, `LinkedBlockingQueue` for optionally bounded, `SynchronousQueue` for direct handoff.
- The queue handles all synchronization; no explicit locks are needed.

### Annotated Complete Step-by-Step Code Examples

```java
// File: ProducerConsumerDemo.java
import java.util.concurrent.*;

public class ProducerConsumerDemo {
    public static void main(String[] args) throws InterruptedException {
        // Step 1: Create a bounded blocking queue
        BlockingQueue<Integer> queue = new ArrayBlockingQueue<>(5);

        // Step 2: Producer thread
        Thread producer = new Thread(() -> {
            try {
                for (int i = 1; i <= 10; i++) {
                    queue.put(i);  // Blocks if full
                    System.out.println("Produced: " + i +
                        " (queue size: " + queue.size() + ")");
                    Thread.sleep(200);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }, "Producer");

        // Step 3: Consumer thread
        Thread consumer = new Thread(() -> {
            try {
                for (int i = 0; i < 10; i++) {
                    int value = queue.take();  // Blocks if empty
                    System.out.println("Consumed: " + value +
                        " (queue size: " + queue.size() + ")");
                    Thread.sleep(500);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }, "Consumer");

        // Step 4: Start both threads
        producer.start();
        consumer.start();
        producer.join();
        consumer.join();

        System.out.println("Done");
    }
}
```

**Expected Output (interleaved):**
```
Produced: 1 (queue size: 1)
Consumed: 1 (queue size: 0)
Produced: 2 (queue size: 1)
Produced: 3 (queue size: 2)
Consumed: 2 (queue size: 1)
...
Done
```

**Step-by-Step Explanation:**

1. An `ArrayBlockingQueue` with capacity 5 is created as the shared buffer.
2. The producer calls `queue.put(i)`, which blocks if the queue is full (size 5).
3. The consumer calls `queue.take()`, which blocks if the queue is empty.
4. The queue handles all synchronization — no explicit locks or `wait()`/`notify()` needed.
5. The producer sleeps 200ms between items; the consumer sleeps 500ms, so the producer is faster and the queue fills up, causing the producer to block.
6. Both threads complete after 10 items.

### Real-World Cases

- **Web Servers:** Request producers (network listeners) and request consumers (worker threads).
- **Log Processing:** Log producers (application threads) and log consumers (file writers).
- **Data Pipelines:** Data producers (sensors) and data consumers (analytics engines).
- **Task Queues:** `ThreadPoolExecutor` uses a `BlockingQueue` internally.

---

## 2. Reader-Writer

### Definitions

**Core Definition:** The reader-writer pattern allows multiple concurrent readers or a single exclusive writer to access a shared resource, improving concurrency for read-heavy workloads.

**Technical Definition:** "A ReadWriteLock maintains a pair of associated locks, one for read-only operations and one for writing. The read lock may be held simultaneously by multiple reader threads, so long as there are no writers. The write lock is exclusive". `ReentrantReadWriteLock` and `StampedLock` are the primary implementations in Java.

**Beginner-Friendly Explanation:** Imagine a library with a rare book. Many people can read the book at the same time (readers), but if someone wants to edit the book, they need exclusive access (writer). The reader-writer pattern allows this: many readers OR one writer, but never both simultaneously.

### Purposes

- To allow concurrent read access while ensuring exclusive write access
- To improve throughput in read-heavy scenarios
- To prevent data corruption from concurrent reads and writes
- To provide a structured way to manage read and write access

### Syntax Rules and Structure

```java
ReentrantReadWriteLock rwLock = new ReentrantReadWriteLock();
Lock readLock = rwLock.readLock();
Lock writeLock = rwLock.writeLock();

// Read
readLock.lock();
try { /* read */ } finally { readLock.unlock(); }

// Write
writeLock.lock();
try { /* write */ } finally { writeLock.unlock(); }
```

**Key Characteristics:**

| Aspect | Description |
|--------|-------------|
| Read lock | Shared — multiple readers can hold it |
| Write lock | Exclusive — only one writer, no readers |
| Downgrade | Write → read allowed |
| Upgrade | Read → write not allowed (deadlock) |

### Annotated Complete Step-by-Step Code Examples

```java
// File: ReaderWriterDemo.java
import java.util.concurrent.locks.*;

public class ReaderWriterDemo {
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
        Thread r1 = new Thread(ReaderWriterDemo::readData, "Reader-1");
        Thread r2 = new Thread(ReaderWriterDemo::readData, "Reader-2");
        Thread w1 = new Thread(() -> writeData(42), "Writer-1");

        r1.start(); r2.start(); w1.start();
        r1.join(); r2.join(); w1.join();
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

1. Two reader threads acquire the read lock concurrently — both can read simultaneously.
2. The writer thread acquires the write lock, which blocks all readers.
3. The writer updates the data and releases the write lock.
4. Readers can resume if they wish.

### Real-World Cases

- **Caching Systems:** Many reads, occasional cache invalidation.
- **Configuration Management:** Frequent reads, infrequent updates.
- **Database Access:** Read-heavy workloads with periodic writes.

---

## 3. Worker Pool (Thread Pool)

### Definitions

**Core Definition:** The worker pool (thread pool) pattern maintains a set of reusable worker threads that execute submitted tasks from a shared queue.

**Technical Definition:** "Thread pools address two different problems: they usually provide improved performance when executing large numbers of asynchronous tasks, due to reduced per-task invocation overhead, and they provide a means of bounding and managing the resources, including threads, consumed when executing a collection of tasks". In Java, this is implemented via `ExecutorService` and `ThreadPoolExecutor`.

**Beginner-Friendly Explanation:** A worker pool is like a team of permanent employees. When work arrives, an available employee handles it. When done, they wait for the next task. This is much more efficient than hiring and firing someone for every task.

### Purposes

- To reduce the overhead of thread creation and destruction
- To bound the number of concurrent threads and manage resources
- To provide task queuing and backpressure
- To improve performance for large numbers of small tasks

### Syntax Rules and Structure

```java
ExecutorService executor = Executors.newFixedThreadPool(4);

// Submit tasks
Future<?> future = executor.submit(() -> { /* task */ });

// Shutdown
executor.shutdown();
executor.awaitTermination(5, TimeUnit.SECONDS);
```

**Key Concepts:**

| Concept | Description |
|---------|-------------|
| Core Pool Size | Number of threads to keep, even if idle |
| Maximum Pool Size | Maximum number of threads allowed |
| Keep-Alive Time | How long excess idle threads wait before terminating |
| Work Queue | Queue holding tasks before execution |
| Rejection Handler | Policy for handling tasks when the queue is full |

### Annotated Complete Step-by-Step Code Examples

```java
// File: WorkerPoolDemo.java
import java.util.concurrent.*;

public class WorkerPoolDemo {
    public static void main(String[] args) throws Exception {
        // Create a custom thread pool
        ThreadPoolExecutor executor = new ThreadPoolExecutor(
            2,                              // corePoolSize
            4,                              // maximumPoolSize
            60, TimeUnit.SECONDS,           // keepAliveTime
            new ArrayBlockingQueue<>(10),   // bounded work queue
            Executors.defaultThreadFactory(),
            new ThreadPoolExecutor.CallerRunsPolicy()
        );

        // Submit tasks
        for (int i = 1; i <= 20; i++) {
            final int taskId = i;
            executor.execute(() -> {
                System.out.println(Thread.currentThread().getName() +
                    " executing task " + taskId);
                try { Thread.sleep(100); } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            });
        }

        executor.shutdown();
        executor.awaitTermination(10, TimeUnit.SECONDS);
        System.out.println("All tasks completed");
    }
}
```

**Expected Output (interleaved):**
```
pool-1-thread-1 executing task 1
pool-1-thread-2 executing task 2
main executing task 11
...
All tasks completed
```

### Real-World Cases

- **Web Servers:** Bounded pools for request handling.
- **Database Connection Pools:** Fixed pools of connection worker threads.
- **Image Processing:** Parallel processing of image tiles.

---

## 4. Future and Promise

### Definitions

**Core Definition:** `Future` represents the result of an asynchronous computation; `Promise` (in Java, `CompletableFuture`) is a writable `Future` that can be explicitly completed and composed.

**Technical Definition:** "A `Future` represents the result of an asynchronous computation. Methods are provided to check if the computation is complete, to wait for its completion, and to retrieve the result". "A `CompletableFuture` is a `Future` that may be explicitly completed, and may be used as a `CompletionStage`, supporting dependent functions and actions that trigger upon its completion".

**Beginner-Friendly Explanation:** A `Future` is like a claim ticket at a coat check — you can use it to retrieve your coat later. A `Promise` (`CompletableFuture`) is like a claim ticket that you can also give to someone else and say "when the coat is ready, add a scarf and then notify me."

### Purposes

- To represent the result of an asynchronous computation
- To decouple task submission from result retrieval
- To enable non-blocking composition of asynchronous operations
- To provide robust error handling in async pipelines

### Syntax Rules and Structure

```java
// Future (basic)
Future<String> future = executor.submit(() -> "result");
String result = future.get();  // Blocks

// CompletableFuture (composable)
CompletableFuture<String> cf = CompletableFuture
    .supplyAsync(() -> "Hello")
    .thenApply(s -> s + " World")
    .thenApply(String::toUpperCase);
System.out.println(cf.get());  // HELLO WORLD
```

### Annotated Code Example

```java
// File: FuturePromiseDemo.java
import java.util.concurrent.*;

public class FuturePromiseDemo {
    public static void main(String[] args) throws Exception {
        // Future: basic async result
        ExecutorService executor = Executors.newSingleThreadExecutor();
        Future<Integer> future = executor.submit(() -> {
            Thread.sleep(500);
            return 42;
        });
        System.out.println("Future result: " + future.get());

        // CompletableFuture: composable pipeline
        CompletableFuture<String> pipeline = CompletableFuture
            .supplyAsync(() -> "user-123")
            .thenCompose(id -> CompletableFuture.supplyAsync(() -> "Profile: " + id))
            .thenApply(String::toUpperCase);
        System.out.println("Pipeline: " + pipeline.get());  // PROFILE: USER-123

        // Exception handling
        CompletableFuture<String> withFallback = CompletableFuture
            .supplyAsync(() -> { throw new RuntimeException("Error"); })
            .exceptionally(ex -> "Fallback");
        System.out.println("Fallback: " + withFallback.get());  // Fallback

        executor.shutdown();
    }
}
```

**Expected Output:**
```
Future result: 42
Pipeline: PROFILE: USER-123
Fallback: Fallback
```

### Real-World Cases

- **API Calls:** Making HTTP requests asynchronously and retrieving responses.
- **Database Queries:** Running long queries in the background.
- **Microservices:** Combining results from multiple services.

---

## 5. Actor Model

### Definitions

**Core Definition:** The Actor model is a concurrency paradigm where "actors" are independent entities that communicate exclusively through asynchronous message passing, each processing messages sequentially from its own mailbox.

**Technical Definition:** "The Actor model is a mathematical model of concurrent computation that treats 'actors' as the universal primitives of concurrent computation. In response to a message that it receives, an actor can: make local decisions, create more actors, send more messages, and determine how to respond to the next message received. Actors may modify their own private state, but can only affect each other through messages". In Java, the Actor model is not built into the JDK but can be implemented using `BlockingQueue` or third-party libraries like Akka.

**Beginner-Friendly Explanation:** The Actor model is like a company where each employee has their own inbox. Employees never talk to each other directly — they only send messages to each other's inboxes. Each employee processes their inbox one message at a time. If an employee needs something from another, they send a message and continue working; the reply will arrive in their inbox later. This eliminates shared-memory concurrency problems entirely.

### Purposes

- To eliminate shared-memory concurrency issues (no locks needed)
- To provide a natural model for distributed systems
- To enable location transparency (actors can be local or remote)
- To simplify reasoning about concurrent behaviour
- To support fault tolerance through supervision hierarchies

### Syntax Rules and Structure (Simplified Implementation)

```java
class Actor extends Thread {
    private final BlockingQueue<Message> mailbox = new LinkedBlockingQueue<>();
    private volatile boolean running = true;

    public void send(Message msg) { mailbox.offer(msg); }

    public void run() {
        while (running) {
            Message msg = mailbox.take();
            process(msg);
        }
    }
}
```

**Key Concepts:**

| Concept | Description |
|---------|-------------|
| Actor | An independent entity with private state |
| Mailbox | A queue of incoming messages |
| Message | An immutable data object sent between actors |
| Supervision | Parent actors manage child actor failures |

### Annotated Complete Step-by-Step Code Examples

```java
// File: ActorModelDemo.java
import java.util.concurrent.*;

public class ActorModelDemo {
    // Message type
    record Message(String from, String content) {}

    // Actor implementation
    static class Actor extends Thread {
        private final BlockingQueue<Message> mailbox = new LinkedBlockingQueue<>();
        private final String name;

        Actor(String name) {
            super(name);
            this.name = name;
        }

        void send(Message msg) {
            mailbox.offer(msg);
        }

        @Override
        public void run() {
            try {
                while (!Thread.currentThread().isInterrupted()) {
                    Message msg = mailbox.take();  // Blocks until message arrives
                    System.out.println(name + " received from " + msg.from() +
                        ": " + msg.content());
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Actor alice = new Actor("Alice");
        Actor bob = new Actor("Bob");

        alice.start();
        bob.start();

        // Send messages
        alice.send(new Message("Bob", "Hello Alice!"));
        bob.send(new Message("Alice", "Hi Bob!"));

        Thread.sleep(1000);

        alice.interrupt();
        bob.interrupt();
    }
}
```

**Expected Output:**
```
Alice received from Bob: Hello Alice!
Bob received from Alice: Hi Bob!
```

**Step-by-Step Explanation:**

1. Each `Actor` has a `BlockingQueue` as its mailbox.
2. `send()` puts a message in the mailbox (non-blocking).
3. `run()` loops, taking messages from the mailbox and processing them.
4. Actors never share state — they only communicate via messages.
5. No locks or synchronization are needed because each actor processes messages sequentially.

### Real-World Cases

- **Distributed Systems:** Akka, Erlang/OTP, and Orleans use the Actor model.
- **Game Servers:** Each game entity can be an actor.
- **IoT Systems:** Devices communicate via messages.
- **Microservices:** Actors model service boundaries and communication.

---

## 6. Reactor Pattern

### Definitions

**Core Definition:** The Reactor pattern is an event-driven architecture where a single-threaded event loop demultiplexes incoming events and dispatches them to registered handlers.

**Technical Definition:** "The reactor design pattern is an event handling pattern for handling service requests delivered concurrently to a service handler by one or more inputs. The service handler then demultiplexes the incoming requests and dispatches them synchronously to the associated request handlers". In Java, this pattern is implemented using `Selector` and `SelectableChannel` from the NIO package, and is the foundation of frameworks like Netty.

**Beginner-Friendly Explanation:** The Reactor pattern is like a restaurant host who greets all incoming customers, takes their names, and directs them to the appropriate table. The host doesn't cook or serve — they just manage the flow of customers. In Java NIO, the `Selector` is the host, channels are the customers, and handlers are the tables.

### Purposes

- To handle many concurrent connections with a single thread
- To decouple event demultiplexing from event handling
- To enable non-blocking, event-driven I/O
- To provide a scalable foundation for network servers

### Syntax Rules and Structure (Simplified)

```java
Selector selector = Selector.open();
ServerSocketChannel server = ServerSocketChannel.open();
server.configureBlocking(false);
server.register(selector, SelectionKey.OP_ACCEPT);

while (true) {
    selector.select();  // Blocks until events
    Set<SelectionKey> keys = selector.selectedKeys();
    for (SelectionKey key : keys) {
        if (key.isAcceptable()) handleAccept(key);
        if (key.isReadable()) handleRead(key);
    }
    keys.clear();
}
```

### Annotated Code Example

```java
// File: ReactorPatternDemo.java
import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.*;
import java.util.Iterator;
import java.util.Set;

public class ReactorPatternDemo {
    public static void main(String[] args) throws IOException {
        // Create selector and server channel
        Selector selector = Selector.open();
        ServerSocketChannel server = ServerSocketChannel.open();
        server.bind(new InetSocketAddress(8080));
        server.configureBlocking(false);
        server.register(selector, SelectionKey.OP_ACCEPT);

        System.out.println("Reactor server on port 8080");
        ByteBuffer buffer = ByteBuffer.allocate(256);

        while (true) {
            selector.select();  // Block until events
            Set<SelectionKey> keys = selector.selectedKeys();
            Iterator<SelectionKey> iter = keys.iterator();

            while (iter.hasNext()) {
                SelectionKey key = iter.next();
                iter.remove();

                if (key.isAcceptable()) {
                    // Handle new connection
                    SocketChannel client = server.accept();
                    client.configureBlocking(false);
                    client.register(selector, SelectionKey.OP_READ);
                    System.out.println("Accepted: " + client.getRemoteAddress());
                } else if (key.isReadable()) {
                    // Handle read event
                    SocketChannel client = (SocketChannel) key.channel();
                    buffer.clear();
                    int bytesRead = client.read(buffer);
                    if (bytesRead == -1) { client.close(); continue; }
                    buffer.flip();
                    byte[] data = new byte[buffer.remaining()];
                    buffer.get(data);
                    System.out.println("Received: " + new String(data).trim());
                }
            }
        }
    }
}
```

**Expected Output (when a client sends "Hello"):**
```
Reactor server on port 8080
Accepted: /127.0.0.1:52345
Received: Hello
```

### Real-World Cases

- **Netty:** A high-performance network framework based on the Reactor pattern.
- **Node.js:** Uses an event loop similar to the Reactor pattern.
- **Nginx:** Uses an event-driven architecture for handling connections.
- **Vert.x:** A polyglot event-driven framework for the JVM.

---

## 7. Concurrency Best Practices

### Definitions

**Core Definition:** Concurrency best practices are guidelines for writing correct, efficient, and maintainable concurrent Java code.

**Technical Definition:** Best practices derive from authoritative sources including the SEI CERT Oracle Coding Standard for Java. Key principles include using thread pools, avoiding blocking while holding locks, preferring higher-level concurrency utilities, and documenting concurrency policies.

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
| Use Virtual Threads for I/O-Bound Tasks | "Virtual threads are suitable for executing tasks that spend most of the time blocked". | Oracle |
| Document Concurrency Policy | Explain which locks protect which state. | Effective Java |

### Annotated Code Example

```java
// File: ConcurrencyBestPractices.java
import java.util.concurrent.*;

public class ConcurrencyBestPractices {
    public static void main(String[] args) throws Exception {
        // BEST PRACTICE: Use thread pools, not ad-hoc threads
        ExecutorService executor = Executors.newFixedThreadPool(4);

        // BEST PRACTICE: Use bounded queues to prevent memory exhaustion
        ThreadPoolExecutor boundedPool = new ThreadPoolExecutor(
            2, 4, 60L, TimeUnit.SECONDS,
            new ArrayBlockingQueue<>(100),
            new ThreadPoolExecutor.CallerRunsPolicy()
        );

        // BEST PRACTICE: Submit interruptible tasks
        Future<?> future = executor.submit(() -> {
            try {
                while (!Thread.currentThread().isInterrupted()) {
                    // Work
                    Thread.sleep(100);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });

        // BEST PRACTICE: Graceful shutdown
        executor.shutdown();
        if (!executor.awaitTermination(5, TimeUnit.SECONDS)) {
            executor.shutdownNow();
        }

        // BEST PRACTICE: Use CompletableFuture for async composition
        CompletableFuture<String> result = CompletableFuture
            .supplyAsync(() -> "Hello")
            .thenApply(s -> s + " World")
            .exceptionally(ex -> "Fallback");

        System.out.println(result.get());

        boundedPool.shutdown();
    }
}
```

**Expected Output:**
```
Hello World
```

### Real-World Cases

- **Web Servers:** Bounded thread pools with graceful shutdown.
- **Microservices:** CompletableFuture for parallel service calls.
- **Data Processing:** Virtual threads for I/O-bound data pipelines.

---

## References Links

- Processes and Threads — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/essential/concurrency/procthread.html
- Class Thread — Java Platform SE 27 API Specification - https://docs.oracle.com/en/java/javase/27/docs/api/java.base/java/lang/Thread.html
- Virtual Threads — Oracle Java Core Libraries Developer Guide - https://docs.oracle.com/en/java/javase/26/core/virtual-threads.html
- JEP 444: Virtual Threads — OpenJDK - https://openjdk.org/jeps/444
- StructuredTaskScope — JSR 396: Java SE 21 API Specification - https://cr.openjdk.org/~iris/se/21/latestSpec/apidiffs/java.base/java/util/concurrent/StructuredTaskScope.html
- Structured Concurrency — JCP - https://jcp.org
- CON02-J. Facilitate thread reuse by using Thread Pools — SEI CERT - https://wiki.sei.cmu.edu/confluence/pages/viewpage.action?pageId=88502000
- CON20-J. Do not perform operations that may block while holding a lock — SEI CERT - https://wiki.sei.cmu.edu/confluence/display/java/CON20-J.+Do+not+perform+operations+that+may+block+while+holding+a+lock
- CON31-J. Ensure that tasks submitted to a thread pool are interruptible — SEI CERT - https://wiki.sei.cmu.edu/confluence/display/java/CON31-J.+Ensure+that+tasks+submitted+to+a+thread+pool+are+interruptible
- Concurrency — CMU Software Engineering Institute - https://wiki.sei.cmu.edu/confluence/display/java/Concurrency
- Actor Model — Wikipedia - https://en.wikipedia.org/wiki/Actor_model
- Reactor Pattern — Baeldung - https://www.baeldung.com/java-reactor-pattern
- Java Concurrency in Practice — Brian Goetz et al., Addison-Wesley
- Effective Java (3rd Edition) by Joshua Bloch — Chapter 11: Concurrency