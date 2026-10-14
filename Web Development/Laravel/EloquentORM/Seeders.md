# Laravel Seeders: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Laravel Seeders are dedicated PHP classes that populate a database with initial, test, or lookup data, providing a structured, repeatable, and version-controlled approach to database seeding.

**Technical Definition**

Seeders are classes extending `Illuminate\Database\Seeder` stored in the `database/seeders` directory. Each seeder defines a `run()` method that is invoked by the `db:seed` Artisan command. Seeders can use the query builder (`DB::table()`), Eloquent models, or model factories to insert data. The `DatabaseSeeder` class serves as the main orchestrator, calling other seeders via the `call()` method. Mass assignment protection is automatically disabled during seeding. Seeders support dependency injection in the `run()` method signature, allowing services to be resolved from the Laravel service container.

**Beginner-Friendly Explanation**

A seeder is a PHP class that fills your database with data. Instead of manually inserting records or writing SQL scripts, you write a class that tells Laravel what data to add. This is especially useful when you are developing your application and need test data, or when you deploy your application and need certain default data (like admin users or country lists) to exist. You run one command, and Laravel fills the database for you.

---

### Key Characteristics

- **Structured Data Population**: Seeders provide a class-based, organised approach to inserting data into the database.
- **Orchestration via `DatabaseSeeder`**: The main `DatabaseSeeder` class calls other seeders in a defined order, allowing complex seeding sequences to be composed from smaller, focused seeders.
- **Factory Integration**: Seeders are commonly used with Eloquent model factories to generate large volumes of realistic test data using the Faker PHP library.
- **Automatic Mass Assignment Disabled**: Mass assignment protection is automatically disabled during seeding, allowing direct model creation without `$fillable` constraints.
- **Dependency Injection**: The `run()` method supports type-hinted dependencies resolved by the Laravel service container.
- **Idempotency Support**: Seeders can be written to be idempotent (safe to run multiple times) using `findOrCreate`, `updateOrCreate`, or `upsert`.
- **Context Sharing**: Laravel's `Context` facade allows data to be shared between seeder classes without global state or redundant database queries.
- **Environment-Specific Seeding**: Seeders can be conditionally executed based on the application environment (local, staging, production).
- **WithoutModelEvents Trait**: The `WithoutModelEvents` trait can be used to suppress model events during seeding, which is useful when events would cause side effects.

---

### Prerequisites

- **PHP 8.1 or higher** (Laravel 10+) or **PHP 8.2** (Laravel 11+).
- **A Laravel application** installed via Composer.
- **A configured database** connection in `config/database.php` or `.env`.
- **Eloquent models** (optional but recommended) for factory-based seeding.
- **Basic understanding of Artisan commands** and database migrations.

---

### Related Programming Areas

- **Database Migrations**: Define the schema that seeders populate. Migrations should be run before seeders.
- **Eloquent Model Factories**: Generate large volumes of fake model data using Faker, used within seeders.
- **Eloquent ORM**: Seeders can use Eloquent models for inserting data with proper casting, mutators, and relationships.
- **Query Builder**: Seeders can use the query builder (`DB::table()`) for direct, lightweight data insertion.
- **Testing**: The `seed()` method in Laravel's testing framework invokes seeders during feature tests.
- **Context Facade**: Laravel's `Context` system enables clean data flow between seeder classes.
- **Artisan CLI**: The `make:seeder` and `db:seed` commands are used to create and run seeders.

---

## Core Concepts

---

### 1. Seeders

#### Definitions

**Core Definition**

A seeder is a PHP class that populates a database table (or tables) with data, used for initial setup, test data generation, or lookup data management.

**Technical Definition**

A seeder extends `Illuminate\Database\Seeder` and defines a `run()` method. Seeders are stored in `database/seeders`. The `run()` method can use `DB::table()` for query builder inserts, Eloquent models for model-based inserts, or model factories for bulk data generation. The `DatabaseSeeder` class is the default entry point and can call other seeders via `$this->call()`.

**Beginner-Friendly Explanation**

A seeder is a recipe for filling your database with data. You write it once, and you can run it as many times as you need. It is like a checklist of records that should exist in your application.

#### Purposes

- To populate a database with initial data required for the application to function (e.g., roles, permissions, categories).
- To generate test data for development and staging environments.
- To provide reproducible datasets for automated testing.
- To seed lookup tables with reference data (countries, currencies, statuses).
- To automate database setup during deployment or onboarding.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
<?php
// database/seeders/DatabaseSeeder.php
namespace Database\Seeders;

use Illuminate\Database\Seeder;

class DatabaseSeeder extends Seeder
{
    public function run(): void
    {
        $this->call([
            UserSeeder::class,
            PostSeeder::class,
        ]);
    }
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `extends Seeder` | Required base class. |
| `run(): void` | The method executed when the seeder runs. |
| `$this->call([...])` | Calls other seeders in order. |
| `database/seeders/` | Default directory for seeder classes. |

**Syntax Rules**

- Seeders must extend `Illuminate\Database\Seeder`.
- The `run()` method is called by the `db:seed` command.
- Seeders are stored in `database/seeders/`.
- The `DatabaseSeeder` class is the default entry point.
- Mass assignment protection is automatically disabled during seeding.

**Constraints and Limitations**

- Seeders should not be used for schema changes; migrations handle schema.
- Seeders should be idempotent when possible to avoid duplicate data on re-run.
- Large datasets should be chunked to avoid memory exhaustion.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Basic DatabaseSeeder with Query Builder**

**File**

```php
<?php
// database/seeders/DatabaseSeeder.php
namespace Database\Seeders;

use Illuminate\Database\Seeder;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Str;

class DatabaseSeeder extends Seeder
{
    public function run(): void
    {
        DB::table('users')->insert([
            'name' => Str::random(10),
            'email' => Str::random(10).'@example.com',
            'password' => Hash::make('password'),
            'created_at' => now(),
            'updated_at' => now(),
        ]);
    }
}
```

**Command**

```bash
php artisan db:seed
```

**Expected Output (Database)**

```
users table:
| id | name       | email                  | password (hashed) | created_at |
|----|------------|------------------------|-------------------|------------|
| 1  | aBcDeFgHiJ | xYzAbC@example.com     | $2y$12$...        | 2026-10-14 |
```

**Why This Output Occurs**

The `run()` method uses the query builder to insert a single user record with a random name, email, and a hashed password. The `db:seed` command executes the `DatabaseSeeder` class.

---

**Example 2: DatabaseSeeder Orchestrating Other Seeders**

```php
<?php
// database/seeders/DatabaseSeeder.php
namespace Database\Seeders;

use Illuminate\Database\Seeder;

class DatabaseSeeder extends Seeder
{
    public function run(): void
    {
        $this->call([
            RoleSeeder::class,
            UserSeeder::class,
            PostSeeder::class,
        ]);
    }
}
```

```php
<?php
// database/seeders/RoleSeeder.php
namespace Database\Seeders;

use Illuminate\Database\Seeder;
use App\Models\Role;

class RoleSeeder extends Seeder
{
    public function run(): void
    {
        Role::create(['name' => 'admin']);
        Role::create(['name' => 'editor']);
        Role::create(['name' => 'viewer']);
    }
}
```

**Expected Output**

```
// Roles are created first, then users, then posts
// The call() method ensures the correct order
```

**Why This Output Occurs**

The `call()` method executes seeders in the order specified. This ensures that dependencies (e.g., roles needed by users) are seeded before dependent data.

---

#### Real-World Cases with Explanation

**Case 1: E-commerce Platform Initial Setup**

A new e-commerce platform needs default data: categories, product statuses, and an admin user. A `DatabaseSeeder` calls `CategorySeeder`, `StatusSeeder`, and `AdminUserSeeder` in order. This ensures the application has the necessary reference data to function.

**Explanation**: Seeders automate the initial data setup, reducing manual configuration and ensuring consistency across environments.

**Case 2: Automated Testing with Seed Data**

A testing suite uses `$this->seed()` in the `setUp()` method to populate the database with a known dataset before each test. This ensures tests run against predictable data and are isolated from production data.

**Explanation**: Seeders provide reproducible datasets for reliable automated testing.

---

### 2. Seeder Creation

#### Definitions

**Core Definition**

Seeder creation is the process of generating a new seeder class using Laravel's Artisan CLI.

**Technical Definition**

The `php artisan make:seeder` command generates a PHP class in `database/seeders` extending `Illuminate\Database\Seeder`. The generated class contains a single `run()` method stub. By default, the `WithoutModelEvents` trait is imported and used, which suppresses model events during seeding.

**Beginner-Friendly Explanation**

Creating a seeder is a one-command process. Laravel makes the file, and you fill in the data you want to insert.

#### Purposes

- To automate the generation of seeder class boilerplate.
- To ensure seeders are placed in the correct directory and namespace.
- To provide a consistent structure for seeder files.
- To include the `WithoutModelEvents` trait by default for performance and predictability.

#### Syntax Rules and Structure

**Complete General Syntax**

```bash
php artisan make:seeder UserSeeder
php artisan make:seeder PostSeeder
```

**Component Breakdown**

| Command | Effect |
|---------|--------|
| `make:seeder UserSeeder` | Creates `database/seeders/UserSeeder.php`. |
| `make:seeder PostSeeder` | Creates `database/seeders/PostSeeder.php`. |

**Syntax Rules**

- The seeder name should be PascalCase and descriptive (e.g., `UserSeeder`, `RolePermissionSeeder`).
- The `Seeder` suffix is a strong convention.
- Generated files must be edited to add the `run()` method logic.

**Constraints and Limitations**

- The command does not register the seeder in `DatabaseSeeder`; this must be done manually.
- Generated seeders are empty by default.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Generating a Seeder**

**Command**

```bash
php artisan make:seeder UserSeeder
```

**Generated File**

```php
<?php
// database/seeders/UserSeeder.php
namespace Database\Seeders;

use Illuminate\Database\Seeder;
use Illuminate\Database\Console\Seeds\WithoutModelEvents;

class UserSeeder extends Seeder
{
    use WithoutModelEvents;

    public function run(): void
    {
        //
    }
}
```

**Expected Output (Terminal)**

```
INFO  Seeder [database/seeders/UserSeeder.php] created successfully.
```

**Why This Output Occurs**

Artisan scaffolds the seeder class with the `run()` method and imports the `WithoutModelEvents` trait. The `WithoutModelEvents` trait prevents Eloquent model events from firing during seeding, which is useful when events would trigger unintended side effects.

---

#### Real-World Cases with Explanation

**Case 1: Feature Development with Seed Data**

A developer working on a new "Product" feature runs `php artisan make:seeder ProductSeeder` and immediately writes the data generation logic using factories.

**Explanation**: Artisan accelerates development by generating the necessary boilerplate.

---

### 3. `artisan make:seeder`

#### Definitions

**Core Definition**

`artisan make:seeder` is the Artisan command used to generate new seeder classes.

**Technical Definition**

The command is implemented by `Illuminate\Database\Console\Seeds\SeederMakeCommand`. It generates a PHP class file in `database/seeders`. The generated class extends `Illuminate\Database\Seeder` and includes a `run()` method stub. The `WithoutModelEvents` trait is imported and used by default.

**Beginner-Friendly Explanation**

`artisan make:seeder` is the terminal command you type to create a new seeder file.

#### Purposes

- To generate seeder classes with minimal effort.
- To provide consistent structure for seeder files.
- To enforce Laravel's naming and directory conventions.
- To include the `WithoutModelEvents` trait by default.

#### Syntax Rules and Structure

**Complete General Syntax**

```bash
php artisan make:seeder <Name>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `make:seeder` | The Artisan command. |
| `<Name>` | The seeder class name (PascalCase). |

**Syntax Rules**

- The seeder name is required.
- The command generates a file in `database/seeders/`.
- The `WithoutModelEvents` trait is included by default.

**Constraints and Limitations**

- Seeders are not automatically registered in `DatabaseSeeder`.
- Generated stubs must be filled in with data insertion logic.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Generating a Seeder and Using It**

**Command**

```bash
php artisan make:seeder PostSeeder
```

**Generated File (edited)**

```php
<?php
// database/seeders/PostSeeder.php
namespace Database\Seeders;

use Illuminate\Database\Seeder;
use App\Models\Post;

class PostSeeder extends Seeder
{
    public function run(): void
    {
        Post::factory()->count(50)->create();
    }
}
```

**Expected Output (Terminal)**

```
INFO  Seeder [database/seeders/PostSeeder.php] created successfully.
```

**Why This Output Occurs**

The command generates the seeder file. The developer then edits it to use the `Post` factory to create 50 posts.

---

#### Real-World Cases with Explanation

**Case 1: Modular Seeding Strategy**

A large application uses separate seeders for each domain: `UserSeeder`, `OrderSeeder`, `ProductSeeder`. Each is generated via `make:seeder` and called from `DatabaseSeeder` in the correct order.

**Explanation**: Modular seeders keep each seeder focused and maintainable.

---

### 4. Seeder Methods

#### Definitions

**Core Definition**

Seeder methods are the methods available on seeder classes, primarily the `run()` method which contains the seeding logic, and the `call()` method which orchestrates other seeders.

**Technical Definition**

The `run()` method is the entry point for a seeder and is called by the `db:seed` command. The `call()` method accepts an array of seeder class names and executes them in order. Seeders can also type-hint dependencies in the `run()` method signature for automatic resolution via the service container.

**Beginner-Friendly Explanation**

The `run()` method is where you write the code that inserts data. The `call()` method is how you tell the main seeder to run other seeders.

#### Purposes

- To define the data insertion logic for a seeder (`run()`).
- To orchestrate multiple seeders in a specific order (`call()`).
- To inject dependencies into seeders via the service container.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// run() method
public function run(): void
{
    // Data insertion logic
}

// call() method
public function run(): void
{
    $this->call([
        RoleSeeder::class,
        UserSeeder::class,
    ]);
}
```

**Component Breakdown**

| Method | Description |
|--------|-------------|
| `run()` | Contains the seeding logic. |
| `call(array $seeders)` | Executes other seeders in order. |
| Type-hinting | Dependencies resolved from the container. |

**Syntax Rules**

- `run()` must be defined in every seeder.
- `call()` accepts an array of seeder class names.
- Dependencies can be type-hinted in the `run()` method signature.

**Constraints and Limitations**

- `run()` is executed synchronously; long-running seeders may appear to hang.
- The `call()` method does not support conditional execution.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: `run()` with Dependency Injection**

```php
public function run(UserService $userService): void
{
    $userService->createDefaultUsers();
}
```

**Expected Output**

```
// UserService is resolved from the container and injected into the run method
```

**Why This Output Occurs**

Laravel's service container resolves the `UserService` dependency and passes it to the `run()` method.

---

**Example 2: `call()` for Orchestration**

```php
public function run(): void
{
    $this->call([
        RoleSeeder::class,
        UserSeeder::class,
        PostSeeder::class,
    ]);
}
```

**Expected Output**

```
// RoleSeeder runs first, then UserSeeder, then PostSeeder
```

**Why This Output Occurs**

The `call()` method executes seeders in the order they are listed.

---

#### Real-World Cases with Explanation

**Case 1: E-commerce Seed Order**

An e-commerce seeder calls `CategorySeeder`, `ProductSeeder`, and `OrderSeeder` in that order because orders depend on products, which depend on categories.

**Explanation**: The `call()` method ensures dependencies are seeded before dependent data.

---

### 5. Seeder Execution

#### Definitions

**Core Definition**

Seeder execution is the process of running seeders to populate the database, typically via the `db:seed` Artisan command or the `migrate:fresh --seed` command.

**Technical Definition**

The `db:seed` command resolves the specified seeder class (defaulting to `DatabaseSeeder`) and invokes its `run()` method. The `--class` option specifies a particular seeder. The `migrate:fresh --seed` command drops all tables, re-runs migrations, and then runs the `DatabaseSeeder`. Seeders can also be executed programmatically via `Artisan::call('db:seed')` or in tests via `$this->seed()`.

**Beginner-Friendly Explanation**

You run seeders using Artisan commands in your terminal. `php artisan db:seed` runs the main seeder. You can also run a specific seeder with `--class`. And if you want a completely fresh database, `migrate:fresh --seed` drops everything, recreates the tables, and seeds them.

#### Purposes

- To populate the database with seed data.
- To run a specific seeder class independently.
- To combine migrations and seeding in a single command.
- To seed the database programmatically or during testing.

#### Syntax Rules and Structure

**Complete General Syntax**

```bash
# Run the DatabaseSeeder
php artisan db:seed

# Run a specific seeder
php artisan db:seed --class=UserSeeder

# Fresh migration and seed
php artisan migrate:fresh --seed

# Force seed in production
php artisan db:seed --force
```

**Component Breakdown**

| Command | Description |
|---------|-------------|
| `db:seed` | Runs the `DatabaseSeeder`. |
| `--class=UserSeeder` | Runs a specific seeder. |
| `migrate:fresh --seed` | Drops all tables, re-migrates, and seeds. |
| `--force` | Runs seeding in production. |

**Syntax Rules**

- `db:seed` defaults to `DatabaseSeeder`.
- `--class` specifies a single seeder.
- `migrate:fresh --seed` is destructive (drops all tables).
- `--force` is required in production.

**Constraints and Limitations**

- `migrate:fresh` deletes all existing data.
- Seeders run synchronously; large seeders may take time.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Running the DatabaseSeeder**

**Command**

```bash
php artisan db:seed
```

**Expected Output (Terminal)**

```
Seeding: Database\Seeders\DatabaseSeeder
Seeding: Database\Seeders\UserSeeder
Seeding: Database\Seeders\PostSeeder
Database seeding completed successfully.
```

**Why This Output Occurs**

The `db:seed` command runs the `DatabaseSeeder`, which calls `UserSeeder` and `PostSeeder` in order. The output confirms successful completion.

---

**Example 2: Running a Specific Seeder**

**Command**

```bash
php artisan db:seed --class=UserSeeder
```

**Expected Output (Terminal)**

```
Seeding: Database\Seeders\UserSeeder
Database seeding completed successfully.
```

**Why This Output Occurs**

The `--class` option restricts execution to the specified seeder.

---

**Example 3: Fresh Migration with Seed**

**Command**

```bash
php artisan migrate:fresh --seed
```

**Expected Output (Terminal)**

```
Dropped all tables successfully.
Migration table created successfully.
Migrating: 2014_10_12_000000_create_users_table
Migrated:  2014_10_12_000000_create_users_table
...
Seeding: Database\Seeders\DatabaseSeeder
Database seeding completed successfully.
```

**Why This Output Occurs**

The `migrate:fresh` command drops all tables and re-runs migrations. The `--seed` flag then runs the `DatabaseSeeder`.

---

#### Real-World Cases with Explanation

**Case 1: Local Development Reset**

A developer runs `php artisan migrate:fresh --seed` to reset their local database to a clean state with fresh test data.

**Explanation**: The combined command ensures the database is rebuilt and seeded in one step.

**Case 2: Production Deployment with `--force`**

A deployment script runs `php artisan db:seed --force` to seed required lookup data in production.

**Explanation**: The `--force` flag bypasses the production confirmation prompt.

---

### 6. Seeder Factories

#### Definitions

**Core Definition**

Seeder factories are Eloquent model factories used within seeders to generate large volumes of realistic test data using the Faker PHP library.

**Technical Definition**

Model factories extend `Illuminate\Database\Eloquent\Factories\Factory` and define a `definition()` method returning an array of attribute values. Within a seeder, factories are invoked via `Model::factory()->count(N)->create()`. Factories support states, sequences, callbacks (`afterMaking`, `afterCreating`), and relationship methods (`has`, `for`, `recycle`).

**Beginner-Friendly Explanation**

Factories are recipes for creating fake model data. Instead of manually writing every attribute for every record, you use a factory to generate realistic data automatically. Seeders use factories to create many records quickly.

#### Purposes

- To generate large volumes of realistic test data.
- To reduce boilerplate when creating model records.
- To ensure consistent data generation across environments.
- To support complex relationships and states in generated data.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// In a seeder
User::factory()->count(50)->create();

// With relationships
User::factory()
    ->count(50)
    ->hasPosts(3)
    ->create();

// With state
User::factory()->count(10)->admin()->create();
```

**Component Breakdown**

| Method | Description |
|--------|-------------|
| `factory()` | Returns a factory instance for the model. |
| `count(N)` | Creates N records. |
| `create()` | Persists the records to the database. |
| `hasPosts(N)` | Creates N related posts per user. |
| `admin()` | Applies a factory state. |

**Syntax Rules**

- Factories are defined in `database/factories/`.
- `definition()` returns an array of default attributes.
- `create()` persists records; `make()` creates in-memory instances.
- Relationship methods (`has`, `for`) create related records.

**Constraints and Limitations**

- Factory-generated data is random and not suitable for production.
- Complex relationships may require careful ordering to avoid foreign key violations.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Using a Factory in a Seeder**

**Factory**

```php
<?php
// database/factories/UserFactory.php
namespace Database\Factories;

use Illuminate\Database\Eloquent\Factories\Factory;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Str;

class UserFactory extends Factory
{
    public function definition(): array
    {
        return [
            'name' => fake()->name(),
            'email' => fake()->unique()->safeEmail(),
            'password' => Hash::make('password'),
            'remember_token' => Str::random(10),
        ];
    }
}
```

**Seeder**

```php
<?php
// database/seeders/UserSeeder.php
namespace Database\Seeders;

use Illuminate\Database\Seeder;
use App\Models\User;

class UserSeeder extends Seeder
{
    public function run(): void
    {
        User::factory()->count(50)->create();
    }
}
```

**Expected Output**

```
// 50 users are created in the users table with random names, emails, and hashed passwords
```

**Why This Output Occurs**

The `User::factory()->count(50)->create()` call generates 50 user records using the factory's `definition()` method for attribute values.

---

**Example 2: Seeding with Relationships**

```php
User::factory()
    ->count(50)
    ->hasPosts(3)
    ->create();
```

**Expected Output**

```
// 50 users are created, and each user has 3 associated posts
```

**Why This Output Occurs**

The `hasPosts(3)` method instructs the factory to create 3 related `Post` records for each `User` record.

---

#### Real-World Cases with Explanation

**Case 1: Social Network Development**

A social network application uses factories in seeders to create 100 users, each with 10 posts and 50 comments, providing a realistic dataset for development and testing.

**Explanation**: Factories enable rapid generation of complex, interconnected data.

---

### 7. Seeder Best Practices

#### Definitions

**Core Definition**

Seeder best practices are the recommended guidelines for designing, implementing, and maintaining seeders to maximise maintainability, performance, and safety.

**Technical Definition**

Best practices encompass idempotency, environment awareness, separation of concerns, chunking, context sharing, and using the appropriate insertion method (query builder, Eloquent, factories). They aim to balance speed, reliability, and maintainability.

**Beginner-Friendly Explanation**

Best practices are the tips experienced developers follow to keep their seeders clean, safe, and efficient.

#### Key Best Practices

1. **Make Seeders Idempotent**: Use `firstOrCreate()`, `updateOrCreate()`, or `upsert()` to ensure seeders can be run multiple times without creating duplicate records. This is essential for lookup data and production seeding.

2. **Separate Lookup Data from Test Data**: Use seeders for reference data (roles, permissions, countries) that should exist in all environments. Use factories for test data that should only exist in local and staging environments.

3. **Use Factories for Bulk Data**: Use `Model::factory()->count(N)->create()` instead of loops with `Model::create()` to reduce boilerplate and improve performance.

4. **Chunk Large Datasets**: For seeding thousands of records, use `chunk()` or batch inserts to avoid memory exhaustion and long transaction times.

5. **Use `WithoutModelEvents` for Performance**: The `WithoutModelEvents` trait suppresses model events during seeding, which is useful when events would trigger unnecessary side effects (e.g., sending emails, logging).

6. **Use `Context` for Data Sharing**: Laravel's `Context` facade allows seeders to share data without global state or redundant database queries. Use `#[Context('key')]` attribute injection in dependent seeders.

7. **Organise Seeders by Domain**: Create separate seeder classes for each domain (e.g., `UserSeeder`, `ProductSeeder`, `OrderSeeder`) and orchestrate them via `DatabaseSeeder`.

8. **Never Seed in Production Without `--force`**: The `--force` flag is required to run seeders in production. Use it deliberately and only for required lookup data.

9. **Consider Migrations for Static Data**: For truly static data that rarely changes (e.g., country codes), consider inserting it directly in a migration. Migrations are deterministic and append-only, unlike seeders.

10. **Test Seeders**: Write tests that run seeders and assert the expected database state. Use `$this->seed()` in feature tests to populate the database.

11. **Use `DB::table()` for Direct Inserts**: When model events, casts, or accessors are not needed, use `DB::table()` for faster, simpler inserts.

12. **Document Seeder Dependencies**: Clearly document the order in which seeders should run, especially when using `call()`. Dependencies (e.g., roles before users) must be seeded first.

#### Multiple Annotated Complete Step by Step Code Example and Expected Output

**Example 1: Idempotent Seeder with Context Sharing**

**Role Seeder (Idempotent)**

```php
<?php
// database/seeders/RoleSeeder.php
namespace Database\Seeders;

use Illuminate\Database\Seeder;
use App\Models\Role;
use Illuminate\Support\Facades\Context;

class RoleSeeder extends Seeder
{
    public function run(): void
    {
        $adminRole = Role::firstOrCreate(['name' => 'admin']);
        $editorRole = Role::firstOrCreate(['name' => 'editor']);

        Context::add('admin_role', $adminRole);
        Context::add('editor_role', $editorRole);
    }
}
```

**User Seeder (Uses Context)**

```php
<?php
// database/seeders/UserSeeder.php
namespace Database\Seeders;

use Illuminate\Database\Seeder;
use App\Models\User;
use App\Models\Role;
use Illuminate\Container\Attributes\Context;

class UserSeeder extends Seeder
{
    public function run(
        #[Context('admin_role')] Role $adminRole,
    ): void {
        $admin = User::firstOrCreate(
            ['email' => 'admin@example.com'],
            [
                'name' => 'Admin User',
                'password' => bcrypt('password'),
            ]
        );

        $admin->roles()->syncWithoutDetaching([$adminRole->id]);

        User::factory()->count(20)->create();
    }
}
```

**Expected Output**

```
// Roles are created (or retrieved if they already exist)
// Admin user is created (or retrieved) and assigned the admin role
// 20 additional users are created with the default role
// Running the seeder multiple times does not create duplicate records
```

**Why This Output Occurs**

The `firstOrCreate()` method ensures idempotency: if the role or user already exists, it is returned instead of being duplicated. The `Context` facade shares the `admin_role` instance between seeders without redundant queries. The `User::factory()->count(20)->create()` generates additional test users.

---

#### Real-World Cases with Explanation

**Case 1: SaaS Platform Lookup Data**

A SaaS platform seeds roles, permissions, and subscription plans in production using idempotent seeders. These seeders are run during deployment and use `firstOrCreate()` to ensure they can be safely re-run without duplicating data.

**Explanation**: Idempotent seeders are safe for repeated execution, making them suitable for deployment pipelines.

**Case 2: Multi-Environment Seeding Strategy**

A development team uses `DatabaseSeeder` to call `RoleSeeder` (lookup data) and `DemoDataSeeder` (test data). In production, only `RoleSeeder` is called. In local and staging, both are called.

**Explanation**: Environment-aware seeding ensures production databases contain only necessary reference data, not test data.

---

## References

- Database: Seeding – Laravel 13.x Documentation — https://laravel.com/docs/13.x/seeding
- Database: Seeding – Laravel 12.x Documentation — https://laravel.com/docs/12.x/seeding
- Database: Seeding – Laravel 11.x Documentation — https://laravel.com/docs/11.x/seeding
- Database: Seeding – Laravel 10.x Documentation — https://laravel.com/docs/10.x/seeding
- Eloquent: Factories – Laravel 13.x Documentation — https://laravel.com/docs/13.x/eloquent-factories
- Database Testing: Running Seeders – Laravel 13.x Documentation — https://laravel.com/docs/13.x/database-testing#running-seeders
- Streamlined Data Flow Between Laravel Seeders Using Context – Laravel News — https://laravel-news.com/context-attribute
- Laravel Factories and Seeders: All You Need to Know – Laravel Daily — https://laraveldaily.com/post/laravel-factories-seeders
- Run seeders from migrations – Spatie — https://spatie.be/blog/run-seeders-from-migrations
- Seeding Roles & Permissions – Spatie Laravel Permission — https://mintlify.wiki/spatie/laravel-permission/best-practices/seeders
- Laravel Seeder Generator – Laravel News — https://laravel-news.com/laravel-seeder-generator
- Seeder Class – Laravel API Reference — https://api.laravel.com/docs/12.x/Illuminate/Database/Seeder.html
- WithoutModelEvents Trait – Laravel API Reference — https://api.laravel.com/docs/12.x/Illuminate/Database/Console/Seeds/WithoutModelEvents.html

---

**Version Note**: This cheat sheet is based on Laravel 10.x through 13.x documentation. Key version-specific features include: the `WithoutModelEvents` trait is included by default in Laravel 10+; the `Context` facade for seeder data sharing is available in Laravel 11+; the `#[Context]` attribute for dependency injection is available in Laravel 12+. In Laravel 8 and earlier, seeders were stored in `database/seeds` instead of `database/seeders`. Developers should always consult the documentation for their specific Laravel version.