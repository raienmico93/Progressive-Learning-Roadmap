# JVM Tuning: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** JVM tuning is the systematic process of configuring the Java Virtual Machine's memory layout, garbage collector, and just-in-time compiler to meet an application's performance goals — typically some balance of **throughput** (work done per unit time), **latency** (response time), and **footprint** (memory consumed).

**Technical Definition:** JVM tuning involves adjusting the JVM's runtime parameters — primarily heap sizing (`-Xms`, `-Xmx`), thread stack sizing (`-Xss`), Metaspace limits, garbage collector selection and goals, and JIT compilation thresholds — based on observed workload characteristics. The HotSpot VM's ergonomics automatically size the heap and select a collector based on the detected environment, but explicit tuning is required when application behaviour deviates from the ergonomic assumptions or when specific latency/throughput targets must be met. The discipline emphasises a measure-first workflow: observe via monitoring tools, identify the bottleneck, apply a targeted change, and validate against a quantified baseline.

**Beginner-Friendly Explanation:** Think of the JVM as a delivery company. **Heap sizing** decides how big the warehouse is. **Stack sizing** decides how much desk space each worker gets. **Metaspace** is the filing cabinet for blueprints. **GC selection** chooses which cleaning crew empties the warehouse, and **GC tuning** sets their rules (e.g., "never take longer than 200 ms to clean"). **JIT flags** decide how eagerly the company teaches workers to do tasks faster. **Monitoring tools** are the security cameras and dashboards that tell you what is actually happening. Tuning is not about flipping every dial; it is about watching the system, finding the one bottleneck that matters, and fixing that — then repeating. The most common mistake is tuning blindly without measurement, or tuning to fix a problem that is really in the application code rather than the JVM.

### Key Characteristics

- **Measure-First Discipline:** Tuning without a quantified baseline is guesswork. "先压测立基准，无量化不调优" — benchmark first; without quantification, do not tune.
- **Trade-Offs, Not Magic:** Pause time, throughput, and footprint compete. Reducing pause times often reduces throughput; shrinking the heap often increases GC frequency.
- **Ergonomics as a Baseline:** The JVM's default sizing and collector selection are often adequate. Override defaults only with evidence.
- **Generational Structure:** The heap is divided into young and old generations; most tuning decisions ultimately influence object promotion and collection frequency.
- **Version-Sensitive:** Defaults change across Java versions. G1 became the default collector in Java 9; ZGC and Shenandoah became production-ready in Java 15; generational ZGC arrived in Java 21.
- **Container-Aware:** Modern JVMs respect container memory and CPU limits via `-XX:+UseContainerSupport`, which is critical for cloud deployments.

### Prerequisites

- Basic understanding of JVM architecture (heap, stack, Metaspace, class loading)
- Familiarity with garbage collection concepts (young/old generations, stop-the-world pauses)
- Knowledge of the Java compilation and execution model (interpretation, JIT)
- Access to monitoring tools and the ability to read their output

### Related Programming Areas

- **JVM Architecture:** Memory areas, class loading, and execution engine.
- **Garbage Collection:** Collector algorithms and their performance characteristics.
- **JIT Compilation:** Tiered compilation, inlining, and warm-up behaviour.
- **Performance Engineering:** Benchmarking with JMH, profiling, and load testing.
- **Container Orchestration:** Memory/CPU limits in Kubernetes and Docker.

### Core Concepts / Features

---

## 1. Heap Sizing (`-Xms` / `-Xmx`)

### Definitions

**Core Definition:** Heap sizing sets the initial (`-Xms`) and maximum (`-Xmx`) amount of memory the JVM may use for object allocation.

**Technical Definition:** "The minimum and maximum heap sizes that the garbage collector can use can be set using `-Xms=<nnn>` and `-Xmx=<mmm>` for minimum and maximum heap size respectively". The heap grows or shrinks between these bounds as the collector attempts to satisfy throughput, pause-time, and footprint goals. Setting `-Xms` equal to `-Xmx` fixes the heap size and is a common production practice for server applications to avoid runtime resizing overhead and to make memory usage predictable.

**Beginner-Friendly Explanation:** `-Xms` is how big the warehouse is when the company opens; `-Xmx` is the maximum size it is allowed to grow to. If you set them to the same value, the warehouse is fixed from day one — no expansion or contraction, which makes planning easier and avoids surprises.

### Purposes

- To control the total memory footprint of the JVM
- To prevent excessive GC frequency (heap too small) or wasteful memory use (heap too large)
- To make memory usage predictable in containerised environments
- To provide the collector with the room it needs to meet throughput and pause-time goals

### Syntax Rules and Structure

```java
-Xms<size>    // Initial heap size (e.g., -Xms2g)
-Xmx<size>    // Maximum heap size (e.g., -Xmx2g)
```

**Component Breakdown:**

| Flag | Description |
|------|-------------|
| `-Xms` | Minimum/initial heap size; the heap starts at this size. |
| `-Xmx` | Maximum heap size; the heap will not grow beyond this. |

**Syntax Rules:**

- Sizes may use suffixes `k`, `m`, `g` (e.g., `512m`, `2g`).
- If only `-Xmx` is set, `-Xms` defaults to a fraction of it (ergonomically determined).
- Setting `-Xms` equal to `-Xmx` fixes the heap size.

**Constraints and Limitations:**

- "Don't choose a maximum value for the heap unless you know that you need a heap greater than the default maximum heap size".
- Heap must leave room for non-heap memory (Metaspace, thread stacks, code cache, direct buffers). "Maintain appropriate non-heap headroom (roughly 40% of total memory)".
- In containers, the JVM should be container-aware so it does not size the heap larger than the container limit allows.

### Annotated Complete Step-by-Step Code Examples

```java
// File: HeapSizingDemo.java
public class HeapSizingDemo {
    public static void main(String[] args) {
        // Run with: java -Xms256m -Xmx256m HeapSizingDemo
        Runtime rt = Runtime.getRuntime();
        System.out.println("Max heap (bytes):   " + rt.maxMemory());
        System.out.println("Total heap (bytes): " + rt.totalMemory());
        System.out.println("Free heap (bytes):  " + rt.freeMemory());
    }
}
```

**Expected Output (varies):**
```
Max heap (bytes):   268435456
Total heap (bytes): 268435456
Free heap (bytes):  266338304
```

**Step-by-Step Explanation:**

1. `-Xms256m` and `-Xmx256m` fix the heap at 256 MB (268,435,456 bytes).
2. `maxMemory()` returns the maximum heap size (`-Xmx`).
3. `totalMemory()` returns the currently allocated heap size; with `-Xms` = `-Xmx`, it equals the fixed size.
4. `freeMemory()` reports unused space within the allocated heap.

### Real-World Cases

- **High-throughput batch jobs:** Large fixed heap to minimise GC frequency and maximise throughput.
- **Latency-sensitive services:** Heap sized to leave headroom for non-heap memory and to allow the collector to meet pause goals.
- **Containerised microservices:** `-XX:MaxRAMPercentage` (or `-XX:MaxRAMFraction`) used instead of hard-coded `-Xmx` to adapt to container limits.

---

## 2. Stack Sizing (`-Xss`)

### Definitions

**Core Definition:** Stack sizing sets the size of each thread's stack, which stores local variables, method frames, and operand stacks.

**Technical Definition:** `-Xss` controls the per-thread stack size. Each Java thread has its own stack; the total native memory consumed by stacks is approximately `thread count × -Xss`. Stack size must be large enough to accommodate the deepest call chain (including recursion) without throwing `StackOverflowError`.

**Beginner-Friendly Explanation:** The stack is each worker's personal desk. If the desk is too small, a worker doing a deep task (like deep recursion) runs out of space and crashes (`StackOverflowError`). If the desk is too large, each worker takes up too much floor space and you cannot fit enough workers in the building.

### Purposes

- To prevent `StackOverflowError` in applications with deep call chains or recursion
- To control the total native memory consumed by thread stacks
- To support applications with many threads (each thread consumes stack space)

### Syntax Rules and Structure

```java
-Xss<size>    // Thread stack size (e.g., -Xss512k, -Xss1m)
```

**Component Breakdown:**

| Flag | Description |
|------|-------------|
| `-Xss` | Per-thread stack size. |

**Syntax Rules:**

- Default varies by platform (typically 512 KB to 1 MB).
- Size suffixes `k`, `m` are accepted.
- The total stack memory is `thread count × -Xss` plus overhead.

**Constraints and Limitations:**

- Increasing `-Xss` reduces the maximum number of threads that can be created for a given amount of native memory.
- Virtual threads (Java 21+) have much smaller stack footprints and are not affected by `-Xss` in the same way.

### Annotated Code Example

```java
// File: StackSizingDemo.java
public class StackSizingDemo {
    static int depth = 0;

    static void recurse() {
        depth++;
        recurse(); // Deep recursion
    }

    public static void main(String[] args) {
        // Run with: java -Xss256k StackSizingDemo
        try {
            recurse();
        } catch (StackOverflowError e) {
            System.out.println("Stack overflow at depth: " + depth);
        }
    }
}
```

**Expected Output (varies):**
```
Stack overflow at depth: 1587
```

**Step-by-Step Explanation:**

1. `-Xss256k` sets a small stack, so the recursion overflows at a relatively shallow depth.
2. `recurse()` calls itself indefinitely; each call consumes a stack frame.
3. When the stack is exhausted, `StackOverflowError` is thrown and caught.
4. Increasing `-Xss` would allow deeper recursion before overflowing.

### Real-World Cases

- **Deep parsers / recursive algorithms:** Larger `-Xss` to accommodate deep recursion.
- **Thread-heavy applications:** Smaller `-Xss` to allow more threads within memory limits.
- **Virtual-thread applications:** Stack sizing is largely irrelevant; virtual threads use heap-allocated continuation stacks.

---

## 3. Metaspace Sizing (`-XX:MetaspaceSize` / `-XX:MaxMetaspaceSize`)

### Definitions

**Core Definition:** Metaspace sizing controls the memory used for class metadata (the replacement for PermGen since Java 8).

**Technical Definition:** "`-XX:MetaspaceSize` sets the initial Metaspace size. `-XX:MaxMetaspaceSize` sets the maximum Metaspace size". Metaspace resides in native memory, not the Java heap. If `MaxMetaspaceSize` is not set, Metaspace can grow until native memory is exhausted, which is dangerous in containers. Setting the initial size close to the expected peak avoids frequent expensive Metaspace expansions.

**Beginner-Friendly Explanation:** Metaspace is the filing cabinet for class blueprints. If you let it grow without limit, it can consume all the building's floor space. Set a maximum so that a runaway class loader (or a leak) is caught early rather than taking down the whole container.

### Purposes

- To bound class metadata memory and prevent Metaspace-related `OutOfMemoryError`
- To avoid frequent Metaspace expansion (GC and class unloading) by setting an appropriate initial size
- To make native memory usage predictable in container environments

### Syntax Rules and Structure

```java
-XX:MetaspaceSize=<size>       // Initial Metaspace size (trigger for first GC)
-XX:MaxMetaspaceSize=<size>    // Maximum Metaspace size
```

**Component Breakdown:**

| Flag | Description |
|------|-------------|
| `-XX:MetaspaceSize` | Initial high-water mark; the first Metaspace GC is triggered around this size. |
| `-XX:MaxMetaspaceSize` | Hard upper limit; `OutOfMemoryError: Metaspace` is thrown if exceeded. |

**Syntax Rules:**

- Sizes accept `k`, `m`, `g` suffixes.
- If `MaxMetaspaceSize` is unset, Metaspace is unbounded (limited only by native memory).
- `-XX:CompressedClassSpaceSize` separately bounds the compressed class space (a portion of Metaspace).

**Constraints and Limitations:**

- Metaspace memory is **not** reclaimed by heap GC; it is reclaimed when class loaders become unreachable and classes are unloaded.
- Setting `MaxMetaspaceSize` too low can cause `OutOfMemoryError: Metaspace` during legitimate class loading (e.g., application startup with many frameworks).

### Annotated Code Example

```java
// File: MetaspaceDemo.java
public class MetaspaceDemo {
    public static void main(String[] args) throws Exception {
        // Run with: java -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=256m MetaspaceDemo
        // Continuously generate classes (conceptual — requires a class generator)
        // In practice, frameworks, proxies, and scripting engines generate classes.
        System.out.println("Metaspace flags active");
    }
}
```

**Expected Output:**
```
Metaspace flags active
```

**Step-by-Step Explanation:**

1. `-XX:MetaspaceSize=128m` sets the initial high-water mark for Metaspace.
2. `-XX:MaxMetaspaceSize=256m` caps Metaspace growth.
3. If class loading exceeds 256 MB of metadata, `OutOfMemoryError: Metaspace` is thrown.

### Real-World Cases

- **Application servers:** Many deployed applications generate classes (JSPs, proxies, lambdas); Metaspace limits prevent one application from exhausting native memory.
- **Dynamic languages on the JVM:** Scripting engines generate many classes; Metaspace monitoring is essential.
- **Containers:** Without `MaxMetaspaceSize`, a leak can cause the container to be OOM-killed rather than throwing a JVM-level error.

---

## 4. GC Selection

### Definitions

**Core Definition:** GC selection chooses the garbage collection algorithm that best matches the application's throughput, latency, and footprint requirements.

**Technical Definition:** The HotSpot VM provides several collectors: Serial, Parallel, G1, ZGC, Shenandoah, and Epsilon. G1 is the default since Java 9 and balances throughput and pause time for general-purpose server workloads. ZGC and Shenandoah target ultra-low pauses (typically sub-millisecond to sub-10-millisecond) for latency-sensitive applications, at the cost of higher CPU and memory overhead. Parallel GC maximises throughput for batch workloads where pause time is unimportant.

**Beginner-Friendly Explanation:** Different cleaning crews work in different ways. G1 is the all-rounder — good at most things. ZGC and Shenandoah are the ninja crews — you barely notice them, but they cost more. Parallel is the heavy-duty crew — it gets the job done fastest overall but may take long breaks (pauses). Serial is the one-person crew for small jobs.

### Purposes

- To match collector behaviour to workload characteristics
- To meet specific pause-time or throughput targets
- To avoid the pitfalls of a mismatched collector (e.g., long pauses in a latency-critical service)

### Syntax Rules and Structure

**Collector Flags:**

| Collector | Flag | Best For |
|-----------|------|----------|
| G1 (default) | `-XX:+UseG1GC` | Most server workloads; balanced pause/throughput |
| ZGC | `-XX:+UseZGC` | Ultra-low latency; large heaps (TB-scale) |
| Shenandoah | `-XX:+UseShenandoahGC` | Low latency; concurrent compaction |
| Parallel | `-XX:+UseParallelGC` | Maximum throughput; batch jobs |
| Serial | `-XX:+UseSerialGC` | Small heaps, single-CPU environments |
| Epsilon | `-XX:+UseEpsilonGC` | Benchmarking; no-op GC |

**Syntax Rules:**

- Only one collector flag should be specified; the last one wins if multiple are provided.
- Collector availability depends on JDK version and platform.
- Epsilon requires `-XX:+UnlockExperimentalVMOptions`.

**Constraints and Limitations:**

- ZGC and Shenandoah have higher CPU and memory overhead than G1.
- G1's `-XX:MaxGCPauseMillis` is a soft goal; it may not always be met.
- ZGC's pause times are largely independent of heap size, but its throughput is generally lower than G1's.

### Annotated Code Example

```java
// File: GCSelectionDemo.java
public class GCSelectionDemo {
    public static void main(String[] args) {
        // Run with: java -XX:+UseG1GC GCSelectionDemo
        // Run with: java -XX:+UseZGC GCSelectionDemo
        // Run with: java -XX:+UseParallelGC GCSelectionDemo
        var beans = java.lang.management.ManagementFactory.getGarbageCollectorMXBeans();
        beans.forEach(b -> System.out.println("GC: " + b.getName()));
    }
}
```

**Expected Output (varies):**
```
GC: G1 Young Generation
GC: G1 Old Generation
```

**Step-by-Step Explanation:**

1. The JVM starts with the collector specified by the flag.
2. `ManagementFactory.getGarbageCollectorMXBeans()` exposes the active collectors.
3. The output confirms which collector was selected.

### Real-World Cases

- **Low-latency microservices:** ZGC or Shenandoah for sub-10-ms pauses.
- **Batch ETL pipelines:** Parallel GC for maximum throughput.
- **General-purpose web services:** G1 as the default balanced choice.
- **Small CLI tools:** Serial GC for minimal overhead.

---

## 5. GC Tuning (`-XX:MaxGCPauseMillis` / `-XX:GCTimeRatio`)

### Definitions

**Core Definition:** GC tuning sets the collector's goals for maximum pause time and throughput, allowing the JVM to adjust heap sizing and collection behaviour to meet them.

**Technical Definition:** `-XX:MaxGCPauseMillis=<nnn>` is "interpreted as a hint to the garbage collector that a pause-time of `<nnn>` milliseconds or fewer is desired". `-XX:GCTimeRatio=<nnn>` sets the ratio of garbage collection time to application time to `1 / (1 + nnn)`; for example, `GCTimeRatio=19` targets 5% of total time in GC. The collector balances these competing goals with the heap size, typically causing the heap to oscillate as it tries to satisfy both.

**Beginner-Friendly Explanation:** `MaxGCPauseMillis` says "never let the cleaning crew stop the assembly line for more than 200 ms at a time." `GCTimeRatio` says "the cleaning crew should spend no more than 5% of the total time cleaning." The JVM adjusts the warehouse size to try to meet both goals, but they can conflict — a bigger warehouse means fewer cleanings (better ratio) but each cleaning may take longer (worse pause).

### Purposes

- To meet latency targets (pause-time goal)
- To meet throughput targets (GC time ratio goal)
- To guide the collector's heap-sizing decisions
- To quantify and communicate performance requirements to the JVM

### Syntax Rules and Structure

```java
-XX:MaxGCPauseMillis=<ms>    // Pause-time goal in milliseconds
-XX:GCTimeRatio=<ratio>      // Throughput goal: GC time / (GC + app time) = 1/(1+ratio)
```

**Component Breakdown:**

| Flag | Description |
|------|-------------|
| `-XX:MaxGCPauseMillis` | Soft pause-time goal (milliseconds). |
| `-XX:GCTimeRatio` | Throughput goal; `19` means 5% GC time. |

**Syntax Rules:**

- Both are **goals**, not guarantees; the collector may not always meet them.
- Default values vary by collector.
- Setting both creates competing pressures; choose values that represent an acceptable compromise.

**Constraints and Limitations:**

- "These adjustments may cause garbage collection to occur more frequently, reducing the overall throughput of the application".
- `MaxGCPauseMillis` has no effect on collectors that do not support pause-time goals (e.g., CMS historically).

### Annotated Code Example

```java
// File: GCTuningDemo.java
public class GCTuningDemo {
    public static void main(String[] args) {
        // Run with: java -XX:+UseG1GC -XX:MaxGCPauseMillis=200 -XX:GCTimeRatio=19 GCTuningDemo
        // Allocate objects to observe GC behaviour
        for (int i = 0; i < 10_000_000; i++) {
            byte[] b = new byte[1024]; // 1 KB
        }
        System.out.println("Allocation complete");
    }
}
```

**Expected Output (varies):**
```
Allocation complete
```

**Step-by-Step Explanation:**

1. `-XX:+UseG1GC` selects G1.
2. `-XX:MaxGCPauseMillis=200` tells G1 to aim for pauses ≤ 200 ms.
3. `-XX:GCTimeRatio=19` tells the collector to target no more than 5% of total time in GC.
4. G1 adjusts the heap size and young generation size to try to meet both goals simultaneously.

### Real-World Cases

- **Latency-sensitive APIs:** `MaxGCPauseMillis=50` or lower, often with ZGC or Shenandoah.
- **Throughput-oriented batch jobs:** `GCTimeRatio=9` (10% GC time) or higher, typically with Parallel GC.
- **Balanced services:** G1 with `MaxGCPauseMillis=200` and default `GCTimeRatio`.

---

## 6. JIT Flags (`-XX:+TieredCompilation` / `-XX:CompileThreshold`)

### Definitions

**Core Definition:** JIT flags control how the just-in-time compiler identifies and compiles hot methods and loops into native machine code.

**Technical Definition:** `-XX:+TieredCompilation` enables tiered compilation (default since Java 8), which uses the interpreter (Tier 0), C1 (Tiers 1–3), and C2 (Tier 4) to balance startup and peak performance. `-XX:CompileThreshold=<N>` sets the number of method invocations or loop back-edges before compilation is triggered; the non-tiered default is 10000. Lower thresholds compile sooner (faster warm-up, but less profiling data for optimization).

**Beginner-Friendly Explanation:** The JIT is like a coach who decides when to write down a better game plan. `TieredCompilation` says "use a quick plan first, then refine it." `CompileThreshold` says "wait until you've seen this play N times before writing the better plan." Lowering the threshold means the better plan arrives sooner, but it may be based on less observation.

### Purposes

- To balance application startup time and peak performance
- To control when compilation occurs and how much profiling data is gathered
- To reduce warm-up time for short-lived applications
- To ensure hot code is compiled to native code for long-running applications

### Syntax Rules and Structure

```java
-XX:+TieredCompilation          // Enable tiered compilation (default)
-XX:-TieredCompilation          // Disable tiered compilation
-XX:CompileThreshold=<N>        // Invocation threshold for compilation
```

**Component Breakdown:**

| Flag | Description |
|------|-------------|
| `-XX:+/-TieredCompilation` | Enables/disables tiered compilation. |
| `-XX:CompileThreshold=<N>` | Invocations before compilation (default ~10000 non-tiered). |

**Syntax Rules:**

- Tiered compilation uses tier-specific internal thresholds; `CompileThreshold` primarily affects the non-tiered path or the first tier.
- Lower thresholds reduce warm-up time but may sacrifice some optimization quality.

**Constraints and Limitations:**

- Disabling tiered compilation (`-XX:-TieredCompilation`) means methods wait longer for C2, increasing warm-up time.
- Setting `CompileThreshold` too low can cause compilation of cold code, wasting code cache and CPU.

### Annotated Code Example

```java
// File: JitFlagsDemo.java
public class JitFlagsDemo {
    static long compute(int n) {
        long sum = 0;
        for (int i = 0; i < n; i++) sum += i;
        return sum;
    }

    public static void main(String[] args) {
        // Run with: java -XX:CompileThreshold=5000 JitFlagsDemo
        long result = 0;
        for (int i = 0; i < 20_000; i++) {
            result = compute(1000);
        }
        System.out.println("Result: " + result);
    }
}
```

**Expected Output:**
```
Result: 499500
```

**Step-by-Step Explanation:**

1. `-XX:CompileThreshold=5000` lowers the invocation threshold so `compute` is compiled after 5,000 calls instead of the default 10,000.
2. The first 5,000 calls run in the interpreter/C1; subsequent calls run in JIT-compiled native code.
3. The total execution time reflects the earlier transition to compiled code.

### Real-World Cases

- **Short-lived serverless functions:** `-XX:TieredStopAtLevel=1` (C1-only) for minimal warm-up.
- **Long-running servers:** Default tiered compilation lets C2 produce peak-performance code after warm-up.
- **Benchmarking:** JMH controls warm-up and compilation to produce reliable measurements.

---

## 7. JVM Monitoring Tools

### Definitions

**Core Definition:** JVM monitoring tools observe the runtime behaviour of a Java process — memory usage, GC activity, thread states, and profiling data — to diagnose performance issues and guide tuning.

**Technical Definition:** The JDK provides command-line tools (`jps`, `jstat`, `jmap`, `jstack`, `jcmd`) and graphical tools (`jconsole`, `jvisualvm`, Java Mission Control). Java Flight Recorder (JFR) is a low-overhead event-recording framework, and JDK Mission Control (JMC) is its analysis GUI. `jcmd` is the recommended replacement for many older tools, offering enhanced diagnostics with reduced performance overhead.

**Beginner-Friendly Explanation:** These are the security cameras and dashboards. `jps` tells you which Java programs are running. `jstat` shows GC statistics over time. `jmap` takes a snapshot of the heap (for finding memory leaks). `jstack` takes a snapshot of thread states (for finding deadlocks). `jcmd` is the all-purpose diagnostic remote control. JFR records everything that happens in the JVM with very little overhead, and JMC is the playback and analysis tool for those recordings.

### Key Tools Table

| Tool | Purpose | Typical Command |
|------|---------|-----------------|
| `jps` | List Java processes and their PIDs | `jps -l` |
| `jstat` | Monitor GC and heap statistics over time | `jstat -gc <PID> 1000` |
| `jmap` | Heap histogram / heap dump | `jmap -histo:live <PID>` / `jmap -dump:file=heap.hprof <PID>` |
| `jstack` | Thread dump (deadlock detection, thread states) | `jstack <PID> > threads.txt` |
| `jcmd` | All-purpose diagnostic commands (recommended over jstack/jmap/jinfo) | `jcmd <PID> Thread.print` / `jcmd <PID> GC.heap_info` |
| `jconsole` | JMX-based GUI monitoring | `jconsole <PID>` |
| `jvisualvm` | GUI profiling and monitoring | `jvisualvm` |
| JFR | Low-overhead event recording | `-XX:StartFlightRecording=...` |
| JMC | GUI analysis of JFR recordings | Open `.jfr` in JMC |

### Annotated Code Example

```java
// File: MonitoringDemo.java
public class MonitoringDemo {
    public static void main(String[] args) throws Exception {
        // Run with: java MonitoringDemo
        // Then from another terminal:
        //   jps -l
        //   jstat -gc <PID> 1000
        //   jcmd <PID> GC.heap_info
        System.out.println("PID: " + ProcessHandle.current().pid());
        Thread.sleep(60_000);
    }
}
```

**Expected Output:**
```
PID: 12345
```

**Step-by-Step Explanation:**

1. The program prints its PID and sleeps for 60 seconds, giving you time to run monitoring tools.
2. `jps -l` lists the Java process and its PID.
3. `jstat -gc <PID> 1000` prints GC statistics every second.
4. `jcmd <PID> GC.heap_info` prints heap usage information.
5. JFR can be started with `jcmd <PID> JFR.start` and dumped with `jcmd <PID> JFR.dump filename=recording.jfr`.

### Real-World Cases

- **Memory leak investigation:** `jmap -histo:live` shows object counts; a steadily growing class count indicates a leak.
- **Deadlock detection:** `jstack <PID>` reveals threads blocked waiting for each other's monitors.
- **GC performance analysis:** `jstat -gcutil <PID> 1000` shows time spent in young/old collections.
- **Production profiling:** JFR records with minimal overhead, then JMC analyzes GC pauses, allocation hotspots, and thread states.

---

## 8. JVM Tuning Best Practices

### Definitions

**Core Definition:** JVM tuning best practices are guidelines for a disciplined, evidence-based approach to configuring the JVM, avoiding common pitfalls, and achieving sustainable performance.

**Technical Definition:** Best practices derive from Oracle's GC tuning guide and production experience. Key principles include: measure before tuning; fix application-level issues first; set `-Xms` = `-Xmx` for servers; use container-aware sizing; choose the right collector for the workload; enable GC logging and monitoring; and avoid over-tuning.

**Beginner-Friendly Explanation:** The golden rule is "don't tune blindly." Most JVM problems are actually application problems (memory leaks, excessive allocation, thread contention). Measure first, find the real bottleneck, apply one change, and verify. The most common mistake is copying someone else's flags without understanding your own workload.

### Key Best Practices Table

| Practice | Description |
|----------|-------------|
| **Measure first, then tune** | "先压测立基准，无量化不调优" — benchmark, then tune; no quantification, no tuning. |
| **Fix application code first** | "先优化业务代码，再调整JVM参数" — 90% of JVM issues are in the application. |
| **Set `-Xms` = `-Xmx` for servers** | Fix the heap to avoid resizing overhead and make memory predictable. |
| **Leave non-heap headroom** | "Maintain appropriate non-heap headroom (roughly 40% of total memory)". |
| **Use container-aware sizing** | Enable `-XX:+UseContainerSupport` and prefer `-XX:MaxRAMPercentage` in containerised environments. |
| **Choose the right collector** | G1 for balanced workloads; ZGC/Shenandoah for ultra-low latency; Parallel for batch throughput. |
| **Enable GC logging** | Unified logging (`-Xlog:gc*`) is essential for diagnosing GC behaviour. |
| **Monitor with JFR/JMC** | Low-overhead production profiling reveals GC, allocation, and thread issues. |
| **Do not over-tune** | "不要过度调优，不要盲目复制参数" — don't over-tune or copy parameters blindly. |
| **Set `MaxMetaspaceSize`** | Bound Metaspace to prevent native memory exhaustion. |
| **Validate changes against baseline** | Every change must have a quantified before/after comparison. |
| **Align JVM memory with container limits** | "Keep JVM Memory Aligned with Container Limits". |

### Annotated Complete Step-by-Step Code Examples

```java
// File: TuningBestPractices.java
public class TuningBestPractices {
    public static void main(String[] args) {
        // Example production-oriented flags:
        // java -Xms2g -Xmx2g \
        //      -XX:MetaspaceSize=256m -XX:MaxMetaspaceSize=512m \
        //      -XX:+UseG1GC -XX:MaxGCPauseMillis=200 \
        //      -Xlog:gc*:file=gc.log:time,uptime,level,tags \
        //      TuningBestPractices

        Runtime rt = Runtime.getRuntime();
        System.out.println("Max heap: " + rt.maxMemory() / (1024 * 1024) + " MB");
        System.out.println("Available processors: " + rt.availableProcessors());
    }
}
```

**Expected Output (varies):**
```
Max heap: 2048 MB
Available processors: 8
```

**Step-by-Step Explanation:**

1. `-Xms2g -Xmx2g` fixes the heap at 2 GB.
2. `-XX:MetaspaceSize=256m -XX:MaxMetaspaceSize=512m` bounds class metadata memory.
3. `-XX:+UseG1GC` selects G1; `-XX:MaxGCPauseMillis=200` sets a pause goal.
4. `-Xlog:gc*` enables detailed GC logging for later analysis.
5. The program confirms the heap size and processor count.

### Real-World Cases

- **Kubernetes microservices:** `-XX:MaxRAMPercentage=75.0` plus `-XX:+UseG1GC` and `-Xlog:gc*` for observability.
- **Large-heap caches/search:** ZGC with `-Xmx` sized to the workload and `-XX:MaxGCPauseMillis` largely unnecessary (ZGC's pauses are already tiny).
- **Batch data pipelines:** Parallel GC with `-Xms` = `-Xmx` and a high `GCTimeRatio` for maximum throughput.

---

## References

- HotSpot Virtual Machine Garbage Collection Tuning Guide — Oracle (Java SE 21) — https://docs.oracle.com/en/java/javase/21/gctuning/hotspot-virtual-machine-garbage-collection-tuning-guide.pdf
- GC Algorithms: G1, ZGC, and Shenandoah — foojay (2026) — https://wordpress.foojay.io/pedia/gc-algorithms-g1-zgc-and-shenandoah/
- Monitoring Tools — Oracle Java SE 8 Troubleshooting Guide — https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/tooldescr025.html
- Administrator Guide: Monitoring, Tuning, and Troubleshooting the JVM — Oracle Communications Converged Application Server — https://docs.oracle.com/en/industries/communications/converged-application-server/8.0/administrator/monitoring-tuning-and-troubleshooting-jvm.html
- JVM Tuning Mastery Guide — iwangjie/jvm-tuning-guide (GitHub) — https://github.com/iwangjie/jvm-tuning-guide
- Recommended JVM Heap Size Configuration Based on Memory Usage — CircleCI Support — https://support.circleci.com
- JIT Compilation Optimization — OneUptime Blog (2026) — https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-01-25-jit-compilation-optimization/README.md
- JVM Performance Tuning Parameters — Azul (2026) — https://www.azul.com
- JVM Performance Tuning — Azul (2026) — https://www.azul.com
- Million-Request Java: A Decision Framework for JVM Tuning in Ultra-Low Latency Microservices — Zenodo — https://zenodo.org
- Spring Boot 性能优化实战 — CSDN (2026) — https://blog.csdn.net
- JVM Memory Management and Tuning — Keycloak Documentation — https://www.keycloak.org
- Linux Java 性能监控 — Yisu (2025) — https://www.yisu.com
- Java 性能优化：量化调优实践 — Alibaba Cloud Developer (2026) — https://developer.aliyun.com