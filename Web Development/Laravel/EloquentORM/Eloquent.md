# Laravel Eloquent Fundamentals & Configuration: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Eloquent is Laravel's Object-Relational Mapper (ORM) that implements the Active Record pattern, allowing developers to interact with database tables as PHP objects. Each model class corresponds to a database table, and each model instance corresponds to a row in that table.

**Technical Definition:** Eloquent is implemented through the `Illuminate\Database\Eloquent\Model` class, which extends the query builder and provides an ActiveRecord implementation. The model system handles attribute casting, mass assignment protection, relationship resolution, event dispatching, and persistence operations (create, read, update, delete) through methods such as `save()`, `find()`, `update()`, and `delete()`.

**Beginner-Friendly Explanation:** Imagine you have a spreadsheet with rows of data. Eloquent lets you treat each row as a PHP object. Instead of writing SQL queries like `SELECT * FROM users WHERE id = 1`, you write `User::find(1)`. The object knows how to save, update, and delete itself—you just work with PHP objects, and Eloquent handles the database behind the scenes.

### Key Characteristics

- **Active Record Pattern:** Each model instance wraps a database row and includes persistence methods.
- **Convention over Configuration:** Table names, primary keys, and timestamps are inferred from class names, reducing boilerplate.
- **Mass Assignment Protection:** Built-in security against over-posting attacks via `$fillable` and `$guarded`.
- **Attribute Tracking:** Models track changes to their attributes through methods like `isDirty()`, `wasChanged()`, and `getOriginal()`.
- **Relationship Management:** Models define relationships (one-to-many, many-to-many, polymorphic) as methods.
- **Event Lifecycle:** Models fire events (`creating`, `created`, `updating`, `updated`, `deleting`, `deleted`) that can be hooked into for custom logic.
- **Multi-Database Support:** Models can specify custom database connections.
- **Route Model Binding:** Eloquent models can be automatically resolved from route parameters.

### Prerequisites

- **PHP 8.1+** (Laravel 10.x) or **PHP 8.2+** (Laravel 11.x/12.x).
- A configured database connection in `config/database.php` and `.env`.
- Basic understanding of SQL and database tables.
- Familiarity with PHP classes, properties, and methods.
- Laravel's `illuminate/database` package (included by default in full Laravel installations).

### Related Programming Areas

- **Database Migrations:** Define the table schema that Eloquent models map to.
- **Query Builder:** Eloquent models extend the query builder, providing access to all its methods.
- **Service Container:** Models are resolved through Laravel's dependency injection container.
- **Events & Listeners:** Model lifecycle events integrate with Laravel's event system.
- **Testing & Seeding:** Mass assignment protection is automatically disabled during seeding; factories use models for test data generation.
- **API Resources:** Eloquent models are the primary data source for API transformation layers.

### Core Concepts / Features

1. Active Record Pattern
2. Model Conventions (Table Names, Primary Keys, Timestamps, Custom Connections)
3. Mass Assignment Protection (`$fillable` vs. `$guarded`, `Model::unguard()`)
4. Model State & Lifecycle (`isDirty()`, `wasChanged()`, `getOriginal()`)

---

## 1. Active Record Pattern

### Definitions

**Core Definition:** The Active Record pattern is a design pattern where an object wraps a row in a database table, encapsulating both the data and the behaviour (persistence methods) for that row.

**Technical Definition:** In the Active Record pattern, each model class corresponds to a database table, and each instance corresponds to a row. The model instance carries its data attributes and provides methods such as `save()`, `update()`, `delete()`, and `refresh()` that operate on the underlying database row. Eloquent's implementation stores the original attribute values in an `$original` array and the current values in an `$attributes` array, using a `$changes` array to track modifications.

**Beginner-Friendly Explanation:** Think of an Active Record model as a "smart row." It's not just a passive container of data—it knows how to save itself to the database, update itself, and delete itself. You don't need a separate "database manager" class; the row object handles everything.

### Purposes

- To provide an intuitive, object-oriented interface for database operations without writing raw SQL.
- To encapsulate persistence logic within the model, keeping data and behaviour together.
- To enable rapid application development by reducing boilerplate code for CRUD operations.
- To support fluent, chainable query building through the model's static methods.
- To facilitate relationship definition and eager loading between related models.
- To provide lifecycle hooks (events) for injecting custom logic at specific persistence stages.

### Syntax Rules and Structure

**Complete General Syntax — Defining a Model:**

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    // Model configuration properties
    protected $table = 'my_flights';
    protected $primaryKey = 'flight_id';
    public $incrementing = false;
    protected $keyType = 'string';
    public $timestamps = false;
    protected $connection = 'mysql2';
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `extends Model` | The base `Illuminate\Database\Eloquent\Model` class. |
| `$table` | Overrides the inferred table name. |
| `$primaryKey` | Overrides the inferred primary key column name. |
| `$incrementing` | Whether the primary key is auto-incrementing. |
| `$keyType` | The data type of the primary key (`'int'` or `'string'`). |
| `$timestamps` | Whether to maintain `created_at` and `updated_at`. |
| `$connection` | The database connection name to use. |

**Complete General Syntax — Basic CRUD Operations:**

```php
// Create
$flight = new Flight();
$flight->name = 'Tokyo';
$flight->save();

// Read
$flight = Flight::find(1);
$flights = Flight::where('active', 1)->get();

// Update
$flight->name = 'Osaka';
$flight->save();

// Delete
$flight->delete();
```

**Syntax Rules:**

- Model class names should be **singular** and in **StudlyCase** (e.g., `User`, `BlogPost`).
- The model must extend `Illuminate\Database\Eloquent\Model`.
- The model's corresponding table should exist in the database before performing operations.
- Static methods (`Flight::where()`, `Flight::find()`) return query builder instances or model instances/collections.
- Instance methods (`$flight->save()`, `$flight->delete()`) operate on a single row.

**Constraints and Limitations:**

- **Composite primary keys are not supported** by Eloquent models. Each model must have a single, uniquely identifying primary key.
- **No automatic schema management:** Eloquent assumes the table exists; use migrations to create it.
- **Performance overhead:** Eloquent adds an abstraction layer; for complex, high-performance queries, raw SQL may be more appropriate.
- **N+1 query problem:** Lazy loading relationships in loops can cause performance issues; use `with()` for eager loading.

### Multiple Annotated Complete Code Examples

**Example 1: Creating and Saving a Model**

```php
<?php
// app/Models/Flight.php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    // Table name inferred as 'flights' (plural snake_case)
}
```

```php
<?php
// routes/web.php or a controller

use App\Models\Flight;

// Create a new model instance (no database row yet)
$flight = new Flight();

// Set attributes
$flight->name = 'Tokyo';
$flight->destination = 'Osaka';
$flight->price = 150;

// Persist to the database — INSERT INTO flights ...
$flight->save();

// After save(), the model has its primary key populated
echo $flight->id; // e.g., 1
```

**Expected Output:** A new row is inserted into the `flights` table with `name = 'Tokyo'`, `destination = 'Osaka'`, `price = 150`, and `created_at`/`updated_at` timestamps set. `$flight->id` returns the auto-incremented primary key.

**Why:** The `save()` method determines whether the model exists (via the `exists` property). If it does not, it performs an `INSERT`; if it does, it performs an `UPDATE`. The `created_at` and `updated_at` columns are automatically populated because `$timestamps` defaults to `true`.

---

**Example 2: Retrieving and Updating a Model**

```php
<?php
// Find a model by primary key
$flight = Flight::find(1);

if ($flight) {
    // Modify an attribute
    $flight->price = 200;

    // Persist the change — UPDATE flights SET price = 200 WHERE id = 1
    $flight->save();

    echo "Updated price to {$flight->price}";
} else {
    echo "Flight not found.";
}
```

**Expected Output:** The row with `id = 1` has its `price` column updated to `200`. The `updated_at` timestamp is refreshed. Output: `Updated price to 200`.

**Why:** `find()` retrieves the model by primary key. Changing the `price` attribute marks it as "dirty" (modified). `save()` detects the dirty attribute and issues an `UPDATE` statement for only the changed columns.

---

**Example 3: Deleting a Model**

```php
<?php
// Retrieve the model
$flight = Flight::find(1);

if ($flight) {
    // Delete the row — DELETE FROM flights WHERE id = 1
    $flight->delete();

    echo "Flight deleted.";
}
```

**Expected Output:** The row with `id = 1` is removed from the `flights` table. Output: `Flight deleted.`

**Why:** The `delete()` method issues a `DELETE` statement. After deletion, the model's `exists` property is set to `false`, but the object remains in memory.

### Real-World Cases

- **User Management:** `User::create($request->validated())` creates a new user from form data.
- **Blog Posts:** `Post::where('published', true)->latest()->paginate(10)` retrieves published posts for a blog index.
- **E-commerce:** `Order::find($orderId)->update(['status' => 'shipped'])` updates order status after dispatch.
- **API Responses:** `Product::with('category')->get()` eager-loads relationships for JSON API responses.
- **Audit Trails:** Model events (`updating`, `updated`) log changes to sensitive records.

### References

- Laravel Eloquent Documentation — https://laravel.com/docs/12.x/eloquent
- Laravel 13.x Eloquent Documentation — https://laravel.com/framework/docs/13.x/eloquent
- Active Record Pattern (Wikipedia) — https://en.wikipedia.org/wiki/Active_record_pattern
- Eloquent Model System DeepWiki — https://deepwiki.com/laravel/framework/3-eloquent-model-system

---

## 2. Model Conventions

### Definitions

**Core Definition:** Model conventions are the default naming and behavioural rules that Eloquent assumes when mapping a model class to a database table, including table name, primary key, and timestamp handling.

**Technical Definition:** Eloquent's `Model` class applies conventions during instantiation and query building. The table name is derived from the class name by converting it to snake_case and pluralising it (via `Str::pluralStudly()`). The primary key defaults to `id`, the key type to `int`, and incrementing to `true`. Timestamps default to `true`, expecting `created_at` and `updated_at` columns. The connection defaults to the application's default database connection.

**Beginner-Friendly Explanation:** Laravel makes smart guesses about your database structure. If you have a model called `User`, it assumes the table is called `users`, the primary key is `id`, and there are `created_at` and `updated_at` columns. If your database doesn't follow these rules, you can tell the model what to use.

### Purposes

- To eliminate repetitive configuration by inferring common database structure conventions.
- To reduce boilerplate code in model classes, keeping them focused on business logic.
- To provide sensible defaults that work for the vast majority of applications.
- To allow easy overriding when databases follow different naming standards.
- To enable automatic timestamp management for created and updated records.
- To support multi-database applications through per-model connection configuration.

### Syntax Rules and Structure

**Complete General Syntax — Table Name:**

```php
class Flight extends Model
{
    protected $table = 'my_flights';
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `$table` | The database table name associated with the model. |
| `'my_flights'` | The explicit table name to use instead of the inferred one. |

**Complete General Syntax — Primary Key:**

```php
class Flight extends Model
{
    protected $primaryKey = 'flight_id';
    public $incrementing = false;
    protected $keyType = 'string';
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `$primaryKey` | The column name of the primary key. Default: `'id'`. |
| `$incrementing` | Whether the key auto-increments. Default: `true`. |
| `$keyType` | The key's data type (`'int'` or `'string'`). Default: `'int'`. |

**Complete General Syntax — Timestamps:**

```php
class Flight extends Model
{
    public $timestamps = false;

    // Or customise the format and column names
    const CREATED_AT = 'created';
    const UPDATED_AT = 'updated';
    protected $dateFormat = 'U';
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `$timestamps` | Whether to maintain timestamps. Default: `true`. |
| `CREATED_AT` | Overrides the created-at column name. Default: `'created_at'`. |
| `UPDATED_AT` | Overrides the updated-at column name. Default: `'updated_at'`. |
| `$dateFormat` | The storage format for timestamps (e.g., `'U'` for Unix). |

**Complete General Syntax — Custom Database Connection:**

```php
class Flight extends Model
{
    protected $connection = 'mysql2';
}
```

```php
// Or set at runtime on an instance
$flight = new Flight();
$flight->setConnection('mysql2');
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `$connection` | The database connection name from `config/database.php`. |
| `setConnection('mysql2')` | Sets the connection for a specific model instance. |

**Syntax Rules:**

- Table name convention: **snake_case, plural** form of the class name. `Flight` → `flights`; `SalesPerson` → `sales_persons`.
- Primary key convention: column named `id`, auto-incrementing integer.
- Timestamps: `created_at` and `updated_at` columns must exist if `$timestamps` is `true`.
- Connection: defaults to the `DB_CONNECTION` environment variable value.
- All convention properties are `protected` (except `$incrementing` and `$timestamps`, which are `public`).

**Constraints and Limitations:**

- **Composite primary keys are not supported**; Eloquent requires a single unique identifier.
- **Non-incrementing keys:** If `$incrementing = false`, you must ensure the key is provided before saving.
- **String keys:** If `$keyType = 'string'`, the key will not be cast to an integer.
- **Timestamp columns:** If `$timestamps = true` but the columns don't exist, database errors will occur on save.
- **Connection availability:** The specified connection must be defined in `config/database.php`.

### Multiple Annotated Complete Code Examples

**Example 1: Default Conventions (No Configuration)**

```php
<?php
// app/Models/User.php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    // No table, key, or timestamp configuration needed
}
```

```php
// Usage
$user = User::find(1);
// Eloquent assumes: table 'users', primary key 'id',
// columns 'created_at' and 'updated_at' exist
```

**Expected Output:** `User::find(1)` queries `SELECT * FROM users WHERE id = 1`.

**Why:** The class name `User` is converted to snake_case (`user`) and pluralised to `users`. The primary key defaults to `id`. Timestamps default to `true`.

---

**Example 2: Custom Table Name and Primary Key**

```php
<?php
// app/Models/Flight.php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    // Override the table name
    protected $table = 'my_flights';

    // Override the primary key
    protected $primaryKey = 'flight_id';

    // Disable auto-incrementing (e.g., UUID primary key)
    public $incrementing = false;

    // Set the key type to string
    protected $keyType = 'string';
}
```

```php
// Usage
$flight = Flight::find('a1b2c3d4-...');
// Queries: SELECT * FROM my_flights WHERE flight_id = 'a1b2c3d4-...'
```

**Expected Output:** The model uses the `my_flights` table and `flight_id` primary key with string values.

**Why:** Each convention property overrides Eloquent's default inference. `$incrementing = false` prevents Eloquent from expecting the database to generate the key.

---

**Example 3: Disabling Timestamps and Custom Connection**

```php
<?php
// app/Models/LegacyRecord.php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class LegacyRecord extends Model
{
    // This table has no created_at/updated_at columns
    public $timestamps = false;

    // Use a different database connection
    protected $connection = 'legacy_db';

    // The table name in the legacy database
    protected $table = 'tbl_records';
}
```

```php
// Usage
$record = LegacyRecord::create([
    'name' => 'Test',
    'value' => 100,
]);
// INSERT INTO legacy_db.tbl_records (name, value) VALUES ('Test', 100)
// No created_at/updated_at columns in the INSERT
```

**Expected Output:** A row is inserted into `tbl_records` on the `legacy_db` connection without timestamp columns.

**Why:** `$timestamps = false` disables automatic timestamp management. `$connection = 'legacy_db'` routes the query to the configured legacy database.

### Real-World Cases

- **Legacy Databases:** Applications migrating from legacy systems use `$table` and `$connection` to map to existing tables with non-standard names.
- **UUID Primary Keys:** Modern applications use `$incrementing = false` and `$keyType = 'string'` for UUID-based primary keys.
- **Read/Write Splitting:** Models can specify a read connection for queries and a write connection for inserts/updates via connection configuration.
- **Multi-Tenant Applications:** Each tenant's data lives in a separate database; models set `$connection` dynamically based on the tenant.
- **Pivot Tables:** Pivot models use custom table names (e.g., `role_user` for a `Role` and `User` many-to-many relationship).

### References

- Laravel Eloquent: Table Names — https://laravel.com/docs/12.x/eloquent#table-names
- Laravel Eloquent: Primary Keys — https://laravel.com/docs/12.x/eloquent#primary-keys
- Laravel Eloquent: Timestamps — https://laravel.com/docs/12.x/eloquent#timestamps
- Laravel Eloquent: Database Connections — https://laravel.com/docs/12.x/eloquent#database-connections
- Laravel 8.x Model Conventions (PDF) — https://github.com/DevStorm-Team/laravel-book/blob/master/laravel-docs-8.x.pdf

---

## 3. Mass Assignment Protection

### Definitions

**Core Definition:** Mass assignment protection is Eloquent's security mechanism that prevents unintended attributes from being set on a model when passing an array of data to methods like `create()`, `fill()`, or `update()`.

**Technical Definition:** The `GuardsAttributes` trait (in `Illuminate\Database\Eloquent\Concerns`) provides the `$fillable` and `$guarded` properties, along with static `unguard()` and `reguard()` methods. When mass assignment occurs, Eloquent checks each key in the input array against the model's fillable or guarded list. If a key is not fillable (or is guarded), it is silently discarded unless `preventSilentlyDiscardingAttributes()` is enabled, in which case a `MassAssignmentException` is thrown.

**Beginner-Friendly Explanation:** Mass assignment protection stops users from sneaking in fields you didn't intend to allow. If your form only has `name` and `email`, but a malicious user adds `is_admin=1` to the request, Eloquent ignores it—unless you explicitly allow `is_admin` in your `$fillable` array. This prevents privilege-escalation attacks.

### Purposes

- To prevent over-posting attacks where users inject unexpected fields into requests.
- To provide a whitelist (`$fillable`) or blacklist (`$guarded`) approach to attribute assignment.
- To allow temporary disabling of protection during database seeding and testing via `Model::unguard()`.
- To offer granular control over which attributes can be set in bulk.
- To integrate with Laravel's validation system for secure form handling.
- To provide a strict mode during development that throws exceptions for unfillable attributes.

### Syntax Rules and Structure

**Complete General Syntax — `$fillable` (Whitelist):**

```php
class User extends Model
{
    protected $fillable = [
        'name',
        'email',
        'password',
    ];
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `$fillable` | An array of attribute names that are mass assignable. |
| `'name'`, `'email'` | Only these columns can be set via `create()`, `fill()`, or `update()`. |

**Complete General Syntax — `$guarded` (Blacklist):**

```php
class User extends Model
{
    protected $guarded = [
        'id',
        'is_admin',
        'email_verified_at',
    ];
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `$guarded` | An array of attribute names that are **not** mass assignable. |
| `'id'`, `'is_admin'` | These columns cannot be set via mass assignment. |

**Complete General Syntax — Unguard During Seeding/Testing:**

```php
use Illuminate\Database\Eloquent\Model;

// Disable all mass assignment restrictions
Model::unguard();

// Perform seeding or testing operations
User::create([
    'name' => 'Admin',
    'email' => 'admin@example.com',
    'is_admin' => true, // Now allowed
]);

// Re-enable protection
Model::reguard();
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Model::unguard()` | Statically disables all mass assignment protection. |
| `Model::reguard()` | Re-enables mass assignment protection. |

**Complete General Syntax — `unguarded()` Helper:**

```php
Model::unguarded(function () {
    // Mass assignment protection is disabled inside this closure only
    User::create(['name' => 'Test', 'is_admin' => true]);
});
// Protection is automatically restored after the closure
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Model::unguarded(callable)` | Runs the callback with protection disabled, then automatically restores it. |

**Syntax Rules:**

- Use **either** `$fillable` **or** `$guarded`, not both (though both can be set, `$fillable` takes precedence).
- `$guarded = []` means **all attributes are mass assignable** (not recommended for production).
- `$fillable` is typically the preferred approach for new applications (whitelist security).
- `Model::unguard()` is intended for **seeding, testing, and local development only**. Never use it in production request handling.
- Laravel 13 introduced attribute-based configuration: `#[Fillable([...])]` and `#[Unguarded]`.

**Constraints and Limitations:**

- **Nested JSON attributes:** Laravel does not support updating nested JSON attributes when using `$guarded`; the full key must be in `$fillable`.
- **Silent discarding:** By default, unfillable attributes are silently discarded. Use `Model::preventSilentlyDiscardingAttributes()` to throw exceptions during development.
- **Security risk:** `$guarded = []` or `Model::unguard()` in production opens the application to mass assignment vulnerabilities.
- **Automatic during seeding:** Laravel automatically disables mass assignment protection during database seeding via the `db:seed` command.

### Multiple Annotated Complete Code Examples

**Example 1: Whitelist with `$fillable`**

```php
<?php
// app/Models/User.php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    protected $fillable = [
        'name',
        'email',
        'password',
    ];
}
```

```php
// Controller — safe mass assignment
$user = User::create([
    'name' => 'Alice',
    'email' => 'alice@example.com',
    'password' => bcrypt('secret'),
    'is_admin' => true, // IGNORED — not in $fillable
]);

// $user->is_admin is null (or database default)
```

**Expected Output:** A user is created with `name`, `email`, and `password`. The `is_admin` field is silently discarded and does not appear in the `INSERT` statement.

**Why:** Only attributes listed in `$fillable` are passed to the model's `fill()` method. Keys not in the list are filtered out before the database query is built.

---

**Example 2: Blacklist with `$guarded`**

```php
<?php
// app/Models/Post.php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Post extends Model
{
    protected $guarded = [
        'id',
        'user_id',
        'created_at',
        'updated_at',
    ];
}
```

```php
// Controller — all other fields are mass assignable
$post = Post::create([
    'title' => 'Hello World',
    'body' => 'This is the content.',
    'user_id' => 999, // IGNORED — guarded
    'id' => 500,      // IGNORED — guarded
]);
```

**Expected Output:** A post is created with `title` and `body`. The `user_id` and `id` fields are ignored because they are in the `$guarded` array.

**Why:** `$guarded` acts as a blacklist. Any attribute not listed is mass assignable. This is useful when most fields should be fillable.

---

**Example 3: Unguard During Seeding**

```php
<?php
// database/seeders/DatabaseSeeder.php

namespace Database\Seeders;

use App\Models\User;
use Illuminate\Database\Seeder;
use Illuminate\Database\Eloquent\Model;

class DatabaseSeeder extends Seeder
{
    public function run(): void
    {
        // Disable mass assignment protection for seeding
        Model::unguard();

        User::create([
            'name' => 'Admin',
            'email' => 'admin@example.com',
            'password' => bcrypt('password'),
            'is_admin' => true,
            'email_verified_at' => now(),
        ]);

        // Re-enable protection
        Model::reguard();
    }
}
```

**Expected Output:** A fully populated admin user is inserted, including the `is_admin` and `email_verified_at` fields that would normally be protected.

**Why:** `Model::unguard()` temporarily disables the fillable/guarded checks for all models. `Model::reguard()` restores protection after the seeding operation. Laravel also automatically unguards during `db:seed`.

### Real-World Cases

- **User Registration:** `User::create($request->only(['name', 'email', 'password']))` with `$fillable` ensuring only these fields are set.
- **Profile Updates:** `$user->update($request->validated())` where validation and `$fillable` work together for security.
- **Admin Panels:** Models with `$guarded = ['id']` allow administrators to mass-assign most fields while protecting the primary key.
- **Test Factories:** Factories use `Model::unguard()` implicitly or explicitly to create models with all attributes for testing.
- **Data Import:** Import scripts use `Model::unguarded(fn() => Model::create($row))` to safely insert data from external sources.

### References

- Laravel Eloquent: Mass Assignment — https://laravel.com/docs/12.x/eloquent#mass-assignment
- Laravel API: GuardsAttributes Trait — https://api.laravel.com/docs/9.x/Illuminate/Database/Eloquent/Concerns/GuardsAttributes.html
- Laravel 13.x Eloquent: Mass Assignment — https://laravel.com/framework/docs/13.x/eloquent
- Laravel Database Seeding — https://laravel.com/docs/12.x/seeding

---

## 4. Model State & Lifecycle

### Definitions

**Core Definition:** Model state and lifecycle methods allow developers to inspect how a model's attributes have changed since it was retrieved from the database, and to react to those changes at various points in the model's persistence lifecycle.

**Technical Definition:** Eloquent models maintain three internal arrays: `$attributes` (current values), `$original` (values when the model was loaded or last synced), and `$changes` (attributes modified during the current request cycle). Methods such as `isDirty()`, `isClean()`, `wasChanged()`, `getOriginal()`, and `getDirty()` inspect these arrays. The model lifecycle also includes events (`retrieved`, `creating`, `created`, `updating`, `updated`, `saving`, `saved`, `deleting`, `deleted`) that fire at specific persistence stages.

**Beginner-Friendly Explanation:** When you load a user from the database and then change their email, the model remembers the old email. You can ask: "Is this model dirty?" (has anything changed?), "What was the original email?" (getOriginal), or "Was the email changed when I last saved?" (wasChanged). This is useful for logging, validation, and conditional logic.

### Purposes

- To determine whether a model has unsaved changes before persisting.
- To retrieve the original value of an attribute before it was modified.
- To check whether specific attributes were changed during the last save operation.
- To conditionally execute logic based on whether attributes have changed.
- To provide audit trails by comparing original and current values.
- To optimise database updates by saving only dirty attributes.

### Syntax Rules and Structure

**Complete General Syntax — Checking State:**

```php
$user = User::find(1);

// Check if any attributes have been modified
$user->isDirty();           // true/false

// Check if a specific attribute is dirty
$user->isDirty('email');    // true/false

// Check if the model is clean (no changes)
$user->isClean();           // true/false

// Get all dirty attributes
$user->getDirty();          // ['email' => 'new@example.com']

// Get the original value of an attribute
$user->getOriginal('email'); // 'old@example.com'

// Get all original attributes
$user->getOriginal();       // ['id' => 1, 'name' => '...', 'email' => 'old@...']
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `isDirty()` | Returns `true` if any attribute has been modified but not saved. |
| `isDirty('email')` | Returns `true` if the `email` attribute has been modified. |
| `isClean()` | Opposite of `isDirty()`; returns `true` when no modifications exist. |
| `getDirty()` | Returns an array of modified attributes with their new values. |
| `getOriginal('email')` | Returns the original value of the `email` attribute. |
| `getOriginal()` | Returns all original attribute values as an array. |

**Complete General Syntax — Post-Save State:**

```php
$user = User::find(1);
$user->email = 'new@example.com';
$user->save();

// Check if any attributes were changed when the model was last saved
$user->wasChanged();           // true

// Check if a specific attribute was changed
$user->wasChanged('email');    // true

// Get the original value before the save
$user->getOriginal('email');   // 'old@example.com' (after save, this is refreshed)

// Get the changes made during the last save
$user->getChanges();           // ['email' => 'new@example.com']
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `wasChanged()` | Returns `true` if any attributes were changed during the last save. |
| `wasChanged('email')` | Returns `true` if the `email` attribute was changed during the last save. |
| `getChanges()` | Returns an array of attributes changed during the last save. |

**Syntax Rules:**

- `isDirty()` checks the model's **current** state against its **original** state. It returns `false` after `save()` because the original state is updated.
- `wasChanged()` checks the `$changes` array populated after `save()`. It is the correct method to use **after** saving.
- `getOriginal()` before `save()` returns the value as loaded from the database. After `save()`, the original values are synced to the current values.
- `getChanges()` is only meaningful after `save()`.

**Constraints and Limitations:**

- **`isDirty()` after save:** Always returns `false` after `save()` because the model's original state is refreshed.
- **`wasChanged()` before save:** Returns `false` before `save()` because the `$changes` array is only populated during the save operation.
- **Attribute casting:** Casted attributes (e.g., dates, JSON) are compared after casting, not as raw database values.
- **Mutators and accessors:** Changes applied via mutators are reflected in dirty state tracking.
- **Lifecycle events:** The `updating` event fires **before** the database update, so `isDirty()` is still meaningful there. The `updated` event fires **after** the update, so `wasChanged()` is appropriate there.

### Multiple Annotated Complete Code Examples

**Example 1: Tracking Changes Before Save**

```php
<?php
// Retrieve a user
$user = User::find(1);
// Assume: $user->email = 'old@example.com'

// Modify an attribute
$user->email = 'new@example.com';

// Check dirty state
$isDirty = $user->isDirty();           // true
$isEmailDirty = $user->isDirty('email'); // true
$dirtyAttributes = $user->getDirty();  // ['email' => 'new@example.com']

// Get the original value
$originalEmail = $user->getOriginal('email'); // 'old@example.com'

echo "Changed {$originalEmail} to {$user->email}";
// Output: Changed old@example.com to new@example.com
```

**Expected Output:** `Changed old@example.com to new@example.com`

**Why:** Before `save()`, `isDirty()` compares the current `$attributes` array against the `$original` array. The `email` key differs, so the model is dirty. `getOriginal('email')` returns the value from the `$original` array.

---

**Example 2: Using `wasChanged()` in Model Events**

```php
<?php
// app/Models/User.php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    protected static function booted(): void
    {
        // Fires after the model is updated in the database
        static::updated(function (User $user) {
            if ($user->wasChanged('email')) {
                // Log the email change
                $oldEmail = $user->getOriginal('email');
                $newEmail = $user->email;

                logger("User {$user->id} changed email from {$oldEmail} to {$newEmail}");
            }

            if ($user->wasChanged('password')) {
                // Notify the user that their password was changed
                logger("Password changed for user {$user->id}");
            }
        });
    }
}
```

**Expected Output:** When a user's email is updated via `$user->update(['email' => 'new@example.com'])`, the log records: `User 1 changed email from old@example.com to new@example.com`.

**Why:** The `updated` event fires after the database update. At this point, `wasChanged('email')` returns `true` because the `email` key was modified during the save. `getOriginal('email')` returns the value **before** the save because the `$original` array is synced after the event completes.

---

**Example 3: Conditional Logic with `isDirty()` in Observers**

```php
<?php
// app/Observers/UserObserver.php

namespace App\Observers;

use App\Models\User;

class UserObserver
{
    /**
     * Handle the User "updating" event.
     * Fires BEFORE the database update.
     */
    public function updating(User $user): void
    {
        // Only proceed if the email is being changed
        if ($user->isDirty('email')) {
            // Validate the new email domain
            $newEmail = $user->email;
            $domain = substr(strrchr($newEmail, '@'), 1);

            if (!in_array($domain, ['example.com', 'company.org'])) {
                // Reject the update by returning false from the event
                // (In an observer, you can throw an exception or use a custom check)
                throw new \Exception('Email domain not allowed.');
            }
        }
    }
}
```

```php
// Register the observer in AppServiceProvider
public function boot(): void
{
    User::observe(UserObserver::class);
}
```

```php
// Controller — triggers the observer
$user = User::find(1);
$user->email = 'user@evil.com';
$user->save(); // Exception: Email domain not allowed.
```

**Expected Output:** An exception is thrown before the database update when an invalid email domain is detected.

**Why:** The `updating` event fires before the `UPDATE` statement. `isDirty('email')` returns `true` because the email attribute was modified. The observer throws an exception, preventing the save.

### Real-World Cases

- **Audit Logging:** Track changes to sensitive fields (`email`, `password`, `role`) using `wasChanged()` in `updated` events.
- **Conditional Notifications:** Send an email to a user only when their email address changes (`wasChanged('email')`).
- **Optimistic Locking:** Compare `getOriginal('updated_at')` with the current timestamp to detect concurrent modifications.
- **Data Synchronisation:** Use `getChanges()` to determine which fields to sync to an external system (e.g., CRM, Elasticsearch).
- **Validation in Observers:** Use `isDirty()` in `updating` observers to apply field-specific validation rules before persistence.

### References

- Laravel Eloquent: Checking Attribute Changes — https://laravel.com/docs/12.x/eloquent#examining-attribute-changes
- Laravel API: Model::isDirty() — https://laravel.com/api/12.x/Illuminate/Database/Eloquent/Model.html#method_isDirty
- Laravel API: Model::wasChanged() — https://laravel.com/api/12.x/Illuminate/Database/Eloquent/Model.html#method_wasChanged
- Laravel API: Model::getOriginal() — https://laravel.com/api/12.x/Illuminate/Database/Eloquent/Model.html#method_getOriginal
- Laravel Model Events — https://laravel.com/docs/12.x/eloquent#events

---

## References

- Laravel Eloquent Documentation — https://laravel.com/docs/12.x/eloquent
- Laravel 13.x Eloquent Documentation — https://laravel.com/framework/docs/13.x/eloquent
- Laravel API: GuardsAttributes Trait — https://api.laravel.com/docs/9.x/Illuminate/Database/Eloquent/Concerns/GuardsAttributes.html
- Laravel Database Seeding — https://laravel.com/docs/12.x/seeding
- Laravel Eloquent: Table Names — https://laravel.com/docs/12.x/eloquent#table-names
- Laravel Eloquent: Primary Keys — https://laravel.com/docs/12.x/eloquent#primary-keys
- Laravel Eloquent: Timestamps — https://laravel.com/docs/12.x/eloquent#timestamps
- Laravel Eloquent: Database Connections — https://laravel.com/docs/12.x/eloquent#database-connections
- Laravel Eloquent: Mass Assignment — https://laravel.com/docs/12.x/eloquent#mass-assignment
- Laravel Eloquent: Checking Attribute Changes — https://laravel.com/docs/12.x/eloquent#examining-attribute-changes
- Laravel Model Events — https://laravel.com/docs/12.x/eloquent#events
- Active Record Pattern (Wikipedia) — https://en.wikipedia.org/wiki/Active_record_pattern
- Eloquent Model System DeepWiki — https://deepwiki.com/laravel/framework/3-eloquent-model-system
- Laravel 8.x Model Conventions (PDF) — https://github.com/DevStorm-Team/laravel-book/blob/master/laravel-docs-8.x.pdf