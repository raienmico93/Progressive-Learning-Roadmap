# Python Directory Operations: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Directory operations in Python encompass the creation, deletion, listing, traversal, and inspection of filesystem directories. Python provides two primary interfaces for these tasks: the function-based `os` module (including `os.walk()` and `os.scandir()`) and the object-oriented `pathlib` module (with `Path.iterdir()`, `Path.glob()`, and `Path.rglob()`). Together, they enable programs to organize, navigate, and manage hierarchical filesystem structures programmatically.

**Technical Definition**

A directory (or folder) is a filesystem structure that contains entries mapping names to files and other directories. Python's directory operations are built on the `os` module, which exposes low-level system calls such as `mkdir()`, `rmdir()`, `listdir()`, and `scandir()`. The `pathlib` module provides an object-oriented abstraction where directory operations are methods on `Path` objects. Directory creation distinguishes between single-level (`os.mkdir()`, `Path.mkdir()`) and recursive (`os.makedirs()`, `Path.mkdir(parents=True)`) operations. Deletion distinguishes between empty-directory removal (`os.rmdir()`, `Path.rmdir()`) and recursive tree removal (`shutil.rmtree()`). Listing and traversal range from immediate-child enumeration (`os.listdir()`, `os.scandir()`, `Path.iterdir()`) to recursive pattern matching (`os.walk()`, `Path.glob()`, `Path.rglob()`). Directory entries may be files, subdirectories, symbolic links, or special filesystem objects; queries such as `is_file()` and `is_dir()` disambiguate them.

**Beginner-Friendly Explanation**

Directories are the folders on your computer that hold files and other folders. Python lets you create new folders, delete old ones, list what’s inside a folder, and walk through an entire tree of folders and subfolders. Think of it like organizing a filing cabinet: you can make new drawers (create directories), remove drawers (delete directories), look at what’s in a drawer (list a directory), and systematically open every drawer and sub-drawer (walk a directory tree). Python gives you two toolboxes for this: the classic `os` module and the modern `pathlib` module.

### Key Characteristics

- **Dual interfaces**: `os` (function-based, string-oriented) and `pathlib` (object-oriented, modern).
- **Recursive vs. single-level**: Creation and deletion distinguish between one-level and whole-tree operations.
- **Lazy vs. eager listing**: `os.scandir()`, `Path.iterdir()`, and `os.walk()` yield results lazily; `os.listdir()` returns a list eagerly.
- **Pattern matching**: `Path.glob()` and `Path.rglob()` enable glob-style filtering of directory contents.
- **Pruning and control**: `os.walk()` supports top-down or bottom-up traversal, error handling, and in-place pruning of subdirectories.
- **Efficiency**: `os.scandir()` caches file attributes in `DirEntry` objects, reducing system calls compared to `os.listdir()`.
- **Cross-platform portability**: Both modules abstract away separator and pathname differences between operating systems.
- **Security considerations**: Recursive deletion and traversal must be used carefully to avoid data loss and path traversal vulnerabilities.

### Prerequisites

- Basic Python syntax: variables, loops, and function calls.
- Familiarity with file I/O and path manipulation (`os.path`, `pathlib.Path`).
- Understanding of filesystem concepts: absolute vs. relative paths, current working directory, and directory trees.
- Awareness of exceptions such as `FileNotFoundError`, `PermissionError`, and `FileExistsError`.

### Related Programming Areas

- **File I/O**: Directory operations complement file reading and writing.
- **Build and deployment automation**: Scripts create output directories, clean build artifacts, and copy directory trees.
- **Data engineering**: Pipelines discover datasets by walking directory structures and matching file patterns.
- **Web development**: Static file serving and upload handling rely on directory listing and traversal.
- **Security tooling**: Path traversal defense and safe deletion are critical in file-handling code.

---

## Core Concepts / Features

---

### 1. Directory Creation

#### Definitions

**Core Definition**

Directory creation is the process of making new directories (folders) in the filesystem, either one level at a time or recursively including all missing parent directories.

**Technical Definition**

Python provides two primary functions for directory creation: `os.mkdir(path, mode=0o777, *, dir_fd=None)` creates a single directory and raises `FileExistsError` if it already exists and `FileNotFoundError` if its parent does not exist; `os.makedirs(name, mode=0o777, exist_ok=False)` recursively creates all intermediate-level directories needed to contain the leaf directory, and raises `FileExistsError` if the target exists unless `exist_ok=True`. The `pathlib` equivalents are `Path.mkdir(mode=0o777, parents=False, exist_ok=False)`, where `parents=True` mirrors `os.makedirs()` and `exist_ok=True` suppresses the error for existing directories.

**Beginner-Friendly Explanation**

Creating a directory is like making a new folder on your computer. `os.mkdir()` makes one folder, but it fails if the folder already exists or if the folder above it doesn’t exist. `os.makedirs()` is smarter: it creates every missing folder along the path, like building a whole road so you can reach the new house at the end. You can tell it to ignore the “already exists” error with `exist_ok=True`.

#### Purposes

- To create output directories before writing files into them.
- To build nested directory structures for organizing data, logs, or build artifacts.
- To ensure that required parent directories exist without manual step-by-step creation.
- To avoid `FileNotFoundError` when saving files to paths with missing parent directories.
- To support application scaffolding and project initialization.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
import os

os.mkdir(path, mode=0o777, *, dir_fd=None)
os.makedirs(name, mode=0o777, exist_ok=False)

from pathlib import Path
Path.mkdir(mode=0o777, parents=False, exist_ok=False)
```

**Component Breakdown**

| Parameter / Method | Description |
|--------------------|-------------|
| `path` / `name` | The directory path to create. |
| `mode` | Permission bits (default `0o777`, subject to umask). |
| `dir_fd` | Optional directory file descriptor for relative paths. |
| `exist_ok` | If `True`, suppress `FileExistsError` when the target exists. |
| `parents` (`pathlib`) | If `True`, create missing parent directories (like `makedirs`). |

**Syntax Rules**

- `os.mkdir()` creates only one level; its parent must exist.
- `os.makedirs()` creates all missing intermediate directories.
- `exist_ok=False` (the default) raises `FileExistsError` if the leaf directory exists.
- `exist_ok=True` suppresses the error only for the leaf directory, not for intermediate failures.
- `Path.mkdir(parents=True)` is the `pathlib` equivalent of `os.makedirs()`.
- `mode` is affected by the process umask and may be partially ignored on some platforms.

**Constraints and Limitations**

- `os.makedirs()` can behave unexpectedly if the path contains `..` components.
- `exist_ok=True` does not suppress errors for missing parents; it only suppresses the “already exists” error.
- Permission bits are platform-dependent and may be ignored on Windows.
- Recursive creation can create unintended directories if the path is constructed from untrusted input.

#### Annotated Code Examples

**Example 1: Single-Level vs. Recursive Creation**

```python
import os

# Single-level creation.
os.mkdir("output")
print("Created output directory.")

# Recursive creation (creates parents as needed).
os.makedirs("data/raw/2026/january", exist_ok=True)
print("Created nested directory structure.")

# Verify.
print("output exists:", os.path.isdir("output"))
print("nested exists:", os.path.isdir("data/raw/2026/january"))
```

**Expected Output:**

```
Created output directory.
Created nested directory structure.
output exists: True
nested exists: True
```

**Step-by-Step Breakdown:**

1. `os.mkdir("output")` creates a single directory; it would fail if `"output"` already existed.
2. `os.makedirs("data/raw/2026/january", exist_ok=True)` creates `data`, `data/raw`, `data/raw/2026`, and the leaf `january`.
3. `os.path.isdir()` confirms the directories exist.

**Why the Output is Produced:**

`os.mkdir()` handles one level; `os.makedirs()` recursively creates missing parents and, with `exist_ok=True`, does not raise if the leaf already exists.

---

**Example 2: `pathlib` Directory Creation**

```python
from pathlib import Path

# Single-level creation.
p = Path("reports")
p.mkdir(exist_ok=True)
print("Reports directory ready:", p.exists())

# Recursive creation with parents=True.
nested = Path("build/assets/images")
nested.mkdir(parents=True, exist_ok=True)
print("Nested directory ready:", nested.exists())
```

**Expected Output:**

```
Reports directory ready: True
Nested directory ready: True
```

**Step-by-Step Breakdown:**

1. `Path("reports").mkdir(exist_ok=True)` creates the directory if it does not exist.
2. `Path("build/assets/images").mkdir(parents=True, exist_ok=True)` creates all missing parents.

**Why the Output is Produced:**

`pathlib` unifies single-level and recursive creation into a single method with `parents` and `exist_ok` flags.

#### Real-World Cases

- **Web application uploads**: Creating user-specific upload directories under a base path.
- **Build systems**: Creating `build/`, `dist/`, and nested output directories before compilation.
- **Data pipelines**: Organizing datasets into `data/raw/`, `data/processed/`, and `data/output/`.
- **Log management**: Creating date-stamped log directories such as `logs/2026/01/`.

---

### 2. Directory Deletion

#### Definitions

**Core Definition**

Directory deletion is the process of removing directories from the filesystem, either individually (if empty) or recursively (including all contained files and subdirectories).

**Technical Definition**

Python provides `os.rmdir(path, *, dir_fd=None)` and `Path.rmdir()` to remove an empty directory; they raise `OSError` if the directory is not empty. For recursive deletion of an entire directory tree, `shutil.rmtree(path, ignore_errors=False, onerror=None)` is used; it removes all files and subdirectories and then the directory itself. The `pathlib` module does not provide a direct recursive deletion method; `shutil.rmtree()` is the standard tool. Deletion operations are irreversible: they do not use the trash or recycle bin.

**Beginner-Friendly Explanation**

Deleting a directory is like throwing away a folder. `os.rmdir()` only works if the folder is empty—it refuses to delete a folder that still has papers in it. If you want to throw away the folder and everything inside it, you need `shutil.rmtree()`, which is like dumping the entire filing cabinet into the shredder. Be careful: there’s no undo.

#### Purposes

- To clean up temporary directories after processing.
- To remove empty directories that are no longer needed.
- To delete entire directory trees when resetting build outputs or clearing caches.
- To implement cleanup routines in scripts and applications.
- To manage disk space by removing obsolete data.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
import os
import shutil

os.rmdir(path, *, dir_fd=None)
shutil.rmtree(path, ignore_errors=False, onerror=None)

from pathlib import Path
Path.rmdir()
```

**Component Breakdown**

| Function / Method | Description |
|-------------------|-------------|
| `os.rmdir(path)` | Removes an empty directory. |
| `Path.rmdir()` | Removes an empty directory. |
| `shutil.rmtree(path)` | Recursively removes a directory tree and all its contents. |
| `ignore_errors` | If `True`, suppress errors during recursive removal. |
| `onerror` | Callable invoked on errors with `(function, path, excinfo)`. |

**Syntax Rules**

- `os.rmdir()` and `Path.rmdir()` require the directory to be empty; otherwise `OSError` is raised.
- `shutil.rmtree()` removes everything under the given directory and then the directory itself.
- `shutil.rmtree()` is irreversible and does not use the trash.
- `ignore_errors=True` suppresses all errors, including permission errors.
- `onerror` provides a hook for handling errors such as read-only files.

**Constraints and Limitations**

- `os.rmdir()` cannot delete non-empty directories.
- `shutil.rmtree()` can be dangerous: an incorrect path (e.g., `"/"`) can cause catastrophic data loss.
- Deletion of open files may fail on Windows.
- Symbolic links are not followed by `rmtree`; the link itself is removed.

#### Annotated Code Examples

**Example 1: Removing an Empty Directory**

```python
import os

# Create and then remove an empty directory.
os.mkdir("temp_empty")
print("Before:", os.path.exists("temp_empty"))
os.rmdir("temp_empty")
print("After:", os.path.exists("temp_empty"))
```

**Expected Output:**

```
Before: True
After: False
```

**Step-by-Step Breakdown:**

1. `os.mkdir("temp_empty")` creates the directory.
2. `os.rmdir("temp_empty")` removes it because it is empty.

**Why the Output is Produced:**

`os.rmdir()` succeeds only when the directory contains no entries.

---

**Example 2: Recursive Deletion with `shutil.rmtree()`**

```python
import os
import shutil

# Build a small directory tree.
os.makedirs("temp_tree/subdir", exist_ok=True)
with open("temp_tree/file.txt", "w") as f:
    f.write("data")

print("Tree exists:", os.path.exists("temp_tree"))
shutil.rmtree("temp_tree")
print("Tree exists after rmtree:", os.path.exists("temp_tree"))
```

**Expected Output:**

```
Tree exists: True
Tree exists after rmtree: False
```

**Step-by-Step Breakdown:**

1. `os.makedirs("temp_tree/subdir", exist_ok=True)` creates the tree.
2. A file is created inside the tree.
3. `shutil.rmtree("temp_tree")` removes the file, the subdirectory, and the root directory.

**Why the Output is Produced:**

`shutil.rmtree()` recursively deletes all contents before removing the directory itself.

#### Real-World Cases

- **Build systems**: Removing the `build/` directory before a clean rebuild.
- **Temporary file management**: Deleting `tmp/` directories after task completion.
- **Cache invalidation**: Clearing cached assets or compiled bytecode.
- **Test fixtures**: Tearing down temporary directory structures after tests.

---

### 3. Directory Listing

#### Definitions

**Core Definition**

Directory listing is the process of enumerating the immediate entries (files, subdirectories, and other objects) within a single directory.

**Technical Definition**

Python provides three primary interfaces for listing a single directory level: `os.listdir(path='.')` returns a list of entry names (strings) without path prefixes; `os.scandir(path='.')` returns an iterator of `os.DirEntry` objects that expose the entry name, full path, and cached file attributes, often avoiding extra system calls; and `Path.iterdir()` yields `Path` objects for the immediate children of a directory. None of these functions recurse into subdirectories. `os.listdir()` includes all entries (excluding `.` and `..`), while `os.scandir()` and `Path.iterdir()` also exclude those special entries.

**Beginner-Friendly Explanation**

Listing a directory is like opening a folder and reading the names of everything directly inside it—but not looking inside any subfolders. `os.listdir()` gives you a plain list of names. `os.scandir()` gives you smart entry objects that already know whether each item is a file or folder, which can save time. `Path.iterdir()` gives you `Path` objects, which are convenient because they come with all the path manipulation methods you already know.

#### Purposes

- To discover what files and subdirectories exist in a directory.
- To filter entries by type (files vs. directories) or name pattern.
- To obtain both names and file attributes without additional system calls.
- To iterate over directory contents lazily for memory efficiency.
- To build higher-level operations such as batch processing or inventorying.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
import os

os.listdir(path='.')
os.scandir(path='.')

from pathlib import Path
Path.iterdir()
```

**Component Breakdown**

| Interface | Returns | Laziness |
|-----------|---------|----------|
| `os.listdir()` | List of name strings | Eager |
| `os.scandir()` | Iterator of `DirEntry` objects | Lazy |
| `Path.iterdir()` | Iterator of `Path` objects | Lazy |

**Syntax Rules**

- `os.listdir()` returns names only; combine with `os.path.join()` to get full paths.
- `os.scandir()` should be used as a context manager to ensure the directory stream is closed.
- `DirEntry` objects cache attribute information from the directory scan.
- `Path.iterdir()` yields `Path` objects; use `.name`, `.is_file()`, and `.is_dir()` directly.
- The order of entries is arbitrary and should not be relied upon; use `sorted()` if order matters.

**Constraints and Limitations**

- Listing and subsequent type checks are not atomic; entries may change between operations.
- `os.scandir()` may not cache all attributes; some methods may still trigger system calls.
- `Path.iterdir()` does not support pattern matching; use `glob()` for patterns.
- Permission errors may occur when listing directories without read access.

#### Annotated Code Examples

**Example 1: `os.listdir()` vs. `os.scandir()`**

```python
import os

# os.listdir(): names only.
names = os.listdir(".")
print("First three names:", names[:3])

# os.scandir(): entry objects with cached attributes.
with os.scandir(".") as entries:
    for entry in entries:
        kind = "DIR" if entry.is_dir() else "FILE"
        print(f"{entry.name:30s} {kind}")
```

**Expected Output:**

```
First three names: ['main.py', 'data', 'README.md']
main.py                        FILE
data                           DIR
README.md                      FILE
...
```

**Step-by-Step Breakdown:**

1. `os.listdir(".")` returns a list of names.
2. `os.scandir(".")` yields `DirEntry` objects; `entry.is_dir()` and `entry.is_file()` use cached information.

**Why the Output is Produced:**

`os.scandir()` avoids additional `stat()` calls for type checks, making it more efficient when metadata is needed.

---

**Example 2: `Path.iterdir()` with Filtering**

```python
from pathlib import Path

folder = Path(".")

print("Immediate files:")
for entry in folder.iterdir():
    if entry.is_file():
        print(" ", entry.name)
```

**Expected Output:**

```
Immediate files:
  main.py
  README.md
  ...
```

**Step-by-Step Breakdown:**

1. `folder.iterdir()` yields `Path` objects for immediate children.
2. `entry.is_file()` filters out directories.

**Why the Output is Produced:**

`Path.iterdir()` provides object-oriented access, and `is_file()` discriminates entries.

#### Real-World Cases

- **Batch file processing**: Listing all files in a directory for conversion or analysis.
- **Inventory tools**: Building a manifest of directory contents.
- **Upload handlers**: Enumerating user-uploaded files for validation.
- **Log rotation**: Listing old log files for archiving or deletion.

---

### 4. Directory Traversal and Walking

#### Definitions

**Core Definition**

Directory traversal (or walking) is the process of recursively visiting every directory and file in a directory tree, typically to perform an operation on each entry.

**Technical Definition**

Python provides `os.walk(top, topdown=True, onerror=None, followlinks=False)`, a generator that yields a 3-tuple `(dirpath, dirnames, filenames)` for each directory in the tree rooted at `top`. `dirpath` is the current directory path, `dirnames` is a list of subdirectory names, and `filenames` is a list of non-directory file names. The traversal order is top-down by default; setting `topdown=False` yields bottom-up. Modifying `dirnames` in-place when `topdown=True` prunes the traversal. Errors are ignored by default but can be handled with `onerror`. Symbolic links are not followed unless `followlinks=True`, which risks infinite recursion. The `pathlib` module offers `Path.glob()` and `Path.rglob()` for pattern-based recursive discovery, and `os.scandir()` can be used to implement custom traversal.

**Beginner-Friendly Explanation**

Walking a directory tree is like exploring a building floor by floor. `os.walk()` starts at the root and, for each folder, tells you: “Here is the folder you’re in, here are the subfolders inside it, and here are the files inside it.” You can decide to skip certain subfolders by removing them from the list, which is like closing a door and not exploring that wing of the building. `Path.rglob()` is a shortcut that finds all files matching a pattern anywhere in the tree.

#### Purposes

- To process every file in a directory tree, regardless of nesting depth.
- To implement recursive search, copy, move, or deletion operations.
- To build indexes or inventories of large directory structures.
- To prune traversal for performance or to skip irrelevant directories.
- To support pattern-based discovery of files by name or extension.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
import os

for dirpath, dirnames, filenames in os.walk(top, topdown=True, onerror=None, followlinks=False):
    # process dirpath, dirnames, filenames
    pass

from pathlib import Path
for p in Path(".").rglob("*.py"):
    print(p)
```

**Component Breakdown**

| Parameter | Description |
|-----------|-------------|
| `top` | Root directory of the traversal. |
| `topdown` | If `True` (default), yield parent before children; if `False`, bottom-up. |
| `onerror` | Callable for handling `OSError` during traversal. |
| `followlinks` | If `True`, follow symbolic links to directories. |
| `dirnames` | List of subdirectory names; modify in-place to prune when `topdown=True`. |

**Syntax Rules**

- `os.walk()` yields tuples for every directory, including the root.
- `dirnames` can be modified in-place only when `topdown=True`.
- `followlinks=True` can cause infinite recursion if links point to ancestors.
- `os.walk()` does not change the current working directory; avoid changing it during traversal.
- `Path.rglob(pattern)` is equivalent to `Path.glob("**/" + pattern)` and recursively matches.

**Constraints and Limitations**

- `os.walk()` follows a fixed traversal order; sort lists if deterministic order is required.
- Symbolic link cycles are not automatically detected when `followlinks=True`.
- `os.walk()` can be slower than custom `os.scandir()`-based traversal for specialized needs.
- `Path.rglob()` may be slower than `os.walk()` on very deep trees in some Python versions.

#### Annotated Code Examples

**Example 1: Basic `os.walk()` Traversal**

```python
import os

for dirpath, dirnames, filenames in os.walk("project"):
    print(f"Directory: {dirpath}")
    for name in filenames:
        print(f"  File: {name}")
    for name in dirnames:
        print(f"  Subdir: {name}")
```

**Expected Output:**

```
Directory: project
  File: README.md
  Subdir: src
  Subdir: tests
Directory: project/src
  File: main.py
  File: utils.py
Directory: project/tests
  File: test_main.py
```

**Step-by-Step Breakdown:**

1. The first iteration processes the root `project`, listing its files and subdirectories.
2. Subsequent iterations process each subdirectory in top-down order.

**Why the Output is Produced:**

`os.walk()` recursively yields tuples for every directory in the tree.

---

**Example 2: Pruning with `os.walk()`**

```python
import os

for dirpath, dirnames, filenames in os.walk("project", topdown=True):
    # Remove __pycache__ directories from traversal.
    if "__pycache__" in dirnames:
        dirnames.remove("__pycache__")
        print(f"Pruned __pycache__ in {dirpath}")
    for name in filenames:
        print(os.path.join(dirpath, name))
```

**Expected Output:**

```
project/README.md
project/src/main.py
project/src/utils.py
Pruned __pycache__ in project/src
project/tests/test_main.py
```

**Step-by-Step Breakdown:**

1. When `__pycache__` appears in `dirnames`, it is removed in-place.
2. `os.walk()` does not recurse into removed directories.

**Why the Output is Produced:**

Modifying `dirnames` in-place prunes the traversal, skipping entire subtrees.

---

**Example 3: Recursive Pattern Matching with `Path.rglob()`**

```python
from pathlib import Path

# Find all Python files recursively.
for py_file in Path("project").rglob("*.py"):
    print(py_file)
```

**Expected Output:**

```
project/src/main.py
project/src/utils.py
project/tests/test_main.py
```

**Step-by-Step Breakdown:**

1. `Path("project").rglob("*.py")` recursively searches for files ending in `.py`.
2. Each match is a `Path` object.

**Why the Output is Produced:**

`rglob()` combines recursive traversal with glob-style pattern matching.

#### Real-World Cases

- **Source code analysis**: Finding all Python files in a repository for linting or compilation.
- **Backup tools**: Recursively copying or archiving directory trees.
- **Log aggregation**: Collecting log files from nested application directories.
- **Security scanning**: Searching for files with specific extensions or names across a filesystem.

---

### 5. `os.walk()`

#### Definitions

**Core Definition**

`os.walk()` is a generator function that recursively traverses a directory tree, yielding a 3-tuple for each directory containing the directory path, subdirectory names, and file names.

**Technical Definition**

`os.walk(top, topdown=True, onerror=None, followlinks=False)` generates the file names in a directory tree by walking the tree either top-down or bottom-up. For each directory, it yields `(dirpath, dirnames, filenames)`. When `topdown=True`, the caller may modify `dirnames` in-place to prune the search. Errors from `scandir()` are ignored by default; an `onerror` callable can be provided. By default, symbolic links to directories are not followed. Since Python 3.5, `os.walk()` uses `os.scandir()` internally, improving performance by reducing `stat()` calls.

**Beginner-Friendly Explanation**

`os.walk()` is the classic Python tool for exploring an entire folder tree. It automatically opens every folder, tells you what’s inside, and then goes into each subfolder and does the same thing. It’s like a robot that systematically searches every room in a building and hands you a report for each room: the room’s address, the doors to other rooms, and the objects inside.

#### Purposes

- To recursively process every file and directory in a tree.
- To implement custom search, copy, move, or delete operations over trees.
- To prune traversal by modifying `dirnames` in-place.
- To handle traversal errors gracefully with `onerror`.
- To control traversal order (top-down vs. bottom-up) for operations like recursive deletion.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
import os

for dirpath, dirnames, filenames in os.walk(top, topdown=True, onerror=None, followlinks=False):
    # process
    pass
```

**Component Breakdown**

| Parameter | Description |
|-----------|-------------|
| `top` | Root directory. |
| `topdown` | Traversal order. |
| `onerror` | Error handler. |
| `followlinks` | Whether to follow symlinks. |
| `dirpath` | Current directory path. |
| `dirnames` | List of subdirectory names (mutable for pruning). |
| `filenames` | List of non-directory file names. |

**Syntax Rules**

- `dirnames` can be modified in-place only when `topdown=True`.
- When `topdown=False`, `dirnames` has already been processed and should not be modified.
- `onerror` receives an `OSError` instance.
- `followlinks=True` risks infinite recursion on cycles.
- `os.walk()` does not change the current working directory.

**Constraints and Limitations**

- Order of `dirnames` and `filenames` is arbitrary; sort for determinism.
- `followlinks=True` does not detect cycles.
- `os.walk()` may be slower than custom `scandir()`-based traversal for highly specialized needs.
- Errors are silently ignored unless `onerror` is provided.

#### Annotated Code Examples

**Example 1: Computing Directory Sizes**

```python
import os

for root, dirs, files in os.walk("project"):
    total = sum(os.path.getsize(os.path.join(root, name)) for name in files)
    print(f"{root}: {total} bytes")
```

**Expected Output:**

```
project: 1200 bytes
project/src: 4500 bytes
project/tests: 800 bytes
```

**Step-by-Step Breakdown:**

1. For each directory, the sizes of all files are summed.
2. `os.path.getsize()` obtains each file’s size in bytes.

**Why the Output is Produced:**

`os.walk()` provides the file names for each directory, enabling aggregation.

---

**Example 2: Bottom-Up Deletion**

```python
import os

# Delete all files and empty directories bottom-up (a simplified rmtree).
for root, dirs, files in os.walk("temp_tree", topdown=False):
    for name in files:
        os.remove(os.path.join(root, name))
    for name in dirs:
        os.rmdir(os.path.join(root, name))
os.rmdir("temp_tree")
```

**Expected Output:**

```
(temp_tree is completely removed.)
```

**Step-by-Step Breakdown:**

1. `topdown=False` processes children before parents.
2. Files are removed first, then directories, because `rmdir()` requires empty directories.

**Why the Output is Produced:**

Bottom-up traversal ensures that directories are empty before removal.

#### Real-World Cases

- **Build cleanup**: Removing compiled artifacts and empty directories after a build.
- **Data migration**: Collecting files from a legacy directory hierarchy.
- **Static site generators**: Discovering all content files for processing.
- **Compliance scanning**: Searching entire filesystems for files matching policy patterns.

---

### 6. `os.scandir()`

#### Definitions

**Core Definition**

`os.scandir(path='.')` returns an iterator of `os.DirEntry` objects, providing efficient access to directory entries and their cached attributes.

**Technical Definition**

`os.scandir()` yields `DirEntry` objects for each entry in the directory. `DirEntry` exposes `.name`, `.path`, `.is_file()`, `.is_dir()`, `.is_symlink()`, and `.stat()`, often using cached information from the directory scan to avoid additional system calls. It is more efficient than `os.listdir()` when file metadata is needed. `os.scandir()` should be used as a context manager to ensure the directory stream is closed promptly; otherwise, `ResourceWarning` may be emitted.

**Beginner-Friendly Explanation**

`os.scandir()` is like a supercharged version of `os.listdir()`. Instead of just handing you names, it hands you smart entry objects that already know whether they are files, folders, or links—and they remember some information so Python doesn’t have to ask the operating system again. This makes it faster when you need details about each item.

#### Purposes

- To list directory entries with efficient access to file attributes.
- To avoid redundant system calls when determining entry types.
- To provide full paths alongside names without manual joining.
- To support lazy iteration over large directories.
- To serve as the building block for custom traversal algorithms.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
import os

with os.scandir(path) as entries:
    for entry in entries:
        print(entry.name, entry.is_file(), entry.is_dir())
```

**Component Breakdown**

| Attribute / Method | Description |
|--------------------|-------------|
| `entry.name` | Entry name (string). |
| `entry.path` | Full path (string). |
| `entry.is_file()` | `True` if the entry is a regular file (follows symlinks by default). |
| `entry.is_dir()` | `True` if the entry is a directory. |
| `entry.is_symlink()` | `True` if the entry is a symbolic link. |
| `entry.stat()` | Returns a `stat_result` object. |

**Syntax Rules**

- Use `os.scandir()` as a context manager.
- `DirEntry` methods cache attribute information; subsequent calls may not require system calls.
- `is_file()` and `is_dir()` follow symbolic links by default; pass `follow_symlinks=False` to avoid this.
- Entries are yielded in arbitrary order.

**Constraints and Limitations**

- The directory stream must be closed; use `with`.
- Cached attributes may become stale if the filesystem changes during iteration.
- `DirEntry` objects become invalid after the iterator is exhausted or closed.
- Not all attributes are cached; `stat()` may still trigger a system call.

#### Annotated Code Examples

**Example 1: Listing Entries with Metadata**

```python
import os

with os.scandir(".") as entries:
    for entry in entries:
        if entry.is_file():
            size = entry.stat().st_size
            print(f"{entry.name:30s} {size} bytes")
        elif entry.is_dir():
            print(f"{entry.name:30s} <DIR>")
```

**Expected Output:**

```
main.py                        1024 bytes
data                           <DIR>
README.md                      512 bytes
```

**Step-by-Step Breakdown:**

1. `entry.is_file()` and `entry.is_dir()` classify entries using cached information.
2. `entry.stat().st_size` retrieves the file size.

**Why the Output is Produced:**

`os.scandir()` provides both names and attributes efficiently.

#### Real-World Cases

- **Directory inventory**: Building a list of files with sizes and types.
- **Filtered processing**: Selecting only regular files for batch operations.
- **Custom walkers**: Implementing traversal with pruning and custom error handling.

---

### 7. `Path.iterdir()`, `Path.glob()`, and `Path.rglob()`

#### Definitions

**Core Definition**

`Path.iterdir()` yields `Path` objects for the immediate children of a directory; `Path.glob(pattern)` yields paths matching a glob pattern within a single directory level; and `Path.rglob(pattern)` recursively yields paths matching a pattern throughout the directory tree.

**Technical Definition**

`Path.iterdir()` returns an iterator over the directory’s entries as `Path` objects. `Path.glob(pattern)` matches paths against a glob-style pattern using the pattern language defined by `pathlib`; `**` patterns enable recursive matching. `Path.rglob(pattern)` is equivalent to `Path.glob("**/" + pattern)` and recursively searches subdirectories. Both `glob()` and `rglob()` are lazy generators and support shell-style wildcards such as `*`, `?`, and `[]`.

**Beginner-Friendly Explanation**

`Path.iterdir()` is like opening a folder and getting back a list of smart path objects for everything inside. `Path.glob()` is like using a search filter: “show me all files ending in `.py` in this folder.” `Path.rglob()` is the recursive version: “show me all `.py` files anywhere under this folder, no matter how deep.”

#### Purposes

- To list immediate children as `Path` objects with full path functionality.
- To filter directory contents by name patterns without manual string matching.
- To recursively discover files by extension or name pattern.
- To integrate listing and matching with `pathlib`’s object-oriented API.
- To write concise, readable code for common file-discovery tasks.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
from pathlib import Path

p = Path("directory")

for child in p.iterdir():
    pass

for match in p.glob("*.py"):
    pass

for match in p.rglob("*.py"):
    pass
```

**Component Breakdown**

| Method | Description |
|--------|-------------|
| `iterdir()` | Yields `Path` objects for immediate children. |
| `glob(pattern)` | Yields paths matching `pattern` in the current directory (supports `**`). |
| `rglob(pattern)` | Yields paths matching `pattern` recursively. |

**Syntax Rules**

- `iterdir()` does not accept a pattern.
- `glob()` supports `*`, `?`, `[]`, and `**` for recursive matching.
- `rglob(pattern)` is equivalent to `glob("**/" + pattern)`.
- Results are yielded lazily.
- `glob()` and `rglob()` return `Path` objects, not strings.

**Constraints and Limitations**

- `iterdir()` lists all entries, including hidden files and directories.
- `glob()` patterns are matched against names, not full paths.
- `rglob()` may be slower than `os.walk()` on very deep trees in some Python versions.
- Symlink following behavior is platform- and version-dependent.

#### Annotated Code Examples

**Example 1: `iterdir()` vs. `glob()`**

```python
from pathlib import Path

folder = Path("project")

print("All entries:")
for entry in folder.iterdir():
    print(" ", entry.name)

print("Python files only:")
for entry in folder.glob("*.py"):
    print(" ", entry.name)
```

**Expected Output:**

```
All entries:
  main.py
  README.md
  src
Python files only:
  main.py
```

**Step-by-Step Breakdown:**

1. `iterdir()` yields every immediate child.
2. `glob("*.py")` filters to names ending in `.py`.

**Why the Output is Produced:**

`glob()` applies pattern matching to directory entries.

---

**Example 2: Recursive Discovery with `rglob()`**

```python
from pathlib import Path

for py_file in Path("project").rglob("*.py"):
    print(py_file)
```

**Expected Output:**

```
project/main.py
project/src/utils.py
project/src/models/user.py
```

**Step-by-Step Breakdown:**

1. `rglob("*.py")` recursively searches all subdirectories.
2. Each match is a `Path` object with a full relative path.

**Why the Output is Produced:**

`rglob()` combines recursive traversal with glob-style pattern matching.

#### Real-World Cases

- **Project scaffolding**: Finding all Python files for linting or packaging.
- **Asset discovery**: Locating images, templates, or static files by extension.
- **Configuration scanning**: Searching for `.yaml`, `.json`, or `.ini` files across a project.
- **Test discovery**: Recursively finding test files matching `test_*.py`.

---

### 8. Directory Best Practices

#### Definitions

**Core Definition**

Directory best practices are guidelines for writing directory-handling code that is safe, efficient, portable, and maintainable, leveraging the appropriate APIs and avoiding common pitfalls.

**Technical Definition**

Best practices for Python directory operations include: preferring `pathlib` for new code while remaining fluent in `os` and `os.path` for interoperability and performance-critical paths; using `os.scandir()` or `Path.iterdir()` instead of `os.listdir()` when metadata is needed; using `os.walk()` with `onerror` and pruning for recursive traversal; always resolving and validating user-supplied paths to prevent directory traversal attacks; using `exist_ok=True` and `parents=True` to make directory creation idempotent; and treating `shutil.rmtree()` with extreme caution because it is irreversible.

**Beginner-Friendly Explanation**

Directory best practices are the “safe driving rules” for working with folders. Use the modern `pathlib` tools when you can. Don’t trust paths that come from users—resolve them first to make sure they’re not trying to sneak outside your intended folder. Use `exist_ok=True` so your script doesn’t crash if a folder already exists. And be very careful with `shutil.rmtree()`: it deletes everything without a recycle bin, so double-check the path before pulling the trigger.

#### Purposes

- To write portable code that behaves consistently across operating systems.
- To avoid data loss from accidental recursive deletion.
- To prevent security vulnerabilities such as path traversal attacks.
- To improve performance by choosing the right listing and traversal APIs.
- To make directory operations idempotent and robust in the presence of existing files and permission issues.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
from pathlib import Path
import shutil

# Safe creation.
Path("output").mkdir(parents=True, exist_ok=True)

# Safe traversal with pruning and error handling.
import os
def handle_error(e):
    raise e

for root, dirs, files in os.walk("project", onerror=handle_error, topdown=True):
    if ".git" in dirs:
        dirs.remove(".git")
    # process

# Safe deletion.
shutil.rmtree("temp_tree")
```

**Component Breakdown**

| Practice | Description |
|----------|-------------|
| Prefer `pathlib` | Use `Path` for new code. |
| Use `exist_ok=True` | Avoid errors on existing directories. |
| Use `parents=True` | Create missing parents. |
| Validate user paths | Resolve and check against a base directory. |
| Use `onerror` with `walk` | Make errors explicit. |
| Prune `dirnames` | Skip irrelevant directories. |
| Caution with `rmtree` | Verify the path before recursive deletion. |

**Syntax Rules**

- Always use `exist_ok=True` when creating directories that may already exist.
- Use `Path.resolve()` or `os.path.realpath()` to canonicalize user-supplied paths.
- Use `Path.is_relative_to()` (Python 3.9+) to enforce a base directory.
- Provide `onerror` to `os.walk()` when silent error suppression is unacceptable.
- Sort `dirnames` and `filenames` if deterministic order is required.
- Use `followlinks=False` (default) unless symlink traversal is explicitly intended.

**Constraints and Limitations**

- `pathlib` may be slower than `os.scandir()` for very large directory scans on older Python versions.
- `shutil.rmtree()` cannot be undone; use with caution.
- Path validation is not a substitute for operating-system-level permissions.
- `exist_ok=True` only suppresses errors for the leaf directory, not for permission issues.

#### Annotated Code Examples

**Example 1: Safe Directory Creation and Validation**

```python
from pathlib import Path

def safe_make_dir(base: Path, user_input: str) -> Path:
    """Create a directory under base, preventing path traversal."""
    candidate = (base / user_input).resolve()
    base_resolved = base.resolve()
    if not candidate.is_relative_to(base_resolved):
        raise ValueError("Path escapes the base directory")
    candidate.mkdir(parents=True, exist_ok=True)
    return candidate

# Usage.
try:
    safe_make_dir(Path("uploads"), "user123/reports")
    print("Directory created safely.")
except ValueError as e:
    print("Rejected:", e)
```

**Expected Output:**

```
Directory created safely.
```

**Step-by-Step Breakdown:**

1. The candidate path is resolved to an absolute canonical path.
2. `is_relative_to()` ensures the candidate stays within the base directory.
3. `mkdir(parents=True, exist_ok=True)` creates the directory idempotently.

**Why the Output is Produced:**

Resolving and validating the path prevents directory traversal attacks.

#### Real-World Cases

- **Web upload handlers**: Validating and creating user-specific upload directories safely.
- **Build pipelines**: Idempotently creating output directories across repeated runs.
- **Cleanup scripts**: Carefully deleting temporary trees without touching unrelated data.
- **Log management**: Traversing log directories with pruning to skip archived subdirectories.

---

## References

- Python Documentation – `os` — Miscellaneous operating system interfaces - https://docs.python.org/3/library/os.html
- Python Documentation – `os.mkdir()` - https://docs.python.org/3/library/os.html#os.mkdir
- Python Documentation – `os.makedirs()` - https://docs.python.org/3/library/os.html#os.makedirs
- Python Documentation – `os.rmdir()` - https://docs.python.org/3/library/os.html#os.rmdir
- Python Documentation – `os.walk()` - https://docs.python.org/3/library/os.html#os.walk
- Python Documentation – `os.scandir()` - https://docs.python.org/3/library/os.html#os.scandir
- Python Documentation – `shutil.rmtree()` - https://docs.python.org/3/library/shutil.html#shutil.rmtree
- Python Documentation – `pathlib` — Object-oriented filesystem paths - https://docs.python.org/3/library/pathlib.html
- Python Documentation – `pathlib.Path.iterdir()` - https://docs.python.org/3/library/pathlib.html#pathlib.Path.iterdir
- Python Documentation – `pathlib.Path.glob()` - https://docs.python.org/3/library/pathlib.html#pathlib.Path.glob
- Python Documentation – `pathlib.Path.rglob()` - https://docs.python.org/3/library/pathlib.html#pathlib.Path.rglob
- PEP 428 – The `pathlib` module – object-oriented filesystem paths - https://peps.python.org/pep-0428/
- Real Python – Working With Files in Python - https://realpython.com/working-with-files-in-python/
- The Quick Python Book, Fourth Edition – Chapter 12: Using the filesystem - https://www-qa.manning.com/preview/the-quick-python-book-fourth-edition/chapter-12
- Python os Module: File and Directory Operations Guide - https://docs.kanaries.net/topics/Python/python-os-module
- List files in a directory with Python (Guance) - https://www.guance.com/learn/articles/question-266-python
- Path Traversal Defense with `pathlib` (Inspector) - https://inspector.pypi.io
- Python Documentation – `glob` — Unix style pathname pattern expansion - https://docs.python.org/3/library/glob.html