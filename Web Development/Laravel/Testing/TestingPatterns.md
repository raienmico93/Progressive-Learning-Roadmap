# Laravel Testing Patterns: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Testing patterns are the recurring structures, conventions, and strategies that developers apply when writing automated tests—shaping how tests are organized, which scenarios are covered, and how side effects (jobs, events, mail, notifications) are verified—so that the resulting suite is readable, comprehensive, and maintainable.

**Technical Definition**

Testing patterns in Laravel encompass two dimensions: **structural patterns** (Arrange–Act–Assert and Given–When–Then) that govern the internal layout of a test, and **coverage patterns** (happy path, edge case, error case, authentication, authorization, validation, API, jobs, events, notifications, mail) that determine which scenarios are exercised. Laravel's testing helpers—`actingAs()`, `assertForbidden()`, `assertSessionHasErrors()`, `Queue::fake()`, `Event::fake()`, `Mail::fake()`, `Notification::fake()`—provide the assertions required to implement each coverage pattern cleanly and deterministically.

**Beginner-Friendly Explanation**

Testing patterns are the “recipes” for writing good tests. Some patterns tell you how to lay out a test (like Arrange–Act–Assert: set things up, do the thing, check the result). Others tell you which scenarios to cover (the happy path where everything works, edge cases where inputs are unusual, error cases where things go wrong, and specific checks for authentication, authorization, validation, and side effects like jobs and emails). Following these patterns makes your tests predictable, readable, and thorough—so you rarely miss a bug.

---

### Key Characteristics

- **Structural Consistency**: AAA and GWT give every test the same shape, making failures easy to localize and reviews fast.
- **Coverage Matrix**: A complete suite tests each endpoint or feature across happy paths, edge cases, error cases, and the full authentication/authorization matrix—not just the success case.
- **Side-Effect Verification**: Laravel's fakes (`Queue::fake()`, `Event::fake()`, `Mail::fake()`, `Notification::fake()`) let tests assert that a job was pushed, an event was dispatched, an email was queued, or a notification was sent—without causing real side effects.
- **Failure-First Mindset**: The most valuable tests are often the negative ones—unauthorized users, invalid input, missing resources—because they catch security and correctness bugs.
- **Deterministic by Design**: Patterned tests use factories, fakes, and controlled time (`freezeTime()`) to eliminate randomness and flakiness.
- **Composable**: Each pattern can be combined—an API test for a validation failure is simultaneously a coverage pattern (validation), an error case, and an API test.

---

### Prerequisites

- **PHP 8.1+** (Laravel 10+) or **PHP 8.2+** (Laravel 11+).
- **A Laravel application** with routes, controllers, policies, jobs, events, mailables, and notifications defined.
- **Pest or PHPUnit** installed (Pest recommended for new projects).
- **A test database** configured (SQLite in-memory recommended).
- **Model factories** for all models appearing in tests.
- **Basic familiarity with Laravel's testing helpers** (`actingAs()`, `RefreshDatabase`, fakes, assertions).

---

### Related Programming Areas

- **Feature Testing**: The primary context for coverage patterns (API, auth, validation).
- **Unit Testing**: The context for testing isolated logic and using structural patterns.
- **Database Testing**: Factories and database assertions support every pattern.
- **Queue, Events, Mail, Notifications**: Fakes are the mechanism for testing side effects.
- **TDD**: Patterns are the natural language of the Red–Green–Refactor cycle.
- **CI/CD**: A patterned suite is deterministic and fast enough to run on every push.

---

## Core Concepts

---

### 1. Arrange–Act–Assert (AAA)

#### Definitions

**Core Definition**

Arrange–Act–Assert is a structural testing pattern that divides every test into three clearly delineated phases: arrange the preconditions, act by invoking the behaviour under test, and assert that the observed outcome matches expectations.

**Technical Definition**

The AAA pattern (also called AAA or 3A) is a convention for organizing a test's body. The **Arrange** phase sets up the SUT (system under test) and its collaborators, including database records, mock expectations, and authenticated users. The **Act** phase performs exactly one action—usually a single HTTP request, method call, or command dispatch. The **Assert** phase verifies the response, database state, dispatched jobs/events, and any other observable outcomes. A single-act rule is the essence of the pattern: one action per test makes failures unambiguous.

**Beginner-Friendly Explanation**

Every good test follows three steps: set things up (Arrange), do the thing you're testing (Act), and check the result (Assert). Keeping these phases clearly separated makes tests easy to read and failures easy to debug.

#### Purposes

- To make the intent and mechanics of a test immediately obvious to any reader.
- To enforce the one-action-per-test rule, eliminating ambiguous multi-behaviour tests.
- To make failures easy to localize (if the Act fails, the setup was correct; if the Assert fails, the Act is wrong).
- To standardize test layout across a codebase for consistent code review.

#### Syntax Rules and Structure

**Complete General Syntax (Pest)**

```php
test('authenticated user can create a post', function () {
    // Arrange
    $user = User::factory()->create();

    // Act
    $response = $this->actingAs($user)->post('/posts', [
        'title' => 'My Post',
        'body'  => 'Content',
    ]);

    // Assert
    $response->assertRedirect('/posts');
    $this->assertDatabaseHas('posts', ['title' => 'My Post']);
});
```

**Complete General Syntax (PHPUnit)**

```php
public function test_authenticated_user_can_create_a_post(): void
{
    // Arrange
    $user = User::factory()->create();

    // Act
    $response = $this->actingAs($user)->post('/posts', [
        'title' => 'My Post',
        'body'  => 'Content',
    ]);

    // Assert
    $response->assertRedirect('/posts');
    $this->assertDatabaseHas('posts', ['title' => 'My Post']);
}
```

**Component Breakdown**

| Phase | Content |
|-------|---------|
| Arrange | Factories, mocks, authenticated users, seeders. |
| Act | Exactly one HTTP call or method invocation. |
| Assert | Status codes, redirects, JSON, DB state, dispatched side effects. |

**Syntax Rules**

- The Act phase must contain exactly **one** action.
- Arrange must be complete before Act begins.
- Assertions must be specific (exact status codes, exact values).
- Comments (`// Arrange`, `// Act`, `// Assert`) are optional but strongly recommended for clarity.

**Constraints and Limitations**

- AAA does not prescribe which assertions to make; coverage patterns (below) supplement it.
- Tests with complex setup may need helper methods; keep the Arrange phase readable.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: AAA in a Feature Test**

```php
test('a guest cannot access the dashboard', function () {
    // Arrange: no authentication
    // Act: request the protected route
    $response = $this->get('/dashboard');

    // Assert: redirected to login
    $response->assertRedirect('/login');
});
```

**Expected Output**

```
PASS  Tests\Feature\DashboardTest
✓ a guest cannot access the dashboard
```

**Why This Output Occurs**

Arrange is empty because no setup is needed. Act sends a single GET request. Assert verifies the redirect caused by the `auth` middleware.

---

**Example 2: AAA in a Unit Test**

```php
test('discount calculator applies percentage', function () {
    // Arrange
    $calculator = new DiscountCalculator();

    // Act
    $result = $calculator->apply(100, 15);

    // Assert
    expect($result)->toBe(85.0);
});
```

**Expected Output**

```
PASS  Tests\Unit\DiscountCalculatorTest
✓ discount calculator applies percentage
```

**Why This Output Occurs**

Arrange creates the calculator. Act calls the method once. Assert verifies the exact returned value.

---

#### Real-World Cases with Explanation

**Case 1: Consistent Code Reviews**

A team enforces AAA across every test. During code review, reviewers instantly spot tests that violate the pattern—usually by making two HTTP calls or by asserting before the Act. The convention raises the overall quality of the suite without subjective debate.

**Explanation**: AAA is as much a code-review tool as a testing tool.

---

### 2. Given–When–Then (GWT)

#### Definitions

**Core Definition**

Given–When–Then is a behaviour-driven development (BDD) pattern that describes a test scenario as a sequence of preconditions (Given), an event or action (When), and expected outcomes (Then), mirroring how a user describes expected behaviour in plain language.

**Technical Definition**

GWT is a semantic variant of AAA with a stronger emphasis on user-facing language. **Given** establishes the context (the actor and initial state), **When** describes the single triggering event, and **Then** states the observable outcomes. In Laravel tests, this often maps to `describe()` blocks (context), `it()` statements (behaviour), and assertions that read like sentences. GWT is the natural structure for acceptance criteria and is commonly used in BDD-style suites.

**Beginner-Friendly Explanation**

GWT is AAA explained like a story: “Given I am logged in, When I visit my profile, Then I see my name.” It is the same three phases, but described in language that a non-developer can follow.

#### Purposes

- To express tests as user-facing scenarios, aligning with acceptance criteria and user stories.
- To bridge the communication gap between technical and non-technical stakeholders.
- To improve the readability of test names and assertion chains.
- To naturally encourage scenario coverage (each user story becomes a set of Given–When–Then tests).

#### Syntax Rules and Structure

**Complete General Syntax (Pest with `describe`/`it`)**

```php
describe('post creation', function () {
    it('creates a post when the user is authenticated', function () {
        // Given: an authenticated user
        $user = User::factory()->create();
        $this->actingAs($user);

        // When: the user submits the form
        $response = $this->post('/posts', ['title' => 'Hello', 'body' => 'World']);

        // Then: the post is created
        $response->assertRedirect('/posts');
        $this->assertDatabaseHas('posts', ['title' => 'Hello']);
    });
});
```

**Component Breakdown**

| Phase | Behaviour |
|-------|-----------|
| Given | Sets up the actor, state, and context. |
| When | Describes a single triggering event. |
| Then | States the observable outcome(s). |

**Syntax Rules**

- **Given** is analogous to Arrange; it establishes the scenario.
- **When** is analogous to Act; it describes a single event.
- **Then** is analogous to Assert; it states the expected outcome.
- Test names should read as sentences (`it('rejects an empty cart')`).

**Constraints and Limitations**

- GWT's value depends on the readability of the wording; awkwardly phrased Given/When/Then undermines the benefit.
- GWT can require more comments than AAA; the trade-off is clarity.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: GWT Scenario**

```php
describe('checkout', function () {
    it('rejects checkout when the cart is empty', function () {
        // Given: an empty cart
        $user = User::factory()->create();
        $user->cart()->create();

        // When: the user attempts checkout
        $response = $this->actingAs($user)->post('/checkout');

        // Then: checkout is rejected
        $response->assertSessionHasErrors('cart');
        $this->assertDatabaseMissing('orders', ['user_id' => $user->id]);
    });
});
```

**Expected Output**

```
PASS  Tests\Feature\CheckoutTest
✓ checkout rejects checkout when the cart is empty
```

**Why This Output Occurs**

Given establishes an empty cart. When attempts checkout. Then asserts the validation error and the absence of an order, exactly matching the scenario description.

---

#### Real-World Cases with Explanation

**Case 1: Acceptance Tests from User Stories**

A product owner writes user stories as Given–When–Then. The development team transcribes each story into a GWT test. When the story is fulfilled, the test passes; the test suite becomes a direct, executable translation of the product requirements.

**Explanation**: GWT turns user stories into a living specification.

---

### 3. Testing Happy Paths

#### Definitions

**Core Definition**

Happy-path testing verifies that a feature works correctly when the user provides valid inputs, follows the intended workflow, and encounters no errors.

**Technical Definition**

The happy path is the primary, expected flow of an operation: the success case with no validation errors, no authorization failures, and no exceptions. A happy-path test confirms the full integration—request, controller, validation (passed), persistence, and response—using the most typical valid inputs. It is the first test written for any feature and the baseline against which other scenarios are compared.

**Beginner-Friendly Explanation**

The happy path is the “everything goes right” case. You provide valid data, and the system does what it's supposed to.

#### Purposes

- To establish that the primary user workflow works correctly.
- To serve as a baseline for detecting regressions.
- To document the expected success response and persisted state.
- To verify integration across controllers, validation, and persistence.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
test('authenticated user can create a post with valid data', function () {
    // Arrange
    $user = User::factory()->create();

    // Act
    $response = $this->actingAs($user)->post('/posts', [
        'title' => 'Valid Title',
        'body'  => 'Valid body content',
    ]);

    // Assert: success and persistence
    $response->assertRedirect('/posts');
    $this->assertDatabaseHas('posts', ['title' => 'Valid Title']);
});
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Valid payload | Realistic, complete data. |
| Success status | 200, 201, 302, or 204 depending on context. |
| Persistence assertion | Row exists in the expected table. |

**Syntax Rules**

- Use realistic valid data, not placeholders.
- Assert both the response (status/redirect) and the persisted state.
- Use factories for any related models.

**Constraints and Limitations**

- Happy paths alone do not prove a feature is correct; they must be paired with edge-case, error-case, and authorization tests.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Happy Path for User Registration**

```php
test('a new user can register with valid credentials', function () {
    $response = $this->post('/register', [
        'name'     => 'Alice',
        'email'    => 'alice@example.com',
        'password' => 'password123',
        'password_confirmation' => 'password123',
    ]);

    $response->assertRedirect('/home');
    $this->assertDatabaseHas('users', ['email' => 'alice@example.com']);
});
```

**Expected Output**

```
PASS  Tests\Feature\RegistrationTest
✓ a new user can register with valid credentials
```

**Why This Output Occurs**

Valid data passes validation, the user is created, and the response redirects to `/home`. Both the redirect and the database row are asserted.

---

#### Real-World Cases with Explanation

**Case 1: Baseline for Regression Detection**

A team's happy-path test for checkout runs on every commit. When a developer accidentally breaks the checkout controller, the happy-path test fails immediately, flagging the regression before it reaches staging.

**Explanation**: The happy path is the first line of defense against regressions in critical flows.

---

### 4. Testing Edge Cases

#### Definitions

**Core Definition**

Edge-case testing verifies the behaviour of a feature when inputs or conditions are at the boundary of what is expected—empty values, maximum lengths, zero counts, unicode characters, leap years, or the smallest and largest permitted values.

**Technical Definition**

Edge cases are inputs or states that sit at the boundary of the input domain: empty strings, `null` where nullable, maximum-length strings, zero and negative numbers, boundary dates, and empty collections. Edge-case tests use Pest datasets or PHPUnit data providers to run the same scenario against multiple boundary values, verifying that the system behaves correctly without crashing or producing invalid data.

**Beginner-Friendly Explanation**

Edge cases are the “almost wrong but still allowed” scenarios. What happens if someone types the longest allowed name? What if the cart has exactly zero items? What if the count is exactly at the minimum?

#### Purposes

- To catch off-by-one errors and boundary bugs that happy paths miss.
- To verify behaviour at the extremes of the permitted input domain.
- To ensure graceful handling of empty and boundary collections.
- To document the exact limits of the system's tolerance.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
it('accepts titles at the maximum length', function (int $length, bool $valid) {
    $user = User::factory()->create();

    $response = $this->actingAs($user)->post('/posts', [
        'title' => str_repeat('a', $length),
        'body'  => 'Content',
    ]);

    $valid
        ? $response->assertRedirect('/posts')
        : $response->assertSessionHasErrors('title');
})->with([
    'at limit'   => [255, true],
    'one over'   => [256, false],
    'empty'      => [0, false],
]);
```

**Component Breakdown**

| Boundary Type | Example |
|---------------|---------|
| Length | 0, 1, max, max + 1 |
| Numeric | 0, 1, max, max + 1, negative |
| Collection | Empty, single item, large collection |
| Date | Leap year, end of month, timezone boundaries |

**Syntax Rules**

- Use datasets/providers for multiple boundary values.
- Test both the largest valid value and the smallest invalid value.
- Always test the empty/zero case explicitly.

**Constraints and Limitations**

- Edge cases only cover boundaries; interior values are covered by happy paths.
- Randomised fuzzing can complement edge testing but is not a substitute.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Boundary Length Dataset**

```php
it('validates title length boundaries', function (int $length, bool $valid) {
    $user = User::factory()->create();

    $response = $this->actingAs($user)->post('/posts', [
        'title' => str_repeat('a', $length),
        'body'  => 'Content',
    ]);

    $valid
        ? $response->assertRedirect('/posts')
        : $response->assertSessionHasErrors('title');
})->with([
    'exactly max'   => [255, true],
    'one over max'  => [256, false],
    'empty string'  => [0, false],
]);
```

**Expected Output**

```
PASS  Tests\Feature\PostValidationTest
✓ it validates title length boundaries with data set "exactly max"
✓ it validates title length boundaries with data set "one over max"
✓ it validates title length boundaries with data set "empty string"
```

**Why This Output Occurs**

Each dataset entry supplies a boundary length and the expected outcome. The system is verified to accept exactly the maximum and reject anything beyond or empty.

---

#### Real-World Cases with Explanation

**Case 1: Quantity Boundaries in E-commerce**

An e-commerce cart has a maximum quantity of 99 per item. Edge-case tests verify that quantity 99 succeeds and quantity 100 fails with a validation error, preventing a bug where users could add 100 items and cause inventory issues.

**Explanation**: Edge-case testing protects business rules at their exact limits.

---

### 5. Testing Error Cases

#### Definitions

**Core Definition**

Error-case testing verifies that a feature responds correctly when something goes wrong—validation fails, a resource is missing, an exception is thrown, or a downstream service fails.

**Technical Definition**

Error cases cover the failure branches of application logic: 422 validation errors, 404 not-found responses, 403 forbidden responses, 500 server errors, and exception handling. Error-case tests assert that the correct status code, error message, or fallback behaviour is produced, and that no unintended state change occurs. They are essential for security, robustness, and user experience.

**Beginner-Friendly Explanation**

Error cases are the “something went wrong” scenarios. What happens if the user submits bad data? What if the resource doesn't exist? Does the app show a helpful message, or does it crash?

#### Purposes

- To verify that invalid requests produce the correct error responses.
- To ensure no unintended state changes occur on failure.
- To confirm that error messages are helpful and safe (no leaked internals).
- To test exception handling and fallback behaviour.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
test('invalid input returns 422 with errors', function () {
    $user = User::factory()->create();

    $response = $this->actingAs($user)->post('/posts', []);

    $response->assertStatus(422)
             ->assertJsonValidationErrors(['title', 'body']);

    $this->assertDatabaseCount('posts', 0);
});
```

**Component Breakdown**

| Error Type | Assertion |
|------------|-----------|
| Validation failure (web) | `assertSessionHasErrors()` |
| Validation failure (API) | `assertStatus(422)->assertJsonValidationErrors()` |
| Not found | `assertNotFound()` |
| Unauthorized (unauthenticated) | `assertUnauthorized()` or redirect |
| Forbidden (authenticated) | `assertForbidden()` |
| Server error | `assertStatus(500)` |

**Syntax Rules**

- Assert the exact status code.
- Assert the specific error field(s).
- Assert the absence of unintended state changes (`assertDatabaseCount(0)`).

**Constraints and Limitations**

- Never assert on raw stack traces; assert on user-facing messages.
- Do not over-assert error text, which can change; assert the field name or error code.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: API 404 for Missing Resource**

```php
test('missing post returns 404', function () {
    $this->getJson('/api/posts/999')->assertNotFound();
});
```

**Expected Output**

```
PASS  Tests\Feature\Api\PostTest
✓ missing post returns 404
```

**Why This Output Occurs**

Route model binding fails to find a post with ID 999, and the framework returns a 404.

---

**Example 2: Web 422 with Field-Level Errors**

```php
test('posting without a title produces a field error', function () {
    $user = User::factory()->create();

    $response = $this->actingAs($user)->post('/posts', ['body' => 'Content']);

    $response->assertSessionHasErrors(['title' => 'The title field is required.']);
    $this->assertDatabaseCount('posts', 0);
});
```

**Expected Output**

```
PASS  Tests\Feature\PostValidationTest
✓ posting without a title produces a field error
```

**Why This Output Occurs**

The `title` field is missing, so validation fails with a specific error, and no post is created.

---

#### Real-World Cases with Explanation

**Case 1: Payment Failure Handling**

An e-commerce application charges via an external gateway. An error-case test uses `Http::fake()` to return a 500 from the gateway and asserts that the order remains in `pending` status, no confirmation email is sent, and the user sees a friendly error. This protects against a bug where failed payments were marked as successful.

**Explanation**: Error-case testing is critical for correctness in failure scenarios, especially with external integrations.

---

### 6. Testing Authentication

#### Definitions

**Core Definition**

Authentication testing verifies that the application correctly identifies users—granting access to authenticated requests, rejecting or redirecting unauthenticated ones, and scoping data to the correct user.

**Technical Definition**

Authentication tests use `actingAs($user)` (for session-based guards) or `Sanctum::actingAs($user, $abilities)` (for token-based APIs) to authenticate requests. They assert that authenticated users reach protected resources, that guests are redirected (web) or receive 401 (API), and that users only see their own data. Manually posting to the login endpoint is discouraged; `actingAs()` is faster and more reliable.

**Beginner-Friendly Explanation**

Authentication testing checks that the app knows who is logged in. If you're logged in, you can access your account. If you're not, you're sent to the login page or told to authenticate.

#### Purposes

- To verify that protected routes are inaccessible to guests.
- To confirm that authenticated users can access their own resources.
- To test both web (session) and API (token) authentication.
- To catch misconfigured guards or missing auth middleware.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Web session authentication
$this->actingAs($user)->get('/dashboard')->assertOk();

// Specific guard
$this->actingAs($admin, 'admin')->get('/admin')->assertOk();

// API token with abilities
Sanctum::actingAs($user, ['posts:write']);
$this->postJson('/api/posts', $data)->assertCreated();

// Guest
$this->get('/dashboard')->assertRedirect('/login');
$this->getJson('/api/user')->assertUnauthorized();
```

**Component Breakdown**

| Method | Use Case |
|--------|----------|
| `actingAs($user)` | Authenticate for the default guard. |
| `actingAs($user, $guard)` | Authenticate for a specific guard. |
| `Sanctum::actingAs($user, $abilities)` | API token with scoped abilities. |

**Syntax Rules**

- Authenticate in the Arrange phase.
- Assert both authenticated success and guest failure.
- For APIs, prefer 401 for unauthenticated and 403 for unauthorized.

**Constraints and Limitations**

- `actingAs()` bypasses the login flow; test login separately.
- Sanctum abilities must match those checked in policies/middleware.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Web Authentication**

```php
test('authenticated user can view dashboard', function () {
    $user = User::factory()->create();
    $this->actingAs($user)->get('/dashboard')->assertOk();
});

test('guest is redirected from dashboard', function () {
    $this->get('/dashboard')->assertRedirect('/login');
});
```

**Expected Output**

```
PASS  Tests\Feature\AuthTest
✓ authenticated user can view dashboard
✓ guest is redirected from dashboard
```

**Why This Output Occurs**

The first test authenticates and reaches the dashboard; the second confirms guests are redirected by the `auth` middleware.

---

**Example 2: API Token with Abilities**

```php
use Laravel\Sanctum\Sanctum;

test('token with write ability can create a post', function () {
    $user = User::factory()->create();
    Sanctum::actingAs($user, ['posts:write']);

    $this->postJson('/api/posts', ['title' => 'API Post'])->assertCreated();
});

test('unauthenticated API request returns 401', function () {
    $this->getJson('/api/user')->assertUnauthorized();
});
```

**Expected Output**

```
PASS  Tests\Feature\Api\AuthTest
✓ token with write ability can create a post
✓ unauthenticated API request returns 401
```

**Why This Output Occurs**

`Sanctum::actingAs()` issues a token with the required ability, allowing creation. Without a token, the endpoint returns 401.

---

#### Real-World Cases with Explanation

**Case 1: Multi-Guard Application**

An application has `web` and `admin` guards. Authentication tests verify that a regular user cannot access admin routes, an admin can access both, and a guest is redirected. Each guard is tested independently.

**Explanation**: Multi-guard authentication testing ensures each user type is correctly scoped.

---

### 7. Testing Authorization

#### Definitions

**Core Definition**

Authorization testing verifies that authenticated users can only perform actions and access resources they are permitted to, enforcing policies, gates, and middleware.

**Technical Definition**

Authorization tests assert the complete permission matrix: the resource owner can update/delete, other users cannot, admins may have overrides, and guests are redirected. They assert 200/201 for allowed actions, 403 for forbidden, and (for tenant isolation) 404 to avoid leaking resource existence. Authorization should be tested both at the policy level (unit-style) and the HTTP level (feature-style) to confirm the endpoint actually invokes the check.

**Beginner-Friendly Explanation**

Authorization testing checks that users can only do what they're allowed to. Can Alice edit her post? Can Bob edit Alice's post? (He should get a 403.)

#### Purposes

- To prevent privilege escalation and cross-user data leaks.
- To verify policies and gates are invoked by endpoints.
- To cover the full permission matrix (owner, other user, admin, guest).
- To confirm tenant isolation (often via 404 responses).

#### Syntax Rules and Structure

**Complete General Syntax**

```php
test('owner can update their post', function () {
    $owner = User::factory()->create();
    $post  = Post::factory()->for($owner)->create();

    $this->actingAs($owner)
         ->put("/posts/{$post->id}", ['title' => 'Updated'])
         ->assertRedirect();
});

test('other user cannot update the post', function () {
    $other = User::factory()->create();
    $post  = Post::factory()->for(User::factory())->create();

    $this->actingAs($other)
         ->put("/posts/{$post->id}", ['title' => 'Hacked'])
         ->assertForbidden();
});
```

**Component Breakdown**

| Scenario | Expected Result |
|----------|-----------------|
| Owner acts | 200/201/302 success |
| Non-owner acts | 403 forbidden |
| Guest acts | Redirect or 401 |
| Cross-tenant access | 404 not found (hide existence) |

**Syntax Rules**

- Test both the allowed and forbidden branches.
- Use 404 instead of 403 for tenant isolation to avoid leaking existence.
- Use factories with relationship helpers (`for()`) to establish ownership.

**Constraints and Limitations**

- A 403 does not indicate which layer denied the request; policy-level tests pinpoint the rule.
- Do not rely solely on the UI hiding buttons; test the endpoints directly.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Ownership Matrix**

```php
test('owner can delete their post', function () {
    $owner = User::factory()->create();
    $post  = Post::factory()->for($owner)->create();

    $this->actingAs($owner)->delete("/posts/{$post->id}")->assertRedirect('/posts');
    $this->assertDatabaseMissing('posts', ['id' => $post->id]);
});

test('non-owner cannot delete the post', function () {
    $other = User::factory()->create();
    $post  = Post::factory()->for(User::factory())->create();

    $this->actingAs($other)->delete("/posts/{$post->id}")->assertForbidden();
    $this->assertDatabaseHas('posts', ['id' => $post->id]);
});
```

**Expected Output**

```
PASS  Tests\Feature\AuthorizationTest
✓ owner can delete their post
✓ non-owner cannot delete the post
```

**Why This Output Occurs**

The policy allows the owner to delete and forbids others. The tests assert both the response and the database state, proving the rule was enforced.

---

**Example 2: Admin Override**

```php
test('admin can delete any post', function () {
    $admin = User::factory()->admin()->create();
    $post  = Post::factory()->for(User::factory())->create();

    $this->actingAs($admin)->delete("/posts/{$post->id}")->assertRedirect('/posts');
    $this->assertDatabaseMissing('posts', ['id' => $post->id]);
});
```

**Expected Output**

```
PASS  Tests\Feature\AuthorizationTest
✓ admin can delete any post
```

**Why This Output Occurs**

The policy's `before` method or admin rule grants the admin override, allowing deletion.

---

#### Real-World Cases with Explanation

**Case 1: Multi-Tenant Isolation**

A multi-tenant SaaS application tests that a user from Tenant A cannot access Tenant B's orders. The test asserts a 404 (not 403), preventing the leak of the record's existence. This test has caught cross-tenant bugs in production reviews.

**Explanation**: Authorization testing is the primary defense against cross-tenant data leaks.

---

### 8. Testing Validation

#### Definitions

**Core Definition**

Validation testing verifies that incoming data is validated according to the application's rules, that invalid input produces the expected error response, and that valid input passes through to the controller.

**Technical Definition**

Validation tests submit data that violates Form Request rules and assert on the error response: `assertSessionHasErrors()` for web forms and `assertJsonValidationErrors()` for APIs. Datasets are used to run the same validation test against multiple invalid inputs efficiently. Both the error response and the absence of persistence should be asserted to confirm the request was rejected completely.

**Beginner-Friendly Explanation**

Validation testing checks that bad input is rejected with helpful error messages. If I submit a form without an email, do I get an error telling me the email is required?

#### Purposes

- To verify that every validation rule is enforced.
- To ensure invalid data produces the correct error response (session or 422).
- To confirm that no data is persisted on validation failure.
- To document the input contract of every endpoint.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Web
$this->post('/posts', [])->assertSessionHasErrors(['title', 'body']);

// Specific error message
$this->post('/posts', ['title' => ''])
     ->assertSessionHasErrors(['title' => 'The title field is required.']);

// API
$this->postJson('/api/posts', [])
     ->assertStatus(422)
     ->assertJsonValidationErrors(['title', 'body']);
```

**Component Breakdown**

| Assertion | Use Case |
|-----------|----------|
| `assertSessionHasErrors()` | Web forms (redirect + session). |
| `assertSessionHasNoErrors()` | Valid input passes. |
| `assertJsonValidationErrors()` | API 422 responses. |

**Syntax Rules**

- Assert both the error fields and the absence of persistence.
- Use datasets for multiple invalid inputs.
- Test one rule per test (or one dataset entry per rule) to localize failures.

**Constraints and Limitations**

- Validation error messages can change; assert field names where possible.
- Conditional rules (e.g., `required_if`) require dedicated tests for each branch.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Web Validation**

```php
test('title is required and no post is created', function () {
    $user = User::factory()->create();

    $response = $this->actingAs($user)->post('/posts', ['body' => 'Content']);

    $response->assertSessionHasErrors('title');
    $this->assertDatabaseCount('posts', 0);
});
```

**Expected Output**

```
PASS  Tests\Feature\ValidationTest
✓ title is required and no post is created
```

**Why This Output Occurs**

The missing `title` triggers the `required` rule, produces a session error, and prevents the post from being created.

---

**Example 2: API Validation Dataset**

```php
it('rejects invalid payloads', function (array $payload, array $errors) {
    $this->postJson('/api/users', $payload)
         ->assertStatus(422)
         ->assertJsonValidationErrors($errors);
})->with([
    'missing email'   => [['name' => 'Alice'], ['email']],
    'invalid email'   => [['name' => 'Alice', 'email' => 'bad'], ['email']],
    'short password'  => [['name' => 'Alice', 'email' => 'a@b.com', 'password' => 'x'], ['password']],
]);
```

**Expected Output**

```
PASS  Tests\Feature\Api\UserValidationTest
✓ it rejects invalid payloads with data set "missing email"
✓ it rejects invalid payloads with data set "invalid email"
✓ it rejects invalid payloads with data set "short password"
```

**Why This Output Occurs**

Each dataset entry submits an invalid payload and asserts the specific field(s) that should fail, covering multiple rules with a single test closure.

---

#### Real-World Cases with Explanation

**Case 1: Registration Form Coverage**

A registration form has five rules (name, email uniqueness, password length, password confirmation, terms acceptance). A dataset drives one test per rule, giving complete validation coverage with minimal code and clear failure localization.

**Explanation**: Validation testing with datasets is the most efficient way to cover a complete input contract.

---

### 9. Testing APIs

#### Definitions

**Core Definition**

API testing verifies the behaviour of JSON endpoints: status codes, JSON structure, authentication, validation errors, pagination, and side effects.

**Technical Definition**

API tests use JSON helpers (`getJson()`, `postJson()`, `putJson()`, `patchJson()`, `deleteJson()`) and JSON assertions (`assertJson()`, `assertJsonPath()`, `assertJsonStructure()`, `assertJsonCount()`, and the fluent `AssertableJson` API). They assert exact status codes, structure rather than exact payloads, and validation errors via `assertJsonValidationErrors()`. Authentication for APIs typically uses `Sanctum::actingAs()`.

**Beginner-Friendly Explanation**

API testing checks that your JSON endpoints return the right data in the right shape with the right status code.

#### Purposes

- To verify the JSON contract between backend and frontend.
- To assert exact status codes and error structures.
- To test pagination, filtering, and sorting.
- To confirm authentication and authorization on API endpoints.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
$response = $this->postJson('/api/posts', [
    'title' => 'Hello',
    'body'  => 'World',
]);

$response->assertCreated()
         ->assertJsonStructure([
             'data' => ['id', 'title', 'body', 'created_at'],
         ]);
```

**Component Breakdown**

| Assertion | Purpose |
|-----------|---------|
| `assertCreated()` | 201 Created. |
| `assertOk()` | 200 OK. |
| `assertNoContent()` | 204 No Content. |
| `assertJsonStructure()` | Required keys exist. |
| `assertJsonPath()` | Specific value at a JSON path. |
| `assertJsonValidationErrors()` | 422 validation errors. |

**Syntax Rules**

- Use `*Json()` helpers for API endpoints.
- Assert structure (`assertJsonStructure`) rather than exact payloads.
- For complex payloads, use `AssertableJson` with `->etc()`.
- Assert exact status codes, not just `assertOk()`.

**Constraints and Limitations**

- Exact JSON matching (`assertExactJson`) is brittle; prefer structural assertions.
- API tests must account for pagination wrappers if using `Resource::collection()`.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Create Endpoint with Structure Assertion**

```php
use Laravel\Sanctum\Sanctum;

test('api creates a post and returns the resource', function () {
    $user = User::factory()->create();
    Sanctum::actingAs($user, ['posts:write']);

    $response = $this->postJson('/api/posts', [
        'title' => 'Hello',
        'body'  => 'World',
    ]);

    $response->assertCreated()
             ->assertJsonPath('data.title', 'Hello')
             ->assertJsonStructure([
                 'data' => ['id', 'title', 'body', 'created_at'],
             ]);
});
```

**Expected Output**

```
PASS  Tests\Feature\Api\PostTest
✓ api creates a post and returns the resource
```

**Why This Output Occurs**

The endpoint returns a 201 with the created resource. The assertions verify the exact title and the structure.

---

**Example 2: Fluent AssertableJson for Collections**

```php
use Illuminate\Testing\Fluent\AssertableJson;

test('api lists orders with pagination', function () {
    $user = User::factory()->create();
    Order::factory()->count(3)->for($user)->create();
    Sanctum::actingAs($user, ['orders:read']);

    $this->getJson('/api/orders')
         ->assertOk()
         ->assertJson(fn (AssertableJson $json) =>
             $json->has('data', 3)
                  ->has('data.0', fn (AssertableJson $item) =>
                      $item->hasAll(['id', 'amount_cents', 'status'])->etc()
                  )
                  ->has('links')
                  ->has('meta')
         );
});
```

**Expected Output**

```
PASS  Tests\Feature\Api\OrderTest
✓ api lists orders with pagination
```

**Why This Output Occurs**

The fluent `AssertableJson` API verifies the collection size, required item keys, and pagination links and meta.

---

#### Real-World Cases with Explanation

**Case 1: SPA Backend Contract**

A Vue SPA depends on the API's JSON structure. API tests with `assertJsonStructure()` verify that every endpoint returns the expected keys. When a backend developer accidentally renames `data` to `items`, the API test fails immediately, preventing a broken frontend release.

**Explanation**: API tests act as the contract between backend and frontend.

---

### 10. Testing Jobs

#### Definitions

**Core Definition**

Job testing verifies that a job is dispatched when expected and that the job's `handle()` method performs the correct work when executed.

**Technical Definition**

Jobs are tested in two complementary ways. **Dispatch testing** uses `Queue::fake()` to assert that a job was pushed with the correct payload and queue—without executing it. **Execution testing** instantiates the job directly and calls `handle()`, asserting on the database or other side effects. Jobs should implement `ShouldQueue` for asynchronous dispatch; dispatch tests should assert the job class and key arguments.

**Beginner-Friendly Explanation**

Job testing checks two things: first, that your code correctly queues the job when it should; second, that the job itself does the right work when it runs.

#### Purposes

- To verify that jobs are dispatched under the correct conditions.
- To confirm that job payloads contain the correct data.
- To test the job's actual logic in isolation.
- To ensure jobs go to the correct queue.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
use Illuminate\Support\Facades\Queue;

test('registration dispatches the welcome email job', function () {
    Queue::fake();

    User::factory()->create(['email' => 'a@b.com']);

    Queue::assertPushed(SendWelcomeEmail::class, function ($job) {
        return $job->email === 'a@b.com';
    });
});

test('the job creates a record', function () {
    $job = new SendWelcomeEmail('a@b.com');
    $job->handle();

    $this->assertDatabaseHas('welcome_emails', ['email' => 'a@b.com']);
});
```

**Component Breakdown**

| Approach | Method | Verifies |
|----------|--------|----------|
| Dispatch | `Queue::fake()` + `assertPushed()` | Job queued with correct data. |
| Execution | `$job->handle()` | Job's logic works. |

**Syntax Rules**

- Use `Queue::fake()` for dispatch tests; pass a class name to narrow the fake.
- Assert the job class and key arguments.
- For execution tests, instantiate the job and call `handle()`.
- Assert the queue with `->onQueue()` if the job specifies one.

**Constraints and Limitations**

- `Queue::fake()` prevents the job from running; execution tests must be separate.
- Job properties must be public or accessible for assertions.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Dispatch Assertion**

```php
test('order placement queues an invoice job', function () {
    Queue::fake();

    $order = Order::factory()->create();
    $order->place();

    Queue::assertPushed(GenerateInvoice::class, function ($job) use ($order) {
        return $job->order->id === $order->id;
    });
});
```

**Expected Output**

```
PASS  Tests\Feature\OrderJobTest
✓ order placement queues an invoice job
```

**Why This Output Occurs**

`Queue::fake()` captures the dispatch, and the assertion verifies the correct job class and payload.

---

**Example 2: Job Execution Assertion**

```php
test('the job creates an invoice record', function () {
    $order = Order::factory()->create();

    (new GenerateInvoice($order))->handle();

    $this->assertDatabaseHas('invoices', ['order_id' => $order->id]);
});
```

**Expected Output**

```
PASS  Tests\Unit\GenerateInvoiceTest
✓ the job creates an invoice record
```

**Why This Output Occurs**

The job is instantiated and executed directly, and its database side effect is asserted.

---

#### Real-World Cases with Explanation

**Case 1: Order Fulfillment Pipeline**

An order is placed, and several jobs are dispatched (invoice, shipping label, confirmation email). Dispatch tests verify the pipeline; execution tests verify each job's logic. Together, they ensure the pipeline works without needing a queue worker in the test.

**Explanation**: Splitting dispatch and execution tests keeps each test focused and fast.

---

### 11. Testing Events

#### Definitions

**Core Definition**

Event testing verifies that domain events are dispatched with the correct payload and that listeners react to them appropriately.

**Technical Definition**

Event dispatch is tested with `Event::fake()` and `Event::assertDispatched()` / `assertNotDispatched()`. Pass a specific class to `Event::fake([EventClass::class])` to narrow the fake and avoid hiding unexpected dispatches. Listener behaviour is tested by manually dispatching the event and asserting the listener's side effect, or by using `Event::fake()` plus `assertListening()` to verify wiring.

**Beginner-Friendly Explanation**

Event testing checks that your app fires the right events (like “order shipped”) and that the listeners do the right thing.

#### Purposes

- To verify that the correct events are dispatched with correct payloads.
- To ensure events are not dispatched when they shouldn't be.
- To test listener logic and side effects.
- To confirm event–listener wiring.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
use Illuminate\Support\Facades\Event;

Event::fake([OrderPlaced::class]);

$order->place();

Event::assertDispatched(OrderPlaced::class, function ($event) use ($order) {
    return $event->order->id === $order->id;
});

Event::assertNotDispatched(OrderCancelled::class);
Event::assertListening(OrderPlaced::class, SendInvoiceListener::class);
```

**Component Breakdown**

| Assertion | Verifies |
|-----------|----------|
| `assertDispatched()` | Event was dispatched (with closure payload check). |
| `assertNotDispatched()` | Event was not dispatched. |
| `assertListening()` | Listener is wired to the event. |

**Syntax Rules**

- Pass specific event classes to `Event::fake()` to avoid hiding unrelated dispatches.
- Use `assertListening()` to verify listener registration.
- For listener logic, dispatch the event and assert the side effect.

**Constraints and Limitations**

- `Event::fake()` prevents listeners from running; test listener logic separately.
- Model events (`created`, `updated`) require `Event::fake()` with the model events or dedicated observer tests.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Event Dispatch**

```php
test('placing an order dispatches OrderPlaced', function () {
    Event::fake([OrderPlaced::class]);

    $order = Order::factory()->create();
    $order->place();

    Event::assertDispatched(OrderPlaced::class, fn ($e) => $e->order->is($order));
    Event::assertNotDispatched(OrderCancelled::class);
});
```

**Expected Output**

```
PASS  Tests\Feature\OrderEventTest
✓ placing an order dispatches OrderPlaced
```

**Why This Output Occurs**

`Event::fake([OrderPlaced::class])` captures the dispatch. The assertion verifies the correct event and payload; the negative assertion confirms no cancellation was fired.

---

**Example 2: Listener Side Effect**

```php
test('the listener sends a confirmation email', function () {
    Mail::fake();
    $order = Order::factory()->create();

    event(new OrderPlaced($order));

    Mail::assertSent(OrderConfirmation::class, fn ($mail) => $mail->hasTo($order->user->email));
});
```

**Expected Output**

```
PASS  Tests\Feature\OrderListenerTest
✓ the listener sends a confirmation email
```

**Why This Output Occurs**

The event is dispatched manually, the listener runs, and `Mail::fake()` captures the email that the listener sent.

---

#### Real-World Cases with Explanation

**Case 1: Decoupled Order Pipeline**

An `OrderPlaced` event triggers listeners for invoicing, emailing, and analytics. Dispatch tests verify the event; listener tests verify each side effect. This decouples the tests and keeps them fast.

**Explanation**: Event testing ensures the event-driven architecture is wired correctly.

---

### 12. Testing Notifications

#### Definitions

**Core Definition**

Notification testing verifies that Laravel notifications—sent via mail, SMS, Slack, or database channels—are dispatched to the correct recipients with the correct content.

**Technical Definition**

`Notification::fake()` swaps the notification system for a fake that records notifications. Assertions include `assertSentTo()`, `assertNotSentTo()`, `assertSentOnDemand()`, `assertCount()`, and channel-specific assertions via closures. Notifications are typically tested at the point of dispatch (in a controller or listener) rather than by executing the notification's `toMail()` method directly.

**Beginner-Friendly Explanation**

Notification testing checks that the right notification goes to the right person through the right channel.

#### Purposes

- To verify notifications are sent to the correct recipients.
- To confirm the correct notification class is used.
- To ensure no unintended notifications are sent.
- To test on-demand notifications (to a phone number or email not tied to a model).

#### Syntax Rules and Structure

**Complete General Syntax**

```php
use Illuminate\Support\Facades\Notification;

Notification::fake();

$user->notify(new OrderShipped($order));

Notification::assertSentTo(
    $user,
    OrderShipped::class,
    fn ($notification, $channels) => in_array('mail', $channels)
);

Notification::assertNotSentTo($otherUser, OrderShipped::class);
```

**Component Breakdown**

| Assertion | Verifies |
|-----------|----------|
| `assertSentTo($notifiable, $class)` | Notification sent to the notifiable. |
| `assertNotSentTo()` | Notification not sent. |
| `assertSentOnDemand()` | On-demand notification sent. |
| `assertCount()` | Number of notifications. |

**Syntax Rules**

- `Notification::fake()` must be called before the action.
- The notifiable (usually a user) is the first argument.
- The closure receives the notification and the channels used.

**Constraints and Limitations**

- `Notification::fake()` prevents the notification from actually sending; channel-specific rendering is tested separately.
- Database notifications must be tested via `assertDatabaseHas('notifications', ...)` when using the database channel.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Notification Sent to the Correct User**

```php
test('order shipping notifies the buyer', function () {
    Notification::fake();
    $order = Order::factory()->create();

    $order->ship();

    Notification::assertSentTo($order->user, OrderShipped::class);
    Notification::assertNotSentTo(User::factory()->create(), OrderShipped::class);
});
```

**Expected Output**

```
PASS  Tests\Feature\NotificationTest
✓ order shipping notifies the buyer
```

**Why This Output Occurs**

`Notification::fake()` captures the notification. The assertions confirm it went to the buyer and not to another user.

---

**Example 2: Channel-Specific Assertion**

```php
Notification::assertSentTo(
    $user,
    OrderShipped::class,
    fn ($notification, $channels) =>
        in_array('mail', $channels) && in_array('database', $channels)
);
```

**Expected Output**

The test passes if the notification was sent via both the `mail` and `database` channels.

**Why This Output Occurs**

The closure inspects the channels array and asserts the expected channels were used.

---

#### Real-World Cases with Explanation

**Case 1: Multi-Channel Notifications**

An application sends order updates via email and database notifications. Tests assert both channels are used and that the correct user is the recipient, catching a bug where a previous version sent notifications to the wrong user.

**Explanation**: Notification testing ensures the correct recipient and channels are used for every notification.

---

### 13. Testing Mail

#### Definitions

**Core Definition**

Mail testing verifies that mailables are sent, queued, or rendered correctly, with the right recipients, subject, and content.

**Technical Definition**

`Mail::fake()` swaps the mailer for a fake. Assertions include `assertSent()`, `assertNotSent()`, `assertQueued()`, `assertNothingSent()`, and `assertNothingOutgoing()`. The closure receives the mailable instance and can inspect recipients, subject, and rendered content. Mailables can also be rendered and asserted directly with `$mailable->assertSeeInHtml()`.

**Beginner-Friendly Explanation**

Mail testing checks that the right email goes to the right person with the right content—without actually sending it.

#### Purposes

- To verify mailables are sent or queued at the correct time.
- To assert the recipients, subject, and content.
- To confirm mail is queued (not sent synchronously) in production paths.
- To prevent accidental emails during test runs.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
use Illuminate\Support\Facades\Mail;

Mail::fake();

Mail::assertSent(WelcomeEmail::class, function ($mail) {
    return $mail->hasTo('alice@example.com')
        && $mail->hasSubject('Welcome!');
});

Mail::assertQueued(InvoiceEmail::class);
Mail::assertNothingSent();
```

**Component Breakdown**

| Assertion | Verifies |
|-----------|----------|
| `assertSent()` | Mailable was sent. |
| `assertQueued()` | Mailable was queued. |
| `assertNotSent()` | Mailable was not sent. |
| `assertNothingSent()` | No mailables were sent. |
| `$mail->hasTo()` | Recipient. |
| `$mail->hasSubject()` | Subject line. |

**Syntax Rules**

- `Mail::fake()` must be called before the action.
- Use `assertQueued()` for `ShouldQueue` mailables.
- Assert recipients and subject inside the closure.

**Constraints and Limitations**

- `Mail::fake()` prevents the email from being sent; it does not test the rendering of the email template.
- To test rendering, use `$mailable->assertSeeInHtml()` outside of a fake context.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Assert Mail Was Sent**

```php
test('registration sends a welcome email', function () {
    Mail::fake();

    $this->post('/register', [
        'name' => 'Alice',
        'email' => 'alice@example.com',
        'password' => 'password123',
        'password_confirmation' => 'password123',
    ]);

    Mail::assertSent(WelcomeEmail::class, function ($mail) {
        return $mail->hasTo('alice@example.com')
            && $mail->hasSubject('Welcome to the App!');
    });
});
```

**Expected Output**

```
PASS  Tests\Feature\RegistrationMailTest
✓ registration sends a welcome email
```

**Why This Output Occurs**

The registration flow sends the `WelcomeEmail` to Alice with the expected subject. The closure assertions verify both.

---

**Example 2: Assert Mail Was Queued**

```php
test('invoice email is queued', function () {
    Mail::fake();

    $order = Order::factory()->create();
    $order->generateInvoice();

    Mail::assertQueued(InvoiceEmail::class, fn ($mail) => $mail->hasTo($order->user->email));
});
```

**Expected Output**

```
PASS  Tests\Feature\InvoiceMailTest
✓ invoice email is queued
```

**Why This Output Occurs**

The mailable implements `ShouldQueue`, so it is queued rather than sent synchronously. `assertQueued()` confirms the queue dispatch.

---

#### Real-World Cases with Explanation

**Case 1: Preventing Real Emails in Tests**

A CI pipeline runs the full test suite, including registration and checkout tests that send emails. Without `Mail::fake()`, the suite would send hundreds of real emails. With the fake, the emails are captured, and the tests assert on their content instead.

**Explanation**: Mail fakes are essential for both performance and preventing unintended real-world side effects.

---

## Testing Patterns Best Practices

1. **Apply AAA or GWT Consistently**: Pick one structural pattern and apply it across the entire suite. Consistency aids readability and code review.

2. **Follow the One-Act Rule**: Every test should perform exactly one action (one HTTP call, one method invocation). Multiple actions make failure localization ambiguous.

3. **Test the Full Coverage Matrix**: For every endpoint or feature, test happy paths, edge cases, error cases, and the full authentication/authorization matrix. A feature without a negative test is not fully tested.

4. **Prefer Datasets for Boundary Values**: Pest datasets (or PHPUnit data providers) eliminate duplication when testing multiple boundary values against the same rule.

5. **Use Framework Fakes Over Real Side Effects**: `Queue::fake()`, `Event::fake()`, `Mail::fake()`, `Notification::fake()`, and `Storage::fake()` prevent unintended side effects and provide expressive assertions.

6. **Separate Dispatch Tests from Execution Tests**: For jobs and listeners, test dispatch with a fake and execution by invoking the handler directly. This keeps tests fast and focused.

7. **Narrow Fakes to Specific Classes**: `Event::fake([OrderPlaced::class])` and `Queue::fake([GenerateInvoice::class])` prevent unexpected dispatches from being silently ignored.

8. **Assert Both Response and State**: After a mutation, assert the HTTP response (status/redirect/JSON) and the persisted state (database row, dispatched job, queued email).

9. **Use Explicit Status Codes**: Prefer `assertStatus(201)` or `assertForbidden()` over generic `assertOk()`, which can hide regressions.

10. **Test Tenant Isolation with 404**: For multi-tenant applications, assert 404 (not 403) when accessing another tenant's resources, to avoid leaking existence.

11. **Keep Tests Deterministic**: Use `freezeTime()`, `travelTo()`, and controlled factories; avoid random values in assertions.

12. **Run the Full Pattern Set in CI**: Ensure every pattern (happy, edge, error, auth, validation, side effects) runs on every push to catch regressions before deployment.

---

## References

- Laravel Mocking Documentation — https://laravel.com/docs/7.x/mocking
- Laravel Test Doubles: Mocks, Stubs, Fakes, and Spies — https://laravel.com/docs/12.x/mocking
- Pest Higher Order Testing — https://pestphp.com/docs/higher-order-testing
- Pest Datasets — https://pestphp.com/docs/datasets
- PHPUnit Manual: Writing Tests for PHPUnit — https://docs.phpunit.de/
- Laravel Database Testing — https://laravel.com/docs/12.x/database-testing
- Laravel HTTP Tests — https://laravel.com/docs/12.x/http-tests
- Laravel Queues: Testing — https://laravel.com/docs/12.x/queues#testing
- Laravel Events: Testing — https://laravel.com/docs/12.x/events#testing
- Laravel Notifications: Testing — https://laravel.com/docs/12.x/notifications#testing
- Laravel Mail: Testing — https://laravel.com/docs/12.x/mail#testing
- Test Matrix — QA Skills — https://qaskills.sh/blog/laravel-test-matrix
- Laravel Testing Best Practices — https://github.com/voku/agent-skills/blob/main/skills/laravel-testing/rules/http-feature-testing.md
- Laravel TDD Skill — https://github.com/affaan-m/ECC/blob/main/skills/laravel-tdd/SKILL.md
- The Three A's of Unit Testing (AAA Pattern) — https://martinfowler.com/bliki/GivenWhenThen.html

---

**Version Note**: This cheat sheet is based on Laravel 10.x through 13.x documentation. The structural patterns (AAA, GWT) and coverage patterns are framework-agnostic but illustrated with Laravel-specific helpers. `Queue::fake()`, `Event::fake()`, `Mail::fake()`, and `Notification::fake()` have remained stable across these versions. `assertListening()` was introduced in Laravel 9. `Sanctum::actingAs()` is the recommended API authentication method. Pest 4/5 and PHPUnit 11/12 are the current test frameworks. Always consult the documentation for your specific Laravel version.