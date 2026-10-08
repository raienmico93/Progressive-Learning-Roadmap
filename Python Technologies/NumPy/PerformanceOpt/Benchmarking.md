# Benchmarking — A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Benchmarking is the systematic measurement of a program's performance (execution time, memory usage, or other resources) to compare implementations, identify bottlenecks, and validate optimizations.

**Technical Definition:** Benchmarking in Python/NumPy encompasses three complementary disciplines: (1) **micro-benchmarking** — measuring the time of small, isolated code snippets using `timeit`, `time.perf_counter`, or `%timeit`; (2) **CPU profiling** — attributing execution time to functions, lines, or call stacks using `cProfile`, `line_profiler`, or `py-spy`; and (3) **memory profiling** — tracking heap allocations and peak memory usage using `tracemalloc`, `memory_profiler`, or `resource`. Realistic benchmarking accounts for warm-up effects, cache state, input data distribution, garbage collection, and the distinction between CPU-bound and memory-bound workloads. Statistical rigor requires multiple repetitions, reporting of min/median/mean, and consideration of variance. For NumPy specifically, benchmarking must consider dtype, contiguity, broadcasting, and SIMD utilization.

**Beginner-Friendly Explanation:** Benchmarking is how you measure whether your code is fast or slow. Instead of guessing, you run your code many times, measure how long it takes, and use special tools to see which lines are slow and how much memory it uses. This helps you find and fix the parts that actually matter.

### Key Characteristics

- **Repetition:** Single measurements are noisy; always repeat and report statistics.
- **Warm-Up:** First runs are slower due to imports, JIT, and cache warming.
- **Realistic Inputs:** Benchmark with data that resembles production workloads.
- **Multiple Metrics:** Time, memory, cache misses, and CPU cycles tell different stories.
- **Line-Level Granularity:** Function-level profiling may miss hot lines inside large functions.
- **Memory Tracking:** Peak heap allocation matters as much as execution time for large arrays.

### Prerequisites

- Basic Python and NumPy array manipulation
- Familiarity with the concept of algorithmic complexity
- Understanding of the memory model (contiguous arrays, views, copies)
- Basic knowledge of CPU caches and memory hierarchy

### Related Programming Areas

- High-Performance Computing (HPC)
- Machine Learning (training pipeline optimization)
- Data Science (ETL and analytics)
- Scientific Computing (numerical solvers)
- Web/Backend (performance-critical Python code)
- DevOps (performance regression testing)

### Core Concepts / Features

1. `timeit`
2. Profiling Concepts
3. Measuring Realistic Workloads
4. Memory-Specific Profiling
5. Line-by-Line CPU Profiling

---

## Core Concept 1: `timeit`

### Definitions

**Core Definition:** `timeit` is Python's standard library module for measuring the execution time of small code snippets. It runs the snippet multiple times and reports the best (or total) time, minimizing the influence of background noise and system variability.

**Technical Definition:** `timeit.timeit(stmt='pass', setup='pass', timer=<default timer>, number=1000000, globals=None)` executes `stmt` `number` times and returns the total elapsed time in seconds. The `setup` string is executed once before timing begins. The timer defaults to `time.perf_counter` (high-resolution monotonic clock). `timeit.repeat(stmt, setup, repeat=5, number=1000000)` returns a list of `repeat` measurements, enabling statistical analysis. The IPython magic `%timeit` automatically selects `number` and `repeat` based on the snippet's estimated runtime. For NumPy benchmarks, `%timeit` is preferred for interactive use, while `timeit.repeat` is preferred for scripted benchmarks.

**Beginner-Friendly Explanation:** `timeit` runs your code many times and tells you how long it took. It handles the repetition and noise-averaging for you, so you don't have to write your own timing loops. Just pass it a code snippet as a string (or use the `%timeit` magic in Jupyter) and it reports the average time per run.

### Purposes

- To measure the execution time of small code snippets with statistical rigor.
- To compare alternative implementations of the same operation.
- To detect performance regressions in code changes.
- To validate that an optimization actually improves performance.
- To choose between NumPy functions (e.g., `np.sum` vs. `np.add.reduce`).

### Syntax Rules and Structure

#### Complete General Syntax

```python
import timeit

# Basic timing
t = timeit.timeit(stmt='sum(range(100))', number=10000)

# With setup
t = timeit.timeit(
    stmt='np.sum(a)',
    setup='import numpy as np; a = np.arange(1000)',
    number=10000,
)

# Multiple repetitions
times = timeit.repeat(
    stmt='np.sum(a)',
    setup='import numpy as np; a = np.arange(1000)',
    repeat=5,
    number=10000,
)
print(f"Best: {min(times)/10000*1e6:.2f} µs")
print(f"Mean: {sum(times)/len(times)/10000*1e6:.2f} µs")

# IPython magic (interactive)
# %timeit np.sum(a)
# %timeit -n 1000 -r 5 np.sum(a)
```

**Component Breakdown:**
- `stmt`: Code snippet to time (string or callable).
- `setup`: Code executed once before timing (imports, array creation).
- `timer`: Timer function (default `time.perf_counter`).
- `number`: Number of executions per measurement.
- `repeat`: Number of measurements (returns a list).
- `globals`: Namespace for `stmt` (default: caller's globals).

**Syntax Rules:**
- Use `setup` to avoid including array creation in the timed section.
- Use `number` large enough that the total time is >0.1 s for stability.
- Use `repeat≥3` to assess variance.
- Report `min(times)` as the "best-case" estimate (least noise).
- For callables, pass `stmt` as a lambda or bound method.

**Constraints and Limitations:**
- `timeit` includes the overhead of the `stmt` call itself (negligible for large operations).
- String-based `stmt` prevents access to local variables unless passed via `globals`.
- `timeit` disables garbage collection by default during timing (via `gc.disable()` in `Timer`), which may differ from production behavior.
- Does not measure memory usage.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic `timeit` Usage

```python
import timeit
import numpy as np

# Step 1: Simple Python snippet
t1 = timeit.timeit(stmt='sum(range(1000))', number=10_000)
print(f"Python sum(range(1000)): {t1/10_000*1e6:.2f} µs per call")

# Step 2: NumPy snippet with setup
t2 = timeit.timeit(
    stmt='np.sum(a)',
    setup='import numpy as np; a = np.arange(1000)',
    number=10_000,
)
print(f"NumPy np.sum(a):         {t2/10_000*1e6:.2f} µs per call")

# Step 3: Using a callable instead of a string
def numpy_sum():
    return np.sum(np.arange(1000))

t3 = timeit.timeit(numpy_sum, number=10_000)
print(f"Callable form:           {t3/10_000*1e6:.2f} µs per call")

# Step 4: Multiple repetitions
times = timeit.repeat(
    stmt='np.sum(a)',
    setup='import numpy as np; a = np.arange(1000)',
    repeat=5,
    number=10_000,
)
print(f"\nrepeat=5 measurements (µs per call):")
for i, t in enumerate(times):
    print(f"  Run {i+1}: {t/10_000*1e6:.2f}")
print(f"  Min:  {min(times)/10_000*1e6:.2f}")
print(f"  Mean: {np.mean(times)/10_000*1e6:.2f}")
print(f"  Std:  {np.std(times)/10_000*1e6:.2f}")
```

**Expected Output:**
```
Python sum(range(1000)): 5.67 µs per call
NumPy np.sum(a):         1.23 µs per call
Callable form:           1.21 µs per call

repeat=5 measurements (µs per call):
  Run 1: 1.24
  Run 2: 1.21
  Run 3: 1.22
  Run 4: 1.23
  Run 5: 1.21
  Min:  1.21
  Mean: 1.22
  Std:  0.01
```

**Why This Output Occurs:** `timeit` runs each snippet the specified number of times and divides the total by `number`. NumPy's `sum` is ~4.6× faster than Python's built-in `sum(range(1000))` for this small array due to the absence of per-element Python overhead. The low standard deviation across repetitions indicates stable timing.

#### Example 2: Comparing Alternative Implementations

```python
import timeit
import numpy as np

# Step 1: Setup for the benchmark
setup = '''
import numpy as np
a = np.random.default_rng(42).random(100_000)
b = np.random.default_rng(43).random(100_000)
'''

# Step 2: Compare different addition implementations
implementations = {
    'a + b': 'a + b',
    'np.add(a, b)': 'np.add(a, b)',
    'np.add(a, b, out=c)': 'np.add(a, b, out=c)',
    'a += b (copy)': 'a_copy = a.copy(); a_copy += b',
}

# Pre-allocate output for out= variant
setup_full = setup + '\nc = np.empty_like(a)\n'

print(f"{'Implementation':<25} {'Time (ms)':>12} {'Relative':>10}")
print("-" * 50)
baseline = None
for name, stmt in implementations.items():
    # Use different setup for out= variant to avoid copy overhead
    s = setup_full if 'out=' in stmt else setup
    times = timeit.repeat(stmt=stmt, setup=s, repeat=5, number=1000)
    t = min(times) / 1000 * 1000  # ms per call
    if baseline is None:
        baseline = t
    print(f"{name:<25} {t:>12.4f} {baseline/t:>9.2f}×")

# Step 3: Compare reduction operations
print(f"\nReduction operations (100k elements):")
reductions = {
    'np.sum(a)': 'np.sum(a)',
    'np.add.reduce(a)': 'np.add.reduce(a)',
    'a.sum()': 'a.sum()',
    'np.einsum("i->", a)': 'np.einsum("i->", a)',
}
for name, stmt in reductions.items():
    times = timeit.repeat(stmt=stmt, setup=setup, repeat=5, number=1000)
    t = min(times) / 1000 * 1000
    print(f"  {name:<25} {t:>8.4f} ms")
```

**Expected Output:**
```
Implementation               Time (ms)   Relative
--------------------------------------------------
a + b                          0.1234      1.00×
np.add(a, b)                   0.1221      1.01×
np.add(a, b, out=c)            0.0912      1.35×
a += b (copy)                  0.2134      0.58×

Reduction operations (100k elements):
  np.sum(a)                   0.0456 ms
  np.add.reduce(a)            0.0443 ms
  a.sum()                     0.0451 ms
  np.einsum("i->", a)         0.0678 ms
```

**Why This Output Occurs:** `np.add(a, b, out=c)` is 1.35× faster than `a + b` because it avoids allocating a new output array. The `a += b (copy)` variant is slower because it copies `a` before the in-place operation. All reduction variants perform similarly, with `einsum` slightly slower due to its generality.

### Real-World Cases

- **Library Development:** Benchmarking alternative implementations before committing to one.
- **Code Review:** Detecting performance regressions when changing algorithms.
- **Machine Learning:** Choosing between NumPy, PyTorch, and JAX for tensor operations.
- **Data Science:** Comparing pandas operations for ETL pipelines.

---

## Core Concept 2: Profiling Concepts

### Definitions

**Core Definition:** Profiling is the process of measuring where a program spends its time and resources, attributing costs to functions, lines, or call stacks. It answers "what is slow?" rather than "how slow is it?".

**Technical Definition:** Profiling tools fall into two categories: **deterministic profilers** (e.g., `cProfile`) instrument every function call and return, recording call counts and cumulative/inclusive times; **statistical profilers** (e.g., `py-spy`, `perf`) sample the call stack at fixed intervals, providing lower-overhead but approximate profiles. `cProfile` is Python's built-in deterministic profiler; it records `ncalls` (number of calls), `tottime` (time in the function excluding sub-calls), `cumtime` (cumulative time including sub-calls), and `percall` statistics. `pstats` provides tools to sort, filter, and display the results. For NumPy code, `cProfile` shows the NumPy function calls but not the internal C loops; for those, line-level or native profilers are needed.

**Beginner-Friendly Explanation:** Profiling is like using a stopwatch on every function in your program. It tells you which functions are called most often and which ones take the most time. Instead of guessing which part of your code is slow, you get a ranked list. This is essential for optimizing large programs where the bottleneck may be in an unexpected place.

### Purposes

- To identify the functions that consume the most execution time.
- To distinguish between time spent in Python code vs. C extensions.
- To find functions called excessively (high `ncalls`).
- To understand the call graph and cumulative time distribution.
- To prioritize optimization efforts on the actual bottlenecks.

### Syntax Rules and Structure

#### Complete General Syntax

```python
import cProfile
import pstats
import io

# Basic profiling
cProfile.run('my_function()')

# Profiling with stats object
profiler = cProfile.Profile()
profiler.enable()
my_function()
profiler.disable()
stats = pstats.Stats(profiler)
stats.sort_stats('cumulative').print_stats(10)

# Saving and loading profiles
profiler.dump_stats('profile.prof')
stats = pstats.Stats('profile.prof')

# Line-by-line profiling (requires line_profiler)
# kernprof -l -v script.py
# @profile decorator
```

**Component Breakdown:**
- `cProfile.run(stmt)`: Profiles a string statement.
- `cProfile.Profile()`: Creates a profiler object for programmatic use.
- `pstats.Stats(profiler)`: Wraps the profiler output for analysis.
- `sort_stats('cumulative'|'tottime'|'calls')`: Sorts results by different metrics.
- `print_stats(N)`: Prints the top `N` entries.
- `dump_stats(filename)`: Saves the profile to disk.

**Syntax Rules:**
- `tottime` excludes time in sub-calls; `cumtime` includes it.
- `ncalls` shows the number of calls; for recursive functions, it shows `total/primitive`.
- `percall` is `tottime/ncalls` or `cumtime/ncalls` depending on the sort.
- `cProfile` adds overhead (2–5× slowdown); `py-spy` has near-zero overhead.

**Constraints and Limitations:**
- `cProfile` does not show time inside C extensions (e.g., NumPy's internal loops).
- Overhead can distort profiles of short-running code.
- Statistical profilers may miss short-lived functions.
- Profiling does not measure memory usage.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic `cProfile` Usage

```python
import cProfile
import pstats
import numpy as np

# Step 1: Define functions to profile
def slow_python_loop(n):
    total = 0
    for i in range(n):
        total += i ** 2
    return total

def fast_numpy(n):
    return np.sum(np.arange(n) ** 2)

def mixed_workload(n):
    a = np.random.default_rng(42).random(n)
    b = np.random.default_rng(43).random(n)
    c = a + b                    # Vectorized
    d = slow_python_loop(1000)   # Slow Python
    e = fast_numpy(n)            # Vectorized
    return c.sum() + d + e

# Step 2: Profile the mixed workload
profiler = cProfile.Profile()
profiler.enable()
result = mixed_workload(100_000)
profiler.disable()

# Step 3: Print sorted stats
stats = pstats.Stats(profiler)
stats.sort_stats('cumulative')
print("=== Top 10 by cumulative time ===")
stats.print_stats(10)

# Step 4: Print sorted by tottime
stats.sort_stats('tottime')
print("\n=== Top 5 by tottime (excluding sub-calls) ===")
stats.print_stats(5)

# Step 5: Save and reload profile
profiler.dump_stats('mixed_profile.prof')
loaded_stats = pstats.Stats('mixed_profile.prof')
print(f"\nProfile saved and reloaded successfully.")
```

**Expected Output:**
```
=== Top 10 by cumulative time ===
         1234 function calls in 0.052 seconds

   Ordered by: cumulative time

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.001    0.001    0.052    0.052 <stdin>:1(mixed_workload)
        1    0.030    0.030    0.030    0.030 <stdin>:1(slow_python_loop)
        2    0.010    0.005    0.010    0.005 {method 'random' of numpy.random._generator.Generator}
        1    0.005    0.005    0.005    0.005 <stdin>:1(fast_numpy)
        1    0.002    0.002    0.002    0.002 {method 'sum' of numpy.ndarray}
        1    0.001    0.001    0.001    0.001 {built-in method numpy.arange}
        ...

=== Top 5 by tottime (excluding sub-calls) ===
   Ordered by: internal time

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.030    0.030    0.030    0.030 <stdin>:1(slow_python_loop)
        2    0.010    0.005    0.010    0.005 {method 'random' of ...}
        1    0.005    0.005    0.005    0.005 <stdin>:1(fast_numpy)
        ...

Profile saved and reloaded successfully.
```

**Why This Output Occurs:** `cProfile` records every function call and its cumulative time. The `slow_python_loop` dominates with 0.030 s of tottime (pure Python loop overhead). NumPy operations (`random`, `sum`) appear as C-extension calls with small cumulative times. The `fast_numpy` function is faster than the Python loop despite doing the same work, confirming the vectorization advantage. `cProfile` does not show the internal C loops inside NumPy functions.

#### Example 2: Profiling with `pstats` Filtering

```python
import cProfile
import pstats
import numpy as np

# Step 1: Define a realistic workload
def data_pipeline(n):
    rng = np.random.default_rng(42)
    data = rng.standard_normal((n, 10))
    means = data.mean(axis=0)
    stds = data.std(axis=0)
    normalized = (data - means) / stds
    return normalized.sum()

# Step 2: Profile
profiler = cProfile.Profile()
profiler.enable()
data_pipeline(1_000_000)
profiler.disable()

# Step 3: Filter and print
stats = pstats.Stats(profiler)
stats.strip_dirs()                     # Remove directory names
stats.sort_stats('cumulative')

# Filter to NumPy functions only
print("=== NumPy functions ===")
stats.print_stats('numpy', 10)

# Filter to functions with "mean" or "std"
print("\n=== Functions with 'mean' or 'std' ===")
stats.print_stats('mean|std', 10)

# Print callers of a specific function
print("\n=== Callers of np.mean ===")
stats.print_callers('mean')

# Print callees of data_pipeline
print("\n=== Callees of data_pipeline ===")
stats.print_callees('data_pipeline')
```

**Expected Output:**
```
=== NumPy functions ===
   Ordered by: cumulative time
   List reduced from 15 to 10 due to restriction <'numpy'>

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.045    0.045    0.045    0.045 {method 'mean' of numpy.ndarray}
        1    0.032    0.032    0.032    0.032 {method 'std' of numpy.ndarray}
        1    0.018    0.018    0.018    0.018 {method 'sum' of numpy.ndarray}
        ...

=== Functions with 'mean' or 'std' ===
   ...

=== Callers of np.mean ===
   Ordered by: cumulative time

   Function                           was called by...
     {method 'mean' of numpy.ndarray}  <- data_pipeline at <stdin>:1

=== Callees of data_pipeline ===
   Ordered by: cumulative time

   Function called...
   data_pipeline at <stdin>:1
     -> {method 'mean' of numpy.ndarray}
     -> {method 'std' of numpy.ndarray}
     -> {method 'sum' of numpy.ndarray}
```

**Why This Output Occurs:** `pstats` provides filtering and call-graph analysis. The `strip_dirs` method removes directory prefixes for readability. `print_stats('numpy')` filters to functions with "numpy" in their name. `print_callers` shows who called a function, and `print_callees` shows what a function called. This helps identify the bottleneck in a pipeline: `np.mean` and `np.std` dominate, so optimizing them would have the greatest impact.

### Real-World Cases

- **ML Pipelines:** Profiling data loading, preprocessing, and model training to find bottlenecks.
- **Web Applications:** Identifying slow database queries or serialization functions.
- **Scientific Computing:** Profiling numerical solvers to identify expensive operations.
- **Library Development:** Ensuring that public API functions have acceptable performance.

---

## Core Concept 3: Measuring Realistic Workloads

### Definitions

**Core Definition:** Measuring realistic workloads means benchmarking with input data, access patterns, and environmental conditions that resemble production use, rather than artificial micro-benchmarks that may not reflect actual performance.

**Technical Definition:** Realistic benchmarking accounts for: (1) **input size and distribution** — array sizes, dtypes, sparsity, and value ranges that match production; (2) **access patterns** — row-wise vs. column-wise, contiguous vs. strided, sequential vs. random; (3) **environmental factors** — CPU frequency scaling, thermal throttling, background processes, and cache state; (4) **warm-up effects** — first-run overhead from imports, JIT compilation, and cache warming; (5) **concurrency** — single-threaded vs. multi-threaded workloads; and (6) **total pipeline time** — including I/O, preprocessing, and postprocessing, not just the kernel. Amdahl's Law states that optimizing a kernel that accounts for 20% of total time yields at most a 20% speedup regardless of how fast the kernel becomes.

**Beginner-Friendly Explanation:** A realistic workload is one that looks like what your code will actually do in production. If your production data is a million small arrays, don't benchmark with one huge array. If your data is stored column-wise, don't benchmark with row-wise access. If your pipeline spends 80% of its time loading data, optimizing the compute kernel won't help much.

### Purposes

- To ensure that benchmark results translate to production performance.
- To avoid optimizing code that isn't the actual bottleneck.
- To detect performance issues that only appear with realistic data.
- To set accurate performance expectations.
- To validate that optimizations improve end-to-end performance.

### Syntax Rules and Structure

#### Complete General Syntax

```python
import numpy as np
import time

def benchmark_realistic(data_generator, operation, n_reps=5):
    """Benchmark an operation on realistic data."""
    # Warm-up
    data = data_generator()
    operation(data)

    times = []
    for _ in range(n_reps):
        data = data_generator()  # Fresh data each time
        start = time.perf_counter()
        operation(data)
        times.append(time.perf_counter() - start)
    return min(times), np.mean(times), np.std(times)

# Realistic data generators
def small_arrays():
    return [np.random.default_rng(i).random(100) for i in range(1000)]

def large_array():
    return np.random.default_rng(42).random(10_000_000)

def column_major():
    return np.asfortranarray(np.random.default_rng(42).random((1000, 1000)))

# Realistic operations
def process_small(arrs):
    return sum(a.sum() for a in arrs)

def process_large(arr):
    return (arr ** 2).sum()

def process_column(arr):
    return arr.sum(axis=0)
```

**Component Breakdown:**
- `data_generator`: A function that produces realistic input data.
- `operation`: The function to benchmark.
- `n_reps`: Number of repetitions for statistical stability.
- Warm-up run: Executes once before timing to trigger JIT/cache effects.

**Syntax Rules:**
- Generate fresh data each repetition to avoid cache effects.
- Use data sizes and dtypes that match production.
- Include I/O and preprocessing in the timed section if they are part of the pipeline.
- Report both min and mean; min represents best-case, mean represents typical.

**Constraints and Limitations:**
- Synthetic data may not capture all production characteristics (e.g., real-world value distributions).
- Benchmarking on a development machine may not reflect production hardware.
- Amdahl's Law limits end-to-end speedup to the fraction of time spent in the optimized component.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Realistic vs. Unrealistic Benchmarking

```python
import numpy as np
import time

# Step 1: Unrealistic benchmark — one huge array
def unrealistic_benchmark():
    a = np.random.default_rng(42).random(10_000_000)
    start = time.perf_counter()
    result = (a ** 2).sum()
    return time.perf_counter() - start

# Step 2: Realistic benchmark — many small arrays (batch processing)
def realistic_benchmark():
    rng = np.random.default_rng(42)
    arrays = [rng.random(100) for _ in range(100_000)]
    start = time.perf_counter()
    total = 0.0
    for a in arrays:
        total += (a ** 2).sum()
    return time.perf_counter() - start

# Step 3: Run both
t_unrealistic = unrealistic_benchmark()
t_realistic = realistic_benchmark()

print(f"Unrealistic (1 array of 10M): {t_unrealistic*1000:.2f} ms")
print(f"Realistic (100k arrays of 100): {t_realistic*1000:.2f} ms")

# Step 4: Vectorized version of realistic workload
def vectorized_realistic():
    rng = np.random.default_rng(42)
    batch = rng.random((100_000, 100))   # Stack all arrays
    start = time.perf_counter()
    result = np.sum(batch ** 2)
    return time.perf_counter() - start

t_vectorized = vectorized_realistic()
print(f"Vectorized (100k × 100): {t_vectorized*1000:.2f} ms")
print(f"Speedup vs. loop: {t_realistic/t_vectorized:.1f}×")

# Step 5: Demonstrate Amdahl's Law
def pipeline_with_io():
    """Simulate a pipeline where I/O dominates."""
    # I/O simulation (sleep)
    time.sleep(0.05)
    # Compute
    a = np.random.default_rng(42).random(1_000_000)
    return (a ** 2).sum()

# Full pipeline
start = time.perf_counter()
pipeline_with_io()
t_full = time.perf_counter() - start

# Compute only
a = np.random.default_rng(42).random(1_000_000)
start = time.perf_counter()
_ = (a ** 2).sum()
t_compute = time.perf_counter() - start

print(f"\nAmdahl's Law demonstration:")
print(f"  Full pipeline:  {t_full*1000:.2f} ms")
print(f"  Compute only:   {t_compute*1000:.2f} ms")
print(f"  I/O fraction:   {(t_full - t_compute)/t_full*100:.1f}%")
print(f"  Max speedup from optimizing compute: "
      f"{t_full/t_compute:.2f}× (even if compute became 0)")
```

**Expected Output:**
```
Unrealistic (1 array of 10M): 45.67 ms
Realistic (100k arrays of 100): 234.56 ms
Vectorized (100k × 100): 12.34 ms
Speedup vs. loop: 19.0×

Amdahl's Law demonstration:
  Full pipeline:  52.34 ms
  Compute only:   2.34 ms
  I/O fraction:   95.5%
  Max speedup from optimizing compute: 22.4× (even if compute became 0)
```

**Why This Output Occurs:** The unrealistic benchmark uses one huge array (cache-friendly), while the realistic benchmark processes 100,000 small arrays (loop overhead dominates). Vectorizing the realistic workload (stacking into a 2-D array) yields a 19× speedup. Amdahl's Law shows that I/O accounts for 95.5% of the pipeline time, so even making the compute infinitely fast yields at most a 22.4× speedup — the real bottleneck is I/O.

#### Example 2: Benchmarking with Warm-Up and Cache Effects

```python
import numpy as np
import time

# Step 1: Create a large array
n = 50_000_000
a = np.random.default_rng(42).random(n)

# Step 2: Cold vs. warm benchmark
print("Cold vs. warm cache:")

# Cold: first run (data not in cache)
start = time.perf_counter()
result = a.sum()
t_cold = time.perf_counter() - start
print(f"  Cold (first run):  {t_cold*1000:.2f} ms")

# Warm: subsequent runs (data in cache if it fits)
times = []
for _ in range(5):
    start = time.perf_counter()
    result = a.sum()
    times.append(time.perf_counter() - start)
print(f"  Warm (subsequent): {min(times)*1000:.2f} ms")
print(f"  Cold/warm ratio:   {t_cold/min(times):.2f}×")

# Step 3: Cache size effect
print("\nCache size effect:")
for size in [1_000, 10_000, 100_000, 1_000_000, 10_000_000, 50_000_000]:
    arr = np.random.default_rng(42).random(size)
    arr.sum()  # warm-up
    times = []
    for _ in range(3):
        start = time.perf_counter()
        arr.sum()
        times.append(time.perf_counter() - start)
    t = min(times)
    per_element = t / size * 1e9
    print(f"  size={size:>12,}: {t*1000:>8.3f} ms ({per_element:.3f} ns/element)")

# Step 4: Data-dependent performance
print("\nData-dependent performance:")
# Sorted vs. unsorted data (branch prediction)
sorted_data = np.sort(a)
unsorted_data = a.copy()

for name, data in [('Sorted', sorted_data), ('Unsorted', unsorted_data)]:
    times = []
    for _ in range(3):
        start = time.perf_counter()
        _ = data.sum()
        times.append(time.perf_counter() - start)
    print(f"  {name}: {min(times)*1000:.2f} ms")
```

**Expected Output:**
```
Cold vs. warm cache:
  Cold (first run):  123.45 ms
  Warm (subsequent): 89.12 ms
  Cold/warm ratio:   1.39×

Cache size effect:
  size=       1,000:    0.001 ms (1.234 ns/element)
  size=      10,000:    0.008 ms (0.789 ns/element)
  size=     100,000:    0.067 ms (0.671 ns/element)
  size=   1,000,000:    0.678 ms (0.678 ns/element)
  size=  10,000,000:   12.345 ms (1.234 ns/element)
  size=  50,000,000:   89.123 ms (1.782 ns/element)

Data-dependent performance:
  Sorted:   85.67 ms
  Unsorted: 89.12 ms
```

**Why This Output Occurs:** The cold run is 1.39× slower than warm runs because the data isn't in cache and must be fetched from RAM. As array size increases beyond L2/L3 cache (typically 1–32 MB), per-element time increases because more data must come from RAM. Sorted data is slightly faster due to branch prediction in the summation loop.

### Real-World Cases

- **ML Training:** Benchmarking with realistic batch sizes and data distributions.
- **Data Pipelines:** Measuring end-to-end pipeline time including I/O and preprocessing.
- **Web Services:** Benchmarking with production-like request patterns and concurrency.
- **Scientific Simulation:** Benchmarking with realistic grid sizes and boundary conditions.

---

## Core Concept 4: Memory-Specific Profiling

### Definitions

**Core Definition:** Memory profiling tracks heap allocations, peak memory usage, and object lifetimes during program execution. It answers "how much memory does this use?" and "where are the allocations happening?".

**Technical Definition:** Python's `tracemalloc` module traces memory allocations by hooking into the CPython allocator. `tracemalloc.start()` begins tracing; `tracemalloc.get_traced_memory()` returns `(current, peak)` in bytes; `tracemalloc.take_snapshot()` captures the current allocation state as a `Snapshot` object; `Snapshot.compare_to(other)` computes differences between snapshots; `Snapshot.statistics('lineno')` aggregates allocations by source line. The `memory_profiler` package provides line-by-line memory profiling via the `@profile` decorator and the `mprof` command-line tool. For NumPy specifically, `tracemalloc` tracks Python-level allocations (including `np.array`), but not allocations made by C extensions outside the Python allocator (e.g., some BLAS buffers). The `resource.getrusage()` function reports peak RSS (resident set size) from the OS.

**Beginner-Friendly Explanation:** Memory profiling tells you how much RAM your program uses and where the allocations happen. `tracemalloc` is built into Python and shows you the lines that allocate the most memory. `memory_profiler` gives you a line-by-line breakdown. This is crucial for large NumPy operations where a single array can consume gigabytes.

### Purposes

- To measure peak memory usage during large matrix operations.
- To identify lines or functions that allocate the most memory.
- To detect memory leaks (memory that grows without bound).
- To optimize memory usage by finding unnecessary copies.
- To validate that memory optimizations (in-place operations, chunking) work as intended.

### Syntax Rules and Structure

#### Complete General Syntax

```python
import tracemalloc
import numpy as np

# Basic tracemalloc
tracemalloc.start()
# ... code to profile ...
current, peak = tracemalloc.get_traced_memory()
tracemalloc.stop()
print(f"Current: {current / 1e6:.2f} MB")
print(f"Peak:    {peak / 1e6:.2f} MB")

# Snapshot comparison
tracemalloc.start()
snap1 = tracemalloc.take_snapshot()
# ... code ...
snap2 = tracemalloc.take_snapshot()
top_stats = snap2.compare_to(snap1, 'lineno')
for stat in top_stats[:10]:
    print(stat)

# memory_profiler (requires: pip install memory_profiler)
# @profile
# def my_function():
#     ...
# Run with: python -m memory_profiler script.py

# Line-by-line via mprof
# mprof run script.py
# mprof plot
```

**Component Breakdown:**
- `tracemalloc.start()`: Begins tracing memory allocations.
- `tracemalloc.get_traced_memory()`: Returns `(current, peak)` in bytes.
- `tracemalloc.take_snapshot()`: Captures the current allocation state.
- `Snapshot.compare_to(other, 'lineno')`: Compares two snapshots by source line.
- `Snapshot.statistics('lineno')`: Aggregates allocations by source line.
- `@profile`: Decorator for `memory_profiler` line-by-line profiling.

**Syntax Rules:**
- `tracemalloc` adds overhead (~2–3×); use it for profiling, not production.
- `tracemalloc` tracks Python-level allocations, not all C-level allocations.
- Use `tracemalloc.reset_peak()` to reset the peak measurement between phases.
- `memory_profiler` requires running the script with `python -m memory_profiler`.

**Constraints and Limitations:**
- `tracemalloc` does not track memory allocated by C extensions outside the Python allocator.
- `memory_profiler` adds significant overhead (10–100×) and may distort timing.
- Peak RSS from `resource.getrusage()` includes memory from all sources, not just Python.
- Snapshot comparison requires the same tracing session.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: `tracemalloc` for Matrix Operations

```python
import tracemalloc
import numpy as np
import time

# Step 1: Start tracing
tracemalloc.start()

# Step 2: Record initial state
snap1 = tracemalloc.take_snapshot()

# Step 3: Perform a large matrix operation
n = 2000
A = np.random.default_rng(42).random((n, n))
B = np.random.default_rng(43).random((n, n))

# Chained operation (creates intermediates)
C = A @ B + A.T @ B - A @ B.T

# Step 4: Record final state
snap2 = tracemalloc.take_snapshot()
current, peak = tracemalloc.get_traced_memory()
tracemalloc.stop()

print(f"Matrix size: {n}×{n} ({A.nbytes / 1e6:.2f} MB per matrix)")
print(f"\nMemory usage:")
print(f"  Current: {current / 1e6:.2f} MB")
print(f"  Peak:    {peak / 1e6:.2f} MB")

# Step 5: Compare snapshots
top_stats = snap2.compare_to(snap1, 'lineno')
print(f"\nTop 5 allocation differences:")
for stat in top_stats[:5]:
    print(f"  {stat}")

# Step 6: Statistics by line
print(f"\nAllocation statistics:")
for stat in snap2.statistics('lineno')[:5]:
    print(f"  {stat}")

# Step 7: Compare with in-place version
tracemalloc.start()
snap3 = tracemalloc.take_snapshot()

A2 = np.random.default_rng(42).random((n, n))
B2 = np.random.default_rng(43).random((n, n))
C2 = np.empty_like(A2)
np.matmul(A2, B2, out=C2)
C2 += A2.T @ B2
C2 -= A2 @ B2.T

snap4 = tracemalloc.take_snapshot()
current2, peak2 = tracemalloc.get_traced_memory()
tracemalloc.stop()

print(f"\nIn-place version:")
print(f"  Peak: {peak2 / 1e6:.2f} MB")
print(f"  Savings: {(1 - peak2/peak) * 100:.1f}%")
```

**Expected Output:**
```
Matrix size: 2000×2000 (32.00 MB per matrix)

Memory usage:
  Current: 32.00 MB
  Peak:    192.00 MB

Top 5 allocation differences:
  <stdin>:11: 64.0 MB (+64.0 MB)
  <stdin>:10: 32.0 MB (+32.0 MB)
  ...

Allocation statistics:
  <stdin>:11: 64.0 MB
  <stdin>:10: 32.0 MB
  ...

In-place version:
  Peak: 128.00 MB
  Savings: 33.3%
```

**Why This Output Occurs:** The chained operation `A @ B + A.T @ B - A @ B.T` creates multiple intermediate matrices (each 32 MB), peaking at 192 MB. The in-place version reuses the pre-allocated `C2` buffer, peaking at 128 MB — a 33.3% reduction. `tracemalloc` attributes the allocations to specific source lines, making it easy to identify which operations allocate the most memory.

#### Example 2: `memory_profiler` Line-by-Line

```python
# File: memory_profile_example.py
# Run with: python -m memory_profiler memory_profile_example.py

import numpy as np
from memory_profiler import profile

@profile
def process_data(n):
    # Line 1: Allocate large array
    a = np.random.default_rng(42).random(n)

    # Line 2: Create a copy (doubles memory)
    b = a.copy()

    # Line 3: In-place operation (no new allocation)
    b += 1.0

    # Line 4: Chained operation (creates intermediate)
    c = (a + b) * 2.0

    # Line 5: In-place alternative
    d = a.copy()
    d += b
    d *= 2.0

    return c.sum() + d.sum()

if __name__ == '__main__':
    result = process_data(10_000_000)
    print(f"Result: {result:.2f}")

# Expected output (memory_profiler format):
# Line #    Mem usage    Increment  Occurrences   Line Contents
# =============================================================
#      6    50.0 MiB    50.0 MiB           1   def process_data(n):
#      8   126.3 MiB    76.3 MiB           1       a = np.random...
#     11   202.6 MiB    76.3 MiB           1       b = a.copy()
#     14   202.6 MiB     0.0 MiB           1       b += 1.0
#     17   355.2 MiB   152.6 MiB           1       c = (a + b) * 2.0
#     20   431.5 MiB    76.3 MiB           1       d = a.copy()
#     21   431.5 MiB     0.0 MiB           1       d += b
#     22   431.5 MiB     0.0 MiB           1       d *= 2.0
```

**Expected Output (from `memory_profiler`):**
```
Line #    Mem usage    Increment  Occurrences   Line Contents
=============================================================
     6    50.0 MiB    50.0 MiB           1   def process_data(n):
     8   126.3 MiB    76.3 MiB           1       a = np.random.default_rng(42).random(n)
    11   202.6 MiB    76.3 MiB           1       b = a.copy()
    14   202.6 MiB     0.0 MiB           1       b += 1.0
    17   355.2 MiB   152.6 MiB           1       c = (a + b) * 2.0
    20   431.5 MiB    76.3 MiB           1       d = a.copy()
    21   431.5 MiB     0.0 MiB           1       d += b
    22   431.5 MiB     0.0 MiB           1       d *= 2.0

Result: 40000000.00
```

**Why This Output Occurs:** `memory_profiler` shows the memory increment for each line. Creating `a` allocates 76.3 MB (10M `float64` = 80 MB). Copying to `b` allocates another 76.3 MB. The in-place `b += 1.0` allocates 0 MB. The chained `c = (a + b) * 2.0` allocates 152.6 MB (two intermediates: `a + b` and `* 2.0`). The in-place `d` operations allocate only the initial copy (76.3 MB) and no more.

### Real-World Cases

- **ML Model Training:** Tracking GPU/CPU memory during forward and backward passes.
- **Data Pipelines:** Identifying which transformations allocate the most memory.
- **Scientific Simulation:** Monitoring memory during large time-stepping loops.
- **Web Services:** Detecting memory leaks in long-running processes.

---

## Core Concept 5: Line-by-Line CPU Profiling

### Definitions

**Core Definition:** Line-by-line CPU profiling measures the execution time of individual lines within a function, providing finer granularity than function-level profiling. It identifies the exact lines that consume the most time.

**Technical Definition:** `line_profiler` is a Python package that provides line-by-line profiling via the `@profile` decorator or the `kernprof` command-line tool. It uses a C extension to instrument each line of the decorated function, recording the number of hits (`Hits`), time per hit (`Time`), time per hit (`Per Hit`), and percentage of total time (`% Time`). The output is a table with one row per line of the profiled function. `line_profiler` adds significant overhead (~10–100×) because it records timing for every line execution. It is best used on small, targeted functions after `cProfile` has identified the hot function. The `kernprof -l -v script.py` command runs the script with line profiling enabled and prints the results.

**Beginner-Friendly Explanation:** Line profiling is like putting a stopwatch on every line of your function. It tells you exactly which line is slow — maybe a line that creates a large temporary array, or a line that calls a slow function. This is much more precise than function-level profiling, which only tells you the function is slow but not which part.

### Purposes

- To identify the exact lines that consume the most time within a function.
- To distinguish between time spent in allocation vs. computation.
- To find hidden bottlenecks (e.g., dtype conversions, copies) inside complex functions.
- To validate that a line-level optimization actually helps.
- To understand the performance profile of numerical kernels.

### Syntax Rules and Structure

#### Complete General Syntax

```python
# File: line_profile_example.py
# Run with: kernprof -l -v line_profile_example.py
# Or: python -m line_profiler line_profile_example.py.lprof

import numpy as np

@profile    # Injected by kernprof; no import needed
def my_function(a, b):
    c = a + b                # Line 1
    d = np.sqrt(c)           # Line 2
    e = d ** 2               # Line 3
    return e.sum()           # Line 4

if __name__ == '__main__':
    a = np.random.default_rng(42).random(1_000_000)
    b = np.random.default_rng(43).random(1_000_000)
    result = my_function(a, b)
    print(f"Result: {result}")
```

**Component Breakdown:**
- `@profile`: Decorator injected by `kernprof` to mark functions for line profiling.
- `kernprof -l -v script.py`: Runs the script with line profiling and prints results.
- `-l`: Enables line profiling.
- `-v`: Prints the results to stdout.
- Output columns: `Line #`, `Hits`, `Time`, `Per Hit`, `% Time`, `Line Contents`.

**Syntax Rules:**
- The `@profile` decorator is only available when running under `kernprof`; otherwise, define a no-op `profile` function.
- Profiled functions should be small (10–50 lines) for readable output.
- Use `kernprof -l script.py` to generate a `.lprof` file, then `python -m line_profiler script.py.lprof` to view.
- Line profiling adds substantial overhead; do not use in production.

**Constraints and Limitations:**
- High overhead (10–100×) makes it unsuitable for long-running functions.
- Only works on pure Python functions; not on C extensions or built-in functions.
- NumPy operations appear as single lines; the internal C loop is not profiled.
- Requires the `line_profiler` package (`pip install line_profiler`).

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Line Profiling a NumPy Function

```python
# File: line_profile_numpy.py
# Run: kernprof -l -v line_profile_numpy.py

import numpy as np

@profile
def process_matrix(n):
    # Line 1: Allocate random matrix
    A = np.random.default_rng(42).random((n, n))

    # Line 2: Compute row means
    row_means = A.mean(axis=1)

    # Line 3: Compute column means
    col_means = A.mean(axis=0)

    # Line 4: Center the matrix (row-wise)
    A_centered = A - row_means[:, np.newaxis]

    # Line 5: Compute covariance
    cov = A_centered.T @ A_centered / (n - 1)

    # Line 6: Eigendecomposition
    eigenvalues, eigenvectors = np.linalg.eigh(cov)

    # Line 7: Sort eigenvalues
    idx = np.argsort(eigenvalues)[::-1]
    eigenvalues = eigenvalues[idx]
    eigenvectors = eigenvectors[:, idx]

    # Line 8: Project data
    projected = A_centered @ eigenvectors[:, :10]

    return projected.sum()

if __name__ == '__main__':
    result = process_matrix(1000)
    print(f"Result: {result:.2f}")
```

**Expected Output (from `kernprof`):**
```
Wrote profile results to line_profile_numpy.py.lprof
Timer unit: 1e-06 s

Total time: 0.456789 s
File: line_profile_numpy.py
Function: process_matrix at line 5

Line #      Hits         Time  Per Hit   % Time  Line Contents
==============================================================
     5                                           def process_matrix(n):
     7         1     123456.0 123456.0     27.0      A = np.random.default_rng(42).random((n, n))
    10         1      45678.0  45678.0     10.0      row_means = A.mean(axis=1)
    13         1      78901.0  78901.0     17.3      col_means = A.mean(axis=0)
    16         1      34567.0  34567.0      7.6      A_centered = A - row_means[:, np.newaxis]
    19         1      89012.0  89012.0     19.5      cov = A_centered.T @ A_centered / (n - 1)
    22         1      67890.0  67890.0     14.9      eigenvalues, eigenvectors = np.linalg.eigh(cov)
    25         1       1234.0   1234.0      0.3      idx = np.argsort(eigenvalues)[::-1]
    26         1        567.0    567.0      0.1      eigenvalues = eigenvalues[idx]
    27         1        890.0    890.0      0.2      eigenvectors = eigenvectors[:, idx]
    30         1      13456.0  13456.0      2.9      projected = A_centered @ eigenvectors[:, :10]
```

**Why This Output Occurs:** The line profiler shows that `np.random.random((n, n))` takes 27% of total time (allocation + random generation), `cov = A_centered.T @ A_centered` takes 19.5% (matrix multiplication), `col_means = A.mean(axis=0)` takes 17.3% (strided reduction in C-order), and `np.linalg.eigh` takes 14.9% (eigendecomposition). The indexing operations (lines 25–27) are negligible (<0.5%). This tells us that optimizing the random generation or the matrix multiplication would have the greatest impact.

#### Example 2: Identifying Hidden Copies with Line Profiling

```python
# File: line_profile_copies.py
# Run: kernprof -l -v line_profile_copies.py

import numpy as np

@profile
def process_with_copies(a):
    # Line 1: Transpose (view — fast)
    a_T = a.T

    # Line 2: Contiguous copy (slow — copies data)
    a_C = np.ascontiguousarray(a_T)

    # Line 3: Fancy indexing (copy — slow)
    subset = a[[0, 100, 200, 300]]

    # Line 4: Boolean masking (copy — slow)
    masked = a[a > 0.5]

    # Line 5: Strided view (fast)
    strided = a[::10]

    # Line 6: astype conversion (copy — slow)
    a_float32 = a.astype(np.float32)

    # Line 7: In-place operation (fast)
    a_copy = a.copy()
    a_copy += 1.0

    return (a_C.sum() + subset.sum() + masked.sum() +
            strided.sum() + a_float32.sum() + a_copy.sum())

if __name__ == '__main__':
    a = np.random.default_rng(42).random((1000, 1000))
    result = process_with_copies(a)
    print(f"Result: {result:.2f}")
```

**Expected Output (from `kernprof`):**
```
Wrote profile results to line_profile_copies.py.lprof
Timer unit: 1e-06 s

Total time: 0.123456 s
File: line_profile_copies.py
Function: process_with_copies at line 5

Line #      Hits         Time  Per Hit   % Time  Line Contents
==============================================================
     5                                           def process_with_copies(a):
     7         1          5.0      5.0      0.0      a_T = a.T
    10         1      12345.0  12345.0     10.0      a_C = np.ascontiguousarray(a_T)
    13         1       2345.0   2345.0      1.9      subset = a[[0, 100, 200, 300]]
    16         1      45678.0  45678.0     37.0      masked = a[a > 0.5]
    19         1          3.0      3.0      0.0      strided = a[::10]
    22         1      34567.0  34567.0     28.0      a_float32 = a.astype(np.float32)
    25         1       1234.0   1234.0      1.0      a_copy = a.copy()
    26         1        567.0    567.0      0.5      a_copy += 1.0
    28         1      26712.0  26712.0     21.6      return (a_C.sum() + ...)
```

**Why This Output Occurs:** The line profiler clearly distinguishes fast view operations (`a.T`, `a[::10]`, both ~0.0%) from slow copy operations. Boolean masking (`a[a > 0.5]`) takes 37% because it allocates a new array for the mask and another for the result. `astype(np.float32)` takes 28% due to the dtype conversion. `ascontiguousarray` takes 10% due to the copy. This profile immediately reveals that boolean masking and dtype conversion are the bottlenecks, and suggests alternatives (e.g., using `np.where` or processing in chunks).

### Real-World Cases

- **ML Pipelines:** Identifying which preprocessing step (normalization, augmentation, tokenization) is slowest.
- **Numerical Solvers:** Finding which line in a time-stepping loop dominates.
- **Data Engineering:** Detecting hidden copies in ETL transformations.
- **Image Processing:** Profiling color space conversions, resizing, and filtering.

---

## References

1. **Python `timeit` — Official Documentation** — https://docs.python.org/3/library/timeit.html
2. **Python `time.perf_counter` — Official Documentation** — https://docs.python.org/3/library/time.html#time.perf_counter
3. **Python `cProfile` — Official Documentation** — https://docs.python.org/3/library/profile.html
4. **Python `pstats` — Official Documentation** — https://docs.python.org/3/library/profile.html#module-pstats
5. **Python `tracemalloc` — Official Documentation** — https://docs.python.org/3/library/tracemalloc.html
6. **Python `resource` — Official Documentation** — https://docs.python.org/3/library/resource.html
7. **`line_profiler` — PyPI** — https://pypi.org/project/line-profiler/
8. **`memory_profiler` — PyPI** — https://pypi.org/project/memory-profiler/
9. **`py-spy` — GitHub** — https://github.com/benfred/py-spy
10. **NumPy "Performance" — Official Documentation** — https://numpy.org/doc/stable/reference/random/performance.html
11. **NumPy "NumPy Benchmarks" — Official Documentation** — https://numpy.org/benchmarks/
12. **IPython `%timeit` — Official Documentation** — https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-timeit
13. **`pytest-benchmark` — Official Documentation** — https://pytest-benchmark.readthedocs.io/
14. **Brendan Gregg, "Systems Performance" (2nd ed., Pearson, 2020)** — Comprehensive reference for profiling and benchmarking methodology.
15. **Ulrich Drepper, "What Every Programmer Should Know About Memory" (2007)** — https://people.freebsd.org/~lstewart/articles/cpumemory.pdf