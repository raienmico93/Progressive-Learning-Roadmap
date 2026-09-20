# PHP Input Defense: Sanitization, Validation & Cross-Site Scripting (XSS) — Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**  
PHP Input Defense is the set of programming practices and PHP functions used to protect a web application from malicious or malformed user input. It encompasses validation (accepting or rejecting input based on strict rules), sanitization (cleaning input before internal use), output encoding (making data safe for display), and CSRF protection (ensuring state-changing requests originate from the legitimate user).

**Technical Definition**  
PHP Input Defense refers to the systematic application of the Filter extension (`filter_var()`, `filter_input()`), string encoding functions (`htmlspecialchars()`, `htmlentities()`), and session-based cryptographic token mechanisms (`random_bytes()`, `hash_equals()`) to mitigate injection vulnerabilities including Cross-Site Scripting (XSS), SQL Injection (SQLi), and Cross-Site Request Forgery (CSRF). It operates on the principle that all external input — from `$_GET`, `$_POST`, `$_COOKIE`, `$_FILES`, `$_SERVER`, and `php://input` — is untrusted until proven otherwise, and that defense must occur at the correct layer: validation at input, encoding at output, and token verification for state changes.

**Beginner-Friendly Explanation**  
When users interact with your website, they can type anything into forms or modify URLs. Some of that input may be accidental mistakes; some may be deliberate attacks. PHP Input Defense is your security checklist: check that input looks right before using it (validation), clean up messy input (sanitization), and always convert special characters before displaying anything back to the browser (output encoding). Additionally, for actions like changing a password or transferring money, you add a secret token to your forms so that only your own website can trigger those actions (CSRF protection).

---

### Key Characteristics

- **Validation** rejects or accepts input based on strict structural rules; it does not modify the data.
- **Sanitization** strips or cleans unwanted characters before internal processing.
- **Output Encoding** converts characters with special meaning in HTML into entities, nullifying XSS payloads.
- **CSRF Protection** uses cryptographically secure, unpredictable tokens tied to the user's session to verify that state-changing requests are legitimate.
- All external input (`$_GET`, `$_POST`, `$_COOKIE`, `$_FILES`, `$_SERVER`, `php://input`) is untrusted by default.
- Defense must be **layered**: validation at the point of input, encoding at the point of output, and token verification for state changes.
- Client-side validation is a UX convenience only; it can be bypassed and must never replace server-side checks.
- Deprecated or unsafe functions include `FILTER_SANITIZE_STRING` (deprecated in PHP 8.1), `mysql_real_escape_string()` (removed in PHP 7), `addslashes()` for SQL escaping (insufficient), and `htmlentities()` when `htmlspecialchars()` is sufficient.
- Version-specific behavior: `FILTER_VALIDATE_EMAIL` does not fully implement RFC 5321; `htmlspecialchars()` default flags changed over PHP versions; `session_start()` options throw exceptions in PHP 8.5.0 for invalid types.

---

### Prerequisites

- Basic PHP syntax: variables, arrays, functions, superglobals.
- Basic HTTP: requests, responses, headers, methods.
- HTML forms: `method`, `action`, `enctype`, `name` attributes.
- A web server with PHP installed (Apache, Nginx + PHP-FPM, or PHP built-in server).
- Basic understanding of databases (for context on SQL injection) and sessions.

---

### Related Programming Areas

- Web application security (OWASP Top 10: A03 Injection, A07 Identification and Authentication Failures).
- Session management and authentication.
- Database interaction and prepared statements.
- Content Security Policy (CSP) as defense-in-depth against XSS.
- Framework request validation (Laravel, Symfony).
- Static analysis and security auditing tools.

---

## Core Concepts / Features

## 1. Validation — Rejecting or Accepting Input Based on Strict Structures

### Definitions

**Core Definition**  
Validation is the process of determining whether a given input value conforms to a required format, type, range, or business rule. It returns a boolean-like result: the value is accepted or rejected. It does not alter the original data.

**Technical Definition**  
In PHP, validation is implemented through the Filter extension's `FILTER_VALIDATE_*` filters, which inspect values against structural criteria (e.g., `FILTER_VALIDATE_EMAIL`, `FILTER_VALIDATE_INT`, `FILTER_VALIDATE_URL`, `FILTER_VALIDATE_FLOAT`, `FILTER_VALIDATE_BOOLEAN`, `FILTER_VALIDATE_IP`, `FILTER_VALIDATE_REGEXP`). `filter_var()` applies a validation filter to a variable and returns the validated value or `false` on failure. `filter_input()` reads a raw superglobal and applies a validation filter in one step, returning `null` if the variable is not set and `false` if validation fails. Validation filters do not modify the value; they either accept it as-is or reject it.

**Beginner-Friendly Explanation**  
Validation is like a bouncer at a club: it checks your ID and either lets you in or turns you away. It does not change your ID — it just decides whether you meet the rules. For example, if a form asks for an email address, validation checks whether the input actually looks like an email (`user@example.com`) rather than "hello" or `<script>`. If the check fails, you show an error and ask the user to try again.

### Purposes

- To reject malformed or malicious input before it reaches application logic.
- To enforce type, size, format, and range constraints on user-supplied data.
- To prevent SQL injection, XSS, command injection, and other injection attacks.
- To ensure data integrity before storage or processing.
- To provide meaningful error messages to legitimate users.
- To comply with business rules (e.g., age must be between 18 and 120).
- To avoid relying on client-side JavaScript validation, which can be bypassed.

### Syntax Rules and Structure

**General Syntax: `filter_var()`**
```php
$result = filter_var(
    $value,                           // the value to validate
    FILTER_VALIDATE_EMAIL,            // validation filter constant
    ['options' => ['default' => '']]  // optional options array
);
```

**Component Breakdown**

- `$value` — the scalar value to validate; non-scalars are converted to strings internally.
- `FILTER_VALIDATE_EMAIL` — the validation filter to apply.
- Options array — optional associative array with `default`, `min_range`, `max_range`, `flags`, etc.
- Return value: the validated value on success, `false` on failure (or `null` if `FILTER_NULL_ON_FAILURE` is used).

**General Syntax: `filter_input()`**
```php
$email = filter_input(
    INPUT_POST,                       // INPUT_GET, INPUT_POST, INPUT_COOKIE, INPUT_SERVER, INPUT_ENV
    'email',                          // variable name
    FILTER_VALIDATE_EMAIL,            // validation filter
    ['options' => ['default' => '']]  // optional options
);
```

**Component Breakdown**

- `INPUT_POST` — source superglobal constant.
- `'email'` — the field name.
- `FILTER_VALIDATE_EMAIL` — validation filter.
- Options array — optional.
- Return value: validated value on success, `false` on filter failure, `null` if the variable is not set.

**General Syntax: Integer with Range**
```php
$age = filter_var($input, FILTER_VALIDATE_INT, [
    'options' => [
        'default'   => 0,
        'min_range' => 1,
        'max_range' => 120
    ]
]);
```

**Component Breakdown**

- `FILTER_VALIDATE_INT` — validates that the value is an integer.
- `min_range` — minimum acceptable value.
- `max_range` — maximum acceptable value.
- `default` — value returned if validation fails (unless `FILTER_NULL_ON_FAILURE` is set).

**General Syntax: URL Validation with Path Requirement**
```php
$url = filter_var($input, FILTER_VALIDATE_URL, FILTER_FLAG_PATH_REQUIRED);
```

**Component Breakdown**

- `FILTER_VALIDATE_URL` — validates URL per RFC 2396.
- `FILTER_FLAG_PATH_REQUIRED` — requires a path component in the URL.
- Other URL flags: `FILTER_FLAG_QUERY_REQUIRED`.

**General Syntax: Regular Expression Validation**
```php
$zip = filter_var($input, FILTER_VALIDATE_REGEXP, [
    'options' => ['regexp' => '/^\d{5}(-\d{4})?$/']
]);
```

**Component Breakdown**

- `FILTER_VALIDATE_REGEXP` — validates against a PCRE pattern.
- `regexp` — the Perl-compatible regular expression (must include delimiters).

**Common Validation Filters**

| Filter | Purpose | Notes |
|---|---|---|
| `FILTER_VALIDATE_BOOLEAN` | Validates "1", "true", "on", "yes" as true | `FILTER_NULL_ON_FAILURE` returns null for unrecognized values |
| `FILTER_VALIDATE_EMAIL` | Validates email address | Does not fully implement RFC 5321 |
| `FILTER_VALIDATE_FLOAT` | Validates floating-point number | Supports `FILTER_FLAG_ALLOW_THOUSAND` |
| `FILTER_VALIDATE_INT` | Validates integer | Supports `min_range`, `max_range`, `FILTER_FLAG_ALLOW_OCTAL`, `FILTER_FLAG_ALLOW_HEX` |
| `FILTER_VALIDATE_IP` | Validates IP address | `FILTER_FLAG_IPV4`, `FILTER_FLAG_IPV6`, `FILTER_FLAG_NO_PRIV_RANGE`, `FILTER_FLAG_NO_RES_RANGE` |
| `FILTER_VALIDATE_REGEXP` | Validates against PCRE pattern | Requires `regexp` option |
| `FILTER_VALIDATE_URL` | Validates URL | `FILTER_FLAG_PATH_REQUIRED`, `FILTER_FLAG_QUERY_REQUIRED` |
| `FILTER_VALIDATE_DOMAIN` | Validates domain name | PHP 7.0+; `FILTER_FLAG_HOSTNAME` |

**Syntax Rules**

- Validation filters never modify the value; they accept or reject.
- `filter_var()` returns `false` on failure; check with `=== false`, not `empty()`.
- `filter_input()` returns `null` if the variable is not set, `false` if validation fails.
- Use `FILTER_NULL_ON_FAILURE` to get `null` on failure instead of `false`.
- `min_range` and `max_range` are inclusive.
- The `default` option applies when the filter fails (unless `FILTER_NULL_ON_FAILURE` is set).
- For `FILTER_VALIDATE_INT`, leading zeros may be rejected unless `FILTER_FLAG_ALLOW_OCTAL` is set.
- `FILTER_VALIDATE_EMAIL` rejects some valid RFC 5321 addresses.
- Always validate on the server side; client-side validation is not sufficient.

**Constraints and Limitations**

- Validation alone does not make data safe for all contexts (e.g., SQL, HTML, shell).
- `FILTER_VALIDATE_EMAIL` is not RFC 5321-compliant; it may reject valid addresses.
- `FILTER_VALIDATE_URL` does not guarantee the URL is safe or reachable.
- `FILTER_VALIDATE_REGEXP` requires a valid PCRE pattern with delimiters.
- `filter_var()` converts non-scalar values to strings; arrays must be validated element-wise.
- `filter_input()` reads the raw superglobal, not modifications made after startup.
- Validation filters are not available for all data types; custom validation may be needed.
- Overly strict validation may reject legitimate international input (e.g., Unicode names, internationalized domain names).

### Annotated Code Examples and Expected Outputs

#### Example 1: Validating Email, Integer, and URL

**File: `validate.php`**
```php
<?php
// Validate an email address.
$email = filter_var('alice@example.com', FILTER_VALIDATE_EMAIL);
var_dump($email); // string(17) "alice@example.com"

// Validate an invalid email.
$badEmail = filter_var('not-an-email', FILTER_VALIDATE_EMAIL);
var_dump($badEmail); // bool(false)

// Validate an integer with range.
$age = filter_var('25', FILTER_VALIDATE_INT, [
    'options' => ['min_range' => 18, 'max_range' => 120, 'default' => 0]
]);
var_dump($age); // int(25)

// Validate an out-of-range integer.
$tooOld = filter_var('150', FILTER_VALIDATE_INT, [
    'options' => ['min_range' => 18, 'max_range' => 120, 'default' => 0]
]);
var_dump($tooOld); // int(0) — default applied

// Validate a URL.
$url = filter_var('https://example.com/path?q=1', FILTER_VALIDATE_URL);
var_dump($url); // string(31) "https://example.com/path?q=1"

// Validate a URL requiring a path.
$noPath = filter_var('https://example.com', FILTER_VALIDATE_URL, FILTER_FLAG_PATH_REQUIRED);
var_dump($noPath); // bool(false)
?>
```

**Expected Output**
```
string(17) "alice@example.com"
bool(false)
int(25)
int(0)
string(31) "https://example.com/path?q=1"
bool(false)
```

**Why This Output Occurs**

- `FILTER_VALIDATE_EMAIL` accepts `alice@example.com` and rejects `not-an-email`.
- `FILTER_VALIDATE_INT` with `min_range` and `max_range` accepts `25` and applies the default `0` to `150`.
- `FILTER_VALIDATE_URL` accepts the full URL; with `FILTER_FLAG_PATH_REQUIRED`, it rejects the URL without a path.

#### Example 2: Validating Boolean and IP with `filter_input()`

**File: `process.php`**
```php
<?php
// Simulate POST data.
$_POST['subscribe'] = 'yes';
$_POST['ip'] = '192.168.1.1';

// Validate boolean.
$subscribe = filter_input(INPUT_POST, 'subscribe', FILTER_VALIDATE_BOOLEAN);
var_dump($subscribe); // bool(true)

// Validate IP with private range exclusion.
$ip = filter_input(INPUT_POST, 'ip', FILTER_VALIDATE_IP, FILTER_FLAG_NO_PRIV_RANGE);
var_dump($ip); // bool(false) — 192.168.x.x is private

// Validate IP without exclusion.
$ip2 = filter_input(INPUT_POST, 'ip', FILTER_VALIDATE_IP);
var_dump($ip2); // string(11) "192.168.1.1"
?>
```

**Expected Output**
```
bool(true)
bool(false)
string(11) "192.168.1.1"
```

**Why This Output Occurs**

- `FILTER_VALIDATE_BOOLEAN` accepts `'yes'` as `true`.
- `FILTER_FLAG_NO_PRIV_RANGE` rejects private IP ranges like `192.168.x.x`.
- Without the flag, the IP is accepted as a valid IP.

### Real-World Cases

- **Registration forms:** Validate email, password strength, and age before creating an account.
- **Checkout:** Validate credit card numbers (via Luhn algorithm), expiry dates, and CVV.
- **Search:** Validate and whitelist sort/filter parameters to prevent injection.
- **API endpoints:** Validate JSON payload fields against expected types and ranges.
- **File uploads:** Validate file size, extension, and MIME type before accepting.

---

## 2. Sanitization — Cleaning Unwanted Characters Before Internal Processing

### Definitions

**Core Definition**  
Sanitization is the process of removing or altering unwanted characters from input so that the remaining data is safe for internal processing or storage. Unlike validation, sanitization modifies the data.

**Technical Definition**  
In PHP, sanitization is implemented through `FILTER_SANITIZE_*` filters, which strip or encode characters from a value. Common filters include `FILTER_SANITIZE_EMAIL`, `FILTER_SANITIZE_URL`, `FILTER_SANITIZE_NUMBER_INT`, `FILTER_SANITIZE_NUMBER_FLOAT`, `FILTER_SANITIZE_SPECIAL_CHARS`, and `FILTER_SANITIZE_FULL_SPECIAL_CHARS`. Sanitization filters are applied with `filter_var()` or `filter_input()`. The `FILTER_SANITIZE_STRING` filter was deprecated in PHP 8.1 and should not be used. For HTML escaping, `htmlspecialchars()` is preferred over `FILTER_SANITIZE_SPECIAL_CHARS` in most cases.

**Beginner-Friendly Explanation**  
Sanitization is like washing vegetables before cooking — you remove dirt and anything harmful, but the vegetable is still a vegetable. For example, if a user types spaces or dashes into a phone number field, sanitization can strip those out so you are left with digits only. If a user types HTML tags into a comment, sanitization can remove them. Sanitization is about cleaning input before you store or process it internally.

### Purposes

- To remove characters that are not needed for the intended data type.
- To normalize input before storage (e.g., stripping spaces from phone numbers).
- To reduce the attack surface by eliminating dangerous characters.
- To prepare data for specific contexts (e.g., email, URL).
- To complement validation when the input format is flexible.
- To prevent stored XSS by stripping HTML tags from user content.
- To handle legacy systems that cannot use prepared statements.

### Syntax Rules and Structure

**General Syntax: `filter_var()` with Sanitization Filter**
```php
$clean = filter_var(
    $value,                           // input to sanitize
    FILTER_SANITIZE_EMAIL,            // sanitization filter
    FILTER_FLAG_STRIP_LOW             // optional flags
);
```

**Component Breakdown**

- `$value` — the value to sanitize.
- `FILTER_SANITIZE_EMAIL` — removes all characters except letters, digits, and `!#$%&'*+-=?^_`{|}~@.[]`.
- `FILTER_FLAG_STRIP_LOW` — strips characters with ASCII value < 32.
- Return value: sanitized string.

**General Syntax: `filter_input()` with Sanitization Filter**
```php
$email = filter_input(INPUT_POST, 'email', FILTER_SANITIZE_EMAIL);
```

**Component Breakdown**

- `INPUT_POST` — source superglobal.
- `'email'` — field name.
- `FILTER_SANITIZE_EMAIL` — sanitization filter.
- Returns sanitized string or `null` if not set.

**Common Sanitization Filters**

| Filter | Purpose | Notes |
|---|---|---|
| `FILTER_SANITIZE_EMAIL` | Removes all chars except letters, digits, and `!#$%&'*+-=?^_`{|}~@.[]` | Does not validate; use with `FILTER_VALIDATE_EMAIL` |
| `FILTER_SANITIZE_URL` | Removes all chars except letters, digits, and `$-_.+!*'(),{}|\\^~[]`<>#%";/?:@&=` | Does not validate |
| `FILTER_SANITIZE_NUMBER_INT` | Removes all chars except digits, `+`, `-` | Use with `FILTER_VALIDATE_INT` |
| `FILTER_SANITIZE_NUMBER_FLOAT` | Removes all chars except digits, `+`, `-`, and optionally `.`, `,`, `e` | Use `FILTER_FLAG_ALLOW_FRACTION`, `FILTER_FLAG_ALLOW_THOUSAND`, `FILTER_FLAG_ALLOW_SCIENTIFIC` |
| `FILTER_SANITIZE_SPECIAL_CHARS` | HTML-encodes `&`, `"`, `<`, `>`, and optionally `'` | Similar to `htmlspecialchars()` |
| `FILTER_SANITIZE_FULL_SPECIAL_CHARS` | HTML-encodes all special chars, equivalent to `htmlspecialchars()` with `ENT_QUOTES` | Preferred over `FILTER_SANITIZE_SPECIAL_CHARS` |
| `FILTER_UNSAFE_RAW` | Does nothing by default | Can be combined with flags |
| `FILTER_SANITIZE_STRING` | **DEPRECATED in PHP 8.1** | Use `FILTER_SANITIZE_FULL_SPECIAL_CHARS` instead |

**Sanitization Flags**

| Flag | Effect |
|---|---|
| `FILTER_FLAG_STRIP_LOW` | Strips characters with ASCII value < 32 |
| `FILTER_FLAG_STRIP_HIGH` | Strips characters with ASCII value > 127 |
| `FILTER_FLAG_STRIP_BACKTICK` | Strips backtick characters |
| `FILTER_FLAG_ENCODE_LOW` | Encodes characters with ASCII value < 32 |
| `FILTER_FLAG_ENCODE_HIGH` | Encodes characters with ASCII value > 127 |
| `FILTER_FLAG_ENCODE_AMP` | Encodes `&` as `&amp;` |
| `FILTER_FLAG_NO_ENCODE_QUOTES` | Does not encode quotes (for `FILTER_SANITIZE_SPECIAL_CHARS`) |

**Syntax Rules**

- Sanitization modifies the data; validation does not.
- Sanitization should be used before storing data, not as a substitute for output encoding.
- `FILTER_SANITIZE_EMAIL` and `FILTER_SANITIZE_URL` remove dangerous characters but do not validate the result.
- `FILTER_SANITIZE_NUMBER_INT` removes non-numeric characters; it does not verify the result is a valid integer.
- `FILTER_SANITIZE_FULL_SPECIAL_CHARS` is the safest sanitization filter for HTML output.
- `FILTER_SANITIZE_STRING` is deprecated; avoid it.
- Flags modify the behavior of sanitization filters.
- `FILTER_UNSAFE_RAW` with no flags does nothing; it is the default filter.
- Sanitization should not be used as a replacement for prepared statements in SQL.

**Constraints and Limitations**

- Sanitization can remove legitimate characters (e.g., international names, special formatting).
- Sanitization does not guarantee safety in all contexts; output encoding is still required.
- `FILTER_SANITIZE_EMAIL` may remove valid characters from international email addresses.
- `FILTER_SANITIZE_URL` may break valid URLs with unusual characters.
- Sanitization before storage can make data unrecoverable; consider storing raw and encoding at output.
- `FILTER_SANITIZE_STRING` is deprecated and should not be used in new code.
- Over-sanitization can lead to data loss and frustrated users.
- Sanitization alone does not prevent SQL injection; use prepared statements.

### Annotated Code Examples and Expected Outputs

#### Example 1: Sanitizing Email, URL, and Numbers

**File: `sanitize.php`**
```php
<?php
// Sanitize an email.
$email = filter_var(' alice+spam@example.com ', FILTER_SANITIZE_EMAIL);
var_dump($email); // string(23) "alicespam@example.com" — space and + removed

// Sanitize a URL.
$url = filter_var('https://example.com/path with spaces', FILTER_SANITIZE_URL);
var_dump($url); // string(31) "https://example.com/pathwithspaces" — spaces removed

// Sanitize an integer.
$int = filter_var('+1 (555) 123-4567', FILTER_SANITIZE_NUMBER_INT);
var_dump($int); // string(11) "+15551234567" — parentheses, spaces, dashes removed

// Sanitize a float.
$float = filter_var('1,234.56', FILTER_SANITIZE_NUMBER_FLOAT, FILTER_FLAG_ALLOW_FRACTION | FILTER_FLAG_ALLOW_THOUSAND);
var_dump($float); // string(8) "1234.56" — comma removed
?>
```

**Expected Output**
```
string(23) "alicespam@example.com"
string(31) "https://example.com/pathwithspaces"
string(11) "+15551234567"
string(8) "1234.56"
```

**Why This Output Occurs**

- `FILTER_SANITIZE_EMAIL` removes spaces and the `+` character.
- `FILTER_SANITIZE_URL` removes spaces.
- `FILTER_SANITIZE_NUMBER_INT` removes all non-digit characters except `+` and `-`.
- `FILTER_SANITIZE_NUMBER_FLOAT` with `FILTER_FLAG_ALLOW_FRACTION` and `FILTER_FLAG_ALLOW_THOUSAND` keeps the decimal point but removes the comma.

#### Example 2: Sanitizing HTML with `FILTER_SANITIZE_FULL_SPECIAL_CHARS`

**File: `html_sanitize.php`**
```php
<?php
// Dangerous input.
$input = '<script>alert("XSS")</script>Hello & welcome!';

// Sanitize for HTML context.
$safe = filter_var($input, FILTER_SANITIZE_FULL_SPECIAL_CHARS);

echo $safe;
?>
```

**Expected Output (rendered)**
```
<script>alert("XSS")</script>Hello & welcome!
```
(When viewed in a browser, the `<script>` tags are displayed as text, not executed.)

**Why This Output Occurs**

- `FILTER_SANITIZE_FULL_SPECIAL_CHARS` is equivalent to `htmlspecialchars()` with `ENT_QUOTES`.
- It converts `<`, `>`, `&`, `"`, and `'` to HTML entities.
- The browser renders the entities as literal characters, preventing XSS.

### Real-World Cases

- **User profiles:** Sanitize display names and bios before storage.
- **Comments:** Strip HTML tags from user comments if plain text is required.
- **Search queries:** Sanitize search terms to remove control characters.
- **URL parameters:** Sanitize redirect URLs to prevent open redirects.
- **File names:** Sanitize uploaded file names to remove path traversal characters.
- **Phone numbers:** Strip formatting characters to store digits only.

---

## 3. Output Encoding — Preventing XSS with `htmlspecialchars()` and `htmlentities()`

### Definitions

**Core Definition**  
Output encoding is the process of converting characters that have special meaning in a target context (HTML, attribute, URL, JavaScript) into safe representations (HTML entities, percent-encoding, JavaScript escapes) before the data is sent to the browser.

**Technical Definition**  
In PHP, HTML output encoding is primarily performed by `htmlspecialchars()` and `htmlentities()`. `htmlspecialchars()` converts the five characters `&`, `"`, `'`, `<`, and `>` into their HTML entity equivalents (`&amp;`, `&quot;`, `&#039;`, `&lt;`, `&gt;`). `htmlentities()` performs the same conversion but also converts all characters that have named HTML entity equivalents (e.g., `é` to `&eacute;`). The functions accept flags (`ENT_QUOTES`, `ENT_COMPAT`, `ENT_NOQUOTES`, `ENT_SUBSTITUTE`, `ENT_HTML5`) and a character encoding parameter (default `UTF-8` in PHP 5.4+). Output encoding is the primary defense against reflected and stored XSS.

**Beginner-Friendly Explanation**  
Imagine you have a sign that says "Welcome, [name]!" If a user types their name as `<script>stealCookies()</script>`, the browser would try to run that script instead of displaying it. Output encoding fixes this by turning the dangerous characters into safe versions. For example, `<` becomes `&lt;` and `>` becomes `&gt;`. The browser then displays the text literally instead of executing it. `htmlspecialchars()` is the go-to function for this in PHP.

### Purposes

- To prevent reflected and stored XSS by neutralizing HTML special characters.
- To ensure user input is displayed as literal text, not interpreted as code.
- To protect against attribute injection when outputting into HTML attributes.
- To safely embed user data in HTML, XML, and XHTML documents.
- To comply with secure coding standards (OWASP, CWE-79).
- To provide defense-in-depth even when input validation is imperfect.
- To handle international characters correctly with proper encoding.

### Syntax Rules and Structure

**General Syntax: `htmlspecialchars()`**
```php
$safe = htmlspecialchars(
    $string,                                    // input string
    ENT_QUOTES | ENT_SUBSTITUTE | ENT_HTML5,    // flags
    'UTF-8'                                     // character encoding
);
```

**Component Breakdown**

- `$string` — the string to encode.
- `ENT_QUOTES` — encodes both double and single quotes.
- `ENT_SUBSTITUTE` — replaces invalid code unit sequences with U+FFFD.
- `ENT_HTML5` — uses HTML5 entity tables.
- `'UTF-8'` — character encoding; must match the document encoding.
- Return value: encoded string.

**General Syntax: `htmlentities()`**
```php
$safe = htmlentities(
    $string,
    ENT_QUOTES | ENT_SUBSTITUTE,
    'UTF-8'
);
```

**Component Breakdown**

- Identical to `htmlspecialchars()` except it converts all characters that have named HTML entity equivalents.
- Use `htmlentities()` only when you specifically need named entities for all characters.

**Common Flags**

| Flag | Effect |
|---|---|
| `ENT_COMPAT` | Converts double quotes, leaves single quotes (default) |
| `ENT_QUOTES` | Converts both double and single quotes |
| `ENT_NOQUOTES` | Converts neither double nor single quotes |
| `ENT_IGNORE` | Silently discards invalid code unit sequences (discouraged) |
| `ENT_SUBSTITUTE` | Replaces invalid code unit sequences with U+FFFD |
| `ENT_HTML401` | Uses HTML 4.01 entity tables (default) |
| `ENT_HTML5` | Uses HTML5 entity tables |
| `ENT_XML1` | Uses XML 1 entity tables |
| `ENT_XHTML` | Uses XHTML entity tables |

**Syntax Rules**

- `htmlspecialchars()` is sufficient for HTML content and attributes in almost all cases.
- `htmlentities()` is rarely needed; use it only when you need named entities for all characters.
- Always specify the character encoding explicitly (`'UTF-8'`) to avoid mismatches.
- Use `ENT_QUOTES` when outputting into HTML attributes to prevent attribute injection.
- Use `ENT_SUBSTITUTE` to handle invalid encoding gracefully instead of returning an empty string.
- Output encoding must be applied at the point of output, not at the point of input.
- Do not double-encode; encode once, at output.
- `htmlspecialchars()` does not protect against JavaScript injection in `<script>` blocks or CSS injection in `<style>` blocks; use context-specific escaping.
- The default flags changed across PHP versions; always specify flags explicitly.

**Contexts and Corresponding Encoding**

| Context | Function/Method |
|---|---|
| HTML body | `htmlspecialchars($s, ENT_QUOTES, 'UTF-8')` |
| HTML attribute | `htmlspecialchars($s, ENT_QUOTES, 'UTF-8')` |
| URL parameter | `rawurlencode($s)` |
| JavaScript string | `json_encode($s, JSON_HEX_TAG | JSON_HEX_AMP | JSON_HEX_APOS | JSON_HEX_QUOT)` |
| CSS value | Whitelist or `preg_replace` with strict pattern |
| SQL | Prepared statements (never string escaping) |

**Constraints and Limitations**

- `htmlspecialchars()` does not encode all characters; it only encodes `&`, `"`, `'`, `<`, `>`.
- It does not protect against XSS in JavaScript contexts (`<script>` blocks, event handlers).
- It does not protect against XSS in CSS contexts (`<style>` blocks, `style` attributes).
- It does not protect against XSS in URL contexts (`href`, `src` with `javascript:` URLs).
- `htmlentities()` may encode too much, making output larger and harder to read.
- Invalid encoding sequences may be silently discarded unless `ENT_SUBSTITUTE` is used.
- The function is not a substitute for validating and sanitizing input.
- Double-encoding causes display artifacts (e.g., `&amp;` displayed instead of `&`).
- `htmlspecialchars()` does not protect against SQL injection or command injection.

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic XSS Prevention with `htmlspecialchars()`

**File: `xss_prevent.php`**
```php
<?php
// Simulate user input containing a script tag.
$userInput = '<script>alert("XSS")</script>Hello & welcome!';

// Encode for HTML output.
$safe = htmlspecialchars($userInput, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8');

echo 'Encoded: ' . $safe . '<br>';
echo 'Decoded (browser renders): ' . $safe;
?>
```

**Expected Output (rendered in browser)**
```
Encoded: &lt;script&gt;alert(&quot;XSS&quot;)&lt;/script&gt;Hello &amp; welcome!
Decoded (browser renders): <script>alert("XSS")</script>Hello & welcome!
```
(The second line displays the text literally, without executing the script.)

**Why This Output Occurs**

- `htmlspecialchars()` converts `<` to `&lt;`, `>` to `&gt;`, `"` to `&quot;`, and `&` to `&amp;`.
- The browser interprets these entities as literal characters, not HTML tags.
- No script executes.

#### Example 2: Attribute Context Encoding

**File: `attribute.php`**
```php
<?php
// Dangerous input designed to break out of an attribute.
$value = '" onmouseover="alert(1)" data-x="';

// Encode with ENT_QUOTES.
$safe = htmlspecialchars($value, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8');

echo '<input type="text" value="' . $safe . '">';
?>
```

**Expected Output (HTML source)**
```html
<input type="text" value="&quot; onmouseover=&quot;alert(1)&quot; data-x=&quot;">
```

**Why This Output Occurs**

- `ENT_QUOTES` encodes both double and single quotes.
- The `"` characters are converted to `&quot;`, preventing the attribute from being closed.
- The `onmouseover` event handler is treated as literal text, not executable code.

### Real-World Cases

- **Comment systems:** Encode all user-submitted comments before display.
- **Profile pages:** Encode display names and bios.
- **Search results:** Encode the search query when echoing it back.
- **Error messages:** Encode any user input included in error messages.
- **Admin panels:** Encode data from the database before display.
- **Email templates:** Encode user data in HTML emails.

---

## 4. CSRF Defenses — Cryptographic Anti-CSRF Tokens

### Definitions

**Core Definition**  
Cross-Site Request Forgery (CSRF) is an attack that tricks an authenticated user's browser into sending an unwanted state-changing request to a trusted site. CSRF defense uses unpredictable, cryptographically secure tokens tied to the user's session to verify that the request originated from the legitimate application.

**Technical Definition**  
CSRF protection in PHP is implemented by generating a cryptographically secure random token (`random_bytes(32)`), storing it in the user's session (`$_SESSION['csrf_token']`), embedding it as a hidden field in every state-changing form, and verifying it on submission using a timing-safe comparison (`hash_equals()`). The token should be regenerated after successful validation (per-request rotation) and after authentication (session regeneration). Additional defenses include `SameSite` cookie attributes (`Lax` or `Strict`), requiring custom headers for AJAX requests, and avoiding state-changing operations via GET. CSRF tokens are unnecessary for GET requests that do not modify state.

**Beginner-Friendly Explanation**  
Imagine you are logged into your bank's website. A malicious website could contain a hidden form that submits a request to your bank to transfer money. Because your browser automatically includes your bank's cookies with every request, the bank would think the request came from you. A CSRF token prevents this: your bank's form includes a secret, random value that only your bank's server knows. The malicious website cannot guess this value, so the transfer request is rejected. The token is stored in your session and checked on every form submission.

### Purposes

- To prevent attackers from forging state-changing requests on behalf of authenticated users.
- To ensure that POST requests originate from the legitimate application's forms.
- To protect sensitive operations: password changes, email changes, fund transfers, account deletion.
- To complement `SameSite` cookie attributes as defense-in-depth.
- To provide a per-session, per-form (or per-request) verification mechanism.
- To comply with OWASP CSRF prevention guidelines.
- To prevent replay attacks through token rotation.

### Syntax Rules and Structure

**General Syntax: Generating a CSRF Token**
```php
session_start();

if (empty($_SESSION['csrf_token'])) {
    $_SESSION['csrf_token'] = bin2hex(random_bytes(32));
}

$token = $_SESSION['csrf_token'];
```

**Component Breakdown**

- `session_start()` — starts or resumes the session.
- `random_bytes(32)` — generates 32 cryptographically secure random bytes.
- `bin2hex()` — converts binary to a 64-character hexadecimal string.
- `$_SESSION['csrf_token']` — stores the token in the session.
- `$token` — the token to embed in the form.

**General Syntax: Embedding the Token in a Form**
```html
<form method="post" action="process.php">
    <input type="hidden" name="csrf_token" value="<?= htmlspecialchars($token, ENT_QUOTES, 'UTF-8') ?>">
    <input type="text" name="username">
    <button type="submit">Submit</button>
</form>
```

**Component Breakdown**

- `<input type="hidden">` — hidden field that carries the token.
- `name="csrf_token"` — the field name the server will check.
- `value="..."` — the token, escaped with `htmlspecialchars()` for attribute safety.
- The token is not visible to the user but is submitted with the form.

**General Syntax: Validating the Token**
```php
session_start();

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $submitted = $_POST['csrf_token'] ?? '';
    $stored    = $_SESSION['csrf_token'] ?? '';

    if (!hash_equals($stored, $submitted)) {
        http_response_code(403);
        exit('CSRF token validation failed.');
    }

    // Token is valid; regenerate to prevent replay.
    $_SESSION['csrf_token'] = bin2hex(random_bytes(32));

    // Process the form.
}
```

**Component Breakdown**

- `$_SERVER['REQUEST_METHOD'] === 'POST'` — ensures only POST requests are checked.
- `$_POST['csrf_token']` — the token submitted with the form.
- `$_SESSION['csrf_token']` — the token stored in the session.
- `hash_equals()` — timing-safe string comparison; prevents timing attacks.
- `http_response_code(403)` — returns Forbidden on failure.
- Regeneration — issues a new token after successful validation.

**General Syntax: Setting `SameSite` Cookies (Defense-in-Depth)**
```php
session_set_cookie_params([
    'lifetime' => 0,
    'path'     => '/',
    'domain'   => 'example.com',
    'secure'   => true,
    'httponly' => true,
    'samesite' => 'Lax'   // or 'Strict'
]);
session_start();
```

**Component Breakdown**

- `samesite => 'Lax'` — cookies are sent on top-level navigations and same-site requests, but not on cross-site POST requests.
- `'Strict'` — cookies are sent only on same-site requests (may break external links).
- `'None'` — cookies are sent on all requests; requires `Secure`.

**General Syntax: AJAX CSRF with Custom Header**
```javascript
// Client-side: include a custom header.
fetch('/api/action', {
    method: 'POST',
    headers: {
        'X-CSRF-Token': document.querySelector('meta[name="csrf-token"]').content,
        'Content-Type': 'application/json'
    },
    body: JSON.stringify({ ... })
});
```
```php
// Server-side: check the header.
$headerToken = $_SERVER['HTTP_X_CSRF_TOKEN'] ?? '';
if (!hash_equals($_SESSION['csrf_token'], $headerToken)) {
    http_response_code(403);
    exit('CSRF token validation failed.');
}
```

**Component Breakdown**

- `X-CSRF-Token` — custom header; cross-site forms cannot set custom headers without CORS preflight.
- `HTTP_X_CSRF_TOKEN` — PHP's representation of the header.
- `hash_equals()` — timing-safe comparison.

**Syntax Rules**

- Tokens must be generated with `random_bytes()` or `openssl_random_pseudo_bytes()`; never use `rand()`, `mt_rand()`, or `uniqid()`.
- Tokens must be at least 128 bits (16 bytes); 256 bits (32 bytes) is recommended.
- Tokens must be stored server-side in `$_SESSION`.
- Tokens must be compared with `hash_equals()`, never with `==` or `===`.
- Tokens should be regenerated after successful validation (rotation).
- Tokens should be regenerated after login/logout (`session_regenerate_id(true)`).
- CSRF protection is required for all state-changing requests (POST, PUT, PATCH, DELETE).
- CSRF tokens are not required for GET requests that do not modify state.
- `SameSite` cookies provide defense-in-depth but are not a complete replacement for tokens.
- Tokens must be embedded in every state-changing form and validated on every submission.
- Tokens should be escaped with `htmlspecialchars()` when embedded in HTML attributes.

**Constraints and Limitations**

- CSRF tokens do not protect against XSS; if an attacker can run JavaScript on your site, they can read the token.
- Tokens must be tied to the user's session; if sessions are not used, tokens are ineffective.
- Tokens can break multi-tab browsing if rotated too aggressively; consider per-form tokens or per-session tokens with rotation on sensitive actions.
- AJAX requests need special handling (custom headers or meta tags).
- `SameSite=None` requires `Secure` and may be blocked by third-party cookie restrictions.
- CSRF protection does not protect against clickjacking; use `X-Frame-Options` or CSP.
- Tokens must be stored securely; do not expose them in URLs or logs.
- Token validation must happen before any state-changing logic.

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic CSRF Token Generation and Validation

**File: `form.php`**
```php
<?php
session_start();

// Generate a CSRF token if one doesn't exist.
if (empty($_SESSION['csrf_token'])) {
    $_SESSION['csrf_token'] = bin2hex(random_bytes(32));
}
$token = $_SESSION['csrf_token'];
?>

<!DOCTYPE html>
<html>
<body>
<form method="post" action="process.php">
    <input type="hidden" name="csrf_token" value="<?= htmlspecialchars($token, ENT_QUOTES, 'UTF-8') ?>">
    <input type="text" name="username" placeholder="Username">
    <button type="submit">Submit</button>
</form>
</body>
</html>
```

**File: `process.php`**
```php
<?php
session_start();

if ($_SERVER['REQUEST_METHOD'] !== 'POST') {
    http_response_code(405);
    exit('Method Not Allowed');
}

$submitted = $_POST['csrf_token'] ?? '';
$stored    = $_SESSION['csrf_token'] ?? '';

// Timing-safe comparison.
if (!hash_equals($stored, $submitted)) {
    http_response_code(403);
    exit('CSRF token validation failed.');
}

// Regenerate token after successful validation.
$_SESSION['csrf_token'] = bin2hex(random_bytes(32));

$username = htmlspecialchars($_POST['username'] ?? '', ENT_QUOTES, 'UTF-8');
echo "Welcome, $username!";
?>
```

**Expected Output (valid submission)**
```
Welcome, alice!
```

**Expected Output (CSRF attack)**
```
CSRF token validation failed.
```

**Why This Output Occurs**

- `form.php` generates a random token and stores it in the session.
- The token is embedded as a hidden field.
- `process.php` compares the submitted token with the session token using `hash_equals()`.
- A forged request without the correct token is rejected with 403.
- After successful validation, the token is regenerated to prevent replay.

#### Example 2: CSRF Protection with `SameSite` Cookies and AJAX

**File: `bootstrap.php`**
```php
<?php
// Configure secure session cookies.
session_set_cookie_params([
    'lifetime' => 0,
    'path'     => '/',
    'domain'   => 'example.com',
    'secure'   => true,
    'httponly' => true,
    'samesite' => 'Lax'
]);
session_start();

if (empty($_SESSION['csrf_token'])) {
    $_SESSION['csrf_token'] = bin2hex(random_bytes(32));
}
$csrfToken = $_SESSION['csrf_token'];
?>
```

**File: `ajax_handler.php`**
```php
<?php
require 'bootstrap.php';

if ($_SERVER['REQUEST_METHOD'] !== 'POST') {
    http_response_code(405);
    exit('Method Not Allowed');
}

// Check the custom header for AJAX requests.
$headerToken = $_SERVER['HTTP_X_CSRF_TOKEN'] ?? '';
if (!hash_equals($_SESSION['csrf_token'], $headerToken)) {
    http_response_code(403);
    exit('CSRF token validation failed.');
}

// Regenerate token.
$_SESSION['csrf_token'] = bin2hex(random_bytes(32));

echo json_encode(['status' => 'success']);
?>
```

**Client-Side JavaScript**
```html
<meta name="csrf-token" content="<?= htmlspecialchars($csrfToken, ENT_QUOTES, 'UTF-8') ?>">
<script>
async function sendAction() {
    const response = await fetch('/ajax_handler.php', {
        method: 'POST',
        headers: {
            'X-CSRF-Token': document.querySelector('meta[name="csrf-token"]').content
        }
    });
    const data = await response.json();
    console.log(data);
}
</script>
```

**Expected Output (valid AJAX request)**
```json
{"status":"success"}
```

**Why This Output Occurs**

- The CSRF token is stored in a meta tag and sent via a custom `X-CSRF-Token` header.
- Cross-site forms cannot set custom headers without a CORS preflight, adding protection.
- The server validates the header token with `hash_equals()`.
- `SameSite=Lax` provides defense-in-depth by not sending cookies on cross-site POST requests.

### Real-World Cases

- **Password change forms:** Require CSRF token to prevent forced password changes.
- **Email change:** Prevent attackers from changing the user's email address.
- **Fund transfers:** Protect financial transactions.
- **Account deletion:** Prevent malicious deletion requests.
- **Admin actions:** Protect bulk operations and configuration changes.
- **API endpoints:** Use custom headers or token-based CSRF protection for AJAX.

---

## Security Considerations

### Defense in Depth

- **Validate at input, encode at output, token at state change.** Do not rely on a single layer.
- Use `FILTER_VALIDATE_*` for structural checks and `FILTER_SANITIZE_*` for cleaning.
- Use `htmlspecialchars()` with `ENT_QUOTES` and explicit `'UTF-8'` for all HTML output.
- Use prepared statements (PDO, mysqli) for all database queries.
- Use `password_hash()` and `password_verify()` for password storage.
- Use HTTPS for all traffic; set `Secure` and `HttpOnly` on cookies.
- Set `Content-Security-Policy` headers as defense-in-depth against XSS.

### Common Mistakes

- **Using `FILTER_SANITIZE_STRING`** — deprecated in PHP 8.1; use `FILTER_SANITIZE_FULL_SPECIAL_CHARS`.
- **Relying on `addslashes()` or `mysql_real_escape_string()`** for SQL — use prepared statements.
- **Using `$_REQUEST`** — ambiguous source; use `$_GET` or `$_POST` explicitly.
- **Encoding at input instead of output** — makes data unrecoverable and context-dependent.
- **Double-encoding** — causes display artifacts.
- **Using `htmlspecialchars()` in JavaScript contexts** — does not protect against JS injection.
- **Comparing CSRF tokens with `==` or `===`** — use `hash_equals()`.
- **Using `rand()` or `mt_rand()` for CSRF tokens** — use `random_bytes()`.
- **Not regenerating session ID after login** — enables session fixation.
- **Not escaping output in attributes** — use `ENT_QUOTES`.

### Deprecated, Unsafe, or Version-Specific Features

| Feature | Status | Replacement |
|---|---|---|
| `FILTER_SANITIZE_STRING` | Deprecated in PHP 8.1 | `FILTER_SANITIZE_FULL_SPECIAL_CHARS` |
| `mysql_real_escape_string()` | Removed in PHP 7 | PDO prepared statements |
| `addslashes()` for SQL | Unsafe | Prepared statements |
| `magic_quotes_gpc` | Removed in PHP 5.4 | Explicit escaping |
| `register_globals` | Removed in PHP 5.4 | Superglobals |
| `session_register()` | Removed in PHP 5.4 | `$_SESSION` |
| `htmlentities()` for all output | Overkill | `htmlspecialchars()` |
| `ENT_IGNORE` | Discouraged | `ENT_SUBSTITUTE` |
| `mt_rand()` for tokens | Insecure | `random_bytes()` |
| `uniqid()` for tokens | Predictable | `random_bytes()` |

### References Links

- PHP: Filter Functions — https://www.php.net/manual/en/book.filter.php
- PHP: `filter_var()` — https://www.php.net/manual/en/function.filter-var.php
- PHP: `filter_input()` — https://www.php.net/manual/en/function.filter-input.php
- PHP: Validation Filters — https://www.php.net/manual/en/filter.filters.validate.php
- PHP: Sanitization Filters — https://www.php.net/manual/en/filter.filters.sanitize.php
- PHP: Filter Flags — https://www.php.net/manual/en/filter.filters.flags.php
- PHP: `htmlspecialchars()` — https://www.php.net/manual/en/function.htmlspecialchars.php
- PHP: `htmlentities()` — https://www.php.net/manual/en/function.htmlentities.php
- PHP: `hash_equals()` — https://www.php.net/manual/en/function.hash-equals.php
- PHP: `random_bytes()` — https://www.php.net/manual/en/function.random-bytes.php
- PHP: `session_start()` — https://www.php.net/manual/en/function.session-start.php
- PHP: Session Security — https://www.php.net/manual/en/session.security.php
- PHP: `password_hash()` — https://www.php.net/manual/en/function.password-hash.php
- PHP: `password_verify()` — https://www.php.net/manual/en/function.password-verify.php
- OWASP: Cross Site Scripting Prevention Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html
- OWASP: Cross-Site Request Forgery Prevention Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html
- OWASP: Input Validation Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html
- OWASP: PHP Security Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/PHP_Configuration_Cheat_Sheet.html
- OWASP Top 10: A03 Injection — https://owasp.org/Top10/A03_2021-Injection/
- CWE-79: Improper Neutralization of Input During Web Page Generation ('Cross-site Scripting') — https://cwe.mitre.org/data/definitions/79.html
- CWE-352: Cross-Site Request Forgery (CSRF) — https://cwe.mitre.org/data/definitions/352.html
- MDN: HTML Entities — https://developer.mozilla.org/en-US/docs/Glossary/Entity
- RFC 5321: Simple Mail Transfer Protocol — https://datatracker.ietf.org/doc/html/rfc5321
- RFC 6265: HTTP State Management Mechanism — https://datatracker.ietf.org/doc/html/rfc6265