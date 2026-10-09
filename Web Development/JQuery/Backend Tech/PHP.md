# jQuery with PHP — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** jQuery with PHP is the practice of combining jQuery's client-side DOM manipulation and AJAX capabilities with PHP's server-side scripting to build dynamic, data-driven web applications where the frontend and backend communicate asynchronously without full page reloads.

**Technical Definition:** jQuery with PHP refers to the integration of two technologies: jQuery (a JavaScript library for DOM manipulation, event handling, and AJAX) and PHP (a server-side scripting language for generating dynamic content and interacting with databases). The integration is achieved through HTTP requests initiated by jQuery's `$.ajax()`, `$.get()`, and `$.post()` methods, which send data to PHP scripts. PHP processes the data (validating, storing, retrieving from databases), and returns responses — typically JSON-encoded — which jQuery then parses and uses to update the DOM. This architecture enables Single-Page-Application-like behavior in traditional multi-page PHP applications.

**Beginner-Friendly Explanation:** Imagine a restaurant. The customer (jQuery in the browser) looks at the menu and places an order. The waiter (AJAX) takes the order to the kitchen (PHP on the server). The kitchen prepares the food (processes data, queries the database) and sends it back through the waiter. The customer never leaves their seat — the page never fully reloads. This cheat sheet covers how to make this communication work smoothly: sending orders, receiving food, handling special requests, and managing the customer's account.

### Key Characteristics

- **Asynchronous communication:** jQuery sends requests to PHP in the background; the page does not reload.
- **JSON as the lingua franca:** PHP's `json_encode()` and jQuery's `dataType: "json"` provide a standardized data exchange format.
- **FormData for file uploads:** jQuery's `FormData` object enables multipart form submission, accessible in PHP via `$_POST` and `$_FILES`.
- **Session synchronization:** PHP sessions and jQuery AJAX require careful cookie handling, especially in cross-origin scenarios.
- **CRUD mapping:** UI actions (create, read, update, delete) map directly to PHP scripts that perform database operations.
- **Header management:** Custom headers can be set in jQuery via `beforeSend` or the `headers` option and read in PHP via `$_SERVER['HTTP_*']`.

### Prerequisites

- Basic understanding of HTML, CSS, and JavaScript.
- Familiarity with jQuery fundamentals: selectors, events, and AJAX methods.
- Working knowledge of PHP: variables, arrays, `$_POST`, `$_GET`, `$_SESSION`, and `json_encode()`.
- Basic understanding of HTTP methods (GET, POST) and status codes.

### Related Programming Areas

- **AJAX and Asynchronous Programming:** The core of jQuery-PHP communication.
- **RESTful API Design:** jQuery AJAX calls map to REST endpoints in PHP.
- **Database Interaction (MySQL/PDO):** PHP scripts perform CRUD operations on databases.
- **Session Management and Authentication:** PHP sessions and jQuery-driven login/logout flows.
- **File Upload Handling:** FormData and PHP's `$_FILES` array.
- **Security:** CSRF protection, input validation, and output escaping.

### Core Concepts / Features

This cheat sheet covers six core concepts: AJAX requests, form submission, JSON responses, CRUD interfaces, session and auth synchronization, and header management.

---

## Core Concept 1: AJAX Requests — `$.ajax`, `$.get`, `$.post`

### Definitions

**Core Definition:** AJAX requests are asynchronous HTTP requests initiated by jQuery to a PHP script, allowing data to be sent to and retrieved from the server without reloading the page.

**Technical Definition:** jQuery provides three primary methods for AJAX requests: `$.ajax()` (the full-featured method with all configuration options), `$.get()` (a shorthand for GET requests), and `$.post()` (a shorthand for POST requests). Each method returns a jqXHR object that implements the Promise interface, allowing `.done()`, `.fail()`, and `.always()` callbacks. The `$.ajax()` method accepts an options object specifying the URL, HTTP method, data, data type, success/error callbacks, and more. The `url` option points to a PHP script that processes the request and returns a response.

**Beginner-Friendly Explanation:** Think of `$.ajax()` as a messenger. You tell it where to go (the URL), what to carry (the data), and what to do when it comes back (the success callback). `$.get()` and `$.post()` are like pre-written messages for common cases — one for asking questions (GET) and one for submitting forms (POST).

### Purposes

- To send data from the browser to a PHP script without reloading the page.
- To retrieve data from a PHP script and update the DOM dynamically.
- To provide a consistent, cross-browser API for asynchronous HTTP communication.
- To enable real-time or near-real-time user experiences (search, notifications, chat).
- To separate frontend presentation from backend logic.

### Syntax Rules and Structure

**Complete General Syntaxes with Breakdowns:**

**`$.ajax()` — Full Configuration:**
```javascript
$.ajax({
    url: "process.php",
    type: "POST",
    data: { name: "John", age: 25 },
    dataType: "json",
    success: function(response) { ... },
    error: function(jqXHR, textStatus, errorThrown) { ... }
});
```

| Component | Description |
|-----------|-------------|
| `url` | The PHP script to send the request to. |
| `type` | HTTP method: `"GET"`, `"POST"`, `"PUT"`, `"DELETE"`. |
| `data` | Data to send (object or query string). |
| `dataType` | Expected response type: `"json"`, `"html"`, `"text"`. |
| `success` | Callback invoked on successful response. |
| `error` | Callback invoked on failure. |

**`$.get()` — GET Request:**
```javascript
$.get("data.php", { id: 42 }, function(response) {
    // Handle response
}, "json");
```

**`$.post()` — POST Request:**
```javascript
$.post("save.php", { name: "John" }, function(response) {
    // Handle response
}, "json");
```

**Syntax Rules:**

- The `url` must point to a PHP script that exists and is accessible.
- `dataType: "json"` tells jQuery to automatically parse the response as JSON.
- `$.get()` and `$.post()` are shorthand for `$.ajax()` with `type` set accordingly.
- Use `.done()`, `.fail()`, and `.always()` for Promise-based handling.

**Constraints and Limitations:**

- GET requests have URL length limits (typically 2,000–8,000 characters).
- POST requests are not cached by the browser.
- Cross-origin requests require CORS configuration on the PHP server.
- AJAX requests are subject to the Same-Origin Policy unless CORS is configured.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Sending and Receiving Data with `$.ajax()`**

```html
<!-- index.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>AJAX with PHP Demo</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <button id="loadBtn">Load User</button>
    <div id="result"></div>

    <script>
        $(function() {
            $("#loadBtn").click(function() {
                $.ajax({
                    url: "get_user.php",
                    type: "GET",
                    data: { id: 1 },
                    dataType: "json",
                    success: function(response) {
                        $("#result").html(
                            "<h3>" + response.name + "</h3>" +
                            "<p>Email: " + response.email + "</p>"
                        );
                    },
                    error: function(jqXHR, textStatus, errorThrown) {
                        $("#result").text("Error: " + textStatus);
                    }
                });
            });
        });
    </script>
</body>
</html>
```

```php
<?php
// get_user.php
header('Content-Type: application/json; charset=utf-8');

$userId = isset($_GET['id']) ? (int)$_GET['id'] : 0;

// Simulated database lookup
$users = [
    1 => ['name' => 'Leanne Graham', 'email' => 'Sincere@april.biz'],
    2 => ['name' => 'Ervin Howell', 'email' => 'Shanna@melissa.tv']
];

if (isset($users[$userId])) {
    echo json_encode($users[$userId]);
} else {
    http_response_code(404);
    echo json_encode(['error' => 'User not found']);
}
?>
```

**Expected Output:** Clicking "Load User" displays "Leanne Graham" and "Email: Sincere@april.biz" in the result div.

**Why this output:** jQuery sends a GET request to `get_user.php` with `id=1`. PHP reads `$_GET['id']`, looks up the user in the simulated database, and echoes the JSON-encoded result. jQuery parses the JSON and inserts the data into `#result`.

### Real-World Cases

- **Autocomplete search:** Sending keystrokes to a PHP script that returns matching suggestions.
- **Notifications:** Polling a PHP endpoint for new messages every few seconds.
- **Data loading:** Fetching paginated records from a PHP script as the user scrolls.

---

## Core Concept 2: Form Submission — Handling `$_POST` and `FormData`

### Definitions

**Core Definition:** Form submission with jQuery and PHP is the process of capturing a form's submit event, serializing its data, and sending it to a PHP script via AJAX, where it is accessible through `$_POST` (for text fields) and `$_FILES` (for file uploads).

**Technical Definition:** jQuery provides `.serialize()` for serializing form data into a URL-encoded string and `FormData` for handling multipart data, including file uploads. When using `FormData`, the `processData` and `contentType` options must be set to `false` to prevent jQuery from transforming the data. PHP receives the data in `$_POST` for regular fields and `$_FILES` for uploaded files.

**Beginner-Friendly Explanation:** Submitting a form with AJAX is like mailing a package without going to the post office. You put everything in the box (the form data), label it (the PHP script URL), and send it. PHP opens the box and finds the contents in `$_POST` (letters) and `$_FILES` (photos).

### Purposes

- To submit forms without reloading the page, providing a smoother user experience.
- To validate form data on the server side and return error messages without losing user input.
- To upload files asynchronously alongside text data.
- To provide instant feedback on form submission success or failure.

### Syntax Rules and Structure

**Complete General Syntax (Text-Only Form):**
```javascript
$("#myForm").on("submit", function(e) {
    e.preventDefault();
    $.ajax({
        url: "process.php",
        type: "POST",
        data: $(this).serialize(),
        dataType: "json",
        success: function(response) { ... }
    });
});
```

**Complete General Syntax (FormData with File Upload):**
```javascript
$("#myForm").on("submit", function(e) {
    e.preventDefault();
    var formData = new FormData(this);
    $.ajax({
        url: "upload.php",
        type: "POST",
        data: formData,
        processData: false,
        contentType: false,
        success: function(response) { ... }
    });
});
```

| Component | Description |
|-----------|-------------|
| `.serialize()` | Serializes form data into a URL-encoded string. |
| `new FormData(this)` | Creates a FormData object from the form element. |
| `processData: false` | Prevents jQuery from transforming the data. |
| `contentType: false` | Prevents jQuery from setting the Content-Type header. |

**Syntax Rules:**

- Always call `e.preventDefault()` to stop the browser's default form submission.
- Use `.serialize()` for text-only forms; use `FormData` for file uploads.
- Set `processData: false` and `contentType: false` when using `FormData`.
- In PHP, access regular fields via `$_POST` and files via `$_FILES`.

**Constraints and Limitations:**

- `FormData` is not supported in IE9 and below.
- File uploads via `FormData` require the form to have `enctype="multipart/form-data"` for non-AJAX fallback.
- PHP's `upload_max_filesize` and `post_max_size` settings limit upload sizes.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Form Submission with File Upload**

```html
<!-- index.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Form Submission Demo</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <form id="uploadForm" enctype="multipart/form-data">
        <input type="text" name="username" placeholder="Username" required>
        <input type="file" name="avatar" accept="image/*">
        <button type="submit">Upload</button>
    </form>
    <div id="messages"></div>

    <script>
        $(function() {
            $("#uploadForm").on("submit", function(e) {
                e.preventDefault();
                var formData = new FormData(this);

                $.ajax({
                    url: "upload.php",
                    type: "POST",
                    data: formData,
                    processData: false,
                    contentType: false,
                    success: function(response) {
                        $("#messages").html(
                            "Welcome, " + response.username + "! " +
                            "Avatar saved as: " + response.filename
                        );
                    },
                    error: function() {
                        $("#messages").text("Upload failed.");
                    }
                });
            });
        });
    </script>
</body>
</html>
```

```php
<?php
// upload.php
header('Content-Type: application/json; charset=utf-8');

$username = isset($_POST['username']) ? $_POST['username'] : 'Guest';
$filename = 'none';

if (isset($_FILES['avatar']) && $_FILES['avatar']['error'] === UPLOAD_ERR_OK) {
    $uploadDir = 'uploads/';
    $filename = basename($_FILES['avatar']['name']);
    $targetPath = $uploadDir . $filename;

    if (move_uploaded_file($_FILES['avatar']['tmp_name'], $targetPath)) {
        // File uploaded successfully
    } else {
        $filename = 'error';
    }
}

echo json_encode([
    'username' => $username,
    'filename' => $filename
]);
?>
```

**Expected Output:** Submitting the form displays "Welcome, [username]! Avatar saved as: [filename]" in the messages div. If no file was selected, "none" is displayed.

**Why this output:** jQuery creates a `FormData` object from the form, including the text field and file input. PHP reads `$_POST['username']` and `$_FILES['avatar']`, moves the uploaded file to the `uploads/` directory, and returns a JSON response with the username and filename.

### Real-World Cases

- **User registration with avatar:** Uploading a profile picture alongside username and email.
- **Product creation forms:** Adding product details and multiple images in one submission.
- **Contact forms:** Sending messages with optional file attachments.
- **Document management:** Uploading files to a PHP backend with metadata.

---

## Core Concept 3: JSON Responses — Encoding with `json_encode()` and Decoding in jQuery

### Definitions

**Core Definition:** JSON responses are the standard format for data exchange between PHP and jQuery, where PHP uses `json_encode()` to convert PHP arrays and objects into JSON strings, and jQuery uses `dataType: "json"` to automatically parse those strings into JavaScript objects.

**Technical Definition:** PHP's `json_encode()` function converts PHP data structures (associative arrays, indexed arrays, objects) into JSON-formatted strings. The `header('Content-Type: application/json; charset=utf-8')` header ensures that jQuery correctly identifies the response as JSON. On the client side, `dataType: "json"` in `$.ajax()` or the use of `$.getJSON()` automatically calls `JSON.parse()` on the response, delivering a JavaScript object to the success callback. PHP's `json_decode()` performs the reverse operation, converting JSON strings received from jQuery into PHP arrays.

**Beginner-Friendly Explanation:** JSON is the common language that PHP and JavaScript both understand. PHP speaks PHP (arrays), JavaScript speaks JavaScript (objects), and JSON is the translator. `json_encode()` translates PHP to JSON; jQuery translates JSON back to JavaScript. Without this translation, the two sides cannot understand each other.

### Purposes

- To provide a standardized, language-independent format for data exchange.
- To enable automatic parsing of server responses in jQuery without manual string manipulation.
- To support complex, nested data structures (arrays of objects, multi-dimensional arrays).
- To reduce bandwidth compared to XML or HTML responses.
- To enable clean separation between data (JSON) and presentation (HTML rendered by jQuery).

### Syntax Rules and Structure

**Complete General Syntax (PHP Side):**
```php
<?php
header('Content-Type: application/json; charset=utf-8');

$data = [
    'status' => 'success',
    'user' => [
        'name' => 'John',
        'email' => 'john@example.com'
    ],
    'items' => ['apple', 'banana', 'cherry']
];

echo json_encode($data, JSON_UNESCAPED_UNICODE | JSON_UNESCAPED_SLASHES);
?>
```

**Complete General Syntax (jQuery Side):**
```javascript
$.ajax({
    url: "data.php",
    dataType: "json",
    success: function(response) {
        // response is already a JavaScript object
        console.log(response.user.name);
        $.each(response.items, function(i, item) {
            console.log(item);
        });
    }
});
```

| PHP Function | Purpose |
|--------------|---------|
| `json_encode($data)` | Converts PHP array/object to JSON string. |
| `json_decode($json, true)` | Converts JSON string to PHP associative array. |
| `JSON_UNESCAPED_UNICODE` | Prevents Unicode characters from being escaped as `\uXXXX`. |
| `JSON_UNESCAPED_SLASHES` | Prevents slashes from being escaped as `\/`. |

**Syntax Rules:**

- Always set `header('Content-Type: application/json; charset=utf-8')` before echoing JSON.
- Use `echo json_encode($data)` instead of manually concatenating JSON strings.
- On the jQuery side, set `dataType: "json"` or use `$.getJSON()`.
- jQuery automatically parses the JSON response; do not call `JSON.parse()` again.
- Use `JSON_UNESCAPED_UNICODE` and `JSON_UNESCAPED_SLASHES` for cleaner output.

**Constraints and Limitations:**

- `json_encode()` fails on resources (database connections) and may fail on malformed UTF-8 strings.
- `json_decode()` returns `null` on invalid JSON; always check for errors.
- Large JSON payloads increase transfer time and parsing overhead.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: PHP Array to JSON to jQuery Object**

```html
<!-- index.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>JSON Response Demo</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <ul id="userList"></ul>

    <script>
        $(function() {
            $.getJSON("get_users.php", function(response) {
                // response is a JavaScript array of objects
                $.each(response.users, function(i, user) {
                    $("#userList").append(
                        "<li>" + user.name + " — " + user.email + "</li>"
                    );
                });
            });
        });
    </script>
</body>
</html>
```

```php
<?php
// get_users.php
header('Content-Type: application/json; charset=utf-8');

$users = [
    ['name' => 'Leanne Graham', 'email' => 'Sincere@april.biz'],
    ['name' => 'Ervin Howell', 'email' => 'Shanna@melissa.tv'],
    ['name' => 'Clementine Bauch', 'email' => 'Nathan@yesenia.net']
];

echo json_encode(['users' => $users], JSON_UNESCAPED_UNICODE | JSON_UNESCAPED_SLASHES);
?>
```

**Expected Output:** The user list displays three users with their names and emails.

**Why this output:** PHP's `json_encode()` converts the nested array into a JSON string. jQuery's `$.getJSON()` automatically parses it into a JavaScript object with a `users` property containing an array of user objects. The `$.each()` loop iterates over the array and appends each user as a list item.

### Real-World Cases

- **Data tables:** Returning rows of data from PHP as JSON for rendering in a jQuery DataTable.
- **Autocomplete:** Returning matching suggestions as a JSON array.
- **Dashboard widgets:** Returning statistics and chart data as JSON.
- **API endpoints:** Returning RESTful JSON responses for SPA consumption.

---

## Core Concept 4: CRUD Interfaces — Mapping UI Actions to PHP Database Scripts

### Definitions

**Core Definition:** CRUD interfaces are user interfaces that allow users to Create, Read, Update, and Delete records in a database, with each action mapped to a specific PHP script that performs the corresponding database operation.

**Technical Definition:** CRUD is an acronym for the four fundamental operations of persistent storage: Create (INSERT), Read (SELECT), Update (UPDATE), and Delete (DELETE). In a jQuery-PHP application, each CRUD operation is implemented as an AJAX request to a dedicated PHP script. The Create operation typically sends POST data to an `insert.php` script; Read sends GET requests to `select.php` or `list.php`; Update sends POST or PUT data to `update.php`; and Delete sends a POST or DELETE request to `delete.php`. PHP scripts use PDO or MySQLi to interact with the database and return JSON responses indicating success or failure.

**Beginner-Friendly Explanation:** CRUD is the four things you can do with data: make it, see it, change it, or remove it. In a jQuery-PHP app, each of these actions has its own button (the UI) and its own PHP script (the backend). Clicking "Add" sends a create request; clicking "Delete" sends a delete request. The UI and the database stay in sync through AJAX.

### Purposes

- To provide a complete data management interface for users.
- To separate database operations into distinct, maintainable PHP scripts.
- To enable real-time UI updates after each CRUD operation without page reloads.
- To validate and sanitize data on the server side before database operations.
- To provide user feedback (success/error messages) for each operation.

### Syntax Rules and Structure

**CRUD Mapping Table:**

| UI Action | HTTP Method | PHP Script | SQL Operation | jQuery Method |
|-----------|-------------|-----------|---------------|---------------|
| Create | POST | `insert.php` | INSERT | `$.post()` |
| Read | GET | `select.php` | SELECT | `$.getJSON()` |
| Update | POST | `update.php` | UPDATE | `$.post()` |
| Delete | POST/DELETE | `delete.php` | DELETE | `$.ajax()` |

**Complete General Syntax (Create):**
```javascript
$("#addBtn").click(function() {
    $.post("insert.php", {
        name: $("#name").val(),
        email: $("#email").val()
    }, function(response) {
        if (response.success) {
            $("#userTable").append("<tr><td>" + response.name + "</td></tr>");
        }
    }, "json");
});
```

**Complete General Syntax (Delete):**
```javascript
$(".deleteBtn").click(function() {
    var id = $(this).data("id");
    $.post("delete.php", { id: id }, function(response) {
        if (response.success) {
            $("#row-" + id).remove();
        }
    }, "json");
});
```

**Syntax Rules:**

- Use POST for Create, Update, and Delete operations (never GET for state-changing actions).
- Validate and sanitize all input on the server side in PHP.
- Return a consistent JSON response format (e.g., `{ success: true, message: "..." }`).
- Update the UI only after the server confirms success.
- Use event delegation for dynamically generated CRUD buttons.

**Constraints and Limitations:**

- CRUD operations must be protected against CSRF, SQL injection, and XSS.
- Concurrent updates can cause data conflicts; consider optimistic locking.
- Large datasets require pagination for the Read operation.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Complete CRUD with jQuery and PHP**

```html
<!-- index.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>CRUD Demo</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <h1>User Management</h1>

    <form id="userForm">
        <input type="hidden" id="userId" value="">
        <input type="text" id="userName" placeholder="Name" required>
        <input type="email" id="userEmail" placeholder="Email" required>
        <button type="submit" id="saveBtn">Save</button>
        <button type="button" id="cancelBtn" style="display:none;">Cancel</button>
    </form>

    <table id="userTable">
        <thead><tr><th>Name</th><th>Email</th><th>Actions</th></tr></thead>
        <tbody></tbody>
    </table>

    <script>
        $(function() {
            // ===== READ =====
            function loadUsers() {
                $.getJSON("read.php", function(users) {
                    var rows = "";
                    $.each(users, function(i, user) {
                        rows += "<tr id='row-" + user.id + "'>" +
                            "<td>" + user.name + "</td>" +
                            "<td>" + user.email + "</td>" +
                            "<td>" +
                            "<button class='editBtn' data-id='" + user.id + "' data-name='" + user.name + "' data-email='" + user.email + "'>Edit</button> " +
                            "<button class='deleteBtn' data-id='" + user.id + "'>Delete</button>" +
                            "</td></tr>";
                    });
                    $("#userTable tbody").html(rows);
                });
            }

            loadUsers();

            // ===== CREATE / UPDATE =====
            $("#userForm").on("submit", function(e) {
                e.preventDefault();
                var id = $("#userId").val();
                var url = id ? "update.php" : "insert.php";
                var data = {
                    name: $("#userName").val(),
                    email: $("#userEmail").val()
                };
                if (id) data.id = id;

                $.post(url, data, function(response) {
                    if (response.success) {
                        resetForm();
                        loadUsers();
                    } else {
                        alert("Error: " + response.message);
                    }
                }, "json");
            });

            // ===== DELETE =====
            $(document).on("click", ".deleteBtn", function() {
                if (!confirm("Delete this user?")) return;
                var id = $(this).data("id");
                $.post("delete.php", { id: id }, function(response) {
                    if (response.success) {
                        $("#row-" + id).remove();
                    }
                }, "json");
            });

            // ===== EDIT (populate form) =====
            $(document).on("click", ".editBtn", function() {
                $("#userId").val($(this).data("id"));
                $("#userName").val($(this).data("name"));
                $("#userEmail").val($(this).data("email"));
                $("#saveBtn").text("Update");
                $("#cancelBtn").show();
            });

            $("#cancelBtn").click(resetForm);

            function resetForm() {
                $("#userForm")[0].reset();
                $("#userId").val("");
                $("#saveBtn").text("Save");
                $("#cancelBtn").hide();
            }
        });
    </script>
</body>
</html>
```

```php
<?php
// db.php — database connection
$pdo = new PDO('mysql:host=localhost;dbname=crud_demo;charset=utf8mb4', 'root', '');
$pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
?>
```

```php
<?php
// read.php
require 'db.php';
header('Content-Type: application/json; charset=utf-8');

$stmt = $pdo->query("SELECT id, name, email FROM users ORDER BY id DESC");
$users = $stmt->fetchAll(PDO::FETCH_ASSOC);

echo json_encode($users, JSON_UNESCAPED_UNICODE);
?>
```

```php
<?php
// insert.php
require 'db.php';
header('Content-Type: application/json; charset=utf-8');

$name = isset($_POST['name']) ? trim($_POST['name']) : '';
$email = isset($_POST['email']) ? trim($_POST['email']) : '';

if (empty($name) || empty($email)) {
    echo json_encode(['success' => false, 'message' => 'All fields are required.']);
    exit;
}

$stmt = $pdo->prepare("INSERT INTO users (name, email) VALUES (?, ?)");
$stmt->execute([$name, $email]);

echo json_encode(['success' => true, 'id' => $pdo->lastInsertId()]);
?>
```

```php
<?php
// update.php
require 'db.php';
header('Content-Type: application/json; charset=utf-8');

$id = isset($_POST['id']) ? (int)$_POST['id'] : 0;
$name = isset($_POST['name']) ? trim($_POST['name']) : '';
$email = isset($_POST['email']) ? trim($_POST['email']) : '';

if (!$id || empty($name) || empty($email)) {
    echo json_encode(['success' => false, 'message' => 'Invalid data.']);
    exit;
}

$stmt = $pdo->prepare("UPDATE users SET name = ?, email = ? WHERE id = ?");
$stmt->execute([$name, $email, $id]);

echo json_encode(['success' => true]);
?>
```

```php
<?php
// delete.php
require 'db.php';
header('Content-Type: application/json; charset=utf-8');

$id = isset($_POST['id']) ? (int)$_POST['id'] : 0;

if (!$id) {
    echo json_encode(['success' => false, 'message' => 'Invalid ID.']);
    exit;
}

$stmt = $pdo->prepare("DELETE FROM users WHERE id = ?");
$stmt->execute([$id]);

echo json_encode(['success' => true]);
?>
```

**Expected Output:** The page displays a list of users loaded via `read.php`. Submitting the form creates a new user (via `insert.php`) or updates an existing one (via `update.php`). Clicking "Edit" populates the form with the user's data. Clicking "Delete" removes the user (via `delete.php`) and the row disappears from the table.

**Why this output:** Each CRUD operation maps to a dedicated PHP script that uses PDO prepared statements for security. jQuery handles the AJAX requests and updates the UI based on the JSON responses. The `loadUsers()` function refreshes the list after Create and Update operations.

### Real-World Cases

- **Admin panels:** Managing users, products, orders, and other entities.
- **Content management systems:** Creating, editing, and deleting articles or pages.
- **Inventory management:** Adding, updating, and removing stock items.
- **Customer relationship management:** Managing contacts, deals, and activities.

---

## Core Concept 5: Session & Auth Synchronization — Managing Login State Across Backend Sessions and Frontend States

### Definitions

**Core Definition:** Session and authentication synchronization is the practice of maintaining a consistent login state between PHP's server-side sessions (`$_SESSION`) and the jQuery frontend, ensuring that AJAX requests are authenticated and that the UI reflects the user's login status.

**Technical Definition:** PHP sessions use a session cookie (typically `PHPSESSID`) to identify the user across requests. When jQuery makes an AJAX request to a PHP script, the browser automatically includes the session cookie for same-origin requests, allowing PHP to resume the session via `session_start()`. For cross-origin requests, the `xhrFields: { withCredentials: true }` option must be set in jQuery, and the PHP server must send `Access-Control-Allow-Credentials: true`. Authentication state is typically synchronized by having a PHP endpoint (e.g., `check_session.php`) that returns the current login status as JSON, which jQuery uses to update the UI.

**Beginner-Friendly Explanation:** A PHP session is like a coat check ticket. When you log in, PHP gives you a ticket (the session cookie). Every time you make a request, you show the ticket, and PHP knows who you are. In a jQuery app, the browser automatically shows the ticket for same-origin requests. For cross-origin requests, you have to explicitly tell jQuery to include the ticket. The frontend also needs to ask PHP "Am I still logged in?" to keep the UI in sync.

### Purposes

- To maintain the user's login state across multiple AJAX requests.
- To authenticate AJAX requests and protect sensitive endpoints.
- To synchronize the frontend UI (e.g., showing/hiding admin buttons) with the backend session state.
- To handle session expiration gracefully (redirect to login).
- To support cross-origin authentication when the frontend and backend are on different domains.

### Syntax Rules and Structure

**Complete General Syntax (PHP Session Check):**
```php
<?php
// check_session.php
session_start();
header('Content-Type: application/json; charset=utf-8');

if (isset($_SESSION['user_id'])) {
    echo json_encode([
        'loggedIn' => true,
        'user' => [
            'id' => $_SESSION['user_id'],
            'name' => $_SESSION['user_name']
        ]
    ]);
} else {
    echo json_encode(['loggedIn' => false]);
}
?>
```

**Complete General Syntax (jQuery Session Check):**
```javascript
$.ajax({
    url: "check_session.php",
    dataType: "json",
    success: function(response) {
        if (response.loggedIn) {
            $("#userName").text(response.user.name);
            $(".admin-only").show();
        } else {
            $(".admin-only").hide();
            window.location.href = "login.php";
        }
    }
});
```

**Complete General Syntax (Cross-Origin Credentials):**
```javascript
$.ajax({
    url: "https://api.example.com/check_session.php",
    type: "GET",
    dataType: "json",
    xhrFields: {
        withCredentials: true  // Send cookies cross-origin
    },
    success: function(response) { ... }
});
```

| Component | Description |
|-----------|-------------|
| `session_start()` | Resumes the PHP session. |
| `$_SESSION['user_id']` | Stores the logged-in user's ID. |
| `xhrFields: { withCredentials: true }` | Sends cookies with cross-origin requests. |
| `Access-Control-Allow-Credentials: true` | PHP header for cross-origin credentials. |

**Syntax Rules:**

- Always call `session_start()` at the beginning of any PHP script that accesses session data.
- For cross-origin requests, set `xhrFields: { withCredentials: true }` in jQuery and `Access-Control-Allow-Credentials: true` in PHP.
- Use a dedicated `check_session.php` endpoint to synchronize login state with the frontend.
- Handle 401 (Unauthorized) responses by redirecting to the login page.
- Never store sensitive data in localStorage; rely on HttpOnly session cookies.

**Constraints and Limitations:**

- Cross-origin requests with credentials require specific CORS headers and cannot use `Access-Control-Allow-Origin: *`.
- Session cookies are not sent on cross-origin requests by default.
- PHP sessions are not shared across different domains or subdomains unless configured.
- Session expiration requires graceful handling on the frontend.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Login, Session Check, and Logout**

```html
<!-- index.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Session Auth Demo</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <div id="authStatus">Checking session...</div>
    <div id="userPanel" style="display:none;">
        <span id="userName"></span>
        <button id="logoutBtn">Logout</button>
    </div>
    <div id="loginPanel" style="display:none;">
        <input type="text" id="loginEmail" placeholder="Email">
        <input type="password" id="loginPassword" placeholder="Password">
        <button id="loginBtn">Login</button>
    </div>

    <script>
        $(function() {
            // Step 1: Check session on page load
            function checkSession() {
                $.getJSON("check_session.php", function(response) {
                    if (response.loggedIn) {
                        $("#authStatus").text("Logged in as:");
                        $("#userName").text(response.user.name);
                        $("#userPanel").show();
                        $("#loginPanel").hide();
                    } else {
                        $("#authStatus").text("Not logged in.");
                        $("#userPanel").hide();
                        $("#loginPanel").show();
                    }
                });
            }

            checkSession();

            // Step 2: Login
            $("#loginBtn").click(function() {
                $.post("login.php", {
                    email: $("#loginEmail").val(),
                    password: $("#loginPassword").val()
                }, function(response) {
                    if (response.success) {
                        checkSession();
                    } else {
                        alert("Login failed: " + response.message);
                    }
                }, "json");
            });

            // Step 3: Logout
            $("#logoutBtn").click(function() {
                $.post("logout.php", function() {
                    checkSession();
                });
            });
        });
    </script>
</body>
</html>
```

```php
<?php
// check_session.php
session_start();
header('Content-Type: application/json; charset=utf-8');

if (isset($_SESSION['user_id'])) {
    echo json_encode([
        'loggedIn' => true,
        'user' => [
            'id' => $_SESSION['user_id'],
            'name' => $_SESSION['user_name']
        ]
    ]);
} else {
    echo json_encode(['loggedIn' => false]);
}
?>
```

```php
<?php
// login.php
session_start();
header('Content-Type: application/json; charset=utf-8');

$email = isset($_POST['email']) ? trim($_POST['email']) : '';
$password = isset($_POST['password']) ? $_POST['password'] : '';

// Simulated credential check
if ($email === 'admin@example.com' && $password === 'password123') {
    $_SESSION['user_id'] = 1;
    $_SESSION['user_name'] = 'Admin User';
    echo json_encode(['success' => true]);
} else {
    echo json_encode(['success' => false, 'message' => 'Invalid credentials.']);
}
?>
```

```php
<?php
// logout.php
session_start();
session_destroy();
header('Content-Type: application/json; charset=utf-8');
echo json_encode(['success' => true]);
?>
```

**Expected Output:** On page load, "Checking session..." is displayed, then either the user panel (if logged in) or the login panel (if not). Logging in with `admin@example.com` / `password123` shows the user panel. Clicking "Logout" returns to the login panel.

**Why this output:** `check_session.php` reads `$_SESSION` and returns the login state. `login.php` validates credentials and sets session variables. `logout.php` destroys the session. jQuery coordinates the UI based on the session state.

### Real-World Cases

- **Admin dashboards:** Checking session before rendering admin-only controls.
- **E-commerce checkout:** Ensuring the user is logged in before proceeding.
- **Multi-tab synchronization:** Checking session state when the user switches tabs.
- **Session timeout handling:** Redirecting to login when the session expires.

---

## Core Concept 6: Header Management — Setting and Reading Custom HTTP Headers via `beforeSend`

### Definitions

**Core Definition:** Header management is the practice of setting custom HTTP headers in jQuery AJAX requests and reading them in PHP, commonly used for authentication tokens, CSRF protection, content negotiation, and API versioning.

**Technical Definition:** jQuery allows custom headers to be set via the `headers` option in `$.ajax()` or the `beforeSend` callback, which receives the `jqXHR` object and allows calling `xhr.setRequestHeader(name, value)`. On the PHP side, custom headers are accessed via `$_SERVER['HTTP_HEADER_NAME']` (with dashes converted to underscores and the `HTTP_` prefix added). For common headers like `Authorization`, PHP may use `$_SERVER['HTTP_AUTHORIZATION']` or `$_SERVER['REDIRECT_HTTP_AUTHORIZATION']` depending on the server configuration.

**Beginner-Friendly Explanation:** HTTP headers are like the labels on a package. They tell the server who sent the package, what is inside, and how to handle it. jQuery lets you add custom labels to your AJAX requests, and PHP reads those labels to decide what to do. For example, a "Authorization: Bearer token" label tells PHP that the request is from an authenticated user.

### Purposes

- To send authentication tokens (Bearer tokens, API keys) with AJAX requests.
- To include CSRF tokens for state-changing operations.
- To specify content type and accepted response formats.
- To support API versioning (e.g., `Accept: application/vnd.api.v2+json`).
- To pass custom metadata (request IDs, client version) for logging and debugging.

### Syntax Rules and Structure

**Complete General Syntax (jQuery — `headers` Option):**
```javascript
$.ajax({
    url: "api.php",
    type: "GET",
    headers: {
        "Authorization": "Bearer " + token,
        "X-CSRF-TOKEN": csrfToken,
        "Accept": "application/json"
    },
    success: function(response) { ... }
});
```

**Complete General Syntax (jQuery — `beforeSend` Callback):**
```javascript
$.ajax({
    url: "api.php",
    type: "GET",
    beforeSend: function(xhr) {
        xhr.setRequestHeader("Authorization", "Bearer " + token);
        xhr.setRequestHeader("X-Request-ID", generateRequestId());
    },
    success: function(response) { ... }
});
```

**Complete General Syntax (PHP — Reading Headers):**
```php
<?php
// Read custom headers in PHP
$authHeader = isset($_SERVER['HTTP_AUTHORIZATION']) ? $_SERVER['HTTP_AUTHORIZATION'] : '';
$csrfToken = isset($_SERVER['HTTP_X_CSRF_TOKEN']) ? $_SERVER['HTTP_X_CSRF_TOKEN'] : '';

// Extract Bearer token
if (preg_match('/Bearer\s+(.*)$/i', $authHeader, $matches)) {
    $token = $matches[1];
}

// For servers that use REDIRECT_HTTP_AUTHORIZATION
if (empty($authHeader) && isset($_SERVER['REDIRECT_HTTP_AUTHORIZATION'])) {
    $authHeader = $_SERVER['REDIRECT_HTTP_AUTHORIZATION'];
}
?>
```

| jQuery Option | PHP Access | Purpose |
|---------------|-----------|---------|
| `headers: { "Authorization": "Bearer x" }` | `$_SERVER['HTTP_AUTHORIZATION']` | Authentication |
| `headers: { "X-CSRF-TOKEN": "abc" }` | `$_SERVER['HTTP_X_CSRF_TOKEN']` | CSRF protection |
| `beforeSend: function(xhr) { ... }` | — | Dynamic header setting |
| `headers: { "Accept": "application/json" }` | `$_SERVER['HTTP_ACCEPT']` | Content negotiation |

**Syntax Rules:**

- Use the `headers` option for static headers; use `beforeSend` for dynamic headers.
- Header names in PHP are prefixed with `HTTP_` and have dashes replaced with underscores (e.g., `X-CSRF-TOKEN` → `HTTP_X_CSRF_TOKEN`).
- The `Authorization` header may be stripped by some servers; check `REDIRECT_HTTP_AUTHORIZATION` as a fallback.
- Custom headers trigger a CORS preflight (OPTIONS) request for cross-origin requests.
- Never send sensitive tokens over HTTP; always use HTTPS.

**Constraints and Limitations:**

- Custom headers on cross-origin requests require CORS preflight and `Access-Control-Allow-Headers` on the server.
- Some shared hosting environments strip the `Authorization` header; use `REDIRECT_HTTP_AUTHORIZATION` or send the token in the request body.
- Header names are case-insensitive but PHP normalizes them to uppercase with underscores.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

**Example 1: Bearer Token Authentication with Custom Headers**

```html
<!-- index.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Custom Header Demo</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <button id="fetchData">Fetch Protected Data</button>
    <div id="result"></div>

    <script>
        $(function() {
            // Simulated token (in production, obtain via login)
            var authToken = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...";

            $("#fetchData").click(function() {
                $.ajax({
                    url: "protected.php",
                    type: "GET",
                    headers: {
                        "Authorization": "Bearer " + authToken,
                        "X-Request-ID": "req-" + Date.now()
                    },
                    dataType: "json",
                    success: function(response) {
                        $("#result").html(
                            "Request ID: " + response.requestId + "<br>" +
                            "Message: " + response.message
                        );
                    },
                    error: function(jqXHR) {
                        $("#result").text("Error: " + jqXHR.status);
                    }
                });
            });
        });
    </script>
</body>
</html>
```

```php
<?php
// protected.php
header('Content-Type: application/json; charset=utf-8');

// Read the Authorization header
$authHeader = '';
if (isset($_SERVER['HTTP_AUTHORIZATION'])) {
    $authHeader = $_SERVER['HTTP_AUTHORIZATION'];
} elseif (isset($_SERVER['REDIRECT_HTTP_AUTHORIZATION'])) {
    $authHeader = $_SERVER['REDIRECT_HTTP_AUTHORIZATION'];
}

// Read the request ID header
$requestId = isset($_SERVER['HTTP_X_REQUEST_ID']) ? $_SERVER['HTTP_X_REQUEST_ID'] : 'unknown';

// Validate the token
if (preg_match('/Bearer\s+(.*)$/i', $authHeader, $matches)) {
    $token = $matches[1];
    // In production, validate the token against a database or JWT library
    if (!empty($token)) {
        echo json_encode([
            'requestId' => $requestId,
            'message' => 'Access granted to protected resource.'
        ]);
        exit;
    }
}

http_response_code(401);
echo json_encode(['message' => 'Unauthorized']);
?>
```

**Expected Output:** Clicking "Fetch Protected Data" displays the request ID and "Access granted to protected resource." If the token is missing or invalid, a 401 error is shown.

**Why this output:** jQuery sends the `Authorization` and `X-Request-ID` headers with the request. PHP reads them from `$_SERVER['HTTP_AUTHORIZATION']` and `$_SERVER['HTTP_X_REQUEST_ID']`, validates the token, and returns a JSON response.

### Real-World Cases

- **API authentication:** Sending Bearer tokens with every AJAX request.
- **CSRF protection:** Including `X-CSRF-TOKEN` headers for state-changing operations.
- **API versioning:** Using the `Accept` header to request a specific API version.
- **Request tracing:** Sending `X-Request-ID` headers for logging and debugging.

---

## References

- Passing PHP Arrays to jQuery — Tencent Cloud — https://cloud.tencent.cn/developer/information/将php数组传递给jquery-salon
- FormData and AJAX Form Submission — Stack Overflow — https://stackoverflow.com/revisions/0ac41b4e-1d58-4ecf-b1ed-6d773133b290/view-source
- DATATABLES PHP MYSQL CRUD — GitHub — https://github.com/VILHALVA/DATATABLES-PHP-MYSQL
- AdminLTE PHP Dashboard with Session Auth — GitHub — https://github.com/nirajang20/AdminLTE-PhP-Dashboard
- Custom Headers for AJAX and PHP — Stack Overflow — https://stackoverflow.com/revisions/0f6c8a0f-a832-4675-8208-ec8e28db78aa/view-source
- How to Correctly Parse and Use JSON Data Returned by PHP in jQuery — php.cn — https://global.php.cn/faq/1797021707.html
- PHP Cross-Domain POST Request — Session Cookie Persistence — Stack Overflow — https://stackoverflow.com/questions/79328699
- jQuery AJAX with Credentials (withCredentials) — Tencent Cloud — https://cloud.tencent.cn/developer/information/ajax域名导致session
- Sending OAuth Access Token in jQuery AJAX Request — Stack Overflow — https://stackoverflow.com/revisions/2f7d2f3f
- PHP json_encode and jQuery AJAX — Stack Overflow — https://stackoverflow.com/revisions/5b95834f-629d-40dc-b3c9-9e7a4f1119e4/view-source
- jQuery FormData PHP $_FILES Upload — Stack Overflow — https://stackoverflow.com/revisions/2
- How to Update Data in a Table Using jQuery, AJAX, PHP, and MySQL — Tencent Cloud — https://cloud.tencent.cn/developer/information/如何使用jquery、ajax、php和mysql更新表中的数据
- Server-Side AJAX jQuery CRUD DataTable — Tencent Cloud — https://cloud.tencent.cn/developer/information/服务器端Ajax JQuery CRUD DataTable
- jQuery AJAX CRUD with PHP MySQLi — GitHub — https://github.com/shindesharad71/CRUD-PHP-JQuery-AJAX
- How to Use jQuery's beforeSend Function — Onelinerhub — https://github.com/Onelinerhub/onelinerhub/blob/master/jquery/send--how-can-i-use-jquery-s-beforesend-function.md
- Cross-Domain User Verification Failure — PHP Handling Authorization Header — php.cn — https://www.php.cn/faq/1796947443.html
- PHP Cross-Domain Session Loss — PHP Frontend/Backend Separation — php.cn — https://www.php.cn/faq/1796947443.html
- How to Implement Multiple File Uploads in PHP Using jQuery — php.cn — https://global.php.cn/faq/1797009444.html
- Multiple File Upload with FormData — Stack Overflow — https://stackoverflow.com/revisions/0
- jQuery AJAX CRUD PHP MySQL Tutorial — Tencent Cloud — https://cloud.tencent.cn/developer/information/如何使用jquery、ajax、php和mysql更新表中的数据
- PHP json_encode with JSON_UNESCAPED_UNICODE and JSON_UNESCAPED_SLASHES — php.cn — https://global.php.cn/faq/1797021707.html