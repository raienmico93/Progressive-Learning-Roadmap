# PHP Advanced File Upload Management — Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**  
PHP Advanced File Upload Management is the practice of securely receiving files from HTTP clients, validating their content and metadata, and storing them on the server without introducing remote code execution, filesystem traversal, or denial-of-service vulnerabilities.

**Technical Definition**  
It encompasses the `$_FILES` superglobal structure (populated by PHP's SAPI for `multipart/form-data` POST requests), the `move_uploaded_file()` function that validates the temporary upload source, the Filter extension's `finfo_file()` function for server-side MIME type detection, `php.ini` directives (`upload_max_filesize`, `post_max_size`, `max_file_uploads`, `file_uploads`), and defensive filename generation using cryptographic randomness. It operates on the principle that the client controls every byte of an upload — filename, stated MIME type, and content — and the server must independently verify all of them before persistence.

**Beginner-Friendly Explanation**  
When a user uploads a file, PHP saves it to a temporary location and puts information about it into `$_FILES`. The problem is that the client (the browser or a malicious script) can lie about the file's name and type. An attacker might upload a file called `photo.jpg` that actually contains PHP code. If your server saves it in a web-accessible folder and the server executes `.php` files, the attacker can run commands on your server. Secure file upload management is the checklist that prevents this: check the error code, verify the real content type by inspecting the bytes, generate a random safe filename, and store the file where it cannot be executed.

### Key Characteristics

- `$_FILES` is populated only for POST requests with `enctype="multipart/form-data"`.
- The client controls `name`, `type`, and `full_path`; PHP controls `tmp_name` and `size`.
- `$_FILES['type']` is supplied by the browser and is trivially spoofed; it must never be trusted.
- `finfo_file()` inspects the actual bytes (magic numbers) to determine the real MIME type.
- `move_uploaded_file()` verifies that the source file is a genuine HTTP upload before moving it.
- Uploaded files are stored in a temporary directory and are deleted at the end of the request if not moved.
- `upload_max_filesize` limits each file; `post_max_size` limits the entire POST body.
- `max_file_uploads` limits the number of files per request (default 20).
- Safe filename generation using `random_bytes()` prevents path traversal, double-extension attacks, and overwrites.
- Storing uploads outside the web root or in a directory with execution disabled prevents webshell execution.
- Deprecated/unsafe: trusting `$_FILES['type']`, using the client filename on disk, blacklist-based extension filtering, `copy()` instead of `move_uploaded_file()`.

### Prerequisites

- Basic PHP syntax: superglobals, functions, file system operations.
- Basic HTTP: multipart/form-data, POST requests.
- HTML forms: `enctype="multipart/form-data"`, `<input type="file">`.
- A web server with PHP installed (Apache, Nginx + PHP-FPM, or PHP built-in server).
- Write access to a storage directory outside or protected within the web root.
- The `fileinfo` extension enabled (default in modern PHP).

### Related Programming Areas

- HTTP protocol and multipart form encoding (RFC 7578).
- Web application security (OWASP Top 10: A05 Security Misconfiguration, A04 Insecure Design).
- Filesystem operations and permissions.
- MIME type detection and magic numbers.
- Content Delivery Networks (CDNs) and object storage (S3, Azure Blob).
- Image processing and validation libraries.

---

## Core Concepts / Features

## 1. The `$_FILES` Blueprint — Navigating File Field Metadata

### Definitions

**Core Definition**  
`$_FILES` is a superglobal associative array that PHP populates with metadata about files uploaded via a `multipart/form-data` POST request.

**Technical Definition**  
For each file input in the form, `$_FILES['field_name']` contains an array with the keys `name` (original client filename), `type` (client-supplied MIME type), `size` (file size in bytes), `tmp_name` (server-side temporary path), `error` (upload error code), and — as of PHP 8.1.0 — `full_path` (full path as submitted by the browser). For array field names (`photos[]`), each key becomes an array indexed by file position. Only `tmp_name` and `size` are set by PHP itself; `name`, `type`, and `full_path` come from the client and are untrusted. The `error` key contains one of the `UPLOAD_ERR_*` constants.

**Beginner-Friendly Explanation**  
When you upload a file, PHP saves it to a temporary location on the server and gives you a small "receipt" with information about it. The receipt tells you the original filename (`name`), the file size (`size`), where the temporary file is (`tmp_name`), an error code (`error`), and what the browser claims the file type is (`type`). The problem is that the browser can lie about the name and type. Only `tmp_name` and `size` are set by PHP itself, so those are the only values you can start to trust.

### Purposes

- To receive uploaded file content from HTML forms.
- To validate file size, error status, and content type.
- To move the temporary file to permanent storage.
- To handle multiple file uploads under one field name.
- To detect upload failures (size exceeded, partial upload, no file).
- To store file metadata for database records.
- To reject malicious or oversized uploads before processing.

### Syntax Rules and Structure

**General Syntax: `$_FILES` Structure (Single File)**
```php
$_FILES['field'] = [
    'name'      => 'photo.jpg',       // original client filename (UNTRUSTED)
    'type'      => 'image/jpeg',      // client-supplied MIME type (UNTRUSTED)
    'size'      => 123456,            // bytes (set by PHP)
    'tmp_name'  => '/tmp/phpABC123',  // temporary server path (set by PHP)
    'error'     => UPLOAD_ERR_OK,     // error code (set by PHP)
    'full_path' => 'C:\...\photo.jpg' // PHP 8.1.0+; UNTRUSTED
];
```

**Component Breakdown**

- `name` — original filename from the client; can contain path traversal sequences (`../`), null bytes (on older PHP), and other dangerous characters.
- `type` — MIME type supplied by the browser; trivially spoofed via the `Content-Type` header.
- `size` — file size in bytes as measured by PHP.
- `tmp_name` — temporary server path; trustworthy.
- `error` — one of the `UPLOAD_ERR_*` constants.
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

**General Syntax: Upload Error Codes**
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

**General Syntax: Checking Upload Errors**
```php
if ($_FILES['field']['error'] !== UPLOAD_ERR_OK) {
    // Handle error based on code.
}
```

**Component Breakdown**

- Always check `error` before using any other key.
- `$_FILES['field']['size']` is `0` and `tmp_name` is empty if no file was selected.

**Syntax Rules**

- `$_FILES` is populated only for POST requests with `enctype="multipart/form-data"`.
- The form must use `method="post"`.
- `name`, `type`, and `full_path` are untrusted; only `tmp_name` and `size` are set by PHP.
- For array field names, each key becomes an array.
- The temporary file is deleted at the end of the request if not moved.
- `$_FILES` is empty if `post_max_size` is exceeded.
- `full_path` is available only as of PHP 8.1.0.

**Constraints and Limitations**

- `$_FILES` is empty if the form does not use `multipart/form-data`.
- `$_FILES` is empty if `post_max_size` is exceeded.
- The temporary file is deleted at the end of the request if not moved.
- `type` is browser-supplied and can be forged.
- `name` can contain path traversal sequences and null bytes.
- `full_path` is untrusted and should never be used for filesystem operations.
- `$_FILES` does not include files if `file_uploads` is off.

### Annotated Code Examples and Expected Outputs

#### Example 1: Inspecting `$_FILES` Metadata

**File: `inspect.php`**
```php
<?php
if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_FILES['upload'])) {
    $file = $_FILES['upload'];

    // Always check the error code first.
    if ($file['error'] !== UPLOAD_ERR_OK) {
        echo 'Upload error code: ' . (int)$file['error'];
        exit;
    }

    // Display metadata (escape for HTML output).
    echo 'Name: ' . htmlspecialchars($file['name'], ENT_QUOTES, 'UTF-8') . '<br>';
    echo 'Type (untrusted): ' . htmlspecialchars($file['type'], ENT_QUOTES, 'UTF-8') . '<br>';
    echo 'Size: ' . (int)$file['size'] . ' bytes<br>';
    echo 'Temporary path: ' . htmlspecialchars($file['tmp_name'], ENT_QUOTES, 'UTF-8');
}
?>

<form method="post" action="inspect.php" enctype="multipart/form-data">
  <input type="file" name="upload">
  <button type="submit">Upload</button>
</form>
```

**Expected Output**
```
Name: photo.jpg
Type (untrusted): image/jpeg
Size: 123456 bytes
Temporary path: /tmp/phpABC123
```

**Why This Output Occurs**

- `$_FILES['upload']` contains the metadata for the uploaded file.
- `name`, `type`, and `size` are displayed (with escaping).
- `tmp_name` is the server-side temporary path.
- The error code is checked first; if it is not `UPLOAD_ERR_OK`, the script exits.

### Real-World Cases

- **Avatar uploads:** Inspect `name`, `size`, and `type` before validation.
- **Document management:** Check `error` to detect size-limit failures.
- **Multi-file galleries:** Iterate over array keys in `$_FILES['photos']`.
- **Debugging:** Log `$_FILES` to understand upload behavior.
- **API endpoints:** Validate `$_FILES` before processing.

---

## 2. Upload Security Safeguards

### 2.1 Handling Upload Limit Settings in `php.ini`

#### Definitions

**Core Definition**  
Upload limit settings in `php.ini` control the maximum size of individual files, the maximum size of the entire POST body, and the number of files that can be uploaded in a single request.

**Technical Definition**  
`upload_max_filesize` sets the maximum size of a single uploaded file (default 2M). `post_max_size` sets the maximum size of the entire POST body, including file uploads and regular fields (default 8M). `post_max_size` must be larger than `upload_max_filesize`. `max_file_uploads` limits the number of files per request (default 20). `file_uploads` enables or disables file uploads entirely (default On). `upload_tmp_dir` specifies the temporary directory. `max_input_time` limits the time spent parsing input. If `post_max_size` is exceeded, `$_POST` and `$_FILES` are empty and `$_SERVER['CONTENT_LENGTH']` exceeds the limit.

**Beginner-Friendly Explanation**  
These settings are like the rules at a shipping counter: how heavy can one package be (`upload_max_filesize`), how heavy can the whole shipment be (`post_max_size`), and how many packages can you send at once (`max_file_uploads`). If you try to send something too heavy, the shipment is rejected before it even reaches your code. You need to configure these settings on the server and also validate sizes in your PHP code.

#### Purposes

- To prevent denial-of-service attacks via disk-filling uploads.
- To enforce business rules on maximum file sizes.
- To control server resource consumption (memory, disk, bandwidth).
- To provide meaningful error messages when limits are exceeded.
- To align upload limits with application requirements.
- To prevent accidental uploads of huge files.

#### Syntax Rules and Structure

**General Syntax: `php.ini` Configuration**
```ini
file_uploads = On
upload_max_filesize = 20M
post_max_size = 25M
max_file_uploads = 50
memory_limit = 256M
max_execution_time = 300
upload_tmp_dir = /var/tmp
```

**Component Breakdown**

- `file_uploads` — enables or disables file uploads.
- `upload_max_filesize` — maximum size per file.
- `post_max_size` — maximum size of the entire POST body; must be larger than `upload_max_filesize`.
- `max_file_uploads` — maximum number of files per request.
- `memory_limit` — maximum memory a script can consume.
- `max_execution_time` — maximum script execution time.
- `upload_tmp_dir` — temporary directory for uploaded files.

**General Syntax: `.htaccess` Override (Apache)**
```apache
<IfModule mod_php.c>
    php_value upload_max_filesize 20M
    php_value post_max_size 25M
    php_value max_file_uploads 50
</IfModule>
```

**Component Breakdown**

- Overrides `php.ini` for the directory.
- Only works with mod_php; PHP-FPM requires pool configuration.

**General Syntax: PHP-FPM Pool Configuration**
```ini
php_admin_value[upload_max_filesize] = 20M
php_admin_value[post_max_size] = 25M
php_admin_value[max_file_uploads] = 50
```

**Component Breakdown**

- `php_admin_value` — sets values that cannot be overridden by `ini_set()`.

**General Syntax: Runtime Check**
```php
$maxBytes = ini_get('upload_max_filesize');
echo 'Upload max: ' . $maxBytes;
```

**Component Breakdown**

- `ini_get()` — reads the current value of a configuration directive.
- Values are strings with shorthand notation (e.g., `20M`).

**General Syntax: Converting Shorthand Notation**
```php
function parseBytes(string $value): int {
    $value = trim($value);
    $last = strtolower($value[strlen($value) - 1]);
    $num = (int)$value;
    return match ($last) {
        'g' => $num * 1024 * 1024 * 1024,
        'm' => $num * 1024 * 1024,
        'k' => $num * 1024,
        default => $num,
    };
}
```

**Component Breakdown**

- Converts `php.ini` shorthand (`20M`, `1G`) to bytes.
- Useful for comparing with `$_FILES['size']`.

**Syntax Rules**

- `post_max_size` must be larger than `upload_max_filesize`.
- `max_file_uploads` limits the number of files per request.
- If `post_max_size` is exceeded, `$_POST` and `$_FILES` are empty.
- `upload_max_filesize` can be set to `-1` for unlimited (not recommended).
- `.htaccess` overrides only work with mod_php.
- PHP-FPM uses pool configuration.
- `ini_set()` cannot change `upload_max_filesize` or `post_max_size` at runtime for the current request.

**Constraints and Limitations**

- `ini_set()` cannot change `upload_max_filesize` or `post_max_size` at runtime.
- `.htaccess` overrides only work with mod_php.
- PHP-FPM ignores `.htaccess` for PHP settings.
- Default values (2M / 8M) may be too small for many applications.
- Large `post_max_size` values can increase memory usage.
- `max_file_uploads` defaults to 20; increase for multi-file uploads.
- The temporary directory must have sufficient disk space and write permissions.

#### Annotated Code Examples and Expected Outputs

##### Example 1: Checking Current Upload Limits

**File: `check_limits.php`**
```php
<?php
// Read and display current upload limits.
$uploadMax = ini_get('upload_max_filesize');
$postMax   = ini_get('post_max_size');
$maxFiles  = ini_get('max_file_uploads');
$fileUploads = ini_get('file_uploads');

echo "upload_max_filesize: $uploadMax<br>";
echo "post_max_size: $postMax<br>";
echo "max_file_uploads: $maxFiles<br>";
echo "file_uploads: $fileUploads<br>";

// Convert to bytes for comparison.
function parseBytes(string $value): int {
    $value = trim($value);
    $last = strtolower($value[strlen($value) - 1]);
    $num = (int)$value;
    return match ($last) {
        'g' => $num * 1024 * 1024 * 1024,
        'm' => $num * 1024 * 1024,
        'k' => $num * 1024,
        default => $num,
    };
}

echo 'Upload max in bytes: ' . parseBytes($uploadMax);
?>
```

**Expected Output**
```
upload_max_filesize: 20M
post_max_size: 25M
max_file_uploads: 50
file_uploads: 1
Upload max in bytes: 20971520
```

**Why This Output Occurs**

- `ini_get()` reads the current configuration values.
- `parseBytes()` converts the shorthand notation to bytes.

##### Example 2: Handling `post_max_size` Exceeded

**File: `upload.php`**
```php
<?php
// Check if POST data was truncated due to post_max_size.
if ($_SERVER['REQUEST_METHOD'] === 'POST' && empty($_POST) && empty($_FILES) && ($_SERVER['CONTENT_LENGTH'] ?? 0) > 0) {
    echo 'The uploaded data exceeded the post_max_size limit.';
    exit;
}

if (isset($_FILES['upload']) && $_FILES['upload']['error'] === UPLOAD_ERR_OK) {
    echo 'File received.';
} else {
    echo 'No file uploaded or upload error.';
}
?>
<form method="post" action="upload.php" enctype="multipart/form-data">
  <input type="file" name="upload">
  <button type="submit">Upload</button>
</form>
```

**Expected Output (post_max_size exceeded)**
```
The uploaded data exceeded the post_max_size limit.
```

**Why This Output Occurs**

- When `post_max_size` is exceeded, `$_POST` and `$_FILES` are empty.
- `$_SERVER['CONTENT_LENGTH']` still contains the original content length.
- The script detects this condition and displays a meaningful error.

#### Real-World Cases

- **Video uploads:** Set `upload_max_filesize` to 500M and `post_max_size` to 550M.
- **Image galleries:** Set `max_file_uploads` to 100 for batch uploads.
- **Document management:** Set `upload_max_filesize` to 50M for PDFs.
- **Shared hosting:** Use `.htaccess` to override limits when `php.ini` is not accessible.
- **Docker/Kubernetes:** Set limits via PHP-FPM pool configuration or environment-specific `php.ini`.

---

### 2.2 Moving Files Securely via `move_uploaded_file()`

#### Definitions

**Core Definition**  
`move_uploaded_file()` is a PHP function that moves an uploaded file from its temporary location to a permanent destination, verifying that the source is a genuine HTTP upload.

**Technical Definition**  
`move_uploaded_file(string $from, string $to): bool` checks that the file at `$from` was uploaded via HTTP POST (using `is_uploaded_file()` internally) and, if so, moves it to `$to`. If the check fails, it returns `false` and does not move the file. This prevents attackers from tricking the script into moving arbitrary files (e.g., `/etc/passwd`) by manipulating the `tmp_name` value. It must be used instead of `copy()`, `rename()`, or `file_put_contents()` for uploaded files.

**Beginner-Friendly Explanation**  
`move_uploaded_file()` is a special function that only works on files that were genuinely uploaded through an HTTP POST request. It checks the "receipt" (the temporary file marker) before moving the file. If an attacker tries to trick your script into moving a system file by changing the `tmp_name` value, `move_uploaded_file()` refuses to move it. This is why you should never use `copy()` or `rename()` for uploaded files.

#### Purposes

- To safely move an uploaded file from temporary to permanent storage.
- To verify that the source file is a genuine HTTP upload.
- To prevent filesystem traversal attacks via `tmp_name` manipulation.
- To ensure the file is written atomically to the destination.
- To integrate with PHP's upload error handling.
- To comply with secure coding standards.

#### Syntax Rules and Structure

**General Syntax: `move_uploaded_file()`**
```php
$success = move_uploaded_file(
    $tmp_name,      // source: $_FILES['field']['tmp_name']
    $destination    // target: absolute path
);
```

**Component Breakdown**

- `$tmp_name` — the temporary path from `$_FILES`.
- `$destination` — the target path (absolute or relative).
- Returns `true` on success, `false` on failure.
- Internally calls `is_uploaded_file()` to verify the source.

**General Syntax: Full Upload Handler**
```php
if ($_FILES['upload']['error'] === UPLOAD_ERR_OK) {
    $tmp  = $_FILES['upload']['tmp_name'];
    $dest = __DIR__ . '/uploads/' . $safeName;
    if (move_uploaded_file($tmp, $dest)) {
        echo 'File moved successfully.';
    } else {
        echo 'Failed to move file.';
    }
}
```

**Component Breakdown**

- Check `error` first.
- Use `tmp_name` as the source.
- Use an absolute destination path.
- Check the return value.

**General Syntax: `is_uploaded_file()` (Explicit Check)**
```php
if (is_uploaded_file($_FILES['upload']['tmp_name'])) {
    // Source is a genuine upload.
}
```

**Component Breakdown**

- `is_uploaded_file()` returns `true` if the file was uploaded via HTTP POST.
- `move_uploaded_file()` performs this check internally; explicit calls are optional.

**Syntax Rules**

- Always use `move_uploaded_file()` for uploaded files.
- Never use `copy()`, `rename()`, or `file_put_contents()` on `tmp_name`.
- Check the return value; it returns `false` on failure.
- Use an absolute destination path to avoid ambiguity.
- Ensure the destination directory exists and is writable.
- Use `basename()` and sanitization on the destination filename.
- Store uploads outside the web root or in a directory with execution disabled.

**Constraints and Limitations**

- `move_uploaded_file()` only works on files uploaded via HTTP POST.
- It does not validate the file's content or MIME type.
- It does not protect against malicious file content (e.g., PHP code in an image).
- It does not create the destination directory; it must exist.
- It does not check for filename collisions; use `random_bytes()` for safe names.
- It returns `false` if the destination is not writable.
- It is not available in CLI SAPI for genuine uploads.

#### Annotated Code Examples and Expected Outputs

##### Example 1: Secure File Move

**File: `move_upload.php`**
```php
<?php
if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_FILES['upload'])) {
    $file = $_FILES['upload'];

    // Check error code.
    if ($file['error'] !== UPLOAD_ERR_OK) {
        echo 'Upload error: ' . (int)$file['error'];
        exit;
    }

    // Generate a safe filename.
    $safeName = bin2hex(random_bytes(16)) . '.dat';
    $uploadDir = __DIR__ . '/uploads';

    if (!is_dir($uploadDir)) {
        mkdir($uploadDir, 0755, true);
    }

    $destination = $uploadDir . '/' . $safeName;

    // Move the file safely.
    if (move_uploaded_file($file['tmp_name'], $destination)) {
        echo 'File saved as: ' . htmlspecialchars($safeName, ENT_QUOTES, 'UTF-8');
    } else {
        echo 'Failed to move uploaded file.';
    }
}
?>
<form method="post" action="move_upload.php" enctype="multipart/form-data">
  <input type="file" name="upload">
  <button type="submit">Upload</button>
</form>
```

**Expected Output**
```
File saved as: a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6.dat
```

**Why This Output Occurs**

- `move_uploaded_file()` verifies the source and moves it.
- The filename is randomly generated, preventing path traversal and overwrites.
- The destination is an absolute path.

##### Example 2: Preventing `tmp_name` Manipulation

**File: `fake_move.php`**
```php
<?php
// Simulate an attacker trying to move /etc/passwd.
$fakeTmp = '/etc/passwd';
$destination = __DIR__ . '/uploads/stolen.txt';

if (move_uploaded_file($fakeTmp, $destination)) {
    echo 'Moved!';
} else {
    echo 'Failed: source is not a genuine upload.';
}
?>
```

**Expected Output**
```
Failed: source is not a genuine upload.
```

**Why This Output Occurs**

- `move_uploaded_file()` calls `is_uploaded_file()` internally.
- `/etc/passwd` is not an uploaded file, so the function returns `false`.
- The file is not moved.

#### Real-World Cases

- **Avatar uploads:** Move the file to a public images directory.
- **Document management:** Move PDFs to a secure storage directory.
- **E-commerce:** Move product images to a CDN-synced directory.
- **CMS:** Move media library assets to an uploads folder.
- **API endpoints:** Move files to object storage (S3) after validation.

---

### 2.3 MIME-Type Spoofing Protection — Enforcing Native Validation via `finfo_file()`

#### Definitions

**Core Definition**  
MIME-type spoofing is an attack where the client supplies a false `Content-Type` header (reflected in `$_FILES['type']`) to make a malicious file appear benign. Native validation using `finfo_file()` inspects the actual bytes of the file to determine its real MIME type.

**Technical Definition**  
`finfo_file()` (or the `finfo` class) uses the libmagic library to examine a file's magic numbers — the first few bytes that identify its format — and returns the detected MIME type. It ignores the client-supplied `Content-Type` header. The function `finfo_open(FILEINFO_MIME_TYPE)` creates a resource; `finfo_file($finfo, $path)` returns the MIME type. An allowlist of expected MIME types is compared against the detected type. `finfo_file()` can be tricked in rare cases (e.g., files with ambiguous magic numbers), so it should be combined with extension validation and content inspection where possible.

**Beginner-Friendly Explanation**  
When you upload a file, the browser tells the server what type it thinks the file is (e.g., `image/jpeg`). But an attacker can lie and say a PHP script is an image. `finfo_file()` ignores what the browser says and looks at the actual bytes of the file to figure out what it really is. It is like opening a package and looking inside instead of trusting the label. You then compare the detected type against a list of allowed types (e.g., `image/jpeg`, `image/png`) and reject anything else.

#### Purposes

- To prevent attackers from uploading executable files disguised as images.
- To verify the true content type independent of client claims.
- To enforce an allowlist of permitted file types.
- To prevent XSS via SVG or HTML uploads.
- To comply with OWASP file upload security guidelines.
- To complement extension and size validation.

#### Syntax Rules and Structure

**General Syntax: `finfo_file()`**
```php
$finfo = new finfo(FILEINFO_MIME_TYPE);
$mimeType = $finfo->file($tmp_name);
```

**Component Breakdown**

- `new finfo(FILEINFO_MIME_TYPE)` — creates a finfo resource for MIME type detection.
- `$finfo->file($tmp_name)` — returns the detected MIME type.
- `$tmp_name` — the temporary path from `$_FILES`.

**General Syntax: Procedural Style**
```php
$finfo = finfo_open(FILEINFO_MIME_TYPE);
$mimeType = finfo_file($finfo, $tmp_name);
finfo_close($finfo);
```

**Component Breakdown**

- `finfo_open()` — opens a finfo resource.
- `finfo_file()` — detects the MIME type.
- `finfo_close()` — frees the resource.

**General Syntax: Allowlist Comparison**
```php
$allowed = [
    'image/jpeg' => 'jpg',
    'image/png'  => 'png',
    'image/webp' => 'webp',
    'application/pdf' => 'pdf'
];

$finfo = new finfo(FILEINFO_MIME_TYPE);
$mimeType = $finfo->file($tmp_name);

if (!isset($allowed[$mimeType])) {
    throw new RuntimeException("Unsupported file type: $mimeType");
}

$extension = $allowed[$mimeType];
```

**Component Breakdown**

- `$allowed` — associative array mapping MIME types to safe extensions.
- `isset($allowed[$mimeType])` — checks if the detected type is allowed.
- `$extension` — the safe extension to use for the stored file.

**General Syntax: Buffer-Based Detection**
```php
$finfo = new finfo(FILEINFO_MIME_TYPE);
$contents = file_get_contents($tmp_name);
$mimeType = $finfo->buffer($contents);
```

**Component Breakdown**

- `$finfo->buffer()` — detects MIME type from a string buffer.
- Uses more memory but useful when the file is already in memory.

**Syntax Rules**

- Always use `finfo_file()` or `mime_content_type()` instead of `$_FILES['type']`.
- Use `FILEINFO_MIME_TYPE` to get the MIME type without additional parameters.
- Compare against an allowlist, not a blocklist.
- Map allowed MIME types to safe extensions.
- Use the detected extension, not the client-supplied filename extension.
- Combine with extension validation and size checks.
- Be aware that `finfo_file()` can be tricked by polyglot files.

**Constraints and Limitations**

- `finfo_file()` can be tricked by files with ambiguous magic numbers (polyglots).
- It may return `application/octet-stream` for some valid files.
- It requires the `fileinfo` extension (enabled by default in modern PHP).
- It does not validate the file content beyond the magic number.
- It may return different results across libmagic versions.
- It does not protect against all malicious content (e.g., SVG with embedded JavaScript).
- It should be combined with other validation layers.

#### Annotated Code Examples and Expected Outputs

##### Example 1: Detecting Spoofed MIME Type

**File: `mime_check.php`**
```php
<?php
if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_FILES['upload'])) {
    $file = $_FILES['upload'];

    if ($file['error'] !== UPLOAD_ERR_OK) {
        echo 'Upload error: ' . (int)$file['error'];
        exit;
    }

    // Client-supplied type (untrusted).
    $clientType = $file['type'];

    // Native detection (trusted).
    $finfo = new finfo(FILEINFO_MIME_TYPE);
    $realType = $finfo->file($file['tmp_name']);

    echo 'Client claims: ' . htmlspecialchars($clientType, ENT_QUOTES, 'UTF-8') . '<br>';
    echo 'Actual type: ' . htmlspecialchars($realType, ENT_QUOTES, 'UTF-8');
}
?>
<form method="post" action="mime_check.php" enctype="multipart/form-data">
  <input type="file" name="upload">
  <button type="submit">Upload</button>
</form>
```

**Expected Output (spoofed upload)**
```
Client claims: image/jpeg
Actual type: text/x-php
```

**Why This Output Occurs**

- The client sends `Content-Type: image/jpeg` in the multipart request.
- `$_FILES['upload']['type']` reflects the client claim.
- `finfo_file()` inspects the bytes and detects PHP code.
- The script reveals the discrepancy.

##### Example 2: Enforcing an Allowlist

**File: `secure_mime.php`**
```php
<?php
$allowed = [
    'image/jpeg' => 'jpg',
    'image/png'  => 'png',
    'image/webp' => 'webp',
    'application/pdf' => 'pdf'
];

if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_FILES['upload'])) {
    $file = $_FILES['upload'];

    if ($file['error'] !== UPLOAD_ERR_OK) {
        echo 'Upload error: ' . (int)$file['error'];
        exit;
    }

    $finfo = new finfo(FILEINFO_MIME_TYPE);
    $mimeType = $finfo->file($file['tmp_name']);

    if (!isset($allowed[$mimeType])) {
        echo 'Unsupported file type: ' . htmlspecialchars($mimeType, ENT_QUOTES, 'UTF-8');
        exit;
    }

    $extension = $allowed[$mimeType];
    echo 'Accepted. Safe extension: ' . htmlspecialchars($extension, ENT_QUOTES, 'UTF-8');
}
?>
<form method="post" action="secure_mime.php" enctype="multipart/form-data">
  <input type="file" name="upload">
  <button type="submit">Upload</button>
</form>
```

**Expected Output (valid image)**
```
Accepted. Safe extension: jpg
```
**Expected Output (PHP file)**
```
Unsupported file type: text/x-php
```

**Why This Output Occurs**

- `finfo_file()` detects the real MIME type.
- The allowlist maps allowed types to safe extensions.
- Unsupported types are rejected.

#### Real-World Cases

- **Avatar uploads:** Allow only `image/jpeg`, `image/png`, `image/webp`.
- **Document management:** Allow only `application/pdf`.
- **Video uploads:** Allow only `video/mp4`, `video/webm`.
- **CMS:** Validate all media uploads against an allowlist.
- **API endpoints:** Reject any file that does not match expected MIME types.

---

### 2.4 Safe Name Generation — Avoiding Execution Directory Traps

#### Definitions

**Core Definition**  
Safe name generation is the practice of discarding the client-supplied filename and generating a new, random, safe filename for the stored file, preventing path traversal, double-extension attacks, overwrites, and execution in web-accessible directories.

**Technical Definition**  
The client-supplied `$_FILES['name']` can contain path traversal sequences (`../`), null bytes, double extensions (`shell.php.jpg`), and other dangerous characters. Safe name generation uses `random_bytes()` to produce a cryptographically random string, concatenated with an extension derived from the validated MIME type (not the client filename). The resulting filename is stored in a directory outside the web root or in a directory with execution disabled. The original filename is discarded or stored in a database for display purposes only.

**Beginner-Friendly Explanation**  
Imagine an attacker uploads a file named `../../etc/passwd` or `shell.php.jpg`. If your server uses that name directly, the attacker can overwrite system files or trick the server into executing PHP code. Safe name generation fixes this by throwing away the attacker's name and creating a new one from random characters. For example, instead of `shell.php.jpg`, the file becomes `a1b2c3d4e5f6.jpg`. The attacker cannot predict or control the name, and the file cannot be executed because the extension is safe and the directory has execution disabled.

#### Purposes

- To prevent path traversal attacks via malicious filenames.
- To prevent double-extension attacks (`shell.php.jpg`).
- To prevent overwriting existing files.
- To prevent execution of uploaded files in web-accessible directories.
- To make filenames unpredictable to attackers.
- To separate storage names from display names.

#### Syntax Rules and Structure

**General Syntax: Generating a Random Name**
```php
$extension = $allowed[$mimeType]; // safe extension from MIME allowlist
$safeName = bin2hex(random_bytes(16)) . '.' . $extension;
```

**Component Breakdown**

- `random_bytes(16)` — generates 16 cryptographically secure random bytes.
- `bin2hex()` — converts to a 32-character hexadecimal string.
- `$extension` — derived from the validated MIME type, not the client filename.
- `$safeName` — the final filename (e.g., `a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6.jpg`).

**General Syntax: Alternative with `uniqid()` (Not Recommended for Security)**
```php
$safeName = uniqid('upload_', true) . '.' . $extension;
```

**Component Breakdown**

- `uniqid()` — generates a unique ID based on the current time in microseconds.
- Not cryptographically secure; predictable.
- Use `random_bytes()` instead.

**General Syntax: Sanitizing the Original Filename (for Display)**
```php
$originalName = basename($_FILES['upload']['name']);
$originalName = preg_replace('/[^A-Za-z0-9._-]/', '_', $originalName);
$originalName = substr($originalName, 0, 255);
```

**Component Breakdown**

- `basename()` — strips directory components.
- `preg_replace()` — replaces unsafe characters with underscores.
- `substr()` — limits length to 255 characters.
- Used for display only; never used for filesystem storage.

**General Syntax: Storing Outside the Web Root**
```php
$storageDir = '/var/www/storage/uploads'; // outside document root
$publicDir = '/var/www/html/uploads';     // web-accessible
```

**Component Breakdown**

- `$storageDir` — outside the web root; cannot be accessed via URL.
- `$publicDir` — web-accessible; should have execution disabled.

**General Syntax: Disabling Execution in `.htaccess`**
```apache
# In the uploads directory
php_flag engine off
<FilesMatch "\.php$">
    Require all denied
</FilesMatch>
```

**Component Breakdown**

- `php_flag engine off` — disables PHP execution in the directory.
- `FilesMatch` — denies access to `.php` files.

**Syntax Rules**

- Never use the client-supplied filename for filesystem storage.
- Use `random_bytes()` for cryptographically secure random names.
- Derive the extension from the validated MIME type, not the client filename.
- Store uploads outside the web root or in a directory with execution disabled.
- Limit filename length to 255 characters.
- Use `basename()` and sanitization for display names.
- Store the original filename in a database if needed.
- Use `mkdir($dir, 0755, true)` to create the storage directory.
- Set appropriate file permissions (e.g., `0644`).

**Constraints and Limitations**

- `random_bytes()` can throw an exception if no source of randomness is available.
- Very long filenames may exceed filesystem limits.
- Some filesystems are case-insensitive; random hex avoids case issues.
- Storing outside the web root requires a controller to serve files.
- Disabling execution via `.htaccess` only works with Apache.
- Nginx requires a `location` block to deny execution.
- File permissions must allow the web server to write but not execute.

#### Annotated Code Examples and Expected Outputs

##### Example 1: Generating a Safe Filename

**File: `safe_name.php`**
```php
<?php
$allowed = [
    'image/jpeg' => 'jpg',
    'image/png'  => 'png',
];

if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_FILES['upload'])) {
    $file = $_FILES['upload'];

    if ($file['error'] !== UPLOAD_ERR_OK) {
        echo 'Upload error: ' . (int)$file['error'];
        exit;
    }

    $finfo = new finfo(FILEINFO_MIME_TYPE);
    $mimeType = $finfo->file($file['tmp_name']);

    if (!isset($allowed[$mimeType])) {
        echo 'Unsupported type.';
        exit;
    }

    $extension = $allowed[$mimeType];
    $safeName = bin2hex(random_bytes(16)) . '.' . $extension;

    echo 'Original name (untrusted): ' . htmlspecialchars($file['name'], ENT_QUOTES, 'UTF-8') . '<br>';
    echo 'Safe stored name: ' . htmlspecialchars($safeName, ENT_QUOTES, 'UTF-8');
}
?>
<form method="post" action="safe_name.php" enctype="multipart/form-data">
  <input type="file" name="upload">
  <button type="submit">Upload</button>
</form>
```

**Expected Output**
```
Original name (untrusted): ../../shell.php.jpg
Safe stored name: a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6.jpg
```

**Why This Output Occurs**

- The client-supplied name is displayed (escaped) for reference.
- The stored name is generated from random bytes and the safe extension.
- The attacker cannot control or predict the stored name.

##### Example 2: Storing Outside the Web Root

**File: `store_outside.php`**
```php
<?php
$storageDir = '/var/www/storage/uploads'; // outside document root

if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_FILES['upload'])) {
    $file = $_FILES['upload'];

    if ($file['error'] !== UPLOAD_ERR_OK) {
        echo 'Upload error: ' . (int)$file['error'];
        exit;
    }

    $finfo = new finfo(FILEINFO_MIME_TYPE);
    $mimeType = $finfo->file($file['tmp_name']);

    $allowed = ['image/jpeg' => 'jpg', 'image/png' => 'png'];
    if (!isset($allowed[$mimeType])) {
        echo 'Unsupported type.';
        exit;
    }

    $extension = $allowed[$mimeType];
    $safeName = bin2hex(random_bytes(16)) . '.' . $extension;

    if (!is_dir($storageDir)) {
        mkdir($storageDir, 0755, true);
    }

    $destination = $storageDir . '/' . $safeName;

    if (move_uploaded_file($file['tmp_name'], $destination)) {
        echo 'Stored outside web root as: ' . htmlspecialchars($safeName, ENT_QUOTES, 'UTF-8');
    } else {
        echo 'Failed to move file.';
    }
}
?>
<form method="post" action="store_outside.php" enctype="multipart/form-data">
  <input type="file" name="upload">
  <button type="submit">Upload</button>
</form>
```

**Expected Output**
```
Stored outside web root as: a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6.jpg
```

**Why This Output Occurs**

- The storage directory is outside the web root.
- Even if the file contained PHP code, it cannot be accessed via URL.
- The filename is random and safe.

#### Real-World Cases

- **Social media:** Store user uploads in a non-executable directory.
- **Document management:** Store PDFs outside the web root and serve via a controller.
- **E-commerce:** Store product images in a CDN-synced directory with execution disabled.
- **CMS:** Store media library files with random names and a database record.
- **Healthcare:** Store patient documents in an encrypted, non-executable directory.

---

## Security Considerations

### Upload Attack Vectors

| Attack | Description | Mitigation |
|---|---|---|
| Webshell upload | Upload a `.php` file and execute it | Allowlist extensions, `finfo_file()`, store outside web root, disable execution |
| Path traversal | Filename contains `../` to escape directory | Use `basename()`, generate random names, never use client filename |
| Double extension | `shell.php.jpg` bypasses extension checks | Use `finfo_file()`, derive extension from MIME type |
| Null byte injection | Filename contains `%00` to truncate | Validate filenames, use `random_bytes()` |
| MIME spoofing | Client claims `image/jpeg` for a PHP file | Use `finfo_file()`, ignore `$_FILES['type']` |
| Size-based DoS | Upload huge files to fill disk | Set `upload_max_filesize`, `post_max_size`, validate size in code |
| Polyglot files | File is both a valid image and PHP code | Use `finfo_file()`, re-encode images, store outside web root |
| Overwrite attack | Filename matches existing file | Use `random_bytes()` for unique names |
| SVG/HTML XSS | Upload SVG or HTML with embedded JavaScript | Allowlist MIME types, sanitize content, serve with `Content-Disposition: attachment` |

### Defense-in-Depth Checklist

1. **Check `error` first.** Never process a file with an error code other than `UPLOAD_ERR_OK`.
2. **Validate size.** Compare `$_FILES['size']` against a maximum.
3. **Detect MIME with `finfo_file()`.** Never trust `$_FILES['type']`.
4. **Use an allowlist.** Reject any type not explicitly permitted.
5. **Generate a random filename.** Use `random_bytes()` and a safe extension.
6. **Store outside the web root** or in a directory with execution disabled.
7. **Use `move_uploaded_file()`.** Never use `copy()` or `rename()`.
8. **Set `post_max_size` > `upload_max_filesize`.** Configure `max_file_uploads`.
9. **Serve files through a controller** with access checks if stored outside the web root.
10. **Re-encode images** to strip embedded code (for image uploads).
11. **Log upload activity** for auditing and anomaly detection.
12. **Set `Content-Disposition: attachment`** when serving user-uploaded files.

### Deprecated, Unsafe, or Version-Specific Features

| Feature | Status | Replacement |
|---|---|---|
| `$_FILES['type']` | Untrusted | `finfo_file()` |
| `copy()` on `tmp_name` | Unsafe | `move_uploaded_file()` |
| `rename()` on `tmp_name` | Unsafe | `move_uploaded_file()` |
| Client filename on disk | Unsafe | `random_bytes()` |
| Blacklist extension filtering | Bypassable | Allowlist |
| `is_uploaded_file()` alone | Insufficient | `move_uploaded_file()` |
| `uniqid()` for filenames | Predictable | `random_bytes()` |
| `$_FILES['full_path']` | PHP 8.1.0+; untrusted | `basename()` |
| `MAX_FILE_SIZE` | Client-side only | Server-side size check |

### References Links

- PHP: Handling File Uploads — https://www.php.net/manual/en/features.file-upload.php
- PHP: `$_FILES` — https://www.php.net/manual/en/reserved.variables.files.php
- PHP: `move_uploaded_file()` — https://www.php.net/manual/en/function.move-uploaded-file.php
- PHP: `is_uploaded_file()` — https://www.php.net/manual/en/function.is-uploaded-file.php
- PHP: `finfo_file()` — https://www.php.net/manual/en/function.finfo-file.php
- PHP: `finfo_open()` — https://www.php.net/manual/en/function.finfo-open.php
- PHP: `random_bytes()` — https://www.php.net/manual/en/function.random-bytes.php
- PHP: `basename()` — https://www.php.net/manual/en/function.basename.php
- PHP: `pathinfo()` — https://www.php.net/manual/en/function.pathinfo.php
- PHP: File Upload Configuration — https://www.php.net/manual/en/ini.core.php#ini.upload-max-filesize
- PHP: `upload_max_filesize` — https://www.php.net/manual/en/ini.core.php#ini.upload-max-filesize
- PHP: `post_max_size` — https://www.php.net/manual/en/ini.core.php#ini.post-max-size
- PHP: `max_file_uploads` — https://www.php.net/manual/en/ini.core.php#ini.max-file-uploads
- PHP: `file_uploads` — https://www.php.net/manual/en/ini.core.php#ini.file-uploads
- OWASP: File Upload Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html
- OWASP: PHP Security Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/PHP_Configuration_Cheat_Sheet.html
- OWASP Top 10: A05 Security Misconfiguration — https://owasp.org/Top10/A05_2021-Security_Misconfiguration/
- RFC 7578: Returning Values from Forms: multipart/form-data — https://datatracker.ietf.org/doc/html/rfc7578
- CWE-434: Unrestricted Upload of File with Dangerous Type — https://cwe.mitre.org/data/definitions/434.html
- CWE-22: Improper Limitation of a Pathname to a Restricted Directory — https://cwe.mitre.org/data/definitions/22.html
- PSR-7: HTTP Message Interfaces (UploadedFileInterface) — https://www.php-fig.org/psr/psr-7/