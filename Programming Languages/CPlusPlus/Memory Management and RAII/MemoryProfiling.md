# C++ Memory Profiling — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Memory profiling is the practice of measuring and analysing a program's memory usage—how much it allocates, where allocations come from, how memory is distributed over time, and whether any memory is leaked or misused.

**Technical Definition**
Memory profiling encompasses a family of dynamic analysis techniques and tools that observe a program's runtime memory behaviour. These include **leak detection** (finding memory that is never freed), **heap profiling** (tracking allocation sizes, counts, and call stacks), **memory error detection** (catching use-after-free, buffer overflows, and uninitialised reads), and **memory usage visualisation** (plotting heap growth over time). The primary tools in the C++ ecosystem are **Valgrind** (with Memcheck, Massif, and Helgrind), **Heaptrack**, and the **Sanitizer** family (AddressSanitizer, LeakSanitizer, MemorySanitizer, ThreadSanitizer, UndefinedBehaviorSanitizer).

**Beginner-Friendly Explanation**
Memory profiling is like giving your program a medical check-up for its memory habits. It answers questions such as: “How much memory does my program use?”, “Where are the allocations coming from?”, “Am I forgetting to free anything?”, and “Am I reading memory I shouldn't?” Tools like Valgrind, Heaptrack, and the Sanitizers are the diagnostic instruments.

### Key Characteristics

- **Dynamic analysis** – profiling observes the program at runtime, unlike static analysis which examines source code.
- **Overhead trade-offs** – tools range from near-zero overhead (some sanitizers in certain modes) to 10–100× slowdown (Valgrind).
- **Complementary tools** – Valgrind excels at leak/error detection; Massif and Heaptrack excel at heap profiling; sanitizers excel at fast error detection in development and CI.
- **Build integration** – sanitizers require compiler flags (`-fsanitize=...`); Valgrind and Heaptrack work on existing binaries (with debug info).
- **Production vs. development** – some sanitizers (e.g., ASan) can run in production with tuned options; Valgrind is generally development-only due to overhead.

### Prerequisites

- Solid C++ knowledge, especially dynamic memory (`new`/`delete`, `malloc`/`free`) and smart pointers.
- Familiarity with build systems and compiler flags (GCC/Clang).
- Basic understanding of the heap, stack, and virtual memory.
- Awareness of debugging symbols (`-g`) and how to read stack traces.

### Related Programming Areas

- **Dynamic memory** – the primary subject of profiling.
- **Smart pointers** – RAII reduces leaks, but profiling verifies correctness.
- **Exception safety** – leaks often occur on exception paths; profiling catches them.
- **Concurrency** – ThreadSanitizer and Helgrind detect data races.
- **Performance engineering** – heap profiling reveals allocation hot spots.

### Core Concepts / Features

1. Memory profiling (the overarching practice)
2. Valgrind (Memcheck, Massif, Helgrind)
3. Massif
4. Heaptrack
5. AddressSanitizer (ASan)
6. LeakSanitizer (LSan)
7. MemorySanitizer (MSan)
8. Memory profiling best practices

---

## 1. Memory Profiling (The Overarching Practice)

### Definitions

**Core Definition**
Memory profiling is the systematic measurement of a program's memory allocation, usage, and deallocation behaviour to find leaks, errors, and inefficiencies.

**Technical Definition**
Memory profiling is a form of dynamic program analysis that instruments or observes a running program to record memory events: allocations, deallocations, reads, writes, and lifetime. The output typically includes allocation counts, sizes, call stacks, and timelines. Profiling serves two main goals: **correctness** (detecting leaks, double frees, use-after-free, uninitialised reads) and **performance** (identifying allocation hot spots, fragmentation, and peak memory usage).

**Beginner-Friendly Explanation**
Memory profiling is “watching your program use memory” so you can spot problems. It tells you where memory is going, whether it is being freed, and whether it is being used correctly.

### Purposes

- To detect memory leaks.
- To find invalid memory accesses (use-after-free, buffer overflows).
- To identify allocation hot spots and reduce memory usage.
- To measure peak memory consumption.
- To verify correctness of custom allocators and smart pointer usage.

### Syntax Rules and Structure

**Profiling Categories**

| Category | Goal | Representative Tools |
|----------|------|----------------------|
| Leak detection | Find unfreed memory | Valgrind Memcheck, LeakSanitizer |
| Error detection | Find invalid accesses | AddressSanitizer, Memcheck |
| Heap profiling | Measure allocation patterns | Massif, Heaptrack |
| Uninitialised reads | Find use of uninitialised memory | MemorySanitizer, Memcheck |
| Race detection | Find data races | ThreadSanitizer, Helgrind |

**Syntax Rules**
- Most tools require debug symbols (`-g`) for useful stack traces.
- Sanitizers require recompilation with `-fsanitize=...`.
- Valgrind and Heaptrack run on existing binaries (recompilation with `-g` recommended).

**Constraints and Limitations**
- Dynamic analysis only observes executed paths; unexercised code is not profiled.
- Overhead varies widely between tools.

### Annotated Code Example (Leak to Detect)

```cpp
#include <iostream>

void leak() {
    int* p = new int(42);              // never deleted
}

int main() {
    for (int i = 0; i < 10; ++i) leak();
    std::cout << "Done\n";
}
```

**Expected Output**
```
Done
```

**Why this output?**  
The program runs and prints `Done`, but it leaks ten `int`s. A memory profiler (Valgrind or LeakSanitizer) would report the leaks with stack traces pointing to `leak()`.

### Real-World Cases

- **CI pipelines** – running sanitizers on every test.
- **Development debugging** – using Valgrind to find leaks.
- **Production monitoring** – using ASan in a canary deployment.

---

## 2. Valgrind

### Definitions

**Core Definition**
Valgrind is a dynamic analysis framework that provides a suite of tools for memory error detection, leak detection, heap profiling, and thread analysis.

**Technical Definition**
Valgrind is an instrumentation framework that runs a program on a synthetic CPU, allowing it to observe every memory access and instruction. Its main tools are **Memcheck** (memory error and leak detection), **Massif** (heap profiling), **Helgrind** (thread error detection), **DRD** (data race detection), and **Cachegrind** (cache profiling). Valgrind binaries are typically 10–50× slower than native execution.

**Beginner-Friendly Explanation**
Valgrind is a “virtual machine” that watches your program do everything with memory. Its main tool, Memcheck, finds leaks and memory errors. It is slow but thorough.

### Purposes

- To detect memory leaks and invalid memory accesses.
- To profile heap usage over time (via Massif).
- To detect data races (via Helgrind/DRD).
- To analyse cache behaviour (via Cachegrind).

### Syntax Rules and Structure

**General Syntax**
```bash
valgrind --tool=memcheck --leak-check=full ./program
valgrind --tool=massif ./program
valgrind --tool=helgrind ./program
```

**Component Breakdown**

| Tool | Purpose |
|------|---------|
| Memcheck | Memory errors and leaks. |
| Massif | Heap profiling. |
| Helgrind | Thread errors (races, lock order). |
| DRD | Data race detection. |
| Cachegrind | Cache profiling. |

**Constraints and Limitations**
- Significant slowdown (10–50×).
- Requires debug symbols for useful output.
- May not work well with some JIT or GPU code.

### Annotated Code Example

```cpp
#include <iostream>

int main() {
    int* p = new int(42);
    std::cout << *p << '\n';
    // forgot delete
}
```

**Valgrind Command**
```bash
valgrind --leak-check=full ./program
```

**Expected Valgrind Output** (abridged)
```
==12345== HEAP SUMMARY:
==12345==     in use at exit: 4 bytes in 1 blocks
==12345==   total heap usage: 1 allocs, 0 frees, 4 bytes allocated
==12345==
==12345== 4 bytes in 1 blocks are definitely lost in loss record 1 of 1
==12345==    at 0x...: operator new(unsigned long)
==12345==    by 0x...: main (main.cpp:4)
```

**Why this output?**  
Memcheck detects the leaked `int` and reports the allocation site with a stack trace.

### Real-World Cases

- **Pre-release validation** – running the test suite under Memcheck.
- **Heap profiling** – using Massif to find peak memory usage.
- **Race detection** – using Helgrind on multithreaded code.

---

## 3. Massif

### Definitions

**Core Definition**
Massif is Valgrind's heap profiler, which measures how much heap memory a program uses over time and where the allocations originate.

**Technical Definition**
Massif is a Valgrind tool that profiles heap usage by taking snapshots of the heap at regular intervals. It records allocation sizes, call stacks, and a timeline, producing output that can be visualised with `ms_print` or massif-visualizer. It distinguishes between **useful heap** (directly allocated by the program) and **extra heap** (allocator overhead).

**Beginner-Friendly Explanation**
Massif is a “memory usage graph” tool. It shows how your program's heap grows and shrinks over time and which functions are responsible for the most memory.

### Purposes

- To measure peak heap usage.
- To identify functions that allocate the most memory.
- To visualise heap growth over time.
- To detect memory bloat and fragmentation.

### Syntax Rules and Structure

**General Syntax**
```bash
valgrind --tool=massif ./program
ms_print massif.out.<pid>
```

**Component Breakdown**

| Output | Description |
|--------|-------------|
| `massif.out.<pid>` | Raw profile data. |
| `ms_print` | Text-based visualisation. |
| massif-visualizer | Graphical visualisation (KDE). |

**Constraints and Limitations**
- Samples at intervals; very short allocations may be missed.
- Does not detect leaks (use Memcheck for that).

### Annotated Code Example

```cpp
#include <vector>

int main() {
    std::vector<int> v;
    for (int i = 0; i < 1'000'000; ++i) v.push_back(i);
}
```

**Command**
```bash
valgrind --tool=massif ./program
ms_print massif.out.*
```

**Expected Output** (abridged text graph)
```
    MB
3.5^                                                                      #
   |                                                                     :#
   |                                                                   :::#
   |                                                                 :::::#
   |                                                               :::::::#
   |                                                             :::::::::#
   |                                                           :::::::::::#
   |                                                         :::::::::::::#
   |                                                       :::::::::::::::#
   |                                                     :::::::::::::::::#
   |                                                   :::::::::::::::::::#
   |                                                 :::::::::::::::::::::#
   |                                               :::::::::::::::::::::::#
   |                                             :::::::::::::::::::::::::#
   |                                           :::::::::::::::::::::::::::#
   |                                         :::::::::::::::::::::::::::::#
   |                                       :::::::::::::::::::::::::::::::#
   |                                     :::::::::::::::::::::::::::::::::#
   |                                   :::::::::::::::::::::::::::::::::::#
   |                                 :::::::::::::::::::::::::::::::::::::#
   |                               :::::::::::::::::::::::::::::::::::::::#
   |                             :::::::::::::::::::::::::::::::::::::::::#
   |                           :::::::::::::::::::::::::::::::::::::::::::#
   |                         :::::::::::::::::::::::::::::::::::::::::::::#
   |                       :::::::::::::::::::::::::::::::::::::::::::::::#
   |                     :::::::::::::::::::::::::::::::::::::::::::::::::#
   |                   :::::::::::::::::::::::::::::::::::::::::::::::::::#
   |                 :::::::::::::::::::::::::::::::::::::::::::::::::::::#
   |               :::::::::::::::::::::::::::::::::::::::::::::::::::::::#
   |             :::::::::::::::::::::::::::::::::::::::::::::::::::::::::#
   |           :::::::::::::::::::::::::::::::::::::::::::::::::::::::::::#
   |         :::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::#
   |       :::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::#
   |     :::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::#
   |   :::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::#
   | :::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::#
0 +----------------------------------------------------------------------->Gi
```

**Why this output?**  
Massif shows the heap growing as the vector fills with one million integers (≈4 MB, plus overhead).

### Real-World Cases

- **Memory budget analysis** – finding peak memory usage.
- **Optimisation** – identifying allocation hot spots.

---

## 4. Heaptrack

### Definitions

**Core Definition**
Heaptrack is a heap profiler for Linux that records every allocation and deallocation, providing detailed reports on allocation counts, sizes, and call stacks.

**Technical Definition**
Heaptrack (from KDE) uses `LD_PRELOAD` to intercept allocation functions (`malloc`, `free`, `new`, `delete`) and records every allocation with its size and backtrace. It produces a data file that can be analysed with the `heaptrack` GUI or `heaptrack_print` CLI. It has lower overhead than Valgrind and is designed for profiling real workloads.

**Beginner-Friendly Explanation**
Heaptrack is a “lighter, faster” heap profiler than Valgrind. It tracks every allocation and shows you where memory is going, with a nice GUI.

### Purposes

- To profile heap allocations with low overhead.
- To identify allocation hot spots and leaks.
- To visualise allocation timelines and call graphs.
- To analyse memory usage in real applications.

### Syntax Rules and Structure

**General Syntax**
```bash
heaptrack ./program
heaptrack_gui heaptrack.program.<pid>.gz
heaptrack_print heaptrack.program.<pid>.gz
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `heaptrack` | Launcher that profiles the program. |
| `heaptrack_gui` | Graphical analysis. |
| `heaptrack_print` | Text analysis. |

**Constraints and Limitations**
- Linux-only.
- Intercepts allocations via `LD_PRELOAD`; may not catch all custom allocators.
- Overhead is lower than Valgrind but not zero.

### Annotated Code Example

```cpp
#include <vector>
#include <string>

int main() {
    std::vector<std::string> v;
    for (int i = 0; i < 100'000; ++i)
        v.push_back("item " + std::to_string(i));
}
```

**Command**
```bash
heaptrack ./program
heaptrack_print heaptrack.program.*.gz
```

**Expected Output** (abridged)
```
total runtime: 0.15s
calls to allocation functions: 200003
temporary allocations: 100001
peak heap memory consumption: 5.2MB
peak RSS: 12.1MB
total memory leaked: 0
```

**Why this output?**  
Heaptrack reports allocation counts, peak heap, and leaks, showing that the strings allocate memory but no leaks occur.

### Real-World Cases

- **Desktop application profiling** – finding allocation hot spots in KDE applications.
- **Server workload analysis** – measuring peak memory during request processing.

---

## 5. AddressSanitizer (ASan)

### Definitions

**Core Definition**
AddressSanitizer is a fast memory error detector that catches use-after-free, buffer overflows, stack-use-after-return, and other invalid memory accesses.

**Technical Definition**
ASan (part of the LLVM/GCC sanitizer suite) instruments memory accesses at compile time and uses shadow memory to track the validity of every byte. It detects heap-buffer-overflow, stack-buffer-overflow, global-buffer-overflow, use-after-free, use-after-scope, double-free, and invalid-free. Overhead is typically 2× slowdown and 2–3× memory. It is enabled with `-fsanitize=address`.

**Beginner-Friendly Explanation**
ASan is a “memory safety checker” that catches bugs like reading past the end of an array or using memory after it has been freed. It is fast enough to run in CI and even in production canary deployments.

### Purposes

- To detect buffer overflows (heap, stack, global).
- To detect use-after-free and double-free.
- To detect use-after-scope and stack-use-after-return.
- To run as part of CI and fuzzing.

### Syntax Rules and Structure

**General Syntax**
```bash
g++ -fsanitize=address -g program.cpp -o program
./program
```

**Component Breakdown**

| Error Type | Detected |
|------------|----------|
| Heap buffer overflow | Yes |
| Stack buffer overflow | Yes |
| Global buffer overflow | Yes |
| Use-after-free | Yes |
| Use-after-scope | Yes |
| Double-free | Yes |
| Invalid-free | Yes |

**Constraints and Limitations**
- 2× slowdown, 2–3× memory overhead.
- Does not detect uninitialised reads (use MSan).
- Requires recompilation.

### Annotated Code Example

```cpp
#include <iostream>

int main() {
    int* arr = new int[5];
    arr[5] = 42;                       // out-of-bounds write
    std::cout << arr[5] << '\n';
    delete[] arr;
}
```

**Command**
```bash
g++ -fsanitize=address -g program.cpp -o program
./program
```

**Expected ASan Output** (abridged)
```
==12345==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x...
WRITE of size 4 at 0x... thread T0
    #0 0x... in main program.cpp:5
...
```

**Why this output?**  
ASan detects the out-of-bounds write to `arr[5]` and reports the exact source location.

### Real-World Cases

- **CI pipelines** – running tests under ASan.
- **Fuzzing** – libFuzzer + ASan is a standard combination.
- **Production canaries** – ASan can run with `ASAN_OPTIONS` tuned for production.

---

## 6. LeakSanitizer (LSan)

### Definitions

**Core Definition**
LeakSanitizer is a memory leak detector that runs at program exit and reports memory that was allocated but never freed.

**Technical Definition**
LSan is a standalone leak detector that is also integrated into ASan. It performs a stop-the-world scan at exit, tracing reachability from roots. It reports direct leaks (unreachable blocks) and indirect leaks (blocks reachable only from leaked blocks). It is enabled with `-fsanitize=leak` (standalone) or `-fsanitize=address` (integrated).

**Beginner-Friendly Explanation**
LSan is a “leak checker” that runs when your program exits and tells you if you forgot to free anything.

### Purposes

- To detect memory leaks at program exit.
- To integrate leak detection into CI.
- To report leak locations with stack traces.

### Syntax Rules and Structure

**General Syntax**
```bash
g++ -fsanitize=leak -g program.cpp -o program
./program
```

**Component Breakdown**

| Feature | Description |
|---------|-------------|
| Direct leaks | Unreachable blocks. |
| Indirect leaks | Blocks reachable only from leaked blocks. |
| Suppressions | `LSAN_OPTIONS=suppressions=...` |

**Constraints and Limitations**
- Reports only at exit (not during execution).
- May report false positives with custom allocators.

### Annotated Code Example

```cpp
#include <iostream>

int main() {
    int* p = new int(42);              // never deleted
    std::cout << *p << '\n';
}
```

**Command**
```bash
g++ -fsanitize=leak -g program.cpp -o program
./program
```

**Expected LSan Output** (abridged)
```
=================================================================
==12345==ERROR: LeakSanitizer: detected memory leaks
Direct leak of 4 byte(s) in 1 object(s) allocated from:
    #0 0x... in operator new(unsigned long)
    #1 0x... in main program.cpp:4
```

**Why this output?**  
LSan detects the leaked `int` at exit and reports the allocation stack.

### Real-World Cases

- **Test suites** – ensuring no leaks in unit tests.
- **CI** – failing builds on leaks.

---

## 7. MemorySanitizer (MSan)

### Definitions

**Core Definition**
MemorySanitizer is a detector for uninitialised memory reads, catching cases where a program uses memory that was never initialised.

**Technical Definition**
MSan tracks the initialization state of every byte of memory and reports when an uninitialised value is used in a branch, system call, or other observable operation. It requires all code (including libraries) to be instrumented; uninstrumented libraries can cause false positives. It is enabled with `-fsanitize=memory`.

**Beginner-Friendly Explanation**
MSan catches “reading garbage” bugs—when your program uses memory that was never given a value. It needs your whole program (and its libraries) to be built with MSan.

### Purposes

- To detect use of uninitialised memory.
- To catch subtle bugs that may only manifest under certain conditions.
- To improve code correctness.

### Syntax Rules and Structure

**General Syntax**
```bash
clang++ -fsanitize=memory -g program.cpp -o program
./program
```

**Component Breakdown**

| Feature | Description |
|---------|-------------|
| Uninitialised reads | Detected |
| Origins tracking | `-fsanitize-memory-track-origins` |
| Instrumented libs | Required |

**Constraints and Limitations**
- Requires all dependencies to be instrumented.
- Higher overhead than ASan.
- Not supported by all compilers equally.

### Annotated Code Example

```cpp
#include <iostream>

int main() {
    int* p = new int;                  // uninitialized
    if (*p > 0)                        // MSan flags this read
        std::cout << "positive\n";
    delete p;
}
```

**Expected MSan Output** (abridged)
```
==12345==WARNING: MemorySanitizer: use-of-uninitialized-value
    #0 0x... in main program.cpp:5
```

**Why this output?**  
MSan detects the read of uninitialised memory at `*p` and reports the source location.

### Real-World Cases

- **Security-sensitive code** – catching uninitialised reads that could leak data.
- **Cryptography** – ensuring no uninitialised memory is used.

---

## 8. Memory Profiling Best Practices

### Definitions

**Core Definition**
Memory profiling best practices are guidelines for choosing, configuring, and integrating memory profiling tools into a development workflow.

**Technical Definition**
These practices are informed by the C++ Core Guidelines (especially CP.9: “Whenever feasible use tools to validate your concurrent code” and the resource management rules) and community consensus: always build with debug symbols; run sanitizers in CI; use Valgrind for thorough leak detection when sanitizers are insufficient; use Massif or Heaptrack for heap profiling; and integrate profiling into the development lifecycle rather than treating it as an afterthought.

**Beginner-Friendly Explanation**
Best practices for memory profiling are simple: always compile with `-g`, run sanitizers on every test, use Valgrind for deep analysis, and use Massif/Heaptrack to understand memory usage.

### Key Guidelines

| Guideline | Summary |
|-----------|---------|
| **Build with `-g`** | Debug symbols make profiles useful. |
| **Run sanitizers in CI** | ASan + LSan on every test run. |
| **Use MSan for uninitialised reads** | Especially in security-sensitive code. |
| **Use Valgrind for deep analysis** | When sanitizers miss something. |
| **Use Massif/Heaptrack for heap profiling** | To find peak memory and hot spots. |
| **Profile realistic workloads** | Synthetic tests may miss real allocation patterns. |
| **Monitor production** | ASan can run in canary deployments. |
| **Fix leaks promptly** | Leaks in libraries multiply in consumers. |
| **Document tool configurations** | Share suppressions and CI settings. |
| **Combine tools** | Each tool catches different classes of bugs. |

### Annotated Code Example

```cpp
#include <memory>
#include <vector>
#include <iostream>

int main() {
    // Good: RAII, no manual new/delete
    std::vector<std::unique_ptr<int>> v;
    for (int i = 0; i < 100; ++i)
        v.push_back(std::make_unique<int>(i));

    std::cout << v.size() << '\n';
} // everything freed automatically
```

**Command**
```bash
g++ -fsanitize=address,undefined -g program.cpp -o program
./program
```

**Expected Output**
```
100
```

**Why this output?**  
The program uses RAII and passes ASan/UBSan without errors, demonstrating good memory hygiene.

### Real-World Cases

- **CI pipelines** – running sanitizers on every commit.
- **Code reviews** – flagging code that fails sanitizers.
- **Release engineering** – running Valgrind on release candidates.

---

## References

- Valgrind – https://valgrind.org/
- Valgrind Memcheck manual – https://valgrind.org/docs/manual/mc-manual.html
- Valgrind Massif manual – https://valgrind.org/docs/manual/ms-manual.html
- Heaptrack — KDE – https://github.com/KDE/heaptrack
- AddressSanitizer — Clang documentation – https://clang.llvm.org/docs/AddressSanitizer.html
- LeakSanitizer — Clang documentation – https://clang.llvm.org/docs/LeakSanitizer.html
- MemorySanitizer — Clang documentation – https://clang.llvm.org/docs/MemorySanitizer.html
- ThreadSanitizer — Clang documentation – https://clang.llvm.org/docs/ThreadSanitizer.html
- C++ Core Guidelines — CP.9: Whenever feasible use tools to validate your concurrent code – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-tools
- Valgrind — Wikipedia – https://en.wikipedia.org/wiki/Valgrind
- AddressSanitizer — Wikipedia – https://en.wikipedia.org/wiki/AddressSanitizer