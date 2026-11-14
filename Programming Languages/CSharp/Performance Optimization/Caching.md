# Caching in C# — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Caching is the practice of storing the results of expensive operations — database queries, HTTP calls, complex computations — in a fast-access storage layer so that subsequent requests for the same data can be served without repeating the original work.

**Technical Definition**
Caching in .NET is implemented through layered abstractions and storage backends. At the application level, `IMemoryCache` stores objects in the process's managed heap, providing nanosecond-to-microsecond access times at the cost of per-process isolation and memory consumption. At the distributed level, `IDistributedCache` abstracts external stores — Redis, SQL Server, NCache, or an in-memory distributed cache — that are shared across application instances, enabling cache coherence in multi-server deployments. Redis, accessed either through the `IDistributedCache` abstraction (`Microsoft.Extensions.Caching.StackExchangeRedis`) or the lower-level `StackExchange.Redis` client, is the dominant distributed cache in .NET architectures. Introduced in .NET 9, `HybridCache` unifies in-memory (L1) and distributed (L2) caching behind a single `GetOrCreateAsync` API, with built-in stampede protection, configurable serialization, and tag-based invalidation. Caching is not a single technique but a discipline encompassing key design, expiration policy (absolute, sliding, or both), eviction priority, size limits, invalidation strategy, and monitoring.

**Beginner-Friendly Explanation**
Caching is "keeping a copy of something expensive close at hand." If your application queries a database for the same product list every second, you store that list in memory or in Redis for a few minutes so the database is not hit repeatedly. The first request is slow (cache miss); subsequent requests are fast (cache hit). The hard parts are knowing when the copy is stale (invalidation) and preventing many simultaneous requests from all trying to refresh the copy at once (stampede).

---

### Key Characteristics

- **Speed vs. freshness trade-off** — Cached data is fast but may be stale; the expiration and invalidation policy determines the acceptable staleness window.
- **Layered** — In-process memory (fastest, per-node), distributed cache (shared, network-hop), and hybrid caching (both layers coordinated) serve different deployment topologies.
- **Eviction-aware** — `IMemoryCache` evicts entries under memory pressure based on priority; it does not automatically enforce a size limit unless `SizeLimit` is configured.
- **Stampede-prone** — When a popular entry expires, multiple concurrent requests may simultaneously attempt to regenerate it; `HybridCache` and manual locking patterns address this.
- **Key-design-sensitive** — Cache keys must be deterministic, namespaced, and free of unbounded user input; "Don't insert external input into the cache" is a documented guideline.
- **Measurable** — Hit ratio, miss rate, eviction count, and latency are the primary metrics for cache effectiveness.

---

### Prerequisites

- Working knowledge of C# classes, interfaces, and dependency injection in ASP.NET Core.
- Familiarity with `Microsoft.Extensions.Caching.Memory` and `Microsoft.Extensions.Caching.Distributed` namespaces.
- Basic understanding of serialization (JSON, protobuf) for distributed cache values.
- A Redis server or Azure Managed Redis instance for distributed caching scenarios.
- Awareness of the difference between a cache miss and a stale cache hit.
- Familiarity with `async`/`await` for asynchronous cache operations.

---

### Related Programming Areas

- **Performance fundamentals** — Latency, throughput, and resource utilization.
- **Entity Framework Core** — Second-level caching and compiled queries.
- **ASP.NET Core output caching** — Whole-response caching middleware.
- **Memory optimization** — Cache size limits and eviction pressure.
- **Distributed systems** — Cache coherence, replication, and failover.
- **Observability** — Cache hit ratio metrics and distributed tracing.

---

### Core Concepts / Features

1. Caching (overarching discipline and patterns)
2. `IMemoryCache`
3. `IDistributedCache`
4. Redis
5. Cache Invalidation
6. Cache Best Practices

Each concept follows the same uniform structure: definitions, purposes (each beginning with "To"), syntax rules and structure, annotated code examples with expected outputs, real-world cases, and references.

---

## 1. Caching (Overarching Discipline and Patterns)

### Definitions

**Core Definition**
Caching is the storage of frequently accessed data in a high-speed layer to reduce latency and load on the underlying data source.

**Technical Definition**
Caching in .NET is organized around the **cache-aside pattern**: the application checks the cache first; on a hit, it returns the cached value; on a miss, it fetches from the source, stores the result in the cache with an expiration policy, and returns it. This pattern is the foundation of `IMemoryCache`, `IDistributedCache`, and `HybridCache` usage. The **read-through** and **write-through** patterns are less common in .NET but are supported by some distributed cache providers. The **write-behind** pattern is not natively supported by `IDistributedCache` and requires custom implementation. All caching patterns must address **expiration** (absolute, sliding, or both), **eviction priority** (Low/Normal/High/NeverRemove), and **size limits** when memory is constrained. The cache is a scarce resource; the ASP.NET Core runtime does not limit `IMemoryCache` size based on memory pressure — the developer must configure `SizeLimit` and `Size` explicitly.

**Beginner-Friendly Explanation**
Caching is "check the fridge before going to the grocery store." The cache-aside pattern is the standard recipe: look in the cache first; if the item is there, use it; if not, go to the database, put a copy in the cache, and use it. Expiration is the "best before" date on the cached copy. Invalidation is throwing the copy away early because you know the source changed.

---

### Purposes

- To reduce latency by serving data from a faster storage layer.
- To reduce load on databases, APIs, and other downstream services.
- To improve scalability by decoupling request volume from backend capacity.
- To smooth out bursts of traffic through cached responses.
- To provide a predictable performance envelope for read-heavy workloads.
- To enable offline or degraded-mode operation when the source is unavailable.

---

### Syntax Rules and Structure

#### General Cache-Aside Pattern

```csharp
// 1. Try to get from cache
if (!cache.TryGetValue(key, out var value))
{
    // 2. Cache miss: fetch from source
    value = await source.GetAsync(key);

    // 3. Store in cache with expiration
    cache.Set(key, value, new MemoryCacheEntryOptions
    {
        AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(10),
        SlidingExpiration = TimeSpan.FromMinutes(2),
        Priority = CacheItemPriority.High,
        Size = 1
    });
}

return value;
```

**Component breakdown**

- `TryGetValue` — Checks the cache without throwing on a miss.
- `source.GetAsync` — The expensive operation to avoid repeating.
- `AbsoluteExpirationRelativeToNow` — Hard TTL; the entry expires regardless of access.
- `SlidingExpiration` — Extends the TTL on each access; pair with an absolute cap to prevent "immortal" hot keys.
- `Priority` — Eviction hint under memory pressure.
- `Size` — Required when `SizeLimit` is configured on the cache.

#### Syntax Rules

- Always have a fallback to fetch from the source; never depend on the cache being populated.
- Do not use unbounded external input as cache keys; use structured, namespaced keys (`product:42`).
- Set an expiration on every entry; an unexpiring cache is a memory leak.
- When `SizeLimit` is configured, every entry must specify `Size`; mixing sized and unsized entries on a shared cache can cause failures.
- Use `GetOrCreate` / `GetOrCreateAsync` extension methods to encapsulate the cache-aside logic and avoid duplication.

#### Constraints and Limitations

- `IMemoryCache` is per-process; it does not provide coherence across multiple application instances.
- `IDistributedCache` values are stored as `byte[]`; complex objects must be serialized and deserialized, adding CPU and allocation overhead.
- Sliding expiration without an absolute cap can keep hot entries alive indefinitely.
- The cache is not a source of truth; treat every cached value as potentially stale.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Cache-Aside with IMemoryCache

```csharp
public class ProductService
{
    private readonly IMemoryCache _cache;
    private readonly IProductRepository _repository;

    public ProductService(IMemoryCache cache, IProductRepository repository)
    {
        _cache = cache;
        _repository = repository;
    }

    public async Task<Product?> GetProductAsync(int id)
    {
        var key = $"product:{id}";

        if (_cache.TryGetValue(key, out Product? product))
        {
            Console.WriteLine("Cache hit");
            return product;
        }

        Console.WriteLine("Cache miss — fetching from database");
        product = await _repository.GetByIdAsync(id);

        if (product is not null)
        {
            _cache.Set(key, product, new MemoryCacheEntryOptions
            {
                AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(5),
                Size = 1
            });
        }

        return product;
    }
}
```

**Expected Output (first call)**

```
Cache miss — fetching from database
```

**Expected Output (second call within 5 minutes)**

```
Cache hit
```

**Why this output occurs**

- The first call finds no entry and fetches from the repository, then stores the result.
- The second call retrieves the cached object without touching the database.

---

#### Example 2: GetOrCreateAsync Encapsulation

```csharp
public async Task<Product?> GetProductAsync(int id)
{
    return await _cache.GetOrCreateAsync($"product:{id}", async entry =>
    {
        entry.AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(5);
        entry.SlidingExpiration = TimeSpan.FromMinutes(1);
        entry.Priority = CacheItemPriority.High;
        entry.Size = 1;
        return await _repository.GetByIdAsync(id);
    });
}
```

**Expected Output (conceptual)**

```
The GetOrCreateAsync method checks the cache, invokes the factory on a miss,
and stores the result with the specified expiration and priority.
```

**Why this output occurs**

- The extension method encapsulates the try-get/fetch/set pattern, reducing duplication and ensuring consistent options.

---

### Real-World Cases

- **E-commerce product catalogs** — Product details and price lookups cached for minutes to hours.
- **Configuration and feature flags** — Small, hot values cached in memory.
- **API response caching** — External API results cached to respect rate limits.
- **Session state** — Distributed cache for session data in multi-server deployments.

---

### References

- Cache in-memory in ASP.NET Core — Microsoft Learn - https://learn.microsoft.com/en-us/aspnet/core/performance/caching/memory
- Caching in .NET — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/core/extensions/caching
- Caching guidance — Azure Architecture Center - https://learn.microsoft.com/en-us/azure/architecture/best-practices/caching

---

## 2. `IMemoryCache`

### Definitions

**Core Definition**
`IMemoryCache` is the ASP.NET Core in-process cache abstraction that stores objects in the application's managed heap for fast, local access.

**Technical Definition**
`Microsoft.Extensions.Caching.Memory.IMemoryCache` is the recommended in-memory cache for ASP.NET Core, replacing `System.Runtime.Caching.MemoryCache`. It integrates natively with dependency injection and provides thread-safe accessors (`TryGetValue`, `Get`, `Set`, `Remove`, `GetOrCreate`, `GetOrCreateAsync`). Entries support absolute expiration, sliding expiration, eviction priority (`Low`, `Normal`, `High`, `NeverRemove`), and size limits via `SetSize`, `Size`, and `SizeLimit`. Critically, the ASP.NET Core runtime does not limit `IMemoryCache` size based on memory pressure — the developer must configure `SizeLimit` and specify `Size` on every entry when a limit is set. Using `SetSize` on a shared cache consumed by other frameworks can cause failures if those frameworks do not specify sizes.

**Beginner-Friendly Explanation**
`IMemoryCache` is "the cache that lives inside your application's memory." It is the fastest cache available because reading from it does not involve the network. It is ideal for single-server applications or for node-local data that does not need to be shared. But it is not free: it consumes RAM, it is not shared across servers, and you must tell it when to evict entries and how large the cache is allowed to grow.

---

### Purposes

- To provide the fastest possible cache access (no network hop).
- To store small, hot objects such as lookups, configuration, and per-request computations.
- To integrate cleanly with ASP.NET Core dependency injection.
- To support fine-grained expiration and eviction policies.
- To reduce database load for read-heavy single-instance workloads.
- To serve as the L1 layer in hybrid caching architectures.

---

### Syntax Rules and Structure

#### General Syntax

```csharp
// Registration
builder.Services.AddMemoryCache();

// Injection
public class MyService
{
    private readonly IMemoryCache _cache;
    public MyService(IMemoryCache cache) => _cache = cache;
}

// Set with options
_cache.Set("key", value, new MemoryCacheEntryOptions
{
    AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(10),
    SlidingExpiration = TimeSpan.FromMinutes(2),
    Priority = CacheItemPriority.High,
    Size = 1
});

// Get or create
var value = await _cache.GetOrCreateAsync("key", async entry =>
{
    entry.AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(5);
    return await FetchAsync();
});
```

**Component breakdown**

- `AddMemoryCache()` — Registers `IMemoryCache` as a singleton.
- `MemoryCacheEntryOptions` — Configures expiration, priority, and size.
- `AbsoluteExpirationRelativeToNow` — Hard TTL from the current time.
- `SlidingExpiration` — TTL extended on each access.
- `Priority` — Eviction hint (`Low`, `Normal`, `High`, `NeverRemove`).
- `Size` — Required when `SizeLimit` is configured.

#### Syntax Rules

- Always set an expiration; unexpiring entries are a memory leak.
- When `SizeLimit` is configured, every entry must specify `Size`; mixing sized and unsized entries on a shared cache can cause the application to fail.
- Do not use external, unbounded user input as cache keys.
- Use `GetOrCreateAsync` to avoid duplicating the cache-aside logic.
- Create a dedicated cache singleton if you need size limits and the shared cache is used by other frameworks.

#### Constraints and Limitations

- The cache is per-process; it is not coherent across multiple application instances.
- The runtime does not trim the cache automatically based on memory pressure; size limits are the developer's responsibility.
- Sliding expiration without an absolute cap can keep hot entries alive indefinitely.
- Large objects cached in memory increase GC pressure and reduce available memory for the application.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Absolute and Sliding Expiration

```csharp
_cache.Set("time", DateTime.UtcNow, new MemoryCacheEntryOptions
{
    AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(10),
    SlidingExpiration = TimeSpan.FromMinutes(2)
});

// First access after 1 minute: sliding TTL resets
var cached = _cache.Get<DateTime>("time");

// After 10 minutes from the initial set: entry expires regardless of access
```

**Expected Output (conceptual)**

```
Within the sliding window: cache hit, sliding TTL extended.
After the absolute expiration: cache miss, entry evicted.
```

**Why this output occurs**

- Sliding expiration extends the TTL on each access, up to the absolute expiration ceiling.
- The absolute expiration is a hard limit that cannot be extended.

---

#### Example 2: Size Limit with Dedicated Cache

```csharp
// Dedicated cache singleton with a size limit
var cache = new MemoryCache(new MemoryCacheOptions
{
    SizeLimit = 1024  // total size units (app-defined)
});

cache.Set("a", "value", new MemoryCacheEntryOptions { Size = 1 });
```

**Expected Output (conceptual)**

```
Each entry declares its size (1 unit).
The cache evicts entries to stay within the 1024-unit limit.
```

**Why this output occurs**

- `SizeLimit` enables size-based eviction; every entry must declare `Size`.
- The units are application-defined (e.g., number of items, approximate bytes).

---

### Real-World Cases

- **Lookup tables** — Country codes, currency rates, and category lists cached in memory.
- **Feature flags** — Small configuration objects read on every request.
- **Per-request computations** — Expensive calculations reused within a request pipeline.
- **Session affinity scenarios** — Node-local session data where stickiness is guaranteed.

---

### References

- Cache in-memory in ASP.NET Core — Microsoft Learn - https://learn.microsoft.com/en-us/aspnet/core/performance/caching/memory
- IMemoryCache interface — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.caching.memory.imemorycache
- MemoryCacheEntryOptions — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.caching.memory.memorycacheentryoptions

---

## 3. `IDistributedCache`

### Definitions

**Core Definition**
`IDistributedCache` is the ASP.NET Core abstraction for a cache shared by multiple application servers, providing a consistent API for storing and retrieving byte arrays and strings.

**Technical Definition**
`Microsoft.Extensions.Caching.Distributed.IDistributedCache` defines asynchronous methods — `GetAsync`, `SetAsync`, `RefreshAsync`, `RemoveAsync` — for manipulating cache entries. Values are stored as `byte[]`; strings can be stored directly via `GetStringAsync`/`SetStringAsync`, while complex objects require serialization (typically JSON). Implementations include `AddDistributedMemoryCache` (in-process, for development and testing), `AddStackExchangeRedisCache` (Redis), `AddDistributedSqlServerCache` (SQL Server), and `AddNCacheDistributedCache` (NCache). The abstraction enables swapping cache implementations without changing business logic. Because every access involves serialization and often a network hop, `IDistributedCache` is slower than `IMemoryCache` but provides cache coherence across instances.

**Beginner-Friendly Explanation**
`IDistributedCache` is "the shared cache for multiple servers." If you run three instances of your API, `IMemoryCache` gives each instance its own copy — one instance might have a fresh value while another has a stale one. `IDistributedCache` uses an external store (usually Redis) so all instances see the same data. The cost is that every read and write goes over the network and must serialize the data.

---

### Purposes

- To provide cache coherence across multiple application instances.
- To abstract away the specific distributed cache implementation (Redis, SQL Server, NCache).
- To support session state, output caching, and data protection key storage in multi-server deployments.
- To enable cache sharing without coupling business logic to a specific provider.
- To provide asynchronous cache operations for scalable I/O.

---

### Syntax Rules and Structure

#### General Syntax

```csharp
// Registration (Redis)
builder.Services.AddStackExchangeRedisCache(options =>
{
    options.Configuration = "localhost:6379";
    options.InstanceName = "MyApp:";
});

// Injection
public class CacheService
{
    private readonly IDistributedCache _cache;
    public CacheService(IDistributedCache cache) => _cache = cache;

    public async Task SetAsync(string key, string value)
    {
        await _cache.SetStringAsync(key, value, new DistributedCacheEntryOptions
        {
            AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(10)
        });
    }

    public async Task<string?> GetAsync(string key)
        => await _cache.GetStringAsync(key);

    public async Task RemoveAsync(string key)
        => await _cache.RemoveAsync(key);

    public async Task RefreshAsync(string key)
        => await _cache.RefreshAsync(key);
}
```

**Component breakdown**

- `AddStackExchangeRedisCache` — Registers Redis as the `IDistributedCache` implementation.
- `InstanceName` — Optional key prefix to avoid collisions between applications.
- `DistributedCacheEntryOptions` — Configures absolute and sliding expiration.
- `SetStringAsync` / `GetStringAsync` — String convenience methods.
- `RefreshAsync` — Resets the sliding expiration of an entry.

#### Syntax Rules

- Serialize complex objects to JSON (or another format) before storing; `IDistributedCache` stores `byte[]`.
- Use `DistributedCacheEntryOptions` to set expiration on every entry.
- Use a consistent key namespace (`MyApp:user:42`) to avoid collisions.
- Call `RemoveAsync` on writes to invalidate stale entries.
- Prefer `SetStringAsync`/`GetStringAsync` for strings; use `SetAsync`/`GetAsync` with `byte[]` for binary data.

#### Constraints and Limitations

- Every operation involves serialization and often a network round trip; `IDistributedCache` is slower than `IMemoryCache`.
- The abstraction exposes only basic key-value operations; advanced Redis data structures (hashes, lists, sorted sets) are not available.
- Large values (> 100 KB) may benefit from compression or streaming.
- The distributed cache is a single point of failure unless the underlying store is replicated.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Caching a Complex Object with JSON

```csharp
public class UserCacheService
{
    private readonly IDistributedCache _cache;
    private static readonly JsonSerializerOptions JsonOptions = new()
    {
        PropertyNameCaseInsensitive = true
    };

    public UserCacheService(IDistributedCache cache) => _cache = cache;

    public async Task CacheUserAsync(User user, int ttlSeconds = 3600)
    {
        var json = JsonSerializer.Serialize(user, JsonOptions);
        await _cache.SetStringAsync($"user:{user.Id}", json,
            new DistributedCacheEntryOptions
            {
                AbsoluteExpirationRelativeToNow = TimeSpan.FromSeconds(ttlSeconds)
            });
    }

    public async Task<User?> GetUserAsync(int userId)
    {
        var json = await _cache.GetStringAsync($"user:{userId}");
        return json is null ? null : JsonSerializer.Deserialize<User>(json, JsonOptions);
    }

    public async Task InvalidateUserAsync(int userId)
        => await _cache.RemoveAsync($"user:{userId}");
}
```

**Expected Output (conceptual)**

```
CacheUserAsync serializes the user to JSON and stores it in Redis.
GetUserAsync retrieves the JSON and deserializes it.
InvalidateUserAsync removes the entry, forcing the next read to hit the source.
```

**Why this output occurs**

- `IDistributedCache` stores `byte[]`; JSON serialization converts the object to bytes.
- `RemoveAsync` invalidates the entry when the underlying data changes.

---

#### Example 2: Cache-Aside with IDistributedCache

```csharp
public async Task<User?> GetOrFetchAsync(int userId)
{
    var cached = await _cache.GetStringAsync($"user:{userId}");
    if (cached is not null)
        return JsonSerializer.Deserialize<User>(cached);

    var user = await _repository.GetByIdAsync(userId);
    if (user is not null)
    {
        await _cache.SetStringAsync($"user:{userId}",
            JsonSerializer.Serialize(user),
            new DistributedCacheEntryOptions
            {
                AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(10)
            });
    }
    return user;
}
```

**Expected Output (conceptual)**

```
Cache hit: returns deserialized user.
Cache miss: fetches from repository, serializes, stores, and returns.
```

**Why this output occurs**

- The cache-aside pattern checks the distributed cache first, then falls back to the source.

---

### Real-World Cases

- **Multi-server APIs** — Shared user profiles, permissions, and configuration across instances.
- **Session state** — ASP.NET Core session middleware backed by Redis.
- **Output caching** — Whole-response caching with a distributed store for scale-out.
- **Data protection keys** — Shared key ring for encrypting cookies and tokens across instances.

---

### References

- Distributed caching in ASP.NET Core — Microsoft Learn - https://learn.microsoft.com/en-us/aspnet/core/performance/caching/distributed
- IDistributedCache interface — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.caching.distributed.idistributedcache
- DistributedCacheEntryOptions — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.caching.distributed.distributedcacheentryoptions

---

## 4. Redis

### Definitions

**Core Definition**
Redis is an open-source, in-memory data structure store used as a distributed cache, message broker, and database, accessed in .NET through `StackExchange.Redis` or the `IDistributedCache` Redis integration.

**Technical Definition**
Redis stores data in memory with optional persistence (RDB snapshots and AOF logs). It supports strings, hashes, lists, sets, sorted sets, streams, and probabilistic data structures. In .NET, there are two primary access paths: the low-level `StackExchange.Redis` client, which exposes the full command surface (atomic operations, transactions, Lua scripting, pub/sub, pipelining), and `Microsoft.Extensions.Caching.StackExchangeRedis`, which adapts Redis to `IDistributedCache` for simple key-value caching where values are opaque byte arrays. The `ConnectionMultiplexer` is designed to be created once and reused for the lifetime of the application; concurrent threads can share the same connection. Redis is the most common distributed cache in .NET architectures due to its low latency (sub-millisecond for local reads) and high throughput.

**Beginner-Friendly Explanation**
Redis is "the shared cache server." It is a separate process (or managed service) that stores data in memory and answers requests very quickly. In .NET, you either use the `IDistributedCache` abstraction for simple key-value caching, or you use `StackExchange.Redis` directly when you need advanced features like atomic increments, sorted sets, or pub/sub.

---

### Purposes

- To provide a high-performance, shared cache across application instances.
- To reduce database load and API latency in distributed systems.
- To support advanced caching patterns (leaderboards, rate limiting, queues) through native data structures.
- To provide a durable, replicated cache with configurable persistence.
- To enable atomic operations and Lua scripting for complex cache logic.
- To serve as the L2 layer in hybrid caching architectures.

---

### Syntax Rules and Structure

#### General Syntax (StackExchange.Redis)

```csharp
using StackExchange.Redis;

// Create the multiplexer once and reuse it
var redis = await ConnectionMultiplexer.ConnectAsync("localhost:6379");
var db = redis.GetDatabase();

// String operations
await db.StringSetAsync("user:1:name", "Alice", TimeSpan.FromMinutes(10));
var name = await db.StringGetAsync("user:1:name");

// Hash operations
await db.HashSetAsync("user:1", new HashEntry[]
{
    new("email", "alice@example.com"),
    new("role", "admin")
});
var entries = await db.HashGetAllAsync("user:1");

// Atomic increment (rate limiting)
await db.StringIncrementAsync("rate:user:1", 1);
```

**Component breakdown**

- `ConnectionMultiplexer.ConnectAsync` — Establishes the shared Redis connection.
- `GetDatabase()` — Returns an `IDatabase` for cache operations.
- `StringSetAsync` / `StringGetAsync` — Basic key-value operations.
- `HashSetAsync` / `HashGetAllAsync` — Hash operations for structured objects.
- `StringIncrementAsync` — Atomic counter for rate limiting or metrics.

#### General Syntax (IDistributedCache with Redis)

```csharp
builder.Services.AddStackExchangeRedisCache(options =>
{
    options.Configuration = "localhost:6379";
    options.InstanceName = "MyApp:";
});
```

- `AddStackExchangeRedisCache` — Registers Redis as `IDistributedCache`.
- `InstanceName` — Key prefix for namespacing.

#### Syntax Rules

- Create the `ConnectionMultiplexer` once and reuse it; do not create a new connection per operation.
- Use structured keys (`user:100`, `orders:100`) to avoid collisions.
- Set an expiration on every cached entry.
- Use `StringIncrementAsync` for atomic counters; do not read-modify-write with separate commands.
- For `IDistributedCache`, serialize complex objects to JSON before storing.

#### Constraints and Limitations

- Redis is a single-threaded command processor; long-running commands (e.g., `KEYS`) can block the server.
- `IDistributedCache` exposes only basic key-value operations; advanced data structures require `StackExchange.Redis`.
- Network latency and serialization overhead make Redis slower than `IMemoryCache`.
- The `ConnectionMultiplexer` must be disposed gracefully on application shutdown.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Cache-Aside with StackExchange.Redis

```csharp
public async Task<string?> GetItemAsync(string itemKey)
{
    var cached = await db.StringGetAsync(itemKey);
    if (cached.HasValue)
        return cached;

    var value = await FetchFromSourceAsync(itemKey);
    if (value is not null)
    {
        await db.StringSetAsync(itemKey, value, TimeSpan.FromMinutes(5));
    }
    return value;
}
```

**Expected Output (conceptual)**

```
Cache hit: returns the cached string.
Cache miss: fetches from the source, stores in Redis with a 5-minute TTL, and returns.
```

**Why this output occurs**

- `StringGetAsync` checks Redis first.
- On a miss, the source is queried and the result is stored with an expiration.

---

#### Example 2: Atomic Counter for Rate Limiting

```csharp
var count = await db.StringIncrementAsync($"rate:{userId}", 1);
if (count == 1)
{
    await db.KeyExpireAsync($"rate:{userId}", TimeSpan.FromMinutes(1));
}
if (count > 100)
{
    throw new RateLimitExceededException();
}
```

**Expected Output (conceptual)**

```
Each request atomically increments the counter.
The first request sets a 1-minute expiration.
When the count exceeds 100, the rate limit is enforced.
```

**Why this output occurs**

- `StringIncrementAsync` is atomic, preventing race conditions.
- `KeyExpireAsync` sets the window duration.

---

### Real-World Cases

- **API response caching** — JSON responses cached as Redis strings with TTLs.
- **Rate limiting** — Atomic counters with expiration windows.
- **Leaderboards** — Sorted sets for real-time rankings.
- **Session state** — Shared session store for multi-server ASP.NET Core apps.
- **Pub/sub messaging** — Lightweight event distribution across instances.

---

### References

- StackExchange.Redis documentation - https://stackexchange.github.io/StackExchange.Redis/
- Caching guidance — Azure Architecture Center (Redis client guidance) - https://learn.microsoft.com/en-us/azure/architecture/best-practices/caching
- Redis distributed caching in ASP.NET Core — Microsoft Learn - https://learn.microsoft.com/en-us/aspnet/core/performance/caching/distributed
- Redis cache-aside with StackExchange.Redis — Redis.io - https://redis.io/docs/latest/develop/clients/dotnet/

---

## 5. Cache Invalidation

### Definitions

**Core Definition**
Cache invalidation is the process of removing or marking stale entries in a cache so that subsequent requests fetch fresh data from the source.

**Technical Definition**
Cache invalidation strategies in .NET include **time-based expiration** (absolute and sliding TTLs), **explicit removal** (`Remove`/`RemoveAsync` on writes), **tag-based invalidation** (grouping related entries with tags and invalidating them together, supported by `HybridCache` and some output-cache implementations), and **event-driven invalidation** (publishing invalidation messages to a pub/sub channel so all instances drop their local copies). The two hard problems in caching are naming things, cache invalidation, and off-by-one errors — invalidation is hard because the cache and the source of truth are separate systems, and there is no distributed transaction that atomically updates both. The safest approach is to invalidate on every write (remove the entry after updating the source) and to accept a bounded staleness window through TTLs. **Stampede protection** is a related concern: when a popular entry expires, many concurrent requests may simultaneously attempt to regenerate it; `HybridCache` solves this with built-in coordination, while `IMemoryCache` and `IDistributedCache` require manual locking or a library.

**Beginner-Friendly Explanation**
Cache invalidation is "throwing away the old copy when you know it is wrong." If a user updates their profile, the cached profile is now stale. You remove it from the cache so the next read fetches the new data. The hard part is that the cache and the database are separate — you can remove the cache entry, but if the database update fails, you have thrown away a valid copy for nothing. And if you forget to remove it, users see stale data. This is why caching is often summarized as "there are only two hard things in computer science: cache invalidation and naming things."

---

### Purposes

- To prevent stale data from being served after a write.
- To bound the staleness window with time-based expiration.
- To group related entries and invalidate them together (tag-based).
- To coordinate invalidation across multiple application instances.
- To prevent cache stampedes when popular entries expire.
- To recover gracefully from failed writes or partial updates.

---

### Syntax Rules and Structure

#### Invalidation Strategies

| Strategy | Mechanism | Best For |
|---|---|---|
| **TTL expiration** | `AbsoluteExpirationRelativeToNow`, `SlidingExpiration` | Bounded staleness, infrequently changing data |
| **Explicit removal** | `RemoveAsync` after a write | Strong consistency requirements |
| **Tag-based** | `HybridCache` tags; output-cache tags | Related entries (e.g., all products in a category) |
| **Pub/sub invalidation** | Redis pub/sub to notify all instances | Multi-server coherence with local L1 caches |
| **Stampede protection** | `HybridCache.GetOrCreateAsync`; manual `SemaphoreSlim` | High-traffic entries that expire simultaneously |

#### General Syntax (Explicit Removal)

```csharp
public async Task UpdateProductAsync(Product product)
{
    await _repository.UpdateAsync(product);
    await _cache.RemoveAsync($"product:{product.Id}");
}
```

**Component breakdown**

- The source is updated first.
- The cache entry is removed after the update succeeds.
- The next read fetches the fresh data and repopulates the cache.

#### General Syntax (Tag-Based with HybridCache)

```csharp
await cache.GetOrCreateAsync("product:42", async ct => await FetchAsync(42),
    tags: new[] { "products", "category:electronics" });

// Invalidate all entries tagged "category:electronics"
await cache.RemoveByTagAsync("category:electronics");
```

**Component breakdown**

- `tags` — Groups related entries.
- `RemoveByTagAsync` — Invalidates all entries with the tag.

#### General Syntax (Stampede Protection)

```csharp
private static readonly SemaphoreSlim _lock = new(1, 1);

public async Task<Product?> GetWithLockAsync(int id)
{
    var key = $"product:{id}";
    if (_cache.TryGetValue(key, out Product? product))
        return product;

    await _lock.WaitAsync();
    try
    {
        if (_cache.TryGetValue(key, out product))  // double-check
            return product;

        product = await _repository.GetByIdAsync(id);
        _cache.Set(key, product, TimeSpan.FromMinutes(5));
        return product;
    }
    finally
    {
        _lock.Release();
    }
}
```

**Expected Output (conceptual)**

```
Without stampede protection: N concurrent misses trigger N database queries.
With stampede protection: one query executes; others wait and reuse the result.
```

**Why this output occurs**

- The semaphore serializes the regeneration, preventing duplicate work.

#### Syntax Rules

- Update the source first, then invalidate the cache.
- Use `RemoveAsync` for explicit invalidation on writes.
- Set an absolute expiration as a backstop even when using explicit invalidation; it bounds the damage of missed invalidations.
- Use tag-based invalidation for groups of related entries.
- Use stampede protection for entries that are expensive to regenerate and accessed concurrently.
- In multi-server deployments, use `HybridCache` or a pub/sub mechanism to propagate invalidation to L1 caches.

#### Constraints and Limitations

- There is no atomic transaction across the cache and the source; a crash between the write and the invalidation leaves a stale entry until TTL expiry.
- Tag-based invalidation is not supported by `IMemoryCache` or `IDistributedCache` directly; `HybridCache` provides it.
- Stampede protection with `SemaphoreSlim` is per-process; it does not coordinate across instances.
- Pub/sub invalidation requires reliable message delivery and handling of missed messages.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Write-Through Invalidation

```csharp
public async Task UpdateProductAsync(Product product)
{
    await _repository.UpdateAsync(product);
    await _cache.RemoveAsync($"product:{product.Id}");
    Console.WriteLine("Cache invalidated");
}
```

**Expected Output (conceptual)**

```
The product is updated in the database.
The cache entry is removed.
The next read fetches the updated product.
```

**Why this output occurs**

- Removing the entry forces the next read to fetch fresh data from the source.

---

#### Example 2: Tag-Based Invalidation with HybridCache

```csharp
// Store with tags
await cache.GetOrCreateAsync($"product:{id}", async ct => await FetchAsync(id),
    tags: new[] { "products", $"category:{categoryId}" });

// Invalidate a category
await cache.RemoveByTagAsync($"category:{categoryId}");
```

**Expected Output (conceptual)**

```
All product entries tagged with the category are invalidated together.
```

**Why this output occurs**

- `RemoveByTagAsync` removes every entry associated with the tag.

---

### Real-World Cases

- **E-commerce inventory** — Invalidate product entries when stock changes.
- **User profiles** — Remove cached profile after an update.
- **Content management** — Tag-based invalidation for articles in a category.
- **Configuration changes** — Pub/sub invalidation to refresh feature flags across instances.

---

### References

- Caching in .NET — Tag-based invalidation and stampede protection - https://learn.microsoft.com/en-us/dotnet/core/extensions/caching
- HybridCache in ASP.NET Core — Microsoft Learn - https://learn.microsoft.com/en-us/aspnet/core/performance/caching/hybrid
- Cache invalidation strategies — Azure Architecture Center - https://learn.microsoft.com/en-us/azure/architecture/best-practices/caching

---

## 6. Cache Best Practices

### Definitions

**Core Definition**
Caching best practices are empirically grounded guidelines for designing, implementing, and operating caches that improve performance without introducing correctness, memory, or operational hazards.

**Technical Definition**
Best practices span **key design** (deterministic, namespaced, free of unbounded user input), **expiration policy** (every entry has a TTL; sliding expiration is paired with an absolute cap), **size management** (`SizeLimit` and `Size` on `IMemoryCache`; compression for large distributed values), **invalidation discipline** (remove on write; accept bounded staleness; use tag-based invalidation for groups), **stampede protection** (coordinate regeneration of hot entries), **serialization efficiency** (JSON for `IDistributedCache`; consider protobuf or compression for large payloads), **failure handling** (the cache is not a source of truth; always have a fallback to the source), **observability** (hit ratio, miss rate, eviction count, latency), and **layer selection** (single-server → `IMemoryCache`; multi-server → `IDistributedCache`/Redis; both → `HybridCache`).

**Beginner-Friendly Explanation**
Cache best practices are "the rules that keep a cache helpful instead of harmful." They include: always set an expiration, do not use raw user input as a key, do not assume the cache is always populated, remove entries when data changes, and use `HybridCache` when you need both local speed and cross-server consistency. Following these rules prevents memory leaks, stale data, and mysterious production incidents.

---

### Purposes

- To prevent stale data from being served to users.
- To avoid unbounded memory growth and cache-induced outages.
- To ensure cache coherence across multiple instances.
- To protect against cache stampedes under high concurrency.
- To make cache behavior observable and debuggable.
- To choose the right cache layer for the deployment topology.
- To measure whether caching is actually improving performance.

---

### Guidelines and Structure

#### Choose the Right Layer

```
Single server, small hot data → IMemoryCache
Multi-server, shared data → IDistributedCache (Redis)
Multi-server + local speed + stampede protection + tags → HybridCache
Whole HTTP responses → Output caching
Client/CDN responses → Response caching (HTTP headers)
```

- `IMemoryCache` for per-node speed; `IDistributedCache` for coherence; `HybridCache` for both.
- For single-server applications with simple needs, `IMemoryCache` is sufficient.
- For multi-server applications without stampede/tag needs, consider distributed caching alone.

#### Design Keys Carefully

```csharp
// ✅ Structured, namespaced, bounded
var key = $"user:{userId}:profile";

// ❌ Unbounded external input
var key = userProvidedString;
```

- Use colons for hierarchy (`product:42:details`).
- Never use arbitrary user-provided input as a cache key.
- Include a version prefix when the serialized shape changes (`v2:product:42`).

#### Set Expiration on Every Entry

```csharp
new MemoryCacheEntryOptions
{
    AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(10),
    SlidingExpiration = TimeSpan.FromMinutes(2)
}
```

- Absolute expiration is a hard ceiling.
- Sliding expiration extends on access; pair it with an absolute cap.
- Do not create immortal cache entries.

#### Manage Size

```csharp
var cache = new MemoryCache(new MemoryCacheOptions { SizeLimit = 1024 });
cache.Set("key", value, new MemoryCacheEntryOptions { Size = 1 });
```

- The runtime does not trim `IMemoryCache` based on memory pressure; configure `SizeLimit`.
- Every entry must specify `Size` when `SizeLimit` is set.
- For distributed caches, compress values over ~100 KB.

#### Invalidate on Write

```csharp
await _repository.UpdateAsync(entity);
await _cache.RemoveAsync(key);
```

- Update the source first, then remove the cache entry.
- Use tag-based invalidation for groups of related entries.
- Accept bounded staleness through TTL as a backstop.

#### Protect Against Stampedes

- Use `HybridCache`'s built-in stampede protection.
- For `IMemoryCache`/`IDistributedCache`, use a `SemaphoreSlim` with double-checked locking.
- Consider pre-warming hot entries before traffic peaks.

#### Handle Failures Gracefully

- Always have a fallback to fetch from the source.
- If the cache is unavailable, log and continue with the source; do not fail the request.
- Treat the cache as an optimization, not a source of truth.

#### Measure and Monitor

- Track **hit ratio** (hits / (hits + misses)).
- Track **latency** for hits vs. misses.
- Track **eviction count** and **memory usage**.
- Alert on sudden drops in hit ratio or spikes in latency.
- Use `dotnet-counters` and distributed tracing to observe cache behavior.

#### Avoid Over-Caching

- Do not cache data that is rarely accessed or changes every request.
- Do not cache large objects without compression.
- Do not cache sensitive data without encryption and access controls.
- Do not assume caching fixes a slow query; fix the query first.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Production-Ready IMemoryCache Usage

```csharp
public class ProductService
{
    private readonly IMemoryCache _cache;
    private readonly IProductRepository _repository;

    public async Task<Product?> GetAsync(int id)
    {
        return await _cache.GetOrCreateAsync($"product:{id}", async entry =>
        {
            entry.AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(10);
            entry.SlidingExpiration = TimeSpan.FromMinutes(2);
            entry.Priority = CacheItemPriority.High;
            entry.Size = 1;
            return await _repository.GetByIdAsync(id);
        });
    }
}
```

**Expected Output (conceptual)**

```
Cache hit: returns the cached product.
Cache miss: fetches from the repository, stores with expiration and size, and returns.
```

**Why this output occurs**

- `GetOrCreateAsync` encapsulates the cache-aside pattern.
- Expiration and size are configured on every entry.

---

#### Example 2: HybridCache for Two-Level Caching

```csharp
public async Task<Product?> GetAsync(int id, HybridCache cache)
{
    return await cache.GetOrCreateAsync(
        $"product:{id}",
        async ct => await _repository.GetByIdAsync(id, ct),
        tags: new[] { "products", $"category:{categoryId}" });
}
```

**Expected Output (conceptual)**

```
L1 hit: returns from in-memory cache.
L1 miss, L2 hit: returns from distributed cache and populates L1.
Both miss: fetches from source, stores in L1 and L2, and returns.
Stampede protection: only one concurrent request executes the factory.
```

**Why this output occurs**

- `HybridCache` coordinates L1 and L2 and prevents duplicate concurrent factory executions.

---

### Real-World Cases

- **E-commerce** — Product catalog cached with `HybridCache`, invalidated by category tags on price updates.
- **SaaS platforms** — Per-tenant configuration cached in memory with short TTLs.
- **API gateways** — External API responses cached in Redis with stampede protection.
- **Content sites** — Whole-page output caching with tag-based invalidation on publish.

---

### References

- Cache in-memory in ASP.NET Core — Microsoft Learn - https://learn.microsoft.com/en-us/aspnet/core/performance/caching/memory
- Distributed caching in ASP.NET Core — Microsoft Learn - https://learn.microsoft.com/en-us/aspnet/core/performance/caching/distributed
- Caching in .NET — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/core/extensions/caching
- Caching guidance — Azure Architecture Center - https://learn.microsoft.com/en-us/azure/architecture/best-practices/caching
- HybridCache in ASP.NET Core — Microsoft Learn - https://learn.microsoft.com/en-us/aspnet/core/performance/caching/hybrid

---

## Summary Table: Choosing a Cache Layer

| Requirement | Recommended Layer | Key Consideration |
|---|---|---|
| **Fastest reads, single server** | `IMemoryCache` | Per-process; no cross-node coherence |
| **Shared cache across servers** | `IDistributedCache` + Redis | Serialization and network latency |
| **Both local speed and cross-server coherence** | `HybridCache` | Built-in stampede protection and tags |
| **Whole HTTP response caching** | ASP.NET Core output caching | Server-side; Redis store for scale-out |
| **CDN / browser caching** | `[ResponseCache]` / `Cache-Control` | Client-side; no server involvement |
| **Tag-based group invalidation** | `HybridCache` | Not available in `IMemoryCache`/`IDistributedCache` |
| **Stampede protection** | `HybridCache` or manual `SemaphoreSlim` | Per-instance only with manual locking |

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| **`System.Runtime.Caching.MemoryCache`** | Legacy; `Microsoft.Extensions.Caching.Memory/IMemoryCache` is recommended for ASP.NET Core |
| **`IMemoryCache` automatic memory-pressure trimming** | The ASP.NET Core runtime does not limit cache size based on memory pressure; size limits are the developer's responsibility |
| **`SetSize` on a shared cache** | Can cause failures if other consumers do not specify sizes; create a dedicated cache singleton |
| **Sliding expiration without absolute cap** | Can keep hot entries alive indefinitely; pair with an absolute expiration |
| **Unbounded external input as cache key** | Security and memory risk; use structured, namespaced keys |
| **`HybridCache`** | Introduced in .NET 9; requires the `Microsoft.Extensions.Caching.Hybrid` package |
| **`IDistributedCache` advanced Redis structures** | Not exposed by the abstraction; use `StackExchange.Redis` directly |
| **`ConnectionMultiplexer` per-operation creation** | Performance anti-pattern; create once and reuse |
| **Large distributed cache values** | Serialization and network overhead; consider compression and streaming for values over ~100 KB |

---

## References (Consolidated)

- Cache in-memory in ASP.NET Core — Microsoft Learn - https://learn.microsoft.com/en-us/aspnet/core/performance/caching/memory
- Distributed caching in ASP.NET Core — Microsoft Learn - https://learn.microsoft.com/en-us/aspnet/core/performance/caching/distributed
- Caching in .NET — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/core/extensions/caching
- Caching guidance — Azure Architecture Center - https://learn.microsoft.com/en-us/azure/architecture/best-practices/caching
- HybridCache in ASP.NET Core — Microsoft Learn - https://learn.microsoft.com/en-us/aspnet/core/performance/caching/hybrid
- IMemoryCache interface — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.caching.memory.imemorycache
- IDistributedCache interface — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.caching.distributed.idistributedcache
- MemoryCacheEntryOptions — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.caching.memory.memorycacheentryoptions
- DistributedCacheEntryOptions — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.caching.distributed.distributedcacheentryoptions
- StackExchange.Redis documentation - https://stackexchange.github.io/StackExchange.Redis/
- Redis cache-aside with StackExchange.Redis — Redis.io - https://redis.io/docs/latest/develop/clients/dotnet/
- Caching in C#: A Complete Guide to .NET Performance and Scalability — Devart - https://www.devart.com/blog/caching-in-csharp-dotnet-guide.html