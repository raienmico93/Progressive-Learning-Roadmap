# Laravel Relationship Management: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Laravel relationship management encompasses the strategies, methods, and best practices for loading, querying, counting, and synchronizing Eloquent model relationships efficiently while avoiding common performance pitfalls such as the N+1 query problem.

**Technical Definition:** Eloquent provides a suite of relationship management tools including lazy loading (automatic on-demand loading when a relationship property is accessed), eager loading (pre-loading relationships via the `with()` method to reduce query count), lazy eager loading (loading relationships on already-retrieved models via `load()` and `loadMissing()`), relationship existence queries (`has()`, `whereHas()`, `doesntHave()`), relationship counts (`withCount()`, `loadCount()`), and many-to-many synchronization (`attach()`, `detach()`, `sync()`, `syncWithoutDetaching()`). Laravel 8.43+ also includes built-in N+1 query detection via `Model::preventLazyLoading()`.

**Beginner-Friendly Explanation:** When you work with related data in Laravel—like a user's posts or a post's comments—how you load that data matters enormously for performance. Loading relationships one by one in a loop causes hundreds of database queries (the "N+1 problem"). Laravel gives you tools to load them all at once (eager loading), check if relationships exist without loading them, count them efficiently, and manage many-to-many connections through pivot tables.

### Key Characteristics

- **Query Efficiency:** Eager loading reduces N+1 queries by fetching relationships in a single additional query.
- **Lazy Loading vs. Eager Loading:** Lazy loading is convenient but dangerous in loops; eager loading is proactive and performant.
- **Existence Without Loading:** `has()` and `whereHas()` filter records based on relationship existence without hydrating the related models.
- **Aggregate Without Loading:** `withCount()` adds a `{relation}_count` column without loading the full relationship.
- **Pivot Table Management:** `attach()`, `detach()`, `sync()`, and `toggle()` manage many-to-many associations.
- **Strict Mode:** `Model::preventLazyLoading()` throws exceptions when lazy loading occurs in non-production environments.
- **Automatic Eager Loading:** Laravel 12.8+ offers `Model::automaticallyEagerLoadRelationships()` for automatic relation loading.

### Prerequisites

- A working Laravel application with models and relationships defined.
- At least two Eloquent models with a defined relationship (hasOne, hasMany, belongsTo, belongsToMany, etc.).
- Understanding of Eloquent relationship types (covered in a separate cheat sheet).
- For N+1 prevention: Laravel 8.43 or later.
- For automatic eager loading: Laravel 12.8 or later.

### Related Programming Areas

- **Eloquent Relationships:** The foundation—defining the relationships being managed.
- **Query Builder:** Eloquent models extend the query builder; all constraints apply.
- **Database Indexing:** Foreign key columns should be indexed for efficient relationship queries.
- **API Resources:** Eager loading directly impacts API response performance.
- **Testing:** `preventLazyLoading()` is typically disabled during tests.

### Core Concepts / Features

1. Loading Relationships (Dynamic Property vs. Method Access)
2. Lazy Loading (On-Demand Relationship Loading)
3. Eager Loading (`with()`, Nested, Constrained)
4. Preventing N+1 Queries (`preventLazyLoading()`, `loadMissing()`, Automatic Eager Loading)
5. Relationship Existence Queries (`has()`, `whereHas()`, `doesntHave()`)
6. Relationship Counts (`withCount()`, `loadCount()`)
7. Synchronization (`attach()`, `detach()`, `sync()`, `toggle()`)

---

## 1. Loading Relationships

### Definitions

**Core Definition:** Loading relationships refers to the act of retrieving related model data through Eloquent's relationship methods, either as dynamic properties or as query builders.

**Technical Definition:** When a relationship is accessed as a dynamic property (e.g., `$user->posts`), Eloquent executes the relationship query and returns the results (a Model or Collection). When accessed as a method call (e.g., `$user->posts()`), Eloquent returns the underlying `Relation` instance (a query builder), allowing additional constraints before execution.

**Beginner-Friendly Explanation:** You can either grab the related data directly (`$user->posts` gives you all posts) or get a query builder (`$user->posts()` lets you add `->where(...)` before fetching). The first is convenient; the second is more flexible.

### Purposes

- To retrieve related model data from a parent model using an object-oriented interface.
- To apply additional query constraints on related data before retrieval.
- To distinguish between direct data access (property) and query building (method).
- To enable both lazy and eager loading strategies depending on context.
- To provide a consistent API across all relationship types.

### Syntax Rules and Structure

**Complete General Syntax — Dynamic Property:**

```php
$related = $parent->relationshipName;
```

**Complete General Syntax — Method Call:**

```php
$query = $parent->relationshipName();
$related = $parent->relationshipName()->where('active', 1)->get();
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `$parent->relationshipName` | Returns the loaded relation (Model or Collection). Triggers lazy loading if not eager loaded. |
| `$parent->relationshipName()` | Returns the `Relation` query builder instance. |
| `->where(...)->get()` | Applies constraints and executes the query. |

**Syntax Rules:**

- Dynamic property access triggers lazy loading (executes the query immediately).
- Method call access returns the query builder, allowing chaining.
- After method-call chaining, use `->get()`, `->first()`, `->count()`, etc. to execute.
- Once a relationship is loaded via property access, subsequent accesses return the cached result.

**Constraints and Limitations:**

- **Caching:** Property access caches the result on the model. Method access does not.
- **N+1 risk:** Accessing relationships as properties in loops causes N+1 queries.

### Multiple Annotated Complete Code Examples

**Example 1: Dynamic Property Access**

```php
<?php
use App\Models\User;

$user = User::find(1);

// Dynamic property — triggers lazy loading
$posts = $user->posts; // Collection of Post models

foreach ($posts as $post) {
    echo $post->title . "\n";
}
```

**Expected Output:** All post titles for user 1 are printed. A single `SELECT * FROM posts WHERE user_id = 1` query is executed.

**Why:** The `posts` property is resolved via `__get()`, which calls the `posts()` relationship method and executes the query.

---

**Example 2: Method Call with Constraints**

```php
<?php
use App\Models\User;

$user = User::find(1);

// Method call — returns query builder
$activePosts = $user->posts()
    ->where('active', 1)
    ->orderBy('created_at', 'desc')
    ->get();

foreach ($activePosts as $post) {
    echo $post->title;
}
```

**Expected Output:** Only active posts for user 1, ordered by creation date descending.

**Why:** Calling `$user->posts()` returns the `HasMany` relation instance, which extends the query builder. Additional constraints are applied before `get()` executes.

### Real-World Cases

- **User Dashboard:** `$user->posts` for displaying recent posts.
- **Filtered Relationships:** `$user->posts()->where('published', true)->get()` for published posts only.
- **API Endpoints:** `Post::find($id)->comments()->latest()->paginate(10)` for paginated comments.

### References

- Laravel Eloquent: Relationship Methods vs. Dynamic Properties — https://laravel.com/docs/12.x/eloquent-relationships#relationship-methods-vs-dynamic-properties
- Laravel Eloquent: Relationships — https://laravel.com/docs/12.x/eloquent-relationships

---

## 2. Lazy Loading

### Definitions

**Core Definition:** Lazy loading is the automatic, on-demand retrieval of related model data when a relationship is accessed as a dynamic property.

**Technical Definition:** When a relationship is accessed as a property (e.g., `$user->posts`), Eloquent checks if the relationship has already been loaded. If not, it executes the relationship query and caches the result on the model. This behaviour is implemented via the `Model::__get()` method and the `getRelationshipFromMethod()` method.

**Beginner-Friendly Explanation:** Lazy loading means "load it when I ask for it." If you never access `$user->posts`, the posts are never queried. It's convenient but can cause the N+1 problem when used inside loops.

### Purposes

- To provide convenient, on-demand access to related data without explicit loading.
- To avoid loading relationships that are not needed for a given request.
- To simplify code when relationship data is accessed conditionally.
- To serve as the default behaviour when no eager loading is specified.

### Syntax Rules and Structure

```php
$user = User::find(1);
$posts = $user->posts; // Lazy loads posts on first access
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `$user->posts` | Triggers `getRelationshipFromMethod('posts')`. |
| Result | Executes `SELECT * FROM posts WHERE user_id = 1` and caches it. |

**Syntax Rules:**

- Lazy loading occurs automatically when a relationship property is accessed.
- The result is cached on the model instance for subsequent accesses.
- Lazy loading does not occur if the relationship was previously eager loaded.

**Constraints and Limitations:**

- **N+1 problem:** Accessing relationships in a loop causes one query per model.
- **Performance:** Lazy loading is convenient but can be catastrophic for performance in loops.
- **Strict mode:** `Model::preventLazyLoading()` throws `LazyLoadingViolationException` when lazy loading is attempted.

### Multiple Annotated Complete Code Examples

**Example 1: Lazy Loading in a Loop (N+1 Problem)**

```php
<?php
use App\Models\Book;

$books = Book::all(); // 1 query: SELECT * FROM books

foreach ($books as $book) {
    echo $book->author->name; // N queries: SELECT * FROM authors WHERE id = ?
}
```

**Expected Output:** For 100 books, this executes **101 queries**: 1 for the books, plus 100 for each author.

**Why:** Each `$book->author` access triggers a separate query because authors were not eager loaded. This is the classic N+1 query problem.

---

**Example 2: Lazy Loading with Caching**

```php
<?php
$user = User::find(1);

// First access — queries database
$posts1 = $user->posts; // SELECT * FROM posts WHERE user_id = 1

// Second access — returns cached result, no query
$posts2 = $user->posts; // No query executed
```

**Expected Output:** Only one query is executed for the `posts` relationship, even though it was accessed twice.

**Why:** Eloquent caches the loaded relationship on the model instance. Subsequent property accesses return the cached Collection without re-querying.

### Real-World Cases

- **Single Record Detail:** `$post->author->name` for a single blog post view is fine—one additional query.
- **Conditional Loading:** `if ($request->wantsJson()) { $data = $user->posts; }`—only load if needed.
- **Tinker/Prototyping:** Lazy loading is convenient during debugging and prototyping.

### References

- Laravel Eloquent: Lazy Loading — https://laravel.com/docs/12.x/eloquent-relationships#lazy-loading
- Detect and Prevent N+1 Query (Laravel Daily) — https://laraveldaily.com/lesson/laravel-eloquent-expert/detect-and-prevent-n1-query

---

## 3. Eager Loading

### Definitions

**Core Definition:** Eager loading is the proactive retrieval of relationships at the time the parent model is queried, using the `with()` method to execute additional queries upfront and eliminate the N+1 problem.

**Technical Definition:** The `with()` method accepts relationship names (as strings or arrays) and instructs Eloquent to execute a separate query for each relationship immediately after the parent models are retrieved. The results are then mapped onto the parent models, so subsequent property access returns the pre-loaded data without additional queries. Nested relationships are supported via dot notation (`'posts.comments'`), and constraints can be applied via closures.

**Beginner-Friendly Explanation:** Instead of loading related data one-by-one as you access it (lazy loading), eager loading loads everything upfront in just one or two queries. For 100 books, instead of 101 queries, you get 2 queries: one for all books, one for all authors.

### Purposes

- To eliminate the N+1 query problem by loading relationships in a single additional query.
- To improve application performance when related data is known to be needed.
- To support nested relationship loading (`with('posts.comments')`).
- To allow constrained eager loading for filtering related records.
- To enable eager loading of specific columns to reduce memory usage.
- To define default eager loading via the `$with` property on models.

### Syntax Rules and Structure

**Complete General Syntax — Basic Eager Loading:**

```php
$books = Book::with('author')->get();
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Book::with('author')` | Specifies the relationship to eager load. |
| `->get()` | Executes the query and eager loads the relationship. |
| Result | 2 queries: `SELECT * FROM books` + `SELECT * FROM authors WHERE id IN (...)`. |

**Complete General Syntax — Multiple Relationships:**

```php
$books = Book::with(['author', 'publisher'])->get();
```

**Complete General Syntax — Nested Eager Loading:**

```php
$books = Book::with('author.contacts')->get();
$books = Book::with(['author.contacts', 'publisher'])->get();
```

**Complete General Syntax — Constrained Eager Loading:**

```php
$users = User::with(['posts' => function ($query) {
    $query->where('title', 'like', '%code%');
}])->get();
```

**Complete General Syntax — Eager Loading Specific Columns:**

```php
$books = Book::with('author:id,name,book_id')->get();
```

**Complete General Syntax — Default Eager Loading:**

```php
class Book extends Model
{
    protected $with = ['author'];
}
```

**Syntax Rules:**

- `with()` accepts a string or an array of relationship names.
- Nested relationships use dot notation: `'posts.comments'`.
- Constraints are applied via a closure keyed by the relationship name.
- When selecting specific columns, always include the `id` and foreign key columns.
- `$with` on the model defines default eager loading for all queries.
- `without('relation')` removes a relationship from the default `$with` for a single query.
- `withOnly('relation')` overrides all `$with` definitions for a single query.

**Constraints and Limitations:**

- **`limit` and `take`:** The `limit` and `take` query builder methods cannot be used when constraining eager loads.
- **Column requirements:** When specifying columns, the `id` and foreign key columns must be included.
- **MorphTo complexity:** Eager loading `morphTo` relationships runs multiple queries (one per parent type).

### Multiple Annotated Complete Code Examples

**Example 1: Basic Eager Loading**

```php
<?php
use App\Models\Book;

// Eager load the author relationship
$books = Book::with('author')->get();

foreach ($books as $book) {
    echo $book->author->name; // No additional query
}
```

**Expected Output:** 2 queries executed: one for all books, one for all authors referenced by those books. No N+1 problem.

**Why:** `with('author')` instructs Eloquent to execute a second query: `SELECT * FROM authors WHERE id IN (1, 2, 3, ...)`. The results are mapped to each book.

---

**Example 2: Nested Eager Loading**

```php
<?php
use App\Models\Book;

$books = Book::with('author.contacts')->get();

foreach ($books as $book) {
    echo $book->author->name;
    foreach ($book->author->contacts as $contact) {
        echo $contact->email;
    }
}
```

**Expected Output:** 3 queries: books, authors, and contacts. All relationships are pre-loaded.

**Why:** Dot notation loads the `author` relationship and then the `contacts` relationship on each author.

---

**Example 3: Constrained Eager Loading**

```php
<?php
use App\Models\User;

$users = User::with(['posts' => function ($query) {
    $query->where('published', true)
          ->orderBy('created_at', 'desc');
}])->get();

foreach ($users as $user) {
    foreach ($user->posts as $post) {
        echo $post->title;
    }
}
```

**Expected Output:** Only published posts are loaded for each user, ordered by creation date descending.

**Why:** The closure receives the relationship query builder, allowing constraints to be applied to the eager loading query only.

---

**Example 4: Eager Loading with Specific Columns**

```php
<?php
use App\Models\Book;

$books = Book::with('author:id,name,book_id')->get();

foreach ($books as $book) {
    echo $book->author->name; // Only id, name, book_id loaded
    // $book->author->email — not loaded (would be null)
}
```

**Expected Output:** The `authors` table query only selects `id`, `name`, and `book_id` columns.

**Why:** Specifying columns reduces memory usage by avoiding unnecessary column retrieval. The `id` and foreign key (`book_id`) are required for mapping.

### Real-World Cases

- **Blog Index:** `Post::with('author', 'category')->latest()->paginate(10)` for a blog listing page.
- **E-commerce:** `Product::with('images', 'category')->get()` for a product catalogue.
- **API Responses:** `User::with('posts.comments')->find($id)` for a nested API resource.
- **Dashboard:** `Order::with('customer', 'items.product')->get()` for an order management dashboard.

### References

- Laravel Eloquent: Eager Loading — https://laravel.com/docs/12.x/eloquent-relationships#eager-loading
- Laravel Eloquent: Constraining Eager Loads — https://laravel.com/docs/12.x/eloquent-relationships#constraining-eager-loads
- Laravel Eloquent: Eager Loading Specific Columns — https://laravel.com/docs/12.x/eloquent-relationships#eager-loading-specific-columns

---

## 4. Preventing N+1 Queries

### Definitions

**Core Definition:** The N+1 query problem occurs when an application executes one query to retrieve N parent records, then executes N additional queries to retrieve a relationship for each parent. Laravel provides tools to detect and prevent this problem.

**Technical Definition:** Laravel's `Model::preventLazyLoading()` method, introduced in Laravel 8.43, enables strict mode where any attempt to lazy load a relationship throws an `Illuminate\Database\LazyLoadingViolationException`. The `loadMissing()` method provides a safer alternative for conditionally eager loading relationships on already-retrieved models. Laravel 12.8+ introduces `Model::automaticallyEagerLoadRelationships()` for automatic relationship loading.

**Beginner-Friendly Explanation:** The N+1 problem is the most common performance issue in Laravel applications. Laravel gives you tools to catch it during development (strict mode throws exceptions) and fix it (eager loading with `with()` or `loadMissing()`).

### Purposes

- To detect N+1 query problems during development via exception-throwing strict mode.
- To prevent N+1 problems in production by enforcing eager loading.
- To conditionally eager load relationships on already-retrieved models using `loadMissing()`.
- To automate eager loading with `automaticallyEagerLoadRelationships()` (Laravel 12.8+).
- To log lazy loading violations instead of throwing exceptions, for non-breaking monitoring.

### Syntax Rules and Structure

**Complete General Syntax — `preventLazyLoading()`:**

```php
// In AppServiceProvider::boot()
use Illuminate\Database\Eloquent\Model;

Model::preventLazyLoading(! $this->app->isProduction());
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `preventLazyLoading(true)` | Enables strict mode; throws on lazy loading. |
| `preventLazyLoading(false)` | Disables strict mode. |
| `! $this->app->isProduction()` | Only enable in non-production environments. |

**Complete General Syntax — `handleLazyLoadingViolationUsing()`:**

```php
Model::handleLazyLoadingViolationUsing(function (Model $model, string $relation) {
    info("Lazy loaded [{$relation}] on " . get_class($model));
});
```

**Complete General Syntax — `loadMissing()`:**

```php
$book->loadMissing('author');
$books->loadMissing(['author', 'publisher']);
$book->loadMissing(['comments' => function ($query) {
    $query->latest()->take(5);
}]);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `loadMissing('relation')` | Eager loads only if not already loaded. |
| Array form | Loads multiple relationships. |
| Closure form | Applies constraints to the eager load. |

**Complete General Syntax — Automatic Eager Loading (Laravel 12.8+):**

```php
// In AppServiceProvider::boot()
Model::automaticallyEagerLoadRelationships();
```

**Syntax Rules:**

- `preventLazyLoading()` should be called in `AppServiceProvider::boot()`.
- Typically enabled only in non-production environments.
- `handleLazyLoadingViolationUsing()` customises the violation response (e.g., log instead of throw).
- `loadMissing()` loads a relationship only if it hasn't been loaded already.
- `automaticallyEagerLoadRelationships()` is a beta feature in Laravel 12.8+.

**Constraints and Limitations:**

- **Production safety:** Never enable `preventLazyLoading()` in production without a fallback handler—it will break the application.
- **`loadMissing()` pivot fields:** If a relationship was already loaded without certain pivot fields, `loadMissing()` will not re-query to add them.
- **Automatic loading:** The `automaticallyEagerLoadRelationships()` feature does not work with `BelongsToMany` in all cases.
- **Performance:** Automatic eager loading may load more data than necessary.

### Multiple Annotated Complete Code Examples

**Example 1: Enabling Strict Mode**

```php
<?php
// app/Providers/AppServiceProvider.php

namespace App\Providers;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        // Enable strict lazy loading prevention outside production
        Model::preventLazyLoading(! $this->app->isProduction());
    }
}
```

```php
// This will throw LazyLoadingViolationException in development
$books = Book::all();
foreach ($books as $book) {
    echo $book->author->name; // Exception thrown!
}
```

**Expected Output:** In development, a `LazyLoadingViolationException` is thrown when `$book->author` is accessed without eager loading. In production, the code works normally.

**Why:** `preventLazyLoading(true)` instructs Eloquent to throw an exception whenever a relationship is lazy loaded, forcing developers to fix N+1 issues before deployment.

---

**Example 2: Logging Instead of Throwing**

```php
<?php
// app/Providers/AppServiceProvider.php

use Illuminate\Database\Eloquent\Model;

public function boot(): void
{
    Model::preventLazyLoading(! $this->app->isProduction());

    // Log violations instead of throwing exceptions
    Model::handleLazyLoadingViolationUsing(function (Model $model, string $relation) {
        $class = $model::class;
        info("Attempted to lazy load [{$relation}] on model [{$class}].");
    });
}
```

**Expected Output:** Lazy loading attempts are logged to `laravel.log` instead of throwing exceptions. The application continues to function.

**Why:** `handleLazyLoadingViolationUsing()` replaces the default exception-throwing behaviour with a custom callback, allowing monitoring without breaking the application.

---

**Example 3: Using `loadMissing()` for Conditional Loading**

```php
<?php
use App\Models\Dashboard;

class DashboardController extends Controller
{
    public function show(Request $request, Dashboard $dashboard)
    {
        // Always load base relationships
        $dashboard->loadMissing(['widgets', 'owner']);

        // Conditionally load additional data
        if ($request->section === 'analytics') {
            $dashboard->loadMissing([
                'widgets.viewHistory' => function ($query) {
                    $query->whereBetween('viewed_at', [
                        now()->subDays(30), now()
                    ]);
                },
                'widgets.interactions'
            ]);
        }

        return $dashboard;
    }
}
```

**Expected Output:** Base relationships are always loaded. Analytics relationships are loaded only when the `section` query parameter equals `analytics`.

**Why:** `loadMissing()` intelligently loads only the relationships that have not already been loaded, preventing duplicate queries while allowing conditional loading.

### Real-World Cases

- **Development Strict Mode:** All Laravel applications should enable `preventLazyLoading()` in local/testing environments.
- **API Conditional Includes:** `loadMissing()` for APIs where clients can request optional relationship data via query parameters.
- **Dashboard Optimisation:** Loading different relationship sets based on which dashboard widgets are visible.
- **Production Monitoring:** Logging lazy loading violations to identify performance bottlenecks without breaking the application.

### References

- Laravel Eloquent: Preventing Lazy Loading — https://laravel.com/docs/12.x/eloquent-relationships#preventing-lazy-loading
- Laravel Eloquent: Lazy Eager Loading — https://laravel.com/docs/12.x/eloquent-relationships#lazy-eager-loading
- Dynamic Relationship Loading in Laravel (Laravel News) — https://laravel-news.com/index.php/loadMissing
- Automatic Relation Loading in Laravel 12.8 (Laravel News) — https://laravel-news.com/automatic-relation-loading-laravel-12-8
- Detect and Prevent N+1 Query (Laravel Daily) — https://laraveldaily.com/lesson/laravel-eloquent-expert/detect-and-prevent-n1-query

---

## 5. Relationship Existence Queries

### Definitions

**Core Definition:** Relationship existence queries allow you to filter parent models based on whether they have (or do not have) related records, without actually loading those related records.

**Technical Definition:** The `has()` method adds a `WHERE EXISTS` subquery to the parent query, filtering records that have at least one related record. `whereHas()` extends this with additional constraints on the related query. `doesntHave()` and `whereDoesntHave()` perform the inverse—filtering records that do **not** have related records. All methods support dot notation for nested relationships and polymorphic variants (`whereHasMorph`, `whereDoesntHaveMorph`).

**Beginner-Friendly Explanation:** Instead of loading all posts and then checking which ones have comments, you can ask the database directly: "Give me only posts that have at least one comment." This is more efficient because the related records are never loaded into memory.

### Purposes

- To filter parent records based on the existence of related records without loading them.
- To apply constraints on the related records during existence filtering.
- To query relationship absence (records without related records).
- To support nested relationship existence (`has('comments.author')`).
- To filter polymorphic relationships using `whereHasMorph()`.
- To count related records via `has('comments', '>=', 5)`.

### Syntax Rules and Structure

**Complete General Syntax — `has()`:**

```php
$posts = Post::has('comments')->get();
$posts = Post::has('comments', '>=', 3)->get();
$posts = Post::has('comments', '>=', 3)->orHas('votes')->get();
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `has('comments')` | Filters posts that have at least one comment. |
| `has('comments', '>=', 3)` | Filters posts with 3 or more comments. |
| `orHas('votes')` | OR condition for relationship existence. |

**Complete General Syntax — `whereHas()`:**

```php
$posts = Post::whereHas('comments', function (Builder $query) {
    $query->where('content', 'like', 'code%');
})->get();
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `whereHas('comments', $closure)` | Filters posts that have at least one comment matching the closure. |
| `$query` | The relationship query builder for adding constraints. |

**Complete General Syntax — `doesntHave()` / `whereDoesntHave()`:**

```php
$posts = Post::doesntHave('comments')->get();
$posts = Post::whereDoesntHave('comments', function (Builder $query) {
    $query->where('content', 'like', 'foo%');
})->get();
```

**Complete General Syntax — Polymorphic:**

```php
$comments = Comment::whereHasMorph('commentable', [Post::class, Video::class], function (Builder $query) {
    $query->where('title', 'like', 'foo%');
})->get();

$comments = Comment::whereDoesntHaveMorph('commentable', Post::class, function (Builder $query) {
    $query->where('title', 'like', 'foo%');
})->get();
```

**Syntax Rules:**

- `has()` and `whereHas()` add a `WHERE EXISTS` subquery to the parent query.
- `whereHas()` accepts a closure that receives the relationship query builder.
- Nested relationships use dot notation: `has('comments.author')`.
- `doesntHave()` and `whereDoesntHave()` add `WHERE NOT EXISTS`.
- `whereHasMorph()` accepts an array of model classes or `'*'` as a wildcard.
- The `$type` parameter in the closure allows different constraints per morph type.

**Constraints and Limitations:**

- **Cross-database:** Eloquent does not support relationship existence queries across different databases.
- **Performance:** `whereHas()` uses correlated subqueries, which can be slower than joins on very large tables.
- **`limit` and `take`:** Cannot be used within `whereHas()` closures.

### Multiple Annotated Complete Code Examples

**Example 1: Basic `has()` and `whereHas()`**

```php
<?php
use App\Models\Post;
use Illuminate\Database\Eloquent\Builder;

// Posts with at least one comment
$posts = Post::has('comments')->get();

// Posts with at least 10 comments
$posts = Post::has('comments', '>=', 10)->get();

// Posts with at least one comment containing 'code'
$posts = Post::whereHas('comments', function (Builder $query) {
    $query->where('content', 'like', '%code%');
})->get();
```

**Expected Output:** The first query returns all posts with one or more comments. The second returns posts with 10+ comments. The third returns posts with at least one comment containing "code".

**Why:** `has()` adds a `WHERE EXISTS (SELECT * FROM comments WHERE comments.post_id = posts.id)` subquery. `whereHas()` extends this with additional constraints in the subquery.

---

**Example 2: `doesntHave()` for Absence**

```php
<?php
use App\Models\Post;

// Posts with no comments
$posts = Post::doesntHave('comments')->get();

// Posts without comments containing 'spam'
$posts = Post::whereDoesntHave('comments', function ($query) {
    $query->where('content', 'like', '%spam%');
})->get();
```

**Expected Output:** The first query returns posts that have zero comments. The second returns posts where no comment contains "spam".

**Why:** `doesntHave()` adds `WHERE NOT EXISTS`. `whereDoesntHave()` adds `WHERE NOT EXISTS` with additional constraints.

---

**Example 3: Nested and Polymorphic Existence**

```php
<?php
use App\Models\Post;
use App\Models\Comment;
use Illuminate\Database\Eloquent\Builder;

// Posts with comments from non-banned authors
$posts = Post::whereHas('comments.author', function (Builder $query) {
    $query->where('banned', 0);
})->get();

// Comments on posts or videos with title like 'foo%'
$comments = Comment::whereHasMorph('commentable', [Post::class, Video::class], function (Builder $query, $type) {
    $query->where('title', 'like', 'foo%');

    if ($type === Post::class) {
        $query->orWhere('content', 'like', 'foo%');
    }
})->get();
```

**Expected Output:** The first query returns posts that have at least one comment whose author is not banned. The second returns comments on posts or videos with matching titles.

**Why:** Dot notation (`comments.author`) traverses nested relationships. `whereHasMorph()` handles polymorphic relationships and allows type-specific constraints.

### Real-World Cases

- **Blog Filtering:** `Post::has('comments')->get()` for a "most discussed" section.
- **User Engagement:** `User::whereHas('orders', fn($q) => $q->where('total', '>', 1000))->get()` for VIP customers.
- **Content Moderation:** `Comment::whereDoesntHaveMorph('commentable', Post::class)->get()` for orphaned comments.
- **Inventory:** `Product::doesntHave('orders')->get()` for products never ordered.

### References

- Laravel Eloquent: Querying Relationship Existence — https://laravel.com/docs/12.x/eloquent-relationships#querying-relationship-existence
- Laravel Eloquent: Querying Relationship Absence — https://laravel.com/docs/12.x/eloquent-relationships#querying-relationship-absence
- Laravel Eloquent: Querying Morph To Relationships — https://laravel.com/docs/12.x/eloquent-relationships#querying-morph-to-relationships
- Laravel API: QueriesRelationships Trait — https://api.laravel.com/docs/12.x//Illuminate/Database/Eloquent/Concerns/QueriesRelationships.html

---

## 6. Relationship Counts

### Definitions

**Core Definition:** Relationship counts allow you to retrieve the number of related records for each parent model without loading the related records themselves.

**Technical Definition:** The `withCount()` method adds a `{relation}_count` column to the parent query using a correlated subquery (`SELECT COUNT(*) FROM related WHERE ...`). The count is accessible as an attribute on each model. `withCount()` supports multiple relationships, constrained counts via closures, and aliasing for multiple counts on the same relationship. The `loadCount()` method provides the same functionality for already-retrieved models.

**Beginner-Friendly Explanation:** Instead of loading all of a user's 500 posts just to count them, `withCount('posts')` asks the database to count them and adds a `posts_count` attribute to the user model. This is far more efficient.

### Purposes

- To retrieve the number of related records without loading them into memory.
- To avoid memory overhead when only the count is needed.
- To add count constraints to the parent query (e.g., users with 5+ posts).
- To alias counts for multiple conditions on the same relationship.
- To load counts on already-retrieved models using `loadCount()`.
- To combine counts with eager loading (`with()` + `withCount()`).

### Syntax Rules and Structure

**Complete General Syntax — `withCount()`:**

```php
$posts = Post::withCount('comments')->get();
foreach ($posts as $post) {
    echo $post->comments_count;
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `withCount('comments')` | Adds a `comments_count` attribute via subquery. |
| `$post->comments_count` | The count of related comments. |

**Complete General Syntax — Multiple Counts:**

```php
$posts = Post::withCount(['votes', 'comments'])->get();
echo $posts[0]->votes_count;
echo $posts[0]->comments_count;
```

**Complete General Syntax — Constrained Counts:**

```php
$posts = Post::withCount(['comments' => function (Builder $query) {
    $query->where('approved', true);
}])->get();
```

**Complete General Syntax — Aliased Counts:**

```php
$posts = Post::withCount([
    'comments',
    'comments as pending_comments_count' => function (Builder $query) {
        $query->where('approved', false);
    }
])->get();
```

**Complete General Syntax — `loadCount()`:**

```php
$post = Post::find(1);
$post->loadCount('comments');
echo $post->comments_count;
```

**Syntax Rules:**

- `withCount()` must be called **before** `get()` or `paginate()`.
- If combining with `select()`, call `withCount()` **after** `select()`.
- The count attribute is named `{relation}_count` in snake_case.
- Aliases allow multiple counts on the same relationship.
- `loadCount()` works on already-retrieved models.
- Count constraints can be applied in the closure.

**Constraints and Limitations:**

- **`withCount()` only works with eager loading**, not lazy loading. For lazy counts, use `loadCount()`.
- **Multiple counts on same relation:** Use aliases to avoid naming conflicts.
- **Performance:** Counts use correlated subqueries; for very large datasets, consider denormalised counter caches.

### Multiple Annotated Complete Code Examples

**Example 1: Basic `withCount()`**

```php
<?php
use App\Models\Post;

$posts = Post::withCount('comments')->get();

foreach ($posts as $post) {
    echo "{$post->title}: {$post->comments_count} comments\n";
}
```

**Expected Output:** Each post's title and comment count is printed. 2 queries executed: one for posts, one for the count subquery.

**Why:** `withCount('comments')` adds a `SELECT COUNT(*) FROM comments WHERE comments.post_id = posts.id AS comments_count` subquery to the main posts query.

---

**Example 2: Constrained and Aliased Counts**

```php
<?php
use App\Models\Post;
use Illuminate\Database\Eloquent\Builder;

$posts = Post::withCount([
    'comments',
    'comments as pending_count' => function (Builder $query) {
        $query->where('approved', false);
    },
    'comments as approved_count' => function (Builder $query) {
        $query->where('approved', true);
    }
])->get();

foreach ($posts as $post) {
    echo "Total: {$post->comments_count}, ";
    echo "Pending: {$post->pending_count}, ";
    echo "Approved: {$post->approved_count}\n";
}
```

**Expected Output:** Each post shows total, pending, and approved comment counts.

**Why:** Aliases (`pending_count`, `approved_count`) allow multiple counts on the same relationship with different constraints.

---

**Example 3: `loadCount()` on Retrieved Models**

```php
<?php
use App\Models\Post;

$post = Post::find(1);
$post->loadCount('comments');

echo $post->comments_count;
```

**Expected Output:** The comment count for post 1 is printed.

**Why:** `loadCount()` adds the count to an already-retrieved model without re-querying the entire model.

### Real-World Cases

- **Blog Index:** `Post::withCount('comments')->latest()->paginate(10)` for displaying comment counts.
- **User Dashboard:** `User::withCount(['posts', 'followers'])->find($id)` for profile statistics.
- **E-commerce:** `Product::withCount('reviews')->get()` for review counts on product listings.
- **Analytics:** `Category::withCount('products')->get()` for category product counts.

### References

- Laravel Eloquent: Counting Related Models — https://laravel.com/docs/12.x/eloquent-relationships#counting-related-models
- Laravel API: QueriesRelationships::withCount() — https://api.laravel.com/docs/12.x//Illuminate/Database/Eloquent/Concerns/QueriesRelationships.html#method_withCount

---

## 7. Synchronization

### Definitions

**Core Definition:** Synchronization refers to the methods used to manage many-to-many relationship records in the pivot (intermediate) table, including attaching, detaching, syncing, and toggling related model associations.

**Technical Definition:** The `BelongsToMany` relationship class provides `attach()`, `detach()`, `sync()`, `syncWithoutDetaching()`, `toggle()`, and `updateExistingPivot()` methods via the `InteractsWithPivotTable` trait. These methods generate INSERT, DELETE, and UPDATE statements on the pivot table. `sync()` accepts an array of IDs and performs a diff: it detaches IDs not in the array and attaches IDs that are in the array but not currently attached. `attach()` accepts additional pivot column values as a second argument.

**Beginner-Friendly Explanation:** In a many-to-many relationship (like users and roles), you need to manage which users have which roles. `attach()` adds a role, `detach()` removes one, and `sync()` makes the user's roles exactly match a given list—adding missing ones and removing extras.

### Purposes

- To add new associations to a many-to-many relationship using `attach()`.
- To remove existing associations using `detach()`.
- To synchronise associations to match an exact list using `sync()`.
- To add associations without removing existing ones using `syncWithoutDetaching()`.
- To toggle associations (add if missing, remove if present) using `toggle()`.
- To update pivot table columns without removing the association using `updateExistingPivot()`.

### Syntax Rules and Structure

**Complete General Syntax — `attach()`:**

```php
$user->roles()->attach($roleId);
$user->roles()->attach($roleId, ['expires' => $expires]);
$user->roles()->attach([1, 2, 3]);
$user->roles()->attach([1 => ['expires' => true], 2, 3]);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `attach($id)` | Inserts a row into the pivot table. |
| `attach($id, $attributes)` | Inserts with additional pivot column values. |
| `attach([...])` | Attaches multiple IDs at once. |
| `attach([id => attributes])` | Attaches with per-ID pivot values. |

**Complete General Syntax — `detach()`:**

```php
$user->roles()->detach($roleId);
$user->roles()->detach([1, 2, 3]);
$user->roles()->detach(); // Detach all
```

**Complete General Syntax — `sync()`:**

```php
$user->roles()->sync([1, 2, 3]);
$user->roles()->sync([1 => ['expires' => true], 2, 3]);
$user->roles()->sync([1, 2, 3], false); // Don't detach
$user->roles()->syncWithoutDetaching([1, 2, 3]);
```

**Complete General Syntax — `toggle()`:**

```php
$user->roles()->toggle([1, 2, 3]);
```

**Complete General Syntax — `updateExistingPivot()`:**

```php
$user->roles()->updateExistingPivot($roleId, ['active' => true]);
```

**Syntax Rules:**

- `attach()` accepts a single ID, an array of IDs, or IDs with pivot attribute arrays.
- `detach()` accepts an ID, an array of IDs, or no arguments (detaches all).
- `sync()` accepts an array of IDs; IDs not in the array are detached, IDs in the array not currently attached are attached.
- `syncWithoutDetaching()` is equivalent to `sync($ids, false)`—adds without removing.
- `toggle()` adds IDs that are not attached and removes IDs that are attached.
- `updateExistingPivot()` updates pivot columns for an existing association.

**Constraints and Limitations:**

- **Events:** Pivot table operations (`attach`, `detach`, `sync`) do not dispatch model events by default. Use `plank/laravel-pivot-events` package for pivot events.
- **Custom pivot models:** When using `using(CustomPivot::class)`, `sync()` and related methods respect the custom pivot model.
- **`sync()` diff:** `sync()` performs a diff and only executes the necessary INSERT and DELETE statements.
- **Detaching all:** `detach()` with no arguments removes all pivot rows for the parent model.

### Multiple Annotated Complete Code Examples

**Example 1: `attach()` and `detach()`**

```php
<?php
use App\Models\User;
use App\Models\Role;

$user = User::find(1);
$adminRole = Role::where('name', 'admin')->first();

// Attach a role
$user->roles()->attach($adminRole->id);
// INSERT INTO role_user (user_id, role_id) VALUES (1, 1)

// Attach with pivot data
$user->roles()->attach($adminRole->id, ['expires' => now()->addYear()]);

// Attach multiple roles
$user->roles()->attach([1, 2, 3]);

// Detach a role
$user->roles()->detach($adminRole->id);
// DELETE FROM role_user WHERE user_id = 1 AND role_id = 1

// Detach all roles
$user->roles()->detach();
```

**Expected Output:** `attach()` inserts rows into the `role_user` pivot table. `detach()` removes them.

**Why:** `attach()` generates an INSERT statement; `detach()` generates a DELETE statement.

---

**Example 2: `sync()` and `syncWithoutDetaching()`**

```php
<?php
use App\Models\User;

$user = User::find(1);
// Assume current roles: [1, 2, 3]

// Sync to exact list — detaches 3, keeps 1 and 2
$user->roles()->sync([1, 2]);
// DELETE FROM role_user WHERE user_id = 1 AND role_id NOT IN (1, 2)
// (No INSERT needed since 1 and 2 already attached)

// Sync to add and remove
$user->roles()->sync([2, 3, 4]);
// DELETE: role_id = 1 (not in list)
// INSERT: role_id = 4 (new)

// Add without detaching
$user->roles()->syncWithoutDetaching([5]);
// INSERT: role_id = 5 (existing roles remain)

// With pivot values
$user->roles()->sync([1 => ['expires' => true], 2, 3]);
```

**Expected Output:** `sync()` makes the pivot table match the given list exactly. `syncWithoutDetaching()` adds new associations without removing existing ones.

**Why:** `sync()` computes the difference between the current and desired states, executing only the necessary INSERT and DELETE statements.

---

**Example 3: `toggle()` and `updateExistingPivot()`**

```php
<?php
use App\Models\User;

$user = User::find(1);

// Toggle roles: add if missing, remove if present
$user->roles()->toggle([1, 2, 3]);
// If role 1 was attached: DELETE. If role 2 was not: INSERT.

// Update pivot data without removing the association
$user->roles()->updateExistingPivot(1, ['active' => false]);
// UPDATE role_user SET active = false WHERE user_id = 1 AND role_id = 1
```

**Expected Output:** `toggle()` adds or removes associations based on their current state. `updateExistingPivot()` modifies pivot columns without changing the association.

**Why:** `toggle()` checks each ID and performs the opposite operation. `updateExistingPivot()` updates the pivot row directly.

### Real-World Cases

- **User Role Management:** `$user->roles()->sync($request->role_ids)` for updating user roles from a form.
- **Tagging System:** `$post->tags()->sync($tagIds)` for syncing post tags.
- **Subscription Management:** `$user->subscriptions()->attach($planId, ['expires_at' => now()->addMonth()])`.
- **Permission Assignment:** `$role->permissions()->sync([1, 2, 3])` for role-permission management.

### References

- Laravel Eloquent: Many To Many Relationships — https://laravel.com/docs/12.x/eloquent-relationships#many-to-many
- Laravel API: InteractsWithPivotTable Trait — https://api.laravel.com/docs/7.x/Illuminate/Database/Eloquent/Relations/Concerns/InteractsWithPivotTable.html
- Laravel Eloquent: Syncing Associations — https://laravel.com/docs/12.x/eloquent-relationships#syncing-associations

---

## References

- Laravel Eloquent: Relationships — https://laravel.com/docs/12.x/eloquent-relationships
- Laravel Eloquent: Eager Loading — https://laravel.com/docs/12.x/eloquent-relationships#eager-loading
- Laravel Eloquent: Lazy Loading — https://laravel.com/docs/12.x/eloquent-relationships#lazy-loading
- Laravel Eloquent: Preventing Lazy Loading — https://laravel.com/docs/12.x/eloquent-relationships#preventing-lazy-loading
- Laravel Eloquent: Querying Relationship Existence — https://laravel.com/docs/12.x/eloquent-relationships#querying-relationship-existence
- Laravel Eloquent: Counting Related Models — https://laravel.com/docs/12.x/eloquent-relationships#counting-related-models
- Laravel Eloquent: Syncing Associations — https://laravel.com/docs/12.x/eloquent-relationships#syncing-associations
- Laravel API: QueriesRelationships Trait — https://api.laravel.com/docs/12.x//Illuminate/Database/Eloquent/Concerns/QueriesRelationships.html
- Laravel API: InteractsWithPivotTable Trait — https://api.laravel.com/docs/7.x/Illuminate/Database/Eloquent/Relations/Concerns/InteractsWithPivotTable.html
- Dynamic Relationship Loading in Laravel (Laravel News) — https://laravel-news.com/index.php/loadMissing
- Automatic Relation Loading in Laravel 12.8 (Laravel News) — https://laravel-news.com/automatic-relation-loading-laravel-12-8
- Detect and Prevent N+1 Query (Laravel Daily) — https://laraveldaily.com/lesson/laravel-eloquent-expert/detect-and-prevent-n1-query
- New Eloquent Relation Existence Methods in Laravel 11.37 (Laravel News) — https://laravel-news.com/new-eloquent-relation-existence-methods
- Laravel Eloquent: Relationships (Laravel 10.x) — https://laravel.com/docs/10.x/eloquent-relationships
- Laravel Eloquent: Relationships (Laravel 7.x) — https://laravel.com/docs/7.x/eloquent-relationships