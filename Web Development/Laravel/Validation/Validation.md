# Laravel Validation Fundamentals: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Validation in Laravel is the process of verifying that incoming HTTP request data meets specified rules before it is passed to application logic, ensuring data integrity and providing user feedback on invalid submissions.

**Technical Definition:** Laravel's validation system is implemented through the `Illuminate\Validation\Validator` class and the `ValidatesRequests` trait. The `$request->validate()` method creates a validator instance, runs the specified rules against the request data, and either returns the validated data or throws an `Illuminate\Validation\ValidationException`. On failure in a web context, this exception is automatically converted to a redirect response with errors flashed to the session; in an AJAX/XHR context, it returns a JSON response with a 422 status code.

**Beginner-Friendly Explanation:** Validation is like a bouncer at a club door—it checks that everyone entering meets the rules (age, dress code, etc.) before letting them in. In Laravel, you tell the bouncer what rules to check (required fields, email format, maximum length) and Laravel automatically handles rejecting invalid data, showing error messages, and sending users back to fix their mistakes.

### Key Characteristics

- **Inline Simplicity:** `$request->validate()` performs validation and automatic redirection in a single method call.
- **Rule Chaining:** Rules are defined as strings (`'required|email|max:255'`) or arrays for complex configurations.
- **Automatic Redirection:** Failed validation automatically redirects web requests back with errors and old input preserved.
- **XHR Awareness:** AJAX requests receive a 422 JSON response instead of a redirect.
- **Blade Integration:** The `$errors` variable and `@error` directive provide ready-to-use error display.
- **Old Input Preservation:** The `old()` helper restores previous input values after a failed submission.

### Prerequisites

- A Laravel application with routes and controllers.
- Basic understanding of HTTP requests and form submissions.
- Blade templating engine for displaying errors.
- Familiarity with controller methods and request handling.

### Related Programming Areas

- **Form Requests:** Class-based validation for complex scenarios (separate from this cheat sheet).
- **Blade Templates:** Error display and old input restoration.
- **HTTP Responses:** Redirection and JSON responses on validation failure.
- **Session Management:** Errors and old input are stored in the session.
- **Middleware:** `TrimStrings` and `ConvertEmptyStringsToNull` affect validation input.

### Core Concepts / Features

1. The Validation Lifecycle (`$request->validate()`)
2. Standard Validation Rules (Structural & Data Types)
3. Formatting and Length Constraints
4. Displaying Errors in Blade (`@error`, `$errors`)
5. Automatic Redirection and `old()` Helper

---

## 1. The Validation Lifecycle: Inline Controller Validation

### Definitions

**Core Definition:** The validation lifecycle refers to the sequence of events that occur when `$request->validate()` is called: rule evaluation, failure handling, redirection, and error/session storage.

**Technical Definition:** The `ValidatesRequests::validate()` method accepts a rules array and optional custom messages/attributes. It creates a `Validator` instance via the `ValidationFactory`. If validation fails, a `ValidationException` is thrown with the validator instance. The exception handler converts this to a redirect (web) or JSON response (XHR). On success, the validated data array is returned.

**Beginner-Friendly Explanation:** When you call `$request->validate()`, Laravel checks the data against your rules. If something fails, it automatically sends the user back to the form with error messages and their previous input. If everything passes, it returns the validated data so you can save it.

### Purposes

- To verify request data meets application requirements before processing.
- To automatically redirect users back with errors on failure.
- To return clean, validated data for safe persistence.
- To provide a single method call for validation and error handling.
- To support AJAX requests with JSON error responses.

### Syntax Rules and Structure

**Complete General Syntax:**

```php
$validated = $request->validate([
    'field' => 'rule1|rule2|rule3',
    'another_field' => ['rule1', 'rule2'],
]);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `$request->validate([...])` | Runs validation; redirects on failure. |
| `'field' => 'rule1|rule2'` | String syntax for simple rules. |
| `['rule1', 'rule2']` | Array syntax for rules requiring parameters. |
| `$validated` | Array of only the validated fields on success. |

**Syntax Rules:**
- The method must be called on an `Illuminate\Http\Request` instance.
- Rules are passed as an associative array: field names as keys, rules as values.
- On success, only the validated keys are returned (safe for mass assignment).
- On failure, a `ValidationException` is thrown automatically.
- For AJAX requests, a JSON response with 422 status is returned instead of a redirect.

**Constraints and Limitations:**
- **No custom error format:** `$request->validate()` uses default error formatting; use Form Requests for customisation.
- **Named error bags:** `$request->validate()` does not support named error bags; use `Validator::make()->validateWithBag()` for multiple forms on one page.
- **XHR behaviour:** AJAX requests do not redirect; they receive JSON. Ensure your frontend handles 422 responses.

### Multiple Annotated Complete Code Examples

**Example 1: Basic Inline Validation**

```php
<?php
namespace App\Http\Controllers;

use Illuminate\Http\Request;

class PostController extends Controller
{
    public function store(Request $request)
    {
        // Validate the request data
        $validated = $request->validate([
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
            'published_at' => 'nullable|date',
        ]);

        // If validation passes, $validated contains only the validated fields
        $post = Post::create($validated);

        return redirect()->route('posts.show', $post);
    }
}
```

**Expected Output:**
- Valid submission: Post is created and user redirected to the post.
- Invalid submission: User is redirected back to the form with errors and old input.
- AJAX submission: JSON response with `{"message": "...", "errors": {...}}` and 422 status.

**Why:** `$request->validate()` performs validation and automatically handles the failure response. The `$validated` array contains only the fields that passed validation, making it safe for `create()`.

---

**Example 2: Array Syntax for Rules with Parameters**

```php
<?php
public function update(Request $request, Post $post)
{
    $validated = $request->validate([
        'title' => ['required', 'max:255', Rule::unique('posts')->ignore($post->id)],
        'body' => ['required'],
        'status' => ['required', Rule::in(['draft', 'published', 'archived'])],
    ]);

    $post->update($validated);

    return back()->with('success', 'Post updated successfully.');
}
```

**Expected Output:** Validation runs against the rules. On success, the post is updated. On failure, redirection with errors.

**Why:** Array syntax allows rule objects (like `Rule::unique()` and `Rule::in()`) to be used alongside string rules. This is required for rules that accept parameters via fluent methods.

### Real-World Cases

- **User Registration:** Validate `name`, `email` (unique), `password` (confirmed) before creating the user.
- **Blog Post Creation:** Validate `title`, `slug` (unique), `body`, `category_id` (exists) before saving.
- **Profile Updates:** Validate optional fields with `nullable` and `sometimes` for partial updates.
- **API Endpoints:** Validate JSON payloads before processing, returning 422 on failure.

### References

- Laravel Validation Documentation — https://laravel.com/docs/12.x/validation
- Laravel Validation: Form Request Validation — https://laravel.com/docs/12.x/validation#form-request-validation

---

## 2. Standard Validation Rules: Structural & Data Types

### Definitions

**Core Definition:** Structural rules determine whether a field is present and required, while data-type rules verify that a field's value is of the expected type (string, numeric, boolean, date).

**Technical Definition:** Structural rules include `required`, `nullable`, and `sometimes`. `required` fails if the field is absent or empty. `nullable` allows `null` values. `sometimes` skips validation if the field is absent. Data-type rules include `string`, `numeric`, `integer`, `boolean`, `date`, `array`, and `json`.

**Beginner-Friendly Explanation:** Structural rules answer "does this field exist and is it allowed to be empty?" Data-type rules answer "is this value actually a number/date/string?" For example, `'age' => 'required|integer'` means the field must be present and must be a whole number.

### Purposes

- To enforce the presence or absence of fields using `required`, `nullable`, `sometimes`.
- To verify values match expected PHP data types using `string`, `numeric`, `boolean`, `date`.
- To conditionally validate fields based on other input using `sometimes`.
- To allow optional fields that may be omitted from the request.

### Syntax Rules and Structure

**Complete General Syntax — Structural Rules:**

```php
$request->validate([
    'name' => 'required|string|max:255',
    'bio' => 'nullable|string',
    'newsletter' => 'sometimes|boolean',
]);
```

**Complete General Syntax — Data Types:**

```php
$request->validate([
    'age' => 'required|integer|min:18',
    'price' => 'required|numeric|between:0,9999.99',
    'is_active' => 'required|boolean',
    'birth_date' => 'required|date',
]);
```

**Component Breakdown:**

| Rule | Description |
|------|-------------|
| `required` | Field must be present and not empty. |
| `nullable` | Field may be `null`. |
| `sometimes` | Run validation only if field is present. |
| `string` | Value must be a string. |
| `numeric` | Value must be numeric (int or float). |
| `integer` | Value must be an integer. |
| `boolean` | Value must be `true`, `false`, `1`, `0`, `"1"`, `"0"`. |
| `date` | Value must be a valid date. |
| `array` | Value must be a PHP array. |

**Syntax Rules:**
- Rules are separated by `|` in string syntax or commas in array syntax.
- `required` implicitly includes `sometimes` behaviour (fails if absent).
- `nullable` only allows `null`; it does not skip other rules for non-null values.
- `sometimes` is useful for PATCH requests where fields may be omitted.
- `boolean` accepts `true`, `false`, `1`, `0`, `"1"`, `"0"` but not `"true"`/`"false"`.

**Constraints and Limitations:**
- **`required` vs `nullable`:** `required` rejects empty strings, `null`, empty arrays; `nullable` only allows `null`.
- **`sometimes` in `$request->validate()`:** Works as a rule string, but for complex conditional logic, use `Validator::sometimes()`.
- **`date` is permissive:** The `date` rule accepts many formats; use `date_format` for strict format validation.

### Multiple Annotated Complete Code Examples

**Example 1: Required, Nullable, and Sometimes**

```php
<?php
$request->validate([
    'name' => 'required|string|max:255',      // Must be present
    'bio' => 'nullable|string|max:1000',      // May be null
    'newsletter' => 'sometimes|boolean',      // Validated only if present
]);
```

**Expected Output:**
- Missing `name`: Validation fails with "The name field is required."
- `bio` is `null`: Validation passes.
- `newsletter` is absent: Validation passes (skipped).
- `newsletter` is `"yes"`: Validation fails (not a boolean).

**Why:** `required` enforces presence, `nullable` allows `null`, and `sometimes` skips validation when the field is absent.

---

**Example 2: Data Type Rules**

```php
<?php
$request->validate([
    'age' => 'required|integer|min:18|max:120',
    'price' => 'required|numeric|min:0',
    'is_admin' => 'required|boolean',
    'birth_date' => 'required|date',
    'tags' => 'nullable|array',
]);
```

**Expected Output:**
- `age = "25"`: Passes (numeric strings are accepted for `integer`).
- `age = "abc"`: Fails.
- `price = "19.99"`: Passes.
- `is_admin = "true"`: **Fails** (only `1`, `0`, `"1"`, `"0"`, `true`, `false` are valid booleans).
- `birth_date = "2024-01-15"`: Passes.

**Why:** Data-type rules validate the PHP type of the input. `integer` accepts numeric strings but not non-numeric strings. `boolean` is strict about accepted values.

### Real-World Cases

- **User Profiles:** `name` (required), `bio` (nullable), `website` (sometimes|url).
- **E-commerce Products:** `price` (required|numeric), `stock` (required|integer|min:0), `on_sale` (required|boolean).
- **Event Registration:** `start_date` (required|date|after:today), `end_date` (required|date|after:start_date).
- **API Updates:** PATCH requests use `sometimes` to allow partial updates.

### References

- Laravel Validation: Available Rules — https://laravel.com/docs/12.x/validation#available-validation-rules
- Laravel Validation: Rule Objects — https://laravel.com/docs/12.x/validation#using-rule-objects

---

## 3. Formatting and Length Constraints

### Definitions

**Core Definition:** Formatting and length constraints verify that a field's value matches a specific pattern (email, URL) or falls within acceptable size boundaries (min, max, digits).

**Technical Definition:** Length rules include `min`, `max`, `size`, `between`, `digits`, `digits_between`. Format rules include `email`, `url`, `alpha`, `alpha_num`, `alpha_dash`, `regex`. These rules apply to strings (character count), numerics (value), arrays (element count), and files (kilobytes).

**Beginner-Friendly Explanation:** These rules answer "is this the right format?" and "is this the right length?" For example, `'email' => 'required|email|max:255'` means the email must be a valid email address and no longer than 255 characters.

### Purposes

- To enforce maximum and minimum string lengths using `max` and `min`.
- To validate exact digit counts (e.g., credit card CVV) using `digits`.
- To verify email and URL formats using `email` and `url`.
- To enforce alphanumeric or alphabetic-only input using `alpha`, `alpha_num`, `alpha_dash`.
- To apply custom regex patterns using `regex`.

### Syntax Rules and Structure

**Complete General Syntax — Length:**

```php
$request->validate([
    'username' => 'required|string|min:3|max:20',
    'bio' => 'nullable|string|max:500',
    'pin' => 'required|digits:4',
    'quantity' => 'required|integer|between:1,100',
]);
```

**Complete General Syntax — Format:**

```php
$request->validate([
    'email' => 'required|email|max:255',
    'website' => 'nullable|url',
    'slug' => 'required|alpha_dash',
    'code' => ['required', 'regex:/^[A-Z]{2}-\d{4}$/'],
]);
```

**Component Breakdown:**

| Rule | Description |
|------|-------------|
| `min:n` | Minimum value/length. |
| `max:n` | Maximum value/length. |
| `between:a,b` | Value between `a` and `b`. |
| `digits:n` | Exactly `n` digits. |
| `email` | Valid email address. |
| `url` | Valid URL. |
| `alpha` | Only alphabetic characters. |
| `alpha_num` | Only alphanumeric characters. |
| `alpha_dash` | Alphanumeric, dashes, underscores. |
| `regex:pattern` | Matches the given regex. |

**Syntax Rules:**
- `min`/`max` apply to strings (character count), numerics (value), arrays (element count), files (kilobytes).
- `digits:n` requires exactly `n` digits (not a numeric value with `n` digits, but the string length).
- `email` uses PHP's `filter_var()` with `FILTER_VALIDATE_EMAIL`.
- `regex` patterns must be valid PCRE; delimiters are required (e.g., `/.../`).
- When using `regex` with array syntax, the pattern must be a complete regex string.

**Constraints and Limitations:**
- **`min`/`max` context-dependent:** For numerics, `min`/`max` check value; for strings, they check length. This can be confusing.
- **`email` vs `email:rfc,dns`:** The default `email` rule uses `filter_var`; stricter validation requires `email:rfc,dns`.
- **`regex` risk:** Complex regex can be slow or cause catastrophic backtracking. Keep patterns simple.
- **`url` permissiveness:** The `url` rule accepts many schemes; use `url:http,https` to restrict.

### Multiple Annotated Complete Code Examples

**Example 1: Length Constraints**

```php
<?php
$request->validate([
    'username' => 'required|string|min:3|max:20',
    'bio' => 'nullable|string|max:500',
    'cv_code' => 'required|digits:3',
    'rating' => 'required|integer|between:1,5',
]);
```

**Expected Output:**
- `username = "ab"`: Fails (too short).
- `username = "abc"`: Passes.
- `cv_code = "123"`: Passes.
- `cv_code = "1234"`: Fails (must be exactly 3 digits).
- `rating = 3`: Passes.

**Why:** `min`/`max` on strings check character count. `digits:3` requires exactly three digit characters. `between` checks numeric range.

---

**Example 2: Format Constraints**

```php
<?php
$request->validate([
    'email' => 'required|email|max:255',
    'website' => 'nullable|url|max:255',
    'slug' => 'required|alpha_dash|max:100',
    'product_code' => ['required', 'regex:/^[A-Z]{3}-\d{4}$/'],
]);
```

**Expected Output:**
- `email = "user@example.com"`: Passes.
- `email = "not-an-email"`: Fails.
- `website = "https://example.com"`: Passes.
- `slug = "my-post-title"`: Passes.
- `slug = "my post title"`: Fails (spaces not allowed).
- `product_code = "ABC-1234"`: Passes.

**Why:** `email` validates format via `filter_var`. `alpha_dash` allows letters, numbers, dashes, and underscores. `regex` enforces the exact pattern.

### Real-World Cases

- **User Registration:** `username` (min:3|max:20), `email` (email|max:255), `password` (min:8).
- **E-commerce Checkout:** `card_number` (digits:16), `cvv` (digits:3), `expiry` (date_format:m/y).
- **Content Management:** `slug` (alpha_dash|unique:posts), `title` (max:255).
- **API Validation:** `url` (url), `json_payload` (json).

### References

- Laravel Validation: Rule Reference — https://laravel.com/docs/12.x/validation#available-validation-rules
- Laravel Validation: Regex Rule — https://laravel.com/docs/12.x/validation#rule-regex

---

## 4. Displaying Validation Errors in Blade

### Definitions

**Core Definition:** Blade provides two mechanisms for displaying validation errors: the `@error` directive for field-specific errors and the global `$errors` variable for accessing all errors.

**Technical Definition:** The `$errors` variable is an instance of `Illuminate\Support\MessageBag`, automatically shared with all views after a failed validation redirect. The `@error('field')` directive checks `$errors->has('field')` and provides the `$message` variable within its block. Named error bags are accessible via `$errors->{bagName}`.

**Beginner-Friendly Explanation:** After a failed form submission, Laravel automatically gives your view an `$errors` object containing all error messages. You can display errors next to each field using `@error('field_name')` or check `$errors->has('field')` for conditional styling.

### Purposes

- To display field-specific error messages using `@error`.
- To conditionally style form inputs with error classes using `@error`/`@else`.
- To access all error messages using the `$errors` variable.
- To support named error bags for multiple forms on one page.
- To provide immediate user feedback on invalid submissions.

### Syntax Rules and Structure

**Complete General Syntax — `@error` Directive:**

```blade
<input type="email" name="email" value="{{ old('email') }}">

@error('email')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `@error('email')` | Checks if the `email` field has an error. |
| `$message` | The first error message for the field. |
| `@enderror` | Closes the directive. |

**Complete General Syntax — `$errors` Variable:**

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
```

**Complete General Syntax — Conditional Styling:**

```blade
<input type="email" name="email"
       class="form-control @error('email') is-invalid @enderror"
       value="{{ old('email') }}">

@error('email')
    <span class="invalid-feedback">{{ $message }}</span>
@enderror
```

**Complete General Syntax — Named Error Bags:**

```blade
{{ $errors->login->first('email') }}
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `$errors->any()` | Returns `true` if any errors exist. |
| `$errors->has('field')` | Returns `true` if the field has errors. |
| `$errors->first('field')` | First error message for the field. |
| `$errors->get('field')` | All error messages for the field. |
| `$errors->all()` | All error messages. |

**Syntax Rules:**
- `$errors` is automatically available in all views after a failed validation redirect.
- `@error('field')` only displays if the field has at least one error.
- `$message` within `@error` is the first error message for that field.
- Named error bags are accessed via `$errors->{bagName}`.
- Use `old('field')` to repopulate input values.

**Constraints and Limitations:**
- **`$errors` is empty on GET requests:** It only contains errors after a failed validation redirect.
- **Multiple forms:** Use named error bags (`withErrors($validator, 'login')`) to avoid conflicts.
- **Array fields:** Use dot notation and wildcards for nested fields (`$errors->first('items.0.name')`).

### Multiple Annotated Complete Code Examples

**Example 1: Basic `@error` Usage**

```blade
<form action="/register" method="POST">
    @csrf

    <div>
        <label>Email</label>
        <input type="email" name="email" value="{{ old('email') }}">
        @error('email')
            <div class="error">{{ $message }}</div>
        @enderror
    </div>

    <div>
        <label>Password</label>
        <input type="password" name="password">
        @error('password')
            <div class="error">{{ $message }}</div>
        @enderror
    </div>

    <button type="submit">Register</button>
</form>
```

**Expected Output:**
- Valid submission: No errors displayed.
- Invalid submission: Error messages appear below each invalid field. Email field retains its value via `old('email')`.

**Why:** `@error('email')` checks `$errors->has('email')`. If true, `$message` contains the first error message for that field.

---

**Example 2: Conditional Styling with `@error`**

```blade
<input type="email" name="email"
       class="form-control @error('email') is-invalid @enderror"
       value="{{ old('email') }}">

@error('email')
    <span class="invalid-feedback">{{ $message }}</span>
@enderror
```

**Expected Output:** The input receives the `is-invalid` CSS class when validation fails, and the error message displays in a styled `<span>`.

**Why:** `@error('email')` conditionally adds the `is-invalid` class. The `@error` block outputs the message with Bootstrap-compatible styling.

---

**Example 3: Displaying All Errors**

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
```

**Expected Output:** A single alert box listing all validation errors.

**Why:** `$errors->any()` checks if any errors exist. `$errors->all()` returns all error messages as a flat array.

---

**Example 4: Named Error Bags**

```php
// Controller
return redirect('register')->withErrors($validator, 'login');
```

```blade
{{ $errors->login->first('email') }}
```

**Expected Output:** The error message from the `login` error bag is displayed, isolated from other error bags.

**Why:** Named bags prevent conflicts when multiple forms exist on one page. Access via `$errors->{bagName}`.

### Real-World Cases

- **User Registration:** Display errors for `name`, `email`, `password` fields.
- **Login Forms:** Use named error bag (`login`) to separate from registration errors.
- **Multi-Step Forms:** Display errors for the current step only.
- **API Validation:** JSON responses include errors; Blade displays them after redirect.

### References

- Laravel Validation: Displaying Errors — https://laravel.com/docs/12.x/validation#displaying-the-validation-errors
- Laravel Validation: Named Error Bags — https://laravel.com/docs/12.x/validation#named-error-bags
- Laravel Blade: `@error` Directive — https://laravel.com/docs/12.x/blade#the-error-directive

---

## 5. Automatic Redirection and `old()` Helper

### Definitions

**Core Definition:** Automatic redirection refers to Laravel's built-in behaviour of redirecting users back to the form page when validation fails, while the `old()` helper restores previously submitted input values.

**Technical Definition:** On validation failure in a web context, a `ValidationException` is thrown. The exception handler converts it to a redirect response to the previous URL with errors flashed to the session via `withErrors()` and input flashed via `withInput()`. The `old()` helper retrieves flashed input from the session. For AJAX/XHR requests, no redirect occurs; a JSON response with 422 status is returned instead.

**Beginner-Friendly Explanation:** When validation fails, Laravel automatically sends the user back to the form with their previous input still filled in and error messages displayed. You don't have to write any redirect code—it's all automatic. The `old()` helper remembers what the user typed.

### Purposes

- To automatically redirect users back to the form on validation failure.
- To preserve previously entered input using the `old()` helper.
- To provide a seamless user experience without losing form data.
- To return JSON responses for AJAX requests instead of redirects.
- To allow custom redirection via `withInput()` and `withErrors()`.

### Syntax Rules and Structure

**Complete General Syntax — Automatic Redirection:**

```php
$request->validate([
    'title' => 'required|max:255',
]);

// On failure: automatic redirect back with errors + old input
```

**Complete General Syntax — `old()` Helper:**

```blade
<input type="text" name="title" value="{{ old('title') }}">

<!-- With default value -->
<input type="text" name="title" value="{{ old('title', 'Default Title') }}">
```

**Complete General Syntax — Custom Redirect:**

```php
if ($validator->fails()) {
    return redirect('post/create')
        ->withErrors($validator)
        ->withInput();
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| Automatic redirect | Built into `$request->validate()`. |
| `old('field')` | Retrieves flashed input from session. |
| `old('field', $default)` | Returns default if no old input exists. |
| `withInput()` | Flashes all input to session. |
| `withErrors($validator)` | Flashes errors to session. |

**Syntax Rules:**
- Automatic redirection only works with `$request->validate()` or `$validator->validate()`.
- For XHR requests, a JSON response with 422 status is returned instead of a redirect.
- `old()` works for all input types: text, email, textarea, select, checkbox, radio.
- For checkboxes, use `old('field')` with a conditional `checked` attribute.
- The `withInput()` method accepts an optional array to flash specific input.

**Constraints and Limitations:**
- **XHR behaviour:** AJAX requests do not redirect; they receive JSON. Frontend must handle 422 responses.
- **`old()` only works after redirect:** It reads from the flashed session data; it does not persist across multiple redirects.
- **File inputs:** `old()` does not restore file inputs (browsers do not allow it for security reasons).
- **API routes:** `$request->validate()` still redirects for non-XHR requests even on API routes; use `$request->expectsJson()` or Form Requests for pure APIs.

### Multiple Annotated Complete Code Examples

**Example 1: Automatic Redirection with `old()`**

```php
<?php
// Controller
public function store(Request $request)
{
    $validated = $request->validate([
        'title' => 'required|max:255',
        'body' => 'required',
    ]);

    // If validation fails, redirect happens automatically
    Post::create($validated);

    return redirect()->route('posts.index');
}
```

```blade
<!-- Blade form -->
<form action="{{ route('posts.store') }}" method="POST">
    @csrf
    <input type="text" name="title" value="{{ old('title') }}">
    @error('title') <span>{{ $message }}</span> @enderror

    <textarea name="body">{{ old('body') }}</textarea>
    @error('body') <span>{{ $message }}</span> @enderror

    <button type="submit">Create Post</button>
</form>
```

**Expected Output:**
- Valid submission: Post created, redirected to index.
- Invalid submission: Redirected back to form. `title` and `body` fields retain their values. Errors displayed.

**Why:** Laravel automatically redirects on validation failure and flashes input to the session. The `old()` helper retrieves the flashed values.

---

**Example 2: Checkbox and Select with `old()`**

```blade
<!-- Checkbox -->
<input type="checkbox" name="is_active" value="1"
       @if(old('is_active')) checked @endif>

<!-- Select -->
<select name="category_id">
    <option value="">Select Category</option>
    @foreach($categories as $category)
        <option value="{{ $category->id }}"
                @if(old('category_id') == $category->id) selected @endif>
            {{ $category->name }}
        </option>
    @endforeach
</select>
```

**Expected Output:** The checkbox remains checked and the selected option remains selected after a failed submission.

**Why:** `old('is_active')` returns the submitted value (`1` if checked). The `@if` condition checks truthiness. For selects, comparison with `==` handles type differences.

---

**Example 3: XHR/AJAX Behaviour**

```php
// JavaScript (axios)
axios.post('/posts', formData)
    .then(response => {
        // Success
    })
    .catch(error => {
        if (error.response.status === 422) {
            // Validation errors in error.response.data.errors
            const errors = error.response.data.errors;
            // Display errors manually
        }
    });
```

**Expected Output:** AJAX request receives `422 Unprocessable Entity` with JSON body: `{"message": "...", "errors": {"title": ["The title field is required."]}}`.

**Why:** Laravel detects XHR requests via the `X-Requested-With` header (or `Accept: application/json`). It returns JSON instead of redirecting.

### Real-World Cases

- **User Registration:** Preserve name/email after failed registration.
- **Checkout Forms:** Preserve shipping address after validation failure.
- **Profile Updates:** Preserve all form fields after failed update.
- **SPA/AJAX Apps:** Handle 422 responses and display errors client-side.

### References

- Laravel Validation: The `$request->validate` Method — https://laravel.com/docs/12.x/validation#the-request-validate-method
- Laravel Validation: Working With Validated Input — https://laravel.com/docs/12.x/validation#working-with-validated-input
- Laravel Helpers: `old()` — https://laravel.com/docs/12.x/helpers#method-old

---

## References

- Laravel Validation Documentation — https://laravel.com/docs/12.x/validation
- Laravel Validation: Available Rules — https://laravel.com/docs/12.x/validation#available-validation-rules
- Laravel Validation: Displaying Errors — https://laravel.com/docs/12.x/validation#displaying-the-validation-errors
- Laravel Validation: Named Error Bags — https://laravel.com/docs/12.x/validation#named-error-bags
- Laravel Validation: The `$request->validate` Method — https://laravel.com/docs/12.x/validation#the-request-validate-method
- Laravel Validation: Working With Validated Input — https://laravel.com/docs/12.x/validation#working-with-validated-input
- Laravel Blade: `@error` Directive — https://laravel.com/docs/12.x/blade#the-error-directive
- Laravel Helpers: `old()` — https://laravel.com/docs/12.x/helpers#method-old
- Laravel 10.x Validation Documentation — https://laravel.com/framework/docs/10.x/validation
- Laravel 6.x Validation Documentation (PDF) — https://github.com/DevStorm-Team/laravel-book/blob/1a1d7668d9f5c0007cb8f30e3dbebbe8fd3cb659/laravel-docs-6.x.pdf
- Laravel 8.x Validation Documentation (PDF) — https://github.com/DevStorm-Team/laravel-book/blob/master/laravel-docs-8.x.pdf