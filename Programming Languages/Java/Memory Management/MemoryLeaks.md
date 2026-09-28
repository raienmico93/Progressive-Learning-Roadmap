# Java Memory Leaks: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Core Definition

A Java memory leak is a condition in which objects that are no longer needed by the application remain reachable from GC Roots (or from long-lived objects), preventing the garbage collector from reclaiming their memory and eventually causing `OutOfMemoryError`.

### Technical Definition

In Java, a memory leak occurs when objects that are no longer used by the application are still referenced by other objects, so they cannot be garbage-collected. Unlike C/C++ where memory leaks result from forgotten `free()` calls, Java memory leaks are caused by **unintended object retention** — references held in static fields, collections, caches, listeners, ThreadLocal storage, or other long-lived structures. Because the garbage collector uses reachability analysis from GC Roots, any object reachable through a chain of references is considered "live" and cannot be reclaimed, regardless of whether the application will ever use it again.

### Beginner-Friendly Explanation

Imagine a library (the JVM heap) where books (objects) are borrowed and returned. Normally, when a reader (application code) is done with a book, they return it, and the librarian (garbage collector) can put it back on the shelf for others. A memory leak is like a reader who keeps a book checked out forever, even though they'll never read it again. The librarian can't reclaim the book because the checkout record (reference) still exists. Over time, more and more books are stuck in limbo, and eventually the library runs out of space (OutOfMemoryError).

### Key Characteristics

- **Silent**: No compile-time or runtime warning; the application continues to function.
- **Gradual**: Memory usage grows slowly, eventually causing performance degradation.
- **Non-Deterministic**: May take hours, days, or weeks to manifest.
- **Environment-Dependent**: May only appear under production load, not in testing.
- **Cumulative**: Each leak adds to the accumulated retained memory.
- **Fatal**: Eventually causes `OutOfMemoryError: Java heap space`.

### Prerequisites

- Solid understanding of Java references and object lifecycle.
- Familiarity with garbage collection and reachability analysis.
- Experience with Java collections and concurrency.
- Awareness of heap monitoring and diagnostic tools.

### Related Programming Areas

- **Garbage Collection**: Reachability, GC Roots, reference types.
- **Concurrent Programming**: ThreadLocal, thread pools, synchronization.
- **Event-Driven Programming**: Listeners, observers, callbacks.
- **Caching**: Eviction policies, TTL, LRU.
- **Diagnostics**: Heap dumps, profilers, JFR.

### Core Concepts Overview

1. **Unintended Object Retention**: Retaining references to unneeded objects prevents GC reclamation.
2. **Static References**: Static collections and fields anchor objects to the class loader lifecycle.
3. **Listener and Observer References**: Unregistered listeners remain anchored to long-lived publishers.
4. **Caches**: Unbounded caches without eviction policies grow indefinitely.
5. **Thread-Local Retention**: Values left in ThreadLocal storage leak in thread pools.
6. **Diagnostics and Tools**: Heap dumps, profilers, and JFR identify leak origins.

---

## Core Concept 1: Unintended Object Retention

### Definitions

**Core Definition**: Unintended object retention is the accidental holding of references to objects that are no longer needed, preventing the garbage collector from reclaiming their memory.

**Technical Definition**: Unintended object retention occurs when a program maintains a reference to an object beyond the object's useful lifetime. Because the garbage collector uses reachability analysis, any object reachable through a chain of references from a GC Root is considered live and cannot be reclaimed. Common causes include: (1) collections that grow without bound, (2) references in long-lived objects (e.g., caches, singletons), (3) unclosed resources, and (4) event listeners that are never deregistered. The retained objects accumulate over time, increasing heap usage until `OutOfMemoryError` occurs.

**Beginner-Friendly Explanation**: Think of a warehouse (heap) where you store boxes (objects). Every box has a label (reference) that says who owns it. When you're done with a box, you remove the label so the warehouse can reuse the space. A memory leak is like leaving labels on boxes you'll never open again — the warehouse can't reclaim the space because the label says someone still owns it. Over time, the warehouse fills up with "owned" boxes that nobody uses.

### Purposes

- To explain the fundamental mechanism of Java memory leaks.
- To identify common patterns that lead to object retention.
- To provide strategies for detecting and preventing leaks.
- To understand the relationship between reachability and leak formation.
- To establish a foundation for diagnosing specific leak categories.

### Syntax Rules and Structure

#### Complete General Syntax: Object Retention Mechanism

```
OBJECT RETENTION MECHANISM
│
├── 1. Normal Object Lifecycle
│   ├── Object created → referenced → used → dereferenced → collected
│   └── No leak: reference removed when object no longer needed
│
├── 2. Leaked Object Lifecycle
│   ├── Object created → referenced → used → still referenced (leak)
│   └── Object remains reachable from GC Root indefinitely
│
├── 3. Common Retention Patterns
│   ├── Static collections that grow without bound
│   ├── Long-lived objects holding short-lived references
│   ├── Caches without eviction policies
│   ├── Listeners never deregistered
│   └── ThreadLocal values not cleaned up
│
└── 4. Detection Strategies
    ├── Heap monitoring (jstat, JConsole)
    ├── Heap dump analysis (MAT, VisualVM)
    └── Profiling (JFR, async-profiler)
```

#### Component Breakdown

| Retention Source | Lifetime | Common Cause |
|-----------------|----------|--------------|
| Static Collections | Application lifetime | Unbounded growth |
| Caches | Application lifetime | No eviction policy |
| Listeners | Publisher lifetime | No deregistration |
| ThreadLocal | Thread lifetime | No cleanup |
| Singletons | Application lifetime | Holding references |

#### Syntax Rules

- An object is retained if it is reachable from any GC Root.
- GC Roots include thread stacks, static fields, JNI handles, and system classes.
- Retained objects cannot be garbage-collected, even if the application never uses them again.
- Leaks accumulate over time; each leak adds to the retained memory.
- `OutOfMemoryError: Java heap space` occurs when the heap cannot accommodate new allocations.

#### Constraints and Limitations

- Not all retention is a leak; long-lived objects are expected to hold references.
- Distinguishing "intentional" from "unintentional" retention requires understanding application semantics.
- Some leaks are only visible under production load patterns.
- Heap dumps capture a snapshot; multiple dumps over time are needed to identify growth.
- Some leaks are caused by third-party libraries, not application code.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Simple Memory Leak via Static Collection

**Setup Guide**: Save as `SimpleLeakDemo.java`, compile, and run with `-Xmx64m`.

```java
// SimpleLeakDemo.java
import java.util.ArrayList;
import java.util.List;

public class SimpleLeakDemo {
    
    // Static collection — leaks because it grows without bound
    // and lives for the entire application lifetime
    private static final List<byte[]> LEAK = new ArrayList<>();
    
    public static void main(String[] args) {
        System.out.println("=== Simple Memory Leak Demo ===");
        System.out.println("Max heap: 64 MB");
        System.out.println("Leaking 1 MB per iteration...\n");
        
        int iteration = 0;
        try {
            while (true) {
                // Allocate 1 MB and add to static list
                // The static list keeps the reference forever
                LEAK.add(new byte[1024 * 1024]);
                iteration++;
                
                if (iteration % 10 == 0) {
                    long used = Runtime.getRuntime().totalMemory() - 
                                Runtime.getRuntime().freeMemory();
                    System.out.println("Iteration " + iteration + 
                        " — Used: " + used / (1024 * 1024) + " MB");
                }
            }
        } catch (OutOfMemoryError e) {
            System.out.println("\n=== OutOfMemoryError! ===");
            System.out.println("Iterations completed: " + iteration);
            System.out.println("Leaked memory: ~" + iteration + " MB");
            System.out.println("\nThe static LEAK list held all references.");
        }
    }
}
```

**Expected Output** (with `-Xmx64m`):
```
=== Simple Memory Leak Demo ===
Max heap: 64 MB
Leaking 1 MB per iteration...

Iteration 10 — Used: 12 MB
Iteration 20 — Used: 22 MB
Iteration 30 — Used: 32 MB
Iteration 40 — Used: 42 MB
Iteration 50 — Used: 52 MB
Iteration 60 — Used: 62 MB

=== OutOfMemoryError! ===
Iterations completed: 63
Leaked memory: ~63 MB

The static LEAK list held all references.
```

**Why This Output**: The `LEAK` static list holds strong references to every allocated `byte[]` array. Because the list is static, it lives for the entire application lifetime. Each iteration adds a new reference, growing the list indefinitely. The garbage collector cannot reclaim the arrays because they are reachable from the static field. Eventually, the heap fills up (at ~63 MB), and `OutOfMemoryError` is thrown.

---

### Real-World Cases

- **Web Applications**: Session objects accumulate in a static `HttpSession` map without cleanup.
- **Batch Jobs**: Intermediate results are cached in a static map and never cleared.
- **Event Processing**: Event listeners accumulate in a static list without deregistration.
- **Data Processing**: Large datasets are held in static collections for "reuse" but never released.

### References

- Memory Leak - Wikipedia - https://en.wikipedia.org/wiki/Memory_leak
- Java Memory Leaks: Causes and Solutions - Baeldung - https://www.baeldung.com/java-memory-leaks
- 3 Ways to Detect Java Memory Leaks - https://www.dynatrace.com/news/blog/3-ways-to-detect-java-memory-leaks/

---

## Core Concept 2: Static References

### Definitions

**Core Definition**: Static reference leaks occur when objects are stored in static fields or static collections, bounding their lifecycle to the root class loader's lifetime instead of a temporary scope.

**Technical Definition**: Static fields are associated with the `Class` object of their declaring class and are stored in the method area (Metaspace). Because the root class loader (bootstrap or system class loader) lives for the entire JVM lifetime, static fields are effectively GC Roots. Any object referenced by a static field is considered strongly reachable and cannot be garbage-collected as long as the class is loaded. When static collections (e.g., `static List`, `static Map`) are used to store objects that are no longer needed, those objects are retained indefinitely, causing a memory leak.

**Beginner-Friendly Explanation**: Think of static fields as "permanent storage" in the JVM. Anything you put in a static field stays there until the JVM shuts down. If you use a static list to store objects you think are temporary, they're actually permanent — the list never goes out of scope. This is a common mistake: developers treat static collections as temporary because they're used in a method, but static means "belongs to the class," which outlives any method call.

### Purposes

- To understand why static fields are GC Roots.
- To identify patterns where static collections cause leaks.
- To provide strategies for managing static collection lifecycles.
- To distinguish intentional static state from accidental retention.
- To recognize when static references become leak sources.

### Syntax Rules and Structure

#### Complete General Syntax: Static Reference Lifecycle

```
STATIC REFERENCE LIFECYCLE
│
├── Class Loading
│   └── Static field allocated in Metaspace
│
├── Static Field Initialization
│   └── Static collection created (e.g., static List)
│
├── Object Addition
│   └── Objects added to static collection
│
├── Object Retention
│   └── Static field holds reference → object cannot be GC'd
│
└── Class Unloading
    └── Only if class loader is GC'd (rare for system classes)
```

#### Component Breakdown

| Static Structure | Typical Leak Pattern | Fix |
|-----------------|---------------------|-----|
| `static List` | Objects added but never removed | Use bounded list or clear regularly |
| `static Map` | Keys/values accumulate | Use WeakHashMap or evict |
| `static Set` | Duplicates accumulate | Use bounded set or clear |
| `static` singleton | Holds references to short-lived objects | Use weak references or clear state |

#### Syntax Rules

- Static fields are initialized during class initialization (`<clinit>`).
- Static fields are stored in the method area (Metaspace).
- Static fields are GC Roots; objects reachable from them are never collected.
- Static fields live as long as the class is loaded.
- System classes (bootstrap-loaded) are never unloaded.
- Custom class loaders can be unloaded, releasing their static fields.

#### Constraints and Limitations

- Not all static fields are leaks; constants and configuration are expected.
- Static collections can be legitimate (e.g., application-wide registries).
- The distinction between "intentional" and "unintentional" is application-specific.
- Class unloading requires the class loader to be unreachable.
- Static references are difficult to test because leaks manifest only under load.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Static List Leak

**Setup Guide**: Save as `StaticLeakDemo.java`, compile, and run with `-Xmx64m`.

```java
// StaticLeakDemo.java
import java.util.ArrayList;
import java.util.List;

public class StaticLeakDemo {
    
    // Static list — lives for the entire application lifetime
    private static final List<String> CACHE = new ArrayList<>();
    
    // Method that adds to the static cache but never removes
    static void processRequest(int requestId) {
        // Simulate processing — store request data in static cache
        // This data is only needed during the request but is stored permanently
        CACHE.add("Request " + requestId + " data: " + 
            new byte[1024]); // 1 KB per entry
        
        // Intended: cache should be cleared after request
        // Actual: cache grows indefinitely
    }
    
    public static void main(String[] args) {
        System.out.println("=== Static Reference Leak Demo ===");
        System.out.println("Static cache grows without bound.\n");
        
        try {
            for (int i = 1; i <= 10_000; i++) {
                processRequest(i);
                
                if (i % 1000 == 0) {
                    long used = Runtime.getRuntime().totalMemory() - 
                                Runtime.getRuntime().freeMemory();
                    System.out.println("Requests: " + i + 
                        " — Cache size: " + CACHE.size() + 
                        " — Used: " + used / 1024 + " KB");
                }
            }
        } catch (OutOfMemoryError e) {
            System.out.println("\n=== OutOfMemoryError! ===");
            System.out.println("Cache size at failure: " + CACHE.size());
            System.out.println("The static CACHE list retained all entries.");
        }
    }
}
```

**Expected Output** (with `-Xmx64m`):
```
=== Static Reference Leak Demo ===
Static cache grows without bound.

Requests: 1000 — Cache size: 1000 — Used: 2048 KB
Requests: 2000 — Cache size: 2000 — Used: 4096 KB
...
Requests: 50000 — Cache size: 50000 — Used: 51200 KB

=== OutOfMemoryError! ===
Cache size at failure: 63234
The static CACHE list retained all entries.
```

**Why This Output**: The `CACHE` static list holds every string added by `processRequest`. Although the request data is only needed during processing, it's stored permanently in the static list. The list grows indefinitely, consuming heap memory until `OutOfMemoryError` occurs. The fix would be to use a local variable for request data, or to clear the cache after each request.

---

### Real-World Cases

- **Session Management**: Web applications store sessions in a static map without expiration.
- **Configuration Registries**: Static maps hold configuration objects that are reloaded but not removed.
- **Metrics Collection**: Static lists accumulate metric samples without bounds.
- **Connection Pools**: Static pools hold connections that are never released.
- **Class Caches**: Static maps cache class metadata without eviction.

### References

- Java Static Fields and Memory Leaks - Baeldung - https://www.baeldung.com/java-memory-leaks
- Understanding Static References in Java - https://www.baeldung.com/java-static
- 3 Ways to Detect Java Memory Leaks - Dynatrace - https://www.dynatrace.com/news/blog/3-ways-to-detect-java-memory-leaks/

---

## Core Concept 3: Listener and Observer References

### Definitions

**Core Definition**: Listener and observer reference leaks occur when UI components or data callbacks are registered with long-lived publishers but never explicitly unregistered, leaving dead objects anchored to the publisher's lifecycle.

**Technical Definition**: The observer pattern is widely used in Java for event handling. Listeners are registered with a publisher (e.g., `EventSource`, `Observable`, UI components), which maintains a collection of listener references. If a listener is not explicitly removed via `removeListener()` or `unregister()`, the publisher holds a strong reference to it indefinitely. Since publishers are often long-lived (singletons, application-scoped components), the listener remains reachable from a GC Root and cannot be garbage-collected. This is particularly problematic in UI frameworks (Swing, JavaFX) and event-driven systems.

**Beginner-Friendly Explanation**: Think of subscribing to a magazine. You sign up (register as a listener), and the magazine company (publisher) keeps your address (reference) on file. When you move or lose interest, you should cancel your subscription (unregister). If you don't, the company keeps sending magazines to your old address, and your mailbox fills up. In Java, the "mailbox" is memory: the publisher keeps referencing the listener, preventing it from being garbage-collected.

### Purposes

- To understand why unregistered listeners cause memory leaks.
- To identify listener registration patterns that lack deregistration.
- To provide strategies for managing listener lifecycles.
- To recognize the role of long-lived publishers in leak formation.
- To establish best practices for event-driven architectures.

### Syntax Rules and Structure

#### Complete General Syntax: Listener Leak Mechanism

```
LISTENER LEAK MECHANISM
│
├── 1. Listener Registration
│   ├── publisher.addListener(listener)
│   └── Publisher stores strong reference to listener
│
├── 2. Normal Usage
│   ├── Events published → listener invoked
│   └── Listener processes events
│
├── 3. Listener No Longer Needed
│   ├── Component disposed or destroyed
│   └── Listener should be unregistered
│
├── 4. Leak Condition (no unregistration)
│   ├── Publisher still holds reference
│   ├── Listener cannot be GC'd
│   └── Memory leak accumulates
│
└── 5. Fix: Unregistration
    ├── publisher.removeListener(listener)
    └── Publisher releases reference → listener collected
```

#### Component Breakdown

| Pattern | Registration | Unregistration | Leak Risk |
|---------|-------------|----------------|-----------|
| Swing | `addActionListener()` | `removeActionListener()` | High |
| JavaFX | `addListener()` | `removeListener()` | High |
| PropertyChange | `addPropertyChangeListener()` | `removePropertyChangeListener()` | High |
| Custom Observer | `addObserver()` | `deleteObserver()` | High |
| Spring Events | `@EventListener` | Automatic (context lifecycle) | Low |

#### Syntax Rules

- Listeners must be explicitly unregistered when no longer needed.
- Publishers typically hold strong references to listeners.
- Use weak references for listeners when the publisher outlives the listener.
- UI frameworks often provide `dispose()` or `destroy()` methods that unregister listeners.
- Spring's `@EventListener` is automatically deregistered when the bean is destroyed.
- Lambda-based listeners are especially prone to leaks because they capture enclosing references.

#### Constraints and Limitations

- The distinction between "long-lived" and "short-lived" is application-specific.
- Some frameworks automatically manage listener lifecycles.
- Weak listener references require careful handling (the listener must be strongly referenced elsewhere).
- Detecting listener leaks requires heap dump analysis to identify the publisher-listener chain.
- Not all listeners need explicit unregistration (e.g., application-scoped listeners).

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Listener Leak Demonstration

**Setup Guide**: Save as `ListenerLeakDemo.java`, compile, and run with `-Xmx64m`.

```java
// ListenerLeakDemo.java
import java.util.ArrayList;
import java.util.List;

public class ListenerLeakDemo {
    
    // Publisher — long-lived (singleton-like)
    static class EventPublisher {
        private final List<EventListener> listeners = new ArrayList<>();
        
        void addListener(EventListener listener) {
            listeners.add(listener);
        }
        
        // Missing: removeListener() method
        
        void publishEvent(String event) {
            for (EventListener listener : listeners) {
                listener.onEvent(event);
            }
        }
        
        int listenerCount() {
            return listeners.size();
        }
    }
    
    // Listener — should be short-lived
    static class EventListener {
        private final String name;
        private final byte[] data = new byte[1024 * 100]; // 100 KB
        
        EventListener(String name) { this.name = name; }
        
        void onEvent(String event) {
            // Process event
        }
    }
    
    // Long-lived publisher
    private static final EventPublisher PUBLISHER = new EventPublisher();
    
    public static void main(String[] args) {
        System.out.println("=== Listener Leak Demo ===");
        System.out.println("Listeners registered but never unregistered.\n");
        
        try {
            for (int i = 1; i <= 1000; i++) {
                // Create listener, register it, never unregister
                EventListener listener = new EventListener("Listener-" + i);
                PUBLISHER.addListener(listener);
                
                // Listener is no longer needed after this iteration
                // But publisher still holds a reference
                
                if (i % 100 == 0) {
                    System.out.println("Listeners registered: " + 
                        PUBLISHER.listenerCount());
                }
            }
        } catch (OutOfMemoryError e) {
            System.out.println("\n=== OutOfMemoryError! ===");
            System.out.println("Listeners retained: " + 
                PUBLISHER.listenerCount());
            System.out.println("The publisher held all listener references.");
        }
    }
}
```

**Expected Output**:
```
=== Listener Leak Demo ===
Listeners registered but never unregistered.

Listeners registered: 100
Listeners registered: 200
...
=== OutOfMemoryError! ===
Listeners retained: 631
The publisher held all listener references.
```

**Why This Output**: Each `EventListener` is registered with the `PUBLISHER` but never unregistered. The publisher's `listeners` list holds strong references to every listener, so none can be garbage-collected. Each listener holds 100 KB of data, so after ~600 registrations, the heap (64 MB) fills up. The fix is to provide a `removeListener()` method and call it when the listener is no longer needed.

---

### Real-World Cases

- **Swing Applications**: Components register listeners with parent containers; forgetting to remove them causes leaks when components are disposed.
- **JavaFX Applications**: Property listeners accumulate if not removed when nodes are removed from the scene graph.
- **Android Applications**: Activities register listeners with system services; forgetting to unregister causes leaks.
- **Web Applications**: HTTP session listeners accumulate if not removed when sessions expire.
- **Event Bus**: Custom event buses accumulate subscribers if not deregistered.

### References

- Memory Leak in Java - Baeldung - https://www.baeldung.com/java-memory-leaks
- Listener Leaks in Java - https://www.baeldung.com/java-memory-leaks
- 3 Ways to Detect Java Memory Leaks - Dynatrace - https://www.dynatrace.com/news/blog/3-ways-to-detect-java-memory-leaks/

---

## Core Concept 4: Caches

### Definitions

**Core Definition**: Cache memory leaks occur when internal caching structures (e.g., `HashMap`) grow without bounds, lack expiration policies (TTL), or lack eviction strategies (LRU), causing cached objects to accumulate indefinitely.

**Technical Definition**: Caching is a common performance optimization that stores frequently accessed data in memory. However, unbounded caches or caches without eviction policies cause memory leaks because cached entries remain referenced indefinitely. A proper cache implementation requires: (1) **maximum size limit** (bounded cache), (2) **eviction policy** (LRU, LFU, FIFO), (3) **time-to-live (TTL)** for entries, and (4) **time-to-idle (TTI)** for idle entries. The `java.util.HashMap` used as a cache without these policies is a common leak source. Libraries like Caffeine, Guava Cache, and Ehcache provide production-ready cache implementations with these features.

**Beginner-Friendly Explanation**: Think of a cache as a refrigerator. You put food (data) in to keep it fresh and accessible. But if you never clean out the refrigerator, old food accumulates, taking up space and eventually preventing you from storing new food. A good cache has an expiration date (TTL) and a maximum capacity (LRU eviction), just like a well-managed refrigerator. Using a `HashMap` as a cache without these policies is like a refrigerator that never gets cleaned.

### Purposes

- To understand why unbounded caches cause memory leaks.
- To identify cache implementations lacking eviction policies.
- To provide strategies for bounded, evicting caches.
- To recognize the role of TTL and TTI in cache management.
- To establish best practices for caching in Java.

### Syntax Rules and Structure

#### Complete General Syntax: Cache Leak vs. Bounded Cache

```
CACHE LEAK VS. BOUNDED CACHE
│
├── LEAK: Unbounded HashMap Cache
│   ├── Map<String, Data> cache = new HashMap<>();
│   ├── cache.put(key, value) — no size limit
│   ├── Entries never removed
│   └── Grows until OutOfMemoryError
│
├── BOUNDED: LRU Cache with Eviction
│   ├── Maximum size limit (e.g., 10,000 entries)
│   ├── LRU eviction when full
│   ├── TTL for entries (e.g., 5 minutes)
│   └── Automatic cleanup
│
└── LIBRARIES
    ├── Caffeine: High-performance, near-optimal hit rate
    ├── Guava Cache: Simple, well-documented
    └── Ehcache: Enterprise-grade, disk overflow
```

#### Component Breakdown

| Cache Property | Purpose | Example |
|---------------|---------|---------|
| Maximum Size | Bound memory usage | 10,000 entries |
| Eviction Policy | Remove entries when full | LRU, LFU, FIFO |
| TTL | Expire entries after time | 5 minutes |
| TTI | Expire idle entries | 1 minute |
| Weak/Soft Keys | Allow GC of keys | `WeakHashMap` |

#### Syntax Rules

- `HashMap` used as a cache without bounds is a leak risk.
- Use `LinkedHashMap` with `removeEldestEntry()` for simple LRU.
- Use `WeakHashMap` for caches where keys can be reclaimed.
- Use Caffeine or Guava Cache for production-grade caching.
- Set `maximumSize()` to bound cache size.
- Set `expireAfterWrite()` for TTL.
- Set `expireAfterAccess()` for TTI.

#### Constraints and Limitations

- Even bounded caches can leak if keys are never evicted (e.g., unique keys).
- TTL and TTI require a background thread or maintenance operations.
- Weak/soft reference caches may clear entries unexpectedly.
- Cache size tuning requires understanding of working set size.
- Distributed caches (Redis, Hazelcast) have different leak characteristics.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Unbounded Cache Leak

**Setup Guide**: Save as `CacheLeakDemo.java`, compile, and run with `-Xmx64m`.

```java
// CacheLeakDemo.java
import java.util.HashMap;
import java.util.Map;

public class CacheLeakDemo {
    
    // Unbounded cache — leaks because entries are never removed
    private static final Map<String, byte[]> CACHE = new HashMap<>();
    
    static byte[] getData(String key) {
        // Check cache
        byte[] cached = CACHE.get(key);
        if (cached != null) {
            return cached;
        }
        
        // Cache miss — compute and store
        byte[] data = new byte[1024 * 10]; // 10 KB
        CACHE.put(key, data);
        return data;
    }
    
    public static void main(String[] args) {
        System.out.println("=== Unbounded Cache Leak Demo ===");
        System.out.println("Cache grows without bound.\n");
        
        try {
            for (int i = 1; i <= 10_000; i++) {
                // Unique key each time — cache never hits
                getData("key-" + i);
                
                if (i % 1000 == 0) {
                    System.out.println("Keys: " + i + 
                        " — Cache size: " + CACHE.size() +
                        " — Used: " + 
                        (Runtime.getRuntime().totalMemory() - 
                         Runtime.getRuntime().freeMemory()) / 1024 + " KB");
                }
            }
        } catch (OutOfMemoryError e) {
            System.out.println("\n=== OutOfMemoryError! ===");
            System.out.println("Cache size: " + CACHE.size());
            System.out.println("The HashMap grew without bound.");
        }
    }
}
```

**Expected Output**:
```
=== Unbounded Cache Leak Demo ===
Cache grows without bound.

Keys: 1000 — Cache size: 1000 — Used: 10240 KB
Keys: 2000 — Cache size: 2000 — Used: 20480 KB
...
=== OutOfMemoryError! ===
Cache size: 6323
The HashMap grew without bound.
```

**Why This Output**: The `CACHE` `HashMap` stores every unique key-value pair. Since keys are unique (never repeated), every call to `getData` adds a new entry. The cache never evicts, so it grows until the heap fills up. The fix is to use a bounded cache (e.g., Caffeine with `maximumSize(1000)`), which would evict older entries when the limit is reached.

---

#### Example 2: Bounded Cache with Caffeine

**Setup Guide**: This example requires the Caffeine library. Save as `BoundedCacheDemo.java`, add Caffeine to the classpath, compile, and run.

```java
// BoundedCacheDemo.java
import com.github.benmanes.caffeine.cache.Cache;
import com.github.benmanes.caffeine.cache.Caffeine;
import java.util.concurrent.TimeUnit;

public class BoundedCacheDemo {
    
    // Bounded cache with maximum size and TTL
    private static final Cache<String, byte[]> CACHE = 
        Caffeine.newBuilder()
            .maximumSize(1000)                    // Max 1000 entries
            .expireAfterWrite(5, TimeUnit.MINUTES) // TTL: 5 minutes
            .build();
    
    static byte[] getData(String key) {
        return CACHE.get(key, k -> new byte[1024 * 10]); // 10 KB
    }
    
    public static void main(String[] args) throws Exception {
        System.out.println("=== Bounded Cache Demo ===");
        System.out.println("Caffeine cache with max 1000 entries, 5-min TTL.\n");
        
        for (int i = 1; i <= 10_000; i++) {
            getData("key-" + i);
            
            if (i % 1000 == 0) {
                // Force cleanup of expired entries
                CACHE.cleanUp();
                System.out.println("Keys: " + i + 
                    " — Cache size: " + CACHE.estimatedSize());
            }
        }
        
        System.out.println("\nCache size bounded at ~1000 entries.");
        System.out.println("No OutOfMemoryError — entries evicted automatically.");
    }
}
```

**Expected Output**:
```
=== Bounded Cache Demo ===
Caffeine cache with max 1000 entries, 5-min TTL.

Keys: 1000 — Cache size: 1000
Keys: 2000 — Cache size: 1000
Keys: 3000 — Cache size: 1000
...
Keys: 10000 — Cache size: 1000

Cache size bounded at ~1000 entries.
No OutOfMemoryError — entries evicted automatically.
```

**Why This Output**: The Caffeine cache is configured with `maximumSize(1000)` and `expireAfterWrite(5, TimeUnit.MINUTES)`. When the cache reaches 1000 entries, new entries cause eviction of the least-recently-used entry (LRU policy). The cache size remains bounded at ~1000, preventing the memory leak. The `cleanUp()` call forces removal of expired entries, but even without it, Caffeine performs maintenance automatically.

---

### Real-World Cases

- **Web Applications**: HTTP response caches must be bounded to prevent memory exhaustion.
- **Database Query Caches**: Query result caches must have TTL and size limits.
- **Image Caches**: Image caches in UI applications must be bounded (e.g., 100 images).
- **Session Caches**: Session data caches must expire inactive sessions.
- **Configuration Caches**: Configuration caches must be invalidated when configuration changes.

### References

- Caffeine - https://github.com/ben-manes/caffeine
- Guava Cache - https://github.com/google/guava/wiki/CachesExplained
- Java Caching Best Practices - Baeldung - https://www.baeldung.com/java-caching

---

## Core Concept 5: Thread-Local Retention

### Definitions

**Core Definition**: Thread-local retention leaks occur when values are left in `ThreadLocal` storage after the execution cycle completes, causing severe leaks when threads are recycled inside reusable thread pools.

**Technical Definition**: `ThreadLocal` provides thread-local variables: each thread has its own independently initialized copy of the variable. Internally, each `Thread` object maintains a `ThreadLocalMap` that maps `ThreadLocal` instances to values. The `ThreadLocalMap` uses weak references for keys (the `ThreadLocal` instances) but strong references for values. When a `ThreadLocal` is no longer referenced, its key becomes eligible for GC, but the value remains strongly referenced by the thread's `ThreadLocalMap` until the thread itself is garbage-collected. In thread pools, threads are reused indefinitely, so values left in `ThreadLocal` storage leak. The fix is to always call `ThreadLocal.remove()` in a finally block.

**Beginner-Friendly Explanation**: Think of `ThreadLocal` as a personal locker assigned to each worker (thread). Each worker can store their tools (values) in their locker. When a worker finishes a task, they should clean out their locker for the next worker. But if workers are recycled (thread pools), the next worker inherits the previous worker's tools. If those tools are never cleaned, they accumulate. Worse, even if the locker key (ThreadLocal) is thrown away, the tools inside remain until the locker itself is destroyed (thread dies).

### Purposes

- To understand why ThreadLocal values leak in thread pools.
- To identify patterns where ThreadLocal is not cleaned up.
- To provide strategies for safe ThreadLocal usage.
- To recognize the interaction between ThreadLocal and thread pools.
- To establish best practices for ThreadLocal-based context propagation.

### Syntax Rules and Structure

#### Complete General Syntax: ThreadLocal Leak Mechanism

```
THREADLOCAL LEAK MECHANISM
│
├── 1. ThreadLocal Creation
│   └── ThreadLocal<Context> tl = new ThreadLocal<>();
│
├── 2. Value Storage
│   ├── tl.set(context) — stores value in thread's ThreadLocalMap
│   └── ThreadLocalMap entry: (weak key: tl, strong value: context)
│
├── 3. Thread Reuse (Thread Pool)
│   ├── Thread returns to pool
│   ├── ThreadLocalMap still holds value
│   └── Next task inherits stale value
│
├── 4. Leak Condition
│   ├── ThreadLocal key becomes unreachable
│   ├── Key is weakly referenced → eligible for GC
│   ├── Value remains strongly referenced by ThreadLocalMap
│   └── Value leaks until thread dies
│
└── 5. Fix: ThreadLocal.remove()
    ├── try { tl.set(value); ... } finally { tl.remove(); }
    └── Value removed from ThreadLocalMap
```

#### Component Breakdown

| Component | Reference Strength | Lifetime |
|-----------|-------------------|----------|
| ThreadLocal (key) | Weak | Until ThreadLocal is unreachable |
| Value | Strong | Until ThreadLocalMap entry is removed or thread dies |
| ThreadLocalMap | Strong (thread → map) | Thread lifetime |

#### Syntax Rules

- Always call `ThreadLocal.remove()` in a `finally` block.
- Use `try-finally` or `try-with-resources` for ThreadLocal cleanup.
- Thread pools reuse threads, so ThreadLocal values persist across tasks.
- The `ThreadLocalMap` key is weak; the value is strong.
- `InheritableThreadLocal` propagates values to child threads (also requires cleanup).
- Use `ThreadLocal.withInitial()` for default values (but still requires cleanup).

#### Constraints and Limitations

- `ThreadLocal.remove()` must be called explicitly; there is no automatic cleanup.
- The weak key means the ThreadLocal object can be GC'd, but the value remains.
- Long-lived thread pools (e.g., web servers) accumulate leaked values indefinitely.
- `ThreadLocal` is not suitable for passing data between threads.
- `InheritableThreadLocal` has additional complexity with thread pools.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: ThreadLocal Leak in Thread Pool

**Setup Guide**: Save as `ThreadLocalLeakDemo.java`, compile, and run with `-Xmx64m`.

```java
// ThreadLocalLeakDemo.java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class ThreadLocalLeakDemo {
    
    // ThreadLocal without cleanup — leaks in thread pools
    private static final ThreadLocal<byte[]> CONTEXT = new ThreadLocal<>();
    
    static void processTask(int taskId) {
        // Set ThreadLocal value
        CONTEXT.set(new byte[1024 * 100]); // 100 KB
        
        // Process task...
        
        // Missing: CONTEXT.remove()
        // The value remains in the thread's ThreadLocalMap
    }
    
    public static void main(String[] args) throws Exception {
        System.out.println("=== ThreadLocal Leak Demo ===");
        System.out.println("Thread pool reuses threads; ThreadLocal not cleaned.\n");
        
        // Fixed thread pool — threads are reused
        ExecutorService executor = Executors.newFixedThreadPool(4);
        
        try {
            for (int i = 1; i <= 10_000; i++) {
                final int taskId = i;
                executor.submit(() -> processTask(taskId));
                
                if (i % 1000 == 0) {
                    long used = Runtime.getRuntime().totalMemory() - 
                                Runtime.getRuntime().freeMemory();
                    System.out.println("Tasks: " + i + 
                        " — Used: " + used / 1024 + " KB");
                }
            }
        } catch (OutOfMemoryError e) {
            System.out.println("\n=== OutOfMemoryError! ===");
            System.out.println("ThreadLocal values leaked in thread pool.");
        } finally {
            executor.shutdown();
        }
    }
}
```

**Expected Output**:
```
=== ThreadLocal Leak Demo ===
Thread pool reuses threads; ThreadLocal not cleaned.

Tasks: 1000 — Used: 10240 KB
Tasks: 2000 — Used: 20480 KB
...
=== OutOfMemoryError! ===
ThreadLocal values leaked in thread pool.
```

**Why This Output**: Each task calls `CONTEXT.set(new byte[100 KB])` but never calls `CONTEXT.remove()`. Because the thread pool reuses the same 4 threads, each thread's `ThreadLocalMap` accumulates values. Since each thread can only hold one value per ThreadLocal, the leak is bounded by the number of threads (4 × 100 KB = 400 KB). However, if the ThreadLocal itself becomes unreachable (e.g., in a web application redeploy), the value leaks until the thread dies. The fix is to call `CONTEXT.remove()` in a `finally` block.

---

#### Example 2: Correct ThreadLocal Cleanup

**Setup Guide**: Save as `ThreadLocalFixDemo.java`, compile, and run.

```java
// ThreadLocalFixDemo.java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class ThreadLocalFixDemo {
    
    private static final ThreadLocal<byte[]> CONTEXT = new ThreadLocal<>();
    
    static void processTask(int taskId) {
        try {
            // Set ThreadLocal value
            CONTEXT.set(new byte[1024 * 100]); // 100 KB
            
            // Process task...
            
        } finally {
            // ALWAYS remove ThreadLocal value
            CONTEXT.remove();
        }
    }
    
    public static void main(String[] args) throws Exception {
        System.out.println("=== ThreadLocal Cleanup Demo ===");
        System.out.println("ThreadLocal removed in finally block.\n");
        
        ExecutorService executor = Executors.newFixedThreadPool(4);
        
        for (int i = 1; i <= 10_000; i++) {
            final int taskId = i;
            executor.submit(() -> processTask(taskId));
            
            if (i % 2000 == 0) {
                long used = Runtime.getRuntime().totalMemory() - 
                            Runtime.getRuntime().freeMemory();
                System.out.println("Tasks: " + i + 
                    " — Used: " + used / 1024 + " KB");
            }
        }
        
        executor.shutdown();
        executor.awaitTermination(10, java.util.concurrent.TimeUnit.SECONDS);
        
        System.out.println("\nNo OutOfMemoryError.");
        System.out.println("ThreadLocal values cleaned up after each task.");
    }
}
```

**Expected Output**:
```
=== ThreadLocal Cleanup Demo ===
ThreadLocal removed in finally block.

Tasks: 2000 — Used: 2048 KB
Tasks: 4000 — Used: 2048 KB
Tasks: 6000 — Used: 2048 KB
Tasks: 8000 — Used: 2048 KB
Tasks: 10000 — Used: 2048 KB

No OutOfMemoryError.
ThreadLocal values cleaned up after each task.
```

**Why This Output**: The `processTask` method now calls `CONTEXT.remove()` in a `finally` block, ensuring the ThreadLocal value is always cleaned up, even if an exception occurs. The memory usage remains stable at ~2 MB (baseline JVM memory) throughout all 10,000 tasks. This demonstrates the correct pattern for ThreadLocal usage in thread pools.

---

### Real-World Cases

- **Web Applications**: Request context (user, transaction) stored in ThreadLocal must be cleaned after each request.
- **Database Connection Management**: Transaction contexts in ThreadLocal must be removed after commit/rollback.
- **Security Contexts**: User authentication contexts in ThreadLocal must be cleared after request processing.
- **Logging MDC**: Mapped Diagnostic Context (MDC) in logging frameworks uses ThreadLocal and must be cleaned.
- **Spring Framework**: `RequestContextHolder` uses ThreadLocal and requires cleanup.

### References

- ThreadLocal - Java SE 23 - https://docs.oracle.com/en/java/javase/23/docs/api/java.base/java/lang/ThreadLocal.html
- ThreadLocal Memory Leak in Java - Baeldung - https://www.baeldung.com/java-threadlocal-memory-leak
- Java ThreadLocal - https://www.baeldung.com/java-threadlocal

---

## Core Concept 6: Diagnostics and Tools

### Definitions

**Core Definition**: Memory leak diagnostics is the practice of profiling memory footprints and identifying leak origins using heap dump analysis tools (Eclipse Memory Analyzer / MAT), monitoring tools (VisualVM, JConsole), and programmatic triggers (JDK Flight Recorder / JFR).

**Technical Definition**: The JDK provides multiple tools for memory leak diagnosis: **jps** (JVM process status), **jstat** (JVM statistics), **jmap** (memory map and heap dump), **jcmd** (diagnostic commands), **JConsole** (JMX-based monitoring), **VisualVM** (all-in-one profiling), **Eclipse Memory Analyzer (MAT)** (heap dump analysis), and **JDK Flight Recorder (JFR)** (low-overhead event recording). Heap dumps capture a snapshot of the heap at a point in time; comparing multiple dumps reveals objects that grow over time. MAT's dominator tree and leak suspects report identify the largest retained object sets. JFR records allocation events, GC events, and object statistics for later analysis.

**Beginner-Friendly Explanation**: Memory leak diagnostics is like being a detective. You have several tools: a camera that takes a snapshot of the crime scene (heap dump), a video recorder that captures everything that happens (JFR), and a magnifying glass that helps you examine the evidence (MAT). By comparing snapshots over time, you can identify which objects are accumulating and trace them back to their source (the leak).

### Purposes

- To identify objects that accumulate over time (leak suspects).
- To trace leak origins to specific classes, collections, or references.
- To capture heap dumps for post-mortem analysis.
- To monitor memory usage in real time.
- To record allocation and GC events with low overhead.
- To compare heap snapshots to identify growth patterns.

### Syntax Rules and Structure

#### Complete General Syntax: Diagnostics Toolchain

```
MEMORY LEAK DIAGNOSTICS TOOLCHAIN
│
├── 1. MONITORING (Real-Time)
│   ├── jstat -gc <pid> <interval>
│   ├── JConsole (JMX-based GUI)
│   └── VisualVM (all-in-one profiler)
│
├── 2. HEAP DUMP CAPTURE
│   ├── jmap -dump:live,format=b,file=heap.hprof <pid>
│   ├── jcmd <pid> GC.heap_dump heap.hprof
│   └── -XX:+HeapDumpOnOutOfMemoryError
│
├── 3. HEAP DUMP ANALYSIS
│   ├── Eclipse Memory Analyzer (MAT)
│   ├── VisualVM Heap Viewer
│   └── jhat (deprecated)
│
├── 4. EVENT RECORDING
│   ├── JFR: -XX:StartFlightRecording
│   ├── jcmd <pid> JFR.start
│   └── JDK Mission Control (JMC)
│
└── 5. PROGRAMMATIC TRIGGERS
    ├── MemoryMXBean for heap usage
    ├── NotificationListener for GC events
    └── Custom leak detectors
```

#### Component Breakdown

| Tool | Purpose | Output |
|------|---------|--------|
| jstat | Real-time GC statistics | Tabular |
| jmap | Heap dump capture | Binary (.hprof) |
| jcmd | Unified diagnostic commands | Text/JSON |
| JConsole | JMX monitoring GUI | Graphical |
| VisualVM | Profiling and monitoring | Graphical |
| MAT | Heap dump analysis | Graphical |
| JFR | Event recording | Binary (.jfr) |
| JMC | JFR analysis | Graphical |

#### Syntax Rules

- Heap dumps are captured with `jmap -dump` or `jcmd GC.heap_dump`.
- `-XX:+HeapDumpOnOutOfMemoryError` automatically captures a dump on OOM.
- MAT analyzes `.hprof` files; use the "Leak Suspects" report for quick diagnosis.
- JFR is enabled with `-XX:StartFlightRecording` or `jcmd JFR.start`.
- JFR recordings are analyzed with JDK Mission Control (JMC).
- `jstat -gc <pid> 1000` prints GC statistics every second.
- VisualVM can capture heap dumps and perform CPU/memory profiling.

#### Constraints and Limitations

- Heap dumps are large (can be several GB) and impact application performance.
- `jmap -dump:live` triggers a Full GC, causing a pause.
- MAT requires significant memory to analyze large heap dumps.
- JFR has low overhead but requires JDK 11+ for production use (JDK 8 requires commercial license).
- Real-time monitoring tools may miss short-lived leaks.
- Identifying the exact leak source requires understanding application semantics.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Programmatic Leak Detection with MemoryMXBean

**Setup Guide**: Save as `LeakDetector.java`, compile, and run.

```java
// LeakDetector.java
import java.lang.management.ManagementFactory;
import java.lang.management.MemoryMXBean;
import java.lang.management.MemoryUsage;

public class LeakDetector {
    
    private static final MemoryMXBean MEMORY_BEAN = 
        ManagementFactory.getMemoryMXBean();
    
    // Threshold: alert if heap usage grows by more than 10 MB
    private static final long GROWTH_THRESHOLD = 10 * 1024 * 1024;
    
    private static long lastUsed = 0;
    
    static void checkForLeak(String checkpoint) {
        MemoryUsage heap = MEMORY_BEAN.getHeapMemoryUsage();
        long used = heap.getUsed();
        
        if (lastUsed > 0) {
            long growth = used - lastUsed;
            if (growth > GROWTH_THRESHOLD) {
                System.out.println("⚠ LEAK ALERT at " + checkpoint + 
                    ": heap grew by " + growth / (1024 * 1024) + " MB");
            }
        }
        
        System.out.println(checkpoint + ": Used = " + 
            used / (1024 * 1024) + " MB, Max = " + 
            heap.getMax() / (1024 * 1024) + " MB");
        
        lastUsed = used;
    }
    
    public static void main(String[] args) {
        System.out.println("=== Programmatic Leak Detector ===");
        
        // Simulate a leaky application
        java.util.List<byte[]> leak = new java.util.ArrayList<>();
        
        checkForLeak("Startup");
        
        for (int i = 1; i <= 5; i++) {
            // Allocate 5 MB per cycle
            for (int j = 0; j < 5; j++) {
                leak.add(new byte[1024 * 1024]);
            }
            checkForLeak("Cycle " + i);
        }
        
        System.out.println("\nLeak detector identified heap growth.");
    }
}
```

**Expected Output**:
```
=== Programmatic Leak Detector ===
Startup: Used = 2 MB, Max = 4096 MB
Cycle 1: Used = 7 MB, Max = 4096 MB
⚠ LEAK ALERT at Cycle 2: heap grew by 10 MB
Cycle 2: Used = 12 MB, Max = 4096 MB
⚠ LEAK ALERT at Cycle 3: heap grew by 10 MB
Cycle 3: Used = 17 MB, Max = 4096 MB
...

Leak detector identified heap growth.
```

**Why This Output**: The `MemoryMXBean` provides real-time heap usage statistics. The `checkForLeak` method compares the current heap usage to the previous checkpoint. When growth exceeds the threshold (10 MB), it prints a leak alert. This demonstrates how to build a simple programmatic leak detector using JMX.

---

#### Example 2: Heap Dump Capture and Analysis

**Setup Guide**: Save as `HeapDumpDemo.java`, compile, and run with `-XX:+HeapDumpOnOutOfMemoryError -Xmx64m`.

```java
// HeapDumpDemo.java
import java.util.ArrayList;
import java.util.List;

public class HeapDumpDemo {
    
    // Static leak — will be visible in heap dump
    private static final List<byte[]> LEAK = new ArrayList<>();
    
    public static void main(String[] args) {
        System.out.println("=== Heap Dump Demo ===");
        System.out.println("PID: " + ProcessHandle.current().pid());
        System.out.println("Run with: -XX:+HeapDumpOnOutOfMemoryError -Xmx64m");
        System.out.println("Heap dump captured on OOM.\n");
        
        try {
            while (true) {
                LEAK.add(new byte[1024 * 1024]); // 1 MB
            }
        } catch (OutOfMemoryError e) {
            System.out.println("OutOfMemoryError caught!");
            System.out.println("Heap dump should be in current directory.");
            System.out.println("Analyze with: mat heap dump file");
        }
    }
}
```

**Expected Output**:
```
=== Heap Dump Demo ===
PID: 12345
Run with: -XX:+HeapDumpOnOutOfMemoryError -Xmx64m
Heap dump captured on OOM.

OutOfMemoryError caught!
Heap dump should be in current directory.
Analyze with: mat heap dump file
```

**Heap Dump Analysis (MAT)** :
```
Leak Suspects Report:
- java.util.ArrayList (static LEAK field)
  - Retained size: 63 MB
  - Objects: 63 byte[] arrays
  - Dominator tree: HeapDumpDemo.LEAK dominates 63 MB
```

**Why This Output**: With `-XX:+HeapDumpOnOutOfMemoryError`, the JVM automatically captures a heap dump when OOM occurs. The dump is written to a `.hprof` file in the current directory. MAT analyzes the dump and identifies the `LEAK` static list as the dominator of 63 MB (the leaked `byte[]` arrays). The "Leak Suspects" report points directly to the leak source.

---

### Real-World Cases

- **Production Monitoring**: JFR with continuous recording captures leak events over days without significant overhead.
- **Incident Response**: Heap dumps captured on OOM are analyzed post-mortem to identify leak sources.
- **Development Testing**: VisualVM is used during development to profile memory usage and catch leaks early.
- **CI/CD Pipelines**: Automated heap analysis tools detect leaks in test runs.
- **Container Monitoring**: `jcmd` and `jstat` are used in containerized environments to monitor JVM memory.

### References

- Eclipse Memory Analyzer (MAT) - https://eclipse.dev/mat/
- JDK Flight Recorder - https://docs.oracle.com/en/java/javase/26/troubleshoot/
- VisualVM - https://visualvm.github.io/
- jstat - Java Platform SE 8 - https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jstat.html
- jmap - Java Platform SE 8 - https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jmap.html

---

## Deprecation and Safety Notes

| Feature | Status | Notes |
|---------|--------|-------|
| `finalize()` | Deprecated (JDK 9+) | Use `Cleaner` or try-with-resources. |
| `System.gc()` | Discouraged | Suggestion only; may trigger full GC. |
| `jhat` | Removed (JDK 9) | Use MAT or VisualVM instead. |
| `-XX:+HeapDumpOnOutOfMemoryError` | Active | Recommended for production. |
| JFR (JDK 8) | Commercial license required | Free in JDK 11+. |
| `ThreadLocal` | Active | Always call `remove()` in finally. |
| `WeakHashMap` | Active | Keys are weakly referenced. |
| Caffeine | Active | Recommended cache library. |
| Guava Cache | Active | Alternative cache library. |

---

## References

### Official Documentation

- Java Platform, Standard Edition Troubleshooting Guide - https://docs.oracle.com/en/java/javase/26/troubleshoot/
- ThreadLocal - Java SE 23 - https://docs.oracle.com/en/java/javase/23/docs/api/java.base/java/lang/ThreadLocal.html
- Package java.lang.ref - Java SE 23 - https://docs.oracle.com/en/java/javase/23/docs/api/java.base/java/lang/ref/package-summary.html
- MemoryMXBean - Java SE 23 - https://docs.oracle.com/en/java/javase/23/docs/api/java.management/java/lang/management/MemoryMXBean.html

### Tools

- Eclipse Memory Analyzer (MAT) - https://eclipse.dev/mat/
- VisualVM - https://visualvm.github.io/
- JDK Mission Control - https://docs.oracle.com/en/java/javase/26/troubleshoot/
- jstat - https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jstat.html
- jmap - https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jmap.html

### Tutorials and Articles

- Java Memory Leaks: Causes and Solutions - Baeldung - https://www.baeldung.com/java-memory-leaks
- 3 Ways to Detect Java Memory Leaks - Dynatrace - https://www.dynatrace.com/news/blog/3-ways-to-detect-java-memory-leaks/
- ThreadLocal Memory Leak in Java - Baeldung - https://www.baeldung.com/java-threadlocal-memory-leak
- Java Caching Best Practices - Baeldung - https://www.baeldung.com/java-caching
- Caffeine - https://github.com/ben-manes/caffeine
- Guava Cache - https://github.com/google/guava/wiki/CachesExplained

### Academic Resources

- BestGC: An Automatic GC Selector - https://ieeexplore.ieee.org/
- Million-Request Java: A Decision Framework for JVM Tuning in Ultra-Low Latency Microservices - https://zenodo.org/