# Laravel API Validation: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Laravel API Validation is the framework's system for validating incoming HTTP request data in API contexts, automatically returning structured JSON error responses with a 422 status code when validation fails, and providing tools for customizing error formatting, status codes, and response structure.

**Technical Definition**

Laravel's API validation leverages the same `Illuminate\Validation\Validator` engine used for web validation, but detects API requests via the `Accept: application/json` header or the `expectsJson()` method. When validation fails, a `ValidationException` is thrown, which the exception handler renders as a JSON response with a 422 status code containing a `message` key and a field-keyed `errors` object. Form Request classes provide the primary mechanism for encapsulating API validation rules and authorization, with extension points (`failedValidation()`, `response()`, `withValidator()`) for customizing the error response format. Laravel also supports OpenAPI-spec-based validation, machine-readable error codes, and RFC 9457 Problem Details for standardized API error responses.

**Beginner-Friendly Explanation**

When someone sends data to your API—like creating a new user or placing an order—you need to make sure the data is correct before saving it. Laravel's API validation does this automatically. If something is wrong (like a missing email or a password that is too short), Laravel sends back a JSON response with a 422 status code and a clear list of what went wrong. The client (like a mobile app or frontend) can then show those errors to the user. Laravel handles all the technical details, so you just define your rules and let the framework do the rest.

---

### Key Characteristics

- **Automatic JSON Detection**: Laravel automatically returns JSON error responses for requests that send the `Accept: application/json` header or use AJAX.
- **422 Status Code**: Validation failures return a 422 Unprocessable Entity status code, the standard HTTP status for validation errors.
- **Structured Error Format**: Errors are returned as a field-keyed object under an `errors` key, with a top-level `message` key summarizing the failure.
- **Form Request Integration**: Form Request classes encapsulate validation and authorization, with extension points for customizing error responses.
- **Customizable Error Messages and Attributes**: The `messages()` and `attributes()` methods allow full control over error wording and field names.
- **Error Code Support**: Packages like `kdabrow/validation-codes` add machine-readable error codes to validation responses.
- **OpenAPI Validation**: Packages like `kirschbaum-development/laravel-openapi-validator` validate requests and responses against OpenAPI specifications.
- **RFC 9457 Problem Details**: Laravel 13+ supports Problem Details for HTTP APIs, a standardized error response format.
- **Consistent Error Handling**: Laravel's exception handler provides consistent error responses across all API endpoints.
- **Testability**: Laravel provides `assertJsonValidationErrors()` and `assertOnlyJsonValidationErrors()` for precise validation error testing.

---

### Prerequisites

- **PHP 8.1 or higher** (Laravel 10+) or **PHP 8.2** (Laravel 11+).
- **A Laravel application** installed via Composer.
- **Eloquent models** for data validation against database state.
- **A configured API route file** (`routes/api.php`), created via `php artisan install:api`.
- **Basic understanding of Laravel validation rules** and Form Requests.
- **Knowledge of HTTP status codes** and JSON structure.

---

### Related Programming Areas

- **Form Request Validation**: The primary mechanism for encapsulating API validation rules and authorization.
- **Exception Handling**: Laravel's exception handler renders `ValidationException` as JSON 422 responses for API requests.
- **API Resources**: Transform validated data into consistent JSON responses.
- **Middleware**: The `api` middleware group provides stateless request handling for APIs.
- **Rate Limiting**: Protects API validation endpoints from abuse.
- **API Documentation**: OpenAPI specifications can drive both validation and documentation.
- **Testing**: Laravel's HTTP testing API provides assertions for JSON validation errors.

---

## Core Concepts

---

### 1. API Validation

#### Definitions

**Core Definition**

API validation is the process of verifying that data submitted to an API endpoint meets the application's requirements before it is processed or persisted, with validation failures returned as structured JSON responses.

**Technical Definition**

API validation in Laravel uses the same `Validator` engine as web validation, but the response format differs. When a `ValidationException` is thrown during an API request (detected via the `Accept: application/json` header or `expectsJson()`), Laravel's exception handler renders a JSON response with a 422 status code. The response structure includes a `message` key (e.g., "The name field is required. (and 1 more error)") and an `errors` object keyed by field name. Form Request classes are the recommended approach for API validation, as they encapsulate rules, authorization, and response customization.

**Beginner-Friendly Explanation**

API validation is like a quality control checkpoint for data coming into your API. Before your application uses the data, Laravel checks it against the rules you define. If something is wrong, the API sends back a clear JSON message telling the client exactly what needs to be fixed.

#### Purposes

- To ensure that API request data meets the application's requirements before processing.
- To protect the application from invalid, malicious, or malformed data.
- To provide clear, actionable feedback to API clients when their input is incorrect.
- To enforce data integrity constraints at the API boundary.
- To centralize validation logic in a reusable, testable manner.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
<?php
// app/Http/Requests/StoreUserRequest.php
namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Validation\Rule;

class StoreUserRequest extends FormRequest
{
    public function authorize(): bool
    {
        return $this->user() !== null;
    }

    public function rules(): array
    {
        return [
            'name' => ['required', 'string', 'max:255'],
            'email' => ['required', 'email', Rule::unique('users')],
            'password' => ['required', 'string', 'min:8', 'confirmed'],
        ];
    }

    public function messages(): array
    {
        return [
            'email.required' => 'We need your email address.',
            'email.unique' => 'This email is already registered.',
        ];
    }

    public function attributes(): array
    {
        return [
            'email' => 'email address',
        ];
    }
}
```

```php
// Controller usage
public function store(StoreUserRequest $request)
{
    $user = User::create($request->validated());

    return new UserResource($user);
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `extends FormRequest` | Base class providing validation and authorization infrastructure. |
| `rules(): array` | Returns the validation rules. |
| `messages(): array` | Custom error messages keyed by `field.rule`. |
| `attributes(): array` | Custom attribute names for error messages. |
| `$request->validated()` | Returns only the validated fields. |

**Syntax Rules**

- Form Requests are resolved automatically when type-hinted in a controller method.
- The `Accept: application/json` header triggers JSON error responses.
- Validation failures return a 422 status code with a JSON error structure.
- The `validated()` method returns only the fields present in the `rules()` array.

**Constraints and Limitations**

- API validation requires the request to include the `Accept: application/json` header; otherwise, Laravel returns a redirect response.
- Form Requests cannot be used in Closure-based routes.
- Complex validation logic should be extracted into custom rule classes.

---

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Basic API Validation with Form Request**

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
        return $this->user() !== null;
    }

    public function rules(): array
    {
        return [
            'title' => ['required', 'string', 'max:255'],
            'body' => ['required', 'string', 'min:10'],
        ];
    }
}
```

**Controller**

```php
<?php
// app/Http/Controllers/Api/PostController.php
namespace App\Http\Controllers\Api;

use App\Http\Requests\StorePostRequest;
use App\Http\Resources\PostResource;
use App\Models\Post;

class PostController extends Controller
{
    public function store(StorePostRequest $request)
    {
        $post = Post::create($request->validated());

        return new PostResource($post);
    }
}
```

**Route**

```php
// routes/api.php
Route::middleware('auth:sanctum')->post('/posts', [PostController::class, 'store']);
```

**Expected Output (Validation Failure)**

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

The `StorePostRequest` validates the request data. When validation fails, Laravel detects the API request (via the `Accept: application/json` header) and returns a JSON response with a 422 status code. The `errors` object contains the field-keyed error messages.

---

**Example 2: Custom Error Messages and Attributes**

**Form Request**

```php
public function rules(): array
{
    return [
        'customer_email' => ['required', 'email', 'max:255'],
        'items' => ['required', 'array', 'min:1'],
        'items.*.product_id' => ['required', 'integer', 'exists:products,id'],
        'items.*.quantity' => ['required', 'integer', 'min:1', 'max:100'],
    ];
}

public function messages(): array
{
    return [
        'customer_email.required' => 'We need your email address to process the order.',
        'items.required' => 'Your order must contain at least one item.',
        'items.*.product_id.exists' => 'One of the selected products is no longer available.',
        'items.*.quantity.max' => 'You cannot order more than :max of a single item.',
    ];
}

public function attributes(): array
{
    return [
        'customer_email' => 'email address',
        'items.*.product_id' => 'product',
        'items.*.quantity' => 'quantity',
    ];
}
```

**Expected Output (Validation Failure)**

```json
{
    "message": "We need your email address to process the order. (and 2 more errors)",
    "errors": {
        "customer_email": ["We need your email address to process the order."],
        "items.0.product_id": ["One of the selected products is no longer available."],
        "items.1.quantity": ["You cannot order more than 100 of a single item."]
    }
}
```

**Why This Output Occurs**

The `messages()` method overrides default error messages, and `attributes()` replaces technical field names with human-readable ones. The wildcard notation (`items.*`) applies rules to each array element, and errors include the specific index.

---

#### Real-World Cases with Explanation

**Case 1: E-commerce Order API**

An e-commerce platform's order creation API validates `customer_email`, `items`, `items.*.product_id`, and `items.*.quantity`. The Form Request uses nested array validation with custom messages, ensuring every order item is valid before the order is created.

**Explanation**: API validation ensures data integrity across all clients (mobile, web, third-party) with consistent error responses.

**Case 2: User Registration API**

A mobile app's registration endpoint validates `name`, `email`, and `password`. The Form Request returns 422 JSON errors that the app parses to display field-level error messages next to each input.

**Explanation**: API validation provides clear, actionable feedback that frontend clients can use for precise error display.

---

### 2. Validation Errors

#### Definitions

**Core Definition**

Validation errors are the structured messages returned when API request data fails validation, containing field-specific error details that clients can parse and display.

**Technical Definition**

Laravel's `ValidationException` carries a `Validator` instance whose `errors()` method returns a `MessageBag` containing field-keyed error messages. When rendered as JSON, the response includes a `message` key (a summary) and an `errors` object where each key is a field name and each value is an array of error messages. The `MessageBag` provides methods like `first()`, `get()`, `all()`, `has()`, and `isEmpty()` for programmatic error inspection.

**Beginner-Friendly Explanation**

Validation errors are the messages that tell the client what went wrong. Laravel groups them by field name, so the client can easily show "Email is required" next to the email field and "Password is too short" next to the password field.

#### Purposes

- To provide clear, field-specific feedback to API clients.
- To enable frontend frameworks to display errors next to the corresponding input fields.
- To support programmatic error handling in API clients.
- To maintain a consistent error structure across all API endpoints.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Default JSON error response
{
    "message": "The given data was invalid.",
    "errors": {
        "field1": ["Error message 1", "Error message 2"],
        "field2": ["Error message 3"]
    }
}
```

```php
// Accessing errors programmatically
try {
    $validated = $request->validate([...]);
} catch (ValidationException $e) {
    $errors = $e->errors(); // Returns array of field => messages
    $firstError = $e->validator->errors()->first('email');
    $hasEmailError = $e->validator->errors()->has('email');
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `message` | Summary message (e.g., "The given data was invalid.") |
| `errors` | Field-keyed object with error message arrays |
| `$e->errors()` | Returns the errors array |
| `$e->validator->errors()->first('field')` | Returns the first error for a field |
| `$e->validator->errors()->has('field')` | Checks if a field has errors |

**Syntax Rules**

- The `errors` object is keyed by field name (including dot notation for nested fields).
- Each field maps to an array of error messages.
- The `message` key provides a human-readable summary.
- Custom error messages can be defined per field and rule.

**Constraints and Limitations**

- The default `message` is generic; customize it via `failedValidation()` or the exception handler.
- Field names in the `errors` object use dot notation for nested arrays (e.g., `items.0.product_id`).

---

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Default Validation Error Response**

**Request**

```bash
curl -X POST http://api.example.com/api/users \
  -H "Accept: application/json" \
  -H "Content-Type: application/json" \
  -d '{}'
```

**Expected Output**

```json
{
    "message": "The name field is required. (and 2 more errors)",
    "errors": {
        "name": ["The name field is required."],
        "email": ["The email field is required."],
        "password": ["The password field is required."]
    }
}
```

**Why This Output Occurs**

The empty request body fails all `required` rules. Laravel returns a 422 JSON response with the field-keyed errors and a summary message.

---

**Example 2: Nested Array Validation Errors**

**Request**

```json
{
    "customer_email": "john@example.com",
    "items": [
        {"product_id": 1, "quantity": 5},
        {"product_id": 999, "quantity": 0}
    ]
}
```

**Expected Output**

```json
{
    "message": "One of the selected products is no longer available. (and 1 more error)",
    "errors": {
        "items.1.product_id": ["One of the selected products is no longer available."],
        "items.1.quantity": ["The items.1.quantity must be at least 1."]
    }
}
```

**Why This Output Occurs**

The `exists` rule fails for `items.1.product_id` (999 does not exist), and the `min:1` rule fails for `items.1.quantity` (0 is below the minimum). The errors are keyed by the specific array index.

---

#### Real-World Cases with Explanation

**Case 1: Frontend Form Error Display**

A React frontend receives a 422 validation error response and parses the `errors` object to display messages next to the corresponding form fields.

**Explanation**: Field-keyed errors enable precise, user-friendly error display.

---

### 3. Error Responses

#### Definitions

**Core Definition**

Error responses are the HTTP responses returned when API validation fails, containing structured error information with appropriate status codes and JSON payloads.

**Technical Definition**

Laravel's `ValidationException` is rendered by the exception handler (`Illuminate\Foundation\Exceptions\Handler`) into an `Illuminate\Http\JsonResponse` with a 422 status code. The response includes a `message` key and an `errors` object. The `failedValidation()` method on Form Requests can override the default response, and the `response()` method can customize the response for specific validation scenarios.

**Beginner-Friendly Explanation**

An error response is the JSON message your API sends back when something is wrong with the data. It includes a status code (422 for validation errors) and a structured body that tells the client exactly what failed.

#### Purposes

- To return a consistent, structured error response for validation failures.
- To provide the appropriate HTTP status code for the error type.
- To enable clients to programmatically handle validation errors.
- To support customization of the error response format.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Default error response structure
{
    "message": "The given data was invalid.",
    "errors": {
        "field": ["Error message"]
    }
}

// Custom error response via failedValidation()
protected function failedValidation(Validator $validator)
{
    throw new HttpResponseException(
        response()->json([
            'status' => 'error',
            'message' => 'Validation failed',
            'errors' => $validator->errors(),
        ], 422)
    );
}
```

**Component Breakdown**

| Method | Description |
|--------|-------------|
| `failedValidation(Validator)` | Overrides the default validation failure response. |
| `response(array $errors)` | Customizes the response for specific scenarios. |
| `withValidator(Validator)` | Adds after-validation logic. |

**Syntax Rules**

- The default response includes `message` and `errors` keys.
- `failedValidation()` must throw an exception or return a response.
- The `response()` method can return a custom array.

**Constraints and Limitations**

- Overriding `failedValidation()` affects all validation failures for that Form Request.
- Custom response formats must be documented for API clients.

---

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Custom Validation Error Response**

**Form Request**

```php
<?php
// app/Http/Requests/ApiFormRequest.php
namespace App\Http\Requests;

use Illuminate\Contracts\Validation\Validator;
use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Http\Exceptions\HttpResponseException;

abstract class ApiFormRequest extends FormRequest
{
    protected function failedValidation(Validator $validator)
    {
        throw new HttpResponseException(
            response()->json([
                'status' => 'error',
                'message' => 'The provided data failed validation.',
                'errors' => $validator->errors(),
            ], 422)
        );
    }
}
```

**Usage**

```php
class StoreUserRequest extends ApiFormRequest
{
    public function rules(): array
    {
        return [
            'name' => ['required', 'string', 'max:255'],
            'email' => ['required', 'email', 'unique:users'],
        ];
    }
}
```

**Expected Output**

```json
{
    "status": "error",
    "message": "The provided data failed validation.",
    "errors": {
        "name": ["The name field is required."],
        "email": ["The email field is required."]
    }
}
```

**Why This Output Occurs**

The `ApiFormRequest` base class overrides `failedValidation()` to return a custom JSON response with a `status` key. All Form Requests extending it inherit this behavior.

---

**Example 2: Problem Details (RFC 9457) Response**

**Exception Handler**

```php
// bootstrap/app.php (Laravel 11+)
->withExceptions(function (Exceptions $exceptions) {
    $exceptions->render(function (ValidationException $e, Request $request) {
        if ($request->expectsJson()) {
            return response()->json([
                'type' => 'https://example.com/probs/validation-error',
                'title' => 'Validation Error',
                'status' => 422,
                'detail' => 'One or more fields failed validation.',
                'instance' => $request->path(),
                'errors' => $e->errors(),
            ], 422)->header('Content-Type', 'application/problem+json');
        }
    });
})
```

**Expected Output**

```json
{
    "type": "https://example.com/probs/validation-error",
    "title": "Validation Error",
    "status": 422,
    "detail": "One or more fields failed validation.",
    "instance": "api/users",
    "errors": {
        "email": ["The email field is required."]
    }
}
```

**Why This Output Occurs**

The exception handler overrides the default validation error response with a Problem Details structure, a standardized format for HTTP API errors. The `Content-Type` header is set to `application/problem+json`.

---

#### Real-World Cases with Explanation

**Case 1: Mobile App Error Handling**

A mobile app receives a 422 response with a custom `status` key and displays a toast message with the `message` value. It also parses the `errors` object to highlight invalid fields.

**Explanation**: Custom error responses enable consistent client-side error handling.

---

### 4. Error Formatting

#### Definitions

**Core Definition**

Error formatting is the process of customizing the structure, keys, and presentation of validation error responses to meet specific API design requirements.

**Technical Definition**

Laravel provides multiple extension points for error formatting: the `failedValidation()` method on Form Requests, the `render()` method on exceptions, the exception handler's `register()` method, and packages like `kdabrow/validation-codes` (adds rule codes) and `jccoca/laravel-api-exceptions` (custom exception responses). The `MessageBag` class provides methods for accessing and formatting errors, including `first()`, `all()`, `get()`, and `toArray()`.

**Beginner-Friendly Explanation**

Error formatting lets you change what the error response looks like. The default format works well, but sometimes your frontend team wants a different structure—like adding error codes, changing the key names, or grouping errors differently.

#### Purposes

- To match the API's error response format to frontend requirements.
- To add machine-readable error codes for programmatic handling.
- To comply with API standards like RFC 9457 (Problem Details) or JSON:API.
- To provide consistent error responses across all API endpoints.
- To include additional metadata in error responses.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Custom format via failedValidation()
protected function failedValidation(Validator $validator)
{
    $errors = collect($validator->errors()->toArray())
        ->map(fn ($messages) => $messages[0])
        ->toArray();

    throw new HttpResponseException(
        response()->json([
            'success' => false,
            'code' => 'VALIDATION_ERROR',
            'errors' => $errors,
        ], 422)
    );
}
```

```php
// Using a package for error codes (kdabrow/validation-codes)
// Response format:
{
    "message": "validation errors",
    "errors": {
        "user_id": ["Field user_id is required"]
    },
    "codes": {
        "user_id": ["E104"]
    }
}
```

**Component Breakdown**

| Format | Structure | Use Case |
|--------|-----------|----------|
| Default | `{message, errors}` | Standard Laravel APIs |
| Simplified | `{success, code, errors}` | Custom frontend requirements |
| Error Codes | `{message, errors, codes}` | Machine-readable error handling |
| Problem Details | `{type, title, status, detail, instance, errors}` | RFC 9457 compliance |

**Syntax Rules**

- Custom formatting can be applied globally via the exception handler or per Form Request.
- Error codes require a mapping of rules to codes.
- The `errors` key should contain field-keyed error information.

**Constraints and Limitations**

- Custom formats may break existing API clients if not versioned.
- Documentation must be updated to reflect custom formats.

---

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Adding Error Codes**

**Installation**

```bash
composer require kdabrow/validation-codes
```

**Publish Configuration**

```bash
php artisan vendor:publish --tag=validation_codes
```

**Configuration** (`config/validation_codes.php`)

```php
return [
    'fallback_error' => 'E0',
    'required' => 'E1',
    'email' => 'E2',
    'min' => 'E3',
    'max' => 'E4',
    'unique' => 'E5',
    // ...
];
```

**Expected Output**

```json
{
    "message": "validation errors",
    "errors": {
        "email": ["The email field is required."],
        "password": ["The password must be at least 8 characters."]
    },
    "codes": {
        "email": ["E1"],
        "password": ["E3"]
    }
}
```

**Why This Output Occurs**

The package extends Laravel's exception handler to include a `codes` key alongside `errors`. Each validation rule is mapped to a code (e.g., `required` → `E1`).

---

**Example 2: Grouped Error Format**

**Form Request**

```php
protected function failedValidation(Validator $validator)
{
    $errors = $validator->errors()->toArray();

    $groupedErrors = [];
    foreach ($errors as $field => $messages) {
        $groupedErrors[] = [
            'field' => $field,
            'messages' => $messages,
        ];
    }

    throw new HttpResponseException(
        response()->json([
            'status' => 'error',
            'error_count' => count($groupedErrors),
            'errors' => $groupedErrors,
        ], 422)
    );
}
```

**Expected Output**

```json
{
    "status": "error",
    "error_count": 2,
    "errors": [
        {
            "field": "email",
            "messages": ["The email field is required."]
        },
        {
            "field": "password",
            "messages": ["The password must be at least 8 characters."]
        }
    ]
}
```

**Why This Output Occurs**

The `failedValidation()` method transforms the field-keyed errors object into an array of objects with `field` and `messages` keys, providing a different structure for clients that prefer arrays over objects.

---

#### Real-World Cases with Explanation

**Case 1: Multi-Language API**

An API supports multiple languages and uses custom error formatting to include translated error messages. The `codes` key allows the frontend to map error codes to translations in the client's language.

**Explanation**: Error codes decouple error identification from error messages, enabling client-side internationalization.

**Case 2: GraphQL-Style Errors**

An API mimics GraphQL's error format by including `path`, `extensions`, and `message` keys in validation errors, providing a consistent experience for clients familiar with GraphQL.

**Explanation**: Custom error formatting allows API designers to align error responses with specific client expectations.

---

### 5. Error Status Codes

#### Definitions

**Core Definition**

Error status codes are the HTTP status codes returned with API error responses, indicating the type and category of the error.

**Technical Definition**

Laravel uses standard HTTP status codes for API errors. Validation failures return **422 Unprocessable Entity**. Other common codes include **400 Bad Request** (malformed request syntax), **401 Unauthorized** (missing or invalid authentication), **403 Forbidden** (authenticated but not authorized), **404 Not Found** (resource does not exist), **429 Too Many Requests** (rate limited), and **500 Internal Server Error** (unexpected server error).

**Beginner-Friendly Explanation**

Error status codes are numbers that tell the client what kind of error occurred. 422 means "the data you sent is invalid." 401 means "you need to log in." 403 means "you are logged in but not allowed." Using the right code helps clients handle errors correctly.

#### Purposes

- To provide a standardized indication of the error type.
- To enable clients to handle errors programmatically based on status code.
- To comply with HTTP specifications and REST conventions.
- To distinguish between different error categories (client vs. server errors).

#### Syntax Rules and Structure

**Status Code Reference**

| Code | Name | When to Use |
|------|------|-------------|
| 400 | Bad Request | Malformed request syntax or invalid parameters. |
| 401 | Unauthorized | Missing or invalid authentication credentials. |
| 403 | Forbidden | Authenticated but not authorized for the action. |
| 404 | Not Found | The requested resource does not exist. |
| 405 | Method Not Allowed | The HTTP method is not supported for the route. |
| 422 | Unprocessable Entity | Validation failed. |
| 429 | Too Many Requests | Rate limit exceeded. |
| 500 | Internal Server Error | Unexpected server error. |

**Syntax Rules**

- Validation failures automatically return 422 in Laravel.
- Custom status codes can be set via `response()->json($data, $statusCode)`.
- The `abort()` helper can throw HTTP exceptions with specific codes.

**Constraints and Limitations**

- Using incorrect status codes can confuse API clients and break integrations.
- Some clients may not handle non-standard status codes gracefully.

---

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Validation Error (422)**

**Request**

```bash
curl -X POST http://api.example.com/api/users \
  -H "Accept: application/json" \
  -d '{}'
```

**Response Headers**

```
HTTP/1.1 422 Unprocessable Entity
Content-Type: application/json
```

**Expected Output**

```json
{
    "message": "The name field is required.",
    "errors": {
        "name": ["The name field is required."]
    }
}
```

**Why This Output Occurs**

The validation failure triggers a `ValidationException`, which Laravel renders as a 422 JSON response.

---

**Example 2: Custom Status Code**

```php
// Custom response with 400 status
return response()->json([
    'error' => 'Invalid request format',
], 400);
```

**Expected Output**

```json
{
    "error": "Invalid request format"
}
```

**Why This Output Occurs**

The `response()->json()` method accepts a status code as the second argument, allowing custom status codes for specific error scenarios.

---

#### Real-World Cases with Explanation

**Case 1: E-commerce API**

An e-commerce API returns 422 for validation errors, 404 for missing products, and 429 for rate-limited requests. The mobile app handles each status code differently: 422 highlights form fields, 404 shows a "not found" page, and 429 displays a "slow down" message.

**Explanation**: Correct status codes enable precise client-side error handling.

---

### 6. API Validation Best Practices

#### Definitions

**Core Definition**

API validation best practices are the recommended guidelines for designing, implementing, and maintaining validation logic in Laravel APIs to maximise security, consistency, and developer experience.

**Technical Definition**

Best practices encompass using Form Request classes, returning appropriate HTTP status codes, providing field-keyed error messages, using error codes for machine-readable errors, testing validation thoroughly, documenting error formats, and avoiding common pitfalls such as returning 200 for errors or exposing internal error details.

**Beginner-Friendly Explanation**

Best practices are the tips experienced developers follow to make their API validation reliable, secure, and easy for clients to work with.

#### Key Best Practices

1. **Use Form Request Classes for All API Validation**: Extract validation rules and authorization logic into dedicated Form Request classes. This keeps controllers thin and validation reusable across endpoints.

2. **Always Use `validated()` for Mass Assignment**: Never use `$request->all()` for mass operations. Use `$request->validated()` to ensure only validated data is passed to `create()` or `update()`.

3. **Return 422 for Validation Errors**: Always return a 422 Unprocessable Entity status code for validation failures. Do not return 200 when errors occur.

4. **Provide Field-Keyed Error Messages**: Return errors as a field-keyed object so clients can display messages next to the corresponding form fields.

5. **Use Error Codes for Machine-Readable Errors**: Add error codes (e.g., `E104`) to validation responses so clients can handle errors programmatically without parsing messages.

6. **Customize Error Messages and Attributes**: Override `messages()` and `attributes()` to provide user-friendly, localized error messages and field names.

7. **Never Expose Internal Errors**: Ensure `APP_DEBUG=false` in production to prevent stack traces and SQL errors from being returned to clients.

8. **Use `prepareForValidation()` to Sanitize Input**: Normalise input (e.g., lowercase emails, trim whitespace) before validation rules are applied.

9. **Test Validation Thoroughly**: Use `assertJsonValidationErrors()` and `assertOnlyJsonValidationErrors()` to verify validation behavior in feature tests.

10. **Document Your Error Format**: Provide clear documentation of the API's error response structure, including status codes, error keys, and example responses.

11. **Consider RFC 9457 Problem Details**: For standards-compliant APIs, use the Problem Details format for error responses, which includes `type`, `title`, `status`, `detail`, and `instance` keys.

12. **Use OpenAPI Specs for Validation**: Consider using OpenAPI specifications as the source of truth for both API documentation and validation, ensuring consistency between the two.

13. **Use `dontExpandRelationshipsByDefault()` for Logic-Only Tests**: When testing validation logic that does not require related models, disable relationship expansion to speed up tests.

14. **Version Your API Error Format**: If you change the error response format, version your API to avoid breaking existing clients.

15. **Monitor Validation Failures**: Track validation failure rates and common failure reasons to identify usability issues or potential attacks.

---

#### Multiple Annotated Complete Step by Step Code Example and Expected Output

**Example 1: Production-Ready API Validation**

**Base API Form Request**

```php
<?php
// app/Http/Requests/ApiFormRequest.php
namespace App\Http\Requests;

use Illuminate\Contracts\Validation\Validator;
use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Http\Exceptions\HttpResponseException;

abstract class ApiFormRequest extends FormRequest
{
    protected function failedValidation(Validator $validator)
    {
        throw new HttpResponseException(
            response()->json([
                'status' => 'error',
                'message' => 'Validation failed.',
                'errors' => $validator->errors(),
            ], 422)
        );
    }
}
```

**Specific Form Request**

```php
<?php
// app/Http/Requests/StoreOrderRequest.php
namespace App\Http\Requests;

use App\Rules\SufficientInventoryRule;
use Illuminate\Validation\Rule;

class StoreOrderRequest extends ApiFormRequest
{
    public function authorize(): bool
    {
        return $this->user()->can('create', Order::class);
    }

    public function rules(): array
    {
        return [
            'customer_email' => ['required', 'string', 'email:rfc,dns', 'max:255'],
            'items' => ['required', 'array', 'min:1'],
            'items.*.product_id' => ['required', 'integer', 'exists:products,id'],
            'items.*.quantity' => ['required', 'integer', 'min:1', 'max:100', new SufficientInventoryRule()],
            'shipping_method' => ['required', 'string', Rule::in(['standard', 'express'])],
            'express_notes' => [
                Rule::when($this->input('shipping_method') === 'express', ['required', 'string', 'max:500']),
            ],
        ];
    }

    public function messages(): array
    {
        return [
            'customer_email.required' => 'We need your email address to process the order.',
            'customer_email.email' => 'Please enter a valid email address.',
            'items.required' => 'Your order must contain at least one item.',
            'items.*.product_id.exists' => 'One of the selected products is no longer available.',
            'items.*.quantity.max' => 'You cannot order more than :max of a single item.',
            'express_notes.required' => 'Please provide delivery notes for express shipping.',
        ];
    }

    public function attributes(): array
    {
        return [
            'customer_email' => 'email address',
            'items.*.product_id' => 'product',
            'items.*.quantity' => 'quantity',
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
<?php
// app/Http/Controllers/Api/OrderController.php
namespace App\Http\Controllers\Api;

use App\Http\Requests\StoreOrderRequest;
use App\Http\Resources\OrderResource;
use App\Models\Order;

class OrderController extends Controller
{
    public function store(StoreOrderRequest $request)
    {
        $order = Order::create($request->validated());

        return new OrderResource($order);
    }
}
```

**Route**

```php
// routes/api.php
Route::middleware(['auth:sanctum', 'throttle:api'])
    ->post('/orders', [OrderController::class, 'store']);
```

**Expected Output (Validation Failure)**

```json
{
    "status": "error",
    "message": "Validation failed.",
    "errors": {
        "customer_email": ["We need your email address to process the order."],
        "items.0.product_id": ["One of the selected products is no longer available."],
        "items.1.quantity": ["You cannot order more than 100 of a single item."],
        "shipping_method": ["Express shipping is not available for orders over 50 items."]
    }
}
```

**Response Headers**

```
HTTP/1.1 422 Unprocessable Entity
Content-Type: application/json
```

**Why This Output Occurs**

The `StoreOrderRequest` follows all best practices:
- Extends a base `ApiFormRequest` that customizes the error format.
- Uses nested array validation with wildcards.
- Provides custom messages and attributes.
- Uses `Rule::when()` for conditional validation.
- Uses `after()` for multi-field validation.
- Uses `prepareForValidation()` to sanitize input.
- The controller uses `$request->validated()` for mass assignment.
- The route applies rate limiting and authentication middleware.

---

#### Real-World Cases with Explanation

**Case 1: Enterprise SaaS API**

A SaaS platform's API follows all validation best practices: Form Requests for all endpoints, 422 status codes, field-keyed errors with error codes, custom messages, and comprehensive tests. The API serves mobile apps, SPAs, and third-party integrations with consistent error handling.

**Explanation**: Following validation best practices ensures the API is secure, reliable, and easy to integrate with.

---

## References

- Validation – Laravel 12.x Documentation — https://laravel.com/docs/12.x/validation
- Validation – Laravel 11.x Documentation — https://laravel.com/docs/11.x/validation
- Validation – Laravel 10.x Documentation — https://laravel.com/docs/10.x/validation
- Validation – Laravel master Documentation — https://laravel.com/framework/docs/master/validation
- HTTP Tests: Validation Errors – Laravel 12.x Documentation — https://laravel.com/docs/12.x/http-tests#assert-validation-errors
- Use Form Request Classes – Dutch Laravel Foundation Best Practices — https://github.com/Dutch-Laravel-Foundation/best-practices/blob/main/routing/use-form-request-classes.md
- Validation Codes for Laravel – GitHub (kdabrow/validation-codes) — https://github.com/karoldabro/validation-codes
- Laravel OpenAPI Validator – GitHub (kirschbaum-development) — https://github.com/kirschbaum-development/laravel-openapi-validator
- Enhance Validation Testing Precision with Laravel's assertOnlyJsonValidationErrors – Laravel News — https://laravel-news.com/assert-only-json-validation-errors
- 6 Bad Practices When Building Laravel APIs – Laravel Daily — https://laraveldaily.com/post/6-bad-practices-when-building-laravel-apis
- Validation, Errors and Status Codes – Laravel Daily — https://laraveldaily.com/lesson/api-laravel/validation-errors-and-status-codes
- Customizing Laravel Validation JSON Message Format – Stack Overflow — https://stackoverflow.com/questions/68611194/customizing-laravel-validation-json-message-format
- ECC/examples/laravel-api-CLAUDE.md – GitHub (affaan-m) — https://github.com/affaan-m/ECC/blob/main/examples/laravel-api-CLAUDE.md
- Laravel API Exceptions – Packagist (jccoca/laravel-api-exceptions) — https://packagist.org/packages/jccoca/laravel-api-exceptions
- Validation | Laravel 10.x – Customizing the Error Messages — https://laravel.com/framework/docs/10.x/validation#2

---

**Version Note**: This cheat sheet is based on Laravel 10.x through 13.x documentation. Key version-specific features include: the `failedValidation()` method for custom error responses (Laravel 10+); the `after()` method for Form Requests (Laravel 10+); RFC 9457 Problem Details support (Laravel 13+); and the `install:api` command for API scaffolding (Laravel 11+). Packages like `kdabrow/validation-codes` provide error code support across Laravel 8–13. Developers should always consult the documentation for their specific Laravel version.