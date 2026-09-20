# PHP HTML Forms & State Management — Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**  
PHP HTML Forms & State Management is the practice of receiving data submitted by a browser through HTML form controls, processing that data in PHP, and managing application state across stateless HTTP requests so that user interactions remain consistent, secure, and predictable.

**Technical Definition**  
It encompasses the HTTP request methods `GET` and `POST`, HTML form encoding types (`application/x-www-form-urlencoded`, `multipart/form-data`, and `text/plain`), PHP superglobal arrays (`$_GET`, `$_POST`, `$_FILES`, `$_REQUEST`), server-side validation and filtering, file upload handling, session/cookie state, and the Post/Redirect/Get (PRG) design pattern used to prevent duplicate form submissions. It also includes security controls such as CSRF tokens, output escaping, and upload validation.

**Beginner-Friendly Explanation**  
When you fill out a web form and click “Submit,” the browser packages your answers and sends them to a PHP script. The script reads the answers, validates them, and does something useful — save to a database, send an email, upload a photo. Because HTTP does not remember you from one request to the next, PHP must use sessions, redirects, hidden fields, or cookies to manage “state.” The PRG pattern is a common trick: after a successful POST, the server redirects the browser to a normal GET page, so refreshing the page does not accidentally submit the form again.

---

### Key Characteristics

- HTTP is stateless; state must be simulated with sessions, cookies, redirects, or hidden fields.
- `GET` is safe, idempotent, cacheable, and carries data in the URL query string.
- `POST` is not safe, not idempotent, and carries data in the request body.
- HTML forms can encode data as URL-encoded, multipart, or plain text.
- PHP exposes incoming data through superglobals: `$_GET`, `$_POST`, `$_FILES`, `$_REQUEST`.
- Server-side validation is mandatory; client-side validation is only a convenience.
- File uploads require `enctype="multipart/form-data"` and careful server-side handling.
- PRG eliminates accidental duplicate submissions on browser refresh.
- Security threats include cross-site request forgery (CSRF), cross-site scripting (XSS), SQL injection, and malicious file uploads.
- Deprecated or unsafe features include `register_globals` (removed), `$_REQUEST` for state-changing operations, `FILTER_SANITIZE_STRING` (deprecated in PHP 8.1), and relying on the browser-supplied MIME type for uploads.

---

### Prerequisites

- Basic HTML: forms, inputs, labels, buttons.
- Basic HTTP: requests, responses, headers, status codes.
- Basic PHP: variables, arrays, superglobals, functions, `header()`.
- A web server with PHP installed (Apache, Nginx + PHP-FPM, or PHP built-in server).
- Understanding of basic web security concepts.

---

### Related Programming Areas

- HTTP protocol and REST semantics.
- Web application security (OWASP Top 10).
- Session management and authentication.
- Front-end form validation and accessibility.
- File system operations and MIME type detection.
- Database input parameterization and prepared statements.
- MVC frameworks (Laravel, Symfony, CodeIgniter) and their form/request abstractions.

---

## Core Concepts / Features

## 1. HTTP GET Method

### Definitions

**Core Definition**  
`GET` is an HTTP method used to request a representation of a resource; in HTML forms, it appends form data to the URL query string.

**Technical Definition**  
The `GET` method requests a representation of the specified resource. Requests using `GET` should only retrieve data and should not contain a request body. `GET` is safe, idempotent, and cacheable. When used with an HTML form, the browser constructs a query string from the form’s successful controls and appends it to the `action` URL after a `?`. PHP populates `$_GET` with the decoded query-string parameters.

**Beginner-Friendly Explanation**  
A GET form puts your answers directly into the web address. If you search for “PHP forms,” the URL might look like `search.php?q=PHP+forms`. Because the data is in the URL, you can bookmark it, share it, and see it in your browser history. GET is best for searches, filters, and pages that only read data.

### Purposes

- To retrieve data from the server without causing side effects.
- To make form submissions bookmarkable, shareable, and cacheable.
- To pass small, non-sensitive parameters through the URL.
- To support back/forward navigation and refresh without resubmission warnings.
- To allow server-side scripts to read query parameters through `$_GET`.

### Syntax Rules and Structure

**General Syntax: HTTP GET Request**
```
GET /path/resource?key1=value1&key2=value2 HTTP/1.1
Host: example.com
User-Agent: ...
Accept: ...
```

**Component Breakdown**

- `GET` — HTTP method token.
- `/path/resource` — request target, usually an absolute path on the origin server.
- `?` — delimiter that begins the query component.
- `key1=value1` — one query parameter; keys and values are percent-encoded.
- `&` — separator between query parameters.
- `Host: example.com` — required in HTTP/1.1; identifies the target host.
- No request body — `GET` should not contain content.

**General Syntax: HTML GET Form**
```html
<form action="search.php" method="get">
  <label for="q">Search:</label>
  <input type="text" id="q" name="q">
  <button type="submit">Search</button>
</form>
```

**Component Breakdown**

- `<form>` — container for controls.
- `action="search.php"` — URL that receives the request.
- `method="get"` — instructs the browser to use `GET`.
- `name="q"` — parameter name that becomes `$_GET['q']` in PHP.
- `<button type="submit">` — triggers submission.

**General Syntax: PHP Access**
```php
$value = $_GET['key'] ?? 'default';
$clean = filter_input(INPUT_GET, 'key', FILTER_SANITIZE_SPECIAL_CHARS);
```

**Component Breakdown**

- `$_GET` — associative array of query-string variables.
- `??` — null-coalescing operator for default values.
- `filter_input(INPUT_GET, ...)` — reads and optionally filters a GET variable.

**Syntax Rules**

- `GET` requests should not modify server state.
- Query-string parameters are visible in the URL, browser history, and server logs.
- URL length is limited by browsers and servers; practical limits vary but are often around 2,000–8,000 characters.
- PHP automatically URL-decodes values in `$_GET`.
- Dots and spaces in parameter names are converted to underscores by PHP.

**Constraints and Limitations**

- Do not use `GET` for passwords, tokens, or sensitive data.
- Do not use `GET` for operations that create, update, or delete data.
- Query strings have length constraints.
- `GET` is cacheable; stale data may be served unless cache headers are managed.
- `$_GET` is populated whenever a query string is present, regardless of the HTTP request method.

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic GET Search

**File: `search.php`**
```php
<?php
// Read the 'q' parameter from the query string.
// filter_input() returns null if the variable is not set.
$q = filter_input(INPUT_GET, 'q', FILTER_SANITIZE_SPECIAL_CHARS);

// If a search term exists, display it after escaping for HTML context.
if ($q !== null && $q !== '') {
    echo 'You searched for: ' . htmlspecialchars($q, ENT_QUOTES, 'UTF-8');
} else {
    echo 'No search term provided.';
}
?>
```

**How to Run**

1. Save as `search.php` in your web root.
2. Visit `search.php?q=PHP%20forms`.
3. The browser sends `GET /search.php?q=PHP%20forms`.

**Expected Output**
```
You searched for: PHP forms
```

**Why This Output Occurs**

- The query string contains `q=PHP%20forms`.
- PHP decodes `%20` to a space and places `PHP forms` in `$_GET['q']`.
- `filter_input()` reads the raw GET variable and applies `FILTER_SANITIZE_SPECIAL_CHARS`.
- `htmlspecialchars()` escapes characters that have special meaning in HTML, preventing XSS when echoing user input.

#### Example 2: GET with Multiple Parameters and Defaults

**File: `filter.php`**
```php
<?php
// Read page and sort with defaults.
$page = filter_input(INPUT_GET, 'page', FILTER_VALIDATE_INT, [
    'options' => ['default' => 1, 'min_range' => 1]
]);

$sort = $_GET['sort'] ?? 'newest';

// Whitelist allowed sort values.
$allowedSort = ['newest', 'oldest', 'popular'];
if (!in_array($sort, $allowedSort, true)) {
    $sort = 'newest';
}

echo "Page: $page<br>";
echo "Sort: $sort";
?>
```

**How to Run**

1. Save as `filter.php`.
2. Visit `filter.php?page=3&sort=popular`.

**Expected Output**
```
Page: 3
Sort: popular
```

**Why This Output Occurs**

- `FILTER_VALIDATE_INT` validates that `page` is an integer and applies the default `1` if missing or invalid.
- `$_GET['sort']` is read directly, then checked against a whitelist.
- Whitelisting prevents unexpected values from reaching application logic.

### Real-World Cases

- **Search pages:** `GET /search?q=keyboard&category=electronics` — bookmarkable and cacheable.
- **Pagination:** `GET /articles?page=3&per_page=20` — safe to refresh and share.
- **Filtering:** `GET /products?color=red&size=large` — no server state change.
- **Analytics:** Query parameters are logged by servers and analytics tools; avoid sensitive data.

---

## 2. HTTP POST Method

### Definitions

**Core Definition**  
`POST` is an HTTP method used to submit an entity to the server, often causing a change in state or side effects.

**Technical Definition**  
The `POST` method sends data to the server in the request body. The body format is indicated by the `Content-Type` header. HTML forms typically use `application/x-www-form-urlencoded` or `multipart/form-data`. `POST` is not safe and not idempotent: successive identical requests may create multiple resources or repeat side effects. PHP populates `$_POST` for URL-encoded and multipart form submissions, and `$_FILES` for uploaded files.

**Beginner-Friendly Explanation**  
A POST form sends your answers in the body of the request, not in the URL. You won’t see them in the address bar. POST is used when the server should do something — create an account, place an order, submit a comment, upload a file. Because POST can change data, refreshing the result page can accidentally repeat the action; this is why the PRG pattern is important.

### Purposes

- To submit data that changes server state.
- To send larger or binary payloads, including file uploads.
- To hide form data from the URL, browser history, and basic server logs.
- To trigger actions such as account creation, checkout, or data deletion.
- To work with `$_POST` and `$_FILES` in PHP.

### Syntax Rules and Structure

**General Syntax: HTTP POST Request**
```
POST /process.php HTTP/1.1
Host: example.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 27

name=Alice&email=alice%40example.com
```

**Component Breakdown**

- `POST` — HTTP method token.
- `/process.php` — request target.
- `Host` — target host.
- `Content-Type` — media type of the body.
- `Content-Length` — size of the body in bytes.
- Blank line — separates headers from body.
- Body — encoded form data.

**General Syntax: HTML POST Form**
```html
<form action="process.php" method="post">
  <label for="name">Name:</label>
  <input type="text" id="name" name="name">
  <label for="email">Email:</label>
  <input type="email" id="email" name="email">
  <button type="submit">Submit</button>
</form>
```

**Component Breakdown**

- `method="post"` — instructs the browser to use `POST`.
- `name` attributes — become keys in `$_POST`.
- No `enctype` — defaults to `application/x-www-form-urlencoded`.
- `action` — URL that receives the POST body.

**General Syntax: PHP Access**
```php
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $name = $_POST['name'] ?? '';
    $email = filter_input(INPUT_POST, 'email', FILTER_VALIDATE_EMAIL);
}
```

**Component Breakdown**

- `$_SERVER['REQUEST_METHOD']` — reliable way to check the method.
- `$_POST['name']` — raw POST value; must be validated/escaped.
- `filter_input(INPUT_POST, ...)` — reads and filters a POST variable.

**Syntax Rules**

- `POST` may have a body; `GET` should not.
- `POST` is not idempotent; repeated requests may repeat side effects.
- `POST` is not safe; it can change server state.
- `POST` is cacheable only when explicit freshness information is included.
- `$_POST` is populated only for `application/x-www-form-urlencoded` and `multipart/form-data`.
- For `application/json` or XML, use `php://input`.

**Constraints and Limitations**

- `POST` data is not visible in the URL, but it is still user-controlled and untrusted.
- `POST` does not guarantee confidentiality over plain HTTP; use HTTPS.
- `post_max_size` in `php.ini` limits total POST body size.
- `max_input_vars` limits the number of form fields.
- File uploads require `multipart/form-data` and are subject to `upload_max_filesize`, `post_max_size`, and `max_file_uploads`.

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic POST Processing

**File: `process.php`**
```php
<?php
// Only process if the request method is POST.
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    // Read and trim the name; use null coalescing to avoid notices.
    $name = trim($_POST['name'] ?? '');

    // Basic server-side validation.
    if ($name === '') {
        echo 'Name is required.';
        exit;
    }

    // Escape output for HTML context.
    echo 'Hello ' . htmlspecialchars($name, ENT_QUOTES, 'UTF-8') . '!';
} else {
    echo 'Please submit the form.';
}
?>
```

**How to Run**

1. Save as `process.php`.
2. Create an HTML form that posts `name=Alice` to `process.php`.
3. Submit the form.

**Expected Output**
```
Hello Alice!
```

**Why This Output Occurs**

- `$_SERVER['REQUEST_METHOD']` confirms a POST request.
- `$_POST['name']` contains `Alice`.
- `trim()` removes accidental whitespace.
- `htmlspecialchars()` prevents HTML injection.
- The script outputs the escaped value.

#### Example 2: POST with Validation and Error Handling

**File: `register.php`**
```php
<?php
$errors = [];
$name = '';
$email = '';

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $name = trim($_POST['name'] ?? '');
    $email = trim($_POST['email'] ?? '');

    if ($name === '') {
        $errors[] = 'Name is required.';
    }

    // Validate email using PHP's filter.
    if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
        $errors[] = 'A valid email is required.';
    }

    if (empty($errors)) {
        echo 'Registration successful for ' . htmlspecialchars($name, ENT_QUOTES, 'UTF-8');
        exit;
    }
}

// Display errors and re-populate the form.
foreach ($errors as $error) {
    echo '<p style="color:red;">' . htmlspecialchars($error, ENT_QUOTES, 'UTF-8') . '</p>';
}
?>
<form method="post" action="register.php">
  <input type="text" name="name" value="<?= htmlspecialchars($name, ENT_QUOTES, 'UTF-8') ?>" placeholder="Name">
  <input type="email" name="email" value="<?= htmlspecialchars($email, ENT_QUOTES, 'UTF-8') ?>" placeholder="Email">
  <button type="submit">Register</button>
</form>
```

**Expected Output (invalid email)**
```
A valid email is required.
```
Then the form is re-displayed with the previously entered name and email.

**Why This Output Occurs**

- `filter_var()` with `FILTER_VALIDATE_EMAIL` returns `false` for invalid emails.
- Errors are collected in an array.
- If errors exist, the script displays them and re-renders the form with escaped old values.
- If validation passes, it outputs a success message.

### Real-World Cases

- **User registration:** POST to create an account.
- **Checkout:** POST to place an order and charge a payment method.
- **Comment submission:** POST to store a comment.
- **File upload:** POST with `multipart/form-data`.
- **Login:** POST credentials over HTTPS; never use GET.

---

## 3. Form Controls & Metadata

### Definitions

**Core Definition**  
Form controls are the interactive HTML elements — inputs, selects, checkboxes, radios, textareas, buttons — that collect user data. Metadata includes `name`, `value`, `type`, `method`, `action`, `enctype`, labels, and field grouping.

**Technical Definition**  
The HTML form submission algorithm constructs an entry list from successful controls. Each control contributes a name/value pair. The `name` attribute determines the key in `$_GET` or `$_POST`. Controls with names ending in `[]` cause PHP to create arrays. Checkboxes and radios contribute values only when checked/selected. `<select multiple>` submits multiple values for the same name. File inputs contribute entries to `$_FILES`.

**Beginner-Friendly Explanation**  
Think of a form as a collection of labeled boxes. Each box has a `name` tag so PHP knows what it is, and a `value` that is sent when the form is submitted. If you want PHP to receive multiple values under one name, you add `[]` to the name. Checkboxes are like light switches: only the ones turned on send a value. Radio buttons are like a group of mutually exclusive options: only one can be on. A multiple-select box lets users pick several options.

### Purposes

- To collect structured user input in a predictable way.
- To map HTML control names to PHP array keys.
- To group related controls using `<fieldset>` and `<legend>`.
- To support single-value, multi-value, boolean, and file inputs.
- To improve accessibility through labels and semantic controls.
- To control encoding via `enctype` and submission via `method`/`action`.

### Syntax Rules and Structure

**General Syntax: Common Form Controls**
```html
<form action="process.php" method="post" enctype="multipart/form-data">
  <label for="username">Username:</label>
  <input type="text" id="username" name="username" required>

  <label for="bio">Bio:</label>
  <textarea id="bio" name="bio"></textarea>

  <label for="country">Country:</label>
  <select id="country" name="country">
    <option value="">Choose...</option>
    <option value="us">United States</option>
    <option value="ca">Canada</option>
  </select>

  <fieldset>
    <legend>Hobbies</legend>
    <label><input type="checkbox" name="hobbies[]" value="reading"> Reading</label>
    <label><input type="checkbox" name="hobbies[]" value="coding"> Coding</label>
  </fieldset>

  <fieldset>
    <legend>Gender</legend>
    <label><input type="radio" name="gender" value="female"> Female</label>
    <label><input type="radio" name="gender" value="male"> Male</label>
  </fieldset>

  <label for="colors">Favorite colors:</label>
  <select id="colors" name="colors[]" multiple>
    <option value="red">Red</option>
    <option value="blue">Blue</option>
    <option value="green">Green</option>
  </select>

  <label for="avatar">Avatar:</label>
  <input type="file" id="avatar" name="avatar" accept="image/*">

  <button type="submit">Submit</button>
</form>
```

**Component Breakdown**

- `<form>` — container; `action` is the processing URL, `method` is GET or POST, `enctype` controls encoding.
- `<label for="...">` — associates a label with a control by ID; improves accessibility.
- `<input type="text">` — single-line text; `name` becomes the key.
- `<textarea>` — multi-line text.
- `<select>` — drop-down; single-value unless `multiple` is present.
- `<option value="...">` — option value submitted when selected.
- `<input type="checkbox" name="hobbies[]">` — multiple values can be selected; `[]` makes PHP create an array.
- `<input type="radio" name="gender">` — same `name` groups radios; only one value is submitted.
- `<input type="file">` — file upload control; requires `enctype="multipart/form-data"`.
- `<button type="submit">` — submits the form.

**General Syntax: PHP Access to Arrays**
```php
$hobbies = $_POST['hobbies'] ?? [];      // array or empty array
$colors  = $_POST['colors'] ?? [];       // array from multiple select
$gender  = $_POST['gender'] ?? '';       // scalar
```

**Component Breakdown**

- `$_POST['hobbies']` — array if the name ended with `[]` and at least one checkbox was checked.
- `$_POST['colors']` — array from `<select multiple>`.
- `$_POST['gender']` — scalar string from radio group.

**Syntax Rules**

- Every control that should submit data must have a `name`.
- Controls without a `name` are not submitted.
- Disabled controls are not submitted.
- Checkboxes and radios submit only when checked/selected.
- `[]` at the end of a name tells PHP to build an array.
- You can use explicit keys: `name="items[product_id]"`.
- `<select multiple>` submits an array; use `name="colors[]"`.
- File inputs require `method="post"` and `enctype="multipart/form-data"`.
- `MAX_FILE_SIZE` hidden field is a client-side convenience only; it cannot be trusted.

**Constraints and Limitations**

- `max_input_vars` limits the number of form fields PHP will parse.
- Nested arrays deeper than the `max_input_nesting_level` may be truncated.
- Dots and spaces in field names are converted to underscores in PHP.
- If a field name begins with array syntax, trailing characters are silently ignored.
- `$_POST` does not include unchecked checkboxes or unselected radios.
- Multiple select values must use array syntax; otherwise only the last value is available.
- File uploads are subject to PHP configuration limits.

### Annotated Code Examples and Expected Outputs

#### Example 1: Checkbox Array, Radio, and Multiple Select

**File: `form.html`**
```html
<form method="post" action="controls.php">
    <fieldset>
        <legend>Hobbies</legend>
        <label><input type="checkbox" name="hobbies[]" value="reading"> Reading</label>
        <label><input type="checkbox" name="hobbies[]" value="coding"> Coding</label>
    </fieldset>
  
    <fieldset>
        <legend>Gender</legend>
        <label><input type="radio" name="gender" value="female"> Female</label>
        <label><input type="radio" name="gender" value="male"> Male</label>
    </fieldset>
  
    <label for="colors">Colors:</label>
    <select id="colors" name="colors[]" multiple>
        <option value="red">Red</option>
        <option value="blue">Blue</option>
        <option value="green">Green</option>
    </select>
  
    <button type="submit">Submit</button>
</form>
```

**File: `controls.php`**
```php
<?php
// Read arrays with defaults.
$hobbies = $_POST['hobbies'] ?? [];
$colors  = $_POST['colors'] ?? [];
$gender  = $_POST['gender'] ?? 'not specified';

// Escape each value before output.
$hobbies = array_map(fn($v) => htmlspecialchars($v, ENT_QUOTES, 'UTF-8'), $hobbies);
$colors  = array_map(fn($v) => htmlspecialchars($v, ENT_QUOTES, 'UTF-8'), $colors);

echo 'Hobbies: ' . implode(', ', $hobbies) . '<br>';
echo 'Colors: ' . implode(', ', $colors) . '<br>';
echo 'Gender: ' . htmlspecialchars($gender, ENT_QUOTES, 'UTF-8');
?>
```

**How to Run**

1. Save `form.html` and `controls.php` in the same directory.
2. Open `form.html` in a browser.
3. Select “Reading” and “Coding,” choose “Female,” select “Red” and “Blue,” then submit.

**Expected Output**
```
Hobbies: reading, coding
Colors: red, blue
Gender: female
```

**Why This Output Occurs**

- `name="hobbies[]"` makes PHP create `$_POST['hobbies']` as an array.
- Only checked checkboxes are submitted.
- `name="colors[]"` makes the multiple select submit an array.
- `name="gender"` is a radio group; only the selected value is submitted.
- `array_map()` escapes each array element before `implode()`.

#### Example 2: Nested Array Names and File Input Metadata

**File: `order.php`**
```php
<?php
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    // Nested array from names like items[0][product_id].
    $items = $_POST['items'] ?? [];

    foreach ($items as $index => $item) {
        $productId = htmlspecialchars($item['product_id'] ?? '', ENT_QUOTES, 'UTF-8');
        $qty = (int)($item['qty'] ?? 0);
        echo "Item $index: Product $productId, Qty $qty<br>";
    }

    // File metadata (if a file input named 'receipt' exists).
    if (isset($_FILES['receipt'])) {
        $file = $_FILES['receipt'];
        echo 'File name: ' . htmlspecialchars($file['name'], ENT_QUOTES, 'UTF-8') . '<br>';
        echo 'File size: ' . (int)$file['size'] . ' bytes<br>';
        echo 'Temporary path: ' . htmlspecialchars($file['tmp_name'], ENT_QUOTES, 'UTF-8');
    }
}
?>
<form method="post" action="order.php" enctype="multipart/form-data">
  <input type="text" name="items[0][product_id]" value="P100">
  <input type="number" name="items[0][qty]" value="2">
  <input type="text" name="items[1][product_id]" value="P200">
  <input type="number" name="items[1][qty]" value="1">
  <input type="file" name="receipt">
  <button type="submit">Submit</button>
</form>
```

**Expected Output**
```
Item 0: Product P100, Qty 2
Item 1: Product P200, Qty 1
File name: receipt.pdf
File size: 12345 bytes
Temporary path: /tmp/phpXYZ123
```

**Why This Output Occurs**

- `name="items[0][product_id]"` creates a nested PHP array: `$_POST['items'][0]['product_id']`.
- The loop iterates over each item and outputs escaped product IDs and integer quantities.
- `$_FILES['receipt']` contains metadata for the uploaded file.
- `tmp_name` is the server-side temporary path; `name` is the client-side original name.
- `size` is the file size in bytes.

### Real-World Cases

- **E-commerce checkout:** nested arrays for line items, quantities, and options.
- **Surveys:** checkbox arrays for multiple-choice questions.
- **Profile settings:** radio groups for preferences, multi-select for skills.
- **File upload forms:** resume, avatar, receipt, or document upload.
- **Admin panels:** bulk actions with checkbox arrays.

---

## 4. Multipart Form Data & File Uploads

### Definitions

**Core Definition**  
`multipart/form-data` is a MIME type used to submit forms that contain files or binary data. Each form control becomes a separate “part” in the request body.

**Technical Definition**  
Defined by RFC 7578, `multipart/form-data` encodes each form field as a MIME part with a `Content-Disposition` header containing a `name` parameter and optionally a `filename` parameter. A boundary string separates parts. PHP parses multipart requests into `$_POST` for regular fields and `$_FILES` for uploaded files. File uploads are stored in a temporary directory and must be moved with `move_uploaded_file()`.

**Beginner-Friendly Explanation**  
A normal form sends data as one long string. A multipart form breaks the data into separate labeled chunks — one chunk for your name, one chunk for your email, one chunk for the file you uploaded. This is necessary because files are binary and cannot be safely squeezed into a URL-encoded string. In PHP, the regular fields appear in `$_POST`, and the file details appear in `$_FILES`.

### Purposes

- To upload files from the browser to the server.
- To submit binary data safely without percent-encoding.
- To separate form fields into distinct MIME parts.
- To provide file metadata: original name, MIME type, size, temporary path, error code.
- To support multiple files under one field name.

### Syntax Rules and Structure

**General Syntax: Multipart HTTP Request**
```
POST /upload.php HTTP/1.1
Host: example.com
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW

------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="username"

Alice
------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="avatar"; filename="me.png"
Content-Type: image/png

<binary PNG data>
------WebKitFormBoundary7MA4YWxkTrZu0gW--
```

**Component Breakdown**

- `Content-Type: multipart/form-data; boundary=...` — declares multipart encoding and the boundary string.
- `--boundary` — delimiter that separates parts.
- `Content-Disposition: form-data; name="username"` — names the part; regular field.
- `Content-Disposition: form-data; name="avatar"; filename="me.png"` — file part with original filename.
- `Content-Type: image/png` — optional part-level MIME type; browser-supplied and not trustworthy.
- Blank line — separates part headers from part body.
- `--boundary--` — closing boundary.

**General Syntax: HTML Multipart Form**
```html
<form action="upload.php" method="post" enctype="multipart/form-data">
  <label for="username">Username:</label>
  <input type="text" id="username" name="username">

  <label for="avatar">Avatar:</label>
  <input type="file" id="avatar" name="avatar" accept="image/*">

  <button type="submit">Upload</button>
</form>
```

**Component Breakdown**

- `enctype="multipart/form-data"` — required for file uploads.
- `method="post"` — required; file uploads cannot use GET.
- `type="file"` — file selection control.
- `name="avatar"` — becomes the key in `$_FILES`.
- `accept="image/*"` — client-side hint only; not a security control.

**General Syntax: PHP `$_FILES` Structure**
```php
$_FILES['avatar'] = [
    'name'      => 'me.png',          // original client filename
    'type'      => 'image/png',       // browser-supplied MIME type (untrusted)
    'size'      => 12345,             // bytes
    'tmp_name'  => '/tmp/phpXYZ123',  // temporary server path
    'error'     => UPLOAD_ERR_OK,     // error code
    'full_path' => 'C:\Users\...\me.png' // as of PHP 8.1.0; untrusted
];
```

**Component Breakdown**

- `name` — original filename from the client; must be sanitized.
- `type` — MIME type supplied by the browser; do not trust.
- `size` — file size in bytes.
- `tmp_name` — temporary path on the server.
- `error` — upload error code (`UPLOAD_ERR_OK`, `UPLOAD_ERR_INI_SIZE`, etc.).
- `full_path` — full path as submitted by the browser; available as of PHP 8.1.0; cannot be trusted.

**General Syntax: Moving an Uploaded File**
```php
if ($_FILES['avatar']['error'] === UPLOAD_ERR_OK) {
    $tmp = $_FILES['avatar']['tmp_name'];
    $dest = __DIR__ . '/uploads/' . basename($_FILES['avatar']['name']);
    move_uploaded_file($tmp, $dest);
}
```

**Component Breakdown**

- `UPLOAD_ERR_OK` — no error.
- `$tmp` — temporary path from `$_FILES`.
- `basename()` — strips directory components to prevent path traversal.
- `move_uploaded_file()` — safely moves an uploaded file; checks that the source is a valid upload.

**Syntax Rules**

- File upload forms must use `method="post"` and `enctype="multipart/form-data"`.
- `MAX_FILE_SIZE` must precede the file input; it is client-side only.
- PHP stores uploaded files in a temporary directory.
- Use `move_uploaded_file()` or `is_uploaded_file()` to validate the upload source.
- Never use `rename()` or `copy()` directly on `tmp_name` without upload validation.
- Always check `$_FILES['field']['error']`.
- Validate file size, extension, and MIME type server-side.
- Generate a safe filename; do not trust the client filename.

**Constraints and Limitations**

- `upload_max_filesize` limits individual file size.
- `post_max_size` limits total POST body size.
- `max_file_uploads` limits the number of files per request.
- `upload_tmp_dir` controls the temporary directory.
- `max_input_time` limits parsing time.
- `$_FILES` is empty if the form does not use `multipart/form-data`.
- `full_path` is available only as of PHP 8.1.0 and is untrusted.
- `is_uploaded_file()` is unnecessary before `move_uploaded_file()` because `move_uploaded_file()` performs the same check.
- Uploaded filenames can contain path traversal sequences if not sanitized.
- Browser-supplied MIME types can be forged.

### Annotated Code Examples and Expected Outputs

#### Example 1: Single File Upload

**File: `upload_form.html`**
```html
<form action="upload.php" method="post" enctype="multipart/form-data">
    <label for="avatar">Choose an image:</label>
    <input type="file" id="avatar" name="avatar" accept="image/*" required>
    <button type="submit">Upload</button>
</form>
```

**File: `upload.php`**
```php
<?php
// Ensure the request is POST and the file field exists.
if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_FILES['avatar'])) {
    $file = $_FILES['avatar'];

    // Check for upload errors.
    if ($file['error'] !== UPLOAD_ERR_OK) {
        echo 'Upload error code: ' . (int)$file['error'];
        exit;
    }

    // Basic size limit: 2 MB.
    if ($file['size'] > 2 * 1024 * 1024) {
        echo 'File too large.';
        exit;
    }

    // Sanitize the filename and prevent path traversal.
    $safeName = basename($file['name']);
    $safeName = preg_replace('/[^A-Za-z0-9._-]/', '_', $safeName);

    // Ensure the uploads directory exists and is writable.
    $uploadsDir = __DIR__ . '/uploads';
    if (!is_dir($uploadsDir)) {
        mkdir($uploadsDir, 0755, true);
    }

    $destination = $uploadsDir . '/' . $safeName;

    // Move the uploaded file safely.
    if (move_uploaded_file($file['tmp_name'], $destination)) {
        echo 'Uploaded: ' . htmlspecialchars($safeName, ENT_QUOTES, 'UTF-8');
    } else {
        echo 'Failed to move uploaded file.';
    }
} else {
    echo 'No file uploaded.';
}
?>
```

**How to Run**

1. Save both files in the same directory.
2. Create an `uploads/` directory or let the script create it.
3. Open `upload_form.html`, choose an image, and submit.

**Expected Output**
```
Uploaded: me.png
```

**Why This Output Occurs**

- `$_FILES['avatar']['error']` is `UPLOAD_ERR_OK` for a successful upload.
- The size check rejects files larger than 2 MB.
- `basename()` strips directories; `preg_replace()` replaces unsafe characters.
- `move_uploaded_file()` verifies the source is a valid upload and moves it.
- The script outputs the sanitized filename.

#### Example 2: Multiple File Upload with Array Field Name

**File: `multi_upload.html`**
```html
<form action="multi_upload.php" method="post" enctype="multipart/form-data">
  <label for="photos">Choose photos:</label>
  <input type="file" id="photos" name="photos[]" multiple accept="image/*">
  <button type="submit">Upload</button>
</form>
```

**File: `multi_upload.php`**
```php
<?php
if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_FILES['photos'])) {
    $files = $_FILES['photos'];
    $uploadsDir = __DIR__ . '/uploads';
    if (!is_dir($uploadsDir)) {
        mkdir($uploadsDir, 0755, true);
    }

    // Iterate over each file.
    for ($i = 0; $i < count($files['name']); $i++) {
        if ($files['error'][$i] !== UPLOAD_ERR_OK) {
            echo "File $i error code: " . (int)$files['error'][$i] . "<br>";
            continue;
        }

        $safeName = basename($files['name'][$i]);
        $safeName = preg_replace('/[^A-Za-z0-9._-]/', '_', $safeName);
        $destination = $uploadsDir . '/' . $safeName;

        if (move_uploaded_file($files['tmp_name'][$i], $destination)) {
            echo "Uploaded: " . htmlspecialchars($safeName, ENT_QUOTES, 'UTF-8') . "<br>";
        } else {
            echo "Failed to move file $i.<br>";
        }
    }
}
?>
```

**Expected Output**
```
Uploaded: photo1.jpg
Uploaded: photo2.png
```

**Why This Output Occurs**

- `name="photos[]"` causes `$_FILES['photos']` to have array keys: `name`, `type`, `tmp_name`, `error`, `size`.
- The loop iterates over each file by index.
- Each file is validated, sanitized, and moved individually.
- The script outputs one success line per uploaded file.

### Real-World Cases

- **User avatars:** upload profile pictures.
- **Document management:** upload PDFs, Word files, or spreadsheets.
- **E-commerce:** upload product images and receipts.
- **Social media:** upload photos and videos.
- **CMS:** upload media library assets.

---

## 5. PHP Form Data Access & Validation

### Definitions

**Core Definition**  
PHP provides superglobal arrays and filtering functions to read, validate, and sanitize form data received from GET and POST requests.

**Technical Definition**  
PHP automatically populates `$_GET`, `$_POST`, `$_FILES`, and `$_REQUEST` when a form is submitted. `filter_input()` and `filter_var()` apply validation or sanitization filters. `htmlspecialchars()` escapes output for HTML context. `filter_input_array()` and `filter_var_array()` apply filters to multiple values. `php://input` provides raw request body access for non-form content types.

**Beginner-Friendly Explanation**  
When a form is submitted, PHP puts the data into special arrays. `$_GET` holds URL parameters, `$_POST` holds form fields, and `$_FILES` holds uploaded file information. You should never trust this data directly. Use PHP’s filter functions to check email addresses, integers, URLs, and to remove dangerous characters. When you display user data back to the browser, use `htmlspecialchars()` so that HTML tags or JavaScript cannot be injected.

### Purposes

- To read form data reliably regardless of request method.
- To validate data types, ranges, formats, and presence.
- To sanitize data for safe storage or display.
- To avoid notices and warnings when fields are missing.
- To provide a consistent server-side validation layer.
- To prevent XSS, SQL injection, and other injection attacks.

### Syntax Rules and Structure

**General Syntax: Superglobals**
```php
$_GET['key']          // query-string value
$_POST['key']         // POST body value
$_FILES['key']        // uploaded file metadata
$_REQUEST['key']      // GET, POST, and COOKIE (order depends on configuration)
```

**Component Breakdown**

- `$_GET` — associative array of URL query parameters.
- `$_POST` — associative array of POST body parameters.
- `$_FILES` — associative array of uploaded file metadata.
- `$_REQUEST` — combined array; unsafe for state-changing operations because it may include cookies.

**General Syntax: `filter_input()`**
```php
$value = filter_input(
    INPUT_POST,                    // INPUT_GET, INPUT_POST, INPUT_COOKIE, INPUT_SERVER, INPUT_ENV
    'email',                       // variable name
    FILTER_VALIDATE_EMAIL,         // filter
    ['options' => ['default' => '']] // optional options/flags
);
```

**Component Breakdown**

- `INPUT_POST` — source superglobal.
- `'email'` — variable name.
- `FILTER_VALIDATE_EMAIL` — validation filter.
- Options array — default value, range, flags, etc.
- Returns the filtered value, `false` on filter failure, or `null` if the variable is not set.

**General Syntax: `filter_var()`**
```php
$clean = filter_var($raw, FILTER_SANITIZE_SPECIAL_CHARS);
$valid = filter_var($email, FILTER_VALIDATE_EMAIL);
```

**Component Breakdown**

- `$raw` — value to filter.
- `FILTER_SANITIZE_SPECIAL_CHARS` — sanitization filter.
- `FILTER_VALIDATE_EMAIL` — validation filter.
- Returns filtered value or `false` on failure.

**General Syntax: Output Escaping**
```php
echo htmlspecialchars($userInput, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8');
```

**Component Breakdown**

- `htmlspecialchars()` — converts `&`, `"`, `'`, `<`, `>` to HTML entities.
- `ENT_QUOTES` — escape both double and single quotes.
- `ENT_SUBSTITUTE` — replace invalid encoding sequences.
- `'UTF-8'` — character encoding.

**Syntax Rules**

- Treat all external data as untrusted.
- Validate on the server side; client-side validation is not sufficient.
- Use whitelists instead of blacklists when possible.
- Use `filter_input()` for raw superglobal values.
- Use `filter_var()` for values already in variables.
- Escape output for the correct context (HTML, attribute, URL, JavaScript, SQL).
- Use prepared statements for database queries.
- Use `password_hash()` and `password_verify()` for passwords.
- Do not use `$_REQUEST` for state-changing operations.
- Do not use `FILTER_SANITIZE_STRING` in PHP 8.1+; it is deprecated.
- Do not rely on `register_globals`; it was removed in PHP 5.4.

**Constraints and Limitations**

- `filter_input()` reads the original raw superglobal, not modifications made after startup.
- `filter_input()` returns `null` for missing variables and `false` for failed filters.
- `filter_var()` does not know whether a variable was missing or invalid.
- Sanitization filters do not make data safe for all contexts.
- Validation filters may reject valid international formats.
- `htmlspecialchars()` is not a substitute for context-specific escaping.
- `$_REQUEST` order depends on `request_order` and `variables_order`.

### Annotated Code Examples and Expected Outputs

#### Example 1: Validating Email and Integer with `filter_input()`

**File: `validate.php`**
```php
<?php
// Validate email from POST.
$email = filter_input(INPUT_POST, 'email', FILTER_VALIDATE_EMAIL);

// Validate age as an integer between 18 and 120.
$age = filter_input(INPUT_POST, 'age', FILTER_VALIDATE_INT, [
    'options' => [
        'default'   => 0,
        'min_range' => 18,
        'max_range' => 120
    ]
]);

if ($email === false) {
    echo 'Invalid email address.<br>';
} else {
    echo 'Email: ' . htmlspecialchars($email, ENT_QUOTES, 'UTF-8') . '<br>';
}

if ($age === false || $age === 0) {
    echo 'Age must be between 18 and 120.';
} else {
    echo 'Age: ' . (int)$age;
}
?>
```

**How to Run**

1. Save as `validate.php`.
2. Submit a POST request with `email=alice@example.com&age=30`.

**Expected Output**
```
Email: alice@example.com
Age: 30
```

**Why This Output Occurs**

- `FILTER_VALIDATE_EMAIL` accepts `alice@example.com`.
- `FILTER_VALIDATE_INT` with `min_range` and `max_range` accepts `30`.
- The default `0` is used if `age` is missing.
- Output is escaped and cast to integer.

#### Example 2: Sanitizing and Escaping Output

**File: `comments.php`**
```php
<?php
// Simulate POST data.
$_POST['comment'] = '<script>alert("XSS")</script>Hello & welcome!';

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    // Raw value.
    $raw = $_POST['comment'] ?? '';

    // Sanitize for general text storage (remove tags).
    $safeForStorage = strip_tags($raw);

    // Escape for HTML output.
    $safeForHtml = htmlspecialchars($safeForStorage, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8');

    echo 'Raw: ' . $raw . '<br>';
    echo 'Safe for HTML: ' . $safeForHtml;
}
?>
```

**Expected Output (rendered in browser)**
```
Raw: Hello & welcome!
Safe for HTML: Hello &amp; welcome!
```
(If you view source, the raw `<script>` tag would be visible as text, not executed, because `strip_tags()` removed it. In a real application, never echo raw input.)

**Why This Output Occurs**

- `strip_tags()` removes HTML/PHP tags from the string.
- `htmlspecialchars()` converts `&` to `&amp;`, `<` to `&lt;`, etc.
- The browser renders entities as literal characters, preventing XSS.
- `ENT_QUOTES` escapes both single and double quotes.
- `ENT_SUBSTITUTE` handles invalid encoding safely.

### Real-World Cases

- **Login forms:** validate email/username and verify password hashes.
- **Registration:** validate email, password strength, and age.
- **Contact forms:** sanitize message content and escape output in admin panels.
- **Search:** validate and whitelist sort/filter parameters.
- **Payment forms:** validate card numbers, expiry dates, and CVV server-side.

---

## 6. Post/Redirect/Get (PRG) Pattern

### Definitions

**Core Definition**  
Post/Redirect/Get (PRG) is a web development design pattern where a POST request is followed by a redirect (usually HTTP 303) to a GET request, preventing duplicate form submissions on refresh.

**Technical Definition**  
In PRG, the server receives a POST, processes the data, and responds with a `303 See Other` status and a `Location` header pointing to a GET-accessible resource. The browser follows the redirect with a GET request. This separates the state-changing POST from the result-rendering GET, so refreshing the result page repeats the harmless GET rather than the dangerous POST. It is commonly combined with session-based flash messages.

**Beginner-Friendly Explanation**  
Imagine you buy a movie ticket online. You fill out a form and click “Buy.” The server charges your card, then tells your browser, “Go to the confirmation page.” Your browser then does a normal GET request for that page. If you hit refresh on the confirmation page, you just reload the confirmation — you do not buy another ticket. Without PRG, refreshing the POST result could charge you twice.

### Purposes

- To prevent accidental duplicate database inserts on refresh.
- To avoid browser “Confirm form resubmission” warnings.
- To make the result page bookmarkable and shareable via GET.
- To improve back/forward navigation behavior.
- To combine with flash messages for one-time success/error notifications.
- To align with HTTP semantics: POST for state change, GET for retrieval.

### Syntax Rules and Structure

**General Syntax: PRG Flow**
```
Client: POST /submit.php
Server: process data
Server: 303 See Other
        Location: /success.php
Client: GET /success.php
Server: 200 OK (success page)
```

**Component Breakdown**

- `POST /submit.php` — state-changing request.
- Server processing — validate, save, update, delete.
- `303 See Other` — status code that instructs the client to switch to GET.
- `Location: /success.php` — target of the redirect.
- `GET /success.php` — safe retrieval request.
- `200 OK` — final response.

**General Syntax: PHP PRG Implementation**
```php
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    // Process data.
    // ...
    // Redirect with 303.
    header('Location: success.php', true, 303);
    exit;
}
// GET request: display page.
```

**Component Breakdown**

- `$_SERVER['REQUEST_METHOD'] === 'POST'` — detects POST.
- Processing block — validation, database work, session updates.
- `header('Location: ...', true, 303)` — sends 303 and Location header.
- `exit` — stops script execution after redirect.
- GET branch — renders the page normally.

**General Syntax: Flash Message with PRG**
```php
session_start();
$_SESSION['flash'] = 'Saved successfully!';
header('Location: result.php', true, 303);
exit;
```
```php
session_start();
$message = $_SESSION['flash'] ?? '';
unset($_SESSION['flash']);
echo htmlspecialchars($message, ENT_QUOTES, 'UTF-8');
```

**Component Breakdown**

- `session_start()` — starts or resumes session.
- `$_SESSION['flash']` — stores one-time message.
- `unset()` — removes message after reading.
- `htmlspecialchars()` — escapes output.

**Syntax Rules**

- Use `303 See Other` for PRG after POST.
- Do not use `302 Found` for PRG if you want strict method conversion; `303` is the correct choice.
- Do not use `301 Moved Permanently` for PRG; browsers may not convert POST to GET.
- Call `exit` or `die` after `header('Location: ...')`.
- Do not output anything before `header()`.
- Use sessions or query parameters for flash messages.
- Validate and process data before redirecting.
- Redirect only after successful processing, or redirect back to the form with errors.

**Constraints and Limitations**

- PRG adds an extra round trip; this is usually negligible.
- Session-based flash messages require session support and cookies.
- Query-string flash messages are visible and can be replayed; sessions are preferred.
- PRG does not prevent duplicate submissions caused by double-clicking before the first response; use CSRF tokens, disabled buttons, or idempotency keys for that.
- `header()` must be called before any output.
- `exit` is essential to prevent further execution.

### Annotated Code Examples and Expected Outputs

#### Example 1: Minimal PRG

**File: `form.php`**
```html
<form method="post" action="submit.php">
  <input type="text" name="name" required>
  <button type="submit">Submit</button>
</form>
```

**File: `submit.php`**
```php
<?php
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $name = trim($_POST['name'] ?? '');
    if ($name !== '') {
        // Simulate saving to a database.
        // ...

        // Redirect to a GET page.
        header('Location: success.php', true, 303);
        exit;
    }
}
// If not POST or invalid, redirect back.
header('Location: form.php', true, 303);
exit;
?>
```

**File: `success.php`**
```php
<?php
echo 'Thank you! Your submission was received.';
?>
```

**How to Run**

1. Save all three files.
2. Open `form.php`, enter a name, and submit.
3. The browser sends POST to `submit.php`.
4. `submit.php` responds with `303` and `Location: success.php`.
5. The browser performs a GET for `success.php`.

**Expected Output**
```
Thank you! Your submission was received.
```

**Why This Output Occurs**

- The POST is processed and then a 303 redirect is sent.
- The browser converts the request to GET for the redirect target.
- Refreshing `success.php` repeats only the GET, not the POST.
- No duplicate database insert occurs.

#### Example 2: PRG with Session Flash Message and Validation Errors

**File: `newsletter.php`**
```php
<?php
session_start();

// Read and clear flash data.
$errors = $_SESSION['errors'] ?? [];
$old = $_SESSION['old'] ?? [];
unset($_SESSION['errors'], $_SESSION['old']);
?>
<!DOCTYPE html>
<html>
<body>
<?php if (isset($_SESSION['success'])): ?>
    <p style="color:green;"><?= htmlspecialchars($_SESSION['success'], ENT_QUOTES, 'UTF-8') ?></p>
    <?php unset($_SESSION['success']); ?>
<?php endif; ?>

<?php foreach ($errors as $error): ?>
    <p style="color:red;"><?= htmlspecialchars($error, ENT_QUOTES, 'UTF-8') ?></p>
<?php endforeach; ?>

<form method="post" action="newsletter_submit.php">
  <input type="email" name="email" value="<?= htmlspecialchars($old['email'] ?? '', ENT_QUOTES, 'UTF-8') ?>" placeholder="Email">
  <button type="submit">Subscribe</button>
</form>
</body>
</html>
```

**File: `newsletter_submit.php`**
```php
<?php
session_start();

$email = filter_input(INPUT_POST, 'email', FILTER_VALIDATE_EMAIL);

if (!$email) {
    $_SESSION['errors'] = ['Please enter a valid email address.'];
    $_SESSION['old']['email'] = $_POST['email'] ?? '';
    header('Location: newsletter.php', true, 303);
    exit;
}

// Simulate saving to database.
// ...

$_SESSION['success'] = 'Subscribed successfully!';
header('Location: newsletter.php', true, 303);
exit;
?>
```

**Expected Output (invalid email)**
```
Please enter a valid email address.
```
The form is redisplayed with the previously entered email. On refresh, the error remains only for one request because it is unset after display.

**Expected Output (valid email)**
```
Subscribed successfully!
```

**Why This Output Occurs**

- `newsletter_submit.php` validates the email.
- On failure, it stores errors and old input in the session, then redirects back with 303.
- On success, it stores a flash success message and redirects with 303.
- `newsletter.php` reads and clears flash data, so refreshing does not repeat the message.
- The browser’s final request is a GET, so refresh does not resubmit the POST.

### Real-World Cases

- **User registration:** POST creates account, redirect to welcome page.
- **E-commerce checkout:** POST places order, redirect to confirmation page.
- **Comment posting:** POST saves comment, redirect to article page.
- **Newsletter subscription:** POST subscribes, redirect to thank-you page.
- **Admin CRUD:** POST creates/updates record, redirect to list page.

---

## Security Considerations

### CSRF (Cross-Site Request Forgery)

- **Definition:** An attack that tricks an authenticated user’s browser into sending an unwanted state-changing request to a trusted site.
- **Mitigation:** Add unpredictable CSRF tokens to all state-changing forms and validate them on the server. Use the synchronizer token pattern for stateful applications. Use `SameSite` cookies as defense in depth. Do not use GET for state-changing operations.
- **References:** OWASP CSRF Prevention Cheat Sheet .

### XSS (Cross-Site Scripting)

- **Definition:** Injection of malicious scripts into pages viewed by other users.
- **Mitigation:** Escape all user-controlled output with `htmlspecialchars()` for HTML context. Use context-aware escaping for attributes, JavaScript, CSS, and URLs. Validate input on the server side. Use Content Security Policy (CSP) as defense in depth.
- **Reference:** PHP `htmlspecialchars()` manual .

### File Upload Security

- Check `$_FILES['field']['error']`.
- Validate file size and extension against a whitelist.
- Verify MIME type server-side using `finfo_file()`, not the browser-supplied type.
- Generate a safe filename; use `basename()` and replace unsafe characters.
- Store uploads outside the web root when possible.
- Use `move_uploaded_file()` to move the file.
- Never execute uploaded files.
- Limit `upload_max_filesize`, `post_max_size`, and `max_file_uploads`.
- References: PHP file upload manual , `move_uploaded_file()` , `is_uploaded_file()` .

### Deprecated, Unsafe, or Version-Specific Features

- `register_globals` — removed in PHP 5.4; never use.
- `$_REQUEST` — unsafe for state-changing operations because it may include cookies.
- `FILTER_SANITIZE_STRING` — deprecated in PHP 8.1; use `FILTER_SANITIZE_SPECIAL_CHARS` or `htmlspecialchars()` instead.
- `MAX_FILE_SIZE` — client-side convenience only; server-side limits are authoritative.
- `$_FILES['full_path']` — available as of PHP 8.1.0; untrusted.
- `session_start()` options — PHP 8.5.0 throws `ValueError` for non-string keys and `TypeError` for incompatible `read_and_close`.
- `header('Location: ...')` without an explicit status code — PHP may default to `302 Found`; for PRG, explicitly use `303 See Other`.

---

## References

- PHP: Variables From External Sources — https://www.php.net/manual/en/language.variables.external.php
- PHP: `$_POST` — https://www.php.net/manual/en/reserved.variables.post.php
- PHP: `$_GET` — https://www.php.net/manual/en/reserved.variables.get.php
- PHP: `$_REQUEST` — https://www.php.net/manual/en/reserved.variables.request.php
- PHP: `$_FILES` — https://www.php.net/manual/en/reserved.variables.files.php
- PHP: POST Method Uploads — https://www.php.net/manual/en/features.file-upload.post-method.php
- PHP: `move_uploaded_file()` — https://www.php.net/manual/en/function.move-uploaded-file.php
- PHP: `is_uploaded_file()` — https://www.php.net/manual/en/function.is-uploaded-file.php
- PHP: `filter_input()` — https://www.php.net/manual/en/function.filter-input.php
- PHP: `htmlspecialchars()` — https://www.php.net/manual/en/function.htmlspecialchars.php
- PHP: `session_start()` — https://www.php.net/manual/en/function.session-start.php
- PHP: PHP and HTML FAQ (form arrays) — https://www.php.net/manual/en/faq.html.php#faq.html.arrays
- MDN: HTTP Request Methods — https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods
- MDN: GET Request Method — https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods/GET
- MDN: POST Request Method — https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods/POST
- MDN: 303 See Other — https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status/303
- MDN: `HTMLFormElement.enctype` — https://developer.mozilla.org/en-US/docs/Web/API/HTMLFormElement/enctype
- RFC 9110: HTTP Semantics — https://datatracker.ietf.org/doc/rfc9110/
- RFC 7578: Returning Values from Forms: multipart/form-data — https://datatracker.ietf.org/doc/html/rfc7578
- WHATWG HTML Living Standard: Form Control Infrastructure — https://html.spec.whatwg.org/multipage/form-control-infrastructure.html
- OWASP: Cross-Site Request Forgery Prevention Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html
- Wikipedia: Post/Redirect/Get — https://en.wikipedia.org/wiki/Post/Redirect/Get