# Laravel Eloquent Relationships (The 9 Core Types): A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Eloquent relationships are methods defined on model classes that describe how database tables are connected to one another, enabling object-oriented navigation between related records without writing manual JOIN queries.

**Technical Definition:** Eloquent relationships are implemented through the `HasRelationships` trait (in `Illuminate\Database\Eloquent\Concerns`), which provides methods such as `hasOne()`, `hasMany()`, `belongsTo()`, `belongsToMany()`, `hasManyThrough()`, `morphOne()`, `morphMany()`, and `morphToMany()`. Each relationship method returns a `Relation` subclass instance (`HasOne`, `HasMany`, `BelongsTo`, `BelongsToMany`, etc.) that extends the query builder and defines the foreign key conventions, local key constraints, and eager-loading behaviour. The `Relation` class also provides morph map registration via `morphMap()` and `enforceMorphMap()` for decoupling stored type strings from class names.

**Beginner-Friendly Explanation:** Think of your database tables as houses on a street. Some houses are connected by a single road (one-to-one), some have many driveways leading to them (one-to-many), and some share a communal parking lot (many-to-many). Eloquent relationships are the road signs and maps that tell Laravel how to travel from one house to another. Instead of writing `SELECT * FROM phones WHERE user_id = 1`, you simply write `$user->phone`—Eloquent figures out the connection for you.

### Key Characteristics

- **Method-Based Definition:** Relationships are defined as public methods on model classes, returning typed `Relation` instances.
- **Convention over Configuration:** Foreign key names (e.g., `user_id`), pivot table names (e.g., `role_user`), and local keys are inferred automatically.
- **Query Builder Integration:** Relationship methods return query builders, enabling chaining (e.g., `$user->posts()->where('active', 1)->get()`).
- **Dynamic Properties:** Accessed as properties (e.g., `$user->phone`), Eloquent resolves the relationship lazily or uses eager-loaded results.
- **Eager Loading:** The `with()` method preloads relationships to prevent the N+1 query problem.
- **Polymorphic Support:** Models can belong to multiple parent types using morph columns (`*_type` and `*_id`).
- **Morph Maps:** Optional registration of aliases (`enforceMorphMap`) decouples stored strings from class names.

### Prerequisites

- A working Laravel application with a configured database connection.
- At least two Eloquent models and their corresponding migrations.
- Understanding of foreign key columns and database table structure.
- For polymorphic relationships: `*_type` and `*_id` columns (via `$table->morphs()`).
- For many-to-many: a pivot table with foreign key columns for both related models.

### Related Programming Areas

- **Database Migrations:** Define foreign keys, pivot tables, and morph columns.
- **Eloquent Query Builder:** All relationship methods extend the query builder.
- **Eager Loading:** The `with()` method integrates with all relationship types.
- **Route Model Binding:** Implicit binding can resolve related models from route parameters.
- **API Resources:** Relationships are serialised into JSON responses via API resources.

### Core Concepts / Features

1. One-to-One / One-to-Many (`hasOne()`, `belongsTo()`, `hasMany()`)
2. Many-to-Many (`belongsToMany()`, Pivot Tables, Custom Pivot Models via `using()`)
3. Advanced Through Relationships (`hasManyThrough()`, `hasOneThrough()`)
4. Polymorphic Relationships (One-to-One, One-to-Many, Many-to-Many)
5. Enforcing Strict Morph Types (`Relation::enforceMorphMap()`)

---

## 1. One-to-One / One-to-Many Relationships

### Definitions

**Core Definition:** A one-to-one relationship connects a single record in one table to exactly one record in another table. A one-to-many relationship connects a single record to multiple records in another table.

**Technical Definition:** `hasOne()` defines a one-to-one relationship where the foreign key resides on the **related** (child) model's table. `belongsTo()` defines the inverse of both `hasOne()` and `hasMany()`, where the foreign key resides on the **current** model's table. `hasMany()` defines a one-to-many relationship where the foreign key resides on the related model's table, allowing multiple child records per parent.

**Beginner-Friendly Explanation:** A user has one phone (one-to-one). A user has many posts (one-to-many). The "foreign key" is like a label on the child record saying "I belong to user #1." `hasOne()` and `hasMany()` are declared on the parent (User) side; `belongsTo()` is declared on the child (Phone/Post) side to point back to the parent.

### Purposes

- To retrieve a single related record (e.g., a user's profile or phone) from the parent model.
- To retrieve all related records (e.g., all posts by a user) from the parent model.
- To access the parent model from the child model using the inverse relationship.
- To enable chained query constraints on related records (e.g., only active posts).
- To support eager loading with `with()` to prevent the N+1 query problem.
- To automatically infer foreign key names and local key names from model class names.

### Syntax Rules and Structure

**Complete General Syntax — `hasOne()`:**

```php
public function relationshipName(): HasOne
{
    return $this->hasOne(RelatedModel::class, 'foreign_key', 'local_key');
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `$this->hasOne(RelatedModel::class)` | Defines a one-to-one relationship. |
| `'foreign_key'` | Optional. The foreign key on the related model's table. Default: `{parent_snake}_id`. |
| `'local_key'` | Optional. The local key on the parent model. Default: `id`. |

**Complete General Syntax — `belongsTo()`:**

```php
public function relationshipName(): BelongsTo
{
    return $this->belongsTo(ParentModel::class, 'foreign_key', 'owner_key');
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `$this->belongsTo(ParentModel::class)` | Defines the inverse one-to-one or one-to-many relationship. |
| `'foreign_key'` | Optional. The foreign key on the **current** model's table. |
| `'owner_key'` | Optional. The key on the parent model. Default: `id`. |

**Complete General Syntax — `hasMany()`:**

```php
public function relationshipName(): HasMany
{
    return $this->hasMany(RelatedModel::class, 'foreign_key', 'local_key');
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `$this->hasMany(RelatedModel::class)` | Defines a one-to-many relationship. |
| `'foreign_key'` | Optional. The foreign key on the related model's table. |
| `'local_key'` | Optional. The local key on the parent model. Default: `id`. |

**Syntax Rules:**

- Relationship methods must be **public** and return a `Relation` type-hint.
- The foreign key convention for `hasOne`/`hasMany` is `{parent_model_snake_case}_id` (e.g., `user_id` for a `User` model).
- The foreign key convention for `belongsTo` is derived from the **method name**: the method `phone()` on `Phone` model would look for `phone_id` by default. Override by passing the foreign key as the second argument.
- Dynamic property access (`$user->phone`) returns the result; method access (`$user->phone()->where(...)`) returns a query builder.

**Constraints and Limitations:**

- **N+1 query problem:** Accessing relationships in a loop without `with()` causes one query per iteration. Always use eager loading for collections.
- **Method vs. property:** `$user->phone` returns the model (or `null`); `$user->phone()` returns the relationship query builder.
- **Foreign key nullability:** `hasOne`/`hasMany` return `null` or an empty collection if no related records exist; `belongsTo` returns `null` if the foreign key is `null` or points to a non-existent record.
- **Composite keys:** Eloquent relationships do not support composite foreign keys.

### Multiple Annotated Complete Code Examples

**Example 1: One-to-One (User hasOne Phone)**

```php
<?php
// app/Models/User.php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasOne;

class User extends Model
{
    /**
     * Get the phone associated with the user.
     */
    public function phone(): HasOne
    {
        return $this->hasOne(Phone::class);
    }
}
```

```php
<?php
// app/Models/Phone.php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Phone extends Model
{
    /**
     * Get the user that owns the phone.
     */
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }
}
```

```php
// Usage
$phone = User::find(1)->phone; // Returns Phone model or null
$user = Phone::find(1)->user;  // Returns User model or null
```

**Expected Output:** For a user with `id = 1` and a phone with `user_id = 1`, `$user->phone` returns the `Phone` instance. `$phone->user` returns the `User` instance.

**Why:** Eloquent infers the foreign key `user_id` on the `phones` table from the `User` model name. `belongsTo` infers `user_id` from the method name `user()`.

---

**Example 2: One-to-Many (User hasMany Posts)**

```php
<?php
// app/Models/User.php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasMany;

class User extends Model
{
    /**
     * Get all of the posts for the user.
     */
    public function posts(): HasMany
    {
        return $this->hasMany(Post::class);
    }
}
```

```php
<?php
// app/Models/Post.php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Post extends Model
{
    /**
     * Get the user that owns the post.
     */
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }
}
```

```php
// Usage
$posts = User::find(1)->posts; // Returns Collection of Post models
$post = Post::find(1)->user;   // Returns User model

// Chaining query constraints
$activePosts = User::find(1)->posts()->where('active', 1)->get();
```

**Expected Output:** `User::find(1)->posts` returns an Eloquent Collection of all posts where `user_id = 1`. The chained query returns only posts where `active = 1` and `user_id = 1`.

**Why:** `hasMany()` returns a `HasMany` relation. Dynamic property access executes the query and returns a collection. Method access returns the builder for further constraints.

---

**Example 3: Custom Foreign Key and Local Key**

```php
<?php
// app/Models/User.php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasMany;

class User extends Model
{
    /**
     * Get the posts for the user using a custom foreign key.
     */
    public function posts(): HasMany
    {
        return $this->hasMany(Post::class, 'author_id', 'user_uuid');
    }
}
```

**Expected Output:** `User::find(1)->posts` queries `SELECT * FROM posts WHERE author_id = (the value of user_uuid on the User model)`.

**Why:** The second argument `'author_id'` overrides the default foreign key. The third argument `'user_uuid'` overrides the default local key (`id`).

### Real-World Cases

- **User Profiles:** `User hasOne Profile`—each user has exactly one profile record.
- **Blog Posts:** `User hasMany Post`—users write multiple blog posts.
- **Order Items:** `Order hasMany OrderItem`—an order contains multiple line items.
- **Country–User:** `User belongsTo Country`—each user belongs to one country.

### References

- Laravel Eloquent: One To One — https://laravel.com/docs/12.x/eloquent-relationships#one-to-one
- Laravel Eloquent: One To Many — https://laravel.com/docs/12.x/eloquent-relationships#one-to-many
- Laravel Eloquent: One To Many (Inverse) / Belongs To — https://laravel.com/docs/12.x/eloquent-relationships#one-to-many-inverse
- Laravel 5.1 Eloquent Relationships (PDF) — https://github.com/DevStorm-Team/laravel-book/blob/a0b5184eb2ba228bdb58ab83e8cf82b91cf440e3/laravel-docs-5.1.pdf

---

## 2. Many-to-Many Relationships

### Definitions

**Core Definition:** A many-to-many relationship connects multiple records in one table to multiple records in another table through an intermediate (pivot) table.

**Technical Definition:** `belongsToMany()` defines a many-to-many relationship using an intermediate pivot table. The pivot table contains foreign key columns for both related models (e.g., `user_id` and `role_id`). The relationship returns a `BelongsToMany` instance, which provides pivot-specific methods such as `withPivot()`, `withTimestamps()`, `wherePivot()`, and `using()` for custom pivot models.

**Beginner-Friendly Explanation:** Think of students and courses. A student can enrol in many courses, and a course can have many students. You need a third table (the pivot table) to record which students are in which courses. Eloquent's `belongsToMany()` handles this automatically.

### Purposes

- To relate two models through a shared intermediate table without duplicating data.
- To access and manage pivot table columns (e.g., `created_at`, `role`, `quantity`) through the `pivot` attribute.
- To filter and order query results based on pivot table columns using `wherePivot()` and `orderByPivot()`.
- To define custom pivot models with `using()` for adding behaviour to intermediate records.
- To use convenient attach/detach/sync methods for managing relationships.
- To support eager loading of many-to-many relationships with `with()`.

### Syntax Rules and Structure

**Complete General Syntax — `belongsToMany()`:**

```php
public function relationshipName(): BelongsToMany
{
    return $this->belongsToMany(
        RelatedModel::class,
        'pivot_table',
        'foreign_pivot_key',
        'related_pivot_key',
        'parent_key',
        'related_key'
    );
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `RelatedModel::class` | The related model class. |
| `'pivot_table'` | Optional. The pivot table name. Default: `{singular_model1}_{singular_model2}` alphabetically. |
| `'foreign_pivot_key'` | Optional. The foreign key for the **current** model on the pivot table. |
| `'related_pivot_key'` | Optional. The foreign key for the **related** model on the pivot table. |
| `'parent_key'` | Optional. The local key on the current model. Default: `id`. |
| `'related_key'` | Optional. The local key on the related model. Default: `id`. |

**Complete General Syntax — Custom Pivot Model:**

```php
public function relationshipName(): BelongsToMany
{
    return $this->belongsToMany(RelatedModel::class)
        ->using(CustomPivot::class)
        ->withPivot('column1', 'column2')
        ->withTimestamps();
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `->using(CustomPivot::class)` | Uses a custom pivot model extending `Pivot`. |
| `->withPivot('column1', 'column2')` | Makes additional pivot columns accessible via `$model->pivot->column1`. |
| `->withTimestamps()` | Automatically maintains `created_at` and `updated_at` on the pivot table. |
| `->wherePivot('column', 'value')` | Adds a constraint on the pivot table. |
| `->as('subscription')` | Renames the pivot attribute accessor from `pivot` to `subscription`. |

**Syntax Rules:**

- The pivot table name convention is the **alphabetically ordered** singular model names joined by an underscore (e.g., `role_user` for `Role` and `User`).
- Both `belongsToMany()` declarations on the two models must reference the **same** pivot table.
- Custom pivot models must extend `Illuminate\Database\Eloquent\Relations\Pivot`.
- The `pivot` attribute is automatically available on related models and contains pivot columns.
- Pivot columns are only accessible if declared via `withPivot()` or if the `Pivot` model's `$guarded` is empty.

**Constraints and Limitations:**

- **Pivot table required:** A database table must exist with the correct foreign key columns.
- **Pivot timestamps:** `created_at` and `updated_at` columns must exist if `withTimestamps()` is used.
- **Custom pivot events:** The `Pivot` model does not fire events by default; use `using()` with a custom model and `$dispatchesEvents` if needed.
- **Sync performance:** `sync()` executes multiple queries; for large datasets, consider `syncWithoutDetaching()`.
- **Pivot model incrementing:** If the pivot table has its own auto-incrementing `id`, set `public $incrementing = true` on the custom pivot model.

### Multiple Annotated Complete Code Examples

**Example 1: Basic Many-to-Many (Users and Roles)**

```php
<?php
// app/Models/User.php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsToMany;

class User extends Model
{
    /**
     * The roles that belong to the user.
     */
    public function roles(): BelongsToMany
    {
        return $this->belongsToMany(Role::class);
    }
}
```

```php
<?php
// app/Models/Role.php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsToMany;

class Role extends Model
{
    /**
     * The users that belong to the role.
     */
    public function users(): BelongsToMany
    {
        return $this->belongsToMany(User::class);
    }
}
```

```php
// Usage
$user = User::find(1);
foreach ($user->roles as $role) {
    echo $role->name;
}

// Attach a role to a user (INSERT INTO role_user ...)
$user->roles()->attach($roleId);

// Detach a role
$user->roles()->detach($roleId);

// Sync roles (removes all, then attaches the given IDs)
$user->roles()->sync([1, 2, 3]);
```

**Expected Output:** `$user->roles` returns a Collection of `Role` models. `attach()` inserts a row into the `role_user` pivot table. `sync()` replaces all existing roles with the provided IDs.

**Why:** `belongsToMany()` uses the `role_user` pivot table (inferred alphabetically). The `attach` and `detach` methods manage rows in this pivot table.

---

**Example 2: Accessing Pivot Columns**

```php
<?php
// app/Models/User.php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsToMany;

class User extends Model
{
    public function roles(): BelongsToMany
    {
        return $this->belongsToMany(Role::class)
            ->withPivot('approved', 'created_by')
            ->withTimestamps();
    }
}
```

```php
// Usage
$user = User::find(1);
foreach ($user->roles as $role) {
    echo $role->pivot->approved;    // Access pivot column
    echo $role->pivot->created_at;  // Access pivot timestamp
}
```

**Expected Output:** The `pivot` attribute on each `Role` model contains the `approved`, `created_by`, `created_at`, and `updated_at` columns from the `role_user` pivot table.

**Why:** `withPivot()` makes additional pivot columns accessible via the `$role->pivot` attribute. Without it, only the foreign keys are available.

---

**Example 3: Custom Pivot Model with `using()`**

```php
<?php
// app/Models/RoleUser.php

namespace App\Models;

use Illuminate\Database\Eloquent\Relations\Pivot;

class RoleUser extends Pivot
{
    /**
     * Indicates if the IDs are auto-incrementing.
     */
    public $incrementing = true;

    /**
     * The attributes that should be cast.
     */
    protected $casts = [
        'approved_at' => 'datetime',
    ];
}
```

```php
<?php
// app/Models/User.php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsToMany;

class User extends Model
{
    public function roles(): BelongsToMany
    {
        return $this->belongsToMany(Role::class)
            ->using(RoleUser::class)
            ->withPivot('approved_at')
            ->withTimestamps();
    }
}
```

```php
// Usage
$user = User::find(1);
$role = $user->roles->first();

// Access the custom pivot model
$pivot = $role->pivot; // RoleUser instance
echo $pivot->approved_at->format('Y-m-d'); // Cast to Carbon
```

**Expected Output:** The `pivot` attribute is an instance of `RoleUser`, with the `approved_at` column cast to a `Carbon` instance.

**Why:** The `using()` method tells Eloquent to use `RoleUser` instead of the default `Pivot` class. Casts defined on the custom pivot model are applied automatically.

### Real-World Cases

- **User–Role Systems:** Users have many roles; roles have many users.
- **E-commerce:** Products belong to many categories; categories contain many products.
- **Social Networks:** Users follow many users; users are followed by many users.
- **Tags:** Blog posts have many tags; tags belong to many posts.
- **Subscriptions:** Users subscribe to many podcasts; podcasts have many subscribers (with a custom `subscription` pivot accessor).

### References

- Laravel Eloquent: Many To Many Relationships — https://laravel.com/docs/12.x/eloquent-relationships#many-to-many
- Laravel Eloquent: Defining Custom Intermediate Table Models — https://laravel.com/docs/12.x/eloquent-relationships#defining-custom-intermediate-table-models
- Laravel Eloquent: Retrieving Intermediate Table Columns — https://laravel.com/docs/12.x/eloquent-relationships#retrieving-intermediate-table-columns
- Laravel 5.6 Pivot Table Documentation (PDF) — https://raw.githubusercontent.com/TIM168/technical_books/master/PHP/laravel框架5_6.pdf

---

## 3. Advanced Through Relationships

### Definitions

**Core Definition:** Through relationships allow access to a distant related model by traversing an intermediate model, without requiring a direct foreign key relationship.

**Technical Definition:** `hasOneThrough()` defines a one-to-one relationship through an intermediate model. `hasManyThrough()` defines a one-to-many relationship through an intermediate model. Both methods accept the final model, the intermediate model, and optional key customisation arguments. The relationship is constructed by joining the intermediate table to the final table and constraining on the parent's local key.

**Beginner-Friendly Explanation:** Imagine a mechanic works on a car, and the car has an owner. The mechanic doesn't directly "own" the owner, but can reach them through the car. `hasOneThrough` lets the mechanic access the car's owner directly. Similarly, a country has many users, and each user has many posts—so the country has many posts through users.

### Purposes

- To access deeply nested related records without defining intermediate relationships manually.
- To simplify queries that would otherwise require multiple joins or nested `whereHas()` clauses.
- To provide a convenient shortcut for common hierarchical data access patterns.
- To support eager loading of distant relationships via `with()`.
- To reduce the number of relationship methods needed on intermediate models.

### Syntax Rules and Structure

**Complete General Syntax — `hasOneThrough()`:**

```php
public function relationshipName(): HasOneThrough
{
    return $this->hasOneThrough(
        FinalModel::class,
        IntermediateModel::class,
        'foreign_key_on_intermediate',
        'foreign_key_on_final',
        'local_key_on_parent',
        'local_key_on_intermediate'
    );
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `FinalModel::class` | The model we want to access. |
| `IntermediateModel::class` | The model we go through. |
| `'foreign_key_on_intermediate'` | Foreign key on the intermediate table pointing to the parent. |
| `'foreign_key_on_final'` | Foreign key on the final table pointing to the intermediate. |
| `'local_key_on_parent'` | Local key on the parent model. Default: `id`. |
| `'local_key_on_intermediate'` | Local key on the intermediate model. Default: `id`. |

**Complete General Syntax — `hasManyThrough()`:**

```php
public function relationshipName(): HasManyThrough
{
    return $this->hasManyThrough(
        FinalModel::class,
        IntermediateModel::class,
        'foreign_key_on_intermediate',
        'foreign_key_on_final',
        'local_key_on_parent',
        'local_key_on_intermediate'
    );
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `FinalModel::class` | The model we want to access (many). |
| `IntermediateModel::class` | The model we go through. |
| Remaining arguments | Same as `hasOneThrough()`. |

**Syntax Rules:**

- The intermediate model must have a foreign key pointing to the parent.
- The final model must have a foreign key pointing to the intermediate.
- Typical foreign key conventions apply; custom keys can be passed as arguments 3–6.
- In Laravel 11+, the `through()` fluent syntax can be used: `return $this->through('environments')->has('deployments');`.
- Both methods support eager loading with `with()`.

**Constraints and Limitations:**

- **No inverse:** Through relationships do not have a `belongsTo` inverse. You cannot traverse back from the final model to the parent using the same relationship.
- **Key customisation:** All six arguments must be correct if customising keys; an error in any argument produces incorrect results.
- **Performance:** Through relationships generate joins, which can be slower than direct relationships on large tables.
- **Pivot tables:** Through relationships do not support many-to-many intermediate tables; use `belongsToMany` for those.

### Multiple Annotated Complete Code Examples

**Example 1: `hasOneThrough` (Mechanic → Car → Owner)**

```php
<?php
// app/Models/Mechanic.php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasOneThrough;

class Mechanic extends Model
{
    /**
     * Get the car's owner.
     */
    public function carOwner(): HasOneThrough
    {
        return $this->hasOneThrough(Owner::class, Car::class);
    }
}
```

```php
// Usage
$mechanic = Mechanic::find(1);
$owner = $mechanic->carOwner; // Returns Owner model or null
```

**Expected Output:** `$mechanic->carOwner` returns the `Owner` model associated with the car that the mechanic works on.

**Why:** Eloquent joins `cars` to `owners` on `cars.owner_id = owners.id`, then constrains `cars.mechanic_id = (mechanic's id)`.

---

**Example 2: `hasManyThrough` (Application → Environments → Deployments)**

```php
<?php
// app/Models/Application.php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasManyThrough;

class Application extends Model
{
    /**
     * Get all of the deployments for the application.
     */
    public function deployments(): HasManyThrough
    {
        return $this->hasManyThrough(Deployment::class, Environment::class);
    }
}
```

**Expected Output:** `Application::find(1)->deployments` returns a Collection of all `Deployment` models whose `environment_id` belongs to an `Environment` whose `application_id = 1`.

**Why:** Eloquent joins `environments` to `deployments` on `environments.id = deployments.environment_id`, then constrains `environments.application_id = (application's id)`.

---

**Example 3: Custom Keys in `hasManyThrough`**

```php
<?php
// app/Models/Country.php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasManyThrough;

class Country extends Model
{
    /**
     * Get all of the posts for the country.
     */
    public function posts(): HasManyThrough
    {
        return $this->hasManyThrough(
            Post::class,
            User::class,
            'country_code',  // Foreign key on users table
            'author_id',     // Foreign key on posts table
            'iso_code',      // Local key on countries table
            'user_uuid'      // Local key on users table
        );
    }
}
```

**Expected Output:** `Country::find(1)->posts` joins `users` to `posts` on `users.user_uuid = posts.author_id`, then constrains `users.country_code = countries.iso_code`.

**Why:** All six arguments customise the key names to match a non-standard database schema.

### Real-World Cases

- **Deployment Platforms:** `Application hasManyThrough Deployment` via `Environment`.
- **Geographic Content:** `Country hasManyThrough Post` via `User`.
- **Organisational Hierarchies:** `Division hasManyThrough Employee` via `Department`.
- **Academic Systems:** `School hasManyThrough Student` via `Classroom`.
- **E-commerce:** `Shop hasManyThrough Review` via `Product`.

### References

- Laravel Eloquent: Has One Through — https://laravel.com/docs/12.x/eloquent-relationships#has-one-through
- Laravel Eloquent: Has Many Through — https://laravel.com/docs/12.x/eloquent-relationships#has-many-through
- Laravel 13.x Eloquent Relationships — https://laravel.com/framework/docs/eloquent-relationships

---

## 4. Polymorphic Relationships

### Definitions

**Core Definition:** A polymorphic relationship allows a model to belong to more than one other model type on a single association, using a `*_type` column to store the parent model class and a `*_id` column to store the parent's primary key.

**Technical Definition:** Laravel supports three polymorphic relationship types: `morphOne()`/`morphTo()` (one-to-one), `morphMany()`/`morphTo()` (one-to-many), and `morphToMany()`/`morphedByMany()` (many-to-many). The `morphTo()` method on the child model dynamically resolves the parent based on the `*_type` and `*_id` columns. The `morphs()` migration macro creates both columns and an index.

**Beginner-Friendly Explanation:** Imagine a "comments" table that stores comments for both blog posts and videos. Instead of having separate `post_comments` and `video_comments` tables, you have one `comments` table with `commentable_type` (either `Post` or `Video`) and `commentable_id` (the ID of the post or video). This is a polymorphic relationship.

### Purposes

- To allow a single table to be associated with multiple parent model types without duplicating schema.
- To reduce the number of tables needed when multiple models share a common child (e.g., comments, images, tags).
- To provide a unified API for accessing related records across different model types.
- To support eager loading of polymorphic relationships using `morphWith()`.
- To enable many-to-many polymorphic relationships (e.g., tags on both posts and videos).
- To decouple stored type strings from class names using morph maps.

### Syntax Rules and Structure

**Complete General Syntax — One-to-One Polymorphic (`morphOne` / `morphTo`):**

```php
// Parent (e.g., Post, User)
public function image(): MorphOne
{
    return $this->morphOne(Image::class, 'imageable');
}

// Child (Image)
public function imageable(): MorphTo
{
    return $this->morphTo();
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `$this->morphOne(Image::class, 'imageable')` | Defines a one-to-one polymorphic relationship. |
| `'imageable'` | The prefix for the `_type` and `_id` columns (`imageable_type`, `imageable_id`). |
| `$this->morphTo()` | Defines the inverse polymorphic relationship. |

**Complete General Syntax — One-to-Many Polymorphic (`morphMany` / `morphTo`):**

```php
// Parent (Post, Video)
public function comments(): MorphMany
{
    return $this->morphMany(Comment::class, 'commentable');
}

// Child (Comment)
public function commentable(): MorphTo
{
    return $this->morphTo();
}
```

**Complete General Syntax — Many-to-Many Polymorphic (`morphToMany` / `morphedByMany`):**

```php
// Post model
public function tags(): MorphToMany
{
    return $this->morphToMany(Tag::class, 'taggable');
}

// Tag model (inverse)
public function posts(): MorphToMany
{
    return $this->morphedByMany(Post::class, 'taggable');
}

public function videos(): MorphToMany
{
    return $this->morphedByMany(Video::class, 'taggable');
}
```

**Syntax Rules:**

- The migration must include `$table->morphs('commentable')` or `$table->nullableMorphs('commentable')` to create the `_type` and `_id` columns.
- The `morphTo()` method accepts an optional first argument to specify the method name if it differs from the column prefix.
- `morphToMany()` and `morphedByMany()` require a pivot table with `*_type` and `*_id` columns plus the related model's foreign key.
- By default, the `_type` column stores the fully qualified class name (e.g., `App\Models\Post`). Use morph maps to store aliases instead.

**Constraints and Limitations:**

- **Strict morph enforcement:** `Relation::enforceMorphMap()` throws `ClassMorphViolationException` if a model is not in the map.
- **Database coupling:** Storing class names in the database couples data to code structure; renaming a class breaks existing records.
- **Index requirements:** Polymorphic columns should be indexed (the `morphs()` macro does this automatically).
- **Eager loading:** Use `morphWith()` to eager-load relationships on polymorphic parents.
- **Type column length:** The `_type` column defaults to `VARCHAR(255)`; morph aliases reduce storage size.

### Multiple Annotated Complete Code Examples

**Example 1: One-to-One Polymorphic (Image for Post and User)**

```php
<?php
// app/Models/Post.php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphOne;

class Post extends Model
{
    public function image(): MorphOne
    {
        return $this->morphOne(Image::class, 'imageable');
    }
}
```

```php
<?php
// app/Models/User.php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphOne;

class User extends Model
{
    public function image(): MorphOne
    {
        return $this->morphOne(Image::class, 'imageable');
    }
}
```

```php
<?php
// app/Models/Image.php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphTo;

class Image extends Model
{
    public function imageable(): MorphTo
    {
        return $this->morphTo();
    }
}
```

**Migration:**

```php
Schema::create('images', function (Blueprint $table) {
    $table->id();
    $table->morphs('imageable'); // imageable_type, imageable_id
    $table->string('url');
    $table->timestamps();
});
```

**Expected Output:** `Post::find(1)->image` returns the `Image` where `imageable_type = 'App\Models\Post'` and `imageable_id = 1`. `User::find(1)->image` returns the image for user 1.

**Why:** The `morphOne` method creates a relationship constrained by both the type and ID columns. The `morphTo` method resolves the parent by reading the `_type` and `_id` columns.

---

**Example 2: One-to-Many Polymorphic (Comments on Posts and Videos)**

```php
<?php
// app/Models/Post.php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphMany;

class Post extends Model
{
    public function comments(): MorphMany
    {
        return $this->morphMany(Comment::class, 'commentable');
    }
}
```

```php
<?php
// app/Models/Comment.php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphTo;

class Comment extends Model
{
    public function commentable(): MorphTo
    {
        return $this->morphTo();
    }
}
```

```php
// Usage
$post = Post::find(1);
foreach ($post->comments as $comment) {
    echo $comment->body;
}

// Access the parent from a comment
$comment = Comment::find(1);
$parent = $comment->commentable; // Post or Video instance
```

**Expected Output:** `$post->comments` returns all comments where `commentable_type = 'App\Models\Post'` and `commentable_id = 1`. `$comment->commentable` returns the parent `Post` or `Video` model.

**Why:** `morphMany()` applies both the type and ID constraints. `morphTo()` dynamically determines the parent model class from the `_type` column and fetches the record.

---

**Example 3: Many-to-Many Polymorphic (Tags on Posts and Videos)**

```php
<?php
// app/Models/Post.php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphToMany;

class Post extends Model
{
    public function tags(): MorphToMany
    {
        return $this->morphToMany(Tag::class, 'taggable');
    }
}
```

```php
<?php
// app/Models/Tag.php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphToMany;

class Tag extends Model
{
    public function posts(): MorphToMany
    {
        return $this->morphedByMany(Post::class, 'taggable');
    }

    public function videos(): MorphToMany
    {
        return $this->morphedByMany(Video::class, 'taggable');
    }
}
```

**Migration for Pivot Table:**

```php
Schema::create('taggables', function (Blueprint $table) {
    $table->id();
    $table->foreignId('tag_id')->constrained();
    $table->morphs('taggable'); // taggable_type, taggable_id
    $table->timestamps();
});
```

```php
// Usage
$post = Post::find(1);
$post->tags()->attach($tagId);

$tag = Tag::find(1);
foreach ($tag->posts as $post) {
    echo $post->title;
}
```

**Expected Output:** `$post->tags` returns all tags associated with the post via the `taggables` pivot table. `$tag->posts` returns all posts tagged with that tag.

**Why:** `morphToMany()` uses the `taggables` pivot table with `taggable_type` and `taggable_id` columns. `morphedByMany()` provides the inverse, allowing the tag to retrieve its associated posts and videos.

### Real-World Cases

- **Comments:** Comments on posts, videos, and products (one-to-many polymorphic).
- **Images:** Profile images, post images, and product images (one-to-one polymorphic).
- **Tags:** Tags applied to posts, videos, and questions (many-to-many polymorphic).
- **Likes:** Likes on posts, comments, and videos.
- **Activity Logs:** Activities referencing different subject types.

### References

- Laravel Eloquent: Polymorphic Relationships — https://laravel.com/docs/12.x/eloquent-relationships#polymorphic-relationships
- Laravel Eloquent: One To One (Polymorphic) — https://laravel.com/docs/12.x/eloquent-relationships#one-to-one-polymorphic-relations
- Laravel Eloquent: One To Many (Polymorphic) — https://laravel.com/docs/12.x/eloquent-relationships#one-to-many-polymorphic-relations
- Laravel Eloquent: Many To Many (Polymorphic) — https://laravel.com/docs/12.x/eloquent-relationships#many-to-many-polymorphic-relations

---

## 5. Enforcing Strict Morph Types

### Definitions

**Core Definition:** `Relation::enforceMorphMap()` is a static method that enforces the use of morph aliases instead of fully qualified class names in polymorphic `*_type` columns, throwing an exception if a model is not mapped.

**Technical Definition:** The `Relation` class provides `morphMap()` (registers aliases but does not enforce) and `enforceMorphMap()` (registers aliases and throws `ClassMorphViolationException` if a model without a morph alias is used in a polymorphic relationship). The `requireMorphMap()` method can be called separately to enable enforcement without registering aliases immediately.

**Beginner-Friendly Explanation:** By default, Laravel stores the full class name (e.g., `App\Models\Post`) in the database for polymorphic types. This is dangerous—if you rename the class, existing data breaks. A morph map lets you store short aliases (e.g., `post`) instead. `enforceMorphMap()` forces every polymorphic model to have an alias, preventing accidental class-name storage.

### Purposes

- To decouple the database's stored type strings from the application's class names.
- To prevent `ClassMorphViolationException` by ensuring all polymorphic models are mapped.
- To reduce database storage by using short aliases instead of long class names.
- To improve performance by avoiding string comparison of long class names.
- To provide a safety net during refactoring (renaming classes does not break existing data).
- To standardise morph type values across the application.

### Syntax Rules and Structure

**Complete General Syntax — `enforceMorphMap()`:**

```php
use Illuminate\Database\Eloquent\Relations\Relation;

// In AppServiceProvider::boot()
Relation::enforceMorphMap([
    'post' => \App\Models\Post::class,
    'video' => \App\Models\Video::class,
    'user' => \App\Models\User::class,
]);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Relation::enforceMorphMap([...])` | Registers aliases and enables strict enforcement. |
| `'post' => Post::class` | Maps the alias `'post'` to the `Post` model. |
| `'video' => Video::class` | Maps the alias `'video'` to the `Video` model. |

**Complete General Syntax — `requireMorphMap()` (Enforcement Without Aliases):**

```php
use Illuminate\Database\Eloquent\Relations\Relation;

// Enable enforcement
Relation::requireMorphMap();

// Register aliases separately
Relation::morphMap([
    'post' => \App\Models\Post::class,
    'video' => \App\Models\Video::class,
]);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Relation::requireMorphMap()` | Enables enforcement without registering aliases. |
| `Relation::morphMap([...])` | Registers aliases in the standard way. |

**Syntax Rules:**

- `enforceMorphMap()` must be called **before** any polymorphic relationship is used (typically in `AppServiceProvider::boot()`).
- Every model used in a polymorphic relationship **must** be included in the map.
- The alias must be a string; it is stored in the `*_type` column.
- When retrofitting a morph map to an existing application, all existing `*_type` values must be updated to match the new aliases.
- `enforceMorphMap()` throws `ClassMorphViolationException` if an unmapped model is used.

**Constraints and Limitations:**

- **Retrofit migration required:** Existing databases with class-name `*_type` values must be migrated (e.g., `UPDATE comments SET commentable_type = 'post' WHERE commentable_type = 'App\\Models\\Post'`).
- **Exception handling:** `ClassMorphViolationException` can occur in unexpected places (e.g., Livewire collections, Nova resources) if a model is not mapped.
- **Performance overhead:** The enforcement check adds a small overhead per polymorphic resolution.
- **Alias uniqueness:** Aliases must be unique across the application.

### Multiple Annotated Complete Code Examples

**Example 1: Enforcing a Morph Map**

```php
<?php
// app/Providers/AppServiceProvider.php

namespace App\Providers;

use Illuminate\Database\Eloquent\Relations\Relation;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        // Enforce morph map — all polymorphic models must be mapped
        Relation::enforceMorphMap([
            'post' => \App\Models\Post::class,
            'video' => \App\Models\Video::class,
            'user' => \App\Models\User::class,
        ]);
    }
}
```

```php
// Usage — comments on posts
$comment = Comment::create([
    'body' => 'Great post!',
    'commentable_type' => 'post', // Stores 'post', not 'App\Models\Post'
    'commentable_id' => 1,
]);

// Accessing the parent
$parent = $comment->commentable; // Resolves to Post model
```

**Expected Output:** The `commentable_type` column stores `'post'` instead of `'App\Models\Post'`. `$comment->commentable` resolves correctly.

**Why:** `enforceMorphMap()` registers the alias and enables strict mode. When Eloquent needs to determine the morph class, it looks up the alias from the map. If a model without an alias is used, `ClassMorphViolationException` is thrown.

---

**Example 2: `ClassMorphViolationException` in Action**

```php
<?php
// Without enforceMorphMap, this works:
$image = Image::create([
    'imageable_type' => 'App\Models\Post', // Fully qualified class name
    'imageable_id' => 1,
]);

// With enforceMorphMap, this throws an exception:
// Relation::enforceMorphMap(['post' => Post::class, 'video' => Video::class]);

$image = Image::create([
    'imageable_type' => 'App\Models\Post', // Not in morph map
]);
// Illuminate\Database\Eloquent\ClassMorphViolationException:
// No morph map defined for model [App\Models\Post].
```

**Expected Output:** After enforcing the morph map, attempting to use an unmapped model's class name throws `ClassMorphViolationException`.

**Why:** `enforceMorphMap()` replaces the default behaviour of storing class names. Any model not explicitly mapped triggers the exception, forcing developers to register all polymorphic models.

---

**Example 3: Retrofit Migration for Existing Data**

```php
<?php
// database/migrations/2026_01_01_000000_update_polymorphic_types.php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Support\Facades\DB;

return new class extends Migration
{
    public function up(): void
    {
        // Convert existing class names to morph aliases
        DB::table('comments')
            ->where('commentable_type', 'App\Models\Post')
            ->update(['commentable_type' => 'post']);

        DB::table('comments')
            ->where('commentable_type', 'App\Models\Video')
            ->update(['commentable_type' => 'video']);

        DB::table('images')
            ->where('imageable_type', 'App\Models\Post')
            ->update(['imageable_type' => 'post']);

        DB::table('images')
            ->where('imageable_type', 'App\Models\User')
            ->update(['imageable_type' => 'user']);
    }
};
```

**Expected Output:** All existing polymorphic `*_type` columns are updated from fully qualified class names to short aliases. The morph map can now be enforced without breaking existing data.

**Why:** When retrofitting a morph map to an existing application, the database must be migrated first. The `enforceMorphMap()` call should be deployed only after the migration completes.

### Real-World Cases

- **Large Applications:** Preventing accidental class-name storage when multiple developers work on polymorphic models.
- **Refactoring:** Renaming model classes (e.g., `App\Models\Article` to `App\Models\BlogPost`) without breaking existing polymorphic records.
- **Multi-Tenant Systems:** Using tenant-specific aliases (e.g., `'tenant_post'`) to isolate polymorphic data.
- **Package Development:** Package authors enforce morph maps to ensure consuming applications follow conventions.
- **Performance Optimisation:** Reducing `*_type` column storage from `VARCHAR(255)` to short alias strings.

### References

- Laravel Eloquent: Custom Polymorphic Types — https://laravel.com/docs/12.x/eloquent-relationships#custom-polymorphic-types
- Enforcing Morph Maps in Laravel (Laravel News) — https://laravel-news.com/enforcing-morph-maps-in-laravel
- Laravel API: Relation Class — https://api.laravel.com/docs/12.x/Illuminate/Database/Eloquent/Relations/Relation.html
- Laravel 8.59 Release Notes — https://laravel-news.com/laravel-8-59-released

---

## References

- Laravel Eloquent: Relationships — https://laravel.com/docs/12.x/eloquent-relationships
- Laravel 13.x Eloquent Relationships — https://laravel.com/framework/docs/eloquent-relationships
- Laravel 5.1 Eloquent Relationships (PDF) — https://github.com/DevStorm-Team/laravel-book/blob/a0b5184eb2ba228bdb58ab83e8cf82b91cf440e3/laravel-docs-5.1.pdf
- Laravel 5.6 Pivot Table Documentation (PDF) — https://raw.githubusercontent.com/TIM168/technical_books/master/PHP/laravel框架5_6.pdf
- Enforcing Morph Maps in Laravel (Laravel News) — https://laravel-news.com/enforcing-morph-maps-in-laravel
- Laravel API: Relation Class — https://api.laravel.com/docs/12.x/Illuminate/Database/Eloquent/Relations/Relation.html
- Laravel 8.59 Release Notes — https://laravel-news.com/laravel-8-59-released
- Laravel Eloquent: Pruning Models — https://laravel.com/docs/12.x/eloquent#pruning-models