# Laravel Eloquent Advanced Features: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Laravel Eloquent's advanced features encompass the mechanisms that extend the fundamental Active Record pattern, enabling sophisticated attribute transformation, reusable query composition, event-driven lifecycle management, and powerful data manipulation through collections.

**Technical Definition:** These features are implemented through several traits and classes within the `Illuminate\Database\Eloquent` namespace. Accessors and mutators (since Laravel 9) are unified under the `Attribute` class via the `HasAttributes` trait. Attribute casting is handled by the `HasAttributes::casts()` method and the `CastsAttributes` contract. Query scopes (local) are implemented as `scope*`-prefixed methods or `#[Scope]`-attributed methods, while global scopes implement the `Illuminate\Database\Eloquent\Scope` interface. Model events are dispatched by the `HasEvents` trait at each persistence stage. Observers group event listeners into dedicated classes registered via the `ObservedBy` attribute or the `observe()` method. Eloquent Collections extend the base `Illuminate\Support\Collection` and add model-specific methods.

**Beginner-Friendly Explanation:** Basic Eloquent lets you CRUD records. Advanced features let you transform data as it enters and leaves the database (accessors, mutators, casting), create reusable query "recipes" (scopes), hook into every moment of a model's lifecycle (events, observers), and work with groups of models in powerful ways (collections). These features turn Eloquent from a simple ORM into a complete data management layer.

### Key Characteristics

- **Unified Attribute API:** Since Laravel 9, a single `Attribute` class handles both get and set transformations.
- **Declarative Casting:** The `casts()` method (Laravel 11+) or `$casts` property maps attributes to rich types.
- **Composable Queries:** Local scopes allow named, reusable query constraints.
- **Automatic Filtering:** Global scopes apply constraints to every query for a model.
- **Event-Driven:** Ten built-in model events fire at each lifecycle stage.
- **Observer Pattern:** Observers group all event handlers for a model into one class.
- **Immutable Collections:** Collection methods return new instances, preserving the original.

### Prerequisites

- A working Laravel 12.x application with Eloquent models and a configured database.
- Understanding of basic Eloquent CRUD and relationships.
- PHP 8.1+ (Laravel 12 requires PHP 8.2+ in practice).
- For custom casts: knowledge of the `CastsAttributes` contract.

### Related Programming Areas

- **Eloquent Relationships:** Accessors and casts often interact with relationship data.
- **API Resources:** Casts and accessors shape JSON output.
- **Service Providers:** Global scopes and observers are registered in `AppServiceProvider`.
- **Queue Jobs:** Observers often dispatch queued jobs for background processing.

---

## 1. Accessors

### Definitions

**Core Definition:** An accessor is a method that transforms an Eloquent attribute's value when it is retrieved from the model.

**Technical Definition:** In Laravel 9+, accessors are defined as protected methods returning an `Illuminate\Database\Eloquent\Casts\Attribute` instance. The `get` closure receives the raw database value and optionally the `$attributes` array, returning the transformed value. The `HasAttributes` trait intercepts `__get()` calls and resolves the accessor.

**Beginner-Friendly Explanation:** An accessor is a "getter" that cleans up data when you read it. If your database stores `first_name` as `"john"`, an accessor can automatically return `"John"` when you access `$user->first_name`.

### Purposes

- To transform raw database values into human-readable or application-specific formats.
- To combine multiple database columns into a single computed attribute.
- To format dates, currency, names, or other data consistently.
- To provide computed properties (e.g., `full_name` from `first_name` and `last_name`).

### Syntax Rules and Structure

**Complete General Syntax:**

```php
protected function attributeName(): Attribute
{
    return Attribute::make(
        get: fn (mixed $value, array $attributes) => /* transformed value */,
    );
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `attributeName()` | Camel-case method name matching the attribute. |
| `Attribute::make(get: ...)` | Creates the `Attribute` instance with a get closure. |
| `$value` | The raw database value. |
| `$attributes` | Array of all current model attributes. |

**Syntax Rules:**
- Method must be `protected` and return `Attribute`.
- Method name is camelCase of the database column.
- Accessors are cached per model instance.
- Append computed values via `$appends` for JSON inclusion.

**Constraints and Limitations:**
- Accessors are not automatically included in `toArray()`/`toJson()` unless appended.
- Accessor caching for primitive values requires explicit enabling in Laravel 10+.

### Multiple Annotated Complete Code Examples

**Example 1: Basic Name Accessor**

```php
<?php
namespace App\Models;

use Illuminate\Database\Eloquent\Casts\Attribute;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    protected function firstName(): Attribute
    {
        return Attribute::make(
            get: fn (string $value) => ucfirst($value),
        );
    }
}

// Usage
$user = User::find(1);
echo $user->first_name; // "John" (if stored as "john")
```

**Expected Output:** `"John"` (the first letter is capitalised).

**Why:** The `get` closure receives the raw value `"john"` and returns `ucfirst("john")`. Eloquent caches the result on the model.

---

**Example 2: Multi-Attribute Accessor (Value Object)**

```php
protected function address(): Attribute
{
    return Attribute::make(
        get: fn (mixed $value, array $attributes) => new Address(
            $attributes['address_line_one'],
            $attributes['address_line_two'],
        ),
    );
}

$user->address->lineOne = 'Updated';
$user->save(); // Syncs back to model
```

**Expected Output:** An `Address` value object is returned. Modifying it and saving syncs back to the underlying columns.

**Why:** The accessor receives the full `$attributes` array, enabling composition of multiple columns. Eloquent caches the value object instance.

### Real-World Cases
- Formatting phone numbers, postal codes, or currency values.
- Computing `full_name` from `first_name` and `last_name`.
- Decrypting sensitive attributes on retrieval.

### References
- Eloquent: Mutators & Casting — https://laravel.com/docs/12.x/eloquent-mutators
- Eloquent Mutators (GitHub) — https://github.com/Vectorial1024/docs/blob/12.x/eloquent-mutators.md

---

## 2. Mutators

### Definitions

**Core Definition:** A mutator is a method that transforms an Eloquent attribute's value before it is stored in the database.

**Technical Definition:** In Laravel 9+, mutators are defined within the same `Attribute` instance using the `set` closure. The `set` closure receives the value being assigned and must return the value to be stored. It may also return an array to set multiple columns.

**Beginner-Friendly Explanation:** A mutator is a "setter" that prepares data before it hits the database. If you set `$user->password = 'secret'`, a mutator can automatically hash it before saving.

### Purposes

- To hash passwords before storage.
- To normalise input (trim, lowercase, format).
- To encrypt or encode sensitive data.
- To set multiple related columns from a single assignment.

### Syntax Rules and Structure

```php
protected function attributeName(): Attribute
{
    return Attribute::make(
        set: fn (mixed $value) => /* value to store */,
    );
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `set` closure | Receives the assigned value. |
| Return value | The value to store in the database. |

**Syntax Rules:**
- The `set` closure may return a string or an array (for multiple columns).
- When returning an array, keys become column names.
- Accessors and mutators can be combined in a single `Attribute::make()` call.

**Constraints and Limitations:**
- Mutators are not called on mass assignment if the attribute is not fillable.
- Returning an array from a mutator sets multiple model attributes.

### Multiple Annotated Complete Code Examples

**Example 1: Password Hashing Mutator**

```php
protected function password(): Attribute
{
    return Attribute::make(
        set: fn (string $value) => bcrypt($value),
    );
}

$user->password = 'secret';
$user->save(); // Stores the bcrypt hash
```

**Expected Output:** The `password` column stores a bcrypt hash instead of `"secret"`.

**Why:** The `set` closure intercepts the assignment and transforms the value before persistence.

---

**Example 2: Combining Accessor and Mutator**

```php
protected function firstName(): Attribute
{
    return Attribute::make(
        get: fn (string $value) => ucfirst($value),
        set: fn (string $value) => strtolower($value),
    );
}

$user->first_name = 'JOHN';   // Stored as 'john'
echo $user->first_name;       // 'John'
```

**Expected Output:** Storage: `"john"`; retrieval: `"John"`.

**Why:** The `set` closure normalises to lowercase; the `get` closure capitalises on retrieval.

### Real-World Cases
- Hashing passwords before storage.
- Normalising email addresses to lowercase.
- Formatting phone numbers on input.

### References
- Eloquent: Mutators & Casting — https://laravel.com/docs/12.x/eloquent-mutators

---

## 3. Attribute Casting

### Definitions

**Core Definition:** Attribute casting provides automatic type conversion for model attributes when retrieved from or stored in the database, without requiring custom accessor/mutator methods.

**Technical Definition:** The `casts()` method (Laravel 11+) or `$casts` property defines a map of attribute names to cast types. Laravel's `HasAttributes` trait applies the cast during `getAttributeValue()` and `setAttribute()`. Built-in casts include `integer`, `float`, `boolean`, `string`, `array`, `json`, `object`, `collection`, `date`, `datetime`, `timestamp`, `decimal`, `encrypted`, and more.

**Beginner-Friendly Explanation:** Casting is like telling Eloquent "this column is actually a boolean" or "this is a JSON array." You don't write transformation code—you just declare the type.

### Purposes
- To automatically convert database values to PHP types.
- To eliminate repetitive accessor/mutator boilerplate.
- To handle JSON/array columns transparently.
- To cast dates to `Carbon` instances.

### Syntax Rules and Structure

```php
protected function casts(): array
{
    return [
        'is_admin' => 'boolean',
        'options' => 'array',
        'created_at' => 'datetime',
        'price' => 'decimal:2',
    ];
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `casts()` | Method returning cast definitions (Laravel 11+). |
| `'boolean'` | Converts `0`/`1` to `true`/`false`. |
| `'array'` | JSON-decodes to PHP array. |
| `'decimal:2'` | Formats to 2 decimal places. |

**Syntax Rules:**
- Use `casts()` method (Laravel 11+) or `protected $casts` property (older).
- Multiple cast parameters use `:` (e.g., `decimal:2`).
- Casts override accessor/mutator behaviour for the same attribute.

**Constraints and Limitations:**
- Casting `array`/`json` requires the database column to store valid JSON.
- Encrypted casts require an `APP_KEY`.

### Multiple Annotated Complete Code Examples

**Example 1: Basic Casting**

```php
protected function casts(): array
{
    return [
        'is_admin' => 'boolean',
        'settings' => 'array',
        'published_at' => 'datetime',
    ];
}

$user->is_admin;    // true (stored as 1)
$user->settings;    // ['theme' => 'dark'] (stored as JSON)
$user->published_at; // Carbon instance
```

**Expected Output:** Type-appropriate PHP values are returned automatically.

**Why:** The cast type instructs the `HasAttributes` trait how to transform the raw database value.

### Real-World Cases
- Storing JSON settings in a single column.
- Casting `published_at` to `Carbon` for date manipulation.
- Boolean flags stored as `TINYINT`.

### References
- Eloquent: Mutators & Casting — https://laravel.com/docs/12.x/eloquent-mutators

---

## 4. Custom Casts

### Definitions

**Core Definition:** A custom cast is a user-defined class implementing the `CastsAttributes` interface, allowing transformation of a database attribute into a rich PHP value object or complex type.

**Technical Definition:** Custom cast classes implement `get()` and `set()` methods. The class is referenced in the `casts()` array by its fully qualified class name. Laravel's `HasAttributes` trait resolves the class and calls `get()` during retrieval and `set()` during assignment.

**Beginner-Friendly Explanation:** When built-in casts aren't enough, you write your own cast. For example, a `Money` cast could convert a database integer (cents) into a `MoneyValueObject` with currency formatting methods.

### Purposes
- To transform database columns into domain-specific value objects.
- To encapsulate complex serialisation/deserialisation logic.
- To provide type safety and behaviour for attributes.

### Syntax Rules and Structure

```php
namespace App\Casts;

use Illuminate\Contracts\Database\Eloquent\CastsAttributes;

class MoneyCast implements CastsAttributes
{
    public function get($model, $key, $value, $attributes): MoneyValueObject
    {
        return new MoneyValueObject($value, $attributes['currency'] ?? 'USD');
    }

    public function set($model, $key, $value, $attributes): array
    {
        return [
            'amount' => $value->amount,
            'currency' => $value->currency,
        ];
    }
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `get()` | Transforms database value to PHP value. |
| `set()` | Transforms PHP value to database-storable value. |
| `$attributes` | All model attributes (for accessing related columns). |

**Syntax Rules:**
- `get()` and `set()` are required.
- `set()` may return an array for multiple columns.
- Custom casts are referenced via `'column' => MoneyCast::class`.

**Constraints and Limitations:**
- Constructor dependency injection is not supported in custom casts.
- Casts are resolved without the service container.

### Multiple Annotated Complete Code Examples

**Example 1: Money Value Object Cast**

```php
class Money implements CastsAttributes
{
    public function get($model, $key, $value, $attributes)
    {
        return new MoneyValueObject($value, $attributes['currency'] ?? 'USD');
    }

    public function set($model, $key, $value, $attributes)
    {
        return [
            'amount' => $value->amount, 
            'currency' => $value->currency
        ];
    }
}

// Usage
class Invoice extends Model
{
    protected $casts = ['total' => Money::class];
}

$invoice->total;              // MoneyValueObject
$invoice->total->formatted(); // "$1,234.56"
```

**Expected Output:** `$invoice->total` returns a `MoneyValueObject` with formatting methods.

**Why:** The `get()` method constructs the value object from the raw integer and the `currency` column.

### Real-World Cases
- Money/currency value objects.
- Address value objects spanning multiple columns.
- Enum-like casts for status fields.

### References
- Custom Object Casting in Laravel — https://laravel-news.com/cast-attributes

---

## 5. Query Scopes

### Definitions

**Core Definition:** A local query scope is a reusable query constraint defined as a method on a model, callable as a method on the model's query builder.

**Technical Definition:** Local scopes are defined as methods prefixed with `scope` (e.g., `scopeActive()`) or, in Laravel 12+, marked with the `#[Scope]` attribute. Laravel strips the prefix when calling, so `scopeActive()` becomes `->active()`. Dynamic scopes accept parameters.

**Beginner-Friendly Explanation:** Scopes are "query recipes" you define once and reuse everywhere. Instead of writing `->where('active', true)` repeatedly, you define `->active()` and call it wherever needed.

### Purposes
- To encapsulate commonly used query constraints.
- To make queries more readable and expressive.
- To centralise business logic for filtering.
- To support dynamic, parameterised filtering.

### Syntax Rules and Structure

```php
// Traditional (Laravel 11 and below)
public function scopeActive(Builder $query): void
{
    $query->where('active', true);
}
```

```php
// Laravel 12+ with attribute
use Illuminate\Database\Eloquent\Attributes\Scope;

#[Scope]
protected function active(Builder $query): void
{
    $query->where('active', true);
}

// Usage
$users = User::active()->get();
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `scope` prefix | Traditional way; Laravel strips it. |
| `#[Scope]` | Laravel 12 attribute; keeps method name as-is. |
| `Builder $query` | The query builder instance to modify. |
| Return | Scopes return `void`; they modify the builder. |

**Syntax Rules:**
- Scopes must accept a `Builder` as the first argument.
- Dynamic scopes add extra parameters after `$query`.
- Scopes can be chained with other query methods and scopes.

**Constraints and Limitations:**
- Scopes cannot be called on relationship query builders without modification.
- Scope names must not conflict with existing query builder methods.

### Multiple Annotated Complete Code Examples

**Example 1: Basic Local Scope**

```php
public function scopePopular(Builder $query): void
{
    $query->where('votes', '>', 100);
}

// Usage
$posts = Post::popular()->get();
```

**Expected Output:** Posts with more than 100 votes.

**Why:** The `scopePopular` method adds a `WHERE votes > 100` constraint.

---

**Example 2: Dynamic Scope**

```php
public function scopeOfType(Builder $query, string $type): void
{
    $query->where('type', $type);
}

// Usage
$posts = Post::ofType('article')->get();
```

**Expected Output:** Posts where `type = 'article'`.

**Why:** Dynamic scopes accept parameters that are injected into the query constraints.

### Real-World Cases
- `Post::published()` for filtering published content.
- `User::active()` for active accounts.
- `Order::recent()` for orders within the last 30 days.

### References
- Eloquent: Getting Started — https://laravel.com/docs/12.x/eloquent
- Local Model Scopes with `#[Scope]` — https://laravel-news.com/local-model-scopes-with-the-scope-attribute

---

## 6. Global Scopes

### Definitions

**Core Definition:** A global scope is a query constraint automatically applied to **every** query for a given model, without requiring explicit invocation.

**Technical Definition:** Global scopes implement the `Illuminate\Database\Eloquent\Scope` interface with an `apply()` method. They are registered via the `#[ScopedBy]` attribute (Laravel 12+) or the `addGlobalScope()` method. Soft deletes are implemented as a global scope.

**Beginner-Friendly Explanation:** Global scopes are like "invisible filters" that apply to every query. The best example is soft deletes—Laravel automatically excludes trashed records from all queries without you writing `where deleted_at IS NULL`.

### Purposes
- To enforce data consistency rules across all queries.
- To implement multi-tenancy isolation automatically.
- To filter out soft-deleted records.
- To apply default ordering or visibility constraints.

### Syntax Rules and Structure

```php
namespace App\Models\Scopes;

use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Scope;

class ActiveScope implements Scope
{
    public function apply(Builder $builder, Model $model): void
    {
        $builder->where('is_active', true);
    }
}

// Register via attribute (Laravel 12+)
#[ScopedBy([ActiveScope::class])]
class Product extends Model {}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `apply()` | Receives the builder and model; adds constraints. |
| `#[ScopedBy]` | Attribute to register scopes on a model. |
| `withoutGlobalScopes()` | Bypasses all global scopes. |
| `withoutGlobalScope(Scope::class)` | Bypasses a specific scope. |

**Syntax Rules:**
- Global scopes are registered in the model or via `addGlobalScope()` in a service provider.
- Use `withoutGlobalScopes()` to bypass for specific queries.
- Anonymous global scopes use a closure instead of a class.

**Constraints and Limitations:**
- Global scopes are not applied to relationship queries unless explicitly configured.
- Overuse can make debugging difficult.

### Multiple Annotated Complete Code Examples

**Example 1: Multi-Tenant Global Scope**

```php
class TenantScope implements Scope
{
    public function apply(Builder $builder, Model $model): void
    {
        if (auth()->check() && auth()->user()->tenant_id) {
            $builder->where('tenant_id', auth()->user()->tenant_id);
        }
    }
}

#[ScopedBy([TenantScope::class])]
class Document extends Model {}

// All queries now filter by tenant
$documents = Document::all(); // Only current tenant's documents

// Bypass for admin
$allDocuments = Document::withoutGlobalScopes()->get();
```

**Expected Output:** Normal queries return only the current tenant's documents. `withoutGlobalScopes()` returns all documents.

**Why:** The `TenantScope` applies a `WHERE tenant_id = ?` constraint automatically to every query.

### Real-World Cases
- Multi-tenant SaaS data isolation.
- Soft delete filtering.
- Published-only content filtering.

### References
- Laravel Global Scopes — https://laravel-news.com/index.php/global-scopes-query-filtering

---

## 7. Model Events

### Definitions

**Core Definition:** Model events are hooks that fire at specific points in a model's lifecycle—creation, update, deletion, retrieval, and restoration—allowing custom logic to execute automatically.

**Technical Definition:** The `HasEvents` trait dispatches events via the `fireModelEvent()` method. Available events: `retrieved`, `creating`, `created`, `updating`, `updated`, `saving`, `saved`, `deleting`, `deleted`, `restoring`, `restored`, `forceDeleting`, `forceDeleted`, `replicating`, and `trashed` (for soft deletes).

**Beginner-Friendly Explanation:** Events let you say "when a user is created, send a welcome email" or "before a post is deleted, archive its comments." You hook into the lifecycle without modifying the core save/delete logic.

### Purposes
- To execute logic before or after persistence operations.
- To validate, transform, or augment data during saves.
- To trigger side effects (notifications, logging, cache invalidation).
- To prevent operations by returning `false` from a "before" event.

### Syntax Rules and Structure

```php
// In the model's booted() method
protected static function booted(): void
{
    static::creating(function (User $user) {
        $user->uuid = Str::uuid();
    });

    static::created(function (User $user) {
        Mail::to($user)->send(new WelcomeEmail($user));
    });
}
```

**Component Breakdown:**

| Event | Timing |
|-------|--------|
| `retrieved` | After model fetched from DB. |
| `creating` / `created` | Before / after INSERT. |
| `updating` / `updated` | Before / after UPDATE. |
| `saving` / `saved` | Before / after INSERT or UPDATE. |
| `deleting` / `deleted` | Before / after DELETE. |
| `restoring` / `restored` | Before / after soft-delete restore. |

**Syntax Rules:**
- Events are registered in the `booted()` static method.
- Return `false` from a "before" event to cancel the operation.
- `saving` fires for both creates and updates.

**Constraints and Limitations:**
- Events do not fire on mass updates (`Model::where(...)->update()`).
- Events are not fired when using `insert()` for bulk operations.

### Multiple Annotated Complete Code Examples

**Example 1: UUID Generation on Create**

```php
protected static function booted(): void
{
    static::creating(function (User $user) {
        $user->uuid = (string) Str::uuid();
    });
}

$user = User::create(['name' => 'John']);
echo $user->uuid; // "550e8400-e29b-41d4-a716-446655440000"
```

**Expected Output:** A UUID is automatically assigned before insertion.

**Why:** The `creating` event fires before the `INSERT` statement, allowing attribute augmentation.

### Real-World Cases
- Sending welcome emails on user creation.
- Invalidating cache on update.
- Logging deletions for audit trails.

### References
- Events — https://laravel.com/docs/12.x/events
- Model Events List — https://github.com/voku/agent-skills/blob/main/AGENTS.md

---

## 8. Observers

### Definitions

**Core Definition:** An observer is a dedicated class that groups event listeners for a specific model, providing a clean, organised way to handle multiple model events.

**Technical Definition:** Observers are registered via the `#[ObservedBy]` attribute (Laravel 10+) or the `observe()` method in a service provider's `boot()` method. Each observer method corresponds to a model event (`created`, `updated`, `deleted`, etc.).

**Beginner-Friendly Explanation:** Instead of scattering event listeners across your codebase, an observer collects all of a model's event handlers in one class. It's like a "lifecycle manager" for a specific model.

### Purposes
- To group all event handling for a model into one class.
- To keep models clean by moving side-effect logic out.
- To enable testing of lifecycle logic independently.
- To register multiple event handlers in one place.

### Syntax Rules and Structure

```php
namespace App\Observers;

use App\Models\User;

class UserObserver
{
    public function creating(User $user): void
    {
        $user->uuid = (string) Str::uuid();
    }

    public function created(User $user): void
    {
        Mail::to($user)->send(new WelcomeEmail($user));
    }

    public function deleting(User $user): void
    {
        // Cleanup
    }
}

// Register via attribute
#[ObservedBy(UserObserver::class)]
class User extends Model {}
```

**Component Breakdown:**

| Method | Corresponds to Event |
|--------|----------------------|
| `creating()` | `creating` event |
| `created()` | `created` event |
| `updating()` | `updating` event |
| `deleting()` | `deleting` event |

**Syntax Rules:**
- Observer methods receive the model instance.
- Method names match the event name.
- Register via `#[ObservedBy]` or `User::observe(UserObserver::class)`.

**Constraints and Limitations:**
- Observers do not fire on mass updates.
- Observers add startup overhead if registered for unused models.

### Multiple Annotated Complete Code Examples

**Example 1: User Observer**

```php
namespace App\Observers;

use App\Models\User;
use Illuminate\Support\Str;

class UserObserver
{
    public function creating(User $user): void
    {
        $user->uuid = (string) Str::uuid();
    }

    public function created(User $user): void
    {
        logger("User {$user->id} created");
    }
}

#[ObservedBy(UserObserver::class)]
class User extends Model {}
```

**Expected Output:** A UUID is generated on creation, and a log entry is recorded.

**Why:** The observer's `creating` method runs before `INSERT`; `created` runs after.

### Real-World Cases
- User lifecycle management (UUID, welcome email, audit log).
- Order processing (inventory updates, notifications).
- Soft-delete cleanup (restoring related records).

### References
- Events — https://laravel.com/docs/12.x/events
- HasEvents Trait — https://api.laravel.com/docs/12.x/Illuminate/Database/Eloquent/Concerns/HasEvents.html

---

## 9. Collections

### Definitions

**Core Definition:** Eloquent Collections are enhanced arrays that wrap multiple model instances and provide dozens of chainable methods for filtering, mapping, reducing, and transforming data.

**Technical Definition:** Eloquent queries returning multiple results return `Illuminate\Database\Eloquent\Collection` instances, which extend `Illuminate\Support\Collection`. Eloquent collections add model-specific methods such as `load()`, `loadMissing()`, `modelKeys()`, `find()`, `fresh()`, `append()`, and `contains()`.

**Beginner-Friendly Explanation:** Collections are like arrays on steroids. You can loop over them like arrays, but you can also chain powerful methods: `->filter()`, `->map()`, `->sum()`, `->pluck()`, `->groupBy()`, and many more.

### Purposes
- To provide fluent, chainable manipulation of multiple models.
- To filter, map, and reduce data without explicit loops.
- To lazy-load relationships on already-retrieved collections.
- To perform aggregate operations (`sum`, `avg`, `max`).

### Syntax Rules and Structure

```php
$users = User::where('active', 1)->get();

$names = $users->map(fn($user) => $user->name)->filter()->values();

$totalVotes = Post::all()->sum('votes');

$grouped = Order::all()->groupBy('status');
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `map()` | Transforms each element. |
| `filter()` | Keeps elements passing a truth test. |
| `reject()` | Removes elements passing a truth test. |
| `pluck()` | Extracts a single column. |
| `sum()` / `avg()` | Aggregate operations. |
| `groupBy()` | Groups elements by a key. |
| `load()` | Eager loads relationships on the collection. |

**Syntax Rules:**
- Collection methods are immutable; each returns a new collection.
- Methods can be chained indefinitely.
- Eloquent collections convert to base collections for certain methods (`pluck`, `keys`, `zip`).

**Constraints and Limitations:**
- Collections load all records into memory; use `chunk()` or `cursor()` for large datasets.
- `map()` returning non-models converts to a base collection.

### Multiple Annotated Complete Code Examples

**Example 1: Chaining Collection Methods**

```php
$users = User::all();

$names = $users
    ->reject(fn($user) => $user->active === false)
    ->map(fn($user) => $user->name)
    ->sort()
    ->values();

// Output: ['Alice', 'Bob', 'Charlie']
```

**Expected Output:** A sorted, indexed collection of names of active users.

**Why:** `reject()` removes inactive users, `map()` extracts names, `sort()` sorts them, `values()` re-indexes.

---

**Example 2: Eloquent Collection `load()`**

```php
$posts = Post::all(); // No relationships loaded
$posts->load('author', 'comments'); // Eager loads now

foreach ($posts as $post) {
    echo $post->author->name; // No additional query
}
```

**Expected Output:** Authors and comments are eagerly loaded for the entire collection in 2 queries.

**Why:** `load()` performs eager loading on an already-retrieved collection, avoiding N+1 queries.

### Real-World Cases
- Building API responses with filtered/mapped data.
- Dashboard statistics (`sum`, `avg`, `count`).
- Grouping records for display.

### References
- Collections — https://laravel.com/docs/12.x/collections
- Eloquent: Collections — https://laravel.com/framework/docs/12.x/eloquent-collections
- `hasMany()` Collection Method (Laravel 12.50) — https://laravel-news.com/hasmany-collection-method-laravel-12-50

---

## References

- Eloquent: Mutators & Casting — https://laravel.com/docs/12.x/eloquent-mutators
- Eloquent: Getting Started — https://laravel.com/docs/12.x/eloquent
- Events — https://laravel.com/docs/12.x/events
- Collections — https://laravel.com/docs/12.x/collections
- Eloquent: Collections — https://laravel.com/framework/docs/12.x/eloquent-collections
- Custom Object Casting in Laravel — https://laravel-news.com/cast-attributes
- Laravel Global Scopes — https://laravel-news.com/index.php/global-scopes-query-filtering
- Local Model Scopes with `#[Scope]` — https://laravel-news.com/local-model-scopes-with-the-scope-attribute
- `hasMany()` Collection Method (Laravel 12.50) — https://laravel-news.com/hasmany-collection-method-laravel-12-50
- HasEvents Trait — https://api.laravel.com/docs/12.x/Illuminate/Database/Eloquent/Concerns/HasEvents.html
- Model Events List — https://github.com/voku/agent-skills/blob/main/AGENTS.md
- Eloquent Mutators (GitHub) — https://github.com/Vectorial1024/docs/blob/12.x/eloquent-mutators.md