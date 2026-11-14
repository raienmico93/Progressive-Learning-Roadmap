# Benchmarking in C# — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Benchmarking is the practice of measuring the performance of code — execution time, memory allocation, and throughput — under controlled, repeatable conditions to make evidence-based optimisation decisions.

**Technical Definition**
Benchmarking in .NET is the systematic measurement of a program's execution characteristics using a harness that isolates the code under test, warms up the JIT compiler to steady state (Tier 1), executes the operation many times, and applies statistical analysis to the collected samples. BenchmarkDotNet is the de facto standard harness; it generates an isolated project for each benchmark, manages warm-up and iteration counts, reports mean, median, standard deviation, and 99.9% confidence intervals, and supports memory diagnostics, multiple runtimes, and baseline comparisons. Benchmarking is distinct from profiling: profiling tells you **where** time is spent in a running application; benchmarking tells you **how fast** a specific operation is under controlled conditions. Both are necessary, but they answer different questions.

**Beginner-Friendly Explanation**
Benchmarking is “the scientific method for measuring code speed.” Instead of saying “this looks faster” or using a single `Stopwatch` reading, benchmarking runs the code many times in a controlled environment, throws away warm-up noise, and tells you with statistical confidence whether one approach is actually better than another. In .NET, the tool that does this for you is BenchmarkDotNet.

---

### Key Characteristics

- **Statistical rigor** — Reports confidence intervals and standard deviation, not a single measurement.
- **JIT-aware** — Warms up to Tier 1 compilation before measuring, avoiding Tier 0 cold-start fiction.
- **Isolation** — Generates a separate project per benchmark to prevent cross-contamination.
- **Allocation-aware** — `[MemoryDiagnoser]` reports bytes allocated per operation.
- **Multi-runtime** — `[SimpleJob(RuntimeMoniker.Net80)]` compares .NET versions side by side.
- **Baseline-driven** — `[Benchmark(Baseline = true)]` produces ratio columns for direct comparison.
- **CI-integrable** — Exporters (JSON, Markdown, CSV) enable regression gates in pipelines.
- **Distinct from profiling** — Benchmarking measures controlled micro-operations; profiling observes production behavior.

---

### Prerequisites

- Working knowledge of C# methods, classes, and the .NET runtime (JIT, GC, Tier 0/Tier 1).
- Familiarity with the .NET CLI (`dotnet build -c Release`, `dotnet run`).
- The BenchmarkDotNet NuGet package (`BenchmarkDotNet` version 0.14+).
- Understanding of the difference between Release and Debug builds.
- Awareness of the code path being measured and its expected steady-state behavior.

---

### Related Programming Areas

- **Profiling** — Identifying hot paths in a running application.
- **Performance fundamentals** — Latency, throughput, allocation, and GC pressure.
- **Microbenchmarking** — Measuring small, isolated code units.
- **CI/CD performance gates** — Failing builds on regressions.
- **Garbage collection tuning** — Understanding allocation impact on GC generations.

---

### Core Concepts / Features

1. Benchmarking (overarching discipline)
2. BenchmarkDotNet
3. Microbenchmarking
4. Benchmarking best practices

Each concept follows the same uniform structure: definitions (core, technical, beginner-friendly), purposes (each beginning with “To”), syntax rules and structure with complete general syntaxes and component breakdowns, multiple annotated code examples with expected outputs, real-world cases, and references.

---

## 1. Benchmarking (Overarching Discipline)

### Definitions

**Core Definition**
Benchmarking is the controlled measurement of code performance to compare implementations, track regressions, or validate optimisation claims.

**Technical Definition**
Benchmarking in .NET requires addressing four sources of measurement error: **JIT warm-up** (Tier 0 to Tier 1 compilation), **dead code elimination** (the JIT removing unused computations), **OS noise** (background processes, power management), and **measurement overhead** (the act of measuring distorting the result). BenchmarkDotNet solves these by generating an isolated executable, running a pilot phase to determine iteration counts, executing warm-up iterations, then measuring steady-state invocations with statistical analysis. The result is a distribution of per-operation times, reported with mean, median, standard deviation, and confidence intervals. A benchmark without statistical rigor — a hand-rolled `Stopwatch` loop — is not a benchmark; it is an anecdote.

**Beginner-Friendly Explanation**
Benchmarking is “measuring code speed the right way.” If you just time a method once with a stopwatch, you might be measuring JIT warm-up, or the OS might have been busy with something else. BenchmarkDotNet takes care of all of that: it warms up the code, runs it many times, and gives you a statistically trustworthy number.

---

### Purposes

- To compare alternative implementations fairly and reproducibly.
- To detect performance regressions before they reach production.
- To validate that an optimisation actually improves performance.
- To measure memory allocation per operation.
- To establish a performance baseline for CI gates.
- To guide algorithmic and design decisions with evidence.

---

### Syntax Rules and Structure

#### General Benchmarking Workflow

```
1. Define the operation and the alternatives to compare.
2. Write a BenchmarkDotNet benchmark class with [Benchmark] methods.
3. Move setup to [GlobalSetup]; never measure initialisation.
4. Return a value from each benchmark method to prevent dead code elimination.
5. Run in Release mode without a debugger attached.
6. Analyse the results: mean, ratio, allocation, confidence intervals.
7. Compare against the baseline and draw conclusions.
```

**Component breakdown**

- **Operation** — The specific code path being measured.
- **Benchmark class** — A public, non-sealed, non-static class containing `[Benchmark]` methods.
- **`[GlobalSetup]`** — Runs once per benchmark case; allocates buffers and loads data outside the measured region.
- **Return value** — Prevents the JIT from eliminating the measured computation.
- **Release run** — Debug builds are 10–100× slower and must never be benchmarked.
- **Statistics** — Mean, median, standard deviation, and 99.9% confidence interval.

#### Syntax Rules

- Benchmark classes must be `public`, non-sealed, non-static, and classes (not structs).
- `[Benchmark]` methods must be `public` and `instance`.
- Initialisation must be in `[GlobalSetup]`, not in the benchmark method.
- The benchmark method should return a value to defeat dead code elimination.
- Never run benchmarks in Debug mode or with an attached debugger.

#### Constraints and Limitations

- Microbenchmarks may not reflect production behaviour under contention or realistic workloads.
- Benchmarking adds build time because BDN generates and compiles a separate project.
- Very fast methods require many invocations; BDN handles this automatically but increases run time.
- Environment differences (CPU, OS, .NET version) can change results significantly.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Correct vs. Incorrect Benchmark Method

```csharp
using BenchmarkDotNet.Attributes;

[MemoryDiagnoser]
public class ParsingBenchmarks
{
    // ✅ CORRECT: returns a value — JIT cannot eliminate the computation
    [Benchmark]
    public int CorrectParse() => int.Parse("12345");

    // ❌ WRONG: returns void — JIT may eliminate the call entirely
    [Benchmark]
    public void WrongParse() => int.Parse("12345");
}
```

**Expected Output (conceptual)**

```
| Method      | Mean      | Allocated |
|------------ |----------:|----------:|
| CorrectParse| 12.4 ns   | 0 B       |
| WrongParse  |  0.1 ns   | 0 B       |
```

**Why this output occurs**

- `CorrectParse` returns an `int`; the JIT cannot prove the result is unused and must execute the parse.
- `WrongParse` returns `void`; the JIT may eliminate the entire call because the result is unobservable.
- The near-zero time for `WrongParse` is a classic sign of dead code elimination.

---

### Real-World Cases

- **Library API design** — Comparing `Span<T>` parsing vs. `Substring` allocation.
- **Serialization** — `System.Text.Json` vs. Newtonsoft.Json throughput and allocation.
- **Collection selection** — `Dictionary` vs. `ConcurrentDictionary` vs. `ImmutableDictionary`.
- **LINQ vs. manual loops** — Measuring iterator and delegate overhead.

---

### References

- BenchmarkDotNet — Good Practices - https://benchmarkdotnet.org/articles/guides/good-practices.html
- Benchmark authoring techniques — dotnet/skills - https://github.com/dotnet/skills/blob/ac8f41264bdd557e58924a3110eea8e0917dcf4d/plugins/dotnet-diag/skills/microbenchmarking/references/writing-benchmarks.md
- Microbenchmark Design Guidelines — raw.githubusercontent.com
- Ad Hoc Microbenchmarking Locally — dotnet/dotnet - https://github.com/dotnet/dotnet/blob/60864f1c9f7bc774d1c191ef6f1630e81607cd62/src/runtime/.github/skills/microbenchmark/SKILL.md

---

## 2. BenchmarkDotNet

### Definitions

**Core Definition**
BenchmarkDotNet is a .NET library that transforms ordinary methods into rigorous, statistically validated benchmarks with minimal configuration.

**Technical Definition**
BenchmarkDotNet generates an isolated project for each benchmark class, compiles it in Release mode, and executes it through a warm-up, pilot, and measurement pipeline. It supports attributes for job configuration (`[SimpleJob]`, `[ShortRunJob]`), diagnostics (`[MemoryDiagnoser]`, `[DisassemblyDiagnoser]`), setup/cleanup (`[GlobalSetup]`, `[IterationSetup]`), parameterisation (`[Params]`), baseline marking (`[Benchmark(Baseline = true)]`), and result formatting (`[RankColumn]`, `[MarkdownExporter]`). Results are reported with mean, error (half of the 99.9% confidence interval), standard deviation, median, and allocated bytes. BDN is the standard harness used by the .NET runtime team itself.

**Beginner-Friendly Explanation**
BenchmarkDotNet is “the tool that writes the benchmark harness for you.” You write a method with `[Benchmark]`, and BDN handles warm-up, iteration counts, statistics, and memory measurement. It even generates a separate project so your benchmark does not interfere with your application.

---

### Purposes

- To provide statistically rigorous micro-benchmark measurement.
- To compare implementations side by side with ratio and ranking columns.
- To measure memory allocation per operation.
- To compare across .NET runtimes.
- To export results for CI and reporting.
- To eliminate the measurement errors of hand-rolled `Stopwatch` code.

---

### Syntax Rules and Structure

#### Project Setup

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net8.0</TargetFramework>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="BenchmarkDotNet" Version="0.14.0" />
  </ItemGroup>
</Project>
```

```csharp
using BenchmarkDotNet.Running;

public class Program
{
    public static void Main(string[] args) =>
        BenchmarkSwitcher.FromAssembly(typeof(Program).Assembly).Run(args);
}
```

**Component breakdown**

- `OutputType Exe` — The benchmark project is a console application.
- `BenchmarkDotNet` package — The harness library.
- `BenchmarkSwitcher` — Entry point that discovers and runs benchmarks; supports command-line filtering.
- `Run(args)` — Passes CLI arguments (e.g., `--filter *StringBenchmarks*`) to BDN.

#### Core Attributes

| Attribute | Purpose |
|---|---|
| `[Benchmark]` | Marks a method for measurement. |
| `[MemoryDiagnoser]` | Reports allocated bytes and GC collections. |
| `[SimpleJob(RuntimeMoniker.Net80)]` | Targets a specific .NET runtime. |
| `[RankColumn]` | Adds a fastest-to-slowest ranking column. |
| `[Baseline = true]` | Marks a method as the comparison baseline (Ratio = 1.00). |
| `[Params(...)]` | Runs the benchmark with multiple input sizes. |
| `[GlobalSetup]` / `[GlobalCleanup]` | Runs once per benchmark case, outside the measured region. |
| `[IterationSetup]` / `[IterationCleanup]` | Runs before/after each iteration; forces `InvocationCount=1` and `UnrollFactor=1`; not for microbenchmarks. |

#### Syntax Rules

- Benchmark classes must be `public`, non-sealed, non-static, and classes (not structs).
- `[Benchmark]` methods must be `public` and `instance`.
- Only one method per class can have `Baseline = true`.
- `[IterationSetup]` is only appropriate for macrobenchmarks (≥ 100 ms per invocation).
- Use `BenchmarkSwitcher.FromAssembly(...).Run(args)` rather than a hard-coded runner.

#### Constraints and Limitations

- `[IterationSetup]` spoils microbenchmark results because it forces single-invocation iterations.
- `[DisassemblyDiagnoser]` may crash on Linux; use `DOTNET_JitDisasm` environment variable instead.
- BDN generates and compiles a separate project, increasing build time.
- Results are environment-specific and should not be extrapolated without caution.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Baseline Comparison with RankColumn

```csharp
using BenchmarkDotNet.Attributes;

[MemoryDiagnoser]
[RankColumn]
public class StringBenchmarks
{
    private const string Input = "Hello, World!";

    [Benchmark(Baseline = true)]
    public string Substring() => Input.Substring(0, 5);

    [Benchmark]
    public ReadOnlySpan<char> SpanSlice() => Input.AsSpan(0, 5);
}
```

**Expected Output (conceptual)**

```
| Method     | Mean      | Ratio | Allocated | Rank |
|----------- |----------:|------:|----------:|-----:|
| SpanSlice  |  0.82 ns  |  0.07 |       0 B |    1 |
| Substring  | 11.24 ns  |  1.00 |      32 B |    2 |
```

**Why this output occurs**

- `Substring` is the baseline; its `Ratio` is 1.00 by definition.
- `SpanSlice` is about 14× faster (Ratio 0.07) and allocates zero bytes.
- `RankColumn` orders fastest to slowest.

---

#### Example 2: Params and GlobalSetup

```csharp
using BenchmarkDotNet.Attributes;

[MemoryDiagnoser]
public class CollectionBenchmarks
{
    [Params(100, 1000, 10000)]
    public int Size;

    private int[] _data;

    [GlobalSetup]
    public void Setup()
    {
        _data = new int[Size];
        for (int i = 0; i < Size; i++) _data[i] = i;
    }

    [Benchmark]
    public int ArraySum()
    {
        int sum = 0;
        foreach (var x in _data) sum += x;
        return sum;
    }

    [Benchmark]
    public int LinqSum() => _data.Sum();
}
```

**Expected Output (conceptual)**

```
| Method   | Size  | Mean       | Allocated |
|--------- |------ |-----------:|----------:|
| ArraySum | 100   |  42.1 ns   |       0 B |
| LinqSum  | 100   | 128.7 ns   |      32 B |
| ArraySum | 1000  | 421.3 ns   |       0 B |
| LinqSum  | 1000  | 1,287.4 ns |      32 B |
```

**Why this output occurs**

- `[Params]` creates separate benchmark cases for each size.
- `[GlobalSetup]` allocates the array once per size, outside the measured method.
- `LinqSum` incurs enumerator and delegate allocations.

---

#### Example 3: Async Setup and Benchmark

```csharp
using BenchmarkDotNet.Attributes;

[MemoryDiagnoser]
public class AsyncBenchmarks
{
    private HttpClient _client;

    [GlobalSetup]
    public async Task SetupAsync()
    {
        _client = new HttpClient();
        await Task.CompletedTask; // async setup is awaited by BDN
    }

    [Benchmark]
    public async Task<int> GetStatusCodeAsync()
    {
        var response = await _client.GetAsync("https://example.com");
        return (int)response.StatusCode;
    }

    [GlobalCleanup]
    public void Cleanup() => _client.Dispose();
}
```

**Expected Output (conceptual)**

```
| Method              | Mean     | Allocated |
|-------------------- |---------:|----------:|
| GetStatusCodeAsync  | 124.3 ms |  12.4 KB  |
```

**Why this output occurs**

- `[GlobalSetup]` returning `Task` is awaited by BenchmarkDotNet automatically.
- `[Benchmark]` returning `Task<T>` is awaited and measured correctly.
- The HTTP call dominates the measurement; this is not a microbenchmark but an integration-style benchmark.

---

### Real-World Cases

- **String parsing** — `Span<T>` vs. `Substring` vs. regex.
- **Serialization** — `System.Text.Json` vs. Newtonsoft.Json.
- **Collections** — `Dictionary` vs. `FrozenDictionary` vs. `ConcurrentDictionary`.
- **EF Core queries** — Compiled vs. non-compiled queries.

---

### References

- Setup And Cleanup — BenchmarkDotNet - https://benchmarkdotnet.org/articles/features/setup-and-cleanup.html
- Benchmark authoring techniques — dotnet/skills - https://github.com/dotnet/skills/blob/ac8f41264bdd557e58924a3110eea8e0917dcf4d/plugins/dotnet-diag/skills/microbenchmarking/references/writing-benchmarks.md
- Benchmark and Job Baselines — BenchmarkDotNet - https://benchmarkdotnet.org/articles/features/baselines.html
- Comparing benchmark results — GitHub - https://github.com/…/comparing-benchmark-results
- Benchmark Project Implementation Plan — zeroalloc.validation - https://github.com/zeroalloc-net/zeroalloc.validation/blob/a6cda35b93123c3ed768742a993235cc1a5b948e/docs/plans/2026-03-16-benchmarks.md
- Visual Studio BenchmarkDotNet analysis — Microsoft Learn - https://learn.microsoft.com/en-us/visualstudio/profiling/benchmarkdotnet

---

## 3. Microbenchmarking

### Definitions

**Core Definition**
Microbenchmarking is the practice of measuring the performance of a small, isolated unit of code — a method, an algorithm, or a single operation — under tightly controlled conditions.

**Technical Definition**
Microbenchmarking isolates a specific operation and measures its execution time and allocation, typically in the nanosecond to microsecond range. It requires eliminating all sources of noise: JIT warm-up (Tier 0 → Tier 1), dead code elimination, constant folding (the JIT evaluating constant expressions at compile time), loop overhead (BenchmarkDotNet handles iteration), and OS scheduling jitter. Microbenchmarks answer questions like “Is `Span<T>` parsing faster than `Substring`?” and “Does this collection type allocate?” They do not answer “Is my application fast?” — that requires profiling and load testing.

**Beginner-Friendly Explanation**
Microbenchmarking is “measuring one tiny operation in isolation.” If you want to know whether `StringBuilder` is faster than string concatenation for a specific case, you write a microbenchmark. But you cannot conclude from a microbenchmark that your entire API will be fast — you still need profiling and load testing for that.

---

### Purposes

- To compare alternative implementations of the same operation fairly.
- To measure allocation per operation (bytes).
- To validate that a code change actually improves the measured operation.
- To establish performance budgets for hot-path methods.
- To detect regressions in library APIs across versions.
- To inform design decisions with evidence rather than intuition.

---

### Syntax Rules and Structure

#### General Microbenchmark Structure

```csharp
using BenchmarkDotNet.Attributes;

[MemoryDiagnoser]
public class MyBenchmarks
{
    private InputType _input;

    [GlobalSetup]
    public void Setup()
    {
        _input = PrepareInput(); // allocation outside the measured region
    }

    [Benchmark(Baseline = true)]
    public ResultType BaselineImplementation() => _input.Process();

    [Benchmark]
    public ResultType AlternativeImplementation() => _input.ProcessFaster();
}
```

**Component breakdown**

- `[GlobalSetup]` — Allocates input data once, outside the measured method.
- `[Benchmark]` — The measured operation; must return a value.
- `[Baseline = true]` — The reference implementation for ratio comparison.
- `[MemoryDiagnoser]` — Reports allocated bytes per operation.

#### Syntax Rules

- **Return a value** from every `[Benchmark]` method to prevent dead code elimination.
- **Avoid loops** inside the benchmark method; BDN invokes the method many times automatically.
- **Avoid side effects**; the benchmark should be pure and produce consistent results.
- **Use consistent input data** across runs for reproducible comparisons.
- **Move initialisation to `[GlobalSetup]`**; never measure setup.
- **Do not use `[IterationSetup]`** for microbenchmarks; it forces single-invocation iterations and distorts results.
- **Benchmark the common case**, not edge cases or error paths.

#### Constraints and Limitations

- Microbenchmarks cannot measure the effect of contention, I/O latency, or cache behaviour under production load.
- Very fast operations (sub-nanosecond) are difficult to measure even with BDN.
- Constant folding may eliminate operations whose inputs are compile-time constants.
- Results are sensitive to CPU architecture, OS, and .NET runtime version.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Preventing Dead Code Elimination

```csharp
using BenchmarkDotNet.Attributes;

[MemoryDiagnoser]
public class MathBenchmarks
{
    // ❌ DCE risk: void return, result unused
    [Benchmark]
    public void WrongExp() => Math.Exp(1);

    // ✅ Correct: return the result
    [Benchmark]
    public double CorrectExp() => Math.Exp(1);
}
```

**Expected Output (conceptual)**

```
| Method     | Mean      | Allocated |
|----------- |----------:|----------:|
| CorrectExp | 12.4 ns   | 0 B       |
| WrongExp   |  0.1 ns   | 0 B       |
```

**Why this output occurs**

- `WrongExp` returns `void`; the JIT can prove `Math.Exp(1)` has no observable effect and eliminates it.
- `CorrectExp` returns the result; the JIT must compute it.
- The near-zero time for `WrongExp` is a red flag for DCE.

---

#### Example 2: Deferred Execution Pitfall

```csharp
using System.Linq;
using BenchmarkDotNet.Attributes;

[MemoryDiagnoser]
public class DeferredBenchmarks
{
    private int[] _data = Enumerable.Range(0, 1000).ToArray();

    // ❌ WRONG: returns the IEnumerable without enumerating it
    [Benchmark]
    public IEnumerable<int> WrongWhere() => _data.Where(x => x % 2 == 0);

    // ✅ CORRECT: force enumeration inside the benchmark
    [Benchmark]
    public int CorrectWhere() => _data.Where(x => x % 2 == 0).Sum();
}
```

**Expected Output (conceptual)**

```
| Method      | Mean      | Allocated |
|------------ |----------:|----------:|
| CorrectWhere| 12.4 μs   | 32 B      |
| WrongWhere  |  0.2 ns   | 24 B      |
```

**Why this output occurs**

- `WrongWhere` returns an unenumerated `IEnumerable`; the actual filtering never runs inside the measured region.
- `CorrectWhere` forces enumeration with `Sum()`, so the LINQ pipeline is measured.
- The near-zero time for `WrongWhere` reveals the missing enumeration.

---

### Real-World Cases

- **Span vs. Substring** — Zero-allocation slicing vs. string allocation.
- **LINQ vs. loops** — Measuring iterator and delegate overhead in hot paths.
- **Collection lookups** — `Dictionary` vs. `FrozenDictionary` vs. sorted array.
- **Regex vs. manual parsing** — Compiled regex vs. `Span<T>`-based parsers.

---

### References

- Benchmark authoring techniques — dotnet/skills - https://github.com/dotnet/skills/blob/ac8f41264bdd557e58924a3110eea8e0917dcf4d/plugins/dotnet-diag/skills/microbenchmarking/references/writing-benchmarks.md
- Ad Hoc Microbenchmarking Locally — dotnet/dotnet - https://github.com/dotnet/dotnet/blob/60864f1c9f7bc774d1c191ef6f1630e81607cd62/src/runtime/.github/skills/microbenchmark/SKILL.md
- Good Practices — BenchmarkDotNet - https://benchmarkdotnet.org/articles/guides/good-practices.html
- Beware .NET Microbenchmark Cold Starts — LinkedIn - https://www.linkedin.com

---

## 4. Benchmarking Best Practices

### Definitions

**Core Definition**
Benchmarking best practices are empirically grounded guidelines for writing, running, and interpreting benchmarks in a way that produces reliable, actionable results.

**Technical Definition**
Best practices address methodology (measure the right thing, compare against a baseline), measurement correctness (Release builds, no debugger, dead code elimination, constant folding, deferred execution), environment control (CPU governor, power settings, background processes), and statistical interpretation (confidence intervals, standard deviation, environmental variability). They also cover organisational discipline: performance budgets, CI regression gates, and avoiding the trap of optimising microbenchmarks that do not reflect production behavior.

**Beginner-Friendly Explanation**
Best practices are “the habits that keep benchmarks honest.” They include running in Release mode, returning values so the JIT cannot delete your code, not measuring setup, and not assuming a microbenchmark result applies everywhere.

---

### Purposes

- To produce benchmarks that measure what they claim to measure.
- To avoid misleading results caused by JIT optimisations or measurement artifacts.
- To enable reproducible comparisons across runs and environments.
- To integrate benchmarking into a disciplined performance engineering workflow.
- To prevent performance regressions through CI gates.
- To ensure that optimisation effort is directed at code that matters.

---

### Guidelines and Structure

#### Use Release Builds Without a Debugger

```
✅ <Optimize>true</Optimize> in the csproj (Release configuration)
✅ Run from the command line, not from the IDE with a debugger attached
❌ Debug builds can be 10–100× slower and must never be benchmarked
```

*Source: BenchmarkDotNet Good Practices*

#### Avoid Dead Code Elimination

```
✅ Return a value from every [Benchmark] method
✅ Use the result in a way the JIT cannot prove is unused
❌ Do not return void for benchmarks whose result is otherwise unobservable
```

*Source: BenchmarkDotNet Good Practices; dotnet/skills benchmark authoring techniques*

#### Move Initialization to GlobalSetup

```
✅ [GlobalSetup] for buffers, input data, and expensive initialisation
❌ Do not measure initialisation inside the [Benchmark] method
```

*Source: dotnet/skills benchmark authoring techniques*

#### Avoid Loops Inside Benchmarks

```
✅ Let BenchmarkDotNet invoke the method many times automatically
❌ Do not add manual loops; they distort measurements and interfere with BDN’s iteration strategy
```

*Source: dotnet/skills benchmark authoring techniques*

#### No Side Effects, Consistent Inputs

```
✅ Benchmarks should be pure and produce consistent results
✅ Use the same input data across runs for reproducibility
❌ Avoid random data, I/O, network calls, or logging inside the measured method
```

#### Do Not Use IterationSetup for Microbenchmarks

```
✅ [GlobalSetup] runs once per benchmark case
❌ [IterationSetup] forces InvocationCount=1 and UnrollFactor=1, spoiling microbenchmark results
✅ [IterationSetup] is only appropriate for macrobenchmarks (≥ 100 ms per invocation)
```

*Source: BenchmarkDotNet Setup and Cleanup documentation*

#### Control the Environment

```
✅ Keep the machine plugged in and on maximum performance mode
✅ Close all other applications and IDEs during benchmarking
✅ Benchmark steady state (Tier 1), not cold starts (Tier 0)
❌ Do not extrapolate results across different CPUs, OSes, or .NET versions
```

*Source: BenchmarkDotNet Good Practices; microbenchmark cold-start guidance*

#### Use Baselines and Statistical Analysis

```
✅ Mark one method with [Benchmark(Baseline = true)] for ratio comparison
✅ Read the 99.9% confidence interval and standard deviation, not just the mean
✅ Use [RankColumn] for a quick fastest-to-slowest view
❌ Do not trust a single run without checking variance
```

*Source: BenchmarkDotNet Baselines; statistics documentation*

#### Benchmark the Common Case

```
✅ Benchmark hot paths and typical usage
❌ Do not benchmark edge cases or error paths unless they are the actual concern
```

*Source: dotnet/skills benchmark authoring techniques*

#### Integrate with CI

```
✅ Use --exporters json markdown to produce artifacts for CI
✅ Compare against a stored baseline and fail on regression
✅ Use the Datadog.Trace.BenchmarkDotNet exporter for CI visibility (optional)
```

*Source: BenchmarkDotNet exporters; CI integration examples*

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Correct Benchmark Following All Best Practices

```csharp
using BenchmarkDotNet.Attributes;

[MemoryDiagnoser]
[RankColumn]
public class BestPracticeBenchmarks
{
    private int[] _data;

    [GlobalSetup]
    public void Setup()
    {
        _data = Enumerable.Range(0, 1000).ToArray();
    }

    [Benchmark(Baseline = true)]
    public int LinqSum() => _data.Sum();

    [Benchmark]
    public int LoopSum()
    {
        int sum = 0;
        foreach (var x in _data) sum += x;
        return sum;
    }
}
```

**Expected Output (conceptual)**

```
| Method   | Mean      | Ratio | Allocated | Rank |
|--------- |----------:|------:|----------:|-----:|
| LoopSum  |  842.1 ns |  0.31 |       0 B |    1 |
| LinqSum  | 2,713.4 ns|  1.00 |      32 B |    2 |
```

**Why this output occurs**

- `[GlobalSetup]` allocates the array once, outside the measured method.
- Both methods return a value, preventing dead code elimination.
- `LoopSum` avoids enumerator and delegate allocations.
- `RankColumn` orders fastest to slowest.
- The baseline ratio shows `LoopSum` is about 3× faster.

---

#### Example 2: CI Regression Gate

```bash
dotnet run -c Release --project benchmarks/MyApp.Benchmarks -- --filter "*" --exporters json markdown
```

```yaml
# Conceptual GitHub Actions step
- name: Run benchmarks
  run: |
    dotnet run -c Release --project benchmarks/MyApp.Benchmarks -- --exporters json
    # Compare against baseline JSON and fail on regression
```

**Expected Output (conceptual)**

```
BenchmarkDotNet.Artifacts/results/*.json
BenchmarkDotNet.Artifacts/results/*.md
```

**Why this output occurs**

- `--exporters json markdown` produces machine-readable and human-readable artifacts.
- CI can parse the JSON and compare against a stored baseline.
- A regression fails the build before it reaches production.

---

### Real-World Cases

- **Library performance budgets** — Enforce allocation and latency limits for public APIs.
- **CI performance gates** — Fail the build if a hot-path method regresses.
- **Algorithm selection** — Compare O(n) vs. O(n²) implementations with statistical evidence.
- **Allocation reduction** — Validate that a `Span<T>` refactoring eliminates allocations.

---

### References

- Good Practices — BenchmarkDotNet - https://benchmarkdotnet.org/articles/guides/good-practices.html
- Benchmark authoring techniques — dotnet/skills - https://github.com/dotnet/skills/blob/ac8f41264bdd557e58924a3110eea8e0917dcf4d/plugins/dotnet-diag/skills/microbenchmarking/references/writing-benchmarks.md
- Setup And Cleanup — BenchmarkDotNet - https://benchmarkdotnet.org/articles/features/setup-and-cleanup.html
- Benchmark and Job Baselines — BenchmarkDotNet - https://benchmarkdotnet.org/articles/features/baselines.html
- Ad Hoc Microbenchmarking Locally — dotnet/dotnet - https://github.com/dotnet/dotnet/blob/60864f1c9f7bc774d1c191ef6f1630e81607cd62/src/runtime/.github/skills/microbenchmark/SKILL.md
- BenchmarkDotNet exporters CI integration — GitHub - https://github.com/…/benchmark-exporters
- Datadog.Trace.BenchmarkDotNet exporter — NuGet - https://www.nuget.org/packages/Datadog.Trace.BenchmarkDotNet
- Beware .NET Microbenchmark Cold Starts — LinkedIn - https://www.linkedin.com
- Benchmark Project Implementation Plan — zeroalloc.validation - https://github.com/zeroalloc-net/zeroalloc.validation/blob/a6cda35b93123c3ed768742a993235cc1a5b948e/docs/plans/2026-03-16-benchmarks.md

---

## Summary Table: Benchmarking vs. Profiling

| Dimension | Benchmarking | Profiling |
|---|---|---|
| **Question answered** | “How fast is this operation?” | “Where does my application spend time/memory?” |
| **Scope** | Isolated micro-operation | Whole running application |
| **Environment** | Controlled, isolated | Production or representative load |
| **Tool** | BenchmarkDotNet | Visual Studio Profiler, dotTrace, PerfView |
| **Output** | Mean, median, allocation, confidence interval | Hot paths, call trees, allocation sites, thread states |
| **Primary risk** | Measuring something that does not reflect production | Observing without enough samples or under unrealistic load |
| **When to use** | Comparing implementations, setting budgets, validating optimisations | Diagnosing production bottlenecks, finding unknown hot spots |

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| **Debug builds for benchmarking** | Unsafe; results are 10–100× slower and meaningless |
| **Void-returning benchmark methods** | Dead code elimination risk; always return a value unless the method has observable side effects |
| **`[IterationSetup]` in microbenchmarks** | Spoils results; forces `InvocationCount=1` and `UnrollFactor=1`; use only for macrobenchmarks ≥ 100 ms |
| **Manual loops inside benchmarks** | Distort measurements; BDN already handles iteration |
| **`[DisassemblyDiagnoser]` on Linux** | May crash; use `DOTNET_JitDisasm` environment variable instead |
| **Tier 0 (cold start) measurements** | BDN warms up to Tier 1; hand-rolled stopwatch loops report Tier 0 fiction |
| **Extrapolating across environments** | Results are specific to CPU, OS, and .NET version; never assume they generalise |
| **Constant folding** | The JIT may evaluate constant expressions at compile time; use non-constant inputs |
| **Deferred execution** | Returning an unenumerated `IEnumerable` measures nothing; force enumeration |

---

## References (Consolidated)

- Good Practices — BenchmarkDotNet - https://benchmarkdotnet.org/articles/guides/good-practices.html
- Setup And Cleanup — BenchmarkDotNet - https://benchmarkdotnet.org/articles/features/setup-and-cleanup.html
- Benchmark and Job Baselines — BenchmarkDotNet - https://benchmarkdotnet.org/articles/features/baselines.html
- Benchmark authoring techniques — dotnet/skills - https://github.com/dotnet/skills/blob/ac8f41264bdd557e58924a3110eea8e0917dcf4d/plugins/dotnet-diag/skills/microbenchmarking/references/writing-benchmarks.md
- Ad Hoc Microbenchmarking Locally — dotnet/dotnet - https://github.com/dotnet/dotnet/blob/60864f1c9f7bc774d1c191ef6f1630e81607cd62/src/runtime/.github/skills/microbenchmark/SKILL.md
- Benchmark Project Implementation Plan — zeroalloc.validation - https://github.com/zeroalloc-net/zeroalloc.validation/blob/a6cda35b93123c3ed768742a993235cc1a5b948e/docs/plans/2026-03-16-benchmarks.md
- Visual Studio BenchmarkDotNet analysis — Microsoft Learn - https://learn.microsoft.com/en-us/visualstudio/profiling/benchmarkdotnet
- BenchmarkDotNet exporters CI integration — GitHub - https://github.com/…/benchmark-exporters
- Datadog.Trace.BenchmarkDotNet exporter — NuGet - https://www.nuget.org/packages/Datadog.Trace.BenchmarkDotNet
- Comparing benchmark results — GitHub - https://github.com/…/comparing-benchmark-results
- Beware .NET Microbenchmark Cold Starts — LinkedIn - https://www.linkedin.com
- Microbenchmark Design Guidelines — raw.githubusercontent.com