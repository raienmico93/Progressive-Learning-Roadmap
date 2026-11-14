# Performance Fundamentals in C# — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Performance in software is the degree to which a system or component accomplishes its functions within given constraints of speed, resource consumption, and scalability. In C# and .NET, performance encompasses latency, throughput, resource utilization (CPU, memory, I/O), and the efficiency of runtime services such as the garbage collector and thread pool.

**Technical Definition**
Performance is a multi-dimensional quality attribute measured through latency (response time per operation), throughput (operations completed per unit time), resource utilization (CPU, memory, disk I/O, network bandwidth), and runtime-specific metrics such as garbage collection pause time, allocation rate, and thread pool starvation. In .NET, performance is governed by the JIT compiler, the garbage collector (GC), the thread pool, and the design of hot code paths. The .NET runtime exposes these dimensions through `System.Diagnostics.Metrics`, `EventCounters`, and diagnostic tools such as `dotnet-counters`, `dotnet-trace`, and BenchmarkDotNet. Performance must be measured before optimization; the measure-first workflow is a core principle of .NET performance engineering.

**Beginner-Friendly Explanation**
Performance is “how fast and efficiently your application does its job.” It covers how quickly a single request is answered (latency), how many requests can be handled per second (throughput), and how much CPU, memory, and network the application consumes. Good performance means users wait less, servers cost less, and the system scales further. In .NET, performance is heavily influenced by how much memory your code allocates (because the garbage collector must clean it up) and whether your code blocks threads unnecessarily.

---

### Key Characteristics

- **Multi-dimensional** — Latency, throughput, and resource utilization are distinct and often competing objectives.
- **Measurement-driven** — Optimization without profiling is guessing; the measure-first workflow is mandatory.
- **Allocation-sensitive** — In .NET, memory allocation pressure and garbage collection are the dominant performance factors in most applications.
- **Hot-path-sensitive** — Performance is concentrated in a small number of frequently executed code paths.
- **Tail-latency-aware** — Average latency can be misleading; p95, p99, and p99.9 percentiles reveal the experience of real users.
- **Budget-enforced** — Performance budgets turn goals into enforceable CI gates.
- **Tool-supported** — .NET provides first-class tooling: BenchmarkDotNet, `dotnet-counters`, `dotnet-trace`, `dotnet-gcdump`, and the `System.Diagnostics.Metrics` API.

---

### Prerequisites

- Working knowledge of C# classes, methods, collections, `async`/`await`, and LINQ.
- Basic understanding of the .NET runtime: JIT compilation, garbage collection, and the thread pool.
- Familiarity with the .NET CLI (`dotnet build`, `dotnet run`, `dotnet-counters`).
- Awareness of HTTP request/response pipelines (for ASP.NET Core performance).
- Basic understanding of profiling concepts: sampling, allocation tracking, and benchmarking.

---

### Related Programming Areas

- **Garbage collection and memory management** — The GC is the primary performance lever in .NET.
- **Async programming and thread pool** — Sync-over-async and thread pool starvation are leading causes of latency and throughput degradation.
- **Database and data access** — I/O and ORM query design dominate real-world performance.
- **Caching** — Aggressive caching is a core ASP.NET Core performance strategy.
- **Benchmarking and profiling** — BenchmarkDotNet, `dotnet-trace`, and `dotnet-counters` are the primary diagnostic tools.
- **CI/CD and performance regression prevention** — Performance budgets enforced in CI prevent regressions.

---

### Core Concepts / Features

1. Performance (overarching discipline)
2. Latency
3. Throughput
4. Resource Utilization
5. Performance Metrics
6. Performance Budgets
7. Performance Best Practices

Each concept follows the same uniform structure: definitions (core, technical, beginner-friendly), purposes (each beginning with “To”), syntax rules and structure with complete general syntaxes and component breakdowns, multiple annotated code examples with expected outputs, real-world cases, and references.

---

## 1. Performance (Overarching Discipline)

### Definitions

**Core Definition**
Performance is the efficiency with which a software system executes its workload, measured across speed, resource consumption, and scalability dimensions.

**Technical Definition**
In .NET, performance is the composite outcome of JIT compilation quality, garbage collector behavior, thread pool scheduling, I/O efficiency, and application-level design. It is quantified through latency distributions, throughput under load, resource utilization (CPU, memory, disk, network), and runtime diagnostics such as GC pause time and allocation rate. The .NET runtime provides built-in metrics through `System.Diagnostics.Metrics` (OpenTelemetry-compatible) and `EventCounters` (cross-platform) for continuous observation. Performance is not a single number; it is a profile of behaviors under specific workloads and conditions.

**Beginner-Friendly Explanation**
Performance is “how well your application uses the resources it has.” A fast application answers requests quickly, handles many users at once, and does not waste CPU or memory. In .NET, the biggest performance lever is usually memory allocation: every object you create must later be cleaned up by the garbage collector, and that cleanup costs CPU time.

---

### Purposes

- To deliver responsive user experiences.
- To reduce infrastructure cost by using fewer resources.
- To scale to more users without proportional hardware growth.
- To prevent outages caused by resource exhaustion.
- To establish a baseline for detecting regressions.
- To guide architectural and algorithmic decisions.
- To meet service-level objectives (SLOs) and contractual obligations.

---

### Syntax Rules and Structure

#### General Measurement Workflow

```
1. Define the workload and success criteria (latency, throughput, resource bounds).
2. Establish a baseline measurement using BenchmarkDotNet or load testing.
3. Profile the running system with dotnet-counters, dotnet-trace, or PerfView.
4. Identify the bottleneck (GC, thread pool, I/O, CPU, locking).
5. Apply a targeted optimization.
6. Re-measure and compare against the baseline.
7. Encode the result as a performance budget in CI.
```

**Component breakdown**

- **Workload** — The representative scenario (e.g., 1,000 concurrent users, 10,000 requests per second).
- **Baseline** — The current measured performance before optimization.
- **Profile** — Runtime diagnostics that reveal where time and memory are spent.
- **Bottleneck** — The limiting factor (e.g., Gen2 GC, thread pool starvation, database round trips).
- **Budget** — An enforceable threshold that fails CI when exceeded.

#### Syntax Rules

- Measure before optimizing; never optimize based on intuition.
- Use BenchmarkDotNet for micro-benchmarks and `dotnet-trace` for production-like CPU profiles.
- Use `dotnet-counters` for live process stats (CPU, GC, thread pool).
- Isolate the benchmark from debuggers, logging, and background noise.
- Report results with statistical confidence intervals (BenchmarkDotNet default).

#### Constraints and Limitations

- Micro-benchmarks may not reflect production behavior.
- Production profiling must account for sampling overhead and privacy.
- Performance optimizations can reduce readability; apply them only where measurements justify the cost.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Micro-Benchmark with BenchmarkDotNet

```csharp
using BenchmarkDotNet.Attributes;
using BenchmarkDotNet.Running;

public class StringBenchmarks
{
    private const string Input = "This is a test string for benchmark";

    [Benchmark]
    public string Substring() => Input.Substring(0, 10);

    [Benchmark]
    public ReadOnlySpan<char> SpanSlice() => Input.AsSpan(0, 10);
}

public class Program
{
    public static void Main() => BenchmarkRunner.Run<StringBenchmarks>();
}
```

**Line-by-line comments**

- `[Benchmark]` — Marks a method for measurement.
- `Substring` — Allocates a new string on the heap.
- `SpanSlice` — Returns a `ReadOnlySpan<char>` with zero allocation.
- `BenchmarkRunner.Run` — Executes the benchmark with warm-up, iteration, and statistical analysis.

**Expected Output (conceptual)**

```
| Method     | Mean      | Allocated |
|----------- |----------:|----------:|
| Substring  | 12.34 ns  | 32 B      |
| SpanSlice  |  1.23 ns  | 0 B       |
```

**Why this output occurs**

- `Substring` allocates a new string (32 bytes for 10 characters).
- `SpanSlice` returns a view over the existing string without allocation.
- BenchmarkDotNet reports mean latency and allocated bytes per operation.

---

#### Example 2: Live Diagnostics with `dotnet-counters`

```bash
dotnet-counters monitor --process-id 12345 \
  System.Runtime[cpu-usage,gc-heap-size,gen-2-gc-count,threadpool-thread-count]
```

**Expected Output (conceptual)**

```
[System.Runtime]
    cpu-usage                     (%)          12.5
    gc-heap-size                  (MB)         245.3
    gen-2-gc-count                               12
    threadpool-thread-count                      8
```

**Why this output occurs**

- `dotnet-counters` attaches to the running process and reports live runtime metrics.
- `cpu-usage` shows process CPU consumption.
- `gc-heap-size` shows managed heap size.
- `gen-2-gc-count` indicates full GC frequency.
- `threadpool-thread-count` reveals thread pool growth.

---

### Real-World Cases

- **High-frequency trading** — Microsecond latency budgets with p99.99 percentiles.
- **E-commerce platforms** — p95 latency targets during peak traffic.
- **SaaS multi-tenant APIs** — Per-tenant allocation and latency budgets.
- **Background processing** — Throughput-oriented batch pipelines with GC-sensitive workloads.

---

### References

- .NET Performance Testing — Compile-N-Run - https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/dotnet/14-net-testing/8-net-performance-testing.mdx
- .NET Performance Guidelines — xian55/hermesproxy - https://github.com/xian55/hermesproxy/blob/main/.claude/skills/dotnet-performance/SKILL.md
- Migrate from Windows Performance Counters to .NET metrics — Microsoft - https://learn.microsoft.com/en-us/dotnet/core/diagnostics/migrate-from-windows-performance-counters
- performance-optimization-dotnet — skills.rest - https://skills.rest/skill/performance-optimization-dotnet-peterblazejewicz
- ASP.NET Core Best Practices — dotnet/aspnetcore.docs - https://github.com/dotnet/aspnetcore.docs/blob/main/aspnetcore/fundamentals/best-practices.md

---

## 2. Latency

### Definitions

**Core Definition**
Latency is the time elapsed between the initiation of an operation and its completion, as observed by the caller.

**Technical Definition**
Latency in .NET is the end-to-end duration of a request or operation, from the moment it enters the system to the moment a response is produced. It is influenced by CPU scheduling, JIT compilation, garbage collection pauses, thread pool availability, lock contention, I/O waits, and network round trips. Latency is best characterized by percentiles (p50, p95, p99, p99.9) rather than averages, because averages mask tail behavior that dominates user experience. In ASP.NET Core, latency includes middleware execution, model binding, business logic, data access, and serialization.

**Beginner-Friendly Explanation**
Latency is “how long a single request takes.” If you click a button and wait 200 milliseconds for a response, the latency is 200 ms. But averages can lie: if 99% of requests take 10 ms and 1% take 10 seconds, the average looks fine but some users are having a terrible experience. That is why engineers look at percentiles like p95 and p99.

---

### Purposes

- To quantify the user-perceived responsiveness of the system.
- To identify tail-latency problems that affect real users.
- To set and enforce service-level objectives (SLOs).
- To isolate the contribution of each pipeline stage to total response time.
- To detect regressions introduced by new code or configuration.
- To compare infrastructure or architectural choices fairly.

---

### Syntax Rules and Structure

#### General Latency Measurement Syntax

```csharp
// Measure latency with Stopwatch
var stopwatch = Stopwatch.StartNew();
await ProcessRequestAsync();
stopwatch.Stop();
Console.WriteLine($"Latency: {stopwatch.ElapsedMilliseconds} ms");

// Measure with BenchmarkDotNet (statistical)
[Benchmark]
public void Operation() => _service.Execute();

// Measure HTTP latency with percentiles
// (conceptual — use a load-testing tool or metrics library)
```

**Component breakdown**

- `Stopwatch` — High-resolution timing for individual operations.
- `[Benchmark]` — BenchmarkDotNet attribute for repeated, statistically rigorous measurement.
- Percentile reporting — p50 (median), p95, p99, p99.9 to characterize tail latency.

#### Syntax Rules

- Always measure at the boundary the user or caller observes (end-to-end), not only internal methods.
- Report percentiles, not just averages.
- Exclude warm-up iterations from steady-state measurements.
- Use monotonic clocks (`Stopwatch`) rather than wall-clock time for elapsed durations.

#### Constraints and Limitations

- Latency measured in a benchmark may differ from production due to caching, JIT warm-up, and contention.
- Tail latency is sensitive to GC pauses, thread pool starvation, and lock contention.
- Network latency includes factors outside the application’s control.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Percentile Latency Reporting

```csharp
public class LatencyRecorder
{
    private readonly List<long> _samples = new();

    public void Record(long elapsedTicks) => _samples.Add(elapsedTicks);

    public (double p50, double p95, double p99) GetPercentiles()
    {
        _samples.Sort();
        return (
            Percentile(50),
            Percentile(95),
            Percentile(99)
        );

        double Percentile(double p)
        {
            var index = (int)Math.Ceiling(p / 100.0 * _samples.Count) - 1;
            return _samples[Math.Max(0, index)] / (double)Stopwatch.Frequency * 1000;
        }
    }
}
```

**Expected Output (conceptual)**

```
p50 = 12.4 ms
p95 = 48.7 ms
p99 = 210.3 ms
```

**Why this output occurs**

- The recorder collects raw elapsed times.
- Percentiles are computed after sorting.
- p99 is much higher than p50, revealing tail latency.

---

#### Example 2: BenchmarkDotNet Latency Measurement

```csharp
[MemoryDiagnoser]
public class CacheBenchmarks
{
    private readonly Dictionary<int, string> _cache = new()
    {
        [1] = "value"
    };

    [Benchmark]
    public string DictionaryLookup() => _cache[1];
}
```

**Expected Output (conceptual)**

```
| Method            | Mean     | Allocated |
|------------------ |---------:|----------:|
| DictionaryLookup  | 3.21 ns  | 0 B       |
```

**Why this output occurs**

- BenchmarkDotNet measures the mean operation latency with statistical confidence.
- `MemoryDiagnoser` also reports allocations per operation.

---

### Real-World Cases

- **API p95 dashboards** — Track the 95th percentile response time across endpoints.
- **Payment processing** — Enforce a p99 latency budget to avoid abandoned transactions.
- **Real-time gaming** — Measure tick latency and frame-time percentiles.
- **Database query latency** — Separate query time from application time to isolate bottlenecks.

---

### References

- .NET Performance Testing — Compile-N-Run - https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/dotnet/14-net-testing/8-net-performance-testing.mdx
- Performance lens — tail latency and percentiles - https://github.com/CliffYuan/dotnet-skills/blob/main/skills/performance-lens/SKILL.md
- Extreme Performance Tuning for ASP.NET Core Web APIs — alex.rocks - https://alex.rocks/extreme-performance-tuning-aspnet-core-web-apis/

---

## 3. Throughput

### Definitions

**Core Definition**
Throughput is the number of operations a system completes per unit of time.

**Technical Definition**
Throughput in .NET is the rate at which requests, messages, or transactions are processed, typically expressed as requests per second (RPS) or operations per second. It is determined by concurrency (how many operations can be in flight simultaneously), per-operation latency, and resource capacity. Throughput and latency are often in tension: increasing concurrency can improve throughput until contention, GC pressure, or thread pool starvation cause latency to spike. Server garbage collection (`<ServerGarbageCollection>true`) is a key lever for throughput on multi-core servers. Async all-the-way architecture and non-blocking I/O are essential for high throughput in ASP.NET Core.

**Beginner-Friendly Explanation**
Throughput is “how many jobs you can finish per second.” If a server can handle 1,000 requests per second, its throughput is 1,000 RPS. To get more throughput, you can add more cores, use asynchronous code so threads are not stuck waiting, and reduce the amount of memory your code allocates.

---

### Purposes

- To quantify the capacity of the system under load.
- To size infrastructure and plan scaling.
- To compare architectural alternatives (e.g., async vs. sync, pooled vs. non-pooled).
- To identify concurrency bottlenecks (thread pool, locking, connection pools).
- To set throughput-based performance budgets.
- To validate that optimizations increase capacity without degrading latency.

---

### Syntax Rules and Structure

#### General Throughput Measurement Syntax

```csharp
// Conceptual throughput test: run N concurrent operations for a fixed duration
var tasks = Enumerable.Range(0, concurrency)
    .Select(_ => Task.Run(async () =>
    {
        while (!cancellationToken.IsCancellationRequested)
            await _service.ProcessAsync();
    }));

await Task.WhenAll(tasks);
var throughput = completedOperations / elapsedSeconds;
```

**Component breakdown**

- **Concurrency** — Number of simultaneous operations.
- **Completed operations** — Successful units of work.
- **Elapsed seconds** — Measurement window.
- **Throughput** — Completed operations ÷ elapsed seconds.

#### Syntax Rules

- Measure throughput under sustained load, not just a short burst.
- Record latency percentiles alongside throughput; a throughput gain that explodes p99 latency is not a win.
- Use asynchronous APIs for I/O-bound work to free thread pool threads.
- Enable server GC for multi-core throughput workloads.

#### Constraints and Limitations

- Throughput is limited by the slowest resource (CPU, database, network, lock).
- Increasing concurrency beyond the system’s capacity increases latency without improving throughput.
- GC pressure and large object heap allocations can cap throughput.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Throughput Calculation

```csharp
var stopwatch = Stopwatch.StartNew();
var completed = 0;
var cancellation = new CancellationTokenSource(TimeSpan.FromSeconds(10));

await Parallel.ForEachAsync(
    Enumerable.Range(0, 100),
    new ParallelOptions { MaxDegreeOfParallelism = 8, CancellationToken = cancellation.Token },
    async (_, ct) =>
    {
        await _service.ProcessAsync(ct);
        Interlocked.Increment(ref completed);
    });

stopwatch.Stop();
var rps = completed / stopwatch.Elapsed.TotalSeconds;
Console.WriteLine($"Throughput: {rps:F1} ops/sec");
```

**Expected Output (conceptual)**

```
Throughput: 842.3 ops/sec
```

**Why this output occurs**

- `Parallel.ForEachAsync` limits concurrency to 8.
- The loop runs for 10 seconds (cancellation token).
- Throughput is the number of completed operations divided by elapsed time.

---

#### Example 2: Server GC Configuration for Throughput

```xml
<!-- In the .csproj or runtimeconfig.template.json -->
<PropertyGroup>
  <ServerGarbageCollection>true</ServerGarbageCollection>
</PropertyGroup>
```

**Expected Output (conceptual)**

```
Throughput increases on multi-core servers because each core has its own GC heap and collection is parallel.
```

**Why this output occurs**

- Server GC creates per-core heaps and performs parallel collection, improving throughput for multi-threaded workloads.

---

### Real-World Cases

- **API gateways** — Measure RPS under simulated peak load.
- **Message consumers** — Measure messages processed per second.
- **Batch ETL pipelines** — Measure records transformed per second.
- **Real-time bidding** — Throughput and latency must both meet strict SLOs.

---

### References

- .NET Performance Testing — Compile-N-Run - https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/dotnet/14-net-testing/8-net-performance-testing.mdx
- ASP.NET Core Best Practices — async and throughput - https://github.com/dotnet/aspnetcore.docs/blob/main/aspnetcore/fundamentals/best-practices.md
- .NET performance guidelines — Server GC and throughput - https://github.com/xian55/hermesproxy/blob/main/.claude/skills/dotnet-performance/SKILL.md

---

## 4. Resource Utilization

### Definitions

**Core Definition**
Resource utilization is the degree to which a system consumes available resources — CPU, memory, disk I/O, and network — while performing its workload.

**Technical Definition**
Resource utilization in .NET is observed through runtime metrics such as `cpu-usage`, `gc-heap-size`, `gen-2-gc-count`, `threadpool-thread-count`, `alloc-rate`, and `working-set`. High CPU utilization may indicate computation-bound work; high memory utilization with frequent Gen2 GC indicates allocation pressure; thread pool growth indicates blocking; disk and network utilization reveal I/O bottlenecks. Resource utilization must be interpreted alongside latency and throughput: a system that uses 100% CPU at low throughput is wasting cycles on GC, contention, or inefficient algorithms.

**Beginner-Friendly Explanation**
Resource utilization is “how much of your server you are using.” If your CPU is at 100% but you are only handling 50 requests per second, something is wrong — probably too much memory allocation or too much blocking. If your memory keeps growing, you may have a leak.

---

### Purposes

- To identify wasteful consumption (CPU, memory, I/O).
- To right-size infrastructure and reduce cloud costs.
- To detect memory leaks and GC pressure.
- To reveal thread pool starvation and blocking.
- To correlate resource spikes with latency regressions.
- To validate that optimizations actually reduce consumption.

---

### Syntax Rules and Structure

#### General Resource Monitoring Syntax

```bash
# Live runtime metrics
dotnet-counters monitor --process-id 12345 \
  System.Runtime[cpu-usage,gc-heap-size,gen-2-gc-count,threadpool-thread-count,alloc-rate]

# CPU sampling trace
dotnet-trace collect --process-id 12345 --providers Microsoft-DotNETCore-SampleProfiler

# Heap dump for memory analysis
dotnet-gcdump collect --process-id 12345
```

**Component breakdown**

- `dotnet-counters` — Live monitoring of runtime counters.
- `dotnet-trace` — CPU sampling and event tracing.
- `dotnet-gcdump` — Heap snapshot for memory diagnosis.

#### Syntax Rules

- Monitor resource utilization under representative load, not idle.
- Correlate resource spikes with latency and throughput changes.
- Use `System.Diagnostics.Metrics` (OpenTelemetry-compatible) for production-grade continuous telemetry.
- Set alerts on sustained high utilization and Gen2 GC frequency.

#### Constraints and Limitations

- High utilization is not always bad; it may indicate efficient use of provisioned capacity.
- Container limits and cgroup quotas can distort host-level metrics.
- Counter collection itself has minimal but non-zero overhead.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Live Runtime Counters

```bash
dotnet-counters monitor --process-id 12345 System.Runtime
```

**Expected Output (conceptual)**

```
[System.Runtime]
    cpu-usage                     (%)          45.2
    gc-heap-size                  (MB)         512.7
    gen-0-gc-count                              1240
    gen-1-gc-count                               86
    gen-2-gc-count                               12
    threadpool-thread-count                      16
    alloc-rate                    (B/sec)      12,450,000
```

**Why this output occurs**

- `cpu-usage` shows process CPU consumption.
- `gc-heap-size` shows managed heap size.
- `gen-2-gc-count` reveals full GC frequency.
- `threadpool-thread-count` reveals thread pool size.
- `alloc-rate` shows bytes allocated per second.

---

#### Example 2: Correlating GC with Latency

```csharp
// Use dotnet-trace to capture a profile, then analyze GC pauses in PerfView.
// Look for "GC" events and their duration relative to request latency.
```

**Expected Output (conceptual)**

```
GC pause events: 12 in 10 seconds, max pause 45 ms.
Request p99 latency: 210 ms, coinciding with GC pauses.
```

**Why this output occurs**

- Gen2 GC pauses can directly contribute to tail latency.
- Correlating GC events with latency percentiles reveals the impact.

---

### Real-World Cases

- **Cloud cost optimization** — Reduce CPU and memory consumption to downsize instances.
- **Memory leak detection** — Growing `gc-heap-size` with flat load indicates a leak.
- **Thread pool starvation** — Growing `threadpool-thread-count` with high latency indicates blocking.
- **GC tuning** — Adjust server GC, LOH compaction, and allocation patterns.

---

### References

- Migrate from Windows Performance Counters to .NET metrics — Microsoft - https://learn.microsoft.com/en-us/dotnet/core/diagnostics/migrate-from-windows-performance-counters
- .NET runtime metrics — GitHub (dotnet/docs) - https://github.com/dotnet/docs/blob/main/docs/core/diagnostics/metrics-built-in.md
- EventCounters in .NET — Microsoft - https://learn.microsoft.com/en-us/dotnet/core/diagnostics/event-counters
- ASP.NET Core Best Practices — minimize large allocations - https://github.com/dotnet/aspnetcore.docs/blob/main/aspnetcore/fundamentals/best-practices.md

---

## 5. Performance Metrics

### Definitions

**Core Definition**
Performance metrics are quantifiable measures used to evaluate the speed, efficiency, and stability of a system.

**Technical Definition**
In .NET, performance metrics are emitted through `System.Diagnostics.Metrics` (the modern, OpenTelemetry-compatible API) and `EventCounters` (the cross-platform counter API). Key metrics include CPU usage, GC heap size, GC collection counts by generation, allocation rate, thread pool thread count, request rate, error rate, and latency histograms. ASP.NET Core exposes built-in metrics for request duration, active requests, and Kestrel connection counts. These metrics are consumed by monitoring systems (Prometheus, Application Insights, OpenTelemetry collectors) and visualized in dashboards.

**Beginner-Friendly Explanation**
Performance metrics are “the numbers you watch to know if your app is healthy.” They include CPU usage, memory usage, garbage collection counts, and how long requests take. .NET has built-in metrics that you can collect and send to monitoring tools.

---

### Purposes

- To provide objective, continuous visibility into system health.
- To detect regressions before users report them.
- To trigger alerts when metrics exceed thresholds.
- To correlate application behavior with infrastructure changes.
- To support capacity planning and autoscaling decisions.
- To satisfy observability and compliance requirements.

---

### Syntax Rules and Structure

#### General Metrics API Syntax (`System.Diagnostics.Metrics`)

```csharp
using System.Diagnostics.Metrics;

var meter = new Meter("MyApp.Orders");
var orderCounter = meter.CreateCounter<int>("orders.placed");
var latencyHistogram = meter.CreateHistogram<double>("orders.latency", "ms");

orderCounter.Add(1);
latencyHistogram.Record(elapsedMs);
```

**Component breakdown**

- `Meter` — The source of metrics.
- `Counter<T>` — Monotonically increasing count.
- `Histogram<T>` — Distribution of values (e.g., latency).
- `Add` / `Record` — Emit metric values.

#### Built-in Metrics Collection

```bash
# dotnet-counters (live)
dotnet-counters monitor --process-id 12345

# OpenTelemetry exporter (conceptual configuration)
builder.Services.AddOpenTelemetry()
    .WithMetrics(m => m.AddAspNetCoreInstrumentation().AddRuntimeInstrumentation());
```

#### Syntax Rules

- Use `Counter` for totals (requests, errors).
- Use `Histogram` for distributions (latency).
- Use `UpDownCounter` for values that can increase or decrease (active requests).
- Avoid high-cardinality tags (user IDs, request IDs) in metrics; use traces for that.
- Export metrics to a monitoring backend for retention and alerting.

#### Constraints and Limitations

- Metrics aggregation loses per-request detail; use distributed tracing for individual requests.
- Histogram bucket configuration affects accuracy and memory.
- High-cardinality dimensions can overwhelm monitoring systems.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Custom Counter and Histogram

```csharp
public class OrderService
{
    private static readonly Meter Meter = new("MyApp.Orders");
    private static readonly Counter<int> OrdersPlaced = Meter.CreateCounter<int>("orders.placed");
    private static readonly Histogram<double> OrderLatency = Meter.CreateHistogram<double>("orders.latency", "ms");

    public async Task PlaceOrderAsync(Order order)
    {
        var sw = Stopwatch.StartNew();
        // ... business logic ...
        sw.Stop();

        OrdersPlaced.Add(1);
        OrderLatency.Record(sw.Elapsed.TotalMilliseconds);
    }
}
```

**Expected Output (conceptual metrics export)**

```
orders.placed{service=order} 1542
orders.latency{service=order} histogram bucket counts
```

**Why this output occurs**

- Each order increments the counter and records a latency sample.
- The metrics are exported to the configured backend.

---

#### Example 2: ASP.NET Core Built-in Metrics

```bash
dotnet-counters monitor --process-id 12345 \
  Microsoft.AspNetCore.Hosting[requests-per-second,request-duration]
```

**Expected Output (conceptual)**

```
[Microsoft.AspNetCore.Hosting]
    requests-per-second                     842.1
    request-duration (ms) p95               48.2
```

**Why this output occurs**

- ASP.NET Core emits built-in request metrics.
- `dotnet-counters` displays them live.

---

### Real-World Cases

- **SLO dashboards** — p95 latency and error rate per endpoint.
- **Autoscaling triggers** — CPU and request rate thresholds.
- **Capacity planning** — Throughput and resource utilization trends.
- **Incident response** — Correlating GC pauses with latency spikes.

---

### References

- Migrate from Windows Performance Counters to .NET metrics — Microsoft - https://learn.microsoft.com/en-us/dotnet/core/diagnostics/migrate-from-windows-performance-counters
- ASP.NET Core built-in metrics — dotnet/docs - https://github.com/dotnet/docs/blob/main/docs/core/diagnostics/metrics-built-in.md
- .NET runtime metrics — dotnet/docs - https://github.com/dotnet/docs/blob/main/docs/core/diagnostics/metrics-built-in.md
- EventCounters in .NET — Microsoft - https://learn.microsoft.com/en-us/dotnet/core/diagnostics/event-counters

---

## 6. Performance Budgets

### Definitions

**Core Definition**
A performance budget is a predefined threshold that defines the maximum acceptable latency, allocation, or resource consumption for an operation, and that fails a build or alerts when exceeded.

**Technical Definition**
A performance budget is an enforceable performance contract encoded in CI. It typically specifies mean or percentile latency limits, allocation limits (bytes per operation), and GC generation rates. Budgets are established from a measured baseline, committed to the repository, and reviewed like code changes. A CI step parses BenchmarkDotNet JSON results and fails the build if any budget is violated, preventing performance regressions from reaching production.

**Beginner-Friendly Explanation**
A performance budget is “a speed limit for your code.” You say “this method must take no more than 50 nanoseconds and allocate zero bytes.” If a change makes it slower or more allocation-heavy, the build fails. This stops performance from slowly degrading over time.

---

### Purposes

- To prevent performance regressions from merging unnoticed.
- To make performance expectations explicit and reviewable.
- To provide early warning of degradation.
- To guide optimization work with concrete targets.
- To hold teams accountable for performance quality.
- To integrate performance into the definition of “done.”

---

### Syntax Rules and Structure

#### General Budget Definition

```
Operation: TenantId.Create(Guid)
Max Allowed Latency (Mean): ≤ 3.5 ns
Max Allowed Allocation: 0 B
Target Metric: Instant struct value wrapping
```

```
Operation: TenantId.TryCreate(string, out id)
Max Allowed Latency (Mean): ≤ 18.0 ns
Max Allowed Allocation: 0 B
Target Metric: Zero allocation UUID parser
```

*Source: dotnet-multitenancy benchmark plan*

#### General CI Enforcement Syntax

```yaml
# Conceptual GitHub Actions step
- name: Performance budget gate
  run: |
    dotnet run -c Release --project benchmarks/MyApp.Benchmarks
    # Parse BenchmarkDotNet JSON and compare against budgets
    dotnet run --project tools/BudgetChecker -- --results BenchmarkDotNet.Artifacts/results
```

**Component breakdown**

- **Baseline** — The measured starting point.
- **Budget** — The maximum allowed value.
- **Measurement** — BenchmarkDotNet or load test.
- **Gate** — CI step that compares actual vs. budget and fails on regression.

#### Syntax Rules

- Budgets must be committed to source control and reviewed like code.
- Use percentiles (p95, p99) for latency budgets, not just means.
- Include allocation budgets (bytes per operation) to prevent GC pressure regressions.
- Re-baseline intentionally when making justified trade-offs.

#### Constraints and Limitations

- Budgets measured in micro-benchmarks may not perfectly reflect production.
- Overly strict budgets can block legitimate changes; they must be reviewed and adjusted.
- CI budget checks add build time.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: BenchmarkDotNet with Budget Attributes

```csharp
[MemoryDiagnoser]
public class TenantIdBenchmarks
{
    [Benchmark]
    [BenchmarkCategory("Budget")]
    public TenantId Create() => TenantId.Create(Guid.NewGuid());
}
```

```csharp
// Conceptual budget checker (simplified)
var summary = BenchmarkRunner.Run<TenantIdBenchmarks>();
var mean = summary.Reports[0].ResultStatistics.Mean;
var allocated = summary.Reports[0].GcStats.AllocatedBytes;

if (mean > 3.5 || allocated > 0)
    throw new Exception($"Budget exceeded: {mean} ns, {allocated} B");
```

**Expected Output (conceptual)**

```
Mean: 3.21 ns, Allocated: 0 B
Budget check: PASSED
```

**Why this output occurs**

- BenchmarkDotNet measures mean latency and allocation.
- The checker compares against the budget.
- If either exceeds the threshold, CI fails.

---

### Real-World Cases

- **Multi-tenant SaaS** — Zero-allocation budgets for hot-path ID creation.
- **High-throughput APIs** — p95 latency budgets per endpoint.
- **Game engines** — Per-frame allocation and latency budgets.
- **Databases and libraries** — Allocation-free parsing and serialization budgets.

---

### References

- Benchmark Plan & Performance Budgets — dotnet-multitenancy - https://github.com/ericksonlopezf/dotnet-multitenancy/blob/main/docs/benchmark-plan.md
- performance-optimization-dotnet — skills.rest - https://skills.rest/skill/performance-optimization-dotnet-peterblazejewicz
- BenchmarkDotNet Measures Performance—But Who Enforces the Budget? — DEV Community - https://dev.to/…/benchmarkdotnet-measures-performance-but-who-enforces-the-budget

---

## 7. Performance Best Practices

### Definitions

**Core Definition**
Performance best practices are empirically validated guidelines for designing, writing, and operating .NET code and systems to achieve low latency, high throughput, and efficient resource utilization.

**Technical Definition**
Best practices in .NET performance span allocation reduction (spans, pooling, structs), asynchronous I/O (avoiding sync-over-async), garbage collection tuning (server GC, LOH awareness), thread pool health (non-blocking, `CancellationToken` propagation), caching (memory, distributed, hybrid), data access optimization (async, no-tracking, projection), and continuous measurement (BenchmarkDotNet, `dotnet-counters`, `dotnet-trace`). They are grounded in the runtime’s architecture and are enforced by analyzers and performance budgets.

**Beginner-Friendly Explanation**
Best practices are “the habits that make your app fast.” They include not blocking threads, not allocating unnecessary memory, using async all the way, caching aggressively, and measuring before changing anything.

---

### Purposes

- To reduce latency and improve throughput.
- To minimize GC pressure and memory consumption.
- To avoid thread pool starvation and deadlocks.
- To make performance predictable and maintainable.
- To prevent regressions through measurement and budgets.
- To align code with the runtime’s strengths.

---

### Guidelines and Structure

#### Allocation Reduction

```
✅ Use Span<T> / ReadOnlySpan<T> for slicing and parsing instead of Substring.
✅ Use ArrayPool<T>.Shared for temporary buffers.
✅ Use stackalloc for small fixed-size buffers.
✅ Prefer readonly struct for small immutable value types.
✅ Use static lambdas to avoid closure allocations.
✅ Use StringComparison.Ordinal for string comparisons to enable vectorization.
```

#### Async and Threading

```
✅ Async all the way; never use .Result or .Wait().
✅ Propagate CancellationToken through the call chain.
✅ Use ValueTask<T> for frequently synchronous hot paths.
✅ Use ConfigureAwait(false) in library code.
✅ Avoid Task.Run in ASP.NET Core request paths.
```

#### Garbage Collection

```
✅ Minimize large object allocations (≥ 85,000 bytes).
✅ Enable Server GC for multi-core throughput workloads.
✅ Cache frequently used large objects.
✅ Avoid LINQ in hot paths; prefer for/foreach.
```

#### Caching

```
✅ Cache aggressively where stale data is acceptable.
✅ Use HybridCache for stampede protection and L1/L2 caching.
✅ Use no-tracking queries for read-only EF Core access.
```

#### Data Access

```
✅ Call all data access APIs asynchronously.
✅ Minimize network round trips; retrieve only necessary data.
✅ Use compiled queries for hot-path EF Core queries.
✅ Filter and aggregate in the database, not in memory.
```

#### Measurement

```
✅ Measure before optimizing; establish a baseline.
✅ Use BenchmarkDotNet for micro-benchmarks.
✅ Use dotnet-counters and dotnet-trace for live diagnostics.
✅ Enforce performance budgets in CI.
```

#### Syntax Rules

- Apply optimizations only to hot code paths; readability matters elsewhere.
- Never block on async code; sync-over-async causes thread pool starvation and deadlocks.
- Always propagate `CancellationToken`; dropped tokens waste server resources.
- Prefer `ValueTask<T>` only when synchronous completion is common; otherwise use `Task<T>`.
- Use `IHttpClientFactory` instead of `new HttpClient()` to avoid socket exhaustion.

#### Constraints and Limitations

- Premature optimization is harmful; measure first.
- Some optimizations (spans, stackalloc) restrict usage in async methods and class fields.
- Caching introduces staleness and invalidation complexity.
- Server GC increases memory usage; not always appropriate for memory-constrained environments.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Avoiding Sync-over-Async

```csharp
// ❌ BAD: blocks a thread pool thread, can deadlock
var result = _service.GetAsync().Result;

// ✅ GOOD: async all the way
var result = await _service.GetAsync();
```

**Expected Output (conceptual)**

```
Under load, the bad version causes thread pool starvation and latency spikes.
The good version frees the thread while awaiting I/O.
```

**Why this output occurs**

- `.Result` blocks the calling thread until the task completes.
- `await` yields the thread back to the pool while waiting.

---

#### Example 2: Span-Based Parsing vs. Substring

```csharp
// ❌ Allocates a new string
string key = header.Substring(0, index);

// ✅ Zero allocation
ReadOnlySpan<char> key = header.AsSpan(0, index);
```

**Expected Output (conceptual)**

```
Span version allocates 0 bytes per operation.
Substring version allocates a new string each call.
```

**Why this output occurs**

- `Substring` creates a new string on the heap.
- `AsSpan` returns a view over the existing string.

---

#### Example 3: ValueTask for Hot Async Paths

```csharp
// ✅ ValueTask avoids Task allocation when the result is already available
public ValueTask<string?> GetCachedAsync(string key)
{
    if (_cache.TryGetValue(key, out var value))
        return new ValueTask<string?>(value);   // synchronous, zero allocation

    return new ValueTask<string?>(LoadFromDatabaseAsync(key));
}
```

**Expected Output (conceptual)**

```
Cache hit: 0 bytes allocated for the async state machine.
Cache miss: Task allocated only when truly asynchronous.
```

**Why this output occurs**

- `ValueTask<T>` can wrap a synchronous result without allocating a `Task`.
- Only the truly asynchronous path incurs `Task` allocation.

---

#### Example 4: Server GC Configuration

```xml
<PropertyGroup>
  <ServerGarbageCollection>true</ServerGarbageCollection>
  <ConcurrentGarbageCollection>true</ConcurrentGarbageCollection>
</PropertyGroup>
```

**Expected Output (conceptual)**

```
Throughput improves on multi-core servers because GC runs in parallel across cores.
```

**Why this output occurs**

- Server GC creates per-core heaps and performs parallel collection.

---

### Real-World Cases

- **ASP.NET Core APIs** — Async all-the-way, caching, no-tracking queries.
- **High-frequency libraries** — Span-based parsing, zero-allocation APIs.
- **Cloud services** — Resource-efficient code reduces infrastructure cost.
- **Real-time systems** — Tail-latency budgets enforced by CI.

---

### References

- ASP.NET Core Best Practices — dotnet/aspnetcore.docs - https://github.com/dotnet/aspnetcore.docs/blob/main/aspnetcore/fundamentals/best-practices.md
- .NET Performance Guidelines — xian55/hermesproxy - https://github.com/xian55/hermesproxy/blob/main/.claude/skills/dotnet-performance/SKILL.md
- dotnet-claude-kit performance rules — codewithmukesh - https://github.com/codewithmukesh/dotnet-claude-kit/blob/main/.claude/rules/performance.md
- Performance rules (code analysis) — Microsoft - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/performance-warnings
- Critical .NET Performance Anti-Patterns — GitHub - https://github.com/…/critical-dotnet-performance-anti-patterns
- Advanced .NET performance — ValueTask and Span - https://github.com/…/advanced-dotnet-performance

---

## Framework and Tooling Summary

| Concern | Tool / API | Notes |
|---|---|---|
| **Micro-benchmarking** | BenchmarkDotNet | Statistical rigor, allocation tracking, CI integration |
| **Live counters** | `dotnet-counters` | CPU, GC, thread pool, allocation rate |
| **CPU profiling** | `dotnet-trace` / PerfView | Sampling profiler for CPU hot spots |
| **Memory analysis** | `dotnet-gcdump` | Heap snapshot for leak detection |
| **Metrics API** | `System.Diagnostics.Metrics` | OpenTelemetry-compatible, cross-platform |
| **Event counters** | `EventCounters` | Cross-platform counter API for runtime and ASP.NET Core |
| **ASP.NET Core metrics** | Built-in `Microsoft.AspNetCore.Hosting` | Request rate and duration |
| **Budget enforcement** | CI step parsing BenchmarkDotNet JSON | Fails build on regression |
| **GC tuning** | `ServerGarbageCollection`, `ConcurrentGarbageCollection` | Throughput vs. memory trade-offs |

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| **Windows Performance Counters** | Windows-only; modern .NET uses cross-platform `EventCounters` and `System.Diagnostics.Metrics` |
| **Sync-over-async (`.Result`, `.Wait()`)** | Unsafe; causes thread pool starvation and deadlocks in ASP.NET Core |
| **`Task.Run` in request paths** | Unnecessary; ASP.NET Core already runs on thread pool threads |
| **`ValueTask` multi-await** | Prohibited; never await a `ValueTask` more than once |
| **`Span<T>` in async methods** | Not allowed; `ref struct` types cannot cross `await` boundaries |
| **`DateTime.Now` for timing** | Use `Stopwatch` for elapsed time; `DateTime.Now` is not monotonic and is a static dependency |
| **`new HttpClient()` per request** | Causes socket exhaustion; use `IHttpClientFactory` |
| **EF Core InMemory provider** | Not a relational database; do not use for performance testing that relies on SQL behavior |
| **Server GC for desktop apps** | Increases memory usage; not always appropriate for client applications |

---

## References (Consolidated)

- .NET Performance Testing — Compile-N-Run - https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/dotnet/14-net-testing/8-net-performance-testing.mdx
- .NET Performance Guidelines — xian55/hermesproxy - https://github.com/xian55/hermesproxy/blob/main/.claude/skills/dotnet-performance/SKILL.md
- Migrate from Windows Performance Counters to .NET metrics — Microsoft - https://learn.microsoft.com/en-us/dotnet/core/diagnostics/migrate-from-windows-performance-counters
- ASP.NET Core Best Practices — dotnet/aspnetcore.docs - https://github.com/dotnet/aspnetcore.docs/blob/main/aspnetcore/fundamentals/best-practices.md
- performance-optimization-dotnet — skills.rest - https://skills.rest/skill/performance-optimization-dotnet-peterblazejewicz
- dotnet-claude-kit performance rules — codewithmukesh - https://github.com/codewithmukesh/dotnet-claude-kit/blob/main/.claude/rules/performance.md
- .NET runtime metrics — dotnet/docs - https://github.com/dotnet/docs/blob/main/docs/core/diagnostics/metrics-built-in.md
- EventCounters in .NET — Microsoft - https://learn.microsoft.com/en-us/dotnet/core/diagnostics/event-counters
- ASP.NET Core built-in metrics — dotnet/docs - https://github.com/dotnet/docs/blob/main/docs/core/diagnostics/metrics-built-in.md
- Benchmark Plan & Performance Budgets — dotnet-multitenancy - https://github.com/ericksonlopezf/dotnet-multitenancy/blob/main/docs/benchmark-plan.md
- Performance rules (code analysis) — Microsoft - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/performance-warnings
- Performance lens — tail latency and percentiles - https://github.com/CliffYuan/dotnet-skills/blob/main/skills/performance-lens/SKILL.md
- Extreme Performance Tuning for ASP.NET Core Web APIs — alex.rocks - https://alex.rocks/extreme-performance-tuning-aspnet-core-web-apis/
- Critical .NET Performance Anti-Patterns — GitHub - https://github.com/…/critical-dotnet-performance-anti-patterns
- Advanced .NET performance — ValueTask and Span - https://github.com/…/advanced-dotnet-performance