# Laravel Database Seeders: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Database seeders are classes that populate your database with data, primarily used for development and testing environments.

**Technical Definition:** A seeder is a class extending `Illuminate\Database\Seeder` that implements a `run()` method. Seeders are stored in `database/seeders/` and are executed via the `db:seed` Artisan command. The central `DatabaseSeeder` class orchestrates the execution of individual seeders via the `call()` method. Mass assignment protection is automatically disabled during seeding.

**Beginner-Friendly Explanation:** Seeders are the "fill in the blanks" tool for your database. After you create your tables with migrations, seeders let you populate them with realistic test data—like a handful of users, some blog posts, or a list of categories—so your app has something to work with during development.

### Key Characteristics

- **Class-Based:** Each seeder is a dedicated PHP class in `database/seeders/`.
- **Single Entry Point:** The `run()` method is called when the seeder executes.
- **Orchestrated Execution:** `DatabaseSeeder` calls other seeders via `$this->call()`.
- **Factory Integration:** Seeders typically use model factories to generate bulk data.
- **Mass Assignment Disabled:** Protection is automatically off during seeding.
- **Idempotent-Friendly:** Can be written to safely run multiple times without errors.

### Prerequisites

- A Laravel application with configured database connection.
- Models and migrations already created.
- Model factories defined for data generation (optional but recommended).
- Understanding of Eloquent models and factories.

### Related Programming Areas

- **Database Migrations:** Define the schema seeders populate.
- **Model Factories:** Generate realistic fake data for seeding.
- **Artisan Commands:** `db:seed` executes seeders from the command line.
- **Environment Configuration:** `APP_ENV` determines seeding strategies.

### Core Concepts / Features

1. Crafting Seeder Classes (`run()` Method)
2. Orchestrating via `DatabaseSeeder` (`$this->call()`)
3. Environment-Specific Strategies
4. Idempotent Seeders
5. Artisan Execution Commands

---

## 1. Crafting Seeder Classes and Utilizing the `run()` Method

### Definitions

**Core Definition:** A seeder class is a PHP class containing a `run()` method that inserts data into the database when executed.

**Technical Definition:** Seeder classes extend `Illuminate\Database\Seeder` and are generated via `php artisan make:seeder`. The `run()` method is invoked by the `db:seed` command. Within `run()`, you can use the query builder, Eloquent models, or model factories to insert data. Dependencies type-hinted in the `run()` signature are resolved via the service container.

**Beginner-Friendly Explanation:** The `run()` method is where you write the code that inserts data. You can write raw database inserts, use Eloquent to create models, or use factories to generate lots of fake records at once.

### Purposes

- To populate the database with initial or test data.
- To define a single, reusable class per data type.
- To leverage model factories for bulk data generation.
- To type-hint dependencies for automatic resolution.

### Syntax Rules and Structure

**Complete General Syntax:**

```php
namespace Database\Seeders;

use Illuminate\Database\Seeder;

class UserSeeder extends Seeder
{
    public function run(): void
    {
        // Insert data here
    }
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `extends Seeder` | Base seeder class. |
| `run(): void` | Method invoked by `db:seed`. |
| `UserSeeder` | Convention: `{ModelName}Seeder`. |

**Complete General Syntax — Using Factories:**

```php
use App\Models\User;

public function run(): void
{
    User::factory()
        ->count(50)
        ->hasPosts(1)
        ->create();
}
```

**Syntax Rules:**
- Seeder classes must be in the `Database\Seeders` namespace.
- The `run()` method is required.
- Mass assignment protection is automatically disabled during seeding.

**Constraints and Limitations:**
- Seeders run in the order specified by `DatabaseSeeder`.
- Foreign key constraints may require careful ordering.

### Multiple Annotated Complete Code Examples

**Example 1: Basic Seeder with Query Builder**

```php
<?php
namespace Database\Seeders;

use Illuminate\Database\Seeder;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Str;

class UserSeeder extends Seeder
{
    public function run(): void
    {
        DB::table('users')->insert([
            'name' => 'Admin User',
            'email' => 'admin@example.com',
            'password' => Hash::make('password'),
        ]);
    }
}
```

**Expected Output:** A single admin user is inserted into the `users` table.

**Why:** The query builder's `insert()` method executes a direct SQL INSERT statement.

---

**Example 2: Seeder Using Model Factories**

```php
<?php
namespace Database\Seeders;

use App\Models\User;
use Illuminate\Database\Seeder;

class UserSeeder extends Seeder
{
    public function run(): void
    {
        User::factory()
            ->count(50)
            ->hasPosts(1)
            ->create();
    }
}
```

**Expected Output:** 50 users are created, each with one associated post.

**Why:** The factory generates realistic fake data. The `hasPosts(1)` method creates a post for each user.

---

**Example 3: Seeder with Dependency Injection**

```php
public function run(UserService $userService): void
{
    $userService->createDefaultUsers();
}
```

**Expected Output:** The `UserService` is resolved from the container and used within the seeder.

**Why:** Laravel automatically resolves type-hinted dependencies in the `run()` method signature.

### Real-World Cases

- **Development Setup:** Seed a few users and posts so developers have data to work with.
- **Testing:** Generate specific data sets for feature tests.
- **Staging Environments:** Populate staging with realistic data for QA.

### References

- Laravel Database Seeding — https://laravel.com/framework/docs/master/seeding

---

## 2. Orchestrating Seeders via `DatabaseSeeder` (`$this->call()`)

### Definitions

**Core Definition:** The `DatabaseSeeder` class is the central orchestrator that calls individual seeder classes in a defined order using the `call()` method.

**Technical Definition:** By default, `db:seed` runs the `DatabaseSeeder` class. Within its `run()` method, the `call()` method accepts an array of seeder classes to execute in sequence. This allows breaking seeding logic into smaller, focused files.

**Beginner-Friendly Explanation:** Instead of putting all your seeding code in one giant file, you split it into small seeders and tell `DatabaseSeeder` which ones to run and in what order.

### Purposes
- To break seeding logic into manageable, focused classes.
- To control the execution order of seeders.
- To allow selective seeding via `--class`.

### Syntax Rules and Structure

**Complete General Syntax:**

```php
public function run(): void
{
    $this->call([
        UserSeeder::class,
        PostSeeder::class,
        CommentSeeder::class,
    ]);
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `$this->call([...])` | Executes seeders in the provided order. |
| `UserSeeder::class` | Seeder class to execute. |

**Syntax Rules:**
- Seeders are executed in array order.
- Foreign key dependencies require correct ordering.

### Multiple Annotated Complete Code Examples

**Example 1: Basic Orchestration**

```php
public function run(): void
{
    $this->call([
        UserSeeder::class,
        PostSeeder::class,
    ]);
}
```

**Expected Output:** `UserSeeder` runs first, then `PostSeeder`.

**Why:** The `call()` method iterates the array and executes each seeder.

---

**Example 2: Muting Model Events**

```php
use Illuminate\Database\Console\Seeds\WithoutModelEvents;

class DatabaseSeeder extends Seeder
{
    use WithoutModelEvents;

    public function run(): void
    {
        $this->call([
            UserSeeder::class,
        ]);
    }
}
```

**Expected Output:** Model events are not dispatched during seeding.

**Why:** The `WithoutModelEvents` trait suppresses event dispatch.

### Real-World Cases
- **E-commerce:** `CategorySeeder` → `ProductSeeder` → `ReviewSeeder`.
- **CMS:** `UserSeeder` → `PostSeeder` → `CommentSeeder`.

### References
- Laravel Database Seeding — https://laravel.com/framework/docs/master/seeding

---

## 3. Environment-Specific Strategies

### Definitions

**Core Definition:** Environment-specific seeding strategies use conditional logic to prevent destructive or inappropriate data insertion in production environments.

**Technical Definition:** The `app()->environment()` helper returns the current application environment (`local`, `testing`, `production`, etc.). Seeders can check this value to conditionally execute logic. Additionally, `migrate:fresh` and `db:seed` prompt for confirmation in production; the `--force` flag bypasses this.

**Beginner-Friendly Explanation:** You don't want test data in production. Environment checks let seeders skip themselves or run different logic based on where the app is running.

### Purposes
- To prevent seeding test data in production.
- To run different seeders per environment.
- To bypass production prompts with `--force`.

### Syntax Rules and Structure

```php
public function run(): void
{
    if (app()->environment('production')) {
        return; // Skip in production
    }

    // Seed test data
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `app()->environment('production')` | Returns true if in production. |
| `--force` | Bypasses production confirmation prompts. |

### Multiple Annotated Complete Code Examples

**Example 1: Production Safeguard**

```php
public function run(): void
{
    if (app()->environment() === 'production') {
        $this->command->warn('Skipping seeder in production.');
        return;
    }

    User::factory()->count(50)->create();
}
```

**Expected Output:** In production, the seeder logs a warning and exits. In local, 50 users are created.

**Why:** The environment check prevents accidental production data pollution.

---

**Example 2: Using `--force`**

```bash
php artisan db:seed --force
```

**Expected Output:** The seeder runs without the production confirmation prompt.

**Why:** The `--force` flag bypasses interactive confirmation in non-interactive environments.

### Real-World Cases
- **Production Deployments:** Seed only reference data (countries, currencies).
- **Local Development:** Seed generous test data sets.
- **Testing:** Seed minimal, deterministic data.

### References
- Fix Production Command Failures — https://laravel.com/cloud/docs/knowledge-base/command-failed-prod-app#1

---

## 4. Idempotent Seeders

### Definitions

**Core Definition:** An idempotent seeder is one that can run multiple times without causing duplicate-key or unique-constraint errors.

**Technical Definition:** Idempotent seeders use `insertOrIgnore()`, `updateOrCreate()`, `firstOrCreate()`, or explicit existence checks to ensure records are only created once. This is critical for production seeders that may be re-run during deployments.

**Beginner-Friendly Explanation:** Idempotent means "safe to run again." If you run the seeder twice, you still get the same result—no duplicate records, no errors.

### Purposes
- To allow safe re-execution during deployments.
- To avoid duplicate-key errors on unique columns.
- To support incremental data additions.

### Syntax Rules and Structure

**Complete General Syntax — `insertOrIgnore()`:**

```php
DB::table('promo_codes')->insertOrIgnore([
    ['code' => 'BLACKFRIDAY2024'],
    ['code' => 'CHRISTMAS2024'],
]);
```

**Complete General Syntax — `updateOrCreate()`:**

```php
Role::updateOrCreate(
    ['name' => 'admin'],
    ['description' => 'Administrator role']
);
```

**Syntax Rules:**
- `insertOrIgnore()` silently discards duplicate-key violations.
- `updateOrCreate()` updates existing or creates new records.

### Multiple Annotated Complete Code Examples

**Example 1: Promo Code Seeder with `insertOrIgnore()`**

```php
class PromoCodeSeeder extends Seeder
{
    protected array $codes = [
        'BLACKFRIDAY2024',
        'CHRISTMAS2024',
        'BOXINGDAY2024',
    ];

    public function run(): void
    {
        foreach ($this->codes as $code) {
            DB::table('promo_codes')->insertOrIgnore([
                'code' => $code,
            ]);
        }
    }
}
```

**Expected Output:** Running multiple times only inserts missing codes; duplicates are silently ignored.

**Why:** `insertOrIgnore()` discards duplicate-key violations, making the seeder idempotent.

---

**Example 2: Role Seeder with `updateOrCreate()`**

```php
class RoleSeeder extends Seeder
{
    public function run(): void
    {
        $roles = [
            ['name' => 'admin', 'description' => 'Administrator'],
            ['name' => 'editor', 'description' => 'Editor'],
            ['name' => 'viewer', 'description' => 'Viewer'],
        ];

        foreach ($roles as $role) {
            Role::updateOrCreate(
                ['name' => $role['name']],
                ['description' => $role['description']]
            );
        }
    }
}
```

**Expected Output:** Roles are created or updated, never duplicated.

**Why:** `updateOrCreate()` checks for existence by the first array, then updates or creates.

### Real-World Cases
- **Reference Data:** Countries, currencies, statuses seeded idempotently.
- **Permissions:** Spatie permissions seeded with `findOrCreate()`.
- **Feature Flags:** Configuration values seeded safely.

### References
- Seeder inheritance — https://laracasts.com/discuss/channels/laravel/seeder-inheritance
- Seeding Roles & Permissions — https://mintlify.wiki/spatie/laravel-permission/advanced-usage/seeding

---

## 5. Executing Seeders via Artisan Command Line Flags

### Definitions

**Core Definition:** Artisan provides several commands for executing seeders, with flags for specific classes, production bypass, and combined migration-seeding workflows.

**Technical Definition:** The `db:seed` command runs the `DatabaseSeeder` by default. The `--class` flag specifies a particular seeder. `migrate:fresh --seed` drops all tables, re-runs migrations, and seeds in one command. The `--force` flag bypasses production confirmation prompts.

**Beginner-Friendly Explanation:** You run seeders from the command line. The most common is `php artisan db:seed`, but you can also combine it with migrations for a complete database rebuild.

### Purposes
- To execute seeders from the command line.
- To run specific seeders with `--class`.
- To rebuild and seed the database in one command.
- To bypass production prompts with `--force`.

### Syntax Rules and Structure

**Complete General Syntax — Basic Seeding:**

```bash
php artisan db:seed
php artisan db:seed --class=UserSeeder
```

**Complete General Syntax — Migrate and Seed:**

```bash
php artisan migrate:fresh --seed
php artisan migrate:fresh --seed --force
```

**Complete General Syntax — Production:**

```bash
php artisan db:seed --force
php artisan migrate:fresh --force
```

**Component Breakdown:**

| Command/Flag | Description |
|--------------|-------------|
| `db:seed` | Runs `DatabaseSeeder`. |
| `--class=UserSeeder` | Runs a specific seeder. |
| `migrate:fresh` | Drops all tables and re-migrates. |
| `--seed` | Seeds after migration. |
| `--force` | Bypasses production prompts. |

**Syntax Rules:**
- `migrate:fresh --seed` is destructive—drops all tables.
- `--force` is required in non-interactive production environments.
- `--class` can be used with `db:seed` only.

**Constraints and Limitations:**
- `migrate:fresh` deletes all data.
- `--force` bypasses safety prompts—use with caution.

### Multiple Annotated Complete Code Examples

**Example 1: Standard Seeding**

```bash
# Run the default DatabaseSeeder
php artisan db:seed

# Run a specific seeder
php artisan db:seed --class=UserSeeder
```

**Expected Output:** The specified seeder(s) execute and populate the database.

**Why:** `db:seed` resolves and runs the seeder class's `run()` method.

---

**Example 2: Rebuild and Seed**

```bash
# Drop all tables, re-migrate, and seed
php artisan migrate:fresh --seed

# In production (bypass prompt)
php artisan migrate:fresh --seed --force
```

**Expected Output:** All tables are dropped, migrations run, and seeders execute.

**Why:** The `--seed` flag triggers `db:seed` after migration completes.

### Real-World Cases
- **Local Development Reset:** `php artisan migrate:fresh --seed` to start clean.
- **CI/CD Pipelines:** `php artisan migrate --seed --force` for automated setup.
- **Production Reference Data:** `php artisan db:seed --class=CountrySeeder --force`.

### References
- Laravel Database Seeding — https://laravel.com/framework/docs/master/seeding
- Fix Production Command Failures — https://laravel.com/cloud/docs/knowledge-base/command-failed-prod-app#1
- Laravel 6.x Seeding — https://laravel.com/framework/docs/6.x/seeding

---

## References

- Laravel Database Seeding (Master) — https://laravel.com/framework/docs/master/seeding
- Fix Production Command Failures — https://laravel.com/cloud/docs/knowledge-base/command-failed-prod-app#1
- Laravel 6.x Seeding — https://laravel.com/framework/docs/6.x/seeding
- Seeder inheritance (Laracasts) — https://laracasts.com/discuss/channels/laravel/seeder-inheritance
- Seeding Roles & Permissions (Spatie) — https://mintlify.wiki/spatie/laravel-permission/advanced-usage/seeding
- Conditional Seeders Package — https://root.packagist.org/packages/krzysztofrewak/laravel-conditional-seeders
- SuperSeeder Package — https://packagist.org/packages/riftweb/superseeder
- OI Laravel Seeds — https://packagist.org/packages/oi-lab/oi-laravel-seeds
- SeedFlow Package — https://packagist.org/packages/webed/seed-flow
- Eloquent Factories — https://laravel.com/framework/docs/eloquent-factories