# Laravel File Storage — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Laravel File Storage is a unified filesystem abstraction layer that provides a consistent API for interacting with local disks, network-attached storage, and cloud object storage services within Laravel applications.

**Technical Definition:** Laravel File Storage is an implementation of the Flysystem PHP package (by Frank de Jonge) integrated into the Laravel framework. It provides a `Storage` facade and a `FilesystemAdapter` class that abstract filesystem operations — reading, writing, deleting, listing, and URL generation — behind a driver-based architecture configured in `config/filesystems.php`. Each "disk" represents a named storage location bound to a specific Flysystem adapter (local, SFTP, S3, etc.).

**Beginner-Friendly Explanation:** Imagine your application needs to save files — user avatars, invoices, log archives, or uploaded videos. Instead of writing different code for your laptop's hard drive, a remote server, and Amazon's cloud, Laravel gives you one set of commands that works everywhere. You just tell Laravel *where* to put things (a "disk"), and the same `Storage::put()`, `Storage::get()`, and `Storage::delete()` calls work the same way regardless of the underlying storage.

### Key Characteristics

- **Driver-based abstraction:** The same API works across local, SFTP, S3, and custom backends.
- **Configuration-driven:** All disks are defined in `config/filesystems.php` and credentials are typically injected via `.env` variables.
- **Flysystem foundation:** Built on `league/flysystem`, a mature, standards-compliant PHP filesystem abstraction.
- **Multiple disks:** An application can define unlimited disks, including multiple disks sharing the same driver but different roots or credentials.
- **Environment parity:** The same code runs in local development and production; only the `.env` configuration changes.
- **Testing support:** The `Storage::fake()` method allows filesystem operations to be tested without touching real storage.

### Prerequisites

- PHP 8.1 or higher (Laravel 10+; Laravel 11 requires PHP 8.2+).
- Composer dependency manager installed.
- A Laravel application (any recent version; code examples target Laravel 10/11 syntax).
- Basic understanding of PHP namespaces and service providers.
- For cloud drivers: valid credentials for the respective cloud provider.
- For local storage: appropriate filesystem permissions on the server.

### Related Programming Areas

- **Flysystem** — The underlying PHP package providing the adapter ecosystem.
- **Service Providers** — Used to register custom drivers via `Storage::extend()`.
- **Environment Configuration** — `.env` files and `config/` files control disk selection and credentials.
- **Artisan Console** — Commands like `storage:link` and `storage:unlink` manage symbolic links.
- **Blade Templating** — The `asset()` helper generates public URLs for stored files.
- **HTTP File Uploads** — The `store()` method on `UploadedFile` instances integrates directly with Storage disks.

### Core Concepts / Features

1. Local Storage (the isolated `storage/app` directory and server file permissions)
2. Public Storage (configuring `storage:link` to map public-facing assets)
3. Cloud Storage (integrating Flysystem drivers for AWS S3, Google Cloud Storage, DigitalOcean Spaces)
4. Storage Disks (defining custom disks and shifting default drivers via `config/filesystems.php`)
5. Custom Flysystem Drivers (extending the storage system with third-party or custom adapters)

---

## 1. Local Storage

### Definitions

**Core Definition:** Local storage refers to file operations performed on the filesystem of the server running the Laravel application, rooted at the `storage/app` directory (or a subdirectory thereof).

**Technical Definition:** The `local` driver in Laravel uses `League\Flysystem\Local\LocalFilesystemAdapter` to interact with files stored on the host machine's filesystem. All operations are relative to the `root` path defined in the disk configuration, which defaults to `storage_path('app/private')` in Laravel 10+ (or `storage_path('app')` in older versions). Visibility (permissions) can be configured per-file and per-directory.

**Beginner-Friendly Explanation:** "Local" means the files live on the same machine that runs your Laravel app — the server's own hard drive. They are tucked away inside the `storage/app` folder, which is *not* directly accessible from the web. This is the safest place for private files like logs, backups, or user data that should never be publicly exposed.

### Purposes

- To store application files that should not be publicly accessible from the web.
- To isolate file operations within the `storage` directory, keeping them separate from application code.
- To control file and directory permissions explicitly through configuration.
- To provide a consistent API that mirrors the behaviour of cloud storage during local development.
- To serve as the default backing store for the `local` disk used by Laravel's session, cache, and logging systems.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// config/filesystems.php
'local' => [
    'driver' => 'local',
    'root' => storage_path('app/private'),
    'permissions' => [
        'file' => [
            'public' => 0644,
            'private' => 0600,
        ],
        'dir' => [
            'public' => 0755,
            'private' => 0700,
        ],
    ],
],
```

**Component Breakdown:**

- `'driver' => 'local'` — Selects the local filesystem driver.
- `'root' => storage_path('app/private')` — The base directory on disk. All paths passed to Storage methods are relative to this root.
- `'permissions'` — An optional array mapping visibility levels (`public`/`private`) to octal permission values.
- `'permissions.file.public' => 0644` — Files marked "public" receive `rw-r--r--` permissions.
- `'permissions.file.private' => 0600` — Files marked "private" receive `rw-------` permissions.
- `'permissions.dir.public' => 0755` — Directories marked "public" receive `rwxr-xr-x`.
- `'permissions.dir.private' => 0700` — Directories marked "private" receive `rwx------`.

```php
// Using the local disk
use Illuminate\Support\Facades\Storage;

Storage::disk('local')->put('example.txt', 'Contents');
$content = Storage::disk('local')->get('example.txt');
Storage::disk('local')->delete('example.txt');
```

**Syntax Rules:**

- All paths are relative to the disk's `root`; absolute paths are not permitted.
- If `permissions` is omitted, Laravel applies default permissions (0755 for directories, 0644 for files).
- The `local` disk is typically the default disk (configured via `FILESYSTEM_DISK` in `.env`), meaning `Storage::put()` without an explicit disk name uses it.

**Constraints and Limitations:**

- Local storage is **not** shared across multiple servers unless the `storage/app` directory is mounted on a shared network filesystem (e.g., NFS). In a load-balanced deployment, files written on one server will not be visible on another unless a shared volume is used.
- File paths are case-sensitive on Linux but case-insensitive on Windows and macOS, which can cause portability issues.
- The `local` driver does not support presigned URLs or temporary upload URLs — features exclusive to cloud drivers.

### Annotated Code Examples

**Example 1: Storing and Retrieving a Private File**

```php
<?php
// File: app/Http/Controllers/DocumentController.php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\Storage;
use Illuminate\Http\Request;

class DocumentController extends Controller
{
    public function store(Request $request)
    {
        // Step 1: Validate the uploaded file
        $request->validate([
            'document' => 'required|file|mimes:pdf|max:10240', // max 10MB
        ]);

        // Step 2: Store the file on the 'local' disk
        // The store() method generates a unique filename and returns the path
        // File will be placed at: storage/app/private/documents/<random>.pdf
        $path = $request->file('document')->store('documents', 'local');

        // Step 3: Persist the path in the database for later retrieval
        // (Assuming a Document model with a 'path' column)
        $document = \App\Models\Document::create([
            'original_name' => $request->file('document')->getClientOriginalName(),
            'path'          => $path,
        ]);

        return response()->json([
            'message' => 'Document stored successfully.',
            'path'    => $path,
        ]);
    }

    public function download($id)
    {
        // Step 4: Retrieve the document record
        $document = \App\Models\Document::findOrFail($id);

        // Step 5: Verify the file exists on the local disk
        if (!Storage::disk('local')->exists($document->path)) {
            abort(404, 'File not found on disk.');
        }

        // Step 6: Return a download response
        return Storage::disk('local')->download($document->path, $document->original_name);
    }
}
```

**Step-by-Step Setup:**

1. Create a `documents` table with columns `id`, `original_name`, `path`, `timestamps`.
2. Define the `Document` Eloquent model.
3. Register the routes: `POST /documents` → `DocumentController@store`; `GET /documents/{id}/download` → `DocumentController@download`.
4. Ensure the `storage/app/private/documents` directory is writable by the web server user (typically `www-data`).

**Expected Output:**

- On a successful upload, the JSON response returns `{"message": "Document stored successfully.", "path": "documents/AbC123xYz.pdf"}`.
- The file exists at `storage/app/private/documents/AbC123xYz.pdf` with permissions `0600` (private).
- On download, the browser receives the file with the original filename.

**Why This Output Occurs:** The `store()` method uses the disk's root (`storage/app/private`) as the base, appends the directory `documents`, generates a random filename with the correct extension, and writes the file with private visibility (0600) because the local disk's default visibility for the `store()` method is private. The `download()` method streams the file with the specified filename as the `Content-Disposition`.

---

**Example 2: Listing and Deleting Files on the Local Disk**

```php
<?php
// File: routes/console.php (or a custom Artisan command)

use Illuminate\Support\Facades\Storage;

// List all files in the 'backups' directory on the local disk
$backups = Storage::disk('local')->files('backups');

foreach ($backups as $file) {
    // $file is the relative path, e.g., 'backups/2025-01-15.sql'
    $size = Storage::disk('local')->size($file);        // bytes
    $modified = Storage::disk('local')->lastModified($file); // UNIX timestamp
    echo "File: {$file} | Size: {$size} bytes | Modified: " . date('Y-m-d H:i:s', $modified) . PHP_EOL;
}

// Delete files older than 30 days
$threshold = now()->subDays(30)->timestamp;

foreach ($backups as $file) {
    if (Storage::disk('local')->lastModified($file) < $threshold) {
        Storage::disk('local')->delete($file);
        echo "Deleted: {$file}" . PHP_EOL;
    }
}
```

**Expected Output:**

```
File: backups/2025-01-15.sql | Size: 2048000 bytes | Modified: 2025-01-15 03:00:00
File: backups/2025-01-14.sql | Size: 1984000 bytes | Modified: 2025-01-14 03:00:00
Deleted: backups/2024-12-01.sql
Deleted: backups/2024-12-02.sql
```

**Why This Output Occurs:** `Storage::disk('local')->files('backups')` returns an array of relative paths for all files directly inside the `backups` directory (non-recursive). The `size()` and `lastModified()` methods read the corresponding filesystem metadata. The deletion loop compares the modification timestamp against the 30-day threshold and removes matching files.

### Real-World Cases

- **Private document management systems:** Storing contracts, invoices, or medical records that must never be web-accessible. Files are served only through authenticated controller actions using `Storage::download()`.
- **Application logging and backups:** Laravel's default logging channel writes to `storage/logs/laravel.log` on the local disk. Backup packages like `spatie/laravel-backup` use local disks as intermediate storage before uploading to cloud destinations.
- **Session and cache storage:** The `file` session driver and `file` cache driver both use the local filesystem under `storage/framework/` — a subdirectory of the local disk's root.

---

## 2. Public Storage

### Definitions

**Core Definition:** Public storage refers to files stored on a disk that are intended to be directly accessible from the web via a URL, typically served through a symbolic link from `public/storage` to `storage/app/public`.

**Technical Definition:** Laravel's `public` disk is a pre-configured disk that uses the `local` driver with its root set to `storage_path('app/public')` and a `url` configuration pointing to `APP_URL/storage`. The `php artisan storage:link` command creates a symbolic link from `public/storage` to `storage/app/public`, making files within that directory accessible at `https://example.com/storage/filename.jpg`. Additional links can be defined in the `links` array of `config/filesystems.php`.

**Beginner-Friendly Explanation:** Some files — like profile pictures or product images — need to be visible to anyone visiting your website. But Laravel deliberately keeps the `storage` folder private. The solution is a "shortcut" (symbolic link) that makes a specific subfolder (`storage/app/public`) appear as if it lives inside the `public` folder, which is where your web server looks for files. The `storage:link` command creates that shortcut.

### Purposes

- To make user-uploaded assets (avatars, images, documents) publicly accessible via a clean URL.
- To keep publicly accessible files organised within the `storage` directory, separate from compiled assets.
- To share public files across deployments using zero-downtime deployment tools like Envoyer.
- To configure multiple symbolic links for different public-facing directories.
- To provide a consistent URL generation mechanism via the `asset()` helper.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// config/filesystems.php — Public disk definition
'public' => [
    'driver'     => 'local',
    'root'       => storage_path('app/public'),
    'url'        => env('APP_URL') . '/storage',
    'visibility' => 'public',
],

// Additional symbolic links
'links' => [
    public_path('storage') => storage_path('app/public'),
    public_path('images')  => storage_path('app/images'),
],
```

**Component Breakdown:**

- `'driver' => 'local'` — The public disk uses the local driver by default; it can also use `s3` for cloud-hosted public files.
- `'root' => storage_path('app/public')` — The base directory for files on this disk.
- `'url' => env('APP_URL') . '/storage'` — The base URL prepended to file paths when generating public URLs.
- `'visibility' => 'public'` — Files stored on this disk are marked as publicly readable (0644 on local).
- `'links'` — An array of source-to-target mappings. Each mapping creates a symbolic link when `storage:link` is executed.

```bash
# Artisan commands
php artisan storage:link      # Create symbolic links
php artisan storage:unlink    # Remove symbolic links
```

```php
// Generating a public URL
echo asset('storage/file.txt');  // https://example.com/storage/file.txt

// Storing a file on the public disk
Storage::disk('public')->put('avatars/user1.jpg', $contents);

// Storing an uploaded file on the public disk
$path = $request->file('avatar')->store('avatars', 'public');
```

**Syntax Rules:**

- The `storage:link` command must be executed after deployment (or after any change to the `links` array).
- The `asset()` helper prepends `APP_URL` and the path segment defined in the disk's `url` configuration.
- The symbolic link is created relative to the `public` directory; on shared hosting, the public directory may be at a different path (e.g., `public_html`), requiring a custom link mapping.

**Constraints and Limitations:**

- **Symbolic links may not work on Windows** without Developer Mode or administrator privileges; the `storage:link` command will fail silently or emit an error.
- On **shared hosting environments**, the `public` directory may not be named `public` (e.g., `public_html`), and the symlink may need to be created manually or via a custom `links` configuration.
- **Symbolic links are not included in deployment archives** created by tools that dereference symlinks; the `storage:link` command must be run as part of the deployment script.
- Files on the `public` disk are **not truly private** — anyone who knows the URL can access them. Do not store sensitive data on the public disk.

### Annotated Code Examples

**Example 1: Configuring and Using the Public Disk**

```php
<?php
// File: config/filesystems.php (excerpt)

return [
    'default' => env('FILESYSTEM_DISK', 'local'),

    'disks' => [
        'local' => [
            'driver' => 'local',
            'root'   => storage_path('app/private'),
            'serve'  => true,
        ],

        'public' => [
            'driver'     => 'local',
            'root'       => storage_path('app/public'),
            'url'        => env('APP_URL') . '/storage',
            'visibility' => 'public',
            'throw'      => false,
        ],
    ],

    'links' => [
        public_path('storage') => storage_path('app/public'),
    ],
];
```

```php
<?php
// File: app/Http/Controllers/AvatarController.php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Storage;

class AvatarController extends Controller
{
    public function upload(Request $request)
    {
        // Step 1: Validate the uploaded image
        $request->validate([
            'avatar' => 'required|image|mimes:jpeg,png,gif|max:2048', // 2MB max
        ]);

        // Step 2: Store the file on the 'public' disk
        // File will be saved to: storage/app/public/avatars/<random>.jpg
        $path = $request->file('avatar')->store('avatars', 'public');

        // Step 3: Generate the public URL for the stored file
        // Uses the 'url' config: APP_URL/storage/avatars/<random>.jpg
        $url = Storage::disk('public')->url($path);

        return response()->json([
            'message' => 'Avatar uploaded.',
            'path'    => $path,
            'url'     => $url,  // e.g., https://example.com/storage/avatars/xyz.jpg
        ]);
    }
}
```

**Step-by-Step Setup:**

1. Run `php artisan storage:link` to create the symbolic link from `public/storage` to `storage/app/public`.
2. Ensure the `public` directory is the web server's document root.
3. Ensure `storage/app/public` is writable by the web server.
4. (Optional) Add `storage/app/public` to `.gitignore` but keep the directory with a `.gitkeep` file.

**Expected Output:**

- The symbolic link exists: `public/storage → ../storage/app/public`.
- The uploaded file is stored at `storage/app/public/avatars/random.jpg` with permissions `0644`.
- The JSON response contains `"url": "https://example.com/storage/avatars/random.jpg"`.
- Navigating to that URL in a browser displays the image.

**Why This Output Occurs:** The `store()` method with the `'public'` disk writes the file to `storage/app/public/avatars/`. The `url()` method combines the disk's `url` configuration (`APP_URL/storage`) with the relative path. The symbolic link makes `storage/app/public/avatars/random.jpg` accessible at `public/storage/avatars/random.jpg`, which the web server serves directly.

---

**Example 2: Multiple Symbolic Links for Organised Assets**

```php
// config/filesystems.php — 'links' array
'links' => [
    public_path('storage') => storage_path('app/public'),
    public_path('media')   => storage_path('app/media'),
    public_path('reports') => storage_path('app/reports'),
],
```

```bash
php artisan storage:link
```

**Expected Output:**

```
The [public/storage] link has been connected to [storage/app/public].
The [public/media] link has been connected to [storage/app/media].
The [public/reports] link has been connected to [storage/app/reports].
```

**Why This Output Occurs:** The `storage:link` command iterates over the `links` array, creating a symbolic link for each source-target pair. This allows different categories of public files to be organised into separate storage directories while remaining accessible from distinct URL paths.

### Real-World Cases

- **User avatar and profile image uploads:** Stored on the `public` disk, served directly by the web server via the symlink, and referenced in Blade templates using `asset('storage/avatars/' . $user->avatar)`.
- **E-commerce product images:** Multiple image sizes can be stored in subdirectories of the public disk, with URLs generated for frontend display.
- **Public document sharing:** PDFs or reports that are intended to be downloadable by anyone with the link can be placed on the public disk, avoiding the overhead of a controller action for each download.

---

## 3. Cloud Storage

### Definitions

**Core Definition:** Cloud storage in Laravel refers to the use of Flysystem drivers (S3, Google Cloud Storage, DigitalOcean Spaces, etc.) to store files on remote object storage services instead of the local filesystem.

**Technical Definition:** Laravel's `s3` driver uses `League\Flysystem\AwsS3V3\AwsS3V3Adapter` to communicate with any S3-compatible object storage API, including Amazon S3, DigitalOcean Spaces, Cloudflare R2, Vultr Object Storage, and MinIO. Google Cloud Storage is supported through community-maintained Flysystem adapters registered as custom drivers. All cloud drivers expose the same `Storage` facade API as the local driver.

**Beginner-Friendly Explanation:** Instead of saving files on your own server's hard drive, you save them on a giant, reliable cloud service like Amazon S3 or Google Cloud. This means your files are safe even if your server crashes, they can be served to users from data centres close to them, and you never have to worry about running out of disk space. Laravel makes the switch almost invisible — you change a few lines in a config file and your existing code keeps working.

### Purposes

- To store files durably on remote object storage that survives server failures.
- To scale file storage independently of application server disk capacity.
- To serve files globally through CDN integration and region-specific endpoints.
- To reduce application server load by offloading static asset serving to cloud infrastructure.
- To enable presigned URLs for temporary, secure access to private files without proxying through the application server.

### Syntax Rules and Structure

#### Complete General Syntax (S3 / S3-Compatible)

```php
// Step 1: Install the S3 Flysystem adapter
// composer require league/flysystem-aws-s3-v3 "^3.0" --with-all-dependencies

// Step 2: Configure environment variables in .env
AWS_ACCESS_KEY_ID=your-access-key
AWS_SECRET_ACCESS_KEY=your-secret-key
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=your-bucket-name
AWS_URL=https://your-bucket.s3.amazonaws.com
AWS_ENDPOINT=https://your-custom-endpoint.com  # Optional, for S3-compatible services
AWS_USE_PATH_STYLE_ENDPOINT=false

// Step 3: Define the disk in config/filesystems.php
's3' => [
    'driver'                  => 's3',
    'key'                     => env('AWS_ACCESS_KEY_ID'),
    'secret'                  => env('AWS_SECRET_ACCESS_KEY'),
    'region'                  => env('AWS_DEFAULT_REGION'),
    'bucket'                  => env('AWS_BUCKET'),
    'url'                     => env('AWS_URL'),
    'endpoint'                => env('AWS_ENDPOINT'),
    'use_path_style_endpoint' => env('AWS_USE_PATH_STYLE_ENDPOINT', false),
    'throw'                   => false,
],
```

**Component Breakdown:**

- `'driver' => 's3'` — Selects the S3 Flysystem adapter.
- `'key'` / `'secret'` — Credentials for authenticating with the S3 API.
- `'region'` — The AWS region or equivalent for the storage provider.
- `'bucket'` — The name of the storage bucket (container).
- `'url'` — The base URL for generating public file URLs (optional).
- `'endpoint'` — Custom API endpoint for S3-compatible services (DigitalOcean Spaces, MinIO, etc.).
- `'use_path_style_endpoint'` — When `true`, uses `endpoint/bucket/key` format instead of `bucket.endpoint/key` (required for MinIO and some local S3 emulators).
- `'throw' => true` — Failures throw exceptions instead of returning `false`, which is essential for debugging.

#### Complete General Syntax (Google Cloud Storage)

```php
// Step 1: Install a community GCS adapter (e.g., Spatie's package)
// composer require spatie/laravel-google-cloud-storage

// Step 2: Define the disk in config/filesystems.php
'gcs' => [
    'driver' => 'gcs',
    'project_id' => env('GOOGLE_CLOUD_PROJECT_ID', 'your-project-id'),
    'key_file_path' => env('GOOGLE_CLOUD_KEY_FILE', base_path('google-key.json')),
    'bucket' => env('GOOGLE_CLOUD_STORAGE_BUCKET', 'your-bucket'),
    'path_prefix' => env('GOOGLE_CLOUD_STORAGE_PATH_PREFIX', null),
    'storage_api_uri' => env('GOOGLE_CLOUD_STORAGE_API_URI', null),
    'visibility' => 'private',
],
```

**Syntax Rules:**

- The `s3` driver is **not limited to AWS S3** — any service implementing the S3 API can be used by setting a custom `endpoint`.
- Google Cloud Storage has **no built-in driver** in Laravel; a third-party Flysystem adapter and service provider must be installed and registered.
- Credentials should **always** be stored in `.env` files, never committed to version control.
- The `throw` option should be set to `true` during development to surface errors that would otherwise be silently swallowed as `false` return values.

**Constraints and Limitations:**

- **Version-specific:** Laravel 9+ requires `league/flysystem-aws-s3-v3:^3.0`. Older Laravel versions (6.x, 7.x, 8.x) use `^1.0` or `^2.0` and have different adapter APIs.
- **S3-compatible services may have subtle incompatibilities** with the AWS SDK — particularly around multipart uploads, ACLs, and path-style vs. virtual-hosted-style URLs.
- **Google Cloud Storage adapters are community-maintained** and may not track Laravel's release cycle; compatibility with the latest Laravel version is not guaranteed.
- **Data transfer costs** apply when transferring files between the application server and cloud storage; large volumes of uploads/downloads can incur significant egress fees.
- **Presigned URLs have a maximum expiration time** (typically 7 days for S3); longer-lived access requires alternative strategies.

### Annotated Code Examples

**Example 1: Uploading to Amazon S3 with Presigned URLs**

```php
<?php
// File: app/Http/Controllers/MediaController.php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Storage;
use Illuminate\Support\Str;

class MediaController extends Controller
{
    /**
     * Upload a private file to S3 and return a temporary presigned URL.
     */
    public function upload(Request $request)
    {
        // Step 1: Validate the uploaded file
        $request->validate([
            'media' => 'required|file|mimes:jpg,png,mp4|max:51200', // 50MB max
        ]);

        // Step 2: Generate a unique storage path
        $extension = $request->file('media')->getClientOriginalExtension();
        $filename  = Str::uuid() . '.' . $extension;
        $directory = 'private-media/' . date('Y/m');

        // Step 3: Store on the S3 disk
        // The file is uploaded directly to the S3 bucket via the AWS SDK
        $path = $request->file('media')->storeAs($directory, $filename, 's3');

        // Step 4: Generate a temporary presigned URL (valid for 1 hour)
        // This URL grants time-limited access without making the file public
        $temporaryUrl = Storage::disk('s3')->temporaryUrl(
            $path,
            now()->addHour()
        );

        return response()->json([
            'path'          => $path,
            'temporary_url' => $temporaryUrl,
            'expires_at'    => now()->addHour()->toIso8601String(),
        ]);
    }

    /**
     * Permanently delete a file from S3.
     */
    public function destroy(Request $request)
    {
        $request->validate(['path' => 'required|string']);

        // Step 5: Check existence before deleting
        if (Storage::disk('s3')->exists($request->path)) {
            Storage::disk('s3')->delete($request->path);
            return response()->json(['message' => 'File deleted from S3.']);
        }

        return response()->json(['message' => 'File not found.'], 404);
    }
}
```

**Step-by-Step Setup:**

1. Create an S3 bucket in the AWS console (or equivalent for S3-compatible services).
2. Create an IAM user with programmatic access and attach a policy granting `s3:PutObject`, `s3:GetObject`, `s3:DeleteObject`, and `s3:ListBucket` on the bucket.
3. Add the credentials to `.env` as `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_DEFAULT_REGION`, and `AWS_BUCKET`.
4. Run `composer require league/flysystem-aws-s3-v3 "^3.0" --with-all-dependencies`.
5. Ensure the `s3` disk is defined in `config/filesystems.php` (it is included by default in Laravel 9+).
6. Register the routes: `POST /media` → `MediaController@upload`; `DELETE /media` → `MediaController@destroy`.

**Expected Output:**

- The JSON response: `{"path": "private-media/2025/06/uuid.jpg", "temporary_url": "https://bucket.s3.amazonaws.com/private-media/...?X-Amz-Signature=...", "expires_at": "2025-06-15T14:30:00+00:00"}`.
- The file is stored in the S3 bucket under the specified path.
- The presigned URL is valid for one hour; after expiry, accessing it returns an `AccessDenied` error.
- The file is **not** publicly accessible without the presigned URL.

**Why This Output Occurs:** The `storeAs()` method uploads the file to S3 using the AWS SDK via the Flysystem adapter. The `temporaryUrl()` method calls the S3 `GetObject` operation with a pre-signed query string that includes the access key, expiration, and a signature. The signature is computed using the secret key, so only someone with the secret can generate valid URLs. After expiration, S3 rejects the request because the signature's timestamp is outside the validity window.

---

**Example 2: Configuring DigitalOcean Spaces as an S3-Compatible Disk**

```php
// config/filesystems.php
'spaces' => [
    'driver'                  => 's3',
    'key'                     => env('DO_SPACES_KEY'),
    'secret'                  => env('DO_SPACES_SECRET'),
    'endpoint'                => env('DO_SPACES_ENDPOINT'), // e.g., https://fra1.digitaloceanspaces.com
    'region'                  => env('DO_SPACES_REGION'),   // e.g., fra1
    'bucket'                  => env('DO_SPACES_BUCKET'),
    'url'                     => env('DO_SPACES_URL'),      // e.g., https://bucket.fra1.digitaloceanspaces.com
    'use_path_style_endpoint' => false,
    'throw'                   => true,
],
```

```php
// Usage — identical API to any other disk
Storage::disk('spaces')->put('uploads/photo.jpg', $imageContents);
$url = Storage::disk('spaces')->url('uploads/photo.jpg');
```

**Expected Output:**

- The file is uploaded to the specified DigitalOcean Spaces bucket.
- The `url()` method returns the Spaces CDN-compatible URL (if configured) or the direct Spaces URL.

**Why This Output Occurs:** DigitalOcean Spaces implements the S3 API, so the same `league/flysystem-aws-s3-v3` adapter works without modification. The `endpoint` configuration overrides the default AWS endpoint, directing requests to DigitalOcean's infrastructure. The `use_path_style_endpoint` remains `false` because Spaces supports virtual-hosted-style URLs.

### Real-World Cases

- **Media-heavy applications (video streaming, image galleries):** Storing original and processed media on S3, with presigned URLs for private content and public URLs for CDN-served content.
- **Multi-region deployments:** Using S3 Cross-Region Replication or Google Cloud Storage's multi-region buckets to serve files from locations close to users.
- **Backup and archival systems:** Storing database dumps and application backups in cloud object storage with lifecycle policies that transition older data to cheaper storage classes (e.g., S3 Glacier).
- **SaaS platforms with tenant isolation:** Each tenant can have a dedicated S3 prefix or bucket, with IAM policies restricting access to their own files.

---

## 4. Storage Disks

### Definitions

**Core Definition:** A storage disk is a named configuration entry in `config/filesystems.php` that binds a driver, a root location, and driver-specific options into a single, reusable storage endpoint.

**Technical Definition:** In Laravel's Filesystem abstraction, a "disk" is an array key within the `disks` configuration array. Each disk is resolved by the `FilesystemManager` into an instance of `Illuminate\Filesystem\FilesystemAdapter`, which wraps a `League\Flysystem\Filesystem` instance configured with the appropriate adapter. The `default` key in `config/filesystems.php` (or the `FILESYSTEM_DISK` environment variable) determines which disk is used when no disk name is specified.

**Beginner-Friendly Explanation:** Think of a disk as a "storage profile" — a named set of instructions that tells Laravel where files should go and how to get there. You might have a `local` profile for private files, a `public` profile for web-accessible images, and an `s3` profile for cloud backups. When you say `Storage::disk('s3')->put(...)`, Laravel looks up the `s3` profile and uses those settings. You can create as many profiles as you need, each with its own driver and location.

### Purposes

- To define multiple storage destinations within a single application, each with different credentials or drivers.
- To switch the default storage backend between environments (local in development, S3 in production) without changing application code.
- To isolate different categories of files (e.g., user uploads vs. system backups) into separate storage locations.
- To provide a centralised configuration point for storage credentials and options.
- To enable disk-specific behaviour, such as public visibility for one disk and private visibility for another.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// config/filesystems.php

return [
    /*
    |--------------------------------------------------------------------------
    | Default Filesystem Disk
    |--------------------------------------------------------------------------
    | Here you may specify the default filesystem disk that should be used
    | by the framework. The "local" disk, as well as a variety of cloud
    | based disks are available to your application for file storage.
    */
    'default' => env('FILESYSTEM_DISK', 'local'),

    /*
    |--------------------------------------------------------------------------
    | Filesystem Disks
    |--------------------------------------------------------------------------
    | Below you may configure as many filesystem disks as necessary, and you
    | may even configure multiple disks for the same driver. Examples for
    | most supported storage drivers are configured here for reference.
    */
    'disks' => [
        'local' => [
            'driver' => 'local',
            'root'   => storage_path('app/private'),
            'serve'  => true,
            'throw'  => false,
        ],

        'public' => [
            'driver'     => 'local',
            'root'       => storage_path('app/public'),
            'url'        => env('APP_URL') . '/storage',
            'visibility' => 'public',
            'throw'      => false,
        ],

        's3' => [
            'driver'                  => 's3',
            'key'                     => env('AWS_ACCESS_KEY_ID'),
            'secret'                  => env('AWS_SECRET_ACCESS_KEY'),
            'region'                  => env('AWS_DEFAULT_REGION'),
            'bucket'                  => env('AWS_BUCKET'),
            'url'                     => env('AWS_URL'),
            'endpoint'                => env('AWS_ENDPOINT'),
            'use_path_style_endpoint' => env('AWS_USE_PATH_STYLE_ENDPOINT', false),
            'throw'                   => false,
        ],

        // Custom disk example: public folder on local filesystem
        'public_folder' => [
            'driver' => 'local',
            'root'   => public_path(),
            'url'    => env('APP_URL'),
            'throw'  => false,
        ],
    ],

    /*
    |--------------------------------------------------------------------------
    | Symbolic Links
    |--------------------------------------------------------------------------
    | Here you may configure the symbolic links that will be created when
    | the `storage:link` Artisan command is executed. The array keys
    | should be the location of the links, and the values their target.
    */
    'links' => [
        public_path('storage') => storage_path('app/public'),
    ],
];
```

**Component Breakdown:**

- `'default'` — The disk used when no explicit disk is specified. Controlled by `FILESYSTEM_DISK` in `.env`.
- `'disks'` — An associative array where each key is the disk name and each value is a configuration array.
- `'driver'` — The Flysystem driver identifier (`local`, `s3`, `sftp`, `ftp`, or a custom name).
- `'root'` — The base path or prefix for all operations on this disk.
- `'url'` — The base URL for public file access (used by the `url()` method).
- `'visibility'` — The default visibility (`public` or `private`) for files written to this disk.
- `'throw'` — Whether to throw exceptions on operation failure (`true`) or return `false` (`false`).
- `'links'` — Source-to-target mappings for symbolic link creation.

```php
// Using a specific disk
Storage::disk('public')->put('image.jpg', $data);
Storage::disk('s3')->put('backup.sql', $data);
Storage::disk('public_folder')->put('downloads/report.pdf', $data);

// Using the default disk (no disk name)
Storage::put('file.txt', $data); // Uses the 'default' disk
```

**Syntax Rules:**

- Disk names must be unique keys in the `disks` array.
- Multiple disks can share the same driver; for example, two `local` disks with different `root` paths.
- The `default` key is optional but recommended; if omitted, Laravel falls back to `local`.
- The `visibility` option is only meaningful for drivers that support per-file permissions (local, S3 with ACLs).
- The `url` option is required for the `url()` method to work on local disks; S3 disks derive the URL from the bucket and region unless overridden.

**Constraints and Limitations:**

- **Changing the `default` disk does not migrate existing files.** Files stored on the old default disk remain there and must be migrated manually.
- **Disk configuration is cached** by `php artisan config:cache`. After modifying `config/filesystems.php`, run `php artisan config:clear` and re-cache.
- **Environment variables in disk configuration are resolved at config-cache time.** If `.env` values change, the config cache must be regenerated.
- **Not all drivers support all methods.** For example, `temporaryUrl()` is only available on S3 and S3-compatible disks; calling it on a `local` disk throws an exception.

### Annotated Code Examples

**Example 1: Defining and Using Multiple Disks**

```php
<?php
// File: config/filesystems.php (custom disk configuration)

'disks' => [

    // Default private local disk
    'local' => [
        'driver' => 'local',
        'root'   => storage_path('app/private'),
        'throw'  => false,
    ],

    // Public web-accessible disk
    'public' => [
        'driver'     => 'local',
        'root'       => storage_path('app/public'),
        'url'        => env('APP_URL') . '/storage',
        'visibility' => 'public',
        'throw'      => false,
    ],

    // S3 disk for cloud backups
    's3_backups' => [
        'driver'   => 's3',
        'key'      => env('AWS_ACCESS_KEY_ID'),
        'secret'   => env('AWS_SECRET_ACCESS_KEY'),
        'region'   => env('AWS_DEFAULT_REGION'),
        'bucket'   => env('AWS_BACKUP_BUCKET'), // Different bucket
        'throw'    => true,
    ],

    // Local disk writing directly to the public directory
    'public_downloads' => [
        'driver' => 'local',
        'root'   => public_path('downloads'),
        'url'    => env('APP_URL') . '/downloads',
        'throw'  => false,
    ],
],
```

```php
<?php
// File: app/Services/ReportService.php

namespace App\Services;

use Illuminate\Support\Facades\Storage;

class ReportService
{
    /**
     * Generate a monthly report and store it in multiple locations.
     */
    public function generateMonthlyReport(int $month, int $year): array
    {
        $filename = "reports/{$year}-{$month}.pdf";
        $content  = $this->buildPdfContent($month, $year);

        // Store a private copy on the local disk
        Storage::disk('local')->put("private/{$filename}", $content);

        // Store a public copy accessible via the web
        Storage::disk('public_downloads')->put($filename, $content);

        // Store a backup copy on S3 for disaster recovery
        Storage::disk('s3_backups')->put("backups/{$filename}", $content);

        return [
            'private_path'  => "private/{$filename}",
            'public_url'    => Storage::disk('public_downloads')->url($filename),
            's3_backup_path' => "backups/{$filename}",
        ];
    }

    private function buildPdfContent(int $month, int $year): string
    {
        // Placeholder for actual PDF generation logic
        return "PDF content for {$year}-{$month}";
    }
}
```

**Step-by-Step Setup:**

1. Define all four disks in `config/filesystems.php` as shown.
2. Add `AWS_BACKUP_BUCKET` to `.env` with the name of a separate S3 bucket.
3. Ensure the `public/downloads` directory exists and is writable.
4. Run `php artisan config:clear` to reload the configuration.

**Expected Output:**

- The private copy exists at `storage/app/private/private/reports/2025-06.pdf`.
- The public copy exists at `public/downloads/reports/2025-06.pdf` and is accessible at `https://example.com/downloads/reports/2025-06.pdf`.
- The S3 backup exists in the `AWS_BACKUP_BUCKET` under the `backups/reports/2025-06.pdf` key.
- The method returns an array with the paths and public URL.

**Why This Output Occurs:** Each `Storage::disk()` call resolves the corresponding disk configuration. The `local` disk writes relative to `storage/app/private`. The `public_downloads` disk writes relative to the `public/downloads` directory (directly inside the web root, no symlink required). The `s3_backups` disk uploads to the specified S3 bucket. Because the disks use different drivers and roots, the same filename can coexist in all three locations without conflict.

---

**Example 2: Shifting the Default Disk via Environment Configuration**

```bash
# .env — Development environment
FILESYSTEM_DISK=local
```

```bash
# .env — Production environment
FILESYSTEM_DISK=s3
```

```php
// Application code — No changes required
Storage::put('uploads/file.jpg', $data);

// In development: writes to storage/app/private/uploads/file.jpg
// In production:  writes to the S3 bucket
```

**Expected Output:**

- In development, files are written to the local `storage/app/private` directory.
- In production, the same `Storage::put()` call writes to S3.
- No code changes are needed between environments.

**Why This Output Occurs:** The `default` configuration key reads `env('FILESYSTEM_DISK', 'local')`. When the environment variable is set, Laravel uses the corresponding disk. The `Storage` facade's methods without an explicit disk name resolve to the default disk. This pattern is the foundation of Laravel's environment parity philosophy: code remains identical across environments, and only configuration changes.

### Real-World Cases

- **Development-to-production parity:** Developers use the `local` disk on their machines while production uses `s3`, with identical application code.
- **Multi-cloud or hybrid storage:** An application might store user uploads on S3, temporary files on local disk, and audit logs on Google Cloud Storage — each as a separate disk.
- **Tenant-specific storage:** A SaaS application can define a disk per tenant, each with its own S3 bucket or prefix, and dynamically select the disk based on the authenticated user's tenant ID.
- **Read replicas and failover:** Multiple disks pointing to different S3 buckets in different regions can be used for failover or read distribution.

---

## 5. Custom Flysystem Drivers

### Definitions

**Core Definition:** A custom Flysystem driver is a user-defined storage driver that extends Laravel's Filesystem abstraction to support a storage backend not included in the framework's built-in drivers.

**Technical Definition:** Laravel's `FilesystemManager` provides an `extend()` method that allows developers to register custom driver creators. Each creator is a closure that receives the application container and the disk configuration array, and returns an instance of `Illuminate\Filesystem\FilesystemAdapter` (which wraps a `League\Flysystem\Filesystem` configured with a third-party adapter). Custom drivers are typically registered in the `boot()` method of a service provider.

**Beginner-Friendly Explanation:** Laravel comes with drivers for local storage, S3, and SFTP. But what if you want to store files on Dropbox, Google Drive, or a custom FTP server? Flysystem — the engine behind Laravel's storage — has adapters for dozens of other services. A "custom driver" is simply a small piece of code that tells Laravel how to use one of those extra adapters. You write a few lines in a service provider, and suddenly `Storage::disk('dropbox')` works just like any other disk.

### Purposes

- To integrate storage backends that are not supported by Laravel's built-in drivers.
- To use community-maintained Flysystem adapters (Dropbox, Google Drive, WebDAV, etc.) within Laravel's unified Storage API.
- To implement proprietary or in-house storage protocols behind the same facade.
- To wrap existing storage services with custom logic (e.g., encryption, compression) before they are exposed as a disk.
- To maintain a consistent developer experience across all storage backends.

### Syntax Rules and Structure

#### Complete General Syntax

```php
<?php
// File: app/Providers/AppServiceProvider.php

namespace App\Providers;

use Illuminate\Contracts\Foundation\Application;
use Illuminate\Filesystem\FilesystemAdapter;
use Illuminate\Support\Facades\Storage;
use Illuminate\Support\ServiceProvider;
use League\Flysystem\Filesystem;

class AppServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        Storage::extend('driver_name', function (Application $app, array $config) {
            // Step 1: Instantiate the third-party adapter
            $adapter = new \SomePackage\SomeAdapter(
                $config['option1'],
                $config['option2']
            );

            // Step 2: Wrap the adapter in a Flysystem instance
            $flysystem = new Filesystem($adapter, $config);

            // Step 3: Return a Laravel FilesystemAdapter
            return new FilesystemAdapter($flysystem, $adapter, $config);
        });
    }
}
```

**Component Breakdown:**

- `Storage::extend('driver_name', ...)` — Registers a custom driver under the name `driver_name`.
- The closure receives `$app` (the Laravel application container) and `$config` (the disk configuration array from `config/filesystems.php`).
- The third-party adapter is instantiated with values from `$config`.
- `new Filesystem($adapter, $config)` — Creates a Flysystem instance. The `$config` array is optional but passed through to enable Flysystem-level configuration.
- `new FilesystemAdapter($flysystem, $adapter, $config)` — Wraps the Flysystem instance in Laravel's adapter, which provides the `Storage` facade methods (`put`, `get`, `delete`, `url`, etc.).

```php
// config/filesystems.php — Disk definition for the custom driver
'custom_disk' => [
    'driver'   => 'driver_name',  // Must match the name registered in extend()
    'option1'  => env('CUSTOM_OPTION_1'),
    'option2'  => env('CUSTOM_OPTION_2'),
],
```

**Syntax Rules:**

- `Storage::extend()` **must be called in the `boot()` method** of a service provider, not `register()`. The `Storage` facade is not yet fully resolved during the registration phase.
- The closure **must return** an instance of `Illuminate\Filesystem\FilesystemAdapter` in Laravel 9+ (in older versions, it returned `League\Flysystem\Filesystem`).
- The disk's `driver` key in `config/filesystems.php` must match the name passed to `extend()`.
- The `$config` array contains all keys from the disk configuration, including the `driver` key itself.

**Constraints and Limitations:**

- **Flysystem v3 vs. v2 vs. v1:** The `FilesystemAdapter` constructor signature and the expected return type differ between Flysystem major versions. Laravel 9+ uses Flysystem v3. Community adapters must be compatible with the Flysystem version used by your Laravel installation.
- **Community adapters may be unmaintained.** Before using a third-party adapter, check its update history, Laravel compatibility, and test coverage.
- **Custom drivers registered via `extend()` are not automatically available in queued jobs** unless the service provider is loaded in the queue worker context (it is, by default, but custom service providers registered in `config/app.php` must be present).
- **Testing custom drivers requires `Storage::fake()`** or a mock adapter, as the real backend may not be available in CI environments.

### Annotated Code Examples

**Example 1: Registering a Dropbox Custom Driver**

```php
<?php
// Step 1: Install the community adapter
// composer require spatie/flysystem-dropbox

// Step 2: Create a service provider
// File: app/Providers/DropboxServiceProvider.php

namespace App\Providers;

use Illuminate\Contracts\Foundation\Application;
use Illuminate\Filesystem\FilesystemAdapter;
use Illuminate\Support\Facades\Storage;
use Illuminate\Support\ServiceProvider;
use League\Flysystem\Filesystem;
use Spatie\Dropbox\Client as DropboxClient;
use Spatie\FlysystemDropbox\DropboxAdapter;

class DropboxServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        Storage::extend('dropbox', function (Application $app, array $config) {
            // Step 3: Create the Dropbox API client using the authorization token
            $client = new DropboxClient(
                $config['authorization_token']
            );

            // Step 4: Create the Flysystem Dropbox adapter
            $adapter = new DropboxAdapter($client);

            // Step 5: Create the Flysystem instance
            $flysystem = new Filesystem($adapter, $config);

            // Step 6: Return Laravel's FilesystemAdapter
            return new FilesystemAdapter($flysystem, $adapter, $config);
        });
    }
}
```

```php
// Step 7: Register the service provider in config/app.php (Laravel 10 and below)
// For Laravel 11+, service providers are auto-discovered or registered in bootstrap/providers.php
'providers' => [
    // ...
    App\Providers\DropboxServiceProvider::class,
],
```

```php
// Step 8: Define the disk in config/filesystems.php
'dropbox' => [
    'driver'              => 'dropbox',
    'authorization_token' => env('DROPBOX_AUTHORIZATION_TOKEN'),
],
```

```php
// Step 9: Use the disk like any other
Storage::disk('dropbox')->put('documents/report.pdf', $pdfContent);
$url = Storage::disk('dropbox')->url('documents/report.pdf');
```

**Step-by-Step Setup:**

1. Run `composer require spatie/flysystem-dropbox`.
2. Create the `DropboxServiceProvider` as shown above.
3. Register the provider in `config/app.php` (Laravel ≤10) or `bootstrap/providers.php` (Laravel 11+).
4. Generate a Dropbox API access token with the `files.content.write` and `files.content.read` scopes from the Dropbox App Console.
5. Add `DROPBOX_AUTHORIZATION_TOKEN` to `.env`.
6. Add the `dropbox` disk to `config/filesystems.php`.
7. Run `php artisan config:clear`.

**Expected Output:**

- The file `documents/report.pdf` is uploaded to the Dropbox account associated with the access token.
- `Storage::disk('dropbox')->url()` returns a Dropbox shared link (note: Dropbox URLs are temporary and may require refreshing).
- All standard `Storage` methods (`put`, `get`, `delete`, `exists`, `files`) work on the `dropbox` disk.

**Why This Output Occurs:** The `Storage::extend()` call registers a creator closure for the `dropbox` driver name. When `Storage::disk('dropbox')` is first called, the `FilesystemManager` invokes the closure, which instantiates the Dropbox adapter, wraps it in a Flysystem instance, and returns a `FilesystemAdapter`. Subsequent calls to that disk reuse the resolved instance. The Dropbox adapter translates Flysystem operations into Dropbox API calls.

---

**Example 2: Building a Custom Adapter for a Proprietary Storage System**

```php
<?php
// File: app/Filesystem/InHouseStorageAdapter.php

namespace App\Filesystem;

use League\Flysystem\Adapter\AbstractAdapter; // Flysystem v1/v2 style
use League\Flysystem\Config;
use League\Flysystem\FileAttributes;
use League\Flysystem\FilesystemAdapter as FlysystemAdapterInterface;
use League\Flysystem\UnableToReadFile;
use League\Flysystem\UnableToWriteFile;

/**
 * A minimal Flysystem v3 adapter for a hypothetical in-house storage API.
 * In practice, this would use Guzzle or an SDK to communicate with the API.
 */
class InHouseStorageAdapter implements FlysystemAdapterInterface
{
    private string $apiEndpoint;
    private string $apiKey;

    public function __construct(string $apiEndpoint, string $apiKey)
    {
        $this->apiEndpoint = rtrim($apiEndpoint, '/');
        $this->apiKey      = $apiKey;
    }

    public function fileExists(string $path): bool
    {
        // Call the in-house API to check existence
        return $this->apiRequest('HEAD', $path)['status'] === 200;
    }

    public function read(string $path): string
    {
        $response = $this->apiRequest('GET', $path);
        if ($response['status'] !== 200) {
            throw UnableToReadFile::fromLocation($path);
        }
        return $response['body'];
    }

    public function write(string $path, string $contents, Config $config): void
    {
        $response = $this->apiRequest('PUT', $path, $contents);
        if ($response['status'] !== 201) {
            throw UnableToWriteFile::fromLocation($path);
        }
    }

    public function delete(string $path): void
    {
        $this->apiRequest('DELETE', $path);
    }

    // Other required methods (listContents, visibility, etc.) omitted for brevity.

    private function apiRequest(string $method, string $path, ?string $body = null): array
    {
        // Placeholder for actual HTTP client logic
        return ['status' => 200, 'body' => ''];
    }
}
```

```php
// File: app/Providers/InHouseStorageServiceProvider.php

namespace App\Providers;

use App\Filesystem\InHouseStorageAdapter;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Filesystem\FilesystemAdapter;
use Illuminate\Support\Facades\Storage;
use Illuminate\Support\ServiceProvider;
use League\Flysystem\Filesystem;

class InHouseStorageServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        Storage::extend('inhouse', function (Application $app, array $config) {
            $adapter = new InHouseStorageAdapter(
                $config['endpoint'],
                $config['api_key']
            );

            return new FilesystemAdapter(
                new Filesystem($adapter, $config),
                $adapter,
                $config
            );
        });
    }
}
```

```php
// config/filesystems.php
'inhouse' => [
    'driver'   => 'inhouse',
    'endpoint' => env('INHOUSE_STORAGE_ENDPOINT'),
    'api_key'  => env('INHOUSE_STORAGE_API_KEY'),
],
```

**Step-by-Step Setup:**

1. Implement the Flysystem adapter interface with all required methods (`fileExists`, `read`, `write`, `delete`, `listContents`, `visibility`, etc.).
2. Create a service provider that registers the `inhouse` driver via `Storage::extend()`.
3. Register the provider in `config/app.php` or `bootstrap/providers.php`.
4. Define the `inhouse` disk in `config/filesystems.php`.
5. Add the endpoint and API key to `.env`.

**Expected Output:**

- `Storage::disk('inhouse')->put('file.txt', 'Hello')` writes the content through the in-house API.
- `Storage::disk('inhouse')->get('file.txt')` retrieves the content.
- All other Storage facade methods work through the adapter.

**Why This Output Occurs:** The custom adapter implements the Flysystem v3 interface, translating abstract filesystem operations into API calls. Laravel's `FilesystemAdapter` delegates to the Flysystem instance, which in turn calls the custom adapter's methods. Because the adapter conforms to the Flysystem contract, Laravel's Storage facade can interact with it without any knowledge of the underlying API.

### Real-World Cases

- **Dropbox or Google Drive integration for user files:** Many applications allow users to connect their own cloud storage; a custom driver enables Laravel to write to the user's Dropbox or Drive account.
- **WebDAV or SMB/CIFS network shares:** Corporate environments often require files to be stored on network-attached storage accessible via WebDAV or SMB; community adapters exist for both.
- **Custom encryption-at-rest storage:** A custom driver can wrap another adapter and transparently encrypt/decrypt file contents using Laravel's `Crypt` facade before writing to or after reading from the underlying storage.
- **In-house object storage or on-premises S3 emulators:** MinIO, Ceph, and similar systems implement the S3 API but often require path-style endpoints; a custom driver can encapsulate the specific configuration and authentication requirements.

---

## References

- Laravel File Storage Documentation (Master) — https://laravel.com/framework/docs/master/filesystem 
- Laravel File Storage Documentation (Laravel 10.x) — https://laravel.com/framework/docs/10.x/filesystem 
- Laravel File Storage Documentation (Laravel 6.x) — https://laravel.com/framework/docs/6.x/filesystem 
- League Flysystem — https://github.com/thephpleague/flysystem 
- Flysystem AWS S3 V3 Adapter — https://flysystem.thephpleague.com/docs/adapter/aws-s3-v3/ 
- Laravel `storage:link` Artisan Command — https://laravel.com/docs/artisan#storage-link 
- Spatie Flysystem Dropbox — https://github.com/spatie/flysystem-dropbox 
- Spatie Laravel Google Cloud Storage — https://github.com/spatie/laravel-google-cloud-storage 
- DigitalOcean Spaces as S3-Compatible Storage — https://docs.digitalocean.com/products/spaces/how-to/configure-laravel/ 
- Laravel `FilesystemManager` API — https://api.laravel.com/docs/10.x/Illuminate/Filesystem/FilesystemManager.html 
- `League\Flysystem\FilesystemAdapter` Interface — https://flysystem.thephpleague.com/docs/api/filesystem-adapter/ 