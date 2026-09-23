# Caching Strategies & Performance Tuning: A Comprehensive Cheat Sheet

## Topic Overview

### Definitions

**Core Definition:** Caching strategies and performance tuning in Laravel encompass the architectural decisions and implementation patterns used to store frequently accessed data in fast-access layers, balance staleness against performance, and proactively prepare cache entries to avoid latency spikes.

**Technical Definition:** Laravel's caching layer is managed by the `CacheManager`, which resolves stores implementing the `Repository` contract. Performance tuning involves selecting appropriate drivers (Redis, Memcached vs. file, database), applying invalidation strategies (TTL, manual eviction, event-driven), grouping keys with tags (supported only by Redis, Memcached, and DynamoDB), pre-warming caches via Artisan commands, and implementing stale-while-revalidate patterns to maintain availability under load.

**Beginner-Friendly Explanation:** Caching is about keeping data on your desk instead of walking to the filing cabinet every time. But a cache is only useful if you manage it correctly: how long does data stay? When do you clear it? How do you group related items? And how do you make sure the first visitor after a deploy doesn't wait 4 seconds for a slow query? This cheat sheet covers the strategies that make caching work in production.

### Key Characteristics

- **Driver-Dependent Features:** Tags and atomic locks work only with Redis, Memcached, and DynamoDB — not file, database, or array drivers .
- **Invalidation is Critical:** Caching without proper invalidation ships stale data .
- **Stale-While-Revalidate:** `Cache::flexible()` serves stale data while recomputing in the background, preventing stampedes .
- **Pre-Warming Eliminates Cold Starts:** Scheduled commands populate caches before users arrive .
- **Event-Driven Invalidation:** Model events (`saved`, `deleted`) automatically flush related cache entries .

### Prerequisites

- A Laravel application with `config/cache.php` configured.
- Redis or Memcached for production-grade tags and locks.
- Understanding of the `Cache` facade and the `remember()` pattern.
- Artisan scheduler configured for pre-warming commands.

### Related Programming Areas

- **Queue Processing:** Cache invalidation can be dispatched as jobs.
- **Eloquent Model Events:** Observers handle cache invalidation on model changes.
- **Task Scheduling:** Pre-warming commands run on a schedule.
- **Session Management:** Can share Redis infrastructure with cache.

### Core Concepts / Features

1. Query Caching (When to Use and Avoid)
2. Cache Invalidation Lifecycle (TTL vs. Manual Eviction)
3. Tagged Caching (`Cache::tags()`) and Driver Limitations
4. Cache Warming & Prerendering
5. Stale-While-Revalidate and Fallback Cache Patterns

---

## 1. Query Caching: Safely Offloading Heavy Database Lookups

### Definitions

**Core Definition:** Query caching stores the results of database queries in a fast-access cache layer, reducing database load for frequently executed, read-heavy queries.

**Technical Definition:** Query caching is implemented via `Cache::remember($key, $ttl, $callback)` where the callback executes the database query. The result is serialized and stored. On subsequent calls within the TTL, the cached result is returned without hitting the database. For Eloquent, packages like `codewithdennis/laravel-model-cache` add a `HasCache` trait that automatically caches queries .

**Beginner-Friendly Explanation:** Instead of running the same `SELECT * FROM products` query on every page load, you run it once, store the result, and serve it from memory for the next hour. But if a product is updated, you need to clear that cache or your users see old data.

### Purposes

- To reduce database load for read-heavy, write-light queries.
- To improve response times for expensive queries (joins, aggregations).
- To cache reference data that rarely changes (settings, categories).
- To offload complex reporting queries from the primary database.

### Syntax Rules and Structure

**Complete General Syntax — Basic Query Caching:**

```php
use Illuminate\Support\Facades\Cache;

$products = Cache::remember('products:all', 3600, function () {
    return Product::with('category')->get();
});
```

**Complete General Syntax — Model-Based Caching (Package):**

```php
use CodeWithDennis\LaravelModelCache\Traits\HasCache;

class Product extends Model
{
    use HasCache;

    protected int $cacheTtl = 300; // 5 minutes
}

// Queries are automatically cached
$products = Product::where('active', true)->get();
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Cache::remember($key, $ttl, $callback)` | Retrieves or computes and caches. |
| `$key` | Unique identifier for the cached query. |
| `$ttl` | Time-to-live in seconds. |
| `$callback` | The database query to execute on cache miss. |

**Syntax Rules:**
- Cache keys must be unique per query variation (e.g., include filter parameters).
- TTL should match the data's volatility. Settings might use `rememberForever()`, product lists might use 5 minutes .
- Model-based caching packages automatically invalidate on `saved`, `deleted`, `restored` events .

**Constraints and Limitations:**
- **Stale data:** Cached queries return old data if the underlying table changes without invalidation.
- **N+1 risk:** Caching queries doesn't fix relationship N+1 problems; those require eager loading.
- **Memory usage:** Large result sets consume cache memory. Redis eviction policies apply.
- **Write-heavy tables:** Avoid caching queries on tables with frequent writes — invalidation overhead outweighs benefits.

### Multiple Annotated Complete Code Examples

**Example 1: Caching a Product List**

```php
<?php
use Illuminate\Support\Facades\Cache;
use App\Models\Product;

$products = Cache::remember('products:featured', 1800, function () {
    return Product::where('featured', true)
        ->with('category')
        ->orderBy('name')
        ->get();
});

// First call: database query executed
// Subsequent calls (within 30 min): served from cache
```

**Expected Output:** The featured products are cached for 30 minutes. Repeated calls return the cached collection.

**Why:** `remember()` checks the cache first. On miss, it executes the callback and stores the result .

---

**Example 2: Event-Driven Invalidation**

```php
<?php
// app/Models/Product.php

protected static function booted(): void
{
    static::saved(function (Product $product) {
        Cache::forget('products:featured');
    });

    static::deleted(function (Product $product) {
        Cache::forget('products:featured');
    });
}
```

**Expected Output:** When a product is created, updated, or deleted, the cached featured products list is cleared. The next request recomputes and re-caches.

**Why:** Model events trigger cache invalidation, ensuring stale data is not served after changes .

---

**Example 3: When to Avoid Caching**

```php
<?php
// ❌ Bad: Caching a query on a write-heavy table
$orders = Cache::remember('orders:recent', 300, function () {
    return Order::latest()->take(50)->get();
});
// Orders are created constantly; cache is invalidated immediately

// ✅ Good: Cache reference data that rarely changes
$categories = Cache::rememberForever('categories:tree', function () {
    return Category::with('children')->get();
});
```

**Expected Output:** The orders cache provides no benefit (constantly invalidated). The categories cache is effective (rarely changes).

**Why:** Caching is beneficial for read-heavy, write-light data. Write-heavy tables cause constant invalidation .

### Real-World Cases

- **Settings:** `Cache::rememberForever('app:settings', fn() => Setting::all()->pluck('value', 'key'))` — settings are read constantly but written rarely .
- **Category Trees:** Cache for hours; invalidate on category save.
- **Dashboard Stats:** Cache for 5-15 minutes with event-driven invalidation.

### References

- Eloquent Caching: Main Things You Need To Know (Laravel Daily) — https://laraveldaily.com/post/eloquent-caching-things-you-need-to-know
- The Laravel pattern behind yesterday's 100k-query fix (LinkedIn) — https://www.linkedin.com/posts/bilal-ahmad-ai_the-laravel-pattern-behind-yesterdays-100k-query-activity-7488201685664870400--g7-
- codewithdennis/laravel-model-cache (Packagist) — https://packagist.org/packages/codewithdennis/laravel-model-cache

---

## 2. Cache Invalidation Lifecycle: TTL vs. Manual Eviction

### Definitions

**Core Definition:** Cache invalidation determines when cached data is removed — either automatically after a time-to-live (TTL) expires or manually via `forget()` or `flush()`.

**Technical Definition:** TTL-based expiration sets a timestamp on cached entries; the driver removes them when accessed after expiration. Manual eviction uses `Cache::forget($key)` to remove specific entries or `Cache::flush()` to clear the entire store. Event-driven invalidation combines both: model events trigger `forget()` calls. The `cache:clear` Artisan command clears the entire cache store.

**Beginner-Friendly Explanation:** TTL is like a self-destruct timer. Manual eviction is like throwing something away when you know it's outdated. The best strategy often combines both: set a reasonable TTL as a safety net, and manually clear when data changes.

### Purposes

- To ensure stale data is removed from the cache.
- To balance freshness (short TTL) against performance (long TTL).
- To tie invalidation to data changes using model events.
- To provide a safety net for cases where manual invalidation is missed.

### Syntax Rules and Structure

**Complete General Syntax — TTL-Based:**

```php
// Cache for 1 hour
Cache::put('key', $value, 3600);

// Cache with Carbon instance
Cache::put('key', $value, now()->addHours(1));

// Remember with TTL
Cache::remember('key', 3600, fn() => 'value');
```

**Complete General Syntax — Manual Eviction:**

```php
// Remove a specific key
Cache::forget('key');

// Remove multiple keys
Cache::forget(['key1', 'key2']);

// Clear the entire cache store
Cache::flush();
```

**Complete General Syntax — Event-Driven Invalidation:**

```php
// In model booted()
static::saved(function ($model) {
    Cache::forget("model:{$model->id}");
    Cache::forget('model:list');
});
```

**Component Breakdown:**

| Strategy | Trigger | Use Case |
|----------|---------|----------|
| TTL | Time elapsed | Data with known freshness window |
| `forget()` | Explicit call | Known data changes |
| `flush()` | Explicit call | Full cache reset (deployments) |
| Model events | Data change | Automatic invalidation |

**Syntax Rules:**
- TTL is always in **seconds** unless using a `DateTimeInterface` instance.
- `forget()` accepts a string or array of keys.
- `flush()` clears the entire store (not tag-scoped unless using tags).
- Model events (`saved`, `deleted`, `restored`) are ideal for automatic invalidation .

**Constraints and Limitations:**
- **File/database drivers:** Expired entries are not automatically deleted; they are removed on next access .
- **`flush()` is broad:** It clears everything in the store, not just related entries.
- **Missed invalidation:** Without event-driven invalidation, manual `forget()` calls can be forgotten, leading to stale data .
- **TTL reset:** Updating a cached value resets its TTL.

### Multiple Annotated Complete Code Examples

**Example 1: TTL with Event-Driven Safety Net**

```php
<?php
// Cache with 1-hour TTL
$settings = Cache::remember('app:settings', 3600, function () {
    return Setting::all()->pluck('value', 'key');
});

// Event-driven invalidation: clear when settings change
protected static function booted(): void
{
    static::saved(function (Setting $setting) {
        Cache::forget('app:settings');
    });
}
```

**Expected Output:** Settings are cached for 1 hour. If any setting is updated, the cache is cleared immediately. If no updates occur, the cache refreshes after 1 hour.

**Why:** TTL provides a safety net; event-driven invalidation ensures freshness when data changes .

---

**Example 2: Manual Eviction for Bulk Updates**

```php
<?php
// Bulk update all products
Product::where('category_id', $categoryId)->update(['featured' => false]);

// Clear related caches manually
Cache::forget('products:featured');
Cache::forget("products:category:{$categoryId}");
```

**Expected Output:** After a bulk update (which bypasses model events), caches are manually cleared.

**Why:** Mass updates via query builder do **not** fire model events. Manual invalidation is required .

---

**Example 3: Flush on Deployment**

```bash
# In deployment script
php artisan cache:clear
```

```php
<?php
// Or programmatically
Cache::flush();
```

**Expected Output:** The entire cache store is cleared. All subsequent requests recompute and re-cache.

**Why:** Clearing the cache on deployment ensures new code serves fresh data .

### Real-World Cases

- **Settings:** `rememberForever()` + `forget()` on save .
- **Product Lists:** TTL of 5-15 minutes + invalidation on product save.
- **Deployments:** `cache:clear` in deployment scripts .

### References

- Eloquent Caching: Main Things You Need To Know (Laravel Daily) — https://laraveldaily.com/post/eloquent-caching-things-you-need-to-know
- The Laravel pattern behind yesterday's 100k-query fix (LinkedIn) — https://www.linkedin.com/posts/bilal-ahmad-ai_the-laravel-pattern-behind-yesterdays-100k-query-activity-7488201685664870400--g7-
- Cache Pre-warming Explained (Laravel News) — https://laravel-news.com/cache-pre-warming-explained-laravel-in-practice-ep11

---

## 3. Tagged Caching: Grouping Related Items

### Definitions

**Core Definition:** Cache tags allow grouping related cache entries under one or more labels, enabling broad invalidation of all entries with a specific tag using `Cache::tags(['tag'])->flush()`.

**Technical Definition:** Tags are implemented only by Redis, Memcached, and DynamoDB drivers. The `Cache::tags(['tag1', 'tag2'])->put($key, $value, $ttl)` method associates entries with tags. `Cache::tags(['tag1'])->flush()` removes all entries associated with `tag1`. File, database, and array drivers do **not** support tags and will throw an exception if used .

**Beginner-Friendly Explanation:** Tags let you group cache entries. If you cache posts, comments, and likes all related to a specific user, you can tag them all with `user:1`. When that user's data changes, you flush the `user:1` tag and all related caches are cleared at once.

### Purposes

- To group related cache entries under a common label.
- To invalidate multiple keys with a single `flush()` call.
- To avoid tracking individual cache keys for invalidation.
- To enable broad cache clearing without flushing the entire store.

### Syntax Rules and Structure

**Complete General Syntax — Tagging:**

```php
use Illuminate\Support\Facades\Cache;

// Store with tags
Cache::tags(['posts', 'user:1'])->put('user:1:posts', $posts, 3600);

// Retrieve with tags
$posts = Cache::tags(['posts', 'user:1'])->get('user:1:posts');

// Remember with tags
$posts = Cache::tags(['posts', 'user:1'])->remember('user:1:posts', 3600, function () {
    return Post::where('user_id', 1)->get();
});

// Flush by tag
Cache::tags(['user:1'])->flush(); // Clears all caches tagged 'user:1'
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `Cache::tags([...])` | Returns a tagged cache repository. |
| `->put($key, $value, $ttl)` | Stores with tags. |
| `->get($key)` | Retrieves with tags. |
| `->remember(...)` | Remember pattern with tags. |
| `->flush()` | Flushes all entries with the tag. |

**Complete General Syntax — Supported Drivers:**

```php
// config/cache.php
'default' => env('CACHE_STORE', 'redis'),

'stores' => [
    'redis' => [
        'driver' => 'redis',
        'connection' => 'cache',
    ],
    'memcached' => [
        'driver' => 'memcached',
        // ...
    ],
],
```

**Syntax Rules:**
- Tags require Redis, Memcached, or DynamoDB drivers. File, database, and array drivers **do not support tags** and will throw an exception .
- Multiple tags can be applied: `Cache::tags(['posts', 'user:1'])`.
- `flush()` on a tag removes all entries with that tag.
- Tag names should be descriptive and follow a naming convention (e.g., `user:{id}`, `resource:{name}`) .

**Constraints and Limitations:**
- **Driver incompatibility:** File, database, and array drivers do not support tags .
- **Performance overhead:** Tags add metadata to cache entries, slightly increasing storage and retrieval overhead.
- **Tag explosion:** Creating too many unique tags can increase memory usage.

### Multiple Annotated Complete Code Examples

**Example 1: Tagging User-Related Data**

```php
<?php
use Illuminate\Support\Facades\Cache;

$userId = 1;

// Cache posts with tags
$posts = Cache::tags(['posts', "user:{$userId}"])->remember(
    "user:{$userId}:posts",
    3600,
    fn() => Post::where('user_id', $userId)->get()
);

// Cache comments with tags
$comments = Cache::tags(['comments', "user:{$userId}"])->remember(
    "user:{$userId}:comments",
    3600,
    fn() => Comment::where('user_id', $userId)->get()
);

// Invalidate all user data at once
Cache::tags(["user:{$userId}"])->flush();
// Both posts and comments caches are cleared
```

**Expected Output:** All cache entries tagged with `user:1` are cleared in a single `flush()` call.

**Why:** Tags group related entries, enabling broad invalidation without tracking individual keys .

---

**Example 2: Driver Compatibility Check**

```php
<?php
// ✅ Good: Redis driver supports tags
'default' => 'redis',
Cache::tags(['posts'])->put('key', 'value', 3600); // Works

// ❌ Bad: File driver does not support tags
'default' => 'file',
Cache::tags(['posts'])->put('key', 'value', 3600); // Throws exception
```

**Expected Output:** Using tags with the file driver throws an exception. Use Redis or Memcached for tag support.

**Why:** Tags require driver-level support for metadata. File and database drivers lack this capability .

---

**Example 3: Patterned Invalidation Without Tags**

```php
<?php
// Fallback for drivers without tag support
Cache::forget('user:1:posts');
Cache::forget('user:1:comments');
Cache::forget('user:1:profile');
// Manual tracking required for each key
```

**Expected Output:** Without tags, each cache key must be tracked and forgotten individually.

**Why:** When tags are unavailable (file/database drivers), fallback to careful key naming and manual eviction .

### Real-World Cases

- **User Data:** Tag all user-related caches with `user:{id}` for bulk invalidation.
- **Resource Groups:** Tag API responses with `resource:posts` for broad clearing .
- **Multi-Tenant:** Tag with `tenant:{id}` to isolate invalidation per tenant.

### References

- Status of Cache Tags in Laravel 12 (Laracasts) — https://laracasts.com/discuss/channels/laravel/status-of-cache-tags-in-laravel-12?reply=222858
- Laravel 5 cache tags on file drivers (Stack Overflow) — https://stackoverflow.com/questions/32680866/laravel-5-cache-tags-on-file-drivers
- My Laravel 12 API Was Taking 4 Seconds (Medium) — https://medium.com/@aiman.asfia/my-laravel-12-api-was-taking-4-seconds-heres-exactly-how-i-made-it-300ms-be1a1e4e6d80

---

## 4. Cache Warming & Prerendering

### Definitions

**Core Definition:** Cache warming (or pre-warming) is the process of proactively populating cache entries before they are requested by users, eliminating the "cold cache" latency spike that occurs after deployments or cache clears.

**Technical Definition:** Warming is implemented as an Artisan command that executes heavy queries and stores results using `Cache::put()`. The command is scheduled or triggered post-deployment. For Eloquent, packages like `codewithdennis/laravel-model-cache` provide a `warmup()` method that caches query results indefinitely . The command should run in a CLI context (not in `AppServiceProvider::boot()` or middleware) to avoid executing on every request .

**Beginner-Friendly Explanation:** After you clear the cache or deploy new code, the first user who visits a page triggers a slow database query. Cache warming runs that query ahead of time (via a scheduled command) so the first user gets a fast response. It's like pre-heating the oven before guests arrive.

### Purposes

- To eliminate cold-start latency spikes after deployments or cache clears.
- To ensure critical pages load quickly for the first user.
- To populate caches for high-traffic periods (e.g., morning traffic after overnight cache expiry).
- To integrate with deployment scripts for zero-downtime cache readiness.

### Syntax Rules and Structure

**Complete General Syntax — Artisan Command:**

```php
<?php
// app/Console/Commands/CacheWarmup.php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use Illuminate\Support\Facades\Cache;
use App\Models\Product;
use App\Models\Category;

class CacheWarmup extends Command
{
    protected $signature = 'cache:warmup';
    protected $description = 'Preload critical data into cache';

    public function handle(): void
    {
        // Warm categories tree
        Cache::put('categories_tree', Category::with('children')->get(), 7200);

        // Warm featured products
        Cache::put('products:featured', Product::where('featured', true)->get(), 1800);

        $this->info('Cache warmed successfully.');
    }
}
```

**Complete General Syntax — Scheduling:**

```php
<?php
// routes/console.php (Laravel 11+) or Kernel.php

use Illuminate\Support\Facades\Schedule;

Schedule::command('cache:warmup')->hourly();
Schedule::command('cache:warmup')->dailyAt('06:00');
```

**Complete General Syntax — Deployment Script:**

```bash
#!/bin/bash
php artisan optimize:clear
php artisan config:cache
php artisan cache:warmup  # Warm before serving traffic
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `cache:warmup` | Artisan command signature. |
| `Cache::put()` | Store precomputed data. |
| `Schedule::command()` | Register scheduled warming. |
| Deployment script | Run after cache clear, before traffic. |

**Complete General Syntax — Model Cache Package Warmup:**

```php
<?php
// codewithdennis/laravel-model-cache
$stats = User::warmup()
    ->where('created_at', '>=', now()->startOfMonth())
    ->orderBy('created_at')
    ->get();
// Cached indefinitely; first user gets cached result
```

**Syntax Rules:**
- Warming commands run in CLI context, not in web request lifecycle .
- Use `Cache::put()` with explicit TTL rather than `rememberForever()` for warming .
- Schedule warming to run **before** traffic peaks (e.g., 6 AM for business hours) .
- Integrate warming into deployment scripts after `cache:clear` and `config:cache` .
- For Eloquent, `warmup()` stores results without expiry until invalidated .

**Constraints and Limitations:**
- **CLI environment:** The CLI environment must have access to the database and Redis. Docker network issues can cause connection failures .
- **Timing:** Warming must complete before traffic hits. If it fails, users experience cold cache.
- **Memory:** Warming large datasets consumes memory; chunk if necessary.
- **Invalidation:** Warmed caches must be invalidated on data changes, or they serve stale data indefinitely.

### Multiple Annotated Complete Code Examples

**Example 1: Basic Warming Command**

```php
<?php
namespace App\Console\Commands;

use Illuminate\Console\Command;
use Illuminate\Support\Facades\Cache;
use App\Models\Post;
use App\Models\Category;

class CacheWarmup extends Command
{
    protected $signature = 'cache:warmup';
    protected $description = 'Preload critical data into cache';

    public function handle(): void
    {
        // Warm categories (rarely changes, cache for 2 hours)
        $categories = Category::with('children')->get();
        Cache::put('categories_tree', $categories, 7200);

        // Warm popular posts (changes hourly, cache for 30 min)
        $popular = Post::popular()->take(8)->get();
        Cache::put('homepage_popular_posts', $popular, 1800);

        $this->info('Cache warmed: categories + popular posts');
    }
}
```

**Expected Output:** Running `php artisan cache:warmup` populates two cache keys. The next user requesting these pages gets cached data immediately.

**Why:** Warming precomputes expensive queries before users arrive, eliminating the cold-cache delay .

---

**Example 2: Scheduled Warming**

```php
<?php
// routes/console.php

use Illuminate\Support\Facades\Schedule;

// Warm every hour during business hours
Schedule::command('cache:warmup')->hourly();

// Or specifically at 6 AM before morning traffic
Schedule::command('cache:warmup')->dailyAt('06:00');
```

**Expected Output:** The cache is refreshed hourly or at 6 AM, ensuring warm data during peak usage.

**Why:** Scheduling warming aligns cache freshness with traffic patterns .

---

**Example 3: Warming with Model Cache Package**

```php
<?php
// Using codewithdennis/laravel-model-cache
use App\Models\User;

// Run in an Artisan command
$stats = User::warmup()
    ->where('created_at', '>=', now()->startOfMonth())
    ->orderBy('created_at')
    ->get();

// The query is cached indefinitely (no TTL)
// First user request gets the cached result
```

**Expected Output:** The query result is cached without expiry. The first user gets the cached result instead of triggering a slow query.

**Why:** `warmup()` stores results indefinitely, and the package invalidates on model events .

### Real-World Cases

- **Homepage:** Warm featured products, categories, and popular posts before morning traffic .
- **Dashboard:** Warm statistics after nightly data refresh.
- **API Endpoints:** Warm frequently accessed JSON responses after deployment.
- **Deployments:** Run warming in deployment scripts to ensure first users get fast responses .

### References

- Cache Pre-warming Explained (Laravel News) — https://laravel-news.com/cache-pre-warming-explained-laravel-in-practice-ep11
- codewithdennis/laravel-model-cache (Packagist) — https://packagist.org/packages/codewithdennis/laravel-model-cache
- Laravel如何做缓存预热 (php.cn) — https://www.php.cn/faq/2438014.html
- Laravel高并发下如何预热缓存 (php.cn) — https://www.php.cn/faq/2529424.html

---

## 5. Stale-While-Revalidate and Fallback Cache Patterns

### Definitions

**Core Definition:** Stale-while-revalidate is a caching strategy where stale (expired) data is served immediately while a background process recomputes and refreshes the cache, preventing latency spikes and cache stampedes.

**Technical Definition:** `Cache::flexible($key, [$freshSeconds, $staleSeconds], $callback)` implements this pattern. Within the fresh period, the cached value is returned directly. Between fresh and stale expiry, the stale value is returned while a deferred refresh is triggered. After stale expiry, the callback executes synchronously. The `hybrid-cache` package extends this with a local + distributed two-layer architecture .

**Beginner-Friendly Explanation:** Normally, when a cache expires, the next user waits for the slow database query. Stale-while-revalidate says "serve the old data immediately, and refresh it in the background." Users never wait, and the cache stays warm.

### Purposes

- To prevent cache stampedes (thundering herd) when popular keys expire.
- To maintain low latency even after cache expiration.
- To improve availability by serving stale data during backend issues.
- To combine local and distributed caches for ultra-fast reads.

### Syntax Rules and Structure

**Complete General Syntax — `Cache::flexible()`:**

```php
use Illuminate\Support\Facades\Cache;

// Fresh for 5 seconds, stale for 10 seconds
$value = Cache::flexible('key', [5, 10], function () {
    return expensiveComputation();
});
```

**Component Breakdown:**

| Parameter | Description |
|-----------|-------------|
| `$key` | Cache key. |
| `[5, 10]` | `[freshSeconds, staleSeconds]`. |
| `$callback` | Computation executed on refresh. |

**Complete General Syntax — Hybrid Cache (Package):**

```php
use Rajmundtoth0\HybridCache\Facades\HybridCache;

$value = HybridCache::flexible('key', [300, 60], function () {
    return expensiveComputation();
});
```

**Syntax Rules:**
- `flexible()` requires the `memcached`, `redis`, `dynamodb`, `database`, `file`, or `array` driver .
- The fresh period is when the cached value is considered current.
- The stale period is when the cached value is served while refreshing.
- After stale expiry, the callback runs synchronously.

**Constraints and Limitations:**
- **Stale data:** Users may see data that is up to `staleSeconds` old.
- **Background refresh:** Requires a mechanism to trigger the refresh (Laravel handles this via deferred execution).
- **Driver support:** Not all drivers support `flexible()` equally.

### Multiple Annotated Complete Code Examples

**Example 1: Basic Stale-While-Revalidate**

```php
<?php
use Illuminate\Support\Facades\Cache;

// Fresh for 5 minutes, stale for 10 minutes
$trending = Cache::flexible('trending:posts', [300, 600], function () {
    return Post::withCount('views')
        ->orderByDesc('views_count')
        ->take(10)
        ->get();
});
```

**Expected Output:**
- 0-5 minutes: Returns fresh cached value instantly.
- 5-15 minutes: Returns stale value, refreshes in background.
- After 15 minutes: Executes callback, stores fresh value.

**Why:** `flexible()` prevents the latency spike when a popular cache key expires .

---

**Example 2: Hybrid Cache with Local + Distributed**

```php
<?php
use Rajmundtoth0\HybridCache\Facades\HybridCache;

// Configure hybrid: APCu local + Redis distributed
// In config/cache.php
'hybrid' => [
    'driver' => 'hybrid',
    'local_store' => 'apc',
    'distributed_store' => 'redis',
],

// Usage
$data = Cache::store('hybrid')->flexible('heavy:report', [300, 60], function () {
    return buildComplexReport();
});
```

**Expected Output:** The local APCu cache serves reads in microseconds. If missing, the distributed Redis cache is checked. Stale data is served while refreshing.

**Why:** Hybrid caching combines the speed of local memory with the shared state of distributed cache .

---

**Example 3: Fallback Cache for Availability**

```php
<?php
use Illuminate\Support\Facades\Cache;

// Fallback: if Redis is down, use database cache
try {
    $settings = Cache::store('redis')->remember('settings', 3600, function () {
        return Setting::all();
    });
} catch (\Exception $e) {
    $settings = Cache::store('database')->remember('settings', 3600, function () {
        return Setting::all();
    });
}
```

**Expected Output:** If Redis is unavailable, the database cache store is used as a fallback, maintaining availability.

**Why:** Fallback patterns ensure the application continues functioning even if the primary cache store fails .

### Real-World Cases

- **Trending Content:** `Cache::flexible('trending', [60, 120], fn() => getTrending())` — always fast, slightly stale.
- **Dashboard Stats:** `flexible()` for statistics that tolerate brief staleness.
- **API Responses:** Serve stale JSON while refreshing for high-traffic endpoints.
- **High Availability:** Fallback to database cache if Redis fails.

### References

- Laravel 11.x Cache Documentation (SWR) — https://laravel.com/framework/docs/11.x/cache#swr
- My Laravel 12 API Was Taking 4 Seconds (Medium) — https://medium.com/@aiman.asfia/my-laravel-12-api-was-taking-4-seconds-heres-exactly-how-i-made-it-300ms-be1a1e4e6d80
- rajmundtoth0/hybrid-cache (Packagist) — https://packagist.org/packages/rajmundtoth0/hybrid-cache

---

## References

- Laravel 11.x Cache Documentation (SWR) — https://laravel.com/framework/docs/11.x/cache#swr
- Status of Cache Tags in Laravel 12 (Laracasts) — https://laracasts.com/discuss/channels/laravel/status-of-cache-tags-in-laravel-12?reply=222858
- Laravel 5 cache tags on file drivers (Stack Overflow) — https://stackoverflow.com/questions/32680866/laravel-5-cache-tags-on-file-drivers
- Cache Pre-warming Explained (Laravel News) — https://laravel-news.com/cache-pre-warming-explained-laravel-in-practice-ep11
- Eloquent Caching: Main Things You Need To Know (Laravel Daily) — https://laraveldaily.com/post/eloquent-caching-things-you-need-to-know
- The Laravel pattern behind yesterday's 100k-query fix (LinkedIn) — https://www.linkedin.com/posts/bilal-ahmad-ai_the-laravel-pattern-behind-yesterdays-100k-query-activity-7488201685664870400--g7-
- codewithdennis/laravel-model-cache (Packagist) — https://packagist.org/packages/codewithdennis/laravel-model-cache
- rajmundtoth0/hybrid-cache (Packagist) — https://packagist.org/packages/rajmundtoth0/hybrid-cache
- Laravel如何做缓存预热 (php.cn) — https://www.php.cn/faq/2438014.html
- Laravel高并发下如何预热缓存 (php.cn) — https://www.php.cn/faq/2529424.html
- My Laravel 12 API Was Taking 4 Seconds (Medium) — https://medium.com/@aiman.asfia/my-laravel-12-api-was-taking-4-seconds-heres-exactly-how-i-made-it-300ms-be1a1e4e6d80