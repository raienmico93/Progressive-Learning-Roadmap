# ASP.NET Core Caching: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core** (.NET 6–10) and the full caching stack: **in-memory caching** (`IMemoryCache`), **distributed caching** (`IDistributedCache`, Redis), **response caching** (HTTP-header-driven client/proxy caching), and **output caching** (server-side, configuration-driven response caching, introduced in .NET 7). A critical distinction: **`IMemoryCache` lives inside a single process** — it does not share data across server instances. **`IDistributedCache`** is the abstraction for a shared external store (Redis, SQL Server, etc.). **Response caching** is driven by HTTP `Cache-Control` headers and is limited to memory. **Output caching** is server-controlled, extensible, and supports cache invalidation and resource locking. Each requested topic is treated independently — no sections are merged.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Caching is the technique of storing frequently accessed data in a faster, temporary storage layer so that future requests for that data can be served more quickly, reducing latency, database load, and infrastructure cost. |
| **Technical Definition** | ASP.NET Core supports multiple caching layers: **in-memory caching** (`IMemoryCache`) stores data in the web server's process memory; **distributed caching** (`IDistributedCache`) stores data in a shared external store (Redis, SQL Server, NCache) accessible across multiple app instances; **response caching** uses HTTP `Cache-Control` headers to instruct clients and proxies to cache responses; and **output caching** (ASP.NET Core 7+) stores complete HTTP responses server-side with configurable policies, invalidation, and resource locking. **HybridCache** (.NET 9+) combines in-memory and distributed caching with stampede protection. |
| **Beginner-Friendly Explanation** | Caching is the **"keep a copy nearby"** technique. Instead of fetching the same data from the database or recomputing it every time, you store a copy in fast memory. The next request finds the copy instantly. Different caches serve different purposes: **in-memory** is fastest but lives inside one server; **distributed** is shared across servers; **response caching** tells browsers and proxies to keep a copy; **output caching** lets the server keep the whole response. The trick is knowing **what** to cache, **for how long**, and **when to evict** it. |

### Key Characteristics

- **Layered** – Different caching mechanisms solve different problems; they are complementary, not mutually exclusive.
- **Process-bound vs. shared** – `IMemoryCache` is per-process; `IDistributedCache` is shared across instances.
- **TTL-driven** – Cached entries expire (absolute or sliding); expiry is the primary eviction mechanism.
- **Fallback-required** – Code must always be able to fetch data if the cache is empty or unavailable.
- **Stampede-prone** – Without protection, many concurrent requests can repopulate the same expired entry simultaneously; output caching and HybridCache provide resource locking.

### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core project.
- Working knowledge of **dependency injection**, **middleware**, and **async/await**.
- For distributed caching: **Redis** (StackExchange.Redis) or another distributed cache provider.
- Familiarity with **HTTP caching semantics** (`Cache-Control`, `Vary`, `ETag`).
- `Microsoft.Extensions.Caching.Memory` (in-memory), `Microsoft.Extensions.Caching.StackExchangeRedis` (Redis), `Microsoft.AspNetCore.OutputCaching` (output caching, .NET 7+).

### Related Programming Areas

- **Performance** – Caching is the highest-impact latency and throughput lever.
- **Database access** – Caching reduces query load and round trips.
- **Middleware pipeline** – Response and output caching are pipeline-level features.
- **Distributed systems** – Shared caches require consistency and invalidation strategies.
- **Observability** – Cache hit rates, eviction counts, and latency improvements must be measured.

### Core Concepts / Features

1. **Caching** – The overarching concept, layers, and trade-offs.
2. **In-memory caching** – Process-local caching with `IMemoryCache`.
3. **`IMemoryCache`** – The API, expiration policies, and cache-aside pattern.
4. **Distributed caching** – Shared caching with `IDistributedCache`.
5. **`IDistributedCache`** – The API, serialization, and Redis integration.
6. **Redis** – The recommended distributed cache backend.
7. **Response caching** – HTTP-header-driven client/proxy caching.
8. **Output caching** – Server-side response caching with policies and invalidation.
9. **Caching best practices** – What to cache, eviction, stampede protection, and security.

---

## 1. Caching (The Overarching Concept)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Caching is the practice of storing copies of data in a faster, temporary storage layer to reduce the time and resources required to serve future requests for that data. |
| **Technical Definition** | Caching in ASP.NET Core is implemented at multiple layers: **application-level** (`IMemoryCache`, `IDistributedCache`, `HybridCache`), **HTTP-level** (response caching via `Cache-Control`), and **middleware-level** (output caching). Each layer has its own storage medium, scope, and invalidation mechanism. The **cache-aside pattern** (check cache → on miss, fetch data → populate cache → return) is the most common application-level strategy. |
| **Beginner-Friendly Explanation** | Caching is the **"don't do the same work twice"** principle. If fetching a product from the database takes 50 ms and 1,000 users request it, that is 50 seconds of database work. If you cache it, the first request takes 50 ms and the next 999 take 1 ms. Caching trades memory (and some freshness) for speed. The layers let you choose the right trade-off: fastest (in-memory) but not shared, shared (distributed) but slower, or HTTP-level (browser/proxy) that offloads the server entirely. |

### Purposes

- **To** **reduce latency** — serve cached data in microseconds instead of milliseconds (or seconds).
- **To** **reduce database and external-service load** — fewer round trips means lower cost and higher capacity.
- **To** **improve throughput** — cached responses free server resources for other work.
- **To** **protect downstream dependencies** — cache results from fragile or rate-limited external APIs.
- **To** **provide graceful degradation** — a cache can serve stale data when the origin is unavailable.

### Syntax Rules and Structure

#### The Caching Layers

| Layer | Storage | Scope | Invalidation | When to Use |
|-------|---------|-------|--------------|-------------|
| **`IMemoryCache`** | Server memory | Single process | TTL, manual | Single server; fastest access |
| **`IDistributedCache`** | Redis / SQL Server | All instances | TTL, manual | Multi-instance, shared data |
| **Response caching** | Client / proxy | Per-client | HTTP headers | Public, cacheable GET responses |
| **Output caching** | Server memory (extensible) | Per-server (or distributed) | Policies, tags | Any cacheable HTTP response, with server control |

**Syntax rules:**

- **Use `IMemoryCache` for single-server scenarios** — it is the simplest and fastest option.
- **Use `IDistributedCache` when multiple instances must see the same data** — `IMemoryCache` gives each server its own copy.
- **Use response caching for browser/proxy caching** — it reduces requests but depends on client headers.
- **Use output caching for server-side control** — it works regardless of client headers and supports invalidation.

**Constraints and limitations:**

- **`IMemoryCache` is not shared** — each server has its own cache; in a load-balanced environment, data can be inconsistent.
- **Caching is not free** — it consumes memory; unbounded caches can cause memory pressure or OOM.
- **Stale data** — cached data can become outdated; TTL and invalidation strategies are essential.

### Annotated Complete Step‑by‑Step Code Example (Choosing a Layer)

```csharp
// Single-server, fastest: IMemoryCache
builder.Services.AddMemoryCache();

// Multi-server, shared: IDistributedCache + Redis
builder.Services.AddStackExchangeRedisCache(options =>
{
    options.Configuration = builder.Configuration.GetConnectionString("Redis");
});

// HTTP-level: response caching (client/proxy)
builder.Services.AddResponseCaching();

// Server-side control: output caching (.NET 7+)
builder.Services.AddOutputCache();
```

**Step-by-step explanation:**

1. `AddMemoryCache()` registers `IMemoryCache` for process-local caching.
2. `AddStackExchangeRedisCache()` registers `IDistributedCache` backed by Redis for cross-instance sharing.
3. `AddResponseCaching()` registers the middleware that respects `Cache-Control` headers.
4. `AddOutputCache()` registers the server-side output caching middleware.

**Expected Output:** The application has access to all four caching layers; the developer chooses the appropriate layer per scenario.

**Why this output?** Each registration adds a different caching capability; they coexist and serve different purposes.

### Real-World Cases

- **Product catalogue:** `IMemoryCache` for single-server; Redis for multi-instance.
- **Public API responses:** Output caching for `/api/products` with a 60-second policy.
- **Third-party API results:** Distributed cache with a TTL to avoid rate limits.

### References

- Overview of caching in ASP.NET Core – https://github.com/dotnet/aspnetcore.docs/blob/0885e7518455420953be73e5e0d36c490b314dde/aspnetcore/performance/caching/overview.md
- Cache in-memory in ASP.NET Core – https://learn.microsoft.com/el-gr/aspnet/core/performance/caching/memory?view=aspnetcore-3.1#2
- Distributed caching in ASP.NET Core – https://learn.microsoft.com/hi-in/aspnet/core/performance/caching/distributed?view=aspnetcore-3.1#2

---

## 2. In‑Memory Caching

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | In-memory caching stores cached data in the memory of the running application process, providing the fastest possible access but limited to a single server instance. |
| **Technical Definition** | `IMemoryCache` (in `Microsoft.Extensions.Caching.Memory`) is the in-memory cache implementation for ASP.NET Core. It stores entries in the server's process memory, supports **absolute expiration**, **sliding expiration**, **size limits**, and **eviction callbacks**. It is registered via `AddMemoryCache()` and injected via constructor. It is **not** shared across processes or servers. |
| **Beginner-Friendly Explanation** | In-memory caching is the **“sticky note on your desk”** cache. It is the fastest way to store data because it lives right inside the app's memory. But if you have multiple servers, each server has its own sticky note — they do not share. It is perfect for single-server apps or data that does not need to be perfectly consistent across instances. |

### Purposes

- **To** provide the **fastest possible cache access** — no network round-trip, no serialization overhead.
- **To** cache **frequently accessed, relatively static data** — configuration, reference data, computed values.
- **To** serve **single-server deployments** where cross-instance consistency is not required.
- **To** reduce **database load** for repeated queries within one process.

### Syntax Rules and Structure

```csharp
builder.Services.AddMemoryCache();
```

```csharp
public class ProductService
{
    private readonly IMemoryCache _cache;
    public ProductService(IMemoryCache cache) => _cache = cache;
}
```

**Syntax rules:**

- **Register with `AddMemoryCache()`** — this is required before injecting `IMemoryCache`.
- **Inject `IMemoryCache` via constructor** — it is a singleton service.
- **Use `TryGetValue`/`Set` or `GetOrCreate`** — `GetOrCreate` is the most common pattern for cache-aside.
- **Always provide a fallback** — code must fetch data if the cache is empty.

**Constraints and limitations:**

- **Not shared across processes** — each server instance has its own cache.
- **Memory is a scarce resource** — the runtime does not limit cache size by default; developers must use `SetSize`/`SizeLimit` or expirations.
- **Entries are lost on app restart** — in-memory cache is volatile.

### Annotated Complete Step‑by‑Step Code Example (Basic Setup)

```csharp
builder.Services.AddMemoryCache();

public class ReferenceDataService
{
    private readonly IMemoryCache _cache;
    private readonly AppDbContext _db;

    public ReferenceDataService(IMemoryCache cache, AppDbContext db)
    {
        _cache = cache;
        _db = db;
    }

    public async Task<List<Country>> GetCountriesAsync()
    {
        if (_cache.TryGetValue("countries", out List<Country>? cached))
            return cached!;

        var countries = await _db.Countries.AsNoTracking().ToListAsync();
        _cache.Set("countries", countries, TimeSpan.FromHours(1));
        return countries;
    }
}
```

**Step-by-step explanation:**

1. `AddMemoryCache()` registers the cache service.
2. `ReferenceDataService` injects `IMemoryCache` and the database context.
3. `GetCountriesAsync` checks the cache first; on a miss, it queries the database and caches the result for one hour.

**Expected Output:** The first call queries the database; subsequent calls within one hour return the cached list.

**Why this output?** The cache-aside pattern avoids repeated database queries for static reference data.

### Real-World Cases

- **Country/currency lists:** Rarely change; cache for hours.
- **Feature flags:** Cache configuration values for short periods.
- **Computed aggregates:** Cache expensive calculations with a sliding expiration.

### References

- Cache in-memory in ASP.NET Core – https://learn.microsoft.com/el-gr/aspnet/core/performance/caching/memory?view=aspnetcore-3.1#2
- `IMemoryCache` interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.caching.memory.imemorycache

---

## 3. `IMemoryCache` (The API)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `IMemoryCache` is the interface representing a cache stored in the memory of the web server, providing methods for storing and retrieving cached entries. |
| **Technical Definition** | `IMemoryCache` exposes `TryGetValue`, `Set`, `Remove`, and the `GetOrCreate`/`GetOrCreateAsync` extension methods. Entries are configured with `MemoryCacheEntryOptions`, which control **absolute expiration** (`AbsoluteExpirationRelativeToNow`), **sliding expiration** (`SlidingExpiration`), **size**, and **eviction callbacks**. The `GetOrCreate` pattern is atomic and thread-safe, avoiding the race condition of separate `TryGetValue` + `Set` calls. |
| **Beginner-Friendly Explanation** | `IMemoryCache` is the **“tool”** for in-memory caching. You give it a key and a value, and it holds onto it. You can say “expire this after 5 minutes” or “expire this if nobody has accessed it for 10 minutes.” The `GetOrCreate` method is the easiest pattern: “give me the value for this key, or if it's not there, run this function to create it and store it.” |

### Purposes

- **To** provide a **simple, thread-safe API** for caching data in process memory.
- **To** support **flexible expiration policies** —absolute, sliding, or both.
- **To** enable **size-limited caches** to prevent unbounded memory growth.
- **To** offer the **`GetOrCreate` pattern** for clean cache-aside code.
- **To** support **eviction callbacks** for cleanup when entries are removed.

### Syntax Rules and Structure

#### Basic TryGetValue + Set

```csharp
if (!_cache.TryGetValue(key, out MyData? data))
{
    data = await FetchFromSourceAsync();
    _cache.Set(key, data, TimeSpan.FromMinutes(5));
}
return data;
```

#### GetOrCreateAsync (Recommended)

```csharp
var data = await _cache.GetOrCreateAsync(key, async entry =>
{
    entry.AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(5);
    return await FetchFromSourceAsync();
});
```

**Syntax rules:**

- **`GetOrCreateAsync` is atomic** — it prevents the race condition where two requests both miss and both fetch.
- **Use `AbsoluteExpirationRelativeToNow`** for a fixed TTL; **`SlidingExpiration`** resets the TTL on access.
- **Avoid using external input as cache keys** — validate or hash keys to prevent cache poisoning.

**Constraints and limitations:**

- **`GetOrCreate` can still have a stampede** — if many requests miss simultaneously, multiple factory calls may run; use locking (e.g., `SemaphoreSlim`) or output caching for stampede protection.
- **`IMemoryCache` does not enforce a size limit by default** — use `SetSize`/`SizeLimit` on a dedicated cache instance.

### Annotated Complete Step‑by‑Step Code Example (GetOrCreateAsync)

```csharp
public class ProductService
{
    private readonly IMemoryCache _cache;
    private readonly AppDbContext _db;

    public ProductService(IMemoryCache cache, AppDbContext db)
    {
        _cache = cache;
        _db = db;
    }

    public async Task<Product?> GetProductAsync(int id)
    {
        return await _cache.GetOrCreateAsync($"product:{id}", async entry =>
        {
            entry.SlidingExpiration = TimeSpan.FromMinutes(10);
            entry.AbsoluteExpirationRelativeToNow = TimeSpan.FromHours(1);
            return await _db.Products.AsNoTracking()
                .FirstOrDefaultAsync(p => p.Id == id);
        });
    }
}
```

**Step-by-step explanation:**

1. `GetOrCreateAsync` checks the cache for `product:{id}`.
2. On a miss, the factory runs, queries the database, and returns the product.
3. The entry has a sliding expiration of 10 minutes (reset on access) and an absolute limit of 1 hour.

**Expected Output:** Repeated requests for the same product within 10 minutes return the cached instance without hitting the database; after 1 hour, the entry expires regardless of access.

**Why this output?** The sliding + absolute combination bounds both idle lifetime and total lifetime.

### Real-World Cases

- **Product details:** Cache with sliding expiration to keep hot products cached.
- **Configuration values:** Absolute expiration to ensure periodic refresh.
- **User profile data:** Short sliding expiration to reflect changes.

### References

- `IMemoryCache` interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.caching.memory.imemorycache
- `GetOrCreate` pattern – https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-01-27-dotnet-imemorycache-redis-caching/README.md

---

## 4. Distributed Caching

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Distributed caching stores cached data in a shared external store accessible by multiple application instances, ensuring consistency across a server farm or cloud deployment. |
| **Technical Definition** | `IDistributedCache` is the ASP.NET Core abstraction for distributed caches. Implementations include **Redis** (`AddStackExchangeRedisCache`), **SQL Server** (`AddDistributedSqlServerCache`), **NCache**, and **Distributed Memory Cache** (`AddDistributedMemoryCache`, for development only). Data is serialized (typically JSON or byte arrays) and stored externally; every instance reads and writes the same store. |
| **Beginner-Friendly Explanation** | Distributed caching is the **“shared whiteboard”** cache. Instead of each server having its own sticky note, all servers read and write to the same whiteboard (Redis, SQL Server, etc.). If one server caches a product, every server sees it. This is essential when you have multiple servers behind a load balancer — otherwise, users see inconsistent data depending on which server handles their request. |

### Purposes

- **To** **share cached data across multiple instances** — consistent data regardless of which server handles the request.
- **To** **survive app restarts** — Redis and SQL Server persist data independently of the app process.
- **To** **centralise cache invalidation** — one write invalidates the cache for all instances.
- **To** **scale out** — add more app instances without fragmenting the cache.

### Syntax Rules and Structure

```csharp
// Redis (recommended for production)
builder.Services.AddStackExchangeRedisCache(options =>
{
    options.Configuration = builder.Configuration.GetConnectionString("Redis");
    options.InstanceName = "MyApp:";
});
```

**Syntax rules:**

- **Use `AddStackExchangeRedisCache` for Redis** — the recommended production implementation.
- **Use `AddDistributedMemoryCache` only in development** — it is not a real distributed cache.
- **Configure `InstanceName`** to namespace keys when sharing a Redis instance.
- **Store connection strings in secure configuration** — not in source code.

**Constraints and limitations:**

- **Serialization overhead** — distributed cache entries must be serialized/deserialized, adding latency compared to `IMemoryCache`.
- **Network dependency** — the cache store becomes a dependency; plan for outages.
- **`AddDistributedMemoryCache` is not distributed** — it stores data in the local process, defeating the purpose in multi-instance deployments.

### Annotated Complete Step‑by‑Step Code Example

```csharp
builder.Services.AddStackExchangeRedisCache(options =>
{
    options.Configuration = builder.Configuration.GetConnectionString("Redis");
    options.InstanceName = "Catalog:";
});
```

**Expected Output:** `IDistributedCache` resolves to a Redis-backed cache shared across all app instances.

**Why this output?** The registration wires the abstraction to the Redis implementation; every instance connects to the same Redis server.

### Real-World Cases

- **Multi-instance APIs:** Session state, user profiles, and reference data shared across instances.
- **Cloud deployments:** Azure Cache for Redis as a managed distributed cache.
- **Microservices:** Shared cache for cross-service data.

### References

- Distributed caching in ASP.NET Core – https://learn.microsoft.com/hi-in/aspnet/core/performance/caching/distributed?view=aspnetcore-3.1#2
- `IDistributedCache` interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.caching.distributed.idistributedcache

---

## 5. `IDistributedCache` (The API)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `IDistributedCache` is the interface representing a distributed cache, providing methods for storing and retrieving serialized data from a shared external store. |
| **Technical Definition** | `IDistributedCache` exposes string-based methods (`GetString`, `SetString`, `GetStringAsync`, `SetStringAsync`), byte-array methods (`Get`, `Set`, `GetAsync`, `SetAsync`), and `Refresh`, `Remove`, `RemoveAsync`. `DistributedCacheEntryOptions` controls absolute and sliding expiration. Complex objects must be serialized (e.g., JSON) before storage. |
| **Beginner-Friendly Explanation** | `IDistributedCache` is the **“API for the shared whiteboard”** . You give it a key and a value (usually a JSON string), and it stores it in Redis or SQL Server. You can set an expiration so the data does not live forever. When you need the data, you ask for it by key — if it's there, you get it back; if not, you fetch it from the source and store it. |

### Purposes

- **To** provide a **unified API** for any distributed cache implementation (Redis, SQL Server, NCache).
- **To** support **string and byte-array storage** for flexible serialization.
- **To** enable **expiration policies** (`DistributedCacheEntryOptions`).
- **To** allow **swapping the backing store** without changing application code.

### Syntax Rules and Structure

#### GetStringAsync / SetStringAsync (Recommended)

```csharp
var cached = await _cache.GetStringAsync(cacheKey);
if (cached is not null)
    return JsonSerializer.Deserialize<Product>(cached);

var product = await _db.Products.FindAsync(id);
await _cache.SetStringAsync(cacheKey, JsonSerializer.Serialize(product),
    new DistributedCacheEntryOptions
    {
        AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(10)
    });
return product;
```

**Syntax rules:**

- **Use `GetStringAsync`/`SetStringAsync` for JSON-serialized objects** — the most common pattern.
- **Set an expiration** via `DistributedCacheEntryOptions` — otherwise, the entry may never expire.
- **Handle null** — a cache miss returns `null`; always fall back to the source.

**Constraints and limitations:**

- **Serialization is the developer's responsibility** — `IDistributedCache` stores strings or byte arrays, not objects.
- **Larger payloads increase latency** — keep cached values small.

### Annotated Complete Step‑by‑Step Code Example

```csharp
public class ProductService
{
    private readonly IDistributedCache _cache;
    private readonly AppDbContext _db;

    public ProductService(IDistributedCache cache, AppDbContext db)
    {
        _cache = cache;
        _db = db;
    }

    public async Task<Product?> GetProductAsync(int id)
    {
        var key = $"product:{id}";
        var cached = await _cache.GetStringAsync(key);
        if (cached is not null)
            return JsonSerializer.Deserialize<Product>(cached);

        var product = await _db.Products.AsNoTracking()
            .FirstOrDefaultAsync(p => p.Id == id);
        if (product is not null)
        {
            await _cache.SetStringAsync(key, JsonSerializer.Serialize(product),
                new DistributedCacheEntryOptions
                {
                    AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(10)
                });
        }
        return product;
    }
}
```

**Expected Output:** The first call queries the database and stores the JSON-serialized product in Redis for 10 minutes; subsequent calls deserialize the cached JSON.

**Why this output?** The cache-aside pattern with `IDistributedCache` works across all instances sharing the Redis store.

### Real-World Cases

- **User sessions:** Store serialized session data in Redis for multi-instance apps.
- **Reference data:** Cache country lists, categories, and configuration across instances.
- **API responses:** Cache expensive query results for a short TTL.

### References

- `IDistributedCache` interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.caching.distributed.idistributedcache
- Redis distributed cache example – https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-03-31-redis-aspnet-core-distributed-cache/README.md

---

## 6. Redis (The Recommended Distributed Cache Backend)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Redis is an open-source, in-memory data structure store commonly used as a distributed cache, providing sub-millisecond latency and support for strings, hashes, lists, sets, and more. |
| **Technical Definition** | In ASP.NET Core, Redis is integrated via the `Microsoft.Extensions.Caching.StackExchangeRedis` package, which registers `RedisCache` as the `IDistributedCache` implementation. Configuration uses `AddStackExchangeRedisCache` with a connection string and optional `InstanceName` for key prefixing. Azure Cache for Redis is the managed Azure offering. |
| **Beginner-Friendly Explanation** | Redis is the **“fast shared memory”** for distributed caching. It is a separate server (or managed service) that holds data in memory, so all your app servers can read and write to it. It is much faster than a database and is the recommended production choice for distributed caching in ASP.NET Core. |

### Purposes

- **To** provide **high-performance, shared caching** for multi-instance applications.
- **To** serve as the **recommended production `IDistributedCache` implementation**.
- **To** support **key expiration, eviction policies, and persistence** (if configured).
- **To** integrate with **Azure Cache for Redis** for managed deployments.

### Syntax Rules and Structure

```csharp
dotnet add package Microsoft.Extensions.Caching.StackExchangeRedis
```

```csharp
builder.Services.AddStackExchangeRedisCache(options =>
{
    options.Configuration = builder.Configuration.GetConnectionString("Redis");
    options.InstanceName = "MyApp:";
});
```

**Syntax rules:**

- **Install the Redis cache package** — `Microsoft.Extensions.Caching.StackExchangeRedis`.
- **Configure the connection string** from secure configuration.
- **Set `InstanceName`** to namespace keys when sharing Redis with other applications.

**Constraints and limitations:**

- **Redis is an external dependency** — plan for network latency and outages.
- **Connection string management** — use Key Vault or secrets, not source code.

### Annotated Complete Step‑by‑Step Code Example

```csharp
builder.Services.AddStackExchangeRedisCache(options =>
{
    options.Configuration = builder.Configuration.GetConnectionString("MyRedisConStr");
    options.InstanceName = "SampleInstance";
});
```

**Expected Output:** `IDistributedCache` is backed by Redis; all app instances connect to the same Redis server and share cached entries.

**Why this output?** The `AddStackExchangeRedisCache` registration wires the distributed cache abstraction to the Redis implementation.

### Real-World Cases

- **Azure deployments:** Azure Cache for Redis as a managed, highly available cache.
- **Session state:** Redis as the backing store for distributed sessions.
- **Rate limiting and counters:** Redis atomic operations for shared counters.

### References

- Distributed Redis Cache – https://learn.microsoft.com/hi-in/aspnet/core/performance/caching/distributed?view=aspnetcore-3.1#2
- `AddStackExchangeRedisCache` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.stackexchangerediscacheextensions.addstackexchangerediscache

---

## 7. Response Caching (HTTP-Header-Driven)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Response caching is an HTTP-level caching mechanism that uses `Cache-Control` headers to instruct clients and intermediate proxies to cache responses, reducing the number of requests that reach the server. |
| **Technical Definition** | ASP.NET Core’s **Response Caching Middleware** respects `Cache-Control` headers set by the application (via `[ResponseCache]` or manual headers) and caches server responses that result in **200 OK** for the duration specified. It does not cache responses with `Set-Cookie`, `Authorization`, or other non-cacheable headers by default. The middleware is limited to **memory** and depends on the client honoring the headers. |
| **Beginner-Friendly Explanation** | Response caching is the **“tell the browser to remember this”** cache. The server says “this response is good for 60 seconds” via the `Cache-Control` header. The browser (or a proxy) stores it and serves it without asking the server again. It is great for public, cacheable GET responses, but you have little control over client behaviour — some clients send `Cache-Control: max-age=0` and bypass the cache entirely. |

### Purposes

- **To** **reduce requests to the server** — clients and proxies serve cached responses directly.
- **To** **leverage HTTP caching semantics** — standard, interoperable caching across browsers and CDNs.
- **To** **cache public GET responses** — product lists, static content, public API data.
- **To** **reduce bandwidth and server load** — especially for repetitive requests.

### Syntax Rules and Structure

```csharp
// Program.cs
builder.Services.AddResponseCaching();
app.UseResponseCaching();
```

```csharp
// Controller action
[ResponseCache(Duration = 60, Location = ResponseCacheLocation.Any)]
public IActionResult GetProducts() => Ok(products);
```

**Syntax rules:**

- **`AddResponseCaching()` + `UseResponseCaching()`** — required for the middleware.
- **`[ResponseCache]`** sets the `Cache-Control` header; `Duration` is in seconds.
- **`VaryByQueryKeys`** requires the middleware to vary the cache by query string.
- **Only 200 OK responses are cached**.

**Constraints and limitations:**

- **Clients can override** — a browser sending `Cache-Control: max-age=0` bypasses the cache.
- **Limited invalidation** — no programmatic invalidation; expiry is header-driven.
- **Memory-only** — the middleware does not use distributed storage.

### Annotated Complete Step‑by‑Step Code Example

```csharp
builder.Services.AddResponseCaching();
app.UseResponseCaching();

[HttpGet]
[ResponseCache(Duration = 60, Location = ResponseCacheLocation.Any, VaryByQueryKeys = new[] { "category" })]
public IActionResult GetProducts(string? category)
{
    return Ok(_service.GetProducts(category));
}
```

**Expected Output:** The first request returns `200 OK` with `Cache-Control: public,max-age=60`; subsequent requests within 60 seconds may be served from the client/proxy cache if the client honors the header.

**Why this output?** The `[ResponseCache]` attribute sets the HTTP caching headers; the middleware caches the server-side response.

### Real-World Cases

- **Public product listings:** Cache for 60 seconds to reduce database load.
- **Static API data:** Country lists, categories — cache with `Location = Any`.
- **CDN integration:** `Cache-Control` headers control CDN edge caching.

### References

- Response caching in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/performance/caching/response
- Response caching middleware – https://github.com/dotnet/AspNetCore.Docs/blob/634aa0e227eafee34f6f86ca032a1f087b5b0b15/aspnetcore/performance/caching/response.md

---

## 8. Output Caching (Server-Side Response Caching, .NET 7+)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Output caching is a server-side caching mechanism (introduced in ASP.NET Core 7) that stores complete HTTP responses and serves them directly for subsequent requests, with configuration controlled entirely by the server. |
| **Technical Definition** | The **Output Caching Middleware** (`AddOutputCache()`, `UseOutputCache()`) caches HTTP responses based on **policies** — not HTTP headers. It supports **named policies**, **cache tagging**, **programmatic invalidation** (`IOutputCacheStore.EvictByTagAsync`), **resource locking** (to prevent cache stampede), and **cache revalidation** (304 Not Modified). The storage medium is extensible (memory by default; Redis possible). |
| **Beginner-Friendly Explanation** | Output caching is the **“server remembers the whole response”** cache. Unlike response caching, which depends on the client honoring headers, output caching is controlled by the server. You say “cache this endpoint for 60 seconds, and tag it as ‘products’.” The server keeps the response and serves it without re-running the action. You can programmatically invalidate it by tag — for example, when a product is updated, evict all responses tagged “products”. It also includes **resource locking** to prevent many requests from hammering the database when a cache entry expires. |

### Purposes

- **To** cache HTTP responses **server-side**, independent of client headers.
- **To** support **programmatic invalidation** via tags — evict related entries when data changes.
- **To** prevent **cache stampede** through resource locking.
- **To** reduce **bandwidth** through cache revalidation (304 Not Modified).
- **To** provide a **more powerful alternative to response caching** for API endpoints.

### Syntax Rules and Structure

```csharp
builder.Services.AddOutputCache(options =>
{
    options.AddBasePolicy(builder => builder.Expire(TimeSpan.FromSeconds(60)));
    options.AddPolicy("Products", builder => builder
        .Expire(TimeSpan.FromMinutes(5))
        .SetVaryByQuery("category")
        .Tag("products"));
});

app.UseOutputCache();

[HttpGet]
[OutputCache(PolicyName = "Products")]
public IActionResult GetProducts(string? category) => Ok(_service.GetProducts(category));
```

**Syntax rules:**

- **`AddOutputCache()` + `UseOutputCache()`** — required for the middleware.
- **`[OutputCache]`** on controllers/actions (or `.CacheOutput()` on Minimal APIs) applies a policy.
- **Tags** enable `EvictByTagAsync` for invalidation.
- **`SetVaryByQuery`** varies the cache by query string parameters.
- **Resource locking is built-in** — concurrent requests for the same cache entry wait for the first to populate it.

**Constraints and limitations:**

- **Requires .NET 7+**.
- **Memory storage by default** — Redis-backed output caching requires additional configuration.
- **Not a replacement for `IMemoryCache` for non-HTTP data** — output caching is specifically for HTTP responses.

### Annotated Complete Step‑by‑Step Code Example

```csharp
builder.Services.AddOutputCache(options =>
{
    options.AddPolicy("Products", builder => builder
        .Expire(TimeSpan.FromMinutes(5))
        .SetVaryByQuery("category")
        .Tag("products"));
});

app.UseOutputCache();

[HttpGet]
[OutputCache(PolicyName = "Products")]
public async Task<IActionResult> GetProducts(string? category)
{
    var products = await _service.GetProductsAsync(category);
    return Ok(products);
}

// Invalidation on update
[HttpPost("products/{id}")]
public async Task<IActionResult> UpdateProduct(int id, ProductInput input)
{
    await _service.UpdateAsync(id, input);
    await _outputCacheStore.EvictByTagAsync("products", HttpContext.RequestAborted);
    return NoContent();
}
```

**Expected Output:** `GET /api/products?category=electronics` is cached for 5 minutes; when a product is updated, all responses tagged “products” are evicted, and the next request regenerates the cache.

**Why this output?** Output caching stores the complete response server-side; tag-based invalidation ensures stale responses are removed when data changes.

### Real-World Cases

- **Public API endpoints:** Cache `/api/products` with query-string variation and tag-based invalidation.
- **High-traffic endpoints:** Use resource locking to prevent database stampede on cache expiry.
- **Content pages:** Cache rendered HTML for anonymous users.

### References

- Output caching middleware in ASP.NET Core – https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/performance/caching/output.md
- Output caching vs response caching – https://www.ssw.com.au/rules/do-you-use-output-caching-for-better-api-performance

---

## 9. Caching Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Caching best practices are guidelines for choosing what to cache, how long to cache it, how to invalidate it safely, and how to prevent common pitfalls like cache stampede, memory exhaustion, and stale data. |
| **Technical Definition** | Best practices encompass **cache-aside with fallback** (always able to fetch from source), **TTL discipline** (absolute + sliding where appropriate), **eviction and invalidation** (tags, keys, or versioning), **stampede protection** (locking, output caching resource locking), **memory limits** (`SizeLimit` or dedicated cache instances), **key hygiene** (no external input as keys, namespaced keys), **security** (never cache sensitive user-specific data in shared caches), and **observability** (hit/miss ratios, eviction counts). |
| **Beginner-Friendly Explanation** | Best practices are the **“do's and don'ts”** for caching — they help you avoid the classic disasters: a cache that grows until the server runs out of memory, a stale product page that never updates, a cache stampede that crashes the database when a popular entry expires, and a security breach from caching one user's data and serving it to another. |

### Purposes

- **To** ensure **code always works without the cache** — the cache is an optimization, not a source of truth.
- **To** prevent **unbounded memory growth** — use expirations and size limits.
- **To** avoid **cache stampede** — use locking or output caching's resource locking.
- **To** keep cached data **fresh** — invalidate on writes or use short TTLs.
- **To** protect **security** — never cache per-user data in a shared cache without proper isolation.

### Guidelines Checklist

| Area | Do | Don't |
|------|----|-------|
| **Fallback** | Always have a path to fetch data if the cache is empty or unavailable. | Assume the cache is always populated. |
| **TTL** | Use absolute expiration for static data; sliding for frequently accessed data. | Cache forever without an eviction strategy. |
| **Size limits** | Use `SetSize`/`SizeLimit` on dedicated cache instances; monitor memory. | Let the cache grow unbounded. |
| **Keys** | Use namespaced, validated keys (e.g., `catalog:product:42`). | Use raw external input as cache keys. |
| **Stampede** | Use output caching resource locking or a `SemaphoreSlim` around `GetOrCreate`. | Let many concurrent requests all miss and hit the database. |
| **Security** | Never cache user-specific data in `IMemoryCache` or a shared Redis instance without isolation. | Cache authorization-sensitive responses in a shared cache. |
| **Invalidation** | Use output caching tags or explicit `Remove` calls when data changes. | Let stale data live until TTL expiry. |
| **Observability** | Track hit/miss ratios, eviction counts, and latency. | Cache without measuring whether it helps. |

### Annotated Code Example (Best-Practice Pattern)

```csharp
public async Task<Product?> GetProductAsync(int id)
{
    var key = $"catalog:product:{id}";
    if (_cache.TryGetValue(key, out Product? cached))
        return cached;

    var product = await _db.Products.AsNoTracking()
        .FirstOrDefaultAsync(p => p.Id == id);

    if (product is not null)
    {
        _cache.Set(key, product, new MemoryCacheEntryOptions
        {
            SlidingExpiration = TimeSpan.FromMinutes(10),
            AbsoluteExpirationRelativeToNow = TimeSpan.FromHours(1)
        });
    }
    return product;
}

// Invalidation on update
public async Task UpdateProductAsync(Product product)
{
    await _db.SaveChangesAsync();
    _cache.Remove($"catalog:product:{product.Id}");
}
```

**Why this follows best practices:**

- **Namespaced key** (`catalog:product:{id}`) avoids collisions.
- **Sliding + absolute TTL** bounds both idle and total lifetime.
- **Fallback to database** ensures the code works without the cache.
- **Explicit invalidation** on update keeps the cache fresh.

### Real-World Cases

- **E-commerce:** Cache product details with tag invalidation on price/stock updates.
- **SaaS dashboards:** Cache reference data in Redis with 5-minute TTL; never cache user-specific rows in the shared cache.
- **High-traffic APIs:** Use output caching with resource locking to prevent stampede.

### References

- Cache guidelines (Microsoft Learn) – https://learn.microsoft.com/el-gr/aspnet/core/performance/caching/memory?view=aspnetcore-3.1#2
- Output caching vs response caching (SSW Rules) – https://www.ssw.com.au/rules/do-you-use-output-caching-for-better-api-performance
- Caching best practices (MojoAuth) – https://mojoauth.com

---

## Summary Table: Caching Mechanisms at a Glance

| Mechanism | Storage | Scope | Control | Best For |
|-----------|---------|-------|---------|----------|
| **`IMemoryCache`** | Server memory | Single process | TTL, size | Fastest access; single-server apps |
| **`IDistributedCache`** | Redis, SQL Server | All instances | TTL | Shared data across instances |
| **Response caching** | Client/proxy | Per-client | HTTP headers | Public GET responses; CDN offload |
| **Output caching** | Server (extensible) | Per-server (or distributed) | Policies, tags | Server-controlled response caching with invalidation |
| **Redis** | External in-memory | All instances | TTL, eviction | Production distributed cache |
| **Best practices** | — | — | — | Every caching implementation |

---

## References (Full List)

- Overview of caching in ASP.NET Core – https://github.com/dotnet/aspnetcore.docs/blob/0885e7518455420953be73e5e0d36c490b314dde/aspnetcore/performance/caching/overview.md
- Cache in-memory in ASP.NET Core – https://learn.microsoft.com/el-gr/aspnet/core/performance/caching/memory?view=aspnetcore-3.1#2
- Distributed caching in ASP.NET Core – https://learn.microsoft.com/hi-in/aspnet/core/performance/caching/distributed?view=aspnetcore-3.1#2
- Response caching in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/performance/caching/response
- Response caching middleware – https://github.com/dotnet/AspNetCore.Docs/blob/634aa0e227eafee34f6f86ca032a1f087b5b0b15/aspnetcore/performance/caching/response.md
- Output caching middleware in ASP.NET Core – https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/performance/caching/output.md
- Output caching vs response caching (SSW Rules) – https://www.ssw.com.au/rules/do-you-use-output-caching-for-better-api-performance
- `IMemoryCache` interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.caching.memory.imemorycache
- `IDistributedCache` interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.caching.distributed.idistributedcache
- `AddStackExchangeRedisCache` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.stackexchangerediscacheextensions.addstackexchangerediscache
- GetOrCreate pattern (OneUptime) – https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-01-27-dotnet-imemorycache-redis-caching/README.md
- Redis distributed cache example (OneUptime) – https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-03-31-redis-aspnet-core-distributed-cache/README.md
- Cache guidelines (Microsoft Learn) – https://learn.microsoft.com/el-gr/aspnet/core/performance/caching/memory?view=aspnetcore-3.1#2
- ASP.NET Core caching overview (GitHub) – https://github.com/dotnet/aspnetcore.docs/blob/0885e7518455420953be73e5e0d36c490b314dde/aspnetcore/performance/caching/overview.md