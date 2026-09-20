# PHP HTTP Superglobals — Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**  
PHP HTTP Superglobals are built-in, globally accessible associative arrays that PHP automatically populates with data from the HTTP request, server environment, cookies, sessions, and system environment. They are available in every scope without requiring `global` or `use` declarations.

**Technical Definition**  
Superglobals are predeclared variables in the PHP runtime: `$GLOBALS`, `$_SERVER`, `$_GET`, `$_POST`, `$_FILES`, `$_COOKIE`, `$_SESSION`, `$_REQUEST`, and `$_ENV`. They are populated during PHP's request startup phase by the SAPI (Server API) layer — typically by `php_request_startup()` — and are accessible from any scope, including inside functions, classes, and closures, without the `global` keyword. They are defined by the PHP language specification and documented in the PHP manual under "Predefined Variables."

**Beginner-Friendly Explanation**  
When someone visits a PHP page, the web server hands PHP a big bundle of information: the URL parameters, form data, uploaded files, cookies, headers, the visitor's IP address, and the server's environment settings. PHP sorts this bundle into a set of special arrays whose names start with `$_` (like `$_GET`, `$_POST`, `$_SERVER`). These arrays are called "superglobals" because they are visible everywhere in your script — inside functions, classes, or included files — without any extra work. They are the primary way PHP receives input from the outside world and reads configuration from the server.

---

### Key Characteristics

- Automatically populated by PHP at request startup; no manual initialization is required.
- Accessible from any scope (global, function, method, closure) without `global`.
- Names are case-sensitive: `$_GET` is not the same as `$_get`.
- All values are user-controlled or environment-controlled unless explicitly validated; they must be treated as untrusted input.
- `$_GET`, `$_POST`, `$_COOKIE`, and `$_REQUEST` are affected by PHP configuration directives (`variables_order`, `request_order`).
- `$_SESSION` requires `session_start()` before use.
- `$_ENV` depends on the `variables_order` directive and the `E` flag.
- `$_FILES` is only populated for `multipart/form-data` POST requests.
- `$_REQUEST` merges GET, POST, and COOKIE data in an order determined by `request_order`; it is widely considered bad practice for state-changing logic.
- Superglobals are not available in CLI scripts in the same way as in web SAPIs; `$_GET`, `$_POST`, and `$_COOKIE` are typically empty in CLI.
- Modifying a superglobal at runtime affects only the current request; it does not persist to the next request.

---

### Prerequisites

- Basic PHP syntax: variables, arrays, functions, scope.
- Basic HTTP: requests, responses, headers, methods, status codes.
- Basic HTML forms: `method`, `action`, `enctype`, `name`.
- A web server with PHP installed (Apache, Nginx + PHP-FPM, or the PHP built-in server).
- Basic understanding of cookies and sessions.

---

### Related Programming Areas

- HTTP protocol and request/response semantics.
- Web application security (XSS, CSRF, SQL injection, session fixation).
- PHP configuration (`php.ini`) and SAPI behavior.
- Session management and authentication.
- File upload handling and filesystem operations.
- Environment configuration and containerized deployments (Docker, Kubernetes).
- Framework request abstractions (Laravel `Request`, Symfony `Request`).

---

## Core Concepts / Features

## 1. `$_GET` — URL Query String Parameters

### Definitions

**Core Definition**  
`$_GET` is an associative array of variables passed to the current script via the URL query string.

**Technical Definition**  
`$_GET` is populated by PHP's SAPI layer from the query component of the request target (the part after `?` in the URL). It is decoded according to the URL-encoding rules defined in RFC 3986 and the HTML form submission algorithm. It is populated regardless of the HTTP method — a POST request with a query string will also populate `$_GET`. Keys are strings; values are strings or arrays of strings. It is affected by the `variables_order` directive: if `G` is not present, `$_GET` is not populated.

**Beginner-Friendly Explanation**  
If you visit `page.php?name=Alice&age=30`, PHP puts `Alice` into `$_GET['name']` and `30` into `$_GET['age']`. You can read these values in your script to customize the page — for example, to show search results or filter a product list. Because the data is in the URL, it is visible to anyone who sees the link.

### Purposes

- To retrieve small, non-sensitive parameters from the URL.
- To make pages bookmarkable, shareable, and cacheable.
- To implement search, filtering, pagination, and sorting.
- To pass optional configuration flags (e.g., `?debug=1`).
- To read query parameters without parsing `$_SERVER['QUERY_STRING']` manually.

### Syntax Rules and Structure

**General Syntax: Accessing `$_GET`**
```php
$value = $_GET['key'];                       // direct access
$value = $_GET['key'] ?? 'default';          // safe with default
$value = filter_input(INPUT_GET, 'key', FILTER_SANITIZE_SPECIAL_CHARS);
```

**Component Breakdown**

- `$_GET['key']` — reads the value associated with `key`; triggers a warning if missing.
- `?? 'default'` — null-coalescing operator returns `default` if the key is absent or null.
- `filter_input(INPUT_GET, 'key', FILTER_...)` — reads and filters a single GET variable.

**General Syntax: Array Parameters**
```php
// URL: page.php?items[]=a&items[]=b&items[]=c
$items = $_GET['items'] ?? [];   // ['a', 'b', 'c']
```

**Component Breakdown**

- `items[]` — array syntax in the query string.
- PHP builds `$_GET['items']` as an indexed array.

**General Syntax: Nested Parameters**
```php
// URL: page.php?filter[color]=red&filter[size]=large
$filter = $_GET['filter'] ?? [];
echo $filter['color']; // red
echo $filter['size'];  // large
```

**Component Breakdown**

- `filter[color]` — nested array syntax.
- PHP builds `$_GET['filter']['color']`.

**Syntax Rules**

- Keys and values are strings; numeric-looking strings are not automatically cast to integers in `$_GET`.
- Dots (`.`) and spaces in parameter names are converted to underscores (`_`) by PHP.
- Duplicate keys without array syntax keep only the last value.
- Array syntax (`[]`) creates an array; explicit keys (`[key]`) create associative arrays.
- Deeply nested arrays are limited by `max_input_nesting_level`.
- The total number of input variables is limited by `max_input_vars`.
- `$_GET` is always populated from the URL query string, even for POST requests.

**Constraints and Limitations**

- URL length is limited by browsers, proxies, and servers (commonly ~2,000–8,000 characters).
- Data is visible in the URL, browser history, referrer headers, and server logs.
- Do not use `GET` for passwords, tokens, or state-changing operations.
- Values are user-controlled; always validate and escape.
- `$_GET` is empty in CLI SAPI.
- `variables_order` must include `G` for `$_GET` to be populated.

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic GET Parameter

**File: `greet.php`**
```php
<?php
// Read 'name' from the query string; default to 'Guest'.
$name = $_GET['name'] ?? 'Guest';

// Escape for HTML context before output.
$safeName = htmlspecialchars($name, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8');

echo "Hello, $safeName!";
?>
```

**How to Run**

1. Save as `greet.php` in your web root.
2. Visit `greet.php?name=Alice`.

**Expected Output**
```
Hello, Alice!
```

**Why This Output Occurs**

- The query string `name=Alice` populates `$_GET['name']`.
- `htmlspecialchars()` escapes any HTML-special characters.
- The script echoes the escaped value.

#### Example 2: Array and Nested Parameters

**File: `shop.php`**
```php
<?php
// URL: shop.php?filter[color]=red&filter[size]=large&tags[]=sale&tags[]=new
$filter = $_GET['filter'] ?? [];
$tags   = $_GET['tags'] ?? [];

$color = htmlspecialchars($filter['color'] ?? 'any', ENT_QUOTES, 'UTF-8');
$size  = htmlspecialchars($filter['size'] ?? 'any', ENT_QUOTES, 'UTF-8');

echo "Color: $color<br>";
echo "Size: $size<br>";
echo "Tags: " . implode(', ', array_map(
    fn($t) => htmlspecialchars($t, ENT_QUOTES, 'UTF-8'),
    $tags
));
?>
```

**Expected Output**
```
Color: red
Size: large
Tags: sale, new
```

**Why This Output Occurs**

- `filter[color]=red` becomes `$_GET['filter']['color'] = 'red'`.
- `tags[]=sale&tags[]=new` becomes `$_GET['tags'] = ['sale', 'new']`.
- Each value is escaped before output.

### Real-World Cases

- **Search:** `search.php?q=keyboard&category=electronics`.
- **Pagination:** `articles.php?page=3&per_page=20`.
- **Filtering:** `products.php?color=red&size=large`.
- **Sorting:** `list.php?sort=price&order=asc`.
- **Debug flags:** `page.php?debug=1`.

---

## 2. `$_POST` — HTTP POST Body Parameters

### Definitions

**Core Definition**  
`$_POST` is an associative array of variables passed to the current script via the HTTP POST method when the request uses `application/x-www-form-urlencoded` or `multipart/form-data` encoding.

**Technical Definition**  
`$_POST` is populated by PHP's SAPI layer from the request body for POST requests with content types `application/x-www-form-urlencoded` or `multipart/form-data`. For other content types (e.g., `application/json`), `$_POST` is empty and the raw body must be read from `php://input`. It is affected by the `variables_order` directive: if `P` is not present, `$_POST` is not populated. `post_max_size` limits the total size of the POST body.

**Beginner-Friendly Explanation**  
When you fill out a form and click "Submit" using `method="post"`, the browser sends your answers in the body of the request rather than the URL. PHP puts those answers into `$_POST`. This is the standard way to handle logins, registrations, comments, and any action that changes data on the server.

### Purposes

- To receive form submissions that change server state.
- To handle larger payloads than GET can carry.
- To accept file uploads (alongside `$_FILES`).
- To hide data from the URL and browser history.
- To avoid URL length limits.
- To distinguish state-changing requests from safe reads.

### Syntax Rules and Structure

**General Syntax: Accessing `$_POST`**
```php
$value = $_POST['key'];
$value = $_POST['key'] ?? '';
$value = filter_input(INPUT_POST, 'key', FILTER_VALIDATE_EMAIL);
```

**Component Breakdown**

- `$_POST['key']` — reads a POST field; warning if missing.
- `?? ''` — provides a default.
- `filter_input(INPUT_POST, ...)` — reads and filters a POST variable.

**General Syntax: Array Fields**
```html
<input type="checkbox" name="hobbies[]" value="reading">
<input type="checkbox" name="hobbies[]" value="coding">
```
```php
$hobbies = $_POST['hobbies'] ?? []; // ['reading', 'coding']
```

**Component Breakdown**

- `name="hobbies[]"` — array syntax.
- PHP creates `$_POST['hobbies']` as an indexed array.

**General Syntax: Nested Fields**
```html
<input type="text" name="items[0][product_id]" value="P100">
<input type="number" name="items[0][qty]" value="2">
```
```php
$items = $_POST['items'] ?? [];
// $items[0]['product_id'] === 'P100'
```

**Component Breakdown**

- `items[0][product_id]` — nested array syntax.
- PHP builds a multi-dimensional array.

**General Syntax: Detecting POST**
```php
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    // process POST
}
```

**Component Breakdown**

- `$_SERVER['REQUEST_METHOD']` — reliable method detection.
- Avoid relying on `isset($_POST)` alone; a POST with an empty body still populates `$_POST` as an empty array.

**Syntax Rules**

- `$_POST` is populated only for `application/x-www-form-urlencoded` and `multipart/form-data`.
- For `application/json`, read `file_get_contents('php://input')`.
- Keys and values are strings unless array syntax is used.
- `post_max_size` limits the total POST body; exceeding it may empty `$_POST` and `$_FILES`.
- `max_input_vars` limits the number of fields.
- `max_input_nesting_level` limits array nesting depth.
- Dots and spaces in field names become underscores.
- `$_POST` is empty in CLI SAPI.

**Constraints and Limitations**

- Do not assume `$_POST` is set just because the method is POST.
- `$_POST` does not include unchecked checkboxes or unselected radios.
- `$_POST` does not include files; use `$_FILES`.
- Values are untrusted; always validate and escape.
- `$_POST` is not populated for PUT, PATCH, or DELETE requests.
- `post_max_size` must be larger than `upload_max_filesize` for uploads to work.

### Annotated Code Examples and Expected Outputs

#### Example 1: Login Form Processing

**File: `login.php`**
```php
<?php
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $username = trim($_POST['username'] ?? '');
    $password = $_POST['password'] ?? '';

    // Validate presence.
    if ($username === '' || $password === '') {
        echo 'Username and password are required.';
        exit;
    }

    // In a real app: fetch hash from DB and use password_verify().
    $storedHash = '$2y$10$examplehash...';

    if (password_verify($password, $storedHash)) {
        echo 'Welcome, ' . htmlspecialchars($username, ENT_QUOTES, 'UTF-8') . '!';
    } else {
        echo 'Invalid credentials.';
    }
}
?>
<form method="post" action="login.php">
  <input type="text" name="username" placeholder="Username">
  <input type="password" name="password" placeholder="Password">
  <button type="submit">Log in</button>
</form>
```

**Expected Output (valid credentials)**
```
Welcome, alice!
```

**Why This Output Occurs**

- `$_POST['username']` and `$_POST['password']` contain the submitted values.
- `password_verify()` compares the plain password with the stored hash.
- The username is escaped before output.

#### Example 2: Nested Array from Form

**File: `order.php`**
```php
<?php
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $items = $_POST['items'] ?? [];
    foreach ($items as $i => $item) {
        $id  = htmlspecialchars($item['product_id'] ?? '', ENT_QUOTES, 'UTF-8');
        $qty = (int)($item['qty'] ?? 0);
        echo "Item $i: $id x $qty<br>";
    }
}
?>
<form method="post" action="order.php">
  <input type="text" name="items[0][product_id]" value="P100">
  <input type="number" name="items[0][qty]" value="2">
  <input type="text" name="items[1][product_id]" value="P200">
  <input type="number" name="items[1][qty]" value="1">
  <button type="submit">Submit</button>
</form>
```

**Expected Output**
```
Item 0: P100 x 2
Item 1: P200 x 1
```

**Why This Output Occurs**

- The nested field names create a multi-dimensional `$_POST['items']` array.
- The loop reads each item and casts `qty` to an integer.
- Output is escaped and cast.

### Real-World Cases

- **Login/registration:** POST credentials and profile data.
- **Checkout:** POST cart items, shipping, and payment.
- **Comment posting:** POST comment text and metadata.
- **Settings update:** POST user preferences.
- **Admin CRUD:** POST create/update/delete operations.

---

## 3. `$_REQUEST` — Combined Input Array (and Why It Is Bad Practice)

### Definitions

**Core Definition**  
`$_REQUEST` is an associative array that merges the contents of `$_GET`, `$_POST`, and `$_COOKIE` (and sometimes `$_SERVER` in CLI) according to the `request_order` or `variables_order` directive.

**Technical Definition**  
`$_REQUEST` is populated by PHP by merging the enabled superglobals in the order specified by `request_order` (default `"GP"` in many distributions, meaning GET then POST, with POST overriding GET). The default `variables_order` is `"EGPCS"`. Because the merge order and inclusion of cookies depend on configuration, `$_REQUEST` is non-deterministic across environments. It is also a common source of security vulnerabilities because an attacker may inject a parameter via GET that overrides a POST value or vice versa.

**Beginner-Friendly Explanation**  
`$_REQUEST` is a "catch-all" array that combines URL parameters, form data, and cookies into one place. At first it seems convenient — you do not have to remember whether the data came from GET or POST. But this convenience is dangerous: an attacker can add a query-string parameter that overrides a form field, or a cookie that overrides both. Because you cannot tell where the value came from, you cannot validate it properly. Best practice is to use `$_GET` and `$_POST` explicitly.

### Purposes

- To read input without distinguishing between GET and POST (discouraged).
- To support legacy code that predates strict method handling.
- To quickly prototype simple scripts (not for production).
- To provide a fallback when the method is not known in advance (rare).
- To illustrate why explicit source selection is safer (educational).

### Syntax Rules and Structure

**General Syntax: Accessing `$_REQUEST`**
```php
$value = $_REQUEST['key'];           // DISCouraged
$value = $_REQUEST['key'] ?? '';
```

**Component Breakdown**

- `$_REQUEST['key']` — returns the value from GET, POST, or COOKIE depending on configuration.
- The source is ambiguous; you cannot tell whether it came from the URL, body, or cookie.

**General Syntax: Configuration Directives**
```ini
; php.ini
variables_order = "EGPCS"   ; E=ENV, G=GET, P=POST, C=COOKIE, S=SERVER
request_order   = "GP"      ; GET then POST; POST overrides GET
```

**Component Breakdown**

- `variables_order` — controls which superglobals are populated.
- `request_order` — controls the merge order for `$_REQUEST`.
- If `request_order` is empty, `variables_order` is used.

**Syntax Rules**

- `$_REQUEST` is populated only if `request_order` or `variables_order` includes `G`, `P`, or `C`.
- The default merge order is implementation-dependent and configurable.
- Later sources override earlier ones; e.g., with `"GP"`, POST overrides GET.
- `$_REQUEST` does not include `$_FILES` or `$_SESSION`.
- `$_REQUEST` is empty in CLI SAPI by default.

**Constraints and Limitations**

- **Ambiguity:** You cannot tell whether a value came from GET, POST, or COOKIE.
- **Override attacks:** An attacker can supply a GET parameter that overrides a POST value, or a cookie that overrides both.
- **Cookie pollution:** If cookies are included, a malicious cookie can inject values.
- **CSRF risk:** State-changing operations using `$_REQUEST` may accept GET requests, enabling CSRF via image tags or links.
- **Caching risk:** GET requests are cacheable; state-changing operations should not be cacheable.
- **Configuration drift:** The behavior differs across `php.ini` configurations, making code non-portable.
- **Debugging difficulty:** Tracing the source of a value is harder.
- **Security audits:** Static analysis tools flag `$_REQUEST` as a risk.

### Annotated Code Examples and Expected Outputs

#### Example 1: Demonstrating the Override Problem

**File: `transfer.php`**
```php
<?php
// DANGEROUS: using $_REQUEST allows GET to override POST.
$amount = $_REQUEST['amount'] ?? 0;
$to     = $_REQUEST['to'] ?? '';

// In a real app, this would transfer money.
echo "Transferring $amount to $to";
?>
```

**Attack Scenario**

An attacker crafts a link:
```
https://bank.example/transfer.php?amount=10000&to=attacker
```
If the victim clicks it while logged in, the GET parameters may be used instead of the intended POST body.

**Expected Output (attack)**
```
Transferring 10000 to attacker
```

**Why This Output Occurs**

- `$_REQUEST` merges GET and POST; with default `request_order="GP"`, POST overrides GET, but if the request is a GET (no POST body), the GET values are used.
- The script does not verify the request method or the source of the values.
- A state-changing operation is performed via a GET request, enabling CSRF.

#### Example 2: Safe Alternative — Explicit Source

**File: `transfer_safe.php`**
```php
<?php
// SAFE: only accept POST, use $_POST explicitly.
if ($_SERVER['REQUEST_METHOD'] !== 'POST') {
    http_response_code(405);
    echo 'Method Not Allowed';
    exit;
}

// CSRF token check would go here.
$amount = filter_input(INPUT_POST, 'amount', FILTER_VALIDATE_FLOAT);
$to     = filter_input(INPUT_POST, 'to', FILTER_SANITIZE_SPECIAL_CHARS);

if ($amount === false || $amount <= 0 || !$to) {
    echo 'Invalid input.';
    exit;
}

echo "Transferring $amount to " . htmlspecialchars($to, ENT_QUOTES, 'UTF-8');
?>
```

**Expected Output (valid POST)**
```
Transferring 100.5 to alice
```

**Why This Output Occurs**

- The script rejects non-POST requests with `405 Method Not Allowed`.
- `filter_input(INPUT_POST, ...)` reads only POST data, eliminating GET/cookie override.
- Values are validated and escaped.
- State-changing logic is protected from CSRF via method enforcement and (in a real app) a CSRF token.

### Real-World Cases

- **Legacy code migration:** Older PHP applications often use `$_REQUEST`; refactoring to `$_GET`/`$_POST` improves security.
- **Security audits:** `$_REQUEST` is a common finding in penetration tests.
- **Framework migration:** Modern frameworks inject a `Request` object with explicit `query`, `request`, and `cookies` bags, avoiding the ambiguity.
- **CSRF protection:** Using `$_REQUEST` for state-changing operations is a known CSRF vector.
- **Cookie injection:** If cookies are included, an attacker who can set a cookie may influence request handling.

---

## 4. `$_SERVER` — Server and Execution Environment

### Definitions

**Core Definition**  
`$_SERVER` is an associative array containing information about the server, the execution environment, request headers, paths, script locations, and routing metadata.

**Technical Definition**  
`$_SERVER` is populated by the web server and PHP's SAPI layer. It contains CGI-style variables (`REQUEST_METHOD`, `QUERY_STRING`, `SCRIPT_NAME`, `PATH_INFO`, `REMOTE_ADDR`, `REMOTE_PORT`, `SERVER_ADDR`, `SERVER_PORT`, `SERVER_NAME`, `HTTP_*` headers, etc.). The exact contents depend on the SAPI (Apache, Nginx + PHP-FPM, CLI) and the web server configuration. It is affected by `variables_order`: if `S` is not present, `$_SERVER` may not be fully populated (though PHP always sets some entries).

**Beginner-Friendly Explanation**  
`$_SERVER` is like the server's "ID card" for the current request. It tells you the request method (GET or POST), the URL path, the query string, the visitor's IP address, the server's IP address, and the HTTP headers the browser sent. It is essential for routing, logging, security checks, and building URLs.

### Purposes

- To detect the HTTP request method (`REQUEST_METHOD`).
- To read request headers (`HTTP_*`).
- To identify the client IP address (`REMOTE_ADDR`) for logging or rate limiting.
- To determine the script path and name (`SCRIPT_NAME`, `SCRIPT_FILENAME`, `PHP_SELF`).
- To build absolute URLs (`HTTPS`, `HTTP_HOST`, `SERVER_PORT`).
- To implement routing based on `PATH_INFO` or `REQUEST_URI`.
- To detect HTTPS (`HTTPS`, `SERVER_PORT`).
- To read server software and protocol (`SERVER_SOFTWARE`, `SERVER_PROTOCOL`).
- To access the raw query string (`QUERY_STRING`).

### Syntax Rules and Structure

**General Syntax: Accessing `$_SERVER`**
```php
$method = $_SERVER['REQUEST_METHOD'] ?? 'GET';
$ip     = $_SERVER['REMOTE_ADDR'] ?? '0.0.0.0';
$host   = $_SERVER['HTTP_HOST'] ?? 'localhost';
```

**Component Breakdown**

- `$_SERVER['REQUEST_METHOD']` — HTTP method (GET, POST, PUT, etc.).
- `$_SERVER['REMOTE_ADDR']` — client IP address as seen by the server.
- `$_SERVER['HTTP_HOST']` — value of the `Host` header.

**General Syntax: Common Keys**
```php
$_SERVER['REQUEST_METHOD']   // 'GET', 'POST', ...
$_SERVER['REQUEST_URI']      // '/path?query=1'
$_SERVER['QUERY_STRING']     // 'query=1'
$_SERVER['SCRIPT_NAME']      // '/index.php'
$_SERVER['SCRIPT_FILENAME']  // '/var/www/index.php'
$_SERVER['PHP_SELF']         // '/index.php' (unsafe if echoed unescaped)
$_SERVER['PATH_INFO']        // '/extra/path'
$_SERVER['DOCUMENT_ROOT']    // '/var/www/html'
$_SERVER['SERVER_NAME']      // 'example.com'
$_SERVER['SERVER_ADDR']      // '192.0.2.1'
$_SERVER['SERVER_PORT']      // '443'
$_SERVER['REMOTE_ADDR']      // '203.0.113.5'
$_SERVER['REMOTE_PORT']      // '54321'
$_SERVER['HTTPS']            // 'on' or unset
$_SERVER['HTTP_USER_AGENT']  // browser user agent
$_SERVER['HTTP_ACCEPT']      // accepted media types
$_SERVER['HTTP_REFERER']     // referring page (untrusted)
```

**Component Breakdown**

- `REQUEST_METHOD` — method of the current request.
- `REQUEST_URI` — full request target including query string.
- `QUERY_STRING` — raw query string without `?`.
- `SCRIPT_NAME` — path to the current script.
- `SCRIPT_FILENAME` — absolute filesystem path to the script.
- `PHP_SELF` — path of the current script; can contain user input if PATH_INFO is used; escape before output.
- `PATH_INFO` — extra path after the script name.
- `DOCUMENT_ROOT` — web root directory.
- `SERVER_NAME` — configured server name.
- `SERVER_ADDR` — server IP address.
- `SERVER_PORT` — server port.
- `REMOTE_ADDR` — client IP address (not spoofable at the TCP level, but may be a proxy).
- `REMOTE_PORT` — client port.
- `HTTPS` — set to `'on'` if the request is over HTTPS; may be unset otherwise.
- `HTTP_*` — request headers; e.g., `HTTP_USER_AGENT`, `HTTP_ACCEPT`, `HTTP_REFERER`.

**General Syntax: Reading Request Headers Safely**
```php
$headers = function_exists('getallheaders') ? getallheaders() : [];
$userAgent = $_SERVER['HTTP_USER_AGENT'] ?? '';
```

**Component Breakdown**

- `getallheaders()` — available in Apache and some SAPIs; returns all request headers.
- `$_SERVER['HTTP_*']` — always available for common headers.

**Syntax Rules**

- Keys are case-sensitive and uppercase.
- HTTP headers are prefixed with `HTTP_` and have dashes replaced with underscores (`User-Agent` → `HTTP_USER_AGENT`).
- `CONTENT_TYPE` and `CONTENT_LENGTH` do not have the `HTTP_` prefix.
- `REMOTE_ADDR` is the direct connection IP; behind a proxy, use `X-Forwarded-For` only if trusted.
- `HTTP_HOST` is user-controlled (from the `Host` header); validate against a whitelist before using in URLs.
- `PHP_SELF` can contain user input; escape before output to prevent XSS.
- `HTTPS` may be set to `'off'` or unset; check both.

**Constraints and Limitations**

- `$_SERVER` contents vary by SAPI; CLI has different keys.
- `REMOTE_ADDR` may be a proxy or load balancer IP.
- `HTTP_X_FORWARDED_FOR` and similar headers are user-controlled unless set by a trusted proxy.
- `HTTP_HOST` is user-controlled; never trust it for security decisions.
- `HTTP_REFERER` is optional and can be spoofed; never rely on it for security.
- `PHP_SELF` can contain path traversal or XSS payloads if used with PATH_INFO.
- `variables_order` must include `S` for full `$_SERVER` population.

### Annotated Code Examples and Expected Outputs

#### Example 1: Request Method and Client IP

**File: `info.php`**
```php
<?php
$method = $_SERVER['REQUEST_METHOD'] ?? 'UNKNOWN';
$ip     = $_SERVER['REMOTE_ADDR'] ?? 'UNKNOWN';
$uri    = $_SERVER['REQUEST_URI'] ?? '/';

echo "Method: " . htmlspecialchars($method, ENT_QUOTES, 'UTF-8') . "<br>";
echo "IP: " . htmlspecialchars($ip, ENT_QUOTES, 'UTF-8') . "<br>";
echo "URI: " . htmlspecialchars($uri, ENT_QUOTES, 'UTF-8');
?>
```

**How to Run**

1. Save as `info.php`.
2. Visit `info.php?x=1` from a browser.

**Expected Output**
```
Method: GET
IP: 203.0.113.5
URI: /info.php?x=1
```

**Why This Output Occurs**

- `REQUEST_METHOD` is `GET` for a browser navigation.
- `REMOTE_ADDR` is the client's IP as seen by the server.
- `REQUEST_URI` includes the path and query string.

#### Example 2: Detecting HTTPS and Building a Base URL

**File: `baseurl.php`**
```php
<?php
function isHttps(): bool {
    if (!empty($_SERVER['HTTPS']) && $_SERVER['HTTPS'] !== 'off') {
        return true;
    }
    // Behind a trusted proxy:
    if (($_SERVER['HTTP_X_FORWARDED_PROTO'] ?? '') === 'https') {
        return true;
    }
    return false;
}

$scheme = isHttps() ? 'https' : 'http';
$host   = $_SERVER['HTTP_HOST'] ?? 'localhost';
$base   = $scheme . '://' . $host;

echo 'Base URL: ' . htmlspecialchars($base, ENT_QUOTES, 'UTF-8');
?>
```

**Expected Output (HTTPS request)**
```
Base URL: https://example.com
```

**Why This Output Occurs**

- `isHttps()` checks `$_SERVER['HTTPS']` and a trusted proxy header.
- `HTTP_HOST` provides the host; it is escaped before output.
- The scheme and host are concatenated.

### Real-World Cases

- **Routing:** Using `REQUEST_URI` and `PATH_INFO` to dispatch requests.
- **Logging:** Recording `REMOTE_ADDR`, `HTTP_USER_AGENT`, and `REQUEST_URI`.
- **Rate limiting:** Keying on `REMOTE_ADDR`.
- **HTTPS enforcement:** Redirecting HTTP to HTTPS based on `HTTPS`/`SERVER_PORT`.
- **Building canonical URLs:** Using `HTTP_HOST` and `REQUEST_URI`.
- **Security checks:** Validating `HTTP_HOST` against a whitelist to prevent host header injection.

---

## 5. `$_FILES` — Uploaded File Metadata

### Definitions

**Core Definition**  
`$_FILES` is an associative array of uploaded file metadata for the current request, populated when the form uses `multipart/form-data`.

**Technical Definition**  
`$_FILES` is populated by PHP's SAPI layer for POST requests with `Content-Type: multipart/form-data`. Each file input creates an entry with keys `name`, `type`, `size`, `tmp_name`, `error`, and (as of PHP 8.1.0) `full_path`. For array field names (`photos[]`), each key becomes an array. PHP stores uploaded files in the temporary directory configured by `upload_tmp_dir`; they must be moved with `move_uploaded_file()` before the request ends.

**Beginner-Friendly Explanation**  
When a user uploads a file through a form, PHP saves it to a temporary location and puts information about it into `$_FILES`. You get the original filename, the file size, the temporary path, and an error code. To keep the file, you must move it from the temporary path to a permanent location using `move_uploaded_file()`. If you do nothing, PHP deletes the temporary file when the request ends.

### Purposes

- To receive uploaded files from HTML forms.
- To validate file size, type, and error status.
- To move uploaded files to a permanent location.
- To handle multiple file uploads under one field name.
- To reject malicious or oversized uploads.
- To store file metadata for database records.

### Syntax Rules and Structure

**General Syntax: `$_FILES` Structure (Single File)**
```php
$_FILES['field'] = [
    'name'      => 'photo.jpg',       // original client filename
    'type'      => 'image/jpeg',      // browser-supplied MIME type (untrusted)
    'size'      => 123456,            // bytes
    'tmp_name'  => '/tmp/phpABC123',  // temporary server path
    'error'     => UPLOAD_ERR_OK,     // error code
    'full_path' => 'C:\...\photo.jpg' // PHP 8.1.0+; untrusted
];
```

**Component Breakdown**

- `name` — original filename from the client; sanitize before use.
- `type` — MIME type supplied by the browser; do not trust.
- `size` — file size in bytes.
- `tmp_name` — temporary server path.
- `error` — upload error code.
- `full_path` — full path as submitted by the browser (PHP 8.1.0+); untrusted.

**General Syntax: `$_FILES` Structure (Multiple Files)**
```php
$_FILES['photos'] = [
    'name'      => ['a.jpg', 'b.png'],
    'type'      => ['image/jpeg', 'image/png'],
    'size'      => [123, 456],
    'tmp_name'  => ['/tmp/phpA', '/tmp/phpB'],
    'error'     => [UPLOAD_ERR_OK, UPLOAD_ERR_OK],
    'full_path' => ['C:\...\a.jpg', 'C:\...\b.png']
];
```

**Component Breakdown**

- Each key is an array indexed by file position.
- Iterate with `for ($i = 0; $i < count($_FILES['photos']['name']); $i++)`.

**General Syntax: Error Codes**
```php
UPLOAD_ERR_OK         // 0: success
UPLOAD_ERR_INI_SIZE   // 1: exceeds upload_max_filesize
UPLOAD_ERR_FORM_SIZE  // 2: exceeds MAX_FILE_SIZE
UPLOAD_ERR_PARTIAL    // 3: partial upload
UPLOAD_ERR_NO_FILE    // 4: no file uploaded
UPLOAD_ERR_NO_TMP_DIR // 6: missing temp directory
UPLOAD_ERR_CANT_WRITE // 7: failed to write to disk
UPLOAD_ERR_EXTENSION  // 8: stopped by extension
```

**Component Breakdown**

- `UPLOAD_ERR_OK` — success.
- `UPLOAD_ERR_INI_SIZE` — file exceeds `upload_max_filesize`.
- `UPLOAD_ERR_FORM_SIZE` — file exceeds `MAX_FILE_SIZE`.
- `UPLOAD_ERR_PARTIAL` — file was only partially uploaded.
- `UPLOAD_ERR_NO_FILE` — no file was uploaded.
- `UPLOAD_ERR_NO_TMP_DIR` — temporary folder missing.
- `UPLOAD_ERR_CANT_WRITE` — failed to write to disk.
- `UPLOAD_ERR_EXTENSION` — a PHP extension stopped the upload.

**General Syntax: Moving an Uploaded File**
```php
if ($_FILES['photo']['error'] === UPLOAD_ERR_OK) {
    $tmp  = $_FILES['photo']['tmp_name'];
    $dest = __DIR__ . '/uploads/' . basename($_FILES['photo']['name']);
    move_uploaded_file($tmp, $dest);
}
```

**Component Breakdown**

- `UPLOAD_ERR_OK` — check for success.
- `$tmp` — temporary path.
- `basename()` — strips directory components.
- `move_uploaded_file()` — safely moves the file; verifies it is an uploaded file.

**Syntax Rules**

- The form must use `method="post"` and `enctype="multipart/form-data"`.
- `MAX_FILE_SIZE` must precede the file input; it is client-side only.
- `upload_max_filesize` limits each file.
- `post_max_size` limits the total POST body.
- `max_file_uploads` limits the number of files.
- `upload_tmp_dir` sets the temporary directory.
- Use `move_uploaded_file()` or `is_uploaded_file()` to validate the source.
- Never trust `name`, `type`, or `full_path`.
- Validate size, extension, and MIME type server-side.
- Generate a safe filename; do not use the client filename directly.

**Constraints and Limitations**

- `$_FILES` is empty if the form does not use `multipart/form-data`.
- `$_FILES` is empty if `post_max_size` is exceeded.
- The temporary file is deleted at the end of the request if not moved.
- `type` is browser-supplied and can be forged.
- `full_path` is available only as of PHP 8.1.0 and is untrusted.
- `is_uploaded_file()` is unnecessary before `move_uploaded_file()`.
- Uploaded files can contain malicious content; never execute them.
- `max_file_uploads` defaults to 20.

### Annotated Code Examples and Expected Outputs

#### Example 1: Single File Upload with Validation

**File: `upload.php`**
```php
<?php
if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_FILES['photo'])) {
    $file = $_FILES['photo'];

    // Check error code.
    if ($file['error'] !== UPLOAD_ERR_OK) {
        echo 'Upload error code: ' . (int)$file['error'];
        exit;
    }

    // Size limit: 2 MB.
    if ($file['size'] > 2 * 1024 * 1024) {
        echo 'File too large.';
        exit;
    }

    // Verify MIME type server-side.
    $finfo = new finfo(FILEINFO_MIME_TYPE);
    $mime  = $finfo->file($file['tmp_name']);
    $allowed = ['image/jpeg', 'image/png', 'image/gif'];
    if (!in_array($mime, $allowed, true)) {
        echo 'Unsupported file type.';
        exit;
    }

    // Sanitize filename.
    $safeName = basename($file['name']);
    $safeName = preg_replace('/[^A-Za-z0-9._-]/', '_', $safeName);

    $dir = __DIR__ . '/uploads';
    if (!is_dir($dir)) {
        mkdir($dir, 0755, true);
    }

    if (move_uploaded_file($file['tmp_name'], $dir . '/' . $safeName)) {
        echo 'Uploaded: ' . htmlspecialchars($safeName, ENT_QUOTES, 'UTF-8');
    } else {
        echo 'Failed to move file.';
    }
}
?>
<form method="post" action="upload.php" enctype="multipart/form-data">
  <input type="file" name="photo" accept="image/*">
  <button type="submit">Upload</button>
</form>
```

**Expected Output**
```
Uploaded: photo.jpg
```

**Why This Output Occurs**

- `$_FILES['photo']['error']` is `UPLOAD_ERR_OK`.
- Size and MIME type are validated server-side.
- `basename()` and `preg_replace()` sanitize the filename.
- `move_uploaded_file()` moves the file to the uploads directory.

#### Example 2: Multiple File Upload

**File: `multi.php`**
```php
<?php
if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_FILES['photos'])) {
    $files = $_FILES['photos'];
    $dir = __DIR__ . '/uploads';
    if (!is_dir($dir)) mkdir($dir, 0755, true);

    for ($i = 0; $i < count($files['name']); $i++) {
        if ($files['error'][$i] !== UPLOAD_ERR_OK) {
            echo "File $i error: " . (int)$files['error'][$i] . "<br>";
            continue;
        }
        $safeName = preg_replace('/[^A-Za-z0-9._-]/', '_', basename($files['name'][$i]));
        if (move_uploaded_file($files['tmp_name'][$i], $dir . '/' . $safeName)) {
            echo "Uploaded: " . htmlspecialchars($safeName, ENT_QUOTES, 'UTF-8') . "<br>";
        }
    }
}
?>
<form method="post" action="multi.php" enctype="multipart/form-data">
  <input type="file" name="photos[]" multiple>
  <button type="submit">Upload</button>
</form>
```

**Expected Output**
```
Uploaded: photo1.jpg
Uploaded: photo2.png
```

**Why This Output Occurs**

- `name="photos[]"` creates array keys in `$_FILES['photos']`.
- The loop iterates over each file by index.
- Each file is validated, sanitized, and moved individually.

### Real-World Cases

- **Avatars:** Upload profile pictures.
- **Documents:** Upload PDFs, resumes, or contracts.
- **E-commerce:** Upload product images and receipts.
- **CMS:** Upload media library assets.
- **Social media:** Upload photos and videos.

---

## 6. `$_COOKIE` — HTTP Cookies

### Definitions

**Core Definition**  
`$_COOKIE` is an associative array of cookies sent by the client in the request's `Cookie` header.

**Technical Definition**  
`$_COOKIE` is populated by PHP from the `Cookie` header of the incoming HTTP request. Each cookie is a name/value pair. Cookies are set by the server using `setcookie()` or `setrawcookie()`, which send a `Set-Cookie` header. Cookies can have attributes: `Expires`, `Max-Age`, `Domain`, `Path`, `Secure`, `HttpOnly`, and `SameSite`. `$_COOKIE` is affected by `variables_order`: if `C` is not present, it is not populated. PHP automatically URL-decodes cookie values (unless `setrawcookie()` was used).

**Beginner-Friendly Explanation**  
Cookies are small pieces of data that the server asks the browser to store and send back on every subsequent request to the same domain. `$_COOKIE` lets PHP read those values. Cookies are commonly used for "remember me" tokens, language preferences, and session IDs. Because cookies are stored on the user's machine, they can be viewed, modified, or deleted by the user — never trust them for security-critical decisions.

### Purposes

- To persist small amounts of data across requests.
- To implement "remember me" functionality.
- To store user preferences (language, theme).
- To track sessions via session cookies.
- To read cookies set by JavaScript or other server-side code.
- To support A/B testing and analytics.

### Syntax Rules and Structure

**General Syntax: Reading a Cookie**
```php
$value = $_COOKIE['name'] ?? 'default';
```

**Component Breakdown**

- `$_COOKIE['name']` — value of the cookie named `name`.
- `?? 'default'` — default if the cookie is absent.

**General Syntax: Setting a Cookie**
```php
setcookie(
    'name',                    // cookie name
    'value',                   // cookie value
    [
        'expires' => time() + 3600,   // expiration timestamp
        'path'    => '/',              // path scope
        'domain'  => 'example.com',    // domain scope
        'secure'  => true,             // HTTPS only
        'httponly'=> true,             // not accessible to JavaScript
        'samesite'=> 'Lax'             // CSRF mitigation
    ]
);
```

**Component Breakdown**

- `name` — cookie name.
- `value` — cookie value.
- `expires` — Unix timestamp; 0 means session cookie.
- `path` — URL path scope.
- `domain` — domain scope.
- `secure` — only send over HTTPS.
- `httponly` — block JavaScript access.
- `samesite` — `Strict`, `Lax`, or `None`.

**General Syntax: Deleting a Cookie**
```php
setcookie('name', '', [
    'expires' => time() - 3600,
    'path'    => '/',
    'secure'  => true,
    'httponly'=> true,
    'samesite'=> 'Lax'
]);
```

**Component Breakdown**

- Setting an expired timestamp deletes the cookie.
- Must match the original `path`, `domain`, and other attributes.

**Syntax Rules**

- `setcookie()` must be called before any output (unless output buffering is active).
- Cookie names cannot contain `=`, `,`, `;`, whitespace, or control characters.
- Cookie values are URL-encoded by `setcookie()` and automatically decoded into `$_COOKIE`.
- `setrawcookie()` does not URL-encode; `$_COOKIE` will contain the raw value.
- Cookies are sent on every request to the matching domain/path.
- `SameSite=None` requires `Secure`.
- Modern PHP (7.3+) supports the array syntax for `setcookie()` options.
- `$_COOKIE` is populated from the `Cookie` request header; it does not include cookies set in the same request via `setcookie()`.

**Constraints and Limitations**

- Cookies are stored on the client and can be modified, deleted, or blocked.
- Total cookie size per domain is typically ~4 KB.
- Browsers limit the number of cookies per domain (~50).
- Cookies are sent on every request, increasing bandwidth.
- Cookies are visible to the user and to JavaScript unless `HttpOnly` is set.
- Cookies can be intercepted over plain HTTP unless `Secure` is set.
- Cookies are a CSRF vector; use `SameSite` and CSRF tokens.
- `$_COOKIE` does not reflect cookies set during the current request.
- Third-party cookie restrictions are increasingly strict in modern browsers.

### Annotated Code Examples and Expected Outputs

#### Example 1: Setting and Reading a Preference Cookie

**File: `theme.php`**
```php
<?php
// Set a theme cookie if requested.
if (isset($_GET['theme'])) {
    $theme = $_GET['theme'] === 'dark' ? 'dark' : 'light';
    setcookie('theme', $theme, [
        'expires'  => time() + 60 * 60 * 24 * 30, // 30 days
        'path'     => '/',
        'secure'   => true,
        'httponly' => true,
        'samesite' => 'Lax'
    ]);
    // $_COOKIE is not updated until the next request.
    header('Location: theme.php', true, 303);
    exit;
}

$theme = $_COOKIE['theme'] ?? 'light';
echo 'Current theme: ' . htmlspecialchars($theme, ENT_QUOTES, 'UTF-8');
?>
```

**How to Run**

1. Save as `theme.php`.
2. Visit `theme.php?theme=dark`.
3. The browser stores the cookie and redirects.
4. On the next request, `$_COOKIE['theme']` is `dark`.

**Expected Output**
```
Current theme: dark
```

**Why This Output Occurs**

- `setcookie()` sends a `Set-Cookie` header.
- The redirect causes a new request that includes the cookie.
- `$_COOKIE['theme']` is populated from the `Cookie` header.
- The value is escaped before output.

#### Example 2: Deleting a Cookie

**File: `logout.php`**
```php
<?php
// Delete the 'remember' cookie.
setcookie('remember', '', [
    'expires'  => time() - 3600,
    'path'     => '/',
    'secure'   => true,
    'httponly' => true,
    'samesite' => 'Lax'
]);

echo 'Cookie deleted.';
?>
```

**Expected Output**
```
Cookie deleted.
```

**Why This Output Occurs**

- Setting an expiration in the past instructs the browser to delete the cookie.
- The attributes match the original cookie so the browser identifies it correctly.
- On the next request, `$_COOKIE['remember']` is absent.

### Real-World Cases

- **Remember me:** Store a long-lived token cookie.
- **Language preference:** Store the user's chosen language.
- **Theme:** Store light/dark mode preference.
- **Session ID:** PHP's session cookie (`PHPSESSID`) links the browser to server-side session data.
- **Analytics:** Store a visitor ID for tracking.

---

## 7. `$_SESSION` — Server-Side Session Data

### Definitions

**Core Definition**  
`$_SESSION` is an associative array that stores data on the server, associated with a client via a session ID (usually stored in a cookie).

**Technical Definition**  
`$_SESSION` is populated after `session_start()` is called. PHP generates or resumes a session identified by a session ID (default cookie name `PHPSESSID`). Session data is serialized and stored server-side (files, Redis, Memcached, database). The session ID is transmitted to the client via a cookie (or URL if `session.use_only_cookies=0`, which is discouraged). `session_start()` must be called before any output unless output buffering is enabled. Session data persists across requests until the session expires or is destroyed.

**Beginner-Friendly Explanation**  
Sessions let you remember a user across multiple page requests. When a user logs in, you store their user ID in `$_SESSION`. On the next page, PHP reads the session ID from the cookie, loads the session data, and `$_SESSION['user_id']` is available. The data itself is stored on the server, so the user cannot see or modify it directly (unlike cookies). Sessions are the standard way to implement login state, shopping carts, and multi-step forms.

### Purposes

- To persist user data across requests.
- To implement login/authentication state.
- To store shopping cart contents.
- To store flash messages (one-time notifications).
- To protect against CSRF with per-session tokens.
- To store multi-step form data.
- To track user preferences server-side.
- To manage access control.

### Syntax Rules and Structure

**General Syntax: Starting a Session**
```php
session_start();
```

**Component Breakdown**

- `session_start()` — starts a new session or resumes an existing one.
- Must be called before any output.
- Returns `true` on success, `false` on failure.

**General Syntax: Reading and Writing Session Data**
```php
$_SESSION['user_id'] = 42;
$userId = $_SESSION['user_id'] ?? null;
```

**Component Breakdown**

- `$_SESSION['key']` — reads or writes session data.
- Data can be scalars, arrays, or objects (with serialization caveats).

**General Syntax: Removing and Destroying**
```php
unset($_SESSION['key']);       // remove one key
$_SESSION = [];                // clear all data
session_destroy();             // destroy session data
setcookie(session_name(), '', time() - 3600, '/'); // delete session cookie
```

**Component Breakdown**

- `unset()` — removes a specific key.
- `$_SESSION = []` — clears the array.
- `session_destroy()` — destroys the session data on the server.
- Deleting the cookie removes the client-side reference.

**General Syntax: Regenerating the Session ID**
```php
session_regenerate_id(true);
```

**Component Breakdown**

- `session_regenerate_id(true)` — generates a new session ID and deletes the old session file.
- Call after login to prevent session fixation.

**General Syntax: Secure Session Configuration**
```php
session_set_cookie_params([
    'lifetime' => 0,
    'path'     => '/',
    'domain'   => 'example.com',
    'secure'   => true,
    'httponly' => true,
    'samesite' => 'Lax'
]);
session_start();
```

**Component Breakdown**

- `lifetime` — 0 means session cookie.
- `secure` — HTTPS only.
- `httponly` — block JavaScript access.
- `samesite` — CSRF mitigation.

**Syntax Rules**

- `session_start()` must be called before any output.
- Only one session per request.
- Session data is serialized; resources and closures cannot be stored.
- Session IDs should be regenerated after privilege changes.
- `session_destroy()` does not unset `$_SESSION` or the cookie; do all three.
- `session.use_strict_mode=1` prevents session fixation.
- `session.cookie_secure=1`, `session.cookie_httponly=1`, `session.cookie_samesite=Lax` are recommended.
- `session.gc_maxlifetime` controls garbage collection.
- `session.save_path` controls where session files are stored.
- `session.save_handler` can be `files`, `redis`, `memcached`, etc.

**Constraints and Limitations**

- Sessions require cookies (or URL rewriting, discouraged) to link the client to the server.
- Session data is stored server-side; large sessions consume server resources.
- Sessions are not shared across servers unless a shared handler is used.
- Session fixation is possible if the ID is not regenerated after login.
- Session hijacking is possible if the ID is stolen (e.g., via XSS or insecure transport).
- `session_start()` cannot be called after output unless output buffering is on.
- PHP 8.5.0 throws `ValueError` for non-string keys and `TypeError` for incompatible `read_and_close` options in `session_start()`.
- Concurrent requests may cause session locking issues.
- Session data is not encrypted by default; use HTTPS and secure storage.

### Annotated Code Examples and Expected Outputs

#### Example 1: Login and Session

**File: `login.php`**
```php
<?php
session_start();

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $username = trim($_POST['username'] ?? '');
    $password = $_POST['password'] ?? '';

    // In a real app: fetch hash from DB.
    $storedHash = '$2y$10$examplehash...';

    if ($username !== '' && password_verify($password, $storedHash)) {
        // Regenerate session ID to prevent fixation.
        session_regenerate_id(true);

        $_SESSION['user_id'] = 42;
        $_SESSION['username'] = $username;

        header('Location: dashboard.php', true, 303);
        exit;
    }

    echo 'Invalid credentials.';
}
?>
<form method="post" action="login.php">
  <input type="text" name="username" placeholder="Username">
  <input type="password" name="password" placeholder="Password">
  <button type="submit">Log in</button>
</form>
```

**File: `dashboard.php`**
```php
<?php
session_start();

if (!isset($_SESSION['user_id'])) {
    header('Location: login.php', true, 303);
    exit;
}

echo 'Welcome, ' . htmlspecialchars($_SESSION['username'], ENT_QUOTES, 'UTF-8') . '!';
?>
```

**Expected Output (after login)**
```
Welcome, alice!
```

**Why This Output Occurs**

- `session_start()` resumes the session using the session cookie.
- `$_SESSION['user_id']` and `$_SESSION['username']` persist across requests.
- `session_regenerate_id(true)` prevents session fixation.
- The dashboard checks for `user_id`; if absent, redirects to login.

#### Example 2: Flash Message with Session

**File: `save.php`**
```php
<?php
session_start();

// Simulate saving data.
$_SESSION['flash'] = 'Item saved successfully!';

header('Location: result.php', true, 303);
exit;
?>
```

**File: `result.php`**
```php
<?php
session_start();

$message = $_SESSION['flash'] ?? '';
unset($_SESSION['flash']); // one-time message

echo htmlspecialchars($message, ENT_QUOTES, 'UTF-8');
?>
```

**Expected Output**
```
Item saved successfully!
```

**Why This Output Occurs**

- `$_SESSION['flash']` stores a one-time message.
- `result.php` reads and unsets it, so a refresh does not repeat the message.
- The value is escaped before output.

### Real-World Cases

- **Authentication:** Store user ID and roles.
- **Shopping cart:** Store cart items server-side.
- **Multi-step forms:** Store partial data between steps.
- **CSRF protection:** Store a per-session token.
- **Flash messages:** One-time success/error notifications.
- **Rate limiting:** Store request counts per session.

---

## 8. `$_ENV` — System Environment Variables

### Definitions

**Core Definition**  
`$_ENV` is an associative array of environment variables passed to PHP by the operating system, web server, or process manager.

**Technical Definition**  
`$_ENV` is populated by PHP from the process environment when `variables_order` includes `E`. In many SAPIs (especially PHP-FPM and Apache with `clear_env = on`), `$_ENV` may be empty for security reasons, and environment variables are accessible only via `getenv()`. The `putenv()` function can set environment variables at runtime (affecting `getenv()` but not necessarily `$_ENV`). `$_ENV` is not populated in CLI by default unless `variables_order` includes `E`.

**Beginner-Friendly Explanation**  
`$_ENV` lets PHP read configuration values that are set outside the code — for example, database passwords, API keys, and environment names (development, staging, production). These values are set by the server, container, or operating system, not by users. Using environment variables keeps secrets out of the codebase. However, `$_ENV` is often empty in production for security reasons; `getenv()` is a more reliable alternative.

### Purposes

- To read configuration values from the environment.
- To keep secrets (API keys, database passwords) out of source code.
- To support different configurations per environment (dev, staging, prod).
- To integrate with container orchestration (Docker, Kubernetes).
- To read system-level settings (PATH, HOME, etc.).
- To follow twelve-factor app methodology.

### Syntax Rules and Structure

**General Syntax: Reading `$_ENV`**
```php
$dbHost = $_ENV['DB_HOST'] ?? 'localhost';
```

**Component Breakdown**

- `$_ENV['DB_HOST']` — reads the environment variable `DB_HOST`.
- `?? 'localhost'` — default if absent.

**General Syntax: `getenv()` Alternative**
```php
$dbHost = getenv('DB_HOST') ?: 'localhost';
```

**Component Breakdown**

- `getenv('DB_HOST')` — reads the environment variable; returns `false` if not set.
- `?: 'localhost'` — default if false.

**General Syntax: Setting at Runtime**
```php
putenv('APP_ENV=testing');
$env = getenv('APP_ENV'); // 'testing'
```

**Component Breakdown**

- `putenv()` — sets an environment variable for the current process.
- `getenv()` — reads it back.
- `$_ENV` is not updated by `putenv()`.

**General Syntax: `php.ini` Configuration**
```ini
variables_order = "EGPCS"   ; includes E for ENV
```

**Component Breakdown**

- `E` — enables `$_ENV` population.
- Without `E`, `$_ENV` is empty even if environment variables exist.

**Syntax Rules**

- `$_ENV` is populated only if `variables_order` includes `E`.
- PHP-FPM's `clear_env = on` (default) clears environment variables for security.
- Apache's `SetEnv` and `PassEnv` control which variables are available.
- `getenv()` is more reliable than `$_ENV` in many SAPIs.
- `putenv()` affects `getenv()` but not `$_ENV`.
- Environment variables are strings; cast as needed.
- Do not store secrets in version control; use `.env` files or secrets managers.
- `.env` files are not automatically loaded by PHP; use a library like `vlucas/phpdotenv`.

**Constraints and Limitations**

- `$_ENV` is often empty in PHP-FPM and Apache due to `clear_env`.
- `variables_order` must include `E`.
- Environment variables are process-wide; changing them affects the whole process.
- `putenv()` is not thread-safe in some SAPIs.
- Secrets in environment variables can leak via `phpinfo()`, error pages, or process listings.
- Environment variables are not encrypted.
- `.env` files must not be committed to version control.
- `getenv()` without arguments returns all environment variables (PHP 7.1+); avoid exposing them.

### Annotated Code Examples and Expected Outputs

#### Example 1: Reading Configuration from Environment

**File: `config.php`**
```php
<?php
// Prefer getenv() for reliability; fall back to $_ENV.
function env(string $key, ?string $default = null): ?string {
    $value = getenv($key);
    if ($value === false) {
        $value = $_ENV[$key] ?? null;
    }
    return $value ?? $default;
}

$dbHost = env('DB_HOST', 'localhost');
$dbName = env('DB_NAME', 'app');
$appEnv = env('APP_ENV', 'production');

echo "DB Host: " . htmlspecialchars($dbHost, ENT_QUOTES, 'UTF-8') . "<br>";
echo "DB Name: " . htmlspecialchars($dbName, ENT_QUOTES, 'UTF-8') . "<br>";
echo "App Env: " . htmlspecialchars($appEnv, ENT_QUOTES, 'UTF-8');
?>
```

**How to Run**

1. Set environment variables: `export DB_HOST=db.example.com DB_NAME=myapp APP_ENV=development`.
2. Run `php -S localhost:8000` in the same shell.
3. Visit `config.php`.

**Expected Output**
```
DB Host: db.example.com
DB Name: myapp
App Env: development
```

**Why This Output Occurs**

- `getenv('DB_HOST')` reads the environment variable set in the shell.
- The `env()` helper falls back to `$_ENV` and then to the default.
- Values are escaped before output.

#### Example 2: Using a `.env` File with `vlucas/phpdotenv`

**File: `.env`**
```
APP_ENV=local
DB_HOST=127.0.0.1
DB_NAME=myapp
```

**File: `bootstrap.php`**
```php
<?php
require __DIR__ . '/vendor/autoload.php';

$dotenv = Dotenv\Dotenv::createImmutable(__DIR__);
$dotenv->load();

// Now $_ENV and getenv() are populated.
echo 'APP_ENV: ' . htmlspecialchars($_ENV['APP_ENV'] ?? 'unknown', ENT_QUOTES, 'UTF-8');
```

**Expected Output**
```
APP_ENV: local
```

**Why This Output Occurs**

- `Dotenv::createImmutable()` loads `.env` without overwriting existing variables.
- `load()` populates `$_ENV` and `getenv()`.
- The value is escaped before output.

### Real-World Cases

- **Database credentials:** `DB_HOST`, `DB_USER`, `DB_PASS`.
- **API keys:** `STRIPE_KEY`, `AWS_ACCESS_KEY_ID`.
- **Environment:** `APP_ENV=production`.
- **Feature flags:** `FEATURE_NEW_UI=true`.
- **Container orchestration:** Kubernetes `ConfigMap` and `Secret` injected as environment variables.

---

## Security Considerations

### Input Validation and Escaping

- Treat all superglobal data as untrusted.
- Use `filter_input()` and `filter_var()` for validation.
- Use `htmlspecialchars()` for HTML output.
- Use prepared statements for database queries.
- Use context-aware escaping for attributes, URLs, JavaScript, and CSS.

### `$_REQUEST` Risks

- Avoid `$_REQUEST` for state-changing operations.
- Use `$_GET` and `$_POST` explicitly.
- Enforce the request method with `$_SERVER['REQUEST_METHOD']`.
- Implement CSRF tokens for state-changing forms.

### Session Security

- Call `session_regenerate_id(true)` after login.
- Set `session.cookie_secure=1`, `session.cookie_httponly=1`, `session.cookie_samesite=Lax`.
- Set `session.use_strict_mode=1`.
- Use HTTPS.
- Store sessions in a secure, shared backend for multi-server deployments.
- Destroy sessions completely on logout: `$_SESSION = []`, `session_destroy()`, delete cookie.

### Cookie Security

- Use `Secure`, `HttpOnly`, and `SameSite` attributes.
- Do not store sensitive data in cookies.
- Use opaque tokens, not user data.
- Validate cookie values server-side.

### File Upload Security

- Validate error code, size, extension, and MIME type.
- Use `finfo_file()` for server-side MIME detection.
- Sanitize filenames with `basename()` and character whitelisting.
- Store uploads outside the web root.
- Never execute uploaded files.
- Limit `upload_max_filesize`, `post_max_size`, `max_file_uploads`.

### `$_SERVER` Risks

- `HTTP_HOST` is user-controlled; validate against a whitelist.
- `PHP_SELF` can contain XSS payloads; escape before output.
- `HTTP_X_FORWARDED_FOR` is untrusted unless set by a trusted proxy.
- `HTTP_REFERER` can be spoofed; do not use for security.

### `$_ENV` Risks

- `phpinfo()` and error pages can expose environment variables.
- Process listings (`ps`) can expose environment variables.
- Use secrets managers for production.
- Do not commit `.env` files.

### Deprecated, Unsafe, or Version-Specific Features

- `register_globals` — removed in PHP 5.4; never use.
- `$_REQUEST` — unsafe for state-changing operations.
- `FILTER_SANITIZE_STRING` — deprecated in PHP 8.1; use `FILTER_SANITIZE_SPECIAL_CHARS` or `htmlspecialchars()`.
- `session_register()`, `session_unregister()`, `session_is_registered()` — removed in PHP 5.4.
- `set_magic_quotes_runtime()` — removed in PHP 5.4.
- `magic_quotes_gpc` — removed in PHP 5.4; do not rely on it.
- `$_FILES['full_path']` — available as of PHP 8.1.0; untrusted.
- `session_start()` options — PHP 8.5.0 throws `ValueError` for non-string keys and `TypeError` for incompatible `read_and_close`.

---

## References

- PHP: Predefined Variables — https://www.php.net/manual/en/reserved.variables.php
- PHP: `$_GET` — https://www.php.net/manual/en/reserved.variables.get.php
- PHP: `$_POST` — https://www.php.net/manual/en/reserved.variables.post.php
- PHP: `$_REQUEST` — https://www.php.net/manual/en/reserved.variables.request.php
- PHP: `$_SERVER` — https://www.php.net/manual/en/reserved.variables.server.php
- PHP: `$_FILES` — https://www.php.net/manual/en/reserved.variables.files.php
- PHP: `$_COOKIE` — https://www.php.net/manual/en/reserved.variables.cookies.php
- PHP: `$_SESSION` — https://www.php.net/manual/en/reserved.variables.session.php
- PHP: `$_ENV` — https://www.php.net/manual/en/reserved.variables.environment.php
- PHP: `filter_input()` — https://www.php.net/manual/en/function.filter-input.php
- PHP: `htmlspecialchars()` — https://www.php.net/manual/en/function.htmlspecialchars.php
- PHP: `setcookie()` — https://www.php.net/manual/en/function.setcookie.php
- PHP: `session_start()` — https://www.php.net/manual/en/function.session-start.php
- PHP: `session_regenerate_id()` — https://www.php.net/manual/en/function.session-regenerate-id.php
- PHP: `move_uploaded_file()` — https://www.php.net/manual/en/function.move-uploaded-file.php
- PHP: `getenv()` — https://www.php.net/manual/en/function.getenv.php
- PHP: `putenv()` — https://www.php.net/manual/en/function.putenv.php
- PHP: Handling File Uploads — https://www.php.net/manual/en/features.file-upload.php
- PHP: Session Security — https://www.php.net/manual/en/session.security.php
- PHP: `php.ini` Directives — https://www.php.net/manual/en/ini.list.php
- MDN: HTTP Cookies — https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Cookies
- MDN: `Set-Cookie` — https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Set-Cookie
- MDN: HTTP Headers — https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers
- RFC 6265: HTTP State Management Mechanism — https://datatracker.ietf.org/doc/html/rfc6265
- RFC 9110: HTTP Semantics — https://datatracker.ietf.org/doc/rfc9110/
- OWASP: Cross-Site Request Forgery Prevention Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html
- OWASP: Session Management Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html
- OWASP: File Upload Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html
- The Twelve-Factor App: Config — https://12factor.net/config