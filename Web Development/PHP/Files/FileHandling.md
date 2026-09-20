# PHP File Handling — Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**  
PHP File Handling is the set of built-in functions and language constructs used to read, write, append to, inspect, and manage files on the server's filesystem. It provides both high-level convenience functions (e.g., `file_get_contents()`, `file_put_contents()`) and low-level stream-based functions (e.g., `fopen()`, `fread()`, `fwrite()`, `fclose()`).

**Technical Definition**  
PHP's filesystem extension exposes a procedural API for interacting with the local filesystem through stream wrappers. It includes functions for opening files with specific access modes (`fopen()`), reading data (`fread()`, `fgets()`, `fgetc()`, `file_get_contents()`), writing data (`fwrite()`, `file_put_contents()`), checking existence (`file_exists()`), retrieving metadata (`stat()`, `filesize()`, `filemtime()`, `fileperms()`), and changing permissions (`chmod()`). The API operates on file paths or stream resources and is subject to `open_basedir` restrictions, `safe_mode` (removed in PHP 5.4), and filesystem permissions. PHP 8.1.0 deprecated `auto_detect_line_endings`; PHP 8.3.0 throws `ValueError` for invalid flags in `file()`.

**Beginner-Friendly Explanation**  
PHP can work with files on the server the same way you work with files on your computer — you can open them, read their contents, write new data, add data to the end, check if they exist, look at their size and modification date, and change who is allowed to read or write them. PHP provides simple functions for everyday tasks (like `file_get_contents()` to grab the entire contents of a file) and more detailed functions for when you need finer control (like `fopen()` and `fread()` for reading a file piece by piece).

---

### Key Characteristics

- PHP provides both high-level convenience functions and low-level stream functions.
- High-level functions (`file_get_contents()`, `file_put_contents()`, `readfile()`, `file()`) handle opening, reading/writing, and closing automatically.
- Low-level functions (`fopen()`, `fread()`, `fwrite()`, `fclose()`) require explicit handling but offer more control.
- File access modes determine whether a file is opened for reading, writing, appending, or exclusive creation.
- Filesystem functions results are cached; use `clearstatcache()` when a file may have changed during the same request.
- File permissions are managed via `chmod()` and checked via `is_readable()`, `is_writable()`, `is_executable()`.
- File metadata is accessible via `stat()`, `filesize()`, `filemtime()`, `filetype()`, `fileperms()`, and the `SplFileInfo` class.
- Security restrictions include `open_basedir`, `allow_url_fopen`, and `allow_url_include`.
- `auto_detect_line_endings` was deprecated in PHP 8.1.0.
- `file()` throws `ValueError` for invalid flags as of PHP 8.3.0.
- Files larger than 2 GB may cause unexpected results on 32-bit platforms with some functions.

---

### Prerequisites

- Basic PHP syntax: variables, functions, conditionals, loops.
- Basic filesystem concepts: paths, directories, permissions.
- A web server with PHP installed.
- Write permissions on the target directories for writing operations.
- Understanding of Unix file permissions (for `chmod()`).

---

### Related Programming Areas

- Stream wrappers and protocols (`php://`, `http://`, `ftp://`).
- File upload handling (`$_FILES`, `move_uploaded_file()`).
- Directory handling (`scandir()`, `glob()`, `mkdir()`).
- File locking (`flock()`).
- Object-oriented file handling (`SplFileObject`, `SplFileInfo`).
- CSV and JSON file processing (`fgetcsv()`, `json_encode()`).

---

## Core Concepts / Features

## 1. Reading Files

### Definitions

**Core Definition**  
Reading files is the process of opening a file and retrieving its contents, either as a whole or in parts (lines, characters, or byte ranges).

**Technical Definition**  
PHP offers multiple reading strategies. `file_get_contents()` reads an entire file into a string. `file()` reads an entire file into an array of lines. `readfile()` reads a file and writes it directly to the output buffer. `fopen()` combined with `fread()`, `fgets()`, or `fgetc()` allows piecewise reading. `fgetcsv()` parses CSV lines into arrays. All functions return `false` on failure and emit `E_WARNING`.

**Beginner-Friendly Explanation**  
Reading a file means getting its contents into your PHP script so you can display it, process it, or pass it along. The simplest way is `file_get_contents()`, which grabs the whole file at once. If you need to process the file line by line, `file()` gives you an array of lines, or you can open the file with `fopen()` and read it line by line with `fgets()`.

### Purposes

- To load configuration files, templates, or data files.
- To process log files line by line.
- To read CSV, JSON, or XML files for parsing.
- To display file contents to the browser (`readfile()`).
- To read large files in chunks to conserve memory.
- To read remote files via URL wrappers.
- To process user-uploaded files.

### Syntax Rules and Structure

**General Syntax: `file_get_contents()`**
```php
$contents = file_get_contents(
    string $filename,
    bool $use_include_path = false,
    ?resource $context = null,
    int $offset = 0,
    ?int $length = null
): string|false;
```

**Component Breakdown**

- `$filename` — path to the file or URL.
- `$use_include_path` — search the include path.
- `$context` — stream context resource.
- `$offset` — starting position (PHP 5.1+).
- `$length` — maximum bytes to read (PHP 5.1+).
- Returns the file contents as a string, or `false` on failure.

**General Syntax: `file()`**
```php
$lines = file(
    string $filename,
    int $flags = 0,
    ?resource $context = null
): array|false;
```

**Component Breakdown**

- `$filename` — path to the file.
- `$flags` — `FILE_USE_INCLUDE_PATH`, `FILE_IGNORE_NEW_LINES`, `FILE_SKIP_EMPTY_LINES`, `FILE_NO_DEFAULT_CONTEXT`.
- Returns an array of lines, each including the trailing newline unless `FILE_IGNORE_NEW_LINES` is set .

**General Syntax: `readfile()`**
```php
$bytes = readfile(
    string $filename,
    bool $use_include_path = false,
    ?resource $context = null
): int|false;
```

**Component Breakdown**

- Reads a file and writes it to the output buffer.
- Returns the number of bytes read, or `false` on failure.

**General Syntax: `fopen()` + `fgets()`**
```php
$handle = fopen($filename, 'r');
while (($line = fgets($handle)) !== false) {
    echo $line;
}
fclose($handle);
```

**Component Breakdown**

- `fopen($filename, 'r')` — opens the file for reading.
- `fgets($handle)` — reads one line at a time.
- `fclose($handle)` — closes the file handle.

**General Syntax: `fread()`**
```php
$handle = fopen($filename, 'rb');
$contents = fread($handle, filesize($filename));
fclose($handle);
```

**Component Breakdown**

- `'rb'` — read mode with binary safety.
- `filesize($filename)` — determines how many bytes to read.
- `fread()` — reads the specified number of bytes .

**Syntax Rules**

- `file_get_contents()` is the preferred method for reading an entire file into a string.
- `file()` returns an array of lines; use `FILE_IGNORE_NEW_LINES` to omit newlines.
- `readfile()` outputs directly; wrap in `ob_start()`/`ob_get_clean()` to capture.
- `fgets()` reads up to a newline or a specified length.
- `fgetc()` reads a single character.
- `fread()` reads a specified number of bytes.
- Always check for `false` return values.
- Use `'b'` in the mode for binary-safe operations on Windows.

**Constraints and Limitations**

- `file_get_contents()` loads the entire file into memory; not suitable for very large files.
- `readfile()` outputs directly; use output buffering to capture.
- `file()` loads the entire file into an array; memory-intensive for large files.
- `fgets()` may return `false` at EOF or on error.
- `fread()` requires a valid file handle and a positive length.
- Remote file reading requires `allow_url_fopen=On`.
- `auto_detect_line_endings` is deprecated in PHP 8.1.0.
- `file()` throws `ValueError` for invalid flags in PHP 8.3.0.

### Annotated Code Examples and Expected Outputs

#### Example 1: Reading with `file_get_contents()`

**File: `read_whole.php`**
```php
<?php
// Read the entire file into a string.
$contents = file_get_contents('data.txt');

if ($contents === false) {
    echo 'Failed to read file.';
} else {
    // Escape for HTML output.
    echo htmlspecialchars($contents, ENT_QUOTES, 'UTF-8');
}
?>
```

**Expected Output**
```
Hello, World!
This is a test file.
```

**Why This Output Occurs**

- `file_get_contents()` reads the entire file into `$contents`.
- The `false` check handles read failures.
- `htmlspecialchars()` escapes the content for safe HTML output.

#### Example 2: Reading Line by Line with `fgets()`

**File: `read_lines.php`**
```php
<?php
$handle = fopen('data.txt', 'r');

if ($handle === false) {
    echo 'Failed to open file.';
    exit;
}

$lineNumber = 1;
while (($line = fgets($handle)) !== false) {
    echo $lineNumber . ': ' . htmlspecialchars($line, ENT_QUOTES, 'UTF-8') . '<br>';
    $lineNumber++;
}

fclose($handle);
?>
```

**Expected Output**
```
1: Hello, World!<br>
2: This is a test file.<br>
```

**Why This Output Occurs**

- `fopen()` opens the file for reading.
- `fgets()` reads one line at a time until EOF.
- `fclose()` releases the file handle.

### Real-World Cases

- **Configuration files:** Read JSON or INI files with `file_get_contents()`.
- **Log processing:** Process large logs line by line with `fgets()`.
- **CSV import:** Use `fgetcsv()` to parse CSV rows.
- **Template rendering:** Load HTML templates with `file_get_contents()`.
- **File download:** Use `readfile()` to stream a file to the browser.

---

## 2. Writing Files

### Definitions

**Core Definition**  
Writing files is the process of creating a new file or overwriting an existing file with new content.

**Technical Definition**  
`file_put_contents()` writes a string to a file, creating it if it does not exist and truncating it if it does. `fopen()` with mode `'w'` opens a file for writing, truncating it to zero length; `fwrite()` writes data to the handle. Mode `'x'` creates a new file exclusively and fails if the file already exists. All write operations are subject to filesystem permissions.

**Beginner-Friendly Explanation**  
Writing a file means putting content into it. If the file already exists, writing replaces what was there. The simplest way is `file_put_contents()`, which writes a string to a file in one step. For more control, open the file with `fopen()` using `'w'` mode and write with `fwrite()`.

### Purposes

- To save user-generated content to disk.
- To write configuration files.
- To generate reports, logs, or cached data.
- To export data to CSV, JSON, or text files.
- To create files programmatically.
- To persist application state.
- To write files atomically using temporary files and `rename()`.

### Syntax Rules and Structure

**General Syntax: `file_put_contents()`**
```php
$bytes = file_put_contents(
    string $filename,
    mixed $data,
    int $flags = 0,
    ?resource $context = null
): int|false;
```

**Component Breakdown**

- `$filename` — path to the file.
- `$data` — string, array, or stream resource.
- `$flags` — `FILE_APPEND`, `LOCK_EX`, `FILE_USE_INCLUDE_PATH`.
- Returns the number of bytes written, or `false` on failure.

**General Syntax: `fopen()` + `fwrite()`**
```php
$handle = fopen('output.txt', 'w');
fwrite($handle, 'Hello, World!');
fclose($handle);
```

**Component Breakdown**

- `'w'` — write mode; truncates the file to zero length.
- `fwrite()` — writes the string to the file .
- `fclose()` — closes the handle.

**General Syntax: Exclusive Creation (`'x'` mode)**
```php
$handle = fopen('newfile.txt', 'x');
if ($handle === false) {
    echo 'File already exists.';
} else {
    fwrite($handle, 'Exclusive content');
    fclose($handle);
}
```

**Component Breakdown**

- `'x'` — creates a new file exclusively; fails if the file exists.
- Returns `false` and emits `E_WARNING` if the file exists .

**Common `fopen()` Modes**

| Mode | Description |
|---|---|
| `'r'` | Read only; pointer at beginning |
| `'r+'` | Read/write; pointer at beginning |
| `'w'` | Write only; truncates to zero; creates if not exists |
| `'w+'` | Read/write; truncates to zero; creates if not exists |
| `'a'` | Append; pointer at end; creates if not exists |
| `'a+'` | Read/append; pointer at end; creates if not exists |
| `'x'` | Exclusive write; fails if file exists |
| `'x+'` | Exclusive read/write; fails if file exists |
| `'c'` | Write only; creates if not exists; does not truncate |
| `'c+'` | Read/write; creates if not exists; does not truncate |

**Syntax Rules**

- `file_put_contents()` is equivalent to `fopen()`, `fwrite()`, `fclose()` in sequence.
- `'w'` mode truncates the file; `'a'` mode appends.
- `'x'` mode fails if the file exists.
- `'c'` mode does not truncate; pointer at beginning.
- Add `'b'` for binary-safe operations on Windows.
- Use `LOCK_EX` with `file_put_contents()` to acquire an exclusive lock.
- Check return values for `false` to detect failures.
- Writes are subject to filesystem permissions and `open_basedir`.

**Constraints and Limitations**

- `file_put_contents()` loads all data into memory; not suitable for huge datasets.
- `'w'` mode destroys existing content without warning.
- `'x'` mode is not supported on all filesystems.
- `fwrite()` may write fewer bytes than requested; check the return value .
- Concurrent writes without locking can corrupt data.
- Windows requires `'b'` for binary files.
- `file_put_contents()` with `FILE_APPEND` is atomic for small writes on local filesystems.

### Annotated Code Examples and Expected Outputs

#### Example 1: Writing with `file_put_contents()`

**File: `write_simple.php`**
```php
<?php
$data = "Hello, World!\nSecond line.\n";
$bytes = file_put_contents('output.txt', $data);

if ($bytes === false) {
    echo 'Failed to write file.';
} else {
    echo "Wrote $bytes bytes.";
}
?>
```

**Expected Output**
```
Wrote 27 bytes.
```

**Why This Output Occurs**

- `file_put_contents()` writes the string to `output.txt`.
- The return value is the number of bytes written.
- The file is created if it does not exist.

#### Example 2: Exclusive Creation with `'x'` Mode

**File: `write_exclusive.php`**
```php
<?php
$handle = fopen('unique.txt', 'x');

if ($handle === false) {
    echo 'File already exists or cannot be created.';
} else {
    fwrite($handle, 'Exclusive content');
    fclose($handle);
    echo 'File created successfully.';
}
?>
```

**Expected Output (first run)**
```
File created successfully.
```
**Expected Output (second run)**
```
File already exists or cannot be created.
```

**Why This Output Occurs**

- `'x'` mode creates the file only if it does not already exist.
- The second run fails because the file exists.
- This prevents accidental overwrites.

### Real-World Cases

- **Logging:** Write log entries with `file_put_contents(..., FILE_APPEND | LOCK_EX)`.
- **Cache files:** Write serialized data or JSON to a cache file.
- **Configuration:** Write user settings to a config file.
- **Report generation:** Write HTML or CSV reports.
- **File uploads:** Move uploaded files to a permanent location.

---

## 3. Appending Files

### Definitions

**Core Definition**  
Appending is the process of adding new content to the end of an existing file without modifying or deleting the existing content.

**Technical Definition**  
Appending is achieved by opening a file with mode `'a'` or `'a+'`, which places the file pointer at the end of the file. `file_put_contents()` with the `FILE_APPEND` flag performs the same operation. Writes in append mode are atomic on local filesystems for data not exceeding the filesystem's block size .

**Beginner-Friendly Explanation**  
Appending is like adding a new page to the end of a notebook instead of rewriting the whole book. Open the file with `'a'` mode and write — PHP automatically places your new content at the end. The existing content stays untouched.

### Purposes

- To add log entries without rewriting the entire log file.
- To append data to CSV or text files.
- To maintain a running history of events.
- To accumulate results across multiple requests.
- To implement simple audit trails.
- To add entries to a queue file.
- To avoid reading and rewriting large files.

### Syntax Rules and Structure

**General Syntax: `fopen()` with `'a'` Mode**
```php
$handle = fopen('log.txt', 'a');
fwrite($handle, "New log entry\n");
fclose($handle);
```

**Component Breakdown**

- `'a'` — append mode; pointer at end of file.
- `fwrite()` — writes data at the end.
- The file is created if it does not exist.

**General Syntax: `file_put_contents()` with `FILE_APPEND`**
```php
file_put_contents('log.txt', "New entry\n", FILE_APPEND | LOCK_EX);
```

**Component Breakdown**

- `FILE_APPEND` — appends instead of overwriting.
- `LOCK_EX` — acquires an exclusive lock during the write.

**General Syntax: `'a+'` Mode (Read and Append)**
```php
$handle = fopen('data.txt', 'a+');
fwrite($handle, "Appended line\n");
rewind($handle); // Move pointer to beginning to read.
$contents = fread($handle, filesize('data.txt'));
fclose($handle);
```

**Component Breakdown**

- `'a+'` — read and append; pointer at end for writing.
- `rewind()` — moves the pointer to the beginning for reading.

**Syntax Rules**

- `'a'` mode places the pointer at the end of the file for every write.
- `'a+'` allows reading and appending.
- `FILE_APPEND` is the flag for `file_put_contents()`.
- Appending is atomic for small writes on local filesystems.
- Use `LOCK_EX` to prevent concurrent write corruption.
- The file is created if it does not exist in append mode.

**Constraints and Limitations**

- Appending does not allow modifying existing content.
- Atomicity is not guaranteed on network filesystems or for large writes.
- Concurrent appends without locking may interleave data.
- `'a'` mode does not truncate the file, but it also does not provide random access for writing.
- `fseek()` does not affect where `fwrite()` writes in `'a'` mode; writes always go to the end.

### Annotated Code Examples and Expected Outputs

#### Example 1: Appending to a Log File

**File: `append_log.php`**
```php
<?php
$logEntry = date('Y-m-d H:i:s') . " - User logged in\n";

// Append with an exclusive lock.
$result = file_put_contents('access.log', $logEntry, FILE_APPEND | LOCK_EX);

if ($result === false) {
    echo 'Failed to write log.';
} else {
    echo "Logged $result bytes.";
}
?>
```

**Expected Output**
```
Logged 35 bytes.
```

**Why This Output Occurs**

- `FILE_APPEND` adds the entry to the end of `access.log`.
- `LOCK_EX` prevents concurrent writes.
- The return value is the number of bytes written.

#### Example 2: Appending with `fopen()` and `'a'`

**File: `append_fopen.php`**
```php
<?php
$handle = fopen('counter.txt', 'a+');

if ($handle === false) {
    echo 'Failed to open file.';
    exit;
}

// Read current count.
rewind($handle);
$current = (int)fread($handle, filesize('counter.txt'));
$new = $current + 1;

// Append new count.
fwrite($handle, $new . "\n");
fclose($handle);

echo "Count: $new";
?>
```

**Expected Output**
```
Count: 1
Count: 2
Count: 3
...
```

**Why This Output Occurs**

- `'a+'` opens the file for reading and appending.
- `rewind()` moves the pointer to the beginning to read the current count.
- `fwrite()` appends the new count to the end.
- Each execution increments the counter.

### Real-World Cases

- **Access logs:** Append request information to a log file.
- **Error logs:** Append error messages for debugging.
- **Audit trails:** Append user actions for compliance.
- **Data collection:** Append sensor readings or form submissions.
- **Simple counters:** Append incrementing values to a file.

---

## 4. File Existence

### Definitions

**Core Definition**  
File existence checks determine whether a file or directory exists at a given path.

**Technical Definition**  
`file_exists()` returns `true` if a file or directory exists. `is_file()` returns `true` only for regular files. `is_dir()` returns `true` only for directories. `is_link()` returns `true` for symbolic links. Results are cached; use `clearstatcache()` to force a fresh check .

**Beginner-Friendly Explanation**  
Before reading or writing a file, you often need to check whether it exists. `file_exists()` tells you if something is there (file or directory). `is_file()` tells you specifically that it is a regular file. `is_dir()` tells you it is a directory.

### Purposes

- To avoid errors when reading or writing files that may not exist.
- To check whether a file needs to be created.
- To validate user-supplied file paths.
- To implement conditional logic based on file presence.
- To check for the existence of lock files or marker files.
- To verify that a directory exists before writing into it.
- To distinguish between files and directories.

### Syntax Rules and Structure

**General Syntax: `file_exists()`**
```php
$exists = file_exists(string $filename): bool;
```

**Component Breakdown**

- Returns `true` if the file or directory exists; `false` otherwise.
- Results are cached; use `clearstatcache()` to refresh.

**General Syntax: `is_file()`**
```php
$isFile = is_file(string $filename): bool;
```

**Component Breakdown**

- Returns `true` only if the path is a regular file.
- Returns `false` for directories, symlinks to directories, etc. .

**General Syntax: `is_dir()`**
```php
$isDir = is_dir(string $filename): bool;
```

**Component Breakdown**

- Returns `true` only if the path is a directory .

**General Syntax: `is_link()`**
```php
$isLink = is_link(string $filename): bool;
```

**Component Breakdown**

- Returns `true` if the path is a symbolic link .

**Syntax Rules**

- `file_exists()` returns `true` for both files and directories.
- `is_file()` returns `true` only for regular files.
- `is_dir()` returns `true` only for directories.
- Results are cached; call `clearstatcache()` if the file may have changed.
- `file_exists()` does not distinguish between files and directories.
- Use `is_readable()` and `is_writable()` to check access permissions.

**Constraints and Limitations**

- Results are cached per request; `clearstatcache()` is needed after changes.
- `file_exists()` returns `false` for broken symlinks.
- `is_file()` may return `false` for special files (e.g., FIFOs, devices).
- On Windows, `is_executable()` is not reliable.
- `file_exists()` may return `false` for files outside `open_basedir`.

### Annotated Code Examples and Expected Outputs

#### Example 1: Checking File Existence

**File: `check_exists.php`**
```php
<?php
$path = 'data.txt';

if (file_exists($path)) {
    echo 'Path exists.';
    if (is_file($path)) {
        echo ' It is a regular file.';
    } elseif (is_dir($path)) {
        echo ' It is a directory.';
    }
} else {
    echo 'Path does not exist.';
}
?>
```

**Expected Output (file exists)**
```
Path exists. It is a regular file.
```
**Expected Output (directory exists)**
```
Path exists. It is a directory.
```
**Expected Output (nothing exists)**
```
Path does not exist.
```

**Why This Output Occurs**

- `file_exists()` checks existence.
- `is_file()` and `is_dir()` distinguish the type.

#### Example 2: Using `clearstatcache()`

**File: `stat_cache.php`**
```php
<?php
// First check.
var_dump(file_exists('test.txt')); // false

// Create the file.
file_put_contents('test.txt', 'content');

// Cached result may still return false.
var_dump(file_exists('test.txt')); // may still be false

// Clear the cache.
clearstatcache();

// Now the check is accurate.
var_dump(file_exists('test.txt')); // true
?>
```

**Expected Output**
```
bool(false)
bool(false)
bool(true)
```

**Why This Output Occurs**

- `file_exists()` caches results.
- `clearstatcache()` forces a fresh check.
- Without clearing, the cached `false` persists.

### Real-World Cases

- **Configuration loading:** Check if a config file exists before reading.
- **Cache systems:** Check for cached files before regenerating.
- **File uploads:** Verify the target directory exists.
- **Lock files:** Check for a lock file to prevent concurrent processes.
- **Include files:** Check if a template file exists before including.

---

## 5. File Metadata

### Definitions

**Core Definition**  
File metadata is information about a file other than its contents, such as size, modification time, type, permissions, and owner.

**Technical Definition**  
`stat()` returns an array of file statistics including `size`, `mtime`, `atime`, `ctime`, `mode`, `uid`, `gid`, and more . Convenience functions like `filesize()`, `filemtime()`, `filetype()`, and `fileperms()` extract individual values. The `SplFileInfo` class provides an object-oriented interface.

**Beginner-Friendly Explanation**  
Metadata is "data about data." For a file, it includes how big it is, when it was last changed, whether it is a file or directory, and who can read or write it. PHP provides simple functions like `filesize()` and `filemtime()` to get these values.

### Purposes

- To display file size in a user interface.
- To check when a file was last modified (cache invalidation).
- To determine if a path is a file or directory.
- To display file permissions or owner information.
- To implement file versioning or change detection.
- To validate file types before processing.
- To build file managers or galleries.

### Syntax Rules and Structure

**General Syntax: `stat()`**
```php
$stats = stat(string $filename): array|false;
```

**Component Breakdown**

- Returns an array with both numeric and associative keys.
- Keys include `dev`, `ino`, `mode`, `nlink`, `uid`, `gid`, `size`, `atime`, `mtime`, `ctime`, `blksize`, `blocks`.
- Returns `false` on failure .

**General Syntax: `filesize()`**
```php
$size = filesize(string $filename): int|false;
```

**Component Breakdown**

- Returns the file size in bytes, or `false` on error.
- Not available for remote files .

**General Syntax: `filemtime()`**
```php
$mtime = filemtime(string $filename): int|false;
```

**Component Breakdown**

- Returns the last modification time as a Unix timestamp.
- Use `date()` to format it .

**General Syntax: `filetype()`**
```php
$type = filetype(string $filename): string|false;
```

**Component Breakdown**

- Returns one of: `fifo`, `char`, `dir`, `block`, `link`, `file`, `unknown` .

**General Syntax: `fileperms()`**
```php
$perms = fileperms(string $filename): int|false;
```

**Component Breakdown**

- Returns the file permissions as an integer.
- Use `decoct()` or `substr(sprintf('%o', fileperms(...)), -4)` to display .

**General Syntax: `SplFileInfo`**
```php
$info = new SplFileInfo('data.txt');
echo $info->getSize();
echo $info->getMTime();
echo $info->getType();
```

**Component Breakdown**

- Object-oriented interface for file metadata.
- Methods: `getSize()`, `getMTime()`, `getATime()`, `getCTime()`, `getType()`, `getPerms()`, etc.

**Syntax Rules**

- Metadata results are cached; use `clearstatcache()`.
- `filesize()` may return incorrect results for files > 2 GB on 32-bit systems.
- `filemtime()` returns the modification time, not creation time.
- `fileperms()` returns an integer; use `decoct()` or `sprintf()` to format.
- `stat()` returns both numeric and string keys.
- `SplFileInfo` provides a cleaner object-oriented API.

**Constraints and Limitations**

- Results are cached; `clearstatcache()` is needed.
- `filesize()` may be inaccurate for files > 2 GB on 32-bit platforms.
- `filemtime()` is not available for remote files.
- `fileperms()` behavior varies on Windows.
- `stat()` may return `false` for inaccessible files.
- `SplFileInfo` requires an existing file for some methods.

### Annotated Code Examples and Expected Outputs

#### Example 1: Displaying File Metadata

**File: `metadata.php`**
```php
<?php
$file = 'data.txt';

if (!file_exists($file)) {
    echo 'File not found.';
    exit;
}

$size = filesize($file);
$mtime = filemtime($file);
$type = filetype($file);
$perms = substr(sprintf('%o', fileperms($file)), -4);

echo "Size: $size bytes<br>";
echo "Last modified: " . date('Y-m-d H:i:s', $mtime) . "<br>";
echo "Type: $type<br>";
echo "Permissions: $perms";
?>
```

**Expected Output**
```
Size: 27 bytes<br>
Last modified: 2026-09-20 14:30:00<br>
Type: file<br>
Permissions: 0644
```

**Why This Output Occurs**

- `filesize()` returns the size in bytes.
- `filemtime()` returns a Unix timestamp formatted by `date()`.
- `filetype()` returns `file`.
- `fileperms()` returns permissions formatted as octal.

#### Example 2: Using `SplFileInfo`

**File: `spl_info.php`**
```php
<?php
$info = new SplFileInfo('data.txt');

echo 'Size: ' . $info->getSize() . ' bytes<br>';
echo 'Modified: ' . date('Y-m-d H:i:s', $info->getMTime()) . '<br>';
echo 'Type: ' . $info->getType() . '<br>';
echo 'Perms: ' . substr(sprintf('%o', $info->getPerms()), -4);
?>
```

**Expected Output**
```
Size: 27 bytes<br>
Modified: 2026-09-20 14:30:00<br>
Type: file<br>
Perms: 0644
```

**Why This Output Occurs**

- `SplFileInfo` provides methods for each metadata value.
- The output matches the procedural version.

### Real-World Cases

- **File managers:** Display size, type, and modification date.
- **Cache systems:** Compare `filemtime()` to invalidate stale caches.
- **Upload validation:** Check `filesize()` against a maximum.
- **Log rotation:** Use `filemtime()` to determine when to rotate logs.
- **Content management:** Display file metadata to users.

---

## 6. File Permissions

### Definitions

**Core Definition**  
File permissions determine who can read, write, and execute a file. PHP can inspect and change these permissions.

**Technical Definition**  
`chmod()` changes file permissions using an octal mode. `fileperms()` retrieves the current permissions. `is_readable()`, `is_writable()`, and `is_executable()` check access. Permissions are represented as a three-digit octal number: owner, group, and others. Each digit is the sum of 4 (read), 2 (write), and 1 (execute) .

**Beginner-Friendly Explanation**  
File permissions are like rules about who can do what with a file. The three groups are the owner, the group, and everyone else. For each group, you can allow reading (4), writing (2), and executing (1). For example, `0644` means the owner can read and write, and everyone else can only read. PHP can check and change these permissions.

### Purposes

- To secure files by restricting access.
- To make scripts executable.
- To ensure the web server can write to a directory.
- To protect sensitive configuration files.
- To comply with security best practices.
- To set appropriate permissions on uploaded files.
- To diagnose permission-related errors.

### Syntax Rules and Structure

**General Syntax: `chmod()`**
```php
$success = chmod(string $filename, int $permissions): bool;
```

**Component Breakdown**

- `$filename` — path to the file.
- `$permissions` — octal mode (e.g., `0644`).
- Returns `true` on success, `false` on failure.
- Must be prefixed with `0` to be treated as octal .

**General Syntax: `fileperms()`**
```php
$perms = fileperms(string $filename): int|false;
```

**Component Breakdown**

- Returns the permissions as an integer.
- Format with `substr(sprintf('%o', $perms), -4)` .

**General Syntax: `is_readable()`, `is_writable()`, `is_executable()`**
```php
$readable = is_readable(string $filename): bool;
$writable = is_writable(string $filename): bool;
$executable = is_executable(string $filename): bool;
```

**Component Breakdown**

- `is_readable()` — checks if the file can be read.
- `is_writable()` — checks if the file can be written.
- `is_executable()` — checks if the file can be executed .

**Common Permission Modes**

| Mode | Meaning |
|---|---|
| `0600` | Owner read/write only |
| `0644` | Owner read/write; others read |
| `0755` | Owner read/write/execute; others read/execute |
| `0750` | Owner read/write/execute; group read/execute |
| `0700` | Owner read/write/execute only |

**Syntax Rules**

- `chmod()` requires the permissions to be an octal number (prefix with `0`).
- The file owner (or root) must execute `chmod()`.
- `is_writable()` may return `true` for directories.
- `is_executable()` is not reliable on Windows.
- Results are cached; use `clearstatcache()`.
- `chmod()` does not work on remote files.

**Constraints and Limitations**

- `chmod()` only works if the PHP user owns the file or is root.
- Windows does not support Unix-style permissions fully.
- `is_executable()` may not work correctly on Windows.
- Permissions may be affected by umask.
- `chmod()` does not work on remote files.
- Setting permissions too loosely is a security risk.

### Annotated Code Examples and Expected Outputs

#### Example 1: Checking and Changing Permissions

**File: `permissions.php`**
```php
<?php
$file = 'data.txt';

// Display current permissions.
$perms = substr(sprintf('%o', fileperms($file)), -4);
echo "Current permissions: $perms<br>";

// Check access.
echo 'Readable: ' . (is_readable($file) ? 'yes' : 'no') . '<br>';
echo 'Writable: ' . (is_writable($file) ? 'yes' : 'no') . '<br>';

// Change permissions to 0600 (owner read/write only).
if (chmod($file, 0600)) {
    echo 'Permissions changed to 0600.';
} else {
    echo 'Failed to change permissions.';
}
?>
```

**Expected Output**
```
Current permissions: 0644<br>
Readable: yes<br>
Writable: yes<br>
Permissions changed to 0600.
```

**Why This Output Occurs**

- `fileperms()` retrieves the current mode.
- `is_readable()` and `is_writable()` check access.
- `chmod($file, 0600)` changes permissions to owner-only read/write.

#### Example 2: Using `chmod()` Correctly

```php
<?php
// Correct: octal mode with leading zero.
chmod('/somedir/somefile', 0755);

// Incorrect: decimal 755 (likely wrong).
chmod('/somedir/somefile', 755);
?>
```

**Explanation**

- `0755` is octal 755 (owner rwx, group rx, others rx).
- `755` is decimal 755, which is a different permission set.
- Always prefix with `0` to use octal notation .

### Real-World Cases

- **Upload directories:** Set `0755` on upload directories.
- **Config files:** Set `0600` on sensitive configuration files.
- **Scripts:** Set `0755` on CLI scripts to make them executable.
- **Cache files:** Set `0644` on cache files.
- **Log files:** Set `0644` on log files.

---

## Security Considerations

### File Path Traversal

- **Attack:** Using `../` in file paths to access files outside the intended directory.
- **Mitigation:** Use `basename()`, validate paths against a whitelist, use `realpath()`.

### Remote File Inclusion

- **Attack:** Including remote files via `allow_url_include`.
- **Mitigation:** Set `allow_url_include=Off`, `allow_url_fopen=Off` where possible.

### Race Conditions (TOCTOU)

- **Attack:** Time-of-check to time-of-use between `file_exists()` and `fopen()`.
- **Mitigation:** Use exclusive creation modes (`'x'`), atomic operations, or file locking.

### World-Writable Files

- **Risk:** Files with `0777` permissions can be modified by any user.
- **Mitigation:** Use `0644` for files and `0755` for directories; never use `0777`.

### Deprecated, Unsafe, or Version-Specific Features

| Feature | Status | Replacement |
|---|---|---|
| `auto_detect_line_endings` | Deprecated in PHP 8.1.0 | Handle `\r` manually |
| `safe_mode` | Removed in PHP 5.4 | `open_basedir` |
| `magic_quotes_runtime` | Removed in PHP 5.4 | Manual escaping |
| `file()` invalid flags | `ValueError` in PHP 8.3.0 | Use valid flags |

### References Links

- PHP: Filesystem Functions — https://www.php.net/manual/en/ref.filesystem.php
- PHP: `file_get_contents()` — https://www.php.net/manual/en/function.file-get-contents.php
- PHP: `file()` — https://www.php.net/manual/en/function.file.php
- PHP: `readfile()` — https://www.php.net/manual/en/function.readfile.php
- PHP: `fopen()` — https://www.php.net/manual/en/function.fopen.php
- PHP: `fread()` — https://www.php.net/manual/en/function.fread.php
- PHP: `fgets()` — https://www.php.net/manual/en/function.fgets.php
- PHP: `fwrite()` — https://www.php.net/manual/en/function.fwrite.php
- PHP: `file_put_contents()` — https://www.php.net/manual/en/function.file-put-contents.php
- PHP: `file_exists()` — https://www.php.net/manual/en/function.file-exists.php
- PHP: `is_file()` — https://www.php.net/manual/en/function.is-file.php
- PHP: `is_dir()` — https://www.php.net/manual/en/function.is-dir.php
- PHP: `stat()` — https://www.php.net/manual/en/function.stat.php
- PHP: `filesize()` — https://www.php.net/manual/en/function.filesize.php
- PHP: `filemtime()` — https://www.php.net/manual/en/function.filemtime.php
- PHP: `filetype()` — https://www.php.net/manual/en/function.filetype.php
- PHP: `fileperms()` — https://www.php.net/manual/en/function.fileperms.php
- PHP: `chmod()` — https://www.php.net/manual/en/function.chmod.php
- PHP: `is_readable()` — https://www.php.net/manual/en/function.is-readable.php
- PHP: `is_writable()` — https://www.php.net/manual/en/function.is-writable.php
- PHP: `is_executable()` — https://www.php.net/manual/en/function.is-executable.php
- PHP: `clearstatcache()` — https://www.php.net/manual/en/function.clearstatcache.php
- PHP: `SplFileInfo` — https://www.php.net/manual/en/class.splfileinfo.php
- PHP: `SplFileObject` — https://www.php.net/manual/en/class.splfileobject.php
- OWASP: Path Traversal — https://owasp.org/www-community/attacks/Path_Traversal
- CWE-22: Improper Limitation of a Pathname — https://cwe.mitre.org/data/definitions/22.html
- CWE-377: Insecure Temporary File — https://cwe.mitre.org/data/definitions/377.html
- CWE-732: Incorrect Permission Assignment for Critical Resource — https://cwe.mitre.org/data/definitions/732.html