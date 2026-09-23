# Laravel Form Request Validation: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Form Requests are dedicated classes that encapsulate validation logic, authorization checks, and input preparation for a specific HTTP request, isolating this logic from controllers.

**Technical Definition:** Form Requests extend `Illuminate\Foundation\Http\FormRequest`, which itself extends `Illuminate\Http\Request`. The class defines a `rules()` method returning the validation rules, an `authorize()` method returning a boolean for permission checks, and optional lifecycle hooks (`prepareForValidation()`, `passedValidation()`). When type-hinted in a controller method, Laravel resolves the Form Request via the service container and runs authorization and validation before the controller action executes.

**Beginner-Friendly Explanation:** Instead of writing validation rules directly in your controller, you create a separate class that handles all the validation for a specific form. When the user submits, Laravel checks the rules, and if anything fails, it automatically redirects them back with errors—all before your controller code even runs.

### Key Characteristics

- **Dedicated Classes:** Validation logic lives in `app/Http/Requests/`.
- **Automatic Resolution:** Type-hinting the Form Request in a controller triggers validation automatically.
- **Authorization First:** The `authorize()` method runs before validation rules.
- **Customisable Messages:** Override default error messages via `messages()` and `attributes()`.
- **Input Lifecycle Hooks:** `prepareForValidation()` sanitises input before validation; `passedValidation()` transforms data after.
- **Validated Data Access:** `$request->validated()` returns only rule-checked fields.

### Prerequisites

- A Laravel application with controllers and routes.
- Basic understanding of HTTP requests and validation rules.
- Familiarity with controller method injection.
- Artisan CLI for generating Form Request classes.

### Related Programming Areas

- **Validation Fundamentals:** Inline validation rules and error display.
- **Service Container:** Form Requests are resolved and injected automatically.
- **Authorization:** Gates and Policies integrate with the `authorize()` method.
- **Middleware:** Form Request validation runs after middleware (e.g., `auth`).

### Core Concepts / Features

1. Generating and Isolating Validation Logic
2. Request Authorization (`authorize()`)
3. Overriding Validation Vocabulary (`messages()`, `attributes()`)
4. Data Lifecycle Hooks (`prepareForValidation()`, `passedValidation()`)

---

## 1. Generating and Isolating Validation Logic

### Definitions

**Core Definition:** A Form Request is a class that isolates validation rules and authorization logic for a specific request, keeping controllers thin and focused on application logic.

**Technical Definition:** Generate a Form Request using `php artisan make:request StorePostRequest`. The class extends `FormRequest` and must implement `rules(): array`. When type-hinted in a controller method, Laravel resolves the Form Request, runs `authorize()`, creates a validator from `rules()`, and either redirects/throws on failure or passes the request to the controller.

**Beginner-Friendly Explanation:** Instead of cluttering your controller with validation rules, you move them into a dedicated class. When the controller method runs, Laravel has already validated the request. If validation fails, the controller never executes.

### Purposes

- To isolate validation logic into a dedicated, testable class.
- To keep controllers focused on application logic rather than input validation.
- To reuse the same validation rules across multiple controllers.
- To automatically run validation before the controller action executes.

### Syntax Rules and Structure

**Complete General Syntax:**

```php
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
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
        ];
    }
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `extends FormRequest` | Base class providing validation lifecycle. |
| `authorize(): bool` | Returns `true`/`false` for permission checks. |
| `rules(): array` | Returns validation rules for the request. |
| `$request->validated()` | Returns only the validated fields. |

**Syntax Rules:**
- Form Request classes live in `app/Http/Requests/`.
- Generate via `php artisan make:request RequestName`.
- The `authorize()` method must return `true` or `false`.
- The `rules()` method must return an array of validation rules.
- Type-hint the Form Request in a controller method to trigger validation.

**Constraints and Limitations:**
- **GET requests:** Form Requests work with any HTTP verb, including GET, but are less common for query-string validation.
- **Authorization:** A `false` return from `authorize()` throws a `403` HTTP exception.
- **Empty rules:** Returning an empty array from `rules()` skips validation entirely.

### Multiple Annotated Complete Code Examples

**Example 1: Basic Form Request**

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
            'title' => 'required|unique:posts|max:255',
            'body' => 'required|min:10',
            'published_at' => 'nullable|date',
        ];
    }
}
```

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
        // Validation already passed; $request->validated() contains only validated fields
        $post = Post::create($request->validated());

        return redirect()->route('posts.show', $post);
    }
}
```

**Expected Output:** On valid submission, the post is created. On invalid submission, the user is redirected back with errors and old input.

**Why:** The type-hinted `StorePostRequest` triggers automatic validation before the `store()` method body executes. `$request->validated()` returns only the fields that passed validation.

---

**Example 2: Validated Data Access**

```php
public function store(StorePostRequest $request)
{
    // Option 1: validated() — returns array of validated data
    $validated = $request->validated();

    // Option 2: safe() — returns ValidatedInput object
    $validated = $request->safe()->only(['title', 'body']);
    $validated = $request->safe()->except(['published_at']);
    $validated = $request->safe()->all();
}
```

**Expected Output:** `$request->validated()` returns an array of only the validated keys. `safe()->only(...)` returns a subset.

**Why:** `validated()` and `safe()` ensure only rule-checked fields are used, preventing attackers from injecting extra columns .

### Real-World Cases

- **Blog Posts:** `StorePostRequest` and `UpdatePostRequest` separate create/update rules.
- **User Registration:** `RegisterRequest` handles name, email, password validation.
- **API Endpoints:** Form Requests validate JSON payloads with `Accept: application/json`.

### References

- Laravel Validation: Form Request Validation — https://laravel.com/docs/12.x/validation#form-request-validation
- Laravel Form Requests (Laravel News) — https://laravel-news.com/form-requests
- Form Request Validation for Filament — https://packagist.org/packages/occ-therapist/form-request-validation-for-filament

---

## 2. Request Authorization (`authorize()`)

### Definitions

**Core Definition:** The `authorize()` method in a Form Request determines whether the authenticated user has permission to perform the requested action.

**Technical Definition:** Before validation rules run, Laravel calls `authorize()`. If it returns `false`, a `Illuminate\Auth\Access\AuthorizationException` is thrown, resulting in a 403 HTTP response. The method can type-hint dependencies (e.g., `Gate`) for complex authorization logic.

**Beginner-Friendly Explanation:** Even if the data is valid, a user might not have permission to perform the action. The `authorize()` method checks this first—before validation even runs. Return `true` to allow, `false` to deny.

### Purposes

- To verify user permissions before processing the request.
- To prevent unauthorized users from reaching validation or controller logic.
- To centralise authorization checks alongside validation rules.
- To integrate with Laravel's Gate and Policy systems.

### Syntax Rules and Structure

**Complete General Syntax:**

```php
public function authorize(): bool
{
    return $this->user()->can('update', $this->route('post'));
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `$this->user()` | The authenticated user instance. |
| `->can('ability', $model)` | Gate/Policy check. |
| `$this->route('post')` | Route parameter (e.g., the `Post` model). |

**Syntax Rules:**
- `authorize()` must return a boolean.
- Returning `false` throws `AuthorizationException` → 403 response.
- Type-hint dependencies (e.g., `Gate`) for complex checks.
- Access route parameters via `$this->route('name')`.

**Constraints and Limitations:**
- **No automatic redirect:** Unlike validation failures, authorization failures return 403, not a redirect.
- **Route model binding:** Route parameters are resolved before `authorize()` runs.

### Multiple Annotated Complete Code Examples

**Example 1: Basic Authorization Check**

```php
public function authorize(): bool
{
    return $this->user() && $this->user()->isAdmin();
}
```

**Expected Output:** Admin users pass; non-admins receive 403.

**Why:** The method checks the authenticated user's role before allowing the request.

---

**Example 2: Policy-Based Authorization**

```php
public function authorize(): bool
{
    return $this->user()->can('update', $this->route('post'));
}
```

**Expected Output:** Users who can update the route-bound `Post` model pass; others receive 403.

**Why:** The `can()` method delegates to the `PostPolicy::update()` method .

### Real-World Cases

- **Post Updates:** Only the post author or an admin can update.
- **Role Assignment:** Only admins can assign roles .
- **Comment Deletion:** Only the comment owner or a moderator can delete.

### References

- Laravel Authorization — https://laravel.com/docs/12.x/authorization
- Validating Role Assignments via Form Request (Laracasts) — https://laracasts.com/index.php/discuss/channels/laravel/validating-role-assignments-via-form-request

---

## 3. Overriding Validation Vocabulary (`messages()` and `attributes()`)

### Definitions

**Core Definition:** The `messages()` method customises error messages for specific rules, while `attributes()` provides human-readable names for fields in error messages.

**Technical Definition:** `messages()` returns an array keyed by `attribute.rule` (e.g., `'email.required' => '...'`). `attributes()` returns an array mapping field names to friendly labels (e.g., `'email' => 'email address'`). These override the default messages in `lang/en/validation.php`.

**Beginner-Friendly Explanation:** Instead of "The email field is required," you can say "We need your email address." `messages()` customises the message, and `attributes()` customises the field name.

### Purposes

- To provide user-friendly error messages instead of technical defaults.
- To rename fields in error messages (e.g., `first_name` → "first name").
- To localise messages for specific attributes or rules.
- To maintain a consistent voice throughout the application.

### Syntax Rules and Structure

**Complete General Syntax:**

```php
public function messages(): array
{
    return [
        'email.required' => 'We need your email address!',
        'title.max' => 'The title must not exceed 255 characters.',
    ];
}

public function attributes(): array
{
    return [
        'email' => 'email address',
        'first_name' => 'first name',
    ];
}
```

**Component Breakdown:**

| Method | Key Format | Purpose |
|--------|-----------|---------|
| `messages()` | `attribute.rule` | Override message for specific rule. |
| `attributes()` | `attribute` | Override field name in messages. |

**Syntax Rules:**
- `messages()` keys use dot notation: `attribute.rule`.
- `attributes()` keys are field names; values are display names.
- The `:attribute` placeholder in default messages is replaced by the value from `attributes()`.

**Constraints and Limitations:**
- **Rule-specific:** `messages()` targets specific rule failures, not the field in general.
- **Global vs. local:** These methods override only for the current Form Request; use language files for global overrides.

### Multiple Annotated Complete Code Examples

**Example 1: Custom Messages and Attributes**

```php
public function messages(): array
{
    return [
        'email.required' => 'We need your email address!',
        'email.email' => 'Please provide a valid email address.',
        'title.max' => 'The title is too long (max 255 characters).',
    ];
}

public function attributes(): array
{
    return [
        'email' => 'email address',
        'title' => 'post title',
    ];
}
```

**Expected Output:**
- Missing email: "We need your email address!"
- Invalid email: "Please provide a valid email address."
- Title too long: "The post title is too long (max 255 characters)."

**Why:** `messages()` targets specific `attribute.rule` combinations. `attributes()` replaces `:attribute` in default messages .

### Real-World Cases

- **User Registration:** Friendly messages for email, password, and name fields.
- **E-commerce Checkout:** Clear messages for card number and expiry fields.
- **API Responses:** Consistent, machine-readable error messages.

### References

- Laravel Validation: Customising Error Messages — https://laravel.com/docs/12.x/validation#customizing-the-error-messages
- Laravel Validation: Specifying Custom Attributes — https://laravel.com/docs/12.x/validation#specifying-custom-attribute-values

---

## 4. Data Lifecycle Hooks (`prepareForValidation()` and `passedValidation()`)

### Definitions

**Core Definition:** `prepareForValidation()` sanitises or augments input before validation runs, while `passedValidation()` transforms data after validation succeeds but before the controller receives it.

**Technical Definition:** `prepareForValidation()` is called by `validated()` before creating the validator. It typically uses `$this->merge()` or `$this->replace()` to modify request data. `passedValidation()` is called after the validator succeeds, before the controller action executes, allowing final data transformations.

**Beginner-Friendly Explanation:** Sometimes input needs cleaning before validation (e.g., trimming spaces, converting a comma-separated string to an array). `prepareForValidation()` does this. After validation passes, you might want to hash a password or format data—`passedValidation()` handles that.

### Purposes

- To normalise input before validation (trim, lowercase, convert formats).
- To merge additional data into the request (e.g., authenticated user's locale).
- To transform validated data after validation (hash passwords, uppercase names).
- To convert value objects or compute derived fields.

### Syntax Rules and Structure

**Complete General Syntax — `prepareForValidation()`:**

```php
protected function prepareForValidation(): void
{
    $this->merge([
        'slug' => Str::slug($this->title),
    ]);
}
```

**Complete General Syntax — `passedValidation()`:**

```php
protected function passedValidation(): void
{
    $this->replace([
        'password' => Hash::make($this->password),
    ]);
}
```

**Component Breakdown:**

| Method | Timing | Purpose |
|--------|--------|---------|
| `prepareForValidation()` | Before validation | Sanitise, merge, normalise. |
| `passedValidation()` | After validation | Transform, hash, format. |

**Syntax Rules:**
- `prepareForValidation()` receives no arguments; use `$this->merge()` or `$this->replace()`.
- `passedValidation()` receives no arguments; use `$this->replace()` to modify data.
- Both methods are `protected` and return `void`.

**Constraints and Limitations:**
- **`prepareForValidation()` does not validate:** It only prepares data; rules run afterward.
- **`passedValidation()` runs before controller:** The controller receives the transformed data.
- **Manual instantiation:** These hooks only run through the normal request lifecycle, not when manually instantiating a Form Request .

### Multiple Annotated Complete Code Examples

**Example 1: `prepareForValidation()` — Normalising Input**

```php
protected function prepareForValidation(): void
{
    $this->merge([
        'slug' => Str::slug($this->title),
        'email' => strtolower($this->email),
    ]);
}
```

**Expected Output:** The `slug` and `email` fields are normalised before validation rules run.

**Why:** `prepareForValidation()` modifies the request data before the validator inspects it .

---

**Example 2: `prepareForValidation()` — Converting String to Array**

```php
protected function prepareForValidation(): void
{
    if (is_string($this->roles)) {
        $this->merge([
            'roles' => array_map('trim', explode(',', $this->roles)),
        ]);
    }
}
```

**Expected Output:** A comma-separated `roles` string is converted to an array, allowing `'roles.*' => 'exists:roles,name'` validation .

**Why:** The input is normalised to match the expected array structure before validation.

---

**Example 3: `passedValidation()` — Hashing Password**

```php
protected function passedValidation(): void
{
    $this->replace([
        'password' => Hash::make($this->password),
    ]);
}
```

**Expected Output:** The validated password is hashed before reaching the controller.

**Why:** `passedValidation()` runs after validation passes, allowing secure transformation of sensitive data .

---

**Example 4: `passedValidation()` — Adding Computed Fields**

```php
protected function passedValidation(): void
{
    $this->merge([
        'total_discount_amount' => $this->calculateDiscount(),
        'processed_at' => now(),
    ]);
}
```

**Expected Output:** Computed fields are added to the validated data before the controller receives it.

**Why:** `passedValidation()` can augment the request with derived data that should not be validated but is needed by the controller.

### Real-World Cases

- **User Registration:** `prepareForValidation()` normalises email; `passedValidation()` hashes the password.
- **Role Assignment:** `prepareForValidation()` converts a string to an array for validation .
- **E-commerce:** `prepareForValidation()` formats discount strings; `passedValidation()` computes totals.
- **Multi-tenant:** `prepareForValidation()` merges the tenant ID from the authenticated user.

### References

- Laravel Form Requests (Laravel News) — https://laravel-news.com/form-requests
- Form Request Validation Tips (Laravel Magazine) — https://laravelmagazine.com/form-request-validation-tips-for-cleaner-controllers
- Validating Role Assignments via Form Request (Laracasts) — https://laracasts.com/index.php/discuss/channels/laravel/validating-role-assignments-via-form-request
- How to Create a Form Request Manually (Stack Overflow) — https://stackoverflow.com/questions/75508715/how-to-create-an-form-request-manually-in-laravel-9

---

## References

- Laravel Validation Documentation (12.x) — https://laravel.com/docs/12.x/validation
- Laravel Validation: Form Request Validation — https://laravel.com/docs/12.x/validation#form-request-validation
- Laravel Validation: Working With Validated Input — https://laravel.com/docs/12.x/validation#working-with-validated-input
- Laravel Validation: Customizing Error Messages — https://laravel.com/docs/12.x/validation#customizing-the-error-messages
- Laravel Validation: Specifying Custom Attributes — https://laravel.com/docs/12.x/validation#specifying-custom-attribute-values
- Laravel FormRequest API (8.x) — https://api.laravel.com/docs/8.x/Illuminate/Foundation/Http/FormRequest.html
- Let's talk about Form Requests (Laravel News) — https://laravel-news.com/form-requests
- Form Request Validation Tips for Cleaner Controllers (Laravel Magazine) — https://laravelmagazine.com/form-request-validation-tips-for-cleaner-controllers
- Validating Role Assignments via Form Request (Laracasts) — https://laracasts.com/index.php/discuss/channels/laravel/validating-role-assignments-via-form-request
- Form Request Validation for Filament — https://packagist.org/packages/occ-therapist/form-request-validation-for-filament
- Laravel Typed Request Package — https://root.packagist.org/packages/pi-space/laravel-typed-request