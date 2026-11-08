# ASP.NET Core Performance Fundamentals: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core** (.NET 6–10) and the discipline of **performance engineering** — understanding, measuring, and improving how an application responds under load. Performance is not a single feature; it is a cross‑cutting property shaped by architecture (async‑all‑the‑way, caching, pagination), infrastructure (Kestrel, HTTP/2, compression), and observability (metrics, traces, profiling). The guidance here draws on Microsoft’s official **ASP.NET Core Best Practices** and the built‑in **diagnostic tooling** and **metrics** that ship with the framework. Each requested topic is treated independently — no sections are merged.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Performance is the degree to which a software system meets its **responsiveness** (how quickly it responds) and **capacity** (how much work it can handle) requirements under expected and peak load. |
| **Technical Definition** | In ASP.NET Core, performance is characterised by two primary dimensions: **latency** (the time to complete a single request, often measured as `http.server.request.duration`) and **throughput** (the rate at which requests are processed, measured in requests per second). These are governed by **Little’s Law** (`L = λW` — concurrency = throughput × latency) and are constrained by the **hot code paths** (frequently executed code where most execution time is spent), **thread‑pool health**, **allocation behaviour**, and **I/O efficiency**. Performance is observed through **metrics**, **traces**, and **profiling tools**, and improved through **best practices** such as caching, asynchronous I/O, pagination, and compression. |
| **Beginner‑Friendly Explanation** | Performance is **how fast and how much** your app can handle. **Latency** is how long one user waits for a response (e.g., 200 ms). **Throughput** is how many users you can serve per second (e.g., 1,000 requests/sec). They are connected: if requests take longer, fewer can be handled simultaneously. Performance engineering is the practice of measuring these numbers, finding the bottlenecks (usually in a few “hot” code paths), and applying proven techniques — caching, async, pagination, compression — to improve them. |

### Key Characteristics

- **Two‑dimensional** — Latency and throughput are distinct but interrelated; optimising one can affect the other.
- **Bottleneck‑driven** — A small number of **hot code paths** typically dominate execution time and limit scale‑out.
- **Observability‑dependent** — You cannot improve what you do not measure; metrics, traces, and profilers are essential.
- **Architecture‑sensitive** — Async‑all‑the‑way, caching, and pagination are structural choices, not afterthoughts.
- **Environment‑specific** — Performance characteristics differ between Development (debugger attached, no compression) and Production (Release build, compression enabled, HTTPS).

### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core project.
- Working knowledge of **middleware**, **dependency injection**, and **async/await** in C#.
- Familiarity with **HTTP** (status codes, headers, caching semantics).
- Basic understanding of **threads**, **thread pool**, and **garbage collection** in .NET.

### Related Programming Areas

- **Async programming** — The single largest architectural lever for throughput.
- **Caching** — Reduces repeated work and latency for frequently accessed data.
- **Database access (EF Core / Dapper)** — I/O latency and query efficiency dominate many APIs.
- **Middleware pipeline** — Compression, response caching, and rate limiting are pipeline‑level performance features.
- **Observability** — Metrics, distributed tracing, and profiling tools (PerfView, Application Insights, `dotnet-counters`).

### Core Concepts / Features

1. **Performance** — The overarching discipline, trade‑offs, and measurement philosophy.
2. **Latency** — Response time for a single request, its components, and how to reduce it.
3. **Throughput** — Requests per second, Little’s Law, and how to scale it.
4. **Performance metrics** — Built‑in ASP.NET Core metrics, external tools, and what to measure.
5. **Performance best practices** — Caching, async, pagination, compression, and profiling.

---

## 1. Performance (The Discipline)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Performance is the measure of how efficiently a system uses resources to deliver responses within acceptable latency and throughput bounds. |
| **Technical Definition** | Performance in ASP.NET Core is governed by the interaction of **CPU** (hot code paths, serialization, JIT), **memory** (allocations, GC pressure, large object heap), **I/O** (database queries, HTTP calls, file access), and **concurrency** (thread‑pool availability, async efficiency). The framework provides built‑in **metrics** (`Microsoft.AspNetCore.Hosting`, `Microsoft.AspNetCore.Server.Kestrel`, etc.), **diagnostic tools** (Visual Studio Diagnostic Tools, Application Insights, PerfView, `dotnet-counters`), and **best practices** (cache aggressively, avoid blocking calls, return large collections across smaller pages) to measure and improve these dimensions. |
| **Beginner‑Friendly Explanation** | Performance is the **overall health of your app under load**. It is not just “is it fast?” — it is “is it fast **and** can it handle many users **and** does it use memory and CPU efficiently?” A fast single request is useless if the app collapses under 100 concurrent users. Performance engineering is about finding the few lines of code (the hot paths) that cause most of the slowdown and fixing them systematically. |

### Purposes

- **To** deliver **responsive user experiences** — users abandon slow applications.
- **To** **scale efficiently** — handle more traffic without proportionally more hardware.
- **To** **reduce operational cost** — lower CPU, memory, and bandwidth consumption.
- **To** **prevent outages** — identify bottlenecks before they cause failures under peak load.
- **To** **make informed architectural decisions** — measure the impact of caching, async, and pagination before committing.

### Syntax Rules and Structure

#### The Performance Measurement Loop

```text
Measure → Identify bottleneck (hot path) → Apply technique → Measure again → Repeat
```

**Component breakdown:**

| Phase | Activity | Tooling |
|-------|----------|---------|
| **Measure** | Capture baseline latency, throughput, CPU, allocations. | `dotnet-counters`, Application Insights, BenchmarkDotNet |
| **Identify** | Find the hot code path consuming the most time. | PerfView, Visual Studio profiler, traces |
| **Apply** | Cache, make async, paginate, compress, or optimise queries. | Code changes + configuration |
| **Verify** | Re‑measure to confirm improvement and detect regressions. | Same tooling as baseline |

**Syntax rules:**

- **Measure first** — do not optimise based on intuition; profile to find the actual bottleneck.
- **Focus on hot code paths** — a small percentage of code typically accounts for most execution time.
- **Establish a baseline** before any change so improvement is provable.

**Constraints and limitations:**

- **Premature optimisation is the root of all evil** — optimising code that is not a bottleneck wastes effort and adds complexity.
- **Development and Production behave differently** — debugger overhead, lack of compression, and different logging levels skew measurements.

### Annotated Complete Step‑by‑Step Code Example (Baseline Measurement)

```bash
# Install dotnet-counters (global tool)
dotnet tool install --global dotnet-counters

# Run the app in Release mode
dotnet run -c Release

# In another terminal, monitor key counters
dotnet-counters monitor --process-id <PID> \
  --counters System.Runtime,Microsoft.AspNetCore.Hosting
```

**Step-by-step explanation:**

1. `dotnet-counters` is installed as a global tool.
2. The application is started in **Release** configuration (no debugger, optimisations enabled).
3. `dotnet-counters monitor` attaches to the process and streams live metrics — CPU usage, GC heap size, thread‑pool queue length, and HTTP request rate.

**Expected Output:** A live table of counters showing CPU %, GC heap size (MB), thread‑pool queue length, and request rate. High thread‑pool queue length indicates blocking calls; growing GC heap indicates allocation pressure.

**Why this output?** `dotnet-counters` reads the .NET runtime’s **EventPipe** counters without any code changes, providing an immediate performance baseline.

### Real-World Cases

- **API under load:** Baseline latency and throughput before and after enabling response compression.
- **Memory investigation:** GC heap growth in a long‑running service.
- **Startup time:** Measure cold‑start latency for serverless or containerised deployments.

### References

- ASP.NET Core Best Practices (Microsoft Learn) – https://github.com/dotnet/aspnetcore.docs/blob/7e6be970565b61d251d839555bec7cf0a1a20a53/aspnetcore/fundamentals/best-practices.md
- Performance Diagnostic Tools – https://learn.microsoft.com/en-us/aspnet/core/performance/diagnostic-tools?view=aspnetcore-10.0

---

## 2. Latency (Response Time)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Latency is the time it takes for a single request to travel from the client to the server and for the response to return — the user‑perceived delay. |
| **Technical Definition** | Latency in ASP.NET Core decomposes into **network latency** (client ↔ server), **queuing latency** (time waiting for a thread‑pool thread or database connection), **processing latency** (middleware, routing, model binding, controller execution, serialization), and **dependency latency** (database queries, HTTP calls to external services). The built‑in metric `http.server.request.duration` measures server‑side processing time. Little’s Law relates latency to throughput and concurrency: `L = λW` (concurrency = throughput × latency). |
| **Beginner‑Friendly Explanation** | Latency is **how long one user waits**. If a user clicks a button and waits 200 ms for a response, the latency is 200 ms. It is made up of many small delays: the network trip, waiting in line for a thread, running your code, querying the database, and sending the response back. Reducing latency means finding the biggest chunk of that wait and shrinking it — usually by caching, optimising queries, or avoiding blocking calls. |

### Purposes

- **To** **minimise user‑perceived wait time** — faster responses improve satisfaction and retention.
- **To** **identify the largest latency contributors** — network, queuing, processing, or dependencies.
- **To** **set realistic SLOs** — e.g., “95th percentile latency < 200 ms.”
- **To** **guide optimisation** — if 80% of latency is database time, optimise queries, not serialization.
- **To** **understand the latency‑throughput trade‑off** — reducing latency often increases capacity.

### Syntax Rules and Structure

#### Latency Decomposition

```text
Total latency ≈ Network RTT + Queue wait + Processing time + Dependency time
```

**Component breakdown:**

| Component | Typical contributors | How to reduce |
|-----------|----------------------|---------------|
| **Network RTT** | Geographic distance, TLS handshake | CDN, connection reuse, HTTP/2 |
| **Queue wait** | Thread‑pool starvation, DB connection pool exhaustion | Async‑all‑the‑way, connection pooling |
| **Processing** | Middleware, routing, model binding, serialization | Efficient middleware order, minimal allocations |
| **Dependency** | Database queries, external HTTP calls | Caching, query optimisation, parallel calls |

**Syntax rules:**

- **Measure `http.server.request.duration`** —it is the built‑in server‑side latency metric.
- **Break down latency by dependency** —Application Insights shows dependency durations (SQL, HTTP).
- **Use percentiles (P50, P95, P99)** —averages hide tail latency, which is what users feel.

**Constraints and limitations:**

- **Server‑side latency is not total latency** —network time and client rendering are excluded from `http.server.request.duration`.
- **Average latency is misleading** —a small number of very slow requests can dominate user experience.

### Annotated Complete Step‑by‑Step Code Example (Reducing Dependency Latency with Caching)

```csharp
// Before: every request queries the database
public async Task<Product?> GetProductAsync(int id)
    => await _db.Products.FindAsync(id);

// After: cache aggressively for frequently accessed products
public async Task<Product?> GetProductAsync(int id)
{
    var cacheKey = $"product:{id}";
    if (_cache.TryGetValue(cacheKey, out Product? cached))
        return cached;

    var product = await _db.Products.FindAsync(id);
    if (product is not null)
        _cache.Set(cacheKey, product, TimeSpan.FromMinutes(5));

    return product;
}
```

**Step-by-step explanation:**

1. The original method queries the database on every request —dependency latency dominates.
2. The optimised method checks an in‑memory cache first; a hit returns immediately, bypassing the database.
3. On a miss, the database is queried and the result is cached for five minutes.

**Expected Output:** Repeated requests for the same product return in microseconds (cache hit) instead of milliseconds (database round‑trip).

**Why this output?** Caching eliminates the dependency latency for frequently accessed data, directly reducing total request latency.

### Real-World Cases

- **Product pages:** Cache product details for 5–10 minutes; invalidate on update.
- **Reference data:** Cache countries, categories, and configuration in memory.
- **External API calls:** Cache responses from third‑party services with appropriate TTLs.

### References

- ASP.NET Core Best Practices — Cache aggressively – https://github.com/dotnet/aspnetcore.docs/blob/7e6be970565b61d251d839555bec7cf0a1a20a53/aspnetcore/fundamentals/best-practices.md
- ASP.NET Core built-in metrics (`http.server.request.duration`) – https://github.com/dotnet/aspnetcore.docs/blob/main/aspnetcore/log-mon/metrics/built-in.md

---

## 3. Throughput (Requests per Second)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Throughput is the rate at which a system successfully processes requests, typically measured in requests per second (RPS) or transactions per second (TPS). |
| **Technical Definition** | Throughput is bounded by **Little’s Law**: `Throughput (λ) = Concurrency (L) / Latency (W)`. If a system can handle `L` concurrent requests and each takes `W` time, throughput is `L/W`. Throughput is limited by **thread‑pool size**, **CPU cores**, **database connection pools**, **network bandwidth**, and **blocking calls** that starve the thread pool. Asynchronous I/O allows a small thread pool to handle thousands of concurrent requests by freeing threads while waiting. |
| **Beginner‑Friendly Explanation** | Throughput is **how many users you can serve per second**. If each request takes 100 ms and you can handle 100 concurrent requests, throughput is 100 / 0.1 = 1,000 requests per second. The key to high throughput is **not blocking threads** —if every request waits synchronously for a database, threads are tied up and you cannot handle many requests. Async/await frees the thread while waiting, allowing far more concurrent work. |

### Purposes

- **To** **maximise capacity** —serve more users with the same hardware.
- **To** **reduce infrastructure cost** —higher throughput per instance means fewer instances.
- **To** **prevent thread‑pool starvation** —the most common throughput killer in ASP.NET Core.
- **To** **understand the latency‑throughput trade‑off** —reducing latency directly increases throughput (Little’s Law).
- **To** **set scaling targets** —know when to add instances vs. optimise code.

### Syntax Rules and Structure

#### Little’s Law

```text
Concurrency (L) = Throughput (λ) × Latency (W)
Throughput (λ) = Concurrency (L) / Latency (W)
```

**Component breakdown:**

| Variable | Meaning | Example |
|----------|---------|---------|
| **λ (Throughput)** | Requests per second | 1,000 req/s |
| **W (Latency)** | Time per request | 0.1 s (100 ms) |
| **L (Concurrency)** | Requests in flight | 100 |

**Syntax rules:**

- **To increase throughput, either increase concurrency or decrease latency.**
- **Async I/O increases effective concurrency** by freeing threads while waiting.
- **Thread‑pool starvation caps concurrency** —blocking calls reduce `L` to the number of available threads.

**Constraints and limitations:**

- **Concurrency is not unlimited** —CPU cores, memory, database connections, and downstream services impose hard limits.
- **Adding concurrency beyond a bottleneck can increase latency** without increasing throughput.

### Annotated Complete Step‑by‑Step Code Example (Async All‑the‑Way for Throughput)

```csharp
// ❌ Blocking — thread is tied up while waiting for the database
public IActionResult GetProducts()
{
    var products = _db.Products.ToList();   // Synchronous — thread blocked
    return Ok(products);
}

// ✅ Asynchronous — thread is freed while waiting
public async Task<IActionResult> GetProductsAsync()
{
    var products = await _db.Products.ToListAsync();   // Thread freed during I/O
    return Ok(products);
}
```

**Step-by-step explanation:**

1. The synchronous version blocks the thread while the database query runs. Under load, threads run out and requests queue.
2. The asynchronous version `await`s the query; the thread returns to the pool to handle other requests while the database works.

**Expected Output:** Under concurrent load, the async endpoint sustains far higher throughput (more requests per second) because threads are not starved.

**Why this output?** Async I/O increases effective concurrency (`L`) without increasing thread count, directly raising throughput per Little’s Law.

### Real-World Cases

- **High‑traffic APIs:** Async controllers and `ToListAsync`/`FirstOrDefaultAsync` for all database calls.
- **External HTTP calls:** `HttpClient` with async methods and connection reuse.
- **File uploads/downloads:** Streaming instead of buffering entire payloads in memory.

### References

- Little’s Law (Microsoft Learn) – https://learn.microsoft.com/en-us/azure/devops/load-test/little-law?view=azure-devops
- ASP.NET Core Best Practices — Avoid blocking calls – https://github.com/dotnet/aspnetcore.docs/blob/7e6be970565b61d251d839555bec7cf0a1a20a53/aspnetcore/fundamentals/best-practices.md
- dotnet-claude-kit performance rules (async all the way) – https://github.com/codewithmukesh/dotnet-claude-kit/blob/main/.claude/rules/performance.md

---

## 4. Performance Metrics (What to Measure)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Performance metrics are quantitative measurements of a system’s behaviour — latency, throughput, error rate, resource usage — used to assess health, detect regressions, and guide optimisation. |
| **Technical Definition** | ASP.NET Core ships **built‑in metrics** across namespaces including `Microsoft.AspNetCore.Hosting` (`http.server.request.duration`), `Microsoft.AspNetCore.Server.Kestrel` (`kestrel.active_connections`), `Microsoft.AspNetCore.Routing` (`aspnetcore.routing.match_attempts`), and `Microsoft.AspNetCore.Diagnostics` (`aspnetcore.diagnostics.exceptions`). These metrics are exposed via **EventPipe** (consumable by `dotnet-counters`, OpenTelemetry, Prometheus) and can be visualised in Grafana, Azure Monitor, or SigNoz. **Diagnostic tools** (PerfView, Application Insights Profiler, Visual Studio Diagnostic Tools) provide deeper CPU, memory, and allocation analysis. |
| **Beginner‑Friendly Explanation** | Performance metrics are the **dashboard for your app**. They tell you: How many requests per second are we handling? How long are they taking? Are errors increasing? How much memory and CPU are we using? Without metrics, you are flying blind — you cannot know if a change made things better or worse. ASP.NET Core gives you many of these metrics for free; you just need to collect and visualise them. |

### Purposes

- **To** **detect performance regressions** before users notice.
- **To** **establish baselines and SLOs** —e.g., “P95 latency must stay below 300 ms.”
- **To** **identify bottlenecks** —high thread‑pool queue length indicates blocking; growing GC heap indicates allocations.
- **To** **validate optimisations** —measure before and after a caching or async change.
- **To** **alert on anomalies** —error rate spikes or latency increases trigger investigation.

### Syntax Rules and Structure

#### Built‑in Metrics (Selected)

| Metric | Namespace | What It Tells You |
|--------|-----------|-------------------|
| `http.server.request.duration` | `Microsoft.AspNetCore.Hosting` | Server‑side request latency (histogram). |
| `kestrel.active_connections` | `Microsoft.AspNetCore.Server.Kestrel` | Current open connections. |
| `aspnetcore.routing.match_attempts` | `Microsoft.AspNetCore.Routing` | Route matching attempts and outcomes. |
| `aspnetcore.diagnostics.exceptions` | `Microsoft.AspNetCore.Diagnostics` | Unhandled exception count. |
| `aspnetcore.authorization.attempts` | `Microsoft.AspNetCore.Authorization` | Authorization decisions. |
| `dotnet.gc.heap.total_allocated` | `System.Runtime` | Allocation rate (GC pressure). |
| `dotnet.thread_pool.queue.length` | `System.Runtime` | Thread‑pool starvation indicator. |

#### Collection and Visualisation

```bash
# Live metrics in the terminal
dotnet-counters monitor --counters System.Runtime,Microsoft.AspNetCore.Hosting
```

```csharp
// OpenTelemetry + Prometheus (Program.cs)
builder.Services.AddOpenTelemetry()
    .WithMetrics(metrics => metrics
        .AddAspNetCoreInstrumentation()
        .AddPrometheusExporter());
```

**Syntax rules:**

- **Use `dotnet-counters` for ad‑hoc investigation** —no code changes required.
- **Use OpenTelemetry for production observability** —export to Prometheus, Grafana, or Azure Monitor.
- **Monitor percentiles** (P50, P95, P99) —not just averages.
- **Correlate metrics with traces** —a high‑latency metric should lead you to the slow span.

**Constraints and limitations:**

- **Metrics alone do not explain why** —they show symptoms; use traces and profilers to find causes.
- **Built‑in metrics require .NET 8+** —older versions rely on third‑party instrumentation.

### Annotated Complete Step‑by‑Step Code Example (Monitoring a Hot Path)

```csharp
// Add a custom metric to track a critical operation
private static readonly Meter _meter = new("MyApp.Orders", "1.0");
private static readonly Histogram<double> _orderProcessingTime =
    _meter.CreateHistogram<double>("order.processing.duration", "ms");

public async Task<IActionResult> PlaceOrderAsync(OrderInput input)
{
    var sw = Stopwatch.StartNew();
    try
    {
        var order = await _orderService.PlaceAsync(input);
        return Ok(order);
    }
    finally
    {
        sw.Stop();
        _orderProcessingTime.Record(sw.Elapsed.TotalMilliseconds);
    }
}
```

**Step-by-step explanation:**

1. A `Meter` and `Histogram` are created for the order‑processing operation.
2. The action measures elapsed time with a `Stopwatch`.
3. The duration is recorded in the histogram, which can be exported via OpenTelemetry.

**Expected Output:** A histogram metric (`order.processing.duration`) that can be visualised as percentiles in Grafana or Prometheus.

**Why this output?** Custom metrics capture domain‑specific performance data that built‑in HTTP metrics cannot see.

### Real-World Cases

- **SLO dashboards:** P95 latency and error rate per endpoint.
- **Thread‑pool monitoring:** `dotnet.thread_pool.queue.length` > 0 indicates blocking calls.
- **Allocation tracking:** `dotnet.gc.heap.total_allocated` reveals high‑allocation hot paths.

### References

- ASP.NET Core built-in metrics – https://github.com/dotnet/aspnetcore.docs/blob/main/aspnetcore/log-mon/metrics/built-in.md
- Performance Diagnostic Tools – https://learn.microsoft.com/en-us/aspnet/core/performance/diagnostic-tools?view=aspnetcore-10.0
- dotnet-counters (Microsoft Learn) – https://learn.microsoft.com/en-us/aspnet/core/performance/metrics

---

## 5. Performance Best Practices (Proven Techniques)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Performance best practices are the proven, repeatable techniques that Microsoft and the .NET community recommend for maximising ASP.NET Core application performance and reliability. |
| **Technical Definition** | The canonical best practices from Microsoft’s official guidance include: **cache aggressively** (memory, distributed, response caching, output caching); **understand and optimise hot code paths**; **avoid blocking calls** (no `.Result`, `.Wait()`, or `Task.Run` + await); **return large collections across multiple smaller pages** (pagination); **minimise large object allocations**; **optimise data access and I/O** (async all the way, minimise round trips); and **use response compression** (Brotli/Gzip) and **HTTP/2/3** where applicable. Profiling tools (PerfView, Application Insights Profiler) confirm the impact of each change. |
| **Beginner‑Friendly Explanation** | These are the **“greatest hits” of performance advice** —the techniques that consistently deliver the biggest wins. They are not micro‑optimisations; they are structural choices: cache instead of re‑querying, use async instead of blocking, paginate instead of loading everything, compress responses instead of sending raw JSON. Following these practices from the start is easier than retrofitting them later. |

### Purposes

- **To** **eliminate the most common performance killers** —blocking calls, uncached database queries, unbounded collections.
- **To** **provide a checklist for code review** —new code should follow these practices by default.
- **To** **prioritise effort** —these techniques have the highest impact for the least complexity.
- **To** **align with Microsoft’s official guidance** —the same practices used by the ASP.NET Core team.
- **To** **measure and verify** —each practice can be validated with metrics before and after.

### Guidelines Checklist

| Area | Practice | Why It Matters |
|------|----------|----------------|
| **Async** | Use `async`/`await` throughout the call stack; never `.Result` or `.Wait()`. | Blocking calls starve the thread pool and cap throughput. |
| **Caching** | Cache frequently accessed data in memory or a distributed cache. | Eliminates repeated database and network latency. |
| **Pagination** | Return collections in pages (`page`, `pageSize`); use `IAsyncEnumerable` for streaming. | Prevents memory spikes, thread‑pool starvation, and slow responses. |
| **Data access** | Call all data access APIs asynchronously; minimise round trips; project only needed columns. | Database latency dominates many APIs; async frees threads. |
| **Compression** | Enable `UseResponseCompression` with Brotli and Gzip. | Reduces payload size and bandwidth, especially for JSON. |
| **Allocations** | Minimise large object allocations; reuse buffers; avoid `Task.Run` for async wrapping. | Reduces GC pressure and pause times. |
| **Profiling** | Use PerfView, Application Insights Profiler, or `dotnet-counters` to find hot paths. | Optimising without measurement wastes effort. |

### Annotated Code Example (Best‑Practice Pipeline for Performance)

```csharp
var builder = WebApplication.CreateBuilder(args);

// 1. Response compression — reduce payload size
builder.Services.AddResponseCompression(options =>
{
    options.EnableForHttps = true;
    options.Providers.Add<BrotliCompressionProvider>();
    options.Providers.Add<GzipCompressionProvider>();
});

// 2. Memory cache — eliminate repeated database work
builder.Services.AddMemoryCache();
builder.Services.AddScoped<IProductService, ProductService>();

var app = builder.Build();

// 3. Middleware order: compression before static files and endpoints
app.UseResponseCompression();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();
app.Run();
```

**Why this follows best practices:**

- **Response compression is enabled** —Brotli/Gzip reduce JSON payload size significantly.
- **Memory cache is registered** —the service can cache frequently accessed products.
- **Middleware order** places compression before the components that generate compressible responses.
- **Async is used throughout** the service layer (not shown, but assumed).

### Real‑World Cases

- **JSON‑heavy APIs:** Response compression reduces payload size by 60–80%.
- **Product catalogues:** In‑memory caching of product details with 5–10 minute TTL.
- **Large list endpoints:** Pagination prevents memory spikes and slow responses.

### References

- ASP.NET Core Best Practices (official) – https://github.com/dotnet/aspnetcore.docs/blob/7e6be970565b61d251d839555bec7cf0a1a20a53/aspnetcore/fundamentals/best-practices.md
- Response compression in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/performance/response-compression
- Testing, Performance, And Operations (GitHub) – https://github.com/midudev/autoskills/blob/2dbab03b41988327e5c714d3fb233ebbce915e8c/packages/autoskills/skills-registry/aspnet-core/references/testing-performance-and-operations.md
- dotnet-claude-kit performance rules – https://github.com/codewithmukesh/dotnet-claude-kit/blob/main/.claude/rules/performance.md

---

## Summary Table: Performance Fundamentals at a Glance

| Concept | Primary Role | Key Metric / Tool | When to Focus |
|---------|--------------|-------------------|---------------|
| **Performance** | Overall system efficiency under load | `dotnet-counters`, profilers, metrics | Every production system |
| **Latency** | Time per request | `http.server.request.duration` (P95/P99) | When users complain about slowness |
| **Throughput** | Requests per second | `http.server.request.duration` + concurrency | When scaling or hitting thread‑pool limits |
| **Performance metrics** | Measure, alert, and diagnose | Built‑in metrics, OpenTelemetry, `dotnet-counters` | Before, during, and after optimisation |
| **Best practices** | Proven techniques for improvement | Async, caching, pagination, compression | New code and refactoring |

---

## References (Full List)

- ASP.NET Core Best Practices (Microsoft Learn) – https://github.com/dotnet/aspnetcore.docs/blob/7e6be970565b61d251d839555bec7cf0a1a20a53/aspnetcore/fundamentals/best-practices.md
- Performance Diagnostic Tools – https://learn.microsoft.com/en-us/aspnet/core/performance/diagnostic-tools?view=aspnetcore-10.0
- ASP.NET Core built-in metrics – https://github.com/dotnet/aspnetcore.docs/blob/main/aspnetcore/log-mon/metrics/built-in.md
- Response compression in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/performance/response-compression
- Little’s Law (Microsoft Learn) – https://learn.microsoft.com/en-us/azure/devops/load-test/little-law?view=azure-devops
- dotnet-counters (Microsoft Learn) – https://learn.microsoft.com/en-us/aspnet/core/performance/metrics
- Testing, Performance, And Operations (GitHub) – https://github.com/midudev/autoskills/blob/2dbab03b41988327e5c714d3fb233ebbce915e8c/packages/autoskills/skills-registry/aspnet-core/references/testing-performance-and-operations.md
- dotnet-claude-kit performance rules – https://github.com/codewithmukesh/dotnet-claude-kit/blob/main/.claude/rules/performance.md
- Memory management and garbage collection in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/performance/memory
- Application Insights for ASP.NET Core – https://learn.microsoft.com/en-us/azure/azure-monitor/app/asp-net-core
- PerfView (GitHub) – https://github.com/microsoft/perfview
- ASP.NET Core performance overview (OneUptime) – https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2025-12-23-aspnet-core-performance/README.md
- ASP.NET Core Best Practices (GitHub raw) – https://raw.githubusercontent.com/dotnet/aspnetcore.docs/main/aspnetcore/fundamentals/best-practices.md