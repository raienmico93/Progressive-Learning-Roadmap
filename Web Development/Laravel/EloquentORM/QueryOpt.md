# Laravel Eloquent Query Optimization: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Query optimization in Laravel Eloquent is the practice of writing database interactions that minimize query count, reduce memory usage, and leverage indexing and caching to achieve optimal application performance.

**Technical Definition**

Query optimization encompasses a set of techniques and patterns—including eager loading, lazy loading strategies, chunking, cursor iteration, database indexing, query/result caching, and query logging—that reduce the number of SQL queries executed, minimize the amount of data transferred, and ensure that database operations scale efficiently with dataset size. The primary performance bottleneck in Laravel applications is typically the database layer, making these techniques critical for production applications.

**Beginner-Friendly Explanation**

Query optimization is about making your database work smarter, not harder. Instead of asking the database for information one piece at a time (which is slow), you learn to ask for everything you need in a few efficient requests. You also learn to handle huge amounts of data without running out of memory, and you add "shortcuts" (indexes and caches) so the database can find things faster.

---

### Key Characteristics

- **N+1 Elimination**: Eager loading reduces query counts from N+1 to a fixed number (typically 2) regardless of collection size.
- **Memory-Conscious Iteration**: Chunking and cursor methods process large datasets in batches, preventing memory exhaustion.
- **Mutation-Safe Processing**: `chunkById()`, `lazyById()`, and their descending variants use ID-based pagination instead of OFFSET, preventing skipped or duplicated records when modifying data during iteration.
- **Relationship-Aware Strategies**: `lazy()` supports eager loading while chunking, whereas `cursor()` cannot eager load relationships.
- **Index-Driven Performance**: Strategic indexing of `WHERE`, `ORDER BY`, `JOIN`, and foreign key columns dramatically accelerates query execution.
- **Layered Caching**: Query results can be cached via `Cache::remember()`, reducing database load for frequently accessed, rarely changing data.
- **Built-in Debugging**: `DB::listen()`, `DB::enableQueryLog()`, `toRawSql()`, and `preventLazyLoading()` provide visibility into query execution and N+1 violations.

---

### Prerequisites

- **PHP 8.1 or higher** (Laravel 10+) or **PHP 8.2** (Laravel 11+).
- **A Laravel application** with Eloquent models and database tables.
- **Basic knowledge of Eloquent** (models, relationships, query builder).
- **Understanding of SQL fundamentals** (SELECT, WHERE, JOIN, indexes).
- **A configured cache driver** (Redis, Memcached, or file) for caching strategies.

---

### Related Programming Areas

- **Eloquent Relationships**: Eager loading directly interacts with relationship definitions.
- **Database Migrations**: Indexes are defined in migrations.
- **Laravel Cache**: Query and result caching integrate with Laravel's cache system.
- **Laravel Telescope / Debugbar**: Query monitoring tools that complement logging and debugging.
- **Pagination**: Cursor pagination is a query optimization technique for large result sets.

---

## Core Concepts

---

### 1. Query Optimization

#### Definitions

**Core Definition**

Query optimization is the process of improving the efficiency of database queries to reduce execution time, resource consumption, and the number of queries executed.

**Technical Definition**

In Laravel, query optimization involves minimizing the number of SQL queries (e.g., via eager loading), reducing the amount of data retrieved (e.g., selecting only needed columns), processing large datasets in memory-safe batches (e.g., chunking, cursors), leveraging database indexes, and caching results. The goal is to achieve O(1) or O(log n) query complexity instead of O(n) where possible.

**Beginner-Friendly Explanation**

Query optimization is about making your database queries faster and lighter. It means asking for less data, asking fewer times, and making sure the database can find what you need quickly.

#### Purposes

- To reduce the number of SQL queries executed per request.
- To minimize memory consumption when handling large datasets.
- To decrease response times and improve user experience.
- To reduce load on the database server.
- To ensure the application scales as data grows.

#### Syntax Rules and Structure

Query optimization in Laravel is not a single syntax but a collection of patterns and methods applied across Eloquent queries, relationship loading, iteration, and caching.

**Constraints and Limitations**

- Optimization techniques must be applied contextually; over-optimization can reduce code readability.
- Caching introduces staleness risks and requires invalidation strategies.
- Indexing slows down write operations.

---

### 2. N+1 Problem

#### Definitions

**Core Definition**

The N+1 query problem occurs when an application executes one query to retrieve a collection of parent models and then executes an additional query for each parent to retrieve a related model, resulting in N+1 total queries.

**Technical Definition**

The N+1 problem is a performance anti-pattern where lazy loading of relationships inside a loop causes one query for the initial collection plus N queries (one per model) for the relationship. This results in O(n) query complexity, which becomes catastrophic as the collection size grows.

**Beginner-Friendly Explanation**

Imagine you have 100 blog posts, and each post has an author. Without optimization, Laravel runs one query to get all 100 posts, and then 100 separate queries—one for each post—to get its author. That's 101 queries total (1 + 100). If you had 1,000 posts, it would be 1,001 queries. This is the N+1 problem.

#### Purposes (of Solving It)

- To reduce query count from N+1 to a fixed, small number.
- To dramatically improve response times for pages that display related data.
- To prevent database overload as datasets grow.
- To reduce memory usage by avoiding redundant model hydration.

#### Syntax Rules and Structure

**The Problem (Bad)**

```php
$posts = Post::all(); // 1 query
foreach ($posts as $post) {
    echo $post->author->name; // N queries (one per post)
}
```

**The Solution (Good)**

```php
$posts = Post::with('author')->get(); // 2 queries total
foreach ($posts as $post) {
    echo $post->author->name;
}
```

**Component Breakdown**

| Pattern | Query Count |
|---------|-------------|
| Lazy loading in loop | 1 + N |
| Eager loading with `with()` | 2 (regardless of N) |

**Constraints and Limitations**

- Eager loading loads all related models, even if only some are needed; use constrained eager loading for large relationships.
- Eager loading multiple relationships can result in more than 2 queries (one per relationship).

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Demonstrating and Fixing N+1**

**Bad Code (N+1)**

```php
// 1 query for posts + N queries for authors
$posts = Post::all();
foreach ($posts as $post) {
    echo $post->author->name;
}
```

**Good Code (Eager Loading)**

```php
// 2 queries total
$posts = Post::with('author')->get();
foreach ($posts as $post) {
    echo $post->author->name;
}
```

**Expected Output**

Both produce the same output, but the good version executes 2 queries instead of N+1.

**Why This Output Occurs**

`with('author')` instructs Eloquent to retrieve all posts and all their authors in two queries, then match them in memory.

---

**Example 2: Detecting N+1 with `preventLazyLoading()`**

**AppServiceProvider**

```php
use Illuminate\Database\Eloquent\Model;

public function boot(): void
{
    Model::preventLazyLoading(! app()->isProduction());
}
```

**Expected Output**

When a relationship is accessed lazily outside production, a `LazyLoadingViolationException` is thrown.

**Why This Output Occurs**

`preventLazyLoading()` makes lazy loading throw an exception in non-production environments, forcing developers to fix N+1 issues before they reach production.

---

#### Real-World Cases with Explanation

**Case 1: Blog Index with Authors**

A blog index displays 50 posts with author names. Without eager loading, this executes 51 queries. With `Post::with('author')->get()`, it executes 2 queries.

**Explanation**: Eager loading eliminates the N+1 problem, making the page load significantly faster.

---

### 3. Eager Loading

#### Definitions

**Core Definition**

Eager loading is the technique of retrieving relationships in advance using the `with()` method, so that related models are loaded in a single additional query rather than one query per model.

**Technical Definition**

Eager loading uses the `with()` method on an Eloquent query to specify relationships that should be loaded alongside the primary models. Eloquent executes a separate query for each eager-loaded relationship (or a single query with joins in some cases) and matches the results to the parent models in memory. Constrained eager loading allows filtering and column selection within the relationship query.

**Beginner-Friendly Explanation**

Eager loading means loading all the related data you need up front, in one or two queries, instead of fetching it piece by piece as you go. It's like grabbing all the ingredients for a recipe at once instead of making multiple trips to the store.

#### Purposes

- To eliminate the N+1 query problem.
- To reduce total query count and improve response times.
- To ensure predictable performance regardless of collection size.
- To allow constrained loading of relationships (filtering, column selection).

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Basic eager loading
$posts = Post::with('author')->get();

// Multiple relationships
$posts = Post::with(['author', 'comments'])->get();

// Constrained eager loading
$posts = Post::with(['comments' => function ($query) {
    $query->where('approved', true)->latest();
}])->get();

// Nested eager loading
$posts = Post::with('author.profile')->get();

// Selective columns (must include foreign key)
$posts = Post::with('author:id,name')->get();
```

**Component Breakdown**

| Method/Syntax | Description |
|---------------|-------------|
| `with('relation')` | Eager loads a relationship. |
| `with(['a', 'b'])` | Eager loads multiple relationships. |
| `with(['rel' => fn($q) => ...])` | Constrained eager loading. |
| `with('rel.column')` | Selective columns (include foreign key). |
| `loadMissing()` | Eager loads relationships that are not already loaded. |

**Syntax Rules**

- Relationship names use camelCase (matching the model method name).
- Constrained eager loading closures receive the relationship query builder.
- When selecting columns, always include the foreign key column, or the relationship will not match.

**Constraints and Limitations**

- Eager loading loads all related records by default; use constraints to limit.
- Deeply nested eager loading can generate complex queries.
- Eager loading does not work with `cursor()`.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Basic and Constrained Eager Loading**

**Code**

```php
// Basic: 2 queries
$posts = Post::with('author')->get();

// Constrained: only approved comments, limited to 3 per post
$posts = Post::with(['comments' => function ($query) {
    $query->where('approved', true)->latest()->limit(3);
}])->get();
```

**Expected Output**

Each post includes its author and up to 3 approved comments, loaded in 3 queries total (posts, authors, comments).

**Why This Output Occurs**

The constrained closure filters the eager-loaded comments query, so only approved comments are retrieved.

---

**Example 2: Selective Columns in Eager Loading**

**Code**

```php
$posts = Post::select('id', 'title', 'user_id')
    ->with(['author:id,name,email'])
    ->get();
```

**Expected Output**

Only the specified columns are retrieved for both posts and authors, reducing data transfer.

**Why This Output Occurs**

Selecting only needed columns reduces memory usage and transfer time. The foreign key (`user_id`) is included to enable relationship matching.

---

#### Real-World Cases with Explanation

**Case 1: E-commerce Order List**

An order list displays orders with their customer names and item counts. Using `with('customer')->withCount('items')` executes 3 queries instead of 1 + N + N.

**Explanation**: Eager loading and `withCount()` together eliminate multiple N+1 problems in a single query.

---

### 4. Lazy Loading

#### Definitions

**Core Definition**

Lazy loading is the default Eloquent behavior where relationship data is retrieved from the database only when the relationship property is accessed for the first time.

**Technical Definition**

Lazy loading defers the execution of relationship queries until the relationship is accessed. While convenient, it is the primary cause of the N+1 problem when relationships are accessed inside loops. Laravel provides `preventLazyLoading()` to detect lazy loading in non-production environments.

**Beginner-Friendly Explanation**

Lazy loading means Laravel waits until you actually use a relationship before loading it. It's convenient, but if you use it inside a loop, it causes the N+1 problem.

#### Purposes

- To defer database queries for relationships that may not always be needed.
- To simplify code when relationship data is accessed infrequently.
- To provide a default behavior that "just works" for simple cases.

#### Syntax Rules and Structure

**Implicit Lazy Loading**

```php
$posts = Post::all();
foreach ($posts as $post) {
    echo $post->author->name; // Lazy loads author for each post
}
```

**Preventing Lazy Loading**

```php
Model::preventLazyLoading(! app()->isProduction());
```

**Constraints and Limitations**

- Lazy loading inside loops causes the N+1 problem.
- Lazy loading is the default and requires conscious effort to avoid in performance-sensitive areas.
- `preventLazyLoading()` throws exceptions in non-production, which can break existing code if not addressed.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Detecting Lazy Loading Violations**

**AppServiceProvider**

```php
public function boot(): void
{
    Model::preventLazyLoading(! app()->isProduction());
}
```

**Expected Output**

```
Illuminate\Database\LazyLoadingViolationException: Attempted to lazy load [author] on model [App\Models\Post] but lazy loading is disabled.
```

**Why This Output Occurs**

`preventLazyLoading()` throws an exception when a relationship is accessed without being eager-loaded, alerting the developer to the N+1 issue.

---

#### Real-World Cases with Explanation

**Case 1: Admin Panel with Optional Details**

An admin panel loads a list of users but only loads their order history when a "View Orders" button is clicked. Lazy loading is appropriate here because the relationship is not always needed.

**Explanation**: Lazy loading is suitable when relationship data is accessed conditionally, not in loops.

---

### 5. Chunking

#### Definitions

**Core Definition**

Chunking is the technique of processing large datasets in small, fixed-size batches (chunks) rather than loading all records into memory at once.

**Technical Definition**

The `chunk()` and `chunkById()` methods retrieve records in batches, passing each batch to a callback. `chunk()` uses OFFSET-based pagination, while `chunkById()` uses `WHERE id > last_id` pagination, which is safe when modifying records during iteration.

**Beginner-Friendly Explanation**

Chunking is like eating a large meal one bite at a time instead of trying to swallow it whole. You process a few records, then the next few, and so on, without filling up memory.

#### Purposes

- To prevent memory exhaustion when processing large datasets.
- To allow safe iteration while modifying records (using `chunkById()`).
- To reduce peak memory usage in batch jobs and exports.
- To enable processing of datasets larger than available RAM.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Basic chunk
User::where('active', true)->chunk(200, function ($users) {
    foreach ($users as $user) {
        // Process user
    }
});

// Mutation-safe chunk
User::where('active', false)->chunkById(200, function ($users) {
    $users->each->delete();
});
```

**Component Breakdown**

| Method | Pagination | Safe for Mutations |
|--------|------------|-------------------|
| `chunk()` | OFFSET | No |
| `chunkById()` | `id > last_id` | Yes |

**Syntax Rules**

- The chunk size is the first argument.
- The callback receives a collection of models.
- `chunkById()` requires an ordered, unique column (default: `id`).

**Constraints and Limitations**

- `chunk()` can skip or duplicate records if the underlying data changes during iteration.
- `chunkById()` requires the ID column to be sortable and unique.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Sending Notifications in Batches**

**Code**

```php
User::where('subscribed', true)->chunk(200, function ($users) {
    foreach ($users as $user) {
        $user->notify(new WeeklyDigest);
    }
});
```

**Expected Output**

Notifications are sent to all subscribed users in batches of 200, with constant memory usage.

**Why This Output Occurs**

Each batch of 200 users is loaded, processed, and released before the next batch is loaded.

---

**Example 2: Mutation-Safe Deletion**

**Code**

```php
User::where('active', false)->chunkById(200, function ($users) {
    $users->each->delete();
});
```

**Expected Output**

All inactive users are deleted without skipping or duplicating records.

**Why This Output Occurs**

`chunkById()` uses `id > last_id` instead of OFFSET, so deletions do not shift the pagination window.

---

#### Real-World Cases with Explanation

**Case 1: Data Export**

A nightly job exports all orders to a CSV file. Using `chunk(500)` keeps memory usage under 50MB instead of loading millions of records at once.

**Explanation**: Chunking is essential for batch processing and exports.

---

### 6. Cursor

#### Definitions

**Core Definition**

The `cursor()` method returns a `LazyCollection` that retrieves records one at a time using a PHP generator, holding only a single Eloquent model in memory at any given moment.

**Technical Definition**

`cursor()` executes a single database query and hydrates each model only as it is iterated over. It uses PHP generators to yield models one by one. Because it holds only one model at a time, it cannot eager load relationships. It is the most memory-efficient iteration method but may still exhaust memory due to PDO buffering for very large result sets.

**Beginner-Friendly Explanation**

Cursor is like reading a book one word at a time instead of photocopying the whole book. You only ever hold one word in your hand, but it's slow if you need to look at multiple words together.

#### Purposes

- To achieve maximum memory efficiency when iterating large datasets.
- To process records one at a time without loading the entire result set.
- To avoid the overhead of model collection hydration.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
foreach (User::where('active', true)->cursor() as $user) {
    // Process one user at a time
}
```

**Component Breakdown**

| Characteristic | Description |
|----------------|-------------|
| Memory usage | One model at a time |
| Query count | One query |
| Eager loading | Not supported |
| Return type | `LazyCollection` |

**Syntax Rules**

- `cursor()` returns a `LazyCollection`.
- LazyCollection methods (e.g., `filter()`, `map()`) can be chained.
- Eager loading cannot be used with `cursor()`.

**Constraints and Limitations**

- Cannot eager load relationships (N+1 risk if relationships are accessed).
- May still exhaust memory due to PDO buffering for very large result sets; `lazy()` is recommended for such cases.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Memory-Efficient Iteration**

**Code**

```php
foreach (User::where('active', true)->cursor() as $user) {
    ProcessUser::dispatch($user->id);
}
```

**Expected Output**

Each user is processed individually with minimal memory overhead.

**Why This Output Occurs**

`cursor()` hydrates one model at a time, keeping memory usage constant regardless of dataset size.

---

#### Real-World Cases with Explanation

**Case 1: Dispatching Jobs for Millions of Users**

A system dispatches a job for each active user. Using `cursor()` avoids loading millions of models into memory at once.

**Explanation**: Cursor is ideal for simple per-record processing where relationships are not needed.

---

### 7. Lazy

#### Definitions

**Core Definition**

The `lazy()` method returns a `LazyCollection` that retrieves records in chunks behind the scenes, but allows iteration as a single flat stream and supports eager loading.

**Technical Definition**

`lazy()` executes the query in chunks (using OFFSET-based pagination internally) and yields models one by one as a flattened `LazyCollection`. Unlike `cursor()`, it supports eager loading. `lazyById()` uses ID-based pagination for mutation-safe iteration.

**Beginner-Friendly Explanation**

Lazy is the best of both worlds: it processes records in small batches (like chunking) but lets you use it like a single continuous list. It also supports loading relationships.

#### Purposes

- To combine the memory efficiency of chunking with the ergonomics of a single stream.
- To support eager loading during chunked iteration.
- To provide mutation-safe iteration via `lazyById()`.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Basic lazy
foreach (Flight::lazy() as $flight) {
    // ...
}

// Lazy with eager loading
foreach (User::with('roles')->lazy() as $user) {
    echo $user->roles->count();
}

// Mutation-safe lazy
User::where('needs_update', true)->lazyById()->each(function ($user) {
    $user->update(['needs_update' => false]);
});
```

**Component Breakdown**

| Method | Pagination | Eager Loading | Mutation-Safe |
|--------|------------|---------------|---------------|
| `lazy()` | OFFSET | Yes | No |
| `lazyById()` | ID-based | Yes | Yes |

**Syntax Rules**

- `lazy()` returns a `LazyCollection`.
- Chunk size can be specified as the first argument (default: 1000).
- `lazyById()` uses `id > last_id` pagination.

**Constraints and Limitations**

- `lazy()` uses OFFSET pagination, which can skip or duplicate records if data changes during iteration.
- `lazyById()` requires a sortable, unique ID column.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Lazy with Eager Loading**

**Code**

```php
foreach (User::with('roles')->lazy() as $user) {
    echo $user->roles->count();
}
```

**Expected Output**

Users and their roles are loaded in chunks, with eager loading preventing N+1 queries.

**Why This Output Occurs**

`lazy()` chunks the query and supports `with()`, so roles are eager-loaded within each chunk.

---

**Example 2: Mutation-Safe Lazy Iteration**

**Code**

```php
User::where('needs_update', true)->lazyById()->each(function ($user) {
    $user->update(['needs_update' => false]);
});
```

**Expected Output**

All matching users are updated without skipped or duplicated records.

**Why This Output Occurs**

`lazyById()` uses ID-based pagination, which is unaffected by the updates being made during iteration.

---

#### Real-World Cases with Explanation

**Case 1: Processing Users with Relationships**

A scheduled job processes users and sends personalized emails based on their roles. Using `with('roles')->lazy()` ensures both memory efficiency and relationship availability.

**Explanation**: Lazy is the recommended choice when relationships are needed during chunked processing.

---

### 8. Indexing

#### Definitions

**Core Definition**

Database indexing is the creation of data structures (indexes) on table columns to accelerate query execution, particularly for `WHERE`, `ORDER BY`, and `JOIN` operations.

**Technical Definition**

An index is a separate data structure (typically a B-tree) that stores column values in a sorted order, enabling the database to locate rows without scanning the entire table. In Laravel, indexes are defined in migrations using `$table->index()`, `$table->unique()`, and foreign key constraints. Composite indexes cover multiple columns, and their order matters.

**Beginner-Friendly Explanation**

An index is like the index at the back of a book. Instead of flipping through every page to find a topic, you look it up in the index and go directly to the right page. Without an index, the database has to scan every row.

#### Purposes

- To dramatically speed up SELECT queries that filter or sort by indexed columns.
- To accelerate JOIN operations on foreign keys.
- To enforce uniqueness constraints.
- To support efficient range queries.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// In a migration
Schema::table('users', function (Blueprint $table) {
    $table->index('email');
    $table->index(['status', 'created_at']); // Composite index
    $table->unique('username');
    $table->foreignId('user_id')->constrained()->index();
});
```

**Component Breakdown**

| Index Type | Use Case |
|------------|----------|
| Single-column | `WHERE` or `ORDER BY` on one column. |
| Composite | Multi-column `WHERE` or `ORDER BY`. |
| Unique | Enforce uniqueness. |
| Foreign key | Accelerate JOINs and enforce referential integrity. |

**Syntax Rules**

- Index columns used in `WHERE`, `ORDER BY`, `JOIN`, and foreign keys.
- For composite indexes, place the most selective column first.
- Do not over-index; every index slows down INSERT, UPDATE, and DELETE.

**Constraints and Limitations**

- Indexes consume disk space.
- Indexes slow down write operations.
- Not all columns benefit from indexing (e.g., low-cardinality columns).

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Adding a Composite Index**

**Migration**

```php
Schema::table('orders', function (Blueprint $table) {
    $table->index(['status', 'created_at']);
});
```

**Expected Output**

Queries filtering by `status` and sorting by `created_at` use the composite index, reducing scan time from seconds to milliseconds.

**Why This Output Occurs**

The composite index allows the database to locate rows by `status` and return them in `created_at` order without a filesort.

---

#### Real-World Cases with Explanation

**Case 1: E-commerce Order Filtering**

An admin panel filters orders by status and sorts by date. A composite index on `(status, created_at)` makes this query fast even with millions of orders.

**Explanation**: Composite indexes match multi-column query patterns.

---

### 9. Query Caching

#### Definitions

**Core Definition**

Query caching is the practice of storing the results of expensive database queries in a cache store (e.g., Redis, Memcached) and serving subsequent requests from the cache instead of hitting the database.

**Technical Definition**

Laravel's cache system provides `Cache::remember($key, $ttl, $callback)` to cache query results. The callback is executed only on a cache miss. Cache tags (supported by Redis and Memcached) allow grouped invalidation. Cache keys should be deterministic and include all parameters that affect the result.

**Beginner-Friendly Explanation**

Query caching is like keeping a copy of a frequently used report on your desk. Instead of regenerating it every time someone asks, you hand them the copy. You only regenerate it when the underlying data changes.

#### Purposes

- To reduce database load for frequently accessed, rarely changing data.
- To improve response times for expensive queries.
- To handle traffic spikes without overwhelming the database.
- To enable sub-millisecond response times for cached results.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
use Illuminate\Support\Facades\Cache;

$popularPosts = Cache::remember('posts:popular', 3600, function () {
    return Post::withCount('comments')
        ->orderByDesc('comments_count')
        ->take(10)
        ->get();
});
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `Cache::remember()` | Retrieve from cache or execute callback and store. |
| `$key` | Unique cache key. |
| `$ttl` | Time-to-live in seconds. |
| `$callback` | Query to execute on cache miss. |

**Syntax Rules**

- Cache keys must be unique and deterministic.
- Include all query parameters in the cache key.
- Use cache tags for grouped invalidation (Redis/Memcached).
- Set appropriate TTLs based on data volatility.

**Constraints and Limitations**

- Cache invalidation is one of the hardest problems in computer science.
- Stale data can be served if TTLs are too long or invalidation is missed.
- Cache stampede (thundering herd) can occur on concurrent misses.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Caching Popular Posts**

**Code**

```php
$popularPosts = Cache::remember('posts:popular', 3600, function () {
    return Post::withCount('comments')
        ->orderByDesc('comments_count')
        ->take(10)
        ->get();
});
```

**Expected Output**

The first request executes the query and stores the result for 1 hour. Subsequent requests within that hour retrieve the result from cache without hitting the database.

**Why This Output Occurs**

`Cache::remember()` checks the cache for the key. If found, it returns the cached value; if not, it executes the callback and stores the result.

---

#### Real-World Cases with Explanation

**Case 1: Homepage "Popular Posts"**

A blog homepage displays the 10 most commented posts. This query is expensive but the result changes infrequently. Caching for 1 hour reduces database load dramatically.

**Explanation**: Caching is ideal for expensive, read-heavy queries on slowly changing data.

---

### 10. Database Caching

#### Definitions

**Core Definition**

Database caching is the use of Laravel's cache system to store query results, reducing the number of direct database interactions.

**Technical Definition**

Database caching is a subset of query caching that specifically targets database query results. It leverages Laravel's cache drivers (file, database, Redis, Memcached) to store serialized query results. Unlike Eloquent's built-in query cache (which does not exist by default), this is a manual caching layer applied by the developer.

**Beginner-Friendly Explanation**

Database caching is simply using Laravel's cache to remember the results of database queries so you don't have to run them again.

#### Purposes

- To reduce database load.
- To speed up repeated queries.
- To scale read-heavy applications.

#### Syntax Rules and Structure

Same as query caching (`Cache::remember()`).

---

### 11. Query Logging

#### Definitions

**Core Definition**

Query logging is the practice of recording executed SQL queries, their bindings, and execution times for debugging and performance analysis.

**Technical Definition**

Laravel provides `DB::listen()` to register a callback that is invoked for every query executed. The callback receives a `QueryExecuted` object with `sql`, `bindings`, and `time` properties. `DB::enableQueryLog()` and `DB::getQueryLog()` provide an alternative mechanism for capturing queries in memory.

**Beginner-Friendly Explanation**

Query logging is like keeping a diary of every database query your application runs. You can look at the diary to see which queries are slow or how many are being executed.

#### Purposes

- To identify slow queries.
- To detect N+1 problems by counting queries.
- To debug unexpected query behavior.
- To monitor query performance in development.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// In AppServiceProvider::boot()
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Log;

DB::listen(function ($query) {
    Log::info($query->sql, [
        'bindings' => $query->bindings,
        'time' => $query->time,
    ]);
});
```

**Component Breakdown**

| Property | Description |
|----------|-------------|
| `$query->sql` | The SQL string with placeholders. |
| `$query->bindings` | The bound values. |
| `$query->time` | Execution time in milliseconds. |

**Syntax Rules**

- Register in a service provider's `boot()` method.
- Use conditionally (e.g., only in local environment) to avoid production overhead.

**Constraints and Limitations**

- Logging every query can generate large log files.
- `DB::listen()` adds slight overhead to every query.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Logging All Queries**

**AppServiceProvider**

```php
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Log;
use Illuminate\Support\Str;

public function boot(): void
{
    if (app()->environment('local')) {
        DB::listen(function ($query) {
            Log::info(
                Str::replaceArray('?', $query->bindings, $query->sql),
                ['time' => $query->time . 'ms']
            );
        });
    }
}
```

**Expected Output (in `laravel.log`)**

```
select * from `users` where `active` = 1 ["time" => "12.5ms"]
```

**Why This Output Occurs**

`DB::listen()` intercepts every query and logs the SQL with bindings replaced.

---

#### Real-World Cases with Explanation

**Case 1: Debugging a Slow Page**

A page takes 5 seconds to load. Query logging reveals 200 queries, 190 of which are N+1 relationship queries. The developer adds eager loading to fix the issue.

**Explanation**: Query logging makes performance problems visible.

---

### 12. Query Debugging

#### Definitions

**Core Definition**

Query debugging is the process of inspecting the SQL generated by Eloquent and analyzing its execution plan to identify performance issues.

**Technical Definition**

Laravel provides `toSql()`, `toRawSql()`, `dd()`, and `ddRawSql()` methods to inspect queries. `toSql()` returns the SQL with placeholders; `toRawSql()` returns the SQL with bindings substituted. Database-level tools like `EXPLAIN` and `EXPLAIN ANALYZE` reveal query execution plans.

**Beginner-Friendly Explanation**

Query debugging is figuring out what SQL your Eloquent code is actually running and why it might be slow. You can print the SQL, see the bindings, and ask the database how it plans to execute the query.

#### Purposes

- To understand what SQL Eloquent generates.
- To identify missing indexes or inefficient query patterns.
- To verify that eager loading is working.
- To debug unexpected query results.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Get SQL with placeholders
$sql = User::where('active', true)->toSql();

// Get SQL with bindings
$sql = User::where('active', true)->toRawSql();

// Dump and die
User::where('active', true)->dd();

// Dump raw SQL
User::where('active', true)->ddRawSql();
```

**Component Breakdown**

| Method | Output |
|--------|--------|
| `toSql()` | SQL with `?` placeholders. |
| `toRawSql()` | SQL with bindings substituted. |
| `dd()` | Dumps query and stops execution. |
| `ddRawSql()` | Dumps raw SQL and stops. |

**Constraints and Limitations**

- `toRawSql()` and `ddRawSql()` require Laravel 10.15+.
- `EXPLAIN` analysis must be performed at the database level.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Inspecting Generated SQL**

**Code**

```php
$query = User::where('status', 'active')->orderBy('created_at');
echo $query->toRawSql();
```

**Expected Output**

```sql
select * from `users` where `status` = 'active' order by `created_at` asc
```

**Why This Output Occurs**

`toRawSql()` substitutes the bindings into the SQL string, producing a fully executable query.

---

#### Real-World Cases with Explanation

**Case 1: Verifying Index Usage**

A developer suspects a query is slow because of a missing index. Using `EXPLAIN` on the raw SQL reveals a full table scan; adding an index fixes the problem.

**Explanation**: Query debugging bridges the gap between Eloquent and the database engine.

---

### 13. Database Performance Best Practices

#### Definitions

**Core Definition**

Database performance best practices are the recommended guidelines for structuring queries, indexes, and caching to achieve optimal database performance in Laravel applications.

**Technical Definition**

Best practices encompass selecting only needed columns, eager loading relationships, preventing lazy loading outside production, using appropriate iteration strategies, indexing strategically, caching expensive queries, and monitoring query performance.

#### Key Best Practices

1. **Eager Load Relationships**: Use `with()` to prevent N+1 queries. Enable `preventLazyLoading(!app()->isProduction())` to catch violations early.

2. **Select Only Needed Columns**: Avoid `SELECT *`. Specify columns explicitly, and always include foreign keys when eager loading.

3. **Use `withCount()` for Counting**: Instead of loading entire relationships to count them, use `withCount()` to add a count subquery.

4. **Choose the Right Iteration Strategy**:
   - `chunk()` for simple batch processing.
   - `chunkById()` when modifying records.
   - `cursor()` for maximum memory efficiency (no relationships).
   - `lazy()` when relationships are needed.
   - `lazyById()` when modifying records with relationships.

5. **Index Strategically**: Index columns used in `WHERE`, `ORDER BY`, `JOIN`, and foreign keys. Use composite indexes for multi-column queries. Avoid over-indexing.

6. **Cache Expensive Queries**: Use `Cache::remember()` for read-heavy, slowly changing data with appropriate TTLs.

7. **Use Cursor Pagination for Large Datasets**: `cursorPaginate()` is more efficient than offset pagination for large result sets.

8. **Avoid `whereHas()` on Large Datasets**: Use `whereIn` with `pluck()` for better index usage.

9. **Log and Monitor Queries**: Use `DB::listen()` in local development and tools like Laravel Telescope or Debugbar to identify slow queries.

10. **Prefer Query Builder for Hot Paths**: For extremely performance-sensitive queries, the query builder or raw SQL may be faster than Eloquent.

#### Real-World Cases with Explanation

**Case 1: Optimizing a Dashboard**

A dashboard displays counts of users, posts, and comments. Using `withCount()` and caching the results for 5 minutes reduces the query count from hundreds to a handful and serves most requests from cache.

**Explanation**: Combining multiple optimization techniques yields the best results.

---

## References

- Prevent N+1 Queries – Dutch Laravel Foundation Best Practices — https://github.com/Dutch-Laravel-Foundation/best-practices/blob/main/database-and-eloquent-orm/prevent-n-plus-one-queries.md
- Use Chunking for Large Datasets – Dutch Laravel Foundation Best Practices — https://github.com/Dutch-Laravel-Foundation/best-practices/blob/main/database-and-eloquent-orm/use-chunking-for-large-datasets.md
- Laravel Database Optimization Reference – turkeryildirim/ai-skills — https://github.com/turkeryildirim/ai-skills/blob/main/laravel/references/database-optimization.md
- Laravel Database Optimization Skill – voku/agent-skills — https://github.com/voku/agent-skills/tree/refs/heads/copilot/update-php-best-practices/skills/laravel-database-optimization
- Eloquent: Getting Started – Laravel 9.x Documentation (Chunking, Cursors) — https://laravel.com/docs/9.x/eloquent
- Optimizing Laravel Eloquent and DB Speed – Laravel Daily — https://laraveldaily.com/post/optimizing-laravel-eloquent-db-speed
- Prevent Lazy Loading – Laravel Strict Mode (Packagist) — https://packagist.org/packages/kainiklas/laravel-strict-mode
- Query Prevent Lazy Loading – voku/agent-skills — https://github.com/voku/agent-skills/blob/main/laravel/rules/query-prevent-lazy-loading.md
- How to Get the SQL Query Formatted by Eloquent – Laracasts — https://laracasts.com/discuss/channels/eloquent/how-can-i-get-the-sql-query-formatted-by-eloquent
- toRawSql Query Debugging – Packagist — https://packagist.org/packages/mouadziani/laravel-query-inspector
- Fixing Queries & Adding Indexes: From 400 Queries to 4 – Laravel News — https://laravel-news.com/fixing-queries-adding-indexes
- Clean Code in Laravel: Database Best Practices – GitHub — https://github.com/ahmadmayahi/clean-code-in-laravel/blob/main/03-model-clean/005-database-best-practices.md
- Laravel Query Caching – Yisu — https://m.yisu.com/zixun/1048632.html
- How to Cache Laravel Eloquent Queries with Redis – GitHub — https://raw.githubusercontent.com/…/cache-remember.md
- Laravel Telescope – Laravel 12.x Documentation — https://laravel.com/docs/12.x/telescope
- Laravel Nightwatch – Cache Monitoring — https://nightwatch.laravel.com

---

**Version Note**: This cheat sheet is based on Laravel 9.x through 12.x documentation. `cursor()` and `lazy()` have been available since Laravel 6. `preventLazyLoading()` was introduced in Laravel 8.15. `toRawSql()` and `ddRawSql()` were added in Laravel 10.15. `chunkById()` and `lazyById()` are available in all recent versions. Developers should always consult the documentation for their specific Laravel version.