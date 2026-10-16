# Java NIO.2: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** NIO.2 (New I/O 2), introduced in Java 7, is a comprehensive file I/O API in the `java.nio.file` package that provides a modern, feature-rich alternative to the legacy `java.io.File` class.

**Technical Definition:** "The `java.nio.file` package and its related package, `java.nio.file.attribute`, provide comprehensive support for file I/O and for accessing the default file system" . The `Path` interface is "the primary entry point for the package" , and the `Files` class "contains methods that deal with file operations" . NIO.2 also introduces the `WatchService` API for file change notification and the `FileVisitor` interface for recursive directory traversal.

**Beginner-Friendly Explanation:** NIO.2 is the modern way to work with files and directories in Java. The old `java.io.File` class had many limitations — it returned booleans instead of throwing useful exceptions, didn't support symbolic links, and couldn't watch directories for changes. NIO.2 fixes all of these problems. Think of `Path` as a smart address for a file, `Files` as a toolbox full of file operations, `WatchService` as a security camera that watches for changes, and `FileVisitor` as a tour guide that walks you through every file in a directory tree.

### Key Characteristics

- **Path-Centric:** Uses the `Path` interface as the primary abstraction for file and directory locations .
- **Rich Error Handling:** Methods throw detailed `IOException` subclasses like `NoSuchFileException` and `AccessDeniedException` .
- **Comprehensive File Operations:** `Files` provides static utility methods for reading, writing, copying, moving, and deleting .
- **Stream-Based Directory Walking:** `Files.walk()` and `Files.walkFileTree()` enable recursive directory traversal .
- **File Watching:** `WatchService` monitors directories for file creation, modification, and deletion events .
- **Attribute Support:** `BasicFileAttributes` and `PosixFileAttributes` provide rich file metadata .
- **Asynchronous I/O:** `AsynchronousFileChannel` and `AsynchronousSocketChannel` enable callback-driven I/O.
- **Interoperability:** `Path.toFile()` and `File.toPath()` bridge between legacy and modern APIs .

### Prerequisites

- Basic understanding of Java classes and methods
- Familiarity with exceptions and try-catch-finally
- Knowledge of the `java.io` package (for comparison)
- Understanding of lambda expressions (for stream-based operations)

### Related Programming Areas

- **File I/O:** Reading and writing files using streams and channels.
- **Directory Operations:** Listing, walking, and watching directories.
- **File Attributes:** Metadata, permissions, and timestamps.
- **Asynchronous I/O:** Callback-driven file and network operations.
- **NIO:** The original New I/O API (channels, buffers, selectors) that NIO.2 builds upon.

### Core Concepts / Features

---

## 1. NIO.2 (Java 7 NIO.2)

### Definitions

**Core Definition:** NIO.2 is the file I/O API introduced in Java 7 as part of JSR 203, providing a modern replacement for `java.io.File`.

**Technical Definition:** "The `java.nio.file` package and its related package, `java.nio.file.attribute`, provide comprehensive support for file I/O and for accessing the default file system" . NIO.2 addresses the shortcomings of the legacy `File` class: "Many methods didn't throw exceptions when they failed, so it was impossible to obtain a useful error message... The rename method didn't work consistently across platforms... There was no real support for symbolic links... Accessing file metadata was inefficient" .

**Beginner-Friendly Explanation:** NIO.2 is a complete overhaul of Java's file handling. Instead of using the old `File` class with its boolean-returning methods, you use `Path` to represent locations and `Files` to perform operations. Every operation either succeeds or throws a descriptive exception, making errors much easier to diagnose.

### Purposes

- To provide a modern, robust API for file and directory operations
- To replace the legacy `java.io.File` class
- To offer rich error reporting with specific exception types
- To support symbolic links, file attributes, and directory watching
- To enable stream-based and asynchronous file I/O

### Syntax Rules and Structure

**NIO.2 Core Classes:**

| Class/Interface | Purpose |
|-----------------|---------|
| `Path` | Represents a file or directory location  |
| `Paths` | Factory class for creating `Path` instances  |
| `Files` | Static utility methods for file operations  |
| `FileSystems` | Factory for file systems and `WatchService`  |
| `WatchService` | Monitors directories for changes  |
| `FileVisitor` | Interface for recursive directory traversal  |
| `BasicFileAttributes` | Basic file metadata  |

### Annotated Complete Step-by-Step Code Examples

**Example 1: Basic NIO.2 File Operations**

```java
// File: Nio2BasicDemo.java
import java.io.IOException;
import java.nio.file.*;
import java.nio.charset.StandardCharsets;

public class Nio2BasicDemo {
    public static void main(String[] args) {
        // Create a Path (Java 11+ syntax)
        Path path = Path.of("nio2_example.txt");

        try {
            // Write a string to a file (Java 11+)
            Files.writeString(path, "Hello, NIO.2!", StandardCharsets.UTF_8);
            System.out.println("File written: " + path);

            // Read the file back
            String content = Files.readString(path, StandardCharsets.UTF_8);
            System.out.println("Content: " + content);

            // Check file existence
            System.out.println("Exists: " + Files.exists(path));
            System.out.println("Is regular file: " + Files.isRegularFile(path));
            System.out.println("Size: " + Files.size(path) + " bytes");

            // Delete the file
            Files.delete(path);
            System.out.println("File deleted");

        } catch (IOException e) {
            System.out.println("I/O error: " + e.getMessage());
        }
    }
}
```

**Expected Output:**
```
File written: nio2_example.txt
Content: Hello, NIO.2!
Exists: true
Is regular file: true
Size: 13 bytes
File deleted
```

**Step-by-Step Explanation:**

1. `Path.of("nio2_example.txt")` creates a `Path` for the file.
2. `Files.writeString()` writes text using UTF-8 encoding.
3. `Files.readString()` reads the entire file into a string.
4. `Files.exists()`, `Files.isRegularFile()`, and `Files.size()` inspect the file.
5. `Files.delete()` removes the file.
6. All operations throw `IOException` on failure, providing clear error messages.

### Real-World Cases

- **Application Configuration:** Loading and saving configuration files with UTF-8 encoding.
- **File Uploads:** Handling uploaded files with proper error reporting.
- **Log Processing:** Reading and writing log files efficiently.
- **Data Import/Export:** Processing CSV, JSON, and other data formats.

---

## 2. `Path`

### Definitions

**Core Definition:** `Path` is an interface representing a hierarchical location of a file or directory on a file system.

**Technical Definition:** "An object that may be used to locate a file in a file system. It will typically represent a system dependent file path. A `Path` represents a path that is hierarchical and composed of a sequence of directory and file name elements separated by a special separator or delimiter" . "A `Path` is considered to be an empty path if it consists solely of one name element that is empty" .

**Beginner-Friendly Explanation:** A `Path` is like a smart address for a file or directory. It knows its own structure — you can ask it for its file name, parent directory, root, or how many name elements it has. Unlike the old `File` class, `Path` uses a clean, modern API and works consistently across platforms.

### Purposes

- To represent file and directory locations in a platform-independent way
- To provide methods for accessing path components (file name, parent, root)
- To enable path manipulation (resolve, relativize, normalize)
- To serve as the entry point for all NIO.2 file operations

### Syntax Rules and Structure

**Creating Paths:**

```java
// Java 11+ preferred
Path path = Path.of("dir", "file.txt");

// Java 7+ (legacy)
Path path = Paths.get("dir", "file.txt");

// From URI
Path path = Path.of(URI.create("file:///path/to/file.txt"));
```

**Path Methods:**

| Method | Description |
|--------|-------------|
| `getFileName()` | Returns the file name component  |
| `getParent()` | Returns the parent path  |
| `getRoot()` | Returns the root component  |
| `getNameCount()` | Returns the number of name elements  |
| `getName(int)` | Returns a name element at the given index  |
| `resolve(Path)` | Combines paths  |
| `relativize(Path)` | Constructs a relative path between two paths  |
| `normalize()` | Removes redundant elements |
| `toAbsolutePath()` | Converts to an absolute path |
| `toRealPath()` | Returns the real path of an existing file  |
| `startsWith(Path)` / `endsWith(Path)` | Tests path prefixes/suffixes  |

**Syntax Rules:**

- `Path` instances are immutable and safe for concurrent use .
- Paths do not need to exist when the `Path` object is created .
- Use `resolve()` to combine paths and `relativize()` to compute relative paths.

**Constraints and Limitations:**

- `Path` is an interface; actual instances are provided by the file system.
- `toRealPath()` throws `IOException` if the file does not exist.
- Symbolic links require special handling.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Path Operations**

```java
// File: PathDemo.java
import java.nio.file.*;

public class PathDemo {
    public static void main(String[] args) {
        // Create a path
        Path path = Path.of("/home/joe/logs/foo.log");

        // Inspect path components
        System.out.println("toString: " + path);
        System.out.println("getFileName: " + path.getFileName());
        System.out.println("getParent: " + path.getParent());
        System.out.println("getRoot: " + path.getRoot());
        System.out.println("getNameCount: " + path.getNameCount());
        System.out.println("getName(0): " + path.getName(0));
        System.out.println("getName(2): " + path.getName(2));
        System.out.println("subpath(0,2): " + path.subpath(0, 2));

        // Resolve paths
        Path base = Path.of("/home/joe");
        Path resolved = base.resolve("documents/report.txt");
        System.out.println("Resolved: " + resolved);

        // Relativize paths
        Path other = Path.of("/home/joe/logs/access.log");
        Path relative = path.relativize(other);
        System.out.println("Relativized: " + relative);

        // Normalize a path with redundant elements
        Path messy = Path.of("/home/joe/../joe/./logs/foo.log");
        System.out.println("Normalized: " + messy.normalize());
    }
}
```

**Expected Output:**
```
toString: /home/joe/logs/foo.log
getFileName: foo.log
getParent: /home/joe/logs
getRoot: /
getNameCount: 4
getName(0): home
getName(2): logs
subpath(0,2): home/joe
Resolved: /home/joe/documents/report.txt
Relativized: access.log
Normalized: /home/joe/logs/foo.log
```

**Step-by-Step Explanation:**

1. `Path.of("/home/joe/logs/foo.log")` creates an absolute path.
2. `getFileName()` returns the final component `foo.log`.
3. `getParent()` returns the parent directory path.
4. `getRoot()` returns the root `/`.
5. `getNameCount()` returns `4` (home, joe, logs, foo.log).
6. `getName(0)` returns `home`, `getName(2)` returns `logs`.
7. `subpath(0, 2)` returns `home/joe` (elements 0 and 1).
8. `resolve()` combines a base path with a relative path.
9. `relativize()` computes the relative path from `path` to `other`.
10. `normalize()` removes `..` and `.` elements.

### Real-World Cases

- **File Browsers:** Displaying the directory tree using `getFileName()`, `getParent()`, and `getNameCount()`.
- **Configuration Loading:** Resolving configuration file paths relative to the application home.
- **Path Validation:** Checking that paths are absolute or relative as expected.

---

## 3. `Paths`

### Definitions

**Core Definition:** `Paths` is a utility class with static methods for creating `Path` instances.

**Technical Definition:** "The `Paths` class is a helper class that provides `get()` methods for creating `Path` instances" . The `Paths.get()` method is "shorthand for" `FileSystems.getDefault().getPath()` .

**Beginner-Friendly Explanation:** `Paths` is like a factory that builds `Path` objects for you. You give it a string or a sequence of strings, and it creates a `Path`.

### Purposes

- To provide factory methods for creating `Path` instances
- To convert between strings, URIs, and `Path` objects
- To serve as the primary entry point for NIO.2 path creation

### Syntax Rules and Structure

```java
// Create a Path from a string
Path path = Paths.get("/tmp/foo");

// Create a Path from parent and child
Path path = Paths.get("/tmp", "foo", "bar.txt");

// Create a Path from a URI
Path path = Paths.get(URI.create("file:///Users/joe/FileTest.java"));
```

**Syntax Rules:**

- `Paths.get()` accepts one or more strings that are joined to form a path .
- The first argument may be a URI.
- `Paths.get()` throws `InvalidPathException` if the path string contains invalid characters.

### Annotated Complete Step-by-Step Code Examples

```java
// File: PathsDemo.java
import java.net.URI;
import java.nio.file.*;

public class PathsDemo {
    public static void main(String[] args) {
        // From a single string
        Path p1 = Paths.get("/tmp/foo");
        System.out.println("p1: " + p1);

        // From multiple strings
        Path p2 = Paths.get(System.getProperty("user.home"), "logs", "app.log");
        System.out.println("p2: " + p2);

        // From a URI
        Path p3 = Paths.get(URI.create("file:///Users/joe/FileTest.java"));
        System.out.println("p3: " + p3);

        // Using FileSystems directly
        Path p4 = FileSystems.getDefault().getPath("/users/sally");
        System.out.println("p4: " + p4);
    }
}
```

**Expected Output (varies by system):**
```
p1: /tmp/foo
p2: /home/user/logs/app.log
p3: /Users/joe/FileTest.java
p4: /users/sally
```

**Step-by-Step Explanation:**

1. `Paths.get("/tmp/foo")` creates a `Path` from a single string.
2. `Paths.get(user.home, "logs", "app.log")` joins multiple path components.
3. `Paths.get(URI.create(...))` creates a `Path` from a URI.
4. `FileSystems.getDefault().getPath(...)` is the underlying call that `Paths.get()` delegates to.

### Real-World Cases

- **Cross-Platform Paths:** Using `System.getProperty("user.home")` to build platform-independent paths.
- **Configuration Files:** Creating paths to configuration files in standard locations.
- **Resource Loading:** Creating paths from URIs for resource access.

---

## 4. `Files`

### Definitions

**Core Definition:** `Files` is a utility class containing static methods for operating on files, directories, and other types of files.

**Technical Definition:** "The `java.nio.file.Files` class contains static utility methods that perform operations on files and directories" . Methods include reading (`readString`, `readAllLines`, `readAllBytes`), writing (`writeString`, `write`), copying (`copy`), moving (`move`), deleting (`delete`, `deleteIfExists`), and inspecting (`exists`, `size`, `isDirectory`) .

**Beginner-Friendly Explanation:** `Files` is like a Swiss Army knife for file operations. It has a method for almost everything you'd want to do with files and directories. All methods are static, so you just call `Files.methodName()` without creating an instance.

### Purposes

- To read and write file content concisely
- To copy, move, and delete files and directories
- To create files, directories, and temporary files
- To inspect file metadata and attributes
- To list directory contents and walk directory trees

### Syntax Rules and Structure

**Common `Files` Methods:**

| Category | Methods |
|----------|---------|
| Reading | `readString()`, `readAllLines()`, `readAllBytes()`, `lines()`, `newBufferedReader()` |
| Writing | `writeString()`, `write()`, `newBufferedWriter()` |
| Copy/Move | `copy()`, `move()` |
| Delete | `delete()`, `deleteIfExists()` |
| Create | `createFile()`, `createDirectory()`, `createDirectories()`, `createTempFile()` |
| Inspect | `exists()`, `isRegularFile()`, `isDirectory()`, `size()`, `getLastModifiedTime()` |
| List/Walk | `list()`, `walk()`, `walkFileTree()` |

### Annotated Complete Step-by-Step Code Examples

```java
// File: FilesDemo.java
import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.nio.file.*;
import java.util.List;

public class FilesDemo {
    public static void main(String[] args) {
        Path path = Path.of("files_demo.txt");
        Path copyPath = Path.of("files_demo_copy.txt");
        Path dir = Path.of("demo_dir");

        try {
            // Write and read
            Files.writeString(path, "Line 1\nLine 2\nLine 3\n", StandardCharsets.UTF_8);
            System.out.println("Written: " + path);

            List<String> lines = Files.readAllLines(path, StandardCharsets.UTF_8);
            System.out.println("Lines: " + lines);

            // Copy
            Files.copy(path, copyPath, StandardCopyOption.REPLACE_EXISTING);
            System.out.println("Copied to: " + copyPath);

            // Create directory and move
            Files.createDirectories(dir);
            Path moved = dir.resolve("moved.txt");
            Files.move(copyPath, moved, StandardCopyOption.REPLACE_EXISTING);
            System.out.println("Moved to: " + moved);

            // Inspect
            System.out.println("Size: " + Files.size(moved) + " bytes");
            System.out.println("Last modified: " + Files.getLastModifiedTime(moved));

            // Delete
            Files.delete(moved);
            Files.delete(path);
            Files.delete(dir);
            System.out.println("Cleaned up");

        } catch (IOException e) {
            System.out.println("I/O error: " + e.getMessage());
        }
    }
}
```

**Expected Output:**
```
Written: files_demo.txt
Lines: [Line 1, Line 2, Line 3]
Copied to: files_demo_copy.txt
Moved to: demo_dir/moved.txt
Size: 21 bytes
Last modified: 2026-10-16T10:30:00Z
Cleaned up
```

**Step-by-Step Explanation:**

1. `Files.writeString()` writes text to a file using UTF-8.
2. `Files.readAllLines()` reads all lines into a `List<String>`.
3. `Files.copy()` copies the file with `REPLACE_EXISTING`.
4. `Files.createDirectories()` creates the target directory.
5. `Files.move()` moves the file into the directory.
6. `Files.size()` and `Files.getLastModifiedTime()` inspect the file.
7. `Files.delete()` removes the files and directory.

### Real-World Cases

- **Configuration Management:** Reading and writing `.properties` files.
- **Log Processing:** Reading log files line by line with `Files.lines()`.
- **File Uploads:** Copying uploaded files to permanent storage.
- **Temporary Files:** Creating and cleaning up temporary files.

---

## 5. File Operations

### Definitions

**Core Definition:** File operations are the actions performed on files, including creating, reading, writing, copying, moving, and deleting.

**Technical Definition:** NIO.2 provides comprehensive file operation methods in the `Files` class, including `createFile()`, `createDirectory()`, `createDirectories()`, `copy()`, `move()`, `delete()`, `deleteIfExists()`, `readString()`, `writeString()`, `readAllLines()`, `readAllBytes()`, and `write()`.

**Beginner-Friendly Explanation:** File operations are the everyday tasks you perform with files. NIO.2 makes these tasks simple with single-line method calls and clear error reporting.

### Purposes

- To create, read, write, copy, move, and delete files
- To manage file lifecycle
- To process file content efficiently
- To handle file operations safely with proper error handling

### Syntax Rules and Structure

**File Operation Methods:**

| Operation | Method | Notes |
|-----------|--------|-------|
| Create file | `Files.createFile(path)` | Throws `FileAlreadyExistsException` if exists |
| Create directories | `Files.createDirectories(path)` | Creates parents if needed |
| Write string | `Files.writeString(path, content)` | Java 11+ |
| Read string | `Files.readString(path)` | Java 11+ |
| Read lines | `Files.readAllLines(path)` | Returns `List<String>` |
| Copy | `Files.copy(source, target, options)` | Options: `REPLACE_EXISTING`, `COPY_ATTRIBUTES` |
| Move | `Files.move(source, target, options)` | Options: `REPLACE_EXISTING`, `ATOMIC_MOVE` |
| Delete | `Files.delete(path)` | Throws if not found |
| Delete if exists | `Files.deleteIfExists(path)` | Returns `boolean` |

### Annotated Complete Step-by-Step Code Examples

```java
// File: FileOperationsDemo.java
import java.io.IOException;
import java.nio.file.*;
import java.nio.charset.StandardCharsets;

public class FileOperationsDemo {
    public static void main(String[] args) {
        Path dir = Path.of("operations_dir");
        Path file = dir.resolve("data.txt");

        try {
            // Create directories
            Files.createDirectories(dir);
            System.out.println("Directory created: " + dir);

            // Create and write
            Files.writeString(file, "Important data\n", StandardCharsets.UTF_8);
            System.out.println("File written: " + file);

            // Read
            String content = Files.readString(file, StandardCharsets.UTF_8);
            System.out.println("Content: " + content.trim());

            // Copy
            Path backup = dir.resolve("data.backup.txt");
            Files.copy(file, backup, StandardCopyOption.COPY_ATTRIBUTES);
            System.out.println("Backup created: " + backup);

            // Move
            Path archive = dir.resolve("archive");
            Files.createDirectories(archive);
            Files.move(backup, archive.resolve("data.backup.txt"),
                StandardCopyOption.REPLACE_EXISTING);
            System.out.println("Moved to archive");

            // Delete
            Files.deleteIfExists(file);
            Files.deleteIfExists(archive.resolve("data.backup.txt"));
            Files.deleteIfExists(archive);
            Files.deleteIfExists(dir);
            System.out.println("Cleaned up");

        } catch (IOException e) {
            System.out.println("I/O error: " + e.getMessage());
        }
    }
}
```

**Expected Output:**
```
Directory created: operations_dir
File written: operations_dir/data.txt
Content: Important data
Backup created: operations_dir/data.backup.txt
Moved to archive
Cleaned up
```

**Step-by-Step Explanation:**

1. `Files.createDirectories(dir)` creates the directory and any missing parents.
2. `Files.writeString()` writes text to the file.
3. `Files.readString()` reads the content back.
4. `Files.copy()` creates a backup with `COPY_ATTRIBUTES`.
5. `Files.move()` moves the backup to an archive directory.
6. `Files.deleteIfExists()` cleans up all created files and directories.

### Real-World Cases

- **Backup Systems:** Copying files with `Files.copy()` and `COPY_ATTRIBUTES`.
- **Data Archiving:** Moving processed files to archive directories.
- **File Uploads:** Writing uploaded content to disk with `Files.write()`.
- **Log Rotation:** Renaming and archiving old log files.

---

## 6. Directory Operations

### Definitions

**Core Definition:** Directory operations are the actions performed on directories, including creation, listing, and deletion.

**Technical Definition:** NIO.2 provides directory operations through `Files.createDirectory()`, `Files.createDirectories()`, `Files.list()`, `Files.walk()`, `Files.walkFileTree()`, and `Files.newDirectoryStream()`. "Use `Files.list` or `Files.walk` to traverse directory entries" .

**Beginner-Friendly Explanation:** Directory operations let you work with folders — creating them, listing their contents, and walking through entire directory trees. `Files.list()` gives you immediate children, while `Files.walk()` goes deep into subdirectories.

### Purposes

- To create directories (including parent directories)
- To list directory contents
- To recursively walk directory trees
- To filter and process directory entries

### Syntax Rules and Structure

```java
// Create a single directory
Files.createDirectory(path);

// Create directory and parents
Files.createDirectories(path);

// List direct entries (must close the stream)
try (Stream<Path> entries = Files.list(dir)) {
    entries.forEach(System.out::println);
}

// Walk recursively (must close the stream)
try (Stream<Path> tree = Files.walk(dir)) {
    tree.filter(Files::isRegularFile).forEach(System.out::println);
}
```

**Syntax Rules:**

- `Files.list()` and `Files.walk()` return streams that must be closed (use try-with-resources) .
- `Files.walk()` follows symbolic links by default.
- `Files.createDirectories()` creates the directory and any necessary parent directories.

### Annotated Complete Step-by-Step Code Examples

```java
// File: DirectoryOpsDemo.java
import java.io.IOException;
import java.nio.file.*;
import java.util.stream.Stream;

public class DirectoryOpsDemo {
    public static void main(String[] args) {
        Path root = Path.of("dir_ops_demo");

        try {
            // Create directory structure
            Files.createDirectories(root.resolve("sub1/sub2"));
            Files.writeString(root.resolve("file1.txt"), "A");
            Files.writeString(root.resolve("file2.txt"), "B");
            Files.writeString(root.resolve("sub1/file3.txt"), "C");

            // List direct entries
            System.out.println("Direct entries:");
            try (Stream<Path> entries = Files.list(root)) {
                entries.forEach(p -> System.out.println("  " + p.getFileName()));
            }

            // Walk recursively
            System.out.println("All files (recursive):");
            try (Stream<Path> tree = Files.walk(root)) {
                tree.filter(Files::isRegularFile)
                    .forEach(p -> System.out.println("  " + p));
            }

            // Clean up (delete children before parents)
            try (Stream<Path> cleanup = Files.walk(root)) {
                cleanup.sorted(java.util.Comparator.reverseOrder())
                       .forEach(p -> {
                           try { Files.delete(p); }
                           catch (IOException e) { /* ignore */ }
                       });
            }

        } catch (IOException e) {
            System.out.println("I/O error: " + e.getMessage());
        }
    }
}
```

**Expected Output:**
```
Direct entries:
  file1.txt
  file2.txt
  sub1
All files (recursive):
  dir_ops_demo/file1.txt
  dir_ops_demo/file2.txt
  dir_ops_demo/sub1/file3.txt
```

**Step-by-Step Explanation:**

1. `Files.createDirectories()` creates the nested directory structure.
2. `Files.writeString()` creates test files.
3. `Files.list(root)` returns a stream of direct entries.
4. `Files.walk(root)` returns a recursive stream; `filter(Files::isRegularFile)` keeps only files.
5. Cleanup uses `Files.walk()` with reverse sorting to delete children before parents.

### Real-World Cases

- **Project Scanners:** Finding all `.java` files in a source tree.
- **Log Aggregation:** Reading all log files in a directory.
- **Build Tools:** Walking source directories for compilation.
- **Backup Systems:** Enumerating files to back up.

---

## 7. File Attributes

### Definitions

**Core Definition:** File attributes are metadata about a file, such as its size, creation time, last modified time, and type.

**Technical Definition:** NIO.2 provides `BasicFileAttributes` for "basic attributes associated with a file in a file system" and `PosixFileAttributes` for "file attributes associated with files on file systems used by operating systems that implement the Portable Operating System Interface (POSIX) family of standards" . Attributes are read via `Files.readAttributes(Path, Class)`.

**Beginner-Friendly Explanation:** File attributes are the "details" about a file — when it was created, when it was last changed, how big it is, and whether it's a file or a directory. NIO.2 provides rich metadata access through attribute views.

### Purposes

- To inspect file metadata for display or processing
- To determine when a file was last modified for cache validation
- To check file size before processing
- To identify file types and permissions

### Syntax Rules and Structure

```java
// Read basic attributes
BasicFileAttributes attrs = Files.readAttributes(path, BasicFileAttributes.class);

attrs.creationTime();      // FileTime
attrs.lastModifiedTime();  // FileTime
attrs.lastAccessTime();    // FileTime
attrs.size();              // long
attrs.isRegularFile();     // boolean
attrs.isDirectory();       // boolean
attrs.isSymbolicLink();    // boolean

// Read POSIX attributes (on POSIX systems)
PosixFileAttributes posixAttrs = Files.readAttributes(path, PosixFileAttributes.class);
posixAttrs.owner();        // UserPrincipal
posixAttrs.group();        // GroupPrincipal
posixAttrs.permissions();  // Set<PosixFilePermission>
```

**Attribute Views:**

| View | Description |
|------|-------------|
| `BasicFileAttributeView` | Basic attributes common to many file systems  |
| `PosixFileAttributeView` | POSIX permissions, owner, group  |
| `DosFileAttributeView` | Legacy DOS attributes  |
| `AclFileAttributeView` | Access Control Lists  |
| `FileOwnerAttributeView` | File owner  |

### Annotated Complete Step-by-Step Code Examples

```java
// File: AttributesDemo.java
import java.io.IOException;
import java.nio.file.*;
import java.nio.file.attribute.*;

public class AttributesDemo {
    public static void main(String[] args) {
        Path file = Path.of("attributes_demo.txt");

        try {
            Files.writeString(file, "Hello, attributes!");

            // Read basic attributes
            BasicFileAttributes attrs = Files.readAttributes(file, BasicFileAttributes.class);
            System.out.println("Created: " + attrs.creationTime());
            System.out.println("Modified: " + attrs.lastModifiedTime());
            System.out.println("Size: " + attrs.size() + " bytes");
            System.out.println("Is file: " + attrs.isRegularFile());

            // Try POSIX attributes
            if (Files.getFileStore(file).supportsFileAttributeView(PosixFileAttributeView.class)) {
                PosixFileAttributes posixAttrs =
                    Files.readAttributes(file, PosixFileAttributes.class);
                System.out.println("Owner: " + posixAttrs.owner().getName());
                System.out.println("Permissions: " +
                    PosixFilePermissions.toString(posixAttrs.permissions()));
            }

            Files.delete(file);

        } catch (IOException e) {
            System.out.println("I/O error: " + e.getMessage());
        }
    }
}
```

**Expected Output (on Linux/macOS):**
```
Created: 2026-10-16T10:30:00Z
Modified: 2026-10-16T10:30:00Z
Size: 18 bytes
Is file: true
Owner: user
Permissions: rw-r--r--
```

**Step-by-Step Explanation:**

1. `Files.writeString()` creates the file.
2. `Files.readAttributes()` reads `BasicFileAttributes`.
3. `attrs.creationTime()`, `attrs.lastModifiedTime()`, and `attrs.size()` provide metadata.
4. `PosixFileAttributes` is read if the file system supports POSIX.
5. `posixAttrs.owner()` and `PosixFilePermissions.toString()` provide owner and permissions.

### Real-World Cases

- **Cache Validation:** Checking `lastModifiedTime()` to decide if a cached file is stale.
- **File Managers:** Displaying file size and modification date.
- **Security:** Checking file permissions before accessing sensitive files.
- **Backup Systems:** Comparing modification times to determine what to back up.

---

## 8. File Walking

### Definitions

**Core Definition:** File walking is the process of recursively traversing a directory tree, visiting each file and directory.

**Technical Definition:** NIO.2 provides two approaches to file walking: the stream-based `Files.walk()` method, which returns a lazy `Stream<Path>`, and the visitor-based `Files.walkFileTree()` method, which accepts a `FileVisitor` to process each entry .

**Beginner-Friendly Explanation:** File walking is like exploring a file system tree. You start at the root and visit every file and directory. NIO.2 gives you two ways to do this: a stream-based approach that's concise and functional, and a visitor-based approach that gives you more control over the traversal.

### Purposes

- To recursively process all files in a directory tree
- To find files matching specific criteria
- To compute statistics (total size, file count) across a tree
- To implement file copy, move, or delete operations on entire trees

### Syntax Rules and Structure

```java
// Stream-based walking
try (Stream<Path> tree = Files.walk(startDir)) {
    tree.filter(Files::isRegularFile)
        .filter(p -> p.toString().endsWith(".java"))
        .forEach(System.out::println);
}

// Visitor-based walking
Files.walkFileTree(startDir, new SimpleFileVisitor<Path>() {
    @Override
    public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) {
        System.out.println("Visited: " + file);
        return FileVisitResult.CONTINUE;
    }
});
```

**`FileVisitor` Methods:**

| Method | Description |
|--------|-------------|
| `preVisitDirectory(dir, attrs)` | Called before entries in a directory are visited  |
| `visitFile(file, attrs)` | Called for a file in a directory  |
| `visitFileFailed(file, exc)` | Called for a file that could not be visited  |
| `postVisitDirectory(dir, exc)` | Called after all entries in a directory are visited  |

### Annotated Complete Step-by-Step Code Examples

```java
// File: FileWalkingDemo.java
import java.io.IOException;
import java.nio.file.*;
import java.nio.file.attribute.BasicFileAttributes;
import java.util.stream.Stream;

public class FileWalkingDemo {
    public static void main(String[] args) {
        Path root = Path.of("walk_demo");

        try {
            // Create sample structure
            Files.createDirectories(root.resolve("src/main"));
            Files.createDirectories(root.resolve("src/test"));
            Files.writeString(root.resolve("src/main/App.java"), "class App {}");
            Files.writeString(root.resolve("src/test/AppTest.java"), "class AppTest {}");
            Files.writeString(root.resolve("README.md"), "# Project");

            // Stream-based walking: find all .java files
            System.out.println("Java files (stream):");
            try (Stream<Path> tree = Files.walk(root)) {
                tree.filter(p -> p.toString().endsWith(".java"))
                    .forEach(p -> System.out.println("  " + p));
            }

            // Visitor-based walking: compute total size
            System.out.println("Visitor-based walking:");
            Files.walkFileTree(root, new SimpleFileVisitor<Path>() {
                @Override
                public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) {
                    System.out.println("  File: " + file.getFileName() +
                        " (" + attrs.size() + " bytes)");
                    return FileVisitResult.CONTINUE;
                }

                @Override
                public FileVisitResult postVisitDirectory(Path dir, IOException exc) {
                    System.out.println("  Directory: " + dir.getFileName());
                    return FileVisitResult.CONTINUE;
                }
            });

            // Clean up
            try (Stream<Path> cleanup = Files.walk(root)) {
                cleanup.sorted(java.util.Comparator.reverseOrder())
                       .forEach(p -> {
                           try { Files.delete(p); }
                           catch (IOException e) { /* ignore */ }
                       });
            }

        } catch (IOException e) {
            System.out.println("I/O error: " + e.getMessage());
        }
    }
}
```

**Expected Output:**
```
Java files (stream):
  walk_demo/src/main/App.java
  walk_demo/src/test/AppTest.java
Visitor-based walking:
  File: App.java (12 bytes)
  Directory: main
  File: AppTest.java (17 bytes)
  Directory: test
  File: README.md (9 bytes)
  Directory: src
  Directory: walk_demo
```

**Step-by-Step Explanation:**

1. `Files.createDirectories()` and `Files.writeString()` create a sample project structure.
2. `Files.walk(root)` returns a recursive stream; `filter()` keeps only `.java` files.
3. `Files.walkFileTree()` uses a `SimpleFileVisitor` that prints files and directories.
4. `postVisitDirectory()` is called after all entries in a directory are visited.
5. Cleanup uses `Files.walk()` with reverse sorting to delete children before parents.

### Real-World Cases

- **Source Code Analysis:** Finding all `.java` files in a project.
- **Build Tools:** Walking source directories for compilation.
- **Backup Systems:** Enumerating all files to back up.
- **Cleanup Utilities:** Deleting temporary files recursively.

---

## 9. File Watching

### Definitions

**Core Definition:** File watching is the process of monitoring a directory for changes such as file creation, modification, or deletion.

**Technical Definition:** "The `java.nio.file` package provides a file change notification API, called the Watch Service API. This API enables you to register a directory (or directories) with the watch service. When registering, you tell the service which types of events you are interested in: file creation, file deletion, or file modification" .

**Beginner-Friendly Explanation:** File watching is like setting up a security camera for a directory. Whenever a file is created, modified, or deleted, the WatchService notifies your program, so it can react immediately.

### Purposes

- To monitor directories for file changes
- To implement hot-reload and auto-refresh features
- To trigger actions when files are added or removed
- To build file synchronization tools

### Syntax Rules and Structure

```java
// Create a WatchService
WatchService watchService = FileSystems.getDefault().newWatchService();

// Register a directory for specific events
Path dir = Paths.get("watched_dir");
WatchKey key = dir.register(watchService,
    StandardWatchEventKinds.ENTRY_CREATE,
    StandardWatchEventKinds.ENTRY_MODIFY,
    StandardWatchEventKinds.ENTRY_DELETE);

// Event processing loop
while (true) {
    WatchKey signaledKey = watchService.take();  // Blocks until event
    for (WatchEvent<?> event : signaledKey.pollEvents()) {
        WatchEvent.Kind<?> kind = event.kind();
        Path affected = (Path) event.context();
        System.out.println(kind + ": " + affected);
    }
    signaledKey.reset();  // Ready for next events
}
```

**Event Types:**

| Event | Description |
|-------|-------------|
| `ENTRY_CREATE` | A new entry is made (file creation or rename)  |
| `ENTRY_MODIFY` | An existing entry is modified  |
| `ENTRY_DELETE` | An entry is deleted, moved, or renamed  |
| `OVERFLOW` | Lost or discarded events  |

### Annotated Complete Step-by-Step Code Examples

```java
// File: WatchServiceDemo.java
import java.io.IOException;
import java.nio.file.*;
import static java.nio.file.StandardWatchEventKinds.*;

public class WatchServiceDemo {
    public static void main(String[] args) throws IOException, InterruptedException {
        Path dir = Path.of("watched_dir");
        Files.createDirectories(dir);

        // Create the watch service
        WatchService watchService = FileSystems.getDefault().newWatchService();

        // Register the directory for events
        dir.register(watchService, ENTRY_CREATE, ENTRY_MODIFY, ENTRY_DELETE);
        System.out.println("Watching: " + dir.toAbsolutePath());

        // Start a thread to simulate file changes
        new Thread(() -> {
            try {
                Thread.sleep(500);
                Files.writeString(dir.resolve("test1.txt"), "Created");
                Thread.sleep(500);
                Files.writeString(dir.resolve("test1.txt"), "Modified");
                Thread.sleep(500);
                Files.delete(dir.resolve("test1.txt"));
            } catch (Exception e) { /* ignore */ }
        }).start();

        // Process events for a limited time
        long start = System.currentTimeMillis();
        while (System.currentTimeMillis() - start < 3000) {
            WatchKey key = watchService.poll();
            if (key == null) continue;

            for (WatchEvent<?> event : key.pollEvents()) {
                WatchEvent.Kind<?> kind = event.kind();
                Path affected = (Path) event.context();
                System.out.println(kind.name() + ": " + affected);
            }
            key.reset();
        }

        // Clean up
        Files.deleteIfExists(dir.resolve("test1.txt"));
        Files.deleteIfExists(dir);
        watchService.close();
        System.out.println("Done");
    }
}
```

**Expected Output:**
```
Watching: /current/dir/watched_dir
ENTRY_CREATE: test1.txt
ENTRY_MODIFY: test1.txt
ENTRY_DELETE: test1.txt
Done
```

**Step-by-Step Explanation:**

1. `FileSystems.getDefault().newWatchService()` creates a watch service .
2. `dir.register()` registers the directory for create, modify, and delete events .
3. `watchService.poll()` retrieves the next key without blocking indefinitely.
4. `key.pollEvents()` returns the list of events for that key.
5. `event.context()` returns the affected file name.
6. `key.reset()` resets the key so it can receive further events.
7. The watch service is closed with `close()`.

### Real-World Cases

- **IDE Auto-Refresh:** Detecting source file changes and reloading them.
- **Hot Reloading:** Automatically restarting applications when code changes.
- **File Synchronization:** Keeping directories in sync across systems.
- **Log Monitoring:** Watching for new log entries.

---

## 10. `WatchService`

### Definitions

**Core Definition:** `WatchService` is an interface for monitoring directories for file system events.

**Technical Definition:** "The `WatchService` API provides a scalable solution for monitoring directories for changes. It has a clean API and is so well optimized for performance that we don't need to implement our own solution" . "This API enables you to register a directory (or directories) with the watch service. When registering, you tell the service which types of events you are interested in" .

**Beginner-Friendly Explanation:** `WatchService` is like a notification service for your file system. You tell it which directories to watch and what kind of changes to look for, and it tells you when those changes happen.

### Purposes

- To monitor directories for file changes
- To implement reactive file processing
- To avoid inefficient polling
- To build file synchronization and backup tools

### Syntax Rules and Structure

**Key Methods:**

| Method | Description |
|--------|-------------|
| `FileSystems.getDefault().newWatchService()` | Creates a new watch service  |
| `path.register(watchService, events...)` | Registers a directory for events  |
| `watchService.take()` | Blocks until an event occurs |
| `watchService.poll()` | Returns the next key, or `null` if none |
| `watchService.poll(timeout, unit)` | Waits up to the timeout |
| `key.pollEvents()` | Returns the list of events |
| `key.reset()` | Resets the key for further events |
| `watchService.close()` | Closes the watch service |

**Syntax Rules:**

- The watch service monitors directories, not individual files.
- A `WatchKey` represents the registration of a directory.
- After processing events, `key.reset()` must be called to receive further events.
- The watch service should be closed with `close()` when no longer needed.

### Annotated Complete Step-by-Step Code Examples

```java
// File: WatchKeyDemo.java
import java.io.IOException;
import java.nio.file.*;
import static java.nio.file.StandardWatchEventKinds.*;

public class WatchKeyDemo {
    public static void main(String[] args) throws IOException, InterruptedException {
        Path dir = Path.of("watchkey_demo");
        Files.createDirectories(dir);

        WatchService ws = FileSystems.getDefault().newWatchService();
        WatchKey key = dir.register(ws, ENTRY_CREATE, ENTRY_DELETE);

        // Simulate a file creation
        new Thread(() -> {
            try {
                Thread.sleep(300);
                Files.writeString(dir.resolve("newfile.txt"), "hello");
            } catch (Exception e) { /* ignore */ }
        }).start();

        // Wait for events
        WatchKey signaled = ws.take();
        for (WatchEvent<?> event : signaled.pollEvents()) {
            System.out.println("Event: " + event.kind().name() +
                ", File: " + event.context());
        }

        // Reset the key to continue watching
        boolean valid = signaled.reset();
        System.out.println("Key reset: " + valid);

        // Clean up
        Files.deleteIfExists(dir.resolve("newfile.txt"));
        Files.deleteIfExists(dir);
        ws.close();
    }
}
```

**Expected Output:**
```
Event: ENTRY_CREATE, File: newfile.txt
Key reset: true
```

**Step-by-Step Explanation:**

1. `FileSystems.getDefault().newWatchService()` creates the watch service.
2. `dir.register(ws, ENTRY_CREATE, ENTRY_DELETE)` registers the directory for create and delete events.
3. `ws.take()` blocks until an event occurs.
4. `signaled.pollEvents()` returns the list of events.
5. `signaled.reset()` resets the key so it can receive further events.
6. `ws.close()` closes the watch service.

### Real-World Cases

- **File Upload Directories:** Watching for new file uploads and processing them.
- **Configuration Reloading:** Detecting changes to configuration files.
- **Build Automation:** Triggering builds when source files change.
- **Log Monitoring:** Watching for log file changes.

---

## 11. `FileVisitor`

### Definitions

**Core Definition:** `FileVisitor` is an interface for visiting files in a file tree, used with `Files.walkFileTree()`.

**Technical Definition:** "A visitor of files. An implementation of this interface is provided to the `Files.walkFileTree` methods to visit each file in a file tree" . The interface defines four methods: `preVisitDirectory`, `visitFile`, `visitFileFailed`, and `postVisitDirectory`.

**Beginner-Friendly Explanation:** `FileVisitor` is like a tour guide for a directory tree. You provide a visitor object that knows what to do at each stop (before entering a directory, when visiting a file, when leaving a directory). The `walkFileTree()` method takes your visitor and walks through every file and directory.

### Purposes

- To process each file in a directory tree
- To implement recursive file operations (copy, delete, search)
- To have fine-grained control over traversal
- To handle errors during traversal

### Syntax Rules and Structure

**`FileVisitor` Methods:**

| Method | Called When |
|--------|-------------|
| `preVisitDirectory(dir, attrs)` | Before entries in a directory are visited  |
| `visitFile(file, attrs)` | For each file in a directory  |
| `visitFileFailed(file, exc)` | When a file cannot be visited  |
| `postVisitDirectory(dir, exc)` | After all entries in a directory are visited  |

**Return Values (`FileVisitResult`):**

| Value | Meaning |
|-------|---------|
| `CONTINUE` | Continue walking |
| `TERMINATE` | Stop walking |
| `SKIP_SUBTREE` | Skip the current directory's entries |
| `SKIP_SIBLINGS` | Skip remaining siblings |

### Annotated Complete Step-by-Step Code Examples

```java
// File: FileVisitorDemo.java
import java.io.IOException;
import java.nio.file.*;
import java.nio.file.attribute.BasicFileAttributes;

public class FileVisitorDemo {
    public static void main(String[] args) throws IOException {
        Path root = Path.of("visitor_demo");
        Files.createDirectories(root.resolve("a/b"));
        Files.writeString(root.resolve("a/file1.txt"), "A");
        Files.writeString(root.resolve("a/b/file2.txt"), "B");

        // Custom FileVisitor
        Files.walkFileTree(root, new SimpleFileVisitor<Path>() {
            @Override
            public FileVisitResult preVisitDirectory(Path dir, BasicFileAttributes attrs) {
                System.out.println("Entering: " + dir.getFileName());
                return FileVisitResult.CONTINUE;
            }

            @Override
            public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) {
                System.out.println("  File: " + file.getFileName() +
                    " (" + attrs.size() + " bytes)");
                return FileVisitResult.CONTINUE;
            }

            @Override
            public FileVisitResult postVisitDirectory(Path dir, IOException exc) {
                System.out.println("Leaving: " + dir.getFileName());
                return FileVisitResult.CONTINUE;
            }
        });

        // Clean up
        try (var stream = Files.walk(root)) {
            stream.sorted(java.util.Comparator.reverseOrder())
                  .forEach(p -> {
                      try { Files.delete(p); }
                      catch (IOException e) { /* ignore */ }
                  });
        }
    }
}
```

**Expected Output:**
```
Entering: visitor_demo
Entering: a
  File: file1.txt (1 bytes)
Entering: b
  File: file2.txt (1 bytes)
Leaving: b
Leaving: a
Leaving: visitor_demo
```

**Step-by-Step Explanation:**

1. A sample directory structure is created with nested directories and files.
2. `Files.walkFileTree()` is called with a `SimpleFileVisitor`.
3. `preVisitDirectory()` is called before entering each directory.
4. `visitFile()` is called for each file.
5. `postVisitDirectory()` is called after leaving each directory.
6. The output shows the traversal order: enter root, enter `a`, visit `file1.txt`, enter `b`, visit `file2.txt`, leave `b`, leave `a`, leave root.

### Real-World Cases

- **File Copying:** Implementing recursive directory copy with `preVisitDirectory` and `visitFile`.
- **File Deletion:** Deleting entire directory trees with `visitFile` and `postVisitDirectory`.
- **Search Utilities:** Finding files matching criteria during traversal.
- **Disk Usage:** Computing total size of all files in a tree.

---

## 12. `SimpleFileVisitor`

### Definitions

**Core Definition:** `SimpleFileVisitor` is a convenience implementation of `FileVisitor` that provides default behaviour for all methods.

**Technical Definition:** "A simple visitor of files with default behavior to visit all files and to re-throw I/O errors" . It implements all four `FileVisitor` methods with default behaviour that continues traversal and re-throws I/O exceptions.

**Beginner-Friendly Explanation:** `SimpleFileVisitor` is like a template for a file visitor. Instead of implementing all four methods of `FileVisitor`, you extend `SimpleFileVisitor` and override only the methods you care about. The default behaviour is to continue visiting all files and re-throw any I/O errors.

### Purposes

- To simplify the implementation of `FileVisitor`
- To provide default behaviour that can be selectively overridden
- To reduce boilerplate code for common traversal patterns
- To handle I/O errors by default

### Syntax Rules and Structure

```java
Files.walkFileTree(startDir, new SimpleFileVisitor<Path>() {
    @Override
    public FileVisitResult visitFile(Path file, BasicFileAttributes attrs)
            throws IOException {
        // Custom behaviour for each file
        return FileVisitResult.CONTINUE;
    }

    @Override
    public FileVisitResult postVisitDirectory(Path dir, IOException exc)
            throws IOException {
        // Custom behaviour after visiting a directory
        return FileVisitResult.CONTINUE;
    }
});
```

**Default Behaviour:**

| Method | Default Behaviour |
|--------|-------------------|
| `preVisitDirectory` | Returns `CONTINUE` |
| `visitFile` | Returns `CONTINUE` |
| `visitFileFailed` | Re-throws the `IOException` |
| `postVisitDirectory` | Returns `CONTINUE` |

### Annotated Complete Step-by-Step Code Examples

```java
// File: SimpleFileVisitorDemo.java
import java.io.IOException;
import java.nio.file.*;
import java.nio.file.attribute.BasicFileAttributes;

public class SimpleFileVisitorDemo {
    public static void main(String[] args) throws IOException {
        Path root = Path.of("simple_visitor_demo");
        Files.createDirectories(root.resolve("docs"));
        Files.writeString(root.resolve("docs/readme.txt"), "Readme");
        Files.writeString(root.resolve("docs/guide.txt"), "Guide");
        Files.writeString(root.resolve("notes.md"), "Notes");

        // Count files and total size using SimpleFileVisitor
        final int[] fileCount = {0};
        final long[] totalSize = {0};

        Files.walkFileTree(root, new SimpleFileVisitor<Path>() {
            @Override
            public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) {
                fileCount[0]++;
                totalSize[0] += attrs.size();
                System.out.println("Visited: " + file.getFileName());
                return FileVisitResult.CONTINUE;
            }
        });

        System.out.println("Total files: " + fileCount[0]);
        System.out.println("Total size: " + totalSize[0] + " bytes");

        // Clean up
        try (var stream = Files.walk(root)) {
            stream.sorted(java.util.Comparator.reverseOrder())
                  .forEach(p -> {
                      try { Files.delete(p); }
                      catch (IOException e) { /* ignore */ }
                  });
        }
    }
}
```

**Expected Output:**
```
Visited: readme.txt
Visited: guide.txt
Visited: notes.md
Total files: 3
Total size: 18 bytes
```

**Step-by-Step Explanation:**

1. A sample directory structure is created.
2. `Files.walkFileTree()` is called with a `SimpleFileVisitor` that overrides `visitFile()`.
3. `visitFile()` increments the file count and adds to the total size.
4. The other methods use the default behaviour (continue and re-throw).
5. The total file count and size are printed.

### Real-World Cases

- **File Counters:** Counting files in a directory tree.
- **Size Calculators:** Computing total disk usage.
- **Search Utilities:** Finding files matching patterns.
- **Cleanup Tools:** Deleting files matching criteria.

---

## 13. NIO.2 Best Practices

### Definitions

**Core Definition:** NIO.2 best practices are guidelines for using the modern file I/O API effectively, safely, and efficiently.

**Technical Definition:** Best practices derive from authoritative sources including Oracle's Java Tutorials and Dev.java. Key principles include using NIO.2 instead of legacy `java.io.File`, using try-with-resources for streams, preferring `Files` utility methods, and using UTF-8 encoding.

**Beginner-Friendly Explanation:** Best practices help you use NIO.2 correctly without falling into common traps. They're like rules of the road for modern file I/O programming.

### Purposes

- To write efficient and maintainable file I/O code
- To avoid resource leaks and common pitfalls
- To ensure portability across platforms
- To handle errors gracefully

### Key Best Practices Table

| Practice | Description | Source |
|----------|-------------|--------|
| Prefer NIO.2 over `java.io.File` | "Java 7 introduced NIO.2 to replace the legacy `java.io.File` API" . | Dev.java |
| Use Try-with-Resources for Streams | "The stream returned by `Files.lines` needs to be closed. To ensure that this happens, use a try-with-resources statement" . | SEI CERT |
| Use `Files.writeString`/`readString` | "Use `Files.writeString()` and `Files.readString()` for concise text I/O" . | Dev.java |
| Use `Files.lines` for Large Files | "The `Files.lines` method returns a stream of the String objects, one per line from the file" . | Stack Overflow |
| Use `Files.walk` for Directory Traversal | "NIO.2 provides stream-based recursive directory traversal using `Files.walk()`" . | Dev.java |
| Specify Charset for Portability | Always specify UTF-8 for text files. | Best practice |
| Use `Path.of` (Java 11+) | Modern factory method for creating paths . | Dev.java |
| Handle Exceptions with `IOException` | NIO.2 methods throw descriptive exceptions. | Oracle |
| Use `Files.createDirectories` for Nested Dirs | Creates parent directories as needed. | Oracle |
| Use `WatchService` for File Monitoring | "WatchService provides a scalable solution for monitoring directories for changes" . | Baeldung |
| Use `SimpleFileVisitor` for Traversal | Extend `SimpleFileVisitor` and override only needed methods. | Oracle |
| Check File Attributes Before Operations | Use `Files.exists()`, `Files.isRegularFile()` before reading. | Best practice |

### Annotated Complete Step-by-Step Code Examples

```java
// File: Nio2BestPracticesDemo.java
import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.nio.file.*;
import java.util.stream.Stream;

public class Nio2BestPracticesDemo {
    public static void main(String[] args) {
        Path path = Path.of("best_practices.txt");

        // BEST PRACTICE: Use Files.writeString (Java 11+)
        try {
            Files.writeString(path, "Line 1\nLine 2\n", StandardCharsets.UTF_8);
            System.out.println("File written with UTF-8");
        } catch (IOException e) {
            System.out.println("Write error: " + e.getMessage());
            return;
        }

        // BEST PRACTICE: Use Files.readString (Java 11+)
        try {
            String content = Files.readString(path, StandardCharsets.UTF_8);
            System.out.println("Content: " + content.trim());
        } catch (IOException e) {
            System.out.println("Read error: " + e.getMessage());
        }

        // BEST PRACTICE: Use Files.lines with try-with-resources
        try (Stream<String> lines = Files.lines(path, StandardCharsets.UTF_8)) {
            lines.forEach(line -> System.out.println("Line: " + line));
        } catch (IOException e) {
            System.out.println("Error: " + e.getMessage());
        }

        // BEST PRACTICE: Check before operations
        if (Files.exists(path)) {
            System.out.println("File exists, size: " + Files.size(path));
        }

        // BEST PRACTICE: Use Files.walk for traversal
        try (Stream<Path> tree = Files.walk(Path.of("."))) {
            tree.filter(p -> p.toString().endsWith(".txt"))
                .forEach(p -> System.out.println("Found: " + p));
        } catch (IOException e) {
            System.out.println("Error: " + e.getMessage());
        }

        // Clean up
        try {
            Files.deleteIfExists(path);
            System.out.println("Cleaned up");
        } catch (IOException e) {
            System.out.println("Cleanup error: " + e.getMessage());
        }
    }
}
```

**Expected Output (varies):**
```
File written with UTF-8
Content: Line 1
Line 2
Line: Line 1
Line: Line 2
File exists, size: 14
Found: ./best_practices.txt
Cleaned up
```

**Step-by-Step Explanation:**

1. `Files.writeString()` writes text using UTF-8 encoding.
2. `Files.readString()` reads the entire file into a string.
3. `Files.lines()` returns a lazy stream processed line by line, enclosed in try-with-resources.
4. `Files.exists()` and `Files.size()` inspect the file before operations.
5. `Files.walk()` traverses the directory tree, filtering for `.txt` files.
6. `Files.deleteIfExists()` cleans up.

### Real-World Cases

- **Configuration Loading:** Reading configuration files with UTF-8.
- **Log Processing:** Processing large log files with `Files.lines()` for memory efficiency.
- **File Search:** Using `Files.walk()` to find files matching criteria.
- **Directory Monitoring:** Using `WatchService` to react to file changes.

---

## References Links

- File I/O (Featuring NIO.2) — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/essential/io/fileio.html
- Path Operations — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/essential/io/pathOps.html
- Watching a Directory for Changes — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/essential/io/notification.html
- Interface Path — Java Platform SE 11 API Specification - https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/Path.html
- Interface FileVisitor — Java Platform SE 21 API Specification - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/nio/file/FileVisitor.html
- A Guide To NIO2 WatchService — Baeldung - https://www.baeldung.com/java-nio2-watchservice
- Watching a Directory for Changes — Dev.java - https://dev.java/learn/java-io/file-system/watching-dir-changes/
- Java NIO.2: Files and Paths — Sanjay-Mathivanan/JAVA_Zero-to-Advanced (GitHub) - https://github.com/Sanjay-Mathivanan/JAVA_Zero-to-Advanced/blob/refs/heads/main/15_File-Handling/06_NIO_Files_and_Paths.md
- Package java.nio.file.attribute — Android Developers - https://android-dot-devsite-v2-prod.appspot.com/reference/java/nio/file/attribute/package-summary
- JAVA_Zero-to-Advanced — NIO Files and Paths (GitHub) - https://github.com/Sanjay-Mathivanan/JAVA_Zero-to-Advanced
- Advanced NIO.2 Stream Operations and Directory Handling — Open Exam Prep - https://open-exam-prep.com
- Five ways to maximize Java NIO and NIO.2 — InfoWorld - https://www.infoworld.com