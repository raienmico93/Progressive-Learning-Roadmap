# Laravel Validation Rules: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Laravel Validation is a system for verifying that incoming application data meets specified criteria before it is processed or stored, protecting the application from invalid, malformed, or malicious input.

**Technical Definition**

Laravel's validation system is implemented by the `Validator` class, which accepts input data and an array of rule definitions. Each rule is applied to a corresponding field, and failures are collected in a `MessageBag`. Rules can be expressed as strings (e.g., `'required|email'`), arrays, `Rule` objects, or closures. The system supports conditional rules, custom rule objects, implicit rules, and integration with Form Requests for automatic validation before controller actions.

**Beginner-Friendly Explanation**

Validation is like a security checkpoint for data entering your application. When a user submits a form, Laravel checks that every field meets your rules—like "this field must be filled in," "this must be a valid email," or "this number must be between 1 and 100." If anything fails the check, Laravel sends the user back with helpful error messages instead of letting bad data into your database.

---

### Key Characteristics

- **Declarative Rule Definitions**: Rules are defined as readable strings or fluent objects, separating validation logic from controller code.
- **Comprehensive Rule Library**: Over 60 built-in rules cover presence, data types, formats, numeric ranges, dates, database checks, and file uploads.
- **Conditional Validation**: Rules like `required_if`, `required_unless`, `prohibited`, and `prohibits` enable dynamic validation based on other field values.
- **Custom Rule Objects**: Developers can create reusable rule classes implementing `ValidationRule` for complex or application-specific validation.
- **Form Request Integration**: Validation rules can be encapsulated in dedicated Form Request classes, keeping controllers clean.
- **Automatic Redirection**: On failure, Laravel automatically redirects back with errors and old input for web requests.
- **Localized Error Messages**: Error messages are stored in `lang/en/validation.php` and can be customized or translated.

---

### Prerequisites

- **PHP 8.1 or higher** (Laravel 10+) or **PHP 8.2** (Laravel 11+).
- **A Laravel application** installed via Composer.
- **Basic knowledge of HTTP requests, controllers, and Blade**.
- **Familiarity with Laravel's `Request` object**.
- **A working database connection** for rules like `exists`, `unique`, and `current_password`.

---

### Related Programming Areas

- **Form Requests**: Dedicated classes for encapsulating validation logic.
- **Authorization**: Validation and authorization often run together in Form Requests.
- **Database**: Rules like `exists`, `unique`, and `current_password` interact with the database.
- **File Storage**: File and image rules validate uploads before storage.
- **Localization**: Validation messages are translatable via language files.

---

## Core Concepts

---

### 1. Presence & Requirement Rules

#### Definitions

**Core Definition**

Presence and requirement rules determine whether a field must be present in the input data, whether it may be empty, and whether it must contain a specific value.

**Technical Definition**

These rules control the existence and emptiness of fields. "Empty" in Laravel means `null`, an empty string, an empty array or `Countable`, or an uploaded file with an empty path. `required` demands presence and non-emptiness; `present` demands presence but allows emptiness; `nullable` allows `null`; `filled` demands non-emptiness only if present; `missing` demands absence; `accepted` and `declined` check for specific affirmative/negative values.

**Beginner-Friendly Explanation**

These rules answer questions like "Is this field here?" "Can it be blank?" and "Does it contain the right confirmation value?"

#### Purposes

- To enforce that critical fields are provided and not left blank.
- To allow optional fields to be omitted or null.
- To validate checkbox and toggle confirmations (accepted, declined).
- To ensure that certain keys exist within an array (required_array_keys).

#### Syntax Rules and Structure

**Complete General Syntax**

```php
'field' => 'required|nullable|present|filled|missing|accepted|declined',
'permissions' => 'required|array|required_array_keys:read,write',
```

**Component Breakdown**

| Rule | Behavior |
|------|----------|
| `required` | Field must be present and not empty. |
| `nullable` | Field may be `null`. |
| `present` | Field must be present but may be empty. |
| `filled` | Field must not be empty when present. |
| `missing` | Field must not be present. |
| `accepted` | Field must be `"yes"`, `"on"`, `1`, `"1"`, `true`, or `"true"`. |
| `declined` | Field must be `"no"`, `"off"`, `0`, `"0"`, `false`, or `"false"`. |
| `required_array_keys` | Array must contain at least the specified keys. |

**Syntax Rules**

- Presence rules are often combined with data type rules.
- `nullable` must be used with other rules to allow `null` values to skip them.
- `required_array_keys` requires the `array` rule as a companion.

**Constraints and Limitations**

- `required` and `nullable` together are contradictory for non-null values; use `nullable` alone for optional fields.
- `accepted` and `declined` are designed for checkboxes and terms acceptance.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Required and Nullable**

**Code**

```php
$request->validate([
    'name' => 'required|string|max:255',
    'bio' => 'nullable|string|max:1000',
]);
```

**Expected Output**

- Missing `name` → validation error.
- `bio` omitted or `null` → passes.

**Why This Output Occurs**

`required` enforces `name`; `nullable` allows `bio` to be absent or `null` without triggering other rules.

---

**Example 2: Accepted and Declined**

**Code**

```php
$request->validate([
    'terms' => 'accepted',
    'newsletter' => 'declined',
]);
```

**Expected Output**

- `terms = "on"` → passes; `terms = "no"` → fails.
- `newsletter = "no"` → passes; `newsletter = "yes"` → fails.

**Why This Output Occurs**

`accepted` requires an affirmative value; `declined` requires a negative value.

---

**Example 3: Required Array Keys**

**Code**

```php
$request->validate([
    'permissions' => 'required|array|required_array_keys:read,write',
]);
```

**Expected Output**

- `['read' => true, 'write' => true]` → passes.
- `['read' => true]` → fails (missing `write`).

**Why This Output Occurs**

`required_array_keys` ensures all specified keys exist in the array.

---

#### Real-World Cases with Explanation

**Case 1: Registration Form Terms Acceptance**

A registration form uses `accepted` for the terms checkbox, ensuring users explicitly agree before creating an account.

**Explanation**: `accepted` enforces affirmative consent for legal or policy agreements.

---

### 2. Conditional Requirements

#### Definitions

**Core Definition**

Conditional requirement rules make a field's required, prohibited, or present status dependent on the value or presence of other fields.

**Technical Definition**

Rules like `required_if`, `required_unless`, `required_with`, `required_without`, `prohibited`, and `prohibits` evaluate the state of other fields to determine whether the target field must be present, must be empty, or must be absent. These rules use the `anotherfield` parameter to reference sibling fields.

**Beginner-Friendly Explanation**

These rules say "only require this field if that other field has a certain value" or "don't allow this field if that one is filled in."

#### Purposes

- To implement dynamic forms where requirements depend on user choices.
- To prevent conflicting fields from being submitted together.
- To enforce mutually exclusive or dependent field relationships.
- To simplify complex validation logic without custom closures.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
'field' => 'required_if:anotherfield,value1,value2',
'field' => 'required_unless:anotherfield,value',
'field' => 'required_with:foo,bar',
'field' => 'required_with_all:foo,bar',
'field' => 'required_without:foo,bar',
'field' => 'required_without_all:foo,bar',
'field' => 'prohibited',
'field' => 'prohibits:anotherfield1,anotherfield2',
'field' => 'required_if_accepted:anotherfield',
```

**Component Breakdown**

| Rule | Behavior |
|------|----------|
| `required_if` | Required if another field equals any listed value. |
| `required_if_accepted` | Required if another field is accepted (`yes`, `on`, `1`, etc.). |
| `required_unless` | Required unless another field equals a value. |
| `required_with` | Required if any listed fields are present. |
| `required_with_all` | Required if all listed fields are present. |
| `required_without` | Required if any listed fields are absent. |
| `required_without_all` | Required if all listed fields are absent. |
| `prohibited` | Field must be missing or empty. |
| `prohibits` | If this field is present, the listed fields must be missing or empty. |

**Syntax Rules**

- Conditional rules reference sibling fields by name.
- `required_if` and `required_unless` accept multiple values (OR logic).
- `prohibits` is the inverse of `required_with` in terms of field relationships.

**Constraints and Limitations**

- Complex conditions may require `Rule::requiredIf()` with a closure.
- Conditional rules do not automatically make a field nullable; combine with `nullable` where appropriate.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Required If**

**Code**

```php
$request->validate([
    'is_company' => 'boolean',
    'company_name' => 'required_if:is_company,true|nullable|string',
]);
```

**Expected Output**

- `is_company = true` and no `company_name` → fails.
- `is_company = false` → passes without `company_name`.

**Why This Output Occurs**

`required_if` makes `company_name` required only when `is_company` is `true`.

---

**Example 2: Prohibited**

**Code**

```php
$request->validate([
    'coupon_code' => 'prohibited_if:payment_method,credit_card',
]);
```

**Expected Output**

- `payment_method = "credit_card"` with `coupon_code` → fails.
- `payment_method = "paypal"` with `coupon_code` → passes.

**Why This Output Occurs**

`prohibited_if` prevents the field from being present under the specified condition.

---

**Example 3: Prohibits**

**Code**

```php
$request->validate([
    'is_gift' => 'boolean|prohibits:gift_message',
]);
```

**Expected Output**

- `is_gift = true` and `gift_message` present → fails.
- `is_gift = false` and `gift_message` present → passes.

**Why This Output Occurs**

`prohibits` ensures that if `is_gift` is present, `gift_message` must be absent or empty.

---

#### Real-World Cases with Explanation

**Case 1: Business Account Registration**

A registration form requires a company name only if the user selects "Business Account" (`required_if:account_type,business`).

**Explanation**: Conditional rules enable dynamic forms that adapt to user choices.

---

### 3. Data Types & Formats

#### Definitions

**Core Definition**

Data type and format rules verify that a field contains a value of a specific type (e.g., integer, string, array) or matches a recognized format (e.g., email, date, UUID, JSON).

**Technical Definition**

These rules use PHP type checks, regular expressions, and built-in functions to validate the structural correctness of data. Examples include `integer` (checks if the value is an integer or integer-like string), `string`, `array`, `boolean`, `date` (validates via `strtotime`), `json` (validates via `json_decode`), `enum` (validates against a PHP enum class), and `timezone` (validates against `timezone_identifiers_list()`).

**Beginner-Friendly Explanation**

These rules check that a field "looks like" what it's supposed to be—a number where a number is expected, a valid email address, a properly formatted date, and so on.

#### Purposes

- To ensure data conforms to expected types before processing.
- To validate formats like email addresses, URLs, and dates without custom regex.
- To enforce that JSON fields contain valid JSON.
- To validate enum values against PHP enums.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
'age' => 'integer',
'name' => 'string|max:255',
'tags' => 'array',
'is_active' => 'boolean',
'birthday' => 'date',
'metadata' => 'json',
'status' => [new Enum(Status::class)],
'timezone' => 'timezone',
'username' => 'alpha_dash',
'code' => 'alpha_num',
```

**Component Breakdown**

| Rule | Validates |
|------|-----------|
| `alpha` | Only alphabetic characters. |
| `alpha_dash` | Alphabetic, numeric, dashes, underscores. |
| `alpha_num` | Only alphabetic and numeric characters. |
| `array` | Value is a PHP array. |
| `boolean` | Value is a boolean-like value. |
| `date` | Value is a valid date (via `strtotime`). |
| `enum` | Value is a valid case of a PHP enum. |
| `integer` | Value is an integer (or integer-like string). |
| `json` | Value is valid JSON. |
| `numeric` | Value is numeric. |
| `string` | Value is a string. |
| `timezone` | Value is a valid timezone identifier. |

**Syntax Rules**

- `integer` accepts both integers and strings that are integers (e.g., `"123"`).
- `date` should not be combined with `date_format`; use one or the other.
- `enum` requires PHP 8.1+ and accepts an enum class name.

**Constraints and Limitations**

- `boolean` accepts `true`, `false`, `1`, `0`, `"1"`, `"0"` but not `"true"`/`"false"` strings (unless using `boolean` with `Rule::in`).
- `json` validates structure, not schema.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Integer and Boolean**

**Code**

```php
$request->validate([
    'quantity' => 'required|integer|min:1',
    'is_gift' => 'boolean',
]);
```

**Expected Output**

- `quantity = "5"` → passes (integer-like string).
- `quantity = "abc"` → fails.
- `is_gift = 1` → passes; `is_gift = "yes"` → fails.

**Why This Output Occurs**

`integer` accepts integer-like strings; `boolean` accepts `1`, `0`, `true`, `false`.

---

**Example 2: Enum Validation**

**Code**

```php
use App\Enums\OrderStatus;

$request->validate([
    'status' => ['required', new Enum(OrderStatus::class)],
]);
```

**Expected Output**

- `status = "pending"` (valid enum case) → passes.
- `status = "invalid"` → fails.

**Why This Output Occurs**

The `Enum` rule checks that the value is a valid case of the specified enum class.

---

**Example 3: JSON and Array**

**Code**

```php
$request->validate([
    'settings' => 'json',
    'tags' => 'array',
]);
```

**Expected Output**

- `settings = '{"theme":"dark"}'` → passes.
- `settings = 'not-json'` → fails.
- `tags = ["a","b"]` → passes.

**Why This Output Occurs**

`json` validates the string via `json_decode`; `array` checks the PHP type.

---

#### Real-World Cases with Explanation

**Case 1: API Payload Validation**

An API accepts JSON payloads and uses `json` to ensure the `metadata` field contains valid JSON before decoding it for processing.

**Explanation**: Format rules prevent malformed data from reaching application logic.

---

### 4. Numbers, Sizes & Ranges

#### Definitions

**Core Definition**

Numeric range and size rules constrain the magnitude, digit count, length, or multiplicity of a value.

**Technical Definition**

These rules evaluate numeric values, string lengths, array counts, and file sizes. `between`, `min`, `max`, and `size` use the same size conventions across strings (character count), numerics (numeric value), arrays (element count), and files (kilobytes). `digits` and `digits_between` apply only to integers. `gt`, `gte`, `lt`, and `lte` compare against another field or a literal value. `multiple_of` checks divisibility.

**Beginner-Friendly Explanation**

These rules control "how big" or "how many" something is—like a password being at least 8 characters, a quantity being between 1 and 100, or a file being no more than 2MB.

#### Purposes

- To enforce minimum and maximum lengths for strings.
- To constrain numeric values to acceptable ranges.
- To validate digit counts for integers (e.g., PIN codes).
- To check divisibility (e.g., quantities in multiples of 5).
- To compare one field's value against another.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
'name' => 'string|between:2,50',
'age' => 'integer|min:18|max:120',
'quantity' => 'integer|multiple_of:5',
'pin' => 'digits:4',
'code' => 'digits_between:4,6',
'end_date' => 'date|after:start_date',
'price' => 'numeric|gt:0',
```

**Component Breakdown**

| Rule | Behavior |
|------|----------|
| `between:min,max` | Size/value between min and max (inclusive). |
| `digits:value` | Integer must have exactly `value` digits. |
| `digits_between:min,max` | Integer digit count between min and max. |
| `gt:field` | Greater than another field or value. |
| `gte:field` | Greater than or equal. |
| `lt:field` | Less than another field or value. |
| `lte:field` | Less than or equal. |
| `max:value` | Maximum size/value. |
| `min:value` | Minimum size/value. |
| `multiple_of:value` | Numeric value is a multiple of `value`. |
| `size:value` | Exact size (string length, array count, file KB, numeric value). |

**Syntax Rules**

- `between`, `min`, `max`, and `size` evaluate differently based on the data type.
- `gt`, `gte`, `lt`, `lte` can reference another field by name.
- `digits` and `digits_between` only apply to integers.

**Constraints and Limitations**

- `size` for numeric values checks the numeric value itself, not digit count (use `digits` for that).
- `multiple_of` only works with numeric values.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Between and Digits**

**Code**

```php
$request->validate([
    'age' => 'integer|between:18,120',
    'pin' => 'digits:4',
]);
```

**Expected Output**

- `age = 25` → passes; `age = 15` → fails.
- `pin = "1234"` → passes; `pin = "123"` → fails.

**Why This Output Occurs**

`between` constrains the numeric value; `digits` requires exactly 4 digits.

---

**Example 2: Greater Than and Multiple Of**

**Code**

```php
$request->validate([
    'price' => 'numeric|gt:0',
    'quantity' => 'integer|multiple_of:5',
]);
```

**Expected Output**

- `price = 9.99` → passes; `price = 0` → fails.
- `quantity = 10` → passes; `quantity = 7` → fails.

**Why This Output Occurs**

`gt:0` requires price to be greater than 0; `multiple_of:5` requires quantity to be divisible by 5.

---

#### Real-World Cases with Explanation

**Case 1: Password Length**

A registration form uses `min:8` and `max:72` for passwords, aligning with bcrypt's 72-byte limit.

**Explanation**: Size rules enforce security and technical constraints.

---

### 5. Strings & Text Patterns

#### Definitions

**Core Definition**

String pattern rules validate the content of strings against prefixes, suffixes, regular expressions, and identifier formats like UUIDs and ULIDs.

**Technical Definition**

`starts_with` and `ends_with` check literal prefixes/suffixes. `regex` and `not_regex` apply PCRE patterns. `uuid` validates RFC 4122 UUIDs, and `ulid` validates ULIDs (Universally Unique Lexicographically Sortable Identifiers). These rules operate on string values only.

**Beginner-Friendly Explanation**

These rules check that text starts with, ends with, or matches a specific pattern—like a username starting with a letter, a file ending in `.pdf`, or a code being a valid UUID.

#### Purposes

- To enforce naming conventions (e.g., prefixes).
- To validate file extensions or MIME-related suffixes.
- To apply custom pattern matching with regular expressions.
- To validate modern identifier formats (UUID, ULID).

#### Syntax Rules and Structure

**Complete General Syntax**

```php
'username' => 'starts_with:user_',
'filename' => 'ends_with:.pdf,.docx',
'code' => 'regex:/^[A-Z]{3}-\d{3}$/',
'identifier' => 'uuid',
'order_id' => 'ulid',
```

**Component Breakdown**

| Rule | Behavior |
|------|----------|
| `starts_with:foo,bar` | String must start with one of the values. |
| `ends_with:foo,bar` | String must end with one of the values. |
| `regex:pattern` | String must match the PCRE pattern. |
| `not_regex:pattern` | String must not match the pattern. |
| `uuid` | String must be a valid UUID. |
| `ulid` | String must be a valid ULID. |

**Syntax Rules**

- `regex` patterns must include delimiters (e.g., `/pattern/`).
- When using `regex` with the `|` delimiter, use an array of rules to avoid conflicts.
- `starts_with` and `ends_with` accept multiple values (OR logic).

**Constraints and Limitations**

- `regex` patterns can be complex and should be tested.
- `uuid` and `ulid` validate format, not uniqueness.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Starts With and Ends With**

**Code**

```php
$request->validate([
    'referral_code' => 'starts_with:REF-',
    'document' => 'ends_with:.pdf,.docx',
]);
```

**Expected Output**

- `referral_code = "REF-123"` → passes; `"ABC-123"` → fails.
- `document = "file.pdf"` → passes; `"file.exe"` → fails.

**Why This Output Occurs**

`starts_with` and `ends_with` check literal prefixes and suffixes.

---

**Example 2: Regex and UUID**

**Code**

```php
$request->validate([
    'license_plate' => 'regex:/^[A-Z]{2}-\d{2}-[A-Z]{2}$/',
    'device_id' => 'uuid',
]);
```

**Expected Output**

- `license_plate = "AB-12-CD"` → passes; `"ABC-123"` → fails.
- `device_id = "550e8400-e29b-41d4-a716-446655440000"` → passes.

**Why This Output Occurs**

`regex` applies the pattern; `uuid` validates the RFC 4122 format.

---

#### Real-World Cases with Explanation

**Case 1: Referral Code Format**

A marketing system requires referral codes to start with `REF-`, enforced by `starts_with:REF-`.

**Explanation**: Pattern rules enforce business-specific naming conventions.

---

### 6. Networks & Identifiers

#### Definitions

**Core Definition**

Network and identifier rules validate email addresses, URLs, IP addresses, and DNS-resolvable hostnames.

**Technical Definition**

`email` validates RFC-compliant email addresses (optionally with DNS checking via `email:dns`). `url` validates URLs per RFC 3986. `active_url` requires the hostname to have a valid A or AAAA DNS record. `ip`, `ipv4`, and `ipv6` validate IP address formats using PHP's `filter_var()`.

**Beginner-Friendly Explanation**

These rules check that a field contains a properly formatted email address, website URL, or IP address—and optionally that the domain actually exists.

#### Purposes

- To validate contact information (email addresses).
- To validate website URLs in user profiles.
- To ensure IP addresses are correctly formatted.
- To optionally verify that a domain resolves via DNS (active_url).

#### Syntax Rules and Structure

**Complete General Syntax**

```php
'email' => 'required|email:rfc,dns',
'website' => 'url',
'homepage' => 'active_url',
'server_ip' => 'ip',
'ipv4_address' => 'ipv4',
'ipv6_address' => 'ipv6',
```

**Component Breakdown**

| Rule | Behavior |
|------|----------|
| `email` | Valid email format; optional `rfc`, `strict`, `dns`, `spoof`, `filter` parameters. |
| `url` | Valid URL. |
| `active_url` | URL with a valid DNS A/AAAA record. |
| `ip` | Valid IPv4 or IPv6 address. |
| `ipv4` | Valid IPv4 address. |
| `ipv6` | Valid IPv6 address. |

**Syntax Rules**

- `email` can accept parameters like `rfc`, `dns`, `spoof`, `filter`, and `strict`.
- `active_url` performs a DNS lookup and may be slow; use sparingly.
- `ip` accepts both IPv4 and IPv6.

**Constraints and Limitations**

- `active_url` requires network access and can fail in offline environments.
- `email:dns` requires the `intl` extension in some configurations.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Email with DNS**

**Code**

```php
$request->validate([
    'email' => 'required|email:rfc,dns',
]);
```

**Expected Output**

- `user@example.com` (with valid MX record) → passes.
- `user@nonexistent-domain-xyz.com` → fails.

**Why This Output Occurs**

The `dns` parameter verifies that the domain has valid DNS records.

---

**Example 2: IP Addresses**

**Code**

```php
$request->validate([
    'ip' => 'ip',
    'ipv4' => 'ipv4',
    'ipv6' => 'ipv6',
]);
```

**Expected Output**

- `ip = "192.168.1.1"` → passes; `"999.999.999.999"` → fails.
- `ipv4 = "10.0.0.1"` → passes; `ipv6 = "::1"` → passes for `ipv6` but fails for `ipv4`.

**Why This Output Occurs**

The rules use PHP's `filter_var()` with the corresponding IP flags.

---

#### Real-World Cases with Explanation

**Case 1: User Registration Email**

A registration form uses `email:rfc,dns` to ensure the email is syntactically valid and the domain exists, reducing fake registrations.

**Explanation**: DNS validation adds an extra layer of confidence in email addresses.

---

### 7. Dates & Times

#### Definitions

**Core Definition**

Date and time rules validate that a value is a valid date, matches a specific format, or falls before, after, or on a specified date.

**Technical Definition**

`date` validates via `strtotime()`. `date_format` validates against a specific format using `DateTime::createFromFormat()`. `after`, `after_or_equal`, `before`, `before_or_equal`, and `date_equals` compare against a date string, another field, or `today`/`now`. The dates are parsed by PHP's `strtotime()`.

**Beginner-Friendly Explanation**

These rules check that a date is real and falls within the right timeframe—like a booking date being after today or a birthdate being before today.

#### Purposes

- To ensure a field contains a valid date.
- To enforce that dates fall within a specific range.
- To validate custom date formats (e.g., `d/m/Y`).
- To compare two date fields (e.g., end date after start date).

#### Syntax Rules and Structure

**Complete General Syntax**

```php
'start_date' => 'date|after_or_equal:today',
'end_date' => 'date|after:start_date',
'birthday' => 'date|before:today',
'appointment' => 'date_format:Y-m-d H:i',
'deadline' => 'date|date_equals:2026-12-31',
```

**Component Breakdown**

| Rule | Behavior |
|------|----------|
| `after:date` | Date must be after the given date. |
| `after_or_equal:date` | Date must be after or equal. |
| `before:date` | Date must be before. |
| `before_or_equal:date` | Date must be before or equal. |
| `date_equals:date` | Date must equal the given date. |
| `date_format:format` | Date must match the given format. |

**Syntax Rules**

- Use either `date` or `date_format`, not both.
- `after` and `before` accept another field name or a literal date string.
- `today` and `now` are special keywords.

**Constraints and Limitations**

- `date_format` validates the format, not necessarily that the date is logically valid (e.g., `2026-02-30` may pass depending on PHP version).
- Timezone considerations apply to date comparisons.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Date Range**

**Code**

```php
$request->validate([
    'start_date' => 'required|date|after_or_equal:today',
    'end_date' => 'required|date|after:start_date',
]);
```

**Expected Output**

- `start_date = "2026-11-01"`, `end_date = "2026-11-10"` → passes.
- `end_date` before `start_date` → fails.

**Why This Output Occurs**

`after_or_equal:today` ensures the start date is not in the past; `after:start_date` ensures the end date is after the start date.

---

**Example 2: Custom Date Format**

**Code**

```php
$request->validate([
    'appointment' => 'date_format:Y-m-d H:i',
]);
```

**Expected Output**

- `appointment = "2026-11-01 14:30"` → passes.
- `appointment = "01/11/2026 2:30 PM"` → fails.

**Why This Output Occurs**

`date_format` requires the exact format specified.

---

#### Real-World Cases with Explanation

**Case 1: Hotel Booking**

A hotel booking system validates that the check-out date is after the check-in date using `after:check_in_date`.

**Explanation**: Date comparison rules prevent logical errors in date ranges.

---

### 8. Comparisons & Relationships

#### Definitions

**Core Definition**

Comparison and relationship rules validate that a field matches, differs from, or is included/excluded from a set of values or another field.

**Technical Definition**

`confirmed` requires a matching `{field}_confirmation` field. `same` and `different` compare against another field. `in` and `not_in` check membership in a list. `in_array` checks membership in another array field. `distinct` ensures no duplicate values in an array.

**Beginner-Friendly Explanation**

These rules check relationships between fields—like "password confirmation must match password" or "the selected color must be one of the available options."

#### Purposes

- To enforce password confirmation matching.
- To ensure two fields are different (e.g., new password differs from old).
- To validate that a value is one of a predefined set.
- To prevent duplicate values in arrays.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
'password' => 'required|confirmed',
'new_password' => 'different:old_password',
'color' => 'in:red,green,blue',
'role' => 'not_in:admin,superadmin',
'tags' => 'array|distinct',
'selected_ids' => 'array|in_array:available_ids',
```

**Component Breakdown**

| Rule | Behavior |
|------|----------|
| `confirmed` | Requires `{field}_confirmation` to match. |
| `different:field` | Field must differ from another field. |
| `distinct` | Array must not contain duplicates. |
| `in:foo,bar` | Value must be in the list. |
| `in_array:field` | Value must be in another array field. |
| `not_in:foo,bar` | Value must not be in the list. |
| `same:field` | Field must match another field. |

**Syntax Rules**

- `confirmed` looks for `{field}_confirmation` automatically.
- `in` and `not_in` accept comma-separated values.
- `in_array` references another field containing an array.

**Constraints and Limitations**

- `confirmed` requires the confirmation field to be present in the input.
- `distinct` uses loose comparison by default; add `strict` for strict comparison.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Confirmed Password**

**Code**

```php
$request->validate([
    'password' => 'required|confirmed|min:8',
]);
```

**Expected Output**

- `password = "secret123"`, `password_confirmation = "secret123"` → passes.
- Mismatched confirmation → fails.

**Why This Output Occurs**

`confirmed` checks for a matching `password_confirmation` field.

---

**Example 2: In and Distinct**

**Code**

```php
$request->validate([
    'color' => 'in:red,green,blue',
    'tags' => 'array|distinct',
]);
```

**Expected Output**

- `color = "red"` → passes; `"yellow"` → fails.
- `tags = ["a","b"]` → passes; `["a","a"]` → fails.

**Why This Output Occurs**

`in` restricts to the listed values; `distinct` prevents duplicate array elements.

---

#### Real-World Cases with Explanation

**Case 1: Password Change Form**

A password change form uses `different:current_password` to ensure the new password is not the same as the old one.

**Explanation**: Comparison rules enforce security-related field relationships.

---

### 9. Database & Authentication

#### Definitions

**Core Definition**

Database and authentication rules validate data against records in the database or the currently authenticated user's credentials.

**Technical Definition**

`exists:table,column` checks that a value exists in a database table. `unique:table,column,except,idColumn` checks that a value does not exist (with optional exclusion for updates). `current_password` verifies that the value matches the authenticated user's password (optionally using a specific guard). `password` (the `Password` rule object) enforces password complexity and optionally checks against breached password databases.

**Beginner-Friendly Explanation**

These rules check things like "does this email already exist in our database?" or "is the password you entered actually your current password?"

#### Purposes

- To ensure foreign keys reference existing records.
- To enforce uniqueness of emails, usernames, and other identifiers.
- To verify the current password before sensitive changes.
- To enforce strong password policies.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
'email' => 'required|email|unique:users,email',
'category_id' => 'required|exists:categories,id',
'current_password' => 'required|current_password',
'password' => ['required', 'confirmed', Password::min(8)->mixedCase()->numbers()],
```

**Component Breakdown**

| Rule | Behavior |
|------|----------|
| `exists:table,column` | Value must exist in the table/column. |
| `unique:table,column,except,idColumn` | Value must be unique (optionally excluding a record). |
| `current_password:guard` | Must match the authenticated user's password. |
| `password` | Enforces password complexity (via `Password` rule object). |

**Syntax Rules**

- `unique` can exclude a record for updates: `unique:users,email,{$userId}`.
- `exists` and `unique` can specify a database connection.
- `Password::min()` provides a fluent API for password rules.

**Constraints and Limitations**

- Database rules execute queries and can impact performance; index the relevant columns.
- `current_password` requires an authenticated user.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Unique Email on Update**

**Code**

```php
$userId = $request->user()->id;
$request->validate([
    'email' => "required|email|unique:users,email,{$userId}",
]);
```

**Expected Output**

- Email already used by another user → fails.
- Email unchanged for the current user → passes (excluded by ID).

**Why This Output Occurs**

The third parameter to `unique` excludes the current user's record from the uniqueness check.

---

**Example 2: Current Password and Password Complexity**

**Code**

```php
$request->validate([
    'current_password' => 'required|current_password',
    'password' => ['required', 'confirmed', Password::min(12)->mixedCase()->numbers()->symbols()],
]);
```

**Expected Output**

- Wrong current password → fails.
- New password not meeting complexity → fails.

**Why This Output Occurs**

`current_password` verifies the existing password; the `Password` rule enforces complexity.

---

#### Real-World Cases with Explanation

**Case 1: User Profile Update**

A profile update form uses `unique:users,email,{$userId}` to allow users to keep their existing email while preventing duplicates.

**Explanation**: The exclusion parameter is essential for update forms.

---

### 10. Files & Images

#### Definitions

**Core Definition**

File and image rules validate uploaded files, including their type, size, MIME type, and dimensions.

**Technical Definition**

`file` ensures the value is an uploaded file. `image` ensures it is an image (jpeg, png, bmp, gif, svg, webp). `mimes` validates by file extension (guessing MIME type from contents). `mimetypes` validates the actual MIME type. `dimensions` checks image width, height, and aspect ratio using parameters like `min_width`, `max_width`, `min_height`, `max_height`, and `ratio`.

**Beginner-Friendly Explanation**

These rules check that a user uploaded a real file, that it's an image of the right type, that it's not too large, and that its dimensions meet your requirements.

#### Purposes

- To restrict uploads to specific file types.
- To enforce maximum file sizes.
- To validate image dimensions for consistent layouts.
- To ensure the uploaded file is a genuine image, not a disguised executable.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
'avatar' => 'required|file|image|mimes:jpeg,png,webp|max:2048|dimensions:min_width=100,min_height=100',
'document' => 'required|file|mimetypes:application/pdf|max:5120',
```

**Component Breakdown**

| Rule | Behavior |
|------|----------|
| `file` | Value must be an uploaded file. |
| `image` | File must be an image (JPEG, PNG, BMP, GIF, SVG, WebP). |
| `mimes:ext1,ext2` | File extension must match one of the listed types (determined by content). |
| `mimetypes:type1,type2` | Actual MIME type must match. |
| `dimensions:params` | Image must meet dimension constraints (`min_width`, `max_width`, `ratio`, etc.). |

**Syntax Rules**

- `mimes` guesses the MIME type from the file contents, not the extension.
- `dimensions` requires the file to be an image.
- `max` for files is in kilobytes.

**Constraints and Limitations**

- `image` may not detect all image formats depending on PHP's GD extension.
- `dimensions` can be slow for large images.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Avatar Upload**

**Code**

```php
$request->validate([
    'avatar' => 'required|file|image|mimes:jpeg,png,webp|max:2048|dimensions:min_width=100,min_height=100',
]);
```

**Expected Output**

- A 500KB JPEG of 200x200 → passes.
- A 3MB PNG → fails (max 2048KB).
- A 50x50 image → fails (min dimensions).

**Why This Output Occurs**

The rules validate file type, size, and dimensions.

---

**Example 2: PDF Document**

**Code**

```php
$request->validate([
    'document' => 'required|file|mimetypes:application/pdf|max:5120',
]);
```

**Expected Output**

- A genuine PDF under 5MB → passes.
- A `.pdf` file that is actually an executable → fails.

**Why This Output Occurs**

`mimetypes` checks the actual MIME type, not just the extension.

---

#### Real-World Cases with Explanation

**Case 1: Profile Picture Upload**

A social platform validates avatars with `image`, `mimes:jpeg,png`, `max:2048`, and `dimensions:min_width=100,min_height=100`.

**Explanation**: File rules ensure uploads are safe, appropriately sized, and visually consistent.

---

### 11. Execution Controls

#### Definitions

**Core Definition**

Execution control rules modify how the validator processes subsequent rules, most notably `bail`, which stops validation for a field after the first failure.

**Technical Definition**

`bail` instructs the validator to cease evaluating further rules for the current field once a rule fails. This is implemented by the `validateBail()` method, which breaks the rule loop. Without `bail`, all rules are evaluated and all applicable error messages are collected.

**Beginner-Friendly Explanation**

`bail` means "stop at the first problem." If a field fails one rule, don't bother checking the rest for that field—just report the first error.

#### Purposes

- To avoid unnecessary validation work after a known failure.
- To present only the most relevant error message per field.
- To improve performance when rules are expensive (e.g., database checks).

#### Syntax Rules and Structure

**Complete General Syntax**

```php
'email' => 'bail|required|email|unique:users,email',
```

**Component Breakdown**

| Rule | Behavior |
|------|----------|
| `bail` | Stops validating the field after the first failure. |

**Syntax Rules**

- `bail` should be the first rule in the list.
- It applies only to the field it is assigned to.
- `stopOnFirstFailure()` on the validator stops on the first failure across all fields.

**Constraints and Limitations**

- `bail` may hide additional issues that the user would need to fix in subsequent submissions.
- Use judiciously; sometimes showing all errors is more helpful.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Bail with Unique Email**

**Code**

```php
$request->validate([
    'email' => 'bail|required|email|unique:users,email',
]);
```

**Expected Output**

- Empty email → fails `required` and stops; `email` and `unique` are not evaluated.
- Invalid email → fails `email` and stops; `unique` is not evaluated.
- Valid but duplicate email → fails `unique`.

**Why This Output Occurs**

`bail` short-circuits the rule chain on the first failure.

---

#### Real-World Cases with Explanation

**Case 1: Registration Form with Unique Email**

A registration form uses `bail` on the email field to avoid running a database `unique` check when the email is already syntactically invalid.

**Explanation**: `bail` improves performance by skipping expensive checks after a basic validation failure.

---

### 12. Custom & Advanced Architecture

#### Definitions

**Core Definition**

Custom and advanced validation architecture encompasses user-defined rules, rule objects, conditional rule builders, rule groups, and architectural best practices for organizing validation logic.

**Technical Definition**

Custom rules can be implemented as closures (for one-off logic) or as classes implementing `Illuminate\Contracts\Validation\ValidationRule` (for reusable logic). The `Rule` facade provides fluent builders for many rules (e.g., `Rule::unique()`, `Rule::requiredIf()`). Form Requests encapsulate validation rules in dedicated classes. Rule groups and rule sets (often via packages) organize reusable combinations of rules.

**Beginner-Friendly Explanation**

Custom rules let you define your own validation logic when the built-in rules aren't enough—like checking that a username isn't a reserved word, or that a promo code is currently active. Advanced architecture is about organizing these rules cleanly.

#### Purposes

- To implement application-specific validation logic.
- To reuse complex validation rules across multiple forms.
- To keep controllers clean by moving validation to Form Requests.
- To organize rule sets for DRY (Don't Repeat Yourself) validation.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Closure rule
'username' => [
    'required',
    function (string $attribute, mixed $value, Closure $fail) {
        if (in_array($value, ['admin', 'root'])) {
            $fail("The {$attribute} is reserved.");
        }
    },
],

// Rule object
use App\Rules\ValidPromoCode;
'promo_code' => ['required', new ValidPromoCode],

// Form Request
public function rules(): array
{
    return [
        'email' => ['required', 'email'],
        'password' => ['required', 'confirmed', Password::min(8)],
    ];
}
```

**Component Breakdown**

| Approach | Description |
|----------|-------------|
| Closure | Inline custom rule for one-off logic. |
| Rule object | Class implementing `ValidationRule`; reusable. |
| `Rule` facade | Fluent builders for common rules. |
| Form Request | Dedicated class encapsulating rules and authorization. |
| Rule groups | Reusable arrays of rules (via packages or custom classes). |

**Syntax Rules**

- Closure rules receive `$attribute`, `$value`, and `$fail` callback.
- Rule objects implement `validate(string $attribute, mixed $value, Closure $fail)`.
- Form Requests are generated with `php artisan make:request`.
- `Rule::requiredIf()` accepts a boolean or closure.

**Constraints and Limitations**

- Custom rules should return early and call `$fail()` with a clear message on failure.
- Form Requests add a layer of indirection but improve organization.
- Rule groups are not a native Laravel feature; use packages or custom classes.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Closure Rule for Reserved Usernames**

**Code**

```php
$request->validate([
    'username' => [
        'required',
        'string',
        'max:20',
        function (string $attribute, mixed $value, Closure $fail) {
            if (in_array(strtolower($value), ['admin', 'root', 'system'])) {
                $fail("The {$attribute} is reserved.");
            }
        },
    ],
]);
```

**Expected Output**

- `username = "john"` → passes.
- `username = "admin"` → fails with "The username is reserved."

**Why This Output Occurs**

The closure checks the value against a reserved list and calls `$fail()` if matched.

---

**Example 2: Rule Object for Promo Code**

**Rule Class**

```php
<?php
namespace App\Rules;

use Closure;
use Illuminate\Contracts\Validation\ValidationRule;

class ValidPromoCode implements ValidationRule
{
    public function validate(string $attribute, mixed $value, Closure $fail): void
    {
        if (! PromoCode::where('code', $value)->where('expires_at', '>', now())->exists()) {
            $fail('The promo code is invalid or expired.');
        }
    }
}
```

**Usage**

```php
$request->validate([
    'promo_code' => ['required', new ValidPromoCode],
]);
```

**Expected Output**

- Valid, unexpired code → passes.
- Invalid or expired code → fails.

**Why This Output Occurs**

The rule object queries the database and calls `$fail()` if no valid code exists.

---

**Example 3: Form Request**

**Command**

```bash
php artisan make:request StorePostRequest
```

**Form Request**

```php
// app/Http/Requests/StorePostRequest.php
public function rules(): array
{
    return [
        'title' => ['required', 'string', 'max:255'],
        'body' => ['required', 'string'],
        'published_at' => ['nullable', 'date', 'after_or_equal:today'],
        'tags' => ['array', 'distinct'],
        'tags.*' => ['string', 'max:50'],
    ];
}
```

**Usage**

```php
public function store(StorePostRequest $request)
{
    Post::create($request->validated());
}
```

**Expected Output**

Validation runs automatically before the controller method, returning errors and old input on failure.

**Why This Output Occurs**

Form Requests encapsulate rules and authorization, keeping controllers clean.

---

**Example 4: Conditional Rule Builder**

**Code**

```php
use Illuminate\Validation\Rule;

$request->validate([
    'role_id' => Rule::requiredIf($request->user()->is_admin),
]);
```

**Expected Output**

- Admin user → `role_id` is required.
- Non-admin → `role_id` is optional.

**Why This Output Occurs**

`Rule::requiredIf()` accepts a boolean or closure to dynamically determine requirement.

---

#### Real-World Cases with Explanation

**Case 1: E-commerce Promo Code Validation**

An e-commerce checkout uses a custom `ValidPromoCode` rule object to check that a promo code exists in the database and has not expired.

**Explanation**: Rule objects encapsulate reusable, application-specific validation logic.

**Case 2: Multi-Step Form Validation**

A multi-step wizard uses different Form Requests for each step, keeping validation rules organized and focused.

**Explanation**: Form Requests provide a clean architecture for complex, multi-step forms.

---

## Validation Best Practices

1. **Use Form Requests for Complex Forms**: Encapsulate rules and authorization in dedicated classes.

2. **Prefer Array Syntax for Complex Rules**: Use arrays instead of pipe-delimited strings when rules contain parameters with commas or `|`.

3. **Use `bail` Judiciously**: Stop validation on first failure for expensive rules, but consider whether showing all errors is more helpful to users.

4. **Combine `nullable` with Other Rules**: Allow optional fields to be null without triggering downstream rules.

5. **Use `required_array_keys` for API Payloads**: Ensure required keys exist in array inputs.

6. **Customize Error Messages**: Override messages in `lang/en/validation.php` or via the `messages()` method in Form Requests.

7. **Validate at the Boundary**: Validate all incoming data at the controller or Form Request level; never trust user input.

8. **Use `Rule` Objects for Reusable Logic**: Avoid duplicating complex validation across forms.

9. **Test Validation Rules**: Write tests for validation logic, especially custom rules.

10. **Be Mindful of Database Rules**: `exists`, `unique`, and `current_password` execute queries; index relevant columns and use `bail` to avoid unnecessary checks.

---

## References

- Laravel Validation Documentation (Laravel 12.x) — https://laravel.com/docs/12.x/validation
- Laravel Validation Documentation (Laravel 11.x) — https://laravel.com/docs/11.x/validation
- Laravel Validation Documentation (Laravel 10.x) — https://laravel.com/framework/docs/10.x/validation
- Laravel Validation Documentation (Laravel 13.x) — https://laravel.com/framework/docs/13.x/validation
- Laravel Custom Validation Rules – Laravel News — https://laravel-news.com/custom-validation-rules
- Laravel `required_array_keys` Rule – Laravel Daily — https://laraveldaily.com/post/new-array-validation-rule-required-array-keys
- Laravel `bail` Rule – Laravel 12.x Documentation — https://laravel.com/docs/12.x/validation#rule-bail
- Laravel `active_url` Rule – Laravel 13.x Documentation — https://laravel.com/framework/docs/13.x/validation#rule-active-url
- Laravel `enum` Rule – Laravel 8.x Documentation — https://laravel.com/docs/8.x/validation#rule-enum
- Laravel `dimensions` Rule – Laravel News — https://laravel-news.com/image-dimension-validation-with-laravels-dimensions-rule
- Laravel `Password` Rule Object – Laravel 12.x Documentation — https://laravel.com/docs/12.x/validation#rule-password
- Laravel Validation Patterns – GitHub (voku/agent-skills) — https://github.com/voku/agent-skills/blob/main/skills/laravel-best-practices/AGENTS.md

---

**Version Note**: This cheat sheet is based on Laravel 10.x through 13.x documentation. Validation rules have remained largely stable across these versions. `required_array_keys` was added in Laravel 8.82. The `Password` rule object was introduced in Laravel 8. The `Enum` rule requires PHP 8.1+. `active_url` performs DNS lookups and should be used cautiously in performance-sensitive contexts. Developers should always consult the documentation for their specific Laravel version.