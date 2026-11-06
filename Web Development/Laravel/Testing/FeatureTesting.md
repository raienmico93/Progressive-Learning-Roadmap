# Laravel Feature Testing: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Feature testing in Laravel is the practice of testing complete HTTP endpoints—sending in-process requests through the full application stack (routing, middleware, controllers, validation, policies, and database) and asserting on the resulting response and persisted state.

**Technical Definition**

Feature tests reside in `tests/Feature` and boot the full Laravel application kernel, unlike unit tests. They use Laravel's HTTP testing helpers (`$this->get()`, `$this->post()`, `$this->postJson()`, etc.) to simulate requests without issuing real network calls. The returned `Illuminate\Testing\TestResponse` instance provides fluent assertions for status codes, headers, JSON structure, redirects, session data, validation errors, and views. Feature tests are isolated using `RefreshDatabase` (or `DatabaseMigrations`/`DatabaseTruncation`) and authenticate via `actingAs()` or `Sanctum::actingAs()`.

**Beginner-Friendly Explanation**

Feature testing is how you check that your whole application works from the outside in. Instead of testing one tiny method in isolation, you pretend to be a user (or an API client) and send a request to your app: “If I POST to `/posts` with this data, do I get redirected back with a success message, and was the post actually saved to the database?” Feature tests catch integration bugs that unit tests miss—wrong routes, missing middleware, broken validation, or authorization loopholes.

---

### Key Characteristics

- **Full Stack Exercise**: Feature tests exercise routing, middleware, validation, controllers, authorization, and database persistence together in a single honest test.
- **In-Process Requests**: Laravel’s HTTP helpers simulate requests internally; no live server is required, and tests run fast.
- **Fluent Assertions**: `TestResponse` provides dozens of named assertion methods (`assertStatus`, `assertJson`, `assertRedirect`, `assertSessionHasErrors`, etc.) that target the exact subject of the check.
- **Database Isolation**: `RefreshDatabase` wraps each test in a transaction that is rolled back, keeping tests independent and fast.
- **Authentication Helpers**: `actingAs($user)` authenticates a session-based user; `Sanctum::actingAs($user, $abilities)` authenticates API tokens with scoped abilities.
- **Authorization Matrix Testing**: Feature tests verify that the correct user succeeds and the wrong user is refused—the “second half” is what prevents cross-user data leaks.
- **Arrange–Act–Assert Structure**: Feature tests are structured around setup (Arrange), the HTTP request (Act), and assertions on response and database state (Assert).

---

### Prerequisites

- **PHP 8.1+** (Laravel 10+) or **PHP 8.2+** (Laravel 11+).
- **A Laravel application** with routes defined in `routes/web.php` and/or `routes/api.php`.
- **Pest or PHPUnit** installed (both ship with Laravel).
- **A test database** configured (SQLite in-memory is recommended for speed).
- **Model factories** for generating test data.
- **`RefreshDatabase`** trait for database isolation.

---

### Related Programming Areas

- **Unit Testing**: Complements feature tests by isolating pure logic; feature tests catch integration issues.
- **HTTP Testing**: The mechanism by which feature tests simulate requests and inspect responses.
- **Authentication and Authorization**: Feature tests verify that guards, gates, and policies are correctly wired.
- **Validation**: Feature tests confirm that Form Requests reject invalid data and return the correct error responses.
- **Database Testing**: Factories, `RefreshDatabase`, and `assertDatabaseHas` verify persistence.
- **Browser Testing**: Dusk/Playwright tests cover JavaScript behaviour that HTTP feature tests cannot reach.

---

## Core Concepts

---

### 1. Feature Testing

#### Definitions

**Core Definition**

Feature testing is the practice of sending requests through the full application stack and asserting that the response and resulting application state match expectations.

**Technical Definition**

A feature test extends `Tests\TestCase`, which boots the Laravel application. The test calls an HTTP helper method (`get`, `post`, `put`, `patch`, `delete`) with a URI and optional data, receiving a `TestResponse`. Assertions are chained on the response to verify status codes, headers, body content, JSON structure, session state, and database records. Feature tests are the primary test type in Laravel applications because they provide the most confidence per unit of effort.

**Beginner-Friendly Explanation**

A feature test is a “pretend request.” You tell Laravel, “Pretend I just visited `/posts`,” and then you check what happened: did I get a 200? Did the page contain my post? Did the database get updated?

#### Purposes

- To verify that entire features work end-to-end, not just isolated methods.
- To catch integration bugs (wrong middleware, broken routes, missing validation).
- To provide the highest confidence that the application behaves correctly from a user’s perspective.
- To document the expected behaviour of every endpoint.

#### Syntax Rules and Structure

**Complete General Syntax (Pest)**

```php
<?php
// tests/Feature/PostTest.php

use App\Models\Post;
use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;

uses(RefreshDatabase::class);

test('authenticated user can create a post', function () {
    $user = User::factory()->create();

    $response = $this->actingAs($user)->post('/posts', [
        'title' => 'My First Post',
        'body'  => 'Hello world',
    ]);

    $response->assertRedirect('/posts');
    $this->assertDatabaseHas('posts', ['title' => 'My First Post']);
});
```

**Complete General Syntax (PHPUnit)**

```php
<?php
namespace Tests\Feature;

use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class PostTest extends TestCase
{
    use RefreshDatabase;

    public function test_authenticated_user_can_create_a_post(): void
    {
        $user = User::factory()->create();

        $response = $this->actingAs($user)->post('/posts', [
            'title' => 'My First Post',
            'body'  => 'Hello world',
        ]);

        $response->assertRedirect('/posts');
        $this->assertDatabaseHas('posts', ['title' => 'My First Post']);
    }
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `uses(RefreshDatabase::class)` | Isolates database state per test. |
| `$this->actingAs($user)` | Authenticates the request as the given user. |
| `$this->post('/posts', $data)` | Sends a POST request with form data. |
| `$response->assertRedirect('/posts')` | Asserts the response redirects to `/posts`. |
| `$this->assertDatabaseHas()` | Asserts the record exists in the database. |

**Syntax Rules**

- Each test should make only **one** request to the application.
- `RefreshDatabase` should be applied to feature test classes or via `pest()->use()`.
- Factories should be used to create test data instead of manual inserts.
- Assertions should target the specific expected behaviour (exact status code, specific redirect, specific database state).

**Constraints and Limitations**

- Feature tests are slower than unit tests because they boot the framework and interact with the database.
- They cannot easily test JavaScript-driven behaviour; browser tests are required for that.
- Mocking the `Request` facade is discouraged; use HTTP testing helpers instead.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: A Basic Feature Test**

**Code**

```php
<?php
// tests/Feature/HomePageTest.php

test('home page returns 200', function () {
    $response = $this->get('/');
    $response->assertStatus(200);
});
```

**Expected Output**

```
PASS  Tests\Feature\HomePageTest
✓ home page returns 200
```

**Why This Output Occurs**

The `get('/')` helper simulates a GET request to the root URI. The application returns a response, and `assertStatus(200)` verifies the HTTP status code.

---

**Example 2: Feature Test with Database Assertion**

**Code**

```php
<?php
// tests/Feature/PostCreationTest.php

use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;

uses(RefreshDatabase::class);

test('a post can be created and persisted', function () {
    $user = User::factory()->create();

    $response = $this->actingAs($user)->post('/posts', [
        'title' => 'New Post',
        'body'  => 'Content',
    ]);

    $response->assertRedirect('/posts');
    $this->assertDatabaseHas('posts', [
        'title'   => 'New Post',
        'user_id' => $user->id,
    ]);
});
```

**Expected Output**

```
PASS  Tests\Feature\PostCreationTest
✓ a post can be created and persisted
```

**Why This Output Occurs**

The test authenticates as a user, POSTs valid data, asserts the redirect, and verifies the database record exists with the correct foreign key.

---

#### Real-World Cases with Explanation

**Case 1: Blog Post Creation Workflow**

A blog application needs to ensure that authenticated users can create posts, guests are redirected to login, and validation errors are returned for missing fields. Feature tests cover all three scenarios: one test for the successful creation, one for the guest redirect, and one for the validation error. This single test file documents and enforces the complete contract of the `POST /posts` endpoint.

**Explanation**: Feature tests exercise the entire request lifecycle, catching bugs that unit tests cannot—such as a missing middleware, an incorrect route definition, or a validation rule that was accidentally removed.

---

### 2. HTTP Testing

#### Definitions

**Core Definition**

HTTP testing is the set of Laravel helpers and assertions that allow feature tests to simulate HTTP requests and inspect the resulting responses.

**Technical Definition**

Laravel’s HTTP testing helpers (`$this->get()`, `$this->post()`, `$this->postJson()`, etc.) simulate requests internally without issuing real network calls. They return an `Illuminate\Testing\TestResponse` instance, which provides fluent assertion methods for status codes, headers, cookies, session data, JSON structure, redirects, and views. CSRF middleware is automatically disabled during tests.

**Beginner-Friendly Explanation**

HTTP testing helpers are the “pretend browser” in your test. You say “pretend I sent a POST request with this data,” and Laravel runs that request through the real application exactly as if it came from a browser or API client.

#### Purposes

- To simulate HTTP requests without a live server.
- To inspect every part of the response (status, headers, body, JSON, session).
- To test the full request lifecycle, including middleware and validation.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
$response = $this->get('/uri');
$response = $this->post('/uri', $data);
$response = $this->put('/uri', $data);
$response = $this->patch('/uri', $data);
$response = $this->delete('/uri');
$response = $this->postJson('/api/uri', $data);
$response = $this->withHeaders(['X-Header' => 'Value'])->post('/uri', $data);
```

**Component Breakdown**

| Method | Description |
|--------|-------------|
| `get()` | Simulates a GET request. |
| `post()` | Simulates a POST request with form data. |
| `postJson()` | Simulates a POST request with JSON payload. |
| `withHeaders()` | Sets custom request headers before sending. |
| `withCookie()` | Sets a cookie before sending. |

**Constraints and Limitations**

- Only one request per test is recommended; multiple requests can produce unexpected behaviour.
- `postJson()` should be used for API endpoints expecting JSON; `post()` is for form submissions.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: GET Request**

**Code**

```php
test('user profile page loads', function () {
    $user = User::factory()->create();

    $response = $this->actingAs($user)->get("/users/{$user->id}");

    $response->assertOk();
    $response->assertSee($user->name);
});
```

**Expected Output**

```
PASS  Tests\Feature\ProfileTest
✓ user profile page loads
```

**Why This Output Occurs**

The `get()` helper sends a GET request to the user’s profile URI. `assertOk()` verifies a 200 status, and `assertSee()` verifies the user’s name appears in the HTML.

---

**Example 2: POST with Custom Headers**

**Code**

```php
test('api request with custom header', function () {
    $response = $this->withHeaders([
        'X-API-Key' => 'secret',
    ])->postJson('/api/data', ['key' => 'value']);

    $response->assertStatus(201);
});
```

**Expected Output**

```
PASS  Tests\Feature\ApiTest
✓ api request with custom header
```

**Why This Output Occurs**

`withHeaders()` adds the custom header to the request before `postJson()` sends it. The application can then act on the header, and the assertion verifies the response status.

---

#### Real-World Cases with Explanation

**Case 1: API Authentication Header**

An API requires an `Authorization: Bearer` header. A feature test uses `withHeaders(['Authorization' => 'Bearer token'])` to simulate an authenticated request and asserts the response status. This verifies that the authentication middleware correctly processes the header.

**Explanation**: HTTP testing helpers allow precise control over the request, including headers, cookies, and body format.

---

### 3. Route Testing

#### Definitions

**Core Definition**

Route testing verifies that specific URIs respond to the correct HTTP verbs, return the expected status codes, and are protected by the correct middleware and authorization.

**Technical Definition**

Route tests are a subset of feature tests that focus on the route layer: the URI, HTTP method, middleware stack, and route model binding. They often assert status codes, redirects, and the presence of specific middleware (e.g., `auth`, `verified`, `throttle`). Route testing is essential for catching misconfigured routes, missing middleware, and 404/405 responses.

**Beginner-Friendly Explanation**

Route testing checks that your URLs are wired up correctly. “Does `GET /posts` actually exist? Does it return a 200 for logged-in users and a redirect for guests?”

#### Purposes

- To verify that routes are defined with the correct URI and HTTP verb.
- To ensure that middleware (auth, verified, throttle) is applied.
- To catch 404 and 405 errors before users encounter them.
- To test route model binding resolution.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
test('route requires authentication', function () {
    $this->get('/dashboard')->assertRedirect('/login');
});

test('route returns 404 for missing model', function () {
    $this->actingAs(User::factory()->create())
         ->get('/posts/999')
         ->assertNotFound();
});
```

**Component Breakdown**

| Assertion | Verifies |
|-----------|----------|
| `assertRedirect('/login')` | Route redirects unauthenticated users. |
| `assertNotFound()` | Route returns 404 for missing resources. |
| `assertForbidden()` | Route returns 403 for unauthorized users. |
| `assertMethodNotAllowed()` | Route returns 405 for incorrect HTTP verbs. |

**Constraints and Limitations**

- Route tests do not verify the business logic behind the route; feature and unit tests cover that.
- Route caching can cause stale routes in tests; run `php artisan route:clear` if needed.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Guest Redirect**

**Code**

```php
test('guests are redirected from dashboard', function () {
    $this->get('/dashboard')->assertRedirect('/login');
});
```

**Expected Output**

```
PASS  Tests\Feature\RouteTest
✓ guests are redirected from dashboard
```

**Why This Output Occurs**

The `/dashboard` route is protected by the `auth` middleware. An unauthenticated request is redirected to `/login`.

---

**Example 2: 404 for Missing Model**

**Code**

```php
test('missing post returns 404', function () {
    $user = User::factory()->create();

    $this->actingAs($user)->get('/posts/999')->assertNotFound();
});
```

**Expected Output**

```
PASS  Tests\Feature\RouteTest
✓ missing post returns 404
```

**Why This Output Occurs**

Route model binding attempts to resolve the post with ID 999. Since no such record exists, the framework returns a 404.

---

#### Real-World Cases with Explanation

**Case 1: Admin Route Protection**

An application has an `/admin` prefix protected by `auth` and `can:admin` middleware. Route tests verify that guests are redirected, non-admin users receive a 403, and admin users receive a 200. This ensures the middleware stack is correctly applied.

**Explanation**: Route tests catch misconfigured middleware before they become security vulnerabilities.

---

### 4. Controller Testing

#### Definitions

**Core Definition**

Controller testing verifies that a controller action receives the correct input, interacts with the correct models and services, and returns the expected response (view, redirect, or JSON).

**Technical Definition**

Controller tests are feature tests that target a specific controller method. They assert that the action produces the correct HTTP status, view, redirect, or JSON payload, and that it persists the correct data. Controllers should be kept thin; complex business logic should be delegated to services or actions, which are tested separately.

**Beginner-Friendly Explanation**

Controller testing checks that the “traffic cop” of your application—the controller—routes the request to the right place and returns the right response.

#### Purposes

- To verify that controller actions return the correct response type (view, redirect, JSON).
- To ensure that the controller persists the correct data.
- To catch missing validation or authorization logic in the controller.
- To test the integration between the controller and its dependencies.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
test('store action creates a post', function () {
    $user = User::factory()->create();

    $response = $this->actingAs($user)->post('/posts', [
        'title' => 'Test Post',
        'body'  => 'Content',
    ]);

    $response->assertRedirect('/posts');
    $this->assertDatabaseHas('posts', ['title' => 'Test Post']);
});
```

**Component Breakdown**

| Assertion | Verifies |
|-----------|----------|
| `assertRedirect()` | Controller returned a redirect. |
| `assertViewIs()` | Controller returned a specific view. |
| `assertJson()` | Controller returned a JSON payload. |
| `assertDatabaseHas()` | Controller persisted the correct data. |

**Constraints and Limitations**

- Controllers should be thin; if a controller test requires mocking many services, the controller may be doing too much.
- Testing private controller methods is not possible; test them through public actions.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Index Action Returns a View**

**Code**

```php
test('index action returns the posts index view', function () {
    $user = User::factory()->create();

    $response = $this->actingAs($user)->get('/posts');

    $response->assertOk();
    $response->assertViewIs('posts.index');
});
```

**Expected Output**

```
PASS  Tests\Feature\PostControllerTest
✓ index action returns the posts index view
```

**Why This Output Occurs**

The `index` action returns `view('posts.index')`. The test asserts that the response is a 200 and that the correct view was rendered.

---

**Example 2: Store Action Persists Data**

**Code**

```php
test('store action persists a post', function () {
    $user = User::factory()->create();

    $this->actingAs($user)->post('/posts', [
        'title' => 'Persisted Post',
        'body'  => 'Body',
    ])->assertRedirect('/posts');

    $this->assertDatabaseHas('posts', [
        'title'   => 'Persisted Post',
        'user_id' => $user->id,
    ]);
});
```

**Expected Output**

```
PASS  Tests\Feature\PostControllerTest
✓ store action persists a post
```

**Why This Output Occurs**

The `store` action creates a post and redirects. The test verifies both the redirect and the database record.

---

#### Real-World Cases with Explanation

**Case 1: API Resource Controller**

A `PostController` handles `index`, `store`, `show`, `update`, and `destroy`. Controller tests verify each action returns the correct JSON status (200, 201, 204) and that the database reflects the expected changes.

**Explanation**: Controller tests document the API contract and catch regressions when controller logic changes.

---

### 5. Middleware Testing

#### Definitions

**Core Definition**

Middleware testing verifies that HTTP middleware correctly allows, blocks, or modifies requests based on the conditions it enforces (authentication, authorization, rate limiting, etc.).

**Technical Definition**

Middleware tests are feature tests that send requests through routes protected by middleware and assert on the response. They verify that the middleware’s `handle` method produces the correct outcome: a redirect for unauthenticated users, a 403 for unauthorized users, a 429 for rate-limited users, or a successful response when the condition is met. Because middleware runs as part of the full HTTP kernel, feature tests are the natural way to test it.

**Beginner-Friendly Explanation**

Middleware is the “bouncer” at the door of your routes. Middleware testing checks that the bouncer correctly lets the right people in and turns the wrong people away.

#### Purposes

- To verify that authentication middleware redirects guests and allows authenticated users.
- To verify that authorization middleware returns 403 for unauthorized users.
- To verify that rate-limiting middleware returns 429 after too many requests.
- To catch middleware that is accidentally missing from a route.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
test('auth middleware redirects guests', function () {
    $this->get('/dashboard')->assertRedirect('/login');
});

test('auth middleware allows authenticated users', function () {
    $user = User::factory()->create();
    $this->actingAs($user)->get('/dashboard')->assertOk();
});
```

**Component Breakdown**

| Scenario | Expected Response |
|----------|-------------------|
| Guest hits `auth` route | Redirect to `/login`. |
| Authenticated user hits `auth` route | 200 OK. |
| Unauthorized user hits `can` route | 403 Forbidden. |
| Too many requests | 429 Too Many Requests. |

**Constraints and Limitations**

- Middleware tests cannot isolate the middleware from the rest of the stack; they test the middleware as part of the full request.
- Custom middleware with complex logic may benefit from dedicated unit tests in addition to feature tests.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Auth Middleware**

**Code**

```php
test('guests are redirected by auth middleware', function () {
    $this->get('/dashboard')->assertRedirect('/login');
});

test('authenticated users pass auth middleware', function () {
    $user = User::factory()->create();
    $this->actingAs($user)->get('/dashboard')->assertOk();
});
```

**Expected Output**

```
PASS  Tests\Feature\MiddlewareTest
✓ guests are redirected by auth middleware
✓ authenticated users pass auth middleware
```

**Why This Output Occurs**

The `auth` middleware redirects unauthenticated requests and allows authenticated requests to proceed. The tests verify both branches of the middleware’s behaviour.

---

**Example 2: Authorization Middleware**

**Code**

```php
test('non-admin users are forbidden by admin middleware', function () {
    $user = User::factory()->create(['role' => 'user']);
    $this->actingAs($user)->get('/admin')->assertForbidden();
});

test('admin users pass admin middleware', function () {
    $admin = User::factory()->create(['role' => 'admin']);
    $this->actingAs($admin)->get('/admin')->assertOk();
});
```

**Expected Output**

```
PASS  Tests\Feature\MiddlewareTest
✓ non-admin users are forbidden by admin middleware
✓ admin users pass admin middleware
```

**Why This Output Occurs**

The admin middleware checks the user’s role. Non-admin users receive a 403, while admin users receive a 200.

---

#### Real-World Cases with Explanation

**Case 1: Tenant Isolation Middleware**

A multi-tenant application uses middleware to ensure that users can only access resources belonging to their tenant. Middleware tests verify that a user from Tenant A cannot access Tenant B’s resources (expecting a 404 rather than a 403 to avoid leaking the existence of the record).

**Explanation**: Middleware tests are critical for security-sensitive cross-tenant isolation.

---

### 6. Authentication Testing

#### Definitions

**Core Definition**

Authentication testing verifies that the application correctly identifies users, redirects guests, and enforces authentication requirements on protected routes.

**Technical Definition**

Authentication tests use `actingAs($user)` for session-based (web) guards and `Sanctum::actingAs($user, $abilities)` for token-based API guards. They assert that authenticated users receive successful responses, guests receive redirects or 401 responses, and that the correct guard is being used. Manually posting to the login endpoint in every test is discouraged; `actingAs()` is faster and more reliable.

**Beginner-Friendly Explanation**

Authentication testing checks that your app knows who is logged in. “If I am logged in as Alice, can I see Alice’s dashboard? If I am not logged in, am I redirected to the login page?”

#### Purposes

- To verify that protected routes are inaccessible to guests.
- To ensure that authenticated users can access their own resources.
- To test both web (session) and API (token) authentication paths.
- To catch misconfigured guards or missing authentication middleware.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Session-based authentication
$this->actingAs($user)->get('/dashboard')->assertOk();

// Multiple guards
$this->actingAs($user, 'admin')->get('/admin')->assertOk();

// Sanctum token authentication with abilities
Sanctum::actingAs($user, ['posts:write']);
$this->postJson('/api/posts', $data)->assertCreated();
```

**Component Breakdown**

| Method | Use Case |
|--------|----------|
| `actingAs($user)` | Authenticates for the default guard. |
| `actingAs($user, $guard)` | Authenticates for a specific guard. |
| `Sanctum::actingAs($user, $abilities)` | Authenticates an API token with scoped abilities. |

**Constraints and Limitations**

- `actingAs()` does not test the login process itself; it bypasses the login endpoint.
- Token abilities must be explicitly provided when testing scoped API endpoints.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Web Authentication**

**Code**

```php
test('authenticated user can view dashboard', function () {
    $user = User::factory()->create();

    $this->actingAs($user)
         ->get('/dashboard')
         ->assertOk()
         ->assertSee($user->name);
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

`actingAs($user)` sets the authenticated user for the request. The dashboard route, protected by `auth`, returns a 200 for the authenticated user and a redirect for the guest.

---

**Example 2: Sanctum Token with Abilities**

**Code**

```php
use Laravel\Sanctum\Sanctum;

test('token with write ability can create a post', function () {
    $user = User::factory()->create();
    Sanctum::actingAs($user, ['posts:write']);

    $this->postJson('/api/posts', ['title' => 'API Post'])
         ->assertCreated();
});

test('token without write ability is forbidden', function () {
    $user = User::factory()->create();
    Sanctum::actingAs($user, ['posts:read']);

    $this->postJson('/api/posts', ['title' => 'API Post'])
         ->assertForbidden();
});
```

**Expected Output**

```
PASS  Tests\Feature\ApiAuthTest
✓ token with write ability can create a post
✓ token without write ability is forbidden
```

**Why This Output Occurs**

`Sanctum::actingAs()` authenticates the request with the specified abilities. The API endpoint checks for the `posts:write` ability; the first test passes, the second receives a 403.

---

#### Real-World Cases with Explanation

**Case 1: Multi-Guard Application**

An application has separate guards for `web` (regular users) and `admin` (administrators). Authentication tests verify that regular users cannot access admin routes and that admin users can access both regular and admin routes. Each guard is tested independently with `actingAs($user, 'guard')`.

**Explanation**: Multi-guard authentication testing ensures that each user type is correctly scoped to its intended routes.

---

### 7. Authorization Testing

#### Definitions

**Core Definition**

Authorization testing verifies that authenticated users can only perform actions and access resources they are permitted to, according to policies, gates, and middleware.

**Technical Definition**

Authorization tests assert that the correct HTTP status (200, 201, 403, 404) is returned for users with different permissions. They test the complete permission matrix: the owner of a resource can update it, a different user cannot, an admin can delete it, and a guest is redirected. Authorization should be tested both at the policy level (directly against the gate/policy) and at the HTTP level (to confirm that the endpoint actually invokes the authorization check).

**Beginner-Friendly Explanation**

Authorization testing checks that users can only do what they are allowed to do. “Can Alice edit her own post? Can Bob edit Alice’s post? (He should get a 403.)”

#### Purposes

- To prevent privilege escalation and cross-user data leaks.
- To verify that policies and gates are correctly wired into controllers and routes.
- To test the complete permission matrix for each role.
- To catch missing authorization checks in endpoints.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
test('user can update own post', function () {
    $user = User::factory()->create();
    $post = Post::factory()->for($user)->create();

    $this->actingAs($user)
         ->put("/posts/{$post->id}", ['title' => 'Updated'])
         ->assertRedirect();
});

test('user cannot update another users post', function () {
    $user  = User::factory()->create();
    $other = User::factory()->create();
    $post  = Post::factory()->for($other)->create();

    $this->actingAs($user)
         ->put("/posts/{$post->id}", ['title' => 'Hacked'])
         ->assertForbidden();
});
```

**Component Breakdown**

| Assertion | Verifies |
|-----------|----------|
| `assertOk()` | User is authorized. |
| `assertForbidden()` | User is authenticated but not authorized (403). |
| `assertNotFound()` | Resource is hidden (404) to avoid leaking existence. |

**Constraints and Limitations**

- A 403 does not reveal *which* mechanism refused the request (middleware, policy, or `abort()`); policy-level tests identify the exact rule.
- For tenant isolation, prefer 404 over 403 to avoid confirming the existence of another tenant’s record.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Owner vs. Non-Owner**

**Code**

```php
test('owner can delete their post', function () {
    $user = User::factory()->create();
    $post = Post::factory()->for($user)->create();

    $this->actingAs($user)
         ->delete("/posts/{$post->id}")
         ->assertRedirect('/posts');

    $this->assertDatabaseMissing('posts', ['id' => $post->id]);
});

test('non-owner cannot delete the post', function () {
    $user  = User::factory()->create();
    $other = User::factory()->create();
    $post  = Post::factory()->for($other)->create();

    $this->actingAs($user)
         ->delete("/posts/{$post->id}")
         ->assertForbidden();
});
```

**Expected Output**

```
PASS  Tests\Feature\AuthorizationTest
✓ owner can delete their post
✓ non-owner cannot delete the post
```

**Why This Output Occurs**

The `PostPolicy::delete` method checks that the authenticated user is the post’s author. The owner is allowed (redirect + database missing), while the non-owner receives a 403.

---

**Example 2: Admin Override**

**Code**

```php
test('admin can delete any post', function () {
    $admin = User::factory()->admin()->create();
    $post  = Post::factory()->for(User::factory())->create();

    $this->actingAs($admin)
         ->delete("/posts/{$post->id}")
         ->assertRedirect('/posts');

    $this->assertDatabaseMissing('posts', ['id' => $post->id]);
});
```

**Expected Output**

```
PASS  Tests\Feature\AuthorizationTest
✓ admin can delete any post
```

**Why This Output Occurs**

The policy allows admins to bypass the ownership check, so the admin can delete the post.

---

#### Real-World Cases with Explanation

**Case 1: Multi-Tenant Data Isolation**

A multi-tenant application must ensure that users from Tenant A cannot access Tenant B’s orders. Authorization tests create orders for both tenants and verify that a user from Tenant A receives a 404 (not a 403) when accessing Tenant B’s order, preventing the leak of the record’s existence.

**Explanation**: Authorization testing is the primary defense against cross-tenant data leaks and privilege escalation.

---

### 8. Validation Testing

#### Definitions

**Core Definition**

Validation testing verifies that incoming request data is correctly validated, that invalid data produces the expected error response, and that valid data passes through to the controller.

**Technical Definition**

Validation tests submit data that violates Form Request rules and assert that the response is a redirect (for web) or a 422 (for JSON) with the correct validation errors. `assertSessionHasErrors()` verifies that specific fields have errors in the session, while `assertJsonValidationErrors()` verifies the JSON error structure for API endpoints. Each validation rule should have at least one test, ideally using datasets to cover multiple invalid inputs efficiently.

**Beginner-Friendly Explanation**

Validation testing checks that your app rejects bad data. “If I submit a form without an email, do I get an error message telling me the email is required?”

#### Purposes

- To verify that every validation rule is correctly applied.
- To ensure that invalid data produces the correct error response (redirect + session errors for web, 422 + JSON errors for API).
- To catch validation rules that are accidentally removed or weakened.
- To document the input contract of every endpoint.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Web validation
$this->post('/posts', [])->assertSessionHasErrors(['title', 'body']);

// Specific error message
$this->post('/posts', ['title' => ''])
     ->assertSessionHasErrors(['title' => 'The title field is required.']);

// API validation
$this->postJson('/api/posts', [])
     ->assertStatus(422)
     ->assertJsonValidationErrors(['title']);
```

**Component Breakdown**

| Assertion | Use Case |
|-----------|----------|
| `assertSessionHasErrors()` | Web form validation errors. |
| `assertSessionHasNoErrors()` | Valid data passes. |
| `assertJsonValidationErrors()` | API validation errors (422). |

**Constraints and Limitations**

- Validation tests should use datasets to avoid repeating the same test for every rule.
- Moving validation to Form Requests makes testing easier; assert errors from the request class.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Required Field Validation**

**Code**

```php
test('title is required', function () {
    $user = User::factory()->create();

    $this->actingAs($user)
         ->post('/posts', ['body' => 'Content'])
         ->assertSessionHasErrors('title');
});
```

**Expected Output**

```
PASS  Tests\Feature\ValidationTest
✓ title is required
```

**Why This Output Occurs**

The `title` field is missing from the request. The Form Request rule `required` fails, and the validation error is flashed to the session. `assertSessionHasErrors('title')` confirms the error exists.

---

**Example 2: API Validation with 422**

**Code**

```php
test('api returns 422 for invalid email', function () {
    $this->postJson('/api/users', ['email' => 'not-an-email'])
         ->assertStatus(422)
         ->assertJsonValidationErrors(['email']);
});
```

**Expected Output**

```
PASS  Tests\Feature\ValidationTest
✓ api returns 422 for invalid email
```

**Why This Output Occurs**

The JSON request contains an invalid email. The validation fails, and the API returns a 422 response with a JSON error structure. `assertJsonValidationErrors` confirms the `email` field is in the error payload.

---

#### Real-World Cases with Explanation

**Case 1: Registration Form Validation**

A registration form requires name, email, password, and password confirmation. Validation tests use a dataset to submit each invalid combination (missing name, invalid email, short password, mismatched confirmation) and assert that the correct field has an error. This ensures every rule is enforced.

**Explanation**: Validation testing with datasets efficiently covers the entire input domain and catches accidentally weakened rules.

---

### 9. Response Testing

#### Definitions

**Core Definition**

Response testing verifies the shape and content of the HTTP response: status code, headers, body, JSON structure, redirects, and session data.

**Technical Definition**

`TestResponse` provides a comprehensive set of assertion methods. For status codes, use `assertStatus()`, `assertOk()`, `assertCreated()`, `assertNoContent()`, `assertNotFound()`, `assertForbidden()`, `assertUnauthorized()`, and `assertUnprocessable()`. For JSON, use `assertJson()`, `assertJsonPath()`, `assertJsonStructure()`, `assertJsonCount()`, and the fluent `AssertableJson` API. For redirects, use `assertRedirect()`. For headers, use `assertHeader()`. Always assert the specific expected status code; `assertOk()` alone can hide a 201 or 422 regression.

**Beginner-Friendly Explanation**

Response testing checks that the response your app sends back is exactly what you expect—right status code, right JSON keys, right redirect location.

#### Purposes

- To verify that the API or web endpoint returns the correct status code.
- To assert that the JSON payload contains the expected keys and values.
- To verify redirects and session flash messages.
- To catch response-shape regressions before they break frontend clients.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
$response->assertStatus(200);
$response->assertOk();
$response->assertCreated();
$response->assertNoContent();
$response->assertRedirect('/posts');
$response->assertJson(['key' => 'value']);
$response->assertJsonPath('data.author.name', 'Alice');
$response->assertJsonStructure(['data' => ['id', 'title']]);
$response->assertJsonValidationErrors(['email']);
$response->assertHeader('Content-Type', 'application/json');
```

**Component Breakdown**

| Assertion | Use Case |
|-----------|----------|
| `assertStatus(int)` | Exact status code. |
| `assertOk()` | 200 OK. |
| `assertCreated()` | 201 Created. |
| `assertNoContent()` | 204 No Content. |
| `assertRedirect(url)` | Redirect to a specific URI. |
| `assertJsonPath(key, value)` | Specific JSON path value. |
| `assertJsonStructure(array)` | JSON key structure exists. |
| `assertJsonValidationErrors(array)` | Validation error fields. |

**Constraints and Limitations**

- `assertJson()` checks a subset; use `assertExactJson()` for exact matches (but this is fragile and generally discouraged).
- Fluent `AssertableJson` with `->etc()` allows flexibility while asserting required structure.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Exact Status and JSON Structure**

**Code**

```php
test('api returns 201 with correct structure', function () {
    $user = User::factory()->create();

    $response = $this->actingAs($user)->postJson('/api/posts', [
        'title' => 'Hello',
        'body'  => 'World',
    ]);

    $response->assertStatus(201)
             ->assertJsonStructure([
                 'data' => ['id', 'title', 'body', 'created_at'],
             ]);
});
```

**Expected Output**

```
PASS  Tests\Feature\ResponseTest
✓ api returns 201 with correct structure
```

**Why This Output Occurs**

The API returns a 201 with a JSON payload containing the specified keys. `assertJsonStructure` verifies that the structure matches, without being brittle about exact values.

---

**Example 2: Fluent JSON Assertions**

**Code**

```php
use Illuminate\Testing\Fluent\AssertableJson;

test('orders list has correct shape', function () {
    $user = User::factory()->create();
    Order::factory()->count(3)->for($user)->create();

    $this->actingAs($user)
         ->getJson('/api/orders')
         ->assertOk()
         ->assertJson(fn (AssertableJson $json) =>
             $json->has('data', 3)
                  ->has('data.0', fn (AssertableJson $item) =>
                      $item->hasAll(['id', 'amount_cents', 'status', 'created_at'])
                           ->whereType('id', 'integer')
                           ->whereType('amount_cents', 'integer')
                           ->etc()
                  )
                  ->has('links')
                  ->has('meta')
         );
});
```

**Expected Output**

```
PASS  Tests\Feature\ResponseTest
✓ orders list has correct shape
```

**Why This Output Occurs**

The fluent `AssertableJson` API asserts that the payload has a `data` array with 3 items, that each item has the required keys with the correct types, and that `links` and `meta` are present. The `->etc()` allows additional fields without breaking the test.

---

#### Real-World Cases with Explanation

**Case 1: API Contract Testing**

A frontend team depends on the JSON structure of an API. Response tests use `assertJsonStructure()` to verify that every endpoint returns the expected keys. When a backend developer accidentally renames `data` to `items`, the response test fails immediately, preventing a broken frontend release.

**Explanation**: Response tests act as a contract between backend and frontend, catching breaking changes before deployment.

---

### 10. Feature Testing Best Practices

#### Definitions

**Core Definition**

Feature testing best practices are the guidelines for writing feature tests that are reliable, fast, readable, and genuinely useful for catching regressions.

**Technical Definition**

Best practices encompass: making one request per test, using `RefreshDatabase` for isolation, authenticating with `actingAs()` instead of manually posting to login endpoints, asserting specific status codes rather than generic ones, testing the authorization matrix (both success and failure), using datasets for validation rules, structuring tests with Arrange–Act–Assert, and using factories for test data. Feature tests should test through the “front door” (HTTP) rather than calling controller methods directly, so they exercise routing, middleware, validation, and authorization together.

**Beginner-Friendly Explanation**

Best practices are the habits that make your feature tests helpful instead of flaky. They ensure each test checks one thing, runs fast, and gives a clear error when something breaks.

#### Key Best Practices

1. **One Request per Test**: Each test should make exactly one HTTP request to the application; multiple requests in one test can produce unexpected behaviour.

2. **Always Use `RefreshDatabase`**: Wrap each test in a transaction that is rolled back, ensuring test isolation and speed.

3. **Authenticate with `actingAs()`, Not Login Requests**: `actingAs($user)` is faster and more reliable than posting credentials to the login endpoint in every test.

4. **Assert Specific Status Codes**: Use `assertStatus(201)` or `assertStatus(422)` instead of `assertOk()` when a specific code is expected; `assertOk()` can hide a regression from 201 to 200.

5. **Test the Authorization Matrix**: For every protected endpoint, test that the correct user succeeds **and** the wrong user is refused. The refusal test is what prevents cross-user data leaks.

6. **Test Validation Errors**: Submit invalid data and assert `assertSessionHasErrors()` (web) or `assertJsonValidationErrors()` (API) for each field.

7. **Use Datasets for Validation Rules**: Pest datasets allow the same validation test to run with multiple invalid inputs without duplication.

8. **Structure Tests with Arrange–Act–Assert**: Clearly separate setup, the HTTP request, and the assertions.

9. **Use Factories for Test Data**: Factories produce realistic, maintainable data and support states (e.g., `admin()`) for role-based testing.

10. **Assert Both Response and Database State**: After a successful mutation, assert both the HTTP response (redirect or JSON) and the persisted database record.

11. **Prefer Fluent JSON Assertions for APIs**: Use `AssertableJson` with `->has()`, `->whereType()`, and `->etc()` to assert structure without being brittle about exact values.

12. **Test Through the Front Door**: Send requests via HTTP helpers rather than calling controller methods directly, so the full middleware, routing, and validation stack is exercised.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Best Practices in a Single Test**

**Code**

```php
<?php
// tests/Feature/PostTest.php

use App\Models\Post;
use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;

uses(RefreshDatabase::class);

test('authenticated user can create a post', function () {
    // Arrange
    $user = User::factory()->create();

    // Act
    $response = $this->actingAs($user)->post('/posts', [
        'title' => 'My Post',
        'body'  => 'Content',
    ]);

    // Assert: response
    $response->assertStatus(302)
             ->assertRedirect('/posts');

    // Assert: database state
    $this->assertDatabaseHas('posts', [
        'title'   => 'My Post',
        'user_id' => $user->id,
    ]);
});
```

**Expected Output**

```
PASS  Tests\Feature\PostTest
✓ authenticated user can create a post
```

**Why This Output Occurs**

The test follows every best practice: one request, `RefreshDatabase`, `actingAs()`, specific status assertion (`assertStatus(302)`), and both response and database assertions.

---

**Example 2: Authorization Matrix with Datasets**

**Code**

```php
<?php

use App\Models\Post;
use App\Models\User;

test('post update respects ownership', function (string $role, bool $canUpdate) {
    $owner = User::factory()->create();
    $actor = $role === 'owner'
        ? $owner
        : User::factory()->create(['role' => $role]);

    $post = Post::factory()->for($owner)->create();

    $response = $this->actingAs($actor)
                     ->put("/posts/{$post->id}", ['title' => 'Updated']);

    $canUpdate
        ? $response->assertRedirect()
        : $response->assertForbidden();
})->with([
    'owner' => ['owner', true],
    'other user' => ['user', false],
    'admin' => ['admin', true],
]);
```

**Expected Output**

```
PASS  Tests\Feature\PostAuthorizationTest
✓ post update respects ownership with data set "owner"
✓ post update respects ownership with data set "other user"
✓ post update respects ownership with data set "admin"
```

**Why This Output Occurs**

The dataset runs the same authorization test for three roles: the owner (allowed), another user (forbidden), and an admin (allowed). This efficiently covers the complete permission matrix without duplicating test code.

---

#### Real-World Cases with Explanation

**Case 1: CI Pipeline with Fast, Reliable Feature Tests**

A team runs `php artisan test` in GitHub Actions on every push. Feature tests use `RefreshDatabase` with SQLite in-memory, authenticate with `actingAs()`, and assert specific status codes. The suite runs in under 30 seconds and catches integration regressions before they reach staging. The authorization matrix tests have prevented two potential cross-tenant data leaks in the past year.

**Explanation**: Following feature testing best practices results in a fast, reliable suite that provides real confidence and catches costly security bugs.

---

## References

- Laravel Testing with TDD — affaan-m/ECC — https://github.com/affaan-m/ECC/blob/main/skills/laravel-tdd/SKILL.md
- Test My Laravel App: Feature Tests, Pest, PHPUnit — RAITHub — https://www.raithub.com/blog/test-my-laravel-app
- HTTP Feature Tests and Fluent JSON Assertions — voku/agent-skills — https://github.com/voku/agent-skills/blob/main/skills/laravel-testing/rules/http-feature-testing.md
- HTTP Tests — Laravel Documentation (raw source) — https://github.com/kimchanhyung98/laravel-docs-source/blob/53e241bfe00992aeeb7b494f83a15adc7688157c/11.x/origin/http-tests.md
- Authentication Testing: Web Guards and Sanctum Tokens — voku/agent-skills — https://github.com/voku/agent-skills/blob/main/skills/laravel-testing/rules/auth-testing.md
- Endpoint Tests — coollabsio/coolify — https://github.com/coollabsio/coolify/blob/3b33e8a4ab753308dde26282ebe057e578d561d2/.claude/skills/testing-best-practices/rules/endpoint-tests.md
- Feature Test Template — fusengine/agents — https://github.com/fusengine/agents/blob/main/plugins/laravel-expert/skills/laravel-testing/references/templates/FeatureTest.php.md
- HTTP Response Assertions — turkeryildirim/ai-skills — https://github.com/turkeryildirim/ai-skills/blob/main/laravel/rules/http-assert-response.md
- HTTP Tests — Laravel 12.x Documentation — https://laravel.com/docs/12.x/http-tests
- Validation Testing — fusengine/agents — https://github.com/fusengine/agents/blob/main/plugins/laravel-expert/skills/laravel-testing/references/templates/FeatureTest.php.md
- Testing Authorization in Laravel — CodeCraft Diary (Medium) — https://medium.com/@codecraftdiary/testing-authorization-in-laravel-policies-gates-and-middleware

---

**Version Note**: This cheat sheet is based on Laravel 10.x through 13.x documentation. `actingAs()`, `RefreshDatabase`, and the `TestResponse` assertion API have remained stable across these versions. `Sanctum::actingAs()` is the recommended method for API token authentication testing. `assertJsonValidationErrors()` and `assertSessionHasErrors()` are available in all recent versions. The fluent `AssertableJson` API was introduced in Laravel 8. Developers should always consult the documentation for their specific Laravel version.