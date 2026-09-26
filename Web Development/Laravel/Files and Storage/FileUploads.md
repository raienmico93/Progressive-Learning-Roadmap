# Laravel File Uploads — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Laravel File Uploads is the framework's subsystem for receiving, validating, processing, and persisting files submitted by clients via HTTP multipart/form-data requests, providing a unified API that integrates with Laravel's Filesystem abstraction layer.

**Technical Definition:** Laravel File Uploads is implemented through the `Illuminate\Http\UploadedFile` class (which extends PHP's native `SplFileInfo` and wraps the `Symfony\Component\HttpFoundation\File\UploadedFile`), the `Illuminate\Http\Request` object's file access methods, the `Illuminate\Validation\Rules\File` fluent validation rule builder, and the `Illuminate\Filesystem\FilesystemAdapter` storage layer. The subsystem handles the complete lifecycle: HTTP request parsing, temporary file management by PHP's upload mechanism, validation via Laravel's Validator, and final persistence via `store()`, `storeAs()`, or direct Filesystem operations.

**Beginner-Friendly Explanation:** When a user selects a file on a web form and clicks "Upload," that file travels from their computer to your server. Laravel gives you tools to check that the file is safe (right type, right size), give it a safe name, save it somewhere (your server or the cloud), and later show it to users or let them download it. It handles all the messy parts — like preventing malicious filenames, cleaning up temporary files, and making sure large uploads don't crash your server.

### Key Characteristics

- **Security-first design:** Built-in validation rules for file size, MIME type, image dimensions, and file extensions.
- **Fluent validation API:** The `File` rule object provides a chainable, expressive way to define file constraints.
- **Unified storage integration:** Uploaded files are seamlessly stored via the same `Storage` facade used for all filesystem operations.
- **Automatic temporary file management:** PHP's upload mechanism places files in a temporary directory; Laravel's `UploadedFile` class abstracts access to this location.
- **Multiple storage strategies:** Files can be processed synchronously (`store()`, `storeAs()`) or uploaded directly to cloud storage via presigned URLs.
- **Testing support:** `UploadedFile::fake()` allows generating fake files for testing without real uploads.

### Prerequisites

- PHP 8.1 or higher (Laravel 10+; Laravel 11 requires PHP 8.2+).
- The `fileinfo` PHP extension enabled (required for MIME type detection via `finfo_file()`).
- The `file_uploads` PHP directive set to `On` (default).
- Appropriate `upload_max_filesize` and `post_max_size` PHP INI settings for the expected file sizes.
- A Laravel application with the Storage subsystem configured (see Laravel File Storage cheat sheet).

### Related Programming Areas

- **HTTP Request Handling** — `Illuminate\Http\Request` provides the `file()` method for accessing uploaded files.
- **Validation** — Laravel's Validator and the `File` rule object enforce upload constraints.
- **Filesystem Abstraction** — Uploaded files are persisted through the `Storage` facade and its disk system.
- **MIME Type Detection** — PHP's `finfo` extension and `mime_content_type()` function read file magic numbers to determine the true file type.
- **Cloud Storage** — S3 presigned URLs and multipart uploads enable direct-to-cloud file transfers.
- **Image Processing** — Packages like Intervention Image are commonly used to re-encode uploads, stripping metadata and embedded code.

### Core Concepts / Features

1. Validation (Sizing thresholds, `UploadedFile` structural evaluation, and malware/executable vector prevention)
2. File Naming (Using `hashName()`, original extensions, and preventing directory traversal attacks)
3. MIME Validation (Validating server-side magic numbers vs. client-supplied MIME types)
4. Storage (Direct processing via `store()`, `storeAs()`, and targeted disk allocation)
5. Retrieval (Asset generation, stream responses, and symlink abstraction rules)
6. Temporary Files (Handling multi-part chunks and cleaning up abandoned server uploads)
7. Large File Optimizations (Streaming direct uploads to cloud storage via signed post requests to bypass server bottlenecks)

---

## 1. Validation

### Definitions

**Core Definition:** Validation is the process of verifying that an uploaded file meets predefined constraints — including size thresholds, MIME type restrictions, image dimensions, and structural integrity — before the file is accepted and processed.

**Technical Definition:** Laravel's file validation is implemented through the Validator class's file-specific rules: `file`, `image`, `mimes`, `mimetypes`, `size`, `min`, `max`, and `dimensions`. These rules operate on the `UploadedFile` instance, which extends `Symfony\Component\HttpFoundation\File\UploadedFile` and provides methods such as `getSize()`, `getMimeType()`, `getClientOriginalExtension()`, and `isValid()` for structural evaluation. The `File` rule object (`Illuminate\Validation\Rules\File`) provides a fluent builder API for composing these constraints.

**Beginner-Friendly Explanation:** Validation is like a security checkpoint for uploaded files. Before Laravel lets a file through, it checks: Is it the right kind of file (not a virus disguised as an image)? Is it too big? Is it actually a valid image file, or just something renamed to end in `.jpg`? If the file fails any of these checks, Laravel rejects it and sends an error back to the user.

### Purposes

- To prevent files that exceed server or application size limits from being processed.
- To restrict uploads to a whitelist of permitted file types (images, PDFs, etc.).
- To verify that uploaded files are structurally valid and not corrupted or maliciously crafted.
- To block executable file vectors (`.php`, `.phtml`, `.phar`) that could lead to remote code execution.
- To enforce image dimension constraints to prevent decompression bombs and layout-breaking uploads.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// Using string-based validation rules
$request->validate([
    'document' => 'required|file|mimes:pdf,docx|max:10240',
    'avatar'   => 'required|image|mimes:jpeg,png|max:2048',
]);

// Using the fluent File rule builder (Laravel 9+)
use Illuminate\Validation\Rules\File;

$request->validate([
    'attachment' => [
        'required',
        File::types(['mp3', 'wav'])
            ->min(1024)          // Minimum 1 MB
            ->max(12 * 1024),    // Maximum 12 MB
    ],
    'photo' => [
        'required',
        File::image()
            ->min(1024)
            ->max(12 * 1024)
            ->dimensions(
                Rule::dimensions()->maxWidth(1000)->maxHeight(500)
            ),
    ],
]);
```

**Component Breakdown:**

- `'required'` — The field must be present in the request.
- `'file'` — The field must be an uploaded file (not a string or array).
- `'mimes:pdf,docx'` — The file's MIME type must correspond to one of the listed extensions. Laravel reads the file's contents (via `finfo`) to determine the actual MIME type, not the client-supplied value.
- `'max:10240'` — The file size must not exceed 10,240 kilobytes (10 MB).
- `'min:1024'` — The file size must be at least 1,024 kilobytes (1 MB).
- `'image'` — The file must be an image (jpeg, png, bmp, gif, svg, or webp).
- `File::types(['mp3', 'wav'])` — Fluent API equivalent of `mimes:mp3,wav`.
- `File::image()->dimensions(...)` — Validates that the uploaded image meets width/height constraints.

**Syntax Rules:**

- The `mimes` rule validates by **reading the file's contents** and guessing the MIME type — it does not trust the client-supplied `Content-Type` header. This is critical for security.
- The `mimetypes` rule validates against the actual MIME type string (e.g., `image/jpeg`), while `mimes` validates against file extensions (e.g., `jpeg`). Both use server-side detection.
- The `size` rule checks the file size in kilobytes. For files, `min` and `max` also check size in kilobytes.
- The `dimensions` rule can only be used on image uploads and requires the GD extension.
- When validating arrays of files, use `'files.*' => 'mimes:pdf'` syntax.

**Constraints and Limitations:**

- **`mimes` does not guarantee the file is safe.** A file can have a valid MIME type but still contain malicious payloads (e.g., a valid JPEG with embedded PHP code in EXIF data). Additional processing (re-encoding) is recommended for images.
- **The `finfo` extension must be enabled** for `mimes` and `mimetypes` rules to work. Without it, Laravel cannot read magic numbers.
- **PHP's `upload_max_filesize` and `post_max_size` directives take precedence.** If a file exceeds these server limits, Laravel's validation never runs because PHP rejects the upload before the request reaches the application.
- **The `image` rule uses `getimagesize()`, which can be fooled** by polyglot files (files valid as both images and scripts). Re-encoding is the only reliable mitigation.
- **Laravel 8 and earlier do not support `mimetypes`.** Use `mimes` instead on older versions.

### Annotated Code Examples

**Example 1: Comprehensive File Validation with the Fluent Rule Builder**

```php
<?php
// File: app/Http/Controllers/UploadController.php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Validation\Rules\File;
use Illuminate\Validation\Rule;

class UploadController extends Controller
{
    public function store(Request $request)
    {
        // Step 1: Define comprehensive validation rules
        $validated = $request->validate([
            // Document upload: PDF or Word, 1KB to 10MB
            'document' => [
                'required',
                File::types(['pdf', 'doc', 'docx'])
                    ->min(1)              // At least 1 KB
                    ->max(10 * 1024),     // At most 10 MB
            ],

            // Profile photo: JPEG or PNG image, max 2MB, max 500x500 pixels
            'photo' => [
                'required',
                File::image()
                    ->max(2 * 1024)       // 2 MB
                    ->dimensions(
                        Rule::dimensions()
                            ->maxWidth(500)
                            ->maxHeight(500)
                    ),
            ],

            // Video upload: MP4 or MOV, max 100MB
            'video' => [
                'nullable',
                File::types(['mp4', 'mov'])
                    ->max(100 * 1024),
            ],
        ]);

        // Step 2: At this point, all files have passed validation.
        // The $validated array contains the validated file instances.
        // Proceed with storage (see Section 4).

        return response()->json([
            'message'  => 'All files validated successfully.',
            'document' => $validated['document']->getClientOriginalName(),
        ]);
    }
}
```

**Step-by-Step Setup:**

1. Create the controller and register the route: `POST /upload` → `UploadController@store`.
2. Ensure the `fileinfo` PHP extension is enabled (`php -m | grep fileinfo`).
3. Set `upload_max_filesize = 100M` and `post_max_size = 100M` in `php.ini` to accommodate the largest expected upload.
4. For image dimension validation, ensure the GD extension is installed (`php -m | grep gd`).

**Expected Output:**

- If all files pass: `{"message": "All files validated successfully.", "document": "contract.pdf"}`.
- If the document exceeds 10 MB: a 422 JSON response with `{"errors": {"document": ["The document must not be greater than 10240 kilobytes."]}}`.
- If the photo is 600×600 pixels: a 422 JSON response with a dimension error.
- If a `.php` file is uploaded as the document: a 422 JSON response with a MIME type error, because the file's content is detected as `text/x-php`, not `application/pdf`.

**Why This Output Occurs:** The `File::types(['pdf'])` rule reads the uploaded file's magic bytes via `finfo_file()`. A PHP script renamed to `contract.pdf` has the magic bytes of a text file, not a PDF (`%PDF` = `25504446`). Laravel's MIME detection correctly identifies it as `text/x-php` and rejects it. The `dimensions` rule reads the image header via `getimagesize()` and compares the width and height against the configured maximums.

---

**Example 2: Structural Evaluation with UploadedFile Methods**

```php
<?php
// File: app/Http/Controllers/InspectionController.php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Http\UploadedFile;

class InspectionController extends Controller
{
    public function inspect(Request $request)
    {
        $request->validate([
            'file' => 'required|file|max:5120', // 5MB max
        ]);

        /** @var UploadedFile $file */
        $file = $request->file('file');

        // Structural evaluation — inspect the UploadedFile instance
        $info = [
            'original_name'      => $file->getClientOriginalName(),
            'original_extension' => $file->getClientOriginalExtension(),
            'client_mime_type'   => $file->getClientMimeType(),
            'server_mime_type'   => $file->getMimeType(),      // Uses finfo
            'guessed_extension'  => $file->extension(),        // From MIME type
            'size_bytes'         => $file->getSize(),
            'size_kb'            => round($file->getSize() / 1024, 2),
            'is_valid'           => $file->isValid(),
            'real_path'          => $file->getRealPath(),
            'error_code'         => $file->getError(),
            'error_message'      => $file->getErrorMessage(),
        ];

        return response()->json($info);
    }
}
```

**Expected Output:**

```json
{
    "original_name": "vacation.jpg",
    "original_extension": "jpg",
    "client_mime_type": "image/jpeg",
    "server_mime_type": "image/jpeg",
    "guessed_extension": "jpg",
    "size_bytes": 1048576,
    "size_kb": 1024.0,
    "is_valid": true,
    "real_path": "/tmp/phpXaBcDe",
    "error_code": 0,
    "error_message": ""
}
```

**Why This Output Occurs:** `getClientOriginalName()` and `getClientOriginalExtension()` return the values supplied by the client's browser — these are **untrusted** and should never be used for security decisions. `getMimeType()` uses PHP's `finfo` extension to read the file's actual content and return the server-determined MIME type. `extension()` guesses the extension from the server-determined MIME type. `isValid()` checks PHP's upload error code — `0` (`UPLOAD_ERR_OK`) means the upload succeeded; non-zero values indicate errors like file too large or partial upload.

### Real-World Cases

- **User avatar uploads:** Validating that the uploaded file is a real image (not a PHP script disguised as `.jpg`), within a 2 MB limit, and no larger than 1000×1000 pixels to prevent layout issues and decompression bombs.
- **Document management systems:** Restricting uploads to PDF and Office formats, with a 50 MB limit, and verifying that the file's magic number matches its extension.
- **Video sharing platforms:** Validating that uploads are MP4 or WebM files, within a 500 MB limit, and rejecting files with corrupted headers that would fail during transcoding.
- **E-commerce product image uploads:** Enforcing JPEG or PNG format, maximum 5 MB, and minimum 200×200 pixels to ensure product images meet display requirements.

---

## 2. File Naming

### Definitions

**Core Definition:** File naming is the process of generating a safe, unique filename for an uploaded file that prevents collisions, directory traversal attacks, and the execution of malicious files.

**Technical Definition:** Laravel's `UploadedFile` class provides two primary methods for safe filename generation: `hashName()`, which generates an MD5 hash of the file's contents and appends the server-determined extension, and `extension()`, which returns the extension guessed from the file's MIME type. The `store()` method calls `hashName()` internally, while `storeAs()` allows the developer to supply a custom filename. Directory traversal prevention is achieved by discarding the client-supplied filename entirely and generating a new one, or by sanitising any user-supplied path components using `basename()` and `realpath()`.

**Beginner-Friendly Explanation:** When someone uploads a file called `../../etc/passwd.jpg` or `virus.php.jpg`, you don't want to save it under that name. Laravel's solution is simple: ignore the name the user gave the file, and generate a random, safe name instead. The `hashName()` method creates a long random string (based on the file's content) and adds the correct extension. This means every file gets a unique name, and no one can trick your server into saving a file where it shouldn't.

### Purposes

- To prevent filename collisions when multiple users upload files with the same original name.
- To eliminate directory traversal attack vectors such as `../../` in filenames.
- To prevent attackers from using executable extensions (`.php`, `.phtml`) to achieve remote code execution.
- To ensure that the stored filename's extension matches the file's actual content type (server-determined).
- To provide a predictable, secure naming convention across all uploaded files.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// Automatic safe naming via store() — uses hashName() internally
$path = $request->file('avatar')->store('avatars', 'public');
// Result: avatars/8b2f3c1d5e6a7b8c9d0e1f2a3b4c5d6e.jpg

// Explicit safe naming via hashName()
$safeName = $request->file('avatar')->hashName();
// Result: 8b2f3c1d5e6a7b8c9d0e1f2a3b4c5d6e.jpg

// Custom naming via storeAs() — developer supplies the filename
$path = $request->file('avatar')->storeAs(
    'avatars',                    // Directory
    'user-' . $user->id . '.jpg', // Custom filename
    'public'                      // Disk
);

// Using Storage::putFileAs() — equivalent to storeAs()
Storage::putFileAs('avatars', $request->file('avatar'), 'user-' . $user->id . '.jpg');

// Manual directory traversal prevention
$unsafeName = $request->input('filename');
$safeName = basename($unsafeName); // Strips ../ and directory components
```

**Component Breakdown:**

- `hashName()` — Returns an MD5 hash of the file's contents concatenated with the server-guessed extension. The hash is deterministic for the same file content, but the extension is appended with a dot.
- `extension()` — Returns the file extension guessed from the server-detected MIME type (e.g., `jpg`, `png`, `pdf`). This may differ from the client-supplied extension.
- `storeAs($path, $name, $disk)` — Stores the file at the given path with the given name. The developer is responsible for ensuring the name is safe.
- `basename($path)` — PHP function that returns the trailing name component of a path, stripping any directory traversal sequences.
- `realpath($path)` — PHP function that resolves a path to its absolute canonical form, resolving `../` and symbolic links.

**Syntax Rules:**

- `hashName()` generates a 32-character MD5 hash followed by a dot and the server-guessed extension. The MD5 hash is computed from the file's **contents**, not its name.
- If `storeAs()` is used with a user-supplied filename, **the filename must be sanitised** using `basename()` or a similar function. Never pass user input directly to `storeAs()`.
- The `extension()` method is preferred over `getClientOriginalExtension()` for determining the extension, because it reads the file's content to guess the type, while the latter trusts the client's browser.
- When using `store()` or `storeAs()`, the returned path is the relative path within the disk's root, including the generated filename.

**Constraints and Limitations:**

- **`hashName()` is deterministic:** Two files with identical content will produce the same hash. This means uploading the same file twice will result in the second upload overwriting the first (unless the path includes a unique identifier). To guarantee uniqueness, combine `hashName()` with a unique prefix or use `Str::uuid()`.
- **`getClientOriginalName()` and `getClientOriginalExtension()` are unsafe** and should never be used for filesystem operations or validation decisions. They are useful only for display purposes (e.g., showing the original filename to the user).
- **Directory traversal prevention requires more than just naming.** Even with a safe filename, the **path** passed to `store()` or `storeAs()` must not contain user-supplied directory components. Always use hardcoded or validated directory paths.
- **On case-insensitive filesystems** (Windows, macOS), `hashName()` results are case-insensitive, which can lead to collisions if two different hashes differ only in case.

### Annotated Code Examples

**Example 1: Safe Naming with hashName() and Content-Based Uniqueness**

```php
<?php
// File: app/Http/Controllers/DocumentController.php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Str;
use Illuminate\Support\Facades\Storage;

class DocumentController extends Controller
{
    public function store(Request $request)
    {
        $request->validate([
            'document' => 'required|file|mimes:pdf,docx|max:20480',
        ]);

        $file = $request->file('document');

        // Step 1: Generate a unique, safe filename
        // hashName() creates: <md5-of-contents>.<server-guessed-extension>
        // Prepend a UUID to guarantee uniqueness even for identical files
        $uniquePrefix = Str::uuid()->toString(); // e.g., "550e8400-e29b-41d4-a716-446655440000"
        $safeName = $uniquePrefix . '_' . $file->hashName();

        // Step 2: Store the file with the safe name
        // The directory is hardcoded — no user input involved
        $path = $file->storeAs(
            'documents/' . date('Y/m'),  // Organised by year/month
            $safeName,
            'local'
        );

        // Step 3: Verify the stored file's actual extension matches expectations
        $storedExtension = pathinfo($path, PATHINFO_EXTENSION);
        if (!in_array($storedExtension, ['pdf', 'docx'])) {
            // This should never happen after validation, but defence in depth
            Storage::disk('local')->delete($path);
            abort(422, 'Invalid file type detected during storage.');
        }

        return response()->json([
            'message'       => 'Document stored securely.',
            'original_name' => $file->getClientOriginalName(), // For display only
            'stored_path'   => $path,
        ]);
    }
}
```

**Step-by-Step Setup:**

1. Ensure the `local` disk is configured with root `storage/app/private`.
2. Ensure the `documents` subdirectory is writable by the web server.
3. The `Str::uuid()` method requires the `ramsey/uuid` package (included with Laravel by default).

**Expected Output:**

```json
{
    "message": "Document stored securely.",
    "original_name": "quarterly-report.pdf",
    "stored_path": "documents/2025/06/550e8400-e29b-41d4-a716-446655440000_8b2f3c1d5e6a7b8c9d0e1f2a3b4c5d6e.pdf"
}
```

**Why This Output Occurs:** The `hashName()` method computes an MD5 hash of the file's contents and appends the server-guessed extension (`.pdf`). The UUID prefix guarantees uniqueness even if two users upload identical files. The directory path (`documents/2025/06`) is hardcoded with a date component — no user input is used to construct the path, eliminating directory traversal risk. The final `pathinfo()` check provides defence in depth: even if an attacker somehow bypassed validation, the extension check would catch an unexpected file type.

---

**Example 2: Directory Traversal Prevention with Path Sanitisation**

```php
<?php
// File: app/Http/Controllers/DownloadController.php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Storage;

class DownloadController extends Controller
{
    public function download(Request $request)
    {
        $request->validate([
            'file' => 'required|string|max:255',
        ]);

        // Step 1: Sanitise the user-supplied filename
        // basename() strips any directory components: "../../.env" → ".env"
        $requestedFile = basename($request->input('file'));

        // Step 2: Define the allowed directory (whitelist)
        $allowedDirectory = 'downloads';

        // Step 3: Construct the full path within the allowed directory
        $fullPath = $allowedDirectory . '/' . $requestedFile;

        // Step 4: Use realpath() to resolve and verify the path
        $realPath = realpath(storage_path('app/private/' . $fullPath));

        // Step 5: Verify the resolved path is still within the allowed directory
        $baseDir = realpath(storage_path('app/private/' . $allowedDirectory));

        if ($realPath === false || strpos($realPath, $baseDir) !== 0) {
            abort(403, 'Access denied.');
        }

        // Step 6: Verify the file exists
        if (!Storage::disk('local')->exists($fullPath)) {
            abort(404, 'File not found.');
        }

        // Step 7: Return the file as a download
        return Storage::disk('local')->download($fullPath);
    }
}
```

**Expected Output:**

- Requesting `?file=report.pdf` downloads the file from `storage/app/private/downloads/report.pdf`.
- Requesting `?file=../../.env` returns a **403 Access denied** response because `basename('../../.env')` returns `.env`, and then `realpath()` resolves the path to `storage/app/private/downloads/.env`, which does not exist — and even if it did, the `strpos()` check would confirm it is within the allowed directory.
- Requesting `?file=../../../etc/passwd` returns **403** because `basename()` reduces it to `passwd`, and the path resolution and prefix check prevent access outside the `downloads` directory.

**Why This Output Occurs:** The `basename()` function strips all directory traversal components from the user input. The `realpath()` function resolves the resulting path to its absolute canonical form, eliminating any `../` sequences that might have survived. The `strpos($realPath, $baseDir) !== 0` check ensures the resolved path begins with the allowed directory path. This is a defence-in-depth approach: even if `basename()` were bypassed, `realpath()` and the prefix check would catch the traversal attempt.

### Real-World Cases

- **User avatar systems:** Every uploaded avatar receives a UUID-based filename, preventing user A from overwriting user B's avatar by uploading a file with the same original name.
- **Document management platforms:** Files are stored with content-hash-based names, allowing deduplication (identical files share the same hash) while UUID prefixes guarantee per-upload uniqueness.
- **File download endpoints:** User-supplied filenames are sanitised with `basename()` and validated with `realpath()` before any filesystem operation, preventing attackers from reading arbitrary files on the server.
- **Multi-tenant SaaS applications:** Each tenant's files are stored in a tenant-specific directory, and the directory path is derived from the authenticated user's tenant ID — never from user input — eliminating traversal risk.

---

## 3. MIME Validation

### Definitions

**Core Definition:** MIME validation is the process of determining a file's actual type by inspecting its binary content (magic numbers) rather than trusting the client-supplied `Content-Type` header or file extension.

**Technical Definition:** Laravel's `mimes` and `mimetypes` validation rules use PHP's `finfo_file()` function (from the `fileinfo` extension) to read the first few hundred bytes of an uploaded file and match them against a magic number database. The magic number is a unique byte sequence at the start of a file that identifies its format — for example, PDF files begin with `%PDF` (`25 50 44 46` in hexadecimal), PNG files begin with `89 50 4E 47`, and JPEG files begin with `FF D8 FF`. The `UploadedFile::getMimeType()` method exposes this server-determined MIME type, while `getClientMimeType()` returns the untrusted client-supplied value.

**Beginner-Friendly Explanation:** When a browser uploads a file, it tells the server "this is an image/jpeg" — but that's just a label the browser puts on the file. An attacker can easily change that label. To find out what a file *really* is, you have to look inside it. Every file format starts with a unique "signature" — a few bytes that are always the same. PDFs start with `%PDF`, PNGs start with `‰PNG`, and so on. Laravel reads these bytes and figures out the true type, completely ignoring what the browser claimed.

### Purposes

- To prevent attackers from uploading executable files disguised with image or document extensions.
- To ensure that the file's actual content matches the expected format before processing.
- To detect polyglot files that are valid in multiple formats (e.g., a file that is both a valid GIF and a valid PHP script).
- To provide a reliable basis for validation decisions that cannot be spoofed by manipulating HTTP headers.
- To comply with security standards that require content-based file type verification.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// Using the mimes rule (validates by extension, reads content for MIME)
$request->validate([
    'document' => 'required|file|mimes:pdf,docx',
]);

// Using the mimetypes rule (validates by actual MIME type string)
$request->validate([
    'video' => 'required|file|mimetypes:video/mp4,video/quicktime',
]);

// Manual MIME validation in application code
$file = $request->file('upload');
$serverMime = $file->getMimeType();       // finfo-based detection
$clientMime = $file->getClientMimeType(); // Untrusted, from browser

// Manual magic number inspection (bypassing finfo limitations)
$handle = fopen($file->getRealPath(), 'rb');
$magicBytes = fread($handle, 8);
fclose($handle);
$hex = bin2hex($magicBytes);

// Compare against known magic numbers
$magicNumbers = [
    'pdf'  => '25504446',       // %PDF
    'png'  => '89504e47',       // ‰PNG
    'jpeg' => 'ffd8ffe0',       // JPEG (JFIF)
    'gif'  => '47494638',       // GIF8
];
```

**Component Breakdown:**

- `getMimeType()` — Returns the MIME type detected by `finfo_file()`. This reads the file's magic bytes and is the authoritative source for file type.
- `getClientMimeType()` — Returns the MIME type supplied by the client's browser in the `Content-Type` header of the multipart request. This value is **completely untrusted** and can be arbitrarily set by an attacker.
- `mimes:pdf,docx` — Validates that the file's server-detected MIME type corresponds to one of the listed extensions. Laravel maps extensions to MIME types using a built-in lookup table.
- `mimetypes:application/pdf` — Validates against the actual MIME type string. More precise than `mimes` but less convenient.
- `finfo_file()` — PHP function that opens a file, reads its magic bytes, and returns the detected MIME type. Requires the `fileinfo` extension.

**Syntax Rules:**

- The `mimes` rule uses the extension list as a **convenience** — internally, it maps each extension to its MIME type and validates against the server-detected MIME type.
- The `mimetypes` rule validates against the MIME type string directly. Use this when you need to accept multiple extensions that map to the same MIME type (e.g., `image/jpeg` covers both `.jpg` and `.jpeg`).
- Both rules ignore the client-supplied MIME type entirely. The `getClientMimeType()` method returns the untrusted value and should never be used for validation.
- The `finfo` extension must be enabled for these rules to function. Check with `php -m | grep fileinfo`.

**Constraints and Limitations:**

- **`finfo_file()` is not infallible.** Some file formats share magic numbers (e.g., ZIP-based formats like `.docx`, `.xlsx`, and `.jar` all begin with `PK`). An attacker could craft a file that passes the MIME check but is still malicious.
- **Magic number validation is a necessary but not sufficient security measure.** It should be combined with extension whitelisting, size limits, and content re-encoding for images.
- **`finfo` may consume significant memory for large files** because it reads a portion of the file into memory. This is generally not an issue for uploads within reasonable size limits.
- **Some legitimate files may have unusual magic numbers** — for example, certain PDF generators produce files with minimal headers that `finfo` may misidentify.

### Annotated Code Examples

**Example 1: Server-Side vs. Client-Side MIME Comparison**

```php
<?php
// File: app/Http/Controllers/SecurityCheckController.php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class SecurityCheckController extends Controller
{
    public function analyse(Request $request)
    {
        $request->validate([
            'file' => 'required|file|max:10240',
        ]);

        $file = $request->file('file');

        // Client-supplied MIME type — UNTRUSTED
        $clientMime = $file->getClientMimeType();

        // Server-detected MIME type — from magic numbers via finfo
        $serverMime = $file->getMimeType();

        // Server-guessed extension — derived from the detected MIME
        $serverExtension = $file->extension();

        // Manual magic number inspection
        $handle = fopen($file->getRealPath(), 'rb');
        $magicBytes = bin2hex(fread($handle, 8));
        fclose($handle);

        // Known magic numbers for comparison
        $knownMagic = [
            '25504446' => 'PDF',
            '89504e47' => 'PNG',
            'ffd8ffe0' => 'JPEG (JFIF)',
            'ffd8ffe1' => 'JPEG (EXIF)',
            '47494638' => 'GIF',
            '504b0304' => 'ZIP-based (docx, xlsx, jar, etc.)',
            '3c3f7068' => 'PHP script (<?ph)',
            '3c68746d' => 'HTML file (<htm)',
        ];

        $detectedByMagic = 'Unknown';
        foreach ($knownMagic as $prefix => $format) {
            if (str_starts_with($magicBytes, $prefix)) {
                $detectedByMagic = $format;
                break;
            }
        }

        // Check for suspicious discrepancies
        $suspicious = false;
        $warnings = [];

        if ($clientMime !== $serverMime) {
            $suspicious = true;
            $warnings[] = "Client MIME ({$clientMime}) differs from server MIME ({$serverMime}).";
        }

        if (str_contains($detectedByMagic, 'PHP') || str_contains($detectedByMagic, 'HTML')) {
            $suspicious = true;
            $warnings[] = "File contains executable/script content ({$detectedByMagic}).";
        }

        return response()->json([
            'client_mime'       => $clientMime,
            'server_mime'       => $serverMime,
            'server_extension'  => $serverExtension,
            'magic_bytes_hex'   => $magicBytes,
            'detected_by_magic' => $detectedByMagic,
            'suspicious'        => $suspicious,
            'warnings'          => $warnings,
        ]);
    }
}
```

**Expected Output (for a legitimate JPEG):**

```json
{
    "client_mime": "image/jpeg",
    "server_mime": "image/jpeg",
    "server_extension": "jpg",
    "magic_bytes_hex": "ffd8ffe000104a46",
    "detected_by_magic": "JPEG (JFIF)",
    "suspicious": false,
    "warnings": []
}
```

**Expected Output (for a PHP script renamed to .jpg):**

```json
{
    "client_mime": "image/jpeg",
    "server_mime": "text/x-php",
    "server_extension": "php",
    "magic_bytes_hex": "3c3f7068700a0a",
    "detected_by_magic": "PHP script (<?ph)",
    "suspicious": true,
    "warnings": [
        "Client MIME (image/jpeg) differs from server MIME (text/x-php).",
        "File contains executable/script content (PHP script (<?ph))."
    ]
}
```

**Why This Output Occurs:** The `getClientMimeType()` method returns the value from the HTTP `Content-Type` header, which the browser set to `image/jpeg` based on the file's extension. The `getMimeType()` method uses `finfo_file()` to read the file's magic bytes — a PHP script's magic bytes are `3c3f7068` (`<?ph`), so `finfo` correctly identifies it as `text/x-php`. The manual magic number comparison confirms this detection. The discrepancy between client and server MIME types is a strong indicator of a malicious upload attempt.

---

**Example 2: Using mimetypes for Precise Video Format Validation**

```php
<?php
// File: app/Http/Controllers/VideoController.php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Validation\Rules\File;

class VideoController extends Controller
{
    public function upload(Request $request)
    {
        // Step 1: Validate using mimetypes rule for precise MIME control
        // This accepts only MP4 and QuickTime videos, regardless of extension
        $request->validate([
            'video' => [
                'required',
                'file',
                'mimetypes:video/mp4,video/quicktime',
                'max:512000', // 500 MB
            ],
        ]);

        $file = $request->file('video');

        // Step 2: Confirm the server-detected MIME type
        $serverMime = $file->getMimeType();

        // Step 3: Log the detection for auditing
        \Log::info('Video upload detected', [
            'original_name' => $file->getClientOriginalName(),
            'client_mime'   => $file->getClientMimeType(),
            'server_mime'   => $serverMime,
            'size'          => $file->getSize(),
        ]);

        return response()->json([
            'message'     => 'Video validated.',
            'mime_type'   => $serverMime,
            'extension'   => $file->extension(),
        ]);
    }
}
```

**Expected Output (for a legitimate MP4):**

```json
{
    "message": "Video validated.",
    "mime_type": "video/mp4",
    "extension": "mp4"
}
```

**Expected Output (for an AVI file renamed to .mp4):**

A 422 JSON response with the error: `{"errors": {"video": ["The video must be a file of type: video/mp4, video/quicktime."]}}`.

**Why This Output Occurs:** The `mimetypes` rule validates against the **exact MIME type string** returned by `finfo_file()`. An AVI file has the magic number `52494646` (`RIFF`) and the MIME type `video/x-msvideo`, which does not match `video/mp4` or `video/quicktime`. Laravel rejects the file even though its extension is `.mp4`. This is a critical security measure: an attacker cannot bypass validation by simply renaming a file's extension.

### Real-World Cases

- **Social media image uploads:** Validating that uploaded profile pictures are genuine JPEG or PNG files by inspecting magic numbers, preventing attackers from uploading PHP shells disguised as images.
- **Document collaboration platforms:** Ensuring that uploaded PDFs and Office documents have the correct magic numbers, rejecting files that have been crafted to exploit vulnerabilities in PDF readers or Office suites.
- **Video hosting services:** Using `mimetypes` to accept only specific video containers (MP4, WebM) and rejecting others (AVI, MKV) that may not be compatible with the platform's transcoding pipeline.
- **Healthcare and legal applications:** Verifying that uploaded medical images (DICOM) or legal documents (PDF/A) have the correct file signatures, ensuring regulatory compliance and preventing document corruption.

---

## 4. Storage

### Definitions

**Core Definition:** Storage is the process of persisting an uploaded file to a configured disk — local or cloud — using Laravel's unified Filesystem API.

**Technical Definition:** Laravel's `UploadedFile` class provides `store()` and `storeAs()` methods that delegate to the `FilesystemAdapter` for the target disk. The `store()` method accepts a directory path and a disk name, generates a safe filename via `hashName()`, and returns the relative path. The `storeAs()` method accepts a directory path, a custom filename, and a disk name. Both methods utilise Flysystem's `writeStream()` operation, which streams the file from PHP's temporary upload location to the destination, avoiding loading the entire file into memory.

**Beginner-Friendly Explanation:** Once a file has been validated, you need to save it somewhere. Laravel gives you two main ways to do this: `store()` (which saves the file with an automatically generated safe name) and `storeAs()` (which lets you choose the filename). You also tell Laravel *where* to save it — on your local server, on Amazon S3, or on any other configured disk. The best part is that the same code works regardless of where the file actually ends up.

### Purposes

- To persist uploaded files to a durable storage location with a safe, unique filename.
- To target specific disks (local, public, S3) based on the file's intended visibility and access pattern.
- To stream large files from the temporary upload directory to the destination without exhausting PHP's memory limit.
- To organise files into directory structures based on business logic (dates, user IDs, categories).
- To integrate file uploads with Laravel's existing Storage facade for subsequent retrieval and management.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// Using store() — automatic filename generation
$path = $request->file('avatar')->store('avatars', 'public');
// Result: avatars/8b2f3c...e.jpg (relative to the disk's root)

// Using store() with the default disk
$path = $request->file('document')->store('documents');
// Uses the default disk (configured via FILESYSTEM_DISK env var)

// Using storeAs() — custom filename
$path = $request->file('avatar')->storeAs(
    'avatars',                          // Directory
    'user-' . $user->id . '.jpg',       // Custom filename
    'public'                            // Disk
);

// Using Storage::putFile() — equivalent to store()
$path = Storage::putFile('avatars', $request->file('avatar'), 'public');

// Using Storage::putFileAs() — equivalent to storeAs()
$path = Storage::putFileAs(
    'avatars',
    $request->file('avatar'),
    'user-' . $user->id . '.jpg',
    'public'
);

// Storing with explicit visibility
$path = $request->file('document')->storeAs(
    'private-docs',
    $filename,
    ['disk' => 'local', 'visibility' => 'private']
);
```

**Component Breakdown:**

- `store($path, $disk)` — Stores the uploaded file at the given directory path on the given disk. Returns the relative path of the stored file. The filename is generated by `hashName()`.
- `storeAs($path, $name, $disk)` — Stores the uploaded file at the given directory path with the given filename on the given disk. The developer is responsible for filename safety.
- `$disk` — The disk name as defined in `config/filesystems.php`. If omitted, the default disk is used.
- `Storage::putFile($path, $file, $disk)` — Static facade equivalent of `store()`. Useful when working with an `UploadedFile` instance outside a controller method.
- `Storage::putFileAs($path, $file, $name, $disk)` — Static facade equivalent of `storeAs()`.

**Syntax Rules:**

- The `store()` method returns the **relative path** of the stored file (e.g., `avatars/abc123.jpg`), not the absolute filesystem path. Store this relative path in your database.
- If the directory specified in `store()` or `storeAs()` does not exist, Laravel creates it automatically (with the configured directory permissions).
- The `store()` method uses `hashName()` internally, which generates an MD5 hash of the file's contents plus the server-guessed extension.
- The `storeAs()` method does **not** sanitise the filename. If the filename is derived from user input, it must be sanitised with `basename()` first.
- When storing to a cloud disk (e.g., S3), the file is streamed to the cloud service using Flysystem's `writeStream()` operation.

**Constraints and Limitations:**

- **`store()` and `storeAs()` are synchronous.** They block the request until the file is fully written to the destination. For large files on slow cloud connections, this can lead to request timeouts.
- **PHP's `upload_max_filesize` and `post_max_size` limits apply.** Files larger than these limits never reach the `store()` method because PHP rejects them during the upload phase.
- **The `storeAs()` method does not validate the filename.** Passing `../../.env` as the filename will write outside the intended directory. Always use `basename()` or a UUID generator for custom filenames.
- **Concurrent uploads with the same `storeAs()` filename** will result in the later upload overwriting the earlier one. Use unique filenames to prevent this.

### Annotated Code Examples

**Example 1: Targeted Disk Allocation with store() and storeAs()**

```php
<?php
// File: app/Http/Controllers/MediaController.php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Storage;
use Illuminate\Support\Str;

class MediaController extends Controller
{
    public function upload(Request $request)
    {
        $request->validate([
            'avatar'    => 'required|image|mimes:jpeg,png|max:2048',
            'document'  => 'required|file|mimes:pdf|max:10240',
            'backup'    => 'nullable|file|max:51200',
        ]);

        $results = [];

        // Step 1: Avatar → public disk (web-accessible)
        // Uses store() for automatic safe naming
        $avatarPath = $request->file('avatar')->store('avatars', 'public');
        $results['avatar'] = [
            'path' => $avatarPath,
            'url'  => Storage::disk('public')->url($avatarPath),
        ];

        // Step 2: Document → local disk (private, with custom naming)
        // Uses storeAs() with a UUID-based filename
        $docFilename = Str::uuid() . '.pdf';
        $docPath = $request->file('document')->storeAs(
            'documents/' . date('Y/m'),
            $docFilename,
            'local'
        );
        $results['document'] = [
            'path' => $docPath,
            'disk' => 'local',
        ];

        // Step 3: Backup → S3 disk (cloud, if provided)
        if ($request->hasFile('backup')) {
            $backupPath = $request->file('backup')->store('backups/' . date('Y/m/d'), 's3');
            $results['backup'] = [
                'path' => $backupPath,
                'disk' => 's3',
            ];
        }

        return response()->json([
            'message' => 'Files stored successfully.',
            'files'   => $results,
        ]);
    }
}
```

**Step-by-Step Setup:**

1. Configure the `public`, `local`, and `s3` disks in `config/filesystems.php`.
2. Run `php artisan storage:link` to create the public symlink.
3. Ensure the `s3` disk has valid AWS credentials in `.env`.
4. Create the `avatars`, `documents`, and `backups` directories (or allow Laravel to create them automatically).

**Expected Output:**

```json
{
    "message": "Files stored successfully.",
    "files": {
        "avatar": {
            "path": "avatars/8b2f3c1d5e6a7b8c9d0e1f2a3b4c5d6e.jpg",
            "url": "https://example.com/storage/avatars/8b2f3c1d5e6a7b8c9d0e1f2a3b4c5d6e.jpg"
        },
        "document": {
            "path": "documents/2025/06/550e8400-e29b-41d4-a716-446655440000.pdf",
            "disk": "local"
        },
        "backup": {
            "path": "backups/2025/06/15/abc123def456.zip",
            "disk": "s3"
        }
    }
}
```

**Why This Output Occurs:** The avatar is stored on the `public` disk at `storage/app/public/avatars/` with an automatically generated `hashName()` filename. The `url()` method combines the disk's `url` configuration (`APP_URL/storage`) with the relative path. The document is stored on the `local` disk with a UUID-based filename, ensuring uniqueness and preventing collisions. The backup is streamed to S3 under a date-organised prefix. Each file targets a different disk based on its intended visibility and access pattern.

---

**Example 2: Streaming Large Files with putFile() and Memory Efficiency**

```php
<?php
// File: app/Http/Controllers/LargeUploadController.php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Storage;

class LargeUploadController extends Controller
{
    public function upload(Request $request)
    {
        $request->validate([
            'video' => 'required|file|mimetypes:video/mp4|max:512000', // 500 MB
        ]);

        $file = $request->file('video');

        // Step 1: Store the file using Storage::putFile()
        // This streams the file to the destination without loading it into memory
        $path = Storage::putFile(
            'videos/' . date('Y/m'),
            $file,
            's3'  // Upload directly to S3
        );

        // Step 2: The file is now on S3. Verify the upload.
        if (!Storage::disk('s3')->exists($path)) {
            abort(500, 'Upload failed: file not found on S3.');
        }

        // Step 3: Get the file size from S3 to confirm integrity
        $remoteSize = Storage::disk('s3')->size($path);
        $localSize  = $file->getSize();

        if ($remoteSize !== $localSize) {
            Storage::disk('s3')->delete($path);
            abort(500, 'Upload failed: size mismatch.');
        }

        return response()->json([
            'message'     => 'Video uploaded to S3.',
            'path'        => $path,
            'size_bytes'  => $remoteSize,
        ]);
    }
}
```

**Expected Output:**

```json
{
    "message": "Video uploaded to S3.",
    "path": "videos/2025/06/abc123def456.mp4",
    "size_bytes": 52428800
}
```

**Why This Output Occurs:** `Storage::putFile()` uses Flysystem's `writeStream()` operation, which opens a read stream on the temporary upload file and a write stream on the destination (S3). The file is transferred in chunks, so the PHP process never holds more than a small buffer of the file in memory. The `exists()` and `size()` checks verify that the file arrived intact on S3, and the size comparison provides a basic integrity check.

### Real-World Cases

- **Social media platforms:** Avatars and post images are stored on the `public` disk (or a CDN-backed S3 disk) with automatically generated filenames, while private messages and drafts are stored on the `local` disk.
- **Document management systems:** PDFs and Office documents are stored on the `local` disk with UUID filenames, and their paths are recorded in the database for retrieval through authenticated controller actions.
- **Video streaming services:** Large video files are stored directly on S3 with date-based directory organisation, and the resulting S3 paths are used to generate streaming URLs.
- **Backup systems:** Database dumps and application backups are streamed to S3 or another cloud disk, with retention policies managed by lifecycle rules on the cloud provider side.

---

## 5. Retrieval

### Definitions

**Core Definition:** Retrieval is the process of accessing stored files and serving them to users — either by generating publicly accessible URLs or by streaming file contents through the application with appropriate access controls.

**Technical Definition:** Laravel's FilesystemAdapter provides several retrieval mechanisms: the `url()` method generates a public URL for a file (based on the disk's `url` configuration), the `get()` method reads the file's contents into a string, the `response()` method returns an HTTP response with the file contents, the `download()` method returns a response that triggers a browser download, and the `temporaryUrl()` method (S3 only) generates a time-limited presigned URL. For files on the `public` disk, the `asset()` helper combined with the `storage` prefix generates URLs that are served directly by the web server via the symbolic link.

**Beginner-Friendly Explanation:** Once a file is saved, you need to show it to users or let them download it. If the file is on the "public" disk, you can just create a URL and put it in an `<img>` tag or a link — the web server handles the rest. But if the file is private (like a medical record or an invoice), you can't just give out a URL. Instead, you write a controller action that checks if the user is allowed to see the file, then streams it to them through Laravel.

### Purposes

- To generate publicly accessible URLs for files stored on the public disk.
- To serve private files through authenticated controller actions, preventing unauthorised access.
- To stream file contents without loading them entirely into memory.
- To provide temporary, signed URLs for private files on cloud storage without proxying through the application server.
- To abstract the underlying storage location — the same URL generation code works whether the file is local or in the cloud.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// Generating a public URL for a file on the public disk
$url = Storage::disk('public')->url('avatars/user1.jpg');
// Result: https://example.com/storage/avatars/user1.jpg

// Using the asset() helper (requires storage:link)
$url = asset('storage/avatars/user1.jpg');

// Reading file contents into a string
$content = Storage::disk('local')->get('documents/report.pdf');

// Returning a file as an HTTP response (streamed)
return Storage::disk('local')->response('documents/report.pdf');

// Returning a file as a download response
return Storage::disk('local')->download('documents/report.pdf', 'report.pdf');

// Generating a temporary presigned URL (S3 only)
$url = Storage::disk('s3')->temporaryUrl(
    'private/report.pdf',
    now()->addMinutes(30)
);

// Checking file existence before retrieval
if (Storage::disk('local')->exists('documents/report.pdf')) {
    // File exists — proceed with retrieval
}
```

**Component Breakdown:**

- `url($path)` — Returns the full public URL for the given file path, using the disk's `url` configuration. Only works for disks that have a `url` configured (or S3 disks, which derive the URL from the bucket and region).
- `asset('storage/' . $path)` — Blade/helper function that prepends `APP_URL` to the given path. Requires the `storage:link` to have been created.
- `get($path)` — Reads the entire file into a string. **Not suitable for large files** — use `response()` or `download()` for streaming.
- `response($path, $name, $headers)` — Returns an HTTP response with the file contents, using a streamed response. The file is not loaded entirely into memory.
- `download($path, $name, $headers)` — Returns an HTTP response that prompts the browser to download the file. The filename presented to the user can differ from the stored filename.
- `temporaryUrl($path, $expiration)` — Generates a presigned URL for S3 (and S3-compatible) disks. The URL is valid for the specified duration.

**Syntax Rules:**

- The `url()` method requires the disk to have a `url` key in its configuration. The `public` disk has this by default (`APP_URL/storage`).
- For local disks, `url()` simply concatenates the disk's `url` configuration with the file path. It does **not** verify that the file exists.
- The `response()` and `download()` methods use streamed responses, so they are suitable for large files.
- The `temporaryUrl()` method throws an exception if called on a disk that does not support temporary URLs (e.g., the `local` disk).
- Private files should **never** be stored on the `public` disk. Use the `local` disk and serve them through `response()` or `download()` with appropriate authorisation checks.

**Constraints and Limitations:**

- **`temporaryUrl()` is S3-only.** It is not available on local or FTP disks. Attempting to call it on an unsupported disk throws an exception.
- **The `url()` method does not check file existence.** It will generate a URL even for files that do not exist, leading to 404 errors when the URL is accessed.
- **Symbolic links may not work on all hosting environments.** Shared hosting providers sometimes disable the `symlink()` function. In such cases, files on the `public` disk must be served through a controller action using `response()`.
- **Streamed responses do not support range requests by default.** For video streaming with seeking, a custom streaming response with `Accept-Ranges` and `Content-Range` headers is required.

### Annotated Code Examples

**Example 1: Public vs. Private File Retrieval**

```php
<?php
// File: app/Http/Controllers/FileController.php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Storage;

class FileController extends Controller
{
    /**
     * Serve a PUBLIC file — stored on the 'public' disk.
     * The URL is generated with the asset() helper.
     */
    public function showPublic(string $filename)
    {
        $path = 'images/' . basename($filename);

        // Verify the file exists before generating the URL
        if (!Storage::disk('public')->exists($path)) {
            abort(404);
        }

        $url = Storage::disk('public')->url($path);

        return response()->json(['url' => $url]);
        // Client uses this URL directly in <img> tags or links
    }

    /**
     * Serve a PRIVATE file — stored on the 'local' disk.
     * Access is controlled through this controller action.
     */
    public function showPrivate(Request $request, int $documentId)
    {
        // Step 1: Authorisation check (assumes a Document model)
        $document = \App\Models\Document::findOrFail($documentId);

        if ($request->user()->cannot('view', $document)) {
            abort(403, 'You do not have permission to view this document.');
        }

        // Step 2: Verify the file exists on the private disk
        if (!Storage::disk('local')->exists($document->path)) {
            abort(404, 'File not found.');
        }

        // Step 3: Stream the file as a download
        // The original filename is presented to the user
        return Storage::disk('local')->download(
            $document->path,
            $document->original_name
        );
    }

    /**
     * Generate a temporary S3 URL for a private file.
     */
    public function temporaryUrl(Request $request, int $mediaId)
    {
        $media = \App\Models\Media::findOrFail($mediaId);

        // Authorisation check
        if ($request->user()->cannot('view', $media)) {
            abort(403);
        }

        // Generate a URL valid for 15 minutes
        $url = Storage::disk('s3')->temporaryUrl(
            $media->s3_path,
            now()->addMinutes(15)
        );

        return response()->json([
            'temporary_url' => $url,
            'expires_at'    => now()->addMinutes(15)->toIso8601String(),
        ]);
    }
}
```

**Step-by-Step Setup:**

1. Define the routes: `GET /public/{filename}` → `showPublic`; `GET /documents/{id}` → `showPrivate`; `GET /media/{id}/url` → `temporaryUrl`.
2. Run `php artisan storage:link` for public file access.
3. Create an authorisation policy for the `Document` and `Media` models.
4. Ensure the `s3` disk is configured with valid credentials.

**Expected Output (Public):**

```json
{
    "url": "https://example.com/storage/images/photo.jpg"
}
```

**Expected Output (Private Download):**

The browser receives the file as a download with the original filename. The response headers include `Content-Disposition: attachment; filename="quarterly-report.pdf"` and `Content-Type: application/pdf`.

**Expected Output (Temporary URL):**

```json
{
    "temporary_url": "https://bucket.s3.amazonaws.com/private/video.mp4?X-Amz-Signature=...&X-Amz-Expires=900",
    "expires_at": "2025-06-15T14:30:00+00:00"
}
```

**Why This Output Occurs:** The public file URL is generated by concatenating the `public` disk's `url` configuration (`APP_URL/storage`) with the relative path. The web server serves the file directly through the symbolic link. The private file is streamed through `Storage::download()`, which sets the `Content-Disposition` header to `attachment` and the `Content-Type` header based on the file's detected MIME type. The temporary URL is generated by the S3 SDK, which creates a signed query string containing the access key, expiration time, and a cryptographic signature. The signature ensures that only someone with the secret key (the Laravel application) can generate valid URLs.

---

**Example 2: Streaming File Contents with response()**

```php
<?php
// File: app/Http/Controllers/StreamController.php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Storage;

class StreamController extends Controller
{
    public function stream(Request $request, int $videoId)
    {
        $video = \App\Models\Video::findOrFail($videoId);

        if ($request->user()->cannot('view', $video)) {
            abort(403);
        }

        // Step 1: Verify file existence
        if (!Storage::disk('local')->exists($video->path)) {
            abort(404);
        }

        // Step 2: Get the file size for the Content-Length header
        $size = Storage::disk('local')->size($video->path);

        // Step 3: Return a streamed response
        // The file is read in chunks and sent to the browser
        return Storage::disk('local')->response(
            $video->path,
            null,  // Auto-detect filename
            [
                'Content-Type'   => 'video/mp4',
                'Content-Length' => $size,
                'Accept-Ranges'  => 'bytes',
                'Cache-Control'  => 'public, max-age=3600',
            ]
        );
    }
}
```

**Expected Output:**

The browser begins receiving the video stream. The response headers include:

```
Content-Type: video/mp4
Content-Length: 52428800
Accept-Ranges: bytes
Cache-Control: public, max-age=3600
```

**Why This Output Occurs:** The `response()` method uses Flysystem's `readStream()` to open the file as a stream, then wraps it in a Symfony `StreamedResponse`. The file is read in chunks (typically 8KB) and sent to the client as they are read, so PHP's memory limit is never exceeded, regardless of the file size. The custom headers inform the browser that the content is a video, its size, and that range requests are supported (allowing seeking in the video player).

### Real-World Cases

- **E-commerce product images:** Stored on the `public` disk with URLs generated via `Storage::url()` or `asset('storage/...')`, embedded directly in HTML.
- **Healthcare patient records:** Stored on the `local` disk and served through authenticated controller actions using `Storage::download()`, with audit logging of every access.
- **Media streaming platforms:** Videos stored on S3 with presigned temporary URLs generated for authenticated users, allowing direct streaming from S3 without proxying through the application server.
- **SaaS document sharing:** Files stored on the `local` disk, with access controlled by a policy that checks the user's role and the document's sharing settings before returning a `Storage::response()`.

---

## 6. Temporary Files

### Definitions

**Core Definition:** Temporary files are intermediate files created during the upload process — either by PHP's upload mechanism (which places incoming files in a temporary directory) or by chunked upload systems (which store individual chunks before assembling them into the final file).

**Technical Definition:** When PHP receives a multipart/form-data upload, it stores the file in the directory specified by the `upload_tmp_dir` PHP INI directive (or the system default, typically `/tmp` on Linux). The `UploadedFile` instance's `getRealPath()` method returns this temporary location. For chunked uploads, Laravel packages (or custom implementations) store individual chunks in a configured temporary directory, then merge them using streaming operations. Abandoned chunks must be cleaned up periodically — either by a scheduled Artisan command or a package-provided cleanup mechanism.

**Beginner-Friendly Explanation:** When a large file is uploaded in pieces (chunks), those pieces have to be stored somewhere until the whole file arrives and can be assembled. These "temporary files" take up disk space. If a user starts an upload but never finishes it, those chunks are left behind. Laravel and its ecosystem provide tools to clean these up automatically — either through scheduled commands or built-in cleanup mechanisms in chunked upload packages.

### Purposes

- To store individual chunks of large files during multi-part uploads before final assembly.
- To provide a staging area for uploads that are pending validation or finalisation.
- To enable resumable uploads — chunks can be stored persistently until the upload is resumed.
- To prevent abandoned uploads from consuming disk space indefinitely through automatic cleanup.
- To isolate temporary upload data from final storage, allowing for atomic finalisation.

### Syntax Rules and Structure

#### Complete General Syntax (Custom Chunked Upload)

```php
// config/filesystems.php — Temporary upload disk
'temp_uploads' => [
    'driver' => 'local',
    'root'   => storage_path('app/temp-uploads'),
    'throw'  => false,
],

// Chunk upload endpoint
Route::post('/upload/chunk', function (Request $request) {
    $request->validate([
        'file'      => 'required|file|max:10240', // Each chunk max 10MB
        'chunkIndex' => 'required|integer|min:0',
        'totalChunks' => 'required|integer|min:1',
        'uploadId'  => 'required|string|max:64',
    ]);

    $uploadId    = $request->input('uploadId');
    $chunkIndex  = $request->input('chunkIndex');
    $chunkDir    = "chunks/{$uploadId}";

    // Store the chunk with a predictable name
    $chunkPath = $request->file('file')->storeAs(
        $chunkDir,
        "chunk-{$chunkIndex}",
        'temp_uploads'
    );

    return response()->json([
        'chunk'   => $chunkIndex,
        'path'    => $chunkPath,
        'total'   => $request->input('totalChunks'),
    ]);
});
```

```php
// Chunk assembly (finalisation) endpoint
Route::post('/upload/finalize', function (Request $request) {
    $request->validate([
        'uploadId'    => 'required|string',
        'totalChunks' => 'required|integer',
        'filename'    => 'required|string',
    ]);

    $uploadId    = $request->input('uploadId');
    $totalChunks = $request->input('totalChunks');
    $chunkDir    = "chunks/{$uploadId}";

    // Step 1: Verify all chunks are present
    $existingChunks = Storage::disk('temp_uploads')->files($chunkDir);
    if (count($existingChunks) !== $totalChunks) {
        abort(422, "Expected {$totalChunks} chunks, found " . count($existingChunks));
    }

    // Step 2: Create the final file by streaming chunks in order
    $finalPath = 'assembled/' . $uploadId . '_' . basename($request->input('filename'));
    $finalStream = fopen(Storage::disk('temp_uploads')->path($finalPath), 'wb');

    for ($i = 0; $i < $totalChunks; $i++) {
        $chunkPath = "{$chunkDir}/chunk-{$i}";
        $chunkStream = Storage::disk('temp_uploads')->readStream($chunkPath);

        if ($chunkStream === null) {
            fclose($finalStream);
            abort(422, "Missing chunk {$i}");
        }

        stream_copy_to_stream($chunkStream, $finalStream);
        fclose($chunkStream);
    }

    fclose($finalStream);

    // Step 3: Move the assembled file to permanent storage
    $permanentPath = Storage::disk('local')->putFileAs(
        'documents',
        Storage::disk('temp_uploads')->path($finalPath),
        $request->input('filename')
    );

    // Step 4: Clean up temporary chunks
    Storage::disk('temp_uploads')->deleteDirectory($chunkDir);
    Storage::disk('temp_uploads')->delete($finalPath);

    return response()->json([
        'message' => 'File assembled and stored.',
        'path'    => $permanentPath,
    ]);
});
```

**Component Breakdown:**

- `temp_uploads` disk — A dedicated local disk for storing chunks and temporary assembled files.
- `$uploadId` — A unique identifier generated by the client (e.g., a UUID) to group chunks belonging to the same upload.
- `chunk-{$chunkIndex}` — Predictable chunk naming allows ordered assembly.
- `readStream()` / `writeStream()` — Flysystem methods for streaming file contents without loading them into memory.
- `stream_copy_to_stream()` — PHP function that copies data from one stream to another in chunks.
- `deleteDirectory()` — Removes the chunk directory after successful assembly.

**Syntax Rules:**

- Chunks should be stored on a dedicated temporary disk, separate from final storage, to simplify cleanup.
- Chunk filenames should encode the index (e.g., `chunk-0`, `chunk-1`) so that assembly order is deterministic.
- The assembly step should use streaming (`readStream` + `stream_copy_to_stream`) to avoid loading the entire file into memory.
- After successful assembly, the temporary chunks and the assembled intermediate file should be deleted.
- A scheduled cleanup command should remove chunks from abandoned uploads based on age.

**Constraints and Limitations:**

- **Chunked upload requires client-side JavaScript** to split the file and send chunks sequentially or in parallel. The Laravel backend cannot initiate chunking on its own.
- **Chunk size must be configured consistently** between the client and the server. If the server's validation `max` is smaller than the chunk size, uploads will fail.
- **Abandoned chunks consume disk space** until cleaned up. Without a scheduled cleanup command, the `temp_uploads` disk will grow indefinitely.
- **Concurrent uploads with the same `uploadId`** can corrupt the assembly. The client should generate a unique `uploadId` for each upload session.
- **Some packages handle chunked uploads automatically** (e.g., `pion/laravel-chunk-upload`, `devwizardhq/laravel-filex`), providing built-in cleanup and assembly logic.

### Annotated Code Examples

**Example 1: Scheduled Cleanup of Abandoned Chunks**

```php
<?php
// File: app/Console/Commands/CleanupTempUploads.php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use Illuminate\Support\Facades\Storage;
use Carbon\Carbon;

class CleanupTempUploads extends Command
{
    protected $signature = 'uploads:cleanup {--hours=24 : Age threshold in hours}';
    protected $description = 'Remove temporary upload chunks older than the specified age';

    public function handle(): int
    {
        $hours = (int) $this->option('hours');
        $threshold = Carbon::now()->subHours($hours)->timestamp;

        $disk = Storage::disk('temp_uploads');
        $deletedCount = 0;
        $freedBytes = 0;

        // Step 1: Iterate over all chunk directories
        foreach ($disk->directories('chunks') as $directory) {
            $files = $disk->files($directory);
            $allExpired = true;

            foreach ($files as $file) {
                // Step 2: Check each chunk's modification time
                if ($disk->lastModified($file) >= $threshold) {
                    $allExpired = false;
                    break;
                }
            }

            if ($allExpired && count($files) > 0) {
                // Step 3: Calculate the size before deletion
                $size = 0;
                foreach ($files as $file) {
                    $size += $disk->size($file);
                }

                $disk->deleteDirectory($directory);
                $deletedCount++;
                $freedBytes += $size;

                $this->info("Deleted: {$directory} (" . number_format($size / 1024 / 1024, 2) . " MB)");
            }
        }

        // Step 4: Also clean up assembled intermediate files
        foreach ($disk->files('assembled') as $file) {
            if ($disk->lastModified($file) < $threshold) {
                $freedBytes += $disk->size($file);
                $disk->delete($file);
                $this->info("Deleted assembled: {$file}");
            }
        }

        $this->info("Cleanup complete. Deleted {$deletedCount} chunk directories, freed " .
                     number_format($freedBytes / 1024 / 1024, 2) . " MB.");

        return Command::SUCCESS;
    }
}
```

```php
// File: app/Console/Kernel.php (Laravel 10 and below)
protected function schedule(Schedule $schedule): void
{
    $schedule->command('uploads:cleanup --hours=24')
             ->hourly()
             ->withoutOverlapping();
}

// File: routes/console.php (Laravel 11+)
Schedule::command('uploads:cleanup --hours=24')
    ->hourly()
    ->withoutOverlapping();
```

**Step-by-Step Setup:**

1. Create the Artisan command with `php artisan make:command CleanupTempUploads`.
2. Register the command in the scheduler.
3. Ensure the `temp_uploads` disk is configured in `config/filesystems.php`.
4. Run `php artisan schedule:list` to verify the schedule is registered.

**Expected Output:**

```
Deleted: chunks/550e8400-e29b-41d4-a716-446655440000 (15.24 MB)
Deleted assembled: assembled/550e8400-e29b-41d4-a716-446655440000_video.mp4
Cleanup complete. Deleted 1 chunk directories, freed 16.87 MB.
```

**Why This Output Occurs:** The command iterates over all directories within `chunks/`. For each directory, it checks whether all contained files have a `lastModified` timestamp older than the threshold (24 hours). If so, it calculates the total size, deletes the directory, and reports the freed space. This prevents abandoned uploads from consuming disk space indefinitely.

---

**Example 2: Using a Package for Chunked Uploads with Automatic Cleanup**

```php
// Install the package
// composer require devwizardhq/laravel-filex

// File: config/filex.php (published by the package)
return [
    'temporary_uploads' => [
        'disk'     => 'temp_uploads',
        'lifetime' => 3600, // 1 hour in seconds
        'cleanup'  => [
            'enabled'    => true,
            'schedule'   => 'hourly',
        ],
    ],
    'chunk_size' => 5 * 1024 * 1024, // 5 MB chunks
];
```

```blade
{{-- Blade template — the package provides a drop-in component --}}
<x-filex-uploader
    name="video"
    :multiple="false"
    maxSize="500"
    accept="video/mp4,video/quicktime"
    :chunked="true"
/>
```

**Expected Output:**

- The uploader component renders a drag-and-drop interface with automatic chunking for files larger than 5 MB.
- Chunks are stored in `storage/app/temp-uploads` and automatically cleaned up after 1 hour if the upload is not finalised.
- The package's scheduled command runs hourly to remove orphaned chunks.

**Why This Output Occurs:** The `laravel-filex` package provides a complete chunked upload solution with built-in temporary file management, automatic expiration, and scheduled cleanup. The `temporary_uploads.lifetime` configuration determines how long chunks are retained before being eligible for cleanup. The package's Blade component handles client-side chunking and progress tracking, while the backend manages chunk storage and assembly.

### Real-World Cases

- **Video upload platforms:** Large video files (1–10 GB) are uploaded in chunks, with each chunk stored temporarily and assembled into a final file before being moved to S3 or another permanent storage.
- **Cloud storage gateways:** Services that allow users to upload files to their cloud accounts use chunked uploads with temporary storage to handle large files and network interruptions.
- **Healthcare imaging systems:** Large DICOM image sets (hundreds of MB) are uploaded in chunks, with temporary storage ensuring that partial uploads do not corrupt the final image archive.
- **Enterprise document management:** Multi-gigabyte CAD files and engineering drawings are uploaded in chunks, with abandoned uploads automatically cleaned up to prevent storage bloat.

---

## 7. Large File Optimizations

### Definitions

**Core Definition:** Large file optimisations are techniques that bypass the application server as an intermediary in the upload process, allowing files to be transferred directly from the client to cloud storage using pre-signed URLs and multipart uploads.

**Technical Definition:** Laravel's S3 disk supports `temporaryUrl()` and `temporaryUploadUrl()` methods, which generate pre-signed URLs that grant time-limited write access to a specific S3 object key. For files larger than 5 MB, S3's Multipart Upload API allows the client to split the file into parts, each uploaded with its own pre-signed URL, and then instructed S3 to assemble the parts into the final object. The Laravel backend generates the URLs and coordinates the multipart lifecycle (initiate, sign parts, complete) without ever handling the file data itself.

**Beginner-Friendly Explanation:** Normally, when a user uploads a large file, it goes: user → your server → S3. This means your server has to receive the entire file and then send it to S3, which doubles the time and uses your server's bandwidth. The optimisation is to cut out the middleman: your server generates a special "permission slip" (pre-signed URL) that lets the user's browser upload the file directly to S3. Your server never touches the file data, so it doesn't get bogged down, and uploads are much faster.

### Purposes

- To bypass the application server as a data relay, reducing server load and bandwidth consumption.
- To enable uploads of files larger than PHP's `upload_max_filesize` and `post_max_size` limits.
- To leverage S3's Multipart Upload API for parallel chunk uploads, increasing throughput.
- To reduce latency by allowing clients to upload directly to the nearest S3 endpoint.
- To avoid request timeouts on the application server during long uploads.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// Step 1: Generate a pre-signed upload URL for small files (< 5 MB)
$url = Storage::disk('s3')->temporaryUploadUrl(
    'uploads/' . $filename,
    now()->addMinutes(15),
    ['ContentType' => 'video/mp4']
);

// Returns an array with 'url' and 'headers'
// [
//     'url' => 'https://bucket.s3.amazonaws.com/uploads/...?X-Amz-Signature=...',
//     'headers' => ['Content-Type' => 'video/mp4']
// ]

// Step 2: For large files (> 5 MB), initiate a multipart upload
$client = Storage::disk('s3')->getClient(); // AWS SDK client
$bucket = config('filesystems.disks.s3.bucket');
$key = 'uploads/' . $filename;

$multipart = $client->createMultipartUpload([
    'Bucket'      => $bucket,
    'Key'         => $key,
    'ContentType' => 'video/mp4',
]);

$uploadId = $multipart['UploadId'];

// Step 3: Generate pre-signed URLs for each part
$partUrls = [];
$partSize = 10 * 1024 * 1024; // 10 MB per part
$fileSize = $request->input('fileSize');
$totalParts = ceil($fileSize / $partSize);

for ($i = 1; $i <= $totalParts; $i++) {
    $command = $client->getCommand('UploadPart', [
        'Bucket'     => $bucket,
        'Key'        => $key,
        'UploadId'   => $uploadId,
        'PartNumber' => $i,
    ]);

    $presignedRequest = $client->createPresignedRequest($command, '+30 minutes');
    $partUrls[] = (string) $presignedRequest->getUri();
}

// Step 4: Return the upload ID and part URLs to the client
return response()->json([
    'uploadId' => $uploadId,
    'key'      => $key,
    'partUrls' => $partUrls,
    'partSize' => $partSize,
]);

// Step 5: Client uploads each part using the pre-signed URLs
// (Client-side JavaScript — not shown here)

// Step 6: Complete the multipart upload after all parts are uploaded
$client->completeMultipartUpload([
    'Bucket'          => $bucket,
    'Key'             => $key,
    'UploadId'        => $uploadId,
    'MultipartUpload' => [
        'Parts' => [
            ['PartNumber' => 1, 'ETag' => $etag1],
            ['PartNumber' => 2, 'ETag' => $etag2],
            // ...
        ],
    ],
]);
```

**Component Breakdown:**

- `temporaryUploadUrl($path, $expiration)` — Generates a pre-signed URL that permits a PUT request to the specified S3 object key.
- `createMultipartUpload()` — Initiates a multipart upload session in S3, returning an `UploadId` that identifies the session.
- `UploadPart` command — An AWS SDK command that represents the upload of a single part. `createPresignedRequest()` generates a URL that authorises this specific part upload.
- `PartNumber` — The sequential number of the part (1-indexed). Parts can be uploaded in any order but must be assembled in order.
- `completeMultipartUpload()` — Tells S3 to assemble the uploaded parts into the final object. S3 requires the `ETag` of each part.
- `getClient()` — Returns the underlying AWS SDK `S3Client` instance from the Flysystem adapter, enabling advanced S3 operations not exposed by Laravel's FilesystemAdapter.

**Syntax Rules:**

- Pre-signed URLs have a **maximum expiration** (typically 7 days for S3). For multipart uploads, each part URL should have a shorter expiration (e.g., 30 minutes) to limit exposure.
- The `ContentType` parameter in `temporaryUploadUrl()` must match the MIME type the client will send. If omitted, the content type is determined by the client's request.
- Multipart uploads require **all parts to be at least 5 MB** except the last part. Parts smaller than 5 MB (other than the final part) will cause the complete operation to fail.
- The `ETag` returned by S3 after each part upload must be stored and supplied when completing the multipart upload. The `ETag` is provided in the response headers of the part upload.
- The `getClient()` method requires the `league/flysystem-aws-s3-v3` package to be installed and the `s3` disk to be configured with valid credentials.

**Constraints and Limitations:**

- **Multipart upload requires client-side coordination.** The JavaScript code must split the file, upload parts in parallel or sequence, collect ETags, and call the completion endpoint. Laravel cannot initiate chunking on its own.
- **The AWS SDK `getClient()` method is not part of Laravel's public API.** It relies on the internal structure of the Flysystem S3 adapter and may break in future versions. Use with caution and test thoroughly after upgrades.
- **Multipart uploads have a maximum of 10,000 parts.** For a 5 TB file, each part must be at least 500 MB.
- **Incomplete multipart uploads consume storage** in S3 until they are explicitly aborted. Configure a lifecycle rule on the S3 bucket to automatically abort incomplete multipart uploads after a specified number of days.
- **Pre-signed URLs can be replayed** within their validity period. Use short expiration times and consider one-time-use URLs for sensitive uploads.

### Annotated Code Examples

**Example 1: Complete Multipart Upload Flow (Laravel Backend)**

```php
<?php
// File: app/Http/Controllers/S3MultipartController.php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Storage;
use Illuminate\Support\Str;

class S3MultipartController extends Controller
{
    /**
     * Step 1: Initiate a multipart upload and return part URLs.
     */
    public function initiate(Request $request)
    {
        $request->validate([
            'filename' => 'required|string|max:255',
            'fileSize' => 'required|integer|min:1',
            'mimeType' => 'required|string|max:128',
        ]);

        $client = Storage::disk('s3')->getClient();
        $bucket = config('filesystems.disks.s3.bucket');

        // Generate a unique S3 key
        $key = 'uploads/' . date('Y/m') . '/' . Str::uuid() . '_' . basename($request->filename);

        // Initiate the multipart upload
        $multipart = $client->createMultipartUpload([
            'Bucket'      => $bucket,
            'Key'         => $key,
            'ContentType' => $request->mimeType,
        ]);

        $uploadId = $multipart['UploadId'];

        // Calculate parts: 10 MB each, minimum 5 MB
        $partSize   = 10 * 1024 * 1024;
        $totalParts = (int) ceil($request->fileSize / $partSize);

        // Generate pre-signed URLs for each part
        $partUrls = [];
        for ($i = 1; $i <= $totalParts; $i++) {
            $command = $client->getCommand('UploadPart', [
                'Bucket'     => $bucket,
                'Key'        => $key,
                'UploadId'   => $uploadId,
                'PartNumber' => $i,
            ]);

            $presigned = $client->createPresignedRequest($command, '+30 minutes');
            $partUrls[] = [
                'partNumber' => $i,
                'url'        => (string) $presigned->getUri(),
            ];
        }

        return response()->json([
            'uploadId'  => $uploadId,
            'key'       => $key,
            'partSize'  => $partSize,
            'totalParts' => $totalParts,
            'partUrls'  => $partUrls,
        ]);
    }

    /**
     * Step 2: Complete the multipart upload after all parts are uploaded.
     */
    public function complete(Request $request)
    {
        $request->validate([
            'uploadId' => 'required|string',
            'key'      => 'required|string',
            'parts'    => 'required|array|min:1',
            'parts.*.PartNumber' => 'required|integer|min:1',
            'parts.*.ETag'       => 'required|string',
        ]);

        $client = Storage::disk('s3')->getClient();
        $bucket = config('filesystems.disks.s3.bucket');

        // Complete the multipart upload
        $result = $client->completeMultipartUpload([
            'Bucket'          => $bucket,
            'Key'             => $request->key,
            'UploadId'        => $request->uploadId,
            'MultipartUpload' => [
                'Parts' => $request->parts,
            ],
        ]);

        return response()->json([
            'message' => 'Multipart upload completed.',
            'location' => $result['Location'],
            'key'      => $request->key,
        ]);
    }

    /**
     * Step 3: Abort a multipart upload (cleanup).
     */
    public function abort(Request $request)
    {
        $request->validate([
            'uploadId' => 'required|string',
            'key'      => 'required|string',
        ]);

        $client = Storage::disk('s3')->getClient();
        $bucket = config('filesystems.disks.s3.bucket');

        $client->abortMultipartUpload([
            'Bucket'   => $bucket,
            'Key'      => $request->key,
            'UploadId' => $request->uploadId,
        ]);

        return response()->json(['message' => 'Multipart upload aborted.']);
    }
}
```

**Step-by-Step Setup:**

1. Install the S3 Flysystem adapter: `composer require league/flysystem-aws-s3-v3 "^3.0" --with-all-dependencies`.
2. Configure the `s3` disk in `config/filesystems.php` with valid AWS credentials.
3. Register the routes: `POST /s3/initiate` → `initiate`; `POST /s3/complete` → `complete`; `POST /s3/abort` → `abort`.
4. Implement client-side JavaScript to split the file, upload parts to the pre-signed URLs, collect ETags, and call the complete endpoint.
5. Configure an S3 lifecycle rule to abort incomplete multipart uploads after 7 days.

**Expected Output (Initiate):**

```json
{
    "uploadId": "Vk3...",
    "key": "uploads/2025/06/uuid_video.mp4",
    "partSize": 10485760,
    "totalParts": 5,
    "partUrls": [
        {"partNumber": 1, "url": "https://bucket.s3.amazonaws.com/...?X-Amz-Signature=..."},
        {"partNumber": 2, "url": "https://bucket.s3.amazonaws.com/...?X-Amz-Signature=..."},
        ...
    ]
}
```

**Expected Output (Complete):**

```json
{
    "message": "Multipart upload completed.",
    "location": "https://bucket.s3.amazonaws.com/uploads/2025/06/uuid_video.mp4",
    "key": "uploads/2025/06/uuid_video.mp4"
}
```

**Why This Output Occurs:** The `initiate` endpoint calls S3's `CreateMultipartUpload` API to start a multipart session. It then generates pre-signed URLs for each part using the `UploadPart` command. The client uploads each part directly to S3 using these URLs — the Laravel server never receives the file data. After all parts are uploaded, the client sends the ETags to the `complete` endpoint, which calls S3's `CompleteMultipartUpload` API to assemble the parts into the final object. The server's role is limited to generating URLs and coordinating the lifecycle, keeping its CPU and bandwidth usage minimal.

### Real-World Cases

- **Video streaming platforms:** Users upload multi-gigabyte video files directly to S3 using multipart uploads with pre-signed URLs, bypassing the application server entirely and reducing upload times by 50–70%.
- **Cloud storage services:** Web applications that provide a frontend for S3 storage use multipart uploads to handle large files while keeping the application server stateless and scalable.
- **Media asset management systems:** Professional photographers and videographers upload RAW files and high-resolution videos directly to cloud storage, with the application server only handling metadata and post-upload processing.
- **Machine learning data pipelines:** Large datasets (training data, model weights) are uploaded directly to cloud storage from client machines, with the application server coordinating the process without acting as a data relay.

---

## References

- Laravel File Storage Documentation (Master) — https://laravel.com/framework/docs/master/filesystem 
- Laravel File Storage Documentation (Laravel 10.x) — https://laravel.com/framework/docs/10.x/filesystem 
- Laravel Validation: Validating Files — https://laravel.com/docs/validation#validating-files 
- Laravel `UploadedFile` API — https://api.laravel.com/docs/10.x/Illuminate/Http/UploadedFile.html 
- Laravel `Illuminate\Validation\Rules\File` API — https://api.laravel.com/docs/10.x/Illuminate/Validation/Rules/File.html 
- PHP `finfo_file()` Documentation — https://www.php.net/manual/en/function.finfo-file.php 
- PHP File Upload Handling — https://www.php.net/manual/en/features.file-upload.php 
- AWS S3 Multipart Upload Overview — https://docs.aws.amazon.com/AmazonS3/latest/userguide/mpuoverview.html 
- AWS S3 Presigned URLs — https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-presigned-url.html 
- OWASP File Upload Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html 
- League Flysystem AWS S3 V3 Adapter — https://flysystem.thephpleague.com/docs/adapter/aws-s3-v3/ 
- Spatie Flysystem Dropbox — https://github.com/spatie/flysystem-dropbox 
- Pion Laravel Chunk Upload — https://github.com/pion/laravel-chunk-upload 
- DevWizard Laravel Filex — https://github.com/devwizardhq/laravel-filex 
- Laravel S3 Large File Uploader (Community Reference) — https://github.com/CodingGagan/laravel-s3-large-file-uploader 
- CVE-2026-49970 (Laravel-Mediable Path Traversal) — https://feedly.com/cve/CVE-2026-49970