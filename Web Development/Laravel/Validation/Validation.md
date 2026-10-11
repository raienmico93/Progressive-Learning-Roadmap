# Laravel Validation Fundamentals: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Laravel Validation is the framework's built-in system for validating incoming application data (typically HTTP request input) against a set of defined rules, ensuring that data meets specified constraints before it is processed or persisted.

**Technical Definition**

Laravel's validation system is implemented by the `Illuminate\Validation\Validator` class, which is resolved via the `Validator` facade. Validation rules are defined as arrays of rule strings or `Rule` objects, applied to input data via the `validate()` method on `Illuminate\Http\Request`, the `Validator::make()` factory method, or dedicated `FormRequest` classes. Validation failures throw an `Illuminate\Validation\ValidationException`, which is automatically converted to a redirect with flashed error messages for web requests or a JSON 422 response for AJAX/API requests. The system supports over 100 built-in rules, custom rule objects, conditional rules, nested array validation, custom error messages, and attribute name customization.

**Beginner-Friendly Explanation**

Validation is how you make sure the data people send to your application is correct and safe before you use it. For example, if someone signs up, you want to make sure their email looks like an email, their password is long enough, and their name is not empty. Laravel gives you a simple way to write these rules, and if something is wrong, it automatically sends the user back with helpful error messages. You do not have to write the checking code yourself—you just describe the rules, and Laravel enforces them.

---

### Key Characteristics

- **Declarative Rule Definition**: Rules are defined as readable strings (`'required|email|max:255'`) or array notation (`['required', 'email', 'max:255']`), making validation logic easy to understand and maintain.
- **Multiple Validation Entry Points**: Validation can be performed via the `validate()` method on requests, the `Validator::make()` facade, or dedicated `FormRequest` classes.
- **Automatic Error Handling**: Failed validation automatically redirects web requests back with flashed error messages and a JSON 422 response for AJAX/API requests.
- **100+ Built-in Rules**: Laravel ships with rules for required fields, string length, numeric ranges, email formats, database uniqueness, file uploads, dates, and more.
- **Custom Rules and Messages**: Developers can create custom rule objects, define custom error messages per field and rule, and customize attribute display names.
- **Nested Array Validation**: The wildcard notation (`items.*.id`) allows validation of nested arrays and collections with precise per-element rules.
- **Conditional Validation**: The `Rule::when()` method and conditional rule variants (`required_if`, `prohibited_if`, etc.) enable context-dependent validation logic.
- **Form Request Classes**: Validation logic can be extracted into dedicated form request classes, keeping controllers thin and validation reusable.
- **After Validation Hooks**: The `after()` method allows custom validation logic that depends on multiple fields, executed after the standard rules.

---

### Prerequisites

- **PHP 8.1 or higher** (Laravel 10+) or **PHP 8.2** (Laravel 11+).
- **A Laravel application** installed via Composer.
- **A basic understanding of HTTP requests** and form handling.
- **Familiarity with Blade templates** for displaying error messages.
- **Knowledge of Eloquent models** for database-backed rules (`unique`, `exists`).

---

### Related Programming Areas

- **Form Request Classes**: Dedicated classes for encapsulating validation and authorization logic, keeping controllers thin.
- **Blade Templating**: The `@error` directive and `$errors` variable display validation errors in views.
- **Eloquent ORM**: The `unique` and `exists` rules query the database for uniqueness and existence checks.
- **API Development**: Validation automatically returns JSON 422 responses for API requests, with structured error data.
- **Authorization**: Form Request classes combine validation with authorization via the `authorize()` method.
- **Custom Rule Objects**: Invokable classes implementing `ValidationRule` provide reusable, testable validation logic.

---

## Core Concepts

---

### 1. Validation

#### Definitions

**Core Definition**

Validation is the process of checking whether incoming data (typically from an HTTP request) conforms to a set of defined rules before the application uses or stores it.

**Technical Definition**

Laravel's validation system is built around the `Validator` class and the `ValidatesRequests` trait on the base controller. The `validate()` method on `Illuminate\Http\Request` resolves a validator instance, applies the specified rules to the input data, and throws `ValidationException` if any rule fails. The exception is automatically converted to a redirect with flashed errors for web requests or a JSON 422 response for AJAX/API requests. The `Validator` facade provides a `make()` method for manual validation instance creation.

**Beginner-Friendly Explanation**

Validation is like a checklist. When someone fills out a form and submits it, Laravel checks every field against the rules you set. If everything passes, the data is used. If something fails, Laravel sends the user back to the form with helpful messages telling them what to fix.

#### Purposes

- To ensure that incoming data meets the application's requirements before it is processed or stored.
- To protect the application from invalid, malicious, or malformed data.
- To provide clear, actionable feedback to users when their input is incorrect.
- To enforce data integrity constraints at the application level (complementing database constraints).
- To centralise validation logic in a reusable, testable manner.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Request validate() method
$validated = $request->validate([
    'title' => 'required|unique:posts|max:255',
    'body'  => 'required',
    'email' => 'required|email|unique:users',
]);

// Validator::make() facade
use Illuminate\Support\Facades\Validator;

$validator = Validator::make($request->all(), [
    'title' => 'required|unique:posts|max:255',
    'body'  => 'required',
]);

if ($validator->fails()) {
    return redirect('post/create')
        ->withErrors($validator)
        ->withInput();
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `$request->validate([...])` | Validates request input and returns validated data. |
| `Validator::make($data, $rules)` | Creates a validator instance manually. |
| `$validator->fails()` | Returns `true` if validation failed. |
| `->withErrors($validator)` | Flashes error messages to the session. |
| `->withInput()` | Flashes input data back to the session. |

**Syntax Rules**

- The `validate()` method is available on `Illuminate\Http\Request` via the `ValidatesRequests` trait.
- Rules are defined as an associative array where keys are field names and values are rule strings or arrays.
- Multiple rules are separated by `|` in string notation or as separate array elements.
- The `validate()` method throws `ValidationException` on failure; the `Validator::make()` approach requires manual failure checking.

**Constraints and Limitations**

- The `validate()` method only returns the fields that were validated; unvalidated fields are excluded from the result.
- `$request->all()` returns all input, including unvalidated data—always use `$request->validated()` for mass assignment.
- Closure routes cannot use the `validate()` method directly (it is a controller method).

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Basic Validation with `validate()`**

**Controller**

```php
<?php
// app/Http/Controllers/PostController.php
namespace App\Http\Controllers;

use Illuminate\Http\Request;

class PostController extends Controller
{
    public function store(Request $request)
    {
        // Validate the request data
        $validated = $request->validate([
            'title' => 'required|string|max:255',
            'body'  => 'required|string|min:10',
            'email' => 'required|email|unique:users',
        ]);

        // $validated contains only the validated fields
        Post::create($validated);

        return redirect()->route('posts.index')
            ->with('success', 'Post created successfully.');
    }
}
```

**Route**

```php
// routes/web.php
Route::post('/posts', [PostController::class, 'store']);
```

**Expected Output ()**

- **Success**: Validation passes → post is created, redirect to posts.index with success message
- **Failure**: Validation fails → redirect back to form with error messages and old input

**Why This Output Occurs**

The `validate()` method applies the rules to the request data. If all rules pass, `$validated` contains only the validated fields, which are used to create the post. If any rule fails, `ValidationException` is thrown and automatically converted to a redirect with flashed errors and input.

---

**Example 2: Manual Validation with `Validator::make()`**

**Controller**

```php
<?php
// app/Http/Controllers/PostController.php
namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Validator;

class PostController extends Controller
{
    public function store(Request $request)
    {
        $validator = Validator::make($request->all(), [
            'title' => 'required|unique:posts|max:255',
            'body'  => 'required',
        ]);

        if ($validator->fails()) {
            return redirect('post/create')
                ->withErrors($validator)
                ->withInput();
        }

        // Store the blog post...
        return redirect()->route('posts.index');
    }
}
```

**Expected Output**

- If validation fails → redirect to post/create with errors and input
- If validation passes → proceed to store the post

**Why This Output Occurs**

`Validator::make()` creates a validator instance. `$validator->fails()` returns `true` if any rule fails. The `withErrors()` method flashes the error messages to the session, and `withInput()` flashes the input data so the form can be repopulated.

---

#### Real-World Cases with Explanation

**Case 1: User Registration Form**

A registration form validates `name` (required, string, max 255), `email` (required, email, unique in users table), and `password` (required, confirmed, min 8). The `validate()` method returns only these fields, preventing mass assignment of unintended fields like `is_admin`.

**Explanation**: Validation ensures that user-supplied data meets security and business requirements before it reaches the database.

**Case 2: API Endpoint Validation**

A REST API endpoint for creating orders validates `customer_email` (required, email), `items` (required, array, min 1), and nested fields `items.*.product_id` (required, exists in products) and `items.*.quantity` (required, integer, min 1). Validation failures return a 422 JSON response with structured error messages.

**Explanation**: API validation ensures data integrity across all clients (mobile, web, third-party) and provides consistent error responses.

---

### 2. `validate()`

#### Definitions

**Core Definition**

The `validate()` method is a controller helper provided by the `ValidatesRequests` trait that validates incoming HTTP request data against a set of rules, returning only the validated fields or throwing a `ValidationException` on failure.

**Technical Definition**

`$request->validate(array $rules, array $messages = [], array $attributes = [])` resolves a `Validator` instance via the `Validator` facade, applies the rules to the request's input data, and calls `$validator->validate()`. If validation passes, the method returns the validated data as an array. If validation fails, `ValidationException` is thrown, which Laravel's exception handler converts to an appropriate response (redirect for web, JSON 422 for AJAX/API).

**Beginner-Friendly Explanation**

`validate()` is the quickest way to validate a request. You give it your rules, and it either gives you the clean data back or automatically sends the user back with error messages. You do not have to write any `if` statements or redirect logic yourself.

#### Purposes

- To provide a one-line validation solution in controllers.
- To automatically handle validation failure responses (redirect or JSON).
- To return only the validated fields, preventing mass assignment of unvalidated data.
- To reduce boilerplate by eliminating manual validator instantiation and failure handling.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
$validated = $request->validate([
    'field1' => 'rule1|rule2',
    'field2' => ['rule1', 'rule2'],
], [
    'field1.rule1' => 'Custom message',
], [
    'field1' => 'Custom attribute name',
]);
```

**Component Breakdown**

| Parameter | Description |
|-----------|-------------|
| `$rules` | Associative array of field names to rules. |
| `$messages` (optional) | Custom error messages keyed by `field.rule`. |
| `$attributes` (optional) | Custom attribute names for error messages. |

**Syntax Rules**

- The `validate()` method is available on `Illuminate\Http\Request` in controllers that use the `ValidatesRequests` trait (included in the base `Controller` class).
- It returns an array of validated data.
- It throws `ValidationException` on failure, which is automatically handled by Laravel.
- For AJAX/API requests, a JSON 422 response is returned automatically.

**Constraints and Limitations**

- Only the fields present in the rules array are returned in the validated data.
- The method cannot be used in closure-based routes; use `Validator::make()` instead.
- The method does not support the `after()` validation hook directly; use `Validator::make()` or a `FormRequest` for post-validation logic.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Basic `validate()` Usage**

**Controller**

```php
public function store(Request $request)
{
    $validated = $request->validate([
        'title' => 'required|string|max:255',
        'body'  => 'required|string',
    ]);

    Post::create($validated);

    return redirect()->route('posts.index');
}
```

**Expected Output**

```
// POST with valid data → post created, redirect
// POST with missing title → redirect back with error "The title field is required."
```

**Why This Output Occurs**

The `validate()` method applies the rules. If `title` is missing, the `required` rule fails, and Laravel redirects back with the error message.

---

**Example 2: `validate()` with Custom Messages**

```php
$validated = $request->validate([
    'email' => 'required|email|unique:users',
], [
    'email.required' => 'We need your email address.',
    'email.unique'   => 'This email is already registered.',
]);
```

**Expected Output**

```
// If email is missing → "We need your email address."
// If email is taken → "This email is already registered."
```

**Why This Output Occurs**

The custom messages array overrides the default messages for the specified `field.rule` combinations.

---

#### Real-World Cases with Explanation

**Case 1: Blog Post Creation**

A controller's `store` method uses `$request->validate([...])` to validate the post title and body. If validation passes, the validated data is used to create the post. If it fails, the user is redirected back to the create form with errors.

**Explanation**: `validate()` provides a clean, expressive way to handle validation in controllers without manual error handling.

---

### 3. `Validator::make()`

#### Definitions

**Core Definition**

`Validator::make()` is a factory method on the `Validator` facade that creates a new validator instance for manual validation control.

**Technical Definition**

`Validator::make(array $data, array $rules, array $messages = [], array $attributes = [])` returns a `Validator` instance. The first argument is the data under validation (typically `$request->all()`), and the second is the rules array. The instance provides methods for checking validation status (`fails()`, `passes()`, `validate()`), retrieving errors (`errors()`, `messages()`), and adding after-validation hooks (`after()`).

**Beginner-Friendly Explanation**

`Validator::make()` gives you more control than `validate()`. You create the validator, check if it fails, and decide what to do. It is useful when you need custom redirect logic or want to perform additional checks after validation.

#### Purposes

- To provide manual control over the validation process.
- To allow custom failure handling (e.g., returning specific response types).
- To support after-validation hooks for multi-field logic.
- To enable validation in non-controller contexts (e.g., console commands, jobs).

#### Syntax Rules and Structure

**Complete General Syntax**

```php
use Illuminate\Support\Facades\Validator;

$validator = Validator::make($request->all(), [
    'title' => 'required|unique:posts|max:255',
    'body'  => 'required',
]);

if ($validator->fails()) {
    return redirect('post/create')
        ->withErrors($validator)
        ->withInput();
}

// Validation passed
```

**Component Breakdown**

| Method | Description |
|--------|-------------|
| `Validator::make($data, $rules)` | Creates a validator instance. |
| `$validator->fails()` | Returns `true` if validation failed. |
| `$validator->passes()` | Returns `true` if validation passed. |
| `$validator->errors()` | Returns a `MessageBag` of error messages. |
| `$validator->validate()` | Throws `ValidationException` if failed (automatic redirect). |

**Syntax Rules**

- The first argument is the data array to validate.
- The second argument is the rules array.
- `fails()` and `passes()` provide manual control.
- `validate()` provides automatic redirection on failure.

**Constraints and Limitations**

- Manual instantiation requires more boilerplate than `$request->validate()`.
- The `withErrors()` method is required to flash errors to the session for web requests.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Manual Validation with Custom Failure Handling**

```php
use Illuminate\Support\Facades\Validator;

public function store(Request $request)
{
    $validator = Validator::make($request->all(), [
        'title' => 'required|unique:posts|max:255',
        'body'  => 'required',
    ]);

    if ($validator->fails()) {
        // Custom failure handling
        return response()->json([
            'errors' => $validator->errors(),
        ], 422);
    }

    // Validation passed
    Post::create($validator->validated());

    return response()->json(['message' => 'Post created'], 201);
}
```

**Expected Output**

```json
// On failure: {"errors": {"title": ["The title field is required."]}} with 422 status
// On success: {"message": "Post created"} with 201 status
```

**Why This Output Occurs**

`Validator::make()` creates a validator instance. `$validator->fails()` checks if validation failed. On failure, a custom JSON response is returned instead of the default redirect. On success, `$validator->validated()` returns the validated data.

---

**Example 2: After Validation Hook**

```php
$validator = Validator::make($request->all(), [
    'quantity'   => 'required|integer|min:1',
    'product_id' => 'required|exists:products,id',
]);

$validator->after(function ($validator) use ($request) {
    $product = Product::find($request->product_id);
    if ($product && $request->quantity > $product->stock) {
        $validator->errors()->add('quantity', 'Not enough stock available.');
    }
});

if ($validator->fails()) {
    return redirect()->back()->withErrors($validator)->withInput();
}
```

**Expected Output**

```
// If quantity exceeds stock → "Not enough stock available." error added
// If quantity is within stock → validation passes
```

**Why This Output Occurs**

The `after()` callback runs after the standard validation rules. It adds a custom error to the `quantity` field if the requested quantity exceeds the product's stock.

---

#### Real-World Cases with Explanation

**Case 1: API Validation with Custom Error Structure**

An API controller uses `Validator::make()` to validate request data and return a custom JSON error structure (e.g., `{ "error": { "code": 422, "messages": {...} } }`) instead of Laravel's default structure.

**Explanation**: `Validator::make()` provides the flexibility needed for custom API response formatting.

**Case 2: Multi-Field Business Logic Validation**

An order form validates that the `quantity` does not exceed the `product`'s available stock. The `after()` hook performs this cross-field check after the standard rules pass.

**Explanation**: `after()` hooks handle validation logic that depends on multiple fields or database state.

---

### 4. Validation Rules

#### Definitions

**Core Definition**

Validation rules are the individual constraints applied to form fields, defining what constitutes valid data for each field.

**Technical Definition**

Rules are specified as strings (`'required|email|max:255'`) or as elements in an array (`['required', 'email', 'max:255']`). Laravel provides over 100 built-in rules covering presence, type, length, format, database uniqueness, file uploads, dates, and conditional logic. Rules can also be custom objects implementing `Illuminate\Contracts\Validation\ValidationRule` or `Illuminate\Contracts\Validation\InvokableRule`. The `Rule` facade provides static factory methods for complex rules like `Rule::unique()`, `Rule::in()`, and `Rule::when()`.

**Beginner-Friendly Explanation**

Rules are the specific checks you want Laravel to perform on each field. For example, `required` means the field cannot be empty, `email` means it must look like an email address, and `max:255` means it cannot be longer than 255 characters. You combine rules with a pipe (`|`) to apply multiple checks to one field.

#### Purposes

- To define the specific constraints that valid data must satisfy.
- To provide a declarative, readable way to express validation logic.
- To cover a wide range of validation scenarios (presence, type, format, database, files, dates).
- To enable custom rule creation for domain-specific validation.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// String notation
'title' => 'required|string|max:255|unique:posts',

// Array notation (preferred for complex rules)
'title' => ['required', 'string', 'max:255', Rule::unique('posts')],

// Nested array validation
'items.*.product_id' => 'required|integer|exists:products,id',
'items.*.quantity'   => 'required|integer|min:1|max:100',
```

**Component Breakdown**

| Rule Category | Examples |
|--------------|----------|
| Presence | `required`, `nullable`, `filled`, `present` |
| Type | `string`, `integer`, `numeric`, `boolean`, `array` |
| Length | `min:value`, `max:value`, `between:min,max`, `size:value` |
| Format | `email`, `url`, `ip`, `json`, `regex:pattern` |
| Database | `unique:table,column`, `exists:table,column` |
| Date | `date`, `after:date`, `before:date`, `date_format:format` |
| File | `file`, `image`, `mimes:jpg,png`, `max:2048` |
| Conditional | `required_if`, `prohibited_if`, `required_unless`, `Rule::when()` |
| Comparison | `same:field`, `different:field`, `gt:field`, `lt:field` |

**Syntax Rules**

- Rules are separated by `|` in string notation or as separate array elements.
- Rules with parameters use a colon (`rule:parameter`).
- Multiple parameters are comma-separated (`between:1,10`).
- Array notation is preferred for rules with parameters containing commas or `|` characters (e.g., `regex`).
- The `bail` rule stops validation on the first failure for a field.
- The `nullable` rule allows `null` values to pass validation.

**Constraints and Limitations**

- Some rules require database tables to exist (`unique`, `exists`).
- Rules with the `|` character in parameters (e.g., regex patterns) must use array notation.
- The `required` rule considers `null`, empty string, empty array, and empty uploaded file as "empty."

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Common Validation Rules**

```php
$validated = $request->validate([
    'name'     => 'required|string|max:255',
    'email'    => 'required|email|unique:users,email',
    'age'      => 'required|integer|min:18|max:120',
    'password' => 'required|string|min:8|confirmed',
    'role'     => 'required|in:admin,editor,viewer',
    'avatar'   => 'nullable|image|mimes:jpg,png|max:2048',
    'website'  => 'nullable|url',
    'birthdate'=> 'required|date|before:today',
]);
```

**Expected Output**

```
// Valid data → passes validation, returns validated fields
// Invalid email → "The email must be a valid email address."
// Password too short → "The password must be at least 8 characters."
// Age under 18 → "The age must be at least 18."
```

**Why This Output Occurs**

Each rule applies its specific constraint. Multiple rules for a field are applied in sequence, and the first failure generates an error message.

---

**Example 2: Nested Array Validation with Wildcards**

```php
$validated = $request->validate([
    'items'              => 'required|array|min:1',
    'items.*.product_id' => 'required|integer|exists:products,id',
    'items.*.quantity'   => 'required|integer|min:1|max:100',
    'items.*.price'      => 'required|numeric|min:0',
]);
```

**Expected Output**

```
// Valid items array → passes
// items[0].product_id missing → "The items.0.product_id field is required."
// items[1].quantity is 0 → "The items.1.quantity must be at least 1."
```

**Why This Output Occurs**

The wildcard `*` applies the rules to every element in the `items` array. Error messages include the specific index (e.g., `items.0.product_id`).

---

#### Real-World Cases with Explanation

**Case 1: E-commerce Order Validation**

An order creation endpoint validates:
- `customer_email` (required, email, max 255)
- `items` (required, array, min 1)
- `items.*.product_id` (required, integer, exists:products,id)
- `items.*.quantity` (required, integer, min 1, max 100)
- `shipping_method` (required, in:standard,express)
- `express_notes` (required_if:shipping_method,express, string, max 500)

**Explanation**: Nested array validation ensures that every item in the order is valid, and conditional rules ensure that express shipping requires notes.

**Case 2: User Profile Update**

A profile update form validates `name` (required, string, max 255), `email` (required, email, unique:users,email,{id}), `bio` (nullable, string, max 1000), and `avatar` (nullable, image, mimes:jpg,png, max 2048). The `unique` rule excludes the current user's ID.

**Explanation**: The `unique` rule with an ID exception allows users to keep their existing email while preventing duplicates.

---

### 5. Validation Messages

#### Definitions

**Core Definition**

Validation messages are the human-readable error messages displayed to users when validation rules fail, which can be customized per field and per rule.

**Technical Definition**

Laravel provides default error messages for all built-in rules, stored in `lang/en/validation.php`. Custom messages can be specified as the third argument to `Validator::make()` or `$request->validate()`, or by overriding the `messages()` method in a `FormRequest` class. Messages use the `:attribute` placeholder, which is replaced with the field name (or a custom attribute name). The `:value` placeholder is replaced with the invalid value, and other rule-specific placeholders (e.g., `:min`, `:max`) are replaced with rule parameters.

**Beginner-Friendly Explanation**

Validation messages are the error messages users see when they make a mistake. Laravel has sensible defaults, but you can customize them to be more specific or friendlier. For example, instead of "The email field is required," you can say "Please provide your email address."

#### Purposes

- To provide clear, user-friendly feedback when validation fails.
- To customize error messages to match the application's tone and language.
- To support localization and multi-language applications.
- To include rule-specific parameters (e.g., minimum length) in messages.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Inline custom messages
$request->validate([
    'email' => 'required|email|unique:users',
], [
    'email.required' => 'We need your email address.',
    'email.email'    => 'Please enter a valid email address.',
    'email.unique'   => 'This email is already registered.',
]);
```

```php
// FormRequest messages() method
public function messages(): array
{
    return [
        'title.required' => 'A title is required',
        'body.required'  => 'A message is required',
    ];
}
```

**Component Breakdown**

| Placeholder | Description |
|-------------|-------------|
| `:attribute` | The field name (or custom attribute name). |
| `:value` | The invalid value submitted. |
| `:min`, `:max` | Rule parameters (e.g., minimum/maximum length). |
| `:date` | Date parameter for date rules. |
| `:other` | The other field name for comparison rules. |

**Syntax Rules**

- Messages are keyed by `field.rule` (e.g., `email.required`).
- The `:attribute` placeholder is replaced with the field name or custom attribute name.
- Messages can be defined inline, in a `FormRequest`, or in language files (`lang/{locale}/validation.php`).
- Custom messages override default messages for the specified rule.

**Constraints and Limitations**

- The `:value` placeholder should be used cautiously, as it may expose sensitive input.
- Custom messages must be defined for each `field.rule` combination.
- Language files require the `lang` directory to be published (`php artisan lang:publish`).

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Inline Custom Messages**

```php
$request->validate([
    'email'    => 'required|email|unique:users',
    'password' => 'required|min:8|confirmed',
], [
    'email.required'   => 'We need your email address to create your account.',
    'email.email'      => 'That does not look like a valid email address.',
    'email.unique'     => 'An account with this email already exists.',
    'password.required'=> 'Please choose a password.',
    'password.min'     => 'Your password must be at least :min characters.',
    'password.confirmed' => 'The password confirmation does not match.',
]);
```

**Expected Output**

```
// If email is missing → "We need your email address to create your account."
// If email is invalid → "That does not look like a valid email address."
// If password is too short → "Your password must be at least 8 characters."
```

**Why This Output Occurs**

The custom messages array overrides the default messages for the specified `field.rule` combinations. The `:min` placeholder is replaced with the rule parameter (8).

---

**Example 2: FormRequest Custom Messages**

```php
<?php
// app/Http/Requests/StorePostRequest.php
namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class StorePostRequest extends FormRequest
{
    public function rules(): array
    {
        return [
            'title' => 'required|string|max:255',
            'body'  => 'required|string|min:10',
        ];
    }

    public function messages(): array
    {
        return [
            'title.required' => 'Every post needs a title.',
            'title.max'      => 'Titles cannot be longer than :max characters.',
            'body.required'  => 'Please write some content for your post.',
            'body.min'       => 'Your post must be at least :min characters long.',
        ];
    }
}
```

**Expected Output**

```
// If title is missing → "Every post needs a title."
// If title exceeds 255 chars → "Titles cannot be longer than 255 characters."
// If body is too short → "Your post must be at least 10 characters long."
```

**Why This Output Occurs**

The `messages()` method returns an array of custom messages that override the defaults. The `:max` and `:min` placeholders are replaced with the rule parameters.

---

#### Real-World Cases with Explanation

**Case 1: Localized Validation Messages**

A multi-language application publishes language files for English, Spanish, and French. Validation messages are defined in `lang/es/validation.php` and `lang/fr/validation.php`, and Laravel automatically uses the messages matching the application's current locale.

**Explanation**: Custom messages enable full localization of validation feedback, improving user experience for international audiences.

**Case 2: User-Friendly Registration Messages**

A registration form uses custom messages like "We need your email address" instead of the default "The email field is required." This makes the form feel more personal and less robotic.

**Explanation**: Custom messages improve the user experience by providing clear, friendly guidance.

---

### 6. Validation Attributes

#### Definitions

**Core Definition**

Validation attributes are the human-readable names used in error messages to refer to form fields, replacing the technical field name (e.g., `email` → `email address`, `first_name` → `first name`).

**Technical Definition**

Laravel uses the `:attribute` placeholder in error messages, which is replaced with the field name by default. The `attributes()` method on a `FormRequest` or the fourth argument to `Validator::make()` allows developers to specify custom attribute names. The `attributes` array in `lang/{locale}/validation.php` provides global attribute name customizations.

**Beginner-Friendly Explanation**

When Laravel says "The email field is required," the word "email" is the attribute. If you want it to say "The email address field is required," you can customize the attribute name.

#### Purposes

- To improve error message readability by using natural language field names.
- To replace technical field names (e.g., `first_name`) with human-friendly names (e.g., `first name`).
- To support localization of attribute names alongside messages.
- To provide consistent field naming across error messages.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// FormRequest attributes() method
public function attributes(): array
{
    return [
        'email'     => 'email address',
        'first_name'=> 'first name',
        'dob'       => 'date of birth',
    ];
}
```

```php
// Validator::make() fourth argument
Validator::make($request->all(), $rules, $messages, [
    'email' => 'email address',
]);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `attributes()` | FormRequest method returning attribute name mappings. |
| Fourth argument to `Validator::make()` | Inline attribute name mappings. |
| `lang/{locale}/validation.php` | Global attribute name customizations. |

**Syntax Rules**

- Attribute names are keyed by the field name.
- The `:attribute` placeholder in error messages is replaced with the custom attribute name.
- Attribute names can be defined per FormRequest, inline, or globally.

**Constraints and Limitations**

- Attribute names must be defined for each field that needs customization.
- Global attribute names apply to all validators unless overridden.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: FormRequest Attribute Customization**

```php
<?php
// app/Http/Requests/StoreUserRequest.php
namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class StoreUserRequest extends FormRequest
{
    public function rules(): array
    {
        return [
            'email'      => 'required|email|unique:users',
            'first_name' => 'required|string|max:255',
            'dob'        => 'required|date|before:today',
        ];
    }

    public function attributes(): array
    {
        return [
            'email'      => 'email address',
            'first_name' => 'first name',
            'dob'        => 'date of birth',
        ];
    }
}
```

**Expected Output**

```
// If email is missing → "The email address field is required."
// If first_name is missing → "The first name field is required."
// If dob is invalid → "The date of birth must be a date before today."
```

**Why This Output Occurs**

The `attributes()` method maps field names to human-readable names. The `:attribute` placeholder in error messages is replaced with these names instead of the technical field names.

---

#### Real-World Cases with Explanation

**Case 1: Multi-Word Field Names**

A form uses field names like `first_name`, `last_name`, and `phone_number`. The `attributes()` method maps these to "first name," "last name," and "phone number," producing natural-sounding error messages.

**Explanation**: Custom attribute names make error messages more readable, especially for fields with underscores or technical naming.

---

### 7. Validation Error Handling

#### Definitions

**Core Definition**

Validation error handling is the process by which Laravel detects validation failures, throws exceptions, and converts them into appropriate responses (redirects with flashed errors for web requests, or JSON 422 responses for AJAX/API requests).

**Technical Definition**

When validation fails, Laravel throws `Illuminate\Validation\ValidationException`. The exception's `render()` method checks the request type: for standard HTTP requests, it redirects back to the previous URL with errors flashed to the session and input flashed for repopulation; for AJAX/API requests expecting JSON, it returns a JSON response with a 422 status code containing the error messages. The `ValidationException` class also provides methods for customising the redirect URL and error bag.

**Beginner-Friendly Explanation**

When validation fails, Laravel automatically handles the error for you. If you are using a regular web form, it sends the user back to the form with error messages. If you are building an API, it returns a JSON response with the errors. You do not have to write this logic yourself.

#### Purposes

- To provide automatic, consistent error handling for validation failures.
- To eliminate manual redirect and error-flashing boilerplate.
- To support both web and API response formats seamlessly.
- To allow customisation of the failure response when needed.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Automatic handling via $request->validate()
$validated = $request->validate([...]); // Throws ValidationException on failure

// Manual handling via Validator::make()
if ($validator->fails()) {
    return redirect()->back()->withErrors($validator)->withInput();
}

// Custom redirect URL
$request->validateWithBag('post', [...]);

// Custom redirect URL via ValidationException
throw (new ValidationException($validator))
    ->redirectTo(route('posts.create'));
```

**Component Breakdown**

| Method | Description |
|--------|-------------|
| `ValidationException` | Thrown on validation failure. |
| `$validator->validate()` | Throws `ValidationException` if failed. |
| `->redirectTo()` | Custom redirect URL for web requests. |
| `->status(422)` | Custom HTTP status for JSON responses. |
| `validateWithBag()` | Stores errors in a named error bag. |

**Syntax Rules**

- `$request->validate()` and `$validator->validate()` automatically throw `ValidationException`.
- The exception is handled by Laravel's exception handler.
- Web requests are redirected back with errors and input.
- AJAX/API requests receive a JSON 422 response.
- `redirectTo()` customises the redirect URL for web requests.

**Constraints and Limitations**

- The automatic redirect assumes a previous URL exists; in some cases, a `redirectTo` may be necessary.
- JSON responses are only returned if the request expects JSON (`Accept: application/json` header).

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Automatic Error Handling**

```php
public function store(Request $request)
{
    $validated = $request->validate([
        'title' => 'required|string|max:255',
        'body'  => 'required|string',
    ]);

    Post::create($validated);

    return redirect()->route('posts.index');
}
```

**Expected Output (Web Request)**

```
// POST with invalid data → 302 redirect back with errors and input flashed
// Browser displays: "The title field is required."
```

**Expected Output (API Request)**

```json
{
    "message": "The title field is required. (and 1 more error)",
    "errors": {
        "title": ["The title field is required."],
        "body": ["The body field is required."]
    }
}
```

**Why This Output Occurs**

Laravel's exception handler detects the request type. For web requests, it redirects back. For JSON requests, it returns a 422 response with the error messages.

---

**Example 2: Custom Redirect URL**

```php
use Illuminate\Validation\ValidationException;

public function store(Request $request)
{
    $validator = Validator::make($request->all(), [
        'title' => 'required|unique:posts|max:255',
        'body'  => 'required',
    ]);

    if ($validator->fails()) {
        throw (new ValidationException($validator))
            ->redirectTo(route('posts.create'))
            ->errorBag('createPost');
    }

    // ...
}
```

**Expected Output**

```
// Validation fails → redirect to posts.create with errors in the 'createPost' error bag
```

**Why This Output Occurs**

The `redirectTo()` method customises the redirect URL, and `errorBag()` stores the errors in a named bag for separate handling.

---

#### Real-World Cases with Explanation

**Case 1: API Validation with Consistent JSON Errors**

An API controller uses `$request->validate()`, and validation failures automatically return a JSON 422 response with a structured `errors` object. The frontend can parse this and display errors next to the corresponding form fields.

**Explanation**: Automatic JSON error responses ensure consistent API error handling.

**Case 2: Multi-Form Page with Named Error Bags**

A page contains both a login form and a registration form. Each form uses a named error bag (`login` and `register`), and the Blade template displays errors from the appropriate bag using `@error('email', 'login')`.

**Explanation**: Named error bags prevent error messages from one form appearing in another.

---

### 8. Validation Error Display

#### Definitions

**Core Definition**

Validation error display is the process of rendering validation error messages in Blade views, typically using the `@error` directive or the `$errors` variable.

**Technical Definition**

After a failed validation redirect, Laravel flashes the error messages to the session as a `ViewErrorBag` instance, which is automatically shared with all views as the `$errors` variable. The `@error` Blade directive checks if an error exists for a given field (and optionally an error bag) and provides the `$message` variable within the directive. The `$errors->any()` method checks if any errors exist, and `$errors->all()` returns all error messages.

**Beginner-Friendly Explanation**

After validation fails, Laravel puts the error messages in the session. In your Blade templates, you can use `@error('field')` to show the error message for a specific field, or loop through `$errors` to show all errors at once.

#### Purposes

- To display validation errors to the user in a clear, accessible manner.
- To highlight invalid form fields with conditional styling.
- To show all errors at the top of the form for quick review.
- To provide per-field error messages for precise feedback.

#### Syntax Rules and Structure

**Complete General Syntax**

```blade
{{-- Per-field error --}}
<input type="text" name="email" class="@error('email') is-invalid @enderror">
@error('email')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror

{{-- All errors --}}
@if ($errors->any())
    <div class="alert alert-danger">
        <ul>
            @foreach ($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    </div>
@endif

{{-- Named error bag --}}
@error('email', 'login')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror
```

**Component Breakdown**

| Directive | Description |
|-----------|-------------|
| `@error('field')` | Checks if an error exists for the field. |
| `$message` | The error message (available within `@error`). |
| `@enderror` | Closes the `@error` block. |
| `$errors->any()` | Returns `true` if any errors exist. |
| `$errors->all()` | Returns all error messages. |
| `@error('field', 'bag')` | Checks a named error bag. |

**Syntax Rules**

- The `@error` directive compiles to an `if` statement and supports `@else`.
- The `$errors` variable is automatically available in all views after a validation redirect.
- The second argument to `@error` specifies the error bag name.
- `$errors->first('field')` returns the first error message for a field.

**Constraints and Limitations**

- The `$errors` variable is only populated after a validation redirect; it is empty on a fresh page load.
- Named error bags require the `validateWithBag()` method or `errorBag()` on the exception.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Per-Field Error Display**

```blade
<!-- resources/views/posts/create.blade.php -->
<form method="POST" action="{{ route('posts.store') }}">
    @csrf

    <label for="title">Title</label>
    <input
        type="text"
        name="title"
        id="title"
        value="{{ old('title') }}"
        class="@error('title') is-invalid @enderror"
    >
    @error('title')
        <div class="alert alert-danger">{{ $message }}</div>
    @enderror

    <label for="body">Body</label>
    <textarea
        name="body"
        id="body"
        class="@error('body') is-invalid @enderror"
    >{{ old('body') }}</textarea>
    @error('body')
        <div class="alert alert-danger">{{ $message }}</div>
    @enderror

    <button type="submit">Create Post</button>
</form>
```

**Expected Output (with validation errors)**

```html
<label for="title">Title</label>
<input type="text" name="title" id="title" class="is-invalid">
<div class="alert alert-danger">The title field is required.</div>

<label for="body">Body</label>
<textarea name="body" id="body" class="is-invalid"></textarea>
<div class="alert alert-danger">The body field is required.</div>
```

**Why This Output Occurs**

The `@error('title')` directive checks if an error exists for the `title` field. If so, it renders the error message and adds the `is-invalid` class to the input for styling.

---

**Example 2: Displaying All Errors**

```blade
@if ($errors->any())
    <div class="alert alert-danger">
        <ul>
            @foreach ($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    </div>
@endif

<form method="POST" action="{{ route('posts.store') }}">
    @csrf
    <!-- form fields -->
</form>
```

**Expected Output (with validation errors)**

```html
<div class="alert alert-danger">
    <ul>
        <li>The title field is required.</li>
        <li>The body field is required.</li>
    </ul>
</div>
```

**Why This Output Occurs**

`$errors->any()` returns `true` if any errors exist. `$errors->all()` returns all error messages, which are iterated and displayed in a list.

---

**Example 3: Named Error Bag**

```blade
{{-- Login form --}}
<form method="POST" action="{{ route('login') }}">
    @csrf
    <input type="email" name="email" class="@error('email', 'login') is-invalid @enderror">
    @error('email', 'login')
        <div class="alert alert-danger">{{ $message }}</div>
    @enderror
</form>

{{-- Registration form --}}
<form method="POST" action="{{ route('register') }}">
    @csrf
    <input type="email" name="email" class="@error('email', 'register') is-invalid @enderror">
    @error('email', 'register')
        <div class="alert alert-danger">{{ $message }}</div>
    @enderror
</form>
```

**Expected Output**

```
// Login form errors are scoped to the 'login' bag
// Registration form errors are scoped to the 'register' bag
// No cross-contamination between forms
```

**Why This Output Occurs**

The second argument to `@error` specifies the error bag name. Errors are stored in separate bags, so each form only displays its own errors.

---

#### Real-World Cases with Explanation

**Case 1: Registration Form with Per-Field Errors**

A registration form uses `@error('email')` and `@error('password')` to display error messages next to each field. The input fields use `@error('field') is-invalid @enderror` to apply a red border to invalid fields.

**Explanation**: Per-field error display provides precise, contextual feedback to users.

**Case 2: Multi-Form Page with Named Error Bags**

A page contains both a login form and a registration form. Each form uses a named error bag (`login` and `register`), and the Blade template displays errors from the appropriate bag.

**Explanation**: Named error bags prevent error messages from one form appearing in another.

---

### 9. Validation Best Practices

#### Definitions

**Core Definition**

Validation best practices are the recommended guidelines for designing, implementing, and maintaining validation logic in Laravel applications to maximise security, maintainability, and user experience.

**Technical Definition**

Best practices encompass the use of Form Request classes, the `validated()` method, array notation for rules, custom rule objects, conditional validation, after-validation hooks, nested array validation, and comprehensive testing.

**Beginner-Friendly Explanation**

Best practices are the tips experienced developers follow to keep their validation logic clean, secure, and easy to maintain.

#### Key Best Practices

1. **Use Form Request Classes for Complex Validation**: Extract validation logic into dedicated `FormRequest` classes for anything beyond trivial validation. This keeps controllers thin and validation reusable.

2. **Always Use `validated()` for Mass Assignment**: Never use `$request->all()` for mass operations. Use `$request->validated()` to ensure only validated data is passed to `create()` or `update()`.

3. **Prefer Array Notation for Rules**: Array syntax (`['required', 'email', Rule::unique('users')]`) is more readable and composes cleanly with `Rule` objects, especially for rules with parameters containing commas or `|` characters.

4. **Use `Rule::when()` for Conditional Validation**: The `Rule::when()` method provides a clean way to apply rules conditionally, avoiding complex `if` statements in the rules array.

5. **Use `after()` for Multi-Field Validation**: The `after()` method (available in Form Requests) allows custom validation logic that depends on multiple fields or database state, executed after the standard rules.

6. **Validate Nested Arrays with Wildcards**: Use wildcard notation (`items.*.product_id`) to validate every element in a nested array. Always validate child elements, not just the parent array.

7. **Create Custom Rule Objects for Domain Logic**: For validation logic that is reused or complex, create invokable `ValidationRule` classes. Custom rules are testable and integrate seamlessly with Laravel's validation system.

8. **Customise Messages and Attributes**: Override the `messages()` and `attributes()` methods in Form Requests to provide user-friendly error messages and field names.

9. **Use `bail` to Stop on First Failure**: For fields with multiple rules, add the `bail` rule to stop validation on the first failure, improving performance and providing a single, clear error message.

10. **Test Validation Thoroughly**: Write tests for validation rules, including edge cases (missing fields, invalid formats, boundary values). Use `assertSessionHasErrors()` or `assertJsonValidationErrors()` in feature tests.

11. **Prepare Input Before Validation**: Use the `prepareForValidation()` method in Form Requests to sanitise or transform input (e.g., slug generation) before validation rules are applied.

12. **Use Named Error Bags for Multiple Forms**: When multiple forms exist on the same page, use named error bags (`validateWithBag()`) to prevent error cross-contamination.

#### Multiple Annotated Complete Step by Step Code Example and Expected Output

**Example 1: Well-Structured Form Request with Best Practices**

**Form Request**

```php
<?php
declare(strict_types=1);

namespace App\Http\Requests\Orders;

use App\Rules\SufficientInventoryRule;
use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Validation\Rule;

final class StoreOrderRequest extends FormRequest
{
    public function authorize(): bool
    {
        return $this->user() !== null;
    }

    public function rules(): array
    {
        return [
            'customer_email' => ['required', 'string', 'email:rfc,dns', 'max:255'],
            'coupon_code'    => ['nullable', 'string', 'max:50'],
            'items'          => ['required', 'array', 'min:1'],
            'items.*.product_id' => ['required', 'integer', 'exists:products,id'],
            'items.*.quantity'   => ['required', 'integer', 'min:1', 'max:100', new SufficientInventoryRule()],
            'shipping_method' => ['required', 'string', Rule::in(['standard', 'express'])],
            'express_notes'   => [
                Rule::when($this->input('shipping_method') === 'express', ['required', 'string', 'max:500']),
            ],
        ];
    }

    public function messages(): array
    {
        return [
            'customer_email.required' => 'We need your email address to process the order.',
            'customer_email.email'    => 'Please enter a valid email address.',
            'items.required'          => 'Your order must contain at least one item.',
            'items.*.product_id.exists' => 'One of the selected products is no longer available.',
            'items.*.quantity.max'    => 'You cannot order more than :max of a single item.',
            'express_notes.required'  => 'Please provide delivery notes for express shipping.',
        ];
    }

    public function attributes(): array
    {
        return [
            'customer_email' => 'email address',
            'items.*.product_id' => 'product',
            'items.*.quantity'   => 'quantity',
        ];
    }

    protected function prepareForValidation(): void
    {
        $this->merge([
            'customer_email' => strtolower($this->customer_email),
        ]);
    }

    public function after(): array
    {
        return [
            function ($validator) {
                if ($this->shipping_method === 'express' && $this->items) {
                    $totalQuantity = collect($this->items)->sum('quantity');
                    if ($totalQuantity > 50) {
                        $validator->errors()->add(
                            'shipping_method',
                            'Express shipping is not available for orders over 50 items.'
                        );
                    }
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

```
// Valid order → created, redirect to order show page
// Invalid email → "Please enter a valid email address."
// Missing items → "Your order must contain at least one item."
// Product not found → "One of the selected products is no longer available."
// Express shipping with >50 items → custom error added via after() hook
```

**Why This Output Occurs**

The Form Request encapsulates all validation logic:
- `rules()` defines the validation rules using array notation and `Rule::when()` for conditional validation.
- `messages()` provides user-friendly custom error messages.
- `attributes()` maps technical field names to human-readable names.
- `prepareForValidation()` normalises the email to lowercase before validation.
- `after()` adds a custom multi-field validation check (express shipping quantity limit).

The controller type-hints `StoreOrderRequest`, which triggers automatic validation and authorization before the method executes. `$request->validated()` ensures only validated data is passed to `Order::create()`.

---

#### Real-World Cases with Explanation

**Case 1: Enterprise E-commerce Platform**

An e-commerce platform uses Form Request classes for all order-related validation, with nested array validation for order items, custom rule objects for inventory checks, conditional rules for shipping methods, and after-validation hooks for complex business logic. The validation logic is centralised, testable, and reusable across web and API controllers.

**Explanation**: Following validation best practices ensures the platform handles complex order data securely and consistently.

---

## References

- Validation – Laravel 13.x Documentation — https://laravel.com/docs/13.x/validation
- Validation – Laravel 12.x Documentation — https://laravel.com/docs/12.x/validation
- Validation – Laravel 11.x Documentation — https://laravel.com/docs/11.x/validation
- Validation – Laravel 10.x Documentation — https://laravel.com/docs/10.x/validation
- Validation – Laravel master Documentation — https://laravel.com/framework/docs/master/validation
- Blade Templates: Validation Errors – Laravel 13.x Documentation — https://laravel.com/docs/13.x/blade#validation-errors
- Form Requests, Array Validation, and Custom Rules – voku/agent-skills (GitHub) — https://github.com/voku/agent-skills/blob/main/skills/laravel-best-practices/rules/validation-rules.md
- Use Form Request Classes – Dutch Laravel Foundation Best Practices (GitHub) — https://github.com/Dutch-Laravel-Foundation/best-practices/blob/main/routing/use-form-request-classes.md
- Laravel Custom Validation Rules for Enhanced Data Integrity – Laravel News — https://laravel-news.com/custom-validation-rules
- Show Validation Error Message in Blade – GitHub (itstuffsolutions) — https://github.com/itstuffsolutions/laravel-show-validation-error-message-in-blade
- Laravel Validation Rules List – Laravel Documentation (GitHub) — https://github.com/laravel/docs/blob/master/validation.md
- Validator Facade – Laravel API Reference — https://api.laravel.com/docs/12.x/Illuminate/Validation/Validator.html
- FormRequest Class – Laravel API Reference — https://api.laravel.com/docs/12.x/Illuminate/Foundation/Http/FormRequest.html
- ValidationRule Interface – Laravel API Reference — https://api.laravel.com/docs/12.x/Illuminate/Contracts/Validation/ValidationRule.html
- Rule Facade – Laravel API Reference — https://api.laravel.com/docs/12.x/Illuminate/Validation/Rule.html

---

**Version Note**: This cheat sheet is based on Laravel 10.x through 13.x documentation. Key version-specific features include: the `Rule::when()` method (Laravel 8+), the `after()` method for Form Requests (Laravel 10+), and the `prepareForValidation()` method (Laravel 6+). The `validateWithBag()` method is available in Laravel 8+. The `lang:publish` Artisan command was introduced in Laravel 9. Custom rule objects implementing `ValidationRule` (with a `validate()` method) are the modern approach; the older `Rule` interface with `passes()` and `message()` methods is deprecated but still supported. Developers should always consult the documentation for their specific Laravel version.