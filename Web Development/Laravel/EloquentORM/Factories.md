# Laravel Factories: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

A Laravel factory is a dedicated PHP class that defines a default set of attribute values for an Eloquent model, enabling the rapid generation of realistic, fake model instances for testing and database seeding.

**Technical Definition**

Factories extend `Illuminate\Database\Eloquent\Factories\Factory` and define a `definition()` method returning an array of default attribute values. They are stored in `database/factories` and are resolved via the `HasFactory` trait on the model. Factories leverage the Faker PHP library (accessed via the `fake()` helper) to generate random data, and support advanced features including states, callbacks, sequences, relationships, and recycling. The `create()` method persists models to the database, while `make()` instantiates them in memory.

**Beginner-Friendly Explanation**

A factory is a recipe for creating fake data for your application. Instead of manually typing out every attribute for every test record, you write a factory once and tell it "make me 50 users." Laravel fills in realistic names, emails, and other details automatically. It is like a cookie cutter for your database records—you define the shape once, and you can stamp out as many as you need.

---

### Key Characteristics

- **Definition-Based Data Generation**: Factories define a default set of attributes in a `definition()` method, using Faker to generate random, realistic values.
- **Class-Based Architecture**: Since Laravel 8, factories are class-based (replacing the older global `factory()` helper), with first-class relationship support and autoloading.
- **Automatic Model Discovery**: The `HasFactory` trait on models automatically resolves the corresponding factory class using naming conventions.
- **Persistence Control**: `create()` persists models to the database; `make()` creates in-memory instances without touching the database.
- **Factory States**: Named state methods (e.g., `admin()`, `unverified()`) encapsulate discrete attribute variations.
- **Factory Callbacks**: `afterMaking()` and `afterCreating()` hooks allow additional tasks to run after model instantiation or persistence.
- **Sequences**: The `sequence()` method generates alternating attribute values across multiple models.
- **Relationship Support**: Fluent methods (`has()`, `for()`, `hasAttached()`, `recycle()`) create and associate related models.
- **Relationship Expansion Control**: The `dontExpandRelationshipsByDefault()` method prevents factories from automatically creating parent relationships when only the primary model is needed.

---

### Prerequisites

- **PHP 8.1 or higher** (Laravel 10+) or **PHP 8.2** (Laravel 11+).
- **A Laravel application** installed via Composer.
- **Eloquent models** with the `HasFactory` trait (included by default).
- **A configured database** for persisting factory-created models.
- **Basic understanding of Eloquent models and relationships**.

---

### Related Programming Areas

- **Database Seeding**: Factories are commonly used within seeders to populate databases with test or demo data.
- **Eloquent ORM**: Factories generate Eloquent model instances, leveraging the full power of the ORM.
- **Testing**: Factories are the primary tool for generating test data in feature and unit tests.
- **Faker PHP Library**: Factories use Faker (via the `fake()` helper) to generate random, realistic attribute values.
- **Database Migrations**: Factories depend on the schema defined by migrations.

---

## Core Concepts

---

### 1. Factories

#### Definitions

**Core Definition**

A factory is a class that defines a default set of attributes for an Eloquent model, used to generate fake model instances for testing and seeding.

**Technical Definition**

Factories extend `Illuminate\Database\Eloquent\Factories\Factory` and define a `definition()` method returning an array of default attribute values. The class is stored in `database/factories` and is resolved via the `HasFactory` trait on the model. The `factory()` static method on the model returns a factory instance, which is then chained with `create()` or `make()` to generate models.

**Beginner-Friendly Explanation**

A factory is a blueprint for creating model records. You write it once, and then you can use it to create as many records as you need, with realistic fake data filled in automatically.

#### Purposes

- To define a default set of attributes for an Eloquent model.
- To generate large volumes of realistic test data quickly.
- To reduce boilerplate when creating model records in tests and seeders.
- To provide a consistent, reusable source of fake data across the application.
- To support complex data generation with states, sequences, and relationships.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
<?php
// database/factories/UserFactory.php
namespace Database\Factories;

use Illuminate\Database\Eloquent\Factories\Factory;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Str;

/**
 * @extends \Illuminate\Database\Eloquent\Factories\Factory<\App\Models\User>
 */
class UserFactory extends Factory
{
    protected static ?string $password;

    public function definition(): array
    {
        return [
            'name' => fake()->name(),
            'email' => fake()->unique()->safeEmail(),
            'email_verified_at' => now(),
            'password' => static::$password ??= Hash::make('password'),
            'remember_token' => Str::random(10),
        ];
    }
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `extends Factory` | Required base class. |
| `definition(): array` | Returns the default attribute values. |
| `fake()` | The Faker instance for generating random data. |
| `database/factories/` | Default directory for factory classes. |
| `Database\Factories` | Default namespace for factories. |

**Syntax Rules**

- Factories must extend `Illuminate\Database\Eloquent\Factories\Factory`.
- The `definition()` method is required and must return an array.
- The `fake()` helper provides access to the Faker PHP library.
- Factories are stored in `database/factories` and use the `Database\Factories` namespace.
- The model must use the `HasFactory` trait for automatic factory resolution.

**Constraints and Limitations**

- Factories generate fake data and should not be used in production for real data.
- The `definition()` method should not contain complex logic; use states or callbacks for variations.
- Factory-generated data is random and may not be suitable for tests that require deterministic values.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Basic User Factory**

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
    protected static ?string $password;

    public function definition(): array
    {
        return [
            'name' => fake()->name(),
            'email' => fake()->unique()->safeEmail(),
            'email_verified_at' => now(),
            'password' => static::$password ??= Hash::make('password'),
            'remember_token' => Str::random(10),
        ];
    }
}
```

**Model**

```php
<?php
// app/Models/User.php
namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable
{
    use HasFactory;
}
```

**Usage**

```php
// Create a single user (persisted to database)
$user = User::factory()->create();

// Create 50 users
$users = User::factory()->count(50)->create();

// Create a user without persisting
$user = User::factory()->make();
```

**Expected Output**

```php
// User::factory()->create() → User model persisted with id, random name, unique email, hashed password
// User::factory()->count(50)->create() → Collection of 50 User models
// User::factory()->make() → User model instance (not persisted, no id)
```

**Why This Output Occurs**

The `definition()` method returns the default attributes. The `create()` method persists the model to the database, generating an auto-incrementing ID and storing the hashed password. The `make()` method instantiates the model without persisting it.

---

**Example 2: Using `dontExpandRelationshipsByDefault()`**

```php
use App\Models\OrderItem;
use Database\Factories\OrderItemFactory;

public function test_price_calculation(): void
{
    OrderItemFactory::dontExpandRelationshipsByDefault();

    $item = OrderItem::factory()->make([
        'quantity' => 2,
        'unit_price' => 15.50,
        'order_id' => 999,
    ]);

    $this->assertEquals(31.00, $item->calculateTotal());
}
```

**Expected Output**

```
// The OrderItem is created without creating Order, Product, or Customer records
// Only the OrderItem instance is generated, speeding up the test
```

**Why This Output Occurs**

By default, factories automatically create related models when relationships are defined in the factory. The `dontExpandRelationshipsByDefault()` method disables this behaviour, preventing unnecessary database writes when only the primary model is needed.

---

#### Real-World Cases with Explanation

**Case 1: Testing User Registration**

A developer writes a feature test for user registration. The test uses `User::factory()->make()` to create an in-memory user instance with the same attributes that the registration form would submit, then posts to the registration endpoint and asserts the user was created.

**Explanation**: Factories provide a quick, consistent way to generate test data that mimics real user input.

**Case 2: Seeding Demo Data**

A SaaS application seeds demo data for new tenants. The seeder uses `User::factory()->count(10)->hasPosts(5)->create()` to create 10 users, each with 5 posts, providing a rich demo experience.

**Explanation**: Factories enable the rapid generation of complex, interconnected data for development and demonstration purposes.

---

### 2. Factory Creation

#### Definitions

**Core Definition**

Factory creation is the process of generating a new factory class using Laravel's Artisan CLI.

**Technical Definition**

The `php artisan make:factory` command generates a PHP class in `database/factories` extending `Illuminate\Database\Eloquent\Factories\Factory`. The `--model` option pre-fills the generated factory with the specified model class and imports it.

**Beginner-Friendly Explanation**

Creating a factory is a one-command process. Laravel makes the file for you, and you fill in the default attributes.

#### Purposes

- To automate the generation of factory class boilerplate.
- To ensure factories are placed in the correct namespace and directory.
- To pre-fill the factory with the specified model class using the `--model` option.
- To reduce manual file creation errors.

#### Syntax Rules and Structure

**Complete General Syntax**

```bash
php artisan make:factory PostFactory
php artisan make:factory PostFactory --model=Post
```

**Component Breakdown**

| Command | Effect |
|---------|--------|
| `make:factory PostFactory` | Creates `database/factories/PostFactory.php`. |
| `--model=Post` | Pre-fills the factory with the `Post` model. |

**Syntax Rules**

- The factory name should be PascalCase and end with `Factory`.
- The `--model` option is optional but recommended.
- Generated files must be edited to add `definition()` logic.

**Constraints and Limitations**

- The command does not register the factory anywhere; it is resolved automatically via conventions.
- The generated factory contains an empty `definition()` method.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Generating a Factory with Model**

**Command**

```bash
php artisan make:factory PostFactory --model=Post
```

**Generated File**

```php
<?php
// database/factories/PostFactory.php
namespace Database\Factories;

use App\Models\Post;
use Illuminate\Database\Eloquent\Factories\Factory;

/**
 * @extends \Illuminate\Database\Eloquent\Factories\Factory<\App\Models\Post>
 */
class PostFactory extends Factory
{
    protected $model = Post::class;

    public function definition(): array
    {
        return [
            //
        ];
    }
}
```

**Expected Output (Terminal)**

```
INFO  Factory [database/factories/PostFactory.php] created successfully.
```

**Why This Output Occurs**

Artisan scaffolds the factory class, imports the `Post` model, and sets the `$model` property. The `definition()` method is empty and must be filled in.

---

#### Real-World Cases with Explanation

**Case 1: Rapid Test Setup**

A developer runs `php artisan make:factory ProductFactory --model=Product` and immediately fills in the `definition()` method with attributes for name, price, and description.

**Explanation**: Artisan accelerates development by generating the necessary boilerplate.

---

### 3. `artisan make:factory`

#### Definitions

**Core Definition**

`artisan make:factory` is the Artisan command used to generate new factory classes.

**Technical Definition**

The command is implemented by `Illuminate\Database\Console\Factories\FactoryMakeCommand`. It generates a PHP class file in `database/factories`. The `--model` option specifies the model class and pre-fills the `$model` property and the docblock.

**Beginner-Friendly Explanation**

`artisan make:factory` is the terminal command you type to create a new factory file.

#### Purposes

- To generate factory classes with minimal effort.
- To provide consistent structure for factory files.
- To pre-fill the factory with the model class using the `--model` option.
- To enforce Laravel's naming and directory conventions.

#### Syntax Rules and Structure

**Complete General Syntax**

```bash
php artisan make:factory <Name> [--model=<Model>]
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `make:factory` | The Artisan command. |
| `<Name>` | The factory class name (PascalCase, ends with `Factory`). |
| `--model` | Optional model class to associate. |

**Syntax Rules**

- The factory name is required.
- The `--model` option accepts a fully qualified class name or a short name.
- Generated files are placed in `database/factories`.

**Constraints and Limitations**

- The command does not populate the `definition()` method.
- The model must exist for the `--model` option to work correctly.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Generating a Factory with Fully Qualified Model**

**Command**

```bash
php artisan make:factory PostFactory --model="App\Models\Post"
```

**Expected Output**

```
INFO  Factory [database/factories/PostFactory.php] created successfully.
```

**Generated File**

```php
<?php
// database/factories/PostFactory.php
namespace Database\Factories;

use App\Models\Post;
use Illuminate\Database\Eloquent\Factories\Factory;

class PostFactory extends Factory
{
    protected $model = Post::class;

    public function definition(): array
    {
        return [
            //
        ];
    }
}
```

**Why This Output Occurs**

The `--model` option specifies the model class, which is imported and assigned to the `$model` property.

---

#### Real-World Cases with Explanation

**Case 1: Modular Factory Generation**

A large application uses separate factories for each model. The developer runs `make:factory` for each model, ensuring consistent naming and structure.

**Explanation**: The `make:factory` command enforces conventions and saves time.

---

### 4. Factory Definition

#### Definitions

**Core Definition**

Factory definition is the `definition()` method on a factory class that returns the default set of attribute values for a model.

**Technical Definition**

`definition(): array` is the only required method on a factory class. It returns an associative array where keys are attribute names and values are the default values. Values can be static, Faker-generated (`fake()->name()`), or closures that receive the evaluated attributes array.

**Beginner-Friendly Explanation**

The `definition()` method is where you list the default values for each attribute of your model. For example, you might say "the name should be a random name, the email should be a random email, and the password should be a hashed password."

#### Purposes

- To define the default attribute values for a model.
- To generate realistic fake data using Faker.
- To provide a single source of truth for model attributes in tests.
- To support closures for attributes that depend on other attributes.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
public function definition(): array
{
    return [
        'attribute1' => value1,
        'attribute2' => fake()->method(),
        'attribute3' => function (array $attributes) {
            return derivedValue($attributes['attribute1']);
        },
    ];
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `definition()` | Required method returning an array. |
| `fake()` | Faker instance for random data. |
| `function (array $attributes)` | Closure receiving evaluated attributes. |

**Syntax Rules**

- `definition()` must return an array.
- Attribute keys correspond to database columns.
- Closures are evaluated after other attributes, receiving the attributes array.
- The `fake()` helper provides access to Faker methods.

**Constraints and Limitations**

- `definition()` should not contain complex business logic.
- Attributes that depend on other attributes should use closures.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Basic Definition**

```php
public function definition(): array
{
    return [
        'name' => fake()->name(),
        'email' => fake()->unique()->safeEmail(),
        'email_verified_at' => now(),
        'password' => Hash::make('password'),
        'remember_token' => Str::random(10),
    ];
}
```

**Expected Output**

```
// Each user is created with a random name, unique email, current timestamp, hashed password, and random token
```

**Why This Output Occurs**

The `definition()` method returns the default attributes. Faker generates random names and emails, while `Hash::make()` creates a hashed password.

---

**Example 2: Closure-Based Attribute**

```php
public function definition(): array
{
    return [
        'user_id' => User::factory(),
        'user_type' => function (array $attributes) {
            return User::find($attributes['user_id'])->type;
        },
        'title' => fake()->title(),
    ];
}
```

**Expected Output**

```
// The user_type attribute is derived from the user_id attribute after it is evaluated
```

**Why This Output Occurs**

Closures receive the evaluated attributes array, allowing derived attributes to depend on other attributes.

---

#### Real-World Cases with Explanation

**Case 1: Product Factory with Derived SKU**

A `ProductFactory` generates a `sku` attribute by combining the category abbreviation and a random number. The `definition()` method uses a closure to access the `category` attribute.

**Explanation**: Closures in `definition()` enable derived attributes.

---

### 5. Factory States

#### Definitions

**Core Definition**

A factory state is a named method on a factory class that applies a discrete set of attribute modifications to the default definition.

**Technical Definition**

State methods use `$this->state(fn (array $attributes) => [...])` to return a modified factory instance. States are named as adjectives or past participles (e.g., `admin()`, `unverified()`, `suspended()`) and describe what the model IS in that state. Multiple states can be chained.

**Beginner-Friendly Explanation**

A state lets you say "make this model an admin" or "make this model unverified." Instead of specifying all the changes manually every time, you define a named state method once and reuse it.

#### Purposes

- To encapsulate discrete variations of a model.
- To provide a clean, readable API for creating models in specific states.
- To reduce repetition when the same attribute modifications are needed repeatedly.
- To name states descriptively, improving test readability.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
public function admin(): static
{
    return $this->state(fn (array $attributes) => [
        'role' => 'admin',
        'is_active' => true,
    ]);
}

// Usage
$admin = User::factory()->admin()->create();
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `state(Closure)` | Returns a modified factory instance. |
| `fn (array $attributes)` | Receives the current attributes. |
| State method name | Descriptive adjective or past participle. |

**Syntax Rules**

- State methods should return `static`.
- State names should describe what the model IS (e.g., `admin()`, `unverified()`).
- States can be chained: `User::factory()->admin()->unverified()->create()`.
- States can be combined with `count()`.

**Constraints and Limitations**

- State methods should not contain complex logic; they modify attributes.
- State names should not be verbs (e.g., `makeAdmin()` is discouraged).

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Unverified State**

```php
/**
 * Indicate that the model's email address should be unverified.
 */
public function unverified(): static
{
    return $this->state(fn (array $attributes) => [
        'email_verified_at' => null,
    ]);
}
```

**Usage**

```php
$unverifiedUser = User::factory()->unverified()->create();
```

**Expected Output**

```
// User is created with email_verified_at set to null
```

**Why This Output Occurs**

The `unverified()` state modifies the `email_verified_at` attribute to `null`.

---

**Example 2: Chaining States**

```php
$user = User::factory()->admin()->unverified()->create();
```

**Expected Output**

```
// User is created with admin role AND unverified email
```

**Why This Output Occurs**

States are chainable and each modifies the attribute array.

---

#### Real-World Cases with Explanation

**Case 1: Testing Admin vs. Regular User**

A test suite needs to verify that admins can access the admin dashboard while regular users cannot. States `admin()` and `regular()` are defined, and tests use `User::factory()->admin()->create()` and `User::factory()->create()`.

**Explanation**: States make tests more readable and intention-revealing.

---

### 6. Factory Callbacks

#### Definitions

**Core Definition**

Factory callbacks are hooks (`afterMaking` and `afterCreating`) that allow additional tasks to be executed after a model is instantiated or persisted by a factory.

**Technical Definition**

Callbacks are registered in the `configure()` method using `$this->afterMaking(Closure)` and `$this->afterCreating(Closure)`. `afterMaking` runs after `make()` instantiates a model; `afterCreating` runs after `create()` persists a model. Callbacks receive the model instance and can perform side effects such as creating related records, logging, or attaching files.

**Beginner-Friendly Explanation**

Callbacks let you do something extra after a model is created. For example, after creating a user, you might want to automatically create a profile for them.

#### Purposes

- To perform additional tasks after a model is made or created.
- To create related records automatically (e.g., a profile after a user).
- To set up file attachments or external integrations.
- To log or audit factory-generated models.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
public function configure(): static
{
    return $this->afterMaking(function (User $user) {
        // Runs after make()
    })->afterCreating(function (User $user) {
        // Runs after create()
    });
}
```

**Component Breakdown**

| Method | Description |
|--------|-------------|
| `configure()` | Where callbacks are registered. |
| `afterMaking(Closure)` | Runs after `make()`. |
| `afterCreating(Closure)` | Runs after `create()`. |
| `$this` | The factory instance. |

**Syntax Rules**

- Callbacks are registered in `configure()`.
- Callbacks receive the model instance.
- `afterMaking` runs for both `make()` and `create()` (before persistence).
- `afterCreating` runs only for `create()` (after persistence).

**Constraints and Limitations**

- Callbacks cannot modify the model's attributes before creation (use states for that).
- Callbacks should be idempotent if factories are run multiple times.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: After Creating Callback**

```php
public function configure(): static
{
    return $this->afterCreating(function (User $user) {
        $user->profile()->create([
            'bio' => fake()->paragraph(),
        ]);
    });
}
```

**Usage**

```php
$user = User::factory()->create();
```

**Expected Output**

```
// A User is created, and a Profile is automatically created and associated with the User
```

**Why This Output Occurs**

The `afterCreating` callback runs after the user is persisted, creating a related profile record.

---

**Example 2: After Making Callback**

```php
public function configure(): static
{
    return $this->afterMaking(function (User $user) {
        $user->setAttribute('temporary_flag', true);
    });
}
```

**Expected Output**

```
// The temporary_flag attribute is set after the model is instantiated but before persistence
```

**Why This Output Occurs**

`afterMaking` runs after `make()` or before `create()` persists the model.

---

#### Real-World Cases with Explanation

**Case 1: User with Profile**

A `UserFactory` uses `afterCreating` to automatically create a `Profile` for each user, ensuring every user has an associated profile.

**Explanation**: Callbacks automate related record creation.

---

### 7. Factory Relationships

#### Definitions

**Core Definition**

Factory relationships are fluent methods (`has()`, `for()`, `hasAttached()`, `recycle()`) that create and associate related models when generating a model.

**Technical Definition**

The `has()` method creates child models for a `hasMany` or `hasOne` relationship. The `for()` method defines the parent model for a `belongsTo` relationship. The `hasAttached()` method creates and attaches models for a `belongsToMany` relationship, with optional pivot attributes. The `recycle()` method reuses existing model instances across multiple factory calls.

**Beginner-Friendly Explanation**

Factory relationships let you create a user and their posts in one line: `User::factory()->hasPosts(3)->create()`. You do not have to manually create the posts and set the foreign key.

#### Purposes

- To create related models automatically when generating a model.
- To reduce boilerplate when setting up test data with relationships.
- To support all relationship types (hasMany, belongsTo, belongsToMany, polymorphic).
- To optimise performance by recycling shared parent models.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Has many
$user = User::factory()->has(Post::factory()->count(3))->create();
$user = User::factory()->hasPosts(3)->create();

// Belongs to
$posts = Post::factory()->count(3)->for(User::factory())->create();
$posts = Post::factory()->count(3)->for($user)->create();

// Many to many
$user = User::factory()->hasAttached(Role::factory()->count(3), ['active' => true])->create();

// Recycle
$tickets = Ticket::factory()->recycle($airline)->create();
```

**Component Breakdown**

| Method | Description |
|--------|-------------|
| `has(Factory)` | Creates related models for hasMany/hasOne. |
| `for(Factory\|Model)` | Defines the parent for belongsTo. |
| `hasAttached(Factory, $pivot)` | Creates and attaches for belongsToMany. |
| `recycle(Model)` | Reuses an existing model instance. |

**Syntax Rules**

- `has()` accepts a factory instance or a magic method (`hasPosts(3)`).
- `for()` accepts a factory instance or an existing model.
- `hasAttached()` accepts pivot attributes as the second argument.
- `recycle()` accepts a model instance or collection.

**Constraints and Limitations**

- Relationship expansion can create unnecessary records; use `dontExpandRelationshipsByDefault()` when only the primary model is needed.
- Magic methods (`hasPosts`) rely on naming conventions.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Has Many Relationship**

```php
$user = User::factory()
    ->has(Post::factory()->count(3))
    ->create();

// Magic method equivalent
$user = User::factory()->hasPosts(3)->create();
```

**Expected Output**

```
// A User is created with 3 associated Posts
// Each Post's user_id is set to the User's ID
```

**Why This Output Occurs**

The `has()` method creates child models and sets the foreign key automatically.

---

**Example 2: Belongs To Relationship**

```php
$posts = Post::factory()
    ->count(3)
    ->for(User::factory()->state(['name' => 'Jessica Archer']))
    ->create();
```

**Expected Output**

```
// 3 Posts are created, all belonging to a single User named "Jessica Archer"
```

**Why This Output Occurs**

The `for()` method creates a parent model and sets the foreign key on the child.

---

**Example 3: Many to Many with Pivot Attributes**

```php
$user = User::factory()
    ->hasAttached(
        Role::factory()->count(3),
        ['active' => true]
    )
    ->create();
```

**Expected Output**

```
// A User is created with 3 Roles, each with pivot attribute 'active' => true
```

**Why This Output Occurs**

The `hasAttached()` method creates related models and inserts pivot records with the specified attributes.

---

**Example 4: Recycling an Existing Model**

```php
$airline = Airline::factory()->create();

$tickets = Ticket::factory()
    ->recycle($airline)
    ->create();
```

**Expected Output**

```
// Tickets are created, all sharing the same Airline instance
```

**Why This Output Occurs**

The `recycle()` method reuses the provided airline for all tickets, avoiding redundant airline creation.

---

#### Real-World Cases with Explanation

**Case 1: E-commerce Test Data**

A test creates an order with 5 items, each belonging to a product, and all belonging to the same customer. The factory uses `for($customer)`, `has(OrderItem::factory()->count(5))`, and `recycle($customer)` to build the structure efficiently.

**Explanation**: Factory relationships enable complex data structures to be created in a few lines.

---

### 8. Factory Sequences

#### Definitions

**Core Definition**

A factory sequence is a mechanism for generating alternating attribute values across multiple models created in a single factory call.

**Technical Definition**

The `sequence()` method accepts an array of attribute arrays or a closure that receives a `Sequence` instance with an `index` property. Each model created in the batch receives attributes from the sequence in order.

**Beginner-Friendly Explanation**

A sequence lets you say "make the first user an admin, the second an editor, and the third a viewer." It alternates values across multiple records.

#### Purposes

- To generate models with alternating attribute values.
- To create predictable patterns in test data.
- To avoid duplicate values in unique columns.
- To assign different roles or statuses to a batch of models.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
$users = User::factory()
    ->count(3)
    ->sequence(
        ['role' => 'admin'],
        ['role' => 'editor'],
        ['role' => 'viewer'],
    )
    ->create();

// With closure
$users = User::factory()
    ->count(10)
    ->sequence(fn ($sequence) => ['order' => $sequence->index + 1])
    ->create();
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `sequence(array ...$sequences)` | Accepts alternating attribute arrays. |
| `Sequence $sequence` | Provides the `index` property. |
| `$sequence->index` | Zero-based index of the current model. |

**Syntax Rules**

- Sequences are consumed in order; if fewer sequences than models, the sequence repeats.
- The `Sequence` object provides an `index` property.
- Sequences can be combined with states.

**Constraints and Limitations**

- Sequences do not reset between factory calls by default.
- Sequences are not suitable for random distributions; use Faker for that.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Alternating Roles**

```php
$users = User::factory()
    ->count(3)
    ->sequence(
        ['role' => 'admin'],
        ['role' => 'editor'],
        ['role' => 'viewer'],
    )
    ->create();
```

**Expected Output**

```
// User 1: role = admin
// User 2: role = editor
// User 3: role = viewer
```

**Why This Output Occurs**

The sequence provides attributes in order for each model created.

---

**Example 2: Using Sequence Index**

```php
$posts = Post::factory()
    ->count(10)
    ->sequence(fn ($sequence) => ['order' => $sequence->index + 1])
    ->create();
```

**Expected Output**

```
// Post 1: order = 1
// Post 2: order = 2
// ...
// Post 10: order = 10
```

**Why This Output Occurs**

The closure receives the `Sequence` object and uses its `index` property to generate sequential values.

---

#### Real-World Cases with Explanation

**Case 1: Seeding a Leaderboard**

A game seeder creates 100 players with sequential ranks. The sequence assigns rank 1 to the first player, rank 2 to the second, and so on.

**Explanation**: Sequences generate ordered data patterns.

---

### 9. Factory Best Practices

#### Definitions

**Core Definition**

Factory best practices are the recommended guidelines for designing, implementing, and maintaining factories to maximise maintainability, test performance, and data consistency.

**Technical Definition**

Best practices encompass keeping `definition()` minimal, using states for variations, leveraging relationships fluently, using sequences for alternating values, recycling shared models, controlling relationship expansion, and testing factories.

**Beginner-Friendly Explanation**

Best practices are the tips experienced developers follow to keep their factories clean, fast, and easy to work with.

#### Key Best Practices

1. **Keep `definition()` Minimal**: Define only the essential default attributes. Use states for variations and overrides for specific test needs.

2. **Use States for Variations**: Encapsulate discrete variations (e.g., `admin()`, `unverified()`, `suspended()`) in state methods rather than passing attributes manually.

3. **Use Relationships Fluently**: Leverage `has()`, `for()`, `hasAttached()`, and magic methods to create related models in a single chain.

4. **Recycle Shared Models**: Use `recycle()` to reuse existing model instances across multiple factory calls, reducing database writes and improving performance.

5. **Control Relationship Expansion**: Use `dontExpandRelationshipsByDefault()` when only the primary model is needed, avoiding unnecessary related record creation.

6. **Use Sequences for Alternating Values**: Use `sequence()` to generate predictable patterns (e.g., alternating roles, sequential ranks).

7. **Avoid Complex Logic in Factories**: Factories should generate data, not perform business logic. Move complex setup to seeders or test helpers.

8. **Test Factories**: Write tests that verify factories generate valid models and that states produce the expected attributes.

9. **Use `make()` for Logic-Only Tests**: When testing model logic that does not require persistence, use `make()` to avoid database writes.

10. **Use `UseFactory` Attribute for Custom Resolution**: When factory naming conventions do not apply, use the `#[UseFactory]` attribute on the model to specify the factory explicitly.

11. **Keep Faker Locale Consistent**: Set the `faker_locale` option in `config/app.php` to ensure consistent locale-specific data generation.

12. **Document Factory States**: Add PHPDoc comments to state methods describing what the state represents.

#### Multiple Annotated Complete Step by Step Code Example and Expected Output

**Example 1: Well-Structured Factory with Best Practices**

**Factory**

```php
<?php
declare(strict_types=1);

namespace Database\Factories;

use App\Models\User;
use Illuminate\Database\Eloquent\Factories\Factory;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Str;

/**
 * @extends Factory<User>
 */
final class UserFactory extends Factory
{
    protected $model = User::class;

    protected static ?string $password;

    public function definition(): array
    {
        return [
            'name' => fake()->name(),
            'email' => fake()->unique()->safeEmail(),
            'email_verified_at' => now(),
            'password' => static::$password ??= Hash::make('password'),
            'remember_token' => Str::random(10),
        ];
    }

    public function unverified(): static
    {
        return $this->state(fn (array $attributes) => [
            'email_verified_at' => null,
        ]);
    }

    public function admin(): static
    {
        return $this->state(fn (array $attributes) => [
            'role' => 'admin',
        ]);
    }

    public function configure(): static
    {
        return $this->afterCreating(function (User $user) {
            $user->profile()->create([
                'bio' => fake()->paragraph(),
            ]);
        });
    }
}
```

**Usage**

```php
// Create a verified admin user with a profile
$admin = User::factory()->admin()->create();

// Create an unverified regular user without a profile
$user = User::factory()->unverified()->create();

// Create 5 users with 3 posts each, reusing a single category
$category = Category::factory()->create();
$users = User::factory()
    ->count(5)
    ->hasPosts(3)
    ->recycle($category)
    ->create();
```

**Expected Output**

```
// Admin user: role = admin, email verified, profile created
// Unverified user: email_verified_at = null
// 5 users with 3 posts each, all posts belonging to the same category
```

**Why This Output Occurs**

The factory follows all best practices: minimal `definition()`, states for variations, `afterCreating` for profile creation, and `recycle()` for shared category. The result is clean, readable, and efficient.

---

#### Real-World Cases with Explanation

**Case 1: Enterprise Test Suite**

A large application's test suite uses factories with states, sequences, and recycling to generate precise test data for hundreds of test cases. The `dontExpandRelationshipsByDefault()` method is used in logic-only tests to keep them fast.

**Explanation**: Following factory best practices ensures tests are fast, reliable, and maintainable.

---

## References

- Eloquent: Factories – Laravel 13.x Documentation — https://laravel.com/docs/13.x/eloquent-factories
- Eloquent: Factories – Laravel 12.x Documentation — https://laravel.com/docs/12.x/eloquent-factories
- Eloquent: Factories – Laravel 11.x Documentation — https://laravel.com/docs/11.x/eloquent-factories
- Eloquent: Factories – Laravel 10.x Documentation — https://laravel.com/docs/10.x/eloquent-factories
- Factory Class – Laravel API Reference — https://api.laravel.com/docs/12.x/Illuminate/Database/Eloquent/Factories/Factory.html
- FactoryBuilder Class – Laravel API Reference — https://api.laravel.com/docs/12.x/Illuminate/Database/Eloquent/Factories/FactoryBuilder.html
- Controlling Factory Relationship Expansion in Laravel – Laravel News — https://laravel-news.com/dont-expand-relationships-by-default
- Optimizing Factory Data Creation with Laravel's recycle Method – Laravel News — https://laravel-news.com/recycle
- Simplify Factory Associations with Laravel's UseFactory Attribute – Laravel News — https://laravel-news.com/use-factory-attribute
- Factory makeMany() Method in Laravel 12.52.0 – Laravel News — https://laravel-news.com/factory-makemany-method
- Model Factories, States, Relationships, and Sequences – voku/agent-skills (GitHub) — https://github.com/voku/agent-skills/blob/main/skills/laravel-testing/rules/factory-definitions.md
- Laravel Factories and Seeders: All You Need to Know – Laravel Daily — https://laraveldaily.com/post/laravel-factories-seeders

---

**Version Note**: This cheat sheet is based on Laravel 10.x through 13.x documentation. Key version-specific features include: the `dontExpandRelationshipsByDefault()` method (Laravel 12.x+), the `UseFactory` attribute (Laravel 11.x+), and the `recycle()` method (Laravel 9.x+). Class-based factories were introduced in Laravel 8, replacing the global `factory()` helper. The `withoutAfterMaking()` and `withoutAfterCreating()` helpers were added in Laravel 12.52. Developers should always consult the documentation for their specific Laravel version.