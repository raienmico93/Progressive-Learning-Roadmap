# Garbage Collection: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Core Definition

Garbage Collection (GC) is the automatic memory management process by which the Java Virtual Machine (JVM) identifies and reclaims heap memory occupied by objects that are no longer reachable from any live thread, eliminating manual memory deallocation and preventing memory leaks.

### Technical Definition

Garbage collection is the automatic reclamation of heap-allocated memory that is no longer in use by a program. In the JVM, a garbage collector is a background process that monitors objects in memory, periodically checking if objects are still reachable, removing unreachable objects, and reorganizing surviving objects to improve memory efficiency. The JVM's GC subsystem is tightly integrated with the runtime data areas, particularly the heap and method area (Metaspace), and is governed by the Java Virtual Machine Specification, which does not mandate a specific GC algorithm but defines the conditions under which objects become eligible for collection.

### Beginner-Friendly Explanation

Imagine you're running a large office building (the JVM). Employees (objects) come and go. Some stay for years, others leave immediately. Instead of hiring someone to manually track who has left and clean their desks (manual memory management), the building has an automated cleaning crew (the garbage collector). This crew periodically checks every desk, determines whose occupants are still in the building (reachable), and cleans up the empty desks. The crew even reorganizes the remaining desks to make more room. You never have to tell the crew what to clean—they figure it out automatically.

### Key Characteristics

- **Automatic**: No manual `free` or `delete` calls; the JVM handles reclamation.
- **Non-Deterministic**: Collection timing is not guaranteed; it depends on heap pressure and GC algorithm.
- **Generational**: Exploits the Weak Generational Hypothesis to optimize collection frequency.
- **Concurrent**: Modern collectors perform most work concurrently with application threads.
- **Pausable**: Some phases require Stop-The-World pauses where application threads are halted.
- **Tunable**: Heap sizes, pause goals, and collector selection can be configured.

### Prerequisites

- Basic Java programming knowledge (classes, objects, references).
- Familiarity with the JVM memory model (heap, stack, method area).
- Understanding of object lifetime and references in Java.
- Awareness of command-line JVM flags.

### Related Programming Areas

- **JVM Memory Model**: Heap structure, object allocation, and promotion.
- **JIT Compilation**: Interaction with GC for allocation optimizations (escape analysis).
- **Concurrent Programming**: Java Memory Model (JMM) and thread synchronization.
- **Performance Engineering**: GC tuning for latency and throughput.
- **Native Memory**: Off-heap allocations and their interaction with GC.

### Core Concepts Overview

1. **Automatic Memory Management**: Tracking allocation and reclamation of heap-bound objects.
2. **Object Reachability**: Determining object lifecycles using GC Roots and reference types.
3. **Generational Concepts**: Exploiting the Weak Generational Hypothesis.
4. **Garbage-Collector Algorithms**: Serial, Parallel, CMS (deprecated), G1, ZGC, and Shenandoah.
5. **GC Pauses**: Stop-The-World safe-point pauses and tuning for latency vs. throughput.
6. **GC Monitoring**: Profiling collector operations using diagnostic tools.

---

## Core Concept 1: Automatic Memory Management

### Definitions

**Core Definition**: Automatic memory management is the process by which the JVM automatically allocates memory for new objects and reclaims memory from objects that are no longer reachable, without explicit programmer intervention.

**Technical Definition**: Automatic memory management in the JVM comprises two primary activities: **allocation** (assigning heap space to new objects, typically in Eden via Thread-Local Allocation Buffers) and **reclamation** (identifying unreachable objects and returning their memory to the pool of available heap space). The GC subsystem performs reclamation through a combination of marking (identifying live objects), sweeping (reclaiming dead objects), and compacting (reducing fragmentation). Manual memory management, by contrast, requires the programmer to explicitly allocate and free memory, introducing risks of memory leaks (failure to free) and dangling pointers (freeing memory still in use).

**Beginner-Friendly Explanation**: In languages like C and C++, you must manually tell the computer when you're done with a piece of memory (using `free` or `delete`). If you forget, the memory stays occupied forever—a memory leak. If you free memory too early, your program crashes. Java eliminates both problems: the JVM's garbage collector automatically finds memory that's no longer in use and reclaims it, letting you focus on writing your application logic.

### Purposes

- To eliminate memory leaks caused by forgotten deallocations.
- To prevent dangling pointer errors caused by premature deallocations.
- To reduce programmer cognitive load by abstracting memory lifecycle management.
- To enable safe, efficient reclamation of heap memory without explicit free calls.
- To support object reorganization (compaction) that reduces fragmentation.
- To provide a foundation for higher-level memory abstractions (reference types, finalization).

### Syntax Rules and Structure

#### Complete General Syntax: Allocation and Reclamation Cycle

```
AUTOMATIC MEMORY MANAGEMENT CYCLE
│
├── 1. ALLOCATION
│   ├── Triggered by: new, newarray, anewarray, multianewarray
│   ├── Location: Eden (Young Generation) via TLAB
│   ├── If TLAB full: allocate in shared Eden space
│   └── If Eden full: trigger Minor GC
│
├── 2. REACHABILITY ANALYSIS
│   ├── GC Roots identified (thread stacks, static fields, JNI refs)
│   ├── Mark phase: trace all reachable objects from roots
│   └── Unreachable objects identified as garbage
│
├── 3. RECLAMATION
│   ├── Sweep: reclaim memory from unmarked objects
│   ├── Compact: move surviving objects to reduce fragmentation
│   └── Update references to moved objects
│
└── 4. HEAP RESIZE (Optional)
    ├── Expand heap if allocation rate exceeds reclamation
    └── Shrink heap if free space exceeds threshold
```

#### Component Breakdown

| Phase | Responsibility | Trigger |
|-------|---------------|---------|
| Allocation | Assign heap space to new objects | `new`, array creation |
| Reachability Analysis | Identify live objects | GC cycle initiation |
| Reclamation | Free memory from dead objects | After marking |
| Compaction | Reduce fragmentation | During reclamation (optional) |
| Heap Resize | Adjust heap boundaries | Ergonomics policy |

#### Syntax Rules

- Objects are allocated in the Young Generation (Eden) by default.
- Large objects may be allocated directly in the Old Generation (`-XX:PretenureSizeThreshold`).
- `System.gc()` suggests a full GC but does not guarantee it.
- `Runtime.getRuntime().gc()` is equivalent to `System.gc()`.
- Memory leaks can still occur if objects remain reachable but are no longer needed (logical leaks).
- Finalizers (`finalize()`) are deprecated and should not be relied upon for resource cleanup.

#### Constraints and Limitations

- GC introduces pauses that halt application threads (Stop-The-World).
- GC cannot reclaim objects that are still reachable, even if the programmer considers them "unused."
- Memory leaks can still occur through unintentional references (e.g., static collections, ThreadLocal).
- `System.gc()` is a hint, not a command; the JVM may ignore it.
- Finalizers are non-deterministic and can cause performance issues.
- GC is not a substitute for proper resource management (files, sockets, native memory).

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Demonstrating Automatic Reclamation

**Setup Guide**: Save as `AutomaticMemoryDemo.java`, compile, and run.

```java
// AutomaticMemoryDemo.java
public class AutomaticMemoryDemo {
    
    static class Data {
        byte[] payload = new byte[1024]; // 1 KB per object
    }
    
    public static void main(String[] args) throws Exception {
        System.out.println("=== Automatic Memory Management Demo ===");
        
        Runtime rt = Runtime.getRuntime();
        
        // Report initial memory usage
        System.out.println("\n--- Initial Memory ---");
        System.out.println("Used: " + 
            (rt.totalMemory() - rt.freeMemory()) / 1024 + " KB");
        
        // Allocate 10,000 objects (most become garbage immediately)
        for (int i = 0; i < 10_000; i++) {
            Data d = new Data(); // Allocated in Eden
            // d goes out of scope immediately — eligible for GC
        }
        
        // Report memory after allocation
        System.out.println("\n--- After Allocating 10,000 Objects ---");
        System.out.println("Used: " + 
            (rt.totalMemory() - rt.freeMemory()) / 1024 + " KB");
        System.out.println("(Many objects are already garbage)");
        
        // Suggest GC (not guaranteed)
        System.gc();
        Thread.sleep(100); // Allow GC to run
        
        System.out.println("\n--- After GC Suggestion ---");
        System.out.println("Used: " + 
            (rt.totalMemory() - rt.freeMemory()) / 1024 + " KB");
        
        System.out.println("\nThe JVM automatically reclaimed memory");
        System.out.println("from objects that were no longer reachable.");
    }
}
```

**Expected Output** (approximate):
```
=== Automatic Memory Management Demo ===

--- Initial Memory ---
Used: 2048 KB

--- After Allocating 10,000 Objects ---
Used: 12288 KB
(Many objects are already garbage)

--- After GC Suggestion ---
Used: 3072 KB

The JVM automatically reclaimed memory
from objects that were no longer reachable.
```

**Why This Output**: The initial memory usage reflects the JVM's baseline (classes loaded, runtime structures). After allocating 10,000 `Data` objects, memory usage increases. However, since each `Data` object goes out of scope immediately after creation, they become eligible for garbage collection. The `System.gc()` call suggests a GC cycle, which reclaims most of the memory. The final usage is close to the initial usage, demonstrating automatic reclamation.

---

### Real-World Cases

- **Web Applications**: Each HTTP request creates objects (request, response, session data); GC automatically reclaims them after the request completes.
- **Batch Processing**: Large datasets are processed in chunks; intermediate objects are reclaimed between chunks, keeping memory usage bounded.
- **Caching**: Soft references allow caches to hold objects until memory pressure forces reclamation.
- **Event-Driven Systems**: Short-lived event objects are allocated and discarded rapidly; GC handles reclamation efficiently.

### References

- Introduction to Garbage Collection - Dev.java - https://dev.java/learn/jvm/tool/garbage-collection/intro/
- HotSpot Virtual Machine Garbage Collection Tuning Guide - https://docs.oracle.com/en/java/javase/26/gctuning/
- Java Platform, Standard Edition Troubleshooting Guide - https://docs.oracle.com/en/java/javase/26/troubleshoot/

---

## Core Concept 2: Object Reachability

### Definitions

**Core Definition**: Object reachability is the property that determines whether an object can be accessed by any live thread in the program, forming the basis for the garbage collector's decision to reclaim or retain an object.

**Technical Definition**: A reachable object is any object that can be accessed in any potential continuing computation from any live thread (as stated in JLS 12.6.1). Going from strongest to weakest, the different levels of reachability reflect the life cycle of an object: **strongly reachable** (reachable without traversing a Reference object), **softly reachable** (reachable only through soft references), **weakly reachable** (reachable only through weak references), **phantom reachable** (reachable only through phantom references after finalization), and **unreachable** (eligible for reclamation).

**Beginner-Friendly Explanation**: Think of reachability as a chain of connections from a set of "anchor points" (GC Roots) to objects. If you can follow a chain of references from an anchor to an object, that object is reachable and must not be collected. GC Roots are like the foundation of a building—they're always accessible. Objects connected to the foundation are safe. Objects that become disconnected (no chain from any root) are garbage. Java also provides different "strength" of connections: strong (normal), soft (cache-like), weak (mapping-like), and phantom (cleanup-like).

### Purposes

- To define the precise conditions under which an object becomes eligible for garbage collection.
- To enable memory-sensitive caching through soft references.
- To support canonicalizing mappings (e.g., `WeakHashMap`) without preventing key reclamation.
- To provide post-mortem cleanup notification through phantom references.
- To allow the garbage collector to make informed decisions about object retention.
- To support reference queues for notification of reachability changes.

### Syntax Rules and Structure

#### Complete General Syntax: Reachability Levels and GC Roots

```
REACHABILITY HIERARCHY
│
├── GC ROOTS (Always Reachable)
│   ├── Thread Stacks (local variables, operands)
│   ├── Static Fields (class variables)
│   ├── JNI References (native code references)
│   ├── Active Threads
│   └── System Classes (loaded by bootstrap loader)
│
├── STRONG REFERENCE (Normal Java references)
│   ├── Object obj = new Object();
│   └── Never collected while strongly reachable
│
├── SOFT REFERENCE (java.lang.ref.SoftReference)
│   ├── Collected only under memory pressure
│   └── Ideal for memory-sensitive caches
│
├── WEAK REFERENCE (java.lang.ref.WeakReference)
│   ├── Collected at next GC cycle
│   └── Ideal for canonicalizing mappings
│
├── PHANTOM REFERENCE (java.lang.ref.PhantomReference)
│   ├── Collected after finalization
│   └── Used for post-mortem cleanup scheduling
│
└── UNREACHABLE (Eligible for reclamation)
    └── No path from any GC root
```

#### Component Breakdown

| Reference Type | Class | Collection Trigger | Use Case |
|---------------|-------|-------------------|----------|
| Strong | (implicit) | Never (while reachable) | Normal object references |
| Soft | `SoftReference` | Memory pressure | Caches |
| Weak | `WeakReference` | Next GC cycle | Canonicalizing mappings |
| Phantom | `PhantomReference` | After finalization | Post-mortem cleanup |

#### Syntax Rules

- GC Roots include thread stacks, static fields, JNI references, and system classes.
- An object is strongly reachable if it can be accessed without traversing a `Reference` object.
- Soft references are cleared at the discretion of the garbage collector in response to memory demand.
- Weak references are cleared at the next GC cycle after the referent becomes weakly reachable.
- Phantom references are enqueued after the referent has been finalized and is phantom reachable.
- Reference queues (`ReferenceQueue`) notify programs of reachability changes.
- `Cleaner` (Java 9+) provides a safer alternative to finalization for cleanup.

#### Constraints and Limitations

- Soft references may be cleared at any time; they are not a guarantee of cache retention.
- Weak references are cleared aggressively; they may be collected even if memory is abundant.
- Phantom references are not automatically cleared; the program must handle cleanup.
- Finalization is deprecated and unreliable; use `Cleaner` or try-with-resources instead.
- Reference processing can add latency to GC cycles.
- GC Roots may vary by JVM implementation (HotSpot vs. OpenJ9).

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Demonstrating Reference Types

**Setup Guide**: Save as `ReferenceTypesDemo.java`, compile, and run.

```java
// ReferenceTypesDemo.java
import java.lang.ref.*;

public class ReferenceTypesDemo {
    
    static class Data {
        String name;
        Data(String name) { this.name = name; }
        @Override
        public String toString() { return "Data(" + name + ")"; }
    }
    
    public static void main(String[] args) throws Exception {
        System.out.println("=== Reference Types Demo ===");
        
        // Strong reference — normal object
        Data strong = new Data("strong");
        System.out.println("Strong: " + strong);
        
        // Soft reference — collected under memory pressure
        SoftReference<Data> soft = new SoftReference<>(new Data("soft"));
        System.out.println("Soft get: " + soft.get());
        
        // Weak reference — collected at next GC
        WeakReference<Data> weak = new WeakReference<>(new Data("weak"));
        System.out.println("Weak get (before GC): " + weak.get());
        
        // Phantom reference — enqueued after finalization
        ReferenceQueue<Data> queue = new ReferenceQueue<>();
        PhantomReference<Data> phantom = 
            new PhantomReference<>(new Data("phantom"), queue);
        System.out.println("Phantom get: " + phantom.get()); // Always null
        
        // Suggest GC
        System.gc();
        Thread.sleep(200);
        
        System.out.println("\n--- After GC ---");
        System.out.println("Strong: " + strong);        // Still alive
        System.out.println("Soft get: " + soft.get());  // May or may not be cleared
        System.out.println("Weak get: " + weak.get());  // Likely cleared (null)
        
        // Check reference queue
        Reference<?> ref = queue.poll();
        if (ref != null) {
            System.out.println("Phantom reference enqueued!");
        } else {
            System.out.println("Phantom reference not yet enqueued.");
        }
    }
}
```

**Expected Output**:
```
=== Reference Types Demo ===
Strong: Data(strong)
Soft get: Data(soft)
Weak get (before GC): Data(weak)
Phantom get: null

--- After GC ---
Strong: Data(strong)
Soft get: Data(soft)
Weak get: null
Phantom reference enqueued!
```

**Why This Output**: The strong reference remains alive throughout. The soft reference may survive GC if memory is abundant (it likely does here). The weak reference is cleared at the next GC cycle, so `get()` returns `null`. Phantom references always return `null` from `get()` and are enqueued after the referent is finalized. The reference queue confirms the phantom reference was processed.

---

#### Example 2: Using WeakHashMap for Canonicalizing Mappings

**Setup Guide**: Save as `WeakHashMapDemo.java`, compile, and run.

```java
// WeakHashMapDemo.java
import java.util.WeakHashMap;

public class WeakHashMapDemo {
    
    public static void main(String[] args) throws Exception {
        System.out.println("=== WeakHashMap Demo ===");
        
        // WeakHashMap: keys are weakly referenced
        // When a key is no longer strongly referenced, the entry is removed
        WeakHashMap<Object, String> map = new WeakHashMap<>();
        
        // Create keys
        Object key1 = new Object();
        Object key2 = new Object();
        
        map.put(key1, "value1");
        map.put(key2, "value2");
        
        System.out.println("Initial size: " + map.size());
        System.out.println("key1 -> " + map.get(key1));
        System.out.println("key2 -> " + map.get(key2));
        
        // Remove strong reference to key1
        key1 = null;
        
        // Suggest GC
        System.gc();
        Thread.sleep(200);
        
        System.out.println("\n--- After GC ---");
        System.out.println("Size: " + map.size());
        System.out.println("key2 -> " + map.get(key2));
        // key1 entry should be removed because key1 is no longer strongly reachable
        
        System.out.println("\nWeakHashMap entries are removed when keys");
        System.out.println("are no longer strongly referenced.");
    }
}
```

**Expected Output**:
```
=== WeakHashMap Demo ===
Initial size: 2
key1 -> value1
key2 -> value2

--- After GC ---
Size: 1
key2 -> value2

WeakHashMap entries are removed when keys
are no longer strongly referenced.
```

**Why This Output**: In a `WeakHashMap`, keys are held via weak references. When `key1` is set to `null`, the only reference to that object is the weak reference inside the map. At the next GC cycle, the weak reference is cleared, and the entry is removed from the map. The size decreases from 2 to 1. This demonstrates how weak references enable canonicalizing mappings that do not prevent key reclamation.

---

### Real-World Cases

- **Caching (Soft References)**: Image caches and data caches use soft references to hold expensive-to-compute objects until memory pressure forces reclamation.
- **Canonicalizing Mappings (Weak References)** : `WeakHashMap` is used for class loaders, where entries should be removed when the class loader is no longer referenced.
- **Resource Cleanup (Phantom References)** : `Cleaner` uses phantom references to schedule cleanup actions (e.g., closing files, freeing native memory) after object finalization.
- **ThreadLocal Cleanup**: ThreadLocal uses weak references for keys to allow thread-local values to be reclaimed when the thread dies.
- **Listener Registries**: Weak references prevent listener registries from leaking memory when listeners are no longer referenced.

### References

- Package java.lang.ref - Java SE 23 - https://docs.oracle.com/en/java/javase/23/docs/api/java.base/java/lang/ref/package-summary.html
- Java Platform, Standard Edition Troubleshooting Guide - https://docs.oracle.com/en/java/javase/26/troubleshoot/
- JLS 12.6.1: Implementing Finalization - https://docs.oracle.com/javase/specs/jls/se23/html/jls-12.html#jls-12.6.1

---

## Core Concept 3: Generational Concepts

### Definitions

**Core Definition**: Generational garbage collection is a strategy that partitions the heap into generations based on object age, exploiting the empirical observation that most objects die young to optimize collection frequency and minimize broad heap scans.

**Technical Definition**: The most important observed property exploited by generational collection is the **weak generational hypothesis**, which states that most objects survive for only a short period of time. To optimize for this scenario, memory is managed in generations (memory pools holding objects of different ages). Garbage collection occurs in each generation when the generation fills up. The vast majority of objects are allocated in a pool dedicated to young objects (the young generation), and most objects die there. When the young generation fills up, it causes a **minor collection** in which only the young generation is collected. Objects that survive minor collections are eventually promoted to the **old generation**, where they are collected during **major collections** (or full GC) when the old generation fills up.

**Beginner-Friendly Explanation**: Imagine a cafeteria with two areas: a "quick lunch" area for people who eat and leave quickly, and a "long-term seating" area for people who stay for hours. The cafeteria staff cleans the quick lunch area frequently (minor GC) because most people there leave soon after finishing. Only people who stay a long time get moved to the long-term area, which is cleaned less often (major GC). This is much more efficient than cleaning the entire cafeteria every time someone leaves.

### Purposes

- To optimize collection frequency by focusing on the generation where most garbage accumulates.
- To minimize the cost of collection by scanning only the young generation during minor collections.
- To reduce pause times by collecting smaller memory regions.
- To exploit the observed distribution of object lifetimes for performance gains.
- To separate short-lived and long-lived objects, reducing promotion overhead.
- To enable efficient use of copying collectors for young generation reclamation.

### Syntax Rules and Structure

#### Complete General Syntax: Generational Heap Layout

```
GENERATIONAL HEAP LAYOUT
│
├── YOUNG GENERATION (Frequently Collected)
│   ├── Eden (new objects allocated here)
│   ├── Survivor S0 (from-space)
│   └── Survivor S1 (to-space)
│
└── OLD GENERATION (Infrequently Collected)
    └── Long-lived objects (promoted from Survivor spaces)
```

#### Component Breakdown

| Generation | Collection Event | Trigger | Cost |
|-----------|-----------------|---------|------|
| Young (Eden, S0, S1) | Minor GC | Eden fills | Low (small region) |
| Old | Major/Full GC | Old fills | High (entire heap) |

#### Syntax Rules

- New objects are allocated in Eden.
- Minor GC copies surviving objects to a Survivor space.
- Objects surviving multiple minor GCs are promoted to the Old Generation.
- The tenuring threshold determines when promotion occurs.
- Young Generation size is configured with `-Xmn` or `-XX:NewRatio`.
- Survivor space size is configured with `-XX:SurvivorRatio`.
- `-XX:MaxTenuringThreshold` sets the maximum age before promotion.

#### Constraints and Limitations

- The Weak Generational Hypothesis does not hold for all applications (e.g., caches with long-lived objects).
- Premature promotion can occur if Survivor spaces are too small.
- Large objects may be allocated directly in the Old Generation.
- Minor GC pauses increase with Young Generation size.
- Major GC pauses increase with Old Generation size.
- Generational organization is less effective for applications with uniformly long-lived objects.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Observing Generational Collection

**Setup Guide**: Save as `GenerationalDemo.java`, compile, and run with `-Xlog:gc*` (JDK 9+) or `-XX:+PrintGCDetails` (JDK 8).

```java
// GenerationalDemo.java
import java.util.ArrayList;
import java.util.List;

public class GenerationalDemo {
    
    static class ShortLived {
        byte[] data = new byte[1024]; // 1 KB
    }
    
    static class LongLived {
        byte[] data = new byte[1024]; // 1 KB
    }
    
    public static void main(String[] args) {
        System.out.println("=== Generational Collection Demo ===");
        
        // Long-lived objects — promoted to Old Generation
        List<LongLived> oldGen = new ArrayList<>();
        for (int i = 0; i < 500; i++) {
            oldGen.add(new LongLived());
        }
        
        // Short-lived objects — most die in Eden
        for (int cycle = 0; cycle < 10; cycle++) {
            for (int i = 0; i < 5000; i++) {
                ShortLived s = new ShortLived(); // Dies immediately
            }
            System.out.println("Cycle " + (cycle + 1) + 
                " — Old Gen size: " + oldGen.size());
        }
        
        System.out.println("\nShort-lived objects were collected in Eden.");
        System.out.println("Long-lived objects survived and were promoted.");
    }
}
```

**Expected GC Log Output** (with `-Xlog:gc`):
```
[0.123s][info][gc] GC(0) Pause Young (Normal) (G1 Evacuation Pause) 12M->3M(256M) 5.234ms
[0.234s][info][gc] GC(1) Pause Young (Normal) (G1 Evacuation Pause) 15M->4M(256M) 4.567ms
[0.345s][info][gc] GC(2) Pause Young (Normal) (G1 Evacuation Pause) 18M->5M(256M) 5.123ms
...
[1.234s][info][gc] GC(9) Pause Young (Normal) (G1 Evacuation Pause) 28M->8M(256M) 6.789ms
```

**Why This Output**: The GC log shows repeated **Young** collections (minor GCs) triggered by the short-lived `ShortLived` objects filling Eden. Each collection reclaims most of the young generation (e.g., 12M→3M). The `LongLived` objects survive multiple minor collections and are eventually promoted to the Old Generation. The Old Generation size remains stable (500 objects), while the Young Generation is collected frequently.

---

### Real-World Cases

- **Web Servers**: Request/response objects are short-lived and collected in Young Generation; session objects are long-lived and promoted to Old Generation.
- **Batch Processing**: Intermediate computation objects die young; configuration objects live for the entire batch.
- **Caching**: Cached objects survive many minor collections and are promoted to Old Generation.
- **Event Processing**: Event objects are created and discarded rapidly; listener objects are long-lived.

### References

- 3 Generations - Oracle Garbage Collection Tuning Guide - https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/generations.html
- Weak Generational Hypothesis - Inside.java - https://inside.java/2024/11/22/mark-scavenge/
- Java Garbage Collection Basics - Oracle - https://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/index.html

---

## Core Concept 4: Garbage-Collector Algorithms

### Definitions

**Core Definition**: Garbage-collector algorithms are the specific strategies and implementations the JVM uses to reclaim heap memory, each optimized for different performance characteristics (throughput, latency, memory footprint).

**Technical Definition**: The JVM provides multiple garbage collectors, each implementing different collection algorithms. **Serial GC** uses a single thread for all collection work. **Parallel GC** (Throughput Collector) uses multiple threads for minor collections. **CMS** (Concurrent Mark Sweep, deprecated) performs most work concurrently with application threads. **G1** (Garbage-First) is a server-style collector for multiprocessor machines with large memory, meeting pause-time goals with high probability. **ZGC** and **Shenandoah** are ultra-low-latency concurrent collectors targeting sub-10ms pauses on multi-terabyte heaps.

**Beginner-Friendly Explanation**: Different garbage collectors are like different cleaning crews. The Serial GC is a single janitor—simple, efficient for small buildings, but slow for large ones. The Parallel GC is a team of janitors working simultaneously—faster for big buildings. The G1 is a smart team that prioritizes the messiest rooms first. ZGC and Shenandoah are like a crew that cleans while people are still working, causing almost no disruption.

### Purposes

- To provide collection strategies optimized for different application requirements.
- To enable throughput-oriented applications to maximize work done per unit time.
- To enable latency-sensitive applications to minimize pause times.
- To support very large heaps (multi-terabyte) with bounded pause times.
- To allow applications to select the collector best suited to their workload.
- To provide a progression of collectors as JVM technology evolves.

### Syntax Rules and Structure

#### Complete General Syntax: Collector Selection

```
COLLECTOR SELECTION
│
├── Serial GC: -XX:+UseSerialGC
│   └── Single-threaded, stop-the-world
│
├── Parallel GC: -XX:+UseParallelGC
│   └── Multi-threaded, stop-the-world
│
├── CMS (Deprecated): -XX:+UseConcMarkSweepGC
│   └── Concurrent, low-pause (removed in JDK 14)
│
├── G1 GC (Default): -XX:+UseG1GC
│   └── Region-based, concurrent, pause-time goal
│
├── ZGC: -XX:+UseZGC
│   └── Concurrent, ultra-low-latency, scalable
│
└── Shenandoah: -XX:+UseShenandoahGC
    └── Concurrent, low-pause, region-based
```

#### Component Breakdown

| Collector | Threads | Pause Goal | Best For | Heap Size |
|-----------|---------|-----------|----------|-----------|
| Serial | Single | High (100-500ms) | Small apps, embedded | < 100 MB |
| Parallel | Multiple | High (100ms-sec) | Throughput | 100 MB - 4 GB |
| CMS (Deprecated) | Concurrent | Low (< 200ms) | Web apps | 1-16 GB |
| G1 | Concurrent | Configurable (< 200ms) | Balanced | 4 GB - 64 GB |
| ZGC | Concurrent | Ultra-low (< 10ms) | Latency-critical | Up to 16 TB |
| Shenandoah | Concurrent | Low (< 10ms) | Containerized | Up to 16 TB |

#### Syntax Rules

- Serial GC is the default for client-class machines (single-core, small memory).
- Parallel GC is the default for server-class machines in JDK 8.
- G1 GC is the default collector since JDK 9.
- ZGC and Shenandoah are experimental in some JDK versions but production-ready in JDK 21+.
- CMS was deprecated in JDK 9 and removed in JDK 14.
- Collector selection is done via `-XX:+Use<CollectorName>` flags.
- Pause-time goal is set with `-XX:MaxGCPauseMillis=<ms>` (G1, ZGC, Shenandoah).

#### Constraints and Limitations

- Serial GC is not suitable for multi-core servers.
- Parallel GC has long pause times for large heaps.
- CMS has fragmentation issues and was removed.
- G1 has overhead for very small heaps.
- ZGC and Shenandoah have higher memory overhead and lower throughput than Parallel GC.
- Collector behavior varies by JDK version and vendor implementation.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Comparing Collector Performance

**Setup Guide**: Save as `CollectorComparison.java`, compile, and run with different collector flags.

```java
// CollectorComparison.java
public class CollectorComparison {
    
    static long computeSum(int n) {
        long sum = 0;
        for (int i = 1; i <= n; i++) {
            sum += i;
        }
        return sum;
    }
    
    public static void main(String[] args) {
        System.out.println("=== Collector Performance Comparison ===");
        System.out.println("JVM: " + System.getProperty("java.vm.name"));
        
        // Allocate objects to trigger GC
        long start = System.nanoTime();
        
        for (int cycle = 0; cycle < 100; cycle++) {
            // Allocate short-lived objects
            for (int i = 0; i < 10_000; i++) {
                byte[] temp = new byte[128];
            }
            
            // Compute to keep CPU busy
            computeSum(1000);
        }
        
        long end = System.nanoTime();
        
        System.out.println("Elapsed: " + 
            String.format("%.2f ms", (end - start) / 1_000_000.0));
        System.out.println("Collector: " + 
            System.getProperty("java.vm.gc.name", "unknown"));
    }
}
```

**Run with different collectors**:
```bash
java -XX:+UseSerialGC CollectorComparison
java -XX:+UseParallelGC CollectorComparison
java -XX:+UseG1GC CollectorComparison
java -XX:+UseZGC CollectorComparison
java -XX:+UseShenandoahGC CollectorComparison
```

**Expected Output** (approximate, varies by hardware):
```
=== Collector Performance Comparison ===
JVM: OpenJDK 64-Bit Server VM
Elapsed: 45.23 ms
Collector: Serial
```

**Why This Output**: The elapsed time varies by collector. Serial GC is slowest for multi-core machines. Parallel GC is faster. G1 balances pause and throughput. ZGC and Shenandoah have slightly lower throughput but much shorter pauses. The exact numbers depend on heap size, allocation rate, and hardware.

---

### Real-World Cases

- **Embedded Devices**: Serial GC is used for minimal memory footprint.
- **Batch Processing**: Parallel GC maximizes throughput for CPU-bound jobs.
- **Web Applications (Legacy)** : CMS was used for low-pause web serving.
- **General-Purpose Servers**: G1 is the default for balanced performance.
- **Low-Latency Trading**: ZGC or Shenandoah provides sub-millisecond pauses.
- **Containerized Microservices**: Shenandoah is preferred for container-aware memory management.

### References

- 5 Available Collectors - Oracle Garbage Collection Tuning Guide - https://docs.oracle.com/en/java/javase/26/gctuning/available-collectors.html
- Garbage Collection in Java - Dev.java - https://dev.java/learn/jvm/tool/garbage-collection/
- JEP 189: Shenandoah: A Low-Pause Garbage Collector - https://openjdk.org/jeps/189
- JEP 333: ZGC: A Scalable Low-Latency Garbage Collector - https://openjdk.org/jeps/333

---

## Core Concept 5: GC Pauses

### Definitions

**Core Definition**: A GC pause (Stop-The-World pause) is a period during garbage collection when all application threads are halted to allow the collector to perform critical operations that require a consistent heap state.

**Technical Definition**: The pause time is the duration during which the garbage collector stops the application and recovers space that is no longer in use. The HotSpot VM uses a **safepoint** mechanism to stop all threads: threads must reach a safepoint (a point in execution where the thread's state is known and safe to modify) before the VM can perform certain operations. Safe points are triggered by GC, JIT deoptimization, thread dumps, and other global operations. The maximum pause-time goal is specified with `-XX:MaxGCPauseMillis=<nnn>`, which is interpreted as a hint to the collector that a pause time of `<nnn>` milliseconds or fewer is desired.

**Beginner-Friendly Explanation**: Imagine a busy restaurant kitchen. Normally, all chefs (application threads) are cooking. But once in a while, the head chef needs to do a quick inventory of all ingredients (GC). To do this accurately, they ask everyone to stop cooking for a moment (Stop-The-World pause). The chefs put down their tools at safe points and wait. The inventory is taken, and then everyone resumes. Modern collectors (like G1, ZGC, Shenandoah) try to do most of the inventory while chefs are still cooking, only stopping them briefly for the final count.

### Purposes

- To ensure a consistent heap state during critical GC operations.
- To allow the collector to safely move objects and update references.
- To enable accurate marking of reachable objects.
- To bound the maximum pause time for latency-sensitive applications.
- To provide a trade-off mechanism between throughput and latency.
- To support safe-point-based global synchronization.

### Syntax Rules and Structure

#### Complete General Syntax: Pause Time Goals

```
PAUSE TIME TUNING
│
├── MaxGCPauseMillis
│   ├── Flag: -XX:MaxGCPauseMillis=<nnn>
│   ├── Interpreted as a hint to the collector
│   ├── Collector adjusts young gen size to meet goal
│   └── Does NOT guarantee the pause will be under the goal
│
├── GCTimeRatio (Throughput Goal)
│   ├── Flag: -XX:GCTimeRatio=<nnn>
│   ├── Ratio of GC time to application time
│   ├── Default: 99 (1% GC time)
│   └── Trade-off: higher throughput vs. longer pauses
│
└── Safe Point Tuning
    ├── GuaranteedSafepointInterval: -XX:GuaranteedSafepointInterval=<ms>
    ├── Default: 1000ms (forced safepoint interval)
    └── Balance safepoint overhead vs. pause stability
```

#### Component Breakdown

| Flag | Purpose | Default | Collector |
|------|---------|---------|-----------|
| `-XX:MaxGCPauseMillis` | Maximum pause-time goal | Not set | G1, ZGC, Shenandoah |
| `-XX:GCTimeRatio` | Throughput goal (GC/app time) | 99 | All |
| `-XX:GuaranteedSafepointInterval` | Forced safepoint interval | 1000ms | All |
| `-XX:+UseCountedLoopSafepoints` | Counted loop safepoints | Enabled | All |

#### Syntax Rules

- Pause time is the duration during which GC stops the application.
- `-XX:MaxGCPauseMillis` is a hint, not a guarantee.
- The collector may adjust Young Generation size to meet the pause goal.
- Throughput and latency are opposing goals; improving one typically degrades the other.
- Safe points are reached by threads at known execution points.
- `GuaranteedSafepointInterval` forces a safepoint even if no GC is needed.
- Setting `MaxGCPauseMillis` too low may cause increased GC frequency.

#### Constraints and Limitations

- No collector can guarantee a specific pause time.
- Very low pause goals may reduce throughput significantly.
- Safe points introduce latency even when GC is not running.
- Long-running counted loops may delay safepoint arrival.
- The trade-off between latency and throughput is fundamental.
- Pause times increase with heap size for stop-the-world collectors.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Measuring GC Pauses with `-Xlog:gc`

**Setup Guide**: Save as `GCPauseDemo.java`, compile, and run with `-Xlog:gc:file=gc.log`.

```java
// GCPauseDemo.java
import java.util.ArrayList;
import java.util.List;

public class GCPauseDemo {
    
    public static void main(String[] args) throws Exception {
        System.out.println("=== GC Pause Measurement ===");
        
        List<byte[]> list = new ArrayList<>();
        
        // Allocate objects to trigger GC
        for (int i = 0; i < 1000; i++) {
            list.add(new byte[1024 * 1024]); // 1 MB each
            
            if (i % 100 == 0) {
                System.out.println("Allocated " + (i + 1) + " MB");
            }
            
            // Periodically release to trigger GC
            if (i % 200 == 0 && i > 0) {
                list.clear();
                System.gc();
            }
        }
        
        System.out.println("\nCheck gc.log for pause times:");
        System.out.println("- Look for 'Pause Young' and 'Pause Full' entries");
        System.out.println("- The time after the pause is the duration");
    }
}
```

**Expected GC Log Output** (partial, with `-Xlog:gc`):
```
[0.123s][info][gc] GC(0) Pause Young (Normal) (G1 Evacuation Pause) 12M->3M(256M) 5.234ms
[0.456s][info][gc] GC(1) Pause Young (Normal) (G1 Evacuation Pause) 28M->6M(256M) 6.789ms
[0.789s][info][gc] GC(2) Pause Full (G1 Compaction Pause) 45M->12M(256M) 45.123ms
```

**Expected Program Output**:
```
=== GC Pause Measurement ===
Allocated 1 MB
Allocated 101 MB
Allocated 201 MB
...
Allocated 1001 MB

Check gc.log for pause times:
- Look for 'Pause Young' and 'Pause Full' entries
- The time after the pause is the duration
```

**Why This Output**: The GC log shows pause times for Young collections (5-7ms) and Full collections (45ms). Young collections are frequent and short; Full collections are rare and long. The pause times are the durations during which application threads were halted. Tuning `-XX:MaxGCPauseMillis` would influence how the collector balances Young Generation size to meet the pause goal.

---

#### Example 2: Tuning Pause Time Goal

**Setup Guide**: Save as `PauseTuning.java`, compile, and run with different `-XX:MaxGCPauseMillis` values.

```java
// PauseTuning.java
public class PauseTuning {
    
    static long work(int n) {
        long sum = 0;
        for (int i = 0; i < n; i++) {
            sum += i * i;
        }
        return sum;
    }
    
    public static void main(String[] args) {
        System.out.println("=== Pause Time Goal Tuning ===");
        System.out.println("Run with:");
        System.out.println("  -XX:MaxGCPauseMillis=50   (aggressive)");
        System.out.println("  -XX:MaxGCPauseMillis=200  (balanced)");
        System.out.println("  -XX:MaxGCPauseMillis=500  (relaxed)");
        System.out.println();
        
        // Mixed workload: allocation + computation
        long total = 0;
        for (int cycle = 0; cycle < 50; cycle++) {
            // Allocate
            byte[] temp = new byte[1024 * 1024]; // 1 MB
            
            // Compute
            total += work(100_000);
        }
        
        System.out.println("Total: " + total);
        System.out.println("\nCheck GC log for pause times.");
        System.out.println("Lower goal = smaller young gen = more frequent GC");
        System.out.println("Higher goal = larger young gen = less frequent GC");
    }
}
```

**Expected Output** (varies by setting):
```
=== Pause Time Goal Tuning ===
Run with:
  -XX:MaxGCPauseMillis=50   (aggressive)
  -XX:MaxGCPauseMillis=200  (balanced)
  -XX:MaxGCPauseMillis=500  (relaxed)

Total: 166666650000

Check GC log for pause times.
Lower goal = smaller young gen = more frequent GC
Higher goal = larger young gen = less frequent GC
```

**Why This Output**: With `MaxGCPauseMillis=50`, G1 uses a smaller Young Generation to meet the 50ms goal, triggering more frequent minor GCs. With `MaxGCPauseMillis=500`, G1 uses a larger Young Generation, triggering fewer but longer minor GCs. The total computation remains the same, but the GC frequency and pause duration differ. This demonstrates the trade-off between pause time and GC frequency.

---

### Real-World Cases

- **Low-Latency Trading**: Setting `-XX:MaxGCPauseMillis=5` with ZGC ensures sub-10ms pauses for order-matching systems.
- **Web Applications**: G1 with `-XX:MaxGCPauseMillis=200` balances pause time and throughput for request processing.
- **Batch Processing**: Relaxing pause goals (`-XX:MaxGCPauseMillis=1000`) maximizes throughput for CPU-bound jobs.
- **Interactive Applications**: Tuning safepoint intervals reduces UI freezes caused by forced safepoints.
- **Containerized Services**: Setting pause goals prevents GC pauses from exceeding container liveness probe timeouts.

### References

- HotSpot Virtual Machine Garbage Collection Tuning Guide - https://docs.oracle.com/en/java/javase/26/gctuning/
- Java SafePoint: The Core Mechanism of JVM Pauses - https://developer.aliyun.com/article/1659014
- Garbage-First Garbage Collector Tuning - https://docs.oracle.com/en/java/javase/26/gctuning/garbage-first-garbage-collector-tuning.html

---

## Core Concept 6: GC Monitoring

### Definitions

**Core Definition**: GC monitoring is the practice of observing, logging, and analyzing garbage collection behavior using diagnostic tools to diagnose performance issues, identify memory leaks, and tune collector parameters.

**Technical Definition**: The JDK provides multiple tools for GC monitoring: **jstat** (JVM statistics monitoring tool for real-time GC data), **jcmd** (JVM diagnostic command utility for heap dumps, GC runs, and Flight Recordings), **jmap** (memory map tool for heap histograms and dumps), and **JDK Mission Control** (graphical analysis of Java Flight Recorder data). GC logging is configured with `-Xlog:gc*` (JDK 9+) or legacy flags (`-XX:+PrintGCDetails -Xloggc:<file>` in JDK 8). The `-Xlog:gc*` option logs messages tagged with the `gc` tag at info level to stdout.

**Beginner-Friendly Explanation**: GC monitoring is like having a dashboard for your car's engine. The `jstat` tool gives you real-time gauges (like RPM and temperature). GC logs are like a flight recorder that captures everything that happens. JDK Mission Control is like a diagnostic computer that reads the flight recorder and shows you graphs and alerts. Together, these tools help you understand what the garbage collector is doing and whether it's causing problems.

### Purposes

- To observe GC frequency, duration, and pause times in real time.
- To diagnose memory leaks by tracking heap usage over time.
- To identify performance bottlenecks caused by GC.
- To tune collector parameters based on empirical data.
- To capture GC logs for post-mortem analysis.
- To analyze allocation patterns and object lifetimes.

### Syntax Rules and Structure

#### Complete General Syntax: GC Monitoring Tools

```
GC MONITORING TOOLS
│
├── jstat (Real-time Statistics)
│   ├── jstat -gc <pid> <interval> <count>
│   ├── jstat -gcutil <pid> <interval>
│   ├── jstat -gccapacity <pid>
│   └── jstat -gcnew / -gcold <pid>
│
├── jcmd (Diagnostic Commands)
│   ├── jcmd <pid> GC.heap_info
│   ├── jcmd <pid> GC.run
│   ├── jcmd <pid> GC.heap_dump <file>
│   └── jcmd <pid> JFR.start
│
├── jmap (Memory Map)
│   ├── jmap -heap <pid>
│   ├── jmap -histo <pid>
│   └── jmap -dump:format=b,file=<file> <pid>
│
├── GC Logging
│   ├── JDK 9+: -Xlog:gc*:file=gc.log:time,uptime
│   ├── JDK 8: -XX:+PrintGCDetails -Xloggc:gc.log
│   └── Rotation: -Xlog:gc*:file=gc.log:time,uptime,level,tags
│
└── JDK Mission Control
    ├── Graphical analysis of Java Flight Recorder data
    ├── GC tab shows pauses, heap usage, allocation
    └── Requires JFR recording
```

#### Component Breakdown

| Tool | Purpose | Output Format |
|------|---------|---------------|
| jstat | Real-time GC statistics | Tabular (stdout) |
| jcmd | Diagnostic commands | Text/JSON |
| jmap | Heap histogram/dump | Text/binary |
| GC Logging | Persistent GC events | Text file/stdout |
| JMC | Visual analysis | GUI |

#### Syntax Rules

- `jstat` requires the process ID (PID) of the target JVM.
- `jcmd` provides a unified interface for multiple diagnostic commands.
- `jmap` can trigger a Full GC before dumping the heap (`-dump:live`).
- `-Xlog:gc*` logs all GC-related messages at all levels.
- GC log rotation is configured with `file=<file>:time` or `filecount=<n>`.
- JFR recording is started with `-XX:StartFlightRecording` or `jcmd JFR.start`.
- JDK Mission Control analyzes JFR recordings (`.jfr` files).

#### Constraints and Limitations

- `jstat` requires attaching to the target JVM; may not work with all security policies.
- `jmap` heap dumps can be large and impact application performance.
- GC logging adds minor overhead; high-frequency logging can affect performance.
- JFR requires a commercial license in some Oracle JDK versions (free in OpenJDK).
- JMC is not included in all JDK distributions (available as separate download).
- Real-time monitoring tools may not capture short-lived GC events.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Real-Time Monitoring with `jstat`

**Setup Guide**: Start a Java application, then run `jstat -gc <pid> 1000` in a separate terminal.

```java
// JStatDemo.java
import java.util.ArrayList;
import java.util.List;

public class JStatDemo {
    public static void main(String[] args) throws Exception {
        System.out.println("=== jstat Monitoring Demo ===");
        System.out.println("PID: " + ProcessHandle.current().pid());
        System.out.println("Run: jstat -gc " + 
            ProcessHandle.current().pid() + " 1000");
        System.out.println();
        
        List<byte[]> list = new ArrayList<>();
        
        // Allocate memory to trigger GC
        for (int i = 0; i < 200; i++) {
            list.add(new byte[1024 * 1024]); // 1 MB
            
            if (i % 50 == 0) {
                System.out.println("Allocated " + (i + 1) + " MB");
            }
            
            if (i % 100 == 0 && i > 0) {
                list.clear();
                System.gc();
            }
            
            Thread.sleep(100);
        }
        
        System.out.println("Done.");
    }
}
```

**jstat Output** (in separate terminal):
```
  S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT
 5120.0 5120.0  0.0   5120.0  32768.0  16384.0   131072.0   13107.2   4864.0 4685.3 512.0  480.2      5    0.123   1      0.045    0.168
 5120.0 5120.0  0.0   5120.0  32768.0  32768.0   131072.0   26214.4   4864.0 4685.3 512.0  480.2      8    0.234   1      0.045    0.279
...
```

**Expected Program Output**:
```
=== jstat Monitoring Demo ===
PID: 12345
Run: jstat -gc 12345 1000

Allocated 1 MB
Allocated 51 MB
Allocated 101 MB
Allocated 151 MB
Done.
```

**Why This Output**: The `jstat -gc` output shows columns for Survivor spaces (S0C, S1C, S0U, S1U), Eden (EC, EU), Old Generation (OC, OU), Metaspace (MC, MU), and GC counts (YGC, YGCT, FGC, FGCT). As the program allocates memory, YGC (Young GC count) increases. The EU (Eden Used) column fluctuates as Eden fills and is collected. This real-time view helps diagnose allocation patterns and GC frequency.

---

#### Example 2: GC Logging Analysis

**Setup Guide**: Save as `GCLogDemo.java`, compile, and run with `-Xlog:gc*:file=gc.log:time,uptime`.

```java
// GCLogDemo.java
public class GCLogDemo {
    public static void main(String[] args) {
        System.out.println("=== GC Logging Demo ===");
        System.out.println("GC events logged to gc.log");
        
        // Allocate memory to trigger GC
        for (int i = 0; i < 100; i++) {
            byte[] data = new byte[1024 * 1024]; // 1 MB
            if (i % 20 == 0) {
                System.out.println("Allocated " + (i + 1) + " MB");
            }
        }
        
        System.out.println("\nAnalyze gc.log for:");
        System.out.println("- GC type (Young, Full)");
        System.out.println("- Pause duration");
        System.out.println("- Heap usage before/after");
        System.out.println("- GC cause (Allocation Failure, etc.)");
    }
}
```

**Expected GC Log Output** (partial):
```
[2026-09-28T10:30:15.123+0000][0.123s][info][gc] GC(0) Pause Young (Normal) (G1 Evacuation Pause) 12M->3M(256M) 5.234ms
[2026-09-28T10:30:15.456+0000][0.456s][info][gc] GC(1) Pause Young (Normal) (G1 Evacuation Pause) 28M->6M(256M) 6.789ms
[2026-09-28T10:30:15.789+0000][0.789s][info][gc] GC(2) Pause Full (G1 Compaction Pause) 45M->12M(256M) 45.123ms
```

**Expected Program Output**:
```
=== GC Logging Demo ===
GC events logged to gc.log
Allocated 1 MB
Allocated 21 MB
Allocated 41 MB
Allocated 61 MB
Allocated 81 MB

Analyze gc.log for:
- GC type (Young, Full)
- Pause duration
- Heap usage before/after
- GC cause (Allocation Failure, etc.)
```

**Why This Output**: The GC log entries show timestamps, uptime, log level (info), tags (gc), GC ID, type (Young/Full), cause (G1 Evacuation Pause), heap usage before and after (12M→3M), total heap capacity (256M), and pause duration (5.234ms). This information is invaluable for understanding GC behavior: Young collections are frequent and short; Full collections are rare and long. The log can be analyzed with tools like GCViewer or JDK Mission Control.

---

### Real-World Cases

- **Production Monitoring**: `jstat -gcutil <pid> 5000` is used to monitor GC utilization in production without significant overhead.
- **Memory Leak Diagnosis**: `jmap -histo:live <pid>` identifies object counts by class, revealing leaks (e.g., growing collections).
- **Post-Mortem Analysis**: Heap dumps (`jmap -dump`) are analyzed with Eclipse MAT to find memory leaks.
- **Flight Recording**: JFR captures detailed GC events with low overhead for later analysis in JMC.
- **Container Monitoring**: GC logs are shipped to centralized logging systems (ELK, Splunk) for alerting on Full GC frequency.

### References

- Lesson 3: Diagnostic Data Collection and Analysis Tools - Oracle - https://www.oracle.com/webfolder/technetwork/tutorials/mooc/JVM_Troubleshooting/week3/lesson3.pdf
- JDK Mission Control - Oracle - https://docs.oracle.com/en/java/javase/26/troubleshoot/
- JVM Logging - Sip of Java - https://inside.java/2022/11/07/sip-of-java-jvm-logging/
- jstat - Java Platform SE 8 - https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jstat.html

---

## Deprecation and Safety Notes

| Feature | Status | Notes |
|---------|--------|-------|
| CMS GC | Removed (JDK 14) | Deprecated in JDK 9, removed in JDK 14. Use G1 or ZGC. |
| `-XX:+UseConcMarkSweepGC` | Removed | Flag no longer recognized in JDK 14+. |
| `finalize()` | Deprecated (JDK 9+) | Use `Cleaner` or try-with-resources instead. |
| `-XX:+PrintGCDetails` | Deprecated (JDK 9+) | Use `-Xlog:gc*` instead. |
| `-Xloggc:<file>` | Deprecated (JDK 9+) | Use `-Xlog:gc:file=<file>` instead. |
| `System.gc()` | Discouraged | Suggestion only; may trigger full GC, causing pauses. |
| Parallel GC | Active | Default for JDK 8 server-class machines. |
| G1 GC | Active (Default) | Default collector since JDK 9. |
| ZGC | Active (Production) | Production-ready since JDK 21. Generational in JDK 21+. |
| Shenandoah | Active (Production) | Production-ready since JDK 15. |
| Soft References | Active | Cleared under memory pressure; not for critical data. |
| Weak References | Active | Cleared at next GC; ideal for canonicalizing mappings. |
| Phantom References | Active | Enqueued after finalization; `get()` always returns null. |

---

## References

### Official Specifications and Guides

- The Java Virtual Machine Specification, Java SE 26 Edition - https://docs.oracle.com/javase/specs/jvms/se26/html/index.html
- HotSpot Virtual Machine Garbage Collection Tuning Guide - https://docs.oracle.com/en/java/javase/26/gctuning/
- 3 Generations - Oracle Garbage Collection Tuning Guide - https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/generations.html
- 5 Available Collectors - Oracle Garbage Collection Tuning Guide - https://docs.oracle.com/en/java/javase/26/gctuning/available-collectors.html
- Garbage-First Garbage Collector Tuning - https://docs.oracle.com/en/java/javase/26/gctuning/garbage-first-garbage-collector-tuning.html
- Java Platform, Standard Edition Troubleshooting Guide - https://docs.oracle.com/en/java/javase/26/troubleshoot/

### API Documentation

- Package java.lang.ref - Java SE 23 - https://docs.oracle.com/en/java/javase/23/docs/api/java.base/java/lang/ref/package-summary.html
- ReferenceQueue - Java SE 23 - https://docs.oracle.com/en/java/javase/23/docs/api/java.base/java/lang/ref/ReferenceQueue.html
- Cleaner - Java SE 23 - https://docs.oracle.com/en/java/javase/23/docs/api/java.base/java/lang/ref/Cleaner.html

### OpenJDK JEPs

- JEP 189: Shenandoah: A Low-Pause Garbage Collector - https://openjdk.org/jeps/189
- JEP 333: ZGC: A Scalable Low-Latency Garbage Collector - https://openjdk.org/jeps/333
- JEP 376: ZGC: Concurrent Thread-Stack Processing - https://openjdk.org/jeps/376
- JEP 379: Shenandoah: A Low-Pause-Time Garbage Collector (Production) - https://openjdk.org/jeps/379

### Tutorials and Articles

- Introduction to Garbage Collection - Dev.java - https://dev.java/learn/jvm/tool/garbage-collection/intro/
- Garbage Collection in Java - Dev.java - https://dev.java/learn/jvm/tool/garbage-collection/
- Mark–Scavenge: Waiting for Trash to Take Itself Out - Inside.java - https://inside.java/2024/11/22/mark-scavenge/
- JVM Logging - Sip of Java - https://inside.java/2022/11/07/sip-of-java-jvm-logging/
- Lesson 3: Diagnostic Data Collection and Analysis Tools - Oracle - https://www.oracle.com/webfolder/technetwork/tutorials/mooc/JVM_Troubleshooting/week3/lesson3.pdf
- Java SafePoint: The Core Mechanism of JVM Pauses - https://developer.aliyun.com/article/1659014

### Academic Resources

- BestGC: An Automatic GC Selector - Zhao et al. - https://ieeexplore.ieee.org/
- Million-Request Java: A Decision Framework for JVM Tuning in Ultra-Low Latency Microservices - https://zenodo.org/
- Pretenuring in Java by Object Lifetime and Reference Density - https://ieeexplore.ieee.org/