# PHP Directory Operations — Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**  
PHP Directory Operations are the built-in functions and object-oriented classes used to create, read, navigate, inspect, and delete directories on the server's filesystem.

**Technical Definition**  
PHP's directory API is part of the filesystem extension and includes procedural functions (`opendir()`, `readdir()`, `closedir()`, `scandir()`, `glob()`, `mkdir()`, `rmdir()`, `chdir()`, `getcwd()`, `dir()`) and object-oriented abstractions (`DirectoryIterator`, `FilesystemIterator`, `RecursiveDirectoryIterator`, `RecursiveIteratorIterator`, `SplFileInfo`). It operates on directory paths subject to filesystem permissions, `open_basedir` restrictions, and the `umask` setting. `mkdir()` supports recursive creation with the `recursive` flag (PHP 5.0+), and `rmdir()` removes only empty directories. Recursive deletion requires manual iteration or a custom function. PHP 8.0.0 changed `opendir()` to throw `ValueError` for invalid paths; PHP 8.3.0 throws `ValueError` for invalid `glob()` flags.

**Beginner-Friendly Explanation**  
Just as you organize files on your computer into folders, PHP can create, list, open, and delete folders on the server. You can list all files in a directory, create a new folder, check if a folder exists, move around the filesystem, and even delete folders and everything inside them. PHP gives you both simple functions (like `scandir()` to list a directory) and powerful object-oriented iterators for walking through complex directory trees.

---

### Key Characteristics

- Directory operations are subject to filesystem permissions and `open_basedir` restrictions.
- `mkdir()` creates directories; use the `recursive` flag to create parent directories.
- `rmdir()` removes only empty directories; recursive deletion requires iteration.
- `scandir()` returns a sorted array of directory entries, including `.` and `..`.
- `glob()` matches paths using shell-style patterns (`*`, `?`, `[]`).
- `opendir()`/`readdir()`/`closedir()` provide low-level, memory-efficient iteration.
- `DirectoryIterator` and `RecursiveDirectoryIterator` provide object-oriented iteration.
- `chdir()` changes the current working directory; `getcwd()` returns it.
- Results are cached; use `clearstatcache()` when directories change during a request.
- `scandir()` and `glob()` sort results by default; `readdir()` does not guarantee order.
- Symbolic links are not followed by default in some functions; `RecursiveDirectoryIterator` requires `FOLLOW_SYMLINKS`.
- PHP 8.0.0: `opendir()` throws `ValueError` for invalid paths.
- PHP 8.3.0: `glob()` throws `ValueError` for invalid flags.

---

### Prerequisites

- Basic PHP syntax: variables, arrays, loops, functions.
- Basic filesystem concepts: paths, directories, permissions.
- Understanding of Unix permissions (for `mkdir()` and `chmod()`).
- A web server with PHP installed.
- Read/write permissions on the target directories.

---

### Related Programming Areas

- File handling (`fopen()`, `file_get_contents()`).
- Stream wrappers and filesystem iterators.
- File upload handling and storage directories.
- Recursive algorithms and tree traversal.
- Build tools and cache management.
- Security: path traversal, symlink attacks, TOCTOU.

---

## Core Concepts / Features

## 1. Reading Directories — `scandir()`, `glob()`, and `opendir()`/`readdir()`

### Definitions

**Core Definition**  
Reading a directory means retrieving the list of files and subdirectories it contains.

**Technical Definition**  
`scandir()` returns a sorted array of directory entries including `.` and `..`. `glob()` returns an array of paths matching a shell-style pattern. `opendir()` opens a directory handle; `readdir()` reads entries one at a time; `closedir()` closes the handle. `DirectoryIterator` provides an object-oriented interface. All functions return `false` on failure and emit `E_WARNING`.

**Beginner-Friendly Explanation**  
Reading a directory is like opening a folder and looking at what is inside. `scandir()` gives you a simple array of names. `glob()` lets you filter by pattern — for example, only `.jpg` files. `opendir()` and `readdir()` let you read one entry at a time, which is useful for very large directories.

### Purposes

- To list files in a directory for display or processing.
- To filter files by extension or pattern.
- To iterate over directory contents efficiently.
- To build file managers, galleries, and media libraries.
- To scan upload directories for cleanup.
- To discover configuration or template files.
- To implement batch processing of directory contents.

### Syntax Rules and Structure

**General Syntax: `scandir()`**
```php
$entries = scandir(
    string $directory,
    int $sorting_order = SCANDIR_SORT_ASCENDING,
    ?resource $context = null
): array|false;
```

**Component Breakdown**

- `$directory` — path to the directory.
- `$sorting_order` — `SCANDIR_SORT_ASCENDING` (default), `SCANDIR_SORT_DESCENDING`, or `SCANDIR_SORT_NONE`.
- `$context` — stream context.
- Returns an array of entries including `.` and `..`, or `false` on failure.

**General Syntax: `glob()`**
```php
$matches = glob(
    string $pattern,
    int $flags = 0
): array|false;
```

**Component Breakdown**

- `$pattern` — shell-style pattern (`*`, `?`, `[]`, `{}`).
- `$flags` — `GLOB_BRACE`, `GLOB_ONLYDIR`, `GLOB_MARK`, `GLOB_NOSORT`, `GLOB_NOCHECK`, `GLOB_NOESCAPE`, `GLOB_ERR`.
- Returns an array of matching paths, or `false` on failure.
- Returns an empty array if no matches.

**General Syntax: `opendir()` / `readdir()` / `closedir()`**
```php
$handle = opendir(string $directory, ?resource $context = null): resource|false;

while (($entry = readdir($handle)) !== false) {
    // process $entry
}

closedir($handle);
```

**Component Breakdown**

- `opendir()` — opens a directory handle.
- `readdir()` — returns the next entry, or `false` at end or on error.
- `closedir()` — closes the handle.
- `rewinddir()` — resets the pointer to the beginning.

**General Syntax: `DirectoryIterator`**
```php
foreach (new DirectoryIterator($directory) as $fileInfo) {
    if ($fileInfo->isDot()) continue;
    echo $fileInfo->getFilename();
}
```

**Component Breakdown**

- `DirectoryIterator` — iterates over directory entries.
- `isDot()` — skips `.` and `..`.
- `getFilename()` — returns the entry name.

**Syntax Rules**

- `scandir()` includes `.` and `..`; filter them out if not needed.
- `glob()` uses shell-style patterns; use `GLOB_BRACE` for `{a,b}` alternation.
- `glob()` returns an empty array if no matches (not `false`).
- `readdir()` returns the filename only, not the full path.
- `readdir()` does not guarantee sort order.
- Always call `closedir()` to free the handle.
- `DirectoryIterator` throws exceptions on invalid paths.
- Use `clearstatcache()` if the directory changed during the request.

**Constraints and Limitations**

- `scandir()` loads the entire directory into memory.
- `glob()` may not work on some systems with very large directories.
- `readdir()` order is filesystem-dependent.
- `opendir()` fails if the directory does not exist or is not readable.
- `DirectoryIterator` requires PHP 5.0+.
- Symbolic links are returned as entries; use `isLink()` to detect them.
- Hidden files (starting with `.`) are included on Unix; on Windows, use `GLOB_NOSORT` carefully.

### Annotated Code Examples and Expected Outputs

#### Example 1: Listing Files with `scandir()`

**File: `list_dir.php`**
```php
<?php
$directory = __DIR__ . '/uploads';

// Check if the directory exists.
if (!is_dir($directory)) {
    echo 'Directory not found.';
    exit;
}

// Get the entries.
$entries = scandir($directory);

if ($entries === false) {
    echo 'Failed to read directory.';
    exit;
}

// Filter out . and ..
$entries = array_filter($entries, fn($e) => $e !== '.' && $e !== '..');

foreach ($entries as $entry) {
    echo htmlspecialchars($entry, ENT_QUOTES, 'UTF-8') . '<br>';
}
?>
```

**Expected Output**
```
avatar.jpg<br>
document.pdf<br>
photo.png<br>
```

**Why This Output Occurs**

- `scandir()` returns all entries including `.` and `..`.
- `array_filter()` removes the dot entries.
- Each filename is escaped for HTML output.

#### Example 2: Filtering with `glob()`

**File: `glob_images.php`**
```php
<?php
// Find all JPEG and PNG files.
$images = glob(__DIR__ . '/uploads/*.{jpg,jpeg,png}', GLOB_BRACE);

if ($images === false) {
    echo 'Glob failed.';
    exit;
}

foreach ($images as $path) {
    echo htmlspecialchars(basename($path), ENT_QUOTES, 'UTF-8') . '<br>';
}
?>
```

**Expected Output**
```
avatar.jpg<br>
photo.png<br>
```

**Why This Output Occurs**

- `GLOB_BRACE` enables `{jpg,jpeg,png}` alternation.
- `glob()` returns matching paths.
- `basename()` extracts the filename.

#### Example 3: Low-Level Iteration with `opendir()`/`readdir()`

**File: `readdir_loop.php`**
```php
<?php
$handle = opendir(__DIR__ . '/uploads');

if ($handle === false) {
    echo 'Failed to open directory.';
    exit;
}

while (($entry = readdir($handle)) !== false) {
    if ($entry === '.' || $entry === '..') continue;
    echo htmlspecialchars($entry, ENT_QUOTES, 'UTF-8') . '<br>';
}

closedir($handle);
?>
```

**Expected Output**
```
avatar.jpg<br>
document.pdf<br>
photo.png<br>
```

**Why This Output Occurs**

- `opendir()` opens the directory.
- `readdir()` returns entries one at a time.
- Dot entries are skipped.
- `closedir()` releases the handle.

### Real-World Cases

- **File managers:** List directory contents with `scandir()`.
- **Image galleries:** Use `glob('*.{jpg,png}')` to find images.
- **Log rotation:** Scan log directories for old files.
- **Cache cleanup:** Iterate over cache directories.
- **Template discovery:** Find all `.php` templates in a directory.

---

## 2. Creating Directories — `mkdir()`

### Definitions

**Core Definition**  
`mkdir()` creates a new directory at the specified path.

**Technical Definition**  
`mkdir(string $directory, int $permissions = 0777, bool $recursive = false, ?resource $context = null): bool` creates a directory with the given permissions (modified by `umask`). With `$recursive = true`, parent directories are created as needed. Returns `true` on success, `false` on failure. Emits `E_WARNING` if the directory already exists or cannot be created.

**Beginner-Friendly Explanation**  
`mkdir()` creates a new folder. If you want to create nested folders in one call — like `a/b/c` — pass `true` as the third argument. The permissions argument sets who can read, write, or execute the folder.

### Purposes

- To create storage directories for uploads, caches, or logs.
- To set up directory structures programmatically.
- To create user-specific directories.
- To organize generated content.
- To prepare destinations before moving files.
- To support multi-tenant applications.
- To create temporary working directories.

### Syntax Rules and Structure

**General Syntax: `mkdir()`**
```php
$success = mkdir(
    string $directory,
    int $permissions = 0777,
    bool $recursive = false,
    ?resource $context = null
): bool;
```

**Component Breakdown**

- `$directory` — path to the new directory.
- `$permissions` — octal mode (default `0777`, modified by `umask`).
- `$recursive` — create parent directories if needed.
- `$context` — stream context.
- Returns `true` on success, `false` on failure.

**General Syntax: Safe Directory Creation**
```php
$dir = __DIR__ . '/uploads';
if (!is_dir($dir)) {
    if (!mkdir($dir, 0755, true)) {
        throw new RuntimeException("Failed to create directory: $dir");
    }
}
```

**Component Breakdown**

- `is_dir()` — checks if the directory already exists.
- `mkdir($dir, 0755, true)` — creates the directory and any missing parents.
- Throws an exception on failure.

**Syntax Rules**

- The default permissions `0777` are modified by `umask` (typically `0022`, resulting in `0755`).
- Use `0755` for directories that need to be readable by others.
- Use `0700` for directories that should be private.
- The `recursive` flag creates missing parent directories.
- `mkdir()` fails if the directory already exists.
- `mkdir()` requires write permission on the parent directory.
- `mkdir()` is subject to `open_basedir` restrictions.

**Constraints and Limitations**

- `mkdir()` fails if the parent directory does not exist and `recursive` is `false`.
- The actual permissions are affected by `umask`.
- On Windows, the permissions argument is ignored.
- `mkdir()` does not work on remote filesystems unless the stream wrapper supports it.
- Race conditions: check `is_dir()` then `mkdir()` can fail if another process creates the directory between the two calls.
- Use `@mkdir()` only if you handle the error properly; suppressing errors is discouraged.

### Annotated Code Examples and Expected Outputs

#### Example 1: Creating a Single Directory

**File: `create_dir.php`**
```php
<?php
$dir = __DIR__ . '/uploads';

if (is_dir($dir)) {
    echo 'Directory already exists.';
} else {
    if (mkdir($dir, 0755)) {
        echo 'Directory created.';
    } else {
        echo 'Failed to create directory.';
    }
}
?>
```

**Expected Output (first run)**
```
Directory created.
```
**Expected Output (second run)**
```
Directory already exists.
```

**Why This Output Occurs**

- `is_dir()` checks existence.
- `mkdir()` creates the directory with `0755` permissions (modified by `umask`).
- The second run finds the directory already present.

#### Example 2: Recursive Directory Creation

**File: `create_recursive.php`**
```php
<?php
$dir = __DIR__ . '/storage/cache/images';

if (!is_dir($dir)) {
    if (mkdir($dir, 0755, true)) {
        echo 'Nested directories created.';
    } else {
        echo 'Failed to create directories.';
    }
} else {
    echo 'Directories already exist.';
}
?>
```

**Expected Output**
```
Nested directories created.
```

**Why This Output Occurs**

- The `recursive` flag creates `storage`, `cache`, and `images` in one call.
- Each level is created with `0755` permissions.

### Real-World Cases

- **Upload storage:** Create a dated directory structure for uploads.
- **Cache systems:** Create cache directories per user or resource.
- **Multi-tenant apps:** Create a directory per tenant.
- **Log organization:** Create monthly or daily log directories.
- **Temporary workspaces:** Create a temp directory for batch processing.

---

## 3. Deleting Directories — `rmdir()` and Recursive Deletion

### Definitions

**Core Definition**  
`rmdir()` removes an empty directory. Recursive deletion removes a directory and all its contents.

**Technical Definition**  
`rmdir(string $directory, ?resource $context = null): bool` removes the directory if it is empty. Returns `true` on success, `false` on failure. Emits `E_WARNING` if the directory is not empty or does not exist. Recursive deletion requires iterating over the directory contents and deleting each file/subdirectory before removing the directory itself.

**Beginner-Friendly Explanation**  
`rmdir()` deletes a folder, but only if it is empty. If the folder contains files or subfolders, you must delete them first. Recursive deletion is a custom routine that walks through the folder and deletes everything inside before removing the folder itself.

### Purposes

- To clean up temporary directories.
- To remove user-specific directories on account deletion.
- To implement cache invalidation.
- To manage disk space by deleting old data.
- To reset directory structures.
- To remove uploaded content when no longer needed.
- To implement "trash" or "purge" functionality.

### Syntax Rules and Structure

**General Syntax: `rmdir()`**
```php
$success = rmdir(string $directory, ?resource $context = null): bool;
```

**Component Breakdown**

- `$directory` — path to the directory.
- Returns `true` on success, `false` on failure.
- Fails if the directory is not empty.

**General Syntax: Recursive Deletion**
```php
function deleteDirectory(string $dir): bool {
    if (!is_dir($dir)) {
        return false;
    }
    $items = scandir($dir);
    foreach ($items as $item) {
        if ($item === '.' || $item === '..') continue;
        $path = $dir . DIRECTORY_SEPARATOR . $item;
        if (is_dir($path) && !is_link($path)) {
            deleteDirectory($path);
        } else {
            unlink($path);
        }
    }
    return rmdir($dir);
}
```

**Component Breakdown**

- `is_dir()` — checks if the path is a directory.
- `scandir()` — lists entries.
- `is_link()` — avoids following symlinks.
- `unlink()` — deletes files.
- `rmdir()` — removes the now-empty directory.
- Recursion handles nested directories.

**General Syntax: Recursive Deletion with `RecursiveIteratorIterator`**
```php
$iterator = new RecursiveIteratorIterator(
    new RecursiveDirectoryIterator($dir, FilesystemIterator::SKIP_DOTS),
    RecursiveIteratorIterator::CHILD_FIRST
);

foreach ($iterator as $file) {
    if ($file->isDir()) {
        rmdir($file->getPathname());
    } else {
        unlink($file->getPathname());
    }
}
rmdir($dir);
```

**Component Breakdown**

- `RecursiveDirectoryIterator` — iterates over the tree.
- `SKIP_DOTS` — skips `.` and `..`.
- `CHILD_FIRST` — visits children before parents.
- `isDir()` — distinguishes directories from files.

**Syntax Rules**

- `rmdir()` only removes empty directories.
- Use `unlink()` to delete files before removing the directory.
- Use `is_link()` to avoid following symlinks during recursive deletion.
- Use `CHILD_FIRST` when using `RecursiveIteratorIterator` for deletion.
- Check return values for failures.
- `rmdir()` is subject to `open_basedir` restrictions.

**Constraints and Limitations**

- `rmdir()` fails if the directory is not empty.
- Recursive deletion can be dangerous; validate paths carefully.
- Symlinks must be handled explicitly; deleting a symlink deletes the link, not the target.
- Permissions may prevent deletion of files owned by other users.
- Race conditions can occur if files are added during deletion.
- `rmdir()` does not work on remote filesystems unless the stream wrapper supports it.

### Annotated Code Examples and Expected Outputs

#### Example 1: Removing an Empty Directory

**File: `remove_dir.php`**
```php
<?php
$dir = __DIR__ . '/empty_dir';

if (!is_dir($dir)) {
    echo 'Directory does not exist.';
} else {
    if (rmdir($dir)) {
        echo 'Directory removed.';
    } else {
        echo 'Failed to remove directory (not empty?).';
    }
}
?>
```

**Expected Output**
```
Directory removed.
```

**Why This Output Occurs**

- `is_dir()` confirms the directory exists.
- `rmdir()` removes it because it is empty.

#### Example 2: Recursive Deletion

**File: `recursive_delete.php`**
```php
<?php
function deleteDirectory(string $dir): bool {
    if (!is_dir($dir)) return false;

    $items = scandir($dir);
    foreach ($items as $item) {
        if ($item === '.' || $item === '..') continue;
        $path = $dir . DIRECTORY_SEPARATOR . $item;

        if (is_dir($path) && !is_link($path)) {
            deleteDirectory($path);
        } else {
            unlink($path);
        }
    }

    return rmdir($dir);
}

$target = __DIR__ . '/cache';
if (deleteDirectory($target)) {
    echo 'Directory and contents deleted.';
} else {
    echo 'Failed to delete directory.';
}
?>
```

**Expected Output**
```
Directory and contents deleted.
```

**Why This Output Occurs**

- The function recursively deletes all contents.
- Files are deleted with `unlink()`.
- Symlinks are treated as files.
- Empty directories are removed with `rmdir()`.

### Real-World Cases

- **Cache cleanup:** Recursively delete old cache directories.
- **Account deletion:** Remove all user files and directories.
- **Temporary workspaces:** Delete temp directories after processing.
- **Log rotation:** Delete old log directories.
- **Reset functionality:** Wipe a directory and recreate it.

---

## 4. Directory Navigation — `chdir()`, `getcwd()`, and Path Helpers

### Definitions

**Core Definition**  
Directory navigation is the process of changing the current working directory and resolving paths relative to it.

**Technical Definition**  
`chdir(string $directory): bool` changes the current working directory of the PHP process. `getcwd(): string|false` returns the current working directory. `dirname()`, `basename()`, and `pathinfo()` manipulate paths. `realpath()` resolves a path to an absolute canonical path. `DIRECTORY_SEPARATOR` is the platform-specific directory separator.

**Beginner-Friendly Explanation**  
The "current working directory" is the folder PHP considers itself to be in. `getcwd()` tells you where you are; `chdir()` moves you somewhere else. Path helpers like `basename()` extract the filename from a path, and `dirname()` extracts the directory.

### Purposes

- To change the working directory for relative path operations.
- To determine the current working directory.
- To resolve relative paths to absolute paths.
- To extract filenames and directory names from paths.
- To build portable paths using `DIRECTORY_SEPARATOR`.
- To canonicalize paths for security checks.
- To navigate to a script's own directory.

### Syntax Rules and Structure

**General Syntax: `chdir()` and `getcwd()`**
```php
$success = chdir(string $directory): bool;
$cwd = getcwd(): string|false;
```

**Component Breakdown**

- `chdir()` — changes the current working directory.
- `getcwd()` — returns the current working directory.
- Returns `false` on failure.

**General Syntax: Path Helpers**
```php
$basename = basename(string $path, string $suffix = ''): string;
$dirname  = dirname(string $path, int $levels = 1): string;
$info     = pathinfo(string $path, int $flags = PATHINFO_ALL): array|string;
$real     = realpath(string $path): string|false;
```

**Component Breakdown**

- `basename()` — returns the trailing name component.
- `dirname()` — returns the parent directory path.
- `pathinfo()` — returns an array with `dirname`, `basename`, `extension`, `filename`.
- `realpath()` — resolves to an absolute canonical path; returns `false` if the path does not exist.

**General Syntax: `DIRECTORY_SEPARATOR`**
```php
$path = 'storage' . DIRECTORY_SEPARATOR . 'cache' . DIRECTORY_SEPARATOR . 'file.txt';
```

**Component Breakdown**

- `DIRECTORY_SEPARATOR` — `/` on Unix, `\` on Windows.
- Use it for portable path construction.

**Syntax Rules**

- `chdir()` affects relative path resolution for the rest of the request.
- `getcwd()` returns the current working directory.
- `basename()` strips directory components.
- `dirname()` returns the parent directory.
- `pathinfo()` returns an array or a specific component when a flag is passed.
- `realpath()` resolves symlinks and returns the canonical path.
- Use `DIRECTORY_SEPARATOR` for portable paths.

**Constraints and Limitations**

- `chdir()` can break relative includes; use absolute paths.
- `realpath()` returns `false` for non-existent paths.
- `basename()` is locale-dependent for multibyte filenames in some PHP versions.
- `pathinfo()` may not handle multibyte filenames correctly in older PHP versions.
- `chdir()` is not thread-safe in some SAPIs.
- `getcwd()` may return `false` if the current directory has been deleted.

### Annotated Code Examples and Expected Outputs

#### Example 1: Navigating and Returning

**File: `navigate.php`**
```php
<?php
echo 'Start: ' . getcwd() . '<br>';

if (chdir('/tmp')) {
    echo 'After chdir: ' . getcwd() . '<br>';
} else {
    echo 'Failed to change directory.';
}
?>
```

**Expected Output**
```
Start: /var/www/html
After chdir: /tmp
```

**Why This Output Occurs**

- `getcwd()` returns the initial working directory.
- `chdir('/tmp')` changes it.
- `getcwd()` confirms the change.

#### Example 2: Path Helpers

**File: `path_helpers.php`**
```php
<?php
$path = '/var/www/html/uploads/photo.jpg';

echo 'basename: ' . basename($path) . '<br>';
echo 'dirname: ' . dirname($path) . '<br>';

$info = pathinfo($path);
echo 'extension: ' . $info['extension'] . '<br>';
echo 'filename: ' . $info['filename'] . '<br>';

echo 'realpath: ' . (realpath($path) ?: 'not found') . '<br>';
echo 'separator: ' . DIRECTORY_SEPARATOR;
?>
```

**Expected Output**
```
basename: photo.jpg
dirname: /var/www/html/uploads
extension: jpg
filename: photo
realpath: not found
separator: /
```

**Why This Output Occurs**

- `basename()` extracts `photo.jpg`.
- `dirname()` extracts `/var/www/html/uploads`.
- `pathinfo()` returns an array with components.
- `realpath()` returns `false` because the file does not exist.
- `DIRECTORY_SEPARATOR` is `/` on Unix.

### Real-World Cases

- **Script bootstrapping:** Use `__DIR__` and `dirname()` to locate includes.
- **Path resolution:** Use `realpath()` to canonicalize paths for security checks.
- **Portable paths:** Use `DIRECTORY_SEPARATOR` for cross-platform code.
- **File managers:** Display relative paths.
- **CLI tools:** Change directory to process files in a target folder.

---

## 5. Directory Metadata and Permissions

### Definitions

**Core Definition**  
Directory metadata includes information about a directory such as permissions, size, modification time, and type. Permissions determine who can read, write, or execute within the directory.

**Technical Definition**  
`is_dir()`, `is_readable()`, `is_writable()`, `is_executable()` check directory properties. `stat()` returns directory statistics. `fileperms()` returns permissions. `chmod()` changes permissions. `SplFileInfo` and `DirectoryIterator` provide object-oriented access. `umask()` affects newly created directories.

**Beginner-Friendly Explanation**  
Directories have properties just like files: who can read them, who can write to them, when they were last changed, and how big they are. PHP provides functions to check and change these properties. The most important permission for a directory is "execute" — on Unix, you need execute permission to enter a directory and access its contents.

### Purposes

- To check if a directory is readable or writable before operating on it.
- To set appropriate permissions on created directories.
- To display directory metadata in a file manager.
- To diagnose permission errors.
- To verify that a directory is not world-writable.
- To implement access control.
- To detect symbolic links.

### Syntax Rules and Structure

**General Syntax: Permission Checks**
```php
$isDir     = is_dir($path);
$readable  = is_readable($path);
$writable  = is_writable($path);
$executable = is_executable($path);
$link      = is_link($path);
```

**Component Breakdown**

- `is_dir()` — true if the path is a directory.
- `is_readable()` — true if the directory can be read.
- `is_writable()` — true if the directory can be written to.
- `is_executable()` — true if the directory can be entered (Unix).
- `is_link()` — true if the path is a symbolic link.

**General Syntax: `stat()` and `fileperms()`**
```php
$stats = stat($dir);
$perms = substr(sprintf('%o', fileperms($dir)), -4);
```

**Component Breakdown**

- `stat()` returns an array of directory statistics.
- `fileperms()` returns permissions as an integer.
- Format with `sprintf('%o', ...)` and `substr(..., -4)`.

**General Syntax: `chmod()`**
```php
chmod($dir, 0755);
```

**Component Breakdown**

- `0755` — owner rwx, group rx, others rx.
- Must be octal (prefix with `0`).

**Common Directory Permission Modes**

| Mode | Meaning | Use Case |
|---|---|---|
| `0700` | Owner rwx only | Private directories |
| `0750` | Owner rwx; group rx | Group-shared directories |
| `0755` | Owner rwx; others rx | Public directories |
| `0775` | Owner rwx; group rwx; others rx | Shared write directories |
| `0777` | Everyone rwx | Never use (insecure) |

**Syntax Rules**

- Directories need execute permission to be entered.
- `is_writable()` returns `true` if the PHP user can create files in the directory.
- `chmod()` requires ownership or root.
- Use `0755` for public directories, `0700` for private.
- Results are cached; use `clearstatcache()`.

**Constraints and Limitations**

- On Windows, permission functions behave differently.
- `is_executable()` is not reliable on Windows.
- `chmod()` may be affected by `umask`.
- Permissions may be inherited from the parent directory.
- `is_writable()` checks the PHP user's permissions, not the web user's in all contexts.

### Annotated Code Examples and Expected Outputs

#### Example 1: Checking Directory Permissions

**File: `dir_perms.php`**
```php
<?php
$dir = __DIR__ . '/uploads';

if (!is_dir($dir)) {
    echo 'Not a directory.';
    exit;
}

echo 'Readable: ' . (is_readable($dir) ? 'yes' : 'no') . '<br>';
echo 'Writable: ' . (is_writable($dir) ? 'yes' : 'no') . '<br>';
echo 'Executable: ' . (is_executable($dir) ? 'yes' : 'no') . '<br>';
echo 'Permissions: ' . substr(sprintf('%o', fileperms($dir)), -4);
?>
```

**Expected Output**
```
Readable: yes
Writable: yes
Executable: yes
Permissions: 0755
```

**Why This Output Occurs**

- `is_readable()` and `is_writable()` check access.
- `is_executable()` checks the execute bit.
- `fileperms()` returns the octal permission string.

#### Example 2: Setting Secure Permissions

**File: `set_perms.php`**
```php
<?php
$dir = __DIR__ . '/private';

if (!is_dir($dir)) {
    mkdir($dir, 0700, true);
}

// Ensure the directory is private.
if (chmod($dir, 0700)) {
    echo 'Permissions set to 0700.';
} else {
    echo 'Failed to set permissions.';
}
?>
```

**Expected Output**
```
Permissions set to 0700.
```

**Why This Output Occurs**

- `mkdir()` creates the directory with `0700`.
- `chmod()` ensures the permissions are correct.
- Only the owner can read, write, or execute.

### Real-World Cases

- **Upload directories:** Set `0755` on public upload directories.
- **Private storage:** Set `0700` on directories containing sensitive data.
- **Shared workspaces:** Set `0770` for group-writable directories.
- **Cache directories:** Set `0755` for web-accessible caches.
- **Security audits:** Detect world-writable directories.

---

## 6. Recursive Directory Traversal — Iterators

### Definitions

**Core Definition**  
Recursive directory traversal is the process of visiting every file and subdirectory within a directory tree.

**Technical Definition**  
`RecursiveDirectoryIterator` extends `DirectoryIterator` to provide recursive iteration. `RecursiveIteratorIterator` flattens the recursive structure into a linear sequence. Flags like `SKIP_DOTS`, `CHILD_FIRST`, `LEAVES_ONLY`, `SELF_FIRST`, and `FOLLOW_SYMLINKS` control behavior. `FilesystemIterator` provides additional flags like `CURRENT_AS_PATHNAME`, `CURRENT_AS_FILEINFO`, `KEY_AS_PATHNAME`.

**Beginner-Friendly Explanation**  
Recursive traversal is like walking through a tree of folders and visiting every file and subfolder. PHP's iterators make this easy: `RecursiveDirectoryIterator` walks the tree, and `RecursiveIteratorIterator` lets you loop over every item with a simple `foreach`. Flags let you control whether you visit folders before or after their contents.

### Purposes

- To process every file in a directory tree.
- To calculate total directory size.
- To search for files matching a pattern.
- To build file indexes or manifests.
- To perform bulk operations (copy, move, delete).
- To implement backup or sync tools.
- To generate sitemaps or file listings.

### Syntax Rules and Structure

**General Syntax: Basic Recursive Traversal**
```php
$iterator = new RecursiveIteratorIterator(
    new RecursiveDirectoryIterator($dir, FilesystemIterator::SKIP_DOTS)
);

foreach ($iterator as $file) {
    echo $file->getPathname() . "\n";
}
```

**Component Breakdown**

- `RecursiveDirectoryIterator($dir, SKIP_DOTS)` — iterates the tree.
- `RecursiveIteratorIterator(...)` — flattens the tree.
- `getPathname()` — full path to the current item.

**General Syntax: Child-First Traversal**
```php
$iterator = new RecursiveIteratorIterator(
    new RecursiveDirectoryIterator($dir, FilesystemIterator::SKIP_DOTS),
    RecursiveIteratorIterator::CHILD_FIRST
);
```

**Component Breakdown**

- `CHILD_FIRST` — visits children before parents; useful for deletion.
- `SELF_FIRST` — visits parents before children; useful for copying.

**General Syntax: Filtering by Extension**
```php
$iterator = new RecursiveIteratorIterator(
    new RecursiveDirectoryIterator($dir, FilesystemIterator::SKIP_DOTS)
);

foreach ($iterator as $file) {
    if ($file->isFile() && $file->getExtension() === 'php') {
        echo $file->getPathname() . "\n";
    }
}
```

**Component Breakdown**

- `isFile()` — true for regular files.
- `getExtension()` — returns the file extension.
- Filter as needed.

**General Syntax: Following Symlinks**
```php
$iterator = new RecursiveIteratorIterator(
    new RecursiveDirectoryIterator($dir, FilesystemIterator::SKIP_DOTS | FilesystemIterator::FOLLOW_SYMLINKS)
);
```

**Component Breakdown**

- `FOLLOW_SYMLINKS` — follows symbolic links (use with caution; can cause infinite loops).

**Syntax Rules**

- Use `SKIP_DOTS` to avoid `.` and `..`.
- Use `CHILD_FIRST` for deletion; `SELF_FIRST` for copying.
- Use `LEAVES_ONLY` (default) to visit only leaves.
- Use `FOLLOW_SYMLINKS` carefully; it can cause infinite recursion.
- `getPathname()` returns the full path.
- `getFilename()` returns the name only.
- `getExtension()` returns the extension.
- `isDir()`, `isFile()`, `isLink()` classify entries.

**Constraints and Limitations**

- Following symlinks can cause infinite loops.
- Large directory trees consume memory.
- `RecursiveDirectoryIterator` throws `UnexpectedValueException` on unreadable directories.
- Permission errors may interrupt traversal.
- Performance depends on filesystem and cache.

### Annotated Code Examples and Expected Outputs

#### Example 1: Listing All Files Recursively

**File: `recursive_list.php`**
```php
<?php
$dir = __DIR__ . '/project';

$iterator = new RecursiveIteratorIterator(
    new RecursiveDirectoryIterator($dir, FilesystemIterator::SKIP_DOTS)
);

foreach ($iterator as $file) {
    if ($file->isFile()) {
        echo $file->getPathname() . '<br>';
    }
}
?>
```

**Expected Output**
```
/var/www/project/index.php<br>
/var/www/project/src/App.php<br>
/var/www/project/src/Controller.php<br>
/var/www/project/assets/style.css<br>
```

**Why This Output Occurs**

- `RecursiveDirectoryIterator` walks the tree.
- `SKIP_DOTS` skips `.` and `..`.
- `isFile()` filters out directories.
- `getPathname()` returns the full path.

#### Example 2: Calculating Total Directory Size

**File: `dir_size.php`**
```php
<?php
function directorySize(string $dir): int {
    $total = 0;
    $iterator = new RecursiveIteratorIterator(
        new RecursiveDirectoryIterator($dir, FilesystemIterator::SKIP_DOTS)
    );

    foreach ($iterator as $file) {
        if ($file->isFile()) {
            $total += $file->getSize();
        }
    }
    return $total;
}

$dir = __DIR__ . '/uploads';
echo 'Total size: ' . directorySize($dir) . ' bytes';
?>
```

**Expected Output**
```
Total size: 1048576 bytes
```

**Why This Output Occurs**

- The iterator visits every file.
- `getSize()` returns each file's size in bytes.
- The total is accumulated.

### Real-World Cases

- **Backup tools:** Traverse a directory tree and copy files.
- **Search engines:** Index all files in a project.
- **Cleanup scripts:** Find and delete old files.
- **Build tools:** Process source files recursively.
- **Storage analyzers:** Calculate directory sizes.

---

## Security Considerations

### Path Traversal

- **Attack:** Using `../` in user-supplied paths to escape the intended directory.
- **Mitigation:** Use `basename()`, validate against a whitelist, resolve with `realpath()` and verify the result starts with the allowed base directory.

### Symlink Attacks

- **Attack:** A symlink points outside the intended directory, causing operations to affect unintended files.
- **Mitigation:** Use `is_link()` to detect symlinks; avoid `FOLLOW_SYMLINKS` unless necessary; use `realpath()` and verify the resolved path.

### TOCTOU (Time-of-Check to Time-of-Use)

- **Attack:** A file or directory changes between the check (`is_dir()`) and the operation (`rmdir()`).
- **Mitigation:** Use atomic operations where possible; handle failures gracefully.

### World-Writable Directories

- **Risk:** Directories with `0777` permissions allow any user to create or delete files.
- **Mitigation:** Use `0755` for public directories, `0700` for private; never use `0777`.

### `open_basedir` Restrictions

- **Purpose:** Limits PHP file operations to specified directories.
- **Mitigation:** Configure `open_basedir` in `php.ini` to restrict filesystem access.

### Deprecated, Unsafe, or Version-Specific Features

| Feature | Status | Replacement |
|---|---|---|
| `dir()` | Still supported | `DirectoryIterator` |
| `readdir()` order | Filesystem-dependent | `scandir()` for sorted results |
| `opendir()` invalid path | `ValueError` in PHP 8.0.0 | Validate paths |
| `glob()` invalid flags | `ValueError` in PHP 8.3.0 | Use valid flags |
| `rmdir()` on non-empty dir | Fails | Recursive deletion |

### References Links

- PHP: Directory Functions — https://www.php.net/manual/en/ref.dir.php
- PHP: `scandir()` — https://www.php.net/manual/en/function.scandir.php
- PHP: `glob()` — https://www.php.net/manual/en/function.glob.php
- PHP: `opendir()` — https://www.php.net/manual/en/function.opendir.php
- PHP: `readdir()` — https://www.php.net/manual/en/function.readdir.php
- PHP: `closedir()` — https://www.php.net/manual/en/function.closedir.php
- PHP: `rewinddir()` — https://www.php.net/manual/en/function.rewinddir.php
- PHP: `mkdir()` — https://www.php.net/manual/en/function.mkdir.php
- PHP: `rmdir()` — https://www.php.net/manual/en/function.rmdir.php
- PHP: `chdir()` — https://www.php.net/manual/en/function.chdir.php
- PHP: `getcwd()` — https://www.php.net/manual/en/function.getcwd.php
- PHP: `dirname()` — https://www.php.net/manual/en/function.dirname.php
- PHP: `basename()` — https://www.php.net/manual/en/function.basename.php
- PHP: `pathinfo()` — https://www.php.net/manual/en/function.pathinfo.php
- PHP: `realpath()` — https://www.php.net/manual/en/function.realpath.php
- PHP: `is_dir()` — https://www.php.net/manual/en/function.is-dir.php
- PHP: `is_readable()` — https://www.php.net/manual/en/function.is-readable.php
- PHP: `is_writable()` — https://www.php.net/manual/en/function.is-writable.php
- PHP: `is_executable()` — https://www.php.net/manual/en/function.is-executable.php
- PHP: `chmod()` — https://www.php.net/manual/en/function.chmod.php
- PHP: `fileperms()` — https://www.php.net/manual/en/function.fileperms.php
- PHP: `stat()` — https://www.php.net/manual/en/function.stat.php
- PHP: `DirectoryIterator` — https://www.php.net/manual/en/class.directoryiterator.php
- PHP: `FilesystemIterator` — https://www.php.net/manual/en/class.filesystemiterator.php
- PHP: `RecursiveDirectoryIterator` — https://www.php.net/manual/en/class.recursivedirectoryiterator.php
- PHP: `RecursiveIteratorIterator` — https://www.php.net/manual/en/class.recursiveiteratoriterator.php
- PHP: `SplFileInfo` — https://www.php.net/manual/en/class.splfileinfo.php
- PHP: `clearstatcache()` — https://www.php.net/manual/en/function.clearstatcache.php
- OWASP: Path Traversal — https://owasp.org/www-community/attacks/Path_Traversal
- CWE-22: Improper Limitation of a Pathname — https://cwe.mitre.org/data/definitions/22.html
- CWE-59: Improper Link Resolution Before File Access — https://cwe.mitre.org/data/definitions/59.html
- CWE-732: Incorrect Permission Assignment — https://cwe.mitre.org/data/definitions/732.html