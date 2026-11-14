# Profiling in C# — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Profiling is the systematic practice of measuring a running program's execution characteristics — CPU time, memory allocation, garbage collection behaviour, thread contention, and I/O waits — to identify performance bottlenecks and guide targeted optimisation.

**Technical Definition**
Profiling is a dynamic program analysis technique that instruments or samples a running process to collect runtime data about method execution times, call counts, call stacks, object allocations, GC events, thread states, and blocked time. In .NET, profiling is enabled by the CLR's profiling APIs, EventPipe, Event Tracing for Windows (ETW), and runtime-provided diagnostic events. The collected data is aggregated into snapshots or traces that reveal hot paths (inclusive and exclusive time), allocation hotspots, and concurrency issues. Profiling is distinct from benchmarking (controlled, isolated measurement of specific operations) and from static analysis (which examines code without executing it).

**Beginner-Friendly Explanation**
Profiling is “finding out where your program actually spends its time and memory.” You cannot guess which method is slow or which line allocates too much — profiling tools record what the program really does while it runs. Think of it as a medical check-up for your application: instead of guessing why it feels sluggish, you measure its vital signs and pinpoint the exact problem.

---

### Key Characteristics

- **Measurement-driven** — Profiling answers “where” and “how much,” not “why” in isolation; it must be combined with code understanding.
- **Sampling vs. instrumentation** — Sampling profilers periodically capture call stacks with low overhead; instrumentation profilers record every method entry/exit with higher accuracy but higher overhead.
- **Multi-dimensional** — CPU time, wall-clock time, allocation volume, GC generation frequency, thread contention, and blocked time are distinct signals.
- **Production-safe options exist** — `dotnet-trace`, `dotnet-counters`, and PerfView can be used in production with acceptable overhead when configured carefully.
- **Tool-specific strengths** — Visual Studio Profiler integrates with the IDE; dotTrace excels at timeline and multithreading analysis; PerfView provides the deepest ETW-based tracing on Windows; BenchmarkDotNet isolates micro-benchmarks with statistical rigour.
- **No single tool fits all** — The choice of tool depends on the symptom (high CPU, high allocation, low CPU with slow response), the environment (Windows, Linux, container), and the level of detail required.

---

### Prerequisites

- Working knowledge of C# and the .NET runtime (JIT, GC, thread pool).
- Familiarity with the .NET CLI (`dotnet build`, `dotnet run`, `dotnet-trace`, `dotnet-counters`).
- Understanding of the difference between Debug and Release builds.
- Basic knowledge of how to reproduce a performance issue (a representative workload or scenario).
- Access to the target environment (local machine, container, or production server).

---

### Related Programming Areas

- **Performance fundamentals** — Latency, throughput, resource utilisation, and performance budgets.
- **Garbage collection** — Allocation profiling and GC tuning.
- **Concurrency** — Thread contention, thread pool starvation, and lock analysis.
- **Benchmarking** — BenchmarkDotNet for isolated micro-benchmarks.
- **Diagnostics and observability** — `dotnet-counters`, `dotnet-trace`, `dotnet-gcdump`, and OpenTelemetry.
- **CI/CD** — Performance regression gates using BenchmarkDotNet results.

---

### Core Concepts / Features

1. Profiling (overarching discipline)
2. Visual Studio Profiler
3. dotTrace
4. PerfView
5. BenchmarkDotNet
6. Profiling best practices

Each concept follows the same uniform structure: definitions (core, technical, beginner-friendly), purposes (each beginning with “To”), syntax rules and structure with complete general syntaxes and component breakdowns, multiple annotated code examples with expected outputs, real-world cases, and references.

---

## 1. Profiling (Overarching Discipline)

### Definitions

**Core Definition**
Profiling is the measurement of a running program's resource consumption and execution behaviour to locate performance bottlenecks and guide optimisation.

**Technical Definition**
Profiling in .NET is supported by multiple runtime mechanisms. Sampling profilers use the CLR's profiling API or EventPipe to periodically capture managed call stacks at millisecond intervals; instrumentation profilers inject probes at method entry and exit to record exact call counts and timings. ETW providers (on Windows) expose runtime, GC, JIT, thread pool, and ASP.NET Core events. `dotnet-trace` and `dotnet-counters` provide cross-platform diagnostics via EventPipe. The profiler aggregates samples into call trees, where **inclusive time** is the total time spent in a method and its callees, and **exclusive time** is the time spent in the method body alone. The measure-first methodology — establish a baseline, profile, identify the bottleneck, optimise, and re-measure — is the foundational principle of performance engineering.

**Beginner-Friendly Explanation**
Profiling is “using a stopwatch and a memory meter on your running program.” Instead of reading code and guessing, you watch the program work and record exactly where it spends its time and memory. The rule is simple: **measure first, optimise second**.

---

### Purposes

- To identify CPU hot spots (methods consuming the most processor time).
- To detect allocation-heavy code paths that drive garbage collection pressure.
- To diagnose thread pool starvation and lock contention.
- To reveal I/O waits and blocked time that inflate wall-clock latency.
- To establish a baseline before and after optimisation.
- To validate that a change actually improves performance.
- To prioritise engineering effort on the code that matters most.

---

### Syntax Rules and Structure

#### General Profiling Workflow

```
1. Define the symptom (high CPU, high latency, memory growth, low throughput).
2. Choose the tool based on the symptom and environment.
3. Reproduce the issue under a representative workload.
4. Collect a trace or snapshot (sampling or instrumentation).
5. Analyse inclusive/exclusive time, allocation counts, and thread states.
6. Identify the bottleneck and form a hypothesis.
7. Apply a targeted optimisation.
8. Re-measure and compare against the baseline.
```

**Component breakdown**

- **Symptom** — The observed behaviour that needs explanation.
- **Tool** — Matched to the symptom (CPU sampling, allocation tracing, ThreadTime, or micro-benchmarking).
- **Workload** — The scenario that reliably reproduces the issue.
- **Trace/Snapshot** — The recorded data.
- **Analysis** — Reading call trees, hot paths, and allocation sites.
- **Baseline** — The pre-optimisation measurement.

#### Syntax Rules

- Always profile a **Release** build without the debugger attached.
- Collect a sufficient sample size (at least 1,000 samples, ideally 5,000, for CPU sampling).
- Profile under realistic load, not idle or synthetic micro-loads.
- Separate CPU-bound problems from blocking/IO-bound problems before choosing a tool.
- Use `dotnet-trace` as a first cross-platform capture, then escalate to PerfView for deep ETW analysis.

#### Constraints and Limitations

- Sampling profilers cannot measure exact call counts; instrumentation profilers add overhead.
- ETW-based tools are Windows-only; `dotnet-trace` is cross-platform.
- Profiling in production must account for sampling overhead and privacy.
- Micro-benchmarks may not reflect production behaviour under contention.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: A Minimal Profiling Target

```csharp
using System.Diagnostics;

public class ProfilingTarget
{
    public void Process(int[] data)
    {
        var sw = Stopwatch.StartNew();

        // Simulated hot path: unnecessary allocation
        var results = new List<string>();
        foreach (var item in data)
        {
            results.Add(item.ToString().Substring(0, 1));
        }

        sw.Stop();
        Console.WriteLine($"Processing took {sw.ElapsedMilliseconds} ms");
    }
}
```

**Expected Output (before optimisation)**

```
Processing took 1240 ms
```

**Why this output occurs**

- `ToString().Substring(0,1)` allocates two strings per element.
- The `List<string>` grows and causes additional allocations.
- A profiler would show high allocation counts in `Process`.

---

### Real-World Cases

- **High-CPU API endpoint** — A CPU sampling trace reveals a regex or LINQ query consuming 60% of processor time.
- **Memory growth in a background service** — An allocation trace shows a growing cache without eviction.
- **Slow responses with low CPU** — A ThreadTime trace reveals thread pool starvation caused by sync-over-async.

---

### References

- Isolate a performance issue — Microsoft Learn - https://learn.microsoft.com/en-us/visualstudio/profiling/isolate-performance-issue
- .NET Performance Testing — Compile-N-Run - https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/dotnet/14-net-testing/8-net-performance-testing.mdx
- .NET Profiling Patterns Reference — GitHub - https://github.com/…/dotnet-profiling-patterns
- Do you profile your code when optimizing performance? — SSW - https://www.ssw.com.au/rules/profile-your-code

---

## 2. Visual Studio Profiler

### Definitions

**Core Definition**
The Visual Studio Profiler is the built-in diagnostic suite in Visual Studio that collects CPU usage, memory allocation, .NET counters, and instrumentation data for applications developed and run from the IDE.

**Technical Definition**
The Visual Studio Profiler is accessible via **Debug → Performance Profiler (Alt+F2)** or **Analyze → Performance Profiler**. It provides multiple analysis tools: **.NET Counters** (live runtime metrics), **CPU Usage** (sampling-based CPU hot-path analysis), **.NET Object Allocation Tracking** (allocation call stacks and types), **Instrumentation** (exact call counts and timing), **Database** (query timing), and **Memory Usage** (heap snapshots). It operates on Release builds without the debugger attached and produces detailed reports with caller/callee views and hot-path navigation. The CPU Usage tool uses sampling; the Instrumentation tool uses method-level instrumentation for exact call counts.

**Beginner-Friendly Explanation**
The Visual Studio Profiler is “the profiler already inside your IDE.” You press Alt+F2, choose what you want to measure (CPU, memory, or counters), run your app, and Visual Studio shows you which methods are slow and which lines allocate the most.

---

### Purposes

- To provide a zero-setup profiling experience inside Visual Studio.
- To measure CPU usage with sampling and identify hot methods.
- To track object allocations and their call stacks.
- To monitor .NET runtime counters live during a session.
- To collect exact call counts with instrumentation.
- To diagnose database query performance.
- To integrate profiling into the development workflow.

---

### Syntax Rules and Structure

#### General Workflow

```
1. Set the solution configuration to Release.
2. Open Debug → Performance Profiler (Alt+F2).
3. Select one or more tools:
   - .NET Counters
   - CPU Usage
   - .NET Object Allocation Tracking
   - Instrumentation
   - Database
   - Memory Usage
4. Start the profiling session.
5. Exercise the application workflow that reproduces the issue.
6. Stop collection.
7. Analyse the report: Hot Path, Caller/Callee, Functions view, Allocation view.
```

**Component breakdown**

- **Release configuration** — Debug builds distort performance; always profile Release.
- **.NET Counters** — Live counters (CPU, GC heap, thread pool, allocation rate).
- **CPU Usage** — Sampling-based hot-path analysis.
- **.NET Object Allocation Tracking** — Allocation call stacks and type distribution.
- **Instrumentation** — Exact call counts and timing (higher overhead).
- **Hot Path** — The most expensive call chain.
- **Caller/Callee** — Who calls the method and what it calls.

#### Syntax Rules

- Always use a Release build without the debugger attached (start without debugging).
- Collect a trace long enough to capture the problematic behaviour (not just startup).
- Use `.NET Counters` first to determine whether the problem is high CPU, high allocation, or thread pool starvation.
- Use CPU Usage when CPU is high; use Instrumentation or ThreadTime when CPU is low but response is slow.
- Compare before/after traces to validate improvements.

#### Constraints and Limitations

- The profiler is Windows and Visual Studio–dependent.
- Sampling cannot measure exact call counts.
- Instrumentation adds overhead and may distort timing.
- Profiling startup does not help diagnose a slow button click later in the application lifecycle.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: CPU Usage Hot Path

```csharp
public class OrderCalculator
{
    public decimal Calculate(IEnumerable<Order> orders)
    {
        return orders
            .Where(o => o.IsActive)
            .Select(o => o.Total * 1.2m)
            .Sum();
    }
}
```

**Profiling procedure**

1. Run the app under **CPU Usage** in the Performance Profiler.
2. Execute the checkout workflow that calls `Calculate`.
3. Stop collection and open the **Hot Path** view.

**Expected Output (conceptual CPU report)**

```
Hot Path:
  OrderCalculator.Calculate (inclusive: 62.3%, exclusive: 8.1%)
    → Enumerable.Where (inclusive: 28.4%)
    → Enumerable.Select (inclusive: 18.2%)
    → Enumerable.Sum (inclusive: 7.6%)
```

**Why this output occurs**

- The LINQ pipeline dominates inclusive time.
- The `Where`, `Select`, and `Sum` iterator machinery contributes to CPU time.
- Replacing the LINQ chain with a single `foreach` loop may reduce overhead.

---

#### Example 2: Allocation Tracking

```csharp
public string FormatOrder(Order order)
{
    return $"{order.Id}-{order.CustomerName}-{order.Total:C}";
}
```

**Profiling procedure**

1. Run the app under **.NET Object Allocation Tracking**.
2. Execute the code path that calls `FormatOrder` repeatedly.
3. Open the **Allocation** view and sort by allocation size.

**Expected Output (conceptual allocation report)**

```
Type              | Allocated Bytes | Count | Call Stack
System.String     | 4.2 MB          | 52,000| FormatOrder → string.Format
System.Text.StringBuilder | 1.8 MB   | 52,000| FormatOrder → string.Format
```

**Why this output occurs**

- String interpolation allocates a new string and a `StringBuilder` (or equivalent) per call.
- In a hot loop, this creates significant GC pressure.
- Using `StringBuilder` pooling or `DefaultInterpolatedStringHandler` can reduce allocations.

---

### Real-World Cases

- **ASP.NET Core API debugging** — Use .NET Counters to spot thread pool starvation, then CPU Usage or Instrumentation to find the blocking call.
- **Desktop application freezes** — Use the CPU Usage and Timeline views to identify UI-thread blocking.
- **Database-heavy operations** — Use the Database tool to see query durations and call counts.

---

### References

- Case study: Isolate a performance issue — Microsoft Learn - https://learn.microsoft.com/en-us/visualstudio/profiling/isolate-performance-issue
- Case study: Beginner’s guide to optimising code — Microsoft Learn - https://learn.microsoft.com/en-us/visualstudio/profiling/optimize-code-using-profiling-tools
- Measure Twice, Optimize Once: Real-World .NET Profiling with Visual Studio — VSLive - https://vslive.com
- Part 1.1: Profiling Tools for .NET Performance Engineers — Medium - https://medium.com
- Visual Studio Profiler — CPU Usage deep dive — Medium - https://medium.com

---

## 3. dotTrace

### Definitions

**Core Definition**
dotTrace is JetBrains’ performance profiler for .NET and .NET Core applications, offering sampling, tracing, line-by-line, and timeline profiling modes with deep multithreading analysis.

**Technical Definition**
dotTrace (part of the JetBrains .NET tools suite, also integrated into Rider) attaches to running .NET processes or launches applications via run configurations. It supports four profiling types: **Sampling** (accurate time measurement, no call counts, low overhead), **Tracing** (accurate call counts, timing may be distorted by overhead), **Line-by-line** (per-line measurement, Windows only, highest overhead), and **Timeline** (sampling plus ETW-based temporal data about thread states, events, and multithreading behaviour). Timeline profiling is recommended for analysing UI freezes, excessive GC, uneven workload distribution, and I/O insufficiency. dotTrace can profile .NET Framework, .NET Core, and .NET 5+ processes on Windows, Linux, and macOS.

**Beginner-Friendly Explanation**
dotTrace is “the profiler that shows you not just which method is slow, but when and on which thread it ran.” It is especially good at revealing thread contention, UI freezes, and garbage collection pauses across time.

---

### Purposes

- To evaluate overall application performance with low-overhead sampling.
- To measure exact call counts with tracing.
- To analyse multithreaded behaviour with timeline data.
- To identify UI freezes, GC pauses, and I/O bottlenecks.
- To profile applications on Windows, Linux, and macOS.
- To integrate profiling into Rider and Visual Studio workflows.

---

### Syntax Rules and Structure

#### General Workflow (Standalone)

```
1. Launch dotTrace.
2. Choose “New Process Run” and add a run configuration (path to executable, arguments, working directory, environment variables).
3. Select a profiling type:
   - Sampling (recommended for first investigation)
   - Timeline (recommended for multithreaded analysis)
   - Tracing (when call counts matter)
   - Line-by-line (advanced, targeted methods)
4. Start the session.
5. Exercise the application workflow.
6. Collect snapshots using “Get Snapshot.”
7. Analyse in dotTrace Viewer: Hot Spots, Call Tree, Events, Threads.
```

#### General Workflow (Attach to Running Process)

```
1. Run | Profile running process.
2. Select a managed .NET process.
3. Choose a profiling configuration.
4. Collect snapshots.
```

**Component breakdown**

- **Run configuration** — Tells dotTrace how to launch the application.
- **Profiling type** — Controls data collection strategy and overhead.
- **Snapshot** — A recorded profiling session.
- **Hot Spots** — The slowest methods (exclusive/inclusive time).
- **Call Tree** — The hierarchical call structure.
- **Events** — GC, JIT, exceptions, and other runtime events.
- **Threads** — Thread states and timeline distribution.

#### Syntax Rules

- Choose **Sampling** for a first investigation; it has the lowest overhead and accurate time measurement.
- Choose **Timeline** for multithreaded applications, UI freezes, and GC analysis.
- Choose **Tracing** when exact call counts are required (e.g., algorithm complexity analysis).
- Choose **Line-by-line** only for targeted methods; overhead is high.
- On Windows, Timeline profiling requires the JetBrains ETW Host Service with administrative privileges.

#### Constraints and Limitations

- Sampling does not measure call counts.
- Tracing timing may be inaccurate due to overhead.
- Line-by-line is Windows-only and has the highest overhead.
- Timeline on Windows requires ETW and elevated permissions.
- Profiling adds overhead; do not enable every mode simultaneously.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Sampling for Hot Spots

```csharp
public class ReportGenerator
{
    public string Generate(IEnumerable<Sale> sales)
    {
        var grouped = sales.GroupBy(s => s.Region);
        var builder = new StringBuilder();
        foreach (var group in grouped)
        {
            builder.AppendLine($"{group.Key}: {group.Sum(s => s.Amount):C}");
        }
        return builder.ToString();
    }
}
```

**Profiling procedure**

1. Launch dotTrace with a **Sampling** run configuration.
2. Execute the report generation path.
3. Collect a snapshot and open **Hot Spots**.

**Expected Output (conceptual)**

```
Hot Spots:
  ReportGenerator.Generate (exclusive: 12.4%, inclusive: 88.7%)
    → Enumerable.GroupBy (inclusive: 31.2%)
    → Enumerable.Sum (inclusive: 24.8%)
    → StringBuilder.AppendLine (inclusive: 9.1%)
```

**Why this output occurs**

- Grouping and aggregation dominate inclusive time.
- `AppendLine` is relatively cheap; the LINQ operators are the main cost.
- Replacing `GroupBy` + `Sum` with a single dictionary accumulation may reduce CPU time.

---

#### Example 2: Timeline for Thread Contention

```csharp
public class SharedCache
{
    private readonly object _lock = new();
    private readonly Dictionary<int, string> _data = new();

    public string Get(int key)
    {
        lock (_lock)
        {
            return _data.TryGetValue(key, out var value) ? value : "default";
        }
    }
}
```

**Profiling procedure**

1. Launch dotTrace with **Timeline** profiling.
2. Run a workload with multiple threads calling `Get`.
3. Collect a snapshot and open the **Threads** and **Events** views.

**Expected Output (conceptual timeline report)**

```
Timeline:
  Thread 1: [Running] [Blocked: 45 ms] [Running] [Blocked: 62 ms]
  Thread 2: [Blocked: 58 ms] [Running] ...
  Events: Monitor.Enter contention on SharedCache._lock (total blocked time: 165 ms)
```

**Why this output occurs**

- Multiple threads contend on the same lock.
- The timeline shows blocked time and context switches.
- Reducing lock duration or using `ConcurrentDictionary` or a reader-writer lock may improve concurrency.

---

### Real-World Cases

- **Multithreaded server applications** — Timeline profiling reveals thread pool starvation, lock contention, and uneven workload distribution.
- **Desktop UI applications** — Identify UI-thread blocking and GC pauses that cause freezes.
- **Algorithm complexity analysis** — Tracing provides exact call counts to validate O(n) vs. O(n²) behaviour.

---

### References

- Configure Profiling Session — dotTrace Help - https://www.jetbrains.com/help/profiler/Profiler_Options.html
- Profile .NET Core or .NET Application — dotTrace Help - https://www.jetbrains.com/help/profiler/Profile_.NET_Core_Application.html
- Basics. Profiling Types — dotTrace - https://www.jetbrains.com/help/profiler/Basic_Concepts.html
- JetBrains Rider 2026.3 Help — Profile running process - https://www.jetbrains.com/help/rider
- Do you profile your code when optimizing performance? — SSW - https://www.ssw.com.au/rules/profile-your-code

---

## 4. PerfView

### Definitions

**Core Definition**
PerfView is a free, Windows-only performance analysis tool from Microsoft that uses Event Tracing for Windows (ETW) to collect and analyse CPU samples, managed allocations, GC events, thread time, and a wide range of runtime and OS events.

**Technical Definition**
PerfView is built on ETW and the CLR's event providers. It collects traces using configurable buffers and circular logging, then presents them in stack-based views (CPU Stacks, Thread Time Stacks, GC Heap Stacks, Allocation Stacks). **CPU sampling** defaults to one sample per millisecond per processor, and each sample approximates 1 ms of CPU time. **ThreadTime** collection adds context-switch events, revealing where threads were **blocked** (waiting on locks, I/O, or thread pool availability) in addition to where they consumed CPU. PerfView is the richest diagnostic tool for .NET on Windows and is capable of analysing native code, managed code, GC behaviour, JIT events, and ASP.NET Core request processing.

**Beginner-Friendly Explanation**
PerfView is “the deepest .NET profiler on Windows.” It is free, extremely detailed, and can tell you not only where CPU time went but also where threads were stuck waiting — which is crucial when the CPU is idle but the application is slow. It is more complex than other tools but gives you the most information.

---

### Purposes

- To analyse CPU consumption with high-resolution sampling.
- To reveal blocked time and thread contention with ThreadTime collection.
- To inspect GC behaviour, allocations, and heap growth.
- To analyse native and managed code together.
- To diagnose ASP.NET Core request latency and Kestrel behaviour.
- To capture long-running reproductions with circular buffers and stop triggers.

---

### Syntax Rules and Structure

#### General Collection Commands

```powershell
# CPU trace (default providers)
PerfView collect /BufferSizeMB:1024 /CircularMB:2048

# ThreadTime trace (CPU + context switches for blocked-time analysis)
PerfView /ThreadTime collect /BufferSizeMB:1024 /CircularMB:2048

# GC and allocation events
PerfView collect /GCCollectOnly /BufferSizeMB:1024 /CircularMB:2048

# Specific providers (e.g., .NET runtime)
PerfView collect /Providers:Microsoft-Windows-DotNETRuntime /BufferSizeMB:1024 /CircularMB:2048

# ASP.NET Core / Kestrel providers
PerfView /ThreadTime collect /Providers:*Microsoft.AspNetCore.Hosting,*Microsoft-AspNetCore-Server-Kestrel /BufferSizeMB:1024 /CircularMB:2048

# Stop on CPU spike (trigger)
PerfView collect /StopOnPerfCounter:"Processor:% Processor Time:_Total>80" /BufferSizeMB:2048 /CircularMB:4096

# Stop on Gen2 GC
PerfView collect /StopOnGCEvent /BufferSizeMB:2048 /CircularMB:4096
```

**Component breakdown**

- `/BufferSizeMB` and `/CircularMB` — Buffer sizes for trace collection; always include for short traces.
- `/ThreadTime` — Adds context-switch events for blocked-time analysis.
- `/GCCollectOnly` — Collects only GC and allocation events.
- `/Providers:...` — Selects specific ETW providers.
- `/StopOn...` — Trigger conditions for long-running reproductions with circular buffers.
- **CPU Stacks** — Where CPU time was spent.
- **Thread Time Stacks** — CPU time **plus** blocked time.
- **GC Heap Stacks** — Allocation call stacks.

#### Syntax Rules

- Always include `/BufferSizeMB:1024 /CircularMB:2048` for short traces.
- Use **CPU Stacks** for CPU-bound problems; use **Thread Time Stacks** for low-CPU-but-slow problems.
- CPU sampling defaults to ~1 ms per processor; collect at least 1,000 samples (ideally 5,000) before drawing conclusions.
- Use `/StopOn` triggers with circular buffers for long-running reproductions; the trigger should fire on the **symptom**, not the recovery.
- Omit `/StartOn` unless the start event is certain to precede the stop event.

#### Constraints and Limitations

- PerfView is **Windows-only**.
- Requires administrative privileges.
- ETW collection can generate large trace files; circular buffers mitigate this.
- CPU sampling does not reveal blocked time; ThreadTime is required for that.
- Interpreting PerfView’s stack views requires practice and familiarity with inclusive/exclusive metrics.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: CPU Stacks for a CPU-Bound Method

```csharp
public class PrimeCalculator
{
    public bool IsPrime(int n)
    {
        if (n < 2) return false;
        for (int i = 2; i <= Math.Sqrt(n); i++)
            if (n % i == 0) return false;
        return true;
    }
}
```

**Profiling procedure**

1. Run `PerfView collect` and execute the prime calculation loop.
2. Open the trace and navigate to **CPU Stacks**.
3. Sort by **Exc** (exclusive) time.

**Expected Output (conceptual CPU Stacks view)**

```
Name                          Exc %   Exc   Inc %   Inc
PrimeCalculator.IsPrime       72.4%   3,620 72.4%   3,620
  → System.Math.Sqrt          8.1%     405   8.1%     405
  → Program.Main               5.2%     260  99.8%   4,990
```

**Why this output occurs**

- `IsPrime` consumes the majority of CPU time.
- `Math.Sqrt` is called repeatedly inside the loop.
- Hoisting `Math.Sqrt(n)` outside the loop would reduce CPU time.

---

#### Example 2: ThreadTime for a Low-CPU, Slow-Response Problem

```csharp
public class SlowService
{
    public string GetData()
    {
        using var client = new HttpClient();
        return client.GetStringAsync("https://example.com").Result; // sync-over-async
    }
}
```

**Profiling procedure**

1. Run `PerfView /ThreadTime collect` and reproduce the slow response.
2. Open **Thread Time Stacks** and examine **BLOCKED_TIME**.

**Expected Output (conceptual ThreadTime view)**

```
Name                                   Exc %   Exc   Inc %   Inc
BLOCKED_TIME                           68.2%   12,400 68.2%  12,400
  → System.Threading.Tasks.Task.Wait    42.1%   7,650 42.1%   7,650
  → SlowService.GetData                 18.4%   3,340 98.7%  17,940
```

**Why this output occurs**

- The `.Result` call blocks the thread on a `Task`, inflating wall-clock time without consuming CPU.
- CPU is low, but response time is high.
- Replacing `.Result` with `await` resolves the problem.

---

### Real-World Cases

- **ASP.NET Core request latency** — ThreadTime collection with Kestrel providers reveals where requests spend blocked time.
- **GC pressure in production** — `GCCollectOnly` collection identifies allocation-heavy paths.
- **High-CPU batch processing** — CPU Stacks reveal the hottest methods.

---

### References

- PerfView — Microsoft GitHub Releases - https://github.com/microsoft/perfview/releases
- Pinpointing “Slow” with PerfView and dotnet-trace — Zenodo - https://zenodo.org
- PerfView reference — dotnet/skills - https://github.com/dotnet/skills
- PerfView — Cratis reference - https://github.com/cratis/applicationmodel

---

## 5. BenchmarkDotNet

### Definitions

**Core Definition**
BenchmarkDotNet is a .NET library that transforms ordinary methods into rigorous, statistically validated benchmarks, reporting latency, throughput, and memory allocations with confidence intervals.

**Technical Definition**
BenchmarkDotNet generates an isolated project for each benchmark, runs it through a warm-up, pilot, and measurement phase, and reports mean, median, standard deviation, and percentiles for execution time and allocations. It supports multiple runtimes (`[SimpleJob(RuntimeMoniker.Net80)]`), parameterisation (`[Params]`), setup and cleanup lifecycles (`[GlobalSetup]`, `[IterationSetup]`), memory diagnostics (`[MemoryDiagnoser]`), and ranking (`[RankColumn]`). It is the de facto standard for micro-benchmarking in .NET and integrates with CI for performance budget enforcement.

**Beginner-Friendly Explanation**
BenchmarkDotNet is “the scientific method for micro-benchmarks.” If you want to know whether `StringBuilder` is faster than string concatenation, or whether `Span<T>` parsing beats `Substring`, BenchmarkDotNet runs the comparison many times, accounts for warm-up and noise, and tells you with statistical confidence.

---

### Purposes

- To measure the performance of small code units with statistical rigour.
- To compare alternative implementations fairly.
- To track memory allocations per operation.
- To detect performance regressions across runtime versions.
- To enforce performance budgets in CI.
- To provide a reproducible, isolated benchmarking environment.

---

### Syntax Rules and Structure

#### General Benchmark Structure

```csharp
using BenchmarkDotNet.Attributes;
using BenchmarkDotNet.Running;

[MemoryDiagnoser]
[SimpleJob(RuntimeMoniker.Net80)]
[RankColumn]
public class StringBenchmarks
{
    private const string Input = "This is a test string for benchmark";
    private int[] _data;

    [GlobalSetup]
    public void Setup()
    {
        _data = new int[1000];
        for (int i = 0; i < _data.Length; i++) _data[i] = i;
    }

    [Benchmark]
    public string Substring() => Input.Substring(0, 10);

    [Benchmark]
    public ReadOnlySpan<char> SpanSlice() => Input.AsSpan(0, 10);
}

public class Program
{
    public static void Main(string[] args) =>
        BenchmarkSwitcher.FromAssembly(typeof(Program).Assembly).Run(args);
}
```

**Component breakdown**

- `[MemoryDiagnoser]` — Reports allocated bytes and GC collections.
- `[SimpleJob(RuntimeMoniker.Net80)]` — Targets .NET 8.
- `[RankColumn]` — Adds a ranking column to the results.
- `[GlobalSetup]` — Runs once per benchmark method before all invocations; used for expensive setup that should not be measured.
- `[Benchmark]` — Marks a method for measurement.
- `BenchmarkSwitcher` — Entry point that discovers and runs benchmarks.

#### Syntax Rules

- Move expensive setup to `[GlobalSetup]`; never include it in the measured method.
- Avoid I/O, networking, logging, or side effects inside `[Benchmark]` methods.
- Use `[Params]` for simple parameterisation; avoid complex global configurations.
- Use `[IterationSetup]` only for macro-benchmarks (≥ 100 ms); it forces `UnrollFactor=1` and `InvocationCount=1` and can distort micro-benchmarks.
- Always run benchmarks in Release mode.
- Use `BenchmarkSwitcher.FromAssembly(...).Run(args)` to allow command-line selection.

#### Constraints and Limitations

- Micro-benchmarks may not reflect production behaviour under contention or realistic workloads.
- `[IterationSetup]` adds significant overhead and is not recommended for micro-benchmarks.
- BenchmarkDotNet generates and compiles a separate project, which adds build time.
- Statistical results require a sufficient number of iterations; very fast methods need many invocations.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: String Allocation Benchmark

```csharp
[MemoryDiagnoser]
[RankColumn]
public class StringAllocationBenchmarks
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

- `Substring` allocates a new 32-byte string.
- `SpanSlice` returns a view with zero allocation.
- `Ratio` compares against the baseline (`Substring`).
- `Rank` orders fastest to slowest.

---

#### Example 2: Parameterised Benchmark with GlobalSetup

```csharp
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
- `LinqSum` incurs enumerator and delegate overhead.

---

### Real-World Cases

- **Library API comparisons** — Compare `Span<T>` vs. `Substring` vs. `StringBuilder`.
- **Serialization benchmarks** — Compare `System.Text.Json` vs. Newtonsoft.Json.
- **Collection choice** — Compare `Dictionary` vs. `ConcurrentDictionary` vs. `ImmutableDictionary`.
- **CI performance gates** — Fail the build if allocation or latency regressions exceed budgets.

---

### References

- BenchmarkDotNet Setup and Cleanup — Official Docs - https://benchmarkdotnet.org/articles/features/setup-and-cleanup.html
- BenchmarkDotNet Attributes — DeepWiki - https://deepwiki.com/dotnet/BenchmarkDotNet
- Ad Hoc Performance Benchmarking — GitHub - https://github.com/…/benchmarking
- Benchmark Fixture Prompt (Tuning Benchmarks) — GitHub - https://github.com/…/benchmark-fixture
- Measurement and diagnostics — BenchmarkDotNet - https://github.com/dotnet/BenchmarkDotNet
- Benchmark authoring techniques — BenchmarkDotNet - https://github.com/dotnet/BenchmarkDotNet

---

## 6. Profiling Best Practices

### Definitions

**Core Definition**
Profiling best practices are empirically grounded guidelines for collecting, interpreting, and acting on profiling data in a way that produces reliable insights and avoids wasted optimisation effort.

**Technical Definition**
Best practices span methodology (measure-first, symptom-driven tool selection, baseline comparison), data collection (Release builds, representative workloads, sufficient sample sizes), interpretation (inclusive vs. exclusive time, CPU vs. wall-clock, allocation vs. GC), and organisational discipline (performance budgets, regression gates, avoiding premature optimisation). They are supported by the .NET diagnostics toolchain and by empirical studies of profiling effectiveness.

**Beginner-Friendly Explanation**
Best practices are “the habits that make profiling useful instead of misleading.” They include measuring before guessing, using the right tool for the symptom, profiling Release builds, and always comparing against a baseline.

---

### Purposes

- To avoid optimising code that does not matter.
- To ensure profiling data is accurate and representative.
- To select the right tool for the symptom.
- To interpret results correctly (inclusive vs. exclusive, CPU vs. wall-clock).
- To integrate profiling into a disciplined performance engineering workflow.
- To prevent performance regressions through continuous measurement.

---

### Guidelines and Structure

#### Measure First, Optimise Second

```
✅ Define the symptom and reproduce it under load.
✅ Collect a baseline trace before changing any code.
✅ Profile, identify the bottleneck, form a hypothesis.
✅ Apply one change at a time and re-measure.
❌ Do not optimise based on intuition or code reading alone.
```

- The first three steps of profiling are “measure, measure, measure.”

#### Match the Tool to the Symptom

```
High CPU → CPU sampling (Visual Studio CPU Usage, dotTrace Sampling, PerfView CPU Stacks)
Low CPU, slow response → ThreadTime (PerfView ThreadTime, dotTrace Timeline, Visual Studio Instrumentation)
High allocation / GC → Allocation tracking (Visual Studio .NET Object Allocation, PerfView GCCollectOnly, dotTrace)
Micro-benchmark comparison → BenchmarkDotNet
Production live metrics → dotnet-counters, System.Diagnostics.Metrics
```

- CPU sampling answers “where did the processor go?”
- ThreadTime answers “where were threads blocked?”
- Allocation tracking answers “what is allocating and why?”

#### Profile Release Builds Without the Debugger

```
✅ Configuration = Release
✅ Start without debugging (Ctrl+F5)
❌ Debug builds disable JIT optimisations and distort results.
```

#### Collect Sufficient Samples

- CPU sampling defaults to ~1 ms per processor; collect at least 1,000 samples (ideally 5,000) before drawing conclusions.
- Very short traces may not capture the bottleneck.

#### Interpret Inclusive vs. Exclusive Time Correctly

- **Inclusive time** — Total time in the method **and its callees**. A high inclusive time means the method or something it calls is expensive.
- **Exclusive time** — Time in the method body **alone**. A high exclusive time means the method itself is the hot spot.
- Do not optimise a method with high inclusive but low exclusive time; optimise its callees.

#### Avoid Common Pitfalls

```
❌ Profiling startup when the problem occurs later.
❌ Enabling every tool at once and drowning in overhead.
❌ Optimising a method because it “looks slow” without evidence.
❌ Ignoring allocation when GC pressure is the real bottleneck.
❌ Using sync-over-async and blaming the profiler for “slow” results.
❌ Trusting a single benchmark run without statistical confidence.
```

#### Use Performance Budgets and Regression Gates

- Encode latency and allocation limits in CI.
- Run BenchmarkDotNet in CI and fail the build on regressions.
- Re-baseline intentionally when making justified trade-offs.

#### Combine Profiling with Runtime Metrics

- Use `dotnet-counters` for live CPU, GC, and thread pool monitoring.
- Use `dotnet-trace` for cross-platform EventPipe collection.
- Correlate profiling snapshots with runtime metrics to confirm the bottleneck.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Symptom-Driven Tool Selection

```csharp
// Symptom: high CPU usage in a parsing method.
// Tool: CPU sampling (Visual Studio CPU Usage or dotTrace Sampling).
// Finding: Regex.Match consumes 72% of CPU time.
// Optimisation: Replace regex with compiled regex or span-based parsing.
// Re-measure: CPU time drops by 60%.
```

**Expected Output (conceptual)**

```
Before: 1,200 ms CPU time
After:  480 ms CPU time
```

**Why this output occurs**

- The profiler identified the true hot spot.
- The optimisation targeted the measured bottleneck, not a guessed one.

---

#### Example 2: Baseline Benchmarking with BenchmarkDotNet

```csharp
[MemoryDiagnoser]
[RankColumn]
public class BaselineBenchmarks
{
    private int[] _data = Enumerable.Range(0, 10000).ToArray();

    [Benchmark(Baseline = true)]
    public int LinqWhereSum() => _data.Where(x => x % 2 == 0).Sum();

    [Benchmark]
    public int LoopSum()
    {
        int sum = 0;
        foreach (var x in _data)
            if (x % 2 == 0) sum += x;
        return sum;
    }
}
```

**Expected Output (conceptual)**

```
| Method        | Mean       | Ratio | Allocated | Rank |
|-------------- |-----------:|------:|----------:|-----:|
| LoopSum       |  8.42 μs   |  0.31 |       0 B |    1 |
| LinqWhereSum  | 27.13 μs   |  1.00 |      32 B |    2 |
```

**Why this output occurs**

- `LoopSum` avoids enumerator and delegate allocations.
- The ratio shows `LoopSum` is about 3× faster.
- The results are statistically validated and reproducible.

---

### Real-World Cases

- **High-traffic API optimisation** — CPU sampling identifies a LINQ hot path; BenchmarkDotNet validates the replacement.
- **Memory leak investigation** — Allocation tracking reveals a growing cache; GC tuning and eviction policy reduce pressure.
- **Thread pool starvation** — ThreadTime profiling reveals sync-over-async; converting to async resolves the bottleneck.
- **CI performance gates** — BenchmarkDotNet results are parsed and compared against budgets, failing the build on regression.

---

### References

- Isolate a performance issue — Microsoft Learn - https://learn.microsoft.com/en-us/visualstudio/profiling/isolate-performance-issue
- .NET Profiling Patterns Reference — GitHub - https://github.com/…/dotnet-profiling-patterns
- Pinpointing “Slow” with PerfView and dotnet-trace — Zenodo - https://zenodo.org
- Do you profile your code when optimizing performance? — SSW - https://www.ssw.com.au/rules/profile-your-code
- Performance & Profiling — LearnWiseJ - https://learnwisej.com
- BenchmarkDotNet Setup and Cleanup — Official Docs - https://benchmarkdotnet.org/articles/features/setup-and-cleanup.html
- Visual Studio Profiler case study — Microsoft Learn - https://learn.microsoft.com/en-us/visualstudio/profiling/isolate-performance-issue
- dotTrace Configure Profiling Session — JetBrains - https://www.jetbrains.com/help/profiler/Profiler_Options.html

---

## Tool Comparison Summary

| Tool | Platform | Profiling Type | Best For | Overhead | Key Strength |
|---|---|---|---|---|---|
| **Visual Studio Profiler** | Windows (Visual Studio) | Sampling, Instrumentation, Allocation, Counters | IDE-integrated profiling, .NET Counters, CPU Usage, Allocation Tracking | Low–Medium | Zero-setup, integrated workflow |
| **dotTrace** | Windows, Linux, macOS | Sampling, Tracing, Line-by-line, Timeline | Multithreaded analysis, UI freezes, GC pauses, thread contention | Low (sampling) – High (line-by-line) | Timeline and thread-state visualisation |
| **PerfView** | Windows only | ETW-based sampling, ThreadTime, GC, Allocation | Deep CPU and blocked-time analysis, native + managed, ASP.NET Core | Medium | Richest ETW data, ThreadTime blocked-time analysis |
| **BenchmarkDotNet** | Cross-platform | Micro-benchmarking | Comparing implementations, allocation measurement, CI regression gates | Very low (isolated) | Statistical rigour, allocation diagnostics, reproducible |

*Sources: tool documentation and profiling guides*

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| **Profiling Debug builds** | Results are distorted by disabled JIT optimisations; always profile Release. |
| **Sync-over-async (`.Result`, `.Wait()`)** | Causes thread pool starvation; ThreadTime profiling reveals it, but the fix is code-level. |
| **`[IterationSetup]` in micro-benchmarks** | Forces `UnrollFactor=1` and `InvocationCount=1`; can spoil results. Use only for macro-benchmarks ≥ 100 ms. |
| **PerfView on Linux/macOS** | Not supported; use `dotnet-trace` for cross-platform collection. |
| **dotTrace Line-by-line on Linux/macOS** | Windows only; high overhead. |
| **CPU sampling sample size** | Fewer than 1,000 samples may not be representative; collect at least 1,000 (ideally 5,000). |
| **Visual Studio Profiler .NET Counters** | Live counters only; not a substitute for deep CPU or allocation analysis. |
| **BenchmarkDotNet `SimpleJob` without runtime moniker** | Defaults may not match the target runtime; specify `RuntimeMoniker` explicitly. |
| **ETW on Windows** | Requires administrative privileges; container support is limited to Windows containers. |

---

## References (Consolidated)

- Isolate a performance issue — Microsoft Learn - https://learn.microsoft.com/en-us/visualstudio/profiling/isolate-performance-issue
- Case study: Beginner’s guide to optimising code — Microsoft Learn - https://learn.microsoft.com/en-us/visualstudio/profiling/optimize-code-using-profiling-tools
- Measure Twice, Optimize Once — VSLive - https://vslive.com
- Part 1.1: Profiling Tools for .NET Performance Engineers — Medium - https://medium.com
- Visual Studio Profiler — CPU Usage deep dive — Medium - https://medium.com
- Configure Profiling Session — dotTrace Help - https://www.jetbrains.com/help/profiler/Profiler_Options.html
- Profile .NET Core or .NET Application — dotTrace Help - https://www.jetbrains.com/help/profiler/Profile_.NET_Core_Application.html
- Basics. Profiling Types — dotTrace - https://www.jetbrains.com/help/profiler/Basic_Concepts.html
- PerfView — Microsoft GitHub Releases - https://github.com/microsoft/perfview/releases
- Pinpointing “Slow” with PerfView and dotnet-trace — Zenodo - https://zenodo.org
- PerfView reference — dotnet/skills - https://github.com/dotnet/skills
- PerfView — Cratis reference - https://github.com/cratis/applicationmodel
- BenchmarkDotNet Setup and Cleanup — Official Docs - https://benchmarkdotnet.org/articles/features/setup-and-cleanup.html
- BenchmarkDotNet Attributes — DeepWiki - https://deepwiki.com/dotnet/BenchmarkDotNet
- Ad Hoc Performance Benchmarking — GitHub - https://github.com/…/benchmarking
- Benchmark Fixture Prompt (Tuning Benchmarks) — GitHub - https://github.com/…/benchmark-fixture
- Measurement and diagnostics — BenchmarkDotNet - https://github.com/dotnet/BenchmarkDotNet
- Benchmark authoring techniques — BenchmarkDotNet - https://github.com/dotnet/BenchmarkDotNet
- .NET Profiling Patterns Reference — GitHub - https://github.com/…/dotnet-profiling-patterns
- Do you profile your code when optimizing performance? — SSW - https://www.ssw.com.au/rules/profile-your-code
- Performance & Profiling — LearnWiseJ - https://learnwisej.com
- .NET Performance Testing — Compile-N-Run - https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/dotnet/14-net-testing/8-net-performance-testing.mdx