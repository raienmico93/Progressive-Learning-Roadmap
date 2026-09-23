# Advanced, Database, and Array Validation: A Comprehensive Cheat Sheet

## Topic Overview

### Definitions

**Core Definition:** Advanced validation in Laravel encompasses database-aware constraints, nested array targeting, complex file/image rules, conditional rule application, custom validation logic, and API-specific error handling—extending beyond simple string-based rules into a fully programmatic validation layer.

**Technical Definition:** The `Illuminate\Validation\Rule` class provides static factory methods for creating fluent, database-aware rule instances such as `Rule::unique()`, `Rule::exists()`, and `Rule::when()`. Array validation uses wildcard dot notation (`products.*.id`) to target nested elements. File validation leverages the `Illuminate\Validation\Rules\File` class with `types()`, `image()`, `dimensions()`, and size constraints. The `Password` rule class enforces complexity requirements including `symbols()` and `uncompromised()`. Custom validation is achieved via closure rules, dedicated `ValidationRule` implementations, or `Validator::extend()` macros. In API contexts, validation failures throw `ValidationException` which is automatically converted to a structured `422 Unprocessable Content` JSON response.

**Beginner-Friendly Explanation:** Basic validation handles simple things like "this field is required." Advanced validation handles complex scenarios: checking if an email is unique in the database (while ignoring the current user during updates), validating every item in a shopping cart array, ensuring uploaded images meet dimension requirements, making rules conditional on user permissions, writing your own custom validation logic, and returning clean JSON errors for APIs.

### Key Characteristics

- **Fluent Rule Objects:** `Rule::unique()` and `Rule::exists()` provide a programmatic, chainable alternative to string rules.
- **Wildcard Array Targeting:** The `*` character validates every element in an array without knowing its size.
- **Rich File Validation:** The `File` rule class supports MIME types, image dimensions, and human-readable size units.
- **Password Complexity:** The `Password` rule enforces symbols, numbers, and breached-password checks.
- **Conditional Rules:** `Rule::when()` applies rules based on runtime state.
- **Multiple Extension Points:** Closures, rule objects, and `Validator::extend()` offer flexibility.
- **API-Aware:** `ValidationException` produces 422 JSON with structured error keys.

### Prerequisites

- A Laravel application with controllers, routes, and validation experience.
- Understanding of basic validation rules (`required`, `string`, `max`).
- Familiarity with Form Requests (recommended for complex validation).
- A configured database connection for `unique`/`exists` rules.
- For file validation: `php.ini` upload settings configured.

### Related Programming Areas

- **Eloquent ORM:** `unique` and `exists` rules query the database via the query builder.
- **Form Requests:** Advanced rules are commonly placed in Form Request classes.
- **API Development:** Structured 422 responses are essential for API consumers.
- **File Storage:** File validation integrates with Laravel's filesystem abstraction.

### Core Concepts / Features

1. Fluent Database Rules (`Rule::unique()`, `Rule::exists()`)
2. Comprehensive Array Validation (`*` Wildcard Dot Notation)
3. Advanced Constraints (File, Image, Password)
4. Conditional Rule Application (`Rule::when()`)
5. Extensibility (Closures, Rule Objects, `Validator::extend()`)
6. API-Specific Behaviour (Structured 422 JSON Responses)

---

## 1. Fluent Database Rules (`Rule::unique()`, `Rule::exists()`)

### Definitions

**Core Definition:** Fluent database rules are programmatic rule objects created via the `Rule` class that validate a field's uniqueness or existence against a database table, with full support for column customisation, ignore IDs, and additional where clauses.

**Technical Definition:** `Rule::unique(string $table, string $column = 'NULL')` returns an `Illuminate\Validation\Rules\Unique` instance. The `ignore()` method accepts a model instance, primary key value, or `Model` to exclude from the uniqueness check. Additional `where()` clauses can be chained. `Rule::exists()` returns an `Exists` instance with similar capabilities. These objects implement the `ValidationRule` interface and are resolved during validation.

**Beginner-Friendly Explanation:** When updating a user's profile, you need to check that the new email isn't already taken—but you must ignore the current user's own email. `Rule::unique('users')->ignore($user)` handles this automatically. Similarly, `Rule::exists('categories', 'id')` checks that a submitted category ID actually exists in the database.

### Purposes

- To validate that a field value is unique in a database table.
- To ignore the current record during uniqueness checks on updates.
- To validate that a field value exists in a database table.
- To add additional where clauses to uniqueness/existence queries.
- To validate composite unique constraints using multiple columns.

### Syntax Rules and Structure

**Complete General Syntax — `Rule::unique()`:**

```php
use Illuminate\Validation\Rule;

'email' => [
    'required',
    'email',
    Rule::unique('users', 'email')->ignore($user->id),
],
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Rule::unique('users', 'email')` | Checks uniqueness in `users.email`. |
| `->ignore($user->id)` | Excludes the current user's ID from the check. |
| `->where('account_id', 1)` | Adds additional constraints. |

**Complete General Syntax — `Rule::exists()`:**

```php
'category_id' => [
    'required',
    Rule::exists('categories', 'id')->where('active', true),
],
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Rule::exists('categories', 'id')` | Checks that the value exists in `categories.id`. |
| `->where('active', true)` | Only considers active categories. |

**Syntax Rules:**
- `Rule::unique()` accepts the table name and optional column name.
- `ignore()` accepts a model instance, ID value, or `Model` object.
- `where()` accepts a column/value pair or a closure for complex constraints.
- These rule objects can be mixed with string rules in an array.

**Constraints and Limitations:**
- **Ignore with soft deletes:** By default, soft-deleted records are included in uniqueness checks. Use `->whereNull('deleted_at')` to exclude them.
- **Composite unique constraints:** `Rule::unique()` only checks single-column uniqueness; multi-column unique indexes require custom logic or packages like `reinbier/laravel-unique-with` .
- **Performance:** Database rules execute queries during validation; index the relevant columns.

### Multiple Annotated Complete Code Examples

**Example 1: Unique Email on Update**

```php
<?php
use Illuminate\Validation\Rule;

public function rules(): array
{
    return [
        'email' => [
            'required',
            'email',
            'max:255',
            Rule::unique('users', 'email')->ignore($this->user()->id),
        ],
    ];
}
```

**Expected Output:** On update, the email is checked for uniqueness, but the current user's own email is excluded. If another user has the email, validation fails.

**Why:** `->ignore($this->user()->id)` adds a `WHERE id != ?` clause to the uniqueness query .

---

**Example 2: Exists with Additional Constraints**

```php
<?php
use Illuminate\Validation\Rule;

public function rules(): array
{
    return [
        'category_id' => [
            'required',
            Rule::exists('categories', 'id')->where(function ($query) {
                $query->where('active', true)
                      ->where('type', 'product');
            }),
        ],
    ];
}
```

**Expected Output:** The category must exist, be active, and be of type `product`.

**Why:** The `where()` closure adds constraints to the existence query .

---

**Example 3: Combining Unique with Additional Where Clauses**

```php
<?php
use Illuminate\Validation\Rule;

'email' => [
    Rule::unique('users', 'email')
        ->ignore($user->id)
        ->where('account_id', $accountId),
],
```

**Expected Output:** The email must be unique within the specified account, ignoring the current user.

**Why:** The `where()` method scopes the uniqueness check to a specific account .

### Real-World Cases

- **User Profile Updates:** `Rule::unique('users')->ignore($user)` prevents "email already taken" errors for the current user.
- **Category Selection:** `Rule::exists('categories', 'id')` ensures submitted category IDs are valid.
- **Multi-Tenant Applications:** `Rule::unique('users')->where('tenant_id', $tenantId)` scopes uniqueness per tenant.

### References

- Laravel Validation: Rule::unique — https://api.laravel.com/docs/12.x/Illuminate/Validation/Rule.html#method_unique 
- Laravel Validation: Rule::exists — https://api.laravel.com/docs/12.x/Illuminate/Validation/Rule.html#method_exists 
- Forcing Unique Rule to Ignore an ID — https://laravel-doctrine-orm-official.readthedocs.io/en/3.0.0/validation.html 

---

## 2. Comprehensive Array Validation (`*` Wildcard Dot Notation)

### Definitions

**Core Definition:** Array validation with wildcard dot notation allows validating every element of a nested array without knowing the array's size, using the `*` character as a placeholder for each index.

**Technical Definition:** The `Validator` class expands wildcard keys (e.g., `products.*.id`) into individual validation rules for each array element at runtime. The `*` can be nested (`photos.*.attributes.*.string`) and combined with custom messages using position placeholders (`:position`, `:second-position`). The `Rule::forEach()` method provides programmatic per-element rule generation.

**Beginner-Friendly Explanation:** If you have a form that submits an array of products (`products[0][id]`, `products[1][id]`, etc.), you can write a single rule `'products.*.id' => 'required|exists:products,id'` to validate every product's ID. The `*` says "check all of them."

### Purposes

- To validate every element of an array without knowing its size.
- To validate deeply nested array structures (arrays within arrays).
- To provide per-position custom error messages.
- To apply rules conditionally to array elements using `Rule::forEach()`.

### Syntax Rules and Structure

**Complete General Syntax:**

```php
$request->validate([
    'products' => 'required|array',
    'products.*.id' => 'required|exists:products,id',
    'products.*.quantity' => 'required|integer|min:1',
    'products.*.price' => 'required|numeric|min:0',
]);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `'products' => 'required|array'` | Ensures the field is an array. |
| `'products.*.id'` | Validates every element's `id` key. |
| `'products.*.quantity'` | Validates every element's `quantity` key. |

**Complete General Syntax — Nested Wildcards:**

```php
'photos.*.attributes.*.string' => 'required|string',
```

**Complete General Syntax — Custom Position Messages:**

```php
$validator = Validator::make($input, [
    'photos.*.description' => 'required',
], [
    'photos.*.description.required' => 'Please describe photo #:position.',
]);
```

**Component Breakdown:**

| Placeholder | Description |
|-------------|-------------|
| `:position` | The current position in the array (1-based). |
| `:second-position` | The second-level position. |

**Syntax Rules:**
- Always pair wildcard rules with a `'field' => 'array'` rule to ensure the structure.
- The `*` represents a numeric index or associative key.
- Custom messages can use `:position` placeholders for user-friendly indexing .
- `Rule::forEach()` provides programmatic rule arrays per element.

**Constraints and Limitations:**
- **No wildcard for associative keys:** The `*` matches any key, but you cannot target only specific keys within a wildcard.
- **Performance:** Large arrays with many elements generate many individual rules.
- **Missing parent array:** If the parent array is absent, wildcard rules are skipped (unless `required|array` is present).

### Multiple Annotated Complete Code Examples

**Example 1: Basic Product Array Validation**

```php
<?php
$validated = $request->validate([
    'products' => 'required|array|min:1',
    'products.*.id' => 'required|integer|exists:products,id',
    'products.*.quantity' => 'required|integer|min:1|max:100',
    'products.*.notes' => 'nullable|string|max:500',
]);
```

The Database & Model
```php
// database/migrations/xxxx_xx_xx_create_products_table.php
Schema::create('products', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->decimal('price', 8, 2);
    $table->timestamps();
});
```

The Controller Route
```php
// routes/api.php
use App\Http\Controllers\OrderController;

Route::post('/orders', [OrderController::class, 'store']);
```

The Controller Method
```php
// app/Http/Controllers/OrderController.php
namespace App\Http\Controllers;

use Illuminate\Http\Request;

class OrderController extends Controller
{
    public function store(Request $request)
    {
        // 1. Validate the incoming nested array data
        $validated = $request->validate([
            'products'            => 'required|array|min:1',
            'products.*.id'       => 'required|integer|exists:products,id',
            'products.*.quantity' => 'required|integer|min:1|max:100',
            'products.*.notes'    => 'nullable|string|max:500',
        ]);

        // 2. If validation passes, process the validated data example: Loop through the validated products
        foreach ($validated['products'] as $productData) {
            // Process $productData['id'], $productData['quantity'], etc.
        }

        return response()->json([
            'message' => 'Order processed successfully!',
            'data' => $validated
        ], 201);
    }
}
```

**Expected Output:** Each product in the array must have a valid `id`, a `quantity` between 1 and 100, and optional notes. If `products` is empty, validation fails.
```
{
    "message": 'Order processed successfully!',
    "data": {
        "products": [
            {
                "id": 4,
                "quantity": 2,
                "notes": "Please wrap it as a gift."
            },
            {
                "id": 12,
                "quantity": 5
            }
        ]
    }
}
```

**Why:** The `products.*.id` rule generates validation for `products.0.id`, `products.1.id`, etc. .

---

**Example 2: Nested Array Validation with Position Messages**

```php
<?php
use Illuminate\Support\Facades\Validator;

$validator = Validator::make($request->all(), [
    'photos.*.description' => 'required|string',
    'photos.*.attributes.*.value' => 'required',
], [
    'photos.*.description.required' => 'Please describe photo #:position.',
    'photos.*.attributes.*.value.required' => 'Missing value for attribute #:second-position on photo #:position.',
]);
```

**Expected Output:** Missing descriptions produce: "Please describe photo #2." Missing attribute values produce: "Missing value for attribute #1 on photo #3."

**Why:** The `:position` placeholder is replaced with the 1-based index of the failing element .

---

**Example 3: `Rule::forEach()` for Programmatic Rules**

```php
<?php
use Illuminate\Validation\Rule;

$request->validate([
    'companies.*.id' => Rule::forEach(function ($value, $attribute) {
        return [
            'required',
            'integer',
            Rule::exists('companies', 'id')->where('user_id', auth()->id()),
        ];
    }),
]);
```

**Expected Output:** Each company ID must exist and belong to the authenticated user.

**Why:** `Rule::forEach()` receives the element value and its fully-expanded attribute name, returning a rule array per element .

### Real-World Cases

- **Shopping Cart:** `products.*.id` and `products.*.quantity` validate every cart item.
- **Multi-Image Upload:** `photos.*.description` validates metadata for each uploaded image.
- **Bulk API Operations:** `items.*.id` validates arrays in batch API requests.
- **Dynamic Forms:** `contacts.*.email` validates repeated email fields.

### References

- Laravel Validation: Validating Arrays — https://laravel.com/index.php/docs/11.x/validation 
- Laravel Validation: Nested Array Validation (GitHub) — https://raw.githubusercontent.com/laravel/docs/4b639895e4a56fc15f282334824dbd1b3f350c16/validation.md 

---

## 3. Advanced Constraints (File, Image, Password)

### Definitions

**Core Definition:** Advanced constraints are specialised rule classes providing rich validation for files (MIME types, dimensions, sizes) and passwords (symbols, numbers, breached-password checks).

**Technical Definition:** The `Illuminate\Validation\Rules\File` class provides `types()`, `image()`, `min()`, `max()`, and `dimensions()` methods. The `Illuminate\Validation\Rules\Password` class provides `min()`, `mixedCase()`, `numbers()`, `symbols()`, and `uncompromised()` methods. `Rule::dimensions()` creates a `Dimensions` instance for width/height constraints.

**Beginner-Friendly Explanation:** Instead of writing multiple string rules for file validation (`mimes:jpg,png|max:2048|dimensions:max_width=1000`), you can use a fluent `File::image()->max('2mb')->dimensions(...)` builder. For passwords, `Password::min(8)->symbols()->uncompromised()` enforces strong password policies.

### Purposes

- To validate file types by extension or MIME type.
- To validate image dimensions (width, height, aspect ratio).
- To enforce file size limits with human-readable units.
- To enforce password complexity (length, case, numbers, symbols).
- To check passwords against known data breaches using `uncompromised()`.

### Syntax Rules and Structure

**Complete General Syntax — File Validation:**

```php
use Illuminate\Validation\Rules\File;
use Illuminate\Validation\Rule;

$request->validate([
    'attachment' => [
        'required',
        File::types(['mp3', 'wav'])
            ->min(1024)
            ->max(12 * 1024),
    ],
    'photo' => [
        'required',
        File::image()
            ->min('1kb')
            ->max('10mb')
            ->dimensions(Rule::dimensions()->maxWidth(1000)->maxHeight(500)),
    ],
]);
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `File::types([...])` | Validates file extensions/MIME types. |
| `File::image()` | Shorthand for common image types. |
| `->min()` / `->max()` | File size in KB or human-readable units (`kb`, `mb`, `gb`). |
| `->dimensions()` | Image dimension constraints. |
| `Rule::dimensions()` | Width/height/aspect ratio constraints. |

**Complete General Syntax — Password Validation:**

```php
use Illuminate\Validation\Rules\Password;

$request->validate([
    'password' => [
        'required',
        'confirmed',
        Password::min(8)
            ->mixedCase()
            ->numbers()
            ->symbols()
            ->uncompromised(),
    ],
]);
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `Password::min(8)` | Minimum length. |
| `->mixedCase()` | Requires upper and lowercase. |
| `->numbers()` | Requires at least one number. |
| `->symbols()` | Requires at least one symbol. |
| `->uncompromised()` | Checks against data breaches (HaveIBeenPwned). |

**Syntax Rules:**
- File sizes can be specified as integers (KB) or strings with units (`'1kb'`, `'10mb'`) .
- `File::types()` validates MIME type by reading file contents, not just extension .
- Password rules can be combined in any order.
- `uncompromised()` makes an external API call; use cautiously in high-traffic scenarios.

**Constraints and Limitations:**
- **`uncompromised()` requires internet access:** The HaveIBeenPwned API is queried.
- **`File::image()` limits types:** It accepts common image formats (jpg, jpeg, png, bmp, gif, svg, webp).
- **Dimension validation requires image processing:** The GD or Imagick extension must be available.

### Multiple Annotated Complete Code Examples

**Example 1: File Validation with Types and Size**

```php
<?php
use Illuminate\Validation\Rules\File;

$request->validate([
    'document' => [
        'required',
        File::types(['pdf', 'docx'])
            ->max('5mb'),
    ],
]);
```

**Expected Output:** Only PDF and DOCX files under 5 MB are accepted.

**Why:** `File::types()` validates both extension and MIME type. The `max('5mb')` limits file size .

---

**Example 2: Image Validation with Dimensions**

```php
<?php
use Illuminate\Validation\Rule;
use Illuminate\Validation\Rules\File;

$request->validate([
    'avatar' => [
        'required',
        File::image()
            ->min('1kb')
            ->max('2mb')
            ->dimensions(
                Rule::dimensions()
                    ->minWidth(100)
                    ->maxWidth(1000)
                    ->minHeight(100)
                    ->maxHeight(1000)
            ),
    ],
]);
```

**Expected Output:** Images must be between 100x100 and 1000x1000 pixels, between 1 KB and 2 MB.

**Why:** `Rule::dimensions()` provides fluent width/height constraints .

---

**Example 3: Strong Password Validation**

```php
<?php
use Illuminate\Validation\Rules\Password;

$request->validate([
    'password' => [
        'required',
        'confirmed',
        Password::min(8)
            ->mixedCase()
            ->numbers()
            ->symbols()
            ->uncompromised(3), // Allow up to 3 breach occurrences
    ],
]);
```

**Expected Output:** Passwords must be 8+ characters, contain mixed case, numbers, symbols, and not appear in known breaches (with a threshold of 3).

**Why:** The `Password` rule enforces complexity. `uncompromised(3)` allows passwords that appear up to 3 times in breach data .

### Real-World Cases

- **Profile Avatars:** `File::image()->dimensions(Rule::dimensions()->maxWidth(500))` for user avatars.
- **Document Uploads:** `File::types(['pdf'])->max('10mb')` for invoice uploads.
- **User Registration:** `Password::min(12)->symbols()->uncompromised()` for high-security applications.
- **Product Images:** `File::image()->dimensions(Rule::dimensions()->ratio(1/1))` for square product thumbnails.

### References

- Laravel Validation: File Rules (Laravel 10.x) — https://laravel.com/index.php/framework/docs/10.x/validation 
- Laravel Validation: File Types (GitHub) — https://raw.githubusercontent.com/laravel/docs/d01cfeb1d6e83af7e322eb851e87ae26094b9f4b/validation.md 
- Laravel API: Password Rule — https://api.laravel.com/docs/11.x//Illuminate/Validation/Rules/Password.html 

---

## 4. Conditional Rule Application (`Rule::when()`)

### Definitions

**Core Definition:** Conditional rule application allows adding validation rules dynamically based on runtime state, using `Rule::when()` or complex closures.

**Technical Definition:** `Rule::when($condition, $rules, $defaultRules = [])` returns either the provided rules or the default rules based on the condition. This enables context-aware validation, such as requiring a field only if the user has permission, or applying different rules based on the HTTP method.

**Beginner-Friendly Explanation:** Sometimes a field should only be validated under certain conditions. For example, a "vote" field should only be required if the user is allowed to vote on the post. `Rule::when($user->can('vote', $post), 'required|int|between:1,5')` applies the rules only when the condition is true.

### Purposes

- To apply validation rules conditionally based on runtime state.
- To simplify complex `required_if` / `required_unless` chains.
- To support context-aware validation (create vs. update, user permissions).
- To allow closures to determine rules dynamically.

### Syntax Rules and Structure

**Complete General Syntax:**

```php
use Illuminate\Validation\Rule;

$request->validate([
    'vote' => Rule::when(
        $user->can('vote', $post),
        'required|integer|between:1,5',
        'prohibited' // Applied when condition is false
    ),
]);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Rule::when($condition, $rules)` | Applies `$rules` when `$condition` is truthy. |
| `$defaultRules` | Optional rules when condition is falsy. |

**Complete General Syntax — With Closures:**

```php
'name' => Rule::when(
    $this->method() === 'POST',
    ['required', 'string', 'max:255'],
    ['sometimes', 'string', 'max:255']
),
```

**Syntax Rules:**
- The condition can be any boolean expression.
- Rules can be a string, array, or rule objects.
- The default rules (third argument) are optional.
- Multiple `Rule::when()` calls can be chained for complex logic.

**Constraints and Limitations:**
- **No nested conditions:** For very complex conditional logic, use a Form Request with a `rules()` method containing PHP conditionals.
- **Readability:** Heavy use of `Rule::when()` can make rules harder to read than explicit conditionals.

### Multiple Annotated Complete Code Examples

**Example 1: Permission-Based Rule**

```php
<?php
use Illuminate\Validation\Rule;

public function rules(): array
{
    return [
        'vote' => Rule::when(
            $this->user()->can('vote', $this->route('post')),
            'required|integer|between:1,5',
        ),
    ];
}
```

**Expected Output:** The vote is validated only if the user has permission to vote on the post.

**Why:** The condition checks the user's permission. If false, no rules are applied .

---

**Example 2: Create vs. Update Rules**

```php
<?php
use Illuminate\Validation\Rule;

public function rules(): array
{
    $isCreate = $this->method() === 'POST';

    return [
        'email' => Rule::when(
            $isCreate,
            'required|email|unique:users,email',
            'sometimes|email|unique:users,email,' . $this->user()->id
        ),
    ];
}
```

**Expected Output:** On create, email is required and unique. On update, email is validated only if present, ignoring the current user.

**Why:** The condition determines which set of rules applies based on the HTTP method .

### Real-World Cases

- **Role-Based Validation:** Require an `admin_code` only if the user is creating an admin account.
- **Feature Flags:** Apply stricter validation when a feature is enabled.
- **API Versioning:** Apply different rules for v1 vs. v2 endpoints.
- **Conditional Required Fields:** Require `shipping_address` only if `delivery_method` is `shipping`.

### References

- Laravel Daily: Rule::when() — https://laraveldaily.com/tip/with-rulewhen-we-can-conditionally-apply-validation-rules 
- Laravel Rules Package: Conditional Rules — https://packagist.org/packages/bradietilley/laravel-rules 

---

## 5. Extensibility (Closures, Rule Objects, `Validator::extend()`)

### Definitions

**Core Definition:** Validation extensibility refers to the three mechanisms for creating custom validation logic: closure-based rules, dedicated `ValidationRule` objects, and `Validator::extend()` macros.

**Technical Definition:** Closure rules are passed directly in the rules array as `function ($attribute, $value, $fail) {}`. Rule objects implement `Illuminate\Contracts\Validation\ValidationRule` with a `validate()` method. `Validator::extend()` registers a named rule in a service provider. Each approach offers different trade-offs between simplicity, reusability, and testability.

**Beginner-Friendly Explanation:** When built-in rules don't cover your needs, you can write your own. A closure is quick and inline. A rule object is a dedicated class that's easy to test and reuse. `Validator::extend()` creates a string-callable rule like `'my_rule'`.

### Purposes

- To validate domain-specific business rules not covered by built-in rules.
- To reuse validation logic across multiple forms.
- To test validation logic in isolation.
- To integrate with external APIs or complex algorithms.

### Syntax Rules and Structure

**Complete General Syntax — Closure Rule:**

```php
$request->validate([
    'word' => [
        'required',
        function (string $attribute, mixed $value, Closure $fail) {
            if (strtoupper($value) !== $value) {
                $fail("The {$attribute} must be uppercase.");
            }
        },
    ],
]);
```

**Complete General Syntax — Rule Object:**

```php
namespace App\Rules;

use Closure;
use Illuminate\Contracts\Validation\ValidationRule;

class Uppercase implements ValidationRule
{
    public function validate(string $attribute, mixed $value, Closure $fail): void
    {
        if (strtoupper($value) !== $value) {
            $fail("The {$attribute} must be uppercase.");
        }
    }
}

// Usage
'word' => [new Uppercase()],
```

**Complete General Syntax — `Validator::extend()`:**

```php
// In a service provider
Validator::extend('uppercase', function ($attribute, $value, $parameters, $validator) {
    return strtoupper($value) === $value;
});

// Usage
'word' => 'required|uppercase',
```

**Component Breakdown:**

| Approach | Description |
|----------|-------------|
| Closure | Inline, quick, not reusable. |
| Rule Object | Dedicated class, testable, reusable. |
| `Validator::extend()` | Named string rule, registered globally. |

**Syntax Rules:**
- Closure rules receive `$attribute`, `$value`, and `$fail`.
- Rule objects implement `ValidationRule` and may use `DataAwareRule` or `ValidatorAwareRule`.
- `Validator::extend()` registers the rule globally.
- Rule objects can accept constructor arguments for parameterised rules .

**Constraints and Limitations:**
- **Closure rules:** Not reusable across forms; harder to test independently.
- **Rule objects:** Require a class file; constructor dependencies are not injected.
- **`Validator::extend()`:** Registered globally; can conflict with package rules.

### Multiple Annotated Complete Code Examples

**Example 1: Closure Rule**

```php
<?php
$request->validate([
    'palindrome' => [
        'required',
        'string',
        function (string $attribute, mixed $value, Closure $fail) {
            if ($value !== strrev($value)) {
                $fail("The {$attribute} must be a palindrome.");
            }
        },
    ],
]);
```

**Expected Output:** "racecar" passes; "laravel" fails with "The palindrome must be a palindrome."

**Why:** The closure checks if the value reads the same forwards and backwards .

---

**Example 2: Rule Object with Parameters**

```php
<?php
namespace App\Rules;

use Closure;
use Illuminate\Contracts\Validation\ValidationRule;

class BetweenDates implements ValidationRule
{
    public function __construct(
        private string $start,
        private string $end,
    ) {}

    public function validate(string $attribute, mixed $value, Closure $fail): void
    {
        if ($value < $this->start || $value > $this->end) {
            $fail("The {$attribute} must be between {$this->start} and {$this->end}.");
        }
    }
}

// Usage
'event_date' => [new BetweenDates('2024-01-01', '2024-12-31')],
```

**Expected Output:** Dates outside the range fail with a descriptive message.

**Why:** Constructor arguments parameterise the rule for reuse .

---

**Example 3: `Validator::extend()` Macro**

```php
<?php
// AppServiceProvider::boot()
use Illuminate\Support\Facades\Validator;

Validator::extend('uppercase', function ($attribute, $value, $parameters, $validator) {
    return strtoupper($value) === $value;
});

// Usage
$request->validate([
    'code' => 'required|uppercase',
]);
```

**Expected Output:** "ABC" passes; "abc" fails.

**Why:** `Validator::extend()` registers a named rule accessible via string syntax.

### Real-World Cases

- **Domain Rules:** Validate that a booking date is within business hours.
- **External APIs:** Validate that a VAT number exists via an external service.
- **Complex Formats:** Validate IBAN, SWIFT codes, or custom serial numbers.
- **Business Logic:** Validate that an order total matches the sum of line items.

### References

- Laravel Validation: Using Closures — https://laravel-news.com/laravel-validation 
- Laravel Validation: Using Rule Objects — https://stackoverflow.com/revisions/e6aabeb8-3819-484c-bf4f-7a21385f512a/view-source 
- Laravel Validation: Custom Rule Objects — https://stackoverflow.com/revisions/f45ef3ff-3087-48ee-98e1-35f83912af78/view-source 

---

## 6. API-Specific Behaviour (Structured 422 JSON Responses)

### Definitions

**Core Definition:** In API contexts, Laravel automatically converts validation failures into a structured JSON response with HTTP status `422 Unprocessable Content`, containing a `message` and an `errors` object keyed by field name.

**Technical Definition:** The `Illuminate\Validation\ValidationException` is thrown when validation fails. Laravel's exception handler checks if the request expects JSON (via `Accept: application/json` or `X-Requested-With: XMLHttpRequest`) and returns a `JsonResponse` with the structure `{"message": "...", "errors": {"field": ["error1", "error2"]}}`. The `invalidJson()` method in the exception handler can customise the response.

**Beginner-Friendly Explanation:** When your API receives invalid data, Laravel doesn't redirect—it returns JSON with error details. The response has a `422` status code and lists which fields failed and why, so your frontend can display errors next to the right inputs.

### Purposes

- To provide machine-readable validation errors for API consumers.
- To maintain a consistent error format across all API endpoints.
- To allow frontend applications to map errors to specific form fields.
- To customise error responses for API-specific requirements.

### Syntax Rules and Structure

**Complete General Syntax — Default API Response:**

```json
{
    "message": "The email field is required. (and 2 more errors)",
    "errors": {
        "email": [
            "The email field is required."
        ],
        "password": [
            "The password field is required."
        ]
    }
}
```

**Complete General Syntax — Customising via Exception Handler:**

```php
// app/Exceptions/Handler.php or bootstrap/app.php
use Illuminate\Validation\ValidationException;

protected function invalidJson($request, ValidationException $exception)
{
    return response()->json([
        'message' => $exception->getMessage(),
        'errors' => $exception->errors(),
        'status' => 422,
    ], 422);
}
```

**Component Breakdown:**

| Key | Description |
|-----|-------------|
| `message` | Summary message (first error + count). |
| `errors` | Object keyed by field name, arrays of messages. |
| `status` | HTTP status code (422). |

**Syntax Rules:**
- Requests must include `Accept: application/json` to receive JSON.
- The `errors` object uses field names as keys and arrays of messages as values.
- Array field errors use dot notation: `"products.0.id"`.
- The response status is always `422`.

**Constraints and Limitations:**
- **Non-JSON requests redirect:** Without the `Accept` header, Laravel redirects even on API routes.
- **Customisation location:** The `invalidJson()` method must be in the exception handler.
- **Message truncation:** The default `message` truncates after the first error plus a count.

### Multiple Annotated Complete Code Examples

**Example 1: Default 422 Response**

```php
<?php
// API route
Route::post('/users', function (Request $request) {
    $request->validate([
        'name' => 'required|string|max:255',
        'email' => 'required|email|unique:users',
        'password' => 'required|min:8|confirmed',
    ]);

    return User::create($request->validated());
});
```

```json
// Request: POST /api/users (Accept: application/json)
// Body: {"name": "", "email": "invalid", "password": "short"}
{
    "message": "The name field is required. (and 2 more errors)",
    "errors": {
        "name": ["The name field is required."],
        "email": ["The email field must be a valid email address."],
        "password": ["The password field must be at least 8 characters."]
    }
}
```

**Expected Output:** HTTP 422 with the JSON structure above.

**Why:** Laravel detects the JSON request and returns structured errors instead of redirecting.

---

**Example 2: Custom API Error Response**

```php
<?php
// app/Exceptions/Handler.php
use Illuminate\Validation\ValidationException;

protected function invalidJson($request, ValidationException $exception)
{
    return response()->json([
        'success' => false,
        'message' => 'Validation failed',
        'errors' => $exception->errors(),
        'error_count' => count($exception->errors()),
        'timestamp' => now()->toIso8601String(),
    ], 422);
}
```

**Expected Output:** A customised 422 response with additional metadata.

**Why:** Overriding `invalidJson()` allows full control over the API error format.

---

**Example 3: Array Field Errors**

```php
<?php
$request->validate([
    'products.*.id' => 'required|exists:products,id',
    'products.*.quantity' => 'required|integer|min:1',
]);
```

```json
{
    "message": "The products.0.id field is required. (and 1 more error)",
    "errors": {
        "products.0.id": ["The products.0.id field is required."],
        "products.1.quantity": ["The products.1.quantity field must be at least 1."]
    }
}
```

**Expected Output:** Array field errors are keyed with full dot notation.

**Why:** Laravel uses the fully-expanded attribute name for array validation errors.

### Real-World Cases

- **SPA Frontends:** Vue/React apps map `errors` keys to form fields for inline error display.
- **Mobile Apps:** Native apps parse the JSON structure to show field-specific errors.
- **API Documentation:** 422 responses document all possible validation errors.
- **Third-Party Integrations:** API consumers rely on structured errors for automated handling.

### References

- Laravel Validation Error Response (API) — https://packagist.org/packages/a-h-abid/laravalerors 
- Validation Codes for Laravel — https://packagist.org/packages/kdabrow/validation-codes 
- Laravel Validation: API Error Handling — https://laravel.com/index.php/docs/11.x/validation 

---

## References

- Laravel Validation: Rule API — https://api.laravel.com/docs/12.x/Illuminate/Validation/Rule.html 
- Laravel Nova Validation — https://nova.laravel.com/docs/v5/resources/validation 
- Laravel 11.x Validation Documentation — https://laravel.com/index.php/docs/11.x/validation 
- Laravel 10.x Validation: File Rules — https://laravel.com/index.php/framework/docs/10.x/validation 
- Laravel API: Password Rule — https://api.laravel.com/docs/11.x//Illuminate/Validation/Rules/Password.html 
- Laravel Rules Package: Conditional Rules — https://packagist.org/packages/bradietilley/laravel-rules 
- Custom Validation Rule with Arguments (Stack Overflow) — https://stackoverflow.com/revisions/e6aabeb8-3819-484c-bf4f-7a21385f512a/view-source 
- Validation Codes for Laravel — https://packagist.org/packages/kdabrow/validation-codes 
- Unique With Validator — https://packagist.org/packages/reinbier/laravel-unique-with 
- Laravel Doctrine ORM: Unique and Exists — https://laravel-doctrine-orm-official.readthedocs.io/en/3.0.0/validation.html 
- Laravel Prompts: Additional Validation — https://laravel.com/framework/docs/prompts 
- Laravel Validation: Nested Array Validation (GitHub) — https://raw.githubusercontent.com/laravel/docs/4b639895e4a56fc15f282334824dbd1b3f350c16/validation.md 
- Laravel Validation: File Types (GitHub) — https://raw.githubusercontent.com/laravel/docs/d01cfeb1d6e83af7e322eb851e87ae26094b9f4b/validation.md 
- Laravel Validation: A Practical Guide (Laravel News) — https://laravel-news.com/laravel-validation 
- Laravel Daily: Rule::when() — https://laraveldaily.com/tip/with-rulewhen-we-can-conditionally-apply-validation-rules 
- Custom Validation Rule with Arguments (Stack Overflow Revision) — https://stackoverflow.com/revisions/f45ef3ff-3087-48ee-98e1-35f83912af78/view-source 
- Laravel API Validator Error Response — https://packagist.org/packages/a-h-abid/laravalerors 
- Update a Customer's Code (Laracasts) — https://laracasts.com/index.php/discuss/channels/laravel/update-a-customers-code-which-was-initially-given-wrong 
- Exclude Value from Validation (Laracasts) — https://laracasts.com/discuss/channels/laravel/exclude-value-from-validation-if-it-already-exists-in-the-db-is-not-unique 
- Livewire Validation — https://livewire.laravel.com/docs/4.x/validation 