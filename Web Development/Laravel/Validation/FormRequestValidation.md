# Laravel Form Request Validation: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

A Laravel Form Request is a dedicated class that encapsulates validation and authorization logic for a specific HTTP request, extracting it from controllers into a reusable, testable unit.

**Technical Definition**

Form Requests extend `Illuminate\Foundation\Http\FormRequest`, which itself extends the base `Request` class and implements the `ValidatesWhenResolved` contract. When a controller method type-hints a Form Request class, Laravel's service container resolves the class and automatically triggers the `authorize()` method and the validation rules defined in `rules()`. If authorization fails, an HTTP 403 response is returned and the controller method never executes. If validation fails, the user is redirected back with flashed error messages and input, or a JSON 422 response is returned for AJAX/API requests. The class provides extension points for customising error messages (`messages()`), attribute names (`attributes()`), input preparation (`prepareForValidation()`), post-validation normalization (`passedValidation()`), and after-validation hooks (`after()`).

**Beginner-Friendly Explanation**

A Form Request is a dedicated class that handles all the "checking" for a specific form or API endpoint. Instead of writing validation rules inside your controller—which can make the controller messy and hard to read—you move them into their own class. When someone submits a form, Laravel automatically finds the right Form Request class (based on the type-hint in your controller), runs the authorization check, then runs all the validation rules. If anything fails, Laravel handles the redirect and error messages for you. It keeps your controllers clean and your validation logic reusable.


### Key Characteristics

- **Automatic Execution**: Type-hinting a Form Request in a controller method triggers authorization and validation automatically before the method executes—no manual `validate()` call is needed.
- **Separation of Concerns**: Validation and authorization logic live in dedicated classes, keeping controllers thin and focused on business logic.
- **Reusability**: The same Form Request can be used across multiple controllers or actions, ensuring consistent validation rules.
- **Authorization Integration**: The `authorize()` method determines if the authenticated user has permission to make the request; returning `false` results in an automatic 403 response.
- **Customisable Error Messages and Attributes**: The `messages()` and `attributes()` methods allow full control over the wording and field names used in error messages.
- **Input Preparation and Normalisation**: The `prepareForValidation()` method sanitises or transforms input before validation; `passedValidation()` normalises data after validation passes.
- **After-Validation Hooks**: The `after()` method supports custom validation logic that depends on multiple fields or database state.
- **Named Error Bags**: The `$errorBag` property or `validateWithBag()` method stores errors in a named bag, preventing cross-contamination when multiple forms exist on one page.
- **Configurable Redirects**: The `$redirect`, `$redirectRoute`, and `$redirectAction` properties customise where users are redirected on validation failure.
- **Stop on First Failure**: The `$stopOnFirstFailure` property instructs the validator to stop validating all attributes once a single failure occurs.


### Prerequisites

- **PHP 8.1 or higher** (Laravel 10+) or **PHP 8.2** (Laravel 11+).
- **A Laravel application** installed via Composer.
- **A basic understanding of HTTP requests** and form handling.
- **Familiarity with Laravel's validation rules** (see the Validation Fundamentals cheat sheet).
- **Knowledge of Eloquent models** for database-backed rules (`unique`, `exists`).
- **A text editor or IDE** with PHP support.


### Related Programming Areas

- **Validation**: Form Requests are the recommended approach for non-trivial validation in Laravel.
- **Authorization**: The `authorize()` method integrates with Gates and Policies for request-level access control.
- **Controllers**: Form Requests keep controllers thin by extracting validation and authorization logic.
- **Blade Templating**: Validation errors flashed to the session are displayed in Blade views using the `@error` directive.
- **API Development**: Form Requests automatically return JSON 422 responses for API requests, providing consistent error structures.
- **Service Container**: Dependencies type-hinted in the `authorize()` or `rules()` methods are automatically resolved.


## Core Concepts

---

### 1. Form Requests

#### Definitions

**Core Definition**

A Form Request is a custom request class that contains validation rules and authorization logic for a specific HTTP request, executed automatically when the request is resolved.

**Technical Definition**

Form Requests extend `Illuminate\Foundation\Http\FormRequest`. The class must define a `rules()` method returning an array of validation rules and may define an `authorize()` method returning a boolean. When a controller method type-hints the Form Request, Laravel's `ValidatesWhenResolved` contract triggers validation during route resolution. The `validated()` method returns only the fields that were validated, preventing mass assignment of unvalidated data.

**Beginner-Friendly Explanation**

A Form Request is a class you create for each form (or API endpoint) that needs validation. It holds all the rules and permission checks. When the form is submitted, Laravel finds the class automatically and runs everything. You do not have to call any validation method yourself—it just works.

#### Purposes

- To extract validation and authorization logic from controllers into dedicated, reusable classes.
- To ensure consistent validation rules across multiple controllers or actions.
- To automatically handle validation failure responses (redirects or JSON) without manual intervention.
- To provide a single source of truth for a request's validation and authorization requirements.
- To enable dependency injection for authorization and validation logic.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
<?php
// app/Http/Requests/StorePostRequest.php
namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class StorePostRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;
    }

    public function rules(): array
    {
        return [
            'title' => 'required|string|max:255',
            'body'  => 'required|string|min:10',
        ];
    }
}
```

```php
// Controller usage
public function store(StorePostRequest $request)
{
    Post::create($request->validated());
    return redirect()->route('posts.index');
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `extends FormRequest` | Base class providing validation and authorization infrastructure. |
| `authorize(): bool` | Determines if the user is authorized. Return `true` to allow. |
| `rules(): array` | Returns the validation rules for the request. |
| `$request->validated()` | Returns only the validated fields. |

**Syntax Rules**

- The class must extend `FormRequest`.
- `rules()` is required; `authorize()` is optional (defaults to allowing all).
- The controller method must type-hint the Form Request as its first parameter.
- The `validated()` method returns an array of only the validated fields.
- Form Requests are stored in `app/Http/Requests`.

**Constraints and Limitations**

- Form Requests cannot be used in Closure-based routes; they require a controller method.
- Extremely simple endpoints with one or two trivial rules may not warrant a Form Request class.
- The `authorize()` method must return a boolean; returning `false` results in a 403 response.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Basic Form Request with Authorization**

**Form Request**

```php
<?php
// app/Http/Requests/StorePostRequest.php
namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class StorePostRequest extends FormRequest
{
    public function authorize(): bool
    {
        // Only authenticated users can create posts
        return $this->user() !== null;
    }

    public function rules(): array
    {
        return [
            'title' => 'required|string|max:255',
            'body'  => 'required|string|min:10',
        ];
    }
}
```

**Controller**

```php
<?php
// app/Http/Controllers/PostController.php
namespace App\Http\Controllers;

use App\Http\Requests\StorePostRequest;
use App\Models\Post;

class PostController extends Controller
{
    public function store(StorePostRequest $request)
    {
        $post = Post::create($request->validated());

        return redirect()->route('posts.show', $post)
            ->with('success', 'Post created successfully.');
    }
}
```

**Route**

```php
// routes/web.php
Route::post('/posts', [PostController::class, 'store']);
```

**Expected Output (Success)**

```
// Authenticated user, valid data → post created, redirect to posts.show with success message
```

**Expected Output (Unauthenticated)**

```
// Guest user → 403 Forbidden response, controller method not executed
```

**Expected Output (Validation Failure)**

```
// Missing title → redirect back with "The title field is required." and old input
```

**Why This Output Occurs**

The `authorize()` method checks if a user is authenticated. If not, Laravel returns a 403 response before the controller method runs. If authorized, the `rules()` method validates the input. On failure, Laravel redirects back with errors; on success, `$request->validated()` returns only the validated fields, which are used to create the post.

---

**Example 2: Form Request with Route Model Binding**

**Form Request**

```php
<?php
// app/Http/Requests/UpdatePostRequest.php
namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class UpdatePostRequest extends FormRequest
{
    public function authorize(): bool
    {
        // Only the post author or an admin can update
        $post = $this->route('post');
        return $this->user()->id === $post->user_id
            || $this->user()->hasRole('admin');
    }

    public function rules(): array
    {
        return [
            'title' => 'required|string|max:255',
            'body'  => 'required|string|min:10',
        ];
    }
}
```

**Controller**

```php
public function update(UpdatePostRequest $request, Post $post)
{
    $post->update($request->validated());

    return redirect()->route('posts.show', $post);
}
```

**Expected Output**

```
// Author → update proceeds
// Non-author non-admin → 403 Forbidden
// Validation fails → redirect back with errors
```

**Why This Output Occurs**

The `authorize()` method accesses the resolved `Post` model via `$this->route('post')` (route model binding) and checks ownership or admin role. If authorized, validation runs; otherwise, a 403 response is returned.

---

#### Real-World Cases with Explanation

**Case 1: Blog Post Management**

A blog application uses `StorePostRequest` for creating posts and `UpdatePostRequest` for updating posts. Both classes share similar rules but have different authorization logic (create requires authentication; update requires ownership or admin). The controllers remain thin, and the validation logic is reusable.

**Explanation**: Form Requests centralise validation and authorization, making the application easier to maintain and test.

**Case 2: API Endpoint with Consistent Error Responses**

A REST API uses Form Requests for all write endpoints. Validation failures automatically return a JSON 422 response with structured error data, which the frontend parses to display field-level errors. The API controllers contain only the business logic for creating or updating resources.

**Explanation**: Form Requests ensure consistent API error handling across all endpoints without duplicating validation logic.


### 2. Form Request Creation

#### Definitions

**Core Definition**

Form Request creation is the process of generating a new Form Request class using Laravel's Artisan CLI.

**Technical Definition**

The `php artisan make:request` command generates a PHP class in `app/Http/Requests` extending `Illuminate\Foundation\Http\FormRequest`. The generated class contains stubs for the `authorize()` and `rules()` methods. The `authorize()` method defaults to returning `false`, requiring the developer to explicitly enable authorization.

**Beginner-Friendly Explanation**

Creating a Form Request is a one-command process. Laravel makes the file for you with the two methods you need to fill in: one for permissions and one for validation rules.

#### Purposes

- To automate the creation of Form Request boilerplate.
- To ensure Form Requests are placed in the correct namespace and directory.
- To provide consistent method stubs for authorization and validation.
- To reduce manual file creation errors.

#### Syntax Rules and Structure

**Complete General Syntax**

```bash
php artisan make:request StorePostRequest
php artisan make:request UpdatePostRequest
```

**Component Breakdown**

| Command | Effect |
|---------|--------|
| `make:request StorePostRequest` | Creates `app/Http/Requests/StorePostRequest.php`. |
| `make:request UpdatePostRequest` | Creates `app/Http/Requests/UpdatePostRequest.php`. |

**Syntax Rules**

- The request name should be PascalCase and descriptive (e.g., `StorePostRequest`, `UpdateUserRequest`).
- The `Request` suffix is a strong convention but not enforced.
- Generated files must be edited to add rules and authorization logic.

**Constraints and Limitations**

- The command does not register the Form Request automatically; it must be type-hinted in a controller.
- The default `authorize()` method returns `false`, which blocks all requests until changed.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Generating a Form Request**

**Command**

```bash
php artisan make:request StorePostRequest
```

**Generated File**

```php
<?php
// app/Http/Requests/StorePostRequest.php
namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class StorePostRequest extends FormRequest
{
    public function authorize(): bool
    {
        return false;
    }

    public function rules(): array
    {
        return [
            //
        ];
    }
}
```

**Expected Output (Terminal)**

```
INFO  Request [app/Http/Requests/StorePostRequest.php] created successfully.
```

**Why This Output Occurs**

Artisan scaffolds the Form Request class with the required methods. The `authorize()` method returns `false` by default, and `rules()` returns an empty array.

---

#### Real-World Cases with Explanation

**Case 1: Rapid Feature Development**

A developer working on a new "Product" feature runs `php artisan make:request StoreProductRequest` and immediately fills in the rules and authorization logic. The Form Request is then type-hinted in the `ProductController`'s `store` method.

**Explanation**: Artisan accelerates development by generating the necessary boilerplate.


### 3. Form Request Validation

#### Definitions

**Core Definition**

Form Request validation is the process by which the `rules()` method's validation rules are applied to the incoming request data when the Form Request is resolved.

**Technical Definition**

When a controller method type-hints a Form Request, Laravel's route dispatcher resolves the class and calls its `validateResolved()` method. This method invokes `$this->validator = $this->createDefaultValidator($this->container['validation.validator'])`, which applies the rules returned by `rules()` to the request input. If validation fails, a `ValidationException` is thrown and converted to a redirect or JSON response. On success, `$request->validated()` returns only the validated data.

**Beginner-Friendly Explanation**

Validation happens automatically when you type-hint a Form Request in your controller. You just list your rules in the `rules()` method, and Laravel checks them before your controller code runs.

#### Purposes

- To apply validation rules to incoming request data automatically.
- To ensure only validated data is used for mass assignment.
- To provide a consistent validation experience across all request types (web, API).
- To keep validation rules separate from controller logic.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
public function rules(): array
{
    return [
        'title' => 'required|string|max:255',
        'body'  => ['required', 'string', 'min:10'],
        'email' => ['required', 'email', Rule::unique('users')],
    ];
}
```

**Component Breakdown**

| Rule Category | Examples |
|--------------|----------|
| Presence | `required`, `nullable`, `filled` |
| Type | `string`, `integer`, `numeric`, `boolean`, `array` |
| Length | `min:value`, `max:value`, `between:min,max` |
| Format | `email`, `url`, `ip`, `json`, `regex:pattern` |
| Database | `unique:table,column`, `exists:table,column` |
| Conditional | `required_if`, `Rule::when()` |

**Syntax Rules**

- `rules()` must return an array.
- Rules can be strings (`'required|email'`) or arrays (`['required', 'email']`).
- Array notation is preferred for rules with parameters containing commas or `|` characters.
- The `bail` rule stops validation for a field on first failure.
- The `nullable` rule allows `null` values.

**Constraints and Limitations**

- The `validated()` method returns only the fields present in the rules array.
- Rules that depend on the authenticated user (e.g., `unique:users,email,{$userId}`) can access the user via `$this->user()`.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Basic Validation Rules**

**Form Request**

```php
public function rules(): array
{
    return [
        'name'     => 'required|string|max:255',
        'email'    => 'required|email|unique:users,email',
        'password' => 'required|string|min:8|confirmed',
        'role'     => 'required|in:admin,editor,viewer',
    ];
}
```

**Expected Output**

```
// Valid data → passes validation
// Missing email → "The email field is required."
// Password too short → "The password must be at least 8 characters."
// Role not in list → "The selected role is invalid."
```

**Why This Output Occurs**

Each rule applies its specific constraint. Multiple rules for a field are applied in sequence, and the first failure generates an error message.

---

**Example 2: Nested Array Validation with Wildcards**

```php
public function rules(): array
{
    return [
        'items'              => 'required|array|min:1',
        'items.*.product_id' => 'required|integer|exists:products,id',
        'items.*.quantity'   => 'required|integer|min:1|max:100',
    ];
}
```

**Expected Output**

```
// Valid items array → passes
// items[0].product_id missing → "The items.0.product_id field is required."
// items[1].quantity is 0 → "The items.1.quantity must be at least 1."
```

**Why This Output Occurs**

The wildcard `*` applies the rules to every element in the `items` array. Error messages include the specific index.

---

#### Real-World Cases with Explanation

**Case 1: E-commerce Order Validation**

An order Form Request validates `customer_email`, `items` (nested array), `items.*.product_id` (exists in products), and `items.*.quantity` (integer, min 1). This ensures every order item is valid before the order is created.

**Explanation**: Nested array validation is essential for complex data structures like orders.

---

### 4. Form Request Authorization

#### Definitions

**Core Definition**

Form Request authorization is the process by which the `authorize()` method determines whether the authenticated user has permission to perform the request, returning a boolean.

**Technical Definition**

The `authorize()` method is called before validation. It must return `true` (authorized) or `false` (denied). If `false` is returned, Laravel automatically returns an HTTP 403 response, and the controller method is never executed. The method has access to the authenticated user via `$this->user()` and route parameters via `$this->route('parameter')`. Dependencies type-hinted in the `authorize()` signature are resolved by the service container.

**Beginner-Friendly Explanation**

Authorization is the "can this user do this?" check. You put your permission logic in the `authorize()` method. If the user is not allowed, Laravel stops the request and returns a "Forbidden" error. You do not have to write any extra code in your controller.

#### Purposes

- To determine if the authenticated user is permitted to make the request.
- To automatically return a 403 response for unauthorized requests.
- To keep authorization logic separate from validation and controller logic.
- To leverage Gates and Policies for complex permission checks.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
public function authorize(): bool
{
    return $this->user() !== null;
}

// With model binding
public function authorize(): bool
{
    $post = $this->route('post');
    return $this->user()->can('update', $post);
}

// With dependency injection
public function authorize(SomeService $service): bool
{
    return $service->check($this->user());
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `$this->user()` | The authenticated user instance. |
| `$this->route('param')` | Access route parameters (including bound models). |
| `$this->can('ability', $model)` | Check against a Gate or Policy. |

**Syntax Rules**

- Must return `bool`.
- Returning `false` triggers a 403 response.
- Can type-hint dependencies for automatic resolution.
- If authorization is handled elsewhere, return `true` or remove the method.

**Constraints and Limitations**

- The `authorize()` method runs before validation, so invalid input does not bypass authorization.
- Route parameters are available via `$this->route()`.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Ownership Check with Route Model Binding**

**Form Request**

```php
public function authorize(): bool
{
    $post = $this->route('post');
    return $this->user()->id === $post->user_id;
}
```

**Controller**

```php
public function update(UpdatePostRequest $request, Post $post)
{
    $post->update($request->validated());
    return redirect()->route('posts.show', $post);
}
```

**Expected Output**

```
// Author → update proceeds
// Non-author → 403 Forbidden
```

**Why This Output Occurs**

The `authorize()` method retrieves the `Post` model via route model binding and checks if the authenticated user's ID matches the post's `user_id`.

---

**Example 2: Gate/Policy Authorization**

```php
public function authorize(): bool
{
    return $this->user()->can('update', $this->route('post'));
}
```

**Expected Output**

```
// If the user passes the 'update' policy → authorized
// If not → 403 Forbidden
```

**Why This Output Occurs**

The `can()` method delegates to the registered Policy for the `Post` model, checking the `update` ability.

---

#### Real-World Cases with Explanation

**Case 1: Admin-Only Resource Management**

An admin panel's `StoreUserRequest` uses `authorize()` to check `$this->user()->hasRole('admin')`. Only admins can create users; other authenticated users receive a 403 response.

**Explanation**: Form Request authorization centralises permission checks for a specific action.


### 5. Form Request Customization

#### Definitions

**Core Definition**

Form Request customization refers to the extension points that allow developers to control input preparation, post-validation normalisation, redirect behaviour, and after-validation logic.

**Technical Definition**

The `FormRequest` class provides several overridable methods and properties: `prepareForValidation()` (sanitise input before validation), `passedValidation()` (normalise data after validation passes), `after()` (custom validation logic after standard rules), `$redirect` / `$redirectRoute` / `$redirectAction` (custom redirect locations), `$errorBag` (named error bag), and `$stopOnFirstFailure` (halt on first failure).

**Beginner-Friendly Explanation**

Customization lets you fine-tune how the Form Request behaves. You can clean up input before checking it, modify data after checking it, redirect to a different page on failure, or run your own custom validation logic.

#### Purposes

- To sanitise or transform input before validation rules are applied.
- To normalise data after validation passes (e.g., hashing passwords, generating slugs).
- To redirect users to a specific page when validation fails.
- To store errors in a named bag for multi-form pages.
- To stop validation on the first failure for performance.
- To add custom validation logic that depends on multiple fields.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Prepare input before validation
protected function prepareForValidation(): void
{
    $this->merge([
        'slug' => Str::slug($this->title),
    ]);
}

// Normalise after validation passes
protected function passedValidation(): void
{
    $this->replace(['name' => 'Taylor']);
}

// Custom redirect on failure
protected $redirect = '/dashboard';
protected $redirectRoute = 'posts.create';

// Named error bag
protected $errorBag = 'createPost';

// Stop on first failure
protected $stopOnFirstFailure = true;

// After-validation hook
public function after(): array
{
    return [
        function ($validator) {
            if ($this->quantity > Product::find($this->product_id)?->stock) {
                $validator->errors()->add('quantity', 'Not enough stock.');
            }
        },
    ];
}
```

**Component Breakdown**

| Method/Property | Description |
|----------------|-------------|
| `prepareForValidation()` | Modifies input before validation. |
| `passedValidation()` | Modifies data after validation passes. |
| `$redirect` | URI to redirect to on failure. |
| `$redirectRoute` | Named route to redirect to on failure. |
| `$errorBag` | Name of the error bag. |
| `$stopOnFirstFailure` | Stop validation on first failure. |
| `after()` | Custom validation logic after standard rules. |

**Syntax Rules**

- `prepareForValidation()` and `passedValidation()` are `protected` methods.
- `$redirect` accepts a URI string; `$redirectRoute` accepts a route name.
- `$stopOnFirstFailure` is a `protected` boolean property.
- `after()` returns an array of closures or invokable classes.

**Constraints and Limitations**

- `passedValidation()` mutations are not reflected in `validated()` or `safe()`; they are reflected in `all()` and `input()`.
- The `$redirect` and `$redirectRoute` properties only apply to web requests.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Preparing Input Before Validation**

**Form Request**

```php
use Illuminate\Support\Str;

protected function prepareForValidation(): void
{
    $this->merge([
        'slug' => Str::slug($this->title),
    ]);
}
```

**Expected Output**

```
// If title is "My First Post" → slug is "my-first-post" before validation
```

**Why This Output Occurs**

The `prepareForValidation()` method merges the generated slug into the request data before the `rules()` method is applied.

---

**Example 2: After-Validation Hook for Multi-Field Logic**

**Form Request**

```php
public function after(): array
{
    return [
        function ($validator) {
            if ($this->shipping_method === 'express'
                && collect($this->items)->sum('quantity') > 50) {
                $validator->errors()->add(
                    'shipping_method',
                    'Express shipping is not available for orders over 50 items.'
                );
            }
        },
    ];
}
```

**Expected Output**

```
// Express shipping with 60 items → error added to shipping_method
// Express shipping with 30 items → validation passes
```

**Why This Output Occurs**

The `after()` closure runs after the standard rules. It checks the combined quantity of all items and adds a custom error if the express shipping limit is exceeded.

---

#### Real-World Cases with Explanation

**Case 1: Slug Generation**

A blog's `StorePostRequest` uses `prepareForValidation()` to generate a URL slug from the title before validation, ensuring the slug is unique and properly formatted.

**Explanation**: `prepareForValidation()` ensures derived fields are available to validation rules.

---

### 6. Form Request Error Messages

#### Definitions

**Core Definition**

Form Request error messages are the human-readable messages displayed to users when validation rules fail, customisable via the `messages()` method.

**Technical Definition**

The `messages()` method returns an array of `field.rule` keyed messages that override Laravel's default messages. The `:attribute` placeholder is replaced with the field name (or a custom attribute name from `attributes()`). Messages can include rule-specific placeholders like `:min`, `:max`, and `:value`.

**Beginner-Friendly Explanation**

Error messages are what users see when they make a mistake. You can customise them to be friendlier or more specific to your application.

#### Purposes

- To provide clear, user-friendly feedback when validation fails.
- To customise messages to match the application's tone and language.
- To support localization and multi-language applications.
- To include rule-specific parameters in messages.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
public function messages(): array
{
    return [
        'title.required' => 'A title is required',
        'body.required'  => 'A message is required',
        'email.unique'   => 'This email is already registered.',
        'password.min'   => 'Your password must be at least :min characters.',
    ];
}
```

**Component Breakdown**

| Placeholder | Description |
|-------------|-------------|
| `:attribute` | The field name (or custom attribute name). |
| `:value` | The invalid value submitted. |
| `:min`, `:max` | Rule parameters (e.g., minimum/maximum length). |

**Syntax Rules**

- Messages are keyed by `field.rule` (e.g., `email.required`).
- The `:attribute` placeholder is replaced with the field name.
- Custom messages override default messages for the specified rule.

**Constraints and Limitations**

- The `:value` placeholder should be used cautiously, as it may expose sensitive input.
- Custom messages must be defined for each `field.rule` combination.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Custom Error Messages**

**Form Request**

```php
public function messages(): array
{
    return [
        'email.required' => 'We need your email address to create your account.',
        'email.email'    => 'That does not look like a valid email address.',
        'email.unique'   => 'An account with this email already exists.',
        'password.min'   => 'Your password must be at least :min characters.',
    ];
}
```

**Expected Output**

```
// If email is missing → "We need your email address to create your account."
// If password is too short → "Your password must be at least 8 characters."
```

**Why This Output Occurs**

The `messages()` method overrides the default messages for the specified `field.rule` combinations.

---

#### Real-World Cases with Explanation

**Case 1: User-Friendly Registration Form**

A registration form uses custom messages like "We need your email address" instead of the default "The email field is required." This makes the form feel more personal and less robotic.

**Explanation**: Custom error messages improve the user experience by providing clear, friendly guidance.


### 7. Form Request Attributes

#### Definitions

**Core Definition**

Form Request attributes are the human-readable names used in error messages to refer to form fields, replacing technical field names via the `attributes()` method.

**Technical Definition**

The `attributes()` method returns an array mapping field names to custom attribute names. The `:attribute` placeholder in error messages is replaced with these names. This is particularly useful for fields with underscores (e.g., `first_name` → `first name`) or technical names (e.g., `dob` → `date of birth`).

**Beginner-Friendly Explanation**

Attributes let you replace technical field names with natural language. Instead of "The first_name field is required," you get "The first name field is required."

#### Purposes

- To improve error message readability by using natural language field names.
- To replace technical field names with human-friendly names.
- To support localization of attribute names.
- To provide consistent field naming across error messages.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
public function attributes(): array
{
    return [
        'email'      => 'email address',
        'first_name' => 'first name',
        'dob'        => 'date of birth',
    ];
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `attributes()` | Returns attribute name mappings. |
| `'email' => 'email address'` | Maps the `email` field to "email address" in messages. |

**Syntax Rules**

- Attribute names are keyed by the field name.
- The `:attribute` placeholder is replaced with the custom attribute name.
- Attributes can be defined per Form Request.

**Constraints and Limitations**

- Attribute names must be defined for each field that needs customisation.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Custom Attribute Names**

**Form Request**

```php
public function attributes(): array
{
    return [
        'email'      => 'email address',
        'first_name' => 'first name',
        'dob'        => 'date of birth',
    ];
}
```

**Expected Output**

- If email is missing → "The email address field is required."
- If first_name is missing → "The first name field is required."
- If dob is invalid → "The date of birth must be a date before today."


**Why This Output Occurs**

The `attributes()` method maps field names to human-readable names, which are substituted into the `:attribute` placeholder in error messages.

---

#### Real-World Cases with Explanation

**Case 1: Multi-Word Field Names**

A form uses field names like `first_name`, `last_name`, and `phone_number`. The `attributes()` method maps these to "first name," "last name," and "phone number," producing natural-sounding error messages.

**Explanation**: Custom attribute names make error messages more readable, especially for fields with underscores or technical naming.


### 8. Form Request Best Practices

#### Definitions

**Core Definition**

Form Request best practices are the recommended guidelines for designing, implementing, and maintaining Form Request classes to maximise maintainability, security, and consistency.

**Technical Definition**

Best practices encompass using Form Requests for non-trivial validation, always using `validated()` for mass assignment, preferring array notation for rules, using `Rule::when()` for conditional validation, using `after()` for multi-field logic, and testing Form Requests thoroughly.

**Beginner-Friendly Explanation**

Best practices are the tips experienced developers follow to keep their Form Requests clean, secure, and easy to maintain.

#### Key Best Practices

1. **Use Form Requests for Any Non-Trivial Validation**: Extract validation and authorization logic into dedicated classes for forms with multiple rules or complex logic.

2. **Always Use `validated()` for Mass Assignment**: Never use `$request->all()` for mass operations. Use `$request->validated()` to ensure only validated data is passed to `create()` or `update()`.

3. **Prefer Array Notation for Rules**: Array syntax (`['required', 'email', Rule::unique('users')]`) is more readable and composes cleanly with `Rule` objects.

4. **Use `Rule::when()` for Conditional Validation**: The `Rule::when()` method provides a clean way to apply rules conditionally.

5. **Use `after()` for Multi-Field Validation**: The `after()` method allows custom validation logic that depends on multiple fields or database state, executed after standard rules.

6. **Use Policies in `authorize()`**: Rather than hardcoding role checks, use `$this->user()->can('ability', $model)` to delegate to Policies.

7. **Customise Messages and Attributes**: Override `messages()` and `attributes()` to provide user-friendly error messages and field names.

8. **Use `prepareForValidation()` to Sanitise Input**: Normalise input (e.g., lowercase emails, generate slugs) before validation rules are applied.

9. **Use `passedValidation()` for Post-Validation Normalisation**: Normalise data (e.g., hash passwords, format dates) after validation passes.

10. **Use Named Error Bags for Multiple Forms**: When multiple forms exist on one page, use `$errorBag` or `validateWithBag()` to prevent error cross-contamination.

11. **Test Form Requests Thoroughly**: Write tests for rules, authorization, custom messages, and edge cases using `assertSessionHasErrors()` and `assertForbidden()`.

12. **Use `stopOnFirstFailure` for Performance**: For requests with many rules, use `protected $stopOnFirstFailure = true` to halt validation on the first failure.

#### Multiple Annotated Complete Step by Step Code Example and Expected Output

**Example 1: Well-Structured Form Request with Best Practices**

**Form Request**

```php
<?php
declare(strict_types=1);

namespace App\Http\Requests\Orders;

use App\Rules\SufficientInventoryRule;
use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Support\Str;
use Illuminate\Validation\Rule;

final class StoreOrderRequest extends FormRequest
{
    protected $stopOnFirstFailure = true;
    protected $errorBag = 'createOrder';

    public function authorize(): bool
    {
        return $this->user()->can('create', Order::class);
    }

    public function rules(): array
    {
        return [
            'customer_email'     => ['required', 'string', 'email:rfc,dns', 'max:255'],
            'items'              => ['required', 'array', 'min:1'],
            'items.*.product_id' => ['required', 'integer', 'exists:products,id'],
            'items.*.quantity'   => ['required', 'integer', 'min:1', 'max:100', new SufficientInventoryRule()],
            'shipping_method'    => ['required', 'string', Rule::in(['standard', 'express'])],
            'express_notes'      => [
                Rule::when($this->input('shipping_method') === 'express', ['required', 'string', 'max:500']),
            ],
        ];
    }

    public function messages(): array
    {
        return [
            'customer_email.required'   => 'We need your email address to process the order.',
            'items.required'            => 'Your order must contain at least one item.',
            'items.*.product_id.exists' => 'One of the selected products is no longer available.',
            'express_notes.required'    => 'Please provide delivery notes for express shipping.',
        ];
    }

    public function attributes(): array
    {
        return [
            'customer_email' => 'email address',
            'items.*.product_id' => 'product',
        ];
    }

    protected function prepareForValidation(): void
    {
        $this->merge([
            'customer_email' => Str::lower($this->customer_email),
        ]);
    }

    public function after(): array
    {
        return [
            function ($validator) {
                if ($this->shipping_method === 'express'
                    && collect($this->items)->sum('quantity') > 50) {
                    $validator->errors()->add(
                        'shipping_method',
                        'Express shipping is not available for orders over 50 items.'
                    );
                }
            },
        ];
    }
}
```

**Controller**

```php
public function store(StoreOrderRequest $request)
{
    $order = Order::create($request->validated());

    return redirect()->route('orders.show', $order)
        ->with('success', 'Order placed successfully.');
}
```

**Expected Output**

- Authorized user with valid data → order created, redirect to orders.show
- Unauthorized user → 403 Forbidden
- Invalid email → "We need your email address to process the order."
- Missing items → "Your order must contain at least one item."
- Product not found → "One of the selected products is no longer available."
- Express shipping with >50 items → custom error added via after() hook

**Why This Output Occurs**

The Form Request encapsulates all validation and authorization logic:
- `authorize()` delegates to a Policy for the `Order` model.
- `rules()` uses array notation and `Rule::when()` for conditional validation.
- `messages()` and `attributes()` provide user-friendly error feedback.
- `prepareForValidation()` normalises the email to lowercase.
- `after()` adds a custom multi-field validation check.
- `$stopOnFirstFailure` halts validation on the first error.
- `$errorBag` stores errors in a named bag.

The controller type-hints `StoreOrderRequest`, which triggers automatic authorization and validation before the method executes. `$request->validated()` ensures only validated data is passed to `Order::create()`.

---

#### Real-World Cases with Explanation

**Case 1: Enterprise E-commerce Platform**

An e-commerce platform uses Form Requests for all order-related validation, with nested array validation for order items, custom rule objects for inventory checks, conditional rules for shipping methods, and after-validation hooks for complex business logic. The validation logic is centralised, testable, and reusable across web and API controllers.

**Explanation**: Following Form Request best practices ensures the platform handles complex order data securely and consistently.


## References

- Validation – Laravel 13.x Documentation — https://laravel.com/docs/13.x/validation
- Validation – Laravel 12.x Documentation — https://laravel.com/docs/12.x/validation
- Validation – Laravel 11.x Documentation — https://laravel.com/docs/11.x/validation
- Validation – Laravel 10.x Documentation — https://laravel.com/docs/10.x/validation
- Validation – Laravel master Documentation — https://laravel.com/framework/docs/master/validation
- FormRequest Class – Laravel API Reference — https://api.laravel.com/docs/12.x/Illuminate/Foundation/Http/FormRequest.html
- Use Form Request Classes – Dutch Laravel Foundation Best Practices — https://github.com/Dutch-Laravel-Foundation/best-practices/blob/main/routing/use-form-request-classes.md
- Form Requests, Array Validation, and Custom Rules – voku/agent-skills (GitHub) — https://github.com/voku/agent-skills/blob/main/skills/laravel-best-practices/rules/validation-rules.md
- Laravel Validation: Preparing Input for Validation – Laravel Documentation — https://laravel.com/docs/validation#preparing-input-for-validation
- Laravel Validation: Customizing the Redirect Location – Laravel Documentation — https://laravel.com/docs/validation#customizing-the-redirect-location
- Laravel Validation: Named Error Bags – Laravel Documentation — https://laravel.com/docs/validation#named-error-bags
- Laravel Validation: Stopping on First Failure – Laravel Documentation — https://laravel.com/docs/validation#stopping-on-first-validation-failure


**Version Note**: This cheat sheet is based on Laravel 10.x through 13.x documentation. Key version-specific features include: the `after()` method for Form Requests (Laravel 10+), the `passedValidation()` method (Laravel 9+), and `prepareForValidation()` (Laravel 6+). The `validateWithBag()` method and named error bags are available in Laravel 8+. The `$stopOnFirstFailure` property is available in Laravel 8+. Custom rule objects implementing `ValidationRule` (with a `validate()` method) are the modern approach. Developers should always consult the documentation for their specific Laravel version.