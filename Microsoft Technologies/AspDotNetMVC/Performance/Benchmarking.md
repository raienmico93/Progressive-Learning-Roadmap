# ASP.NET Core Benchmarking: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core** (.NET 6–10) and the discipline of **benchmarking** — the practice of measuring the performance of specific methods or code paths with statistical rigour. Benchmarking is distinct from profiling: profiling answers **“where is the time going?”** in a running system; benchmarking answers **“which of these implementations is faster, and by how much?”** in a controlled experiment. The de facto standard tool for .NET microbenchmarking is **BenchmarkDotNet**, an open‑source library adopted by Microsoft, the .NET runtime team, and thousands of projects. A critical framing note: **BenchmarkDotNet works only with console applications** — it does not host ASP.NET Core directly. You benchmark the code *used by* an ASP.NET Core app in a separate console project, then validate system‑level impact with load testing or profiling. Each requested topic is treated independently — no sections are merged.


## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Benchmarking is the practice of measuring the performance of a specific piece of code — a method, an algorithm, or a data structure — under controlled, repeatable conditions to compare alternatives or track regressions. |
| **Technical Definition** | Benchmarking in .NET uses **BenchmarkDotNet** to transform annotated methods (`[Benchmark]`) into statistically rigorous experiments. The library executes benchmarks through **Pilot**, **OverheadWarmup**, **ActualWarmup**, and **ActualWorkload** stages, running multiple iterations and launches to produce **Mean**, **Median**, **StdDev**, **Error**, and **memory diagnostics** (Gen0/Gen1/Gen2 collections and allocated bytes). It isolates each benchmark in a separate process to eliminate JIT and GC cross‑contamination. |
| **Beginner‑Friendly Explanation** | Benchmarking is the **“lab‑grade stopwatch”** for your code. If you want to know whether `StringBuilder` or string concatenation is faster for a specific operation, you do not guess — you write a benchmark. BenchmarkDotNet handles all the hard parts: warming up the JIT compiler, running enough iterations for statistical confidence, measuring allocations, and producing a clean table of results. It exists because naive stopwatch measurements are almost always misleading. |

### Key Characteristics

- **Statistically rigorous** — produces mean, median, standard deviation, and error, not a single number.
- **Process‑isolated** — each benchmark runs in its own process, preventing JIT and GC interference.
- **Allocation‑aware** — `[MemoryDiagnoser]` reports Gen0/Gen1/Gen2 collections and bytes allocated per operation.
- **Multi‑runtime** — can benchmark .NET Framework, .NET Core/.NET, Mono, and NativeAOT side by side.
- **Reproducible** — benchmark projects are executable and shareable; results include environment metadata.
- **Microbenchmark‑scoped** — measures methods in isolation, not full HTTP requests, database round‑trips, or startup latency.

### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core project.
- **A separate console application** for benchmarks — BenchmarkDotNet does **not** support ASP.NET Core or web project types directly.
- `BenchmarkDotNet` NuGet package in the benchmark project.
- **Release configuration** — BenchmarkDotNet refuses to run benchmarks in Debug mode.
- Working knowledge of **C#**, **JIT compilation**, and **garbage collection**.

### Related Programming Areas

- **Profiling** — identifies hotspots in a running application; benchmarking measures specific methods in isolation.
- **Performance fundamentals** — latency, throughput, and the measurement loop.
- **Memory management** — GC pressure and allocations are first‑class benchmarking concerns.
- **CI/CD** — benchmarks can run in pipelines to detect performance regressions.
- **Unit testing** — benchmarks are *not* unit tests; they measure performance, not correctness.

### Core Concepts / Features

1. **Benchmarking** — the discipline, the methodology, and the distinction from profiling.
2. **BenchmarkDotNet** — the library, its attributes, stages, results, and diagnostics.
3. **Benchmarking best practices** — correct benchmark authoring, avoiding measurement errors, and CI integration.


## 1. Benchmarking (The Discipline)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Benchmarking is the controlled measurement of code performance — execution time and resource consumption — to compare implementations, validate optimizations, or detect regressions. |
| **Technical Definition** | A benchmark is an experiment where a **workload** (a method or algorithm) is executed repeatedly under controlled conditions, and the **dependent variables** (wall‑clock time, allocated bytes, GC collections) are measured. BenchmarkDotNet automates the **experimental design**: it determines the number of warmup iterations, the invocation count, and the number of launches required to produce statistically meaningful results. The output includes **confidence intervals**, **outlier handling**, and **environment metadata** (runtime version, CPU model, OS). |
| **Beginner‑Friendly Explanation** | Benchmarking is **“measuring before you optimize.”** Developers are famously bad at guessing where the bottleneck is. A benchmark gives you evidence: “Implementation A takes 12.3 ns with 0 bytes allocated; Implementation B takes 45.1 ns and allocates 32 bytes.” With that evidence, you can make a rational decision instead of a guess. |

### Purposes

- **To** **compare alternative implementations** objectively — `StringBuilder` vs concatenation, `Any()` vs `Count() > 0`, `System.Text.Json` vs `Newtonsoft.Json`.
- **To** **validate that an optimization actually helped** — measure before and after, rather than assuming.
- **To** **detect performance regressions** before they reach production, especially when run in CI.
- **To** **understand allocation behaviour** — a method that is slightly slower but allocates zero bytes may be preferable in a hot path.
- **To** **guide API and library design** — benchmark data informs decisions about default configurations and internal algorithms.

### Syntax Rules and Structure

#### The Benchmarking Lifecycle

```text
Design experiment → Write benchmark → Run in Release → Collect results → Interpret → Decide
```

**Component breakdown:**

| Phase | Activity | Tooling |
|-------|----------|---------|
| **Design** | Define the hypothesis (“A is faster than B”) and the variables to measure. | Methodology |
| **Write** | Create a benchmark class with `[Benchmark]` methods in a console project. | BenchmarkDotNet |
| **Run** | Execute in Release mode with sufficient iterations. | `dotnet run -c Release` |
| **Interpret** | Compare means, allocations, and statistical significance. | Results table, charts |
| **Decide** | Choose the implementation, or iterate. | Engineering judgement |

**Syntax rules:**

- **Benchmarking is for microbenchmarks** — method‑level comparisons, not full request pipelines.
- **Benchmarking must be done in Release mode** — Debug builds disable optimisations and produce meaningless results.
- **Benchmarking is not profiling** — use profilers to find the hotspot, then benchmarks to compare candidate fixes.

**Constraints and limitations:**

- **Benchmarks do not measure integration effects** — a method that is 10× faster in isolation may be irrelevant if it is called once per request and the database dominates latency.
- **Benchmarks do not measure startup time** — cold‑start latency requires a different approach.
- **Benchmarks are environment‑sensitive** — CPU, memory, and runtime version affect results; always include environment metadata.

### Annotated Complete Step‑by‑Step Code Example (Conceptual)

```csharp
// A conceptual benchmark comparing two string-building approaches
[MemoryDiagnoser]
public class StringBuildingBenchmarks
{
    private string[] _data = Array.Empty<string>();

    [GlobalSetup]
    public void Setup() => _data = Enumerable.Range(0, 100)
        .Select(i => i.ToString()).ToArray();

    [Benchmark(Baseline = true)]
    public string Concatenation()
    {
        var result = "";
        foreach (var s in _data) result += s;
        return result;   // non-void return prevents DCE
    }

    [Benchmark]
    public string StringBuilder()
    {
        var sb = new StringBuilder();
        foreach (var s in _data) sb.Append(s);
        return sb.ToString();
    }
}
```

**Step-by-step explanation:**

1. The class is marked `[MemoryDiagnoser]` to report allocations.
2. `[GlobalSetup]` runs once per benchmark case to prepare the input data.
3. `[Benchmark]` marks the two methods to compare.
4. `Baseline = true` designates the concatenation method as the comparison baseline.
5. Both methods return a non‑void value, preventing the JIT from eliminating the computation.

**Expected Output:** A results table showing that `StringBuilder` is significantly faster and allocates far less memory than string concatenation for this workload.

**Why this output?** `StringBuilder` avoids creating a new string for every concatenation; BenchmarkDotNet measures the difference with statistical rigour.

### Real-World Cases

- **Collection API choice:** `Any()` vs `Count() > 0` — `Any()` short‑circuits and is almost always faster and allocation‑free.
- **Serialization:** `System.Text.Json` vs `Newtonsoft.Json` for a specific payload shape.
- **Caching strategy:** In‑memory dictionary vs `IMemoryCache` for a hot lookup path.

### References

- BenchmarkDotNet: How to run your benchmarks – https://benchmarkdotnet.org/articles/guides/how-to-run.html
- BenchmarkDotNet: Overview – https://benchmarkdotnet.org/articles/overview.html
- BenchmarkDotNet: Welcome to the documentation – https://github.com/dotnet/BenchmarkDotNet


## 2. BenchmarkDotNet (The Library)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | BenchmarkDotNet is an open‑source .NET library that transforms annotated methods into statistically rigorous benchmarks, automating warmup, iteration control, process isolation, and result reporting. |
| **Technical Definition** | BenchmarkDotNet discovers benchmark classes via reflection, generates a separate console process for each benchmark, and executes them through a pipeline of **Pilot** (determines invocation count), **OverheadWarmup**, **ActualWarmup**, and **ActualWorkload** stages. It supports **Jobs** (runtime, JIT, platform, environment), **Diagnosers** (`MemoryDiagnoser`, `ThreadingDiagnoser`, `DisassemblyDiagnoser`, `EventPipeProfiler`), **Exporters** (Markdown, CSV, HTML), and **Parameterization** (`[Params]`, `[ParamsSource]`). |
| **Beginner‑Friendly Explanation** | BenchmarkDotNet is the **“benchmarking framework that does the statistics for you.”** You write a normal C# method, annotate it with `[Benchmark]`, and run the project. BenchmarkDotNet warms up the JIT, runs the method hundreds or thousands of times, measures allocations, and produces a table with mean, standard deviation, and memory statistics. You do not need to be a statistician — the library handles the experimental design. |

### Purposes

- **To** **transform methods into benchmarks** with a single `[Benchmark]` attribute.
- **To** **handle JIT warmup and tiered compilation** automatically, so results reflect steady‑state performance.
- **To** **isolate each benchmark in its own process**, preventing cross‑benchmark interference.
- **To** **report memory allocations** via `[MemoryDiagnoser]` — often more important than wall‑clock time.
- **To** **compare multiple runtimes** (e.g., .NET 8 vs .NET 9) and environments in one run.
- **To** **produce shareable, reproducible results** with environment metadata and export formats.

### Syntax Rules and Structure

#### Installation and Project Setup

```bash
dotnet new console -n MyApp.Benchmarks
cd MyApp.Benchmarks
dotnet add package BenchmarkDotNet
```

**Syntax rules:**

- **Benchmarks must live in a console application** — ASP.NET Core projects are not supported as benchmark hosts.
- **The project must be built in Release configuration** — BenchmarkDotNet issues an error if Debug is used.
- **The benchmark class must be `public`** and reside in the benchmark assembly.

#### Core Attributes

| Attribute | Purpose |
|-----------|---------|
| `[Benchmark]` | Marks a method as a benchmark. |
| `[MemoryDiagnoser]` | Reports Gen0/Gen1/Gen2 collections and allocated bytes. |
| `[GlobalSetup]` / `[GlobalCleanup]` | Runs once per benchmark case, before/after all iterations. |
| `[IterationSetup]` / `[IterationCleanup]` | Runs before/after every iteration (use sparingly). |
| `[Params]` | Supplies parameter values for parameterized benchmarks. |
| `[SimpleJob]` | Configures runtime, JIT, warmup count, iteration count. |
| `[Benchmark(Baseline = true)]` | Marks a benchmark as the baseline for comparison. |

#### General Syntax (Complete Benchmark Class)

```csharp
using BenchmarkDotNet.Attributes;

[MemoryDiagnoser]
[SimpleJob(warmupCount: 3, iterationCount: 10)]
public class HashBenchmarks
{
    private byte[] _data = Array.Empty<byte>();

    [GlobalSetup]
    public void Setup()
    {
        _data = new byte[10000];
        new Random(42).NextBytes(_data);
    }

    [Benchmark(Baseline = true)]
    public byte[] Md5() => MD5.HashData(_data);

    [Benchmark]
    public byte[] Sha256() => SHA256.HashData(_data);
}
```

#### Running Benchmarks

```csharp
using BenchmarkDotNet.Running;

var summary = BenchmarkRunner.Run<HashBenchmarks>();
```

**Syntax rules:**

- **Use `BenchmarkRunner.Run<T>()`** for a single benchmark class; use `BenchmarkSwitcher` for multiple classes with CLI selection.
- **Pass `args` to `BenchmarkSwitcher.Run(args)`** — otherwise, CLI flags are silently ignored.
- **Always run with `dotnet run -c Release`** —BenchmarkDotNet will not run in Debug.

#### Interpreting Results

BenchmarkDotNet produces a table like this:

| Method | Mean | Error | StdDev | Gen0 | Allocated |
|--------|------|-------|--------|------|-----------|
| Md5 | 21.91 μs | 0.138 μs | 0.129 μs | 1.0000 | 32 B |
| Sha256 | 51.57 μs | 0.311 μs | 0.291 μs | 1.0000 | 32 B |

**Component breakdown:**

| Column | Meaning |
|--------|---------|
| **Mean** | Average execution time per operation. |
| **Error** | Half‑width of the 99.9% confidence interval. |
| **StdDev** | Standard deviation of the measurements. |
| **Gen0/Gen1/Gen2** | GC collections per **1,000 operations** (value `1.0000` = one collection per 1,000 ops). |
| **Allocated** | Managed bytes allocated per operation. |

**Constraints and limitations:**

- **`MemoryDiagnoser` Gen columns are per 1,000 operations** — not per operation. A value of `1.0000` means one collection per 1,000 operations.
- **`void` benchmark methods risk dead code elimination** — the JIT may optimise away unused computations; return a value.
- **`[IterationSetup]` forces `InvocationCount=1`** — the benchmark must be long enough for a single invocation to be measurable.

### Annotated Complete Step-by-Step Code Example

```csharp
using BenchmarkDotNet.Attributes;
using BenchmarkDotNet.Running;

[MemoryDiagnoser]
public class CollectionLookupBenchmarks
{
    private List<int> _list = new();
    private HashSet<int> _set = new();

    [GlobalSetup]
    public void Setup()
    {
        var data = Enumerable.Range(0, 10_000).ToArray();
        _list = data.ToList();
        _set = data.ToHashSet();
    }

    [Benchmark(Baseline = true)]
    public bool ListContains() => _list.Contains(9_999);

    [Benchmark]
    public bool HashSetContains() => _set.Contains(9_999);
}

public class Program
{
    public static void Main(string[] args)
        => BenchmarkSwitcher.FromAssembly(typeof(Program).Assembly).Run(args);
}
```

**Step-by-step explanation:**

1. `[MemoryDiagnoser]` enables allocation reporting.
2. `[GlobalSetup]` prepares the `List<int>` and `HashSet<int>` once per benchmark case.
3. `ListContains` is the baseline; `HashSetContains` is the candidate.
4. `BenchmarkSwitcher` allows running both benchmarks and selecting them via CLI.
5. The results will show `HashSet` is O(1) and dramatically faster than the O(n) `List.Contains`.

**Expected Output:** A table showing `HashSetContains` with a mean time orders of magnitude lower than `ListContains`, with zero allocations for both.

**Why this output?** `HashSet` uses hashing for O(1) lookups; `List` scans linearly. BenchmarkDotNet quantifies the difference with statistical confidence.

### Real-World Cases

- **API design decisions:** `Any()` vs `Count() > 0`, `StringBuilder` vs concatenation.
- **Serialization libraries:** `System.Text.Json` vs `Newtonsoft.Json` for a specific payload.
- **Collection choices:** `List<T>` vs `HashSet<T>` vs `Dictionary<TKey, TValue>` for lookup‑heavy workloads.
- **CI regression detection:** Run benchmarks on every PR and fail if a hot path regresses beyond a threshold.

### References

- BenchmarkDotNet: How to run your benchmarks – https://benchmarkdotnet.org/articles/guides/how-to-run.html
- BenchmarkDotNet: Overview – https://benchmarkdotnet.org/articles/overview.html
- BenchmarkDotNet: Measurement and diagnostics – https://github.com/cratis/applicationmodel/blob/0eac455579644b25fac8967e7cfc505c58d3aa1b/.cratis/ai/skills/microbenchmarking/references/diagnosers-and-exporters.md
- BenchmarkDotNet: Jobs – https://benchmarkdotnet.org/articles/configs/jobs.html
- BenchmarkDotNet: SimpleJobAttribute – https://benchmarkdotnet.org/api/BenchmarkDotNet.Attributes.SimpleJobAttribute.html
- BenchmarkDotNet: Baseline – https://benchmarkdotnet.org/articles/features/baselines.html


## 3. Benchmarking Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Benchmarking best practices are the guidelines that ensure benchmark results are correct, reproducible, and actionable — not silently misleading. |
| **Technical Definition** | Best practices encompass **dead code elimination prevention** (returning a value from `[Benchmark]` methods), **setup and cleanup discipline** (`[GlobalSetup]` vs `[IterationSetup]`), **avoiding measured initialization**, **correct handling of async and deferred execution**, **statistical interpretation** (confidence intervals, repeated runs), **process isolation awareness**, and **CI integration** (regression detection). |
| **Beginner‑Friendly Explanation** | Benchmarking best practices are the **“rules of measurement”** —they help you avoid the classic mistakes: benchmarking a `void` method that the JIT optimises away, putting setup code inside the benchmark and measuring it, trusting a single run, and benchmarking a Debug build. The single most important rule is: **incorrect benchmarks silently produce misleading results** —there is no error message; you just get wrong numbers. |

### Purposes

- **To** prevent **dead code elimination** —ensure the benchmarked computation is not optimised away by the JIT.
- **To** keep **setup and cleanup outside the measurement** —initialization logic inside the benchmark method is measured.
- **To** ensure **statistical validity** —run enough iterations and repeat runs to trust the results.
- **To** **benchmark the right thing** —hot‑path changes, not architectural changes that require load testing.
- **To** **integrate benchmarks into CI** —catch performance regressions before they reach production.

### Guidelines Checklist

| Area | Do | Don’t |
|------|----|-------|
| **Dead code elimination** | Return a value from `[Benchmark]` methods. | Use `void` benchmarks for computations with no observable side effects. |
| **Setup** | Use `[GlobalSetup]`; keep initialization out of the benchmark method. | Put data loading or object creation inside the measured method. |
| **Iteration setup** | Use `[IterationSetup]` only when state must be reset between iterations (e.g., sorting). | Use `[IterationSetup]` casually —it forces single‑invocation measurements and requires long operations. |
| **Async** | Use `async Task` benchmark methods; `await` the work inside. | Use `.Result` or `.Wait()` in benchmarks —it distorts timing and can deadlock. |
| **Deferred execution** | Force enumeration with `Consumer` or `.ToList()` when benchmarking LINQ. | Return an unenumerated `IEnumerable<T>` —the work is not measured. |
| **Loops** | Benchmark a single operation; BenchmarkDotNet handles repetition. | Write your own loop inside the benchmark —it distorts the measurement. |
| **Release mode** | Always `dotnet run -c Release`. | Benchmark in Debug mode —results are meaningless. |
| **Statistical interpretation** | Compare means with confidence intervals; repeat runs before acting on small deltas. | Decide based on a single run’s best or worst number. |
| **Scope** | Benchmark hot‑path methods where microseconds matter. | Benchmark architectural changes (caching, DI lifetime) —use load testing. |

### Annotated Code Example (Best-Practice Async Benchmark)

```csharp
[MemoryDiagnoser]
public class AsyncBenchmarks
{
    private HttpClient _client = null!;

    [GlobalSetup]
    public void Setup() => _client = new HttpClient
    {
        BaseAddress = new Uri("https://localhost:5001")
    };

    [Benchmark]
    public async Task<string> GetStringAsync()
    {
        var response = await _client.GetAsync("/api/values");
        return await response.Content.ReadAsStringAsync();
    }
}
```

**Why this follows best practices:**

- **`async Task` benchmark method** —BenchmarkDotNet correctly awaits the asynchronous work.
- **`[GlobalSetup]`** creates the `HttpClient` once, not per iteration.
- **The method returns `Task<string>`** —the result is observable, preventing dead code elimination.
- **No `.Result` or `.Wait()`** —blocking would distort timing and consume threads.

### Real-World Cases

- **Hot-path optimization:** Benchmark a serialization method before and after switching from `Newtonsoft.Json` to `System.Text.Json`.
- **API design:** Benchmark `Any()` vs `Count() > 0` and use the result to standardize code reviews.
- **CI gates:** Run `dotnet run -c Release` in CI and compare against a stored baseline; fail if the mean regresses beyond a threshold.

### References

- Benchmark authoring techniques (dead code elimination, setup, async, deferred execution) – https://github.com/foundatiofx/foundatio/blob/be10785cfed4e1850350dfaa024e1c3f25b7fac7/.agents/skills/microbenchmarking/references/writing-benchmarks.md
- Measurement and diagnostics (MemoryDiagnoser, Gen columns, diagnosers) – https://github.com/cratis/applicationmodel/blob/0eac455579644b25fac8967e7cfc505c58d3aa1b/.cratis/ai/skills/microbenchmarking/references/diagnosers-and-exporters.md
- BenchmarkDotNet: How to run your benchmarks (Release mode requirement) – https://benchmarkdotnet.org/articles/guides/how-to-run.html
- BenchmarkDotNet: Overview (Release mode requirement) – https://benchmarkdotnet.org/articles/overview.html
- Measurement Guide (when to benchmark vs load test) – https://github.com/…/measurement-guide.md
- Microbenchmark design guidelines (BenchmarkDotNet authoring) – https://github.com/dotnet/skills/blob/6a37d48731ae86a009cf92c4b24796b0ed9f0ea5/plugins/dotnet-diag/skills/microbenchmarking/references/writing-benchmarks.md


## Summary Table: Benchmarking at a Glance

| Concept | Primary Role | Key Tool / Syntax | When to Use |
|---------|--------------|-------------------|-------------|
| **Benchmarking** | Measure method‑level performance rigorously | `[Benchmark]`, `[MemoryDiagnoser]`, `dotnet run -c Release` | Comparing implementations, validating optimizations, CI regression detection |
| **BenchmarkDotNet** | The library that does the statistics | `BenchmarkRunner.Run<T>()`, `BenchmarkSwitcher`, Jobs, Diagnosers | Every .NET microbenchmark |
| **Best practices** | Correct, reproducible, actionable results | Non‑void returns, `[GlobalSetup]`, `async Task`, Release mode | Every benchmark authoring session |

---

## References (Full List)

- BenchmarkDotNet: How to run your benchmarks – https://benchmarkdotnet.org/articles/guides/how-to-run.html
- BenchmarkDotNet: Overview – https://benchmarkdotnet.org/articles/overview.html
- BenchmarkDotNet: Welcome to the documentation – https://github.com/dotnet/BenchmarkDotNet
- BenchmarkDotNet: Measurement and diagnostics – https://github.com/cratis/applicationmodel/blob/0eac455579644b25fac8967e7cfc505c58d3aa1b/.cratis/ai/skills/microbenchmarking/references/diagnosers-and-exporters.md
- BenchmarkDotNet: Benchmark authoring techniques – https://github.com/foundatiofx/foundatio/blob/be10785cfed4e1850350dfaa024e1c3f25b7fac7/.agents/skills/microbenchmarking/references/writing-benchmarks.md
- BenchmarkDotNet: Jobs – https://benchmarkdotnet.org/articles/configs/jobs.html
- BenchmarkDotNet: SimpleJobAttribute – https://benchmarkdotnet.org/api/BenchmarkDotNet.Attributes.SimpleJobAttribute.html
- BenchmarkDotNet: Baseline – https://benchmarkdotnet.org/articles/features/baselines.html
- BenchmarkDotNet: MemoryDiagnoser (Gen columns explanation) – https://github.com/cratis/applicationmodel/blob/0eac455579644b25fac8967e7cfc505c58d3aa1b/.cratis/ai/skills/microbenchmarking/references/diagnosers-and-exporters.md
- BenchmarkDotNet: Release configuration requirement – https://benchmarkdotnet.org/articles/overview.html
- BenchmarkDotNet: Console app requirement – https://benchmarkdotnet.org/articles/guides/how-to-run.html
- Measurement Guide (when to benchmark vs load test) – https://github.com/…/measurement-guide.md
- Microbenchmark design guidelines (BenchmarkDotNet authoring) – https://github.com/dotnet/skills/blob/6a37d48731ae86a009cf92c4b24796b0ed9f0ea5/plugins/dotnet-diag/skills/microbenchmarking/references/writing-benchmarks.md
- BenchmarkDotNet in ASP.NET Core projects (separate console project) – https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/dotnet/14-net-testing/8-net-performance-testing.mdx
- BenchmarkDotNet with Visual Studio Profiler – https://cloud.tencent.cn/developer/article/2487064