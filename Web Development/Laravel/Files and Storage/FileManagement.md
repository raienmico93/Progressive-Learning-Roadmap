# Laravel File Management — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Laravel File Management is the framework's subsystem for performing the full lifecycle of file operations — writing, reading, downloading, deleting, copying, moving, and streaming — through a unified, driver-agnostic API provided by the `Illuminate\Support\Facades\Storage` facade and the `Illuminate\Filesystem\FilesystemAdapter` class.

**Technical Definition:** Laravel File Management is implemented through the `FilesystemAdapter` class, which wraps a `League\Flysystem\Filesystem` instance and exposes methods such as `put()`, `putFile()`, `putFileAs()`, `get()`, `download()`, `delete()`, `copy()`, `move()`, `readStream()`, `writeStream()`, `temporaryUrl()`, `getVisibility()`, and `setVisibility()`. The facade delegates to the `FilesystemManager`, which resolves named disks from `config/filesystems.php` into `FilesystemAdapter` instances. All operations are driver-agnostic: the same method call works on local, S3, FTP, SFTP, or any custom Flysystem adapter.

**Beginner-Friendly Explanation:** Laravel gives you one set of commands to manage files everywhere — whether they live on your server's hard drive or in the cloud on Amazon S3. You can write files, read them, let users download them, delete them, copy them between different storage locations, and even generate temporary links that expire after a set time. You never have to worry about the underlying storage details; you just tell Laravel *what* to do, and it figures out *how*.

### Key Characteristics

- **Driver-agnostic API:** The same `Storage` facade methods work across local, cloud, and custom storage backends.
- **Resource-based operations:** File contents can be provided as strings, `UploadedFile` instances, `File` objects, or PHP resource handles.
- **Streaming support:** `readStream()` and `writeStream()` enable low-memory file operations for large files.
- **Visibility abstraction:** Files can be declared `public` or `private`, with the underlying permissions translated per driver.
- **Temporary URLs:** S3 and S3-compatible disks support time-limited, signed URLs for secure access to private files.
- **Batch operations:** `delete()` accepts arrays of paths, and `deleteDirectory()` removes entire directory trees.
- **Cross-disk transfers:** Streams enable copying and moving files between different physical disks without loading them into memory.

### Prerequisites

- PHP 8.1 or higher (Laravel 10+; Laravel 11 requires PHP 8.2+).
- A Laravel application with the Storage subsystem configured (see the Laravel File Storage cheat sheet).
- For cloud operations: the `league/flysystem-aws-s3-v3` package and valid cloud credentials.
- Basic understanding of the `Storage` facade and disk configuration.

### Related Programming Areas

- **Flysystem** — The underlying PHP package providing the adapter ecosystem.
- **HTTP Responses** — `download()` and `response()` return Symfony HTTP response objects.
- **Streams** — PHP's stream abstraction enables low-memory file operations.
- **Visibility & Permissions** — File-level and bucket-level access control.
- **Cloud Storage** — S3 presigned URLs and multipart uploads.

### Core Concepts / Features

1. Upload (Writing files via `Storage::put()`, `putFile()`, and resource handles)
2. Download (Generating standard file downloads using `Storage::download()`)
3. Delete (Single and batch purges via `Storage::delete()`)
4. Copy & Move (Duplicating and migrating files across different physical disks)
5. Visibility (Managing public vs. private access settings at the file and bucket level)
6. Signed URLs (Generating temporary, time-restricted access links via `Storage::temporaryUrl()`)
7. Stream Management (Reading and writing files with low-memory footprints using `Storage::readStream()` and `writeStream()`)

---

## 1. Upload

### Definitions

**Core Definition:** Upload in the context of file management refers to writing file contents to a storage disk — whether the content originates from a user upload, a string, a resource handle, or an external source.

**Technical Definition:** Laravel provides three primary methods for writing files: `put()`, which accepts a path and contents (string, `UploadedFile`, `File`, or `resource`) and writes the file directly; `putFile()`, which accepts a directory and a file instance, generates a unique filename via `hashName()`, and streams the file to the destination; and `putFileAs()`, which is identical to `putFile()` but accepts an explicit filename. All three methods delegate to Flysystem's `writeStream()` or `write()` operations, depending on the input type. When a PHP resource handle is passed, Flysystem uses `writeStream()` to avoid loading the file into memory.

**Beginner-Friendly Explanation:** "Upload" here means "save a file to storage." You can save a file that a user uploaded through a form, or you can save a file you generated in code (like a PDF or a log entry). Laravel gives you three ways: `put()` for when you have the content ready (as a string or stream), `putFile()` for when you have a file object and want Laravel to name it automatically, and `putFileAs()` for when you have a file object but want to choose the name yourself.

### Purposes

- To persist file contents to a storage disk with automatic or explicit naming.
- To store user-uploaded files with unique, safe filenames.
- To write generated content (reports, exports, logs) to disk without intermediate files.
- To stream large files from external sources (HTTP responses, resource handles) directly to storage.
- To target specific disks for different file categories.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// put() — write contents directly (string, UploadedFile, File, or resource)
Storage::put(string $path, $contents, array $options = []): bool

// putFile() — store a file with automatic unique naming
Storage::putFile(string $path, $file = null, $options = []): string|false

// putFileAs() — store a file with an explicit name
Storage::putFileAs(string $path, $file, $name = null, $options = []): string|false
```

**Component Breakdown:**

- `$path` — The destination directory (for `putFile`/`putFileAs`) or full file path (for `put`).
- `$contents` — The file contents: a string, `UploadedFile`, `File`, `StreamInterface`, or PHP `resource`.
- `$file` — The file to store: an `UploadedFile`, `File`, or string path.
- `$name` — The explicit filename for `putFileAs()`.
- `$options` — Driver-specific options (e.g., `['visibility' => 'public']`).

```php
// Using put() with a string
Storage::put('file.txt', 'Contents');

// Using put() with an uploaded file
Storage::put('avatars/1', $request->file('avatar'));

// Using put() with a resource handle
$stream = fopen('/path/to/large-file.zip', 'r');
Storage::put('backups/large-file.zip', $stream);
fclose($stream);

// Using putFile() — automatic naming
$path = Storage::putFile('avatars', $request->file('avatar'));

// Using putFileAs() — explicit naming
$path = Storage::putFileAs('avatars', $request->file('avatar'), 'user-1.jpg');
```

**Syntax Rules:**

- When `putFile()` is called without a filename, Laravel generates a unique ID and determines the extension from the file's MIME type.
- The `putFile()` and `putFileAs()` methods return the relative path of the stored file, which should be persisted in the database.
- When a resource handle is passed to `put()`, Flysystem streams the data, avoiding loading the entire file into memory.
- The `visibility` option can be passed as a string (`'public'` or `'private'`) or as part of an options array.
- File paths are normalized using `League\Flysystem\WhitespacePathNormalizer::normalizePath`, which removes unprintable and invalid Unicode characters.

**Constraints and Limitations:**

- **`put()` with a string loads the entire content into memory.** For large files, use `putFile()` or a resource handle.
- **`putFile()` and `putFileAs()` require an `UploadedFile`, `File`, or string path.** Passing a raw string of contents to `putFile()` will be interpreted as a file path, not file contents.
- **The `visibility` option is only meaningful on drivers that support per-file permissions** (local, S3). On FTP, visibility is determined by the FTP server's configuration.
- **Concurrent calls to `putFileAs()` with the same filename** will result in the later call overwriting the earlier one. Use unique filenames for concurrent operations.

### Annotated Code Examples

**Example 1: Uploading with put(), putFile(), and Resource Handles**

```php
<?php
// File: app/Http/Controllers/UploadController.php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Storage;
use Illuminate\Http\File;

class UploadController extends Controller
{
    public function upload(Request $request)
    {
        $request->validate([
            'avatar'  => 'required|image|max:2048',
            'report'  => 'required|file|mimes:pdf|max:10240',
        ]);

        // Method 1: putFile() — automatic unique naming
        // Generates: avatars/<uuid>.jpg (extension from MIME type)
        $avatarPath = Storage::putFile('avatars', $request->file('avatar'));

        // Method 2: putFileAs() — explicit naming
        // Stores: reports/2025-06-report.pdf
        $reportPath = Storage::putFileAs(
            'reports',
            $request->file('report'),
            date('Y-m') . '-report.pdf'
        );

        // Method 3: put() with a resource handle — low memory
        // Reads a large file from disk and writes it to storage
        $sourcePath = '/var/backups/database.sql';
        $stream = fopen($sourcePath, 'rb');
        Storage::put('backups/database-' . date('Y-m-d') . '.sql', $stream);
        fclose($stream);

        // Method 4: put() with a string — simple content
        Storage::put('logs/upload-' . date('Y-m-d') . '.log',
            "Uploaded avatar: {$avatarPath}\nUploaded report: {$reportPath}\n"
        );

        return response()->json([
            'avatar' => $avatarPath,
            'report' => $reportPath,
        ]);
    }
}
```

**Step-by-Step Setup:**

1. Ensure the `avatars`, `reports`, `backups`, and `logs` directories exist or allow Laravel to create them automatically.
2. Ensure the default disk is configured (local or S3).
3. Register the route: `POST /upload` → `UploadController@upload`.

**Expected Output:**

```json
{
    "avatar": "avatars/8b2f3c1d5e6a7b8c9d0e1f2a3b4c5d6e.jpg",
    "report": "reports/2025-06-report.pdf"
}
```

**Why This Output Occurs:** `putFile()` generates a unique filename using `hashName()`, which is an MD5 hash of the file's contents plus the server-guessed extension. `putFileAs()` uses the explicit filename `2025-06-report.pdf`. The resource handle approach streams the database dump from the local filesystem to the storage disk without loading it into PHP's memory. The string-based `put()` writes a simple log entry.

---

**Example 2: Streaming External HTTP Responses to Storage**

```php
<?php
// File: app/Services/DocumentService.php

namespace App\Services;

use Illuminate\Support\Facades\Http;
use Illuminate\Support\Facades\Storage;

class DocumentService
{
    /**
     * Download a document from an external URL and store it locally.
     * Uses the resource() method to stream the HTTP response directly.
     */
    public function fetchAndStore(string $url, string $reference): array
    {
        // Step 1: Fetch the document with a timeout
        $response = Http::timeout(60)->get($url);

        if (!$response->successful()) {
            throw new \RuntimeException("Failed to fetch: {$response->status()}");
        }

        // Step 2: Stream the response body directly to storage
        // The resource() method converts the HTTP response body into a PHP stream
        $path = "documents/{$reference}/original.pdf";
        Storage::put($path, $response->resource());

        return [
            'path' => $path,
            'size' => Storage::size($path),
        ];
    }
}
```

**Expected Output:**

```php
[
    'path' => 'documents/REF-001/original.pdf',
    'size' => 2048576,
]
```

**Why This Output Occurs:** The `Http::get()->resource()` method converts the HTTP response body into a PHP stream resource, which is then passed to `Storage::put()`. Flysystem streams the data from the HTTP response directly to the storage disk without loading the entire file into memory. This pattern is particularly valuable when downloading large files from external sources.

### Real-World Cases

- **User avatar uploads:** `Storage::putFile('avatars', $request->file('avatar'))` generates a unique filename and stores the image, returning the path for database persistence.
- **Report generation:** A scheduled command generates a PDF report and uses `Storage::put()` with a resource handle to write it to cloud storage without loading the entire PDF into memory.
- **External data ingestion:** A webhook receiver downloads a file from a third-party API and streams it directly to S3 using `Storage::put()` with the HTTP response's `resource()` method.
- **Log archiving:** A daily command reads the previous day's log file as a resource, compresses it, and writes it to S3 with a date-stamped filename.

---

## 2. Download

### Definitions

**Core Definition:** Download is the process of generating an HTTP response that delivers a stored file to the user's browser, either as a direct download (prompting a save dialog) or as an inline stream (rendered in the browser).

**Technical Definition:** Laravel's `Storage::download()` method returns a `Symfony\Component\HttpFoundation\StreamedResponse` that sets the `Content-Disposition: attachment` header, prompting the browser to download the file. The method accepts a path, an optional custom filename, and optional HTTP headers. For inline display (e.g., viewing a PDF in the browser), `Storage::response()` returns a streamed response without the attachment disposition. Both methods stream the file contents in chunks, avoiding loading the entire file into PHP memory.

**Beginner-Friendly Explanation:** Once a file is stored, you often need to let users download it. Laravel's `Storage::download()` does all the work: it reads the file from wherever it's stored, streams it to the user's browser, and tells the browser to save it with a specific filename. If you want the file to open in the browser instead of downloading (like a PDF preview), you use `Storage::response()` instead.

### Purposes

- To deliver stored files to users as downloadable attachments.
- To control the filename presented to the user (which may differ from the stored filename).
- To set custom HTTP headers (Content-Type, Cache-Control, etc.) on the download response.
- To serve files from any storage disk through a single, consistent API.
- To stream large files to the browser without exhausting server memory.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// download() — force download with Content-Disposition: attachment
Storage::download(string $path, string $name = null, array $headers = []): StreamedResponse

// response() — inline display (no attachment disposition)
Storage::response(string $path, string $name = null, array $headers = []): StreamedResponse

// url() — generate a public URL (for public files)
Storage::url(string $path): string
```

**Component Breakdown:**

- `$path` — The relative path of the file on the disk.
- `$name` — The filename presented to the user (defaults to the basename of `$path`).
- `$headers` — Additional HTTP headers to include in the response.

```php
// Basic download
return Storage::download('documents/report.pdf');

// Download with custom filename and headers
return Storage::download(
    'documents/report.pdf',
    'quarterly-report-2025.pdf',
    ['Content-Type' => 'application/pdf']
);

// Inline display (e.g., PDF viewer in browser)
return Storage::response('documents/report.pdf');

// Generate a public URL (for files on the public disk)
$url = Storage::url('avatars/user1.jpg');
```

**Syntax Rules:**

- `download()` sets the `Content-Disposition` header to `attachment; filename="..."`, triggering a browser download.
- `response()` does not set the attachment disposition, allowing the browser to display the file inline (if the MIME type is supported).
- The `$name` parameter in `download()` controls the filename the user sees, not the stored filename.
- Both methods return `StreamedResponse` instances, so they stream the file rather than loading it into memory.
- For files on the `public` disk, `Storage::url()` generates a direct URL that the web server serves via the symbolic link.

**Constraints and Limitations:**

- **`download()` and `response()` proxy the file through the application server.** For large files on cloud storage, this consumes application server bandwidth. Consider using `temporaryUrl()` for direct cloud downloads.
- **Streamed responses do not support HTTP range requests by default.** For video streaming with seeking, a custom streaming response with `Accept-Ranges` and `Content-Range` headers is required.
- **The `download()` method does not verify authorisation.** You must implement authorisation checks in your controller before calling it.
- **The `response()` method may not render all file types inline.** Browsers have varying support for inline display of different MIME types.

### Annotated Code Examples

**Example 1: Secure File Download with Authorisation**

```php
<?php
// File: app/Http/Controllers/DocumentController.php

namespace App\Http\Controllers;

use App\Models\Document;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Storage;

class DocumentController extends Controller
{
    public function download(Request $request, Document $document)
    {
        // Step 1: Authorisation check
        if (!$request->user()->can('download', $document)) {
            abort(403, 'You do not have permission to download this document.');
        }

        // Step 2: Verify the file exists
        if (!Storage::disk('local')->exists($document->path)) {
            abort(404, 'File not found.');
        }

        // Step 3: Increment download count for auditing
        $document->increment('download_count');

        // Step 4: Return the download response with custom headers
        return Storage::disk('local')->download(
            $document->path,
            $document->original_name,
            [
                'Content-Type'        => $document->mime_type,
                'Content-Disposition' => 'attachment',
                'Cache-Control'       => 'no-cache, must-revalidate',
            ]
        );
    }
}
```

**Expected Output:**

The browser receives the file as a download with the original filename. The response headers include:

```
Content-Type: application/pdf
Content-Disposition: attachment; filename="quarterly-report.pdf"
Cache-Control: no-cache, must-revalidate
```

**Why This Output Occurs:** The `download()` method streams the file from the `local` disk and sets the `Content-Disposition` header to `attachment`, triggering the browser's save dialog. The custom `Content-Type` and `Cache-Control` headers provide additional control over browser behaviour. The `increment()` call updates the download count for audit purposes.

---

**Example 2: Inline PDF Display with Response**

```php
<?php
// File: app/Http/Controllers/PdfController.php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Storage;

class PdfController extends Controller
{
    public function view(Request $request, string $filename)
    {
        $path = 'documents/' . basename($filename);

        // Verify existence before serving
        if (!Storage::disk('local')->exists($path)) {
            abort(404);
        }

        // Return an inline response — browser displays the PDF
        return Storage::disk('local')->response(
            $path,
            null, // Use stored filename
            [
                'Content-Type'        => 'application/pdf',
                'Content-Disposition' => 'inline; filename="' . basename($filename) . '"',
            ]
        );
    }
}
```

**Expected Output:**

The browser displays the PDF inline (in a PDF viewer). The response headers include:

```
Content-Type: application/pdf
Content-Disposition: inline; filename="report.pdf"
```

**Why This Output Occurs:** The `response()` method streams the file without setting the attachment disposition. The explicit `Content-Disposition: inline` header tells the browser to display the file in the browser window rather than downloading it. The `Content-Type: application/pdf` header ensures the browser recognises the file as a PDF and invokes its built-in viewer.

### Real-World Cases

- **Document management systems:** Users download invoices, contracts, and reports through authenticated controller actions using `Storage::download()` with authorisation checks.
- **Media galleries:** Images are served inline using `Storage::response()` with appropriate `Content-Type` headers, allowing the browser to render them directly.
- **Healthcare portals:** Medical records are streamed to authenticated patients using `Storage::download()`, with every access logged for compliance.
- **E-learning platforms:** Course materials (PDFs, videos) are served through `Storage::response()` for inline viewing, or `Storage::download()` for offline access.

---

## 3. Delete

### Definitions

**Core Definition:** Delete is the process of removing files or directories from a storage disk, either individually or in batches.

**Technical Definition:** Laravel's `Storage::delete()` method accepts a single file path or an array of file paths and removes the corresponding files from the disk. The `deleteDirectory()` method removes a directory and all of its contents recursively. The `delete()` method returns `true` on success and `false` on failure (unless the `throw` option is enabled on the disk, in which case it throws an exception). For batch deletes on S3, Laravel does not use the S3 batch delete API; instead, it issues individual `DeleteObject` calls for each file.

**Beginner-Friendly Explanation:** Deleting files in Laravel is straightforward: you pass the path (or an array of paths) to `Storage::delete()`. If you want to wipe out an entire directory and everything inside it, you use `Storage::deleteDirectory()`. The methods work the same way whether the files are on your local server or in the cloud.

### Purposes

- To remove individual files that are no longer needed.
- To batch-delete multiple files in a single method call.
- To recursively remove entire directory trees (e.g., temporary upload directories, user-specific folders).
- To clean up files after processing (e.g., after generating a report, after a user deletes their account).
- To manage storage quotas by removing obsolete or orphaned files.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// Delete a single file
Storage::delete(string $path): bool

// Delete multiple files
Storage::delete(array $paths): bool

// Delete a directory and all its contents
Storage::deleteDirectory(string $directory): bool

// Delete files on a specific disk
Storage::disk('s3')->delete('folder/file.jpg');
```

**Component Breakdown:**

- `$path` — The relative path of the file to delete.
- `$paths` — An array of relative file paths to delete.
- `$directory` — The directory to remove recursively.

```php
// Single file
Storage::delete('file.jpg');

// Multiple files
Storage::delete(['file1.jpg', 'file2.jpg', 'file3.jpg']);

// Delete a directory and all its contents
Storage::deleteDirectory('temp-uploads/session-123');

// Delete on a specific disk
Storage::disk('s3')->delete('backups/2024-01-01.sql');
```

**Syntax Rules:**

- The `delete()` method accepts either a string (single file) or an array of strings (multiple files).
- The `deleteDirectory()` method removes the directory **and** all of its contents recursively.
- Both methods return `true` on success and `false` on failure (unless `'throw' => true` is set in the disk configuration).
- When deleting multiple files, Laravel iterates over the array and calls the driver's delete method for each file individually.

**Constraints and Limitations:**

- **No batch delete API for S3.** Laravel issues individual `DeleteObject` calls for each file in an array, which can be slow for large batches. For bulk deletion of thousands of files, consider using the AWS SDK's `deleteObjects` API directly.
- **`deleteDirectory()` is irreversible.** There is no "undo" or recycle bin; deleted files are permanently removed.
- **The `delete()` method does not check file existence.** Deleting a non-existent file returns `true` (no error) unless the `throw` option is enabled.
- **Some cloud providers impose rate limits** on delete operations. Deleting thousands of files in rapid succession may trigger throttling.

### Annotated Code Examples

**Example 1: Batch Delete of Processed Files**

```php
<?php
// File: app/Console/Commands/CleanupProcessedFiles.php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use Illuminate\Support\Facades\Storage;

class CleanupProcessedFiles extends Command
{
    protected $signature = 'files:cleanup-processed';
    protected $description = 'Delete processed files older than 7 days';

    public function handle(): int
    {
        $disk = Storage::disk('local');
        $cutoff = now()->subDays(7)->timestamp;
        $toDelete = [];

        // Step 1: Collect files eligible for deletion
        foreach ($disk->files('processed') as $file) {
            if ($disk->lastModified($file) < $cutoff) {
                $toDelete[] = $file;
            }
        }

        if (empty($toDelete)) {
            $this->info('No files to delete.');
            return Command::SUCCESS;
        }

        // Step 2: Batch delete
        $disk->delete($toDelete);

        $this->info('Deleted ' . count($toDelete) . ' files.');

        return Command::SUCCESS;
    }
}
```

**Expected Output:**

```
Deleted 42 files.
```

**Why This Output Occurs:** The command iterates over all files in the `processed` directory, checks each file's `lastModified` timestamp against the 7-day cutoff, and collects eligible paths into an array. The `delete()` method accepts the array and removes each file from the disk.

---

**Example 2: Recursive Directory Deletion for User Account Cleanup**

```php
<?php
// File: app/Http/Controllers/UserController.php

namespace App\Http\Controllers;

use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Storage;

class UserController extends Controller
{
    public function destroy(Request $request, User $user)
    {
        // Step 1: Authorisation check
        if ($request->user()->cannot('delete', $user)) {
            abort(403);
        }

        // Step 2: Delete the user's storage directory recursively
        // This removes all files in the user's folder, including subdirectories
        if (Storage::disk('local')->exists("users/{$user->id}")) {
            Storage::disk('local')->deleteDirectory("users/{$user->id}");
        }

        // Step 3: Delete public assets
        $publicFiles = Storage::disk('public')->files("avatars/{$user->id}");
        if (!empty($publicFiles)) {
            Storage::disk('public')->delete($publicFiles);
        }

        // Step 4: Delete the user record from the database
        $user->delete();

        return response()->json(['message' => 'User and all associated files deleted.']);
    }
}
```

**Expected Output:**

```json
{
    "message": "User and all associated files deleted."
}
```

**Why This Output Occurs:** The `deleteDirectory()` method removes the entire `users/{id}` directory tree from the `local` disk, including all nested subdirectories and files. The `files()` method returns all files directly within the `avatars/{id}` directory on the `public` disk, and the `delete()` method removes them in a batch. This pattern ensures that no orphaned files remain after user deletion.

### Real-World Cases

- **Temporary file cleanup:** Scheduled commands delete temporary upload chunks and intermediate files older than a specified age using `Storage::delete()` with arrays.
- **User account deletion:** All of a user's files (avatars, documents, uploads) are removed recursively using `Storage::deleteDirectory()` when they delete their account.
- **Log rotation:** Old log files are deleted in batches based on their modification date, freeing storage space on local disks.
- **Backup retention:** S3 backup files older than the retention policy are deleted in batches using `Storage::disk('s3')->delete()`.

---

## 4. Copy & Move

### Definitions

**Core Definition:** Copy and move are operations that duplicate or relocate files, either within the same disk or across different physical storage disks.

**Technical Definition:** `Storage::copy()` creates a duplicate of a file at a new path on the same disk. `Storage::move()` renames or relocates a file within the same disk. For cross-disk operations, Laravel's `copy()` and `move()` methods only work within a single disk; transferring between disks requires using `readStream()` and `writeStream()` to stream the file from the source disk to the destination disk. The stream-based approach avoids loading the file into memory and works reliably across drivers (S3, SFTP, local, etc.).

**Beginner-Friendly Explanation:** If you need to duplicate a file or move it to a different folder on the same disk, Laravel's `copy()` and `move()` methods handle it. But if you need to move a file from one storage system to another (like from local storage to S3), you have to stream it using `readStream()` and `writeStream()`. This streams the file in small chunks, so even very large files can be transferred without using much memory.

### Purposes

- To duplicate files for backup, versioning, or archival purposes.
- To rename files within the same disk.
- To migrate files between different physical storage disks (e.g., local to S3, S3 to local).
- To reorganise directory structures by moving files to new locations.
- To transfer files between cloud providers without downloading them to the application server.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// copy() — duplicate a file within the same disk
Storage::copy(string $from, string $to): bool

// move() — rename or relocate a file within the same disk
Storage::move(string $from, string $to): bool

// Cross-disk transfer using streams
Storage::disk('destination')->writeStream(
    $destinationPath,
    Storage::disk('source')->readStream($sourcePath)
);
```

**Component Breakdown:**

- `$from` — The current path of the file.
- `$to` — The new path for the copy or move.
- `readStream()` — Opens a read stream on the source file.
- `writeStream()` — Writes the stream to the destination disk.

```php
// Copy within the same disk
Storage::copy('old/file.jpg', 'new/file.jpg');

// Move within the same disk (also renames)
Storage::move('old/file.jpg', 'new/file.jpg');

// Cross-disk copy using streams (local to S3)
Storage::disk('s3')->writeStream(
    'backups/file.zip',
    Storage::disk('local')->readStream('backups/file.zip')
);

// Cross-disk move (copy then delete)
Storage::disk('s3')->writeStream(
    'backups/file.zip',
    Storage::disk('local')->readStream('backups/file.zip')
);
Storage::disk('local')->delete('backups/file.zip');
```

**Syntax Rules:**

- `copy()` and `move()` operate within a **single disk**. To transfer between disks, you must use streams or the `Storage::put()` method with a stream resource.
- The `move()` method is equivalent to a rename: if the source and destination are on the same filesystem, it may be an atomic operation.
- For cross-disk transfers, the stream-based approach is preferred over `get()` + `put()` because it avoids loading the entire file into memory.
- The `readStream()` method returns a PHP resource that can be passed directly to `writeStream()` on another disk.

**Constraints and Limitations:**

- **`copy()` and `move()` do not work across disks.** Attempting to copy from one disk to another will throw an error or fail silently.
- **Cross-disk moves require two operations:** write the file to the destination disk, then delete it from the source disk. There is no atomic cross-disk move in Laravel.
- **Stream-based transfers may fail for very large files** if the destination disk has a maximum file size limit (e.g., S3's 5 TB object limit).
- **Network interruptions during cross-disk transfers** can result in partially transferred files. Consider implementing retry logic or using the cloud provider's native copy functionality.

### Annotated Code Examples

**Example 1: Cross-Disk File Migration Using Streams**

```php
<?php
// File: app/Console/Commands/MigrateFilesToS3.php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use Illuminate\Support\Facades\Storage;

class MigrateFilesToS3 extends Command
{
    protected $signature = 'files:migrate-to-s3 {directory}';
    protected $description = 'Migrate files from local disk to S3 using streams';

    public function handle(): int
    {
        $directory = $this->argument('directory');
        $localDisk = Storage::disk('local');
        $s3Disk = Storage::disk('s3');

        // Step 1: Get all files recursively from the local directory
        $files = $localDisk->allFiles($directory);

        $this->info("Found " . count($files) . " files to migrate.");

        $bar = $this->output->createProgressBar(count($files));
        $bar->start();

        foreach ($files as $file) {
            // Step 2: Skip if already on S3
            if ($s3Disk->exists($file)) {
                $bar->advance();
                continue;
            }

            // Step 3: Stream from local to S3
            // readStream() opens a read stream on the source
            // writeStream() writes the stream to the destination
            $s3Disk->writeStream($file, $localDisk->readStream($file));

            // Step 4: Verify the transfer succeeded
            if (!$s3Disk->exists($file)) {
                $this->error("Failed to migrate: {$file}");
                continue;
            }

            // Step 5: Delete the local copy (move, not copy)
            $localDisk->delete($file);

            $bar->advance();
        }

        $bar->finish();
        $this->newLine();
        $this->info('Migration complete.');

        return Command::SUCCESS;
    }
}
```

**Step-by-Step Setup:**

1. Ensure the `s3` disk is configured with valid AWS credentials.
2. Ensure the `local` disk contains the files to migrate.
3. Run `php artisan files:migrate-to-s3 backups` to migrate the `backups` directory.

**Expected Output:**

```
Found 150 files to migrate.
... (progress bar) ...
Migration complete.
```

**Why This Output Occurs:** The command iterates over every file in the specified directory. For each file, it opens a read stream from the local disk and writes that stream to the S3 disk using `writeStream()`. This streams the file in small chunks, so even large files (gigabytes) are transferred without loading them into memory. After a successful transfer, the local copy is deleted, completing the "move" operation.

---

**Example 2: Copy and Move Within the Same Disk**

```php
<?php
// File: app/Http/Controllers/FileManagementController.php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Storage;

class FileManagementController extends Controller
{
    public function archive(Request $request, string $filename)
    {
        $path = 'documents/' . basename($filename);

        if (!Storage::disk('local')->exists($path)) {
            abort(404);
        }

        // Copy the file to the archive directory
        $archivePath = 'archive/' . date('Y') . '/' . basename($filename);
        Storage::disk('local')->copy($path, $archivePath);

        // Move the original to a 'processed' directory (rename)
        $processedPath = 'processed/' . basename($filename);
        Storage::disk('local')->move($path, $processedPath);

        return response()->json([
            'archived' => $archivePath,
            'processed' => $processedPath,
        ]);
    }
}
```

**Expected Output:**

```json
{
    "archived": "archive/2025/file.pdf",
    "processed": "processed/file.pdf"
}
```

**Why This Output Occurs:** The `copy()` method creates a duplicate of `documents/file.pdf` at `archive/2025/file.pdf`. The `move()` method then relocates the original `documents/file.pdf` to `processed/file.pdf` (which is effectively a rename). Both operations are performed within the same `local` disk.

### Real-World Cases

- **Backup systems:** Nightly commands copy database dumps from the local disk to S3 using stream-based migration, then delete the local copies.
- **Content delivery:** Images uploaded to a local disk are copied to a CDN-backed S3 disk using streams, and the local copy is retained for processing.
- **Storage tiering:** Files older than 30 days are moved from S3 Standard to S3 Glacier using the AWS SDK's copy operation, or streamed through Laravel if the bucket configuration allows.
- **User file migration:** When a user upgrades their account, their files are copied from a shared disk to a dedicated disk using stream-based transfers.

---

## 5. Visibility

### Definitions

**Core Definition:** Visibility is an abstraction of file permissions that determines whether a stored file is publicly accessible (readable by anyone with the URL) or private (accessible only through authenticated application routes).

**Technical Definition:** In Laravel's Flysystem integration, visibility is expressed as either `public` or `private`. On local disks, `public` maps to file permissions `0644` and directory permissions `0755`; `private` maps to `0600` and `0700` respectively. On S3, `public` sets the object ACL to `public-read`; `private` leaves the object ACL unset (default private). Visibility can be set at write time (via the `put()` options array) or modified after storage using `setVisibility()`. The `getVisibility()` method retrieves the current visibility of a stored file.

**Beginner-Friendly Explanation:** Some files should be visible to anyone (like a profile picture), while others should only be accessible to authorised users (like a private document). Laravel calls this "visibility." You can mark a file as `public` (anyone with the link can see it) or `private` (only your application can serve it). The same setting works on local disks and cloud storage — Laravel translates it into the appropriate permissions for each system.

### Purposes

- To control whether a file is directly accessible via its URL or only through authenticated application routes.
- To set appropriate file permissions on local disks (0644 for public, 0600 for private).
- To configure S3 object ACLs for public-read or private access.
- To change a file's visibility after it has been stored.
- To audit and enforce visibility policies across different storage disks.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// Set visibility at write time
Storage::put('file.jpg', $contents, 'public');
Storage::put('file.jpg', $contents, ['visibility' => 'private']);

// Retrieve visibility after storage
$visibility = Storage::getVisibility('file.jpg'); // 'public' or 'private'

// Change visibility
Storage::setVisibility('file.jpg', 'public');
Storage::setVisibility('file.jpg', 'private');

// Configure default visibility for a disk
'public' => [
    'driver'     => 'local',
    'root'       => storage_path('app/public'),
    'url'        => env('APP_URL') . '/storage',
    'visibility' => 'public',
],
```

**Component Breakdown:**

- `'public'` — File is accessible to anyone with the URL. On local disks, permissions are set to 0644 (files) and 0755 (directories).
- `'private'` — File is only accessible through application routes. On local disks, permissions are set to 0600 (files) and 0700 (directories).
- `getVisibility($path)` — Returns the current visibility string.
- `setVisibility($path, $visibility)` — Changes the visibility of a stored file.
- The `visibility` configuration key on a disk sets the default visibility for all files written to that disk.

**Syntax Rules:**

- Visibility can be passed as a string (`'public'` or `'private'`) or as part of an options array (`['visibility' => 'public']`).
- The `putFile()` and `putFileAs()` methods accept a visibility argument as their last parameter.
- On local disks, changing visibility via `setVisibility()` calls `chmod()` on the file.
- On S3, changing visibility via `setVisibility()` issues a `PutObjectAcl` API call.
- The `public` disk in `config/filesystems.php` is pre-configured with `'visibility' => 'public'`.

**Constraints and Limitations:**

- **Visibility is only meaningful on drivers that support per-file permissions.** FTP and SFTP disks do not support visibility changes after storage.
- **S3 buckets with ACLs disabled** (the default for newer buckets) do not support per-object ACLs. In such cases, use bucket policies or presigned URLs instead of visibility.
- **The `visibility` setting does not control access to files on the `local` disk** in the sense of web access — the `local` disk's root is outside the web root, so files are never directly accessible regardless of permissions.
- **Public files on S3 are accessible to anyone who knows the URL.** There is no authentication or access control for `public-read` objects.

### Annotated Code Examples

**Example 1: Setting and Changing Visibility**

```php
<?php
// File: app/Http/Controllers/VisibilityController.php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Storage;

class VisibilityController extends Controller
{
    public function store(Request $request)
    {
        $request->validate([
            'photo' => 'required|image|max:2048',
        ]);

        // Step 1: Store with public visibility
        // The file is immediately accessible via URL
        $path = Storage::putFile(
            'photos',
            $request->file('photo'),
            'public'  // Visibility option
        );

        // Step 2: Verify the visibility
        $visibility = Storage::getVisibility($path);
        // Returns 'public'

        return response()->json([
            'path'       => $path,
            'visibility' => $visibility,
            'url'        => Storage::url($path),
        ]);
    }

    public function makePrivate(Request $request)
    {
        $request->validate(['path' => 'required|string']);

        // Step 3: Change visibility to private
        Storage::setVisibility($request->path, 'private');

        // Step 4: Verify the change
        $visibility = Storage::getVisibility($request->path);

        return response()->json([
            'path'       => $request->path,
            'visibility' => $visibility, // 'private'
        ]);
    }
}
```

**Expected Output (Store):**

```json
{
    "path": "photos/abc123.jpg",
    "visibility": "public",
    "url": "https://example.com/storage/photos/abc123.jpg"
}
```

**Expected Output (Make Private):**

```json
{
    "path": "photos/abc123.jpg",
    "visibility": "private"
}
```

**Why This Output Occurs:** The `putFile()` method with the `'public'` argument stores the file with public visibility. On a local disk, this sets file permissions to `0644`. On S3, it sets the object ACL to `public-read`. The `getVisibility()` method reads the current visibility. The `setVisibility()` method changes it — on local disks via `chmod()`, on S3 via `PutObjectAcl`. After the change, the file is no longer accessible via the public URL.

---

**Example 2: Configuring Disk-Level Visibility**

```php
// File: config/filesystems.php

'disks' => [
    // Public disk — all files are public by default
    'public' => [
        'driver'     => 'local',
        'root'       => storage_path('app/public'),
        'url'        => env('APP_URL') . '/storage',
        'visibility' => 'public',
    ],

    // Private disk — all files are private by default
    'private' => [
        'driver'     => 'local',
        'root'       => storage_path('app/private'),
        'visibility' => 'private',
    ],
],
```

```php
// Usage — visibility is inherited from the disk configuration
Storage::disk('public')->put('avatars/user1.jpg', $contents);
// File is public (0644 on local, public-read on S3)

Storage::disk('private')->put('contracts/contract1.pdf', $contents);
// File is private (0600 on local, private on S3)
```

**Expected Output:**

- Files on the `public` disk are accessible via `APP_URL/storage/...`.
- Files on the `private` disk are only accessible through controller actions.

**Why This Output Occurs:** The `visibility` configuration key on each disk sets the default visibility for all files written to that disk. When `put()` is called without an explicit visibility option, Laravel uses the disk's default. This pattern simplifies code by eliminating the need to specify visibility on every write operation.

### Real-World Cases

- **Avatar uploads:** Stored on the `public` disk with `public` visibility, allowing direct URL access and browser caching.
- **Contract and invoice storage:** Stored on a `private` disk with `private` visibility, served only through authenticated controller actions.
- **Compliance-driven access control:** Files are stored as `private` and their visibility is audited periodically to ensure compliance with data protection regulations.
- **Content management systems:** Editor-uploaded images are stored as `public` for immediate display, while draft documents are stored as `private` until published.

---

## 6. Signed URLs

### Definitions

**Core Definition:** A signed URL is a time-limited, cryptographically signed URL that grants temporary access to a private file without requiring the user to authenticate through the application.

**Technical Definition:** Laravel's `Storage::temporaryUrl()` method generates a presigned URL for S3 and S3-compatible disks. The URL includes query parameters for the access key ID, expiration timestamp, and a cryptographic signature computed from the request details and the secret key. Any request to the URL within the validity period is authorised by the storage service directly, without involving the Laravel application. For disks that do not natively support temporary URLs (e.g., local disks), Laravel provides the `buildTemporaryUrlsUsing()` method to define custom temporary URL generation logic, typically using Laravel's `URL::temporarySignedRoute()`.

**Beginner-Friendly Explanation:** A signed URL is like a temporary key to a private file. Instead of making the file public or forcing the user to log in every time they view it, you generate a special link that only works for a short time (like 5 minutes or 1 hour). Anyone with the link can access the file during that window, and then the link expires automatically. This is perfect for things like private videos, medical records, or downloadable reports.

### Purposes

- To provide temporary, secure access to private files without exposing them permanently.
- To offload file serving from the application server to the cloud storage provider.
- To enable direct browser-to-cloud downloads for large files, bypassing server bandwidth limits.
- To generate time-limited access links for sharing private content with external parties.
- To support use cases where the user's browser needs to access a private file directly (e.g., video streaming, PDF viewing).

### Syntax Rules and Structure

#### Complete General Syntax

```php
// Generate a temporary URL for S3 (or S3-compatible disks)
$url = Storage::disk('s3')->temporaryUrl(
    string $path,
    DateTimeInterface $expiration,
    array $options = []
): string;

// Custom temporary URL builder for non-S3 disks
Storage::disk('local')->buildTemporaryUrlsUsing(
    function (string $path, DateTimeInterface $expiration, array $options) {
        return URL::temporarySignedRoute(
            'files.download',
            $expiration,
            ['path' => $path]
        );
    }
);
```

**Component Breakdown:**

- `$path` — The relative path of the file on the disk.
- `$expiration` — A `DateTimeInterface` instance specifying when the URL should expire.
- `$options` — Additional options passed to the underlying driver (e.g., response headers).
- `buildTemporaryUrlsUsing()` — A macro that registers a custom temporary URL builder for a specific disk.

```php
// S3 temporary URL (valid for 30 minutes)
$url = Storage::disk('s3')->temporaryUrl(
    'private/report.pdf',
    now()->addMinutes(30)
);

// With custom options (e.g., response content disposition)
$url = Storage::disk('s3')->temporaryUrl(
    'private/report.pdf',
    now()->addHour(),
    ['ResponseContentDisposition' => 'attachment; filename="report.pdf"']
);
```

**Syntax Rules:**

- `temporaryUrl()` is only available on disks that support it (S3, S3-compatible services). Calling it on a local disk throws an exception unless a custom builder is registered.
- The expiration is a `DateTimeInterface` instance (e.g., `now()->addHour()`).
- S3 presigned URLs have a maximum expiration of 7 days.
- The `buildTemporaryUrlsUsing()` macro should be called in the `boot()` method of a service provider.
- Custom temporary URLs typically use Laravel's `URL::temporarySignedRoute()` to generate a signed route that the application handles.

**Constraints and Limitations:**

- **`temporaryUrl()` does not work on local disks by default.** You must register a custom builder using `buildTemporaryUrlsUsing()`.
- **S3 presigned URLs are not revocable** before their expiration. Once generated, the URL is valid until it expires, even if the user's permissions are revoked.
- **The URL can be shared** with anyone during its validity period. Use short expiration times for sensitive files.
- **Custom temporary URL builders require a route** that handles the signed request and serves the file, which means the application server still proxies the file.
- **S3 buckets with public access blocked** require the presigned URL to grant `GetObject` permission, which it does.

### Annotated Code Examples

**Example 1: S3 Temporary URL for Private File Sharing**

```php
<?php
// File: app/Http/Controllers/ShareController.php

namespace App\Http\Controllers;

use App\Models\Document;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Storage;

class ShareController extends Controller
{
    public function generateShareLink(Request $request, Document $document)
    {
        // Step 1: Authorisation check
        if (!$request->user()->can('share', $document)) {
            abort(403, 'You do not have permission to share this document.');
        }

        // Step 2: Generate a temporary URL valid for 15 minutes
        $url = Storage::disk('s3')->temporaryUrl(
            $document->s3_path,
            now()->addMinutes(15),
            [
                'ResponseContentDisposition' => 'attachment; filename="' .
                    $document->original_name . '"',
            ]
        );

        // Step 3: Return the temporary URL
        return response()->json([
            'url'        => $url,
            'expires_at' => now()->addMinutes(15)->toIso8601String(),
        ]);
    }
}
```

**Expected Output:**

```json
{
    "url": "https://bucket.s3.amazonaws.com/documents/report.pdf?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=...&X-Amz-Expires=900&X-Amz-Signature=...",
    "expires_at": "2025-06-15T14:30:00+00:00"
}
```

**Why This Output Occurs:** The `temporaryUrl()` method uses the AWS SDK to generate a presigned URL. The URL includes the access key ID, expiration timestamp, and a cryptographic signature computed from the request details and the secret key. The `ResponseContentDisposition` option adds a query parameter that instructs S3 to set the `Content-Disposition` header in the response, forcing a download with the specified filename. The URL is valid for 15 minutes; after expiration, S3 returns an `AccessDenied` error.

---

**Example 2: Custom Temporary URLs for Local Disks**

```php
<?php
// File: app/Providers/AppServiceProvider.php

namespace App\Providers;

use Illuminate\Support\Facades\Storage;
use Illuminate\Support\Facades\URL;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        // Register a custom temporary URL builder for the local disk
        Storage::disk('local')->buildTemporaryUrlsUsing(
            function (string $path, \DateTimeInterface $expiration, array $options) {
                return URL::temporarySignedRoute(
                    'files.temporary',
                    $expiration,
                    ['path' => $path]
                );
            }
        );
    }
}
```

```php
// File: routes/web.php

use App\Http\Controllers\TemporaryFileController;

Route::get('/files/temporary/{path}', [TemporaryFileController::class, 'serve'])
    ->name('files.temporary')
    ->middleware('signed');
```

```php
// File: app/Http/Controllers/TemporaryFileController.php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Storage;

class TemporaryFileController extends Controller
{
    public function serve(Request $request, string $path)
    {
        // The 'signed' middleware verifies the signature and expiration
        if (!Storage::disk('local')->exists($path)) {
            abort(404);
        }

        return Storage::disk('local')->download($path);
    }
}
```

**Expected Output:**

- `Storage::disk('local')->temporaryUrl('documents/report.pdf', now()->addMinutes(15))` returns a URL like `https://example.com/files/temporary/documents/report.pdf?expires=...&signature=...`.
- Accessing the URL within 15 minutes downloads the file.
- Accessing the URL after expiration returns a 403 error.

**Why This Output Occurs:** The `buildTemporaryUrlsUsing()` macro registers a custom builder that generates a signed route URL using `URL::temporarySignedRoute()`. The signature is computed from the route parameters and the application key, and the expiration timestamp is included as a query parameter. The `signed` middleware verifies the signature and expiration before allowing the request to proceed. This pattern effectively adds temporary URL support to local disks, though the file is still served through the application server.

### Real-World Cases

- **Video streaming platforms:** Private videos are stored on S3 and served via temporary URLs generated for each viewing session, allowing direct streaming from S3 without proxying through the application server.
- **Healthcare portals:** Medical records are shared with external specialists via temporary URLs that expire after 1 hour, ensuring that access is time-limited and auditable.
- **E-commerce digital downloads:** After purchase, customers receive a temporary URL to download their digital product; the URL expires after 24 hours to prevent sharing.
- **Enterprise file sharing:** Employees generate temporary share links for large files stored on S3, with expiration times ranging from 15 minutes to 7 days depending on the sensitivity of the content.

---

## 7. Stream Management

### Definitions

**Core Definition:** Stream management is the practice of reading and writing file data in small, sequential chunks (streams) rather than loading entire files into memory, enabling efficient handling of large files with minimal memory usage.

**Technical Definition:** Laravel's `readStream()` method returns a PHP resource (a stream) that can be used to read a file's contents incrementally. The `writeStream()` method accepts a PHP resource and writes its contents to a storage disk in chunks. These methods are wrappers around Flysystem's `readStream()` and `writeStream()` operations. The `resource()` method on Laravel's HTTP Client response converts the response body into a stream resource that can be passed directly to `Storage::put()` or `writeStream()`. Stream-based operations are essential for transferring large files between disks, processing large uploads, and serving large files without exhausting PHP's memory limit.

**Beginner-Friendly Explanation:** Normally, when you read a file in PHP, the entire file is loaded into memory. For a 2 GB video, that's a problem — your server might not have enough RAM. Streams solve this by reading and writing the file in small pieces (typically 8 KB at a time). You never hold the whole file in memory, just one small chunk at a time. This means you can transfer files of any size without worrying about memory limits.

### Purposes

- To transfer large files between storage disks without loading them into memory.
- To read and process large files incrementally (e.g., line-by-line parsing, data migration).
- To write HTTP response bodies directly to storage without buffering the entire response.
- To serve large files to clients using streamed responses.
- To copy files between different storage systems (local to S3, S3 to local, S3 to FTP) efficiently.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// readStream() — get a read resource for a file
Storage::readStream(string $path): resource|null

// writeStream() — write a resource to a file
Storage::writeStream(string $path, resource $resource, array $options = []): bool

// put() with a resource — write a stream directly
Storage::put(string $path, resource $resource): bool

// HTTP Client resource() — get a stream from an HTTP response
$response = Http::get($url);
$stream = $response->resource();
Storage::put('file.pdf', $stream);
```

**Component Breakdown:**

- `readStream($path)` — Returns a PHP resource for reading the file at the given path, or `null` if the file does not exist.
- `writeStream($path, $resource, $options)` — Writes the contents of the resource to the file at the given path.
- `put($path, $resource)` — Equivalent to `writeStream()` when a resource is passed as the contents.
- `$response->resource()` — Converts the HTTP response body into a PHP stream resource.

```php
// Read a file as a stream
$stream = Storage::readStream('large-file.zip');
if ($stream) {
    while (!feof($stream)) {
        $chunk = fread($stream, 8192);
        // Process the chunk
    }
    fclose($stream);
}

// Write a stream to storage
$source = fopen('/path/to/large-file.zip', 'rb');
Storage::writeStream('backups/large-file.zip', $source);
fclose($source);

// Cross-disk copy using streams (local to S3)
Storage::disk('s3')->writeStream(
    'backups/file.zip',
    Storage::disk('local')->readStream('backups/file.zip')
);

// Stream an HTTP response to storage
$response = Http::get('https://example.com/large-file.zip');
Storage::put('downloads/large-file.zip', $response->resource());
```

**Syntax Rules:**

- `readStream()` returns `null` if the file does not exist. Always check for `null` before using the resource.
- `writeStream()` returns `true` on success and `false` on failure.
- The `put()` method accepts a resource as its second argument, internally calling `writeStream()`.
- Streams must be explicitly closed with `fclose()` after use to free the underlying resource.
- When using `readStream()` on a local disk, the resource is a file handle. On cloud disks, it is a stream wrapper that reads from the remote source.

**Constraints and Limitations:**

- **Stream resources are not serializable.** They cannot be stored in sessions, queues, or caches.
- **`readStream()` may return `null` for non-existent files** on some drivers, while others may throw an exception. Always check for `null` and handle exceptions.
- **Stream-based transfers do not verify integrity.** There is no built-in checksum validation after a stream transfer. Consider computing and comparing checksums for critical files.
- **Concurrent reads on the same stream** can corrupt the data. Each `readStream()` call returns a new resource, but reading from the same resource from multiple processes is unsafe.

### Annotated Code Examples

**Example 1: Memory-Efficient File Transfer Between Disks**

```php
<?php
// File: app/Console/Commands/TransferFiles.php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use Illuminate\Support\Facades\Storage;

class TransferFiles extends Command
{
    protected $signature = 'files:transfer {from} {to} {directory}';
    protected $description = 'Transfer files between disks using streams';

    public function handle(): int
    {
        $fromDisk = Storage::disk($this->argument('from'));
        $toDisk   = Storage::disk($this->argument('to'));
        $directory = $this->argument('directory');

        // Step 1: Get all files from the source directory
        $files = $fromDisk->allFiles($directory);

        $this->info("Transferring " . count($files) . " files from " .
                     $this->argument('from') . " to " . $this->argument('to'));

        foreach ($files as $file) {
            // Step 2: Skip if already exists on destination
            if ($toDisk->exists($file)) {
                $this->line("Skipped (exists): {$file}");
                continue;
            }

            // Step 3: Open a read stream on the source
            $stream = $fromDisk->readStream($file);

            if ($stream === null) {
                $this->error("Failed to read: {$file}");
                continue;
            }

            // Step 4: Write the stream to the destination
            $success = $toDisk->writeStream($file, $stream);

            // Step 5: Close the stream
            fclose($stream);

            if ($success) {
                $this->info("Transferred: {$file}");
            } else {
                $this->error("Failed to write: {$file}");
            }
        }

        return Command::SUCCESS;
    }
}
```

**Expected Output:**

```
Transferring 25 files from local to s3
Transferred: backups/db-2025-01.sql
Transferred: backups/db-2025-02.sql
Skipped (exists): backups/db-2025-03.sql
...
```

**Why This Output Occurs:** The command reads each file from the source disk using `readStream()`, which returns a PHP resource. The resource is passed to `writeStream()` on the destination disk, which reads the stream in chunks and writes them to the destination. The `fclose()` call releases the resource. Memory usage remains constant regardless of file size because only one chunk is held in memory at a time.

---

**Example 2: Streaming HTTP Response Directly to Storage**

```php
<?php
// File: app/Services/RemoteFileService.php

namespace App\Services;

use Illuminate\Support\Facades\Http;
use Illuminate\Support\Facades\Storage;

class RemoteFileService
{
    /**
     * Download a remote file and store it without buffering the entire response.
     */
    public function downloadAndStore(string $url, string $destinationPath): bool
    {
        // Step 1: Make the HTTP request
        $response = Http::timeout(120)->get($url);

        if (!$response->successful()) {
            throw new \RuntimeException("HTTP {$response->status()}: {$url}");
        }

        // Step 2: Stream the response body directly to storage
        // resource() converts the response into a PHP stream resource
        // put() internally calls writeStream() when given a resource
        $success = Storage::put($destinationPath, $response->resource());

        if (!$success) {
            throw new \RuntimeException("Failed to store: {$destinationPath}");
        }

        return true;
    }
}
```

**Expected Output:**

```php
// Returns true on success
$service->downloadAndStore('https://example.com/large-dataset.zip', 'datasets/large-dataset.zip');
// true
```

**Why This Output Occurs:** The `resource()` method converts the HTTP response body into a PHP stream resource without loading the entire response into memory. `Storage::put()` detects the resource and uses Flysystem's `writeStream()` operation to write the stream to disk in chunks. This pattern is ideal for downloading large files from external APIs and storing them directly in cloud storage.

### Real-World Cases

- **Database backup migration:** A command reads a database dump as a stream from the local disk and writes it to S3 using `readStream()` and `writeStream()`, handling multi-gigabyte files without memory issues.
- **Video processing pipelines:** Raw video files are streamed from S3 to a processing service, transcoded, and the output is streamed back to S3 — all without loading the video into memory.
- **Data ingestion from external APIs:** Large datasets are downloaded from third-party APIs using `Http::get()->resource()` and streamed directly to storage, avoiding temporary files and memory exhaustion.
- **Log aggregation:** Log files are read line-by-line using `readStream()` and `fgets()`, processed, and aggregated into compressed archives written to cloud storage using `writeStream()`.

---

## References

- Laravel File Storage Documentation (Master) — https://laravel.com/framework/docs/master/filesystem 
- Laravel File Storage Documentation (Laravel 10.x) — https://laravel.com/framework/docs/10.x/filesystem 
- Laravel `Filesystem` Contract API — https://api.laravel.com/docs/12.x/Illuminate/Contracts/Filesystem/Filesystem.html 
- Laravel `FilesystemAdapter` API — https://api.laravel.com/docs/12.x/Illuminate/Filesystem/FilesystemAdapter.html 
- Laravel `Cloud` Contract API — https://api.laravel.com/docs/11.x/Illuminate/Contracts/Filesystem/Cloud.html 
- Laravel `Storage::download` (Laravel News) — https://laravel-news.com/storagedownload 
- Simplified Stream Response Handling in Laravel (Laravel News) — https://laravel-news.com/stream-response-handling 
- Copy File or All Files from a Folder (Laravel Daily) — https://laraveldaily.com/post/laravel-copy-file-or-all-files-from-folder 
- Laravel File Visibility (GitHub Documentation) — https://github.com/laravel/docs/blob/10.x/filesystem.md 
- Laravel Temporary URLs (Laracasts Discussion) — https://laracasts.com/discuss/channels/general-discussion/view-pdf-stored-on-s3-in-new-tab 
- League Flysystem — https://github.com/thephpleague/flysystem 
- Flysystem AWS S3 V3 Adapter — https://flysystem.thephpleague.com/docs/adapter/aws-s3-v3/ 
- Laravel `Storage::temporaryUrl` (Community Reference) — https://github.com/laravel/docs/blob/10.x/filesystem.md#temporary-urls 
- Laravel `buildTemporaryUrlsUsing` (Stack Overflow) — https://stackoverflow.com/questions/61812345/how-to-create-temporary-url-for-custom-storage-in-laravel 
- Laravel Storage Migrator Package — https://github.com/diego-drese/laravel-storage-migrator 
- Laravel Sharded Storage Package — https://packagist.org/packages/stormkix/laravel-sharded-storage