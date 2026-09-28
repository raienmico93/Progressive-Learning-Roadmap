# JIT Compilation: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Core Definition

Just-in-Time (JIT) compilation is a runtime compilation technique in which the Java Virtual Machine (JVM) dynamically translates frequently executed bytecode into optimized native machine code during program execution.

### Technical Definition

The JVM execution engine manages the execution stream through a combination of an interpreter, one or more just-in-time compilers, and the garbage collector. HotSpot uses a template-based interpreter (generated at VM startup) that maps each bytecode instruction to a predefined machine code sequence. The JIT compilation system includes two compilers: **C1** (client compiler), which performs quick compilation with some profiling, and **C2** (server compiler), which performs aggressive optimistic optimizations but takes longer to compile. Tiered compilation orchestrates these components, allowing methods to progress through five levels of execution: interpreter (level 0), C1 with profiling (levels 1–3), and C2 (level 4).

### Beginner-Friendly Explanation

Imagine you're reading a book in a foreign language. At first, you translate each sentence word-by-word (interpretation)—slow but immediate. As you notice you're reading the same phrases repeatedly, you memorize their translations (JIT compilation). The more often you see a phrase, the more effort you put into finding the best translation (C2 optimization). The JVM does exactly this: it starts by interpreting bytecode, then identifies frequently executed code ("hot spots") and compiles it to native machine code for much faster execution.

### Key Characteristics

- **Adaptive**: Compilation decisions are based on runtime profiling data.
- **Tiered**: Multiple compilation levels balance startup time and peak performance.
- **Speculative**: Optimizations rely on optimistic assumptions that can be deoptimized.
- **Background**: Compilation occurs on separate compiler threads, not the application thread.
- **Profiling-Guided**: Runtime type and branch profiles inform optimization decisions.
- **Platform-Specific**: Generates native machine code for the host CPU architecture.

### Prerequisites

- Basic Java programming knowledge (classes, methods, loops).
- Familiarity with compiling and running Java programs.
- Understanding of JVM architecture (runtime data areas, class loading).
- Basic knowledge of bytecode instructions.

### Related Programming Areas

- **Bytecode Execution**: Interpretation and execution engine mechanics.
- **Garbage Collection**: Interaction between JIT and GC for allocation optimizations.
- **Performance Engineering**: Profiling, benchmarking, and tuning JIT behavior.
- **Compiler Design**: Optimization passes and intermediate representations.
- **Native Compilation**: AOT compilation and GraalVM native images.

### Core Concepts Overview

1. **Interpretation**: Executing raw bytecode linearly via the interpreter loop.
2. **Just-in-Time Compilation**: Compiling hot bytecode into native machine code using tiered compilation (C1/C2).
3. **Hot Code Detection**: Identifying high-execution hotspots using invocation and backedge counters.
4. **Runtime Optimization**: Applying method inlining, escape analysis, loop unrolling, and dead code elimination.
5. **Alternative Compilation**: AOT compilation and GraalVM native binaries.

---

## Core Concept 1: Interpretation

### Definitions

**Core Definition**: Interpretation is the process by which the JVM executes bytecode instructions one at a time using an interpreter loop, without generating native machine code.

**Technical Definition**: The JVM's interpreter executes bytecode by looking at each instruction in turn, decoding it, and performing the actions demanded by it. HotSpot uses a **template-based interpreter** (also called a token-threaded dispatch interpreter) that maps each bytecode instruction to a predefined machine code sequence generated at VM startup. The template interpreter is faster than a C++ interpreter because bytecodes are executed using handcrafted machine code.

**Beginner-Friendly Explanation**: The interpreter is like a translator who reads a book sentence by sentence, translating each one on the fly. It's slow compared to having the whole book pre-translated, but it starts immediately—no waiting. The JVM uses the interpreter for every method initially, and only switches to JIT compilation when it detects that a method is being executed frequently.

### Purposes

- To enable immediate program startup without waiting for compilation.
- To execute code that is not hot enough to warrant JIT compilation.
- To gather profiling information that guides future compilation decisions.
- To serve as the fallback execution mode when optimized code is deoptimized.
- To provide a reference implementation for JIT-compiled code correctness.
- To execute cold code paths that would waste compilation resources.

### Syntax Rules and Structure

#### Complete General Syntax: Interpreter Execution Loop

```
INTERPRETER EXECUTION LOOP
│
├── Fetch bytecode instruction at PC register
├── Decode opcode
├── Dispatch to template for this opcode
│   └── Template contains pre-generated machine code
├── Execute the machine code template
│   ├── May manipulate operand stack
│   ├── May access local variables
│   └── May update PC register
└── Repeat until method returns
```

#### Component Breakdown

| Component | Role | Implementation |
|-----------|------|----------------|
| PC Register | Tracks current instruction | Per-thread |
| Opcode Decoder | Identifies instruction type | Dispatch table |
| Template Table | Maps opcodes to machine code | Generated at VM startup |
| Operand Stack | Holds intermediate values | Per-frame |
| Local Variable Array | Stores method parameters and locals | Per-frame |

#### Syntax Rules

- The interpreter is always enabled and executes all methods initially.
- Each bytecode instruction is fetched, decoded, and executed sequentially.
- The PC register advances to the next instruction after each execution.
- Template-based interpreters generate machine code templates at VM startup.
- The interpreter collects basic profiling data (invocation counts, branch profiles).
- The interpreter is used for level 0 of tiered compilation.

#### Constraints and Limitations

- Interpretation is significantly slower than native execution (10–100× slower).
- The interpreter cannot apply cross-method optimizations like inlining.
- Profiling data collected by the interpreter is less detailed than C1 profiling.
- The interpreter must handle all possible bytecode sequences, including rare paths.
- Interpreter execution does not benefit from CPU-specific optimizations.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Observing Interpreter Execution

**Setup Guide**: Save as `InterpreterDemo.java`, compile, and run with `-XX:+PrintCompilation -Xint` (interpreted mode only).

```java
// InterpreterDemo.java
public class InterpreterDemo {
    
    // A simple method — will be interpreted initially
    static int add(int a, int b) {
        return a + b;
    }
    
    // A loop method — will trigger OSR compilation when hot
    static long sumLoop(int n) {
        long sum = 0;
        for (int i = 1; i <= n; i++) {
            sum += i;
        }
        return sum;
    }
    
    public static void main(String[] args) {
        System.out.println("=== Interpreter Execution Demo ===");
        
        // Warm-up phase: interpreter executes these calls
        for (int i = 0; i < 1000; i++) {
            add(i, i + 1);
        }
        
        System.out.println("Warm-up complete (interpreted)");
        
        // This loop will trigger OSR compilation when hot
        long result = sumLoop(1_000_000);
        System.out.println("sumLoop(1,000,000) = " + result);
        
        System.out.println("\nInterpretation prioritizes:");
        System.out.println("- Immediate startup (no compilation wait)");
        System.out.println("- Lower memory usage (no compiled code)");
        System.out.println("- Profiling data collection for JIT");
    }
}
```

**Expected Output** (with `-Xint`):
```
=== Interpreter Execution Demo ===
Warm-up complete (interpreted)
sumLoop(1,000,000) = 500000500000

Interpretation prioritizes:
- Immediate startup (no compilation wait)
- Lower memory usage (no compiled code)
- Profiling data collection for JIT
```

**Why This Output**: With `-Xint`, the JVM runs entirely in interpreted mode (no JIT compilation). The program still produces correct results because the interpreter executes every bytecode instruction correctly, just more slowly. The `sumLoop` method computes the sum from 1 to 1,000,000, which is 500,000,500,000. Without `-Xint`, the JVM would JIT-compile `sumLoop` after it becomes hot, making it much faster.

---

#### Example 2: Comparing Interpreted vs. JIT-Compiled Performance

**Setup Guide**: Save as `InterpreterVsJIT.java`, compile, and run with different JVM flags.

```java
// InterpreterVsJIT.java
public class InterpreterVsJIT {
    
    static long computeSum(int n) {
        long sum = 0;
        for (int i = 1; i <= n; i++) {
            sum += i;
        }
        return sum;
    }
    
    public static void main(String[] args) {
        int n = 10_000_000;
        
        // Warm-up: let JIT compile computeSum
        for (int i = 0; i < 10; i++) {
            computeSum(1000);
        }
        
        // Measure execution time
        long start = System.nanoTime();
        long result = computeSum(n);
        long end = System.nanoTime();
        
        System.out.println("Result: " + result);
        System.out.printf("Time: %.2f ms%n", (end - start) / 1_000_000.0);
        System.out.println("Mode: " + 
            (System.getProperty("java.vm.info") != null ? 
             System.getProperty("java.vm.info") : "JIT"));
    }
}
```

**Run with interpretation only**:
```bash
java -Xint InterpreterVsJIT
```

**Run with JIT enabled** (default):
```bash
java InterpreterVsJIT
```

**Expected Output (Interpreted)** :
```
Result: 50000005000000
Time: 45.23 ms
Mode: interpreted mode
```

**Expected Output (JIT)** :
```
Result: 50000005000000
Time: 3.87 ms
Mode: mixed mode
```

**Why This Output**: With `-Xint`, the loop is interpreted instruction-by-instruction, taking ~45 ms. With JIT, the loop is compiled to native machine code with loop unrolling and other optimizations, taking ~4 ms—roughly 10× faster. The JIT-compiled version benefits from C2's aggressive optimizations, while the interpreted version executes each bytecode sequentially.

---

### Real-World Cases

- **Application Startup**: The interpreter enables immediate execution of application code while the JIT compiler warms up in the background.
- **Short-Lived Programs**: Command-line tools that run for less than a second benefit from interpretation because JIT compilation overhead would exceed the benefit.
- **Cold Code Paths**: Error handling and rarely executed branches are interpreted to avoid wasting compilation resources.
- **Profiling**: The interpreter collects the initial profiling data that guides C1 and C2 compilation decisions.
- **Deoptimization Fallback**: When JIT-compiled code is invalidated (e.g., due to class loading), execution falls back to the interpreter.

### References

- The Java HotSpot VM Under the Hood - https://cr.openjdk.org/~thartmann/talks/2017-Hotspot_Under_The_Hood.pdf
- x86 Interpreters in HotSpot - https://mail.openjdk.org/
- Overview of Java Technology-Based Software Execution - https://docs.oracle.com/

---

## Core Concept 2: Just-in-Time Compilation

### Definitions

**Core Definition**: Just-in-time compilation is the dynamic translation of frequently executed bytecode into optimized native machine code at runtime, using tiered compilation with C1 and C2 compilers.

**Technical Definition**: HotSpot includes two JIT compilers: **C1** (client compiler), which performs fast code generation with basic optimizations and a compilation threshold of ~1,500 invocations, and **C2** (server compiler), which performs highly optimized code generation with aggressive optimizations relying on profile data and a compilation threshold of ~10,000 invocations. **Tiered compilation** (default since Java 8) combines these compilers with the interpreter across five execution levels: level 0 (interpreter), levels 1–3 (C1 with varying profiling), and level 4 (C2).

**Beginner-Friendly Explanation**: JIT compilation is like having a personal tutor who watches you solve math problems. At first, you solve each problem step-by-step (interpretation). When the tutor notices you're solving the same type of problem repeatedly, they teach you a shortcut (C1 compilation). If you keep solving that problem type, the tutor teaches you an even faster method (C2 compilation). The JVM does this automatically: it starts by interpreting, then compiles hot methods to native code, using progressively more aggressive optimizations.

### Purposes

- To achieve near-native execution speed for hot code paths.
- To balance startup time and peak performance through tiered compilation.
- To exploit runtime profiling data for optimizations impossible at compile time.
- To enable speculative optimizations with deoptimization fallback.
- To adapt to the application's actual execution patterns dynamically.
- To support on-stack replacement (OSR) for long-running loops.

### Syntax Rules and Structure

#### Complete General Syntax: Tiered Compilation Levels

```
TIERED COMPILATION LEVELS
│
├── Level 0: INTERPRETER
│   ├── Executes bytecode instruction by instruction
│   ├── Collects basic profiling data
│   └── No compilation overhead
│
├── Level 1: C1 (No Profiling)
│   ├── Quick compilation, minimal optimizations
│   ├── No profiling information collected
│   └── Used for trivial methods
│
├── Level 2: C1 (Limited Profiling)
│   ├── C1 compiled with invocation and backedge counters
│   ├── Some profiling (call counts)
│   └── Used when C2 queue is long
│
├── Level 3: C1 (Full Profiling)
│   ├── C1 compiled with full profiling (MDO)
│   ├── Collects type profiles, branch profiles
│   └── Prepares method for C2 compilation
│
└── Level 4: C2 (Optimized)
    ├── Fully optimized native code
    ├── Aggressive inlining, escape analysis, loop unrolling
    └── Based on profiling data from level 3
```

#### Component Breakdown

| Level | Compiler | Profiling | Use Case |
|-------|----------|-----------|----------|
| 0 | Interpreter | Basic | All methods initially |
| 1 | C1 | None | Trivial methods, fast startup |
| 2 | C1 | Limited | When C2 queue is long |
| 3 | C1 | Full | Preparing for C2 |
| 4 | C2 | None (uses L3 data) | Hot methods, peak performance |

#### Syntax Rules

- Tiered compilation is enabled by default since Java 8 (`-XX:+TieredCompilation`).
- Methods start at level 0 (interpreter) and progress to higher levels based on invocation counts.
- C1 compiles at ~1,500 invocations; C2 compiles at ~10,000 invocations.
- Level 3 collects full profiling data (MDO) used by C2 for optimization.
- On-stack replacement (OSR) allows a running loop to switch to compiled code.
- Deoptimization returns execution to the interpreter if optimistic assumptions fail.

#### Constraints and Limitations

- Tiered compilation increases code size by 2–4× compared to C2-only compilation.
- C2 compilation is slow and consumes significant CPU resources.
- The code cache has a fixed size (`-XX:ReservedCodeCacheSize`); if full, compilation stops.
- Compilation occurs on background threads and may not be available immediately.
- Aggressive optimizations may be deoptimized if class hierarchy assumptions change.
- The compilation policy is complex and implementation-dependent.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Observing Tiered Compilation with `-XX:+PrintCompilation`

**Setup Guide**: Save as `TieredCompilationDemo.java`, compile, and run with `java -XX:+PrintCompilation TieredCompilationDemo`.

```java
// TieredCompilationDemo.java
public class TieredCompilationDemo {
    
    // Method that will be JIT-compiled after warm-up
    static int hotMethod(int x) {
        return x * 2 + 1;
    }
    
    // Method with a loop — triggers OSR compilation
    static long loopMethod(int n) {
        long sum = 0;
        for (int i = 0; i < n; i++) {
            sum += hotMethod(i);
        }
        return sum;
    }
    
    public static void main(String[] args) {
        System.out.println("=== Tiered Compilation Demo ===");
        
        // Warm-up: interpreter executes these calls
        long warmup = 0;
        for (int i = 0; i < 20_000; i++) {
            warmup += hotMethod(i);
        }
        System.out.println("Warm-up complete: " + warmup);
        
        // This loop triggers OSR compilation
        long result = loopMethod(1_000_000);
        System.out.println("loopMethod result: " + result);
        
        System.out.println("\nCheck -XX:+PrintCompilation output for:");
        System.out.println("- Compilation events (method name, tier level)");
        System.out.println("- OSR compilations (% marker)");
        System.out.println("- Deoptimizations (made not entrant)");
    }
}
```

**Expected Output** (with `-XX:+PrintCompilation`):
```
    100    1       3       java.lang.Object::<init> (1 bytes)
    105    2       3       TieredCompilationDemo::hotMethod (7 bytes)
    110    3       3       java.lang.String::hashCode (55 bytes)
    ...
    250    4       3       TieredCompilationDemo::hotMethod (7 bytes)
    300    5 %     4       TieredCompilationDemo::loopMethod @ 2 (25 bytes)
    ...
=== Tiered Compilation Demo ===
Warm-up complete: 400019999
loopMethod result: 500000500000

Check -XX:+PrintCompilation output for:
- Compilation events (method name, tier level)
- OSR compilations (% marker)
- Deoptimizations (made not entrant)
```

**Why This Output**: The `-XX:+PrintCompilation` flag prints a line for each compilation event. Columns include: timestamp, compile ID, tier level (3 = C1 with full profiling, 4 = C2), method name, and bytecode size. The `%` marker indicates an OSR (On-Stack Replacement) compilation, which occurs when a loop is compiled while it's running. `hotMethod` is compiled to level 3 (C1 profiling), then potentially to level 4 (C2) after enough profiling data is collected.

---

#### Example 2: Configuring Compilation Thresholds

**Setup Guide**: Save as `CompilationThresholds.java`, compile, and run with different threshold flags.

```java
// CompilationThresholds.java
public class CompilationThresholds {
    
    static int compute(int x) {
        return x * x;
    }
    
    public static void main(String[] args) {
        System.out.println("=== Compilation Thresholds Demo ===");
        System.out.println("JVM: " + System.getProperty("java.vm.name"));
        
        // Print relevant compilation flags
        System.out.println("\nCompilation flags:");
        System.out.println("  Tier3InvocationThreshold: " + 
            System.getProperty("tier3.invoke.threshold", "default"));
        System.out.println("  Tier4InvocationThreshold: " + 
            System.getProperty("tier4.invoke.threshold", "default"));
        
        // Call the method many times to trigger compilation
        long sum = 0;
        for (int i = 0; i < 50_000; i++) {
            sum += compute(i);
        }
        
        System.out.println("\nAfter 50,000 invocations:");
        System.out.println("  compute() should be compiled");
        System.out.println("  sum = " + sum);
        
        System.out.println("\nTo see compilation events, run with:");
        System.out.println("  -XX:+PrintCompilation");
        System.out.println("To set custom thresholds:");
        System.out.println("  -XX:Tier3InvocationThreshold=500");
        System.out.println("  -XX:Tier4InvocationThreshold=5000");
    }
}
```

**Expected Output**:
```
=== Compilation Thresholds Demo ===
JVM: OpenJDK 64-Bit Server VM

Compilation flags:
  Tier3InvocationThreshold: default
  Tier4InvocationThreshold: default

After 50,000 invocations:
  compute() should be compiled
  sum = 41654166650000

To see compilation events, run with:
  -XX:+PrintCompilation
To set custom thresholds:
  -XX:Tier3InvocationThreshold=500
  -XX:Tier4InvocationThreshold=5000
```

**Why This Output**: After 50,000 invocations, `compute()` has far exceeded both the C1 (~1,500) and C2 (~10,000) thresholds, so it should be compiled to native code. The exact thresholds can be inspected with `-XX:+PrintFlagsFinal` and customized with `-XX:Tier3InvocationThreshold` and `-XX:Tier4InvocationThreshold`. Lowering these thresholds triggers compilation earlier, which helps short-lived applications but may waste resources on methods that are not actually hot.

---

### Real-World Cases

- **High-Frequency Trading**: JIT compilation with C2 optimization enables sub-millisecond latency by inlining virtual calls and optimizing tight loops.
- **Big Data Processing (Spark, Hadoop)**: JIT compilation is critical for processing large datasets; Spark's Tungsten engine generates specialized bytecode.
- **Web Servers (Tomcat, Netty)**: JIT compilation optimizes request-handling paths that execute millions of times per second.
- **Serverless Functions**: Setting `-XX:TieredStopAtLevel=1` improves cold-start performance by using only C1 compilation.
- **Long-Running Services**: C2 compilation achieves peak throughput for methods that execute millions of times.

### References

- Tiered Compilation in HotSpot - https://cr.openjdk.org/~thartmann/talks/2016-Hotspot_Under_The_Hood.pdf
- Compilation Levels - https://cr.openjdk.org/~iveresov/tiered/Tiered.pdf
- Java HotSpot Virtual Machine Performance Enhancements - https://docs.oracle.com/en/java/javase/17/performance/

---

## Core Concept 3: Hot Code Detection

### Definitions

**Core Definition**: Hot code detection is the mechanism by which the JVM identifies frequently executed methods and loops (hot spots) using runtime invocation counters and loop backedge counters to trigger JIT compilation.

**Technical Definition**: Each method has two counters in its method data structure: an **invocation counter** (counting method entries) and a **backedge counter** (counting backward branches taken within the method, i.e., loop iterations). When these counters reach certain frequency values (e.g., `Tier3InvocationThreshold`, `Tier3BackEdgeThreshold`), a compilation policy is called to decide whether to compile the method. Backward branches typically denote loops in the code. The compilation policy scales thresholds based on the current load of C1 and C2 compiler threads.

**Beginner-Friendly Explanation**: The JVM is like a teacher who watches students solve problems. Every time a student (method) is called, the teacher makes a tally mark (invocation counter). Every time a student repeats a loop, the teacher makes another tally mark (backedge counter). When the tally marks reach a certain number, the teacher decides the student needs a faster method—so they call in a specialist (JIT compiler) to create an optimized solution. The more tally marks, the more effort the teacher puts into getting the best solution.

### Purposes

- To identify methods and loops that would benefit most from JIT compilation.
- To avoid wasting compilation resources on cold code paths.
- To trigger on-stack replacement (OSR) for long-running loops.
- To balance compilation effort against execution benefit.
- To adapt compilation decisions to the application's actual execution profile.
- To prevent the JIT compiler from being overwhelmed by too many compilation requests.

### Syntax Rules and Structure

#### Complete General Syntax: Hot Code Detection Mechanism

```
HOT CODE DETECTION
│
├── Method Invocation Counter
│   ├── Incremented on each method entry
│   ├── Used for regular compilation decisions
│   └── Threshold: TierXInvocationThreshold
│
├── Backedge Counter
│   ├── Incremented on each backward branch (loop iteration)
│   ├── Used for OSR compilation decisions
│   └── Threshold: TierXBackEdgeThreshold
│
├── Frequency Notifications
│   ├── Counters reach notify frequency (TierXInvokeNotifyFreqLog)
│   ├── Compilation policy is called
│   └── Decision made based on counters and compiler queue load
│
└── Compilation Decision
    ├── Continue interpretation
    ├── Start profiling (level 3)
    ├── Compile with C1 (level 1 or 2)
    └── Compile with C2 (level 4)
```

#### Component Breakdown

| Counter | Incremented When | Threshold Flag | Purpose |
|---------|-----------------|----------------|---------|
| Invocation | Method entry | `Tier3InvocationThreshold` | Trigger C1 compilation |
| Backedge | Backward branch | `Tier3BackEdgeThreshold` | Trigger OSR compilation |
| Invocation (C2) | Method entry | `Tier4InvocationThreshold` | Trigger C2 compilation |
| Backedge (C2) | Backward branch | `Tier4BackEdgeThreshold` | Trigger C2 OSR compilation |

#### Syntax Rules

- Every method has both an invocation counter and a backedge counter.
- Counters are incremented in the interpreter and in C1-compiled code (with profiling).
- The compilation policy is called when counters reach a frequency notification threshold.
- The policy considers counter values, compiler queue lengths, and method characteristics.
- Thresholds are scaled dynamically based on compiler thread availability.
- Backedge counters specifically trigger OSR compilation for loops.
- Counters can be decayed over time to identify "stale" hot methods.

#### Constraints and Limitations

- Counter values are implementation-dependent and version-specific.
- Default thresholds differ between tiered and non-tiered compilation.
- Counters can overflow; the JVM handles this by capping values.
- The compilation policy is complex and not documented in the JVM specification.
- Thresholds can be tuned with `-XX:` flags but doing so may hurt performance.
- Counter-based detection may miss methods that are hot in bursts but not continuously.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Demonstrating Hot Code Detection

**Setup Guide**: Save as `HotCodeDetection.java`, compile, and run with `-XX:+PrintCompilation`.

```java
// HotCodeDetection.java
public class HotCodeDetection {
    
    // A method that will be called many times — becomes "hot"
    static int hotMethod(int x) {
        return x * 2;
    }
    
    // A method with a loop — backedge counter triggers OSR
    static long loopMethod(int n) {
        long sum = 0;
        for (int i = 0; i < n; i++) {
            sum += i;
        }
        return sum;
    }
    
    // A cold method — called only once
    static void coldMethod() {
        System.out.println("Cold method executed (not hot)");
    }
    
    public static void main(String[] args) {
        System.out.println("=== Hot Code Detection Demo ===");
        
        // Cold method — will not be compiled
        coldMethod();
        
        // Hot method — called many times, will be compiled
        long sum = 0;
        for (int i = 0; i < 50_000; i++) {
            sum += hotMethod(i);
        }
        System.out.println("Hot method sum: " + sum);
        
        // Loop method — backedge counter triggers OSR
        long loopSum = loopMethod(1_000_000);
        System.out.println("Loop method sum: " + loopSum);
        
        System.out.println("\nDetection triggers:");
        System.out.println("- hotMethod: invocation counter (50,000 calls)");
        System.out.println("- loopMethod: backedge counter (1,000,000 iterations)");
        System.out.println("- coldMethod: no compilation (called once)");
    }
}
```

**Expected Output** (with `-XX:+PrintCompilation`):
```
    100    1       3       java.lang.Object::<init> (1 bytes)
    105    2       3       HotCodeDetection::hotMethod (5 bytes)
    110    3       3       java.lang.String::hashCode (55 bytes)
    250    4 %     4       HotCodeDetection::loopMethod @ 2 (20 bytes)
Cold method executed (not hot)
Hot method sum: 2499950000
Loop method sum: 499999500000

Detection triggers:
- hotMethod: invocation counter (50,000 calls)
- loopMethod: backedge counter (1,000,000 iterations)
- coldMethod: no compilation (called once)
```

**Why This Output**: `hotMethod` is compiled to level 3 (C1 with profiling) after its invocation counter reaches the threshold. `loopMethod` triggers an OSR compilation (`%` marker) at level 4 (C2) because its backedge counter exceeds the threshold while the loop is still running. `coldMethod` is never compiled because it is called only once and never becomes hot. This demonstrates how the JVM selectively compiles only frequently executed code.

---

#### Example 2: Tuning Hot Code Detection Thresholds

**Setup Guide**: Save as `ThresholdTuning.java`, compile, and run with different threshold settings.

```java
// ThresholdTuning.java
public class ThresholdTuning {
    
    static long compute(long x) {
        return x * x + 1;
    }
    
    public static void main(String[] args) {
        System.out.println("=== Hot Code Detection Threshold Tuning ===");
        
        // Print current threshold values
        System.out.println("\nCurrent thresholds (from JVM flags):");
        System.out.println("  Run with -XX:+PrintFlagsFinal to see all");
        
        // Call method many times
        long sum = 0;
        for (int i = 0; i < 100_000; i++) {
            sum += compute(i);
        }
        System.out.println("\nSum after 100,000 calls: " + sum);
        
        System.out.println("\nThreshold tuning options:");
        System.out.println("  -XX:Tier3InvocationThreshold=500   (default ~1500)");
        System.out.println("  -XX:Tier4InvocationThreshold=5000  (default ~10000)");
        System.out.println("  -XX:Tier3BackEdgeThreshold=30000   (default ~60000)");
        System.out.println("  -XX:Tier4BackEdgeThreshold=20000   (default ~40000)");
        System.out.println("\nLower thresholds = earlier compilation");
        System.out.println("Higher thresholds = less compilation overhead");
    }
}
```

**Expected Output**:
```
=== Hot Code Detection Threshold Tuning ===

Current thresholds (from JVM flags):
  Run with -XX:+PrintFlagsFinal to see all

Sum after 100,000 calls: 333338333350000

Threshold tuning options:
  -XX:Tier3InvocationThreshold=500   (default ~1500)
  -XX:Tier4InvocationThreshold=5000  (default ~10000)
  -XX:Tier3BackEdgeThreshold=30000   (default ~60000)
  -XX:Tier4BackEdgeThreshold=20000   (default ~40000)

Lower thresholds = earlier compilation
Higher thresholds = less compilation overhead
```

**Why This Output**: After 100,000 calls, `compute()` has far exceeded the default thresholds and is compiled to C2. The threshold values can be inspected and tuned. Lowering thresholds triggers compilation earlier (beneficial for short-lived applications), while raising them reduces compilation overhead (beneficial for applications with many short-lived methods).

---

### Real-World Cases

- **Microservices**: Lowering `Tier3InvocationThreshold` helps microservices reach peak performance faster, reducing latency in request-handling paths.
- **Batch Processing**: Methods that process large datasets become hot and are compiled to C2, achieving peak throughput.
- **Interactive Applications**: The backedge counter triggers OSR compilation for long-running UI event loops.
- **Warm-up Optimization**: Frameworks like Spring Boot may warm up hot methods during startup to trigger compilation before serving requests.
- **Performance Testing**: Understanding hot code detection helps interpret benchmark results (e.g., JMH warm-up iterations).

### References

- How Does the JVM Decide to JIT-Compile a Method? - https://browse.library.kiwix.org/content/stackoverflow.com_en_all_2023-11/
- Compilation Levels - https://cr.openjdk.org/~iveresov/tiered/Tiered.pdf
- HotSpot Compilation Policy - https://cr.openjdk.org/

---

## Core Concept 4: Runtime Optimization

### Definitions

**Core Definition**: Runtime optimization is the application of advanced program transformations—including method inlining, escape analysis, loop unrolling, and dead code elimination—by the JIT compiler to improve execution performance.

**Technical Definition**: HotSpot C2 applies numerous optimizations based on runtime profiling data. **Method inlining** replaces a method call with the body of the called method, enabling further optimizations. **Escape analysis** determines whether an object's scope is confined to the allocating method or thread; if so, the object can be allocated on the stack (scalar replacement) or its synchronization removed. **Loop unrolling** replicates the loop body to reduce loop overhead. **Dead code elimination** removes code that has no effect on program output. These optimizations are enabled by default (`-XX:+DoEscapeAnalysis`, inlining enabled) and controlled by various flags.

**Beginner-Friendly Explanation**: Runtime optimization is like a chef who watches you cook and then suggests shortcuts. If you always chop onions the same way, the chef teaches you a faster technique (method inlining). If you always use a small bowl for mixing but never take it out of the kitchen, the chef suggests using a measuring cup directly (escape analysis—no need for the bowl). If you always stir the pot 10 times, the chef shows you how to stir once with a bigger spoon (loop unrolling). And if you're doing a step that doesn't change the final dish, the chef tells you to skip it (dead code elimination).

### Purposes

- To reduce method call overhead by inlining hot methods.
- To eliminate unnecessary object allocations through escape analysis.
- To reduce loop overhead by unrolling loop bodies.
- To remove code that does not affect program output.
- To enable further optimizations by exposing code structure.
- To improve cache locality and reduce memory traffic.

### Syntax Rules and Structure

#### Complete General Syntax: Key Runtime Optimizations

```
RUNTIME OPTIMIZATIONS
│
├── Method Inlining
│   ├── Replaces call site with method body
│   ├── Enabled by: -XX:+Inline (default)
│   ├── Size limits: MaxInlineSize (35), FreqInlineSize (325)
│   └── Enables: constant folding, dead code elimination, further inlining
│
├── Escape Analysis
│   ├── Determines if object escapes method/thread
│   ├── Enabled by: -XX:+DoEscapeAnalysis (default)
│   ├── Stack allocation: object allocated on stack instead of heap
│   └── Scalar replacement: object fields replaced by local variables
│
├── Loop Unrolling
│   ├── Replicates loop body multiple times
│   ├── Reduces loop control overhead
│   ├── Maximum unrolls: 16 (LoopUnrollLimit)
│   └── Small loops (≤3 iterations) fully unrolled
│
└── Dead Code Elimination
    ├── Removes code with no effect on output
    ├── Enabled by inlining and constant propagation
    └── Removes unreachable branches and unused computations
```

#### Component Breakdown

| Optimization | Trigger | Effect | Flag |
|-------------|---------|--------|------|
| Method Inlining | Hot call site | Replaces call with method body | `-XX:+Inline` |
| Escape Analysis | Object creation | Stack allocation or scalar replacement | `-XX:+DoEscapeAnalysis` |
| Loop Unrolling | Counted loop | Replicates body, reduces overhead | `-XX:LoopUnrollLimit` |
| Dead Code Elimination | Unreachable code | Removes code | Enabled by inlining |

#### Syntax Rules

- Method inlining is the most important optimization; it enables all others.
- Inlining decisions consider method size, call frequency, and profiling data.
- Escape analysis works on a per-object basis within a compilation unit.
- Scalar replacement replaces object fields with local variables, eliminating allocation.
- Loop unrolling only applies to counted loops with known iteration counts.
- Dead code elimination removes code that is unreachable or has no effect.
- These optimizations are speculative and can be deoptimized if assumptions fail.

#### Constraints and Limitations

- Inlining large methods can increase code size and compilation time.
- Escape analysis is limited to objects that do not escape the compilation unit.
- Scalar replacement works only when all object field accesses are known at compile time.
- Loop unrolling increases code size; the JVM limits unrolling to avoid code bloat.
- Dead code elimination cannot remove code with side effects (e.g., I/O, exceptions).
- Aggressive optimizations may cause deoptimization if profiling data becomes invalid.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Observing Method Inlining

**Setup Guide**: Save as `InliningDemo.java`, compile, and run with `-XX:+PrintInlining -XX:+UnlockDiagnosticVMOptions`.

```java
// InliningDemo.java
public class InliningDemo {
    
    // Small method — prime candidate for inlining
    static int add(int a, int b) {
        return a + b;
    }
    
    // Larger method — may or may not be inlined
    static int compute(int x) {
        int result = add(x, 1);      // add() will be inlined here
        result = result * 2;
        result = add(result, 3);     // add() will be inlined here too
        return result;
    }
    
    public static void main(String[] args) {
        System.out.println("=== Method Inlining Demo ===");
        
        // Warm up — triggers compilation and inlining
        long sum = 0;
        for (int i = 0; i < 50_000; i++) {
            sum += compute(i);
        }
        
        System.out.println("Sum: " + sum);
        System.out.println("\nCheck -XX:+PrintInlining output for:");
        System.out.println("- add() inlined into compute()");
        System.out.println("- compute() inlined into main()");
        System.out.println("- Reason if not inlined (too large, etc.)");
    }
}
```

**Expected Output** (with `-XX:+PrintInlining`):
```
=== Method Inlining Demo ===
Sum: 2500149999

Check -XX:+PrintInlining output for:
- add() inlined into compute()
- compute() inlined into main()
- Reason if not inlined (too large, etc.)
```

**Inlining Output** (from `-XX:+PrintInlining`):
```
@ 4   InliningDemo::add (4 bytes)   inline (hot)
@ 12  InliningDemo::add (4 bytes)   inline (hot)
@ 3   InliningDemo::compute (16 bytes)   inline (hot)
```

**Why This Output**: The `add` method is small (4 bytes of bytecode) and called from hot code, so it is inlined into `compute` at both call sites. The `compute` method is also inlined into `main` because it is hot and not too large. Inlining enables further optimizations: the additions become part of a larger expression, allowing constant folding and dead code elimination. Without inlining, each call would require a method invocation with stack frame setup and teardown.

---

#### Example 2: Demonstrating Escape Analysis

**Setup Guide**: Save as `EscapeAnalysisDemo.java`, compile, and run with `-XX:+DoEscapeAnalysis` (default).

```java
// EscapeAnalysisDemo.java
public class EscapeAnalysisDemo {
    
    // A simple point class
    static class Point {
        int x, y;
        Point(int x, int y) { this.x = x; this.y = y; }
        int sum() { return x + y; }
    }
    
    // Method where Point does NOT escape — escape analysis can eliminate allocation
    static int nonEscaping() {
        Point p = new Point(10, 20);  // Object does not escape this method
        return p.sum();                // Scalar replacement can eliminate allocation
    }
    
    // Method where Point DOES escape — allocation must occur on heap
    static Point escaping() {
        Point p = new Point(10, 20);
        return p;  // Object escapes the method
    }
    
    public static void main(String[] args) {
        System.out.println("=== Escape Analysis Demo ===");
        
        // Warm up
        long sum1 = 0, sum2 = 0;
        for (int i = 0; i < 100_000; i++) {
            sum1 += nonEscaping();
            sum2 += escaping().sum();
        }
        
        System.out.println("Non-escaping sum: " + sum1);
        System.out.println("Escaping sum: " + sum2);
        
        System.out.println("\nEscape Analysis Results:");
        System.out.println("- nonEscaping(): Point allocation eliminated (scalar replaced)");
        System.out.println("- escaping(): Point allocation remains (object escapes)");
        System.out.println("\nEscape analysis is enabled by default (-XX:+DoEscapeAnalysis).");
    }
}
```

**Expected Output**:
```
=== Escape Analysis Demo ===
Non-escaping sum: 3000000
Escaping sum: 3000000

Escape Analysis Results:
- nonEscaping(): Point allocation eliminated (scalar replaced)
- escaping(): Point allocation remains (object escapes)

Escape analysis is enabled by default (-XX:+DoEscapeAnalysis).
```

**Why This Output**: In `nonEscaping()`, the `Point` object is created, used, and discarded within the method—it never escapes. Escape analysis detects this and replaces the object allocation with scalar variables (the `x` and `y` fields become local variables), eliminating heap allocation entirely. In `escaping()`, the `Point` object is returned, so it escapes the method and must be allocated on the heap. Both methods produce the same result (30 per call), but the non-escaping version is faster due to reduced allocation and garbage collection pressure.

---

### Real-World Cases

- **High-Performance Computing**: Method inlining and escape analysis are critical for numerical computation libraries (e.g., matrix multiplication).
- **Object Allocation Elimination**: Escape analysis eliminates temporary object allocations in tight loops, reducing GC pressure.
- **String Processing**: String concatenation (via `StringBuilder`) is optimized through inlining and escape analysis.
- **Game Engines**: JIT optimizations improve frame rates by optimizing hot rendering paths.
- **Database Drivers**: Inlining and escape analysis reduce overhead in result set processing.

### References

- Inside.java Episode 64: JIT Compiler From the Ground Up - https://inside.java/
- Escape Analysis in HotSpot - https://shipilev.net/jvm/anatomy-quarks/18-scalar-replacement/
- Loop Optimizations in C2 - https://wiki.openjdk.org/display/HotSpot/Loop+Optimizations

---

## Core Concept 5: Alternative Compilation

### Definitions

**Core Definition**: Alternative compilation refers to approaches that deviate from traditional JIT compilation, including Ahead-of-Time (AOT) compilation and GraalVM's native image generation, producing standalone native executables.

**Technical Definition**: **Ahead-of-Time (AOT) compilation** compiles Java bytecode to native machine code before program execution, eliminating the need for a JVM at runtime. **GraalVM Native Image** allows compiling applications ahead-of-time to executable native binaries that are standalone, start instantly, and have lower memory usage. The main trade-off is that analysis and compilation happen under the closed-world assumption, meaning the static analysis needs to process all bytecode which will ever be executed in the application, making dynamic features like reflection and dynamic class loading tricky.

**Beginner-Friendly Explanation**: Traditional JIT compilation is like having a live translator who listens to you and translates in real time—flexible but requires the translator to be present. AOT compilation is like translating your entire speech into a written document before you deliver it—no translator needed at delivery time, but you can't change the speech on the fly. GraalVM Native Image takes this further, packaging your entire Java application into a single executable file that starts in milliseconds and uses less memory, ideal for cloud and serverless environments.

### Purposes

- To eliminate JVM startup overhead and achieve instant startup.
- To reduce memory footprint for cloud and containerized deployments.
- To produce standalone executables that don't require a JVM installation.
- To enable deployment in environments where JIT compilation is not allowed.
- To reduce cold-start latency in serverless and microservice architectures.
- To support polyglot applications through GraalVM's multi-language runtime.

### Syntax Rules and Structure

#### Complete General Syntax: GraalVM Native Image Build

```
NATIVE IMAGE BUILD PROCESS
│
├── Prerequisites
│   ├── GraalVM JDK installed
│   ├── native-image tool available
│   └── Application compiled to .class files
│
├── Build Command
│   ├── native-image -jar myapp.jar
│   ├── native-image -cp myapp.jar com.example.Main
│   └── native-image --language:java -H:Name=myapp Main
│
├── AOT Analysis
│   ├── Static analysis of all reachable code
│   ├── Reflection configuration (if needed)
│   └── Resource inclusion (if needed)
│
└── Output
    ├── Standalone native executable
    ├── No JVM required at runtime
    └── Platform-specific binary
```

#### Component Breakdown

| Aspect | JIT (HotSpot) | AOT (GraalVM Native Image) |
|--------|---------------|----------------------------|
| Compilation Time | Runtime (warm-up) | Build time |
| Startup | 100s of ms | 10s of ms |
| Memory | JVM + heap + metaspace | Application only |
| Peak Performance | Excellent (C2) | Good (limited optimizations) |
| Reflection | Full support | Requires configuration |
| Dynamic Loading | Full support | Closed-world assumption |
| Platform | JVM-dependent | Platform-specific binary |

#### Syntax Rules

- GraalVM Native Image requires the `native-image` tool, installed via `gu install native-image`.
- The build command is `native-image [options] class` or `native-image -jar app.jar`.
- Reflection, JNI, and dynamic proxies require configuration files (`reflect-config.json`).
- Resources must be explicitly included with `-H:IncludeResources`.
- The output is a platform-specific executable (e.g., `myapp` on Linux, `myapp.exe` on Windows).
- Spring Boot 3+ provides built-in AOT support for native image generation.

#### Constraints and Limitations

- Native images use a closed-world assumption: all classes must be known at build time.
- Reflection, dynamic class loading, and JNI require explicit configuration.
- Peak throughput may be lower than JIT-compiled HotSpot (no C2 optimizations).
- Build times are longer than JAR packaging.
- Debugging native images is more difficult than debugging JVM applications.
- Not all Java libraries are compatible with native image (e.g., those relying on dynamic features).

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Building a GraalVM Native Image

**Setup Guide**: This example requires GraalVM with `native-image` installed. Save as `NativeImageDemo.java`, compile, and build with `native-image`.

```java
// NativeImageDemo.java
public class NativeImageDemo {
    
    public static void main(String[] args) {
        System.out.println("=== GraalVM Native Image Demo ===");
        System.out.println("This program was compiled ahead-of-time.");
        System.out.println("It started instantly without a JVM.");
        
        // Simple computation
        long sum = 0;
        for (int i = 1; i <= 1_000_000; i++) {
            sum += i;
        }
        System.out.println("Sum: " + sum);
        
        // Show memory usage (if available)
        Runtime rt = Runtime.getRuntime();
        System.out.println("Max memory (MB): " + rt.maxMemory() / (1024 * 1024));
        
        System.out.println("\nAdvantages:");
        System.out.println("- Instant startup (no JVM warm-up)");
        System.out.println("- Lower memory footprint");
        System.out.println("- Standalone executable");
    }
}
```

**Build Commands**:
```bash
# Compile Java source
javac NativeImageDemo.java

# Build native image
native-image NativeImageDemo

# Run the native executable
./nativeimagedemo
```

**Expected Output**:
```
=== GraalVM Native Image Demo ===
This program was compiled ahead-of-time.
It started instantly without a JVM.
Sum: 500000500000
Max memory (MB): 64

Advantages:
- Instant startup (no JVM warm-up)
- Lower memory footprint
- Standalone executable
```

**Why This Output**: The `native-image` tool performs AOT compilation, analyzing all reachable code and generating a standalone executable. The program starts instantly because there is no JVM warm-up or JIT compilation. Memory usage is lower because there is no JVM overhead (no metaspace, no code cache, no interpreter). The trade-off is that the binary is platform-specific and dynamic features require configuration.

---

#### Example 2: Mixing AOT and JIT with GraalVM

**Setup Guide**: This example demonstrates GraalVM's ability to mix AOT and JIT compilation. Save as `MixedModeDemo.java`, compile, and build.

```java
// MixedModeDemo.java
import java.lang.reflect.Method;

public class MixedModeDemo {
    
    // Method that will be AOT-compiled
    static int staticCompute(int x) {
        return x * 2;
    }
    
    public static void main(String[] args) throws Exception {
        System.out.println("=== GraalVM Mixed AOT + JIT Demo ===");
        
        // AOT-compiled path: static computation
        int result = staticCompute(21);
        System.out.println("Static compute: " + result);
        
        // Dynamic path: reflection (requires configuration for native image)
        // In a native image, this would use the included JIT compiler
        System.out.println("\nDynamic features available with GraalVM:");
        System.out.println("- Reflection (with configuration)");
        System.out.println("- Dynamic class loading (Truffle)");
        System.out.println("- Polyglot execution (JS, Python, Ruby)");
        
        // Show that the program can still use dynamic features
        // (in GraalVM, Java on Truffle provides a JVM within the native image)
        System.out.println("\nThe native executable includes:");
        System.out.println("- AOT-compiled static code");
        System.out.println("- Optional JIT compiler for dynamic code");
        System.out.println("- Truffle framework for polyglot execution");
    }
}
```

**Expected Output**:
```
=== GraalVM Mixed AOT + JIT Demo ===
Static compute: 42

Dynamic features available with GraalVM:
- Reflection (with configuration)
- Dynamic class loading (Truffle)
- Polyglot execution (JS, Python, Ruby)

The native executable includes:
- AOT-compiled static code
- Optional JIT compiler for dynamic code
- Truffle framework for polyglot execution
```

**Why This Output**: GraalVM Native Image can include a JIT compiler (Java on Truffle) within the native executable, allowing dynamic features like reflection and dynamic class loading to work. The AOT-compiled code handles the static paths, while the embedded JIT compiler handles dynamic behavior. This "mix" of AOT and JIT is unique to GraalVM and enables use cases like JShell within a native executable.

---

### Real-World Cases

- **Serverless Functions (AWS Lambda, Azure Functions)**: Native images achieve millisecond cold starts, eliminating the JVM warm-up penalty.
- **CLI Tools**: Native executables start instantly and don't require a JVM installation on the user's machine.
- **Microservices**: Lower memory footprint allows higher container density and faster scaling.
- **Spring Boot 3**: Built-in AOT support enables native image generation for production deployments.
- **Polyglot Applications**: GraalVM allows mixing Java, JavaScript, Python, and Ruby in a single native executable.

### References

- Oracle GraalVM Native Image Overview - https://docs.oracle.com/en/graalvm/
- GraalVM Native Image - https://www.graalvm.org/
- Spring Boot AOT Processing - https://docs.spring.io/
- Project Leyden - https://openjdk.org/projects/leyden/

---

## Deprecation and Safety Notes

| Feature | Status | Notes |
|---------|--------|-------|
| `-XX:+UseParallelGC` | Deprecated (JDK 14+) | Consider G1, ZGC, or Shenandoah. |
| `-Xint` | Active | Forces interpreted mode; useful for debugging. |
| `-Xcomp` | Active | Forces compilation; rarely useful for production. |
| `-XX:-TieredCompilation` | Active | Disables tiered compilation; methods go directly to C2. |
| `-XX:+PrintCompilation` | Active | Prints compilation events; useful for JIT analysis. |
| `-XX:+PrintInlining` | Active | Requires `-XX:+UnlockDiagnosticVMOptions`. |
| `-XX:+DoEscapeAnalysis` | Default (enabled) | Enables escape analysis; disable for debugging. |
| `-XX:MaxInlineSize` | Active | Maximum bytecode size for inlining (default: 35). |
| GraalVM Native Image | Active | Requires configuration for reflection and dynamic features. |
| Project Leyden | In development | Aims to bring AOT benefits to standard JDK. |

---

## References

### Official Specifications

- The Java Virtual Machine Specification, Java SE 26 Edition - https://docs.oracle.com/javase/specs/jvms/se26/html/index.html
- The Java HotSpot VM Under the Hood - https://cr.openjdk.org/~thartmann/talks/2017-Hotspot_Under_The_Hood.pdf
- Compilation Levels - https://cr.openjdk.org/~iveresov/tiered/Tiered.pdf

### OpenJDK Resources

- Tiered Compilation in HotSpot - https://cr.openjdk.org/~thartmann/talks/2016-Hotspot_Under_The_Hood.pdf
- Loop Optimizations in C2 - https://wiki.openjdk.org/display/HotSpot/Loop+Optimizations
- HotSpot Compilation Policy - https://cr.openjdk.org/
- JVM Anatomy Quark #18: Scalar Replacement - https://shipilev.net/jvm/anatomy-quarks/18-scalar-replacement/

### GraalVM Resources

- Oracle GraalVM Native Image Overview - https://docs.oracle.com/en/graalvm/
- GraalVM Native Image - https://www.graalvm.org/
- Graal Compiler - https://docs.oracle.com/en/graalvm/

### Academic and Technical Resources

- Understanding and Finding JIT Compiler Performance Bugs - https://dl.acm.org/doi/10.1145/3632947
- Run-Time Support for Optimizations Based on Escape Analysis - https://ieeexplore.ieee.org/
- Partial Escape Analysis and Scalar Replacement for Java - https://dl.acm.org/
- Exploring Single and Multilevel JIT Compilation Policy for Modern Machines - https://dl.acm.org/

### Tutorials and Guides

- Inside.java Episode 64: JIT Compiler From the Ground Up - https://inside.java/
- How Does the JVM Decide to JIT-Compile a Method? - https://browse.library.kiwix.org/content/stackoverflow.com_en_all_2023-11/
- Spring Boot AOT Processing - https://docs.spring.io/
- Customize Java Runtime Startup Behavior for Lambda Functions - https://docs.aws.amazon.com/