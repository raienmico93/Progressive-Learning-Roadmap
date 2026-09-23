# Laravel Database Factories (Class-Based Syntax): A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Laravel Database Factories are class-based blueprints for generating fake model data, primarily used for testing and database seeding.

**Technical Definition:** A factory is a PHP class extending `Illuminate\Database\Eloquent\Factories\Factory` that implements a `definition()` method returning an array of default attribute values. The `HasFactory` trait on a model provides a static `factory()` method that resolves the corresponding factory class via naming conventions. Factory methods like `create()` and `make()` instantiate models, with `create()` persisting them to the database. The `state()` method applies discrete attribute transformations, and fluent relationship methods (`has()`, `for()`, `hasAttached()`) compose related models in a single chain.

**Beginner-Friendly Explanation:** Factories are "recipes" for creating fake records. Instead of manually writing `User::create(['name' => 'John', 'email' => 'john@test.com'])` a hundred times, you define a factory once and call `User::factory()->create()` to generate as many realistic records as you need. You can also create variations (states) like `->suspended()` or `->admin()`, and define relationships like "create a user with three posts" in a single line.

### Key Characteristics

- **Class-Based Definitions:** Each model has a corresponding factory class in `database/factories/`.
- **Fake Data Generation:** The `fake()` helper provides access to Faker PHP for realistic random data.
- **Convention-Based Resolution:** The `HasFactory` trait maps models to factories automatically.
- **Persistence vs. In-Memory:** `create()` saves to the database; `make()` does not.
- **Discrete States:** Named state methods (e.g., `unpublished()`, `suspended()`) define model variations.
- **Fluent Relationships:** `has()`, `for()`, and `hasAttached()` compose relationships in one chain.

### Prerequisites

- A Laravel application with Eloquent models.
- The `HasFactory` trait added to models you want to use with factories.
- Factory classes generated via `php artisan make:factory ModelNameFactory`.
- A working database connection for `create()` operations.
- Understanding of basic Eloquent relationships (hasMany, belongsTo, belongsToMany).

### Related Programming Areas

- **Database Seeding:** Factories are the primary tool for populating development databases.
- **Testing:** Factories generate test data for unit, feature, and integration tests.
- **Eloquent Relationships:** Factories can compose relationships directly.
- **Faker Library:** Factories use Faker to generate realistic random data.

### Core Concepts / Features

1. Defining Factories (`definition()` and the `fake()` Helper)
2. The `HasFactory` Trait (`User::factory()`)
3. Persisting vs. In-Memory (`create()` vs. `make()`)
4. Factory States (`state()` and Named State Methods)
5. Relationship Management (`has()`, `for()`, `hasAttached()`)

---

## 1. Defining Factories (`definition()` and the `fake()` Helper)

### Definitions

**Core Definition:** A factory definition is the `definition()` method on a factory class that returns an array of default attribute values for generated models.

**Technical Definition:** The `definition()` method must return an associative array where keys are database column names and values are either static values, closures, or the result of calling the `fake()` helper. The `fake()` helper resolves the `Faker\Generator` instance from the container, providing access to all Faker formatters (e.g., `name()`, `email()`, `paragraph()`). Factory attributes are evaluated when the factory is instantiated.

**Beginner-Friendly Explanation:** The `definition()` method is where you describe what a "default" model looks like. If you're defining a `UserFactory`, you'd say: name is a fake name, email is a fake safe email, password is a hashed string. The `fake()` helper gives you realistic random data for each attribute.

### Purposes

- To define reusable, default attribute values for generated models.
- To leverage Faker's formatters for realistic random data generation.
- To centralise fake data logic in one place instead of repeating it in tests.
- To support closures and dynamic values based on other attributes.
- To enable `create()` and `make()` to generate models without manual attribute specification.

### Syntax Rules and Structure

**Complete General Syntax:**

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
            'remember_token' => Str::random(10),
        ];
    }
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `extends Factory` | Base factory class. |
| `$model` | The model class this factory creates (auto-inferred by convention). |
| `definition()` | Returns array of default attribute values. |
| `fake()` | Global helper resolving the Faker generator. |
| `fake()->name()` | Faker formatter returning a random person name. |
| `fake()->unique()->safeEmail()` | Ensures the email is unique across all calls. |

**Syntax Rules:**
- `definition()` must return an array.
- The `$model` property is optional if the factory follows naming conventions (`UserFactory` → `User`).
- Use `fake()` (global helper) or `$this->faker` (instance property) to access Faker.
- Closures as values receive the evaluated attribute array and can reference other attributes.

**Constraints and Limitations:**
- Faker's `unique()` modifier can exhaust available values with large counts.
- Factory attributes are evaluated at instantiation, not at definition.
- The `definition()` method is called once per factory instance, not per model.

### Multiple Annotated Complete Code Examples

**Example 1: Basic User Factory**

```php
<?php
// database/factories/UserFactory.php

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
            'email_verified_at' => now(),
            'password' => bcrypt('password'),
            'remember_token' => Str::random(10),
        ];
    }
}
```

**Expected Output:** When `User::factory()->create()` is called, a new user is inserted with a random name, unique email, verified timestamp, hashed password, and random remember token.

**Why:** The `definition()` method provides default values for every column. The `fake()` helper generates realistic random data.

---

**Example 2: Factory with Dynamic Closure Values**

```php
public function definition(): array
{
    return [
        'user_id' => User::factory(),
        'title' => fake()->sentence(),
        'slug' => fn (array $attributes) => Str::slug($attributes['title']),
        'published_at' => fn (array $attributes) => $attributes['user_id'] ? now() : null,
    ];
}
```

**Expected Output:** The `slug` is derived from the `title` attribute via a closure. The `published_at` depends on whether `user_id` is set.

**Why:** Closures receive the evaluated attribute array, allowing values to be computed from other attributes in the same factory definition.

### Real-World Cases

- **User Registration Testing:** Generate 50 users with realistic names and emails for testing registration flows.
- **Blog Post Seeding:** Generate posts with random titles, slugs, and content for development databases.
- **Order Factories:** Create orders with random totals, statuses, and timestamps for testing checkout logic.

### References

- Eloquent: Factories — https://laravel.com/docs/12.x/eloquent-factories
- Eloquent：工厂 — https://laravel.net.cn/docs/12.x/eloquent-factories

---

## 2. The `HasFactory` Trait (`User::factory()`)

### Definitions

**Core Definition:** The `HasFactory` trait is added to Eloquent models to provide a static `factory()` method that instantiates the model's corresponding factory class.

**Technical Definition:** The `Illuminate\Database\Eloquent\Factories\HasFactory` trait defines a static `factory()` method that resolves the factory class from the `Database\Factories` namespace using the model's class name plus the `Factory` suffix. If the convention does not match, the `newFactory()` method can be overridden to return a specific factory instance.

**Beginner-Friendly Explanation:** Adding `use HasFactory` to a model lets you write `User::factory()` instead of `factory(User::class)`. It's a modern, type-safe shortcut that Laravel introduced in version 8.

### Purposes

- To provide a convenient, model-centric syntax for instantiating factories.
- To enable convention-based resolution of factory classes.
- To improve IDE support and type hinting compared to the old `factory()` global helper.
- To allow custom factory resolution via `newFactory()` when conventions don't apply.

### Syntax Rules and Structure

**Complete General Syntax:**

```php
namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    use HasFactory;
}

// Usage
$user = User::factory()->create();
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `use HasFactory;` | Adds the `factory()` static method to the model. |
| `User::factory()` | Resolves `Database\Factories\UserFactory` and returns an instance. |

**Complete General Syntax — Custom Factory Resolution:**

```php
protected static function newFactory(): UserFactory
{
    return UserFactory::new();
}
```

**Syntax Rules:**
- The `HasFactory` trait must be imported and used in the model class.
- The factory class must be in the `Database\Factories` namespace.
- The factory class name must be `{ModelName}Factory`.
- Override `newFactory()` for non-standard factory locations or names.

**Constraints and Limitations:**
- Without the trait, the legacy `factory(User::class)` global helper must be used.
- Factory resolution does not use the service container, so constructor dependencies are not injected.

### Multiple Annotated Complete Code Examples

**Example 1: Standard HasFactory Usage**

```php
<?php
// app/Models/User.php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    use HasFactory;
}
```

```php
// Usage
use App\Models\User;

$user = User::factory()->create();
$users = User::factory()->count(10)->create();
```

**Expected Output:** A single user or 10 users are created and persisted to the database.

**Why:** The `HasFactory` trait's `factory()` method resolves `Database\Factories\UserFactory` and returns an instance ready for `create()` or `make()`.

---

**Example 2: Custom `newFactory()` Override**

```php
protected static function newFactory(): UserFactory
{
    return UserFactory::new();
}
```

**Expected Output:** The model uses the explicitly specified factory class.

**Why:** When factory conventions don't apply (e.g., factories in a different namespace), `newFactory()` provides the correct instance.

### Real-World Cases

- **Every Model with a Factory:** All testable models should use `HasFactory`.
- **Package Development:** Packages can define their own factories and override `newFactory()`.

### References

- Laravel what is use HasFactory? — https://stackoverflow.com/feeds/question/73601972

---

## 3. Persisting vs. In-Memory (`create()` vs. `make()`)

### Definitions

**Core Definition:** `create()` instantiates models and persists them to the database, while `make()` instantiates models in memory without saving them.

**Technical Definition:** The `create()` method calls `save()` on each generated model, executing database INSERT statements. The `make()` method returns model instances with attributes populated but without calling `save()`. Both methods accept an optional array of attribute overrides. Relationship methods (`has()`, `for()`) respect the parent's `create()` or `make()` context when using the `lazy()` method.

**Beginner-Friendly Explanation:** Use `create()` when you need the data in the database (like in feature tests or seeders). Use `make()` when you just need model objects to test logic without hitting the database (like unit tests).

### Purposes

- To persist generated models to the database using `create()`.
- To generate in-memory model instances using `make()` for unit testing.
- To override default factory attributes by passing an array to either method.
- To avoid database writes in tests that don't require persistence.
- To support relationship composition that respects the parent's persistence context.

### Syntax Rules and Structure

**Complete General Syntax — `create()`:**

```php
$user = User::factory()->create();
$users = User::factory()->count(3)->create();
$user = User::factory()->create(['name' => 'Abigail']);
```

**Complete General Syntax — `make()`:**

```php
$user = User::factory()->make();
$users = User::factory()->count(3)->make();
$user = User::factory()->make(['name' => 'Abigail']);
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `create()` | Instantiates and persists models via `save()`. |
| `make()` | Instantiates models without persistence. |
| `count(n)` | Generates `n` models. |
| `['name' => 'Abigail']` | Overrides default factory attributes. |

**Syntax Rules:**
- Both methods accept an overrides array as the first argument.
- `create()` returns the created model(s) with database IDs populated.
- `make()` returns model instances without IDs (unless the factory supplies one).
- `create()` triggers model events; `make()` does not.

**Constraints and Limitations:**
- `make()` models cannot be used in tests that rely on database lookups.
- `create()` requires a working database connection and migrations.
- Relationship methods default to `create()` context; use `lazy()` to respect `make()`.

### Multiple Annotated Complete Code Examples

**Example 1: `create()` Persists to Database**

```php
use App\Models\User;

// Create a single user — INSERT statement executed
$user = User::factory()->create();
echo $user->id; // e.g., 1

// Create three users — three INSERT statements
$users = User::factory()->count(3)->create();
echo $users->count(); // 3
```

**Expected Output:** Users are inserted into the database and have IDs populated.

**Why:** `create()` calls `save()` on each model, executing INSERT statements.

---

**Example 2: `make()` Does Not Persist**

```php
use App\Models\User;

// Create in-memory user — no database query
$user = User::factory()->make();
echo $user->name; // Random name
echo $user->id;   // null (not persisted)

// Create three in-memory users
$users = User::factory()->count(3)->make();
```

**Expected Output:** Model instances are created with attributes populated but no database writes occur. `id` is `null`.

**Why:** `make()` returns model instances without calling `save()`, so no INSERT statements are executed.

### Real-World Cases

- **Feature Tests:** `User::factory()->create()` for tests that hit the database.
- **Unit Tests:** `User::factory()->make()` for testing model logic without database.
- **Database Seeding:** `User::factory()->count(50)->create()` for populating development databases.
- **API Response Testing:** `make()` for testing serialization logic without database overhead.

### References

- Laravel factory - What's the difference between create and make — https://stackoverflow.com/feeds/question/44119401
- Eloquent: Factories — https://laravel.com/docs/12.x/eloquent-factories

---

## 4. Factory States (`state()` and Named State Methods)

### Definitions

**Core Definition:** A factory state is a discrete variation of a model's default attributes, defined as a named method that modifies the base definition.

**Technical Definition:** The `state()` method on a factory accepts a closure that receives the base attribute array and returns an array of overrides. Named state methods (e.g., `suspended()`, `unpublished()`) call `state()` internally and return `$this` for fluent chaining. Factory states can also use the `Sequence` class to alternate attribute values across multiple models.

**Beginner-Friendly Explanation:** States let you create "flavors" of a model. Your `UserFactory` might have an `admin()` state that sets `is_admin = true`, or a `suspended()` state that sets `account_status = 'suspended'`. You call `User::factory()->admin()->create()` to get an admin user.

### Purposes

- To define discrete model variations (admin, suspended, published, draft).
- To avoid repeating attribute overrides across tests.
- To compose multiple states in a single factory chain.
- To alternate attribute values across multiple models using sequences.
- To create expressive, self-documenting test setup code.

### Syntax Rules and Structure

**Complete General Syntax — Named State Method:**

```php
public function suspended(): static
{
    return $this->state(fn (array $attributes) => [
        'account_status' => 'suspended',
    ]);
}
```

**Complete General Syntax — Inline `state()`:**

```php
User::factory()->state([
    'name' => 'Abigail',
])->create();
```

**Complete General Syntax — Sequence:**

```php
use Illuminate\Database\Eloquent\Factories\Sequence;

$users = User::factory()
    ->count(10)
    ->state(new Sequence(
        ['admin' => 'Y'],
        ['admin' => 'N'],
    ))
    ->create();
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `state(fn($attributes) => [...])` | Modifies base attributes. |
| Named method | A descriptive wrapper around `state()`. |
| `Sequence` | Alternates attributes across generated models. |
| `$sequence->index` | The current iteration index in a sequence closure. |

**Syntax Rules:**
- Named state methods should return `static` (or `Factory`) for chaining.
- State methods can be chained in any order.
- Sequences alternate through provided arrays or closures.
- The `trashed()` state is built-in for models using `SoftDeletes`.

**Constraints and Limitations:**
- State methods receive the **base** attribute array, not the result of previous states unless chained appropriately.
- Sequences with a single array apply the same values to all models.

### Multiple Annotated Complete Code Examples

**Example 1: Named State Method**

```php
public function suspended(): static
{
    return $this->state(fn (array $attributes) => [
        'account_status' => 'suspended',
    ]);
}

// Usage
$user = User::factory()->suspended()->create();
```

**Expected Output:** A user is created with `account_status = 'suspended'`.

**Why:** The `suspended()` method calls `state()` with a closure that overrides the `account_status` attribute.

---

**Example 2: Sequence for Alternating Values**

```php
use Illuminate\Database\Eloquent\Factories\Sequence;

$users = User::factory()
    ->count(10)
    ->state(new Sequence(
        ['admin' => 'Y'],
        ['admin' => 'N'],
    ))
    ->create();
```

**Expected Output:** Five users have `admin = 'Y'`, and five have `admin = 'N'`.

**Why:** The `Sequence` cycles through the provided arrays for each generated model.

---

**Example 3: Inline State Transformation**

```php
$user = User::factory()
    ->state([
        'name' => 'Abigail Otwell',
        'email' => 'abigail@example.com',
    ])
    ->create();
```

**Expected Output:** A user is created with the specified name and email, overriding the factory defaults.

**Why:** The `state()` method applies inline attribute overrides without defining a named method.

### Real-World Cases

- **Admin vs. Regular Users:** `->admin()` and `->regular()` states.
- **Published vs. Draft Posts:** `->published()` and `->unpublished()` states.
- **Order Statuses:** `->pending()`, `->shipped()`, `->delivered()` states.
- **Soft-Deleted Records:** `->trashed()` built-in state.

### References

- Factory Patterns — https://raw.githubusercontent.com/iliaal/whetstone/refs/heads/master/plugins/whetstone/skills/ia-php-laravel/references/factories.md
- Laravel Factories and Seeders: All You Need to Know — https://laraveldaily.com/post/laravel-factories-seeders?mtm_campaign=20250417-newsletter

---

## 5. Relationship Management (`has()`, `for()`, `hasAttached()`)

### Definitions

**Core Definition:** Factory relationship methods allow defining and creating related models directly within the factory chain, composing complex data graphs in a single expression.

**Technical Definition:** The `has()` method creates child models for a parent (hasMany, hasOne), the `for()` method creates a parent for a child (belongsTo), and `hasAttached()` manages many-to-many relationships with pivot table attributes. Magic methods like `hasPosts()` and `forUser()` provide convention-based shortcuts.

**Beginner-Friendly Explanation:** Instead of creating a user, then creating posts separately and linking them, you can write `User::factory()->hasPosts(3)->create()` to create a user with three posts in one line. The `for()` method does the reverse: `Post::factory()->forUser()->create()` creates a post with a new user.

### Purposes

- To compose related models in a single factory chain.
- To avoid manual relationship setup after model creation.
- To apply states to related models during creation.
- To manage pivot table attributes for many-to-many relationships.
- To create realistic data graphs for testing and seeding.

### Syntax Rules and Structure

**Complete General Syntax — `has()` (HasMany):**

```php
$user = User::factory()
    ->has(Post::factory()->count(3))
    ->create();
```

**Complete General Syntax — `for()` (BelongsTo):**

```php
$post = Post::factory()
    ->for(User::factory()->state(['name' => 'Jessica']))
    ->create();
```

**Complete General Syntax — `hasAttached()` (Many-to-Many):**

```php
$user = User::factory()
    ->hasAttached(
        Role::factory()->count(3),
        ['active' => true]
    )
    ->create();
```

**Complete General Syntax — Magic Methods:**

```php
User::factory()->hasPosts(3)->create();
Post::factory()->forUser()->create();
User::factory()->hasRoles(1, ['name' => 'Editor'])->create();
```

**Component Breakdown:**

| Method | Relationship | Description |
|--------|-------------|-------------|
| `has(Factory)` | hasMany / hasOne | Creates child models. |
| `for(Factory\|Model)` | belongsTo | Creates or uses a parent model. |
| `hasAttached(Factory, Pivot)` | belongsToMany | Creates related models with pivot attributes. |
| `hasPosts(3)` | hasMany (magic) | Convention-based `has(Post::factory()->count(3))`. |
| `forUser()` | belongsTo (magic) | Convention-based `for(User::factory())`. |

**Syntax Rules:**
- `has()` and `for()` accept either factory instances or model instances.
- Relationship names are inferred from the related model class (e.g., `Post::factory()` → `posts` relationship).
- Explicit relationship names can be passed as the second argument to `has()`.
- Pivot attributes are passed as the second argument to `hasAttached()`.
- Magic methods derive relationship names from the method name (e.g., `hasPosts` → `posts`).

**Constraints and Limitations:**
- Magic methods may not be used for `morphTo` relationships; use `for()` explicitly.
- Relationship methods default to `create()` context; use `lazy()` to respect `make()`.
- Custom pivot models require the `using()` method for advanced pivot behaviour.

### Multiple Annotated Complete Code Examples

**Example 1: HasMany Relationship**

```php
use App\Models\Post;
use App\Models\User;

// Create a user with three posts
$user = User::factory()
    ->has(Post::factory()->count(3))
    ->create();

echo $user->posts()->count(); // 3

// Using magic method
$user = User::factory()->hasPosts(3)->create();

// With state on related models
$user = User::factory()
    ->has(Post::factory()->count(3)->state([
        'published' => false,
    ]))
    ->create();
```

**Expected Output:** A user is created with three posts, all with `user_id` pointing to the new user.

**Why:** The `has()` method creates the related models and sets the foreign key automatically.

---

**Example 2: BelongsTo Relationship**

```php
use App\Models\Post;
use App\Models\User;

// Create three posts belonging to a specific user
$posts = Post::factory()
    ->count(3)
    ->for(User::factory()->state([
        'name' => 'Jessica Archer',
    ]))
    ->create();

// Using magic method
$posts = Post::factory()
    ->count(3)
    ->forUser(['name' => 'Jessica Archer'])
    ->create();

// Using an existing model instance
$user = User::factory()->create();
$posts = Post::factory()->count(3)->for($user)->create();
```

**Expected Output:** Three posts are created with `user_id` pointing to the specified or newly created user.

**Why:** The `for()` method creates or uses a parent model and sets the foreign key on the child.

---

**Example 3: Many-to-Many Relationship**

```php
use App\Models\Role;
use App\Models\User;

// Create a user with three roles, with pivot attributes
$user = User::factory()
    ->hasAttached(
        Role::factory()->count(3),
        ['active' => true]
    )
    ->create();

// Using magic method
$user = User::factory()->hasRoles(1, ['name' => 'Editor'])->create();

// Attaching existing models
$roles = Role::factory()->count(3)->create();
$user = User::factory()
    ->hasAttached($roles, ['active' => true])
    ->create();
```

**Expected Output:** A user is created with three roles attached via the `role_user` pivot table, each with `active = true`.

**Why:** The `hasAttached()` method creates the related models and inserts pivot table rows with the specified attributes.

---

**Example 4: Defining Relationships Within Factories**

```php
// In PostFactory::definition()
public function definition(): array
{
    return [
        'user_id' => User::factory(),
        'title' => fake()->sentence(),
        'content' => fake()->paragraph(),
    ];
}

// Usage — creates a user automatically
$post = Post::factory()->create();
```

**Expected Output:** A post is created, and a new user is automatically created for it.

**Why:** Assigning a factory instance to the `user_id` foreign key creates the parent model automatically.

### Real-World Cases

- **Blog Seeding:** `User::factory()->hasPosts(5)->create()` for realistic blog data.
- **E-commerce:** `Order::factory()->hasItems(3)->forUser()->create()` for order data.
- **Role Management:** `User::factory()->hasRoles(2)->create()` for user-role assignments.
- **Polymorphic Data:** `Post::factory()->hasComments(3)->create()` for comments.

### References

- Eloquent: Factories (Relationships) — https://laravel.com/docs/12.x/eloquent-factories#factory-relationships
- Laravel factory relationships... respect create() or make() — https://stackoverflow.com/feeds/question/44123803

---

## References

- Eloquent: Factories — https://laravel.com/docs/12.x/eloquent-factories
- Eloquent：工厂 — https://laravel.net.cn/docs/12.x/eloquent-factories
- Laravel Factories and Seeders: All You Need to Know — https://laraveldaily.com/post/laravel-factories-seeders?mtm_campaign=20250417-newsletter
- Laravel factory - What's the difference between create and make — https://stackoverflow.com/feeds/question/44119401
- Laravel factory relationships... respect create() or make() — https://stackoverflow.com/feeds/question/44123803
- Laravel what is use HasFactory? — https://stackoverflow.com/feeds/question/73601972
- Factory Patterns — https://raw.githubusercontent.com/iliaal/whetstone/refs/heads/master/plugins/whetstone/skills/ia-php-laravel/references/factories.md