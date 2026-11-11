# C# Exception Patterns — Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition:** Exception patterns are **reusable architectural and coding strategies** for responding to errors in a principled, predictable way. They guide whether an application should stop immediately, continue in a degraded mode, retry the operation, substitute a fallback value, or record the failure for later analysis. These patterns sit above individual `try`/`catch` blocks and define the **policy** of error handling across an application.

**Technical Definition:** Exception patterns in .NET are implemented through a combination of language constructs (`try`/`catch`/`finally`, exception filters), resilience libraries (`Microsoft.Extensions.Http.Resilience`, Polly), logging abstractions (`ILogger`, Serilog), and observability tooling (OpenTelemetry, Application Insights). The choice of pattern determines the **failure mode** of the system: fail fast surfaces defects immediately, fail safe preserves data integrity, graceful degradation maintains partial functionality, retry logic handles transient faults, and fallback values provide substitutes when primary operations fail. These patterns are not mutually exclusive; production systems typically combine several of them at different layers.

**Beginner-Friendly Explanation:** Exception patterns are the **“rules of engagement”** for errors. Imagine a restaurant kitchen: **fail fast** is the chef stopping the entire service when a critical ingredient is spoiled; **graceful degradation** is removing that dish from the menu but continuing to serve everything else; **retry** is asking the supplier for a replacement delivery; **fallback** is using a substitute ingredient; **logging** is writing down what went wrong; and **monitoring** is the manager watching a dashboard for patterns of failures. Each pattern answers the question: *“When something breaks, what should the system do next?”*

---

### Key Characteristics

- **Policy-driven** — Patterns define *how* the system responds, not just *whether* an exception is caught.
- **Layered** — Different patterns apply at different boundaries (controller, service, repository).
- **Complementary** — Retry and fallback are often combined; fail fast and graceful degradation are chosen based on criticality.
- **Observability-dependent** — Logging and monitoring make patterns diagnosable.
- **Resilience-oriented** — Patterns aim to recover from transient faults, not just hide them.
- **Context-sensitive** — The same exception may trigger fail fast in one layer and graceful degradation in another.

---

### Prerequisites

- Solid understanding of `try`/`catch`/`finally`, exception hierarchy, and exception best practices.
- Familiarity with `async`/`await` and `Task`-based asynchronous programming.
- Knowledge of `ILogger` and structured logging.
- Awareness of resilience libraries (Polly, `Microsoft.Extensions.Http.Resilience`).
- Basic understanding of distributed systems and transient faults.

---

### Related Programming Areas

- **Resilience engineering** — Retry, circuit breaker, fallback, timeout.
- **Observability** — Structured logging, distributed tracing, metrics.
- **Domain-driven design** — Domain exceptions, anti-corruption layers.
- **API design** — Problem details, error responses, status codes.
- **Cloud-native patterns** — Graceful degradation, bulkheads, health checks.

---

### Core Concepts / Features

1. Fail Fast
2. Fail Safe
3. Graceful Degradation
4. Retry Logic
5. Fallback Values
6. Error Logging
7. Error Monitoring
8. Exception Pattern Best Practices

---

## 1. Fail Fast

### Definitions

- **Core Definition:** Fail fast is a strategy where the application **immediately throws an error or exception** when something goes wrong, rather than attempting to recover or work around the problem.
- **Technical Definition:** Fail fast surfaces defects at the earliest possible moment by validating preconditions, throwing exceptions immediately when invariants are violated, and refusing to continue in an undefined state. It is the default for **critical failures** where continuing could cause data corruption, security breaches, or cascading failures.
- **Beginner-Friendly Explanation:** Fail fast is **“stop the line”** —when something is seriously wrong, stop immediately so the problem can be diagnosed and fixed.

### Purposes

- To **surface defects early** so they can be diagnosed and fixed quickly.
- To **prevent undefined behaviour** and data corruption.
- To **simplify debugging** by stopping at the point of failure.
- To **avoid cascading failures** in critical paths.
- To **enforce invariants** and contract preconditions.

### Syntax Rules and Structure

```csharp
// Guard clause: fail fast on invalid argument
public void ProcessOrder(Order order)
{
    ArgumentNullException.ThrowIfNull(order);
    if (order.Items.Count == 0)
        throw new ArgumentException("Order must contain at least one item.", nameof(order));
    // continue with valid state
}
```

**Component breakdown**

- **Guard clauses** — Validate preconditions at the start of a method.
- **`ArgumentNullException.ThrowIfNull`** — Modern fail-fast helper (.NET 6+).
- **Custom domain exceptions** — Throw immediately when invariants are violated.

#### Syntax Rules

- Validate all inputs before performing work.
- Throw the most specific exception type.
- Do not catch fail-fast exceptions unless at a top-level boundary.
- Fail fast is appropriate for **programming errors**, **corrupted state**, and **unrecoverable conditions**.

#### Constraints and Limitations

- Overuse of fail fast in distributed systems can cause cascading failures.
- Not appropriate for transient failures or expected error conditions.

### Annotated Code Examples and Expected Outputs

#### Example: Fail-Fast Validation

```csharp
using System;

public sealed class OrderProcessor
{
    public void Process(Order order)
    {
        ArgumentNullException.ThrowIfNull(order);
        if (order.Total <= 0)
            throw new ArgumentOutOfRangeException(nameof(order), "Total must be positive.");

        Console.WriteLine($"Processing order with total {order.Total:C}");
    }
}

public class Order { public decimal Total { get; set; } }

class Program
{
    static void Main()
    {
        try
        {
            new OrderProcessor().Process(new Order { Total = 0 });
        }
        catch (ArgumentOutOfRangeException ex)
        {
            Console.WriteLine($"Failed fast: {ex.Message}");
        }
    }
}
```

**Expected Output**
```
Failed fast: Total must be positive. (Parameter 'order')
```

**Why this output occurs:** The guard clause throws immediately when the invariant (`Total > 0`) is violated, preventing any further processing with invalid data.

### Real-World Cases

- **Public API input validation** — Reject invalid requests immediately with `400 Bad Request`.
- **Financial transactions** — Refuse to process if account state is inconsistent.
- **Configuration startup** — Crash at startup if required settings are missing.

### References

- Resiliency approaches — Failing fast vs. graceful degradation — Microsoft Learn (GitHub) — https://github.com/microsoftdocs/learn/blob/3b1c46a60dffd575ec3f94d27003fefbb24ffb52/learn-pr/aspnetcore/microservices-resiliency-aspnet-core/includes/2-application-infrastructure-resiliency.md

---

## 2. Fail Safe

### Definitions

- **Core Definition:** Fail safe is a strategy where, upon failure, the system **defaults to a safe state** that preserves data integrity and prevents harm, even if functionality is lost.
- **Technical Definition:** Fail safe prioritizes **correctness and safety over availability**. When an error occurs, the system rolls back changes, releases resources, and enters a known-good state. It is common in transactional systems, security-sensitive operations, and embedded systems.
- **Beginner-Friendly Explanation:** Fail safe is **“when in doubt, do nothing harmful”** —if an operation fails, leave everything as it was before.

### Purposes

- To **preserve data integrity** and avoid partial updates.
- To **prevent security breaches** when authentication or authorization fails.
- To **guarantee resource release** even on failure.
- To **maintain a known-good state** for recovery.

### Syntax Rules and Structure

```csharp
using var transaction = connection.BeginTransaction();
try
{
    // perform multiple related operations
    transaction.Commit();
}
catch (Exception)
{
    transaction.Rollback();   // fail safe: revert to previous state
    throw;
}
```

**Component breakdown**

- **Transactions** — Atomicity: either all operations succeed or none.
- **`using` / `finally`** — Guarantee resource cleanup.
- **Rollback** — Revert partial changes on failure.

#### Syntax Rules

- Use transactions for multi-step operations.
- Always release resources in `finally` or `using`.
- Do not leave shared state in an inconsistent condition.
- Rethrow after rollback to preserve the failure signal.

#### Constraints and Limitations

- Transactions add overhead; not every operation needs them.
- Fail safe may reduce availability (the operation fails rather than degrading).

### Annotated Code Examples and Expected Outputs

#### Example: Fail-Safe Transaction Rollback

```csharp
using System;

public class AccountService
{
    public void Transfer(decimal amount)
    {
        bool committed = false;
        try
        {
            Console.WriteLine("Debiting source...");
            Console.WriteLine("Crediting destination...");
            committed = true;
        }
        finally
        {
            if (!committed)
            {
                Console.WriteLine("Rolled back: safe state restored.");
            }
        }
    }
}

class Program
{
    static void Main()
    {
        try
        {
            new AccountService().Transfer(100m);
        }
        catch (Exception ex)
        {
            Console.WriteLine(ex.Message);
        }
    }
}
```

**Expected Output**
```
Debiting source...
Crediting destination...
```

**Why this output occurs:** The `finally` block ensures that if the operation did not commit, the safe-state message is printed, preserving integrity.

### Real-World Cases

- **Banking transfers** — Roll back debits if credits fail.
- **Inventory reservations** — Release reserved stock if checkout fails.
- **Authentication** — Deny access by default when the identity provider is unavailable.

### References

- Best practices for exceptions — Use `try`/`catch`/`finally` to recover from errors or release resources — Microsoft Learn (GitHub) — https://raw.githubusercontent.com/dotnet/docs/86590e2ef30ffff18842e7bb28036e9738630645/docs/standard/exceptions/best-practices-for-exceptions.md

---

## 3. Graceful Degradation

### Definitions

- **Core Definition:** Graceful degradation is a strategy where the application **continues to operate in a limited capacity** even when some components fail, rather than failing completely.
- **Technical Definition:** Graceful degradation (also called **resilient degradation**) keeps critical user-facing functions available while non-critical features are disabled or served from cached/fallback data. It is the recommended approach for **cloud-native and microservices** architectures, where partial failures are expected and failing fast in one service can cascade to others.
- **Beginner-Friendly Explanation:** Graceful degradation is **“do what you can with what you have”** —if the recommendation service is down, show the page without recommendations instead of showing an error page.

### Purposes

- To **maintain critical functionality** during partial failures.
- To **avoid cascading failures** across services.
- To **improve user experience** by hiding non-critical failures.
- To **allow failed components time to recover**.
- To **support fault isolation** and rapid recovery.

### Syntax Rules and Structure

```csharp
public async Task<HomePage> GetHomePageAsync()
{
    var products = await _productService.GetProductsAsync();   // critical
    IReadOnlyList<Recommendation> recommendations;
    try
    {
        recommendations = await _recommendationService.GetAsync();
    }
    catch (Exception ex)
    {
        _logger.LogWarning(ex, "Recommendations unavailable; degrading gracefully.");
        recommendations = Array.Empty<Recommendation>();        // degrade
    }
    return new HomePage(products, recommendations);
}
```

**Component breakdown**

- **Critical path** — Must succeed; failures propagate.
- **Non-critical path** — Wrapped in `try`/`catch`; failure substitutes an empty or cached result.
- **Logging** — Record the degradation for monitoring.

#### Syntax Rules

- Identify **critical** vs. **non-critical** dependencies.
- Catch exceptions only around non-critical calls.
- Substitute empty/cached/default values.
- Log degradation at `Warning` level for monitoring.
- Consider circuit breakers to avoid repeated failures.

#### Constraints and Limitations

- Degradation must be **intentional and observable**; silent failures are dangerous.
- Critical operations must not be degraded (use fail fast or fail safe instead).
- Cached data may be stale; communicate freshness where necessary.

### Annotated Code Examples and Expected Outputs

#### Example: Graceful Degradation with Fallback to Cached Data

```csharp
using System;
using System.Collections.Generic;

public interface IProductService { IReadOnlyList<string> GetProducts(); }
public interface IRecommendationService { IReadOnlyList<string> GetRecommendations(); }

public sealed class CachedRecommendationService : IRecommendationService
{
    public IReadOnlyList<string> GetRecommendations() =>
        throw new HttpRequestException("Recommendation service unavailable.");
}

public sealed class HomePageService
{
    private readonly IProductService _products;
    private readonly IRecommendationService _recommendations;

    public HomePageService(IProductService products, IRecommendationService recommendations) =>
        (_products, _recommendations) = (products, recommendations);

    public (IReadOnlyList<string> Products, IReadOnlyList<string> Recommendations) GetHomePage()
    {
        var products = _products.GetProducts();
        IReadOnlyList<string> recs;
        try
        {
            recs = _recommendations.GetRecommendations();
        }
        catch (HttpRequestException ex)
        {
            Console.WriteLine($"Degraded: {ex.Message}");
            recs = Array.Empty<string>();
        }
        return (products, recs);
    }
}

class Program
{
    static void Main()
    {
        var service = new HomePageService(
            new ProductService(),
            new CachedRecommendationService());
        var (products, recs) = service.GetHomePage();
        Console.WriteLine($"Products: {products.Count}, Recommendations: {recs.Count}");
    }
}
```

**Expected Output**
```
Degraded: Recommendation service unavailable.
Products: 3, Recommendations: 0
```

**Why this output occurs:** The critical product call succeeds; the non-critical recommendation call fails and is degraded to an empty list, allowing the page to render.

### Real-World Cases

- **E-commerce** — Show products even if reviews or recommendations are down.
- **Dashboards** — Display available widgets while hiding failed data sources.
- **Microservices** — Continue serving traffic when a non-critical dependency is unavailable.

### References

- Resiliency approaches — Graceful degradation — Microsoft Learn (GitHub) — https://github.com/microsoftdocs/learn/blob/3b1c46a60dffd575ec3f94d27003fefbb24ffb52/learn-pr/aspnetcore/microservices-resiliency-aspnet-core/includes/2-application-infrastructure-resiliency.md

---

## 4. Retry Logic

### Definitions

- **Core Definition:** Retry logic automatically **re-executes a failed operation** a limited number of times, usually with a delay, to recover from transient failures.
- **Technical Definition:** Retry is a resilience strategy that handles **transient faults** (network glitches, temporary service unavailability, throttling) by reattempting the operation. Best practice uses **exponential backoff with jitter** to avoid overwhelming the target and the “thundering herd” problem. In .NET, retry is typically implemented with **Polly** (`Microsoft.Extensions.Http.Resilience`) or `RetryPolicy` from `Microsoft.Extensions.Http.Resilience`.
- **Beginner-Friendly Explanation:** Retry is **“try again if it might work this time”** —but with a limit and a smart delay.

### Purposes

- To **recover from transient failures** without surfacing them to users.
- To **improve resilience** of network and I/O operations.
- To **handle throttling** (HTTP 429) and temporary outages.
- To **reduce manual intervention** for recoverable errors.

### Syntax Rules and Structure

```csharp
using Polly;

var pipeline = new ResiliencePipelineBuilder<HttpResponseMessage>()
    .AddRetry(new RetryStrategyOptions<HttpResponseMessage>
    {
        MaxRetryAttempts = 3,
        Delay = TimeSpan.FromMilliseconds(200),
        BackoffType = DelayBackoffType.Exponential,
        UseJitter = true
    })
    .Build();

var response = await pipeline.ExecuteAsync(async ct =>
    await httpClient.GetAsync(url, ct), cancellationToken);
```

**Component breakdown**

- **MaxRetryAttempts** — Limit on retries.
- **Delay** — Initial delay between attempts.
- **BackoffType** — `Linear` or `Exponential`.
- **UseJitter** — Randomizes delays to prevent synchronized retries.
- **ShouldHandle** — Predicate defining which exceptions/results trigger retries.

#### Syntax Rules

- Retry only **transient** and **idempotent** operations.
- Use **exponential backoff with jitter**.
- Cap total retry time.
- Do not retry non-idempotent operations (e.g., `POST` without idempotency keys).
- Log retries at `Warning` or `Information` level.

#### Constraints and Limitations

- Blind retries can **amplify outages** (thundering herd).
- Retries add latency; callers may need timeouts.
- Non-idempotent retries can cause duplicate side effects.

### Annotated Code Examples and Expected Outputs

#### Example: Retry with Exponential Backoff and Jitter

```csharp
using System;
using System.Net.Http;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        using var http = new HttpClient();
        for (int attempt = 1; attempt <= 3; attempt++)
        {
            try
            {
                var response = await http.GetAsync("https://httpstat.us/503");
                Console.WriteLine($"Attempt {attempt}: {response.StatusCode}");
            }
            catch (HttpRequestException ex)
            {
                Console.WriteLine($"Attempt {attempt} failed: {ex.Message}");
                if (attempt == 3) throw;
                var delay = TimeSpan.FromMilliseconds(200 * Math.Pow(2, attempt) + Random.Shared.Next(0, 100));
                Console.WriteLine($"Retrying in {delay.TotalMilliseconds:F0} ms");
                await Task.Delay(delay);
            }
        }
    }
}
```

**Expected Output (approximate)**
```
Attempt 1 failed: Response status code does not indicate success: 503 (Service Unavailable).
Retrying in 450 ms
Attempt 2 failed: ...
Retrying in 850 ms
Attempt 3 failed: ...
```

**Why this output occurs:** The loop retries on `HttpRequestException` with exponential backoff and jitter, stopping after the third attempt.

### Real-World Cases

- **HTTP API calls** — Retry on `429`, `503`, and network errors.
- **Database connections** — Retry transient connection failures.
- **Message queues** — Retry message consumption on transient broker errors.

### References

- Polly Resilience Pipelines with Exponential Backoff and Decorrelated Jitter — GitHub — https://github.com/Arasz/ai-raccoon/blob/main/docs/adr/0031-polly-resilience-pipelines.md
- Retry strategy — Microsoft.Extensions.Http.Resilience — Microsoft Learn (GitHub) — https://github.com/microsoftdocs/learn/blob/3b1c46a60dffd575ec3f94d27003fefbb24ffb52/learn-pr/aspnetcore/microservices-resiliency-aspnet-core/includes/2-application-infrastructure-resiliency.md

---

## 5. Fallback Values

### Definitions

- **Core Definition:** A fallback value is a **substitute result** returned when the primary operation fails, allowing the caller to continue with a sensible default or degraded outcome.
- **Technical Definition:** Fallback is a **reactive resilience strategy** that provides a substitute value or action when execution fails. It is typically used as a **last resort** after retries or other strategies have been exhausted, and may use cached data, defaults, or dynamically generated values. Polly’s `FallbackStrategy` implements this pattern with `ShouldHandle`, `FallbackAction`, and `OnFallback`.
- **Beginner-Friendly Explanation:** Fallback is **“if you can’t get the real thing, use this instead”** —like showing a generic avatar when a user’s photo fails to load.

### Purposes

- To **provide a substitute** when an operation fails.
- To **use cached data** when fresh data is unavailable.
- To **support graceful degradation** with a concrete value.
- To **call an alternative service** or endpoint.
- To **avoid returning `null`** or throwing to the caller.

### Syntax Rules and Structure

```csharp
var options = new FallbackStrategyOptions<UserAvatar>
{
    ShouldHandle = new PredicateBuilder<UserAvatar>()
        .Handle<HttpRequestException>()
        .HandleResult(r => r is null),
    FallbackAction = args => Outcome.FromResultAsValueTask(UserAvatar.Blank),
    OnFallback = args =>
    {
        logger.LogWarning(args.Outcome.Exception, "Fallback triggered");
        return default;
    }
};
```

**Component breakdown**

- **`ShouldHandle`** — Which exceptions/results trigger fallback.
- **`FallbackAction`** — Produces the substitute value.
- **`OnFallback`** — Optional side effect (logging, metrics).

#### Syntax Rules

- Define a clear predicate for when fallback applies.
- Use cached/default/dynamically generated values.
- Log fallback events at `Warning` level.
- Do not fallback on `OperationCanceledException` by default.
- Use fallback as a last resort after retries.

#### Constraints and Limitations

- Fallback values may be **stale or inaccurate**; communicate this to callers.
- Silent fallbacks can hide serious failures.

### Annotated Code Examples and Expected Outputs

#### Example: Fallback to Default Avatar

```csharp
using System;

public class UserAvatar
{
    public string Url { get; }
    public static UserAvatar Blank { get; } = new UserAvatar("blank.png");
    private UserAvatar(string url) => Url = url;
    public override string ToString() => Url;
}

class Program
{
    static UserAvatar FetchAvatar() => throw new HttpRequestException("Avatar service down");

    static UserAvatar GetAvatarWithFallback()
    {
        try { return FetchAvatar(); }
        catch (HttpRequestException ex)
        {
            Console.WriteLine($"Fallback triggered: {ex.Message}");
            return UserAvatar.Blank;
        }
    }

    static void Main() => Console.WriteLine(GetAvatarWithFallback());
}
```

**Expected Output**
```
Fallback triggered: Avatar service down
blank.png
```

**Why this output occurs:** The fallback returns a default avatar when the primary fetch fails, allowing the caller to continue.

### Real-World Cases

- **User profiles** — Default avatar when the image service is down.
- **Weather apps** — Cached weather data when the API is unavailable.
- **Configuration** — Default values when remote configuration fails.

### References

- Fallback resilience strategy — Polly (GitHub) — https://github.com/app-vnext/polly/blob/016dd909db880b9d8d07954ff5f4ea3cc1e11c67/docs/strategies/fallback.md
- Fallback Strategy — Polly (Mintlify) — https://mintlify.wiki/App-vNext/Polly/strategies/fallback

---

## 6. Error Logging

### Definitions

- **Core Definition:** Error logging is the practice of **recording exception details** (message, stack trace, context) to a persistent store for diagnosis and auditing.
- **Technical Definition:** .NET provides `ILogger<T>` as the standard logging abstraction, with providers for console, files, EventLog, Application Insights, and third-party frameworks like **Serilog**. Structured logging captures properties (not just strings) for queryability. Logging inside a `catch` block should **pass the exception object** so providers can persist it structurally.
- **Beginner-Friendly Explanation:** Error logging is **“writing down what went wrong”** so you can figure it out later.

### Purposes

- To **diagnose failures** with full context (message, stack trace, inner exception).
- To **audit** security and business-critical events.
- To **feed monitoring and alerting** systems.
- To **provide structured, queryable** log data.

### Syntax Rules and Structure

```csharp
public class OrderController
{
    private readonly ILogger<OrderController> _logger;
    public OrderController(ILogger<OrderController> logger) => _logger = logger;

    public void Process(Order order)
    {
        try { /* work */ }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to process order {OrderId}", order.Id);
            throw;
        }
    }
}
```

**Component breakdown**

- **`ILogger<T>`** — Standard abstraction; inject where needed.
- **`LogError(ex, ...)`** — Pass the exception object for structured logging.
- **Message templates** — Use `{Property}` placeholders for structured properties.

#### Syntax Rules

- **Always pass the exception** to the logger in `catch` blocks.
- Use **structured logging** (message templates, properties).
- Include context: what failed, who was affected, which operation.
- Use appropriate **log levels** (`Error`, `Warning`, `Critical`).
- Do not log sensitive data (passwords, tokens, PII).

#### Constraints and Limitations

- Logging every exception at `Error` can create noise.
- String concatenation prevents structured querying.

### Annotated Code Examples and Expected Outputs

#### Example: Structured Error Logging

```csharp
using System;
using Microsoft.Extensions.Logging;

class Program
{
    static void Main()
    {
        using var factory = LoggerFactory.Create(b => b.AddConsole());
        var logger = factory.CreateLogger<Program>();
        try
        {
            throw new InvalidOperationException("Payment gateway timeout");
        }
        catch (Exception ex)
        {
            logger.LogError(ex, "Payment failed for order {OrderId}", 42);
        }
    }
}
```

**Expected Output (approximate)**
```
fail: Program[0]
      Payment failed for order 42
      System.InvalidOperationException: Payment gateway timeout
         at Program.Main(...)
```

**Why this output occurs:** `LogError` records the exception with its stack trace and the structured `OrderId` property.

### Real-World Cases

- **API middleware** — Log unhandled exceptions with request context.
- **Background services** — Log job failures with job IDs.
- **Security** — Log authentication and authorization failures.

### References

- Logging guidelines — Altinn Studio — https://docs.altinn.studio/en/community/contributing/handbook/back-end/logging/
- Logging exceptions inside `catch` — GitHub — https://github.com

---

## 7. Error Monitoring

### Definitions

- **Core Definition:** Error monitoring is the **continuous observation** of application errors through dashboards, alerts, and telemetry, enabling proactive detection and response.
- **Technical Definition:** Error monitoring integrates with observability platforms such as **Application Insights** and **OpenTelemetry**. `ExceptionTelemetry` (Application Insights) records exceptions; OpenTelemetry captures exceptions as **span events** with `Activity` status set to `Error`. Metrics (exception rate, counts) and alerts enable operations teams to detect anomalies.
- **Beginner-Friendly Explanation:** Error monitoring is **“watching the smoke alarm”** —not just writing down fires, but noticing when they happen and responding.

### Purposes

- To **detect errors in production** before users report them.
- To **alert on-call engineers** for critical failures.
- To **track error trends** and rates over time.
- To **correlate exceptions** with traces and metrics.
- To **measure the impact** of resilience patterns.

### Syntax Rules and Structure

```csharp
using System.Diagnostics;
using OpenTelemetry.Trace;

using var activity = ActivitySource.StartActivity("ProcessOrder");
try
{
    // operation
}
catch (Exception ex)
{
    activity?.SetStatus(ActivityStatusCode.Error, ex.Message);
    activity?.RecordException(ex);
    throw;
}
```

**Component breakdown**

- **`Activity`** — OpenTelemetry span.
- **`SetStatus(Error)`** — Marks the span as failed.
- **`RecordException`** — Attaches the exception to the span.
- **`ExceptionTelemetry`** — Application Insights legacy API; migrate to OpenTelemetry.

#### Syntax Rules

- Use **OpenTelemetry** for cross-platform monitoring.
- Record exceptions on spans with `Error` status.
- Emit metrics for exception counts and rates.
- Alert on thresholds (e.g., exception rate > baseline).

#### Constraints and Limitations

- Monitoring without logging lacks context; combine both.
- High cardinality in metrics can be expensive.

### Annotated Code Examples and Expected Outputs

#### Example: Recording an Exception on an Activity

```csharp
using System;
using System.Diagnostics;

class Program
{
    static readonly ActivitySource Source = new("MyApp");

    static void Main()
    {
        using var activity = Source.StartActivity("RiskyOperation");
        try
        {
            throw new InvalidOperationException("Simulated failure");
        }
        catch (Exception ex)
        {
            activity?.SetStatus(ActivityStatusCode.Error, ex.Message);
            activity?.RecordException(ex);
            Console.WriteLine($"Recorded: {ex.Message} on span {activity?.DisplayName}");
        }
    }
}
```

**Expected Output**
```
Recorded: Simulated failure on span RiskyOperation
```

**Why this output occurs:** The exception is attached to the OpenTelemetry activity and the span status is set to `Error`, making it visible in monitoring dashboards.

### Real-World Cases

- **Cloud applications** — Application Insights / Azure Monitor alerts.
- **Kubernetes** — OpenTelemetry traces exported to Jaeger/Tempo.
- **SLA monitoring** — Exception rate dashboards.

### References

- Application Insights FAQ — ExceptionTelemetry and OpenTelemetry — Microsoft Learn — https://learn.microsoft.com/en-ie/azure/azure-monitor/app/application-insights-faq#application-insights-api-for-custom-events-and-metrics
- OpenTelemetry exception recording — Microsoft Learn

---

## 8. Exception Pattern Best Practices

### Definitions

- **Core Definition:** Exception pattern best practices are the guidelines for **choosing and combining** error-handling strategies so that systems are reliable, diagnosable, and resilient.
- **Technical Definition:** Best practices derive from .NET exception guidance and resilience engineering: **use `try`/`catch`/`finally` to recover or release resources**; **handle common conditions to avoid exceptions**; **catch cancellation and async exceptions**; **design classes so exceptions can be avoided**; **restore state when methods don’t complete**; **capture and rethrow exceptions properly**. For patterns, **fail fast on defects**, **fail safe on critical state**, **degrade gracefully on non-critical dependencies**, **retry transient faults**, **fallback as a last resort**, and **always log and monitor**.
- **Beginner-Friendly Explanation:** The best practices are: **pick the right pattern for the right failure**, **don’t hide errors silently**, **always clean up**, **log and monitor**, and **test your resilience**.

### Guidelines and Structure

#### General Guidelines

```csharp
// 1. Fail fast on invalid input
ArgumentNullException.ThrowIfNull(order);

// 2. Fail safe with transactions
using var tx = connection.BeginTransaction();
try { /* work */ tx.Commit(); } catch { tx.Rollback(); throw; }

// 3. Degrade gracefully for non-critical dependencies
try { recs = await _recs.GetAsync(); }
catch { recs = Array.Empty<Recommendation>(); }

// 4. Retry transient failures with backoff and jitter
await pipeline.ExecuteAsync(ct => httpClient.GetAsync(url, ct));

// 5. Fallback to cached/default values
return fallback.Execute(() => Fetch());

// 6. Log with the exception object and structured properties
logger.LogError(ex, "Operation {Operation} failed", name);

// 7. Monitor with OpenTelemetry spans and metrics
activity?.RecordException(ex);
```

**Component breakdown**

- **Fail fast** — Guards and immediate throws.
- **Fail safe** — Transactions and cleanup.
- **Graceful degradation** — Isolate non-critical calls.
- **Retry** — Exponential backoff + jitter.
- **Fallback** — Substitute values.
- **Logging** — Structured, context-rich.
- **Monitoring** — Spans, metrics, alerts.

#### Rules and Constraints

- **Do** choose the pattern based on criticality and failure type.
- **Do** combine patterns (retry → fallback → degrade).
- **Do** always log and monitor.
- **Do** clean up resources in `finally`/`using`.
- **Do not** swallow exceptions silently.
- **Do not** retry non-idempotent operations.
- **Do not** fail fast for expected transient faults in distributed systems.

### Annotated Code Examples and Expected Outputs

#### Example: Combined Pattern Pipeline

```csharp
using System;
using System.Threading.Tasks;

class Program
{
    static async Task<string> FetchWithResilienceAsync()
    {
        for (int attempt = 1; attempt <= 3; attempt++)
        {
            try
            {
                return await FetchAsync();   // may fail
            }
            catch (Exception ex) when (attempt < 3)
            {
                Console.WriteLine($"Retry {attempt}: {ex.Message}");
                await Task.Delay(TimeSpan.FromMilliseconds(100 * Math.Pow(2, attempt)));
            }
        }
        Console.WriteLine("Fallback: returning cached value");
        return "cached";
    }

    static Task<string> FetchAsync() => throw new HttpRequestException("Network error");

    static async Task Main() => Console.WriteLine(await FetchWithResilienceAsync());
}
```

**Expected Output**
```
Retry 1: Network error
Retry 2: Network error
Fallback: returning cached value
cached
```

**Why this output occurs:** Retries are attempted twice, then the fallback value (`"cached"`) is returned, combining retry and fallback patterns.

### Real-World Cases

- **Microservices** — Retry + fallback + graceful degradation.
- **APIs** — Fail fast on invalid requests, log, and monitor.
- **Financial systems** — Fail safe transactions, structured logging, and alerting.

### References

- Best practices for exceptions — Microsoft Learn (GitHub) — https://raw.githubusercontent.com/dotnet/docs/86590e2ef30ffff18842e7bb28036e9738630645/docs/standard/exceptions/best-practices-for-exceptions.md
- Resiliency approaches — Microsoft Learn (GitHub) — https://github.com/microsoftdocs/learn/blob/3b1c46a60dffd575ec3f94d27003fefbb24ffb52/learn-pr/aspnetcore/microservices-resiliency-aspnet-core/includes/2-application-infrastructure-resiliency.md

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| `ExceptionTelemetry` (Application Insights) | **Legacy**; migrate to OpenTelemetry for cross-platform monitoring. |
| `System.ApplicationException` | **Discouraged** as a base for custom exceptions. |
| `throw ex;` | **Resets stack trace**; use `throw;`. |
| Retry without jitter | **Risk of thundering herd**; use jitter. |
| `OperationCanceledException` in fallback | **Excluded by default** from fallback handling. |
| Fail fast in microservices | **Can cascade failures**; prefer graceful degradation for non-critical services. |
| Structured logging with string concatenation | **Loses queryability**; use message templates. |

---

## References (Consolidated)

- Resiliency approaches — Fail fast vs. graceful degradation — Microsoft Learn (GitHub) — https://github.com/microsoftdocs/learn/blob/3b1c46a60dffd575ec3f94d27003fefbb24ffb52/learn-pr/aspnetcore/microservices-resiliency-aspnet-core/includes/2-application-infrastructure-resiliency.md
- Best practices for exceptions — Microsoft Learn (GitHub) — https://raw.githubusercontent.com/dotnet/docs/86590e2ef30ffff18842e7bb28036e9738630645/docs/standard/exceptions/best-practices-for-exceptions.md
- Polly Resilience Pipelines with Exponential Backoff and Decorrelated Jitter — GitHub — https://github.com/Arasz/ai-raccoon/blob/main/docs/adr/0031-polly-resilience-pipelines.md
- Fallback resilience strategy — Polly (GitHub) — https://github.com/app-vnext/polly/blob/016dd909db880b9d8d07954ff5f4ea3cc1e11c67/docs/strategies/fallback.md
- Fallback Strategy — Polly (Mintlify) — https://mintlify.wiki/App-vNext/Polly/strategies/fallback
- Logging guidelines — Altinn Studio — https://docs.altinn.studio/en/community/contributing/handbook/back-end/logging/
- Application Insights FAQ — ExceptionTelemetry and OpenTelemetry — Microsoft Learn — https://learn.microsoft.com/en-ie/azure/azure-monitor/app/application-insights-faq
- Logging exceptions inside `catch` — GitHub — https://github.com
- OpenTelemetry exception recording — Microsoft Learn — https://learn.microsoft.com