# Caching Architecture & Key-Value Storage: A Comprehensive Cheat Sheet

## Topic Overview

### Definitions

**Core Definition:** Caching architecture in Laravel is the strategic storage of frequently accessed data in a fast-access layer (Redis, Memcached, or files) to reduce database load and improve response times, with key-value storage patterns providing atomic operations for concurrency-safe data manipulation.

**Technical Definition:** Laravel's caching system is managed by the `CacheManager` class, which resolves cache stores implementing the `Illuminate\Contracts\Cache\Repository` contract. Each store implements the `Illuminate\Contracts\Cache\Store` interface, defining methods like `get()`, `put()`, `increment()`, `decrement()`, `forever()`, and `forget()`. The "remember" pattern combines retrieval and storage in a single atomic operation. Distributed locks (`Cache::lock()`) provide mutual exclusion across processes using Redis or Memcached. Atomic operations (`increment()`, `decrement()`) modify counter values without race conditions.

**Beginner-Friendly Explanation:** Caching is like keeping frequently used items on your desk instead of walking to the filing cabinet every time. Redis and Memcached are fast in-memory stores that work across multiple servers. The `remember()` pattern says "give me the cached data, but if it's not there, fetch it and store it for next time." Atomic locks prevent two processes from doing the same thing simultaneously, and atomic increment/decrement let you safely modify counters.

### Key Characteristics

- **Distributed-Ready:** Redis and Memcached stores share data across multiple application servers .
- **Serialization-Aware:** All cached items are serialized, allowing models, collections, and arrays to be stored safely .
- **Atomic Operations:** `increment()`, `decrement()`, and `add()` are atomic at the store level .
- **Remember Pattern:** `remember()` and `rememberForever()` encapsulate the retrieval-or-compute logic .
- **Distributed Locks:** `Cache::lock()` provides cross-process mutual exclusion with automatic expiry .
- **Stale-While-Revalidate:** `Cache::flexible()` serves stale data while recomputing in the background .

### Prerequisites

- A Laravel application with cache configuration in `config/cache.php`.
- A cache driver configured in `.env` (`CACHE_STORE`).
- For Redis: `predis/predis` package or the PHP Redis extension.
- For Memcached: the Memcached PECL extension.
- For database cache: a `cache` table migration.

### Related Programming Areas

- **Session Management:** Sessions can use the same Redis/Memcached stores.
- **Queue Processing:** Redis is commonly used for both cache and queue backends.
- **Eloquent Model Caching:** Packages like `genealabs/laravel-model-caching` extend caching to model queries .
- **Rate Limiting:** Cache atomic operations power Laravel's rate limiter.

---

## 1. Cache Store Configuration: Distributed vs. Localized

### Definitions

**Core Definition:** Cache store configuration determines where cached data is physically stored and how it is accessed across application instances.

**Technical Definition:** The `config/cache.php` file defines stores under the `stores` key, each with a `driver` (Redis, Memcached, file, database, array, etc.) and driver-specific configuration. The `default` key determines which store is used by the `Cache` facade unless explicitly specified via `Cache::store('name')`. Distributed stores (Redis, Memcached) provide shared access across multiple application servers, while localized stores (file, array) are limited to a single process or server.

**Beginner-Friendly Explanation:** Where should your cached data live? If you have one server, a file cache is fine. If you have multiple servers behind a load balancer, you need a shared cache like Redis or Memcached so all servers see the same data. Laravel lets you configure multiple cache "stores" and choose which one to use.

### Purposes

- To select the appropriate storage backend for application scale.
- To enable horizontal scaling with shared cache stores.
- To provide high-performance in-memory caching with Redis/Memcached.
- To support local development with file or array drivers.
- To allow multiple cache stores for different data types (e.g., Redis for sessions, database for application cache).

### Syntax Rules and Structure

**Complete General Syntax — Configuration:**

```php
// config/cache.php
return [
    'default' => env('CACHE_STORE', 'redis'),

    'stores' => [
        'array' => [
            'driver' => 'array',
            'serialize' => false,
        ],

        'database' => [
            'driver' => 'database',
            'connection' => env('DB_CACHE_CONNECTION'),
            'table' => env('DB_CACHE_TABLE', 'cache'),
            'lock_connection' => env('DB_CACHE_LOCK_CONNECTION'),
            'lock_table' => env('DB_CACHE_LOCK_TABLE'),
        ],

        'file' => [
            'driver' => 'file',
            'path' => storage_path('framework/cache/data'),
            'lock_path' => storage_path('framework/cache/data'),
        ],

        'memcached' => [
            'driver' => 'memcached',
            'persistent_id' => env('MEMCACHED_PERSISTENT_ID'),
            'sasl' => [
                env('MEMCACHED_USERNAME'),
                env('MEMCACHED_PASSWORD'),
            ],
            'options' => [
                // Memcached::OPT_CONNECT_TIMEOUT => 2000,
            ],
            'servers' => [
                [
                    'host' => env('MEMCACHED_HOST', '127.0.0.1'),
                    'port' => env('MEMCACHED_PORT', 11211),
                    'weight' => 100,
                ],
            ],
        ],

        'redis' => [
            'driver' => 'redis',
            'connection' => env('REDIS_CACHE_CONNECTION', 'cache'),
            'lock_connection' => env('REDIS_CACHE_LOCK_CONNECTION', 'default'),
        ],
    ],

    'prefix' => env('CACHE_PREFIX', 'laravel_cache'),
];
```

**Component Breakdown:**

| Driver | Storage | Scaling | Use Case |
|--------|---------|---------|----------|
| `array` | PHP memory | None | Testing, single request |
| `file` | Filesystem | Single server | Local development |
| `database` | Database table | Multi-server | Simple shared cache |
| `memcached` | Memcached server | Multi-server | High-performance cache |
| `redis` | Redis server | Multi-server | High-performance cache + locks |

**Complete General Syntax — Selecting a Store:**

```php
use Illuminate\Support\Facades\Cache;

// Use default store
Cache::put('key', 'value', 60);

// Use a specific store
Cache::store('redis')->put('key', 'value', 60);
Cache::store('database')->get('key');
```

**Complete General Syntax — Database Cache Table:**

```php
// Create cache table migration
php artisan cache:table
php artisan migrate
```

```php
// Example cache table schema
Schema::create('cache', function (Blueprint $table) {
    $table->string('key')->primary();
    $table->mediumText('value');
    $table->integer('expiration');
});

Schema::create('cache_locks', function (Blueprint $table) {
    $table->string('key')->primary();
    $table->string('owner');
    $table->integer('expiration');
});
```

**Syntax Rules:**
- The `default` key determines the store used by `Cache::` calls without `store()`.
- Redis is recommended for production due to performance and native tag support .
- The `prefix` key isolates cache entries for multi-tenant applications.
- Database driver requires the `cache` table (and `cache_locks` for locks).

**Constraints and Limitations:**
- **File driver:** Cannot scale across multiple servers; each server has its own filesystem .
- **Array driver:** Data is lost after the request; no persistence.
- **Database driver:** Adds database load; slower than Redis/Memcached.
- **Redis connection:** Requires a configured Redis connection in `config/database.php`.
- **Eviction policies:** Redis memory limits require an eviction policy (e.g., `allkeys-lru`) .

### Multiple Annotated Complete Code Examples

**Example 1: Redis Configuration for Production**

```env
# .env
CACHE_STORE=redis
REDIS_CLIENT=predis
REDIS_HOST=127.0.0.1
REDIS_PORT=6379
REDIS_CACHE_DB=1
```

```php
<?php
// config/database.php (redis section)
'redis' => [
    'client' => env('REDIS_CLIENT', 'phpredis'),
    'options' => [
        'cluster' => env('REDIS_CLUSTER', 'redis'),
        'prefix' => env('REDIS_PREFIX', 'laravel_database_'),
    ],
    'default' => [
        'host' => env('REDIS_HOST', '127.0.0.1'),
        'password' => env('REDIS_PASSWORD'),
        'port' => env('REDIS_PORT', '6379'),
        'database' => env('REDIS_DB', '0'),
    ],
    'cache' => [
        'host' => env('REDIS_HOST', '127.0.0.1'),
        'password' => env('REDIS_PASSWORD'),
        'port' => env('REDIS_PORT', '6379'),
        'database' => env('REDIS_CACHE_DB', '1'),
    ],
],
```

**Expected Output:** Cache data is stored in Redis database 1, separate from other Redis usage (e.g., sessions). The `cache` connection is used by default for cache operations.

**Why:** Redis provides sub-millisecond read/write performance and works across multiple application servers. Using a separate database for cache isolates it from session or queue data .

---

**Example 2: Database Cache for Simple Shared Storage**

```bash
php artisan cache:table
php artisan migrate
```

```env
# .env
CACHE_STORE=database
```

```php
<?php
// Usage
Cache::put('user_count', 100, 3600);
$count = Cache::get('user_count');
```

**Expected Output:** Cache data is stored in the `cache` table and shared across all application servers connected to the same database.

**Why:** The database driver provides simple shared caching without requiring Redis or Memcached infrastructure .

---

**Example 3: Multi-Store Configuration**

```php
<?php
// config/cache.php
return [
    'default' => env('CACHE_STORE', 'redis'),

    'stores' => [
        'redis' => [
            'driver' => 'redis',
            'connection' => 'cache',
        ],
        'file' => [
            'driver' => 'file',
            'path' => storage_path('framework/cache/data'),
        ],
    ],
];
```

```php
<?php
// Use different stores for different purposes
Cache::store('redis')->put('heavy_data', $largeDataset, 3600);
Cache::store('file')->put('local_config', $config, 300);
```

**Expected Output:** Heavy data is cached in Redis (shared across servers), while local config is cached in files (server-specific).

**Why:** Multiple stores allow different caching strategies for different data types. Redis for shared, high-performance data; files for server-specific or less critical data.

### Real-World Cases

- **Load-Balanced Applications:** Redis or Memcached for shared cache across servers .
- **Single Server:** File driver is sufficient for small applications.
- **Testing:** Array driver ensures cache isolation between tests.
- **AWS Serverless:** DynamoDB cache for serverless applications .

### References

- Laravel Cache Documentation (master) — https://laravel.com/docs/master/cache 
- Laravel Valkey (Cloud) — https://laravel.com/cloud/docs/resources/caches/valkey 
- Laravel 4.2 Cache Documentation — https://laravel.com/docs/4.2/cache 
- Lumen Cache Documentation — https://lumen.laravel.com/docs/5.1/cache 
- GeneaLabs Laravel Model Caching (DynamoDB vs Redis) — https://packagist.org/packages/mike-bronner/laravel-model-caching 

---

## 2. Structural Storage Patterns: Strings, Models, and Collections

### Definitions

**Core Definition:** Structural storage patterns determine how different data types (strings, Eloquent models, collections, arrays) are serialized and stored in the cache.

**Technical Definition:** Laravel's cache stores serialize all values before storage. This means Eloquent models, collections, arrays, and objects are automatically serialized and deserialized on retrieval. The `serialize` option in some drivers (e.g., `array`) controls whether PHP's `serialize()` function is used. For models, the serialized form includes attributes, relationships, and metadata. Caching models requires careful invalidation to prevent stale data.

**Beginner-Friendly Explanation:** You can cache almost anything in Laravel—strings, arrays, entire Eloquent models, or collections. Laravel automatically converts them to a storable format and back. However, caching models means you need to invalidate the cache when the model changes, or you'll serve stale data.

### Purposes

- To cache complex data structures without manual serialization.
- To store Eloquent models and collections for fast retrieval.
- To reduce database queries for frequently accessed records.
- To cache API responses or computed data structures.

### Syntax Rules and Structure

**Complete General Syntax — Storing Different Types:**

```php
use Illuminate\Support\Facades\Cache;

// Strings
Cache::put('site_name', 'My Application', 3600);

// Arrays
Cache::put('config', ['theme' => 'dark', 'lang' => 'en'], 3600);

// Eloquent models
$user = User::find(1);
Cache::put('user:1', $user, 3600);

// Collections
$users = User::all();
Cache::put('all_users', $users, 3600);

// Collections with relationships
$posts = Post::with('author', 'comments')->get();
Cache::put('posts_with_relations', $posts, 3600);
```

**Complete General Syntax — Retrieving:**

```php
// Retrieve string
$name = Cache::get('site_name');

// Retrieve array
$config = Cache::get('config', []);

// Retrieve model
$user = Cache::get('user:1'); // Returns User instance

// Retrieve collection
$users = Cache::get('all_users'); // Returns Collection of User models
```

**Component Breakdown:**

| Data Type | Serialization | Retrieval |
|-----------|--------------|-----------|
| String | Direct | String |
| Array | PHP serialize | Array |
| Model | PHP serialize | Model instance |
| Collection | PHP serialize | Collection instance |

**Complete General Syntax — Model Caching Package:**

```php
// Using genealabs/laravel-model-caching
use GeneaLabs\LaravelModelCaching\Traits\Cachable;

class Post extends Model
{
    use Cachable;
}

// Automatic caching on query
$posts = Post::where('published', true)->get(); // Cached automatically

// Manual control
$posts = Post::disableCache()->where('published', true)->get();
```

**Syntax Rules:**
- All values are serialized before storage, so any PHP value can be cached .
- Eloquent models are serialized with their attributes and loaded relationships.
- Collections preserve their type and contents.
- Model caching requires invalidation when the model changes (via observers or packages) .
- The `serialize` option can be disabled for performance in some drivers.

**Constraints and Limitations:**
- **Stale models:** Cached models become stale if the database changes without cache invalidation.
- **Relationship loading:** Serialized models include loaded relationships; changes to related models are not reflected .
- **Size limits:** Some drivers (e.g., Memcached) have value size limits (1MB default).
- **Serialization overhead:** Large collections consume memory and serialization time.

### Multiple Annotated Complete Code Examples

**Example 1: Caching an Eloquent Model**

```php
<?php
use App\Models\User;
use Illuminate\Support\Facades\Cache;

// Cache a user model
$user = User::find(1);
Cache::put('user:1', $user, 3600);

// Retrieve the cached model
$cachedUser = Cache::get('user:1');
echo $cachedUser->name; // Works like a normal model
echo get_class($cachedUser); // App\Models\User
```

**Expected Output:** The `User` model is cached and retrieved as a fully functional Eloquent model instance.

**Why:** Laravel serializes the model before storage and unserializes on retrieval. The retrieved model has all attributes and methods .

---

**Example 2: Caching a Collection with Relationships**

```php
<?php
use App\Models\Post;
use Illuminate\Support\Facades\Cache;

// Cache posts with relationships
$posts = Post::with('author', 'comments')->where('published', true)->get();
Cache::put('published_posts', $posts, 3600);

// Retrieve
$cachedPosts = Cache::get('published_posts');
foreach ($cachedPosts as $post) {
    echo $post->author->name; // Relationship is preserved
}
```

**Expected Output:** The collection is cached with its loaded relationships. Iterating and accessing relationships works without additional queries.

**Why:** Serialization preserves the collection's contents, including loaded relationship data. However, changes to related models after caching are not reflected .

---

**Example 3: Model Caching Package with Automatic Invalidation**

```php
<?php
// Using genealabs/laravel-model-caching
namespace App\Models;

use GeneaLabs\LaravelModelCaching\Traits\Cachable;
use Illuminate\Database\Eloquent\Model;

class Post extends Model
{
    use Cachable;
}
```

```php
<?php
// Queries are automatically cached
$posts = Post::where('published', true)->get(); // Cached

// Model updates automatically invalidate the cache
$post = Post::find(1);
$post->title = 'Updated Title';
$post->save(); // Cache for Post queries is flushed
```

**Expected Output:** Queries are cached automatically, and model saves/deletes automatically invalidate the cache for that model.

**Why:** The `Cachable` trait hooks into model events (saved, deleted, restored) to flush the relevant cache entries. Cache tags track which queries include which models .

### Real-World Cases

- **User Profiles:** Cache user models to avoid repeated database queries.
- **Product Catalogues:** Cache collections of products with categories and images.
- **API Responses:** Cache computed API responses as arrays or collections.
- **Configuration Data:** Cache arrays of application settings.

### References

- Laravel Cache Documentation (4.2) — https://laravel.com/docs/4.2/cache 
- GeneaLabs Laravel Model Caching — https://packagist.org/packages/mike-bronner/laravel-model-caching 
- GhostCompiler Laravel Model Caching — https://packagist.org/packages/ghostcompiler/laravel-model-caching 
- AutoCache Package — https://packagist.org/packages/wddyousuf/eloquent-autocache 
- GeneaLabs Model Caching (GitHub) — https://github.com/mike-bronner/laravel-model-caching 

---

## 3. The "Remember" Pattern

### Definitions

**Core Definition:** The "remember" pattern combines cache retrieval and computation in a single operation: if the cached value exists, return it; otherwise, execute the callback, store the result, and return it.

**Technical Definition:** `Cache::remember($key, $ttl, $callback)` checks the cache for the key. If present, it returns the cached value. If absent, it executes the callback, stores the result for the specified TTL (or forever for `rememberForever()`), and returns the result. The `flexible()` method extends this with stale-while-revalidate behaviour, serving stale data while recomputing in the background.

**Beginner-Friendly Explanation:** The `remember()` pattern is the cleanest way to cache data. Instead of writing "check cache, if empty fetch from database, store in cache" every time, you write it once with `remember()`. Laravel handles the logic. `rememberForever()` does the same but without expiration.

### Purposes

- To eliminate repetitive cache-checking code.
- To provide atomic retrieval-or-compute behaviour.
- To simplify caching of expensive computations.
- To support stale-while-revalidate for high-traffic scenarios.

### Syntax Rules and Structure

**Complete General Syntax — `remember()`:**

```php
use Illuminate\Support\Facades\Cache;

$users = Cache::remember('users', $seconds, function () {
    return DB::table('users')->get();
});
```

**Complete General Syntax — `rememberForever()`:**

```php
$users = Cache::rememberForever('users', function () {
    return DB::table('users')->get();
});
```

**Complete General Syntax — `flexible()` (Stale-While-Revalidate):**

```php
$users = Cache::flexible('users', [5, 10], function () {
    return DB::table('users')->get();
});
```

**Component Breakdown:**

| Method | TTL Behaviour | Use Case |
|--------|--------------|----------|
| `remember()` | Expires after TTL | Standard caching |
| `rememberForever()` | Never expires | Permanent configuration |
| `flexible()` | Fresh + stale periods | High-traffic, avoids slow responses |

**Syntax Rules:**
- The callback is only executed on cache miss .
- The callback's return value is automatically stored .
- `rememberForever()` stores until explicitly forgotten.
- `flexible()` accepts an array `[freshSeconds, staleSeconds]` .

**Constraints and Limitations:**
- **Cache stampede:** Multiple concurrent misses can all execute the callback simultaneously.
- **Stale data:** `flexible()` serves stale data during the stale period.
- **Callback execution:** The callback runs on every miss; expensive callbacks increase latency.

### Multiple Annotated Complete Code Examples

**Example 1: Basic `remember()`**

```php
<?php
use Illuminate\Support\Facades\Cache;
use App\Models\User;

$users = Cache::remember('all_users', 3600, function () {
    return User::with('posts')->get();
});

foreach ($users as $user) {
    echo $user->name;
}
```

**Expected Output:** On first call, the database is queried and the result is cached for 1 hour. Subsequent calls return the cached collection without a database query.

**Why:** `remember()` checks the cache, executes the callback on miss, stores the result, and returns it .

---

**Example 2: `rememberForever()` for Configuration**

```php
<?php
$currencies = Cache::rememberForever('accepted_currencies', function () {
    return ['USD', 'EUR', 'GBP', 'JPY'];
});
```

**Expected Output:** The currencies array is cached permanently. It only updates when explicitly forgotten.

**Why:** `rememberForever()` does not set a TTL; the value persists until `forget()` is called .

---

**Example 3: `flexible()` for Stale-While-Revalidate**

```php
<?php
// Fresh for 5 seconds, stale for 10 seconds
$trendingPosts = Cache::flexible('trending_posts', [5, 10], function () {
    return Post::withCount('views')
        ->orderByDesc('views_count')
        ->take(10)
        ->get();
});
```

**Expected Output:**
- Within 5 seconds: returns cached value immediately.
- Between 5-10 seconds: returns stale value, recomputes in background.
- After 10 seconds: recomputes immediately.

**Why:** `flexible()` implements the stale-while-revalidate pattern, preventing slow responses when the cache expires .

### Real-World Cases

- **Dashboard Statistics:** `Cache::remember('stats', 300, fn() => calculateStats())`.
- **Navigation Menus:** `Cache::rememberForever('menu', fn() => buildMenu())`.
- **API Responses:** `Cache::flexible('trending', [30, 60], fn() => getTrending())`.

### References

- Laravel Cache Documentation (master) — https://laravel.com/docs/master/cache 
- Laravel Cache Documentation (4.2) — https://laravel.com/docs/4.2/cache 
- Educative: Laravel Cache Handling — https://www.educative.io/answers/how-to-handle-cache-data-in-laravel 
- Lumen Cache Documentation — https://lumen.laravel.com/docs/5.1/cache 

---

## 4. Atomic Locks: Managing Concurrency

### Definitions

**Core Definition:** Atomic locks provide distributed mutual exclusion, ensuring that only one process can execute a critical section of code at a time, preventing race conditions across multiple servers or workers.

**Technical Definition:** `Cache::lock($name, $seconds)` returns a `Lock` instance. The `get()` method attempts to acquire the lock atomically and returns `true` or `false`. The `block($seconds)` method waits up to the specified seconds for the lock. The `release()` method releases the lock. Locks have an automatic expiry to prevent deadlocks if a process crashes. The `Lock` class also provides a `get()` callback that executes the closure while holding the lock.

**Beginner-Friendly Explanation:** Imagine two users trying to buy the last concert ticket at the same time. Without a lock, both might see "1 ticket available," both buy it, and now you've oversold. A distributed lock ensures only one process can check and decrement the stock at a time. The other process waits or fails gracefully.

### Purposes

- To prevent race conditions in concurrent operations.
- To ensure exclusive access to shared resources.
- To protect inventory, balances, and counters from overselling.
- To manage distributed transactions across multiple servers.
- To prevent duplicate job processing.

### Syntax Rules and Structure

**Complete General Syntax — `Cache::lock()`:**

```php
use Illuminate\Support\Facades\Cache;

$lock = Cache::lock('resource-name', $seconds);

if ($lock->get()) {
    try {
        // Critical section
    } finally {
        $lock->release();
    }
}
```

**Complete General Syntax — `block()` (Wait for Lock):**

```php
$lock = Cache::lock('resource-name', 10);

if ($lock->block(5)) {
    // Lock acquired within 5 seconds
    $lock->release();
} else {
    // Timeout
}
```

**Complete General Syntax — Callback with Lock:**

```php
Cache::lock('resource-name', 10)->get(function () {
    // Lock acquired; automatically released when callback completes
});
```

**Component Breakdown:**

| Method | Behaviour |
|--------|-----------|
| `get()` | Attempts to acquire; returns boolean. |
| `block($seconds)` | Waits up to `$seconds` for acquisition. |
| `release()` | Releases the lock. |
| `get($callback)` | Acquires, executes callback, releases. |

**Syntax Rules:**
- Locks must be released in a `finally` block to prevent deadlocks .
- Lock names should be unique per resource (e.g., `"stock-update-{$productId}"`).
- The second argument to `lock()` is the lock's maximum lifetime in seconds.
- `block()` returns `true` if acquired, `false` if timeout .
- Locks are driver-dependent; Redis and Memcached support distributed locks.

**Constraints and Limitations:**
- **Lock expiry:** If a process crashes without releasing, the lock expires after the specified seconds.
- **Non-atomic with file driver:** File-based locks are not truly distributed.
- **Waiting behaviour:** `block()` waits synchronously; long waits can tie up workers.
- **Deadlock risk:** Failing to release locks in `finally` blocks can cause deadlocks .

### Multiple Annotated Complete Code Examples

**Example 1: Inventory Update with Lock**

```php
<?php
use Illuminate\Support\Facades\Cache;
use App\Models\Product;

class InventoryService
{
    public function decrementStock(int $productId, int $quantity): bool
    {
        $lock = Cache::lock("stock-update-{$productId}", 30);

        if ($lock->get()) {
            try {
                $product = Product::find($productId);

                if ($product->stock < $quantity) {
                    return false; // Insufficient stock
                }

                $product->decrement('stock', $quantity);
                return true;
            } finally {
                $lock->release();
            }
        }

        // Could not acquire lock (another process updating)
        return false;
    }
}
```

**Expected Output:** Only one process can check and decrement the stock at a time. Concurrent requests either wait or return `false`.

**Why:** The lock ensures the check-and-decrement operation is atomic. The `finally` block guarantees release even on exception .

---

**Example 2: Payment Processing with Block**

```php
<?php
use Illuminate\Support\Facades\Cache;
use App\Models\User;

class PaymentProcessor
{
    public function process(int $userId, float $amount): void
    {
        $lock = Cache::lock("payment-{$userId}", 60);

        try {
            if ($lock->block(10)) {
                try {
                    $user = User::find($userId);

                    if ($user->balance < $amount) {
                        throw new \Exception('Insufficient funds');
                    }

                    $user->decrement('balance', $amount);
                    // Process payment...
                } finally {
                    $lock->release();
                }
            } else {
                throw new \Exception('Payment already in progress');
            }
        } catch (\Exception $e) {
            Log::error('Payment failed: ' . $e->getMessage());
            throw $e;
        }
    }
}
```

**Expected Output:** The payment processor waits up to 10 seconds for the lock. If acquired, it processes the payment. If not, it throws an exception.

**Why:** `block(10)` waits up to 10 seconds for the lock, allowing the first request to complete. This prevents duplicate payments .

---

**Example 3: Callback with Automatic Release**

```php
<?php
use Illuminate\Support\Facades\Cache;

$result = Cache::lock('expensive-operation', 30)->get(function () {
    // Perform expensive operation
    return calculateSomethingExpensive();
});

// Lock is automatically released when the callback completes
```

**Expected Output:** The lock is acquired, the callback executes, and the lock is released automatically—even if the callback throws an exception.

**Why:** The `get($callback)` method wraps the callback in a `try/finally` block, ensuring the lock is always released .

### Real-World Cases

- **Inventory Management:** Prevent overselling during concurrent purchases.
- **Financial Transactions:** Ensure balance updates are atomic.
- **Job Processing:** Prevent duplicate job execution across workers.
- **User Registration:** Prevent duplicate account creation from rapid form submissions.

### References

- Preventing Race Conditions with Laravel's Atomic Cache Locks (Laravel News) — https://laravel-news.com/index.php/atomic-cache-locks 
- Issue with Cache Lock for cart items (Laracasts) — https://laracasts.com/discuss/channels/laravel/issue-with-cache-lock-for-cart-items 
- How to Prevent Race Conditions in Laravel with Atomic Locks (Twilio) — https://www.twilio.com/en-us/blog/developers/tutorials/prevent-race-conditions-laravel-atomic-locks 
- Laravel Lock Package — https://packagist.org/packages/zaber-dev/laravel-lock 

---

## 5. Atomic Values: Increment and Decrement

### Definitions

**Core Definition:** Atomic increment and decrement operations modify integer cache values without race conditions, ensuring accurate counters even under concurrent access.

**Technical Definition:** `Cache::increment($key, $amount)` atomically increases the integer value at `$key` by `$amount` (default 1). `Cache::decrement($key, $amount)` atomically decreases it. If the key does not exist, the operation initializes it to the amount (for increment) or zero (for decrement, depending on driver). These operations are atomic at the store level for Redis and Memcached.

**Beginner-Friendly Explanation:** If you have a page view counter and 100 people visit simultaneously, incrementing the counter naively could lose counts. Atomic increment ensures every visit is counted accurately—no matter how many happen at once. Laravel handles the locking internally.

### Purposes

- To maintain accurate counters under concurrent access.
- To implement rate limiting and throttling.
- To track statistics (views, downloads, votes).
- To manage inventory or resource counts atomically.
- To initialize counters safely with `add()`.

### Syntax Rules and Structure

**Complete General Syntax — Increment/Decrement:**

```php
use Illuminate\Support\Facades\Cache;

// Increment by 1
Cache::increment('page_views');

// Increment by custom amount
Cache::increment('page_views', 5);

// Decrement by 1
Cache::decrement('stock');

// Decrement by custom amount
Cache::decrement('stock', 3);
```

**Complete General Syntax — Initialize with `add()`:**

```php
// Initialize counter only if it doesn't exist
Cache::add('page_views', 0, now()->plus(hours: 4));

// Now increment safely
Cache::increment('page_views');
```

**Complete General Syntax — With Expiration:**

```php
// Set a value with TTL, then increment
Cache::put('counter', 0, 3600);
Cache::increment('counter');
```

**Component Breakdown:**

| Method | Behaviour |
|--------|-----------|
| `increment($key, $amount)` | Atomically adds `$amount` (default 1). |
| `decrement($key, $amount)` | Atomically subtracts `$amount` (default 1). |
| `add($key, $value, $ttl)` | Initializes only if key doesn't exist. |

**Syntax Rules:**
- `increment()` and `decrement()` are atomic for Redis and Memcached .
- If the key doesn't exist, `increment()` sets it to the amount (or 1) .
- Use `add()` to safely initialize a counter before incrementing .
- The `increment()` and `decrement()` methods do not work reliably with the database driver for all scenarios .

**Constraints and Limitations:**
- **DynamoDB bug:** The DynamoDB driver's `increment()` does not initialize non-existent keys correctly .
- **Non-integer values:** Incrementing a non-integer value may fail or behave unexpectedly.
- **TTL reset:** Incrementing does not reset the TTL of an existing key.
- **File driver:** File-based increment is not truly atomic across processes.

### Multiple Annotated Complete Code Examples

**Example 1: Page View Counter**

```php
<?php
use Illuminate\Support\Facades\Cache;

// Initialize counter (only if not exists)
Cache::add('post:1:views', 0, now()->plus(days: 7));

// Increment atomically
Cache::increment('post:1:views');

// Get the current count
$views = Cache::get('post:1:views');
```

**Expected Output:** The view count increments atomically, even if multiple requests arrive simultaneously.

**Why:** `add()` initializes the counter to 0. `increment()` atomically adds 1. No race conditions occur .

---

**Example 2: Rate Limiting with Atomic Counters**

```php
<?php
use Illuminate\Support\Facades\Cache;

class RateLimiter
{
    public function attempt(string $key, int $maxAttempts, int $decaySeconds): bool
    {
        $current = Cache::get($key, 0);

        if ($current >= $maxAttempts) {
            return false; // Rate limit exceeded
        }

        // Atomic increment
        Cache::increment($key);

        // Set expiry on first attempt
        if ($current === 0) {
            Cache::put($key, 1, $decaySeconds);
        }

        return true;
    }
}
```

**Expected Output:** The rate limiter accurately counts attempts and denies requests exceeding the limit.

**Why:** Atomic increment prevents race conditions where multiple requests could bypass the limit .

---

**Example 3: Inventory Stock with Atomic Decrement**

```php
<?php
use Illuminate\Support\Facades\Cache;

// Initialize stock
Cache::put('product:1:stock', 100, 3600);

// Decrement atomically
$remaining = Cache::decrement('product:1:stock', 5);

if ($remaining < 0) {
    // Oversold — restore and reject
    Cache::increment('product:1:stock', 5);
    throw new \Exception('Out of stock');
}
```

**Expected Output:** Stock is decremented atomically. Negative values indicate overselling, which can be corrected.

**Why:** `decrement()` atomically reduces the stock value, preventing two processes from both reading "5" and both decrementing to "0" .

### Real-World Cases

- **Analytics:** Page views, downloads, API calls.
- **Rate Limiting:** Login attempts, API request throttling.
- **Inventory:** Stock counts, available seats, remaining tickets.
- **Voting:** Upvotes, likes, reactions.

### References

- Laravel Cache Documentation (master) — https://laravel.com/docs/master/cache 
- Laravel Cache Documentation (5.6 PDF) — https://raw.githubusercontent.com/TIM168/technical_books/master/PHP/laravel框架5_6.pdf 
- DynamoDB increment bug (GitHub Issue) — https://github.com/laravel/framework/issues/46858 
- Cache Store API (Laravel 7.x) — https://api.laravel.com/docs/7.x/Illuminate/Contracts/Cache/Store.html 

---

## References

- Laravel Cache Documentation (master) — https://laravel.com/docs/master/cache 
- Laravel Cache Documentation (4.2) — https://laravel.com/docs/4.2/cache 
- Laravel Valkey (Cloud) — https://laravel.com/cloud/docs/resources/caches/valkey 
- Preventing Race Conditions with Laravel's Atomic Cache Locks (Laravel News) — https://laravel-news.com/index.php/atomic-cache-locks 
- Issue with Cache Lock for cart items (Laracasts) — https://laracasts.com/discuss/channels/laravel/issue-with-cache-lock-for-cart-items 
- How to Prevent Race Conditions in Laravel with Atomic Locks (Twilio) — https://www.twilio.com/en-us/blog/developers/tutorials/prevent-race-conditions-laravel-atomic-locks 
- GeneaLabs Laravel Model Caching — https://packagist.org/packages/mike-bronner/laravel-model-caching 
- GeneaLabs Model Caching (GitHub) — https://github.com/mike-bronner/laravel-model-caching 
- GhostCompiler Laravel Model Caching — https://packagist.org/packages/ghostcompiler/laravel-model-caching 
- AutoCache Package — https://packagist.org/packages/wddyousuf/eloquent-autocache 
- Educative: Laravel Cache Handling — https://www.educative.io/answers/how-to-handle-cache-data-in-laravel 
- Lumen Cache Documentation — https://lumen.laravel.com/docs/5.1/cache 
- Laravel Lock Package — https://packagist.org/packages/zaber-dev/laravel-lock 
- DynamoDB increment bug (GitHub Issue) — https://github.com/laravel/framework/issues/46858 
- Cache Store API (Laravel 7.x) — https://api.laravel.com/docs/7.x/Illuminate/Contracts/Cache/Store.html 
- Laravel Cache Documentation (5.6 PDF) — https://raw.githubusercontent.com/TIM168/technical_books/master/PHP/laravel框架5_6.pdf 