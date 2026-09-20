# Laravel HTTP Requests & Input Lifecycle: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Laravel's HTTP Request system provides an object-oriented interface for interacting with the current HTTP request, encapsulating all incoming data—query strings, form inputs, JSON payloads, headers, cookies, files, and server metadata—into a single `Illuminate\Http\Request` instance that flows through the application's lifecycle.

**Technical Definition:** The `Illuminate\Http\Request` class extends `Symfony\Component\HttpFoundation\Request` and is bound into Laravel's service container as a singleton for the duration of each request. It implements `ArrayAccess` and `Illuminate\Contracts\Support\Arrayable`, allowing array-style access to input data. The request lifecycle begins at `public/index.php`, passes through the HTTP kernel's middleware stack, and is resolved by the router into a controller or closure action. Throughout this lifecycle, the request instance accumulates input data (from query strings, form bodies, and JSON payloads), files, headers, and server parameters, all accessible through a fluent, expressive API.

**Beginner-Friendly Explanation:** Every time someone visits your Laravel application, the framework packages everything about that visit—what URL they requested, what data they submitted, what browser they're using, their IP address—into a single "request" object. Your code can then ask this object questions like "What's the value of the `name` field?" or "Is this a POST request?" Instead of dealing with raw PHP superglobals like `$_GET` and `$_POST`, you get a clean, consistent, and testable interface.

### Key Characteristics

- **Object-Oriented Wrapper:** Encapsulates all HTTP request data in a single, injectable object.
- **Service Container Integration:** Automatically injected into controllers and closures via type-hinting.
- **Multiple Data Sources:** Handles query strings, form data, JSON payloads, files, cookies, headers, and server parameters.
- **Auto-Casting:** Convenience methods like `boolean()`, `date()`, `enum()`, and `integer()` automatically cast input values to the appropriate PHP type.
- **Immutable by Default:** The request instance is a singleton per request; its input data can be inspected without side effects (though `merge()` and `replace()` allow controlled mutation).
- **PSR-7 Compatible:** Can be converted to a PSR-7 request for interoperability with other libraries.

### Prerequisites

- **Laravel 10.x, 11.x, or 12.x** (the `Request` API is stable across versions; newer methods like `boolean()`, `date()`, `enum()`, and `mergeIfMissing()` are available from Laravel 8+ and 10+ respectively).
- **PHP 8.1+** (Laravel 10) or **PHP 8.2+** (Laravel 11/12).
- A Laravel application with routing and controller configuration.
- Basic understanding of HTTP methods, headers, and request/response cycles.

### Related Programming Areas

- **Routing:** The request is matched to routes based on path, method, and host.
- **Middleware:** The request passes through middleware before reaching the controller, where it can be inspected or modified.
- **Validation:** Form Requests and inline validation operate on the request's input data.
- **Controllers:** The request is the primary input to controller actions, providing access to all incoming data.
- **Session & Cookies:** The request carries session identifiers and cookies that Laravel decrypts and makes available.

### Core Concepts / Features

1. The Request Instance
2. Input Extraction Methods
3. Input Interception & Manipulation
4. State Verification
5. Metadata & Transport Tracing

---

## 1. The Request Instance

### Definitions

**Core Definition:** The Request instance is Laravel's object-oriented representation of the current HTTP request, providing methods to access input data, headers, files, and server metadata.

**Technical Definition:** `Illuminate\Http\Request` extends `Symfony\Component\HttpFoundation\Request` and is resolved from the service container as a singleton per request lifecycle. It encapsulates the query string (`$request->query`), request body (`$request->request`), server parameters (`$request->server`), headers (`$request->headers`), cookies (`$request->cookies`), and files (`$request->files`). The request is injected into controllers and closures via type-hinting, or retrieved via the global `request()` helper.

**Beginner-Friendly Explanation:** The Request instance is like a clipboard that contains everything about the visitor's request: their URL, their form data, their browser type, and their IP address. You don't have to go hunting for this information—Laravel hands you the clipboard, and you can read anything from it.

### Purposes

- To capture the current HTTP state using type-hinted parameters in controllers and closures.
- To provide a single, consistent interface for accessing all request data.
- To eliminate reliance on PHP superglobals (`$_GET`, `$_POST`, `$_SERVER`).
- To enable dependency injection and testability of request-dependent code.
- To provide a foundation for Laravel's validation, authorization, and middleware systems.

### Syntax Rules and Structure

**Complete General Syntax — Type-Hint Injection:**

```php
use Illuminate\Http\Request;

public function store(Request $request)
{
    $name = $request->input('name');
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Request $request` | Type-hint the `Illuminate\Http\Request` class. |
| Automatic resolution | The service container injects the current request instance. |

**Complete General Syntax — `request()` Helper:**

```php
$name = request()->input('name');
$name = request('name'); // Shorthand
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `request()` | Returns the current request instance (or a specific input value). |
| `request('name')` | Shorthand for `request()->input('name')`. |

**Complete General Syntax — Dependency Injection with Route Parameters:**

```php
Route::put('/user/{id}', [UserController::class, 'update']);

public function update(Request $request, string $id)
{
    // $request is injected; $id is the route parameter
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Request $request` | The injected request (comes first). |
| `string $id` | The route parameter (comes after dependencies). |

**Syntax Rules:**

- Route parameters must be listed **after** injected dependencies in the method signature.
- The `request()` helper returns the current request instance when called without arguments.
- The `request('key')` shorthand returns the value of the specified input key.
- The request instance is a singleton—calling `request()` multiple times returns the same instance.

**Constraints and Limitations:**

- **Route parameter order:** If route parameters are listed before dependencies, the container may attempt to resolve them as dependencies and fail.
- **Singleton scope:** The request instance is scoped to a single request lifecycle. It should not be stored in long-lived containers or static properties.
- **Immutability:** The request's input data should generally be treated as read-only. Use `merge()` or `replace()` for controlled mutation.

### Multiple Annotated Complete Code Examples

**Example 1: Type-Hint Injection in a Controller**

```php
<?php
// app/Http/Controllers/UserController.php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Http\RedirectResponse;

class UserController extends Controller
{
    /**
     * Store a new user.
     */
    public function store(Request $request): RedirectResponse
    {
        // Access input data via the injected request
        $name = $request->input('name');
        $email = $request->input('email');

        // Store the user...
        return redirect('/users');
    }
}
```

```php
<?php
// routes/web.php

use App\Http\Controllers\UserController;
use Illuminate\Support\Facades\Route;

Route::post('/users', [UserController::class, 'store']);
```

**Expected Output:** `POST /users` with `name=Alice&email=alice@example.com` stores the user and redirects to `/users`.

**Why:** The `Request` type-hint tells the service container to inject the current request instance. The `input()` method retrieves the specified field from the request body (or query string).

---

**Example 2: Request Injection in a Route Closure**

```php
<?php
// routes/web.php

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;

Route::get('/', function (Request $request) {
    return 'Path: ' . $request->path();
});
```

**Expected Output:** `GET /` returns `Path: /` (the root path is returned as `/` by `path()`).

**Why:** The `Request` type-hint on the closure parameter is resolved by the service container. The `path()` method returns the request's path information without the query string.

---

**Example 3: Dependency Injection with Route Parameters**

```php
<?php
// app/Http/Controllers/UserController.php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class UserController extends Controller
{
    /**
     * Update the specified user.
     */
    public function update(Request $request, string $id)
    {
        // $request is injected; $id is the route parameter
        $name = $request->input('name');

        return "Updating user {$id} with name {$name}";
    }
}
```

```php
<?php
// routes/web.php

use App\Http\Controllers\UserController;
use Illuminate\Support\Facades\Route;

Route::put('/user/{id}', [UserController::class, 'update']);
```

**Expected Output:** `PUT /user/5` with `name=Bob` returns `Updating user 5 with name Bob`.

**Why:** The `Request` dependency is resolved first, then the `{id}` route parameter is passed as the second argument. This order is required for proper resolution.

### Real-World Cases

- **Form Submissions:** Controllers receive the `Request` to access submitted form data.
- **API Endpoints:** The `Request` provides access to JSON payloads and headers for API consumers.
- **Webhook Handlers:** The `Request` carries the webhook payload and signature headers.
- **Middleware:** Middleware receives the `Request` to inspect or modify incoming data.
- **File Uploads:** The `Request` provides access to uploaded files via `$request->file()`.

### References

- Laravel HTTP Requests: Accessing the Request — https://laravel.com/docs/12.x/requests#accessing-the-request
- Laravel Service Container — https://laravel.com/docs/12.x/container

---

## 2. Input Extraction Methods

### Definitions

**Core Definition:** Input extraction methods are the `Request` instance's API for retrieving data from the query string, request body, or JSON payload, with options for filtering, casting, and subset selection.

**Technical Definition:** The `Request` class provides `input()`, `query()`, `only()`, `except()`, `all()`, `boolean()`, `date()`, `enum()`, and other methods for extracting input data. The `input()` method retrieves values from the combined input data (query string + body), while `query()` retrieves only from the query string. The `only()` and `except()` methods return subsets of the input array. Auto-casting methods (`boolean()`, `date()`, `enum()`, `integer()`, `string()`) convert raw input values to their corresponding PHP types. `all()` returns the entire input array. These methods use dot notation for nested array access.

**Beginner-Friendly Explanation:** When a user submits a form or an API sends JSON, you need to read that data. Laravel gives you many different tools for this. You can get a single value, a subset of values, everything except certain fields, or you can get values pre-cast to the right type (like a boolean or a date). It's like having a toolbox with different tools for different ways of reading the data.

### Purposes

- To extract general data from the request body or query string via `input()`.
- To extract strictly URL query string values via `query()`.
- To manage incoming structural arrays via `only()`, `except()`, and `all()`.
- To fetch auto-casted data directly from inputs via `boolean()`, `date()`, and `enum()`.
- To access nested array or JSON values using dot notation.
- To provide default values when input keys are missing.

### Syntax Rules and Structure

**Complete General Syntax — `input()` and `query()`:**

```php
$name = $request->input('name');
$name = $request->input('name', 'default');
$productName = $request->input('products.0.name');

$queryName = $request->query('name');
$queryName = $request->query('name', 'default');
$allQuery = $request->query();
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `input('name')` | Retrieves from the combined input data (query + body). |
| `input('name', 'default')` | Returns the default if the key is missing. |
| `input('products.0.name')` | Dot notation accesses nested arrays. |
| `query('name')` | Retrieves **only** from the URL query string. |
| `query()` | Returns the entire query string array. |

**Complete General Syntax — `only()`, `except()`, `all()`:**

```php
$credentials = $request->only(['username', 'password']);
$allExceptPassword = $request->except(['password', 'password_confirmation']);
$allInput = $request->all();
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `only([...])` | Returns an array with only the specified keys. |
| `except([...])` | Returns an array with all keys except those specified. |
| `all()` | Returns the entire input array. |

**Complete General Syntax — Auto-Casting:**

```php
$archived = $request->boolean('archived');
$birthday = $request->date('birthday');
$status = $request->enum('status', Status::class);
$perPage = $request->integer('per_page');
$name = $request->string('name')->trim();
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `boolean('archived')` | Casts `"1"`, `"true"`, `"on"`, `"yes"` to `true`; others to `false`. |
| `date('birthday')` | Returns a `Carbon` instance. |
| `enum('status', Status::class)` | Returns the matching enum case or `null`. |
| `integer('per_page')` | Casts the value to an integer. |
| `string('name')` | Returns a `Stringable` instance (supports `->trim()`, etc.). |

**Syntax Rules:**

- `input()` retrieves from the combined input data (query string + request body).
- `query()` retrieves **only** from the URL query string, ignoring the request body.
- Dot notation is supported for nested arrays and JSON payloads: `input('user.name')`, `input('products.0.price')`.
- `only()` and `except()` accept a single array or a dynamic list of arguments: `only('username', 'password')`.
- Auto-casting methods return `null` if the key is missing (or the specified default).

**Constraints and Limitations:**

- **`input()` vs. `query()`:** `input()` includes body data; `query()` does not. Use `query()` when you specifically need URL parameters.
- **JSON nested access:** Dot notation works for JSON payloads in Laravel 12+. In earlier versions, you may need to access nested arrays directly.
- **Enum casting:** Returns `null` if the input value does not match any enum case's backing value.
- **`all()` security:** Using `all()` blindly can expose mass-assignment vulnerabilities. Prefer `only()` or `validated()` in production code.

### Multiple Annotated Complete Code Examples

**Example 1: Basic Input Extraction**

```php
<?php
// routes/web.php

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;

Route::post('/search', function (Request $request) {
    // Retrieve a single value with a default
    $query = $request->input('q', 'default search');

    // Retrieve only specific fields
    $filters = $request->only(['category', 'sort', 'page']);

    // Retrieve everything except sensitive fields
    $safeData = $request->except(['csrf_token', 'password']);

    return response()->json([
        'query'   => $query,
        'filters' => $filters,
        'safe'    => $safeData,
    ]);
});
```

**Expected Output:** `POST /search` with `q=laravel&category=php&sort=latest&csrf_token=abc&password=secret` returns:
```json
{
    "query": "laravel",
    "filters": {"category": "php", "sort": "latest"},
    "safe": {"q": "laravel", "category": "php", "sort": "latest"}
}
```

**Why:** `input()` retrieves the `q` value from the request body. `only()` returns a subset of fields. `except()` returns everything except the specified sensitive fields.

---

**Example 2: Auto-Casting Input Values**

```php
<?php
// app/Http/Controllers/SettingsController.php

namespace App\Http\Controllers;

use App\Enums\Theme;
use Illuminate\Http\Request;

class SettingsController extends Controller
{
    public function update(Request $request)
    {
        // Auto-cast boolean (accepts "1", "true", "on", "yes" as true)
        $notifications = $request->boolean('notifications');

        // Auto-cast date to Carbon instance
        $birthday = $request->date('birthday');

        // Auto-cast to enum
        $theme = $request->enum('theme', Theme::class);

        // Auto-cast to integer
        $perPage = $request->integer('per_page', 25);

        return response()->json([
            'notifications' => $notifications,
            'birthday'      => $birthday?->toDateString(),
            'theme'         => $theme?->value,
            'per_page'      => $perPage,
        ]);
    }
}
```

```php
<?php
// app/Enums/Theme.php

namespace App\Enums;

enum Theme: string
{
    case Light = 'light';
    case Dark = 'dark';
    case System = 'system';
}
```

**Expected Output:** `POST /settings` with `notifications=on&birthday=1990-05-15&theme=dark&per_page=50` returns:
```json
{
    "notifications": true,
    "birthday": "1990-05-15",
    "theme": "dark",
    "per_page": 50
}
```

**Why:** `boolean()` casts `"on"` to `true`. `date()` returns a Carbon instance. `enum()` returns the matching `Theme::Dark` case. `integer()` casts `"50"` to `50`.

---

**Example 3: Nested Array and JSON Access**

```php
<?php
// routes/api.php

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;

Route::post('/orders', function (Request $request) {
    // Access nested JSON values with dot notation
    $productName = $request->input('items.0.name');
    $quantity = $request->integer('items.0.quantity');

    // Retrieve the entire nested array
    $items = $request->input('items');

    return response()->json([
        'first_product' => $productName,
        'quantity'      => $quantity,
        'items'         => $items,
    ]);
});
```

**Expected Output:** `POST /api/orders` with JSON body:
```json
{
    "items": [
        {"name": "Laptop", "quantity": 2},
        {"name": "Mouse", "quantity": 1}
    ]
}
```
Returns:
```json
{
    "first_product": "Laptop",
    "quantity": 2,
    "items": [
        {"name": "Laptop", "quantity": 2},
        {"name": "Mouse", "quantity": 1}
    ]
}
```

**Why:** Dot notation (`items.0.name`) accesses nested array elements. The `integer()` method casts the nested `quantity` value to an integer.

### Real-World Cases

- **Search Forms:** `input('q')` retrieves the search query; `only(['category', 'sort'])` retrieves filters.
- **API Payloads:** `input('user.email')` accesses nested JSON fields.
- **Settings Forms:** `boolean('notifications')` handles checkbox values.
- **Date Inputs:** `date('birthday')` returns a Carbon instance for date manipulation.
- **Enum Selection:** `enum('status', Status::class)` ensures the input matches a valid enum case.

### References

- Laravel HTTP Requests: Retrieving Input — https://laravel.com/docs/12.x/requests#retrieving-input
- Laravel HTTP Requests: Retrieving Boolean Input Values — https://laravel.com/docs/12.x/requests#retrieving-boolean-input-values
- Laravel HTTP Requests: Retrieving Date Input Values — https://laravel.com/docs/12.x/requests#retrieving-date-input-values
- Laravel HTTP Requests: Retrieving Enum Input Values — https://laravel.com/docs/12.x/requests#retrieving-enum-input-values
- Laravel Daily: 12 Laravel Request Methods You Might Not Know — https://laraveldaily.com/post/laravel-request-methods-you-might-not-know

---

## 3. Input Interception & Manipulation

### Definitions

**Core Definition:** Input interception and manipulation refers to the ability to modify the request's input data on the fly using `merge()` (to add or overwrite values) or `replace()` (to completely replace the input array).

**Technical Definition:** The `Request` class provides `merge(array $input)` and `mergeIfMissing(array $input)` methods that combine the given array with the request's existing input data. `merge()` overwrites existing keys; `mergeIfMissing()` only adds keys that do not already exist. The `replace(array $input)` method completely replaces the entire input array. These methods are commonly used in middleware, Form Request `prepareForValidation()`, and controllers to normalise, sanitise, or augment request data before it reaches validation or business logic.

**Beginner-Friendly Explanation:** Sometimes you need to change the request data before your application uses it. Maybe you want to add a default value that the user didn't provide, or strip out data that shouldn't be there. `merge()` lets you add or overwrite specific values. `replace()` lets you throw away the entire input and start fresh. It's like editing a document before handing it to someone else.

### Purposes

- To modify parameters on the fly via `merge()` (adding or overwriting specific values).
- To strip data with `replace()` (completely replacing the input array).
- To add default values for missing input keys via `mergeIfMissing()`.
- To normalise input data before validation (e.g., trimming strings, converting empty strings to null).
- To augment request data with computed values (e.g., generating a slug from a title).
- To sanitise input by removing sensitive or unnecessary fields.

### Syntax Rules and Structure

**Complete General Syntax — `merge()`:**

```php
$request->merge(['votes' => 0]);
$request->merge(['name' => 'Updated Name', 'status' => 'active']);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `merge(['key' => 'value'])` | Adds or overwrites the specified keys in the input data. |
| Existing keys | Are overwritten by the values in the merge array. |
| New keys | Are added to the input data. |

**Complete General Syntax — `mergeIfMissing()`:**

```php
$request->mergeIfMissing(['votes' => 0]);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `mergeIfMissing(['votes' => 0])` | Adds the `votes` key only if it does not already exist. |

**Complete General Syntax — `replace()`:**

```php
$request->replace(['name' => 'New Name', 'email' => 'new@example.com']);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `replace([...])` | Completely replaces the entire input array with the given array. |

**Syntax Rules:**

- `merge()` overwrites existing keys and adds new ones.
- `mergeIfMissing()` only adds keys that are not already present.
- `replace()` discards the entire existing input array and replaces it with the given array.
- These methods modify the request instance in place—they do not return a new instance.
- `merge()` and `replace()` are commonly used in `prepareForValidation()` on Form Requests.

**Constraints and Limitations:**

- **Irreversibility:** Once input is merged or replaced, the original data is lost. There is no "undo" method.
- **Middleware ordering:** If multiple middleware modify the input, the order of execution matters.
- **Validation impact:** Merging input before validation can affect what the validator sees. This is intentional but should be used carefully.
- **`replace()` caution:** Replacing the entire input array removes all existing data. Use `merge()` if you only need to change specific values.

### Multiple Annotated Complete Code Examples

**Example 1: Merging Additional Input**

```php
<?php
// routes/web.php

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;

Route::post('/posts', function (Request $request) {
    // Merge a default value for the 'status' key
    $request->merge(['status' => 'draft']);

    // Merge a computed slug from the title
    $request->merge(['slug' => \Str::slug($request->input('title'))]);

    return response()->json([
        'title'  => $request->input('title'),
        'status' => $request->input('status'),
        'slug'   => $request->input('slug'),
    ]);
});
```

**Expected Output:** `POST /posts` with `title=Hello World` returns:
```json
{
    "title": "Hello World",
    "status": "draft",
    "slug": "hello-world"
}
```

**Why:** `merge()` adds the `status` key with a default value and the `slug` key with a computed value. Both are now available alongside the original input.

---

**Example 2: Using `mergeIfMissing()` for Defaults**

```php
<?php
// routes/web.php

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;

Route::patch('/users/{id}', function (Request $request, string $id) {
    // Only set 'role' to 'user' if it's not already provided
    $request->mergeIfMissing(['role' => 'user']);

    return response()->json([
        'id'   => $id,
        'role' => $request->input('role'),
        'name' => $request->input('name'),
    ]);
});
```

**Expected Output:**
- `PATCH /users/1` with `name=Alice` → `{"id":"1","role":"user","name":"Alice"}`
- `PATCH /users/1` with `name=Alice&role=admin` → `{"id":"1","role":"admin","name":"Alice"}`

**Why:** `mergeIfMissing()` adds the `role` key only if it's not already present. If the client provides a `role`, that value is preserved.

---

**Example 3: Replacing the Entire Input Array**

```php
<?php
// routes/web.php

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;

Route::post('/webhook', function (Request $request) {
    // Replace the entire input with a sanitised version
    $request->replace([
        'event'   => $request->input('type'),
        'payload' => $request->input('data'),
    ]);

    // Now only 'event' and 'payload' are available
    return response()->json([
        'event'   => $request->input('event'),
        'payload' => $request->input('payload'),
        'type'    => $request->input('type'), // null — original key is gone
    ]);
});
```

**Expected Output:** `POST /webhook` with `type=user.created&data={...}&extra=ignored` returns:
```json
{
    "event": "user.created",
    "payload": {...},
    "type": null
}
```

**Why:** `replace()` completely replaces the input array. The original `type` and `data` keys are removed, and only the new `event` and `payload` keys remain.

### Real-World Cases

- **Form Requests:** `prepareForValidation()` uses `merge()` to normalise input (e.g., trimming, slug generation).
- **Middleware:** Middleware can `merge()` tenant IDs or `replace()` sensitive data before it reaches the controller.
- **API Gateways:** Incoming payloads can be transformed with `merge()` before validation.
- **Webhook Handlers:** `replace()` can sanitise webhook payloads to only the relevant fields.
- **Multi-Step Forms:** `merge()` can carry forward data from previous steps that isn't in the current request.

### References

- Laravel HTTP Requests: Merging Additional Input — https://laravel.com/docs/12.x/requests#merging-additional-input
- Laravel HTTP Requests: Old Input — https://laravel.com/docs/12.x/requests#old-input
- Laravel Form Request Validation — https://laravel.com/docs/12.x/validation#form-request-validation

---

## 4. State Verification

### Definitions

**Core Definition:** State verification methods allow you to inspect the request to determine whether specific input keys are present, filled, or missing, and to check the HTTP method used for the request.

**Technical Definition:** The `Request` class provides `has()`, `hasAny()`, `filled()`, `isNotFilled()`, `anyFilled()`, `whenFilled()`, `whenHas()`, `missing()`, `whenMissing()`, `isMethod()`, and `method()` methods. `has()` checks if a key is present (even with a `null` value). `filled()` checks if a key is present and not an empty string. `whenFilled()` executes a closure conditionally based on whether the key is filled. `isMethod()` verifies the HTTP verb. These methods are essential for conditional logic, validation preparation, and middleware branching.

**Beginner-Friendly Explanation:** Before you use input data, you often need to check if it exists. Did the user actually fill out the `name` field? Is the `email` field present but empty? What HTTP method did they use? These methods let you ask those questions and branch your logic accordingly. It's like checking whether a box is empty, filled, or missing entirely before deciding what to do with it.

### Purposes

- To inspect requests using `has()` (key presence) and `filled()` (non-empty value).
- To conditionally execute logic with `whenFilled()` and `whenHas()`.
- To determine if a key is absent using `missing()` and `whenMissing()`.
- To verify the HTTP method using `isMethod()` and `method()`.
- To check multiple keys at once with `hasAny()`, `anyFilled()`, and `isNotFilled()`.
- To enable conditional validation and middleware behaviour.

### Syntax Rules and Structure

**Complete General Syntax — Presence Checks:**

```php
if ($request->has('name')) { /* key exists */ }
if ($request->has(['name', 'email'])) { /* both keys exist */ }
if ($request->hasAny(['name', 'email'])) { /* at least one exists */ }
if ($request->missing('name')) { /* key does not exist */ }
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `has('key')` | Returns `true` if the key is present (even with `null` value). |
| `has(['a', 'b'])` | Returns `true` if **all** specified keys are present. |
| `hasAny(['a', 'b'])` | Returns `true` if **any** specified key is present. |
| `missing('key')` | Returns `true` if the key is absent. |

**Complete General Syntax — Filled Checks:**

```php
if ($request->filled('name')) { /* key exists and is not empty string */ }
if ($request->isNotFilled('name')) { /* key is missing or empty string */ }
if ($request->anyFilled(['name', 'email'])) { /* at least one is filled */ }
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `filled('key')` | Returns `true` if the key is present and not an empty string. |
| `isNotFilled('key')` | Returns `true` if the key is missing or an empty string. |
| `anyFilled(['a', 'b'])` | Returns `true` if at least one key is filled. |

**Complete General Syntax — Conditional Closures:**

```php
$request->whenFilled('name', function (string $input) {
    // Executed if 'name' is filled
}, function () {
    // Executed if 'name' is not filled
});

$request->whenHas('name', function (string $input) {
    // Executed if 'name' is present
});
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `whenFilled()` | Executes the closure if the key is present and not empty. |
| `whenHas()` | Executes the closure if the key is present. |
| Second closure | Optional fallback executed when the condition is false. |

**Complete General Syntax — HTTP Method Checks:**

```php
if ($request->isMethod('post')) { /* ... */ }
$method = $request->method(); // "GET", "POST", etc.
```

**Syntax Rules:**

- `has()` returns `true` even if the value is `null` or an empty string—it only checks key presence.
- `filled()` returns `false` for `null` and empty strings, but `true` for whitespace strings (`" "`).
- `whenFilled()` executes the closure only if the key is filled (present and non-empty).
- `isMethod()` accepts a string (e.g., `'post'`) or an array of methods (e.g., `['post', 'put']`).
- `missing()` and `isNotFilled()` are the logical opposites of `has()` and `filled()` respectively.

**Constraints and Limitations:**

- **`has()` vs. `filled()`:** `has('name')` returns `true` even if `name` is `null` or `""`. Use `filled()` when you need a non-empty value.
- **Whitespace:** `filled()` considers a string of spaces as filled. Use `trim()` if whitespace should be treated as empty.
- **`whenFilled()` return value:** The closure's return value is not automatically returned by `whenFilled()`. Use `$request->whenFilled(...)` and capture the result if needed.

### Multiple Annotated Complete Code Examples

**Example 1: Presence and Filled Checks**

```php
<?php
// routes/web.php

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;

Route::post('/profile', function (Request $request) {
    $messages = [];

    // Check if 'name' is present (even if empty)
    if ($request->has('name')) {
        $messages[] = 'Name key is present.';
    }

    // Check if 'email' is filled (non-empty)
    if ($request->filled('email')) {
        $messages[] = 'Email is filled.';
    }

    // Check if 'phone' is missing
    if ($request->missing('phone')) {
        $messages[] = 'Phone is missing.';
    }

    return response()->json(['messages' => $messages]);
});
```

**Expected Output:** `POST /profile` with `name=&email=alice@example.com` returns:
```json
{
    "messages": [
        "Name key is present.",
        "Email is filled.",
        "Phone is missing."
    ]
}
```

**Why:** `has('name')` returns `true` because the key is present, even though its value is an empty string. `filled('email')` returns `true` because the email is non-empty. `missing('phone')` returns `true` because the phone key is absent.

---

**Example 2: Conditional Logic with `whenFilled()`**

```php
<?php
// routes/web.php

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;

Route::post('/subscribe', function (Request $request) {
    $request->whenFilled('promo_code', function (string $code) {
        // Apply promo code logic only if promo_code is provided
        \Log::info("Promo code applied: {$code}");
    }, function () {
        // No promo code provided
        \Log::info('No promo code provided');
    });

    return response()->json(['status' => 'subscribed']);
});
```

**Expected Output:**
- `POST /subscribe` with `promo_code=SAVE20` → logs `"Promo code applied: SAVE20"`.
- `POST /subscribe` without `promo_code` → logs `"No promo code provided"`.

**Why:** `whenFilled()` executes the first closure only when the `promo_code` key is present and non-empty. Otherwise, the second closure (fallback) is executed.

---

**Example 3: HTTP Method Verification**

```php
<?php
// routes/web.php

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;

Route::match(['get', 'post'], '/contact', function (Request $request) {
    if ($request->isMethod('post')) {
        // Handle form submission
        return 'Thank you for your message!';
    }

    // Handle form display
    return 'Please fill out the contact form.';
});
```

**Expected Output:**
- `GET /contact` → `Please fill out the contact form.`
- `POST /contact` → `Thank you for your message!`

**Why:** `isMethod('post')` returns `true` only for POST requests. The same route handles both GET and POST, branching the response based on the HTTP method.

### Real-World Cases

- **Conditional Validation:** `whenFilled()` triggers validation rules only for fields that are provided.
- **Form Repopulation:** `has()` and `filled()` determine which form fields to pre-populate with old input.
- **API Versioning:** `isMethod()` branches logic for different HTTP verbs on the same URI.
- **Middleware:** `missing()` checks for required headers or API keys before proceeding.
- **Webhooks:** `hasAny()` checks for the presence of specific webhook headers.

### References

- Laravel HTTP Requests: Determining If Input Is Present — https://laravel.com/docs/12.x/requests#determining-if-input-is-present
- Laravel HTTP Requests: Request Method — https://laravel.com/docs/12.x/requests#request-method
- Laravel Daily: Laravel Request `has` vs `filled` — https://laraveldaily.com/post/laravel-request-has-vs-filled

---

## 5. Metadata & Transport Tracing

### Definitions

**Core Definition:** Metadata and transport tracing methods provide access to the request's transport-layer information—headers, IP address, bearer tokens, path patterns, and content negotiation preferences—enabling authentication, routing decisions, and client detection.

**Technical Definition:** The `Request` class exposes `header()`, `hasHeader()`, `bearerToken()`, `ip()`, `ips()`, `path()`, `is()`, `routeIs()`, `url()`, `fullUrl()`, `method()`, `isMethod()`, `accepts()`, `prefers()`, and `expectsJson()` methods. These methods wrap the underlying Symfony request's header bag, server parameters, and path information. `bearerToken()` extracts the token from the `Authorization: Bearer <token>` header. `is()` matches the request path against a wildcard pattern. `expectsJson()` checks the `Accept` header for JSON expectation.

**Beginner-Friendly Explanation:** Every request comes with a lot of information beyond just the data the user submitted. What browser are they using? What's their IP address? Do they want HTML or JSON? Are they authenticated with a token? These methods let you read all that "metadata" about the request. It's like checking the envelope before opening the letter—who sent it, where it came from, and what kind of letter it is.

### Purposes

- To extract client properties like headers via `header()`.
- To identify the client's IP address via `ip()` and `ips()`.
- To retrieve bearer tokens for API authentication via `bearerToken()`.
- To match path patterns using `is('api/*')` and `routeIs('admin.*')`.
- To perform content negotiation via `accepts()`, `prefers()`, and `expectsJson()`.
- To inspect the request URL and method for routing and logging purposes.

### Syntax Rules and Structure

**Complete General Syntax — Headers:**

```php
$value = $request->header('X-Header-Name');
$value = $request->header('X-Header-Name', 'default');
if ($request->hasHeader('X-Header-Name')) { /* ... */ }
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `header('name')` | Returns the header value or `null`. |
| `header('name', 'default')` | Returns the default if the header is absent. |
| `hasHeader('name')` | Returns `true` if the header is present. |

**Complete General Syntax — IP Address:**

```php
$ip = $request->ip();
$ips = $request->ips();
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `ip()` | Returns the client's IP address (respects trusted proxies). |
| `ips()` | Returns an array of IPs (client + proxies). |

**Complete General Syntax — Bearer Token:**

```php
$token = $request->bearerToken();
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `bearerToken()` | Extracts the token from the `Authorization: Bearer <token>` header. |
| No header | Returns an empty string (`''`), not `null`. |

**Complete General Syntax — Path Pattern Matching:**

```php
if ($request->is('api/*')) { /* ... */ }
if ($request->routeIs('admin.*')) { /* ... */ }
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `is('api/*')` | Matches the request path against a wildcard pattern. |
| `routeIs('admin.*')` | Matches the named route against a wildcard pattern. |

**Complete General Syntax — Content Negotiation:**

```php
if ($request->accepts(['text/html', 'application/json'])) { /* ... */ }
$preferred = $request->prefers(['text/html', 'application/json']);
if ($request->expectsJson()) { /* ... */ }
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `accepts([...])` | Returns `true` if any of the content types are accepted. |
| `prefers([...])` | Returns the most preferred content type. |
| `expectsJson()` | Returns `true` if the request expects a JSON response. |

**Syntax Rules:**

- `header()` returns `null` if the header is absent, unless a default is provided.
- `bearerToken()` returns an empty string (`''`) if no `Authorization: Bearer` header exists.
- `is()` uses `*` as a wildcard character (e.g., `admin/*` matches `admin/users`).
- `routeIs()` matches against the **named route** pattern, not the URI.
- `expectsJson()` checks the `Accept` header for `application/json` or `+json` suffixes.

**Constraints and Limitations:**

- **Trusted proxies:** `ip()` respects the `TrustProxies` middleware. Without it, the IP may be the proxy's address, not the client's.
- **Bearer token case:** The header name is case-insensitive, but the `Bearer` prefix must be present.
- **`expectsJson()` vs. `wantsJson()`:** `expectsJson()` returns `true` for AJAX requests and JSON `Accept` headers. `wantsJson()` is a more permissive check.
- **Path pattern anchoring:** `is()` patterns are anchored to the full path. Use `*` for prefix matching.

### Multiple Annotated Complete Code Examples

**Example 1: Header and Bearer Token Extraction**

```php
<?php
// routes/api.php

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;

Route::get('/user', function (Request $request) {
    // Extract the bearer token
    $token = $request->bearerToken();

    // Extract a custom header
    $apiVersion = $request->header('X-API-Version', '1.0');

    return response()->json([
        'token'       => $token ? 'present' : 'missing',
        'api_version' => $apiVersion,
    ]);
});
```

**Expected Output:** `GET /api/user` with `Authorization: Bearer abc123` and `X-API-Version: 2.0` returns:
```json
{
    "token": "present",
    "api_version": "2.0"
}
```

**Why:** `bearerToken()` extracts the token from the `Authorization` header. `header()` retrieves the custom `X-API-Version` header with a default fallback.

---

**Example 2: Path Pattern Matching and IP Address**

```php
<?php
// routes/web.php

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;

Route::get('/{any}', function (Request $request) {
    $response = [
        'path' => $request->path(),
        'ip'   => $request->ip(),
    ];

    if ($request->is('admin/*')) {
        $response['area'] = 'admin';
    } elseif ($request->is('api/*')) {
        $response['area'] = 'api';
    } else {
        $response['area'] = 'web';
    }

    return response()->json($response);
})->where('any', '.*');
```

**Expected Output:** `GET /admin/users` returns:
```json
{
    "path": "admin/users",
    "ip": "127.0.0.1",
    "area": "admin"
}
```

**Why:** `is('admin/*')` matches the path `admin/users`. The `ip()` method returns the client's IP address (respecting trusted proxies).

---

**Example 3: Content Negotiation with `expectsJson()` and `prefers()`**

```php
<?php
// routes/web.php

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;

Route::get('/posts', function (Request $request) {
    $posts = ['Post 1', 'Post 2'];

    if ($request->expectsJson()) {
        return response()->json($posts);
    }

    $preferred = $request->prefers(['text/html', 'application/json']);

    return response("Preferred content type: {$preferred}");
});
```

**Expected Output:**
- Browser visiting `/posts` with `Accept: text/html` → `Preferred content type: text/html`
- API client with `Accept: application/json` → `["Post 1", "Post 2"]`

**Why:** `expectsJson()` returns `true` for JSON `Accept` headers, triggering the JSON response. Otherwise, `prefers()` determines the most preferred content type between HTML and JSON.

### Real-World Cases

- **API Authentication:** `bearerToken()` extracts the token for Sanctum or Passport authentication.
- **Multi-Tenancy:** `header('X-Tenant-ID')` identifies the tenant for data scoping.
- **Admin Route Protection:** `is('admin/*')` triggers admin-specific middleware or logic.
- **Content Negotiation:** `expectsJson()` returns JSON for API clients and HTML for browsers.
- **Geo-Location:** `ip()` identifies the client for geo-location or rate limiting.
- **Webhook Verification:** `header('X-Signature')` retrieves the webhook signature for verification.

### References

- Laravel HTTP Requests: Request Headers — https://laravel.com/docs/12.x/requests#request-headers
- Laravel HTTP Requests: Request IP Address — https://laravel.com/docs/12.x/requests#request-ip-address
- Laravel HTTP Requests: Request Path, Host, and Method — https://laravel.com/docs/12.x/requests#request-path-and-method
- Laravel HTTP Requests: Content Negotiation — https://laravel.com/docs/12.x/requests#content-negotiation
- Laravel Daily: Laravel Request Methods You Might Not Know — https://laraveldaily.com/post/laravel-request-methods-you-might-not-know

---

## References

- Laravel HTTP Requests Documentation (12.x) — https://laravel.com/docs/12.x/requests
- Laravel HTTP Requests: Accessing the Request — https://laravel.com/docs/12.x/requests#accessing-the-request
- Laravel HTTP Requests: Retrieving Input — https://laravel.com/docs/12.x/requests#retrieving-input
- Laravel HTTP Requests: Merging Additional Input — https://laravel.com/docs/12.x/requests#merging-additional-input
- Laravel HTTP Requests: Determining If Input Is Present — https://laravel.com/docs/12.x/requests#determining-if-input-is-present
- Laravel HTTP Requests: Request Headers — https://laravel.com/docs/12.x/requests#request-headers
- Laravel HTTP Requests: Request IP Address — https://laravel.com/docs/12.x/requests#request-ip-address
- Laravel HTTP Requests: Content Negotiation — https://laravel.com/docs/12.x/requests#content-negotiation
- Laravel Request Lifecycle Documentation — https://laravel.com/docs/12.x/lifecycle
- Laravel Service Container Documentation — https://laravel.com/docs/12.x/container
- Laravel Validation Documentation — https://laravel.com/docs/12.x/validation
- Laravel 12 request updates: Nested JSON, mergeIfMissing, bearerToken, prefers (LinkedIn) — https://www.linkedin.com/posts/im-rahulkumar-rajput_laravel-php-webdevelopment-activity-7357482718110101504-SpAM
- Laravel `Request` チートシート (Zenn) — https://zenn.dev/s_kitade/articles/aa88fa010d8ddb
- Laravel Daily: 12 Laravel Request Methods You Might Not Know — https://laraveldaily.com/post/laravel-request-methods-you-might-not-know
- Laravel Daily: Laravel Request `has` vs `filled` — https://laraveldaily.com/post/laravel-request-has-vs-filled
- These 32 HTTP Request Concepts Every Laravel Developer Should Know (LinkedIn) — https://www.linkedin.com/posts/kamil-wojtalak_these-32-http-request-concepts-every-laravel-activity-7324354407326109696-_Jf1