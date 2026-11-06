# Laravel Testing Tools: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Laravel testing tools are the integrated ecosystem of frameworks and libraries—Pest, PHPUnit, Laravel Dusk, Mockery, Faker, and Collision—that enable developers to write, execute, and debug automated tests for Laravel applications, covering unit logic, HTTP features, browser interactions, test doubles, fake data, and human-readable error reporting.

**Technical Definition**

Laravel ships with first-class support for PHPUnit (the underlying test engine) and Pest (an expressive framework built on PHPUnit). The ecosystem extends to Laravel Dusk for real-browser end-to-end testing, Mockery for creating test doubles (mocks, stubs, spies, partial mocks), Faker (accessed via the `fake()` helper) for generating realistic test data in model factories, and Collision for rendering elegant, syntax-highlighted test output and error reports in the console. Together, these tools form a layered testing strategy: unit tests for isolated logic, feature tests for HTTP workflows, browser tests for JavaScript-dependent journeys, mocks for external dependencies, factories for data, and Collision for debugging clarity.

**Beginner-Friendly Explanation**

Laravel gives you a complete toolbox for testing. **PHPUnit** is the engine that runs your tests. **Pest** is a friendlier way to write them—more like plain English. **Dusk** lets you test your app in a real browser, clicking buttons and filling forms. **Mockery** creates “fake” versions of dependencies so you can test one thing at a time. **Faker** generates realistic fake data (names, emails, dates) for your tests. **Collision** makes error messages and test failures beautiful and easy to read. You do not have to use every tool for every test—you pick the right tool for the job.

---

### Key Characteristics

- **Layered Strategy**: Unit tests (PHPUnit/Pest) → Feature tests (PHPUnit/Pest + Laravel helpers) → Browser tests (Dusk/Pest Browser) → Mocking (Mockery) → Data (Faker) → Output (Collision).
- **PHPUnit as the Foundation**: Pest, Dusk, Mockery, and Collision all integrate with or build upon PHPUnit; PHPUnit remains the underlying test runner.
- **Pest as the Recommended Default**: For new Laravel projects, Pest is the recommended testing framework due to its expressive `test()` / `it()` syntax and `expect()` API.
- **Fakes Before Mocks**: Laravel’s built-in fakes (`Mail::fake()`, `Queue::fake()`, etc.) should be preferred for framework services; Mockery is reserved for your own container-resolved contracts and interaction verification.
- **Factories, Not Manual Inserts**: Faker-backed model factories are the standard way to generate test data, replacing raw database inserts.
- **Collision Included by Default**: Collision is bundled with Laravel and provides syntax-highlighted error reporting without additional configuration.
- **Version Sensitivity**: Pest 4/5, PHPUnit 11/12, Dusk 8.x, and Collision 8.x are the current versions in the Laravel 12–13 ecosystem; PHP attributes have replaced annotations in PHPUnit 12.

---

### Prerequisites

- **PHP 8.1+** (Laravel 10+) or **PHP 8.2+** (Laravel 11+); **Pest 4/5 requires PHP 8.4+**.
- **A Laravel application** with `composer.json` and `phpunit.xml` pre-configured.
- **Composer** for installing dev dependencies (`--dev`).
- **Google Chrome** for Dusk browser tests.
- **Basic knowledge of PHP, Laravel, and PHPUnit assertions**.
- **For Pest**: Familiarity with closure-based tests and the `expect()` API.

---

### Related Programming Areas

- **Test-Driven Development (TDD)**: The Red–Green–Refactor cycle is supported by Pest/PHPUnit and Laravel’s test generators.
- **Feature Testing**: HTTP tests that exercise routes, middleware, validation, and database persistence.
- **Unit Testing**: Isolated logic tests that do not boot Laravel.
- **Database Testing**: Factories, `RefreshDatabase`, and database assertions.
- **CI/CD**: All tools can run headlessly in GitHub Actions, GitLab CI, and other pipelines.

---

## Core Concepts

---

### 1. Pest

#### Definitions

**Core Definition**

Pest is an elegant, expressive PHP testing framework built on top of PHPUnit, replacing verbose test classes with readable closure-based `test()` and `it()` functions and a fluent `expect()` assertion API.

**Technical Definition**

Pest runs on the same PHPUnit engine, understanding the same assertions and lifecycle hooks, but provides a higher-level API: `test()` / `it()` for test declarations, `expect()` for assertions (inspired by Jest), datasets (`->with()`) for parameterized tests, higher-order tests (`it('is even', fn ($n) => expect($n % 2)->toBe(0))`), and first-party plugins for browser testing, mutation testing, and type coverage. Pest is installed as a dev dependency and initialized with `./vendor/bin/pest --init`, which creates a `Pest.php` configuration file.

**Beginner-Friendly Explanation**

Pest is a testing tool that makes writing tests feel natural. Instead of creating a class with methods that start with `test`, you write `it('does something', function () { ... })` and use `expect($value)->toBe($expected)`. It is built on PHPUnit, so it is powerful, but the syntax is much friendlier.

#### Purposes

- To provide an expressive, readable syntax that lowers the barrier to writing tests.
- To reduce boilerplate compared to traditional PHPUnit test classes.
- To support advanced features like datasets and higher-order tests with minimal code.
- To serve as the recommended default testing framework for new Laravel projects.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
<?php

test('basic example', function () {
    expect(true)->toBeTrue();
});

it('adds numbers', function () {
    expect(1 + 1)->toBe(2);
});

it('validates emails', function (string $email, bool $valid) {
    $validator = validator(['email' => $email], ['email' => 'email']);
    expect($validator->passes())->toBe($valid);
})->with([
    ['user@example.com', true],
    ['invalid-email', false],
]);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `test()` / `it()` | Declares a test with a description. |
| `expect($value)` | Begins an assertion chain. |
| `->toBe()` / `->toBeTrue()` | Assertions on the expected value. |
| `->with([...])` | Provides datasets for parameterized tests. |
| `Pest.php` | Configuration file created by `pest --init`. |

**Syntax Rules**

- Tests are closure-based functions, not class methods.
- `expect()` supports chaining and negation (`->not`).
- Datasets are passed via `->with()` and can be named for readable output.
- Higher-order tests allow expectations directly on properties/methods of the expectation value.

**Constraints and Limitations**

- Pest requires PHP 8.4+ in its current major version.
- Pest and PHPUnit should not be mixed within the same project; choose one framework consistently.
- The `expect()` API is inspired by Jest but does not cover every PHPUnit assertion; PHPUnit assertions remain available.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Basic Pest Test**

```php
<?php
// tests/Unit/ExampleTest.php

test('true is true', function () {
    expect(true)->toBeTrue();
});
```

**Expected Output**

```
PASS  Tests\Unit\ExampleTest
✓ true is true
```

**Why This Output Occurs**

The `test()` function declares a closure. `expect(true)->toBeTrue()` asserts that the value is strictly `true`, and the test passes.

---

**Example 2: Pest Datasets**

```php
it('adds numbers correctly', function (int $a, int $b, int $sum) {
    expect($a + $b)->toBe($sum);
})->with([
    'positive' => [1, 2, 3],
    'negative' => [-1, -2, -3],
    'zero'     => [0, 0, 0],
]);
```

**Expected Output**

```
PASS  Tests\Feature\AdditionTest
✓ it adds numbers correctly with data set "positive"
✓ it adds numbers correctly with data set "negative"
✓ it adds numbers correctly with data set "zero"
```

**Why This Output Occurs**

The dataset supplies three named input/output pairs. Pest runs the same test closure once per dataset entry, reporting each with its dataset name.

---

#### Real-World Cases with Explanation

**Case 1: Validation Rule Coverage**

A team needs to test an email validation rule against multiple invalid inputs. Using a Pest dataset, they write one test closure and supply `['invalid-email', false]`, `['missing@', false]`, `['valid@example.com', true]`, and other pairs. The entire input domain is covered without duplicating test code.

**Explanation**: Pest datasets reduce duplication and make parameterized tests a first-class, readable feature.

---

### 2. PHPUnit

#### Definitions

**Core Definition**

PHPUnit is the long-standing, standard testing framework for PHP, providing a class-based, xUnit-style architecture with test classes extending `PHPUnit\Framework\TestCase`, assertion methods, data providers, and lifecycle hooks.

**Technical Definition**

PHPUnit is the underlying test engine for Laravel, Pest, Dusk, and Collision. It provides `setUp()` / `tearDown()` lifecycle hooks, `#[Test]` and `#[DataProvider]` PHP attributes (annotations are deprecated in PHPUnit 11 and removed in PHPUnit 12), and a comprehensive assertion library (`assertSame`, `assertTrue`, `assertCount`, `expectException`, etc.). Laravel’s `phpunit.xml` configures the test suites, environment variables, and database settings.

**Beginner-Friendly Explanation**

PHPUnit is the classic way to write PHP tests. You create a class with methods that start with `test`, and each method is a test. It is powerful, mature, and used by millions of projects. Even if you use Pest, PHPUnit is running underneath.

#### Purposes

- To provide a mature, stable, and widely supported testing foundation.
- To support class-based test organization with lifecycle hooks.
- To serve as the engine for Pest, Dusk, Mockery, and Collision.
- To support data providers and test dependencies for complex test suites.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
<?php

namespace Tests\Unit;

use PHPUnit\Framework\TestCase;
use PHPUnit\Framework\Attributes\Test;
use PHPUnit\Framework\Attributes\DataProvider;

class CalculatorTest extends TestCase
{
    #[Test]
    public function it_adds_numbers(): void
    {
        $this->assertSame(3, 1 + 2);
    }

    #[Test]
    #[DataProvider('additionProvider')]
    public function it_adds_with_data(int $a, int $b, int $sum): void
    {
        $this->assertSame($sum, $a + $b);
    }

    public static function additionProvider(): array
    {
        return [
            'positive' => [1, 2, 3],
            'negative' => [-1, -2, -3],
        ];
    }
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `extends TestCase` | Inherits PHPUnit test capabilities. |
| `#[Test]` | Marks a method as a test (PHP attribute). |
| `#[DataProvider]` | Declares a data provider method. |
| `assertSame()` | Strict identity assertion. |
| `setUp()` / `tearDown()` | Lifecycle hooks (must call `parent`). |

**Syntax Rules**

- Test classes extend `PHPUnit\Framework\TestCase` (or Laravel’s `Tests\TestCase` for feature tests).
- Use PHP attributes (`#[Test]`, `#[DataProvider]`) exclusively; annotations are removed in PHPUnit 12.
- Data providers must be `public static` and return an array or generator.
- Lifecycle hooks must call `parent::setUp()` and `parent::tearDown()`.

**Constraints and Limitations**

- PHPUnit’s class-based syntax is more verbose than Pest.
- Mixing PHPUnit and Pest in the same project is discouraged.
- Unit tests extending the base `TestCase` do not boot Laravel and cannot access the database.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Basic PHPUnit Test**

```php
<?php
// tests/Unit/ExampleTest.php

namespace Tests\Unit;

use PHPUnit\Framework\TestCase;

class ExampleTest extends TestCase
{
    public function test_true_is_true(): void
    {
        $this->assertTrue(true);
    }
}
```

**Expected Output**

```
PASS  Tests\Unit\ExampleTest
✓ true is true
```

**Why This Output Occurs**

The `assertTrue(true)` assertion passes, and PHPUnit reports the test as successful.

---

**Example 2: Data Provider**

```php
#[Test]
#[DataProvider('users')]
public function it_validates_names(string $name, bool $valid): void
{
    $this->assertSame($valid, strlen($name) > 0);
}

public static function users(): array
{
    return [
        ['Alice', true],
        ['', false],
    ];
}
```

**Expected Output**

```
PASS  Tests\Unit\NameTest
✓ it validates names with data set #0
✓ it validates names with data set #1
```

**Why This Output Occurs**

The data provider supplies two input/output pairs. PHPUnit runs the test method once per pair and reports each as a separate dataset.

---

#### Real-World Cases with Explanation

**Case 1: Legacy Codebase**

A legacy Laravel application standardized on PHPUnit years ago. The team continues using PHPUnit because migrating to Pest would introduce unnecessary churn. PHPUnit’s stability and ecosystem compatibility are more valuable than Pest’s syntax for this project.

**Explanation**: PHPUnit remains the correct choice for brownfield projects already using it, and it is fully supported by Laravel.

---

### 3. Laravel Dusk

#### Definitions

**Core Definition**

Laravel Dusk is Laravel’s official browser automation and testing API, providing an expressive, easy-to-use interface for driving a real Chrome browser through end-to-end user journeys without requiring JDK or Selenium.

**Technical Definition**

Dusk uses a standalone ChromeDriver installation by default, eliminating the need for JDK or Selenium. It provides the `Browser` class with a fluent API (`visit`, `type`, `press`, `assertSee`, etc.), Dusk selectors (`@selector`), page objects, components, browser macros, and support for multiple simultaneous browsers. Dusk tests reside in `tests/Browser` and extend `Tests\DuskTestCase`. **Pest 4 now includes automated browser testing and is recommended for new projects**, but Dusk remains fully supported for existing applications.

**Beginner-Friendly Explanation**

Dusk is the tool that lets you test your app in a real browser. You write code that says “open this page, type this, click that, and check the result.” Dusk launches Chrome, does exactly that, and tells you if it worked. It catches JavaScript bugs that ordinary HTTP tests cannot see.

#### Purposes

- To test end-to-end user journeys in a real browser, including JavaScript-driven behaviour.
- To catch UI bugs that HTTP feature tests cannot detect.
- To provide an expressive, Laravel-integrated browser automation API.
- To support multiple browsers, page objects, and reusable browser macros.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
<?php
// tests/Browser/LoginTest.php

namespace Tests\Browser;

use Laravel\Dusk\Browser;
use Tests\DuskTestCase;

class LoginTest extends DuskTestCase
{
    public function test_user_can_login(): void
    {
        $this->browse(function (Browser $browser) {
            $browser->visit('/login')
                    ->type('email', 'user@example.com')
                    ->type('password', 'password')
                    ->press('Login')
                    ->assertPathIs('/dashboard');
        });
    }
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `$this->browse()` | Entry point; receives `Browser` instances. |
| `visit('/uri')` | Navigates to a URI. |
| `type()` / `press()` | Interacts with form elements. |
| `assertPathIs()` | Asserts the current URL path. |
| `@selector` | Dusk selector for stable element targeting. |

**Syntax Rules**

- Dusk tests extend `Tests\DuskTestCase`.
- `dusk:install` creates the `tests/Browser` directory and installs ChromeDriver.
- `APP_URL` must be set correctly in `.env`.
- Use `DatabaseMigrations` (not `RefreshDatabase`) because transactions cannot span HTTP requests.
- Dusk selectors (`@login-button`) are preferred over CSS selectors for stability.

**Constraints and Limitations**

- Dusk tests are slower than feature tests; reserve them for critical journeys.
- Pest 4 browser testing is now recommended for new projects over Dusk.
- ChromeDriver binaries must be executable (`chmod -R 0755 vendor/laravel/dusk/bin/`).

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Basic Dusk Test**

```php
$this->browse(function (Browser $browser) {
    $browser->visit('/login')
            ->type('email', 'user@example.com')
            ->type('password', 'password')
            ->press('Login')
            ->assertPathIs('/dashboard')
            ->assertSee('Welcome');
});
```

**Expected Output**

The browser fills the login form, submits it, is redirected to `/dashboard`, and sees the text “Welcome.”

**Why This Output Occurs**

Dusk drives Chrome through each interaction. `assertPathIs` confirms the redirect, and `assertSee` confirms the rendered text.

---

**Example 2: Database Isolation**

```php
use Illuminate\Foundation\Testing\DatabaseMigrations;

class RegistrationTest extends DuskTestCase
{
    use DatabaseMigrations;

    public function test_user_can_register(): void
    {
        $this->browse(function (Browser $browser) {
            $browser->visit('/register')
                    ->type('name', 'Alice')
                    ->type('email', 'alice@example.com')
                    ->press('Register')
                    ->assertPathIs('/home');
        });

        $this->assertDatabaseHas('users', ['email' => 'alice@example.com']);
    }
}
```

**Expected Output**

The test completes registration in the browser and then asserts the database record exists.

**Why This Output Occurs**

`DatabaseMigrations` resets the schema before the test, and the database assertion verifies persistence after the browser interaction.

---

#### Real-World Cases with Explanation

**Case 1: E-commerce Checkout**

An e-commerce application uses Dusk to test the full checkout flow: add to cart, proceed to checkout, fill shipping details, apply a coupon, and complete payment. This exercises JavaScript-driven UI elements that HTTP feature tests cannot reach.

**Explanation**: Dusk is essential for flows that depend on client-side rendering and third-party JavaScript SDKs.

---

### 4. Mockery

#### Definitions

**Core Definition**

Mockery is a simple yet flexible PHP mock object framework for unit testing, providing a succinct, human-readable DSL for creating test doubles—mocks, stubs, spies, and partial mocks—to isolate the code under test from its dependencies.

**Technical Definition**

Mockery integrates with PHPUnit (via `MockeryPHPUnitIntegration` or `MockeryTestCase`) and provides `Mockery::mock()` for full mocks, `Mockery::spy()` for spies, `->makePartial()` for partial mocks, and `shouldReceive()` / `expects()` / `allows()` for declaring expectations and return values. Laravel’s base test case provides `$this->mock()`, `$this->partialMock()`, and `$this->spy()` helpers that bind mocks into the container.

**Beginner-Friendly Explanation**

Mockery creates “fake” versions of your dependencies. If your code depends on a payment gateway, you can tell Mockery to create a fake gateway that returns `true` when `charge()` is called, so you can test your code without touching a real payment system.

#### Purposes

- To isolate the unit under test from its dependencies.
- To verify that the correct methods are called with the correct arguments.
- To replace slow, non-deterministic, or side-effect-heavy dependencies.
- To support spies and partial mocks when only some methods need to be controlled.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
use Mockery;

// Full mock with expectation
$gateway = Mockery::mock(PaymentGateway::class);
$gateway->shouldReceive('charge')
        ->once()
        ->with(100, 'usd')
        ->andReturn(true);

// Spy (verify after the fact)
$spy = Mockery::spy(AuditLogger::class);
// ... code executes ...
$spy->shouldHaveReceived('log')->with('event', $data);

// Partial mock
$partial = Mockery::mock(ReportService::class)->makePartial();
$partial->shouldReceive('calculateTotals')->andReturn(0);
```

**Component Breakdown**

| Method | Purpose |
|--------|---------|
| `Mockery::mock()` | Creates a full mock. |
| `shouldReceive()` | Declares an expected method call. |
| `->once()` / `->with()` | Narrows the expectation (call count, arguments). |
| `->andReturn()` | Provides a canned return value. |
| `Mockery::spy()` | Creates a spy that records calls. |
| `->makePartial()` | Creates a partial mock. |

**Syntax Rules**

- Mockery can be integrated with PHPUnit via `MockeryPHPUnitIntegration` or `MockeryTestCase`.
- Expectations are verified automatically in `tearDown()`.
- Laravel’s `$this->mock()` binds a mock into the container; `$this->spy()` creates a spy.
- Use `shouldReceive()` for stubs/mocks and `shouldHaveReceived()` for spies.

**Constraints and Limitations**

- Mocks require all called methods to be anticipated; unexpected calls fail.
- Mocking final classes/methods is not possible without refactoring.
- Do not mock the `Request` facade; use HTTP testing helpers instead.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Full Mock**

```php
$gateway = Mockery::mock(PaymentGateway::class);
$gateway->shouldReceive('charge')
        ->once()
        ->with(100, 'usd')
        ->andReturn(true);

$service = new PaymentService($gateway);
$service->processPayment(100, 'usd');
```

**Expected Output**

The test passes if `PaymentService` calls `charge(100, 'usd')` exactly once.

**Why This Output Occurs**

The mock declares a strict expectation. Any deviation (wrong arguments, wrong call count) causes an immediate failure.

---

**Example 2: Spy**

```php
$spy = Mockery::spy(AuditLogger::class);

$service = new UserService($spy);
$service->create(['name' => 'Alice']);

$spy->shouldHaveReceived('log')
    ->with('user.created', Mockery::on(fn ($data) => $data['name'] === 'Alice'));
```

**Expected Output**

The test passes if `UserService` called `log('user.created', ...)` with the correct payload.

**Why This Output Occurs**

The spy records all calls, and `shouldHaveReceived()` asserts after the fact that the expected call occurred.

---

#### Real-World Cases with Explanation

**Case 1: Notification Service**

A `NotificationService` depends on a `Mailer` contract. A unit test mocks the mailer with `shouldReceive('send')->once()->with($user, $message)`. If the service fails to send the email or sends it twice, the mock fails the test immediately.

**Explanation**: Mocking enforces the interaction contract between the service and its mailer, catching bugs that a stub might miss.

---

### 5. Faker

#### Definitions

**Core Definition**

Faker is a PHP library that generates realistic fake data—names, emails, addresses, dates, and more—and is accessed in Laravel through the `fake()` helper, primarily within model factories and seeders.

**Technical Definition**

Faker is bundled with Laravel and exposed via the `fake()` helper. Factories call methods like `fake()->name()`, `fake()->unique()->safeEmail()`, and `fake()->sentence()` in their `definition()` method to generate default attribute values. Faker provides three special modifiers: `unique()` (forces unique values), `optional()` (sometimes returns a default instead of a value), and `valid()` (only accepts values passing a validator). The application’s Faker locale can be configured via `faker_locale` in `config/app.php`.

**Beginner-Friendly Explanation**

Faker generates realistic fake data for your tests. Instead of typing “John Doe” and “john@example.com” manually, you call `fake()->name()` and `fake()->safeEmail()` and get realistic-looking values automatically. This makes your tests more realistic and less brittle.

#### Purposes

- To generate realistic test data without manual attribute assignment.
- To populate model factories and seeders with varied, plausible values.
- To prevent constraint violations with `unique()`.
- To expose edge cases by generating diverse data that manual fixtures might miss.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
use Illuminate\Database\Eloquent\Factories\Factory;

class UserFactory extends Factory
{
    public function definition(): array
    {
        return [
            'name'              => fake()->name(),
            'email'             => fake()->unique()->safeEmail(),
            'bio'               => fake()->optional()->sentence(),
            'email_verified_at' => now(),
        ];
    }
}
```

**Component Breakdown**

| Method | Description |
|--------|-------------|
| `fake()` | Returns a Faker generator instance. |
| `->name()` | Generates a random name. |
| `->unique()->safeEmail()` | Generates a unique, safe email. |
| `->optional()->sentence()` | Sometimes returns a sentence, sometimes `null`. |
| `->valid($validator, $provider)` | Only returns values that pass the validator. |

**Syntax Rules**

- Faker is used inside factories, seeders, or directly in tests via `fake()`.
- `unique()` must be called before the provider method.
- `optional()` accepts a weight and a default value.
- The locale is configured via `faker_locale` in `config/app.php`.

**Constraints and Limitations**

- `unique()` and `valid()` cannot be combined directly; only one modifier can be used at a time.
- Random data can make tests non-deterministic; use explicit overrides or sequences for assertions that depend on specific values.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Factory with Faker**

```php
class PostFactory extends Factory
{
    public function definition(): array
    {
        return [
            'title'   => fake()->sentence(),
            'body'    => fake()->paragraph(),
            'slug'    => fake()->unique()->slug(),
            'published_at' => fake()->optional()->dateTime(),
        ];
    }
}
```

**Expected Output**

Each `Post::factory()->create()` call generates a post with a random sentence title, paragraph body, unique slug, and optionally a published date.

**Why This Output Occurs**

Faker generates varied, realistic values for each attribute. `unique()` ensures no duplicate slugs, and `optional()` sometimes leaves `published_at` as `null`.

---

**Example 2: Using Faker Directly in a Test**

```php
test('a user can be created with fake data', function () {
    $user = User::factory()->create([
        'name' => fake()->name(),
        'email' => fake()->unique()->safeEmail(),
    ]);

    expect($user->name)->not->toBeEmpty();
    expect($user->email)->toContain('@');
});
```

**Expected Output**

The test passes, confirming that the generated name and email are non-empty and contain an `@` symbol.

**Why This Output Occurs**

Faker generates a realistic name and a unique safe email, and the assertions confirm basic validity.

---

#### Real-World Cases with Explanation

**Case 1: Seeding Development Data**

A development seeder uses `User::factory()->count(50)->create()` to populate the database with 50 users, each with a unique email and realistic name. Developers get a realistic dataset without manual data entry.

**Explanation**: Faker-backed factories make it trivial to generate large, realistic datasets for development and testing.

---

### 6. Collision

#### Definitions

**Core Definition**

Collision is a PHP package that provides beautiful, syntax-highlighted error reporting and test output formatting for command-line applications, and is included by default in Laravel.

**Technical Definition**

Collision integrates with Laravel, Symfony, PHPUnit, and Pest to render readable, visually appealing exception reports and test failure output in the console. It is built on top of the Whoops error handler and provides enhanced stack traces, solution suggestions (via Ignition integration), and formatted test results. Collision is a dev dependency and should not be used as a production error tracker.

**Beginner-Friendly Explanation**

Collision makes error messages and test failures look beautiful and easy to read in your terminal. Instead of a wall of confusing text, you get syntax-highlighted code snippets, clear error locations, and helpful suggestions.

#### Purposes

- To provide human-readable, syntax-highlighted error reports in the console.
- To format PHPUnit/Pest test failures with clear context.
- To integrate with Laravel’s Ignition error page for solution suggestions.
- To improve the developer experience when debugging failing tests.

#### Syntax Rules and Structure

**Complete General Syntax**

Collision is included with Laravel and requires no configuration. It automatically registers itself via Laravel’s service provider. For non-Laravel projects, it can be registered manually:

```php
(new \NunoMaduro\Collision\Provider)->register();
```

**Component Breakdown**

| Feature | Description |
|---------|-------------|
| Exception formatting | Syntax-highlighted error reports. |
| Test output formatting | Clear, readable test failure output. |
| Solution providers | Suggests fixes for common errors. |
| Laravel integration | Automatic via service provider. |

**Syntax Rules**

- Collision is included in Laravel’s `require-dev` and works out of the box.
- No explicit configuration is required in Laravel applications.
- For other frameworks, register the provider manually.

**Constraints and Limitations**

- Collision is a development dependency; it should not be used in production.
- It formats errors but does not record them; use a dedicated error tracker for production monitoring.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Beautiful Test Failure Output**

When a test fails, Collision renders a formatted output:

```
FAIL  Tests\Feature\PostTest
⨯ it creates a post
  Failed asserting that false is true.
  at tests/Feature/PostTest.php:12
    8 │ test('it creates a post', function () {
    9 │     $response = $this->post('/posts', []);
   10 │     $response->assertCreated();
   11 │     expect(true)->toBeTrue();
  > 12 │ });
```

**Expected Output**

The failure is displayed with syntax highlighting, line numbers, and a clear pointer to the failing assertion.

**Why This Output Occurs**

Collision intercepts PHPUnit/Pest test events and formats them with syntax highlighting and precise code context.

---

#### Real-World Cases with Explanation

**Case 1: CI Debugging**

A test fails in CI. Instead of a cryptic stack trace, the CI log shows a Collision-formatted error with the exact line and a highlighted code snippet. The developer identifies the bug in seconds rather than minutes.

**Explanation**: Collision reduces debugging time by making failures immediately understandable.

---

## Testing Tools Best Practices

1. **Choose One Framework: Pest or PHPUnit**: For new projects, Pest is recommended; for brownfield projects already on PHPUnit, stay with PHPUnit. Do not mix them.

2. **Use Fakes Before Mocks for Framework Services**: For `Mail`, `Queue`, `Event`, `Bus`, `Notification`, `Storage`, and `Http`, use Laravel’s built-in fakes. Reserve Mockery for your own container-resolved contracts.

3. **Use Factories, Never Manual Inserts**: Every model that appears in tests should have a factory. Use Faker for realistic data and `unique()` to prevent constraint violations.

4. **Use `RefreshDatabase` for Database Isolation**: It is the recommended trait for most tests—fast, reliable, and compatible with in-memory SQLite.

5. **Reserve Dusk (or Pest Browser) for Critical Journeys**: Browser tests are slow; use them for login, registration, checkout, and other high-value flows. Use feature tests for everything else.

6. **Use Dusk Selectors (`@selector`) for Browser Tests**: They are stable across CSS refactors and UI changes.

7. **Control Time and Randomness**: Use `freezeTime()` / `travelTo()` and `Str::createRandomStringsUsing()` to make time- and randomness-dependent tests deterministic.

8. **Run Tests in Parallel Where Possible**: Pest and PHPUnit support parallel testing via ParaTest, significantly reducing suite execution time.

9. **Integrate Tests into CI/CD**: Run `php artisan test` (or `vendor/bin/pest`) on every push to catch regressions before deployment.

10. **Keep Unit Tests Fast**: If a “unit test” takes more than a few milliseconds, it is probably a feature test in disguise. Move it to `tests/Feature`.

11. **Use Collision for Debugging, Not Production Monitoring**: Collision formats errors beautifully in development and CI; use a dedicated error tracker in production.

12. **Stay Version-Aware**: Pest 4/5 requires PHP 8.4+, PHPUnit 12 removed annotations in favour of attributes, and Pest 4 browser testing is now recommended for new projects over Dusk. Always consult the documentation for your Laravel and PHP versions.

---

## References

- Pest Installation — https://pestphp.com/docs
- Pest on Packagist (v5.1.3) — https://packagist.org/packages/pestphp/pest
- Laravel Testing: Getting Started — https://github.com/laravel/docs/blob/091001642ea6991a0790d5e6f1677355aec0bda7/testing.md
- Laravel Dusk — https://github.com/laravel/docs/blob/625d1dfb61018e6bc50cc7ab67fca89ae018dfa2/dusk.md
- Mockery Documentation — https://docs.mockery.io/
- Laravel Eloquent Factories — https://laravel.com/docs/10.x/eloquent-factories
- Collision on Packagist — https://packagist.org/packages/nunomaduro/collision
- PHPUnit Manual — https://docs.phpunit.de/
- FakerPHP Documentation — https://fakerphp.org/
- Laravel Testing Best Practices (Community Skill) — https://github.com/affaan-m/ECC/blob/main/skills/laravel-tdd/SKILL.md
- Pest Higher Order Testing — https://pestphp.com/docs/higher-order-testing
- Collision DeepWiki (Test Output Formatting) — https://deepwiki.com/nunomaduro/collision

---

**Version Note**: This cheat sheet reflects the Laravel 12–13 ecosystem as of late 2026. Pest 4/5 requires PHP 8.4+; PHPUnit 12 removed doc-comment annotations in favour of PHP attributes; Laravel Dusk remains supported but Pest 4 browser testing is recommended for new projects. Mockery, Faker, and Collision are bundled with Laravel and require no additional installation for standard use. Always verify compatibility with your specific Laravel, PHP, and package versions.