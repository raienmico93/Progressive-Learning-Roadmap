# Laravel Unit Testing: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Unit testing in Laravel is the practice of writing automated tests that target small, isolated pieces of application code—typically a single class or method—without booting the Laravel framework, to verify that each unit behaves exactly as specified.

**Technical Definition**

Laravel unit tests reside in `tests/Unit` and extend PHPUnit's base `TestCase` (either `PHPUnit\Framework\TestCase` or `Tests\TestCase`), unlike feature tests which boot the full application kernel. Because the framework is not loaded, unit tests cannot access the database, service container, facades, or Laravel helpers by default; they test pure PHP logic, utility classes, value objects, and self-contained services. Isolation is achieved through test doubles—mocks, stubs, spies, and Laravel's facade fakes—which replace external dependencies and record interactions for post‑execution assertions.

**Beginner-Friendly Explanation**

Unit testing means writing tiny scripts that check one small piece of your code at a time—like a single method that calculates a discount or a function that formats a date. Because these tests do not start up the whole Laravel framework, they run in milliseconds and tell you precisely where a bug lives. If a feature test says “checkout is broken,” a unit test says “the tax calculation returns 18.3 instead of 18.0.”

---

### Key Characteristics

- **No Framework Boot**: Unit tests extend `PHPUnit\Framework\TestCase` directly, making them the fastest tests in the suite.
- **Pure Logic Focus**: Ideal for utility classes, helper functions, value objects, and services that do not depend on Laravel internals.
- **Test Doubles as the Isolation Tool**: Mocking, stubbing, and spying replace collaborators so that the unit under test is the only real code executed.
- **Laravel-Specific Fakes**: Even though unit tests avoid the framework, Laravel provides facade fakes (`Http::fake()`, `Queue::fake()`, etc.) that can be used in any test to intercept and assert side effects.
- **Framework-Agnostic Assertions**: Unit tests rely on PHPUnit’s assertion library (`assertSame`, `assertTrue`, `assertEquals`) rather than HTTP‑response assertions like `assertStatus()`.

---

### Prerequisites

- **PHP 8.1+** (Laravel 10+) or **PHP 8.2+** (Laravel 11+).
- **A Laravel application** with the pre‑configured `phpunit.xml` and `tests/Unit` directory.
- **Pest or PHPUnit** installed (both ship with Laravel by default).
- **Composer** for installing Mockery if custom test doubles are needed (Mockery is already a Laravel dev dependency).
- **Basic knowledge of PHPUnit assertions** and the PHP class under test.

---

### Related Programming Areas

- **Feature Testing**: Boots Laravel and tests HTTP workflows; unit tests complement feature tests by isolating logic.
- **Mocking and Fakes**: The primary mechanisms for isolating dependencies in unit tests.
- **Test‑Driven Development**: Unit tests are the natural starting point for the Red‑Green‑Refactor cycle.
- **Dependency Injection**: Unit testing encourages designing classes with injectable collaborators, which are then replaced by mocks.

---

## Core Concepts

---

### 1. Unit Testing

#### Definitions

**Core Definition**

Unit testing is the practice of verifying the behaviour of the smallest testable unit of code—usually a single method or function—in complete isolation from external systems.

**Technical Definition**

A unit test instantiates the class under test (or calls the function under test), supplies controlled inputs, invokes the target behaviour, and asserts that the observed output or state matches the expected result. Collaborators are replaced with test doubles so that the test fails only if the unit itself is defective, not if a database, network, or framework component misbehaves.

**Beginner-Friendly Explanation**

A unit test is a focused experiment: “If I give this method these inputs, does it return exactly what I expect?” No database, no HTTP, no framework—just the method and its pure logic.

#### Purposes

- To verify the correctness of isolated logic with pinpoint precision.
- To provide lightning‑fast feedback during development (unit tests run in milliseconds).
- To catch regressions in utility classes and services before they propagate.
- To document the intended behaviour of a class or method through executable examples.

#### Syntax Rules and Structure

**Complete General Syntax (Pest)**

```php
<?php
// tests/Unit/DiscountCalculatorTest.php

use App\Services\DiscountCalculator;

test('applies ten percent discount', function () {
    $calculator = new DiscountCalculator();
    expect($calculator->apply(100, 10))->toBe(90.0);
});
```

**Complete General Syntax (PHPUnit)**

```php
<?php
// tests/Unit/DiscountCalculatorTest.php

namespace Tests\Unit;

use App\Services\DiscountCalculator;
use PHPUnit\Framework\TestCase;

class DiscountCalculatorTest extends TestCase
{
    public function test_applies_ten_percent_discount(): void
    {
        $calculator = new DiscountCalculator();
        $this->assertSame(90.0, $calculator->apply(100, 10));
    }
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `extends PHPUnit\Framework\TestCase` | Unit test base class (no Laravel boot). |
| `test()` / `it()` | Pest test declaration. |
| `new DiscountCalculator()` | Instantiates the unit under test directly. |
| `expect()->toBe()` / `assertSame()` | Assertion that the output matches expectation. |

**Constraints and Limitations**

- Unit tests **cannot** use facades, Eloquent, the database, or any Laravel helper that requires the container.
- Unit tests should test one behaviour per test; multiple behaviours in one test make failures ambiguous.
- Collaborators must be injected (constructor injection) or created internally; hidden global state makes units untestable.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Pure Function Unit Test**

**Code**

```php
<?php
// tests/Unit/StringHelperTest.php

use App\Support\StringHelper;

test('converts a string to title case', function () {
    expect(StringHelper::titleCase('hello world'))->toBe('Hello World');
});

test('handles empty strings', function () {
    expect(StringHelper::titleCase(''))->toBe('');
});
```

**Expected Output**

```
PASS  Tests\Unit\StringHelperTest
✓ it converts a string to title case
✓ it handles empty strings
```

**Why This Output Occurs**

`StringHelper::titleCase()` is a pure static method with no dependencies. The tests call it directly and assert the returned strings; no Laravel boot is required.

---

**Example 2: Class with Constructor‑Injected Dependency (Stubbed)**

**Code**

```php
<?php
// tests/Unit/PriceFormatterTest.php

use App\Contracts\CurrencyConverter;
use App\Services\PriceFormatter;

test('formats price using injected converter', function () {
    $converter = Mockery::mock(CurrencyConverter::class);
    $converter->shouldReceive('convert')
              ->with(100, 'USD', 'EUR')
              ->andReturn(85.0);

    $formatter = new PriceFormatter($converter);
    expect($formatter->format(100, 'USD', 'EUR'))->toBe('€85.00');
});
```

**Expected Output**

```
PASS  Tests\Unit\PriceFormatterTest
✓ it formats price using injected converter
```

**Why This Output Occurs**

The `CurrencyConverter` dependency is stubbed with `shouldReceive()->andReturn()`. The unit under test (`PriceFormatter`) receives the stub and produces the expected formatted string. The stub ensures the test is isolated from any real conversion logic.

---

#### Real-World Cases with Explanation

**Case 1: Tax Calculation in an E‑commerce System**

An e‑commerce platform extracts tax logic into a `TaxCalculator` service with no Eloquent dependencies. A unit test verifies that a 20 % VAT on £100 yields £20, and that zero‑rated items return £0. This test runs in under a millisecond and catches rounding errors instantly.

**Explanation**: Pure business logic with no external dependencies is the ideal candidate for unit testing because the test is deterministic and blazingly fast.

---

### 2. Testing Classes

#### Definitions

**Core Definition**

Testing a class means instantiating the class, exercising its public methods with controlled inputs, and asserting that the resulting output or state matches expectations, often while replacing its collaborators with test doubles.

**Technical Definition**

A class test typically follows the Arrange‑Act‑Assert pattern: create the object (and any required test doubles), call the method under test, and assert on the return value or observable state. For classes with dependencies, constructor injection (or setter injection) allows those dependencies to be replaced by mocks, stubs, or spies so that the class is tested in isolation.

**Beginner-Friendly Explanation**

Testing a class means creating an instance of it, calling its methods with known inputs, and checking that it does what it should—while swapping out its “helpers” for fake versions so you are only testing the class itself.

#### Purposes

- To verify the behaviour of a single class in isolation from its collaborators.
- To ensure that constructor dependencies are correctly used.
- To catch regressions when a class’s internal logic changes.
- To document the public contract of a class.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
<?php

use App\Services\OrderProcessor;

test('processes an order', function () {
    $gateway = Mockery::mock(PaymentGateway::class);
    $gateway->shouldReceive('charge')->once()->andReturn(true);

    $processor = new OrderProcessor($gateway);
    $result = $processor->process(['total' => 50]);

    expect($result)->toBeTrue();
});
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `new ClassName($dependency)` | Instantiates the class with a test double. |
| `$dependency->shouldReceive()` | Declares the expected method call on the double. |
| `$processor->process(...)` | Invokes the behaviour under test. |
| `expect()->toBe()` | Asserts the result. |

**Constraints and Limitations**

- Classes with hidden dependencies (e.g., `app()` calls inside methods) are harder to test in isolation.
- Static method calls within a class cannot be easily mocked without facades or refactoring.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Class with Multiple Dependencies**

**Code**

```php
<?php
// tests/Unit/InvoiceGeneratorTest.php

use App\Contracts\PdfRenderer;
use App\Contracts\Mailer;
use App\Services\InvoiceGenerator;

test('generates and emails an invoice', function () {
    $renderer = Mockery::mock(PdfRenderer::class);
    $renderer->shouldReceive('render')->once()->andReturn('%PDF-1.4');

    $mailer = Mockery::mock(Mailer::class);
    $mailer->shouldReceive('send')
           ->once()
           ->with('customer@example.com', '%PDF-1.4');

    $generator = new InvoiceGenerator($renderer, $mailer);
    $generator->generate('customer@example.com');
});
```

**Expected Output**

```
PASS  Tests\Unit\InvoiceGeneratorTest
✓ it generates and emails an invoice
```

**Why This Output Occurs**

Both dependencies are mocked with strict expectations (`once()`). The test passes only if `InvoiceGenerator` calls `render()` once and then passes the rendered content to `send()` exactly once. Any deviation causes an immediate failure.

---

#### Real-World Cases with Explanation

**Case 1: Payment Service with a Gateway**

A `PaymentService` class depends on a `PaymentGateway` interface. A unit test mocks the gateway to return `true` (success) and asserts that the service returns a successful charge result. Another test stubs the gateway to throw an exception and asserts that the service catches it and returns a failure response. Neither test touches a real payment provider.

**Explanation**: Mocking the gateway allows the service’s error‑handling logic to be tested deterministically.

---

### 3. Testing Methods

#### Definitions

**Core Definition**

Testing a method means invoking a specific public method on an object (or a static method on a class) with controlled arguments and asserting on its return value, side effects, or thrown exceptions.

**Technical Definition**

Method‑level tests are the most granular unit tests. They isolate a single method, replace any collaborators that method calls, and assert that the method produces the expected output or invokes the expected collaborators. Edge cases (null inputs, boundary values, exceptions) should each have their own dedicated method test.

**Beginner-Friendly Explanation**

Testing a method means calling one specific method with specific inputs and checking that it gives the right answer—nothing more, nothing less.

#### Purposes

- To verify the behaviour of individual methods, including edge cases.
- To ensure that methods correctly handle invalid inputs and exceptions.
- To document the method’s contract (inputs, outputs, side effects).

#### Syntax Rules and Structure

**Complete General Syntax**

```php
<?php

test('divides two numbers', function () {
    $calculator = new Calculator();
    expect($calculator->divide(10, 2))->toBe(5.0);
});

test('throws on division by zero', function () {
    $calculator = new Calculator();
    $calculator->divide(10, 0);
})->throws(DivisionByZeroError::class);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `$calculator->divide(10, 2)` | Invokes the method under test. |
| `expect()->toBe()` | Asserts the returned value. |
| `->throws()` | Asserts that the method throws the expected exception. |

**Constraints and Limitations**

- Private and protected methods cannot be tested directly; test them through public methods.
- Methods with side effects (e.g., writing to `$_SESSION`) are harder to isolate unless the side effect is injected.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Testing a Method with Edge Cases**

**Code**

```php
<?php
// tests/Unit/CalculatorTest.php

use App\Services\Calculator;

it('adds positive numbers', function () {
    expect((new Calculator)->add(2, 3))->toBe(5);
});

it('adds negative numbers', function () {
    expect((new Calculator)->add(-2, -3))->toBe(-5);
});

it('throws when dividing by zero', function () {
    (new Calculator)->divide(1, 0);
})->throws(DivisionByZeroError::class);
```

**Expected Output**

```
PASS  Tests\Unit\CalculatorTest
✓ it adds positive numbers
✓ it adds negative numbers
✓ it throws when dividing by zero
```

**Why This Output Occurs**

Each test isolates one behaviour of one method. The edge cases (negative numbers, division by zero) are explicitly covered, so the method’s contract is fully documented.

---

#### Real-World Cases with Explanation

**Case 1: Slug Generator Method**

A `Post::generateSlug(string $title)` method is tested with normal titles, titles containing special characters, duplicate titles, and empty titles. Each test targets the same method with a different input to verify the slugification rules.

**Explanation**: Method‑level tests exhaustively cover the input domain and document the slugification algorithm.

---

### 4. Testing Functions

#### Definitions

**Core Definition**

Testing a function means calling a standalone PHP function (or a static helper method) with controlled arguments and asserting on its return value, side effects, or thrown exceptions, without any object instantiation.

**Technical Definition**

Laravel applications often contain helper functions (global functions or static utility methods) that perform pure transformations. These can be tested by simply calling the function and asserting on the result. Because no object state is involved, function tests are the simplest and fastest form of unit test.

**Beginner-Friendly Explanation**

Testing a function is as simple as calling it and checking what it returns—no objects, no setup, just “give it an input, expect an output.”

#### Purposes

- To verify the correctness of utility and helper functions.
- To provide instant feedback on pure logic.
- To document the expected input/output contract of a function.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
<?php

test('formats a date', function () {
    expect(format_date('2026-01-01'))->toBe('1 January 2026');
});
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `format_date('2026-01-01')` | Calls the global function. |
| `expect()->toBe()` | Asserts the returned string. |

**Constraints and Limitations**

- Functions that rely on global state (e.g., `date_default_timezone_get()`) must have that state controlled in the test.
- Static methods on helper classes are tested in exactly the same way.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Testing a Global Helper**

**Code**

```php
<?php
// tests/Unit/HelpersTest.php

test('limits a string to a maximum length', function () {
    expect(str_limit('Hello World', 5))->toBe('Hello...');
});

test('returns the original string when under the limit', function () {
    expect(str_limit('Hi', 5))->toBe('Hi');
});
```

**Expected Output**

```
PASS  Tests\Unit\HelpersTest
✓ it limits a string to a maximum length
✓ it returns the original string when under the limit
```

**Why This Output Occurs**

The `str_limit` helper is a pure function. The tests call it directly and assert the returned strings; no framework boot or object instantiation is needed.

---

#### Real-World Cases with Explanation

**Case 1: Money Formatting Helper**

A `money(float $amount, string $currency)` helper formats amounts for display. Unit tests verify that `money(1234.5, 'USD')` returns `'$1,234.50'` and that `money(0, 'EUR')` returns `'€0.00'`.

**Explanation**: Helper functions are the lowest‑hanging fruit for unit testing and provide immediate confidence in formatting logic.

---

### 5. Mocking

#### Definitions

**Core Definition**

Mocking is the practice of replacing a real dependency with a test double that declares expectations (“this method must be called once with these arguments”) and fails the test if those expectations are not met.

**Technical Definition**

Laravel provides Mockery‑based helpers on the base test case: `$this->mock(Contract::class, $callback)` binds a mock into the container; `$this->partialMock()` mocks only selected methods while allowing the rest to execute normally; and `shouldReceive()` declares a method expectation. A mock is **pre‑programmed**: it requires you to state, before the act, which methods will be called, with what arguments, and how many times.

**Beginner-Friendly Explanation**

Mocking means creating a “stand‑in” for a dependency that has a checklist: “You must call `charge()` exactly once with these arguments.” If the code under test does not follow the checklist, the mock throws an error.

#### Purposes

- To verify that the unit under test interacts with its collaborators in the expected way.
- To replace dependencies that are slow, non‑deterministic, or have side effects.
- To enforce interaction contracts (method names, arguments, call counts).

#### Syntax Rules and Structure

**Complete General Syntax**

```php
$this->mock(Service::class, function ($mock) {
    $mock->shouldReceive('process')->once()->with($arg);
});

$this->partialMock(Service::class, function ($mock) {
    $mock->shouldReceive('process')->once();
});
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `$this->mock()` | Binds a full mock into the container. |
| `$this->partialMock()` | Mocks selected methods only. |
| `shouldReceive('method')` | Declares an expectation. |
| `->once()` | Expects exactly one call. |
| `->with($arg)` | Expects a specific argument. |

**Constraints and Limitations**

- Mocks require **all** called methods to be anticipated; unexpected calls fail the test.
- Mocking concrete classes with final methods is not possible without refactoring.
- Do not mock the `Request` facade; pass input into HTTP testing helpers instead.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Full Mock**

**Code**

```php
<?php

use App\Contracts\PaymentGateway;

test('charges the customer once', function () {
    $gateway = Mockery::mock(PaymentGateway::class);
    $gateway->shouldReceive('charge')
            ->once()
            ->with(100, 'usd')
            ->andReturn(true);

    $service = new PaymentService($gateway);
    $service->processPayment(100, 'usd');

    // Mockery verifies expectations automatically in tearDown.
});
```

**Expected Output**

```
PASS  Tests\Unit\PaymentServiceTest
✓ it charges the customer once
```

**Why This Output Occurs**

The mock declares that `charge(100, 'usd')` must be called exactly once. `PaymentService` does exactly that, so the mock’s expectation is satisfied.

---

**Example 2: Partial Mock**

**Code**

```php
<?php

use App\Services\ReportService;

test('skips expensive calculation in partial mock', function () {
    $mock = $this->partialMock(ReportService::class, function ($mock) {
        $mock->shouldReceive('calculateTotals')->once()->andReturn(0);
    });

    expect($mock->generate())->toBe('Report with 0 totals');
});
```

**Expected Output**

```
PASS  Tests\Unit\ReportServiceTest
✓ it skips expensive calculation in partial mock
```

**Why This Output Occurs**

`partialMock` replaces only `calculateTotals()` while allowing all other methods (e.g., `generate()`) to execute their real implementations.

---

#### Real-World Cases with Explanation

**Case 1: Notification Service with a Mailer Dependency**

A `NotificationService` depends on a `Mailer` contract. A unit test mocks the mailer with `shouldReceive('send')->once()->with($user, $message)`. If the service fails to send the email or sends it twice, the mock fails the test immediately.

**Explanation**: Mocking enforces the interaction contract between the service and its mailer.

---

### 6. Stubbing

#### Definitions

**Core Definition**

Stubbing is the practice of replacing a dependency with a test double that returns canned responses to method calls, without asserting on how or how many times those calls occur.

**Technical Definition**

A stub is a test double configured with `shouldReceive('method')->andReturn($value)`. Unlike a mock, a stub does **not** enforce call counts or argument expectations; it simply provides predictable return values so the unit under test can proceed. Stubs are appropriate when the test’s purpose is to verify the unit’s output, not its interactions.

**Beginner-Friendly Explanation**

Stubbing means creating a “fake” dependency that always gives a predetermined answer. You do not care how many times the code calls it—you just need it to return a known value so you can test what the unit does with that value.

#### Purposes

- To provide predictable inputs to the unit under test without asserting on interactions.
- To isolate the unit from dependencies that are hard to instantiate or slow to call.
- To test the unit’s output logic independently of its collaborators’ behaviour.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
$stub = Mockery::mock(Repository::class);
$stub->shouldReceive('find')->andReturn($fakeModel);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `shouldReceive('find')` | Declares the method that will be called. |
| `->andReturn($value)` | Provides the canned return value. |
| (No `once()` or `with()`) | No interaction expectations. |

**Constraints and Limitations**

- Stubs do not verify that the method was called; a typo in the method name may go unnoticed.
- Over‑stubbing can hide design flaws (e.g., a class that does too much).

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Stubbing a Repository**

**Code**

```php
<?php

use App\Contracts\UserRepository;
use App\Services\UserGreeter;

test('greets a user by name', function () {
    $repository = Mockery::mock(UserRepository::class);
    $repository->shouldReceive('find')
               ->with(1)
               ->andReturn((object) ['name' => 'Alice']);

    $greeter = new UserGreeter($repository);
    expect($greeter->greet(1))->toBe('Hello, Alice!');
});
```

**Expected Output**

```
PASS  Tests\Unit\UserGreeterTest
✓ it greets a user by name
```

**Why This Output Occurs**

The repository stub returns a fake user object when `find(1)` is called. `UserGreeter` uses that object to produce the greeting, and the test asserts on the greeting—not on how many times `find()` was called.

---

#### Real-World Cases with Explanation

**Case 1: Service That Reads Configuration**

A `TaxService` reads the tax rate from a `ConfigRepository` contract. A stub returns `0.2` for `getTaxRate()`. The test asserts that `TaxService::calculate(100)` returns `120`. The stub provides the input without asserting on the repository interaction.

**Explanation**: Stubbing is ideal when the dependency is merely a data source and the test’s focus is the unit’s output.

---

### 7. Spying

#### Definitions

**Core Definition**

Spying is the practice of wrapping a real object (or a test double) in a spy that records all method calls made to it, allowing assertions to be made **after** the code under test has executed.

**Technical Definition**

Laravel’s base test case provides `$this->spy(Service::class)`, which creates a Mockery spy. A spy records every method call (with arguments and return values) but does **not** declare expectations beforehand. After the act, you call `shouldHaveReceived('method')` to assert that a particular call occurred. This is the inverse of mocking: mocks verify **before** (via `shouldReceive`), spies verify **after** (via `shouldHaveReceived`).

**Beginner-Friendly Explanation**

A spy is like a security camera: it records everything that happens to a dependency. After the code runs, you check the footage to confirm that the right method was called with the right arguments.

#### Purposes

- To verify interactions without pre‑declaring expectations.
- To record calls on real objects for post‑hoc inspection.
- To test code that you cannot easily mock because you need the real implementation to run.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
$spy = $this->spy(Service::class);

// ... code under test executes ...

$spy->shouldHaveReceived('process')->with($arg);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `$this->spy()` | Creates a spy wrapper around the service. |
| `shouldHaveReceived('process')` | Asserts, after the fact, that the method was called. |
| `->with($arg)` | Narrows the assertion to calls with specific arguments. |

**Constraints and Limitations**

- Spies do not verify call counts unless you add `->once()` to the post‑hoc assertion.
- A spy on a concrete class still executes the real method unless the class is mocked internally.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Spying on a Service**

**Code**

```php
<?php

use App\Services\AuditLogger;
use App\Services\UserService;

test('logs user creation', function () {
    $spy = $this->spy(AuditLogger::class);

    $service = app(UserService::class);
    $service->create(['name' => 'Alice']);

    $spy->shouldHaveReceived('log')
        ->with('user.created', Mockery::on(fn ($data) => $data['name'] === 'Alice'));
});
```

**Expected Output**

```
PASS  Tests\Unit\UserServiceTest
✓ it logs user creation
```

**Why This Output Occurs**

The spy records the `log()` call made by `UserService`. After the service runs, `shouldHaveReceived()` asserts that the call occurred with the expected event name and payload.

---

#### Real-World Cases with Explanation

**Case 1: Event Dispatcher Verification**

A `RegistrationService` dispatches a `UserRegistered` event through an event dispatcher. A spy on the dispatcher records the `dispatch()` call. After registration, the test asserts `shouldHaveReceived('dispatch')->with(UserRegistered::class, $user)`.

**Explanation**: Spying is natural when the interaction is the side effect you want to verify, and you do not want to pre‑declare expectations.

---

### 8. Fakes

#### Definitions

**Core Definition**

Fakes are Laravel’s pre‑built test doubles for framework services—`Mail`, `Queue`, `Event`, `Bus`, `Notification`, `Storage`, and `Http`—that intercept side effects and provide assertion methods tailored to each service.

**Technical Definition**

Calling `Mail::fake()`, `Queue::fake()`, `Event::fake()`, `Bus::fake()`, `Notification::fake()`, or `Storage::fake()` swaps the real service implementation for a fake that records interactions in memory. Assertions are made **after** the code under test executes: `Mail::assertSent()`, `Queue::assertPushed()`, `Event::assertDispatched()`, `Bus::assertDispatched()`, `Notification::assertSentTo()`, and `Storage::assertExists()`. Fakes are distinct from mocks: a fake preserves the real code path (the code still “sends” mail, but the mail is captured), whereas a mock replaces the dependency entirely.

**Beginner-Friendly Explanation**

Fakes are “dummy versions” of Laravel’s built‑in services. When you call `Mail::fake()`, no real email is sent—but Laravel still records that your code tried to send one. Afterward, you can ask, “Was the welcome email sent to Alice?” and the fake answers.

#### Purposes

- To prevent real side effects (emails, jobs, events, file writes) during tests.
- To assert that the correct service interactions occurred.
- To keep tests fast by avoiding network and filesystem I/O.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
Mail::fake();
Queue::fake();
Event::fake();
Storage::fake('public');

// ... code under test ...

Mail::assertSent(WelcomeEmail::class, fn ($mail) => $mail->hasTo('alice@example.com'));
Queue::assertPushed(SendInvoice::class);
Event::assertDispatched(OrderShipped::class);
Storage::disk('public')->assertExists('avatars/alice.jpg');
```

**Component Breakdown**

| Fake | Assertion Methods |
|------|-------------------|
| `Mail::fake()` | `assertSent`, `assertNotSent`, `assertQueued` |
| `Queue::fake()` | `assertPushed`, `assertNotPushed`, `assertNothingPushed` |
| `Event::fake()` | `assertDispatched`, `assertNotDispatched` |
| `Bus::fake()` | `assertDispatched`, `assertDispatchedSync` |
| `Notification::fake()` | `assertSentTo`, `assertNotSentTo` |
| `Storage::fake()` | `assertExists`, `assertMissing` |
| `Http::fake()` | `assertSent`, `assertNotSent` |

**Constraints and Limitations**

- Create each fake **inside** the test method, not in `setUp()`, to prevent state leakage between tests.
- Pass specific class names to `Event::fake([EventA::class])` or `Queue::fake([JobA::class])` when you know which classes should be faked; a bare fake can hide unexpected dispatches.
- Assertions are made after the act, not before.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Faking Mail and Queue**

**Code**

```php
<?php

use App\Mail\WelcomeEmail;
use App\Jobs\SendInvoice;
use Illuminate\Support\Facades\Mail;
use Illuminate\Support\Facades\Queue;

test('registration sends welcome email and queues invoice', function () {
    Mail::fake();
    Queue::fake();

    $service = app(RegistrationService::class);
    $service->register(['email' => 'alice@example.com']);

    Mail::assertSent(WelcomeEmail::class, function ($mail) {
        return $mail->hasTo('alice@example.com');
    });

    Queue::assertPushed(SendInvoice::class);
});
```

**Expected Output**

```
PASS  Tests\Feature\RegistrationTest
✓ it registration sends welcome email and queues invoice
```

**Why This Output Occurs**

`Mail::fake()` and `Queue::fake()` prevent real emails and jobs. The assertions verify that the welcome email was addressed to Alice and that the invoice job was pushed.

---

**Example 2: Faking HTTP Client**

**Code**

```php
<?php

use Illuminate\Support\Facades\Http;

test('fetches user data from external API', function () {
    Http::fake([
        'api.example.com/users/1' => Http::response(['name' => 'Alice'], 200),
    ]);

    $service = new ExternalUserService();
    expect($service->fetch(1)['name'])->toBe('Alice');

    Http::assertSent(function ($request) {
        return $request->url() === 'https://api.example.com/users/1';
    });
});
```

**Expected Output**

```
PASS  Tests\Unit\ExternalUserServiceTest
✓ it fetches user data from external API
```

**Why This Output Occurs**

`Http::fake()` intercepts the outbound request and returns the canned JSON. The assertion verifies that the correct URL was requested.

---

#### Real-World Cases with Explanation

**Case 1: Order Placement Workflow**

An `OrderService` sends a confirmation email, dispatches a `OrderPlaced` event, and queues an invoice job. A unit test fakes `Mail`, `Event`, and `Queue`, then asserts that all three interactions occurred with the correct payloads. No real side effect ever occurs.

**Explanation**: Fakes allow a single test to verify multiple side effects without slowing down or polluting the environment.

---

### 9. Assertions

#### Definitions

**Core Definition**

Assertions are the statements that declare what the test expects to be true; if an assertion fails, the test fails and reports the discrepancy.

**Technical Definition**

Laravel unit tests use PHPUnit’s assertion library (`assertSame`, `assertTrue`, `assertNull`, `assertCount`, `expectException`) or Pest’s `expect()` API (`->toBe()`, `->toBeTrue()`, `->toThrow()`). Laravel adds domain‑specific assertions for HTTP responses (`assertStatus`, `assertJson`) and database state (`assertDatabaseHas`, `assertModelExists`), but these are primarily used in feature tests. In unit tests, the assertions focus on return values, object state, and thrown exceptions.

**Beginner-Friendly Explanation**

Assertions are how you say “this must be true.” If it is not true, the test stops and tells you exactly what went wrong.

#### Purposes

- To express the expected outcome of the unit under test.
- To fail the test loudly and precisely when behaviour deviates.
- To document the contract of the unit.

#### Syntax Rules and Structure

**Complete General Syntax (Pest)**

```php
expect($result)->toBe(42);
expect($result)->toBeTrue();
expect($result)->toBeInstanceOf(User::class);
expect($array)->toHaveCount(3);
```

**Complete General Syntax (PHPUnit)**

```php
$this->assertSame(42, $result);
$this->assertTrue($result);
$this->assertInstanceOf(User::class, $result);
$this->assertCount(3, $array);
```

**Component Breakdown**

| Assertion | Verifies |
|-----------|----------|
| `toBe()` / `assertSame()` | Strict identity (type and value). |
| `toEqual()` / `assertEquals()` | Loose equality. |
| `toBeTrue()` / `assertTrue()` | Boolean true. |
| `toBeInstanceOf()` / `assertInstanceOf()` | Object class. |
| `toHaveCount()` / `assertCount()` | Array or collection size. |
| `toThrow()` / `expectException()` | Exception of a given class. |

**Constraints and Limitations**

- Use strict assertions (`assertSame`, `toBe`) where possible; loose assertions can mask type mismatches.
- Do not assert on implementation details (e.g., private property values); assert on observable behaviour.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Value and Exception Assertions**

**Code**

```php
<?php

test('returns the correct total', function () {
    $cart = new Cart();
    $cart->addItem(['price' => 10]);
    $cart->addItem(['price' => 15]);

    expect($cart->total())->toBe(25.0);
});

test('throws on empty cart checkout', function () {
    (new Cart())->checkout();
})->throws(EmptyCartException::class);
```

**Expected Output**

```
PASS  Tests\Unit\CartTest
✓ it returns the correct total
✓ it throws on empty cart checkout
```

**Why This Output Occurs**

The first test asserts the exact numeric total. The second test asserts that the expected exception is thrown, documenting the method’s error contract.

---

#### Real-World Cases with Explanation

**Case 1: Collection Filtering**

A `PostCollection` class has a `published()` method that filters posts. A unit test asserts `expect($collection->published())->toHaveCount(2)` and `expect($collection->published()->first()->title)->toBe('Published Post')`. The assertions verify both the count and the content of the result.

**Explanation**: Combining multiple focused assertions provides a thorough verification of the method’s output.

---

### 10. Unit Testing Best Practices

#### Definitions

**Core Definition**

Unit testing best practices are the guidelines that keep unit tests fast, isolated, deterministic, readable, and genuinely useful for catching defects.

**Technical Definition**

Best practices encompass testing pure logic without framework dependencies, using strict assertions, isolating dependencies with appropriate test doubles (fakes for framework services, mocks for interaction contracts, stubs for return values, spies for post‑hoc verification), controlling time and randomness, and keeping each test focused on a single behaviour.

#### Key Best Practices

1. **Test Pure Logic in `tests/Unit`**: Reserve unit tests for code that does not need Laravel’s container, database, or HTTP stack. Use feature tests for anything that requires the framework.

2. **Prefer Framework Fakes Over Mocks for Framework Services**: For `Mail`, `Queue`, `Event`, `Bus`, `Notification`, `Storage`, and `Http`, use the built‑in fakes. They preserve the real code path and offer expressive assertion methods.

3. **Choose the Right Test Double**:
   - **Mock** when you need to verify interactions **before** the act (`shouldReceive`).
   - **Stub** when you only need a canned return value.
   - **Spy** when you need to verify interactions **after** the act (`shouldHaveReceived`).

4. **Create Fakes Inside the Test Method**: Do not create fakes in `setUp()`; doing so can leak state between tests and hide unexpected dispatches.

5. **Pass Class Names to `Event::fake()` and `Queue::fake()`**: Restricting the fake to known classes prevents unexpected dispatches from going unnoticed.

6. **Control Time and Randomness**: Use `$this->freezeTime()`, `$this->travelTo()`, and `Str::createRandomStringsUsing()` to make time‑ and randomness‑dependent tests deterministic.

7. **Use `Http::preventStrayRequests()`**: In tests that fake HTTP, prevent any request that lacks a matching fake from reaching the network.

8. **Test One Behaviour per Test**: A test that fails should point to exactly one defect. Avoid asserting multiple unrelated outcomes in the same test.

9. **Use Strict Assertions**: Prefer `toBe()` / `assertSame()` over loose equality to catch type mismatches.

10. **Do Not Mock the Query Builder or `Request` Facade**: Run real queries against the test database, and pass input through HTTP testing helpers instead of mocking `Request`.

11. **Do Not Mock What You Do Not Own**: Framework fakes are better than mocks for Laravel services; mocks are for your own container‑resolved contracts.

12. **Keep Unit Tests Fast**: If a “unit test” takes more than a few milliseconds, it is probably a feature test in disguise. Move it to `tests/Feature`.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Applying Best Practices**

**Code**

```php
<?php
// tests/Unit/OrderTotalTest.php

use App\Contracts\TaxCalculator;
use App\Services\OrderTotal;

test('calculates total with tax', function () {
    // Stub the tax dependency (return value only, no interaction assertion).
    $tax = Mockery::mock(TaxCalculator::class);
    $tax->shouldReceive('calculate')->with(100)->andReturn(20.0);

    $total = new OrderTotal($tax);
    expect($total->withTax(100))->toBe(120.0);
});

test('dispatches order event after total is calculated', function () {
    Event::fake([OrderCalculated::class]);

    $service = app(OrderService::class);
    $service->calculate(100);

    Event::assertDispatched(OrderCalculated::class);
});
```

**Expected Output**

```
PASS  Tests\Unit\OrderTotalTest
✓ it calculates total with tax
✓ it dispatches order event after total is calculated
```

**Why This Output Occurs**

The first test uses a **stub** because it only cares about the returned total, not whether `calculate()` was called a specific number of times. The second test uses `Event::fake()` (a **framework fake**) because it needs to assert that the event was dispatched, and it restricts the fake to `OrderCalculated::class` to avoid hiding unexpected dispatches.

---

#### Real-World Cases with Explanation

**Case 1: CI Pipeline with Fast Unit Tests**

A team enforces a rule: all pure business logic must have unit tests in `tests/Unit` that run without the framework. The CI pipeline runs these tests first (under 2 seconds) and then runs the slower feature tests. Developers get immediate feedback on logic errors before waiting for the full suite.

**Explanation**: Unit tests that follow best practices provide the fastest possible feedback loop, which is the cornerstone of a productive TDD workflow.

---

## References

- Fakes, Mocks, and Determinism — https://github.com/yaojingang/geoflow/blob/df77389bc3d7b83cba751c56b968a021b1a4e223/.claude/skills/testing-best-practices/rules/isolation.md
- Laravel Mocking Documentation (Laravel 8.x PDF excerpt) — https://github.com/DevStorm-Team/laravel-book/blob/f290014989ce7aee8657687973751b6685ad3f8a/laravel-docs-8.x.pdf
- Mocking — Laravel 7.x Documentation — https://laravel.com/docs/7.x/mocking#queue-fake
- How to Get Started with Unit Testing in Laravel — https://zencoder.ai/blog/laravel-unit-testing
- Laravel 13 Testing — Pest PHP 4 & PHPUnit 12 — https://raw.githubusercontent.com/turkeryildirim/ai-skills/refs/heads/main/laravel/references/testing.md
- Mockery Quick Reference — https://docs.mockery.io/en/latest/getting_started/quick_reference.html
- Laravel TDD Workflow — https://github.com/affaan-m/ecc/blob/846ffb75da9a5f4e677d927af1ad4a1951652267/skills/laravel-tdd/SKILL.md
- Laravel Fake Assertions Now Accept Property Arrays — https://laravel-news.com/laravel-13-35-fake-assertions-property-arrays
- Mockery: Spies vs Mocks — https://docs.mockery.io/en/latest/reference/spies.html

---

**Version Note**: This cheat sheet is based on Laravel 10.x through 13.x documentation. Pest 4 and PHPUnit 11/12 are the current testing frameworks in the Laravel 12–13 ecosystem. `Event::fake([Class::class])` class‑name filtering is available in all recent versions. `Http::preventStrayRequests()` was introduced in Laravel 9. Laravel 13.35 added property‑array assertions to `assertPushed`, `assertDispatched`, and `assertSentTo`. Developers should always consult the documentation for their specific Laravel version.