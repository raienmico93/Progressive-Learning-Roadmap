# Laravel Eloquent CRUD: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Laravel Eloquent CRUD refers to the set of object-oriented methods provided by Eloquent, Laravel's Active Record implementation, for Creating, Reading, Updating, and Deleting records in a database, along with Soft Deletion for reversible data removal.

**Technical Definition:** Eloquent's CRUD operations are implemented through the `Illuminate\Database\Eloquent\Model` class and its query builder. Methods such as `create()`, `save()`, `find()`, `update()`, `delete()`, and the `SoftDeletes` trait provide a fluent, object-oriented interface over SQL INSERT, SELECT, UPDATE, and DELETE statements. Each model instance corresponds to a row in the database and carries both the data and the persistence logic for that row.

**Beginner-Friendly Explanation:** Eloquent lets you treat database rows as PHP objects. Instead of writing SQL queries, you create objects, set their properties, and call methods like `save()` or `delete()`. Soft deletion adds a "trash" state—records are marked as deleted but not removed, so you can restore them later.

### Key Characteristics

- **Active Record Pattern:** Each model instance wraps a database row and knows how to persist itself.
- **Fluent Query Builder:** Eloquent models double as query builders, allowing chained constraints before retrieval.
- **Automatic Timestamps:** `created_at` and `updated_at` columns are managed automatically.
- **Mass Assignment Protection:** `$fillable` and `$guarded` control which attributes can be set in bulk.
- **Reversible Deletion:** The `SoftDeletes` trait marks records as deleted without removing them.
- **Event Lifecycle:** Models fire events (`creating`, `created`, `updating`, `updated`, `deleting`, `deleted`) at each persistence stage.

### Prerequisites

- A configured database connection in `config/database.php` and `.env`.
- At least one Eloquent model class extending `Illuminate\Database\Eloquent\Model`.
- Corresponding database tables created via migrations.
- For soft deletes: the `SoftDeletes` trait and a `deleted_at` column in the table.

### Related Programming Areas

- **Database Migrations:** Define the schema that models map to.
- **Query Builder:** Eloquent models extend the query builder, inheriting all its methods.
- **Model Events & Observers:** Hook into the CRUD lifecycle for custom logic.
- **API Resources:** Transform Eloquent models into JSON responses.
- **Task Scheduling:** Pruning and cleanup of stale records via Artisan commands.

### Core Concepts / Features

1. Creating Records (`save()`, `create()`, `insert()`)
2. Reading Records (`all()`, `find()`, `first()`, `get()`, `findOrFail()`)
3. Updating Records (`save()`, `update()`, `increment()`, `decrement()`)
4. Deleting Records (`delete()`, `destroy()`, mass deletes)
5. Soft Deletion (`SoftDeletes` trait, `withTrashed()`, `onlyTrashed()`, `forceDelete()`)

---

## 1. Creating Records

### Definitions

**Core Definition:** Creating records in Eloquent involves instantiating a new model, setting its attributes, and persisting it to the database as a new row.

**Technical Definition:** The `Model::save()` method determines whether to perform an `INSERT` or `UPDATE` based on the model's `exists` property. `Model::create()` is a static shortcut that instantiates a model, fills it with mass-assignable data, and saves it in one call. `Model::insert()` performs a raw bulk insert without instantiating models.

**Beginner-Friendly Explanation:** You can create a record by making a new object, setting its properties, and calling `save()`, or use `create()` to do it in one line. For bulk inserts, `insert()` is fastest but skips model events and timestamps.

### Purposes

- To insert new rows into the database using an object-oriented interface.
- To automatically set `created_at` and `updated_at` timestamps on new records.
- To leverage mass assignment protection when creating records from user input.
- To perform high-performance bulk inserts for large datasets using `insert()`.
- To fire model events (`creating`, `created`) for custom logic during creation.

### Syntax Rules and Structure

**Complete General Syntax — `save()`:**

```php
$model = new Model();
$model->attribute = 'value';
$model->save();
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `new Model()` | Instantiates a new model; no database row exists yet. |
| `$model->attribute = 'value'` | Sets an attribute directly (bypasses mass assignment). |
| `$model->save()` | Persists the model; issues `INSERT` if new, `UPDATE` if existing. |

**Complete General Syntax — `create()`:**

```php
$model = Model::create([
    'attribute1' => 'value1',
    'attribute2' => 'value2',
]);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Model::create([...])` | Static method that fills and saves a new model in one call. |
| `['attribute1' => ...]` | Array of fillable attributes; non-fillable keys are silently discarded. |

**Complete General Syntax — `insert()` (Bulk):**

```php
Model::insert([
    ['attribute1' => 'value1', 'attribute2' => 'value2'],
    ['attribute1' => 'value3', 'attribute2' => 'value4'],
]);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Model::insert([...])` | Raw bulk insert; does not fire events or set timestamps. |
| Nested arrays | Each inner array represents a row to insert. |

**Syntax Rules:**

- `save()` works with any attribute, regardless of fillable status.
- `create()` requires `$fillable` or `$guarded` to be configured for mass assignment.
- `insert()` does **not** set timestamps or fire model events.
- Timestamps are automatically set by `save()` and `create()` unless `$timestamps = false`.

**Constraints and Limitations:**

- **Mass assignment:** `create()` silently discards non-fillable attributes unless strict mode is enabled.
- **Bulk inserts:** `insert()` bypasses Eloquent entirely, so no events, no timestamps, no model instances.
- **Return value:** `save()` returns `true`/`false`; `create()` returns the model instance.

### Multiple Annotated Complete Code Examples

**Example 1: Creating with `save()`**

```php
<?php
// app/Models/Flight.php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    protected $fillable = ['name', 'destination', 'price'];
}
```

```php
<?php
// Create a new flight instance
$flight = new Flight();

// Set attributes individually
$flight->name = 'Tokyo';
$flight->destination = 'Osaka';
$flight->price = 150;

// Persist to database — INSERT INTO flights ...
$flight->save();

// After save(), the model has its primary key populated
echo $flight->id; // e.g., 1
```

**Expected Output:** A new row is inserted into the `flights` table with the specified values. `created_at` and `updated_at` are set automatically. `$flight->id` returns the auto-incremented primary key.

**Why:** The `save()` method checks the `exists` property. Since this is a new model, `exists` is `false`, so Eloquent issues an `INSERT`. After insertion, the model's `id` and timestamps are populated.

---

**Example 2: Creating with `create()`**

```php
<?php
use App\Models\Flight;

// One-line creation with mass assignment
$flight = Flight::create([
    'name' => 'London to Paris',
    'destination' => 'Paris',
    'price' => 200,
]);

echo $flight->id; // e.g., 2
```

**Expected Output:** A new row is inserted. The `Flight` instance is returned with its `id` populated.

**Why:** `create()` instantiates a new model, calls `fill()` with the provided array (respecting mass assignment), then calls `save()`.

---

**Example 3: Bulk Insert with `insert()`**

```php
<?php
use App\Models\Flight;

Flight::insert([
    ['name' => 'NY to LA', 'destination' => 'Los Angeles', 'price' => 300],
    ['name' => 'Chicago to Miami', 'destination' => 'Miami', 'price' => 250],
]);
```

**Expected Output:** Two rows are inserted in a single SQL statement. No timestamps are set, and no model events are fired.

**Why:** `insert()` generates a raw `INSERT INTO ... VALUES (...), (...)` query, bypassing Eloquent's model layer entirely for performance.

### Real-World Cases

- **User Registration:** `User::create($request->validated())` creates a new user from form data.
- **API Endpoints:** `POST /api/posts` uses `Post::create()` to store a new blog post.
- **Data Import:** `insert()` is used to bulk-import thousands of records from a CSV file.
- **Seeders:** `Model::factory()->create()` is the standard way to generate test data.
- **Logging:** `Log::create(['message' => '...', 'level' => 'info'])` records application events.

### References

- Laravel Eloquent: Inserting & Updating Models — https://laravel.com/docs/12.x/eloquent#inserting-and-updating-models
- Laravel Eloquent: Getting Started — https://laravel.com/docs/12.x/eloquent
- Laravel API: Model::create() — https://laravel.com/api/12.x/Illuminate/Database/Eloquent/Model.html#method_create

---

## 2. Reading Records

### Definitions

**Core Definition:** Reading records in Eloquent involves retrieving one or more model instances from the database using static methods and query constraints.

**Technical Definition:** Eloquent provides `all()` to retrieve every record, `find()` to retrieve by primary key, `first()` to retrieve the first matching record, `get()` to retrieve a collection, and `findOrFail()`/`firstOrFail()` to throw a `ModelNotFoundException` when no record is found. Models act as query builders, so any query builder method can be chained before `get()` or `first()`.

**Beginner-Friendly Explanation:** You can ask Eloquent for all records, a single record by ID, or the first record matching a condition. Fail-safe methods like `findOrFail()` throw a 404 error automatically if nothing is found, which is handy in controllers.

### Purposes

- To retrieve all records from a table using `all()` or `get()`.
- To retrieve a single record by primary key using `find()` or `findOrFail()`.
- To retrieve the first record matching query constraints using `first()` or `firstOrFail()`.
- To apply fluent query constraints (where, orderBy, limit) before retrieval.
- To throw exceptions for missing records, enabling automatic 404 responses in web routes.
- To retrieve records in memory-efficient chunks for large datasets using `chunk()` or `cursor()`.

### Syntax Rules and Structure

**Complete General Syntax — `all()` / `get()`:**

```php
$models = Model::all();
$models = Model::where('active', 1)->orderBy('name')->get();
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Model::all()` | Retrieves all records from the table. |
| `->where(...)->get()` | Applies constraints and returns a Collection. |

**Complete General Syntax — `find()` / `findOrFail()`:**

```php
$model = Model::find($id);
$model = Model::findOrFail($id);
$models = Model::find([1, 2, 3]); // Collection
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `find($id)` | Returns a single model or `null`. |
| `findOrFail($id)` | Returns a model or throws `ModelNotFoundException`. |
| `find([1, 2, 3])` | Returns a Collection of matching models. |

**Complete General Syntax — `first()` / `firstOrFail()`:**

```php
$model = Model::where('active', 1)->first();
$model = Model::where('active', 1)->firstOrFail();
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `->first()` | Returns the first matching model or `null`. |
| `->firstOrFail()` | Returns the first matching model or throws `ModelNotFoundException`. |

**Syntax Rules:**

- `find()` accepts a single primary key or an array of keys.
- `first()` returns a single model; `get()` returns a Collection.
- `findOrFail()` and `firstOrFail()` throw `ModelNotFoundException` when no record is found; Laravel automatically converts this to a 404 HTTP response in web contexts.
- All query builder methods (`where`, `orderBy`, `limit`, `join`) can be chained before `get()` or `first()`.

**Constraints and Limitations:**

- **N+1 query problem:** Accessing relationships in a loop without `with()` causes one query per iteration.
- **Memory usage:** `all()` and `get()` load all records into memory; use `chunk()` or `cursor()` for large datasets.
- **`find()` returns `null`:** Always check for `null` or use `findOrFail()` in controllers.

### Multiple Annotated Complete Code Examples

**Example 1: Retrieving All Records**

```php
<?php
use App\Models\Flight;

// Retrieve all flights
foreach (Flight::all() as $flight) {
    echo $flight->name . "\n";
}
```

**Expected Output:** All flight names are printed, one per line.

**Why:** `all()` executes `SELECT * FROM flights` and returns an Eloquent Collection of `Flight` models.

---

**Example 2: Finding by Primary Key**

```php
<?php
use App\Models\Flight;

// Find by ID — returns model or null
$flight = Flight::find(1);
if ($flight) {
    echo $flight->name;
}

// Find or throw 404
$flight = Flight::findOrFail(1);
echo $flight->name; // If not found, 404 response
```

**Expected Output:** If flight 1 exists, its name is printed. `findOrFail(999)` throws `ModelNotFoundException` → 404.

**Why:** `find()` returns `null` when no record matches. `findOrFail()` throws an exception that Laravel's handler converts to a 404 response.

---

**Example 3: Query Constraints with `get()`**

```php
<?php
use App\Models\Flight;

$flights = Flight::where('active', 1)
    ->orderBy('name')
    ->limit(10)
    ->get();

foreach ($flights as $flight) {
    echo $flight->name;
}
```

**Expected Output:** The names of up to 10 active flights, ordered by name.

**Why:** Eloquent models act as query builders. The `where`, `orderBy`, and `limit` constraints are compiled into SQL before `get()` executes the query.

### Real-World Cases

- **Blog Index:** `Post::latest()->paginate(10)` retrieves paginated blog posts.
- **User Profile:** `User::findOrFail($id)` in a controller `show` method returns 404 for missing users.
- **API Endpoints:** `Product::where('category', $slug)->get()` returns products for a category.
- **Dashboard Statistics:** `Order::where('status', 'pending')->count()` retrieves aggregate data.
- **Data Export:** `User::chunk(200, fn($users) => ...)` processes large datasets in memory-efficient batches.

### References

- Laravel Eloquent: Retrieving Models — https://laravel.com/docs/12.x/eloquent#retrieving-models
- Laravel Eloquent: Retrieving Single Models — https://laravel.com/docs/12.x/eloquent#retrieving-single-models
- Laravel Eloquent: Not Found Exceptions — https://laravel.com/docs/12.x/eloquent#not-found-exceptions

---

## 3. Updating Records

### Definitions

**Core Definition:** Updating records in Eloquent involves modifying a model's attributes and persisting the changes to the database.

**Technical Definition:** The `save()` method detects dirty attributes (those changed since retrieval) and issues an `UPDATE` statement for only the modified columns. `update()` fills the model with an array and calls `save()`. `increment()` and `decrement()` execute atomic SQL updates that adjust numeric columns directly.

**Beginner-Friendly Explanation:** You can retrieve a record, change a property, and call `save()`, or use `update()` with an array. For counters (views, stock), `increment()` and `decrement()` adjust values safely without loading the whole row.

### Purposes

- To modify existing records by setting attributes and calling `save()`.
- To perform bulk updates on multiple records using `update()`.
- To atomically increment or decrement numeric columns without race conditions.
- To automatically refresh the `updated_at` timestamp on modification.
- To leverage mass assignment protection when updating from user input.
- To perform updates without firing model events using `updateQuietly()`.

### Syntax Rules and Structure

**Complete General Syntax — `save()`:**

```php
$model = Model::find(1);
$model->attribute = 'new_value';
$model->save();
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `$model->attribute = 'new_value'` | Marks the attribute as dirty. |
| `$model->save()` | Issues `UPDATE` for dirty columns only. |

**Complete General Syntax — `update()`:**

```php
$model = Model::find(1);
$model->update(['attribute1' => 'new', 'attribute2' => 'newer']);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `$model->update([...])` | Fills the model with the array and saves. |
| `['attribute1' => ...]` | Attributes must be fillable; non-fillable keys are discarded. |

**Complete General Syntax — `increment()` / `decrement()`:**

```php
$model = Model::find(1);
$model->increment('views');       // +1
$model->increment('views', 10);   // +10
$model->decrement('stock');       // -1
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `increment('column')` | Atomically adds 1 to the column. |
| `increment('column', $amount)` | Atomically adds `$amount`. |
| `decrement('column')` | Atomically subtracts 1. |

**Syntax Rules:**

- `save()` updates only dirty attributes (those changed since retrieval).
- `update()` requires mass assignment protection for the array keys.
- `increment()` and `decrement()` do **not** fire model events and do not update the in-memory model attribute.
- `updateQuietly()` performs an update without firing events.

**Constraints and Limitations:**

- **Mass assignment:** `update()` silently discards non-fillable attributes unless strict mode is enabled.
- **Atomic operations:** `increment()` and `decrement()` bypass events and do not refresh the model; call `refresh()` to see the new value.
- **Bulk updates:** `Model::where(...)->update([...])` performs a mass update without retrieving models and without firing events.

### Multiple Annotated Complete Code Examples

**Example 1: Updating with `save()`**

```php
<?php
use App\Models\Flight;

$flight = Flight::find(1);
$flight->price = 250;
$flight->save(); // UPDATE flights SET price = 250, updated_at = ... WHERE id = 1
```

**Expected Output:** The row with `id = 1` has its `price` updated to `250`. `updated_at` is refreshed.

**Why:** Changing the `price` attribute marks it as dirty. `save()` detects the dirty attribute and issues an `UPDATE` for only that column, along with the timestamp.

---

**Example 2: Updating with `update()`**

```php
<?php
use App\Models\Flight;

$flight = Flight::find(1);
$flight->update([
    'price' => 300,
    'destination' => 'Kyoto',
]);
```

**Expected Output:** The `price` and `destination` columns are updated in a single `UPDATE` statement.

**Why:** `update()` calls `fill()` with the array (respecting mass assignment) and then calls `save()`. The `updated_at` timestamp is automatically refreshed.

---

**Example 3: Atomic Increment**

```php
<?php
use App\Models\Post;

$post = Post::find(1);
$post->increment('views');       // views = views + 1
$post->increment('views', 10);   // views = views + 10

// Refresh to see the new value in memory
$post->refresh();
echo $post->views; // 11 (if starting from 0)
```

**Expected Output:** The `views` column increases by 11 total. `refresh()` reloads the model to reflect the atomic change.

**Why:** `increment()` generates `UPDATE posts SET views = views + 1 WHERE id = 1`, avoiding the read-modify-write race condition. It does not update the in-memory attribute.

### Real-World Cases

- **User Profiles:** `$user->update(['name' => $request->name])` updates user details.
- **Blog Analytics:** `$post->increment('views')` on every page visit.
- **Inventory:** `$product->decrement('stock', $quantity)` when an order is placed.
- **Settings:** `$user->update(['theme' => 'dark'])` saves a preference change.
- **Batch Status Updates:** `Order::where('status', 'pending')->update(['status' => 'processing'])` processes all pending orders.

### References

- Laravel Eloquent: Updating Models — https://laravel.com/docs/12.x/eloquent#updates
- Laravel Eloquent: Increment & Decrement — https://laravel.com/docs/12.x/queries#increment-and-decrement
- Laravel Eloquent: Mass Updates — https://laravel.com/docs/12.x/eloquent#mass-updates

---

## 4. Deleting Records

### Definitions

**Core Definition:** Deleting records in Eloquent removes a row from the database either by calling `delete()` on a model instance or `destroy()` by primary key.

**Technical Definition:** `Model::delete()` issues a `DELETE` statement for the model's row. `Model::destroy()` accepts primary keys and loads each model individually, firing `deleting` and `deleted` events. Mass deletes via query builder (`Model::where(...)->delete()`) do **not** retrieve models or fire events.

**Beginner-Friendly Explanation:** You can delete a record by retrieving it and calling `delete()`, or delete by ID using `destroy()`. For bulk deletion, use a query—just be aware that events won't fire.

### Purposes

- To remove a single record by calling `delete()` on a model instance.
- To delete one or more records by primary key using `destroy()`.
- To perform bulk deletions matching query constraints using `where(...)->delete()`.
- To fire `deleting` and `deleted` events for custom logic during deletion.
- To permanently remove soft-deleted records using `forceDelete()`.

### Syntax Rules and Structure

**Complete General Syntax — `delete()`:**

```php
$model = Model::find(1);
$model->delete(); // DELETE FROM models WHERE id = 1
```

**Complete General Syntax — `destroy()`:**

```php
Model::destroy(1);
Model::destroy(1, 2, 3);
Model::destroy([1, 2, 3]);
Model::destroy(collect([1, 2, 3]));
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `destroy($id)` | Deletes by primary key; loads each model individually. |
| `destroy([...])` | Accepts an array or Collection of primary keys. |
| Events | Fires `deleting` and `deleted` for each model. |

**Complete General Syntax — Mass Delete:**

```php
$deleted = Model::where('active', 0)->delete();
$deleted = Model::query()->delete(); // Delete all
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `where(...)->delete()` | Deletes all matching records in one query. |
| `Model::query()->delete()` | Deletes all records in the table. |
| Events | `deleting` and `deleted` events are **not** dispatched. |

**Syntax Rules:**

- `delete()` on an instance requires the model to exist.
- `destroy()` loads each model individually so events fire properly.
- Mass deletes via query builder are faster but skip events.
- `forceDelete()` permanently removes soft-deleted records.

**Constraints and Limitations:**

- **Mass delete events:** `Model::where(...)->delete()` does not fire model events because models are never retrieved.
- **Soft deletes:** If the model uses `SoftDeletes`, `delete()` sets `deleted_at` instead of removing the row.
- **`destroy()` performance:** Loads each model individually; for large datasets, mass delete is more efficient.

### Multiple Annotated Complete Code Examples

**Example 1: Deleting a Single Model**

```php
<?php
use App\Models\Flight;

$flight = Flight::find(1);
$flight->delete(); // DELETE FROM flights WHERE id = 1
```

**Expected Output:** The row with `id = 1` is removed. The `deleting` and `deleted` events are fired.

**Why:** `delete()` issues a `DELETE` statement for the specific row. Since the model was retrieved, events are dispatched.

---

**Example 2: Destroying by Primary Key**

```php
<?php
use App\Models\Flight;

Flight::destroy(1);              // Delete ID 1
Flight::destroy(1, 2, 3);        // Delete IDs 1, 2, 3
Flight::destroy([4, 5]);         // Delete IDs 4, 5
Flight::destroy(collect([6, 7])); // Delete IDs 6, 7
```

**Expected Output:** All specified records are deleted. Events fire for each model.

**Why:** `destroy()` loads each model individually and calls `delete()`, ensuring events are dispatched for each deletion.

---

**Example 3: Mass Delete with Query**

```php
<?php
use App\Models\Flight;

// Delete all inactive flights
$deleted = Flight::where('active', 0)->delete();
echo $deleted; // Number of rows deleted

// Delete all flights (danger!)
$deleted = Flight::query()->delete();
```

**Expected Output:** All matching rows are deleted in a single query. No model events are fired.

**Why:** The query builder generates a direct `DELETE FROM flights WHERE active = 0` statement, bypassing model retrieval and event dispatch.

### Real-World Cases

- **User Account Deletion:** `$user->delete()` removes a user and fires events for cleanup.
- **Admin Bulk Cleanup:** `Post::where('spam', true)->delete()` removes all spam posts at once.
- **Cascade Deletion:** `destroy()` is used when related records must be cleaned up via events.
- **API `DELETE` Endpoint:** `Post::destroy($id)` in a controller `destroy` method.
- **Testing:** `Model::truncate()` (not Eloquent) is used to reset tables between tests.

### References

- Laravel Eloquent: Deleting Models — https://laravel.com/docs/12.x/eloquent#deleting-models
- Laravel Eloquent: Deleting by Primary Key — https://laravel.com/docs/12.x/eloquent#deleting-an-existing-model-by-its-primary-key
- Laravel Eloquent: Mass Deletes — https://laravel.com/docs/12.x/eloquent#deleting-models-using-queries

---

## 5. Soft Deletion

### Definitions

**Core Definition:** Soft deletion marks records as deleted by setting a `deleted_at` timestamp instead of physically removing them from the database.

**Technical Definition:** The `Illuminate\Database\Eloquent\SoftDeletes` trait adds a global scope that excludes records with a non-null `deleted_at` column from all queries. The `delete()` method sets `deleted_at` to the current timestamp. `withTrashed()` includes soft-deleted records, `onlyTrashed()` retrieves only soft-deleted records, `restore()` sets `deleted_at` to `null`, and `forceDelete()` permanently removes the row.

**Beginner-Friendly Explanation:** Soft deletes are like moving a file to the trash. The record is still in the database but hidden from normal queries. You can restore it or permanently delete it later. This is useful for audit trails and accidental deletion recovery.

### Purposes

- To preserve deleted records for audit trails and recovery purposes.
- To prevent accidental data loss by providing a reversible "trash" state.
- To maintain referential integrity in relationships.
- To allow administrators to view and restore deleted records.
- To support compliance requirements (e.g., GDPR retention periods).
- To separate logical deletion from physical data removal.

### Syntax Rules and Structure

**Complete General Syntax — Enabling Soft Deletes:**

```php
<?php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class Post extends Model
{
    use SoftDeletes;
}
```

**Migration:**

```php
Schema::table('posts', function (Blueprint $table) {
    $table->softDeletes(); // Adds nullable deleted_at TIMESTAMP
});
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `use SoftDeletes` | Trait that adds soft delete functionality. |
| `$table->softDeletes()` | Migration macro adding the `deleted_at` column. |

**Complete General Syntax — Querying Trashed Records:**

```php
$all = Post::withTrashed()->get();       // Include trashed
$only = Post::onlyTrashed()->get();      // Only trashed
$isTrashed = $post->trashed();           // true/false
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `withTrashed()` | Includes soft-deleted records in the query. |
| `onlyTrashed()` | Retrieves only soft-deleted records. |
| `trashed()` | Instance method; returns `true` if soft-deleted. |

**Complete General Syntax — Restoring and Force Deleting:**

```php
$post->restore();              // Sets deleted_at to null
$post->forceDelete();          // DELETE FROM posts WHERE id = 1

Post::withTrashed()->where('user_id', 5)->restore();
Post::onlyTrashed()->where('deleted_at', '<', now()->subYear())->forceDelete();
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `restore()` | Sets `deleted_at` back to `null`. |
| `forceDelete()` | Permanently removes the record. |
| Bulk restore/force delete | Applies to all matching soft-deleted records. |

**Syntax Rules:**

- The model must use the `SoftDeletes` trait.
- The table must have a `deleted_at` column (nullable timestamp).
- Default queries automatically exclude soft-deleted records.
- `withTrashed()` and `onlyTrashed()` can be chained onto any query.
- `forceDelete()` works on both soft-deleted and non-deleted models.

**Constraints and Limitations:**

- **Unique constraints:** Soft-deleted records still occupy unique indexes.
- **Cascading:** Soft deletes do not automatically cascade unless configured.
- **Pruning:** Soft-deleted models are permanently deleted if they match a prunable query.
- **Performance:** The `deleted_at` column should be indexed for efficient querying.

### Multiple Annotated Complete Code Examples

**Example 1: Basic Soft Delete and Restore**

```php
<?php
use App\Models\Post;

$post = Post::find(1);
$post->delete(); // UPDATE posts SET deleted_at = NOW() WHERE id = 1

// Default query excludes it
$posts = Post::all(); // Does not include post 1

// Include trashed records
$allPosts = Post::withTrashed()->get(); // Includes post 1

// Restore the post
$post = Post::withTrashed()->find(1);
$post->restore(); // UPDATE posts SET deleted_at = NULL WHERE id = 1
```

**Expected Output:** After `delete()`, the post is excluded from normal queries. `withTrashed()` includes it. `restore()` brings it back.

**Why:** The `SoftDeletes` trait overrides `delete()` to set `deleted_at` instead of removing the row. A global scope adds `WHERE deleted_at IS NULL` to all queries.

---

**Example 2: `onlyTrashed()` and `trashed()`**

```php
<?php
use App\Models\Post;

$trashedPosts = Post::onlyTrashed()->get();

foreach ($trashedPosts as $post) {
    echo "Trashed: {$post->title} (deleted at {$post->deleted_at})\n";
}

$post = Post::withTrashed()->find(1);
if ($post->trashed()) {
    echo "This post is in the trash.";
}
```

**Expected Output:** Lists all soft-deleted posts with their deletion timestamps. `trashed()` returns `true` for soft-deleted models.

**Why:** `onlyTrashed()` adds `WHERE deleted_at IS NOT NULL`. The `trashed()` method checks whether `deleted_at` is non-null.

---

**Example 3: Force Delete Permanently**

```php
<?php
use App\Models\Post;

// Permanently delete a soft-deleted post
$post = Post::withTrashed()->find(1);
$post->forceDelete(); // DELETE FROM posts WHERE id = 1

// Bulk force delete of all trashed posts older than 1 year
Post::onlyTrashed()
    ->where('deleted_at', '<', now()->subYear())
    ->forceDelete();
```

**Expected Output:** The post is permanently removed. Bulk force delete removes all matching records in a single `DELETE` statement.

**Why:** `forceDelete()` bypasses the soft delete mechanism and issues a real `DELETE`. It works on individual models or as a bulk query.

### Real-World Cases

- **Content Management:** Blog posts and pages are soft-deleted so editors can recover them from the trash.
- **User Accounts:** Soft-deleted accounts retain data for a grace period before permanent deletion.
- **E-commerce Orders:** Soft-deleted orders preserve transaction history for accounting.
- **Audit Trails:** Soft deletes provide a record of what was deleted and when.
- **GDPR Compliance:** Soft-deleted personal data is permanently removed after a retention period using `forceDelete()`.

### References

- Laravel Eloquent: Soft Deleting — https://laravel.com/docs/12.x/eloquent#soft-deleting
- Laravel API: SoftDeletes Trait — https://api.laravel.com/docs/12.x/Illuminate/Database/Eloquent/SoftDeletes.html
- Laravel Migrations: Soft Deletes Column — https://laravel.com/docs/12.x/migrations#column-method-softDeletes

---

## References

- Laravel Eloquent: Getting Started — https://laravel.com/docs/12.x/eloquent
- Laravel Eloquent: Inserting & Updating Models — https://laravel.com/docs/12.x/eloquent#inserting-and-updating-models
- Laravel Eloquent: Retrieving Models — https://laravel.com/docs/12.x/eloquent#retrieving-models
- Laravel Eloquent: Deleting Models — https://laravel.com/docs/12.x/eloquent#deleting-models
- Laravel Eloquent: Soft Deleting — https://laravel.com/docs/12.x/eloquent#soft-deleting
- Laravel Eloquent: Pruning Models — https://laravel.com/docs/12.x/eloquent#pruning-models
- Laravel Query Builder: Increment & Decrement — https://laravel.com/docs/12.x/queries#increment-and-decrement
- Laravel API: SoftDeletes Trait — https://api.laravel.com/docs/12.x/Illuminate/Database/Eloquent/SoftDeletes.html
- Laravel API: Model::create() — https://laravel.com/api/12.x/Illuminate/Database/Eloquent/Model.html#method_create
- Laravel API: Model::update() — https://laravel.com/api/12.x/Illuminate/Database/Eloquent/Model.html#method_update
- Laravel API: Model::delete() — https://laravel.com/api/12.x/Illuminate/Database/Eloquent/Model.html#method_delete