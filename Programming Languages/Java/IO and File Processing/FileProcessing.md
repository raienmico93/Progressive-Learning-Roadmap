# Java File Processing: A Comprehensive Cheat Sheet

## Topic Overview

### Core Definition

**Java File Processing** encompasses the APIs and techniques for interacting with the file system—creating, reading, writing, copying, moving, deleting files and directories, and accessing file metadata. Modern Java development centers on the **NIO.2 API** (`java.nio.file`), which replaced the legacy `java.io.File` class starting in Java 7.

### Technical Definition

The `java.io.File` class represents an abstract pathname (a file or directory on the file system) and provides methods for querying and manipulating file system objects, though many methods return boolean values rather than throwing exceptions, limiting diagnostic capability . The NIO.2 API (`java.nio.file.Path`, `java.nio.file.Files`, and `java.nio.file.attribute`) provides a more robust, cross-platform, and exception-rich approach. `Path` is an immutable, OS-agnostic representation of a file system location, while `Files` is a utility class of static methods for performing file operations .

### Beginner-Friendly Explanation

Think of the file system as a building with many rooms (files) and floors (directories). The legacy `File` class is like an old, unreliable map that sometimes gives you the wrong information silently. The modern `Path` and `Files` API is like a GPS that tells you exactly what's wrong (e.g., "Access denied" or "File not found") and works the same way on Windows, Linux, and macOS.

### Key Characteristics

- **Legacy vs. modern**: `File` is legacy; `Path`/`Files` is the modern standard .
- **Exception-rich**: NIO.2 methods throw specific exceptions (`NoSuchFileException`, `AccessDeniedException`) .
- **Cross-platform**: `Path` handles OS-specific separators automatically .
- **Metadata support**: NIO.2 provides full access to file attributes (permissions, owner, timestamps) .
- **Stream-based traversal**: `Files.walk()` and `Files.list()` enable efficient directory streaming .

### Prerequisites

- Basic Java syntax (classes, methods, exceptions).
- Understanding of `try-with-resources` for resource management.
- Familiarity with the `java.nio.file` package structure.

### Related Programming Areas

- **I/O Fundamentals**: Streams and readers/writers for file content.
- **File Attributes**: POSIX permissions and metadata management.
- **Directory Traversal**: Recursive file tree processing.
- **Resource Management**: Auto-closing streams and directory handles.

### Core Concepts / Features

1. File Class (Legacy) vs. Path and Files (Modern NIO.2)
2. File Creation and Deletion
3. File Copying and Moving
4. Directory Operations (Creating, Deleting, Listing)
5. Metadata and File Attributes Management
6. Streaming Directory Contents (`Files.list()`, `Files.walk()`, `Files.find()`)


## Core Concept 1: File Class (Legacy) vs. Path and Files (Modern NIO.2)

### Definitions

**Core Definition**: The `java.io.File` class is the legacy API for representing file paths, while `java.nio.file.Path` and `java.nio.file.Files` form the modern NIO.2 API introduced in Java 7 for more robust and feature-rich file system operations.

**Technical Definition**: `File` is an abstract representation of file and directory pathnames, but many of its methods return `boolean` on failure without providing diagnostic information . `Path` is an immutable, OS-independent interface representing a file system location, and `Files` is a utility class containing static methods for all file system operations . `File.toPath()` converts legacy instances, and `Path.toFile()` provides backward compatibility .

**Beginner-Friendly Explanation**: `File` is like an old, silent butler—when something goes wrong, he just says "No" without explaining. `Path`/`Files` is a modern assistant who tells you exactly what went wrong and works the same in every house (OS).

### Purposes

- **Legacy `File`**: To maintain backward compatibility with pre-Java 7 code .
- **Modern `Path`/`Files`**: To provide reliable, cross-platform, exception-rich file operations .
- To separate the path representation (`Path`) from the operation execution (`Files`).

### Syntax Rules and Structure

**Legacy File:**

```java
File file = new File("data.txt");
boolean created = file.createNewFile(); // Returns false on failure, no reason
```

**Modern NIO.2:**

```java
Path path = Path.of("data.txt");
Files.createFile(path); // Throws IOException with specific reason on failure
```

**Comparison Table:**

| Feature | `java.io.File` | `java.nio.file.Path` + `Files` |
|---------|----------------|-------------------------------|
| Error handling | Boolean returns, no details | Detailed exceptions  |
| Cross-platform | Partial | Excellent  |
| Metadata support | Limited | Full (POSIX, owner, permissions)  |
| Directory traversal | Inefficient, memory issues | Stream-based, scalable  |
| Symbolic links | No real support | Full support  |

**Syntax Rules:**

- Use `Path.of(String)` or `Paths.get(String)` to create paths (Java 11+ prefers `Path.of`).
- Use `Files.createFile(path)` instead of `file.createNewFile()`.
- Use `Files.delete(path)` instead of `file.delete()` for better error reporting .
- Convert with `file.toPath()` and `path.toFile()` .

**Constraints and Limitations:**

- `File` methods don't throw exceptions; failures are silent .
- `File` cannot reliably handle symbolic links or circular references .
- `Path` represents a location; it does not perform operations—`Files` does .

### Annotated Complete Code Examples

**Example 1: Legacy vs. Modern File Creation**

```java
import java.io.File;
import java.io.IOException;
import java.nio.file.*;

public class FileVsPathDemo {
    public static void main(String[] args) {
        // Legacy approach
        File legacyFile = new File("legacy.txt");
        try {
            boolean created = legacyFile.createNewFile();
            System.out.println("Legacy created: " + created); // false if exists
        } catch (IOException e) {
            System.out.println("Legacy error: " + e.getMessage());
        }

        // Modern approach
        Path modernPath = Path.of("modern.txt");
        try {
            Files.createFile(modernPath);
            System.out.println("Modern file created.");
        } catch (FileAlreadyExistsException e) {
            System.out.println("Modern: File already exists.");
        } catch (IOException e) {
            System.out.println("Modern error: " + e.getMessage());
        }
    }
}
```

**Expected Output:**

```
Legacy created: true
Modern file created.
```

**Why This Output Occurs:**
- `File.createNewFile()` returns `true` if created, `false` if it exists .
- `Files.createFile()` throws `FileAlreadyExistsException` if the file exists, providing clearer diagnostics .

### Real-World Cases

- **Legacy system integration**: Use `File.toPath()` to migrate old code .
- **New projects**: Always use `Path`/`Files` for reliability .
- **Cross-platform tools**: NIO.2 handles Windows/Linux path differences automatically .

### References

- Oracle – Legacy File I/O Code - https://docs.oracle.com/javase/tutorial/essential/io/legacy.html 
- Dev.java – Accessing Resources using Paths - https://dev.java/learn/java-io/file-system/file-path/ 
- CodeGym – File, Path, Files - https://codegym.cc/quests/lectures/en.codegym.java.core.lecture.level39.lecture02 


## Core Concept 2: File Creation and Deletion

### Definitions

**Core Definition**: **File creation** is the process of creating a new, empty file or directory on the file system; **file deletion** removes an existing file or directory.

**Technical Definition**: In NIO.2, `Files.createFile(Path)` creates a new empty file and throws `FileAlreadyExistsException` if it already exists. `Files.createDirectories(Path)` creates a directory and any necessary parent directories. `Files.delete(Path)` deletes a file or empty directory, throwing `NoSuchFileException` if it doesn't exist and `DirectoryNotEmptyException` if the directory is not empty. `Files.deleteIfExists(Path)` provides a non-throwing alternative .

**Beginner-Friendly Explanation**: Creating a file is like reserving a new room in a building. Deleting it is like removing the room—but you can't remove a floor (directory) if there are still rooms (files) inside it.

### Purposes

- To reserve a new location on the file system for data storage.
- To create directory structures for organizing files.
- To clean up files and directories when they are no longer needed.
- To provide safe, exception-rich deletion with clear failure reasons.

### Syntax Rules and Structure

**Complete General Syntax:**

```java
// File creation
Files.createFile(Path.of("newfile.txt"));

// Directory creation (single level)
Files.createDirectory(Path.of("newdir"));

// Directory creation (with parents)
Files.createDirectories(Path.of("parent/child/grandchild"));

// Deletion
Files.delete(Path.of("file.txt"));
Files.deleteIfExists(Path.of("file.txt")); // Non-throwing
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `Files.createFile(Path)` | Creates an empty file; throws if exists. |
| `Files.createDirectory(Path)` | Creates a single directory. |
| `Files.createDirectories(Path)` | Creates directory and all parents. |
| `Files.delete(Path)` | Deletes file or empty directory. |
| `Files.deleteIfExists(Path)` | Deletes if exists, returns `boolean`. |

**Syntax Rules:**

- `Files.createDirectories()` does not throw if the directory already exists (returns the path) .
- `Files.delete()` requires the directory to be empty for directory deletion .
- `Files.deleteIfExists()` returns `true` if deleted, `false` if not found .

**Constraints and Limitations:**

- `Files.delete()` cannot delete non-empty directories; use recursive traversal first.
- On Windows, files may be locked by other processes, causing deletion to fail .
- `File.delete()` is unreliable; prefer `Files.delete()` .

### Annotated Complete Code Examples

**Example 1: Creating and Deleting Files/Directories**

```java
import java.nio.file.*;

public class CreateDeleteDemo {
    public static void main(String[] args) throws Exception {
        Path dir = Path.of("demo_dir");
        Path file = dir.resolve("demo.txt");

        // Create directory and file
        Files.createDirectories(dir);
        Files.createFile(file);
        System.out.println("Created: " + file);

        // Check existence
        System.out.println("Exists: " + Files.exists(file));

        // Delete file (directory must be empty to delete)
        Files.delete(file);
        System.out.println("File deleted.");

        Files.delete(dir);
        System.out.println("Directory deleted.");

        // Safe deletion
        boolean deleted = Files.deleteIfExists(file);
        System.out.println("DeleteIfExists (already gone): " + deleted);
    }
}
```

**Expected Output:**

```
Created: demo_dir/demo.txt
Exists: true
File deleted.
Directory deleted.
DeleteIfExists (already gone): false
```

**Why This Output Occurs:**
- `createDirectories` creates `demo_dir` and `createFile` creates `demo.txt` inside it .
- `delete` removes the file first, then the now-empty directory .
- `deleteIfExists` returns `false` because the file was already deleted .

### Real-World Cases

- **Log rotation**: Creating new log files and deleting old ones.
- **Temporary file management**: Using `Files.createTempFile()` for scratch space.
- **Build systems**: Creating output directories and cleaning them.

### References

- Oracle – Legacy File I/O Code (Mapping Table) - https://docs.oracle.com/javase/tutorial/essential/io/legacy.html 
- Yisu – Java File文件操作有哪些技巧 - https://m.yisu.com/zixun/979053.html 
- Apache JIRA – AMQ-9856 Windows File Locking - https://issues.apache.org/jira/browse/AMQ-9856 


## Core Concept 3: File Copying and Moving

### Definitions

**Core Definition**: **File copying** creates a duplicate of a file or directory at a new location; **file moving** relocates the original to a new location.

**Technical Definition**: `Files.copy(Path source, Path target, CopyOption...)` copies a file or directory. `Files.move(Path source, Path target, CopyOption...)` moves a file or directory, optionally as an atomic operation. Options include `REPLACE_EXISTING` (overwrite target) and `ATOMIC_MOVE` (move as a single operation, if supported) .

**Beginner-Friendly Explanation**: Copying is like photocopying a document and filing the copy elsewhere. Moving is like picking up the document and placing it in a new folder—the original location is empty afterward.

### Purposes

- To create backups of important files.
- To reorganize file system structure without data loss.
- To move files between directories or storage devices.
- To perform atomic moves that prevent partial file states .

### Syntax Rules and Structure

**Complete General Syntax:**

```java
// Copy with overwrite
Files.copy(source, target, StandardCopyOption.REPLACE_EXISTING);

// Move with overwrite
Files.move(source, target, StandardCopyOption.REPLACE_EXISTING);

// Atomic move (if supported)
Files.move(source, target, StandardCopyOption.ATOMIC_MOVE);
```

**Component Breakdown:**

| Option | Description |
|--------|-------------|
| `REPLACE_EXISTING` | Overwrites target if it exists . |
| `COPY_ATTRIBUTES` | Copies file attributes to target . |
| `ATOMIC_MOVE` | Performs move as atomic operation . |
| `NOFOLLOW_LINKS` | Does not follow symbolic links. |

**Syntax Rules:**

- `Files.copy()` throws `FileAlreadyExistsException` if target exists and `REPLACE_EXISTING` is not specified .
- `Files.move()` on the same partition is typically a rename (fast) .
- Moving a directory requires the directory to be empty if it spans partitions .
- `ATOMIC_MOVE` ensures no process sees a partial file .

**Constraints and Limitations:**

- `ATOMIC_MOVE` throws `AtomicMoveNotSupportedException` if the file system doesn't support it .
- Copying a directory copies only the directory itself, not its contents (unless using `walkFileTree`).
- On Windows, file locking can prevent moves .

### Annotated Complete Code Examples

**Example 1: Copying and Moving Files**

```java
import java.nio.file.*;

public class CopyMoveDemo {
    public static void main(String[] args) throws Exception {
        Path source = Path.of("source.txt");
        Path copy = Path.of("copy.txt");
        Path moved = Path.of("moved.txt");

        // Create source file
        Files.writeString(source, "Hello, NIO.2!");

        // Copy with overwrite
        Files.copy(source, copy, StandardCopyOption.REPLACE_EXISTING);
        System.out.println("Copied to: " + copy);

        // Move (rename) the copy
        Files.move(copy, moved, StandardCopyOption.REPLACE_EXISTING);
        System.out.println("Moved to: " + moved);

        // Verify
        System.out.println("Source still exists: " + Files.exists(source));
        System.out.println("Moved exists: " + Files.exists(moved));

        // Cleanup
        Files.delete(source);
        Files.delete(moved);
    }
}
```

**Expected Output:**

```
Copied to: copy.txt
Moved to: moved.txt
Source still exists: true
Moved exists: true
```

**Why This Output Occurs:**
- `copy()` duplicates the file without removing the source .
- `move()` relocates the copy to a new name; the source remains untouched .
- Both operations succeed because `REPLACE_EXISTING` handles any existing targets .

### Real-World Cases

- **Build artifacts**: Copying compiled JARs to a distribution folder.
- **File uploads**: Moving temporary uploads to permanent storage.
- **Backup systems**: Copying files to backup locations with timestamps.
- **Atomic configuration updates**: Using `ATOMIC_MOVE` to swap config files safely .

### References

- Tencent – Java 中文官方教程（复制和移动） - https://cloud.tencent.com.cn/developer/article/2420557 
- 林信良 – Java JDK 7学习笔记 - /hdd/m0105/deepseek/datasets/duxiu/duxiu_pdf/9990b693afd68df305cef9cc2172cfe7_114135557.pdf 


## Core Concept 4: Directory Operations

### Definitions

**Core Definition**: **Directory operations** include creating directories, deleting them, and listing their contents.

**Technical Definition**: `Files.createDirectories(Path)` creates a directory hierarchy, creating parents as needed. `Files.newDirectoryStream(Path)` provides a `DirectoryStream` for iterating over directory entries without loading all into memory. `Files.list(Path)` returns a `Stream<Path>` for the direct contents of a directory (non-recursive) .

**Beginner-Friendly Explanation**: A directory is a folder. Creating it is like making a new folder in a filing cabinet. Listing its contents is like opening the folder and reading the names of all documents inside—one at a time, so even a folder with millions of files doesn't overwhelm you.

### Purposes

- To organize files into logical groups.
- To enumerate directory contents for processing.
- To create nested directory structures (e.g., `year/month/day`).
- To efficiently handle large directories without memory issues .

### Syntax Rules and Structure

**Complete General Syntax:**

```java
// Create directories (with parents)
Files.createDirectories(Path.of("parent/child"));

// List direct contents (non-recursive) - MUST close the stream
try (Stream<Path> entries = Files.list(Path.of("directory"))) {
    entries.forEach(System.out::println);
}

// Traditional DirectoryStream
try (DirectoryStream<Path> stream = Files.newDirectoryStream(Path.of("directory"))) {
    for (Path entry : stream) {
        System.out.println(entry);
    }
}
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `Files.createDirectories(Path)` | Creates all parent directories if needed. |
| `Files.list(Path)` | Returns `Stream<Path>` of direct children . |
| `Files.newDirectoryStream(Path)` | Returns `DirectoryStream` for iteration . |

**Syntax Rules:**

- `Files.list()` must be used with `try-with-resources` because it holds a system resource .
- `Files.list()` does not recurse into subdirectories .
- `DirectoryStream` uses a single underlying directory handle, efficient for large directories .

**Constraints and Limitations:**

- `Files.list()` returns only direct children, not descendants .
- Operating on a closed stream throws `IllegalStateException` .
- Directory listings can fail with `AccessDeniedException` if permissions are insufficient .

### Annotated Complete Code Examples

**Example 1: Listing Directory Contents**

```java
import java.nio.file.*;
import java.util.stream.Stream;

public class DirectoryListDemo {
    public static void main(String[] args) throws Exception {
        Path dir = Path.of("demo_list");
        Files.createDirectories(dir);
        Files.writeString(dir.resolve("a.txt"), "A");
        Files.writeString(dir.resolve("b.txt"), "B");
        Files.createDirectory(dir.resolve("subdir"));

        System.out.println("Direct contents of " + dir + ":");
        try (Stream<Path> entries = Files.list(dir)) {
            entries.forEach(p -> System.out.println("  " + p.getFileName()));
        }

        // Cleanup
        Files.delete(dir.resolve("a.txt"));
        Files.delete(dir.resolve("b.txt"));
        Files.delete(dir.resolve("subdir"));
        Files.delete(dir);
    }
}
```

**Expected Output:**

```
Direct contents of demo_list:
  a.txt
  b.txt
  subdir
```

**Why This Output Occurs:**
- `Files.list(dir)` streams the direct entries: `a.txt`, `b.txt`, and `subdir` .
- The `try-with-resources` closes the underlying directory handle after use .
- `getFileName()` extracts just the name, not the full path.

### Real-World Cases

- **File indexing**: Building an index of all files in a directory.
- **Cleanup scripts**: Listing and removing temporary files.
- **Directory monitoring**: Polling for new files in a directory.

### References

- 霍斯曼 – 写给大忙人看的Java SE 8 (Files.list) - /hdd/m0205/deepseek/datasets/duxiu/duxiu_pdf/eb9ba57452f33bc2a47c0d9d6b04f59c_31243937.pdf 
- Google Error Prone – StreamResourceLeak - https://chromium.journaldev.googlesource.com/external/github.com/google/error-prone 
- OpenJDK – Files.walk() AccessDeniedException - https://mail.openjdk.org/pipermail/core-libs-dev/2016-May/041425.html 


## Core Concept 5: Metadata and File Attributes Management

### Definitions

**Core Definition**: **File metadata** is data about a file—its size, creation time, last modified time, owner, and permissions—as opposed to the file's content.

**Technical Definition**: NIO.2 provides the `java.nio.file.attribute` package for reading and writing file attributes. `Files.readAttributes(Path, Class)` reads attributes in bulk. `BasicFileAttributes` provides common attributes (size, times, type). `PosixFileAttributes` (on POSIX systems) adds owner, group, and the nine permission bits (`PosixFilePermission`) .

**Beginner-Friendly Explanation**: Metadata is like the information on a package's shipping label—who sent it, when, how much it weighs, and who's allowed to open it. The content inside is separate. NIO.2 lets you read and sometimes change that label.

### Purposes

- To check file size, creation time, and modification time.
- To read and modify file permissions (e.g., make a file read-only).
- To determine the file owner and group (on POSIX systems).
- To make decisions based on file attributes (e.g., skip large files).

### Syntax Rules and Structure

**Complete General Syntax:**

```java
// Read basic attributes
BasicFileAttributes attrs = Files.readAttributes(path, BasicFileAttributes.class);
long size = attrs.size();
FileTime created = attrs.creationTime();
FileTime modified = attrs.lastModifiedTime();

// Read POSIX attributes (UNIX/Linux/macOS)
PosixFileAttributes posix = Files.readAttributes(path, PosixFileAttributes.class);
UserPrincipal owner = posix.owner();
Set<PosixFilePermission> perms = posix.permissions();

// Set permissions
Files.setPosixFilePermissions(path, PosixFilePermissions.fromString("rw-r--r--"));
```

**Component Breakdown:**

| Attribute | Description |
|-----------|-------------|
| `size()` | File size in bytes . |
| `creationTime()` | When file was created. |
| `lastModifiedTime()` | When file was last changed. |
| `owner()` | File owner (POSIX) . |
| `permissions()` | Nine permission bits (POSIX) . |

**Syntax Rules:**

- `BasicFileAttributes` works on all file systems .
- `PosixFileAttributes` requires a POSIX-compliant file system (UNIX, Linux, macOS) .
- Use `Files.getFileAttributeView()` for dynamic attribute access .
- Permission strings use the format `rwxrwxrwx` (owner, group, others).

**Constraints and Limitations:**

- `PosixFileAttributeView` throws `UnsupportedOperationException` on Windows .
- Windows uses ACL-based permissions, not POSIX bits.
- Not all file systems support creation time.

### Annotated Complete Code Examples

**Example 1: Reading File Attributes**

```java
import java.nio.file.*;
import java.nio.file.attribute.*;
import java.util.Set;

public class MetadataDemo {
    public static void main(String[] args) throws Exception {
        Path file = Path.of("metadata.txt");
        Files.writeString(file, "Hello, metadata!");

        // Basic attributes
        BasicFileAttributes basic = Files.readAttributes(file, BasicFileAttributes.class);
        System.out.println("Size: " + basic.size() + " bytes");
        System.out.println("Created: " + basic.creationTime());
        System.out.println("Modified: " + basic.lastModifiedTime());
        System.out.println("Is regular file: " + basic.isRegularFile());

        // POSIX attributes (if supported)
        try {
            PosixFileAttributes posix = Files.readAttributes(file, PosixFileAttributes.class);
            System.out.println("Owner: " + posix.owner().getName());
            System.out.println("Permissions: " + PosixFilePermissions.toString(posix.permissions()));
        } catch (UnsupportedOperationException e) {
            System.out.println("POSIX attributes not supported on this platform.");
        }

        Files.delete(file);
    }
}
```

**Expected Output (on Linux/macOS):**

```
Size: 16 bytes
Created: 2026-09-23T10:30:00Z
Modified: 2026-09-23T10:30:00Z
Is regular file: true
Owner: user
Permissions: rw-r--r--
```

**Why This Output Occurs:**
- `BasicFileAttributes` provides size, times, and type .
- `PosixFileAttributes` provides owner and permissions on POSIX systems .
- The permissions `rw-r--r--` mean owner can read/write, group and others can read.

### Real-World Cases

- **Access control**: Checking if a file is readable before processing.
- **Backup systems**: Using modification time to determine what to back up.
- **File monitoring**: Detecting changes via `lastModifiedTime()`.
- **Security auditing**: Reading file permissions to verify access controls .

### References

- Oracle – PosixFileAttributeView (Java 21) - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/nio/file/attribute/PosixFileAttributeView.html 
- Oracle – PosixFileAttributeView (Java 11) - https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/attribute/PosixFileAttributeView.html 


## Core Concept 6: Streaming Directory Contents (`Files.list()`, `Files.walk()`, `Files.find()`)

### Definitions

**Core Definition**: **Streaming directory contents** means using Java Streams to lazily traverse directory entries, avoiding loading entire directory listings into memory.

**Technical Definition**: `Files.list(Path)` returns a `Stream<Path>` of the direct children of a directory (non-recursive). `Files.walk(Path, int, FileVisitOption...)` returns a `Stream<Path>` of all entries in a directory tree (recursive, depth-first). `Files.find(Path, int, BiPredicate, FileVisitOption...)` returns a `Stream<Path>` filtered by a predicate that receives both the path and its `BasicFileAttributes` .

**Beginner-Friendly Explanation**: Imagine a library with millions of books. Instead of asking for a list of every book at once (which would fill your desk), streaming lets you process them one at a time. `list()` gives you books on the current shelf; `walk()` gives you books on all shelves in the building; `find()` gives you only the books matching your criteria.

### Purposes

- To process large directory trees without memory exhaustion .
- To filter files by attributes (size, type, modification time) efficiently .
- To perform recursive operations (search, delete, copy) on directory trees.
- To leverage Stream API operations (`filter`, `map`, `collect`) on file system data.

### Syntax Rules and Structure

**Complete General Syntax:**

```java
// List direct children (non-recursive) - MUST close
try (Stream<Path> entries = Files.list(Path.of("dir"))) {
    entries.filter(Files::isRegularFile).forEach(System.out::println);
}

// Walk entire tree (recursive, depth-first) - MUST close
try (Stream<Path> entries = Files.walk(Path.of("dir"))) {
    entries.filter(Files::isRegularFile).forEach(System.out::println);
}

// Walk with depth limit
try (Stream<Path> entries = Files.walk(Path.of("dir"), 2)) {
    entries.forEach(System.out::println);
}

// Find with attribute predicate (more efficient than walk + filter)
try (Stream<Path> entries = Files.find(
        Path.of("dir"),
        Integer.MAX_VALUE,
        (path, attrs) -> attrs.isRegularFile() && attrs.size() > 1024)) {
    entries.forEach(System.out::println);
}
```

**Component Breakdown:**

| Method | Recursive? | Use Case |
|--------|-----------|----------|
| `Files.list()` | No | Direct children only  |
| `Files.walk()` | Yes | All descendants, depth-first  |
| `Files.find()` | Yes | All descendants with attribute filter  |

**Syntax Rules:**

- All three methods return streams that **must be closed** (use `try-with-resources`) .
- `Files.walk()` accepts an optional `maxDepth` parameter .
- `Files.find()` is more efficient than `walk()` when filtering by file attributes, because attributes are already loaded .
- `FileVisitOption.FOLLOW_LINKS` enables following symbolic links .

**Constraints and Limitations:**

- `Files.walk()` throws `AccessDeniedException` if it encounters a directory it cannot read .
- Workaround: use `Files.list()` with `flatMap` and permission checks .
- Operating on a closed stream throws `IllegalStateException` .

### Annotated Complete Code Examples

**Example 1: Walking and Finding Files**

```java
import java.nio.file.*;
import java.util.stream.Stream;
import java.io.IOException;

public class WalkFindDemo {
    public static void main(String[] args) throws Exception {
        Path root = Path.of("tree");
        Files.createDirectories(root.resolve("sub1/subsub"));
        Files.writeString(root.resolve("root.txt"), "root content");
        Files.writeString(root.resolve("sub1/file1.txt"), "small");
        Files.writeString(root.resolve("sub1/file2.txt"), "a".repeat(2000));
        Files.writeString(root.resolve("sub1/subsub/deep.txt"), "deep");

        // Walk: all entries recursively
        System.out.println("=== Walk (all files) ===");
        try (Stream<Path> entries = Files.walk(root)) {
            entries.filter(Files::isRegularFile)
                   .forEach(p -> System.out.println(p));
        }

        // Find: only files larger than 1000 bytes
        System.out.println("\n=== Find (files > 1000 bytes) ===");
        try (Stream<Path> entries = Files.find(root, Integer.MAX_VALUE,
                (path, attrs) -> attrs.isRegularFile() && attrs.size() > 1000)) {
            entries.forEach(p -> System.out.println(p + " (" + p.toFile().length() + " bytes)"));
        }

        // Cleanup
        Files.walk(root).sorted(java.util.Comparator.reverseOrder())
             .forEach(p -> { try { Files.delete(p); } catch (IOException ignored) {} });
    }
}
```

**Expected Output:**

```
=== Walk (all files) ===
tree/root.txt
tree/sub1/file1.txt
tree/sub1/file2.txt
tree/sub1/subsub/deep.txt

=== Find (files > 1000 bytes) ===
tree/sub1/file2.txt (2000 bytes)
```

**Why This Output Occurs:**
- `Files.walk()` recursively lists all files in depth-first order .
- `Files.find()` filters using the `BasicFileAttributes` provided by the predicate, avoiding redundant attribute reads .
- Only `file2.txt` exceeds 1000 bytes.

### Real-World Cases

- **Recursive search**: Finding all `.java` files in a project.
- **Cleanup operations**: Deleting all temporary files in a tree.
- **Log aggregation**: Finding all log files modified today.
- **Backup tools**: Walking a directory tree to copy files .

### References

- 霍斯曼 – 写给大忙人看的Java SE 8 (Files.walk, Files.find) - /hdd/m0205/deepseek/datasets/duxiu/duxiu_pdf/eb9ba57452f33bc2a47c0d9d6b04f59c_31243937.pdf 
- GitHub – Java NIO.2: Files and Paths - https://github.com/Sanjay-Mathivanan/JAVA_Zero-to-Advanced 
- Google Error Prone – StreamResourceLeak (Files.list/walk/find) - https://chromium.journaldev.googlesource.com/external/github.com/google/error-prone 
- OpenJDK – Files.walk() AccessDeniedException Workaround - https://mail.openjdk.org/pipermail/core-libs-dev/2016-May/041425.html 


## References

- Dev.java – Accessing Resources using Paths - https://dev.java/learn/java-io/file-system/file-path/ 
- Yisu – Debian Java如何进行文件操作 - https://m.yisu.com/ask/50237881.html 
- Tencent – Java 中文官方教程 2022 版（八） - https://cloud.tencent.com.cn/developer/article/2420557 
- OpenJDK – Files.walk() is unusable because of AccessDeniedException - https://mail.openjdk.org/pipermail/core-libs-dev/2016-May/041425.html 
- Oracle – PosixFileAttributeView (Java 21) - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/nio/file/attribute/PosixFileAttributeView.html 
- Oracle – Legacy File I/O Code - https://docs.oracle.com/javase/tutorial/essential/io/legacy.html 
- 霍斯曼 – 写给大忙人看的Java SE 8 - /hdd/m0205/deepseek/datasets/duxiu/duxiu_pdf/eb9ba57452f33bc2a47c0d9d6b04f59c_31243937.pdf 
- GitHub – Java NIO.2: Files and Paths - https://github.com/Sanjay-Mathivanan/JAVA_Zero-to-Advanced 
- CodeGym – File, Path, Files: getting file information - https://codegym.cc/quests/lectures/en.codegym.java.core.lecture.level39.lecture02 
- Compile-N-Run – Java File Class - https://github.com/Compile-N-Run/Compile-N-Run 