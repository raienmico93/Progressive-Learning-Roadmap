# C++ Filesystem (C++17) — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
The C++17 Filesystem library is a component of the C++ Standard Library, declared in the `<filesystem>` header, that provides facilities for performing operations on file systems and their components, such as paths, regular files, and directories.

**Technical Definition**
The Filesystem library, defined in the `std::filesystem` namespace, offers classes (`path`, `directory_entry`, `directory_iterator`, `recursive_directory_iterator`, etc.) and non-member functions for path manipulation, file and directory operations, file system queries, and error handling via `filesystem_error` exceptions or `error_code` overloads. The library was originally developed as Boost.Filesystem, published as the technical specification ISO/IEC TS 18822:2015, and finally merged into ISO C++ as of C++17.

**Beginner-Friendly Explanation**
The Filesystem library lets your C++ program work with files and folders in a portable, type-safe way. Instead of using platform-specific C APIs, you can use `std::filesystem` to create, copy, move, and delete files and directories, navigate folder trees, check file sizes and attributes, and manipulate paths — all with a consistent interface across Windows, Linux, and macOS.

### Key Characteristics

- **Portable** — works consistently across operating systems, abstracting native path syntax.
- **Type-safe** — operations are performed through strongly typed classes (`path`, `directory_entry`), reducing errors.
- **Exception-safe** — operations can throw `filesystem_error` exceptions on failure, or use `error_code` overloads for non-throwing use.
- **RAII-friendly** — iterators and other objects manage resources automatically.
- **Path-centric** — `std::filesystem::path` is the central type for representing and manipulating paths.
- **Range-based iteration** — `directory_iterator` and `recursive_directory_iterator` work with range-based `for` loops.
- **C++17 and later** — some older compilers require linking with `-lstdc++fs` (GCC < 9.1) or `-lc++fs` (LLVM < 9.0).

### Prerequisites

- Basic C++17 syntax (classes, templates, range-based `for`).
- Familiarity with `std::string` and string manipulation.
- Understanding of exception handling (`try`/`catch`) and `std::error_code`.
- Basic knowledge of file system concepts (files, directories, paths).

### Related Programming Areas

- **File I/O** — `<fstream>` for reading and writing file contents.
- **String manipulation** — path string conversion and parsing.
- **System programming** — file system queries and permissions.
- **Build systems** — scanning source trees, managing output directories.
- **Data processing** — batch file operations, recursive directory traversal.

### Core Concepts / Features

1. `<filesystem>`
2. `std::filesystem::path`
3. Path operations
4. File operations
5. Directory operations
6. File system operations
7. `std::filesystem::directory_iterator`
8. `std::filesystem::recursive_directory_iterator`
9. Filesystem best practices

---

## 1. `<filesystem>`

### Definitions

**Core Definition**
`<filesystem>` is the standard library header that declares the entire Filesystem library, including all classes, functions, and types within the `std::filesystem` namespace.

**Technical Definition**
The header defines the classes `path`, `filesystem_error`, `directory_entry`, `directory_iterator`, `recursive_directory_iterator`, `file_status`, `space_info`, and various enums (`file_type`, `perms`, `perm_options`, `copy_options`, `directory_options`), along with a comprehensive set of non-member functions for file system operations. The header also includes (since C++20) three-way comparison operator support.

**Beginner-Friendly Explanation**
`<filesystem>` is the single header you include to access all the file and directory manipulation tools in C++17. It is your gateway to the entire `std::filesystem` namespace.

### Purposes

- To provide the declarations needed for all filesystem classes and functions.
- To expose the `std::filesystem` namespace.
- To enable portable file system programming.
- To unify file and directory operations under a single header.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <filesystem>
namespace fs = std::filesystem;   // common alias
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `<filesystem>` | Standard header for the Filesystem library. |
| `std::filesystem` | Namespace containing all library components. |

**Syntax Rules**
- Must be included before using any `std::filesystem` type or function.
- The namespace alias `namespace fs = std::filesystem;` is a common convention to shorten code.

**Constraints and Limitations**
- Requires a C++17 (or later) compiler and standard library.
- Some older toolchains require explicit linking with `-lstdc++fs` or `-lc++fs`.
- The library may be unavailable if the implementation does not support a hierarchical file system.

### Annotated Code Example

```cpp
#include <filesystem>    // enable filesystem library
#include <iostream>

namespace fs = std::filesystem;   // alias for brevity

int main() {
    fs::path p = "example.txt";   // std::filesystem::path available
    std::cout << "File: " << p << '\n';
}
```

**Expected Output**
```
File: example.txt
```

**Why this output?**  
`<filesystem>` declares `std::filesystem::path`, which is used to represent the file path; the stream inserter prints the path.

### Real-World Cases

- Any program that needs portable file system access.
- Build tools that scan directories.
- Backup utilities that copy and move files.

### References

- `<filesystem>` header — https://en.cppreference.com/w/cpp/header/filesystem

---

## 2. `std::filesystem::path`

### Definitions

**Core Definition**
`std::filesystem::path` is a class that represents a path on a file system.

**Technical Definition**
Objects of type `path` represent paths on a filesystem. Only syntactic aspects of paths are handled: the pathname may represent a non-existing path or even one that is not allowed to exist on the current file system or OS. A path name has the following syntax: an optional root-name (e.g., `"C:"` or `"//myserver"`), an optional root-directory (a directory separator that marks the path as absolute), and zero or more file-name elements separated by directory separators. Paths are implicitly convertible to and from `std::basic_string`.

**Beginner-Friendly Explanation**
A `path` object is like a smart string that knows about file system paths. It can be a relative path (`"data/file.txt"`) or an absolute path (`"/home/user/file.txt"`), and it knows how to break itself into parts (filename, extension, parent directory) and join with other paths.

### Purposes

- To represent file and directory paths in a portable way.
- To decompose paths into their components (filename, stem, extension, parent).
- To compose paths using the `/` operator.
- To convert between native and generic path formats.
- To serve as the primary argument type for filesystem functions.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::filesystem::path p("some/path");
std::filesystem::path p = "some/path";
```

**Component Breakdown**

| Member / Operation | Description |
|--------------------|-------------|
| Constructor | Creates a path from a string, C-string, or another path. |
| `operator/` | Concatenates two paths with a directory separator. |
| `filename()` | Returns the filename component. |
| `stem()` | Returns the filename without the extension. |
| `extension()` | Returns the file extension. |
| `parent_path()` | Returns the parent directory path. |
| `root_name()`, `root_directory()`, `root_path()` | Return root components. |
| `relative_path()` | Returns the path relative to the root. |
| `is_absolute()`, `is_relative()` | Query path type. |
| `string()`, `wstring()` | Convert to string types. |

**Syntax Rules**
- Paths are implicitly constructible from string types.
- The `/` operator joins paths, inserting a separator if needed.
- `begin()` and `end()` allow element-wise iteration over path components.
- Calling any non-const member function of a `path` invalidates all iterators referring to its elements.

**Constraints and Limitations**
- Only syntactic aspects are handled; the library does not verify that the path exists.
- Native path syntax may differ from the portable generic syntax (e.g., backslashes on Windows).
- Path comparison is lexicographical, not filesystem-aware (use `equivalent()` for that).

### Annotated Code Examples

**Example 1: Path decomposition**

```cpp
#include <filesystem>
#include <iostream>

namespace fs = std::filesystem;

int main() {
    fs::path p = "/usr/local/hello.txt";

    std::cout << "filename: " << p.filename() << '\n';       // "hello.txt"
    std::cout << "stem: " << p.stem() << '\n';               // "hello"
    std::cout << "extension: " << p.extension() << '\n';     // ".txt"
    std::cout << "parent: " << p.parent_path() << '\n';      // "/usr/local"
}
```

**Expected Output**
```
filename: "hello.txt"
stem: "hello"
extension: ".txt"
parent: "/usr/local"
```

**Why this output?**  
`filename()` returns the last component; `stem()` removes the extension; `extension()` returns `.txt`; `parent_path()` returns everything before the filename.

**Example 2: Path composition**

```cpp
#include <filesystem>
#include <iostream>

namespace fs = std::filesystem;

int main() {
    fs::path base = "/home/user";
    fs::path file = "document.txt";
    fs::path full = base / file;      // join with separator

    std::cout << full << '\n';
}
```

**Expected Output**
```
"/home/user/document.txt"
```

**Why this output?**  
The `/` operator concatenates the two paths, inserting the platform's preferred separator.

### Real-World Cases

- **Configuration loaders** — building paths to config files.
- **Build systems** — composing output paths from source paths.
- **File managers** — displaying and manipulating file paths.

### References

- `std::filesystem::path` — https://en.cppreference.com/w/cpp/filesystem/path

---

## 3. Path Operations

### Definitions

**Core Definition**
Path operations are the member functions and non-member functions that manipulate and query `std::filesystem::path` objects.

**Technical Definition**
Path operations include decomposition (`filename()`, `stem()`, `extension()`, `parent_path()`, `root_name()`, `root_directory()`), composition (`operator/`, `operator/=`), modification (`replace_filename()`, `replace_extension()`, `remove_filename()`), and queries (`is_absolute()`, `is_relative()`, `empty()`, `has_extension()`). Non-member functions include `absolute()`, `canonical()`, `relative()`, `proximate()`, and `weakly_canonical()`.

**Beginner-Friendly Explanation**
Path operations are the tools you use to take a path apart, put it back together differently, and ask questions about it (Is it absolute? What is its extension?).

### Purposes

- To decompose a path into its logical components.
- To build new paths from existing ones.
- To convert between absolute, relative, canonical, and weakly canonical forms.
- To query path properties without touching the file system.

### Syntax Rules and Structure

**General Syntax**
```cpp
p.filename();
p.stem();
p.extension();
p.parent_path();
p.replace_extension(".md");
fs::absolute(p);
fs::canonical(p);
fs::relative(p, base);
```

**Component Breakdown**

| Category | Operations |
|----------|------------|
| Decomposition | `filename`, `stem`, `extension`, `parent_path`, `root_name`, `root_directory`, `root_path`, `relative_path` |
| Composition | `operator/`, `operator/=`, `append`, `concat` |
| Modification | `replace_filename`, `replace_extension`, `remove_filename`, `clear` |
| Queries | `is_absolute`, `is_relative`, `empty`, `has_root_path`, `has_filename`, `has_extension` |
| Conversion | `absolute`, `canonical`, `weakly_canonical`, `relative`, `proximate` |

**Syntax Rules**
- Most decomposition functions return a new `path` object (they do not modify the original).
- `canonical()` requires the path to exist; `weakly_canonical()` does not.
- `relative()` computes a path relative to a base directory.

**Constraints and Limitations**
- `canonical()` throws `filesystem_error` if the path does not exist.
- Path operations are purely syntactic; they do not check existence unless explicitly stated.

### Annotated Code Examples

**Example 1: Changing the extension**

```cpp
#include <filesystem>
#include <iostream>

namespace fs = std::filesystem;

int main() {
    fs::path p = "/docs/report.txt";
    p.replace_extension(".md");
    std::cout << p << '\n';
}
```

**Expected Output**
```
"/docs/report.md"
```

**Why this output?**  
`replace_extension` replaces the existing extension with `.md`.

**Example 2: Canonical vs. weakly canonical**

```cpp
#include <filesystem>
#include <iostream>

namespace fs = std::filesystem;

int main() {
    fs::path p = "nonexistent/../file.txt";
    // fs::canonical(p);   // would throw if path does not exist
    fs::path wc = fs::weakly_canonical(p);
    std::cout << wc << '\n';
}
```

**Expected Output**  
(A normalised absolute path, e.g., `/current/dir/file.txt`.)

**Why this output?**  
`weakly_canonical` normalises the path even if some components do not exist.

### Real-World Cases

- **File format converters** — changing extensions before saving.
- **Path normalisation** — ensuring consistent paths for comparison.
- **Build systems** — computing relative paths between source and output.

### References

- `std::filesystem::path` — https://en.cppreference.com/w/cpp/filesystem/path
- `std::filesystem::absolute` — https://en.cppreference.com/w/cpp/filesystem/absolute
- `std::filesystem::canonical` — https://en.cppreference.com/w/cpp/filesystem/canonical

---

## 4. File Operations

### Definitions

**Core Definition**
File operations are the functions in `<filesystem>` that create, copy, move, remove, and query regular files.

**Technical Definition**
The library provides `copy_file()`, `copy()`, `rename()`, `remove()`, `resize_file()`, `file_size()`, `last_write_time()`, and `permissions()` for file manipulation. These functions operate on paths and can throw `filesystem_error` on failure.

**Beginner-Friendly Explanation**
File operations let you copy files, rename them, delete them, check their size, and change their permissions — all using simple function calls.

### Purposes

- To create, copy, and delete files.
- To rename or move files.
- To query file size and modification time.
- To change file permissions.

### Syntax Rules and Structure

**General Syntax**
```cpp
fs::copy_file(from, to);
fs::copy(from, to);
fs::rename(old_path, new_path);
fs::remove(path);
fs::file_size(path);
```

**Component Breakdown**

| Operation | Description |
|-----------|-------------|
| `copy_file` | Copies a single file. |
| `copy` | Copies files or directories (with options). |
| `rename` | Moves or renames a file/directory. |
| `remove` | Removes a file or empty directory. |
| `remove_all` | Removes a file or directory and all contents recursively. |
| `file_size` | Returns the size of a regular file. |
| `resize_file` | Changes the size of a regular file. |
| `last_write_time` | Gets or sets the last modification time. |

**Syntax Rules**
- `copy_file` fails if the destination exists (unless `copy_options::overwrite_existing` is used).
- `remove` returns `true` if the file was removed, `false` if it did not exist.
- `rename` may fail across file system boundaries.

**Constraints and Limitations**
- `file_size` on a non-regular file is implementation-defined.
- Removing a non-empty directory with `remove` fails (use `remove_all`).

### Annotated Code Examples

**Example 1: Copying and renaming a file**

```cpp
#include <filesystem>
#include <iostream>

namespace fs = std::filesystem;

int main() {
    fs::path src = "source.txt";
    fs::path dst = "backup.txt";

    fs::copy_file(src, dst, fs::copy_options::overwrite_existing);
    std::cout << "Copied " << src << " to " << dst << '\n';

    fs::rename(dst, "final.txt");
    std::cout << "Renamed to final.txt\n";
}
```

**Expected Output**
```
Copied "source.txt" to "backup.txt"
Renamed to final.txt
```

**Why this output?**  
`copy_file` duplicates the file; `rename` moves it to the new name.

**Example 2: File size and removal**

```cpp
#include <filesystem>
#include <iostream>

namespace fs = std::filesystem;

int main() {
    fs::path p = "data.bin";
    auto size = fs::file_size(p);
    std::cout << "Size: " << size << " bytes\n";

    if (fs::remove(p)) {
        std::cout << "File removed\n";
    }
}
```

**Expected Output**  
(File size followed by `File removed`.)

**Why this output?**  
`file_size` returns the size; `remove` deletes the file and returns `true`.

### Real-World Cases

- **Backup tools** — copying files to backup locations.
- **Log rotation** — renaming and removing old logs.
- **Installer scripts** — creating and moving application files.

### References

- `std::filesystem::copy_file` — https://en.cppreference.com/w/cpp/filesystem/copy_file
- `std::filesystem::rename` — https://en.cppreference.com/w/cpp/filesystem/rename
- `std::filesystem::remove` — https://en.cppreference.com/w/cpp/filesystem/remove
- `std::filesystem::file_size` — https://en.cppreference.com/w/cpp/filesystem/file_size

---

## 5. Directory Operations

### Definitions

**Core Definition**
Directory operations are the functions that create, remove, and query directories.

**Technical Definition**
The library provides `create_directory()`, `create_directories()`, `remove()`, `remove_all()`, `is_directory()`, `is_empty()`, and `current_path()` / `temp_directory_path()` for directory manipulation.

**Beginner-Friendly Explanation**
Directory operations let you make folders, delete them (even recursively), check whether a path is a directory, and find the current or temporary directory.

### Purposes

- To create single or nested directories.
- To remove directories and their contents recursively.
- To query directory properties.
- To navigate the directory hierarchy.

### Syntax Rules and Structure

**General Syntax**
```cpp
fs::create_directory(path);
fs::create_directories(path);   // creates all missing parents
fs::remove(path);               // empty directory only
fs::remove_all(path);           // recursive removal
fs::is_directory(path);
```

**Component Breakdown**

| Operation | Description |
|-----------|-------------|
| `create_directory` | Creates a single directory. |
| `create_directories` | Creates all missing directories in the path. |
| `remove` | Removes an empty directory. |
| `remove_all` | Removes a directory and all its contents recursively. |
| `is_directory` | Checks whether a path refers to a directory. |
| `current_path` | Gets or sets the current working directory. |
| `temp_directory_path` | Returns a directory suitable for temporary files. |

**Syntax Rules**
- `create_directory` returns `true` if the directory was created, `false` if it already existed.
- `create_directories` is idempotent — it returns `true` only if it created any directory.
- `remove` fails on non-empty directories.

**Constraints and Limitations**
- `remove_all` can be dangerous if used carelessly (it deletes everything under the path).
- Race conditions can occur if the file system changes between checks.

### Annotated Code Examples

**Example 1: Creating nested directories**

```cpp
#include <filesystem>
#include <iostream>

namespace fs = std::filesystem;

int main() {
    fs::path p = "a/b/c";
    if (fs::create_directories(p)) {
        std::cout << "Created " << p << '\n';
    }
}
```

**Expected Output**
```
Created "a/b/c"
```

**Why this output?**  
`create_directories` creates all three directories (`a`, `b`, `c`) if they do not exist.

**Example 2: Recursive removal**

```cpp
#include <filesystem>
#include <iostream>

namespace fs = std::filesystem;

int main() {
    fs::create_directories("temp/sub");
    // ... populate with files ...
    auto removed = fs::remove_all("temp");
    std::cout << "Removed " << removed << " items\n";
}
```

**Expected Output**  
(`Removed N items` where N is the total number of files and directories removed.)

**Why this output?**  
`remove_all` recursively deletes the directory and all its contents.

### Real-World Cases

- **Build systems** — creating output directories.
- **Cleanup scripts** — removing temporary directories.
- **Application installers** — creating directory structures.

### References

- `std::filesystem::create_directory` — https://en.cppreference.com/w/cpp/filesystem/create_directory
- `std::filesystem::create_directories` — https://en.cppreference.com/w/cpp/filesystem/create_directory
- `std::filesystem::remove_all` — https://en.cppreference.com/w/cpp/filesystem/remove

---

## 6. File System Operations

### Definitions

**Core Definition**
File system operations are the broader set of functions that query and manipulate file system-wide properties, such as free space, file status, and paths.

**Technical Definition**
This category includes `space()`, `status()`, `symlink_status()`, `exists()`, `equivalent()`, `is_regular_file()`, `is_symlink()`, and other type-query functions. These functions allow querying the state of file system objects without modifying them.

**Beginner-Friendly Explanation**
File system operations answer questions like “Does this file exist?”, “Is it a regular file or a symlink?”, “How much free space is on this disk?”, and “Are these two paths the same file?”.

### Purposes

- To query whether a path exists.
- To determine the type of a file system object.
- To check available disk space.
- To compare paths for equivalence.
- To inspect symbolic links.

### Syntax Rules and Structure

**General Syntax**
```cpp
fs::exists(path);
fs::is_regular_file(path);
fs::is_directory(path);
fs::is_symlink(path);
fs::space(path);
fs::equivalent(path1, path2);
```

**Component Breakdown**

| Operation | Description |
|-----------|-------------|
| `exists` | Checks whether a path refers to an existing file system object. |
| `equivalent` | Checks whether two paths refer to the same file system object. |
| `is_regular_file` | Checks for a regular file. |
| `is_directory` | Checks for a directory. |
| `is_symlink` | Checks for a symbolic link. |
| `space` | Returns information about free and available space. |
| `status` | Returns file status (type and permissions). |
| `symlink_status` | Returns status without following symlinks. |

**Syntax Rules**
- `exists` follows symlinks; `symlink_status` does not.
- `space` returns a `space_info` struct with `capacity`, `free`, and `available` members.
- Type queries (`is_regular_file`, etc.) are convenience wrappers around `status()`.

**Constraints and Limitations**
- Results may be stale if the file system changes between query and use.
- `space` may not be available on all file systems.

### Annotated Code Examples

**Example 1: Checking existence and type**

```cpp
#include <filesystem>
#include <iostream>

namespace fs = std::filesystem;

int main() {
    fs::path p = "data.txt";
    if (fs::exists(p)) {
        std::cout << p << " exists\n";
        if (fs::is_regular_file(p)) {
            std::cout << "It is a regular file\n";
        }
    }
}
```

**Expected Output**  
(`"data.txt" exists` and `It is a regular file` if applicable.)

**Why this output?**  
`exists` checks for presence; `is_regular_file` confirms the type.

**Example 2: Disk space**

```cpp
#include <filesystem>
#include <iostream>

namespace fs = std::filesystem;

int main() {
    fs::space_info si = fs::space("/");
    std::cout << "Capacity: " << si.capacity << " bytes\n";
    std::cout << "Free: " << si.free << " bytes\n";
}
```

**Expected Output**  
(Disk capacity and free space in bytes.)

**Why this output?**  
`space` returns a `space_info` structure with capacity and free space.

### Real-World Cases

- **Disk space monitors** — alerting when free space is low.
- **File validators** — checking existence and type before processing.
- **Symlink resolvers** — inspecting link targets.

### References

- `std::filesystem::exists` — https://en.cppreference.com/w/cpp/filesystem/exists
- `std::filesystem::space` — https://en.cppreference.com/w/cpp/filesystem/space
- `std::filesystem::status` — https://en.cppreference.com/w/cpp/filesystem/status

---

## 7. `std::filesystem::directory_iterator`

### Definitions

**Core Definition**
`std::filesystem::directory_iterator` is an input iterator that iterates over the entries of a single directory (without recursing into subdirectories).

**Technical Definition**
`directory_iterator` is a `LegacyInputIterator` that iterates over the `directory_entry` elements of a directory. The iteration order is unspecified, except that each directory entry is visited only once. The special pathnames `.` and `..` are skipped. If the iterator reports an error or is advanced past the last entry, it becomes equal to the end iterator.

**Beginner-Friendly Explanation**
`directory_iterator` lets you loop over everything inside a folder — files and subfolders — but it does not go into the subfolders. It is like listing the contents of a directory.

### Purposes

- To list the contents of a directory.
- To process each file or subdirectory in a folder.
- To filter files by extension or type.
- To perform operations on directory entries.

### Syntax Rules and Structure

**General Syntax**
```cpp
for (const auto& entry : fs::directory_iterator(path)) {
    // process entry
}
```

**Component Breakdown**

| Member / Operation | Description |
|--------------------|-------------|
| Constructor | Creates an iterator for a given directory path. |
| `operator*` | Returns the current `directory_entry`. |
| `operator->` | Accesses members of the current entry. |
| `operator++` | Advances to the next entry. |
| `begin()` / `end()` | Support range-based `for` loops (non-member functions). |

**Syntax Rules**
- The iteration order is unspecified.
- `.` and `..` are skipped automatically.
- Changes made to the directory after iterator creation may or may not be observed.

**Constraints and Limitations**
- Does not recurse into subdirectories (use `recursive_directory_iterator` for that).
- Dereferencing or incrementing the end iterator is undefined behaviour.

### Annotated Code Example

```cpp
#include <filesystem>
#include <iostream>

namespace fs = std::filesystem;

int main() {
    fs::path dir = "sandbox";

    for (const auto& entry : fs::directory_iterator(dir)) {
        std::cout << entry.path() << '\n';
    }
}
```

**Expected Output**  
(Lists each entry in `sandbox`, e.g., `"sandbox/file1.txt"`, `"sandbox/dir1"`, etc.)

**Why this output?**  
`directory_iterator` visits each entry in the directory (excluding `.` and `..`).

### Real-World Cases

- **Batch file processors** — iterating over files in a folder.
- **Directory listing tools** — implementing `ls`-like functionality.
- **File watchers** — scanning for new files.

### References

- `std::filesystem::directory_iterator` — https://en.cppreference.com/w/cpp/filesystem/directory_iterator

---

## 8. `std::filesystem::recursive_directory_iterator`

### Definitions

**Core Definition**
`std::filesystem::recursive_directory_iterator` is an input iterator that recursively iterates over the entries of a directory and all its subdirectories.

**Technical Definition**
`recursive_directory_iterator` visits each `directory_entry` in a directory tree, automatically descending into subdirectories. It supports `depth()` (to query the current recursion depth), `pop()` (to skip the remainder of a directory), and `disable_recursion_pending()` (to prevent descending into the current directory).

**Beginner-Friendly Explanation**
`recursive_directory_iterator` is like `directory_iterator`, but it goes into every subfolder, no matter how deep. It is like a “find” command that walks the entire directory tree.

### Purposes

- To traverse an entire directory tree recursively.
- To process all files in nested folders.
- To search for files by name or extension across a hierarchy.
- To compute directory sizes or counts.

### Syntax Rules and Structure

**General Syntax**
```cpp
for (const auto& entry : fs::recursive_directory_iterator(path)) {
    // process entry
}
```

**Component Breakdown**

| Member / Operation | Description |
|--------------------|-------------|
| Constructor | Creates a recursive iterator for a directory. |
| `depth()` | Returns the current recursion depth. |
| `pop()` | Moves the iterator to the next entry, skipping the current directory's contents. |
| `disable_recursion_pending()` | Prevents descending into the current directory. |
| `operator++` | Advances to the next entry (recursing if needed). |

**Syntax Rules**
- The iteration order is unspecified.
- `depth()` returns 0 for the root directory.
- `pop()` and `disable_recursion_pending()` allow pruning the traversal.

**Constraints and Limitations**
- Traversing very deep or large directory trees can be slow.
- Symlinks may be followed depending on `directory_options`.

### Annotated Code Example

```cpp
#include <filesystem>
#include <iostream>

namespace fs = std::filesystem;

int main() {
    fs::path root = "sandbox";

    for (const auto& entry : fs::recursive_directory_iterator(root)) {
        std::cout << entry.path() << '\n';
    }
}
```

**Expected Output**  
(Recursively lists all files and directories under `sandbox`, e.g., `"sandbox/a"`, `"sandbox/a/b"`, `"sandbox/file1.txt"`, etc.)

**Why this output?**  
`recursive_directory_iterator` visits every entry in the tree, including nested subdirectories.

### Real-World Cases

- **Search tools** — finding files across a project.
- **Backup utilities** — traversing all files to back up.
- **Code analysis** — scanning source trees for patterns.

### References

- `std::filesystem::recursive_directory_iterator` — https://en.cppreference.com/w/cpp/filesystem/recursive_directory_iterator

---

## 9. Filesystem Best Practices

### Definitions

**Core Definition**
Filesystem best practices are guidelines for using the C++17 Filesystem library safely, portably, and efficiently.

**Technical Definition**
These practices include preferring `std::filesystem` over C-style path operations (as recommended by the C++ Core Guidelines), handling errors via `filesystem_error` exceptions or `error_code` overloads, checking `exists()` before operations that require existence, and being mindful of file system race conditions (behaviour is undefined if calls introduce a race).

**Beginner-Friendly Explanation**
Best practices are good habits: always check whether a file exists before trying to use it, use exceptions or error codes to handle failures, and prefer `std::filesystem` over older C functions because it is safer and portable.

### Key Guidelines

| Guideline | Summary |
|-----------|---------|
| **Prefer `std::filesystem`** | Use the standard library instead of platform-specific C APIs. |
| **Handle errors** | Catch `filesystem_error` or use `error_code` overloads. |
| **Check existence** | Use `exists()` before operations that require the file to exist. |
| **Beware race conditions** | File system state can change between check and use. |
| **Use `canonical()` carefully** | It throws if the path does not exist; `weakly_canonical()` is safer. |
| **Prefer `remove_all()` with caution** | It deletes recursively and cannot be undone. |
| **Use `copy_options`** | Control overwrite behaviour explicitly. |
| **Test on target platforms** | Path syntax and availability may vary. |

### Annotated Code Example

```cpp
#include <filesystem>
#include <iostream>

namespace fs = std::filesystem;

int main() {
    try {
        fs::path p = "data.txt";
        if (fs::exists(p)) {                     // check before use
            auto size = fs::file_size(p);
            std::cout << "Size: " << size << '\n';
        }
    } catch (const fs::filesystem_error& e) {    // handle errors
        std::cerr << "Filesystem error: " << e.what() << '\n';
    }
}
```

**Expected Output**  
(File size if the file exists; error message otherwise.)

**Why this output?**  
`exists()` guards against operating on a missing file; the `try`/`catch` block handles any `filesystem_error`.

### Real-World Cases

- **Cross-platform applications** — ensuring consistent behaviour on Windows, Linux, and macOS.
- **Robust command-line tools** — validating paths and handling failures gracefully.
- **Library code** — providing portable, error-safe file system utilities.

### References

- C++ Core Guidelines — SL: The Standard Library — https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-stdlib
- `std::filesystem::filesystem_error` — https://en.cppreference.com/w/cpp/filesystem/filesystem_error
- Filesystem library notes on race conditions — https://en.cppreference.com/w/cpp/filesystem

---

## References

- Filesystem library (since C++17) — https://en.cppreference.com/w/cpp/filesystem
- Standard library header `<filesystem>` — https://en.cppreference.com/w/cpp/header/filesystem
- `std::filesystem::path` — https://en.cppreference.com/w/cpp/filesystem/path
- `std::filesystem::directory_iterator` — https://en.cppreference.com/w/cpp/filesystem/directory_iterator
- `std::filesystem::recursive_directory_iterator` — https://en.cppreference.com/w/cpp/filesystem/recursive_directory_iterator
- `std::filesystem::filesystem_error` — https://en.cppreference.com/w/cpp/filesystem/filesystem_error
- C++ Core Guidelines — https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines
- ISO/IEC TS 18822:2015 (Filesystem TS) — https://www.iso.org/standard/63410.html
- C++ Working Draft — Filesystem library — https://eel.is/c++draft/fs

---

This cheat sheet provides a comprehensive, structured reference for the C++17 Filesystem library, covering every requested component individually, with definitions, purposes, syntax rules, annotated code examples, real-world cases, and authoritative references.