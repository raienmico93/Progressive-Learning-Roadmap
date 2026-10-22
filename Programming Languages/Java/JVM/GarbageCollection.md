# Java Garbage Collection: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Garbage collection (GC) is the automatic process by which the Java Virtual Machine (JVM) identifies and reclaims memory occupied by objects that are no longer reachable by the application, eliminating the need for manual memory deallocation.

**Technical Definition:** Garbage collection is a form of automatic memory management. The garbage collector attempts to reclaim garbage, or memory occupied by objects that are no longer in use by the program. The JVM specification defines garbage collection as the process of reclaiming the runtime heap space occupied by unreachable objects. The Java Language Specification (JLS) states that “every object can be characterized by two attributes: it may be reachable, finalizer-reachable, or unreachable”. A reachable object is any object that can be accessed in any potential continuing computation from any live thread. Modern JVMs use generational collectors based on the weak generational hypothesis: most objects die young. The HotSpot VM offers multiple garbage collectors, each with distinct performance characteristics, including Serial, Parallel, G1, ZGC, and Shenandoah.

**Beginner-Friendly Explanation:** Imagine a warehouse (the heap) where workers (your program) create and use items (objects). Without a system, the warehouse would fill up with unused items until there's no space left. The garbage collector is an automatic cleanup crew. It scans the warehouse, identifies items that no longer have any labels (references) pointing to them, and removes them to free up space. You never need to tell the crew “remove this item” — it figures it out on its own. Different crews (collectors) work in different ways: some work alone, some work in parallel, and some work so quietly you barely notice them.

### Key Characteristics

- **Automatic:** No manual `free()` or `delete()` — the JVM handles reclamation.
- **Reachability-Based:** Objects are collected only when they are no longer reachable from GC roots.
- **Generational:** The heap is divided into young and old generations based on object age.
- **Stop-the-World Pauses:** Some collection phases pause all application threads; concurrent collectors minimise these pauses.
- **Multiple Algorithms:** Serial, Parallel, G1, ZGC, Shenandoah, and Epsilon, each targeting different goals.
- **Configurable:** Heap size, collector choice, and tuning parameters are controlled via JVM flags.
- **Finalization-Aware:** The GC interacts with finalizers, though their use is strongly discouraged.

### Prerequisites

- Understanding of the JVM runtime data areas (heap, stack, method area)
- Basic knowledge of object-oriented programming and references
- Familiarity with threads and stop-the-world concepts
- Ability to read JVM logs and use basic monitoring tools

### Related Programming Areas

- **JVM Architecture:** Memory layout, class loading, and execution engine.
- **Memory Model:** Happens-before, visibility, and object publication.
- **Performance Tuning:** Latency, throughput, and footprint optimisation.
- **Profiling:** Monitoring GC activity with `jcmd`, `jstat`, and GC logs.
- **Concurrent Programming:** Stop-the-world pauses affect all threads.

### Core Concepts / Features

---

## 1. Garbage Collection

### Definitions

**Core Definition:** Garbage collection is the automatic reclamation of heap memory occupied by objects that are no longer reachable by the application.

**Technical Definition:** The garbage collector identifies objects that are not transitively reachable from GC roots and reclaims their memory. The JVM specification defines several memory management characteristics: safety (never reclaim reachable objects), throughput (application performance), completeness (all garbage eventually reclaimed), promptness (how quickly garbage is reclaimed), and pause time (stop-the-world durations). The HotSpot VM uses a generational heap divided into young and old generations, with young collections (minor GC) occurring frequently and old collections (major/full GC) occurring rarely.

**Beginner-Friendly Explanation:** Garbage collection is the JVM’s automatic cleanup system. It finds objects that your program can no longer access and removes them to free memory. This prevents your program from running out of memory and eliminates the bugs associated with manual memory management.

### Purposes

- To automatically reclaim memory from unreachable objects
- To prevent memory leaks and `OutOfMemoryError`
- To eliminate use-after-free and double-free bugs
- To manage heap memory efficiently across object lifetimes
- To provide predictable performance through configurable collectors

### Syntax Rules and Structure

Garbage collection is triggered automatically by the JVM based on heap occupancy. The collector can be selected via JVM flags:

| Collector | Flag |
|-----------|------|
| Serial | `-XX:+UseSerialGC` |
| Parallel | `-XX:+UseParallelGC` |
| G1 (default) | `-XX:+UseG1GC` |
| ZGC | `-XX:+UseZGC` |
| Shenandoah | `-XX:+UseShenandoahGC` |
| Epsilon | `-XX:+UseEpsilonGC` |

**Syntax Rules:**

- The JVM decides when to collect based on allocation pressure and heap thresholds.
- `System.gc()` suggests collection but is not guaranteed and is generally discouraged.
- The choice of collector significantly affects pause times and throughput.

**Constraints and Limitations:**

- Stop-the-world pauses are unavoidable for some phases in most collectors.
- Finalizers are unpredictable and should be avoided.
- `System.gc()` can cause unnecessary full collections and should be disabled with `-XX:+DisableExplicitGC` in production.

### Annotated Complete Step-by-Step Code Examples

```java
// File: GCDemo.java
public class GCDemo {
    public static void main(String[] args) {
        // Allocate objects to trigger GC activity
        for (int i = 0; i < 10_000; i++) {
            byte[] data = new byte[1024]; // 1 KB object
            // The object becomes unreachable after each iteration
        }
        System.out.println("Allocation complete.");
        System.out.println("Free memory (bytes): " +
            Runtime.getRuntime().freeMemory());
    }
}
```

**Expected Output (varies):**
```
Allocation complete.
Free memory (bytes): 254803968
```

**Step-by-Step Explanation:**

1. The loop allocates 10,000 byte arrays, each 1 KB.
2. After each iteration, the previous array becomes unreachable.
3. The garbage collector may run during the loop to reclaim unreachable objects.
4. `freeMemory()` reports the free memory in the JVM heap after the allocation loop.

### Real-World Cases

- **Web Servers:** G1 handles large heaps with predictable pause times.
- **Real-Time Systems:** ZGC provides sub-millisecond pause times.
- **Batch Processing:** Parallel GC maximises throughput for offline jobs.

---

## 2. GC Roots and Reachability

### Definitions

**Core Definition:** GC roots are the starting points for reachability analysis; any object reachable from a GC root is considered live, while objects not reachable are candidates for collection.

**Technical Definition:** GC roots include local variables and operand stacks in stack frames, static fields, JNI handles, and VM-internal roots. The garbage collector traverses all objects reachable from these roots using depth-first or breadth-first search. An object is unreachable if it cannot be reached from any GC root.

**Beginner-Friendly Explanation:** GC roots are like the anchors of a spiderweb. If an object is connected to an anchor through a chain of references, it's alive. If the chain is broken and the object floats free, it's garbage.

### Purposes

- To determine which objects are still in use
- To identify candidates for garbage collection
- To ensure that reachable objects are never reclaimed (safety)

### Syntax Rules and Structure

**Common GC Roots:**

| Root Type | Description |
|-----------|-------------|
| Local variables | Variables in active stack frames |
| Static fields | Class-level variables |
| JNI references | Native code references |
| Active threads | Thread objects and their stacks |
| VM internal roots | JVM-held references |

### Annotated Code Example

```java
// File: ReachabilityDemo.java
public class ReachabilityDemo {
    private static Object staticRoot; // Static field — GC root

    public static void main(String[] args) {
        Object localRoot = new Object(); // Local variable — GC root
        staticRoot = new Object();       // Reachable via static field

        Object unreachable = new Object(); // Initially reachable
        unreachable = null;                // Now unreachable — eligible for GC

        System.out.println("Local root: " + localRoot);
        System.out.println("Static root: " + staticRoot);
    }
}
```

**Expected Output:**
```
Local root: java.lang.Object@...
Static root: java.lang.Object@...
```

### Real-World Cases

- **Caching:** Static caches hold strong references to cached objects, preventing collection.
- **Thread-Local Variables:** Variables in thread stacks are GC roots.
- **JNI:** Native code can create GC roots via JNI handles.

---

## 3. Mark and Sweep

### Definitions

**Core Definition:** Mark and Sweep is a tracing garbage collection algorithm that marks all reachable objects and then sweeps the heap to reclaim unmarked objects.

**Technical Definition:** The mark phase traverses all objects reachable from GC roots and marks them as live. The sweep phase scans the heap linearly, collecting unmarked objects and returning their memory to the free list. The algorithm does not compact memory, so it can lead to fragmentation.

**Beginner-Friendly Explanation:** Mark and Sweep is like checking every item in a warehouse: first, you put a sticker on every item that's still needed (mark), then you remove everything without a sticker (sweep). The downside is that removing items leaves gaps.

### Purposes

- To reclaim memory from unreachable objects
- To provide a simple, foundational tracing algorithm
- To serve as the basis for more advanced collectors

### Syntax Rules and Structure

```
1. Mark phase: Traverse from roots, mark reachable objects.
2. Sweep phase: Scan heap, free unmarked objects.
```

**Constraints and Limitations:**

- Fragmentation: free memory is not contiguous.
- Slower allocation due to free-list management.

### Annotated Code Example

The algorithm is internal; the following demonstrates fragmentation conceptually.

```java
// File: FragmentationDemo.java
public class FragmentationDemo {
    public static void main(String[] args) {
        // Allocate objects, free some (GC), leaving gaps
        Object[] objects = new Object[10];
        for (int i = 0; i < 10; i++) {
            objects[i] = new byte[1024 * 1024]; // 1 MB each
        }
        // Free every other object
        for (int i = 0; i < 10; i += 2) {
            objects[i] = null;
        }
        System.gc(); // Fragmentation may occur
        System.out.println("Fragmented heap after GC");
    }
}
```

### Real-World Cases

- **CMS (Deprecated):** Used mark-sweep for old generation collections.
- **Embedded Systems:** Simple mark-sweep collectors are used in memory-constrained environments.

---

## 4. Mark and Compact

### Definitions

**Core Definition:** Mark and Compact is a tracing algorithm that marks reachable objects and then compacts them into a contiguous block, eliminating fragmentation.

**Technical Definition:** After marking, the compact phase slides live objects to one end of the heap, updating references and freeing a contiguous block of memory. This eliminates fragmentation and makes allocation fast (pointer bumping). The cost is increased pause time due to object movement.

**Beginner-Friendly Explanation:** Mark and Compact is like Mark and Sweep, but after removing garbage, it slides all remaining items together so there are no gaps. This makes future placements faster, but rearranging takes time.

### Purposes

- To eliminate memory fragmentation
- To enable fast, contiguous allocation
- To provide predictable memory layout

### Syntax Rules and Structure

```
1. Mark phase: Mark reachable objects.
2. Compact phase: Slide live objects to one end.
3. Update all references to moved objects.
```

**Constraints and Limitations:**

- Longer pause times due to object movement.
- Requires updating all references to moved objects.

### Annotated Code Example

```java
// File: CompactionDemo.java
public class CompactionDemo {
    public static void main(String[] args) {
        // Allocate objects, free some, then compact
        Object[] objects = new Object[100];
        for (int i = 0; i < 100; i++) {
            objects[i] = new byte[1024];
        }
        for (int i = 0; i < 100; i += 2) {
            objects[i] = null;
        }
        System.gc(); // Compaction may occur
        System.out.println("Heap compacted after GC");
    }
}
```

### Real-World Cases

- **Serial Old:** Uses mark-compact for old generation collections.
- **G1:** Uses compaction during evacuation pauses.

---

## 5. Copying Collection

### Definitions

**Core Definition:** Copying collection divides the heap into two semi-spaces; live objects are copied from the active space to the inactive space, and the active space is cleared entirely.

**Technical Definition:** The heap is divided into two equal-sized semi-spaces (from-space and to-space). During collection, live objects are copied from from-space to to-space, and from-space is reclaimed as a contiguous free block. The roles of the spaces are then swapped. This algorithm has no fragmentation but wastes half the heap.

**Beginner-Friendly Explanation:** Copying collection is like moving all your furniture from one room to another and then cleaning the first room completely. The new room is perfectly organised, but you can only use half your house at a time.

### Purposes

- To eliminate fragmentation entirely
- To enable extremely fast allocation (pointer bumping)
- To provide excellent performance for young generations where most objects die

### Syntax Rules and Structure

```
1. Traverse from roots, copying live objects to to-space.
2. Swap from-space and to-space.
3. Clear from-space (now the old to-space).
```

**Constraints and Limitations:**

- Requires double the heap space (50% overhead).
- Copying costs increase with the number of live objects.

### Annotated Code Example

```java
// File: CopyingGCDemo.java
public class CopyingGCDemo {
    public static void main(String[] args) {
        // In a copying collector, the young generation uses a copying algorithm.
        // The JVM handles this automatically; this is conceptual.
        for (int i = 0; i < 1000; i++) {
            Object obj = new Object();
        }
        System.gc();
        System.out.println("Copying collection completed in young gen");
    }
}
```

### Real-World Cases

- **Young Generation:** Most collectors use copying for the young generation (Eden + Survivor spaces).
- **Semi-Space Collectors:** Used in research and some embedded JVMs.

---

## 6. Generational GC

### Definitions

**Core Definition:** Generational GC is a strategy that divides the heap into generations based on object age, collecting younger generations more frequently and older generations less often.

**Technical Definition:** The weak generational hypothesis states that most objects die young, and few references exist from older to younger objects. The heap is divided into a young generation (frequently collected) and an old generation (rarely collected). Young collections (minor GC) are fast and frequent; old collections (major/full GC) are slow and rare.

**Beginner-Friendly Explanation:** Generational GC is like sorting laundry by how dirty it is. The “young” pile (new objects) is washed frequently because it gets dirty quickly. The “old” pile (long-lived objects) is washed less often. This saves time and energy.

### Purposes

- To optimise collection frequency based on object lifetime
- To minimise pause times by collecting only the young generation most of the time
- To improve throughput by avoiding scanning the entire heap frequently

### Syntax Rules and Structure

**Heap Layout:**

| Generation | Description |
|------------|-------------|
| Young | Eden + two Survivor spaces |
| Old (Tenured) | Long-lived objects |

**Collection Types:**

| Type | Scope | Frequency |
|------|-------|-----------|
| Minor GC | Young generation | Frequent |
| Major/Full GC | Whole heap | Rare |

### Annotated Code Example

```java
// File: GenerationalDemo.java
public class GenerationalDemo {
    public static void main(String[] args) {
        // Short-lived objects: allocated in Eden, collected in minor GC
        for (int i = 0; i < 10000; i++) {
            byte[] temp = new byte[1024]; // Dies quickly
        }

        // Long-lived objects: promoted to old generation
        Object[] longLived = new Object[10];
        for (int i = 0; i < 10; i++) {
            longLived[i] = new byte[1024 * 1024];
        }

        System.gc(); // Minor or full GC may occur
        System.out.println("Generational GC completed");
    }
}
```

### Real-World Cases

- **Default JVM Behaviour:** Most collectors (Serial, Parallel, G1) use generational collection.
- **Web Applications:** Short-lived request objects are collected quickly in the young generation.

---

## 7. Young Generation, Old Generation, Eden and Survivor Spaces

### Definitions

**Core Definition:** The young generation is where new objects are allocated; it is divided into Eden and two Survivor spaces. The old generation holds objects that have survived multiple young collections.

**Technical Definition:** New objects are allocated in Eden. When Eden fills, a minor GC occurs: live objects are copied to a Survivor space, and Eden is cleared. Objects that survive multiple minor GCs are promoted to the old generation. The typical layout is Eden:Survivor0:Survivor1 = 8:1:1.

**Beginner-Friendly Explanation:** Eden is the newborn ward of the heap. Survivor spaces are the recovery rooms. Objects that keep surviving get promoted to the old generation — the retirement home of the heap.

### Purposes

- To isolate short-lived objects for frequent, fast collection
- To reduce the cost of collecting long-lived objects
- To enable efficient copying-based collection for young objects

### Syntax Rules and Structure

| Space | Purpose |
|-------|---------|
| Eden | New object allocation |
| Survivor 0 / 1 | Hold survivors between minor GCs; one is always empty |
| Old Generation | Long-lived objects promoted from young |

**Collection Process:**

1. New objects allocated in Eden.
2. Eden fills → minor GC triggered.
3. Live Eden objects copied to empty Survivor space.
4. Survivor objects age; after threshold, promoted to Old.
5. Eden and the previously used Survivor space are cleared.

### Annotated Code Example

```java
// File: EdenSurvivorDemo.java
public class EdenSurvivorDemo {
    public static void main(String[] args) {
        // Allocate many short-lived objects in Eden
        for (int i = 0; i < 100_000; i++) {
            new Object(); // Most die in Eden
        }

        // Long-lived objects survive and are promoted
        Object survivor = new Object();
        for (int i = 0; i < 10; i++) {
            System.gc(); // Force minor GCs; survivor promoted
        }

        System.out.println("Survivor object: " + survivor);
    }
}
```

### Real-World Cases

- **Request Handling:** HTTP request objects are allocated in Eden and die after the request completes.
- **Session Objects:** Long-lived sessions are promoted to the old generation.

---

## 8. GC Algorithms

### Definitions

**Core Definition:** Java provides multiple garbage collection algorithms, each optimised for different performance goals: pause time, throughput, or footprint.

**Technical Definition:** The HotSpot VM offers Serial, Parallel, CMS (deprecated/removed), G1, ZGC, Shenandoah, and Epsilon collectors. G1 is the default since JDK 9. ZGC and Shenandoah target ultra-low pause times through concurrent compaction. Epsilon performs no collection, useful for benchmarking.

**Beginner-Friendly Explanation:** Different collectors are like different cleaning crews. The Serial crew is one person — slow but uses minimal supplies. The Parallel crew is a team — fast but noisy. The G1 crew is a smart team that cleans the dirtiest areas first. ZGC and Shenandoah are ninja crews — you barely notice them. Epsilon does nothing at all.

### Key Algorithms Table

| Collector | Status | Target | Flag |
|-----------|--------|--------|------|
| Serial | Supported | Small footprint | `-XX:+UseSerialGC` |
| Parallel | Supported | High throughput | `-XX:+UseParallelGC` |
| CMS | Deprecated (removed JDK 14) | Low latency | `-XX:+UseConcMarkSweepGC` |
| G1 | Supported (default) | Balanced, large heaps | `-XX:+UseG1GC` |
| ZGC | Supported | Low latency, huge heaps | `-XX:+UseZGC` |
| Shenandoah | Supported | Low latency, consistent | `-XX:+UseShenandoahGC` |
| Epsilon | Experimental | No collection | `-XX:+UseEpsilonGC` |

**Sources:** 

### Purposes

- To match collector characteristics to application requirements
- To minimise pause times for latency-sensitive applications
- To maximise throughput for batch applications
- To reduce memory footprint for constrained environments

### Annotated Code Example (Selecting a Collector)

```java
// Run with: java -XX:+UseG1GC MyApp
// Run with: java -XX:+UseZGC MyApp
// Run with: java -XX:+UseSerialGC MyApp
public class CollectorSelectionDemo {
    public static void main(String[] args) {
        System.out.println("GC: " +
            java.lang.management.ManagementFactory.getGarbageCollectorMXBeans());
    }
}
```

**Expected Output (varies by flag):**
```
GC: [G1 Young Generation, G1 Old Generation]
```

### Real-World Cases

- **Trading Systems:** ZGC for sub-millisecond pauses.
- **Batch ETL:** Parallel GC for maximum throughput.
- **Small Containers:** Serial GC for minimal overhead.

---

## 9. GC Tuning

### Definitions

**Core Definition:** GC tuning is the process of configuring the JVM’s heap size, collector choice, and collector-specific parameters to meet performance goals.

**Technical Definition:** Key tuning parameters include heap size (`-Xms`, `-Xmx`), young generation size, survivor space ratio, and collector-specific flags. The most important tuning option for ZGC is setting the maximum heap size via `-Xmx`. For G1, tuning focuses on pause-time goals and heap sizing.

**Beginner-Friendly Explanation:** GC tuning is like adjusting the thermostat and cleaning schedule for a building. You set the rules (heap size) and choose the crew (collector), then fine-tune based on whether you care more about quiet operation (low pause) or getting the job done fast (throughput).

### Purposes

- To meet latency or throughput goals
- To avoid `OutOfMemoryError`
- To reduce GC overhead
- To optimise heap utilisation

### Key Flags Table

| Flag | Description |
|------|-------------|
| `-Xms` | Initial heap size |
| `-Xmx` | Maximum heap size |
| `-XX:MaxGCPauseMillis` | Target maximum pause time (G1) |
| `-XX:GCTimeRatio` | Throughput goal |
| `-XX:SurvivorRatio` | Eden-to-survivor ratio |
| `-XX:MaxTenuringThreshold` | Promotions threshold |

**Syntax Rules:**

- Set `-Xms` and `-Xmx` to the same value for server applications to avoid heap resizing.
- Use `-XX:MaxGCPauseMillis` for latency-sensitive applications.
- Monitor GC logs to validate tuning decisions.

### Annotated Code Example

```java
// File: GCTuningDemo.java
public class GCTuningDemo {
    public static void main(String[] args) {
        // Run with: java -Xms512m -Xmx512m -XX:+UseG1GC GCTuningDemo
        Runtime runtime = Runtime.getRuntime();
        System.out.println("Max heap: " + runtime.maxMemory() / (1024 * 1024) + " MB");
        System.out.println("Total heap: " + runtime.totalMemory() / (1024 * 1024) + " MB");
    }
}
```

**Expected Output:**
```
Max heap: 512 MB
Total heap: 512 MB
```

### Real-World Cases

- **Low-Latency Trading:** `-XX:MaxGCPauseMillis=5` with G1 or ZGC.
- **High-Throughput Batch:** Parallel GC with large heap and high `GCTimeRatio`.

---

## 10. GC Logs

### Definitions

**Core Definition:** GC logs record garbage collection events, including pause times, heap occupancy, and collection types, enabling diagnosis and tuning.

**Technical Definition:** Modern JVMs use the Unified Logging framework. Key flags include `-Xlog:gc`, `-Xlog:gc*`, and `-Xlog:gc+heap=debug`. Logs show young and full collections, pause durations, and heap usage before and after collection.

**Beginner-Friendly Explanation:** GC logs are the collector’s diary. They tell you when it ran, how long it took, and how much memory it freed. Reading them helps you spot problems like frequent full GCs or long pauses.

### Purposes

- To diagnose GC performance issues
- To validate tuning changes
- To detect memory leaks (steadily increasing heap usage)
- To identify excessive full GC activity

### Key Logging Flags Table

| Flag | Description |
|------|-------------|
| `-Xlog:gc` | Basic GC logging |
| `-Xlog:gc*` | Detailed GC logging |
| `-Xlog:gc+heap=debug` | Heap occupancy details |
| `-Xloggc:<file>` | Redirect logs to a file |

### Annotated Code Example

```java
// File: GCLogDemo.java
public class GCLogDemo {
    public static void main(String[] args) {
        // Run with: java -Xlog:gc -Xmx64m GCLogDemo
        for (int i = 0; i < 1_000_000; i++) {
            byte[] data = new byte[1024];
        }
        System.out.println("Done");
    }
}
```

**Expected Output (log excerpt):**
```
[0.123s][info][gc] Using G1
[0.456s][info][gc] GC(0) Pause Young (Normal) (G1 Evacuation Pause) 15M->2M(64M) 5.678ms
Done
```

### Real-World Cases

- **Production Monitoring:** `-Xlog:gc*:file=gc.log:time,uptime,level,tags` for detailed logs.
- **Incident Analysis:** GC logs reveal pause spikes and memory pressure.

---

## 11. GC Best Practices

### Definitions

**Core Definition:** GC best practices are guidelines for configuring, monitoring, and working with the garbage collector to achieve optimal application performance.

**Technical Definition:** Best practices derive from Oracle’s HotSpot GC Tuning Guide. Key principles include choosing the right collector for the workload, sizing the heap appropriately, enabling GC logging, avoiding `System.gc()`, and understanding the trade-offs between latency and throughput.

**Beginner-Friendly Explanation:** GC best practices help you get the most out of the JVM without guessing. They prevent common mistakes like using the wrong collector, setting the heap too small, or ignoring GC logs.

### Key Best Practices Table

| Practice | Description |
|----------|-------------|
| Choose the Right Collector | Match collector to workload (latency vs. throughput). |
| Size the Heap Appropriately | Set `-Xms` = `-Xmx` for servers; avoid resizing overhead. |
| Enable GC Logging | Always run with `-Xlog:gc*` in production for diagnosis. |
| Avoid `System.gc()` | Rely on JVM heuristics; disable explicit GC with `-XX:+DisableExplicitGC`. |
| Monitor Metaspace | Set `-XX:MaxMetaspaceSize` to prevent unbounded class metadata growth. |
| Understand Generational Hypothesis | Most objects die young; tune young generation accordingly. |
| Prefer G1 for Most Workloads | G1 is the default and balances pause and throughput. |
| Use ZGC for Ultra-Low Latency | ZGC provides sub-millisecond pauses for large heaps. |
| Avoid Finalizers | Use `Cleaner` or try-with-resources instead. |
| Profile Before Tuning | Use `jcmd`, `jstat`, and GC logs to identify bottlenecks. |

### Annotated Code Example

```java
// File: GCBestPractices.java
public class GCBestPractices {
    public static void main(String[] args) {
        // BEST PRACTICE: Monitor heap usage
        Runtime runtime = Runtime.getRuntime();
        long max = runtime.maxMemory();
        long total = runtime.totalMemory();
        long free = runtime.freeMemory();

        System.out.println("Max   : " + max / (1024 * 1024) + " MB");
        System.out.println("Total : " + total / (1024 * 1024) + " MB");
        System.out.println("Free  : " + free / (1024 * 1024) + " MB");
        System.out.println("Used  : " + (total - free) / (1024 * 1024) + " MB");

        // BEST PRACTICE: Do not call System.gc()
        // BEST PRACTICE: Let the JVM choose collection timing
    }
}
```

**Expected Output (varies):**
```
Max   : 4096 MB
Total : 256 MB
Free  : 240 MB
Used  : 16 MB
```

### Real-World Cases

- **Cloud Deployments:** Container-aware JVM settings prevent OOM kills.
- **Low-Latency Systems:** Tuning G1 or ZGC for sub-millisecond pauses.
- **Batch Jobs:** Parallel GC for maximum throughput.

---

## References Links

- HotSpot Virtual Machine Garbage Collection Tuning Guide — Oracle - https://docs.oracle.com/en/java/javase/24/gctuning/toc.htm#1
- Chapter 12. Execution — Java Language Specification - https://docs.oracle.com/javase/specs/jls/se7/html/jls-12.html#jls-12.5
- Java Garbage Collectors (GC) and How to Switch Between Them — GitHub - https://github.com/sadiul-hakim/Hakim-Code-Collection/blob/main/GC.md#1
- Effective Software Lecture 13: Memory Management in JVM — Czech Technical University - https://esw.pages.fel.cvut.cz/pdfs/ESW13_2025.pdf#1#1
- Memory Management in the JVM — Czech Technical University - https://esw.pages.fel.cvut.cz/lectures/ESW13_2026.pdf#1#1
- In generation garbage collection, memory is divided into generations — Aalto University - https://aaltodoc.aalto.fi/server/api/core/bitstreams/23e75cdd-0ef5-418a-b547-a57995169f4b/content#11#6
- MET12-J. Do not use finalizers — SEI CERT - https://wiki.sei.cmu.edu/confluence/display/java/MET12-J.+Do+not+use+finalizers