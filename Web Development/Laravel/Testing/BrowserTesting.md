# Laravel Browser Testing: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Laravel browser testing is the practice of using Laravel Dusk, an expressive browser automation and testing API, to drive a real browser (Google Chrome by default) through end-to-end user journeys—clicking buttons, filling forms, and asserting on rendered pages—to verify that the full application works exactly as a human user would experience it.

**Technical Definition**

Laravel Dusk is a first-party package (`laravel/dusk`) that wraps Facebook’s WebDriver protocol and provides a standalone ChromeDriver installation, eliminating the need for JDK or Selenium. Dusk tests reside in `tests/Browser`, extend `Tests\DuskTestCase`, and use a fluent `Browser` API to navigate pages, interact with DOM elements via CSS or Dusk selectors, wait for asynchronous JavaScript, and assert on page content, URLs, form values, cookies, and database state. Dusk supports multiple simultaneous browsers, page objects, browser macros, and database isolation via `DatabaseMigrations` (not `RefreshDatabase`, because transactions cannot span HTTP requests).

**Beginner-Friendly Explanation**

Browser testing means testing your app the way a real person uses it: opening a browser, visiting a URL, typing into fields, clicking buttons, and checking that the page shows what it should. Laravel Dusk is the tool that lets you write these “robot user” tests. Instead of just checking that an HTTP response has a 200 status code, Dusk actually launches Chrome, loads your page with all its JavaScript and CSS, and interacts with it like a real user would. If your login button does not work because of a JavaScript bug, a feature test might miss it—but a Dusk browser test will catch it.

---

### Key Characteristics

- **Real Browser Execution**: Dusk drives an actual Chrome browser (or any Selenium‑compatible browser) through WebDriver, so JavaScript, CSS, and client‑side rendering are fully exercised.
- **Standalone ChromeDriver**: No need to install JDK or Selenium; Dusk ships with a standalone ChromeDriver binary that it manages automatically.
- **Expressive, Fluent API**: The `Browser` class provides chainable methods (`visit`, `type`, `press`, `assertSee`, etc.) that read like natural language.
- **Dusk Selectors**: Instead of brittle CSS classes, Dusk recommends `dusk` HTML attributes accessed with the `@` prefix (e.g., `@login-button`), which remain stable across UI refactors.
- **Explicit Waiting**: Dusk provides `waitFor`, `waitForText`, `waitUntil`, `waitForLocation`, and many other wait methods to handle asynchronous JavaScript without arbitrary `sleep()` calls.
- **Page Objects and Components**: Encapsulate page‑specific selectors and actions into reusable classes for maintainability.
- **Database Migrations, Not Transactions**: Dusk tests must use `DatabaseMigrations` (or `DatabaseTruncation`) because `RefreshDatabase`’s transactions cannot span the separate HTTP request made by the browser.

---

### Prerequisites

- **PHP 8.1+** (Laravel 10+) or **PHP 8.2+** (Laravel 11+).
- **Google Chrome** installed on the machine running the tests.
- **Composer** for installing `laravel/dusk`.
- **A Laravel application** with a `.env` file (the `APP_URL` must be set correctly).
- **A test database** configured; use `DatabaseMigrations` or `DatabaseTruncation` for isolation.
- **Node.js and NPM** if the application uses Vite or compiled assets that the browser must load.

---

### Related Programming Areas

- **Feature Testing**: Dusk complements in‑process feature tests by covering JavaScript and real browser rendering.
- **Database Testing**: Dusk tests use the same factories and database assertions as feature tests, but with `DatabaseMigrations` isolation.
- **Authentication Testing**: Dusk’s `loginAs()` method bypasses the login screen for faster tests.
- **CI/CD**: Dusk tests can run headless in CI pipelines (GitHub Actions, GitLab CI, etc.).
- **Pest 4 Browser Testing**: Pest 4 now includes its own browser testing, which is recommended for new projects over Dusk.

---

## Core Concepts

---

### 1. Browser Testing

#### Definitions

**Core Definition**

Browser testing is the verification of application behaviour through a real web browser, exercising the complete user interface including HTML, CSS, and JavaScript.

**Technical Definition**

Browser tests automate a browser instance to navigate to pages, interact with DOM elements, and assert on the rendered page state. In the Laravel ecosystem, browser testing is implemented by Laravel Dusk (or Pest 4’s browser testing). Unlike HTTP feature tests, which simulate requests in‑process, browser tests launch an actual browser and communicate with it via the WebDriver protocol, so they catch issues that only manifest in a real rendering engine.

**Beginner-Friendly Explanation**

Browser testing is the “robot clicks through your app” kind of test. It checks that what users actually see and interact with works correctly, including anything powered by JavaScript.

#### Purposes

- To verify end‑to‑end user journeys (registration, checkout, login) in a real browser.
- To catch JavaScript‑dependent bugs that HTTP feature tests cannot detect.
- To test CSS‑driven behaviour (modals, dropdowns, animations) that depends on a browser engine.
- To validate the complete integration of frontend and backend.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
$this->browse(function (Browser $browser) {
    $browser->visit('/login')
            ->type('email', 'user@example.com')
            ->type('password', 'password')
            ->press('Login')
            ->assertPathIs('/dashboard');
});
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `$this->browse()` | Dusk entry point; receives one or more `Browser` instances. |
| `visit('/uri')` | Navigates to the URI. |
| `type()` / `press()` | Interacts with form elements. |
| `assertPathIs()` | Asserts the current URL path. |

**Constraints and Limitations**

- Browser tests are the slowest type of test and should be reserved for critical user journeys.
- They require a real browser and a running application server (Dusk handles this automatically).
- `RefreshDatabase` cannot be used; `DatabaseMigrations` or `DatabaseTruncation` is required.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Basic Browser Test**

**Code**

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

**Expected Output**

```
PASS  Tests\Browser\LoginTest
✓ user can login
```

**Why This Output Occurs**

Dusk launches Chrome, navigates to `/login`, fills in the email and password fields, clicks the “Login” button, and asserts that the browser is redirected to `/dashboard`.

---

**Example 2: JavaScript‑Dependent Interaction**

**Code**

```php
public function test_modal_opens_on_click(): void
{
    $this->browse(function (Browser $browser) {
        $browser->visit('/dashboard')
                ->click('@open-modal')
                ->waitFor('.modal')
                ->assertSee('Confirmation Required');
    });
}
```

**Expected Output**

```
PASS  Tests\Browser\ModalTest
✓ modal opens on click
```

**Why This Output Occurs**

The `click('@open-modal')` triggers JavaScript that displays a modal. `waitFor('.modal')` pauses until the modal appears in the DOM, and `assertSee()` confirms the expected text is rendered inside it.

---

#### Real-World Cases with Explanation

**Case 1: E‑commerce Checkout**

An e‑commerce application uses a Dusk test that adds a product to the cart, proceeds to checkout, fills in the shipping form, applies a coupon code, and completes payment. This test exercises Stripe’s JavaScript SDK and the application’s own Vue components—something no HTTP feature test could verify.

**Explanation**: Browser testing is indispensable for flows that depend on JavaScript and third‑party frontend integrations.

---

### 2. Laravel Dusk

#### Definitions

**Core Definition**

Laravel Dusk is Laravel’s official browser automation and testing tool, providing an expressive API for writing end‑to‑end tests that run against a real browser.

**Technical Definition**

Dusk (`laravel/dusk`) is a Composer package that integrates Facebook’s WebDriver with Laravel. It provides a `Browser` class with a fluent API, a set of test case base classes (`DuskTestCase`), Artisan commands (`dusk`, `dusk:install`, `dusk:make`, `dusk:page`, `dusk:chrome-driver`), and a standalone ChromeDriver binary. Dusk tests are stored in `tests/Browser` and are run with `php artisan dusk`.

**Beginner-Friendly Explanation**

Laravel Dusk is the package that makes browser testing in Laravel easy. It gives you a clean, chainable API so you can write tests like “visit this page, type this, click that, and check the result” without wrestling with low‑level browser automation code.

#### Purposes

- To provide a first‑party, Laravel‑integrated browser testing solution.
- To eliminate the complexity of setting up Selenium or JDK.
- To offer an expressive, fluent API that reads like plain English.
- To support advanced patterns like page objects, multiple browsers, and browser macros.

#### Syntax Rules and Structure

**Installation**

```bash
composer require laravel/dusk --dev
php artisan dusk:install
```

The `dusk:install` command creates the `tests/Browser` directory, generates an example test, and installs the ChromeDriver binary.

**Running Tests**

```bash
php artisan dusk
php artisan dusk --filter=LoginTest
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `tests/Browser` | Default directory for Dusk tests. |
| `tests/DuskTestCase.php` | Base test case for Dusk tests. |
| `php artisan dusk` | Runs all Dusk tests. |
| `php artisan dusk:make` | Generates a new Dusk test. |
| `php artisan dusk:page` | Generates a page object. |
| `php artisan dusk:chrome-driver` | Manages ChromeDriver versions. |

**Constraints and Limitations**

- Dusk is a development dependency (`--dev`) and must never be installed in production.
- Dusk tests are slower than feature tests; use them judiciously.
- Pest 4’s browser testing is now recommended for new projects.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Installing and Running Dusk**

**Commands**

```bash
composer require laravel/dusk --dev
php artisan dusk:install
php artisan dusk
```

**Expected Output (Terminal)**

```
INFO  Dusk installed successfully.
PHPUnit 10.x by Sebastian Bergmann and contributors.
...
PASS  Tests\Browser\ExampleTest
✓ basic example
```

**Why This Output Occurs**

`dusk:install` scaffolds the test directory and installs ChromeDriver. `php artisan dusk` runs all tests in `tests/Browser`, reporting results in PHPUnit/Pest format.

---

**Example 2: Generating a Dusk Test**

**Command**

```bash
php artisan dusk:make LoginTest
```

**Generated File**

```php
<?php
// tests/Browser/LoginTest.php

namespace Tests\Browser;

use Laravel\Dusk\Browser;
use Tests\DuskTestCase;

class LoginTest extends DuskTestCase
{
    public function test_example(): void
    {
        $this->browse(function (Browser $browser) {
            $browser->visit('/')
                    ->assertSee('Laravel');
        });
    }
}
```

**Expected Output**

```
INFO  Dusk test created successfully.
```

**Why This Output Occurs**

The `dusk:make` command generates a boilerplate Dusk test with the `browse()` closure and a basic visit/assert example.

---

#### Real-World Cases with Explanation

**Case 1: SaaS Onboarding Flow**

A SaaS application uses Dusk to test its multi‑step onboarding wizard: sign up, verify email (simulated), select a plan, and reach the dashboard. The Dusk test ensures that every step renders correctly and that JavaScript‑driven validation works as expected.

**Explanation**: Dusk is the right tool for critical, JavaScript‑heavy user journeys that must work flawlessly.

---

### 3. Dusk Installation

#### Definitions

**Core Definition**

Dusk installation is the process of adding the `laravel/dusk` package, running the `dusk:install` Artisan command, and configuring the `APP_URL` environment variable so Dusk can connect to the application.

**Technical Definition**

Installation involves: (1) requiring the package via Composer as a dev dependency; (2) running `php artisan dusk:install`, which publishes the `tests/Browser` directory, creates an example test, and installs the ChromeDriver binary; (3) setting `APP_URL` in `.env` to match the application’s accessible URL; and optionally (4) managing ChromeDriver versions with `dusk:chrome-driver`.

**Beginner-Friendly Explanation**

Installing Dusk is a three‑step process: install the package, run the install command, and tell Dusk where your app lives (`APP_URL`). That is it—Dusk handles the browser driver automatically.

#### Purposes

- To add Dusk as a development dependency without affecting production.
- To scaffold the test directory and example test.
- To ensure ChromeDriver is available and compatible with the installed Chrome version.
- To configure the application URL that Dusk will visit.

#### Syntax Rules and Structure

**Complete General Syntax**

```bash
composer require laravel/dusk --dev
php artisan dusk:install
# Set APP_URL in .env
php artisan dusk:chrome-driver --detect
```

**Component Breakdown**

| Step | Command / Action | Purpose |
|------|------------------|---------|
| 1 | `composer require laravel/dusk --dev` | Installs the package. |
| 2 | `php artisan dusk:install` | Scaffolds tests and ChromeDriver. |
| 3 | Set `APP_URL` in `.env` | Points Dusk at the application. |
| 4 | `dusk:chrome-driver --detect` | Matches ChromeDriver to Chrome. |

**Syntax Rules**

- Dusk must be installed with `--dev`.
- `dusk:install` must be run after Composer installation.
- `APP_URL` must include the protocol and port (e.g., `http://localhost:8000`).
- ChromeDriver binaries must be executable (`chmod -R 0755 vendor/laravel/dusk/bin/`).

**Constraints and Limitations**

- Dusk’s service provider must never be registered in production.
- If `APP_URL` is incorrect, Dusk tests will fail with connection errors.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Complete Installation Sequence**

**Commands**

```bash
composer require laravel/dusk --dev
php artisan dusk:install
php artisan dusk:chrome-driver --detect
```

**`.env` Addition**

```
APP_URL=http://localhost:8000
```

**Expected Output**

```
INFO  Dusk installed successfully.
INFO  ChromeDriver installed successfully.
```

**Why This Output Occurs**

The sequence installs the package, scaffolds the test infrastructure, and ensures the correct ChromeDriver version is available.

---

**Example 2: Verifying the Installation**

**Command**

```bash
php artisan dusk
```

**Expected Output**

```
PASS  Tests\Browser\ExampleTest
✓ basic example
```

**Why This Output Occurs**

The example test generated by `dusk:install` navigates to `/` and asserts it sees “Laravel.” A passing result confirms that Dusk, ChromeDriver, and the application server are all correctly configured.

---

#### Real-World Cases with Explanation

**Case 1: CI Pipeline Setup**

A team’s GitHub Actions workflow installs Dusk, runs `dusk:chrome-driver --detect`, and executes `php artisan dusk` in headless mode. The `--detect` flag ensures the ChromeDriver version matches the runner’s Chrome version, preventing “session not created” errors.

**Explanation**: Proper installation and ChromeDriver management are essential for reliable CI execution.

---

### 4. Dusk Configuration

#### Definitions

**Core Definition**

Dusk configuration is the set of settings that control how Dusk launches the browser, connects to the application, manages timeouts, and isolates the database.

**Technical Definition**

Configuration is spread across `.env` (`APP_URL`, `DUSK_DRIVER_URL`), `tests/DuskTestCase.php` (the `driver()` method and `prepare()` hook), and `phpunit.dusk.xml` (environment variables). The `driver()` method returns a `RemoteWebDriver` instance; the `prepare()` hook starts ChromeDriver. Dusk also supports headless mode, custom browser capabilities, and alternative browsers via a Selenium server.

**Beginner-Friendly Explanation**

Configuration is how you tell Dusk which browser to use, where your app is running, and how long to wait for things to load. Most of it is already set up for you, but you can customize it if needed.

#### Purposes

- To point Dusk at the correct application URL.
- To choose the browser and driver (Chrome by default, or any Selenium‑compatible browser).
- To control timeouts and waiting behaviour.
- To configure the database isolation strategy.
- To run tests in headless mode for CI environments.

#### Syntax Rules and Structure

**Complete General Syntax**

**`.env`**

```
APP_URL=http://localhost:8000
DUSK_DRIVER_URL=http://localhost:9515
```

**`tests/DuskTestCase.php` — Custom Driver**

```php
protected function driver(): RemoteWebDriver
{
    return RemoteWebDriver::create(
        'http://localhost:4444/wd/hub',
        DesiredCapabilities::chrome()
    );
}
```

**`tests/DuskTestCase.php` — Headless Mode**

```php
protected function driver(): RemoteWebDriver
{
    $options = (new ChromeOptions)->addArguments([
        '--headless',
        '--disable-gpu',
    ]);

    return RemoteWebDriver::create(
        'http://localhost:9515',
        DesiredCapabilities::chrome()->setCapability(
            ChromeOptions::CAPABILITY, $options
        )
    );
}
```

**Database Isolation**

```php
use Illuminate\Foundation\Testing\DatabaseMigrations;

class LoginTest extends DuskTestCase
{
    use DatabaseMigrations;
}
```

**Component Breakdown**

| Configuration | Location | Purpose |
|---------------|----------|---------|
| `APP_URL` | `.env` | Application URL Dusk visits. |
| `DUSK_DRIVER_URL` | `.env` | ChromeDriver address (default `localhost:9515`). |
| `driver()` | `DuskTestCase.php` | Creates the WebDriver instance. |
| `prepare()` | `DuskTestCase.php` | Starts ChromeDriver. |
| `--headless` | `ChromeOptions` | Runs Chrome without a visible window. |
| `DatabaseMigrations` | Test class | Isolates database state per test. |

**Syntax Rules**

- `APP_URL` must include the protocol and port and must not have a trailing slash.
- `DUSK_DRIVER_URL` defaults to `localhost:9515` but can be changed if the port is occupied.
- Headless mode is strongly recommended for CI.
- `DatabaseMigrations` must be used instead of `RefreshDatabase` because transactions cannot span HTTP requests.

**Constraints and Limitations**

- Custom browsers require a running Selenium server; the `driver()` method must be modified accordingly.
- Changing the driver without updating `DUSK_DRIVER_URL` can cause connection failures.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Headless Configuration for CI**

**Code**

```php
// tests/DuskTestCase.php

use Facebook\WebDriver\Chrome\ChromeOptions;
use Facebook\WebDriver\Remote\DesiredCapabilities;
use Facebook\WebDriver\Remote\RemoteWebDriver;

protected function driver(): RemoteWebDriver
{
    $options = (new ChromeOptions)->addArguments([
        '--headless',
        '--disable-gpu',
        '--no-sandbox',
    ]);

    return RemoteWebDriver::create(
        env('DUSK_DRIVER_URL', 'http://localhost:9515'),
        DesiredCapabilities::chrome()->setCapability(
            ChromeOptions::CAPABILITY, $options
        )
    );
}
```

**Expected Output**

Dusk tests run without opening a visible Chrome window, which is required in CI environments that lack a display server.

**Why This Output Occurs**

The `--headless` argument tells Chrome to run without a GUI. `--no-sandbox` is often required in Docker/CI environments.

---

**Example 2: Database Isolation with `DatabaseMigrations`**

**Code**

```php
<?php
// tests/Browser/UserRegistrationTest.php

namespace Tests\Browser;

use Illuminate\Foundation\Testing\DatabaseMigrations;
use Laravel\Dusk\Browser;
use Tests\DuskTestCase;

class UserRegistrationTest extends DuskTestCase
{
    use DatabaseMigrations;

    public function test_user_can_register(): void
    {
        $this->browse(function (Browser $browser) {
            $browser->visit('/register')
                    ->type('name', 'Alice')
                    ->type('email', 'alice@example.com')
                    ->type('password', 'password')
                    ->type('password_confirmation', 'password')
                    ->press('Register')
                    ->assertPathIs('/home');
        });

        $this->assertDatabaseHas('users', [
            'email' => 'alice@example.com',
        ]);
    }
}
```

**Expected Output**

```
PASS  Tests\Browser\UserRegistrationTest
✓ user can register
```

**Why This Output Occurs**

`DatabaseMigrations` runs the migrations before the test, giving a clean schema. The test completes registration in the browser and then asserts the database record exists.

---

#### Real-World Cases with Explanation

**Case 1: Docker‑Based CI**

A team runs Dusk tests in a Docker container. They configure `DUSK_DRIVER_URL` to point to a ChromeDriver service running on a different host and add `--no-sandbox` to the ChromeOptions. This allows Dusk to run reliably in the containerized CI environment.

**Explanation**: Dusk’s configuration is flexible enough to support complex deployment environments like Docker and Kubernetes.

---

### 5. Dusk Browser Automation

#### Definitions

**Core Definition**

Dusk browser automation is the set of `Browser` methods that navigate pages, interact with form elements, click links and buttons, scroll, and manipulate the DOM in a real browser.

**Technical Definition**

The `Laravel\Dusk\Browser` class provides a fluent API for automation: `visit()` to navigate, `type()` / `append()` / `clear()` for text inputs, `select()` for dropdowns, `check()` / `radio()` for checkboxes and radio buttons, `attach()` for file uploads, `click()` / `clickLink()` / `press()` for clickable elements, `scrollTo()` / `scrollIntoView()` for scrolling, and `with()` / `elsewhere()` for scoped interactions. Dusk selectors (`@selector`) are recommended over CSS selectors for stability.

**Beginner-Friendly Explanation**

Browser automation is the “doing” part of Dusk: visiting URLs, typing text, selecting options, clicking buttons, and uploading files. It is how you make the robot browser actually use your app.

#### Purposes

- To navigate to pages and interact with every type of form element.
- To simulate complex user interactions (multi‑select, file uploads, checkboxes).
- To scope interactions to specific elements (e.g., modals, iframes).
- To use stable Dusk selectors instead of brittle CSS classes.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
$browser->visit('/products')
        ->type('search', 'laptop')
        ->select('category', 'Electronics')
        ->check('in_stock')
        ->attach('image', __DIR__.'/laptop.jpg')
        ->click('@search-button')
        ->assertSee('Laptop');
```

**Component Breakdown**

| Method | Action |
|--------|--------|
| `visit($uri)` | Navigate to a URI. |
| `type($field, $value)` | Type into an input. |
| `append($field, $value)` | Append to existing input text. |
| `clear($field)` | Clear an input. |
| `select($field, $value)` | Select a dropdown option. |
| `check($field)` / `uncheck($field)` | Check or uncheck a checkbox. |
| `radio($field, $value)` | Select a radio button. |
| `attach($field, $path)` | Upload a file. |
| `click($selector)` | Click an element. |
| `press($buttonText)` | Click a button by its text. |
| `clickLink($linkText)` | Click a link by its text. |
| `with($selector, $callback)` | Scope interactions to an element. |

**Syntax Rules**

- Dusk selectors use the `@` prefix and correspond to `dusk` HTML attributes.
- CSS selectors are also supported but are less stable.
- Scoped interactions (`with`) receive a `Browser` instance for the scoped element.

**Constraints and Limitations**

- Interactions on elements that are not yet visible may fail; use `waitFor` before interacting with dynamically loaded content.
- File uploads require the file to exist on the machine running the tests.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Form Interaction**

**Code**

```php
$browser->visit('/register')
        ->type('name', 'Alice')
        ->type('email', 'alice@example.com')
        ->type('password', 'secret123')
        ->type('password_confirmation', 'secret123')
        ->check('terms')
        ->press('Register')
        ->assertPathIs('/home');
```

**Expected Output**

The browser fills in the registration form, checks the terms checkbox, submits the form, and lands on `/home`.

**Why This Output Occurs**

Each method interacts with the corresponding form element. `press('Register')` clicks the button with the text “Register,” and `assertPathIs('/home')` confirms the redirect.

---

**Example 2: Scoped Interaction with a Modal**

**Code**

```php
$browser->visit('/dashboard')
        ->click('@delete-account')
        ->waitFor('.modal')
        ->with('.modal', function (Browser $modal) {
            $modal->assertSee('Are you sure?')
                  ->press('Confirm');
        })
        ->assertSee('Account deleted');
```

**Expected Output**

The modal opens, the test asserts on its content, clicks the confirm button inside the modal, and verifies the success message.

**Why This Output Occurs**

`with('.modal', ...)` scopes the interactions to the modal, ensuring that the correct “Confirm” button is pressed even if other buttons exist on the page.

---

#### Real-World Cases with Explanation

**Case 1: Admin Panel Bulk Actions**

An admin panel has a table with checkboxes for bulk deletion. A Dusk test checks multiple rows, selects “Delete” from a dropdown, and confirms the modal. Automation methods like `check()`, `select()`, and `with()` make this complex interaction straightforward to test.

**Explanation**: Dusk’s automation API handles complex UI patterns like bulk actions and modals with readable, maintainable code.

---

### 6. Dusk Assertions

#### Definitions

**Core Definition**

Dusk assertions are methods that verify the state of the browser and the rendered page—URL, title, visible text, form values, cookies, JavaScript expressions, and more.

**Technical Definition**

The `Browser` class provides dozens of assertion methods, including `assertTitle`, `assertPathIs`, `assertSee`, `assertDontSee`, `assertSeeIn`, `assertInputValue`, `assertChecked`, `assertRadioSelected`, `assertSelectHasOption`, `assertScript`, `assertHasCookie`, `assertHasPlainCookie`, `assertSourceHas`, `assertSeeLink`, and many others. Dusk also supports Vue‑specific assertions (`assertVue`) and JavaScript expression assertions (`assertScript`).

**Beginner-Friendly Explanation**

Assertions are how you check that the browser is showing the right thing. “Does the page contain this text? Is this checkbox checked? Is the URL correct?” If the assertion fails, the test fails and tells you what went wrong.

#### Purposes

- To verify page navigation and URL correctness.
- To confirm that expected text is visible (or absent).
- To check form input values and selection states.
- To assert on cookies, JavaScript variables, and page source.
- To validate Vue component state (if applicable).

#### Syntax Rules and Structure

**Complete General Syntax**

```php
$browser->assertPathIs('/dashboard');
$browser->assertSee('Welcome');
$browser->assertDontSee('Error');
$browser->assertInputValue('email', 'user@example.com');
$browser->assertChecked('terms');
$browser->assertSelectHasOption('size', 'Large');
$browser->assertScript('window.isLoaded');
$browser->assertHasCookie('session_id');
```

**Component Breakdown**

| Assertion | Verifies |
|-----------|----------|
| `assertPathIs($path)` | Current URL path. |
| `assertSee($text)` | Text is visible on the page. |
| `assertDontSee($text)` | Text is not visible. |
| `assertSeeIn($selector, $text)` | Text within a specific element. |
| `assertInputValue($field, $value)` | Input field value. |
| `assertChecked($field)` | Checkbox is checked. |
| `assertSelectHasOption($field, $value)` | Dropdown has an option. |
| `assertScript($expression, $value)` | JavaScript expression result. |
| `assertHasCookie($name)` | Cookie is present. |
| `assertSourceHas($code)` | Source code is present. |

**Syntax Rules**

- Assertions are chainable on the `$browser` instance.
- Text assertions check the rendered text, not the raw HTML source.
- `assertScript` evaluates a JavaScript expression in the browser context.
- `assertSeeIn` scopes the text search to a specific selector.

**Constraints and Limitations**

- `assertSee` checks visible text; hidden elements are ignored.
- Some assertions (e.g., `assertScript`) require the page’s JavaScript to have finished executing.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: URL and Text Assertions**

**Code**

```php
$browser->visit('/login')
        ->type('email', 'user@example.com')
        ->type('password', 'password')
        ->press('Login')
        ->assertPathIs('/dashboard')
        ->assertSee('Welcome, Alice')
        ->assertDontSee('Invalid credentials');
```

**Expected Output**

The test passes if the login succeeds, the URL becomes `/dashboard`, and the welcome message appears without any error text.

**Why This Output Occurs**

`assertPathIs` confirms the redirect, `assertSee` confirms the success message, and `assertDontSee` confirms that no error message is shown.

---

**Example 2: JavaScript Expression Assertion**

**Code**

```php
$browser->visit('/chart')
        ->waitUntil('App.chartData.length > 0')
        ->assertScript('App.chartData.length', 3);
```

**Expected Output**

The test waits until the JavaScript variable `App.chartData` has entries, then asserts that it contains exactly 3 items.

**Why This Output Occurs**

`waitUntil` pauses until the condition is true, and `assertScript` evaluates the expression and compares the result to the expected value.

---

#### Real-World Cases with Explanation

**Case 1: Form Validation Feedback**

A registration form displays inline validation errors. A Dusk test submits an empty form and asserts `assertSee('The email field is required')`. It then fills in a valid email and asserts that the error message disappears (`assertDontSee`).

**Explanation**: Assertions on visible text are the most direct way to verify that user‑facing feedback is correct.

---

### 7. Dusk Best Practices

#### Definitions

**Core Definition**

Dusk best practices are the guidelines that keep browser tests fast, stable, maintainable, and genuinely useful for catching real user‑facing bugs.

**Technical Definition**

Best practices encompass: using Dusk selectors (`dusk` attributes) instead of brittle CSS classes; always waiting explicitly with `waitFor`/`waitUntil` rather than arbitrary `pause()` calls; using page objects and components to encapsulate page‑specific logic; using `loginAs()` to bypass the login screen; using `DatabaseMigrations` (not `RefreshDatabase`) for database isolation; running tests in headless mode in CI; and reserving browser tests for critical user journeys.

**Beginner-Friendly Explanation**

Best practices are the habits that make your Dusk tests reliable. They prevent flaky tests, keep tests fast, and make them easy to understand and maintain.

#### Key Best Practices

1. **Use Dusk Selectors (`dusk` Attributes)**: Add `dusk="login-button"` to HTML elements and reference them as `@login-button`. These selectors are stable across CSS refactors and UI changes.

2. **Always Wait Explicitly**: Use `waitFor`, `waitForText`, `waitUntil`, `waitForLocation`, or `waitForReload` instead of `pause()` or `sleep()`. Arbitrary pauses make tests slow and flaky.

3. **Use Page Objects for Complex Pages**: Generate page objects with `php artisan dusk:page` and define selectors and actions in reusable methods. This keeps tests readable and maintainable.

4. **Use `loginAs()` for Authentication**: Bypass the login screen by calling `$browser->loginAs($user)`. This saves time and avoids repeating the login flow in every test.

5. **Isolate the Database with `DatabaseMigrations`**: Do not use `RefreshDatabase`; its transactions cannot span HTTP requests. `DatabaseMigrations` (or `DatabaseTruncation`) ensures a clean database for each Dusk test.

6. **Run in Headless Mode in CI**: Add `--headless` and `--no-sandbox` to ChromeOptions for CI environments.

7. **Use Browser Macros for Reusable Actions**: Register macros on the `Browser` class for common interactions (e.g., `scrollToElement`) and reuse them across tests.

8. **Test Multiple Browsers When Necessary**: Use the `browse()` method with multiple `Browser` arguments to simulate multi‑user interactions (e.g., chat, collaborative editing).

9. **Reserve Dusk for Critical Journeys**: Browser tests are slow; use them for login, registration, checkout, and other high‑value flows. Use feature tests for everything else.

10. **Use Descriptive Test Names**: Name tests after the user journey, e.g., `test_user_can_complete_checkout`.

11. **Keep Tests Independent**: Each test should start from a clean state and not depend on data created by another test.

12. **Screenshot on Failure**: Dusk automatically captures screenshots on failure in `tests/Browser/screenshots`, which are invaluable for debugging CI failures.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Page Object Pattern**

**Page Object**

```php
<?php
// tests/Browser/Pages/Login.php

namespace Tests\Browser\Pages;

use Laravel\Dusk\Browser;
use Laravel\Dusk\Page;

class Login extends Page
{
    public function url(): string
    {
        return '/login';
    }

    public function elements(): array
    {
        return [
            '@email'    => 'input[name=email]',
            '@password' => 'input[name=password]',
            '@submit'   => 'button[type=submit]',
        ];
    }

    public function login(Browser $browser, string $email, string $password): void
    {
        $browser->type('@email', $email)
                ->type('@password', $password)
                ->press('@submit');
    }
}
```

**Usage**

```php
$browser->visit(new Login)
        ->login('user@example.com', 'password')
        ->assertPathIs('/dashboard');
```

**Expected Output**

The test uses the page object’s `login()` method to perform the login, then asserts the redirect. The page object encapsulates the selectors and the login logic.

**Why This Output Occurs**

The page object defines the URL, element shortcuts, and reusable actions. The test becomes a high‑level description of the journey rather than a series of low‑level interactions.

---

**Example 2: Explicit Waiting with `waitFor`**

**Code**

```php
$browser->visit('/dashboard')
        ->click('@load-data')
        ->waitForText('Data loaded', 10)
        ->assertSee('Data loaded');
```

**Expected Output**

The test clicks a button, waits up to 10 seconds for the text “Data loaded” to appear, and then asserts that it is visible.

**Why This Output Occurs**

`waitForText` polls the DOM until the text appears or the timeout expires, eliminating the need for arbitrary `sleep()` calls and reducing flakiness.

---

**Example 3: Authentication with `loginAs()`**

**Code**

```php
$user = User::factory()->create();

$this->browse(function (Browser $browser) use ($user) {
    $browser->loginAs($user)
            ->visit('/settings')
            ->assertSee('Account Settings');
});
```

**Expected Output**

The test authenticates as the user without visiting the login page, then navigates directly to `/settings`.

**Why This Output Occurs**

`loginAs($user)` sets the authenticated user for the browser session, bypassing the login flow entirely.

---

#### Real-World Cases with Explanation

**Case 1: CI Pipeline with Stable Dusk Tests**

A team’s Dusk suite was flaky due to `pause()` calls and brittle CSS selectors. They refactored to use Dusk selectors (`@login-button`), explicit `waitFor` calls, and page objects. The suite now runs reliably in headless mode on GitHub Actions and catches real UI regressions without false failures.

**Explanation**: Following Dusk best practices transforms a flaky, slow suite into a reliable safety net for critical user journeys.

---

## References

- Laravel Dusk — Laravel 13.x Documentation — https://laravel.com/framework/docs/13.x/dusk
- Laravel Dusk — Laravel 10.x Documentation (raw Markdown) — https://raw.githubusercontent.com/tomhrtly/docs/refs/heads/10.x/dusk.md
- Laravel Dusk — Laravel 7.x Documentation — https://laravel.com/docs/7.x/dusk
- Laravel Dusk — Laravel 5.5 Documentation — https://laravel.com/docs/5.5/dusk
- Laravel Dusk Skill — SkillsForge Marketplace — https://github.com/rawveg/skillsforge-marketplace/blob/9561ce20885bada1d6012d24d9b598d69c097ef2/laravel-dusk/SKILL.md
- Laravel Dusk Testing Skill — GitHub — https://github.com/Fusengine/claude-plugins/blob/main/plugins/laravel-expert/skills/laravel-dusk/SKILL.md
- Laravel Dusk + Preview Environments — Bunnyshell — https://www.bunnyshell.com/blog/laravel-dusk-preview-environments
- Dusk waitFor Timeout Best Practices — GitHub — https://github.com/laravel/dusk
- Dusk Database Assertions — QASkills.sh — https://qaskills.sh/blog/laravel-dusk-complete-guide
- Laravel Dusk Testing — LobeHub Skills — https://lobehub.com/skills/laravel-dusk

---

**Version Note**: This cheat sheet is based on Laravel 10.x through 13.x documentation. Laravel Dusk remains stable across these versions, but Pest 4 now includes its own browser testing, which the official documentation recommends for new projects. Dusk uses `DatabaseMigrations` (not `RefreshDatabase`) for database isolation because transactions cannot span HTTP requests. ChromeDriver management commands (`dusk:chrome-driver --detect`) are essential for CI environments. Developers should always consult the documentation for their specific Laravel version.