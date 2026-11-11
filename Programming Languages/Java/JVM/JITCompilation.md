# Java JIT Compilation: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Just-In-Time (JIT) compilation is the dynamic translation of Java bytecode into native machine code at runtime, performed by the JVM to improve execution speed beyond what interpretation alone can achieve.

**Technical Definition:** The Java HotSpot VM features a Java bytecode interpreter in addition to two JIT compilers — the client compiler (C1) and the server compiler (C2). The VM starts by interpreting bytecode and uses runtime profiling to identify "hot" methods — those executed a predetermined number of times — which are then compiled to native code. HotSpot provides a speedup of approximately 100× relative to interpretation and employs aggressive optimistic optimizations. Tiered compilation, the default mode since Java 8, combines C1 and C2: methods are first interpreted (Tier 0), then compiled by C1 (Tiers 1–3), and finally compiled by C2 (Tier 4) for peak performance.

**Beginner-Friendly Explanation:** Imagine reading a recipe in a foreign language. At first, you translate each sentence one at a time (interpretation) — it works, but it's slow. After you've made the dish a few times (the method becomes "hot"), you write down the translation in your own language on a sticky note (JIT compilation). The next time you cook, you read the sticky note directly and finish much faster. The JIT compiler is the automated translator that watches which recipes you use most and creates optimised native versions for exactly those. It never translates recipes you never cook, saving time and effort.

### Key Characteristics

- **Runtime Optimisation:** Compilation happens during execution, not ahead of time.
- **Profile-Guided:** Hot methods are identified by invocation counts and back-edge (loop) counts.
- **Two-Tier Compilers:** C1 (client) compiles quickly with limited optimizations; C2 (server) compiles slowly but generates highly optimised code.
- **Tiered Compilation:** The default mode smoothly transitions methods from interpretation to C1 to C2.
- **Speculative Optimisation:** The JIT makes optimistic assumptions based on runtime profiles and can deoptimise (fall back to interpretation) if assumptions fail.
- **On-Stack Replacement (OSR):** Hot loops within a method can be compiled and replaced while the method is still running, without waiting for the method to be invoked again.
- **Adaptive:** The JIT continuously observes program behaviour and adjusts compilation decisions accordingly.

### Prerequisites

- Basic understanding of the JVM architecture and bytecode
- Familiarity with the Java compilation model (`javac` → `.class` → execution)
- Knowledge of methods, loops, and object allocation
- Understanding of performance concepts (latency, throughput, warm-up)

### Related Programming Areas

- **JVM Architecture:** The execution engine, runtime data areas, and class loading.
- **Performance Tuning:** JVM flags, warm-up strategies, and code cache management.
- **Profiling and Monitoring:** `-XX:+PrintCompilation`, JITWatch, and JMH.
- **Escape Analysis:** Stack allocation, scalar replacement, and lock elision.
- **Memory Model:** Object publication and deoptimisation.

---

## Core Concepts / Features

---

## 1. JIT Compilation

### Definitions

**Core Definition:** JIT compilation is the process of converting bytecode into native machine code during program execution, triggered by the method becoming "hot".

**Technical Definition:** "HotSpot VM defaults to interpreting Java byte code. It compiles (JIT compilation) methods that runtime profiling determines to be 'hot', that is, the methods that are executed for a predetermined number of times". The compilation system provides a speedup of approximately 100× relative to interpretation and uses aggressive optimistic optimizations.

**Beginner-Friendly Explanation:** JIT compilation is the JVM's way of "learning on the job." It watches your code run, spots the parts you use most, and converts them into fast machine code so they run better next time.

### Purposes

- To improve execution speed by translating hot bytecode to native machine code
- To balance startup time and peak performance through tiered compilation
- To apply runtime-profile-guided optimizations that are impossible ahead of time
- To adapt to the actual behaviour of the running application

### Syntax Rules and Structure

JIT compilation is controlled by JVM flags. Key options include:

| Flag | Description |
|------|-------------|
| `-XX:+TieredCompilation` | Enables tiered compilation (default in Java 8+) |
| `-XX:-TieredCompilation` | Disables tiered compilation |
| `-Xint` | Runs in interpreter-only mode (no JIT) |
| `-XX:+PrintCompilation` | Logs JIT compilation activity |

**Syntax Rules:**

- The JIT compiles a method after its invocation count (or loop back-edge count) exceeds a threshold.
- Tiered compilation uses different thresholds for each tier (C1 vs C2).
- The code cache stores JIT-compiled methods; its size can be tuned.

### Annotated Complete Step-by-Step Code Examples

```java
// File: JitDemo.java
public class JitDemo {
    // This method will become "hot" and be JIT-compiled
    static long compute(int n) {
        long sum = 0;
        for (int i = 0; i < n; i++) {
            sum += (long) i * i;
        }
        return sum;
    }

    public static void main(String[] args) {
        long start = System.nanoTime();
        long result = 0;
        // Call many times to trigger JIT compilation
        for (int i = 0; i < 100_000; i++) {
            result = compute(1000);
        }
        long elapsed = (System.nanoTime() - start) / 1_000_000;
        System.out.println("Result: " + result + " (" + elapsed + " ms)");
    }
}
```

**Expected Output (varies):**
```
Result: 332833500 (12 ms)
```

**Step-by-Step Explanation:**

1. `compute` is called 100,000 times. The JVM's profiling counters record the invocation count and the loop's back-edge count.
2. Once the threshold is exceeded, the method is queued for compilation.
3. C1 (and later C2, if tiered) compiles `compute` to native machine code.
4. Subsequent invocations execute the optimised native code, reducing execution time.
5. The final elapsed time reflects both interpreted and compiled execution phases.

### Real-World Cases

- **Server Applications:** Long-running services benefit from C2's aggressive optimizations.
- **Startup Performance:** C1 compilation provides faster warm-up for short-lived processes.
- **Adaptive Optimisation:** Deoptimisation handles cases where speculative optimizations fail.

---

## 2. HotSpot

### Definitions

**Core Definition:** HotSpot is the default JVM implementation (and its JIT compilation subsystem) that identifies "hot spots" in code and optimises them at runtime.

**Technical Definition:** "HotSpot features a Java byte code interpreter in addition to two different Just In Time (JIT) compilers: client (also known as C1) and server (also known as C2)". The name "HotSpot" reflects its core strategy: rather than compiling everything, it focuses compilation effort on the "hot spots" — the methods and loops that are executed most frequently.

**Beginner-Friendly Explanation:** HotSpot is the engine that makes Java fast. It's called "HotSpot" because it focuses its effort on the hot spots — the parts of your code that run the most.

### Purposes

- To provide a high-performance execution environment for Java applications
- To balance interpretation, fast compilation (C1), and aggressive optimization (C2)
- To adapt to the runtime behaviour of the application
- To support a wide range of workloads from short-lived CLI tools to long-running servers

### Syntax Rules and Structure

**HotSpot Execution Tiers:**

| Tier | Name | Description |
|------|------|-------------|
| 0 | Interpreter | Executes bytecode with profiling |
| 1 | C1 (pure) | Fast compilation, simple optimizations |
| 2 | C1 (with counters) | C1 with invocation/back-edge counting |
| 3 | C1 (full profiling) | C1 with full profiling |
| 4 | C2 | Highly optimised server compiler |

*Source: *

**Syntax Rules:**

- HotSpot starts in the interpreter and escalates methods through tiers based on profiling data.
- Tiered compilation is the default for 64-bit server VMs.
- On-stack replacement (OSR) allows hot loops to be compiled while the enclosing method is still running.

### Annotated Code Example

```java
// File: HotSpotTierDemo.java
public class HotSpotTierDemo {
    public static void main(String[] args) {
        // Run with: java -XX:+PrintCompilation HotSpotTierDemo
        for (int i = 0; i < 20_000; i++) {
            work(i);
        }
    }

    static void work(int n) {
        long sum = 0;
        for (int i = 0; i < 100; i++) {
            sum += n * i;
        }
    }
}
```

**Expected Output (log excerpt with `-XX:+PrintCompilation`):**
```
  123   1       3       HotSpotTierDemo::work (18 bytes)
  456   2       4       HotSpotTierDemo::work (18 bytes)
```

**Step-by-Step Explanation:**

1. The `work` method is initially interpreted (Tier 0).
2. As calls accumulate, C1 compiles it with profiling (Tier 3).
3. Once the C2 threshold is reached, C2 compiles the method with aggressive optimizations (Tier 4).
4. The `-XX:+PrintCompilation` log shows the tier transitions.

### Real-World Cases

- **Web Servers:** HotSpot's tiered compilation provides fast startup and high peak throughput.
- **Trading Systems:** C2's optimizations are critical for low-latency paths.
- **Big Data Frameworks:** Spark and Kafka rely on HotSpot's adaptive compilation.

---

## 3. C1 Compiler

### Definitions

**Core Definition:** The C1 compiler (client compiler) is a fast, lightly optimising JIT compiler designed for quick startup and small memory footprint.

**Technical Definition:** "C1 compiler: Limited set of optimizations, fast compilation, small footprint". "Client compiler: It compiles methods quickly but emits machine code that is less optimized than the server compiler. This compiler is used for quick startup. Also, in this compiler, the smaller memory footprint is more important than steady-state performance". C1 also generates compiled versions of methods that collect profiling information about themselves, feeding data to C2.

**Beginner-Friendly Explanation:** C1 is the "quick responder." It gets your code running fast without spending too much time on optimization. It's great for applications that need to start up quickly and don't run for long.

### Purposes

- To provide fast compilation and quick application startup
- To keep memory footprint small
- To collect profiling information for C2 in tiered compilation
- To serve as the first compiled tier in the tiered scheme

### Syntax Rules and Structure

- **Compilation Speed:** Fast.
- **Optimisation Scope:** Limited set of simple, reliable optimizations (inlining, constant propagation, dead code elimination).
- **Role in Tiered Compilation:** Provides compiled code with profiling (Tiers 1–3).

### Annotated Code Example

```java
// File: C1Demo.java
public class C1Demo {
    static int add(int a, int b) {
        return a + b;
    }

    public static void main(String[] args) {
        // Run with: java -XX:TieredStopAtLevel=1 C1Demo
        int sum = 0;
        for (int i = 0; i < 10_000; i++) {
            sum = add(sum, i);
        }
        System.out.println("Sum: " + sum);
    }
}
```

**Expected Output:**
```
Sum: 49995000
```

**Step-by-Step Explanation:**

1. With `-XX:TieredStopAtLevel=1`, compilation stops at C1 (pure C1).
2. `add` is compiled by C1 with simple optimizations.
3. The loop runs with C1-compiled code, providing fast startup but not the peak performance of C2.

### Real-World Cases

- **CLI Tools:** Fast startup is more important than peak throughput.
- **Client Applications:** Desktop apps benefit from quick warm-up.
- **Tiered Warm-up:** C1 prepares methods for C2 in long-running applications.

---

## 4. C2 Compiler

### Definitions

**Core Definition:** The C2 compiler (server compiler) is an aggressively optimising JIT compiler designed for peak steady-state performance.

**Technical Definition:** "Server compiler: The compiler often takes more time (and memory) to compile the same methods. However, it generates better optimized machine code than the code generated by the client compiler. It provides better runtime performance after the application reaches the steady state". C2 performs a wide range of advanced optimizations, including dead code elimination, loop unrolling, loop invariant hoisting, escape analysis, and full inlining.

**Beginner-Friendly Explanation:** C2 is the "perfectionist." It takes longer to do its job, but the result is highly optimised machine code that runs as fast as possible. It's ideal for long-running server applications where peak performance matters.

### Purposes

- To generate highly optimised native code for peak performance
- To apply advanced optimizations based on runtime profiling
- To handle the steady-state execution of long-running applications
- To serve as the final tier in tiered compilation

### Syntax Rules and Structure

- **Compilation Speed:** Slower than C1, uses more memory.
- **Optimisation Scope:** Aggressive, including speculative and profile-guided optimizations.
- **Role in Tiered Compilation:** Final tier (Tier 4).

### Annotated Code Example

```java
// File: C2Demo.java
public class C2Demo {
    static long fib(int n) {
        if (n <= 1) return n;
        return fib(n - 1) + fib(n - 2);
    }

    public static void main(String[] args) {
        // Run with: java -XX:-TieredCompilation C2Demo
        long start = System.nanoTime();
        long result = fib(35);
        long elapsed = (System.nanoTime() - start) / 1_000_000;
        System.out.println("Fib(35): " + result + " (" + elapsed + " ms)");
    }
}
```

**Expected Output (varies):**
```
Fib(35): 9227465 (45 ms)
```

**Step-by-Step Explanation:**

1. With tiered compilation disabled (`-XX:-TieredCompilation`), methods are compiled directly by C2 after the interpreter threshold is reached.
2. `fib` is a recursive method that benefits heavily from C2's inlining and other optimizations.
3. The C2-compiled code executes much faster than interpreted or C1-compiled code.

### Real-World Cases

- **Application Servers:** Tomcat, Jetty, and Spring Boot benefit from C2's peak performance.
- **Financial Systems:** Low-latency trading relies on C2's aggressive optimizations.
- **Big Data Processing:** Spark and Flink use C2 for CPU-intensive transformations.

---

## 5. Tiered Compilation

### Definitions

**Core Definition:** Tiered compilation is a strategy that combines the C1 and C2 compilers, starting with fast C1 compilation and progressively moving hot methods to C2 for peak performance.

**Technical Definition:** "Tiered compilation enhances the server VM startup speed equivalent to the client VM speed by using client compiler as the first tier. A server VM uses the interpreter to collect the profiling information about the methods that is fed into the compiler". "The tiered scheme can also achieve better peak performance than a regular server VM. This is because the faster profiling phase allows a longer period of profiling, which yields better optimization". Tiered compilation is the default mode for the server VM since Java 8.

**Beginner-Friendly Explanation:** Tiered compilation is the "best of both worlds." It starts with the fast C1 compiler to get your application running quickly, then gradually upgrades hot methods to the highly optimising C2 compiler for peak speed. It's like starting a car in first gear and smoothly shifting up to fifth.

### Purposes

- To combine fast startup (C1) with high peak performance (C2)
- To provide better profiling data for C2 by using compiled C1 code
- To balance resource usage and performance
- To be the default execution mode for modern JVMs

### Syntax Rules and Structure

**Tiered Compilation Levels:**

| Level | Compiler | Description |
|-------|----------|-------------|
| 0 | Interpreter | Profiling only |
| 1 | C1 | No profiling |
| 2 | C1 | Limited profiling |
| 3 | C1 | Full profiling |
| 4 | C2 | Aggressive optimization |

*Source: *

**Syntax Rules:**

- Tiered compilation is enabled by default (`-XX:+TieredCompilation`).
- Thresholds for each tier can be tuned via JVM flags.
- `-XX:TieredStopAtLevel=1` forces C1-only compilation for fast startup.

### Annotated Code Example

```java
// File: TieredCompilationDemo.java
public class TieredCompilationDemo {
    public static void main(String[] args) {
        // Run with: java -XX:+TieredCompilation TieredCompilationDemo
        long sum = 0;
        for (int i = 0; i < 1_000_000; i++) {
            sum += compute(i);
        }
        System.out.println("Sum: " + sum);
    }

    static long compute(int n) {
        return (long) n * n;
    }
}
```

**Expected Output:**
```
Sum: 332833500
```

**Step-by-Step Explanation:**

1. `compute` is initially interpreted (Tier 0).
2. After enough invocations, C1 compiles it with profiling (Tier 3).
3. Once C2's threshold is reached, C2 recompiles it with aggressive optimizations (Tier 4).
4. The JIT log (`-XX:+PrintCompilation`) would show the tier transitions.

### Real-World Cases

- **Default Configuration:** Modern JVMs (Java 8+) use tiered compilation by default for all applications.
- **Server Workloads:** Long-running services benefit from the smooth warm-up and peak performance.
- **Serverless Functions:** Short-lived functions may use `-XX:TieredStopAtLevel=1` for faster cold starts.

---

## 6. Inlining

### Definitions

**Core Definition:** Inlining is the JIT optimization that replaces a method call with the body of the called method, eliminating call overhead and enabling further optimizations.

**Technical Definition:** Inlining is a foundational optimization that the JIT performs heavily to maximize opportunities for type propagation and other optimizations. For virtual calls, the JIT can speculatively inline a target method if profiling shows that only one implementation is used; if the class hierarchy changes later, the compiled code is deoptimised. Inlining is controlled by flags such as `MaxInlineSize` (default 35 bytes) and `FreqInlineSize` (default 325 bytes on Linux x86_64).

**Beginner-Friendly Explanation:** Inlining is like replacing a "call the helper" instruction with the helper's actual work. Instead of stopping, calling a function, and returning, the JIT copies the function's code directly into the caller. This saves the overhead of the call and lets the optimiser do an even better job.

### Purposes

- To eliminate method call overhead
- To enable further optimizations (constant propagation, dead code elimination) across method boundaries
- To improve code locality and instruction cache behaviour
- To devirtualise virtual calls through speculative inlining

### Syntax Rules and Structure

**Key Inlining Flags:**

| Flag | Default | Description |
|------|---------|-------------|
| `MaxInlineSize` | 35 bytes | Maximum bytecode size for a method to be inlined regardless of hotness |
| `FreqInlineSize` | 325 bytes (Linux x86_64) | Maximum bytecode size for hot call sites |
| `MaxInlineLevel` | 9 | Maximum depth of nested inlining |

**Syntax Rules:**

- Small methods (`< MaxInlineSize`) are inlined eagerly.
- Hot call sites can inline larger methods (up to `FreqInlineSize`).
- Virtual calls are inlined speculatively based on profile data; deoptimisation occurs if the assumption fails.

**Constraints and Limitations:**

- Inlining increases code size; excessive inlining can pressure the code cache.
- Very large methods are not inlined.

### Annotated Code Example

```java
// File: InliningDemo.java
public class InliningDemo {
    // Small method — a candidate for inlining
    static int square(int x) {
        return x * x;
    }

    public static void main(String[] args) {
        long sum = 0;
        for (int i = 0; i < 1_000_000; i++) {
            sum += square(i);  // JIT inlines square into the loop
        }
        System.out.println("Sum: " + sum);
    }
}
```

**Expected Output:**
```
Sum: 332833500
```

**Step-by-Step Explanation:**

1. `square` is a tiny method (a few bytes of bytecode), well under `MaxInlineSize`.
2. The JIT inlines `square` into the loop body, eliminating the method call overhead.
3. The optimised code performs the multiplication directly inside the loop.

### Real-World Cases

- **Getters and Setters:** Trivial accessor methods are prime inlining candidates.
- **Stream Pipelines:** Lambda bodies and functional operations are heavily inlined.
- **Virtual Calls:** Speculative inlining of monomorphic call sites is critical for object-oriented code performance.

---

## 7. Escape Analysis

### Definitions

**Core Definition:** Escape analysis is a JIT optimization that determines whether an object is accessible outside the method or thread that created it, enabling optimizations such as scalar replacement and lock elision.

**Technical Definition:** "Escape Analysis allows a compiler to determine whether an object is accessible outside the allocating method or thread. This information is used to perform optimizations such as Scalar Replacement, Stack Allocation and Lock Elision". In HotSpot, escape analysis currently performs scalar replacement (placing object fields into registers) rather than true stack allocation.

**Beginner-Friendly Explanation:** Escape analysis asks: "Does this object ever leave the method that created it?" If the answer is no (the object "does not escape"), the JIT can avoid allocating it on the heap entirely — it can keep the object's fields in registers (scalar replacement). This reduces garbage collection pressure and speeds up execution.

### Purposes

- To eliminate heap allocation for non-escaping objects (scalar replacement)
- To reduce garbage collection pressure
- To enable lock elision for objects that do not escape their thread
- To improve memory access performance

### Syntax Rules and Structure

- Escape analysis is performed automatically by C2; no flags are required.
- Objects can be classified as `NoEscape`, `ArgEscape`, or `GlobalEscape`.
- `NoEscape` objects are candidates for scalar replacement.

### Annotated Code Example

```java
// File: EscapeAnalysisDemo.java
public class EscapeAnalysisDemo {
    static class Point {
        int x, y;
        Point(int x, int y) { this.x = x; this.y = y; }
    }

    static int sum() {
        int total = 0;
        for (int i = 0; i < 1_000_000; i++) {
            Point p = new Point(i, i);  // Does not escape
            total += p.x + p.y;
        }
        return total;
    }

    public static void main(String[] args) {
        long start = System.nanoTime();
        int result = sum();
        long elapsed = (System.nanoTime() - start) / 1_000_000;
        System.out.println("Result: " + result + " (" + elapsed + " ms)");
    }
}
```

**Expected Output (varies):**
```
Result: 999999000000 (5 ms)
```

**Step-by-Step Explanation:**

1. The `Point` object `p` is created inside the loop and never used outside it — it does not escape.
2. C2's escape analysis identifies `p` as `NoEscape`.
3. Scalar replacement eliminates the heap allocation, keeping `x` and `y` in registers.
4. The loop runs faster and generates no garbage from `Point` allocations.

### Real-World Cases

- **Iterators:** Short-lived iterator objects that never escape can be scalar-replaced.
- **Temporary Objects:** Intermediate objects in stream pipelines often do not escape.
- **Lock Elision:** Objects used only within a single thread can have their synchronization elided.

---

## 8. Loop Optimization

### Definitions

**Core Definition:** Loop optimization is a family of JIT techniques that improve the performance of loops, including loop unrolling, loop invariant hoisting, and range check elimination.

**Technical Definition:** C2 performs "dead code elimination, loop unrolling, loop invariant hoisting, common subexpression elimination, constant propagation full inlining, full de-optimization (back to level 0) escape analysis". The server compiler prefers loops with an `int` counter, a constant stride, and a loop-invariant limit. Loops are typically peeled by one iteration to "shake out" loop-invariant tests.

**Beginner-Friendly Explanation:** Loop optimization makes repetitive code run faster by reducing redundant work. Loop unrolling does several iterations at once to reduce loop overhead. Loop invariant hoisting moves work that doesn't change out of the loop. Range check elimination removes redundant array bounds checks.

### Purposes

- To reduce loop overhead (counter increments, condition checks)
- To eliminate redundant computations inside loops
- To enable vectorisation and parallelisation
- To improve instruction-level parallelism

### Key Loop Optimizations Table

| Optimization | Description |
|--------------|-------------|
| **Loop Unrolling** | Replicates the loop body to reduce iteration overhead |
| **Loop Invariant Hoisting** | Moves computations that do not change out of the loop |
| **Range Check Elimination** | Removes redundant array bounds checks for loop-invariant limits |
| **Loop Peeling** | Peels one iteration to handle edge cases separately |
| **On-Stack Replacement (OSR)** | Compiles hot loops while the method is still running |

### Annotated Code Example

```java
// File: LoopOptimizationDemo.java
public class LoopOptimizationDemo {
    static int sumArray(int[] array) {
        int sum = 0;
        // JIT optimises this loop: hoists array.length, eliminates range checks
        for (int i = 0; i < array.length; i++) {
            sum += array[i];
        }
        return sum;
    }

    public static void main(String[] args) {
        int[] data = new int[1_000_000];
        for (int i = 0; i < data.length; i++) data[i] = i;
        long start = System.nanoTime();
        int result = sumArray(data);
        long elapsed = (System.nanoTime() - start) / 1_000_000;
        System.out.println("Sum: " + result + " (" + elapsed + " ms)");
    }
}
```

**Expected Output (varies):**
```
Sum: 1783293664 (2 ms)
```

**Step-by-Step Explanation:**

1. `array.length` is loop-invariant; the JIT hoists it out of the loop.
2. The JIT eliminates redundant range checks because `i` is bounded by the hoisted length.
3. The loop may be unrolled to process multiple elements per iteration.
4. The optimised loop runs significantly faster.

### Real-World Cases

- **Array Processing:** Summation, filtering, and transformation of large arrays.
- **Matrix Operations:** Nested loops in linear algebra benefit from unrolling and invariant hoisting.
- **Stream Pipelines:** Internal iteration in the Streams API relies on loop optimizations.

---

## 9. Dead Code Elimination

### Definitions

**Core Definition:** Dead code elimination is the JIT optimization that removes code whose results are never used or whose execution can never affect the program's observable behaviour.

**Technical Definition:** C2 performs "dead code elimination" as part of its optimization pipeline. The JIT uses profiling to identify branches that are never taken and converts them into "uncommon traps" — code paths that are effectively free unless the impossible condition actually occurs. Dead code elimination is often enabled by escape analysis: if an object is never used, its allocation and initialization can be removed entirely.

**Beginner-Friendly Explanation:** Dead code elimination is like a cleaning crew that removes instructions that don't do anything useful. If you compute a value but never use it, the JIT can delete the computation. If a branch is never taken, the JIT can make it nearly free.

### Purposes

- To reduce the amount of executed code
- To improve instruction cache efficiency
- To eliminate unnecessary object allocations
- To simplify the code for further optimizations

### Syntax Rules and Structure

- Dead code elimination is performed automatically by C1 and C2.
- It works in conjunction with escape analysis: non-escaping unused objects are eliminated.
- Profiling data identifies branches that are never taken; these become uncommon traps.

### Annotated Code Example

```java
// File: DeadCodeDemo.java
public class DeadCodeDemo {
    static int compute(int n) {
        int unused = n * 42;  // Dead code — value never used
        return n + 1;
    }

    public static void main(String[] args) {
        long sum = 0;
        for (int i = 0; i < 1_000_000; i++) {
            sum += compute(i);
        }
        System.out.println("Sum: " + sum);
    }
}
```

**Expected Output:**
```
Sum: 500000500000
```

**Step-by-Step Explanation:**

1. `unused` is computed but never read.
2. The JIT's dead code elimination removes the `n * 42` computation entirely.
3. Only `n + 1` remains, reducing the work per call.

### Real-World Cases

- **Debug Code:** Logging or diagnostic code guarded by conditions that are always false in production can be eliminated.
- **Unused Computations:** Intermediate values that are never read are removed.
- **Branch Elimination:** Branches that profiling shows are never taken become uncommon traps.

---

## 10. JIT Best Practices

### Definitions

**Core Definition:** JIT best practices are guidelines for writing code and configuring the JVM to work optimally with the JIT compiler, maximizing warm-up speed and steady-state performance.

**Technical Definition:** Best practices derive from JVM documentation and performance engineering experience. Key principles include writing small, focused methods (to enable inlining), avoiding megamorphic call sites (to enable devirtualisation and speculative inlining), warming up the JIT before measuring performance, and monitoring compilation activity with tools like `-XX:+PrintCompilation` and JITWatch.

**Beginner-Friendly Explanation:** Best practices help you help the JIT. Write code that is easy for the JIT to optimise — small methods, simple loops, predictable types — and configure the JVM appropriately for your workload.

### Key Best Practices Table

| Practice | Description |
|----------|-------------|
| **Write small, focused methods** | Methods under `MaxInlineSize` (35 bytes) are inlined eagerly; small methods enable further optimizations. |
| **Avoid megamorphic call sites** | Call sites with many implementations cannot be inlined or devirtualised. |
| **Use `int` counters for loops** | The server compiler optimises loops with `int` counters, constant strides, and invariant limits. |
| **Warm up the JIT** | Run representative workloads before measuring or serving production traffic to allow C2 compilation. |
| **Enable tiered compilation** | It is the default and provides the best balance of startup and peak performance. |
| **Monitor compilation** | Use `-XX:+PrintCompilation` and JITWatch to understand what is being compiled and when. |
| **Tune inlining judiciously** | `MaxInlineSize` and `FreqInlineSize` can be adjusted, but defaults work well for most applications. |
| **Consider `-XX:TieredStopAtLevel=1` for short-lived apps** | For CLI tools and serverless functions, C1-only compilation reduces warm-up time. |
| **Keep code cache appropriately sized** | The code cache stores compiled methods; monitor and tune its size if needed. |
| **Avoid relying on JIT for correctness** | JIT optimizations are speculative; ensure your code is correct without them. |

### Annotated Complete Step-by-Step Code Examples

```java
// File: JitBestPractices.java
public class JitBestPractices {
    // BEST PRACTICE: Small method — easily inlined
    static int add(int a, int b) {
        return a + b;
    }

    public static void main(String[] args) throws Exception {
        // BEST PRACTICE: Warm up the JIT before measuring
        for (int i = 0; i < 100_000; i++) {
            add(i, i + 1);
        }

        // Now measure performance with JIT-compiled code
        long start = System.nanoTime();
        long sum = 0;
        for (int i = 0; i < 10_000_000; i++) {
            sum += add(i, i + 1);
        }
        long elapsed = (System.nanoTime() - start) / 1_000_000;

        System.out.println("Sum: " + sum + " (" + elapsed + " ms)");
    }
}
```

**Expected Output (varies):**
```
Sum: 100001000000000 (15 ms)
```

**Step-by-Step Explanation:**

1. `add` is a tiny method, well under `MaxInlineSize`; it is inlined into the loop.
2. The warm-up loop triggers JIT compilation of `main` and `add`.
3. The measurement loop runs with fully optimised native code.
4. The elapsed time reflects the steady-state performance of the JIT-compiled code.

### Real-World Cases

- **Benchmarking:** JMH (Java Microbenchmark Harness) handles warm-up and JIT profiling automatically to produce reliable measurements.
- **Server Deployment:** Warming up critical endpoints before routing production traffic prevents latency spikes during cold start.
- **Serverless:** Using `-XX:TieredStopAtLevel=1` reduces cold-start latency for short-lived functions.

---

## References

- The Java HotSpot VM Under the Hood — Tobias Hartmann, Oracle — https://cr.openjdk.org/~thartmann/talks/2017-Hotspot_Under_The_Hood.pdf
- Java Platform, Standard Edition JRockit to HotSpot Migration Guide — Compilation Optimization — https://docs.oracle.com/javase/9/jrockit-hotspot/compilation-optimization.htm
- Compiling Code, Just in Time — Manas Thakur, IIT Bombay — https://www.cse.iitb.ac.in/~manas/docs/slides/crs2023-manas.pdf
- Performance Techniques Used in the HotSpot JVM — OpenJDK Wiki — https://wiki.openjdk.org/display/HotSpot/PerformanceTechniques
- Partial Escape Analysis and Scalar Replacement for Java — Stadler et al. (2014) — https://scholar.google.fr/citations?user=... (via search result )
- JVM Anatomy Quark #18: Scalar Replacement — Aleksey Shipilev — https://shipilev.net/jvm/anatomy-quarks/18-scalar-replacement/
- Limits of JVM JIT-Compiler Optimizations — Stack Overflow — https://stackoverflow.com/questions/41806291/limits-of-jvm-jit-compiler-optimizations
- Inlining Tuning — OpenJDK — https://cr.openjdk.org/~vlivanov/talks/2016-JIT_meetup_Inlining.pdf (via search result )
- Java JIT Compilation Optimization — OneUptime Blog — https://github.com/OneUptime/blog/blob/master/posts/2026-01-25-jit-compilation-optimization/README.md
- Java JIT Warm-up / JIT Compilation — awesome-agv (GitHub) — https://github.com/irahardianto/awesome-agv/blob/main/.agents/skills/perf-optimization/languages/java.md
- `MaxInlineSize` and `FreqInlineSize` — OpenJDK Man Pages — https://manpages.debian.org/unstable/openjdk-27-jre-headless/java.1.en.html
- Java JIT Compiler Options — Oracle JRockit to HotSpot Migration Guide — https://docs.oracle.com/javase/9/jrockit-hotspot/compilation-optimization.htm