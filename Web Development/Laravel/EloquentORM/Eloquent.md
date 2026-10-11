# Laravel Eloquent Fundamentals: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Eloquent is Laravel's object-relational mapper (ORM), a system that maps database tables to PHP classes called "Models," enabling developers to interact with database records as objects rather than writing raw SQL.

**Technical Definition**

Eloquent is an ActiveRecord implementation included with the Laravel framework. Each database table has a corresponding "Model" class that extends `Illuminate\Database\Eloquent\Model`. Models provide methods for retrieving, inserting, updating, and deleting records, as well as defining relationships, query scopes, attribute mutators, and lifecycle events. Eloquent compiles model operations into SQL queries via its query builder, which is built on top of Laravel's database abstraction layer.

**Beginner-Friendly Explanation**

Eloquent is a tool that lets you work with your database using PHP objects instead of writing SQL queries. Instead of saying "SELECT * FROM users WHERE id = 1," you write `$user = User::find(1);`. Each table in your database gets its own PHP class (a "model"), and you use that class to find, create, update, and delete records. It makes database work feel natural and expressive.

---

### Key Characteristics

- **ActiveRecord Pattern**: Each model instance represents a row in the database, and model methods correspond to database operations.
- **Convention Over Configuration**: Table names, primary keys, and timestamps follow sensible defaults that require no configuration.
- **Expressive Query Builder**: Fluent methods like `where()`, `orderBy()`, and `with()` make complex queries readable.
- **Relationships**: Models can define relationships (hasOne, hasMany, belongsTo, belongsToMany, etc.) that enable easy traversal of related data.
- **Attribute Accessors and Mutators**: Attribute values can be transformed when reading from or writing to the database.
- **Type Casting**: Attributes can be automatically cast to native PHP types, enums, or custom cast classes.
- **Lifecycle Events**: Models fire events at key points (creating, created, updating, etc.), enabling hooks for custom logic.
- **Serialization**: Models can be converted to arrays or JSON for API responses.
- **Testing Support**: Model factories and seeders provide a robust system for generating test and development data.

---

### Prerequisites

- **PHP 8.1 or higher** (Laravel 10+) or **PHP 8.2** (Laravel 11+).
- **A Laravel application** installed via Composer.
- **A configured database connection** in `config/database.php`.
- **Basic knowledge of PHP classes and objects**.
- **Understanding of SQL fundamentals** (SELECT, INSERT, UPDATE, DELETE) is helpful but not required.
- **A text editor or IDE** with PHP support.

---

### Related Programming Areas

- **Database Migrations**: Define the schema that models interact with.
- **Query Builder**: Eloquent's underlying query construction layer; can be used directly when needed.
- **API Resources**: Transform models into JSON responses for APIs.
- **Database Seeding**: Populate the database with test or initial data using seeders and factories.
- **Service Container**: Eloquent models are resolved by Laravel's container and can be dependency-injected.
- **Events and Listeners**: Model events integrate with Laravel's event system.

---

## Core Concepts

---

### 1. Eloquent

#### Definitions

**Core Definition**

Eloquent is Laravel's ORM (Object-Relational Mapper), providing an ActiveRecord implementation for working with database tables through PHP objects.

**Technical Definition**

Eloquent is included with Laravel and consists of the `Model` abstract class, the `Builder` query builder, the `Collection` class, and various traits that provide relationships, events, casting, and serialization. All Eloquent models extend `Illuminate\Database\Eloquent\Model`, which provides magic methods for attribute access, relationship resolution, and query scoping.

**Beginner-Friendly Explanation**

Eloquent is the part of Laravel that lets you talk to your database using PHP objects. Instead of writing SQL, you create a class for each table and use methods like `find()`, `where()`, and `save()`.

#### Purposes

- To provide a simple, expressive interface for database interaction.
- To reduce the amount of boilerplate code needed for common database operations.
- To enable a domain-driven design where database tables map to application concepts.
- To support relationships between tables through object-oriented methods.
- To abstract away the differences between database engines (MySQL, PostgreSQL, SQLite, SQL Server).

#### Syntax Rules and Structure

**Complete General Syntax**

```php
<?php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    // Model definition
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `namespace App\Models;` | Default namespace for models. |
| `use Illuminate\Database\Eloquent\Model;` | Imports the base Model class. |
| `class Flight extends Model` | The model class extending Eloquent's base class. |

**Syntax Rules**

- Models must extend `Illuminate\Database\Eloquent\Model`.
- Models are typically placed in `app/Models`.
- The class name should be singular and PascalCase.
- The table name is inferred as the snake_case plural of the class name.

**Constraints and Limitations**

- Eloquent has performance overhead compared to raw queries for large datasets.
- Complex queries may require the underlying query builder or raw SQL.
- Magic methods can obscure the actual database interactions for unfamiliar developers.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Basic Model Usage**

**Setup Guide**

1. Create a model with `php artisan make:model Flight`.
2. Ensure a `flights` table exists in the database.

**Model**

```php
<?php
// app/Models/Flight.php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    // Table name inferred as 'flights'
}
```

**Usage**

```php
use App\Models\Flight;

// Retrieve all flights
$flights = Flight::all();

// Find a flight by primary key
$flight = Flight::find(1);

// Create a new flight
$flight = new Flight;
$flight->name = 'London to Paris';
$flight->save();
```

**Expected Output**

- $flights contains a Collection of Flight objects
- $flight contains a Flight object with id = 1
- The new flight is saved to the database

**Why This Output Occurs**

Eloquent maps the `Flight` class to the `flights` table. `all()` returns all records as model instances. `find(1)` retrieves the record with `id = 1`. `save()` inserts the new record.

---

#### Real-World Cases with Explanation

**Case 1: User Management System**

A user management system uses a `User` model to interact with the `users` table. Controllers use `User::find()`, `User::where()`, and `$user->save()` for all CRUD operations.

**Explanation**: Eloquent centralizes database logic in the model layer, keeping controllers clean.

---

### 2. Models

#### Definitions

**Core Definition**

An Eloquent model is a PHP class that represents a database table and provides methods for interacting with its records.

**Technical Definition**

Each model extends `Illuminate\Database\Eloquent\Model` and maps to a database table. Models define attributes (corresponding to columns), relationships (to other models), scopes (reusable query constraints), accessors/mutators (attribute transformation), casts (type conversion), and events (lifecycle hooks). The model's `newQuery()` method returns a query builder instance pre-configured for the model's table.

**Beginner-Friendly Explanation**

A model is a PHP class that represents a table in your database. It knows how to find, create, update, and delete records in that table.

#### Purposes

- To represent a database table as a PHP class.
- To provide a natural, object-oriented interface for database operations.
- To centralize table-specific logic (relationships, scopes, accessors) in one place.
- To enable the use of Eloquent's advanced features (events, casting, serialization).

#### Syntax Rules and Structure

**Complete General Syntax**

```php
<?php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    protected $table = 'my_flights';
    protected $primaryKey = 'flight_id';
    public $timestamps = false;
    protected $fillable = ['name', 'airline'];
}
```

**Component Breakdown**

| Property | Description |
|----------|-------------|
| `$table` | Custom table name (default: snake_case plural). |
| `$primaryKey` | Custom primary key column (default: `id`). |
| `$timestamps` | Whether to maintain `created_at` and `updated_at`. |
| `$fillable` | Attributes mass-assignable via `create()` or `fill()`. |
| `$guarded` | Attributes not mass-assignable (inverse of fillable). |

**Syntax Rules**

- Models must extend `Model`.
- Class name should be singular PascalCase.
- Table name is inferred but can be overridden with `$table`.
- Primary key is assumed to be `id` but can be overridden with `$primaryKey`.

**Constraints and Limitations**

- Models should not contain business logic that belongs in service classes.
- Large models can become unwieldy; consider using traits to organize.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Model with Custom Table and Key**

**Model**

```php
<?php
// app/Models/Flight.php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    protected $table = 'my_flights';
    protected $primaryKey = 'flight_id';
    public $incrementing = false;
    protected $keyType = 'string';
}
```

**Usage**

```php
$flight = Flight::find('abc-123');
```

**Expected Output**

```php
// $flight is a Flight object where flight_id = 'abc-123'
```

**Why This Output Occurs**

The model is configured to use the `my_flights` table with `flight_id` as a non-incrementing string primary key.

---

#### Real-World Cases with Explanation

**Case 1: Legacy Database Integration**

A legacy database uses table names and primary keys that do not follow Laravel conventions. Models use `$table` and `$primaryKey` to map to the existing schema.

**Explanation**: Eloquent can work with any database schema through configuration.

---

### 3. Model Creation

#### Definitions

**Core Definition**

Model creation is the process of generating a new Eloquent model class using Laravel's Artisan CLI.

**Technical Definition**

The `php artisan make:model` command generates a PHP class file in `app/Models` that extends `Illuminate\Database\Eloquent\Model`. Options allow generating associated migration, factory, seeder, policy, controller, and form request classes in a single command.

**Beginner-Friendly Explanation**

Creating a model is easy—just run a command in your terminal. Laravel makes the file for you, and you fill in the details.

#### Purposes

- To automate the creation of model boilerplate.
- To ensure models are placed in the correct namespace and directory.
- To generate related classes (migration, factory, etc.) in one step.
- To reduce manual file creation errors.

#### Syntax Rules and Structure

**Complete General Syntax**

```bash
php artisan make:model Flight
php artisan make:model Flight --migration
php artisan make:model Flight -mfsc
php artisan make:model Flight --all
```

**Component Breakdown**

| Command | Effect |
|---------|--------|
| `make:model Flight` | Creates `app/Models/Flight.php`. |
| `--migration` or `-m` | Also creates a migration. |
| `--factory` or `-f` | Also creates a factory. |
| `--seed` or `-s` | Also creates a seeder. |
| `--controller` or `-c` | Also creates a controller. |
| `--all` or `-a` | Creates model, migration, factory, seeder, policy, controller, and form requests. |

**Syntax Rules**

- The model name should be singular PascalCase.
- Options can be combined (e.g., `-mfsc`).
- The `--all` shortcut generates a complete set of classes.

**Constraints and Limitations**

- Generated files must be edited to add logic and configuration.
- The model name must be unique.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Creating a Model with Migration**

**Command**

```bash
php artisan make:model Flight --migration
```

**Generated Files**

`app/Models/Flight.php`:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    //
}
```

`database/migrations/xxxx_xx_xx_create_flights_table.php`:

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('flights', function (Blueprint $table) {
            $table->id();
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('flights');
    }
};
```

**Expected Output (Terminal)**

```
INFO  Model [app/Models/Flight.php] created successfully.
INFO  Migration [database/migrations/xxxx_xx_xx_create_flights_table.php] created successfully.
```

**Why This Output Occurs**

Artisan scaffolds the model and migration, placing them in the conventional directories.

---

#### Real-World Cases with Explanation

**Case 1: Rapid Prototyping**

A developer creates a new feature by running `php artisan make:model Post -mfsc`, generating all necessary files in one command.

**Explanation**: Artisan reduces setup time and enforces Laravel conventions.

---

### 4. `artisan make:model`

#### Definitions

**Core Definition**

`artisan make:model` is the Artisan command used to generate new Eloquent model classes.

**Technical Definition**

The command is implemented by `Illuminate\Foundation\Console\ModelMakeCommand`. It generates a PHP class file in `app/Models` and optionally invokes other make commands for migration, factory, seeder, policy, controller, and form request generation.

**Beginner-Friendly Explanation**

`artisan make:model` is the command you type in your terminal to create a new model file.

#### Purposes

- To generate model classes with minimal effort.
- To provide a consistent structure for model files.
- To combine model generation with related class generation.
- To enforce Laravel's naming and directory conventions.

#### Syntax Rules and Structure

**Complete General Syntax**

```bash
php artisan make:model <Name> [options]
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `make:model` | The Artisan command. |
| `<Name>` | The model class name (singular PascalCase). |
| `[options]` | Flags for generating related classes. |

**Syntax Rules**

- The model name is required.
- Options are prefixed with `--` or `-`.
- Multiple options can be combined.

**Constraints and Limitations**

- Generated files are not automatically registered or configured.
- The command does not create database tables (migrations must be run separately).

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Generating a Model with All Related Classes**

**Command**

```bash
php artisan make:model Post --all
```

**Expected Output (Terminal)**

```
INFO  Model [app/Models/Post.php] created successfully.
INFO  Migration [database/migrations/xxxx_xx_xx_create_posts_table.php] created successfully.
INFO  Factory [database/factories/PostFactory.php] created successfully.
INFO  Seeder [database/seeders/PostSeeder.php] created successfully.
INFO  Policy [app/Policies/PostPolicy.php] created successfully.
INFO  Controller [app/Http/Controllers/PostController.php] created successfully.
INFO  Request [app/Http/Requests/StorePostRequest.php] created successfully.
INFO  Request [app/Http/Requests/UpdatePostRequest.php] created successfully.
```

**Why This Output Occurs**

The `--all` flag instructs Artisan to generate the model and all associated classes.

---

#### Real-World Cases with Explanation

**Case 1: Starting a New Resource**

A developer beginning work on a new "Product" resource runs `php artisan make:model Product --all` to scaffold everything needed.

**Explanation**: The `--all` shortcut accelerates development by generating all standard files at once.

---

### 5. Model Conventions

#### Definitions

**Core Definition**

Model conventions are the default assumptions Eloquent makes about table names, primary keys, timestamps, and other properties, which reduce the need for explicit configuration.

**Technical Definition**

Eloquent's conventions include: the table name is the snake_case plural of the class name; the primary key is `id`; the primary key is an auto-incrementing integer; the table has `created_at` and `updated_at` timestamp columns; and dates are cast to `Carbon` instances. These conventions can be overridden by defining properties on the model.

**Beginner-Friendly Explanation**

Conventions are the "default rules" Eloquent follows so you don't have to configure everything. For example, a `User` model automatically uses the `users` table.

#### Purposes

- To reduce configuration and boilerplate code.
- To provide a predictable structure for developers.
- To enable rapid development without specifying every detail.
- To allow overrides when conventions do not fit.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
class Flight extends Model
{
    // Convention: table = 'flights', primary key = 'id', timestamps = true
}
```

**Component Breakdown**

| Convention | Default |
|------------|---------|
| Table Name | Snake_case plural of class name |
| Primary Key | `id` |
| Key Type | Integer, auto-incrementing |
| Timestamps | `created_at` and `updated_at` |
| Date Format | `Y-m-d H:i:s` |

**Syntax Rules**

- Conventions are applied automatically.
- Override by defining properties (`$table`, `$primaryKey`, etc.).
- Conventions can be mixed with overrides.

**Constraints and Limitations**

- Conventions may not fit legacy databases.
- Overriding conventions adds configuration.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Convention vs. Override**

**Convention (no configuration)**

```php
class User extends Model
{
    // Table: users, Primary Key: id, Timestamps: true
}
```

**Override**

```php
class User extends Model
{
    protected $table = 'app_users';
    protected $primaryKey = 'user_id';
    public $timestamps = false;
}
```

**Expected Output**

```php
// Without overrides: queries the 'users' table
// With overrides: queries the 'app_users' table using 'user_id'
```

**Why This Output Occurs**

Eloquent applies its conventions unless overridden by properties on the model.

---

#### Real-World Cases with Explanation

**Case 1: New Laravel Project**

A new project follows all conventions: `User` maps to `users`, `Post` maps to `posts`, etc. No configuration is needed.

**Explanation**: Conventions enable rapid development in greenfield projects.

---

### 6. Table Names

#### Definitions

**Core Definition**

The table name is the database table that an Eloquent model corresponds to. By convention, it is the snake_case plural of the model class name.

**Technical Definition**

Eloquent determines the table name by converting the class name to snake_case and pluralising it. This is done via the `Str::snake()` and `Str::plural()` methods. The table name can be overridden by setting the `$table` property on the model.

**Beginner-Friendly Explanation**

Eloquent assumes your table is named after your model, but plural and in snake_case. So `Flight` uses `flights`, and `AirTrafficController` uses `air_traffic_controllers`.

#### Purposes

- To provide a predictable mapping from class names to table names.
- To reduce the need for explicit table configuration.
- To support rapid development with sensible defaults.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Convention: table inferred
class Flight extends Model { }

// Override: custom table
class Flight extends Model
{
    protected $table = 'my_flights';
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Class name | Singular PascalCase (e.g., `Flight`). |
| Inferred table | Snake_case plural (e.g., `flights`). |
| `$table` | Property to override the inferred table name. |

**Syntax Rules**

- Table name inference is automatic.
- Override with `protected $table`.
- The table name should be unique.

**Constraints and Limitations**

- Irregular plurals may not follow the convention (e.g., `Person` → `people`).
- Eloquent uses `Str::plural()` which handles many irregular cases.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Inferred Table Name**

**Model**

```php
class AirTrafficController extends Model { }
```

**Expected Table**

```
air_traffic_controllers
```

**Why This Output Occurs**

Eloquent converts `AirTrafficController` to snake_case (`air_traffic_controller`) and pluralises it to `air_traffic_controllers`.

---

**Example 2: Custom Table Name**

**Model**

```php
class Flight extends Model
{
    protected $table = 'flight_schedule';
}
```

**Expected Table**

```
flight_schedule
```

**Why This Output Occurs**

The `$table` property overrides the inferred table name.

---

#### Real-World Cases with Explanation

**Case 1: Database with Prefixes**

A database uses table prefixes (e.g., `app_users`). The `User` model sets `$table = 'app_users'`.

**Explanation**: The `$table` property enables compatibility with non-standard naming.

---

### 7. Primary Keys

#### Definitions

**Core Definition**

The primary key is the column that uniquely identifies each row in a table. Eloquent assumes it is named `id` and is an auto-incrementing integer.

**Technical Definition**

Eloquent uses the primary key for `find()`, `findOrFail()`, `update()`, and relationship resolution. The primary key column name can be overridden with `$primaryKey`, and the incrementing and type can be configured with `$incrementing` and `$keyType`.

**Beginner-Friendly Explanation**

The primary key is the unique ID for each row. Eloquent assumes it's called `id` and is a number that goes up automatically.

#### Purposes

- To provide a unique identifier for each record.
- To enable efficient lookups and relationships.
- To support custom primary key configurations for non-standard databases.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
class Flight extends Model
{
    protected $primaryKey = 'flight_id';
    public $incrementing = false;
    protected $keyType = 'string';
}
```

**Component Breakdown**

| Property | Default | Description |
|----------|---------|-------------|
| `$primaryKey` | `id` | Primary key column name. |
| `$incrementing` | `true` | Whether the key auto-increments. |
| `$keyType` | `int` | The type of the key. |

**Syntax Rules**

- Override `$primaryKey` for non-standard column names.
- Set `$incrementing = false` for non-auto-incrementing keys.
- Set `$keyType = 'string'` for UUIDs or other string keys.

**Constraints and Limitations**

- Composite primary keys are not supported natively; use packages or workarounds.
- Non-incrementing keys may affect relationship performance.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: UUID Primary Key**

**Model**

```php
class Flight extends Model
{
    protected $primaryKey = 'uuid';
    public $incrementing = false;
    protected $keyType = 'string';
}
```

**Usage**

```php
$flight = Flight::find('550e8400-e29b-41d4-a716-446655440000');
```

**Expected Output**

```php
// $flight is the Flight with the specified UUID
```

**Why This Output Occurs**

Eloquent uses the `uuid` column as the primary key and treats it as a string.

---

#### Real-World Cases with Explanation

**Case 1: UUID-Based Tables**

A system uses UUIDs as primary keys for security and distributed systems. Models set `$keyType = 'string'` and `$incrementing = false`.

**Explanation**: Eloquent supports non-integer primary keys through configuration.

---

### 8. Timestamps

#### Definitions

**Core Definition**

Timestamps are the `created_at` and `updated_at` columns that Eloquent automatically maintains to track when records are created and modified.

**Technical Definition**

When `$timestamps` is `true` (the default), Eloquent sets `created_at` on insert and updates `updated_at` on save. The columns are cast to `Carbon` instances. The timestamp format can be customised via `$dateFormat`. To disable timestamps, set `$timestamps = false`.

**Beginner-Friendly Explanation**

Timestamps are automatic date fields that record when a row was created and last updated.

#### Purposes

- To provide automatic audit trails for record creation and modification.
- To enable time-based queries and sorting.
- To reduce manual date management.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
class Flight extends Model
{
    public $timestamps = true; // default
    protected $dateFormat = 'U'; // Unix timestamp
}
```

**Component Breakdown**

| Property | Default | Description |
|----------|---------|-------------|
| `$timestamps` | `true` | Whether to maintain timestamps. |
| `$dateFormat` | `Y-m-d H:i:s` | Storage format for dates. |
| `CREATED_AT` | `created_at` | Custom column name. |
| `UPDATED_AT` | `updated_at` | Custom column name. |

**Syntax Rules**

- Timestamps require the corresponding columns in the table.
- Disable with `$timestamps = false`.
- Customise column names with constants.

**Constraints and Limitations**

- Tables must have `created_at` and `updated_at` columns unless disabled.
- Timestamps use the application's timezone.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Disabling Timestamps**

**Model**

```php
class Flight extends Model
{
    public $timestamps = false;
}
```

**Expected Output**

```php
// No created_at or updated_at columns are set on insert/update
```

**Why This Output Occurs**

Eloquent skips timestamp management when `$timestamps` is `false`.

---

#### Real-World Cases with Explanation

**Case 1: Log Tables Without Timestamps**

A log table uses its own timestamp columns. The model disables Eloquent timestamps to avoid conflicts.

**Explanation**: Timestamps can be disabled for tables with custom time tracking.

---

### 9. Model Attributes

#### Definitions

**Core Definition**

Model attributes are the data properties of a model instance, corresponding to columns in the database table.

**Technical Definition**

Eloquent stores attributes in a protected `$attributes` array. Attribute access is handled by magic methods (`__get`, `__set`), which check for accessors, mutators, and casts. The `$fillable` and `$guarded` properties control mass assignment.

**Beginner-Friendly Explanation**

Attributes are the pieces of data a model holds, like a user's name or email. You access them as properties: `$user->name`.

#### Purposes

- To represent database column values as object properties.
- To enable attribute transformation via accessors and mutators.
- To control which attributes can be mass-assigned.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
$flight->name = 'London to Paris';
echo $flight->name;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `$model->attribute` | Access an attribute. |
| `$model->attribute = value` | Set an attribute. |
| `$fillable` | Attributes allowed for mass assignment. |
| `$guarded` | Attributes blocked from mass assignment. |

**Syntax Rules**

- Attributes are accessed as public properties.
- The `$fillable` property is recommended for mass assignment protection.
- `$guarded = []` disables mass assignment protection (use with caution).

**Constraints and Limitations**

- Accessing undefined attributes returns `null`.
- Mass assignment without `$fillable` or `$guarded` throws an exception.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Mass Assignment**

**Model**

```php
class Flight extends Model
{
    protected $fillable = ['name', 'airline'];
}
```

**Usage**

```php
$flight = Flight::create(['name' => 'Paris', 'airline' => 'Air France']);
```

**Expected Output**

```php
// $flight is created with name and airline set
```

**Why This Output Occurs**

The `$fillable` property allows `name` and `airline` to be mass-assigned via `create()`.

---

#### Real-World Cases with Explanation

**Case 1: Form Submission**

A controller uses `Flight::create($request->validated())` to create a record from form data.

**Explanation**: Mass assignment with `$fillable` provides a clean, safe way to create records from user input.

---

### 10. Model Methods

#### Definitions

**Core Definition**

Model methods are instance or static methods on a model class that provide database operations and custom behaviour.

**Technical Definition**

Eloquent's `Model` class provides dozens of methods (`find`, `where`, `save`, `update`, `delete`, `all`, `first`, etc.). Models can also define their own methods for custom business logic, relationships, scopes, and accessors.

**Beginner-Friendly Explanation**

Model methods are the actions you can perform on a model, like saving or finding records.

#### Purposes

- To provide a rich API for database operations.
- To allow custom logic to be encapsulated in the model.
- To support relationship definitions and query scopes.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Static methods
Flight::find(1);
Flight::where('active', true)->get();

// Instance methods
$flight->save();
$flight->update(['name' => 'New Name']);
$flight->delete();
```

**Component Breakdown**

| Method | Type | Description |
|--------|------|-------------|
| `find($id)` | Static | Find by primary key. |
| `where(...)` | Static | Add a query constraint. |
| `all()` | Static | Retrieve all records. |
| `save()` | Instance | Insert or update the model. |
| `update(array)` | Instance | Update attributes and save. |
| `delete()` | Instance | Delete the record. |

**Syntax Rules**

- Static methods return a query builder or collection.
- Instance methods operate on a single model instance.
- Methods are chainable for query building.

**Constraints and Limitations**

- `save()` must be called to persist changes.
- Mass updates via `update()` do not fire model events.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Retrieving and Updating**

**Usage**

```php
$flight = Flight::find(1);
$flight->name = 'New Flight Name';
$flight->save();
```

**Expected Output**

```php
// The flight with id = 1 is updated in the database
```

**Why This Output Occurs**

`find(1)` retrieves the model, the attribute is modified, and `save()` persists the change.

---

#### Real-World Cases with Explanation

**Case 1: CRUD Controller**

A controller uses `Model::all()`, `Model::find()`, `$model->save()`, and `$model->delete()` for standard CRUD operations.

**Explanation**: Eloquent's methods cover all common database operations.

---

### 11. Model Events

#### Definitions

**Core Definition**

Model events are hooks that fire at specific points in a model's lifecycle, such as before or after creation, update, or deletion.

**Technical Definition**

Eloquent fires the following events: `retrieved`, `creating`, `created`, `updating`, `updated`, `saving`, `saved`, `deleting`, `deleted`, `restoring`, `restored`, `forceDeleting`, `forceDeleted`, `replicating`. Listeners can be registered via closures in the model's `booted()` method, via event listeners, or via observers.

**Beginner-Friendly Explanation**

Model events are like notifications that go off when something happens to a model—like when it's about to be saved or deleted. You can listen for these notifications and run your own code.

#### Purposes

- To execute custom logic at specific points in a model's lifecycle.
- To automate tasks like logging, caching, or sending notifications.
- To enforce business rules before or after database operations.
- To decouple side effects from the main application flow.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
class Flight extends Model
{
    protected static function booted(): void
    {
        static::creating(function (Flight $flight) {
            // Before creation
        });

        static::created(function (Flight $flight) {
            // After creation
        });
    }
}
```

**Component Breakdown**

| Event | When Fired |
|-------|------------|
| `creating` | Before a new model is saved. |
| `created` | After a new model is saved. |
| `updating` | Before an existing model is updated. |
| `updated` | After an existing model is updated. |
| `saving` | Before saving (create or update). |
| `saved` | After saving (create or update). |
| `deleting` | Before a model is deleted. |
| `deleted` | After a model is deleted. |
| `retrieved` | After a model is retrieved from the database. |

**Syntax Rules**

- Register listeners in `booted()`.
- Use `static::eventName(callback)` to register.
- Events receive the model instance as an argument.

**Constraints and Limitations**

- Mass updates via `Model::where()->update()` do not fire `saving`, `saved`, `updating`, or `updated` events.
- Events can impact performance if not used carefully.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Logging on Create**

**Model**

```php
class Flight extends Model
{
    protected static function booted(): void
    {
        static::created(function (Flight $flight) {
            logger("Flight {$flight->id} was created.");
        });
    }
}
```

**Usage**

```php
Flight::create(['name' => 'Paris', 'airline' => 'Air France']);
```

**Expected Output (in log)**

```
Flight 1 was created.
```

**Why This Output Occurs**

The `created` event fires after the model is saved, and the closure logs the message.

---

#### Real-World Cases with Explanation

**Case 1: Slug Generation**

A `Post` model generates a slug from the title in the `creating` event if no slug is provided.

**Explanation**: Model events enable automatic, consistent behaviour across all model operations.

---

### 12. Model Observers

#### Definitions

**Core Definition**

Model observers are classes that group event listeners for a model into a single class, with methods named after the events they handle.

**Technical Definition**

An observer class has methods like `creating`, `created`, `updating`, `updated`, etc. It is registered on a model via the `ObservedBy` attribute or the `observe()` method in the model's `boot()` method. Observers are useful when a model has many event listeners.

**Beginner-Friendly Explanation**

An observer is like a dedicated listener class for a model's events. If you have many event handlers, grouping them in an observer keeps your model clean.

#### Purposes

- To group multiple event listeners for a model into a single class.
- To keep model classes clean and focused.
- To enable reusable event handling logic across models.
- To provide a clear, organized structure for lifecycle logic.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Observer class
namespace App\Observers;

use App\Models\Flight;

class FlightObserver
{
    public function creating(Flight $flight): void
    {
        // Before creation
    }

    public function created(Flight $flight): void
    {
        // After creation
    }
}
```

```php
// Registering the observer
use App\Observers\FlightObserver;

#[ObservedBy([FlightObserver::class])]
class Flight extends Model
{
    // ...
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Observer class | Contains methods named after events. |
| `ObservedBy` attribute | Registers the observer on the model. |
| `observe()` | Programmatic registration. |

**Syntax Rules**

- Observer methods receive the model instance.
- Register via `ObservedBy` attribute (Laravel 10+) or `Model::observe()`.
- Observers are typically placed in `app/Observers`.

**Constraints and Limitations**

- Observers add a layer of indirection.
- Event firing order can be complex with multiple observers.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: User Observer**

**Observer**

```php
<?php
// app/Observers/UserObserver.php
namespace App\Observers;

use App\Models\User;

class UserObserver
{
    public function created(User $user): void
    {
        logger("User {$user->email} registered.");
    }
}
```

**Model**

```php
<?php
// app/Models/User.php
namespace App\Models;

use App\Observers\UserObserver;
use Illuminate\Database\Eloquent\Attributes\ObservedBy;
use Illuminate\Database\Eloquent\Model;

#[ObservedBy([UserObserver::class])]
class User extends Model
{
    // ...
}
```

**Expected Output (in log)**

```
User john@example.com registered.
```

**Why This Output Occurs**

The `UserObserver` is registered on the `User` model, and its `created` method is called when a user is created.

---

#### Real-World Cases with Explanation

**Case 1: Sending Welcome Emails**

A `UserObserver` sends a welcome email in the `created` method. This keeps the user registration controller clean.

**Explanation**: Observers centralise event logic, making the codebase more maintainable.

---

### 13. Model Scopes

#### Definitions

**Core Definition**

Model scopes are reusable query constraints that can be applied to model queries, either globally (applied to all queries) or locally (applied on demand).

**Technical Definition**

Local scopes are defined as methods on the model, either prefixed with `scope` or annotated with the `Scope` attribute. Global scopes are classes implementing `Illuminate\Database\Eloquent\Scope` or closures registered via `addGlobalScope()`. Global scopes are applied to all queries unless explicitly removed with `withoutGlobalScope()`.

**Beginner-Friendly Explanation**

Scopes are pre-built query filters. Instead of writing the same `where` clause everywhere, you define it once as a scope and reuse it.

#### Purposes

- To encapsulate common query constraints in reusable methods.
- To reduce duplication of query logic.
- To apply consistent filtering across an application (global scopes).
- To provide a clean, expressive API for complex queries.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Local scope (prefixed)
public function scopePopular($query)
{
    return $query->where('votes', '>', 100);
}

// Local scope (attribute)
use Illuminate\Database\Eloquent\Attributes\Scope;

#[Scope]
protected function popular(Builder $query): void
{
    $query->where('votes', '>', 100);
}

// Global scope
protected static function booted(): void
{
    static::addGlobalScope('active', function (Builder $builder) {
        $builder->where('active', true);
    });
}
```

**Component Breakdown**

| Type | Description |
|------|-------------|
| Local scope | Reusable query constraint called explicitly. |
| Global scope | Applied automatically to all queries. |
| `#[Scope]` | Attribute for defining local scopes. |

**Syntax Rules**

- Local scopes are called as static methods: `Model::popular()->get()`.
- Global scopes are registered in `booted()`.
- Global scopes can be removed with `withoutGlobalScope()`.

**Constraints and Limitations**

- Global scopes can be surprising if not documented.
- Scopes should not contain logic that depends on request context unless intended.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Local Scope**

**Model**

```php
class User extends Model
{
    #[Scope]
    protected function popular(Builder $query): void
    {
        $query->where('votes', '>', 100);
    }
}
```

**Usage**

```php
$popularUsers = User::popular()->get();
```

**Expected Output**

```php
// Returns all users with votes > 100
```

**Why This Output Occurs**

The `popular` scope adds a `where('votes', '>', 100)` constraint to the query.

---

**Example 2: Global Scope**

**Model**

```php
protected static function booted(): void
{
    static::addGlobalScope('active', function (Builder $builder) {
        $builder->where('active', true);
    });
}
```

**Usage**

```php
$users = User::all(); // Only returns active users
```

**Expected Output**

```php
// Returns only users where active = true
```

**Why This Output Occurs**

The global scope automatically adds `where('active', true)` to every query.

---

#### Real-World Cases with Explanation

**Case 1: Soft Deletes**

Laravel's `SoftDeletes` trait is a global scope that excludes soft-deleted records from queries.

**Explanation**: Global scopes provide consistent, automatic filtering.

---

### 14. Model Accessors

#### Definitions

**Core Definition**

Accessors are methods that transform attribute values when they are retrieved from the model.

**Technical Definition**

In Laravel 9+, accessors are defined using the `Attribute` class:

```php
protected function firstName(): Attribute
{
    return Attribute::make(
        get: fn (string $value) => ucfirst($value),
    );
}
```

The `get` callback receives the raw attribute value and returns the transformed value. Legacy accessors use the `getFooAttribute` naming convention.

**Beginner-Friendly Explanation**

An accessor lets you change how an attribute looks when you read it. For example, you can capitalise a name or format a date.

#### Purposes

- To format attribute values for display or further processing.
- To compute derived values from raw data.
- To encapsulate presentation logic in the model.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
use Illuminate\Database\Eloquent\Casts\Attribute;

protected function firstName(): Attribute
{
    return Attribute::make(
        get: fn (string $value) => ucfirst($value),
    );
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `Attribute::make()` | Creates an Attribute instance. |
| `get:` | Callback for reading the attribute. |
| `$value` | The raw attribute value. |

**Syntax Rules**

- Accessors are defined as `protected` methods.
- The method name is the camelCase attribute name.
- The `get` callback must return the transformed value.

**Constraints and Limitations**

- Accessors are not applied to raw database queries.
- Accessors can impact performance if they perform expensive operations.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Name Accessor**

**Model**

```php
protected function firstName(): Attribute
{
    return Attribute::make(
        get: fn (string $value) => ucfirst($value),
    );
}
```

**Usage**

```php
echo $user->first_name; // 'john' in DB, 'John' in output
```

**Expected Output**

```
John
```

**Why This Output Occurs**

The accessor capitalises the first letter of the `first_name` attribute when it is read.

---

#### Real-World Cases with Explanation

**Case 1: Formatting Prices**

A `Product` model has an accessor that formats the `price` attribute as a currency string.

**Explanation**: Accessors keep formatting logic in the model, where it belongs.

---

### 15. Model Mutators

#### Definitions

**Core Definition**

Mutators are methods that transform attribute values before they are stored in the database.

**Technical Definition**

Mutators are defined using the `Attribute` class with a `set` callback:

```php
protected function firstName(): Attribute
{
    return Attribute::make(
        set: fn (string $value) => strtolower($value),
    );
}
```

The `set` callback receives the value being assigned and returns the value to be stored.

**Beginner-Friendly Explanation**

A mutator changes an attribute before it's saved. For example, you can lowercase an email address before storing it.

#### Purposes

- To normalise data before storage.
- To encrypt or hash sensitive values.
- To transform user input into a consistent format.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
protected function email(): Attribute
{
    return Attribute::make(
        set: fn (string $value) => strtolower($value),
    );
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `Attribute::make()` | Creates an Attribute instance. |
| `set:` | Callback for setting the attribute. |
| `$value` | The value being assigned. |

**Syntax Rules**

- Mutators are defined as `protected` methods.
- The `set` callback must return the value to be stored.

**Constraints and Limitations**

- Mutators are not applied when using raw SQL.
- Mutators can impact performance.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Email Mutator**

**Model**

```php
protected function email(): Attribute
{
    return Attribute::make(
        set: fn (string $value) => strtolower($value),
    );
}
```

**Usage**

```php
$user->email = 'John@Example.com';
$user->save();
// Stored as 'john@example.com'
```

**Expected Output**

```
john@example.com
```

**Why This Output Occurs**

The mutator lowercases the email before it is stored in the database.

---

#### Real-World Cases with Explanation

**Case 1: Password Hashing**

A `User` model has a mutator that hashes the password attribute before storage.

**Explanation**: Mutators ensure sensitive data is consistently transformed.

---

### 16. Model Casting

#### Definitions

**Core Definition**

Model casting is the automatic conversion of attribute values between database types and PHP types.

**Technical Definition**

Casts are defined in the `casts()` method (Laravel 11+) or the `$casts` property. Supported casts include `int`, `float`, `string`, `bool`, `array`, `json`, `object`, `collection`, `date`, `datetime`, `timestamp`, `encrypted`, and enum classes. Custom casts implement `CastsAttributes`.

**Beginner-Friendly Explanation**

Casting automatically changes an attribute's type when you read or write it. For example, a JSON column can become a PHP array automatically.

#### Purposes

- To eliminate manual type conversion.
- To ensure attributes are always the correct type.
- To support JSON, encrypted, and enum columns.
- To enable custom type transformations.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
protected function casts(): array
{
    return [
        'is_admin' => 'boolean',
        'options' => 'array',
        'created_at' => 'datetime',
        'status' => ServerStatus::class,
    ];
}
```

**Component Breakdown**

| Cast | Description |
|------|-------------|
| `int`, `float`, `string`, `bool` | Primitive types. |
| `array`, `json`, `object`, `collection` | JSON-based types. |
| `date`, `datetime`, `timestamp` | Date types. |
| Enum class | Cast to PHP enum. |

**Syntax Rules**

- Casts are defined in the `casts()` method.
- The array key is the attribute name, the value is the cast type.
- Custom casts implement `CastsAttributes`.

**Constraints and Limitations**

- Casting large JSON columns can impact performance.
- Encrypted casts require a valid encryption key.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Boolean and Array Casts**

**Model**

```php
protected function casts(): array
{
    return [
        'is_admin' => 'boolean',
        'options' => 'array',
    ];
}
```

**Usage**

```php
$user->is_admin = 1; // Stored as true
$options = $user->options; // Automatically decoded to array
```

**Expected Output**

```php
// $user->is_admin === true
// is_array($options) === true
```

**Why This Output Occurs**

The `boolean` cast converts `1` to `true`. The `array` cast decodes the JSON column to a PHP array.

---

#### Real-World Cases with Explanation

**Case 1: Settings JSON Column**

A `User` model casts a `settings` column to `array`, allowing settings to be accessed as a PHP array.

**Explanation**: Casting eliminates manual JSON encoding/decoding.

---

### 17. Model Serialization

#### Definitions

**Core Definition**

Model serialization is the conversion of a model and its relationships into an array or JSON representation.

**Technical Definition**

Eloquent provides `toArray()` and `toJson()` methods. The `$hidden` property excludes attributes from serialization, while `$visible` whitelists them. The `$appends` property adds accessor values to the serialized output. For APIs, Laravel recommends using API Resources instead of serializing models directly.

**Beginner-Friendly Explanation**

Serialization turns a model into an array or JSON so you can send it to an API or store it.

#### Purposes

- To prepare model data for API responses.
- To convert models to JSON for JavaScript consumption.
- To control which attributes are included in the output.
- To include computed values via accessors.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
$array = $model->toArray();
$json = $model->toJson();
```

```php
class User extends Model
{
    protected $hidden = ['password', 'remember_token'];
    protected $appends = ['full_name'];
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `toArray()` | Converts model to array. |
| `toJson()` | Converts model to JSON. |
| `$hidden` | Attributes excluded from serialization. |
| `$visible` | Attributes included in serialization. |
| `$appends` | Accessors added to serialization. |

**Syntax Rules**

- `toArray()` is recursive for loaded relationships.
- `$hidden` and `$visible` are mutually exclusive in practice.
- `$appends` adds accessor results to the output.

**Constraints and Limitations**

- Serializing models directly in API responses can leak schema information.
- Use API Resources for production APIs.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Hiding Sensitive Attributes**

**Model**

```php
class User extends Model
{
    protected $hidden = ['password'];
}
```

**Usage**

```php
return $user->toJson();
```

**Expected Output**

```json
{"id":1,"name":"John","email":"john@example.com"}
```

**Why This Output Occurs**

The `password` attribute is excluded from the JSON output due to the `$hidden` property.

---

#### Real-World Cases with Explanation

**Case 1: API Response**

A controller returns `User::all()` directly. Laravel automatically serializes the collection to JSON, hiding sensitive attributes.

**Explanation**: Serialization makes API responses easy to generate.

---

### 18. Model Factories

#### Definitions

**Core Definition**

Model factories are classes that define a default set of attributes for generating fake model instances for testing and seeding.

**Technical Definition**

Factories extend `Illuminate\Database\Eloquent\Factories\Factory` and implement a `definition()` method returning an array of attribute values. The `fake()` helper provides access to Faker. Factories support states, sequences, and callbacks.

**Beginner-Friendly Explanation**

A factory is a recipe for creating fake data. It's used in tests and seeders to generate realistic records.

#### Purposes

- To generate realistic test data quickly.
- To ensure consistency across tests.
- To reduce boilerplate in seeders and tests.
- To support complex data generation with states and sequences.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
namespace Database\Factories;

use App\Models\User;
use Illuminate\Database\Eloquent\Factories\Factory;
use Illuminate\Support\Str;

class UserFactory extends Factory
{
    protected $model = User::class;

    public function definition(): array
    {
        return [
            'name' => fake()->name(),
            'email' => fake()->unique()->safeEmail(),
            'password' => bcrypt('password'),
        ];
    }
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `definition()` | Returns default attributes. |
| `fake()` | Faker instance. |
| `$model` | The model class. |
| States | Named variations of the factory. |

**Syntax Rules**

- Factories are placed in `database/factories`.
- The `definition()` method must return an array.
- Use `fake()` for random data.

**Constraints and Limitations**

- Factories require a corresponding model.
- Complex relationships may require additional configuration.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Using a Factory**

**Factory**

```php
class UserFactory extends Factory
{
    public function definition(): array
    {
        return [
            'name' => fake()->name(),
            'email' => fake()->unique()->safeEmail(),
        ];
    }
}
```

**Usage**

```php
$user = User::factory()->create();
```

**Expected Output**

```php
// $user is a persisted User with random name and email
```

**Why This Output Occurs**

The `create()` method generates a model instance and persists it to the database.

---

#### Real-World Cases with Explanation

**Case 1: Seeding Development Data**

A seeder uses `User::factory()->count(50)->create()` to populate the database with test users.

**Explanation**: Factories make it easy to generate realistic data for development.

---

### 19. Model Seeders

#### Definitions

**Core Definition**

Seeders are classes that populate the database with initial or test data using the query builder or model factories.

**Technical Definition**

Seeders extend `Illuminate\Database\Seeder` and implement a `run()` method. The `DatabaseSeeder` class serves as the entry point and can call other seeders via the `call()` method. Seeders are run with `php artisan db:seed`.

**Beginner-Friendly Explanation**

A seeder is a script that fills your database with data. It's useful for setting up default records or test data.

#### Purposes

- To populate the database with initial data.
- To provide test data for development.
- To automate database setup for testing.
- To ensure consistent data across environments.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
namespace Database\Seeders;

use App\Models\User;
use Illuminate\Database\Seeder;

class UserSeeder extends Seeder
{
    public function run(): void
    {
        User::factory()->count(10)->create();
    }
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `run()` | The seeder logic. |
| `DatabaseSeeder` | The main entry point. |
| `call()` | Calls other seeders. |

**Syntax Rules**

- Seeders are placed in `database/seeders`.
- `run()` is called when the seeder is executed.
- Use `db:seed` to run seeders.

**Constraints and Limitations**

- Seeders should be idempotent (safe to run multiple times).
- Large seeders can be slow.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Seeding Users**

**Seeder**

```php
class UserSeeder extends Seeder
{
    public function run(): void
    {
        User::factory()->count(50)->create();
    }
}
```

**DatabaseSeeder**

```php
class DatabaseSeeder extends Seeder
{
    public function run(): void
    {
        $this->call([
            UserSeeder::class,
        ]);
    }
}
```

**Command**

```bash
php artisan db:seed
```

**Expected Output**

```php
// 50 users are created in the database
```

**Why This Output Occurs**

The `UserSeeder` uses the factory to create 50 users, and the `DatabaseSeeder` calls it.

---

#### Real-World Cases with Explanation

**Case 1: Default Admin User**

A seeder creates a default admin user for a new application.

**Explanation**: Seeders ensure essential data exists in every environment.

---

### 20. Eloquent Best Practices

#### Definitions

**Core Definition**

Eloquent best practices are the recommended guidelines for using Eloquent effectively, performantly, and maintainably.

**Technical Definition**

Best practices encompass query optimisation (eager loading, chunking), model organisation (traits, services), mass assignment safety, serialization strategies, testing, and architectural patterns.

**Beginner-Friendly Explanation**

Best practices are the tips and tricks experienced developers use to get the most out of Eloquent while avoiding common pitfalls.

#### Key Best Practices

1. **Use Eager Loading to Prevent N+1 Queries**: Use `with()` to load relationships in advance.

2. **Use `chunk()` or `cursor()` for Large Datasets**: Process large record sets in batches to avoid memory exhaustion.

3. **Prefer Local Scopes for Reusable Query Logic**: Encapsulate common constraints in scopes.

4. **Use `$fillable` for Mass Assignment Protection**: Never use `$guarded = []` in production.

5. **Use API Resources for API Responses**: Do not serialize models directly in API controllers.

6. **Hide Sensitive Attributes**: Use `$hidden` for passwords, tokens, and internal fields.

7. **Use Casting for Type Safety**: Cast attributes to their expected types.

8. **Keep Models Focused**: Move complex business logic to service classes or action classes.

9. **Use Observers for Event Logic**: Group event listeners in observers rather than bloating the model.

10. **Test with Factories**: Use factories to generate consistent test data.

11. **Avoid Queries in Loops**: Batch queries or use eager loading.

12. **Index Frequently Queried Columns**: Ensure database indexes support your queries.

#### Real-World Cases with Explanation

**Case 1: Eager Loading in a Blog**

A blog index page loads posts and their authors. Using `Post::with('author')->get()` prevents N+1 queries.

**Explanation**: Eager loading is the single most impactful performance optimisation for Eloquent.

---

## References

- Laravel Eloquent: Getting Started (Laravel 12.x) — https://laravel.com/docs/12.x/eloquent
- Laravel Eloquent: Getting Started (Laravel master) — https://laravel.com/framework/docs/master/eloquent
- Laravel Eloquent: Mutators & Casting (Laravel 12.x) — https://laravel.com/docs/12.x/eloquent-mutators
- Laravel Eloquent: Serialization (Laravel 12.x) — https://laravel.com/docs/12.x/eloquent-serialization
- Laravel Eloquent: Factories (Laravel 12.x) — https://laravel.com/docs/12.x/eloquent-factories
- Laravel Database: Seeding (Laravel 12.x) — https://laravel.com/docs/12.x/seeding
- Laravel Eloquent: API Resources (Laravel 12.x) — https://laravel.com/docs/12.x/eloquent-resources
- Laravel API: Model Class — https://api.laravel.com/docs/12.x/Illuminate/Database/Eloquent/Model.html
- Laravel API: Builder Class — https://api.laravel.com/docs/12.x/Illuminate/Database/Eloquent/Builder.html
- Laravel API: Attribute Cast — https://api.laravel.com/docs/12.x/Illuminate/Database/Eloquent/Casts/Attribute.html
- Laravel News: Getting Started with Laravel Model Events — https://laravel-news.com/laravel-model-events
- Laravel Daily: Optimizing Laravel Eloquent and DB Speed — https://laraveldaily.com/post/optimizing-laravel-eloquent-db-speed-all-you-need-to-know
- Kinsta: How To Generate and Use Fake Records with Laravel Model Factories — https://kinsta.com/blog/laravel-model-factories/

---

**Version Note**: This cheat sheet is based on Laravel 10.x through 13.x documentation. Eloquent has remained highly stable across these versions, but developers should always consult the documentation for their specific Laravel version. Features such as the `Attribute` class for accessors/mutators (Laravel 9+), the `#[Scope]` attribute (Laravel 11+), and the `casts()` method (Laravel 11+) are version-specific. The `ObservedBy` attribute was introduced in Laravel 10.44. Developers should verify feature availability for their target Laravel version.