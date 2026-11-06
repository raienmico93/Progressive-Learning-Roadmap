# Laravel Database Testing: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Laravel database testing is the practice of writing automated tests that interact with the application's database to verify that models, queries, relationships, and persistence logic behave correctly under controlled, isolated conditions.

**Technical Definition**

Laravel provides a suite of tools for database testing: isolation traits (`RefreshDatabase`, `DatabaseTransactions`, `DatabaseMigrations`, `DatabaseTruncation`) that reset database state between tests; model factories that generate realistic test data using Faker; seeders that populate the database with known records; and a rich set of assertion helpers (`assertDatabaseHas`, `assertDatabaseMissing`, `assertDatabaseCount`, `assertModelExists`, etc.) that verify persisted state. These tools integrate with Pest and PHPUnit, allowing tests to run against an in-memory SQLite database or a dedicated test database without affecting production data.

**Beginner-Friendly Explanation**

Database testing is how you prove that your app's data layer actually works. Instead of manually clicking through your app and checking that a user was saved, you write a test that creates a user, asks the database "is this user really there?", and gets an automatic yes or no. Laravel gives you tools to reset the database after every test so one test never messes up another, factories to quickly create fake but realistic data, and assertions to check that records exist, are missing, or have the right values.

---

### Key Characteristics

- **Isolation First**: Every database test must start from a clean, known state. Laravel's traits wrap tests in transactions or reset the schema to guarantee isolation.
- **`RefreshDatabase` as the Default**: The recommended trait for most tests—it migrates once per test run and wraps each test in a rolled-back transaction.
- **Factories Over Manual Inserts**: Model factories produce realistic, maintainable test data and support states, sequences, and relationships.
- **Seeders for Known State**: Seeders populate reference data (roles, statuses) that tests depend on, and can be invoked directly within tests.
- **Rich Assertion API**: `assertDatabaseHas`, `assertDatabaseMissing`, `assertDatabaseCount`, `assertDatabaseEmpty`, `assertModelExists`, `assertModelMissing`, `assertSoftDeleted`, and `assertNotSoftDeleted` cover the full spectrum of persistence verification.
- **Transaction-Aware**: Traits respect `$connectionsToTransact` for multi-database applications, so transaction-based isolation applies only to supported connections.

---

### Prerequisites

- **PHP 8.1+** (Laravel 10+) or **PHP 8.2+** (Laravel 11+).
- **A Laravel application** with Eloquent models, migrations, factories, and seeders.
- **A configured test database**—SQLite in-memory is the recommended default for speed; PostgreSQL/MySQL may be needed if tests rely on database-specific features.
- **Pest or PHPUnit** installed (both ship with Laravel).
- **`doctrine/dbal`** if using `DatabaseTruncation` or modifying columns on SQLite.

---

### Related Programming Areas

- **Feature Testing**: Database tests are often feature tests that exercise HTTP endpoints and verify persistence.
- **Unit Testing**: Pure logic tests do not touch the database; database tests complement them by validating the data layer.
- **Eloquent**: Factories, relationships, and model events are central to database testing.
- **Migrations**: The schema under test is created by migrations; `RefreshDatabase` and `DatabaseMigrations` interact directly with the migration system.
- **Seeders**: Reference data and initial state are created by seeders, which can be invoked in tests.

---

## Core Concepts

---

### 1. Database Testing

#### Definitions

**Core Definition**

Database testing is the verification that an application's interaction with its database—queries, inserts, updates, deletes, relationships, and constraints—produces the correct persisted state.

**Technical Definition**

A database test sends a request or invokes application code, then asserts on the database using helpers like `assertDatabaseHas()`, `assertDatabaseCount()`, or `assertModelExists()`. The test runs against a dedicated test database, isolated by a trait (`RefreshDatabase`, `DatabaseTransactions`, `DatabaseMigrations`, or `DatabaseTruncation`), and uses factories and/or seeders to arrange the initial state.

**Beginner-Friendly Explanation**

A database test is a script that says: “Do something in the app, then check the database to make sure it happened correctly.”

#### Purposes

- To verify that create, read, update, and delete operations persist the correct data.
- To ensure that relationships and foreign keys behave as expected.
- To catch regressions in queries, scopes, and model events.
- To confirm that validation and authorization correctly prevent or allow database changes.

#### Syntax Rules and Structure

**Complete General Syntax (Pest)**

```php
<?php
use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;

uses(RefreshDatabase::class);

test('a user can be created', function () {
    $user = User::factory()->create(['email' => 'alice@example.com']);

    $this->assertDatabaseHas('users', [
        'email' => 'alice@example.com',
    ]);
});
```

**Complete General Syntax (PHPUnit)**

```php
<?php
namespace Tests\Feature;

use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class UserTest extends TestCase
{
    use RefreshDatabase;

    public function test_a_user_can_be_created(): void
    {
        $user = User::factory()->create(['email' => 'alice@example.com']);

        $this->assertDatabaseHas('users', [
            'email' => 'alice@example.com',
        ]);
    }
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `uses(RefreshDatabase::class)` / `use RefreshDatabase` | Isolates database state per test. |
| `Model::factory()->create()` | Creates and persists a model instance. |
| `assertDatabaseHas()` | Asserts a row matching the given column values exists. |

**Constraints and Limitations**

- Unit tests (extending `PHPUnit\Framework\TestCase`) do not boot Laravel and cannot access the database; database tests must be feature tests or extend `Tests\TestCase`.
- Tests that forget the isolation trait can leak data into subsequent tests, causing flaky failures.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Verifying a Record Exists**

**Code**

```php
<?php
use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;

uses(RefreshDatabase::class);

test('a created user exists in the database', function () {
    $user = User::factory()->create(['name' => 'Alice']);

    $this->assertDatabaseHas('users', [
        'id'   => $user->id,
        'name' => 'Alice',
    ]);
});
```

**Expected Output**

```
PASS  Tests\Feature\UserDatabaseTest
✓ a created user exists in the database
```

**Why This Output Occurs**

The factory persists a user with the name “Alice.” `assertDatabaseHas()` queries the `users` table for a row matching both the ID and the name, and the assertion passes because the record exists.

---

**Example 2: Verifying a Record Is Missing**

**Code**

```php
test('a deleted user no longer exists', function () {
    $user = User::factory()->create();
    $user->delete();

    $this->assertDatabaseMissing('users', ['id' => $user->id]);
});
```

**Expected Output**

```
PASS  Tests\Feature\UserDatabaseTest
✓ a deleted user no longer exists
```

**Why This Output Occurs**

The user is created and then deleted. `assertDatabaseMissing()` confirms that no row with the deleted user’s ID remains in the `users` table.

---

#### Real-World Cases with Explanation

**Case 1: Blog Post Creation Endpoint**

A feature test POSTs to `/posts`, then asserts that a row exists in the `posts` table with the submitted title and the authenticated user’s ID. This verifies that the controller, validation, and Eloquent persistence all work together correctly.

**Explanation**: Database testing bridges the gap between HTTP behaviour and persisted state, catching bugs where a request “succeeds” but nothing is actually saved.

---

### 2. `RefreshDatabase`

#### Definitions

**Core Definition**

`RefreshDatabase` is Laravel’s recommended trait for database isolation: it runs migrations once per test run and wraps each test in a database transaction that is rolled back after the test finishes.

**Technical Definition**

The trait checks whether the database schema is up to date. If it is, it does not re-migrate; it simply begins a transaction before each test and rolls it back in `tearDown()`. If the schema is stale, it runs `migrate:fresh` once. This approach is significantly faster than re-migrating for every test and is compatible with in-memory SQLite as well as traditional databases.

**Beginner-Friendly Explanation**

`RefreshDatabase` gives every test a clean database without the cost of rebuilding the whole schema each time. It wraps your test in a “undo” button: whatever you do during the test gets rolled back afterward.

#### Purposes

- To isolate tests from each other so data from one test cannot affect another.
- To provide the fastest practical database reset strategy for most test suites.
- To work seamlessly with in-memory SQLite and traditional databases.
- To avoid the overhead of running migrations before every single test.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
use Illuminate\Foundation\Testing\RefreshDatabase;

uses(RefreshDatabase::class);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `RefreshDatabase` | The trait that provides migration + transaction isolation. |
| `uses()` (Pest) / `use` (PHPUnit) | Applies the trait to the test file or class. |
| `$connectionsToTransact` | Optional property to specify which connections are wrapped in transactions. |

**Syntax Rules**

- Apply the trait at the test-class level (PHPUnit) or via `uses()` in Pest.
- The trait migrates the database only if the schema is out of date; otherwise, it uses transactions.
- For multi-database applications, set `protected $connectionsToTransact = ['sqlite', 'mysql'];` to control which connections are transactional.

**Constraints and Limitations**

- If the schema is up to date, `RefreshDatabase` does **not** migrate; tests that do not use the trait can still leave behind data.
- `RefreshDatabase` cannot be used with databases that do not support transactions (e.g., ClickHouse).

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Basic `RefreshDatabase` Usage**

**Code**

```php
<?php
use App\Models\Post;
use Illuminate\Foundation\Testing\RefreshDatabase;

uses(RefreshDatabase::class);

test('posts are isolated between tests', function () {
    Post::factory()->create(['title' => 'First']);
    expect(Post::count())->toBe(1);
});

test('the database is clean for the next test', function () {
    expect(Post::count())->toBe(0);
});
```

**Expected Output**

```
PASS  Tests\Feature\PostTest
✓ posts are isolated between tests
✓ the database is clean for the next test
```

**Why This Output Occurs**

Each test runs inside a transaction that is rolled back. The post created in the first test is removed before the second test runs, so the second test sees an empty table.

---

**Example 2: Applying `RefreshDatabase` Globally in Pest**

**Code**

```php
<?php
// tests/Pest.php

pest()->extend(Tests\TestCase::class)
    ->use(Illuminate\Foundation\Testing\RefreshDatabase::class)
    ->in('Feature');
```

**Expected Output**

Every test in `tests/Feature` automatically uses `RefreshDatabase`, eliminating the need to add the trait to each file manually.

**Why This Output Occurs**

The `pest()->extend()->use()->in()` chain applies the trait to all tests in the `Feature` directory, ensuring consistent isolation.

---

#### Real-World Cases with Explanation

**Case 1: Large Feature Test Suite**

A SaaS application has 200 feature tests that touch the database. Using `RefreshDatabase` globally in `tests/Pest.php` ensures every test is isolated without repeating the trait, and the suite runs in under a minute because migrations run only once per process.

**Explanation**: `RefreshDatabase` is the pragmatic default for most Laravel applications because it balances speed and isolation.

---

### 3. `DatabaseTransactions`

#### Definitions

**Core Definition**

`DatabaseTransactions` is a trait that wraps each test in a database transaction and rolls it back, but never runs migrations—the schema must already exist.

**Technical Definition**

Unlike `RefreshDatabase`, `DatabaseTransactions` assumes the test database schema is already migrated. It begins a transaction before each test and rolls it back afterward, providing isolation without the overhead of checking or running migrations. This makes it slightly faster than `RefreshDatabase` when the schema is stable and pre-migrated.

**Beginner-Friendly Explanation**

`DatabaseTransactions` is like `RefreshDatabase` but without the migration step. If you already know your database schema is up to date, this trait simply wraps each test in a transaction and undoes it afterward.

#### Purposes

- To provide transaction-based isolation without migration overhead.
- To speed up test suites where the schema is already migrated and stable.
- To avoid the migration check that `RefreshDatabase` performs.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
use Illuminate\Foundation\Testing\DatabaseTransactions;

uses(DatabaseTransactions::class);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `DatabaseTransactions` | The trait that provides transaction-only isolation. |
| `$connectionsToTransact` | Optional property to specify transactional connections. |

**Syntax Rules**

- The test database must already be migrated before running tests.
- The trait wraps each test in a transaction and rolls it back in `tearDown()`.
- For multi-connection setups, list the connections in `$connectionsToTransact`.

**Constraints and Limitations**

- If the schema is missing or outdated, tests will fail with “table not found” errors.
- Cannot be used with databases that do not support transactions.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Transaction-Only Isolation**

**Code**

```php
<?php
use App\Models\User;
use Illuminate\Foundation\Testing\DatabaseTransactions;

uses(DatabaseTransactions::class);

test('a user is created and rolled back', function () {
    User::factory()->create(['email' => 'alice@example.com']);
    $this->assertDatabaseHas('users', ['email' => 'alice@example.com']);
});

test('the previous user is gone', function () {
    $this->assertDatabaseMissing('users', ['email' => 'alice@example.com']);
});
```

**Expected Output**

```
PASS  Tests\Feature\UserTest
✓ a user is created and rolled back
✓ the previous user is gone
```

**Why This Output Occurs**

The first test creates a user inside a transaction. The transaction is rolled back after the test, so the second test sees no user with that email.

---

#### Real-World Cases with Explanation

**Case 1: Pre-Migrated CI Environment**

A CI pipeline runs `php artisan migrate` once before the test suite. The test suite uses `DatabaseTransactions` instead of `RefreshDatabase` to shave off the migration-check overhead, resulting in a marginally faster run.

**Explanation**: `DatabaseTransactions` is a viable alternative when the schema is guaranteed to be current and the suite is large enough that the migration check becomes noticeable.

---

### 4. `DatabaseMigrations`

#### Definitions

**Core Definition**

`DatabaseMigrations` is a trait that runs the full migration set before each test and rolls it back after, completely resetting the schema for every single test.

**Technical Definition**

`DatabaseMigrations` executes `migrate:fresh` (or the equivalent migration process) before each test and `migrate:rollback` after. This guarantees a completely fresh schema for every test, at the cost of significant execution time. It is slower than both `RefreshDatabase` and `DatabaseTransactions` and is typically reserved for tests that cannot use transactions (e.g., tests that alter the schema).

**Beginner-Friendly Explanation**

`DatabaseMigrations` rebuilds the entire database from scratch before every single test. It is the most thorough but slowest option—use it only when transactions cannot give you the isolation you need.

#### Purposes

- To provide a completely fresh schema for every test.
- To support tests that modify the database schema (e.g., adding columns).
- To guarantee isolation when transactions are insufficient or unavailable.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
use Illuminate\Foundation\Testing\DatabaseMigrations;

uses(DatabaseMigrations::class);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `DatabaseMigrations` | The trait that migrates before every test. |
| Migration rollback | Automatically performed after each test. |

**Syntax Rules**

- The trait runs migrations before each test and rolls them back afterward.
- It is significantly slower than `RefreshDatabase` and should be used sparingly.
- Best suited for tests that need a pristine schema or that cannot run inside transactions.

**Constraints and Limitations**

- Each test incurs the full migration cost, making large suites very slow.
- Not suitable for in-memory SQLite in some configurations due to repeated schema creation.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Schema-Altering Test**

**Code**

```php
<?php
use Illuminate\Foundation\Testing\DatabaseMigrations;
use Illuminate\Support\Facades\Schema;

uses(DatabaseMigrations::class);

test('a temporary column can be added', function () {
    Schema::table('users', function ($table) {
        $table->string('temporary_column')->nullable();
    });

    expect(Schema::hasColumn('users', 'temporary_column'))->toBeTrue();
});
```

**Expected Output**

```
PASS  Tests\Feature\SchemaTest
✓ a temporary column can be added
```

**Why This Output Occurs**

`DatabaseMigrations` runs the full migration set before the test, allowing the test to add a temporary column. After the test, the schema is rolled back, so the column does not persist.

---

#### Real-World Cases with Explanation

**Case 1: Migration Testing**

A test verifies that a new migration correctly adds an index or column. `DatabaseMigrations` ensures the schema is fresh before the test, so the migration’s effect can be isolated and verified without interference from previous tests.

**Explanation**: Schema-altering tests require the full reset that `DatabaseMigrations` provides.

---

### 5. Factories

#### Definitions

**Core Definition**

Model factories are classes that define default attributes for Eloquent models, allowing tests to generate realistic, persistent records with a single expressive call.

**Technical Definition**

A factory extends `Illuminate\Database\Eloquent\Factories\Factory` and implements a `definition()` method that returns an array of default attribute values, typically using the `fake()` helper (Faker). Models use the `HasFactory` trait to expose a static `factory()` method. Factories support states (`->state()`), sequences, callbacks, and relationship creation (`->for()`, `->has()`).

**Beginner-Friendly Explanation**

A factory is a recipe for creating fake data. Instead of manually typing every column value, you call `User::factory()->create()` and get a fully populated user record with a random name, email, and password.

#### Purposes

- To generate realistic test data without manual attribute assignment.
- To support different model states (e.g., admin, unverified) via states.
- To create related models easily through relationship helpers.
- To keep tests expressive and maintainable.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Basic creation
$user = User::factory()->create();

// With attribute overrides
$user = User::factory()->create(['email' => 'alice@example.com']);

// With a state
$admin = User::factory()->admin()->create();

// With a relationship
$post = Post::factory()->for($user)->create();
```

**Component Breakdown**

| Method | Description |
|--------|-------------|
| `factory()` | Returns a factory instance for the model. |
| `create()` | Persists the model to the database. |
| `make()` | Instantiates the model without persisting. |
| `state()` / named state | Applies a named state definition. |
| `for()` | Associates a belongsTo relationship. |
| `has()` | Creates a hasMany relationship. |

**Syntax Rules**

- Factories live in `database/factories` and are named `{Model}Factory`.
- The model must use the `HasFactory` trait.
- `create()` persists; `make()` does not.

**Constraints and Limitations**

- Factories use Faker, which generates random data; use overrides or sequences for deterministic values.
- Relationship helpers must match the actual relationship method names.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Factory with Overrides and State**

**Code**

```php
<?php
use App\Models\User;

test('an admin user can be created', function () {
    $admin = User::factory()->admin()->create([
        'email' => 'admin@example.com',
    ]);

    expect($admin->is_admin)->toBeTrue();
    $this->assertDatabaseHas('users', [
        'email'   => 'admin@example.com',
        'is_admin' => true,
    ]);
});
```

**Expected Output**

```
PASS  Tests\Feature\AdminUserTest
✓ an admin user can be created
```

**Why This Output Occurs**

The `admin()` state sets `is_admin` to `true`. The override sets a specific email. The assertion confirms both values in the database.

---

**Example 2: Factory with Relationship**

**Code**

```php
test('a post belongs to a user', function () {
    $post = Post::factory()
        ->for(User::factory())
        ->create(['title' => 'Hello']);

    $this->assertDatabaseHas('posts', [
        'title'   => 'Hello',
        'user_id' => $post->user_id,
    ]);
});
```

**Expected Output**

```
PASS  Tests\Feature\PostRelationshipTest
✓ a post belongs to a user
```

**Why This Output Occurs**

`->for(User::factory())` creates a user and associates it with the post via the `user_id` foreign key. The assertion verifies the relationship was persisted correctly.

---

#### Real-World Cases with Explanation

**Case 1: E-commerce Order Testing**

An order factory creates an order with a random total, a belongsTo user, and a hasMany order items. Tests use `Order::factory()->for($user)->hasItems(3)->create()` to generate a realistic order with three items in a single line of code.

**Explanation**: Factories eliminate boilerplate and make relationship-heavy test data trivial to generate.

---

### 6. Seeders

#### Definitions

**Core Definition**

Seeders are classes that populate the database with known data—reference values, roles, statuses, or initial records—using the query builder or model factories.

**Technical Definition**

Seeders extend `Illuminate\Database\Seeder` and implement a `run()` method. The `DatabaseSeeder` class is the entry point and can call other seeders via `$this->call([...])`. Seeders are stored in `database/seeders` and are executed via `php artisan db:seed`. Within a feature test, the `seed()` method can invoke the `DatabaseSeeder` or a specific seeder class.

**Beginner-Friendly Explanation**

A seeder is a script that fills your database with data. It is useful for adding required reference data (like roles or categories) or for creating a known starting state for tests.

#### Purposes

- To populate reference data that the application depends on.
- To provide a consistent, known starting state for tests.
- To generate large amounts of development or test data using factories.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
<?php
namespace Database\Seeders;

use Illuminate\Database\Seeder;

class RoleSeeder extends Seeder
{
    public function run(): void
    {
        Role::create(['name' => 'admin']);
        Role::create(['name' => 'user']);
    }
}
```

**Invoking Seeders in Tests**

```php
$this->seed(); // Runs DatabaseSeeder
$this->seed(RoleSeeder::class); // Runs a specific seeder
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `run()` | The method that inserts data. |
| `DatabaseSeeder` | The entry point that calls other seeders. |
| `$this->seed()` | Invokes seeders within a test. |

**Syntax Rules**

- Seeders are placed in `database/seeders`.
- `run()` is called when the seeder is executed.
- `DatabaseSeeder` can call other seeders via `$this->call()`.

**Constraints and Limitations**

- Seeders should be idempotent where possible; use `updateOrCreate` or `firstOrCreate` to avoid duplicate records.
- Seeders that depend on factories may introduce randomness; use explicit values for deterministic tests.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Seeding Reference Data in a Test**

**Code**

```php
<?php
use Database\Seeders\RoleSeeder;
use Illuminate\Foundation\Testing\RefreshDatabase;

uses(RefreshDatabase::class);

test('roles are seeded correctly', function () {
    $this->seed(RoleSeeder::class);

    $this->assertDatabaseCount('roles', 2);
    $this->assertDatabaseHas('roles', ['name' => 'admin']);
    $this->assertDatabaseHas('roles', ['name' => 'user']);
});
```

**Expected Output**

```
PASS  Tests\Feature\RoleSeederTest
✓ roles are seeded correctly
```

**Why This Output Occurs**

The `RoleSeeder` inserts two roles. The assertions verify the count and the presence of both records.

---

#### Real-World Cases with Explanation

**Case 1: Status Reference Table**

An order status table contains fixed values (`pending`, `shipped`, `delivered`). A seeder populates these values, and tests invoke the seeder to ensure the statuses exist before testing order workflows. This decouples test data from production data and keeps tests deterministic.

**Explanation**: Seeders provide a reliable, repeatable way to establish reference data that tests depend on.

---

### 7. Database Assertions

#### Definitions

**Core Definition**

Database assertions are helper methods that verify the presence, absence, count, or state of records in the database after application code has executed.

**Technical Definition**

Laravel’s `InteractsWithDatabase` trait provides `assertDatabaseHas()`, `assertDatabaseMissing()`, `assertDatabaseCount()`, `assertDatabaseEmpty()`, `assertModelExists()`, `assertModelMissing()`, `assertSoftDeleted()`, and `assertNotSoftDeleted()`. These methods accept either a table name or a model class, and an array of column-value conditions. They execute a `SELECT` query against the test database and assert that the result matches expectations.

**Beginner-Friendly Explanation**

Assertions are how you ask the database “did this actually happen?” `assertDatabaseHas` says “this row must exist.” `assertDatabaseMissing` says “this row must not exist.” They are the final check in a database test.

#### Purposes

- To verify that records were created, updated, or deleted.
- To confirm that relationships and foreign keys were persisted correctly.
- To check record counts and emptiness.
- To verify soft-delete behaviour without querying manually.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
$this->assertDatabaseHas('users', ['email' => 'alice@example.com']);
$this->assertDatabaseMissing('users', ['email' => 'bob@example.com']);
$this->assertDatabaseCount('users', 5);
$this->assertDatabaseEmpty('users');
$this->assertModelExists($user);
$this->assertModelMissing($user);
$this->assertSoftDeleted($post);
$this->assertNotSoftDeleted($post);
```

**Component Breakdown**

| Assertion | Verifies |
|-----------|----------|
| `assertDatabaseHas` | A row matching the conditions exists. |
| `assertDatabaseMissing` | No row matches the conditions. |
| `assertDatabaseCount` | The table has exactly N rows. |
| `assertDatabaseEmpty` | The table has zero rows. |
| `assertModelExists` | The model’s record exists. |
| `assertModelMissing` | The model’s record does not exist. |
| `assertSoftDeleted` | The record is soft-deleted. |

**Syntax Rules**

- Accept a table name (string) or a model class name.
- Pass an associative array of column-value conditions.
- An optional third argument specifies the database connection.

**Constraints and Limitations**

- Assertions run `SELECT` queries; excessive assertions can slow down tests.
- `assertDatabaseHas` checks for exact column values; use model assertions for complex conditions.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Count and Missing Assertions**

**Code**

```php
test('only one user is created', function () {
    User::factory()->create();

    $this->assertDatabaseCount('users', 1);
    $this->assertDatabaseMissing('users', ['email' => 'nonexistent@example.com']);
});
```

**Expected Output**

```
PASS  Tests\Feature\UserCountTest
✓ only one user is created
```

**Why This Output Occurs**

The factory creates one user. `assertDatabaseCount` verifies the table has exactly one row, and `assertDatabaseMissing` confirms that a different email does not exist.

---

**Example 2: Soft-Delete Assertion**

**Code**

```php
test('a post can be soft-deleted', function () {
    $post = Post::factory()->create();
    $post->delete();

    $this->assertSoftDeleted($post);
    $this->assertDatabaseMissing('posts', ['id' => $post->id, 'deleted_at' => null]);
});
```

**Expected Output**

```
PASS  Tests\Feature\PostSoftDeleteTest
✓ a post can be soft-deleted
```

**Why This Output Occurs**

`assertSoftDeleted` confirms the `deleted_at` column is set. The `assertDatabaseMissing` with `deleted_at` as `null` confirms the record is no longer considered “active.”

---

#### Real-World Cases with Explanation

**Case 1: Authorization and Data Integrity**

An authorization test verifies that a non-owner cannot delete a post. The test attempts the deletion and then asserts `assertDatabaseHas('posts', ['id' => $post->id])` to confirm the record still exists—proving that the authorization check prevented the deletion.

**Explanation**: Database assertions are the final proof that authorization and validation rules actually protected the data.

---

### 8. Database Testing Best Practices

#### Definitions

**Core Definition**

Database testing best practices are the guidelines that keep database tests isolated, fast, deterministic, and genuinely useful for catching persistence bugs.

**Technical Definition**

Best practices encompass: using `RefreshDatabase` as the default isolation trait; applying it globally in `tests/Pest.php` for consistency; using factories instead of manual inserts; keeping seeders simple and deterministic; preferring model assertions over raw table queries; testing both success and failure paths; and using in-memory SQLite for speed while acknowledging its limitations.

**Beginner-Friendly Explanation**

Best practices are the habits that make database tests reliable: always reset the database, always use factories, and always check that your assertions match what actually happened.

#### Key Best Practices

1. **Use `RefreshDatabase` by Default**: It is the recommended trait for most tests—fast, reliable, and compatible with in-memory SQLite.

2. **Apply `RefreshDatabase` Globally in Pest**: In `tests/Pest.php`, chain `->use(RefreshDatabase::class)->in('Feature')` so every feature test is isolated without repeating the trait.

3. **Never Point Tests at Production**: Always use a dedicated test database or in-memory SQLite.

4. **Use Factories, Not Manual Inserts**: Factories are expressive, maintainable, and support states and relationships.

5. **Keep Seeders Simple and Deterministic**: Seed reference data with explicit values; avoid random data in seeders used for tests.

6. **Prefer Model Assertions When Possible**: `assertModelExists($user)` is more expressive than `assertDatabaseHas('users', ['id' => $user->id])`.

7. **Test Both Success and Failure Paths**: For every database mutation, test that the happy path persists and that the failure path (validation, authorization) does not.

8. **Use `DatabaseTruncation` When Transactions Are Unsuitable**: For browser tests or schema-altering tests, `DatabaseTruncation` is faster than `DatabaseMigrations`.

9. **Beware of Missing Traits**: A test that forgets `RefreshDatabase` can leak data and cause flaky failures in other tests. Use grep or static analysis to catch offenders.

10. **Choose the Right Engine**: In-memory SQLite is fast but may not support all features. Use a dedicated MySQL/PostgreSQL test database if your tests depend on database-specific behaviour.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Global `RefreshDatabase` + Factory + Assertion**

**Code**

```php
<?php
// tests/Pest.php
pest()->extend(Tests\TestCase::class)
    ->use(Illuminate\Foundation\Testing\RefreshDatabase::class)
    ->in('Feature');

// tests/Feature/PostTest.php
test('a post is persisted with the correct author', function () {
    $user = User::factory()->create();
    $post = Post::factory()->for($user)->create(['title' => 'Best Practices']);

    $this->assertDatabaseHas('posts', [
        'title'   => 'Best Practices',
        'user_id' => $user->id,
    ]);
});
```

**Expected Output**

```
PASS  Tests\Feature\PostTest
✓ a post is persisted with the correct author
```

**Why This Output Occurs**

The global `RefreshDatabase` ensures isolation. The factory creates a user and a post with the correct foreign key. The assertion verifies the persisted relationship.

---

**Example 2: Success and Failure Paths**

**Code**

```php
test('authorized user can delete a post', function () {
    $user = User::factory()->create();
    $post = Post::factory()->for($user)->create();

    $this->actingAs($user)->delete("/posts/{$post->id}");

    $this->assertDatabaseMissing('posts', ['id' => $post->id]);
});

test('unauthorized user cannot delete a post', function () {
    $user  = User::factory()->create();
    $other = User::factory()->create();
    $post  = Post::factory()->for($other)->create();

    $this->actingAs($user)->delete("/posts/{$post->id}");

    $this->assertDatabaseHas('posts', ['id' => $post->id]);
});
```

**Expected Output**

```
PASS  Tests\Feature\PostDeletionTest
✓ authorized user can delete a post
✓ unauthorized user cannot delete a post
```

**Why This Output Occurs**

The first test confirms the owner can delete the post (record is missing). The second test confirms a non-owner cannot delete the post (record still exists). Together, they verify the authorization rule at the database level.

---

#### Real-World Cases with Explanation

**Case 1: CI Pipeline with Database Testing**

A team runs `php artisan test` in GitHub Actions with SQLite in-memory and `RefreshDatabase` applied globally. Every feature test is isolated, factories generate data, and assertions verify persistence. The suite runs in under 30 seconds and has caught multiple regressions where a controller “succeeded” but failed to save the record.

**Explanation**: Following database testing best practices results in a fast, reliable suite that provides genuine confidence in the data layer.

---

## References

- Database Testing — Laravel 13.x Documentation — https://laravel.com/framework/docs/13.x/database-testing
- Eloquent: Factories — Laravel 13.x Documentation — https://laravel.com/framework/docs/eloquent-factories
- Database: Seeding — Laravel 13.x Documentation — https://laravel.com/framework/docs/13.x/seeding
- InteractsWithDatabase (API Reference) — https://api.laravel.com/docs/9.x/Illuminate/Foundation/Testing/Concerns/InteractsWithDatabase.html
- Database Testing (testing traits table) — https://github.com/laravel-clickhouse/laravel-clickhouse/blob/bda27f8633f047e2c9b7c92da6c198c3d04a7fde/docs/docs/testing.md
- Find Feature Tests Creating Database Records without Refreshing the Database — Laravel News — https://laravel-news.com/find-feature-tests-creating-database-records-without-refreshing-the-database-in-laravel
- Laravel TDD Skill (Database Strategy) — https://github.com/affaan-m/ECC/blob/8bdf88e5ad8877bcd00a4aba7ccbfb50f235f10f/skills/laravel-tdd/SKILL.md
- Laravel Testing Database Basics — https://github.com/Fusengine/claude-plugins/blob/main/plugins/laravel-expert/skills/laravel-testing/references/database-basics.md
- DatabaseTruncation (API Reference) — https://api.laravel.com/docs/9.x/Illuminate/Foundation/Testing/DatabaseTruncation.html
- DatabaseTruncation vs DatabaseMigrations (Stack Overflow revision) — https://stackoverflow.com/revisions/9d3b8d06-3579-4581-aef8-6175f4802c94/view-source

---

**Version Note**: This cheat sheet is based on Laravel 10.x through 13.x documentation. `RefreshDatabase` is the recommended default trait for database isolation. `DatabaseTruncation` was introduced in Laravel 8.51/9 and is faster than `DatabaseMigrations` for tests that cannot use transactions. Pest’s `pest()->use()->in()` global trait application is the recommended way to enforce `RefreshDatabase` across all feature tests. Developers should always consult the documentation for their specific Laravel version.